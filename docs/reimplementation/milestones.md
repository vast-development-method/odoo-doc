# Milestones and acceptance gates

Each milestone closes one step of the [build sequence](build-sequence.md). A gate is a list of statements that must be demonstrably true, each provable by a test in the [equivalence test plan](equivalence-test-plan.md). A gate is not a review meeting: every item is either passing or it is not.

A milestone may close with acknowledged differences. An acknowledged difference is recorded in [coverage and evidence](coverage-and-evidence.md) with what differs, why, and who decided. An unrecorded difference is a failure of the gate, even if someone knows about it.

## Milestone one: the platform holds

Closes step one, the platform foundation.

| # | The gate | Proved by |
|---|---|---|
| 1.1 | An entity can be defined with stored, computed, related, company-dependent and translatable fields, and all of them read back correctly. | Layer two |
| 1.2 | Writing a stored field causes every computed field that depends on it, directly or through a relation, to return the new value on the next read, without an explicit instruction to recalculate. | Layer one |
| 1.3 | A computed field that is stored is written to storage with the same value a non-stored one would return. | Layer two |
| 1.4 | A second package extends an existing entity with a new field and overrides an operation, and the override can invoke the previous definition. | Layer two |
| 1.5 | Prototype copying and delegation embedding both work, and an embedded parent's fields are readable and writable through the child. | Layer two |
| 1.6 | The filter grammar returns the specified records for every operator, including traversal through a relation and through a hierarchy. | Layer one |
| 1.7 | A failed validation rolls back every change in the same unit of work, leaving no partial effect. | Layer five |
| 1.8 | Two concurrent writes to the same record resolve as specified, with the loser retried automatically. | Layer eight |
| 1.9 | Shipped data loads by external identifier, and reloading it updates the records it owns and leaves records marked as not updatable alone. | Layer two |
| 1.10 | A translatable field returns the value for the active language and falls back as specified when that language has no value. | Layer one |
| 1.11 | A numbering sequence produces the specified format, resets per period as configured, and pads as specified. | Layer one |

**Do not proceed** until 1.2 holds under a dependency that crosses two relations. It is the single most common place a rebuild silently diverges, and every later computed total depends on it.

## Milestone two: access is enforced

Closes step two.

| # | The gate | Proved by |
|---|---|---|
| 2.1 | A user with no groups can read nothing that requires a group. | Layer seven |
| 2.2 | Access rights are enforced for all four operations on every entity, with the specified refusal message identifying the entity and the operation. | Layer seven |
| 2.3 | Global record rules combine with a logical conjunction; rules from the groups a user belongs to combine with a logical disjunction; the two combine with a conjunction. | Layer seven |
| 2.4 | A field restricted to a group is absent for a non-member on read and refused on write. | Layer seven |
| 2.5 | A restricted record cannot be reached indirectly by following a relation from a permitted record. | Layer seven |
| 2.6 | Implied groups are closed transitively: a user in a group that implies another has the second group's rights. | Layer seven |
| 2.7 | Switching the active company changes which records are visible exactly as specified, and a record belonging to a company outside the active set is unreachable. | Layer seven |
| 2.8 | A relation between records of different companies is refused where the consistency check applies, with the specified message. | Layer three |
| 2.9 | The elevate-privileges contract bypasses access rights and record rules and nothing else. | Layer seven |
| 2.10 | Every authentication method in scope succeeds with valid credentials and fails as specified with invalid ones, including the repeated-failure cooldown. | Layer four |

## Milestone three: clients can work

Closes step three.

| # | The gate | Proved by |
|---|---|---|
| 3.1 | A client obtains the menu tree filtered by the user's groups. | Layer six |
| 3.2 | Opening a window action returns the specified views, filter and context. | Layer six |
| 3.3 | View inheritance applies extensions in priority order and the specified node-matching grammar resolves to the specified result. | Layer two |
| 3.4 | Every generic entity operation accepts the specified arguments and returns the specified shape, including grouped reading with aggregation. | Layer six |
| 3.5 | On-change evaluation returns the specified changed values and warnings without persisting anything. | Layer four |
| 3.6 | An error returns the specified envelope with the specified kind. | Layer six |
| 3.7 | An attachment uploads, deduplicates by content, downloads, and is refused to a user who cannot read its record. | Layer seven |
| 3.8 | A report renders to a printable document containing the specified sections, and re-rendering an already-stored document reuses it where the specification says so. | Layer four |

## Milestone four: the system communicates

Closes step four.

| # | The gate | Proved by |
|---|---|---|
| 4.1 | A message posted on a document reaches exactly the followers subscribed to its subtype, by the channel each follower's settings select. | Layer four |
| 4.2 | Changing a tracked field produces a tracking entry recording the old and new values, and posts it with the specified subtype. | Layer four |
| 4.3 | Assigning a responsible user subscribes that user automatically where the specification says so. | Layer four |
| 4.4 | An inbound message routed by reply reference appends to the existing thread; one routed by alias creates a record with the specified field mapping; one that matches nothing is handled as specified. | Layer four |
| 4.5 | A bounced message increments the counter on the party and suppresses further sending as specified. | Layer four |
| 4.6 | An activity falls due, is marked done with feedback, posts a message and creates its chained successor. | Layer three |
| 4.7 | A scheduled job acquires an exclusive lock, so a second worker starting simultaneously does not run it. | Layer eight |
| 4.8 | A scheduled job that fails records the failure and does not block its next run. | Layer four |

## Milestone five: master data is exact

Closes step five.

| # | The gate | Proved by |
|---|---|---|
| 5.1 | Every shipped reference record loads with its specified values, and the counts match the catalogue. | Layer two |
| 5.2 | Unit conversion is exact in both directions for every worked example, with the specified rounding. | Layer one |
| 5.3 | A quantity expressed in a packaging converts to the base unit and back with no drift. | Layer one |
| 5.4 | Currency conversion uses the rate in force on the stated date and rounds to the target currency's precision. | Layer one |
| 5.5 | The rounding primitives match every case, including a value exactly on a rounding boundary. | Layer one |
| 5.6 | Variant generation produces exactly the specified combinations, honours exclusions, and prices each with its extra amount. | Layer four |
| 5.7 | The price computation returns the specified price for every worked example, including a rule based on another price list in a different currency. | Layer one |
| 5.8 | A party's commercial parent resolves across three levels and the specified fields synchronise to children. | Layer four |

**Do not proceed** until 5.2 and 5.5 hold exactly. Every amount and quantity in the remaining steps inherits this arithmetic.

## Milestone six: the money is right

Closes step six. This is the most consequential gate in the plan.

| # | The gate | Proved by |
|---|---|---|
| 6.1 | Every posted entry balances, per entry and per currency, with no exception. | Layer five |
| 6.2 | Posting assigns a number in the specified format, with gaps only where permitted, and resequencing behaves as specified. | Layer three |
| 6.3 | Every lock date refuses a posting inside the locked period with the specified message, and an exception permits exactly what it grants. | Layer three |
| 6.4 | The tax engine reproduces every worked example: percentage, fixed, division and group taxes; price-included extraction; base-amount chaining; per-line against global rounding on the three-line case, to the cent. | Layer one |
| 6.5 | A fiscal position substitutes taxes and accounts as specified on a document and on its items. | Layer four |
| 6.6 | An invoice's term lines distribute the total exactly, with the remainder on the last instalment, for every worked example. | Layer one |
| 6.7 | An early payment discount computes identically under all three modes, and taking it produces the specified entry. | Layer four |
| 6.8 | Cash rounding produces the specified line under both strategies and all rounding methods. | Layer one |
| 6.9 | Reconciling two items produces the specified partial records and leaves the specified residual in both currencies. | Layer one |
| 6.10 | A payment across a rate change produces the specified exchange difference on the specified accounts, and unreconciling reverses it. | Layer four |
| 6.11 | Deferred tax exigibility produces its entry at reconciliation, proportional to the amount settled, with the last part absorbing the rounding. | Layer one |
| 6.12 | An analytic distribution produces lines whose signed amounts sum to the originating item. | Layer five |
| 6.13 | The balance sheet balances and the profit and loss statement ties to the ledger for the seeded data. | Layer five |
| 6.14 | Reversing a document produces the exact opposite entry and reconciles it as specified. | Layer four |
| 6.15 | A tax period closes with the specified entry and the grids agree with the item tags. | Layer four |

## Milestone seven: the goods are right

Closes step seven.

| # | The gate | Proved by |
|---|---|---|
| 7.1 | Quantity is conserved: for every product and location, on hand equals the signed sum of completed movements. | Layer five |
| 7.2 | Reservation follows the specified ordering for every removal strategy, including across batches with different arrival dates. | Layer four |
| 7.3 | Reserved quantity never exceeds on hand and never goes negative, under sequential and concurrent reservation. | Layer five, layer eight |
| 7.4 | A partial validation produces the specified backorder, or none, according to the operation type's policy. | Layer three |
| 7.5 | Multi-step receipt and delivery configurations create exactly the specified locations, operation types, routes and rules. | Layer two |
| 7.6 | Each costing method produces the specified value and unit cost for every worked example, including the negative-stock correction. | Layer one |
| 7.7 | The valuation account balance equals the sum of valuation layers, continuously. | Layer five |
| 7.8 | A landed cost apportions by each split method exactly as specified and adjusts the layers it targets. | Layer one |
| 7.9 | Deferred cost recognition produces the specified entries at invoicing for a partly delivered order. | Layer four |
| 7.10 | A reordering rule proposes the specified quantity, rounded up to its multiple. | Layer one |
| 7.11 | The scheduler groups procurements into the specified number of documents with the specified lines. | Layer four |
| 7.12 | Lead times produce the specified dates for a pick, pack and ship chain and for a purchase. | Layer one |
| 7.13 | A bill of materials explodes with correct unit conversion, and a cycle is refused with the specified message. | Layer three |
| 7.14 | A production consumes, produces and values as specified, including by-product cost shares and a backorder. | Layer four |

## Milestone eight: selling works in every channel

Closes step eight.

| # | The gate | Proved by |
|---|---|---|
| 8.1 | Confirming an order creates exactly the specified downstream records and sets the specified statuses. | Layer four |
| 8.2 | Invoicing on ordered and on delivered quantities each produce the specified lines and quantities. | Layer four |
| 8.3 | An advance invoice deducts exactly on the final invoice, with the specified line and tax treatment. | Layer four |
| 8.4 | The counter application's price, tax, discount and rounding results match the server's for every worked example. | Layer one |
| 8.5 | A counter session closes with the specified entry, line by line, including the cash difference and the excluded invoiced orders. | Layer four |
| 8.6 | Orders created while the counter is offline synchronise once, with no duplicate on replay. | Layer eight |
| 8.7 | A storefront checkout reserves, authorises a payment, confirms the order and produces the specified documents. | Layer four |
| 8.8 | A duplicate payment notification is processed once. | Layer eight |
| 8.9 | A promotion and a gift card on one order produce the specified reward lines, prices and tax treatment. | Layer four |
| 8.10 | Predictive probability reproduces the specified value from the stated frequency counts. | Layer one |

## Milestone nine: people, time and services

Closes step nine.

| # | The gate | Proved by |
|---|---|---|
| 9.1 | Working time yields the specified hours and days across a period containing a public holiday, in a named time zone. | Layer one |
| 9.2 | An alternating two-week schedule resolves to the specified week on a stated date. | Layer one |
| 9.3 | An absence request in days, half days and hours each computes the specified duration and reduces working time accordingly. | Layer four |
| 9.4 | An accrual produces the specified balance across multiple periods, including proration and carry-over. | Layer one |
| 9.5 | Work entry generation produces exactly the specified entries for a two-week period, with a validated absence overlaying correctly and a conflict raised for an overlap. | Layer four |
| 9.6 | Overtime computes with the specified thresholds and absorbs small overruns as specified. | Layer one |
| 9.7 | An expense report posts the specified entry for each payment mode and reaches the specified settled state. | Layer four |
| 9.8 | A timesheet line costs and bills as specified, and a validated line refuses edits with the specified message. | Layer three |
| 9.9 | Project visibility rules admit and refuse exactly the specified users for each privacy setting. | Layer seven |

## Milestone ten: the remaining capabilities

Closes step ten.

| # | The gate | Proved by |
|---|---|---|
| 10.1 | A recurrence generates the specified occurrences; detaching one edits it alone; the three scopes of edit behave as specified. | Layer four |
| 10.2 | Event seat availability and the communication schedule match the specified values. | Layer one |
| 10.3 | A mailing sends to the specified recipients, suppresses the specified ones, and the statistics match the stated counts. | Layer one |
| 10.4 | A questionnaire scores as specified, including partial credit, conditional display and attempt limits. | Layer one |
| 10.5 | Two spreadsheet editors changing different cells converge to the same document, and a reconnecting editor catches up. | Layer eight |
| 10.6 | An automation rule fires on exactly the specified condition, does not retrigger itself, and its time-based form fires at the specified offset. | Layer four |
| 10.7 | An import of two hundred rows with five invalid reports exactly those five and commits the rest, or none, according to the specified mode. | Layer four |

## Closing the programme

The rebuild is complete when every milestone is closed at the conformance level claimed, the invariants of layer five hold continuously across the whole suite, and [coverage and evidence](coverage-and-evidence.md) shows no artefact in the specified column without an entry in either the verified column or the acknowledged-difference list.
