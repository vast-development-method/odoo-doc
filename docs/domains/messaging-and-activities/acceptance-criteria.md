# Messaging and Activities — Acceptance Criteria

Numbered Given, When and Then scenarios that an implementation must pass. Each is independently verifiable and uses concrete records, concrete inputs and concrete results. The identifiers in square brackets point at the rule of [business-rules.md](business-rules.md) that the scenario exercises; a reference to another document names the document and the section.

Scenario identifiers are stable. Gaps in the numbering are deliberate: an identifier is never reused.

Contents:

1. [Posting a message](#1-posting-a-message)
2. [The notification pass](#2-the-notification-pass)
3. [Notification content](#3-notification-content)
4. [Followers](#4-followers)
5. [Field change tracking](#5-field-change-tracking)
6. [Message access](#6-message-access)
7. [Editing and deleting a message](#7-editing-and-deleting-a-message)
8. [Aliases and alias domains](#8-aliases-and-alias-domains)
9. [The incoming gateway](#9-the-incoming-gateway)
10. [The outgoing queue](#10-the-outgoing-queue)
11. [Templates](#11-templates)
12. [The composer](#12-the-composer)
13. [Scheduled messages and deferred notifications](#13-scheduled-messages-and-deferred-notifications)
14. [Activities](#14-activities)
15. [Channels](#15-channels)
16. [Live chat](#16-live-chat)
17. [Text messages](#17-text-messages)
18. [Postal letters](#18-postal-letters)
19. [Ratings as live chat uses them](#19-ratings-as-live-chat-uses-them)
20. [Mailing groups](#20-mailing-groups)
21. [Presence and the event bus](#21-presence-and-the-event-bus)
22. [Retention and collection](#22-retention-and-collection)
23. [Concurrency](#23-concurrency)
24. [Multiple companies](#24-multiple-companies)
25. [Interface behaviour](#25-interface-behaviour)
26. [The assistant bot conversation](#26-the-assistant-bot-conversation)
27. [The electronic-mail client plugin](#27-the-electronic-mail-client-plugin)
28. [The publisher announcement service](#28-the-publisher-announcement-service)
29. [Recipient resolution](#29-recipient-resolution)
30. [Notification layout variants](#30-notification-layout-variants)
31. [Digests](#31-digests)
32. [Browser push](#32-browser-push)
33. [Roles, canned responses and mentions](#33-roles-canned-responses-and-mentions)
34. [Out-of-office answers](#34-out-of-office-answers)
35. [The suppression list](#35-the-suppression-list)

---

## 1. Posting a message

**AC-001 Posting on a record creates a message and notifies the followers** `[MSG-040]`
Given a thread-enabled record followed by Alice, an internal user whose notification preference is the in-application inbox, and by Bob, a customer contact with no user, both subscribed to the shipped Discussions subtype,
When Marc posts a comment with the body "Delivery confirmed for Friday" and the Discussions subtype,
Then one Message exists with author Marc, that body, the message type `comment` and the Discussions subtype;
And one Notification of the inbox channel in status "delivered" exists for Alice, unread;
And one Notification of the electronic-mail channel in status "ready" exists for Bob, already marked read;
And one Outgoing Mail exists with Bob as its recipient;
And no Notification exists for Marc.

**AC-002 Posting on the behavior itself is refused** `[MSG-040]`
When the posting operation is invoked with no record,
Then it fails with "Posting a message should be done on a business document. Use message_notify to send a notification to an user."

**AC-003 The user-specific type is refused when posting** `[MSG-041]`
When the posting operation is invoked with the message type `user_notification`,
Then it fails with "Use message_notify to send a notification to an user."

**AC-004 An unknown parameter is refused** `[MSG-042]`
When the posting operation is invoked with a parameter named `foo`,
Then it fails with "Those values are not supported when posting or notifying: foo".

**AC-005 Attachment shapes are validated** `[MSG-044]`
When the posting operation receives inline attachments as one pair instead of a list of pairs,
Then it fails with "Posting a message should receive attachments as a list of list or tuples (received ...)";
And when it receives existing attachments as write commands instead of identifiers,
Then it fails with "Posting a message should receive attachments records as a list of IDs (received ...)".

**AC-006 The author is subscribed when they comment** `[MSG-056]`
Given an internal user Marc who does not follow a record,
When Marc posts a message with the Discussions subtype and the message type `comment`,
Then Marc's contact becomes a Follower of the record.

**AC-007 The author is not subscribed when the platform posts** `[MSG-056]`
When a tracking message of the message type `notification` is produced on a record,
Then the acting user does not become a Follower.

**AC-008 A flat thread attaches a message to the ancestor** `[calculations.md section 6]`
Given a flat-threaded model whose record already carries message 10, an internal note, and message 12, a comment,
When a new message is posted with no explicit parent,
Then its parent is message 12, because a conversational message sorts before a note at equal recency.

**AC-009 An explicit parent of another record is ignored** `[calculations.md section 6]`
Given message 99 belonging to a different record,
When a message is posted with parent 99 on a flat-threaded record that already has messages,
Then the parent of the new message is the record's own ancestor, not message 99.

**AC-010 An inline image in the body becomes an attachment** `[entities.md section 2]`
When a message is posted whose body carries an image encoded inline,
Then an Attachment is created owned by the record and given an access token,
And the body references that attachment through a link carrying the token and a marker,
And the interface therefore shows the image once, not twice.

**AC-011 A portal caller may attach only its own composer files** `[MSG-053]`
Given a portal user with two attachment identifiers, one created by them on a composer and one created by somebody else,
When they post a message naming both,
Then only the first is linked to the message and the second is silently dropped.

**AC-012 The message keeps the company and the alias domain in force** `[MSG-418, MSG-419]`
Given a record of company A whose alias domain is `a.example`,
When a message is posted on it,
Then the message stores company A and that alias domain, and both stay unchanged if the record later moves to company B.

---

## 2. The notification pass

**AC-020 The author is never notified of their own message** `[MSG-063]`
Given Marc follows a record,
When Marc posts a message,
Then no Notification exists for Marc.

**AC-021 The author is notified when mentioned and the switch is on** `[MSG-063]`
Given Marc follows a record,
When Marc posts a message that names Marc among the direct recipients, with the "notify the author when mentioned" switch on,
Then a Notification exists for Marc.

**AC-022 An external follower does not receive an internal note** `[MSG-067]`
Given a record followed by Jane, a customer contact subscribed to the shipped Note subtype,
When an internal note is logged with the Note subtype,
Then no Notification exists for Jane.

**AC-023 A person already reached by the incoming message is not mailed twice** `[MSG-065]`
Given an incoming electronic mail whose "to" header contained `sales@partner.example`, stored on the message as already reached,
And a contact whose normalized address is `sales@partner.example` who follows the record,
When the gateway posts that message,
Then no Notification and no Outgoing Mail is produced for that contact.

**AC-024 A contact with no user is reached by electronic mail** `[MSG-068]`
Given a follower contact with no user,
When a message is posted,
Then the produced Notification has the electronic-mail channel.

**AC-025 An internal user beats a portal user of the same contact** `[MSG-070]`
Given a contact linked to internal user 7 whose preference is the inbox and to portal user 51,
When a message is posted to that contact,
Then the produced Notification has the inbox channel.

**AC-026 A contact receives at most one notification per message** `[MSG-072]`
Given a contact who both follows a record and is a direct recipient of a message,
When the message is posted,
Then exactly one Notification exists for that contact.

**AC-027 The message and the recipient of a notification may not change** `[MSG-073]`
When an existing Notification is written with a different message,
Then it fails with "Can not update the message or recipient of a notification."

**AC-028 Below the immediate limit the mails are sent after the commit** `[MSG-077]`
Given the immediate-sending limit is 100, the generation batch size is 50, and a record with 260 followers who are all customers reached by electronic mail,
When a message is posted with the force-send switch on,
Then six Outgoing Mails are created, five carrying 50 recipients and one carrying 10, and 260 Notifications exist;
And because six is below 100 they are sent after the transaction commits.

**AC-029 Above the immediate limit the queue takes over** `[MSG-077]`
Given the same 260 followers and an immediate-sending limit of 5,
When a message is posted with the force-send switch on,
Then the six Outgoing Mails stay in the outgoing state and are drained by the scheduled job.

**AC-030 A future scheduled moment defers the pass** `[state-machines.md section 19]`
Given the current moment is 1 April 2026 at 09:00,
When a message is posted with a scheduled moment of 1 April 2026 at 17:00,
Then one Message Notification Schedule exists carrying that moment and the serialized notification parameters,
And no Notification and no Outgoing Mail exists yet,
And the release job is woken for 17:00.

**AC-031 A past scheduled moment is ignored** `[state-machines.md section 19]`
When a message is posted with a scheduled moment already passed,
Then the notification pass runs at once and no Message Notification Schedule is created.

**AC-032 A released pass is idempotent** `[MSG-080]`
Given a Message Notification Schedule whose message already carries a Notification for Alice,
When the release runs,
Then no second Notification is created for Alice, and the schedule row is deleted.

**AC-033 A released pass whose record disappeared is discarded** `[MSG-080]`
Given a Message Notification Schedule whose target record has been deleted,
When the release runs,
Then nothing is sent and the row is deleted.

**AC-034 External addresses are listed below the limit** `[MSG-078]`
Given a message with three external recipients and a limit of fifty,
When the notification electronic mails are built,
Then the reply-to-all header lists the three formatted addresses.

**AC-035 External addresses are hidden above the limit** `[MSG-078]`
Given a message with sixty external recipients and a limit of fifty,
When the notification electronic mails are built,
Then the reply-to-all header is omitted entirely.

**AC-036 The return path is the alias domain's bounce address** `[MSG-199]`
Given a record whose alias domain has the bounce local part `bounce` on the domain `acme.example`,
When a notification electronic mail is built,
Then its return path is `bounce@acme.example`.

**AC-037 A channel message notifies only mentions and pushes** `[workflows.md section 4]`
Given a channel of the plain channel type with members Gita, Hugo and Iris, none of whom is muted, and Hugo's personal notification preference set to mentions only,
When Gita posts a comment mentioning Iris,
Then one recipient entry exists for Iris with her user's preference, no recipient entry exists for Hugo, and browser-push entries exist for the members whose preference allows it;
And every recipient is forced into the customer rendering group, so the notification electronic mail carries no access button.

---

## 3. Notification content

**AC-040 An internal user receives the access button** `[interfaces.md section 9.1]`
Given a message on a record, a recipient classified as an internal user, and the standard layout,
When the notification electronic mail is rendered,
Then it carries a button labelled "View <model description>" pointing at the record,
And the button background is the company's secondary colour and its text is the company's primary colour.

**AC-041 A customer receives no access button** `[interfaces.md section 9.1]`
Given a recipient classified as a customer,
When the notification electronic mail is rendered,
Then it carries no header row and no access button.

**AC-042 Tracking values are rendered as lines** `[calculations.md section 7]`
Given a tracking message recording a stage change from "New" to "In progress" and a priority change from empty to "High",
When the notification electronic mail is rendered,
Then it carries the lines "Stage: New → In progress" and "Priority: None → High".

**AC-043 A tracking value on a restricted field is omitted** `[MSG-091]`
Given a tracked field restricted to an access group the recipient does not belong to,
When the notification electronic mail is rendered for that recipient,
Then that line does not appear.

**AC-044 The signature is appended when asked** `[interfaces.md section 9.1]`
Given the author is an internal user with a signature and the message asks for one,
When the notification electronic mail is rendered,
Then the signature appears after the body, preceded by a separator line.

**AC-045 The preview line is capped at 190 characters** `[calculations.md section 30]`
Given a body whose plain text is 300 characters long,
When the preview is computed,
Then the result is at most 190 characters including the trailing ellipsis marker, cut on a word boundary.

**AC-046 A missing layout falls back to the raw body** `[interfaces.md section 9.2]`
Given a message naming a layout that does not exist,
When the notification electronic mail is rendered,
Then its body is the raw message body and a warning is logged.

---

## 4. Followers

**AC-050 A contact may not follow twice** `[MSG-058]`
When a Follower row is created for a contact that already follows the record,
Then it fails with "Error, a partner cannot follow twice the same object."

**AC-051 Subscribing oneself needs only read access** `[MSG-054]`
Given a user with read but not write access on a record,
When that user subscribes their own contact,
Then the Follower row is created.

**AC-052 Subscribing somebody else needs write access** `[MSG-054]`
Given the same user,
When that user subscribes another contact,
Then the operation is refused with the standard access refusal.

**AC-053 Subscribing oneself with no access fails silently** `[MSG-054]`
Given a user with no access at all on a record,
When that user subscribes their own contact,
Then the operation returns false and raises nothing.

**AC-054 A customer receives only the external default subtypes** `[MSG-061]`
Given the default subtypes of the model are Discussions, which is external, and Note, which is internal,
When a customer contact is subscribed with the default subtypes,
Then the Follower row carries only Discussions.

**AC-055 A channel may not have followers** `[MSG-060]`
When a follower is subscribed on a Channel,
Then it fails with "Adding followers on channels is not possible. Consider adding members instead."

**AC-056 Deleting a record removes its messages, followers and scheduled messages** `[MSG-062]`
Given a record with three messages, two followers and one Scheduled Message,
When the record is deleted,
Then no Message, no Follower and no Scheduled Message referring to it remains.

**AC-057 Assigning the responsible subscribes and notifies them** `[calculations.md section 10]`
Given a model whose responsible field is tracked,
When Marc sets the responsible to Alice,
Then Alice's contact becomes a Follower with the default subtypes,
And Alice receives a notification whose subject is "You have been assigned to <the record display name>".

**AC-058 Assigning oneself sends no assignment notice** `[calculations.md section 10]`
When Alice sets herself as the responsible,
Then Alice becomes a Follower and receives no assignment notification.

**AC-059 Subtype inheritance from a container record** `[calculations.md section 10]`
Given a project followed by Alice on the container subtype whose child subtype is the task-created subtype,
When a task is created in that project,
Then Alice becomes a Follower of the task carrying that child subtype.

**AC-060 A follower may edit only their own subscription** `[workflows.md section 8]`
Given Alice and Bob both follow a record and Alice is not an administrator,
When Alice unchecks a subtype on Bob's follower row,
Then the operation is refused; and when she unchecks one on her own row, the subtype is removed with the replace policy.

**AC-061 The bulk follower editor reports what it did** `[workflows.md section 8]`
When the bulk editor adds two contacts to four records with the notify switch on and a typed message,
Then eight Follower rows exist, one message is posted per record carrying the two contacts as direct recipients, and the interface reports "Followers added".

---

## 5. Field change tracking

**AC-070 A tracked field change produces a tracking value** `[MSG-081]`
Given a record whose stage field is tracked with order 10,
When the stage changes from "New" to "In progress",
Then a Message is produced carrying one Tracking Value whose old text is "New" and whose new text is "In progress", and whose old and new integer columns hold the two stage identifiers.

**AC-071 A subtype rule turns the tracking into a notification** `[MSG-086]`
Given the model returns a "Stage change" subtype for a stage change,
When the stage changes,
Then the tracking message carries that subtype and its followers are notified.

**AC-072 With no subtype the tracking is a silent note** `[MSG-086]`
Given the model returns no subtype,
When a tracked field changes,
Then an internal note is logged carrying the tracking values and nobody is notified.

**AC-073 A falsy value at creation is not a change** `[MSG-084]`
When a record is created with the tracked field left empty,
Then no tracking message is produced for it.

**AC-074 A second write in one transaction keeps the first old value** `[MSG-083]`
When a tracked field is written twice in one transaction, from "A" to "B" and then to "C",
Then one Tracking Value is produced with old value "A" and new value "C".

**AC-075 A message carrying tracking values may not be edited** `[MSG-026]`
When the content update operation is invoked on a tracking message,
Then it fails with "Messages with tracking values cannot be modified".

**AC-076 A monetary tracking value keeps its currency** `[entities.md section 6]`
When a monetary field changes from 1 200.00 to 1 450.00,
Then the Tracking Value stores 1 200.00 and 1 450.00 in the decimal columns and the currency of the field's companion field, so the rendered line shows both amounts with that currency.

**AC-077 Deleting a tracked field preserves the history** `[MSG-090]`
Given Tracking Values referring to a custom tracked field,
When that field definition is deleted,
Then the rows keep the snapshot of the field description and still render their line, and the rows become visible only to a system administrator.

**AC-078 Duration tracking sums the intervals** `[calculations.md section 9]`
Given a record created on 5 January 2026 at 09:00 in the stage "New", moved to "In progress" on 7 January at 14:00 and to "Done" on 8 January at 10:30, read on 9 January at 09:00,
Then the duration map holds 190 800 seconds for "New", 73 800 for "In progress" and 81 000 for "Done", totalling 345 600 seconds.

**AC-079 Staleness uses the threshold of the current value** `[MSG-087]`
Given the stage "In progress" declares a staleness threshold of 5 days,
When a record has been in that stage for 7 days and 4 hours,
Then its staleness day count is 7 and it is reported stale.

**AC-080 Searching staleness with an inequality is refused** `[MSG-088]`
When records are searched with a "greater than" operator on the staleness flag,
Then it fails with "For performance reasons, use "=" operators on rotting fields."

**AC-081 The display order reverses the insertion order** `[calculations.md section 8]`
Given three tracked fields with the orders 1, 1 and 30 changing at once, whose names sort as `priority`, `stage_id`, `user_id`,
When the tracking values are created,
Then they are inserted in the order priority, responsible, stage, so that reading the table newest-first yields stage, responsible, priority, which is the display order.

---

## 6. Message access

**AC-090 An author may read their own message** `[MSG-002]`
Given Marc wrote a message on a record he can no longer read,
When Marc reads the message,
Then it is returned.

**AC-091 A notified party may read the message** `[MSG-002]`
Given Alice holds an inbox Notification for a message on a record she cannot read,
When Alice reads the message,
Then it is returned.

**AC-092 A party with no link at all is refused** `[MSG-002]`
Given Bob is neither the author, nor a direct recipient, nor notified, and cannot read the record,
When Bob reads the message,
Then it fails with the five-line access refusal naming the document type "Message" and the operation "read".

**AC-093 A portal user never sees an internal note** `[MSG-001]`
Given a record shared with a portal user and an internal note on it,
When the portal user reads the record's messages,
Then the note is not returned, and the same filter is pushed into the query so the row never leaves the database.

**AC-094 A portal user may comment on a record they follow** `[MSG-003]`
Given a portal user who follows a record,
When they post a comment,
Then the Message is created.

**AC-095 Only an administrator may move a message** `[MSG-025]`
When a caller who is not an administrator writes a different record identifier on a message,
Then it fails with "Only administrators can modify 'model' and 'res_id' fields."

**AC-096 A non-internal caller cannot set the author** `[MSG-029]`
When a portal user posts a message supplying an author contact,
Then the message is created with the author resolved from their own identity and the supplied value is dropped without an error.

**AC-097 Only an administrator may export messages** `[MSG-030]`
When a caller who is not an administrator exports messages,
Then it fails with "Only administrators are allowed to export mail message".

**AC-098 A guest may not read a message of another channel** `[MSG-013]`
Given a guest whose token belongs to channel A,
When the guest reads a message of channel B,
Then it is refused.

**AC-099 Deleting a message needs write access on the record** `[MSG-005]`
Given Marc is the author of a message on a record he may only read,
When Marc deletes his own message,
Then it is refused: there is no author exception on deletion.

---

## 7. Editing and deleting a message

**AC-110 Only a comment may be edited** `[MSG-027]`
When the content update operation is invoked on a message of the type `notification`,
Then it fails with "Only messages type comment can have their content updated".

**AC-111 A channel checks only the message type** `[MSG-028]`
When the content update operation is invoked on a non-comment message of a channel,
Then it fails with "Only messages type comment can have their content updated on model 'discuss.channel'".

**AC-112 An edited marker is appended** `[MSG-033]`
When a message body is edited,
Then the stored body ends with an edited marker inside its last block-level element, or at the end when the body is plain text.

**AC-113 Editing deletes the translations** `[MSG-034]`
Given a message with two cached translations,
When its body is edited,
Then no Message Translation of that message remains.

**AC-114 Emptying a message removes its link previews** `[MSG-035]`
Given a message whose body is a single link, carrying one link preview,
When the body is cleared and the message has no attachment, no tracking value and no subtype description,
Then the link preview attachment is removed and the message counts as void; in a channel its parent link is cleared as well.

**AC-115 Deleting a message removes only its own attachments** `[MSG-031]`
Given a message with one attachment owned by the message and one owned by the record,
When the message is deleted,
Then the message-owned attachment is deleted and the record-owned one remains.

**AC-116 An edit broadcasts the changed fields** `[MSG-036]`
When a message body and subject are edited,
Then the broadcast carries the attachments sorted by identifier, the body, the direct recipients with name and avatar, the pinning moment, the modification moment, the linked-message summaries, the new subject and a marker clearing any cached translation.

---

## 8. Aliases and alias domains

**AC-130 An alias local part is unique within its domain** `[MSG-129]`
Given the alias `jobs@acme.example` on the recruitment model, owned by a job position,
When a second alias with the local part `jobs` is created on the same domain,
Then it fails with the long refusal naming the conflicting alias, the model it serves and the owner document, ending "Choose another value or change it on the other document."

**AC-131 A local part is sanitized** `[calculations.md section 1]`
When an alias is created with the local part " Créa.tion@Whatever ",
Then the stored local part is `crea.tion`.

**AC-132 A local part that sanitizes to nothing is stored empty** `[calculations.md section 1]`
When an alias is created with the local part "...",
Then the stored local part is empty and the alias displays as "Inactive Alias".

**AC-133 A local part may not be the catch-all local part** `[MSG-128]`
Given the domain's catch-all local part is `catchall`,
When an alias with the local part `catchall` is created on that domain,
Then it fails with "Aliases catchall is already used as bounce or catchall address. Please choose another alias."

**AC-134 The default values must be a literal mapping** `[MSG-127]`
When an alias is saved with the default values "not a mapping",
Then it fails with "Invalid expression, it must be a literal python dictionary definition e.g. "{'field': 'value'}"".

**AC-135 The alias domain must match the owner's company** `[MSG-131]`
Given a project of company A and an alias domain bound to company B,
When the project's alias is set to that domain,
Then it fails with the sentence naming the alias, the domain, company B and company A and ending "while the owner document belongs to company A."

**AC-136 A bounce local part is unique per domain name** `[MSG-133]`
When two Alias Domains named `acme.example` both use the bounce local part `bounce`,
Then the second fails with "Bounce emails should be unique".

**AC-137 The default sender may not be a personal relay address** `[MSG-137]`
Given a personal relay owned by Marc whose filter accepts `marc@acme.example`,
When the domain's default sender is set to `marc`,
Then it fails with "A personal mail server is using that address, you can not use it."

**AC-138 Deleting an owner deletes its alias** `[MSG-139]`
When a project owning an alias is deleted,
Then the Alias no longer exists.

**AC-139 Duplicating an owner does not copy the alias** `[MSG-140]`
When a project owning the alias `redesign` is duplicated,
Then the copy has no alias.

**AC-140 The first alias domain is propagated** `[entities.md section 23]`
Given a database with no Alias Domain, two active companies, one archived company and three aliases with no domain,
When the first Alias Domain is created,
Then all three companies, the archived one included, and all three aliases point at it.

**AC-141 Changing the policy resets the validity** `[state-machines.md section 5]`
Given an alias whose status is "valid",
When its contact-security policy is changed,
Then its status becomes "not tested".

---

## 9. The incoming gateway

**AC-150 A reply is routed to the original record** `[calculations.md section 16]`
Given a notification electronic mail was sent for invoice 42 with the message identifier `<abc-42-account.move@server>`,
When a message arrives whose in-reply-to header is that identifier,
Then a Message is posted on invoice 42 with the Discussions subtype and no record is created.

**AC-151 An alias creates a record** `[MSG-152]`
Given the alias `jobs@acme.example` creating applicants with the policy "everyone",
When a message arrives addressed to it with the subject "Application for developer",
Then one applicant is created whose display field is "Application for developer" and whose primary address field is the sender,
And a Message of the incoming type is posted on it,
And the alias status becomes "valid".

**AC-152 A followers-only alias refuses a stranger** `[MSG-156]`
Given the alias `redesign@acme.example` with the policy "followers",
When a message arrives from an address matching no contact,
Then no record is created,
And a refusal is mailed to the sender containing "Only some specific addresses are allowed to contact it.",
And the alias status is unchanged.

**AC-153 A known-authors alias refuses an unknown sender** `[MSG-155]`
Given the same alias with the policy "partners",
When a message arrives from an address matching no contact,
Then the refusal body contains "Only addresses linked to registered partners are allowed to contact it."

**AC-154 A failed creation marks the alias invalid** `[MSG-158]`
Given an alias whose default values name a field that does not exist,
When a message arrives at it,
Then the alias status becomes "invalid", that write survives the rollback of the failed creation, and the invalid-alias bounce is mailed, containing "Please try again later or contact <company name> instead."

**AC-155 A duplicate message identifier is discarded** `[MSG-146]`
Given a Message with the identifier `<x@y>` already exists,
When a message with that identifier arrives,
Then nothing is created and the event is logged.

**AC-156 Concurrent delivery of one identifier creates one message** `[MSG-147]`
When the same message is delivered twice concurrently,
Then exactly one Message exists afterwards, because the loser of the advisory lock treats it as a duplicate.

**AC-157 A bounce updates the notification and the counters** `[MSG-149]`
Given a notification electronic mail was sent to `jane@client.example` for invoice 42, and Jane's contact has a bounce counter of 2,
When a bounce arrives naming that address as the final recipient and quoting the original message identifier,
Then the Notification of that message for Jane becomes status "bounced" with the failure type "bounce" and the plain text of the bounce as the reason,
And Jane's bounce counter becomes 3, as does the counter of every other blacklist-enabled record holding the same address,
And no Message is posted anywhere.

**AC-158 A valid message resets the bounce counter** `[MSG-150]`
Given a contact whose bounce counter is 4,
When a message that is not a bounce arrives from that address,
Then the counter becomes 0.

**AC-159 Writing directly to the catch-all bounces** `[MSG-151]`
When a message arrives whose only recipient is `catchall@acme.example` and which replies to nothing,
Then no record is created and the catch-all bounce is mailed to the sender, carrying the loop-detection tag in its references.

**AC-160 A reply reaching the catch-all is routed, not bounced** `[calculations.md section 16]`
Given the same catch-all address,
When a message arrives addressed to it whose in-reply-to header matches an existing Message,
Then it is routed as a reply and no bounce is produced.

**AC-161 Creation loop detection** `[MSG-148]`
Given the loop threshold is 20 and the window is 120 minutes, and 20 applicants created from `noreply@partner.example` exist inside the window,
When another message arrives from that address at the applicant alias,
Then no applicant is created and the notification-limit bounce is mailed, carrying the loop-detection tag in its references.

**AC-162 A reply to a loop bounce is discarded** `[MSG-153]`
When a message arrives whose references contain the loop-detection tag,
Then it is discarded silently.

**AC-163 An allowed sender bypasses loop detection** `[MSG-154]`
Given `noreply@partner.example` is a Gateway Allowed Sender,
When the twenty-first message arrives from it,
Then the applicant is created normally.

**AC-164 A forward to another alias creates a record of the other model** `[MSG-159]`
Given a message that replies to a lead message but is addressed to the applicant alias,
When it is routed,
Then no Message is posted on the lead and an applicant is created instead.

**AC-165 A reply to an internal note stays internal** `[MSG-162]`
Given an internal note on a record, mailed to an internal user,
When that user replies,
Then the posted Message carries the Note subtype and the parent message's author is added to the direct recipients.

**AC-166 Two aliases in one message create two records** `[calculations.md section 16]`
When a message is addressed to both `jobs@acme.example` and `redesign@acme.example`,
Then one applicant and one task are created, each carrying its own copy of the message.

**AC-167 An unroutable message fails loudly** `[MSG-160]`
When a message arrives that matches no reply, no alias, no fallback model and no catch-all,
Then routing fails with "No possible route found for incoming message from ... to ... (Message-Id ...:). Create an appropriate mail.alias or force the destination model."

**AC-168 Carbon copies are recorded on a carbon-copy model** `[entities.md section 16.2]`
Given a model adopting the carbon-copy behavior,
When a message with two carbon copies creates a record,
Then both raw addresses are stored on the record;
And when a later message adds a third, it is appended without duplicating the first two.

**AC-169 The parsed recipients are stored as already reached** `[calculations.md section 16]`
When a message addressed to `jane@client.example`, copied to `bob@client.example` and to the alias creates a record,
Then the Message records `jane@client.example` as an already-reached recipient and `bob@client.example` as an already-reached carbon copy, and the alias address appears in neither.

**AC-170 The gateway acts as the sender's user** `[calculations.md section 16, phase H]`
Given `erik@candidate.example` is the address of an internal user's contact,
When a message from that address creates a record through an alias,
Then the record is created on behalf of that user, and the message is posted with the system identity so the real author resolves to the sender's contact.

**AC-171 A message with no identifier receives a synthetic one** `[MSG-163]`
When a message with no message-identifier header arrives,
Then a synthetic identifier of the shape `<timestamp@localhost>` is generated and the event is logged.

---

## 10. The outgoing queue

**AC-180 The queue selects only due outgoing mails** `[MSG-181]`
Given three Outgoing Mails: one outgoing with no scheduled moment, one outgoing scheduled for tomorrow, one already sent,
When the queue runs,
Then only the first is handed to the relay.

**AC-181 A connection failure fails the whole group** `[MSG-188]`
When the relay cannot be reached,
Then every Outgoing Mail of that group becomes the failure state with the failure type "connection failed",
And their Notifications become "exception" with the same failure type;
And when the caller asked for exceptions, the call aborts with "Unable to connect to SMTP Server".

**AC-182 A rejected recipient fails only that sub-message** `[MSG-187]`
Given an Outgoing Mail addressed to `good@example.test` and to `not-an-address`,
When it is sent,
Then the first recipient receives the mail, the Outgoing Mail ends in the sent state, and the Notification of the second recipient becomes "exception" with the failure type "invalid email address".

**AC-183 A mail with no recipient fails with the missing type** `[MSG-186]`
Given an Outgoing Mail with no free recipients, no contact recipients and no carbon copies,
When the queue runs,
Then it ends in the failure state with the failure type "missing email address".

**AC-184 Automatic deletion applies after a success** `[MSG-190]`
Given an Outgoing Mail with the automatic-deletion flag that is not a notification mail,
When it is sent successfully,
Then both the Outgoing Mail and its Message are deleted.

**AC-185 A notification mail keeps its message** `[MSG-191]`
Given a notification mail with the automatic-deletion flag,
When it is sent successfully,
Then the Outgoing Mail is deleted and the Message stays in the record's conversation.

**AC-186 A real failure keeps the mail** `[MSG-190]`
Given an Outgoing Mail with the automatic-deletion flag,
When sending fails with the relay-failure type,
Then the row stays in the failure state so it can be inspected and retried.

**AC-187 Retry moves only the failed rows** `[MSG-192]`
Given Outgoing Mails in the failure, sent and cancelled states,
When retry is invoked on all three,
Then only the failed one returns to the outgoing state.

**AC-188 Cancellation propagates to the notifications** `[state-machines.md section 1]`
When an Outgoing Mail is cancelled,
Then its state is cancelled and its Notifications are cancelled.

**AC-189 Mails are grouped by sending configuration** `[MSG-182]`
Given four Outgoing Mails: two with the sender `a@acme.example` on domain A, one with `b@acme.example` on domain A, one with `a@other.example` on domain B,
When the queue runs,
Then three connections are opened, one per distinct combination of relay, alias domain and transport sender.

**AC-190 A personal relay is throttled and split** `[MSG-184]`
Given a personal relay whose limit is 10 per minute and whose counter for the current minute is 7, and three mails A with 2 recipients, B with 5 and C with 1,
When the queue processes them,
Then A is sent and the counter becomes 9;
And B is split, a copy taking the first recipient and the original keeping four and losing its free addresses, the counter becoming 10, the copy sent and the original delayed;
And C is delayed;
And the delayed rows are scheduled for the next minute and the job is woken at that minute plus 59 seconds.

**AC-191 A personal relay may not be forced by another flow** `[MSG-183]`
When a flow forces a relay that has an owner,
Then it fails with "The server "<name>" cannot be forced as it belongs to a user."

**AC-192 A mail may not use another user's personal relay** `[MSG-181]`
When Marc creates an Outgoing Mail naming Alice's personal relay,
Then it fails with "You may not create a message using another user's mail server."

**AC-193 A large attachment becomes a link** `[MSG-196]`
Given an Outgoing Mail with a three-mebibyte attachment owned by a business record and a relay maximum of three mebibytes,
When the mail is prepared,
Then the attachment is replaced by a signed download link appended to the body.

**AC-194 A message-owned attachment is never converted** `[MSG-196]`
Given the same situation but the attachment is owned by the message,
When the mail is prepared,
Then the attachment is still attached, because a link to it would break when the message is deleted.

**AC-195 An attachment already shown inline is not attached again** `[MSG-194]`
Given a body carrying a link to attachment 15 and attachment 15 among the mail's attachments,
When the mail is prepared,
Then attachment 15 is not attached a second time.

**AC-196 The unfollow block is stripped for a customer** `[MSG-198]`
Given a notification mail whose body carries the unfollow placeholder,
When it is personalized for an external customer who does not follow the record,
Then the whole unfollow block is removed from the body.

**AC-197 The unfollow link is signed for an internal follower** `[MSG-198]`
Given the same mail personalized for an internal follower,
Then the placeholder is replaced by a signed unsubscription link naming the model, the record and the contact.

**AC-198 One sub-message per contact recipient** `[workflows.md section 12]`
Given an Outgoing Mail with two contact recipients and one free recipient,
When it is sent,
Then three sub-messages are produced: one for the free recipient carrying the carbon copies, and one per contact.

**AC-199 The estimate decides the link fallback** `[calculations.md section 21]`
Given headers of 800 bytes, a body of 12 000 bytes, two attachments of 9 000 000 and 11 000 000 bytes owned by the record, and a relay maximum of 25 mebibytes,
When the estimate is computed,
Then it is 26 689 706.67 bytes, which exceeds 26 214 400, so both attachments become links.

---

## 11. Templates

**AC-210 An abstract model is refused** `[MSG-097]`
When a Template is created on an abstract model,
Then it fails with "You may not define a template on an abstract model: <model>".

**AC-211 A template whose fields do not render cannot be saved** `[MSG-102]`
When a Template is saved whose subject expression raises,
Then it fails with "Oops! We couldn't save your template due to an issue." followed by the error details and "Correct it and try again."

**AC-212 The rendering restriction blocks a non-editor** `[MSG-108]`
Given the rendering restriction is on,
When a user who is not a template editor saves a Template whose body carries a conditional expression,
Then it fails with "Only members of Mail Template Editor group are allowed to edit templates containing sensible placeholders".

**AC-213 A plain field path is always allowed** `[MSG-108]`
Given the same restriction,
When the same user saves a Template whose body contains only the placeholder for the record's name,
Then it is saved.

**AC-214 Restricted rendering uses the safe evaluator** `[MSG-109]`
Given the restriction is on and a Template carrying a conditional expression,
When a caller who is not an editor renders it,
Then only plain field paths are substituted and anything else is refused as a syntax error.

**AC-215 Default recipients come from the record** `[calculations.md section 11]`
Given a Template that uses default recipients on a model whose customer is Jane,
When it is rendered for a record,
Then the produced recipients contain Jane's contact.

**AC-216 Explicit recipients override the defaults** `[calculations.md section 11]`
Given a Template whose default-recipient switch is off and whose contact list expression yields the identifiers 7 and 9,
When it is rendered,
Then the produced recipients are contacts 7 and 9, and an identifier that no longer exists is dropped silently.

**AC-217 A report is rendered per record** `[MSG-110]`
Given a Template carrying one report,
When it is sent for three records,
Then three Outgoing Mails are produced, each with its own generated document.

**AC-218 The rendered language wins** `[calculations.md section 22]`
Given a Template whose language expression yields the customer's language,
When it is sent to a French-speaking customer and a Dutch-speaking customer,
Then each mail is rendered in the corresponding language, the layout labels included.

**AC-219 An empty rendered sender is dropped** `[MSG-112]`
Given a Template whose sender expression yields nothing,
When an Outgoing Mail is produced,
Then the sender field is left unset so the platform default applies.

**AC-220 Resetting restores the shipped content** `[MSG-114]`
Given a shipped Template whose body was modified,
When it is reset,
Then its body equals the shipped body again, in every installed language;
And when the shipped definition cannot be found, the operation reports "The following email templates could not be reset because their related source files could not be found:" followed by the names.

**AC-221 An employee may not modify somebody else's template** `[MSG-113]`
Given a Template created by Alice,
When Marc, an internal user who is not a template editor, modifies it,
Then it is refused.

**AC-222 The preview renders every field** `[interfaces.md section 3]`
When a Template is previewed against a record in a chosen language,
Then the window shows the rendered subject, sender, recipient addresses, recipient contacts, carbon copy, reply address, scheduled moment, body, reports and attachments, and a broken placeholder is reported in the error field rather than raised.

---

## 12. The composer

**AC-230 Comment mode posts on each record** `[workflows.md section 5]`
Given three records selected,
When the composer sends in the "post on a document" mode,
Then three Messages are posted, one per record, each with the body rendered for that record.

**AC-231 Comment mode with no record fails** `[MSG-116]`
When the composer sends in comment mode with an empty selection,
Then it fails with "Mail composer in comment mode should run on at least one record. No records found (model <model>)."

**AC-232 Mass mode queues mails without posting** `[workflows.md section 5]`
Given three records selected in mass mode,
When the composer sends,
Then three Outgoing Mails exist and no Message appears in the records' conversations beyond the traces the composer was asked to keep.

**AC-233 A suppressed address is not mailed** `[MSG-124]`
Given a recipient whose address is on the suppression list and the exclusion switch is on,
When the composer sends in mass mode,
Then the corresponding Notification carries the failure type "blacklisted address" and nothing is handed to the relay.

**AC-234 A record with no address produces no mail** `[MSG-123]`
Given a record whose recipient has no address,
When the composer sends in mass mode without keeping logs,
Then that record is skipped entirely; and with logs kept, a failed trace is recorded instead.

**AC-235 Clearing the template resets the content** `[workflows.md section 5]`
Given a composer whose subject, body and recipients came from a Template,
When the Template is cleared,
Then the subject, the body, the recipients, the attachments, the reply address, the scheduled moment, the layout, the relay and the language return to their values before the template was chosen.

**AC-236 Scheduling needs a single record and a moment** `[MSG-118, MSG-119]`
When the composer schedules a message in batch mode,
Then it fails with "A message can only be scheduled in monocomment mode";
And when it schedules with no moment,
Then it fails with "A scheduled date is needed to schedule a message".

**AC-237 Saving as a template needs a model** `[MSG-120]`
When the composer saves its content as a Template with no model set,
Then it fails with "Template creation from composer requires a valid model."

**AC-238 A user sees only their own composer rows** `[MSG-121]`
When Marc reads the composer rows,
Then only the ones he created are returned.

**AC-239 A notification with no recipient aborts** `[MSG-117]`
Given a target model that has no conversation and a composer whose recipient list resolves to nothing,
When the composer sends,
Then it fails with "No recipient found."

---

## 13. Scheduled messages and deferred notifications

**AC-250 A scheduled message is posted later** `[state-machines.md section 19]`
Given a Scheduled Message for 10 April 2026 at 08:00 on a record,
When the posting job runs on 10 April 2026 at 08:05,
Then a Message exists on the record with the stored body, subject, attachments and direct recipients,
And the Scheduled Message no longer exists.

**AC-251 A model with no conversation is refused** `[MSG-093]`
When a message is scheduled on a model that is not thread-enabled,
Then it fails with "A message cannot be scheduled on a model that does not have a mail thread."

**AC-252 A past moment is refused** `[MSG-094]`
When a message is scheduled in the past,
Then it fails with "A Scheduled Message cannot be scheduled in the past".

**AC-253 The target may not be changed** `[MSG-023]`
When the target record of a Scheduled Message is changed,
Then it fails with "You are not allowed to change the target record of a scheduled message."

**AC-254 Posting re-checks the creator's permission** `[MSG-024]`
Given the creator lost the permission to post on the record,
When the Scheduled Message is posted,
Then it fails with "You are not allowed to send this scheduled message".

**AC-255 A failed posting still notifies the author** `[MSG-096]`
Given the posting of a Scheduled Message fails because the record was deleted,
When the job runs,
Then the creator is notified with the subject "A scheduled message could not be sent" and a body carrying the original content,
And the Scheduled Message is deleted anyway.

---

## 14. Activities

**AC-270 An activity that names a model must name a record** `[MSG-206]`
When an Activity is created with a model but no record identifier,
Then it fails with "Activities have to be linked to records with a not null res_id."

**AC-271 A free-standing activity must be assigned** `[MSG-207]`
When an Activity with no model and no assignee is created,
Then it fails with "Activities must be assigned if not attached to a document."

**AC-272 The state follows the assignee's time zone** `[MSG-428]`
Given the current moment is 31 March 2026 at 23:10 in coordinated universal time and the assignee's time zone is three hours ahead,
When an Activity has the due date 1 April 2026,
Then its state is "today" for that assignee, and "planned" for an assignee with no time zone.

**AC-273 The due date comes from the type** `[calculations.md section 13]`
Given the type "To-Do" with a delay of 5 days counted from today,
When an Activity of that type is created on 10 February 2026,
Then its due date is 15 February 2026.

**AC-274 A chained due date counts from the previous due date** `[calculations.md section 13]`
Given the type "Call" triggers the type "Send quotation" whose delay is 2 days counted from the previous due date,
When a call activity due 10 June 2026 is completed on 12 June 2026,
Then the created successor is due 12 June 2026; and had the successor counted from the completion date, it would be due 14 June 2026.

**AC-275 Month arithmetic clamps** `[calculations.md section 13]`
Given today is 31 January 2026 and a type with a delay of one month counted from today,
When an Activity of that type is scheduled,
Then its due date is 28 February 2026.

**AC-276 Completing posts a message and archives** `[MSG-212]`
When an Activity is completed with the feedback "Customer confirmed",
Then a Message is posted on the record with the Activities subtype, carrying the activity type and that feedback,
And the Activity is archived with a completion date stamped at the current moment,
And its state is "done".

**AC-277 The activity's attachments move onto the message** `[MSG-213]`
Given an Activity with two attachments,
When it is completed,
Then both attachments belong to the completion message.

**AC-278 A completion whose record vanished deletes instead of archiving** `[MSG-214]`
Given the related record was deleted,
When the Activity is completed,
Then no message is posted, the attachments are deleted, and the Activity is deleted rather than archived.

**AC-279 Cancelling posts nothing** `[MSG-215]`
When an Activity is cancelled,
Then it no longer exists and no message was posted.

**AC-280 Assigning to somebody else notifies them** `[MSG-210]`
When Marc creates an Activity assigned to Alice,
Then Alice receives a notification whose subject is ""<the record name>: <summary>" assigned to you" and whose subtitles are "Activity: <type name>" and "Deadline: <due date in Alice's date format>",
And Alice's contact becomes a Follower of the record,
And the notification is rendered in Alice's language.

**AC-281 Reassigning notifies the new assignee** `[MSG-211]`
When the assignee changes from Alice to Bob,
Then Bob receives the assignment notification and becomes a Follower.

**AC-282 Next week lands on a Monday** `[MSG-429]`
Given today is Wednesday 8 July 2026,
When the Activity is rescheduled to next week,
Then its due date is Monday 13 July 2026.

**AC-283 An assignee may read their activity without record access** `[MSG-016]`
Given Alice is assigned an Activity on a record she cannot read,
When Alice lists her activities,
Then the Activity is returned.

**AC-284 A user with no posting permission cannot create an activity** `[MSG-017]`
Given a user with read-only access on a record whose post-access attribute is write,
When they create an Activity on it,
Then it is refused with the five-line refusal naming the document type "Activity".

**AC-285 A protected type cannot be deleted** `[MSG-218]`
When the "Call" type is deleted,
Then it fails with "You cannot delete Call as it is required in various apps."

**AC-286 The generic to-do type cannot be archived** `[MSG-220]`
When the "To-Do" type is archived,
Then it fails with "The 'To-Do' activity type is used to create reminders from the top bar menu and the command palette. Consequently, it cannot be archived or deleted."

**AC-287 Deleting a type reassigns its activities** `[MSG-221]`
Given a custom type with three activities,
When the type is deleted,
Then the three activities carry the generic to-do type.

**AC-288 Suggestion and triggering are exclusive** `[MSG-222]`
When a type carrying suggestions is switched to triggering,
Then the suggestions are cleared;
And when a triggering type is switched to suggesting,
Then the triggered successor is cleared.

**AC-289 A plan creates one activity per line** `[calculations.md section 14]`
Given a plan with three lines — prepare the workstation 5 days before, welcome meeting on the day, first review 1 month after — and an anchor date of Monday 1 June 2026,
When it is launched on one record with the on-demand assignee Alice,
Then three Activities exist, due 27 May 2026, 1 June 2026 and 1 July 2026,
And a summary note listing them is logged on the record.

**AC-290 A line asking at launch with no assignee blocks** `[MSG-225]`
When the plan is launched without choosing an assignee and one line asks at launch,
Then the window reports "No responsible specified for <type name>: <summary>." and creates nothing.

**AC-291 An incompatible line type blocks the plan** `[MSG-223]`
When a line uses a type restricted to another model,
Then saving fails with "The activity type "<type name>" is not compatible with the plan "<plan name>" because it is limited to the model "<type model>"."

**AC-292 Automated activities are found by their type** `[interfaces.md section 2]`
Given business code scheduled an automated Activity of the type "Exception",
When the same code searches with that type's external identifier and the "only automated" flag,
Then the Activity is returned, and one of the same type created by hand is not.

**AC-293 Old overdue activities are purged** `[MSG-230]`
Given the retention is three years and an Activity whose due date is four years old,
When the purge routine runs,
Then the Activity is deleted, up to ten thousand rows per run;
And when the retention parameter is zero, nothing is deleted and a warning is logged;
And when it is negative, nothing is deleted and a different warning is logged.

**AC-294 The record indicator takes the worst state** `[state-machines.md section 4]`
Given a record with three live activities due yesterday, today and tomorrow,
Then the record's activity indicator is "overdue"; and once the overdue one is completed it becomes "today".

**AC-295 The counter broadcast follows the due date** `[entities.md section 17]`
Given Alice has one Activity due today,
When its due date is moved to next week,
Then a decrement is broadcast to Alice; and when it is moved back to today, an increment is broadcast.

---

## 15. Channels

**AC-310 The channel type may not change** `[MSG-231]`
When a Channel's type is written,
Then it fails with "Cannot change the channel type of: <names>".

**AC-311 A direct conversation is limited to two members** `[MSG-234]`
When a third member is added to a chat,
Then it fails with "Adding more members to this chat isn't possible; it's designed for just two people."

**AC-312 Creating a chat with three contacts is refused** `[MSG-235]`
When a chat is requested between three contacts,
Then it fails with "A chat should not be created with more than 2 persons. Create a group instead."

**AC-313 An existing chat is reused** `[workflows.md section 15]`
Given a chat already exists whose member set is exactly Alice and Bob,
When Alice requests a chat with Bob,
Then that Channel is returned and no second one is created,
And Alice's membership is pinned and her last-interest moment is refreshed.

**AC-314 A public user must join as a guest** `[MSG-239]`
When a membership is created with the public user's contact,
Then it fails with "Channel members cannot include public users."

**AC-315 Group authorization only on a channel** `[MSG-233]`
When an authorization group is set on a group conversation,
Then it fails with "For <names>, channel_type should be 'channel' to have the group-based authorization or group auto-subscription."

**AC-316 Automatic subscription adds the group's members** `[entities.md section 34]`
Given a channel whose auto-subscription group is the internal-user group and three active internal users,
When the channel is created,
Then the three contacts are members, plus the creator.

**AC-317 Joining a group posts a notice** `[MSG-246]`
When Alice invites Bob to a group,
Then a notification message is posted whose body reads "invited <Bob> to the channel";
And when Bob joins a group by himself, the body reads "joined the channel".

**AC-318 Leaving a plain channel posts nothing** `[MSG-245]`
When a member leaves a channel of the plain channel type,
Then no notice is posted;
And when a member leaves a chat or a group, the notice "left the channel" is posted.

**AC-319 A mention is limited to parties with access** `[MSG-243]`
Given a private group whose members are Alice and Bob,
When Alice posts a message mentioning Carol, who is not a member,
Then Carol is removed from the message's direct recipients.

**AC-320 A message raises the last-interest moment** `[MSG-241]`
When a comment is posted in a channel,
Then the channel's last-interest moment becomes the current moment;
And when a system notification is posted, it does not.

**AC-321 A member sees their own message as read** `[MSG-242]`
When Alice posts in a channel,
Then her membership's last seen message is that message and her separator is that identifier plus one.

**AC-322 Unread counting excludes the two notification types** `[calculations.md section 23]`
Given a channel with the messages 100 to 110, of which 103 and 107 are system notifications, and a member whose separator is 105,
Then that member's unread counter is 5, counting 105, 106, 108, 109 and 110.

**AC-323 Marking read never moves the separator backwards** `[MSG-441]`
Given a member with the separator 100,
When two calls report reading up to 118 and then up to 112,
Then the separator is 119 and the second call only re-broadcasts the member.

**AC-324 A channel unpins and re-pins itself** `[state-machines.md section 17]`
Given a member unpinned a channel on 1 February 2026 at 09:00,
When a message arrives on 3 February 2026 at 14:22,
Then the channel's last-interest moment is at or after the unpin moment, so the channel is pinned again for that member.

**AC-325 A muted member receives no push** `[workflows.md section 4]`
Given a member whose mute moment is in the future,
When a message is posted in the channel,
Then no browser-push recipient is produced for them, while their unread counter still rises.

**AC-326 The mentions-only preference** `[workflows.md section 4]`
Given a channel member whose personal preference is mentions only,
When a message that does not mention them is posted,
Then no push recipient is produced for them;
And when a message mentioning them is posted, one is.

**AC-327 Sub-thread parent rules** `[MSG-237, MSG-238]`
When a sub-thread is created whose parent is itself a sub-thread,
Then it fails with "Cannot create <names>: parent should not be a sub-channel and should be of type 'channel' or 'group'. The sub-channel should have the same type as the parent.";
And when its source message belongs to an unrelated channel,
Then it fails with "Cannot create <names>: initial message should belong to parent channel or one of its sub-channels."

**AC-328 Mentioning somebody in a sub-thread invites them** `[MSG-244]`
Given a sub-thread of a group,
When a message mentions a member of the parent whose notification preference is not "nothing",
Then that member is added to the sub-thread.

**AC-329 A guest may post only in its own channel** `[MSG-266]`
Given a guest whose token belongs to channel A,
When the guest posts in channel B,
Then the request is refused.

**AC-330 One call session per member** `[MSG-249]`
When a member who already has a session joins a call again,
Then the previous session is deleted and exactly one session exists.

**AC-331 Joining a call rings the others** `[state-machines.md section 21]`
Given a group of four members with no running call,
When one of them joins the call,
Then the three others receive a ringing invitation and a browser push titled "Incoming call" with the body "Conference: <channel display name>" and the buttons "Decline" and "Accept".

**AC-332 The call history closes with the last participant** `[entities.md section 36.2]`
When the last participant leaves a call,
Then the open Call History row receives the current moment as its end moment.

**AC-333 A lapsed heartbeat collects the session** `[MSG-460]`
Given a call session whose heartbeat stopped beyond the inactivity window,
When the collection routine runs,
Then the session is deleted and the remaining participants are told.

**AC-334 The forwarding unit threshold** `[state-machines.md section 21]`
Given the threshold is three participants and a forwarding unit is configured,
When a third participant joins,
Then the channel obtains a forwarding-unit channel identifier and address and every existing session is told to switch over;
And when the count falls back below three, the identifier and the address are cleared.

**AC-335 A message may open only one sub-thread** `[MSG-236]`
When a second sub-thread is created from the same source message,
Then it fails with "Messages can only be linked to one sub-channel".

**AC-336 A reaction is unique per party and emoji** `[MSG-253]`
When Alice reacts twice with the same emoji on the same message,
Then only one Message Reaction exists and the whole group is re-broadcast unchanged.

**AC-337 Deleting the shipped company channel is refused** `[MSG-232]`
When the general channel is deleted,
Then it fails with "You cannot delete those groups, as the Whole Company group is required by other modules."

**AC-338 A member's channel, contact and guest may never change** `[MSG-240]`
When the channel of an existing membership is written,
Then it fails with "You can not write on <field name>."

**AC-339 A direct conversation stays hidden until the first message** `[workflows.md section 15]`
When Alice opens a direct conversation with Bob for the first time,
Then Bob's membership carries an unpin moment strictly later than its last-interest moment, so the conversation does not appear in Bob's sidebar until the first message is posted.

**AC-340 The display name of a nameless group** `[calculations.md section 24]`
Given a group with five members created in the order Jonas, Kira, Liam, Mira, Noor and no name,
Then its display name is "Jonas, Kira, Liam and 2 others"; with four members "Jonas, Kira, Liam and 1 other"; with three or fewer, the plain list.

**AC-341 A bounce at the limit removes the member** `[MSG-247]`
Given a member whose bounce counter reaches ten,
When another bounce is received for that address,
Then the member is removed from the channel.

**AC-342 Inviting by address is limited by type** `[MSG-248]`
When a member of a channel that has an authorization group invites by address,
Then it fails with "Inviting by email is not allowed for this channel type (<type>).";
And when a caller who is not an internal user invites by address, it fails with "You don't have access to invite users to this channel."

**AC-343 A sub-thread pin is collected when idle** `[MSG-461]`
Given a member of a sub-thread whose own and whose channel's last-interest moments are both older than two days, with no non-notification message at or after their separator,
When the collection routine runs,
Then the member is unpinned and their client is told to close the window.

---

## 16. Live chat

**AC-350 Availability needs an operator or a chatbot** `[MSG-275]`
Given an entry point with no chatbot script and no online operator,
When a visitor asks whether chat is available,
Then the answer is no;
And when a chatbot script is configured on one of its rules, the answer is yes.

**AC-351 The session limit removes an operator** `[MSG-274]`
Given an entry point limited to two concurrent sessions per operator and an operator with two open sessions whose last-interest moments are inside the last fifteen minutes,
Then that operator is not available;
And when one of those sessions was last active twenty minutes ago, the operator is available again.

**AC-352 Blocking during a call removes an operator** `[MSG-274]`
Given an entry point that blocks assignment during calls and an operator who is in a call,
Then that operator is not available.

**AC-353 The previous operator is preferred** `[MSG-276]`
Given a returning visitor whose previous operator is online with one open conversation,
When a session is requested,
Then that operator is chosen.

**AC-354 The previous operator is skipped when busy and in a call** `[MSG-276]`
Given the previous operator is in a call and already carries two open conversations,
Then the preference ladder runs instead.

**AC-355 Language beats country** `[calculations.md section 25]`
Given the visitor speaks French and is in Belgium, and the candidates are a French speaker in France and a Dutch speaker in Belgium,
Then the French speaker is chosen.

**AC-356 Expertise refines inside a language** `[calculations.md section 25]`
Given two French-speaking candidates, one holding the requested skill and one not,
Then the one holding it is chosen.

**AC-357 A busier free operator beats a lighter one in a call** `[calculations.md section 25]`
Given Alice with four conversations and not in a call, and Dan with two conversations and in a call,
Then Alice is chosen, because the first ordering key is "fewer than two conversations or not in a call".

**AC-358 The buffer avoids two assignments inside two minutes** `[MSG-277]`
Given Bob was assigned a conversation 30 seconds ago and Carol was not, and both match the winning preference line,
Then Carol is chosen;
And when Bob is the only candidate of that line, Bob is chosen anyway.

**AC-359 A session records its participants** `[entities.md section 40.1]`
When a session is created with a chatbot,
Then two Live Chat Member Histories exist: one of the bot type carrying the script, one of the visitor type.

**AC-360 The operator's session is not pinned until the visitor writes** `[workflows.md section 17]`
When a session is created,
Then the operator's membership carries an unpin moment equal to the creation moment and a last-interest moment one second earlier, so the session does not appear in the operator's sidebar yet.

**AC-361 A non-persisted session writes nothing** `[workflows.md section 17]`
When a session is requested without persistence,
Then no Channel exists and the client receives a temporary conversation identified as −1, carrying the welcome step identifiers.

**AC-362 A live chat session must name an operator** `[MSG-279]`
When a live chat session is created with no operator contact,
Then it fails with "Livechat Operator ID is required for a channel of type livechat."

**AC-363 A closed session may not carry a working status** `[MSG-280]`
When an end moment and a working status are both written,
Then it fails with "Closed Live Chat session should not have a status."

**AC-364 Closing posts the visitor-left notice** `[MSG-289]`
Given a session with at least one message,
When the visitor leaves,
Then the end moment is stamped and a notification message with the body "Visitor left the conversation." is posted.

**AC-365 Closing an empty session posts nothing** `[MSG-289]`
Given a session with no message at all,
When the visitor leaves,
Then the end moment is stamped and nothing is posted.

**AC-366 The last operator leaving closes the session** `[state-machines.md section 11]`
Given a session whose only remaining member would be the visitor,
When the last operator leaves,
Then the end moment is stamped.

**AC-367 Escalation is detected** `[state-machines.md section 13]`
Given a session in which two different operators took part,
Then its escalation flag is true and its outcome is "Escalated".

**AC-368 A hand-over replaces the bot** `[state-machines.md section 14]`
Given a chatbot session reaching a forwarding step and an available operator,
When the step is processed,
Then the operator becomes a member of the agent type, the bot's membership is removed with no leave notice, the session's operator contact becomes the operator's contact, the failure marker becomes "never answered", and the session is renamed to include the operator's live chat display name.

**AC-369 A hand-over with no operator continues the script** `[state-machines.md section 14]`
Given no operator is available,
When the forwarding step is processed,
Then nothing at all is posted, the failure marker becomes "no one available", and the script continues to its next step, for example asking for an address.

**AC-370 An invalid address answer keeps the step** `[MSG-285]`
Given an address question step,
When the visitor answers "not an address",
Then the error naming the value as not a valid address is returned and the current step does not advance.

**AC-371 Answer conditions combine with "or" inside a step and "and" across steps** `[workflows.md section 17]`
Given step 4 is conditioned on the answers A, B, C and E, where A and B belong to step 1, C and D to step 2 and E to step 3,
When the visitor selected A, C and E,
Then step 4 is played;
And when the visitor selected B, D and E, it is not, because no answer of step 2 that appears in the condition was selected.

**AC-372 A question step must have answers** `[MSG-284]`
When a question step is saved with no answer,
Then it fails with "Step of type 'Question' must have answers."

**AC-373 Duplicating a script re-points the conditions** `[MSG-287]`
When a script with conditional steps is duplicated,
Then the copy's conditions point at the copy's own answers, matched by position, and its title ends with " (copy)".

**AC-374 A rating rolls up to the entry point** `[MSG-371]`
Given a session of the entry point "Support",
When the visitor rates it happy,
Then a Rating with the value 5 exists whose parent is that entry point, and the entry point's satisfaction over the last fourteen days includes it.

**AC-375 A country rule beats a rule without a country** `[entities.md section 39.2]`
Given one rule listing Belgium with sequence 20 and one rule with no country and sequence 10, both matching the page,
When a visitor from Belgium loads the page,
Then the Belgian rule applies.

**AC-376 A rule naming an empty script is skipped** `[MSG-288]`
Given the only matching rule names an archived script,
Then that rule does not match and the next candidate is examined.

**AC-377 The chatbot condition is honoured** `[entities.md section 39.2]`
Given a rule whose chatbot condition is "only when no operator is available" and an available operator exists,
Then the rule does not match.

**AC-378 A visitor may not start a call** `[MSG-292]`
When a visitor invokes the join-call operation,
Then it is refused.

**AC-379 An operator reads every session but writes none** `[MSG-291]`
Given a live chat operator who is not a member of a session,
When they read it, it is returned; and when they modify it, the write is refused.

**AC-380 A bot-only session is collected** `[MSG-462]`
Given a session handled only by a bot with no activity for more than a day,
When the collection routine runs,
Then the session is archived; and an empty session is deleted.

**AC-381 The response time skips the bot phase** `[calculations.md section 26]`
Given a session starting at 10:00:00, a bot line at 10:00:12 and the first operator message at 10:01:42,
Then the reported response time is 0.025000 hours.

**AC-382 The duration is reported in minutes** `[calculations.md section 26]`
Given a session from 10:00:00 to 10:09:00,
Then the reported duration is 9.00 minutes.

**AC-383 Remaining capacity** `[calculations.md section 26]`
Given an entry point allowing 5 sessions per operator with three operators, one of whom is in a call while the channel blocks assignment during calls, the other two carrying 4 and 1 ongoing sessions,
Then the remaining capacity is 5.

**AC-384 Asking for help publishes the session** `[state-machines.md section 12]`
When an operator sets the working status to "Looking for help",
Then the session is broadcast to the whole live-chat-operator group on the dedicated sub-channel;
And when another operator joins while the status is still "Looking for help", they become a member and the session leaves that list;
And when a second operator tries after the first, the join returns a refusal.

**AC-385 A restart clears the script** `[state-machines.md section 14]`
When the visitor restarts the script,
Then the current step is cleared, the end moment is cleared, the recorded chatbot messages are cleared, and the welcome steps are played again.

**AC-386 The transcript is rendered in the visitor's time zone** `[interfaces.md section 9]`
Given a session whose visitor declares a time zone three hours ahead,
When the transcript is mailed,
Then every moment in it is shown in that time zone and the subject is "Conversation with <operator display name>".

---

## 17. Text messages

**AC-400 A number is formatted to the strict international form** `[calculations.md section 28]`
Given a Belgian contact with the number "0470 12 34 56" and a record whose country is Belgium,
When the composer resolves the recipient,
Then the number used is `+32470123456`, that is the country prefix followed by the national number with no separator.

**AC-401 An unusable number blocks a single send** `[MSG-330]`
Given a single recipient whose number cannot be formatted,
When the composer sends,
Then it fails with "Invalid recipient number. Please update it."

**AC-402 A batch reports its invalid recipients** `[MSG-331]`
Given ten recipients of whom three are unusable,
When the composer sends in batch mode,
Then it reports "3 invalid recipients".

**AC-403 A suppressed number is cancelled** `[calculations.md section 28]`
Given the suppression list contains `+32470112233` and three targets: Tomas with "0470 11 22 33", Ulla with "0470/11.22.33" and Viktor with "not a number", the exclusion switch being on,
When the composer sends in batch mode,
Then Tomas's row is cancelled with the suppressed-address failure type, Ulla's row is cancelled with the same type — not as a duplicate, because the suppression test is evaluated per record — and Viktor's row is cancelled with the wrong-number-format type;
And nothing at all is handed to the provider.

**AC-404 A corrected number is written back** `[entities.md section 44.4]`
Given a single recipient whose number the sender corrects in the window,
When the message is sent,
Then the recipient's record carries the corrected number.

**AC-405 The queue groups by body** `[workflows.md section 19]`
Given twelve queued Text Messages of which nine share one body,
When the queue runs,
Then the provider call carries one entry with that body and its nine pairs of number and correlation token, plus the remaining entries.

**AC-406 Provider states map to message states** `[state-machines.md section 7]`
When the provider answers "processing", then a delivery report says "sent", then a delivery report says "delivered",
Then the Text Message passes through processing, sent-to-carrier and delivered,
And the Notification passes through processing, sent and delivered.

**AC-407 A provider failure maps to a failure type** `[state-machines.md section 7]`
When the provider answers with the insufficient-credit code,
Then the Text Message becomes the error state with the insufficient-credit failure type,
And the Notification becomes "exception" with the same failure type.

**AC-408 An unmapped provider code becomes unknown** `[MSG-334]`
When the provider answers with a code the platform does not know,
Then the failure type is the unknown one and the provider's own text is kept as the failure reason.

**AC-409 Resending a failed message** `[MSG-333]`
Given one failed Text Message that is not marked for deletion,
When resend is invoked,
Then it returns to the outgoing state, is sent again, and the user is told "1 out of the 1 selected SMS Text Messages have successfully been resent.";
And when nothing qualifies, "There are no SMS Text Messages to resend."

**AC-410 A late delivery report still applies** `[entities.md section 44.3]`
Given a Text Message that was removed after a successful send but whose Tracker remains,
When a delivery report arrives for its correlation token,
Then the Notification is updated and nothing fails.

**AC-411 The monotonic rule refuses a backward move** `[MSG-336]`
Given a Notification already in the delivered status,
When a late report carrying the sent status arrives,
Then the status stays delivered.

**AC-412 Unblocking a number requires the right** `[MSG-326]`
When a user without the settings-administrator right unblocks a number,
Then it fails with "You do not have the access right to unblacklist phone numbers. Please contact your administrator."

**AC-413 Searching a number sanitizes the term** `[MSG-328]`
Given a record storing "0470 12 34 56" whose sanitized value is `+32470123456`,
When the number search is given `+32470123456`, the record is found;
And when it is given "04", it fails with "Please enter at least 3 characters when searching a Phone number."

**AC-414 The sender name is validated** `[MSG-338]`
When a sender name of two characters is submitted,
Then it fails with "Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters."

**AC-415 The telephony account identifier is validated** `[MSG-339]`
When an account identifier that does not start with the two letters `AC` is saved,
Then it fails with "Invalid Twilio Account SID: must start with 'AC'";
And when the remainder is not made of letters and digits, "Invalid Twilio Account SID: must only contain alphanumeric characters after 'AC'".

**AC-416 A blocked number may not be created twice** `[MSG-324]`
When the same number is blocked twice,
Then it fails with "Number already exists";
And when it exists but is archived, blocking it re-activates the row instead.

**AC-417 A number that cannot be parsed is refused** `[MSG-321]`
When a number is offered whose country prefix is not a valid one,
Then it fails with "Impossible number <number>: not a valid country prefix.";
And a too-short number fails with "Impossible number <number>: not enough digits.";
And a too-long number, after the two repairs have been tried, fails with "Impossible number <number>: too many digits."

---

## 18. Postal letters

**AC-430 Creating a letter freezes the address** `[MSG-352]`
Given a contact at "1 Main Street, 1000 Brussels, Belgium",
When a Postal Letter is created for that contact,
Then the letter stores that street, postal code, city and country;
And when the contact later moves, the letter still carries the original address.

**AC-431 An incomplete address fails without calling the service** `[MSG-346]`
Given a contact with no city,
When the letter is processed,
Then its state is the error state, its error code is the missing-fields code, its explanation is "The address of the recipient is not complete",
And its Notification is "exception" with the missing-fields failure type,
And no call was made to the printing service.

**AC-432 A successful send records the tracking reference** `[state-machines.md section 8]`
When the service accepts a letter and returns the reference "PX-99812",
Then the letter's state is sent and its explanation reads "The document was correctly sent by post.<br>The tracking id is PX-99812",
And its Notification is delivered,
And the operator is informed with "Snail Mails are successfully sent".

**AC-433 An insufficient-credit failure stops the run** `[MSG-350]`
Given five pending letters and an account with no credits,
When the queue runs,
Then the first letter fails with the credit error code, an operator warning titled "Not enough credits for Snail Mail" is raised, and the run stops, leaving the remaining four pending.

**AC-434 Only retryable errors are re-attempted** `[MSG-351]`
Given letters in error with the credit code and with the no-price code,
When the queue runs after credits were bought,
Then the first is retried and the second is not.

**AC-435 Cancelling a letter cancels its notification** `[state-machines.md section 8]`
When a pending letter is cancelled,
Then its state is cancelled, its error code is cleared and its Notification is cancelled.

**AC-436 A wrong paper format is refused** `[MSG-347]`
When the report of a letter uses a paper format the service does not accept,
Then the operation fails with "Please use an A4 Paper format."

**AC-437 A document over the page limit is refused** `[MSG-348]`
When the rendered document exceeds eight pages,
Then the service refuses it with "The document to be sent exceeds the maximum allowed limit of 8 pages."

**AC-438 Creating a letter posts a message and a read notification** `[workflows.md section 21]`
When a Postal Letter is created from an invoice,
Then a Message of the postal type with the body "Letter sent by post with Snailmail" is posted on the invoice,
And one Notification of the postal channel exists, already marked read, in the ready status, so no inbox is filled.

---

## 19. Ratings as live chat uses them

**AC-450 A value out of range is refused** `[MSG-372]`
When a rating of 6 is applied,
Then it fails with "Wrong rating value. A rate should be between 0 and 5 (received 6)."

**AC-451 An unknown token is refused** `[MSG-373]`
When a rating is applied with a token that matches nothing,
Then it fails with "Invalid token or rating."

**AC-452 Applying a rating posts a message** `[MSG-374]`
When a visitor applies a rating of 5 with the comment "Perfect service",
Then the rating is marked consumed and a message is posted on the session showing the face for 5 and the comment.

**AC-453 Changing an answer updates the same message** `[MSG-374]`
Given a rating already linked to a message,
When the visitor submits a different value,
Then the same message is updated rather than a second one posted.

**AC-454 A delayed notification gives two hours** `[MSG-375]`
When a rating is applied with the delayed-notification option,
Then the notification of the rating message is deferred by two hours.

**AC-455 A request reuses an unconsumed rating** `[MSG-376]`
Given an unconsumed rating already exists for that visitor and that session,
When a new rating link is requested,
Then the same rating is reused and its token is returned.

**AC-456 Aggregates exclude unanswered requests** `[calculations.md section 33]`
Given the ratings 5, 5, 4, 3, 1, 1 and one unanswered request stored as 0,
Then the count is 6, the average is 3.17, the average grade is the neutral one and the satisfaction percentage is 50.0.

**AC-457 A record with no rating reports minus one** `[calculations.md section 33]`
Given a record with no rating at all,
Then its satisfaction percentage is −1, which the interface shows as "never rated", distinct from a percentage of zero.

**AC-458 The parent aggregate uses a hundred-point scale** `[calculations.md section 33]`
Given an entry point whose sessions average 4.20,
Then its average expressed on a hundred-point scale is 84.0.

**AC-459 Publishing an answer requires write access** `[MSG-378]`
When a user without write access on the rated record writes the public answer,
Then it fails with "Updating rating comment require write access on related record".

---

## 20. Mailing groups

**AC-470 An unmoderated list relays at once** `[state-machines.md section 9]`
Given an unmoderated list with three members,
When a message arrives at its address from a fourth address,
Then the post is accepted and three Outgoing Mails are queued, one per member.

**AC-471 The author never receives their own post** `[MSG-306]`
Given the sender is also a member,
When the post is relayed,
Then no Outgoing Mail is queued for the sender's address.

**AC-472 A moderated list holds an unknown sender** `[state-machines.md section 9]`
Given a moderated list with the automatic acknowledgement switched on,
When a message arrives from an address with no permanent rule,
Then the post is pending and an acknowledgement with the subject "Re: <original subject>" is queued to the sender, sent from the company catch-all or company address and deleted after sending.

**AC-473 An allowed sender is accepted automatically** `[state-machines.md section 9]`
Given a permanent allow rule for the sender,
When the message arrives,
Then the post is accepted and relayed with no moderation step.

**AC-474 A banned sender is rejected silently** `[state-machines.md section 9]`
Given a permanent ban rule for the sender,
When the message arrives,
Then the post is rejected and nothing is sent.

**AC-475 Allowing an author accepts their pending posts** `[state-machines.md section 9]`
Given two pending posts from one address,
When a moderator whitelists that author,
Then a permanent allow rule exists and both posts are accepted and relayed.

**AC-476 Banning an author rejects their pending posts** `[state-machines.md section 9]`
Given the same situation,
When a moderator bans the author with a comment,
Then a permanent ban rule exists, both posts are rejected, and an explanation carrying the comment followed by the original body is mailed to the author.

**AC-477 A closed list bounces** `[MSG-164]`
Given a closed list,
When a message arrives at its address,
Then nothing is stored and the closed-list bounce is sent to the sender.

**AC-478 The list headers are present** `[workflows.md section 18]`
When a post is relayed,
Then every copy carries the archive, subscribe, unsubscribe, one-click unsubscribe, precedence and automatic-answer suppression headers,
And, when the list has an address, the list identifier, the post address and the forge-to header,
And a reply carries the parent's identifier as its in-reply-to header.

**AC-479 An anonymous subscription needs a confirmation** `[MSG-308]`
When an anonymous visitor requests a subscription with an address,
Then no member is created and a confirmation carrying a signed link is sent;
And when the link is followed, the member is created.

**AC-480 A confirmation token is bound to its address** `[MSG-309]`
When a confirmation link generated for one address is replayed with another address,
Then the action is refused.

**AC-481 A moderated list needs moderators** `[MSG-297]`
When moderation is switched on with no moderator,
Then it fails with "Moderated group must have moderators."

**AC-482 A moderator must have an address** `[MSG-296]`
When a user with no address is added as a moderator,
Then it fails with "Moderators must have an email address."

**AC-483 A duplicate permanent rule is refused** `[MSG-304]`
When a second rule is created for the same address in the same list,
Then it fails with "You can create only one rule for a given email address in a group."

**AC-484 A portal user sees only accepted posts** `[MSG-310]`
Given one accepted and one pending post in a public list,
When a portal user reads the archive,
Then only the accepted post is returned.

**AC-485 Joining a closed list is refused** `[MSG-298]`
When a user joins a closed list,
Then it fails with "You can not join a closed group."

**AC-486 Guidelines are never sent to a banned address** `[MSG-302]`
Given a new member whose address carries a permanent ban rule,
When the guidelines are sent to new members,
Then that member receives nothing.

**AC-487 Moderating a post that is not pending is refused** `[MSG-303]`
When a moderator accepts an already-accepted post,
Then it fails with "This message can not be moderated";
And for several posts at once, "Those messages can not be moderated: <subjects>."

---

## 21. Presence and the event bus

**AC-500 Inactivity turns a presence away** `[state-machines.md section 16]`
Given the away threshold is 1800 seconds,
When a presence update reports an inactivity of 1801 seconds,
Then the status becomes away and the change is pushed on the presence channel.

**AC-501 Closing the connection turns a presence offline** `[state-machines.md section 16]`
When the connection of a user closes,
Then their status becomes offline and the change is pushed.

**AC-502 A manual override wins** `[state-machines.md section 16]`
Given a user who set their status to do-not-disturb,
When their detected presence would otherwise be online,
Then the displayed status is the override, and the user is excluded from live chat assignment and from call invitations.

**AC-503 Missed presences are delivered on subscribing** `[interfaces.md section 6.3]`
Given a client reconnects after an interruption,
When it subscribes,
Then it receives the current status of everybody it is entitled to see.

**AC-504 Entries survive until the retention window** `[MSG-452]`
Given the retention is one day,
When a client reconnects two hours after an interruption with the identifier of the last entry it saw,
Then it receives every entry produced since then.

**AC-505 A large announcement is split** `[calculations.md section 32]`
Given 901 touched channels producing an announcement larger than the transport limit,
When the transaction commits,
Then two or more announcements are emitted, each inside the limit.

**AC-506 Entries are written before the announcement** `[MSG-442]`
When a client polls immediately after a commit,
Then the entries announced are already readable.

**AC-507 A transient message is stored nowhere** `[workflows.md section 15]`
When the help command is used in a channel,
Then a payload is pushed to the caller alone and no Message row exists afterwards.

---

## 22. Retention and collection

**AC-520 Old notifications are deleted** `[MSG-451]`
Given the retention is 180 days and 250 read, delivered notifications of non-customer contacts older than that,
When the deletion job runs with a batch limit below 250,
Then one batch is deleted and the job reports that more remain.

**AC-521 Translations are collected** `[MSG-453]`
When the collection routine runs,
Then Message Translations older than the retention window no longer exist.

**AC-522 Composer attachments that never became a message are collected** `[MSG-458]`
Given an attachment owned by a composer with no record,
When the collection routine runs,
Then it is deleted.

**AC-523 Unused link previews are collected** `[MSG-459]`
Given a Link Preview no message refers to,
When the collection routine runs,
Then it is deleted.

**AC-524 Expired mutes are cleared** `[MSG-463]`
Given a membership whose mute moment has passed,
When the daily job runs,
Then the mute moment is cleared and the member is re-broadcast.

**AC-525 Text messages marked for deletion are removed** `[MSG-456]`
Given Text Messages marked for deletion,
When the collection routine runs,
Then the rows no longer exist and their Notifications remain.

**AC-526 Event bus entries are collected** `[MSG-452]`
Given entries older than the retention window,
When the collection routine runs,
Then they no longer exist.

---

## 23. Concurrency

**AC-540 The queue never sends twice after a rollback** `[MSG-437]`
Given an Outgoing Mail that is handed to the relay successfully but whose transaction then fails,
When the queue runs again,
Then the row is in the failure state, not the outgoing state, so it is not sent a second time without an explicit retry.

**AC-541 A bounce arriving mid-batch is not overwritten** `[MSG-438]`
Given a bounce for a message arrives while its batch is still being sent,
When the batch finishes,
Then the bounced Notification keeps the bounced status, because only notifications that are neither delivered nor cancelled and that belong to the successful recipients are rewritten.

**AC-542 Two queue runs do not deadlock** `[MSG-435]`
When two queue runs start at the same moment,
Then both take their row locks in ascending identifier order and neither deadlocks.

**AC-543 The text message queue locks its rows** `[MSG-436]`
When two queue runs start at the same moment,
Then each Text Message is handed to the provider exactly once.

**AC-544 A concurrent fetched-message write gives up** `[MSG-440]`
When two clients of one member report a fetched message at the same moment,
Then the second write skips the locked row instead of deadlocking.

**AC-545 The alias invalidation survives the rollback** `[MSG-158]`
Given a record creation through an alias that raises,
When the transaction rolls back,
Then the alias status is still "invalid", because it was written on an independent connection.

---

## 24. Multiple companies

**AC-560 The reply address follows the record's company** `[MSG-420]`
Given company A with the catch-all `catchall@a.example` and company B with `catchall@b.example`,
When a message is posted on a record of company B that has no record-specific alias,
Then its reply address is the formatted form of `catchall@b.example`.

**AC-561 The company is frozen on the message** `[MSG-418]`
Given a message posted on a record of company A,
When the record is later moved to company B,
Then the message still records company A and company A's alias domain.

**AC-562 Two companies are not mixed in one connection** `[MSG-422]`
Given queued Outgoing Mails of company A and company B with different alias domains,
When the queue runs,
Then at least two connections are opened, one per alias domain.

**AC-563 A text message uses the company of its message** `[MSG-423]`
Given a record of company B whose text-message provider differs from company A's,
When a text message is queued for it,
Then company B's provider and credentials are used.

**AC-564 A plan is offered only for its company** `[entities.md section 19.1]`
Given a plan bound to company A and a record of company B,
When the scheduling window opens on that record,
Then the plan is not offered; and a plan with no company is offered for both.

---

## 25. Interface behaviour

**AC-580 The failure badge appears only for the author** `[interfaces.md section 8.1]`
Given a message whose notification failed,
When the author opens the record, a failure badge is shown with the recipient and the reason;
And when another user opens the record, no badge is shown.

**AC-581 Cancelling failures touches only one's own** `[MSG-069]`
Given failures of two different authors on the same model,
When one author cancels the failures of one channel,
Then only their own failing notifications become cancelled.

**AC-582 An external user never sees the activity fields** `[entities.md section 20]`
When a portal user reads a record,
Then the activity fields are not returned at all.

**AC-583 A notification link falls back gracefully** `[interfaces.md section 13]`
Given a notification link to a record the reader cannot open,
When the link is followed,
Then the reader is redirected, in order of availability, to a public page, to the record with an access token, to a login page, or to an explanatory page.

**AC-584 The activity indicator respects its limit** `[configuration.md section 3]`
Given a user with more activities than the aggregation limit,
When the indicator is computed,
Then the aggregation stops at the limit and the indicator stays responsive.

**AC-585 Only noteworthy notifications reach the client** `[entities.md section 5]`
Given a message with one delivered inbox notification for an internal user and one bounced notification for a customer,
When the message is sent to the interface,
Then the bounced one is included and the delivered internal one is not, unless the subtype is marked "track recipients".

---

## 26. The assistant bot conversation

**AC-600 The conversation is created on the first sign-in** `[state-machines.md section 15]`
Given an internal user whose onboarding state is empty,
When that user opens the client for the first time,
Then a direct conversation exists between the bot contact and the user's contact,
And it carries one message authored by the bot whose three lines end with "Try to send me an emoji",
And the message is silent, so no unread counter rises,
And the user's onboarding state is the emoji step.

**AC-601 An emoji advances the guide** `[state-machines.md section 15]`
Given a user at the emoji step whose failure flag is set,
When that user posts a message containing an emoji,
Then the bot answers "Great! 👍<br>To access special commands, **start your sentence with** `/`. Try getting help.",
And the state becomes the command step,
And the failure flag is cleared.

**AC-602 A wrong gesture repeats the question and sets the flag** `[state-machines.md section 15]`
Given a user at the emoji step,
When that user posts "hello there",
Then the bot answers "Not exactly. To continue the tour, send an emoji: **type**` :)` and press enter.",
And the state stays at the emoji step,
And the failure flag is set.

**AC-603 The failure flag turns every sentence into a request for help** `[MSG-388]`
Given a user at the emoji step whose failure flag is set,
When that user posts "what now",
Then the bot answers the help sentence rather than the per-step hint.

**AC-604 The help command advances the command step** `[state-machines.md section 15]`
Given a user at the command step,
When that user runs the help command in the bot conversation,
Then the bot answers "Wow you are a natural!<br>Ping someone with @username to grab their attention. **Try to ping me using** `@OdooBot` in a sentence.",
And the state becomes the mention step.

**AC-605 Mentioning the bot advances the mention step** `[state-machines.md section 15]`
Given a user at the mention step,
When that user posts a message whose direct recipients contain the bot contact,
Then the bot answers "Yep, I am here! 🎉 <br>Now, try **sending an attachment**, like a picture of your cute dog...",
And the state becomes the attachment step.

**AC-606 An attachment creates the temporary canned response** `[state-machines.md section 15]`
Given a user at the attachment step,
When that user posts a message carrying one attachment,
Then a Canned Response is created by that user with the shortcut "Thanks" and the substitution "Thanks for your feedback. Goodbye!",
And the bot answers "Wonderful! 😇<br>Try typing `::` to use canned responses. I've created a temporary one for you.",
And the state becomes the canned-response step.

**AC-607 Using a canned response ends the guide** `[state-machines.md section 15]`
Given a user at the canned-response step who owns the temporary response,
When that user posts a message and the posting call reports that a canned response was used,
Then the temporary Canned Response no longer exists,
And the bot posts two messages, the second of which invites the user to close the conversation or to type the restart phrase,
And the state becomes idle.

**AC-608 The guide can be restarted** `[state-machines.md section 15]`
Given a user in the idle state,
When that user types the restart phrase,
Then the bot answers "To start, try to send me an emoji :)" and the state becomes the emoji step.

**AC-609 The bot stays silent outside a direct conversation** `[MSG-381]`
Given a channel that the bot contact is not a member of,
When a user mentions the bot there,
Then nothing is answered and no state changes.

**AC-610 The disabled state silences the bot** `[MSG-384]`
Given a user in the disabled state,
When that user posts anything in a conversation containing the bot,
Then nothing is posted and the state stays disabled.

**AC-611 The bot never answers itself** `[MSG-382]`
Given a bot conversation,
When a message authored by the bot contact is posted,
Then the answering logic produces nothing.

---

## 27. The electronic-mail client plugin

**AC-620 A consent code expires after three minutes** `[MSG-394]`
Given a consent code issued at 09:00:00,
When it is exchanged at 09:03:01,
Then the exchange answers the invalid-code error and no application key is created.

**AC-621 A tampered consent code is refused** `[MSG-395]`
Given a valid consent code whose payload is changed by one character,
When it is exchanged,
Then the exchange answers the invalid-code error, the signatures having been compared in constant time.

**AC-622 A valid consent code yields a one-day key** `[MSG-396]`
Given a valid consent code issued for the internal user Alice with a named add-in,
When it is exchanged at 09:01:00 on 4 March 2026,
Then an application key is created for Alice, scoped to the plugin scope, labelled with the grant name, expiring on 5 March 2026 at 09:01:00,
And the answer carries that key.

**AC-623 An external user cannot open the consent page** `[MSG-393]`
Given a portal user,
When that user opens the consent page,
Then the error page shows "Access Error: Only Internal Users can link their inboxes to this database." and no code is issued.

**AC-624 A missing key is refused** `[MSG-397]`
When a bridge route is called with no authorization header,
Then the request is refused with "Access token missing".

**AC-625 Looking up an unknown address returns a placeholder and enriches** `[MSG-401]`
Given no contact holds `sam@acme-widgets.example`, an acting user who may create contacts, and an enrichment service that answers with a company, a street, a city, a postal code and a country code,
When the bridge looks the address up,
Then the answer carries a contact with the identifier −1, the requested address and name,
And a company contact exists with those address fields and that country,
And an enrichment record exists for it whose search key is the address domain preceded by the at sign,
And the outcome reported is "company created",
And an internal note rendered from the answer is present on the company.

**AC-626 A second person at the same company reuses the enrichment** `[MSG-402]`
Given the situation of AC-625 already applied,
When the bridge looks up a second address at the same domain,
Then no new company is created and no call is made to the enrichment service.

**AC-627 A generic mailbox provider is never enriched** `[MSG-403]`
When the bridge looks up an address whose domain is a known generic mailbox provider,
Then no company is created, the outcome is "missing data", and the search key used is the whole address rather than the domain.

**AC-628 The notification address is protected** `[MSG-399]`
Given an Alias Domain whose default sender is `notifications@example.com`,
When the bridge looks that address up,
Then the answer is a contact named "Notification" carrying the platform-side explanation "This is your notification address. Search the Contact manually to link this email to a record.";
And when the bridge is asked to create a contact with that address, the request is refused.

**AC-629 Enrichment without credit reports the purchase address** `[MSG-404]`
Given an exhausted prepaid balance on the enrichment service,
When a company enrichment is requested,
Then no company is created and the outcome is "insufficient credit", carrying the address where credits can be bought.

**AC-630 Updating a company fills only empty fields** `[MSG-406]`
Given a company contact with a telephone number and no site address, and an answer carrying a different telephone number and a site address,
When the update is requested,
Then the telephone number is unchanged, the site address is filled, and the outcome is "company updated".

**AC-631 Enriching a contact that already has a parent is refused** `[MSG-407]`
Given a contact whose parent company is set,
When the create-and-link enrichment is requested,
Then it answers "The partner already has a company related to him" and nothing is created.

**AC-632 Filing a message is limited to the allowed models** `[MSG-410]`
When the bridge is asked to file a body on a record of a model that is not in the allowed list,
Then the request is refused and no Message is created.

**AC-633 Filing a message posts a real message** `[MSG-410]`
Given a contact followed by Alice,
When the bridge files the body "Signed contract attached" with one attachment,
Then a Message exists on the contact with that body and that attachment,
And Alice is notified exactly as for a message typed in the platform.

**AC-634 A company the caller may not read leaks nothing** `[MSG-411]`
Given a company contact the acting user may not read,
When it is returned as part of a lookup,
Then only its identifier and the name "No Access" are returned.

---

## 28. The publisher announcement service

**AC-640 Announcements become messages in the company-wide channel** `[MSG-413]`
Given the announcement service answers with two announcements,
When the weekly job runs,
Then two messages with the Discussions subtype exist in the company-wide channel, addressed to the root user's contact.

**AC-641 One failing announcement does not lose the others** `[MSG-414]`
Given an answer with three announcements where posting the second one fails,
When the weekly job runs,
Then the first and the third are posted.

**AC-642 A transport failure never fails the scheduler** `[MSG-415]`
Given the announcement service is unreachable,
When the weekly job runs,
Then the run reports failure without raising and the next run is scheduled normally.

**AC-643 An interactive call surfaces the transport failure** `[MSG-415]`
Given the announcement service is unreachable,
When an administrator triggers the exchange by hand,
Then it fails with "Error during communication with the publisher warranty server."

**AC-644 Subscription information is stored** `[MSG-416]`
Given an answer carrying an expiry date of 31 January 2027 and no expiry reason,
When the weekly job runs,
Then the expiry-date parameter holds that date and the expiry-reason parameter holds the trial value.

---

## 29. Recipient resolution

**AC-650 An installation address never becomes a contact** `[MSG-168]`
Given the alias domain `mydomain.example` with the catch-all local part `desk`,
And an incoming message whose "to" header carries `desk@mydomain.example` and `ann@client.example`,
When the composer proposes recipients for a reply,
Then `desk@mydomain.example` is absent from the proposal and no contact was created for it.

**AC-651 The ranking prefers an internal user's contact** `[MSG-171]`
Given contact 71 with the address `jo@client.example`, company South, no user,
And contact 92 with the same address, company North, linked to an internal user,
And a record of company South,
When that address is resolved on that record,
Then contact 92 is returned, because "is not a customer" outranks "same company".

**AC-652 The ranking prefers the acting user's own contact** `[MSG-171]`
Given two active contacts holding the same address, one of which is the acting user's contact,
When the address is resolved,
Then the acting user's contact is returned.

**AC-653 A created contact inherits the record's company and customer information** `[MSG-173]`
Given a record of company North whose customer information maps `cy@client.example` to the name "Cy Root" and the country France,
When that address is resolved with creation allowed,
Then a contact is created with that name, that address, company North and that country.

**AC-654 A malformed address is refused for creation** `[MSG-167]`
When a contact is created from the input "not an address",
Then it fails with "not an address is not recognized as a valid email. This is required to create a new customer."

**AC-655 Default recipients prefer the customer contact** `[calculations.md section 11]`
Given a record whose customer contact 55 holds `ann@client.example` and whose primary address field holds the same address,
When the default recipients are computed,
Then the result is the single contact 55 and an empty address list.

**AC-656 A customer with an unusable address is still returned as a contact** `[calculations.md section 11]`
Given a record whose only customer contact holds the unnormalizable value "not an address", also stored in the primary address field,
When the default recipients are computed,
Then the result is that contact and an empty address list, so a later failure is visible on a named contact.

**AC-657 A model that prefers addresses sends to the address** `[calculations.md section 11]`
Given a model that prioritizes addresses,
And a record whose customer contact holds `sales@client.example` while the primary address field holds `buyer@client.example`,
When the default recipients are computed,
Then the contact list is empty and the address list is `buyer@client.example`.

**AC-658 Suggested recipients combine the record, the responsible and the last discussion message** `[calculations.md section 11]`
Given a record whose responsible is Bea, whose customer contact 55 holds `ann@client.example`, followed by contact 55 and by the acting user Marc,
And a last incoming comment from `"Cy Root" <cy@client.example>` whose "to" header is `desk@mydomain.example, ann@client.example` and whose carbon-copy header is `ops@client.example`,
And `desk@mydomain.example` being an installation address,
When the suggested recipients are computed with the discussion considered and creation not allowed,
Then the proposal is exactly Bea's contact, contact 55, `Cy Root <cy@client.example>` with no contact identifier, and `ops@client.example` with no contact identifier,
And Marc is not proposed,
And no contact was created.

**AC-659 The acting user is never proposed as the responsible** `[MSG-176]`
Given a record whose responsible is the acting user,
When the suggested recipients are computed,
Then the acting user's contact is not proposed on that ground.

**AC-660 A note never triggers a reply-all proposal** `[MSG-177]`
Given a record whose only messages are internal notes and tracking messages,
When the suggested recipients are computed with the discussion considered,
Then nothing is proposed from the discussion; only the record's own customers and its responsible are proposed.

---

## 30. Notification layout variants

**AC-670 The preview line carries the first tracking value** `[interfaces.md section 9.1]`
Given a tracking message carrying the triples "Stage: New → In progress" and "Priority: None → High", and a preview line "Order confirmed",
When the notification electronic mail is rendered with the standard layout,
Then its hidden preview block reads "Stage: New → In progress |... | Order confirmed", followed by the invisible padding.

**AC-671 An internal message is marked in the preview** `[interfaces.md section 9.1]`
Given a message whose subtype is internal and whose preview line is "Check the margin",
When the notification electronic mail is rendered,
Then the hidden preview block reads "Internal communication: Check the margin".

**AC-672 The responsible-signature layout uses the record's responsible** `[interfaces.md section 9.2]`
Given a record whose responsible is Bea, whose signature is "Bea, Sales", and a message posted by Marc with the signature switch on,
When the mail is rendered with the layout that forces the responsible's signature,
Then the signature block carries "Bea, Sales" and not Marc's signature.

**AC-673 That layout renders nothing when the responsible has no signature** `[interfaces.md section 9.2]`
Given the same record where Bea has no signature,
When the mail is rendered with that layout,
Then no signature block is rendered at all, not even the author's.

**AC-674 The follower invitation layout announces the subscription** `[interfaces.md section 9.2]`
Given Marc adding Ann as a follower of an invoice with the note "Please review",
When the invitation mail is rendered,
Then its body reads "Marc (<Marc's address>) added you as a follower of this Invoice." followed by "Please review" in grey,
And the footer offers "Not interested by this? Unfollow".

**AC-675 The light layout omits the access button** `[interfaces.md section 9.2]`
When a rating request is rendered with the light layout,
Then the mail carries no header and no access button.

**AC-676 A missing layout falls back to the bare body** `[interfaces.md section 9.2]`
Given a message naming a layout that does not exist,
When the mail is produced,
Then the message body is sent unwrapped and a warning is recorded.

---

## 31. Digests

**AC-700 A weekly digest computes three columns per indicator** `[calculations.md section 27]`
Given a Digest with the weekly periodicity, one company, the messages indicator on and the connected-users indicator off, sent on Monday 8 June 2026 at 07:00 in the company's calendar time zone,
And the counts 4 and 5 for the last twenty-four hours, 143 and 110 for the last seven days, 602 and 655 for the last thirty days,
Then the three margins are −20.00, 30.00 and −8.09,
And the switched-off indicator contributes no column at all.

**AC-701 A margin against zero is zero** `[calculations.md section 27]`
Given a current value of 27 and a previous value of 0,
Then the margin is 0; and with a current value of 0 and a previous value of 20, the margin is 0 as well.

**AC-702 The slow-down moves the periodicity one step** `[state-machines.md section 20]`
Given a weekly Digest whose recipients have not signed in for seven days,
When the automatic sending runs,
Then the mail carries the line "We have noticed you did not connect these last few days. We have automatically switched your preference to monthly Digests.",
And after sending the periodicity is monthly and the next run date is today plus one month.

**AC-703 A manual send never changes the periodicity** `[MSG-364]`
Given the same digest,
When an administrator sends it by hand,
Then no slow-down check runs and the periodicity is unchanged.

**AC-704 Each sending consumes one tip** `[MSG-365]`
Given three Digest Tips, one of which the recipient already received and one whose group the recipient does not belong to,
When the digest is sent,
Then the remaining tip is rendered into the mail and the recipient is added to that tip's already-received list.

**AC-705 An unreadable indicator is dropped** `[MSG-362]`
Given a recipient who may not read the model behind one indicator,
When the digest is built for them,
Then that indicator does not appear in their copy, while it appears in another recipient's copy.

**AC-706 The one-click unsubscribe token is bound** `[MSG-366]`
Given the unsubscribe address of digest 3 for recipient 12,
When the same token is replayed for digest 4,
Then the action is refused.

**AC-707 A deactivated digest is skipped** `[MSG-361]`
Given a Digest in the deactivated state whose next run date has passed,
When the daily job runs,
Then nothing is sent and the next run date is unchanged.

**AC-708 The messages indicator is not company scoped** `[calculations.md section 27]`
Given two companies each producing conversational messages in the window,
When the messages indicator is computed for the digest of one company,
Then the count covers the messages of both companies.

---

## 32. Browser push

**AC-720 A subscription registers a device** `[workflows.md section 22]`
When a browser registers with an endpoint, a public key and an authentication secret,
Then one Push Device exists for the acting user's contact carrying all three and the declared expiry;
And re-registering the same endpoint updates the existing row rather than creating a second one.

**AC-721 Below five devices the payloads are pushed directly** `[workflows.md section 4]`
Given a recipient with three registered devices,
When a comment is posted,
Then three direct pushes are attempted and no Push Notification row is created.

**AC-722 From five devices the payloads are queued** `[workflows.md section 4]`
Given recipients with seven registered devices in total,
When a comment is posted,
Then seven Push Notification rows are created and the delivery job is woken.

**AC-723 A gone device is deleted** `[workflows.md section 22]`
When the push service reports an endpoint as permanently gone,
Then the Push Device is deleted and the remaining pushes continue.

**AC-724 The payload body is truncated without breaking an escape** `[calculations.md section 29]`
Given a body of "BØDY" whose serialized form is nine characters and a maximum that requires removing three,
When the payload is truncated,
Then the cut at six characters does not decode, so the cut moves back to just before the escape marker and the body becomes "B".

**AC-725 The push body falls back to the attachment names** `[workflows.md section 4]`
Given a message with an empty body and three attachments, one of which is a voice recording,
When the push payload is built,
Then the body reads the first attachment name, then " and 2 other attachments"; and a voice recording is named "Voice Message".

---

## 33. Roles, canned responses and mentions

**AC-740 A role name is unique** `[MSG-254]`
When a second Role is created with an existing name,
Then it fails with "A role with the same name already exists."

**AC-741 Mentioning a role notifies its members** `[MSG-255]`
Given a Role with three users, one of whom cannot access the conversation,
When the role is mentioned in a message,
Then the two who can access it become direct recipients and the third is dropped silently.

**AC-742 A canned response is shared only with groups the creator belongs to** `[MSG-256]`
When a user shares a Canned Response with a group they do not belong to,
Then the group is not offered in the selection.

**AC-743 Using a canned response updates its last-used moment** `[entities.md section 14]`
When a Canned Response is inserted in a composer,
Then its last-used moment becomes the current moment, so it rises in the proposal order.

**AC-744 A mention of everyone expands to the members** `[MSG-243]`
When a message in a group of five mentions everyone,
Then the five member contacts become direct recipients.

---

## 34. Out-of-office answers

**AC-760 An absent recipient answers once** `[workflows.md section 4]`
Given Alice is inside her absence window with a non-empty absence message and is a direct recipient of a comment written by Marc,
When the comment is posted,
Then one message of the out-of-office type is posted on the record, authored by Alice, addressed to Marc, with the subject "Auto: <original subject or the record display name>",
And its headers mark it as an automatic reply and suppress further automatic answers.

**AC-761 The same correspondent is not answered twice in four days** `[workflows.md section 4]`
Given Alice already produced an out-of-office answer to Marc two days ago,
When Marc posts again,
Then no second answer is produced.

**AC-762 The record's responsible may answer even without being a recipient** `[workflows.md section 4]`
Given the record's responsible is absent and is not among the direct recipients,
When a comment is posted by somebody else,
Then the responsible's absence answer is produced.

**AC-763 An out-of-office answer is never produced for a system message** `[workflows.md section 4]`
When a tracking message is produced on the record,
Then no absence answer is produced, because the message type is neither a comment nor an incoming electronic mail.

---

## 35. The suppression list

**AC-780 Adding an address is traceable** `[MSG-322]`
When an address is added to the suppression list,
Then a Blacklist Entry exists with the normalized address, and its own conversation records the change because both its fields are tracked.

**AC-781 Removing an address archives it** `[MSG-323]`
When an administrator removes an address with the reason "customer asked to be contacted again",
Then the entry is archived rather than deleted and the reason is logged in its conversation.

**AC-782 A duplicate address is refused** `[MSG-322]`
When the same address is added twice,
Then it fails with "Email address already exists!"; and when the existing row is archived, adding it re-activates that row.

**AC-783 A single deliberate message ignores the suppression list** `[entities.md section 12]`
Given a suppressed address,
When a user writes one message to that address from a record,
Then the message is sent, because only mass sending consults the suppression list.

**AC-784 Removing an address needs the right** `[MSG-325]`
When a user without the right removes an address,
Then it fails with "You do not have the access right to unblacklist emails. Please contact your administrator."

---

## Reconciliation notes

1. **The preview length in AC-045.** One source version asserted 100 characters and the ellipsis. The observable behaviour caps at 190 characters including the marker; the scenario states 190. See [calculations.md](calculations.md), section 30, where the discrepancy is recorded as a **compatibility finding**.
2. **The unread counter in AC-322.** One source version excluded the member's own messages from the counter; the counter in fact excludes the two notification message types and counts everything else at or above the separator, the author's own messages included. The author never sees their own message as unread only because the posting hook moves their separator past it. The scenario states the observable rule.
3. **The channel display name in AC-340.** One source version joined every other member's name; the display name uses the first three members by identifier and appends "1 other" or "<n> others" above three. The scenario states the observable rule.
4. **Rating scenarios.** Ratings are owned by [../learning-surveys-and-gamification/](../learning-surveys-and-gamification/). Section 19 keeps only the scenarios that a live chat implementation must pass, because live chat is the caller specified here.
