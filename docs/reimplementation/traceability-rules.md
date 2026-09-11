# Traceability rules

Traceability is what lets a reader move between a business capability, the entity that carries it, the document that specifies it, the catalogue entry that encodes it and the scenario that proves it, without searching. These rules define the links and keep them stable.

## The five identifiers

Everything in this repository is addressed by one of five identifiers. They never change once published.

| Identifier | Form | Example of the form | Where it is authoritative |
|---|---|---|---|
| Entity transport name | Lower-case words joined by full stops | the dotted name of a business object | `schemas/data/entity-index.json` |
| Entity storage name | Lower-case words joined by underscores | the table name | `schemas/data/physical-tables.json` |
| Domain slug | Lower-case words joined by hyphens | the folder name under `docs/domains/` | The folder itself |
| Capability package technical name | Lower-case words joined by underscores | the package key | `schemas/source-artifacts/packages.json` |
| Scenario reference | The domain slug, then a full stop, then the scenario number | a numbered case in a domain's acceptance criteria | `docs/domains/<slug>/acceptance-criteria.md` |

An entity's full business name, in words, is not an identifier: it is a label, and it may be improved. The dictionary that maps every entity to its full name is [the entity name dictionary](../references/entity-name-dictionary.md), with the machine-readable form in `schemas/traceability/entity-name-dictionary.json`.

## The four maps

Four catalogues under `schemas/traceability/` carry the links. Each is regenerated whenever documents or catalogues change, so that a broken link is detectable rather than latent.

**Capability to entity.** For every business capability in the map in the root charter, the entities that implement it, the domain that specifies it and the packages that contribute to it. Answers: *if I am building this capability, what records do I need?*

**Entity to document.** For every entity, the domain folder that owns it, the specific documents that describe its fields, states, rules and calculations, and the generated reference page. Answers: *I found this entity in the catalogue; where is it explained?*

**Acceptance to workflow.** For every numbered scenario, the workflow steps it exercises, the calculations it checks and the ledger effects it asserts. Answers: *this test failed; what behaviour is it protecting?* And in reverse: *I changed this calculation; which scenarios cover it?*

**Coverage.** For every specified artefact, what evidence exists for it. Its structure is defined in [coverage and evidence](coverage-and-evidence.md).

## Link rules in documents

1. Every cross-reference is a relative link. A link from one domain to another goes up and across, for example from a sales document to `../taxes/calculations.md`. A link from a domain to a catalogue goes up to the repository root and down, for example `../../../schemas/data/entities/`.
2. A link points at a file, and where the file is long, at a heading within it. It never points at a line number, because line numbers move.
3. Prose names the destination in words as well as linking it, so that a reader of a printed copy can still find it.
4. No link leaves the repository.

## Ownership rules

Every fact has exactly one owning document. Other documents link to it rather than restating it.

- A field is owned by the `entities.md` of its domain.
- A state and its transitions are owned by `state-machines.md`.
- A formula is owned by `calculations.md` and mirrored, in structured form, in the mathematics catalogue.
- A ledger effect is owned by `accounting-effects.md` of the domain that causes the event, even when the accounts belong to the general ledger. The general ledger owns the mechanics of posting; the causing domain owns what is posted.
- A route is owned by the endpoint catalogue; a domain's `interfaces.md` links to it and explains the business meaning.
- A shared calculation used by several domains is owned by the domain that defines it and referenced by the others. Tax computation is owned by taxes; payment term distribution is owned by receivables; unit conversion is owned by units of measure.

When two domains would each reasonably own a fact, the one earlier in the build sequence owns it.

## What must be regenerated together

Certain artefacts are derived and must never be edited by hand, because the next regeneration would discard the edit:

- Every file under `docs/references/entities/` and the reference index pages.
- Every catalogue under `schemas/data/`, `schemas/interfaces/`, `schemas/operational/` and `schemas/source-artifacts/`.
- The traceability maps.
- The counts in the coverage report and in the charter.

Hand-written material — the domain documents, the overview, runtime, data and interface documents, the mathematics catalogue and this folder — is authored and reviewed, never generated.

A change to the described system therefore means: regenerate the derived catalogues, then revise the authored documents that the regeneration contradicts, then regenerate the traceability maps and the counts.

## Stability promises

- An entity's transport name and storage name will not change. If the system renames one, the specification records both, marks the former as superseded and keeps the link working.
- A domain slug will not change. A domain that splits keeps its slug for the larger part and gains a new sibling.
- A scenario number will not be reused. A withdrawn scenario keeps its number and is marked withdrawn, so that a failing test in a rebuild can always be traced to what it once asserted.
- A catalogue's file name and its record shape will not change incompatibly. A new field may be added; an existing field will not be repurposed.

## Checking traceability

The repository is traceable when all of the following hold, and each is mechanically checkable:

1. Every relative link resolves to a file that exists.
2. Every entity in the entity index appears in exactly one domain's entity-to-document entry.
3. Every domain folder contains all eleven documents.
4. Every numbered scenario resolves to at least one workflow step and one calculation or ledger effect.
5. Every formula in a `calculations.md` file has a counterpart record in the mathematics catalogue, and every mathematics record links back to a document that exists.
6. No file contains a forbidden term under the [documentation rules](../references/documentation-rules.md).
7. The counts stated in the charter equal the counts computed from the catalogues.
