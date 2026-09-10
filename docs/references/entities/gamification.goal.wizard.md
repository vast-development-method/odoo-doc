# Gamification Goal Wizard (`gamification.goal.wizard`)

**Transport name:** `gamification.goal.wizard`  
**Storage name:** `gamification_goal_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `gamification`

Description: Gamification Goal Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `goal_id` | Goal | many to one | `gamification.goal` | required |
| `current` | Current | float |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_update_current` | user action | self | `gamification` |  | Wizard action for updating the current value |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `gamification` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.view_goal_wizard_update_current` | form |  | `goal_id`, `current` | `Update`, `Cancel` |  | `gamification` |

Machine-readable definition: `../../../schemas/data/entities/gamification.goal.wizard.json`; views: `../../../schemas/interfaces/views/gamification.goal.wizard.json`.
