# Messaging and Activities — Workflows

End-to-end operational procedures, step by step, with the role that performs each one, its preconditions, the records it creates or changes, and its failure conditions.

Contents:

1. [Posting a message on a record](#1-posting-a-message-on-a-record)
2. [Logging an internal note](#2-logging-an-internal-note)
3. [Notifying specific people without touching the conversation](#3-notifying-specific-people-without-touching-the-conversation)
4. [Delivering the notifications](#4-delivering-the-notifications)
5. [Composing with the composer](#5-composing-with-the-composer)
6. [Scheduling a message for later](#6-scheduling-a-message-for-later)
7. [Deferring only the notifications](#7-deferring-only-the-notifications)
8. [Managing followers](#8-managing-followers)
9. [Recording a tracked field change](#9-recording-a-tracked-field-change)
10. [Fetching and processing incoming mail](#10-fetching-and-processing-incoming-mail)
11. [Handling a bounce](#11-handling-a-bounce)
12. [Emptying the outgoing queue](#12-emptying-the-outgoing-queue)
13. [Scheduling and completing an activity](#13-scheduling-and-completing-an-activity)
14. [Launching an activity plan](#14-launching-an-activity-plan)
15. [Opening and using a conversation channel](#15-opening-and-using-a-conversation-channel)
16. [Running a call](#16-running-a-call)
17. [Handling a live chat visitor](#17-handling-a-live-chat-visitor)
18. [Running a mailing list](#18-running-a-mailing-list)
19. [Sending text messages](#19-sending-text-messages)
20. [Shipping a digest](#20-shipping-a-digest)
21. [Sending postal mail](#21-sending-postal-mail)
22. [Subscribing a browser to push notifications](#22-subscribing-a-browser-to-push-notifications)
23. [Managing the suppression list](#23-managing-the-suppression-list)
24. [Resetting a template](#24-resetting-a-template)
25. [Using the electronic-mail client plugin](#25-using-the-electronic-mail-client-plugin)

---

## 1. Posting a message on a record

**Role:** any user, internal or portal, who satisfies the create branch of the message access policy on the target record.

**Preconditions:** the target is a single real record of a thread-enabled model; the message type is not the user-specific type.

### Steps

1. **Validate the parameters.** Reject the forbidden names; reject a call on the behavior itself or on a record with no identifier; reject the user-specific type; check the shape of the attachment lists and the recipient list. See [business-rules.md](business-rules.md), section 4.
2. **Split the extra parameters** into those that are fields of a Message and those that are notification parameters.
3. **Set the language** of the operation to the acting user's language when the caller did not set one, so that every later rendering is consistent.
4. **Determine the author.**
   - If the caller supplied no author, the acting user is a public user and a guest token is present: the author is that guest, and both the author contact and the sender address are cleared.
   - Otherwise compute the pair (author contact, sender address): when no author was given and a sender address was, look the address up among contacts without creating one; when neither was given, use the acting user's contact and its formatted address; when an author was given but no address, use the author's formatted address.
5. **Resolve the subtype.** An external identifier wins; failing that the supplied identifier; failing that the internal-note subtype.
6. **Subscribe recipients if asked.** When the subscribe switch is on and there are direct recipients, subscribe them. Otherwise, when the model declares a strong customer tie and the model's main customer is among the direct recipients, subscribe that customer alone.
7. **Build the message values:** author, guest author, sender address, model, record, body (escaped when it is plain text, preserved when it is already markup), message type, parent (see the parent rule), subject, subtype, direct recipients, incoming "to", incoming carbon copy, outgoing "to", and the add-signature flag defaulting to on.
8. **Fill the environment values** that were not supplied: the alias domain of the record, the company of the record, and the reply address.
9. **Process the attachments.** Transfer any attachment that belongs to a composer or a scheduled message and was created by the acting user onto the record; for a non-internal caller, restrict the list to exactly those; create the inline attachments, giving an access token to any that the body references by content identifier or by file name, and rewrite those references in the body to point at the stored attachment.
10. **Create the Message.** During creation, every image embedded inline in the body is extracted into a real attachment with an access token and the body is rewritten; the tracking values supplied are created with elevated rights; the message identifier and the reply address are generated when absent.
11. **Subscribe the author** when all of: the skip switch is off; the message type is not one of system notification, user-specific notification, automated notification or out-of-office answer; and the subtype is exactly the shipped "Discussions" subtype. Only the *real* author is subscribed, and only when that contact is not a customer.
12. **Run the model's post hook.**
13. **Notify** (see section 4).
14. **Return the Message.**

### Records created or changed

| Record | Change |
|---|---|
| Message | created |
| Attachment | created for inline content and embedded images; transferred from the composer |
| Follower | created for the author, and for the recipients when asked |
| Notification | one per recipient and channel |
| Outgoing Mail | one per (language, group, chunk) |
| Message Notification Schedule | created instead of the above when the notification is deferred |
| the record itself | its unread and error counters change; a channel's last-interest moment is raised |

### Failure conditions

Any of the validations of [business-rules.md](business-rules.md), section 4; the message access policy refusing the creation; read access missing on an attachment that belongs to another record.

---

## 2. Logging an internal note

**Role:** internal code, or a user through the composer in note mode.

The logging operation is deliberately cheaper than posting: it performs **no** notification at all and creates the message with elevated rights, so it must only be called where access has already been checked.

### Steps

1. Reject attachments or tracking values when more than one record is targeted.
2. Compute the author pair as in step 4 of the posting workflow.
3. Build one value set per record with: the author; the sender address; the model; no alias domain; no company; the attachments; the message type (default system notification); the employee-only flag set; the subject; the internal-note subtype; the tracking values; the add-signature flag **off**; a generated message identifier; the direct recipients, which are stored but used by no notification mechanism; and the generic reply address.
4. Create the messages in one call, with elevated rights.

A variant renders the body of each record from a view before step 3.

---

## 3. Notifying specific people without touching the conversation

**Role:** internal code, and the composer when the target model has no conversation.

**Purpose:** push something to named people's inbox or mailbox without adding an entry to any conversation. The resulting message is of the user-specific type, which the conversation query excludes and which the access policy hides from everyone but the recipients.

### Steps

1. Reject the call when there are no recipients (log a warning and return nothing).
2. Validate the forbidden parameters and the shapes.
3. Split the extra parameters; force the "notify the author when mentioned" switch on unless the caller set it, because a person notifying themselves should see it.
4. Compute the author pair.
5. Accept a model and a record only when **both** are given; this allows notifying about a record whose model has no conversation.
6. Resolve the subtype, defaulting to the internal note.
7. Build the values: author, sender address, model, record, body, employee-only flag on, message type "user specific notification", subject, subtype, a generated message identifier carrying the notify tag, the recipients, and the add-signature flag on.
8. Fill the alias domain, the company and the reply address when a record is present.
9. Process the attachments as in posting.
10. Create the message and notify.

---

## 4. Delivering the notifications

**Role:** the platform, immediately after a message is created.

### Steps

1. Set the language of the operation.
2. Validate the notification parameters.
3. **Compute the recipients** (see [calculations.md](calculations.md), section 2).
4. Warm the cache with the contacts of the users found, so that reading them costs nothing.
5. **Run the out-of-office pass** *before* checking whether there are recipients, because an absent person's automatic answer may involve people the recipient list does not contain.
6. If there are no recipients, stop.
7. If a scheduled moment was supplied and it parses to a future moment, create a Message Notification Schedule carrying the message, the moment and the serialized parameters, and stop.
8. Otherwise, in order: deliver to the in-application inbox; deliver by electronic mail; deliver by browser push.

### The inbox pass

1. Keep the recipients whose channel is the inbox and that have both a contact and a user; sort them.
2. Create one Notification per recipient: author, message, status "delivered", channel "inbox", recipient.
3. Fetch, in one query, the follower rows of those users on this record.
4. For each user, build the client payload — the message rendered for that user with an empty company scope, plus the follower information — and broadcast it on that user's own channel under the inbox event name, carrying the message identifier and the payload.

### The electronic-mail pass

1. Keep the recipients whose channel is electronic mail. If there are none, stop.
2. Build the **base mail values**: the message link; the reference chain; the subject, which is the message subject, else the record's computed subject, else the record name, with newlines replaced by spaces; the headers.
3. Build the headers: the caller's extra headers; the list of external addresses to add so that a reply-to-all reaches them, but only when there are fewer than the customer-header limit of 50; the return path taken from the message's alias domain; and the model-and-record header. A return path already present is not overwritten.
4. Build the **base notification values**: author, read flag on, message, status "ready", channel "email".
5. Iterate over the (language, group) pairs (see [calculations.md](calculations.md), section 3): render the layout; create the outgoing mails in chunks; create the notifications.
6. Decide whether to send now and, if so, send after the transaction commits.

### The browser-push pass

1. Select the recipients eligible for an extra notification: never the author, and never someone outside the platform unless the message is a comment. Precisely: for a comment, every active recipient with a contact except the author; for a system notification, a user-specific notification or an incoming electronic mail, every active recipient with a contact except the author **and** except those whose channel is not the inbox; for anything else, nobody.
2. Fetch their registered devices and the signing key pair. Stop when either is missing.
3. Build the payload: the title is the record name, prefixed by the author's name when there is one; the icon is the author's avatar, or the platform icon; the body is the plain text of the message body plus the tracking summary; the data block carries the model and the record.
4. When the body is empty and there are attachments, the body becomes the attachment name, or "<first> and <second>", or "<first> and <count> other attachments"; a voice recording is named "Voice Message".
5. Truncate the payload (see [calculations.md](calculations.md), section 29).
6. If there are fewer than five devices, push to each directly, deleting any device the service reports as permanently gone and logging any other error without aborting. Otherwise create one queued Push Notification per device and wake the push job.

### The out-of-office pass

1. Stop unless the target is a real, non-transient record and the message type is comment or incoming electronic mail.
2. Determine who is to receive the automatic answer: the real author's contact, or, when there is none, the sender address.
3. Collect the internal users to check: every recipient that is active, has a contact **and** is among the direct recipients, is not the author, has a user and is not shared; plus the record's responsible when the model has one and it is not the author; plus the parent message's author when that author is active, is not already a recipient, is not the author and is not an external contact.
4. Keep those that are currently absent and have a non-empty absence message.
5. Look for an automatic answer already sent by any of them, to this same recipient or address, in the last **four days**. Those are skipped.
6. For each remaining absent user, post a message on the record with: the shipped absence template rendered with the absence message, the replied body and the user's signature; the author set to the absent user; the sender set to their formatted address; the headers marking the message as an automatic reply and suppressing further automatic answers; the message type "out-of-office"; the notify-author switch on and the skip-followers switch on; the recipient set to the original author, or the outgoing address when there is none; the subject "Auto: <original subject or the record display name>"; and the "Discussions" subtype.

---

## 5. Composing with the composer

**Role:** any user who may post on the target records.

### Opening

The window opens with a mode, a model and a set of records. The mode is "post on a document" for one record, and either mode for many, depending on what the caller asked for.

Every field is computed from the template when one is chosen, and remains editable. Choosing a template recomputes the subject, the body, the language, the attachments, the sender, the recipients, the reply address, the relay, the scheduled moment and the automatic-deletion flag; clearing the template clears them again.

### Sending in "post on a document" mode

1. Evaluate the target records: either the stored list, or the stored filter evaluated with the responsible user's rights.
2. If there is no record, abort with "Mail composer in comment mode should run on at least one record. No records found (<model>)."
3. Prepare the per-record values. In single-record mode nothing is rendered: the typed subject and body are used verbatim. In batch mode the subject and body are rendered per record.
4. For a batch, turn on the switch that prevents the author from being subscribed to every record.
5. For each record: if the model has a conversation, post; otherwise notify, having removed the message type (forced to the user-specific type) and the parent, and having added the model and the record.
6. A notification that finds no recipient aborts with "No recipient found."

### Sending in "mass mail" mode

1. Evaluate the target records as above.
2. Split them into batches of the configured generation size (default 50).
3. For each batch: render the values per record; create one Outgoing Mail per record with elevated rights; create the notifications.
4. Create notifications **only** when the mails are not going to be deleted, or when the "keep a copy" switch is on. For each mail: one notification per contact recipient, plus one per free address; when the mail has neither, a single notification with no recipient at all, which will carry the "missing address" failure.
5. Run the model's mass-mail hook.
6. If the "send directly" switch is on, filter out the mails whose scheduled moment is still in the future and send the rest; otherwise report progress and clear the cache, so that a very large run does not accumulate memory.

### Saving the content as a template

The window offers saving what is typed as a new Template, using the template-name field. The new template belongs to the acting user unless they are a template editor.

---

## 6. Scheduling a message for later

**Role:** any user who may post on the record.

**Precondition:** single-record mode only; a scheduled moment strictly in the future.

### Steps

1. Reject the call unless the mode is "post on a document" and the batch flag is off — "A message can only be scheduled in monocomment mode".
2. Prepare the posting values exactly as for an immediate send.
3. Reject the call when no scheduled moment resulted — "A scheduled date is needed to schedule a message".
4. Build the Scheduled Message: the attachments, the author, the body, the comment option, whether it is a note, the model, the recipients, the record, the scheduled moment, the cleaned sending context, the subject, and — as the notification parameters — everything else that was prepared, serialized.
5. Create it. The attachments that belong to the composer and were created by the acting user are transferred onto it.
6. Wake the posting job for the set of scheduled moments.

### The scheduled posting job

1. Select up to 50 scheduled messages whose moment has passed.
2. For each, acting **as its creator**: re-check that the creator still has the required permission on the record; post the message with the stored values, keeping only the allowed notification parameters; run the creation hook; commit.
3. On failure: roll back, log, and notify the creator with the subject "A scheduled message could not be sent" and a body carrying the original content.
4. Delete every processed scheduled message, successful or not.
5. If more remain, wake the job again.

### Editing or cancelling

The author, or an administrator, may reopen the entry and change any of its content, including the moment; changing the moment re-wakes the job for the new moment. Cancelling deletes the entry and posts nothing. Sending immediately is restricted to the creator and to administrators.

---

## 7. Deferring only the notifications

**Role:** internal code that posts a message with a scheduled moment.

The message is posted immediately and is visible in the conversation; only the notification pass is deferred.

1. The notification pass detects the future moment, creates the Message Notification Schedule and stops.
2. The scheduled job, run hourly, picks up every schedule whose moment has passed, replays the stored parameters and produces the inbox, electronic-mail and browser-push deliveries, then deletes the schedule.
3. A caller may force the release early, or change the moment.

---

## 8. Managing followers

### Subscribing one person

**Role:** any user with write access on the record; or any user with read access when subscribing themselves.

1. Determine whether the caller is subscribing only themselves.
2. Check the permission: read when subscribing only oneself (and silently give up when even that is missing), write otherwise.
3. When subscribing others, drop the archived contacts.
4. Insert the followers with the default subtypes when none were given, using the "skip existing" policy; or with the given subtypes, using the "replace" policy.

### Unsubscribing

1. Check the permission: write when removing somebody else; read when removing oneself and not being an internal user; nothing at all when an internal user removes themselves.
2. Delete the matching Follower rows with elevated rights.

### The bulk editor

**Role:** any internal user.

The window collects a model, a set of records, an operation (add or remove), a set of contacts and, optionally, a notification message.

1. On "add": subscribe the contacts to every record; when a message was typed and the notify switch is on, post it on each record with the contacts as direct recipients.
2. On "remove": unsubscribe the contacts from every record.
3. Report "Followers added", "Followers removed" or "Followers updated".

### Editing the subtypes of one follower

A follower may change **their own** subscription from the conversation panel. Writing a subtype set uses the "replace" policy, so unchecking a box removes that subtype.

---

## 9. Recording a tracked field change

**Role:** the platform, on every modification of a thread-enabled record.

### Steps

1. Before the write, snapshot the tracked fields (first value wins).
2. Write.
3. Run automatic subscription on the written values.
4. At the end of the transaction: for every record with a snapshot that is not the discard marker, compare and build the tracking entries.
5. Ask the model which subtype the set of changed fields raises.
6. Post a message with that subtype, or log an internal note when there is none but there are entries.
7. Ask the model which template to post for the set of changes, and post or mass-mail it.

See [calculations.md](calculations.md), section 7, for the full comparison rules and a worked example.

---

## 10. Fetching and processing incoming mail

### The polling job

**Role:** the platform, every five minutes, and only when at least one incoming server is confirmed.

1. Select the confirmed incoming servers, ordered by priority ascending.
2. For each, open a connection of the configured kind.
3. Fetch the messages the server offers, honouring the "keep attachments" and "keep the original" settings: when attachments are not kept, they are stripped before processing; when the original is kept, the whole raw message is attached as a file.
4. Hand each raw message to the processing algorithm, with the server's fallback model.
5. On success, stamp the last-fetch moment and clear the error fields. On failure, stamp the error moment and the error text; the server stays confirmed.

An alternative intake exists: the external mail transfer agent pipes the raw message directly into a shipped script, which calls the same processing entry point over the remote interface. The script path is shown, read-only, on the server record.

### Processing one raw message

The complete algorithm is in [calculations.md](calculations.md), section 16. In outline:

1. Parse.
2. Guard against duplicates and against replies to the platform's own bounces.
3. Route: bounce, reply, catch-all, alias, fallback, or refusal.
4. Guard against loops.
5. Enrich the parsed values with the author and the recipients resolved in the context of the routes.
6. Process each route: create or update the record, then post the message.

### Worked example

See [calculations.md](calculations.md), section 16, "an inbound message on an alias creating a record".

---

## 11. Handling a bounce

**Role:** the platform, on receiving a message recognized as a bounce.

1. Recognize the bounce (three independent detections).
2. Extract the bounced address, the bounced contact, the original message and the references.
3. Raise the bounce counter of every blacklist-enabled record whose normalized address equals the bounced address, and of the originally addressed record when it was not already caught.
4. Update the matching Notifications of the original message: failure reason, failure type "bounce", status "bounced".
5. Re-broadcast the original message to its author's client so the error badge appears.
6. Return no route: a bounce never creates or updates a record.

A channel additionally removes a member whose bounce counter has reached ten.

See [calculations.md](calculations.md), section 18, for a worked example.

---

## 12. Emptying the outgoing queue

**Role:** the platform, hourly, and on demand.

### Steps

1. Select the queue: mails in the outgoing state whose scheduled moment is empty or past, ordered by identifier, limited to the batch size (default 1000). When a specific set of identifiers is requested, the limit is ten times the batch size and the result is intersected with that set.
2. Sort the identifiers ascending.
3. Group by sending configuration and resolve a relay per group (see [calculations.md](calculations.md), section 19).
4. For a group whose relay is personal, apply the throttle (see [calculations.md](calculations.md), section 20); a group left with nothing to send is skipped.
5. Open one connection per group. A failure to connect marks the whole group failed with the relay-failure type; when the caller asked for exceptions, abort with "Unable to connect to SMTP Server".
6. For each mail of the group, in order:
   1. skip it unless it is still in the outgoing state;
   2. write the provisional failure state and flush the notifications;
   3. build the list of sub-messages (one per free recipient block, one per contact recipient), personalizing the body of each (the unfollow link is rewritten for a recipient allowed to unfollow, and stripped otherwise);
   4. prepare the attachments, converting the oversized and the link-only ones into links;
   5. send each sub-message, recording the successful contacts and addresses; an invalid-recipient failure on one sub-message is recorded and the loop continues, so one bad address does not block the others;
   6. if at least one sub-message went out, write the sent state and the returned identifier and clear the failure fields; otherwise write the accumulated failure reason and type;
   7. post-process: split the notifications into delivered and failed, re-broadcast the failed messages to their authors, and delete the mail when the automatic-deletion flag is set and either there was no failure or the failure is a missing or invalid address;
   8. commit when running with auto-commit and report progress.
7. Close the connection.

### Failures that abort rather than mark

A memory exhaustion, a database error and a disconnected relay session are re-raised; the transaction rolls back and the mails stay queued.

---

## 13. Scheduling and completing an activity

### Scheduling by hand

**Role:** any user with the required permission on the record.

1. Open the scheduling window from the record.
2. Choose a type. Choosing it fills the summary from the type's default summary, the due date from the type's delay rule, the assignee from the type's default user or the acting user, and the note from the type's default note.
3. Adjust the summary, the note, the due date and the assignee.
4. Save.

### Scheduling from business code

The helper takes a type named by external identifier, a due date defaulting to today in the acting user's time zone, a summary and a note, and creates one activity per record with the **automated** flag set. A named type that does not exist, or that belongs to another model, falls back to the model's default type with a warning.

Everything is skipped when the "skip activity automation" switch is on.

### Side effects at creation

1. Every assignee other than the acting user is notified with the assignment notification, rendered in their language (see [entities.md](entities.md), section 17), unless the quick-update switch is on.
2. Every assignee's contact is subscribed to the related record.
3. Every assignee with an activity due today or earlier gets a counter increment broadcast.

### Completing

**Role:** the assignee, or any user with the required permission on the record.

1. Optionally type a feedback text and attach files.
2. Confirm.
3. The completion sequence runs (see [state-machines.md](state-machines.md), section 3): prepare the successor, post the completion message, move the attachments, create the successor, archive, store the feedback.

Two variants exist: "complete and schedule the next one", which opens the scheduling window pre-filled with the previous type and the previous due date unless a successor was created automatically; and "complete and go back to the list", which returns to the list of the user's remaining activities.

### Rescheduling and cancelling

Three one-click reschedules exist: to today, to tomorrow, and to the Monday of the following week. Cancelling deletes the activity outright and posts nothing.

### Purging

A maintenance routine deletes up to ten thousand activities per run whose due date is older than the configured number of years. A missing or zero setting disables the routine with a warning; a negative setting disables it with a different warning.

### Worked example

See [calculations.md](calculations.md), section 13.

---

## 14. Launching an activity plan

**Role:** any user with the required permission on the records.

**Precondition:** the targets are real documents — launching a plan on nothing is refused with "Plan-based scheduling are available only on documents."

### Steps

1. Open the scheduling window with a set of records of one model. Records are processed in batches of 500.
2. The window offers the plans applicable to that model and, when the plan carries one, that company.
3. Choose a plan. The window shows one preview row per line: a description, the computed due date and the responsible.
4. Choose the plan date; it defaults to today in the acting user's time zone.
5. When at least one line asks for the assignee at launch, choose that user; it defaults to the acting user.
6. The window computes the blocking errors and the non-blocking warnings and displays them as lists. A blocking error prevents the launch.
7. Confirm. One Activity is created per (record, line): the line's type, the line's summary, the line's note, the computed due date and the resolved assignee.
8. Each created activity triggers the ordinary creation side effects: assignment notification, subscription, counter broadcast.

### Blocking errors

- a line in "Ask at launch" mode with no user chosen: "No responsible specified for <type name>: <summary or a hyphen>.";
- an assignee that the company check refuses.

### Worked example

See [calculations.md](calculations.md), section 14.

---

## 15. Opening and using a conversation channel

### Creating a public channel

**Role:** any internal user.

1. Supply a name and, optionally, an authorization group.
2. A Channel of type "channel" is created. The acting user is added as a member automatically. Every active user of every auto-subscription group is added as well.
3. A notification message "created this channel." is posted.
4. The channel is broadcast to the creator's client.

### Creating a group conversation

**Role:** any internal user.

1. Supply a list of contacts, optionally a name and optionally a default display mode.
2. A Channel of type "group" is created with one member per contact, plus the acting user.
3. The channel header is broadcast to every member.

### Opening a direct conversation

**Role:** any internal user.

1. Supply the other contact.
2. Refuse when more than two people are involved — "A chat should not be created with more than 2 persons. Create a group instead."
3. Search for an existing conversation of type "chat" whose member set is **exactly** that pair.
4. If one exists: when pinning was requested, raise the acting member's last-interest moment and clear its unpin moment; broadcast the header to the acting user.
5. If none exists: create one, with the acting user's member unpinned-never and the correspondent's member unpinned **now**, so that the conversation stays invisible to the correspondent until the first message; name it with the two names joined by a comma; broadcast the header to both.

All three use the same "now": the last-interest moment is one second before it, so that the unpin moment is strictly later and the pin rule evaluates to "not pinned" for the correspondent.

### Starting a sub-thread

**Role:** any member of a channel or group.

1. Optionally supply the message the thread starts from, and a name.
2. When no name is given, it is "New Thread"; or, when the source message is void, "This message has been removed"; or the first 30 characters of the source message's text.
3. A Channel of the same type is created with the parent and the source message; the acting user and the source message's author are added as members, without a join message.
4. A notification message is posted in the **parent**: "<user> started a thread: <link to the sub-thread>."

### Joining and leaving

| Action | Effect |
|---|---|
| join | a member row is created; a "joined the channel" notification is posted for any type other than a plain channel; the channel and the new member are broadcast |
| invite | the same, with the message "invited <name> to the channel" and the inviting contact as author; the invited member's client receives a dedicated event carrying the channel, the call-invitation flag and the inviting user |
| leave | for a type allowing it: the member is told to close the window, a "left the channel" notification is posted for any type other than a plain channel, the member row is deleted, and the removal plus the new member count are broadcast |
| leave, for a type not allowing it | the conversation is unpinned instead |

Adding a member to a sub-thread also adds them to the parent. Removing a member from a channel also removes them from every sub-thread.

### Inviting by address

**Role:** an internal user with read access on the channel.

**Precondition:** the channel is a group, or a channel with no authorization group.

1. Normalize and deduplicate the addresses.
2. Remove the addresses that already belong to a member.
3. For each remaining address, render the shipped invitation template with the base address, the channel, a signed token bound to that address, the invitation body and the acting user.
4. Create one electronic mail per address, of the user-specific type, attached to the channel, with the subject "<acting user name> has invited you to a channel", and send them immediately, raising on failure.

### Reading, pinning, muting, renaming

| Action | Effect |
|---|---|
| mark read up to a message | see [calculations.md](calculations.md), section 23 |
| mark fetched | the last downloaded identifier is written with a lock-skipping update, and a lightweight event is broadcast to the channel; only for direct conversations |
| pin | the member's unpin moment is cleared and the header is broadcast to the acting user |
| unpin | the unpin moment is stamped and the client is told to close the window |
| mute until a moment | the moment is written and the un-mute job is woken for it |
| set a personal name | the member's custom name is written and broadcast to that member only |
| rename the channel | the name is written and a notification message carrying the new name is posted |
| change the description | the description is written |
| pin a message | the message's pinning moment is written **by direct update**, deliberately without touching the modification moment, so that pinning does not look like an edit; the new value is broadcast; a notification message "<member> pinned a message to this channel." with a link to the message and a link to the pin list is posted |
| unpin a message | the pinning moment is cleared and broadcast; no message is posted |

### Commands

Three commands are shipped.

| Command | Effect |
|---|---|
| help | a transient message is sent to the acting user alone: in a channel, "You are in channel **#<name>**."; otherwise "You are in a private conversation with <member links>." or "You are alone in a private conversation."; followed by the five usage lines describing mentions, channel references, commands, canned responses and emoji |
| leave | leaves the channel for a type allowing it, otherwise unpins it |
| who | a transient message listing up to 30 other members as links, with a final "you" or "more"; or "You are alone in this channel." |

A transient message is broadcast to one recipient and stored nowhere.

---

## 16. Running a call

**Role:** any member of a channel.

### Joining

1. Delete every other call session of the same party anywhere.
2. Clear the member's ringing link.
3. Create the session with the requested camera state.
4. Collect the dead sessions of the channel.
5. Fetch the traversal server list: from the configured external provider when one is set, otherwise the stored servers.
6. Decide the topology: below three sessions, clear the channel's forwarding-unit fields and stay browser-to-browser; from three on, request a channel from the forwarding unit using a 30-second signed token, store its identifier and address, and tell every existing session to switch over. A unit that cannot be reached leaves the call browser-to-browser with a warning.
7. Send the client the current sessions, the outdated sessions to forget, the traversal servers, its own session and the forwarding-unit information (address, channel identifier and an eight-hour signed token).
8. When this is the first session of a conversation that is not a plain channel, ring the other members.

### Ringing

1. Select the members eligible to be rung (see [state-machines.md](state-machines.md), section 21).
2. Set their ringing link and broadcast the invited list to the channel.
3. Push a browser notification titled "Incoming call" with the body "Conference: <channel display name>", a vibration pattern, the "requires interaction" flag, the grouping tag `call_<channel identifier>`, and the Decline and Accept buttons. The icon is the channel avatar for a channel, or the caller's avatar for a direct conversation. The payload is rendered once per language present among the target devices.

### Leaving and cancelling

Leaving deletes the session, or the named session. A member with no session who leaves simply has their invitations cancelled. Cancelling the invitations clears the links, broadcasts the removal and pushes a cancellation payload with the same grouping tag, so the notification disappears from the device.

A call history row is opened when the first session appears and closed when the last one leaves.

---

## 17. Handling a live chat visitor

### Deciding what the visitor sees

**Role:** the public page, on every page load.

1. Ask for the channel's information: whether it is available, the base address, the client worker version and, when available, the look settings, the welcome text, the button text, the review address and a default visitor name.
2. A channel is available when it has at least one chatbot script on a rule, or at least one available operator.
3. Match a rule against the page address and the visitor's country (see [entities.md](entities.md), section 39.2). The matched rule decides whether the button is shown, shown with a floating text, opened automatically after a delay, or hidden, and whether a chatbot answers.

### Starting a session

**Role:** the visitor.

1. Determine the operator: when the matched rule names a script and the script is allowed for this channel, the operator is the script's operator contact and the operator kind is "chatbot"; otherwise run the operator assignment (see [calculations.md](calculations.md), section 25) and the operator kind is "agent".
2. Build the session: type "livechat"; the operator contact; the live chat channel; the working status "in progress"; the failure marker "never answered" for a human or "no failure" for a bot; the current script step set to the last welcome step for a bot; one member for the operator (with the participant type, the script when it is a bot, and an unpin moment set to **now** so the session does not clutter the operator's sidebar yet); one member for the visitor, as a guest when the visitor is anonymous or as a contact when they are logged in; and a name.
3. The name is the script title for a bot, and "<visitor display name> <operator live chat name or real name>" for a human.
4. An anonymous visitor gets a Guest record created, with a name, a time zone and a country, and a cookie carrying the identifier and the access token.

### Running the script

1. Play the welcome steps.
2. At each step, post the step message as a message authored by the script's operator contact, and create a Chatbot Message linking the posted message to the step.
3. For a question step, offer the answers as buttons. When the visitor chooses one, the Chatbot Message of the question is updated with the answer, both as a link and as a plain identifier, and the client is told which answer was selected.
4. Advance to the next step by sequence whose triggering answers are all among the answers already chosen.
5. At a forwarding step, run the operator assignment with the step's skills as the requested skills.

### Forwarding to a human

On success: post the step message; add the human as a member with the participant type "agent" and the step's skills; record those skills on the session; remove the bot without a leave message; rename the session; reset the failure marker to "never answered"; broadcast the session to the new operator and pin it for them; tell the client that an operator was found.

On failure: post **nothing**, so the script can continue with fallback steps; set the failure marker to "no one available".

### During the conversation

- Every message posted by a participant increments that participant's message count, unless the message is a notification.
- The first message an operator posts stamps their response time and clears the session's failure marker to "no failure".
- A message posted by the visitor while the status is "waiting for customer" sets the status back to "in progress".
- An operator may set the status to "waiting for customer", or to "looking for help"; the latter broadcasts the session to every operator on a dedicated sub-channel so that the "needing help" list updates everywhere.
- Another operator may join a session that needs help; the join is refused when somebody was faster.
- A second human on the session makes it escalated, and therefore makes the outcome "Escalated".

### Ending

1. The visitor closes the window, or the last member leaves.
2. The visitor's call session, if any, is left.
3. The end moment is stamped and broadcast; the working status becomes empty.
4. When the session has at least one message, a notification message "Visitor left the conversation." is posted.
5. The visitor may be asked for a rating; the rating targets the operator's contact and is attached to the session. A positive rating may redirect the visitor to the channel's review address.
6. The visitor may ask for a transcript by electronic mail; the transcript is rendered in the customer's time zone when one is known.

### Restarting

A visitor may restart the script: the current step is cleared, the end moment is cleared, and the recorded chatbot messages are cleared.

### Worked example

See [calculations.md](calculations.md), section 25.

---

## 18. Running a mailing list

### Subscribing

**Role:** a visitor, or an internal user.

1. From the public page, supply an address. A confirmation electronic mail is sent carrying a signed action address.
2. Following that address subscribes the address: a Mailing Group Member is created with the address and, when a contact matches, that contact.
3. A logged-in user may join directly; joining a closed list is refused with "You can not join a closed group."
4. When the list sends guidelines automatically, the new member receives them, unless their address is banned.
5. Joining with an address that is already a member simply refreshes the stored address and contact, so a member subscribed by address alone becomes linked to their contact.

### Unsubscribing

Leaving removes the most appropriate member row: the one matching the contact when one is given, otherwise the one matching the address. A variant removes **every** member with the same normalized address, which is used when unsubscribing through the one-click header.

### Posting

**Role:** anybody the list's privacy allows.

1. A message arrives on the list address. The alias forces the thread to the list itself.
2. The privacy check runs (see [business-rules.md](business-rules.md), section 16). A closed list bounces.
3. The body is cleaned: the platform's own list footer is stripped.
4. A Message is created with the list as its record, the sender, the author, the subject, the cleaned body and the attachments; the reply address is forced to the list address; the message identifier is generated with the list tag when absent.
5. A Mailing Group Message is created wrapping it, with the parent resolved from the wrapped message's parent, and the moderation status set to pending when the list is moderated and accepted otherwise.
6. The moderation branch runs (see [state-machines.md](state-machines.md), section 9).

### Relaying

1. Build the member address map, keyed by normalized address, so each address receives exactly one copy.
2. Split into batches of the configured session size (default 500).
3. For each member other than the author: build the list headers; append the member-specific footer carrying the list address, the list page address and the member's own unsubscribe address; create an automatically deleted Outgoing Mail with the wrapped message's identifier, reply address and subject, the attachments, the member's address as the sole recipient, and the list as the related record.

### Moderating

**Role:** a moderator of the list, or an administrator.

The four decisions and their consequences are in [state-machines.md](state-machines.md), section 9. A rejection or a ban may carry a subject and a comment, which are sent to the author as an automatically deleted electronic mail whose body is the comment followed by the original body, referencing the original message identifier.

A periodic job notifies every moderator of every moderated list that has pending posts, with the subject "Messages are pending moderation".

---

## 19. Sending text messages

### From a record

**Role:** any user with the required permission on the record.

1. Open the window from the record. The mode is guessed: "post on a document" for one record, "send in batch" for several.
2. The recipient is resolved from the record (see [calculations.md](calculations.md), section 28). The window shows the recipient in words, the stored number, the number actually used, and whether it is valid.
3. Editing the number writes it back onto the recipient's record.
4. Type the body, or choose a template, which renders the body against the record in the recipient's language.
5. Send.

### In single-record mode

1. A Message of the text-message type is posted on the record, with the body converted to a readable form (addresses turned into links).
2. A Notification of the text-message channel is created for the recipient.
3. A Text Message row is created with the number, the body and a fresh identifier.
4. A Text Message Tracker links the identifier to the notification.
5. The sending job is woken.

### In batch mode

1. Render the body per record.
2. Resolve the recipient per record.
3. Apply the suppression rules in order (see [calculations.md](calculations.md), section 28): suppressed, opted out, duplicate, unformattable, missing.
4. Create one Text Message per record with the resulting state and failure type.
5. When the "keep a note" switch is on, log a note on each record.
6. When the "send directly" switch is on, send immediately; otherwise leave the queue to the job.

### The sending job

1. Select the queue: rows in the outgoing state that are not marked for deletion, locked for update, limited to the configured batch size (default 500), ordered by identifier.
2. Group by provider, then split into batches.
3. Group the batch by identical body, so that one provider call carries one body and many numbers.
4. Call the provider with the delivery-report address.
5. Map every returned state (see [state-machines.md](state-machines.md), section 7): a success state writes the mapped state and pushes it through the tracker onto the notification; a failure state writes the error state, the failure type and pushes the corresponding notification status, using the bounce status for the three destination-rejection errors.
6. Mark the rows for deletion when the caller asked for it, and re-broadcast the affected messages to their authors.
7. A maintenance routine physically deletes the rows marked for deletion; the notifications are never deleted.

### Receiving a delivery report

The provider calls back with the identifier and a state. The tracker for that identifier is found, and the state is pushed onto the notification subject to the monotonic rule.

### Resending

Selecting failed rows and resending puts them back in the outgoing state and sends them immediately, then reports how many succeeded.

### Worked example

See [calculations.md](calculations.md), section 28.

---

## 20. Shipping a digest

**Role:** the platform, daily.

1. Select the digests whose next run date has passed and whose state is activated.
2. For each, run the automatic send.
3. A delivery failure is caught and logged; the digest is left for the next run.

### The automatic send

1. Determine which digests must be slowed down (see [state-machines.md](state-machines.md), section 20).
2. For each digest and each recipient, in the recipient's language: build the body.
3. After all recipients: move the periodicity one step when the digest is being slowed down; recompute the next run date.

### Building the body

1. Compute the indicator table: one row per enabled indicator, three columns, each with a value, a margin and a column label (see [calculations.md](calculations.md), section 27).
2. Compute the preferences block: the slow-down explanation when applicable; otherwise, for a daily digest read by a configuration manager, an offer to switch to weekly; and, for any configuration manager, an offer to customize the indicators.
3. Consume one tip.
4. Render the shipped digest body with: the title, a "Connect" button pointing at the base address, the recipient's company, the recipient, the unsubscribe token, the tip count, today's date formatted as a long date, a mobile banner flag, the indicator table, the tips and the preferences.
5. Wrap the result in the shipped digest layout with the company and the recipient.
6. Create an automatically deleted Outgoing Mail with the subject "<company name>: <digest name>", the recipient's formatted address, the sender resolved in three steps, and the three unsubscribe and automatic-reply headers.

### Manual send

The same, except that the slow-down check is skipped entirely and the periodicity is never changed.

### Unsubscribing

Two paths exist: the in-application action, restricted to internal users acting on themselves; and the one-click address carrying the signed token, which removes the recipient from the digest.

---

## 21. Sending postal mail

**Role:** any user with the required permission on the source document.

### Creating

1. From the document, choose the postal sending method.
2. A Postal Letter is created per addressee: the source model and record, the addressee, the company, the report to render, the colour, cover-page and double-sided flags defaulted from the company.
3. A message of the postal type with the body "Letter sent by post with Snailmail" is posted on the source document.
4. The addressee's postal address is copied onto the letter.
5. One Notification of the postal channel is created per letter, already marked read, in the "ready" status.
6. Read access on the attachment is checked.

### Printing

1. Split the letters into those with a complete address and those without.
2. Incomplete ones are failed immediately with the missing-fields error and the explanation "The address of the recipient is not complete"; their notifications become exceptions and the message is re-broadcast.
3. For each complete one, in turn, committing after each:
   1. render the report when the letter has no attachment yet; a rendering failure fails the letter with the attachment error;
   2. build the request: the document as a stored file, the company logo, the addressee block, the sender block, and the three print options plus the currency;
   3. call the external service;
   4. on success, write the sent state, clear the error code, store "The document was correctly sent by post.<br>The tracking id is <identifier>", set the notification to delivered, and inform the operator with "Snail Mails are successfully sent";
   5. on failure, write the error state, the recognized error code or the unknown one, the explanation "An error occurred when sending the document by post.<br>Error: <message>", and set the notification to an exception with the mapped failure type; a credit failure additionally raises the operator warning "Not enough credits for Snail Mail";
   6. a service that cannot be reached fails every letter of the call with the unknown code and re-raises.
4. Re-broadcast the affected messages to their authors.

### Re-queuing and cancelling

Re-queuing writes the pending state, clears the failure fields on the notification and re-broadcasts; a single letter is then printed immediately. Cancelling writes the cancelled state, clears the error code and cancels the notification.

### Estimating

A separate call asks the service for the number of pages and the price without printing, which is what the batch-sending window uses to show the number of stamps a batch will consume.

---

## 22. Subscribing a browser to push notifications

**Role:** any user, from their browser.

1. The client asks the platform for the public half of the signing key pair.
2. The browser produces a subscription: an endpoint address and a key pair of its own, of which only the public half and an authentication secret leave the browser.
3. The client registers the subscription. A Push Device is created for the acting user's contact with the endpoint, the keys and the declared expiry.
4. The endpoint is unique; re-registering the same browser updates the existing row.
5. Unsubscribing deletes the row.
6. A device the push service reports as permanently gone is deleted automatically during a send.

---

## 23. Managing the suppression list

### Adding an address

**Role:** internal code, or a system administrator.

1. Normalize the address.
2. Create the Blacklist Entry, or unarchive an existing archived one.
3. Because the entry is a thread with two tracked fields, the change is recorded in its own conversation.

### Removing an address

**Role:** a system administrator.

1. Open the removal window from the entry. It shows the address read-only and asks for a reason.
2. Confirm. The entry is archived and the reason is logged in its conversation.

Two confirmations are shown to the user before the action: "Are you sure you want to unblacklist this Email Address?" and "Are you sure you want to unblacklist this email address?" depending on the entry point. A user without the right sees "You do not have the access right to unblacklist emails. Please contact your administrator."

### The effect

A record whose normalized address matches an active entry reports itself as suppressed. Mass sending consults that flag; single messages do not, because a person writing to one counterpart deliberately is not performing a mailing.

---

## 24. Resetting a template

**Role:** a template editor.

1. Select one or several templates.
2. Confirm. For each template that records the shipped definition file it came from, the shipped content is re-read and written back over the current content, in every installed language.
3. A template with no recorded file cannot be reset.
4. The same mechanism exists for text-message templates and reports "SMS Templates have been reset".

---

## 25. Using the electronic-mail client plugin

**Role:** an internal user, from inside their electronic-mail client.

### Authenticating

1. The plugin opens the authentication page of the platform and obtains a token bound to the user.
2. Every later call carries that token. A non-internal user is refused with "Access Error: Only Internal Users can link their inboxes to this database."

### Looking up the sender of the open message

1. The plugin sends the sender address and, optionally, the display name.
2. The platform normalizes the address and searches for a contact.
3. When none is found and enrichment is available, the domain of the address is looked up with the external enrichment service and the answer is cached on a dedicated record so the same domain is never enriched twice. A contact with an invalid address cannot be enriched — "The email of this contact is not valid and we can not enrich it". A contact that already has a parent company is not re-enriched — "The partner already has a company related to him".
4. The answer carries: the contact when found, its name, its address, its image, its company information, and the list of records of the enabled models that are linked to it.
5. When the address is one of the platform's own notification addresses, the answer says so: "This is your notification address. Search the Contact manually to link this email to a record."

### Creating a contact

The plugin may create a contact from the open message, supplying at least a contact identifier, or a name and an address — otherwise "You need to specify at least the partner_id or the name and the email".

### Logging the open message onto a record

1. The plugin sends the target model, the target record and the message content.
2. The platform posts an internal note on that record carrying the content and, optionally, the attachments.
3. The answer confirms the record so the plugin can display a link.

### Creating a record from the open message

The plugin may create a record of an enabled model from the open message; the created record is returned with its display name and a link.
