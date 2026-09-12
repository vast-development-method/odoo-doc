# Record operations and query notation

This document specifies the generic operations that every entity supports and the complete grammar of the filter notation used to select records. For each operation it gives the inputs, the checks performed, the records read and written, the ordering of the steps, the outputs and every error path. It then gives the full operator set of the filter notation with the semantics of each operator on each field type, the ordering, limiting and counting rules, the grouped-reading contract with its aggregate functions and date granularities, the naming operations, duplication, default-value retrieval, the export and load shapes, the catalogue of context keys, and the elevated-rights execution mode.

Read [the architecture](architecture.md) first for the record set, the environment and the unit of work; [the entity and field system](entity-and-field-system.md) for what a field is and how a computation is declared; and [the security model](security-model.md) for the gates whose position in each procedure is stated here.

---

## Table of contents

1. [Record sets](#1-record-sets)
2. [Create](#2-create)
3. [Read, fetch and the read format](#3-read-fetch-and-the-read-format)
4. [The filter notation](#4-the-filter-notation)
5. [Ordering, limiting, counting](#5-ordering-limiting-counting)
6. [Archived-record visibility](#6-archived-record-visibility)
7. [Search](#7-search)
8. [Write](#8-write)
9. [Delete](#9-delete)
10. [Grouped reading](#10-grouped-reading)
11. [Naming operations](#11-naming-operations)
12. [Duplication](#12-duplication)
13. [Retrieving default values](#13-retrieving-default-values)
14. [Export](#14-export)
15. [The generic load operation](#15-the-generic-load-operation)
16. [Context keys](#16-context-keys)
17. [Elevated-rights execution](#17-elevated-rights-execution)
18. [The permission model as the operations apply it](#18-the-permission-model-as-the-operations-apply-it)
19. [Worked examples](#19-worked-examples)
20. [Invariants a rebuild must preserve](#20-invariants-a-rebuild-must-preserve)
21. [Acceptance criteria](#21-acceptance-criteria)
22. [Reconciliation notes](#22-reconciliation-notes)

---

## 1. Record sets

Every operation is invoked on a record set. A record set carries three things: the environment it runs in, an ordered tuple of record identifiers, and a prefetch identifier sequence.

### 1.1 Identity and equality

- Two record sets are **equal** when they belong to the same entity and hold the same identifiers in the same order. The environment is not part of equality.
- A record set is **contained in** another when its identifier tuple is a sub-multiset of the other's, entity equality included.
- Order comparison between two record sets of the same entity compares their identifier sets: strictly contained is "less than", contained is "less than or equal".
- A record set counts as present exactly when it holds at least one identifier; an empty set counts as absent.
- Iterating a record set yields one single-record set per identifier, in order. When the set is larger than the prefetch limit of 1,000 records and its prefetch sequence is its own identifier tuple, iteration proceeds in chunks of 1,000 and each yielded record carries its chunk as its prefetch sequence.
- Reversed iteration yields the records in the opposite order, with the prefetch sequence reversed as well.

### 1.2 Set operations

| Operation | Result |
|---|---|
| Concatenation | The identifiers of the left set followed by the identifiers of the right set, duplicates kept, order preserved |
| Difference | The identifiers of the left set that are not in the right set, the left order preserved |
| Intersection | The identifiers of the left set that are also in the right set, the left order preserved, duplicates removed |
| Union | The identifiers of the left set followed by those of the right set that are not in the left, duplicates removed |

All four require both operands to belong to the same entity. The resulting record set carries the environment of the left operand.

### 1.3 In-place selection and mapping

| Operation | Contract |
|---|---|
| Selection by a rule | Keeps the records for which the rule holds, preserving order. The rule may be a predicate, a field name meaning the presence of a value in that field, a path meaning the presence of any value reached, or a filter expression, in which case it behaves as the next row. An empty rule returns the set unchanged. |
| Selection by a filter | Keeps the records satisfying the filter, evaluated **in memory** ([section 4.10](#410-in-memory-evaluation)), preserving order. |
| Mapping | Applies a rule to every record. With a path of field names, the path is walked one field at a time for the whole set; the result is a **record set without duplicates** when the last field is relational, and a plain list otherwise, with duplicates kept and order preserved. When an intermediate set exceeds the prefetch limit of 1,000 records, the last field is explicitly fetched for the whole set first. With a predicate, the results are combined by union when they are record sets and listed otherwise. Applied to an empty set, the rule is invoked once on the empty set so that the target entity can be determined. |
| Grouping | Returns a map from key value to record set. The key may be a field name or a predicate. All the resulting record sets share the original prefetch sequence. Input order does not matter and the operation is not lazy. |
| Sorting | Returns the records ordered by a key ([section 5.3](#53-in-memory-sorting)). |
| Requiring exactly one | Returns the set when it holds exactly one record; otherwise refuses with **"Expected singleton: "** followed by the record set. |

### 1.4 Prefetching

Every record set carries a **prefetch identifier sequence**. When a field is read on one record and the value is not in the cache, the platform reads that field, and every field of the same prefetch group, for up to 1,000 records taken from the prefetch sequence that also lack the value. Consequences:

- A search returns a record set whose prefetch sequence is its own identifier tuple, therefore iterating the result and reading a field issues one statement, not one per record.
- Reading a many-to-one returns a record set whose prefetch sequence is the set of values of that same field over the owner's prefetch sequence, therefore reading a field on the linked record of every owner issues one statement.
- Reading a to-many field returns a record set whose prefetch sequence is the concatenation of that field's values over the owner's prefetch sequence.
- The prefetch sequence can be replaced explicitly; replacing it with nothing resets it to the set's own identifiers.
- Prefetching is disabled entirely for a record set whose context sets `prefetch_fields`, the prefetch-fields key, to false, and for fields whose prefetch group is declared false.

### 1.5 In-memory records

A record set may hold **in-memory identifiers** instead of database identifiers. An in-memory identifier:

- always counts as absent, therefore the identifier of such a record reads as empty;
- may carry an **origin**, the database identifier of the record it stands for;
- may carry a **reference**, an arbitrary label used to recognise it among other in-memory records;
- compares equal to another in-memory identifier when both have the same origin, or when both have the same reference.

Building an in-memory record takes a value map, an optional origin and an optional reference, and puts the values into the cache without validation. Asking for the origins of a set returns the record set of the origins, dropping in-memory records that have none.

Mixing in-memory and database identifiers in one record set is accepted for reading, but every write path asserts that a set is either entirely in memory or entirely stored:

> "\<the record set\> contains a mix of real and new records. It is not supported."

An existence test reports in-memory records as existing, by convention.

---

## 2. Create

### 2.1 Contract

**Inputs.** An ordered list of value maps. A single map is accepted and treated as a one-element list, in which case a one-record set is returned. An empty list returns an empty record set without any check.

**Output.** A record set of the created records, in the order of the input list.

### 2.2 The procedure

1. **Entity permission.** The create right is checked on the entity.
2. **Field permission.** The union of the keys of every value map, plus every field named by a default-value context key, is checked for write permission field by field. A key that is not a field is refused with **"Invalid field '"** the name **"' in '"** the transport name **"'"**.
3. **Completion.** Each value map is completed ([section 2.3](#23-completing-one-value-map)): defaults are added, forbidden keys are removed, audit fields are set, precomputed fields are evaluated.
4. **Classification.** For each completed map the keys are classified as: *stored*, when the field has storage; *embedded*, when the field belongs to an embedded parent, in which case the value is routed to the parent entity; *inversible*, when the field declares an inverse rule and was not precomputed; and *protected*, being every field of the co-computed group of any assigned computed field that is either editable or precomputed. A many-to-one marked as bypassing the target's read permission is additionally checked: when the environment is restricted, the target record's read permission is verified now.
5. **Parents.** For each embedding declared by the entity, records whose link field has no value get a parent created from the embedded subset of their values, in one batch per parent entity; records whose link field already has a value get the embedded subset written onto the existing parent.
6. **Insertion.** The rows are inserted ([section 2.4](#24-insertion)).
7. **Inverse rules.** With the protected fields protected, each inverse group is invoked once on the records that carry at least one of its fields, after writing the group's non-stored values into their cache. Non-stored to-many members of the group are invalidated afterwards.
8. **Validation of inversely assigned fields.** Validations watching an inversely assigned field run, excluding those that watch a stored field, which already ran inside step 6.
9. **Company consistency.** When the entity enables the automatic check, it runs on the created records ([multi-company, section 6](multi-company.md#6-the-company-consistency-check)).
10. **External identifiers.** When the creation happens inside a data load, an external-identifier key present in the original value map is used to create or update an external identifier for the record, prefixed with the loading package's technical name when it contains no full stop.

### 2.3 Completing one value map

1. Build the forbidden set: the identifier and the materialised ancestor path. When audit logging is enabled, and unless the acting user is the root identity while the registry is still loading, add the four audit fields — creation instant, creating user, last update instant and last updating user. Add every field that is precomputed and read-only.
2. Add the missing defaults: collect every field of the entity absent from the values, skipping fields embedded from a parent entity whose link field **is** present in the values, directly or through an ancestor; resolve their defaults by the order of [the entity and field system, section 13.1](entity-and-field-system.md#131-the-resolution-order); normalise relational defaults into command form; then overlay the caller's values on top. Caller values always win.
3. Remove every key of the forbidden set.
4. When audit logging is enabled, set the creation instant and the last update instant to the transaction timestamp and the creating and last updating user to the acting user, in each case only where the value is absent.
5. For every custom-properties field, apply the container's property defaults.
6. **Precomputation.** For every field declared precomputed that is still absent, build an in-memory record from the map, read the field on it — with elevated rights when the field is computed with elevated rights — convert the value to write format, store it in the map, and record the field as precomputed.

### 2.4 Insertion

1. The completed maps are processed in batches of 100.
2. For one batch, the union of stored field names is taken and sorted. Fields with a column contribute a column and a value per row; a row that has no value for a column contributes the database default marker. Fields without a column — attachment-backed binary content, to-many fields, custom-properties fields — are collected separately. A custom-properties field is collected separately **even though it has a column**, because inserting it may also have to update the container's definition.
3. When no column at all is contributed, the insertion lists only the identifier with the database default marker, which creates an empty row.
4. One insertion per batch returns the generated identifiers in order.
5. The cache is primed: every stored field that is not part of the insertion is set to empty for the new records — to-many fields to the empty set, the rest to unset; every inserted value is converted to cache format and stored; many-to-one and identifier-only polymorphic values additionally update the cached inverse lists of the linked records.
6. The materialised ancestor path is computed for the new records when the entity stores one.
7. With the protected fields protected: every field of the entity is marked as modified in **creation mode**, which schedules the dependent computations while skipping the inverse traversal of many-to-one edges, since on creation nothing else can already reference the new record; then the fields without a column are written in declaration order, each receiving the pairs of record and value of the records that supplied a value; then those fields are marked as modified in creation mode too.
8. Validations watching any stored field that was supplied run.
9. The create right is checked **again**, now against the created records, which applies record rules that may depend on the values just written.

### 2.5 Errors

| Situation | Result |
|---|---|
| The acting user may not create records of the entity | The entity-gate refusal naming the entity and the operation |
| The acting user may not write one of the supplied fields | The field-gate refusal of [the security model, section 7.5](security-model.md#75-the-refusal-message) |
| A supplied key is not a field | "Invalid field '\<name\>' in '\<transport name\>'" |
| A selection value is not in the list | "Wrong value for \<transport name\>.\<field\>: \<value\>" |
| A required field ends up empty | The not-null violation, reported as in [the entity and field system, section 15.3](entity-and-field-system.md#153-database-constraints) |
| A declared database constraint is violated | The constraint's declared message, wrapped as "The operation cannot be completed: " followed by the message |
| A validation refuses | The validation's own message |
| The created record violates a record rule | The record-gate refusal for the create operation |
| A hierarchy cycle would be created | "Recursion Detected." |

---

## 3. Read, fetch and the read format

Three operations exist, at three levels: fetching puts values in the cache, reading produces the transport format, and the combined forms do both from one query.

### 3.1 Fetching field values

Fetching ensures the named fields are in the cache for the records of the set, reading from the database what is missing.

1. In-memory records are dropped from the set first.
2. The **fields to fetch** are determined ([section 3.2](#32-determining-the-fields-to-fetch)). Naming an unknown field is refused with **"Invalid field '"** the name **"' on '"** the transport name **"'"**. Naming a field the acting user may not read raises the field-gate refusal.
3. When at least one field to fetch has a column, a query restricted to the identifiers of the set is built **with archived-record visibility turned off** and with the record rules of the read operation applied. Otherwise the read right is checked directly and the query is built from the record set itself.
4. The selected columns are read in one statement and stored in the cache **without overwriting values already present**, which preserves pending writes.
5. Fields without a column read themselves: attachment-backed binary content issues one search over attachments; a one-to-many field issues one search over the target entity; a many-to-many field issues one join over the association table.
6. When the query returned fewer records than the set holds, the missing ones are tested for existence; those that exist but were filtered out by a record rule raise the read refusal naming them. Records that simply no longer exist do **not** raise.

### 3.2 Determining the fields to fetch

- When no field list is given, the fields to fetch are every field whose prefetch group is exactly the default group and which the acting user may read.
- When a list is given it is processed as a work list. Each named field is checked for read permission. A **stored** field is added to the fetch list. A **non-stored** field is replaced by the first segment of each of its dependency paths, and those fields are queued, provided they are either non-stored themselves or in the default prefetch group and readable; this makes reading a non-stored computed field pull its stored inputs in one statement. The identifier is always skipped. A field already in the cache for every record of the set is skipped when the caller asked for the "ignore cached" behaviour, which plain fetching does and the combined search-and-fetch does not.

### 3.3 Reading

1. When no field list is given, it is the list of every readable field.
2. When the set is empty and the environment is restricted, the field list is still validated for read permission.
3. The origins of the set are fetched.
4. The read format is produced ([section 3.4](#34-the-read-format)).

The result is one map per record **that still exists**, each carrying the identifier plus the requested fields. Records that disappeared between the fetch and the formatting are dropped silently.

### 3.4 The read format

| Type | Read format |
|---|---|
| Boolean | True or false |
| Integer | A whole number; a value above 2,147,483,647 is returned as a decimal |
| Decimal number, monetary amount | A decimal |
| Short text, long text | Text, or false when empty |
| Markup | Markup-safe text, or false when empty |
| Date, date and time | The canonical text form, or false when empty |
| Selection | The stored value, or false |
| Polymorphic reference | The transport name, a comma and the identifier, or false |
| Identifier-only polymorphic link | A whole number |
| Many-to-one | The pair of identifier and display name in the default load mode; the bare identifier when the load mode is "without display names". The display name is produced with elevated rights. When the referenced row no longer exists, the value is false. |
| One-to-many, many-to-many | The ordered list of identifiers |
| Binary content | Base-64 text, or the size as text when the size-only context key is set |
| Structured document | The document, deep-copied |
| Custom properties | The merged list of property definitions, each carrying its value and, for link properties, the display names |

Custom-properties fields are converted for the **whole batch at once**, so that existence checks and display names of linked records are resolved in one pass.

### 3.5 Searching and fetching in one query

The combined search-and-fetch runs a search and fetches the named fields in the smallest number of statements: the query produced by the search is reused as the source of the column read, therefore no intermediate identifier list is materialised. When the query is provably empty, no statement is issued at all, but the field list is still validated for read permission when the environment is restricted.

### 3.6 Searching and reading in one query

The combined search-and-read is the previous operation followed by the read format. Additionally, the archived-record visibility key is removed from the environment's context before formatting, because it was meant for the top-level search only and must not leak into the searches performed by to-many fields or by computed fields during formatting.

---

## 4. The filter notation

A **filter** is the value passed to every search, every record rule, every relational field's candidate restriction and every action's fixed criteria. It is a first-order logical expression over conditions on fields. [The entity and field system, section 20](entity-and-field-system.md#20-the-filter-grammar) states where filters appear and how they are stored; the grammar itself is here.

### 4.1 Shape

A filter is written either as an abstract expression — a constant, a condition, a conjunction, a disjunction or a negation — or as a **flat list in prefix notation**, in which a condition is a triple and the three logical connectives are single-character tokens.

| Token | Arity | Meaning |
|---|---|---|
| `&` | Binary | Conjunction of the next two items |
| `\|` | Binary | Disjunction of the next two items |
| `!` | Unary | Negation of the next item |

Items that are adjacent with no connective between them are conjoined. An empty list is the constant true, meaning every record.

| Filter | Meaning |
|---|---|
| `[]` | Every record |
| `[('state', '=', 'posted')]` | Posted records |
| `[('state', '=', 'posted'), ('amount', '>', 100)]` | Posted **and** over one hundred |
| `['\|', ('state', '=', 'draft'), ('state', '=', 'posted')]` | Draft **or** posted |
| `['&', '!', ('state', '=', 'cancel'), '\|', ('a', '=', 1), ('b', '=', 2)]` | Not cancelled, and either the first field is 1 or the second is 2 |

The two historic constant conditions — a condition whose field expression and value are both the number one with the equality operator, and the same with a field expression of zero — are recognised as the constants true and false respectively.

The **tree notation** is the internal form: a node is either the constant true, the constant false, a condition, a conjunction of nodes, a disjunction of nodes, a negation of a node, or a custom node carrying its own compilation rule. Filters are immutable; combining them produces new ones.

Parsing the list notation into the tree is a right-to-left stack walk: conditions push, the two binary connectives pop two nodes and push the combination, and the negation pops one and pushes its negation. A malformed list — a connective with too few operands — is refused with **"Domain() malformed domain "** followed by the list. An item that is neither a triple nor a connective is refused with **"Domain() invalid item in domain: "** followed by the item.

In the prose of this specification a filter is also written in a readable infix form, for example "the state equals posted and the journal kind is one of sale or purchase".

### 4.2 Conditions

A condition is a triple of a **field expression**, an **operator** and a **value**.

| Field expression form | Meaning |
|---|---|
| A field name | A field of the entity being searched |
| A path of field names separated by full stops | Every segment but the last is relational and is traversed; the condition applies to the last field |
| A date field followed by a granularity | A numeric part of a date or date-and-time value |
| A custom-properties field followed by a property name | One property of that field |
| The identifier | The primary key of the entity |
| The display name | The derived display name, resolved through the entity's name-search fields |

An empty field expression is refused with **"Empty field name in condition "** followed by the condition. A field expression naming an unknown field is refused with **"Invalid field "** the transport name, a full stop and the name **" in condition "** followed by the condition.

### 4.3 The operator set

**Standard operators.** These are the operators that reach the query builder and that a generated search rule must be able to serve.

| Operator | Reading | Value | Semantics |
|---|---|---|---|
| `in` | Membership | A set of values | True when the field's value is one of the values; an entry that is the empty value additionally matches a field that is not set |
| `not in` | Non-membership | A set of values | The negation of membership, with the empty-value handling of [section 4.6](#46-empty-values-and-the-three-valued-logic) |
| `<` | Less than | A single value | Strict inequality |
| `<=` | Less than or equal | A single value | Inequality |
| `>` | Greater than | A single value | Strict inequality |
| `>=` | Greater than or equal | A single value | Inequality |
| `like` | Contains the pattern, case-sensitive | Text | The pattern is surrounded with wildcards before matching |
| `not like` | The negation | Text | Also matches a field that is not set |
| `ilike` | Contains the pattern, case-insensitive and accent-insensitive | Text | The pattern is surrounded with wildcards; both sides are accent-folded where the database offers accent folding |
| `not ilike` | The negation | Text | Also matches a field that is not set |
| `=like` | Matches the pattern exactly, case-sensitive | Text | No wildcards are added; the pattern must match the whole value |
| `not =like` | The negation | Text | Also matches a field that is not set |
| `=ilike` | Matches the pattern exactly, case-insensitive and accent-insensitive | Text | As above, folded |
| `not =ilike` | The negation | Text | Also matches a field that is not set |
| `any` | Existence over a relation | A sub-filter, a query, or a text query expression | True when at least one record reached through the field satisfies the sub-filter |
| `not any` | Absence over a relation | The same | True when no record reached through the field satisfies the sub-filter |

Inside a pattern, the per-cent sign matches any sequence of characters, the underscore matches exactly one character, and the backslash escapes the next character.

**Convenience operators**, accepted in a filter and always rewritten into standard ones before compilation:

| Operator | Rewritten to |
|---|---|
| `=` | Membership against a one-element collection. Against a collection, membership of that collection; against an **empty** collection, membership of the collection containing only the empty value, because a screen writes "is not set" that way |
| `!=` | Non-membership, by the same rules |
| `=?` | The constant true when the value is empty, otherwise equality. Used in stored filters where a blank parameter should not restrict |
| `child_of` | See [section 4.9](#49-hierarchy-operators) |
| `parent_of` | See [section 4.9](#49-hierarchy-operators) |

**Internal operators.** Two privileged variants exist, `any!` and `not any!`. They behave exactly like their ordinary counterparts except that the record rules of the target entity are **not** applied while resolving the sub-filter. They may be produced by the optimiser and by a generated search rule, but a filter supplied by a caller in list form may not contain them; supplying one is refused with "Domain() invalid item in domain: " followed by the item.

**Operator case.** Operator tokens are matched without regard to case; a token that is not already in lower case is lowered.

Any operator outside these three sets is refused when the condition is checked, with a message naming the operator and the condition.

### 4.4 Value normalisation

Before any evaluation:

1. An absent value is replaced by the empty value.
2. A record set used as a value is replaced by its identifier list, with the log warning "The domain condition " the condition " should not have a value which is a model".
3. An in-memory identifier used as a value is refused: the condition becomes membership of the empty set, or non-membership of it for a negative operator, with the log warning "Domains do not support in-memory identifiers, use the identifier list instead, for " followed by the condition.
4. A sub-filter or a query used with an operator other than the two existence operators or the two membership operators logs "The domain condition " the condition " should use the 'any' or 'not any' operator."
5. A membership value that is not a collection is wrapped into a one-element collection.
6. An **empty** membership collection short-circuits: membership of nothing is the constant false, non-membership of nothing is the constant true.
7. A membership value on a field that is required, not nullable in the database and has no declared empty value has every empty entry removed, provided the record set being filtered contains no in-memory records.
8. A pattern value that is not text is coerced to text, except for the anchored operators, where a non-text value is refused with **"The pattern to match must be a string in condition "** followed by the condition.
9. An **empty** pattern value is resolved immediately. Let the outcome be true exactly when the operator is negative and the operator is anchored, or when it is neither. For a relational field or an anchored operator the condition becomes "the field is set" when the outcome is true and "the field is not set" otherwise; for any other field the condition becomes the constant of that outcome. In particular a containment pattern against the empty string is the constant true, and its negation is the constant false, because every value contains the empty string.

### 4.5 Paths

A path of two or more segments is rewritten, before any other optimisation, into nested existence conditions: a condition on the path "first, second, third" becomes an existence condition on the first field whose sub-filter is an existence condition on the second field whose sub-filter is the original condition on the third field.

- The rewrite applies only when the first segment is a **relational** field. When it is not relational, the remainder of the expression is a *property* of that field — a date granularity or a custom-property name — and not a path.
- An **embedded** field is rewritten as an existence condition on the link field carrying the same condition, which resolves the condition on the parent entity's table. Because embedding implies bypassing the parent's read permission, this adds no record rules.
- A **related field that is not stored** is rewritten by its generated search rule ([the entity and field system, section 10.3](entity-and-field-system.md#103-derived-behaviour)), which adds tolerance for an unset link on a non-required relation.
- Traversal depth is not bounded by the platform; a very deep path produces a correspondingly deep sub-query.

### 4.6 Empty values and the three-valued logic

This is the subtlest part of the grammar and the most common source of divergence in a rebuild. The notation is two-valued from the caller's point of view: every record either matches or does not. The mapping onto a database that has a third, unknown state is defined as follows.

A field **can be empty** when its column is not covered by a not-null constraint. The **declared empty value** of a type is: zero for an integer, zero for a decimal number and a monetary amount, the empty string for the text types, false for a boolean, and none for the other types. Where a declared empty value exists, the compilation treats it and "not set" as the same thing.

| Condition | Compiled as |
|---|---|
| Membership of a set containing the empty value or the declared empty value | Membership of the set without the empty markers, **or** the field is not set; when the field cannot be empty and the remaining set is itself empty, the constant false |
| Membership of a set containing no empty marker | Membership of the set |
| Non-membership of a set containing no empty marker | Non-membership of the set **or** the field is not set; when the field cannot be empty, non-membership alone |
| Non-membership of a set containing only empty markers | The field is set; when the field cannot be empty, the constant true |
| A positive pattern operator | The pattern comparison alone; a field that is not set never matches |
| A negative pattern operator | The negated pattern comparison **or** the field is not set, when the field can be empty |
| An inequality on a type **with** a declared empty value | The comparison, plus "or the field is not set" exactly when the declared empty value itself satisfies the comparison and the field can be empty |
| An inequality on a type **without** a declared empty value | The comparison alone; a field that is not set never matches |

The negation of an inequality is therefore not simply the opposite inequality: negating "greater than a value" on a type without a declared empty value yields "less than or equal to the value, or the field is not set", because a record with no value satisfies neither on its own.

Pattern comparison always casts the field to text first, unless the field is already a text type.

**Worked examples on a nullable column.**

| Filter | Matches |
|---|---|
| The count equals zero | Records whose count is zero **and** records whose count is not set |
| The count differs from zero | Records whose count is neither zero nor unset |
| The count is below one | Records whose count is below one **and** records whose count is not set, because zero is below one |
| The count is above minus one | Records whose count is above minus one **and** records whose count is not set, because zero is above minus one |
| The name equals the empty value | Records whose name is not set **and** records whose name is the empty string |
| The name does not contain the letter x | Records whose name does not contain it **and** records whose name is not set |
| The link equals the empty value, on a many-to-one | Records with no link — a many-to-one has no declared empty value, so only the unset test applies |

**When the column cannot be unset.** The registry records which columns really carry a not-null constraint. For those, an empty value in a membership list is removed before compilation; a membership test on nothing but the empty value becomes the constant false; a non-membership test on nothing but the empty value becomes the constant true; and the extra unset tests are omitted. This is why the post-installation verification that the registry and the database agree about not-null matters: if the registry believes a column is not-null when it is not, filters will silently miss records.

### 4.7 Semantics per field type

| Field type | Membership | Pattern operators | Inequalities | Existence operators |
|---|---|---|---|---|
| Boolean | Values are coerced to booleans; text is parsed, with "true", "1", "yes", "y", "t" and "on" reading as true; a single-element set holding only false is inverted into non-membership of the set holding true; the set holding both collapses to the constant true, and only at the outermost optimisation stage | Compiled by casting to text | Compiled by casting to text | Not applicable |
| Integer | The value is coerced to a whole number; zero counts as empty | Cast to text | Plain comparison; an unset field matches when zero satisfies the comparison | Not applicable |
| Decimal number, monetary amount | The value is coerced to a decimal; zero counts as empty | Cast to text | As integer, with zero as the declared empty value | Not applicable |
| Short text, long text, markup | Exact comparison; the empty string counts as empty | Full pattern support; accent folding applied to the case-insensitive variants where the database offers it | Lexicographic comparison; the empty string counts as empty | Not applicable |
| Translated text | As text, but evaluated against the language fallback chain; a three-letter-sequence pre-filter over all languages is added when the index exists | As text | As text | Not applicable |
| Date | Text values are parsed; a value that parses as a relative expression is resolved at the dynamic-values stage; comparison with an empty value under an inequality is the constant false | Cast to text | Plain comparison | Not applicable |
| Date and time | As date, then: a strict lower bound is turned into a non-strict bound on the next unit; a non-strict upper bound is turned into a strict bound on the next unit; the unit is one day when the value was written as a date and one second when it was written as a date and time. An equality against a date and time becomes the half-open interval from the value truncated to the second to one second later; against a plain date it becomes the half-open interval of one day in the acting user's time zone. An overflow when adding the unit resolves to the constant false for a strict lower bound and to "the field is set" for a non-strict upper bound. | Cast to text | As above | Not applicable |
| Selection | Exact comparison on the stored value | Pattern over the stored value | Lexicographic | Not applicable |
| Polymorphic reference | Exact comparison on the transport name, a comma and the identifier | Pattern over that text | Lexicographic | Not applicable |
| Identifier-only polymorphic link | As integer | As integer | As integer | Not applicable |
| Binary content stored in the table | Exact comparison on the bytes | Refused with "Cannot use like operators with binary fields" | Supported but meaningless | Not applicable |
| Binary content stored as an attachment | Only membership and non-membership of the set holding the empty value are supported, compiled as the absence or presence of an attachment row. Any other operator is refused, logged with its call stack, and the condition is replaced by the constant true | Refused | Refused | Not applicable |
| Many-to-one | A text value is rewritten as an existence condition on the target's display name; whole numbers compare against the stored identifier and **bypass the target's record rules** | Rewritten as an existence condition on the target's display name | Refused when the value is text, with "Inequality not supported for relational field using a string"; otherwise compares identifiers | The sub-filter is resolved on the target entity with archived-record visibility turned off |
| One-to-many, many-to-many | Rewritten into an existence condition with an identifier condition; a text value is rewritten as an existence condition on the display name; membership of the set holding the empty value means "has no linked record" | Rewritten as an existence condition on the display name | Refused when the value is text | The sub-filter is resolved on the target entity with the field's declared context applied and the field's declared filter conjoined |
| Structured document | Not supported | Not supported | Not supported | Not applicable |
| Custom properties | Conditions address one property by naming it after the field; date and date-and-time properties have their values serialised as text before comparison | Supported on text properties | Supported | Not applicable |

Mixed membership values on a relational field are split: the text entries become an existence branch on the display name and the remaining entries stay as an identifier condition; the two branches are combined by disjunction for a positive operator and by conjunction for a negative one.

### 4.8 Resolution of an existence condition

For an existence condition on a relational field:

1. The sub-filter is optimised against the **target entity**.
2. When it optimises to the constant false, the whole condition becomes the constant false; the negative form becomes the constant true.
3. When the field expression is the identifier itself, an existence condition is simply the sub-filter, and the negative form is its negation.
4. When the environment has elevated rights, or when the field declares that it bypasses the target's read permission, the operator is upgraded to its privileged variant.
5. For a many-to-one, the target entity is searched with archived-record visibility turned off. For a to-many field, the target entity is searched with the field's declared context applied and the field's declared filter conjoined to the sub-filter.
6. The result is either an identifier sub-query or, for the privileged variant with a sub-filter, a left join onto the target table with the sub-filter applied as an additional join condition. The join form is chosen for a positive condition, and for a negative condition only when the sub-filter's conditions are predominantly negative or when it constrains the identifier positively; otherwise a non-membership sub-query is used, which uses indexes better.
7. Empty handling: for a positive condition over a nullable link, the link must be set **and** the condition must hold; for a negative one, the link is not set **or** the condition holds.
8. For a one-to-many, the sub-query additionally excludes rows whose inverse field is empty, because a membership test against a set containing an unknown value is never true.
9. Two shortcuts avoid a sub-query on a many-to-one when the sub-filter is a single condition on the identifier and the privileged variant is in force:

| Condition | Rewritten to |
|---|---|
| The link privately satisfies "identifier in a set" | The link is in that set with the empty value removed |
| The link privately satisfies "identifier not in a set" | The link is not in that set with the empty value added |
| The link privately satisfies a nested privileged existence condition | The link privately satisfies the inner sub-filter |
| The link privately satisfies a nested privileged absence condition | The link is set **and** the link privately fails the inner sub-filter |
| The four negative forms | The negation of the corresponding row |

### 4.9 Hierarchy operators

The two hierarchy operators resolve a parent-child relation and are always rewritten into standard operators at the outermost optimisation stage.

| Field named by the expression | Target entity used to resolve the value | Hierarchy field |
|---|---|---|
| The identifier | The entity being searched | The entity's declared parent field |
| A many-to-one | Its target entity, archived-record visibility turned off | The target's declared parent field, or the named field itself when the target is the same entity |
| A one-to-many or many-to-many | Its target entity with the field's declared context | The target's declared parent field |
| Any other field | Refused with **"Cannot execute "** the operator **" for "** the transport name, a full stop and the field **", works only for relational fields"** |

**Resolving the starting set from the value.**

1. A single whole number or a single text value is wrapped into a one-element collection. Anything that is not a collection is refused with **"Value of type "** the type **" is not supported"**.
2. The collection is split into whole numbers and other values.
3. For a many-to-many field, the whole numbers are **also** resolved through a search rather than used directly, because the link is not a column.
4. Every non-numeric value contributes a case-insensitive containment condition on the display name; the disjunction of those conditions is searched on the target entity, ordered by identifier, and the results are appended to the starting set.
5. An empty starting set makes the whole condition the constant false; a value that is the empty value does so immediately.

**Resolving the hierarchy**, with elevated rights and archived-record visibility turned off:

- **Descendants.** Every record in the sub-tree rooted at each starting record, the starting records included. When the entity stores the materialised ancestor path and the hierarchy field is the declared parent field, the result is the disjunction, over the starting records' paths, of the condition that the path begins with that record's path; starting records that no longer exist are dropped first. Otherwise the platform walks level by level: accumulate the current level's identifiers, search the target entity for records whose hierarchy field is in the current level, subtract the identifiers already accumulated, and repeat until nothing new is found.
- **Ancestors.** Every ancestor of each starting record, the starting records included. With the materialised ancestor path, the result is the set of all identifiers appearing in the starting records' paths. Otherwise the platform walks upwards: accumulate the current identifiers, move to the hierarchy field of the current records, drop the ones already accumulated, and repeat.

In both cases the elevation is deliberate: the hierarchy must be walked completely, and the filtering of records the user may not see is left to the record rules applied to the search as a whole.

**Formatting the result.** When the expression named the identifier, the resolved filter replaces the condition directly. Otherwise the condition becomes a privileged existence condition on the field carrying the resolved filter when the resolution produced a filter, and a membership condition on the field against the resolved identifier set when it produced a set.

### 4.10 In-memory evaluation

A filter can also be evaluated against records already in memory, which is what a client-side dynamic filter, the in-place selection of [section 1.3](#13-in-place-selection-and-mapping) and the company consistency check use. The semantics must match the compiled form; a rebuild that implements the two evaluations independently will drift, so deriving both from one normalised tree is strongly preferable.

- The filter is optimised to the dynamic-values stage first, which resolves relative dates. Hierarchy operators are optimised all the way.
- A condition whose value is a query or a raw query expression is resolved by running a search restricted to the record set's identifiers, and the result is turned into an identifier membership test.
- The display name is evaluated through a wrapper that returns the empty string instead of raising when the acting user may not read the linked record.
- The identifier is evaluated against the record's **origin** for an in-memory record, which lets a form filter unsaved records against their stored counterpart.
- Only positive operators are implemented; a negative operator is evaluated as the logical negation of its positive counterpart.
- Pattern operators are compiled into a pattern matcher: the per-cent sign becomes "any sequence", the underscore becomes "any single character", the backslash escapes, and the anchored variants add start and end anchors while the unanchored variants do not, matching being performed from the start of the value so that the leading "any sequence" is what makes them unanchored. The case-insensitive variants lower-case and accent-fold both sides.
- Inequalities that raise a type mismatch evaluate to false rather than raising.
- On a relational field, an existence condition filters the linked records with the sub-filter and then tests membership. When the field declares that it bypasses the target's read permission, or when the privileged variant is used, the linked records are read with elevated rights and a marker is placed in the context; a nested non-privileged existence condition under that marker drops back to the acting user's rights and keeps only the records the user may read.
- When the record set is empty, every condition evaluates to false.
- Empty values follow [section 4.6](#46-empty-values-and-the-three-valued-logic); a custom node evaluates through its own rule.

### 4.11 Optimisation stages

A filter is optimised in four increasing stages. Each stage is applied repeatedly until a fixed point is reached, with a ceiling of one thousand rounds.

| Stage | What it does |
|---|---|
| None | The filter as written |
| Basic | Operator rewrites of the three convenience operators; value normalisation; path decomposition; pattern coercion; type coercion of date and date-and-time values written in the canonical form; boolean coercion; binary-field validation; merging of sibling conditions; constant folding, so that a conjunction containing the constant false becomes false and a disjunction containing the constant true becomes true; negations pushed down by inverting operators |
| Dynamic values | Resolution of relative date expressions against the current date and the acting user's time zone. This stage is separated because its result depends on *when* the filter is evaluated, so a filter optimised at this stage cannot be cached across time |
| Full | Embedded-field rewriting; invocation of generated search rules; hierarchy operators; privileged-variant upgrades; tautology removal on boolean fields; the many-to-one shortcuts of [section 4.8](#48-resolution-of-an-existence-condition) |

Only a fully optimised filter may be compiled into a query. A condition that still carries a non-standard operator at the end of the full stage is refused with **"Not standard operator left in condition "** followed by the condition.

For a **company-dependent** field whose index excludes empty entries, the compiled condition is additionally prefixed with a test that the stored mapping is not empty, whenever the fallback value does not itself satisfy the condition. This preserves the result while letting the partial index be used.

### 4.12 Merging sibling conditions

Inside a conjunction or a disjunction, conditions on the same field expression are merged. Conditions are first sorted into a canonical order, so that equivalent filters optimise to the same form and the merge is deterministic.

| Situation | Merge |
|---|---|
| Several membership conditions on the same field inside a conjunction | Intersection of the value sets |
| Several membership conditions on the same field inside a disjunction | Union of the value sets |
| Several non-membership conditions inside a conjunction | Union of the value sets |
| Several non-membership conditions inside a disjunction | Intersection of the value sets |
| Several existence conditions on the same relation inside a disjunction | One existence condition whose sub-filter is the disjunction of the sub-filters |
| Several absence conditions on the same relation inside a conjunction | One absence condition whose sub-filter is the disjunction of the sub-filters |
| Identical conditions | Deduplicated |

Merging membership conditions on a to-many relation inside a conjunction is **not** allowed to intersect the value sets, because "the lines include line one" and "the lines include line two" means the record has both lines, not that it has a line that is both.

---

## 5. Ordering, limiting, counting

### 5.1 The order specification

An order is a comma-separated list of terms. Each term names a field, optionally followed by a full stop and a property or by a colon and a granularity, then optionally by an ascending or descending marker, then optionally by a null-placement marker. Case is ignored for the direction and the null placement. A specification that does not match this shape is refused with:

> "Invalid \"order\" specified (`\<the specification\>`). A valid \"order\" specification is a comma-separated list of valid field names (optionally followed by asc/desc for the direction)"

Rules:

- When no order is given, the entity's default order applies. When the default order is empty, no ordering clause is produced and the result order is unspecified.
- Every field named must be readable by the acting user; the read permission is checked while the ordering term is produced.
- A boolean field is ordered on its value, with an empty value treated as false.
- A many-to-one field is ordered by **the target entity's default order**, not by the raw identifier — unless the target's default order is the identifier, or unless the term explicitly names the identifier of the target, in which case the raw column is used. The target table is joined with a left join so that records with no link are still returned. When the null placement is given, a leading term testing whether the field is set is added before the delegated order. The delegated order is reversed when the direction is descending.
- Ordering by a many-to-one is guarded against cycles: a field already traversed in the current ordering resolution contributes no term.
- Ordering by a to-many field is refused in memory with **"Invalid order on relational field "** followed by the term **" to sort"**.

### 5.2 Limit and offset

- The limit is the maximum number of records returned. A limit of false means no limit; a limit of true is treated as one.
- The offset is the number of matching records skipped before the first returned one.
- Applying an offset without an order gives an unspecified result; a caller that pages must supply an order that is total.

### 5.3 In-memory sorting

- With no key, the entity's default order is used.
- With a text key, it is parsed as an order specification and each term becomes a comparison key.
- With a predicate, the predicate's result is the comparison key.
- Null placement defaults to "nulls first" for a descending term and "nulls last" for an ascending term, which reproduces the database's default behaviour for the same direction.
- An empty value other than false on a boolean field is kept as it is; on every other type an empty value is mapped to the null marker and placed according to the null placement.
- A many-to-one term recursively builds the comparison key from the target's default order, guarded against cycles in the same way as [section 5.1](#51-the-order-specification).
- A record set of zero or one record is returned unchanged.

### 5.4 Counting

Counting returns the number of matching records. When a limit is given the count is capped at that number, which lets a client ask cheaply whether there are more than a given number of matches.

---

## 6. Archived-record visibility

An entity that declares an archive flag hides archived records from searches.

**The rule.** When the entity has an archive flag, **and** the caller did not disable the behaviour, **and** the context does not set the archived-visibility key to false, **and** the filter contains no condition whose field expression is the archive flag, then the condition "the archive flag is true" is conjoined to the filter.

Consequences:

- Mentioning the archive flag anywhere in the filter, at any nesting depth, suppresses the implicit condition entirely. Asking for records whose flag is false therefore returns only archived records, and asking for membership of both values returns both.
- The implicit condition is added **before** optimisation, which means the tautology removal of [section 4.7](#47-semantics-per-field-type) can remove it only at the outermost stage and never inside a sub-filter.
- Setting the archived-visibility key to false in the context suppresses the condition for every search performed under that context, including the searches issued while reading to-many fields.
- The values of a to-many field held in the cache always include archived linked records; they are filtered out when the cached value is turned into a record set, using the field's own declared context first and the environment's context second.
- Record-rule evaluation, company-consistency checking, hierarchy resolution and dependency inversion all run with archived-record visibility turned off, so that archiving a record never silently changes a permission decision or a computed value.

Two operations toggle the flag: archiving sets it to false on the records of the set that are currently active, and unarchiving sets it to true on the records that are currently archived. Both are plain writes, so every write-time behaviour — validation, tracking, recomputation — applies.

---

## 7. Search

1. When the environment is restricted and the caller did not ask to bypass access, the read right is checked on the entity.
2. The archived-record condition of [section 6](#6-archived-record-visibility) is conjoined when applicable.
3. The filter is optimised to the full stage against the entity. When it optimises to the constant false, an empty record set is returned with no statement issued.
4. A query is built over the entity's table, or over its stored query expression for an entity backed by one.
5. When access checking applies, the record rules for the read operation are computed, optimised with elevated rights and with archived-record visibility turned off, and conjoined to the query. When the rule filter optimises to the constant false, an empty record set is returned.
6. The order specification is compiled ([section 5.1](#51-the-order-specification)). The limit and the offset are applied.
7. The query is executed, after flushing every pending write on every field mentioned in the query ([caching](../runtime/caching.md)).

The returned record set is ordered as the query returned it and carries its own identifiers as its prefetch sequence.

A lower-level form of the same operation returns the **query** rather than the record set, without executing it, which is how a sub-filter becomes a sub-query.

---

## 8. Write

### 8.1 The procedure

1. An empty record set returns immediately.
2. The write right is checked on the entity and then the record rules are checked on the records.
3. Each key is checked for field-level write permission. A key that is not a field is refused with "Invalid field '\<name\>' in '\<transport name\>'".
4. Forbidden keys are removed: the identifier, the materialised ancestor path, and the four audit fields unless the acting user is the root identity while the registry is loading.
5. The last updating user and the last update instant are set to the acting user and the transaction timestamp unless already supplied.
6. The keys are classified. Fields with an inverse rule are grouped by rule; for a to-many field with an inverse rule the field's current value is read **before** the write, because the field is protected during the write and would otherwise be seen as empty. Fields whose modification can change which records depend on them are collected as relation-modifying. Fields that are inversible, or computed and editable, contribute their whole co-computed group to the **protected** set, except for non-stored to-many fields, which are deliberately not protected so that they are recomputed after the inverse rule ran.
7. Every protected field that is computed and **not** in the value map is recomputed first, so that the protection does not freeze a stale value.
8. With the protected set protected:
   1. the relation-modifying fields are marked as modified in **before mode**;
   2. the fields are written in ascending declared write order — rank 0 for ordinary fields, rank 10 for monetary amounts and the custom-property value field, rank 20 for the to-many fields, as [the entity and field system, section 5.2](entity-and-field-system.md#52-storage-attributes) sets out;
   3. every written field is marked as modified;
   4. when the entity stores the materialised ancestor path and the parent field was written, the parent field is flushed;
   5. validations watching a written non-inversible field run on the records that exist in the database;
   6. for each inverse group, any non-stored member whose cache entry was lost is written again, and the inverse rule runs once;
   7. validations watching a written inversible field run.
9. When the entity enables the automatic company check, it runs on the written fields.

### 8.2 Writing one field

1. The field is removed from the recomputation schedule for those records.
2. The value is converted to cache format.
3. Records whose cached value already equals the new cached value are **skipped**. For a monetary field the skip additionally requires the cached value to be correctly rounded for that record's currency.
4. The cache is updated and the records are marked dirty for that field, which enqueues the column update.

Relational fields override this: a many-to-one also removes the records from the old target's cached inverse list and adds them to the new target's; a to-many field executes the command list of [the entity and field system, section 7.5](entity-and-field-system.md#75-writing-to-a-to-many-field), which also specifies in what order the commands of one list take effect — which of them run at once and which accumulate until the end of the list — separately for a one-to-many and for a many-to-many.

### 8.3 Assignment through a record attribute

Assigning a field on a record set, rather than invoking the write operation, dispatches on the identifiers:

| Records | Behaviour |
|---|---|
| Currently protected | The value is written straight to the cache; no business behaviour, no recomputation |
| In memory | The co-computed group is protected, a relational field is marked modified in before mode, the value is written to the cache, and the field is marked modified; when the field is embedded, the same value is additionally assigned on the in-memory parent |
| Stored | The value is converted to write format and a full write is performed |

A mixed set is split into these three groups and each is handled accordingly.

### 8.4 Low-level column update

1. Records requiring a materialised-ancestor-path update are identified **before** the parent column changes.
2. The audit fields are set again on every row.
3. Rows are grouped by their exact set of updated columns.
4. Each group is written in batches of 100 rows with a single update that joins the table against an inline table of values.
5. Per-column expressions: a **translated** field's column is set to unset when the new value is unset, and otherwise to the existing document — or a fresh document holding the new value under the base language when the existing one is unset — merged with the new document; a **company-dependent** field's column is set to the existing document merged with the new document, then filtered to drop every entry equal to that company's fallback; every other column is assigned the value, cast to the column's type.
6. The materialised ancestor paths are rewritten for the identified records and their descendants.

---

## 9. Delete

### 9.1 The procedure

1. An empty record set returns immediately.
2. The delete right is checked on the entity and then the record rules are checked on the records.
3. Every declared deletion guard runs. A guard whose "applies during package removal" flag is false is skipped when a package is being removed.
4. Every field of the entity is removed from the recomputation schedule for these records, which avoids an endless loop when the computation of one field triggers the computation of a co-computed sibling.
5. The whole transaction is flushed.
6. With every field of the entity protected, the records are marked as modified in before mode over all their fields, which schedules the recomputation of everything that depended on them — for example the total of a parent after one of its lines is removed.
7. The identifiers are processed in batches sized to the database's parameter limit. For each batch: the rows are deleted; every external identifier pointing at a deleted record is collected; every attachment whose referenced entity and identifier match a deleted record is collected, with a direct statement because the attachment entity overrides searching; the company-dependent link handling of [the entity and field system, section 11.6](entity-and-field-system.md#116-company-dependent-many-to-one-and-referential-integrity) runs; and every user default value whose value is one of the deleted identifiers is deleted.
8. The **whole cache** is invalidated without flushing, because cascading deletions performed by the database are not visible to the platform.
9. The collected external identifiers and attachments are deleted.
10. An audit line is written to the deletion log naming the acting user, the entity and the identifiers.

### 9.2 Deletion behaviour of incoming links

| Declared policy on the incoming many-to-one | Effect |
|---|---|
| `restrict` | The deletion is refused by the database; the platform reports **"Another model is using the record you are trying to delete."** followed by the entity and the constraint, and the suggestion **"How about archiving the record instead?"**. [The entity and field system, section 15.3](entity-and-field-system.md#153-database-constraints) gives the message in full, with the two intervening lines and the rules that compose the entity and field placeholders |
| `cascade` | The referencing rows are deleted by the database |
| `set null` | The referencing columns are emptied by the database |

For many-to-many links, the association row is always removed by the cascade on the own column, and the target column carries the field's declared policy, which is `cascade` by default and `restrict` when declared.

For company-dependent links no foreign key exists, and the behaviour is implemented explicitly at step 7.

Deletion guards are the mechanism for business refusals such as "a confirmed order cannot be deleted". They are preferred over redefining the delete operation, because they can be switched off while the declaring package is removed, which keeps the database consistent.

---

## 10. Grouped reading

### 10.1 Contract

**Inputs.** A filter; an ordered list of group specifications; an ordered list of aggregate specifications; a having filter; an offset; a limit; an order.

- A **group specification** is a field name, or a field name and a granularity separated by a colon, or a many-to-one field and a field of the target separated by a full stop with an optional granularity, or a custom-properties field and a property name with an optional granularity.
- An **aggregate specification** is a field name and a function separated by a colon, or the special count token.
- The **having filter** is a filter whose field expressions are aggregate specifications rather than field names, restricted to the two membership operators, the four inequalities and the two equality operators. An unsupported operator is refused with **"Invalid having clause "** the item **": supported comparators are ('in', 'not in', '<', '>', '<=', '>=', '=', '!=')"**. An item that is neither a connective nor a triple is refused with **"Invalid having clause "** the item **": it should be a domain-like clause"**.

**Output.** A list of tuples. Each tuple holds the group values in the order of the group specifications, followed by the aggregate values in the order of the aggregate specifications.

With **no** group specification, exactly one tuple is returned, holding the aggregates over the whole filtered set; when the filter is provably empty, that single tuple holds the empty value of each specification ([section 10.5](#105-empty-values)) and no statement is issued. With at least one group specification and a provably empty filter, an empty list is returned.

The read right is checked on the entity before anything else, and every field named in a group or aggregate specification is checked for read permission while its query term is produced.

### 10.2 Aggregate functions

| Function | Meaning | Empty-group value |
|---|---|---|
| Count of rows | The number of rows in the group | Zero |
| Count of set values | The number of rows where the field is not empty | Zero |
| Count of distinct values | The number of distinct non-empty values | Zero |
| Sum | The sum of the values | False |
| Average | The arithmetic mean | False |
| Minimum | The smallest value | False |
| Maximum | The largest value | False |
| Conjunction | True when every value is true | False |
| Disjunction | True when at least one value is true | False |
| Value list | The ordered list of values, ordered by the record identifier | The empty list |
| Distinct value list | The ordered list of distinct values, ordered by the value | The empty list |
| Record set | The value list converted into a record set of the field's target entity, or of the entity itself when the field is the identifier. Allowed only on a relational field or on the identifier; otherwise refused with **"Aggregate method 'recordset' can be only used on relational field (or identifier) (for "** the specification **")."** | The empty record set |
| Sum in the company currency | The sum of a monetary field after converting each row to the currency of the root of the current company. Allowed only on a monetary field; otherwise refused with **"Aggregator \"sum_currency\" only works on currency field for "** followed by the field | False |

An unknown function is refused with **"Invalid aggregate method '"** the function **"' for '"** the specification **"'."** A specification without a function is refused with **"Aggregate method is mandatory for '"** the field **"'"**. A specification naming an unknown field is refused with **"Invalid field '"** the field **"' on model '"** the transport name **"' for '"** the specification **"'."** A path in an aggregate specification is refused with **"Invalid "** the specification **", this dot notation is not supported"**.

The currency-conversion aggregate joins, for each currency, the single most relevant rate: among the rates of that currency whose company is empty or is the root of the current company, the one with the greatest date on or before today, falling back to the smallest date after today. Rows whose currency has no rate are divided by one.

### 10.3 Group granularities

A granularity may be attached to a date field, a date-and-time field, or a date-typed custom property. Attaching one to any other type is refused with **"Granularity set on a no-datetime field or property: '"** the specification **"'"**. Omitting one on a date or date-and-time field is refused with **"Granularity not set on a date(time) field: '"** the specification **"'"**. An unknown granularity is refused with **"Granularity specification isn't correct: '"** the granularity **"'"**.

**Period granularities.** The group value is the start of the period, as a date for a date field and as a date and time for a date-and-time field.

| Granularity | Period |
|---|---|
| Hour | One hour |
| Day | One day |
| Week | Seven days, starting on the first day of the week of the acting user's language |
| Month | One calendar month |
| Quarter | Three calendar months |
| Year | One calendar year |

**Numeric granularities.** The group value is a number.

| Granularity | Value |
|---|---|
| Year number | The calendar year |
| Quarter number | The quarter, 1 to 4 |
| Month number | The month, 1 to 12 |
| Standard week number | The week number of the international standard |
| Day of year | The day within the year, 1 to 366 |
| Day of month | The day within the month, 1 to 31 |
| Day of week | The day within the week, as reported by the database, where zero is Sunday |
| Hour number | The hour, 0 to 23 |
| Minute number | The minute, 0 to 59 |
| Second number | The second, 0 to 59 |

**Time zone rule.** For a **date-and-time** field the value is first converted from coordinated universal time to the time zone named by the time-zone context key, when that key names a known zone. When the key is absent, no conversion is applied and the grouping is performed in coordinated universal time. When the key names an unknown zone, a warning is logged and no conversion is applied. For a **date** field no conversion is applied, because a date has no instant.

**Week rule.** The start of the week is taken from the acting user's language. Truncation to the week is performed by shifting the value back by the difference between seven and the first day of the week, taken as a remainder over seven days, truncating to the standard week, and shifting forward again by the same amount.

**Ordering rule for the day of week.** When the groups are ordered by that granularity, the ordering key is the sum of seven minus the first day of the week and the database's day number, taken as a remainder over seven, which makes Monday the first group when the language starts the week on Monday and Sunday the first group when it starts on Sunday.

### 10.4 Grouping by a relation

- Grouping by a many-to-one yields, as the group value, a one-record set of the target entity — or the empty record set for an unset link — with a prefetch sequence covering every group value, so that reading the display names of all groups issues one statement.
- Grouping by a many-to-one and a field of its target joins the target table with a left join and groups on the target field. When the acting user's record rules on the target entity are non-trivial, the joined table is the rule-filtered sub-query and the join alias is made user-specific, so that the query cache is not shared across users. Only a many-to-one may be traversed this way; any other relation is refused with **"Only many2one path is accepted for the "** the specification **" groupby spec"**.
- Grouping by a many-to-many joins the association table with a left join, restricted to the target records that satisfy the field's declared filter, and groups on the target column. A record linked to several targets therefore appears in several groups. A non-stored many-to-many is refused with **"Group by non-stored many2many field: '"** the specification **"'"**; a non-stored related many-to-many is first resolved to its stored target.
- Grouping by a one-to-many is not supported.
- Grouping by a boolean treats an empty value as false.

### 10.5 Empty values

| Specification | Empty value |
|---|---|
| Any of the three counting functions | Zero |
| Either value-list function | The empty list |
| A relational field or the identifier, with no function or with the record-set function | The empty record set of the target entity, or of the entity itself for the identifier |
| Anything else | False |

### 10.6 Ordering the groups

- With no explicit order, the groups are ordered by the group specifications in the order they were given.
- With an explicit order, each term must name either a group specification or a valid aggregate specification; otherwise it is refused with **"Order term '"** the term **"' is not a valid aggregate nor valid groupby"**.
- A term naming an aggregate orders by that aggregate.
- A term naming a many-to-one group **and** given explicitly orders by the target entity's default order, joining the target table, and the additional ordering columns are added to the grouping terms so that the query remains valid.
- A term naming a day-of-week group orders by the language-adjusted key of [section 10.3](#103-group-granularities).
- The direction defaults to ascending. Null placement is honoured when given.

### 10.7 Group expansion and temporal filling

Two post-processing behaviours exist on top of the raw grouped read. Both belong to the presentation layer and both change the returned rows.

**Group expansion.** When the first group specification's field declares a group-expansion rule, the rule is called with the groups that were actually returned and with the filter, and returns the full ordered list of groups to display. Groups the query did not return are added with every aggregate set to its empty value. The returned order is preserved, and any group returned by the query but absent from the expansion is appended at the end. When the field is relational and the target entity declares a fold flag, each group additionally carries the fold flag of its record.

**Temporal filling.** When the temporal-filling context key is set, either to true or to a map, and the first group specification carries a period granularity, the gaps between the returned periods are filled with empty groups. The map accepts three keys:

| Key | Meaning |
|---|---|
| Fill from | Inclusive lower bound of the range to fill, as a canonical date or date and time; when absent, the earliest returned group is the bound |
| Fill to | Inclusive upper bound; when absent, the latest returned group is the bound |
| Minimum groups | The minimum number of contiguous groups to return, counted from the lower bound when given and from the earliest returned group otherwise; it is not limited by the upper bound |

Groups outside the bounds are **kept**, not removed; the bounds only control where filling occurs. When neither bound is given and no group exists, nothing is returned. The filled groups use the same week offset as the grouping itself, so that the filled periods align with the real ones.

---

## 11. Naming operations

### 11.1 The display name

The display name of a record is specified in [the entity and field system, section 17.1](entity-and-field-system.md#171-the-display-name); the field that supplies it is determined as in section 17.2 of that document.

### 11.2 Searching by name

1. The condition "the display name matches the given text under the given operator" is conjoined with the caller's filter.
2. A combined search-and-fetch is run for the display name with the given limit, whose default is 100.
3. The result is the list of pairs of identifier and display name, the display names produced **with elevated rights**.

The default operator is the case-insensitive containment operator. The rewriting of a display-name condition into conditions on the entity's name-search fields is specified in [the entity and field system, section 17.3](entity-and-field-system.md#173-searching-by-display-name).

### 11.3 Creating from a name

Creating a record from a name is specified in [the entity and field system, section 17.5](entity-and-field-system.md#175-creating-from-a-name); the operation returns the pair of the new identifier and its display name.

---

## 12. Duplication

Duplication copies each record of the set and returns the new records in the same order.

### 12.1 Building the copy values

For each record, with archived-record visibility turned off:

1. A **blacklist** is built: the identifier, the four audit fields, the materialised ancestor path, and every link field of an embedding. When an override supplies a link field, every field of that parent entity that the child does not itself redeclare is blacklisted too, because the caller has given the parent explicitly. Otherwise the blacklist is extended recursively through the parent's own embeddings.
2. The **fields to copy** are the fields whose copy attribute is true, that are not overridden by the caller, and that are not blacklisted.
3. For each such field: a one-to-many field is copied by duplicating its linked records, whose copy values are computed recursively, **ordered by identifier**, and turned into creation commands, the links being re-parented automatically because the creation command sets the inverse; a many-to-many field is copied as a single replace command holding the identifiers of the linked records **the acting user may read**, which avoids failing the write on an inaccessible link; every other field is copied by converting its current value to write format.
4. A cycle guard records, per entity, the identifiers already visited; revisiting one yields no copy values for it, which terminates duplication of circular structures.

### 12.2 Creating the copies

The copy values are passed to the create operation, therefore every creation-time behaviour applies: defaults for fields that are not copied, precomputation, validation and company consistency.

### 12.3 Copying translations

After creation, translations are copied record by record:

- for every copied field that is translated, stored, not overridden by the caller and non-empty;
- for a whole-value translated field, every stored language entry except the language of the environment is written onto the copy, the environment's language already holding the value through the ordinary copy;
- for a term-by-term translated field, the term dictionary of the source is rebuilt from the stored document and applied to the copy;
- for a one-to-many field that was not overridden, the operation recurses on the pairs of source and copied lines matched **by ascending identifier**, which is why [section 12.1](#121-building-the-copy-values) sorts the lines by identifier;
- an embedded field whose link field was supplied by the caller is skipped, because the parent is not a copy;
- only installed languages plus the base language are considered;
- a cycle guard per entity prevents infinite recursion.

---

## 13. Retrieving default values

The default-values operation returns, for the named fields, the default that a creation would apply. The resolution order is specified in [the entity and field system, section 13.1](entity-and-field-system.md#131-the-resolution-order). Three behaviours belong to this operation itself:

1. Fields with no default at all are simply absent from the result; the caller must not read their absence as "empty".
2. Every returned value is normalised through the cache format and back to the write format, which turns relational command lists into the single replace form that a client understands.
3. Embedded fields are resolved by asking the parent entity for the corresponding parent field, and the parent's answers are merged into the result under the parent field names.

---

## 14. Export

An export takes a list of column paths, each path being a list of field names, and returns a matrix of text cells. Two special names are allowed:

| Name | Meaning |
|---|---|
| The external-identifier name, written as the last element of a path | The record's **external identifier**, in the form of the package's technical name, a full stop and the local name; missing external identifiers are created on the fly under the package name `__export__` |
| The raw-identifier name | The record's raw database identifier |

**Permission.** The acting user must be a settings administrator or belong to the export group; otherwise the export is refused with:

> "You don't have the rights to export data. Please contact an Administrator."

**The procedure.**

1. Every field named anywhere in the column paths is fetched for the whole record set, recursively through relations, and custom-property values are resolved in one batch per property field.
2. For each record, one row is produced.
3. For each column, in order: a raw-identifier column holds the identifier rendered as text; an external-identifier column holds a marker resolved in one final pass over the whole matrix; a non-relational field holds its **export rendering**, in which a selection renders its label, a relation renders display names, a date or date and time renders the value with a date and time converted to the acting user's time zone, an empty value renders an empty cell, and a numeric zero renders as itself; a relational field with sub-columns is expanded, the sub-columns being exported recursively for the linked records, the first produced row merged into the record's own row and the remaining rows appended after it, which produces the familiar shape of one row per line with the parent columns filled only on the first line; and a relational field with no sub-column is exported as if the display-name sub-column had been asked for.
4. In re-importable mode, which is the default, two overrides apply: a polymorphic reference renders as the transport name, a comma and the identifier; and a many-to-many always renders in a **single cell** as a comma-separated list — of external identifiers when the external-identifier sub-column was asked for, and of display names otherwise.

---

## 15. The generic load operation

This section states the generic record-loading operation and its shapes. The package data files, their load order and their no-update semantics belong to [the package system](package-system.md) and to [data loading and exchange](../data/data-loading-and-exchange.md), which also holds the file formats and the field-path notation used by an import screen.

### 15.1 Contract

- The **column paths** mirror the export shape: each element is a list of field names, where the empty name stands for the display name, the external-identifier name stands for the external identifier and the raw-identifier name stands for the raw database identifier.
- The **matrix** is a row-major list of rows of text values.
- The **result** holds the list of created or updated identifiers in input order — or false when any error occurred; a list of message maps; and the index of the next row to process when the operation stopped on a row limit.

### 15.2 Row grouping

Rows are grouped into records: a row starts a new record, and each following row that carries values **only** in one-to-many columns belongs to the same record and contributes one line. A many-to-one column never starts a new record even though it is relational, because a link cannot span rows.

For each record: non-relational columns are copied into the record map; each relational column group is extracted recursively, producing a list of sub-records; and each custom-property column contributes one entry to a property list built from the container's definition. A property column whose definition cannot be found is refused with **"Property '"** the name **"' doesn't have any definition on '"** the field **"' field"**.

### 15.3 Conversion and resolution

Each extracted record is converted from text to typed values by a converter that resolves, per type: selection labels to stored values; relation display names or external identifiers to identifiers; dates and dates-and-times from the configured input formats; decimals from the configured separators; and booleans from the configured affirmative words. Every failure produces a message map carrying the row range, the column, a readable field label and the error text, and marks the record as failed.

### 15.4 Writing

1. Converted records are accumulated into a batch. Each entry carries the external identifier — prefixed with the current package's technical name when it contains no full stop — the values and the row range.
2. The batch is written inside a savepoint: existing external identifiers are updated, the rest are created. A record map carrying a raw database identifier updates that record.
3. When the batch fails, the savepoint is rolled back and the records are retried **one by one**, each inside its own savepoint, in order to attribute the failure to a row.
4. Per-record failures are classified: a database warning becomes a warning message; a database error becomes an error message carrying the translated constraint message and, when exactly one column is implicated, that column's name; a refusal raised by business behaviour becomes an error message carrying its text; any other failure becomes **"Unknown error during import: "** the failure kind, a colon and the message, with the advice **"Resolve other errors first"**.
5. Error throttling: once ten errors have been recorded **and** the error count is at least one tenth of the rows processed, processing stops with **"Found more than 10 errors and more than one error per 10 records, interrupted to avoid showing too many errors."**
6. When the batch failed as a whole with a business refusal but the records also failed individually, the batch-level message is inserted first, because it usually explains the real cause.
7. When any error was recorded, the whole operation is rolled back to the initial savepoint, the returned identifier list is false, and the registry changes made during the operation are discarded.
8. Records already written are flushed before a name lookup is attempted for a later row, which lets a row refer by name to a record created by an earlier row of the same file.
9. After a load, every custom-properties field of the loaded records is cleaned of properties that are no longer in the container's definition.

### 15.5 Modes

| Context key | Values | Effect |
|---|---|---|
| `mode`, the load mode | `init`, the default, or `update` | In update mode an existing external identifier marked "no update" is left untouched, and a record map with neither an external identifier nor a raw identifier is refused with **"Cannot update a record without specifying its identifier or external identifier"** |
| `module`, the package prefix | A package's technical name; the default is `__import__` | The prefix applied to external identifiers that contain no full stop |
| `noupdate`, the no-update flag | True or false | The "no update" flag stored on the created external identifiers |
| `import_file`, the file-import marker | True or false | Turns on the check that refuses an external identifier prefixed with the technical name of an installed package |
| `import_skip_records`, the skip list | A list of field names | Rows where any of these fields converted to nothing are skipped silently |
| `_import_limit`, the row limit | A whole number | The maximum number of records processed in this call; the returned next-row index tells the caller where to resume |

The guard against reusing an installed package's prefix reports:

> "The record `\<external identifier\>` has the module prefix `\<package\>`. This is the part before the '.' in the external identifier. Because the prefix refers to an existing module, the record would be deleted when the module is upgraded. Use either no prefix and no dot or a prefix that isn't an existing module. For example, `__import__`, resulting in the external identifier `__import__.\<name\>`."

### 15.6 External identifiers

An external identifier links a stable text key — a package's technical name, a full stop and a local name — to one record of one entity, with a no-update flag.

- Loading a record with an external identifier that already exists updates that record, unless the mode is update and the flag is set.
- An external identifier that points at an entity different from the one being loaded is refused with **"For external identifier "** the key **" when trying to create/update a record of model "** the transport name **" found record of different model "** the other transport name **" ("** the identifier **")"**.
- An external identifier whose target row no longer exists is deleted and the record is created anew.
- When a record created through an embedding creates its parent implicitly, the parent also receives an external identifier, named after the child's key, an underscore, and the parent's transport name with its full stops replaced by underscores.
- Deleting a record deletes its external identifiers.

---

## 16. Context keys

The context is an immutable map carried by the environment. It is propagated to every record set derived from a record set, and to every record set reached through a relational field, except where a key's own row says otherwise.

### 16.1 Well-known keys

The company keys are specified in full in [multi-company](multi-company.md) and the language key in [translation](../runtime/translation.md); the effects listed here are the ones the entity layer itself applies.

| Key | Type | Effect |
|---|---|---|
| `lang`, the language | A language code | The language used for translated field values, for selection labels, for field labels and help text, and for message translation. An unknown code is refused with **"Invalid language code: "** followed by the code. Absent, or the base code, means the base language |
| `tz`, the time zone | A time zone name | The zone used to render dates and times, to compute "today" for default values, and to truncate dates and times when grouping. When absent, the acting user's own time zone is used; when that is absent too, coordinated universal time is used. An unknown name is ignored with a debug log entry |
| `active_test`, the archived-visibility key | True or false | When false, archived records are not filtered out of searches and to-many reads |
| `default_` followed by a field name | Any | A default value for that field, applied by every creation performed under this context |
| `allowed_company_ids`, the activated companies | A list of company identifiers | The companies the environment may act in; the **first** is the current company. In a restricted environment every listed company must belong to the acting user's companies, otherwise **"Access to unauthorized or invalid companies."** When absent, the current company is the acting user's default company and the activated companies are all of the user's companies |
| `bin_size`, the size-only key | True or false | When true, binary fields read their size instead of their content |
| `bin_size_` followed by a field name | True or false | The same, for one field only |
| `prefetch_fields`, the prefetch-fields key | True or false | When false, reading a field reads only that field, never its prefetch group |
| `prefetch_langs`, the all-languages key | True or false | When true, a translated field's cache holds every language at once rather than only the context language |
| `edit_translations`, the translation-editing key | True or false | When true, term-by-term translated values are returned wrapped in per-term markers carrying the term's translation state and a digest of the source term |
| `check_translations`, the pending-translation key | True or false | When true, a term-by-term translated value is read in the underscore-prefixed language, which exposes pending translations |
| `delay_translations`, the deferred-translation key | True or false | When true, writing a term-by-term translated field stores the rebuilt translations under underscore-prefixed keys, marking them as pending review |
| `fill_temporal`, the temporal-filling key | True or false, or a map | Temporal gap filling for grouped reads ([section 10.7](#107-group-expansion-and-temporal-filling)) |
| `install_module`, the installing package | A package's technical name | The package currently being installed; used to warn when a record is created with an external identifier belonging to another package |
| `_import_current_module`, the loading package | A package's technical name | The prefix for external identifiers during a load |
| `mode`, `module`, `noupdate`, `import_file`, `import_skip_records`, `_import_limit` | See [section 15.5](#155-modes) | Load behaviour |
| `recursive_onchanges`, the iteration key | True or false | When false, the on-change protocol performs a single pass instead of iterating to a fixed point |
| The duplication cycle guard | Internal | Records the entities and identifiers already visited while duplicating |
| The translation-copy cycle guard | Internal | The same, while copying translations |
| The many-to-one ordering guards | Internal | Cycle guards while resolving a many-to-one ordering, one for the query form and one for the in-memory form |
| The privileged-evaluation marker | Internal | Placed while evaluating a privileged existence condition in memory |

### 16.2 Deriving an environment

| Derivation | Result |
|---|---|
| With a whole context | Replaces the whole context |
| With context overrides | Copies the context and overlays the overrides |
| With a company | Sets the activated companies to the given company followed by the previous list with that company removed; an empty company is refused with a message naming the empty value and stating that it is not a valid company |
| With a user | Replaces the acting user and turns elevated rights **off**; an empty user is refused |
| With elevated rights | See [section 17](#17-elevated-rights-execution) |
| With a whole environment | Replaces the environment entirely |

**Context cleaning.** When elevated rights are turned on from a restricted environment and no explicit context is given, the context is *cleaned*: every default-value key and the archived-visibility key are removed. The same cleaning is applied when the fields without a column of a newly created record are written, so that a default meant for the parent record does not leak into its lines.

### 16.3 Context keys and the cache

A field may declare that it depends on context keys. The cache then holds one entry per combination of key values and record.

| Key | Cache key value |
|---|---|
| The company | The current company's identifier |
| The user | The acting user's identifier when the field is computed with elevated rights; the pair of acting user identifier and unrestricted flag otherwise |
| The language | The context value, or nothing |
| The archived-visibility key | The context value, defaulting to the field's own declared context value, defaulting to true |
| Any key beginning with the size-only prefix | Whether the context value is present |
| Any other key | The context value; a list is converted to a fixed sequence; a value that cannot be used as a map key is refused with **"Can only create cache keys from hashable values, got non-hashable value "** the value **" at context key "** the key **" (dependency of field "** the qualified field name **")"** |

---

## 17. Elevated-rights execution

Deriving an environment with elevated rights returns a record set whose environment carries the unrestricted flag.

| Check | Behaviour with elevated rights |
|---|---|
| The entity gate, for all four operations | Skipped |
| The record gate | Skipped |
| The field gate | Skipped |
| The authorisation of the activated companies | Skipped: any company may be made current |
| Everything else — validations, database constraints, deletion guards, the company consistency check | Unchanged |

Rules:

- The acting user is **not** changed. Elevated rights are a flag, not a user switch; messages, audit fields and the creating-user field still name the real user.
- Deriving an environment for another user turns elevated rights **off**.
- Turning elevated rights on from a restricted environment cleans the context as described in [section 16.2](#162-deriving-an-environment).
- Elevated rights can be turned off again explicitly.
- The root identity's identifier is `1`. An environment whose acting user is the root identity is reported as unrestricted independently of the flag.
- Three predicates are available: whether the environment is unrestricted, being the flag; whether the acting identity administers access, being the flag or membership of the access-rights group; and whether it administers the tenant, being the flag or membership of the settings group.

**The privileged-command guard.** An entity may declare that it does not allow privileged relational commands. When a to-many command targets such an entity, the target record set is rebuilt with elevated rights **off** and with the acting user reset to the transaction's originating user before the command is executed. This prevents a caller that obtained elevated rights for an unrelated reason from editing security-sensitive records through a relational field.

---

## 18. The permission model as the operations apply it

The full specification of users, groups, access rights and record rules is in [the security model](security-model.md); this section states only how the layers are applied by the generic operations.

1. **The entity gate.** An access right grants one of read, create, write or delete on one entity to one group, or to every identity when no group is named. A user is permitted when at least one applicable right grants the operation. The refusal names the entity and the operation.
2. **The record gate.** A record rule attaches a filter to an entity and to a subset of the four operations, optionally restricted to groups. Rules attached to no group are **global** and conjoin; rules attached to groups the user holds **disjoin** among themselves and the result conjoins with the global ones. The resulting filter is applied as an additional condition of every search, and as an after-the-fact check by the three verification operations below, all of which evaluate the rule filter in memory with elevated rights and with archived-record visibility turned off.
3. **The field gate.** A field's group requirement governs both reading and writing, as specified in [the security model, section 7](security-model.md#7-field-level-restrictions).

Three operations expose the first two gates:

| Operation | Contract |
|---|---|
| The raising check | Refuses when the operation is forbidden on the entity in general or on any record of the set. Invoked on an **empty** record set it checks only the entity gate |
| The non-raising check | The same decision, returned as a yes or no |
| The filtering check | The subset of the record set on which the operation is permitted |

All three return immediately as permitted when the environment is unrestricted. Record rules are evaluated only on records that have a database identifier; a set holding in-memory records skips the record gate entirely.

---

## 19. Worked examples

### 19.1 A filter compiled from end to end

An entity Sales Order has a state as a selection, a contact as a many-to-one that is not required, an order date as a date and time, an archive flag, and lines as a one-to-many whose inverse field on the line is required. The acting user's language starts the week on Monday and the time-zone context key names a zone two hours ahead of coordinated universal time in summer.

The caller's filter is the conjunction of "the state equals sale" with the disjunction of "the country code of the contact equals the two-letter code of Belgium" and "the product of a line is one of two given identifiers".

1. **Archived-record visibility.** No condition mentions the archive flag, therefore the condition "the archive flag is true" is conjoined.
2. **Basic stage.** The state equality becomes a membership condition against a one-element set. The archive equality becomes a membership condition against the set holding true. The country-code path is a path whose head is relational, therefore it becomes an existence condition on the contact whose sub-filter is an existence condition on the country whose sub-filter is the code membership. The line-product path becomes an existence condition on the lines whose sub-filter is the product membership.
3. **Full stage.** The environment is restricted and neither relational field declares that it bypasses the target's read permission, therefore both existence operators stay non-privileged. The archive membership has a single value and is left alone; the tautology rule would fire only on the set holding both values.
4. **Compilation.** The contact branch resolves the country condition on the Contact entity with the read rules of Contact applied, giving an identifier sub-query, and the condition becomes a membership of the contact column in that sub-query. Because the contact link is nullable and the condition is positive, no unset alternative is added. The lines branch resolves the product condition on the Sales Order Line entity with the read rules of that entity applied, and additionally excludes rows whose inverse field is empty; the condition becomes an existence test on that sub-query correlated on the inverse field.
5. **Record rules.** The read rules of Sales Order are computed, optimised with elevated rights and with archived-record visibility turned off, and conjoined.
6. **Flushing.** Before execution, every field named in the compiled query is flushed: the state, the archive flag and the contact on Sales Order; the country and the code on Contact and Country; the inverse field and the product on Sales Order Line.

Adding a condition that the order date equals the plain date 15 March 2026 would, at the basic stage, become the half-open interval from 14 March 2026 at 23:00:00 up to but excluding 15 March 2026 at 23:00:00, because the value was written as a plain date, the interval is one day long, and midnight of that date in the acting user's zone is 23:00 of the previous day in coordinated universal time.

Adding a condition that the order date is strictly after 15 March 2026 at 10:30:00 would become "on or after 15 March 2026 at 10:30:01", because a strict lower bound on a date and time is shifted by one second and turned into a non-strict one.

### 19.2 Ordering by a link

The entity Sales Order has a contact as a many-to-one to Contact, and Contact's default order is its display name then its identifier.

Ordering by "the contact, descending, then the order date" compiles as follows. The contact term is a many-to-one whose target's default order is not the identifier, therefore the target table is joined with a left join and the target's default order is delegated, reversed because the direction is descending: the contact's display name descending, then the contact's identifier descending. The order-date term is compiled directly, ascending. The final ordering is therefore the contact's display name descending, the contact's identifier descending, then the order date ascending. Orders with no contact are included, because the join is a left join.

Ordering by "the identifier of the contact, descending" instead compiles to the raw contact column descending with no join at all, because naming the identifier explicitly suppresses the delegation.

### 19.3 A grouped read with a granularity

The request groups the sales orders whose state is sale by the month of the order date and then by contact, aggregating the count of rows, the sum of the total amount and the record set of the lines, with a having filter requiring the sum to exceed 1,000, ordered by the sum descending, limited to five groups.

Three orders are in the filtered set:

| Order | Order date, in coordinated universal time | Contact | Total amount | Lines |
|---|---|---|---|---|
| 1 | 28 February 2026 at 23:30:00 | 7 | 600.00 | 11, 12 |
| 2 | 1 March 2026 at 08:00:00 | 7 | 900.00 | 13 |
| 3 | 4 March 2026 at 09:00:00 | 9 | 400.00 | 14 |

With the time-zone context key naming a zone one hour ahead in February and March before the changeover, order 1 falls on 1 March 2026 in local time and therefore groups with order 2.

Result before the having filter:

| Month of the order date | Contact | Count | Sum of the total amount | Record set of the lines |
|---|---|---|---|---|
| 1 March 2026 at 00:00:00 | Contact 7 | 2 | 1,500.00 | Lines 11, 12, 13 |
| 1 March 2026 at 00:00:00 | Contact 9 | 1 | 400.00 | Line 14 |

The having filter drops the second row. The order is descending on the sum, therefore the first row is returned first. The limit of five leaves both candidate rows before the having filter and one row after it.

Without the time-zone key, order 1 would fall on 28 February 2026 in coordinated universal time and would form its own group for 1 February 2026 with a count of 1 and a sum of 600.00, which the having filter would then drop.

The record-set aggregate returns a record set of Sales Order Line whose prefetch sequence covers every line of every group, therefore reading a field on the lines of all groups issues one statement.

### 19.4 A load round trip

The column paths are: the external identifier; the name; the external identifier of the contact; the external identifier of the product of a line; and the quantity of a line. The matrix has three rows: the first carries an external identifier of `so_a`, the name "Order A", a contact reference, a product reference and the quantity 2; the second carries values only in the two line columns, a second product reference and the quantity 3; the third carries an external identifier of `so_b`, the name "Order B", another contact reference, the first product reference again and the quantity 1.

1. Row one starts a record. Row two carries values only in line columns, therefore it belongs to the same record. Row three starts a new record.
2. Record A is extracted with its external identifier, its name, one contact sub-record and two line sub-records, with the row range one to two. Record B has the row range three to three.
3. Conversion resolves each external identifier to a database identifier, and the two quantity cells to whole numbers using the configured separators.
4. The batch is written inside a savepoint. Each record becomes a creation with the lines expressed as two creation commands.
5. Each record receives an external identifier prefixed with the current package's technical name; the default prefix is `__import__`.
6. Running the same file a second time updates the two orders instead of creating new ones, because the external identifiers now exist. The line commands, being creation commands, **add** two more lines rather than replacing the existing ones; replacing requires the column set to include a line identifier.
7. If the second contact reference did not resolve, record B would fail conversion; a message map naming the row range three to three, the contact column, the field's label and the text of the resolution failure would be recorded; the whole operation would be rolled back to the initial savepoint; and the returned identifier list would be false.

---

## 20. Invariants a rebuild must preserve

1. Every operation is invoked on a record set, and the record set carries its environment; two sets are equal when they hold the same identifiers in the same order, whatever their environments.
2. A creation checks the create right before the values are completed and **again** on the created records afterwards, because a record rule may depend on the values just written.
3. A write checks the record rules on the records **as they stand before the write**; there is no post-write record check.
4. A read that is filtered out by a record rule refuses and names the records; a search silently omits them.
5. The empty value of a type and "not set" are the same thing in a filter, wherever the type declares an empty value.
6. A filter is fully optimised before it is compiled; a non-standard operator surviving the full stage is an error, not a fallback.
7. A privileged existence operator may be produced by the optimiser but never accepted from a caller.
8. Archived-record visibility is switched off for record-rule evaluation, company-consistency checking, hierarchy resolution and dependency inversion.
9. In-memory evaluation and compiled evaluation of a filter agree on every case, including empty values, traversal and hierarchy expansion.
10. Elevated rights change what is allowed, never who is acting.
11. Deletion never leaves an external identifier or an attachment pointing at a record that no longer exists.
12. A load that records any error rolls the whole operation back and returns no identifiers.

---

## 21. Acceptance criteria

**AC-ROQ-1.** *Given* a record set of three records and another holding the same three in a different order, *when* the two are compared, *then* they are not equal; *and when* one is compared with a copy of itself carrying a different environment, *then* they are equal.

**AC-ROQ-2.** *Given* a search returning 2,500 records, *when* the result is iterated and one field is read on each record, *then* three statements are issued, one per chunk of 1,000 records.

**AC-ROQ-3.** *Given* a record set holding one stored identifier and one in-memory identifier, *when* a write is attempted, *then* it is refused with the mixed-set message; *when* a read is attempted, *then* it succeeds.

**AC-ROQ-4.** *Given* a creation whose value map names a key that is not a field, *when* it runs, *then* it is refused with "Invalid field '\<name\>' in '\<transport name\>'".

**AC-ROQ-5.** *Given* a creation on an entity with a record rule that the created record fails, *when* it runs, *then* the creation is refused after insertion by the second create check.

**AC-ROQ-6.** *Given* a value map that supplies a field and a default for the same field in the context, *when* the map is completed, *then* the caller's value wins.

**AC-ROQ-7.** *Given* a precomputed field absent from the value map, *when* the creation runs, *then* the field is evaluated on an in-memory record before insertion and is inserted as a column value.

**AC-ROQ-8.** *Given* a fetch of a non-stored computed field, *when* it runs, *then* the first segment of each of its dependency paths is fetched instead, in one statement.

**AC-ROQ-9.** *Given* a fetch whose query returns fewer records than the set holds, *when* the missing records still exist but are excluded by a record rule, *then* the read is refused and names them; *when* they no longer exist, *then* no refusal is raised.

**AC-ROQ-10.** *Given* a many-to-one whose target row no longer exists, *when* the record is read, *then* the value is false rather than a dangling pair.

**AC-ROQ-11.** *Given* the filter list with a connective that has too few operands, *when* it is parsed, *then* it is refused with "Domain() malformed domain " followed by the list.

**AC-ROQ-12.** *Given* a filter naming a privileged existence operator, supplied by a caller, *when* it is parsed, *then* it is refused with "Domain() invalid item in domain: " followed by the item.

**AC-ROQ-13.** *Given* a nullable integer column, *when* the filter asks for the value zero, *then* records whose value is unset match as well.

**AC-ROQ-14.** *Given* a column covered by a not-null constraint, *when* the filter asks for the empty value, *then* the condition compiles to the constant false and no unset test is added.

**AC-ROQ-15.** *Given* a path whose first segment is not relational, *when* the filter is optimised, *then* the remainder is read as a property of that field and not as a traversal.

**AC-ROQ-16.** *Given* a condition on a many-to-one compared with text under an inequality, *when* the filter is optimised, *then* it is refused with "Inequality not supported for relational field using a string".

**AC-ROQ-17.** *Given* a descendants condition on an entity that stores a materialised ancestor path, *when* it is resolved, *then* it becomes a disjunction of prefix comparisons on the path and no iterative search is performed.

**AC-ROQ-18.** *Given* a descendants condition on an entity that stores no path, *when* it is resolved, *then* the walk is performed with elevated rights and with archived records visible.

**AC-ROQ-19.** *Given* two membership conditions on the same field inside a conjunction, *when* the filter is optimised, *then* they are merged into the intersection of the value sets; *given* the same two conditions on a to-many relation, *then* they are **not** merged.

**AC-ROQ-20.** *Given* an entity with an archive flag and a filter that does not mention it, *when* a search runs, *then* the condition that the flag is true is conjoined; *given* a filter that mentions the flag inside a nested sub-filter, *then* no implicit condition is added.

**AC-ROQ-21.** *Given* an ordering term naming a many-to-one whose target's default order is the display name, *when* the query is built, *then* the target table is joined with a left join and the delegated order is used; *given* the term names the identifier of the target explicitly, *then* the raw column is used with no join.

**AC-ROQ-22.** *Given* a write that sets a field to the value it already holds, *when* it runs, *then* no column update is enqueued for that record.

**AC-ROQ-23.** *Given* a write on a to-many field with an inverse rule, *when* it runs, *then* the field's current value is read before the write, because the field is protected during it.

**AC-ROQ-24.** *Given* a deletion refused by an incoming link declared as restricting, *when* it runs, *then* the refusal names the other entity and suggests archiving instead.

**AC-ROQ-25.** *Given* a deletion, *when* it completes, *then* every external identifier and every attachment pointing at the deleted records has been deleted and the whole cache has been invalidated.

**AC-ROQ-26.** *Given* a grouped read with no group specification and a provably empty filter, *when* it runs, *then* exactly one tuple is returned holding the empty value of each aggregate and no statement is issued.

**AC-ROQ-27.** *Given* a grouped read by the month of a date-and-time field with a time-zone context key, *when* it runs, *then* the values are converted to that zone before truncation; *without* the key, *then* they are truncated in coordinated universal time.

**AC-ROQ-28.** *Given* a grouped read by a one-to-many field, *when* it runs, *then* it is refused.

**AC-ROQ-29.** *Given* a duplication of a record with lines and translations, *when* it runs, *then* the lines are duplicated in ascending identifier order, the translations of every copied translated field are copied for every installed language, and a circular structure terminates through the cycle guard.

**AC-ROQ-30.** *Given* an export requested by a user who is neither a settings administrator nor a member of the export group, *when* it runs, *then* it is refused with "You don't have the rights to export data. Please contact an Administrator."

**AC-ROQ-31.** *Given* a load whose matrix has one row per line after the first, *when* it runs, *then* the following rows are attached to the first record as lines rather than starting new records.

**AC-ROQ-32.** *Given* a load in which eleven rows fail and the failures amount to at least one in ten of the rows processed, *when* it runs, *then* processing stops with the throttling message and the whole operation is rolled back.

**AC-ROQ-33.** *Given* an external identifier prefixed with the technical name of an installed package and the file-import marker set, *when* the load runs, *then* it is refused with the prefix guidance message.

**AC-ROQ-34.** *Given* an environment derived with elevated rights from a restricted one and no explicit context, *when* the derived context is inspected, *then* the default-value keys and the archived-visibility key are absent and the language, time zone and company selection remain.

**AC-ROQ-35.** *Given* an entity that refuses privileged relational commands and an elevated flow writing commands on a to-many pointing at it, *when* the commands are applied, *then* they run as the transaction's originating user without elevation.

**AC-ROQ-36.** *Given* a raising permission check invoked on an **empty** record set, *when* it runs, *then* only the entity gate is evaluated.

**AC-ROQ-37.** *Given* a field declared to depend on a context key whose value cannot be used as a map key, *when* the field is read, *then* the read is refused with the non-hashable cache-key message.

---

## 22. Reconciliation notes

Five decisions about the shape of this document are recorded so that a reader who expects a subject elsewhere can find it, and so that a rebuild knows which of two plausible readings was verified against the running system.

1. **Where the filter grammar lives.** The grammar could sit with the fields it constrains or here. It is here, because this document is where every operation that consumes a filter is specified and because the grammar is longer than the rest of that document's field material put together. [The entity and field system, section 20](entity-and-field-system.md#20-the-filter-grammar) keeps the definition of a filter, the two notations at a glance and the three forms in which a filter is stored, and links here for the operators, the empty-value rules, the optimisation stages and the per-type semantics. No rule is stated twice.
2. **The operator set.** Each operator is given twice over: its transport spelling is reproduced in code font, because a caller writes it literally, and its meaning is stated in words. Neither form alone is enough for a rebuild.
3. **Empty values.** The rule is one rule, but it needs four examples to be usable, because the integer, text, boolean and many-to-one cases each behave differently. All four are kept.
4. **The naming of the unrestricted mode.** This document says **elevated rights** for the flag and **the root identity** for the special user, matching [the security model](security-model.md); the identifier `1` of the root identity is reproduced because integrations depend on it.
5. **The load operation.** [Section 15](#15-the-generic-load-operation) specifies the operation itself, [the package system](package-system.md) specifies when a package's data files are loaded, and [data loading and exchange](../data/data-loading-and-exchange.md) specifies the file formats and the record declaration grammar. The three do not repeat one another.

---

## Related documents

- [Architecture](architecture.md) — the record set, the environment, the unit of work and the flushing rules these operations drive.
- [The entity and field system](entity-and-field-system.md) — field types and attributes, computations, defaults, validations, ordering, display names, the archive flag and where filters are stored.
- [The security model](security-model.md) — the gates whose position in each procedure is stated here, and the exact refusal texts.
- [Multi-company](multi-company.md) — the company context keys and the consistency check invoked by the create and write procedures.
- [Caching](../runtime/caching.md) — the record cache, the pending-write buffer, the recomputation schedule and the flush that precedes every query.
- [Transactions and concurrency](../runtime/transactions-and-concurrency.md) — savepoints, retry and the transaction boundaries these operations run inside.
- [Translation](../runtime/translation.md) — the language fallback chain used by translated fields in filters, reads and copies.
- [The package system](package-system.md) — when a package's data files are loaded and what the no-update flag means.
- [Data loading and exchange](../data/data-loading-and-exchange.md) — the record declaration grammar, the import and export file formats and the exchange contracts.
- [Remote transport contracts](../interfaces/remote-transport-contracts.md) — how these operations are named and invoked from outside.
