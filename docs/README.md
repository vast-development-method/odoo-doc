# Documentation tree

This folder holds the narrative specification. Read the root `README.md` first for conventions, the architecture at a glance and the capability map.

| Folder | Purpose | Start with |
|---|---|---|
| `overview/` | The platform as a whole: architecture, package system, entity and field system, inheritance and extension, security model, views and actions, messaging model, design principles | `overview/README.md` |
| `data/` | Domain model across all domains, persistence identity and value rules, physical data catalog, data loading and exchange | `data/README.md` |
| `domains/` | One folder per business domain, each with the same eleven files (entities, state machines, workflows, business rules, calculations, accounting effects, configuration, interfaces, acceptance criteria, glossary) | `domains/README.md` |
| `interfaces/` | Desktop workflows, endpoint catalog, external integrations, remote transport contracts, report and export documents, service layer | `interfaces/README.md` |
| `runtime/` | Request lifecycle, sessions, transactions and concurrency, caching, scheduled jobs, notification bus, mail gateway, attachments and file store, report rendering, translation, background workers | `runtime/README.md` |
| `references/` | Generated exhaustive references for every entity, route, view, action, menu, report, scheduled job, sequence, group, access right, record rule and reference data set | `references/README.md` |
| `reimplementation/` | Build sequence, milestones and acceptance gates, equivalence test plan, traceability rules, coverage and evidence | `reimplementation/README.md` |

Reading order for a complete rebuild: `overview` → `data` → `runtime` → `interfaces` → `domains` (in the order of the reimplementation sequence in the root `README.md`) → `references` as needed → `reimplementation`.
