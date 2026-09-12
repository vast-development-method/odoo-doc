# Caching, the unit of work and recomputation

Between the record operations and the database sits a layer of memory. This document specifies all of it: the in-memory record cache and its keying, the prefetch mechanism, the protection mechanism that stops a computation from being undone while it runs, the dirty-value tracking that turns assignments into deferred column updates, the flush that applies them, the invalidation rules, the recomputation engine that keeps derived values consistent, the on-change protocol a form uses, the scratch cache of one cursor, the per-database cache containers held by the entity registry with their complete catalogue of memoized operations, the inter-process signalling protocol that makes one process's invalidation reach the others, the process-level caches, the cache statistics, the view, asset, translation and response caches, the batch sizes, and the failure catalogue. A replacement that follows this document issues the same number of database statements in the same order, produces the same values in the same situations, and recovers from the same failures the same way.

The transaction boundary that brackets all of this, with its commit and rollback sequences, its savepoints, its explicit row locks and its retry loop, is specified in [`transactions-and-concurrency.md`](transactions-and-concurrency.md). Field definitions and derivation rules are in [`../overview/entity-and-field-system.md`](../overview/entity-and-field-system.md).

## 1. The five layers

| Layer | Scope | Lifetime | Keyed by | Invalidated by |
|---|---|---|---|---|
| Record cache | one transaction | the transaction | field, then record, then cache key for context-dependent fields | writing, flushing, explicit invalidation, transaction end |
| Scratch cache | one cursor | the cursor | an arbitrary label chosen by the consumer | closing the cursor, committing, rolling back |
| Registry cache containers | one database, one process | until cleared or until the registry is dropped | entity name, operation, then the operation's declared key expressions | an explicit clear, a signal from another process, a registry reload |
| Process-level caches | the whole process | the process | see section 13 | process restart, or the specific rules of section 13 |
| Response caches | the client and any intermediary | the declared maximum age | the request address | the address changing, or the validator changing |

Only the first two are transactional. The other three are **not**: a value cached in a registry container survives a rollback. This is the reason the registry containers may only hold values that are safe to have been computed from a state that was later abandoned, and the reason a failed attempt's invalidations are cancelled rather than signalled (section 10.5).

## 2. The transaction object

One **transaction object** exists per database session. Every record set whose execution context shares that session shares the transaction object. It holds exactly seven pieces of state.

| State | Shape | Purpose |
|---|---|---|
| Execution contexts | a weak ordered set | every execution context created on this session |
| Default execution context | one execution context, or none | the context used when the transaction must flush on its own behalf, chosen as the first context created with a valid acting user |
| Field data | map from field to a per-field structure | the record cache (section 3) |
| Dirty identifiers | map from field to an ordered set of identifiers | values changed in the cache but still owed to the database (section 6) |
| Pending link patches | map from field to a map from record identifier to a list of identifiers | identifiers to add to a relation-to-many value the first time that value enters the cache (section 3.5) |
| Protection | a stack of maps from field to a set of identifiers | fields and records that must not be invalidated or recomputed (section 5) |
| Pending computations | map from field to an ordered set of identifiers | the recomputation schedule (section 8) |

Two whole-transaction operations exist.

- **Clear.** Empties the record cache, the pending link patches, the dirty identifiers, the pending computations and the cursor's scratch cache. It does **not** touch the database. Pending writes are lost.
- **Reset.** Obtains a fresh entity registry, rebuilds every cached attribute of every execution context of the transaction, and then clears. Used after the registry has been reloaded and after a failed attempt that will be retried.

When the transaction must flush on its own behalf and no default execution context has been recorded, the flush is performed as the public user and the warning that a default context is missing is logged.

## 3. The record cache

### 3.1 Shape

The cache is partitioned **first by field, then by record**. For an ordinary field the per-field structure is a map from record identifier to cached value. This shape makes three common operations cheap: listing which records hold a value for a field, removing a field's values for every record, and reading one field for many records.

There is exactly one cached value per pair of a field and a record, except for the cases in sections 3.2, 3.3 and 3.4.

### 3.2 Context-dependent fields

A field that declares a dependency on context keys has a per-field structure that is a map from **cache key** to a map from record identifier to value. The cache key is the ordered tuple of the values of the declared keys, computed as follows.

| Declared key | Value placed in the cache key |
|---|---|
| `company` | the identifier of the current company of the execution context |
| `uid` (acting user) | the acting user key alone when the field is computed with elevated rights; otherwise the pair of the acting user key and the elevated-rights flag |
| `lang` (language) | the language named in the context, or nothing when the context names none |
| `active_test` (archived-record visibility) | the context value when present, otherwise the field's own declared value, otherwise true |
| a key whose name begins with `bin_size` (size-only reading of binary content) | the truth value of the context entry |
| any other key | the raw context value, with a list converted to a tuple |

A value that cannot be hashed raises `Can only create cache keys from hashable values, got non-hashable value <value> at context key <key> (dependency of field <entity>.<field>)`, in which the placeholders are the offending value, the context key, the entity transport name and the field name.

Consequences:

- Switching the active company, the language or the acting user gives a different cache key and therefore a different value, with no database access when both were already read.
- Every cached entry for such a field on a record is considered dirty as soon as one of them is; the flush step decides which value belongs in the column (section 6.4).
- Invalidating such a field invalidates every cache key at once.

### 3.3 Translated fields

A translated field stores, per record, a **map from language to value**. Reading returns a view over that map fixed to one language, resolved as follows:

- when the context asks to prefetch every language, the raw per-language map is exposed and every installed language, plus the source language, plus their per-term counterparts when per-term editing is in force, is filled from the stored document, missing languages falling back to the source language;
- otherwise a language-fixed view is exposed. Reading a record through that view returns the entry for the effective language; when the field is neither computed nor both stored and real, that is a non-stored field or an in-memory record without an origin, the view falls back to the source-language entry.

Writing through the language-fixed view creates the per-language map when it does not exist yet, sets the entry for the effective language, and, for the non-stored or in-memory case, also sets the source-language entry.

Clearing the language-fixed view removes only the current language's entry from every record's map. The storage format and the fallback order are specified in [`translation.md`](translation.md), sections 3 and 4.

### 3.4 Binary fields and the size-only mode

A binary field depends on the context key `bin_size` and on the field-specific key formed by `bin_size_` followed by the field name. Its cache therefore holds two entries per record: the content and the size text. Computing a binary field always computes the **content** entry first, in an execution context with both keys turned off, and then derives the size entry from it: the content is decoded from its base-64 form when the field is attachment-backed, its length is rendered as a size string, and that string is written into the size entry directly, without marking the record dirty.

### 3.5 In-memory link patches

When a relation-to-many field must gain a link to an **in-memory** record, and that field has no value in the cache yet for the owner, the identifier is appended to the pending link patches rather than being written. The reason is that writing a value would claim "this is the complete list", which is false.

The first time a value is written into the cache for that pair of a field and a record, the pending patch identifiers are merged into it, duplicates removed and order preserved, and the patch entry is dropped.

### 3.6 Reading a field: the cache-miss algorithm

Reading a field on a single record proceeds as follows.

1. Field-level permission is checked, unless the execution has elevated rights.
2. When the record set is not of length one: a set of length greater than one raises `Expected singleton: <record set>`; an empty set returns the field's empty value without touching the cache.
3. When the field is computed **and** stored, the pending computations for it are drained for this record first (section 8.6).
4. The cache is consulted. A hit returns the value converted to record format.
5. On a miss, exactly one of five branches applies, in this order of testing.

| Branch | Condition | Behaviour |
|---|---|---|
| Fetch from the database | the field is stored and the record is real | The prefetch set is computed (section 4) and the field is fetched for it. When the fetch raises an access failure and the prefetch set held more than one record, the fetch is retried for the original record alone, in order that the failure is attributed correctly. When the value is still missing afterwards, the record no longer exists: `Record does not exist or has been deleted.` followed by `(Record: <record>, User: <acting user>)`. |
| Copy from the origin | the field is stored, the record is in-memory with an origin, and the field is not both computed and read-only | Each record of the prefetch set that has an origin copies the origin's value into the cache. On failure with more than one record, the copy is retried for the original record alone. |
| Compute | the field is computed | When the record is protected for the field, the empty value is written and no rule runs. Otherwise the rule runs over the prefetch set, or over the record alone when the field is recursive; on an access or missing-record failure the rule is retried for the record alone. Records that still have no value receive the empty value, unless the field is read-only and not stored, in which case `Compute method failed to assign <records>.<field>` is raised. The cache is re-read afterwards, because a rule may have invalidated everything. |
| Build the delegated parent | the field is a delegated link to a parent entity and the record is in-memory | A new in-memory parent record is created holding every inherited value currently in the record's cache, and linked. The inverse relation-to-many fields of the delegate field are updated for the new parent. |
| Default | otherwise: a non-stored non-computed field, or a stored field on an in-memory record with no origin | The empty value is written first, which is necessary for relation-to-many conversion to work, then the default is resolved and written when one exists. |

6. The value is converted from cache format to record format and returned.

Reading a **relational** field on a record set of length greater than one is optimized: pending computations are drained for the whole set, the cache is consulted record by record, and when more than 1 000 consecutive records are missing the field is fetched for the remainder in one statement instead of falling back to the single-record path.

## 4. Prefetching

### 4.1 The prefetch set

When a field must be read from the database for one record, the platform builds the prefetch set as follows.

1. Take the record's own identifier, then the identifiers of the record's prefetch sequence.
2. Keep only identifiers of the same kind as the record: all real, or all in-memory.
3. Remove duplicates, preserving order.
4. Keep only the identifiers that have no value in the cache for this field.
5. Take the first 1 000 of what remains.

### 4.2 The prefetch group

Reading one field also reads every field of the **same prefetch group**, provided the context does not turn field prefetching off and the field's own prefetch group is not false. The fields of the group are filtered to those the acting user may read. The field being read is always included even when its group would exclude it.

| Prefetch group value | Meaning |
|---|---|
| `true` | the default group: every stored column field that is not user-created |
| a name, for example `company_dependent` (per-company storage) | a named group; fields sharing the name are read together |
| `false` | no grouping; the field is read alone and is never pulled in by another field's read |

The fields whose group is `false` are: every non-stored field, every field without a column, every user-created field, and every binary field.

### 4.3 Prefetch sequences carried by relation fields

- A link to one record carries, as its prefetch sequence, the lazily computed sequence of that field's cached values over the owner's prefetch sequence, without duplicates and skipping empties. Reading a field on the linked record of every owner therefore issues one statement.
- A relation-to-many value carries the concatenation of that field's cached value lists over the owner's prefetch sequence, without duplicates.
- Both sequences are reversible, in order that iterating the owner in reverse still prefetches in a useful order.

### 4.4 Inserting fetched values

Values read from the database are inserted into the cache **without overwriting entries that already exist**. This is what makes it safe to fetch a field for a batch that contains records with pending writes: the pending value wins and is still flushed later.

## 5. Protection

Protection marks a pair of a field and a record as "do not invalidate, do not schedule for recomputation, do not compute".

The protection state is a **stack of maps**; entering a protected section pushes a layer, leaving it pops the layer. Inside a layer, the protected identifier set of a field is the union of the outer layer's set and the newly protected identifiers.

Protection is applied in exactly five situations.

| Situation | What is protected |
|---|---|
| A compute rule is running | every field of the co-computed group, on the records being computed |
| A field with an inverse rule, or an editable computed field, is assigned during a write | every field of its co-computed group, on the written records |
| A field is assigned during a create | every field of the co-computed group of every assigned computed field that is editable or precomputed, on the created records |
| A value is assigned on an **in-memory** record | every field of the co-computed group of the assigned field, on that record |
| A record is being deleted | every field of the entity, on the records being deleted |

Effects of protection:

- The modification walk (section 8.3) removes protected records from every pair it produces, therefore they are neither invalidated nor scheduled.
- Reading a protected computed field returns the empty value instead of running the rule.
- Assigning a field on a protected record writes straight to the cache with no business logic and no recomputation.

## 6. The unit of work

### 6.1 Dirty tracking

Assigning a value to a **stored field that has a column** marks the pair of that field and that record **dirty**: the record identifier is added to the field's dirty set. An in-memory identifier is never marked dirty.

Rules:

- Only stored column fields can be dirty. Attachment-backed binary fields, relation-to-many fields and non-stored fields write through immediately or are handled by their own write path, and are never dirty.
- Writing a value equal to the cached value does not mark the record dirty, because the record is filtered out before the cache update.
- Updating the cache of a field on a record that is already dirty, **without** declaring the update dirty, is a defect in the calling code and is logged with a stack trace whose text is `Field._update_cache() updating the value on <records>.<field> where dirty flag is already set`. The value is still written. This detects a code path that would silently drop a pending write.

### 6.2 The three flush operations

| Operation | Full name | Scope |
|---|---|---|
| `flush_all` | flush everything | every pending computation and every pending column update of the transaction |
| `flush_model` | flush one entity | the pending computations of the named fields of one entity over all records, then, when any of them is dirty anywhere, the entity's whole pending column update |
| `flush_recordset` | flush these records | the pending computations of the named fields for the records of the set, then, when any of them is dirty for any of those records, the entity's whole pending column update |

Both entity-scoped forms first drain the recomputation schedule for the named fields (section 8.6) and then, **only if something is dirty**, run the column update for the entity. The column update is not narrowed to the named fields or to the named records: once the entity must be written, everything pending on it is written.

Flushing everything is a fixed-point loop repeated at most 1 000 times:

1. Drain every pending computation on real records.
2. Collect the entities of every field that has a non-empty dirty set. If there are none, stop.
3. Flush each of those entities.

If the thousandth round completes without the dirty sets becoming empty, `Too many iterations for flushing fields!` is logged and the loop ends. The loop is necessary because writing a column can schedule another computation, and computing a field can dirty another column.

### 6.3 The column update

Flushing one entity proceeds as follows.

1. The dirty sets of every field of the entity are popped out of the transaction state, giving a map from field to identifier set. When it is empty, nothing happens.
2. The union of the dirty identifiers is ordered in such a way that records sharing the **same set of dirty fields** are adjacent. Each identifier is mapped to the bit mask of the fields it is dirty for, and the identifiers are sorted by that mask.
3. The identifiers are processed in batches of 1 000. For each batch, one value map per record is built by asking each dirty field for its column value (section 6.4). A missing cache entry at this point is a defect in the calling code and raises `Could not find all values of <record> to flush them`, followed by the context dictionary and a rendering of the cache.
4. The batch is written by the low-level column update, which issues one statement per group of 100 rows sharing the same set of columns.

### 6.4 Choosing the column value

| Field kind | Value written |
|---|---|
| ordinary stored field | the cached value, converted to column format |
| per-company field | a document built from **every** cache key of that field for the record: for each cache key, the company identifier is the first element of the key and the value is the converted cached value. When no cache key has a value, the column is set to empty. |
| translated field | the raw per-language map held in the cache, or empty when it is empty. A translated field may not depend on context keys; the guard `translated field <field> cannot depend on context` enforces this. |
| a field that depends on context keys but is not per-company | the first cache key that holds a value for the record. Having more than one is a modelling defect, because a column can hold only one value; one is picked and the others are lost. When no cache key holds a value: `Value not in cache for field <field> and identifier=<identifier>`. |
| binary field | the cached value read in an execution context with both size-only keys turned off, in order that the size string is never written to the column |

### 6.5 Flushing driven by a query

Every query carries the set of fields whose values it reads or compares. Before the query is executed, those fields are flushed, grouped by entity. This is what makes a search see the values assigned earlier in the same transaction without the caller having to flush explicitly.

The set is built while the query is compiled: producing the query term for a field registers that field. Two deliberate exceptions:

- The identifier column never registers a flush, because it is never pending.
- When a **non-translated** field is selected for reading, its own flush registration is dropped from the selection term, because a value that is pending in the cache will not be overwritten by the fetch (section 4.4). A translated field keeps the registration, because the fetch must see the stored per-language document in order to resolve the fallback chain.

An entity backed by a stored query expression additionally registers the fields of its declared dependency map, transitively.

## 7. Invalidation

Invalidation removes cached values in order that the next read goes to the database.

| Operation | Full name | Scope | Flush first |
|---|---|---|---|
| `invalidate_all` | invalidate everything | every field of every entity | yes by default |
| `invalidate_model` | invalidate one entity | the named fields, or every field, of one entity, all records | yes by default |
| `invalidate_recordset` | invalidate these records | the named fields, or every field, of one entity, the records of the set | yes by default |

Flushing first is the default because invalidating a dirty entry **discards the pending write**. Skipping the flush is allowed only where the pending writes are known to be irrelevant.

Additional rule: invalidating a field also flushes and invalidates every field declared as its **inverse relation**, over all records. Invalidating a link to one record therefore invalidates the relation-to-many fields that mirror it, because their cached lists may now be wrong.

Two places invalidate everything without flushing:

1. **After a delete**, because cascading deletions performed by the database are invisible to the platform.
2. **After any change to a user default value**, because the fallback of every per-company field may have changed. The registry cache containers are cleared at the same time.

## 8. Recomputation

### 8.1 The schedule

The schedule is a map from field to an ordered set of record identifiers. Only fields that are both computed and stored may appear in it; adding any other field raises `Cannot add to recompute no-store or no-computed field`.

Adding is idempotent. Removing a field's last identifier removes the field from the map.

### 8.2 Declaring a modification

Every operation that changes values declares it, naming the record set, the field names, and two flags: whether the records were just created, and whether the declaration is being made before or after the change. Three modes result.

| Mode | When it is used | Behaviour |
|---|---|---|
| ordinary: neither flag set | after a write, after a relation-command execution, after a materialized-path rewrite | the walk marks fields for recomputation immediately, taking already-scheduled fields into account |
| "before": the before flag set | before writing a relational field, and before deleting records | the walk collects what to mark but marks nothing until the end; it must not see the effects of the modification being announced |
| "creation": the created flag set | immediately after inserting rows, and after writing the non-column fields of new records | edges labelled with a link to one record, or with an identifier-only polymorphic link, are **skipped**, because on creation no other record can already reference the new record |

The double call for relational fields is mandatory. Consider two orders and one line. Moving the line from the first order to the second changes which order each computed total belongs to; only a walk performed before the move can reach the first order, and only a walk performed after it can reach the second.

### 8.3 The walk

The walk proceeds as follows.

1. If the declaration is in "before" mode, take the current schedule as the read-only set of already-marked pairs and create a fresh empty map as the set to mark. Otherwise take an empty set of already-marked pairs and use the current schedule itself as the map to mark, writing into it directly.
2. Merge the trigger trees of the declared fields, keeping in each node only the fields that are either computed and stored, or that currently hold a value in the cache. This pruning is what makes the walk cheap: a whole sub-tree whose fields are all non-stored and absent from the cache is dropped before any record is looked up.
3. Start the work list with one entry: walking the merged tree over the declared record set, carrying the created flag.
4. For each triple of a field, a record set and a created flag produced lazily by the work list, in order:
   1. Remove from the record set the records protected for that field. If nothing remains, continue with the next triple.
   2. If the field is recursive: when it is computed and stored, remove from the record set every record already marked or already collected for that field; when it is not, restrict the record set to the records that hold a value in the cache for that field under any cache key. If nothing remains, continue with the next triple. Otherwise append to the work list a walk of that field's own trigger trees over the remaining records with the created flag cleared.
   3. If the field is computed and stored, add the records to the set to mark for that field. Otherwise remove the field from the cache for those records.
5. If the declaration is in "before" mode, add every collected pair to the schedule now.

Walking a tree over a record set yields, in order:

1. the triple of each field in the tree's root node, that record set, and the created flag;
2. then, for every pair of an edge field and a sub-tree, a walk of the sub-tree over the inverted record set (section 8.4) with the created flag cleared.

The walk runs with **elevated rights** and with archived-record visibility turned off, in order that archiving a record or restricting a user never leaves a derived value stale.

### 8.4 Inverting one edge

Given a set of records of one entity and an edge labelled with a field of another entity, meaning "the dependent field is on the second entity and reads this field, which points at the first entity", the records to reach are computed as follows.

1. **Through a declared inverse.** When the edge field has an inverse field on the first entity that is not a relation-to-many carrying a declared filter:
   - for an identifier-only polymorphic link, iterate the records of the first entity, keep those whose companion entity-name field equals the edge field's entity, and collect the values of the inverse; records that no longer exist are skipped;
   - otherwise read the inverse on the records of the first entity; when that raises "record no longer exists", retry on the subset that exists;
   - when the edge field's entity is the same as the reached record set's entity and the records of the first entity are all in-memory, the reached records are converted to in-memory identifiers too.
2. **By searching.** When the edge field has no usable inverse:
   - the real records are found by searching the second entity for records whose edge field is among the real identifiers, ordered by identifier;
   - the in-memory records are found by scanning the cache of the edge field: every record of the second entity that holds a value for that field and whose value intersects the in-memory identifiers is added.

The second path is the reason every intermediate dependency field must be searchable.

### 8.5 What happens to each reached field

| Field kind | Action |
|---|---|
| computed and stored | scheduled for recomputation on the reached records |
| computed and not stored | removed from the cache for the reached records; it will be recomputed at the next read |
| not computed | cannot be reached; a non-computed field never appears in a trigger tree node |

### 8.6 Draining the schedule

Draining is always lazy and always narrowed.

| Trigger | What is drained |
|---|---|
| reading a computed stored field on a record | that field, for that record and for up to 1 000 other scheduled records of the same entity |
| flushing one entity with named fields | each named computed stored field, for every scheduled record |
| flushing a record set with named fields | each named computed stored field, for the records of the set that are scheduled |
| flushing everything | every computed stored field that is scheduled for at least one real record, repeatedly until the schedule is empty |

Draining one field for a set of records proceeds as follows.

1. Read the schedule for the field. If it is empty, stop.
2. If the field is recursive, then for each record of the set in order: if the record is scheduled, run the rule on that record alone.
3. Otherwise, for each record of the set in order: if the record is scheduled, form a batch consisting of that record followed by other scheduled identifiers of the same kind, capped at 1 000, and run the rule on the batch.

Two recovery paths wrap every rule invocation.

- **Missing records.** When the rule raises "record no longer exists", the batch is narrowed to the records that still exist and the rule is run again on those. The records that do not exist are removed from the schedule **for every field of the co-computed group**, which is what prevents an endless retry loop.
- **Access failure.** When the rule raises an access failure on a batch of more than one record, the rule is run again on the single record that triggered the drain, in order that the failure is attributed to that record and not to an unrelated one pulled in by the batch.

Running the rule itself proceeds as follows.

1. When the field is computed with elevated rights, the record set is switched to elevated rights.
2. Every field of the co-computed group that is stored is removed from the schedule **before** the rule runs. This is deliberate: a rule that reads the old value of its own field would otherwise trigger a fetch, which would flush, which would recursively drain the same field.
3. The co-computed group is protected on the records.
4. The rule runs.
5. When the rule raises, every stored member of the group is put back into the schedule for the records, and the failure propagates.
6. When the field is stored and at least one record is real, the validation rules watching any member of the co-computed group run on the real records.

Draining everything is a fixed-point loop over at most 1 000 rounds; exceeding it logs `Too many iterations for recomputing fields!`. In-memory records are never drained by it: they are recomputed on read.

### 8.7 Worked example

Two entities: an Order with a stored computed total that depends on the subtotal of its lines, and a stored computed line count that depends on its lines; and an Order Line with a stored computed subtotal that depends on its quantity and unit price, and a link to its order which is the inverse of the order's lines.

Starting point: order 1 holds lines 10 and 11; the cache holds the total for order 1; nothing is scheduled.

First operation: write a quantity of 3 on line 10.

| Step | Action | Schedule afterwards | Dirty afterwards |
|---|---|---|---|
| 1 | permissions checked | empty | empty |
| 2 | the quantity is not relation-modifying, therefore no "before" walk runs | empty | empty |
| 3 | the quantity is written to the cache for line 10 | empty | quantity on line 10 |
| 4 | the modification is declared for the quantity on line 10; the trigger tree of the quantity has the root node holding the subtotal and no edges | subtotal on line 10 | quantity on line 10 |
| 5 | validation rules watching the quantity run | subtotal on line 10 | quantity on line 10 |
| 6 | the write returns | subtotal on line 10 | quantity on line 10 |
| 7 | at the flush: the subtotal is drained for line 10; the rule reads the quantity from the cache and the unit price from the database and assigns the subtotal | empty for the subtotal | quantity and subtotal on line 10 |
| 8 | assigning the subtotal declares a modification whose trigger tree has the edge along the lines relation to the order's total; inverting that edge on line 10 gives order 1 | total on order 1 | quantity and subtotal on line 10 |
| 9 | the loop runs again: the total is drained for order 1; the rule reads the subtotals of the lines | empty | quantity and subtotal on line 10, total on order 1 |
| 10 | the loop runs again: nothing is scheduled, therefore the dirty entities are flushed: one update statement for the Order Line, carrying two columns for one row, and one for the Order, carrying one column for one row | empty | empty |

Second operation, from the same starting point: write order 2 on line 10.

| Step | Action | Schedule afterwards |
|---|---|---|
| 1 | the order link is relation-modifying, therefore the "before" walk runs. The trigger tree carries the edge along the lines relation to the order's total and line count. Inverting that edge on line 10 gives order 1. Nothing is marked yet; the pairs are collected. | empty |
| 2 | the collected pairs are added: the total and the line count on order 1 | total and line count on order 1 |
| 3 | the order link is written for line 10; the cached lines of order 1 lose line 10 and the cached lines of order 2 gain it | total and line count on order 1 |
| 4 | the ordinary walk runs. Inverting the same edge on line 10 now gives order 2 | total and line count on orders 1 and 2 |
| 5 | at the flush, both orders are recomputed | empty |

Omitting step 1 would leave order 1 with a stale total, which is exactly what the "before" mode exists to prevent.

## 9. The on-change protocol

A form asks the platform what changes when the user edits a field. The call carries the current form values, the names of the fields the user just changed, and a field specification: the tree of fields the form displays, as a map from field name to a map that may carry a nested specification for a relation-to-many field and a context for it.

The result is a map with two optional members: the fields to change on the caller, in the read format and as command lists for relation-to-many fields, and a warning holding a title, a message and a display kind.

### 9.1 Steps

1. The whole transaction is flushed, in order that the protocol behaves identically inside and outside a test harness.
2. A permission check runs: the write permission when the record set holds a record, the create permission when it is empty. The User entity is exempt, because users may edit fields of their own record without holding write permission on it.
3. A changed-field name that is not a field of the entity aborts the protocol and returns an empty result.
4. **First call.** When the list of changed field names is empty, the call means "a new record is being opened": the changed names become every key of the supplied values except the identifier; for every field of the specification that has no supplied value, the default is resolved; when a default exists it is added to the values and to the changed names; when none exists, the field is set to empty **unless** it is a computed field with at least one declared dependency, which is left unset in order that it be computed.
5. **Line prefetching.** Every field of the specification is fetched. For each relation-to-many field with a nested specification and a supplied value, the union of the currently linked identifiers and the identifiers named by the update, link and set commands is collected, the nested fields are fetched for those records in one statement, and the values are copied into the cache of the corresponding in-memory records. This avoids computing stored computed fields on the in-memory lines.
6. **Value isolation.** The supplied values are split into initial values, everything the user did not just change, and changed values, the fields named in the call. A delegate field supplied as empty is removed from the initial values, in order that delegation is not destroyed by a blank.
7. **Building the working record.** When the record set holds a record, an in-memory record is created with that record as its origin, its cache pre-filled with the record's current values for every field of the specification, and the initial values applied on top. When the record set is empty, the changed field names are set to empty in the initial values, in order that resolving a default is not skipped, and an in-memory record is created from them.
8. **Parent alignment.** For every initial value that is an inherited field, the corresponding value is written into the cache of the in-memory parent, in order that computed fields on the parent see their dependencies at the expected value.
9. **Snapshot zero** is taken: the value of every field of the specification on the working record, recursively for relation-to-many fields, as a map from line identifier to a line snapshot. On a first call the snapshot is taken **without** fetching, because everything is already set.
10. The changed values are written into the working record's cache, which triggers the recomputation of stored computed fields that depend on them, including ones the form does not display.
11. Snapshot zero is refreshed for the changed field names only.
12. **The work list** is, on a first call, the changed names followed by every field of the specification without duplicates; otherwise the changed names alone.
13. With the co-computed groups of the changed fields protected on the working record: every field of the entity on a first call, or the work list otherwise, is declared modified; then, for every work-list field that is an inherited field, the value is explicitly assigned on the in-memory parent, because assigning it on the child does not propagate to the parent on its own.
14. **The on-change loop** runs as follows. Keep a set of done names and a set of rules already visited, both initially empty. While the work list is not empty: for each name in the work list, in order, run the declared on-change rules of that field that have not already run in this pass, add the rules that ran to the visited set, and add the name to the done set. Then, if the context key that enables recursive on-change handling is false, stop; otherwise the new work list is the fields of the specification not in the done set whose snapshot-zero value differs from their current value. A rule may assign fields on the working record and may return a warning. Assignments are applied immediately; warnings are accumulated as a set of triples of title, message and kind.
15. **Snapshot one** is taken with the final values.
16. The result's value member is the difference between snapshot one and snapshot zero (section 9.3), forced to "everything" on a first call.
17. Warnings are formatted: one warning becomes a single entry with its kind defaulting to a dialog; several warnings are merged into a single dialog titled `Warnings` whose message is the concatenation of each warning's title and message, separated by blank lines.

### 9.2 Conditional user defaults

A field whose declaration carries the conditional-default attribute `change_default`, the marker that the field may trigger a user-level default lookup, gets an implicit on-change rule: when the field changes, the user default values for the entity are looked up under a condition formed by the field name, an equals sign and the field's write-format value, and every default found is applied to the working record.

### 9.3 The snapshot difference

A snapshot is a map from field name to value, where a relation-to-many field's value is a map from line identifier to a nested snapshot. The difference of a new snapshot against an old one, with a force flag, is computed as follows.

1. Partition the fields of the specification, excluding the identifier, into simple fields and relation-to-many fields. When the force flag is not set and the old and new values of a field are equal, skip that field entirely.
2. The result starts as the read format of the working record for the simple fields, without the identifier.
3. For each relation-to-many field, build a command list. Let the new lines be that field's value in the new snapshot, and the old lines be its value in the old snapshot, or the empty map when the force flag is set. The old snapshot may describe stored records, so its identifiers are mapped to the corresponding in-memory ones first.
   1. For every identifier present in the old lines and absent from the new lines, append a delete command for a one-sided relation-to-many field, or an unlink command for a two-sided one.
   2. For every identifier and line snapshot in the new lines: when the force flag is not set and the identifier is among the old lines, compute the difference of the line snapshot against the old line snapshot and, if it is non-empty, append an update command carrying it. When the line has no origin, append a create command carrying the difference of the line snapshot against an empty snapshot. Otherwise append a link command carrying the stored record's identifier and read format, then compute the difference of the line snapshot against the stored record's own snapshot and, if it is non-empty, append an update command carrying it.
   3. If the command list is non-empty, place it in the result under that field's name.

The identifier carried by a create or a delete command is the line's origin when it has one, its reference when it has one, and `0` otherwise, which lets the form match the command with the line it holds.

### 9.4 Which fields are recomputed in a form

Exactly the fields that would be recomputed on a stored record: the dependency graph is identical. The only differences are:

- values are written into the cache of an in-memory record, never to the database;
- a stored computed field is still computed, because the result must be shown;
- a field that is stored and not both computed and read-only reads its value from the origin record rather than being computed, which preserves a manually entered value across a form round trip;
- a field that is both computed and read-only is computed even on a record with an origin, because its value must follow the edited dependencies.

### 9.5 Batch form

A batch variant applies the protocol to a list of value maps in one call, returning one result per map. It requires an empty record set, therefore it works only on new records.

## 10. The registry, its cache containers and inter-process signalling

### 10.1 What the registry holds

One registry exists per database and per process. It holds the entity definitions, the field definitions, the relation inverses, the dependency graph that drives recomputation, the identity of the database connections, and the cache containers of section 10.2. It also holds two flags per thread: whether this thread has invalidated the registry, and which cache keys this thread has invalidated.

Registries are kept in a bounded, least-recently-used map. Each acquisition refreshes the registry's last-used instant. When the bound is exceeded the least recently used registry is dropped. A registry may also be dropped after a configured idle period; when that period is zero, idle dropping is disabled.

Dropping a registry loses every container it holds and every memoized value in them, but loses no persistent state: the next acquisition rebuilds the registry from the database.

### 10.2 The eight containers

Eight containers exist, each a bounded least-recently-used map of its own.

| Container | Capacity (entries) | Holds |
|---|---|---|
| `default` | 8 192 | Operations with no better home: access resolution, user context, session tokens, menu trees, external identifier lookups. |
| `stable` | 1 024 | Values that change only when configuration data changes: system parameters, installed languages, currencies, decimal precisions, installed packages, field metadata. |
| `templates` | 1 024 | Compiled templates and postprocessed view definitions. |
| `templates.cached_values` | 2 048 | Values computed **inside** a template render and therefore invalid whenever anything else is. |
| `assets` | 512 | The resolved file list of an asset bundle and the links that reference it. |
| `routing` | 1 024 | The routing table of the database, one entry per site discriminator. |
| `routing.rewrites` | 8 192 | Address rewrite rules, one entry per rewritten address. |
| `groups` | 64 | The access group definitions and the group hierarchy used to post-process views. |

Capacity is a count of entries, not bytes. Reaching capacity evicts the least recently used entry; it never produces an error.

### 10.3 The six invalidation keys

Clearing a key clears a fixed set of containers.

| Key | Containers cleared |
|---|---|
| `default` | `default`, `templates.cached_values` |
| `stable` | `stable`, `default`, `templates.cached_values` |
| `templates` | `templates`, `templates.cached_values` |
| `assets` | `assets`, `templates.cached_values` |
| `routing` | `routing`, `routing.rewrites`, `templates.cached_values` |
| `groups` | `groups`, `templates`, `templates.cached_values` |

Two asymmetries must be reproduced exactly:

- Clearing `stable` also clears `default`, because values in `default` are frequently derived from configuration data held in `stable`. The converse is not true.
- Clearing `groups` also clears `templates`, because the group visibility of a view is baked into the postprocessed view stored in `templates`.

The container `templates.cached_values` is cleared by every key and has no key of its own. Clearing **all** keys clears every container and marks every key as invalidated.

### 10.4 Keying of a memoized operation

The key of an entry is the tuple: the entity name, the operation itself, then the value of each key expression the operation declares, in order. Key expressions are evaluated against the operation's own arguments and against the execution context.

Two operations of two different entities never collide. Two calls of the same operation with different context values collide only if the operation does not declare the context values as key expressions; an operation that depends on the acting user, the company or the language and does not declare them is a defect.

A key that cannot be hashed produces the warning `cache lookup error on <key>`, increments the error counter, and makes the call fall through to the uncached path. It is never fatal.

### 10.5 Inter-process signalling

**Why.** Several worker processes serve one database, each with its own registry and its own containers. A change in one process must reach the others. The platform uses an append-only table per signal rather than a sequence, because sequence values do not propagate through streaming replication and a read replica must be able to observe the signal.

**The signal tables.** Seven tables exist: one for the registry itself and one per invalidation key of section 10.3. Each has an auto-incrementing key and a creation instant. They are created if missing when a registry is built, and one row is inserted at creation in order that the largest key is defined. A registry keeps, in memory, the largest key it has seen in each of the seven tables.

**The check.** At the start of every database-backed request, and before every scheduled job body, the registry reads the largest key of all seven tables in one statement, then:

1. If the registry signal differs from the remembered value, log `Reloading the model registry after database signaling.`, rebuild the registry from scratch, which starts with empty containers, and remember the new registry signal. The cache signals are **not** examined, because a rebuilt registry has nothing to clear.
2. Otherwise, for each cache key whose signal differs from the remembered value, clear every container of that key **without signalling in turn**, and remember the new value.
3. If anything was cleared at step 2, log `Invalidating caches after database signaling: <sorted container names>`.

Clearing during the check deliberately bypasses the ordinary clear operation; otherwise a process would echo the signal back and the processes would clear each other forever.

**The signal.** Signalling happens **after** the commit of the unit of work, and never before:

1. If the registry is not ready, log a warning and do nothing.
2. Otherwise, if this thread invalidated the registry, open a separate transaction, append one row to the registry signal table, and increment the remembered value by one.
3. Otherwise, if this thread invalidated any cache key, open a separate transaction, append one row to each corresponding signal table, and increment each remembered value by one.
4. Clear both thread-local invalidation markers.

A registry invalidation subsumes every cache invalidation, because reloading a registry starts with empty containers; the cache signals are therefore not emitted in that case. If another process appended a row concurrently, the remembered value of this process is now behind the table; the next check detects that and clears again. A redundant clear is harmless.

**Cancelling.** When a unit of work fails, the invalidations it recorded are cancelled instead of signalled:

1. If this thread invalidated the registry, rebuild the entity definitions in place from the database and clear the marker.
2. If this thread invalidated any cache key, clear the corresponding containers locally and clear the markers.

The local containers are still cleared, because the failed attempt may have polluted them with values computed from state that was rolled back. Only the announcement is suppressed.

**Maintenance.** The automatic cleanup job deletes, from each of the seven tables, every row that is neither among the last ten by key nor less than one hour old. This keeps the tables small while preserving a readable recent history of when each signal fired.

## 11. The catalogue of memoized operations

Every operation below is memoized in the named container. The list is exhaustive for the platform and for the capability packages it ships.

### 11.1 Container `stable`

| Operation | Purpose | Cleared by |
|---|---|---|
| Read one system parameter | Returns the stored value of a parameter key, read with a direct statement, which keeps it usable while the entity definitions are being rebuilt. | Creating, writing or deleting any system parameter. |
| The password derivation configuration | The verifier scheme and its iteration count. | Writing the iteration-count parameter, through the parameter rule above. |
| Resolve a language by its active state | The installed languages and their data. | Writing a language record. |
| All currencies with their rounding | The currency table used by every monetary rounding. | Writing a currency or a rate. |
| The decimal precision of a usage | The number of decimal places declared for a usage name. | Writing a precision record. |
| The key of an entity definition, the keys of several, the cached field metadata, all manually declared field data, the access groups of an entity | Entity metadata lookups. | Writing entity or field definitions. |
| The key of a capability package, the set of installed packages | Package state lookups. | Installing, upgrading or uninstalling a package. |
| The telephone prefix of a country | Used by telephone normalization. | Writing a country. |
| Whether the sanitized telephone search index exists | Avoids probing the schema on every search. | A schema change. |
| The property definition record of a property field | Resolves the parent that owns the definitions. | Writing a property definition. |
| The fields a portal user may read on a task, the fields a user may read and write on themselves | Field-level permission sets. | Writing the corresponding entity definitions. |
| The imported package names and their client translations | Packages imported at runtime. | Importing or removing such a package. |
| The cached description of an external directory service | Avoids re-parsing the service description. | Writing the company record that names it. |

### 11.2 Container `default`

| Operation | Purpose |
|---|---|
| Compute an access-rule condition for an entity and a mode | The record-rule condition, per user group set. |
| Verify a user key and secret pair | Avoids re-deriving the verifier on every remote call. |
| Compute a session binding token for an identifier | See [`sessions-and-authentication.md`](sessions-and-authentication.md), section 6. |
| The context of a user | The language, time zone and acting user key placed in every execution context. |
| The company keys of a user, the accessible company branches, the contact keys of companies | Company scoping. |
| The group keys of a user | Permission evaluation. |
| The lock timeouts of a group set | See [`sessions-and-authentication.md`](sessions-and-authentication.md), section 9. |
| Resolve an external identifier to an entity and a key | The external identifier index. |
| The visible menu keys of a user, the menu tree, the root menu tree | Menu rendering. |
| The action bindings of an entity, the existing action keys | Contextual action lists. |
| The default values declared for an entity, and the fallback columns of a default | User and company defaults. |
| Whether a table has any row | Used by installation-time guards. |
| The tracked fields of an entity | Change tracking, see [`logging-and-audit.md`](logging-and-audit.md), section 4. |
| The default message subtypes and the automatic subscription subtypes | Discussion thread subscriptions. |
| The working-hours intervals of a working schedule | Scheduling computations. |
| The default unit of measure of a product, the first possible variant of a template, the variant matching a combination | Product resolution. |
| The default work entry types | Payroll work entries. |
| The analytic plans and the project plan | Analytic distribution. |
| The translation bundle fingerprint for a package set and a language | See section 14.3. |
| The site keys, the site cached values, the front-end language set, the price-list ordering of a site | Storefront resolution. |
| The active lead-generation rules | Lead generation. |
| The external translation project list | Translation service integration. |
| The tax data of a regulated document flow | Regulatory reporting. |

### 11.3 Container `templates`

| Operation | Purpose |
|---|---|
| Resolve a template reference to its definition record | Template lookup by key or by external identifier. |
| Compile a template to its executable form | The compiled template, keyed by the reference and by the declared context keys. |
| Produce the postprocessed view for an entity and a view type | The combined inherited view for **every** group; group-restricted blocks are removed afterwards, per user. |
| Drop the per-transaction compiled batch when the container was cleared | A sentinel entry whose disappearance tells the current transaction to discard its own compiled batch. |
| Whether the menu cache is disabled for a site | Storefront menu rendering. |

Compilation and postprocessing are **not** memoized when the deployment runs in template development mode: the entry is computed on every call, which picks up a changed definition immediately.

### 11.4 Container `assets`

| Operation | Purpose |
|---|---|
| The ordered file list of a bundle, for a set of bundle parameters | Which files belong to a bundle, before compilation. |
| The links that reference a bundle, for a set of bundle parameters, a direction and a prefixing option | The address list handed to the client. |

Both are skipped in template development mode.

### 11.5 Containers `routing`, `routing.rewrites` and `groups`

| Container | Operation | Purpose |
|---|---|---|
| `routing` | The routing table of the database, keyed by an optional site discriminator | Path matching. Logs `Generating routing map for key <key>` on each miss. |
| `routing` | The number of address rewrite rules | Lets the request layer skip rewriting entirely when there are none. |
| `routing` | The static page addresses of a site | Site page routing. |
| `routing.rewrites` | The rewrite of one address | One entry per rewritten address, which is why this container is the largest. |
| `groups` | The group definitions, as a resolvable structure | Permission evaluation and view post-processing. |
| `groups` | The group hierarchy used by the view post-processor | Deciding which blocks of a view a group may see. |

## 12. The scratch cache of a cursor

Every cursor carries a plain key-value store with no eviction. It is cleared when the cursor is closed, when the transaction commits and when it rolls back, and is **not** affected by a savepoint. It may therefore only hold values that are repeatable reads: values that cannot change during the life of the transaction.

| Label | Contents | Consumer |
|---|---|---|
| memoized cache lookups of this transaction | the set of container keys already looked up | the cache statistics of section 15 |
| compiled template batch | the parsed templates already compiled in this transaction | the template engine |
| translation payload | the translation bundle computed for the client | the client bootstrap endpoint, when the caller asks for it |
| property export map | the property definitions already resolved for an export | the export path resolver |

## 13. Process-level caches

| Cache | Contents | Invalidation |
|---|---|---|
| Registry map | The resident registries, bounded and least-recently-used | Eviction by capacity, by idle period, or explicit deletion when a database is dropped or its packages are changed |
| Connection pools | Open database connections | Idle timeout, capacity eviction, explicit close of a database |
| Code translation store | For each pair of a package and a language, the server-side message map and the client-side message map, loaded from the package's translation catalogue on first use | Never, within a process. A change to a shipped catalogue takes effect only in a new process, which is why the translation reload job exists for externally managed translations |
| Session store | Nothing in memory; every read and write goes to the medium | Not applicable |
| Locale parsing | Parsed locale descriptions, keyed by language code | Never |
| Geographic databases | The opened resolution databases | Never |

## 14. Derived-artifact caches

### 14.1 Views

Producing the view a client will render has two stages with two different cache behaviours.

| Stage | Cached | Key | Notes |
|---|---|---|---|
| Combine the base definition with every inheriting definition, validate and post-process | yes, in `templates` | the view key or type, the entity, plus the options the caller passed | The stored result contains the blocks of **every** group. |
| Remove the blocks the acting user's groups may not see | no | none | Performed per request, on the cached result. |

Because the first stage is group-independent, one entry serves every user, and a change to group membership does not invalidate it; only a change to the group **definitions** does, through the `groups` key which also clears `templates`. Writing, creating or deleting a view definition clears the `templates` key.

### 14.2 Asset bundles

An asset bundle is a named list of source files that is concatenated, transformed and served as one resource.

| Stage | Where the result lives | Invalidation |
|---|---|---|
| Resolve the bundle name to an ordered file list | container `assets` | the `assets` key |
| Compile the file list into one resource | an attachment marked public, owned by the superuser, attached to the view entity with record key 0, whose declared address encodes the bundle name and a content fingerprint | deleting those attachments |
| Produce the links the client loads | container `assets` | the `assets` key |

The fingerprint in the address is what makes the compiled resource safely cacheable by the client and by intermediaries for one year: a change in the sources produces a different address.

Serving a bundle whose compiled attachment does not exist yet **generates it during the request**. When the request is running on a read-only cursor, the read-only transaction is rolled back first and a read/write transaction is opened for the generation, because the generation writes an attachment.

Regenerating every bundle is an explicit operation: it deletes every attachment that matches the bundle address pattern, is public, is owned by the superuser and is attached to the view entity with record key 0, and then clears the `assets` key.

Clearing the `assets` key happens on: creating, writing or deleting an asset declaration; changing a company's visual identity; and the explicit regeneration, which deletes the compiled attachments and then clears the key.

### 14.3 Translations

Three distinct caches exist.

| Cache | Layer | Key | Contents | Invalidation |
|---|---|---|---|---|
| Code translations | process | a package and a language | The message maps extracted from the package's translation catalogue, one for server-side messages and one for client-side messages, each filtered by the marker that identifies its origin | Never within a process |
| Client translation fingerprint | registry container `default` | a package set and a language | A hexadecimal digest of the whole client translation payload: the language parameters (name, code, direction, date format, time format, digit grouping, decimal separator, thousands separator, first day of week) and the per-package message maps, plus a flag telling whether more than one language is installed | Any clear of the `default` or `stable` key |
| Record translations | record cache | field, record, language | Translated field values | Ordinary record-cache invalidation |

The fingerprint is what the client sends back as a validator; the payload itself is served from a long-lived response cache keyed by that fingerprint. A language installation, a language removal or a translation load must therefore clear the `stable` key, which clears `default`, which drops the fingerprint.

Only the user's own language is loaded. A regional language inherits from its parent language at load time, not at lookup time. The complete rules are in [`translation.md`](translation.md).

### 14.4 Response caches

| Resource | Maximum age | Validator | Visibility |
|---|---|---|---|
| A static file of a package | 604 800 seconds (one week), or zero in asset development mode | modification instant, size and a checksum of the path | shared caches allowed |
| A resource whose address contains a content fingerprint | 31 536 000 seconds (one year), marked immutable | the same | shared caches allowed |
| An attachment or a binary field | none by default; the caller may request the immutable one-year form | the content checksum | shared caches only when the attachment is public or an access token was presented |
| An image variant | as above | the content checksum extended with the variant parameters (see [`attachments-and-file-store.md`](attachments-and-file-store.md), section 8) | as above |

A response that is not marked public has the private directive set and the public directive removed, therefore an intermediary cache may not store it.

## 15. Cache statistics

For each pair of a database and a memoized operation the process maintains eight counters, which are the observable measure of cache effectiveness.

| Counter | Increment rule |
|---|---|
| hits | A lookup found an entry. |
| misses | A lookup found no entry and the value was computed and stored. |
| errors | The key was not hashable; the value was computed and **not** stored. |
| generation time | Accumulated duration of the computations performed on misses. |
| transaction hits, transaction misses, transaction errors | The same three, but counted only for the **first** lookup of a given key within a transaction. The set of keys already looked up is held in the cursor's scratch cache. |
| container name | The container the operation belongs to. |

Two ratios are derived.

```formula
hit ratio in percent = hits ÷ (hits + misses) × 100
first-lookup hit ratio in percent = transaction hits ÷ (transaction hits + transaction misses) × 100
```

Each denominator is forced to 1 when it would otherwise be 0. The distinction matters: an operation called ten times in one transaction with one miss and nine hits has a hit ratio of 90 percent but a first-lookup ratio of 0 percent, which correctly says that the cache did not help across transactions.

The statistics can be dumped on demand; the dump is produced on a separate thread, which keeps it from ever blocking the worker.

## 16. Batch sizes and limits

| Limit | Value | Where it applies |
|---|---|---|
| Prefetch batch | 1 000 records | building a prefetch set; extending a recomputation batch; chunking the iteration of a large record set |
| Insert batch | 100 value maps | the insert statement of a create |
| Column-update batch | 100 rows | one update statement of a low-level column update |
| Flush batch | 1 000 records | building the value maps of one column-update pass |
| Membership-list limit | 1 000 identifiers | chunking a membership condition, and chunking the delete loop |
| Recomputation fixed point | 1 000 rounds | draining everything and flushing everything |
| Filter optimization fixed point | 1 000 rounds | optimizing one filter |
| Cursor flush rounds | 10 | the pre-commit callback loop of [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 5 |
| Transient vacuum batch | 100 000 rows | one pass of the transient-record cleanup |
| Retry attempts | 5 | the concurrency retry loop |
| Retry backoff | uniformly random between zero seconds and two raised to the power of the attempt number | the concurrency retry loop |

## 17. Rules a replacement must follow

1. **Never store a record set in a registry container.** The container outlives the transaction; a stored record set would hold a closed cursor. Store keys and plain values.
2. **Declare every context value an operation depends on as a key expression.** An operation that reads the acting user, the company or the language without declaring it will return another user's value.
3. **Clear, do not update.** Every registry invalidation is a clear of whole containers, never a surgical removal of one entry. This is what makes the signalling protocol a single integer per key.
4. **Signal after the commit.** Announcing before the commit would publish an invalidation for a transaction that may still abort.
5. **Cancel on failure, but clear locally.** See section 10.5.
6. **Do not signal from inside the check.** See section 10.5.
7. **Treat a redundant clear as free.** The protocol is intentionally allowed to over-clear under concurrency.
8. **Flush before invalidating.** Invalidating a dirty record-cache entry discards a pending write.
9. **Declare relational modifications twice.** Once before the change and once after; see section 8.2.

## 18. Worked examples

### 18.1 A parameter change reaching two workers

Two workers, A and B, serve one database. The `stable` signal table currently has largest key 17.

| Step | Worker A | Worker B |
|---|---|---|
| 1 | A request writes the system parameter that holds the upload limit. Writing a parameter clears the `stable` key locally: containers `stable`, `default` and `templates.cached_values` are emptied, and the key `stable` is recorded as invalidated by this thread. | idle, remembers 17 |
| 2 | The unit of work commits. | no activity |
| 3 | Signalling opens a separate transaction and appends one row to the `stable` signal table, which now has largest key 18. The remembered value of A becomes 18. | no activity |
| 4 | no activity | The next request reads the seven largest keys and finds `stable` at 18 against a remembered 17. It empties `stable`, `default` and `templates.cached_values`, remembers 18, and logs `Invalidating caches after database signaling: ['default', 'stable', 'templates.cached_values']`. |
| 5 | no activity | The endpoint reads the parameter and gets the new value. |

### 18.2 A package installation reaching two workers

| Step | Worker A | Worker B |
|---|---|---|
| 1 | Installs a capability package; the registry is marked invalidated by this thread. | remembers registry signal 4 |
| 2 | Commits; appends one row to the registry signal table, which reaches 5; remembers 5. No cache signals are emitted. | no activity |
| 3 | no activity | The next request finds the registry signal at 5 against 4, logs `Reloading the model registry after database signaling.`, rebuilds the registry from scratch with empty containers, and remembers 5. |

### 18.3 A failed attempt does not invalidate the others

| Step | Effect |
|---|---|
| 1 | A request writes a currency, which clears the `stable` key locally and records the invalidation. |
| 2 | The flush fails with a serialization failure. |
| 3 | The retry loop rolls back, resets the transaction, and cancels the invalidations: containers `stable`, `default` and `templates.cached_values` are emptied again locally, and the marker is cleared. |
| 4 | No row is appended to any signal table. The other workers keep their caches. |
| 5 | The attempt is replayed; if it succeeds, the signal is emitted then. |

### 18.4 Cache statistics of a cached menu tree

In one transaction the menu tree operation is called four times with the same key. The container was empty.

| Call | hits | misses | transaction hits | transaction misses |
|---|---|---|---|---|
| 1 | 0 | 1 | 0 | 1 |
| 2 | 1 | 1 | 0 | 1 |
| 3 | 2 | 1 | 0 | 1 |
| 4 | 3 | 1 | 0 | 1 |

```formula
hit ratio in percent = 3 ÷ (3 + 1) × 100 = 75.00
first-lookup hit ratio in percent = 0 ÷ (0 + 1) × 100 = 0.00
```

In the next transaction the first call is a hit, and both ratios rise.

### 18.5 One read serving a whole list

A list view shows 80 orders and reads the customer name of each. Reading the customer link on the first order fills the cache for all 80, because the prefetch sequence of the record set is the whole set. Reading the name on the first customer then fills it for every distinct customer, because the link value carries the sequence of cached link values over the owner's prefetch sequence. Two statements serve the whole list: one for the orders' own columns of the default prefetch group, one for the customers'.

## 19. Failure catalogue

| Situation | Result |
|---|---|
| Reading a field on a record set of length greater than one where a single record is expected | `Expected singleton: <record set>` |
| Reading a stored field on a record that no longer exists | `Record does not exist or has been deleted.` followed by `(Record: <record>, User: <acting user>)` |
| A compute rule assigns nothing on a read-only non-stored field | `Compute method failed to assign <records>.<field>` |
| Flushing a field whose value is missing from the cache | `Could not find all values of <record> to flush them`, with the context dictionary and a rendering of the cache |
| Flushing a context-dependent non-per-company field with no value under any cache key | `Value not in cache for field <field> and identifier=<identifier>` |
| A translated field declared with a context dependency | `translated field <field> cannot depend on context` |
| Updating a dirty cache entry without declaring the update dirty | logged with a stack trace: `Field._update_cache() updating the value on <records>.<field> where dirty flag is already set` |
| Scheduling a field that is not both computed and stored | `Cannot add to recompute no-store or no-computed field` |
| A context value that cannot be used as a cache key | `Can only create cache keys from hashable values, got non-hashable value <value> at context key <key> (dependency of field <entity>.<field>)` |
| A registry container key that cannot be hashed | logged: `cache lookup error on <key>`; the value is computed and not stored |
| Recomputation does not converge | logged: `Too many iterations for recomputing fields!` |
| Flushing the fields does not converge | logged: `Too many iterations for flushing fields!` |
| The cursor flush loop does not converge | logged: `Too many iterations for flushing the cursor!` |
| A record set mixing real and in-memory records is written | `<record set> contains a mix of real and new records. It is not supported.` |

## 20. Acceptance criteria

1. **Given** a record whose field was read once in a transaction, **when** it is read again without an intervening invalidation, **then** no statement is issued.
2. **Given** a list of 80 records, **when** one column-backed field is read on the first record, **then** exactly one statement is issued and every record of the list holds a value for that field and for every other field of its prefetch group.
3. **Given** a binary field, **when** it is read in size-only mode, **then** the content entry is computed first in a context with the size-only keys turned off, the size entry is derived from it, and the record is not marked dirty.
4. **Given** a context-dependent field with values cached under two cache keys, **when** the entity is flushed and the field is not per-company, **then** the first cache key holding a value is written to the column and the other is lost.
5. **Given** a per-company field with values cached for companies 1 and 2, **when** the entity is flushed, **then** the column holds a document carrying both values keyed by company identifier.
6. **Given** an assignment of a value equal to the cached value, **when** the write completes, **then** the record is not dirty and no statement is issued at the flush.
7. **Given** a write followed by a search whose condition mentions the written field, **when** the search runs, **then** the field is flushed first and the search sees the new value.
8. **Given** a computed stored field scheduled for 1 500 records, **when** it is read on one of them, **then** the rule runs on a batch of at most 1 000 records including that one.
9. **Given** a relational write that moves a line from one parent to another, **when** both walks have run, **then** both parents are scheduled for recomputation.
10. **Given** the same relational write with the "before" walk omitted, **when** the flush completes, **then** the original parent holds a stale derived value; this is the failure the two-walk rule prevents.
11. **Given** a compute rule that raises, **when** the drain fails, **then** every stored member of the co-computed group is put back into the schedule for the affected records.
12. **Given** a compute rule that raises a missing-record failure on a batch, **when** the drain retries, **then** the missing records are removed from the schedule for every field of the co-computed group and the rule runs on the survivors.
13. **Given** a delete, **when** it completes, **then** the whole cache is invalidated without flushing.
14. **Given** a memoized operation whose key expressions do not include the acting user, **when** two different users call it in the same process, **then** both receive the same value, which is why every user-dependent operation must declare the acting user as a key expression.
15. **Given** an entry stored in the container `stable`, **when** the key `stable` is cleared, **then** the containers `stable`, `default` and `templates.cached_values` are all emptied and the containers `templates`, `assets`, `routing`, `routing.rewrites` and `groups` are untouched.
16. **Given** an entry stored in the container `templates`, **when** the key `groups` is cleared, **then** that entry is removed.
17. **Given** an entry stored in the container `groups`, **when** the key `templates` is cleared, **then** that entry survives.
18. **Given** a system parameter write followed by a commit, **when** a second worker serves its next request, **then** it clears the same three containers and reads the new value.
19. **Given** a system parameter write followed by a rollback, **when** a second worker serves its next request, **then** no signal row exists, its caches are untouched, and it still reads the old value.
20. **Given** a capability package installation, **when** a second worker serves its next request, **then** it rebuilds its registry and does not additionally process any cache signal.
21. **Given** a registry map that has reached its capacity, **when** a request arrives for a database that has no resident registry, **then** the least recently used registry is dropped and the new one is built.
22. **Given** an unhashable cache key, **when** the operation is called, **then** a warning is logged, the error counter is incremented, the value is computed, and nothing is stored.
23. **Given** a view definition that is written, **when** the same view is requested afterwards, **then** it is recombined and re-post-processed rather than served from the container.
24. **Given** an asset bundle whose compiled attachment does not exist and a request served on a read-only cursor, **when** the bundle is requested, **then** the read-only transaction is rolled back, a read/write transaction generates the attachment, and the resource is served.
25. **Given** an asset bundle address that contains a content fingerprint, **when** it is served, **then** the response declares a maximum age of 31 536 000 seconds and the immutable directive.
26. **Given** a language installation, **when** the client requests its translation bundle, **then** the fingerprint differs from the previous one and the new bundle is delivered rather than a cached one.
27. **Given** a private attachment, **when** it is streamed, **then** the response carries the private directive and no public directive.
28. **Given** an operation called four times in one transaction on a cold container, **when** the statistics are read, **then** the hit count is 3, the miss count is 1, the transaction hit count is 0 and the transaction miss count is 1.
29. **Given** the automatic cleanup job, **when** it runs on a signal table holding 500 rows of which 3 are less than one hour old, **then** the surviving rows are the last 10 by key together with any of the 3 recent ones not already among them.
30. **Given** a form on-change call with an empty list of changed field names, **when** the protocol runs, **then** defaults are resolved for every unsupplied field of the specification and the result carries every field rather than a difference.

## 21. Reconciliation notes

1. The record cache, the unit of work and the recomputation engine were specified in one document, and the registry cache containers, the signalling protocol and the derived-artifact caches in another. They are one subject: the second layer invalidates on the events the first layer produces. They are merged here, with the layer table of section 1 as the map between them. Every reference elsewhere in this folder now points to this file.
2. Both sources described the transaction boundary, the commit and rollback sequences, the savepoints, the explicit row locks and the retry loop. Those are specified once, in [`transactions-and-concurrency.md`](transactions-and-concurrency.md); this document keeps only the parts that bear on the cache: the flush points, the fixed-point loops, the batch sizes and the invalidation cancellation.
3. The two sources named the flush, invalidation and locking operations differently. The reproduced names are `flush_all`, `flush_model`, `flush_recordset`, `invalidate_all`, `invalidate_model`, `invalidate_recordset`, `lock_for_update` and `try_lock_for_update`; the earlier spellings were paraphrases and are not contractual.
4. One source stated that registering a transaction callback is idempotent when a key is supplied, the other that adding the same callback twice queues it twice. The queue is not de-duplicated: two additions run twice. Callers that need once-only behaviour aggregate their work in the queue's data dictionary and register one callback that consumes it. This is recorded in [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 4.
5. The cache-key computation was referred to by section number in another document. It is written out in full in section 3.2 so that this folder stands alone.
6. The attribute that makes a field trigger a user-level default lookup was described only by its effect. It is declared on the field as `change_default`, and section 9.2 now names it, because a rebuild has to know which declaration produces the implicit on-change rule.
