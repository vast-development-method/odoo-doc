# Domains

One folder per business domain. Every folder contains the same eleven documents so that a reader always knows where to find what:

| File | Content |
|---|---|
| `README.md` | Scope, capabilities, entity list, reading order, dependencies on other domains |
| `entities.md` | Every entity in full: purpose, lifecycle, field table, relations, uniqueness, defaults, computed rules, ordering, display name, archival, multi-company behavior |
| `state-machines.md` | Every state field: states, transitions, guards, side effects, diagrams |
| `workflows.md` | End-to-end operational workflows, step by step |
| `business-rules.md` | Validations, constraints, invariants, error messages, permission checks, locking rules |
| `calculations.md` | Every formula and algorithm with rounding, precision, currency and unit conversion, worked examples |
| `accounting-effects.md` | Every journal entry produced by the domain |
| `configuration.md` | Settings, system parameters, sequences, default records, groups, access rights, record rules, scheduled jobs |
| `interfaces.md` | Menus, views, remote operations, routes, reports, email templates, external integrations, import and export |
| `acceptance-criteria.md` | Numbered Given / When / Then scenarios with concrete numbers |
| `glossary.md` | Every domain term defined |

The list of domains, grouped, is the business capability map in the root `README.md`.
