# Design principles

The system's behaviour is not an accumulation of independent decisions. Ten recurring choices explain most of it, and a rebuild that preserves the tables and the screens but abandons these choices will diverge — not cosmetically, but in what it records, what it refuses, what it can be extended with, and what it costs to run.

Each principle below is stated, then traced through the mechanisms that implement it, then paid for: **every one of them buys something at a price**, and the price is as much part of the specification as the benefit. A rebuild that accepts the benefits and refuses the costs will produce a system that behaves differently under load, under extension, or under audit.

The ten:

1. [Everything is an entity](#1-everything-is-an-entity)
2. [The interface is data](#2-the-interface-is-data)
3. [Configuration is data](#3-configuration-is-data)
4. [External identifiers make data reloadable](#4-external-identifiers-make-data-reloadable)
5. [Archival over deletion](#5-archival-over-deletion)
6. [Audit fields everywhere](#6-audit-fields-everywhere)
7. [Company scope per record](#7-company-scope-per-record)
8. [Strict currency and precision discipline](#8-strict-currency-and-precision-discipline)
9. [Extension over modification](#9-extension-over-modification)
10. [Declared dependencies over manual invalidation](#10-declared-dependencies-over-manual-invalidation)

Four closing sections follow: [when the principles conflict](#11-when-the-principles-conflict) states how the system resolves the cases where two of them pull in opposite directions; [the measured footprint](#12-the-measured-footprint-of-each-principle) gives the counts that show each principle is actually applied rather than merely intended; [signals of divergence](#13-signals-that-a-rebuild-has-diverged) lists the observable symptoms of a rebuild that abandoned one; [principles deliberately not adopted](#14-principles-deliberately-not-adopted) states what the system is **not**, because a rebuild that adds one of them will also diverge; and [the acceptance criteria](#15-acceptance-criteria) check a rebuild has actually adopted them.

---

## 1. Everything is an entity

### 1.1 The statement

There is one kind of persistent thing in the system: a **record of an entity**. A customer invoice, a chart-of-accounts line, a screen definition, a menu, an access right, a scheduled job, a translation, a message, a user preference and the list of installed capability packages are all records of entities, stored in the same database, reached through the same operations, subject to the same access control, the same transactions, the same import and export, and the same extension mechanism.

There is no separate metadata store, no separate configuration file format read at run time, no separate presentation repository, and no privileged class of object that escapes the entity layer.

### 1.2 How it shows up

| Thing that in most systems is not data | Here it is a record of |
|---|---|
| The list of entities and their fields | The entity catalogue and the field catalogue |
| A screen | The View entity |
| A navigation item | The Menu entity |
| What a button does | One of the six action entities |
| Who may do what | The Access Right and Record Rule entities |
| A printable document's definition | The Report Action entity, naming a template that is itself a View record |
| A periodic task | The Scheduled Job entity, whose body is a Server Action record |
| A numbering scheme | The Numbering Sequence entity |
| A translation | A per-language entry inside the translated field's own column |
| The set of installed capabilities | The Capability Package entity |
| The correspondence between a stable name and a record | The External Identifier entity |
| A user's saved search | The Saved Filter entity |
| A message and who was notified | The Message and Notification entities |

### 1.3 What it buys

1. **Capabilities written once work on everything.** Search, grouping, import, export, access control, translation, audit fields, archival, the remote transport and the extension mechanism are written against the entity layer. Adding a new kind of thing costs nothing in those capabilities.
2. **Administration is ordinary use.** An administrator edits an access right with the same screen machinery a salesperson edits an order with — because the access right *is* a record with views.
3. **A tenant is one backup.** Everything that makes the installation what it is lives in one database plus one file store.
4. **Introspection is free.** The system can answer "what entities exist, with what fields, contributed by what package" because the answer is in tables.
5. **The remote transport is uniform.** A client that can create, read, update, delete and search can drive every part of the system, including its configuration.

### 1.4 The mechanics

- Every entity is composed at registry build time from the contributions of installed packages ([architecture, section 5](architecture.md#5-how-installed-packages-build-the-registry)).
- The self-describing entities — the entity catalogue, the field catalogue, the selection-value catalogue, the constraint catalogue, the relation catalogue — are populated from the registry during installation, so the data about the entities is derived from the entities.
- Presentation records are ordinary records with ordinary extension ([views and actions](views-and-actions.md), [inheritance and extension](inheritance-and-extension.md)).

### 1.5 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| Bootstrapping is intricate | The entity catalogue is an entity. The registry must be able to build itself before its own description exists, which is why the foundation package is loaded alone in the first phase and why schema synchronisation runs before data loading. |
| Configuration changes are transactional and therefore contended | Changing a screen or an access right takes the ordinary write path, invalidates caches, and must be signalled to every worker. A configuration change is not a file edit. |
| Metadata volume is real data volume | A full installation holds thousands of view, action, menu and access records. They are rows, they are indexed, they are backed up, and they are read on every request until cached. |
| Everything is reachable, so everything must be protected | Because configuration is data, forgetting an access right on a configuration entity exposes it. This is why the build warns about entities with no access right. |
| A self-referential failure is hard to diagnose | A broken view of the view entity makes the view screen unusable. The system mitigates this with a default view, a previous-description field and a hard reset from the shipped file — all of which a rebuild needs too. |

**What a rebuild must not do instead.** Keeping screens in source files, access rules in a configuration file, or the entity description in code annotations read only at start-up. Each of those breaks extension by third-party packages, breaks per-tenant customisation, and breaks the uniform transport.

---

## 2. The interface is data

### 2.1 The statement

A client never receives rendered markup for a business screen. It receives a **declarative description** — a tree of fields, buttons, groupings and layout containers — plus the metadata of the fields named. The client renders.

The grammar is closed and specified ([views and actions](views-and-actions.md)). A capability that needs a rendering the grammar does not express contributes a **widget name**, which the client resolves to a renderer and, when it does not know the name, falls back to the field type's default.

### 2.2 What it buys

1. **One client drives every capability**, including capabilities written after the client.
2. **A screen is extensible by a package that does not own it**, through the node-matching and position grammar.
3. **A screen is customisable per tenant, per role and per user**, because it is a record with group applicability and per-user variants.
4. **The same description drives several presentations.** A list description drives a desktop table, a small-screen card list and a printed table.
5. **Access control is applied to the description before it leaves the server**: nodes the user may not see are removed, so the client never has to be trusted with the decision.

### 2.3 The mechanics

- Views are records; actions are records; menus are records ([views and actions](views-and-actions.md)).
- The resolved description depends only on the requested kinds, the user's groups and access, the options, the language and the per-kind view-reference keys — nothing else — which is what makes it cacheable ([views and actions, section 14.5](views-and-actions.md#145-what-the-result-may-depend-on)).
- Expressions inside the description are evaluated **by the client** against the record being edited, which is what lets a field's visibility depend on unsaved values.

### 2.4 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| The grammar is a hard boundary | Anything the grammar cannot express cannot be built without changing the client. Capabilities that need a genuinely new interaction must contribute a client component and reference it by name, which reintroduces a client dependency for that one screen. |
| Client-side expressions are not enforcement | A visibility, read-only or required condition, and a relational field's candidate restriction, are guidance. Every one of them must be duplicated as a server-side validation when it is a real rule. Forgetting is the most common security defect in the system's design space. |
| Two evaluation engines | Filters are evaluated both as generated queries on the server and in memory on the client. The two must agree exactly, including on the empty-value rules. |
| Screen assembly costs a round trip and a cache | Resolving a view is expensive enough to need a cache, and the cache needs a precisely specified key. |
| Debugging is indirect | A wrong screen is the result of composing a dozen extensions. The system mitigates this by annotating nodes with their originating view in debug mode and by recording specifications that no longer match. |

**What a rebuild must not do instead.** Serving rendered markup, or letting the client decide what the user may see. Both break extension and both move an access decision to the untrusted side.

---

## 3. Configuration is data

### 3.1 The statement

Every choice an installation makes — which capabilities are installed, which accounts are used for which purpose, how documents are numbered, what the tax rules are, when the periodic jobs run, what the default values are, which languages exist — is stored as records, changeable through the ordinary interface, and subject to the ordinary access control and audit.

There is no configuration file that changes behaviour at run time. The only things outside the database are the location of the database, the search locations for packages, and the file store — none of which change what the system decides.

### 3.2 The four layers of configuration

| Layer | Where | Scope | Example |
|---|---|---|---|
| Which capabilities exist | The Capability Package entity | Tenant | Whether manufacturing is installed |
| Reference data | Ordinary entities, shipped by packages as data | Tenant, or per company | The chart of accounts, the units of measure, the tax definitions |
| System parameters | The System Parameter entity, key and value | Tenant | The tenant secret, a feature switch, a threshold |
| Per-entity defaults | The Default Value entity | Tenant, per company, or per user | The default journal, a user's pinned filter value |

A fifth, narrower layer is the **user's own preference**: language, time zone, notification channel, selected companies. It lives on the user record and on the user settings record.

### 3.3 What it buys

1. **An installation is reproducible**: copying the database copies the configuration exactly.
2. **A change is auditable**: it is a write, with audit fields, and often with tracking.
3. **A change is transactional**: a half-applied configuration change does not exist.
4. **A change is scoped**: the same setting can differ per company through a company-dependent field, and per user through the default catalogue.
5. **Configuration can be imported and exported** with the same tooling as business data.

### 3.4 The mechanics

- Reference data is loaded from a package's data files with external identifiers, so it can be reloaded without duplicating ([package system, sections 7 to 10](package-system.md#7-data-files)).
- A setting a user may change is shipped with the not-updatable flag so that a package update does not overwrite it ([package system, section 10.2](package-system.md#102-the-not-updatable-flag)).
- A setting that differs per company is a company-dependent field, whose fallback is the per-entity default for that company ([entity and field system, section 11](entity-and-field-system.md#11-company-dependent-values)).
- The settings screen is a **transient** entity: opening it reads the current values into a short-lived record, and confirming writes them back to their real homes. This is why a settings screen can present parameters, package installations and per-company fields as one form.

### 3.5 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| Every setting needs a home, a default and a scope | Deciding whether a setting is a parameter, a field on the company, a company-dependent field or a per-user default is a design act, and getting it wrong produces a setting that cannot be varied where it needs to be. |
| Package updates and user edits collide | The not-updatable flag resolves the collision, but it must be set deliberately. A shipped record that is updatable will silently overwrite the tenant's change on the next update; one that is not updatable will silently ignore the package's improvement. |
| Reading configuration costs queries | A decision that depends on three settings reads three records. The system mitigates this with registry caches, which then need invalidation, which then needs cross-worker signalling. |
| Configuration can be wrong in ways code cannot be | A missing default account produces a refusal at posting time, not at installation time. The system mitigates this with configuration steps and with explicit refusals naming the missing setting. |
| Settings that are really code | A setting whose value is an expression — a rule's filter, an action's criteria, a server action's body — is configuration that behaves like code, with none of code's review. It must therefore be restricted to privileged groups and evaluated in a restricted context. |

---

## 4. External identifiers make data reloadable

### 4.1 The statement

Every record contributed by a package carries a **stable, human-chosen name** — the package's technical name, a dot, and a local name — recorded in a catalogue alongside the record's numeric identifier. The numeric identifier is an accident of insertion order; the external identifier is the record's real name.

### 4.2 What it buys

1. **Data files are re-runnable.** Loading the same file twice updates rather than duplicating, because the identifier resolves to the existing record.
2. **Packages can refer to each other's records.** A tax rule can name an account defined by another package.
3. **Removal is possible.** Removing a package finds exactly the records it owns.
4. **Orphan cleanup is possible.** A record the package no longer ships is deleted, because the system can tell "this identifier belongs to this package and was not seen in this build".
5. **Shared ownership is expressible.** Two packages claiming a record means the record survives the removal of either.
6. **Integration is possible.** An external system's key can be recorded as an external identifier, which makes an import idempotent.
7. **The specification itself can name records.** Every reference-data statement in this repository names records by external identifier, which is stable across installations.

### 4.3 The mechanics

- The correspondence is a record of the External Identifier entity ([package system, section 9](package-system.md#9-external-identifiers)).
- Loading resolves, then updates or creates ([package system, section 10.1](package-system.md#101-create-or-update)).
- The not-updatable flag protects a record from the owner's own next update.
- Orphan cleanup at the end of a build deletes records no longer shipped ([package system, section 10.3](package-system.md#103-orphan-cleanup)).
- Duplicating a record gives the copy a derived identifier with a random suffix, so copies never collide.

### 4.4 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| A second row per shipped record | A full installation holds tens of thousands of identifier rows, indexed twice. |
| An extra lookup on every reference | Resolving an identifier is a query; the system caches it, and the cache must be invalidated on every write to the catalogue. |
| Deleting a record from a data file deletes it from every tenant | Orphan cleanup is unforgiving: removing a node from a file on the assumption that "nobody uses it" deletes it, along with anything that cascades. |
| The updatable/not-updatable decision is irreversible in practice | Once a tenant has customised a record, changing the flag either starts overwriting their change or stops shipping improvements. |
| Identifier namespaces must be disciplined | A record created under an installed package's prefix by a user import would be deleted at that package's next update — which is why such an import is refused outright. |
| Deletion order is derived from identifier order | Removal deletes most-recently-identified first, with per-record fallback. A rebuild deleting in a different order hits different referential failures and retains different records. |

---

## 5. Archival over deletion

### 5.1 The statement

The default way to take a record out of use is to **archive** it: set a boolean flag false. The record keeps its identifier, keeps every reference to it, remains readable, remains printable on the documents that used it, and disappears from ordinary searches. Deletion is reserved for records that were created by mistake.

### 5.2 What it buys

1. **History stays truthful.** An invoice issued last year still names the product that was sold, at the price it was sold for, by the salesperson who sold it — even though all three have since been retired.
2. **Referential integrity survives withdrawal.** Nothing has to be re-pointed or nulled.
3. **The action is reversible.** A product archived by mistake is unarchived; a product deleted by mistake is gone.
4. **Deletion can be genuinely restricted.** Because withdrawal has a non-destructive form, the system can afford to refuse deletion of anything referenced, which is what makes the ledger trustworthy.
5. **Reporting on closed periods still works**, because the records the period refers to still exist.

### 5.3 The mechanics

- An entity carries a boolean archive field; the entity layer discovers it by name ([entity and field system, section 18](entity-and-field-system.md#18-the-archive-flag)).
- Every search adds the condition that the flag is true, unless the context switches the filter off or the filter mentions the flag.
- Traversal of a relation in a filter always switches the filter off on the target, so an archived target does not silently drop the records pointing at it.
- Recomputation, duplication, company-consistency checking and record-rule evaluation all switch the filter off.
- Three operations exist: archive, unarchive and toggle.

### 5.4 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| Every query carries an extra condition | The implicit filter is added to every search on every entity that has the flag. It must be indexed, or it costs a scan. |
| "Where did my record go" is the commonest support question | A record that is invisible but exists is confusing in a way a deleted record is not. The system mitigates this by making the filter explicit in the search interface. |
| Two visibility mechanisms interact | Archival and record rules both hide; a user seeing nothing must be told which one is responsible. |
| Cascading is not automatic | Archiving a product should usually archive its variants; the platform cascades nothing, so every capability must decide and implement its own cascade. Inconsistent cascading is a real source of divergence. |
| Uniqueness constraints still apply to archived records | An archived product's reference still occupies the unique code, so a new product cannot reuse it. This surprises users and must be documented per entity. |
| Data grows without bound | Nothing is ever removed by ordinary use. Purging is a deliberate, separate capability with its own rules. |

### 5.5 When deletion is right

| Case | Why |
|---|---|
| A draft document abandoned before it had any effect | It never entered the record; nothing refers to it. |
| A transient dialog's state | It is not a business fact. |
| A duplicate created by a mistaken import | The correct state is that it never existed. |
| Records removed with their owning package | The capability is gone; so are its records. |

And where deletion would falsify the record — a posted entry, a numbered document, anything referenced by an accounting or inventory movement — it is refused outright, and archival is the only route.

---

## 6. Audit fields everywhere

### 6.1 The statement

Every persistent and transient record carries four fields, maintained automatically and never writable: **who created it**, **when**, **who last changed it**, and **when**. An entity may switch them off, and almost none do.

### 6.2 What it buys

1. **Every row answers "who and when" without a separate audit table.**
2. **Concurrency detection is free**: a client that read a record at a given modification stamp can ask the server to refuse a write if the stamp has moved.
3. **The housekeeping of transient records is possible**, because the modification stamp is what says whether a dialog is still being filled in.
4. **Ordering by recency is always available.**
5. **Support and reconciliation are possible**: a discrepancy can be traced to a moment and a user.

### 6.3 The mechanics

- The four fields exist on every entity that keeps them ([entity and field system, section 4](entity-and-field-system.md#4-the-automatic-fields)).
- The recorded user is the environment's **acting** user, not the effective one: elevating privileges does not change who is recorded ([security model, section 7.2](security-model.md#72-what-it-does-not-change)).
- The timestamps come from the **database clock**, so records written by different workers order consistently.
- They are never writable from the transport or from a data file.

### 6.4 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| Four columns on every table | On a full installation that is roughly five thousand columns whose only purpose is provenance, plus their storage on every row. |
| Two foreign keys on every table | Both to the user table, both of which must be maintained. |
| Every write touches two more columns | Which widens every update statement and every index page touched. |
| It is not a full audit trail | It records the *last* change, not every change. Knowing what a field used to be requires field tracking, which is a different and much more expensive mechanism, switched on per field. |
| Deleting a user is complicated | Users are referenced from every table. The deletion behaviour is to clear the reference, which loses provenance; in practice users are archived, which is [principle five](#5-archival-over-deletion) again. |
| Bulk operations record one actor | A scheduled job that writes ten thousand records records itself as the actor on all of them, which is true but not informative. |

### 6.5 Where it is deliberately switched off

Only for very high-volume entities where four columns per row is a measurable fraction of the table, and where provenance is carried by a parent record. Switching them off removes the concurrency check and, for a transient entity, is refused outright because the housekeeping policy needs the stamp.

---

## 7. Company scope per record

### 7.1 The statement

A tenant holds many legal companies. Separation between them is expressed **on the record**, by a company field, and enforced by a **record rule**, not by a separate database, a separate schema or a discriminator on the connection.

An empty company field means "shared by every company".

### 7.2 What it buys

1. **Reference data is shared without duplication.** One party, one product, one currency, one unit of measure serves every company.
2. **Inter-company operations are expressible.** A document in company A can create its counterpart in company B, because both are one query away.
3. **A user can work in several companies at once** by selecting them, without signing in again.
4. **Consolidated reporting is a filter, not an integration.**
5. **Adding a company is a record, not a deployment.**

### 7.3 The mechanics

- A company field on the record, and a **global** record rule per entity restricting to the selected companies ([security model, section 8.5](security-model.md#85-the-standard-global-rule)).
- The selection lives in the environment's context; the current company is the first selected, the allowed companies are the whole selection ([architecture, section 6.5](architecture.md#65-company-selection)).
- An empty selection means **all** the user's companies, not the main one, so that non-interactive operations do not silently lose records.
- A **consistency check** prevents linking records of incompatible companies ([security model, section 9](security-model.md#9-company-consistency)).
- A **company-dependent field** lets one shared record carry a different value per company ([entity and field system, section 11](entity-and-field-system.md#11-company-dependent-values)).

### 7.4 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| Separation is soft | Nothing physically prevents a query from crossing companies. Every protection is a rule, and a rule that is missing, wrong, or bypassed by an elevated operation lets data cross. This is the single largest correctness risk in the design. |
| Every entity must decide its scope | Company field or not; shared or scoped; company-dependent values or not. Getting it wrong is discovered late, when a second company is added. |
| Every cache key must include the company selection | The rule filter cache, the company-dependent value cache and several derived answers all vary by company. A cache key that omits it serves one company's data to another. |
| Consistency must be checked, not assumed | Two records both visible to a user can still be incompatible. The check is opt-in per field and per entity, so omitting it silently allows crossovers. |
| Elevated operations bypass the scope | Which is exactly what inter-company operations need, and exactly what makes an incautious elevation dangerous. |
| Company-dependent values have no foreign keys | Because they live inside a structured document. Deleting a referenced record must clear them explicitly. |
| Reporting must be explicit about scope | A total is meaningless without saying which companies it covers. |

### 7.5 The alternative that was not taken

One database per company would make separation physical and the protections unnecessary. It would also make shared reference data impossible, inter-company operations an integration project, consolidated reporting an extract-and-merge exercise, and adding a company a deployment. The system accepts soft separation and pays for it with discipline.

---

## 8. Strict currency and precision discipline

### 8.1 The statement

Numbers that carry money, quantity, time or a rate are never treated as ordinary numbers.

1. **An amount is always accompanied by its currency**, declared on the field, and is rounded to that currency's precision at the moment of assignment.
2. **A quantity is always accompanied by its unit of measure**, and conversions between units are explicit and rounded.
3. **A decimal number with a declared precision is stored in fixed-point form** and rounded on assignment; one without a declared precision is stored as a floating-point number and is not rounded.
4. **Comparison is precision-aware**: two values are equal when their difference rounds to zero at the relevant precision.
5. **Rounding is half away from zero** everywhere, unless a specific rule says otherwise.

### 8.2 What it buys

1. **Totals reconcile.** A document total equals the sum of its lines because both were rounded at the same precision by the same rule.
2. **Currencies with two, three and zero decimal places all work**, because the precision comes from the currency record.
3. **Comparisons are stable.** A quantity of 0.001 is zero at two decimal places, and the system says so consistently.
4. **Precision is configurable** through precision-setting records, per kind of quantity, without touching any formula.
5. **Exchange differences are computed, not accumulated as noise.**

### 8.3 The mechanics

- A monetary field declares its currency field; the currency must exist on the entity, or the registry build fails ([entity and field system, section 6.5](entity-and-field-system.md#65-monetary-amount)).
- Assignment rounds to the currency's decimal places; assigning across records with different currencies is refused.
- Monetary fields are written after other fields within one write, so a currency set in the same call is known when the amount is rounded.
- A decimal field may declare its precision as a pair or as the name of a precision-setting record; the setting is read at conversion time.
- Precision-aware rounding, zero-testing and comparison are the operations business logic uses, never raw comparison.
- A monetary field can be aggregated in a grouped screen only when its currency field can also be aggregated, so amounts in different currencies are never summed.

```formula
stored_amount   = round_half_away_from_zero( assigned_amount , decimal_places_of( currency ) )
values_equal    = round_half_away_from_zero( first − second , precision ) = 0
```

### 8.4 A worked example of why it matters

A line has quantity 3 and unit price 10.335, in a currency with two decimal places.

| Order of operations | Result |
|---|---|
| Round the unit price, then multiply: round(10.335) = 10.34; 3 × 10.34 | **31.02** |
| Multiply, then round: 3 × 10.335 = 31.005; round(31.005) | **31.01** |

Two defensible implementations differ by one hundredth on one line, and by material amounts on a document with hundreds of lines. Behavioural equivalence therefore requires not only the rounding rule but the **evaluation order**, which is why every formula in this specification states both.

### 8.5 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| Every monetary field needs a currency field | Which means most transactional entities carry a currency link, and related fields must be added to lines so that a line knows its document's currency. |
| Rounding on assignment is surprising | A value assigned and read back is not the value assigned. Code that assumes otherwise is wrong, and code that round-trips through the cache silently rounds. |
| Fixed-point storage is slower than floating point | Sums, averages and comparisons over fixed-point columns cost more. This is why a field with no declared precision stays floating point. |
| The evaluation order is part of the contract | It cannot be refactored freely; changing where the rounding happens changes the amounts. |
| Precision settings are global and retroactive only forward | Changing a precision does not re-round stored values, so an installation can hold values at two different precisions. |
| Multi-currency aggregation must be refused, not approximated | Which makes some obvious-looking reports unavailable until a conversion is specified. |
| Unit conversions compound | Converting a quantity twice can lose more than converting once; conversion points must therefore be specified, not left to the implementation. |

---

## 9. Extension over modification

### 9.1 The statement

A capability package never modifies another package's definitions. It **extends** them: it adds fields to an entity, wraps an operation, inserts nodes into a screen, adds rows to reference data, grants additional access, adds sections to a document. The extended package is unaware, unchanged and independently removable.

### 9.2 What it buys

1. **Six hundred and twenty packages compose into one application** without any of them being forked.
2. **A capability can be added and removed** without editing anything.
3. **A tenant-specific customisation is a package**, with the same mechanics as a shipped one.
4. **Upgrading one package does not lose another's changes**, because the changes were never in the first package's text.
5. **Bridges are possible.** More than four hundred of the shipped packages exist only to make two others work together, and they are installed automatically when both are present.

### 9.3 The mechanics

Six mechanisms, all specified in [inheritance and extension](inheritance-and-extension.md):

| Mechanism | Extension point |
|---|---|
| Extend an entity in place | Entities, fields, operations, constraints |
| Copy a definition into a new entity | A new entity sharing a shape |
| Embed a parent record | A record recorded from two angles |
| Adopt an abstract behaviour | Reusable behaviour such as messaging |
| Modify a view by specification | Screens |
| Contribute records with external identifiers | Data, security, menus, reports |

Two contracts govern them:

- **Field attribute layering**: redeclaring a field to change one attribute keeps everything else.
- **The call-the-previous-definition contract**: an extension of an operation must call the previous definition exactly once, with the same receiver and the same arguments, unless it deliberately suppresses it.

### 9.4 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| The resolved behaviour is nowhere written down | What an operation does is the composition of every extension of it. No single place states the result. Diagnosing requires walking the chain. |
| Extensions can break each other silently | A view specification that replaces a node discards what other extensions inserted inside it. An operation extension that does not call the previous definition disables every extension below it. Neither is detected. |
| Order matters and is only partly controllable | Composition order follows package dependencies; two packages that do not depend on each other may compose in either order. An extension that needs an order must declare a dependency. |
| Anchors are fragile | A view specification matching by a positional path breaks when someone inserts a node. The discipline of matching on stable anchors is unenforced. |
| Extending an abstract behaviour reaches everywhere | Adding a required field to a widely adopted behaviour adds a required column to hundreds of tables, in packages the author has never seen. |
| The registry build is expensive and all-or-nothing | Composition runs per package per build; a failure anywhere discards the whole registry. |
| Nothing can be removed, only overridden | A field, once declared, exists for as long as its package is installed. Deprecation is expressed by hiding, not by removing. |

### 9.5 The discipline the mechanism requires

Stated as rules because the mechanism does not enforce them:

1. Add rather than replace: add a selection value, add a validation, insert a node.
2. Match views on stable anchors — a field name or a named element, not a path.
3. Declare every dependency read, both package dependencies and computation dependencies.
4. Never assume a single record.
5. Keep an extension's effects inside its own concern.
6. Prefer the stronger mechanism where it can express the requirement: a record rule over an extension of the search operation; a computed field over a reaction rule; a database constraint over a declared validation.

---

## 10. Declared dependencies over manual invalidation

### 10.1 The statement

A derived value declares **what it depends on**, as paths that may traverse relations. The system then guarantees that the value is recomputed whenever anything on any of those paths changes, wherever the change happens and whoever makes it.

No call site ever says "and now recompute the order total". A call site that had to would be wrong the moment a new way of changing a line was added.

### 10.2 What it buys

1. **Correctness under extension.** A package that adds a new way of changing a line does not have to know about the order total; the dependency graph reaches it.
2. **Batching.** Deferring computation to the flush point lets one computation run for many records.
3. **Coalescing.** Ten changes that invalidate the same value cause one computation.
4. **Consistency.** A computation that reads another computed field triggers it first, so the result does not depend on the order of the writes.
5. **A single mechanism.** The same machinery maintains stored computed fields, invalidates non-stored ones, and keeps both sides of a relation coherent in memory.

### 10.3 The mechanics

- Dependency paths are declared per field, or per computation rule and unioned across contributing rules ([entity and field system, section 8.3](entity-and-field-system.md#83-declared-dependencies)).
- The registry derives, per field, a **trigger tree**: the fields that depend on it directly, and, at each edge labelled by a relational field, the fields that depend on it through that relation ([entity and field system, section 9.1](entity-and-field-system.md#91-trigger-trees)).
- A change notifies: the tree is traversed, relations are followed **backwards**, stored computed fields are added to the pending map, non-stored ones are dropped from the cache.
- A destructive change — a deletion, or the removal of a link — notifies **before** the change, because afterwards the relation no longer leads anywhere.
- A dependency that reaches the same field through a relation must be declared **recursive**, or the traversal stops one level too early.
- The flush point runs the pending computations to a fixed point, bounded at one hundred iterations, then writes.

### 10.4 A worked example

An order line's subtotal depends on its quantity and unit price. An order's untaxed total depends on its lines' subtotals. A party's total ordered depends on its orders' totals, and is not stored.

Changing one line's quantity:

1. The subtotal of that line is marked pending.
2. Traversing the order link backwards, the order's total is marked pending.
3. Traversing the orders link backwards, the party's total ordered is dropped from the cache — it is not stored, so there is nothing to recompute until somebody reads it.
4. At flush, the line's subtotal is computed for every pending line in one call, then the order's total for every pending order in one call. Two statements reach the database.

Deleting the line instead: the traversal must run **before** the deletion, or step 2 finds nothing and the order's total is left stale.

### 10.5 The trade-off

**What it costs.**

| Cost | Detail |
|---|---|
| An undeclared dependency is a silent wrong answer | The field is simply not recomputed. Nothing fails; the value is just wrong until something else happens to invalidate it. This is the single most common defect class the mechanism admits. |
| The graph is large and must be derived | Nearly four thousand computed fields with declared dependencies produce a large trigger forest, rebuilt whenever the registry is set up. |
| Backwards traversal costs queries | Following a relation backwards uses the inverse where one exists and a search where none does. A dependency through a relation with no usable inverse is expensive on every change. |
| Recursion must be declared by hand | The system cannot infer it, and getting it wrong produces values that are right at one level and stale at the next. |
| Cycles are possible and are only caught by a bound | A dependency cycle produces an excessive-iteration condition after one hundred passes, not a clear error at definition time. |
| Deferred computation changes when errors appear | A computation that refuses appears at flush time, possibly far from the write that caused it. |
| Debugging requires the graph | "Why did this change?" is answered by walking the dependency graph backwards, which is not visible in any one place. |
| Stored derived values duplicate data | A stored computed field is a cached copy kept in step by this machinery. A rebuild that omits the machinery produces stale copies that look like data. |

### 10.6 The alternative that was not taken

Recomputing everything on read would need no dependency declarations and would always be correct. It would also make every list screen recompute every derived value of every row, make ordering and filtering by a derived value impossible without a column, and make a document total a query over its lines on every display. The system accepts declared dependencies and pays for them with the defect class above.

---

## 11. When the principles conflict

The ten principles are not independent, and three pairs pull against each other. How the system resolves them is itself part of the specification.

### 11.1 Configuration is data against extension over modification

A package ships a record; a tenant edits it; the package is updated. Who wins?

**Resolution.** The package author decides, once, by marking the record updatable or not:

| Mark | Meaning | Use for |
|---|---|---|
| Updatable | The package owns the record; a tenant's edit is lost at the next update | Records whose correctness the package is responsible for: a tax computation, a security rule, a view |
| Not updatable | The tenant owns the record after the first installation | Records that are a **starting point**: a default sequence, a sample template, a suggested account |

And reinitialisation exists as the deliberate escape hatch: it reloads a package's data in initial mode, which rewrites even the not-updatable records, discarding the tenant's customisation on purpose.

### 11.2 Archival over deletion against external identifiers make data reloadable

Orphan cleanup **deletes** records a package no longer ships. That is deletion, not archival, and it is irreversible.

**Resolution.** The two apply at different levels. Archival is how a **user** withdraws a record from use. Orphan cleanup is how a **package** withdraws a record it owns. A package that wants a shipped record to survive its own removal from the data files must either keep shipping it, mark it not updatable, or have a second package claim it.

The asymmetry is deliberate: a package's data is part of the package, and a package that no longer ships a record is saying the record should not exist.

### 11.3 Everything is an entity against strict currency and precision discipline

Configuration entities hold amounts too — a credit limit, a threshold, a default price. Those amounts need a currency, which means a configuration entity needs a currency field, which drags the multi-currency machinery into configuration.

**Resolution.** A configuration amount is either:

- expressed in a **named** currency carried on the same record, which makes it a proper monetary field; or
- expressed as a **ratio or a count**, which needs no currency; or
- made **company-dependent**, so it is implicitly in the company's currency.

An amount stored with no currency at all is a defect, and the registry refuses a monetary field whose currency field does not exist.

### 11.4 Company scope against extension

An extension adding a field to an entity must decide whether the field is company-scoped, company-dependent or shared — a decision it can only make correctly if it understands the entity's own scope. An extension that adds a shared field to a scoped entity, or a scoped field to a shared one, produces a subtle multi-company defect.

**Resolution.** The entity's scope is part of its specified contract and is stated in its documentation; an extension is expected to follow it. The consistency check catches the case where a **relation** crosses companies, but nothing catches a plain field with the wrong scope.

### 11.5 Declared dependencies against extension

An extension that changes what a computation reads must also change what it declares. If it declares nothing, the computation is silently stale.

**Resolution.** Declarations **union** across contributing rules: an extension declaring a rule of the same name adds its dependencies to the existing ones without needing to know them. An extension declaring the dependencies **on the field** replaces them, which is the sharper instrument and must be used knowingly.

---

## 12. The measured footprint of each principle

A principle that is stated but rarely applied is an aspiration. The counts below come from parsing the complete definition of the shipped system and from introspecting a full installation. They are the evidence that each principle is structural.

### 12.1 Everything is an entity

| Measure | Count |
|---|---|
| Entities catalogued | 983 |
| — persistent | 600 |
| — transient | 222 |
| — abstract | 161 |
| Persistent tables in a full installation | 1,240 |
| Fields declared across entities | 14,028 |
| Fields resolved at run time, after every extension | 23,235 |
| Operations catalogued on entities | 18,344 |

Of the 983 entities, the ones describing the system to itself — the entity catalogue, the field catalogue, the selection-value catalogue, the constraint catalogue, the relation catalogue, the embedded-parent catalogue, external identifiers, packages, package dependencies, package exclusions, package categories, views, view customisations, menus, the six action kinds, embedded actions, configuration steps, access rights, record rules, groups, privilege families, scheduled jobs and their triggers, sequences and their date ranges, system parameters, per-entity defaults, saved filters, logging records and profiling records — number more than thirty. **Every one of them is an ordinary entity with ordinary fields, ordinary access rights and ordinary views.**

That 222 of the 983 entities are transient is itself a consequence of the principle: a multi-step dialog is a record, so the whole field, default, computation, on-change and validation machinery is available inside a dialog with no special case.

### 12.2 The interface is data

| Measure | Count |
|---|---|
| View declarations | 3,671 |
| Window actions | 978 |
| Menus | 893 |
| Printable report definitions | 94 |
| Routes exposed over the transport | 1,023 |

Nearly four thousand view declarations describe every screen of the system. None of them is code.

### 12.3 Configuration is data

| Measure | Count |
|---|---|
| Shipped reference data sets | 155 |
| Country chart template data sets | 1,078 |
| System parameters shipped | a small fixed set, listed in the operational catalogue |
| Decimal precision settings | a small fixed set, one per kind of quantity |
| Capability packages catalogued | 620 |

The 1,078 country chart template data sets are the clearest illustration: the accounting behaviour of more than a hundred jurisdictions is expressed entirely as records, not as code branches.

### 12.4 External identifiers

Every one of the 155 shipped reference data sets, every one of the 3,671 views, every one of the 978 window actions, every one of the 893 menus, every one of the 1,933 access rights and every one of the 576 record rules is addressed by an external identifier. A full installation therefore holds tens of thousands of identifier rows, which is the price of reloadability and removability.

### 12.5 Archival over deletion

Entities carrying the archive flag are the ones whose records outlive their usefulness: parties, products, users, companies, accounts, journals, taxes, price lists, warehouses, locations, employees, projects, campaigns, and most configuration entities. Entities **not** carrying it are the ones whose records are events rather than things: journal items, stock moves, messages, notifications, attendance records.

The rule of thumb the catalogue follows: **a thing is archivable; an event is not.** An event that should not have happened is reversed, not archived.

### 12.6 Audit fields

The four audit fields exist on essentially every one of the 1,240 persistent tables and every transient one, which is roughly five thousand columns and two and a half thousand foreign keys whose only purpose is provenance. Turning them off is so rare that a transient entity attempting it is refused outright.

### 12.7 Company scope

| Measure | Count |
|---|---|
| Record rules | 576 |
| — of which the multi-company global rules | the large majority of the global ones |
| Company-dependent fields declared | 52 |

The small number of company-dependent fields against the large number of company-scoped entities is informative: **most records belong to one company; only a few shared records need a per-company value.** The 52 are concentrated on parties, products and product categories — exactly the shared master data whose accounting treatment differs per company.

### 12.8 Currency and precision

| Measure | Count |
|---|---|
| Monetary fields declared | 232 |
| Decimal-number fields declared | 697 |
| Integer fields declared | 1,155 |

Every one of the 232 monetary fields names a currency field, and the registry build refuses any that does not. The 697 decimal-number fields split into those with a declared precision — quantities, rates, percentages — which are stored in fixed-point form and rounded on assignment, and those without, which are stored as floating-point numbers and are not rounded.

### 12.9 Extension over modification

| Measure | Count |
|---|---|
| Capability packages | 620 |
| — declaring automatic installation | 402 |
| — presented as applications | 34 |
| Dependency edges between packages | 1,286 |
| Entities carrying at least one adopted abstract behaviour | several hundred |
| — adopting the thread behaviour | 82 |
| — adopting the activity behaviour | 60 |
| Entities embedding a parent record | 12 |

The most extended entities show how far the mechanism is pushed:

| Entity | Extended by |
|---|---|
| Chart of Accounts Template | 179 packages |
| Party | 125 packages |
| Settings | 124 packages |
| Company | 120 packages |
| Journal Entry | 115 packages |
| Product Template | 57 packages |
| User | 53 packages |
| Sales Order | 43 packages |
| Tax | 42 packages |
| Journal Item | 40 packages |

One hundred and fifteen packages contribute to the journal entry. No two of them know about each other. **A rebuild that cannot compose 115 independent contributions into one entity cannot install the shipped catalogue.**

That only 12 entities embed a parent, against several hundred that adopt an abstract behaviour, is also informative: embedding is for the rare case of one real-world thing recorded from two angles; adoption is the everyday mechanism.

### 12.10 Declared dependencies

| Measure | Count |
|---|---|
| Computed fields | 3,979 |
| — of which stored, and therefore maintained by recomputation | 1,537 |
| Related fields | 1,616 |
| Relational fields mapped | 3,985 |
| Fields marked for change tracking | 476 |
| Fields marked translatable | 324 |
| Fields carrying a group restriction | 587 |
| Fields carrying an index | 894 |

Nearly four thousand computed fields, of which more than fifteen hundred are stored, means the trigger forest is large and the recomputation machinery is exercised on almost every write. **A rebuild that computes on read instead will produce the same values and a system that cannot sort, group or filter by any of those fifteen hundred.**

---

## 13. Signals that a rebuild has diverged

Each principle abandoned produces a characteristic symptom. These are what to look for in an implementation that claims equivalence.

| Principle abandoned | Symptom |
|---|---|
| Everything is an entity | Adding a capability requires editing a configuration file, restarting a process, or changing a client. The list of screens cannot be queried. An access right cannot be exported. |
| The interface is data | A new field needs a client change. Two capabilities cannot both add something to the same screen. A user cannot be given a different screen from a colleague without a code branch. |
| Configuration is data | Two installations of the same version behave differently and the difference cannot be found in the database. A configuration change is not in the audit trail. Copying the database does not copy the behaviour. |
| External identifiers | Loading the same reference data twice duplicates it. Removing a capability leaves its records behind, or removes records another capability also owns. A package update cannot withdraw a record it no longer ships. |
| Archival over deletion | Old documents lose the names of the things they refer to. Withdrawing a product requires re-pointing every reference, or is refused. Deleting is the only way to take something out of use, and it is therefore permitted where it should not be. |
| Audit fields | "Who changed this" cannot be answered without a separate audit mechanism. Optimistic concurrency detection is unavailable. Transient records cannot be aged out. |
| Company scope per record | Reference data is duplicated per company. An inter-company operation is an integration. Consolidated reporting is an extract. Adding a company is a deployment. Or, in the failure direction: one company's records appear in another's lists. |
| Currency and precision | Document totals differ from the sum of their lines by small amounts. A currency with three decimal places or zero decimal places misbehaves. A quantity comparison says two visually identical values differ. Amounts in two currencies are summed. |
| Extension over modification | Adding a capability requires editing another's source. Two capabilities cannot both extend the same screen or operation. Removing a capability leaves fragments. A tenant's customisation is lost on update. |
| Declared dependencies | A derived value is correct when changed through the screen and stale when changed through the transport, an import or a scheduled job. A document total drifts from its lines. A new way of changing a line silently breaks a total nobody remembered. |

### 13.1 The three most expensive divergences

Ranked by how much of the system they invalidate:

1. **Declared dependencies.** Abandoning them does not fail loudly; it produces stale derived values on exactly the paths nobody tested. Every financial total, every inventory quantity and every progress indicator in the system is a computed field. A rebuild that recomputes only where the original screen recomputes will pass a screen-driven test suite and fail in production.
2. **Extension over modification.** Abandoning it makes the shipped catalogue uninstallable: 620 packages contributing to 983 entities, one of which is extended by 179 of them, cannot be expressed as a set of mutually-aware modules.
3. **Currency and precision.** Abandoning it produces a system whose numbers are almost right, which is worse than a system whose numbers are obviously wrong, because the discrepancies accumulate silently in the ledger.

---

## 14. Principles deliberately not adopted

A rebuild can diverge by **adding** a principle as easily as by dropping one. The following are recognisable alternatives that the system deliberately does not use, and adopting one will change observable behaviour.

### 14.1 Event sourcing

**Not used.** The system stores **current state** in tables, with a conversation and a field-tracking history alongside. It does not store a log of events from which state is derived.

Consequences of the choice, which a rebuild must reproduce:

- A record's current value is read directly, not folded from a stream.
- The history is **partial and declared**: only the fields marked for tracking have a recorded history, and only at transaction granularity.
- There is no way to reconstruct the state of a record at an arbitrary past instant. Where the business requires that — an accounting period, an inventory valuation — the system stores **explicit periodic records** (a valuation layer, a posted entry, a stock quantity snapshot) rather than replaying a log.
- Correcting a mistake is a **reversal**, which creates a compensating record, not a rewrite of history.

A rebuild that uses event sourcing internally may still be equivalent, provided the externally visible records, their identifiers, their audit fields and their tracking entries are the same. A rebuild that exposes an event log as the source of truth is not.

### 14.2 Separating reads from writes

**Not used.** The same entities, the same fields and the same access rules serve reading and writing. There is no separate read model.

Consequences:

- A value written is immediately readable in the same transaction, through the unit of work.
- A search sees uncommitted changes of its own transaction, because the unit of work flushes before querying.
- There is no eventual consistency anywhere inside a transaction.

The one concession is the **read-only cursor** for endpoints declared read-only, which may be served by a replica — and even that re-runs the whole endpoint on a read/write cursor if it attempts a write, precisely so that the programming model stays single.

### 14.3 Soft-deleting everything

**Not used.** Archival is opt-in per entity, by declaring the flag; most event-like entities do not have it. Deletion is real deletion.

A rebuild that adds a deleted flag to every table would change: the meaning of uniqueness constraints, the cost of every query, the behaviour of removal when a package is uninstalled, and the semantics of the reversal operations that exist precisely because deletion is not available.

### 14.4 A separate audit log

**Not used.** Provenance is the four audit fields; history is field tracking into the conversation. There is no universal write-ahead audit table.

Consequences: tracking is **declared per field** and costs a message per record per transaction; untracked fields have no history at all. A rebuild that logs every write to a shadow table would answer more questions and would also store an order of magnitude more data and behave differently under bulk operations.

### 14.5 Denormalising for read performance

**Used sparingly and always declared.** Where a value is duplicated — a company copied onto every line, a party copied onto every journal item, a total stored on a document — it is a **stored computed field** with declared dependencies, maintained by the recomputation machinery.

A rebuild that denormalises without the declared dependency has produced a cache it must invalidate by hand, which is exactly what [principle ten](#10-declared-dependencies-over-manual-invalidation) exists to avoid.

### 14.6 Per-tenant schema variation beyond packages and user-defined fields

**Not used.** Two tenants with the same installed packages have the same schema. The only per-tenant schema variation is the set of installed packages and the user-defined entities and fields recorded in the database, both of which go through the same registry build and the same schema synchronisation.

A rebuild that lets a tenant's schema drift arbitrarily loses the ability to install, update and remove packages predictably.

### 14.7 Compile-time knowledge of the model

**Not used.** The registry is built at run time from whatever packages are installed, and can be rebuilt while the process is running. Nothing about the entities is fixed at build time.

This is what makes installing a package without a restart possible, and it is why the registry must be signalled between workers. A rebuild that generates code from the model at build time must still reproduce the run-time install path, or accept that installing a capability is a deployment.

### 14.8 A message bus as the authority for coherence

**Not used as the authority.** Coherence between workers is achieved by counters **in the tenant's own database**, read on the same cursor as the data. A bus may be added to reduce latency, but the counters remain the authority, because a worker that can read the data can always read the counters and there is therefore no window in which a stale registry is used against new data.

---

## 15. Acceptance criteria

These check that a rebuild has adopted the principles, not merely the tables.

### Everything is an entity

**AC-PRIN-1.** *Given* a running installation, *when* the list of entities is queried through the ordinary search operation, *then* the entity catalogue itself, the view entity, the menu entity and the access-right entity all appear as ordinary entities with fields.

**AC-PRIN-2.** *Given* an administrator, *when* they create an access right through the ordinary creation operation, *then* it takes effect on the next check without any restart, and the change carries audit fields.

**AC-PRIN-3.** *Given* a capability package, *when* it is removed, *then* its entities, its views, its menus, its access rights and its reference data all disappear by the same removal mechanism.

### The interface is data

**AC-PRIN-4.** *Given* a package adding a field to an entity and inserting a node for it into an existing form, *when* the package is installed, *then* an unmodified client renders the new field.

**AC-PRIN-5.** *Given* a field node whose visibility condition hides it, *when* the field is written directly over the transport, *then* the write succeeds — the condition is not enforcement.

**AC-PRIN-6.** *Given* two requests for the same view by two users with the same groups and the same language but different selected companies, *when* the view is resolved, *then* the same cached description is returned, because the company is not part of the key.

### Configuration is data

**AC-PRIN-7.** *Given* a tenant, *when* its database is copied, *then* the copy makes the same business decisions with no further configuration.

**AC-PRIN-8.** *Given* a shipped record marked not updatable that a user has edited, *when* its package is updated, *then* the edit survives; *when* the package is reinitialised, *then* the shipped value is restored.

**AC-PRIN-9.** *Given* a company-dependent setting with no entry for company 2, *when* it is read in company 2, *then* the per-entity default for company 2 is returned.

### External identifiers

**AC-PRIN-10.** *Given* a data file loaded twice, *when* the second load completes, *then* the record count is unchanged and the values are those of the second load.

**AC-PRIN-11.** *Given* a record claimed by two packages, *when* one of them is removed, *then* the record survives and only that package's identifier row is deleted.

**AC-PRIN-12.** *Given* a record a package no longer ships, marked updatable, *when* the package is updated, *then* the record is deleted.

### Archival over deletion

**AC-PRIN-13.** *Given* a product archived after being sold, *when* the invoice that sold it is read and printed, *then* the product's name and reference still appear.

**AC-PRIN-14.** *Given* the same archived product, *when* an ordinary product search runs, *then* it is absent; *when* a filter mentioning the archive field runs, *then* it is present.

**AC-PRIN-15.** *Given* an invoice whose party is archived, *when* a filter traverses the party relation, *then* the invoice still matches, because traversal switches the archive filter off.

**AC-PRIN-16.** *Given* a posted journal entry, *when* deletion is attempted, *then* it is refused, and archival or reversal is the only route.

### Audit fields

**AC-PRIN-17.** *Given* a restricted user triggering an operation that elevates privileges and creates a record, *when* the record is examined, *then* its creator is the restricted user.

**AC-PRIN-18.** *Given* two records created by two different workers within the same second, *when* they are ordered by creation stamp, *then* the order is consistent, because both stamps come from the database clock.

**AC-PRIN-19.** *Given* a client that read a record at a given modification stamp and a concurrent change, *when* the client writes with the stamp, *then* the write is refused.

### Company scope

**AC-PRIN-20.** *Given* a user allowed in companies 1 and 2 with only company 1 selected, *when* they search an entity carrying the standard global rule, *then* only records of company 1 and shared records are returned.

**AC-PRIN-21.** *Given* an account of company 2 and an invoice of company 1, both visible, *when* the account is set on the invoice on an entity that checks consistency, *then* the write is refused.

**AC-PRIN-22.** *Given* an elevated operation, *when* it selects a company the underlying user does not belong to, *then* it succeeds, because elevation bypasses the authorisation check.

**AC-PRIN-23.** *Given* a rule filter cached for selection [1], *when* the selection becomes [2], *then* a different cache entry is used.

### Currency and precision

**AC-PRIN-24.** *Given* a monetary field whose currency has two decimal places, *when* 12.345 is assigned and read back, *then* the value is 12.35.

**AC-PRIN-25.** *Given* a line with quantity 3 and unit price 10.335 in a two-decimal currency, *when* the subtotal is computed according to the specified evaluation order, *then* it is the value the specification's formula gives, and a rebuild rounding in the other order is not equivalent.

**AC-PRIN-26.** *Given* two quantities differing by 0.001, *when* they are compared at two decimal places, *then* they are equal.

**AC-PRIN-27.** *Given* a list grouped by a field, containing amounts in two currencies, *when* the aggregate is requested, *then* no single total is produced.

**AC-PRIN-28.** *Given* an entity declaring a monetary field whose named currency field does not exist, *when* the registry is built, *then* the build fails.

### Extension over modification

**AC-PRIN-29.** *Given* three packages extending one operation, each calling the previous definition once, *when* it is invoked, *then* all three and the base run, once each.

**AC-PRIN-30.** *Given* a package redeclaring a field only to add a help text, *when* the registry is built, *then* the field keeps its computation, its dependencies, its default and its group restriction.

**AC-PRIN-31.** *Given* a package adding a selection value and another replacing the whole selection, *when* both are installed in that order, *then* the added value is gone — which is why adding is preferred to replacing.

**AC-PRIN-32.** *Given* a package removed, *when* the registry is rebuilt, *then* every field, view node, menu, access right and record it contributed is gone, and the packages it extended are unchanged.

### Declared dependencies

**AC-PRIN-33.** *Given* a stored computed order total depending on its lines' subtotals, *when* a line is created, changed or deleted by **any** path — a form, the transport, an import, a scheduled job, another package's operation — *then* the total is recomputed.

**AC-PRIN-34.** *Given* a line deleted, *when* the transaction flushes, *then* the order's total is correct, which requires the affected order to have been collected before the deletion.

**AC-PRIN-35.** *Given* a computation that reads a field it does not declare, *when* that field changes, *then* the computation does **not** run and the value is stale. This is the specified behaviour and is why declaring every read field is mandatory.

**AC-PRIN-36.** *Given* twelve lines of one order changed in one operation, *when* the transaction flushes, *then* the line computation is invoked once for twelve records and the order computation once for one record.

**AC-PRIN-37.** *Given* a dependency cycle among computed fields, *when* a flush is attempted, *then* it stops after one hundred iterations and reports an excessive-iteration condition.

---

## Related documents

- [Architecture](architecture.md) — the registry, the environment, record sets and the unit of work that principles one, nine and ten rest on.
- [The package system](package-system.md) — external identifiers, reload semantics and the lifecycle that principles three, four and nine rest on.
- [The entity and field system](entity-and-field-system.md) — the archive flag, audit fields, company-dependent values, precision and the recomputation algorithm.
- [Inheritance and extension](inheritance-and-extension.md) — the six extension mechanisms and their contracts.
- [The security model](security-model.md) — company scope, the rules that enforce it, and what is not enforcement.
- [Views and actions](views-and-actions.md) — the presentation contract that principle two defines.
- [The messaging model](messaging-model.md) — the largest adopted behaviour, and the clearest illustration of what adoption costs.
- [Coverage and evidence](../reimplementation/coverage-and-evidence.md) — what remains uncertain, and why a principle can be stated confidently while one of its instances cannot.
