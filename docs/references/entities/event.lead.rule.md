# Event Lead Rules (`event.lead.rule`)

**Transport name:** `event.lead.rule`  
**Storage name:** `event_lead_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event_crm`

Description: Event Lead Rules

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Rule Name | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `lead_ids` | Created Leads | one to many | `crm.lead` | visible only to groups `sales_team.group_sale_salesman`; inverse field `event_lead_rule_id` |
| `lead_creation_basis` | Create | selection |  | required; default `attendee`; Help: Per Attendee: A Lead is created for each Attendee (B2C). Per Order: A single Lead is created per Ticket Batch/Sale Order (B2B) |
| `lead_creation_trigger` | When | selection |  | required; default `create`; Help: Creation: at attendee creation; Registered: at attendee registration, manually or automatically; Attended: when attendance is confirmed and registration set to done; |
| `event_type_ids` | Event Templates | many to many | `event.type` | Help: Filter the attendees to include those of this specific event category. If not set, no event category restriction will be applied. |
| `event_id` | Event | many to one | `event.event` | restricted by domain `[('company_id', 'in', [company_id or current_company_id, False])]`; Help: Filter the attendees to include those of this specific event. If not set, no event restriction will be applied. |
| `company_id` | Company | many to one | `res.company` | Help: Restrict the trigger of this rule to events belonging to a specific company. If not set, no company restriction will be applied. |
| `event_registration_filter` | Registrations Domain | multi line text |  | Help: Filter the attendees that will or not generate leads. |
| `lead_type` | Lead Type | selection |  | required; default computed dynamically (lambda self: 'lead' if self.env.user.has_group('crm.group_use_lead') else 'opportunity'); Help: Default lead type when this rule is applied. |
| `lead_sales_team_id` | Sales Team | many to one | `crm.team` | on delete of the target: set null; Help: Automatically assign the created leads to this Sales Team. |
| `lead_user_id` | Salesperson | many to one | `res.users` | Help: Automatically assign the created leads to this Salesperson. |
| `lead_tag_ids` | Tags | many to many | `crm.tag` | Help: Automatically add these tags to the created leads. |

## Selection values

### `lead_creation_basis` (Create)

| Value | Label |
|---|---|
| `attendee` | Per Attendee |
| `order` | Per Order |

### `lead_creation_trigger` (When)

| Value | Label |
|---|---|
| `create` | Attendees are created |
| `confirm` | Attendees are registered |
| `done` | Attendees attended |

### `lead_type` (Lead Type)

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_lead_sales_team_id` | on change | self | `event_crm` | onchange: `lead_sales_team_id` |  |
| `_run_on_registrations` | background operation | self, registrations | `event_crm` |  | Create or update leads based on rule configuration. Two main lead management type exists    * per attendee: each registration creates a lead;   * per order: registrations are grouped per group and one lead is created     or updated with the batch (used mainly with sale order configuration     in event_sale);  Heuristic    * first, check existing lead linked to registrations to ensure no     duplication. Indeed for example attendee status change may trigger     the same rule several times;   * then for each rule, get the subset of registrations matching its     filters;   * then for each order- |
| `action_execute_rule` | user action | self | `event_crm` |  |  |
| `_filter_registrations` | internal rule | self, registrations | `event_crm` |  | Keep registrations matching rule conditions. Those are    * if a filter is set: filter registrations based on this filter. This is     done like a search, and filter is a domain;   * if a company is set on the rule, it must match event's company. Note     that multi-company rules apply on event_lead_rule;   * if an event category it set, it must match;   * if an event is set, it must match;   * if both event and category are set, one of them must match (OR). If none     of those are set, it is considered as OK;  :param registrations: event.registration recordset on which rule filters   will be |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event_crm` |
| `event.group_event_user` | no | yes | no | no | `event_crm` |
| `event.group_event_manager` | yes | yes | yes | yes | `event_crm` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `event_crm` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event CRM: Multi Company | `[(4, ref('base.group_multi_company'))]` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_crm.event_lead_rule_view_search` | search |  | `name` |  | `Archived`, `Creation Type`, `Trigger Type` | `event_crm` |
| `event_crm.event_lead_rule_view_tree` | list |  | `name`, `lead_creation_basis`, `lead_creation_trigger`, `event_type_ids`, `event_id`, `company_id` |  |  | `event_crm` |
| `event_crm.event_lead_rule_view_form` | form |  | `name`, `active`, `company_id`, `lead_creation_basis`, `lead_creation_trigger`, `event_type_ids`, `company_id`, `event_id`, `event_registration_filter`, `lead_type`, `lead_sales_team_id`, `lead_user_id`, `lead_tag_ids` | `Execute Rule` |  | `event_crm` |
| `event_crm_sale.event_lead_rule_view_tree` | xpath | `event_crm.event_lead_rule_view_tree` |  |  |  | `event_crm_sale` |
| `event_crm_sale.event_lead_rule_view_form` | xpath | `event_crm.event_lead_rule_view_form` |  |  |  | `event_crm_sale` |
| `website_event_crm.event_lead_rule_view_tree` | xpath | `event_crm.event_lead_rule_view_tree` |  |  |  | `website_event_crm` |
| `website_event_crm.event_lead_rule_view_form` | xpath | `event_crm.event_lead_rule_view_form` |  |  |  | `website_event_crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_crm.event_lead_rule_action` | Lead Generation Rule | list,form |  |  |  | `event_crm` |
| `event_crm.event_lead_rule_answer_action` | Event lead Rule | form |  |  |  | `event_crm` |

Machine-readable definition: `../../../schemas/data/entities/event.lead.rule.json`; views: `../../../schemas/interfaces/views/event.lead.rule.json`.
