# The entity and field system

Everything the system stores, computes, validates, displays or exchanges is a **field of an entity**. This document specifies the entity and field machinery in full: the three entity kinds and how each is stored; every field type with its storage form, value semantics, attributes and rounding rules; relational fields, self-references and the materialised ancestor path; computed fields, their declared dependencies, the choice to store or not, their inverse and filter rules, their derived capabilities and the recomputation algorithm including traversal across relations; related fields; company-dependent values and their fallback; translatable values; defaults, including the recorded default values a tenant can add; on-change behaviour and its limits; validation rules and database constraints; ordering, display names and name searching; the archive flag; audit fields; the field descriptions served to clients; how the database schema is derived from all of it; and fields created while the system is running.

Read [the architecture](architecture.md) first: this document assumes the registry, the environment, the record set and the unit of work. The operations that read and write these fields, and the complete grammar of the filter notation, are in [record operations and query notation](record-operations-and-query-notation.md).

---

## Table of contents

1. [Entities](#1-entities)
2. [The three entity kinds](#2-the-three-entity-kinds)
3. [Entity attributes](#3-entity-attributes)
4. [The automatic fields](#4-the-automatic-fields)
5. [Field attributes common to every type](#5-field-attributes-common-to-every-type)
6. [The field types](#6-the-field-types)
7. [Relational fields in depth](#7-relational-fields-in-depth)
8. [Computed fields](#8-computed-fields)
9. [The recomputation algorithm](#9-the-recomputation-algorithm)
10. [Related fields](#10-related-fields)
11. [Company-dependent values](#11-company-dependent-values)
12. [Translatable values](#12-translatable-values)
13. [Defaults](#13-defaults)
14. [On-change behaviour](#14-on-change-behaviour)
15. [Validation](#15-validation)
16. [Ordering](#16-ordering)
17. [Display names and name searching](#17-display-names-and-name-searching)
18. [The archive flag](#18-the-archive-flag)
19. [Copying](#19-copying)
20. [The filter grammar](#20-the-filter-grammar)
21. [Field metadata exposed to clients](#21-field-metadata-exposed-to-clients)
22. [Deriving the database schema](#22-deriving-the-database-schema)
23. [User-created fields](#23-user-created-fields)
24. [A worked example of an entity from end to end](#24-a-worked-example-of-an-entity-from-end-to-end)
25. [Invariants a rebuild must preserve](#25-invariants-a-rebuild-must-preserve)
26. [Acceptance criteria](#26-acceptance-criteria)
27. [Reconciliation notes](#27-reconciliation-notes)

---

## 1. Entities

An **entity** is a named collection of typed fields with declared behaviour. Its identity is its **transport name**: a dotted lowercase name such as `res.partner` (party record), `account.move` (journal entry) or `stock.move` (stock move). The transport name is part of the external contract: clients name it, stored references contain it, external identifiers record it, and access rights are granted against it.

### 1.1 Naming rules

1. A transport name is a sequence of segments separated by dots. Each segment consists of lowercase letters, digits and underscores. A name that does not match is rejected at registry build time with **"The _name attribute "** followed by the name **" is not valid."**
2. By convention the first segment names the capability family and the remaining segments name the thing. Two first segments are reserved by the platform: `res` for shared resources that every capability uses (parties, companies, users, groups, countries, currencies, languages) and `ir` for platform infrastructure (the entity, field, view, action, menu, job, sequence and external-identifier catalogues). Every other first segment is the technical name of the package family that owns the entity: `account` for accounting, `stock` for inventory, `sale` for sales, `purchase` for purchasing, `hr` for human resources, `mrp` for manufacturing, `project` for projects, `crm` for customer relationship management, `mail` for messaging, `pos` for the point of sale, `website` for the storefront. Every capability family follows the same pattern with its own technical name. The convention is not enforced: any name that satisfies rule 1 is accepted, whatever its first segment.
3. The **storage name** — the table — is the transport name with every dot replaced by an underscore, unless the entity declares one explicitly. `account.move` stores in `account_move`.
4. Two entities may not share a transport name. A second declaration of the same name is an *extension* of the first, not a new entity; see [inheritance and extension](inheritance-and-extension.md).

### 1.2 The full-name convention used in this specification

Every entity has a **full name in words**, in title case, used in prose: Journal Entry, Journal Item, Stock Move, Sales Order Line, Capability Package. On first mention in a document, the full name is followed by the transport name and the table in code font. The complete dictionary is in [the entity name dictionary](../references/entity-name-dictionary.md).

---

## 2. The three entity kinds

| Kind | Has a table | Records persist | Rows removed automatically | Typical use |
|---|---|---|---|---|
| Persistent | Yes | Yes | No | Every business record, every configuration record, every presentation record |
| Transient | Yes | Yes, but only briefly | Yes, by the housekeeping job | Assistants and dialogs that gather input for one operation |
| Abstract | No | No | Not applicable | Reusable behaviour adopted by other entities |

### 2.1 Persistent entities

A persistent entity owns exactly one table. The table has:

- an integer primary key column named `id` (identifier), fed by a per-table sequence;
- one column per stored field with a column type (see [section 6](#6-the-field-types));
- the audit columns, unless the entity switches them off (see [section 4](#4-the-automatic-fields));
- indexes, foreign keys, check constraints and unique constraints as declared.

A persistent entity may instead be backed by a **stored query**: the entity declares a query expression rather than a table, the schema synchronisation creates a database view from it, and the entity becomes read-only. Such an entity must declare which entities and fields its query reads, so that the unit of work knows what to flush before querying it. Reporting entities that aggregate transactions are built this way.

### 2.2 Transient entities

A transient entity is stored exactly like a persistent one — it has a real table and real rows — but its rows are expected to be short-lived and are removed by the housekeeping job. It is the mechanism behind every multi-step dialog: the dialog's state is a record, so the whole machinery of fields, defaults, on-change, computation and validation works inside a dialog with no special case.

**Housekeeping rules.** The job runs periodically; per entity, cleaning actually happens at most once every five minutes. A transient entity must keep the audit fields of [section 4](#4-the-automatic-fields) switched on, because the housekeeping job decides what to remove by reading the last-update stamp of each row; a transient entity that switched them off could not be cleaned and its declaration is rejected.

Two limits apply, independently of each other, and both are declared per entity with an installation-wide default:

| Setting | Default | Meaning |
|---|---|---|
| Maximum record count | The installation-wide setting for the maximum number of transient rows | The number of rows tolerated in the table. Zero means unlimited. |
| Maximum idle lifetime, in hours | The installation-wide setting for the transient age limit | The idle lifetime a row may reach. Zero means unlimited. |

**The cleaning procedure for one transient entity.**

1. **Age-based pass.** If the maximum idle lifetime is not zero, delete every row whose last-update stamp is strictly older than the age threshold, subject to the floor of step 3:

```formula
age threshold (point in time) = current time (point in time) − maximum idle lifetime (hours) × 3600 seconds per hour
```

2. **Count-based pass.** If the maximum record count is not zero, count the rows of the table. If the count is **strictly greater** than the maximum record count, delete every row whose last-update stamp is strictly older than the count threshold, subject to the same floor:

```formula
count threshold (point in time) = current time (point in time) − 300 seconds
```

3. **The five-minute floor.** Neither threshold is ever placed less than 300 seconds before the current time. A row updated within the last five minutes is never removed, whatever the two settings say, because a user is probably still filling the dialog in. When the maximum idle lifetime is smaller than 300 seconds the floor replaces it.
4. **The per-pass cap.** One pass deletes at most **100,000 rows**. The pass returns the transport name of the entity it cleaned and a flag saying whether rows remain to be examined, so that the caller can schedule a further pass rather than hold a long transaction over a large table.

Note that the count-based pass does not stop at exactly the maximum record count: it removes **everything** above the five-minute floor. Stopping at the limit would leave the table permanently at its limit, so that the very next insertion would trip it again.

**Worked example.** The entity declares a maximum idle lifetime of 0.2 hours (twelve minutes) and a maximum record count of 20. The table holds 55 rows: 10 updated within the last 5 minutes, 12 updated between 5 and 10 minutes ago, and 33 updated more than 12 minutes ago.

```formula
age threshold = current time − 0.2 hours × 3600 seconds per hour = current time − 720 seconds
```

- Step 1 deletes the 33 rows older than 720 seconds. 55 − 33 = 22 rows remain.
- Step 2 counts 22 rows. 22 > 20, so the count-based pass runs with a threshold of the current time − 300 seconds, and deletes the 12 rows in the five-to-ten-minute band. 22 − 12 = 10 rows remain.
- The 10 rows younger than 300 seconds survive, even though 10 is below the limit of 20 only by accident: the floor, not the limit, is what stopped the deletion.
- Both passes together deleted 45 rows, far below the per-pass cap of 100,000, so the pass reports that nothing remains to be examined.

**Relations to and from transient entities.**

- A many-to-one field from a transient entity to a persistent one defaults its deletion behaviour to *cascade* when the field is required and *set null* otherwise, rather than to *restrict*. Without this, a short-lived dialog record could block the deletion of a business record for ever.
- A many-to-one field from a persistent entity to a transient one is forbidden and rejected at registry build time, because the referenced row will be deleted by the housekeeping job.

Transient entities use the same access rights and record rules as persistent ones.

### 2.3 Abstract entities

An abstract entity has no table and no records. It exists to be **adopted**: another entity declares that it inherits from the abstract entity, and receives its fields, its operations and its declared behaviour. The abstract entity is the unit of reuse.

Three consequences:

1. Fields declared on an abstract entity become **real columns on every adopting entity's table**. Ten entities adopting a behaviour that declares three fields produce thirty columns across ten tables.
2. An abstract entity may declare computations, validations, defaults and operations, all of which the adopting entity gets.
3. An abstract entity cannot be searched, read or written; attempting to do so is a defect.

The platform ships abstract entities for: messaging and followers, activities, avatars, images, address formatting, tax-identifier labelling, custom property definitions, and website publication. They are specified with the capability that owns them; the messaging ones in [the messaging model](messaging-model.md).

### 2.4 Choosing the kind

| Question | If yes |
|---|---|
| Does the record represent a fact the business will refer to later? | Persistent |
| Is the record only the state of a dialog being filled in? | Transient |
| Is the thing a set of fields and behaviour that several entities should share? | Abstract |

---

## 3. Entity attributes

Every entity declares the following. Values shown are the defaults.

| Attribute | Default | Meaning |
|---|---|---|
| Transport name | none | Required for a new entity. |
| Description | none | The human-readable name of the entity, shown in error messages, in the entity catalogue and in the access-rights screen. |
| Kind | persistent for a normal entity | See [section 2](#2-the-three-entity-kinds). |
| Table name | transport name with dots replaced by underscores | Overridable. |
| Stored query | none | When set, the entity is backed by a database view rather than a table. |
| Default ordering | by identifier ascending | See [section 16](#16-ordering). |
| Display-name field | the field named `name` if present, otherwise none | See [section 17](#17-display-names-and-name-searching). |
| Name-search fields | the display-name field | The fields searched when a user types a fragment into a relational selector. |
| Parent field | the field named `parent_id` (parent record) | The many-to-one used by the hierarchy operators. |
| Materialised hierarchy | off | When on, the entity gains a stored path field allowing hierarchy queries without recursion. |
| Archive field | the field named `active` if present, else the field named `x_active` if present, else none | See [section 18](#18-the-archive-flag). |
| Fold field | the field named `fold` | The boolean that tells a grouped screen to collapse a group by default. |
| Audit fields | on | See [section 4](#4-the-automatic-fields). |
| Automatic company consistency check | off | When on, every creation and every write runs the company consistency check of [section 15.5](#155-company-consistency). |
| Allow privileged relational commands | on | When off, relational write commands targeting this entity are refused in an elevated environment, so that a privileged operation cannot be tricked into writing a security-sensitive entity through a relation. |
| Declared query dependencies | none | For entities backed by a stored query: which entities and fields must be flushed before querying. |
| Translation export | on | When off, the entity's translatable values are not exported to translation catalogues. |
| Custom entity | off | Marks an entity created by a user rather than by a package. |

### 3.1 Entity attribute validation

- The display-name field, if declared, must exist on the entity; otherwise the registry build fails with **"Invalid _rec_name="** followed by the value and the entity name.
- The archive field, if declared, must exist and must be named either `active` or `x_active`; otherwise the build fails with a message stating that only those two names are supported and that the field must be present.
- The parent field must be a many-to-one to the entity itself when the materialised hierarchy is on.
- An embedded-parent declaration must name, for each parent entity, a many-to-one field to it that exists on this entity; otherwise the build fails with a message stating that the many-to-one field definition for the embedding reference is missing.

---

## 4. The automatic fields

Four fields exist on every persistent and transient entity that keeps audit fields, plus one that exists on every entity at all.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Identifier (`id`) | Integer | The primary key. Read-only, never written by the application, allocated by the table's sequence. Present on every entity, including abstract ones as a conceptual placeholder. Not copied. |
| Created by (`create_uid`) | Many-to-one to User | The acting user at creation. Set automatically; never writable. Deletion behaviour: set null. |
| Created on (`create_date`) | Date and time | The instant of creation, in Coordinated Universal Time, as recorded by the database clock at the moment of the insert. Set automatically. |
| Last modified by (`write_uid`) | Many-to-one to User | The acting user at the most recent write. Updated on every write. Deletion behaviour: set null. |
| Last modified on (`write_date`) | Date and time | The instant of the most recent write, in Coordinated Universal Time. Updated on every write. |

Rules:

1. The acting user recorded is the **environment's acting user**, not the effective one. Elevating privileges does not change who is recorded. This is what makes the audit trail meaningful.
2. The timestamps come from the **database clock**, not the worker clock, so that records written by different workers are ordered consistently.
3. All four are read-only to clients and cannot be supplied on creation or write. A data file that needs a specific creation stamp must set it through a direct statement.
4. Turning audit fields off removes the four columns. It is done only for very high-volume entities where four extra columns per row matter, and it removes the ability to answer "who changed this and when" for that entity.
5. The last-modification stamp is also the basis of the concurrency check a client may perform: a client that read a record at a given stamp can ask the server to refuse a write if the stamp has moved.

One further field exists on every entity and is **not** stored:

| Field (storage name) | Type | Meaning |
|---|---|---|
| Display name (`display_name`) | Text, computed, not stored | The textual representation of the record. See [section 17](#17-display-names-and-name-searching). |

---

## 5. Field attributes common to every type

Every field, whatever its type, accepts the attributes below.

### 5.1 Presentation attributes

| Attribute | Default | Meaning |
|---|---|---|
| Label | derived from the field name | The name shown to users. When not given, it is derived: a name ending in the singular relational suffix or the plural relational suffix loses that suffix, underscores become spaces, and the result is title-cased. Translatable. |
| Help text | none | The tooltip. Translatable. |
| Read-only | false | Purely a client instruction: the client does not offer the field for editing. **It does not prevent a write**; a stored or inversible field can always be written by code and by the transport. Enforcement of who may write belongs to the security model. |
| Required | false | Two effects: the client refuses to submit an empty value, and the column receives a not-null constraint. See [5.5](#55-required-and-not-null). |
| Value label when empty | none | A word the client shows instead of an empty value. |
| Export label translation | true | Whether the field's label is exported to translation catalogues. |

### 5.2 Storage attributes

| Attribute | Default | Meaning |
|---|---|---|
| Stored | true; **false** when the field is computed or related unless explicitly set | Whether the field has a column. A non-stored field is recomputed on every read and cannot be ordered by, and can only be filtered on if it declares a filter rule. |
| Index kind | none | One of: standard, standard excluding empty values, trigram. See [5.4](#54-indexes). Has no effect on non-stored fields. |
| Prefetch group | true | Whether reading this field on one record opportunistically reads it for the whole prefetch set. Automatically false for non-stored fields, fields without a column, and user-defined fields. |
| Copy | true; **false** for to-many fields, computed fields, related fields, company-dependent fields, and any field named `state` | Whether duplication carries the value over. See [section 19](#19-copying). |
| Default export inclusion | false | Whether a re-importable export includes the field by default. |
| Exportable | true | Whether the field may appear in an export at all. |
| Write order | 0 for most types; **10** for monetary fields and for the custom-property value field; **20** for one-to-many and many-to-many fields | The rank that decides in which order the fields of one write operation are applied. Fields are applied in ascending rank; fields of equal rank keep the order in which the entity declared them. |

**Why the write order has exactly those three values.** A single write may name fields of every type, and three groups must not be applied at an arbitrary point:

1. **Rank 0 — everything else.** Plain columns can be applied in any order among themselves, so they go first and are available to whatever follows.
2. **Rank 10 — monetary amounts and the custom-property value field.** A monetary amount is rounded to the precision of *its* currency ([section 6.5](#65-monetary-amount)), and the currency may itself be one of the fields being written. Applying the amount after every rank-0 field guarantees that the currency in force is the one the same write set, not the one the record had before. The custom-property value field shares the rank because the properties it applies are resolved through the container link, which is a rank-0 field of the same write.
3. **Rank 20 — the to-many fields.** A to-many write can delete linked records ([section 7.5](#75-writing-to-a-to-many-field)), and deleting a record forces the unit of work to flush its pending changes to the database. Applying to-many commands last means that flush happens after every scalar field of the same write has already been applied, so the deletion sees a consistent record rather than a half-written one.

A rebuild that applies the fields of a write in declaration order instead of this rank order produces different rounding on monetary amounts and can lose scalar changes made in the same call as a line deletion.

### 5.3 Behaviour attributes

| Attribute | Default | Meaning |
|---|---|---|
| Default | none | A fixed value or a rule producing one. See [section 13](#13-defaults). |
| Computation | none | The name of the rule that computes the value. See [section 8](#8-computed-fields). |
| Dependencies | derived from the computation's declaration | The paths whose change requires recomputation. |
| Context dependencies | none | The context keys whose value changes the field's value; they partition the cache. |
| Computed with elevated privileges | true when the field is stored, false otherwise | Whether the computation runs unrestricted. |
| Precomputed | false | Whether a stored computed field is computed before the row is inserted rather than after. Only meaningful for stored computed or related fields; a warning is recorded otherwise. |
| Recursive | false | Declares that the field's dependency graph reaches itself through a relation. Must be declared explicitly; without it the recomputation of a recursive field is incorrect. |
| Inverse | none | The rule that writes the sources of a computed field when the computed field is assigned. |
| Filter rule | none | The rule that turns a condition on a non-stored field into a condition on stored ones. |
| Related path | none | A dotted path making this field a mirror of another. See [section 10](#10-related-fields). |
| Company-dependent | false | The value varies by company. See [section 11](#11-company-dependent-values). |
| Restricted to groups | none | A comma-separated list of group external identifiers; only members may read or write the field. A single full stop means the field is never accessible except in an elevated environment. |
| Triggers a user default | false | Marks the field as one whose value a user may pin as a personal default on a screen. |
| Aggregate rule | none, except sum for numeric fields | The aggregate a grouped screen applies. One of: row count, distinct row count, conjunction, disjunction, maximum, minimum, arithmetic mean, sum. |
| Group expansion | none | A rule producing the full set of groups to show when grouping by this field, so that empty groups still appear. For a selection field it may simply be switched on, which expands to every selection value. |
| Manual | false | Marks a field created by a user rather than by a package. |

### 5.4 Indexes

| Kind | Behaviour |
|---|---|
| Standard | An ordinary balanced-tree index over the column. Appropriate for a many-to-one and for any column filtered on exact values. |
| Standard excluding empty values | The same index, restricted to rows whose value is not empty. Appropriate when most rows are empty and emptiness is never searched for; much smaller. |
| Trigram | An inverted index over three-character sequences, which makes substring matching fast. Appropriate for text fields searched with the containment operators. Only created when the tenant's database offers trigram support. |
| None | No index. |

Rules:

1. Index creation and removal happen during package installation and update only.
2. A field that becomes indexed gains its index; a field that stops being indexed loses it.
3. Indexes are never created for non-stored fields or for fields with no column.
4. A company-dependent field defaults to the standard-excluding-empty-values index, because its column is a per-company mapping that is empty for most rows.

### 5.5 Required and not-null

Marking a field required does two distinct things:

1. **Client-side**: the field must be filled before the form can be saved.
2. **Database-side**: the column receives a not-null constraint, applied during schema synchronisation.

The second only succeeds if no existing row violates it. The synchronisation therefore proceeds as follows:

1. Add the column if missing, and initialise the existing rows: a row whose value is empty receives the field's default, evaluated **once** for the whole table rather than per row. When the field is computed or related, the rows are **not** filled with the default; they are scheduled for recomputation instead, and the computation supplies the values ([section 9](#9-the-recomputation-algorithm)).
2. Flush the pending writes for the field, so that values assigned earlier in the same build are in the column before the constraint is judged.
3. Attempt to add the not-null constraint, as the **last** step of the schema update for that table.
4. If it fails, record a warning naming the entity and the field, leave the constraint off, and continue. The field remains required at the client level but is not enforced by the database.
5. At the end of the build, verify agreement: for every field the registry believes is not null, confirm that the column really is. The registry's record of which fields are truly not-null is what the filter compiler consults in order to decide whether a condition needs to consider empty values ([record operations and query notation, section 4.6](record-operations-and-query-notation.md#46-empty-values-and-the-three-valued-logic)).

**Removing the required flag** takes effect at once: the not-null constraint is dropped during the next schema synchronisation, without any inspection of the rows, because relaxing a constraint can never fail.

A required field on a **company-dependent** field is contradictory and records a warning: the per-company mapping cannot be non-empty for a company that has no value.

### 5.6 Attribute layering across declarations

When several packages declare the same field on the same entity, the declarations are layered in package order: each declaration's explicitly given attributes override the previous layer's. The layered set is retained so that the field can be rebuilt from scratch during an incremental registry setup.

Rules:

1. If a later declaration uses a **different and incompatible type** from an earlier one, every attribute collected so far is discarded and layering restarts from that declaration. Changing the type of an existing field is therefore possible but drops everything the earlier declarations said.
2. An attribute name the field type does not recognise is accepted but recorded, and a warning is issued naming the field and the attribute unless the entity declares that the attribute is valid for it. This is how a capability adds its own attribute to fields.
3. The derived defaults of [5.2](#52-storage-attributes) and [5.3](#53-behaviour-attributes) are applied **after** layering, so that adding a computation in a later layer correctly makes the field non-stored, elevated and not copied, unless the layer says otherwise.

---

## 6. The field types

Sixteen field types exist. For each: what it stores, how it is stored, what the empty value is, and its own attributes.

### 6.1 Summary table

| Type | Column type | Value in code | Empty value | Notes |
|---|---|---|---|---|
| `boolean` | boolean | true or false | false | |
| `integer` | four-byte integer | whole number | 0 | |
| `float` | double precision, or fixed-point when digits are declared | decimal number | 0.0 | |
| `monetary` | fixed-point | decimal number | 0.0 | Rounded to its currency's precision |
| `char` | variable-length text with a length limit | text | false | |
| `text` | unlimited text | text | false | |
| `html` | unlimited text | markup | false | Sanitised on write |
| `date` | date | calendar date | false | No time, no zone |
| `datetime` | timestamp without zone | instant | false | Always in Coordinated Universal Time |
| `selection` | variable-length text | one of a declared set of codes | false | |
| `reference` | variable-length text | entity name, comma, identifier | false | Points at a record of any entity in a declared set |
| `many2one` | four-byte integer with a foreign key | record set of size zero or one | empty record set | |
| `many2one_reference` | four-byte integer, no foreign key | integer | 0 | Paired with a field naming the entity |
| `one2many` | none (the inverse column holds it) | record set | empty record set | |
| `many2many` | none (an association table holds it) | record set | empty record set | |
| `binary` | unlimited text, or an attachment record | bytes | false | |
| `json` | structured document | structured value | false | |
| `properties` | structured document | mapping of user-defined values | empty mapping | |
| `properties_definition` | structured document | list of user-defined field definitions | empty list | |

Two storage rules override the column type above:

- A **translatable** field's column becomes a structured document holding one value per language ([section 12](#12-translatable-values)).
- A **company-dependent** field's column becomes a structured document holding one value per company ([section 11](#11-company-dependent-values)).

### 6.2 Boolean

Stores truth. The empty value is false; there is no third state. A boolean column is nullable in the database, and a null is read as false.

Filtering: the only meaningful conditions are equality and inequality against true or false. A condition comparing a boolean to anything else is refused with **"Cannot compare "** the field name **" to "** the type **" which is not a collection of length 1"**.

### 6.3 Integer

Stores a whole number in four bytes. The empty value is zero, and **zero and empty are indistinguishable**: the column may be null, and a null reads as zero. This matters for filtering ([record operations and query notation, section 4.6](record-operations-and-query-notation.md#46-empty-values-and-the-three-valued-logic)).

Default aggregate: sum.

### 6.4 Decimal number

Stores a decimal number.

| Attribute | Meaning |
|---|---|
| Digits | Either a pair (total digits, decimal places), or the name of a precision setting record whose value supplies the decimal places with a total of sixteen. |
| Minimum display digits | Either a number or the name of a precision setting record. The minimum number of decimal places the client shows; a value with fewer is padded with zeros. |

Storage depends on the digits attribute:

- **Digits declared** (including explicitly as zero or absent-but-with-minimum-display-digits): fixed-point storage, which preserves every significant digit exactly.
- **No digits declared**: double-precision floating point, which is faster for sums but does not preserve exact decimal values.

Rounding rules:

1. When digits are declared, the value is rounded to the declared number of decimal places **on assignment**, before it reaches the cache, so that everything downstream sees the rounded value. Rounding is half away from zero.
2. When digits are not declared, no rounding happens at all.
3. The precision-setting records are read at the moment the value is converted, so changing a precision setting changes the rounding of subsequently written values but does not retroactively round stored ones.

Three comparison helpers exist and must be reproduced, because business logic depends on them rather than on raw comparison:

```formula
round_to( value , decimal_places )        = the value rounded to that many decimal places, half away from zero
round_to_step( value , step )             = the nearest multiple of step, half away from zero
is_zero( value , decimal_places )         = true when round_to( value , decimal_places ) equals 0
compare( a , b , decimal_places )         = −1 when round_to( a − b , decimal_places ) < 0
                                            0 when round_to( a − b , decimal_places ) = 0
                                           +1 when round_to( a − b , decimal_places ) > 0
```

Worked example. With two decimal places: `compare(0.001, 0.0, 2)` rounds 0.001 to 0.00 and yields 0 — the two values are *equal at that precision*. Direct comparison would have said 0.001 is greater. Every quantity and amount comparison in the business domains uses the precision-aware form.

Default aggregate: sum.

#### The rounding routine in full

Every decimal and monetary rounding in the platform uses one routine. It takes a value, a **step** — either ten raised to the negative number of decimal places, or an arbitrary positive step such as 0.05 — and a tie-breaking method.

1. If the step is zero or the value is zero, return zero.
2. Normalise: divide the value by the step. When the step is below one, compute the reciprocal of the step exactly and multiply by it instead of dividing, which reduces representation error.
3. Compute a tolerance: two raised to the power of the base-two logarithm of the absolute normalised value, minus fifty.
4. Apply the tie-breaking method to the normalised value.
5. Denormalise: multiply the result by the step, or divide it by the reciprocal when the reciprocal path of step 2 was taken.

| Method | Rule applied at step 4 |
|---|---|
| Half up, the default | Round to the nearest whole number, ties away from zero, after adding the tolerance signed like the normalised value |
| Half down | The same, after subtracting the tolerance signed like the normalised value |
| Half even | Let the integral part be the largest whole number not above the normalised value and the remainder be the absolute difference between the normalised value and that integral part. When the absolute difference between one half and the remainder is below the tolerance, the result is the integral part plus its own remainder on division by two; otherwise round to the nearest whole number, ties away from zero |
| Up | Truncate towards zero after adding, signed like the normalised value, one minus the tolerance |
| Down | Truncate towards zero after adding the tolerance signed like the normalised value |

Rounding to the nearest whole number with ties away from zero preserves the sign of a negative zero.

**Worked example one.** Rounding 2.675 to a step of 0.01, half up. The binary representation of 2.675 is slightly below the exact tie. Normalising gives 267.49999999999997; the tolerance is two to the power of the base-two logarithm of 267.5 minus fifty, approximately 0.00000000000024; adding it gives 267.5000000000002, which rounds to 268; denormalising gives 2.68. Without the tolerance the result would have been 2.67.

**Worked example two.** Rounding 1.3 to a step of 0.5, half up: normalising gives 2.6, which rounds to 3, and 3 × 0.5 = 1.5.

**Worked example three.** Rounding 2.5 to a step of 1, half even, gives 2; rounding 3.5 to the same step and method gives 4.

#### Zero test and comparison

```formula
is zero( value , step ) = ( value = 0 )  OR  ( | round( value , step ) | < step )
```

Comparing two values at a step:

1. If the two values are identical, the result is zero.
2. Round each value at the step; the difference is the first rounded value minus the second.
3. If that difference is zero at the step by the test above, the result is zero.
4. Otherwise the result is minus one when the difference is below zero and plus one when it is above.

Comparison rounds **before** subtracting; the zero test of a difference rounds **after** subtracting. The two are not equivalent, and a business rule must state which it uses.

**Worked example.** At two decimal places, comparing 0.006 with 0.002 gives plus one, because 0.006 rounds to 0.01 and 0.002 rounds to 0.00. The zero test of the difference gives true, because 0.006 − 0.002 = 0.004 rounds to 0.00.

#### Euclidean division at a given precision

1. Scale the first value: round it at the step, divide by the step, and take the nearest whole number.
2. Scale the second value the same way.
3. The quotient is the whole-number division of the first scaled value by the second, rounded towards negative infinity.
4. The remainder is the remainder of that division, multiplied by the step and rounded at the step.

```formula
first value  ≈  quotient × second value  +  remainder      at the given precision
```

**Worked example.** Dividing 10.00 by 3.00 at two decimal places: the scaled values are 1000 and 300; the quotient is 3; the remainder of 1000 divided by 300 is 100, which multiplied by 0.01 and rounded gives 1.00. Indeed 3 × 3.00 + 1.00 = 10.00.

#### Rendering a decimal as text

Rendering produces a fixed-point text with exactly the requested number of decimal places. When the value is zero at that precision by the test above, it is first replaced by zero, so that −0.004 at two decimal places renders as 0.00 and never as a negative zero. Rendering must never be used to round: it is a presentation step only.

### 6.5 Monetary amount

Stores an amount of money. Always fixed-point storage.

| Attribute | Default | Meaning |
|---|---|---|
| Currency field | the field named `currency_id` (currency) if present, else the field named `x_currency_id`, else none | The many-to-one to Currency that says which currency the amount is in. |

Rules:

1. The currency field **must exist** on the entity; otherwise the registry build fails with **"Field "** the field **" with unknown currency_field "** the name.
2. On assignment, the value is rounded to the currency's decimal places. If the record's currency field yields more than one currency — which only happens if a caller assigns across records with different currencies — the assignment fails with **"Got multiple currencies while assigning values of monetary field "** followed by the field.
3. On insertion, the currency is resolved from the values being inserted if they contain it, then from the record, so that the first write of a new record rounds correctly even before the currency column is written.
4. Monetary fields are written **after** other fields within one write, so that a currency assigned in the same call is already known when the amount is rounded.
5. A monetary field can only be aggregated in a grouped screen when its currency field can also be aggregated; otherwise amounts in different currencies would be summed. When the currency field cannot be aggregated, the monetary field reports no aggregate.

```formula
stored_amount = round_to( assigned_amount , decimal_places_of( currency_of_record ) )
```

Worked example: assigning 12.345 to an amount whose currency has two decimal places stores 12.35 (half away from zero). Assigning the same value to an amount in a currency with zero decimal places stores 12.

Default aggregate: sum.

### 6.6 Short text

Stores a single-line string.

| Attribute | Default | Meaning |
|---|---|---|
| Maximum length | none | When set, values are truncated to that many characters on assignment. |
| Trim | true | Whether the client strips leading and trailing whitespace before sending. |
| Translatable | false | See [section 12](#12-translatable-values). |

The empty value is false, and the empty string is treated as the empty value for the purpose of filtering: a text column's declared empty value is the empty string, which means a condition testing emptiness also matches rows whose column holds an empty string.

### 6.7 Long text

Stores an unlimited multi-line string. Same attributes as short text except the length limit.

### 6.8 Markup

Stores rich content as markup.

| Attribute | Default | Meaning |
|---|---|---|
| Sanitise | true | Whether the value is cleaned on write. |
| Sanitise tags | true | Remove disallowed elements. |
| Sanitise attributes | true | Remove disallowed attributes. |
| Sanitise style | false | Clean inline style declarations rather than keeping them as they are. |
| Sanitise form | true | Remove form elements. |
| Sanitise conditional comments | true | Remove conditional comments outright. When false, a conditional comment is kept and its content is cleaned like any other content. |
| Sanitise output method | markup | How the cleaned tree is serialised back to text: with markup rules (elements that carry no content are written in their short form, entities are left as they are) or with strict tree rules (every element is closed explicitly). |
| Strip style | false | Remove the style attribute entirely, so that it is never cleaned. Takes precedence over sanitise style. |
| Strip classes | false | Remove class attributes. |
| Sanitise overridable | false | Whether a member of the bypass-sanitising group may store content the cleaner would otherwise reject. See below. |
| Translatable | false, or term-by-term | A markup field may be translated as a whole value or term by term, which translates the text nodes while keeping the markup shared. Declaring a field both translatable and sanitised forces the term-by-term mode ([section 12](#12-translatable-values)). |

**The outgoing-mail preset.** Declaring the sanitise attribute as the outgoing-mail preset rather than as a boolean is a convenience that expands to five settings at once: sanitise on, sanitise tags **off**, sanitise attributes **off**, sanitise conditional comments **off**, and the output method set to strict tree rules. It exists for content that a foreign electronic-mail client will render, where the client's own restrictions are the real defence and where removing conditional comments would break the layouts those clients rely on.

Sanitisation is applied on assignment, so the stored value is already clean. A field that must accept arbitrary markup — a rendering template, for instance — switches sanitisation off, and the capability that owns it is then responsible for controlling who may write it.

**Overridable sanitising.** When the sanitise-overridable attribute is on and the acting user is **not** a member of the bypass-sanitising group, a write does not simply clean the incoming value. The value already stored is read first and examined in two forms: the stored value cleaned by the sanitiser, and the stored value merely normalised (parsed and serialised again without cleaning). If cleaning the stored value would empty it, or if the cleaned form differs from the normalised form, then the stored value contains content that only a privileged user could have put there, and the write is refused with:

> "The field value you're saving (`<entity>` `<field>`) includes content that is restricted for security reasons. It is possible that someone with higher privileges previously modified it, and you are therefore not able to modify it yourself while preserving the content."

where `<entity>` is the human-readable description of the entity and `<field>` is the label of the field. A report of the difference between the cleaned and the normalised form is written to the diagnostic log at informational level, so that an administrator can see what the ordinary user was not allowed to touch. A member of the bypass-sanitising group writes the value unchanged, with no comparison.

Values read from a markup field are markup-safe: a caller that concatenates one with plain text must escape the plain text itself, because the markup field will not be escaped again on rendering.

### 6.9 Date

Stores a calendar date with no time and no zone.

Rules:

1. The stored value is a date. Reading it yields a date; there is no implicit conversion to an instant.
2. When a date must be compared with an instant — for instance when grouping by month in the user's zone — the conversion is explicit and uses the environment's time zone.
3. Assigning a text value parses it as a calendar date in the unambiguous year-month-day form; the client sends that form regardless of the user's display format.

### 6.10 Date and time

Stores an instant, with no zone attached, **always in Coordinated Universal Time**.

Rules, all of which a rebuild must reproduce because they are visible in stored data:

1. Every stored instant is in Coordinated Universal Time. There is no per-record zone.
2. Conversion to and from the user's zone happens only at the presentation boundary, using the environment's time zone.
3. Assigning a value that carries a zone converts it to Coordinated Universal Time and drops the zone.
4. Assigning a bare date converts it to the instant at the start of that day **in the environment's time zone**, then to Coordinated Universal Time. The years one and nine thousand nine hundred and ninety-nine are treated as already in Coordinated Universal Time, to avoid overflow at the extremes of the range.
5. Grouping by a time granularity converts to the environment's zone, truncates, and converts back, so that a day boundary means the user's midnight, not midnight in Coordinated Universal Time.

### 6.11 Selection

Stores one code from a declared set.

| Attribute | Meaning |
|---|---|
| Selection | The ordered list of pairs (stored code, label). Labels are translatable. May instead be the name of a rule producing the list, which makes the set dynamic. |
| Selection addition | Used by an extension to add codes to an existing selection without redeclaring the whole list. Each added pair is placed after the code it is declared to follow, or at the end. |
| Order of values | The declared order, which is the order a client offers them in. |
| Validate | true | Whether an assigned code is checked against the set. |

Rules:

1. The stored value is the **code**, never the label. Codes are part of the external contract and are reproduced exactly throughout this specification.
2. Assigning a code not in the set is refused with **"Wrong value for "** the entity and field **": "** the value, unless validation is switched off.
3. The empty value is false. A selection field that is required must therefore always hold a code.
4. Codes are reflected into the self-describing catalogue as records, which is what lets an extension add a value and lets the label be translated.
5. Removing a code from the set may leave rows holding it. A selection value record may declare a deletion behaviour: leave the rows as they are, set the field empty, set it to another code, or delete the rows.

### 6.12 Polymorphic reference

Stores a pointer to a record of **any** entity from a declared set, as a single text column holding the entity's transport name, a comma, and the record identifier.

| Attribute | Meaning |
|---|---|
| Selection | The set of entities that may be pointed at, as (transport name, label) pairs, exactly like a selection field. May be a rule producing the list. |

Rules:

1. There is **no foreign key**. Deleting the pointed-at record leaves a dangling pointer; reading it yields an empty record set of that entity, and the pointer is not cleaned up automatically.
2. The entity part must be one of the declared ones; assigning another is refused as for a selection field.
3. Filtering on a polymorphic reference compares the composite text, so a filter must construct the same composite form.

### 6.13 Reference by identifier with a separate entity field

Stores only the identifier, as an integer with **no foreign key**, and names another field of the record that holds the entity's transport name.

| Attribute | Meaning |
|---|---|
| Entity field | The name of the field on the same record that holds the transport name of the pointed-at entity. |

This pairing is used where the entity part is needed for filtering and grouping in its own right — the attachment's owner, the message's subject, the external identifier's target. Like the polymorphic reference, it has no referential integrity; the capability that uses it is responsible for cleaning up.

### 6.14 Binary content

Stores bytes.

| Attribute | Default | Meaning |
|---|---|---|
| Stored as an attachment | true | When true the bytes live in an attachment record, and the entity's column holds nothing. When false the bytes are stored inline in the column, encoded. A binary field declared not stored is forced to not-attachment as well, since there is nothing to attach. |
| Maximum width | none (no limit) | For the image specialisation: the greatest stored width, in pixels. |
| Maximum height | none (no limit) | For the image specialisation: the greatest stored height, in pixels. |
| Verify resolution | true | For the image specialisation: whether the total pixel count of the decoded image is checked against the platform maximum. |

Rules:

1. With attachment storage, writing a value creates or replaces an attachment record whose owner is this record and this field, and whose content is deduplicated by digest in the file store. The attachment row records the field name as its name, the transport name of this entity as the referenced entity, the field name as the referenced field, the identifier of this record as the referenced identifier, and the value as its content. Reading yields the bytes. Writing an empty value deletes the attachment rows; records that had no row get one created; records that had one get it updated.
2. Reading attachment-backed values issues **one** search over the attachment catalogue with elevated rights, restricted to this entity, this field and the identifiers being read, and takes either the content or the recorded byte length depending on rule 3.
3. Reading a binary field with the size-instead-of-content context key set — either the general key or the field-specific variant naming this field — yields a human-readable size instead of the bytes, so that a list screen can show "3.20 kB" without transferring the content. The field declares that key as a context dependency, so the cache holds one entry per value of the key. A write always switches the context back to full content, because a size string is not a value that can be stored.
4. Values are exchanged as text in the base-64 alphabet. A byte value is accepted as it stands. A text value that is neither valid base-64 nor plain seven-bit text is refused with **"ASCII characters are required for "** the value **" in "** the field name.
5. **The vector-image guard.** When the first character of the value is `P` or `<` — the base-64 opening and the plain-text opening of a markup document — the value is decoded and its content type is detected. If the content type is a scalable vector image and the acting user is not a member of the settings group, the write is refused with **"Only admins can upload SVG files."** A vector image is executable content in a way that a raster image is not, which is why it is restricted to administrators rather than sanitised.
6. Binary fields are never prefetched, because transferring content for records nobody asked about is expensive.
7. **Filtering** an attachment-backed binary field supports existence only. The condition "field is empty" compiles to "no attachment row exists for this record and field" and "field is not empty" to "an attachment row exists". Any other operator is refused: the attempt is recorded in the diagnostic log and the condition is replaced by the constant true, so the search returns everything rather than failing. A pattern operator is refused outright with **"Cannot use like operators with binary fields"**.
8. Export renders nothing readable for a binary value and is normally suppressed on such fields.

**The image specialisation.** An image is a binary with the three extra attributes above and a processing step applied on create and on write:

1. If the field is read-only **and** either declares no size limit or follows a path to another image field carrying the same limits, the value is stored exactly as received, with no decoding.
2. The value is decoded from base-64. A value that is not valid base-64 is refused with **"Image is not encoded in base64."**
3. If the decoded content is already in an efficient web image format: with no size limit declared it is stored unchanged; otherwise the platform first looks for an already-resized variant among the attachments of the file store, matching on the content checksum of the original together with a description of the form `resize: <largest limit>`, where `<largest limit>` is the larger of the declared maximum width and maximum height in pixels. A matching variant is stored instead of the original; when none is found the original is stored.
4. Otherwise the image is resized to fit inside the declared maximum width and maximum height, preserving the aspect ratio, the resolution is verified when the verify-resolution attribute is on, and the result is encoded back to base-64.
5. **The resolution ceiling.** When the resolution is verified, an image whose width in pixels multiplied by its height in pixels exceeds **50,000,000 pixels** is refused rather than resized. The ceiling is a defence against a small compressed file that expands to an image large enough to exhaust memory, so it is applied to the decoded dimensions before any resizing work is done.

```formula
total resolution (pixels) = decoded width (pixels) × decoded height (pixels)
```

  A 9,000 × 6,000 photograph is 54,000,000 pixels and is refused; an 8,000 × 6,000 photograph is 48,000,000 pixels and is accepted and then resized to the declared limits.

6. When the field follows a path to another field, the **unprocessed** value is kept in the cache while the write-back to the target field runs, and the cache is corrected with the resized value once the write-back has finished. This is what lets the target field store the full-size image while this field stores the reduced one.
7. When an image assigned to an in-memory record fails processing, the assignment is **silently skipped** rather than refused, because a client that read the field with the size-instead-of-content key set will send the size string back, and refusing would make an ordinary round trip fail.
8. An image field requires its entity to keep the audit fields of [section 4](#4-the-automatic-fields), because the resized variants are cached against the last-update stamp of the record; a definition that breaks this rule is reported as invalid at registry build time.
9. The image specialisation exposes derived fields at fixed sizes (for example a 1024-pixel, a 512-pixel, a 256-pixel and a 128-pixel variant) computed from the original, so that a client asks for the size it needs instead of the full image.

### 6.15 Structured document

Stores an arbitrary structured value — nested mappings, lists, numbers, strings, booleans — as a structured document column.

Rules:

1. The value is stored and returned as-is. No schema is enforced.
2. Structured-document fields are not translatable and not company-dependent.
3. Filtering on them is limited to the operations the database offers on structured documents; the platform exposes equality on the whole value and containment on a named key.
4. The empty value is false.

### 6.16 Custom properties

Two paired types let a user add fields to records of an entity without changing the schema.

**The definition field** holds the ordered list of user-defined field definitions, stored as a structured document. It lives on a *container* record — a project, a sales team, a category — so that all records belonging to that container share the same set of user-defined fields.

**The value field** holds the values for one record, as a mapping from definition name to value. It declares which field of the record points at the container and which field of the container holds the definitions.

| Attribute of the value field | Meaning |
|---|---|
| Definition record | The name of the many-to-one on this entity that leads to the container. |
| Definition record field | The name of the field on the container that holds the definitions. |

#### The grammar of one definition

Each entry of the definition list is a mapping. Exactly twelve keys are allowed, and a capability package may register further allowed keys of its own for the properties it introduces.

| Key | Required | Restricted to | Meaning |
|---|---|---|---|
| `name` | yes | — | The stable identifier of the property, used as the key of the value mapping. Between 1 and 512 characters, made only of lowercase letters, digits and underscores. |
| `type` | yes | — | The data type, one of the fourteen discriminators below. |
| `string` | no | — | The label shown to the user. |
| `default` | no | — | The value applied to a child record when it is attached to this container and leaves the property unset. |
| `comodel` | no | link to one record, link to several records | The transport name of the target entity. |
| `domain` | no | link to one record, link to several records | A filter restricting the candidate records, stored as text. |
| `selection` | no | closed list | The ordered list of value-and-label pairs. |
| `tags` | no | multi-valued label list | The ordered list of value, label and colour-index triples. |
| `currency_field` | no | monetary amount | The name of the property that holds the currency of this amount. |
| `suffix` | no | — | A unit shown after the value. |
| `view_in_cards` | no | — | Whether the property is shown on compact card screens. |
| `fold_by_default` | no | — | Whether a grouping on this property is collapsed by default. |

The fourteen stored type discriminators, with the meaning of each:

| Stored value | Meaning |
|---|---|
| `boolean` | True or false. |
| `integer` | A whole number. |
| `float` | A decimal number. |
| `char` | Single-line text. |
| `text` | Long text. |
| `html` | Rich text carrying markup. |
| `date` | A calendar date. |
| `datetime` | A date and time. |
| `monetary` | An amount in the currency named by the `currency_field` key. |
| `many2one` | A link to one record of the entity named by the `comodel` key. |
| `many2many` | A link to several records of the entity named by the `comodel` key. |
| `selection` | One value of the closed list given by the `selection` key. |
| `tags` | Any number of values of the label list given by the `tags` key. |
| `separator` | A visual separator that carries no value at all; it exists to group the properties that follow it. |

#### Validation of a definition list

A definition list is validated whenever it is written. Each check states its refusal:

1. A key that is not one of the allowed keys, in a definition, is refused with **"Some key are not allowed for a properties definition ["** the offending keys, separated by a comma and a space, **"]."**
2. A definition that omits `name` or `type` is refused with **"Some key are missing for a properties definition ["** the missing keys, separated by a comma and a space, **"]."**
3. A key that belongs to another type than the declared one — a `selection` key on a text property, a `currency_field` key on a date property — is refused with **"Invalid property parameter '"** the key **"'"**. The restriction column of the table above lists which key belongs to which type.
4. A name that is empty, longer than 512 characters, or made of anything but lowercase letters, digits and underscores, is refused with **"Wrong property field value name '"** the name **"'."**
5. A name that is empty or repeats a name already used in the same list is refused with **"The property name '"** the name **"' is not set or duplicated."**
6. A rich-text property whose name does not end with the marker `_html` is refused with **"HTML property name should end with `_html`."**, and a property of any other type whose name does end with that marker is refused with **"Only HTML properties can have the `_html` suffix."**
7. A type that is not one of the fourteen discriminators is refused with **"Wrong property type '"** the value **"'."**
8. A `comodel` key naming an entity that does not exist, or that is transient or abstract, is refused with **"Invalid model name '"** the value **"'"**.

#### Reading a definition back

The definition stored on the container may have aged: a package that declared the target entity may have been removed, a field named in a filter may have gone. Reading the definition therefore degrades rather than fails:

1. A definition that does not carry a non-empty value for both required keys is **skipped**: it is not returned at all.
2. A link property whose target entity no longer exists has its `comodel` key set to false and its `domain` key removed, so that the client shows the property as unconfigured rather than raising.
3. A link property whose filter no longer parses or no longer validates against the target entity — because a field it names has gone — has its `domain` key removed and keeps its target.
4. A closed-list property or a label-list property with no options at all reports an empty option list rather than no key, so that a client never has to distinguish "no options" from "not a list property".

#### The value field in detail

The value field is always stored, always editable, never copied when a record is duplicated, never prefetched, carries write order 10 ([section 5.2](#52-storage-attributes)) and is precomputed. It is declared with a two-part path naming the many-to-one that leads to the container and the definition field on the container, and it is automatically a computed field depending on that many-to-one, which is what makes a change of container re-resolve the properties.

Rules:

1. Reading the value field returns, for each defined property, its definition merged with the record's value under an added value key, so that a client can render it without a second call. A link value additionally carries the display name of the records it points at.
2. Writing accepts either the stored mapping of name to value or the merged list form. The merged list form additionally rewrites the container's definition when an entry carries the definition-changed marker, which is how a user adds a property from the record rather than from the container.
3. **Defaults on attachment.** When a record is attached to a container, every property of the container's definition whose value on the record is unset takes a default, in this order: the context key `default_<value field>.<property name>` when the context carries it, otherwise the `default` key of the definition when that key is non-empty, otherwise nothing.
4. Changing the container of a record re-resolves which properties apply; values whose property no longer exists are dropped by a cleaning pass that runs after every write, load or import that could have changed the definitions.
5. Properties can be filtered and grouped: a condition names the value field, a full stop and the property name. Grouping by a property of relational or closed-list type expands to the defined values.
6. Properties are not columns. No index exists for them beyond what the structured-document column offers, so filtering on a property is slower than filtering on a field.
7. **Rich-text properties are cleaned** on write with a fixed set of sanitising settings, which the container's declaration cannot change: tag cleaning on, attribute cleaning on, form removal on, conditional-comment removal on, style cleaning off, style stripping off, class stripping off ([section 6.8](#68-markup) explains each). The same cleaning is applied to the `default` key of a rich-text property when the definition is written.

---

## 7. Relational fields in depth

### 7.1 Attributes shared by all relational fields

| Attribute | Default | Meaning |
|---|---|---|
| Target entity | required | The transport name of the entity at the other end. Required except on a related field or an extension of an existing field. |
| Client-side filter | none | A filter restricting the candidate values a client offers. May be a literal filter or an expression the client evaluates against the record being edited, which lets the candidates depend on other fields. |
| Client-side context | none | Context keys the client applies when opening or searching the target. |
| Bypass access checks on the target | false | When true, the record rules of the target entity are **not** applied when this relation is traversed in a filter. See [7.6](#76-bypassing-access-on-traversal). |
| Company consistency | false | When true, the field participates in the company consistency check of [section 15.5](#155-company-consistency). |

### 7.2 Many-to-one

Stores the identifier of at most one record of the target entity, in an integer column with a foreign key.

| Attribute | Default | Meaning |
|---|---|---|
| Deletion behaviour | *restrict* when the field is required, *set null* otherwise; but from a transient entity to a persistent one, *cascade* when required and *set null* otherwise | What happens to this record when the pointed-at record is deleted. |
| Embeds the parent | false | Set implicitly when the entity declares this field as an embedded-parent reference. Implies bypassing access checks on traversal. See [inheritance and extension](inheritance-and-extension.md#4-embedding-a-parent-record). |

Deletion behaviours:

| Value | Effect when the target is deleted |
|---|---|
| `set null` | This record's field becomes empty. |
| `restrict` | The deletion is refused while this record exists. |
| `cascade` | This record is deleted too. |

Validation at registry build time:

1. A required field declaring *set null* is rejected with **"The m2o field "** the field **" of model "** the entity **" is required but declares its ondelete policy as being 'set null'. Only 'restrict' and 'cascade' make sense."**
2. A field declaring *restrict* whose target is one of the self-describing catalogue entities is rejected, because restricting there would make a package impossible to remove. The message names the field, the entity and the target and states that the restrict mode is not supported for that kind of target.
3. A field from a persistent entity to a transient entity is rejected with **"Many2one "** the field **" from Model to TransientModel is forbidden"**.

Reading a many-to-one yields a record set of size zero or one, in the same environment. Assigning accepts a record set of size at most one, a bare identifier, or the empty value.

### 7.3 One-to-many

Holds the set of records of the target entity whose named inverse field points at this record. It has **no column**: the relation is stored entirely in the inverse many-to-one's column.

| Attribute | Default | Meaning |
|---|---|---|
| Inverse field | required | The name of the many-to-one on the target entity that points back. Required except on a related field or an extension. |
| Copy | **false** | One-to-many fields are not duplicated by default; see [section 19](#19-copying). |

Rules:

1. The inverse field must exist on the target entity; otherwise the build fails with the inverse field name, the field and the target named, stating that the inverse does not exist.
2. The inverse relationship is registered in both directions, so that writing either side updates the other's cache.
3. A one-to-many whose inverse is a reference-by-identifier field is allowed for reading, but assigning the one-to-many does **not** invalidate the inverse, because the pair (entity field, identifier field) is not a single relation the platform can invert.
4. Ordering of the values follows the target entity's default ordering, unless the field declares its own.
5. Deleting the owning record does **not** automatically delete the target records; that is governed by the inverse many-to-one's deletion behaviour, which is *cascade* on almost every line entity.

### 7.4 Many-to-many

Holds a set of records of the target entity, stored in an **association table** with two integer columns.

| Attribute | Default | Meaning |
|---|---|---|
| Association table | derived | The table holding the pairs. |
| First column | derived | The column referring to this entity. |
| Second column | derived | The column referring to the target entity. |
| Deletion behaviour of the second column | cascade | What happens to the association row when the target record is deleted. Only *cascade* and *restrict* are accepted; anything else is rejected with a message naming the field, the entity and the value and stating that only those two make sense. |

**Derivation of the names**, when not given explicitly:

```formula
association_table = alphabetically_first( this_table , target_table ) + "_" +
                    alphabetically_second( this_table , target_table ) + "_rel"
first_column      = this_table + "_id"
second_column     = target_table + "_id"
```

Sorting the two table names alphabetically makes the derived name **symmetric**: both ends derive the same table, which is how the two sides of a many-to-many find each other. The derivation is impossible when the two entities are the same, because the two derived column names would collide; a self-referencing many-to-many must therefore name its table and columns explicitly.

**Uniqueness of the association schema.** Two many-to-many fields may not use the same table and column pair, because writing one would silently change the other. The rule is enforced at registry build time, with three exceptions:

1. The same field, obviously.
2. Two fields between the **same** pair of entities whose table and columns are both **explicitly** declared. Declaring them explicitly is taken as a statement that the author intends the sharing — this is how a symmetric relation is expressed.
3. Two fields where at least one belongs to an entity that is not backed by a real table.

Otherwise the build fails with **"Many2many fields "** the first **" and "** the second **" use the same table and columns"**.

**Finding the inverse.** The inverse of a many-to-many is any many-to-many declared with the same table and the two columns **swapped**. Both directions are registered so that writing one updates the other's cache.

A non-stored many-to-many has no table and no columns; its value is produced entirely by its computation.

### 7.5 Writing to a to-many field

To-many fields are not assigned wholesale; they are modified by an ordered list of **commands**. Each command is a triple: a code, a record identifier, and a payload.

| Code | Name | Identifier | Payload | Effect |
|---|---|---|---|---|
| 0 | Create | ignored | The values of a new record | Create a record of the target entity and link it. For a one-to-many, one new record per owning record, each linked to its own owner. For a many-to-many, **one** new record shared by every owning record. |
| 1 | Update | the target record | The values to write | Write those values on that target record. |
| 2 | Delete | the target record | ignored | Unlink and delete the target record. For a many-to-many, deletion may be refused if the record is still linked elsewhere. |
| 3 | Unlink | the target record | ignored | Remove the link only. For a one-to-many this sets the inverse empty, or deletes the target if the inverse declares *cascade*. For a many-to-many it removes the association row. |
| 4 | Link | the target record | ignored | Add a link to an existing target record. |
| 5 | Clear | ignored | ignored | Remove every link, as if Unlink had been applied to each. |
| 6 | Set | ignored | The complete list of target identifiers | Replace the whole set: unlink everything not in the list, link everything in it that is not already linked. |

Rules:

1. Commands are applied **in order**. A Clear followed by three Links is not the same as three Links followed by a Clear.
2. A Create command inside a one-to-many may itself carry commands for a nested to-many, to any depth. This is how a whole document with its lines and their sub-lines is created in one call.
3. Commands are the only way to modify a to-many over the transport, because a bare list of identifiers could not express "create these and keep those".
4. When the acting environment is elevated, commands targeting an entity that forbids privileged relational commands are refused. This prevents a privileged operation from being made to write, say, a group membership through a relation it was not meant to touch.
5. Applying Update or Delete through a to-many performs the corresponding access check on the target records, even when the command arrives as a default value.
6. Three shorthands are accepted in place of a command list, and each expands to exactly one command: a record set of the target entity expands to a Set over its identifiers; a plain list of identifiers expands to a Set over those identifiers; an empty value expands to a Clear.

#### Execution order for a one-to-many

The commands of one list are not simply executed one after another: some take effect at once and some accumulate, because a one-to-many is written through the inverse many-to-one on the target entity and the platform issues as few statements as it can. The list is walked once in order, and each command is handled as follows:

1. **Update** is executed immediately: the values are written on the named target record.
2. **Delete** appends the named record to an accumulated **delete list**.
3. **Unlink** depends on the deletion policy of the inverse many-to-one. When that policy is *cascade*, the record is appended to the delete list, because emptying the inverse would delete it anyway. Otherwise the inverse field of the named record is emptied **immediately**, and the record survives with no owner.
4. **Create** appends its values to an accumulated **create list**, each entry carrying the inverse many-to-one already set to the owning record, so that the new record is linked by construction rather than by a second write.
5. **Link** appends the named record to an accumulated **link list**, attached to the **last** record of the written set. Linking one target to several owners is impossible for a one-to-many — the target has one inverse value — so when several records are being written at once the last one wins.
6. **Clear** and **Set** first flush the accumulated delete, create and link lists, so that the state they act on is the real one; then they unlink every currently linked record that is not named in the command, and link the named records to the **last** record of the written set.
7. **The creation-time safety rule.** While a record is being created, a Clear or a Set is not allowed to unlink records that already exist and point elsewhere: nothing has yet been created or linked in this write, so an unqualified unlink would silently steal or orphan other owners' lines. In that situation the command **degrades to a Link** of the named identifiers, and the unlink half is not performed.
8. At the end of the walk the accumulated lists are flushed in one fixed order: **deletions first, then creations, then links**. Deleting before creating keeps a uniqueness constraint from firing between a line being removed and a line being added with the same key. Linking last means a record created earlier in the same list can be linked by a later command. Linking a record whose inverse value cannot be read fails the whole operation rather than half of it.

#### Execution order for a many-to-many

A many-to-many is written through an association table, which can be rewritten as a set difference, so the commands are replayed in memory before anything reaches the database:

1. The current set of links is read for every record of the written set.
2. The commands are replayed in order against an in-memory copy of that set: Link adds an identifier, Unlink removes one, Clear empties the set, Set replaces it wholesale, Update writes on the target immediately as for a one-to-many, Create schedules the creation of one target record shared by every owner and adds its identifier once it exists, and Delete schedules the deletion of the target record and removes its identifier from **both** the old set and the new one, so that no pair referencing it is written.
3. When the execution does not carry elevated rights, the **read** permission of every **newly added** target record is checked — not of the targets that were already linked, which the owner could already see. A failure raises **"Failed to write field "** followed by the transport name of the entity, a full stop and the field name, then a line break and the access message that explains which record was refused.
4. The pairs to add are inserted, ignoring a conflict with a pair that already exists, so that linking twice is harmless.
5. The pairs to remove are deleted, grouped as a union of cartesian products — each group being a set of owners crossed with a set of targets — so that removing many links issues few statements rather than one per pair.
6. Every target record whose link set changed is marked as modified on the **inverse** field, which schedules the recomputation of anything depending on it ([section 9](#9-the-recomputation-algorithm)). Without this step a total computed on the other side of the relation would keep a stale value.

### 7.6 Bypassing access on traversal

When a filter traverses a relation — "invoices whose party is in Belgium" — the inner condition is normally evaluated with the target entity's record rules applied, which means a user who cannot see a party cannot match invoices through it. A relational field may declare that access checks are bypassed on traversal, which evaluates the inner condition against all target records.

Rules:

1. Bypassing is a **performance and correctness** choice made by the field's author, not by the caller. It is declared on the field.
2. Embedding a parent implies bypassing, because the parent record is conceptually part of this record.
3. In an elevated environment every traversal bypasses, since no rules apply at all.
4. Bypassing changes results, not only speed: a rebuild must reproduce the declaration per field.

### 7.7 What reading a relational field yields

| Field | Result |
|---|---|
| Many-to-one | A record set of size zero or one, sharing the reader's environment, whose prefetch set is the union of that field's values over the reader's prefetch set. |
| One-to-many | A record set, ordered by the target's default ordering or the field's own, with the same prefetch behaviour. |
| Many-to-many | The same. |
| Polymorphic reference | A record set of the pointed-at entity, of size zero or one. |
| Reference by identifier | The bare integer. |

### 7.8 Self-references and the materialised ancestor path

An entity may declare a **parent field**, a many-to-one to itself, and may in addition enable **materialised ancestor path storage**.

- With path storage enabled the entity carries a short-text field, `parent_path` — the materialised ancestor path — which must be indexed. It holds the ancestor chain of the record as the root identifier, a separator, each intermediate identifier and a separator, and finally the record's own identifier and a separator; it always ends with a separator.
- On creation the path is the parent's path concatenated with the record's identifier and a separator. A record with no parent gets its own identifier and a separator.
- On a write of the parent field, the records whose parent actually changes are determined **before** the write, by comparing the stored parent with the new one. After the write, each such record and **all its descendants** have their path rewritten by replacing the old prefix with the new parent's path. Descendants are selected by the range from the node's own path up to but excluding the node's path with its last character replaced by the next character, which selects exactly the subtree while remaining index-friendly.
- **Cycle guard.** Before rewriting, the new parent's ancestors are extracted from its path; when any of them is one of the records being moved, the write is refused with **"Recursion Detected."**
- A generic cycle test exists for entities without path storage, and for any self-referencing many-to-one or many-to-many field: it walks the relation level by level until it either exhausts the graph or revisits a starting record. The traversal runs with elevated rights and with archived records visible, so that an archived intermediate record cannot hide a cycle.
- The materialised ancestor path is rejected from creation and write value maps exactly like the audit fields.
- The path is what makes the hierarchy operators of the filter grammar resolve without an iterative search ([record operations and query notation, section 4.9](record-operations-and-query-notation.md#49-hierarchy-operators)).

---

## 8. Computed fields

### 8.1 Definition

A **computed field** declares a rule that derives its value from other data instead of reading it from a column. The rule receives a record set and assigns the field on every record of it.

Two orthogonal choices define the four kinds of computed field:

| Stored | Inversible | Behaviour |
|---|---|---|
| No | No | Recomputed on every read, in the reading environment. Cannot be ordered by. Filterable only if a filter rule is declared. |
| No | Yes | The same, but assignable: assigning it runs the inverse rule, which writes the underlying sources. |
| Yes | No | Has a column. Computed when a dependency changes, written at flush. Orderable and filterable like a plain field. Read-only by default. |
| Yes | Yes | The same, and assignable: assigning it writes the column *and* runs the inverse rule. |

### 8.2 Attribute defaults implied by declaring a computation

Declaring a computation changes four defaults, and a rebuild must apply the same derivation:

1. **Stored** becomes false unless explicitly set true.
2. **Computed with elevated privileges** becomes equal to whether the field is stored. A stored computed field is computed unrestricted; a non-stored one is computed as the acting user. The reasoning: a stored value is shared by every reader, so it must not depend on who triggered the computation; a non-stored value is produced per reader, so it may.
3. **Copy** becomes false, unless the field is both stored and explicitly not read-only.
4. **Read-only** becomes true unless an inverse rule is declared.

A single rule may compute **several** fields at once; they are grouped, and computing any one of them computes all of them in one invocation. Fields computed by the same rule must agree on whether the computation is elevated; disagreeing produces a warning naming the entity and the fields and asking either to make them agree or to use separate rules.

### 8.3 Declared dependencies

A computation declares the paths whose change requires recomputation. A path is one or more field names separated by full stops; every segment but the last must be a relational field, and traversal follows it.

| Path form | Meaning |
|---|---|
| `quantity` | The field on the same record. |
| `line_ids.subtotal` | The subtotal of any line; changing any line's subtotal, adding a line or removing one all trigger. |
| `order_id.partner_id.country_id` | Two relations deep. |
| `line_ids` | The set itself; adding or removing a line triggers, changing a line's fields does not. |

Rules:

1. Every segment must name a field that exists at that point in the path; otherwise the build fails naming the path and the field.
2. Dependencies declared on the **field** take priority over dependencies declared on the computation rule. This lets an extension redeclare what a field depends on without touching the rule.
3. When several rules implement the computation because several packages contributed one, the dependencies of **all** of them are collected and unioned. An extension therefore adds dependencies rather than replacing them.
4. Dependencies may be produced by a rule rather than declared literally, which is how a field can depend on a set determined at registry build time.
5. A dependency on a field of the **same** entity reached through a relation back to itself — a parent's total depending on the children's totals which depend on the parent — must be declared **recursive**. Without the declaration, the recomputation traversal stops at the first level and produces wrong results.

### 8.4 Context dependencies

A field may declare that its value depends on named context keys. The consequences:

1. The cache for that field is partitioned by the tuple of those keys' values, so the same record can hold different values of the field for different contexts within one transaction.
2. Every key's value must be hashable; a list is converted to a tuple, and anything unhashable is refused with the message given in [architecture, section 6.6](architecture.md#66-the-context).
3. Three keys have special resolution when used as cache keys:
   - the company key resolves to the current company's identifier, not to the raw context value;
   - the acting-user key resolves to the acting user's identifier alone for an elevated computation, and to the pair (identifier, elevated flag) otherwise;
   - the archive-filter key resolves to the context value if present, else the field's own declared value for it, else true.
4. A **stored** field may not depend on context, because a column holds one value. Declaring both produces a warning and the context dependency is ignored; a stored translatable field additionally warns that it cannot depend on context.

### 8.5 The inverse rule

An inverse rule makes a computed field writable. It receives the records whose computed field was just assigned and writes whatever underlying data makes the computation produce that value.

Rules:

1. While the inverse runs, the computed field is **protected** on those records, so that writing its sources does not immediately recompute and overwrite the assigned value.
2. For a **stored** inversible computed field, assignment writes the column directly *and* runs the inverse. Both happen; the column is not recomputed afterwards.
3. For a **non-stored** inversible computed field, only the inverse runs; the next read recomputes from the sources.
4. An inverse that cannot honour the assignment must refuse explicitly. Silently ignoring an assignment produces a field that appears editable and is not.

### 8.6 The filter rule

A non-stored computed field has no column, so a condition on it cannot be compiled. A **filter rule** takes the operator and the value of the condition and returns a filter over stored fields that means the same thing.

Rules:

1. The rule receives the condition **after basic normalisation**: equality has become membership, boolean conditions have been normalised to membership against a single-element set, and negations have been pushed where possible.
2. The rule may answer "not supported" for an operator it cannot express. The platform then tries the semantically equivalent alternatives — in particular, the positive form of a negative operator, wrapping the result in a negation.
3. The returned filter replaces the condition in place.
4. A **stored** field may also declare a filter rule. It is then invoked to rewrite conditions on the field, which is how a field can sanitise or broaden what a user may search for.
5. A field with neither a column nor a filter rule cannot be filtered on at all; a condition naming it is refused.

### 8.7 Precomputation

A stored computed field may be marked **precomputed**, which computes it *before* the row is inserted rather than after.

Rules and limits:

1. Precomputation only happens when the creation supplies **neither an explicit value nor a default** for the field. A default disables it.
2. Precomputation is pointless on a non-stored field and produces a warning; likewise on a field that is neither computed nor related.
3. Precomputation is a trade-off: it avoids a second statement per creation, but it computes record by record rather than in a batch, so it loses the benefit of batching and prefetching. It pays off on the lines of a document created in one call, and costs on records created one at a time in a loop.
4. A precomputed field whose computation depends on another precomputed field of the same record is computed in dependency order.

### 8.8 Derived capabilities of a computed field

Whether a computed field can be filtered on, sorted by, grouped by or aggregated is determined by attempting to express it as a query term.

| Capability | Rule |
|---|---|
| Filterable | True when the field is stored, or when it declares a filter rule |
| Sortable | True when the field has a column and is stored; otherwise true when the field is embedded from a parent and the parent field is sortable; otherwise true when an ordering term can be produced for the field, and false when producing one fails |
| Groupable | The same, using a grouping term; for a date or date-and-time field the attempt is made with the month granularity |
| Aggregatable | None when no aggregate is declared; the declared aggregate when the field has a column and is stored; otherwise the parent's aggregate when the field is embedded and the parent is aggregatable; otherwise the declared aggregate when an aggregate term can be produced, and none when producing one fails |

The capabilities are computed once when the registry is built and are served to clients with the field description ([section 21](#21-field-metadata-exposed-to-clients)), which is how a client knows not to offer grouping on a field that cannot be grouped.

### 8.9 Failure catalogue of derivation

| Situation | Message |
|---|---|
| A dependency path names an unknown field | "Wrong @depends on '\<rule\>' (compute method of field \<transport name\>.\<field\>). Dependency field '\<name\>' not found in model \<transport name\>." |
| A dependency path contains the identifier | "Compute method cannot depend on field 'identifier'." |
| A self-dependency that was not declared | "Field \<transport name\>.\<field\> should be declared with recursive=True" |
| Precomputation is not feasible because a dependency is not precomputed | "Field \<transport name\>.\<field\> cannot be precomputed as it depends on non-precomputed field \<transport name\>.\<field\>" |
| An intermediate dependency step is not filterable | "Field \<transport name\>.\<field\> in dependency of \<transport name\>.\<field\> should be searchable. This is necessary to determine which records to recompute when \<transport name\>.\<field\> is modified. You should either make the field searchable, or simplify the field dependency." |
| A computation assigns nothing on a read-only non-stored field | "Compute method failed to assign \<the record set\>.\<field\>" |
| A related field's type disagrees with its target's | "Type of related field \<transport name\>.\<field\> is inconsistent with \<transport name\>.\<field\>" |
| A related field's path names an unknown field | "Field \<name\> referenced in related field definition \<transport name\>.\<field\> does not exist." |
| A filter rule cannot serve the operator | "Unsupported operator on \<field label\> '\<entity description\>' (\<transport name\>) in \<condition\>" |
| A permission failure while traversing a path or an embedded field | The original message, then a blank line, then "Implicitly accessed through '\<entity description\>' (\<transport name\>)." |
| Too many recomputation rounds | "Too many iterations for recomputing fields!", logged rather than raised |
| A co-computed group whose members disagree about elevated computation | "\<transport name\>: inconsistent 'compute_sudo' for computed fields \<list\>. Either set 'compute_sudo' to the same value on all those fields, or use distinct compute methods for sudoed and non-sudoed fields." |
| A co-computed group whose members disagree about storage | "\<transport name\>: inconsistent 'store' for computed fields, accessing \<list\> may recompute and update \<list\>. Use distinct compute methods for stored and non-stored fields." |
| A co-computed group whose members disagree about precomputation | "\<transport name\>: inconsistent 'precompute' for computed fields \<list\>. Either set all fields as precompute=True (if possible), or use distinct compute methods for precomputed and non-precomputed fields." |
| Precomputation declared on a field that is not derived | The attribute is forced to false and a warning is logged |
| Precomputation declared on a field that is not stored | The attribute is forced to false and a warning is logged |
| A company-dependent field declared required, translatable, or of an unsupported type | A warning is logged when the registry is built |

---

## 9. The recomputation algorithm

This is the heart of the system's correctness. It is specified here as an algorithm a rebuild can follow exactly.

### 9.1 Trigger trees

For each field, the registry derives a **trigger tree**. Its root is the set of fields that depend on this field *directly on the same entity*. Each edge is labelled by a relational field, and the sub-tree at that edge is the set of fields that depend on this field *through* that relation.

Construction: for every field *G* in the registry and every dependency path *G* declares, walk the path backwards from its last segment. The last segment names the field *F* that *G* depends on. Each earlier segment is a relation to be traversed backwards. Insert *G* into *F*'s tree at the node reached by following those segments.

Example. Suppose:

- *G* declares a dependency on `amount` of the same entity;
- *H* declares a dependency on `order_id.amount`;
- *I* declares a dependency on `invoice_id.order_id.amount`;
- *J* declares a dependency on `partner_id.amount`.

The trigger tree of `amount` is:

```
root: { G }
 ├─ edge order_id  → { H }
 │    └─ edge invoice_id → { I }
 └─ edge partner_id → { J }
```

Reading it: when `amount` changes on records *R*, recompute *G* on *R*; recompute *H* on the records whose `order_id` leads to *R*; recompute *I* on the records whose `invoice_id` leads to those; recompute *J* on the records whose `partner_id` leads to *R*.

Trigger trees are cached on the registry and discarded whenever the registry is set up or a package is loaded.

### 9.2 Notification of a change

**Preconditions.** A set of records *R* of entity *E*, a set of field names *N* whose values are changing, a flag saying whether this is a creation, and a flag saying whether the notification happens *before* or *after* the change.

**Postcondition.** Every stored computed field that depends on those fields, transitively, has the affected records in the pending-computation map; every non-stored computed field that depends on them has its cached values for the affected records removed.

**Algorithm.**

1. If *R* is empty or *N* is empty, stop.
2. Choose the working maps:
   - **After** the change: the map to add to is the transaction's pending-computation map, and nothing is treated as already marked. Fields are marked as soon as they are discovered.
   - **Before** the change: the map to add to is a scratch map, and the transaction's pending-computation map is consulted as "already marked". The scratch map is merged in at the end. This is used when the change destroys the relation needed to find the affected records — a deletion, or the removal of a link — so the traversal must run first and the marking must not disturb the traversal.
3. Build the merged trigger tree of the fields named by *N*. While merging, discard any sub-tree whose fields are **all** non-stored, non-computed, and absent from the cache: nothing would be invalidated, so the traversal would be pure cost. This pruning is what keeps the algorithm affordable on a registry with many thousands of computed fields.
4. If the merged tree is empty, stop.
5. Traverse the tree, elevated and with the archive filter switched off — an archived record's computed field must still be maintained — yielding a stream of triples (field, records, created):
   1. For each field at the current node, yield (field, current records, created).
   2. For each edge of the current node labelled by relational field *L* with sub-tree *T*:
      - If this is a **creation** and *L* is a many-to-one or a reference-by-identifier, skip the edge: nothing can already point at a record that is being created.
      - Compute the records reached backwards along *L*. Use an inverse of *L* that carries no filter of its own, when one exists; a filtered inverse would miss records. For a reference-by-identifier, collect the identifiers whose paired entity field names the right entity. When no usable inverse exists, run a search for the records whose *L* is among the current records.
      - Recurse into *T* with those records.
6. For each yielded triple:
   1. Remove the records that are **protected** for that field. If none remain, skip.
   2. If the field is declared **recursive**:
      - When the field is stored and computed, remove the records already marked (in either map) — this is what terminates the recursion.
      - When it is not stored, keep only the records that actually have a cached value, in any context — there is nothing to invalidate otherwise.
      - If any remain, recursively notify: the change to this field must itself be propagated. Append that traversal to the work list.
   3. If the field is stored and computed, add the records to the pending-computation map.
   4. Otherwise remove the field's cached values for those records, across every context partition.
7. If the notification was made *before* the change, merge the scratch map into the transaction's pending-computation map now.

### 9.3 Running a pending computation

**Preconditions.** A field *F* that is stored and computed, and a record set that overlaps the pending-computation entries for *F*.

**Algorithm.**

1. Read the pending identifiers for *F*. If there are none, stop.
2. If *F* is **recursive**, compute record by record, in the order of the record set, skipping records no longer pending. Computing one record may resolve others; the per-record loop lets the recursion unwind correctly.
3. Otherwise, for each record of the set that is still pending:
   1. Expand the batch: take that record's identifier followed by the other pending identifiers, up to the prefetch maximum. This batches the computation without loading unrelated records.
   2. Invoke the computation for the batch.
   3. If the invocation raises a refusal because the acting user lacks access, fall back to invoking it for the single record, so that one inaccessible record does not fail the whole batch.
   4. If the invocation raises because some records no longer exist, retry with only the existing ones, then mark the field computed on the missing ones so they do not remain pending for ever.
4. Invoking the computation:
   1. If the field is computed with elevated privileges, elevate.
   2. Take the group of fields computed by the same rule.
   3. **Remove those fields from the pending map for those records before running.** This is essential: a computation that reads the old value of one of its own fields would otherwise trigger a fetch, the fetch would flush, and the flush would recursively compute the same field.
   4. Protect the group on those records for the duration, so that assignments made by the rule do not re-trigger it.
   5. Run the rule.
   6. If it raises, put the records back into the pending map for every stored field of the group, then re-raise.

### 9.4 Reading a computed field

1. If the value is in the cache for this record, this field and this cache key, return it.
2. If the field is stored and the record is pending, run the pending computation (which batches over the other pending records), then return the cached value.
3. If the field is stored and the record is not pending, fetch the column for the prefetch set and return the value.
4. If the field is not stored, invoke the computation for the record and a batch of its prefetch set, then return the cached value.
5. If the computation did not assign the field for this record, the value is the field's empty value — but a computation that fails to assign every record it was given is a defect, because the next read will invoke it again and again.

### 9.5 Worked example of recomputation across relations

**Given** an Order with lines; each line has a stored computed subtotal depending on quantity and unit price; the order has a stored computed untaxed total depending on `line_ids.subtotal`; the party has a non-stored computed total ordered depending on `order_ids.total`.

**When** a line's quantity changes from 2 to 5.

1. Assignment marks `quantity` dirty on the line and notifies a change of `quantity` on that line, after the fact.
2. The trigger tree of `quantity` has `subtotal` at the root. The line is added to the pending map for `subtotal`.
3. Because `subtotal` is now going to change, the traversal continues from `subtotal`: its trigger tree has an edge labelled by the line's `order_id`, with the order's `total` at that node. Traversing backwards along `order_id` — using its inverse `line_ids` — yields the order. The order is added to the pending map for `total`.
4. From `total`, the trigger tree has an edge labelled `order_ids` — no: the party's field depends on `order_ids.total`, so the edge is labelled `order_ids` and the node holds the party's `total_ordered`. Traversing backwards along `order_ids` — using its inverse `partner_id` — yields the party. Because `total_ordered` is **not stored**, its cached value for that party is removed rather than marked pending.
5. At flush: `subtotal` is computed for the line; assigning it notifies again, but the order is already pending so nothing new is added. `total` is computed for the order; assigning it removes the party's cached `total_ordered` again, harmlessly. Two statements reach the database.
6. The next read of the party's total ordered recomputes it from the current orders.

**When instead the line is deleted.**

1. Before the deletion, a *before* notification is issued for every field of the line. The traversal runs while `order_id` still points at the order, so the order is collected. Because the notification is *before*, the collected fields are held in the scratch map.
2. The line is deleted.
3. The scratch map is merged: the order is pending for `total`.
4. At flush the order's total is recomputed from the remaining lines.

Had the notification been issued after the deletion, the traversal backwards along `order_id` would have found nothing and the order's total would have been left stale. **The before/after distinction is therefore part of the specification, not an optimisation.**

### 9.6 Bounds and failure

- The flush loop runs the recomputation to a fixed point, bounded at one hundred iterations; exceeding the bound reports an excessive-iteration condition, which indicates a dependency cycle in the entity definitions.
- A computation that raises a business refusal propagates it: the whole operation fails. A computation is not a place to recover from a business error.
- A computation that raises because a record vanished is retried without that record.

### 9.7 Three worked examples of derivation

**A precomputed field on a line.** An Order Line declares a sequence number as a stored integer, computed by a rule that reads the order's lines and assigns the line's position, declared precomputed.

- Given an Order with no lines, when the order is created with three line-creation commands, then the three lines are inserted in one statement and each row already carries its sequence number, because the value was produced before insertion.
- Given the same entity, when a line is created with an explicit sequence number, then no precomputation occurs and the supplied value is inserted.
- Given a declared default of ten on the sequence number, when a line is created without a value, then the default produces ten and the precomputation is skipped, because a declared default disables precomputation.
- Given the sequence number is additionally declared read-only, when a line is created with an explicit value, then the value is discarded before defaults are resolved and the precomputation runs.
- Given the sequence number depends on the order's total, which is a stored computed field that is **not** precomputed, then when the registry is built the precomputation is cancelled with the corresponding message of [section 8.9](#89-failure-catalogue-of-derivation) and the field is computed at flush time instead.

**A recursive field.** A Category has a parent as a many-to-one to itself, a materialised ancestor path, and a complete name that is stored, computed, declared recursive, and depends on the name and on the parent's complete name.

- Given categories A, A then B, and A then B then C, when A's name is changed to Z, then the traversal marks the complete name on A; draining it computes "Z"; assigning it declares a modification whose trigger tree reaches the complete name on B through the inverse of the parent link; the recursion guard allows B because it was not marked yet; the same happens for C. The three values become "Z", "Z / B" and "Z / B / C".
- Given the same entities, when the complete name is **not** declared recursive, then the self-dependency is detected when the registry is built, the field is declared recursive automatically, a warning is logged, and the behaviour is identical afterwards.
- Given a cycle from A to B and back created concurrently, then the recursion guard stops the traversal when it revisits a record that is already marked, and the materialised-path guard refuses the write that would create the cycle with "Recursion Detected."

**A co-computed group.** An Order declares an untaxed amount, a tax amount and a total amount, all stored, all computed by one rule, all depending on the lines' subtotal and tax.

- Given a write that assigns the total explicitly, the field being editable, then all three fields are protected for the duration of the write, therefore none of them is recomputed and the manual total survives.
- Given a write on a line's subtotal, then the traversal marks all three fields on the order; draining any one of them removes all three from the schedule, runs the rule once, and assigns all three.
- Given one of the three declared not stored while the other two are stored, then the registry build logs the storage-inconsistency message of [section 8.9](#89-failure-catalogue-of-derivation), because reading the non-stored one would silently write the stored ones.

---

## 10. Related fields

### 10.1 Definition

A **related field** declares a dotted path and becomes a mirror of the field at the end of it. It is the most common form of computed field and has its own derivation rules.

```
order_line.partner_id  →  related path "order_id.partner_id"
```

### 10.2 What a related field inherits

At registry build time, the path is resolved segment by segment; each intermediate segment must be relational, and the final field is the **target**. The related field then takes from the target:

| Attribute | Taken from the target unless declared on the related field |
|---|---|
| Type | Always. A declared type inconsistent with the target's is rejected with **"Type of related field "** the field **" is inconsistent with "** the target. |
| Label | Yes |
| Help text | Yes |
| Group restriction | Yes |
| Aggregate rule | Yes |
| Selection values (for a selection field) | Yes |
| Digits and minimum display digits (for a decimal number) | Yes |
| Currency field (for a monetary amount) | Yes, and for an embedded-parent field it is resolved on the parent |
| Target entity, client-side filter and context (for a relational field) | Yes |
| Translatable (for a text field) | Yes |
| Size and trim (for short text) | Yes |
| Sanitisation settings (for markup) | Yes |
| Any attribute the target declares that this entity accepts | Yes |

### 10.3 Derived behaviour

1. **Computation.** The related field's computation traverses the path. It is a *related* computation, whose dependency is the path itself.
2. **Traversal is field by field, not record by record.** For every record, take the first segment; then for every result, take the second; and so on. Doing it in this order lets each step prefetch across the whole batch. Doing it record by record would issue one query per record per segment. A rebuild must traverse in this order or accept a very large query count.
3. **Multi-valued intermediates.** When an intermediate segment yields several records, the **first** is taken. A related field therefore never yields more than one path.
4. **Stored** becomes false unless declared.
5. **Computed with elevated privileges** becomes true unless declared. Reading a related field therefore does not require access to the intermediate records — but see rule 7.
6. **Copy** becomes false and **read-only** becomes true, both unless declared.
7. **Access refusal is re-phrased.** If traversal raises a refusal, the message is extended with a second paragraph reading **"Implicitly accessed through '"** the entity's description **"' ("** the transport name **")."**, so that a user is told which document caused the refusal rather than seeing a bare refusal about an entity they never named.
8. **Inverse.** A related field gains an inverse rule — making it writable — when it is an embedded-parent field, or when neither it nor its target is read-only. The inverse writes the target field on the record at the end of the path, but only when the owning record and the target are **both** persisted or **both** in memory; writing a persisted target from an in-memory record would persist a change the user has not saved.
9. **Filter rule.** A non-stored related field gains a filter rule when every field along the path is itself filterable. The rule rewrites a condition on the related field into nested traversal conditions.
10. **Default.** A read-only related field with no inverse should not declare a default, because the default can never take effect; declaring one records a warning.
11. **Setup dependency.** The related field records that its setup depends on the target's, so that changing the target's declaration in a later package rebuilds the related field too.

### 10.4 Storing a related field

Declaring a related field stored gives it a column and makes it maintained by recomputation like any stored computed field. It is done when the field must be ordered by, grouped by, or filtered on efficiently — typically the company or the party copied onto every line of a document.

Two costs a rebuild must accept:

1. **Duplication.** The value now exists in two places and is kept in step by the recomputation machinery. A rebuild that omits the recomputation will silently produce stale copies.
2. **Translated stored related fields are not reliable in every language**, because the column holds one language's value per language slot and the recomputation only refreshes the language it ran in. Declaring one records a warning naming the field.

### 10.5 Worked example of a related field and a broken path

An Invoice Line declares a customer country following the path from the invoice, to the contact, to the country, where the invoice link is a required many-to-one, the contact link is an optional many-to-one, and the country link is an optional many-to-one.

**Derived attributes.** Not stored; computed with elevated rights; not copied; read-only; one dependency path, the declared path itself; target entity Country; label and help text copied from the contact's country field.

**Reading on three lines** — line one whose invoice has a contact with a country, line two whose invoice has a contact with no country, line three whose invoice has no contact:

1. The invoice link is read for the whole set in one statement, giving three invoices.
2. The contact link is read for the three invoices in one statement, giving two contacts and one empty value.
3. The country link is read for the two contacts in one statement. Line one receives the country, lines two and three receive the empty value.

Three statements for three lines, not nine, because the traversal is performed one field at a time for the whole set.

**Filtering.** A condition that the customer country equals a country name is rewritten by the generated filter rule. The value is a non-empty text and the operator is positive, therefore no tolerance for an unset link is added, and the condition becomes a chain of privileged existence conditions ending in a display-name membership on Country.

A condition that the customer country is the empty value has a value denoting emptiness under a positive operator, therefore every non-required link in the chain also accepts an unset value: the condition becomes an existence condition on the invoice whose sub-filter is the disjunction of "the contact satisfies the disjunction of the country condition and the country being unset" with "the contact is unset". It matches lines two and three but not line one. The invoice step adds no alternative, because the invoice link is required.

A condition that the customer country differs from a country name is declined by the generated filter rule, because the operator is negative and the value is not empty; the platform therefore retries with the positive form and negates the whole result, which correctly returns lines two and three alongside every line whose country is not that country.

---

## 11. Company-dependent values

### 11.1 What it means

A **company-dependent** field holds a different value per company on the same record. The classic case is a party's receivable account: the same party, seen from company A, has account A; seen from company B, account B.

This is not a company *scope* — the record belongs to no company and is shared — it is a per-company *value*.

### 11.2 Which types may be company-dependent

Only these: boolean, integer, decimal number, short text, long text, markup, selection, date, date and time, many-to-one. Declaring any other type company-dependent produces a warning naming the field and the allowed types.

### 11.3 Storage

The column becomes a **structured document** holding a mapping from company identifier, as text, to the value in that field's underlying column type.

```
{ "1": 42, "3": 57 }
```

A company absent from the mapping has no explicit value and falls back (see [11.5](#115-the-fallback)).

### 11.4 Derived attributes

Declaring a field company-dependent sets four defaults:

1. **Copy** becomes false — duplicating a record does not carry per-company values.
2. **Index kind** becomes standard-excluding-empty-values, because most rows have an empty mapping.
3. **Prefetch group** becomes the dedicated company-dependent group, so that these fields are fetched together rather than with ordinary columns.
4. **Context dependency** becomes the company key, which partitions the cache by current company.

And two warnings:

- Declaring a company-dependent field **required** warns, because the not-null constraint would be on the mapping, not on any company's value.
- Declaring it **translatable** warns, because the column cannot be both a per-company mapping and a per-language mapping.

### 11.5 The fallback

A company with no entry in the mapping does not read as empty. It reads the **fallback**: the value recorded in the per-entity default catalogue for this entity, this field and this company, read unrestricted and in that company.

```formula
effective_value( record , field , company )
    = mapping_of( record , field )[ company ]                      if the company has an entry
    = default_catalogue_value( entity , field , company )          otherwise
```

Consequences a rebuild must reproduce:

1. Changing the fallback changes the effective value for every record that has no explicit entry — that is how a company-wide default account is changed in one place.
2. Writing a value **equal to the fallback** on insertion writes nothing at all: the mapping stays empty so that the record keeps following the fallback. Writing it through an update does store it.
3. Filtering compares the effective value: the compiled condition takes the mapping's entry for the current company and, when absent, the fallback expressed as a literal.
4. Because the fallback appears in the compiled condition, a company-dependent field with the standard-excluding-empty-values index gains an additional emptiness test in front of the condition whenever the fallback does not satisfy it, so that the partial index can still be used.

### 11.6 Company-dependent many-to-one and referential integrity

A company-dependent many-to-one has **no foreign key**, because the identifiers live inside a structured document. The registry therefore keeps an index of which entities have company-dependent many-to-one fields, and deleting a record of the target entity must clear those references explicitly. A rebuild that stores them as plain columns will get foreign keys and different deletion behaviour.

### 11.7 Candidate filter, sorting, grouping and invalidation

- **Filtering** compares the effective value, which means the coalesced expression of [section 11.5](#115-the-fallback): a record with no stored entry matches when its fallback satisfies the condition. Sorting and grouping use the same coalesced expression.
- **The index-friendly rewrite.** When the field carries the standard-excluding-empty-values index and the fallback does **not** satisfy the condition, the condition is prefixed with a test that the stored mapping is not empty, which lets the partial index be used, because a record with no entry cannot match anyway. The rewrite is skipped when the condition targets a date granularity rather than the whole value. Whether the fallback satisfies the condition is decided by building a throwaway in-memory record carrying the fallback and evaluating the condition against it in memory; when that evaluation is impossible the rewrite is skipped.
- **The candidate filter sent to a client.** When a company-dependent relational field is also marked for company consistency, the candidate filter is built against the **first activated company** of the environment rather than the record's own company, conjoined with the field's own declared filter ([multi-company, section 4.3](multi-company.md#43-company-checked-relations)).
- **Invalidation.** Creating, writing or deleting any recorded default value invalidates the **entire** record cache and clears the registry caches, because any company-dependent field's fallback may have changed.
- **Deleting a record referenced by a company-dependent link.** Because no foreign key exists, deletion is handled explicitly ([section 11.6](#116-company-dependent-many-to-one-and-referential-integrity)). Its two refusals are **"Unable to delete "** the record's display name **" because it is used as the default value of "** the field, and **"You cannot delete "** the record being deleted **", as it is used by "** the referencing record.

### 11.8 Worked example of a company-dependent value with a fallback

A Product declares a cost as a company-dependent monetary field. Companies 1 and 2 exist, and the company currency has two decimal places.

1. Given no recorded default for the product's cost, when a product is created while company 1 is current with a cost of 12.345, then the value is rounded to 12.35, compared with the fallback of 0.00, found different, and the mapping holding the single entry for company 1 with the value 12.35 is inserted.
2. Given that product, when it is read while company 2 is current, then the mapping has no entry for company 2 and the fallback 0.00 is returned.
3. Given that product, when a recorded default of 12.35 is created for company 1, then the whole record cache is invalidated; reading the cost while company 1 is current still returns 12.35, now from the stored entry; writing 12.35 again while company 1 is current rebuilds the mapping, finds the entry equal to the fallback and removes it, leaving an empty mapping.
4. Given the empty mapping and a fallback of 12.35, when the fallback is changed to 13.00, then reading the cost while company 1 is current returns 13.00, because the record follows the fallback.
5. Given the cost carries the standard-excluding-empty-values index, when records whose cost exceeds 100.00 are searched while company 1 is current, then the fallback of 13.00 does not satisfy the condition, therefore the condition is compiled with the mapping-not-empty prefix and the partial index is used.
6. Given the same, when records whose cost is below 100.00 are searched, then the fallback does satisfy the condition, therefore the prefix is **not** added and records with no entry are correctly returned.

---

## 12. Translatable values

### 12.1 What it means

A field marked **translatable** holds one value per language on the same record. Text, long text and markup fields may be translatable; no other type may.

### 12.2 The two translation modes

| Mode | What is stored | Used for |
|---|---|---|
| Whole value | One complete value per language | Names, labels, short descriptions |
| Term by term | One complete value per language, but the value is *derived* from the source by substituting translated terms | Rich content where the markup must stay identical across languages and only the text nodes differ |

In the term-by-term mode the field declares an extraction rule that walks the value and yields the translatable terms; the same rule reassembles a value by substituting each term's translation.

### 12.3 Storage

The column becomes a **structured document** mapping language code to value:

```
{ "en_US": "Invoice", "fr_FR": "Facture" }
```

The source-language entry is always present and is the value written when no language is active.

### 12.4 Reading and writing

**Reading.** Take the entry for the environment's language. If absent, take the source-language entry. If the environment has no language, take the source-language entry.

**Writing.** Writing with a language active updates that language's entry and leaves the others alone. Writing with no language active updates the source-language entry **and** every entry whose current value equals the old source value — that is, translations that had never diverged follow the source, while translations that were deliberately changed do not.

**Inserting a new record** writes the value into the source-language entry and into the environment's language entry.

### 12.5 Term-by-term specifics

1. The source value's terms are extracted and stored as the source terms.
2. A translation is stored as a mapping from source term to translated term, and the value for a language is produced by substituting.
3. When the source value changes, terms that are still present keep their translations; terms that disappeared lose theirs; new terms are untranslated and fall back to the source term.
4. The translation-authoring context keys make reading return the authoring representation — the value with each term wrapped so that a user interface can offer it for editing — rather than the rendered one. The environment's technical language is prefixed with an underscore in that case.

### 12.6 Restrictions

1. A translatable field **cannot depend on context**, because its column already partitions by language. A stored translatable field declaring context dependencies records a warning and the dependencies are dropped.
2. A **stored related** translatable field is not correctly maintained in every language and records a warning.
3. A translatable field cannot also be company-dependent.
4. When a package update makes a field no longer translatable, the column is converted back: the source-language entry becomes the whole value and the other languages are lost. The build detects this by comparing the catalogue's record of which fields are translated with the registry's, and re-synchronises the affected entities.
5. Conversely, a field that the catalogue records as translated is **forced** translatable during the registry build even if no package declares it so, to prevent a column conversion that would lose data. The forcing happens by adding an extra declaration layer.

### 12.7 Reading and writing many languages at once

Two generic operations exist:

- **Get field translations** returns, for one field, the value in each requested language together with the source terms, so that a translation editor can show them side by side.
- **Update field translations** writes several languages at once, either as whole values or as term substitutions.

A third, exposed for the desktop client, **overrides** a translatable value in *every* language with one value, discarding divergent translations.

---

## 13. Defaults

### 13.1 The resolution order

Asking for the default values of a list of field names resolves each name independently, in this exact order. The **first** rule that produces a value wins; later rules are not consulted.

1. **The context.** If the context holds the key made of the default prefix and the field name, its value is the default.
2. **The recorded default, for a field that is not company-dependent.** If the per-entity default catalogue holds a value for this entity and field — set by an administrator, or pinned by the acting user as a personal default — that value is the default.
3. **The field's declared default.** A fixed value, or a rule evaluated against the empty record set in the current environment.
4. **The recorded default, for a company-dependent field.** The catalogue is consulted *after* the declared default for company-dependent fields, because for them the catalogue entry is the fallback of [11.5](#115-the-fallback) rather than a user's preference, and a field's own declared default should win over a fallback.
5. **The embedded parent.** If the field is exposed from an embedded parent record and the acting user may write it, the request is delegated to the parent entity, whose defaults are resolved by the same algorithm and merged in.

A field for which no rule produces a value simply has no default; it is absent from the result rather than present with an empty value.

### 13.2 Conversion of the result

Every produced default is converted through the cache representation and back to the write representation. This normalises, in particular, to-many values: a list of identifiers becomes a single Set command, which is what a client can apply, rather than a list of Link commands, which it cannot.

While converting a relational default in a **restricted** environment, the access implied by each command is checked: a Delete command checks deletion rights on the target, and an Update command — or, for a one-to-many, an Unlink or Link command — checks write rights. A default supplied through the context therefore cannot be used to perform an operation the caller could not perform directly.

### 13.3 Defaults during creation

Creating a record fills in the defaults of every field the caller did not supply, with two refinements:

1. **Embedded parents already supplied are skipped.** If the creation supplies the link to an embedded parent, the fields exposed from that parent are not defaulted, because the parent already has its own values. The skip is transitive: a parent of a parent that is supplied also blocks defaulting.
2. **Conversion to commands.** A defaulted many-to-many whose value is a list of identifiers becomes a Set command; a defaulted one-to-many whose value is a list of value mappings becomes a list of Create commands.

The supplied values always override the defaults, never the other way around.

### 13.4 Declared defaults

A declared default is either a fixed value or a rule. A fixed value is wrapped into a rule returning it, so that everything downstream sees one form. A rule receives the empty record set in the current environment and can therefore read the acting user, the company, the language and the context.

Common declared defaults and what they mean:

| Declared default | Value |
|---|---|
| A literal | Always that value. |
| The current company | The environment's current company. |
| The acting user | The environment's acting user. |
| Today, or now | The current date in the environment's time zone, or the current instant in Coordinated Universal Time. |
| The next number from a sequence | Deliberately **not** done as a default; numbering is assigned at confirmation or posting time, because a default would consume numbers for records the user abandons. |

Declaring the default explicitly as absent removes an inherited default: an extension can therefore take a default away.

### 13.5 Recorded default values

A **recorded default value** stores a default for one field, optionally narrowed by user, by company and by a condition. It is the second and the fourth step of the resolution order of [section 13.1](#131-the-resolution-order), and it is also the fallback source of a company-dependent field ([section 11.5](#115-the-fallback)).

| Field | Meaning |
|---|---|
| Field | The field definition the default applies to. Required. Deleting the field definition deletes the default |
| User | When set, the default applies only to that user |
| Company | When set, the default applies only while that company is current |
| Condition | When set, the default applies only when the client asks for defaults with this condition. The conventional shape is a field name, an equals sign and the serialised value |
| Serialised value | The value, serialised as a structured document. Required |

**Resolution** for one entity and one condition returns a map from field name to value:

1. Select every recorded default of that entity whose user is empty or is the acting user, whose company is empty or is the current company, and whose condition matches — the empty condition when none is asked for.
2. Order by user, then company, then identifier.
3. Keep the **first** value seen for each field.

Because an empty column sorts before a filled one in that ordering, a default that names neither a user nor a company is kept in preference to a narrower one only when it has the lower identifier. The intended reading is that the ordering makes the result deterministic; a caller that needs a user-specific value must not also define a general one for the same field. [Multi-company, section 7.2](multi-company.md#72-user-default-values) states the company precedence that follows from the same ordering.

**Validation on write.** The serialised value must parse and must convert to the field's type. A malformed document is refused with **"Invalid structured-data format in Default Value field."** A type mismatch is refused with **"Invalid value in Default Value field. Expected type '"** the field type **"' for '"** the qualified field name **"'."** An integer default outside the range of a thirty-two-bit whole number is refused with **"Invalid value for "** the qualified field name **": "** the value **" is out of bounds (integers should be between -2,147,483,648 and 2,147,483,647)"**.

**Permission.** A user may only define a default for a field they may write; the check is performed after the record's own permission check.

**Invalidation.** Creating, writing or deleting a recorded default value invalidates the whole record cache and clears the registry caches, because any company-dependent fallback may have changed.

### 13.6 Declared defaults and copying

- A declared default of "none" explicitly discards any default inherited from a previous declaration of the same field.
- A field named as the state field defaults to not being copied, which means duplicating a record resets the state to its default rather than carrying the current state over.
- A derived field that is read-only and has no inverse rule must not declare a default; declaring one is reported in the log as redundant, because the default can never be observed.

---

## 14. On-change behaviour

### 14.1 What it is

**On-change** is the mechanism that makes a form react while the user types, before anything is saved. It is a server round trip: the client sends the current state of the form and the names of the fields the user just changed; the server computes what else should change and returns the differences.

It exists because the same derivation logic must run whether a record is edited in a form, created over the transport, or imported from a file — and only the form case can afford a round trip per keystroke.

### 14.2 The contract

**Input**

| Parameter | Meaning |
|---|---|
| Values | The complete current state of the form, field name to value, in the write representation. For to-many fields, the list of commands the client would apply. |
| Changed field names | The names the user just changed. **An empty list means "this is a new record"**, which triggers the first-call behaviour of [14.4](#144-the-first-call). |
| Field specification | Which fields the form shows, and for each relational field which sub-fields its inline list shows. Determines what may be returned. |

**Output**

| Key | Meaning |
|---|---|
| Value | The fields whose value changed, in the read representation; for to-many fields, a list of commands the client should apply to its current value. |
| Warning | At most one warning, with a title, a message and a display kind. |

### 14.3 The algorithm

**Preconditions.** The receiver is either empty (creating) or a single persisted record (editing).

1. Flush the unit of work, so that the in-memory state is consistent.
2. Check access: write access on the record when editing, creation access when creating. (The user entity is exempt, because a user may edit a defined subset of their own fields without holding write access on themselves.)
3. If any changed field name is not a field of the entity, return nothing.
4. If this is a first call, run [14.4](#144-the-first-call).
5. **Prefetch the lines.** For every to-many field in the specification whose sub-fields are requested, collect the identifiers appearing in the current value and in the commands, fetch the requested sub-fields on the persisted lines, and copy those values into the cache of the corresponding in-memory line records. This avoids recomputing stored computed fields on in-memory copies, which would be both slow and wrong.
6. **Separate initial values from changed values.** The values of the changed fields are set aside; everything else is the initial state. A link to an embedded parent that is empty is removed from the initial state rather than forced empty.
7. **Build the in-memory record.** When editing, create an in-memory record whose cache is filled from the persisted record for every field in the specification, with the origin set to the persisted record, then overlay the initial values. When creating, create an in-memory record from the initial values, with the changed fields explicitly set empty so that the first snapshot does not trigger defaults for them.
8. **Align embedded parents.** For every field exposed from an embedded parent that appears in the initial values, write the value onto the in-memory parent record too, so that computations on the parent see the form's values.
9. **Snapshot.** Take a snapshot of the in-memory record over the specification: for each scalar field its value, for each to-many field the snapshot of each line over its sub-specification.
10. **Apply the changed values** to the in-memory record's cache, which also notifies dependent fields so that stored computed fields not shown on the form are still recomputed.
11. Refresh the snapshot for the changed fields.
12. **Determine the work list.** On a first call it is every field of the specification; otherwise it is the changed field names.
13. **Notify.** With the changed fields — and the whole group of fields computed by the same rule as each of them — protected on the in-memory record, notify a change of the work list (or of every field on a first call). For a changed field exposed from an embedded parent, also write it onto the parent explicitly, because it was never assigned there.
14. **Apply the reaction rules.** Loop:
    1. For each field in the work list, in order, run every reaction rule declared for that field that has not already run in this round, collecting warnings. Mark the field done.
    2. If the context switches off cascading reactions, stop.
    3. Recompute the work list: every field of the specification that is not yet done **and whose snapshot value has changed** since step 9. If it is empty, stop.
15. **Second snapshot.** Take a fresh snapshot of the in-memory record over the specification.
16. **Difference.** The returned values are the fields whose snapshot differs, in the read representation. On a first call the difference is forced, so that every field of the specification is returned even if unchanged.
17. **Warnings.** With one warning, return it with its title, message and kind, defaulting the kind to a dialog. With several, return a single warning titled with the word for warnings whose message is the warnings' titles and messages joined by blank lines, shown as a dialog.

### 14.4 The first call

When the changed-field list is empty:

1. The changed-field list becomes every field present in the supplied values except the identifier.
2. For every field of the specification **absent** from the supplied values, resolve its default. If a default exists, use it and add the field to the changed list. If not, set the field explicitly empty — **unless** the field is computed and has declared dependencies, in which case it is left alone so that its computation runs.
3. The work list is every field of the specification, so every reaction rule runs once.

This is what fills a brand-new form: defaults first, then computations, then reactions, all in one round trip.

### 14.5 Reaction rules

A **reaction rule** is declared against one or more field names and runs when any of them changes. It may:

- assign other fields on the in-memory record;
- return a warning;
- narrow the candidate values of a relational field by returning a filter for it.

Rules:

1. Reaction rules run **only** through this mechanism. They never run on a write from the transport, from an import, or from code. A rebuild must not "helpfully" invoke them elsewhere; doing so changes behaviour, because the same derivation is expected to be expressed as a computed field when it must hold universally.
2. A reaction rule runs against an **in-memory record**, never a persisted one. Nothing it does is saved by itself.
3. Several rules may be declared for the same field, by several packages; all run, in package order.
4. A rule declared for a field that is not in the specification never runs, because the client never reports that field as changed.

### 14.6 The limits

These limits are the most common source of misunderstanding and must be reproduced exactly:

1. **On-change is not validation.** A rule that refuses a value by raising is not a constraint; the same value written over the transport will be accepted. Validation belongs in [section 15](#15-validation).
2. **On-change is not a computation.** A derivation expressed only as a reaction rule does not happen on import, on transport writes, or on records created by other capabilities. If the derivation must always hold, it is a computed field.
3. **On-change sees only the specification.** A rule that reads a field the form does not show reads whatever the in-memory record has, which for a new record is the default and for an edited record is the persisted value — not necessarily what the user would expect.
4. **On-change results are advisory.** The client may discard them; nothing is persisted until the user saves.
5. **Cascading is bounded by the specification.** A change that would propagate to a field outside the specification does not come back to the client, though the in-memory computation still happened.
6. A batch form — several records edited at once — runs the whole algorithm once per record.

---

## 15. Validation

Five distinct mechanisms enforce correctness. They differ in when they run, what they can see, and what they cost.

### 15.1 Field-level conversion checks

Run on **assignment**, before anything reaches the cache.

| Check | Failure |
|---|---|
| A selection value is in the declared set | **"Wrong value for "** entity and field **": "** the value |
| A monetary amount has exactly one currency | **"Got multiple currencies while assigning values of monetary field "** field |
| A relational value is a record set of the right entity, an identifier, or a command list | A type failure naming the field |
| A date or instant can be parsed | A conversion failure naming the value |
| A text value fits the declared maximum length | Silently truncated, not refused |
| A decimal number is rounded to the declared digits | Silently rounded, not refused |

### 15.2 Declared validations

A **declared validation** is a rule declared against a list of field names. It receives the records and refuses by raising a validation failure with a message.

**When it runs.** After a creation or a write, once the values are in the cache, for every rule at least one of whose declared field names was touched and none of whose declared field names is in the excluded set. The excluded set is used when a write is part of a larger operation that will set the remaining fields in a moment.

**How it runs.** Elevated, exactly like a stored computed field's computation, so that a validation can read records the acting user cannot.

**What it must obey.**

1. It must declare **every** field it reads. A validation that reads an undeclared field will not run when that field changes, and the invariant will be violated silently.
2. It must be written against a record set and check every record.
3. Its message is shown to the user verbatim and must therefore be a complete sentence in the user's language.

### 15.3 Database constraints

A **database constraint** is declared on the entity and applied to the table during schema synchronisation. Three forms exist:

| Form | Declared as | Example purpose |
|---|---|---|
| Check | A condition over the row's columns | An amount must be positive; a start date must not be after an end date |
| Unique | A column list | A code must be unique |
| Foreign key | A column, a target table and a column | A reference the field system does not itself create |

Additionally an entity may declare **unique indexes** and **plain indexes** over expressions, which behave like constraints for the purpose of naming and error reporting.

Rules:

1. The constraint's database name is the table name, an underscore, and the declared local name. The local name must begin with an underscore in the declaration, which keeps constraints out of the field namespace.
2. Every constraint declares a **message** shown when it is violated. It may be a fixed string or a rule producing one from the environment and the violation's details, which is how a uniqueness message can name the conflicting value.
3. Constraint application is **deferred** to the end of the build, because a constraint may fail while other packages are still loading their data. Each is applied once, keyed so that two packages declaring the same constraint do not apply it twice.
4. Applying a constraint that already exists with a different definition drops and recreates it.
5. Applying a constraint that existing rows violate fails; the failure is recorded as a warning naming the table and the constraint, the constraint is not created, and the build continues.
6. Constraints are recorded as records of the constraint catalogue, owned by the declaring package, so that removing the package drops them.

**Violation at run time.** A violation surfaces from the database at flush or at commit, not at assignment. The retry loop catches it, identifies the owning entity by matching the reported table name, translates it through that entity's constraint message catalogue, and raises a validation failure whose text is **"The operation cannot be completed: "** followed by the message. Constraint violations are never retried.

**Generic failure messages.** The declared message of [rule 2](#153-database-constraints) is used when the failure names a constraint the entity declared and a message is recorded for it, either on the constraint catalogue record or by the constraint itself. When no declared message applies — an unnamed constraint, a constraint on another entity's table, a not-null column, a foreign key created by the field system — the platform composes a generic message from the database's own diagnosis. Two placeholders recur, and both are computed before the message is chosen:

- **The entity display** is the human-readable description of the entity followed by its transport name in brackets, in the form `'<description>' (<transport name>)`. It is the literal word **"Unknown"** when the table the database reported is not this entity's table.
- **The field display** is `'<label>' (<field name>)` when exactly one column is implicated and that column is a field of this entity; `'<column list>'` when several columns are implicated, the columns separated as a list in the reader's language; and the literal word **"Unknown"** when no column could be determined at all.

| Failure reported by the database | Message |
|---|---|
| A not-null column received an empty value | **"Missing required value for the field "** the field display **"."** on the first line, **"Model: "** the entity display on the second, **"- create/update: a mandatory field is not set"** on the third, and **"- delete: another model requires the record being deleted, you can archive it instead"** on the fourth |
| A foreign key still points at the row | **"Another model is using the record you are trying to delete."** then a blank line, then **"The troublemaker is: "** the entity display, then **"Thanks to the following constraint: "** the field display, then **"How about archiving the record instead?"**. When more than one column is implicated, the field display is replaced by the name of the constraint the database reported |
| A unique constraint was violated | **"The value for "** the field display **" already exists."** then a blank line, then **"Detail: "** followed by the conflicting key and value exactly as the database reported them. Here the field display is composed differently: `'<column list>' (<label list>)`, the raw column names first and their labels after |
| A check constraint was violated | No specific text can be composed, because a check constraint says nothing about which value is at fault; the database's own text is reported unchanged |

Every one of these is wrapped by the caller in **"The operation cannot be completed: "** followed by the composed message, so that the outer form is the same whether the message was declared or generic.

The two words **"Unknown"** and the four fixed lines of the not-null message are part of observable behaviour: support procedures key on the phrase "a mandatory field is not set" to tell a missing value from a blocked deletion, since the same database failure produces both.

### 15.4 Deletion guards

A **deletion guard** is a rule declared to run before records are deleted. It may refuse.

| Variant | When it runs |
|---|---|
| Always | Before any deletion of records of the entity |
| Except during package removal | Before ordinary deletions, but skipped while a package is being removed, so that removing a package is not blocked by a guard meant for users |

Guards run before anything is deleted, on the whole set at once.

### 15.5 Company consistency

Relational fields marked for company consistency are checked so that a record never links to a record of an incompatible company. Unlike the four data gates, the check is a **validation**: it runs whatever the acting identity and also with elevated rights, because a cross-company link corrupts the books regardless of who created it.

What this document contributes:

| Contribution | Rule |
|---|---|
| The declaration | A relational field declares that it participates; an entity declares whether the check runs automatically on every creation and write, or only where a capability invokes it |
| Which fields are checked | Every field of the entity when no names are given, or when the company field or the company list field is among the names; otherwise only the named ones |
| The split | The participating fields are partitioned into ordinary ones, checked against the record's own companies, and company-dependent ones, checked against the environment's **current** company, because a per-company value belongs to the company it was written for |
| The reads | The linked records are read with elevated rights and with archived records visible, so that neither a permission refusal nor an archived target can turn a violation into a spurious pass or a wrong message |

The company filter of each target entity, the four entity families that override the compatibility rule, the full algorithm, the refusal message with its three line shapes and the worked examples are in [multi-company, section 6](multi-company.md#6-the-company-consistency-check).

```formula
compatible( linked record , owning companies ) =
    ( company of linked record is empty )
    OR ( company of linked record ∈ owning companies )
```

This is the default family's rule; an empty company on the linked record means "shared by all companies", which is why shared reference data can be linked from any company's documents.

### 15.6 Which mechanism to use

| Requirement | Mechanism |
|---|---|
| The value must be one of a fixed set | Selection type |
| The value must satisfy a condition expressible over one row's columns | Database check constraint — cheapest, always enforced, even by direct statements |
| The value must be unique | Database unique constraint |
| The rule spans several records or needs business context | Declared validation |
| The rule should guide the user while typing but not block a transport write | Reaction rule |
| Records must not be deleted in some state | Deletion guard |
| Linked records must belong to compatible companies | Company consistency |

---

## 16. Ordering

### 16.1 The default ordering

Every entity declares a default ordering, defaulting to the identifier ascending. It is applied by every search that does not name its own.

### 16.2 The ordering grammar

An ordering is a comma-separated list of terms. Each term is:

```
<field name> [ "." <property name> ] [ ":" <function> ] [ " " ( "asc" | "desc" ) ] [ " " ( "nulls first" | "nulls last" ) ]
```

| Part | Meaning |
|---|---|
| Field name | Lowercase letters, digits and underscores. Must name a field of the entity. |
| Property name | Only for a custom-properties field: which property to order by. |
| Function | A granularity for a date or instant — for example the year, the quarter, the month, the week or the day — used when ordering a grouped result. |
| Direction | Ascending by default. |
| Empty-value placement | Where empty values go. The default follows the database's convention for the direction: empty values last when ascending, first when descending. |

An ordering that does not match the grammar is refused with **"Invalid "order" specified ("** the ordering **"). A valid "order" specification is a comma-separated list of valid field names (optionally followed by asc/desc for the direction)"**.

### 16.3 What may be ordered by

| Field | Orderable |
|---|---|
| A stored field with a column | Yes |
| A stored computed field | Yes — it has a column |
| A non-stored field | **No** |
| A one-to-many or many-to-many | No |
| A many-to-one | Yes, and see [16.4](#164-ordering-by-a-many-to-one) |
| A custom property | Yes, by naming the property |

Ordering by a field that cannot be ordered by is refused.

### 16.4 Ordering by a many-to-one

Ordering by a many-to-one orders by the **display order of the target entity**, not by the raw identifier. The generated query joins the target table and applies the target's own default ordering to it. This is why a list ordered by party comes out alphabetically rather than in identifier order.

Consequences:

1. The join is a left join, so records with no value still appear.
2. The target's default ordering may itself order by a many-to-one, which joins again. The platform bounds the depth of this recursion to keep the query finite.
3. Ordering by a many-to-one to an entity whose default ordering is the identifier gives identifier order, which is rarely what a user expects; such entities usually declare an ordering by name.

### 16.5 Stability

Ordering is **not** guaranteed stable unless the ordering is total. An ordering by a non-unique field leaves ties in an unspecified order, which differs between runs and between database products. Every ordering that must be reproducible therefore ends with the identifier. A rebuild should treat an ordering without a unique final term as under-specified and append the identifier.

*(industry-standard default, stated explicitly because the tie order is not otherwise observable.)*

### 16.6 In-memory sorting

A record set already in memory can be sorted by a field name or by a computed key. Sorting by a field name reads the field on every record, which may trigger computation; sorting with no key uses the entity's default ordering, re-fetching from the database when the set is large enough to make that cheaper.

In-memory sorting treats empty values as sorting **before** non-empty ones by default, and reverses consistently: reversing a sort reverses the placement of empty values too.

---

## 17. Display names and name searching

### 17.1 The display name

Every entity has a non-stored computed field holding a textual representation of the record.

**Default rule.** If the entity has a display-name field, the display name is that field's value, converted to text by the field's own rule — which for a selection field yields the label, for a many-to-one the target's display name, and for a date the formatted date. If the entity has no display-name field, the display name is the transport name, a comma and the identifier.

**Overriding.** An entity may declare its own rule, which is how a journal entry shows its number and a contact shows the company name followed by the contact name. An overriding rule should declare its dependencies so that the display name is invalidated when they change, and may declare context dependencies so that, for example, showing a product's internal reference can be switched on by a context key.

### 17.2 Determining the display-name field

At registry build time:

1. If the entity declares one, it must exist; otherwise the build fails.
2. Otherwise, if a field named `name` exists, that is it.
3. Otherwise, if the entity is user-defined and a field named `x_name` exists, that is it.
4. Otherwise there is none, and the display name falls back to the transport name and identifier form.

### 17.3 Searching by display name

A condition on the display name is rewritten into a condition over the entity's **name-search fields**, which default to the display-name field alone.

**Algorithm.**

1. If there are no name-search fields, record a warning naming the entity and impose no restriction at all — every record matches.
2. If the operator is a pattern operator, the value is empty, and the operator is not the anchored form, short-circuit: a pattern match against nothing matches everything, so return "everything" for a positive operator and "nothing" for a negative one.
3. Choose the combining connective: disjunction for a positive operator, conjunction for a negative one. (A record matches "not like X" on its name *and* its reference; a record matches "like X" on its name *or* its reference.)
4. For each name-search field — which may itself be a dotted path — resolve the final field:
   - If it is **relational**, produce a condition on that path followed by the display name, which recurses into the target entity's name search.
   - If the operator is a pattern operator, produce the condition directly.
   - Otherwise convert the value to the field's type and produce the condition; a value that cannot be converted for a given field simply contributes nothing, so searching "42" across a name and a numeric reference matches the reference without failing on the name.
5. Combine with the chosen connective.

### 17.4 The name-search operation

The operation behind every relational selector takes a text fragment, an extra filter, an operator (containment, case-insensitive, by default) and a limit (one hundred by default). It searches on the display name with that operator, conjoined with the extra filter, fetches the display names, and returns pairs of identifier and display name — reading the display names unrestricted, so that a user who may see the record through the relation can see its name.

### 17.5 Creating from a name

An entity with a display-name field supports creating a record from a name alone: a record is created with that field set to the given name, plus the usual defaults, and the pair (identifier, display name) is returned. An entity with no display-name field cannot do this; the attempt records a warning naming the entity and returns nothing.

---

## 18. The archive flag

### 18.1 What it is

An entity may carry a boolean field named `active` (active) — or `x_active` on a user-defined entity — meaning "this record is in use". Setting it false **archives** the record: it stays in the database, keeps its identifier, keeps every reference to it, and disappears from ordinary searches.

Archiving exists because deletion is usually wrong: a product that was sold must remain readable from the invoices that sold it, and a user who has left must remain the author of their entries.

### 18.2 The implicit filter

Every search on an entity with an archive field adds the condition that the flag is true, **unless**:

- the context key that switches the filter off is present and false; or
- the filter itself already mentions the archive field, at the top level of the condition list.

The second exception is what lets a user write a filter for archived records without also having to set the context key.

Rules:

1. The implicit filter is added **before** the record rules, so a user sees only unarchived records they are allowed to see.
2. The filter applies to searches, not to reading by identifier: an archived record can still be read, and a relational field still yields it.
3. Traversal of a relation in a filter always switches the archive filter **off** on the target, because an archived target should not silently drop the records pointing at it. This is deliberate and observable.
4. Recomputation traversal likewise switches it off: an archived record's stored computed fields are still maintained.
5. Duplication, company consistency checks and record-rule evaluation all switch it off.

### 18.3 The operations

| Operation | Effect |
|---|---|
| Archive | Set the flag false on every record of the set. |
| Unarchive | Set the flag true on every record of the set. |
| Toggle | Flip the flag on each record individually, so a mixed set becomes inverted rather than uniform. |

Archiving a record commonly cascades in a capability-specific way — archiving a product archives its variants, archiving a company archives its users' access to it — but the platform itself cascades nothing.

### 18.4 Archive versus deletion

| | Archive | Delete |
|---|---|---|
| Record remains readable | Yes | No |
| References remain valid | Yes | Depend on the deletion behaviour |
| Appears in searches | No, unless asked for | No |
| Reversible | Yes | No |
| Frees the identifier | No | No — identifiers are never reused |
| Allowed on a record referenced with *restrict* | Yes | No |

The design principle and its trade-off are in [design principles](design-principles.md#5-archival-over-deletion).

---

## 19. Copying

Duplicating a record produces a new record whose values are derived from the original.

### 19.1 The per-field rule

| Field | Copied by default |
|---|---|
| Plain stored field | Yes |
| Field named `state` | **No** — a duplicate starts in the initial state |
| One-to-many | **No** |
| Many-to-many | Yes — the links are copied, the target records are not |
| Computed field | **No**, unless it is stored *and* explicitly not read-only |
| Related field | **No** |
| Company-dependent field | **No** |
| Audit fields | No — the copy is newly created |
| Identifier | No |

Any field may override the default by declaring whether it is copied.

### 19.2 The algorithm

1. Compute the values to copy: for each record of the set, every field marked as copied, in the write representation, with the archive filter switched off so that archived related records are still seen.
2. Overlay the caller's overrides.
3. For each one-to-many field that **is** marked copied, recursively compute the values of its lines and turn them into Create commands, so the lines are duplicated rather than shared.
4. Create the new records.
5. Copy translations: for every translatable field that was copied, copy every language's value, except for fields the caller excluded.
6. Return the new record set, in the same order as the source.

### 19.3 Consequences

- Duplicating a document duplicates its lines only if the line field is explicitly marked copied — which every document entity does.
- Duplicating a record with a unique code fails on the uniqueness constraint unless the entity overrides the copied values to derive a new code. Entities that need this declare a rule appending a suffix.
- External identifiers are not copied as such; a copy receives a derived identifier with a random suffix ([package system, section 9.7](package-system.md#97-copying)).

---

## 20. The filter grammar

A **filter** — the value passed to every search, every record rule, every relational field's candidate restriction and every action's fixed criteria — is a first-order logical expression over conditions on fields.

### 20.1 The two notations at a glance

The **list notation** is what travels over the transport and what appears in stored records. It is a flat list in prefix notation mixing conditions and connectives: a triple is a condition, `&` conjoins the next two items, `|` disjoins them, and `!` negates the next item. Items adjacent with no connective between them are conjoined, and the empty list means "everything".

| Filter | Meaning |
|---|---|
| `[]` | Every record |
| `[('state', '=', 'posted')]` | Posted records |
| `[('state', '=', 'posted'), ('amount', '>', 100)]` | Posted **and** over one hundred |
| `['\|', ('state', '=', 'draft'), ('state', '=', 'posted')]` | Draft **or** posted |
| `['&', '!', ('state', '=', 'cancel'), '\|', ('a', '=', 1), ('b', '=', 2)]` | Not cancelled, and either the first field is 1 or the second is 2 |

The **tree notation** is the internal form: a node is the constant true, the constant false, a condition, a conjunction, a disjunction, a negation, or a custom node carrying its own compilation rule. Filters are immutable; combining them produces new ones.

A condition is a triple of a field expression, an operator and a value. The field expression names a field of the entity, a path of relational segments, a granularity of a date field, a property of a custom-properties field, the identifier, or the display name.

The complete grammar — the operator set with its standard, convenience and internal operators, value normalisation, path decomposition, the empty-value rules and their compiled forms, the semantics of every operator on every field type, the resolution of an existence condition into a sub-query or a join, the hierarchy operators, in-memory evaluation, the four optimisation stages and the merging of sibling conditions — is specified in [record operations and query notation, section 4](record-operations-and-query-notation.md#4-the-filter-notation). Nothing of the grammar is restated here.

### 20.2 Where the field system contributes to the grammar

Three parts of this document decide how a condition on a field compiles, and a rebuild must keep them consistent with the grammar:

| Contribution | Section |
|---|---|
| The declared empty value of each type, which a filter treats as "not set" | [Section 6](#6-the-field-types) |
| The column expression of a company-dependent field, which is the current company's entry with the fallback applied, and the extra emptiness test that lets its partial index be used | [Section 11.5](#115-the-fallback) |
| The column expression of a translatable field, which is the current language's entry with the source-language fallback | [Section 12.3](#123-storage) |
| The generated search rule of a non-stored computed or related field, which rewrites a condition into one the query builder can serve | [Section 8.6](#86-the-filter-rule) and [section 10.3](#103-derived-behaviour) |
| Whether a field may be filtered on, sorted by, grouped by or aggregated at all | [Section 8.8](#88-derived-capabilities-of-a-computed-field) |

### 20.3 Filters as values

Filters appear in stored records in three forms, and the difference matters:

| Form | Where | Evaluation |
|---|---|---|
| A literal filter | An action's fixed criteria; a record rule with no variables | Parsed and used directly |
| An expression producing a filter | A record rule; a relational field's candidate restriction | Evaluated in a restricted evaluation context providing the acting user, the current company, the current date and a resolver from external identifier to identifier; the result must be a filter |
| An expression evaluated by the client | A relational field's candidate restriction declared as text | Sent to the client as text and evaluated there against the record being edited, so that the candidates can depend on unsaved values |

The third form is the only one that can depend on unsaved form state, and it is therefore the only one that is **not** enforced on the server. A candidate restriction expressed that way is guidance, not a constraint ([the security model, section 20](security-model.md#20-what-is-not-enforcement)); a constraint must also exist as a declared validation.

---

## 21. Field metadata exposed to clients

The describe-fields operation returns, per field, the attributes a client needs. A field the acting user may not read is omitted entirely.

| Attribute | Meaning |
|---|---|
| Name | The field name |
| Type | One of the sixteen types |
| Label | Translated |
| Help text | Translated |
| Required | |
| Read-only | True if declared read-only **or** if the acting user may not write the field |
| Stored | |
| Manual | Whether the field is user-defined |
| Related path | Present when the field is related |
| Company-dependent | |
| Group restriction | |
| Triggers a user default | |
| Included in re-importable exports by default | |
| Exportable | |
| Dependencies | The declared dependency paths |
| Searchable | Whether a condition on the field can be compiled: true when the field is stored with a column, or declares a filter rule |
| Sortable | Whether the field can be ordered by |
| Groupable | Whether the field can be grouped by |
| Aggregate | The aggregate a grouped screen applies, or none |
| Value label when empty | |
| Selection | For selection and polymorphic-reference fields: the list of pairs, labels translated |
| Digits, minimum display digits | For decimal numbers |
| Currency field | For monetary amounts |
| Target entity | For relational fields |
| Client-side filter | For relational fields; either a literal filter or the text expression the client evaluates |
| Client-side context | For relational fields |
| Inverse field name | For one-to-many fields |
| Hierarchy operators allowed | For relational fields: whether the target supports them |
| Entity field name | For reference-by-identifier fields |
| Attachment storage | For binary fields |
| Maximum length, trim | For short text |
| Sanitisation settings | For markup |
| Translatable | For text fields |
| Definition record and definition field | For custom-properties fields |

---

## 22. Deriving the database schema

Installing or updating a package brings the tables into agreement with the registry. The sequence below is what [the package system](package-system.md) invokes; it is stated here because what it produces is decided entirely by field attributes.

### 22.1 Tables

For every persistent or transient entity with an automatic table, the table is created if it is missing, with a single identifier column that is the primary key. Columns are then created, one per stored field that has a column type, in an order that groups identical storage types together.

An entity without an automatic table is left to its own initialisation; the platform neither creates nor alters it.

### 22.2 Columns

- A missing column is created with the field's column type and a comment holding the field's label.
- An existing column whose underlying type differs from the field's type is converted. For translatable fields, and for columns that already hold key-value documents, the conversion is the translation-aware one, which wraps each existing value under the source-language key.
- A short-text column whose declared bound is smaller than the field's declared size, or which is bounded while the field is not, is converted to the new bound.
- Columns present in the table but matching no field are detected and reported; they are **not** dropped automatically.

### 22.3 Derived stored columns filled directly

When a stored related field of exactly two segments is added to a table for the first time, and the traversed many-to-one is itself stored and not derived, and the target field is stored, not derived, not attachment-backed binary content and not a to-many field, the column is filled with a single join-and-set statement instead of by running the derivation record by record. Otherwise the field is scheduled for recomputation over the whole table, in batches.

### 22.4 Indexes

For every stored field with a column, the expected index name is the table name, an underscore, the field name, an underscore and the word `index`.

| Declared index kind | Access method | Indexed expression | Partial condition |
|---|---|---|---|
| Balanced tree, also written as simply "true" | Balanced tree | The column | None |
| Balanced tree excluding empty values | Balanced tree | The column | The column is set |
| Balanced tree excluding empty values, on a company-dependent field | Balanced tree | Whether the column is set | The column is set |
| Three-character-sequence index | Inverted index over three-character sequences | The column, or, for a translatable field, the concatenation of all language values; wrapped in the accent-folding function when that function is available and deterministic | None |
| None | No index is created | | |

Rules:

1. An existing index whose access method does not match the expected one is dropped and recreated.
2. An index that exists while the field declares none is **kept** and reported in the log, so as not to fight a deliberate database tuning.
3. On a translatable field only the three-character-sequence index is honoured; any other declared index is ignored with a warning.
4. A three-character-sequence index is created only when the database offers that index type.
5. The accent-folding wrapper is applied to three-character-sequence indexes only, because accent folding is applied only to case-insensitive pattern matching.

### 22.5 Foreign keys

- A many-to-one produces a foreign key from its column to the target table's primary key, carrying its declared deletion policy — unless the field is company-dependent, or the owning table or the target table is not an ordinary table, or the target entity has no automatic table.
- A many-to-many produces two foreign keys on its association table, one per side.
- Foreign keys are applied at the end of the schema update. An existing key whose target or policy differs is dropped and recreated.

### 22.6 Association tables

An association table that belongs to a package — that is, whose field is not a user-created field — is reflected as a record of the association-table catalogue, which makes it removable when the package is removed. Association tables of user-created fields are not reflected; they are dropped together with the field definition record.

---

## 23. User-created fields

A field may be created while the system is running, through the field definition entity, rather than being declared by a package. Such a field:

- carries the user-created marker, which forces its prefetch group to false, meaning it is never read together with other fields;
- is named with a reserved prefix by convention, so that it can never collide with a field a package may later declare;
- supports the same types and the same attributes as a declared field, with the definition stored as data;
- has its association table, for a many-to-many, dropped when its definition record is deleted;
- participates in every rule of this document identically, once the registry has been rebuilt.

Creating, changing or deleting a user-created field rebuilds the registry and synchronises the schema, exactly as installing a package does.

---

## 24. A worked example of an entity from end to end

Consider an entity Product Lot with: a translatable short-text name; a required link to Product whose deletion policy is `restrict`; a decimal quantity at the unit-of-measure precision; a company-dependent cost in the company currency; an archive flag; and a custom-properties field whose definition lives on the product.

**The schema derived from those declarations.**

| Column | Type | Notes |
|---|---|---|
| The identifier | Integer | Primary key |
| The creation instant | Timestamp | Audit field |
| The creating user | Integer | Foreign key to the user table, deletion policy `set null` |
| The last update instant | Timestamp | Audit field |
| The last updating user | Integer | Foreign key to the user table, deletion policy `set null` |
| The name | Key-value document | Translatable |
| The product | Integer, not null | Foreign key to the product table, deletion policy `restrict` |
| The quantity | Exact decimal | |
| The cost | Key-value document | Company-dependent, monetary |
| The archive flag | Boolean | |
| The properties | Key-value document | Custom properties |

Three indexes are created: a balanced tree on the product column; a balanced tree on whether the cost column is set, restricted to rows where it is set; and a three-character-sequence index over all language values of the name.

**Creating one record** while company 1 is current, with acting user 7, a language other than the source language, a unit-of-measure precision of three decimal places and a company currency with two, supplying the name "Lot A", the product, a quantity of 1.23456 and a cost of 10.005:

1. The identifier, the two instants and the two user fields are removed from the value map and then refilled: both user fields take the acting user, and both instants take the transaction timestamp.
2. Defaults are resolved for every field not supplied; the archive flag receives its declared default of true.
3. The custom-properties field is precomputed: the product's definition list is read and each property's default is applied.
4. The quantity is rounded to three decimal places, giving 1.235.
5. The cost is rounded to the company currency's two decimal places, giving 10.01. The company-dependent mapping is compared with the fallback; with no recorded default in place, the mapping becomes the single entry for company 1 with the value 10.01.
6. The name is stored with two entries, one under the source language and one under the environment's language, both holding "Lot A", because on insertion both receive the value.
7. The row is inserted, the generated identifier is read back, and every stored field absent from the insertion is put in the cache as empty.
8. Validations watching any of the name, the product, the quantity, the cost, the archive flag or the properties run with elevated rights.
9. The create right is checked again against the freshly created record, which applies record rules that may depend on the values just written.

**Reading the record afterwards** as the same user while company 2 is current and in a third language:

- The name returns that language's entry when present, otherwise the source-language entry, otherwise empty.
- The cost has no entry for company 2 and therefore returns the fallback recorded for company 2, or zero when there is none.
- The quantity returns 1.235.
- The display name returns the name rendered in that language.

---

## 25. Invariants a rebuild must preserve

1. An entity's transport name is its identity and is part of the external contract, as are stored selection codes and column names.
2. Abstract entities have no table; their fields become columns on every adopting entity.
3. Transient rows are never removed within five minutes of their last modification.
4. The four audit fields record the environment's acting user, not the effective one, and the database clock, not the worker clock.
5. Declaring a computation makes a field non-stored, elevated when stored, not copied and read-only by default.
6. Recomputation is driven by declared dependency paths traversed backwards through relations, with the before/after distinction for destructive changes.
7. A recursive dependency must be declared; without the declaration the traversal is wrong.
8. Related fields traverse field by field, not record by record, and take the first record at every multi-valued step.
9. A company-dependent value falls back to the per-entity default catalogue entry for the current company.
10. A translatable value falls back to the source language.
11. Defaults resolve in the order: context, recorded default (non-company-dependent), declared default, recorded default (company-dependent), embedded parent.
12. On-change runs only through the on-change operation, only on in-memory records, and never validates.
13. Declared validations run elevated, after the write, for rules whose declared fields were touched.
14. Database constraint violations are translated into the entity's message and are never retried.
15. In a filter, the field type's declared empty value and "not set" are the same thing, except on columns the registry knows to be not-null.
16. Filter traversal applies the target's record rules unless the field bypasses them, and switches the archive filter off.
17. Ordering by a many-to-one orders by the target's default ordering.
18. Archiving hides from searches but not from reads or relations.
19. All decimal and monetary rounding goes through one routine with an explicit step, an explicit tie-breaking method and a tolerance that corrects binary representation error.
20. Comparison at a precision rounds before subtracting; the zero test of a difference rounds after subtracting; the two are not interchangeable.
21. A materialised ancestor path is rewritten for a moved record and all its descendants, and a move that would create a cycle is refused.
22. A recorded default value is resolved by the ordering user, then company, then identifier, keeping the first value per field, and any change to one invalidates the whole record cache.
23. Company consistency is a validation, not a permission: it runs with elevated rights and with archived records visible.
24. An index the database has but the registry does not declare is kept and reported, never dropped.
25. A column the table has but no field claims is reported, never dropped.
26. A user-created field is never prefetched with other fields.

---

## 26. Acceptance criteria

### Entity kinds

**AC-ENT-1.** *Given* a transient entity with a maximum idle lifetime of one hour and a record modified two minutes ago, *when* the housekeeping job runs, *then* the record is not removed.

**AC-ENT-2.** *Given* a transient entity with a maximum record count of twenty and twenty-five rows, ten of them modified in the last five minutes, *when* the housekeeping job runs, *then* the ten recent rows survive and older rows are removed.

**AC-ENT-3.** *Given* a persistent entity declaring a many-to-one to a transient entity, *when* the registry is built, *then* the build fails naming the field.

**AC-ENT-4.** *Given* an abstract entity declaring three fields and three entities adopting it, *when* the schema is synchronised, *then* each of the three tables gains three columns and the abstract entity has no table.

### Field types

**AC-ENT-5.** *Given* a decimal-number field declaring two decimal places, *when* 12.345 is assigned, *then* the value read back is 12.35 and the stored value is 12.35.

**AC-ENT-6.** *Given* a decimal-number field declaring no digits, *when* 12.345 is assigned, *then* the value read back is 12.345.

**AC-ENT-7.** *Given* a monetary field whose currency has two decimal places, *when* 12.345 is assigned, *then* 12.35 is stored. *Given* a currency with zero decimal places, *then* 12 is stored.

**AC-ENT-8.** *Given* a monetary field and a record set of two records with different currencies, *when* one value is assigned to both, *then* the operation fails with "Got multiple currencies while assigning values of monetary field" followed by the field.

**AC-ENT-9.** *Given* a selection field whose declared codes are `draft` and `posted`, *when* `cancelled` is assigned, *then* the operation fails with "Wrong value for" followed by the entity, the field and the value.

**AC-ENT-10.** *Given* an instant field and an environment whose time zone is three hours ahead of Coordinated Universal Time, *when* the calendar date for the fifth of March is assigned, *then* the stored instant is the fourth of March at twenty-one hundred hours Coordinated Universal Time.

**AC-ENT-11.** *Given* a many-to-many between two entities with no explicit table name, *when* the registry is built, *then* the association table's name is the two table names sorted alphabetically, joined by an underscore, followed by the relation suffix, and both ends derive the same name.

**AC-ENT-12.** *Given* two many-to-many fields between two different entity pairs that would derive the same table and columns, *when* the registry is built, *then* the build fails with "Many2many fields" naming both.

**AC-ENT-13.** *Given* a required many-to-one declaring the deletion behaviour that clears the value, *when* the registry is built, *then* the build fails stating that only restrict and cascade make sense.

### Relational writing

**AC-ENT-14.** *Given* a one-to-many holding lines A and B, *when* the commands Clear, then Link A are applied, *then* the field holds only A. *When* instead Link A, then Clear are applied, *then* the field is empty.

**AC-ENT-15.** *Given* a many-to-many and two owning records, *when* a Create command is applied, *then* **one** target record is created and both owners link to it.

**AC-ENT-16.** *Given* a one-to-many and two owning records, *when* a Create command is applied, *then* **two** target records are created, one per owner.

**AC-ENT-17.** *Given* an entity that forbids privileged relational commands, *when* an elevated environment applies a Link command on a relation to it, *then* the operation is refused.

### Computation

**AC-ENT-18.** *Given* a stored computed total on an order depending on `line_ids.subtotal`, *when* a line's subtotal changes, *then* the order is marked pending and the total is recomputed at flush.

**AC-ENT-19.** *Given* the same, *when* a line is deleted, *then* the order's total is recomputed — which requires the affected order to have been collected before the deletion.

**AC-ENT-20.** *Given* a non-stored computed field depending on a related field, *when* the dependency changes, *then* the cached value is removed rather than marked pending, and the next read recomputes it.

**AC-ENT-21.** *Given* a field whose computation is shared with two other fields, *when* any one of the three is read, *then* the rule is invoked once and all three are in the cache.

**AC-ENT-22.** *Given* a stored computed field whose computation reads the field's own previous value, *when* the computation runs, *then* it does not recursively trigger itself, because the field was removed from the pending map before the rule was invoked.

**AC-ENT-23.** *Given* a parent total depending on `child_ids.total` where a child's total depends on its own `child_ids.total`, and the field **not** declared recursive, *when* a leaf changes, *then* only the immediate parent is recomputed. *Given* the field declared recursive, *then* every ancestor is recomputed.

**AC-ENT-24.** *Given* a stored computed field, *when* it is computed, *then* the computation runs elevated. *Given* the same field declared non-stored, *then* the computation runs as the acting user.

**AC-ENT-25.** *Given* a computed field with an inverse rule, *when* the field is assigned, *then* the inverse runs with the field protected, and the field is not immediately recomputed.

**AC-ENT-26.** *Given* a stored computed field declared precomputed and a creation that supplies a default for it, *when* the record is created, *then* the default is used and no precomputation happens.

### Related fields

**AC-ENT-27.** *Given* a related field over `order_id.partner_id` with no explicit label, *when* the field metadata is requested, *then* the label is the label of the party field on the order.

**AC-ENT-28.** *Given* a related field whose declared type differs from the target's, *when* the registry is built, *then* the build fails with "Type of related field" naming both.

**AC-ENT-29.** *Given* two hundred lines whose related party name is read in a loop, *when* the reads happen, *then* the traversal is performed field by field and at most three queries reach the database.

**AC-ENT-30.** *Given* a related field whose traversal is refused for lack of access, *when* it is read, *then* the refusal message ends with a paragraph naming the entity through which the access was implicit.

**AC-ENT-31.** *Given* a writable related field on an in-memory record whose target is a persisted record, *when* the field is assigned, *then* the target is **not** written.

### Company-dependent and translatable values

**AC-ENT-32.** *Given* a company-dependent many-to-one with no entry for company 2 and a fallback recorded for company 2, *when* the field is read in company 2, *then* the fallback is returned.

**AC-ENT-33.** *Given* the same, *when* a value equal to the fallback is supplied at creation, *then* the stored mapping remains empty.

**AC-ENT-34.** *Given* a company-dependent field with entries for companies 1 and 3, *when* a search filters on it in company 3, *then* only company 3's entries and the fallback participate.

**AC-ENT-35.** *Given* a translatable field whose source value is "Invoice" and whose French value is "Facture", *when* it is read with no language, *then* "Invoice" is returned; *when* read in French, "Facture"; *when* read in a language with no entry, "Invoice".

**AC-ENT-36.** *Given* the same field, *when* the source value is changed with no language active and the French value still equals the old source value, *then* the French value follows the change. *Given* the French value had been changed to something else, *then* it does not.

### Defaults and on-change

**AC-ENT-37.** *Given* a field with a declared default of ten and a recorded administrator default of twenty, *when* defaults are requested, *then* twenty is returned.

**AC-ENT-38.** *Given* the same plus a context key setting the default to thirty, *when* defaults are requested, *then* thirty is returned.

**AC-ENT-39.** *Given* a company-dependent field with a declared default of ten and a recorded fallback of twenty, *when* defaults are requested, *then* **ten** is returned, because the declared default precedes the fallback for company-dependent fields.

**AC-ENT-40.** *Given* a default supplied through the context for a one-to-many that contains a Delete command, *when* defaults are requested in a restricted environment, *then* deletion rights are checked on the target and the request is refused if they are absent.

**AC-ENT-41.** *Given* a form with an empty changed-field list, *when* the on-change operation runs, *then* defaults are filled in, computations run, every reaction rule runs once, and every field of the specification is returned.

**AC-ENT-42.** *Given* a reaction rule that refuses a value, *when* the same value is written over the transport, *then* the write succeeds — the rule is not a constraint.

**AC-ENT-43.** *Given* two reaction rules producing warnings in one round trip, *when* the operation returns, *then* one warning is returned whose title is the word for warnings and whose message concatenates both titles and messages separated by blank lines.

### Validation

**AC-ENT-44.** *Given* a declared validation over fields A and B, *when* only field C is written, *then* the validation does not run.

**AC-ENT-45.** *Given* a declared validation that reads field D without declaring it, *when* D changes, *then* the validation does not run and the invariant is silently violated. (This is the specified behaviour; it is why declaring every read field is mandatory.)

**AC-ENT-46.** *Given* a unique constraint on a code and two records created with the same code in one transaction, *when* the transaction flushes, *then* a validation failure is raised whose text begins "The operation cannot be completed: " followed by the constraint's declared message, and the transaction is not retried.

**AC-ENT-47.** *Given* an entity with automatic company consistency and a field marked for checking that points at a record of another company, *when* the record is written, *then* the write is refused with the message beginning "Uh-oh! You’ve got some company inconsistencies here:" and ending "To avoid a mess, no company crossover is allowed!".

**AC-ENT-48.** *Given* the same field pointing at a record whose company is empty, *when* the record is written, *then* the write succeeds.

### Ordering, names, archive

**AC-ENT-49.** *Given* an entity whose default ordering is by a many-to-one to Party, and Party's default ordering is by name, *when* a search runs, *then* the results are ordered by party name.

**AC-ENT-50.** *Given* an ordering term naming a non-stored field, *when* a search runs, *then* it is refused.

**AC-ENT-51.** *Given* an entity with no display-name field, *when* the display name of record 7 is read, *then* it is the transport name, a comma and 7.

**AC-ENT-52.** *Given* an entity whose name-search fields are the name and the reference, *when* a name search for "AB" runs with the containment operator, *then* the filter is the disjunction of the two conditions; *when* it runs with the negated containment operator, *then* it is the conjunction.

**AC-ENT-53.** *Given* a record with the archive flag false, *when* an ordinary search runs, *then* the record is absent; *when* the search filter mentions the archive flag, *then* the implicit condition is not added; *when* the record is read by identifier, *then* it is returned.

**AC-ENT-54.** *Given* an invoice whose party is archived, *when* a filter traverses the party relation, *then* the archived party still participates and the invoice matches.

### The filter grammar

**AC-ENT-55.** *Given* the filter list holding a conjunction connective, a negation connective, a condition, a disjunction connective and two conditions, *when* it is parsed, *then* the tree is the conjunction of the negation of the first condition with the disjunction of the other two.

**AC-ENT-56.** *Given* a filter list whose connective has too few operands, *when* it is parsed, *then* it is refused with "Domain() malformed domain" followed by the list.

**AC-ENT-57.** *Given* a nullable integer column, *when* the filter tests equality with zero, *then* rows whose value is zero **and** rows whose value is unset match.

**AC-ENT-58.** *Given* the same column carrying a not-null constraint known to the registry, *when* the filter tests equality with the empty value only, *then* the filter matches nothing.

**AC-ENT-59.** *Given* a nullable text column, *when* the filter tests the negated containment of "x", *then* rows whose value does not contain "x" **and** rows whose value is unset match.

**AC-ENT-60.** *Given* a many-to-one compared with the text "Acme", *when* the filter is normalised, *then* it becomes a traversal condition on the target's display name.

**AC-ENT-61.** *Given* a many-to-one compared with the text "Acme" using the greater-than operator, *when* the filter is normalised, *then* it is refused with "Inequality not supported for relational field using a string".

**AC-ENT-62.** *Given* a membership test against an empty collection, *when* the filter is normalised, *then* it becomes "nothing"; *given* a non-membership test against an empty collection, *then* it becomes "everything".

**AC-ENT-63.** *Given* a hierarchy descendants operator on an entity with a materialised hierarchy, *when* the filter is normalised at the full level, *then* it becomes a disjunction of path-prefix conditions and performs no recursive search.

**AC-ENT-64.** *Given* the same on an entity without a materialised hierarchy, *when* the filter is normalised, *then* the descendants are found by repeated searching, elevated, with the archive filter switched off.

**AC-ENT-65.** *Given* a traversal condition on a relational field that does not bypass access, *when* the filter is compiled for a restricted user, *then* the target entity's record rules are applied to the sub-query.

**AC-ENT-66.** *Given* the same field declared to bypass access, *when* the filter is compiled, *then* the target's record rules are not applied.

**AC-ENT-67.** *Given* the case-insensitive containment operator and a tenant whose database offers accent folding, *when* the filter is compiled, *then* both the column and the pattern are folded, so that a search for "eleve" matches "élevé".

### Rounding and decimal arithmetic

**AC-ENT-68.** *Given* the value 2.675 and a step of 0.01 with half-up rounding, *when* it is rounded, *then* the result is 2.68, because the tolerance corrects the binary representation of the tie.

**AC-ENT-69.** *Given* the value 1.3 and a step of 0.5 with half-up rounding, *when* it is rounded, *then* the result is 1.5.

**AC-ENT-70.** *Given* the values 2.5 and 3.5 and a step of 1 with half-even rounding, *when* they are rounded, *then* the results are 2 and 4.

**AC-ENT-71.** *Given* the values 0.006 and 0.002 at two decimal places, *when* they are compared, *then* the result is plus one; *when* the zero test is applied to their difference, *then* it reports zero.

**AC-ENT-72.** *Given* 10.00 divided by 3.00 at two decimal places, *when* the Euclidean division runs, *then* the quotient is 3 and the remainder is 1.00.

**AC-ENT-73.** *Given* the value −0.004 at two decimal places, *when* it is rendered, *then* the text is "0.00" and never carries a minus sign.

### Hierarchies, defaults and the schema

**AC-ENT-74.** *Given* a tree of three categories with materialised ancestor paths, *when* the middle one is moved to a new parent, *then* its own path and the paths of all its descendants are rewritten in the same transaction.

**AC-ENT-75.** *Given* the same tree, *when* a record is written with one of its own descendants as its parent, *then* the write is refused with "Recursion Detected."

**AC-ENT-76.** *Given* a recorded default for a field with a user and no company, and another with a company and no user, *when* defaults are resolved for that user while that company is current, *then* the ordering by user, then company, then identifier decides, and exactly one value is returned for the field.

**AC-ENT-77.** *Given* a recorded default whose serialised value does not convert to the field's type, *when* it is written, *then* it is refused with "Invalid value in Default Value field. Expected type '\<field type\>' for '\<qualified field name\>'."

**AC-ENT-78.** *Given* any recorded default is created, written or deleted, *when* the next read of a company-dependent field happens, *then* the value is recomputed, because the whole record cache was invalidated.

**AC-ENT-79.** *Given* a table carrying an index the registry does not declare, *when* the schema is synchronised, *then* the index is kept and reported in the log.

**AC-ENT-80.** *Given* a table carrying a column that matches no field, *when* the schema is synchronised, *then* the column is reported and not dropped.

**AC-ENT-81.** *Given* a translatable field declaring a balanced-tree index, *when* the schema is synchronised, *then* the declared index is ignored with a warning and only a three-character-sequence index may be created.

**AC-ENT-82.** *Given* a stored related field of exactly two segments added to an existing table, *when* the schema is synchronised, *then* the column is filled by a single join-and-set statement rather than record by record.

**AC-ENT-83.** *Given* a user-created field, *when* any other field of the same entity is read, *then* the user-created field is not read with it, because its prefetch group is false.

**AC-ENT-84.** *Given* a user-created many-to-many field, *when* its definition record is deleted, *then* its association table is dropped.

**AC-ENT-85.** *Given* a transient entity declaring a maximum idle lifetime of 0.2 hours and a maximum record count of 20, and a table holding 55 rows of which 10 were updated within the last 300 seconds, 12 between 300 and 600 seconds ago and 33 more than 720 seconds ago, *when* one cleaning pass runs, *then* the age-based pass deletes the 33 rows, the count-based pass deletes the 12 rows because 22 exceeds 20, 10 rows survive, and the pass reports that nothing remains to be examined.

**AC-ENT-86.** *Given* a transient entity declaring a maximum idle lifetime of 0.001 hours (3.6 seconds), *when* a cleaning pass runs, *then* the threshold used is the current time minus 300 seconds, because the floor replaces any smaller value, and no row updated within the last five minutes is deleted.

**AC-ENT-87.** *Given* an image field with verified resolution, *when* a 9,000 × 6,000 image (54,000,000 pixels) is written, *then* the write is refused; *and given* an 8,000 × 6,000 image (48,000,000 pixels), *then* it is accepted and resized to the declared maximum width and height.

**AC-ENT-88.** *Given* a binary field and a text value that is neither valid base-64 nor plain seven-bit text, *when* it is written, *then* the write fails with "ASCII characters are required for " the value " in " the field name.

**AC-ENT-89.** *Given* a value whose first character is `P` and which decodes to a scalable vector image, *when* a user who is not a member of the settings group writes it to a binary field, *then* the write is refused with "Only admins can upload SVG files."

**AC-ENT-90.** *Given* a custom-property definition carrying a `selection` key on a property whose type is `char`, *when* the definition is written, *then* the write fails with "Invalid property parameter 'selection'".

**AC-ENT-91.** *Given* a custom-property definition whose name is `Colour Code`, *when* the definition is written, *then* the write fails with "Wrong property field value name 'Colour Code'." because the name holds a capital letter and a space.

**AC-ENT-92.** *Given* a custom-property definition of type `many2one` whose target entity was removed with its package, *when* the definition is read back, *then* the entry is returned with its target cleared and its filter removed, and no failure is raised.

**AC-ENT-93.** *Given* one write naming a monetary amount, the currency field of that amount and a one-to-many of lines, *when* the write runs, *then* the currency is applied first, the amount is rounded to that currency's precision, and the line commands run last.

**AC-ENT-94.** *Given* a one-to-many written with the command list Create, Link, Delete in that order, *when* the write runs, *then* the deletion is performed first, the creation second and the link last, whatever the order in the list.

**AC-ENT-95.** *Given* a record being created and a command list holding a single Set over two existing records that belong to another owner, *when* the create runs, *then* the Set degrades to a Link of those two records and no record is unlinked from its current owner.

**AC-ENT-96.** *Given* a not-null constraint added to a computed stored field of a table that already holds rows, *when* the schema is synchronised, *then* the existing rows are scheduled for recomputation rather than filled with the field's default, and the constraint is added last.

**AC-ENT-97.** *Given* a deletion refused by a foreign key, *when* the failure is reported, *then* the text is "The operation cannot be completed: " followed by "Another model is using the record you are trying to delete.", a blank line, "The troublemaker is: " the entity display, "Thanks to the following constraint: " the field display, and "How about archiving the record instead?"

**AC-ENT-98.** *Given* a markup field declaring overridable sanitising and a stored value that the sanitiser would change, *when* a user who is not a member of the bypass group writes to it, *then* the write is refused with the restricted-content message naming the entity and the field, and a difference report is recorded in the diagnostic log.

---

## 27. Reconciliation notes

Five behaviours in this document contradict the reading a careful person would most naturally arrive at, and three organisational decisions about which document owns which topic are recorded with them. Each behaviour below was verified against the running system.

1. **Where the filter grammar lives.** The grammar could sit with the fields it constrains or with the operations that consume filters. It lives in [record operations and query notation, section 4](record-operations-and-query-notation.md#4-the-filter-notation). [Section 20](#20-the-filter-grammar) keeps what belongs to fields: the two notations at a glance, the three contributions this document makes to how a condition compiles, and the three forms in which a filter is stored. Nothing was dropped in the move, and no rule is stated in both places.
2. **The rounding rule.** Describing the rounding of a decimal number as "half away from zero" and stopping there is wrong. The routine takes an explicit step, applies one of five tie-breaking methods, and adds a tolerance derived from the magnitude of the normalised value in order to correct binary representation error; without the tolerance, 2.675 rounds to 2.67 rather than 2.68. [Section 6.4](#64-decimal-number) states the full routine, and criteria AC-ENT-68 to AC-ENT-70 assert it.
3. **Comparison against the zero test.** "Compare at a precision" and "is the difference zero at a precision" look like the same operation and are not: comparison rounds each operand before subtracting, while the zero test rounds the difference. Both are specified, with the worked example that separates them, and criterion AC-ENT-71 asserts the difference.
4. **Where company consistency is specified.** [Section 15.5](#155-company-consistency) keeps the part that is a property of fields — the declaration, which fields are checked, the ordinary against company-dependent split, and the elevated archived-visible reads. The algorithm, the entity families and the message are in [multi-company, section 6](multi-company.md#6-the-company-consistency-check).
5. **The precedence of a recorded default.** A recorded default is not simply consulted after the declared default. It is consulted **before** the declared default for an ordinary field and **after** it for a company-dependent field, which is why the resolution order of [section 13.1](#131-the-resolution-order) has five steps rather than four. Invariant 11 of [section 25](#25-invariants-a-rebuild-must-preserve) states the order.
6. **Where the schema derivation belongs.** Every rule that derives the database schema is decided by a field attribute, so the rules are here, in [section 22](#22-deriving-the-database-schema); [the package system](package-system.md) states only when the synchronisation runs.
7. **Naming.** The mechanism by which a record carries the fields of a parent record is called **embedding**, and the many-to-one that carries it is called the **link field**, matching [inheritance and extension](inheritance-and-extension.md). The words "delegation" and "delegate field" name the same mechanism in other treatments of this subject and are not used here.
8. **The stored type discriminators of a custom property.** The fourteen values listed in [section 6.16](#616-custom-properties) are the values written into the definition document and read back by integrations, so they are reproduced exactly, with their meaning in words beside them, rather than being renamed into the type vocabulary this document uses for ordinary fields.
9. **Acceptance criteria identifiers.** The scenarios of this document are numbered in one series with the prefix `AC-ENT`.

---

## Related documents

- [Architecture](architecture.md) — the registry, the environment, the record set and the unit of work.
- [Record operations and query notation](record-operations-and-query-notation.md) — the generic operations that read and write these fields, and the complete filter grammar.
- [Inheritance and extension](inheritance-and-extension.md) — how several packages contribute to one entity and one field, and what embedding a parent record means.
- [The security model](security-model.md) — field restrictions, record rules and the unrestricted actor.
- [Multi-company](multi-company.md) — the current company that decides a company-dependent value, and the full company consistency check.
- [Views and actions](views-and-actions.md) — how fields are presented and how the search grammar reaches this one.
- [The package system](package-system.md) — when the schema synchronisation of [section 22](#22-deriving-the-database-schema) runs.
- [Caching](../runtime/caching.md) — the record cache, the pending-write buffer and the recomputation schedule this document's derivations drive.
- [Translation](../runtime/translation.md) — the language catalogue and the collection of translatable terms behind [section 12](#12-translatable-values).
- [Identity and values](../data/persistence-identity-and-values.md) — identifiers, precision, rounding, dates and sequences in one place.
- [The physical data catalogue](../data/physical-data-catalog.md) — the tables, columns, indexes and constraints a full installation creates.
