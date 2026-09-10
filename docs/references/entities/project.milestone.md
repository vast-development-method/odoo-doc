# Project Milestone (`project.milestone`)

**Transport name:** `project.milestone`  
**Storage name:** `project_milestone`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `sale_project`

Description: Project Milestone

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `sequence, deadline, is_reached desc, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `sequence` | Sequence | integer |  | default `10` |
| `project_id` | Project | many to one | `project.project` | required; default computed dynamically (_get_default_project_id); indexed; on delete of the target: cascade; restricted by domain `[["is_template", "=", false]]` |
| `deadline` | Deadline | date |  | changes are tracked in the message thread; not copied on duplication |
| `is_reached` | Reached | boolean |  | default ; not copied on duplication |
| `reached_date` | Reached Date | date |  | computed by rule `_compute_reached_date` and stored |
| `task_ids` | Tasks | one to many | `project.task` | inverse field `milestone_id` |
| `project_allow_milestones` | Project Allow Milestones | boolean |  | computed by rule `_compute_project_allow_milestones` (not stored); searchable through a search rule |
| `is_deadline_exceeded` | Is Deadline Exceeded | boolean |  | computed by rule `_compute_is_deadline_exceeded` (not stored) |
| `is_deadline_future` | Is Deadline Future | boolean |  | computed by rule `_compute_is_deadline_future` (not stored) |
| `task_count` | # of Tasks | integer |  | computed by rule `_compute_task_count` (not stored); visible only to groups `project.group_project_milestone` |
| `done_task_count` | # of Done Tasks | integer |  | computed by rule `_compute_task_count` (not stored); visible only to groups `project.group_project_milestone` |
| `can_be_marked_as_done` | Can Be Marked As Done | boolean |  | computed by rule `_compute_can_be_marked_as_done` (not stored) |
| `allow_billable` | Allow Billable | boolean |  | related through path `project_id.allow_billable` |
| `project_partner_id` | Project Partner | many to one |  | related through path `project_id.partner_id` |
| `sale_line_id` | Sales Order Item | many to one | `sale.order.line` | default computed dynamically (_default_sale_line_id); indexed (btree_not_null); restricted by domain `[('order_partner_id', '=?', project_partner_id), ('qty_delivered_method', '=', 'milestones')]`; Help: Sales Order Item that will be updated once the milestone is reached. |
| `quantity_percentage` | Quantity (%) | float |  | computed by rule `_compute_quantity_percentage` and stored; Help: Percentage of the ordered quantity that will automatically be delivered once the milestone is reached. |
| `sale_line_display_name` | Sale Line Display Name | single line text |  | related through path `sale_line_id.display_name` |
| `product_uom_id` | Product Unit of measure | many to one |  | related through path `sale_line_id.product_uom_id` |
| `product_uom_qty` | Quantity | float |  | computed by rule `_compute_product_uom_qty` (not stored) |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_project_id` | preparation rule | self | `project` |  |  |
| `_compute_reached_date` | computation | self | `project` | depends: `is_reached` |  |
| `_compute_is_deadline_exceeded` | computation | self | `project` | depends: `is_reached`, `deadline` |  |
| `_compute_is_deadline_future` | computation | self | `project` | depends: `deadline` |  |
| `_compute_task_count` | computation | self | `project` | depends: `task_ids.milestone_id` |  |
| `_compute_can_be_marked_as_done` | computation | self | `project` |  |  |
| `_compute_project_allow_milestones` | computation | self | `project` | depends: `project_id.allow_milestones` |  |
| `_search_project_allow_milestones` | search rule | self, operator, value | `project` |  |  |
| `toggle_is_reached` | operation | self, is_reached | `project` |  |  |
| `action_view_tasks` | user action | self | `project` |  |  |
| `_get_fields_to_export` | preparation rule | self | `project`, `sale_project` | model |  |
| `_get_data` | preparation rule | self | `project` |  |  |
| `_get_data_list` | preparation rule | self | `project` |  |  |
| `copy` | lifecycle override | self, default | `project` |  |  |
| `_compute_display_name` | computation | self | `project` |  |  |
| `_default_sale_line_id` | preparation rule | self | `sale_project` |  |  |
| `_compute_quantity_percentage` | computation | self | `sale_project` | depends: `sale_line_id.product_uom_qty`, `product_uom_qty` |  |
| `_compute_product_uom_qty` | computation | self | `sale_project` | depends: `sale_line_id`, `quantity_percentage` |  |
| `action_view_sale_order` | user action | self | `sale_project` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `project` |
| `base.group_portal` | no | yes | no | no | `project` |
| `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project/Milestone: multi-company | global (all users) | `['\|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]` | True | True | True | True |
| Project/Milestone: employees: follow required for follower-only projects | `[(4, ref('base.group_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 ('project_id.user_id', '=', user.id),         ]` | True | True | True | True |
| Project/Milestone: Project manager can see all project milestones | `[(4, ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Project/milestone portal users: portal user can read with project sharing feature | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),         ]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.project_milestone_view_form` | form |  | `task_count`, `done_task_count`, `project_id`, `name`, `deadline`, `is_reached` | `%(project.action_view_task_from_milestone)d` |  | `project` |
| `project.project_milestone_view_tree` | list |  | `sequence`, `name`, `deadline`, `project_id`, `is_reached` | `View Tasks` |  | `project` |
| `project.project_milestone_view_kanban` | kanban |  | `can_be_marked_as_done`, `is_deadline_exceeded`, `is_reached`, `name`, `deadline` |  |  | `project` |
| `sale_project.project_milestone_view_form` | xpath | `project.project_milestone_view_form` | `allow_billable`, `project_partner_id`, `sale_line_id`, `sale_line_id`, `quantity_percentage`, `quantity_percentage`, `product_uom_qty`, `product_uom_id` |  |  | `sale_project` |
| `sale_project.project_milestone_view_tree` | xpath | `project.project_milestone_view_tree` | `project_partner_id`, `allow_billable`, `sale_line_id`, `sale_line_id`, `quantity_percentage`, `quantity_percentage`, `product_uom_qty`, `product_uom_qty` |  |  | `sale_project` |
| `sale_project.project_milestone_view_kanban_inherit_sale_project` | field | `project.project_milestone_view_kanban` | `is_deadline_exceeded`, `quantity_percentage` |  |  | `sale_project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.project_milestone_action` | Milestones | list,kanban,form | `[('project_id', '=', active_id)]` | `{'default_project_id': active_id}` |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.milestone.json`; views: `../../../schemas/interfaces/views/project.milestone.json`.
