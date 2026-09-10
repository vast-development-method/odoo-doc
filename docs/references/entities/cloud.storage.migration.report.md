# Cloud Storage Migration Report (`cloud.storage.migration.report`)

**Transport name:** `cloud.storage.migration.report`  
**Storage name:** `cloud_storage_migration_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `cloud_storage_migration`

Description: Cloud Storage Migration Report

## Identity and behavior

- Default ordering: `message_sum_size, all_sum_size DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model` | Model | single line text |  | read only |
| `res_model_name` | Model Name | single line text |  | read only; computed by rule `_compute_res_model_name` (not stored) |
| `message_sum_size` | Message Attachments Size (MB) | integer |  | read only; Help: Total size in megabytes of all attachments linked to mail messages for this model |
| `message_max_size` | Message Largest Attachment (MB) | integer |  | read only; Help: Size in megabytes of the largest attachment linked to mail messages for this model |
| `message_count` | Message Attachments Count | integer |  | read only; Help: Total number of attachments linked to mail messages for this model |
| `message_to_migrate` | Message Attachments Migration | boolean |  | computed by rule `_compute_message_to_migrate` (not stored); Help: Indicates whether attachments linked to mail messages for this model are scheduled for cloud storage migration |
| `all_sum_size` | Total Attachments Size (MB) | integer |  | read only; Help: Total size in megabytes of all attachments associated with records of this model |
| `all_max_size` | Largest Attachment (MB) | integer |  | read only; Help: Size in megabytes of the largest attachment associated with any record of this model |
| `all_count` | Total Attachments Count | integer |  | read only; Help: Total number of attachments associated with all records of this model |
| `all_to_migrate` | All Attachments Migration | boolean |  | computed by rule `_compute_all_to_migrate` (not stored); Help: Indicates whether all attachments associated with this model are scheduled for cloud storage migration |
| `has_attachment_rel` | Has Attachment Field | boolean |  | computed by rule `_compute_has_attachment_rel` (not stored); Help: Indicates whether this model has a relational field linking to ir.attachment model |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `cloud_storage_migration` |  | Initialize the SQL view for the cloud storage migration report. |
| `_compute_res_model_name` | computation | self | `cloud_storage_migration` | depends: `res_model` |  |
| `_compute_has_attachment_rel` | computation | self | `cloud_storage_migration` | depends: `res_model` |  |
| `_compute_all_to_migrate` | computation | self | `cloud_storage_migration` |  |  |
| `_compute_message_to_migrate` | computation | self | `cloud_storage_migration` |  |  |
| `get_progress` | operation | self | `cloud_storage_migration` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | no | yes | no | no | `cloud_storage_migration` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `cloud_storage_migration.view_cloud_storage_migration_report_list` | list |  | `res_model_name`, `message_count`, `message_sum_size`, `message_max_size`, `message_to_migrate`, `all_count`, `all_sum_size`, `all_max_size`, `all_to_migrate`, `has_attachment_rel` |  |  | `cloud_storage_migration` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `cloud_storage_migration.action_cloud_storage_migration_report` | Cloud Storage Migration Report | list |  |  |  | `cloud_storage_migration` |

Machine-readable definition: `../../../schemas/data/entities/cloud.storage.migration.report.json`; views: `../../../schemas/interfaces/views/cloud.storage.migration.report.json`.
