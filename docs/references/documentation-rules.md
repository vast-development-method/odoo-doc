# Documentation rules

These rules are binding on every file in this repository. They exist so that the specification can be handed to an engineering team or to an autonomous agent that has never seen the system being described, and be acted on without further research.

## Rule one: no product identity

No file names the product, its vendor, its historic names, its hosted services, its assistant, its marketplace or its community. No file names a source file, a source folder, a version number or a release. No file uses the words that would let a reader identify the origin by search rather than by understanding.

The system is referred to as "the system", "the platform" or "the application". A capability package is named by its full business name, never by the short identifier a package manager would use, except inside the machine-readable catalogues where a package's technical name is a key that tooling needs.

**Why.** A specification that leans on the reader recognising a product becomes a pointer to that product instead of a description of it. A reader who cannot look the product up must still be able to build the system. That constraint is what forces the specification to be complete.

### The carve-out for contractual strings

A small number of stored values and user-visible messages contain the product name, and a compatible rebuild has to reproduce them exactly. A stored selection value is written into the database and read by integrations. A user-visible message is quoted in support procedures and asserted by tests. Changing either would break the compatibility this specification exists to preserve.

Such strings are therefore reproduced verbatim, always in code font when they are stored values and always in quotation marks when they are messages, and the document says at that point that the string is reproduced rather than authored. Three kinds qualify and nothing else does:

1. Stored selection values, sequence codes and external identifiers.
2. Verbatim user-facing messages, including error text and the labels a client displays.
3. Addresses of third-party services that the system contacts, where the address is part of the integration contract.

The specification's own prose never uses the product name, not even to explain one of these strings. Where a reader needs to know what the string means, the explanation describes its function: a stored value is "the full-reference numbering style", not a name to recognise.

**Why the carve-out is narrow.** The test is whether changing the string would change observable behaviour for some party outside the system. If it would, the string is data and is reproduced. If it would not, it is prose and is rewritten.

## Rule two: no implementation language and no code

No file contains source code in any language: no statements, no function signatures, no class declarations, no decorators, no query language, no markup fragments, no configuration file excerpts. No file names a programming language, a framework, a database product, a template engine, a web server, a package manager or a library.

Behaviour is described in four forms and no others:

- **Prose** for meaning, purpose and consequence.
- **Tables** for enumerations: fields, states, transitions, rules, values, counts.
- **Numbered procedures** for algorithms. Each step is a sentence describing what happens, with its preconditions and its failure conditions stated.
- **Formulas** in fenced blocks labelled `formula`, written as plain arithmetic over quantities named in words, using the symbols × ÷ + − =.

Diagram notation is permitted: state diagrams and entity-relationship diagrams written in the Mermaid convention render in the reading interface and carry no implementation commitment.

**Why.** Code from the system being described would make the specification a translation exercise and would tie the rebuild to the original's structure. A rebuild in a language with different idioms must be free to organise itself differently while making the same decisions.

## Rule three: no acronyms or abbreviations in prose

Every term is written in full in prose, headings, table text, labels and diagram captions. Unit of measure, not the two-letter short form. Bill of materials. Purchase order. Request for quotation. Point of sale. Customer relationship management. Human resources. Identifier. Universally unique identifier. Uniform resource locator. Value-added tax. International bank account number. Bank identifier code. Electronic data interchange. Portable Document Format. Comma-separated values. Application programming interface. Remote procedure call. Quick response code. First in first out. Average cost. Cost of goods sold. Make to order. Work in progress. Key performance indicator. One-time password. Two-factor authentication. Optical character recognition. Internet of things.

### The single exception: reproduced identifiers

Storage names, transport names, column names, association table names, route paths, stored selection values, external identifiers, sequence codes and message template keys are reproduced **exactly**, in code font, because they are contractual. A replacement that must import an existing database, honour an existing integration, serve an existing client or exchange a structured document depends on them character for character.

Every reproduced identifier carries its full name in words the first time it appears in a document. A field table always has a column for the identifier and a column for the full name. This is not a loophole for abbreviating prose: it is a precise, bounded allowance for strings that are part of the external contract.

**Why.** An abbreviation is a compression that assumes shared context. A reader rebuilding from scratch has no shared context. But an identifier that crosses a system boundary is data, not prose, and altering it would break the very compatibility the specification exists to preserve.

## Rule four: current behaviour only

The specification describes the system as it behaves now. It contains no deprecated code paths, no compatibility shims, no historical explanations of why something used to work differently, no upgrade or data-migration procedures, and no guidance on deployment, hosting, process supervision, scaling or server sizing.

A setting that a user can still choose is current behaviour even if it has existed for many years, and it is in scope. A code path that nothing can reach is not.

**Why.** The client is building a new system, not maintaining an old one. History and operations are the two largest sources of noise in a specification of this size, and both are irrelevant to the decision the rebuild has to reproduce.

## Rule five: completeness over brevity

Every state, every transition, every validation, every default, every formula, every side effect and every access rule is enumerated. Length is not a constraint; omission is a defect. Where a document would otherwise say "and so on", it lists the remaining cases instead.

## Rule six: formulas carry their arithmetic

A formula is not specified until it states:

1. Each quantity by a name in words, with its unit or its currency.
2. The order in which operations are evaluated, where order changes the result.
3. What is rounded, to what precision, by which method, and at which step.
4. At least one worked numeric example, carried to the last decimal the rule produces.

Rounding is where independent implementations diverge. A formula without its rounding rule is a source of defects, not a specification.

## Rule seven: state machines are complete

A state machine document lists every state with its stored value, its label and its meaning; every transition with its origin, its destination, the operation that triggers it, the conditions that must hold, and the records it creates or changes; and a diagram. States that can only be reached by an automatic process are listed alongside the ones a person can reach.

## Rule eight: validations quote their message

Every validation states the condition that makes it fail and the exact text the system shows when it does. Placeholders inside a message are described in words, so that a rebuild can produce the same message with its own values substituted.

**Why.** Error text is part of observable behaviour. Support procedures, user training and automated tests all key on it.

## Rule nine: ledger effects are itemised

Any event that causes an entry in the ledger is specified with, for each item of that entry: the journal used, the rule that selects the account, whether the amount is a debit or a credit, the formula for the amount, the currency handling and the rate used, the date, the counterparty, the analytic distribution, the tax treatment, and what the item is reconciled against.

**Why.** Financial consequence is the part of an enterprise system where an approximate rebuild is worthless. Two systems that produce different ledger entries from the same event are not equivalent, however similar their screens.

## Rule ten: acceptance criteria carry numbers

Every domain ends with numbered scenarios in Given, When and Then form. A scenario states concrete starting records, a concrete operation with concrete inputs, and the exact resulting records, amounts, quantities and states. Scenarios cover the ordinary path, every validation failure, every state transition, rounding edges, multiple currencies and multiple companies where they apply.

## Rule eleven: gaps are filled and marked

Where the system's behaviour in some situation is genuinely not determinable, the specification states the industry-standard resolution explicitly and marks it with the phrase **industry-standard default**. It never leaves the question open and never silently substitutes a convention for an observed decision.

Where an observed behaviour looks like a defect, it is recorded as observed, marked as a **compatibility finding**, and a note explains what a corrected behaviour would be. The rebuild then makes an informed choice rather than an accidental one.

## Rule twelve: the repository stands alone

Every cross-reference is a relative link to another file in this repository. No document says "see the original", "refer to the source", "as described upstream" or anything that sends the reader outside. A reader with only this repository has everything.

## Applying the rules to generated material

The machine-readable catalogues and the generated reference pages follow the same rules with two accommodations. First, they carry reproduced identifiers as keys, since that is their purpose. Second, their descriptive strings are taken from the system's own labels and help text, scrubbed of product identity, and are accompanied by a full name in words. A generated page states what it was generated from.

## Reviewing against the rules

A document is ready when a reader who knows the business domain but has never seen the system can answer, from the document alone: what records exist and what each field means; what states a record can be in and what moves it; what the system will refuse and what it will say when it refuses; what numbers it will produce and how they are rounded; what appears in the ledger as a result; and how to tell whether a rebuild got it right.
