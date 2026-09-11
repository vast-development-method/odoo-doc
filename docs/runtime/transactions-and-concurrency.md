# Transactions and concurrency

Every unit of work in the platform runs inside exactly one database transaction, at the repeatable-read isolation level, with a retry loop around it that absorbs the serialization failures that this isolation level produces under contention. This document specifies the transaction boundary for each entry point, the connection handling, the isolation semantics and the transaction timestamp, the commit and rollback sequences with their four families of hooks, savepoints, the flush points of the unit of work, the retry loop with its attempt count and its backoff, the explicit row locks used for numbering and for job acquisition, the table lock used by file-store collection, the way concurrent edits on a record are detected and resolved, and the pattern that long-running operations use to commit in batches. The cache and recomputation machinery that the flush drives is specified in [`caching.md`](caching.md); this document specifies the boundary around it.

## 1. Transaction boundaries

| Entry point | Boundary | Commit | Rollback |
|---|---|---|---|
| A request served with a database | One transaction for the whole request, including routing resolution and authentication. | After the endpoint returned, by the retry wrapper. | On any error that escapes the endpoint. |
| A remote procedure call on the generic dispatch service | One transaction for the whole call. The credentials are verified inside it. | After the operation returned. | On any error. |
| A scheduled job | **Two** transactions: a *control* transaction that holds the job lock and records the outcome, and a *work* transaction, opened separately, in which the job body runs and commits as often as it wishes. | The control transaction commits once per job; the work transaction commits at every progress step. | Independently. |
| A message on the notification transport | One transaction per message, obtained from the pool with a bounded retry. | After the message was handled. | On any error. |
| A lifecycle callback of the notification transport | One transaction per callback set. | After the callbacks returned. | On any error. |
| The database-management service (create, duplicate, drop, restore) | Its own connections outside any entity registry. | Per operation. | Per operation. |
| A package installation or upgrade | One transaction per package loading pass, with an intermediate commit after each package's data files are loaded. | At the end of each package. | On any error, discarding the whole pass. |

A transaction is never shared between two requests, and a request never opens a second transaction except in the three places where the platform deliberately opens one: the read-only to read/write escalation (see [`request-lifecycle.md`](request-lifecycle.md), section 7.4), the device log insert on a read-only request (see [`sessions-and-authentication.md`](sessions-and-authentication.md), section 7), and the notification signalling that runs after commit (see [`notification-bus.md`](notification-bus.md), section 3).

## 2. Connections, cursors and pools

### 2.1 Two pools

The process keeps at most two connection pools: one for read/write connections to the primary, one for read-only connections to the read replica when the deployment declares one. Each pool has a maximum size, whose default is 64 connections; a separate, smaller default applies to the event-driven worker. A pool is created lazily on first use.

### 2.2 Borrowing a connection

Borrowing follows these steps in order.

1. Drop from the pool every connection that is not in use and whose last release is older than the idle timeout, whose default is 600 seconds, and close it.
2. Remove every connection that is already closed.
3. Reclaim every connection marked leaked: unmark it and make it available again.
4. Reuse the first available connection whose connection parameters are equal, comparing every parameter except the secret. Reset it before handing it out; if the reset fails, close it and continue looking.
5. If no reusable connection was found and the pool is at its maximum size, evict the first available connection. If none is available, fail with the message `The Connection Pool Is Full`.
6. Otherwise open a new connection.

Connection parameters are compared after normalizing the database-name alias and after removing the secret, therefore two spellings of the same target share connections.

### 2.3 Releasing a connection

Closing a cursor performs, in order: clearing the cursor's scratch cache, closing the underlying statement handle, rolling back (which runs the rollback sequence of section 4.2), and returning the connection to its pool.

A cursor that is collected without having been closed logs the warning `Cursor not closed explicitly` together with the place where it was opened, and marks its connection leaked instead of returning it; leaked connections are reclaimed at the next borrow, at step 3 of section 2.2.

A connection to one of the template or maintenance databases is never kept in the pool: it is closed on release, because keeping it would prevent those databases from being dropped.

### 2.4 The read-only fallback

A request for a read-only cursor when no replica is declared, or when the replica refused a connection within the last 1 200 seconds, returns a read/write cursor on the primary and marks the request's cursor mode as escalated. This is invisible to the caller except through that marker and the logs.

### 2.5 The scratch cache of a cursor

Each cursor carries a free-form scratch cache used by operations that must memoize a value for the life of one transaction and no longer. It is cleared by a commit, by a rollback and by closing the cursor. The catalogue of what is kept there is in [`caching.md`](caching.md), section 8.

## 3. Isolation and the transaction timestamp

### 3.1 Isolation level

Every cursor is opened at the **repeatable read** isolation level. The properties a replacement must preserve are those of snapshot isolation:

- Every statement in the transaction sees the same snapshot of the database, taken at the first statement.
- A read never blocks a write and a write never blocks a read.
- Two transactions that write the same row conflict: the second one to attempt the write waits for the first to end, and is then aborted with a serialization failure if the first one committed.
- Phantom reads do not occur within a transaction, but write skew does: two transactions may each read a set of rows, decide on that basis, and write disjoint rows, producing a state neither would have produced alone.

The platform deliberately does **not** use the strictest isolation level. Write skew is prevented where it matters by explicit row locks (section 7) rather than by the database's own conflict detection, because the explicit locks are cheaper and the aborts they produce are deterministic.

A read-only cursor is additionally marked read-only at the session level, therefore any attempted write fails with a dedicated read-only-transaction error rather than with a permission error. That error is the signal the request lifecycle uses to escalate.

### 3.2 The transaction timestamp

The transaction exposes a single timestamp, read once with a statement that returns the database server's current instant expressed in coordinated universal time, and memoized for the life of the transaction. The memo is cleared by a commit and by a rollback.

Every record created or written in a transaction receives the same creation or update instant, and every comparison against "now" inside one transaction uses the same value. This is what makes a batch of records created in one transaction share an instant exactly, and what makes a scheduled job's due-date comparison stable across the whole selection.

The transaction timestamp is what the creation instant, the update instant and the transient-record cleanup thresholds are all measured against.

Values written by the wall clock of the process, rather than by the transaction timestamp, exist in a few places and are called out where they occur; they can differ from the transaction timestamp by the duration of the transaction. The places are: the last-used instant of a registry, the failure instants of the login cooldown map, the receive instants of the persistent-socket rate limiter, and the elapsed-time budget of a scheduled job.

## 4. Hooks and the commit and rollback sequences

### 4.1 The four queues

Every cursor carries four ordered callback queues. Each queue also carries a free-form data dictionary that callbacks use to aggregate work; the dictionary is cleared when the queue runs.

| Queue | Runs | Typical use |
|---|---|---|
| Pre-commit | During the flush, after the unit of work has been applied, and again for as long as running it produced more work (bounded, see section 5). | Creating the aggregated records a whole transaction accumulated, such as notification rows and tracking rows. |
| Post-commit | Immediately after the database commit succeeded. | Side effects that must not happen unless the transaction is durable: sending the inter-process wake-up, sending queued electronic mail on a new transaction. |
| Pre-rollback | Immediately before the database rollback. | Reverting in-memory state that mirrors the database. |
| Post-rollback | Immediately after the database rollback. | Releasing external resources acquired for the aborted attempt. |

Adding the same callback twice queues it twice; queues run in insertion order and are emptied by running. The queues are **not** de-duplicated by any key. An operation that must act once for a whole transaction, however many times it is reached, therefore registers one callback and accumulates its work in the queue's data dictionary; the callback consumes the dictionary when it runs, and the dictionary is emptied at the same time.

### 4.2 The exact sequences

A commit performs, in this order:

1. Flush, following the bounded loop of section 5.
2. Commit the database transaction.
3. Clear the transaction: discard the cached field values, the dirty markers, the pending computations, the pre-commit queue and the cursor's scratch cache.
4. Forget the memoized transaction timestamp.
5. Clear the pre-rollback queue and the post-rollback queue without running them.
6. Run the post-commit queue.

A rollback performs, in this order:

1. Clear the transaction, exactly as at step 3 of the commit sequence.
2. Clear the post-commit queue without running it.
3. Run the pre-rollback queue.
4. Roll back the database transaction.
5. Forget the memoized transaction timestamp.
6. Run the post-rollback queue.

Two consequences follow and must be reproduced. A post-commit callback never runs when the transaction is rolled back, because the rollback clears that queue before doing anything else. A pre-rollback callback never runs when the transaction commits, for the symmetric reason.

## 5. The flush

The flush turns pending in-memory changes into statements. It repeats the following at most ten times:

1. Flush the transaction, which applies the dirty values and runs the pending computations in the order specified in [`caching.md`](caching.md), section 6.
2. If the pre-commit queue is empty, stop.
3. Run the pre-commit queue.

If the tenth iteration completes without the queue becoming empty, the warning `Too many iterations for flushing the cursor!` is logged and the flush returns anyway; the remaining callbacks stay queued and will run at the next flush or be discarded by a rollback.

The loop exists because a pre-commit callback may itself create records, which produces new dirty values and possibly new pre-commit callbacks.

Flush points are:

| Point | Trigger |
|---|---|
| Before a commit | Always. |
| Before a statement that reads a table with pending changes | The query builder flushes exactly the fields and tables the statement touches. |
| Before entering a flushing savepoint | Always (section 6). |
| Before a raw statement written by an operation | At the operation's discretion, by naming the fields to flush. |
| At the end of the serving function | By the retry wrapper, before the commit. |
| Before drawing from a no-gap numbering sequence | The counter field is flushed so that the locked row carries the pending value (section 7.4). |

Clearing, as opposed to flushing, discards the pending changes and the pre-commit queue without writing anything. Resetting additionally rebuilds the cached attributes of every execution context bound to the transaction and reattaches them to a freshly obtained registry; it is used after the entity registry has been reloaded and after a failed attempt that will be retried.

## 6. Savepoints

A savepoint is a named marker inside the transaction. Two variants exist.

| Variant | On entry | On normal exit | On exceptional exit |
|---|---|---|---|
| Plain | Declare the savepoint. | Release it. | Roll back to it, then release it. |
| Flushing (the default) | Flush first, then declare the savepoint. | Flush, then release; if that flush raises, roll back to the savepoint, release it and re-raise. | Clear the pending changes, roll back to the savepoint, then release it. |

The flushing variant is the one used by every operation that must be able to undo a sub-operation, because without the initial flush the pending in-memory changes would survive a rollback that undid their database counterpart, and without the clear on failure the in-memory state would describe a database state that no longer exists.

A savepoint may be rolled back to any number of times before being released. Releasing is unconditional and happens exactly once; a savepoint that has been released ignores every later request to roll back or release.

Nesting is unlimited. Each savepoint carries a freshly generated universally unique identifier as its name, therefore nested savepoints never collide and the name never has to be escaped against a value supplied by a user.

Savepoints are taken in these places:

| Place | Purpose |
|---|---|
| Around a whole data-loading pass | Undo the pass without losing the enclosing transaction. |
| Around each batch of a data-loading pass | Undo one batch and continue with the next. |
| Around each record of the retry pass of a data load | Attribute a failure to one record. |
| Around the constraint-application step of a schema update | Continue when one constraint cannot be applied. |
| Around the index-creation step of a schema update | Continue when one index cannot be created. |
| Around any operation that offers to undo a sub-operation | The general case; the flushing variant is used. |

## 7. Explicit row locking

### 7.1 The two lock strengths

| Strength | Meaning | Conflicts with | Used for |
|---|---|---|---|
| Exclusive | The strongest row lock, taken while skipping rows another transaction already holds. | Every other row lock, including the implicit shared lock that a foreign key takes to keep a referenced row alive. | Rows whose identity, and not only whose content, is being changed. |
| Exclusive but referenceable | The same, except that it does not block the creation of rows that reference the locked row. | Every other row lock **except** the implicit shared lock of a foreign key. | Rows whose non-identifying content is being changed while other transactions may still create rows referencing them. |

Both forms **skip** rows that are already locked instead of waiting for them. Waiting is deliberately avoided because it would hold the transaction open for an unbounded time and, with the fail-immediately alternative, would abort the whole transaction rather than the single operation.

One place uses a third variant, described in section 7.4: an exclusive lock that neither waits nor skips, and instead aborts the transaction at once when the row is already held.

### 7.2 The two locking operations

| Operation | Full name | Behaviour |
|---|---|---|
| `lock_for_update` | Lock the named records for update | Attempts to lock every named record. If the number of rows actually locked differs from the number requested, the whole operation fails with the message `Cannot grab a lock on records`. |
| `try_lock_for_update` | Lock whichever of the named records is free | Attempts to lock the named records, optionally up to a maximum count, and returns the subset that was locked. Records that do not exist yet in the database, because their values are still in memory, are counted as locked. When a maximum count is given, the records are locked in their declared order. |

The best-effort operation is the primitive used by every queue processor: it lets a worker take whatever is free and leave the rest to the other workers, with no waiting and no deadlock.

Both operations flush the fields their selection mentions before issuing it, exactly as a search does, so that a record whose value is still pending in memory is locked on its current row rather than on a stale one.

### 7.3 Where locks are taken

| Place | Strength | Effect of failure |
|---|---|---|
| Acquiring a scheduled job | referenceable | The job is skipped by this worker (see [`scheduled-jobs.md`](scheduled-jobs.md), section 4). |
| Writing to a Scheduled Action record | referenceable | The refusal `Record cannot be modified right now: This cron task is currently being executed and may not be modified Please try again in a few minutes`, reproduced exactly as the platform composes it. |
| Deleting a Scheduled Action record | exclusive | The same refusal. |
| Toggling a Scheduled Action from a settings switch | referenceable | The toggle is silently skipped, leaving the previous state. |
| Selecting outgoing text messages to send | exclusive | The locked ones are left to the other worker. |
| Selecting an incoming mail server to poll | referenceable | The server is skipped with the note `Skip checking for new mails on mail server <key> (unavailable)`, in which the placeholder is the external identifier of the server. |
| Selecting queued outgoing messages to send | referenceable | The locked ones are left to the other worker (see [`mail-gateway.md`](mail-gateway.md), section 3). |
| Drawing the next value of a no-gap sequence | exclusive, waiting disabled | The transaction aborts immediately; the caller must retry the whole transaction. This is the intended behaviour: a no-gap sequence is a true serialization point. |

### 7.4 Numbering sequences

Two implementations exist and they differ in their concurrency behaviour.

| Implementation | Storage | Concurrency | Gaps |
|---|---|---|---|
| Standard | A database sequence object outside transactional control | Never blocks; two transactions get two different values immediately | A rolled-back transaction leaves a gap |
| No gap | A counter column on the sequence record | The counter row is locked with waiting disabled; a second concurrent drawer aborts at once | No gap can be produced by a rollback, because the drawer holds the row until it commits; gaps can still appear if a numbered record is deleted afterwards |

Drawing a value follows these steps.

1. If the implementation is the standard one, ask the database sequence object for its next value and go to step 6.
2. Flush the counter field, so that any pending in-memory change to it reaches the row before the row is locked.
3. Read the counter of the sequence record under an exclusive lock taken with waiting disabled. If the row is already locked by another transaction, the read fails at once and the whole transaction is aborted.
4. Write the counter back as the value read plus the increment.
5. Invalidate the cached counter, so that a later read in the same transaction returns the written value.
6. Compose the result as the prefix, then the drawn number padded on the left with zeros to the declared padding width, then the suffix.

The prefix and the suffix are interpolated with date placeholders resolved in the request time zone. A malformed placeholder raises the refusal `Invalid prefix or suffix for sequence “<name>”`, in which the placeholder is the name of the sequence.

When a sequence uses date ranges, the sub-sequence covering the requested date is used; if none covers it, a new range is created whose bounds are the calendar year of the date, narrowed so as not to overlap the existing neighbouring ranges: the upper bound is pulled down to the day before the next range's start, and the lower bound is pushed up to the day after the previous range's end.

```formula
drawn number = counter before the update
counter after the update = counter before the update + increment
composed value = prefix + drawn number padded left with zeros to padding width + suffix
```

**Worked example.** A no-gap sequence has prefix `INV/`, padding 5, increment 1 and counter 41.

- Transaction A draws. It locks the counter row, reads 41, writes 42, and composes `INV/` followed by 41 padded to five digits, that is `INV/00041`.
- Transaction B, running at the same moment, attempts to draw. The lock is unavailable, the attempt fails at once and the whole of transaction B is aborted. The retry wrapper of section 8 replays it; by then A has committed, so B reads 42, writes 43 and composes `INV/00042`.
- Had A rolled back instead, the counter would be 41 again and B would compose `INV/00041`. No number is lost.

For the standard implementation the same scenario yields `INV/00041` for A and `INV/00042` for B with no waiting at all, and a rollback by A leaves `INV/00041` permanently unused.

### 7.5 A table-level lock

The file-store collection takes a share-mode lock on the attachment table (see [`attachments-and-file-store.md`](attachments-and-file-store.md), section 7). It sets a lock timeout of 10 seconds first, and abandons the whole collection when the lock cannot be taken in that time. The lock statement must be the first statement of its transaction, because a snapshot taken before it would not see the rows that concurrent transactions are inserting.

## 8. The retry loop

### 8.1 Which failures are retried

| Failure | Retried | Turned into |
|---|---|---|
| Serialization failure | yes | none |
| Deadlock detected | yes | none |
| Lock not available | yes | none |
| Explicit concurrency error raised by an operation | yes | none |
| Integrity violation (unique, check, foreign key, not-null) | **no** | A validation failure whose message is `The operation cannot be completed: <entity-specific explanation>` |
| Read-only transaction violation | not here | Handled one level up by the request lifecycle, which escalates the cursor |
| Anything else | no | Propagated unchanged |

The entity-specific explanation is produced by looking up the table named in the violation among the registered entities and asking that entity to translate the violation into a business message; when no entity owns the table, a generic explanation is used.

### 8.2 The loop

The unit of work is executed as follows. The attempt number starts at one.

1. Run the unit of work. If it returns, and the cursor is still open, flush the cursor, then go to step 9.
2. If the run failed with an integrity violation, an operational failure or an explicit concurrency error, continue at step 3. Any other failure is re-raised after resetting the transaction state and cancelling the registry invalidations of this attempt.
3. If the cursor is closed, re-raise the failure unchanged.
4. Roll back the transaction, reset the transaction state, which also reattaches every execution context to a freshly obtained registry, and cancel the registry invalidations produced by this attempt.
5. If a request is in scope, re-read the session from storage and rewind every uploaded file to its beginning. If one of them cannot be rewound, abandon the retry with the error `Cannot retry request on input file '<name>' after serialization failure`, in which the placeholder is the name of the uploaded file.
6. If the failure is an integrity violation, convert it into a validation failure as described in section 8.1 and raise that, without retrying.
7. If the failure is not one of the retryable kinds listed in section 8.1, re-raise it.
8. If the attempt number is five, log `<error name>, maximum number of tries reached!` at the informational level and re-raise the failure. Otherwise wait a uniformly distributed random duration between zero seconds and two raised to the power of the attempt number, expressed in seconds; log `<error name>, <tries left> tries left, try again in <wait> sec...` at the informational level, with the wait carried to four decimal places; increment the attempt number and go to step 1.
9. If any failure escaped steps 1 to 8, reset the transaction state and cancel the registry invalidations before propagating it.
10. If the cursor is still open, commit it. The commit runs the sequence of section 4.2, including the post-commit queue.
11. Signal the registry invalidations of the successful attempt to the other processes.
12. Return the result of the unit of work.

### 8.3 Properties

- **Five attempts** at most. The waits are drawn from the intervals zero to two, zero to four, zero to eight and zero to sixteen seconds, taken before attempts two, three, four and five respectively.

```formula
average total wait before the fifth attempt = 1 second + 2 seconds + 4 seconds + 8 seconds = 15 seconds
worst-case total wait before the fifth attempt = 2 seconds + 4 seconds + 8 seconds + 16 seconds = 30 seconds
```

- The wait is **random** in the interval, not fixed, which spreads competing workers instead of making them collide again.
- The registry invalidations of a failed attempt are **cancelled**, not applied: a failed attempt must not clear the caches of the other processes.
- The session is re-read from storage, because the failed attempt may have modified the in-memory session in ways that the database no longer reflects.
- Uploaded files are rewound, because the failed attempt may have consumed them.
- The successful attempt's invalidations are **signalled after the commit**, not before, therefore another process never learns of an invalidation caused by a transaction that later aborted.
- An integrity violation is never retried, because retrying it would produce the same violation and would only delay the answer by up to thirty seconds.
- **The unit of work is re-run from its original arguments.** A unit of work that modifies its own arguments would behave differently on the second attempt, therefore arguments must be treated as read-only by everything the retry loop wraps.

### 8.4 Worked example

Two requests confirm the same document. Both read state `draft`, both write state `confirmed`.

| Instant | Request A | Request B |
|---|---|---|
| 0 | Reads the row; snapshot taken. | Reads the row; snapshot taken. |
| 1 | Writes the row; holds the row lock. | Attempts to write the row; waits on the lock. |
| 2 | Flushes, commits. | The wait ends; the write is aborted with a serialization failure. |
| 3 | no activity | Attempt 1 fails. Rolls back, resets, cancels invalidations, re-reads the session, waits a random duration between zero and two seconds. |
| 4 | no activity | Attempt 2 runs from the first line of the endpoint. It now reads state `confirmed` and the guard refuses the transition with the business message the workflow defines. |

The client of request B therefore sees a business error, not a database error. This is the intended outcome: the retry makes the conflict visible as a business decision taken on fresh data.

## 9. Concurrent edits

### 9.1 What the platform does

The platform does **not** send an expected last-modification timestamp with a save and does not compare one on the server. A save that arrives second overwrites the fields it names, field by field. Two users editing different fields of the same record both succeed and both changes survive. Two users editing the same field: the later commit wins.

The three mechanisms that make this safe in practice are:

1. **Field-level writes.** A save writes only the fields the form actually changed, never the whole record. Two users working on different parts of the same document do not overwrite each other.
2. **State guards.** Every workflow transition re-reads the state inside the transaction and refuses the transition when the state is no longer the expected one. Combined with the retry of section 8, the second actor gets a business error rather than a silent overwrite, as in the example of section 8.4.
3. **Explicit locks.** Operations whose correctness depends on a read-then-write pair take the row lock of section 7 before reading.

### 9.2 The audit trail

Every record that supports it carries the update instant and the updating user. Those two fields are the record of who last changed a record and when. They are written from the transaction timestamp, therefore every record touched by one transaction shares one instant.

Records that carry a discussion thread additionally record, per tracked field, the value before and the value after, attached to a message in the thread (see [`logging-and-audit.md`](logging-and-audit.md), section 4). That is the mechanism by which a lost update is detected after the fact.

### 9.3 Industry-standard completion: optimistic concurrency on forms

The reference behaviour has no optimistic concurrency check on record saves. A replacement that wants one, which is standard practice for documents edited by several people, should implement it as follows, and must make it opt-in per entity in order that the observable behaviour of the entities specified in this repository stays unchanged.

1. The client sends, with the save, the update instant it last read.
2. The server takes the referenceable row lock and, inside the transaction, compares that instant with the stored update instant.
3. If the two instants are equal, the save proceeds unchanged.
4. If they differ, the save is refused with a message naming the user who changed the record and the instant of that change, and the response carries the current values of exactly the fields the client wanted to change, so that the client can show a comparison.

This is marked as an **industry-standard default**, not as observed behaviour.

### 9.4 Explicitly raised concurrency errors

Besides the failures the database itself reports, an operation may raise an explicit concurrency error to say "this attempt cannot succeed on this snapshot; replay it". Such an error is retryable exactly like a serialization failure, and the log line names the error rather than a database error code. Three properties make it safe to use:

1. It must be raised **before** any external side effect of the operation.
2. It must not be caught by the expression evaluator used by automation rules, which lets it travel up to the retry loop unchanged.
3. It must describe a condition that a fresh snapshot can resolve; a condition that is permanent must be raised as a business rule violation instead, because retrying it would only add up to thirty seconds of delay before the same answer.

### 9.5 Worked example: the flush loop

A transaction writes one order line. The write marks the order's total for recomputation and queues one pre-commit callback that creates the tracking message for the change.

| Iteration | What the flush does | What the pre-commit queue does |
|---|---|---|
| 1 | Writes the line, recomputes and writes the order total. | Creates the tracking message. Creating it marks the thread's last-message field for recomputation and queues a second callback, the notification fan-out. |
| 2 | Recomputes and writes the thread's last-message field. | Runs the notification fan-out, which creates notification rows and queues nothing further. |
| 3 | Writes the notification rows. | Empty, so the loop stops. |

Three iterations were used of the ten available. Had the chain continued past ten, the warning `Too many iterations for flushing the cursor!` would have been logged and the remaining callbacks would have stayed queued for the next flush.

## 10. Long-running operations

### 10.1 The pattern

An operation that must process more records than fit comfortably in one transaction follows these steps.

1. Select a bounded batch of candidate records, ordered deterministically so that two workers make the same selection.
2. Lock the batch with the best-effort locking operation of section 7.2, keeping only the records that were free.
3. For each record, or each sub-batch, of the locked set: do the work, commit, and report one unit of progress.
4. If more candidates remain after the batch, report the remaining count so that the caller can decide whether to run again at once.

The commit inside the loop is what makes the work durable incrementally. It also releases the row locks taken up to that point, therefore each batch must be small enough for one iteration to stay short.

### 10.2 Progress reporting

Inside a scheduled job the progress report is more than logging: it decides whether the job is finished, partially finished, or failed, and it decides whether the job is rescheduled at once or at its next interval. See [`scheduled-jobs.md`](scheduled-jobs.md), section 5. Outside a scheduled job the progress report degrades to a plain commit and reports an unlimited remaining time.

### 10.3 The consequences a replacement must accept

- **An operation that commits in batches is not atomic.** A failure in the middle leaves the earlier batches applied. Every such operation must therefore be idempotent at the granularity of one batch: re-running it must skip what is already done, which is what the state field or the locked-and-marked pattern provides. The catalogue of those patterns is in [`background-workers.md`](background-workers.md), section 3.
- **A retry cannot undo a committed batch.** The retry loop of section 8 wraps the unit of work; once a batch has committed, a later serialization failure replays only the remaining work.
- **External side effects must follow the commit, not precede it.** An operation that both writes a record and contacts an external service must commit the record first, otherwise a retry sends the message twice. Where the platform cannot follow that rule, it states the exception explicitly: sending a text message contacts an external service inside the transaction, and a retry may therefore send the message twice.

### 10.4 Worked example: the outgoing mail queue

One thousand messages are queued. The queue processor selects at most the batch size, whose default is 1 000, then:

| Step | Effect |
|---|---|
| 1 | Groups the selected messages by outgoing mail server and sender configuration. |
| 2 | For each group, opens one connection to the mail server. |
| 3 | For each message: writes a failure state **before** attempting delivery, attempts delivery, writes the success or failure state, then commits. |
| 4 | Reports one unit of progress after each commit. |

Writing the failure state before the attempt is deliberate: if writing the final state fails, for example because the transaction is aborted, the message stays in a failed state instead of staying queued, which prevents it from being sent a second time. The full specification of that queue is in [`mail-gateway.md`](mail-gateway.md), section 3.

## 11. Registry-level concurrency

Loading or upgrading the capability packages of a database is serialized across workers by a short critical section: the first worker to commit the marker proceeds with the upgrade; the others get a serialization failure, retry, find no marker left, and proceed as ordinary workers. A worker that fails inside that critical section does **not** reset the package states, precisely because another worker may be upgrading at that moment.

When a registry is reloaded or a cache container is cleared, the change is announced to the other processes by appending a row to a per-signal table; every process compares the largest row key it has seen with the current one at the start of every request. The mechanism is specified in [`caching.md`](caching.md), section 10.

## 12. Acceptance criteria

1. **Given** two transactions that write the same row, **when** the first commits, **then** the second is aborted with a serialization failure and is retried.
2. **Given** a transaction that fails with a serialization failure on every attempt, **when** the fifth attempt fails, **then** the failure is propagated, the message `<error name>, maximum number of tries reached!` is logged, and the total elapsed wait is at most 30 seconds.
3. **Given** a retry after a failed attempt, **when** the unit of work runs again, **then** it starts from its first statement, the session has been re-read from storage, and every uploaded file has been rewound.
4. **Given** a unit of work that uploads a file from a stream that cannot be rewound, **when** a serialization failure occurs, **then** the retry is abandoned with `Cannot retry request on input file '<name>' after serialization failure`.
5. **Given** a unique-constraint violation, **when** the unit of work fails, **then** it is **not** retried and the caller receives a validation failure whose message begins with `The operation cannot be completed:`.
6. **Given** a transaction that created three records, **when** they are read back, **then** all three carry the same creation instant.
7. **Given** a transaction that reads the current instant twice with one second of work in between, **when** both values are compared, **then** they are equal.
8. **Given** a flushing savepoint whose body raises, **when** the savepoint exits, **then** the pending in-memory changes made inside it are discarded and the database state is the one before the savepoint.
9. **Given** a pre-commit callback that creates records which themselves register a pre-commit callback, **when** the flush runs, **then** the loop runs until no callback remains or ten iterations have elapsed, in which case the warning `Too many iterations for flushing the cursor!` is logged.
10. **Given** a post-commit callback, **when** the transaction is rolled back instead of committed, **then** the callback does not run.
11. **Given** a pre-rollback callback, **when** the transaction commits, **then** the callback does not run and is discarded.
12. **Given** a no-gap sequence and two concurrent transactions drawing from it, **when** the second attempts to draw while the first holds the counter, **then** the second transaction aborts immediately and is retried by the retry loop.
13. **Given** a no-gap sequence at counter 41, **when** the drawing transaction rolls back, **then** the next drawer receives the number that would have been produced by the rolled-back transaction, namely `INV/00041` for prefix `INV/` and padding 5.
14. **Given** a standard sequence, **when** the drawing transaction rolls back, **then** the drawn number is never reused.
15. **Given** a Scheduled Action record currently being executed by a worker, **when** a user saves a change to it, **then** the save is refused with `Record cannot be modified right now: This cron task is currently being executed and may not be modified Please try again in a few minutes`.
16. **Given** a set of records of which half are already locked by another worker, **when** the best-effort locking operation is called, **then** only the free half is returned and no wait occurs.
17. **Given** a set of records of which one is already locked, **when** the strict locking operation is called, **then** the operation fails with `Cannot grab a lock on records`.
18. **Given** a queue processor that commits after each item, **when** it fails at item 7 of 10, **then** items 1 to 6 remain applied and items 7 to 10 remain queued.
19. **Given** a read-only cursor, **when** a write statement is attempted, **then** the failure is a read-only-transaction failure and the request lifecycle escalates to a read/write cursor rather than retrying on the read-only one.
20. **Given** a connection idle for longer than the idle timeout, **when** a new cursor is requested, **then** that connection is closed and a fresh one is opened.
21. **Given** a pool at its maximum size with every connection in use, **when** another cursor is requested, **then** the request fails with `The Connection Pool Is Full`.
22. **Given** a cursor that is garbage-collected without having been closed, **when** the next borrow runs, **then** the warning `Cursor not closed explicitly` has been logged and the connection has been reclaimed rather than lost.
23. **Given** two users saving different fields of the same record at the same time, **when** both transactions commit, **then** both changes are present and neither user receives an error.
24. **Given** two users confirming the same document, **when** the second transaction is retried, **then** it re-reads the state, finds it already confirmed and reports the workflow's own guard message.
25. **Given** a transaction holding the file-store collection lock, **when** a second collection run starts, **then** it waits at most 10 seconds and then abandons its run without error.
26. **Given** a transaction that commits, **when** the post-commit queue contains a callback that raises, **then** the database change is already durable and the failure is reported separately from the transaction outcome.

## 13. Reconciliation notes

1. The retry loop, the flush loop, the connection borrow and the sequence drawing were written as code-shaped sketches. They are restated as numbered procedures. The attempt count (five), the backoff intervals (zero to two, four, eight and sixteen seconds), the pool maximum (64 connections) and the idle timeout (600 seconds) are unchanged.
2. The lock strengths were named by their statement clauses. They are named by their behaviour instead: exclusive, exclusive but referenceable, and exclusive with waiting disabled. The skip-already-locked behaviour of the first two, and the abort-at-once behaviour of the third, are the observable properties and are stated in full.
3. The commit and rollback sequences were implied rather than stated. They are now enumerated step by step in section 4.2, which is what makes the "post-commit never runs on rollback" rule verifiable.
4. The unit of work and the record cache were specified in a separate document; every reference now points to [`caching.md`](caching.md), which holds both the cache layers and the recomputation engine.
5. One source stated that registering a transaction callback is idempotent when a key is supplied; another stated that adding the same callback twice queues it twice. The second is the behaviour: the queues are plain ordered lists with no de-duplication. The once-per-transaction pattern is achieved through the queue's data dictionary, and section 4.1 now says so.
