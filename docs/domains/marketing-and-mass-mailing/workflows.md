# Marketing workflows

Every operational procedure of the domain, end to end: actors, preconditions, numbered steps, branches, the records written at each step with their field values, the notifications produced and the postconditions. The state machines of the domain are specified in [state-machines.md](state-machines.md); section 1 below only points at them.

## 1. State machines

The three state fields of this domain — the state of a Mass Mailing, the delivery status of a
Mailing Trace and the share status of a Marketing Card — are specified completely, with every
state, every transition, every guard, every refusal message and a diagram per machine, in
[state-machines.md](state-machines.md). The procedures below name the states they move through and
link to that document rather than repeating the tables.

| Machine | Field | States | Specified in |
|---|---|---|---|
| Mailing life cycle | `state` on Mass Mailing | `draft`, `in_queue`, `sending`, `done` | [state-machines.md](state-machines.md#1-the-mailing-life-cycle) |
| Delivery status | `trace_status` on Mailing Trace | `outgoing`, `process`, `pending`, `sent`, `open`, `reply`, `bounce`, `error`, `cancel` | [state-machines.md](state-machines.md#2-the-delivery-status-of-a-mailing-trace) |
| Card sharing | `share_status` on Marketing Card | empty, `visited`, `shared` | [state-machines.md](state-machines.md#3-the-share-status-of-a-marketing-card) |
| Reported mailing state | `state` on Mailing Trace Report | `draft`, `test`, `done` | [state-machines.md](state-machines.md#4-the-reported-mailing-state) |
| Subscription opt-out | `opt_out` on Mailing Subscription | opted in, opted out | [state-machines.md](state-machines.md#5-the-opt-out-condition-of-a-mailing-subscription) |
| Comparison-test completion | `ab_testing_completed` on Campaign | running, completed | [state-machines.md](state-machines.md#6-the-completion-condition-of-a-comparison-test-campaign) |

---

## 2. Design a mailing

**Actor.** Marketing User. **Precondition.** None.

1. The user creates a mailing. Defaults are applied: responsible = the current user; type = `mail`; recipient entity = Mailing List; state = `draft`; schedule type = `now`; exclusion list = on; comparison-test percentage = 10; comparison-test moment = now plus one day; comparison-test criterion = highest open rate; mail server = the configured dedicated server when the dedicated-server setting is on.
2. When the recipient entity is Mailing List and exactly one Mailing List exists in the system, that list is pre-selected.
3. When the form was opened from the calendar view on a day strictly in the future, the schedule type becomes `scheduled` and the schedule date that day.
4. The user types the subject and optionally the preview sentence. Both accept placeholders evaluated against the recipient record.
5. The user chooses a design: an empty body, the basic body, the default themed body, one of the eleven shipped themes, or one of their own favorite designs. Choosing a design fills `body_arch`; the gallery of favorite designs lists every mailing flagged as favorite whose body is not visually empty, ordered by favorite moment descending, and then re-ordered so that designs with no responsible come first, then the current user's designs, then the others. Archived favorites are included.
6. The user edits the body with the building blocks. The available blocks include headers, covers, text blocks, image-and-text blocks, media lists, product lists, event blocks, call-to-action blocks, badges, quotations, ratings, separators, features grids, masonry blocks, company-team blocks, reference blocks, promotional-code blocks, and footers that contain the company's social-network links, the unsubscribe placeholder `/unsubscribe_from_list` and the view-in-browser placeholder `/view`.
7. On every save, inline images embedded directly in the body are converted into stored files and replaced by addresses (see [calculations.md](calculations.md) section 12), including images inside outgoing-mail compatibility comments and background images that have to be cropped for one particular mail client.
8. The user chooses the audience. Either the recipient entity stays Mailing List, in which case they pick one or more lists and the condition becomes `list_ids IN (chosen lists)`; or they pick another mailable entity, in which case they edit the condition directly or load a saved filter.
9. The user optionally opens the settings page to change the sender address, the answer mode and address, the attachments, the campaign, the responsible person, the archive-keeping flag, the dedicated server and the exclusion-list flag.
10. The user saves. Postcondition: a mailing in state `draft` with a unique name derived from the subject, a Campaign Source of that same name, and a medium.

**Records written.** One Mass Mailing; one Campaign Source; zero or more Attachment records re-owned by the mailing; one Campaign when comparison testing was enabled without a campaign.

---

## 3. Send a test

**Actor.** Marketing User. **Precondition.** A saved mailing.

### 3.1 Electronic mail

1. The user opens the test assistant. The address box is pre-filled with the last value this user used in a test during the previous ten hours, otherwise with their own formatted address.
2. The user types one or more addresses, one per line, and confirms.
3. Each line is split into addresses. Lines that yield no address are collected as invalid candidates.
4. The first record of the mailing's recipient entity is fetched. When one exists, the body, the preview and the subject are rendered against it, in order that placeholder errors surface. When none exists, the raw body, preview and subject are used.
5. The preview sentence is prepended to the body inside a hidden block of zero size and zero opacity.
6. The subject becomes `[TEST] <rendered subject>`.
7. Local links in the body are converted to absolute addresses before shortening.
8. For each valid address one outgoing mail is created with elevated rights: sender = the mailing's sender, answer address = the mailing's answer address, destination = that address, subject as above, body wrapped in the marketing layout with the marketing stylesheet, not a notification, linked to the mailing, the mailing's attachments, automatic deletion off, the mailing's server, and the first record as the related record.
9. All of them are sent immediately with the test flag on, which suppresses the replacement of the unsubscribe placeholder.
10. A note is logged on the mailing listing, for each address: *"Test mailing successfully sent to <address>"* when the message reached the relay, or *"Test mailing could not be sent to <address>:"* followed by a line break and the failure reason when it did not; plus *"Mailing addresses incorrect: <comma-separated list>"* when there were invalid candidates.
11. The outgoing mail records are deleted afterwards, because they were created with automatic deletion off.

**Postcondition.** No trace is created; the mailing's measurement is untouched; the mailing stays in `draft`.

### 3.2 Text message

1. The user opens the text-message test assistant, pre-filled in the same way with their own sanitised number as fallback.
2. Each line is sanitised. Valid numbers are kept; invalid ones are collected.
3. The first record of the recipient entity is fetched and the plain-text body is rendered against it when one exists.
4. For each valid number an outgoing text message is created with elevated rights, in state `outgoing`, with a fresh provider key.
5. When the mailing includes the opt-out link, a trace flagged as a test trace is created for each number with the mailing, the record, a fresh three-character code, the number, a provider tracker and the type `sms`; and the body sent to that number is extended with a line break and *"STOP SMS: <base>/sms/<mailing key>/<code>"*, in order that the opt-out flow itself can be tested.
6. The batch is handed to the sending service with the delivery-report address `<base>/sms/status`.
7. A note is logged on the mailing listing *"Test SMS successfully sent to <number>"* for each accepted number, *"Test SMS could not be sent to <number>: <explanation>"* for each rejected one, and *"Test SMS skipped those numbers as they appear invalid: <comma-separated list>"* when numbers were unusable.

---

## 4. Send immediately

**Actor.** Marketing User. **Preconditions.** State `draft`; a body; a recipient condition.

1. The user presses Send and confirms the warning *"Once you send these emails, they'll be making a grand entrance in all the inboxes, creating quite the buzz!"* (title *"Ready to unleash emails?"*, confirmation label *"Send to all"*).
2. `schedule_type` is written to `now`, which empties `schedule_date`.
3. With a marketing card campaign, the card synchronisation count is checked; a non-zero count refuses the operation with *"You should update all the cards for %(mailing)s before scheduling a mailing."*
4. The state becomes `in_queue`.
5. The queue job is asked to wake once per mailing, at the mailing's schedule date or, when empty, at the current moment.

**Postcondition.** State `in_queue`, `calendar_date` = `next_departure`, `next_departure` = now.

---

## 5. Schedule for later

**Actor.** Marketing User. **Precondition.** State `draft`.

1. The user presses Schedule.
2. When the mailing already carries a schedule date strictly in the future, steps 3 to 5 of section 4 run directly.
3. Otherwise the schedule assistant opens.
4. The user chooses a moment and confirms: the mailing is written with `schedule_type = "scheduled"` and that moment, and is then queued as in section 4.

**Postcondition.** State `in_queue`; `calendar_date` equals the chosen moment; the queue job has a wake request for that moment. When the chosen moment is in the past, `next_departure` is the current moment, `next_departure_is_past` is true, and the form shows *"This mailing will be sent as soon as possible."* with a refresh button.

---

## 6. Queue processing

**Actor.** Scheduled job runner, acting as the system user. **Trigger.** The job `Mail Marketing: Process queue`, whose default interval is one day and which is woken on demand whenever a mailing is queued.

1. Select every mailing whose state is `in_queue` or `sending` and whose schedule date is empty or already past.
2. Report the number found as the remaining work.
3. For each mailing, in order:
   1. Determine the acting user: the mailing's responsible, else its last writer, else the current user. Re-read the mailing in that user's language, time zone and company context.
   2. Compute the remaining recipients (section 7.2).
   3. When at least one remains: write the state `sending` and run the sending pass (section 7).
   4. When none remains: write state `done`, `sent_date` = now, and `kpi_mail_required` = true when the mailing had no previous sent date.
   5. Report one unit of progress; the runner commits between mailings.
4. After the loop, when the setting `mass_mailing.mass_mailing_reports` is on, select every mailing with `kpi_mail_required` true, state `done`, and a sent date between five days ago and one day ago, and send their statistics messages (section 20).

**Interruption.** The runner may stop the loop when its time budget is exhausted. Because the state is `sending` and the traces of already-processed recipients exist, the next run resumes exactly where it stopped.

---

## 7. The sending algorithm

This is the core algorithm of the domain. It runs for one mailing, either from the queue job or from a direct send.

### 7.1 Audience

```
condition = the stored recipient condition, parsed;
            an unparsable condition yields the impossible condition id IN ()
with a marketing card campaign:
            condition = condition AND id IN (keys that already have a card)
audience   = keys of the recipient entity matching condition
```

### 7.2 Sample and remaining recipients

When comparison testing is enabled **and** this mailing is not the campaign winner, the audience is reduced to a random sample:

```formula
audience_size  = count of the audience
sample_size    = max(floor(audience_size × ab_testing_pc ÷ 100), 1)
                 when audience_size > 0, else 0
already_mailed = the set of recipient keys already traced for the campaign
remaining      = audience − already_mailed
sample_size    = size(remaining)
                 whenever sample_size > size(remaining),
                 and whenever remaining is not empty while sample_size = 0
audience       = a random sample of sample_size keys drawn from remaining,
                 the remaining set being sorted before the draw
```

When comparison testing is disabled, or when this mailing is the campaign winner, the audience is not sampled.

Then the already-traced keys are removed:

| Case | Delivery records that disqualify a recipient |
|---|---|
| Comparison testing enabled **and** this mailing is the campaign winner | recipient entity name equals this mailing's recipient entity **and** the mailing is any of the comparison-test versions of the campaign |
| Every other case | recipient entity name equals this mailing's recipient entity **and** the mailing is this mailing **and** the recipient key is in the audience |

```formula
remaining_recipients = audience − { recipient keys of the disqualifying delivery records }
```

The winner rule is what makes the final send address only the people no version has reached.

### 7.3 Refusal on an empty audience

When the remaining list is empty and the caller did not supply explicit keys, the operation is refused with *"There are no recipients selected."* The queue job never hits this refusal because it checks the remaining list first and closes the mailing instead.

### 7.4 Composer preparation

A composer is created in mass mode with:

| Value | Source |
|---|---|
| automatic deletion | the opposite of `keep_archives` |
| keep the thread message when deleting | true when the answer mode is `update` |
| author | the mailing's responsible contact when the current user is the system robot, otherwise the current user's contact |
| attachments | the mailing's attachments |
| body | the rich-text body with the preview sentence prepended in a hidden zero-size block |
| subject | the mailing's subject |
| sender | the mailing's sender address |
| answer address | the mailing's answer address, only when the mode is `new` |
| force a new thread for answers | true when the mode is `new` |
| server | the mailing's server |
| lists | the mailing's lists |
| mailing | this mailing |
| recipient entity | `mailing_model_real` |
| template | none |
| exclusion list | the mailing's flag |
| active keys | the remaining recipients |
| link-tracker values | campaign, source, medium and mailing of this mailing |

### 7.5 Batch loop

```
batch_size = system parameter "mail.batch_size", else 50; a value of zero is replaced by 50
for each slice of batch_size keys of the remaining recipients:
    values = prepare one outgoing message per key of the slice     (7.6)
    values = apply the exclusion checks                            (7.7)
    values = drop the cancelled ones, creating their traces        (7.8)
    create the outgoing messages with their traces
    when the messages must be sent at once: send the slice now
    otherwise: report progress, commit, and drop every cached record
```

Because the loop commits between slices, an interrupted run never re-sends a slice that was already created: those recipients now have traces and are removed from the remaining list on the next pass.

### 7.6 Per-recipient preparation

For each key: the subject, the body and the answer address are rendered against that record; the recipient address is resolved from the record; the message key is generated in advance and copied into the references, in order that answers and bounces can be matched even if the relay rewrites it; the body is post-processed, which converts local links to absolute ones and then shortens every link that is not in the skip list `/unsubscribe_from_list`, `/view`, `/cards`, creating or reusing a Link Tracker per distinct (address, campaign, medium, source, label) combination with the mailing attached; the body is wrapped in the marketing layout with the marketing stylesheet.

### 7.7 Exclusion checks, in order

For each prepared message, the **first** matching rule applies and no other:

| Order | Condition | Resulting state | Failure type |
|---|---|---|---|
| 1 | The recipient record is in the blocked set | `cancel` | `mail_bl` |
| 2 | No destination address at all | `cancel` when archives are not kept, `exception` when they are | `mail_email_missing` |
| 3 | No destination address that normalises | same as above | `mail_email_invalid` |
| 4 | Every normalised destination is in the opted-out set | `cancel` | `mail_optout` |
| 5 | Every normalised destination is in the already-contacted set | `cancel` | `mail_dup` |
| 6 | Every normalised destination already received, inside this same run, a message with the same subject, the same body and the same attachments | `cancel` | `mail_dup` |
| 7 | none of the above | kept | — |

A message kept by rule 7 records its normalised destinations in the run's sent map, together with its subject and body, which is what rule 6 later compares against. After the whole batch, the run's sent addresses are appended to the already-contacted set.

For marketing sends, rules 2 and 3 always produce `cancel`, because automatic deletion is on and the thread log is not kept in the ordinary case; when the mailing keeps archives, they produce `exception` instead, which becomes the trace status `error`.

**The blocked set.** Empty when the exclusion-list flag is off. Otherwise every active blocked address is read; when the recipient entity carries a normalised address of its own, the records whose normalised address is in that set are blocked; otherwise the records at least one of whose resolved destinations is in that set are blocked.

**The opted-out set.** Supplied by the recipient entity when it publishes one. For the Mailing List entity it is: the normalised addresses of the contacts that are opted out of at least one of the mailing's lists **and** opted in to none of them. Therefore, when two contacts share an address and one of them is still opted in, the message is sent.

**The already-contacted set.** See section 7.9. With a marketing card campaign this set is forced empty.

### 7.8 Cancelled messages

Prepared values whose state is `cancel` are removed from the batch, and their traces are created directly with elevated rights, carrying the status `cancel` and the failure type. Therefore a cancelled recipient produces a trace but no outgoing message, and appears in the *cancelled* counter of the mailing.

### 7.9 The already-contacted set

| Comparison testing | Addresses collected |
|---|---|
| enabled | the addresses of the delivery records whose campaign is this mailing's campaign |
| disabled | the addresses of the delivery records whose mailing is this mailing and whose recipient entity is this mailing's entity |

In both cases only delivery records that still join to an existing record of the recipient entity and that carry a non-empty address are collected.

This is the rule that lets several versions of one comparison test share an audience without contacting anybody twice, and that makes a resumed send skip everybody already reached.

### 7.10 Closing

After the loop the mailing is written with state `done`, `sent_date` = now and `kpi_mail_required` = true when there was no previous sent date. Outside automated tests the transaction is committed immediately afterwards, in order that the state survives a later failure.

### 7.11 Postconditions

- One Mailing Trace per addressed recipient record, in status `outgoing`, `cancel` or `error`.
- One outgoing mail per kept recipient, with its tracking image, its per-recipient unsubscribe and view links and its four unsubscribe headers.
- One Link Tracker per distinct link of the body, with the campaign, source, medium and mailing attached; the click counter starts at zero.
- The mailing in state `done`.

---

## 8. Import contacts by pasting addresses

**Actor.** Marketing User. **Precondition.** None.

1. The user opens the import assistant, optionally from a list, in which case that list is pre-selected.
2. The user pastes lines and confirms.
3. All lines are joined with commas and split into (name, address) pairs.
4. When no pair is found, nothing is created and the warning *"No valid email address found."* is shown, and the assistant closes.
5. When more than 5000 pairs are found, nothing is created and the warning *"You have to much emails, please upload a file."* is shown, followed by the opening of the general file-import assistant for Mailing Contact.
6. All addresses are lower-cased. The contacts that already exist with one of those normalised addresses **and** are already members of at least one of the chosen lists are loaded.
7. Each pair is processed in order. Addresses already seen with a non-empty name are skipped, therefore the first non-empty name wins for a repeated address. For an address that matches an existing contact and whose chosen lists are not already a subset of that contact's lists, the chosen lists are added to that contact. For an address that matches no existing contact, a creation entry is prepared with the name and one subscription per chosen list.
8. When no creation entry remains, the warning *"No contacts were imported. All email addresses are already in the mailing list."* is shown.
9. Otherwise the contacts are created, with the screen's default list context cleared so that it cannot add lists twice.
10. A success message is shown: *"Contacts successfully imported. Number of contacts imported: <n>"* or, when some lines were duplicates, *"Contacts successfully imported. Number of contacts imported: <n>. Number of duplicates ignored: <d>"* where `d` is the number of parsed pairs minus the number of created contacts. The list of newly created contacts then opens under the title *"New contacts imported"*.

**Worked example.** Pasting eleven lines of which four are unparsable, with `alice@example.com`, `bob@example.com`, `"Bob" <bob@EXAMPLE.com>`, `"Test" <bob@example.com>`, `already_exists_list_1@example.com`, `already_exists_list_2@example.com` and `"Test" <already_exists_list_1_and_2@example.com>`, into the first and third lists, where `already_exists_list_1@example.com` is already in the first list, `already_exists_list_2@example.com` only in the second and `already_exists_list_1_and_2@example.com` in both first and second: five parsed pairs become contacts of the third list, the first list gains `alice@example.com`, `Bob <bob@example.com>` and `already_exists_list_2@example.com`, the existing contact of `already_exists_list_1@example.com` gains the third list rather than being duplicated, and the second list is untouched because the screen's default lists were ignored.

---

## 9. Add selected contacts to a list

**Actor.** Marketing User. **Precondition.** One or more Mailing Contacts selected.

1. The user opens the add-to-list assistant; the selection is pre-loaded.
2. The user chooses the destination list.
3. Either operation computes the subset of selected contacts that are not already members and writes one subscription per contact on the list, each with the creation moment of the operation.
4. The message *"<n> Mailing Contacts have been added. "* is shown, where `n` is the size of that subset.
5. With "Add", the assistant closes. With "Add and Send Mailing", a new mailing form opens with the destination list pre-selected.

---

## 10. Merge mailing lists

**Actor.** Marketing User. **Precondition.** At least one Mailing List selected, from a Mailing List screen.

1. The user opens the merge assistant. The selected lists become the sources; the first of them becomes the default destination.
2. The user chooses to merge into a new list, giving its name, or into an existing list.
3. On confirmation, when the option is `new`, a list with that name is created and becomes the destination.
4. The destination is added to the sources when it was not already among them.
5. All pending writes are flushed.
6. Subscriptions are inserted into the destination for the qualifying contacts: a contact qualifies when it belongs to one of the source lists, is **not** opted out of that source list, and its normalised address is **not** in the blocked-address register. Among all qualifying contacts sharing one address, only the first by address ordering is inserted. A contact is not inserted when another contact with the same raw address is already a member of the destination.
7. The record caches are dropped.
8. When "Archive source mailing lists" is on, every source except the destination is archived. Archiving fails, and therefore the whole merge fails, when one of them is used by an unfinished mailing.

**Worked example.** List A holds `sam@example.com` and `sam@example.net`; list B holds `sam@example.com` and `sam@example.org`; list C is empty. Merging A and B into C inserts three subscriptions: `sam@example.com` once (the duplicate is dropped by the per-address ranking), `sam@example.net` and `sam@example.org`.

---

## 11. Duplicate a mailing and manage favorite designs

**Duplicate.** From a finished mailing, or from the comparison-test page, the user presses Duplicate. A copy is created in `draft`, opened in a form. The copy keeps the subject, the bodies, the condition, the lists and the comparison-test moment; it loses the sent date, the state, the favorite marks, the traces and the exclusion-list override. When the original's mail server is archived, the copy falls back to the configured dedicated server.

**Add to templates.** The user presses "Add to Templates": `favorite` becomes true, `favorite_date` becomes the current moment, and the notification *"Design added to the <recipient entity names> Templates!"* is shown.

**Remove from templates.** `favorite` becomes false, `favorite_date` is cleared, and the notification *"Design removed from the <recipient entity names> Templates!"* is shown.

---

## 12. Comparison testing, end to end

**Actor.** Marketing User (Campaign Manager to see the campaign field). **Precondition.** A draft mailing.

1. The user opens the comparison-test page and turns the option on, then sets the percentage of the audience this version must reach.
2. On save, when the mailing has no campaign, one is created with: name `A/B Test: <subject>` (or the text-message title for a text-message mailing), the comparison-test moment, the comparison-test criterion, this mailing as member, and the mailing's responsible as owner. When the mailing already has a campaign, that campaign is kept.
3. The comparison-test job is asked to wake at the comparison-test moment.
4. The user presses "Create an Alternative Version", which duplicates the mailing. The copy belongs to the same campaign and keeps the comparison-test moment; the user edits its subject or body and sets its own percentage.
5. Each version is sent as in sections 4 to 7. Because comparison testing is on, each send draws a random sample of its percentage from the audience **minus everyone already traced for the campaign**.
6. While versions are running, "Compare Version" opens the list of every mailing of the same campaign with comparison testing enabled and the same mailing type, with their indicators side by side.
7. **Winner selection.**
   - *Manual*: the user presses "Send this as winner" on the chosen version. The version must have comparison testing enabled, otherwise *"A/B test option has not been enabled"*.
   - *Automatic*: at the comparison-test moment the job `Mail Marketing: A/B Testing` selects every campaign whose comparison-test moment has passed, whose criterion is not `manual` and which is not completed. For each, it takes the versions with comparison testing enabled; when none of them has reached `done` the campaign is skipped; otherwise the winner operation runs. The same is done separately for the text-message versions with the text-message criterion.
   - The winner operation refuses when the selected mailings do not share exactly one campaign (*"To send the winner mailing the same campaign should be used by the mailings"*) or when the campaign is already completed (*"To send the winner mailing the campaign should not have been completed."*).
   - With a non-manual criterion, the versions of the campaign are read with elevated rights, restricted to those in state `done`, and sorted on the criterion descending; the first is the winner. When no version has been sent, the operation is refused with *"No mailing for this A/B testing campaign has been sent yet! Send one first and try again later."*
8. **Winner sending.** The winning version is duplicated with `ab_testing_pc = 100` and the name `" <original name> (final)"`. The campaign records that copy as its winner, which sets the campaign as completed and therefore prevents any further winner operation. The copy is sent immediately (state `in_queue`, queue woken now). Because it is the campaign winner, its remaining-recipient computation excludes everybody traced by any version of the campaign. The winner mailing is then opened in a form.

**Worked example (20 percent, winner by open rate after 24 hours).** See [calculations.md](calculations.md) section 4.

---

## 13. Cancel and retry

**Cancel.** From state `in_queue` the user presses Cancel. The mailing is written with state `draft`, an empty schedule date, schedule type `now` and an empty next departure. Already created traces are not removed; when the mailing is sent again, their recipients are skipped.

**Retry.** From state `done` with at least one failure, the user presses Retry.
1. Outgoing mails linked to the mailing whose state is `exception` are loaded in pages of 1000.
2. For each page, the traces of those mails are deleted, then the mails themselves are deleted.
3. The mailing is queued again, which wakes the queue job immediately. The recipients whose traces were removed are therefore no longer "already contacted" and are prepared again.

For a text-message mailing the equivalent operation deletes the outgoing text messages in error together with their traces and then queues the mailing.

---

## 14. Unsubscribe from a mailing addressed to lists

**Actor.** Recipient, from a link in a delivered message. **Endpoint.** `GET /mailing/<mailing key>/unsubscribe` with the parameters `document_id`, `email` and `hash_token`.

1. The identity is resolved: when a token is supplied, or the visitor is anonymous, the supplied address and token are used; otherwise the signed-in user's own normalised address is used and no token is required.
2. The credentials are checked (section 18). A missing mailing answers "not found", which is converted to "unauthorized" in order not to reveal whether the mailing exists.
3. Because the mailing is addressed to lists, the subscription branch runs:
   1. Every one of the mailing's lists is opted out for that address: for every contact with that normalised address, every subscription of theirs to one of those lists that is currently opted in is written to opted out, which stamps the opt-out moment.
   2. A note is posted on each affected contact: *"<contact display name> unsubscribed from the following mailing list(s)"* followed by a bulleted list of the list names.
   3. The heading sentence is computed: when none of the mailing's lists is public, *"You are no longer part of our mailing list(s)."*; when exactly one list is involved, *"You are no longer part of the <list name> mailing list."*; otherwise *"You are no longer part of the <comma-separated names of the public lists> mailing list."*
4. The subscription-management page is rendered with the last action `subscription_updated`, that heading, and the values described in section 17.

**One-click variant.** `POST /mailing/<mailing key>/unsubscribe_oneclick` performs exactly the same work and answers with an empty success. It is the address advertised in the `List-Unsubscribe` header and is exempt from the cross-site request token, because the mail client calls it without a session.

**Confirmation variant.** `GET /mailing/<mailing key>/confirm_unsubscribe` renders a confirmation page instead of acting: `Are you sure you want to unsubscribe from the mailing list "<names of the public lists>"?` when at least one public list is involved and the mailing addresses Mailing Contacts, otherwise *"Are you sure you want to unsubscribe from our mailing list?"*, with an Unsubscribe button posting to `/mailing/confirm_unsubscribe`. That post performs the unsubscription and renders *"Successfully unsubscribed!"* with a "Manage Subscriptions" button pointing back at the management page.

---

## 15. Unsubscribe from a mailing addressed to another entity

Same entry points; the branch differs because there is no list to leave.

1. The address is added to the blocked-address register with elevated rights, with the note *"Blocklist request from unsubscribe link of mailing <link to the mailing> (document <link to the record>)"* when a record key was supplied, or *"Blocklist request from unsubscribe link of mailing <link to the mailing> (direct link usage)"* when it was not.
2. The management page is rendered with the last action `blocklist_add` and the heading *"You are no longer part of our services and will not be contacted again."*

---

## 16. Manage subscriptions from the portal

**Actor.** Signed-in user. **Endpoint.** `GET /mailing/my`.

1. The signed-in user's normalised address is resolved; an empty address answers "unauthorized".
2. The management page is rendered with no mailing, no record key and no token, and with the feedback form disabled.

---

## 17. The subscription-management page

The page is rendered with these values, for the resolved address:

| Value | Rule |
|---|---|
| contacts | Every Mailing Contact whose normalised address equals the resolved address, read with elevated rights. |
| lists the person is in | The active lists of every subscription of those contacts. |
| opted-in lists | The active lists of the subscriptions that are not opted out. |
| opted-out lists | The active lists of the subscriptions that are opted out **and** whose list is not already in the opted-in set. Therefore, when two contacts share an address and disagree, opted in wins. |
| suggested lists | The ten most recently created public lists that are neither opted in nor opted out for this address. |
| address validity | True when the address normalises. |
| self-exclusion possible | True when a blocked-address record exists or could exist for that address. |
| currently excluded | True when an active blocked-address record exists. |
| self-exclusion buttons shown | The value of the system parameter `mass_mailing.show_blacklist_buttons`, default true. |
| opt-out reasons | Every Mailing Opt-Out Reason, in sequence order. |
| feedback enabled | True, except on the portal page reached from `/mailing/my`. |

**Displayed behavior.** When the address is currently excluded, the page states `Your email is currently <strong>in our block list</strong>.` followed by *"You will not receive any news from those mailing lists you are a member of:"* and the opted-in lists (public ones by name, private ones as *"Mailing List #<key>"*), or by *"You will not hear from us anymore."* when there is no such list; all checkboxes are disabled. Otherwise the page shows *"Choose your mailing subscriptions"* with one checkbox per list the person is in, checked when opted in, annotated `Subscribed` or *"Not subscribed"*; private lists that the person is opted out of are hidden. Below, *"You may also be interested in"* lists the suggested public lists. When the person is in no list and none is suggested, the page says *"You are not subscribed to any of our mailing list."* The buttons are *"Apply changes"*, *"Exclude Me"* and *"Come Back"*.

---

## 18. Credential check for the public pages

Every public subscription endpoint runs this check before doing anything.

1. When **no token** is supplied: an anonymous visitor is refused with "bad request"; a signed-in user is refused with "bad request" when a mailing key was supplied and they are not a Marketing User. A signed-in Marketing User may therefore open any mailing's page without a token.
2. When a token **is** supplied: the triple mailing key, address and record key must all be present, otherwise "bad request".
3. When a mailing key is supplied: the mailing is loaded with elevated rights; a missing mailing answers "not found"; with a token, the token must equal the keyed hash of (database name, mailing key, record key, address) computed with the database secret using the strong hash function, compared in constant time, otherwise "unauthorized".
4. When no mailing key is supplied and the caller required one, "bad request".

Callers that render a page convert "not found" into "unauthorized", in order not to reveal which mailing keys exist. Callers that answer structured data return the strings `error` for a bad request and `unauthorized` for the other two.

---

## 19. Update subscriptions, give feedback, exclude and re-include

**Update subscriptions.** Endpoint `/mailing/list/update`, structured data, public. Inputs: mailing key, record key, address, token, the chosen list keys.
1. Credentials are checked; a mailing key is not required.
2. The contacts of the address are loaded.
3. The lists to opt out of are the lists of the person's non-opted-out subscriptions that are not among the chosen ones.
4. The lists to opt in to are the chosen lists that are either public or already a list of one of the contacts. A private list the person never belonged to cannot be joined this way.
5. Opting out switches the matching subscriptions and posts the unsubscription note on each contact.
6. Opting in switches the opted-out subscriptions back and creates a subscription on the **first** contact for each chosen list the person is not a member of; a note is posted: *"<contact display name> subscribed to the following mailing list(s)"* followed by the bulleted names.
7. The answer is the number of lists opted out of.

**Feedback.** Endpoint `/mailing/feedback`, structured data, public. Inputs: mailing key, record key, address, token, the last action, the chosen reason, the free text.
1. Credentials are checked; a mailing key is not required.
2. A missing reason answers `error`.
3. The free text is trimmed. When it is non-empty, the message is *"Feedback from <author>"* followed by a line break and the text, where the author is `<user name> (<address>)` for a signed-in user and the address alone for an anonymous one.
4. When the last action was `blocklist_add`: the blocked-address record of that address receives the reason, and the free-text message is attached to that change entry.
5. When the last action was `subscription_updated` or `subscription_updated_optout`, or when no action was given and either no mailing was supplied or the mailing is addressed to lists: every subscription of the address's contacts that is opted out **and** whose opt-out moment is within the last ten minutes receives the reason; the free-text message is posted on the contacts.
6. Otherwise, when a mailing was supplied and there is a free text, the message is posted on the recipient record named by the record key.
7. The answer is true.

**Exclude me.** Endpoint `/mailing/blocklist/add`, structured data, public. After the credential check the address is added to the blocked-address register with elevated rights, with the note *"Blocklist request from portal of mailing <link> (document <link>)"* when a mailing was supplied, otherwise *"Blocklist request from portal"*. The answer is true.

**Come back.** Endpoint `/mailing/blocklist/remove`, structured data, public. After the credential check the address is removed (archived) from the register, with the note *"Blocklist removal request from portal of mailing <link> (document <link>)"* when both a mailing and a record key were supplied, otherwise *"Blocklist removal request from portal"*. The answer is true.

---

## 20. Open tracking, view in browser and the statistics message

**Open tracking.** Endpoint `GET /mail/track/<outgoing mail key>/<token>/blank.gif`, public. The token must equal the keyed hash of the mail key under the purpose `mass_mailing-mail_mail-open`, compared in constant time; otherwise "unauthorized". Every trace whose stored plain integer mail key equals that key is marked opened. The answer is a one-pixel transparent image with the image content type.

**View in browser.** Endpoint `GET /mailing/<mailing key>/view`, public. Credentials are checked with a mailing key required. The rich-text body is rendered for the named record without forcing a language and without post-processing. When a record key was given, the unsubscribe placeholder is replaced by that recipient's unsubscribe address built on the recipient's own base address; otherwise it is replaced by the generic `<base>/mailing/<mailing key>/unsubscribe`. The result is rendered inside the browser-view page.

**Placeholder endpoints.** `GET /unsubscribe_from_list` is a placeholder that never appears in a real message; it redirects permanently to `/mailing/my` and is deliberately excluded from language prefixing. `GET /view` is a signed-in-only placeholder rendering a generic explanation page, used by the designer's preview.

**Mobile preview.** `GET /mailing/mobile/preview`, signed-in only, renders the mobile preview frame.

**Statistics message.** Produced by the queue job one day after sending, for every mailing with the statistics flag, state `done` and a sent date between one and five days ago.
1. The flag is cleared for the whole selection.
2. For each mailing: the acting identity becomes the responsible user with their language; the mailing type label is `Emails` or *"SMS Text Message"*; the link trackers of the mailing are loaded and sorted by click count descending and rendered as a table.
3. The body is composed of: an engagement block titled *"Engagement on <expected> <type> Sent"* with three columns, `<received_ratio>%` under *"RECEIVED (<delivered>)"*, `<opened_ratio>%` under *"OPENED (<opened>)"* and `<replied_ratio>%` under *"REPLIED (<replied>)"*; for a text-message mailing the block is titled *"Report for <expected> <type> Sent"* with `<received_ratio>%` under *"RECEIVED (<delivered>)"*, `<clicks_ratio>%` under *"CLICKED (<clicked>)"* and `<bounced_ratio>%` under *"BOUNCED (<bounced>)"*; a business block titled *"Business Benefits on <expected> <type> Sent"* filled by the installed bridges with the lead count under `LEADS`, the quotation count under `QUOTATIONS` and the formatted invoiced amount under `INVOICED`; the link-tracker table; and one random tip of the Email Marketing group when any exists.
4. The title and subject are `24H Stats of <type> "<subject>"`; a "More Info" button links to the mailing; the sent date is formatted in the responsible user's time zone and language as month, day and year.
5. When the responsible user is a Marketing User, a deactivation token and their key are added, producing the unsubscribe link of the statistics message itself.
6. One outgoing mail is created with elevated rights: author, sender and destination = the responsible user; answer address = the company address, else the user address; automatic deletion on; state `outgoing`.

**Deactivating the statistics message.** Endpoint `GET /mailing/report/unsubscribe`, public, with a token and a user key. A missing parameter answers "bad request". The user must exist, be a Marketing User and the token must equal the keyed hash of their key under the purpose `mailing-report-deactivated`, compared in constant time; otherwise "unauthorized". The system parameter `mass_mailing.mass_mailing_reports` is set to false and a confirmation page is rendered, carrying a link to the settings screen when the user is a settings administrator.

---

## 21. Click tracking and redirection

**Plain short link.** Endpoint `GET /r/<code>`, public.
1. When the caller is recognised as an automated agent (a preview fetcher or a crawler), no visit is recorded.
2. Otherwise a visit is recorded: the code is resolved to its tracker; when no code matches nothing is recorded. The visit carries the tracker, the caller's network address and, when the caller's country code is known, the matching Country.
3. The final address is resolved from the code (section 9.4 of [entities.md](entities.md)); when the code is unknown the answer is "not found".
4. The visitor is redirected permanently to that address.

**Short link carrying a recipient.** Endpoint `GET /r/<code>/m/<trace key>`, public. The visit is recorded exactly as above, plus the named trace. Before creating the visit, the trace is verified: a missing trace clears the reference; a present trace supplies the campaign and the mailing when those are not already set. After creation, the trace is marked opened and then marked clicked. The redirection is identical. This endpoint does **not** skip automated agents, because the trace key makes the visit attributable.

**Short link carrying a text message.** Endpoint `GET /r/<code>/s/<outgoing text message key>`, public. The trace whose plain integer text-message key equals the supplied key is looked up. Automated agents are skipped. The visit is recorded with that trace, then the redirection happens.

**Effects on measurement.** Each visit increases the tracker's click counter. The mailing's *clicked* counter counts traces with a non-empty last-click moment, therefore several clicks by one person count once; the tracker's counter counts every click.

---

## 22. Text message mailing, end to end

1. The mailing is created with type `sms`. The medium becomes `Text Message`; archives are kept by default; the title is copied into the subject; the body is the plain-text body, optionally seeded from a text-message template.
2. The audience is chosen exactly as for electronic mail.
3. Sending follows sections 4 to 7 with these differences:
   - The body is shortened with the plain-text shortener before sending, using the mailing's tracker values.
   - The exclusion checks are evaluated per record rather than per address: blocked numbers give `sms_blacklist`, opted-out records give `sms_optout`, records whose sanitised number was already used in this run give `sms_duplicate`, an unusable number gives `sms_number_format` and a missing number gives `sms_number_missing`. All of these produce the state `canceled`.
   - Opted-out records for a list-based mailing are the contacts opted out of at least one of the mailing's lists and opted in to none.
   - Already-contacted records are computed from the traces of this mailing joined to the recipient entity on its telephone fields, or on the telephone field of the linked contact when the entity has none; an entity with neither is refused with *"Unsupported %s for mass SMS"*.
   - Each kept message receives a trace with a fresh three-character code and, when the opt-out option is on, the sentence *"STOP SMS: <base>/sms/<mailing key>/<code>"* appended to its body.
   - After creation, every shortened address in each message body receives `/s/<message key>`.
   - When "Send Directly" is on, the batch is sent at once instead of being queued.
4. The provider reports back through the delivery-report endpoint. Each report updates the tracker, which updates the trace (with the status mapping and the ignore rules of section 28.11 of [entities.md](entities.md)) and then closes the mailing when no trace of it is still being processed.

---

## 23. Text message opt-out

**Entry page.** `GET /sms/<mailing key>/<code>`, public.
1. The mailing is loaded with elevated rights; when it does not exist the visitor is redirected to the back office.
2. The traces of type `sms` with that code and that mailing are loaded; when none exists the visitor is redirected to the back office.
3. When the visitor submitted a number, it is sanitised against: the country resolved from their network location, else the company country. A number that cannot be sanitised leaves the page with the error *"Oops! The phone number seems to be incorrect. Please make sure to include the country code."*
4. When the sanitised number matches the number stored on one of the traces, the visitor is redirected to the confirmation page with that number. When it does not, the page is shown again with *"Oops! Number not found"*.

**Confirmation page.** `GET /sms/<mailing key>/unsubscribe/<code>`, public.
1. The traces are re-verified as above; a failure redirects to the back office.
2. The trace matching the submitted number is taken.
3. When the mailing is addressed to lists: every subscription of those lists whose contact has that sanitised number is written to opted out. The opted-out lists are reported.
4. When the mailing is not addressed to lists: the number is added to the blocked-number register and a note is logged on that record: *"Blacklist through SMS Marketing unsubscribe (mailing ID: <mailing key> - model: <recipient entity display name>)"*.
5. The lists that number is still opted in to, outside the mailing's lists, are reported.
6. The confirmation page is rendered with both lists.

---

## 24. Website subscription

**Reading the current state.** Endpoint `/website_mass_mailing/is_subscriber`, structured data, public. Inputs: the list key and the subscription kind (`email` or `mobile`). The current value is the signed-in user's address, else the address stored in the visitor's session, for the `email` kind; the signed-in user's telephone number, else the number stored in the session, for the `mobile` kind. The answer reports whether an opted-in subscription exists for that list and that value, and echoes the value.

**Subscribing.** Endpoint `/website_mass_mailing/subscribe`, structured data, public. Inputs: the list key, the value and the kind.
1. The human-verification token for the action `website_mass_mailing_subscribe` is verified. On failure the answer is a danger notice carrying the verification error text.
2. For the `email` kind the value is parsed into a name and an address; when no name is found, an optional supplied name is used. For the `mobile` kind the value itself is the name.
3. A subscription of that list whose contact carries that value is searched. When none exists, a contact with that value is searched, created when missing, and a subscription is created. When one exists and is opted out, it is switched back to opted in.
4. The value is stored in the visitor's session.
5. The answer is a success notice with the text *"Thanks for subscribing!"*.

**Public-list guard on the generic website form.** When a website form submits to the Mailing Contact entity, the list keys are required, otherwise the answer is the structured error *"Mailing List(s) not found!"*. Any of the named lists that is not public makes the answer *"You cannot subscribe to the following list anymore : <comma-separated names>"*.

**Checkout option.** With the Checkout Newsletter package, when the shopper ticks the newsletter box at checkout and an address is present, the same subscription operation runs for the website's configured newsletter list, using the checkout address and name.

---

## 25. Create a shortened link from the links page

**Actor.** Signed-in user with the right to create link trackers.

1. The user opens `GET /r`, which renders the link-shortening page and tells it whether the user may create trackers and codes.
2. The user types a target address and optional campaign, source and medium names. Each name is turned into a record by the find-or-create operation, which is case-insensitive and reuses archived records.
3. The user submits to `/website_links/new`. An empty address answers `{"error": "empty_url"}`. Otherwise the find-or-create tracker operation runs and the resulting tracker is returned.
4. The page shows the short address. The user may add an extra code through `/website_links/add_code`, which creates a further Link Tracker Code pointing at the same tracker when that code does not already exist for it.
5. The user may list their recent links through `/website_links/recent_links` with one of the three orders.
6. `GET /r/<code>+` renders the statistics page of that code's tracker; an unknown code redirects permanently to the site root.

---

## 26. Marketing cards, end to end

**Actor.** Marketing Card User.

1. The user creates a campaign: a name, a design (template), a preview record, the content mapping, the target address, the post suggestion, the request title and description, and the reward message and link.
2. On creation a Link Tracker is created for the target address, attributed to the shipped `Marketing Card` source, and the campaign's entity is derived from the preview record.
3. The preview image is rendered from the design and the preview record and stored on the campaign.
4. The user presses Preview: the preview record's card is created or refreshed, stored with the current preview image, archived, and its page opens.
5. The user presses Share: a mailing form opens, pre-filled with the campaign, the campaign name as subject, the campaign's entity as recipient entity and a body containing an introduction and a clickable centred preview image.
6. The user chooses the audience and presses "Update cards": for every draft mailing of the campaign, the cards of the current recipients are produced in batches of 100 with a commit between batches.
7. Queueing or sending refuses while any recipient still lacks an up-to-date card.
8. During preparation, each recipient's generic card addresses in the body are replaced by their own card addresses, and the duplicate-address suppression is disabled because every recipient receives a different image.
9. The recipient opens their preview page `GET /cards/<slug>/preview`: when their card has no share status it becomes `visited`; the page shows the request title and description, the image, the post suggestion and the share buttons.
10. When they share, the social network's crawler fetches `GET /cards/<slug>/redirect`: recognising the crawler by its agent string, the system answers a page carrying only the preview metadata (the image address, the post text and the target name). A human visiting the same address is redirected to the campaign's target address, or, when the card is active, to the campaign's tracker short address, which records a click.
11. The crawler then fetches `GET /cards/<slug>/card.jpg`: because the caller is a crawler and the card is not yet `shared`, the status becomes `shared`. The image bytes are answered with the image content type and a download name of `card.jpg`. A card with no image answers "not found".
12. The campaign counters follow: `card_count` counts all cards, `card_click_count` counts `visited` and `shared`, `card_share_count` counts `shared`.
13. After the retention period, the cleanup routine deletes old cards.

**Recognised crawler agents.** `Facebot`, `facebookexternalhit`, `Twitterbot`, `LinkedInBot`, `WhatsApp`, `Pinterest`, `Pinterestbot`. The check is a substring match on the agent string.

---

## 27. Bounce and answer handling

**Answer.** An incoming message reaches the gateway and is routed to a record. The message keys of its reference or in-reply-to headers are looked up among the traces. Matching traces are marked opened and then replied. Because the answer is routed, the recipient's answer also appears in the record's discussion thread when the answer mode was `update`.

**Bounce.** An incoming message is recognised as a bounce.
1. The traces whose message key is among the bounced keys are marked bounced with the plain-text bounce body as reason.
2. Every bounced trace whose stored address differs from the bounced address causes the underlying recipient record to receive the bounce notice, because the standard handling matched on address and would have missed it.
3. The automatic blocking rule of section 28.8 of [entities.md](entities.md) is evaluated.

**Effect on the Mailing Contact.** The bounce counter of the contact increases through the blocked-address mixin, which feeds the list's bouncing percentage and the *Bounced* filter of the contact screen.
