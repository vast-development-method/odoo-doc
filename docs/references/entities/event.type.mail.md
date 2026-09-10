# Mail Scheduling on Event Category (`event.type.mail`)

**Transport name:** `event.type.mail`  
**Storage name:** `event_type_mail`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_sms`

Description: Mail Scheduling on Event Category

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_type_id` | Event Type | many to one | `event.type` | required; on delete of the target: cascade |
| `interval_nbr` | Interval | integer |  | default `1` |
| `interval_unit` | Unit | selection |  | required; default `hours` |
| `interval_type` | Trigger | selection |  | required; default `before_event` |
| `notification_type` | Send | selection |  | computed by rule `_compute_notification_type` (not stored); extended by packages `event_sms` |
| `template_ref` | Template | reference |  | required; on delete of the target: {"sms.template": "cascade"}; extended by packages `event_sms` |

## Selection values

### `interval_unit` (Unit)

| Value | Label |
|---|---|
| `now` | Immediately |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

### `interval_type` (Trigger)

| Value | Label |
|---|---|
| `after_sub` | After each registration |
| `before_event` | Before the event starts |
| `after_event_start` | After the event started |
| `after_event` | After the event ended |
| `before_event_end` | Before the event ends |

### `notification_type` (Send)

| Value | Label |
|---|---|
| `mail` | Mail |
| `sms` | SMS |

### `template_ref` (Template)

| Value | Label |
|---|---|
| `mail.template` | Mail |
| `sms.template` | SMS |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_notification_type` | computation | self | `event_sms`, `event` | depends: `template_ref` | Assigns the type of template in use, if any is set. |
| `_prepare_event_mail_values` | preparation rule | self | `event` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |

Machine-readable definition: `../../../schemas/data/entities/event.type.mail.json`.
