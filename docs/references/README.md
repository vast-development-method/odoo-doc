# References

Generated, exhaustive reference material. Each page is produced from the machine-readable catalogues and describes one kind of artefact completely. Nothing here is hand-written, so nothing here should be hand-edited: the next regeneration would discard the change. Authored explanation lives in the domain folders, which these pages complement.

The one exception is [documentation rules](documentation-rules.md), which is authored and governs everything else.

## Entities

| Page | Content |
|---|---|
| [Entity index](entity-index.md) | Every entity with its transport name, storage name, full name, kind, defining package and counts, each linking to its page |
| [Entity reference pages](entities/) | One page per entity: identity and behaviour, every field with its type and rules, selection values, state fields, database constraints, every operation with its kind and triggers, every validation message, access rights, record rules, views, actions, menus, reports, jobs and templates |
| [Entity name dictionary](entity-name-dictionary.md) | The canonical full name of every entity |

An entity page is the fastest way to answer "what does this record hold and what can be done to it". For why it behaves that way, follow the domain that owns it, given in `schemas/traceability/entity-to-document.json`.

## Interfaces

| Page | Content |
|---|---|
| [Routes](routes.md) | Every route with its request kind, authentication, methods and purpose |
| [Views](views.md) | Every view declaration grouped by entity |
| [Actions and menus](actions-and-menus.md) | Every window action, server action and menu |
| [Printable reports](reports.md) | Every report definition with its template and attachment policy |

## Operations and configuration

| Page | Content |
|---|---|
| [Scheduled jobs](scheduled-jobs.md) | Every job with its interval and the operation it runs |
| [Sequences](sequences.md) | Every numbering sequence with its format |
| [Groups and access](groups-and-access.md) | Every group, every access right and every record rule |
| [Validation messages](validation-messages.md) | Every user-facing message with the entity and operation that raises it |

## Data and packaging

| Page | Content |
|---|---|
| [Reference data](reference-data.md) | Every shipped data set with its record count and the packages that ship it |
| [Capability packages](capability-packages.md) | Every package with its category, summary, dependencies and the entities it defines and extends |

## Rules

| Page | Content |
|---|---|
| [Documentation rules](documentation-rules.md) | The twelve rules every file in this repository obeys, each with its rationale, and how to review against them |
