# Milestones and acceptance gates

Every step of the [build sequence](build-sequence.md) closes with a milestone, and every stage of it closes with a stage gate. A gate is a list of statements that must be demonstrably true, each provable by a test of the [equivalence test plan](equivalence-test-plan.md). A gate is not a review meeting: every item is either passing or it is not.

There are three kinds of gate in this document and they answer three different questions.

| Kind | How many | Question it answers | Where |
|---|---:|---|---|
| Step milestones `M0` to `M20` | 21 | Is this delivery correct and complete on its own terms? Each gate names the fixture, the action and the expected result, and cites the acceptance criteria of the domain that specifies the behavior in full. | Sections 3 to 23 |
| Cross-cutting milestones `MX1` to `MX4` | 4 | Do access control, concurrency, numeric stability and bounded work still hold over everything delivered so far? They are re-checked at every later milestone. | Sections 24 to 27 |
| Stage gates one to ten | 10 | Does the property that the whole stage exists for hold across its steps? Each row names the test layer that proves it. | Section 29 |

A milestone may close with acknowledged differences. An acknowledged difference is recorded in [coverage and evidence](coverage-and-evidence.md) with what differs, why, and who decided. An unrecorded difference is a failure of the gate, even if someone knows about it.

---

## 1. How a gate works

### 1.1 The rules

1. **A gate is binary.** Either the check passes exactly as written, or the milestone is not reached. There is no partial credit and no "passes except for rounding".
2. **A gate cites its evidence.** Every gate names the domain acceptance criteria file and the identifier range that specifies the behavior, or a golden scenario of the [equivalence test plan](equivalence-test-plan.md). The gate is the summary; the cited document is the contract.
3. **A gate is run on the reference fixture.** The fixture is the reference company of the equivalence test plan, section 3. A gate that needs extra data says so and the data becomes part of the fixture from that milestone onward.
4. **Gates accumulate.** Every gate of every earlier milestone is re-run at each later milestone. A change that breaks an earlier gate is a defect, not a trade-off.
5. **Coverage is itself a gate.** Each milestone has a final gate stating the percentage of the cited acceptance criteria that must be executed and passing. The percentage is one hundred for every rule that the milestone's domains own.
6. **Messages count.** When a gate expects a refusal, the observed message text must be the message given in the domain document, character for character, with no product name and no identifier substituted into it unless the document shows the substitution.
7. **A difference is either acknowledged or it is a failure.** A gate may close over a known divergence only when that divergence is written down in [coverage and evidence](coverage-and-evidence.md) with its reason and the decision behind it.

### 1.2 Gate identifiers

Step and cross-cutting gates are numbered `GATE-<milestone>-<nn>`, for example `GATE-M4-07`. They are stable: later documents, test suites and the coverage matrix cite them. The milestone numbers match the step numbers of the build sequence; `M0` is preparatory and `MX1` to `MX4` are cross-cutting. Stage gate rows are numbered `<stage>.<nn>`, for example 6.4, where the stage numbers are those of the stage map in section 1.7 of the build sequence.

### 1.3 Evidence formats accepted for a gate

| Evidence | Accepted when |
|---|---|
| An executed scenario with a stored record dump | The gate concerns records written. The dump lists every field the domain document says the operation writes. |
| An executed scenario with a balanced entry listing | The gate concerns accounting. The listing gives account, partner, label, debit, credit, currency amount, tax grid and analytic distribution for every item. |
| A refusal transcript | The gate concerns a validation. The transcript shows the exact message and that no record was written. |
| A rendered document | The gate concerns a report. The rendering is compared field by field against the content list in the domain's `interfaces.md`. |
| A timed run log | The gate concerns a scheduled job or a bounded amount of work. The log gives the moment, the selected records and the effect. |
| A matrix export | The gate concerns access control. The export lists entity, group, create, read, update, delete and the record rule filter. |

### 1.4 The test layer that proves a gate

A step gate names the evidence a reviewer inspects. A stage gate names the layer of the [equivalence test plan](equivalence-test-plan.md) that produces that evidence automatically. The eleven layers are, in order: layer one arithmetic; layer two entity structure; layer three state machines; layer four business scenarios; layer five invariants; layer six contracts; layer seven authorization; layer eight concurrency and recovery; layer nine business rules; layer ten accounting consequences; layer eleven report content. A stage gate row that names a layer is a claim that the layer's suite contains at least one case asserting exactly that statement.

---

## 2. Milestone map

| Milestone | Name | Step | Stage | Gates | Principal evidence |
|---|---|---|---|---:|---|
| M0 | Conventions and fixtures agreed | before 1 | — | 9 | The fixture loads; naming and rounding conventions are implemented as configuration. |
| M1 | The platform runs | 1 | One, two, three | 22 | Entities, access, screens, sequences, jobs, reports. |
| M2 | The business has an identity | 2 | Four, five | 18 | Contacts, companies, currencies, threads, activities, mail. |
| M3 | The catalog is priced | 3 | Five | 20 | Units, products, variants, pricelists. |
| M4 | The books balance | 4 | Six | 24 | Entries, numbering, posting, reversal, lock dates, hashing, reconciliation. |
| M5 | Tax is correct | 5 | Six | 26 | The tax engine, fiscal positions, analytic distribution. |
| M6 | Money moves | 6 | Six | 28 | Invoices, bills, payments, statements, reconciliation, providers. |
| M7 | Goods move | 7 | Seven | 30 | Transfers, reservation, lots, packages, adjustments, shipping. |
| M8 | Stock has a value | 8 | Seven | 22 | Layers, costing methods, valuation entries, landed costs. |
| M9 | Supply is planned | 9 | Seven | 16 | Rules, reordering, lead times, the scheduler. |
| M10 | Procure to pay works | 10 | Seven | 20 | Orders, receipts, bill control, three-way matching. |
| M11 | Quote to cash works | 11 | Eight | 26 | Quotations, orders, deliveries, invoices, down payments, promotions. |
| M12 | Make to stock works | 12 | Seven | 22 | Explosion, production, work orders, unbuild, repair. |
| M13 | A counter day closes | 13 | Eight | 24 | Session, offline orders, payments, closing entries, restaurant. |
| M14 | Demand is tracked | 14 | Eight | 14 | Leads, conversion, assignment, scoring. |
| M15 | People are managed | 15 | Nine | 28 | Employees, schedules, attendance, time off, expenses, recruitment, fleet. |
| M16 | Work is measured | 16 | Nine | 16 | Projects, tasks, milestones, timesheets, billing. |
| M17 | The business talks | 17 | Four | 18 | Channels, live chat, gateway, text messages, blacklists. |
| M18 | The audience is reached | 18 | Ten | 20 | Mailings, campaigns, events, questionnaires, courses. |
| M19 | The public face works | 19 | Eight | 24 | Pages, catalog, cart, checkout, portal. |
| M20 | The platform is complete | 20 | Ten, and the country and exchange part of six | 22 | Spreadsheets, automation, electronic exchange, localizations, blueprints. |
| MX1 | Access control is complete | every | every | 8 | The access matrix and the record rules. |
| MX2 | Concurrency is safe | every | every | 7 | Simultaneous operations on the same records. |
| MX3 | Numbers never drift | every | every | 6 | Precision, rounding and totals. |
| MX4 | Performance is acceptable | every | every | 6 | Search, grouped read, list rendering, report rendering. |

---

## 3. M0: Conventions and fixtures agreed

**Entry condition.** None. **Exit condition.** Every later milestone can assume the conventions below without restating them.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M0-01 | The rounding function rounds half away from zero on a decimal step: rounding 2.675 to 0.01 gives 2.68; rounding −2.675 gives −2.68; rounding 0.005 gives 0.01; rounding 1.3 to a step of 0.5 gives 1.5; rounding 47.62 to a step of 0.05 gives 47.60. | [`../domains/taxes/calculations.md`](../domains/taxes/calculations.md) section 2.1, rounding to a step. |
| GATE-M0-02 | Comparison of two amounts rounds each amount to the step first and subtracts afterwards, while the zero test subtracts first and rounds afterwards; the two therefore disagree on purpose. With a step of 0.01: 1.004 and 1.0 compare equal and 1.006 and 1.0 do not; 0.006 and 0.002 compare unequal, because they round to 0.01 and 0.00, even though the zero test on their difference of 0.004 reports zero. | [`../domains/taxes/calculations.md`](../domains/taxes/calculations.md) section 2.3, comparison, and section 2.2, zero test. |
| GATE-M0-03 | Every date-only field is stored without a time component and every moment field is stored in coordinated universal time and displayed in the user's time zone. Creating a record at 23:30 in a time zone eight hours ahead stores the moment of the same instant, and the list groups it by the local date. | [`../data/persistence-identity-and-values.md`](../data/persistence-identity-and-values.md). |
| GATE-M0-04 | The seven decimal precision records exist with the values of build sequence section 6, are editable, and a change is observed by the next computation without a restart. | [`../domains/products-and-catalog/configuration.md`](../domains/products-and-catalog/configuration.md). |
| GATE-M0-05 | Currency rounding uses the currency's own step: an amount of 1234.567 in a two-decimal currency stores 1234.57; in a zero-decimal currency stores 1235; in a currency with a step of 0.05 stores 1234.55; in a six-decimal currency stores 1234.567000. | [`../domains/multi-currency/calculations.md`](../domains/multi-currency/calculations.md). |
| GATE-M0-06 | Every persistent entity carries the five common fields (identifier, created on, created by user, last updated on, last updated by user) and archivable entities carry the active flag whose false value hides the record from default queries. | [`../overview/entity-and-field-system.md`](../overview/entity-and-field-system.md). |
| GATE-M0-07 | The reference company fixture of the equivalence test plan loads from an empty database, without manual steps, and produces the record counts the fixture declares. | [equivalence-test-plan.md](equivalence-test-plan.md) section 3. |
| GATE-M0-08 | Every shipped record is addressable by its stable external identifier, and loading the same package twice does not duplicate it, does not overwrite a record marked as customizable, and does overwrite a record not so marked. | [`../overview/package-system.md`](../overview/package-system.md). |
| GATE-M0-09 | The condition notation of [`../overview/record-operations-and-query-notation.md`](../overview/record-operations-and-query-notation.md) is implemented for every operator used by the shipped filters, and an unknown operator is refused rather than ignored. | [`../overview/record-operations-and-query-notation.md`](../overview/record-operations-and-query-notation.md). |

---

## 4. M1: The platform runs

**Entry.** M0. **Scope.** Step 1.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M1-01 | An entity defined at runtime with one single-line text field can be created, read, written, duplicated, searched and deleted, and its table exists. | [`../domains/platform-foundation/acceptance-criteria.md`](../domains/platform-foundation/acceptance-criteria.md) identifiers `AC-PF-001` to `AC-PF-012`. |
| GATE-M1-02 | A derived field recomputes when any of its declared inputs changes, in the same unit of work, and a derived stored field is written to storage. | `AC-PF-013` to `AC-PF-030`. |
| GATE-M1-03 | A required field refused as empty produces the message of the domain document and writes nothing. | `AC-PF-031` onwards. |
| GATE-M1-04 | A uniqueness constraint refuses the second record with the message declared on the constraint. | Platform foundation business rules on constraints. |
| GATE-M1-05 | Deleting a record referenced by a restricting relation is refused; by a cascading relation deletes the dependents; by a nulling relation empties the reference. | Platform foundation entities, deletion behavior column. |
| GATE-M1-06 | Reading a set of records returns them in the entity's declared default order, and an explicit order overrides it. | Platform foundation business rules on ordering. |
| GATE-M1-07 | A grouped read returns one row per distinct value with the counts and the aggregations declared per field, including a grouped read on a date field by day, week, month, quarter and year. | Platform foundation interfaces, grouped read contract. |
| GATE-M1-08 | The unit of work flushes pending writes before a query that depends on them, and an exception rolls back every write of the failed operation with no partial effect. | [`../runtime/transactions-and-concurrency.md`](../runtime/transactions-and-concurrency.md). |
| GATE-M1-09 | The cache is invalidated for every record whose field a write touched, including the inverse side of a relation. | [`../runtime/caching.md`](../runtime/caching.md). |
| GATE-M1-10 | An extension that adds a field to an existing entity makes the field visible on every screen that inherits the extended view, without editing the original view. | [`../overview/inheritance-and-extension.md`](../overview/inheritance-and-extension.md). |
| GATE-M1-11 | Installing a package loads its records in file order, resolves every external identifier, and makes its screens and menus appear; uninstalling removes its entities, fields, records, menus and access rules. | `AC-PF` module lifecycle criteria. |
| GATE-M1-12 | A sequence produces consecutive numbers with the configured prefix, suffix, padding and period reset; two simultaneous callers never receive the same number. | [`../domains/platform-foundation/configuration.md`](../domains/platform-foundation/configuration.md). |
| GATE-M1-13 | An attachment is stored, deduplicated by content, served with its declared media type, and deleted with its owning record. | [`../runtime/attachments-and-file-store.md`](../runtime/attachments-and-file-store.md). |
| GATE-M1-14 | A scheduled action runs at its declared interval, does not overlap with itself, records its last execution, and a failed run is retried according to the declared policy. | [`../domains/platform-foundation/configuration.md`](../domains/platform-foundation/configuration.md). |
| GATE-M1-15 | A report renders to a printable document with the company's layout, the chosen page format and the translations of the recipient's language. | [`../runtime/report-rendering.md`](../runtime/report-rendering.md). |
| GATE-M1-16 | A user without the group required by an entity's access rule is refused the operation with the access message of the domain document, and the refusal names the entity and the operation. | [`../domains/identity-and-access/business-rules.md`](../domains/identity-and-access/business-rules.md). |
| GATE-M1-17 | A record rule restricts a query so that a user sees only the records the rule allows, rules of the same group combine with a logical or and rules of different groups with a logical and. | `IDAC-AC` record rule criteria. |
| GATE-M1-18 | Password authentication, application key authentication and two-factor enrolment each work, and an application key can be revoked and immediately stops working. | `IDAC-AC-001` onwards. |
| GATE-M1-19 | A screen definition of each presentation kind (list, form, kanban, calendar, pivot, graph, activity, hierarchy) renders with the fields, buttons, filters and groupings the definition declares. | [`../domains/platform-foundation/interfaces.md`](../domains/platform-foundation/interfaces.md). |
| GATE-M1-20 | Exporting records produces one row per record with the chosen fields in the chosen order, and re-importing that file reproduces the records including their relations resolved by external identifier. | [`../interfaces/report-and-export-documents.md`](../interfaces/report-and-export-documents.md). |
| GATE-M1-21 | The three foundation scheduled jobs run with their declared effect and are idempotent within one interval. | Build sequence step 1 job table. |
| GATE-M1-22 | One hundred percent of the acceptance criteria of the platform foundation and identity domains are executed and pass. | [`../domains/platform-foundation/acceptance-criteria.md`](../domains/platform-foundation/acceptance-criteria.md), [`../domains/identity-and-access/acceptance-criteria.md`](../domains/identity-and-access/acceptance-criteria.md). |

---

## 5. M2: The business has an identity

**Entry.** M1. **Scope.** Step 2.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M2-01 | A contact of kind individual linked to a contact of kind company inherits the address fields of the parent until they are overridden, and the commercial entity of both is the company. | [`../domains/contacts-and-organizations/business-rules.md`](../domains/contacts-and-organizations/business-rules.md). |
| GATE-M2-02 | The address of a contact renders according to the address format of its country, and a country that requires a state refuses a contact without one. | Contacts business rules, address format. |
| GATE-M2-03 | A tax identification number is validated against the format of its country and the refusal message names the expected format. | Contacts business rules. |
| GATE-M2-04 | Merging two contacts moves every linked document to the survivor, keeps the oldest creation moment, and writes the merge in the discussion thread. | Contacts workflows, merge. |
| GATE-M2-05 | A bank account is unique by number and bank, and a bank account of a contact of another company is refused on a company-scoped document. | Contacts business rules. |
| GATE-M2-06 | Converting 100.00 of a currency into another at a rate of 1.2345 gives the amount rounded to the target currency's step, and converting back does not necessarily return the original amount; the conversion always uses the rate in force at the requested date, which is the latest rate whose date is not after it. | [`../domains/multi-currency/calculations.md`](../domains/multi-currency/calculations.md). |
| GATE-M2-07 | Two rates for the same currency and the same date in the same company are refused. | Multi-currency business rules. |
| GATE-M2-08 | Posting a message on a record creates the message, notifies exactly the followers subscribed to its subtype, and does not notify the author. | [`../domains/messaging-and-activities/acceptance-criteria.md`](../domains/messaging-and-activities/acceptance-criteria.md) `AC-001` to `AC-010`. |
| GATE-M2-09 | Changing a tracked field writes one tracking value per changed field with the old and new display values, in one message. | Messaging acceptance criteria, tracking section. |
| GATE-M2-10 | Scheduling an activity sets its due date from the activity type's delay, notifies the assignee, and marking it done posts the configured message and chains the next activity when the type declares one. | [`../domains/messaging-and-activities/workflows.md`](../domains/messaging-and-activities/workflows.md). |
| GATE-M2-11 | Rendering an electronic mail template substitutes every placeholder, uses the recipient's language, and attaches the report the template declares. | Messaging interfaces, template rendering. |
| GATE-M2-12 | An outgoing electronic mail that fails is marked in error with the failure reason, is not retried more than the configured number of times, and never blocks the rest of the queue. | Build sequence step 2 job table. |
| GATE-M2-13 | A recurring calendar event expands into the declared occurrences, editing one occurrence detaches it, and editing the series from an occurrence rewrites the following ones only. | [`../domains/calendar-and-scheduling/workflows.md`](../domains/calendar-and-scheduling/workflows.md). |
| GATE-M2-14 | A calendar reminder fires once per attendee and is not re-sent after the event moment passes. | Calendar business rules. |
| GATE-M2-15 | A record created in company A is invisible to a user whose allowed companies do not include A, and a document that mixes two companies in its relations is refused with the company consistency message. | [`../overview/multi-company.md`](../overview/multi-company.md). |
| GATE-M2-16 | The document layout chosen by a company applies to every printed document of that company, including the header, the footer, the colors and the page format. | [`../domains/contacts-and-organizations/configuration.md`](../domains/contacts-and-organizations/configuration.md). |
| GATE-M2-17 | The five collaboration scheduled jobs run with their declared effect. | Build sequence step 2 job table. |
| GATE-M2-18 | One hundred percent of the acceptance criteria of contacts, multi-currency and calendar, and of the thread, tracking and activity sections of messaging, are executed and pass. | The four domains' acceptance files. |

---

## 6. M3: The catalog is priced

**Entry.** M2. **Scope.** Step 3.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M3-01 | Converting a quantity between two units of the same category multiplies or divides by the declared ratio and rounds up to the target unit's rounding step; converting between categories is refused with the message of the domain document. | [`../domains/units-of-measure-and-packaging/acceptance-criteria.md`](../domains/units-of-measure-and-packaging/acceptance-criteria.md) `UNIT-AC-001` onwards. |
| GATE-M3-02 | A category has exactly one reference unit, the reference unit has a ratio of one, and a second reference unit in the same category is refused. | `UNIT-AC` reference unit criteria. |
| GATE-M3-03 | Rounding a quantity of 0.007 to a unit whose step is 0.01 gives 0.01 when rounding up is requested and 0.00 when rounding to nearest is requested; the direction used by each caller is the one the domain declares. | [`../domains/units-of-measure-and-packaging/calculations.md`](../domains/units-of-measure-and-packaging/calculations.md). |
| GATE-M3-04 | A product template with two attributes of three and two values generates six variants, archiving a value archives the variants that use it, and reactivating it restores them with their original identifiers. | [`../domains/products-and-catalog/acceptance-criteria.md`](../domains/products-and-catalog/acceptance-criteria.md) `PROD-AC` variant section. |
| GATE-M3-05 | An attribute value with an extra price adds that price to the variant's sales price, and the extra price is expressed in the pricelist currency of the template. | `PROD-AC` extra price criteria. |
| GATE-M3-06 | An exclusion between two attribute values prevents the generation of the combination and refuses its manual creation. | `PROD-AC` exclusion criteria. |
| GATE-M3-07 | A barcode is unique across products and packagings, and scanning a barcode that matches a nomenclature rule returns the parsed quantity, price or weight the rule declares. | `PROD-AC` barcode criteria. |
| GATE-M3-08 | A product cannot be deleted once it appears on any document; archiving is offered instead with the message of the domain document. | `PROD-AC` deletion criteria. |
| GATE-M3-09 | The pricelist resolution picks the rule with the highest specificity in the declared order (variant, template, category with its parents, global), then the smallest minimum quantity that the ordered quantity satisfies, then the most recent validity window. | [`../domains/pricing-and-pricelists/acceptance-criteria.md`](../domains/pricing-and-pricelists/acceptance-criteria.md) `PR-AC` resolution section. |
| GATE-M3-10 | A rule based on the sales price with a twenty percent discount, a surcharge of 5.00 and rounding to 0.95 applied to a list price of 100.00 gives 80.95 and the computation order is discount, surcharge, rounding, minimum margin. | [`../domains/pricing-and-pricelists/calculations.md`](../domains/pricing-and-pricelists/calculations.md). |
| GATE-M3-11 | A pricelist in another currency converts the base price at the rate of the document date before applying the rule. | `PR-AC` currency criteria. |
| GATE-M3-12 | A pricelist rule based on another pricelist resolves recursively and detects a cycle, refusing it with the message of the domain document. | `PR-AC` recursion criteria. |
| GATE-M3-13 | The vendor price of a product returns the price of the vendor with the smallest sequence whose minimum quantity and validity window match, in the vendor's currency and unit of measure. | `PR-AC` vendor price criteria. |
| GATE-M3-14 | A product packaging declares a quantity per package and converting a quantity to packages and back is exact for whole packages. | `UNIT-AC` packaging criteria. |
| GATE-M3-15 | A combo product prices its components according to the declared allocation and the sum of the allocated prices equals the combo price to the currency step. | `PROD-AC` combo criteria. |
| GATE-M3-16 | The four product label layouts render the declared fields at the declared positions for a page of labels and for a single label. | [`../domains/products-and-catalog/interfaces.md`](../domains/products-and-catalog/interfaces.md). |
| GATE-M3-17 | The pricelist report renders one row per product with the price of each selected pricelist and each selected quantity. | Products interfaces, pricelist report. |
| GATE-M3-18 | Changing a product's unit of measure after a stock movement exists is refused with the message of the domain document. | `PROD-AC` unit change criteria. |
| GATE-M3-19 | The product cost and the sales price are stored at the product price precision and are never rounded to the currency step in storage. | Build sequence section 6. |
| GATE-M3-20 | One hundred percent of the acceptance criteria of units of measure, products and pricing are executed and pass. | The three domains' acceptance files. |

---

## 7. M4: The books balance

**Entry.** M2. **Scope.** Step 4.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M4-01 | An unbalanced entry is refused at posting with the exact message of the domain document, and nothing is written. | [`../domains/general-ledger/acceptance-criteria.md`](../domains/general-ledger/acceptance-criteria.md) balance section. |
| GATE-M4-02 | An account code is unique per company and the refusal lists the duplicate codes. | `BOOK-AC-001` to `BOOK-AC-004`. |
| GATE-M4-03 | A receivable or payable account must allow reconciliation, and an off-balance account allows neither reconciliation nor taxes. | `BOOK-AC-005`, `BOOK-AC-006`. |
| GATE-M4-04 | Posting assigns the number from the journal's sequence with the period the accounting date falls in, and two entries posted in the same period receive consecutive numbers. | `BOOK-AC` numbering section. |
| GATE-M4-05 | A gap in the numbering of a journal is detected and reported by the gap check, naming the missing numbers. | `BOOK-AC` gap section. |
| GATE-M4-06 | Resequencing a range of posted entries renumbers them in date order and refuses to cross a lock date. | `BOOK-AC` resequencing section. |
| GATE-M4-07 | Posting an entry whose accounting date is before a lock date is refused with the message of the domain document, and is allowed when an exception was granted to that user for that period. | `BOOK-AC` lock date section. |
| GATE-M4-08 | Reversing a posted entry creates a new entry with the opposite amounts, the reversal date, a link in both directions, and reconciles the two when the domain says it does. | `BOOK-AC` reversal section. |
| GATE-M4-09 | Editing a posted entry is refused; resetting it to draft is allowed only when the domain allows it and is refused after the hash chain covers it. | `BOOK-AC` inalterability section. |
| GATE-M4-10 | The hash of a posted entry depends on the declared fields in the declared order, chains to the previous entry of the same journal, and the integrity check reports the first broken link. | `BOOK-AC` hashing section. |
| GATE-M4-11 | Reconciling two items of the same account and the same partner whose amounts are equal and opposite marks both fully reconciled and creates the full reconciliation record. | `BOOK-AC` reconciliation section. |
| GATE-M4-12 | Reconciling a debit of 100.00 with a credit of 40.00 leaves a residual of 60.00 on the debit and creates one partial reconciliation of 40.00. | `BOOK-AC` partial reconciliation section. |
| GATE-M4-13 | Unreconciling removes the partial reconciliations, restores the residual amounts, and deletes any write-off entry the reconciliation created. | `BOOK-AC` unreconciliation section. |
| GATE-M4-14 | An item on an account that requires a partner without a partner is refused. | `BOOK-AC` item validation section. |
| GATE-M4-15 | An item in a currency other than the company currency carries both the currency amount and the company amount, and the company amount equals the conversion at the entry's date. | `BOOK-AC` currency section. |
| GATE-M4-16 | The chart of accounts template loads accounts, groups, taxes, fiscal positions and journals with their stable identifiers, and loading it into a company that already has posted entries is refused. | `BOOK-AC` template section. |
| GATE-M4-17 | Merging two accounts moves every item, keeps the survivor's code and refuses when the two accounts differ in type or reconciliation setting. | `BOOK-AC` merge section. |
| GATE-M4-18 | The automatic entry wizard produces the cut-off entry with the declared accounts and the reversal entry at the declared date. | `BOOK-AC` automatic entry section. |
| GATE-M4-19 | The opening balance wizard creates one entry per account with the declared amounts and balances it on the unallocated result account. | `BOOK-AC` opening section. |
| GATE-M4-20 | The six shipped accounting consistency tests run and report zero anomalies on the fixture. | Ledger configuration, consistency tests. |
| GATE-M4-21 | The daily automatic posting job posts only entries flagged for it whose accounting date has arrived, and records a failure without stopping the batch. | Build sequence step 4 job table. |
| GATE-M4-22 | Every printed ledger document renders with the content declared in the domain's interfaces file. | [`../domains/general-ledger/interfaces.md`](../domains/general-ledger/interfaces.md). |
| GATE-M4-23 | An entry whose journal belongs to another company than the entry is refused. | `BOOK-AC` company consistency section. |
| GATE-M4-24 | One hundred percent of the general ledger acceptance criteria are executed and pass; that file carries two hundred and seventy-seven numbered scenarios in thirty sections, and none of them may be skipped. | [`../domains/general-ledger/acceptance-criteria.md`](../domains/general-ledger/acceptance-criteria.md). |

---

## 8. M5: Tax is correct

**Entry.** M4. **Scope.** Step 5.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M5-01 | A twenty-one percent price-excluded tax on three units at 19.99 gives a base of 59.97 and a tax of 12.59, and the total including tax is 72.56. | [`../domains/taxes/calculations.md`](../domains/taxes/calculations.md) section 4.2, percentage price-excluded, with the line arithmetic of section 6; section A, single-line computation, of [`../domains/taxes/acceptance-criteria.md`](../domains/taxes/acceptance-criteria.md). |
| GATE-M5-02 | A twenty-one percent price-included tax on 121.00 gives a base of 100.00 and a tax of 21.00; on 11.90 with rounding per line gives 2.07 and 9.83; with global rounding gives 2.065289 and 9.834711. | [`../domains/taxes/calculations.md`](../domains/taxes/calculations.md) section 4.3, percentage price-included, and section 7.8, round per line against round per tax. |
| GATE-M5-03 | A five percent price-included tax on 50.00 in a currency whose step is 0.05 gives a tax of 2.40 and a base of 47.60. | Same section. |
| GATE-M5-04 | Two chained percentage taxes where the first affects the base of the second produce the amounts of the worked example, in the declared evaluation order. | Section 3.10.3. |
| GATE-M5-05 | A fixed tax per unit multiplies by the quantity and is not affected by the discount unless the domain says it is. | Section 3.10.5 and 3.10.7. |
| GATE-M5-06 | A division tax computes its base as the amount divided by one minus the rate, and the worked example's numbers are reproduced exactly. | Section 3.10.8. |
| GATE-M5-07 | A group of taxes expands into its children in their declared sequence and reports one line per child. | Section 3.10.9. |
| GATE-M5-08 | A negative quantity, a negative price and a zero price each produce the amounts of the worked example, with the sign rules of section 1.2. | Section 3.10.10. |
| GATE-M5-09 | Two taxes on one line, one price-included and one price-excluded, produce the amounts of the worked example. | Section 3.10.11. |
| GATE-M5-10 | The rounding method "round per line" and the method "round globally" produce the document totals of the worked examples and differ exactly where the document says they differ. | Section 5. |
| GATE-M5-11 | The tax amount of a document is distributed over the distribution lines by percentage, and the residue from rounding is allocated to the largest line so that the sum equals the rounded total. | Section 5.4. |
| GATE-M5-12 | A tax with a repartition to an account and a tag writes the amount on that account and stamps the tag on the base line and the tax line as declared. | `TAX-AC` distribution section. |
| GATE-M5-13 | A fiscal position substitutes tax A by tax B on a document line and the substitution keeps the same price including tax when the domain declares it does. | `TAX-AC` fiscal position section. |
| GATE-M5-14 | A fiscal position substitutes an income account and the substitution is applied when the invoice line is created and when the customer is changed. | `TAX-AC` account mapping section. |
| GATE-M5-15 | A fiscal position is selected automatically from the customer's country and state, with the declared precedence, and a manually chosen one is never overwritten. | `TAX-AC` automatic detection section. |
| GATE-M5-16 | A cash-basis tax posts no tax at invoice posting and posts it proportionally at each payment, with the base amount on the declared base account. | `TAX-AC` cash basis section. |
| GATE-M5-17 | A withholding tax produces a negative tax line and the net payable equals the total minus the withheld amount. | `TAX-AC` withholding section. |
| GATE-M5-18 | A reverse charge tax produces two opposite tax lines whose net effect on the books is zero and whose grids differ. | Section 3.9. |
| GATE-M5-19 | A tax report aggregates the grids of a period, reports each line with the declared expression, and the comparison with the previous period is computed from the same expressions. | `TAX-AC` report section. |
| GATE-M5-20 | A manual external value added to a report line appears in the total and is attributed to the declared period. | `TAX-AC` external value section. |
| GATE-M5-21 | An analytic distribution of sixty and forty percent over two accounts creates two analytic lines whose amounts sum to the item amount, rounded to the currency step with the residue on the larger share. | [`../domains/analytic-accounting/acceptance-criteria.md`](../domains/analytic-accounting/acceptance-criteria.md) `AN-AC` distribution section. |
| GATE-M5-22 | A mandatory analytic plan refuses the posting of an item that lacks a distribution for that plan, with the message of the domain document. | `AN-AC` mandatory plan section. |
| GATE-M5-23 | An analytic distribution model applies automatically when its applicability conditions match, and the most specific model wins. | `AN-AC` model section. |
| GATE-M5-24 | Changing the distribution on a posted item rewrites its analytic lines and leaves the journal items untouched. | `AN-AC` rewrite section. |
| GATE-M5-25 | An analytic line in a currency other than the plan's currency stores both amounts. | `AN-AC` currency section. |
| GATE-M5-26 | One hundred percent of the acceptance criteria of taxes and analytic accounting are executed and pass. | The two domains' acceptance files. |

---

## 9. M6: Money moves

**Entry.** M5. **Scope.** Step 6.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M6-01 | A new customer invoice takes the sale journal, the draft state, today's accounting date, the customer's payment terms and fiscal position, and an empty number showing the placeholder of the next number. | [`../domains/accounts-receivable/acceptance-criteria.md`](../domains/accounts-receivable/acceptance-criteria.md) `RECV-AC` creation section. |
| GATE-M6-02 | An invoice line created from a product takes the product's sales price, income account, taxes after fiscal position mapping, and the customer's pricelist price when a pricelist applies. | `RECV-AC` line default section. |
| GATE-M6-03 | Posting a customer invoice writes one receivable item per payment term installment with the computed due dates, one item per product line and one item per tax, and the entry balances. | [`../domains/accounts-receivable/accounting-effects.md`](../domains/accounts-receivable/accounting-effects.md). |
| GATE-M6-04 | A payment term of thirty percent immediately and the balance in thirty days on an invoice of 1 000.00 including tax produces two receivable items of 300.00 due today and 700.00 due in thirty days. | `RECV-AC` payment terms section. |
| GATE-M6-05 | An early payment discount of two percent within ten days on an invoice of 1 210.00 including twenty-one percent tax computes the discounted amount, and paying within the window writes the discount to the declared gain account with the tax treatment the term declares. | [`../domains/accounts-receivable/calculations.md`](../domains/accounts-receivable/calculations.md) early discount section. |
| GATE-M6-06 | Cash rounding to the nearest 0.05 on a total of 100.02 adds a rounding line of −0.02 on the declared account and leaves the tax amounts untouched when the method is "add a rounding line". | `RECV-AC` cash rounding section. |
| GATE-M6-07 | Creating a credit note by reversal produces the mirrored entry, links both documents and offers the three reversal modes with their declared effects. | `RECV-AC` credit note section. |
| GATE-M6-08 | Sending an invoice by the sending service marks it sent, stores the rendered document as an attachment, and records the channels used. | `RECV-AC` sending section. |
| GATE-M6-09 | A vendor bill created from a document import proposes the vendor, the date, the due date, the reference and the lines read from the document, and the user's corrections are kept. | [`../domains/accounts-payable/acceptance-criteria.md`](../domains/accounts-payable/acceptance-criteria.md) `AC-PAY` import section. |
| GATE-M6-10 | A duplicate vendor reference on the same vendor raises the duplicate warning with the message of the domain document. | `AC-PAY` duplicate section. |
| GATE-M6-11 | Registering a payment of 300.00 against an invoice of 1 000.00 posts a payment entry, reconciles it partially, and leaves a residual of 700.00 with the invoice in the partially paid state. | [`../domains/payments-and-bank-reconciliation/acceptance-criteria.md`](../domains/payments-and-bank-reconciliation/acceptance-criteria.md) `PAY-AC` registration section. |
| GATE-M6-12 | Registering one payment against two invoices of the same customer with the grouping option produces one payment and two partial reconciliations. | `PAY-AC` grouping section. |
| GATE-M6-13 | A payment in a currency other than the company currency reconciled with an invoice in that currency at a different rate creates the exchange difference entry on the declared gain or loss account, for exactly the rate difference. | `PAY-AC` exchange section; [`../domains/multi-currency/accounting-effects.md`](../domains/multi-currency/accounting-effects.md). |
| GATE-M6-14 | A payment uses the outstanding receipts or outstanding payments account of its method until it is reconciled with a statement line, at which point the liquidity account carries the amount. | `PAY-AC` outstanding section. |
| GATE-M6-15 | Importing a bank statement creates one statement line per transaction, computes the running balance, and refuses a statement whose ending balance does not match the computed one when the journal requires the match. | `PAY-AC` statement section. |
| GATE-M6-16 | A reconciliation model with a matching rule on the payment reference proposes the invoice, and applying it reconciles the statement line and posts any write-off the model declares. | [`../domains/payments-and-bank-reconciliation/workflows.md`](../domains/payments-and-bank-reconciliation/workflows.md). |
| GATE-M6-17 | A structured payment reference is generated with the country's check digits and a payment carrying it is matched automatically. | [`../domains/payments-and-bank-reconciliation/interfaces.md`](../domains/payments-and-bank-reconciliation/interfaces.md). |
| GATE-M6-18 | Undoing the reconciliation of a statement line restores the line to unreconciled, removes the generated items and leaves the invoice with its former residual. | `PAY-AC` undo section. |
| GATE-M6-19 | A check payment reserves a check number, prints in the configured layout, and voiding it releases the number according to the domain rule. | `AC-PAY` check section. |
| GATE-M6-20 | Creating a payment transaction moves it through the declared states and a provider callback that arrives twice has the effect only once. | [`../domains/payment-providers/acceptance-criteria.md`](../domains/payment-providers/acceptance-criteria.md) `PAY-AC` transaction section. |
| GATE-M6-21 | A transaction authorized and then captured posts the payment at capture and not at authorization; a voided authorization posts nothing. | `PAY-AC` authorization section. |
| GATE-M6-22 | A refund transaction creates the linked refund payment and the credit note when the flow declares one. | `PAY-AC` refund section. |
| GATE-M6-23 | A saved payment token charges without re-entering the credentials and is invalidated when the provider reports it as expired. | `PAY-AC` tokenization section. |
| GATE-M6-24 | A webhook whose signature does not verify is rejected and logged, and no transaction changes state. | [`../domains/payment-providers/interfaces.md`](../domains/payment-providers/interfaces.md). |
| GATE-M6-25 | The payment link of an invoice opens the payment page for exactly the residual amount and expires according to the access token rules. | `PAY-AC` link section. |
| GATE-M6-26 | The post-processing job finalizes transactions abandoned in an intermediate state and is safe to run twice. | Build sequence step 6 job table. |
| GATE-M6-27 | Every printed receivable and payable document renders with the content declared in the domains' interfaces files. | The two domains' interfaces files. |
| GATE-M6-28 | One hundred percent of the acceptance criteria of receivables, payables, payments and providers are executed and pass. | The four domains' acceptance files. |

---

## 10. M7: Goods move

**Entry.** M3 and M4. **Scope.** Step 7.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M7-01 | A warehouse creates its locations, its operation types and its sequences on creation, and renaming its short code renames the sequences. | [`../domains/inventory-operations/acceptance-criteria.md`](../domains/inventory-operations/acceptance-criteria.md) `INV-AC` warehouse section. |
| GATE-M7-02 | Confirming a transfer creates the moves in the waiting or ready state according to availability and the reservation policy of the operation type. | `INV-AC` confirmation section. |
| GATE-M7-03 | Checking availability reserves quantities on the quantity records in the order the removal strategy declares, and the reserved quantity never exceeds the available quantity. | `INV-AC` reservation section. |
| GATE-M7-04 | Validating a transfer with a partial quantity offers the backorder dialog, and confirming it creates the backorder with the remaining quantities and links both documents. | `INV-AC` backorder section. |
| GATE-M7-05 | Validating a transfer moves the quantity from the source location to the destination location, writes the move lines with their lots and packages, and sets the transfer to done with the effective date. | `INV-AC` validation section. |
| GATE-M7-06 | A tracked product refuses validation without a lot or serial number, with the message of the domain document. | [`../domains/inventory-operations/business-rules.md`](../domains/inventory-operations/business-rules.md). |
| GATE-M7-07 | A serial number cannot be in two locations at once and the second attempt is refused. | Lots and serial numbers rules. |
| GATE-M7-08 | The first expired first out strategy proposes the lot with the earliest removal date, and lots without a date come last. | `INV-AC` removal strategy section. |
| GATE-M7-09 | A putaway rule directs an incoming quantity to the child location it declares, and a storage category that is full sends it to the next rule or to the default location. | [`../domains/inventory-operations/configuration.md`](../domains/inventory-operations/configuration.md). |
| GATE-M7-10 | Putting move lines in a package creates the package, moves its content together, and unpacking restores the individual lines. | [`../domains/inventory-operations/workflows.md`](../domains/inventory-operations/workflows.md). |
| GATE-M7-11 | An inventory adjustment that counts fewer units than on hand creates the loss move on the declared adjustment location, and the counted quantity becomes the on-hand quantity. | [`../domains/inventory-operations/workflows.md`](../domains/inventory-operations/workflows.md). |
| GATE-M7-12 | Two users counting the same quantity record at the same time produce the conflict dialog and the second write is refused until the conflict is resolved. | `INV-AC` conflict section. |
| GATE-M7-13 | Scrapping a quantity moves it to the scrap location, is refused beyond the available quantity, and records the reason. | `INV-AC` scrap section. |
| GATE-M7-14 | Returning a validated delivery creates the reverse transfer with the returnable quantities and links both documents. | `INV-AC` return section. |
| GATE-M7-15 | Cancelling a transfer cancels its moves, releases their reservations and is refused once the transfer is done. | `INV-AC` cancellation section. |
| GATE-M7-16 | A batch transfer groups transfers of the same operation type, validates them together and splits the backorders per original transfer. | [`../domains/inventory-operations/workflows.md`](../domains/inventory-operations/workflows.md). |
| GATE-M7-17 | A wave transfer groups selected lines across transfers and returns the remaining lines to their original transfers on validation. | Batch and wave rules. |
| GATE-M7-18 | The delivery method computes the shipping price from its price rules on weight, volume, quantity or amount, and a rule that no condition matches makes the method unavailable with the declared message. | [`../domains/delivery-and-shipping/calculations.md`](../domains/delivery-and-shipping/calculations.md). |
| GATE-M7-19 | Sending a shipment to a carrier stores the tracking reference and the label, and cancelling it clears them and records the cancellation. | Shipping rules. |
| GATE-M7-20 | A move whose product has a unit of measure different from the line's unit converts the quantity and stores both quantities consistently. | `INV-AC` unit section. |
| GATE-M7-21 | Forecasted availability on a date reports the on-hand quantity plus incoming minus outgoing up to that date, per warehouse and per location when asked. | `INV-AC` forecast section. |
| GATE-M7-22 | The traceability report of a lot lists every movement of that lot in chronological order with its documents. | Inventory interfaces, traceability. |
| GATE-M7-23 | Every inventory printed document renders the declared content, including barcodes at the declared symbology. | [`../domains/inventory-operations/interfaces.md`](../domains/inventory-operations/interfaces.md). |
| GATE-M7-24 | An operation type with a reservation method of "at confirmation", "manually" and "before the scheduled date" each behaves as declared. | `INV-AC` reservation method section. |
| GATE-M7-25 | Multi-step receipt and delivery routes create the chained transfers in the declared order and propagate cancellation and rescheduling as declared. | `INV-AC` multi-step section. |
| GATE-M7-26 | Two simultaneous reservations of the same quantity record never over-reserve; one of them waits or is refused according to the concurrency rule. | [`../runtime/transactions-and-concurrency.md`](../runtime/transactions-and-concurrency.md). |
| GATE-M7-27 | A quantity record is never negative unless the location allows negative stock, and the refusal message is the one of the domain document. | `INV-AC` negative stock section. |
| GATE-M7-28 | Relocating quantities between locations writes the internal moves and preserves lots and packages. | `INV-AC` relocation section. |
| GATE-M7-29 | Printing lot labels, product labels and location barcodes produces the declared layouts. | Inventory interfaces. |
| GATE-M7-30 | One hundred percent of the inventory acceptance criteria are executed and pass; that file carries two hundred and one numbered scenarios, and none of them may be skipped. | [`../domains/inventory-operations/acceptance-criteria.md`](../domains/inventory-operations/acceptance-criteria.md). |

---

## 11. M8: Stock has a value

**Entry.** M7 and M5. **Scope.** Step 8.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M8-01 | Every movement is classified as incoming, outgoing, internal or neither, by the tests the domain declares, and the classification is stored on the movement. | [`../domains/inventory-valuation-and-costing/acceptance-criteria.md`](../domains/inventory-valuation-and-costing/acceptance-criteria.md) `VAL-AC` classification section. |
| GATE-M8-02 | With standard costing, a receipt of ten units at a bill price of 12.00 while the product cost is 10.00 values the layer at 100.00 and posts the difference of 20.00 to the price difference account. | `VAL-AC` standard costing section. |
| GATE-M8-03 | With average costing, receiving eight units at 10.00 then four at 16.00 gives a cost of 12.00; delivering ten values the delivery at 120.00; receiving two at 6.00 gives a cost of 9.00 and a value of 36.00. | [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md) section 3.2, maintaining the product unit cost, and section 4.3, the average replay. |
| GATE-M8-04 | With average costing and negative stock, the sequence of the worked example ends with a cost of 20.00, a quantity of 5 and a value of 100.00, and no earlier movement is edited. | Section 5.4. |
| GATE-M8-05 | The average is never rounded before it is multiplied: receiving three units at 1.00, 1.00 and 1.01 and delivering all three leaves a total value of exactly 0.00. | Section 5.5. |
| GATE-M8-06 | With first in first out, receipts of sixty-eight at 15.00 and one hundred and forty at 15.50 followed by a delivery of ninety-four value the delivery at 1 423.00 and leave a remaining value of 1 767.00 and a cost of 15.50. | Section 4.5. |
| GATE-M8-07 | With first in first out and negative stock, the later correction reproduces the worked example exactly. | Section 4.6. |
| GATE-M8-08 | An automated valuation posts the stock input, stock valuation and stock output items with the accounts selected by the declared precedence (product, category, company). | [`../domains/inventory-valuation-and-costing/accounting-effects.md`](../domains/inventory-valuation-and-costing/accounting-effects.md). |
| GATE-M8-09 | A manual valuation posts nothing at movement time and the daily closing entry posts the difference between the physical and the accounting value per category. | Accounting effects, periodic section. |
| GATE-M8-10 | A landed cost of 300.00 split by value over two receipts of 1 000.00 and 2 000.00 allocates 100.00 and 200.00, posts the adjustment items and updates the layers. | [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md). |
| GATE-M8-11 | A landed cost split by quantity, by weight and by volume allocates according to those bases, and a base whose total is zero is refused with the declared message. | Landed costs rules. |
| GATE-M8-12 | Revaluing a product writes the revaluation layer and posts the entry; revaluing to a negative total value is refused. | `VAL-AC` revaluation section. |
| GATE-M8-13 | Changing the costing method of a category rewrites the cost of its products according to the declared rule and posts no retroactive entry. | Calculations section 6.2. |
| GATE-M8-14 | A return of a receipt values the return at the value of the original layer, not at the current cost. | Calculations section 2.5. |
| GATE-M8-15 | A dropship movement is valued as an incoming followed by an outgoing and the net effect on the stock accounts is zero. | `VAL-AC` dropship section. |
| GATE-M8-16 | A lot-valued product keeps one cost per lot and a delivery of that lot uses the lot's cost. | [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md). |
| GATE-M8-17 | The valuation statement reports quantity, unit cost and total value per product and per layer, and the sum equals the balance of the stock valuation account when the valuation is automated. | Valuation interfaces. |
| GATE-M8-18 | Correcting the quantity of a done movement recomputes its value and posts the correcting entry. | Calculations section 3.4. |
| GATE-M8-19 | A vendor bill posted after the receipt with a different price posts the price difference according to the costing method, with the declared account. | Calculations section 2.2. |
| GATE-M8-20 | The closing job is idempotent within a day and its entries are dated on the closing date. | Build sequence step 8 job table. |
| GATE-M8-21 | Valuation never uses floating rounding of intermediate averages, proved by a replay of one thousand random movements whose total value equals the sum of the layer values exactly. | [equivalence test plan](equivalence-test-plan.md), the invariants of section 18. |
| GATE-M8-22 | One hundred percent of the valuation acceptance criteria are executed and pass. | [`../domains/inventory-valuation-and-costing/acceptance-criteria.md`](../domains/inventory-valuation-and-costing/acceptance-criteria.md). |

---

## 12. M9: Supply is planned

**Entry.** M7. **Scope.** Step 9.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M9-01 | A demand at a location resolves the rule by walking the location's parents and then the product's routes in the declared order of precedence. | [`../domains/replenishment-and-procurement/acceptance-criteria.md`](../domains/replenishment-and-procurement/acceptance-criteria.md) `AC-001` onwards. |
| GATE-M9-02 | A pull rule creates the supplying move with the declared source and destination and links it to the demand. | Replenishment workflows, pull. |
| GATE-M9-03 | A push rule creates the resulting move after the supplying move is done. | Replenishment workflows, push. |
| GATE-M9-04 | A reordering rule with a minimum of ten, a maximum of fifty and a multiple of five orders forty-five when the forecast is five. | Replenishment calculations, quantity. |
| GATE-M9-05 | A reordering rule whose forecast is above the minimum orders nothing. | Same section. |
| GATE-M9-06 | The lead time of a purchase is the vendor delay plus the purchase security lead time plus the days to purchase, and the order date is computed backwards from the demand date. | Replenishment calculations, lead times. |
| GATE-M9-07 | The lead time of a manufacture is the bill of materials lead time plus the manufacturing security lead time. | Same section. |
| GATE-M9-08 | Make to order creates one supply per demand and never groups two demands. | Replenishment rules, make to order. |
| GATE-M9-09 | Cancelling a demand cancels its propagated supply when the rule declares propagation and leaves it otherwise. | Replenishment rules, propagation. |
| GATE-M9-10 | Rescheduling a demand reschedules its propagated supply and writes the rescheduling message on it. | Same section. |
| GATE-M9-11 | The scheduler processes reordering rules in the declared order, creates the supplies, and reports on each demand it cannot satisfy with the exception message of the domain document. | Replenishment workflows, scheduler. |
| GATE-M9-12 | Running the scheduler twice without changing anything creates no second supply. | Same section. |
| GATE-M9-13 | The forecast report shows on-hand, incoming, outgoing and free-to-use quantities per period, and the numbers agree with the transfers behind them. | Replenishment interfaces. |
| GATE-M9-14 | A snoozed reordering rule is skipped until the snooze date and then processed normally. | Replenishment rules, snooze. |
| GATE-M9-15 | A rule that would create a supply of zero or of a negative quantity creates nothing. | Replenishment rules, guard. |
| GATE-M9-16 | One hundred percent of the replenishment acceptance criteria are executed and pass. | [`../domains/replenishment-and-procurement/acceptance-criteria.md`](../domains/replenishment-and-procurement/acceptance-criteria.md). |

---

## 13. M10: Procure to pay works

**Entry.** M6, M8, M9. **Scope.** Step 10.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M10-01 | A request for quotation takes the vendor's pricelist, currency, payment terms and fiscal position, and its lines take the vendor price of each product. | [`../domains/purchasing/acceptance-criteria.md`](../domains/purchasing/acceptance-criteria.md) `PUR-AC` creation section. |
| GATE-M10-02 | Confirming a purchase order sets the order date, numbers the document, creates the receipt with the scheduled date computed from the lead times, and writes the confirmation message. | `PUR-AC` confirmation section. |
| GATE-M10-03 | An order that exceeds the double validation threshold moves to the waiting-approval state and only a user of the approving group may approve it. | `PUR-AC` approval section. |
| GATE-M10-04 | Receiving five of ten ordered units sets the received quantity to five and leaves the order in the partially received state. | `PUR-AC` receipt section. |
| GATE-M10-05 | With the bill control policy "on ordered quantities", the draft bill proposes ten; with "on received quantities" it proposes five. | `PUR-AC` bill control section. |
| GATE-M10-06 | Billing more than received with the policy "on received quantities" produces the over-billing warning with the declared message. | `PUR-AC` over-billing section. |
| GATE-M10-07 | Three-way matching marks the order fully billed only when the billed quantity equals the received quantity for every line. | `PUR-AC` matching section. |
| GATE-M10-08 | A purchase order line whose product has a different purchase unit of measure converts the quantity to the stock unit on the receipt and back on the bill. | `PUR-AC` unit section. |
| GATE-M10-09 | Cancelling an order cancels its receipts when they are not done and is refused when a bill is posted. | `PUR-AC` cancellation section. |
| GATE-M10-10 | Locking an order prevents further edits and unlocking restores them, with both actions recorded. | `PUR-AC` locking section. |
| GATE-M10-11 | A purchase agreement of type blanket order carries the agreed prices to every order created from it, within its validity window and quantity limits. | `PUR-AC` agreement section. |
| GATE-M10-12 | A call for tenders compares the alternative quotations line by line and confirming one offers to cancel the others. | `PUR-AC` alternatives section. |
| GATE-M10-13 | A dropship order creates the direct move from the vendor to the customer and no warehouse stock is affected. | `PUR-AC` dropship section. |
| GATE-M10-14 | Posting the vendor bill of a received purchase with a different price posts the price difference according to the costing method and the accounts of step 8. | [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md) section 10, price difference at the vendor bill under standard price. |
| GATE-M10-15 | The vendor's currency is kept on the order and the bill, and the company amounts use the rate of each document's date. | `PUR-AC` currency section. |
| GATE-M10-16 | The purchase reminder job sends the reminder only once per order and records the vendor's answer on the order. | Build sequence step 10 job table. |
| GATE-M10-17 | The purchase order and request for quotation documents render the declared content including the vendor reference and the delivery address. | [`../domains/purchasing/interfaces.md`](../domains/purchasing/interfaces.md). |
| GATE-M10-18 | The purchase analysis report aggregates ordered, received and billed amounts per vendor, product and period and agrees with the documents. | Purchasing interfaces, analysis. |
| GATE-M10-19 | The vendor delay report computes the average delay between the promised and the effective receipt date. | Replenishment interfaces, vendor delay. |
| GATE-M10-20 | One hundred percent of the purchasing acceptance criteria are executed and pass. | [`../domains/purchasing/acceptance-criteria.md`](../domains/purchasing/acceptance-criteria.md). |

---

## 14. M11: Quote to cash works

**Entry.** M6, M7, M9. **Scope.** Step 11.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M11-01 | A quotation takes the customer's pricelist, payment terms, fiscal position, delivery address and invoice address, and the lines take the pricelist price. | [`../domains/sales/acceptance-criteria.md`](../domains/sales/acceptance-criteria.md) `SALE-AC` creation section. |
| GATE-M11-02 | Applying a quotation template replaces the lines with the template's lines, including sections, notes and optional lines, and keeps the customer's prices. | `SALE-AC` template section. |
| GATE-M11-03 | Confirming a quotation numbers the order, sets the order date, creates the delivery with the scheduled date from the lead times, and writes the confirmation message. | `SALE-AC` confirmation section. |
| GATE-M11-04 | With the delivery policy "as soon as possible", one delivery is created for the whole order; with "when all products are ready", the delivery waits for full availability. | `SALE-AC` delivery policy section. |
| GATE-M11-05 | With the invoicing policy "ordered quantities", the invoice proposes the ordered quantity; with "delivered quantities", it proposes the delivered quantity and nothing before delivery. | `SALE-AC` invoicing policy section. |
| GATE-M11-06 | A down payment of thirty percent on an order of 1 000.00 excluding tax creates a draft invoice of 300.00 excluding tax with the declared account and the taxes of the order lines, and the final invoice deducts it. | `SALE-AC` down payment section. |
| GATE-M11-07 | A fixed down payment of 250.00 behaves identically with the fixed amount. | Same section. |
| GATE-M11-08 | Applying a global discount of ten percent through the discount dialog writes the discount either as a percentage on every line or as a separate discount line, according to the chosen mode, and the totals match the declared formula. | `SALE-AC` discount section. |
| GATE-M11-09 | Changing the customer recomputes the pricelist prices, the fiscal position and the taxes of every line that the user has not manually overridden. | `SALE-AC` recomputation section. |
| GATE-M11-10 | The margin of a line is the sales price minus the cost, and the margin of the order is the sum of its lines' margins. | [`../domains/sales/calculations.md`](../domains/sales/calculations.md) margin section. |
| GATE-M11-11 | Cancelling an order cancels the deliveries that are not done, and is refused when an invoice is posted, with the declared message. | `SALE-AC` cancellation section. |
| GATE-M11-12 | Locking an order prevents edits; the setting "orders editable after confirmation" changes the default. | `SALE-AC` locking section. |
| GATE-M11-13 | A service product with the policy "based on timesheets" invoices only the hours logged, using the analytic link of step 16 once it exists. | `SALE-AC` service section. |
| GATE-M11-14 | An order signed online by the customer stores the signature and the signer's name, and confirms the order when the setting says so. | `SALE-AC` online signature section. |
| GATE-M11-15 | An order paid online confirms the order and creates the payment and the transaction link. | `SALE-AC` online payment section. |
| GATE-M11-16 | A loyalty program of type discount applied to a cart of 100.00 with a ten percent reward creates the reward line of −10.00 with the declared product and taxes. | [`../domains/loyalty-and-promotions/acceptance-criteria.md`](../domains/loyalty-and-promotions/acceptance-criteria.md) `AC-LOY` discount section. |
| GATE-M11-17 | A loyalty program of type points credits the points on order confirmation and debits them when the reward is claimed, with the history recorded. | `AC-LOY` points section. |
| GATE-M11-18 | A gift card is generated with a unique code, is redeemed once, and a second redemption is refused with the declared message. | `AC-LOY` gift card section. |
| GATE-M11-19 | A coupon with a validity window and a usage limit is refused outside the window and after the limit, with the declared messages. | `AC-LOY` coupon section. |
| GATE-M11-20 | Two programs that both apply are ordered by the declared priority and the domain's combination rule decides whether both apply. | `AC-LOY` combination section. |
| GATE-M11-21 | The quotation, order and pro-forma documents render the declared content, including optional lines and the signature block. | [`../domains/sales/interfaces.md`](../domains/sales/interfaces.md). |
| GATE-M11-22 | The sales analysis report aggregates ordered, delivered and invoiced amounts per salesperson, team, customer, product and period, and agrees with the documents. | Sales interfaces, analysis. |
| GATE-M11-23 | The automatic invoicing job invoices only the orders that meet its conditions and never invoices twice. | Build sequence step 11 job table. |
| GATE-M11-24 | An order whose company differs from its warehouse's company is refused. | `SALE-AC` company consistency section. |
| GATE-M11-25 | Deleting a confirmed order is refused; only a cancelled draft may be deleted. | `SALE-AC` deletion section. |
| GATE-M11-26 | One hundred percent of the sales and loyalty acceptance criteria are executed and pass. | The two domains' acceptance files. |

---

## 15. M12: Make to stock works

**Entry.** M8, M9. **Scope.** Step 12.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M12-01 | Exploding a bill of materials of one finished unit made of two components and one sub-assembly with its own bill produces the declared component quantities at each level, and a cycle in the structure is refused. | [`../domains/manufacturing/calculations.md`](../domains/manufacturing/calculations.md); manufacturing acceptance criteria section A. |
| GATE-M12-02 | A bill of materials with a quantity of five produces, for a manufacturing order of twelve, the component quantities rounded up to the component unit's rounding. | Manufacturing calculations, proportioning. |
| GATE-M12-03 | A kit bill of materials explodes on the sales order delivery and creates no manufacturing order. | Manufacturing acceptance criteria, kit section. |
| GATE-M12-04 | Confirming a manufacturing order creates the component moves and the finished move, reserves according to the operation type, and numbers the document. | Manufacturing workflows, confirmation. |
| GATE-M12-05 | Producing the full quantity consumes the reserved components, produces the finished quantity, and closes the order with the declared states. | Manufacturing workflows, production. |
| GATE-M12-06 | Producing less than planned offers the backorder, and the backorder carries the remaining quantity with its own components. | Manufacturing workflows, backorder. |
| GATE-M12-07 | Consuming more of a component than planned raises the consumption warning, and the user may confirm or correct it. | Manufacturing rules, consumption warning. |
| GATE-M12-08 | The valuation of the finished item equals the sum of the consumed component values plus the declared operation costs, to the currency step. | [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md) section 8.1, the cost of a finished good. |
| GATE-M12-09 | A by-product with a cost share of twenty percent receives twenty percent of the total value and the finished item the rest. | Manufacturing accounting effects, by-products. |
| GATE-M12-10 | Unbuilding a finished unit reverses the component and finished moves at the original values and is refused beyond the produced quantity. | Manufacturing workflows, unbuild. |
| GATE-M12-11 | Planning a manufacturing order schedules its work orders on the work centers respecting their capacity, working schedule and the declared dependencies. | [`../domains/manufacturing/workflows.md`](../domains/manufacturing/workflows.md). |
| GATE-M12-12 | Starting and finishing a work order records productivity with the declared durations, and a blocked work center records the loss reason. | Work orders rules. |
| GATE-M12-13 | The work in progress posting produces the declared items for started but unfinished orders and reverses them when the order closes. | Manufacturing accounting effects, work in progress. |
| GATE-M12-14 | A subcontracted component is received as a finished item from the subcontractor and the components sent are valued as declared. | [`../domains/manufacturing/workflows.md`](../domains/manufacturing/workflows.md). |
| GATE-M12-15 | Splitting a manufacturing order of ten into two of five splits components proportionally and keeps the link to the original. | Manufacturing workflows, split. |
| GATE-M12-16 | Merging two manufacturing orders of the same product and bill of materials sums the quantities and links the sources. | Manufacturing workflows, merge. |
| GATE-M12-17 | A repair order consumes parts, produces the repaired item, and invoices the declared lines when invoicing is requested. | [`../domains/repair-and-maintenance/acceptance-criteria.md`](../domains/repair-and-maintenance/acceptance-criteria.md) `AC-REP` section. |
| GATE-M12-18 | A maintenance request moves through its stages, computes the next preventive date from the declared frequency, and closes with the duration recorded. | `AC-MNT` section. |
| GATE-M12-19 | Equipment linked to an employee and to a work center reports its maintenance history and its downtime. | `AC-MNT` equipment section. |
| GATE-M12-20 | Every manufacturing printed document renders the declared content, including the component list and the operation list. | [`../domains/manufacturing/interfaces.md`](../domains/manufacturing/interfaces.md). |
| GATE-M12-21 | The bill of materials cost overview computes the rolled-up cost of each level and agrees with the product costs. | Manufacturing interfaces, overview. |
| GATE-M12-22 | One hundred percent of the manufacturing and repair acceptance criteria are executed and pass. | The two domains' acceptance files. |

---

## 16. M13: A counter day closes

**Entry.** M6, M7, M11. **Scope.** Step 13.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M13-01 | Creating a counter configuration creates its sequences and refuses to open without the declared journals and payment methods. | [`../domains/point-of-sale/acceptance-criteria.md`](../domains/point-of-sale/acceptance-criteria.md) `AC-001` onwards. |
| GATE-M13-02 | Opening a session records the opening cash balance, loads the declared data set, and refuses a second open session on the same counter. | Point of sale workflows, session. |
| GATE-M13-03 | An order created while the connection is down is stored locally with its own reference and is accepted on synchronization without renumbering. | [`../domains/point-of-sale/interfaces.md`](../domains/point-of-sale/interfaces.md) offline contract. |
| GATE-M13-04 | The same order synchronized twice is stored once. | Same contract. |
| GATE-M13-05 | An order of two lines with price-included taxes computes the same totals as the tax engine of step 5 for the same inputs. | Point of sale calculations, taxes. |
| GATE-M13-06 | A cash payment with change computes the change from the tendered amount and records both. | Point of sale workflows, payment. |
| GATE-M13-07 | A card payment through a terminal moves the payment through the declared states and a failure leaves the order payable. | [`../domains/point-of-sale/interfaces.md`](../domains/point-of-sale/interfaces.md). |
| GATE-M13-08 | Paying on a customer account posts to the declared receivable account and the balance is settled later by a payment. | Point of sale accounting effects, customer account. |
| GATE-M13-09 | Invoicing an order creates the customer invoice with the same amounts and links it to the order. | Point of sale workflows, invoicing. |
| GATE-M13-10 | Refunding an order creates the refund order with negative quantities and links both. | Point of sale workflows, refund. |
| GATE-M13-11 | Closing a session with a counted cash amount different from the theoretical amount posts the difference to the declared profit or loss account. | [`../domains/point-of-sale/accounting-effects.md`](../domains/point-of-sale/accounting-effects.md). |
| GATE-M13-12 | The session closing entry aggregates sales by account and tax, payments by method, and is balanced; invoiced orders are excluded from the sales aggregation. | Session closing accounting. |
| GATE-M13-13 | The session closing creates the stock moves of the sold goods and values them by the costing method of step 8. | Session closing accounting, inventory. |
| GATE-M13-14 | A session that fails to close leaves every order untouched and can be reopened as a rescue session. | Point of sale workflows, rescue. |
| GATE-M13-15 | A counter with a pricelist applies the pricelist price and the price is recomputed when the customer changes. | Point of sale rules, pricing. |
| GATE-M13-16 | A loyalty program applied at the counter produces the same reward lines as the same program on a sales order. | [`../domains/loyalty-and-promotions/workflows.md`](../domains/loyalty-and-promotions/workflows.md). |
| GATE-M13-17 | A restaurant order assigned to a table is transferred to another table with its lines and its payments untouched. | [`../domains/point-of-sale/workflows.md`](../domains/point-of-sale/workflows.md). |
| GATE-M13-18 | Splitting a bill creates the declared child orders whose totals sum to the original total. | Restaurant operations, splitting. |
| GATE-M13-19 | A course fired to the preparation display appears there once and is marked done once. | Restaurant operations, preparation. |
| GATE-M13-20 | A self-ordering session creates the order with the declared preset and the payment method the configuration allows. | [`../domains/point-of-sale/interfaces.md`](../domains/point-of-sale/interfaces.md). |
| GATE-M13-21 | The receipt renders the declared content, including the header, footer, tax summary and the quick response code when enabled. | Point of sale interfaces. |
| GATE-M13-22 | The sales details report totals by product, category, payment method and tax and agrees with the session entry. | Point of sale interfaces, sales details. |
| GATE-M13-23 | Selling a tracked product at the counter records the lot or serial number on the stock move. | Point of sale rules, tracking. |
| GATE-M13-24 | One hundred percent of the point of sale acceptance criteria are executed and pass. | [`../domains/point-of-sale/acceptance-criteria.md`](../domains/point-of-sale/acceptance-criteria.md). |

---

## 17. M14: Demand is tracked

**Entry.** M11. **Scope.** Step 14.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M14-01 | A lead requires a title and refuses to save without one, with the declared message. | [`../domains/customer-relationship-management/acceptance-criteria.md`](../domains/customer-relationship-management/acceptance-criteria.md) `LEAD-AC-001`. |
| GATE-M14-02 | Converting a lead to an opportunity keeps the discussion thread, creates or links the contact according to the chosen mode, and records the conversion. | `LEAD-AC` conversion section. |
| GATE-M14-03 | Merging opportunities keeps the oldest record, concatenates the descriptions, moves the activities and the messages, and deletes the others. | `LEAD-AC` merge section. |
| GATE-M14-04 | Marking an opportunity won sets the probability to one hundred and moves it to the won stage; marking it lost records the lost reason and keeps it out of the open pipeline. | `LEAD-AC` won and lost sections. |
| GATE-M14-05 | Restoring a lost opportunity returns it to its previous stage with its previous probability. | `LEAD-AC` restore section. |
| GATE-M14-06 | The automated probability is computed from the scoring frequencies of the declared fields and is not overwritten once a user sets the probability manually. | [`../domains/customer-relationship-management/calculations.md`](../domains/customer-relationship-management/calculations.md). |
| GATE-M14-07 | The scoring frequency table is rebuilt from won and lost leads and a lead whose values never occurred receives the declared default probability. | Predictive scoring rules. |
| GATE-M14-08 | Assignment distributes unassigned leads to the teams whose filter matches, then to the members, respecting each member's maximum and the declared round-robin order. | [`../domains/customer-relationship-management/workflows.md`](../domains/customer-relationship-management/workflows.md). |
| GATE-M14-09 | A lead already assigned is not reassigned by a later run. | Lead assignment rules. |
| GATE-M14-10 | Creating a quotation from an opportunity carries the customer, the team, the salesperson and the campaign fields, and the order back-links to the opportunity. | `LEAD-AC` quotation section. |
| GATE-M14-11 | The expected revenue of a stage-weighted pipeline equals the sum of each opportunity's expected revenue times its probability. | Customer relationship calculations. |
| GATE-M14-12 | A recurring revenue plan multiplies the recurring amount by the declared factor for the reported yearly value. | `LEAD-AC` recurring section. |
| GATE-M14-13 | The five scheduled jobs of the domain each perform their declared effect and are safe to run twice. | Build sequence step 14 job table. |
| GATE-M14-14 | One hundred percent of the customer relationship acceptance criteria are executed and pass. | [`../domains/customer-relationship-management/acceptance-criteria.md`](../domains/customer-relationship-management/acceptance-criteria.md). |

---

## 18. M15: People are managed

**Entry.** M2, M6. **Scope.** Step 15.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M15-01 | An employee is created with a work contact, a department, a job position and a working schedule, and the private fields are visible only to the declared groups. | [`../domains/human-resources-core/acceptance-criteria.md`](../domains/human-resources-core/acceptance-criteria.md) `HRC-AC` creation section. |
| GATE-M15-02 | An employee version with a future start date does not affect today's values, and the daily job activates it on its start date. | [`../domains/human-resources-core/entities.md`](../domains/human-resources-core/entities.md). |
| GATE-M15-03 | Two overlapping versions of the same employee are refused with the declared message. | Versions rules. |
| GATE-M15-04 | Registering a departure archives the employee on the declared date, closes the current version and keeps the historical records. | `HRC-AC` departure section. |
| GATE-M15-05 | A working schedule of forty hours over five days computes twenty-one point six seven working days for a full month according to the declared convention, and the number of working hours between two moments excludes non-working time and public holidays. | [`../domains/attendances-and-working-time/calculations.md`](../domains/attendances-and-working-time/calculations.md). |
| GATE-M15-06 | Checking in twice without checking out is refused with the declared message. | Attendance rules. |
| GATE-M15-07 | Overtime is computed per day against the schedule with the declared thresholds and rounding, and a manual adjustment overrides it with a trace. | Attendance calculations, overtime. |
| GATE-M15-08 | The automatic check-out job closes attendances longer than the configured maximum and records the automatic closure. | Build sequence step 15 job table. |
| GATE-M15-09 | A time off request for a period that overlaps an existing approved request is refused with the declared message. | [`../domains/time-off/acceptance-criteria.md`](../domains/time-off/acceptance-criteria.md) overlap section. |
| GATE-M15-10 | A half-day request on a morning consumes half a day of the allocation and appears as half a day in the calendar. | Time off calculations, duration. |
| GATE-M15-11 | Approving a request in a two-level approval type requires both approvals in the declared order. | Time off workflows, approval. |
| GATE-M15-12 | Refusing an approved request restores the allocation and notifies the employee. | Time off workflows, refusal. |
| GATE-M15-13 | An accrual plan that grants two days per month on the first of the month credits exactly two days on each first, prorates the first period according to the declared rule, and caps at the declared maximum. | [`../domains/time-off/calculations.md`](../domains/time-off/calculations.md). |
| GATE-M15-14 | The accrual job run twice on the same day credits once. | Accrual plan rules. |
| GATE-M15-15 | A time off that overlaps a mandatory working day is refused. | Time off rules, mandatory days. |
| GATE-M15-16 | Work entries are generated for a period without duplicates, an approved time off produces the time-off work entry type, and regenerating rewrites only the regenerated period. | [`../domains/work-entries/workflows.md`](../domains/work-entries/workflows.md). |
| GATE-M15-17 | Two work entries that overlap in time for the same employee are reported as a conflict and block the period's validation. | Work entries rules. |
| GATE-M15-18 | An expense of 121.00 including twenty-one percent tax paid by the employee posts a payable item of 121.00 to the employee, an expense item of 100.00 and a tax item of 21.00. | [`../domains/expenses/accounting-effects.md`](../domains/expenses/accounting-effects.md). |
| GATE-M15-19 | An expense paid by the company posts to the declared company account and creates no employee payable. | Expenses accounting effects. |
| GATE-M15-20 | Reimbursing an expense creates the payment, reconciles it with the employee payable and sets the expense to paid. | Expenses workflows, reimbursement. |
| GATE-M15-21 | Splitting an expense divides the amount and the taxes so that the parts sum exactly to the original. | Expenses calculations, split. |
| GATE-M15-22 | An applicant moves through the recruitment stages, and hiring creates the employee with the declared fields copied. | [`../domains/recruitment/workflows.md`](../domains/recruitment/workflows.md). |
| GATE-M15-23 | Refusing an applicant records the reason and sends the declared message. | Recruitment workflows, refusal. |
| GATE-M15-24 | A vehicle assignment log records the driver and the period, and two overlapping assignments of the same vehicle are refused. | [`../domains/fleet/business-rules.md`](../domains/fleet/business-rules.md). |
| GATE-M15-25 | A vehicle contract with a monthly recurring cost generates one cost entry per month by the daily job, without duplicates. | Build sequence step 15 job table. |
| GATE-M15-26 | A lunch order placed after the cut-off moment of its alert is refused with the declared message. | [`../domains/lunch-ordering/business-rules.md`](../domains/lunch-ordering/business-rules.md). |
| GATE-M15-27 | A challenge evaluates its goals on the declared frequency and awards the badge exactly once per reached goal. | [`../domains/learning-surveys-and-gamification/workflows.md`](../domains/learning-surveys-and-gamification/workflows.md). |
| GATE-M15-28 | One hundred percent of the acceptance criteria of the seven people domains are executed and pass. | The seven domains' acceptance files. |

---

## 19. M16: Work is measured

**Entry.** M11, M15. **Scope.** Step 16.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M16-01 | A project created from a template copies its stages, tasks and milestones with the declared date shift. | [`../domains/projects-and-tasks/workflows.md`](../domains/projects-and-tasks/workflows.md). |
| GATE-M16-02 | Moving a task to a folded stage marks it closed with the declared closing state, and reopening restores the previous stage. | Projects rules, stages. |
| GATE-M16-03 | A recurring task creates the next occurrence on the declared trigger and never creates two for the same period. | Projects workflows, recurrence. |
| GATE-M16-04 | A task with a dependency cannot start before its predecessor is closed when the setting requires it. | Projects rules, dependencies. |
| GATE-M16-05 | Sharing a project with a portal collaborator grants read access to exactly the declared records and no others. | Projects rules, sharing. |
| GATE-M16-06 | A milestone reached records the moment and appears on the linked sales order line. | Projects workflows, milestones. |
| GATE-M16-07 | Logging four hours on a task creates an analytic line on the project's analytic account with the employee's cost rate as the amount. | [`../domains/timesheets/calculations.md`](../domains/timesheets/calculations.md). |
| GATE-M16-08 | A timesheet line on a billable task with the "based on timesheets" policy increases the delivered quantity of the sales order line by the same hours, converted with the declared unit. | Timesheets workflows, billing. |
| GATE-M16-09 | Invoicing the order invoices exactly the unbilled hours and marks them billed; a later correction of the hours does not change the posted invoice. | Timesheets rules, invoicing. |
| GATE-M16-10 | The remaining hours of a task equal the planned hours minus the logged hours and never go below zero in the report. | Projects calculations. |
| GATE-M16-11 | Project profitability reports revenue, cost and margin from the analytic lines and agrees with the ledger. | Projects interfaces, profitability. |
| GATE-M16-12 | The burndown chart reports the open task count per day over the chosen period and agrees with the stage history. | Projects interfaces, burndown. |
| GATE-M16-13 | The timesheet grid aggregates by day and week and a grid edit writes one analytic line per cell. | Timesheets interfaces. |
| GATE-M16-14 | Deleting an employee with timesheets is refused and the transfer wizard reassigns them. | Timesheets rules, deletion. |
| GATE-M16-15 | The rating request job sends one request per task entering a rating stage and not more than the configured frequency. | Build sequence step 16 job table. |
| GATE-M16-16 | One hundred percent of the projects and timesheets acceptance criteria are executed and pass. | The two domains' acceptance files. |

---

## 20. M17: The business talks

**Entry.** M2. **Scope.** Step 17.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M17-01 | Creating a channel subscribes its members, and a member who leaves stops receiving its messages. | [`../domains/messaging-and-activities/workflows.md`](../domains/messaging-and-activities/workflows.md). |
| GATE-M17-02 | A message posted in a channel reaches every connected member through the notification bus within the declared delay, and a disconnected member receives it on reconnection. | [`../runtime/notification-bus.md`](../runtime/notification-bus.md). |
| GATE-M17-03 | A live chat conversation routes to an available operator by the declared rule and falls back to the declared message when none is available. | Live chat rules. |
| GATE-M17-04 | A chat bot script advances through its steps, branches on the visitor's answer and hands over to an operator at the declared step. | Live chat rules, chat bot. |
| GATE-M17-05 | Rating a conversation stores the rating and the comment on the conversation and on the operator's statistics. | Live chat rules, rating. |
| GATE-M17-06 | An incoming message to a known alias creates the declared record and posts the message on it; an unknown alias bounces with the declared message. | [`../domains/messaging-and-activities/workflows.md`](../domains/messaging-and-activities/workflows.md). |
| GATE-M17-07 | An incoming reply to a message identifier posts on the same thread rather than creating a new record. | Gateway rules, threading. |
| GATE-M17-08 | A loop between two aliases is detected and stopped by the declared guard. | Gateway rules, loops. |
| GATE-M17-09 | A blacklisted address receives no mass message and still receives transactional messages, exactly as the domain declares. | Messaging rules, blacklist. |
| GATE-M17-10 | A text message is queued, sent, and its delivery state recorded; a failure for lack of credit is reported with the declared message and does not lose the message. | [`../domains/messaging-and-activities/workflows.md`](../domains/messaging-and-activities/workflows.md). |
| GATE-M17-11 | A postal letter is queued, rendered, sent and its state recorded. | Postal mail rules. |
| GATE-M17-12 | A scheduled message is posted at its moment and can be cancelled before it. | Messaging rules, scheduled messages. |
| GATE-M17-13 | A mailing list subscription and unsubscription through the public page updates the membership and records the reason. | Messaging interfaces, mailing lists. |
| GATE-M17-14 | A moderated group holds a message until a moderator accepts it, and rejection notifies the author with the declared message. | Messaging rules, moderation. |
| GATE-M17-15 | A push notification reaches a registered device and an expired registration is removed. | Messaging rules, push. |
| GATE-M17-16 | The nine communication scheduled jobs each perform their declared effect. | Build sequence step 17 job table. |
| GATE-M17-17 | The live chat transcript renders the declared content. | Messaging interfaces. |
| GATE-M17-18 | One hundred percent of the messaging acceptance criteria are executed and pass. | [`../domains/messaging-and-activities/acceptance-criteria.md`](../domains/messaging-and-activities/acceptance-criteria.md). |

---

## 21. M18: The audience is reached

**Entry.** M17, M11. **Scope.** Step 18.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M18-01 | A mailing sent to a list of one thousand contacts creates one trace per recipient, excludes blacklisted and opted-out addresses, and reports the counts. | [`../domains/marketing-and-mass-mailing/workflows.md`](../domains/marketing-and-mass-mailing/workflows.md). |
| GATE-M18-02 | Opens, clicks, bounces and replies update the trace and the aggregated statistics. | Marketing rules, statistics. |
| GATE-M18-03 | A tracked link redirects to the target and records one click per visitor according to the declared deduplication. | Marketing rules, link tracker. |
| GATE-M18-04 | A split test sends the variants to the declared share of the audience and the winner to the rest at the declared moment. | Marketing workflows, split test. |
| GATE-M18-05 | A campaign advances a participant through its steps on the declared triggers and stops on the declared conditions. | Marketing workflows, campaigns. |
| GATE-M18-06 | An unsubscribe link removes the contact from the list and records the opt-out reason. | Marketing rules, unsubscribe. |
| GATE-M18-07 | An event created from a template copies its tickets, communications and questions. | [`../domains/events/workflows.md`](../domains/events/workflows.md). |
| GATE-M18-08 | Registering an attendee to a ticket decreases the remaining seats and is refused when the ticket is sold out, with the declared message. | Events rules, seats. |
| GATE-M18-09 | Selling an event ticket on a sales order creates the registration on confirmation and cancels it when the order is cancelled. | Events rules, sales link. |
| GATE-M18-10 | Scanning a badge marks the attendee as attended once and reports a second scan as already attended. | Events workflows, attendance. |
| GATE-M18-11 | The event mail scheduler sends each communication once at its declared offset. | Build sequence step 18 job table. |
| GATE-M18-12 | A booth reservation creates the sales order line and marks the booth unavailable. | Events workflows, booths. |
| GATE-M18-13 | A survey with a scored question computes the score, applies the passing threshold and issues the certification when passed. | [`../domains/learning-surveys-and-gamification/workflows.md`](../domains/learning-surveys-and-gamification/workflows.md). |
| GATE-M18-14 | A survey with a time limit refuses answers after the limit and records the partial participation. | Survey rules, timing. |
| GATE-M18-15 | A survey with randomized questions selects the declared number per section and never repeats one in the same participation. | Survey rules, randomization. |
| GATE-M18-16 | Course progress advances when a content is completed and the course is marked completed at one hundred percent. | Course rules, progress. |
| GATE-M18-17 | A forum answer accepted by the author grants the declared karma and marks the question solved. | Forum rules, karma. |
| GATE-M18-18 | Every event and survey printed document renders the declared content, including the badge and the certificate. | The domains' interfaces files. |
| GATE-M18-19 | The three marketing and event jobs each perform their declared effect and are safe to run twice. | Build sequence step 18 job table. |
| GATE-M18-20 | One hundred percent of the marketing, events and survey acceptance criteria are executed and pass. | The three domains' acceptance files. |

---

## 22. M19: The public face works

**Entry.** M11, M6, M17. **Scope.** Step 19.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M19-01 | A page is created, edited, published and unpublished, and an unpublished page is invisible to a public visitor and visible to an editor. | [`../domains/website-and-storefront/workflows.md`](../domains/website-and-storefront/workflows.md). |
| GATE-M19-02 | A menu entry points at a page or an address, is ordered by its sequence, and a child entry renders under its parent. | Website rules, menus. |
| GATE-M19-03 | A record page (a product, a blog post, an event) is reachable at its declared address pattern and returns the declared not-found behavior when unpublished. | Website rules, record pages. |
| GATE-M19-04 | Search returns the declared record kinds ordered by the declared relevance rule. | Website rules, search. |
| GATE-M19-05 | A visitor is tracked with the declared identifier, and the tracking is removed by the cleanup job after the retention period. | Website rules, visitors. |
| GATE-M19-06 | The storefront lists only products published for that site, in the declared category, with the price of the visitor's pricelist and the declared tax display. | [`../domains/website-and-storefront/entities.md`](../domains/website-and-storefront/entities.md). |
| GATE-M19-07 | Adding a product to the cart creates the draft sales order, and adding it again increases the line rather than adding a second line. | [`../domains/website-and-storefront/workflows.md`](../domains/website-and-storefront/workflows.md). |
| GATE-M19-08 | Changing the delivery address recomputes the fiscal position, the taxes and the delivery price of the cart. | Checkout rules, address. |
| GATE-M19-09 | A product out of stock is refused or allowed in the cart according to the declared setting, with the declared message. | [`../domains/website-and-storefront/business-rules.md`](../domains/website-and-storefront/business-rules.md). |
| GATE-M19-10 | Selecting a delivery method adds the delivery line with the rated price, and changing the cart recomputes it. | Checkout rules, delivery. |
| GATE-M19-11 | Applying a coupon on the cart produces the same reward lines as the same coupon on a sales order. | [`../domains/loyalty-and-promotions/workflows.md`](../domains/loyalty-and-promotions/workflows.md). |
| GATE-M19-12 | Paying the cart confirms the order, creates the transaction and the payment, and empties the cart. | Checkout rules, payment. |
| GATE-M19-13 | An abandoned cart is detected after the declared delay and the recovery message is sent once. | Build sequence step 19 job table. |
| GATE-M19-14 | A back-in-stock subscription notifies the subscriber once when the quantity becomes positive. | Same job table. |
| GATE-M19-15 | A wishlist and a product comparison keep their content per visitor and per signed-in customer as declared. | [`../domains/website-and-storefront/workflows.md`](../domains/website-and-storefront/workflows.md). |
| GATE-M19-16 | A portal user sees exactly their own documents and no others, proved on invoices, orders, deliveries, projects and tickets. | [`../domains/customer-portal/business-rules.md`](../domains/customer-portal/business-rules.md). |
| GATE-M19-17 | A document shared by token is readable with the token and refused without it, and the token can be revoked. | Portal rules, tokens. |
| GATE-M19-18 | Signing a quotation in the portal stores the signature and confirms the order when configured. | Portal workflows, signature. |
| GATE-M19-19 | Paying an invoice in the portal creates the transaction for exactly the residual and reconciles the payment. | Portal workflows, payment. |
| GATE-M19-20 | A portal user granted access by the access wizard receives the invitation with a working link and the access appears in the document's follower list. | Portal workflows, access. |
| GATE-M19-21 | A second website with its own domain, menus, pages and pricelist serves its own content without leaking the first site's records. | Website rules, multi-site. |
| GATE-M19-22 | A page's search engine metadata renders the declared tags and the sitemap lists exactly the published pages. | Website rules, metadata. |
| GATE-M19-23 | The four website and storefront jobs each perform their declared effect. | Build sequence step 19 job table. |
| GATE-M19-24 | One hundred percent of the website, storefront and portal acceptance criteria are executed and pass. | The three domains' acceptance files. |

---

## 23. M20: The platform is complete

**Entry.** every earlier milestone. **Scope.** Step 20.

| Gate | Check | Evidence |
|---|---|---|
| GATE-M20-01 | A spreadsheet inserted with a pivot over journal items recomputes its values when the underlying records change and keeps the user's formatting. | [`../domains/spreadsheets-and-dashboards/workflows.md`](../domains/spreadsheets-and-dashboards/workflows.md). |
| GATE-M20-02 | A dashboard is shared by link and the shared copy is read-only and frozen at the declared moment. | Spreadsheet rules, sharing. |
| GATE-M20-03 | A spreadsheet function that reads a balance returns the same number as the general ledger report for the same filters. | Spreadsheet rules, functions. |
| GATE-M20-04 | An automation rule triggered on creation runs its actions once per created record, in the declared order, and a rule that modifies the same record does not re-trigger itself. | [`../domains/automation-and-integration/business-rules.md`](../domains/automation-and-integration/business-rules.md). |
| GATE-M20-05 | A time-based automation rule fires once per record per window and records the last execution. | Automation rules, timing. |
| GATE-M20-06 | A server action of each supported kind (write a field, create a record, send a message, run several actions) performs exactly the declared effect. | Automation rules, actions. |
| GATE-M20-07 | A data recycling rule proposes duplicates by the declared similarity and merging keeps the declared survivor. | Automation rules, recycling. |
| GATE-M20-08 | A privacy lookup finds every record that references a given address and the anonymization removes it from all of them. | Automation rules, privacy. |
| GATE-M20-09 | Importing a file maps columns to fields, resolves relations by name or by external identifier, reports every rejected row with its reason and imports nothing when the run is aborted. | Automation interfaces, import. |
| GATE-M20-10 | Generating an electronic invoice in each supported format produces a document that validates against that format's rules and carries every mandatory element. | [`../domains/electronic-invoicing-and-document-exchange/business-rules.md`](../domains/electronic-invoicing-and-document-exchange/business-rules.md). |
| GATE-M20-11 | Importing a received electronic invoice creates the draft vendor bill with the vendor, the lines, the taxes and the totals of the document, and reports a mismatch rather than silently adjusting. | Electronic invoicing workflows, import. |
| GATE-M20-12 | Sending a document to the exchange network records the acknowledgement, the business response and the final state, and a rejection is reported with its reason code and text. | Electronic invoicing workflows, network. |
| GATE-M20-13 | Registering a participant on the network moves through the declared states and a failed registration leaves the company able to retry. | Electronic invoicing workflows, registration. |
| GATE-M20-14 | Loading a country package creates its chart of accounts, taxes, tax groups, fiscal positions, tax report structure and document rules with their stable identifiers. | [`../domains/fiscal-localizations/README.md`](../domains/fiscal-localizations/README.md) and its country material. |
| GATE-M20-15 | For each of the ten most-used country packages, an invoice with the country's standard rate posts the amounts and the tax grids the country file declares. | The country material of [`../domains/fiscal-localizations/`](../domains/fiscal-localizations/). |
| GATE-M20-16 | A country's periodic declaration job produces the declaration for the period with the declared totals and marks the period submitted. | Build sequence step 20 job table. |
| GATE-M20-17 | A country package that requires document inalterability refuses to modify a posted document and produces the integrity statement on demand. | Localization rules, inalterability. |
| GATE-M20-18 | Activating an industry blueprint installs exactly the capability packages it declares, seeds exactly the master data it declares, and leaves the system in a state where the vertical's first document can be created without further configuration. | [build sequence](build-sequence.md), step 20. |
| GATE-M20-19 | Activating a blueprint twice does not duplicate its master data. | [build sequence](build-sequence.md), step 20. |
| GATE-M20-20 | The nine exchange and automation jobs each perform their declared effect and are safe to run twice. | Build sequence step 20 job table. |
| GATE-M20-21 | Every gate of milestones M1 to M19 still passes on the complete system. | This document. |
| GATE-M20-22 | One hundred percent of the acceptance criteria of every domain are executed and pass. | Every domain's acceptance file. |

---

## 24. MX1: Access control is complete

Re-checked at every milestone, on the entities delivered so far.

| Gate | Check | Evidence |
|---|---|---|
| GATE-MX1-01 | Every entity has at least one access rule, and an entity with none is refused at load time. | [`../../schemas/operational/access-rights.json`](../../schemas/operational/access-rights.json). |
| GATE-MX1-02 | The access matrix exported from the system equals the shipped matrix entity by entity, group by group, for create, read, update and delete. | Same catalog. |
| GATE-MX1-03 | Every record rule shipped for an entity is enforced on read, write and delete as its declared scope says, and a global rule is enforced for every user including administrators when the rule declares it. | [`../../schemas/operational/record-rules.json`](../../schemas/operational/record-rules.json). |
| GATE-MX1-04 | A portal user reaching a record they do not own receives the declared refusal, never the record. | [`../domains/customer-portal/business-rules.md`](../domains/customer-portal/business-rules.md). |
| GATE-MX1-05 | A field restricted to a group is absent from the read result of a user outside that group and a write to it is refused. | [`../overview/security-model.md`](../overview/security-model.md). |
| GATE-MX1-06 | Company scoping restricts every query to the user's allowed companies and a cross-company relation is refused with the declared message. | [`../overview/multi-company.md`](../overview/multi-company.md). |
| GATE-MX1-07 | An operation performed by a scheduled job runs with the declared identity and its record rules, not with unrestricted access, unless the job declares otherwise. | [`../runtime/scheduled-jobs.md`](../runtime/scheduled-jobs.md). |
| GATE-MX1-08 | Granting and revoking a group takes effect on the next request without a restart. | [`../domains/identity-and-access/workflows.md`](../domains/identity-and-access/workflows.md). |

---

## 25. MX2: Concurrency is safe

| Gate | Check | Evidence |
|---|---|---|
| GATE-MX2-01 | Two simultaneous postings in the same journal and period produce two different consecutive numbers and no gap. | [`../runtime/transactions-and-concurrency.md`](../runtime/transactions-and-concurrency.md); `BOOK-AC` numbering section. |
| GATE-MX2-02 | Two simultaneous reservations of the last available unit result in exactly one reservation; the other waits or fails with the declared message and leaves no partial write. | `INV-AC` concurrency section. |
| GATE-MX2-03 | Two simultaneous validations of the same transfer result in one validation; the second reports that the transfer is already done. | Same section. |
| GATE-MX2-04 | Two simultaneous payments against the same invoice never over-reconcile it; the residual never goes below zero without an explicit credit. | `PAY-AC` concurrency section. |
| GATE-MX2-05 | Two counters closing at the same moment produce two independent entries and no shared number. | Point of sale session closing. |
| GATE-MX2-06 | A failed operation leaves no partial record, no partial entry and no partial reservation. | [`../runtime/transactions-and-concurrency.md`](../runtime/transactions-and-concurrency.md). |
| GATE-MX2-07 | A retried operation after a serialization failure produces exactly one effect, not two. | Same document. |

---

## 26. MX3: Numbers never drift

| Gate | Check | Evidence |
|---|---|---|
| GATE-MX3-01 | The sum of the line amounts of every posted document equals the document total to the currency step, for every document type delivered so far, checked over the whole fixture. | [equivalence test plan](equivalence-test-plan.md), the invariants of section 18. |
| GATE-MX3-02 | The sum of debits equals the sum of credits for every posted entry and for the whole ledger. | Same section. |
| GATE-MX3-03 | The sum of the analytic line amounts of an item equals the item amount. | Same section. |
| GATE-MX3-04 | The stock valuation account balance equals the sum of the layer values, for every product under automated valuation. | Same section. |
| GATE-MX3-05 | The residual of an invoice equals its total minus the sum of its reconciled amounts, at every moment. | Same section. |
| GATE-MX3-06 | No computation rounds an intermediate value that the domain document says is kept unrounded, proved by the worked examples that distinguish the two. | [`../domains/taxes/calculations.md`](../domains/taxes/calculations.md), [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md). |

---

## 27. MX4: Performance is acceptable

Performance is a behavior of the platform, not an infrastructure concern: the gates below are expressed in work performed, not in hardware.

| Gate | Check | Evidence |
|---|---|---|
| GATE-MX4-01 | Listing eighty records of any entity issues a number of queries independent of the number of records, and reading a relation for a list of records does not issue one query per record. | [`../runtime/caching.md`](../runtime/caching.md). |
| GATE-MX4-02 | A grouped read over one million journal items returns within the declared budget and does not load the items into memory. | [`../runtime/`](../runtime/) performance notes. |
| GATE-MX4-03 | A filter on a searchable derived field is translated into a query on its declared searchable expression and never into a scan of every record. | [`../references/computed-fields.md`](../references/computed-fields.md). |
| GATE-MX4-04 | Posting an entry of one thousand items performs a bounded number of writes and recomputations, not one per item per field. | [`../runtime/transactions-and-concurrency.md`](../runtime/transactions-and-concurrency.md). |
| GATE-MX4-05 | Rendering a printed document of one hundred pages streams and does not hold the whole rendering in memory. | [`../runtime/report-rendering.md`](../runtime/report-rendering.md). |
| GATE-MX4-06 | The procurement scheduler over one hundred thousand reordering rules processes them in bounded batches and can resume after an interruption without repeating an effect. | [`../domains/replenishment-and-procurement/workflows.md`](../domains/replenishment-and-procurement/workflows.md). |

---

## 28. Gate summary by evidence kind

| Evidence kind | Gates | Where the detail lives |
|---|---:|---|
| Record dumps after an operation | 96 | Domain `workflows.md` files. |
| Balanced entry listings | 41 | Domain `accounting-effects.md` files. |
| Refusal transcripts with exact messages | 63 | Domain `business-rules.md` files and [`../references/validation-messages.md`](../references/validation-messages.md). |
| Rendered documents | 24 | Domain `interfaces.md` files. |
| Timed job runs | 38 | Build sequence job tables and [`../references/scheduled-jobs.md`](../references/scheduled-jobs.md). |
| Access matrices | 8 | [`../../schemas/operational/access-rights.json`](../../schemas/operational/access-rights.json) and [`../../schemas/operational/record-rules.json`](../../schemas/operational/record-rules.json). |
| Numeric invariants over the whole fixture | 21 | The twenty-one invariants INV-01 to INV-21 of section 18 of the [equivalence test plan](equivalence-test-plan.md). |

---

## 29. The ten stage gates

A stage gate closes a stage of the [build sequence](build-sequence.md). Its rows are properties that must hold across every step of the stage, not checks of one delivery, and each row names the layer of the [equivalence test plan](equivalence-test-plan.md) that proves it. A stage gate is checked when the last step of its stage closes, and re-checked at every later stage gate.

### Stage gate one: the platform holds

Closes stage one, delivered by the platform part of step 1.

| # | The gate | Proved by |
|---|---|---|
| 1.1 | An entity can be defined with stored, derived, related, company-dependent and translatable fields, and all of them read back correctly. | Layer two |
| 1.2 | Writing a stored field causes every derived field that depends on it, directly or through a relation, to return the new value on the next read, without an explicit instruction to recalculate. | Layer one |
| 1.3 | A derived field that is stored is written to storage with the same value a non-stored one would return. | Layer two |
| 1.4 | A second package extends an existing entity with a new field and overrides an operation, and the override can invoke the previous definition. | Layer two |
| 1.5 | Prototype copying and delegation embedding both work, and an embedded parent's fields are readable and writable through the child. | Layer two |
| 1.6 | The filter notation returns the specified records for every operator, including traversal through a relation and through a hierarchy. | Layer one |
| 1.7 | A failed validation rolls back every change in the same unit of work, leaving no partial effect. | Layer five |
| 1.8 | Two concurrent writes to the same record resolve as specified, with the loser retried automatically. | Layer eight |
| 1.9 | Shipped data loads by external identifier, and reloading it updates the records it owns and leaves records marked as not updatable alone. | Layer two |
| 1.10 | A translatable field returns the value for the active language and falls back as specified when that language has no value. | Layer one |
| 1.11 | A numbering sequence produces the specified format, resets per period as configured, and pads as specified. | Layer one |

**Do not proceed** until 1.2 holds under a dependency that crosses two relations. It is the single most common place a rebuild silently diverges, and every later derived total depends on it.

### Stage gate two: access is enforced

Closes stage two, delivered by the identity and access part of step 1.

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

### Stage gate three: clients can work

Closes stage three, delivered by the presentation and transport part of step 1.

| # | The gate | Proved by |
|---|---|---|
| 3.1 | A client obtains the menu tree filtered by the user's groups. | Layer six |
| 3.2 | Opening a window action returns the specified views, filter and context. | Layer six |
| 3.3 | View inheritance applies extensions in priority order and the specified node-matching grammar resolves to the specified result. | Layer two |
| 3.4 | Every generic entity operation accepts the specified arguments and returns the specified shape, including grouped reading with aggregation. | Layer six |
| 3.5 | On-change evaluation returns the specified changed values and warnings without persisting anything. | Layer four |
| 3.6 | An error returns the specified envelope with the specified kind. | Layer six |
| 3.7 | An attachment uploads, deduplicates by content, downloads, and is refused to a user who cannot read its record. | Layer seven |
| 3.8 | A report renders to a printable document containing the specified sections, and re-rendering an already-stored document reuses it where the specification says so. | Layer eleven |

### Stage gate four: the system communicates

Closes stage four, delivered by step 2 for the substrate and step 17 for the communication application.

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

### Stage gate five: master data is exact

Closes stage five, delivered by steps 2 and 3.

| # | The gate | Proved by |
|---|---|---|
| 5.1 | Every shipped reference record loads with its specified values, and the counts match the catalog. | Layer two |
| 5.2 | Unit conversion is exact in both directions for every worked example, with the specified rounding. | Layer one |
| 5.3 | A quantity expressed in a packaging converts to the base unit and back with no drift. | Layer one |
| 5.4 | Currency conversion uses the rate in force on the stated date and rounds to the target currency's precision. | Layer one |
| 5.5 | The rounding primitives match every case, including a value exactly on a rounding boundary. | Layer one |
| 5.6 | Variant generation produces exactly the specified combinations, honors exclusions, and prices each with its extra amount. | Layer four |
| 5.7 | The price computation returns the specified price for every worked example, including a rule based on another price list in a different currency. | Layer one |
| 5.8 | A party's commercial parent resolves across three levels and the specified fields synchronize to children. | Layer four |

**Do not proceed** until 5.2 and 5.5 hold exactly. Every amount and quantity in the remaining stages inherits this arithmetic.

### Stage gate six: the money is right

Closes stage six, delivered by steps 4, 5 and 6, with its country and exchange part re-checked when step 20 closes. This is the most consequential gate in the plan.

| # | The gate | Proved by |
|---|---|---|
| 6.1 | Every posted entry balances, per entry and per currency, with no exception. | Layer five |
| 6.2 | Posting assigns a number in the specified format, with gaps only where permitted, and resequencing behaves as specified. | Layer three |
| 6.3 | Every lock date refuses a posting inside the locked period with the specified message, and an exception permits exactly what it grants. | Layer three |
| 6.4 | The tax engine reproduces every worked example: percentage, fixed, division and group taxes; price-included extraction; base-amount chaining; per-line against global rounding on the three-line case, to the cent. | Layer one |
| 6.5 | A fiscal position substitutes taxes and accounts as specified on a document and on its items. | Layer four |
| 6.6 | An invoice's term lines distribute the total exactly, with the remainder on the last instalment, for every worked example. | Layer one |
| 6.7 | An early payment discount computes identically under all three modes, and taking it produces the specified entry. | Layer ten |
| 6.8 | Cash rounding produces the specified line under both strategies and all rounding methods. | Layer one |
| 6.9 | Reconciling two items produces the specified partial records and leaves the specified residual in both currencies. | Layer one |
| 6.10 | A payment across a rate change produces the specified exchange difference on the specified accounts, and unreconciling reverses it. | Layer ten |
| 6.11 | Deferred tax exigibility produces its entry at reconciliation, proportional to the amount settled, with the last part absorbing the rounding. | Layer one |
| 6.12 | An analytic distribution produces lines whose signed amounts sum to the originating item. | Layer five |
| 6.13 | The balance sheet balances and the profit and loss statement ties to the ledger for the seeded data. | Layer five |
| 6.14 | Reversing a document produces the exact opposite entry and reconciles it as specified. | Layer ten |
| 6.15 | A tax period closes with the specified entry and the grids agree with the item tags. | Layer four |

### Stage gate seven: the goods are right

Closes stage seven, delivered by steps 7, 8, 9, 10 and 12.

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
| 7.9 | Deferred cost recognition produces the specified entries at invoicing for a partly delivered order. | Layer ten |
| 7.10 | A reordering rule proposes the specified quantity, rounded up to its multiple. | Layer one |
| 7.11 | The scheduler groups procurements into the specified number of documents with the specified lines. | Layer four |
| 7.12 | Lead times produce the specified dates for a pick, pack and ship chain and for a purchase. | Layer one |
| 7.13 | A bill of materials explodes with correct unit conversion, and a cycle is refused with the specified message. | Layer three |
| 7.14 | A production consumes, produces and values as specified, including by-product cost shares and a backorder. | Layer four |

### Stage gate eight: selling works in every channel

Closes stage eight, delivered by steps 11, 13, 14 and 19.

| # | The gate | Proved by |
|---|---|---|
| 8.1 | Confirming an order creates exactly the specified subsequent records and sets the specified statuses. | Layer four |
| 8.2 | Invoicing on ordered and on delivered quantities each produce the specified lines and quantities. | Layer four |
| 8.3 | An advance invoice deducts exactly on the final invoice, with the specified line and tax treatment. | Layer ten |
| 8.4 | The counter application's price, tax, discount and rounding results match the server's for every worked example. | Layer one |
| 8.5 | A counter session closes with the specified entry, line by line, including the cash difference and the excluded invoiced orders. | Layer ten |
| 8.6 | Orders created while the counter is offline synchronize once, with no duplicate on replay. | Layer eight |
| 8.7 | A storefront checkout reserves, authorizes a payment, confirms the order and produces the specified documents. | Layer four |
| 8.8 | A duplicate payment notification is processed once. | Layer eight |
| 8.9 | A promotion and a gift card on one order produce the specified reward lines, prices and tax treatment. | Layer four |
| 8.10 | Predictive probability reproduces the specified value from the stated frequency counts. | Layer one |

### Stage gate nine: people, time and services

Closes stage nine, delivered by steps 15 and 16.

| # | The gate | Proved by |
|---|---|---|
| 9.1 | Working time yields the specified hours and days across a period containing a public holiday, in a named time zone. | Layer one |
| 9.2 | An alternating two-week schedule resolves to the specified week on a stated date. | Layer one |
| 9.3 | An absence request in days, half days and hours each computes the specified duration and reduces working time accordingly. | Layer four |
| 9.4 | An accrual produces the specified balance across multiple periods, including proration and carry-over. | Layer one |
| 9.5 | Work entry generation produces exactly the specified entries for a two-week period, with a validated absence overlaying correctly and a conflict raised for an overlap. | Layer four |
| 9.6 | Overtime computes with the specified thresholds and absorbs small overruns as specified. | Layer one |
| 9.7 | An expense report posts the specified entry for each payment mode and reaches the specified settled state. | Layer ten |
| 9.8 | A timesheet line costs and bills as specified, and a validated line refuses edits with the specified message. | Layer three |
| 9.9 | Project visibility rules admit and refuse exactly the specified users for each privacy setting. | Layer seven |

### Stage gate ten: the remaining capabilities

Closes stage ten, delivered by step 18 and the spreadsheet and automation part of step 20; the calendar part of the stage is delivered in step 2 and is checked from milestone M2 onward.

| # | The gate | Proved by |
|---|---|---|
| 10.1 | A recurrence generates the specified occurrences; detaching one edits it alone; the three scopes of edit behave as specified. | Layer four |
| 10.2 | Event seat availability and the communication schedule match the specified values. | Layer one |
| 10.3 | A mailing sends to the specified recipients, suppresses the specified ones, and the statistics match the stated counts. | Layer one |
| 10.4 | A questionnaire scores as specified, including partial credit, conditional display and attempt limits. | Layer one |
| 10.5 | Two spreadsheet editors changing different cells converge to the same document, and a reconnecting editor catches up. | Layer eight |
| 10.6 | An automation rule fires on exactly the specified condition, does not retrigger itself, and its time-based form fires at the specified offset. | Layer four |
| 10.7 | An import of two hundred rows with five invalid reports exactly those five and commits the rest, or none, according to the specified mode. | Layer four |

---

## 30. Closing the program

The rebuild is complete when every step milestone and every stage gate is closed at the conformance level claimed in [conformance profiles](conformance-profiles.md), when the invariants of layer five hold continuously across the whole suite, and when [coverage and evidence](coverage-and-evidence.md) shows no artifact in the specified column without an entry in either the verified column or the acknowledged-difference list.

