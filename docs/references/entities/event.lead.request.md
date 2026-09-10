# Event Lead Request (`event.lead.request`)

**Transport name:** `event.lead.request`  
**Storage name:** `event_lead_request`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event_crm`

Description: Event Lead Request

## Identity and behavior

- Default ordering: `id asc`
- Display name field: `event_id`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_id` | Event | many to one | `event.event` | required; on delete of the target: cascade |
| `event_lead_rule_ids` | Lead Rules | many to many | `event.lead.rule` |  |
| `processed_registration_id` | Processed Registration | integer |  | Help: The ID of the last processed event.registration, used to know where to resume. |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uniq_event` | Constraint | `unique(event_id)` | You can only have one generation request per event at a time. | `event_crm` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_cron_generate_leads` | background operation | self, job_limit, registrations_batch_size | `event_crm` | model | See class docstring for details.  :param job_limit: The maximum amount of 'event.lead.request' to process   Defaults to 100. :param registrations_batch_size: The amount of attendees processed at once.   Defaults to event.lead.request._REGISTRATIONS_BATCH_SIZE |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `event_crm` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `event_crm.ir_cron_generate_leads` | Event CRM: Generate Leads based on Rules | 1 days | `_cron_generate_leads` |  |

Machine-readable definition: `../../../schemas/data/entities/event.lead.request.json`.
