# Machine-readable catalogues

Structured facts for tooling, code generation and test generation. Every catalogue is a document with a `catalog` header carrying its name, description, record count and what it was generated from, followed by a `records` array. Keys are lower snake case. Identifiers in the records are reproduced storage and transport names, and each record also carries a full name in words where one exists.

Three kinds of catalogue are mixed here and they carry different weight. A **declared** catalogue comes from parsing every capability package definition. An **observed** catalogue comes from installing every package and introspecting the running result. An **authored** catalogue is written and reviewed by hand. [Coverage and evidence](../docs/reimplementation/coverage-and-evidence.md) explains why the distinction matters before you rely on an entry.

| Folder | Content |
|---|---|
| [Data](data/README.md) | Entities, fields, relations, selection values, constraints, computed fields, the live physical schema and the shipped reference data |
| [Interfaces](interfaces/README.md) | Routes, actions, menus, views, report definitions and templates |
| [Operational](operational/README.md) | Scheduled jobs, sequences, groups, access rights, record rules, parameters and precisions |
| [Mathematics](mathematics/README.md) | Every calculation that carries money, quantity, time or a rate, with operands, procedure, rounding and worked examples |
| [Source artifacts](source-artifacts/README.md) | Capability packages, their dependency graph and the country chart templates |
| [Source file distribution](source-file-distribution/README.md) | Every file of this repository with its purpose, domain, size and line count |
| [Traceability](traceability/README.md) | The entity name dictionary and the maps that link capabilities, entities, documents and scenarios |

