# Sales Team Member (`crm.team.member`)

**Transport name:** `crm.team.member`  
**Storage name:** `crm_team_member`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sales_team`  
**Extended by packages:** `crm`

Description: Sales Team Member

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `create_date ASC, id`
- Display name field: `user_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `crm_team_id` | Sales Team | many to one | `crm.team` | required; default ; indexed; on delete of the target: cascade |
| `user_id` | Salesperson | many to one | `res.users` | required; indexed; on delete of the target: cascade; restricted by domain `[('share', '=', False), ('id', 'not in', user_in_teams_ids), ('company_ids', 'in', user_company_ids)]`; must belong to the same company |
| `user_in_teams_ids` | User In Teams | many to many | `res.users` | computed by rule `_compute_user_in_teams_ids` (not stored); Help: UX: Give users not to add in the currently chosen team to avoid duplicates |
| `user_company_ids` | User Company | many to many | `res.company` | computed by rule `_compute_user_company_ids` (not stored); Help: UX: Limit to team company or all if no company |
| `active` | Active | boolean |  | default `True` |
| `is_membership_multi` | Multiple Memberships Allowed | boolean |  | computed by rule `_compute_is_membership_multi` (not stored); Help: If True, users may belong to several sales teams. Otherwise membership is limited to a single sales team. |
| `member_warning` | Member Warning | multi line text |  | computed by rule `_compute_member_warning` (not stored) |
| `image_1920` | Image | image |  | related through path `user_id.image_1920` |
| `image_128` | Image (128) | image |  | related through path `user_id.image_128` |
| `name` | Name | single line text |  | related through path `user_id.display_name` |
| `email` | Email | single line text |  | related through path `user_id.email` |
| `phone` | Phone | single line text |  | related through path `user_id.phone` |
| `company_id` | Company | many to one | `res.company` | related through path `user_id.company_id` |
| `assignment_enabled` | Assignment Enabled | boolean |  | related through path `crm_team_id.assignment_enabled` |
| `assignment_domain` | Assignment Domain | single line text |  | changes are tracked in the message thread |
| `assignment_domain_preferred` | Preference assignment Domain | single line text |  | changes are tracked in the message thread |
| `assignment_optout` | Pause assignment | boolean |  |  |
| `assignment_max` | Average Leads Capacity (on 30 days) | integer |  | default `30` |
| `lead_day_count` | Leads (last 24h) | integer |  | computed by rule `_compute_lead_day_count` (not stored); Help: Number of leads assigned to this member in the last 24 hours (lost leads excluded) |
| `lead_month_count` | Leads (30 days) | integer |  | computed by rule `_compute_lead_month_count` (not stored); Help: Number of leads assigned to this member in the last 30 days |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_constrains_membership` | validation | self | `sales_team` | constrains: `crm_team_id`, `user_id`, `active` |  |
| `_constrains_company_membership` | validation | self | `sales_team` | constrains: `crm_team_id`, `user_id` |  |
| `_compute_user_in_teams_ids` | computation | self | `sales_team` | depends: `crm_team_id`, `is_membership_multi`, `user_id`; depends_context: `default_crm_team_id` | Give users not to add in the currently chosen team to avoid duplicates. In multi membership mode this field is empty as duplicates are allowed. |
| `_compute_user_company_ids` | computation | self | `sales_team` | depends: `crm_team_id` |  |
| `_compute_is_membership_multi` | computation | self | `sales_team` | depends: `crm_team_id` |  |
| `_compute_member_warning` | computation | self | `sales_team` | depends: `is_membership_multi`, `active`, `user_id`, `crm_team_id` | Display a warning message to warn user they are about to archive other memberships. Only valid in mono-membership mode and take into account only active memberships as we may keep several archived memberships. |
| `create` | lifecycle override | self, vals_list | `sales_team` | model_create_multi | Specific behavior implemented on create    * mono membership mode: other user memberships are automatically     archived (a warning already told it in form view);   * creating a membership already existing as archived: do nothing as     people can manage them from specific menu "Members";  Also remove autofollow on create. No need to follow team members when creating them as chatter is mainly used for information purpose (tracked fields). |
| `write` | lifecycle override | self, vals | `sales_team` |  | Specific behavior about active. If you change user_id / team_id user get warnings in form view and a raise in constraint check. We support archive / activation of memberships that toggles other memberships. But we do not support manual creation or update of user_id / team_id. This either works, either crashes). Indeed supporting it would lead to complex code with low added value. Users should create or remove members, and maybe archive / activate them. Updating manually memberships by modifying user_id or team_id is advanced and does not benefit from our support. |
| `_synchronize_memberships` | internal rule | self, user_team_ids | `sales_team` |  | Synchronize memberships: archive other memberships.  :param user_team_ids: list of pairs (user_id, crm_team_id) |
| `_compute_lead_day_count` | computation | self | `crm` | depends: `user_id`, `crm_team_id` |  |
| `_compute_lead_month_count` | computation | self | `crm` | depends: `user_id`, `crm_team_id` |  |
| `_get_lead_from_date` | preparation rule | self, date_from, active_test | `crm` |  |  |
| `_constrains_assignment_domain` | validation | self | `crm` | constrains: `assignment_domain` |  |
| `_constrains_assignment_domain_preferred` | validation | self | `crm` | constrains: `assignment_domain_preferred` |  |
| `_get_assignment_quota` | preparation rule | self, force_quota | `crm` |  | Return the remaining daily quota based on the assignment_max and the lead already assigned in the past 24h  :param bool force_quota: see `CrmTeam._action_assign_leads()`; |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_membership` | ValidationError | You are trying to create duplicate membership(s). We found that %(duplicates)s already exist(s). | `sales_team` |
| `_constrains_company_membership` | UserError | User '%(user)s' is not allowed in the company '%(company)s' of the Sales Team '%(team)s'. | `sales_team` |
| `_constrains_assignment_domain` | ValidationError | Member assignment domain for user %(user)s and team %(team)s is incorrectly formatted | `crm` |
| `_constrains_assignment_domain_preferred` | ValidationError | Member preferred assignment domain for user %(user)s and team %(team)s is incorrectly formatted | `crm` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `sales_team` |
| `base.group_user` | no | yes | no | no | `sales_team` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sales_team` |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_team_member_view_tree` | field | `sales_team.crm_team_member_view_tree` | `user_id`, `assignment_enabled`, `assignment_optout`, `assignment_max`, `lead_month_count` |  |  | `crm` |
| `crm.crm_team_member_view_kanban` | field | `sales_team.crm_team_member_view_kanban` | `active`, `assignment_enabled`, `assignment_optout`, `assignment_max` |  |  | `crm` |
| `crm.crm_team_member_view_form` | xpath | `sales_team.crm_team_member_view_form` | `lead_month_count`, `assignment_max`, `assignment_optout`, `assignment_domain_preferred`, `assignment_domain` |  |  | `crm` |
| `sales_team.crm_team_member_view_search` | search |  | `user_id`, `crm_team_id` |  | `Archived`, `Sales Team` | `sales_team` |
| `sales_team.crm_team_member_view_tree` | list |  | `crm_team_id`, `user_id` |  |  | `sales_team` |
| `sales_team.crm_team_member_view_tree_from_team` | xpath | `sales_team.crm_team_member_view_tree` |  |  |  | `sales_team` |
| `sales_team.crm_team_member_view_kanban` | kanban |  | `active`, `user_id`, `user_id`, `crm_team_id`, `email` |  |  | `sales_team` |
| `sales_team.crm_team_member_view_kanban_from_team` | xpath | `sales_team.crm_team_member_view_kanban` |  |  |  | `sales_team` |
| `sales_team.crm_team_member_view_form` | form |  | `active`, `company_id`, `is_membership_multi`, `member_warning`, `user_in_teams_ids`, `user_company_ids`, `member_warning`, `image_1920`, `user_id`, `crm_team_id`, `email`, `phone`, `company_id` | `crm_team_activate_multi_membership` |  | `sales_team` |
| `sales_team.crm_team_member_view_form_from_team` | xpath | `sales_team.crm_team_member_view_form` |  |  |  | `sales_team` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sales_team.crm_team_member_action` | Team Members | kanban,list,form |  |  |  | `sales_team` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.crm_team_member_config` | Teams Members | `crm_menu_config` | `sales_team.crm_team_member_action` | 6 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/crm.team.member.json`; views: `../../../schemas/interfaces/views/crm.team.member.json`.
