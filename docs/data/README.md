# Data

The persistent data model of the whole system.

| Document | Content |
|---|---|
| `domain-model.md` | The domain model across all domains: aggregates, entity clusters, the master data backbone (partners, companies, products, units, currencies, accounts, journals), the transactional backbone (orders, transfers, moves, journal entries, payments), and the cross-domain relations |
| `persistence-identity-and-values.md` | Identity (numeric identifiers, external identifiers, universally unique identifiers, references), value semantics (precision, rounding, currency, dates and times, time zones, binary, structured documents, properties), audit fields, soft deletion, sequences and numbering, uniqueness |
| `physical-data-catalog.md` | Every table with its columns, types, nullability, defaults, indexes, foreign keys, association tables and check constraints; generated from the live schema |
| `data-loading-and-exchange.md` | Loading of reference data with external identifiers, import and export formats, field path notation, relational import, update semantics, and the exchange contracts used by integrations |
| `reference-data.md` | The reference data sets shipped with the system: countries, states, currencies, languages, units of measure, decimal precisions, tax tags, payment methods, and the country chart templates |
