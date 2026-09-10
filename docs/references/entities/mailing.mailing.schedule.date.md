# schedule a mailing (`mailing.mailing.schedule.date`)

**Transport name:** `mailing.mailing.schedule.date`  
**Storage name:** `mailing_mailing_schedule_date`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mass_mailing`

Description: schedule a mailing

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `schedule_date` | Scheduled for | date and time |  |  |
| `mass_mailing_id` | Mass Mailing | many to one | `mailing.mailing` | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_schedule_date` | user action | self | `mass_mailing` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_mailing_schedule_date_view_form` | form |  | `schedule_date` | `Schedule`, `Discard` |  | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_mailing_schedule_date_action` | When do you want to send your mailing? | form |  |  | new | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.mailing.schedule.date.json`; views: `../../../schemas/interfaces/views/mailing.mailing.schedule.date.json`.
