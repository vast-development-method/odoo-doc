# Machine-readable catalogs

All catalogs are JavaScript Object Notation documents, encoded in Unicode Transformation Format eight bits, with keys in lower snake case. Every catalog has a top-level `catalog` object with `name`, `description`, `record_count` and `generated_from` (the specification revision), then `records`.

| Folder | Catalogs |
|---|---|
| `data/` | `entity-index.json`, `entities/<transport name>.json` (one per entity), `physical-tables.json`, `relations.json`, `selection-values.json`, `constraints.json`, `computed-fields.json`, `reference-data/*.json` |
| `interfaces/` | `routes.json`, `window-actions.json`, `server-actions.json`, `client-actions.json`, `address-actions.json`, `report-actions.json`, `menus.json`, `views/<transport name>.json`, `mail-templates.json`, `remote-operations.json`, `web-templates.json` |
| `operational/` | `scheduled-jobs.json`, `sequences.json`, `groups.json`, `privileges.json`, `access-rights.json`, `record-rules.json`, `system-parameters.json`, `decimal-precisions.json`, `message-subtypes.json`, `activity-types.json` |
| `source-artifacts/` | `packages.json`, `package-dependency-graph.json`, `package-contents.json`, `chart-templates.json` |
| `source-file-distribution/` | `files.json` (every file of this repository with purpose, domain, size and line count), `summary.json` |
| `traceability/` | `entity-name-dictionary.json`, `capability-to-entity.json`, `entity-to-document.json`, `acceptance-to-workflow.json`, `coverage.json` |

Identifiers in the catalogs are reproduced storage and transport names; each record also carries `full_name` in words.
