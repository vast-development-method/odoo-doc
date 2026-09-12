# Background workers

Work that must not run inside the transaction that decided on it is written to a queue and drained later by a scheduled job. The business transaction writes one queued record and commits; a worker picks that record up, performs the slow or externally visible part, and commits again. This document specifies the shape every such queue shares, the properties a queue entity must have for a worker to be safely re-run, and each queue the platform ships apart from the two electronic-mail directions, which are specified in [`mail-gateway.md`](mail-gateway.md).

The scheduler that runs these workers, its acquisition lock, its inner loop, its progress protocol, its time budget and its failure counting are specified in [`scheduled-jobs.md`](scheduled-jobs.md). The transaction rules the workers depend on, in particular the batch-commit rule and the row-locking operations, are in [`transactions-and-concurrency.md`](transactions-and-concurrency.md).

## 1. The catalogue of queues

| Queue | Entity | Drained by | Shipped interval | Batch | Commits | Specified in |
|---|---|---|---|---|---|---|
| Outgoing electronic mail | Outgoing Email | Outgoing email queue manager | 1 hour | 1 000 | after each message | [`mail-gateway.md`](mail-gateway.md), section 3 |
| Incoming electronic mail | Incoming Mail Server | Incoming mail fetching | 5 minutes, inactive as shipped | 50 messages per server | after each message and after each server | [`mail-gateway.md`](mail-gateway.md), section 7 |
| Deferred notifications | Scheduled Messages | Notify scheduled messages | 1 hour, plus one trigger per distinct scheduled instant | every record that is due | once, at the end of the job | section 4.1 |
| User-scheduled messages | Scheduled Message | Post scheduled messages | 1 day, plus one trigger per distinct scheduled instant | 50 | after each message, plus a self-trigger while any remain | section 4.2 |
| Outgoing text messages | Outgoing Text Message | Text message queue manager | 24 hours, plus a trigger on every creation | 500 | once per iteration, through the progress report | section 5 |
| Postal letters | Postal Letter | Postal letter queue processing | 24 hours | every pending and every recoverable letter | after each letter | section 6.1 |
| Browser push notifications | Push Notification | Send push notifications | 1 day, plus a self-trigger while any remain | 50 | once, at the end of the job | section 6.2 |
| Digests | Digest Email | Digest email sending | 1 day, first execution two hours after installation | every digest that is due | once, at the end of the job | section 7 |
| Data recycling | Recycling Model and Recycling Record | Data recycling: clean records | 1 day, first execution at 03:00 | 5 000 records in automatic mode, 50 000 in manual mode | after each batch | section 8 |
| Time-based automation | Automation Rule | Automation rules: check and execute | 4 hours, inactive as shipped | every rule, every due record of a rule | after each rule, through the progress report | section 9 |
| Marketing campaigns | Mass Mailing | Marketing campaign queue | 1 day | one campaign per iteration | after each campaign, through the progress report | section 10 |

The three entities named in the last column but not owned by this folder are described in the domains that own them: the outgoing message and the discussion thread in [`../domains/messaging-and-activities/`](../domains/messaging-and-activities/), the mass mailing in [`../domains/website-and-storefront/`](../domains/website-and-storefront/) for its storefront-facing part. This document specifies only their queue behaviour.

## 2. The shape of a queue

### 2.1 The five parts

Every queue in the catalogue is built from the same five parts.

| Part | Obligation |
|---|---|
| A queued record | One row per unit of work, created by the business transaction and committed with it. The row is the only durable evidence that the work is owed. |
| A selection condition | A condition over the queued records that identifies the work still owed. It is evaluated at the start of every iteration, never cached across a commit. |
| A batch limit | An upper bound on the number of records one iteration takes, so that one iteration fits inside the scheduler's time budget. |
| A durable outcome | A change to the queued record, or its deletion, that removes it from the selection condition. It is committed before the worker moves on. |
| A resumption rule | Either a self-trigger while records remain, or the reliance on the next interval. |

A queue that lacks the durable outcome re-does its work after every interruption. A queue that lacks the batch limit is interrupted by the time budget before it commits anything. Both defects are observable as duplicate side effects or as a queue that never drains.

### 2.2 Where the commits fall

Three commit placements are used, and each has a different consequence when the worker is interrupted.

1. **After each item.** The outgoing mail queue, the postal letter queue and the user-scheduled message queue commit after every single item. An interruption loses at most the item in flight. This is the placement used whenever one item causes an irreversible external effect, because it bounds the number of effects that can be repeated to one.
2. **After each batch, through the progress report.** The text message queue, the time-based automation job and the marketing campaign queue call the progress operation of [`scheduled-jobs.md`](scheduled-jobs.md), section 6.5, which commits as part of reporting. The commit and the report are then inseparable: no batch can be reported without being durable, and none can be durable without being reported.
3. **Once, at the end.** The deferred notification queue, the browser push queue and the digest queue commit only when the job body returns. An interruption loses the whole run, which is acceptable for these three because either the work is idempotent by construction or the queued row survives to be retried.

### 2.3 The selection is re-evaluated, never carried

A worker must not carry a set of record keys across a commit and assume the records still qualify. Another worker, an interactive user or a rule may have changed them. Every queue in this document re-reads its selection condition at the start of each iteration, and the ones that lock take their lock inside the same statement that selects.

### 2.4 Ordering

Where the order of processing is observable, it is stated with the queue. Where it is not stated, the order is the entity's default order. Two queues state an explicit order: the outgoing mail queue and the text message queue both process in ascending key order, which makes the oldest queued item the first sent.

## 3. Idempotency, retry and resumption

This section is the catalogue of the patterns that make a batch-committing worker safe to re-run. It is the reference for the batch-commit rule of [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 10.

### 3.1 The four patterns

| Pattern | How the work is claimed | What makes the re-run safe | Used by |
|---|---|---|---|
| **State advance** | The record's state field is written to a non-selectable value before the external effect, and to its final value after. | The selection condition excludes the intermediate value, so a re-run never picks the record up again. | Outgoing electronic mail, postal letters |
| **Lock and mark** | The candidate rows are locked in the selecting statement with the non-blocking form; rows another worker holds are skipped. | Two workers can never hold the same row, so the external call happens once per commit boundary. | Outgoing text messages |
| **Delete on completion** | The queued row is deleted in the same transaction that performed the work. | The row is the only evidence of the debt; deleting it discharges it exactly once. | Deferred notifications, browser push notifications, user-scheduled messages |
| **Watermark** | The worker records the instant of its last successful pass and selects on a half-open interval anchored on it. | A record inside the interval is selected exactly once, because the next pass starts where this one ended. | Time-based automation, digests (whose watermark is the next mailing date) |

### 3.2 The state-advance pattern in detail

The state-advance pattern is the only one of the four that survives a rollback with the correct outcome, and it does so by writing the *failure* state before the attempt rather than after it.

1. Select the records whose state is the queued value.
2. Write the exception state on the record and commit. The record is now excluded from the selection condition.
3. Perform the external effect.
4. Write the final state, either the success value or the exception value with the failure reason, and commit.

A crash between steps 2 and 4 leaves the record in the exception state, never in the queued state. The consequence is deliberate: an interrupted item is reported as failed and waits for a human or an explicit retry, rather than being sent twice. The outgoing mail queue implements exactly this sequence, and its acceptance criteria assert it.

### 3.3 The lock-and-mark pattern in detail

1. Select the candidate rows with the batch limit and the ordering, requesting an exclusive row lock in the non-blocking form.
2. Rows locked by another transaction are dropped from the result rather than waited for.
3. Perform the work on the surviving rows.
4. Commit, which releases the locks.

The non-blocking form is what makes two concurrent workers share the queue instead of serialising on it: each takes the rows the other did not. It is specified in [`transactions-and-concurrency.md`](transactions-and-concurrency.md), section 7.

### 3.4 What is not idempotent

Two effects in this document are not idempotent and cannot be made so from inside the platform.

- **The text message send.** The external service is contacted inside the transaction that will mark the messages as sent. A serialization failure that triggers the retry loop re-sends the batch. This is a **compatibility finding**: a replacement should either accept the duplicate risk, which the original does, or pass a per-message idempotency key to the external service so that the second call is recognised and discarded. The queued record already carries a per-message universally unique identifier, reproduced under the field name `uuid`, which is the natural idempotency key.
- **The postal letter print.** The letter is submitted to the printing service and the outcome is written afterwards; a crash between the two re-submits the letter on the next run. The run therefore commits after every single letter, which bounds the exposure to one letter.

### 3.5 Resumption

A worker that could not finish resumes in one of three ways.

| Resumption | Behaviour | Used by |
|---|---|---|
| Self-trigger | The body counts the records still selectable and, if any remain, requests an immediate new execution of its own job. The queue drains at machine speed rather than at the interval. | User-scheduled messages, browser push notifications |
| Progress report | The body reports a positive remaining count, which makes the scheduler's inner loop iterate again within the same acquisition until the time budget is exhausted. | Outgoing text messages, marketing campaigns, time-based automation |
| Next interval | Nothing is done; the remaining records are taken by the next scheduled execution. | Deferred notifications, postal letters, digests, data recycling |

The self-trigger and the progress report are not alternatives to each other: the progress report keeps one acquisition working, and the self-trigger schedules a fresh acquisition. A queue that must not hold a worker for the whole time budget, because each item is slow and externally visible, uses the self-trigger.

## 4. Deferred notifications and user-scheduled messages

### 4.1 Deferred notifications

A message may be posted now and notified later. The deferral is a **Scheduled Messages** record.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `mail_message_id` | Message | Many-to-one to Message, cascading deletion | yes | The message whose notification is deferred. |
| `notification_parameters` | Notification Parameter | Long text | no | The serialized arguments to replay when notifying. |
| `scheduled_datetime` | Scheduled Send Date | Date and time | yes | The instant at which the notification is to be sent. |

Default ordering is by scheduled instant descending, then by key descending. The display name of a deferral is the message it defers.

Creating deferrals triggers the notification job once for each **distinct** scheduled instant present in the created set. A deferral to a precise instant is therefore honoured to the minute rather than waiting for the next hourly execution. Changing the instant of existing deferrals writes the new instant on all of them and creates one trigger for that new instant; the trigger created for the old instant is left in place and finds nothing to do.

Draining, in order:

1. Select every deferral whose scheduled instant is at or before the current instant. If the set is empty, stop.
2. Log the informational line `Send <n> scheduled messages`, in which the placeholder is the number of deferrals selected.
3. Group the deferrals by the entity named on their message. Deferrals whose message names no entity, or names an entity but no record, form one group of their own that is notified against the abstract discussion thread rather than against a record.
4. For each group, read the referenced records in one operation and determine which of them still exist. Deferrals whose record has been deleted are skipped: no notification is produced for them.
5. For each surviving pair of record and deferral, build the notification arguments. Start from a single default: skip recipients who have already been notified. Then overlay the deserialized notification parameters, first removing the scheduled instant from them so that the replayed call does not defer itself again. A payload that cannot be deserialized leaves the defaults untouched and is not an error.
6. Notify the thread of the record with the message and those arguments.
7. Delete every deferral of the set, including the skipped ones, and let the job's single commit make the deletion durable.

Step 7 is the delete-on-completion pattern of section 3.1: a deferral that was skipped because its record disappeared is discharged all the same, because there is nothing left to notify.

The same procedure is available on demand for a given set of messages, and as a forced send that ignores the scheduled instant entirely.

### 4.2 User-scheduled messages

A **Scheduled Message** is a message a person composed and asked to be posted later. It is a different entity from the deferral above: the deferral posts now and notifies later, while this one posts later.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `subject` | Subject | Text | no | The subject of the message to post. |
| `body` | Contents | Rich text, with style sanitizing | no | The body of the message to post. |
| `scheduled_date` | Scheduled Date | Date and time | yes | The instant at which the message is to be posted. |
| `attachment_ids` | Attachments | Many-to-many to Attachment | no | The files to post with the message. |
| `composition_comment_option` | Comment Options | Selection: `reply_all`, `forward` | no | The composition mode the form presents. |
| `model` | Related Document Model | Text | yes | The transport name of the entity the message will be posted on. |
| `res_id` | Related Document Id | Reference key to that entity | yes | The record the message will be posted on. |
| `author_id` | Author | Many-to-one to Contact | yes | The author the posted message will carry. |
| `partner_ids` | Recipients | Many-to-many to Contact | no | The explicit recipients. |
| `is_note` | Is a note | Boolean, default false | no | Whether the message is posted as an internal note rather than as a comment. |
| `notification_parameters` | Notification parameters | Long text | no | The serialized arguments to replay when posting. |
| `send_context` | Sending Context | Structured value | no | The execution context to apply when posting, which lets the post drive a further action. |

Two validations apply.

- A message may not be scheduled on an entity that does not carry a discussion thread. The refusal reads `A message cannot be scheduled on a model that does not have a mail thread.`
- A message may not be scheduled in the past. The refusal reads `A Scheduled Message cannot be scheduled in the past`.

Changing the target record of an existing scheduled message is refused with `You are not allowed to change the target record of a scheduled message.`

Reading is restricted to the records the reader may post on: the search reads the entity and record key of each candidate row directly, groups the keys by entity, and keeps only the rows whose record passes the posting permission of that entity, which is the write permission unless the entity declares another one. Creating, changing and deleting a scheduled message all re-run that same check.

Creation transfers to the scheduled message every attachment that the composer had attached to itself and that the acting user created, and triggers the posting job once per distinct scheduled instant. Changing the instant triggers the job for the new instant.

Draining:

1. Select at most 50 scheduled messages whose instant is at or before the current instant, and log `Posting <n> scheduled messages`.
2. For each of them, in order, act as its creator, not as the scheduler user. Re-check that the creator may still post on the target record.
3. Post the message on the target record with its subject, body, author, recipients, attachments and the note or comment classification, applying the sending context and the subset of the notification parameters that the whitelist admits. The whitelist is exactly: whether to append the author's signature, the sender address, the layout to wrap the body in, the forced language of the notification, the activity type, whether the notification is deleted after sending, the outgoing mail server, the reply-to address, whether the reply-to forces a new thread, the message type and the subtype.
4. Commit after each posted message.
5. If posting raised, roll back, log the line "Posting of scheduled message with ID <key> failed" with the exception, and notify the creator with the subject `A scheduled message could not be sent` and the body `The message scheduled on <model>(<id>) with the following content could not be sent:` followed by the original body between two horizontal separators. The placeholders are the transport name of the target entity, the key of the target record and the body of the message. Commit that notification.
6. If even the failure notification could not be sent, log the exception and roll back, so that the scheduled message is still deleted at step 7.
7. Delete every scheduled message of the selection, whether it was posted or not.
8. If any scheduled message is still due, trigger the job again immediately.

Steps 5 to 7 are the reason the queue cannot loop on a poisoned message: a message that cannot be posted is reported to its author and removed.

Posting the message creates the ordinary notifications, and those notifications may themselves be deferred by the mechanism of section 4.1.

## 5. Outgoing text messages

### 5.1 The entity

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `uuid` | Universally unique identifier | Text, generated at creation, not copied | The alternate identity of the message, used to match delivery reports returned by the service. Unique. |
| `number` | Number | Text | The destination number. |
| `body` | Body | Long text | The text sent. |
| `partner_id` | Customer | Many-to-one to Contact | The recipient contact, when known. |
| `mail_message_id` | Message | Many-to-one to Message | The thread message this text message belongs to. |
| `state` | Status | Selection, default `outgoing`, read-only | The queue state. |
| `failure_type` | Failure type | Selection | The classification of a failure. |
| `to_delete` | Marked for deletion | Boolean, default false | Marks the row for removal by the cleanup, without removing its notification. |

The states are:

| Stored value | Label | Meaning |
|---|---|---|
| `outgoing` | In Queue | Waiting to be sent. |
| `process` | Processing | Accepted by the service and being processed by it. |
| `pending` | Sent | Handed to the service, delivery not yet confirmed. |
| `sent` | Delivered | Delivery confirmed by the service. |
| `error` | Error | The attempt failed; the classification is in the failure field. |
| `canceled` | Cancelled | Withdrawn before sending. |

The failure classifications are `unknown` (unknown error), `sms_number_missing` (missing number), `sms_number_format` (wrong number format), `sms_country_not_supported` (country not supported), `sms_registration_needed` (country-specific registration required), `sms_credit` (insufficient credit), `sms_server` (server error), `sms_acc` (unregistered account), and three that only the mass-sending path produces: `sms_blacklist` (the number is on the blocked list), `sms_duplicate` (the number appears twice in one campaign) and `sms_optout` (the recipient opted out).

Three of the failure classifications are treated as bounces, meaning that the destination is unreachable rather than that the attempt went wrong: `sms_invalid_destination`, `sms_not_allowed` and `sms_rejected`. Together with `sms_expired` and `sms_not_delivered` they form the set of delivery errors reported asynchronously rather than at send time.

Uniqueness: the identifier field is unique, and the violation message is `UUID must be unique`.

Creating any text message triggers the queue manager immediately, so a message queued by an interactive action is normally sent within the minute rather than at the next daily execution.

### 5.2 Draining

1. Build the selection condition: state is `outgoing` and the deletion mark is not set.
2. Read the batch size from the system parameter `sms.session.batch.size`, the text-message session batch size, defaulting to 500.
3. Select at most that many matching records in ascending key order, requesting an exclusive row lock in the non-blocking form, and keep only the rows the lock was granted on.
4. If nothing is left, stop.
5. Group the surviving records by the service that must carry them, and send each group.
6. Report progress: the processed count is the number of records selected; the remaining count is a full count of the selection condition when the selection filled the batch, and zero otherwise. The report commits.

Step 6 is what makes the queue drain more than one batch per execution: a full batch means more work is owed, the scheduler's inner loop sees a positive remaining count and iterates again.

### 5.3 Sending one group

1. Build one payload per distinct body, carrying the destination number and the identifier of every record that shares that body. Identical bodies are therefore sent as one request with many destinations.
2. Attach the address at which the service is to report deliveries, which is the deployment's own address followed by the route path `/sms/status`.
3. Call the service. On any failure of the call itself, log the batch and treat every record of the batch as having returned the server-error outcome; re-raise instead only when the caller asked for exceptions to propagate.
4. Match the returned outcomes back to records by the identifier, not by position.
5. Group the outcomes by returned state and failure reason, and apply them: a returned processing state becomes `process`, a returned success or sent state becomes `pending`, a returned delivered state becomes `sent`. Any other returned state is a failure, mapped through the service's classification table to one of the failure classifications above, or to `unknown` when the table has no entry.
6. On success, clear the failure classification and, when the caller asked for sent records to be removed, set the deletion mark. On failure, set the state to `error` with the classification and, when the caller asked for failed records to be removed, set the deletion mark.
7. Update the tracking record attached to each text message so that the thread and the campaign statistics follow the outcome, and push the notification update to the thread.

The queue manager asks for sent records to be removed and for failed ones to be kept, which is why a drained queue contains only failures.

### 5.4 Manual resend

The manual resend operation takes a selection, moves the records in the `error` state that are not marked for deletion back to `outgoing`, and sends them at once. It then counts how many records of the selection no longer exist, which is the number of successful sends, and returns one of three notifications.

| Outcome | Title | Type | Message |
|---|---|---|---|
| At least one sent | `Success` | success | "<count> out of the <total> selected SMS Text Messages have successfully been resent." |
| None sent, some attempted | `Warning` | danger | "The SMS Text Messages could not be resent." |
| Nothing to resend | `Warning` | danger | "There are no SMS Text Messages to resend." |

Those three messages are reproduced verbatim. The placeholders are the number of records that disappeared and the size of the selection.

### 5.5 Cleanup

The automatic cleanup of [`scheduled-jobs.md`](scheduled-jobs.md), section 11, deletes every text message row whose deletion mark is set, and logs the number removed. Notifications are never deleted by this cleanup, which is why the thread keeps its record of a message whose queue row is gone.

## 6. Postal letters and browser push notifications

### 6.1 Postal letters

A **Postal Letter** is a document handed to a printing and posting service.

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `user_id` | Sent by | Many-to-one to User | Who requested the letter. |
| `model`, `res_id` | Model, Document Identifier | Text, integer, both required | The record the letter documents. |
| `partner_id` | Recipient | Many-to-one to Contact, required | The addressee. |
| `company_id` | Company | Many-to-one to Company, required, read-only | The sending company. |
| `report_template` | Optional report to print and attach | Many-to-one to Report Action | The printable document to render, when the letter is not a ready-made file. |
| `attachment_id` | Attachment | Many-to-one to Attachment, cascading deletion | The file that is printed. |
| `color`, `cover`, `duplex` | Color, Cover Page, Both side | Booleans, defaulted from the company | The print options. |
| `state` | Status | Selection, default `pending`, read-only | The queue state. |
| `error_code` | Error | Selection over the service's codes | The failure the service returned. |
| `info_msg` | Information | Rich text | The explanation shown to the requester. |
| `message_id` | Snailmail Status Message | Many-to-one to Message | The thread message that carries the delivery status. |
| `notification_ids` | Notifications | One-to-many to Notification | The per-recipient delivery records. |
| `street`, `street2`, `zip`, `city`, `state_id`, `country_id` | Street, Street 2, Zip, City, State, Country | Address fields | The address frozen at creation, so that a later change of the contact does not change what was posted. |

The states are `pending` (In Queue, the default), `sent` (Sent), `error` (Error) and `canceled` (Cancelled).

The service's failure codes are reproduced as stored values: `MISSING_REQUIRED_FIELDS`, `CREDIT_ERROR`, `TRIAL_ERROR`, `NO_PRICE_AVAILABLE`, `FORMAT_ERROR`, `UNKNOWN_ERROR` and `ATTACHMENT_ERROR`. Each maps to a notification failure classification:

| Failure code | Meaning | Notification failure classification |
|---|---|---|
| `TRIAL_ERROR` | The deployment is on a trial that does not permit posting. | `sn_trial` |
| `CREDIT_ERROR` | The account has insufficient credit. | `sn_credit` |
| `NO_PRICE_AVAILABLE` | No price exists for the requested destination. | `sn_price` |
| `MISSING_REQUIRED_FIELDS` | The address lacks a required part. | `sn_fields` |
| `FORMAT_ERROR` | The document could not be handled in the requested format. | `sn_format` |
| `ATTACHMENT_ERROR` | The attached document could not be read. | `sn_error` |
| `UNKNOWN_ERROR` | Anything else. | `sn_error` |

Creating a letter posts a message on the documented record with the body `Letter sent by post with Snailmail` and the message type `snailmail`, freezes the recipient's address onto the letter, and creates one notification of the postal kind, already marked as read so that it does not appear in anyone's inbox, with the status `ready`.

Draining:

1. Select every letter whose state is `pending`, together with every letter whose state is `error` and whose failure code is one of the four recoverable ones: `TRIAL_ERROR`, `CREDIT_ERROR`, `ATTACHMENT_ERROR` and `MISSING_REQUIRED_FIELDS`.
2. For each letter in turn, print it as specified below.
3. If the letter's failure code is now `CREDIT_ERROR`, stop the whole run. Continuing would produce one identical failure, and one identical notification, per remaining letter.
4. Otherwise commit and take the next letter.

Printing a set of letters:

1. Split the set into the letters whose frozen address has a street, a city, a postal code and a country, and the rest.
2. For every letter in the rest: set the state to `error` and the failure code to `MISSING_REQUIRED_FIELDS`, set the explanation to `The address of the recipient is not complete`, move its notifications to the exception status with the matching failure classification and that same explanation as the reason, and push the notification update to the thread.
3. For every letter with a complete address, when the caller asked for immediate printing: submit it to the service and apply the outcome, then commit.

Submitting one letter reads the service address from the system parameter `snailmail.endpoint` and the timeout in seconds from `snailmail.timeout`, whose defaults are the publisher's service and 30 seconds respectively, and calls the printing operation at the path `/iap/snailmail/1/print`. If the call is refused for lack of authorisation, every letter of the request is set to `error` with the code `UNKNOWN_ERROR` and the refusal propagates.

For each document in the answer:

- If the document reports that it was sent and the request as a whole reports success, set the state to `sent`, clear the failure code, set the explanation to `The document was correctly sent by post.` followed by a line break and `The tracking id is <identifier>`, where the placeholder is the tracking identifier the service returned, and set the notifications to the sent status with no failure. A success notification is pushed to the requester with the body `Snail Mails are successfully sent`.
- Otherwise take the failure code from the document, or from the request when the request itself failed. If it is `CREDIT_ERROR`, push the insufficient-credit notification with the title `Not enough credits for Snail Mail`. Set the explanation to `An error occurred when sending the document by post.` followed by a line break and `Error: <message>`, where the placeholder is the human-readable form of the failure code. Set the state to `error` and the failure code to the returned code when it is one of the seven, `UNKNOWN_ERROR` otherwise. Set the notifications to the exception status with the matching failure classification and the explanation as the reason.

Finally push the notification update to the thread.

Re-queuing a letter sets its state back to `pending`, resets its notifications to the ready status with no failure, pushes the update, and prints the letter immediately when exactly one was re-queued. Cancelling sets the state to `canceled`, clears the failure code, sets the notifications to the cancelled status and pushes the update.

### 6.2 Browser push notifications

A queued **Push Notification** holds a reference to a registered device and the serialized payload to deliver.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `mail_push_device_id` | Device | Many-to-one to Push Notification Device, cascading deletion | yes | The browser subscription to deliver to. |
| `payload` | Payload | Long text | no | The serialized notification. |

A **Push Notification Device** records one browser subscription.

| Identifier | Full name | Type | Required | Meaning |
|---|---|---|---|---|
| `partner_id` | Partner | Many-to-one to Contact, defaults to the acting user's contact | yes | Whose device it is. |
| `endpoint` | Browser endpoint | Text | yes | The address the browser's push service exposes for this subscription. Unique; the violation message is `The endpoint must be unique !`. |
| `keys` | Browser keys | Text | yes | The subscription public key the browser generated and the authentication secret, serialized together. The browser keeps the matching private key and uses it to decrypt the payload. |
| `expiration_time` | Expiration Token Date | Date and time | no | When the subscription expires, as the browser declared it. |

The signing key pair is held in the system parameters `mail.web_push_vapid_private_key` and `mail.web_push_vapid_public_key`, the private and public voluntary application server identification keys. Requesting the public key when none is stored deletes every registered device, generates a fresh pair, stores both and logs `WebPush: missing public key, new VAPID keys generated`; deleting the devices is required because every existing subscription was made against the old key and can no longer be verified.

Registering a device verifies that the public key the browser presents is the stored one, and refuses otherwise. A registration that names an endpoint already stored under a different contact re-points that row at the new contact; a registration for an unknown endpoint creates a row.

Draining:

1. Select at most 50 queued notifications, reading only the device reference and the payload. If none, stop.
2. Read the two signing keys. If either is missing, stop without deleting anything: the queue is preserved until a key pair exists.
3. Open one connection to reuse across the batch, and keep a set of devices found unreachable in this run.
4. For each queued notification in turn: if its device is already in the unreachable set, skip it. Otherwise deliver the payload to the device endpoint, signed with the key pair and addressed from the deployment's own address.
5. A delivery that reports the device as unreachable adds the device to the unreachable set. Any other failure is logged as `An error occurred while trying to send web push: <e>`, where the placeholder is the failure, and the run continues; one malformed payload therefore cannot abort the batch.
6. Delete every notification of the selection, whether delivered or not.
7. Delete every device in the unreachable set, which also cascades to that device's other queued notifications.
8. If any queued notification remains, trigger the job again.

Step 6 is the delete-on-completion pattern: a push notification is attempted once and then discarded, because a stale desktop notification has no value.

## 7. Digests

### 7.1 The entity

A **Digest Email** is a periodic summary of chosen indicators, sent to chosen users.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Name | Text, translatable | yes | none | The title of the digest. |
| `user_ids` | Recipients | Many-to-many to User, restricted to non-shared users | no | none | Who receives it. |
| `periodicity` | Periodicity | Selection: `daily`, `weekly`, `monthly`, `quarterly` | yes | `daily` | The sending rhythm. |
| `next_run_date` | Next Mailing Date | Date | no | derived at creation | The next date the digest is due. |
| `company_id` | Company | Many-to-one to Company | no | the acting company | Scoping of the indicators. |
| `currency_id` | Currency | Related to the company's currency | no | derived | The currency amounts are shown in. |
| `state` | Status | Selection: `activated`, `deactivated`, read-only | yes | `activated` | Only activated digests are sent. |
| `is_subscribed` | Is user subscribed | Boolean, computed, not stored | no | derived | Whether the reader is among the recipients. |
| `available_fields` | Available fields | Text, computed, not stored | no | derived | The list of value fields the selected indicators expose to the template. |

Each indicator contributes two fields: a boolean that selects it and a computed value. A field is recognised as an indicator selector when it is boolean and its name begins with `kpi_`, `x_kpi_` or `x_studio_kpi_`; the matching value field carries the same name followed by `_value`. The two the platform itself ships are `kpi_res_users_connected` (Connected Users), whose value counts the users whose last login falls inside the period, and `kpi_mail_message_total` (Messages Sent), whose value counts the comment-type thread messages created inside the period. Capability packages add their own by declaring further pairs.

Subscribing adds the acting user to the recipients, unsubscribing removes them; both are available only to internal users and both act with elevated rights so that a recipient does not need write access to the digest.

### 7.2 The schedule

```formula
next mailing date = today + one period
```

where one period is 1 day for `daily`, 1 week for `weekly`, 1 month for `monthly` and 3 months for `quarterly`. Month and quarter arithmetic is calendar arithmetic: adding one month to 31 January yields 28 February in a common year and 29 February in a leap year, because the day is clamped to the last day of the target month.

The next mailing date is computed at creation when it was not supplied, on every change of the periodicity, and after every send.

### 7.3 Sending

The job selects every digest whose next mailing date is at or before today and whose state is `activated`, and sends each in turn. A delivery failure logs the warning `MailDeliveryException while sending digest <key>. Digest is now scheduled for next cron update.`, where the placeholder is the digest's key, and the job moves to the next digest; the failing digest keeps its next mailing date and is therefore retried at the next execution.

Sending a set of digests:

1. When the caller asked for the periodicity to be adapted, compute the subset to slow down as specified in section 7.4. A manual send does not ask for it.
2. For each digest, for each recipient: render the digest in that recipient's language and with that recipient's company, and create one outgoing message for it.
3. If the digest is in the slow-down subset, replace its periodicity by the next slower one.
4. Set the next mailing date to today plus one period of the periodicity the digest now has.

Rendering one digest for one recipient produces a body containing: the digest title, a button labelled `Connect` pointing at the deployment's own address, the formatted date of the day in the long month-day-year form, the indicator block, one tip, and the preferences block. The body is then wrapped in the digest layout. The outgoing message carries automatic deletion, the sending company's contact address as sender, falling back to the acting user's address and then to the root user's address, the recipient's formatted address, the subject `<company>: <digest name>` and the state `outgoing`, so that the ordinary outgoing queue of [`mail-gateway.md`](mail-gateway.md) delivers it.

Three headers are set on the message so that a mail client can offer a one-click unsubscribe: the unsubscribe address, the declaration that the unsubscribe address accepts a one-click post, and the suppression of automatic out-of-office replies. The unsubscribe address is the deployment's own address followed by the route path `/digest/<key>/unsubscribe_oneclik`, carrying a token and the user key. The token is a keyed digest computed over the pair of digest key and user key with the platform secret, which is what lets an unauthenticated request unsubscribe exactly one user from exactly one digest and nothing else.

The preferences block carries, in order: the slow-down notice when the digest is being slowed down; otherwise, for a daily digest read by a settings administrator, the offer `Prefer a broader overview?` with the link `Switch to weekly Digests` pointing at the route path `/digest/<key>/set_periodicity?periodicity=weekly`; and for any settings administrator, the offer `Want to customize this email?` with the link `Choose the metrics you care about`.

### 7.4 The slow-down rule

| Current periodicity | Inactivity window | Next slower periodicity |
|---|---|---|
| `daily` | 2 days | `weekly` |
| `weekly` | 7 days | `monthly` |
| `monthly` | 1 month | `quarterly` |
| `quarterly` | 3 months | `quarterly` |

A digest is slowed down when **no** login log record created by **any** of its recipients exists within the window measured back from the current instant. One recipient who logged in keeps the whole digest at its current rhythm. A digest already at the quarterly rhythm stays there.

The notice added to the body in that case is `We have noticed you did not connect these last few days. We have automatically switched your preference to %(new_perioridicy_str)s Digests.`, reproduced verbatim including the spelling of its placeholder, which is filled with the name of the new periodicity in the recipient's language: `weekly`, `monthly` or `quarterly`.

A manual send never slows a digest down, because a person asking for the digest now is not evidence that its recipients are inactive.

### 7.5 Indicator values and comparison

Each indicator is computed over three timeframes, and each timeframe is compared with the equally long timeframe immediately before it.

| Timeframe | Current period | Comparison period |
|---|---|---|
| `Last 24 hours` | the last day | the day before that |
| `Last 7 Days` | the last week | the week before that |
| `Last 30 Days` | the last month | the month before that |

The instant the timeframes are measured back from is the current instant expressed in the time zone of the company's working schedule when that schedule declares one, and in coordinated universal time otherwise.

The variation shown beside a value is:

```formula
variation in percent = ( current value − previous value ) ÷ previous value × 100
```

rounded to two decimal places, and reported as zero when the two values are equal, when the current value is zero or when the previous value is zero. The last two exclusions avoid a division by zero and avoid reporting an infinite improvement from nothing.

**Worked example.** An indicator counts 47 connected users this week and 40 the week before. The variation is ( 47 − 40 ) ÷ 40 × 100 = 17.50 percent. With 0 the week before, the variation is reported as 0.00 percent rather than as an infinite increase. With 47 both weeks, the variation is 0.00 percent.

A digest whose indicators are all zero for the current period is still sent; the digest is a rhythm, not a report of activity.

## 8. Data recycling

Data recycling deletes or archives records that a rule declares stale. It is the only queue in this document whose queued record is created by the worker rather than by a business transaction: the worker's first pass finds the stale records and queues them, and either acts on them at once or leaves them for a person to confirm.

### 8.1 The Recycling Model entity

A **Recycling Model** is one rule.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `active` | Active | Boolean | no | true | Deactivating the rule deletes every queued record of that rule. |
| `name` | Name | Text, computed from the entity, writable, stored | yes | the entity's name | The rule's label. |
| `res_model_id` | Model | Many-to-one to Entity, cascading deletion | yes | none | The entity whose records the rule recycles. |
| `res_model_name` | Model Name | Text, related to the entity's transport name, stored, read-only | no | derived | The transport name, kept for searching. |
| `recycle_record_ids` | Records | One-to-many to Recycling Record | no | none | The queued records of this rule. |
| `recycle_mode` | Recycle Mode | Selection: `manual`, `automatic` | yes | `manual` | Whether the action is applied at once or waits for confirmation. |
| `recycle_action` | Recycle Action | Selection: `archive`, `unlink` | yes | `unlink` | Whether the record is archived or deleted. |
| `domain` | Filter | Text holding a condition, computed to the empty condition on a change of entity, writable, stored | no | the empty condition | The extra condition a record must satisfy. |
| `time_field_id` | Time Field | Many-to-one to Entity Field, cascading deletion, restricted to stored date and date-and-time fields of that entity | no | none | The field the age is measured on. |
| `time_field_delta` | Delta | Integer | no | 1 | The age threshold. |
| `time_field_delta_unit` | Delta Unit | Selection: `days`, `weeks`, `months`, `years` | no | `months` | The unit of the threshold. |
| `include_archived` | Include archived | Boolean | no | false | Whether archived records are candidates. |
| `records_to_recycle_count` | Records To Recycle | Integer, computed, not stored | no | derived | How many records are queued for this rule. |
| `notify_user_ids` | Notify Users | Many-to-many to User, restricted to system administrators | no | the acting user | Who is told that records are waiting. |
| `notify_frequency` | Notify | Integer | no | 1 | How often they are told. |
| `notify_frequency_period` | Notify Frequency Period | Selection: `days`, `weeks`, `months` | no | `weeks` | The unit of that frequency. |
| `last_notification` | Last notification | Date and time, read-only | no | none | When the last notification was sent. |

Two validations apply.

- The notification frequency must be greater than zero. The violation message is `The notification frequency should be greater than 0`.
- A rule whose action is `archive` may only name an entity that supports archiving. The refusal reads `This model doesn't manage archived records. Only deletion is possible.`

Deactivating a rule deletes its queued records, so that a rule that is turned off leaves no pending proposals behind.

### 8.2 The Recycling Record entity

A **Recycling Record** is one proposal.

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `active` | Active | Boolean, default true | Discarding a proposal sets this to false rather than deleting it, so that the same record is not proposed again on the next pass. |
| `name` | Record Name | Text, computed with elevated rights, not stored | The display name of the proposed record, or `**Record Deleted**` when it no longer exists, or `Undefined Name` when it exists but has no display name. |
| `recycle_model_id` | Recycle Model | Many-to-one to Recycling Model, cascading deletion | The rule that proposed it. |
| `res_id` | Record Identifier | Integer, indexed, never aggregated | The key of the proposed record. |
| `res_model_id`, `res_model_name` | Model, Model Name | Related to the rule, stored, read-only | The entity of the proposed record. |
| `company_id` | Company | Many-to-one to Company, computed, stored | The company of the proposed record when that entity has one, empty otherwise. |

Resolving proposals to their records is done in one read per entity, with elevated rights and with archived records included, and proposals whose record has disappeared resolve to nothing rather than to an error.

### 8.3 The job

1. Read every rule, with elevated rights, and run the recycling pass over all of them with batch commits enabled.
2. Then run the notification pass.

The recycling pass:

1. Flush the unit of work, so that records changed earlier in this transaction are visible to the searches that follow.
2. Read every existing proposal of the rules in scope, including discarded ones, and index the proposed record keys by rule. This index is what prevents a record from being proposed twice and what keeps a discarded proposal discarded.
3. For each rule: start from the rule's condition, or from the always-true condition when the rule has none. When the rule names a time field and a non-zero threshold with a unit, add the condition that the time field is at or before the current moment minus the threshold. The current moment is today's date when the field is a date, and the current instant when it is a date and time.
4. Search that condition on the rule's entity, including archived records when the rule asks for them.
5. Build one proposal for every found record whose key is not already in the index for that rule.
6. If the rule's mode is `automatic`, create the proposals in batches of 5 000 and validate each batch immediately; commit after each batch unless the pass is running inside a test. If the rule's mode is `manual`, add the proposals to a pending list.
7. After every rule has been examined, create the pending manual proposals in batches of 50 000, committing after each batch unless running inside a test.

The two batch sizes differ by an order of magnitude because the automatic mode acts on each batch, which is slow, while the manual mode only writes proposal rows.

Validating a set of proposals:

1. Resolve every proposal to its record.
2. Group the resolved records by entity and by the action their rule declares.
3. Archive every record whose rule declares `archive`, with elevated rights.
4. Delete every record whose rule declares `unlink`, with elevated rights.
5. Delete every proposal that was examined, including those whose record had already disappeared.

Discarding a proposal instead sets its active flag to false.

The notification pass:

1. For every rule in the `manual` mode that names at least one user to notify and a non-zero frequency: compute the interval from the frequency and its unit.
2. If the rule has never notified, or if the last notification plus the interval is earlier than the current instant, set the last notification to the current instant and send.
3. Sending counts the proposals of that rule created on or after today minus the interval. If the count is zero, nothing is sent, and the last notification instant has still been advanced.
4. Otherwise a notification is posted to the contacts of the users to notify, with the subject `Data to Recycle` and a body that names the count, the label of the entity and a link into the recycling screen.

Advancing the instant before deciding whether there is anything to say means that a period with no new proposals consumes its notification slot. This is a **compatibility finding**: the corrected behaviour would advance the last-notification instant only when a notification was actually sent, so that the first period with new proposals notifies immediately rather than at the end of the following period.

### 8.4 The interactive path

Running a rule from its form performs the recycling pass for that rule alone, without batch commits, and then opens the proposal list filtered on that rule when the mode is `manual`. In the `automatic` mode nothing is opened, because the records have already been acted on.

## 9. Time-based automation

An automation rule whose trigger belongs to the time family is evaluated by a job rather than by a record change. The three time triggers are `on_time` (a delay measured from a chosen date field), `on_time_created` (a delay measured from the creation instant) and `on_time_updated` (a delay measured from the last update instant).

### 9.1 The rule fields that matter here

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `trigger` | Trigger | Selection | The trigger kind; only the three time values are handled by this job. |
| `trg_date_id` | Trigger Date | Many-to-one to Entity Field, restricted to date and date-and-time fields of the rule's entity | The field the delay is measured from. |
| `trg_date_range` | Delay | Integer | The offset. |
| `trg_date_range_mode` | Delay mode | Selection: `after`, `before` | Whether the offset is counted after or before the field value. |
| `trg_date_range_type` | Delay unit | Selection: `minutes`, `hour`, `day`, `month` | The unit of the offset. |
| `trg_date_calendar_id` | Use Calendar | Many-to-one to Working Schedule | When set and the unit is `day`, the offset is counted in working days according to that schedule rather than in calendar days. |
| `filter_domain` | Apply on | Text holding a condition | The extra condition the records must satisfy. |
| `last_run` | Last Run | Date and time | The instant of the last successful evaluation of this rule. |

Changing the trigger away from the time family clears the date field, the offset, its mode and its unit. Selecting `on_time_created` or `on_time_updated` sets the date field to the corresponding automatic field, the unit to `hour` and the mode to `after`. Setting the offset to a negative number on the form flips the mode and takes the absolute value, so that the form never holds a negative offset. Clearing the day unit, or leaving the time family, clears the working schedule.

The offset may not be negative. The refusal reads `Delay must be positive. Set 'Delay mode' to 'Before' to negate the delay.`

Calling the evaluation outside a scheduled job is refused: the operation asserts that it is running in automatic mode and raises otherwise.

### 9.2 The job

1. If the execution context carries no marker of completed actions, add an empty one. The marker is what prevents a rule from re-triggering itself through the writes it performs.
2. Select every rule whose trigger is in the time family, considering only active rules.
3. For each rule, in order:
   1. Re-read the rule's active flag. A rule deactivated since the selection is skipped; a rule deleted since the selection is skipped without error.
   2. Log `Starting time-based automation rule `<name>`.`, where the placeholder is the rule's name.
   3. Take the transaction timestamp as the current instant.
   4. Select the records due for this rule at that instant, as specified in section 9.3.
   5. Run the rule on each due record in turn, then flush.
   6. On any failure of steps 4 or 5: roll back, log the exception under `Error in time-based automation rule `<name>`.`, remember the failure, and continue with the next rule. The rule's last-run instant is **not** advanced, so its due records are selected again at the next execution.
   7. Write the current instant as the rule's last run.
   8. Log `Time-based automation rule `<name>` done.`
   9. Report one unit of progress, which commits.
4. If any rule failed, re-raise the last failure, which makes the job report a failure and increments its failure counter.

Each rule therefore commits independently, a failing rule does not prevent the others from running, and the job still ends as failed so that the operator sees it.

### 9.3 Selecting the due records

Let the **watermark** be the rule's last run, or the beginning of time when it has never run; let **now** be the current instant; let the **sign** be +1 when the mode is `before` and −1 when it is `after`; and let the **range** be the sign multiplied by the offset.

```formula
range = sign × offset
```

The rule's condition is evaluated first and always applies. When the rule names no date field on its entity, the warning `Missing date trigger field in automation rule `<name>`.` is logged and no record is selected.

**The working-schedule variant** applies when the rule names a schedule and the unit is `day`.

1. Select the candidate records: those matching the rule's condition and having a value in the date field. For the automatic last-update field, every record is a candidate, because a record with no value falls back to its creation instant.
2. For each candidate, determine the working schedule that applies to it, which the entity may derive per record.
3. For each distinct schedule, and once only, compute two horizons by planning the range in working days from **now** and from the **watermark**, counting the schedule's leave periods as non-working.
4. Keep the record when its date is at or after the watermark horizon and strictly before the now horizon.

**The ordinary variant** applies otherwise.

```formula
offset in time = unit of the offset × range
relative watermark = watermark + offset in time
relative now = now + offset in time
```

The unit converts as follows: `minutes` is one minute, `hour` is one hour, `day` is one day and `month` is one calendar month.

For a date field, the condition is: the date is strictly after the relative watermark's date **and** at or before the relative now's date. For a date-and-time field, the condition is: the value is at or after the relative watermark **and** strictly before the relative now.

For the automatic last-update field, records with no value are additionally accepted when their creation instant falls in the same window.

If the field is stored or searchable, the whole condition is evaluated by the search. Otherwise the rule's condition alone is searched and the window is applied in memory afterwards.

The window is half-open and anchored on the previous run, which is what makes each record fire exactly once even though the job runs every few hours, and what makes a rule that has never run fire on its whole history at its first execution.

**Compatibility finding.** In the date-field branch, the upper bound of the fallback window for records with no value in the automatic last-update field is taken from the current date rather than from the relative now. When the offset is not zero the two differ, and records created between the two bounds are selected one execution early or late. The corrected behaviour is to use the relative now's date in both bounds of that fallback window, exactly as in the main window.

**Worked example.** A rule sends a reminder 3 days **before** a deadline field. Its mode is `before`, so the sign is +1 and the range is +3 days. It last ran at 2026-03-01 08:00 and runs again at 2026-03-01 12:00.

```formula
relative watermark = 2026-03-01 08:00 + 3 days = 2026-03-04 08:00
relative now       = 2026-03-01 12:00 + 3 days = 2026-03-04 12:00
```

A record whose deadline is 2026-03-04 09:30 is at or after 2026-03-04 08:00 and strictly before 2026-03-04 12:00, so it fires now. A record whose deadline is exactly 2026-03-04 08:00 fires now, because the lower bound is inclusive. A record whose deadline is exactly 2026-03-04 12:00 does not fire now, because the upper bound is exclusive; it fires at the next execution, whose lower bound is that same instant. A record whose deadline is 2026-03-04 12:30 fires at the next execution provided that execution happens after 2026-03-01 12:30.

**Worked example with a working schedule.** The same rule uses a five-day working schedule with Saturday and Sunday off, and the unit `day`. It last ran on Friday 2026-03-06 at 08:00 and runs again the same day at 12:00. Planning 3 working days forward from 12:00 on Friday reaches Wednesday 2026-03-11 at 12:00; from 08:00 on Friday it reaches Wednesday 2026-03-11 at 08:00. A record whose deadline is Wednesday 2026-03-11 at 09:00 fires now. A record whose deadline is Monday 2026-03-09 does not, because Monday is only one working day ahead.

## 10. Marketing campaigns

The marketing campaign queue advances one campaign per iteration.

1. Select every campaign whose state is `in_queue` or `sending` and whose scheduled instant is either unset or already past.
2. Report progress with a remaining count equal to the number of campaigns selected and no processed count, which tells the scheduler how much work the iteration owes before any of it is done.
3. For each campaign in turn:
   1. Adopt the execution context of the campaign's responsible user, falling back to its last editor and then to the acting user. The campaign is therefore rendered in the responsible user's language and companies, not in the scheduler user's.
   2. Count the recipients still to reach. If any remain, set the state to `sending` and send the next portion.
   3. If none remain, set the state to `done`, set the sent instant to the current instant, and mark the campaign as requiring a summary when it had never been sent before.
   4. Report one unit of progress, which commits.
4. When the system parameter `mass_mailing.mass_mailing_reports` is set, select every campaign that is marked as requiring a summary, is in the state `done`, and whose sent instant is between one and five days ago, and send each responsible user the summary of their campaign. Sending a summary clears the mark.

The one-to-five-day window is what makes the summary useful: it is sent once the recipients have had a day to react, and it is not sent at all for a campaign whose results are already stale.

## 11. Worked examples

### 11.1 A text message queue that spans three iterations

1 300 text messages are queued and the batch size is the default 500. Nothing else holds a lock on them.

| Iteration | Selected | Remaining reported | Scheduler decision |
|---|---|---|---|
| 1 | 500, the lowest keys | selection filled the batch, so a full count is taken: 800 | keep looping |
| 2 | 500 | full count: 300 | keep looping |
| 3 | 300 | the batch was not filled, so 0 is reported | `fully done` |

Had a second worker held 120 of the first 500 rows, iteration 1 would have sent 380, reported 500 processed and a full count of 800 remaining, and the 120 would have been taken by whichever worker committed first.

### 11.2 A postal letter run stopped by credit

Six letters are queued: four pending with complete addresses, one pending whose address has no postal code, and one in error with the code `CREDIT_ERROR`. The service has credit for two letters.

1. Letter 1 prints, reaches `sent`, and the run commits.
2. Letter 2 prints, reaches `sent`, and the run commits.
3. Letter 3 is refused with `CREDIT_ERROR`, is set to `error` with that code and the explanation `An error occurred when sending the document by post.` followed by the human-readable form of the code, its notification moves to the exception status with the classification `sn_credit`, and the insufficient-credit notification is pushed with the title `Not enough credits for Snail Mail`.
4. The run stops. Letters 4, 5 and 6 keep their state.

At the next execution the selection contains letter 3 again, because `CREDIT_ERROR` is recoverable, letters 4 and 6, and letter 5 whose incomplete address will set it to `error` with `MISSING_REQUIRED_FIELDS` and the explanation `The address of the recipient is not complete` without any call to the service.

### 11.3 A digest slowed down and then restored

A daily digest has three recipients and its next mailing date is today.

1. The job selects it. No login log record exists for any of the three recipients in the last two days, so the digest is in the slow-down subset.
2. It is rendered once per recipient, each in that recipient's language, with the preferences block carrying the notice `We have noticed you did not connect these last few days. We have automatically switched your preference to weekly Digests.`
3. Its periodicity becomes `weekly` and its next mailing date becomes today plus one week.
4. Tomorrow one recipient logs in. Seven days later the job selects the digest again. A login log record exists inside the seven-day window, so the digest is not slowed down further. It is sent as a weekly digest and its next mailing date becomes that day plus one week.
5. A person then opens the digest and sends it manually. The periodicity is unchanged and the next mailing date is recomputed from today, which moves the next automatic send one week from the manual send.

### 11.4 A recycling rule in automatic mode

A rule names the lead entity, the mode `automatic`, the action `unlink`, the time field the last update instant, a threshold of 18 months, and no extra condition. 12 300 leads match, and none is already proposed.

1. The pass computes the cutoff as the current instant minus 18 months.
2. It searches the leads whose last update is at or before the cutoff and finds 12 300.
3. It creates 5 000 proposals, validates them, which deletes 5 000 leads and then the 5 000 proposals, and commits.
4. It repeats for the next 5 000, and commits.
5. It repeats for the last 2 300, and commits.
6. The notification pass skips the rule, because the rule is not in the `manual` mode.

An interruption after step 4 leaves 10 000 leads deleted and 2 300 still present; the next execution re-searches, finds the 2 300, and finishes the work. That is the batch-commit rule of [`transactions-and-concurrency.md`](transactions-and-concurrency.md) applied: the pass is idempotent at the granularity of one batch because the search is re-evaluated from the surviving data.

### 11.5 A rule that has never run

A time-based rule with a 2-hour `after` offset on a creation date field is activated for the first time on 2026-05-04 at 09:00. Its watermark is the beginning of time.

```formula
range              = −1 × 2 = −2 hours
relative watermark = beginning of time − 2 hours = beginning of time
relative now       = 2026-05-04 09:00 − 2 hours = 2026-05-04 07:00
```

Every record created strictly before 2026-05-04 07:00 is due, which is the entire history of the entity. The rule fires on all of them in one execution. An operator who does not want that sets the rule's last-run instant before activating it, which is the only supported way to limit the first pass.

## 12. Acceptance criteria

1. **Given** a deferred notification scheduled for a precise instant, **when** it is created, **then** a trigger exists for that instant and the notification is sent then rather than at the next hourly execution.
2. **Given** two deferrals created together for the same instant, **when** they are created, **then** exactly one trigger is created, because triggers are created per distinct instant.
3. **Given** a deferred notification whose referenced record has been deleted, **when** the job runs, **then** it is skipped and the deferral row is still deleted.
4. **Given** a deferral whose serialized parameters cannot be read, **when** the job runs, **then** the notification is sent with the default of skipping already-notified recipients and no error is raised.
5. **Given** a deferral whose parameters carry a scheduled instant, **when** the job replays them, **then** that instant is removed from the arguments and the notification is not deferred a second time.
6. **Given** 70 user-scheduled messages that are due, **when** the posting job runs, **then** 50 are posted, each committed separately, and the job triggers itself again for the remaining 20.
7. **Given** a user-scheduled message whose creator has lost write access to the target record, **when** the job reaches it, **then** the posting is rolled back, the creator receives the notification `A scheduled message could not be sent`, and the scheduled message is deleted.
8. **Given** a scheduled message being created on an entity without a discussion thread, **when** it is saved, **then** the save is refused with `A message cannot be scheduled on a model that does not have a mail thread.`
9. **Given** a scheduled message whose instant is in the past, **when** it is saved, **then** the save is refused with `A Scheduled Message cannot be scheduled in the past`.
10. **Given** a scheduled message whose target record is changed, **when** the change is saved, **then** it is refused with `You are not allowed to change the target record of a scheduled message.`
11. **Given** 800 queued text messages and the default batch size of 500, **when** the job runs one iteration, **then** 500 are attempted and the report names 800 as remaining.
12. **Given** 300 queued text messages and the default batch size of 500, **when** the job runs, **then** 300 are attempted and 0 is reported as remaining, because the batch was not filled.
13. **Given** queued text messages of which some are locked by another worker, **when** the job runs, **then** only the unlocked ones are attempted and the locked ones are left for the other worker.
14. **Given** two queued text messages with an identical body and different numbers, **when** they are sent, **then** one request is made carrying both numbers.
15. **Given** a text message whose service call fails as a whole, **when** the outcome is applied, **then** every message of that batch is set to `error` with the classification `sms_server`.
16. **Given** a text message that the service reports as delivered, **when** the outcome is applied, **then** its state is `sent` and, because the queue manager asks for sent records to be removed, its deletion mark is set.
17. **Given** a text message row whose deletion mark is set, **when** the automatic cleanup runs, **then** the row is deleted and its notification is kept.
18. **Given** a selection of three text messages in the `error` state of which two send successfully on resend, **when** the resend finishes, **then** the notification title is `Success` and its message names 2 out of the 3 selected.
19. **Given** a selection containing no resendable text message, **when** the resend is requested, **then** the notification message is "There are no SMS Text Messages to resend."
20. **Given** a postal letter run in which one letter fails with `CREDIT_ERROR`, **when** that letter is processed, **then** the run stops and the remaining letters keep their state.
21. **Given** a postal letter whose frozen address has no postal code, **when** it is printed, **then** its state is `error`, its failure code is `MISSING_REQUIRED_FIELDS`, its explanation is `The address of the recipient is not complete`, its notification is at the exception status with the classification `sn_fields`, and the service is not contacted.
22. **Given** a postal letter that the service accepts, **when** the outcome is applied, **then** its state is `sent`, its explanation names the tracking identifier, and a success notification with the body `Snail Mails are successfully sent` is pushed.
23. **Given** a contact whose street changes after a letter was created, **when** the letter is printed, **then** the address frozen on the letter is used.
24. **Given** more than 50 queued push notifications, **when** the job runs, **then** 50 are delivered, all 50 queue rows are deleted, and the job triggers itself again.
25. **Given** a push notification whose device is reported unreachable, **when** the run finishes, **then** that device row is deleted and the device's other queued notifications were skipped rather than attempted.
26. **Given** a push notification whose delivery raises an unexpected failure, **when** the run continues, **then** the failure is logged, the remaining notifications are still attempted, and the row is still deleted.
27. **Given** no stored signing key pair, **when** the push job runs, **then** it stops without deleting any queued notification.
28. **Given** a request for the public signing key when none is stored, **when** it is served, **then** every registered device is deleted, a new pair is generated and stored, and the new public key is returned.
29. **Given** a daily digest whose three recipients have not logged in for two days, **when** it is sent by the job, **then** it is delivered once per recipient with the slow-down notice, its periodicity becomes `weekly` and its next mailing date becomes today plus one week.
30. **Given** the same digest sent manually, **when** it is delivered, **then** its periodicity is unchanged and its next mailing date is today plus one day.
31. **Given** a quarterly digest whose recipients have not logged in for three months, **when** it is sent, **then** its periodicity stays `quarterly`.
32. **Given** a digest whose delivery raises, **when** the job continues, **then** the warning naming that digest is logged, its next mailing date is unchanged, and the following digests are still sent.
33. **Given** an indicator worth 47 now and 40 in the comparison period, **when** the digest is rendered, **then** the variation shown is 17.50 percent.
34. **Given** an indicator worth 47 now and 0 in the comparison period, **when** the digest is rendered, **then** the variation shown is 0.00 percent.
35. **Given** a recycling rule in `automatic` mode matching 12 300 records, **when** the job runs, **then** the records are deleted in batches of 5 000 with a commit after each batch.
36. **Given** a recycling rule in `manual` mode matching 60 000 records, **when** the job runs, **then** 60 000 proposals are created in batches of 50 000 with a commit after each, and no record is deleted.
37. **Given** a proposal that was discarded, **when** the recycling pass runs again, **then** the same record is not proposed a second time.
38. **Given** a recycling rule that is deactivated, **when** the change is saved, **then** every proposal of that rule is deleted.
39. **Given** a recycling rule whose action is `archive` on an entity that does not support archiving, **when** it is saved, **then** the save is refused with `This model doesn't manage archived records. Only deletion is possible.`
40. **Given** a recycling rule with a notification frequency of zero, **when** it is saved, **then** the save is refused with `The notification frequency should be greater than 0`.
41. **Given** a manual recycling rule that has produced no new proposal since its last notification, **when** the notification pass runs after the interval, **then** the last-notification instant is advanced and nothing is sent.
42. **Given** a time-based automation rule that last ran at 08:00 and runs at 12:00 with a 3-day `before` offset, **when** it selects records, **then** it selects exactly those whose date field is at or after 2026-03-04 08:00 and strictly before 2026-03-04 12:00.
43. **Given** a record whose date field equals the exclusive upper bound, **when** the rule runs, **then** it is not selected, and it is selected by the following execution.
44. **Given** a time-based rule that raises on one record, **when** the job runs, **then** that rule's transaction is rolled back, its last-run instant is not advanced, the remaining rules still run, and the job ends as failed.
45. **Given** a time-based rule deactivated while the job is running, **when** the job reaches it, **then** it is skipped without error.
46. **Given** a time-based rule whose date field no longer exists on its entity, **when** the job reaches it, **then** a warning naming the rule is logged and no record is selected.
47. **Given** a time-based rule with a negative offset, **when** it is saved, **then** the save is refused with `Delay must be positive. Set 'Delay mode' to 'Before' to negate the delay.`
48. **Given** the time-based evaluation invoked outside a scheduled job, **when** it is called, **then** it raises rather than running.
49. **Given** four campaigns due, **when** the marketing queue runs, **then** the first progress report names 4 as remaining and each campaign reports one unit when it has been advanced.
50. **Given** a campaign with no recipient left to reach that has never been sent, **when** the queue reaches it, **then** its state becomes `done`, its sent instant is set, and it is marked as requiring a summary.
51. **Given** a campaign marked as requiring a summary whose sent instant is three days ago and the summary parameter set, **when** the queue runs, **then** the summary is sent and the mark is cleared.
52. **Given** the same campaign with the summary parameter unset, **when** the queue runs, **then** no summary is sent and the mark is kept.

## 13. Reconciliation notes

1. The outgoing and incoming electronic-mail queues were specified in the same document as these queues. They are now in [`mail-gateway.md`](mail-gateway.md), which owns both mail directions in full. The catalogue of section 1 still lists them, with a link, so that the whole set of queues can be read in one table.
2. The queue semantics themselves were implicit in the individual descriptions. They are stated once, in sections 2 and 3, and the idempotency patterns of section 3.1 are the catalogue that [`transactions-and-concurrency.md`](transactions-and-concurrency.md) refers to for the batch-commit rule.
3. The user-scheduled message queue was described as processing every due message in one commit. It processes at most 50, commits after each one, and triggers itself again while any remain. The corrected behaviour is specified in section 4.2, together with the failure path that notifies the author and deletes the message, which had not been described.
4. The batch-size parameter of the text message queue was named by a paraphrase. Its reproduced name is `sms.session.batch.size`.
5. The signing keys of the browser push queue were named by paraphrases. Their reproduced names are `mail.web_push_vapid_private_key` and `mail.web_push_vapid_public_key`, and the consequence of regenerating them, namely that every registered device is deleted, is specified in section 6.2.
6. The cleanup attached to the text message queue was described as removing obsolete device registrations. It removes text message rows whose deletion mark is set; device registrations belong to the browser push queue and are removed there when a device proves unreachable. The correct behaviour is in section 5.5.
7. The three states `process`, `pending` and `sent` of a text message were collapsed into two in one source. All six states are enumerated in section 5.1 with their stored values and labels, and the mapping from the service's returned states to them is in section 5.3.
8. The resend messages of the text message queue were paraphrased with the channel named in full words. They are contractual user-visible text and are reproduced verbatim in section 5.4, including the channel abbreviation they contain.
9. The digest slow-down table, the periodicity arithmetic and the notice are unchanged. The indicator comparison arithmetic, the three timeframes, the unsubscribe token and the one-click unsubscribe headers were not specified in either source and are specified here.
10. Data recycling was named in the job catalogue but never specified. It is specified in full in section 8, including the two batch sizes, the proposal index that makes the pass idempotent, the discard mechanism, and the notification pass with its compatibility finding.
11. The delay unit of a time-based rule was left unnamed. Its four stored values are `minutes`, `hour`, `day` and `month`; note that three of the four are singular and one is plural, which is a reproduced inconsistency in the stored values, not a transcription error.
12. The fallback window of the automatic last-update field in the date-field branch mixes the relative upper bound with the current date. It is recorded as a compatibility finding in section 9.3 with the corrected behaviour.
