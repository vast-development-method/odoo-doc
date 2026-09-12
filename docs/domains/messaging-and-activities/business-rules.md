# Messaging and Activities — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behavior of the domain, with the exact user-facing text the system produces.

Each rule carries a stable identifier of the form `MSG-nnn`. Identifiers are unique inside this folder and are cited by [acceptance-criteria.md](acceptance-criteria.md), by [interfaces.md](interfaces.md) and by the other documents of the folder. A former identifier of either source version maps to a current one in the mapping table at the end of this file.

Where a message carries a value supplied at run time, the placeholder is written in words between angle brackets, for example `<alias name>`. Every message is reproduced verbatim, because support procedures, user training and automated tests all key on it.

Contents:

1. [The message access policy](#1-the-message-access-policy)
2. [The activity access policy](#2-the-activity-access-policy)
3. [The scheduled-message access policy](#3-the-scheduled-message-access-policy)
4. [Editing, deleting and administering a message](#4-editing-deleting-and-administering-a-message)
5. [Posting and notifying: parameter validation](#5-posting-and-notifying-parameter-validation)
6. [Follower rules](#6-follower-rules)
7. [Notification rules](#7-notification-rules)
8. [Field change tracking rules](#8-field-change-tracking-rules)
9. [Scheduled message rules](#9-scheduled-message-rules)
10. [Template and rendering rules](#10-template-and-rendering-rules)
11. [Composer rules](#11-composer-rules)
12. [Alias and alias-domain validations](#12-alias-and-alias-domain-validations)
13. [Incoming gateway rules](#13-incoming-gateway-rules)
14. [Recipient resolution rules](#14-recipient-resolution-rules)
15. [Outgoing mail rules](#15-outgoing-mail-rules)
16. [Activity rules](#16-activity-rules)
17. [Channel rules](#17-channel-rules)
18. [Guest and public-user rules](#18-guest-and-public-user-rules)
19. [Live chat rules](#19-live-chat-rules)
20. [Mailing group rules](#20-mailing-group-rules)
21. [Text message and telephone number rules](#21-text-message-and-telephone-number-rules)
22. [Postal mail rules](#22-postal-mail-rules)
23. [Digest rules](#23-digest-rules)
24. [Rating rules as live chat uses them](#24-rating-rules-as-live-chat-uses-them)
25. [Assistant bot rules](#25-assistant-bot-rules)
26. [Electronic-mail client plugin rules](#26-electronic-mail-client-plugin-rules)
27. [Publisher announcement rules](#27-publisher-announcement-rules)
28. [Company and multi-company rules](#28-company-and-multi-company-rules)
29. [Date, time and rounding rules](#29-date-time-and-rounding-rules)
30. [Locking and concurrency rules](#30-locking-and-concurrency-rules)
31. [Retention and collection rules](#31-retention-and-collection-rules)
32. [Database constraints](#32-database-constraints)
33. [Invariants](#33-invariants)
34. [Rule identifier mapping](#34-rule-identifier-mapping)

---

## 1. The message access policy

The Message model does not rely on record rules alone. Every operation runs the ordinary rules first and then applies the five-branch policy below. A row forbidden by either mechanism is forbidden.

| Rule | Statement | Message |
|---|---|---|
| `MSG-001` | Before anything else, a caller who is not an internal user is refused every message that is marked employee-only, that has no subtype, or whose subtype is marked internal. For the read and the create operations the message must additionally be of the type `comment` (a comment). When such a caller searches, the same three conditions are pushed into the query as a filter before it runs, so the rows never leave the database. | the refusal of `MSG-006` |
| `MSG-002` | **Read** is granted when any of the following holds: the acting contact is the author; the acting user created the row; the acting contact is among the direct recipients; the acting contact has a Notification on the message; or the message names a model and a record, is not a user-specific notification, and the acting user may **read** that record. | the refusal of `MSG-006` |
| `MSG-003` | **Create** is granted when any of the following holds: the message is free-standing (no model, or no record, or it is a user-specific notification); the acting contact is among the direct recipients of the **parent** message; the acting user has, on the related record, the permission named by that model's post-access attribute; or the acting contact is a Follower of the related record. | the refusal of `MSG-006` |
| `MSG-004` | **Write** is granted when any of the following holds: the acting contact is the author; the acting contact is among the direct recipients; the acting contact has a Notification on the message; or the acting user may **write** the related record. | the refusal of `MSG-006` |
| `MSG-005` | **Delete** is granted only when the acting user may **write** the related record. There is no author exception: a person may not delete their own message on a record they cannot write. | the refusal of `MSG-006` |
| `MSG-006` | The refusal text is five lines: "The requested operation cannot be completed due to security restrictions. Please contact your system administrator.", a blank line, "(Document type: Message, Operation: <operation>)", a blank line, "Records: <the first six identifiers>, User: <acting user identifier>". | as stated |
| `MSG-007` | Searching applies the same policy: the ordinary query runs first, then every returned identifier is checked against the five branches and the forbidden ones are dropped silently. | none |
| `MSG-008` | Creating a message whose attachments include one that does not already belong to the same record checks read access on those attachments. | the platform's access refusal |
| `MSG-009` | Writing new attachments onto a message checks read access on them. | the platform's access refusal |
| `MSG-010` | Deleting a message deletes the attachments that belong to the message itself — their model is the message model and their record is one of the deleted messages, or zero — and never the attachments owned by the business record. | none |
| `MSG-011` | On the single-message read path, a public caller carrying a guest token skips the ordinary access-right layer and is judged only by the five branches. That layer predates guests and would otherwise refuse them wrongly. | none |
| `MSG-012` | On that same path, when both checks fail and the message names a model and a record, the message is granted anyway when the acting user has, on that record, the permission that the record's model maps to the requested message operation. | none |
| `MSG-013` | A guest is granted, inside the channel its token belongs to, exactly the message operations a contact would have there, and nothing at all outside it. | the request is refused |
| `MSG-014` | An unknown extra parameter passed to the single-message read path is logged as a warning and ignored, so a stale client cannot break. | none |
| `MSG-015` | The post-access attribute of a model must name a valid record operation. | "Invalid posting access right, should be a valid record operation type" |

---

## 2. The activity access policy

The shipped record rule for internal users grants **write and delete only**, and only on activities the user is assigned to or created. Read and create are governed entirely by the policy below.

| Rule | Statement | Message |
|---|---|---|
| `MSG-016` | **Read** is granted when the record rules allow it **and** either the Activity is assigned to the acting user or the acting user may read the related record. | the five-line refusal with the document type "Activity" |
| `MSG-017` | **Create** is granted when the record rules allow it **and** the acting user has, on the related record, the permission named by the model's post-access attribute. | the same refusal with the operation "create" |
| `MSG-018` | **Write** is granted when the record rules allow it **or** the acting user has that same permission on the related record. | the same refusal |
| `MSG-019` | **Delete** is granted under the same condition as write. | the same refusal |
| `MSG-020` | An Activity with **no** related record is readable, writable and deletable only by its assignee. | the same refusal |
| `MSG-021` | For write and delete the check short-circuits: when the caller holds the permission on the model as a whole, the per-record check is skipped entirely. | none |
| `MSG-022` | Searching applies the same filter: a row assigned to the acting user always passes; any other row passes only when its related record is readable. A user therefore never sees in a list an activity they could not open. | none |

---

## 3. The scheduled-message access policy

A Scheduled Message is governed by the record it will be posted on: creating, writing and deleting one require, on the target record, the permission named by that model's post-access attribute; reading is granted broadly by the record rule and narrowed by the search override to the rows whose target record the acting user may act on; writing is additionally restricted by the record rule to the rows the acting user created. A row the search override drops is dropped silently rather than refused.

| Rule | Statement | Message |
|---|---|---|
| `MSG-023` | The target model and the target record of a Scheduled Message may never be changed. | "You are not allowed to change the target record of a scheduled message." |
| `MSG-024` | Sending a Scheduled Message immediately is allowed only to an administrator and to the person who created it. | "You are not allowed to send this scheduled message" |

---

## 4. Editing, deleting and administering a message

| Rule | Statement | Message |
|---|---|---|
| `MSG-025` | Only a system administrator may change the model or the record of an existing message. | "Only administrators can modify 'model' and 'res_id' fields." |
| `MSG-026` | A message carrying Tracking Values may never have its content updated. | "Messages with tracking values cannot be modified" |
| `MSG-027` | Only a message of the type `comment` may have its content updated. | "Only messages type comment can have their content updated" |
| `MSG-028` | A Channel replaces the default rule rather than extending it: the tracking check does not apply and only the type check remains. | "Only messages type comment can have their content updated on model 'discuss.channel'" |
| `MSG-029` | A caller who is not an internal user silently loses the author and the sender address from any create or write payload; the values are removed rather than refused, and the corresponding context defaults are stripped as well. | none |
| `MSG-030` | Only an administrator may export messages. | "Only administrators are allowed to export mail message" |
| `MSG-031` | Deleting a message cascades to the attachments that belong to the message itself and broadcasts a deletion event to every recipient contact that has a user and to the authors of its notifications. | none |
| `MSG-032` | Where an accounting capability marks a conversation as a restricted audit trail, a message that is part of it may not be deleted. | "You cannot remove parts of a restricted audit trail. Archive the record instead." |
| `MSG-033` | When the new body is not empty, or the message is not becoming void, an "edited" marker is appended inside the last block-level element of the body, or at the end when the body is plain text. | none |
| `MSG-034` | Every cached Message Translation of the message is deleted whenever the body changes. | none |
| `MSG-035` | When the message becomes void, its link previews are removed; in a Channel its parent link is cleared as well. | none |
| `MSG-036` | An edit broadcasts the changed fields: the attachments sorted by identifier, the body, the direct recipients with their avatar and name, the pinning moment, the modification moment, the linked-message summaries and, when the subject changed, the subject plus a marker clearing the translation. | none |
| `MSG-037` | Replacing the attachment list with an empty list deletes the previous attachments and notifies their removal; passing no attachment list at all leaves them alone. | none |
| `MSG-038` | There is **no** time window on editing. The author may edit at any time and an administrator may edit at any time; the edited marker and the refusal of `MSG-026` are what prevent a tracked history from being falsified silently. **industry-standard default**: the observed behaviour defines the guard by content rather than by elapsed time, and this rule records that no time limit exists so a rebuild does not invent one. | none |
| `MSG-039` | A message whose body has no visible content, whose subtype has no description, that has no attachment and no readable tracking value is **void**: the interface hides it and, when it becomes void through an edit, its link previews are removed. The row itself is kept. | none |

---

## 5. Posting and notifying: parameter validation

| Rule | Statement | Message |
|---|---|---|
| `MSG-040` | Posting must target a single real record of a thread-enabled model. | "Posting a message should be done on a business document. Use message_notify to send a notification to an user." |
| `MSG-041` | Posting may not use the user-specific message type; that type is produced only by the notify operation. | "Use message_notify to send a notification to an user." |
| `MSG-042` | Creating a message through the thread operations accepts only these fields, and nothing else: attachments, guest author, author, body, creation date, date, add-signature flag, sender address, layout, incoming carbon copy, incoming "to", employee-only flag, activity type, relay, message identifier, message type, model, outgoing "to", parent, direct recipients, alias domain of the record, company of the record, reply address, force-new-thread flag, record, subject, subtype and tracking values. The parameters `model` (the model name), `res_id` (the record identifier) and `subtype` may never be passed to the posting operation. | "Those values are not supported when posting or notifying: <names>" |
| `MSG-043` | Posting or mailing with a source refuses the parameters `body`, `composition_mode` (the composition mode), `incoming_email_cc`, `incoming_email_to`, `model`, `res_id`, `outgoing_email_to` and `values`. Logging with a view refuses `body`, `bodies`, `incoming_email_cc`, `incoming_email_to` and `outgoing_email_to`. | the same message |
| `MSG-044` | Inline attachments must be a list of two- or three-element groups. | "Posting a message should receive attachments as a list of list or tuples (received <value>)" / "Notification should receive attachments as a list of list or tuples (received <value>)" |
| `MSG-045` | Existing attachments must be a list of identifiers, never write commands. | "Posting a message should receive attachments records as a list of IDs (received <value>)" / "Notification should receive attachments records as a list of IDs (received <value>)" |
| `MSG-046` | The notify operation refuses the parameters `incoming_email_cc`, `incoming_email_to`, `message_id`, `message_type`, `outgoing_email_to` and `parent_id`. | "Those values are not supported when posting or notifying: <names>" |
| `MSG-047` | Both operations validate the recipient list the same way. | "Posting a message should receive partners as a list of IDs (received <value>)" / "Notification should receive partners given as a list of IDs (received <value>)" |
| `MSG-048` | Calling notify with no recipient does nothing at all and logs a warning; it is not an error. | none |
| `MSG-049` | A batch log may not carry attachments or tracking values when more than one record is targeted. | "Batch log cannot support attachments or tracking values on more than 1 document" |
| `MSG-050` | Notify accepts a model and a record only when **both** are supplied; this allows notifying about a record whose model has no conversation. | none |
| `MSG-051` | A notification call accepts only these parameters: forced company, forced language, forced record name, force-send flag, automatic-deletion flag, model description, notify-author flag, notify-author-when-mentioned flag, skip-followers flag, scheduled moment, send-after-commit flag, skip-existing flag and subtitles. An internal or elevated caller may additionally pass extra headers; a portal caller may not, so a portal user cannot inject a header. | "Those values are not supported when posting or notifying: <names>" |
| `MSG-052` | Rendering requires a language in the acting context. | "At this point lang should be correctly set" |
| `MSG-053` | When the caller is not an internal user, the list of existing attachments to link is **replaced** by the subset that is both attached to a composer or a scheduled message and created by that very user. This prevents a portal user from attaching somebody else's file. | none |

---

## 6. Follower rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-054` | Subscribing somebody else requires **write** access on the record. Subscribing only oneself requires **read** access and, when even that is missing, the operation returns false rather than raising, so a follow button degrades gracefully. | the platform's access refusal, or none |
| `MSG-055` | Unsubscribing somebody else requires **write** access. Unsubscribing oneself requires **read** access when the caller is not an internal user; an internal user may always unsubscribe themselves, whatever the active company, so an inbox entry can always be unfollowed. | the platform's access refusal, or none |
| `MSG-056` | The author of a posted message is subscribed only when all of the following hold: the skip switch is off; the message type is none of system notification, user-specific notification, automated targeted notification and out-of-office answer; the subtype is exactly the shipped Discussions subtype; and the **real** author's contact is not a customer. | none |
| `MSG-057` | Subscribing somebody else silently drops archived contacts. A user adding themselves is not filtered. | none |
| `MSG-058` | A contact may follow a record at most once. | "Error, a partner cannot follow twice the same object." |
| `MSG-059` | A caller who is not an internal user may filter records by follower only on themselves or on their commercial entity. | "Portal users can only filter threads by themselves as followers." |
| `MSG-060` | A Channel may not have followers at all; membership replaces them. | "Adding followers on channels is not possible. Consider adding members instead." |
| `MSG-061` | At subscription with no explicit subtypes, a contact that is a customer receives the **external** default set and anyone else receives the **full** default set. An internal subtype is never granted to a customer. | none |
| `MSG-062` | Deleting a thread-enabled record proceeds in this order: discard pending tracking; delete every Message pointing at the records; delete the records; delete every Follower pointing at them; delete every Scheduled Message pointing at them. Neither Messages nor Followers can cascade at database level, the link being by model name plus identifier, which is why the deletion is explicit. Changing followers also invalidates the cached message and notification counters of the followed records, because a follower change alters who may read the record's messages. | none |

---

## 7. Notification rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-063` | The author is never notified of their own message, unless the caller switches on "notify the author", or switches on "notify the author when mentioned" and the author is among the direct recipients. | none |
| `MSG-064` | The **real author** is the acting user's contact when that user is active; when the acting user is inactive — the system user running the gateway, for example — it is the message author, provided that author is active and is not the system contact. | none |
| `MSG-065` | A recipient whose normalized address already appears among the message's already-reached "to" or carbon-copy lists is dropped, so a person who received the original incoming message is not mailed a second time. | none |
| `MSG-066` | An archived contact is never notified. | none |
| `MSG-067` | A follower whose contact is a customer is excluded when the subtype is marked internal. That single condition is what keeps internal notes away from customers. | none |
| `MSG-068` | A recipient's delivery channel is the notification preference of their preferred user; a contact with no user is always reached by electronic mail. | none |
| `MSG-069` | Clearing delivery failures is restricted to internal users and clears only the acting user's **own** failing notifications, in the bounced or exception status, of the given channel, for messages of the model the operation was called on. The affected messages are re-broadcast. | "Access Denied" when the caller is not an internal user |
| `MSG-070` | The preferred user of a contact is its first active user ordered by the sharing flag ascending with unknown first, then by identifier ascending: an internal user beats a portal user, and among equals the lowest identifier wins. | none |
| `MSG-071` | A portal or public user always has the electronic-mail preference; changing group membership or the sharing flag recomputes it. | "Only internal user can receive notifications in Odoo" |
| `MSG-072` | A contact receives at most one Notification per Message. | database-level uniqueness |
| `MSG-073` | The message and the recipient of an existing Notification may not be changed by anyone who is not an administrator. | "Can not update the message or recipient of a notification." |
| `MSG-074` | A Notification of the inbox channel must name a recipient contact. | "Customer is required for inbox notification" |
| `MSG-075` | A Notification of the electronic-mail channel must carry a failure type, a recipient contact, or a non-empty address. | "Customer or email is required for inbox / email notification" |
| `MSG-076` | A Notification of the electronic-mail channel and a Notification of the postal channel are created already marked read, because there is no read-back channel. An inbox Notification is created unread and directly in the delivered status. | none |
| `MSG-077` | Immediate sending is granted only when the caller asked for it **and** the number of produced Outgoing Mails is strictly below the immediate-sending limit. Above it the queue takes over, whatever the caller asked. | none |
| `MSG-078` | The list of external addresses is added to the reply-to-all header only when their number is strictly below the model's external-recipient limit, so the audience of a public record never leaks. | none |
| `MSG-079` | When sending now, the mails are handed to the relay **after** the transaction commits, unless the caller explicitly asked for inline sending. | none |
| `MSG-080` | A deferred notification pass is idempotent: on release, the "skip recipients that already have a notification" switch is forced on, the scheduled moment is removed from the replayed parameters, and a row whose target record has meanwhile been deleted is skipped and deleted. | none |

---

## 8. Field change tracking rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-081` | A field is tracked when its definition carries a tracking marker: a positive integer, which is also the display order, or the value true, which means order 100. The set is computed once per model and cached per acting user and per elevation state, because it is filtered by field-level access. | none |
| `MSG-082` | A dynamic-property field is tracked only when the field that defines its schema is itself tracked and the property field is not explicitly excluded. | none |
| `MSG-083` | Old values are captured once per record per transaction: a value already in the snapshot is **not** overwritten, so the snapshot always holds the value as it was at the first write of the transaction. A computed stored field that is about to be recomputed triggers the same snapshot for the fields it computes. | none |
| `MSG-084` | At creation, only a tracked field that received a non-empty value counts as changed. | none |
| `MSG-085` | When the new value equals the old one, or when **both** are falsy, no tracking entry is produced. An empty text becoming a false value is therefore not a change. | none |
| `MSG-086` | When the model's subtype rule returns a subtype, a message is posted with that subtype and the followers of that subtype are notified; when it returns nothing but tracking entries exist, an internal note is logged and nobody is notified; when there is neither, nothing is written. | none |
| `MSG-087` | The duration-tracking field must exist, must be a link field and must be tracked. | "Field “<field>” on model “<model>” must be of type Many2one and have tracking=True for the computation of duration." |
| `MSG-088` | Staleness may be searched only with the two set operators. | "For performance reasons, use "=" operators on rotting fields." |
| `MSG-089` | Staleness may be searched only on a model that configured the feature. | "Model configuration does not support the rotting feature" |
| `MSG-090` | A field type that has no column pair is rejected at creation of the tracking entry; an unknown field name is rejected as well. Deleting a tracked field definition fills the snapshot on its existing tracking entries so the history stays readable, and those entries then become visible only to a system administrator. | a programming error naming the field and its type |
| `MSG-091` | A tracking entry is visible only when the reader may read that field on its model; an entry whose field was deleted is visible only to a system administrator. A stricter filter applies inside a notification: only entries whose field still exists and carries **no** access group at all are included, because the notification is rendered once for many recipients. | none |
| `MSG-092` | The body of a tracking message is the explicitly registered log message when one was set for the record in this transaction, **even when it is empty**; otherwise the model's default log message for the changed set, which is empty by default. | none |

---

## 9. Scheduled message rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-093` | The target model of a Scheduled Message must be thread-enabled. | "A message cannot be scheduled on a model that does not have a mail thread." |
| `MSG-094` | The scheduled moment must not be in the past. | "A Scheduled Message cannot be scheduled in the past" |
| `MSG-095` | Only this fixed list of notification parameters survives from the captured payload into the actual posting: whether to add a signature, the sender, the layout, the forced language, the activity type, whether to delete the mail, the relay, the message type, the model description, the reply address, whether answers start a new thread, and the subtype. Anything else stored is ignored. | none |
| `MSG-096` | When the posting job fails, the failure is swallowed: the creator is notified with the subject "A scheduled message could not be sent" and the body "The message scheduled on <model>(<the record identifier>) with the following content could not be sent:" followed by the original body between separator lines, and the Scheduled Message is deleted anyway. The notification is sent because the creator may have lost access to the record and would otherwise never learn of the loss. | as stated |

---

## 10. Template and rendering rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-097` | A Template may not target an abstract model. | "You may not define a template on an abstract model: <model>" |
| `MSG-098` | The source of a post-with-source or a mail-with-source must be a Template or a view. | "Invalid template or view source record <value>, is <model> instead" |
| `MSG-099` | An external identifier used as a source must still resolve. | "Invalid template or view source Xml ID <reference> does not exist anymore" |
| `MSG-100` | An external identifier used as a source must resolve to a Template or a view. | "Invalid template or view source reference <value>, is <model> instead" |
| `MSG-101` | A source must be either a record or an external identifier, and must not be empty. | "Invalid template or view source <value> (type <type>), should be a record or an XMLID" / "Mailing or posting with a source should not be called with an empty <template or view>" |
| `MSG-102` | A Template whose fields fail to render may not be saved. | "Oops! We couldn't save your template due to an issue.\n\nError: <details>\n\nCorrect it and try again." |
| `MSG-103` | The Template named by a contextual action must belong to the action's model. | "Mail template model of <action name> does not match action model." |
| `MSG-104` | Rendering must be invoked with a list of record identifiers. | "Template rendering should only be called with a list of IDs. Received “<value>” instead." |
| `MSG-105` | The rendering engine must be one of the three supported ones. | "Template rendering supports only inline_template, qweb, or qweb_view (view or raw); received <engine> instead." |
| `MSG-106` | Rendering a field the Template does not define is refused. | "Rendering of <field name> is not possible as not defined on template." |
| `MSG-107` | Rendering a composer field that has no counterpart on the Template is refused. | "Rendering of <field name> is not possible as no counterpart on template." |
| `MSG-108` | While the rendering restriction is on, only a member of the template-editor group may create or modify a Template containing anything other than a plain field path. The check runs at creation, at modification **and** at translation, so a template cannot be made dynamic through a translation. | "Only members of Mail Template Editor group are allowed to edit templates containing sensible placeholders" |
| `MSG-109` | While the restriction is on and the caller is not an editor, rendering falls back to the direct field-lookup evaluator, which accepts only the seven allowed plain paths and rejects anything else as a syntax error. | as stated |
| `MSG-110` | A report whose output kind cannot be attached is refused. | "Unsupported report type <type> found." |
| `MSG-111` | Sending from a Template requires read access on the target records. | the platform's access refusal |
| `MSG-112` | A Template whose rendered sender is empty leaves the sender unset so the platform default applies; the empty value is never written. | none |
| `MSG-113` | An internal user may modify only the Templates they created or that name them as owner; a template editor and an administrator may modify all of them. | the platform's access refusal |
| `MSG-114` | Resetting restores the shipped values, translations included, only for a Template that records the shipped definition it came from; a Template with no such record cannot be reset. The same mechanism exists for text-message templates and reports "SMS Templates have been reset". | "The following email templates could not be reset because their related source files could not be found:\n- <names>" |
| `MSG-115` | A rendered contact identifier that no longer exists is dropped silently; an unknown rendering option aborts the call naming the invalid options; a failure of the structured engine aborts with "Failed to render template: <reference>" or "Failed to render QWeb template for <field label>", and a failure of the inline engine with "Failed to render inline_template template: <text>\nError details: <error>". | as stated |

---

## 11. Composer rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-116` | Sending in the "post on a document" mode requires at least one target record. | "Mail composer in comment mode should run on at least one record. No records found (model <model>)." |
| `MSG-117` | Sending with no resolvable recipient is refused. An address used as a recipient that is malformed is reported separately, and creating a contact from such an address is refused. | "No recipient found." / "Wrong email address <value>." |
| `MSG-118` | Scheduling a message is possible only in single-record "post on a document" mode. | "A message can only be scheduled in monocomment mode" |
| `MSG-119` | Scheduling a message requires a moment. | "A scheduled date is needed to schedule a message" |
| `MSG-120` | Saving the composer content as a Template requires a model. | "Template creation from composer requires a valid model." |
| `MSG-121` | Each user sees only their own composer rows. | the platform's access refusal |
| `MSG-122` | The alternative record filter must be a valid filter. | "Invalid domain “<filter>” (type “<type>”)" |
| `MSG-123` | In mass mode a record whose address is missing or invalid never produces an outgoing mail. When the composer keeps a log, the failure is recorded as a cancelled or failed row; when it does not, the record is skipped entirely. | none |
| `MSG-124` | In mass mode a suppressed address, an opted-out recipient and a duplicate address are marked with the suppressed-address, opted-out and duplicate failure types respectively, and nothing is handed to the relay for them. | none |
| `MSG-125` | A target model that is not thread-enabled forces replies to be treated as new threads, and the composer uses the notify operation instead of the post operation, after removing the message type and the parent and adding the model and the record. | none |

---

## 12. Alias and alias-domain validations

| Rule | Statement | Message |
|---|---|---|
| `MSG-126` | The local part of an Alias must be a plain unaccented dot-atom. | "You cannot use anything else than unaccented latin characters in the alias address <local part>." |
| `MSG-127` | The default values of an Alias must parse as a literal mapping. | "Invalid expression, it must be a literal python dictionary definition e.g. "{'field': 'value'}"" |
| `MSG-128` | The local part must not equal the bounce or the catch-all local part of its own domain. | "Aliases <names> is already used as bounce or catchall address. Please choose another alias." |
| `MSG-129` | The pair (local part, domain) must be free, treating "no domain" as a distinct value. The refusal names the conflicting alias, the model it serves and, when there is one, the owner document. | "Alias <conflicting display name> (<current identifiers or "your alias" or "new">) is already linked with <model label> (<conflicting identifier>) and used by the <owner display name> <owner model label>. Choose another value or change it on the other document." — or, with no owner, "Alias <conflicting display name> (<current identifiers>) is already linked with <model label> (<conflicting identifier>). Choose another value or change it on the other document." |
| `MSG-130` | The same local part may not be assigned to several records in one operation. | "Email aliases <name> cannot be used on several records at the same time. Please update records one by one." |
| `MSG-131` | When the owner record has a company and the alias domain is bound to companies, they must agree. | "We could not create alias <alias display name> because domain <domain name> belongs to company <company names> while the owner document belongs to company <company name>." |
| `MSG-132` | The same rule applies to the forced target record. | "We could not create alias <alias display name> because domain <domain name> belongs to company <company names> while the target document belongs to company <company name>." |
| `MSG-133` | Two Alias Domains with the same name must not share a bounce local part. | "Bounce alias <address> is already used for another domain with same name. Use another bounce or simply use the other alias domain." / the database uniqueness message "Bounce emails should be unique" |
| `MSG-134` | Two Alias Domains with the same name must not share a catch-all local part. | "Catchall alias <address> is already used for another domain with same name. Use another catchall or simply use the other alias domain." / "Catchall emails should be unique" |
| `MSG-135` | No Alias may already occupy the bounce or the catch-all address of a domain. | "Bounce/Catchall '<address>' is already used by <document display name>. Choose another alias or change it on the other document." — or, when no document can be named, "Bounce/Catchall '<address>' is already used. Choose another alias or change it on the linked model." |
| `MSG-136` | A domain name may not be empty and must be a plain unaccented dot-atom, equal to its own sanitized form. | "You cannot assign an empty domain name." / "You cannot use anything else than unaccented latin characters in the domain name <name>." |
| `MSG-137` | The default sender address of a domain must not fall inside the filter of any personal relay. | "A personal mail server is using that address, you can not use it." |
| `MSG-138` | The allowed-domain parameter must be a comma-separated list of domains; it is trimmed and lower-cased before storage and must not be empty after cleaning. | "Value <value> for `mail.catchall.domain.allowed` cannot be validated.\nIt should be a comma separated list of domains e.g. example.com,example.org." |
| `MSG-139` | Deleting a record that owns an Alias deletes the Alias with it. | none |
| `MSG-140` | Duplicating a record that owns an Alias never copies the Alias. | none |
| `MSG-141` | Deleting an Alias Domain is refused while any Alias still points at it, and deleting an Alias is refused while any owner record still points at it. | the platform's deletion refusal |
| `MSG-142` | The uniqueness check of `MSG-129` is skipped for aliases whose target model or owner model is not currently present in the model registry, which keeps installation and test setups working. | none |
| `MSG-143` | Changing the contact-security policy, the default values or the target model of an Alias resets its validity status to "not tested". No other field does. | none |
| `MSG-144` | A record created successfully through an Alias sets its status to "valid" when it is not already valid. A configuration error sets it to "invalid". A refusal of the contact-security check leaves it unchanged. | none |
| `MSG-145` | When the **first** Alias Domain of the installation is created — detected because the total count equals the number just created — it is assigned to every company that has none, archived companies included, and to every Alias that has none. | none |

---

## 13. Incoming gateway rules

Every rejection produces either a bounce to the sender or a silent drop. The internal warning text appears in the operator's log; the generic wrapper for a routing warning that is raised is "Mailbox unavailable - <error>", deliberately short so that internal diagnostics are not exposed to the sender.

| Rule | Statement | Message or consequence |
|---|---|---|
| `MSG-146` | A message whose identifier already exists as a Message is discarded. | internal text "found duplicated Message-Id during processing"; silently ignored |
| `MSG-147` | Concurrent processing of one identifier is prevented by a transaction-scoped advisory lock keyed on a hash of the identifier; the loser treats the message as a duplicate. | the same internal text; silently ignored |
| `MSG-148` | Loop detection triggers when, inside the observation window, the number of records this sender created through the model's loop filter reaches the threshold, or the number of incoming messages from this sender on one record reaches it. Defaults: 120 minutes and 20. | internal text "created too many <model>" or "too much replies on same <model>"; the notification-limit bounce is sent and the message is ignored |
| `MSG-149` | A bounce never creates or updates a record. It raises the bounce counter of every blacklist-enabled record whose normalized address equals the bounced address and of the originally addressed record when it was not already caught, and it writes the matching Notifications to the bounced status with the bounce failure type and the plain text of the bounce as the reason. | none |
| `MSG-150` | Because a message did arrive from this sender, receiving anything that is **not** a bounce resets to zero the bounce counter of every blacklist-enabled record whose normalized address equals the sender's. | none |
| `MSG-151` | A message that writes directly to the catch-all mailbox — with the strict test, every one of its "to" addresses is a catch-all address — is bounced and never routed; the same bounce is sent when routing ends with no route and the relaxed test finds any catch-all recipient. The company answered for is the acting company, replaced by the first company of the alias domain whose catch-all matched when that domain is bound to companies that do not include the acting one. | internal text "direct write to catchall, bounce" or "write to catchall + other unroutable emails, bounce"; the catch-all bounce, referencing the loop tag, replying to the company address |
| `MSG-152` | Every Alias matching a valid recipient, by full address or by local part when the alias is local-part based, produces one route. One incoming message may therefore legitimately create several records. | none |
| `MSG-153` | A message any of whose references contains the loop-detection tag is a reply to one of the platform's own bounces and is discarded. | internal text "reply to a bounce notification detected by headers"; silently ignored |
| `MSG-154` | A sender listed as a Gateway Allowed Sender is never suppressed by loop detection. | none |
| `MSG-155` | The contact-security policy "authenticated partners" requires the author to have resolved to a contact. | error text "restricted to known authors"; the security bounce; the alias status is unchanged |
| `MSG-156` | The policy "followers" requires the checked object to be a real record, to have followers, and the author to be one of them. | "incorrectly configured alias (unknown reference record)" and "incorrectly configured alias" are configuration errors; "restricted to followers" is a plain refusal |
| `MSG-157` | Only a **configuration** error sets the alias to invalid and sends the invalid-alias bounce; a plain refusal of the sender leaves the status alone and sends the security bounce. | as stated |
| `MSG-158` | When a record creation through an Alias raises, the alias is marked invalid on an **independent** connection so the mark survives the rollback, the invalid-alias bounce is sent, and the error is re-raised. | none |
| `MSG-159` | When a reply target was found and any "to" address matches an Alias of a **different** model, the message is a forward, not a reply: the reply target is discarded and the valid recipient list is narrowed to exactly the addresses that matched those aliases. | none |
| `MSG-160` | A message that matches no reply, no alias, no fallback model and no catch-all, and for which no bounce was sent, fails loudly. | "No possible route found for incoming message from <sender> to <recipients> (Message-Id <identifier>:). Create an appropriate mail.alias or force the destination model." |
| `MSG-161` | A route is validated in order: the model must be named and must exist; a named record must exist and its model must accept updates, otherwise the record is cleared and creation is attempted; with no record the model must accept creation. | "target model unspecified" / "unknown target model <model>" / "reply to missing document (<model>,<record>), fall back on document creation" / "reply to model <model> that does not accept document update, fall back on document creation" / "model <model> does not accept document creation" |
| `MSG-162` | A reply whose parent's subtype is internal, the parent not being an automated notification, is posted as an internal note, and the parent's author is added to the direct recipients; the parent's author is also added when that author is an external contact, so a customer answering is sure to reach the person who wrote to them. | none |
| `MSG-163` | A message with no identifier header receives a synthetic one of the shape `<timestamp@localhost>`, logged at debug level. | none |
| `MSG-164` | A message addressed to a closed Mailing List is bounced with the shipped closed-list body and nothing is stored. | as stated |
| `MSG-165` | A model that does not accept incoming messages at the moment the route is applied aborts the delivery. | "Undeliverable mail with Message-Id <identifier>, model <model> does not accept incoming emails" |

### The security bounce body

When the Alias defines a custom body, that body is used. Otherwise, rendered in the **author's** language when the author is known:

```text
Dear Sender,

The message below could not be accepted by the address <alias display name>. Only <contact description> are allowed to contact it.

Please make sure you are using the correct address or contact us at <default address> instead.

Kind Regards
```

The contact description is "addresses linked to registered partners" for the policy "authenticated partners" and "some specific addresses" for the policy "followers". The default address is the company contact's formatted address, falling back to the company name.

### The invalid-alias bounce body

```text
Dear Sender,

The message below could not be accepted by the address <alias display name>. Please try again later or contact <company name> instead.

Kind Regards
```

### The bounce message itself

| Value | Rule |
|---|---|
| recipient | the return-path header of the incoming message, falling back to its sender |
| subject | "Re: <original subject>" |
| author | none |
| automatic deletion | on |
| sender | "MAILER-DAEMON" plus the company bounce address; failing that the "to" header of the incoming message, but only when none of its addresses is a catch-all; failing that "MAILER-DAEMON" plus the acting user's normalized address |

---

## 14. Recipient resolution rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-166` | Creating a contact from an address requires a usable address. | "An email is required for find_or_create to work" |
| `MSG-167` | Creating a contact from a malformed address is refused. | "<address> is not recognized as a valid email. This is required to create a new customer." |
| `MSG-168` | An address that belongs to the installation — a named Alias, a catch-all, a bounce address or a default-sender address of any Alias Domain — never resolves to a contact, never creates one, and is never proposed as a recipient. This is what stops the gateway from subscribing its own addresses to a record. | none |
| `MSG-169` | The address of the platform's root contact is never proposed as a default or a suggested recipient, unless another contact already holds that address. | none |
| `MSG-170` | A public contact is never a default or a suggested recipient. | none |
| `MSG-171` | When one address matches several contacts, the winner is chosen by seven ordered criteria, each read as "true before false": the candidate is the acting user's contact; it follows the record; it is **not** a customer; it has a user; its company equals the record's company; its formatted address is exactly one of the inputs; it has **no** company. Remaining ties are broken by the lowest identifier. The ranking is deterministic, so two runs over the same data resolve the same contact. | none |
| `MSG-172` | When the resolution is invoked on a record set, the set must match the per-record address map. | "Invoke with either self maching records_emails, either on a void recordset." |
| `MSG-173` | A contact created from an address of a record receives that record's company and the record's customer information for that address. | none |
| `MSG-174` | A follower is not proposed as a suggested recipient, except when it is one of the record's own customers **and** it is a customer contact: a user writing to a customer expects to see the customer in the recipient list. | none |
| `MSG-175` | An address already covered by a follower or by an already-kept contact, compared on both the normalized and the raw form, is not proposed separately. | none |
| `MSG-176` | The record's responsible is a suggested recipient, unless the responsible is the acting user. | none |
| `MSG-177` | Only a message of the type comment or incoming electronic mail whose subtype is the model's creation subtype or the shipped Discussions subtype is considered when proposing recipients from the discussion. A note, a tracking message and a system notification never trigger a reply-all proposal. | none |
| `MSG-178` | The default-recipient computation chooses between contacts and addresses by the six-branch rule of [calculations.md](calculations.md), section 11. The rule prefers a contact that holds a usable address, because a contact carries the language and the notification preference. | none |
| `MSG-179` | The carbon-copy list is gathered only when the caller asks for it; a mailing and an automated action do not. | none |
| `MSG-180` | A suggested-recipient entry carries a name, an address, a contact identifier when one is known, and the creation values the record's customer information holds for that address. An entry for a contact with no name carries its display name so the interface can show something. | none |

---

## 15. Outgoing mail rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-181` | An Outgoing Mail may not name a relay that the creator of its message is not allowed to use. The allowed set is: relays without an owner when the batch has several distinct creators or when personal relays are disabled by system parameter; otherwise relays without an owner plus the relays owned by the creator. | "You may not create a message using another user's mail server." |
| `MSG-182` | Sending groups the queue by two grouping keys in turn: first (chosen relay, alias domain, normalized sender, allowed relay set), then (resolved relay, alias domain, transport sender). One connection serves one second-level group, split into batches of the session batch size. | none |
| `MSG-183` | A relay that has an owner is personal and may never be forced by a flow. | "The server "<name>" cannot be forced as it belongs to a user." / "The server "<name>" cannot be forced as it belongs to a user and is archived." / "The server "<name>" cannot be forced as the owner does not use it anymore." |
| `MSG-184` | A personal relay is throttled to the configured number of **recipients** per minute, thirty when the parameter is unset or zero, with the split, delay and wake-up arithmetic of [calculations.md](calculations.md), section 20. | none |
| `MSG-185` | A whole sending batch may not use a relay outside the allowed set. | "Unauthorized server for some of the sending mails." |
| `MSG-186` | An Outgoing Mail with no free recipients, no contact recipients and no carbon copies fails with the missing-address failure type. | the "no valid recipient" text of the relay layer as the reason |
| `MSG-187` | A rejection of one sub-message does not abort the others: the failure type is "invalid email address" when the sub-message had recipients and "missing email address" when it had none, and the loop continues with the next sub-message. | none |
| `MSG-188` | A relay that cannot be reached fails the whole group with the relay-failure type; when the caller asked for exceptions, the call aborts instead. | "Unable to connect to SMTP Server" |
| `MSG-189` | Failure classification at send time: the relay reports an invalid sender gives "invalid from address"; the relay cannot determine a sender gives "missing from address"; the failure text mentions the outbound spam exception of the corporate mail service gives "detected as spam"; anything else gives the unknown type. An encoding failure is reported as "Invalid text: <object>". | as stated |
| `MSG-190` | Automatic deletion applies only when there was no failure at all, or when the failure type is "missing address" or "invalid address". Any other failure keeps the row so the operator can inspect and retry it. | none |
| `MSG-191` | Deleting an Outgoing Mail that is **not** a notification mail deletes its Message with it; deleting a notification mail leaves the Message in the conversation. | none |
| `MSG-192` | Retry moves only rows in the failure state back to the outgoing state. | none |
| `MSG-193` | The queue selects rows in the outgoing state whose scheduled moment is empty or already passed, ordered by identifier, limited to the batch size; when a specific set of identifiers is requested the limit is ten times the batch size and the result is intersected with that set. | none |
| `MSG-194` | An attachment the body already references by a content or image link is not attached again. | none |
| `MSG-195` | An attachment that is a pure external link — it has an address, no stored bytes, and the address uses the plain, secure or file-transfer scheme — is converted into a link appended to the body. | none |
| `MSG-196` | When the estimated size exceeds the relay's maximum, the attachments that belong to a **business record** are converted into signed download links appended to the body; an attachment owned by the message itself is never converted, because the link would break when the message is deleted. | none |
| `MSG-197` | The remaining attachments are read and attached in ascending identifier order, so the order matches what the sender saw when uploading; one whose content is absent is skipped. | none |
| `MSG-198` | The unfollow block is replaced by a signed unsubscription link only for a recipient who is a contact, on an existing record, where the model offers unfollowing or the recipient is internal, and who actually follows the record. For everybody else the whole block is removed from the body. | none |
| `MSG-199` | The return path of every outgoing electronic mail is the bounce address of the message's alias domain, falling back to the bounce address of the record's company. A return path already present is not overwritten. | none |
| `MSG-200` | A scheduled moment that cannot be parsed is stored as "no schedule", meaning "send as soon as possible"; an empty string is stored the same way rather than failing. | none |
| `MSG-201` | Before any network call, the row is written to the failure state with a provisional reason and its notifications are written and flushed. Writing first provokes any locking failure before the electronic mail leaves, so a rollback can never un-send a sent message. The provisional reasons are: the relay layer's "no valid recipient" text with the missing-address type when there is no recipient; "Error without exception. Probably due to sending an email without computed recipients." with the unknown type when there are recipients; and "Error without exception. Probably due to concurrent access update of notification records. Please see with an administrator." with the unknown type on the notifications. | as stated |
| `MSG-202` | Three exception classes are deliberately **not** caught and are re-raised so the job aborts rather than marking the row failed: a memory exhaustion, a database error and a disconnected relay session. In all three the transaction rolls back and the row stays queued. | none |
| `MSG-203` | Only an internal user may configure a personal relay, only for themselves, and only when their address is set. Testing somebody else's personal relay is refused. | "Only internal users can configure a personal mail server." / "Only internal users can configure personal mail servers." / "Please set your email before connecting your mail server." / "You are not allowed to create a personal mail server." / "You are not allowed to test personal mail servers." / "No mail server configured" |
| `MSG-204` | A personal relay may not use an address that belongs to an Alias Domain, and duplicating a personal relay clears the owner so the copy is usable. Changing a user's address collects the personal relays bound to the old one. | "Your email address is used by an alias domain, and so you can not create a mail server for it." / "Wrong email address <value>." |
| `MSG-205` | Extra headers are stored as a serialized map; malformed content is logged and ignored rather than aborting the queue. | none |

---

## 16. Activity rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-206` | Either both the model and a non-zero record identifier are set, or neither is. | "Activities have to be linked to records with a not null res_id." |
| `MSG-207` | An Activity with no model must have an assignee. | "Activities must be assigned if not attached to a document." |
| `MSG-208` | The type of an Activity is restricted to types whose model is empty or equals the related model; the default is the first such type by ordering. A named type that does not exist, or that belongs to another model, falls back to the model's default type with a warning. | none |
| `MSG-209` | The delay configured on the type produces the due date, except inside an Activity Plan, where the **line's own** delay is used and the type's delay is ignored. | none |
| `MSG-210` | Creating an Activity for somebody other than the acting user notifies that person, rendered in the assignee's language, and subscribes their contact to the related record. The notification is suppressed when the quick-update switch is on. | subject ""<the record name>: <summary>" assigned to you"; subtitles "Activity: <type name>" (falling back to "Todo") and "Deadline: <due date in the recipient's date format>" |
| `MSG-211` | Changing the assignee notifies the new assignee under the same conditions and subscribes them to the related records. | as above |
| `MSG-212` | Completing an Activity archives it rather than deleting it, stamps the completion date the **first** time only, stores the feedback, and posts a message on the record from the shipped completion body with the activity type and the Activities subtype. | none |
| `MSG-213` | The attachments of the Activity are moved onto that completion message. | none |
| `MSG-214` | When the related record no longer exists, no message is posted, the attachments are deleted, and the Activity is **deleted** rather than archived. | none |
| `MSG-215` | Cancelling deletes the Activity outright and posts nothing. | none |
| `MSG-216` | Completion is permitted to the assignee even when they cannot read the related record, because the completion archives the Activity and the archive is what the permission covers; the message posting is performed with elevated rights for the same reason. | none |
| `MSG-217` | The five protected shipped types may not change their model. | "You cannot modify <names> target model as they are are required in various apps." |
| `MSG-218` | Three of them may not be deleted. | "You cannot delete <names> as it is required in various apps." |
| `MSG-219` | Archiving any type other than the generic to-do type is allowed and simply hides it from the selection lists. | none |
| `MSG-220` | The generic to-do type may not be archived. | "The 'To-Do' activity type is used to create reminders from the top bar menu and the command palette. Consequently, it cannot be archived or deleted." |
| `MSG-221` | Deleting any other type first reassigns every Activity of that type to the generic to-do type, then deletes. | none |
| `MSG-222` | The suggested successors and the triggered successor are mutually exclusive: setting one clears the other. A successor must be a type of the same model or a type restricted to no model. | none |
| `MSG-223` | A plan line's Activity Type, when it is model-specific, must equal the plan's model. The check runs both when a line changes and when the plan's model changes. | "The activity type "<type name>" is not compatible with the plan "<plan name>" because it is limited to the model "<type model>"." |
| `MSG-224` | A plan line in the fixed-user assignment mode must name a user. | "When selecting "Default user" assignment, you must specify a responsible." |
| `MSG-225` | A plan line in the ask-at-launch mode must receive a user at launch; the error is blocking and the whole plan refuses to launch. | "No responsible specified for <type name>: <summary or a hyphen>." |
| `MSG-226` | Launching a plan requires real target documents. | "Plan-based scheduling are available only on documents." |
| `MSG-227` | Scheduling a personal activity requires an assignee. | "Scheduling personal activities requires an assigned user." |
| `MSG-228` | A server action of the activity kind may only target an activity-enabled model. | "A next activity can only be planned on models that use activities." |
| `MSG-229` | Choosing an assignee who may not upload documents, for a type whose category is "upload document", warns the user without blocking. | "Selected user '<user>' cannot upload documents on model '<model>'" |
| `MSG-230` | Activities whose due date is older than the configured number of years are deleted, up to ten thousand per run. A missing or zero setting disables the routine with a warning; a negative setting disables it with a different warning. | none |

---

## 17. Channel rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-231` | The channel type may never be changed after creation, so a direct conversation can never become a public channel and expose its history. | "Cannot change the channel type of: <names>" |
| `MSG-232` | The shipped whole-company group may not be deleted. | "You cannot delete those groups, as the Whole Company group is required by other modules." |
| `MSG-233` | The authorization group and the auto-subscription groups are only allowed on a channel. | at validation level "For <names>, channel_type should be 'channel' to have the group-based authorization or group auto-subscription."; at database level "Group authorization and group auto-subscription are only supported on channels." |
| `MSG-234` | A conversation of the chat type may not have more than two members. | "A channel of type 'chat' cannot have more than two users." / "Adding more members to this chat isn't possible; it's designed for just two people." |
| `MSG-235` | A direct conversation is created only between exactly the given contacts, and never with more than two. | "A chat should not be created with more than 2 persons. Create a group instead." |
| `MSG-236` | The initial message of a sub-thread is unique. | "Messages can only be linked to one sub-channel" |
| `MSG-237` | A parent must not itself be a sub-thread, must be of the channel or the group type, and must share the child's type. The initial message and the parent may never be changed afterwards, and the authorization group of a sub-thread may not be changed. | "Cannot create <names>: parent should not be a sub-channel and should be of type 'channel' or 'group'. The sub-channel should have the same type as the parent." / "Cannot change initial message nor parent channel of: <names>." / "Cannot change authorized group of sub-channel: <names>." |
| `MSG-238` | The initial message of a sub-thread must belong to the parent channel or to one of its sub-threads, and must be a channel message. | "Cannot create <names>: initial message should belong to parent channel or one of its sub-channels." |
| `MSG-239` | A public user may never be a Channel Member; a public visitor joins as a Guest. | "Channel members cannot include public users." |
| `MSG-240` | The channel, the contact and the guest of a membership may never change, so a membership can never be transferred. | "You can not write on <field name>." |
| `MSG-241` | Posting a message whose type is neither a system notification nor a user-specific notification raises the channel's last-interest moment. | none |
| `MSG-242` | After posting, the author's own membership is advanced: the message becomes the last seen message and the separator moves to that identifier plus one. An author never sees their own message as unread. | none |
| `MSG-243` | Every mentioned contact is filtered before being stored: in a channel with an authorization group, only contacts whose users belong to that group survive; in any other type, only contacts that are already members survive. A mention of everyone expands to the full member contact list. | none |
| `MSG-244` | After posting in a sub-thread, every mentioned contact that is a member of the parent and whose notification preference is not "nothing" is added to the sub-thread; in a sub-thread of a plain channel, mentioned contacts that are not yet members of the parent are added as well. | none |
| `MSG-245` | Leaving posts the notice "left the channel" for any type other than a plain channel; leaving a plain channel posts nothing. For a type that does not allow leaving, the conversation is unpinned instead. | as stated |
| `MSG-246` | Joining posts the notice "joined the channel" when the member added themselves, and "invited <name> to the channel" with the inviting contact as author when somebody else added them; a plain channel posts nothing. | as stated |
| `MSG-247` | A member whose bounce counter reaches ten is removed from the channel. | none |
| `MSG-248` | Inviting by address requires an internal user with read access on the channel and a channel type that allows it, which is a group, or a channel with no authorization group. | "You don't have access to invite users to this channel." / "Inviting by email is not allowed for this channel type (<type>)." / on a delivery failure "There was an error when trying to deliver your Email, please check your configuration." or, when the relay refused the connection, "Could not contact the mail server, please check your outgoing email server configuration." |
| `MSG-249` | At most one call session may exist per member. | "There can only be one rtc session per channel member" |
| `MSG-250` | A Call History must name a channel and a start moment, and a message may open only one call. | "Call history must have a channel" / "Call history must have a start date" / "Messages can only be linked to one call history" |
| `MSG-251` | A Guest's name may be neither empty nor longer than the allowed length. | "Guest's name cannot be empty." / "Guest's name is too long." |
| `MSG-252` | A user may keep at most one favourite entry per animated image. | "User should not have duplicated favorite GIF" |
| `MSG-253` | A reaction is from a contact or from a guest, never both and never neither, and is unique per message, content and party. Adding one that exists and removing one that does not are both no-ops; after either the whole group is recomputed and broadcast, either as the full list or as a deletion marker naming the message and the content when the group became empty. | "A message reaction must be from a partner or from a guest." |
| `MSG-254` | A Role name is unique. | "A role with the same name already exists." |
| `MSG-255` | Mentioning a Role notifies only the members of the role who can access the conversation; the others are dropped silently. | none |
| `MSG-256` | A Canned Response may be shared only with groups the acting user belongs to, so a user cannot share one with a group they are not in. | none |
| `MSG-257` | Contacts supplied at channel creation must use the link or the replace command. | "Invalid value when creating a channel with members, only 4 or 6 are allowed." |
| `MSG-258` | Memberships supplied at channel creation must use the creation command. | "Invalid value when creating a channel with memberships, only 0 is allowed." |
| `MSG-259` | Only four member fields may be supplied at channel creation: the contact, the guest, the unpin moment and the last-interest moment. | "Invalid field “<field>” when creating a channel with members." |
| `MSG-260` | A membership may not be created without naming its channel. | "It appears you're trying to create a channel member, but it seems like you forgot to specify the related channel. To move forward, please make sure to provide the necessary channel information." |
| `MSG-261` | A Channel is accessible when either the type is not the plain channel type and the acting party is a member or a member of its parent, or the type is the plain channel type and it has no authorization group or the acting user belongs to it. A system administrator has unrestricted access. | the platform's access refusal |
| `MSG-262` | Membership rows follow five rules, deliberately split: own rows may be written and deleted; the rows of an accessible channel may be read; a party may create their **own** row in a plain channel whose authorization group is empty or satisfied; an internal user may create **somebody else's** row in such a channel; and an internal user may create somebody else's row in a type that is neither a plain channel nor a chat when they are a member. The consequence is precise: a person may add themselves only to a public channel or to one whose authorization group they belong to; into a group conversation they must be invited by an existing member; into a direct conversation nobody may be invited at all. | the platform's access refusal |
| `MSG-263` | Uploading an attachment where it is not allowed is refused, and every attachment operation carried out by a guest requires an access token. | "You are not allowed to upload an attachment here." / "You are not allowed to upload attachments on this channel." / "An access token must be provided for each attachment." / "Non existing record or wrong token." |
| `MSG-264` | Reactions are read as groups, one entry per distinct content with its count and its parties; asking for specific fields on a group is refused. | "Fields are not supported for reactions." |
| `MSG-265` | Type-dependent capabilities: leaving as opposed to unpinning is allowed for a channel and a group; read receipts are broadcast to the whole channel for a chat and a group; member-based naming applies to a group; the member list is loaded lazily for a channel and a group; inviting by address is allowed for a group and for a channel with no authorization group. | none |

---

## 18. Guest and public-user rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-266` | A guest is identified solely by the pair of identifier and access token carried in a cookie. The token is readable only by the system group, so it never leaves the server except in the cookie itself. | none |
| `MSG-267` | A public user carrying a guest token posts as that guest: the author contact and the sender address are both cleared. This is the only way a message can have a guest author. | none |
| `MSG-268` | A public user without a guest token may not post at all: the posting path computes an author that resolves to the public contact, and the message access policy then refuses the creation. | the refusal of `MSG-006` |
| `MSG-269` | A request that carries both a user session and a guest cookie ignores the guest cookie. Switching to another user always clears the guest from the acting context, so acting on behalf of a user can never be confused with acting as a guest. | none |
| `MSG-270` | The guest cookie separator character is excluded from the channel token alphabet, so a channel token can never be mistaken for a guest cookie. | none |
| `MSG-271` | A guest may post, react, upload attachments and open link previews in the channels it belongs to, and nothing else. | the request is refused |
| `MSG-272` | A guest carries a language, a time zone and a country, inferred from the request when possible, and has a Presence row like a user, so an operator sees whether the visitor is still connected. | none |

---

## 19. Live chat rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-273` | Only a live chat operator may join a Live Chat Channel. | "Only Live Chat operators can join Live Chat channels" |
| `MSG-274` | An operator is available for an entry point only when their presence status is exactly "online"; and the entry point's session mode is unlimited or their ongoing-session count for that entry point is strictly below the maximum; and the entry point does not block assignment during calls or the operator is not in a call. | none |
| `MSG-275` | An entry point offers chat when it has at least one Chatbot Script configured on a rule, or at least one available operator. | none |
| `MSG-276` | The previous operator of a returning visitor is reused when they are among the candidates and either have no row in the load table, or carry fewer than two conversations, or are not in a call. | none |
| `MSG-277` | An operator who was given a still-open session inside the buffer period of 120 seconds is avoided, unless avoiding them would leave the winning preference line empty. | none |
| `MSG-278` | The maximum concurrent session count of an entry point must be strictly positive. | "Concurrent session number should be greater than zero." |
| `MSG-279` | A session of the live chat type must name an operator contact. | "Livechat Operator ID is required for a channel of type livechat." |
| `MSG-280` | A closed session must have an empty working status. | "Closed Live Chat session should not have a status." |
| `MSG-281` | The review address must use the plain or the secure hypertext transfer scheme and must name a host. | "Invalid URL '<value>'. The Review Link must start with 'http://' or 'https://'." |
| `MSG-282` | A participation history may exist only on a live chat session. | "Cannot create history as it is only available for live chats: <names>." |
| `MSG-283` | Expertise may only be linked or unlinked on a membership, never replaced wholesale. | "Write expertises: Only LINK and UNLINK commands are allowed." |
| `MSG-284` | A step of the question kind must define at least one answer. | "Step of type 'Question' must have answers." |
| `MSG-285` | An answer to an address step must be a usable address; the current step does not advance so the visitor can retry. | ""<value>" is not a valid email." |
| `MSG-286` | Changing a step's type away from the question kind clears its answers, and a triggering answer whose step is no longer earlier in the script is removed automatically. | none |
| `MSG-287` | Duplicating a script re-links every triggering answer of the copied steps to the **copied** answers, matched by position, and appends " (copy)" to the title. Creating a script with no operator contact creates an **archived** contact named after the script. | none |
| `MSG-288` | A display rule whose script is archived or has no step never matches; and a rule requiring an available operator does not match when there is none, while a rule requiring none does not match when there is one. | none |
| `MSG-289` | The notice "Visitor left the conversation." is posted only when the session already carries at least one message. | as stated |
| `MSG-290` | Joining a session that needs help succeeds only while the working status is still "Looking for help"; a later attempt returns a refusal rather than an error, so the client can report that somebody was faster. | none |
| `MSG-291` | A live chat operator may read every live chat session and every live chat membership and may invite anybody into a live chat session, but may not modify or delete a session. | the platform's access refusal |
| `MSG-292` | A visitor may not start a call; only an operator may. | the platform's access refusal |
| `MSG-293` | A visitor may not upload an attachment on a closed session. | the platform's access refusal |
| `MSG-294` | Only an internal user with read access on a session may change its internal note, its working status and its tags; only a live chat operator may set the tags and the expertise. | the platform's access refusal |
| `MSG-295` | A session handled only by a bot with no activity for more than a day is archived; an empty session is deleted; a read session with no activity for a day is unpinned from the operator's sidebar. | none |

---

## 20. Mailing group rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-296` | Every moderator must have an address. | "Moderators must have an email address." |
| `MSG-297` | A moderated list must have at least one moderator. | "Moderated group must have moderators." |
| `MSG-298` | A closed list may not be joined. | "You can not join a closed group." |
| `MSG-299` | Switching the automatic acknowledgement on requires a notification text. | "The notification message is missing." |
| `MSG-300` | Switching the automatic guidelines on requires a guidelines text. | "The guidelines description is missing." |
| `MSG-301` | The privacy mode "selected group of users" requires a group. | "The "Authorized Group" is missing." |
| `MSG-302` | Sending the guidelines requires being an administrator or a moderator, a non-empty text and an open list; a banned address never receives them. | "Only an administrator or a moderator can send guidelines to group members." / "The guidelines description is empty." / "You can not send guidelines for a closed group." |
| `MSG-303` | Only a pending post may be moderated. | "This message can not be moderated" / "Those messages can not be moderated: <subjects>." |
| `MSG-304` | One permanent rule per address per list. | "You can create only one rule for a given email address in a group." |
| `MSG-305` | One subscription per contact per list. Two members with the **same address** and no contact are possible; leaving with the "all" flag removes them all. | "This partner is already subscribed to the group" |
| `MSG-306` | The author of a post never receives the relay of their own post. | none |
| `MSG-307` | The wrapped Message of a post must belong to the mailing-list model and to this very list. | "Group message can only be linked to mail group. Current model is <model>." / "The record of the message should be the group." |
| `MSG-308` | A subscription or unsubscription requested by an anonymous visitor takes effect only after the signed confirmation link is followed. A signed-in user is served at once, and joining with an unknown contact is refused. | "The partner can not be found." |
| `MSG-309` | The confirmation token binds the list, the normalized address and the action, so one address cannot be used to act on another address or on another list. | "Invalid action for URL generation (<value>)" / "Email <value> is invalid" |
| `MSG-310` | A public or portal reader sees only the accepted posts of the lists they may see; a moderator additionally sees the pending and the rejected posts of the lists they moderate. | the platform's access refusal |
| `MSG-311` | The alias contact-security policy of a list defaults to "everyone" when the privacy mode is public and to "followers" otherwise, and follows the privacy mode when it changes. | none |
| `MSG-312` | Switching moderation on adds the acting user to the moderators. | none |
| `MSG-313` | A member is looked up first by contact, then by normalized address; when several rows match the address, the one that carries a contact wins. | none |
| `MSG-314` | The body of an incoming post has the platform's own list footer stripped before storage, so a reply does not accumulate footers. | none |
| `MSG-315` | A relayed post carries the list headers: the archive address, the subscribe address, the member's own unsubscribe address, the one-click unsubscribe marker, the precedence "list", and a header suppressing automatic out-of-office answers; plus, when the list has an address, the list identifier, the post address and a forge-to header; plus, when the source message has a parent, that parent's identifier as the in-reply-to header. | none |
| `MSG-316` | Relaying is batched by the configured session batch size, and the member address map is keyed by normalized address so each address receives exactly one copy. | none |
| `MSG-317` | A moderation status must be one of the three values. | "Wrong status (<value>)" |
| `MSG-318` | The address of a moderation rule must be normalizable and valid. | "Invalid email address “<value>”" / "The email "<value>" is not valid." |
| `MSG-319` | Relaying a post whose list does not match the list of the message is refused. | "The group of the message do not match." |
| `MSG-320` | The shipped guidelines template must exist. | "Template "mail_group.mail_template_guidelines" was not found. No email has been sent. Please contact an administrator to fix this issue." |

---

## 21. Text message and telephone number rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-321` | A number that cannot be parsed or that is not possible is refused. Two repairs are attempted for a number that is too long, in this order: a leading double zero is retried as the international prefix, and a number with no leading plus sign is retried with one added. | "Unable to parse <number>: <details>" / "Impossible number <number>: not a valid country prefix." / "Impossible number <number>: not enough digits." / "Impossible number <number>: too many digits." / "The phone number <number> is invalid! Let's fix it - you are not dialing aliens." / "Invalid number <number>: probably incorrect prefix." |
| `MSG-322` | A Blacklist Entry address is unique and must be normalizable; adding an address that exists but is archived re-activates the row. | "Email address already exists!" / "Invalid email address “<value>”" |
| `MSG-323` | Removing an address from the suppression list archives the entry rather than deleting it, and logs the supplied reason in its own conversation. Two confirmations are shown before the action, depending on the entry point: "Are you sure you want to unblacklist this Email Address?" and "Are you sure you want to unblacklist this email address?". | as stated |
| `MSG-324` | A blocked number is unique and must be parsable; adding one that exists but is archived re-activates the row. | "Number already exists" / "<parser error> Please correct the number and try again." |
| `MSG-325` | Removing an address from the suppression list is restricted. | "You do not have the access right to unblacklist emails. Please contact your administrator." |
| `MSG-326` | Removing a number from the suppression list is restricted. | "You do not have the access right to unblacklist phone numbers. Please contact your administrator." |
| `MSG-327` | A model that supports number search must declare its number fields, and a model that honours the suppression list must declare its primary address field. | "Missing definition of phone fields." / "Invalid primary phone field on model <model>" / "Invalid primary email field on model <model>" |
| `MSG-328` | Searching a number requires at least three characters; the search term is sanitized first, so a nationally formatted term still matches. | "Please enter at least 3 characters when searching a Phone number." |
| `MSG-329` | Every free number typed in the window must format; if any fails, the whole operation aborts. | "Following numbers are not correctly encoded: <list>" |
| `MSG-330` | Sending to a single recipient whose number is unusable is refused; an unusable recipient name is refused as well. | "Invalid recipient number. Please update it." / "Invalid recipient name." |
| `MSG-331` | Sending a batch reports how many recipients are unusable. | "<count> invalid recipients" |
| `MSG-332` | Text messaging requires a non-transient thread-enabled model. | "Sending SMS can only be done on a not transient mail.thread model" |
| `MSG-333` | Only a failed Text Message that is not marked for deletion may be resent; resending puts it back in the outgoing state and sends it immediately. | "<n> out of the <total> selected SMS Text Messages have successfully been resent." / "The SMS Text Messages could not be resent." / "There are no SMS Text Messages to resend." |
| `MSG-334` | A provider state maps to a message state as specified in [state-machines.md](state-machines.md), section 7; a provider error code that the platform does not know becomes the unknown failure type and the provider's own text is kept as the reason. | none |
| `MSG-335` | Two named error sets drive how a late delivery report is interpreted: the **bounce** set — invalid destination, not allowed, rejected — sets the Notification to bounced; the **delivery** set — those three plus expired and not delivered — sets it to exception with that failure type. | none |
| `MSG-336` | The monotonic rule: a status update is ignored when the Notification already holds a status that is at least as advanced. A late "accepted" report can therefore never overwrite a confirmed delivery, and a cancellation can never undo something already handed to the carrier. | none |
| `MSG-337` | An address used to register the sending account must be valid. | "Email <value> is invalid" |
| `MSG-338` | A sender name must be three to eleven characters long and contain only letters and digits; an account that already has one may not change it, and an unregistered account may not set one. | "Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters." / "This account already has an existing sender name and it cannot be changed." / "Your text message account has not been activated yet." |
| `MSG-339` | The telephony account identifier must start with the two letters `AC` and contain only letters and digits after that prefix. | "Invalid Twilio Account SID: must start with 'AC'" / "Invalid Twilio Account SID: must only contain alphanumeric characters after 'AC'" |
| `MSG-340` | Sending a telephony test requires a destination number. | "Please set the number to which you want to send a test SMS." |
| `MSG-341` | Listing the provider's sending numbers may fail. | "An error occurred while fetching the numbers." |
| `MSG-342` | Provider failures are explained to the operator: the destination country is not covered; the content breaks the provider's rules; a trial-account limitation; an unverified recipient on a trial account; an unknown sending failure; an unknown failure; a duplicate suppressed in a batch; a provider authentication failure; a wrong delivery-report address. | "The destination country is not supported." / "The content of the message violates rules applied by our providers." / "Trial Account Limitation" / "Unverified recipient on Trial Account" / "Unknown failure at sending, please contact Odoo support" / "Unknown error, please contact Odoo support" / "This SMS has been removed as the number was already used." / "Twilio Authentication Error" / "Twilio StatusCallback URL is incorrect" |
| `MSG-343` | The text-message Template named by a contextual action must belong to the action's model. | "SMS template model of <action name> does not match action model." |
| `MSG-344` | The correlation token of a Text Message and the token of a Text Message Tracker are each unique. | "UUID must be unique" / "A record for this UUID already exists" |
| `MSG-345` | Account registration reports the service's own explanations: an invalid telephone number; the number could not be reached; the service is suspended for new accounts; the number or account is banned; the country is not supported by sender registration; the installation is not activated; a wrong verification code; an unknown account; too many attempts; and an unmapped failure. | "Invalid phone number. Please make sure to follow the international format, i.e. a plus sign (+), then country code, city code, and local phone number. For example: +1 555-555-555" / "We were not able to reach you via your phone number. If you have requested multiple codes recently, please retry later." / "The Text Message Service is currently unavailable for new users and new accounts registrations are suspended." / "This phone number/account has been banned from our service." / "Your country is not supported due to sender registration legislation" / "Your database is not activated" / "The verification code is incorrect." / "We were not able to find your account in our database." / "You tried too many times. Please retry later." / "An unknown error occurred. Please contact support if this error persists." |

---

## 22. Postal mail rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-346` | A Postal Letter may be sent only when the street, the city, the postal code and the country are all filled; otherwise it fails at once, with no call to the service. | "The address of the recipient is not complete" and the stored explanation "One or more required fields are empty." |
| `MSG-347` | The document must use the paper format the printing service accepts. | "Please use an A4 Paper format." |
| `MSG-348` | The document may not exceed eight pages. | "The document to be sent exceeds the maximum allowed limit of 8 pages." |
| `MSG-349` | Each service error code maps to one explanation and one Notification failure type: credit error, trial error, no price available, missing required fields, format error, attachment error and unknown error. | "You don't have enough credits to perform this operation.<br>Please go to your <link>iap account</link>." / "You don't have an IAP account registered for this service.<br>Please go to <link>iap.odoo.com</link> to claim your free credits." / "The country of the partner is not covered by Snailmail." / "One or more required fields are empty." / "The attachment of the letter could not be sent. Please check its content and contact the support if the problem persists." / "The attachment could not be generated." / "An unknown error happened. Please contact the support." |
| `MSG-350` | The queue stops at the first insufficient-credit failure of a run, because every following call would fail the same way, and raises the operator warning. | "Not enough credits for Snail Mail" |
| `MSG-351` | The queue retries only the letters in error whose code is one of: no registered account, insufficient credits, attachment error and missing required fields. A format error and a country that is not covered are never retried. | none |
| `MSG-352` | The addressee's postal address is **copied** onto the letter at creation and never re-read, so a later change of the contact does not rewrite what was posted. Re-queuing does not refresh it; a corrected address must be written on the letter, or the letter cancelled and a new one created. | none |
| `MSG-353` | One Notification of the postal channel is created per letter, already marked read, so a postal send never fills anyone's inbox. | none |
| `MSG-354` | Read access on the attachment is checked at creation and at every change of the attachment. | the platform's access refusal |
| `MSG-355` | Re-queuing writes the pending state, clears the failure fields on the Notification and re-broadcasts; when exactly one letter is re-queued it is sent immediately, while a batch is left to the job. | none |
| `MSG-356` | A recipient with no name at all and no parent company name fails with the missing-fields code. | "Invalid recipient name." |
| `MSG-357` | When the document cannot be produced, the letter fails with the attachment-error code. | "The attachment could not be generated." |
| `MSG-358` | A service that cannot be reached fails every letter of the call with the unknown code and re-raises. | none |
| `MSG-359` | On success the stored explanation is written and the operator is informed. | "The document was correctly sent by post.<br>The tracking id is <identifier>" / "Snail Mails are successfully sent" |
| `MSG-360` | Cancelling writes the cancelled state, clears the error code and cancels the Notification. | none |

---

## 23. Digest rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-361` | Only a Digest in the activated state is considered by the sending job. | none |
| `MSG-362` | An indicator the recipient may not read is dropped silently from that recipient's copy. | none |
| `MSG-363` | The periodicity must be one of the four values. | "Invalid periodicity set on digest" |
| `MSG-364` | The automatic slow-down runs only for an automatic send, never for a manual one, because a manual send is not something the recipient could perceive as unsolicited repetition. | none |
| `MSG-365` | Each send consumes exactly one tip: the first tip the recipient has not yet received and whose group the recipient belongs to; a tip with no group is offered to everyone. The recipient is then added to that tip's already-received list. | none |
| `MSG-366` | The unsubscribe token is a keyed digest over the pair of digest identifier and recipient identifier with a fixed purpose string, so it cannot be forged or reused for another digest or another recipient. | none |
| `MSG-367` | Subscribing and unsubscribing through the application are restricted to internal users acting on themselves; the one-click address is the only other path. | none |
| `MSG-368` | A delivery failure during a scheduled send is caught and logged; the digest is left for the next run. | none |
| `MSG-369` | After each send the next run date becomes today plus the interval of the current periodicity: one day, one week, one month or three months. | none |
| `MSG-370` | The margin is exactly zero whenever the two values are equal or either of them is zero; the second case avoids a division by zero and avoids reporting an infinite improvement from nothing. | none |

---

## 24. Rating rules as live chat uses them

Ratings are owned by [../learning-surveys-and-gamification/](../learning-surveys-and-gamification/). The rules below are the ones a live chat implementation of this folder must satisfy.

| Rule | Statement | Message |
|---|---|---|
| `MSG-371` | A live chat session's rating parent is its Live Chat Channel, so a session's rating rolls up to the entry point; the entry point's satisfaction is computed over the last fourteen days. | none |
| `MSG-372` | A rating value is between zero and five inclusive. | "Rating should be between 0 and 5" and, when applied, "Wrong rating value. A rate should be between 0 and 5 (received <value>)." |
| `MSG-373` | Applying a rating requires a valid token or an existing rating row. | "Invalid token or rating." |
| `MSG-374` | Applying a rating marks it consumed and posts, or updates, one message in the rated record's conversation showing the face for the value and the comment. A second answer updates the same message rather than posting a new one. | none |
| `MSG-375` | When the caller asks to delay the notification, the notification of the rating message is deferred by two hours, so the customer may still change their answer. | none |
| `MSG-376` | Requesting a rating reuses the customer's unconsumed rating of the record when one exists, instead of creating a second one. | none |
| `MSG-377` | The satisfaction grade of one value is: happy at four and above, neutral from three to below four, unhappy from one to below three, not rated below one. The grade of an average is compared at two decimals: happy from 3.66, neutral from 2.33, unhappy from 1, not rated below 1. | none |
| `MSG-378` | Writing the public answer to a rating requires write access on the rated record. | "Updating rating comment require write access on related record" |
| `MSG-379` | The publisher contact and the publishing moment of a public answer are forced to the acting user and the current moment when they are not supplied. | none |
| `MSG-380` | Ratings are readable, creatable and writable by internal users; a public or portal party reaches a rating only through its access token. | the platform's access refusal |

---

## 25. Assistant bot rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-381` | The bot answers only inside a direct conversation of which the bot contact is a member. In any other channel nothing is answered and no state changes. | none |
| `MSG-382` | The bot never answers a message it authored itself. | none |
| `MSG-383` | The bot answers only a message of the type comment, unless the trigger is a command rather than a message. | none |
| `MSG-384` | A user in the disabled state never receives an answer and never has the conversation initialised. The root user is shipped in that state. | none |
| `MSG-385` | The guided conversation is initialised only for an internal user, and only once: at the first client start where the state is empty or "not initialized". | none |
| `MSG-386` | Every answer the bot posts is marked silent, so it never raises an unread counter and never produces a notification sound. | none |
| `MSG-387` | The state advances only on the exact expected gesture of the current state; anything else repeats the step hint and sets the failure flag. | the per-state hints of [state-machines.md](state-machines.md), section 15 |
| `MSG-388` | While the failure flag is set, every sentence counts as a request for help, so the help answer wins over the per-state hint. | "Unfortunately, I'm just a bot 😞 I don't understand! If you need help discovering our product, please check <our documentation> or <our videos>." |
| `MSG-389` | The temporary Canned Response created at the attachment step belongs to that user and is deleted when the canned-response step succeeds. | none |
| `MSG-390` | The restart phrase restarts the guide only from the empty state, the "not initialized" state and the idle state. | "To start, try to send me an emoji :)" |
| `MSG-391` | The two onboarding fields are read-only to the user; only the bot logic writes them. A user may read their own state. | none |
| `MSG-392` | The body is normalized before matching: non-breaking spaces become ordinary spaces, surrounding whitespace is trimmed, the text is lower-cased, and trailing full stops and exclamation marks are removed. | none |

---

## 26. Electronic-mail client plugin rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-393` | Only an internal user may grant an add-in access to the platform. | "Access Error: Only Internal Users can link their inboxes to this database." |
| `MSG-394` | A consent code is valid for three minutes from the moment it was issued. | the invalid-code answer |
| `MSG-395` | A consent code signature is compared in constant time, so the comparison cannot be used to guess a valid signature. **industry-standard default** for the stated reason; the constant-time comparison itself is observed. | the invalid-code answer |
| `MSG-396` | The application key issued in exchange is valid for one day and is scoped to the plugin, so it cannot be used for ordinary requests. | "Access token invalid" |
| `MSG-397` | Every bridge route other than the version check and the code exchange requires the application key in the authorization header, and then runs with the identity and the context of that key's user, so all ordinary access rules apply. | "Access token missing" / "Access token invalid" |
| `MSG-398` | A lookup must carry either a contact identifier, or both a name and an address. | "You need to specify at least the partner_id or the name and the email" |
| `MSG-399` | The default-sender address of any Alias Domain is never looked up as a contact and never used to create one. | "This is your notification address. Search the Contact manually to link this email to a record." |
| `MSG-400` | An address that cannot be normalized is refused. | "Bad Email." |
| `MSG-401` | The enrichment search key of an address is the address domain preceded by the at sign, except for a known generic mailbox provider, where it is the whole address. One enrichment therefore serves every person of one company. | none |
| `MSG-402` | A company is enriched at most once: an existing enrichment record or an existing company contact matching the search key is reused before any call is made. | none |
| `MSG-403` | An address whose domain is a generic mailbox provider is never sent to the enrichment service. | the "missing data" outcome |
| `MSG-404` | An exhausted prepaid balance produces the insufficient-credit outcome carrying the address where credits can be bought. | none |
| `MSG-405` | A contact that no longer exists cannot be enriched, and only a company contact may be updated by enrichment. | "This partner does not exist" / "Contact must be a company" |
| `MSG-406` | Enrichment of an existing company writes only fields that are still empty. | none |
| `MSG-407` | A contact that already has a parent company cannot be enriched into a new company. | "The partner already has a company related to him" |
| `MSG-408` | A contact whose address cannot be normalized cannot be enriched. | "The email of this contact is not valid and we can not enrich it" |
| `MSG-409` | A logotype download failure never fails the enrichment; the contact is created or updated without an image. The download timeout is two seconds. | none |
| `MSG-410` | A message may be filed only on a model of the allowed list, which contains Contact only unless another capability widens it. Filing goes through the ordinary posting operation, so followers are notified. | the request is refused |
| `MSG-411` | A company the acting user may not read is returned as its identifier and the name "No Access"; no other field leaks. | "No Access" |
| `MSG-412` | At most one enrichment record exists per contact. | "Only one enrichment record is allowed per contact" |

---

## 27. Publisher announcement rules

| Rule | Statement | Message |
|---|---|---|
| `MSG-413` | Every announcement received in the weekly exchange is posted in the company-wide channel with the Discussions subtype, addressed to the root user's contact. | none |
| `MSG-414` | A failure while posting one announcement is swallowed, so the remaining announcements are still delivered. | none |
| `MSG-415` | When the exchange runs as a scheduled job, every error is caught and the run reports failure without raising, so a network problem never interrupts the scheduler; when it is triggered interactively, a transport failure is raised and any other error is re-raised. | "Error during communication with the publisher warranty server." |
| `MSG-416` | Subscription information received with the answer is written to the system parameters for the expiry date, the expiry reason (defaulting to the trial value), the subscription code, the address to reach when the subscription is already linked elsewhere, the address recorded for that link, and the address used to send the reminder for it. | none |
| `MSG-417` | The exchange is the only outbound contact this domain makes to the publisher; a rebuild that has no such service simply omits it, and nothing else in this folder depends on it. | none |

---

## 28. Company and multi-company rules

| Rule | Statement |
|---|---|
| `MSG-418` | Every Message records the company of its record at posting time, so a later company change does not rewrite the sending configuration of the history. |
| `MSG-419` | Every Message records the alias domain of its record at posting time, for the same reason. |
| `MSG-420` | The reply address is resolved against the record's company: a record-specific Alias first, then that company's catch-all address, then the sender address. |
| `MSG-421` | An Alias Domain may be used only by aliases whose owner and target belong to a company that uses that domain. |
| `MSG-422` | Outgoing electronic mails are grouped for sending by alias domain, so one connection never mixes the sending identities of two companies. |
| `MSG-423` | A Text Message uses the sending credentials of the company recorded on its message, falling back to the acting company. |
| `MSG-424` | A Postal Letter is charged to the company recorded on the letter, which is the acting company at creation. |
| `MSG-425` | This domain never holds or manipulates a monetary amount. The single currency link that exists, on a Tracking Value, is a label saying which currency to use when rendering an old and a new monetary value that some other domain's field carried; it takes part in no arithmetic. |
| `MSG-426` | The company of a record is its company field when the model has one and it is set, else the company supplied by the caller, else the acting company. |

---

## 29. Date, time and rounding rules

| Rule | Statement |
|---|---|
| `MSG-427` | Every stored moment is in coordinated universal time. A moment supplied with no zone is understood as coordinated universal time; one supplied with a zone is converted. |
| `MSG-428` | An Activity due date is a calendar date, not a moment, and is compared against today **in the assignee's time zone**; when the assignee has no time zone, the machine's local calendar date is used. |
| `MSG-429` | Rescheduling to next week means the Monday of the following week, computed as today plus one week moved back to the most recent Monday, never simply today plus seven days. |
| `MSG-430` | A live chat session's start hour is a fractional hour; the day of the week runs Monday as 0 through Sunday as 6 on the session, and Sunday as 0 through Saturday as 6 in the reporting view. |
| `MSG-431` | Durations are expressed in hours with two decimals in reporting, in hours with six decimals for the response time, and in whole seconds in the duration-tracking map. |
| `MSG-432` | A rating average is compared with two-decimal precision when it is mapped to a grade, so an average of exactly 3.66 is the happy grade. |
| `MSG-433` | A satisfaction percentage is computed as the happy count times one hundred divided by the total, with no rounding at computation time; the display layer rounds. |
| `MSG-434` | A digest margin is rounded to two decimals by the ordinary half-up rule after the percentage has been computed. |

---

## 30. Locking and concurrency rules

| Rule | Statement |
|---|---|
| `MSG-435` | The outgoing queue sorts the selected identifiers ascending before sending, so two concurrent runs take their row locks in the same order and cannot deadlock. |
| `MSG-436` | The text message queue locks the rows it selects for update, so two runs never hand the same message to the provider twice. |
| `MSG-437` | An Outgoing Mail is written to the failure state **before** the relay is contacted, so a rollback after a successful hand-over cannot cause a second send. |
| `MSG-438` | The notifications of that mail are written and **flushed immediately**, which takes the row lock, so a bounce arriving mid-batch blocks until the send completes rather than interleaving with it. |
| `MSG-439` | The outgoing queue commits after each row when it runs from the scheduler and reports progress, so a failure does not lose the work already done. This is deliberately not transactional. |
| `MSG-440` | The fetched-message write of a Channel Member uses a select-for-update that skips locked rows, so a concurrent tab silently gives up instead of deadlocking. |
| `MSG-441` | The unread separator of a Channel Member only ever moves forward; a late call carrying a lower identifier has no effect and only re-broadcasts the member. |
| `MSG-442` | Event bus entries are written at the end of the transaction and announced only after the commit, so a client polling immediately after the commit finds them. |
| `MSG-443` | The incoming gateway takes a transaction-scoped advisory lock on a hash of the message identifier; the loser treats the message as a duplicate. |
| `MSG-444` | Sending is deferred to a post-commit hook by default and executed on a fresh connection, so a rollback cannot leave a sent electronic mail behind. |
| `MSG-445` | An alias marked invalid because a creation failed is written on an **independent** connection, so the mark survives the rollback of that creation. |
| `MSG-446` | A tracking snapshot taken twice in one transaction keeps the first value: the store refuses to overwrite an existing entry. |
| `MSG-447` | A record deleted while its Activity is being completed is re-checked; a missing record makes the Activity be deleted rather than archived and its attachments removed. |
| `MSG-448` | A record deleted while a Scheduled Message is due makes the posting fail; the creator is notified and the row is deleted. |
| `MSG-449` | The personal relay throttle counter lives on the relay row, which both jobs write, so the second waits for the first. |
| `MSG-450` | The postal queue commits after each letter, so a failure does not resend the letters already handled. |

---

## 31. Retention and collection rules

| Rule | Statement | Default |
|---|---|---|
| `MSG-451` | Notifications that are read, whose read moment is older than the retention window, whose recipient is not a customer, and whose status is delivered or cancelled are deleted in batches; the routine reports whether more remain. | 180 days |
| `MSG-452` | Event bus entries older than the retention window are deleted. | 1 day |
| `MSG-453` | Message Translations older than the retention window are deleted. | the platform default |
| `MSG-454` | Stale Presence rows are deleted. | the inactivity window |
| `MSG-455` | Overdue Activities older than the configured number of years are deleted; see `MSG-230`. | 3 years |
| `MSG-456` | Text Messages marked for deletion are physically removed; their Notifications are **never** deleted with them. | immediately |
| `MSG-457` | An Outgoing Mail kept after a real failure is never collected automatically; only an explicit cancellation or deletion removes it. | none |
| `MSG-458` | Attachments created in a composer that never became a message are deleted. | immediately |
| `MSG-459` | Link Previews no message refers to are deleted. | immediately |
| `MSG-460` | Call sessions whose heartbeat lapsed are deleted and the remaining participants are told. | the inactivity window |
| `MSG-461` | A member of a **sub-thread** is unpinned when the member's and the channel's last-interest moments are both older than two days and there is no non-notification message at or after the member's separator; the client is told to close the window. | 2 days |
| `MSG-462` | A live chat session handled only by a bot and inactive for more than a day is archived; an empty session is deleted. | 1 day |
| `MSG-463` | Expired channel mutes are cleared daily and the affected members are re-broadcast. | daily |
| `MSG-464` | A read live chat session with no activity for a day is unpinned from the operator's sidebar. | 1 day |
| `MSG-465` | Every collection routine deletes at most one batch per run and reports whether more remain, so the scheduler can continue on the next pass. | none |

---

## 32. Database constraints

| Rule | Record | Constraint | Message |
|---|---|---|---|
| `MSG-466` | Follower | unique triple (model, record, contact) | "Error, a partner cannot follow twice the same object." |
| `MSG-467` | Notification | an inbox row must name a recipient | "Customer is required for inbox notification" |
| `MSG-468` | Notification | an electronic-mail row must carry a failure type, a recipient or an address | "Customer or email is required for inbox / email notification" |
| `MSG-469` | Notification | at most one row per (message, recipient) when the recipient is set | database-level uniqueness |
| `MSG-470` | Message Reaction | exactly one of contact and guest | "A message reaction must be from a partner or from a guest." |
| `MSG-471` | Message Reaction | unique per (message, content, contact) and per (message, content, guest) | database-level uniqueness |
| `MSG-472` | Link Preview | unique web address | database-level uniqueness |
| `MSG-473` | Message Link Preview | unique pair (message, preview) | database-level uniqueness |
| `MSG-474` | Message Translation | unique pair (message, target language) | database-level uniqueness |
| `MSG-475` | Blacklist Entry | unique address | "Email address already exists!" |
| `MSG-476` | Role | unique name | "A role with the same name already exists." |
| `MSG-477` | Activity | a model implies a non-zero record, and no model implies no record | "Activities have to be linked to records with a not null res_id." |
| `MSG-478` | Activity | no model implies an assignee | "Activities must be assigned if not attached to a document." |
| `MSG-479` | Alias | unique pair (local part, domain), treating "no domain" as a value | see `MSG-129` |
| `MSG-480` | Alias Domain | unique pair (bounce local part, name) | "Bounce emails should be unique" |
| `MSG-481` | Alias Domain | unique pair (catch-all local part, name) | "Catchall emails should be unique" |
| `MSG-482` | Outgoing Mail Server | unique owner | "owner_user_id must be unique" |
| `MSG-483` | Presence | exactly one of user and guest | "A mail presence must have a user or a guest." |
| `MSG-484` | Presence | at most one row per user and per guest | database-level uniqueness |
| `MSG-485` | Push Device | unique endpoint | "The endpoint must be unique !" |
| `MSG-486` | User | a shared user may not use the in-application inbox | "Only internal user can receive notifications in Odoo" |
| `MSG-487` | Channel | unique initial message | "Messages can only be linked to one sub-channel" |
| `MSG-488` | Channel | unique token | "The channel UUID must be unique" |
| `MSG-489` | Channel | an authorization group only on a channel | "Group authorization and group auto-subscription are only supported on channels." |
| `MSG-490` | Channel Member | exactly one of contact and guest | "A channel member must be a partner or a guest." |
| `MSG-491` | Channel Member | at most one row per (channel, contact) and per (channel, guest) | database-level uniqueness |
| `MSG-492` | Call Session | at most one per member | "There can only be one rtc session per channel member" |
| `MSG-493` | Call History | must name a channel | "Call history must have a channel" |
| `MSG-494` | Call History | must have a start moment | "Call history must have a start date" |
| `MSG-495` | Call History | unique starting message | "Messages can only be linked to one call history" |
| `MSG-496` | User Settings Volume | exactly one of contact and guest | "A volume setting must have a partner or a guest." |
| `MSG-497` | User Settings Volume | at most one row per (settings, contact) and per (settings, guest) | database-level uniqueness |
| `MSG-498` | Favorite Animated Image | unique per (creator, image) | "User should not have duplicated favorite GIF" |
| `MSG-499` | Live Chat Channel | the maximum session count is strictly positive | "Concurrent session number should be greater than zero." |
| `MSG-500` | Live Chat Member History | one row per membership | "Members can only be linked to one history" |
| `MSG-501` | Live Chat Member History | one row per (session, contact) and per (session, guest) | "One partner can only be linked to one history on a channel" / "One guest can only be linked to one history on a channel" |
| `MSG-502` | Live Chat Member History | not both a contact and a guest | "History should either be linked to a partner or a guest but not both" |
| `MSG-503` | Live chat session | an operator is required | "Livechat Operator ID is required for a channel of type livechat." |
| `MSG-504` | Live chat session | a closed session carries no working status | "Closed Live Chat session should not have a status." |
| `MSG-505` | Expertise | unique name | database-level uniqueness |
| `MSG-506` | Conversation Tag | unique name | database-level uniqueness |
| `MSG-507` | Chatbot Message | one row per message | "A mail.message can only be linked to a single chatbot message" |
| `MSG-508` | Mailing Group Member | unique per (contact, list) | "This partner is already subscribed to the group" |
| `MSG-509` | Mailing Group Moderation Rule | unique per (list, address) | "You can create only one rule for a given email address in a group." |
| `MSG-510` | Text Message | unique correlation token | "UUID must be unique" |
| `MSG-511` | Text Message Tracker | unique token | "A record for this UUID already exists" |
| `MSG-512` | Blocked number | unique number | "Number already exists" |
| `MSG-513` | Rating | the value is between zero and five | "Rating should be between 0 and 5" |
| `MSG-514` | Contact enrichment | one row per contact | "Only one enrichment record is allowed per contact" |
| `MSG-515` | Guest | the access token is required and is readable only by the system group | the platform's not-null violation |

---

## 33. Invariants

An implementation must preserve all of the following at all times.

| Rule | Invariant | How it is enforced |
|---|---|---|
| `MSG-516` | A message is never notified twice to the same contact through the same channel. | the uniqueness of (message, recipient) on the Notification |
| `MSG-517` | A follower row never exists twice for the same (model, record, contact). | `MSG-466` |
| `MSG-518` | An internal message never reaches a customer. | three independent places: the follower query excludes a customer when the subtype is internal; the message access policy hides internal messages from non-internal callers; and the reference chain of a notification prefers public ancestors |
| `MSG-519` | A notification status never moves backwards on the text-message channel. | the monotonic rule `MSG-336` |
| `MSG-520` | A sent electronic mail is never un-sent by a rollback. | `MSG-437` and `MSG-444` |
| `MSG-521` | An incoming message is processed at most once. | the identifier lookup `MSG-146` plus the advisory lock `MSG-443` |
| `MSG-522` | A bounce never creates or updates a record. | `MSG-149` |
| `MSG-523` | A reply to the platform's own bounce is never answered. | the loop tag in the references, `MSG-153` |
| `MSG-524` | A tracked field change always produces a tracking entry in the same transaction as the change, or none at all when the caller disabled tracking; there is no window in which the change is stored and the entry is not. | the end-of-transaction comparison |
| `MSG-525` | An Activity's completion date is stamped once and never rewritten. | `MSG-212` |
| `MSG-526` | A Channel's type never changes. | `MSG-231` |
| `MSG-527` | A Channel Member's channel, contact and guest never change. | `MSG-240` |
| `MSG-528` | A live chat session always has an operator, and a closed session never has a working status. | `MSG-503` and `MSG-504` |
| `MSG-529` | A Message keeps the company and the alias domain in force at its creation, so the return path of an already-sent message is stable. | `MSG-418` and `MSG-419` |
| `MSG-530` | A Postal Letter keeps the address in force at its creation. | `MSG-352` |
| `MSG-531` | Every recipient of a Notification is either a contact or a bare address, never both, so a delivery report can always be attributed. | `MSG-467` and `MSG-468` |
| `MSG-532` | The separator of a Channel Member is the identifier of the first unread message, so the unread count and the visual separator can never disagree. | the marking-read arithmetic |
| `MSG-533` | An author never receives a notification for their own message, unless a caller asks for it or the author is a direct recipient and the mention switch is on. | `MSG-063` |
| `MSG-534` | An alias address of the installation is never subscribed to a record and never becomes a contact. | `MSG-168` |
| `MSG-535` | A guest never gains any access outside the channel its token belongs to. | `MSG-013` and `MSG-271` |

---

## 34. Rule identifier mapping

The working-branch version numbered its rules `MSG-RULE-nnn`; the version in the target folder carried no identifiers and organised the same material by section. Both mappings are given here so that a reader holding either version can find the current rule.

### 34.1 From the working-branch identifiers

| Former | Current | Former | Current | Former | Current |
|---|---|---|---|---|---|
| MSG-RULE-001 | MSG-040 | MSG-RULE-230 | MSG-457 | MSG-RULE-450 | MSG-321 |
| MSG-RULE-002 | MSG-041 | MSG-RULE-250 | MSG-097 | MSG-RULE-451 | MSG-321 |
| MSG-RULE-003 | MSG-042 | MSG-RULE-251 | MSG-102 | MSG-RULE-452 | MSG-324 |
| MSG-RULE-004 | MSG-042 | MSG-RULE-252 | MSG-108 | MSG-RULE-453 | MSG-324 |
| MSG-RULE-005 | MSG-044 | MSG-RULE-253 | MSG-109 | MSG-RULE-454 | MSG-326 |
| MSG-RULE-006 | MSG-045 | MSG-RULE-254 | MSG-104 | MSG-RULE-455 | MSG-325 |
| MSG-RULE-007 | MSG-047 | MSG-RULE-255 | MSG-105 | MSG-RULE-456 | MSG-322 |
| MSG-RULE-008 | MSG-046 | MSG-RULE-256 | MSG-115 | MSG-RULE-457 | MSG-328 |
| MSG-RULE-009 | MSG-047 | MSG-RULE-257 | MSG-106 | MSG-RULE-458 | MSG-327 |
| MSG-RULE-010 | MSG-048 | MSG-RULE-258 | MSG-107 | MSG-RULE-459 | MSG-327 |
| MSG-RULE-011 | MSG-051 | MSG-RULE-259 | MSG-110 | MSG-RULE-460 | MSG-344 |
| MSG-RULE-012 | MSG-049 | MSG-RULE-260 | MSG-113 | MSG-RULE-461 | MSG-344 |
| MSG-RULE-013 | MSG-039 | MSG-RULE-261 | MSG-111 | MSG-RULE-462 | MSG-330 |
| MSG-RULE-020 | MSG-026 | MSG-RULE-262 | MSG-112 | MSG-RULE-463 | MSG-331 |
| MSG-RULE-021 | MSG-027 | MSG-RULE-263 | MSG-115 | MSG-RULE-464 | MSG-329 |
| MSG-RULE-022 | MSG-028 | MSG-RULE-264 | MSG-114 | MSG-RULE-465 | MSG-333 |
| MSG-RULE-023 | MSG-033 | MSG-RULE-265 | MSG-097 | MSG-RULE-466 | MSG-338 |
| MSG-RULE-024 | MSG-034 | MSG-RULE-280 | MSG-116 | MSG-RULE-467 | MSG-339 |
| MSG-RULE-025 | MSG-025 | MSG-RULE-281 | MSG-117 | MSG-RULE-468 | MSG-340 |
| MSG-RULE-026 | MSG-029 | MSG-RULE-282 | MSG-118 | MSG-RULE-469 | MSG-346 |
| MSG-RULE-027 | MSG-030 | MSG-RULE-283 | MSG-119 | MSG-RULE-470 | MSG-347 |
| MSG-RULE-028 | MSG-031 | MSG-RULE-284 | MSG-120 | MSG-RULE-471 | MSG-348 |
| MSG-RULE-029 | MSG-032 | MSG-RULE-285 | MSG-122 | MSG-RULE-472 | MSG-349 |
| MSG-RULE-030 | MSG-038 | MSG-RULE-286 | MSG-121 | MSG-RULE-473 | MSG-350 |
| MSG-RULE-040 | MSG-002 | MSG-RULE-287 | MSG-458 | MSG-RULE-474 | MSG-351 |
| MSG-RULE-041 | MSG-003 | MSG-RULE-288 | MSG-123 | MSG-RULE-475 | MSG-352 |
| MSG-RULE-042 | MSG-004 | MSG-RULE-289 | MSG-124 | MSG-RULE-490 | MSG-372 |
| MSG-RULE-043 | MSG-005 | MSG-RULE-290 | MSG-125 | MSG-RULE-491 | MSG-373 |
| MSG-RULE-044 | MSG-001 | MSG-RULE-300 | MSG-093 | MSG-RULE-492 | MSG-374 |
| MSG-RULE-045 | MSG-015 | MSG-RULE-301 | MSG-094 | MSG-RULE-493 | MSG-375 |
| MSG-RULE-046 | MSG-011 | MSG-RULE-302 | MSG-023 | MSG-RULE-494 | MSG-376 |
| MSG-RULE-060 | MSG-058 | MSG-RULE-303 | MSG-024 | MSG-RULE-495 | MSG-378 |
| MSG-RULE-061 | MSG-054 | MSG-RULE-304 | MSG-022 | MSG-RULE-496 | MSG-379 |
| MSG-RULE-062 | MSG-055 | MSG-RULE-305 | MSG-095 | MSG-RULE-497 | MSG-380 |
| MSG-RULE-063 | MSG-057 | MSG-RULE-320 | MSG-206 | MSG-RULE-510 | MSG-296 |
| MSG-RULE-064 | MSG-061 | MSG-RULE-321 | MSG-207 | MSG-RULE-511 | MSG-297 |
| MSG-RULE-065 | MSG-060 | MSG-RULE-322 | MSG-016 | MSG-RULE-512 | MSG-299 |
| MSG-RULE-066 | MSG-062 | MSG-RULE-323 | MSG-017 | MSG-RULE-513 | MSG-300 |
| MSG-RULE-067 | MSG-062 | MSG-RULE-324 | MSG-018 | MSG-RULE-514 | MSG-301 |
| MSG-RULE-080 | MSG-063 | MSG-RULE-325 | MSG-020 | MSG-RULE-515 | MSG-298 |
| MSG-RULE-081 | MSG-064 | MSG-RULE-326 | MSG-210 | MSG-RULE-516 | MSG-302 |
| MSG-RULE-082 | MSG-065 | MSG-RULE-327 | MSG-211 | MSG-RULE-517 | MSG-302 |
| MSG-RULE-083 | MSG-066 | MSG-RULE-328 | MSG-212 | MSG-RULE-518 | MSG-302 |
| MSG-RULE-084 | MSG-067 | MSG-RULE-329 | MSG-213 | MSG-RULE-519 | MSG-307 |
| MSG-RULE-085 | MSG-068 | MSG-RULE-330 | MSG-209 | MSG-RULE-520 | MSG-303 |
| MSG-RULE-086 | MSG-070 | MSG-RULE-331 | MSG-215 | MSG-RULE-521 | MSG-317 |
| MSG-RULE-087 | MSG-071 | MSG-RULE-332 | MSG-218 | MSG-RULE-522 | MSG-304 |
| MSG-RULE-088 | MSG-072 | MSG-RULE-333 | MSG-217 | MSG-RULE-523 | MSG-305 |
| MSG-RULE-089 | MSG-073 | MSG-RULE-334 | MSG-220 | MSG-RULE-524 | MSG-306 |
| MSG-RULE-090 | MSG-074 | MSG-RULE-335 | MSG-221 | MSG-RULE-525 | MSG-319 |
| MSG-RULE-091 | MSG-075 | MSG-RULE-336 | MSG-222 | MSG-RULE-526 | MSG-308 |
| MSG-RULE-092 | MSG-069 | MSG-RULE-337 | MSG-223 | MSG-RULE-527 | MSG-309 |
| MSG-RULE-093 | MSG-077 | MSG-RULE-338 | MSG-224 | MSG-RULE-528 | MSG-310 |
| MSG-RULE-094 | MSG-079 | MSG-RULE-339 | MSG-225 | MSG-RULE-540 | MSG-418 |
| MSG-RULE-095 | MSG-078 | MSG-RULE-340 | MSG-226 | MSG-RULE-541 | MSG-419 |
| MSG-RULE-096 | MSG-080 | MSG-RULE-341 | MSG-227 | MSG-RULE-542 | MSG-420 |
| MSG-RULE-110 | MSG-081 | MSG-RULE-342 | MSG-208 | MSG-RULE-543 | MSG-421 |
| MSG-RULE-111 | MSG-082 | MSG-RULE-343 | MSG-229 | MSG-RULE-544 | MSG-422 |
| MSG-RULE-112 | MSG-083 | MSG-RULE-344 | MSG-230 | MSG-RULE-545 | MSG-423 |
| MSG-RULE-113 | MSG-084 | MSG-RULE-360 | MSG-231 | MSG-RULE-546 | MSG-424 |
| MSG-RULE-114 | MSG-092 | MSG-RULE-361 | MSG-237 | MSG-RULE-547 | MSG-425 |
| MSG-RULE-115 | MSG-086 | MSG-RULE-362 | MSG-237 | MSG-RULE-560 | MSG-427 |
| MSG-RULE-116 | MSG-087 | MSG-RULE-363 | MSG-233 | MSG-RULE-561 | MSG-428 |
| MSG-RULE-117 | MSG-088 | MSG-RULE-364 | MSG-234 | MSG-RULE-562 | MSG-429 |
| MSG-RULE-118 | MSG-089 | MSG-RULE-365 | MSG-235 | MSG-RULE-563 | MSG-430 |
| MSG-RULE-119 | MSG-090 | MSG-RULE-366 | MSG-237 | MSG-RULE-564 | MSG-431 |
| MSG-RULE-120 | MSG-090 | MSG-RULE-367 | MSG-238 | MSG-RULE-565 | MSG-432 |
| MSG-RULE-121 | MSG-091 | MSG-RULE-368 | MSG-490 | MSG-RULE-566 | MSG-433 |
| MSG-RULE-140 | MSG-129 | MSG-RULE-369 | MSG-239 | MSG-RULE-580 | MSG-435 |
| MSG-RULE-141 | MSG-126 | MSG-RULE-370 | MSG-260 | MSG-RULE-581 | MSG-439 |
| MSG-RULE-142 | MSG-127 | MSG-RULE-371 | MSG-259 | MSG-RULE-582 | MSG-436 |
| MSG-RULE-143 | MSG-129 | MSG-RULE-372 | MSG-257 | MSG-RULE-583 | MSG-450 |
| MSG-RULE-144 | MSG-130 | MSG-RULE-373 | MSG-258 | MSG-RULE-584 | MSG-443 |
| MSG-RULE-145 | MSG-128 | MSG-RULE-374 | MSG-240 | MSG-RULE-585 | MSG-437 |
| MSG-RULE-146 | MSG-131 | MSG-RULE-375 | MSG-232 | MSG-RULE-586 | MSG-438 |
| MSG-RULE-147 | MSG-132 | MSG-RULE-376 | MSG-248 | MSG-RULE-587 | MSG-441 |
| MSG-RULE-148 | MSG-133 | MSG-RULE-377 | MSG-243 | MSG-RULE-588 | MSG-442 |
| MSG-RULE-149 | MSG-134 | MSG-RULE-378 | MSG-056 | MSG-RULE-600 | MSG-451 |
| MSG-RULE-150 | MSG-135 | MSG-RULE-379 | MSG-241 | MSG-RULE-601 | MSG-452 |
| MSG-RULE-151 | MSG-136 | MSG-RULE-380 | MSG-242 | MSG-RULE-602 | MSG-453 |
| MSG-RULE-152 | MSG-137 | MSG-RULE-381 | MSG-244 | MSG-RULE-603 | MSG-455 |
| MSG-RULE-153 | MSG-138 | MSG-RULE-382 | MSG-247 | MSG-RULE-604 | MSG-460 |
| MSG-RULE-154 | MSG-141 | MSG-RULE-383 | MSG-245 | MSG-RULE-605 | MSG-454 |
| MSG-RULE-155 | MSG-139 | MSG-RULE-384 | MSG-246 | MSG-RULE-606 | MSG-456 |
| MSG-RULE-156 | MSG-140 | MSG-RULE-385 | MSG-249 | MSG-RULE-607 | MSG-462 |
| MSG-RULE-170 | MSG-146 | MSG-RULE-386 | MSG-250 | MSG-RULE-608 | MSG-458 |
| MSG-RULE-171 | MSG-147 | MSG-RULE-387 | MSG-251 | MSG-RULE-609 | MSG-459 |
| MSG-RULE-172 | MSG-153 | MSG-RULE-388 | MSG-252 | MSG-RULE-610 | MSG-463 |
| MSG-RULE-173 | MSG-149 | MSG-RULE-389 | MSG-253 | MSG-RULE-620 | MSG-381 |
| MSG-RULE-174 | MSG-154 | MSG-RULE-390 | MSG-264 | MSG-RULE-621 | MSG-382 |
| MSG-RULE-175 | MSG-148 | MSG-RULE-391 | MSG-254 | MSG-RULE-622 | MSG-383 |
| MSG-RULE-176 | MSG-151 | MSG-RULE-392 | MSG-255 | MSG-RULE-623 | MSG-384 |
| MSG-RULE-177 | MSG-151 | MSG-RULE-410 | MSG-273 | MSG-RULE-624 | MSG-385 |
| MSG-RULE-178 | MSG-160 | MSG-RULE-411 | MSG-278 | MSG-RULE-625 | MSG-386 |
| MSG-RULE-179 | MSG-161 | MSG-RULE-412 | MSG-281 | MSG-RULE-626 | MSG-387 |
| MSG-RULE-180 | MSG-161 | MSG-RULE-413 | MSG-279 | MSG-RULE-627 | MSG-388 |
| MSG-RULE-181 | MSG-161 | MSG-RULE-414 | MSG-280 | MSG-RULE-628 | MSG-389 |
| MSG-RULE-182 | MSG-161 | MSG-RULE-415 | MSG-282 | MSG-RULE-629 | MSG-390 |
| MSG-RULE-183 | MSG-165 | MSG-RULE-416 | MSG-500 | MSG-RULE-630 | MSG-391 |
| MSG-RULE-184 | MSG-155 | MSG-RULE-417 | MSG-274 | MSG-RULE-640 | MSG-393 |
| MSG-RULE-185 | MSG-156 | MSG-RULE-418 | MSG-275 | MSG-RULE-641 | MSG-394 |
| MSG-RULE-186 | MSG-157 | MSG-RULE-419 | MSG-276 | MSG-RULE-642 | MSG-395 |
| MSG-RULE-187 | MSG-144 | MSG-RULE-420 | MSG-277 | MSG-RULE-643 | MSG-396 |
| MSG-RULE-188 | MSG-158 | MSG-RULE-421 | MSG-284 | MSG-RULE-644 | MSG-397 |
| MSG-RULE-189 | MSG-159 | MSG-RULE-422 | MSG-285 | MSG-RULE-645 | MSG-398 |
| MSG-RULE-190 | MSG-161 | MSG-RULE-423 | MSG-507 | MSG-RULE-646 | MSG-400 |
| MSG-RULE-191 | MSG-162 | MSG-RULE-424 | MSG-505 | MSG-RULE-647 | MSG-399 |
| MSG-RULE-192 | MSG-162 | MSG-RULE-425 | MSG-294 | MSG-RULE-648 | MSG-401 |
| MSG-RULE-193 | MSG-163 | MSG-RULE-426 | MSG-291 | MSG-RULE-649 | MSG-412 |
| MSG-RULE-194 | MSG-164 | MSG-RULE-427 | MSG-292 | MSG-RULE-650 | MSG-402 |
| MSG-RULE-210 | MSG-193 | MSG-RULE-428 | MSG-293 | MSG-RULE-651 | MSG-403 |
| MSG-RULE-211 | MSG-182 | MSG-RULE-429 | MSG-295 | MSG-RULE-652 | MSG-406 |
| MSG-RULE-212 | MSG-181 | MSG-RULE-430 | MSG-295 | MSG-RULE-653 | MSG-407 |
| MSG-RULE-213 | MSG-183 | MSG-RULE-670 | MSG-413 | MSG-RULE-654 | MSG-405 |
| MSG-RULE-214 | MSG-184 | MSG-RULE-671 | MSG-414 | MSG-RULE-655 | MSG-405 |
| MSG-RULE-215 | MSG-201 | MSG-RULE-672 | MSG-415 | MSG-RULE-656 | MSG-408 |
| MSG-RULE-216 | MSG-186 | MSG-RULE-673 | MSG-416 | MSG-RULE-657 | MSG-409 |
| MSG-RULE-217 | MSG-187 | MSG-RULE-680 | MSG-168 | MSG-RULE-658 | MSG-410 |
| MSG-RULE-218 | MSG-188 | MSG-RULE-681 | MSG-169 | MSG-RULE-659 | MSG-411 |
| MSG-RULE-219 | MSG-189 | MSG-RULE-682 | MSG-170 | MSG-RULE-683 | MSG-171 |
| MSG-RULE-220 | MSG-190 | MSG-RULE-684 | MSG-166 | MSG-RULE-686 | MSG-173 |
| MSG-RULE-221 | MSG-191 | MSG-RULE-685 | MSG-167 | MSG-RULE-687 | MSG-174 |
| MSG-RULE-222 | MSG-192 | MSG-RULE-688 | MSG-175 | MSG-RULE-689 | MSG-176 |
| MSG-RULE-223 | MSG-194 | MSG-RULE-690 | MSG-177 | — | — |
| MSG-RULE-224 | MSG-195 | — | — | — | — |
| MSG-RULE-225 | MSG-196 | — | — | — | — |
| MSG-RULE-226 | MSG-197 | — | — | — | — |
| MSG-RULE-227 | MSG-198 | — | — | — | — |
| MSG-RULE-228 | MSG-199 | — | — | — | — |
| MSG-RULE-229 | MSG-200 | — | — | — | — |

### 34.2 From the unnumbered sections of the other version

| Former section | Current rules |
|---|---|
| 1. The message access policy | MSG-001 to MSG-014 |
| 2. The activity access policy | MSG-016 to MSG-022 |
| 3. The scheduled-message access policy | MSG-023, MSG-024, MSG-093 to MSG-096 |
| 4. Posting and notifying: parameter validation | MSG-040 to MSG-053 |
| 5. Editing and deleting a message | MSG-025 to MSG-037 |
| 6. Follower rules | MSG-054 to MSG-062 |
| 7. Database constraints | MSG-466 to MSG-515 |
| 8. Alias and alias-domain validations | MSG-126 to MSG-145 |
| 9. Incoming gateway rejections | MSG-146 to MSG-165 |
| 10. Outgoing mail rules | MSG-181 to MSG-205 |
| 11. Template rules | MSG-097 to MSG-115, MSG-166, MSG-167 |
| 12. Activity rules | MSG-206 to MSG-230 |
| 13. Channel rules | MSG-231 to MSG-265 |
| 14. Guest and public-user rules | MSG-266 to MSG-272 |
| 15. Live chat rules | MSG-273 to MSG-295 |
| 16. Mailing group rules | MSG-296 to MSG-320 |
| 17. Text message rules | MSG-321 to MSG-345 |
| 18. Postal mail rules | MSG-346 to MSG-360 |
| 19. Digest rules | MSG-361 to MSG-370 |
| 20. Locking and concurrency rules | MSG-435 to MSG-450 |
| 21. Invariants | MSG-516 to MSG-535 |

---

## Reconciliation notes

1. **The message access policy on create.** One version listed the create branches as free-standing, follower, parent recipient, post-access permission; the other wrote "may read the parent message". The check is on the recipient list of the **parent**, not a full read evaluation, so `MSG-003` keeps the precise wording.
2. **Message deletion.** One version wrote that deletion is granted "when the user has the posting access right on the record". The observable rule requires **write** access on the record whatever the model's post-access attribute says, so `MSG-005` states write.
3. **The follower deletion order.** One version placed the follower deletion after the record deletion and explained it as "so access checks resolve". The observed order is: discard tracking, delete messages, delete records, delete followers, delete scheduled messages, and the reason is that neither messages nor followers can cascade at database level. `MSG-062` states both.
4. **Personal relay throttling.** One version counted electronic mails per minute, the other recipients per minute. The counter is raised by the number of contact recipients of each mail, or by one when a mail has none, so the limit counts **recipients**. `MSG-184` and [calculations.md](calculations.md), section 20, state that.
5. **The composer recipient failure.** One version attributed "No recipient found." only to the notify path. It is raised by the notify path and by the template path, so `MSG-117` states both, and adds the separate malformed-address message.
6. **Postal retry codes.** One version listed the retryable codes; the other did not enumerate them. `MSG-351` keeps the enumeration and adds which codes are never retried.
7. **Rating grades.** One version gave a four-value textual grade set, the other a three-value set used for the satisfaction percentage. `MSG-377` keeps both, because the four-value set is what a record stores and the three-value set is what the percentage counts.
8. **The channel bounce limit.** Both versions agree on ten; the rule is stated once, at `MSG-247`, and the constant is listed in [configuration.md](configuration.md), section 4.
