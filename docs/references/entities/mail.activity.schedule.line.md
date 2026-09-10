# Mail Activity Schedule Line (`mail.activity.schedule.line`)

**Transport name:** `mail.activity.schedule.line`  
**Storage name:** `mail_activity_schedule_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail`

Description: Mail Activity Schedule Line

## Identity and behavior

- Default ordering: `line_date_deadline asc, id asc`
- Display name field: `activity_schedule_id`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `activity_schedule_id` | Activity Schedule | many to one | `mail.activity.schedule` | required; on delete of the target: cascade |
| `line_description` | Line Description | single line text |  |  |
| `line_date_deadline` | Date Deadline | date |  |  |
| `responsible_user_id` | Responsible User | many to one | `res.users` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.schedule.line.json`.
