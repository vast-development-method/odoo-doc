# Traceability rules

Traceability is what lets a reader move between a business capability, the entity that carries it, the document that specifies it, the catalog entry that encodes it, the rule that constrains it and the scenario that proves it, without searching. These rules define the links and keep them stable.

## The eight identifiers

Everything in this repository is addressed by one of eight identifiers. They never change once published.

| Identifier | Form | Example of the form | Where it is authoritative |
|---|---|---|---|
| Entity transport name | Lower-case words joined by full stops | the dotted name of a business object | [`../../schemas/data/entity-index.json`](../../schemas/data/entity-index.json) |
| Entity storage name | Lower-case words joined by underscores | the table name | [`../../schemas/data/physical-tables.json`](../../schemas/data/physical-tables.json) |
| Domain slug | Lower-case words joined by hyphens | the folder name under `docs/domains/` | The folder itself; the forty-seven of them are listed in section 1.8 of the [build sequence](build-sequence.md) |
| Capability package technical name | Lower-case words joined by underscores | the package key | [`../../schemas/source-artifacts/packages.json`](../../schemas/source-artifacts/packages.json) |
| Rule identifier | A short domain prefix, a hyphen, then a sequence number | a numbered validation, constraint or invariant of one domain | The `business-rules.md` of that domain |
| Scenario reference | The domain slug, then a full stop, then the scenario number | a numbered case in a domain's acceptance criteria | `docs/domains/<slug>/acceptance-criteria.md` |
| Golden scenario reference and invariant reference | The letters of the kind, a hyphen, then a two-digit number | an end-to-end scenario, or a property asserted over the whole fixture | The [equivalence test plan](equivalence-test-plan.md), sections 5 to 17 and section 18 |
| Gate identifier | The word gate, the milestone, then a two-digit number; a stage gate row is the stage number, a full stop and its row number | a binary check that closes a step or a stage | [milestones](milestones.md) |

An entity's full business name, in words, is not an identifier: it is a label, and it may be improved. The dictionary that maps every entity to its full name is [the entity name dictionary](../references/entity-name-dictionary.md), with the machine-readable form in [`../../schemas/traceability/entity-name-dictionary.json`](../../schemas/traceability/entity-name-dictionary.json).

## The four maps

Four catalogs under [`../../schemas/traceability/`](../../schemas/traceability/) carry the links. Each is regenerated whenever documents or catalogs change, so that a broken link is detectable rather than latent.

**Capability to entity.** For every business capability in the map in the root charter of the repository, the entities that implement it, the domain that specifies it and the packages that contribute to it. Answers: *if I am building this capability, what records do I need?*

**Entity to document.** For every entity, the domain folder that owns it, the specific documents that describe its fields, states, rules and calculations, and the generated reference page. Answers: *I found this entity in the catalog; where is it explained?*

**Acceptance to workflow.** For every numbered scenario, the workflow steps it exercises, the calculations it checks and the ledger effects it asserts. Answers: *this test failed; what behavior is it protecting?* And in reverse: *I changed this calculation; which scenarios cover it?*

**Coverage.** For every specified artifact, what evidence exists for it. Its structure is defined in [coverage and evidence](coverage-and-evidence.md).

## The coverage matrix of rules

The four maps answer where a fact is written. A fifth record answers whether it is proved, and it is the one a rebuild keeps for itself rather than for the specification: the coverage matrix. It has one row per rule identifier, and the columns below.

| Column | Content |
|---|---|
| Rule identifier | The numbered rule, exactly as its domain states it |
| Domain | The folder that owns the rule |
| Statement | The document and heading that states it in full |
| Tests | The layer nine cases that assert it, and any golden scenario that exercises it |
| Gates | The gate identifiers that cite it |
| Status | Covered, uncovered, or covered with an acknowledged difference recorded in the coverage report |

Two rules govern the matrix. First, an operation is not started until the rule identifiers it must enforce are listed in the matrix, so that the work is scoped by the rules and not by the screens. Second, an uncovered rule is a build blocker, not a warning: a milestone whose domains still hold uncovered rules has not closed, because rule five of [milestones](milestones.md) makes coverage itself a gate.

## Link rules in documents

1. Every cross-reference is a relative link. A link from one domain to another goes up and across, for example from a sales document to `../taxes/calculations.md`. A link from a domain to a catalog goes up to the repository root and down, for example `../../../schemas/data/entities/`; from a document of this folder the same catalog is `../../schemas/data/entities/`.
2. A link points at a file, and where the file is long, at a heading within it. It never points at a line number, because line numbers move.
3. Prose names the destination in words as well as linking it, so that a reader of a printed copy can still find it.
4. A link into a domain folder points at the folder itself or at one of its eleven standard documents, unless the optional topic file it names is itself part of that folder's published list, so that a link survives a folder redistributing its optional files.
5. No link leaves the repository.

## Ownership rules

Every fact has exactly one owning document. Other documents link to it rather than restating it.

- A field is owned by the `entities.md` of its domain.
- A state and its transitions are owned by `state-machines.md`.
- A formula is owned by `calculations.md` and mirrored, in structured form, in the mathematics catalogs under [`../../schemas/mathematics/`](../../schemas/mathematics/).
- A validation and its exact message are owned by `business-rules.md`, and the message index in [validation messages](../references/validation-messages.md) reproduces it for search.
- A ledger effect is owned by `accounting-effects.md` of the domain that causes the event, even when the accounts belong to the general ledger. The general ledger owns the mechanics of posting; the causing domain owns what is posted.
- A route is owned by the endpoint catalog; a domain's `interfaces.md` links to it and explains the business meaning.
- A shipped record, a group, a sequence, a scheduled job and a message template are owned by the `configuration.md` of the domain that ships them, and are counted in the seed data tables of the [build sequence](build-sequence.md).
- A shared calculation used by several domains is owned by the domain that defines it and referenced by the others. Tax computation is owned by taxes; payment term distribution is owned by receivables; unit conversion is owned by units of measure; working-time interval arithmetic is owned by attendances and working time.
- A test fixture is owned by section 3 of the [equivalence test plan](equivalence-test-plan.md); a step extends it and never redefines it.

When two domains would each reasonably own a fact, the one earlier in the build sequence owns it.

## What must be regenerated together

Certain artifacts are derived and must never be edited by hand, because the next regeneration would discard the edit:

- Every file under `docs/references/entities/` and the reference index pages.
- Every catalog under `schemas/data/`, `schemas/interfaces/`, `schemas/operational/` and `schemas/source-artifacts/`.
- The traceability maps.
- The counts in the coverage report and in the charter.

Hand-written material — the domain documents, the overview, runtime, data and interface documents, the mathematics catalogs and this folder — is authored and reviewed, never generated.

A change to the described system therefore means: regenerate the derived catalogs, then revise the authored documents that the regeneration contradicts, then regenerate the traceability maps and the counts.

## Stability promises

- An entity's transport name and storage name will not change. If the system renames one, the specification records both, marks the former as superseded and keeps the link working.
- A domain slug will not change. A domain that splits keeps its slug for the larger part and gains a new sibling.
- A scenario number will not be reused. A withdrawn scenario keeps its number and is marked withdrawn, so that a failing test in a rebuild can always be traced to what it once asserted.
- A rule identifier will not be reused either, and for the same reason: a gate, a test and a coverage row all cite it.
- A gate identifier is stable once published, including after its milestone has closed, because the coverage matrix and the test suite cite it.
- A catalog's file name and its record shape will not change incompatibly. A new field may be added; an existing field will not be repurposed.

## Checking traceability

The repository is traceable when all of the following hold, and each is mechanically checkable:

1. Every relative link resolves to a file that exists.
2. Every entity in the entity index appears in exactly one domain's entity-to-document entry.
3. Every one of the forty-seven domain folders contains all eleven documents.
4. Every numbered scenario resolves to at least one workflow step and one calculation or ledger effect.
5. Every formula in a `calculations.md` file has a counterpart record in the mathematics catalogs, and every mathematics record links back to a document that exists.
6. Every numbered rule appears exactly once in the coverage matrix, and no rule identifier is used twice within one domain.
7. Every gate cites a document that exists, and every layer a stage gate names is one of the eleven layers of the equivalence test plan.
8. No file contains a forbidden term under the [documentation rules](../references/documentation-rules.md).
9. The counts stated in the charter equal the counts computed from the catalogs.

## Reconciliation notes

1. **Only one of the two versions carried this document**, but the other cited it for two things it did not contain: a coverage matrix of rule identifiers, and the rule that an uncovered rule blocks a build. Both are now stated here, in the coverage matrix section.
2. **Three identifier kinds were added** to the five the surviving version listed: the rule identifier, the golden scenario and invariant references, and the gate identifier. All three are cited by the merged [milestones](milestones.md) and [equivalence test plan](equivalence-test-plan.md), so they have to be governed here.
3. **A link rule was added** for links into a domain folder, because the merged documents cite domain material by topic while a folder is free to distribute its optional topic files as it sees fit.
