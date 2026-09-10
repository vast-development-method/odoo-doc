# Onboarding Progress Tracker (`onboarding.progress`)

**Transport name:** `onboarding.progress`  
**Storage name:** `onboarding_progress`  
**Kind:** persistent entity (one table)  
**Defined by package:** `onboarding`

Description: Onboarding Progress Tracker

## Identity and behavior

- Display name field: `onboarding_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `onboarding_state` | Onboarding progress | selection |  | computed by rule `_compute_onboarding_state` and stored |
| `is_onboarding_closed` | Was panel closed? | boolean |  |  |
| `company_id` | Company | many to one | `res.company` | on delete of the target: cascade |
| `onboarding_id` | Related onboarding tracked | many to one | `onboarding.onboarding` | required; indexed; on delete of the target: cascade |
| `progress_step_ids` | Progress Steps Trackers | many to many | `onboarding.progress.step` |  |

## State fields

State machine fields of this entity: `onboarding_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_onboarding_company_uniq` | UniqueIndex | `(onboarding_id, COALESCE(company_id, 0))` |  | `onboarding` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_onboarding_state` | computation | self | `onboarding` | depends: `onboarding_id.step_ids`, `progress_step_ids`, `progress_step_ids.step_state` |  |
| `_recompute_progress_step_ids` | internal rule | self | `onboarding` |  | Update progress steps when a step (with existing progress) is added to an onboarding. |
| `action_close` | user action | self | `onboarding` |  |  |
| `action_toggle_visibility` | user action | self | `onboarding` |  |  |
| `_get_and_update_onboarding_state` | preparation rule | self | `onboarding` |  | Fetch the progress of an onboarding for rendering its panel.  This method is expected to only be called by the onboarding controller. It also has the responsibility of updating the 'just_done' state into 'done' so that the 'just_done' states are only rendered once. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `onboarding` |
| `base.group_user` | no | no | no | no | `onboarding` |
| `base.group_system` | yes | yes | yes | yes | `onboarding` |

Machine-readable definition: `../../../schemas/data/entities/onboarding.progress.json`.
