# Onboarding Progress Step Tracker (`onboarding.progress.step`)

**Transport name:** `onboarding.progress.step`  
**Storage name:** `onboarding_progress_step`  
**Kind:** persistent entity (one table)  
**Defined by package:** `onboarding`

Description: Onboarding Progress Step Tracker

## Identity and behavior

- Display name field: `step_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `progress_ids` | Related Onboarding Progress Tracker | many to many | `onboarding.progress` |  |
| `step_state` | Onboarding Step Progress | selection |  | default `not_done` |
| `step_id` | Onboarding Step | many to one | `onboarding.onboarding.step` | required; indexed; on delete of the target: cascade |
| `company_id` | Company | many to one | `res.company` | on delete of the target: cascade |

## State fields

State machine fields of this entity: `step_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_company_uniq` | UniqueIndex | `(step_id, COALESCE(company_id, 0))` |  | `onboarding` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_consolidate_just_done` | user action | self | `onboarding` |  |  |
| `action_set_just_done` | user action | self | `onboarding` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `onboarding` |
| `base.group_user` | no | no | no | no | `onboarding` |
| `base.group_system` | yes | yes | yes | yes | `onboarding` |

Machine-readable definition: `../../../schemas/data/entities/onboarding.progress.step.json`.
