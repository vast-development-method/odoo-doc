# Maintenance Maintained Item (`maintenance.mixin`)

**Transport name:** `maintenance.mixin`  
**Storage name:** `maintenance_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `maintenance`

Description: Maintenance Maintained Item

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `effective_date` | Effective Date | date |  | required; default computed dynamically (fields.Date.context_today); Help: This date will be used to compute the Mean Time Between Failure. |
| `maintenance_team_id` | Maintenance Team | many to one | `maintenance.team` | computed by rule `_compute_maintenance_team_id` and stored; indexed (btree_not_null); must belong to the same company |
| `technician_user_id` | Technician | many to one | `res.users` | changes are tracked in the message thread |
| `maintenance_ids` | Maintenance | one to many | `maintenance.request` |  |
| `maintenance_count` | Maintenance Count | integer |  | computed by rule `_compute_maintenance_count` and stored |
| `maintenance_open_count` | Current Maintenance | integer |  | computed by rule `_compute_maintenance_count` and stored |
| `expected_mtbf` | Expected MTBF | integer |  | Help: Expected Mean Time Between Failure |
| `mtbf` | MTBF | integer |  | computed by rule `_compute_maintenance_request` (not stored); Help: Mean Time Between Failure, computed based on done corrective maintenances. |
| `mttr` | MTTR | integer |  | computed by rule `_compute_maintenance_request` (not stored); Help: Mean Time To Repair |
| `estimated_next_failure` | Estimated time before next failure (in days) | date |  | computed by rule `_compute_maintenance_request` (not stored); Help: Computed as Latest Failure Date + MTBF |
| `latest_failure_date` | Latest Failure Date | date |  | computed by rule `_compute_maintenance_request` (not stored) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_maintenance_team_id` | computation | self | `maintenance` | depends: `company_id` |  |
| `_compute_maintenance_request` | computation | self | `maintenance` | depends: `effective_date`, `maintenance_ids.stage_id`, `maintenance_ids.close_date`, `maintenance_ids.request_date` |  |
| `_compute_maintenance_count` | computation | self | `maintenance` | depends: `maintenance_ids.stage_id.done`, `maintenance_ids.archive` |  |

Machine-readable definition: `../../../schemas/data/entities/maintenance.mixin.json`.
