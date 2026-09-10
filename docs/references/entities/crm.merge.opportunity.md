# Merge Opportunities (`crm.merge.opportunity`)

**Transport name:** `crm.merge.opportunity`  
**Storage name:** `crm_merge_opportunity`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `crm`

Description: Merge Opportunities

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `opportunity_ids` | Leads/Opportunities | many to many | `crm.lead` | association table `merge_opportunity_rel` |
| `user_id` | Salesperson | many to one | `res.users` | restricted by domain `[('share', '=', False)]` |
| `team_id` | Sales Team | many to one | `crm.team` | computed by rule `_compute_team_id` and stored |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `crm` | model | Use active_ids from the context to fetch the leads/opps to merge. In order to get merged, these leads/opps cannot be already 'Won' (closed) |
| `action_merge` | user action | self | `crm` |  |  |
| `_compute_team_id` | computation | self | `crm` | depends: `user_id` | When changing the user, also set a team_id or restrict team id to the ones user_id is member of. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.merge_opportunity_form` | form |  | `user_id`, `team_id`, `opportunity_ids`, `create_date`, `name`, `type`, `contact_name`, `email_from`, `phone`, `stage_id`, `user_id`, `team_id` | `Merge`, `Cancel` |  | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.action_merge_opportunities` | Merge | form |  |  | new | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.merge.opportunity.json`; views: `../../../schemas/interfaces/views/crm.merge.opportunity.json`.
