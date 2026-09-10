# Onboarding (`onboarding.onboarding`)

**Transport name:** `onboarding.onboarding`  
**Storage name:** `onboarding_onboarding`  
**Kind:** persistent entity (one table)  
**Defined by package:** `onboarding`  
**Extended by packages:** `account`

Description: Onboarding

## Identity and behavior

- Default ordering: `sequence asc, id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name of the onboarding | single line text |  | translatable |
| `route_name` | One word name | single line text |  | required |
| `step_ids` | Onboarding steps | many to many | `onboarding.onboarding.step` |  |
| `text_completed` | Message at completion | single line text |  | default computed dynamically (lambda s: s.env._('Nice work! Your configuration is done.')); Help: Text shown on onboarding when completed |
| `is_per_company` | Should be done per company? | boolean |  | read only; computed by rule `_compute_is_per_company` (not stored) |
| `panel_close_action_name` | Closing action | single line text |  | Help: Name of the onboarding model action to execute when closing the panel. |
| `current_progress_id` | Onboarding Progress | many to one | `onboarding.progress` | computed by rule `_compute_current_progress` (not stored); Help: Onboarding Progress for the current context (company). |
| `current_onboarding_state` | Completion State | selection |  | read only; computed by rule `_compute_current_progress` (not stored) |
| `is_onboarding_closed` | Was panel closed? | boolean |  | computed by rule `_compute_current_progress` (not stored) |
| `progress_ids` | Onboarding Progress Records | one to many | `onboarding.progress` | read only; inverse field `onboarding_id`; Help: All Onboarding Progress Records (across companies). |
| `sequence` | Sequence | integer |  | default `10` |

## State fields

State machine fields of this entity: `current_onboarding_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_route_name_uniq` | Constraint | `UNIQUE (route_name)` | Onboarding alias must be unique. | `onboarding` |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_per_company` | computation | self | `onboarding` | depends: `progress_ids`, `progress_ids.company_id`, `step_ids`, `step_ids.is_per_company` |  |
| `_compute_current_progress` | computation | self | `onboarding` | depends_context: `company`; depends: `progress_ids`, `progress_ids.is_onboarding_closed`, `progress_ids.onboarding_state`, `progress_ids.company_id` |  |
| `write` | lifecycle override | self, vals | `onboarding` |  | Recompute progress step ids if new steps are added/removed. |
| `action_close` | user action | self | `onboarding` |  | Close the onboarding panel. |
| `action_close_panel` | user action | self, xmlid | `onboarding` | model | Close the onboarding panel identified by its `xmlid`.  If not found, quietly do nothing. |
| `action_refresh_progress_ids` | user action | self | `onboarding` |  | Re-initialize onboarding progress records (after step is_per_company change).  Meant to be called when `is_per_company` of linked steps is modified (or per-company steps are added to an onboarding). |
| `action_toggle_visibility` | user action | self | `onboarding` |  |  |
| `_search_or_create_progress` | search rule | self | `onboarding` |  | Create Progress record(s) as necessary for the context. |
| `_create_progress` | internal rule | self | `onboarding` |  |  |
| `_prepare_rendering_values` | preparation rule | self | `account`, `onboarding` |  | Compute existence of invoices for company. |
| `action_close_panel_account_invoice` | user action | self | `account` | model |  |
| `action_close_panel_account_dashboard` | user action | self | `account` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `onboarding` |
| `base.group_user` | no | no | no | no | `onboarding` |
| `base.group_system` | yes | yes | yes | yes | `onboarding` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `onboarding.onboarding_onboarding_view_tree` | list |  | `sequence`, `name`, `current_onboarding_state`, `is_onboarding_closed`, `is_per_company` | `Toggle visibility` |  | `onboarding` |
| `onboarding.onboarding_onboarding_view_form` | form |  | `current_progress_id`, `name`, `route_name`, `is_per_company`, `is_onboarding_closed`, `step_ids` | `Toggle visibility` |  | `onboarding` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `onboarding.action_view_onboarding_onboarding` | Onboardings | list,form |  |  |  | `onboarding` |

Machine-readable definition: `../../../schemas/data/entities/onboarding.onboarding.json`; views: `../../../schemas/interfaces/views/onboarding.onboarding.json`.
