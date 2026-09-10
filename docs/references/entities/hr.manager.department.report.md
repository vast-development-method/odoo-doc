# Hr Manager Department Report (`hr.manager.department.report`)

**Transport name:** `hr.manager.department.report`  
**Storage name:** `hr_manager_department_report`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `hr`

Description: Hr Manager Department Report

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | read only |
| `has_department_manager_access` | Has Department Manager Access | boolean |  | computed by rule `_compute_has_department_manager_access` (not stored); searchable through a search rule |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_search_has_department_manager_access` | search rule | self, operator, value | `hr` |  |  |
| `_compute_has_department_manager_access` | computation | self | `hr` |  |  |

Machine-readable definition: `../../../schemas/data/entities/hr.manager.department.report.json`.
