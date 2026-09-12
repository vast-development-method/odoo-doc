# Conformance profiles

Not every rebuild needs to satisfy the specification in the same way. A team replacing the system for one company with no external integrations has different obligations from a team that must keep an existing database, existing clients and existing partner connections working. This document defines four levels so that a team can state precisely what it is claiming, and so that a test plan can be scoped to the claim.

A claim is made per level and per domain. "Behavioral conformance in accounting, structural conformance in inventory" is a meaningful and honest statement. "Conformant" without qualification is not.

Whatever the level, a claim names the evidence behind it: the milestones and stage gates of [milestones](milestones.md) that closed, the layers of the [equivalence test plan](equivalence-test-plan.md) that ran, and the entries in [coverage and evidence](coverage-and-evidence.md) that record every acknowledged difference. A claim without that evidence is an opinion.

Every gate a claim rests on is binary. A gate passes exactly as written or the claim does not include it: there is no partial credit and no "passes except for rounding". Section 1.1 of [milestones](milestones.md) states that rule in full, and it applies at every level below.

## Level one: decision conformance

**Claim.** For every supported operation, the rebuild reaches the same accept-or-reject decision, and when it accepts, it produces the same business outcome in the same states, quantities and amounts.

**Obligations.**

- Every state machine has the same states and the same permitted transitions, and each transition has the same guards.
- Every validation refuses the same inputs. The refusal message may be worded differently but must identify the same condition.
- Every calculation produces the same result to the last decimal the specification's rounding rules produce. There is no amount tolerance at any level; section 19 of the equivalence test plan states the rule.
- Every event that touches the ledger produces items with the same accounts, sides and amounts, whatever the entries are keyed or numbered.
- Every access decision is the same: the same user, in the same groups, sees the same records and is refused the same operations.
- Every record the specification says exists is written, including records no user ever sees: tracking entries, follower subscriptions, partial reconciliation rows, valuation layers, analytic lines, activity records and scheduled job logs. A missing invisible record is a failure at this level and at every level above it.
- Where the specification declares an order for a relation, a result set or a printed section, that order is reproduced. Where it declares none, an ordering difference is not a failure at any level.
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


## The milestones and stage gates each level requires

A level is claimed per domain, so the step milestones a claim needs are those of the steps that deliver the claimed domains, listed in section 1.8 of the [build sequence](build-sequence.md). The cross-cutting milestones and the stage gates are claimed for the system as a whole, because they assert properties that no single domain owns. Each level requires everything the levels below it require.

| Level | Step milestones | Cross-cutting milestones | Stage gate rows it adds |
|---|---|---|---|
| One: decision conformance | Every step milestone of every step that delivers a claimed domain, including that milestone's final coverage gate at one hundred percent | `MX1`, access control is complete, and `MX3`, numbers never drift | Every row proved by layers one, three, four, five, seven and ten: the whole of stage gate two, rows 1.2, 1.6, 1.7, 1.10 and 1.11, rows 3.5 and 3.7, the whole of stage gates five, six, seven, eight, nine and ten except the rows named against the higher levels below |
| Two: record conformance | The same, read again against the stored shape of every record each gate writes | The same | Rows 1.1, 1.3, 1.4, 1.5, 1.9, 3.3, 5.1 and 7.5, all proved by layer two |
| Three: contract conformance | The same, read again against the request and response shape of every endpoint and the content of every document each gate produces | The same | Rows 3.1, 3.2, 3.4 and 3.6, proved by layer six, and row 3.8, proved by layer eleven |
| Four: operational conformance | The same, replayed under concurrency and interruption | Additionally `MX2`, concurrency is safe, and `MX4`, performance is acceptable | Rows 1.8, 4.7, 7.3, 8.6, 8.8 and 10.5, all proved by layer eight |

Layer nine, business rules, has no stage gate row of its own: it is asserted by the coverage gate of every step milestone, because a rule belongs to one domain and a stage spans several. A level one claim therefore fails if any rule of a claimed domain is uncovered, even though no stage gate names layer nine.

## How a claim is written

A claim is a short written statement, kept with the rebuild and refreshed whenever a milestone closes. It names six things and nothing may be left out:

1. **The scope.** The domains the claim covers, by their folder names under `docs/domains/`.
2. **The level.** One of the four, per domain, never a range and never "approximately".
3. **The date and the revision.** The moment the evidence was produced and the revision of the rebuild it was produced from.
4. **The evidence.** The milestones and stage gate rows that closed, the layers that ran, and the run in which they ran.
5. **The acknowledged differences.** Every entry of [coverage and evidence](coverage-and-evidence.md) that applies to the claimed domains, by its identifier.
6. **The exclusions.** Anything inside a claimed domain that the claim does not cover, stated positively rather than by silence.

A worked example of a well-formed claim:

> As of the run of 2026-03-14 on revision 4 812, the rebuild claims record conformance in general ledger, taxes and analytic accounting, and decision conformance in inventory operations and inventory valuation and costing. Evidence: milestones M0, M4, M5, M7 and M8 closed with all coverage gates at one hundred percent; cross-cutting milestones MX1 and MX3 closed; stage gate six rows 6.1 to 6.15 and stage gate seven rows 7.1 to 7.14 passed; test layers one, two, three, four, five, seven, nine and ten ran green. Acknowledged differences: three, recorded as the entries for the resequencing message wording, the second decimal of the tax report rounding label, and the ordering of unmatched statement lines. Exclusions: the country tax return structures of the general ledger claim are not covered, because no country package is loaded in the fixture.

A claim that cannot name its run, its revision and its differences is not a claim; it is a hope.

## What each level compares, and how exactly

Each level adds a comparison procedure, and the procedure is what makes the claim checkable by someone who did not run it.

| Level | What is compared | How the comparison is performed | What ends the comparison |
|---|---|---|---|
| One | Decisions and outcomes | For each acceptance scenario and golden scenario, the resulting records are read back by business key and every value the scenario states is compared as a decimal, a string or a state, never as a rendered label | The first differing value, reported with the record's business key and the field name |
| Two | Stored shape | The rebuild's own registry is exported and compared against the entity catalogs entity by entity, field by field, for storage name, type, required flag, default, relation target and cardinality, then the migration data set is loaded and every field is accounted for as mapped, defaulted or explicitly dropped | Any field the load leaves unmapped, any value it reinterprets, any selection value it re-encodes |
| Three | Contracts | Recorded exchanges are replayed against the rebuild and the responses are compared field by field, ignoring only what section 19 of the [equivalence test plan](equivalence-test-plan.md) marks as varying; every structured document is round-tripped; every report is compared against the content list of its domain's `interfaces.md` | Any field absent, renamed, retyped or reordered where an order is declared; any route that answers at a different path or with a different error kind |
| Four | Behavior under load and failure | Conflicting operations are driven in parallel from at least two workers, and operations are interrupted at the points the runtime documents name, then the recovered state is compared against the specified state | Any outcome that depends on the interleaving where the specification says it does not, any duplicated effect after a retry, any partial write after an interruption |

## Acknowledged differences

A difference between the rebuild and the specification is either acknowledged or it is a failure. Acknowledging one is a deliberate, recorded act, not a note in a message thread.

Every acknowledged difference is recorded in [coverage and evidence](coverage-and-evidence.md) with all of the following:

| Field | Content |
|---|---|
| Identifier | A stable reference the claim can cite |
| Scope | The domain and the artifact that differs: a rule identifier, a gate identifier, a scenario reference, a field, a route or a report |
| What the specification requires | Stated in the specification's own words, with a link to the owning document |
| What the rebuild does instead | Stated concretely, with the observable consequence |
| Why | The reason the difference was accepted |
| Who decided | The role that accepted it |
| Level effect | The highest level the claim can still hold in that domain with this difference present |

Four kinds of difference recur, and they do not weigh the same:

1. **A cosmetic difference.** Different wording of a refusal that identifies the same condition. Permitted at level one, because level one requires the same condition, not the same words. It blocks nothing above level one either, unless a client keys on the text, in which case it becomes a contract difference.
2. **A structural difference.** A different storage name, a different table layout, a different selection value. Permitted at level one, fatal at level two.
3. **A contract difference.** A different route path, a different response shape, a different document field. Permitted at levels one and two, fatal at level three.
4. **A behavioral difference.** A different decision, a different amount, a different state, a missing record. Fatal at every level. It is never acknowledged as acceptable; it is acknowledged only as a known defect with a date by which it is corrected, and the domain holds no level until it is.

A compatibility finding is not an acknowledged difference. A finding records that the specified behavior itself looks wrong; the rebuild still has to choose, deliberately, to reproduce it or to depart from it, and a departure is then recorded as an acknowledged difference of the fourth kind.

## Losing a level and regaining it

A claim is not permanent. It is withdrawn, for the affected domain only, when any of the following happens:

1. A gate that the level requires stops passing, for any reason, including a change elsewhere in the rebuild.
2. An acknowledged difference expires without being corrected or re-accepted.
3. The specification changes in a way that adds a rule, a state, a field, a route or a document to a claimed domain, and no test yet covers the addition.
4. A behavioral difference of the fourth kind is discovered.

Regaining the level requires the same evidence as claiming it the first time: the full suite of the level's layers, on the current revision, with every coverage gate back at one hundred percent. Evidence from an earlier revision is never carried forward, because the property being claimed is a property of the system as it now stands.

## Reading a claim as a reviewer

A reviewer verifies a claim by asking six questions in this order, stopping at the first that cannot be answered:

1. Which domains, at which level, on which revision?
2. Which run produced the evidence, and can it be re-run unchanged?
3. Did every coverage gate of the claimed domains reach one hundred percent, or is a rule uncovered?
4. Which stage gate rows are claimed, and does each one name a layer that actually ran?
5. Is every difference between the run and the specification in the acknowledged list, and is each of them of a kind the claimed level permits?
6. What is excluded, and does the exclusion list match the gaps visible in the coverage report?

## Overstatements to avoid

These five claims sound conformant and are not, and each one has cost a rebuild a re-run:

1. **"Conformant."** Without a level and a domain the word carries no obligation, and a reader supplies the strongest reading.
2. **"The tests pass."** A suite proves the level of the layers it contains. A suite of layers one, four and five proves decision conformance in the domains it touches and nothing about record, contract or operational conformance.
3. **"Equivalent apart from rounding."** There is no amount tolerance at any level. A rounding difference is a behavioral difference of the fourth kind.
4. **"The screens look the same."** No level requires a screen to look the same, and no level is earned by one that does. Screen layout is outside every claim.
5. **"We improved it."** A better behavior that differs is still a difference. It is claimable only when it is recorded as an acknowledged difference and the domain's level is reduced accordingly.
