# Logging and audit

The system keeps five independent records of what happened: technical log entries written by the running code, the creation and update stamps carried by every persistent record, the field-by-field change history attached to documents that carry a discussion thread, the login and device history of every user, and, for the records that require it by law, a hash chain that makes a silent alteration detectable. A sixth, the performance profile, records how a request spent its time.

This document specifies each of them: what is written, by whom, when, where it is stored, who may read it, how long it is kept, and, for the hash chain, the exact algorithm and its verification procedure. The request-side hooks are in [`request-lifecycle.md`](request-lifecycle.md); the device history itself is in [`sessions-and-authentication.md`](sessions-and-authentication.md), section 7.

## 1. What is audited and what is not

| Question | Answer |
|---|---|
| Who created a record and when | Every persistent record, through its creation stamps. |
| Who last changed a record and when | Every persistent record, through its update stamps. |
| Which field changed from what to what | Only for tracked fields of entities that carry a discussion thread, specified in section 4. |
| Who read a record | Not recorded. There is no read audit. |
| Who logged in, from where, with which browser | Recorded, at the granularity of one row per user per hour per device, specified in section 5. |
| Who failed to log in | Written as a technical log line, not as a record. |
| Which operation a remote caller invoked | Logged at debug level only. The operation name is kept on the handling thread for the duration of the call and appears in the performance profile. |
| Whether a posted accounting record was altered afterwards | Detectable, for the records covered by a hash chain, specified in section 6. |

## 2. The Log Entry entity

### 2.1 Fields

The transport name is `ir.logging` and the full name is Log Entry. Records are ordered by key descending, so the newest is first.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `create_uid` | Created by | Integer, read-only | no | The key of the acting user, stored as a plain integer with **no** relation to the User entity. |
| `create_date` | Created on | Date and time, read-only | no | When the line was produced, in coordinated universal time. |
| `write_uid` | Last Updated by | Integer, read-only | no | Present for symmetry; never written after creation. |
| `write_date` | Last Updated on | Date and time, read-only | no | The same. |
| `name` | Name | Text | yes | The name of the logger that produced the line. |
| `type` | Type | Selection: `client`, `server`, indexed | yes | Where the line came from. The platform writes `server` lines; `client` is reserved for lines a client program submits. |
| `dbname` | Database Name | Text, indexed | no | The database the thread was serving. |
| `level` | Level | Text, indexed | no | The severity name. |
| `message` | Message | Long text | yes | The formatted message, with the traceback appended after a line break when there is one. |
| `path` | Path | Text | yes | Where the line was produced. |
| `func` | Function | Text | yes | The function that produced it. |
| `line` | Line | Text | yes | The line within that location. |
| `metadata` | Metadata | Structured value | no | Extra structured context, present only when the column exists in the target database. |

Privileged commands executed on behalf of another user are forbidden on this entity, so that a log entry can never be attributed to somebody else through a list-editing command.

### 2.2 Why the stamps are plain integers

The four stamp fields are declared explicitly as plain columns rather than as the usual relations, for two reasons a replacement must respect.

1. Entries are inserted with **direct statements** that bypass the record layer, and a deployment may direct them at a different database than the one being served. A relation to a user of another database would be meaningless.
2. When an entry is produced while the schema of the user table is being altered, the schema change holds an exclusive lock on that table. An insertion that had to validate a relation to it would block, and the package installation would deadlock against its own logging.

For the same reason, any pre-existing constraint linking the update stamp to the user table is neutralized at initialization by disabling the relevant trigger rather than by dropping the constraint, because dropping a constraint takes a stronger lock than disabling a trigger.

### 2.3 Who writes entries

| Producer | Type | Level | Name | Path, function and line |
|---|---|---|---|---|
| The database log handler, when the deployment routes technical logs to a database | `server` | The severity name of the record. | The logger name. | The source location of the call. |
| A code-based Server Action calling the logging helper available in its evaluation context | `server` | The level the caller passed, informational by default. | The action package name. | `action`, then the action key, then the action name. |
| An automation rule whose callback logging is enabled | `server` | `INFO`, or `ERROR` on a failure. | `Webhook Log` | The automation package name and the rule key in parentheses, then two empty values. |

### 2.4 The database log handler

1. Determine the target: the configured log database, or the database the current thread is serving. If there is none, write nothing.
2. Open a **separate** connection to the target.
3. Set a statement timeout of 1 000 milliseconds, which precludes a deadlock between the logging insert and the transaction that produced the line.
4. Build the message: the formatted message, and, when the record carries a traceback, a line break and the traceback.
5. Take the canonical name of the severity, deliberately not the record's own label, which a colouring formatter may have altered.
6. Insert one row with the creation instant taken from the database server, the type `server`, the served database name, the logger name, the level, the message, the source path, the source line and the function name, plus the metadata column when the target database has one.
7. Swallow every failure, silencing the statement logger while doing so.

Every failure is swallowed on purpose: technical logging must never fail a request.

The handler determines once, at construction, whether the target has a metadata column, unless the target is the placeholder meaning "the current database", which cannot be resolved until a request is being served.

### 2.5 Retention

The platform ships **no** retention for log entries. A deployment that routes technical logs to the database is responsible for pruning them. This is an explicit gap; the **industry-standard default** is to add a retention parameter expressed in days and a cleanup that deletes entries whose creation instant is older than that, which a replacement may implement without changing any other behaviour.

## 3. Record stamps

Every persistent entity that declares access logging carries four columns: the creating user, the creation instant, the last updating user and the last update instant. They are written by the record layer, never by business code, and they take the **transaction timestamp**, so every record touched by one transaction carries the same instant. The transaction timestamp is specified in [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 3.

They are the minimal audit of the system: they answer "who and when" for every record, including records that carry no discussion thread.

An entity may opt out of access logging. The entities that do are those written at very high frequency — the notification bus rows, the presence rows, the performance profiles and the log entries themselves — where the four extra columns would dominate the row.

## 4. Change tracking on documents

### 4.1 What is tracked

An entity that carries a discussion thread may declare fields as tracked. The set of tracked field names of an entity is every field of the entity that declares a tracking rank, together with the dynamic-property fields, the latter only when their defining parent changed.

The set is memoized in the registry's default cache container, per entity; the container is catalogued in [`caching.md`](caching.md), section 11.

### 4.2 The tracking procedure

On a write that touches at least one tracked field:

1. Capture the values of the tracked fields **before** the write, per record.
2. Perform the write.
3. For each record, compare the values before and after. Records that disappeared during the write are skipped.
4. For each record with at least one change:
   1. Determine the message subtype from the set of changed fields and their previous values. An entity decides this, typically returning a specific subtype when the state changed and nothing otherwise.
   2. Determine the body: the body a caller explicitly attached to this write, when there is one, even an empty one; otherwise the entity's own default summary for the changed fields.
   3. Determine the author: the author a caller explicitly attached; otherwise the acting user.
   4. If a subtype was determined and it still exists, post a message with that body, that author, that subtype and the change rows. Otherwise, if there are change rows, log a note with that body, that author and the change rows.

A subtype that no longer exists produces a debug note and the record is skipped entirely, so a removed subtype silently drops the tracking rather than failing the write.

The explicitly attached body and author are read from two per-transaction accumulators, keyed by entity name, that a caller fills before the write. They are consumed, that is removed, by the first tracking pass, so they apply to one write only.

### 4.3 The Field Change Tracking Value entity

One record per changed field per message. The transport name is `mail.tracking.value` and the full name is Field Change Tracking Value. Records are ordered by key descending, and the display name comes from the field link.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `field_id` | Field | Many-to-one to the field registry, set to nothing on deletion, indexed, read-only | no | Which field changed. It becomes empty when the field is removed from the system. |
| `field_info` | Removed field information | Structured value | no | A snapshot of the field's description, kept so the row stays readable after the field is removed. |
| `old_value_integer` | Old Value Integer | Integer, read-only | no | The value before, for an integer-shaped field. |
| `old_value_float` | Old Value Float | Decimal, read-only | no | The value before, for a decimal-shaped field. |
| `old_value_char` | Old Value Char | Text, read-only | no | The value before, for a short-text-shaped field. |
| `old_value_text` | Old Value Text | Long text, read-only | no | The value before, for a long-text-shaped field. |
| `old_value_datetime` | Old Value DateTime | Date and time, read-only | no | The value before, for a date-and-time-shaped field. |
| `new_value_integer` | New Value Integer | Integer, read-only | no | The value after, for an integer-shaped field. |
| `new_value_float` | New Value Float | Decimal, read-only | no | The value after, for a decimal-shaped field. |
| `new_value_char` | New Value Char | Text, read-only | no | The value after, for a short-text-shaped field. |
| `new_value_text` | New Value Text | Long text, read-only | no | The value after, for a long-text-shaped field. |
| `new_value_datetime` | New Value Datetime | Date and time, read-only | no | The value after, for a date-and-time-shaped field. |
| `currency_id` | Currency | Many-to-one to Currency, set to nothing on deletion, read-only | no | Set for monetary fields so the amounts can be displayed with their symbol. |
| `mail_message_id` | Message | Many-to-one to Message, cascading deletion, indexed | yes | The message the change belongs to. |

### 4.4 How each field type is stored

| Field type | Columns used | Conversion |
|---|---|---|
| Integer, decimal, short text, long text, date and time | The matching pair. | The raw values. |
| Monetary | The decimal pair, plus the currency taken from the record's currency field. | The raw amounts. |
| Date | The date-and-time pair. | Each date is combined with midnight and stored as an instant. |
| Boolean | The integer pair. | False becomes 0 and true becomes 1. |
| Closed list | The short-text pair. | The **label** of the value in the current language, falling back to the raw stored value when the label is unknown; an unset value becomes the empty text. |
| Link to one record | The short-text pair. | The display name of the record. |

A change on a field the registry does not know refuses the operation with `Unknown field <name> on model <entity>`, with the field name and the entity substituted.

### 4.5 Who may read a change row

Change rows are administrator-only at the entity level and are therefore normally read with elevated rights. Two filters exist for showing them to ordinary users.

| Filter | Rule |
|---|---|
| Access filter | A row linked to a field is visible when the reader has read access to that field of that entity. A row whose field has been removed is visible only to a settings user. |
| Free-access filter | A row is freely shareable only when it is linked to an existing field that declares no access group. This filter is used when a change summary is embedded in an outgoing notification, where the recipient's permissions cannot be evaluated at all. |

### 4.6 The observable result

A tracked change appears in the document's discussion thread as a message whose body is the summary, followed by one line per changed field showing the previous value, an arrow and the new value, with the field label. The message carries an author and an instant, so the thread is the human-readable audit trail of the document.

## 5. Login and device history

### 5.1 The User Login Log entity

One record per successful interactive login. It has **no** fields of its own: the creating user and the creation instant carry all the information.

| Aspect | Rule |
|---|---|
| Creation | One record per successful login, created with elevated rights, with no supplied values. |
| Why creation only | The record is only ever created, never updated, which avoids any interference with concurrent transactions touching the same user. |
| Default ordering | By key descending. |
| Compaction | A cleanup deletes every row for which a strictly more recent row exists for the same user, so only the latest login per user survives. |
| Derived use | The latest login shown on a user is the creation instant of their most recent row. The digest slow-down rule of [`background-workers.md`](background-workers.md), section 7.4, counts rows in a window. |

Because compaction keeps only the latest row per user, the login history is **not** a long-term audit trail. The technical log lines and the device history are.

### 5.2 Login log lines

| Event | Level | Line |
|---|---|---|
| Successful login | Informational | `Login successful for login:<login> from <address>` |
| Failed login | Informational | `Login failed for login:<login> from <address>` |
| Login refused by the cooldown | Warning | A line naming the client address, the attempted login, the database, the failure count and the instant of the last failure, followed by guidance naming the two cooldown parameters. |
| The cooled-down address is private | Warning | An additional line noting that the deployment may be misconfigured behind a proxy, because a private address means every client appears to be the same one. |
| Password change | Informational | `Password change for <login> (#<key>) by <actor login> (#<actor key>) from <address>` |
| Application key created | Informational | `<description> generated: scope: <<scope>> for '<login>' (#<key>) from <address>` |
| Application key removed | Informational | `API key(s) removed: scope: <<scopes>> for '<login>' (#<key>) from <address>` |
| Device log inserted | Informational | `User <key> inserts device log (<prefix>)` |
| Devices revoked | Informational | `User <key> revokes devices (<prefixes>)` |

### 5.3 Device history

The device history is specified in [`sessions-and-authentication.md`](sessions-and-authentication.md), section 7. It is the durable part of the login audit: one row per combination of session prefix, platform, browser and client address per hour, with the resolved country and city, kept until the session disappears and then marked revoked rather than deleted.

## 6. Inalterability hash chains

### 6.1 Purpose and scope

Several jurisdictions require that, once a sales or accounting record is finalized, any later alteration be detectable. The platform implements this with a **hash chain**: each covered record stores a digest computed over its own significant values and over the digest of the previous record in the same chain. Altering a record breaks every digest after it.

A chain is identified by a journal and a numbering prefix. Records are ordered within a chain by their sequence number.

A record is covered when it is posted **and** its journal has the inalterability mode enabled. A forced mode covers every posted record regardless of the journal setting.

### 6.2 The values that are hashed

Nine values enter the computation of one entry's digest: four of the entry itself and five of each of its lines. The key under which each value is placed in the serialized payload is contractual, because the payload is hashed byte for byte; the keys are therefore reproduced exactly here.

| Level | Value | Reproduced field identifier | Key in the serialized payload |
|---|---|---|---|
| The entry | Its name, that is its number | `name` | `name` |
| The entry | Its accounting date | `date` | `date` |
| The entry | Its journal | `journal_id` | `journal_id` |
| The entry | Its company | `company_id` | `company_id` |
| Each of its lines | Its label | `name` | `line_`, the line's numeric key, `_name` |
| Each of its lines | Its debit amount | `debit` | `line_`, the line's numeric key, `_debit` |
| Each of its lines | Its credit amount | `credit` | `line_`, the line's numeric key, `_credit` |
| Each of its lines | Its account | `account_id` | `line_`, the line's numeric key, `_account_id` |
| Each of its lines | Its counterparty | `partner_id` | `line_`, the line's numeric key, `_partner_id` |

No other value enters the payload, and no key other than these appears in it. Every line of the entry contributes its five keys, including a line with a zero debit and a zero credit.

Each value is rendered as text. A link to one record is rendered as the decimal text of the key of the linked record, and an empty link is rendered as the text `False`. A date is rendered as four-digit year, hyphen, two-digit month, hyphen, two-digit day. A monetary amount is rendered with exactly the number of decimal places of the record's currency, which is why the same figures produce different digests under currencies with different precisions.

### 6.3 The algorithm

Given the records in ascending sequence number and the digest of the record before the first of them:

1. Set the carry to the previous digest, or to the empty text when there is none.
2. For each record in turn:
   1. If the carry begins with the version marker, replace it by the part after the second marker separator. The version prefix does not take part in the computation.
   2. Build a map holding exactly the nine kinds of key of section 6.2 and nothing else: the four entry-level keys `name`, `date`, `journal_id` and `company_id`, each mapped to the rendered value of the corresponding entry value; then, for every line of the entry, the five line-level keys `line_<line key>_name`, `line_<line key>_debit`, `line_<line key>_credit`, `line_<line key>_account_id` and `line_<line key>_partner_id`, in which `<line key>` stands for the line's numeric key written in decimal, each mapped to the rendered value of the corresponding line value.
   3. Serialize the map as one line of text: an opening brace; then its entries in ascending order of key, separated by single commas with no spaces; each entry being the key between double quotation marks, then a single colon with no spaces, then the rendered value between double quotation marks; then a closing brace. There is no whitespace anywhere, no indentation and no line break. Any character outside the plain alphabet is escaped so that the serialization contains only plain-alphabet characters, which makes the payload independent of the text encoding a reader uses. The ordering is by the key text, so `company_id` precedes `date`, `date` precedes `journal_id`, every `line_` key precedes `name`, and lines sort by the decimal text of their key rather than by their numeric value.
   4. Compute the digest of the carry followed by the serialization, using the 256-bit member of the standard secure hash family, rendered in hexadecimal.
   5. The stored form is the marker `$`, the version number, the marker `$` and the digest.
   6. Assign the stored form to the record and set the carry to it.

The current version is **4**. The stored form carries the version as a prefix, which lets a verification reproduce the computation that produced an existing digest even after the value list has changed.

### 6.4 Building the chain to seal

Given a set of records to seal:

1. Group the set by journal, then by numbering prefix.
2. For each group, take the record with the greatest sequence number. If that record is not covered, skip the group.
3. Build the common condition: the same journal and the same numbering prefix.
4. Find the last sealed record: the record matching the common condition that has a digest, taking the greatest sequence number first.
5. Build the set to seal: every posted record matching the common condition that has no digest and whose sequence number is at or below the greatest one, restricted, unless the caller asks otherwise, to sequence numbers strictly greater than the last sealed one.
6. Take the previous digest from the last sealed record.
7. Raise the warnings below, then compute the digests as in section 6.3, assign them, and append the note `This journal entry has been secured.` to each record's discussion thread.
8. When any sealed record belongs to a journal whose inalterability mode is off, activate the access group that may view sealed entries, so that the resulting records are visible to somebody.

Warnings raised before sealing:

| Warning | Condition |
|---|---|
| Gap | The sequence numbers of the records to seal are not contiguous with the last sealed one. With a previously sealed record, the count of records to seal must equal the difference between the last sealed number and the last number to seal; without one, that difference is one less. |
| Unreconciled | At least one bank statement line among the records to seal is not reconciled. |
| No document | There is nothing to seal. |

### 6.5 What sealing forbids

Once a record carries a digest, writing any of the hashed values, or the digest itself, is refused with `This document is protected by a hash. Therefore, you cannot edit the following fields: <labels>.`, in which the placeholder is the comma-separated list of the labels of the fields the write touched.

A sealed record also loses its reset-to-draft action: that action is offered only for records without a digest that are either cancelled or posted without a pending cancellation request.

### 6.6 Verification

A verification recomputes the chain from its first sealed record and compares each stored digest with the computed one. Because older records may carry a digest produced by an earlier version of the computation, the verification reads the version from the stored prefix and applies the corresponding value list; a record with no prefix is verified with the earliest list.

A mismatch identifies the first altered record: every record before it verifies and every record from it onwards fails, because each computation carries the previous digest.

### 6.7 The point-of-sale variant

Point-of-sale sessions in the same jurisdictions seal their order totals with the same construction: a per-company sequence assigns a gapless number to each sealed record, the digest covers the order's totals and its sealed number, and the daily, monthly and annual sales closing jobs produce sealed period totals that chain onto each other. The three closing jobs are listed in [`scheduled-jobs.md`](scheduled-jobs.md), section 12.4.

## 7. Performance profiles

### 7.1 Enabling

Profiling is possible only while the system parameter `base.profiling_enabled_until` holds an instant in the future.

Enabling proceeds as follows.

1. If profiling is being switched on: take the limit from the parameter when its value is in the future, and nothing otherwise. Log `User <name> started profiling`, with the acting user's name substituted.
2. If there is no limit: clear the session's profiling marker. When the acting user is a settings user, return an instruction to open the assistant that enables profiling for a period; otherwise refuse with `Profiling is not enabled on this database. Please contact an administrator.`
3. If the session has no profiling marker: generate a new profiling session name derived from the user's name, store it with the expiration, and set the collector list and the parameter map to empty when none were supplied.
4. If profiling is being switched off: clear the marker.
5. If collectors were supplied, store them. If parameters were supplied, store them.
6. Return the marker, the collectors and the parameters.

The assistant offers four durations — 5 minutes, 1 hour, 1 day and 1 month — and writes the corresponding instant into the parameter.

The collectors and parameters live in the **session**, so they are client-defined. A replacement must treat them as untrusted input and validate them before use.

### 7.2 When a request is profiled

The decision is specified in [`request-lifecycle.md`](request-lifecycle.md), section 16. In short: the session must carry a marker, the database must be resolved, the recorded expiration must be in the future, the path must be neither the profiling-toggle path nor a notification-transport path, and the process must not be the event-driven worker.

### 7.3 The Performance Profile entity

The transport name is `ir.profile` and the full name is Performance Profile. Access logging is disabled on this entity, deliberately, to avoid a relation to the user table. Records are ordered by session descending, then by key descending.

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `create_date` | Creation Date | Date and time | When the profile was recorded. |
| `session` | Session | Text, indexed | The profiling session name, which groups the profiles of one investigation. |
| `name` | Description | Text | A description, normally the full path of the profiled request. |
| `duration` | Duration | Decimal with 3 decimal places | Elapsed wall-clock time. |
| `cpu_duration` | Processor Duration | Decimal with 3 decimal places | Processor time, excluding time spent waiting for other processes and for the database. |
| `init_stack_trace` | Initial stack trace | Long text, never prefetched | The call stack at the moment profiling started. Two profiles can be displayed together only when their initial stacks are identical. |
| `sql` | Statements | Long text, never prefetched | The recorded database statements with their timings. |
| `sql_count` | Queries Count | Integer | How many statements were recorded. |
| `traces_async` | Periodic traces | Long text, never prefetched | Stack samples taken at a fixed interval, each optionally carrying a memory reading. |
| `traces_sync` | Synchronous traces | Long text, never prefetched | Stack samples taken at instrumented points. |
| `others` | Other | Long text, never prefetched | Anything a custom collector recorded. |
| `qweb` | Template traces | Long text, never prefetched | Template rendering timings. |
| `entry_count` | Entry count | Integer | The total number of recorded entries. |

Three computed, non-stored values exist: the exportable flame representation, the address that opens it, and the address that opens the configuration form.

### 7.4 The exportable representation

A profile is converted into a flame representation on demand. The options, each a boolean unless stated:

| Option | Default | Effect |
|---|---|---|
| Constant time | false | Renders every frame with equal width instead of proportional to its duration. |
| Aggregate statements | false | Groups identical statements. |
| Use execution context | true | Uses the recorded execution context to label frames. |
| Combined | true when both statements and periodic traces exist | Produces one view mixing both. |
| Statements without gaps | true when statements exist and periodic traces do not | Produces a statement-only view with idle time removed. |
| Statement density | false | Produces a statement-only view showing density rather than duration. |
| Frames | true when periodic traces exist and statements do not | Produces a stack-only view. |
| Memory | true when every profile carries memory readings | Produces a memory view, baselined on the first reading. |
| Aggregation mode | `tabs` | `tabs` produces one view per profile; `temporal` produces one view over the whole set. |

Combining profiles whose initial stacks differ is refused with `All profiles must have the same initial stack trace to be displayed together.`

The memory view subtracts the first recorded reading from every reading, so it shows growth rather than absolute usage.

```formula
displayed memory at sample n = memory reading at sample n − memory reading at sample 1
```

**Worked example.** Three samples read 120.0, 138.5 and 131.25 mebibytes. The displayed series is 0.00, 18.50 and 11.25 mebibytes, carried to two decimal places, which shows that the request grew by 18.50 mebibytes at its peak and released 7.25 of them before it ended.

### 7.5 Retention

A cleanup deletes, in batches of 100 000, every profile created more than 30 days ago. It runs inside the automatic cleanup job of [`scheduled-jobs.md`](scheduled-jobs.md), section 11.

## 8. Server action history

Every execution of a code-based Server Action stores the executed body.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `action_id` | Action | Many-to-one to Server Action, cascading deletion | yes | The action that ran. |
| `code` | Code | Long text | no | The body that was executed. |

Records are ordered by creation instant descending, then by key descending. The display name is the creation instant rendered in the reader's language and time zone, followed by the name of the user who created the entry.

A cleanup keeps at most 100 entries per action, deleting the oldest beyond that.

This is the record of *what code ran*, which matters because the body of a code-based action can be changed between executions, so the action record alone does not say what a past execution did.

## 9. Automation rule logging

An automation rule triggered by an external callback may record each invocation as a Log Entry, as listed in section 2.3. When the recording is enabled it writes:

| Event | Level | Message |
|---|---|---|
| The callback arrived | `INFO` | `Webhook #<key> triggered with payload <payload>` |
| The record resolution failed | `ERROR` | `Webhook #<key> could not be triggered because the record_getter failed:` followed by the traceback |
| No record matched | `ERROR` | `Webhook #<key> could not be triggered because no record to run it on was found.` The caller receives `No record to run the automation on was found.` |
| The rule failed | `ERROR` | `Webhook #<key> failed with error:` followed by the traceback |

Every one of those is also emitted on the technical logger, at debug level for the first and at warning level for the other three, whether or not the recording is enabled. The recording therefore adds durability, not visibility.

## 10. Worked examples

### 10.1 A tracked state change

A sales order moves from `draft` to `sale`, and its expected delivery date changes at the same time.

| Step | Effect |
|---|---|
| 1 | Both fields are tracked. Their previous values are captured. |
| 2 | The write is applied. |
| 3 | The comparison finds two changes. |
| 4 | The entity's subtype rule returns the order-confirmed subtype, because the state changed. |
| 5 | A message is posted with that subtype, authored by the acting user, carrying two change rows. |
| 6 | The state change row stores the **labels** `Quotation` and `Sales Order` in its short-text columns, not the raw values `draft` and `sale`. The date change row stores the two instants in its date-and-time columns. |
| 7 | A user who may not read the delivery date field sees only the state row when the thread is displayed. |

### 10.2 Detecting an alteration in a sealed chain

A journal has five posted sealed entries with the sequence numbers 1 to 5.

| Step | Effect |
|---|---|
| 1 | Somebody alters the accounting date of entry 3 directly in the database, bypassing the application. |
| 2 | The verification recomputes. Entries 1 and 2 match. |
| 3 | Entry 3's recomputed digest differs from its stored digest, because the accounting date is one of the hashed values. |
| 4 | Entries 4 and 5 also differ, because each one's computation carries the previous digest. |
| 5 | The report names entry 3 as the first altered record. |

Had the alteration gone through the application, it would have been refused with the message of section 6.5.

### 10.3 A hash computation with numbers

An entry has the name `INV/2026/0007`, the accounting date `2026-02-14`, the journal key 3 and the company key 1, and one line whose numeric key is 91, with the label `Consulting`, a debit of 1200.00, a credit of 0.00, the account key 55 and no counterparty. The company currency has 2 decimal places. The previous record in the chain carries the stored form `$4$263f836f3d60b4fd118045b80a68d64e74652088508965a5108ca50ffff87ad1`.

| Step | Value |
|---|---|
| The carry after stripping the version prefix | `263f836f3d60b4fd118045b80a68d64e74652088508965a5108ca50ffff87ad1`, that is the digest part alone, without the marker and the version. |
| The map, key by key | `company_id` mapped to `1`; `date` mapped to `2026-02-14`; `journal_id` mapped to `3`; `line_91_account_id` mapped to `55`; `line_91_credit` mapped to `0.00`; `line_91_debit` mapped to `1200.00`; `line_91_name` mapped to `Consulting`; `line_91_partner_id` mapped to `False`; `name` mapped to `INV/2026/0007`. Nine keys, already written here in the ascending order the serialization uses. |
| The serialization, 211 characters | `{"company_id":"1","date":"2026-02-14","journal_id":"3","line_91_account_id":"55","line_91_credit":"0.00","line_91_debit":"1200.00","line_91_name":"Consulting","line_91_partner_id":"False","name":"INV/2026/0007"}` |
| The text that is hashed, 275 characters | The carry immediately followed by the serialization, with nothing between them: `263f836f3d60b4fd118045b80a68d64e74652088508965a5108ca50ffff87ad1{"company_id":"1","date":"2026-02-14","journal_id":"3","line_91_account_id":"55","line_91_credit":"0.00","line_91_debit":"1200.00","line_91_name":"Consulting","line_91_partner_id":"False","name":"INV/2026/0007"}` |
| The digest | `9553fbe1c14118a7bc599cd9c15e0530075e44971317c524c413493e1e715a4f` |
| The stored form | `$4$9553fbe1c14118a7bc599cd9c15e0530075e44971317c524c413493e1e715a4f` |

The arithmetic of the two monetary amounts is the only rounding in the computation:

```formula
rendered debit = debit amount in company currency rounded to (decimal places of the company currency) decimal places, written with exactly that many decimal places
             = 1200 rounded to 2 decimal places, written with 2 decimal places
             = 1200.00

rendered credit = credit amount in company currency rounded to (decimal places of the company currency) decimal places, written with exactly that many decimal places
              = 0 rounded to 2 decimal places, written with 2 decimal places
              = 0.00
```

Rounding is half away from zero and the trailing zeros are kept, because the rendering is fixed-width in the currency's precision rather than shortest-form. A currency declaring three decimal places would render the debit as `1200.000` and the credit as `0.000`, changing the serialization and therefore producing an entirely different digest for the same business figures.

Two further details decide the bytes. The empty counterparty renders as the text `False`, not as an empty text and not as a null, so the key `line_91_partner_id` is present with a four-character value. And the line key 91 is written in decimal inside the key, so a line numbered 100 would sort before a line numbered 91, the ordering being by key text and not by numeric value.

### 10.4 A profiling session

| Step | Effect |
|---|---|
| 1 | An administrator opens the assistant and chooses one hour. The parameter `base.profiling_enabled_until` is set to the current instant plus one hour. |
| 2 | A developer enables profiling on their own session. A profiling session name is generated and stored in their session, together with the expiration and empty collector and parameter sets. |
| 3 | Each of their requests, except those on the excluded paths, is profiled. One profile record is created per request, all carrying the same session name. |
| 4 | One hour later, the first request after the expiration clears the marker and logs `Profiling expiration reached, disabling profiling`. |
| 5 | Thirty days later the cleanup deletes the profiles. |

### 10.5 A code-based action whose body changed

| Step | Effect |
|---|---|
| 1 | A code-based Server Action runs on Monday. One history entry stores the body that ran. |
| 2 | An administrator edits the action's body on Tuesday. |
| 3 | The action runs again on Wednesday. A second history entry stores the new body. |
| 4 | An investigation on Thursday reads the two entries and can tell exactly which body produced which outcome, which the action record alone could not say. |
| 5 | After 100 further executions, the Monday entry is deleted by the cleanup and that distinction is lost, which is why the history is a bounded diagnostic aid rather than an audit trail. |

## 11. Acceptance criteria

1. **Given** a technical log line produced while the deployment routes logs to a database, **when** it is written, **then** exactly one Log Entry exists with the logger name, the canonical severity name, the source path, the function and the line, and with the traceback appended to the message when there was one.
2. **Given** a failure while writing a Log Entry, **when** it occurs, **then** the failure is swallowed and the request that produced the line is unaffected.
3. **Given** a Log Entry insertion, **when** it runs, **then** it runs on a separate connection with a statement timeout of 1 000 milliseconds.
4. **Given** a target database with no metadata column, **when** an entry is written, **then** the metadata column is not named in the insert.
5. **Given** a code-based Server Action calling the logging helper, **when** it executes, **then** a Log Entry exists whose path is `action`, whose function is the action key and whose line is the action name.
6. **Given** a write on three records in one transaction, **when** their stamps are read, **then** all three carry the same update instant.
7. **Given** an entity with a tracked field and a discussion thread, **when** that field is written, **then** a message exists carrying one change row per changed tracked field.
8. **Given** a write that changes only untracked fields, **when** it completes, **then** no message and no change row are created.
9. **Given** a write that changes a tracked field on a record deleted later in the same transaction, **when** the comparison runs, **then** that record is skipped and no change row is created for it.
10. **Given** a tracked closed-list field, **when** it changes, **then** the change row holds the labels of the previous and new values in the current language, not the raw stored values.
11. **Given** a tracked monetary field, **when** it changes, **then** the change row holds the amounts in its decimal columns and names the record's currency.
12. **Given** a tracked date field, **when** it changes, **then** the change row holds the two dates combined with midnight in its date-and-time columns.
13. **Given** a tracked boolean field changing from false to true, **when** the change row is read, **then** the integer columns hold 0 and 1.
14. **Given** a tracked field whose definition is later removed from the system, **when** the change row is read, **then** its field link is empty, its snapshot of the field description is still readable, and only a settings user may see the row.
15. **Given** a change row for a field the reader may not read, **when** the thread is displayed, **then** that row is omitted.
16. **Given** a change row for a field that declares an access group, **when** a change summary is embedded in an outgoing notification, **then** that row is omitted by the free-access filter.
17. **Given** a caller that attached a body and an author before a write, **when** two tracked writes follow, **then** only the first one uses them.
18. **Given** a successful interactive login, **when** it completes, **then** exactly one login log record is created and the line `Login successful for login:<login> from <address>` is emitted.
19. **Given** a user with twelve login records, **when** the compaction runs, **then** exactly one record remains, the most recent one.
20. **Given** a posted entry in a journal with the inalterability mode enabled, **when** it is sealed, **then** it carries a digest prefixed by the version marker and the note `This journal entry has been secured.` appears in its thread.
21. **Given** a sealed entry, **when** a user edits its accounting date, **then** the write is refused with `This document is protected by a hash. Therefore, you cannot edit the following fields: <labels>.`
22. **Given** a sealed entry, **when** the form is displayed, **then** no reset-to-draft action is offered.
23. **Given** a chain of five sealed entries in which the third is altered outside the application, **when** the chain is verified, **then** the first two verify and the third is reported as the first altered record.
24. **Given** a group to seal whose sequence numbers are not contiguous with the last sealed number, **when** sealing is attempted, **then** the gap warning is raised.
25. **Given** a group to seal containing an unreconciled bank statement line, **when** sealing is attempted, **then** the unreconciled warning is raised.
26. **Given** an empty set to seal, **when** sealing is attempted, **then** the no-document warning is raised.
27. **Given** a monetary value of 1200 in a currency with 2 decimal places, **when** the digest is computed, **then** the rendered value is `1200.00`; in a currency with 3 it is `1200.000` and the digest differs.
28. **Given** a stored digest that carries no version prefix, **when** the chain is verified, **then** the earliest value list is used for that record.
29. **Given** the entry of section 10.3 and the carry stated there, **when** the digest is computed, **then** the serialized payload is exactly the 211-character text of that section, the hashed text is exactly the 275-character concatenation of the carry and that payload, and the stored form is `$4$9553fbe1c14118a7bc599cd9c15e0530075e44971317c524c413493e1e715a4f`.
30. **Given** the same entry with a second line whose numeric key is 100, **when** the digest is computed, **then** the payload carries fourteen keys and `line_100_account_id` sorts before `line_91_account_id`, because keys are ordered as text.
31. **Given** an entry line with no counterparty, **when** the digest is computed, **then** its `line_<line key>_partner_id` key is present in the payload with the value `False`, not omitted and not empty.
32. **Given** the profiling parameter unset and a settings user asking to profile, **when** the request is served, **then** the assistant for enabling profiling is returned rather than a refusal.
33. **Given** the profiling parameter unset and a user who is not a settings user asking to profile, **when** the request is served, **then** the refusal `Profiling is not enabled on this database. Please contact an administrator.` is produced.
34. **Given** an active profiling session whose expiration has passed, **when** the next request is served, **then** the marker is cleared, the warning `Profiling expiration reached, disabling profiling` is logged, and the request is not profiled.
35. **Given** two profiles whose initial stacks differ, **when** they are displayed together, **then** the operation is refused with `All profiles must have the same initial stack trace to be displayed together.`
36. **Given** three memory samples of 120.0, 138.5 and 131.25 mebibytes, **when** the memory view is produced, **then** the displayed series is 0.00, 18.50 and 11.25.
37. **Given** a profile created 31 days ago, **when** the cleanup runs, **then** it is deleted.
38. **Given** a Server Action with 130 history entries, **when** the cleanup runs, **then** exactly 100 remain, the most recent ones.
39. **Given** an automation rule with callback logging enabled whose record resolution fails, **when** the callback arrives, **then** a Log Entry at the level `ERROR` exists naming the rule key and carrying the traceback, and the caller receives a failure.
40. **Given** an automation rule with callback logging **disabled** whose rule body fails, **when** the callback arrives, **then** no Log Entry is created and the technical logger still emits the warning.

## 12. Reconciliation notes

1. The field identifiers of the four entities were paraphrased. They are reproduced exactly: for the Log Entry, `create_uid`, `create_date`, `write_uid`, `write_date`, `name`, `type`, `dbname`, `level`, `message`, `path`, `func`, `line` and `metadata`; for the Field Change Tracking Value, `field_id`, `field_info`, the five old-value columns, the five new-value columns, `currency_id` and `mail_message_id`; for the Performance Profile, `create_date`, `session`, `name`, `duration`, `cpu_duration`, `init_stack_trace`, `sql`, `sql_count`, `traces_async`, `traces_sync`, `others`, `qweb` and `entry_count`; and for the server action history, `action_id` and `code`.
2. The change-tracking value columns were described as five typed pairs with one text column. There are in fact two text-shaped pairs, a short-text one and a long-text one, and the table of section 4.4 says which field type uses which.
3. The log line emitted when an application key is removed was paraphrased with the credential's name in full words. It is contractual text and is reproduced verbatim in section 5.2.
4. The digest algorithms were named by their family. They are described by their bit length and family, because a rebuild must reproduce the algorithm and not a library.
5. The memory view was described without arithmetic. The subtraction and a worked example are in section 7.4.
6. The absence of a retention rule for log entries was stated as an explicit gap. It is now marked as an **industry-standard default** with the completion a replacement should implement, as the documentation rules require.
7. The reference to the document that owns the digest slow-down rule was renamed: that rule is in [`background-workers.md`](background-workers.md), section 7.4.
8. The keys of the map that is serialized and hashed were described by their meaning rather than reproduced. Because the digest is the hash of the key-sorted serialization of that map, the keys are part of the arithmetic and a rebuild cannot reproduce a single stored digest without them. They are reproduced in section 6.2 and repeated in the procedure of section 6.3: `name`, `date`, `journal_id` and `company_id` for the entry, and `line_<line key>_name`, `line_<line key>_debit`, `line_<line key>_credit`, `line_<line key>_account_id` and `line_<line key>_partner_id` for each line.
9. The entry-level company key was given as `company`. The reproduced identifier, and therefore the key in the payload, is `company_id`.
10. The worked example of section 10.3 stated the shape of the serialization without producing it. It now carries the exact 211-character payload, the exact 275-character text that is hashed, the resulting digest and the stored form, so that the example can be replayed byte for byte.
