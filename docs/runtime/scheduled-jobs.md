# Scheduled jobs

A scheduled job is a server action that the platform executes on its own, on a repeating interval or on demand, without a client request. This document specifies the three entities that describe jobs, triggers and progress; the worker loop that polls for work and the wake-up channel that shortens that poll; how a due job is selected, locked and skipped; the two transactions a job runs in; the inner loop that lets a job process many batches in one acquisition; the progress protocol that decides whether a job is finished, partially finished or failed; how the next execution instant is computed, including across a daylight-saving change; how repeated failures deactivate a job; how a timeout is detected and counted; and the complete catalogue of the jobs the platform and every capability package ship, with their interval, their default activation and their purpose.

## 1. The three entities

### 1.1 Scheduled Action

A Scheduled Action **is** a Server Action with extra scheduling fields: the two records share one identity, and the Server Action supplies the name, the target entity and the body to execute.

| Identifier | Full name | Type | Required | Default | Copied | Meaning |
|---|---|---|---|---|---|---|
| `server_action` | Server action | link to one Server Action, deletion restricted, indexed | yes | none | yes | The delegated action. Its name, target entity, usage and body belong to the job. The usage is forced to the scheduled-action usage on creation. |
| `job_name` | Job name | text, stored, derived | yes | derived | yes | A copy of the server action's name, read in the source language so that ordering and logging are language-independent. Recomputed whenever the action name changes. |
| `scheduler_user` | Scheduler user | link to one User | yes | the creating user | yes | The user whose permissions and whose time zone apply while the job runs. |
| `active` | Active | boolean | no | true | yes | An inactive job is never selected, and a trigger for it is ignored unless it is in the future. |
| `interval_number` | Interval number | integer | yes | 1 | yes | The repetition count. Must be strictly positive; the constraint message is `The interval number must be a strictly positive number.` Aggregated as an average in grouped views. |
| `interval_type` | Interval unit | selection: `minutes`, `hours`, `days`, `weeks`, `months` | yes | `months` | yes | The repetition unit. |
| `next_execution_on` | Next execution instant | date and time | yes | the current instant | yes | The next planned execution instant. |
| `last_execution_on` | Last execution instant | date and time | no | none | yes | The instant of the last execution that was not a failure. Handed to the job body in its context. |
| `priority` | Priority | integer | yes | 5 | yes | 0 is the highest priority, 10 the lowest. Not aggregated. |
| `failure_count` | Consecutive failures | integer | yes | 0 | yes | Consecutive failures. Reset on any non-failing outcome. |
| `first_failure_on` | First failure instant | date and time | no | none | yes | The instant of the first failure of the current streak. Reset on any non-failing outcome. |

Default ordering: by job name, then by key. Privileged commands executed on behalf of another user are forbidden on this entity.

When a job is created through the interface, the body kind defaults to the executable-code kind, because that is the only kind a scheduled job supports.

### 1.2 Scheduled Action Trigger

A one-shot request to run a job at a given instant, independent of its interval.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `scheduled_action` | Scheduled action | link to one Scheduled Action, deletion cascading, indexed | yes | The job to run. |
| `call_at` | Call instant | date and time, indexed | yes | The instant from which the job becomes due. |

Several triggers may exist for one job. Triggers whose instant has passed are deleted when the job starts (section 5.1). Privileged commands executed on behalf of another user are forbidden on this entity.

### 1.3 Scheduled Action Progress

One record per inner-loop iteration of one job execution. It is the channel through which a job body tells the scheduler how much work it did and how much is left.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `scheduled_action` | Scheduled action | link to one Scheduled Action, deletion cascading, indexed | yes | none | The job. |
| `done` | Units done | integer | no | 0 | Units of work completed in this iteration. |
| `remaining` | Units remaining | integer | no | 0 | Units of work still to do after this iteration. |
| `deactivate` | Deactivate on completion | boolean | no | false | When true and the iteration completes, the job is deactivated. |
| `timed_out_counter` | Consecutive interruptions | integer | no | 0 | How many consecutive executions were interrupted before they could report. |

## 2. The worker loop

### 2.1 Shape

A deployment runs a fixed number of job workers; the shipped default is 2. Each worker repeats the following steps.

1. Wait until either a wake-up arrives on the notification channel or the poll interval elapses. The interval is 60 seconds plus a per-worker offset, and is followed by a short additional pause proportional to the worker's ordinal.
2. Collect the database names carried by the wake-ups received since the last iteration, then clear them.
3. If at least 60 seconds have elapsed since the last full listing, relist every visible database and order the work with the notified databases first, in the order notified, then the remaining databases. Otherwise work only on the notified databases that are known to be visible; if that set is empty, go back to step 1.
4. For each database in that order, process its due jobs as specified in section 3.

The per-worker offset on the interval and the proportional pause exist to spread the workers in time. Without them, a wake-up would make every worker poll the database at the same instant.

A worker whose lifetime exceeds a configured maximum releases its connection and starts a fresh one; when that maximum is zero, the worker lives for the life of the process.

### 2.2 The wake-up channel

Workers subscribe to a single named notification channel on the database server. The payload of a notification is the name of the database whose jobs should be examined. Subscription is skipped, with the warning `PG cluster in recovery mode, cron trigger not activated`, when the server is a standby; in that case the workers fall back to the 60-second poll.

A wake-up is emitted, after the commit of the transaction that caused it, when:

- a trigger is created whose instant is now or in the past;
- a job reports itself partially finished and is therefore rescheduled immediately;
- always, for any trigger creation and for any change of the next execution instant or of the active flag, when the deployment enables eager notification.

### 2.3 Per-database guards

Before selecting any job on a database, two guards run on the control transaction.

| Guard | Rule | Outcome when it fails |
|---|---|---|
| Version | The recorded version of the platform's base package must equal the version of the running code. | The whole database is skipped with the warning `Skipping database <name> as its base version is not <version>.` A recorded version that is unset means the database is mid-installation, and produces the package-state outcome instead. |
| Package state | No package may be in a transient state, that is queued for installation, upgrade or removal. | If there are no due jobs, the database is skipped. Otherwise the oldest due job is examined: its reference instant is the later of its next execution instant and its last update instant, and the oldest such instant across all due jobs is taken. If that instant is less than 18 000 seconds (5 hours) old, the database is skipped with the warning `Skipping database <name> because of modules to install/upgrade/remove.`. If it is older, the transient states are assumed to be stale, they are forcibly reset, and processing continues. |

Other outcomes:

| Condition | Behaviour |
|---|---|
| The job table does not exist | Warning `Tried to poll an undefined table on database <name>.`; the database is skipped. |
| A statement is malformed | Propagated; the worker's supervisor handles it. |
| Any other failure | Warning `Exception in cron:` with the traceback; the database is skipped for this round. |

## 3. Selecting due jobs

On the control transaction, with the transaction timestamp as "now", a job is due when both of the following hold:

1. its active flag is true; and
2. either its next execution instant is at or before now, or a trigger exists for it whose call instant is at or before now.

The due jobs are ordered by consecutive failure count ascending, then by priority ascending, then by key ascending. The ordering means that jobs that have never failed run before jobs that have, and among equals the lowest priority number runs first. A job that keeps failing therefore drifts to the end of the queue and stops starving the healthy ones.

The full list of due job keys is captured **once**, at the start of the round. Jobs that become due while the round is running are picked up by the next round.

## 4. Acquiring one job

For each key in the captured order, on the control transaction, the worker reads every column of the job joined with its **most recent** progress record, that is the one with the largest key, taking from it the progress key, the consecutive-interruption counter, the done count and the remaining count. The read is restricted to that one job, re-applies the due condition of section 3, and takes the exclusive but referenceable row lock while skipping rows another transaction already holds.

| Result | Behaviour |
|---|---|
| One row | The job is acquired; the worker holds the row lock until its control transaction commits. |
| No row | Another worker holds the lock, or the job is no longer due. Debug note `job <key> is being processed by another worker, skip`; move to the next key. |
| A serialization failure | Another worker committed a new next-execution instant an instant before this read. The control transaction is rolled back, a debug note `job <key> has been processed by another worker, skip` is emitted, and the round continues with the next key. |

The lock strength is exclusive but referenceable rather than fully exclusive, precisely in order that other transactions may keep creating triggers that reference the job while it runs.

The three progress values (done, remaining, consecutive interruptions) are normalized to 0 when there is no progress record yet.

After acquisition, the registry's signalling check runs, which guarantees that a job body always executes against an up-to-date registry.

## 5. Processing one job

Everything in this section happens on the **control** transaction, which commits exactly once, at the end, releasing the lock.

### 5.1 Clearing the schedule

Every trigger of this job whose instant is at or before the current instant is deleted. Triggers in the future survive. This is done before the body runs, therefore a body that creates a trigger for itself with an instant in the past is honoured on the next round rather than being erased.

### 5.2 Timeout short-circuit

The execution is short-circuited when the consecutive-interruption counter read at acquisition is at least 3 **and** the done count read at acquisition is 0.

When that holds, the body is **not** executed. The outcome is `failed`, the consecutive-interruption counter of the last progress record is reset to 0, and the error `Job '<name>' (<key>) timed out` is logged, in which the placeholders are the job name and the job key. This prevents a job that reliably exhausts the worker's time limit from consuming the workers forever.

### 5.3 Executing the body

Otherwise the body runs as specified in section 6, producing one of three outcomes: `fully done`, `partially done` or `failed`.

### 5.4 Recording the outcome

The failure counters are updated (section 7), then:

| Outcome | Rescheduling |
|---|---|
| `fully done` | Later, at the next interval boundary (section 8.1). |
| `failed` | Later, at the next interval boundary (section 8.1). |
| `partially done` | Immediately: a trigger is created for the current instant (section 8.2), and a wake-up is emitted after the commit when eager notification is enabled. |

The control transaction then commits, which releases the lock and makes the new next-execution instant, the new failure counters and the new triggers visible to the other workers.

## 6. Executing the body

### 6.1 The work transaction

A **second, separate** transaction is opened for the body. Its execution context carries three values the body may read.

| Context value | Meaning |
|---|---|
| Last execution instant | The job's previous non-failing execution instant. |
| Job key | The key of the running job, used to validate progress reports. |
| Deadline | The monotonic instant 10 seconds after the start of the execution, used by the progress operation to tell the body how much time is left. |

The acting user is the job's scheduler user; the context therefore carries that user's language and time zone.

### 6.2 The inner loop

The loop begins by taking the consecutive-interruption counter read at acquisition, setting the iteration count to zero and the status to none, and logging `Job '<name>' (<key>) starting`. It then repeats the following while the status is still none **and** either the iteration count is below 10 or the current monotonic instant is before the deadline.

1. Create a progress record with a done count of 0, a remaining count of 0, and a consecutive-interruption counter one greater than the value carried into this iteration.
2. Commit the work transaction, which makes the progress row durable before any work is attempted.
3. Run the server action as specified in section 6.4. Record whether it returned without raising.
4. If it raised, log the exception under `Job '<name>' (<key>) server action #<action key> failed`.
5. Whether it raised or not: read the done count and the remaining count from the progress record, apply the decision table of section 6.3, increase the iteration count by one, set the progress record's consecutive-interruption counter to 0, carry 0 forward as the counter for the next iteration, commit the work transaction, and emit the debug note `Job '<name>' (<key>) processed <done> records, <remaining> records remaining`.

When the loop ends, a status that is still none becomes `partially done`. The line `Job '<name>' (<key>) <status> (#loop <iterations>; done <done>; remaining <remaining>; duration <seconds>s)` is logged and the work transaction is closed.

The loop therefore runs **at least ten iterations**, and continues past ten only while fewer than 10 seconds have elapsed. A job that does not use the progress protocol at all reports a remaining count of 0 after its first iteration and stops there.

Setting the progress record's consecutive-interruption counter to 0 at the end of each iteration is what makes the counter measure *consecutive interruptions*: an iteration that completed always clears it, and only an iteration that never reached its end leaves the incremented value behind.

### 6.3 The decision table

"Succeeded" means the body returned without raising; the done and remaining counts are the values the body reported for this iteration.

| Succeeded | Done | Remaining | Decision | Note |
|---|---|---|---|---|
| no | greater than 0 | greater than 0 | keep looping | The body failed but committed some progress; the failure is assumed transient. |
| no | any other combination of done and remaining | | `failed` | Even if a previous iteration had progressed. |
| yes | any | 0 | `fully done` | Also covers bodies that never report progress. If the body asked for deactivation, the job's active flag is set to false. |
| yes | 0 | greater than 0 | `partially done` | When this is the first iteration, the warning `Job '<name>' (<key>) processed no record` is logged. |
| yes | greater than 0 | greater than 0 | keep looping | Ordinary batch progress. |

When the loop ends because the iteration budget and the time budget are both exhausted, the outcome is `partially done`.

### 6.4 Running the server action

One invocation of the server action proceeds as follows.

1. If the registry changed since this worker acquired the job, reset the transaction state, which rebinds the body to the new registry.
2. Run the server action.
3. Flush the work transaction.
4. Signal the registry invalidations of this iteration.
5. Commit the work transaction.
6. On any failure at steps 2 to 5: cancel the registry invalidations, roll back the work transaction, and re-raise.

The commit at step 5 is what makes each iteration durable independently. A job body that raises after having committed inside itself keeps the committed part.

### 6.5 The progress operation

A body reports progress by calling the progress operation, which also commits. It takes a processed count defaulting to zero, an optional remaining count, and a deactivation flag defaulting to false, and it returns the number of seconds left in the time budget. It behaves as follows.

1. If the execution context carries no progress record, the call is not running as a job: commit and return an unbounded budget.
2. Refuse a negative processed count and a negative remaining count.
3. Refuse a progress record that does not belong to the job named in the execution context.
4. If no remaining count was supplied, the new remaining count is the previous remaining count minus the processed count, floored at zero. If one was supplied, it replaces the previous one.
5. The new done count is the previous done count plus the processed count.
6. Write the done count, the remaining count, and the deactivation flag when it was asked for.
7. Commit.
8. Return the deadline minus the current monotonic instant, floored at zero.

Properties:

- The processed count is **cumulative within the iteration**: each call adds to the iteration's done count.
- Omitting the remaining count subtracts the processed count from the previous remaining count, floored at zero. Supplying it replaces it.
- The returned value is the remaining time budget in seconds; a body uses it to decide whether to start another batch. A return value of 0 means the budget is exhausted.
- Calling it outside a job is a plain commit returning an unbounded budget, which is what makes the same operation usable both from a job and from an interactive call.

## 7. Failure counting and deactivation

After the body has produced its outcome, the counters are updated as follows.

1. If the outcome is `failed`: take the transaction timestamp truncated to the second as now; set the failure count to the previous count plus one; set the first-failure instant to the previous one when there was one, otherwise to now.
2. Still in the failure branch, if the new failure count is at least 5 **and** the first-failure instant plus seven days is earlier than now: set the failure count to 0, unset the first-failure instant, set the active flag to false, and notify an administrator.
3. If the outcome is not `failed`: set the failure count to 0, unset the first-failure instant, and leave the active flag unchanged except for a deactivation the body itself asked for.
4. Write the failure count, the first-failure instant and the active flag with a direct statement, which bypasses the modification protection of section 8.5 because the worker already holds the lock.

**Both** thresholds must be met: at least five consecutive failures **and** at least seven days between the first failure of the streak and now. A job that fails five times in one hour is not deactivated; a job that fails once a week for eight weeks is.

The administrator notification message is `Cron job <name> (<key>) has been deactivated after failing 5 times. More information can be found in the server logs around <instant>.`, in which the placeholders are the job name, the job key and the instant of the last failure.

The platform's own implementation of that notification writes a warning to the server log. A deployment that wants a real notification replaces the notification operation; every caller goes through it.

An outcome of `partially done` resets the counters, exactly like `fully done`: a job that is making progress is not failing.

## 8. Rescheduling

### 8.1 At the next interval boundary

1. Take the transaction timestamp truncated to the second as now, and the job's current next execution instant as the candidate.
2. While the candidate is at or before now: convert the candidate to the scheduler user's time zone, add the interval (the interval number, in units of the interval unit), and convert the result back to coordinated universal time.
3. Write the candidate as the next execution instant, and now as the last execution instant.

Three consequences:

- The schedule is **anchored**, not drifting: a job due at 02:00 that runs at 02:07 is next due at 02:00 the following day, not at 02:07.
- A job that was due long ago catches up in one step: the loop advances until the instant is in the future, without running the job once per missed interval.
- The addition is performed in the **scheduler user's time zone**, therefore a daily job keeps its local hour across a daylight-saving change even though the interval between two runs is then 23 or 25 hours.

Unit conversions: `minutes` and `hours` are exact; `days` adds calendar days; `weeks` adds seven calendar days for each unit of the interval number; `months` adds calendar months, clamping to the last day of the target month when the source day does not exist there.

**Worked example, daylight saving.** A job with interval 1 day and scheduler time zone `Europe/Brussels` is next due at `2026-03-28 01:00` in coordinated universal time, that is 02:00 local. It runs at 01:05. The loop converts 01:00 to local 02:00, adds one day giving local `2026-03-29 02:00`, and converts back. On 29 March the local offset changes from one hour ahead to two hours ahead, therefore 02:00 local is `00:00` in coordinated universal time. The stored next instant is `2026-03-29 00:00`: 23 hours after the previous one, and still 02:00 for the user.

```formula
elapsed between the two runs in hours = 24 hours − 1 hour of offset change = 23 hours
```

**Worked example, catch-up.** A job with interval 1 hour is next due at 10:00 and the workers were stopped until 15:20. The loop runs five times, producing 11:00, 12:00, 13:00, 14:00 and 15:00, and stops at 16:00, which is in the future. The job body executes **once**.

### 8.2 Immediately

A trigger is created with its instant set to the current instant. Because the due condition of section 3 accepts a trigger whose instant is at or before now, the job is due again immediately; but it will only be re-acquired after the current control transaction commits, and the round's captured key list is not extended, therefore every other due job of the round gets a turn first.

### 8.3 On demand

The trigger operation schedules a job independently of its interval. It takes an optional instant or list of instants and proceeds as follows.

1. If no instant was supplied, the list of instants is the current instant alone. If one instant was supplied, the list is that instant alone. Otherwise the list is as supplied.
2. If the job is inactive, keep only the instants that are strictly in the future.
3. If nothing is left, return without creating a trigger.
4. Create one trigger record per remaining instant, with privileged rights.
5. If the earliest instant is at or before now, or eager notification is enabled, emit a wake-up after the commit.
6. Return the created triggers.

Dropping past instants for an inactive job avoids accumulating triggers that would fire the moment somebody reactivates the job.

The debug note `Job '<name>' (<key>) will execute at <instants>` is emitted when debug logging is on.

### 8.4 Manual execution

A user with write access to Scheduled Actions may run one immediately from its form.

1. Flush and invalidate everything, because the job runs in another transaction.
2. Acquire the job on the **current request transaction**, ignoring the due condition.
3. If it cannot be acquired, refuse with `Job '<name>' already executing`.
4. Process it exactly as the scheduler would, capturing error-level log records.
5. If an error record with a traceback was captured, return a client instruction that displays that error to the user.
6. Otherwise return success.

The job therefore runs with the same two-transaction structure, the same progress protocol and the same rescheduling as an automatic run. The only differences are that the due condition is ignored and that a failure is shown to the user instead of only being logged.

### 8.5 Protection while running

| Operation | Rule | Message on refusal |
|---|---|---|
| Writing a Scheduled Action | Take the referenceable row lock first. | `Record cannot be modified right now: This cron task is currently being executed and may not be modified Please try again in a few minutes` |
| Deleting a Scheduled Action | Take the exclusive row lock first. | The same message. |
| Toggling a job from a settings switch | Take the referenceable row lock; on failure, do nothing and report success. | no message |

The settings switch additionally does nothing at all when the database is marked neutralized, which prevents a neutralized copy from re-enabling jobs that were deliberately disabled.

## 9. Timeouts

A worker is subject to a wall-clock limit per unit of work. When a job body exceeds it, the worker is recycled: the work transaction is aborted and the control transaction never commits, therefore the job keeps its previous next-execution instant and its lock is released by the connection teardown.

What survives is the progress record created at the start of the interrupted iteration, whose consecutive-interruption counter was incremented **before** the body ran and never reset. The next acquisition reads it and applies the short-circuit of section 5.2 once it reaches 3 with no work done.

| Execution | Counter written at iteration start | Counter after a completed iteration | Counter after an interruption |
|---|---|---|---|
| first | 0 + 1 = 1 | 0 | 1 |
| second | 1 + 1 = 2 | 0 | 2 |
| third | 2 + 1 = 3 | 0 | 3 |
| fourth | not started: the counter is 3 and nothing was done | not applicable | outcome `failed`, counter reset to 0 |

Because the counter is reset when the short-circuit fires, the job is given another three chances after each recorded timeout failure, while the failure counter of section 7 accumulates towards deactivation.

## 10. Maintenance of the job tables

Two cleanups run inside the automatic cleanup job.

| Cleanup | Rule |
|---|---|
| Triggers | Delete, in batches of 100 000, every trigger whose instant is more than one week in the past **and** whose job is inactive. Triggers of active jobs are removed by the schedule clearing of section 5.1. |
| Progress records | Delete, in batches of 100 000, every progress record created more than one week ago. |

Each reports the number deleted and whether a further batch remains, which is the signal the cleanup job uses to requeue itself (section 11).

## 11. The automatic cleanup job

One job, shipped by the platform, runs every cleanup that any entity declares. It refuses to run unless the caller is an administrator **and** the call comes from a scheduled job.

1. Collect every declared cleanup of every entity in the registry.
2. Shuffle them randomly and place them in a queue.
3. While the queue is not empty: take one cleanup; run it; report one unit of progress, which commits.
4. If a cleanup returned a pair of a done count and a remaining flag and the remaining flag is set, put it back at the **far end** of the queue, behind every cleanup still pending.
5. If a cleanup raises, log the exception and roll back, then continue with the next cleanup.

The shuffle exists in order that a cleanup that always fails or always exhausts the budget does not permanently starve the ones behind it. Requeuing at the far end gives every other pending cleanup a turn before the batched one resumes, and the batched one keeps resuming until it reports nothing remaining or until the job's time budget ends the execution.

The cleanups declared by the platform and by the capability packages are:

| Cleanup | What it removes or updates |
|---|---|
| Signal table compaction | Rows of the seven signalling tables that are neither in the last ten nor less than one hour old. |
| Session vacuum | Stored sessions untouched for longer than the effective inactivity limit. Suppressible by a deployment switch. |
| Device log compaction | Log rows superseded by a newer row for the same prefix, platform, browser and address. |
| Device revocation marking | Marks device log rows revoked when their session no longer exists, in batches of 100 000 with a commit per batch. |
| User login log compaction | Keeps only the most recent login row per user. |
| Application key expiry | Deletes keys whose expiration has passed. |
| File store collection | Deletes orphaned files from the file store (see [`attachments-and-file-store.md`](attachments-and-file-store.md), section 7). |
| Transient record vacuum | For every transient entity, deletes rows older than the age limit and, when a count limit is set and exceeded, rows older than 300 seconds. Never deletes rows touched in the last 300 seconds. |
| Server action history | Keeps at most the configured number of history entries per action. |
| Trigger and progress cleanup | Section 10. |
| Performance profile expiry | Deletes profiles older than 30 days, in batches of 100 000. |
| Notification bus retention | Deletes bus rows older than the retention parameter, whose default is 86 400 seconds. |
| Cancelled outgoing mail | Deletes cancelled messages older than the configured number of months, whose default is 6. |
| Message translation expiry | Deletes stored message translations past their retention. |
| Overdue activity cleanup | Deletes overdue activities older than the configured number of years, whose default is 0, meaning disabled. |
| Composer attachment cleanup | Deletes attachments left behind by abandoned message composers. |
| Presence compaction | Deletes presence rows not polled for 43 200 seconds (12 hours). |
| Real-time session cleanup | Deletes inactive real-time conversation sessions. |
| Personal mail server cleanup | Deletes personal outgoing mail server records whose owner no longer has one. |
| Sub-channel unpinning | Unpins conversation sub-channels that have gone quiet. |
| Live conversation cleanup | Deletes empty live conversations, unpins ended ones, and deletes conversations that only ever involved the automated agent. |
| Event closing | Marks past events as finished. |
| Card image cleanup | Deletes generated card images older than the configured number of days, whose default is 60. |
| Abandoned coupon cleanup | Deletes coupons of abandoned baskets past their validity. |
| Course membership cleanup | Deletes stale course membership rows. |
| Wish list cleanup | Deletes wish list entries of expired anonymous sessions. |
| Payment transaction cleanup | Deletes pointless pending payment-request transactions. |
| Bank account verification cleanup | Deletes expired verification records. |
| Document index cleanup | Deletes orphaned documentation index rows. |
| Reordering rule cleanup | Deletes reordering rules created automatically and already processed. |

## 12. The catalogue of shipped jobs

Every job below is created by a capability package at installation. Names are given in canonical full-word form. "Interval" is the shipped interval; a deployment may change it. "Active" is the shipped activation state. Jobs whose interval is measured in hundreds or thousands of months are *trigger-only*: the interval exists to keep them technically scheduled, and the real execution comes from an explicit trigger. The machine-readable list, including the operation each job invokes, is in [`../references/scheduled-jobs.md`](../references/scheduled-jobs.md).

### 12.1 Platform

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Automatic cleanup of internal data | 1 day | 3 | yes | Runs every declared cleanup (section 11). |
| Portal user deletion | 1 day | 8 | yes | Deletes portal users who requested account removal, 50 per batch. |

### 12.2 Discussion, messaging and notification

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Outgoing email queue manager | 1 hour | 6 | yes | Sends queued outgoing messages, 1 000 per run by default. See [`mail-gateway.md`](mail-gateway.md), section 3. |
| Incoming mail fetching | 5 minutes | 5 | **no** | Polls every confirmed incoming mail server and routes the fetched messages. See [`mail-gateway.md`](mail-gateway.md), section 7. |
| Post scheduled messages | 1 day | 5 | yes | Posts messages a user scheduled for a later instant. |
| Notify scheduled messages | 1 hour | 5 | yes | Sends the notifications of messages whose notification was deferred. |
| Notification cleanup | 1 month | 5 | yes | Deletes notification rows older than 180 days. |
| Send push notifications | 1 day | 5 | yes | Delivers queued browser push notifications, 50 per run; re-triggers itself while any remain. |
| Conversation member unmute | 1 day | 5 | yes | Clears expired mute settings on conversation memberships. |
| Subscription status check | 1 week | 1000 | yes | Contacts the publisher's service to refresh the deployment's subscription status. |
| Discussion group moderator notification | 1 day | 1000 | yes | Notifies moderators of pending messages in moderated discussion groups. |
| Text message queue manager | 24 hours | 5 | yes | Sends queued outgoing text messages, 500 per run by default. |
| Postal letter queue processing | 24 hours | 5 | yes | Sends queued postal letters and retries recoverable failures. |
| Digest email sending | 1 day, first run 2 hours after installation | 5 | yes | Sends every digest whose next mailing date has arrived. See [`background-workers.md`](background-workers.md), section 7. |

### 12.3 Automation and integration

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Automation rules: check and execute | 4 hours | 5 | **no** | Evaluates every time-based automation rule. See [`background-workers.md`](background-workers.md), section 9. |
| Data recycling: clean records | 1 day, first run at 03:00 | 5 | yes | Applies the configured recycling rules to stale records. See [`background-workers.md`](background-workers.md), section 8. |
| Translation reload from the external service | 7 days | 5 | yes | Re-reads the message catalogues that an external translation service maintains. See [`translation.md`](translation.md), section 11. |
| Attachment migration to remote storage | 9 999 months (trigger-only) | 5 | yes | Moves attachment content from the local file store to a remote object store. |

### 12.4 Accounting and invoicing

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Post draft entries with automatic posting due | 1 day, first run at 02:00 | 5 | yes | Posts every draft journal entry marked for automatic posting whose accounting date has arrived. |
| Send invoices automatically | 1 day | 5 | yes | Performs the deferred sending of invoices queued by the send-and-print flow. |
| Inventory valuation closing | 1 day | 5 | yes | Produces the periodic inventory valuation entries. |
| Daily sales closing | 1 day | 5 | yes | Produces the daily sealed sales total for jurisdictions that require it. |
| Monthly sales closing | 1 month | 5 | yes | The monthly equivalent. |
| Annual sales closing | 12 months | 5 | yes | The annual equivalent. |

### 12.5 Electronic invoicing and document exchange

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Electronic document exchange: perform web service operations | 1 day | 5 | **no** | Processes up to 20 pending exchange documents per run. |
| Document exchange network: retrieve new documents | 4 hours | 5 | yes | Downloads incoming documents from the exchange network. |
| Document exchange network: update message status | 1 day | 5 | yes | Refreshes the delivery status of sent documents. |
| Document exchange network: update participant status | 1 week | 5 | yes | Refreshes the registration status of the participant. |
| Document exchange network: keep the callback registration alive | 2 weeks | 5 | yes | Renews the callback subscription. |
| Document exchange network: automatic service registration | 999 months (trigger-only) | 5 | yes | Registers the response services once the participant is ready. |
| Danish exchange network: retrieve new documents | 4 hours | 5 | yes | Downloads incoming documents from the Danish network. |
| Danish exchange network: update message status | 1 day | 5 | yes | Refreshes the delivery status of documents sent on the Danish network. |
| Danish exchange network: update participant status | 1 week | 5 | yes | Refreshes the registration status of the Danish participant. |
| Danish exchange network: keep the callback registration alive | 2 weeks | 5 | yes | Renews the callback subscription on the Danish network. |
| French exchange platform: retrieve regulatory documents | 4 hours | 5 | yes | Downloads regulatory documents. |
| French exchange platform: send lifecycle statuses | 12 hours | 5 | yes | Sends the lifecycle status of received documents. |
| French exchange platform: generate periodic flows | 1 day | 5 | yes | Builds and sends the periodic regulatory flow. |
| Italian exchange system: receive invoices | 1 day | 5 | yes | Downloads invoices from the national exchange system. |
| Hungarian tax authority: update status of pending invoices | 1 day, first run at 22:00 | 5 | yes | Polls the tax authority for the status of submitted invoices. |
| Greek tax authority: fetch third-party invoices | 1 day, first run at 22:00 | 5 | yes | Downloads third-party issued invoices and creates draft vendor bills. |
| Croatian exchange service: retrieve new documents | 4 hours | 5 | yes | Downloads incoming documents. |
| Croatian exchange service: update document statuses | 4 hours | 5 | yes | Refreshes statuses. |
| Croatian exchange service: archive signed documents | 4 hours | 5 | yes | Archives the signed source documents. |
| Polish exchange system: check invoice status | 1 week, first run at 22:00 | 5 | yes | Polls the national system for invoice status. |
| Polish exchange system: download vendor bills | 3 hours | 5 | yes | Downloads incoming vendor bills. |
| Polish exchange system: refresh access tokens | 6 days | 5 | yes | Renews the access tokens of the national system. |
| Romanian tax authority: refresh access token | 30 days, first run at 22:00 | 5 | yes | Renews the access token. |
| Romanian tax authority: synchronize invoices | 1 day, first run at 22:00 | 5 | yes | Sends and receives invoices. |
| Spanish sealed-record service: submit or cancel records | 1 day | 5 | yes | Submits the next batch of sealed records. |
| Turkish exchange service: retrieve new purchase documents | 12 hours | 5 | yes | Downloads purchase documents. |
| Turkish exchange service: retrieve new sales documents | 12 hours | 5 | yes | Downloads electronic sales documents. |
| Turkish exchange service: retrieve new archived sales documents | 12 hours | 5 | yes | Downloads archived sales documents. |
| Turkish exchange service: retrieve invoice status | 12 hours | 5 | yes | Refreshes invoice statuses. |
| Turkish exchange service: retrieve sales printable documents | 12 hours | 5 | yes | Downloads the printable representation of sales documents. |
| Malaysian exchange service: document synchronization | 1 hour | 5 | yes | Refreshes the status of submitted documents. |
| Indonesian payment-request service: fetch status | 1 hour | 5 | yes | Refreshes the status of issued payment requests. |
| Tax number validation service synchronization | 1 day | 5 | yes | Refreshes tax number validation results from the validation service. |

### 12.6 Sales, purchase and commerce

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Send ready invoices after payment | 1 day | 5 | **no** | Sends the invoices generated by automatic invoicing. |
| Send pending sales emails | 1 day | 5 | **no** | Sends the confirmation emails deferred by the asynchronous sending setting. |
| Purchase reminder | 1 day | 5 | yes | Asks vendors to confirm the delivery date of open purchase orders. |
| Payment post-processing | 10 minutes | 5 | **no** | Finalizes payment transactions whose provider callback has been received. |
| Abandoned basket email | 1 hour | 5 | yes | Emails customers who left a basket behind. |
| Product availability email | 1 hour | 5 | yes | Emails customers who asked to be told when a product is back in stock. |
| Storefront visitor cleanup | 1 day | 5 | yes | Deletes inactive storefront visitor records. |
| Disable unused storefront asset blocks | 1 week | 5 | yes | Turns off the asset bundles of building blocks no longer used by any page. |
| Marketing campaign queue | 1 day | 6 | yes | Sends the next batch of a running mass mailing. |
| Marketing campaign comparison test | 1 day | 5 | **no** | Picks the winning variant of a comparison test and sends it to the rest of the audience. |
| Assign form fields to quotation documents | 9 999 months (trigger-only) | 5 | yes | Fills the form field descriptions of quotation documents. |

### 12.7 Customer relationship management

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Recompute automated lead probabilities | 1 day | 5 | **no** | Retrains and applies the predictive lead scoring model. |
| Lead assignment | 1 day | 5 | **no** | Distributes unassigned leads to teams and salespeople according to the assignment rules. |
| Lead enrichment | 24 hours | 5 | yes | Enriches leads from the enrichment service. |
| Event-driven lead generation | 1 day | 5 | yes | Creates leads from event registrations according to the configured rules. |
| Visitor-driven lead generation | 1 day | 5 | yes | Creates leads from identified site visitors. |

### 12.8 Inventory, manufacturing and field operations

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Replenishment scheduler | 1 day | 5 | yes | Runs the procurement scheduler: evaluates reordering rules and creates the resulting replenishment documents. |
| Vehicle contract cost generation | 1 day | 5 | yes | Creates the recurring costs of vehicle contracts according to their frequency. |

### 12.9 People, time and engagement

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Notify expiring work permits and contracts | 1 day | 5 | yes | Warns managers about employee documents about to expire. |
| Update the current employee version | 1 day | 5 | yes | Advances each employee record to the version effective today. |
| Automatic check-out | 4 hours | 5 | yes | Closes attendance records left open beyond the configured limit. |
| Absence detection | 4 hours | 5 | yes | Creates absence records for employees expected but not present. |
| Presence check | 1 hour | 5 | yes | Re-evaluates the presence indicator of every employee. |
| Submitted expense reminder | 1 week | 5 | yes | Reminds approvers of expense reports awaiting them. |
| Accrual time-off update | 1 day | 5 | yes | Advances every accrual allocation to today and grants the accrued amounts. |
| Cancel invalid time off | 1 day | 5 | yes | Cancels time-off requests that became invalid, for example after a contract change. |
| Certification activity creation | 1 day | 5 | yes | Creates follow-up activities for employees with missing or expiring certifications. |
| Generate missing work entries | 1 day | 5 | yes | Creates the work entries of every employee version that lacks them. |
| Goal challenge check | 1 day | 5 | yes | Re-evaluates every running goal challenge and closes the ones that ended. |
| Reward point tracking consolidation | 1 month, first run on the first day of next month at 04:00 | 5 | yes | Consolidates reward point history into period totals. |
| Unregistered user reminder | 1 day | 6 | yes | Reminds invited users who never completed their registration, 100 per run. |

### 12.10 Calendar, events and projects

| Job | Interval | Priority | Active | Purpose |
|---|---|---|---|---|
| Calendar reminder | 1 day | 5 | yes | Sends the reminders of upcoming calendar events. |
| Calendar synchronization with the external calendar service | 12 hours | 5 | yes | Pulls and pushes calendar changes. One job per supported external calendar service. |
| Event mail scheduler | 24 hours, first run 15 minutes after installation | 5 | yes | Sends the scheduled communications of every event, committing after each. |
| Project stage rating | no interval set; the unit is days | 5 | yes | Sends the satisfaction rating requests configured on project stages. |

## 13. Worked examples

### 13.1 A batched job that finishes in one acquisition

A queue job has 25 items to process, each batch handling 10.

| Iteration | Body behaviour | Done | Remaining | Decision |
|---|---|---|---|---|
| 1 | processes 10, reports 10 processed and 15 remaining | 10 | 15 | keep looping |
| 2 | processes 10, reports 10 processed and 5 remaining | 10 | 5 | keep looping |
| 3 | processes 5, reports 5 processed and 0 remaining | 5 | 0 | `fully done` |

Three progress records exist, one per iteration. The job is rescheduled at its next interval boundary. The failure counters are reset.

### 13.2 A batched job that runs out of budget

The same job has 500 items.

| Iteration | Elapsed | Decision |
|---|---|---|
| 1 to 10 | 6 seconds | keep looping; each reports a positive remaining count |
| 11 to 14 | 10.2 seconds | keep looping while the elapsed time is below 10 seconds; the loop stops after the iteration that crosses it |
| end of loop | 10.2 seconds | the status is still unset, therefore the outcome is `partially done` |

The job is rescheduled **immediately** by a trigger at the current instant. Its failure counters are reset. Another worker, or the same one on its next round, acquires it again after every other due job of the round has had a turn.

### 13.3 A job that fails in the middle of a batch

| Iteration | Body behaviour | Done | Remaining | Decision |
|---|---|---|---|---|
| 1 | processes 10, reports progress, which commits, then raises | 10 | 15 | keep looping: the failure is assumed transient |
| 2 | raises before reporting anything | 0 | 15 | `failed` |

Outcome `failed`. The failure count becomes 1 and the first-failure instant is set. The job is rescheduled at its next interval boundary. The 10 items processed in iteration 1 stay processed, because the progress report committed them.

### 13.4 A job that deactivates itself

The incoming mail fetching job finds that no confirmed server is left.

| Step | Effect |
|---|---|
| 1 | The body reports progress with the deactivation flag set. |
| 2 | The body returns; the remaining count is 0, therefore the outcome is `fully done`. |
| 3 | Because the progress record carries the deactivation flag, the job's active flag is set to false in the same write as the failure counters. |
| 4 | The job is still rescheduled at its next interval boundary, but the due condition now fails because it is inactive. |

### 13.5 Two workers racing for the same job

| Step | Worker A | Worker B |
|---|---|---|
| 1 | Captures the due keys 7 and 9. | Captures the due keys 7 and 9. |
| 2 | Acquires job 7 and holds its row lock. | Attempts job 7: no row is returned because the lock is held and locked rows are skipped. Debug note, moves on. |
| 3 | Runs job 7, commits, releasing the lock. | Acquires job 9, runs it, commits. |
| 4 | Attempts job 9: no row, because job 9 is no longer due. | no activity |

No job ran twice and no worker waited.

## 14. Acceptance criteria

1. **Given** a job with interval 1 day and next execution at 02:00, **when** it runs at 02:07 and reports `fully done`, **then** its next execution is 02:00 the following day and its last execution is 02:07 truncated to the second.
2. **Given** a job whose next execution is five hours in the past with interval 1 hour, **when** a worker processes it, **then** the body runs exactly once and the next execution is the first hourly boundary in the future.
3. **Given** a job whose scheduler user's time zone observes a spring daylight-saving change, **when** the next execution is computed across that change, **then** the local hour is preserved and the interval between the two instants in coordinated universal time is 23 hours.
4. **Given** a job whose body never reports progress, **when** it returns without raising, **then** the outcome is `fully done` and the inner loop performs exactly one iteration.
5. **Given** a job whose body reports a positive remaining count on every iteration, **when** ten iterations have completed in less than 10 seconds, **then** the loop continues until the 10 seconds have elapsed and the outcome is `partially done`.
6. **Given** a `partially done` outcome, **when** the control transaction commits, **then** a trigger exists for the job with the current instant and the failure counters are 0.
7. **Given** a body that raises on its first iteration without reporting progress, **when** the job completes, **then** the outcome is `failed` and the failure count is incremented.
8. **Given** a body that reports progress and then raises, **when** the job completes, **then** the reported work is durable and the loop continued to the next iteration.
9. **Given** a job with failure count 4 and a first failure 8 days ago, **when** it fails again, **then** it is deactivated, the failure count and first-failure instant are reset, and an administrator notification is emitted.
10. **Given** a job with failure count 4 and a first failure 2 hours ago, **when** it fails again, **then** the failure count becomes 5 and the job stays active.
11. **Given** a job with failure count 3, **when** it reports `partially done`, **then** the failure count returns to 0.
12. **Given** a job whose last progress record has a consecutive-interruption counter of 3 and a done count of 0, **when** it is acquired, **then** its body is not executed, the outcome is `failed`, the counter is reset to 0, and the error `Job '<name>' (<key>) timed out` is logged.
13. **Given** two workers examining the same due job, **when** the first holds the lock, **then** the second acquires nothing for that job and moves to the next one without waiting.
14. **Given** a trigger created with an instant in the past inside a transaction, **when** that transaction commits, **then** a wake-up carrying the database name is emitted and the workers examine that database at once.
15. **Given** an inactive job, **when** a trigger is requested for an instant in the past, **then** no trigger record is created.
16. **Given** an inactive job, **when** a trigger is requested for an instant in the future, **then** the trigger record is created.
17. **Given** a running job, **when** a user saves a change to it, **then** the save is refused with the message of section 8.5.
18. **Given** a job already acquired by a worker, **when** a user asks to run it manually, **then** the request is refused with `Job '<name>' already executing`.
19. **Given** a manual run whose body raises, **when** the operation returns, **then** the user is shown the error rather than only having it logged.
20. **Given** a database whose base package version differs from the running code, **when** a worker polls it, **then** no job of that database is acquired and the version warning is logged.
21. **Given** a database with a package queued for installation and a due job whose reference instant is 2 hours old, **when** a worker polls it, **then** no job is acquired and the package-state warning is logged.
22. **Given** the same database and a due job whose reference instant is 6 hours old, **when** a worker polls it, **then** the transient package states are reset and processing continues.
23. **Given** a cleanup that returns a remaining flag, **when** the automatic cleanup job runs, **then** that cleanup is put back at the far end of the queue and runs again only after every other pending cleanup has had a turn.
24. **Given** a cleanup that raises, **when** the automatic cleanup job runs, **then** the exception is logged, the transaction is rolled back, and the remaining cleanups still run.
25. **Given** the automatic cleanup job invoked outside a scheduled job, or by a user who is not an administrator, **when** the call is made, **then** it is refused with an access denial.
26. **Given** an interval number of zero, **when** the Scheduled Action is saved, **then** the save is refused with `The interval number must be a strictly positive number.`.
27. **Given** a job body that calls the progress operation outside any scheduled job, **when** it returns, **then** the transaction is committed and an unbounded time budget is reported.

## 15. Reconciliation notes

1. The worker loop, the due selection, the acquisition read, the inner loop, the progress operation, the failure counting, the rescheduling arithmetic, the trigger operation, the manual run and the cleanup queue were written as code-shaped sketches. They are restated as numbered procedures. Every constant is unchanged: two workers by default, a 60-second poll, 5 hours of staleness before transient package states are reset, at least 10 iterations then at least 10 seconds, three consecutive interruptions, five failures and seven days before deactivation, and batches of 100 000 rows in the cleanups.
2. The acquisition read was written with its lock clause spelled out. It is described by behaviour instead: the exclusive but referenceable strength, skipping rows another transaction holds. The strengths themselves are specified in [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 7.
3. The jobs that drain the outgoing mail queue, poll the incoming mail servers, send digests, recycle data and evaluate time-based automation rules are listed here with their schedule; what they do is specified in [`mail-gateway.md`](mail-gateway.md) and [`background-workers.md`](background-workers.md), which were one document and are now two.
