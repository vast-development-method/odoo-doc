# Scheduled Actions (`ir.cron`)

**Transport name:** `ir.cron`  
**Storage name:** `ir_cron`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `base_automation`

Description: Scheduled Actions

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Delegation inheritance: embeds `ir.actions.server` through field `ir_actions_server_id`
- Default ordering: `cron_name, id`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `ir_actions_server_id` | Server action | many to one | `ir.actions.server` | required; indexed; on delete of the target: restrict |
| `cron_name` | Name | single line text |  | computed by rule `_compute_cron_name` and stored |
| `user_id` | Scheduler User | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; extended by packages `mail` |
| `active` | Active | boolean |  | default `True` |
| `interval_number` | Interval Number | integer |  | required; default `1`; changes are tracked in the message thread; aggregated with avg; Help: Repeat every x.; extended by packages `mail` |
| `interval_type` | Interval Unit | selection |  | required; default `months`; changes are tracked in the message thread; extended by packages `mail` |
| `nextcall` | Next Execution Date | date and time |  | required; default computed dynamically (fields.Datetime.now); Help: Next planned execution date for this job. |
| `lastcall` | Last Execution Date | date and time |  | Help: Previous time the cron ran successfully, provided to the job through the context on the `lastcall` key |
| `priority` | Priority | integer |  | default `5`; changes are tracked in the message thread; Help: The priority of the job, as an integer: 0 means higher priority, 10 means lower priority.; extended by packages `mail` |
| `failure_count` | Failure Count | integer |  | default ; Help: The number of consecutive failures of this job. It is automatically reset on success. |
| `first_failure_date` | First Failure Date | date and time |  | Help: The first time the cron failed. It is automatically reset on success. |

## Selection values

### `interval_type` (Interval Unit)

| Value | Label |
|---|---|
| `minutes` | Minutes |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_strictly_positive_interval` | Constraint | `CHECK(interval_number > 0)` | The interval number must be a strictly positive number. | `base` |

## Operations (31)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_cron_name` | computation | self | `base` | depends: `ir_actions_server_id.name` |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `default_get` | lifecycle override | self, fields | `base` | model |  |
| `method_direct_trigger` | operation | self | `base` |  | Run the CRON job in the current (HTTP) thread.  The job is still ran as it would be by the scheduler: a new cursor is used for the execution of the job.  :raises UserError: when the job is already running |
| `_process_jobs` | background operation | db_name | `base` |  | Execute every job ready to be run on this database. |
| `_process_jobs_loop` | background operation | cron_cr, job_ids | `base` |  | Process ready jobs to run on this database.  The `cron_cr` is used to lock the currently processed job and relased by committing after each job. |
| `_check_version` | validation | cron_cr | `base` |  | Ensure the code version matches the database version |
| `_check_modules_state` | validation | cr, jobs | `base` |  | Ensure no module is installing or upgrading |
| `_get_ready_sql_condition` | preparation rule | cr | `base` |  |  |
| `_get_all_ready_jobs` | preparation rule | cr | `base` |  | Return a list of all jobs that are ready to be executed |
| `_acquire_one_job` | internal rule | cr, job_id, include_not_ready | `base` |  | Acquire for update the job with id ``job_id``.  The job should not have been processed yet by the current worker. Another worker may process the job again, may that job become ready again quickly enough (e.g. self-triggering, high frequency, or partially done jobs).  Note: It is possible that this function raises a       ``psycopg2.errors.SerializationFailure`` in case the job       has been processed in another worker. In such case it is       advised to roll back the transaction and to go on with the       other jobs. |
| `_notify_admin` | internal rule | self, message | `base`, `mail` |  | Notify ``message`` to some administrator.  The base implementation of this method does nothing. It is supposed to be overridden with some actual communication mechanism. |
| `_process_job` | background operation | cls, cron_cr, job | `base` |  | Execute the cron's server action in a dedicated transaction.  In case the previous process actually timed out, the cron's server action is not executed and the cron is considered ``'failed'``.  The server action can use the progress API via the method :meth:`_commit_progress` to report how many records are done in each batch. Those progress notifications are used to determine the job's ``CompletionStatus`` and to determine the next time the cron will be executed:  - ``'fully done'``: the cron is rescheduled later, it'll be   executed again after its regular time interval or upon a new   trigge |
| `_run_job` | background operation | cls, job | `base` |  | Execute the job's server action multiple times until it completes. The completion status is returned.  It is considered completed when either:  - the server action doesn't use the progress API, or returned   and notified that all records has been processed: ``'fully done'``;  - the server action returned and notified that there are   remaining records to process, but this cron worker ran this   server action 10 times already: ``'partially done'``;  - the server action was able to commit and notify some work done,   but later crashed due to an exception: ``'partially done'``;  - the server acti |
| `_update_failure_count` | internal rule | self, job, status | `base` | model | Update cron ``failure_count`` and ``first_failure_date`` given the job's completion status. Deactivate the cron when BOTH the counter reaches ``MIN_FAILURE_COUNT_BEFORE_DEACTIVATION`` AND the time delta reaches ``MIN_DELTA_BEFORE_DEACTIVATION``.  On ``'fully done'`` and ``'partially done'``, the counter and failure date are reset.  On ``'failed'`` the counter is increased and the first failure date is set if the counter was 0. In case both thresholds are reached, ``active`` is set to ``False`` and both values are reset. |
| `_clear_schedule` | internal rule | self, job | `base` | model | Remove triggers for the given job. |
| `_reschedule_later` | internal rule | self, job | `base` | model | Reschedule the job to be executed later, after its regular interval or upon a trigger. |
| `_reschedule_asap` | internal rule | self, job | `base` | model | Reschedule the job to be executed ASAP, after the other cron jobs had a chance to run. |
| `_callback` | internal rule | self, cron_name, server_action_id | `base` |  | Run the method associated to a given job. It takes care of logging and exception handling. Note that the user running the server action is the user calling this method. |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_unlink_unless_running` | internal rule | self | `base` | ondelete |  |
| `toggle` | operation | self, model, domain | `base` | model |  |
| `_trigger` | internal rule | self, at | `base` |  | Schedule a cron job to be executed soon independently of its ``nextcall`` field value.  By default, the cron is scheduled to be executed the next time the cron worker wakes up, but the optional `at` argument may be given to delay the execution later, with a precision down to 1 minute.  The method may be called with a datetime or an iterable of datetime. The actual implementation is in :meth:`~._trigger_list`, which is the recommended method for overrides.  :param at:     When to execute the cron, at one or several moments in time     instead of as soon as possible. :return: the created trigger |
| `_trigger_list` | internal rule | self, at_list | `base` |  | Implementation of :meth:`~._trigger`.  :param at_list: Execute the cron later, at precise moments in time. :return: the created triggers records |
| `_notifydb` | internal rule | self | `base` | model | Wake up the cron workers The ODOO_NOTIFY_CRON_CHANGES environment variable allows to force the notifydb on both IrCron modification and on trigger creation (regardless of call_at) |
| `_add_progress` | internal rule | self, timed_out_counter | `base` |  | Create a progress record for the given cron and add it to its context.  :param int timed_out_counter: the number of times the cron has     consecutively timed out :return: a pair ``(cron, progress)``, where the progress has     been injected inside the cron's context |
| `_notify_progress` | internal rule | self, done, remaining, deactivate | `base` |  | Log the progress of the cron job. Use ``_commit_progress()`` instead.  :param int done: the number of tasks already processed :param int remaining: the number of tasks left to process :param bool deactivate: whether the cron will be deactivated |
| `_commit_progress` | internal rule | self, processed, remaining, deactivate | `base` | model | Commit and log progress for the batch from a cron function.  The number of items processed is added to the current done count. If you don't specify a remaining count, the number of items processed is subtracted from the existing remaining count.  If called from outside the cron job, the progress function call will just commit.  :param processed: number of processed items in this step :param remaining: set the remaining count to the given count :param deactivate: deactivate the cron after running it :return: remaining time (seconds) for the cron run |
| `action_open_parent_action` | user action | self | `base` |  |  |
| `action_open_scheduled_action` | user action | self | `base` |  |  |
| `action_open_automation` | user action | self | `base_automation` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `method_direct_trigger` | UserError | Job '%s' already executing | `base` |
| `write` | UserError | Record cannot be modified right now: This cron task is currently being executed and may not be modified Please try again in a few minutes | `base` |
| `_unlink_unless_running` | UserError | Record cannot be modified right now: This cron task is currently being executed and may not be modified Please try again in a few minutes | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_cron_view_form` | xpath | `base.view_server_action_form` |  | `Run Manually` |  | `base` |
| `base.ir_cron_view_tree` | list |  | `priority`, `name`, `model_id`, `nextcall`, `interval_number`, `interval_type`, `active` |  |  | `base` |
| `base.ir_cron_view_calendar` | calendar |  | `name` |  |  | `base` |
| `base.ir_cron_view_search` | search |  | `name`, `user_id`, `model_id`, `nextcall`, `active` |  | `All`, `Archived`, `User`, `Execution`, `Model` | `base` |
| `mail.ir_cron_view_form` | xpath | `base.ir_cron_view_form` |  |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_cron_act` | Scheduled Actions | list,form,calendar |  | `{'search_default_all': 1}` |  | `base` |
| `cloud_storage_migration.action_cloud_storage_migration_cron` | Cloud Storage Migration Cron | form |  |  | current | `cloud_storage_migration` |

Machine-readable definition: `../../../schemas/data/entities/ir.cron.json`; views: `../../../schemas/interfaces/views/ir.cron.json`.
