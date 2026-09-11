# Mathematics catalogues

Every calculation in the specification that carries money, quantity, time or a rate, in a structured form that a rebuild can test against. One catalogue per business domain, named after the domain folder under `docs/domains/`, plus `platform.json` for the calculations of the platform documents (identifiers, rounding of values, date arithmetic, sequences).

Each catalogue is a document with a `catalog` header (name, description, domain, record count, the document it was authored from) and a `records` array. Every record mirrors one formula or algorithm of the domain's `calculations.md` and links back to it. The record structure is fixed by [`calculation.schema.json`](calculation.schema.json):

| Key | Content |
|---|---|
| `identifier` | Stable identifier: the domain key, a hyphen, and a three-digit sequence number |
| `name` | The calculation's name in words |
| `document` | Relative path of the `calculations.md` file, with the heading anchor of the section that states the calculation |
| `purpose` | One or two sentences on what the result is used for |
| `operands` | Every input quantity: name in words, unit or currency, where it comes from, and the constraints it must satisfy |
| `preconditions` | The conditions that must hold before the calculation is meaningful, each with the refusal or fallback that applies when it does not |
| `procedure` | The ordered steps, one sentence each, including the order of evaluation where it changes the result |
| `formula` | The arithmetic in words and symbols (× ÷ + − =), exactly as the domain document states it |
| `rounding` | What is rounded, to which precision, by which method (half away from zero, half to even, up, down), at which step |
| `result` | The output quantity: name, unit or currency, and where it is stored or displayed |
| `edge_cases` | Zero, negative, missing, overflow, currency and unit mismatches, and the outcome in each case |
| `worked_examples` | At least one example with named inputs and exact expected outputs, carried to the last decimal the rounding rule produces |
| `related_rules` | Identifiers of the business rules that constrain the calculation |

The catalogues are authored from the domain documents and reviewed against them; the domain document is the owner of the calculation and the catalogue is its structured mirror, as [the traceability rules](../../docs/reimplementation/traceability-rules.md) require. [The equivalence test plan](../../docs/reimplementation/equivalence-test-plan.md) uses the worked examples as test vectors.

| Catalogue | Domain document |
|---|---|
