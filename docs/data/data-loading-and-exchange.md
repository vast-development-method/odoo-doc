# Data loading and exchange

Every way a record enters or leaves the system as data rather than as a screen action: the records a capability package ships and reloads on every update, the external identifiers that make those records addressable and re-writable, the field path notation that lets a flat table describe a graph of records, the generic import operation that turns rows into records, the interactive import session that a user drives, and the export that turns records back into rows.

This document specifies the data contract. The package lifecycle that triggers a load — discovery, dependency order, installation, update, removal — and the full grammar of a declaration document are specified in [`../overview/package-system.md`](../overview/package-system.md); this document restates only what a reader needs in order to understand the data. The identity and value rules the loaded values must satisfy are in [`persistence-identity-and-values.md`](persistence-identity-and-values.md); the tables the values land in are in [`physical-data-catalog.md`](physical-data-catalog.md); the actual record sets a fresh installation contains are enumerated in [`reference-data.md`](reference-data.md); the translation of shipped text is in [`../runtime/translation.md`](../runtime/translation.md).

## 1. Vocabulary

| Term | Meaning |
|---|---|
| Data file | A file shipped by a capability package and executed at installation and at every update of that package. |
| Declaration document | A data file written as a tree of nodes, each node creating, updating, deleting or invoking something. |
| Tabular data file | A data file written as a table of comma-separated values whose first row names the fields. |
| External identifier | A stable textual key of the form package prefix, a dot and a local name, that names a record independently of its numeric identifier. |
| Load mode | Whether the current load is an installation (initial mode) or an update (update mode). |
| Not-updatable | A flag on an external identifier that stops a later update of the owning package from rewriting the record. |
| Field path | A column heading that names a field, optionally followed by a slash and a field of the related entity. |
| Generic import operation | The single operation that turns a field list and a row matrix into records, used by tabular data files, by the interactive import session and by integrations. |
| Import session | An interactive assistant record holding an uploaded file, the chosen options and the chosen column mapping. |
| Test import | An execution of the import that performs every write and then rolls back, so that the user sees the errors without changing anything. |
| Import-compatible export | An export whose columns are field paths and whose values are external identifiers, so that the file can be imported back. |

## 2. External identifiers as the data key

### 2.1 What they are for

An external identifier is a stable, human-chosen name for a record. It has three jobs:

1. It lets a data file refer to a record that it or another package created, across installations and across tenants, so that data files are portable and can be executed again.
2. It records which capability package owns a record, which is what makes removal of that package possible.
3. It gives an import file a key with which to update an existing record instead of creating a second one, and gives an export file a key with which the exported rows can be imported back.

### 2.2 Structure

| Part | Rule |
|---|---|
| Package prefix | The technical name of the capability package that owns the record; or `__import__` for a record created by a user-driven import that supplied no prefix; or `__export__` for a record named by an export that had no identifier yet; or empty, for an identifier that belongs to no package. |
| Local name | A textual key, unique inside the prefix. It may not contain a space. |
| Complete form | The prefix, exactly one dot, and the local name, for example `base.main_company`. The local name may not itself contain a dot. |

### 2.3 The registry entity

External identifiers are records of the entity External Identifier (`ir.model.data`, table `ir_model_data`).

| Field | Full name | Type | Rules |
|---|---|---|---|
| `name` | Name | single-line text, required | The local part of the identifier. A value containing a space is refused by the check constraint `_name_nospaces`, whose message is "External IDs cannot contain spaces". |
| `module` | Package | single-line text, required, default empty | The owning package's technical name, or one of the reserved prefixes. |
| `complete_name` | Complete identifier | single-line text, derived, not stored | The package, a dot and the name. |
| `model` | Entity | single-line text, required | The transport name of the entity the record belongs to. |
| `res_id` | Record | integer, qualified by the entity field | The numeric identifier of the record. |
| `noupdate` | Not updatable | boolean, default false | When true, an update of the owning package leaves the record alone. |
| `reference` | Reference | single-line text, derived, not stored | The entity name, a comma and the record identifier. |

Constraints and indexes: a unique index `module_name_uniq_index` over the pair (`module`, `name`); an index `model_res_id_index` over the pair (`model`, `res_id`), because the reverse lookup — which external identifier names this record — is frequent. The default ordering is `module`, then `model`, then `name`. The display name of an entry is the display name of the target record when it can be read, and the complete identifier otherwise. The entity refuses writes that bypass access control, so that a package cannot silently reassign ownership of another package's records.

### 2.4 Resolution

1. Resolving a complete form returns the pair (entity name, record identifier), or fails with "External ID not found in the system: " followed by the identifier.
2. Resolution is cached, and the cache is maintained as follows. Creating an entry leaves the resolution cache alone, because a complete form that resolves to nothing was never cached. Changing an entry and deleting an entry each clear the resolution cache. Bulk assignment (section 2.5) does not clear it either: it writes the resolved pair straight into the cache for every row it inserted or updated, and, for a row whose creation stamp and modification stamp differ — that is, a row that already existed and really changed — it marks the general cache as invalidated so that the other worker processes drop their copies.
3. Creating, changing or deleting an entry whose entity is the access-group entity (`res.groups`) additionally clears the access-group cache, and so does a bulk assignment in which at least one row names that entity. The access-group cache holds the resolved membership and implication graph, which is read on every permission check; an external identifier that starts, stops or moves pointing at an access group changes what that graph resolves to, so the cache must be dropped with it. A change to an entry of any other entity leaves the access-group cache alone.
4. The convenience resolution used throughout the system additionally reads the record and checks that it still exists. A vanished record yields "No record found for unique ID " followed by the identifier " . It may have been deleted."
5. Resolution does not check access rights. A separate operation resolves the identifier and then verifies that the acting user may read the record; it returns the entity name with no identifier when the record exists but is not readable, or refuses with "Not enough access rights on the external ID" followed by the package part and the local part in double quotation marks when the caller asks for an error.
6. During a load, an identifier written without a dot is completed with the contributing package's name. An identifier whose package part is not the contributing package refers to another package's record; that package must be installed.

### 2.5 Assignment

External identifiers are written in bulk. For each pair of package part and local part the row is inserted; on conflict with an existing row for the same pair, the entity and the record identifier are updated and the modification stamp is refreshed, but **only when the target actually changed**, and, during an update, **only when the row is not marked not updatable**.

Every assignment adds the complete form to the set of identifiers seen during the current build, which is what the orphan cleanup of section 3.4 consumes.

### 2.6 Identifiers for embedded parents

When a record that embeds a parent record — a delegating entity, listed in [`domain-model.md`](domain-model.md), section 5.2 — is created from a data file and the parent is created implicitly, the parent also receives an external identifier: the child's identifier, an underscore, and the transport name of the parent entity with dots replaced by underscores. Without it the implicitly created parent would survive the removal of the package that caused it to exist.

### 2.7 Copying

Duplicating a record that carries an external identifier does not duplicate the identifier. The copy receives a new entry whose local name is the original local name, an underscore and four random hexadecimal characters, so that two copies never collide.

### 2.8 The reserved prefixes

| Prefix | Written by | Meaning |
|---|---|---|
| `__import__` | The generic import operation, when a user-driven import supplies a bare local name | The record is not owned by any package and is never touched by a package update or removal. |
| `__export__` | The export, for every exported record that had no identifier yet | The local name is the table name, an underscore, the record identifier, an underscore and eight random hexadecimal characters. |
| empty | Data written outside a package load | The complete form is the local name alone. |

A user-driven import that supplies a prefix which is the technical name of an installed package is refused, because the record would be deleted the next time that package is updated. The message is "The record " the external identifier " has the module prefix " the prefix ". This is the part before the '.' in the external id. Because the prefix refers to an existing module, the record would be deleted when the module is upgraded. Use either no prefix and no dot or a prefix that isn't an existing module. For example, __import__, resulting in the external id __import__." followed by the local name. The refusal does not apply when the row is marked not updatable.

## 3. Update semantics

### 3.1 The two load modes

| Mode | When | Effect |
|---|---|---|
| Initial | The package is being installed, or reinitialised | Records are created. A record that already exists under the same external identifier is updated. The not-updatable flag does **not** prevent the write. |
| Update | The package is being updated | Records are created when missing and updated when present, **unless** the external identifier is marked not updatable, in which case the record is left untouched. |

The default prefix used when a load supplies no package name is `__import__`, and the default mode when a load supplies none is initial.

### 3.2 Create or update, record by record

For each declared record that carries an external identifier:

1. Look up the identifier.
2. **No row exists.** Create the record and assign the identifier.
3. **A row exists but its recorded entity differs from the entity being written.** Refuse, with a message stating that for the external identifier, while trying to create or update a record of one entity, a record of a different entity was found, naming both entities and the row.
4. **A row exists and the target record still exists.** Register the identifier for reassignment. Update the record's fields, unless the build is an update *and* the identifier is marked not updatable, in which case do nothing.
5. **A row exists but the target record has vanished.** Delete the stale row and create the record afresh.

For a declared record with no external identifier: create it, unless the values carry an explicit numeric identifier, in which case update that record. During an update, a record with neither is refused with "Cannot update a record without specifying its id or xml_id".

### 3.3 The not-updatable flag

| Build | Flag | Result |
|---|---|---|
| Install, initial mode | false | Record created or updated |
| Install, initial mode | true | Record created or updated; the flag does not block initial mode |
| Update | false | Record updated |
| Update | true | Record untouched |

The flag is the mechanism by which a package ships a **starting value that the user may then change**: a default sequence, a sample message template, a suggested account, a chart of accounts. Without it every package update would overwrite the tenant's own configuration. The flag can be toggled from the interface for a single record, which requires the right to modify that record.

**The flag of an existing identifier row is never changed by a later load.** It is decided once, at the moment the identifier row is first created, by the enclosing scope of the data file that created it: a record declared inside a scope marked not updatable gets a row with the flag set, and a record declared outside such a scope gets a row with the flag clear. When a later load writes the same identifier again, the bulk assignment of section 2.5 updates only the entity, the record identifier and the modification stamp; the flag column is left exactly as it was. Three consequences follow, and a package author must know all three:

1. Moving a record in a data file from an ordinary scope into a not-updatable scope does not protect the records already installed in existing tenants; it protects only the tenants that install the package afterwards.
2. Moving a record out of a not-updatable scope does not expose the already-installed records to the package's updates either; they stay protected.
3. The only ways the flag of an existing row changes are the toggle from the interface and the deletion of the row followed by a fresh creation.

Demonstration data is always loaded with the flag set, so that updating a package never re-imposes demonstration records a user has edited or deleted.

### 3.4 Orphan cleanup

A package update may *remove* a record from its data files. The record must then disappear from the tenant, or the tenant would accumulate records that no package owns any more. At the end of a build that updated at least one package:

1. Collect every external-identifier row whose package is one of the updated packages, whose record identifier is set and whose not-updatable flag is false, most recent first.
2. Skip every row whose complete form was seen during this build.
3. Skip rows whose entity no longer exists in the registry.
4. Skip a row whose record is the embedded parent of a child record that *was* seen during this build, so that implicitly created parents survive as long as their children do.
5. If the same record carries another external identifier that is not itself being cleaned, delete only this row, not the record.
6. Otherwise delete the record, in an environment marked as a package removal so that entities which normally refuse deletion allow it.
7. If the record has already vanished, delete the dangling row.
8. Finally, rebuild the per-tenant specialisations of any views that need them, and clear the set of identifiers seen during the build.

The consequence a package author must know: **deleting a record from a data file deletes the record on the next update, unless the record is marked not updatable.**

### 3.5 A worked reload

**Given** a package whose data files declare three records: a rule with identifier `alpha.rule_a`, updatable; a message template with identifier `alpha.template_b`, declared inside a not-updatable scope; and a parameter with identifier `alpha.param_c`, updatable.

**And** the tenant has installed the package, then edited the template and the parameter through the interface.

**When** the package is updated to a version whose data files declare the rule with a changed value, the template unchanged, and no parameter at all.

**Then**:

1. The rule is updated to the new value; the user's edit to it, if any, is lost. This is correct: the record is updatable, so the package owns it.
2. The template is left exactly as the user edited it, because its identifier is marked not updatable.
3. The parameter is deleted at orphan cleanup, because its identifier belongs to the package, was not seen in this build, and is updatable. The user's edit is lost with the record.

## 4. The data files a package ships

### 4.1 Kinds and order

| Kind | Content | Handling |
|---|---|---|
| Declaration document | A tree of nodes describing records and operations. | Validated against the declaration schema, then executed node by node. |
| Tabular data file | A table of comma-separated values whose first row names the fields. The file name up to the first hyphen is the transport name of the entity. | Executed through the generic import operation of section 6. |
| Direct statement file | Statements executed verbatim against the store. Used only where the entity layer cannot express the change. | Executed verbatim inside the installation transaction. |
| Client script file | A script delivered to the client. | Accepted in the list and ignored by the loader; it reaches the client through the asset bundles. |
| Any other suffix | | Refused with "Can't load unknown file type " followed by the file name. |

For an installation the loader executes the initial list and then the ordinary list; for an update and for a reinitialisation it executes the same lists in the same order; demonstration data is a separate list. Order within a list is significant and is part of the package's contract: a file that references an external identifier defined by a later file fails. The conventional order inside the ordinary list is security declarations first, then reference data, then views, then actions and menus, then everything that references them. A file listed twice in the same list is executed twice, with a warning naming the file and the package.

Every data file is executed with the session language forced to none, which means every value written by a data file is written as the **source-language** value, never as a translation, whatever the language of the user or process that triggered the load. A worked example: loading a declaration document that sets a text field to the value "foo" while the session language is French sets the source value to "foo" and leaves the French translation untouched.

### 4.2 The value forms a declared field accepts

The full grammar of a declaration document is in [`../overview/package-system.md`](../overview/package-system.md), section 8. From the data point of view, what matters is the set of forms a field value may take and what each one produces:

| Form | Produces |
|---|---|
| Verbatim text | The node's text, unchanged. This is the default. |
| Reference to an external identifier | The record named by that identifier. For a polymorphic reference field the value becomes the entity name, a comma and the identifier. If the identifier cannot be resolved and creation is not forced, the whole record is skipped with a warning. |
| Evaluated expression | The result of evaluating a restricted expression whose context provides a resolver from external identifier to record identifier, the acting user, the relational write commands, the current date and time helpers, and the entity being written. |
| Search | The result of searching the entity with a filter. For a list on both sides the value is the whole matching set; otherwise it is the first match. An attribute selects which field of the match is taken, defaulting to the identifier. |
| Integer, decimal, list, fixed sequence | The text parsed as that shape. For an integer, the literal word for absent yields no value. |
| Serialised document or markup | The node's children serialised, with external-identifier substitutions applied: an occurrence of the substitution pattern naming an external identifier is replaced by that record's numeric identifier, and a doubled percent sign becomes a single one. |
| File path | The text treated as a path inside the contributing package; the stored value is the package name, a comma and the path. A missing file fails the load with "No such file or directory: " followed by the path and the package. |
| Encoded file content | Only valid together with a file path: the file's bytes, encoded. Using it without a file is refused with "base64 type is only compatible with file data". |

Any other declared form is refused with "Unknown type " followed by the value.

After evaluation the produced value is coerced to the field's type: a link to one record takes the integer or no value; an integer field takes an integer; a decimal or monetary field takes a decimal number; a boolean field parses the text, treating the digit zero and the words for false and for off, case-insensitively, as false, and everything else as true.

A field node naming a list of records may contain record nodes. Each child is created after the parent record with the inverse field set to the parent's identifier; any text value of such a field node is ignored, because the children are written through their own inverse.

### 4.3 Tabular data files

1. Decode the file as an eight-bit Unicode transformation format; the quoting character is the double quote and the separator is the comma. The transport name of the entity to load is the file name up to the first hyphen.
2. The first row is the list of field paths.
3. When the load is not in initial mode, the field list must contain the identifier column `id`; otherwise the file is not executed at all, an error is recorded saying that the import specification does not contain the identifier column and that the load cannot continue, and the loader moves on to the next file. This check is made on the first row as written, before any column is removed.
4. A heading containing the at sign designates a translation column; it is removed, together with its cell in every row, because translations are loaded separately by the translation machinery (section 9).
5. If removing the translation columns leaves no column at all — that is, every heading of the file was a translation column — the file is skipped: no row is loaded from it, no message is produced, and the loader moves on to the next file.
6. Rows that are empty, or that contain only empty cells once the translation columns have been removed, are dropped.
7. The remaining rows are passed to the generic import operation of section 6, with a context recording the load mode, the contributing package, the file name and the not-updatable flag.
8. If the import produces any message of kind error, the whole installation fails with "Module loading " the package name " failed: file " the file name " could not be processed:" followed by the messages.

## 5. Field path notation

A column heading is a **field path**: it names a field of the entity being loaded, and may continue through relations into fields of the related entity.

### 5.1 Grammar

| Written | Normalised to | Meaning |
|---|---|---|
| `name` | `name` | The field `name` of the entity being loaded. |
| `id` | `id` | The external identifier of the record itself. Creates the record under that identifier when it is unknown, updates the record it names when it is known. |
| `.id` | `.id` | The numeric identifier of the record itself. Updates that record; an unknown value is an error. |
| `partner_id/id` | `partner_id/id` | The link `partner_id` resolved from an external identifier. |
| `partner_id/.id` | `partner_id/.id` | The link `partner_id` resolved from a numeric identifier. |
| `partner_id` | `partner_id` | The link `partner_id` resolved from the display name of the target. |
| `partner_id.id` | `partner_id/.id` | The same as the numeric form: a dot before `id` that is not already preceded by a slash is rewritten to a slash. |
| `partner_id:id` | `partner_id/id` | The same as the external-identifier form: a colon before `id` that is not already preceded by a slash is rewritten to a slash. |
| `order_line/product_id/id` | `order_line/product_id/id` | Two steps: the list of records `order_line`, then the link `product_id` of the line, resolved from an external identifier. |
| `properties.deadline` | `properties.deadline` | The user-defined field whose internal name is `deadline`, inside the user-defined field column `properties`. The part after the dot is a property name, not a relation step. |

The normalisation is applied to every path before anything else, so the three spellings of a numeric link column are interchangeable, and so are the two spellings of an external-identifier link column.

### 5.2 What a leaf means, per field kind

| Leaf | Field kind | Value expected in the cell |
|---|---|---|
| The field itself | Any non-relational field | A text that the converter of that field type accepts (section 6.3). |
| The field itself | Link to one record | The display name of exactly one target record. |
| The field itself | List on both sides | The display names of the targets, separated by commas. |
| `id` | Link to one record | One external identifier. |
| `id` | List on both sides | External identifiers separated by commas. |
| `.id` | Link to one record | One numeric identifier. |
| `.id` | List on both sides | Numeric identifiers separated by commas. |
| A sub-path | List of records | The value of that field on the child record; see section 6.2 for how several children are written. |

Only one of the three referencing forms may be given for one relation in one row. Giving two is refused with "Ambiguous specification for field '" the field label "', only provide one of name, external id or database id". Giving a non-referencing sub-field of a link to one record is refused with "Can not create Many-To-One records indirectly, import the field separately".

### 5.3 Worked examples

**A flat file with three columns.** The columns `id`, `name` and `country_id/id`, and the row holding `__import__.acme`, "Acme Incorporated" and `base.us`. The import creates one party named "Acme Incorporated" whose country is the record named by `base.us`, and assigns it the external identifier `__import__.acme`. Running the same file again updates the same record instead of creating a second one.

**A file with a list of records.** The columns `id`, `name`, `order_line/product_id/id` and `order_line/product_uom_qty`, and three rows:

| `id` | `name` | `order_line/product_id/id` | `order_line/product_uom_qty` |
|---|---|---|---|
| `__import__.so1` | `SO0001` | `__import__.desk` | 3 |
| | | `__import__.chair` | 12 |
| | | `__import__.lamp` | 1 |

The first row carries values for non-relational columns, so it starts a record. The second and third rows carry values only in the columns of the list of records, so they continue the same record. The result is one order with three lines.

**A file with a user-defined field.** The columns `id`, `name` and `properties.deadline`, where `deadline` is the internal name of a date definition carried by the parent record. The cell is converted with the rules of the definition's type.

## 6. The generic import operation

One operation turns a field list and a row matrix into records. It is what a tabular data file, an interactive import session and an integration all call, so the behaviour below is identical for all three.

### 6.1 Inputs and outputs

Input: the ordered list of field paths, and the row matrix, each row a list of texts in the same order. Context values steer it: the load mode, the contributing package, the not-updatable flag, a row limit, the set of fields for which a missing target may be created from its name, the set of fields to leave empty when their value cannot be resolved, the set of fields whose failure skips the whole row, and a flag saying that the caller is a user-driven import file.

Output: three values.

| Output | Content |
|---|---|
| Identifiers | The identifiers of the records created or updated, in the order the rows produced them; or the value for absence when at least one message of kind error was produced, because the whole operation is then rolled back. |
| Messages | The list of messages, each carrying its kind (error or warning), its text, the row range it belongs to, the field it belongs to, the offending record data and, where the converter supplies one, a hint. |
| Next row | The index of the first row not processed, when a row limit stopped the operation before the end; zero when everything was processed. |

The whole operation runs inside one savepoint. If any message of kind error was produced, the savepoint is rolled back, the identifier list becomes the value for absence, and every change made to the registry cache during the attempt is discarded.

### 6.2 Row extraction and record spanning

1. Normalise every field path (section 5.1).
2. Walk the rows from the first. For each row, the values of the columns whose first step is **not** a list of records become the record being built.
3. Take the following rows as long as they carry a value in at least one column whose first step **is** a list of records and no value in any other column. These continuation rows belong to the same record.
4. For each column whose first step is a list of records, collect the cells of that column across the record's rows, drop the entirely empty ones, and recurse: the collected cells are a row matrix for the target entity, with the remainder of the path as its field list.
5. Emit the record with the row range it spans, and continue after the last row consumed.

A column whose first step is a link to one record never spans rows: a link takes one value, and the sub-path only says how to resolve it.

### 6.3 Converting a cell to a value

Each cell is converted by the converter of the field's type. An empty cell always produces the empty value of the type without invoking the converter.

| Field type | Rule | Refusal message |
|---|---|---|
| Boolean | True for the digit one and the words for true and for yes, in the base language and in every installed language, case-insensitively. False for the empty text, the digit zero and the words for false and for no, likewise. Anything else yields true **and** a warning. | "Unknown value '" the value "' for boolean field '" the field label "'", with the hint "Use '1' for yes and '0' for no". When the field is in the skip-records set, the row is skipped instead. |
| Integer | The text parsed as a whole number. | "'" the value "' does not seem to be an integer for field '" the field label "'" |
| Decimal, monetary | The text parsed as a number. | "'" the value "' does not seem to be a number for field '" the field label "'" |
| Single-line text, long text, rich text, binary, polymorphic reference | Taken verbatim. | none |
| Date | Parsed, then written in the canonical date form. | "'" the value "' does not seem to be a valid date for field '" the field label "'", with the hint "Use the format '2012-12-31'" |
| Date and time | Parsed, interpreted in the acting environment's time zone, then converted to coordinated universal time. | "'" the value "' does not seem to be a valid datetime for field '" the field label "'", with the hint "Use the format '2012-12-31 23:59:59'" |
| Closed value list | Matched, case-insensitively, against the technical value, then against the label in the base language, then against the label in every installed language. | "Value '" the value "' not found in selection field '" the field label "'", with the list of acceptable labels as the hint. When the field is in the skip-records set the row is skipped; when it is in the set-empty set the value becomes empty. |
| Structured document | Parsed as a document. | "'" the value "' does not seem to be a valid JSON for field '" the field label "'" |
| User-defined field values | Each property is converted by the rule of its own definition type; a closed-list property accepts the technical value or the label, a tag property accepts a comma-separated list of values or labels, a link property is resolved like a link to one record. | For a closed-list property: "'" the value "' does not seem to be a valid Selection value for '" the property label "' (subfield of '" the field label "' field)."; for a tag property the same sentence with "Tag" in place of "Selection"; for a boolean property "Unknown value '" the value "' for boolean '" the property label "' property (subfield of '" the field label "' field)."; for an integer property "'" the value "' does not seem to be an integer for field '" the property label "' property (subfield of '" the field label "' field)."; for a decimal property the same sentence with "an float" in place of "an integer". Writing the whole user-defined field column at once is refused with "Unable to import'" the field label "' Properties field as a whole, target individual property instead." |
| Polymorphic link in two columns | Parsed as a whole number, like an integer. | as for an integer |

### 6.4 Resolving a link

For a link to one record, a list on both sides, or a link property, the cell is resolved to a record identifier by one of three referencing forms:

| Form | Resolution |
|---|---|
| By display name, when the path ends at the field itself | Search the target entity for a record whose name matches the value exactly. When several match, take the first and add the warning "Found multiple matches for value \"" the value "\" in field \"" the field label "\" (" the number of matches " matches)". An empty value resolves to no link. |
| By external identifier, when the path ends in `id` | Complete a bare name with the contributing package, flush the pending batch so that a record created earlier in the same file can be found, then look the identifier up. The lookup returns the entity and the record identifier of the row only when the row's record still exists in the table of the target entity, so a row that points at a deleted record resolves to nothing. When the row exists but its entity is not the entity the link points at, the cell is refused with "Invalid external ID " the complete identifier ": expected model " the transport name of the entity the link points at, between apostrophes ", found " the transport name of the entity the row records, between apostrophes. A value that the boolean converter reads as false resolves to no link. |
| By numeric identifier, when the path ends in `.id` | Parse the value as a whole number and check that the record exists. A value that the boolean converter reads as false resolves to no link. A value that is not a number is refused with "Invalid database id '" the value "' for the field '" the field label "'". |

When nothing is found, the cell is refused with "No matching record found for " the form, one of the words "name", "external id" or "database id", " '" the value "' in field '" the field label "'". The message carries an action that opens the list of possible values: the target entity itself for the name form, and the external identifier registry restricted to the target entity for the two identifier forms. When the field is named in the set-empty set the value becomes empty instead; when it is named in the skip-records set the whole row is skipped instead.

A referencing form other than these three is refused with "Unknown sub-field “" the name of the form "”", where the name is written between the same curly quotation marks the message uses. A file cannot reach this refusal through the ordinary paths, because a sub-path that is neither `id` nor `.id` nor a field of the target entity is already refused by the indirect-creation rule of section 5.2 before the resolver is called; it is the guard that protects the resolver when an integration calls it directly with a sub-field it invented.

A list on both sides splits the cell on commas and resolves each part with the same rule; the result is a set command that replaces the whole list, or, when the caller asks for it, one link command per part, which adds without removing. When the field is in the set-empty set, the parts that did not resolve are dropped; when it is in the skip-records set and any part did not resolve, the whole row is skipped.

A list of records produces one command per child row: a link and an update command when the child was matched by a referencing form, and a create command otherwise. When the value of a list of records is a single cell containing a comma-separated list of references and nothing else, it is expanded into one child per reference, exactly as for a list on both sides.

### 6.5 Creating a missing target from its name

A caller may name the fields for which a missing target should be created from its name alone. The creation applies every default value and every validation of a normal creation. When the creation fails, the savepoint is rolled back and the cell is refused with "No matching record found for name '" the value "' in field '" the field label "' and the following error was encountered when we attempted to create one: " followed by the message "Cannot create new '" the target entity's description "' records from their name alone. Please create those records manually and try importing again."

For a list of records, the set of fields is passed down with the field prefix removed, so that a caller may allow the creation of a target of a sub-field without allowing it at the top level.

### 6.6 Batching, flushing and error recovery

1. Converted records are accumulated into a batch, each carrying its external identifier, its values and its row range.
2. The batch is flushed when the loader needs a record it may have created without having written it to the store: when an external identifier being resolved belongs to the batch, and when a list of records of one of the entities the file can create must be searched by name.
3. A flush first tries to write the whole batch in one operation, inside its own savepoint.
4. If the whole batch fails, the savepoint is rolled back and the records are written one at a time, so that the failing rows can be reported individually. The message of the failed batch write is reported first when at least one individual row also failed.
5. Each individual failure rolls back to the operation's savepoint and appends a message: a store warning becomes a message of kind warning; a store error becomes a message of kind error carrying the translated store message and, when the failing column can be identified, the field; a refusal raised by a validation becomes a message of kind error carrying its text; any other failure becomes "Unknown error during import: " the failure class ": " the failure message, with the hint "Resolve other errors first".
6. After ten errors, and when the errors are more than one per ten rows processed, the loop stops and appends "Found more than 10 errors and more than one error per 10 records, interrupted to avoid showing too many errors."
7. A broken transaction is reported once as "Unknown database error: '" the store's message "'".

### 6.7 Skipping and emptying

Two context sets change what an unresolvable value does, and they are what the interactive session writes when the user answers an error dialogue:

| Set | Effect |
|---|---|
| Fields to leave empty | The value becomes empty instead of failing. For a list on both sides, the parts that did not resolve are dropped and the rest are kept. |
| Fields whose failure skips the record | The converter returns the value for absence, and the whole row is dropped before the batch is built, so no record is created for it. |

## 7. The interactive import session

### 7.1 The entities

| Entity | Transport name | Table | Role |
|---|---|---|---|
| Import session | `base_import.import` | `base_import_import` | An interactive assistant holding the target entity's transport name (`res_model`), the uploaded bytes (`file`, held inline in the table rather than as an attachment), the file name (`file_name`) and the declared content type (`file_type`). Sessions survive for twelve hours before the periodic cleanup removes them, because a user may take a long time over the mapping. |
| Saved column mapping | `base_import.mapping` | `base_import_mapping` | A persistent record remembering that, for one entity (`res_model`), a column heading (`column_name`) was mapped to a field path (`field_name`). It exists because the column names produced by an outside system rarely match the field names here, and a user should not have to repeat the mapping at every import. |

### 7.2 The procedure

1. The user selects the target entity and uploads a file.
2. The session parses a preview and proposes a mapping (sections 7.3 to 7.7).
3. The user corrects the mapping and the options.
4. The user runs a **test import**: every write and every validation is performed and then rolled back, so that the errors are known without changing anything.
5. For each error the user may choose to leave the offending field empty, to skip the offending records, or to substitute a fallback value; those choices are carried into the next attempt as the sets of section 6.7 and the fallback map of section 7.9.
6. The user runs the **real import**. If it produces blocking errors the transaction is rolled back and the user returns to step 5.
7. On success the column mapping is saved (section 7.10) and, when the file was longer than the batch size, the next batch is offered.

### 7.3 Reading the file

The reader is chosen by trying, in order, the content type detected from the bytes, the content type the client declared, and the extension of the file name. Four shapes are accepted: comma-separated values, and the three spreadsheet workbook formats. Anything else is refused with "Unsupported file format \"" the declared type "\", import only supports CSV, ODS, XLS and XLSX". A reader that cannot be used because an optional component is absent is refused with "Unable to load \"" the extension "\" file: requires Python module \"" the component "\"". A file that cannot be read at all is refused with a message naming the file and the reader that was tried.

For a comma-separated file:

1. When no character encoding is given, it is detected from the bytes. A byte-order mark at the start of the file is stripped by choosing the unmarked form of the detected encoding. A decoding failure is refused with "There was an issue decoding the file using encoding “" the encoding "”." followed by a second line saying whether the encoding was detected automatically or selected manually.
2. When no separator is given, the candidates comma, semicolon, tabulation, space, vertical bar and the unit separator are tried in that order; the first candidate for which every row has the same width and that width is at least two wins. When none does, the comma is used, so that the user is told to choose one.
3. The text delimiter must be exactly one character, otherwise the file is refused with "Error while importing records: Text Delimiter should be a single character."
4. Rows in which every cell is empty or blank are dropped.

For a workbook, the sheet to read is the one the options name, or the first sheet; the list of sheet names is returned to the client so that the user can choose. A cell holding an error value is refused with "Invalid cell value at row " the row number ", column " the column number ": " the error text. A date-formatted cell whose format is neither a date nor a date and time is refused with "Invalid cell format at row " the row number ", column " the column number ": " the value ", with format: " the format ", as (" the detected kind ") formats are not supported." A numeric cell whose value is whole is read as a whole number, and otherwise as a decimal.

A file with no readable content is refused with "Import file has no content or is corrupt".

### 7.4 The importable field tree

The session offers the fields of the target entity, three relation steps deep:

1. The external identifier of the record itself, labelled "External ID", is always offered first.
2. Every field of the entity is offered, except the system columns and every field the definition marks read-only.
3. A link to one record and a list on both sides are offered with exactly two sub-fields: the external identifier of the target, labelled "External ID", and its numeric identifier, labelled "Database ID".
4. A list of records is offered with the whole importable tree of the target entity, one step shallower. Its numeric identifier is offered as a sub-field only to a user of the technical group.
5. A user-defined field is expanded into one entry per definition found on the parent records, labelled with the definition's label followed by the parent's display name in parentheses. Definitions of the separator kind, and link definitions whose target entity is not installed, are skipped. A user who may not read the parent entity or the definition field sees no expansion.
6. Each entry carries whether the field is required, so that the client can insist on a value.

### 7.5 Inferring the type of a column

For each column the first ten rows are examined and a set of plausible field types is produced, which narrows the fuzzy matching of section 7.6 and drives the "suggested fields" list the client shows:

| Observation over the sample | Types proposed |
|---|---|
| Every value empty | every type |
| Every value starts with `__export__` | the identifier, and the three relation kinds |
| Every non-empty value is a run of digits | integer, decimal and monetary, plus the identifier and the three relation kinds; and also boolean when the only values are the digits zero and one |
| Every value is one of the words for true and false, or their initials | boolean |
| Every value parses as a number, once a currency symbol and accounting parentheses are removed | decimal and monetary |
| Every value matches one date pattern | date and date and time |
| Every value matches one date pattern followed by one time pattern | date and time |
| Anything else | long text, single-line text, binary, closed value list, rich text and tags |

The date patterns tried are, in order: the pattern the options already name; the date format of the acting user's language; and then every combination of a day, month and year order taken from the four orders "month day year", "day month year", "year month day" and "year day month", each with a four-digit and a two-digit year, joined by a space, a slash, a hyphen, a dot or nothing. The time patterns are the twenty-four-hour forms with seconds, with minutes and with hours only, and the twelve-hour forms with seconds, with minutes and with hours only. The first pattern that matches every sampled value is stored in the options and reused.

The separators of a decimal column are inferred from the sample when the options do not name them: a value containing more than one full stop makes the full stop the grouping separator and the comma the decimal separator; a value containing more than one comma makes the reverse; otherwise the last of the two characters to appear is the decimal separator and the other is the grouping separator.

### 7.6 Proposing a mapping

For each column heading, in order:

1. **Saved mapping.** If a saved column mapping exists for this entity and this heading, its field path is used and the distance is recorded as less than zero, so that it always wins the deduplication of step 4.
2. **Exact match.** If the heading equals, case-insensitively, a field name, a field label in the user's language, or a field label in the base language, that field is used with distance zero.
3. **Fuzzy match.** Otherwise, the fields whose type is among the types inferred for the column are compared with the heading, and the distance is one minus the similarity ratio of the two texts, taken as the smallest of the distances to the field name, to the label in the user's language and to the label in the base language. The closest field wins, and only if its distance is below **0.2**; a larger distance means no proposal.
4. **Deduplication.** When two columns propose the same single field, only the one with the smaller distance keeps it; the other is left unmapped. Columns whose proposal is a path of more than one step are exempt, because a deliberate mapping into a relation is treated as advanced.

A heading that contains slashes is matched one segment at a time, for a heading of any number of segments. The heading is split on the slash and each segment is stripped of the spaces a writer may have put around the slash for readability. The first segment is matched, by the exact and fuzzy rules of steps 2 and 3 above, against the fields offered for the target entity. Every later segment is matched, by the same rules, against the fields offered for the entity reached by the segment before it, so that the entity searched for segment number *n* is always the entity that the field chosen for segment number *n* − 1 points at. The saved-mapping step is not applied to a segment, because a saved mapping stores a whole heading and not a part of one. The field names chosen for the segments, joined by slashes in the order the segments appeared, are the proposed path, and no distance is reported for it. A segment that matches nothing abandons the whole heading, whatever the earlier segments matched, and the column is left unmapped.

### 7.7 The preview

The preview returns, for each column: the first non-empty value truncated to fifty characters, and up to five such values for the hover display; the inferred types; and the proposed field path. It also returns the whole importable field tree, the options as they were completed by the detection steps, whether the file is long enough to need batching, the number of rows, whether the technical group is active, and whether the advanced mode should be switched on. The advanced mode is switched on when any heading is a path of more than one step, or when any proposed mapping is.

### 7.8 The options

| Option | Effect |
|---|---|
| Has headers | Whether the first row names the columns. When false, no mapping is proposed and the user maps every column by hand. |
| Encoding | The character encoding of a comma-separated file. Detected when absent. |
| Separator | The column separator of a comma-separated file. Detected when absent. |
| Quoting | The text delimiter of a comma-separated file. Must be exactly one character. |
| Sheet | The sheet of a workbook to read. |
| Date format, date and time format | The patterns used to parse date columns. Detected when absent. |
| Decimal separator, grouping separator | The characters used to parse decimal columns. Inferred when absent. |
| Skip | How many data rows to drop from the front. Used to resume after a partial import. |
| Limit | How many rows to process in one run. Used for batching. |
| Fields | The chosen mapping, one field path or the value for absence per column. |
| Keep matches | Re-parse the file without proposing a new mapping. |
| Advanced | Whether the client offers relation sub-fields in the mapping list. |
| Create missing records from their name | The set of fields for which a missing target may be created from its name. |
| Set empty on error | The set of fields to leave empty when their value cannot be resolved. |
| Skip records on error | The set of fields whose unresolvable value drops the whole row. |
| Fallback values | Per field, a substitute value for a boolean or closed-list cell that matches nothing. |
| Tracking disabled | Suppresses the change-tracking messages that a write would otherwise post, so that importing thousands of records does not fill the discussion threads. |

### 7.9 Fallback values

For a boolean field, a value that is not one of the digits zero and one and not one of the words for true and false is replaced by the fallback. For a closed value list, a value that matches no technical value and no label, compared case-insensitively, is replaced by the fallback; when the fallback is the word for skipping, no value is written at all. The acceptable labels of the list are read once per field before the rows are scanned.

### 7.10 Several columns mapped on the same field

Two or more columns may be mapped onto one field. The values are then joined before the row is imported:

| Field type | Join |
|---|---|
| Single-line text | The values, in column order, separated by a space. Trailing whitespace is removed first when the field trims. |
| Long text | The values, in column order, separated by a line break. |
| Rich text | The values, in column order, separated by a line-break element. |
| List on both sides | The values, in column order, separated by commas, which is the separator the resolver itself uses. |
| Any other type | The value of the first mapped column; the others are ignored. |

A worked example: two columns both mapped onto a long text field, the first row holding "Value part 1" and "Value part 2" and the second row holding "I am" and "Batman", produce two records whose text is "Value part 1" and "Value part 2" on two lines, and "I am" and "Batman" on two lines.

### 7.11 Binary and image columns

A cell mapped onto a binary field that stores its content as an attachment is treated as follows:

1. A value matching the configured address pattern, whose default is an address beginning with the unsecured or the secured hypertext transfer scheme, is fetched. Only a user the installation allows may do this; the default rule allows an administrator. A user who is not allowed is refused with "You can not import file via URL, check with your administrator or support for the reason."
2. The fetch has a timeout, whose default is three seconds, and a maximum size, whose default is ten mebibytes. A response whose declared length or whose accumulated content exceeds the maximum is refused with "File size exceeds configured maximum (" the maximum in bytes " bytes)". Any other failure is refused with "Could not retrieve URL: " the address " [" the field name ": L" the row number "]: " the failure.
3. A fetched value whose content is an image is checked for a plausible resolution before it is stored.
4. A value that is not an address but contains a dot is treated as a file name: the cell is emptied and the file name is returned to the client, so that the client can upload the matching file separately.
5. Any other value must be valid encoded content, otherwise it is refused with "Found invalid image data, images should be imported as either URLs or base64-encoded data."

### 7.12 Batching

When the file is longer than the row limit, the import runs on the first batch and returns the index of the first unprocessed row. The client then repeats the run with that index as the number of rows to skip. The row limit is removed from the options before the generic import operation is called and passed to it as the row limit of section 6.1, so that the extraction stops at the right row rather than converting the whole file each time.

### 7.13 Saving the mapping

When a run produces at least one identifier and the file has headers, every mapped column is remembered: an existing saved mapping for the same entity and heading is updated to the field chosen this time, and a new one is created otherwise. This is what makes step 1 of section 7.6 succeed the next time.

### 7.14 Import templates

An entity may offer downloadable template files, each with a label and a path, so that a user starts from a file whose headings are already the right field paths. The base offering is empty and each capability package adds the templates of the entities it defines.

## 8. Export

### 8.1 Who may export

Exporting is refused unless the acting user is an administrator or belongs to the export group (`base.group_allow_export`), with "You don't have the rights to export data. Please contact an Administrator."

Every export writes one line to the technical log naming the acting user, the number of records, the entity, the caller's network address, the exported field paths, and either the first ten identifiers or the filter that selected the records.

### 8.2 The two modes

| Mode | Column headings | Relation values | Purpose |
|---|---|---|---|
| Import-compatible | The field paths themselves | A link to one record and a list on both sides are exported as external identifiers or as display names, in one cell | The file can be imported back into this system or into another installation. |
| Labelled | The labels of the fields, in the reader's language | Values are rendered for a human reader | The file is for reading, not for re-importing. |

In import-compatible mode the field tree offered for selection hides every read-only field, and a link to one record or a list on both sides is offered with only two sub-fields: its external identifier and its display name. In labelled mode the numeric identifier of the record is offered as an additional column. The external identifier column is always labelled "External ID"; it is withdrawn for an entity that is read from a stored query, because such an entity has no table and therefore cannot be given one. Fields the definition marks as not exportable are never offered.

The field tree is offered three relation steps deep: a relational field one or two steps from the root is offered with children, and its own value defaults to the external identifier of the target.

### 8.3 Building the rows

1. Normalise every field path with the rule of section 5.1.
2. Fetch, in one pass and recursively, every field named by every path, so that a relational path does not read the related records one at a time. A path that stops at a relational field is completed with the display name of the target.
3. For each record emit one line. For each path, in column order:
   - `.id` writes the numeric identifier.
   - `id` writes the external identifier of the record, creating one under the `__export__` prefix when the record has none (section 2.8).
   - A non-relational field writes its exported form (section 8.5).
   - A polymorphic reference in import-compatible mode writes the entity name, a comma and the identifier.
   - A relational field writes the sub-rows of its targets. The first sub-row is merged into the current line and the remaining sub-rows are appended after it, which is how a record with three lines becomes three rows whose non-relational columns are filled only on the first.
   - In import-compatible mode a list on both sides is never expanded into several rows: it writes one cell holding the external identifiers, or the display names, of every target, separated by commas. The cell is placed in the column the user chose for the external identifier, the name or the display name, and in the column of the relation itself when none of the three was chosen.
   - A user-defined field writes the value of the named property; a link property writes the display name of the target, a tag property the labels separated by commas, and a closed-list property the label.
4. When at least one column is an external identifier, the identifiers of every column are resolved in one pass at the end, one query per entity, so that a large export does not perform one lookup per cell.

An entity read from a stored query cannot produce an external identifier column at all; asking for one refuses with a message stating that the identifier column of that entity cannot be exported because its table is not an ordinary table.

Records are exported in batches, and each batch is discarded from the working set after it has been rendered, so that memory does not grow with the size of the export.

### 8.4 Saved field lists

A user may save a chosen list of field paths and reuse it. It is stored as an Export (`ir.exports`, table `ir_exports`) carrying a name (`name`) and the transport name of the entity (`resource`), with one Export Line (`ir.exports.line`, table `ir_exports_line`) per path, holding the path (`name`) and the link to the export (`export_id`, deleted with it). Exports are ordered by name and then by identifier; lines are ordered by identifier, which is the order the user chose. Duplicating an export duplicates its lines.

### 8.5 Rendering a value

| Value | Written as |
|---|---|
| Absent, or the value for absence | The empty cell |
| Boolean | The words for true and false |
| Integer, decimal, monetary | The number, with the field's own precision |
| Date, date and time | The canonical text form; a date and time is converted to the reader's zone in labelled mode |
| Closed value list | The technical value in import-compatible mode, the label in labelled mode |
| Single-line text, long text, rich text | The text |
| Binary | The encoded content |
| Link to one record | The external identifier, or the display name, depending on the chosen sub-field |

### 8.6 Formats

Two formats are offered. The list of formats always reports both, and reports the workbook format as unavailable when its optional component is absent.

| Format | Content type | Extension | Rules |
|---|---|---|---|
| Comma-separated values | `text/csv;charset=utf8` | `.csv` | Every cell is quoted. The header row is written first. An absent value becomes the empty cell; a byte value is decoded to text. A cell whose text begins with an equals sign, a hyphen or a plus sign is prefixed with an apostrophe, so that a spreadsheet application does not read it as a formula. Grouped export is refused with "Exporting grouped data to csv is not supported." |
| Spreadsheet workbook | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | `.xlsx` | The header row is written in a bold style and every column is widened. A date, a date and time, a decimal and a monetary value keep their own cell style, so that the workbook shows them as numbers and dates rather than as text. A carriage return inside a cell becomes a space. A cell longer than the format's maximum text length is replaced by "The content of this cell is too long for an XLSX file (more than " the maximum " characters). Please use the CSV format for this export." A byte value that cannot be decoded is refused with "Binary fields can not be exported to Excel unless their content is base64-encoded. That does not seem to be the case for " the column heading "." |

The file name is the description of the entity followed by its transport name in parentheses, cleaned of characters a file system would refuse, plus the extension.

### 8.7 Grouped export

In labelled mode the caller may ask for the rows to be grouped, by one or more fields, exactly as the screen groups them. Grouping is available only in the workbook format.

1. The records are exported with the numeric identifier as a hidden first column, so that each exported row can be attributed to its record.
2. The groups are read with their counts and their member identifiers, and a tree of groups is built in the order the grouped read returns them.
3. Each record's rows are placed in every group it belongs to, in the order the plain export produced them, which preserves the natural order of the entity inside each group.
4. Each group writes a header row: the group's label, indented by four spaces per level, followed by the count in parentheses. A group whose value is empty is labelled "Undefined", unless the grouping field is a boolean, whose empty value is meaningful.
5. In every column after the first, the group header row carries the aggregate of that column when the field declares one. The aggregates are the maximum, the minimum, the sum, the conjunction, the disjunction and the average. An unsupported aggregate is skipped and logged. Empty cells produced by row expansion are excluded from the aggregate, so that a text placeholder is never added to a number.
6. The average of a group is the sum of its rows divided by the group's count; the average of a parent group is the sum of each child's average multiplied by that child's count, divided by the parent's count.
7. Aggregates are not computed for a column whose path crosses a relation, and not for the first column, which carries the group label.

## 9. Exchanging translations

The translated text of shipped records travels separately from the records themselves. A translation column in a tabular data file — a heading containing the at sign — is removed before the rows are imported, and the translations are loaded by the translation machinery instead. The export and import of translation catalogues, the interaction between a translation import and the not-updatable rule, and the resolution order applied when a translated value is read are specified in [`../runtime/translation.md`](../runtime/translation.md).

## 10. Exchange contracts used by integrations

The generic import and export of this document are entity-shaped: their unit is a record and their vocabulary is field paths. Three other exchange contracts exist, each with its own document:

| Contract | Where specified |
|---|---|
| The remote transport that a client or an integration calls to create, read, write, search and invoke operations, including the same generic import and export operations | [`../interfaces/`](../interfaces/) |
| Structured business documents exchanged with trading partners — invoices, credit notes, orders and their delivery over a network | [`../domains/electronic-invoicing-and-document-exchange/`](../domains/electronic-invoicing-and-document-exchange/) |
| Printable documents and their rendering | [`../runtime/report-rendering.md`](../runtime/report-rendering.md) |

All three ultimately write through the same entity layer, so every rule of [`persistence-identity-and-values.md`](persistence-identity-and-values.md) applies to them unchanged.

## 11. Acceptance criteria

Each criterion is independently verifiable against a replacement.

**External identifiers**

1. Given a shipped record named `base.main_company` whose entry is not marked not updatable, when its capability package is updated with a changed value in its data file, then the record is updated in place and keeps its numeric identifier.
2. Given a shipped record whose entry is marked not updatable and whose value a user has changed, when the package is updated, then the user's value is kept.
3. Given a data file that declares a record with an external identifier that already names a record of a different entity, when the file is loaded, then the load is refused with the message of section 3.2, step 3.
4. Given a data file that declares a record whose external identifier row exists but whose record has been deleted, when the file is loaded, then the stale row is removed and the record is created afresh with a new numeric identifier.
5. Given an import file with the external identifier `base.some_record` where `base` is an installed package, when the import runs, then it is refused with the module-prefix message of section 2.8.
6. Given an import file with the external identifier `__import__.some_record`, when the import runs twice, then exactly one record exists and the second run updated it.
7. Given a record with no external identifier, when it is exported with the external identifier column, then a row is created under the `__export__` prefix whose local name is the table name, an underscore, the record identifier, an underscore and eight hexadecimal characters, and the same identifier is returned on a second export.

**Orphan cleanup**

8. Given a package at one version whose data file declares three updatable records, when the package is updated to a version whose data file declares only two of them, then the third record is deleted and the other two are updated.
9. Given the same situation where the removed record is marked not updatable, when the package is updated, then the record survives.

**The not-updatable flag**

10. Given a package whose data file declares a record outside a not-updatable scope, when the package is installed, then the identifier row is created with the flag clear.
11. Given that same tenant and a later version of the package that declares the same record inside a not-updatable scope, when the package is updated, then the identifier row still carries the flag clear and the record is overwritten by the package's value.
12. Given a tabular data file all of whose headings are translation columns, when the package is installed, then no record is created from that file and the installation continues.

**Field paths**

13. Given the column headings `partner_id.id`, `partner_id:id` and `partner_id`, when each is normalised, then the results are `partner_id/.id`, `partner_id/id` and `partner_id` respectively.
14. Given a file with the columns `id`, `name`, `order_line/product_id/id` and `order_line/product_uom_qty` and the three rows of section 5.3, when it is imported, then one order is created with three lines, in the order of the rows.
15. Given a row that supplies both `partner_id` and `partner_id/id`, when it is imported, then it is refused with "Ambiguous specification for field '" the field label "', only provide one of name, external id or database id".

**Conversion**

16. Given a boolean column holding the value "maybe", when it is imported, then the value written is true and a warning is produced whose hint is "Use '1' for yes and '0' for no".
17. Given a date column holding "31/12/2012" and a detected date pattern of day, month and year separated by slashes, when it is imported, then the stored value is the last day of December 2012.
18. Given a date and time column holding "2012-12-31 23:59:59" and an acting environment whose zone is two hours ahead of coordinated universal time, when it is imported, then the stored value is 2012-12-31 21:59:59.
19. Given a closed-list column holding a label in the reader's language that is not the technical value, when it is imported, then the technical value is written.
20. Given a closed-list column holding a value that matches nothing and a fallback value declared for that field, when it is imported, then the fallback is written and no error is produced.

**Resolution**

21. Given a link column resolved by name whose value matches two records, when it is imported, then the first match is used and a warning naming the number of matches is produced.
22. Given a link column resolved by name whose value matches nothing, when it is imported, then the row fails with "No matching record found for name '" the value "' in field '" the field label "'".
23. Given the same file with that field named in the set-empty set, when it is imported, then the record is created with the link empty and no error is produced.
24. Given the same file with that field named in the skip-records set, when it is imported, then the row is dropped and no record is created for it.
25. Given a file whose second row refers by external identifier to a record created by its first row, when it is imported, then the reference resolves, because the batch is flushed before the lookup.
26. Given a link column resolved by external identifier whose value names a row that records a different entity from the one the link points at, when it is imported, then the cell is refused with the message of section 6.4, naming the identifier, the entity the link points at and the entity the row records.

**Errors and batching**

27. Given a file of one hundred rows of which twelve fail, when the import runs, then the whole import is rolled back, no record is created, and the messages include the twelve failures.
28. Given a file of two hundred rows of which forty fail from the first row on, when the import runs, then the loop stops after the eleventh failure and appends the message about more than ten errors.
29. Given a file of one thousand rows and a row limit of two hundred, when the import runs, then two hundred records are created and the returned next row is two hundred and one.
30. Given a test import that fails, when it completes, then the store holds no new record and the registry cache holds no change from the attempt.

**Export**

31. Given a user who is neither an administrator nor a member of the export group, when an export is requested, then it is refused with "You don't have the rights to export data. Please contact an Administrator."
32. Given one order with three lines exported in import-compatible mode with the columns `id`, `name`, `order_line/product_id/id` and `order_line/product_uom_qty`, then the file holds three rows, the first carrying the identifier and the name and the other two carrying only the line columns; and importing that file back produces one order with three lines.
33. Given a record whose text field is "=SUM(A1:A9)", when it is exported to comma-separated values, then the cell begins with an apostrophe.
34. Given a grouped export to a workbook with a decimal column that declares a sum, then each group header row carries the sum of its rows in that column and the parent group carries the sum of its children.
35. Given a grouped export requested in the comma-separated format, then it is refused with "Exporting grouped data to csv is not supported."

**Mapping**

36. Given a saved column mapping from the heading "Client" to the field path `partner_id/id` for one entity, when a file with that heading is previewed for that entity, then the proposal is that path, whatever the fuzzy match would have produced.
37. Given two columns headed "Name" and "Nom" and a field whose label is "Name", when the file is previewed, then only the closer of the two columns keeps the proposal and the other is left unmapped.
38. Given a successful import of a file with headers, when it completes, then a saved column mapping exists for every mapped column, and re-running the preview proposes the same mapping.

## 12. Reconciliation notes

The target branch carried no file for this topic. The material comes from the record declaration, reload and import sections of the working branch's package document, from the loading, external-identifier and update behaviour of the foundation package, from the import session of the data import package, and from the export of the web package. The following points were resolved while assembling it.

| Point | Situation | Resolution |
|---|---|---|
| Overlap with the package document | The working branch specified the declaration grammar, the reload rules and the external identifiers inside its package document, and the target branch specifies them in [`../overview/package-system.md`](../overview/package-system.md). | The package lifecycle and the node-by-node grammar stay in the package document and are linked, not repeated. What is repeated here is only what a reader needs in order to reason about data: the set of value forms a field may take, the load modes, the create-or-update rule, the not-updatable flag and the orphan cleanup, because those are the rules that decide what a tenant's tables contain after an update. |
| Identifier spelling | The working branch wrote the registry fields as paraphrases. | Replaced by the field names the registry reports: `name`, `module`, `complete_name`, `model`, `res_id`, `noupdate`, `reference`, and the index names `module_name_uniq_index` and `model_res_id_index`. |
| The constraint message on the local name | The working branch quoted it as "External identifiers cannot contain spaces". | The declared message is "External IDs cannot contain spaces" and is reproduced verbatim. |
| The number of accepted file formats for an interactive import | Not stated. | Four are accepted: comma-separated values and the three workbook formats. The refusal message names them. |
| The fuzzy-match threshold | Not stated. | The distance must be below 0.2, where the distance is one minus the similarity ratio of the two texts. Below that the closest field is proposed; at or above it no field is proposed. |
| The error cut-off | Not stated. | The row-by-row retry stops after ten errors, and only when the errors also exceed one per ten rows processed, so that a small file with a few errors still reports all of them. |
| The formula guard on export | Not stated. | A comma-separated cell beginning with an equals sign, a hyphen or a plus sign is prefixed with an apostrophe. This is observable behaviour of the exported file and a rebuild must reproduce it, or spreadsheet applications will evaluate exported text. |
| The refusal for an external identifier of the wrong entity | The working branch described the check in words: the lookup verifies that the row's entity is the one expected. | The declared text is reproduced verbatim in section 6.4: "Invalid external ID " the identifier ": expected model " the expected transport name between apostrophes ", found " the recorded transport name between apostrophes. Rule eight of the documentation rules requires the exact text. |
| The unknown sub-field guard | The working branch listed the message in its table of import messages without saying when it fires. | Reproduced in section 6.4 as "Unknown sub-field “" the name "”", together with the condition: it is the guard of the resolver, and an ordinary file is stopped earlier by the indirect-creation refusal of section 5.2, so only a caller that invents a referencing form reaches it. |
| The not-updatable flag of an existing identifier row | The working branch stated the flag's effect on a load but not its own lifetime. | Section 3.3 now states that the flag is written once, when the identifier row is first created, from the enclosing scope of the data file, and that a later load updates only the entity, the record identifier and the modification stamp of the row. This was checked against the conflict clause of the bulk assignment, which names those three columns and no other. |
| The caches an external identifier clears | The working branch said only that the resolution cache is cleared. | Section 2.4 now separates the two caches: the resolution cache, cleared on a change and on a deletion and written through on a bulk assignment, and the access-group cache, cleared in addition whenever the entry names the access-group entity, because the permission graph is derived from those entries. |
| A tabular data file of translation columns only | The working branch's procedure had no equivalent step. | Section 4.3 step 5 now states that a file left with no column after the translation columns are removed is skipped in silence. Without the step a rebuild would pass an empty field list to the import operation. |
