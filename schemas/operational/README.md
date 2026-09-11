# Operational catalogues

Configuration that governs how the system runs and who may do what. See [the catalogue index](../README.md) for the record shape.

| Catalogue | Content | Records |
|---|---|---|
| [`scheduled-jobs.json`](scheduled-jobs.json) | Every scheduled job with its interval, entity, operation and priority | 93 |
| [`sequences.json`](sequences.json) | Numbering sequences with prefix, suffix, padding and period reset | 16 |
| [`groups.json`](groups.json) | Security groups with their privilege family and implied groups | 140 |
| [`privileges.json`](privileges.json) | Privilege families that group ordered levels of access | 29 |
| [`package-categories.json`](package-categories.json) | The category tree for capability packages and privileges | 28 |
| [`access-rights.json`](access-rights.json) | Per entity and per group: create, read, update and delete | 1,933 |
| [`record-rules.json`](record-rules.json) | Row-level filters per entity, per group and per operation | 576 |
| [`system-parameters.json`](system-parameters.json) | Parameters shipped with default values | 28 |
| [`decimal-precisions.json`](decimal-precisions.json) | Named precisions and the quantities each governs | 7 |
| [`message-subtypes.json`](message-subtypes.json) | Message subtypes that followers subscribe to | 103 |
| [`activity-types.json`](activity-types.json) | Activity types with their delays, chaining and presentation | 16 |

The access catalogues are the source for the authorisation layer of the [equivalence test plan](../../docs/reimplementation/equivalence-test-plan.md).
