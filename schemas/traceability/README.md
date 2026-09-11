# Traceability catalogues

The links that let a reader move between a capability, an entity, a document and a scenario without searching. The rules that govern them are in [traceability rules](../../docs/reimplementation/traceability-rules.md).

| Catalogue | Content | Records |
|---|---|---|
| [`entity-name-dictionary.json`](entity-name-dictionary.json) | The canonical full name of every entity, with its transport and storage names, and the expansion of every abbreviated token used in reproduced identifiers | 983 |
| [`capability-to-entity.json`](capability-to-entity.json) | For every domain: the capability packages it covers, the entities they define, and the field and operation counts | 45 |
| [`entity-to-document.json`](entity-to-document.json) | For every entity: the owning domain, the documents that specify it, and its generated reference page | 983 |
| [`coverage.json`](coverage.json) | For every domain: documents present, their sizes, the artefacts in scope and the kind of evidence that exists | 45 |

Every entity in the index is assigned to exactly one owning domain, so no part of the system is left without a document responsible for it.
