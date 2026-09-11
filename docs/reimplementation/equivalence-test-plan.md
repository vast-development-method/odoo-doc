# Equivalence test plan

This plan turns the specification into an executable check. Its premise is that behavioural equivalence is not a judgement made at the end but a property held continuously, proved by a suite that grows alongside the rebuild.

The suite has eight layers. Each layer catches a class of divergence that the layers below it cannot see. A rebuild that passes layer three but has no layer four is not wrong; it is making a smaller claim, and [conformance profiles](conformance-profiles.md) names which claim.

## Layer one: arithmetic

**Catches.** Rounding drift, precision loss, wrong evaluation order, wrong currency at the moment of conversion.

**Source of cases.** Every worked example in every `calculations.md` file, and every record in the mathematics catalogues under `schemas/mathematics/`, whose `worked_examples` carry inputs and exact expected outputs.

**Method.** Each case is a pure function test: given the named operands, the calculation returns the stated result exactly. No database, no records, no user. These tests must run in seconds so that they can run on every change.

**Why first.** Arithmetic divergence is the most common and the most damaging failure in a financial rebuild, and it is the cheapest to catch. A tax computed a hundredth of a unit differently will not be noticed by a screen test but will fail an audit.

**Coverage target.** Every formula in the specification has at least one case, and every formula with a rounding step has at least three: one that rounds down, one that rounds up, and one that sits exactly on the boundary.

## Layer two: entity structure

**Catches.** A missing field, a wrong type, a lost default, a dropped constraint, a renamed selection value, a relation with the wrong cardinality or the wrong deletion behaviour.

**Source of cases.** `schemas/data/entity-index.json`, the per-entity documents under `schemas/data/entities/`, `schemas/data/relations.json`, `schemas/data/selection-values.json`, and for a level two claim, `schemas/data/physical-tables.json`.

**Method.** Introspect the rebuild's own registry and schema and compare it, field by field, with the catalogue. Report differences as three lists: missing, extra and divergent. Extra is not automatically a failure — a rebuild may add fields — but every entry must be acknowledged.

**Why here.** A structural gap makes every later test in that area meaningless, and it is found in seconds rather than by a failing business scenario hours later.

## Layer three: state machines

**Catches.** A missing state, a transition that should be refused and is not, a guard evaluated on the wrong condition, a side effect that fires in the wrong order or not at all.

**Source of cases.** Every `state-machines.md`.

**Method.** For each entity with a state, drive it through every transition in the table, and attempt every transition that is *not* in the table and assert refusal. After each permitted transition, assert the specified side effects: the records created, the fields changed, the messages posted and the activities scheduled.

**Note.** The refusal half matters more than the permitted half. Most rebuilds implement the transitions that users perform and omit the guards that stop users performing the wrong one.

## Layer four: business scenarios

**Catches.** Wrong interaction between rules that are individually correct.

**Source of cases.** Every numbered scenario in every `acceptance-criteria.md`, and every trace in [cross-domain transactions](../domains/cross-domain-transactions.md).

**Method.** Each scenario sets up the stated records, performs the stated operation, and asserts every stated outcome: states, quantities, amounts, ledger items, reconciliations and statuses. Scenarios run against a database seeded only with the shipped reference data, so that a scenario's preconditions are entirely explicit.

**Why cross-domain traces matter most.** A rebuild almost always gets a single-domain scenario right and a combined one wrong, because the combined one depends on ordering: whether the valuation layer is written before or after the ledger entry, whether the delivered quantity updates before the invoice status recomputes. The traces exist to pin that ordering down.

## Layer five: invariants

**Catches.** Slow drift that no single scenario reveals.

**Method.** After every test in layers three and four, assert a fixed set of properties over the whole database:

1. Every posted ledger entry balances: the sum of debits equals the sum of credits, per entry and per currency.
2. The balance of each valuation account equals the sum of the valuation layers that feed it.
3. Reconciled amounts net to zero within each reconciliation group, in both the company currency and the document currency.
4. For every product and location, the quantity on hand equals the signed sum of completed movements.
5. Reserved quantity never exceeds quantity on hand at any location, and never falls below zero.
6. For every order line, invoiced quantity never exceeds delivered quantity where the policy is delivery-based, and never exceeds ordered quantity where it is order-based, except by the documented upsell allowance.
7. Every record that declares a company is reachable only by a user whose company set contains it.
8. Every sequence has produced a strictly increasing series within each period, with gaps only where the specification permits them.

These are cheap to check and they fail loudly on exactly the errors that are hardest to attribute later.

## Layer six: contracts

**Catches.** A route that moved, an argument that changed meaning, a response field that disappeared, an error that now reports a different kind.

**Source of cases.** [Endpoint catalogue](../interfaces/endpoint-catalog.md), [remote transport contracts](../interfaces/remote-transport-contracts.md), `schemas/interfaces/routes.json`.

**Method.** Replay a recorded set of exchanges against the rebuild and compare responses field by field, ignoring only the fields the specification marks as varying. Round-trip every structured document format: encode a document, decode it, and assert the decoded record matches the original.

**Required only for a level three claim**, but valuable earlier as a regression net on anything a client already consumes.

## Layer seven: authorisation

**Catches.** A record visible to someone who should not see it, an operation permitted to a group that should be refused, a field readable through a relation that is restricted directly.

**Method.** For each group defined in `schemas/operational/groups.json`, construct a user in that group and only that group, and for each entity assert the four permissions against the access rights catalogue. Then, for each record rule in `schemas/operational/record-rules.json`, create records inside and outside the rule's filter and assert exactly which are visible. Finally, attempt to reach a restricted record indirectly, through a relation from a permitted one, and assert refusal.

**Why a separate layer.** Authorisation failures are silent. A business scenario passes whether or not the data was protected, because the scenario runs as a privileged user.

## Layer eight: concurrency and recovery

**Catches.** Lost updates, two workers running one scheduled job, a reservation granted twice, a partially applied operation surviving a failure.

**Method.** Drive conflicting operations in parallel and assert the specified resolution: two operators reserving the last unit, two closing the same session, two posting into the same numbering sequence, two reconciling the same item. Then interrupt operations at defined points and assert the recovered state contains no partial effect.

**Required only for a level four claim.**

## Test data

Three fixed data sets, versioned alongside the rebuild:

**Reference set.** The shipped reference data only: countries, currencies, languages, units, precisions, activity types, message subtypes. Every scenario starts here and creates what it needs. This keeps preconditions explicit and makes failures reproducible.

**Configured set.** The reference set plus one company with a chart of accounts, journals, taxes, a warehouse with locations and operation types, a price list, and a handful of products covering each costing method and each tracking mode. Used for cross-domain traces.

**Migration set.** For a level two or higher claim, an extract of real records with every field populated at least once, used to prove that a load leaves nothing unmapped.

No test depends on demonstration data, and no test depends on another test's leftovers.

## Comparing against a running reference

Where a running instance of the described system is available, add a differential harness: perform the same operation on both, then compare the resulting records field by field, ignoring identifiers, timestamps and anything the specification marks as varying. This is the strongest evidence available and it finds behaviours no document captured.

Where it is not available, the worked examples and the acceptance scenarios are the reference, and the coverage report must say so plainly. [Coverage and evidence](coverage-and-evidence.md) keeps those two kinds of evidence separate, because a rebuild verified only against the specification inherits every gap the specification has.

## Running order and gating

| When | Layers |
|---|---|
| On every change | One, two, five |
| On every merge | One, two, three, four, five, seven |
| Nightly | All, including six |
| Before a milestone gate | All, plus the differential harness where available |

A milestone in [milestones](milestones.md) closes only when its layers pass with no unacknowledged difference. An acknowledged difference is one recorded in the coverage report with a reason and a decision; an unacknowledged one is a failure.

## Reporting

The suite reports four numbers per domain, and they are not interchangeable:

1. **Specified** — artefacts the specification describes.
2. **Implemented** — artefacts the rebuild provides.
3. **Exercised** — artefacts at least one test touches.
4. **Verified** — artefacts whose specified outcome a test asserts exactly.

The gap between exercised and verified is where false confidence lives. A test that creates an invoice exercises fifty fields and verifies six.
