# Reimplementation

How to rebuild the system, in what order, to what standard, and how to prove you succeeded.

| Document | Content | Read it when |
|---|---|---|
| [`build-sequence.md`](build-sequence.md) | Twenty dependency-ordered steps grouped into ten stages, each step with its purpose, prerequisites, entities, operations, reports, scheduled jobs, interfaces, reading order, decisions to settle first and gate; the step and stage maps; which of the forty-seven domain folders each step delivers; what may be built in parallel and what may never be; the entry conditions, the seed data, the access groups and the numeric precisions of each step | Before writing any code, and again at the start of every step |
| [`milestones.md`](milestones.md) | The gate that closes each step, as binary checks citing the acceptance criteria that specify them: twenty-one step milestones, four cross-cutting milestones for access, concurrency, numeric stability and bounded work, and ten stage gates whose rows each name the test layer that proves them | At the start of each step, to know what finished means, and at its end, to decide whether it is finished |
| [`conformance-profiles.md`](conformance-profiles.md) | Four cumulative levels of claim, from making the same decisions to behaving the same under concurrency, with the obligations, the proof and the test layers of each, and how to declare a different level per domain | Before promising anything to a stakeholder |
| [`equivalence-test-plan.md`](equivalence-test-plan.md) | Eleven test layers from arithmetic to report content, the three fixed data sets, the reference fixture in full, forty-seven end-to-end golden scenarios with their exact records and amounts, twenty-one invariants asserted continuously, the tolerance rules, the running order and the four numbers to report | While building, from the first calculation onward |
| [`traceability-rules.md`](traceability-rules.md) | The eight identifiers, the four maps, the coverage matrix of rules, the linking and ownership rules, what is generated against what is authored, the stability promises and the nine traceability checks | When adding to the specification or keeping it current |
| [`coverage-and-evidence.md`](coverage-and-evidence.md) | Measured counts, coverage per domain, the four kinds of evidence and what is still uncertain | Before claiming conformance, and whenever someone asks how much is really covered |

## The short version

Build the platform first and get dependency-driven recomputation, access enforcement and rounding exactly right before any business domain exists, because everything later assumes them. Then the shared records and the catalog, then money, then goods, then commerce, then people and services, then the remaining capabilities. Encode every worked example, every numbered rule and every acceptance scenario as a test while you implement it, never afterwards. Check the invariants after every test. State the conformance level you are claiming, per domain, and keep the coverage report honest, because an overstated claim costs more than a modest one.
