# Twilio Number (`sms.twilio.number`)

**Transport name:** `sms.twilio.number`  
**Storage name:** `sms_twilio_number`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sms_twilio`

Description: Twilio Number

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); indexed (btree); on delete of the target: cascade |
| `sequence` | Sequence | integer |  | default `1` |
| `number` | Twilio Number | single line text |  | required |
| `country_id` | Country | many to one | `res.country` | required |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `sms_twilio` |  |  |
| `action_unlink` | user action | self | `sms_twilio` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `sms_twilio` |

Machine-readable definition: `../../../schemas/data/entities/sms.twilio.number.json`.
