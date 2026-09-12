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

### How each identifier is written and where it may appear

| Identifier | Written in | Appears in | Never appears as |
|---|---|---|---|
| Entity transport name | Code font, always | Field tables, entity lists, reference page names, machine-readable catalogs, endpoint definitions | A heading on its own; a heading names the entity in words and gives the transport name beside it |
| Entity storage name | Code font, always | The physical table catalog, the record conformance obligations of [conformance profiles](conformance-profiles.md), migration mapping tables | Prose describing behavior; prose uses the full name |
| Domain slug | Plain text inside a link, code font elsewhere | Link targets, the step map and the domain table of the [build sequence](build-sequence.md), coverage rows | A substitute for the domain's name in words; prose names the domain in words and links the slug |
| Capability package technical name | Code font, always | The package catalog and the machine-readable scope files only | Any narrative document; a package is named by its full business name in prose |
| Rule identifier | Code font where a table lists it, plain text inside a sentence that also names the rule | A domain's `business-rules.md`, the coverage matrix, gate evidence columns, layer nine test names | A citation without the domain, unless the citing text is inside that domain's folder |
| Scenario reference | Plain text | Gate evidence columns, the acceptance-to-workflow map, failure reports | A line number; a scenario is addressed by its number, never by where it sits in the file |
| Golden scenario reference and invariant reference | Plain text, upper case, with a two-digit number | The [equivalence test plan](equivalence-test-plan.md), gate evidence columns, invariant reports | A range written with a dash, which would be ambiguous against the identifier's own hyphen; a range is written with the word "to" |
| Gate identifier | Code font | [milestones](milestones.md), the coverage matrix, test suite names, conformance claims | A reused number; a withdrawn gate keeps its identifier |

Two identifiers may share a prefix without colliding, because their remaining segments differ. The clearest case is the letters that begin the inventory operations acceptance citations and the invariant references of the test plan: the first always carries an acceptance segment, the second never does, and section 1.3 of [milestones](milestones.md) states the rule.

## The four maps

Four catalogs under [`../../schemas/traceability/`](../../schemas/traceability/) carry the links. Each is regenerated whenever documents or catalogs change, so that a broken link is detectable rather than latent.

**Capability to entity.** For every business capability in the map in the root charter of the repository, the entities that implement it, the domain that specifies it and the packages that contribute to it. Answers: *if I am building this capability, what records do I need?*

**Entity to document.** For every entity, the domain folder that owns it, the specific documents that describe its fields, states, rules and calculations, and the generated reference page. Answers: *I found this entity in the catalog; where is it explained?*

**Acceptance to workflow.** For every numbered scenario, the workflow steps it exercises, the calculations it checks and the ledger effects it asserts. Answers: *this test failed; what behavior is it protecting?* And in reverse: *I changed this calculation; which scenarios cover it?*

**Coverage.** For every specified artifact, what evidence exists for it. Its structure is defined in [coverage and evidence](coverage-and-evidence.md).

### The record shape of each map

Each map is a catalog file with a header describing what generated it and a list of records. The fields below are the ones a reader or a tool may rely on; a regeneration may add fields and never repurposes one.

| Map | File | One record per | Fields a record carries |
|---|---|---|---|
| Capability to entity | [`../../schemas/traceability/capability-to-entity.json`](../../schemas/traceability/capability-to-entity.json) | Domain | The domain slug, its documentation folder, the capability packages that contribute to it and their count, the entities it owns and their count, the number of fields across those entities, the number of operations |
| Entity to document | [`../../schemas/traceability/entity-to-document.json`](../../schemas/traceability/entity-to-document.json) | Entity | The transport name, the storage name, the full name in words, the kind of entity, the package that defines it, the owning domain, the generated reference page, the machine-readable definition, and the list of documents that describe it |
| Acceptance to workflow | [`../../schemas/traceability/acceptance-to-workflow.json`](../../schemas/traceability/acceptance-to-workflow.json) | Numbered scenario | The domain, the group the scenario belongs to, the scenario number, its reference, its title, the document that states it, the given, when and then text, the amounts it asserts, the behavior it exercises, and whether it carries concrete numbers |
| Coverage | [`../../schemas/traceability/coverage.json`](../../schemas/traceability/coverage.json) | Domain | The documents present against the documents expected, the line count per document, the entities, fields, operations and validation messages in scope, and three measures — structural inventory, narrative specification and numerical verification — with the evidence behind each |

The entity name dictionary, [`../../schemas/traceability/entity-name-dictionary.json`](../../schemas/traceability/entity-name-dictionary.json), is not one of the four maps. It is the smaller lookup the maps and the documents both read: one record per entity, carrying the transport name, the storage name, the full name in words and the kind.

### The traversals the maps exist to serve

| The question | The route through the maps |
|---|---|
| I am building this capability; what records do I need? | Capability to entity, by domain, then entity to document for each entity it names |
| I found this entity in a catalog; where is it explained? | Entity to document, by transport name, then the reference page and the listed documents |
| This test failed; what behavior is it protecting? | Acceptance to workflow, by scenario reference, then the workflow steps and the calculations it names |
| I changed this calculation; which scenarios cover it? | Acceptance to workflow in reverse, filtering on the behavior exercised |
| This rule has no test; may we ship? | The coverage matrix, by rule identifier; the status column answers, and an uncovered rule blocks |
| A gate failed; which domain owns the decision? | [milestones](milestones.md), by gate identifier, to the cited document, then entity to document for the entities it names |
| How much of this domain is really specified? | Coverage, by domain, reading the three measures separately rather than as one number |

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


## Adding to the repository without breaking traceability

Each kind of addition has one correct order of operations. Following it keeps every check above green at each step; departing from it produces a repository that is momentarily inconsistent, and a regeneration then silently records the inconsistency as fact.

**Adding an entity.** Regenerate the entity catalogs and the reference page. Decide the owning domain and add the entity to that domain's `README.md` ownership list and to its `entities.md` with a full field table. Add its full name to the entity name dictionary. Regenerate entity to document and capability to entity. Only then write the states, rules, calculations and effects that reference it.

**Adding a field.** Add the row to the owning `entities.md` field table, with identifier, full name, type, target, required, default, computed rule and its dependencies, whether it is stored, and its meaning. If it is derived, state its dependencies, because the recomputation gate of stage gate one is asserted against them. Regenerate the catalogs. If the field carries a validation, add the rule as below.

**Adding a rule.** Give it the next unused identifier of its domain, never a reused one. State the failing condition and the exact message. Add it to the rule table at the head of the domain's `business-rules.md`. Add a row to the coverage matrix with the status uncovered. Write the layer nine case. Only when the case passes does the status become covered.

**Adding a state or a transition.** Add the state to `state-machines.md` with its stored value, its label and its meaning; add the transition with origin, destination, trigger, guards in order with each guard's refusal message, and the records it creates or changes; extend the diagram. Add a scenario to `acceptance-criteria.md` for the transition and one for each guard's refusal. Extend the transition count that layer three checks against.

**Adding a formula.** Write it in the domain's `calculations.md` in a fenced block labeled as a formula, with every quantity named in words, the evaluation order, the rounding step and method, and at least one worked example carried to the last decimal. Add the matching record to the mathematics catalogs under [`../../schemas/mathematics/`](../../schemas/mathematics/), whose worked example becomes the layer one case.

**Adding a scenario.** Take the next unused number in its section. Never renumber the section: a withdrawn scenario keeps its number and is marked withdrawn. Regenerate acceptance to workflow.

**Adding a route or a document.** Add it to the endpoint catalog or the report catalog, then explain its business meaning in the owning domain's `interfaces.md` with a link. A route that no domain claims belongs to the platform foundation.

**Adding a domain folder.** Create all eleven standard documents at once; a folder with ten is a failure of check three below. Add the slug to the domain table of the [build sequence](build-sequence.md) against the step that delivers it, and to the stage the step belongs to. Regenerate capability to entity and coverage.

**Adding a gate.** Take the next unused number within its milestone. Cite a document that exists and, where the document is long, a heading within it. If the gate asserts a property across a stage rather than a delivery, it is a stage gate row and names the layer that proves it.

## Running the traceability checks

Each of the nine checks is mechanical, and each one has a characteristic failure that says what to do next.

| Check | What it reads | The failure it produces | What the failure means |
|---|---|---|---|
| 1. Links resolve | Every relative link in every document | The file and the unresolved target | Either the target has not been written yet, or a link was written against a folder name that changed |
| 2. Each entity is owned once | The entity index against entity to document | An entity with no owning domain, or with two | The domain boundary is undecided; the build sequence order decides it |
| 3. Eleven documents per folder | The forty-seven domain folders | The folder and the missing document | A folder was published before it was finished |
| 4. Scenarios resolve to behavior | Acceptance to workflow | A scenario with no workflow step and no calculation or ledger effect | The scenario asserts an outcome nothing specifies, so it cannot be a contract |
| 5. Formulas have catalog records | Every `calculations.md` against the mathematics catalogs | A formula with no record, or a record whose document link is dead | Layer one has no case for that arithmetic |
| 6. Rules appear once in the matrix | Every `business-rules.md` against the coverage matrix | A rule missing from the matrix, or an identifier used twice in one domain | A rule that is not in the matrix is a rule nothing has to cover |
| 7. Gates cite what exists | [milestones](milestones.md) against the repository | A gate citing a missing file, a missing section or a layer outside the eleven | The gate cannot be evaluated, so its milestone cannot close |
| 8. No forbidden term | Every file | The file, the line and the term | A term of rule one, two or three of the [documentation rules](../references/documentation-rules.md) escaped code font or quotation marks |
| 9. Counts agree | The counts stated in prose against the counts computed from the catalogs | The stated number and the computed one | A number in prose went stale when a catalog was regenerated |

Checks one, three, seven and eight are run on every change to a document. Checks two, five, six and nine are run after every regeneration of the catalogs. Check four is run before a milestone closes, because it is the check that decides whether the milestone's coverage gate can mean anything.

## Where a fact may be repeated and where it may not

Ownership decides where a fact is written; readability decides where it may be echoed. The line between the two is this:

- A fact may be **restated in summary** in a document that is not its owner, provided the summary links the owner and adds no detail the owner does not carry. A gate summarizing a rule is a summary; a gate adding a condition to it is a second, contradictory statement of the rule.
- A number may be **repeated** only where the repetition is itself checkable by check nine: a count in prose that a catalog computes. A number that no catalog computes lives in exactly one place.
- A message may be **reproduced** wherever a reader needs to recognize it, because the message index exists for that purpose, but the owning `business-rules.md` is the only place that states the condition under which it appears.
- A formula is **never** restated. A document that needs a result links the calculation that produces it, because two copies of a rounding rule become two rounding rules.

## File and heading names

- A domain folder holds the eleven standard file names and nothing else at the top level except optional topic files that its `README.md` lists. A topic file name is lower case with hyphens, names its topic in words, and carries no abbreviation.
- A heading is a phrase, not a sentence, and it names the subject rather than describing it. Numbered headings are used where other documents cite the number; once a heading is numbered, its number is as stable as an identifier, because gates and test names cite it.
- A heading never contains an identifier alone. It names the thing in words and puts the identifier beside it, so that a table of contents is readable without the catalogs.
- Two headings in one file never carry the same text, because a link to a heading resolves to the first match.

## Naming the rebuild's own tests

The trace only survives into the rebuild if the rebuild's test names carry the specification's identifiers. Four naming rules make a failing test self-locating, and they are the reason the identifiers above are stable:

1. A layer one case is named for the formula it checks and the mathematics catalog record it came from.
2. A layer three case is named for the entity, the origin state and the trigger, so that a refusal case and its permitted counterpart sort together.
3. A layer nine case is named for the rule, by identifier where the domain numbers its rules and by document heading and rule name where it names them, exactly as the layer nine section of the [equivalence test plan](equivalence-test-plan.md) requires.
4. A layer four case is named for the scenario reference or the golden scenario reference it implements, never for the screen it drives.

A failure report then carries the identifier without anyone having to look it up, which is what section 20.4 of the [equivalence test plan](equivalence-test-plan.md) requires of it, and the coverage matrix can be filled from the test names alone.
