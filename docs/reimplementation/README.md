# Reimplementation

How to rebuild the system, in what order, to what standard, and how to prove you succeeded.

| Document | Content | Read it when |
|---|---|---|
| [Build sequence](build-sequence.md) | Ten dependency-ordered steps, each with its deliverable, the specification documents that define it, the decisions to settle before starting, and the gate that closes it | Before writing any code |
| [Milestones](milestones.md) | The acceptance gate for each step as a list of statements that must be demonstrably true, each mapped to a test layer | At the start of each step, to know what finished means |
| [Conformance profiles](conformance-profiles.md) | Four cumulative levels of claim, from making the same decisions to behaving the same under concurrency, with the obligations and the proof of each | Before promising anything to a stakeholder |
| [Equivalence test plan](equivalence-test-plan.md) | Eight test layers from arithmetic to concurrency, the three fixed data sets, the running order, and the four numbers to report | While building, from the first calculation onward |
| [Traceability rules](traceability-rules.md) | The five identifiers, the four maps, the linking and ownership rules, what is generated against what is authored, and the stability promises | When adding to the specification or keeping it current |
| [Coverage and evidence](coverage-and-evidence.md) | Measured counts, coverage per domain, the four kinds of evidence and what is still uncertain | Before claiming conformance, and whenever someone asks how much is really covered |

## The short version

Build the platform first and get dependency-driven recomputation, access enforcement and rounding exactly right before any business domain exists, because everything later assumes them. Then master data, then money, then goods, then commerce, then people and services. Encode every worked example and every acceptance scenario as a test while you implement it, never afterwards. Check the invariants after every test. State the conformance level you are claiming and keep the coverage report honest, because an overstated claim costs more than a modest one.
