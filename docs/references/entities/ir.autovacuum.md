# Automatic Vacuum (`ir.autovacuum`)

**Transport name:** `ir.autovacuum`  
**Storage name:** `ir_autovacuum`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`

Description: Automatic Vacuum

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_run_vacuum_cleaner` | background operation | self | `base` |  | Perform a complete database cleanup by safely calling every ``@api.autovacuum`` decorated method. |
| `_gc_orm_signaling` | background operation | self | `base` | autovacuum |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `base.autovacuum_job` | Base: Auto-vacuum internal data | 1 days | `_run_vacuum_cleaner` | 3 |

Machine-readable definition: `../../../schemas/data/entities/ir.autovacuum.json`.
