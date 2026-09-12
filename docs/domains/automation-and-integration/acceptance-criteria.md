# Acceptance criteria

Numbered scenarios in Given, When and Then form. Every scenario states concrete starting records,
concrete inputs and the exact resulting records, values, messages and states. A rebuild is correct
for this domain when every scenario below passes.

Conventions used throughout:

- The installation has two companies, **Main Company** with identifier 1 and **Second Company** with
  identifier 2, unless a scenario says otherwise.
- The base address is `https://example.test`.
- The clock is 12 September 2026, and times are given in the database's own zone.
- *The administrator* is a user in the settings-administration group; *the clerk* is an internal user
  in no other group; *the visitor* is an unauthenticated caller.
- Reproduced messages are quoted exactly.

Contents:

| Range | Subject |
|---|---|
| 1 – 30 | Automation rules and webhooks |
| 31 – 56 | Data import |
| 57 – 66 | Package import and activation |
| 67 – 78 | Metered outside services |
| 79 – 86 | Geocoding and address completion |
| 87 – 95 | Data recycling |
| 96 – 103 | Privacy handling |
| 104 – 113 | Attachments stored outside |
| 114 – 124 | Delegated mail authorisation |
| 125 – 131 | Onboarding and tours |
| 132 – 140 | Translations, documentation, remote calls, sparse storage |
| 141 – 145 | Several companies |

---

# Automation rules and webhooks

## Scenario 1 — A rule fires on creation

- **Given** the administrator creates an Automation Rule named `Greet new contacts`, watching the
  record type `res.partner`, with the trigger `on_create`, with no after-filter, and with one action
  of the code kind that writes the comment `created by rule` onto the record.
- **And** the rule is saved, so the registry patches are reinstalled.
- **When** the clerk creates a Contact named `Acme Industries`.
- **Then** the Contact exists with the comment `created by rule`.
- **And** the action ran once, not twice, although the action itself wrote to the record
  ([`business-rules.md#aut-017`](business-rules.md#aut-017)).
- **And** the rule's last-run stamp is unchanged, because only time-based rules set it.

## Scenario 2 — A rule with an empty name is refused

- **Given** the administrator is creating an Automation Rule and has chosen the record type
  `res.partner` and the trigger `on_create`.
- **When** the rule is saved with the name left empty.
- **Then** the save is refused by the platform's required-field check naming the field label
  "Automation Rule Name" ([`business-rules.md#aut-001`](business-rules.md#aut-001)).
- **And** no rule exists.

## Scenario 3 — A message trigger on a record type without a thread is refused

- **Given** the record type `base.geo_provider` carries no discussion thread and its display name is
  `Geo Provider`.
- **When** the administrator saves a rule watching `base.geo_provider` with the trigger
  `on_message_received`.
- **Then** the save is refused with "Mail event can not be configured on model Geo Provider. Only
  models with discussion feature can be used."
  ([`business-rules.md#aut-002`](business-rules.md#aut-002)).

## Scenario 4 — A negative delay is corrected on the form and refused on save

- **Given** the administrator is editing a rule with the trigger `on_time`, the trigger date field
  `date_deadline`, the delay mode `after` and the delay 5.
- **When** the delay is typed as −3 while the form is open.
- **Then** the delay becomes 3 and the mode becomes `before`.
- **When** the delay −3 is instead written directly, bypassing the form.
- **Then** the save is refused with "Delay must be positive. Set 'Delay mode' to 'Before' to negate
  the delay." ([`business-rules.md#aut-003`](business-rules.md#aut-003)).

## Scenario 5 — An action targeting another record type is refused

- **Given** a rule named `Stage watcher` watching `crm.lead`.
- **And** a Server Action named `Touch partner` whose record type is `res.partner`.
- **When** the administrator attaches `Touch partner` to the rule and saves.
- **Then** the save is refused with "Target model of actions Touch partner are different from rule
  model." ([`business-rules.md#aut-004`](business-rules.md#aut-004)).

## Scenario 6 — Changing the watched record type detaches mismatched actions

- **Given** the rule `Stage watcher` watches `crm.lead` and has two attached actions, both on
  `crm.lead`, named `Set priority` and `Notify owner`.
- **When** the administrator changes the watched record type to `res.partner`.
- **Then** both actions are detached from the rule, with no refusal.
- **And** the trigger is cleared and recomputed for the new record type.
- **And** the watched-field set, the after-filter, the before-filter, the trigger value and the
  reference are all recomputed.

## Scenario 7 — The live-update trigger refuses a non-code action

- **Given** a rule watching `crm.lead` with one action of the message-posting kind.
- **When** the trigger is set to `on_change` and the rule is saved.
- **Then** the save is refused with "\"On live update\" automation rules can only be used with
  \"Execute Python Code\" action type."
  ([`business-rules.md#aut-006`](business-rules.md#aut-006)).
- **And** while the form was open the interface had already shown the title "Warning" and the
  message "The \"On UI change\" Trigger can only be used with the \"Execute Code\" action type".

## Scenario 8 — The deletion trigger refuses a message action

- **Given** a rule watching `crm.lead` with one action that posts a message.
- **When** the trigger is set to `on_unlink` and the rule is saved.
- **Then** the save is refused with "Email, follower or activity action types cannot be used when
  deleting records, as there are no more records to apply these changes to!"
  ([`business-rules.md#aut-007`](business-rules.md#aut-007)).
- **And** while the form was open the interface had shown the title "Warning" and the message "You
  cannot send an email, add followers or create an activity for a deleted record.  It simply does
  not work.", including its double space.

## Scenario 9 — An action carrying its own warning blocks the rule

- **Given** a rule watching `crm.lead` with one attached action named `Push to shipping` of the
  outgoing-webhook kind whose chosen fields include a field visible only to a group.
- **When** the rule is saved.
- **Then** the save is refused with "Following child actions have warnings: Push to shipping"
  ([`business-rules.md#aut-005`](business-rules.md#aut-005)).

## Scenario 10 — The stage trigger writes the after-filter

- **Given** the record type `crm.lead` has the link field `stage_id` and a stage named `Won` with
  identifier 4.
- **When** the administrator sets the trigger to `on_stage_set` and chooses the reference `Won`.
- **Then** the after-filter becomes the condition that `stage_id` equals 4.
- **And** the watched-field set becomes exactly the Field Definition `stage_id`.
- **And** the before-filter stays empty.

## Scenario 11 — The tag trigger writes both filters

- **Given** the record type `crm.lead` has the multiple-link field `tag_ids` and a tag named
  `Urgent` with identifier 9.
- **When** the administrator sets the trigger to `on_tag_set` and chooses the reference `Urgent`.
- **Then** the after-filter becomes the condition that `tag_ids` contains 9.
- **And** the before-filter becomes the condition that `tag_ids` does not contain 9, so the rule
  fires on the transition rather than on every later write.
- **And** the watched-field set becomes exactly `tag_ids`.

## Scenario 12 — Creation and an empty update are distinguished

- **Given** a rule watching `res.partner` with the trigger `on_write` and one counting action.
- **And** a rule watching `res.partner` with the trigger `on_create` and one counting action.
- **When** a Contact is created and then written to with a mapping that changes nothing.
- **Then** the creation rule's counter is 1 and the update rule's counter is 1.
- **And** the creation itself did not run the update rule.

## Scenario 13 — Editing the after-filter keeps a hand-added watched field

- **Given** a rule watching `crm.lead` with the trigger `on_create_or_write`, the after-filter
  `[("stage_id", "=", 3)]`, and the watched-field set holding `stage_id` and `priority`, the latter
  added by hand.
- **When** the after-filter is changed to `[("user_id", "!=", False)]`.
- **Then** the watched-field set becomes exactly `priority` and `user_id`
  ([`calculations.md`](calculations.md) §1.3).

## Scenario 14 — A live-update rule runs against an unsaved form

- **Given** a rule watching `crm.lead` with the trigger `on_change`, the live-update field set
  holding `partner_id`, and one code action that sets `email_from` from the chosen Contact.
- **When** the clerk opens a new lead form and chooses the Contact `Acme Industries`, whose address
  is `contact@acme.test`.
- **Then** the form's `email_from` shows `contact@acme.test` before anything is saved.
- **And** no record exists in the database yet.

## Scenario 15 — A deletion rule runs before the record disappears

- **Given** a rule watching `res.partner` with the trigger `on_unlink` and one code action that
  writes the record's name into a log line.
- **When** the Contact `Acme Industries` with identifier 3814 is deleted.
- **Then** the log line reads `Acme Industries`.
- **And** the Contact no longer exists.

## Scenario 16 — The scheduler interval is recomputed to four minutes

- **Given** three rules with time triggers: 3 days before a deadline, 2 hours after creation, and 45
  minutes after the last update.
- **And** the scheduled action's interval is 4 hours.
- **When** the third rule is saved.
- **Then** the scheduled action is active and its interval is **4 minutes**
  ([`calculations.md`](calculations.md) §3.1, worked example one).

## Scenario 17 — The scheduler interval is never widened automatically

- **Given** the scheduled action's interval is 4 minutes and the only rule with a 45-minute delay is
  deleted, leaving delays of 3 days and 2 hours.
- **When** the deletion is saved.
- **Then** the computed interval would be 12 minutes, but the stored interval stays **4 minutes**,
  because the interval is replaced only when the new one is shorter.
- **And** the scheduled action stays active, because two time-based rules remain.

## Scenario 18 — The scheduler is switched off when no time-based rule remains

- **Given** the last rule with a time trigger is archived.
- **When** the archive is saved.
- **Then** the scheduled action is inactive.
- **And** its interval is left as it was.

## Scenario 19 — A three-days-before rule fires on the right window

- **Given** a rule with the trigger `on_time`, the trigger date field `date_deadline` of kind date
  and time, the delay 3, the mode `before` and the unit days.
- **And** the rule's last run is 9 September 2026 at 06:00.
- **And** three leads with deadlines 12 September 2026 at 05:59, 12 September 2026 at 06:00 and 15
  September 2026 at 06:00.
- **When** the scheduler turn starts at 12 September 2026 at 06:00.
- **Then** the lead with the deadline 12 September 06:00 fires.
- **And** the lead with 05:59 does not fire, because it is before the lower end.
- **And** the lead with 15 September 06:00 does not fire, because the upper end is exclusive.
- **And** the rule's last run becomes 12 September 2026 at 06:00.

## Scenario 20 — A working schedule skips the weekend

- **Given** the same rule as Scenario 19 but with a working schedule that works Monday to Friday and
  the delay 2 days.
- **And** the rule's last run is Thursday 10 September 2026 at 06:00.
- **When** the scheduler turn starts on Friday 11 September 2026 at 06:00.
- **Then** the window runs from Monday 14 September 2026 at 06:00 to Tuesday 15 September 2026 at
  06:00, so a lead whose deadline is Monday 14 September at 09:00 fires
  ([`calculations.md`](calculations.md) §3.3).

## Scenario 21 — A failing rule does not stop the other rules

- **Given** two rules with time triggers, `A` and `B`, in that order, where `A`'s action always
  fails.
- **When** the scheduler turn runs.
- **Then** `A`'s changes are rolled back and its last run is unchanged.
- **And** `B` runs and its last run is set.
- **And** the turn ends by raising `A`'s failure, which marks the job as failed.

## Scenario 22 — A webhook call runs the rule

- **Given** a rule named `Ship on call` watching `stock.picking`, trigger `on_webhook`, webhook
  identifier `0f5d1c7e-2f14-4a5f-9f2b-1c7e2f144a5f`, call logging switched on, and the shipped
  record-finding expression.
- **And** a Transfer with identifier 7.
- **When** the visitor submits to
  `https://example.test/web/hook/0f5d1c7e-2f14-4a5f-9f2b-1c7e2f144a5f` the structured body
  `{"_model": "stock.picking", "_id": 7}`.
- **Then** the answer is the body carrying `ok` under `status` with the status 200.
- **And** the rule's actions have run against the Transfer with identifier 7.
- **And** one log line exists at level `INFO` whose origin names the rule and whose message reads
  "Webhook #<rule identifier> triggered with payload {'_model': 'stock.picking', '_id': 7}".

## Scenario 23 — An unknown webhook address is not found

- **When** the visitor submits to
  `https://example.test/web/hook/00000000-0000-0000-0000-000000000000`.
- **Then** the answer is the body carrying `error` under `status` with the status 404.
- **And** nothing ran and nothing was logged against any rule
  ([`business-rules.md#aut-018`](business-rules.md#aut-018)).

## Scenario 24 — A webhook that finds no record answers with a failure

- **Given** the rule of Scenario 22.
- **When** the visitor submits the body `{"_model": "stock.picking", "_id": 999999}` and no such
  Transfer exists.
- **Then** the answer is the body carrying `error` under `status` with the status 500.
- **And** one log line exists at level `ERROR` reading "Webhook #<rule identifier> could not be
  triggered because no record to run it on was found."
- **And** the refusal text raised inside was "No record to run the automation on was found."
  ([`business-rules.md#aut-008`](business-rules.md#aut-008)).

## Scenario 25 — Renewing the webhook identifier changes the address

- **Given** the rule of Scenario 22.
- **When** the administrator presses *Renew*.
- **Then** the webhook identifier is a different universally unique identifier.
- **And** the web address changes with it.
- **And** a call to the old address answers with the status 404.

## Scenario 26 — Duplicating a rule gives the copy its own actions and identifier

- **Given** the rule `Ship on call` with two attached actions.
- **When** the administrator duplicates it.
- **Then** the copy has two attached actions that are **different records** from the original's.
- **And** the copy's webhook identifier differs from the original's.
- **And** the original still has its own two actions.

## Scenario 27 — An outgoing webhook sends the exact payload

- **Given** a Server Action named `Notify shipping` with identifier 42, of the outgoing-webhook
  kind, on `stock.picking`, with the address `https://receiver.test/hook` and the chosen fields
  `name` and `partner_id`.
- **And** a Transfer with identifier 7, name `WH/OUT/00013` and customer identifier 3814.
- **When** the action runs and the transaction commits.
- **Then** one submission is made to `https://receiver.test/hook` with a one-second timeout, whose
  body carries, with keys in sorted order, `_action` = `Notify shipping(#42)`, `_id` = 7, `_model` =
  `stock.picking`, `name` = `WH/OUT/00013` and `partner_id` = 3814.

## Scenario 28 — An outgoing webhook without an address is refused

- **Given** the same action with its address cleared.
- **When** it runs on the Transfer with identifier 7.
- **Then** the operation fails with "I'll be happy to send a webhook for you, but you really need to
  give me a URL to reach out to..."
  ([`business-rules.md#aut-009`](business-rules.md#aut-009)).
- **When** it instead runs with no active record at all.
- **Then** nothing happens and no failure is raised.

## Scenario 29 — A rolled-back transaction cancels the outgoing call

- **Given** the action of Scenario 27.
- **When** the action runs and the surrounding transaction is then rolled back.
- **Then** no submission is made.
- **And** one warning line is written reading that the call to `https://receiver.test/hook` was
  cancelled due to a rollback.

## Scenario 30 — A rule's action cannot be reused by a multi-action

- **Given** the action `Notify shipping` carries a back-link to the rule `Ship on call`.
- **When** the administrator edits a multi-action on `stock.picking` and opens its child list.
- **Then** `Notify shipping` does not appear among the candidates
  ([`business-rules.md#aut-012`](business-rules.md#aut-012)).

---

# Data import

## Scenario 31 — A delimited file with headings is previewed

- **Given** the clerk has created a Data Import Session for `res.partner` and posted a file named
  `contacts.csv` of media type `text/csv` holding

```
Name;Email;Phone
Acme Industries;contact@acme.test;+32 10 00 00 00
Bolt Supplies;sales@bolt.test;+32 10 00 00 01
```

- **When** the preview is asked for with the options saying the file has headings and with no
  separator given.
- **Then** the separator is inferred as the semicolon and written back into the options.
- **Then** the headings are `Name`, `Email` and `Phone`.
- **And** the preview holds the two data rows.
- **And** the proposal maps column 0 to `name`, column 1 to `email` and column 2 to `phone`, each
  with the distance 0, because each heading equals a field label exactly.
- **And** the answer reports that no further rows remain and that the total row count is 2.

## Scenario 32 — An unsupported format is reported in the preview

- **Given** a session whose file is named `contacts.docx` with the media type
  `application/vnd.openxmlformats-officedocument.wordprocessingml.document`.
- **When** the preview is asked for.
- **Then** the answer carries the failure text "Unsupported file format
  \"application/vnd.openxmlformats-officedocument.wordprocessingml.document\", import only supports
  CSV, ODS, XLS and XLSX" ([`business-rules.md#aut-020`](business-rules.md#aut-020)).
- **And** nothing is imported.

## Scenario 33 — An empty file is reported as having no content

- **Given** a session whose file holds only blank lines.
- **When** the preview is asked for.
- **Then** the answer carries the failure text "Import file has no content or is corrupt"
  ([`business-rules.md#aut-027`](business-rules.md#aut-027)).
- **And** the answer also carries the first two hundred bytes of the file decoded with a single-byte
  encoding.

## Scenario 34 — A two-character text delimiter is refused

- **Given** the file of Scenario 31 and the options carrying the quoting value `""`.
- **When** the preview is asked for.
- **Then** the answer carries "Error while importing records: Text Delimiter should be a single
  character." ([`business-rules.md#aut-026`](business-rules.md#aut-026)).

## Scenario 35 — An undecodable file names how the encoding was chosen

- **Given** a delimited file whose bytes are not valid in the detected encoding `utf-8`, and the
  options carrying no encoding.
- **When** the preview is asked for.
- **Then** the answer carries "There was an issue decoding the file using encoding “utf-8”.\nThis
  encoding was automatically detected."
- **When** the options instead carry the encoding `utf-8` chosen by the user.
- **Then** the answer carries "There was an issue decoding the file using encoding “utf-8”.\nThis
  encoding was manually selected."
  ([`business-rules.md#aut-025`](business-rules.md#aut-025)).

## Scenario 36 — A heading is matched by distance

- **Given** a file whose second heading is `custumer` and a target record type with the field
  `customer` (Customer).
- **When** the preview is asked for.
- **Then** the distance between `custumer` and `customer` is 0.125
  ([`calculations.md`](calculations.md) §5.6).
- **And** the proposal maps that column to `customer`, because 0.125 is below 0.2.

## Scenario 37 — A heading that is too different is not matched

- **Given** a file whose heading is `Customer` and a target record type whose only candidate field
  is labelled `Customer Reference`.
- **When** the preview is asked for.
- **Then** the distance is 0.3846 and no proposal is made for that column.

## Scenario 38 — Two headings competing for one field are deduplicated

- **Given** a file with the headings `Customer` at column 0, `Client` at column 3 and
  `partner_id/name` at column 5.
- **And** `Customer` matches `partner_id` at 0.08 and `Client` matches `partner_id` at 0.05.
- **When** the preview is asked for.
- **Then** the proposal holds column 3 mapped to `partner_id` and column 5 mapped to the path
  `partner_id` then `name`.
- **And** column 0 has no proposal
  ([`calculations.md`](calculations.md) §5.7).

## Scenario 39 — A remembered mapping beats every distance

- **Given** a Data Import Column Mapping for `res.partner` with the heading `client code` pointing
  at `ref`.
- **And** a file whose first heading is `Client Code`.
- **When** the preview is asked for.
- **Then** the proposal maps that column to `ref` with the distance −1, whatever other field the
  heading resembles.

## Scenario 40 — A trial run changes nothing

- **Given** the file of Scenario 31, the mapping of Scenario 31 and no existing Contact named
  `Acme Industries`.
- **When** the import is executed with the trial flag set.
- **Then** the answer reports two rows loaded and no failure.
- **And** no Contact named `Acme Industries` exists afterwards.
- **And** no Data Import Column Mapping was created.
- **And** the whole cache has been cleared and the pending registry changes discarded.

## Scenario 41 — A real run creates the records and remembers the mapping

- **Given** the same starting point.
- **When** the import is executed without the trial flag.
- **Then** two Contacts exist, named `Acme Industries` and `Bolt Supplies`, with the addresses
  `contact@acme.test` and `sales@bolt.test` and the telephone numbers `+32 10 00 00 00` and
  `+32 10 00 00 01`.
- **And** three Data Import Column Mappings exist for `res.partner`: `Name` to `name`, `Email` to
  `email` and `Phone` to `phone`.
- **And** the answer carries the two names, in order, because `name` is the display-name field.

## Scenario 42 — No column mapped is refused before anything is read

- **Given** a session with a valid file and a mapping in which every entry is empty.
- **When** the import is executed.
- **Then** the answer carries the single message "You must configure at least one field to import"
  ([`business-rules.md#aut-028`](business-rules.md#aut-028)).
- **And** nothing is created.

## Scenario 43 — A width mismatch names both counts

- **Given** a file whose heading row has 3 cells and whose first data row has 4, and a mapping of 3
  entries.
- **When** the import is executed.
- **Then** the answer carries "Error while importing records: all rows should be of the same size,
  but the title row has 3 entries while the first row has 4. You may need to change the separator
  character." ([`business-rules.md#aut-029`](business-rules.md#aut-029)).

## Scenario 44 — A grouped decimal with a currency symbol is parsed

- **Given** a column mapped to a monetary field holding the value `1.234,50 €`, and the options
  carrying the full stop as the grouping character and the comma as the decimal character.
- **And** a currency whose symbol is `€`.
- **When** the import is executed.
- **Then** the stored amount is 1234.50
  ([`calculations.md`](calculations.md) §5.9, worked example one).

## Scenario 45 — Brackets mean a negative number

- **Given** a column mapped to a decimal field holding the value `(1 234.50)` and no separators in
  the options.
- **When** the import is executed.
- **Then** the separators are inferred as the space for grouping and the full stop for the decimal.
- **And** the stored amount is −1234.50.

## Scenario 46 — A value that is not a number names the column and the value

- **Given** a column mapped to the decimal field `credit_limit` holding the value `n/a`.
- **When** the import is executed.
- **Then** the answer carries "Column credit_limit contains incorrect values (value: n/a)"
  ([`business-rules.md#aut-030`](business-rules.md#aut-030)).
- **And** nothing is created.

## Scenario 47 — A date pattern is inferred as day, month, year

- **Given** a column mapped to a date field whose preview values are `01/02/2026`, `15/02/2026` and
  `28/02/2026`.
- **When** the preview is asked for.
- **Then** the pattern `%d/%m/%Y` is chosen and written back into the options, because the
  month-first pattern fails on `15/02/2026`.
- **And** the stored dates are 1 February 2026, 15 February 2026 and 28 February 2026.

## Scenario 48 — A bad date names the line

- **Given** a column mapped to the date field `date_deadline`, the pattern `%d/%m/%Y`, and the third
  data row holding `2026-02-31`.
- **When** the import is executed.
- **Then** the answer carries "Column date_deadline contains incorrect values. Error in line 3:"
  followed by the parser's own description
  ([`business-rules.md#aut-033`](business-rules.md#aut-033)).
- **And** nothing is created.

## Scenario 49 — A date value mapped onto a text field is reported

- **Given** a workbook whose cell in the first column is a true date value, mapped onto the single
  line text field `ref`.
- **When** the import is executed.
- **Then** the answer reports the row as failing with "Field 'ref' does not accept date/time
  values." ([`business-rules.md#aut-037`](business-rules.md#aut-037)).

## Scenario 50 — Two columns on one text field are joined with a space

- **Given** two columns both mapped to the single line text field `name`, and a row holding
  `Value part 1` and `Value part 2`.
- **When** the import is executed.
- **Then** the stored name is `Value part 1 Value part 2`.
- **When** a second row holds `I am Batman` and an empty cell.
- **Then** the stored name is `I am Batman`, with no trailing space
  ([`calculations.md`](calculations.md) §5.11).

## Scenario 51 — Two columns on one long-text field are joined with a line break

- **Given** two columns both mapped to the long text field `comment`, and a row holding `First line`
  and `Second line`.
- **When** the import is executed.
- **Then** the stored comment is `First line` then a line break then `Second line`.

## Scenario 52 — A fallback value replaces an unacceptable selection value

- **Given** the field `state` is a selection whose labels are `New`, `In progress` and `Done`, and
  the options carry the fallback `draft` for it.
- **And** a file with two rows holding `PENDING` and `done`.
- **When** the import is executed.
- **Then** the first row's value becomes `draft` and the second row keeps `done`
  ([`calculations.md`](calculations.md) §5.12).
- **When** the fallback is instead the marker `skip`.
- **Then** the first row's cell becomes empty and the loader skips that row.

## Scenario 53 — A batched import resumes at the right row

- **Given** a file of 1 000 data rows and the options carrying the batch size 400.
- **When** the import is executed with no skip.
- **Then** the answer reports the next row as 400.
- **When** it is executed again with the skip 400.
- **Then** the answer reports the next row as 800, and the answered names list is padded at the
  front with 400 empty texts.
- **When** it is executed again with the skip 800.
- **Then** the answer reports the next row as 1 000 and the client stops
  ([`calculations.md`](calculations.md) §5.13).

## Scenario 54 — A clerk may not import a binary value from a web address

- **Given** a column mapped to the binary field `image_1920` holding
  `https://pictures.test/logo.png`, and the clerk is not in the administration group.
- **When** the import is executed.
- **Then** the answer carries "You can not import file via URL, check with your administrator or
  support for the reason."
  ([`business-rules.md#aut-031`](business-rules.md#aut-031)).
- **And** nothing is created.

## Scenario 55 — A fetched file that is too large is refused

- **Given** the administrator runs the same import, and the configured maximum for an uploaded file
  is 25 000 000 bytes.
- **And** the address answers with an announced length of 30 000 000 bytes.
- **When** the import is executed.
- **Then** the answer carries "File size exceeds configured maximum (25000000 bytes)"
  ([`business-rules.md#aut-035`](business-rules.md#aut-035)).
- **And** when the address instead announces nothing but streams 30 000 000 bytes, the same message
  appears once the running total passes the maximum, checked after each block of 32 768 bytes.

## Scenario 56 — One clerk cannot see another clerk's import

- **Given** the clerk `Alice` has created a Data Import Session and posted a file to it.
- **When** the clerk `Bob` searches the sessions.
- **Then** `Bob` finds nothing ([`business-rules.md#aut-038`](business-rules.md#aut-038)).
- **When** `Bob` attempts to delete his own session.
- **Then** the deletion is refused
  ([`business-rules.md#aut-039`](business-rules.md#aut-039)).

---

# Package import and activation

## Scenario 57 — A package archive installs

- **Given** an archive holding one directory `my_theme` with a manifest that declares no
  dependencies, one data file and two files under the static directory.
- **When** the administrator opens the package import screen, chooses the archive and presses
  *Install*.
- **Then** the package `my_theme` exists, marked as imported and installed.
- **And** its data file's records exist.
- **And** its two static files exist as attachments.
- **And** the screen's status becomes `done` and its message names the installed package.

## Scenario 58 — A file that is not an archive is refused

- **When** the administrator chooses a plain text file and presses *Install*.
- **Then** the operation is refused with "Only zip files are supported."
  ([`business-rules.md#aut-042`](business-rules.md#aut-042)).

## Scenario 59 — An archive member that is too large is refused

- **Given** an archive one of whose members declares a size of 104 857 601 bytes and whose path is
  `my_theme/static/src/big.bin`.
- **When** the administrator presses *Install*.
- **Then** the operation is refused with "File 'my_theme/static/src/big.bin' exceed maximum allowed
  file size" ([`business-rules.md#aut-043`](business-rules.md#aut-043)).
- **And** nothing was extracted.

## Scenario 60 — An archive without a manifest is refused

- **Given** an archive holding the directories `good_module` with a manifest and `bad_module`
  without one.
- **When** the administrator presses *Install*.
- **Then** the operation is refused with "No manifest found in 'bad_module'. Can't import the zip
  file." ([`business-rules.md#aut-044`](business-rules.md#aut-044)).
- **And** `good_module` is not installed either.

## Scenario 61 — An unknown dependency is refused

- **Given** an archive whose manifest depends on `not_a_real_module` and on `sale`, where `sale` is
  known but not installed.
- **When** the administrator presses *Install*.
- **Then** the operation is refused with the text "Unknown module dependencies:" followed by a line
  break and " - not_a_real_module"
  ([`business-rules.md#aut-046`](business-rules.md#aut-046)).
- **And** `sale` is not installed, because the refusal happens before the dependency installation.

## Scenario 62 — A wildcard in an asset path is refused

- **Given** an archive whose manifest declares the asset path `my_theme/static/src/**/*.scss`.
- **When** the administrator presses *Install*.
- **Then** the operation is refused with "The assets path in the manifest of imported module
  'my_theme' cannot contain glob wildcards (e.g., *, **)."
  ([`business-rules.md#aut-048`](business-rules.md#aut-048)).

## Scenario 63 — A failing package leaves nothing behind

- **Given** an archive whose data file refers to a record type that does not exist.
- **When** the administrator presses *Install*.
- **Then** the operation is refused with "Error while importing module 'my_theme'." followed by two
  line breaks, a space, the technical trace, and two more line breaks
  ([`business-rules.md#aut-045`](business-rules.md#aut-045)).
- **And** no package row, no record and no attachment from the archive exists.

## Scenario 64 — Uninstalling an imported package removes its row

- **Given** the imported package `my_theme` is installed.
- **When** the administrator uninstalls it.
- **Then** its records and attachments are gone.
- **And** **no package row** named `my_theme` remains, rather than a row in the uninstalled state
  ([`business-rules.md#aut-058`](business-rules.md#aut-058)).
- **And** while the uninstall confirmation was shown, `my_theme` was not listed among the packages
  that would be removed.

## Scenario 65 — A clerk requests an activation and each reviewer gets one message

- **Given** the settings-administration group has three members, all with an address.
- **And** the package `sale` is uninstalled and is not a paid one.
- **When** the clerk presses *Request Access* on `sale`, writes the justification `We need quotations
  for the new team.` and presses *Request Activation*.
- **Then** three messages are sent, one per reviewer, each addressed to that reviewer's Contact.
- **And** each message's subject is "Module Activation Request for \"Sales\"".
- **And** each body names the clerk, the package's short description and the justification, and
  carries a button labelled "Review Request".
- **And** the clerk sees the notice "Your request has been successfully sent" and the dialogue
  closes.

## Scenario 66 — The review screen lists what would be installed

- **Given** the package `A` depends on `B` and `C`; `B` is an application and depends on `D`; `C` is
  not an application.
- **When** a reviewer opens the review screen for `A`.
- **Then** the listed applications are `A`, `B` and `D`
  ([`calculations.md`](calculations.md) §6.5).
- **When** the package is already installed.
- **Then** the screen refuses to render with "The module is already installed."
  ([`business-rules.md#aut-061`](business-rules.md#aut-061)).
- **When** no package is named at all.
- **Then** the screen refuses to render with "No module selected."
  ([`business-rules.md#aut-060`](business-rules.md#aut-060)).

---

# Metered outside services

## Scenario 67 — The first use of a service creates its account

- **Given** the shipped service `Lead Generation` with the technical name `reveal` and no account.
- **When** a feature asks for the account for `reveal`.
- **Then** one In-Application Purchase Account exists, named `Lead Generation`, linked to that
  service, with an empty company list and a token of thirty-two hexadecimal characters.
- **And** the account was created on a separate database connection, so a later rollback of the
  caller's transaction leaves it in place.

## Scenario 68 — An unknown technical name is refused

- **When** a feature asks for the account for the technical name `not_a_service`.
- **Then** the request is refused with "No service exists with the provided technical name"
  ([`business-rules.md#aut-070`](business-rules.md#aut-070)).
- **And** no account exists.

## Scenario 69 — Tokenless accounts are deleted and a company-scoped account wins

- **Given** three accounts for `reveal`: identifier 11 with no token, identifier 12 with a token and
  an empty company list, identifier 13 with a token and the company list holding Main Company.
- **And** the reader's allowed companies are Main Company only.
- **When** a feature asks for the account for `reveal`.
- **Then** account 11 is deleted, on a separate connection and with elevated rights.
- **And** the account returned is **13**, because accounts with a company list are preferred.
- **When** account 13 is removed and the question is asked again.
- **Then** the account returned is 12.

## Scenario 70 — A whole-unit balance is rounded half to even

- **Given** the account for `reveal`, whose service counts in whole units with the unit name
  `Credits`.
- **When** the service answers with the balance 12.5.
- **Then** the stored balance text is `12 Credits`.
- **When** the service answers with 13.5.
- **Then** the stored balance text is `14 Credits`
  ([`calculations.md`](calculations.md) §4.1).

## Scenario 71 — A fractional balance is rounded to four decimals

- **Given** the account for `partner_autocomplete`, whose service counts in whole units — so, for
  this scenario, a service configured with whole units switched off and the unit name `Enrichments`.
- **When** the service answers with the balance 12.45678.
- **Then** the stored balance text is `12.4568 Enrichments`.

## Scenario 72 — The balance refresh matches tokens in constant time

- **Given** two accounts whose tokens are `aaaa…` and `bbbb…`.
- **When** the list is opened and the service answers with information for the token `bbbb…` only.
- **Then** only the second account is written.
- **And** the write carries the suppression flag, so the alert configuration is not pushed back out.
- **And** the write carries the no-tracking flag, so no message is posted on the account.

## Scenario 73 — A negative alert threshold is refused

- **When** the administrator sets the alert threshold to −1 and saves.
- **Then** the save is refused with "Please set a positive email alert threshold."
  ([`business-rules.md#aut-075`](business-rules.md#aut-075)).

## Scenario 74 — An alert recipient without an address is refused

- **Given** the users `Ann` and `Ben`, where `Ben` has no electronic mail address.
- **When** the administrator sets the threshold to 50 and chooses both as recipients.
- **Then** the save is refused with "One of the email alert recipients doesn't have an email address
  set. Users: Ben" ([`business-rules.md#aut-076`](business-rules.md#aut-076)).

## Scenario 75 — Saving the alert configuration pushes it out

- **Given** the account for `reveal` with the token `9f2b…`, and the users `Ann`, whose language is
  `fr_BE`, and `Cleo`, who has no language, the reading language being `en_US`.
- **When** the administrator sets the threshold to 50, chooses both recipients and saves.
- **Then** one call is made to the alert path carrying the token, the threshold 50, and two recipient
  entries: `Ann`'s address with `fr_BE`, and `Cleo`'s address with `en_US`.
- **When** that call fails.
- **Then** the save still succeeds and one warning line is written.

## Scenario 76 — The purchase address carries a hashed token

- **Given** the account for `partner_autocomplete` with the token
  `9f2b1c7e2f144a5f9f2b1c7e2f144a5f+disabled` and the database identifier
  `c0ffee00-1234-5678-9abc-def012345678`.
- **When** the administrator presses *Buy Credit*.
- **Then** the browser is sent to
  `https://iap.odoo.com/iap/1/credit?dbuuid=c0ffee00-1234-5678-9abc-def012345678&service_name=partner_autocomplete&account_token=<hash>&hashed=1`,
  where the hash is the forty-character digest of `9f2b1c7e2f144a5f9f2b1c7e2f144a5f`, the suffix
  having been removed first ([`calculations.md`](calculations.md) §4.2).

## Scenario 77 — An exhausted balance produces a distinct failure and a notification

- **Given** the account for `reveal` with a balance of 0.
- **When** a feature calls the service and the service answers with the insufficient-balance
  failure carrying the balance 0, the service name `reveal`, the purchase address and the message
  "You don't have enough credits on your account to use this service.".
- **Then** the feature receives that distinct failure with all four values
  ([`business-rules.md#aut-074`](business-rules.md#aut-074)).
- **And** the electronic-mail bridge pushes a notification on the channel `iap_notification`
  carrying a title, the kind `no_credit` and the purchase address for `reveal`.

## Scenario 78 — A metered call that times out and one that cannot be reached

- **When** the call to `https://iap.odoo.com/iap/1/get-accounts-information` does not answer within
  fifteen seconds.
- **Then** the refusal is, on one line because it names an address:
  "The request to the service timed out. Please contact the author of the app. The URL it tried to contact was https://iap.odoo.com/iap/1/get-accounts-information"
  ([`business-rules.md#aut-072`](business-rules.md#aut-072)).
- **When** the call instead answers with a failing status.
- **Then** the refusal is, again on one line:
  "An error occurred while reaching https://iap.odoo.com/iap/1/get-accounts-information. Please contact Odoo support if this error persists."
  ([`business-rules.md#aut-073`](business-rules.md#aut-073)).
- **And** in both cases the balance refresh writes a warning line and leaves the stored balances
  unchanged.

---

# Geocoding and address completion

## Scenario 79 — An address is resolved and written

- **Given** the chosen provider is `openstreetmap`.
- **And** the Contact `Acme Industries` with the street `Chaussée de Namur 40`, the postal code
  `1367`, the city `Ramillies`, no state and the country `Belgium`.
- **When** the administrator presses the resolve button with the force flag in the reading context.
- **Then** the query string sent is `Chaussée de Namur 40, 1367 Ramillies, Belgium`
  ([`calculations.md`](calculations.md) §13.1).
- **And** the provider's first result's latitude and longitude are written onto the Contact.
- **And** the date the coordinates were obtained becomes 12 September 2026.

## Scenario 80 — A miss triggers a second, coarser attempt

- **Given** the same Contact and a provider that answers nothing for the full query.
- **When** the resolve button is pressed.
- **Then** a second query `Ramillies, Belgium` is sent.
- **When** that also answers nothing.
- **Then** the coordinates stay as they were.
- **And** one notification is pushed on the channel `simple_notification` with the kind `danger`,
  the title "Warning" and the message "No match found for Acme Industries address(es)."

## Scenario 81 — The second provider without a key is refused

- **Given** the chosen provider is `googlemap` and the system parameter
  `base_geolocalize.google_map_api_key` is empty.
- **When** the resolve button is pressed.
- **Then** the operation is refused with "API key for GeoCoding (Places) required.\nVisit
  https://developers.google.com/maps/documentation/geocoding/get-api-key for more information."
  ([`business-rules.md#aut-081`](business-rules.md#aut-081)).

## Scenario 82 — The second provider refuses a paid feature

- **Given** the chosen provider is `googlemap` with a key, and the provider answers with a status
  that is neither success nor the empty-result status, and the description `REQUEST_DENIED`.
- **When** the resolve button is pressed.
- **Then** the operation is refused with "Unable to geolocate, received the error:\nREQUEST_DENIED"
  followed by the remainder of the text of
  [`business-rules.md#aut-083`](business-rules.md#aut-083).

## Scenario 83 — An inverted country name is reordered

- **Given** the chosen provider is `googlemap` and the Contact's country is
  `Congo, Democratic Republic of the`.
- **When** the resolve button is pressed.
- **Then** the country element of the query string is ` Democratic Republic of the Congo`, retaining
  the leading space, so the query contains two consecutive spaces before it
  ([`calculations.md`](calculations.md) §13.2, marked as a compatibility finding).

## Scenario 84 — Changing the address resets the coordinates

- **Given** the Contact of Scenario 79 with coordinates already written.
- **When** the city is changed to `Jodoigne` without writing both coordinates in the same operation.
- **Then** both coordinates become 0.0.
- **And** the resolve button reappears on the form.
- **When** the city and both coordinates are written in one operation.
- **Then** the written coordinates are kept.

## Scenario 85 — Address completion respects the minimum length

- **Given** the system parameter `google_address_autocomplete.minimal_partial_address_size` is 5 and
  a service key is configured.
- **When** the clerk types `Chau` — four characters — and the client calls the suggestion path.
- **Then** the answer carries an empty result list and no call is made to the provider.
- **When** the clerk types `Chauss` — six characters.
- **Then** the provider is called with a two-and-a-half-second timeout and the suggestions are
  returned, each carrying a formatted address and an opaque place identifier.

## Scenario 86 — The detail call is refused to a visitor and the number is guessed

- **When** the visitor calls the detail path.
- **Then** the call is refused with "You don't have access to the full autocomplete feature."
  ([`business-rules.md#aut-084`](business-rules.md#aut-084)).
- **Given** the clerk instead calls it, having typed `40 Chaussée de Namur, 1367 Ramillies`, and
  the provider answers components for the street, the city, the postal code and the country but no
  house number.
- **Then** the house number is guessed as `40`
  ([`calculations.md`](calculations.md) §13.5).
- **And** the formatted street and number is `40 Chaussée de Namur`.

---

# Data recycling

## Scenario 87 — Archiving is refused for a record type that cannot be archived

- **Given** the record type `privacy.log` has no activation flag.
- **When** the administrator creates a Recycling Model on `privacy.log` with the action `archive`.
- **Then** the save is refused with "This model doesn't manage archived records. Only deletion is
  possible." ([`business-rules.md#aut-090`](business-rules.md#aut-090)).
- **When** the action is instead `unlink`.
- **Then** the rule saves.

## Scenario 88 — A zero notification frequency is refused

- **When** the administrator sets the notification frequency to 0 and saves.
- **Then** the save is refused by the database check with "The notification frequency should be
  greater than 0" ([`business-rules.md#aut-091`](business-rules.md#aut-091)).

## Scenario 89 — The age threshold is computed from the delta

- **Given** a manual rule on `res.partner`, the time field `write_date`, the delta 6 and the unit
  months, and an empty filter.
- **And** three Contacts whose last write dates are 11 March 2026, 12 March 2026 at 03:00 and 13
  March 2026.
- **When** the nightly job runs at 12 September 2026 at 03:00.
- **Then** the threshold is 12 March 2026 at 03:00.
- **And** candidates are created for the first two Contacts and not for the third.

## Scenario 90 — The age threshold clamps at a month boundary

- **Given** the same rule with the delta 1 and the unit months.
- **When** the job runs on 31 March 2026.
- **Then** the threshold is 28 February 2026
  ([`calculations.md`](calculations.md) §9.1).

## Scenario 91 — An automatic rule disposes immediately in batches

- **Given** an automatic rule on `res.partner` with the action `archive` and 12 000 matching
  Contacts.
- **When** the job runs.
- **Then** three batches of 5 000, 5 000 and 2 000 candidates are created, each validated at once and
  each committed.
- **And** all 12 000 Contacts are archived.
- **And** no candidate remains, because validating deletes the candidate.

## Scenario 92 — A manual rule proposes and the administrator decides

- **Given** a manual rule on `res.partner` with the action `unlink` and three matching Contacts with
  the identifiers 3814, 3815 and 3816.
- **When** the job runs.
- **Then** three candidates exist, each naming its original's display name.
- **When** the administrator presses *Validate* on the candidate for 3814.
- **Then** the Contact 3814 is deleted with elevated rights and that candidate is deleted.
- **When** the administrator presses *Discard* on the candidate for 3815.
- **Then** that candidate's activation flag becomes false and the Contact 3815 still exists.
- **When** the job runs again.
- **Then** no new candidate is created for 3815, because discarded candidates are read too.
- **And** a new candidate **is** created for 3814 if that Contact still matched, which it does not
  because it was deleted.

## Scenario 93 — A candidate whose original vanished

- **Given** a candidate for the Contact 3816 and that Contact is deleted elsewhere.
- **When** the administrator opens the candidate list.
- **Then** that candidate's name shows `**Record Deleted**`.
- **When** *Validate* is pressed.
- **Then** the candidate is simply deleted and nothing else happens.

## Scenario 94 — The notification is sent on cadence and counts correctly

- **Given** a manual rule named `Contact` with two recipients, the frequency 1 and the period weeks,
  and the last notification 4 September 2026 at 03:00.
- **And** 143 candidates of that rule created on or after 5 September 2026.
- **When** the job runs at 12 September 2026 at 03:05.
- **Then** the last-notification stamp becomes 12 September 2026 at 03:05.
- **And** one message with the subject "Data to Recycle" is posted on the rule, addressed to the two
  recipients' Contacts, whose body reads "We've identified 143 records to clean with the 'Contact'
  recycling rule." followed by the line offering the link.
- **When** the job runs again on 13 September 2026.
- **Then** no notification is sent, because 12 September plus seven days is in the future.

## Scenario 95 — Archiving a rule deletes its candidates

- **Given** a manual rule with 143 candidates.
- **When** the administrator archives the rule.
- **Then** all 143 candidates are deleted, before the archive itself is written.
- **When** the rule is instead deleted.
- **Then** the candidates are deleted by cascade.

---

# Privacy handling

## Scenario 96 — An address that does not normalise is refused

- **When** the administrator opens the privacy wizard and presses *Lookup* with the address
  `not an address`.
- **Then** the operation is refused with "Invalid email address “not an address”", with the
  typographic quotation marks
  ([`business-rules.md#aut-095`](business-rules.md#aut-095)).

## Scenario 97 — The lookup finds records across record types

- **Given** the Contact `Jean-Luc Picard` with identifier 3814 and the address
  `jl.picard@example.org`.
- **And** a User with identifier 27 whose login is that address and whose Contact is 3814.
- **And** three messages written by that Contact.
- **And** two Sales Orders whose customer is that Contact.
- **And** one event registration whose own address field holds that address.
- **When** the administrator looks the person up by that name and address.
- **Then** seven lines exist: one Contact, one User, three messages, two Sales Orders and one
  registration.
- **And** the line list opens grouped by record type.
- **And** the record description reads, for an ordinary administrator,
  `Contact (1): #3814` then `User (1): #27` then the remaining lines, one per record type.

## Scenario 98 — Archiving a line is recorded and the log is created once

- **Given** the seven lines of Scenario 97 and no Privacy Log for this session.
- **When** the administrator switches the archive control on for the Contact line.
- **Then** the Contact 3814 is archived with elevated rights.
- **And** that line's execution details read `Archived Contact #3814`.
- **And** exactly one Privacy Log exists, carrying the masked name, the masked address, the
  execution details and the record description.
- **When** the administrator then archives one Sales Order line.
- **Then** the same log is updated; no second log is created.

## Scenario 99 — Deleting a line, and deleting it twice

- **Given** the line for the event registration with identifier 501.
- **When** the administrator presses *Delete* and accepts the confirmation "This operation is
  irreversible. Do you wish to proceed to the record deletion?".
- **Then** the registration is deleted with elevated rights.
- **And** the line's execution details read `Deleted Event Registration #501`.
- **And** the line is marked as deleted, so the delete button disappears.
- **When** the delete is attempted again by another route.
- **Then** it is refused with "The record is already unlinked."
  ([`business-rules.md#aut-096`](business-rules.md#aut-096)).

## Scenario 100 — The name and the address are masked in the log

- **Given** the name `Jean-Luc Picard` and the address `jean.luc.picard@enterprise.example.org`.
- **When** the log is written.
- **Then** the masked name is `J******* P*****`.
- **And** the masked address is `j***.l**.p*****@e*********.e******.org`
  ([`calculations.md`](calculations.md) §12).
- **When** the address is instead `jlpicard@gmail.com`.
- **Then** the masked address is `j*******@gmail.com`, the common domain being kept whole.

## Scenario 101 — A record the administrator cannot read still appears

- **Given** a record type whose record rule hides one record from the administrator.
- **And** that record's address field holds the address being looked up.
- **When** the lookup runs.
- **Then** a line for it exists, carrying its record type and its identifier.
- **And** the line's reference is not clickable and its name is empty.

## Scenario 102 — A Contact with posted ledger items cannot be deleted

- **Given** the Contact 3814 is the counterparty of two posted Journal Items.
- **When** the administrator presses *Delete* on that Contact's line.
- **Then** the deletion is refused by the platform's referential protection, which names the
  referencing records.
- **And** the line is not marked as deleted.
- **When** the administrator instead archives the Contact.
- **Then** the archive succeeds and the two Journal Items are untouched.

## Scenario 103 — Only a settings administrator reaches the wizard

- **When** the clerk opens a Contact's action menu.
- **Then** the *Privacy Lookup* entry is not offered.
- **When** the clerk calls the wizard directly.
- **Then** the call is refused
  ([`business-rules.md#aut-098`](business-rules.md#aut-098)).

---

# Attachments stored outside

## Scenario 104 — Configuring the first provider verifies it

- **Given** no provider is configured.
- **When** the administrator chooses `azure`, fills the account name `examplestore`, the container
  `attachments`, the tenant, the client identifier and the client secret, sets the minimum file size
  to 12.5 megabytes and saves.
- **Then** the stored minimum is 12 500 000 bytes.
- **And** a probe blob named `0/` followed by the current instant and `.txt` is uploaded and then
  downloaded through freshly signed addresses, each with a five-second timeout.
- **And** the settings are saved.
- **And** reopening the screen shows 12.5 megabytes again.

## Scenario 105 — A refused probe blocks the save

- **Given** the same input but a configuration whose signature the provider rejects for uploading.
- **When** the administrator saves.
- **Then** the save is refused with "The connection string is not allowed to upload blobs to the
  container." followed by a line break and the provider's answer body
  ([`business-rules.md#aut-103`](business-rules.md#aut-103)).
- **And** the system parameter `cloud_storage_provider` keeps its previous value.

## Scenario 106 — The second provider also writes the cross-origin rules

- **Given** the administrator chooses `google`, fills the bucket `example-bucket` and uploads a
  service account key.
- **When** the administrator saves and the upload and download probes pass but the cross-origin
  write is rejected.
- **Then** the save is refused with "The account info is not allowed to set the bucket's CORS."
  followed by a line break and the provider's answer body
  ([`business-rules.md#aut-106`](business-rules.md#aut-106)).
- **When** all three succeed.
- **Then** the bucket's cross-origin rules allow any origin, the read and write request methods, the
  media-type and disposition response headers, and a maximum age of 300 seconds.

## Scenario 107 — An incomplete configuration cannot be enabled

- **When** the administrator chooses `azure` but leaves the client secret empty and saves.
- **Then** the save is refused with "Please configure the Cloud Storage before enabling it"
  ([`business-rules.md#aut-102`](business-rules.md#aut-102)).

## Scenario 108 — Switching provider while attachments are in use is refused

- **Given** the provider is `azure` and one attachment points at an address on that provider.
- **When** the administrator changes the provider to `google` and saves.
- **Then** the save is refused with "Some Azure attachments are in use, please migrate their cloud
  storages before disable this module"
  ([`business-rules.md#aut-100`](business-rules.md#aut-100)).

## Scenario 109 — A large attachment goes to the outside store

- **Given** the provider is `azure` with the account `examplestore` and the container `attachments`,
  and the minimum file size is 20 000 000 bytes.
- **And** the clerk attaches a file named `contract.pdf` of 25 000 000 bytes to a record type that is
  not on the never-outside list.
- **When** the client posts the upload with the outside flag set.
- **Then** an Attachment with identifier 918 is created, its media type preserved, its bytes dropped,
  its kind `cloud_storage` and its address
  `https://examplestore.blob.core.windows.net/attachments/918%2F<fresh identifier>%2Fcontract.pdf`.
- **And** the answer carries, under `upload_info`, a signed address, the method `PUT`, the expected
  status 201 and the headers holding `BlockBlob` under `x-ms-blob-type` and the media type.
- **And** the client then puts the bytes straight to the provider; they never pass through this
  system.

## Scenario 110 — A small attachment stays local

- **Given** the same configuration.
- **And** the clerk attaches a file of 1 000 000 bytes.
- **When** the upload is posted.
- **Then** the client does not ask for an outside upload, because the size is below the threshold it
  was told in the session description.
- **And** the attachment's kind is `binary` and its bytes are stored locally.

## Scenario 111 — The store is switched off while the page is open

- **Given** the clerk's page was loaded while a provider was configured, and the provider has since
  been cleared.
- **When** the client posts an upload with the outside flag set.
- **Then** the answer carries "Cloud storage configuration has been changed. Please refresh the
  page." under `error` ([`business-rules.md#aut-107`](business-rules.md#aut-107)).
- **And** no attachment is created.

## Scenario 112 — A download is answered with a cached redirection

- **Given** the Attachment 918 of Scenario 109.
- **When** the clerk requests it.
- **Then** the answer is a redirection to a freshly signed address whose lifetime is 300 seconds.
- **And** the redirection may be cached for 290 seconds
  ([`calculations.md`](calculations.md) §14.4).
- **When** the reading context asks for a download rather than a display.
- **Then** the signed address also carries the disposition override naming `contract.pdf`.

## Scenario 113 — Bringing an attachment back to local storage

- **Given** the Attachment 918.
- **When** the administrator brings it back.
- **Then** the blob is downloaded through a signed address with a ten-second timeout.
- **And** the attachment's kind becomes `binary`, its address becomes empty and its bytes are the
  downloaded ones.
- **When** the download instead answers with the status 404 and the reason `Not Found`.
- **Then** the operation is refused with "Failed to download attachment (918) from cloud: 404 - Not
  Found" ([`business-rules.md#aut-109`](business-rules.md#aut-109)).
- **And** the attachment keeps its outside address and its kind.

---

# Delegated mail authorisation

## Scenario 114 — Choosing the first provider fills the connection

- **Given** the administrator creates an outgoing Mail Server named `Sales mailbox`.
- **When** the authentication kind is set to `gmail`.
- **Then** the host becomes `smtp.gmail.com`, the encryption becomes `starttls` and the port becomes
  587.
- **When** the user name is set to `sales@example.test`.
- **Then** the sender filter becomes `sales@example.test`.
- **And** the authentication description reads "Connect your Gmail account with the OAuth
  Authentication process.  \nBy default, only a user with a matching email address will be able to
  use this server. To extend its use, you should set a \"mail.default.from\" system parameter."

## Scenario 115 — A password on a delegated server is refused

- **Given** the server of Scenario 114.
- **When** a password is filled in and the server is saved.
- **Then** the save is refused with "Please leave the password field empty for Gmail mail server
  “Sales mailbox”. The OAuth process does not require it"
  ([`business-rules.md#aut-110`](business-rules.md#aut-110)).
- **And** for the second provider the same situation is refused with "Please leave the password field
  empty for Outlook mail server “Sales mailbox”. The OAuth process does not require it"
  ([`business-rules.md#aut-113`](business-rules.md#aut-113)).

## Scenario 116 — The wrong encryption and a missing user name are refused

- **Given** the server of Scenario 114 with the encryption forced to `ssl`.
- **When** it is saved.
- **Then** the save is refused with "Incorrect Connection Security for Gmail mail server “Sales
  mailbox”. Please set it to \"TLS (STARTTLS)\"."
  ([`business-rules.md#aut-111`](business-rules.md#aut-111)).
- **When** the encryption is `starttls` but the user name is empty.
- **Then** the save is refused with "Please fill the \"Username\" field with your Gmail username
  (your email address). This should be the same account as the one used for the Gmail
  OAuthentication Token." ([`business-rules.md#aut-112`](business-rules.md#aut-112)).

## Scenario 117 — An incoming server must be marked secure

- **Given** the administrator creates an Incoming Mail Server named `Support inbox` with the kind
  `gmail`.
- **Then** the host becomes `imap.gmail.com`, the secure flag is set and the port becomes 993.
- **When** the secure flag is cleared and the server is saved.
- **Then** the save is refused with "SSL is required for server “Support inbox”."
  ([`business-rules.md#aut-116`](business-rules.md#aut-116)).

## Scenario 118 — Only the administrator starts the consent

- **When** the clerk presses the connect button on a server of the first kind.
- **Then** the operation is refused with "Only the administrator can link a Gmail mail server."
  ([`business-rules.md#aut-118`](business-rules.md#aut-118)).
- **And** for the second kind the text is "Only the administrator can link an Outlook mail server."
  ([`business-rules.md#aut-119`](business-rules.md#aut-119)).

## Scenario 119 — An unusable mailbox address is refused

- **Given** the administrator presses the connect button on a server whose user name is `not valid`.
- **Then** the operation is refused with "Please enter a valid email address."
  ([`business-rules.md#aut-120`](business-rules.md#aut-120)).

## Scenario 120 — Without credentials and without the relay the connect is refused

- **Given** the system parameters `google_gmail_client_id` and `google_gmail_client_secret` are
  empty and the running edition may not use the relay.
- **When** the administrator presses the connect button.
- **Then** the operation is refused with "Please configure your Gmail credentials."
  ([`business-rules.md#aut-121`](business-rules.md#aut-121)).
- **And** for the second provider the text is "Please configure your Outlook credentials."

## Scenario 121 — A returned protection token that does not match is forbidden

- **Given** the administrator has started the consent for the outgoing Mail Server with identifier
  55.
- **When** the provider sends the browser back to `/google_gmail/confirm` with a valid code but the
  state carries the protection token `tampered`.
- **Then** the answer is the forbidden status.
- **And** an error line naming a wrong protection token is logged.
- **And** the server's tokens are unchanged
  ([`business-rules.md#aut-125`](business-rules.md#aut-125)).

## Scenario 122 — A consented address that does not match is reported

- **Given** the outgoing Mail Server 55 has the user name `sales@example.test` and an owner.
- **When** the consent returns and the provider reports the verified address
  `other@example.test`.
- **Then** the error page is rendered with "Oops, you're creating an authorization to send from
  other@example.test but your address is sales@example.test. Make sure your addresses match!" and a
  link back to the record
  ([`business-rules.md#aut-126`](business-rules.md#aut-126)).
- **And** the server's tokens are unchanged.

## Scenario 123 — A successful consent writes the record and redirects

- **Given** the same server, owned by nobody, and the reader is a settings administrator.
- **When** the consent returns with a valid code and the exchange answers with the refresh token
  `1//REFRESH`, the access token `ya29.EXAMPLE` and a lifetime of 3 599 seconds at the instant
  1 789 000 000.
- **Then** the server's activation flag is true, its access token is `ya29.EXAMPLE`, its expiry is
  1 789 003 599 and its refresh token is `1//REFRESH`.
- **And** the browser is sent to the server's own form.
- **When** the server is instead the reader's personal outgoing server.
- **Then** the browser is sent to the reader's own preferences.

## Scenario 124 — The access token is renewed inside the threshold

- **Given** the server of Scenario 123 with the stored expiry 1 789 000 008.
- **When** a send happens at the instant 1 789 000 000.
- **Then** the token is renewed, because 1 789 000 008 − 10 is below 1 789 000 000
  ([`calculations.md`](calculations.md) §15.2).
- **And** the authentication string is `user=sales@example.test`, the control character one,
  `auth=Bearer ` and the **new** access token, then two control characters, the whole encoded.
- **When** the same happens on the second provider.
- **Then** the refresh token stored on the server is **replaced** by the new one the exchange
  returned.
- **When** the second provider's server has no refresh token at all.
- **Then** the send fails with "Please connect with your Outlook account before using it."
  ([`business-rules.md#aut-127`](business-rules.md#aut-127)).

---

# Onboarding and tours

## Scenario 125 — A step without an opening operation cannot join a panel

- **Given** an Onboarding Step titled `Configure your bank` whose opening-operation name is empty.
- **When** the administrator links it to a panel.
- **Then** the link is refused with "An \"Opening Action\" is required for the following steps to be
  linked to an onboarding panel: ['Configure your bank']"
  ([`business-rules.md#aut-130`](business-rules.md#aut-130)).

## Scenario 126 — Two panels cannot share a route segment

- **Given** a panel whose route segment is `accounting`.
- **When** a second panel is saved with the same segment.
- **Then** the save is refused with "Onboarding alias must be unique."
  ([`business-rules.md#aut-131`](business-rules.md#aut-131)).

## Scenario 127 — Completing the last step completes the panel once

- **Given** a panel with three steps and a progress record for Main Company in which two steps are
  done.
- **When** the clerk completes the third step.
- **Then** that step's progress becomes `just_done`.
- **And** the panel's progress becomes `just_done`.
- **And** the closing message is shown once.
- **When** the panel is rendered again.
- **Then** every `just_done` state has become `done` and the message is not shown again.

## Scenario 128 — Progress is per company

- **Given** the same panel and two companies.
- **When** the clerk, working in Main Company, completes all three steps.
- **Then** Main Company's progress is complete.
- **And** Second Company's progress is untouched, and the panel is still offered there.

## Scenario 129 — Two tours cannot share a name

- **Given** a Tour named `sale_quote_tour`.
- **When** a second tour is saved with the same name.
- **Then** the save is refused with "A tour already exists with this name . Tour's name must be
  unique!", including the space before the full stop
  ([`business-rules.md#aut-132`](business-rules.md#aut-132)).

## Scenario 130 — The sharing address and the export

- **Given** the tour `sale_quote_tour` with the starting address `/odoo/sales` and two steps.
- **Then** its sharing address is `https://example.test/odoo?tour=sale_quote_tour`.
- **When** the administrator runs the export.
- **Then** one attachment named `sale_quote_tour.js` is created on the tour, holding the declaration
  of the tour with its starting address and its two steps, each carrying its trigger, its
  instruction and its tooltip position under `tooltipPosition`, with the content omitted when empty.
- **And** the browser is sent to that attachment's download address.

## Scenario 131 — A tour is offered once and then consumed

- **Given** the administrator has the tour switch on, no package with demonstration data is
  installed, and two non-custom tours exist that the administrator has not consumed, ordered
  `first_tour` then `second_tour`.
- **When** the client starts a session.
- **Then** the session description offers `first_tour`.
- **When** the client reports that `first_tour` is consumed.
- **Then** the administrator is linked to `first_tour` and the answer offers `second_tour`.
- **When** both are consumed.
- **Then** the session description offers nothing.
- **When** the administrator switches the offer off.
- **Then** the session description reports the switch as off and offers nothing.

---

# Translations, documentation, remote calls, sparse storage

## Scenario 132 — A platform link is built with quoting

- **Given** the system parameter `transifex.project_url` holds `https://app.transifex.com/odoo`, the
  project map holds the package `sale` under the project `odoo-19`, and the language `fr_BE` has the
  standard code `fr`.
- **And** a cached term whose source is `Sales Order`, whose package is `sale` and whose language is
  `fr_BE`.
- **Then** the platform link is
  `https://app.transifex.com/odoo/odoo-19/translate/#fr/sale/42?q=text%3A'Sales+Order'`
  ([`calculations.md`](calculations.md) §16.2).
- **When** the source is instead `Invoice`.
- **Then** the link ends `/42?q=text%3AInvoice`, without quotation marks.

## Scenario 133 — No link is built for the base language or an unmapped package

- **Given** a cached term whose language is the base language.
- **Then** its platform link is empty.
- **Given** a cached term whose package is not in the project map.
- **Then** its platform link is empty.

## Scenario 134 — The term cache is loaded once and reloaded weekly

- **Given** the cached-term table is empty and two languages besides the base one are installed.
- **When** the administrator opens the cached-term list.
- **Then** the table is locked exclusively, one row is created per source term, package and
  language, and the list opens.
- **When** a second session tries to load at the same moment.
- **Then** it takes no lock, creates nothing and returns the value false
  ([`business-rules.md#aut-141`](business-rules.md#aut-141)).
- **When** the weekly job runs.
- **Then** the table is emptied and loaded again.

## Scenario 135 — The documentation page is group-restricted

- **When** the clerk requests `/doc`.
- **Then** the request is refused with "This page is only accessible to Technical Documentation
  users." ([`business-rules.md#aut-133`](business-rules.md#aut-133)).
- **When** the administrator requests it.
- **Then** the page is served with the framing prohibition set to deny.
- **When** a caller presents a bearer credential to `/doc-bearer/index.json`.
- **Then** the index is served without the group check.

## Scenario 136 — The documentation index is cached as an attachment

- **Given** the registry sequence is 42 and the administrator's groups and reading language are
  fixed.
- **When** the administrator requests `/doc/index.json` for the first time.
- **Then** a key is computed, one attachment named `odoo-doc-index-42-<key>.json` is created holding
  the index, and it is streamed.
- **When** the same request is made again with that key.
- **Then** the answer is the not-modified status.
- **When** the request asks for no cache.
- **Then** the index is rebuilt and answered with the directive `no-store` and a disposition naming
  `odoo-doc-index.json`.
- **When** the registry sequence becomes 43 and the housekeeping sweep runs.
- **Then** the attachment for sequence 42 is deleted.

## Scenario 137 — The typed call refuses what it should

- **When** a caller with a bearer credential submits to `/json/2/not.a.model/read`.
- **Then** the answer is the not-found status with the text "the model 'not.a.model' does not exist"
  ([`business-rules.md#aut-134`](business-rules.md#aut-134)).
- **When** the caller submits to `/json/2/res.partner/_private_helper`.
- **Then** the answer is the not-found status
  ([`business-rules.md#aut-135`](business-rules.md#aut-135)).
- **When** the caller submits identifiers to an operation declared at the level of the record type.
- **Then** the answer is the unprocessable status with the text "cannot call res.partner.search with
  ids" ([`business-rules.md#aut-136`](business-rules.md#aut-136)).
- **When** the caller submits to `/json/2/anything`.
- **Then** the answer is the not-found status with the text "Did you mean POST
  /json/2/<model>/<method>?" ([`business-rules.md#aut-138`](business-rules.md#aut-138)).

## Scenario 138 — A typed call succeeds and reduces a record set

- **Given** the Contacts 3814 and 3815 exist.
- **When** a caller with a bearer credential submits to `/json/2/res.partner/read` the body carrying
  the identifiers 3814 and 3815 under `ids` and the field list `["name"]`.
- **Then** the answer carries the two mappings of identifier and name.
- **When** the caller instead invokes an operation that returns a record set.
- **Then** the answer carries the identifiers only, not the records.

## Scenario 139 — The two call endpoints report faults differently

- **Given** an operation that raises an access refusal with the text `Not allowed`.
- **When** it is invoked over `/xmlrpc/2/object`.
- **Then** the fault code is the whole number 4 and the fault text is `Not allowed`.
- **When** it is invoked over `/xmlrpc/object`.
- **Then** the fault code is the text `warning -- AccessError` followed by two line breaks and
  `Not allowed`, and the fault value is empty.
- **When** an operation answers with a text containing the control character 7.
- **Then** that character is removed from the answer, while tabulation, line feed and the
  carriage-return character are kept.

## Scenario 140 — Sparse storage stores, clears and refuses renaming

- **Given** the reference record type with the serialisation field `data` and three sparse fields
  `boolean`, `integer` and `partner`.
- **When** a record is created with true, 42 and the Contact 3814.
- **Then** the stored mapping holds three keys.
- **When** `integer` is written with 0.
- **Then** the stored mapping holds two keys and reading `integer` yields 0.
- **When** the administrator tries to rename the field `integer`.
- **Then** the write is refused with "Renaming sparse field \"integer\" is not allowed"
  ([`business-rules.md#aut-140`](business-rules.md#aut-140)).
- **When** the administrator tries to point `integer` at a different serialisation field.
- **Then** the write is refused with "Changing the storing system for field \"integer\" is not
  allowed." ([`business-rules.md#aut-139`](business-rules.md#aut-139)).

---

# Several companies

## Scenario 141 — A company-scoped metered account is invisible elsewhere

- **Given** an account for `reveal` whose company list holds Main Company only.
- **When** the clerk whose allowed companies are Second Company only searches the accounts.
- **Then** that account is not found
  ([`business-rules.md#aut-079`](business-rules.md#aut-079)).
- **When** the same clerk asks a feature for the account for `reveal`.
- **Then** a **new** account with an empty company list is created for them, because the scoped one
  is invisible.

## Scenario 142 — A recycling candidate carries its original's company

- **Given** a manual rule on `res.partner` and two matching Contacts, one belonging to Main Company
  and one to Second Company.
- **When** the job runs.
- **Then** two candidates exist, whose company fields are Main Company and Second Company
  respectively, derived from their originals.

## Scenario 143 — A privacy lookup spans companies

- **Given** the Contact 3814 and one Sales Order of Main Company and one of Second Company, both
  naming that Contact.
- **When** the administrator, whose allowed companies are both, looks the person up.
- **Then** both Sales Order lines appear.
- **When** an administrator whose allowed companies are Main Company only looks the same person up.
- **Then** both lines still appear, because the query is direct, but the line for Second Company's
  order has no clickable reference and no name.

## Scenario 144 — An automation rule fires in whichever company the record belongs to

- **Given** a rule watching `sale.order` with the trigger `on_create` and one code action that
  writes the order's company name into the order's internal note.
- **When** the clerk creates an order in Second Company.
- **Then** the note reads `Second Company`.
- **And** the rule itself belongs to no company, so one rule serves both.

## Scenario 145 — An import into one company does not leak into the other

- **Given** the clerk's allowed companies are Second Company only.
- **When** the clerk imports two Contacts without a company column.
- **Then** both Contacts are created with Second Company as their company, following the ordinary
  creation rules of the contacts domain.
- **And** the Data Import Column Mappings created are not scoped to a company, so the next import in
  Main Company benefits from the same remembered headings.

---

# How these scenarios map to the rest of the folder

| Scenario range | Rules asserted | Formulas asserted | Procedures asserted |
|---|---|---|---|
| 1 – 30 | `AUT-001` to `AUT-019` | §1, §2, §3 | §1 to §8 |
| 31 – 56 | `AUT-020` to `AUT-039` | §5 | §9, §10 |
| 57 – 66 | `AUT-040` to `AUT-063` | §6 | §11, §12, §13 |
| 67 – 78 | `AUT-070` to `AUT-079` | §4 | §14, §15 |
| 79 – 86 | `AUT-080` to `AUT-086` | §13 | §16, §17, §18 |
| 87 – 95 | `AUT-090` to `AUT-094` | §9 | §19 |
| 96 – 103 | `AUT-095` to `AUT-099` | §11, §12 | §20 |
| 104 – 113 | `AUT-100` to `AUT-109`, `AUT-142` | §14 | §21 |
| 114 – 124 | `AUT-110` to `AUT-129` | §15 | §22 |
| 125 – 131 | `AUT-130` to `AUT-132`, `AUT-143` | §17 | §23, §24 |
| 132 – 140 | `AUT-133` to `AUT-141` | §7, §8, §16 | §25 |
| 141 – 145 | `AUT-038`, `AUT-079`, `AUT-094`, `AUT-098` | §9, §11 | §9, §14, §19, §20 |

The rule identifiers are defined in [`business-rules.md`](business-rules.md), the formula sections in
[`calculations.md`](calculations.md) and the procedure sections in
[`workflows.md`](workflows.md).
