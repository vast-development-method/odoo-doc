# Mass Mailing Statistics (`mailing.trace.report`)

**Transport name:** `mailing.trace.report`  
**Storage name:** `mailing_trace_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`

Description: Mass Mailing Statistics

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Mass Mail | single line text |  | read only |
| `mailing_type` | Type | selection |  | required; default `mail` |
| `campaign` | Mailing Campaign | single line text |  | read only |
| `scheduled_date` | Scheduled Date | date and time |  | read only |
| `state` | Status | selection |  | read only |
| `email_from` | From | single line text |  | read only |
| `scheduled` | Scheduled | integer |  | read only |
| `processing` | Processing | integer |  | read only |
| `pending` | Pending | integer |  | read only |
| `sent` | Sent | integer |  | read only |
| `delivered` | Delivered | integer |  | read only |
| `error` | Error | integer |  | read only |
| `opened` | Opened | integer |  | read only |
| `replied` | Replied | integer |  | read only |
| `bounced` | Bounced | integer |  | read only |
| `canceled` | Canceled | integer |  | read only |
| `clicked` | Clicked | integer |  | read only |

## Selection values

### `mailing_type` (Type)

| Value | Label |
|---|---|
| `mail` | Mail |

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `test` | Tested |
| `done` | Sent |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `mass_mailing` |  | Mass Mail Statistical Report: based on mailing.trace that models the various statistics collected for each mailing, and mailing.mailing model that models the various mailing performed. |
| `_report_get_request` | internal rule | self | `mass_mailing` |  |  |
| `_report_get_request_select_items` | internal rule | self | `mass_mailing` |  |  |
| `_report_get_request_from_items` | internal rule | self | `mass_mailing` |  |  |
| `_report_get_request_where_items` | internal rule | self | `mass_mailing` |  |  |
| `_report_get_request_group_by_items` | internal rule | self | `mass_mailing` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | no | yes | no | no | `mass_mailing` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_trace_report_view_tree` | list |  | `name`, `campaign`, `mailing_type`, `scheduled_date`, `state`, `scheduled`, `sent`, `processing`, `pending`, `delivered`, `opened`, `replied`, `clicked`, `canceled`, `error`, `bounced` |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_report_view_pivot` | pivot |  | `name`, `sent`, `scheduled`, `delivered`, `opened`, `replied`, `clicked`, `canceled`, `error`, `bounced` |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_report_view_graph` | graph |  | `name`, `sent`, `replied`, `clicked` |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_report_view_search` | search |  | `name`, `campaign`, `scheduled_date` |  | `filter_scheduled_date`, `Mass Mailing Campaign`, `State`, `Sent By`, `Scheduled Period` | `mass_mailing` |
| `mass_mailing_sms.mailing_trace_report_sms_view_tree` | xpath | `mass_mailing.mailing_trace_report_view_tree` |  |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_report_sms_view_pivot` | xpath | `mass_mailing.mailing_trace_report_view_pivot` |  |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_report_sms_view_graph` | xpath | `mass_mailing.mailing_trace_report_view_graph` |  |  |  | `mass_mailing_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_trace_report_action_mail` | Mass Mailing Analysis | graph,pivot,list | `[('mailing_type', '=', 'mail')]` |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_trace_report_action_sms` | SMS Marketing Analysis | graph,pivot,list | `[('mailing_type', '=', 'sms')]` |  |  | `mass_mailing_sms` |

Machine-readable definition: `../../../schemas/data/entities/mailing.trace.report.json`; views: `../../../schemas/interfaces/views/mailing.trace.report.json`.
