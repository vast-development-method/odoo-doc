# Conformance profiles

Not every rebuild needs to satisfy the specification in the same way. A team replacing the system for one company with no external integrations has different obligations from a team that must keep an existing database, existing clients and existing partner connections working. This document defines four levels so that a team can state precisely what it is claiming, and so that a test plan can be scoped to the claim.

A claim is made per level and per domain. "Behavioral conformance in accounting, structural conformance in inventory" is a meaningful and honest statement. "Conformant" without qualification is not.

Whatever the level, a claim names the evidence behind it: the milestones and stage gates of [milestones](milestones.md) that closed, the layers of the [equivalence test plan](equivalence-test-plan.md) that ran, and the entries in [coverage and evidence](coverage-and-evidence.md) that record every acknowledged difference. A claim without that evidence is an opinion.

## Level one: decision conformance

**Claim.** For every supported operation, the rebuild reaches the same accept-or-reject decision, and when it accepts, it produces the same business outcome in the same states, quantities and amounts.

**Obligations.**

- Every state machine has the same states and the same permitted transitions, and each transition has the same guards.
- Every validation refuses the same inputs. The refusal message may be worded differently but must identify the same condition.
- Every calculation produces the same result to the last decimal the specification's rounding rules produce. There is no amount tolerance at any level; section 19 of the equivalence test plan states the rule.
- Every event that touches the ledger produces items with the same accounts, sides and amounts, whatever the entries are keyed or numbered.
- Every access decision is the same: the same user, in the same groups, sees the same records and is refused the same operations.
- Quantity and value conservation invariants hold.

**Not required.** Identical stored identifiers, identical table and column names, identical route paths, identical selection values, identical error text, identical document numbering format, identical screen layout.

**Who should target this.** A greenfield deployment with no data to carry over and no external system to satisfy. This is the lowest level that is worth calling a rebuild, and it is the level at which the business gets the same answers.

**How it is proved.** Every worked example, every numbered rule, every state transition, every accounting event and every acceptance scenario of the specification passes, together with the forty-seven golden scenarios. The ledger, valuation, reconciliation and quantity invariants hold after every scenario.

## Level two: record conformance

**Claim.** Level one, plus the persisted records have the same shape, so that data can be moved between the two systems without interpretation.

**Additional obligations.**

- Every entity in scope exists with its specified storage name.
- Every field exists with its specified storage name, its type mapped faithfully, and its meaning unchanged.
- Stored selection values are the specified strings, not a local re-encoding.
- Relations are stored with the specified cardinality, and many-to-many relations use association tables with the specified column names.
- Numeric precision is at least what the specification requires, and rounding happens at the same points.
- Audit fields and the archive flag exist and behave as specified.
- External identifiers exist and are stable, so that shipped and configured data can be reloaded.

**Not required.** Identical indexes, identical constraint names, identical physical table layout, identical route paths.

**Who should target this.** A team migrating an existing database, or one that must run the two systems side by side during a transition and reconcile them record by record.

**How it is proved.** Level one, plus a structural comparison of the rebuild's own registry against the entity catalogs, plus a load of the migration data set into the rebuild with no field left unmapped and no value reinterpreted, followed by a full replay of the level one suite against the loaded data.

## Level three: contract conformance

**Claim.** Level two, plus existing clients and existing integrations keep working without an adapter.

**Additional obligations.**

- Every route in the endpoint catalog exists at the same path, accepts the same methods, requires the same authentication, and returns the same shape.
- Every generic entity operation accepts the same arguments, including the context, and returns the same shape, including the ordering guarantees.
- Entities are addressable by their specified transport names.
- Errors use the specified envelope and the specified error kinds.
- Report definitions are addressable by their specified names and produce documents with the same content, grouping and totals.
- Structured documents exchanged with outside parties validate against the same rules and carry the same fields.
- Message templates and their keys exist.

**Not required.** Identical rendered markup, identical styling, identical internal ordering of code.

**Who should target this.** A team that cannot change the clients, the partner connections or the reporting tools that already talk to the system.

**How it is proved.** Level two, plus a contract suite that replays recorded exchanges against the rebuild and compares responses field by field, a document exchange suite that round-trips every structured format, and a report suite that compares every printed and exported document field by field against its content list.

## Level four: operational conformance

**Claim.** Level three, plus the system behaves the same under concurrency, scheduling and failure.

**Additional obligations.**

- Transaction boundaries are the same: the same set of changes commits or rolls back together.
- The isolation level and the conflict retry behavior produce the same outcome under concurrent writes, including the same resolution when two operators act on the same record.
- Scheduled jobs run with the same triggers and the same locking, so that two workers never run one job concurrently, and a job run twice within its interval has its effect once unless it is specified as repeating.
- The notification fan-out delivers the same events to the same subscribers.
- Long operations batch and resume in the same way, so a restart leaves the same state.
- Access checks are enforced at the same points, so that a partially completed operation leaves the same trace.

**Not required.** Identical throughput, identical latency, identical resource use, identical process topology.

**Who should target this.** A team replacing the system in place for a business that depends on its concurrency semantics, typically one with many simultaneous operators on shared documents such as a warehouse floor or a counter estate.

**How it is proved.** Level three, plus a concurrency suite that drives conflicting operations in parallel and asserts the specified resolution, and a failure suite that interrupts operations at defined points and asserts the recovered state.

## Which test layers each level requires

The layers are those of the [equivalence test plan](equivalence-test-plan.md). A level requires every layer of the levels below it as well.

| Level | Layers it adds | What a reviewer asks for |
|---|---|---|
| One: decision conformance | One arithmetic, three state machines, four business scenarios, five invariants, seven authorization, nine business rules, ten accounting consequences | The worked examples, the golden scenarios and the invariant report |
| Two: record conformance | Two entity structure | The structural comparison against the entity catalogs, and the migration load report |
| Three: contract conformance | Six contracts, eleven report content | The recorded-exchange comparison, the document round-trip report and the rendered documents |
| Four: operational conformance | Eight concurrency and recovery | The parallel-operation report and the interrupted-operation report |

## Choosing a level

| Situation | Level |
|---|---|
| New business, no existing data, no integrations | One |
| Existing data to migrate, integrations can be rewritten | Two |
| Existing data, and clients or partners that cannot change | Three |
| Replacing a busy production system in place | Four |

Levels are cumulative. A higher level always includes the obligations of every lower one.

## Declaring partial conformance

A rebuild delivered in stages will hold different levels in different domains for a long time. Record the claim per domain in a table like the one below and keep it current, because an overstated claim is worse than a low one.

| Domain | Level claimed | Evidence | Known gaps |
|---|---|---|---|
| [general-ledger](../domains/general-ledger/) | Two | Worked examples, rules, accounting events and acceptance scenarios pass; milestone M4 closed; migration load complete | Audit chain not yet implemented |
| [inventory-operations](../domains/inventory-operations/) | One | Acceptance scenarios and golden scenarios pass; milestone M7 closed | Storage names differ; no migration attempted |

The same shape of table is kept, for the specification's own coverage rather than for the rebuild's, in [coverage and evidence](coverage-and-evidence.md); the two are read together.

## What no level requires

No level requires the rebuild to reproduce the internal structure of the original: its layering, its extension mechanism, its package boundaries or its naming of internal operations. Two systems can be conformant at level four and share no structural resemblance. Conformance is about what is observable: the decisions, the records, the contracts and the behavior under load.

Equally, no level permits a rebuild to drop a decision because it seems wrong. An observed behavior that looks like a defect is recorded as a compatibility finding in the domain that owns it, and the rebuild chooses deliberately whether to reproduce it, with the consequence understood. A silent deviation is a conformance failure even when the new behavior is better.

## Reconciliation notes

1. **Only one of the two versions carried this document.** The other stated its position on the same topics inside its test plan and its gates: that a difference in any amount is a failure, that a missing record is a failure even when no user ever sees it, that an ordering difference matters only where an order is declared, and that a gate is binary. Those statements are folded into the levels above and into the layer table, and are stated in full in sections 1 and 19 of the [equivalence test plan](equivalence-test-plan.md) and in section 1 of [milestones](milestones.md).
2. **The layer table is new.** It exists because the merged test plan has eleven layers rather than eight, and a claim has to say which of them ran.
3. **Access decisions were added to the obligations of level one.** Both versions treat a refusal as observable behavior, and one of them tests access as a matrix from its first milestone, so an access decision belongs to the lowest level rather than to a higher one.
