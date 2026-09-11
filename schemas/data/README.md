# Data catalogues

Everything about what is stored. See [the catalogue index](../README.md) for the record shape and the meaning of declared against observed.

## Declared: parsed from every capability package definition

| Catalogue | Content | Records |
|---|---|---|
| [`entity-index.json`](entity-index.json) | Every entity with its transport name, storage name, full name, kind, defining package and counts | 983 |
| [`entities/`](entities/) | One document per entity: every field with its type and rules, constraints, operations, validation messages, states, access rights, record rules, views, actions, menus, reports, jobs and templates | 983 files |
| [`relations.json`](relations.json) | Every relational field: source, target, cardinality, inverse, association table and deletion behaviour | 3,985 |
| [`selection-values.json`](selection-values.json) | Every selection field with its stored values and labels | 853 |
| [`computed-fields.json`](computed-fields.json) | Every computed field with its declared dependencies and whether it is stored | 3,979 |
| [`constraints.json`](constraints.json) | Every declared constraint and index with its definition and message | 390 |

## Observed: introspected from an installation carrying every capability package

| Catalogue | Content | Records |
|---|---|---|
| [`physical-tables.json`](physical-tables.json) | Every table with its columns, storage types, nullability, defaults, precision, indexes and constraints | 1,240 |
| [`resolved-entities.json`](resolved-entities.json) | The runtime registry: every entity with every field as it exists after all extensions are applied | 982 |
| [`indexes.json`](indexes.json) | Every index with its definition and uniqueness | 2,933 |
| [`foreign-keys.json`](foreign-keys.json) | Every foreign key with its referential action | 4,575 |
| [`table-constraints.json`](table-constraints.json) | Every unique and check constraint of the live schema | 296 |
| [`association-tables.json`](association-tables.json) | Tables that implement a many-to-many association | 422 |
| [`database-sequences.json`](database-sequences.json) | Every sequence backing an identifier column | 825 |

## Shipped reference data

[`reference-data/`](reference-data/) holds one catalogue per shipped data set, indexed by [`reference-data/index.json`](reference-data/index.json) (155 sets). It includes countries, states, country groups, currencies, languages, units of measure, decimal precisions, activity types, message subtypes, party titles and industries, banks, payment methods, and the report definitions shipped with the accounting engine. Country chart templates are under [`reference-data/chart-templates/`](reference-data/chart-templates/), one folder per country package.

These are the records a fresh installation must contain before any business data exists. A rebuild loads them first.
