# Conformance profiles

Not every rebuild needs to satisfy the specification in the same way. A team replacing the system for one company with no external integrations has different obligations from a team that must keep an existing database, existing clients and existing partner connections working. This document defines four levels so that a team can state precisely what it is claiming, and so that a test plan can be scoped to the claim.

A claim is made per level and per domain. "Behavioural conformance in accounting, structural conformance in inventory" is a meaningful and honest statement. "Conformant" without qualification is not.

## Level one: decision conformance

**Claim.** For every supported operation, the rebuild reaches the same accept-or-reject decision, and when it accepts, it produces the same business outcome in the same states, quantities and amounts.

**Obligations.**

- Every state machine has the same states and the same permitted transitions, and each transition has the same guards.
- Every validation refuses the same inputs. The refusal message may be worded differently but must identify the same condition.
- Every calculation produces the same result to the last decimal the specification's rounding rules produce.
- Every event that touches the ledger produces items with the same accounts, sides and amounts, whatever the entries are keyed or numbered.
- Quantity and value conservation invariants hold.

**Not required.** Identical stored identifiers, identical table and column names, identical route paths, identical selection values, identical error text, identical document numbering format, identical screen layout.

**Who should target this.** A greenfield deployment with no data to carry over and no external system to satisfy. This is the lowest level that is worth calling a rebuild, and it is the level at which the business gets the same answers.

**How it is proved.** Every worked example and every acceptance scenario in the specification passes. The ledger, valuation and reconciliation invariants hold after every scenario.

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

**How it is proved.** Level one, plus a load of a representative existing data set into the rebuild with no field left unmapped and no value reinterpreted, followed by a full replay of the level one suite against the loaded data.

## Level three: contract conformance

**Claim.** Level two, plus existing clients and existing integrations keep working without an adapter.

**Additional obligations.**

- Every route in the endpoint catalogue exists at the same path, accepts the same methods, requires the same authentication, and returns the same shape.
- Every generic entity operation accepts the same arguments, including the context, and returns the same shape, including the ordering guarantees.
- Entities are addressable by their specified transport names.
- Errors use the specified envelope and the specified error kinds.
- Report definitions are addressable by their specified names and produce documents with the same content.
- Structured documents exchanged with outside parties validate against the same rules and carry the same fields.
- Message templates and their keys exist.

**Not required.** Identical rendered markup, identical styling, identical internal ordering of code.

**Who should target this.** A team that cannot change the clients, the partner connections or the reporting tools that already talk to the system.

**How it is proved.** Level two, plus a contract suite that replays recorded exchanges against the rebuild and compares responses field by field, and a document exchange suite that round-trips every structured format.

## Level four: operational conformance

**Claim.** Level three, plus the system behaves the same under concurrency, scheduling and failure.

**Additional obligations.**

- Transaction boundaries are the same: the same set of changes commits or rolls back together.
- The isolation level and the conflict retry behaviour produce the same outcome under concurrent writes, including the same resolution when two operators act on the same record.
- Scheduled jobs run with the same triggers and the same locking, so that two workers never run one job concurrently.
- The notification fan-out delivers the same events to the same subscribers.
- Long operations batch and resume in the same way, so a restart leaves the same state.
- Access checks are enforced at the same points, so that a partially completed operation leaves the same trace.

**Not required.** Identical throughput, identical latency, identical resource use, identical process topology.

**Who should target this.** A team replacing the system in place for a business that depends on its concurrency semantics, typically one with many simultaneous operators on shared documents such as a warehouse floor or a counter estate.

**How it is proved.** Level three, plus a concurrency suite that drives conflicting operations in parallel and asserts the specified resolution, and a failure suite that interrupts operations at defined points and asserts the recovered state.

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
| General ledger | Two | Worked examples and acceptance scenarios pass; migration load complete | Audit chain not yet implemented |
| Inventory operations | One | Acceptance scenarios pass | Storage names differ; no migration attempted |

## What no level requires

No level requires the rebuild to reproduce the internal structure of the original: its layering, its extension mechanism, its module boundaries or its naming of internal operations. Two systems can be conformant at level four and share no structural resemblance. Conformance is about what is observable: the decisions, the records, the contracts and the behaviour under load.

Equally, no level permits a rebuild to drop a decision because it seems wrong. An observed behaviour that looks like a defect is recorded as a compatibility finding in the domain that owns it, and the rebuild chooses deliberately whether to reproduce it, with the consequence understood. A silent deviation is a conformance failure even when the new behaviour is better.
