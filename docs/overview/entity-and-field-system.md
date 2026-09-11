# The entity and field system

Everything the system stores, computes, validates, displays or exchanges is a **field of an entity**. This document specifies the entity and field machinery in full: the three entity kinds and how each is stored; every field type with its storage form, value semantics and attributes; computed fields, their declared dependencies, the choice to store or not, their inverse and search rules, and the recomputation algorithm including traversal across relations; related fields; company-dependent values; translatable values; defaults and their resolution order; on-change behaviour and its limits; validation rules and database constraints; ordering, display names and name searching; the archive flag; audit fields; and the filter grammar in full, with every operator's exact semantics.

Read [the architecture](architecture.md) first: this document assumes the registry, the environment, the record set and the unit of work.

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
22. [Invariants a rebuild must preserve](#22-invariants-a-rebuild-must-preserve)
23. [Acceptance criteria](#23-acceptance-criteria)

---

## 1. Entities

An **entity** is a named collection of typed fields with declared behaviour. Its identity is its **transport name**: a dotted lowercase name such as `res.partner` (party record), `account.move` (journal entry) or `stock.move` (stock move). The transport name is part of the external contract: clients name it, stored references contain it, external identifiers record it, and access rights are granted against it.

### 1.1 Naming rules

1. A transport name is a sequence of segments separated by dots. Each segment consists of lowercase letters, digits and underscores. A name that does not match is rejected at registry build time with **"The _name attribute "** followed by the name **" is not valid."**
2. By convention the first segment names the capability family (`res` for shared resources, `ir` for platform infrastructure, `account`, `stock`, `sale`, `hr`, and so on) and the remaining segments name the thing.
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

**Housekeeping rules.** The job runs periodically; per entity, cleaning actually happens at most once every five minutes.

| Setting | Default | Meaning |
|---|---|---|
| Maximum record count | Configured per installation | When the row count exceeds it, rows older than five minutes are removed. Zero means unlimited. |
| Maximum idle lifetime, in hours | Configured per installation | Rows whose last modification is older than this are removed. Zero means unlimited. |

Two rules are absolute:

1. **A row modified within the last five minutes is never removed**, whatever the settings say. A user filling in a dialog must not have it deleted underneath them. The age threshold used for a count-based clean is therefore raised to five minutes if it would be lower.
2. Each cleaning pass removes at most a bounded number of rows and reports whether more remain, so that one pass cannot lock the table for long.

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

1. Add the column if missing, with the field's default written into every existing row.
2. Attempt to add the not-null constraint.
3. If it fails, record a warning naming the entity and the field, leave the constraint off, and continue. The field remains required at the client level but is not enforced by the database.
4. At the end of the build, verify agreement: for every field the registry believes is not null, confirm that the column really is. The registry's record of which fields are truly not-null is what the filter compiler consults in order to decide whether a condition needs to consider empty values ([section 20.7](#207-empty-values-and-the-null-question)).

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

Stores a whole number in four bytes. The empty value is zero, and **zero and empty are indistinguishable**: the column may be null, and a null reads as zero. This matters for filtering ([section 20.7](#207-empty-values-and-the-null-question)).

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
| Sanitise style | false | Remove inline style declarations. |
| Sanitise form | true | Remove form elements. |
| Strip style | false | Remove the style attribute entirely. |
| Strip classes | false | Remove class attributes. |
| Translatable | false, or term-by-term | A markup field may be translated as a whole value or term by term, which translates the text nodes while keeping the markup shared. |

Sanitisation is applied on assignment, so the stored value is already clean. A field that must accept arbitrary markup — a rendering template, for instance — switches sanitisation off, and the capability that owns it is then responsible for controlling who may write it.

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
| Stored as an attachment | true | When true the bytes live in an attachment record, and the entity's column holds nothing. When false the bytes are stored inline in the column, encoded. |
| Maximum width, maximum height | none | For the image specialisation: the dimensions the value is resized to on write. |
| Verify resolution | true | For the image specialisation: whether an oversized image is rejected rather than resized. |

Rules:

1. With attachment storage, writing a value creates or replaces an attachment record whose owner is this record and this field, and whose content is deduplicated by digest in the file store. Reading yields the bytes.
2. Reading a binary field with the size-instead-of-content context key set yields a human-readable size instead of the bytes, so that a list screen can show "3.20 kB" without transferring the content.
3. The **image** specialisation additionally resizes on write to fit within the declared maximum dimensions, preserving the aspect ratio, and exposes derived fields at fixed sizes (for example a 1024-pixel, a 512-pixel, a 256-pixel and a 128-pixel variant) that are computed from the original.
4. Binary fields are never prefetched, because transferring content for records nobody asked about is expensive.

### 6.15 Structured document

Stores an arbitrary structured value — nested mappings, lists, numbers, strings, booleans — as a structured document column.

Rules:

1. The value is stored and returned as-is. No schema is enforced.
2. Structured-document fields are not translatable and not company-dependent.
3. Filtering on them is limited to the operations the database offers on structured documents; the platform exposes equality on the whole value and containment on a named key.
4. The empty value is false.

### 6.16 Custom properties

Two paired types let a user add fields to records of an entity without changing the schema.

**The definition field** holds the list of user-defined field definitions: for each, a name, a label, a type, and type-specific attributes such as the selection values, the target entity of a relation, or the tag list with colours. It lives on a *container* record — a project, a sales team, a category — so that all records belonging to that container share the same set of user-defined fields.

**The value field** holds the values for one record, as a mapping from definition name to value. It declares which field of the record points at the container and which field of the container holds the definitions.

| Attribute of the value field | Meaning |
|---|---|
| Definition record | The name of the many-to-one on this entity that leads to the container. |
| Definition record field | The name of the field on the container that holds the definitions. |

Rules:

1. Reading the value field returns, for each defined property, its definition merged with the record's value, so that a client can render it without a second call.
2. Writing accepts either the full list with values or a mapping of name to value.
3. Changing the container of a record re-resolves which properties apply; values whose property no longer exists are dropped by a cleaning pass that runs after every write that could have changed the definitions.
4. Properties can be filtered and grouped: a condition names the value field, a full stop and the property name. Grouping by a property of relational or selection type expands to the defined values.
5. Properties are not columns. No index exists for them beyond what the structured-document column offers, so filtering on a property is slower than filtering on a field.

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

### 15.4 Deletion guards

A **deletion guard** is a rule declared to run before records are deleted. It may refuse.

| Variant | When it runs |
|---|---|
| Always | Before any deletion of records of the entity |
| Except during package removal | Before ordinary deletions, but skipped while a package is being removed, so that removing a package is not blocked by a guard meant for users |

Guards run before anything is deleted, on the whole set at once.

### 15.5 Company consistency

Relational fields marked for company consistency are checked so that a record never links to a record of an incompatible company.

**When it runs.** On every creation and every write, for entities that switch the automatic check on; otherwise explicitly, wherever a capability calls for it.

**Algorithm.**

1. Determine which fields to check. If no names are given, or if the company field or company-set field is among them, check every field of the entity; otherwise only the named ones.
2. Partition the marked relational fields into ordinary ones and company-dependent ones.
3. For each record:
   - **Ordinary fields.** Determine the record's companies: the record itself if the entity *is* the company entity; otherwise the record's company field if it has one; otherwise its company-set field if it has one; otherwise skip with a warning naming the entity and the fields and stating that the entity has neither. For each marked field, read its value unrestricted and check that every linked record's company is either empty or among the record's companies.
   - **Company-dependent fields.** Check against the **environment's current company** instead, because the per-company value belongs to the company it was written for.
4. Collect the inconsistencies. If any, refuse with a message built as follows:
   - First line: **"Uh-oh! You’ve got some company inconsistencies here:"**
   - Then up to five lines, one per inconsistency:
     - when the record is a company: **"- Record is company “"** name **"” while “"** field label **"” ("** field name **": "** the linked records' names **") belongs to another company."**
     - when the record links to itself through its own company field: **"- Only a root company can be set on “"** record name **"”. Currently set to “"** company name **"”"**
     - otherwise: **"- “"** record name **"” belongs to company “"** company names **"” while “"** field label **"” ("** field name **": "** linked record names **") belongs to another company."**
   - Last line: **"To avoid a mess, no company crossover is allowed!"**

**The compatibility rule.** A linked record is compatible when its company is empty or among the owning record's companies. An empty company means "shared by all companies", which is why shared reference data can be linked from any company's documents. An entity may override the rule — for example the user entity checks membership of the company set rather than equality of the company field.

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

### 20.1 Two notations

The **list notation** is what travels over the transport and what appears in stored records. It is a flat list in prefix notation mixing conditions and connectives:

| Item | Meaning |
|---|---|
| A triple | A condition. |
| `&` | Conjunction of the next two items. |
| `|` | Disjunction of the next two items. |
| `!` | Negation of the next item. |

Items that are adjacent with no connective between them are conjoined. The empty list means "everything".

Examples:

| Filter | Meaning |
|---|---|
| `[]` | Every record |
| `[('state', '=', 'posted')]` | Posted records |
| `[('state', '=', 'posted'), ('amount', '>', 100)]` | Posted **and** over one hundred |
| `['|', ('state', '=', 'draft'), ('state', '=', 'posted')]` | Draft **or** posted |
| `['&', '!', ('state', '=', 'cancel'), '|', ('a', '=', 1), ('b', '=', 2)]` | Not cancelled, and (a is 1 or b is 2) |

The **tree notation** is the internal form: a node is either the constant true, the constant false, a condition, a conjunction of nodes, a disjunction of nodes, a negation of a node, or a custom node carrying its own compilation rule. Filters are immutable; combining them produces new ones.

Parsing the list notation into the tree is a right-to-left stack walk: conditions push, `&` and `|` pop two and push the combination, `!` pops one and pushes its negation. A malformed list — a connective with too few operands — is refused with **"Domain() malformed domain "** followed by the list. An item that is neither a triple nor a connective is refused with **"Domain() invalid item in domain: "** followed by the item.

### 20.2 The condition

A condition is a triple: a **field expression**, an **operator** and a **value**.

The field expression is one of:

| Form | Meaning |
|---|---|
| `field` | A field of the entity being searched. |
| `field.sub` … | A path: every segment but the last is relational and is traversed. Equivalent to nesting the `any` operator. |
| `field.property` | For a custom-properties field, one property of it. |
| `field.granularity` | For a date or instant, a derived number such as the year, the quarter, the month, the week-of-year, the day-of-month, the day-of-week or the hour. |
| `id` | The record identifier. |

### 20.3 The operator table

Operators divide into **standard** operators, which every layer supports, and **convenience** operators, which are rewritten into standard ones before compilation.

**Standard operators**

| Operator | Applies to | Exact semantics |
|---|---|---|
| `in` | Any field | The field's value is one of the values in the collection. An entry that is the empty value means *the field is not set*. For a relational field, an integer entry compares identifiers and **bypasses the target's record rules**; a text entry is matched against the target's display name. |
| `not in` | Any field | The negation of `in`, with the empty-value handling of [20.7](#207-empty-values-and-the-null-question). |
| `<`, `>`, `<=`, `>=` | Ordered types | Ordinary comparison against a single value, with the empty-value handling of [20.7](#207-empty-values-and-the-null-question). |
| `like` | Text | The value, **surrounded by wildcards**, matched case-sensitively. The field is cast to text first when it is not a text type. |
| `not like` | Text | The negation, which also matches rows whose value is not set. |
| `ilike` | Text | The same as `like` but case-insensitive **and accent-insensitive** where the tenant's database offers accent folding. Both the column and the pattern are folded. |
| `not ilike` | Text | The negation, which also matches rows whose value is not set. |
| `=like` | Text | The value matched **without** added wildcards, case-sensitively. The pattern's own wildcards still apply. |
| `not =like` | Text | The negation, which also matches unset rows. |
| `=ilike` | Text | The anchored form, case-insensitive and accent-insensitive. |
| `not =ilike` | Text | The negation, which also matches unset rows. |
| `any` | Relational fields and the identifier | The field leads to at least one record matching the given sub-filter. For a many-to-one and for the identifier, the sub-filter is evaluated on the target entity **with the archive filter switched off**; for a to-many field it is evaluated on the target entity with the field's own client-side context applied. The target's record rules **are** applied. |
| `not any` | The same | No record the field leads to matches the sub-filter. |

Two further standard operators exist for internal use and may not appear in a filter received from outside; supplying one is refused with **"Domain() invalid item in domain: "** followed by the item.

| Operator | Semantics |
|---|---|
| `any!` | Like `any`, but the target's record rules are **not** applied. |
| `not any!` | Like `not any`, without the target's record rules. |

**Convenience operators**, each rewritten before compilation:

| Operator | Rewritten to |
|---|---|
| `=` | `in` against a one-element collection. Against a collection, `in` against that collection; against an **empty** collection, `in` against the collection containing only the empty value, because a screen writes "is not set" that way. |
| `!=` | `not in`, by the same rules. |
| `=?` | The whole condition becomes "everything" when the value is empty; otherwise `=`. Used in stored filters where a blank parameter should not restrict. |
| `child_of` | See [20.5](#205-the-hierarchy-operators). |
| `parent_of` | See [20.5](#205-the-hierarchy-operators). |

Any other operator is refused when the condition is checked, with a message naming the operator and the condition.

### 20.4 Normalisation before compilation

A filter is normalised in four increasing levels, applied in order and repeated until stable, with a bound of one thousand iterations.

| Level | What happens |
|---|---|
| None | The filter as parsed. |
| Basic | Convenience operators are rewritten. Equality becomes membership. Values are coerced to collections. Conditions on a field type are normalised per type. Constants are folded: a conjunction containing "nothing" becomes "nothing"; a disjunction containing "everything" becomes "everything"; duplicate conditions are merged. Negations are pushed down by inverting operators. |
| Dynamic values | Relative values are resolved: a text value naming a relative date on a date field becomes an absolute date; the same for instants. This level is separated because its result depends on *when* the filter is evaluated, so a filter normalised at this level cannot be cached across time. |
| Full | The hierarchy operators are expanded, which requires searching. Access-bypassing forms are chosen where the environment or the field allows. A membership test against a many-to-one whose sub-filter is only about identifiers is collapsed into a direct comparison, avoiding a sub-query. |

Normalisation rules worth stating explicitly because they change results:

1. A membership test against an **empty** collection is "nothing"; a non-membership test against an empty collection is "everything".
2. A membership test whose collection contains only values that cannot occur — for instance the empty value on a field the registry knows to be not-null — has those values removed; if nothing remains, the rules above apply.
3. A pattern operator against an **empty** value is folded: for a relational field or an anchored operator it becomes an emptiness test; otherwise it becomes the constant matching the operator's polarity.
4. A pattern operator against a non-text value converts the value to text, except for the anchored forms, which refuse with **"The pattern to match must be a string"**.
5. A condition on a **relational** field compared with text is rewritten to a condition on the target's display name, wrapped in `any`. Comparing a relational field with text using an inequality operator is refused with **"Inequality not supported for relational field using a string"**.
6. A condition on a **boolean** field whose value is not a collection is refused; whose collection contains non-boolean values is normalised by truthiness.
7. Conditions in a conjunction or a disjunction are sorted into a canonical order before merging, so that equivalent filters normalise to the same form and duplicate conditions collapse.

### 20.5 The hierarchy operators

Two operators walk a parent-child relation.

**Semantics.** The field expression is either the identifier, meaning "use the entity's declared parent field", or a relational field. When the field's target entity is the same as the searched entity, the walk happens on that entity; otherwise the condition is equivalent to traversing the field and applying the hierarchy operator to the target's identifier.

The value identifies the starting records: integers are identifiers; text values are matched against display names with case-insensitive containment; a mixture is allowed. For a many-to-many field, identifiers are also resolved by search rather than used directly.

| Operator | Result |
|---|---|
| `child_of` | The starting records **and** all their descendants. |
| `parent_of` | The starting records **and** all their ancestors. |

**Resolution with a materialised hierarchy.** When the entity stores a path field and the walk uses the declared parent field:

- descendants: the disjunction, over the starting records, of the condition that the path begins with that record's path;
- ancestors: the set of identifiers appearing in the starting records' paths, excluding the final segment which is the record itself.

**Resolution without one.** Iteratively, elevated and with the archive filter switched off:

- descendants: repeatedly search for records whose parent is in the current frontier, adding the new ones, until nothing new appears;
- ancestors: repeatedly follow the parent link, adding the new ones, until nothing new appears.

In both cases the elevation is deliberate: the hierarchy must be walked completely, and the filtering of records the user may not see is left to the record rules applied to the search as a whole.

A hierarchy operator against an empty value yields "nothing". A hierarchy operator on a non-relational field is refused with a message stating that it works only for relational fields.

### 20.6 Compilation of each operator

The condition is compiled against the field's column. Let *column* be the field's expression in the query — the plain column, or, for a company-dependent field, the current company's entry in the mapping with the fallback applied, or, for a translatable field, the current language's entry with the source-language fallback.

| Operator | Compiled form |
|---|---|
| `in` with no empty value among the values | *column* is in the list of converted values |
| `in` with an empty value among the values | (*column* is in the list of the other values) **or** *column* is unset — and, if the registry knows the column cannot be unset, just the first part, or "nothing" if the list is otherwise empty |
| `not in` with no empty value | (*column* is not in the list) **or** *column* is unset — because an unset column satisfies "not one of these" — and, if the column cannot be unset, just the first part |
| `not in` with only the empty value | *column* is set — or "everything" if the column cannot be unset |
| `<`, `>`, `<=`, `>=` | *column* compared to the converted value; **and**, when the field has a declared empty value that would itself satisfy the comparison and the column can be unset, *or column is unset* |
| `like`, `ilike` | *column*, cast to text if it is not text, compared with a pattern consisting of a wildcard, the value, and a wildcard; for the case-insensitive form both sides are accent-folded |
| `=like`, `=ilike` | The same without the added wildcards |
| The four negative pattern operators | The negated comparison **or** *column* is unset, when the column can be unset |
| `any!`, `not any!` | *column* is, or is not, in the sub-query produced by compiling the sub-filter on the target entity |

For a **company-dependent** field with the standard-excluding-empty-values index, the compiled condition is additionally prefixed with a test that the raw mapping is not empty, whenever the fallback value does not itself satisfy the condition. This preserves the result while letting the partial index be used.

### 20.7 Empty values and the null question

This is the subtlest part of the grammar and the most common source of divergence in a rebuild.

**The rule.** In a filter, the empty value of the field's type means *the field is not set*. For a text field the declared empty value is the **empty string**, for an integer it is **zero**, for a decimal number and a monetary amount it is **zero**, and for a boolean it is **false**. Where such a declared empty value exists, the compilation treats the empty value and the declared empty value as the same thing:

1. A membership test that includes the empty value also includes the declared empty value in the compared list, and adds the unset test.
2. A membership test that includes the declared empty value is treated as including the empty value, and therefore also adds the unset test.
3. An inequality that the declared empty value would satisfy also matches unset rows.

**Worked examples.**

| Filter | On a nullable integer column | Matches |
|---|---|---|
| `[('count', '=', 0)]` | | Rows whose count is 0 **and** rows whose count is unset |
| `[('count', '!=', 0)]` | | Rows whose count is neither 0 nor unset |
| `[('count', '<', 1)]` | | Rows whose count is below 1 **and** rows whose count is unset, because 0 is below 1 |
| `[('count', '>', -1)]` | | Rows whose count is above −1 **and** rows whose count is unset, because 0 is above −1 |
| `[('name', '=', False)]` | On a nullable text column | Rows whose name is unset **and** rows whose name is the empty string |
| `[('name', 'not like', 'x')]` | | Rows whose name does not contain "x" **and** rows whose name is unset |
| `[('partner_id', '=', False)]` | On a many-to-one | Rows with no party — a many-to-one has no declared empty value, so only the unset test applies |

**When the column cannot be unset.** The registry records which columns really carry a not-null constraint. For those:

- an empty value in a membership list is removed before compilation;
- a membership test on nothing but the empty value becomes "nothing";
- a non-membership test on nothing but the empty value becomes "everything";
- the extra unset tests are omitted.

This is why the post-installation verification of not-null agreement matters: if the registry believes a column is not-null when it is not, filters will silently miss rows.

### 20.8 Traversal

A condition whose field expression is a path is equivalent to nested `any` conditions:

```
('order_id.partner_id.country_id', '=', 21)
    ≡  ('order_id', 'any', [('partner_id', 'any', [('country_id', '=', 21)])])
```

Rules:

1. Each traversal produces a sub-query on the target entity, to which the target's **record rules** are applied unless the field bypasses them or the environment is elevated.
2. Traversal switches the **archive filter off** on the target for a many-to-one; for a to-many it applies the field's own client-side context, which may itself switch the archive filter on or off.
3. A path through a **nullable** many-to-one additionally accepts unset rows when the condition being pushed down is one that an unset value would satisfy — that is, when the value tested is the empty value under a positive operator, or a non-empty value under a negative one. A negative condition against a non-empty value cannot be pushed down this way and is instead handled by evaluating the positive form and negating the whole thing.
4. Traversal depth is not bounded by the platform; a very deep path produces a correspondingly deep sub-query.

### 20.9 In-memory evaluation

A filter can also be evaluated against records already in memory, which is what a client-side dynamic filter and the company consistency check use. The semantics must match the compiled form:

1. Each condition is evaluated by reading the field on the record and applying the operator's semantics.
2. Empty values follow the same rules as [20.7](#207-empty-values-and-the-null-question).
3. Traversal reads the relation and evaluates the sub-filter on the results.
4. Hierarchy operators expand exactly as they do for compilation.
5. A custom node evaluates through its own rule.

A rebuild that implements the two evaluations independently will drift; deriving both from one normalised tree is strongly preferable.

### 20.10 Filters as values

Filters appear in stored records in three forms, and the difference matters:

| Form | Where | Evaluation |
|---|---|---|
| A literal filter | An action's fixed criteria, a record rule with no variables | Parsed and used directly |
| An expression producing a filter | A record rule, a relational field's candidate restriction | Evaluated in a restricted evaluation context providing the acting user, the current company, the current date and a resolver from external identifier to identifier; the result must be a filter |
| An expression evaluated by the client | A relational field's candidate restriction declared as text | Sent to the client as text and evaluated there against the record being edited, so the candidates can depend on unsaved values |

The third form is the only one that can depend on unsaved form state, and it is therefore the only one that is **not** enforced on the server. A candidate restriction expressed that way is guidance, not a constraint; a constraint must also exist as a declared validation.

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

## 22. Invariants a rebuild must preserve

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

---

## 23. Acceptance criteria

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

---

## Related documents

- [Architecture](architecture.md) — the registry, the environment, the record set and the unit of work.
- [Inheritance and extension](inheritance-and-extension.md) — how several packages contribute to one entity and one field.
- [The security model](security-model.md) — field restrictions, record rules, the unrestricted actor and company scope.
- [Views and actions](views-and-actions.md) — how fields are presented and how the search grammar reaches this one.
- [The package system](package-system.md) — when the schema synchronisation of this document runs.
- [Identity and values](../data/persistence-identity-and-values.md) — identifiers, precision, rounding, dates and sequences in one place.
- [The physical data catalogue](../data/physical-data-catalog.md) — the tables, columns, indexes and constraints a full installation creates.
