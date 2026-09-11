# Workflows

Each procedure below is numbered, states its role, lists its steps in order, names the records each
step creates or changes, names the operation invoked, and states the conditions under which the step
fails. Failure texts are reproduced exactly; the rule that owns each text is cited from
[`business-rules.md`](business-rules.md).

Contents:

1. [Configure an automation rule](#1-configure-an-automation-rule)
2. [Run an automation rule on creation and on update](#2-run-an-automation-rule-on-creation-and-on-update)
3. [Run an automation rule while a form is open](#3-run-an-automation-rule-while-a-form-is-open)
4. [Run an automation rule on deletion](#4-run-an-automation-rule-on-deletion)
5. [Run an automation rule when a message is posted](#5-run-an-automation-rule-when-a-message-is-posted)
6. [Run the time-based automation scheduler](#6-run-the-time-based-automation-scheduler)
7. [Receive an incoming webhook call](#7-receive-an-incoming-webhook-call)
8. [Send an outgoing webhook notification](#8-send-an-outgoing-webhook-notification)
9. [Import a data file](#9-import-a-data-file)
10. [Resume a batched import](#10-resume-a-batched-import)
11. [Import a packaged module archive](#11-import-a-packaged-module-archive)
12. [Install a package from the remote directory](#12-install-a-package-from-the-remote-directory)
13. [Request the activation of a package](#13-request-the-activation-of-a-package)
14. [Obtain and use a metered service account](#14-obtain-and-use-a-metered-service-account)
15. [Watch and top up a balance](#15-watch-and-top-up-a-balance)
16. [Enrich a company from the outside directory](#16-enrich-a-company-from-the-outside-directory)
17. [Resolve an address to coordinates](#17-resolve-an-address-to-coordinates)
18. [Complete an address while typing](#18-complete-an-address-while-typing)
19. [Recycle aged records](#19-recycle-aged-records)
20. [Handle a privacy request](#20-handle-a-privacy-request)
21. [Store an attachment in the cloud](#21-store-an-attachment-in-the-cloud)
22. [Link a mail server to an outside provider](#22-link-a-mail-server-to-an-outside-provider)
23. [Complete an onboarding panel](#23-complete-an-onboarding-panel)
24. [Run, record and export a guided tour](#24-run-record-and-export-a-guided-tour)
25. [Refresh and contribute translations](#25-refresh-and-contribute-translations)
26. [Drive a connected peripheral](#26-drive-a-connected-peripheral)

---

# 1. Configure an automation rule

**Role**: settings administrator. Only that group may read, create, write or delete a rule.

## 1.1 Steps

1. Open the automation list. It opens in card form, with the *Include Archived* filter already
   switched on, so archived rules are visible.
2. Create a rule and type its name. **Refusal**: an empty name is refused by the platform's
   required-field check — rule [`AUT-001`](business-rules.md#aut-001).
3. Choose the watched record type. Only non-abstract record types may be chosen. Choosing one
   **clears the trigger**, because the trigger computation depends on the record type; it also
   detaches every action whose own record type no longer matches.
4. Choose the trigger. The choice sets, in one pass:
   - the trigger date field, for the two relative-time triggers;
   - the delay, mode and unit defaults, for the three time triggers;
   - the working schedule, cleared unless the unit is days;
   - the selection choice and the reference, both cleared;
   - the before-filter, set only for the tag trigger;
   - the after-filter, set for the six triggers listed in [`entities.md`](entities.md) §1.6.8;
   - the watched-field set, set to the trigger-specific field.
   **Refusal**: a message trigger on a record type without a discussion thread is refused —
   rule [`AUT-002`](business-rules.md#aut-002).
5. For a stage, tag, state or priority trigger, choose the value the record must reach. The chosen
   value is written into the after-filter, and for the tag trigger also into the before-filter.
6. For a time trigger, type the delay and choose the unit and, for the date-field trigger, the mode.
   **Refusal**: a negative delay is refused on save — rule [`AUT-003`](business-rules.md#aut-003).
   While the form is open, typing a negative delay is silently corrected: the sign is dropped and,
   for the date-field trigger only, the mode is flipped.
7. Optionally narrow the after-filter, and, for update triggers, the before-filter. Editing either
   filter keeps the watched-field set — or, for the live-update trigger, the live-update field set —
   in step with the field names that entered and left the filter.
8. Optionally restrict the watched fields by hand.
9. Add the actions on the *Actions To Do* tab. Each new action inherits the rule's record type and
   the usage value `base_automation`. **Refusals**: an action targeting a different record type is
   refused — rule [`AUT-004`](business-rules.md#aut-004); an action carrying its own warning is
   refused — rule [`AUT-005`](business-rules.md#aut-005); a non-code action under the live-update
   trigger is refused — rule [`AUT-006`](business-rules.md#aut-006); a message, follower or activity
   action under the deletion trigger is refused — rule [`AUT-007`](business-rules.md#aut-007). The
   last two are also warned about while the form is open, with the texts of §1.2 below.
10. Save. The rule is written, the scheduler interval is recomputed, the registry patches are
    reinstalled, and — when the rule is a live-update rule with at least one live-update field — the
    client template cache is emptied so that the affected forms carry the new live-update fields.

## 1.2 The two interface warnings

Both appear while the form is open and neither blocks saving on its own; the equivalent constraints
in step 9 do.

| Situation | Title | Message |
|---|---|---|
| The trigger is the live-update trigger and at least one action is not a code action | "Warning" | "The "%(trigger_value)s" %(trigger_label)s can only be used with the "%(state_value)s" action type" — the first placeholder is the trigger's own label "On UI change", the second is the trigger field's label "Trigger", the third is the action-kind label "Execute Code" |
| The trigger is the deletion trigger and at least one action posts a message, changes followers or creates an activity | "Warning" | "You cannot send an email, add followers or create an activity for a deleted record.  It simply does not work." — reproduced including its double space |

## 1.3 Duplicating a rule

Duplicating first duplicates the action list, then duplicates the rule, then attaches the duplicated
actions to the copy. The copy therefore owns its own actions. The webhook identifier is not copied;
the copy receives a fresh one.

## 1.4 Replacing the webhook identifier

Pressing *Renew* writes a fresh universally unique identifier onto every selected rule. The web
address changes with it; every outside caller must be updated. The button carries the help text "If
renewed, update the secret URL in the third-party app that calls this webhook."

---

# 2. Run an automation rule on creation and on update

**Role**: none — the procedure runs inside whatever operation the user or another program performed.

## 2.1 How the interception is installed

At start-up, and again after any change to a rule's record type, activation, trigger or live-update
fields, every rule in the database is read — including archived ones, because the reading context is
cleared — and the watched record type is patched:

| The rule's trigger family | What is patched on the record type |
|---|---|
| A creation trigger | the creation operation |
| An update trigger | the write operation **and** the stored-computed-field operation |
| The deletion trigger | the delete operation |
| The live-update trigger | one live-update handler is registered for each of the rule's live-update fields |
| A message trigger, on a record type with a discussion thread | the message-posting operation |

A record type is patched at most once per operation, however many rules watch it. A rule whose
record type no longer exists is skipped and a warning naming the rule, its identifier, its record
type and the record type's identifier is logged.

## 2.2 Creation

1. The patched creation operation asks for every active rule whose record type matches and whose
   trigger is one of the seven creation triggers. When there is none, the original creation runs
   unchanged and the procedure ends.
2. The original creation runs, in the rules' own reading environment.
3. For each rule, with the before-values explicitly marked as absent:
   1. The after-filter is evaluated over the created records, with elevated rights, and with the
      feedback flag set so that automations triggered by the evaluation itself are recorded.
   2. The surviving records are processed (§2.4).
   3. Computed fields that depend on the fields being written are preserved across the evaluation,
      so that they are recomputed afterwards rather than dropped.
4. The created records are returned in the caller's own environment.

Because the before-values are absent, the watched-field check of §2.4 step 3 always passes on
creation: on a creation, every field counts as modified.

## 2.3 Update

1. The patched write operation asks for every active rule whose record type matches and whose
   trigger is one of the nine update triggers. When there is none, or the set of records is empty,
   the original write runs unchanged.
2. Records without an identifier — records being built in a form — are dropped.
3. For each rule, the before-filter is evaluated over the records, with elevated rights. The result
   is remembered per rule.
4. The before-values of every stored field being written are read and remembered per record.
5. The original write runs.
6. For each rule, with the remembered before-values available:
   1. The after-filter is evaluated over that rule's remembered pre-set, and both the survivors and
      the evaluated filter are kept.
   2. The survivors are processed (§2.4), and the evaluated filter is passed along so that an action
      can consult it.

The same procedure is installed on the stored-computed-field operation, so that a change made by a
recomputation triggers rules exactly as a direct write would. There the changed fields are every
field produced by the same computation, and the before-values are read for every stored field among
them.

## 2.4 Processing a rule against a set of records

1. Records this rule has already been applied to **in this call chain** are removed. The set of
   already-processed records is carried in the reading context under the key `__action_done`, keyed
   by rule. This is the recursion guard: a rule that writes a field it watches does not run twice.
2. The remaining records are marked as processed. When the feedback flag is set the marking modifies
   the carried mapping in place, so that fields computed during filtering know which rules have
   already run; otherwise a fresh copy of the mapping is made and carried onwards.
3. The records are filtered by the watched-field check: a record survives when the rule has no
   watched fields at all, when the before-values are absent — a creation — or when at least one
   watched field's before-value differs from its current value. The processed marking is then
   narrowed to the survivors.
4. When the record type carries a field named `date_automation_last`, that field is set to the
   transaction's current instant on every surviving record.
5. For each action of the rule, in the action list's own order, and for each surviving record in
   turn, the action is run with the record as the active record, with the whole set as the active
   set, and with the evaluated after-filter available under the name `domain_post`.
6. Any failure raised by an action is annotated, for an internal user only, with the rule's
   identifier and name so that the error screen can name the rule, and is then re-raised. The
   annotation is not added for an external user.

## 2.5 Failure conditions

| Condition | Effect |
|---|---|
| An action fails | The whole enclosing operation fails; nothing is saved. For an internal reader the failure carries the rule's identifier and name. |
| The after-filter is not valid | The evaluation fails and the enclosing operation fails with it. |
| The watched record type was deleted | The rule is skipped at patch time with a logged warning; no record is ever processed. |

---

# 3. Run an automation rule while a form is open

**Role**: any user editing a form of the watched record type.

1. The user edits one of the rule's live-update fields. The client sends the form's current values
   to the server.
2. The registered live-update handler for that rule runs.
3. The after-filter is evaluated over the form's values. When the form does not satisfy it, the
   handler returns nothing at all and the form is unchanged.
4. Otherwise every action of the rule is run with the form's record type as the active record type,
   the underlying stored record as the active record, and the in-progress form values available
   under the name `onchange_self`.
5. Each action's answer is merged into the handler's answer:
   - values are written back onto the form, after dropping any identifier the action tried to set,
     and keeping only names that are actually fields of the record type;
   - field restrictions are accumulated;
   - a warning replaces any previous warning.
6. Any failure is annotated with the rule identifier and name, as in §2.4 step 6, and re-raised.

**Failure condition**: only code actions may be used here — rule
[`AUT-006`](business-rules.md#aut-006) — because any other kind would write to the database from an
unsaved form.

---

# 4. Run an automation rule on deletion

1. The patched delete operation asks for every active rule whose record type matches and whose
   trigger is the deletion trigger.
2. For each rule, the after-filter is evaluated over the records about to be deleted, with the
   feedback flag set, and the survivors are processed as in §2.4.
3. **Then** the original deletion runs.

The order matters: the actions see the records while they still exist. It is also why message,
follower and activity actions are refused here — rule
[`AUT-007`](business-rules.md#aut-007) — since anything they attach would be deleted an instant
later.

---

# 5. Run an automation rule when a message is posted

1. The patched message-posting operation runs the original posting first, so that the message
   exists.
2. The message is read with elevated rights and with archived records included.
3. The procedure stops, returning the message unchanged, when any of the following holds:
   - the reading context already carries the processed-records mapping, which means this message was
     itself produced by an automation and must not trigger another one;
   - the message is marked as internal;
   - the message's subtype is marked as internal;
   - the message's kind is one of `notification`, `auto_comment` or `user_notification`.
4. The trigger to look for is chosen: `on_message_received` when the message has no author at all,
   or when its author is an external party; `on_message_sent` otherwise. A message with no author is
   treated as having come from outside.
5. Every active rule with that trigger and that record type is fetched, with the before-values
   explicitly absent.
6. For each rule, the **before**-filter is evaluated over the record — not the after-filter — and
   the survivors are processed as in §2.4.
7. The message is returned.

---

# 6. Run the time-based automation scheduler

**Role**: the scheduled job named "Automation Rules: check and execute".

## 6.1 Steps

1. The job starts with an empty processed-records mapping.
2. Every active rule whose trigger is one of the three time triggers is read.
3. For each rule in turn:
   1. The rule is re-read on its own; a rule that has been deactivated or deleted since the job
      started is skipped silently.
   2. An information line naming the rule is logged.
   3. The current instant is taken once and used as the upper bound.
   4. The records to act on are searched by the algorithm of
      [`calculations.md`](calculations.md) §3.2.
   5. Each record found is processed on its own, one call per record.
   6. Everything pending is flushed.
   7. On failure: the transaction is rolled back, the failure is logged with its stack, the failure
      is remembered, and the job moves to the next rule. The rule's last-run stamp is **not**
      advanced, so the same window is retried next time.
   8. On success: the rule's last-run stamp is set to the instant taken in step 3.3, and the job
      reports progress so that a long run can be interrupted and resumed.
4. When any rule failed, the last remembered failure is raised at the end, which marks the job run
   as failed while still having processed the rules that worked.

## 6.2 Failure conditions

| Condition | Effect |
|---|---|
| The rule's trigger date field no longer exists on the record type | A warning naming the rule is logged and the rule finds no records at all. |
| An action fails | That rule's whole window is rolled back and retried at the next run; the other rules still run. |
| The scheduled job's record has been deleted | The interval update abandons silently; the job simply never runs. |

---

# 7. Receive an incoming webhook call

**Role**: any outside system that holds the secret address. The route is public; the secret is the
only credential.

## 7.1 Steps

1. The outside system sends a request — either form of the two accepted methods — to the address
   `/web/hook/<the webhook identifier>`. Cross-site protection is disabled on this route and the
   session is deliberately not saved.
2. The payload is read: the structured body when the request carries one, and the query parameters
   otherwise.
3. The rule whose webhook identifier matches is looked up with elevated rights. **Failure**: no
   match answers with the status `{'status': 'error'}` and the not-found status code.
4. The rule is executed against the payload:
   1. A debug line naming the rule and the payload is logged. When the rule logs its calls, a
      logging entry is created with elevated rights, named "Webhook Log", of kind `server`, at level
      `INFO`, with the path `base_automation(<the rule identifier>)` and an empty function and line.
   2. The record-finding expression is evaluated in the context of
      [`calculations.md`](calculations.md) §2.2. **Failure**: the failure is logged as a warning
      with its stack, a logging entry at level `ERROR` is created when call logging is on, and the
      failure is re-raised.
   3. The resulting record is checked for existence. **Failure**: an absent record logs a warning,
      writes an `ERROR` logging entry when call logging is on, and fails with "No record to run the
      automation on was found." — rule [`AUT-008`](business-rules.md#aut-008).
   4. The record is processed as in §2.4. **Failure**: the failure is logged with its stack, an
      `ERROR` logging entry is written when call logging is on, and the failure is re-raised.
5. Any failure escaping step 4 is answered with `{'status': 'error'}` and the server-error status
   code. Success is answered with `{'status': 'ok'}` and the success status code.

## 7.2 Reading the log

The rule form carries a *Logs* button, visible to the technical-features group and only for a
webhook rule, which opens the logging entries whose path equals `base_automation(<the rule
identifier>)` in list and form view.

## 7.3 Note on the default record finder

The shipped expression reads the two keys `_model` and `_id` out of the payload and browses the
record. Those are exactly the two keys an outgoing webhook of this same system always sends, which
is what makes one database's outgoing webhook work against another database's incoming webhook with
no configuration at all.

---

# 8. Send an outgoing webhook notification

**Role**: an action of the webhook kind, run by an automation rule or by hand.

1. The active record is read. When there is none, the action does nothing.
2. **Failure**: an action with no address fails with "I'll be happy to send a webhook for you, but
   you really need to give me a URL to reach out to..." — rule
   [`AUT-009`](business-rules.md#aut-009).
3. The body is assembled as specified in [`calculations.md`](calculations.md) §2.3.
4. An information line naming the address is logged; the body is logged at debug level.
5. Two callbacks are registered on the transaction:
   - if the transaction rolls back, a warning is logged saying the call was cancelled, and nothing
     is sent;
   - if the transaction commits, the call is made.
6. On commit, the body is posted to the address with the structured-text media type and a
   **one-second** timeout.
7. The answer is checked. A successful answer logs an information line. A timeout logs the warning
   that the call may or may not have failed and that a recurring timeout suggests the far end is
   slow or not working. Any other failure is logged.

The delivery is deliberately send-and-forget: the user is never made to wait for the far end, and a
failure never rolls the business operation back.

---

# 9. Import a data file

**Role**: any internal user, for any record type they may create. A record rule limits each user to
their own import sessions.

## 9.1 Upload

1. The client creates an import session naming the target record type.
2. The file is posted to `/base_import/set_file` with the session identifier. The bytes, the file
   name and the media type the browser reported are written onto the session. The answer is the
   write result.

## 9.2 Preview and mapping proposal

The client asks for a preview, passing the options mapping and the number of preview lines, ten by
default.

1. The importable-field tree of the target record type is built
   ([`calculations.md`](calculations.md) §5.1).
2. The file is read ([`calculations.md`](calculations.md) §5.2). **Failures**: an unreadable file, a
   file in an unsupported format, a missing reader library, a bad quoting character or an undecodable
   text file — rules [`AUT-020`](business-rules.md#aut-020) to
   [`AUT-026`](business-rules.md#aut-026). Every failure is caught and returned as a preview answer
   carrying the failure text and, for a delimited file, the first two hundred bytes decoded with a
   never-failing single-byte encoding so that the user can see what went wrong.
3. **Failure**: a file of zero usable length fails with "Import file has no content or is corrupt" —
   rule [`AUT-027`](business-rules.md#aut-027).
4. The first ten rows are kept as the preview.
5. When the options say the file has headings, the first preview row is removed and kept as the
   headings, and the column types are inferred from the remaining preview rows
   ([`calculations.md`](calculations.md) §5.3).
6. The column-to-field proposal is built:
   - when the client asked to keep its own mapping and supplied one, that mapping is used unchanged;
   - otherwise, when the file has headings, the proposal of
     [`calculations.md`](calculations.md) §5.4 is computed and then deduplicated by
     [`calculations.md`](calculations.md) §5.6;
   - otherwise there is no proposal.
7. The advanced editor is switched on when the client asked to keep it on, or, on a fresh parse,
   when any heading or any proposed mapping crosses a relation.
8. For each column, up to five example values are collected from the preview: the first five
   non-empty text values, each truncated to fifty characters with an ellipsis appended when it was
   longer, and date or date-and-time values rendered with the configured patterns. A column with no
   example at all gets a single empty string.
9. Whether more rows remain than one batch can take is determined: when the preview count already
   exceeds the batch size, by comparing the preview length to the batch size; otherwise by looking
   for a row at the position one batch beyond the preview.
10. The answer carries the field tree, the proposal, the headings, the inferred types, the examples,
    the options as they now stand — the reader will have filled in the encoding, the separator and
    the two date patterns — the advanced flag, whether the reader is in the technical-features
    group, whether more rows remain and the total row count.

## 9.3 Test run and real run

Both use the same operation; the only difference is a flag that rolls the changes back at the end.

1. A savepoint is taken.
2. The chosen columns are converted to a dense matrix
   ([`calculations.md`](calculations.md) §5.7). **Failures**: no column chosen, or a heading row of
   a different width from the first data row — rules [`AUT-028`](business-rules.md#aut-028) and
   [`AUT-029`](business-rules.md#aut-029).
3. Dates, decimal numbers and binary values are parsed and converted in place
   ([`calculations.md`](calculations.md) §5.8, §5.9 and §5.10). **Failures**: rules
   [`AUT-030`](business-rules.md#aut-030) to [`AUT-036`](business-rules.md#aut-036). Any of these
   returns immediately with a single message and imports nothing.
4. File names carried in binary columns are extracted and removed from the data, so that the loader
   sees an empty value where a file name stood.
5. Columns mapped onto the same field are merged ([`calculations.md`](calculations.md) §5.11).
6. Fallback values are applied ([`calculations.md`](calculations.md) §5.12).
7. The rows are handed to the platform's loader with four reading-context keys: the import marker,
   the fields allowed to create missing linked records from a name, the fields whose unrecognised
   values become empty, the fields whose unrecognised values skip the row, and the batch size.
8. The savepoint is released, or rolled back when this was a test run. On a test run the whole cache
   is cleared, because identifiers created during the run may have entered it, and pending registry
   changes are discarded so that other workers are not told about a rollback.
9. When the run created at least one record and the file had headings, the column mappings are
   remembered: for each column with a heading, the existing mapping for this record type and heading
   is repointed when it names a different field, and created when it does not exist.
10. When one of the imported fields is the name field, the answer carries the list of names, padded
    at the front with as many empty strings as rows were skipped and at the back to the full length.
    Otherwise it carries an empty list.
11. The loader's "next row" marker is shifted forward by the number of skipped rows so that the
    client can resume from the right place.
12. The extracted binary file names are returned when there were any.

## 9.4 Failure conditions summarised

| When | Effect |
|---|---|
| Reading the file fails | The preview answer carries the failure text and a byte preview; nothing is imported. |
| Pre-validation fails | A single message is returned; nothing is imported; the savepoint is not even taken for the load. |
| The loader reports errors | The savepoint is rolled back and the errors are returned row by row. |
| The run was a test run | The savepoint is rolled back whatever happened, the cache is cleared and the registry changes are discarded. |

---

# 10. Resume a batched import

**Role**: the same user, continuing a large import.

1. The first run is submitted with a batch size in the options. The loader stops after that many
   rows and reports the next row to process.
2. The client adds the reported position to the options under the skip key and submits again.
3. The conversion drops exactly that many rows **after** removing the heading row and after
   discarding rows that became entirely empty, which is why the skip is applied at the very end of
   the conversion rather than while reading.
4. The names answer is padded at the front with as many empty strings as rows were skipped, so that
   the client can line the names up with the original file.
5. The procedure repeats until the loader reports no next row.

**Failure condition**: a batch that fails rolls back only that batch; the batches already committed
are kept, and the client resumes from the reported position.

---

# 11. Import a packaged module archive

**Role**: administrator. The uploading operation refuses anyone else — rule
[`AUT-040`](business-rules.md#aut-040).

## 11.1 Steps

1. Open *Import Module* from the technical menu, or arrive there from the remote directory
   (§12).
2. Choose the archive. **Failures**: no file — rule [`AUT-041`](business-rules.md#aut-041); a file
   that is not an archive — rule [`AUT-042`](business-rules.md#aut-042); any member of the archive
   larger than one hundred mebibytes — rule [`AUT-043`](business-rules.md#aut-043).
3. Optionally switch on *Force init*, visible only to the technical-features group, and *Load demo
   data*.
4. Press *Install*.
5. The archive is opened and every member's size is checked before anything is extracted.
6. A temporary working directory is created. Manifest members — those exactly one directory deep
   whose file name is one of the accepted manifest names — are extracted and sorted by package name.
7. For each manifest, the package description is read and two lists are built: the data files worth
   extracting, taken from the data, initialisation and update lists and, when demonstration data was
   asked for, the demonstration list, keeping only the three accepted extensions; and the package's
   dependency list.
8. The package directories present in the archive are compared with the dependency graph.
   **Failure**: a directory with no manifest is refused — rule
   [`AUT-044`](business-rules.md#aut-044).
9. Only three kinds of member are extracted: the data files listed in step 7, everything under each
   package's static directory, and every translation file under each package's translation
   directory. Nothing else is written to disk.
10. The packages are installed in dependency order (§11.2). **Failure**: any failure during one
    package's installation is wrapped as "Error while importing module '%(module)s'.\n\n
    %(error_message)s \n\n" carrying the package name and the full stack — rule
    [`AUT-045`](business-rules.md#aut-045). Nothing is committed, so a failed archive leaves no data.
11. The browser is sent to the application root.

## 11.2 Installing one package from the working directory

1. The website scoping carried in the session is put aside and restored at the end, so that the
   import does not become website-specific.
2. The package description is read. When it cannot be read, the package is skipped and the
   procedure reports that nothing was installed.
3. The catalogue values are derived from the description. The icon is taken from the description, or
   defaults to the package's own static description icon when that file exists; when neither exists
   the platform's default icon is kept. The latest version is taken from the description. When the
   reading context marks this as a data package, the catalogue kind becomes `industries`. When
   demonstration data was asked for, the demonstration flag is set.
4. The dependencies that are not yet installed are computed.
   - **Failure**: a dependency that is neither installed nor even present in the catalogue is
     refused with "Unknown module dependencies:" followed by one indented line per missing name —
     rule [`AUT-046`](business-rules.md#aut-046).
   - Otherwise the missing dependencies are installed immediately.
   - When there is no missing dependency at all, and the customisation package is not installed, and
     any data file in the archive carries a customisation marker, the import is refused with
     "Studio customizations require the Odoo Studio app." — rule
     [`AUT-047`](business-rules.md#aut-047). The marker is a context mapping carrying a
     customisation key on any record of any data file.
5. If the package already exists in the catalogue, it is written to state `installed` with the
   derived values and the load mode is *update*, or *initialisation* when *Force init* was chosen.
   If it does not exist, it is created as installed and imported, and the load mode is
   *initialisation*.
6. The exclusion list is built from the description's exclusion patterns, resolved against the
   package directory.
7. Data files are loaded in order: first the data list, then the initialisation list, then, when
   asked, the demonstration list. Only the three accepted extensions are loaded; anything else is
   skipped with an information line. A delimited file in the initialisation list is loaded as
   never-updated. Every external identifier a loaded file produced is recorded; those belonging to
   an excluded file are additionally registered under the counting-exclusion package so that the
   code-counting report ignores them.
8. Every file under the static directory becomes an attachment: named after the file, addressed as a
   slash, the package name and the path inside the package, attached to the view record type, of
   kind binary, holding the file's bytes, and marked public when the attachment entity supports it.
   An existing attachment at the same address is rewritten; a new one also receives an external
   identifier in the package. A static file matching an exclusion pattern additionally receives an
   external identifier under the counting-exclusion package.
9. Every translation file directly under the translation directory becomes an attachment named
   `<package>_<language>.po`, addressed `/<package>/i18n/<language>.po`, attached to the package
   record, of kind binary. Sub-directories are not supported and are ignored.
10. Asset declarations are created from the description's asset lists. **Failure**: a path
    containing a wildcard is refused with "The assets path in the manifest of imported module
    '%(module_name)s' cannot contain glob wildcards (e.g., *, **)." — rule
    [`AUT-048`](business-rules.md#aut-048). A path that does not start with a slash gains one.
    Existing declarations with the same name are rewritten; new ones are created and given an
    external identifier in the package.
11. Translations are loaded for every installed language, overwriting existing ones.
12. When the knowledge package is installed and the imported package ships a welcome article and its
    body template, the article's body is rendered in the reader's language and written.
13. The catalogue record is refreshed from the description and an information line reports success.

## 11.3 Uninstalling an imported package

The set of imported packages among those being uninstalled is computed **before** the platform's own
uninstall runs, because the imported flag's column disappears when the import package itself is
being removed. After the uninstall, those catalogue rows are deleted outright: an imported package
cannot be reinstalled without its archive, so leaving a row behind would offer an installation that
always fails.

## 11.4 Upgrading an imported package

The platform's upgrade marks packages for upgrade. Immediately afterwards, every imported package
that was marked is set straight back to `installed`. An imported package is never upgraded.

## 11.5 Uploading without a session

A second door exists for automated deployment of data packages: a request to
`/base_import_module/login_upload` carrying a login, a password, the force flag and the archive.
The request authenticates, then requires the authenticated user to be an administrator — **failure**:
"Only administrators can upload a module" — rule [`AUT-049`](business-rules.md#aut-049) — and then
runs the archive installer. Any failure is answered as plain text with the server-error status code.
Two-factor authentication leaves the session without a user, which the administrator check then
rejects.

---

# 12. Install a package from the remote directory

**Role**: administrator.

## 12.1 Browsing

1. The application list is opened with a filter asking for the `industries` catalogue kind.
2. Instead of reading the local catalogue, the remote directory is called: the current major release
   string, the fields being asked for — always including the name — the catalogue kind, the package
   name when one is being read, the filter, the page size and the page offset are posted as
   structured text to the directory's listing path. The answer is cached for the life of the process
   under the exact payload.
   **Failures**: a protocol failure is refused with "The list of industry applications cannot be
   fetched. Please try again later" — rule [`AUT-050`](business-rules.md#aut-050); a connection
   failure is refused with "Connection to %s failed The list of industry modules cannot be fetched"
   carrying the directory's address — rule [`AUT-051`](business-rules.md#aut-051).
3. Packages already present in the catalogue are matched by name and kept as they are. The others
   are turned into unsaved catalogue rows: their icon address is prefixed with the directory's
   address, their state is set to `uninstalled`, their catalogue kind is set, and their site address
   is built from the directory address, the fixed path segment, the major release and the package
   name.
4. The category condition is removed from the filter before it is applied locally, because the
   remote categories are not the local ones, and the directory has already applied it.
5. Each row's identifier is its real identifier when it exists locally, and the **negated** remote
   identifier when it does not, so that the client can tell the two apart.
6. The category range shown in the search panel is likewise fetched from the directory and cached
   for the life of the process. Both directory calls use a five-second timeout, and both answer with
   an empty list rather than failing when the directory cannot be reached.

## 12.2 Refusal of an unsupported filter

A filter that mentions the catalogue kind with anything other than an equality to `industries` or a
single-valued membership containing `industries` is refused outright — rule
[`AUT-052`](business-rules.md#aut-052).

## 12.3 Installing

1. Press the install button on a remote package. **Failure**: a non-administrator is refused — rule
   [`AUT-053`](business-rules.md#aut-053).
2. The archive is downloaded from the directory's download path, built from the package name and the
   major release, with a five-second timeout. **Failures**: a protocol failure is refused with "The
   module %s cannot be downloaded" — rule [`AUT-054`](business-rules.md#aut-054); a connection
   failure is refused with "Connection to %(url)s failed, the module %(module)s cannot be
   downloaded." — rule [`AUT-055`](business-rules.md#aut-055).
3. The archive's dependencies are examined without extracting it: every manifest one directory deep
   is read; each manifest larger than one hundred mebibytes is refused — rule
   [`AUT-043`](business-rules.md#aut-043). Dependencies that are neither installed nor contained in
   the archive are split into those present in the catalogue and those that are not.
4. When anything is missing from the catalogue, the import is refused, showing the description built
   in §12.4.
5. Otherwise an import wizard is created holding the downloaded archive, the status `init` and the
   dependency description, and its form is opened with the data-package marker in the reading
   context — which hides the file chooser and makes the installed packages carry the `industries`
   catalogue kind. The dialogue is titled "Install an Industry".

## 12.4 The dependency description

| Case | Text |
|---|---|
| Something is missing | "The installation of the data module would fail as the following dependencies can't be found in the addons-path:\n" followed by one line per missing name prefixed with "- ", then "\nYou may need the Enterprise version to install the data module. Please visit https://www.odoo.com/pricing-plan for more information.\nIf you need Website themes, it can be downloaded from https://github.com/odoo/design-themes.\n" |
| Nothing is missing | "Load demo data to test the industry's features with sample records. Do not load them if this is your production database." |

Both texts are reproduced verbatim; the second is what the wizard shows above the demonstration-data
switch.

---

# 13. Request the activation of a package

**Role**: any internal user for the request; settings administrator for the review.

1. The user opens the application list. The package catalogue is readable by every internal user
   because the request package grants that read right. The technical management menu is hidden from
   everyone by the same package, which clears its group list.
2. On an uninstalled package that is not a paid package, a user **without** the settings-administration
   group sees a *Request Access* button.
3. Pressing it opens a request dialogue titled "Activation Request of "%s"" carrying the package's
   short description, with the package preset.
4. The recipient list is computed as every user who holds the settings-administration group, directly
   or through an implication.
5. The user types a justification and presses *Request Activation*.
6. One electronic mail is sent per recipient, immediately rather than queued, rendered from the
   shipped template with the light notification layout, with the recipient and the application menu
   identifier placed in the rendering context.
7. The client shows the success notification "Your request has been successfully sent" and closes
   the dialogue.
8. A recipient follows the *Review Request* link in the message, which opens the review screen on
   the package.
9. The review screen lists the package and every application it pulls in
   ([`entities.md`](entities.md) §11.3). **Failures**: no package — rule
   [`AUT-060`](business-rules.md#aut-060); an already installed package — rule
   [`AUT-061`](business-rules.md#aut-061).
10. Pressing *Install App* installs the package immediately and sends the client to the application
    home.

---

# 14. Obtain and use a metered service account

**Role**: any code that needs an outside metered service.

## 14.1 Find or create the account

1. The caller names the service's technical name.
2. Accounts are searched whose service technical name matches and whose company list is either empty
   or intersects the reader's allowed companies, ordered by descending identifier.
3. Accounts found without a token are deleted — with elevated rights, since ordinary users may not
   delete — using a **separate database connection**, so that a later failure that rolls the main
   transaction back does not resurrect them. Pending work is flushed first to avoid a deadlock.
4. If accounts remain:
   - those with a non-empty company list are preferred, and the first of them is returned;
   - otherwise the first account is returned.
5. If none remains, the service record is looked up by technical name. **Failure**: no such service
   fails with "No service exists with the provided technical name" — rule
   [`AUT-070`](business-rules.md#aut-070).
6. While tests are running, an account is created with elevated rights on the main connection and
   returned, so that tests never commit.
7. Otherwise a **separate database connection** is opened again, pending work is flushed, the search
   is repeated, and an account is created when there is still none — unless the caller asked not to
   create one, in which case nothing is returned. The token is read on that connection and then
   seeded into the main connection's cache, so the caller can use it without re-reading. The reason
   for the second connection is that an exhausted balance raises a failure that rolls the main
   transaction back, which would otherwise undo the account creation and leave the process unable to
   proceed.

## 14.2 Use the account

1. The caller reads the account's token with elevated rights, because the token is readable only by
   the settings-administration group.
2. The caller posts to the outside service, passing the token and the database's universally unique
   identifier.
3. **Failures**: the service refuses a call made while tests are running with "Unavailable during
   tests."; a timeout fails with "The request to the service timed out. Please contact the author of
   the app. The URL it tried to contact was %s"; any other transport or protocol failure fails with
   "An error occurred while reaching %s. Please contact Odoo support if this error persists."; an
   exhausted balance raises the distinct insufficient-balance failure carrying the remaining
   balance, the service name, the purchase address and the message "You don't have enough credits on
   your account to use this service." — rules [`AUT-071`](business-rules.md#aut-071) to
   [`AUT-074`](business-rules.md#aut-074).

## 14.3 Tell the user

The electronic-mail bridge package adds four notifications, each pushed to the calling user's own
browser channel under the name `iap_notification`:

| Notification | Payload |
|---|---|
| Success | the message and the kind `success`, with an optional title |
| Failure | the message and the kind `danger`, with an optional title |
| Any status | the message and the given kind, with an optional title |
| Balance exhausted | the title, the kind `no_credit` and the purchase address for the named service |

---

# 15. Watch and top up a balance

**Role**: settings administrator, or any internal user for reading.

1. Open *In Application Purchase* then *In Application Purchase Accounts* under the technical menu.
2. Opening the list triggers the refresh of §3 of [`state-machines.md`](state-machines.md): the
   outside service is asked for the balance, the alert threshold, the registration state and the
   lock flag of every account being read, passing each account's token and the database identifier.
3. Each answered token is matched against the stored tokens with a constant-time comparison, so that
   a wrong answer cannot be used to probe tokens.
4. For each match the balance is formatted by [`calculations.md`](calculations.md) §4.1 and written
   together with the other three values, with tracking switched off and with the
   update-suppression flag set so that the write does not push the alert configuration back out.
5. The user may set an alert threshold and choose the recipients. **Failures**: a negative threshold
   — rule [`AUT-075`](business-rules.md#aut-075); a recipient with no electronic mail address —
   rule [`AUT-076`](business-rules.md#aut-076). The form makes the recipient list required as soon
   as the threshold is above zero and hides it when the threshold is zero.
6. Saving pushes the new alert configuration to the outside service: the token, the threshold and,
   for each recipient, the address and the language code — the recipient's own language, or the
   reading language when they have none. A failure here is logged as a warning and does not prevent
   the save.
7. Pressing *Buy Credit* opens the purchase address built by
   [`calculations.md`](calculations.md) §4.2 in the browser.

---

# 16. Enrich a company from the outside directory

## 16.1 The automatic one-time enrichment

**Role**: the system, at company creation.

1. A company is created. While tests are running, the enrichment flag is simply set to true and
   nothing else happens.
2. Otherwise the automatic enrichment runs. It does nothing at all unless the reader is a system
   user, the registry is ready and demonstration data is not being installed.
3. Every company whose flag is false is enriched; then the flag is set on all of them, which makes
   the procedure impossible to loop.
4. Enriching one company:
   1. The company's internet domain is derived
      ([`calculations.md`](calculations.md) §10.1). When there is none, nothing happens.
   2. The directory is asked to enrich by that domain, with a **five-second** timeout rather than
      the usual fifteen.
   3. An empty answer or an answer carrying a failure stops the procedure.
   4. The answer is filtered down to keys that are real fields of the company's Contact, that carry
      a value, and that are either the main image or a field the Contact has not already filled.
   5. The country and the state, which arrive as descriptions, are reduced to their identifiers.
   6. The remaining values are written onto the company's Contact.

## 16.2 The manual lookup

**Role**: any user editing a Contact or a company.

1. The user types in the name, the tax registration number or the company registration number field,
   all of which carry the autocomplete widget.
2. The client calls the search by name or the search by tax registration number, passing the typed
   text and a country: the one the user chose, the company's own country when the caller passed
   nothing, and no country at all when the caller passed zero deliberately.
3. Each suggestion is translated by [`calculations.md`](calculations.md) §10.2 and offered.
4. When the search by tax registration number returns nothing, a cross-border verification service
   is consulted directly as a fallback, with the same timeout. A valid answer whose name is not the
   three-hyphen placeholder is turned into a single suggestion: the name, the number, the first
   address line as the street, the first line beginning with a digit split once on a space into the
   postal code and the city, the remaining line as the second street line, and the answered country
   code. A failure is logged as a warning and treated as no answer.
5. Choosing a suggestion may trigger an enrichment by company registration number, by regional tax
   number or by internet domain. The answer is processed by
   [`calculations.md`](calculations.md) §10.3, which turns an exhausted balance into the marker
   `Insufficient Credit`, a directory failure into "Unable to enrich company (no credit was
   consumed).", and a transport failure into its own text.
6. When the tax-number validation package is installed and the answer carries a number and the
   reading context carries the enriched company data, the number is validated against the answered
   country and silently emptied when it does not pass.
7. The result may be posted on the Contact as an internal note rendered from the shipped enrichment
   template, carrying the Contact's telephone number, name, address, site address, image, tax
   registration number or company registration number, the answered entity kind and the answered
   industry codes.

---

# 17. Resolve an address to coordinates

**Role**: directory maintainer.

1. Open the general settings and choose the provider. The choice is stored in the system parameter
   `base_geolocalize.geo_provider`. For the second provider, paste the service key; it is stored in
   `base_geolocalize.google_map_api_key`.
2. Open a Contact and press the resolve button.
3. The operation returns immediately, doing nothing, unless the reading context carries the force
   flag, when any of the following holds: an import is running, a test is running, the registry is
   not ready, or demonstration data is being installed.
4. The Contact is read in the base language, deliberately, because the providers expect country
   names in that language.
5. The query string is built from the street, the postal code, the city, the state name and the
   country name ([`calculations.md`](calculations.md) §13.1).
6. The provider is called with the country name as a constraint.
   **Failures**: an unimplemented provider — rule [`AUT-080`](business-rules.md#aut-080); a missing
   service key for the second provider — rule [`AUT-081`](business-rules.md#aut-081); a transport
   failure — rule [`AUT-082`](business-rules.md#aut-082); a paid-feature refusal from the second
   provider — rule [`AUT-083`](business-rules.md#aut-083).
7. On a hit, the first result's latitude and longitude are taken and written onto the Contact
   together with today's date in the reader's time zone.
8. On a miss, a second attempt is made with a query string built from the city, the state and the
   country only.
9. Contacts that still found nothing are reported to the user in one browser notification titled
   "Warning" with the message "No match found for %(partner_names)s address(es)." carrying the
   comma-separated display names.

Writing any of the street, the postal code, the city, the state or the country onto a Contact resets
both coordinates to zero, unless the same write also sets both of them; this is what makes the
resolve button reappear after an address change. The coordinate fields themselves belong to
[`../contacts-and-organizations/`](../contacts-and-organizations/).

---

# 18. Complete an address while typing

**Role**: any user, including an unauthenticated visitor for the first call.

1. The street field carries the address autocomplete widget, but only when the country catalogue
   enforces a city list.
2. As the user types, the client posts the partial address to `/autocomplete/address`.
3. The service key is read from the system parameter
   `google_address_autocomplete.google_places_api_key`, but only for an internal user; for anyone
   else the key read fails its own assertion and the call answers with an empty result list.
4. A partial address no longer than the minimum length — read from the system parameter
   `google_address_autocomplete.minimal_partial_address_size`, five by default — answers with an
   empty result list without calling anything.
5. The provider's suggestion path is called with the key, the requested answer fields, the input
   kind, the address type, the typed text and, when supplied, a country restriction, a language and
   a session token. The timeout is two and a half seconds. A timeout or an unreadable answer logs
   the failure and answers with an empty result list. An answer carrying a failure message logs it.
6. Each suggestion is returned as a formatted address and an opaque place identifier.
7. When the user chooses a suggestion, the client posts to `/autocomplete/address_full`.
   **Failure**: a non-internal user is refused with "You don't have access to the full autocomplete
   feature." — rule [`AUT-084`](business-rules.md#aut-084).
8. The provider's detail path is called with the key, the place identifier and the two answer fields.
   A timeout or an unreadable answer answers with no address at all.
9. The answer components are reduced to one type each — the first recognised type, or the first type
   at all — sorted by the priority order of [`calculations.md`](calculations.md) §13.4, and
   translated into the standard address fields by that same section.
10. The house number is taken from the answer when present, and otherwise guessed from what the user
    typed ([`calculations.md`](calculations.md) §13.5).

---

# 19. Recycle aged records

**Role**: settings administrator.

## 19.1 Configure a rule

1. Open *Data Cleaning* then *Configuration*, *Rules*, *Recycle Records*.
2. Create a rule. Choose the record type; the name fills itself with the record type's name when it
   is empty.
3. Choose the mode: manual, which proposes candidates, or automatic, which acts at once.
4. Choose the action: archive or delete. **Failure**: choosing archive for a record type that does
   not support archiving is refused with "This model doesn't manage archived records. Only deletion
   is possible." — rule [`AUT-090`](business-rules.md#aut-090).
5. Optionally switch on *Include Archived*, shown only when the action is delete.
6. Optionally write a filter.
7. Optionally choose the time field, the delta and its unit. Only stored date and date-and-time
   fields of the record type may be chosen.
8. For a manual rule, choose who is notified and how often. **Failure**: a frequency of zero or less
   is refused by the database constraint with "The notification frequency should be greater than 0"
   — rule [`AUT-091`](business-rules.md#aut-091). The form requires both the frequency and the
   period as soon as a recipient is chosen.

## 19.2 Search for candidates

Run nightly by the scheduled job "Data Recycle: Clean Records" at three in the morning, and on
demand through *Run Now*.

1. Everything pending is flushed.
2. Every existing candidate of the rules being searched is read, **archived candidates included**,
   and grouped by rule.
3. For each rule:
   1. The filter is parsed; an empty or absent filter means everything.
   2. When a time field, a delta and a unit are all present, the threshold is computed by
      [`calculations.md`](calculations.md) §9.1 and added to the filter as *the time field is at or
      before the threshold*.
   3. The record type is searched, with archived records included when the rule says so.
   4. Originals that already have a candidate for this rule are skipped.
   5. For an automatic rule, the candidates are created in batches of five thousand and each batch
      is validated immediately; outside a test run each batch is committed.
   6. For a manual rule, the candidates are accumulated and created at the end in batches of fifty
      thousand, each batch committed outside a test run.

## 19.3 Notify

Also run by the nightly job, after the search.

1. Every manual rule is examined. A rule with no recipients or a frequency of zero is skipped.
2. The period is turned into a duration: days, weeks or months.
3. When the rule has never notified, or its last notification plus the duration is now in the past,
   the last-notification stamp is set to the current moment and a notification is sent.
4. The notification counts the candidates of that rule created on or after today minus the duration.
   When the count is zero, nobody is notified.
5. Otherwise a message is sent to the recipients' Contacts, on the rule itself, with the subject
   "Data to Recycle" and a body rendered from the shipped template: "We've identified <count>
   records to clean with the '<record type name>' recycling rule." followed by a line offering a
   link to the candidate list.

## 19.4 Decide

1. Open *Data Cleaning* then *Recycle Records*. The side panel groups the candidates by rule.
2. Each row offers *Validate* and, while the candidate is still pending, *Discard*.
3. *Validate* archives or deletes the original with elevated rights according to the rule's action,
   and then deletes the candidate.
4. *Discard* clears the candidate's activation flag. The *Discarded* filter shows discarded
   candidates.
5. A candidate whose original has been deleted elsewhere shows the name `**Record Deleted**` and is
   simply removed when validated.

## 19.5 Retire a rule

Archiving a rule deletes every candidate it had found, before the archive itself is written.
Deleting the rule deletes them by cascade.

---

# 20. Handle a privacy request

**Role**: settings administrator.

1. Open a Contact or a User and choose *Privacy Lookup* from the action menu; the wizard opens with
   the name and the electronic mail address filled in. For a User, the action reads them from the
   User's Contact. Both actions are restricted to the settings-administration group.
2. Press *Lookup*. **Failure**: an address that does not normalise is refused with "Invalid email
   address “%s”" — rule [`AUT-095`](business-rules.md#aut-095). The quotation marks in that message
   are the typographic ones and are reproduced as such.
3. The query of [`calculations.md`](calculations.md) §11 is built and run after flushing everything
   pending. It searches, in one pass: Contacts by normalised address or by name; Users by login or
   through their Contact's address or name; messages by authorship; and every other non-transient
   record type that has a table, by its address-like fields, by its display-name field and by every
   stored non-cascading link to a Contact.
4. The whole line list is replaced by the result and the line list opens, grouped by record type.
5. For each line the interface offers:
   - an archive switch, shown only when the record type supports archiving and the record has not
     been deleted;
   - a *Delete* button, shown until the record is deleted, guarded by the confirmation "This
     operation is irreversible. Do you wish to proceed to the record deletion?";
   - an *Open Record* button, shown only when the record can be read;
   - two mass operations on the selection, *Archive Selection* and *Delete Selection*.
6. Toggling the archive switch writes it onto the found record with elevated rights and records
   `Archived <record type name> #<identifier>` or `Unarchived <record type name> #<identifier>` in
   the line's execution details.
7. Pressing *Delete* deletes the found record with elevated rights, records `Deleted <record type
   name> #<identifier>`, and marks the line as deleted. **Failure**: deleting an already deleted
   line is refused with "The record is already unlinked." — rule
   [`AUT-096`](business-rules.md#aut-096).
8. Every change to a line's execution details recomputes the wizard's own execution details and, as
   a side effect of that recomputation, writes the Privacy Log: the log is created the first time
   there is anything to record, carrying the masked name, the masked address, the execution details
   and the record description, and is updated on every later change. One session therefore produces
   exactly one log.
9. The log is read under *Privacy* then *Privacy Logs*. Creation is disabled there.

**Note on visibility.** Lines are produced by a direct query, so a record the reader may not read
through a record rule still appears — with its identifier and its record type, but with no clickable
reference and no name. That is deliberate: the point of the procedure is to prove that nothing was
missed.

---

# 21. Store an attachment in the cloud

**Role**: settings administrator for the configuration; any user for the upload.

## 21.1 Configure

1. Open the general settings, section *Cloud Storage*.
2. Choose the provider and fill its fields:
   - first provider: the account name, the container name, the tenant identifier, the client
     identifier and the client secret;
   - second provider: the bucket name and the service account key file, which is decoded and stored
     as the account description.
3. Set the minimum file size in megabytes; it is stored as a byte count.
4. Save. **Failures**: switching away from a provider whose attachments are still in use is refused —
   rules [`AUT-100`](business-rules.md#aut-100) and [`AUT-101`](business-rules.md#aut-101);
   enabling a provider whose configuration is incomplete is refused with "Please configure the Cloud
   Storage before enabling it" — rule [`AUT-102`](business-rules.md#aut-102).
5. When the configuration changed and is complete, the provider is verified: a probe blob named `0/`
   followed by the current instant and `.txt` is uploaded and then downloaded through freshly signed
   addresses, with a five-second timeout each. **Failures**: rules
   [`AUT-103`](business-rules.md#aut-103) to [`AUT-106`](business-rules.md#aut-106). For the second
   provider the bucket's cross-origin rules are also written, allowing any origin, the read and
   write methods, the media-type and disposition headers, and a maximum age equal to the download
   address lifetime; a failure is refused with rule
   [`AUT-106`](business-rules.md#aut-106).
6. The first provider also offers *Invalidate Cached Azure User Delegation Key*; setting it raises
   the stored key sequence by one, which invalidates the cached signing key.

## 21.2 Upload

1. The session description tells the client the minimum file size and the record types whose
   attachments must never go to the cloud.
2. For a file at or above that size on a supported record type, the client posts the upload with the
   cloud flag set. **Failure**: when the provider has been switched off since the page was loaded,
   the answer is the failure message "Cloud storage configuration has been changed. Please refresh
   the page." — rule [`AUT-107`](business-rules.md#aut-107).
3. The attachment is created normally, then converted: the media type is preserved deliberately, the
   bytes are dropped, the kind becomes `cloud_storage` and the address becomes the blob address
   built from the attachment identifier, a fresh universally unique identifier and the file name.
   **Failure**: no provider configured is refused with "Cloud Storage is not enabled" — rule
   [`AUT-108`](business-rules.md#aut-108).
4. The answer gains an upload description: a signed address, the request method, the expected
   success status and the request headers. The client then puts the bytes straight to the provider,
   without passing them through this system.

## 21.3 Download

A download request for a cloud attachment is answered with a redirection to a freshly signed
address, cached until ten seconds before that address expires. The signing formulas are in
[`calculations.md`](calculations.md) §14.

## 21.4 Bring an attachment back

An attachment can be brought back to local storage: its blob is downloaded through a signed address
with a ten-second timeout and stored as bytes, the address is cleared and the kind becomes binary.
**Failure**: rule [`AUT-109`](business-rules.md#aut-109).

---

# 22. Link a mail server to an outside provider

**Role**: administrator for a shared server; any user for their own personal server.

## 22.1 Prepare the server

1. Create the mail server and choose the delegated authentication kind. The host, the encryption and
   the port are set automatically: the outgoing hosts are `smtp.gmail.com` and `smtp.outlook.com`
   with encryption `starttls` on port 587; the incoming hosts are `imap.gmail.com` and
   `imap.outlook.com`, secure, on port 993.
2. Fill the user name with the mailbox address. Choosing the kind or changing the user name copies
   it into the sender filter, because such a server may only send as its own mailbox.
3. Save. **Failures**: a password on a delegated server — rules
   [`AUT-110`](business-rules.md#aut-110) and [`AUT-113`](business-rules.md#aut-113); the wrong
   encryption — rules [`AUT-111`](business-rules.md#aut-111) and
   [`AUT-114`](business-rules.md#aut-114); a missing user name — rules
   [`AUT-112`](business-rules.md#aut-112) and [`AUT-115`](business-rules.md#aut-115); an incoming
   server that is not marked secure — rules [`AUT-116`](business-rules.md#aut-116) and
   [`AUT-117`](business-rules.md#aut-117).

## 22.2 Give consent

1. Press the connect button. **Failures**: a non-administrator is refused — rules
   [`AUT-118`](business-rules.md#aut-118) and [`AUT-119`](business-rules.md#aut-119); an address
   that does not normalise is refused — rule [`AUT-120`](business-rules.md#aut-120).
2. When the application identifier and secret are configured, the browser is sent to the provider's
   consent address built by [`entities.md`](entities.md) §21.4, carrying the record's transport name,
   its identifier and a cross-site protection token.
3. When they are not configured, the relay path is used instead. On a community release the
   operation is refused with "Please configure your Gmail credentials." or "Please configure your
   Outlook credentials." — rule [`AUT-121`](business-rules.md#aut-121). Otherwise the relay is asked,
   with a five-second timeout, for a consent address, passing the database identifier and the
   callback address that will receive the tokens. **Failures**: a transport failure — rule
   [`AUT-122`](business-rules.md#aut-122); a relay error token — translated by
   [`entities.md`](entities.md) §21.7.
4. The user consents at the provider.

## 22.3 Return

1. The provider sends the browser back to the callback route with an authorisation code, or with a
   failure. A failure renders the error page with the text "An error occurred during the
   authentication process." and a link back to the application root.
2. The state is parsed. **Failure**: an unparseable state logs an error and answers with the
   forbidden status.
3. The record is fetched. **Failures**: a record type that does not carry the provider behaviour, a
   record that does not exist, or a cross-site protection token that does not match in constant time
   — all answer with the forbidden status after logging an error. These are rules
   [`AUT-123`](business-rules.md#aut-123) to [`AUT-125`](business-rules.md#aut-125).
4. The authorisation code is exchanged for a refresh token, an access token and an expiry instant.
   **Failure**: the exchange failure text is rendered on the error page with a link back to the
   record.
5. For the first provider only, and only when the server has an owner or the reader is not a
   settings administrator, the provider is asked which address the consent was given for.
   **Failures**: an unreadable answer answers with the forbidden status; an unverified address, or
   one that does not normalise to the server's own address, renders the error page with "Oops,
   you're creating an authorization to send from %(email_login)s but your address is
   %(email_server)s. Make sure your addresses match!" — rule
   [`AUT-126`](business-rules.md#aut-126).
6. The record is written: activation flag true, access token, expiry and refresh token.
7. The browser is redirected: to the record's own form when the reader is a settings administrator
   and the record is not that reader's personal outgoing server, and to the reader's own preferences
   otherwise.

## 22.4 Use

Every send or fetch builds the authentication string of
[`calculations.md`](calculations.md) §15.1, refreshing the access token first when it is stale. For
the second provider the refresh also replaces the refresh token.

## 22.5 The relay round trip

The relay flow has four steps: the database asks the relay for a consent address; the browser is
sent there and on to the provider; the provider returns to the relay's own callback with the
authorisation code; the relay exchanges it and sends the browser to this database's relay callback
carrying the record type, the record identifier, the cross-site protection token, the access token,
the refresh token and the expiry. That callback performs the same checks as §22.3 steps 3 and 5 and
then writes the record.

---

# 23. Complete an onboarding panel

**Role**: any user of the screen that carries the panel.

1. The screen asks for its panel, identified by its one-word name, and the panel's progress record
   for the reader's context is found or created. A per-company panel gets one progress record per
   company; a global panel gets one with no company.
2. The panel is rendered from the closing operation name, the fixed record type, the step list, the
   state mapping and the completion message. Rendering consolidates every step that was in
   `just_done` into `done`, exactly once.
3. Each step card shows its illustration, its title and its description, and either its action
   button labelled with the step's button text — falling back to "Let's do it" — when the step is
   not done, or its completion label with its icon — falling back to "All done!" and to a check icon
   — when it is.
4. Pressing a card calls the step's opening operation. What that operation does belongs to whichever
   domain shipped the step.
5. When the step's work is finished, the step is marked as just done: missing progress-step records
   are created for the reader's context and every record in `not_done` moves to `just_done`. A step
   already complete does not move and the caller is told so.
6. Each move recomputes the owning panel progress records: a panel becomes `done` when the count of
   progress steps in `just_done` or `done` equals the count of steps on the panel.
7. The next render sends the panel state `just_done`, which shows the completion message and a
   *Close Panel* button, and consolidates the steps so that the celebration is never shown twice.
8. Pressing *Close Panel*, or the close cross and then *Get them out of my sight!* in the
   confirmation dialogue titled "Hide Onboarding Tips", sets the closed flag; the panel state is then
   sent as `closed` and the banner disappears.
9. An administrator can bring it back with *Toggle visibility*, from the panel list header or the
   panel form header.

**Failure conditions.**

| Condition | Effect |
|---|---|
| A step is linked to a panel with no opening operation | The link is refused — rule [`AUT-130`](business-rules.md#aut-130) |
| Two panels claim the same one-word name | The second is refused — rule [`AUT-131`](business-rules.md#aut-131) |
| A step's per-company flag is changed | Every progress-step record of that step is deleted and the panels refresh their progress records; all completion for that step is lost |
| A step is added to a panel that already has progress | The progress record repoints at the current progress-step records, so the panel drops back to `not_done` until the new step is done |

---

# 24. Run, record and export a guided tour

**Role**: administrator for the catalogue; any internal user for running.

## 24.1 Be offered a tour

1. At every page load, the session description carries the reader's tour switch and the description
   of the next tour to run.
2. The next tour is the first non-recorded tour, in sequence then name then identifier order, that
   the reader has not consumed — and nothing at all unless the reader is an internal user whose tour
   switch is on.
3. The tour switch itself is computed once, at user creation: on for an administrator when no
   package carrying demonstration data is installed and no test is running, off otherwise. It is
   stored and can be written afterwards, and the client can flip it through a dedicated operation.

## 24.2 Run and consume

1. The client runs the tour: it goes to the starting address and walks the steps, each of which
   names an element, an optional instruction, an optional bubble text and a bubble position.
2. At the end the closing message is shown.
3. The client tells the server the tour was consumed. The reader is added to that tour's consumed
   list with elevated rights, and the next tour to run is returned in the same answer.

## 24.3 Share

The tour form shows a copyable address built as the database's base address, `/odoo?tour=` and the
tour's name. Following it starts that tour for whoever opens it.

## 24.4 Record

A tour recorded in the browser is stored as a tour marked as recorded, with one step record per
recorded interaction. Only such a tour shows its step list on the form; a shipped tour's steps live
in program text and the record only describes the tour.

## 24.5 Export

*Export JS* on a tour builds a client script that registers the tour under its name with its
starting address and its step descriptions, stores it as an attachment named after the tour with the
script media type, attached to the tour record, and answers with a download address for it. The
operation is offered as a contextual action bound to the tour form.

---

# 25. Refresh and contribute translations

**Role**: settings administrator.

## 25.1 Load the term cache

1. Open *Translations* then *Transifex Code Translations*. The menu entry runs a code action that
   loads the cache and then opens the list.
2. Loading takes an exclusive table lock **without waiting**. When the lock is held by another
   worker, the operation answers false and does nothing at all; the list still opens, showing
   whatever is already there.
3. With the lock held, the packages default to every installed package and the languages to every
   installed language except the base language.
4. Every pair of package and language already present in the table is skipped, which is what makes
   the operation safe to call repeatedly.
5. For every remaining pair, every source term and its translation are read from the program text
   and created with elevated rights.

## 25.2 Reload

The scheduled job "Transifex: Reload code translations" runs every seven days: it empties the table
and loads it again, so that terms removed from a package disappear and new ones appear.

## 25.3 Contribute

1. Each row shows the source term, the translated value, the package, the language and a link
   labelled "Contribute".
2. The link is built by [`calculations.md`](calculations.md) §16.2 from the platform address stored
   in the system parameter `transifex.project_url`, the project the package belongs to, the
   language's two-part code and the first fifty characters of the source term.
3. A row whose package is not in the project map, whose language has no two-part code, or whose
   language is the base language, has no link at all.
4. The same link is added to every field translation of a record that carries an external identifier
   whose package is part of the current run, so that a translated field value can be corrected at
   the platform as well.

---

# 26. Drive a connected peripheral

**Role**: any user of a screen that uses peripherals; the agent runs on the machine the peripherals
are attached to.

## 26.1 Discover

1. The agent runs one detection loop per connection kind. Each loop lists the devices it can see
   every three seconds, or once only when its delay is zero.
2. Devices that appeared are matched against the registered drivers for that connection kind, taken
   in descending priority order. The first driver that claims the device wins; a device object is
   created, registered under its identifier and started as its own thread.
3. A device no driver claims is registered as unsupported, but only by a loop that allows it,
   carrying the label "Unknown device (<connection kind>)", the identifier, the kind `unsupported`
   and the connection — reported as `direct` for the universal-serial connection kind and as the
   connection kind itself otherwise.
4. Devices that disappeared are disconnected: the device thread is stopped and the registration is
   removed.

## 26.2 Act

1. The client posts to the agent's action path with a session identifier, a device identifier and
   the action data. The request is not protected against cross-site forgery and accepts any origin,
   because it is a local call from a browser to a machine on the same network.
2. When the device identifier is the agent's own identifier, the action `restart_odoo` records a
   success event and restarts the agent after a two-second pause so that the waiting client sees the
   event; any other action simply answers true, which is how a client tests that the long-poll
   protocol works.
3. When the device identifier is one of the nine device kinds rather than a device, the first device
   of that kind is used.
4. An unknown device logs a warning naming the identifier and answers false.
5. The session identifier is copied into the action data and the device's action is called. The
   action name selects the handler; a repeated action-identifier is ignored so that a retried request
   does not print twice. The elapsed time is logged.
6. The handler's answer is wrapped as a success or a failure carrying the original arguments and the
   session, and is published as an event — except for printers and payment terminals, which publish
   their own events.

## 26.3 Listen

1. The client posts to the agent's event path with a session identifier and the devices it cares
   about.
2. A session is registered. Sessions untouched for seventy seconds are dropped.
3. Events older than five seconds are discarded. The first remaining event for a listened device
   that is newer than the client's last-seen marker is returned at once.
4. When there is none, the call waits up to fifty seconds for a new event and returns it, or returns
   nothing.
5. The client reissues the call immediately, which is what makes the protocol a long poll. A failing
   call is retried with a growing delay, starting at one and a half seconds and capped at fifteen.

## 26.4 Other agent operations

The agent also serves a status probe, a status listing per driver, a log download, a home page, a
restart, credential and network management, log-level control, driver reloading and a remote-access
switch. They are catalogued in [`interfaces.md`](interfaces.md) §7.
