# Persistence: identity and values

Everything that determines what a stored value *is*: how records are identified, how they are named and displayed, how amounts are rounded, how dates and times are anchored, how text is cleaned and translated, how structured and binary content is held, how records are archived instead of deleted, how changes are stamped and tracked, how hierarchies are materialized, how a value can differ per company, and which uniqueness, check and deletion rules the store enforces. The physical shapes behind these rules are in [`physical-data-catalog.md`](physical-data-catalog.md); the entity map is in [`domain-model.md`](domain-model.md); the loading, import and export of data is in [`data-loading-and-exchange.md`](data-loading-and-exchange.md); the records a fresh installation must contain are in [`reference-data.md`](reference-data.md).

Every rule below is observable behavior: a replacement that follows it stores the same values, shows the same messages, sorts the same way and rounds to the same cent.

## 1. Vocabulary

| Term | Meaning |
|---|---|
| Record | One instance of an entity, one row of its table. |
| Identifier | The surrogate integer primary key of a record, unique inside its entity. |
| External identifier | A stable textual key, of the form `<package>.<name>`, that names a record across installations. |
| Business reference | A user-visible code or number carried by a record (an invoice number, an order reference, a product code). |
| Display name | The single line of text that represents a record wherever it is referenced. |
| Currency precision | The number of decimal places implied by the rounding factor of a currency. |
| Named decimal precision | A configurable number of decimal places, shared by every field that names it (for example "Product Price"). |
| Active record | A record whose `active` flag is true. Archived records have it false and are hidden from default queries. |
| Company-dependent value | A value that differs per company while living on one record. |
| Acting user | The user under whose rights the current operation runs. |
| Transaction timestamp | The timestamp taken when the database transaction started; every record created or written in one transaction shares it. |

## 2. Surrogate identifiers

### 2.1 Rules

1. Every persistent record and every interactive assistant record has an identifier (`id`): a positive integer, unique inside its entity, assigned by the store from the table's own generator.
2. The identifier is never supplied by a caller. A create that carries an `id` value has that value silently removed before anything else happens. The same holds for `create_date`, `create_uid`, `write_date`, `write_uid` and `parent_path`. The only exception is the system user while the store is being built, which may supply them.
3. Identifiers are never reused. Deleting record 42 does not free the number 42.
4. Identifiers are not consecutive. A transaction that consumes a number and then fails leaves a gap. No business rule may depend on the absence of gaps, and no document number may be derived from an identifier.
5. Identifiers are comparable inside one entity: a larger identifier means a later creation. This ordering is used as the final tie-breaker of every default sort, and nowhere else.
6. Identifiers are not comparable across entities: identifier 42 of Journal Entry and identifier 42 of Contact are unrelated.
7. A record that does not yet exist in the store (a record being edited in a form, a record proposed by a default) carries a temporary identifier that is never written to the store and never leaks out of the editing session. Any value that refers to it must be resolved before the transaction ends.

### 2.2 Identity of a record across entities

A polymorphic link names a record by the pair (entity name, identifier). The entity name is the technical name of the entity, and the pair is written either as two columns (a text column and an integer column) or as one text column holding `<entity name>,<identifier>`. Both forms are used; both are specified in [`physical-data-catalog.md`](physical-data-catalog.md), section 3. Rules:

1. A polymorphic link carries no foreign key. Deleting the target leaves a dangling pair.
2. Every read of a polymorphic link must tolerate a missing target: the link reads as empty, and an operation that needs the target raises the missing-record error.
3. When a record that may be the target of polymorphic links is deleted, the operations that own those links are responsible for cleaning them. The two general cleanups are: attachments whose owner is deleted are deleted with it, and discussion messages whose owner is deleted are deleted with it.

## 3. External identifiers

An external identifier is the stable name of a record. It is what makes shipped data updatable, what makes imports repeatable, and what lets one installation refer to a record of another.

### 3.1 Structure

| Part | Rule |
|---|---|
| Package prefix | The technical name of the capability package that owns the record, or `__import__` for records created by a user import, or `__export__` for records named by an export. |
| Name | A textual key, unique inside the prefix. It may not contain a space; a name with a space is refused by a check constraint whose message is "External IDs cannot contain spaces". |
| Full form | `<prefix>.<name>`, for example `base.main_company`. |

The registry of external identifiers is the entity External Identifier (`ir.model.data`, table `ir_model_data`). It holds, per entry: the local name (`name`), the package prefix (`module`), the entity name (`model`), the record identifier (`res_id`), and a not-updatable flag (`noupdate`). The pair (prefix, name) is unique. The pair (entity name, record identifier) is indexed, because the reverse lookup (which external identifier names this record) is frequent.

### 3.2 Resolution

1. Resolving `<prefix>.<name>` returns the pair (entity name, record identifier) or fails with "External ID not found in the system: `<prefix>.<name>`".
2. Resolution is cached; creating, changing or deleting an entry clears the cache.
3. Resolution does not check access rights. A separate operation resolves the identifier and then verifies that the acting user may read the record, returning the entity name and no identifier when the record exists but is not readable, or raising "Not enough access rights on the external ID" followed by the identifier in double quotation marks when the caller asks for an error.
4. A resolution that finds an entry whose record no longer exists is treated as "not found", and the stale entry is removed the next time the same external identifier is loaded.

### 3.3 Creation and update

External identifiers are created in three situations:

1. **Loading a data file of a capability package.** Every record declared in a data file carries an external identifier prefixed with the package name. On a later installation or upgrade of the same package, the record is found through its external identifier and updated in place, unless its entry is marked "not updatable".
2. **Importing a data file with an external identifier column.** The prefix is `__import__` when the file supplies a bare name, and a user-supplied prefix is refused when it is the name of an installed package, with the message: "The record `<external identifier>` has the module prefix `<prefix>`. This is the part before the '.' in the external id. Because the prefix refers to an existing module, the record would be deleted when the module is upgraded. Use either no prefix and no dot or a prefix that isn't an existing module. For example, `__import__`, resulting in the external id `__import__.<name>`."
3. **Exporting records with their external identifier.** An export that asks for the external identifier column creates one for every exported record that does not have one, with the prefix `__export__` and a generated name of the form `<entity name with underscores>_<identifier>`.

Writing an entry is an insert-or-update on (prefix, name): when a row already exists, the entity name, record identifier and update timestamp are replaced only when the target actually changed.

### 3.4 The "not updatable" flag

| Flag | Behavior on a later package installation or upgrade |
|---|---|
| false (the default) | The record is rewritten with the values from the data file. Values changed by users are overwritten. |
| true | The record is created if missing and left untouched if present. This protects data that users are expected to edit (default configuration records, demonstration records, records that hold accumulated state). |

### 3.5 Deletion

When a capability package is uninstalled, every record named by an external identifier with that package's prefix is deleted, in reverse dependency order, and the entries are removed. A record whose deletion is refused by a restrict rule stops the uninstall with the message of that rule.

Duplicating a record that carries an external identifier does not duplicate the identifier: the copy receives a new entry whose name is the original name followed by an underscore and four random hexadecimal digits, in order that the two records stay distinguishable.

## 4. Business references and document numbering

Three different mechanisms produce user-visible references. They are not interchangeable.

### 4.1 Free references typed by a user

A reference field that a user fills (a customer order reference, a vendor bill reference, a product code) is plain text. The rules it follows are the rules of its entity: it may be required, it may be unique (per company, per entity, per partner), and it may be indexed for search. Nothing generates it.

### 4.2 Counter sequences

A counter sequence is a configuration record of the entity Sequence (`ir.sequence`, table `ir_sequence`) that produces the next value of a counter on demand. Its fields:

| Field | Full name | Type | Default | Meaning |
|---|---|---|---|---|
| `name` | Name | text, required | | The label of the sequence. |
| `code` | Sequence Code | text | empty | The key by which business operations ask for this sequence. Several sequences may share a code, one per company. |
| `implementation` | Implementation | selection: `standard`, `no_gap`, required | `standard` | How the counter is stored and locked (see below). |
| `active` | Active | boolean | true | An archived sequence is not selected by a lookup by code. |
| `prefix` | Prefix | text, never trimmed | empty | Text placed before the number, with placeholders (section 4.3). |
| `suffix` | Suffix | text, never trimmed | empty | Text placed after the number, with placeholders. |
| `padding` | Sequence Size | integer, required | 0 | The number is left-padded with zeros to this width. |
| `number_next` | Next Number | integer, required | 1 | The next number, for the no-gap implementation. |
| `number_next_actual` | Actual Next Number | integer, derived, writable | | The next number as a user sees and edits it: for the no-gap implementation it is `number_next`; for the standard implementation it is read from the database generator. Writing it restarts the generator. |
| `number_increment` | Step | integer, required | 1 | The step. Zero is refused with the message "Step must not be zero." |
| `company_id` | Company | link to Company (`res.company`) | the active company | The company the sequence belongs to; empty means every company. |
| `use_date_range` | Use subsequences per date range | boolean | false | Whether the counter restarts per date range. |
| `date_range_ids` | Subsequences | list of Sequence Date Range (`ir.sequence.date_range`) | empty | The subsequences, one per range. |

A subsequence is a record of the entity Sequence Date Range (`ir.sequence.date_range`, table `ir_sequence_date_range`):

| Field | Full name | Type | Default | Meaning |
|---|---|---|---|---|
| `date_from` | From | date, required | | First day of the range. |
| `date_to` | To | date, required | | Last day of the range. |
| `sequence_id` | Main Sequence | link to Sequence, required | | The sequence this range belongs to. |
| `number_next` | Next Number | integer, required | 1 | The next number inside the range. |
| `number_next_actual` | Actual Next Number | integer, derived, writable | | The same value as a user sees and edits it. |

**The two implementations.**

| Implementation | Behavior | Concurrency | Gaps |
|---|---|---|---|
| `standard` | The counter is a database generator created with the sequence record, starting at `number_next` and stepping by `number_increment`. | Concurrent callers never wait. | A transaction that draws a number and then rolls back leaves a gap. |
| `no_gap` | The counter is the `number_next` column of the sequence record. Drawing a number locks that row for update without waiting (a lock held by another transaction raises an error rather than queueing), reads the value, and increments it in the same statement. | Concurrent callers of the same sequence serialize on the row lock. | No gaps, except when a numbered record is deleted afterwards. |

Changing the implementation of an existing sequence creates or drops the underlying database generator accordingly. Changing `number_next` restarts the generator at that value. Changing `number_increment` changes the step of the generator and of every subsequence.

**Drawing a number.** Two entry points exist:

1. *By record*: the caller holds the sequence record and asks for the next value, optionally for a given date.
2. *By code*: the caller gives a code and, optionally, a date. The sequences whose code matches and whose company is either the active company or empty are read, ordered by company, and the first one is used, which means a company-specific sequence wins over a shared one. When no sequence matches, the operation returns "no value" and the caller must handle the absence; nothing is created implicitly.

**Date ranges.** When `use_date_range` is true, the value comes from a subsequence:

1. The effective date is the date given by the caller, or the date carried by the execution context, or the current date.
2. The subsequence whose range contains the effective date is selected.
3. When none exists, one is created for the calendar year of the effective date, with the range truncated in order that it does not overlap the neighboring ranges: the upper bound becomes the day before the start of the next range, and the lower bound becomes the day after the end of the previous range.
4. A subsequence carries its own next number and its own database generator; the prefix, suffix and padding come from the parent sequence.
5. Two ranges of one sequence may not have the same pair of bounds: "You cannot create two date ranges for the same sequence with the same date range."

### 4.3 Placeholders in prefixes and suffixes

A prefix or suffix may contain placeholders that are replaced when the number is drawn. Three families exist: the effective date of the draw (no qualifier), the start of the date range (`range_` qualifier) and the current moment (`current_` qualifier).

| Placeholder | Value | Example for 5 March 2026, range starting 1 January 2026 |
|---|---|---|
| `year` | four-digit year | 2026 |
| `y` | two-digit year | 26 |
| `month` | two-digit month | 03 |
| `day` | two-digit day of month | 05 |
| `doy` | day of year, one to three digits | 64 |
| `woy` | week of year, Monday as first day | 09 |
| `weekday` | day of week as a digit, Sunday is 0 | 4 |
| `h24` | hour on 24 | 14 |
| `h12` | hour on 12 | 02 |
| `min` | minute | 07 |
| `sec` | second | 33 |
| `isoyear` | year of the week-based calendar | 2026 |
| `isoy` | two-digit year of the week-based calendar | 26 |
| `isoweek` | week number of the week-based calendar | 10 |
| `range_<any of the above>` | the same, computed on the first day of the date range | `range_year` is 2026 |
| `current_<any of the above>` | the same, computed on the current moment | `current_year` is 2026 |

The times are computed in the time zone of the acting user. A placeholder that is not in this list, or a malformed placeholder, raises "Invalid prefix or suffix for sequence “<name>”".

**Worked example.** A sequence with prefix `INV/<year>/`, padding 5, step 1, next number 41, standard implementation, drawn on 5 March 2026, produces `INV/2026/00041`. The next draw produces `INV/2026/00042`. If the same sequence uses date ranges and a range for 2027 is created, the first draw of 2027 produces `INV/2027/00001`, because the subsequence has its own counter.

### 4.4 Self-incrementing document numbers

Accounting and inventory documents do not use counter sequences for their main number. They derive the next number from the highest number already used by comparable documents. The mechanism is carried by every entity that declares a numbered document; it names one text field (the number) and one date field (the date that anchors the period), and it maintains two derived stored columns:

| Column | Rule |
|---|---|
| `sequence_prefix` | Everything before the trailing digits of the number. |
| `sequence_number` | The trailing digits of the number, as an integer, or 0 when the number has none. |

Both are recomputed whenever the number changes, by matching the number against the fixed pattern "any text, then at most nine digits, then any non-digit text". For `INV/2026/00041` the prefix is `INV/2026/` and the number is 41.

**Detecting the reset period.** The pattern of the previous number decides how the series restarts. The candidates are tried in this order, and the first one whose pattern matches and whose named parts are all present wins:

| Reset period | Pattern (in words) | Example |
|---|---|---|
| `year_range_month` | text, start year, separator, end year, separator, two-digit month, separator, digits, optional non-digit suffix | `2025-2026/03/0007` |
| `month` | text, year, optional separator, two-digit month, separator, digits, optional suffix | `INV/2026/03/0007` |
| `year_range` | text, start year, separator, end year, separator, digits, optional suffix | `2025-2026/0007` |
| `year` | text, two- or four-digit year, separator, digits, optional suffix | `INV/2026/0007` |
| `never` | text, at most nine digits, optional non-digit suffix | `INV0007` |

A year range is accepted only when the end year is exactly the start year plus one, compared on the number of digits actually written. Two-digit years are compared modulo one hundred, which makes both `25-26` and `2025-2026` qualify.

**Choosing the previous number.** The candidate is the greatest number, ordered by `sequence_number` descending, among the documents that satisfy the entity's own restriction (typically: same company, same journal or same operation type, same state) and whose `sequence_prefix` equals the prefix of the most recently created such document. The record being numbered is excluded. Because the comparison is on the prefix of the *most recent* document, changing a prefix in the middle of a period does not take effect until the series restarts: with `INV/2026/0001` and `INV/2026/0002` present, renaming the last one to `BILL/2026/0001` still yields `INV/2026/0003` for the next document, because `INV` sorts after `BILL` only when the most recent document carries the `INV` prefix.

**Building the next number.** From the previous number, the mechanism derives a format made of the fixed parts and the variable parts (start year, end year, month, counter), applies the current document's date to the variable parts, and increments the counter:

1. When the reset period is `never`, the counter is incremented and the year and month parts are absent.
2. When it is `year`, the counter restarts at 1 if the year of the document's date differs from the year in the previous number.
3. When it is `month`, the counter restarts at 1 if the year or the month differ.
4. When it is `year_range`, the counter restarts at 1 when the document's date falls in a different range; the range bounds come from the entity (a fiscal year that does not start in January produces ranges that span two calendar years).
5. The counter keeps the width of the previous counter: a previous `0007` produces `0008`, and the ninth thousand produces `9999` then `10000`, widening only when it overflows.
6. When no previous number exists, the entity supplies a starting number (typically `00000000`, which is then incremented to `00000001`, unless the entity defines a richer start such as `<journal code>/<year>/00000`).

**Concurrency.** Assigning a number is serialized through the uniqueness constraint of the number itself:

1. The transaction writes the candidate number on its own row, which takes an exclusive lock on the index entry of the unique constraint that covers the number. The constraint must actually cover the row (a partial constraint restricted to posted documents requires the row to be posted first), otherwise the lock is not taken and numbers may collide under load.
2. If the write fails because the number is taken, the transaction waits for the other transaction to finish, increments the counter and retries, in a loop, until a free number is found. Each attempt runs inside a savepoint.
3. Once a number is taken by the transaction, the following numbers of the same format are assigned from a transaction-local cache, without further savepoints, because the lock already blocks the other transactions.
4. The cache is keyed by the format string with a zero counter and the value of the index field of the entity (typically the journal), and it is cleared whenever the number field is written explicitly.

**The date-alignment rule.** A document whose number does not match its date is refused when the date is later than a configurable threshold date (a configuration parameter, default 1 January 1970):

> The <date field label> (<date>) you've entered isn't aligned with the existing sequence number (<number>). Clear the sequence number to proceed.
> To maintain date-based sequences, select entries and use the resequence option from the actions menu, available in developer mode.

Alignment means: the year written in the number equals the year of the period that the date belongs to, and the month written in the number equals the month of the date. A number without a year part or without a month part is aligned by definition.

**Worked example.** A journal whose last posted entry is `INV/2026/00041` with date 5 March 2026, reset period `year`:

| New document date | Previous number found | Reset decision | Assigned number |
|---|---|---|---|
| 7 March 2026 | `INV/2026/00041` | same year, increment | `INV/2026/00042` |
| 2 January 2027 | `INV/2026/00041` | year changed, restart | `INV/2027/00001` |
| 7 March 2026, second concurrent transaction | `INV/2026/00042` (locked by the first) | wait, then increment | `INV/2026/00043` |

## 5. Monetary and decimal values

### 5.1 The three kinds of numeric value

| Kind | Scale | Storage | Rounded on write |
|---|---|---|---|
| Integer | none | integer (32 bits) | not applicable |
| Decimal with a scale | a fixed pair of digits, or a named decimal precision resolved at write time | exact decimal | yes, to the scale |
| Decimal without a scale | none | double precision (64 bits) | no |
| Monetary | the precision of the currency named by the field's currency field | exact decimal | yes, to the currency's rounding factor |

A decimal without a scale is used where a fixed number of places would destroy information: conversion rates, ratios, geographic coordinates, factors. Everything that a user reads as a quantity, a price or an amount has a scale.

### 5.2 Named decimal precisions

A named decimal precision is a configuration record of the entity Decimal Precision (`decimal.precision`, table `decimal_precision`) with a usage name (`name`) and a number of digits (`digits`). A decimal field may name one instead of declaring a fixed scale; the scale is then read from the record at the moment the value is written or placed in the working set. Rules:

1. Resolving a name returns the number of digits of the record with that usage name, or **2** when no record exists with that name.
2. The effective declaration of such a field is (16 significant digits, the resolved number of decimal places).
3. The resolution is cached for the whole installation and the cache is cleared whenever a precision record is created, changed or deleted.
4. Reducing a precision does not rewrite the stored values. The user is warned when the value is lowered in a form: "The precision has been reduced for <usage>.\nNote that existing data WON'T be updated by this change.\n\nAs decimal precisions impact the whole system, this may cause critical issues.\nE.g. reducing the precision could disturb your financial balance.\n\nTherefore, changing decimal precisions in a running database is not recommended."
5. A usage name is unique: "Only one value can be defined for each given usage!"

The precisions that a fresh installation ships, with their defaults, are listed in [`reference-data.md`](reference-data.md), section 10.

### 5.3 Currency precision

A currency carries a **rounding factor**, a positive decimal (the constraint "The rounding factor must be greater than 0!" refuses zero or a negative value). The number of decimal places is derived from it:

```
decimal_places = ceiling(log10(1 / rounding_factor))   when 0 < rounding_factor < 1
decimal_places = 0                                     otherwise
```

| Rounding factor | Decimal places | Meaning |
|---|---|---|
| 0.01 | 2 | Cents. The common case. |
| 0.001 | 3 | Three decimal places, used by a few currencies. |
| 1 | 0 | Whole units only. |
| 0.05 | 2 | Amounts are multiples of five cents; two decimal places are shown. |
| 0.5 | 1 | Amounts are multiples of half a unit. |

The rounding factor, not the number of decimal places, is what rounding uses: a factor of 0.05 snaps amounts onto a five-cent grid, which two decimal places alone would not do.

### 5.4 The rounding algorithm

Every rounding in the system is performed by one procedure, with one of five tie-breaking rules. Its inputs are a value and either a number of decimal places or a rounding factor, never both.

Procedure:

1. Convert a number of decimal places `n` into the rounding factor `10^-n`. A factor must be strictly positive; a number of decimal places must be a non-negative whole number.
2. If the factor is zero or the value is zero, the result is zero.
3. Normalize: divide the value by the factor. When the factor is smaller than 1, the division is replaced by a multiplication with the exact inverse of the factor, taken from a table of exact inverses for the factors 1, 2 and 5 times a power of ten, and computed from the decimal representation otherwise. This avoids the representation error that a division by a value such as 0.01 would introduce.
4. Compute a correction term: `epsilon = 2^(log2(|normalized value|) − 50)`. It is far below the magnitude of the value and just above the representation error of a sequence of binary floating point operations.
5. Apply the tie-breaking rule to the normalized value:

| Rule | Operation | Name |
|---|---|---|
| Half away from zero (the default everywhere) | add the epsilon with the sign of the value, then round half away from zero | `HALF-UP` |
| Half towards zero | subtract the epsilon with the sign of the value, then round half away from zero | `HALF-DOWN` |
| Half to even | take the integer part; when the remainder is within the epsilon of one half, choose the even neighbor; otherwise round half away from zero | `HALF-EVEN` |
| Always away from zero | truncate the value increased by one minus the epsilon, with the sign of the value | `UP` |
| Always towards zero | truncate the value increased by the epsilon, with the sign of the value | `DOWN` |

6. Denormalize: multiply by the factor again (or divide by the exact inverse).

Any other rule name is a definition error: "unknown rounding method: <name>".

**Worked examples** (tie-breaking half away from zero unless stated):

| Value | Precision | Result | Why |
|---|---|---|---|
| 2.675 | 2 decimal places | 2.68 | The binary value is slightly below 2.675; the correction term lifts it onto the tie, which then rounds away from zero. Without the correction the result would be 2.67. |
| −2.675 | 2 decimal places | −2.68 | The correction carries the sign of the value. |
| 0.015 | 2 decimal places | 0.02 | Tie, away from zero. |
| 0.01499 | 2 decimal places | 0.01 | Below the tie. |
| 0.4555 | 3 decimal places | 0.456 | Tie at the third place. |
| 0.4555 | 4 decimal places | 0.4555 | Already exact at four places. |
| 1.3 | factor 0.5 | 1.5 | Snapping onto a half-unit grid: 1.3 / 0.5 = 2.6, rounds to 3, times 0.5. |
| 1.24 | factor 0.05 | 1.25 | 1.24 / 0.05 = 24.8, rounds to 25, times 0.05. |
| 2.5 | 0 decimal places, half to even | 2 | The even neighbor of the tie. |
| 3.5 | 0 decimal places, half to even | 4 | The even neighbor of the tie. |
| 1.0001 | 0 decimal places, always away from zero | 2 | |
| 1.9999 | 0 decimal places, always towards zero | 1 | |

### 5.5 Comparing and testing for zero

Two operations exist, and they are not equivalent:

| Operation | Definition | Use |
|---|---|---|
| Is zero | Round the value onto the precision grid; the value is zero when the absolute value of the rounded result is strictly smaller than the grid step. | Testing whether a computed residue can be ignored. |
| Compare | Round both values onto the grid, subtract, and test the difference for zero on the same grid. Returns −1, 0 or +1. | Ordering two amounts at a stated precision. |

**Worked example of the difference.** At two decimal places, 0.006 and 0.002:

- Compare: 0.006 rounds to 0.01, 0.002 rounds to 0.00, the difference is 0.01, which is not zero, and the comparison therefore returns +1: the two amounts are *different*.
- Is zero on the difference: 0.006 − 0.002 = 0.004, which rounds to 0.00, and the difference *is* therefore zero.

Both answers are correct for their question. A balance check ("is the entry balanced?") tests the difference for zero. A price comparison ("is the invoiced price higher than the ordered price?") compares. Choosing the wrong one changes behavior, and every rule in this specification therefore states which one it uses.

### 5.6 Representing and splitting

| Operation | Definition | Example |
|---|---|---|
| Represent | Format with exactly the stated number of decimal places, after replacing a value that is zero at that precision by zero, in order that no negative zero is ever printed. | 1.5 at 2 places is `1.50`; −0.001 at 2 places is `0.00`. |
| Split into text parts | Round, represent, then split on the decimal separator; the fractional part always has the stated number of digits, and is empty when the precision is zero. | 1.432 at 2 places is (`1`, `43`); 1.49 at 1 place is (`1`, `5`); 1.1 at 3 places is (`1`, `100`); 1.12 at 0 places is (`1`, ``). |
| Split into numbers | The same, as two whole numbers; the fractional part is zero when the precision is zero. | 1.432 at 2 places is (1, 43). |
| Divide with remainder | Round both operands onto the grid, scale them to whole numbers, divide with remainder, then scale the remainder back. The identity `dividend = quotient × divisor + remainder` holds exactly at the stated precision. | 7.30 divided by 2.40 at 2 places is (3, 0.10). |

### 5.7 Monetary values

1. A monetary field names a currency field on the same record. When it does not, the currency field is the field named `currency_id` if the entity has one, and otherwise the entity has no monetary fields at all; a monetary field without a resolvable currency field is a definition error.
2. On write, the amount is rounded with the rounding factor of the currency, then written with the currency's number of decimal places.
3. The currency used at write time is resolved in this order: the currency present in the values being written; the currency reachable from a value being written through a single link; the currency already stored on the record, read with elevated rights and without prefetching other fields.
4. When no currency is resolved, the amount is written unrounded.
5. When the value being written concerns several records holding different currencies, the write is refused: an amount can only be rounded with one currency.
6. The rounding applies again when the value is placed in the working set, in order that a value written and read back within one transaction compares equal to what was stored.

**Worked example.** An amount of 1234.567 in a currency with rounding factor 0.01 is stored as 1234.57. The same amount in a currency with rounding factor 1 is stored as 1235. The same amount in a currency with rounding factor 0.05 is stored as 1234.55 (1234.567 / 0.05 = 24691.34, rounds to 24691, times 0.05).

### 5.8 Currency conversion

Converting an amount from one currency to another:

1. When the two currencies are the same, the rate is exactly 1 and the amount is returned unchanged.
2. Otherwise the rate is the inverse rate of the source currency expressed in the target currency, at the given date, for the given company. Rates live on the root company of a company tree.
3. The converted amount is rounded with the rounding factor of the **target** currency, unless the caller explicitly asks for an unrounded result.
4. An amount of zero converts to zero without reading any rate.
5. Converting from an unknown currency, or to an unknown currency, is a definition error.

The rate lookup, the rate table and the exchange difference rules are owned by the multi-currency domain; see [`../domains/multi-currency/calculations.md`](../domains/multi-currency/calculations.md).

### 5.9 Aggregation

Sums, averages, minimums, maximums and counts over a decimal or monetary column are computed by the store on the stored values, without re-rounding. The consequence a replacement must reproduce: the sum of rounded amounts is the sum of the stored values, which can differ from the rounded sum of unrounded amounts. Every rule in this specification that aggregates monetary values states whether it sums rounded values (the normal case, because storage is rounded) or rounds a sum.

An amount aggregated across records with different currencies is meaningless; the aggregation of a monetary column is offered only when the currency column can itself be aggregated or grouped, and a report that mixes currencies must convert first.

## 6. Dates and times

### 6.1 The two temporal types

| Type | Holds | Stored | Time zone |
|---|---|---|---|
| `date` | A calendar date | date | None. A date is the same date everywhere. |
| `datetime` | A moment | timestamp without time zone | The stored value is always in coordinated universal time. No offset is stored. |

A moment carries no sub-second part: the current moment is taken with the microseconds removed, and a value that carries an offset is refused when written ("Datetime field expects a naive datetime: <value>").

### 6.2 Textual forms

| Type | Canonical text form | Example |
|---|---|---|
| `date` | year, month, day separated by hyphens | `2026-03-05` |
| `datetime` | the date form, a space, then hours, minutes and seconds on two digits each | `2026-03-05 14:07:33` |

Rules:

1. Parsing a date from text reads only the first ten characters, which makes a moment acceptable where a date is expected: the time part is discarded.
2. Parsing a moment from text accepts a truncated form and completes it: `2026-03-05` becomes `2026-03-05 00:00:00`.
3. Converting a date to a moment sets the time to midnight. Converting a moment to a date discards the time, and with it the time zone: the date obtained is the date in coordinated universal time, not the date in the reader's zone. Every rule that needs "the date as the user sees it" uses the operation of section 6.4 instead.

### 6.3 The time zone of a reader

The effective time zone is, in order: the zone carried by the execution context, then the zone configured on the acting user, then coordinated universal time. An unknown zone name falls back to coordinated universal time and is logged.

The time zone affects only presentation and grouping. It never affects what is stored.

| Operation | Time zone applied |
|---|---|
| Writing a moment | None. The value must already be in coordinated universal time. |
| Reading a moment as a value | None. |
| Rendering a moment for display, in a report or in an export | Converted to the reader's zone, then formatted with the reader's language format. |
| Grouping moments by day, week, month, quarter or year | Converted to the reader's zone before truncation, in order that a moment at 23:30 local time counts in the local day. |
| Filtering moments | None. The condition is evaluated on the stored value; a client that offers "today" as a filter computes the two bounds in the reader's zone and sends them as moments in coordinated universal time. |

### 6.4 "Today" and "now"

Four distinct operations exist, and using the wrong one shifts records by a day:

| Operation | Result | Use |
|---|---|---|
| Current date | The current calendar date of the machine, without any zone conversion. | Never as a default for a user-facing date. |
| Current date as seen by the reader | The current moment, interpreted in coordinated universal time, converted to the reader's zone, reduced to its date part. | The default of every user-facing date field, and every "is it due today" test. |
| Current moment | The current moment, in coordinated universal time, with the microseconds removed. | The default of every moment field, and every timestamp written by an operation. |
| Start of the current day | The current moment with hours, minutes and seconds set to zero. | Bounds of a day-long interval expressed as moments. |

**Worked example.** A user in a zone at coordinated universal time plus 13 hours, at 2026-03-05 09:00 local time, which is 2026-03-04 20:00 in coordinated universal time:

- Current date: `2026-03-04`.
- Current date as seen by the reader: `2026-03-05`. A delivery created by this user must be dated 5 March, which is the date the user sees.
- Current moment: `2026-03-04 20:00:00`.
- A moment stored as `2026-03-04 20:00:00` is displayed to this user as `2026-03-05 09:00:00` and to a reader in coordinated universal time as `2026-03-04 20:00:00`.

### 6.5 Period boundaries

| Granularity | Start | End |
|---|---|---|
| Year | 1 January of the year | 31 December of the year |
| Quarter | 1 January, 1 April, 1 July or 1 October | 31 March, 30 June, 30 September or 31 December |
| Month | the first day of the month | the last day of the month |
| Week | the Monday of the week | the Sunday of the week |
| Day | the date itself | the date itself |
| Hour (moments only) | the moment with minutes and seconds set to zero | the moment with minutes and seconds set to their maximum |

For a moment, the start of a period is that date at 00:00:00 and the end is that date at the last moment of the day. Note the deliberate difference between two week notions:

1. **Period boundaries always use Monday as the first day of the week**, whatever the reader's language says. Every computation that slices time into weeks (forecasts, schedules, recurrence, aggregation by week) is anchored on Monday.
2. **The first day of the week shown in a calendar** comes from the language record of the reader (a setting with the values Monday through Sunday, default Sunday). It is a display setting only.

A week number is always the week number of the week-based calendar, in which week 1 is the week containing the first Thursday of the year.

### 6.6 Fiscal years

A fiscal year is defined by the day and the month on which it ends (default 31 December). The range containing a given date is computed as follows:

1. Build the candidate end date: the given date's year, with the fiscal end month and the fiscal end day, where a day beyond the length of that month is reduced to the last day of the month, and where a fiscal end day of 28 or of the last day of February is always taken as the last day of February (which makes a February-ending fiscal year land on 29 February in a leap year).
2. When the given date is on or before the candidate end date, the range ends on that candidate, and starts on the day after the equivalent end date one year earlier.
3. Otherwise the range starts on the day after the candidate end date, and ends on the equivalent end date one year later.

**Worked examples**, fiscal year ending 30 June:

| Given date | Range start | Range end |
|---|---|---|
| 2026-03-05 | 2025-07-01 | 2026-06-30 |
| 2026-07-01 | 2026-07-01 | 2027-06-30 |
| 2026-06-30 | 2025-07-01 | 2026-06-30 |

Fiscal year ending 28 February: for a date in 2028 (a leap year), the range ends on 2028-02-29.

### 6.7 Grouping granularities

A query may group a date or moment column by an interval or by a number:

| Interval granularity | Result |
|---|---|
| Hour, day, week, month, quarter, year | The truncated period start, with the interval length attached, in the reader's zone for moments. |

| Number granularity | Result | Range |
|---|---|---|
| Year number | the year | unbounded |
| Quarter number | the quarter of the year | 1 to 4 |
| Month number | the month | 1 to 12 |
| Week number | the week number of the week-based calendar | 1 to 53 |
| Day of year | the ordinal day | 1 to 366 |
| Day of month | the day | 1 to 31 |
| Day of week | the day of the week | 0 for Sunday to 6 for Saturday |
| Hour number | the hour | 0 to 23 |
| Minute number | the minute | 0 to 59 |
| Second number | the second | 0 to 59 |

For a date column, the hour, minute and second numbers are always zero. An unsupported granularity name raises "Error when processing the granularity <name> is not supported. Only <list> are supported".

### 6.8 Durations expressed as numbers

Durations that users enter as hours are stored as decimals, not as moments. The conversions are:

1. A decimal number of hours becomes a time of day by taking the whole part as the hour and rounding sixty times the fraction to the nearest whole minute; when rounding carries the minutes to sixty, the hour is increased by one and the minutes set to zero; a result of exactly 24 hours and 0 minutes becomes the last moment of the day.
2. A time of day becomes a decimal number of hours as `hours + minutes ÷ 60 + seconds ÷ 3600`.

**Worked example.** 16.9959 hours is 16 hours and 59.754 minutes; the minutes round to 60, and the result is therefore 17:00. 7.5 hours is 07:30. 

## 7. Names, display names and finding a record by name

### 7.1 The record name field

Every entity may declare one field as its **record name field**. The convention is a field called `name`; an entity may name another field instead (a code, a reference, a computed complete name), and an entity may declare none.

### 7.2 The display name

Every entity carries a derived, never stored, single-line text field called `display_name`. It is what a link shows, what a list shows in its reference column, what an export writes for a link column, and what a search by name matches against.

Default rule:

1. When the entity declares a record name field, the display name is the value of that field converted for display:

| Type of the record name field | Conversion |
|---|---|
| single-line text, long text | the value itself |
| selection | the label of the selected value, in the reader's language |
| date | the canonical date text, `2026-03-05` |
| date and time | the canonical moment text, converted to the reader's zone |
| link to one record | the display name of the target |
| number | the number as text |

2. When the entity declares no record name field, the display name is the entity name, a comma, and the identifier, for example `account.move,42`.
3. When the record name field is translatable, the display name depends on the reader's language, and the dependency is declared, in order that a cached display name is invalidated when the language changes.
4. An entity may replace the rule with its own. The replacements are specified in each entity's section of its domain folder. The recurring patterns are:

| Pattern | Example |
|---|---|
| The full path of a hierarchy, ancestors first, joined by a slash with spaces | A Contact Tag named "Gold" under "Customer" displays as `Customer / Gold`. |
| A composition of two fields | A Contact that is a person attached to a company displays as `<company name>, <contact name>`. |
| A code and a label | An Account displays as `<code> <name>`. |
| A reference and a partner | A Transfer displays as `<reference> - <partner name>`. |
| The entity plus the identifier as a fallback | Records of technical entities with no name. |

### 7.3 Finding a record by name

The "search by name" operation takes a text pattern, an optional additional condition, an operator (default "contains, case-insensitive") and a limit (default 100 records), and returns pairs of identifier and display name, in the entity's default order.

The pattern is matched against the **name search fields** of the entity: the ordered list declared by the entity, or the record name field alone when no list is declared. Rules:

1. A name search field may be a path through links, for example "the name of the partner of this record".
2. For each search field, when the field is a link, the condition applies to the display name of the target; otherwise the condition applies to the field itself, with the value converted to the field's type when the operator is not a text operator. A value that cannot be converted to the field's type is skipped for that field.
3. The conditions of the individual fields are combined with a logical OR for a positive operator, and with a logical AND for a negative operator ("does not contain", "is not"), which is what makes "not in" behave as "in none of the searched fields".
4. An empty pattern with a "contains" operator matches everything (the condition is dropped), and matches nothing for a negative operator.
5. When the entity declares neither a record name field nor name search fields, the search does not restrict anything and a warning is logged.
6. The returned display names are computed with elevated rights, because the visibility of the label of a referenced record follows the reading record, not the target.

### 7.4 Creating a record from a name alone

"Create from name" takes one text and creates a record with the record name field set to that text, applying every default value and every validation of a normal creation, and returns the identifier and the display name. When the entity declares no record name field the operation does nothing and logs a warning. This is the operation behind "create <typed text>" offers in link fields, and behind the option of an import to create missing targets from their name (section 7.5 of [`data-loading-and-exchange.md`](data-loading-and-exchange.md)).

### 7.5 Default ordering

1. Every entity declares a default order, a list of field names each optionally followed by a direction and a null placement. The default, when nothing is declared, is `id` ascending.
2. Only stored columns, and links whose target has a stored order, may appear in an order. An order that names something else is refused with "Invalid \"order\" specified (<order>). A valid \"order\" specification is a comma-separated list of valid field names (optionally followed by asc/desc for the direction)".
3. Ordering by a link sorts by the default order of the target entity, joined into the query.
4. Ordering by the display name is possible only when the record name field is stored; otherwise the order falls back to the identifier.
5. The identifier is appended as the final tie-breaker of every order, in order that paging is stable.
6. Text ordering is case-insensitive and accent-insensitive when the store offers the accent-insensitive function; otherwise it is the store's collation order. A replacement must sort `École` next to `Ecole`, not after `Zulu`.

## 8. Text and rich text

### 8.1 Single-line text

| Rule | Behavior |
|---|---|
| Maximum length | When the field declares one, a longer value is **truncated** on write, not refused. |
| Trimming | Leading and trailing whitespace is removed by the client before sending, and by the import service before writing, in order that a value typed in a form and the same value imported from a file are stored identically. A field may switch trimming off; the sequence prefix and suffix and the number separators of a language switch it off, because their whitespace is meaningful. |
| Empty | An empty text is stored as an empty column, and reads back as the empty text. |
| Newlines | Accepted but meaningless; single-line text is a storage shape, not a validation. |

### 8.2 Long text

Unbounded, no truncation, no trimming, stored as given.

### 8.3 Rich text

A rich text value is markup. It is **sanitized on write** and stored sanitized; reading returns the stored markup unchanged and marks it as safe for rendering.

Sanitization options, per field:

| Option | Default | Effect |
|---|---|---|
| Sanitize | true | Whether any cleaning happens at all. |
| Sanitize elements | true | Only known elements survive. The block list is removed with its content: `base`, `embed`, `frame`, `head`, `iframe`, `link`, `meta`, `noscript`, `object`, `script`, `style`, `title`. The elements `html` and `body` are unwrapped, their content is kept. The allowed set is the standard set of text markup elements plus the semantic elements `article`, `bdi`, `section`, `header`, `footer`, `hgroup`, `nav`, `aside`, `figure`, `main`, plus comments. |
| Sanitize attributes | true | Only attributes of the safe list survive: the standard safe attribute set, plus `style`, plus the editor and quote-detection data attributes the platform itself writes. When attribute sanitization is off, every attribute is kept, and only classes may be stripped. |
| Sanitize styles | false | When true, style declarations are parsed and only known properties survive. When false, style attributes pass through. |
| Sanitize forms | true | Form elements are removed. |
| Sanitize conditional comments | true | Conditional comments are removed. When false they are kept and their content is sanitized. |
| Strip styles | false | When true, style attributes and style elements are removed entirely rather than sanitized. |
| Strip classes | false | When true, class attributes are removed. |
| Output form | markup | Whether the result is written as markup or as strict markup (self-closing elements, strict nesting), used for outgoing electronic mail. |
| Overridable | false | See 8.5. |

A shortcut exists for outgoing electronic mail: it enables sanitization, disables element sanitization, disables attribute sanitization, keeps conditional comments and writes strict markup, because a mail client needs the markup that the composer produced.

### 8.4 Normalization, which always happens before sanitization

1. Encoding attributes inside tags are removed.
2. Malformed comment terminators are repaired, and empty comments are normalized.
3. Word-processor namespace elements are removed.
4. The document is parsed as a tag soup and made well formed.
5. Quoted passages (the part of a reply that repeats the previous message) are detected and marked with data attributes before any cleaning.
6. The result is serialized with the chosen output form; a wrapping element added only by the parsing is removed again.
7. Non-breaking spaces are written as named entities, in order that the stored text matches what a rendering produces.
8. A value that the parser rejects is stored as `<p>ParserError when sanitizing</p>`; any other failure stores `<p>Unknown error when sanitizing</p>`. Both are logged. Sanitization never raises to the caller by default.

### 8.5 Overridable sanitization

A rich text field may be declared overridable. Then:

1. A user of the sanitization override group writes the value unchanged, with no cleaning at all.
2. Any other user's write is refused when the value already stored contains content that sanitization would remove, because saving would silently destroy content that a privileged user put there:

> The field value you're saving (<entity label> <field label>) includes content that is restricted for security reasons. It is possible that someone with higher privileges previously modified it, and you are therefore not able to modify it yourself while preserving the content.

3. The comparison is made between the normalized stored value and the sanitized stored value; the difference is logged as a line-by-line difference.

### 8.6 Emptiness of rich text

A rich text value counts as empty when, after removing the formatting elements `p`, `div`, `section`, `span`, `br`, `b`, `i`, `font` with their attributes, and after unescaping entities, nothing but whitespace remains, and no icon element is present. This is what makes a value such as `<p style="margin: 0"><br></p>`, which editors produce for an empty box, count as empty in conditions and in reports.

### 8.7 Conversions

| Conversion | Rule |
|---|---|
| Rich text to plain text | Elements are removed, block elements become line breaks, list items become dashes, links become their text followed by the target in square brackets, entities are unescaped. |
| Plain text to rich text | The text is escaped, line breaks become break elements, and the whole is wrapped in a paragraph element. |

## 9. Translations

### 9.1 What is translatable

Three different things are translated, by three different mechanisms:

| Kind | Stored where | Resolved when |
|---|---|---|
| Field values marked translatable (names, labels, descriptions, terms inside rich text) | in the record's own column, as a document keyed by language code | at every read |
| Static text of the application (labels, messages, help) | in the code translation tables of each capability package | at every render |
| Content of shipped records that is not a translatable field | not translated | |

This section specifies the first kind. The other two are specified in [`../runtime/translation.md`](../runtime/translation.md).

### 9.2 Storage

A translatable text, long text or rich text column holds a document keyed by language code (section 4.1 of [`physical-data-catalog.md`](physical-data-catalog.md)). Two modes exist:

| Mode | What a language entry holds | Used for |
|---|---|---|
| Whole-value translation | the complete value in that language | names, labels, short descriptions |
| Term-by-term translation | the complete value in that language, whose markup structure is identical to the base value and whose text terms are the translated terms | rich text, page content, mail templates |

In term-by-term mode, the mapping from a base term to its translations is rebuilt by walking the base value and the translated value in parallel: when the two carry the same number of terms, the terms are paired in order; when they do not, every base term maps to itself, and the translation is considered stale.

### 9.3 Reading

The language of a read is the language of the execution context, or the base language when none is set. The fallback chain is:

| Requested language | Chain |
|---|---|
| the base language | the base language only |
| any other language `L` | `L`, then the base language |
| a pending variant `_L` (a translation being prepared) | `_L`, then `L`, then the pending base, then the base |

The chain applies identically to reading a value, to filtering on it, and to sorting by it: a condition is evaluated on the first non-empty entry of the chain.

### 9.4 Writing

1. Creating a record writes the value under the base language code and under the writer's language code.
2. Writing a translatable field in language `L` writes the entry of `L` only, and leaves every other entry untouched.
3. Writing the field in the base language rewrites the base entry and, in term-by-term mode, updates every other language by replacing each base term with its known translation, dropping the translations of terms that disappeared.
4. When the base language is not installed, every write also writes the base entry, in order that the fallback always finds a value.
5. Writing through the dedicated translation operation takes a language and a map of terms and updates only the named terms of that language.

### 9.5 Consequences to reproduce

1. Two users in two languages reading the same record see two different values and the same identifier.
2. A uniqueness constraint on a translatable column applies to the stored document, not to the resolved value: two records may hold the same French value with different base values without violating a uniqueness rule on the base value.
3. A stored derived field that copies a translatable value cannot be correct in every language at once; the platform warns at definition time and the value follows the language of the writer at the moment of the computation.
4. Exporting a translatable field exports the value in the language of the export.
5. A search that must be language independent searches the base entry explicitly.

## 10. Structured data and user-defined fields

### 10.1 Structured data

A structured data field holds one document of keys, values, lists and numbers, written and read as a whole. Rules:

1. The document is stored as given; no schema is enforced by the storage layer.
2. Numbers are written with the shortest text that reads back as the same number, in order that a value written and read compares equal.
3. Filtering on a structured data column is limited to key access; the platform does not index the content.
4. A structured data field is not translatable and not company-dependent.

### 10.2 User-defined fields

A user-defined field (a "property") is a field whose *definition* lives on a parent record and whose *value* lives on the child record. It exists in order to let users add fields without changing the schema.

| Element | Where it lives |
|---|---|
| The list of definitions | a definition column on the parent record (for example on a project, a payroll structure, an event) |
| The values | a value column on each child record (for example on a task, a payslip, a registration) |

A definition carries: an internal name (a generated alphanumeric key of at most 512 characters), a type, a label, an optional default, and, depending on the type, the target entity, the closed list of values or the list of tags with their colors. The allowed types are: boolean, integer, decimal, single-line text, long text, rich text, date, date and time, monetary, link to one record, link to many records, selection, tags, and separator (a heading with no value).

Rules:

1. Reading a user-defined field merges the definitions from the parent with the values from the child, in order that the reader receives complete field descriptions.
2. Writing a value writes only the value, keyed by the internal name.
3. Changing the parent record of a child changes the set of definitions the child is read against; values whose definition no longer exists remain stored until the record is cleaned, and are then removed.
4. A link value whose target record was deleted reads as empty and is removed at the next cleaning.
5. A rich text value is sanitized with the same options as a rich text field, with style sanitization off and form sanitization on.
6. The value column is not copied when a record is duplicated unless the duplication explicitly carries it, and it is written after the parent link, in order that the definitions are known.
7. Filtering on a user-defined field is possible by internal name, and the filter resolves against the value document.

## 11. Binary content and images

### 11.1 Where the bytes live

A binary field stores its content in an attachment record (the default) or, for seven fields, directly in a byte-string column. An attachment holds: the owning entity name, the owning record identifier, the field name, the file name, the content type, the size, the checksum of the content, and either the bytes or a reference into the file store. The file store keys content by its checksum, which means two identical files are stored once.

### 11.2 Rules

1. Values are exchanged encoded as text, not as raw bytes.
2. Reading a binary field with the "sizes instead of content" option returns a human-readable size such as `12.30 Kb` instead of the content, per field.
3. Deleting a record deletes the attachments that hold its binary fields.
4. Duplicating a record duplicates the attachments of the binary fields that are copied.
5. A binary field is never prefetched with the rest of the record; it is read on demand.
6. Uploading a scalable vector image is refused unless the acting user is an administrator: "Only admins can upload SVG files."
7. A value that is not representable as text and not valid encoded content is refused: "ASCII characters are required for <value> in <field>".

### 11.3 Images

An image field is a binary field with a maximum width, a maximum height and a resolution check:

1. On write, an image larger than the maximum width or height is resized, keeping its aspect ratio, before storage.
2. An image whose total resolution exceeds fifty million pixels is refused when the resolution check is on, in order that a decompression bomb cannot be stored.
3. The standard image behavior attached to entities that carry pictures stores one original at most 1920 by 1920 pixels, and four derived stored copies at most 1024, 512, 256 and 128 pixels, each a resized copy of the original. Writing the original rewrites the four copies; writing a copy directly is refused.
4. An entity that carries image fields must keep the audit columns, because the resized copies are invalidated using the last-update timestamp.

## 12. The active flag and archiving

### 12.1 The flag

An entity supports archiving when it has a boolean field named `active` (or, for an entity extended with user-defined fields, `x_active`). The flag defaults to true. Archiving a record sets it to false; unarchiving sets it back to true. No record is ever deleted by archiving, and no data is lost.

### 12.2 The visibility rule

Every search on an archiving entity adds the condition `active = true`, unless one of the following holds:

1. The condition given by the caller already mentions the active field, at any depth. Asking for `active = false` returns the archived records; asking for `active in (true, false)` returns both.
2. The execution context carries "include archived records".
3. The caller explicitly asks for archived records to be included.

The rule applies to searches, to counts, to grouped reads and to the "search by name" operation. It does **not** apply to:

| Operation | Behavior |
|---|---|
| Reading a record by identifier | An archived record reads normally. A link to an archived record resolves and displays normally. |
| Reading a list relation or a many-to-many relation | The target records are read with archived records **included**, in order that the list of lines of a document does not silently shrink when a line's target is archived. |
| Company consistency checks | Evaluated with archived records included. |
| Access rules | Evaluated with archived records included, then the visibility rule is applied. |
| Aggregations behind a stored derived field | Follow the rule of the operation that computes them, which is stated per field. |

### 12.3 What archiving does and does not do

1. Archiving does not cascade. Archiving a parent leaves its children active, unless the entity declares otherwise; every such cascade is specified in the entity's own section.
2. Archiving does not break links. A required link to an archived record stays valid, and the record still displays.
3. Archiving does not release uniqueness. A unique constraint that does not mention the active flag still counts archived rows. A uniqueness rule that must ignore archived rows is declared as a unique index restricted to active rows.
4. Archiving is the recommended answer to a refused deletion. The message shown when a deletion is refused by a reference states it: "Another model is using the record you are trying to delete.\n\nThe troublemaker is: <entity>\nThanks to the following constraint: <constraint>\nHow about archiving the record instead?"
5. An entity may refuse to archive a record in a state that forbids it; those guards are specified per entity.

### 12.4 Worked example

A product is archived while it is on three draft sales orders:

| Question | Answer |
|---|---|
| Does the product appear in the product list? | No: the default search hides it. |
| Do the three order lines still show the product? | Yes: the lines read their link and display the product. |
| Can a user add a new line with that product? | No: the field's list of selectable records is a search, which hides archived records. |
| Can the orders be confirmed? | Yes, unless the domain's own rules forbid it; the link is valid. |
| Does the product still count in a grouped report by product? | Only if the report's search includes archived records. |

## 13. Audit fields, tracking and concurrency

### 13.1 The four audit fields

Specified in section 1.2 of [`physical-data-catalog.md`](physical-data-catalog.md). The behavioral rules:

1. They are written by the store, never by a caller. A value supplied for them is removed silently.
2. `create_date` and `write_date` carry the **transaction** timestamp, not the wall clock at the moment of the statement: every record created in one transaction shares one creation timestamp, and two records written by one operation share one update timestamp.
3. `write_date` and `write_uid` are refreshed by every write, including a write performed only to store a recomputed derived value. A record whose visible values did not change can therefore have a fresh update timestamp.
4. Ten entities carry no audit fields at all; they are listed in [`physical-data-catalog.md`](physical-data-catalog.md), section 12.
5. The update timestamp is the invalidation key of derived stored images (resized pictures, cached renderings) and of the transient record cleanup; a replacement must keep it accurate.

### 13.2 Tracked fields

A field may be declared **tracked**. When such a field changes on a record whose entity carries a discussion thread, the change is recorded as a tracking entry attached to a message in the thread: the field, its label, its old value and its new value, with the type-appropriate rendering (a link tracks the display name of the target, a selection tracks the label).

1. Tracking happens on write, after the validation rules pass and after the derived fields have been recomputed, in order that a derived tracked field is tracked with its final value.
2. Several tracked fields changed in one write produce one message with several tracking entries.
3. A creation does not produce tracking entries; it produces the creation message of the thread.
4. The message that carries tracking entries is posted with the subtype the entity declares for that field, which controls who is notified.
5. Tracking is a business-visible audit trail: it is what a user sees in the record's thread. It is not the technical log.

The complete tracking mechanism, the subtypes and the notification rules belong to the messaging domain: [`../domains/messaging-and-activities/workflows.md`](../domains/messaging-and-activities/workflows.md).

### 13.3 Concurrency

1. There is no version column and no optimistic-lock check on ordinary writes: two transactions writing different fields of one record both succeed; two transactions writing the same field leave the value of the last one to commit.
2. Isolation is serializable, and a transaction that the store aborts because of a conflict is retried from the beginning; the retry rules are in [`../runtime/transactions-and-concurrency.md`](../runtime/transactions-and-concurrency.md).
3. Where a business rule needs stronger protection, the entity takes an explicit lock (document numbering, section 4.4) or relies on a uniqueness constraint to serialize the competing writes.
4. The update timestamp may be used by a client to detect that a record changed under it; it is not enforced by the store.

## 14. Hierarchy paths

### 14.1 Parent and children

An entity forms a hierarchy when it has a link to itself that is declared as its parent field. The default name of that field is `parent_id`; an entity may name another (for example a location names its parent location field, a package names its parent package field, a unit of measure names its reference unit field).

### 14.2 The materialized ancestor path

Eleven entities maintain a materialized path column, `parent_path`, in order that subtree queries are a single indexed prefix match instead of a recursive walk:

| Entity | Transport name | Parent field |
|---|---|---|
| Company | `res.company` | `parent_id` |
| Contact Tag | `res.partner.category` | `parent_id` |
| Menu Item | `ir.ui.menu` | `parent_id` |
| Analytic Plan | `account.analytic.plan` | `parent_id` |
| Department | `hr.department` | `parent_id` |
| Website Product Category | `product.public.category` | `parent_id` |
| Unit of Measure | `uom.uom` | `relative_uom_id` |
| Product Category | `product.category` | `parent_id` |
| Website Menu | `website.menu` | `parent_id` |
| Location | `stock.location` | `location_id` |
| Package | `stock.package` | `parent_package_id` |

**Format.** The path is the list of identifiers from the root to the record itself, each followed by a slash. A root record with identifier 42 has the path `42/`; its child 63 has `42/63/`; the children 84 and 85 of 63 have `42/63/84/` and `42/63/85/`.

The trailing slash is required: without it, `42/63` would be a prefix of `42/630`, and a subtree query would return unrelated records.

**Maintenance.**

1. On creation, the path is the parent's path followed by the new identifier and a slash; a record without a parent gets its own identifier and a slash.
2. On a write that changes the parent, the records whose parent actually changes are collected **before** the parent column is updated, then their paths and the paths of all their descendants are rewritten in one statement: the old prefix is replaced by the new parent's path. Descendants are selected by the range `path >= node path` and `path < node path with the final slash replaced by the next character`, which is the indexed form of "starts with the node path".
3. When the new parent is inside the subtree being moved, the move is refused with "Recursion Detected."
4. When the column is created for the first time (a fresh installation, or an entity that starts maintaining paths), every path is computed at once by walking the hierarchy from the roots.
5. The path column is never writable by a caller; a supplied value is removed silently.

### 14.3 Subtree and ancestor queries

Two operators exist on any hierarchical entity:

| Operator | Meaning | With a materialized path | Without one |
|---|---|---|---|
| "is a descendant of" | the record, and every record under it | one condition per given record: `parent_path` starts with the path of that record | repeated searches by parent, level by level, until no new record is found |
| "is an ancestor of" | the record, and every record above it | split the path of each given record on the slash and take the identifiers | walk the parent link upwards, collecting identifiers, until the root |

Both operators include the given records themselves. Both are evaluated with elevated rights and with archived records included; the access rules and the visibility rule are then applied by the enclosing query, which means a user sees only the part of the subtree they may read.

**Worked example.** A product category tree: `All` (identifier 1, path `1/`), `Goods` (identifier 4, path `1/4/`), `Furniture` (identifier 17, path `1/4/17/`), `Services` (identifier 5, path `1/5/`). A condition "category is a descendant of Goods" becomes `parent_path` starts with `1/4/`, which matches `Goods` and `Furniture` and not `Services`. A condition "category is an ancestor of Furniture" yields the identifiers 1, 4 and 17.

### 14.4 Loop prevention

Every hierarchical entity forbids loops. The check is a single recursive reachability query from the changed records over the parent link (or over the association table, for a many-to-many hierarchy), which stops as soon as a record reaches itself. A loop raises a validation error whose message names the entity, for example "You can not create recursive tags." for Contact Tag, "You cannot create recursive categories." for Product Category, "You cannot create recursive Partner hierarchies." for Contact, and "Error! You cannot create recursive menus." for Menu Item. The check is run by the entity's validation rule on the parent field; it is not enforced by the store, and it takes no exclusive lock, which means two concurrent transactions can, in principle, create a loop between them; the periodic consistency checks report such a case.

## 15. Company-dependent values

### 15.1 What they are

A company-dependent field holds a different value per company on a single record. It is used where master data is shared but its handling differs per company: the payable account of a supplier, the customer payment terms of a partner, the fiscal position of a partner, the barcode of a contact.

### 15.2 Rules

1. Storage is one document per column, keyed by the company identifier written as text (section 4.2 of [`physical-data-catalog.md`](physical-data-catalog.md)).
2. Reading returns the entry of the **active company**. When it is absent, the value falls back to the configured default for that entity, field and company, and when there is none, to the empty value of the type.
3. Writing writes the entry of the active company only. Reading the same field as another company still returns that company's own value.
4. The allowed types are: boolean, integer, decimal, monetary, single-line text, long text, rich text, date, date and time, selection, link to one record. Lists of records may not be company-dependent.
5. A company-dependent field may not be required and may not be translatable.
6. It is not copied when a record is duplicated, unless the duplication states otherwise.
7. It is indexed, by default with a partial index on the rows that hold any entry at all.
8. Its value depends on the active company, which is declared as a context dependency; a cached value is therefore keyed by company.
9. A link that is company-dependent carries no foreign key. Deleting the target leaves entries pointing at a missing record; the value then reads as empty.

### 15.3 The configured default

Defaults are configuration records with: the field, an optional user, an optional company, an optional condition, and the value. Resolution of the fallback for a company-dependent field reads the defaults of the entity, with elevated rights, for the active company, and takes the value of the matching entry. Creating, changing or deleting a default clears every cached company-dependent value, because the fallback of any record may have changed.

### 15.4 Company consistency for company-dependent links

The consistency check treats the two kinds of link differently:

| Kind of link | Rule checked |
|---|---|
| Ordinary link marked for company checking | The company of the target must be empty, or equal to the company of the record (or one of the companies of the record, for an entity that carries several). |
| Company-dependent link marked for company checking | The company of the target must be empty, or equal to the **active company**, which is the company whose entry is being written. |

A violation raises the company inconsistency error, which lists up to five offending records:

> Uh-oh! You’ve got some company inconsistencies here:
> - “<record>” belongs to company “<company>” while “<field label>” (<field>: <values>) belongs to another company.

### 15.5 Worked example

A supplier record is shared by two companies. The company-dependent field "supplier payment terms" holds `{"1": 30 days, "3": 60 days}`.

| Reader | Active company | Value read |
|---|---|---|
| A user of company 1 | 1 | 30 days |
| A user of company 3 | 3 | 60 days |
| A user of company 2 | 2 | The configured default for company 2 if one exists, otherwise no value. |
| A user of company 2 who writes "15 days" | 2 | The document becomes `{"1": 30 days, "3": 60 days, "2": 15 days}`; the entries of companies 1 and 3 are untouched. |

A search for "suppliers whose payment terms are 60 days" run by a user of company 3 matches this record; the same search run by a user of company 1 does not.

## 16. Uniqueness and check rules

### 16.1 Two enforcement levels

| Level | Declared as | Evaluated | Sees |
|---|---|---|---|
| Store level | A table constraint, a unique index or a check expression on the entity's table | By the store, at the end of the statement or of the transaction | Columns of one row (a check), or all rows (a uniqueness rule) |
| Entity level | A validation rule attached to a list of fields | By the platform, after every creation and after every write that touches one of those fields, and after the derived stored fields have been recomputed | Any data reachable from the record, including related records and aggregations |

A rule that can be expressed on columns of one table belongs at store level, because it also protects against writes that bypass the entity's operations (bulk loads, data files, concurrent transactions). A rule that needs to read other records belongs at entity level.

### 16.2 Conventions for uniqueness

| Pattern | Declared as | Example |
|---|---|---|
| A code that must be unique across the installation | a table constraint over one column | the currency code: "The currency code must be unique!" |
| A code that must be unique per company | a table constraint over the column and the company column | the transfer reference: "Reference must be unique per company!" |
| A number that must be unique only in one state | a unique index over the columns, restricted to the rows in that state | the journal entry number, unique per journal among posted entries whose number is not the placeholder: "Another entry with the same name already exists." |
| A pair that must be unique when one member is present | a unique index over the pair, restricted to the rows where the member is present | one conversation history per participant, per channel |
| A value that must be unique among active rows only | a unique index restricted to active rows | one active interchange registration per company and mode |
| A generated key that must be globally unique | a unique index over the generated column | the external identifier registry: the pair prefix and name |

Two consequences:

1. A uniqueness rule that does not mention the active flag counts archived rows. Archiving a record therefore does not free its code; this is deliberate for codes that must stay traceable, and the entities that want the opposite declare the restricted form.
2. A uniqueness rule on a translatable column applies to the stored document, not to the value in any one language (section 9.5).

### 16.3 Conventions for check rules

| Pattern | Example |
|---|---|
| A quantity or amount must not be negative | "The rounding factor must be greater than 0!" |
| Two columns are mutually exclusive | a history row is attached either to a partner or to a guest, never both: "History should either be linked to a partner or a guest but not both" |
| A column is required only in one configuration | a resource of type "web address" must carry a link: "A resource of type web address must contain a link." |
| A text may not contain a character | an external identifier may not contain a space: "External IDs cannot contain spaces" |
| A date range must be ordered | the end date must not precede the start date |

Every declared check carries a message. The complete catalog of declared constraints, with their definitions in canonical field names and their messages, is in [`physical-data-catalog.md`](physical-data-catalog.md), section 11.2.

### 16.4 Entity-level validation rules

1. A validation rule declares the fields it depends on. It runs after a creation, and after a write that touches at least one of those fields.
2. It runs with elevated rights, exactly as a derived stored field is computed, in order that a rule may read records the acting user cannot.
3. It is evaluated on the whole set of records written by the operation, and it must report the first violation with a message that names the offending record.
4. Raising a validation error aborts the entire operation and rolls back every change of the statement, including the changes to other records of the same write.
5. A rule may be excluded for one operation by naming it in the exclusion list of that operation; this is how a record can be created in a temporarily inconsistent state and completed in the same transaction.

The complete list of validation rules per entity is in the entity catalogs and in each domain's `business-rules.md`.

## 17. Deletion behavior

### 17.1 Per relation kind

| Relation | Declared behavior | Effect when the target record is deleted |
|---|---|---|
| Link to one record, `restrict` | the default for a required link between persistent entities | The deletion is refused while any source row points at the target. |
| Link to one record, `set null` | the default for an optional link | The column of the source row is cleared; the source row survives. |
| Link to one record, `cascade` | declared explicitly; the default for a required link from an assistant entity | The source row is deleted with the target. |
| List of records (the inverse of a link) | not declared here; it follows the behavior of the link column on the target entity | Deleting the owner deletes the lines when their link declares `cascade`, and orphans them when it declares `set null`, and is refused when it declares `restrict`. |
| Many-to-many | `cascade` (the default) or `restrict` | `cascade` removes the association rows; `restrict` refuses the deletion of the target while an association row exists. The row of the declaring side is always removed with its record. |
| Polymorphic link | not enforceable | Nothing happens; the pair becomes dangling and reads as empty. The owning operations clean up explicitly. |
| Company-dependent link | `set null` or `restrict`, enforced by the platform, not by the store | `set null` removes the entries that point at the deleted record from every document; `restrict` refuses the deletion with "You cannot delete <record>, as it is used by <record>". |

### 17.2 The deletion procedure

1. Check that the acting user may delete the records, both at entity level and through the record rules.
2. Run the declared deletion guards of the entity. A guard refuses the deletion with its own message (for example, a posted accounting entry may not be deleted). Guards that are declared as applying only outside package removal are skipped while a capability package is being uninstalled.
3. Discard the pending recomputations that target the records.
4. Flush every pending write, in order that the store sees a consistent state.
5. Mark the values that depend on the records as needing recomputation **before** the rows disappear (a total that sums the lines being deleted must be recomputed after the deletion).
6. Delete the rows, in batches.
7. Delete the external identifier entries that name the deleted rows, because they carry no foreign key.
8. Delete the attachments whose owner is a deleted row, found by the pair (entity name, identifier), which also removes the stored binary field content.
9. Refuse the deletion if a deleted record is the configured default value of a company-dependent link: "Unable to delete <record> because it is used as the default value of <field>".
10. Apply the company-dependent link behavior of 17.1 to every document that references a deleted record.
11. Discard the configured default values that hold a deleted record as their value.
12. Invalidate the whole cache, because the store may have cascaded deletions the platform does not know about.
13. Write one audit line naming the acting user, the entity and the deleted identifiers. This line is the only trace a deletion leaves; deleted rows leave no record behind.

### 17.3 Deletion and archiving

Deletion is not the normal end of a business record. The design rule of the whole system is: **records that have been used are archived, not deleted**. A replacement must keep the same balance:

1. Documents that have consequences (posted entries, validated transfers, confirmed orders) refuse deletion in their own guards and offer cancellation and archiving instead.
2. Master data that is referenced refuses deletion through `restrict` and offers archiving.
3. Deletion stays available for drafts, for mistakes made in the same session, and for administrative cleanup.

## 18. Duplicating a record

### 18.1 What is copied

Duplicating a record builds a value map from the original and creates a new record with it. A field is copied when its "copy" flag is true. The defaults of that flag are:

| Field kind | Copied by default |
|---|---|
| Stored, not derived, not a state field | yes |
| A field named `state` | no |
| Derived (computed), not stored | no |
| Derived and stored, and not writable | no |
| Derived and stored, and explicitly writable | yes |
| Copied from another record through a link (related) | no |
| Company-dependent | no |
| List of records (one-to-many) | no by default; the entities that own their lines set it to true, and then the lines are duplicated, in the order of their identifiers, and attached to the copy |
| Many-to-many | yes: the copy links to the same targets |
| User-defined field values | no |
| The audit fields, the identifier and the ancestor path | never |

### 18.2 Rules

1. The values supplied by the caller override the copied values, field by field.
2. A field given by the caller for a delegated parent record is excluded from the copy of the child, in order that the parent is not duplicated twice.
3. A circular chain of records being copied is cut: a record already visited in the same duplication is not duplicated again.
4. The copy is created through the normal creation path: defaults apply to the fields that are not copied, validation rules run, derived values are computed, and the discussion thread starts empty.
5. Translations of translatable fields are copied entry by entry, for every language, after the copy is created.
6. An external identifier is not copied; the copy gets a fresh entry only if something creates one (section 3.5).
7. The display name of a copy is not decorated automatically. Entities whose name must stay unique override the duplication to append a suffix, most commonly "(copy)", and that override is specified in the entity's own section.

## 19. Acceptance criteria

Each criterion is independently verifiable against a replacement.

**Identifiers**

1. Given an entity with records, when a record is created and the transaction is rolled back, and another record is created afterwards, then the second record's identifier is strictly greater than the first's, and the first number is never reused.
2. Given a create whose value map contains `identifier`, `create_date`, `create_uid`, `write_date`, `write_uid` or `parent_path`, when the record is created, then those values are ignored and the store's own values are written.

**External identifiers**

3. Given a shipped record named `base.main_company` whose entry is not marked "not updatable", when the capability package is upgraded with a changed value in its data file, then the record is updated in place and keeps its identifier.
4. Given a shipped record whose entry is marked "not updatable" and whose value a user changed, when the package is upgraded, then the user's value is kept.
5. Given an import file with the external identifier `base.some_record` where `base` is an installed package, when the import runs, then it is refused with the message about the module prefix of section 3.3.

**Document numbering**

6. Given a journal whose last posted entry is `INV/2026/00041` dated 5 March 2026 and a reset period of one year, when an entry dated 7 March 2026 is posted, then its number is `INV/2026/00042`.
7. Given the same journal, when an entry dated 2 January 2027 is posted, then its number is `INV/2027/00001`.
8. Given two concurrent transactions posting into the same journal, when both are committed, then the two entries carry two different consecutive numbers, and neither transaction fails.
9. Given an entry numbered `INV/2026/00042`, when its date is changed to 4 February 2027 and the configured threshold date is earlier than that date, then the write is refused with the date-alignment message of section 4.4.
10. Given a counter sequence with prefix `PAY`, padding 5 and the no-gap implementation, when three numbers are drawn and the second transaction is rolled back, then the numbers drawn afterwards continue from the value the rolled-back transaction released, leaving no gap.

**Rounding**

11. Given a currency with rounding factor 0.01, when the amount 1234.567 is written, then the stored value is 1234.57.
12. Given a currency with rounding factor 1, when the amount 1234.567 is written, then the stored value is 1235.
13. Given a currency with rounding factor 0.05, when the amount 1234.567 is written, then the stored value is 1234.55.
14. Given the value 2.675 and a precision of two decimal places, when it is rounded, then the result is 2.68.
15. Given the values 0.006 and 0.002 at two decimal places, when they are compared, then the comparison reports them as different, and when their difference is tested for zero, then it is reported as zero.

**Dates**

16. Given a user whose time zone is thirteen hours ahead of coordinated universal time, at 09:00 local time on 5 March 2026, when a record with a date field defaulting to "today as seen by the reader" is created, then the stored date is 2026-03-05.
17. Given a moment stored as 2026-03-04 20:00:00, when it is displayed to that user, then it shows 2026-03-05 09:00:00, and when it is displayed to a reader in coordinated universal time, then it shows 2026-03-04 20:00:00.
18. Given a fiscal year ending 30 June and the date 2026-03-05, when the fiscal range is computed, then it is 2025-07-01 to 2026-06-30.
19. Given any date, when the start of its week is computed, then the result is the Monday of that week, regardless of the reader's language setting for the first day of the week.

**Text and translations**

20. Given a translatable name whose stored document is `{"en_US": "Customer Invoice", "fr_FR": "Facture client"}`, when a reader whose language is French reads it, then the value is `Facture client`, and when a reader whose language is German reads it, then the value is `Customer Invoice`.
21. Given the same record, when the French reader writes a new French value, then the base entry is unchanged and only the French entry is replaced.
22. Given a rich text field with default sanitization, when a value containing a script element is written, then the stored value contains no script element and the rest of the markup is preserved.
23. Given a rich text field, when the value `<p style="margin: 0"><br></p>` is written, then a condition testing the field for emptiness reports it as empty.

**Archiving**

24. Given an archived record, when a default search runs, then the record is absent, and when the same search adds a condition on the active flag, then it is present.
25. Given a document line whose target record was archived, when the document is read, then the line still shows the target.

**Hierarchies**

26. Given a category tree with paths `1/`, `1/4/` and `1/4/17/`, when a condition "is a descendant of the record with identifier 4" is evaluated, then it matches the records 4 and 17 and not the record 1.
27. Given a category, when it is moved under one of its own descendants, then the write is refused with the recursion message of the entity.
28. Given a category with descendants, when its parent changes, then the paths of the category and of every descendant are rewritten in one operation, and every path still ends with a slash.

**Company-dependent values**

29. Given a shared record whose company-dependent field holds `{"1": A, "3": B}`, when a user of company 3 reads it, then the value is B, and when a user of company 2 reads it, then the value is the configured default of company 2, or empty.
30. Given the same record, when a user of company 2 writes a value, then the entries of companies 1 and 3 are unchanged.

**Constraints and deletion**

31. Given a uniqueness rule on a code, when a second record is created with the same code, then the write is refused and the message declared with the rule is shown.
32. Given a record referenced by a required link declared `restrict`, when a deletion is attempted, then it is refused with the message that offers archiving instead.
33. Given a record with attachments and an external identifier entry, when it is deleted, then its attachments and its entry are deleted too, and one audit line naming the acting user and the identifiers is written.
34. Given a record that is the configured default value of a company-dependent link, when a deletion is attempted, then it is refused with the message of section 17.2.

**Duplication**

35. Given a record with a state field, a computed stored read-only total, lines and tags, when it is duplicated, then the copy has the default state, its own recomputed total, copies of the lines (if the list is declared copyable) and links to the same tags.
36. Given a record whose translatable name carries three language entries, when it is duplicated, then the copy carries the same three entries.

## 20. Reconciliation notes

This document consolidates two drafts of the same material. The target branch carried no file for this topic, so the body comes from the working branch. The following points were changed, and why.

| Point | Working branch | Resolution |
|---|---|---|
| Identifier spelling | Entities, fields and columns were named by a de-identified paraphrase (for example an identifier column written as `identifier`, audit columns written as `created_on`, `created_by_user`, `last_updated_on`, `last_updated_by_user`, a parent link written as `parent`). | Checked against the source tree and replaced by the contractual names a rebuild must reproduce: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`, `parent_id`, and the per-entity parent links `location_id`, `parent_package_id`, `relative_uom_id`. Rule three of the documentation rules requires identifiers to be reproduced exactly, and the rest of this repository uses the same spellings. |
| Constraint message for the external identifier | "External identifiers cannot contain spaces". | The message declared with the check constraint is "External IDs cannot contain spaces"; the verbatim text is reproduced and the surrounding prose names the concept in full. |
| Access refusal on an external identifier | The message was shown with the identifier inside code font. | The refusal embeds the package part and the local part in double quotation marks; the sentence now says so instead of showing a code-font placeholder that the system never prints. |
| Counter sequence fields | The field list omitted the derived, writable next-number field and did not list the subsequence entity. | Both added from the entity definitions, because the derived field is the one a user edits and a rebuild must offer it. |
| Companion documents | The draft pointed at a single combined document for loading and shipped records. | The material is split here into [`data-loading-and-exchange.md`](data-loading-and-exchange.md) and [`reference-data.md`](reference-data.md), and the references now point at the right one. |
| Translation of static text | The draft pointed at an overview document. | The repository specifies all three kinds of translatable text in [`../runtime/translation.md`](../runtime/translation.md); the reference points there. |
