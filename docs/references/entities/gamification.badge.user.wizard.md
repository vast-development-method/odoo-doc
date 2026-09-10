# Gamification User Badge Wizard (`gamification.badge.user.wizard`)

**Transport name:** `gamification.badge.user.wizard`  
**Storage name:** `gamification_badge_user_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `gamification`  
**Extended by packages:** `hr_gamification`

Description: Gamification User Badge Wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; computed by rule `_compute_user_id` and stored; extended by packages `hr_gamification` |
| `badge_id` | Badge | many to one | `gamification.badge` | required |
| `comment` | Comment | multi line text |  |  |
| `employee_id` | Employee | many to one | `hr.employee` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_grant_badge` | user action | self | `gamification`, `hr_gamification` |  | Wizard action for sending a badge to a chosen user |
| `_compute_user_id` | computation | self | `hr_gamification` | depends: `employee_id` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_grant_badge` | UserError | You can not grant a badge to yourself. | `gamification` |
| `action_grant_badge` | UserError | You can not send a badge to yourself. | `hr_gamification` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `gamification` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.view_badge_wizard_grant` | form |  | `badge_id`, `user_id`, `comment` | `Grant Badge`, `Cancel` |  | `gamification` |
| `hr_gamification.view_badge_wizard_grant_employee` | data | `gamification.view_badge_wizard_grant` | `employee_id` |  |  | `hr_gamification` |
| `hr_gamification.view_badge_wizard_reward` | form |  | `employee_id`, `user_id`, `badge_id`, `badge_id`, `comment` | `Grant a badge`, `Discard` |  | `hr_gamification` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `gamification.action_grant_wizard` | Grant Badge |  |  | `{             'default_badge_id': active_id,             'badge_id': active_id         }` | new | `gamification` |
| `hr_gamification.action_reward_wizard` | Grant a badge | form | `[]` | `{'default_employee_id': active_id, 'employee_id': active_id, 'dialog_size': 'medium'}` | new | `hr_gamification` |

Machine-readable definition: `../../../schemas/data/entities/gamification.badge.user.wizard.json`; views: `../../../schemas/interfaces/views/gamification.badge.user.wizard.json`.
