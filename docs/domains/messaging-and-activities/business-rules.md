# Messaging and Activities — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behavior of the domain, with the exact user-facing message the system produces.

Where a message contains a value supplied at run time, the placeholder is written in words between angle brackets, for example `<alias name>`.

Contents:

1. [The message access policy](#1-the-message-access-policy)
2. [The activity access policy](#2-the-activity-access-policy)
3. [The scheduled-message access policy](#3-the-scheduled-message-access-policy)
4. [Posting and notifying: parameter validation](#4-posting-and-notifying-parameter-validation)
5. [Editing and deleting a message](#5-editing-and-deleting-a-message)
6. [Follower rules](#6-follower-rules)
7. [Database constraints](#7-database-constraints)
8. [Alias and alias-domain validations](#8-alias-and-alias-domain-validations)
9. [Incoming gateway rejections](#9-incoming-gateway-rejections)
10. [Outgoing mail rules](#10-outgoing-mail-rules)
11. [Template rules](#11-template-rules)
12. [Activity rules](#12-activity-rules)
13. [Channel rules](#13-channel-rules)
14. [Guest and public-user rules](#14-guest-and-public-user-rules)
15. [Live chat rules](#15-live-chat-rules)
16. [Mailing group rules](#16-mailing-group-rules)
17. [Text message rules](#17-text-message-rules)
18. [Postal mail rules](#18-postal-mail-rules)
19. [Digest rules](#19-digest-rules)
20. [Locking and concurrency rules](#20-locking-and-concurrency-rules)
21. [Invariants](#21-invariants)

---

## 1. The message access policy

The Message model does not rely on record rules alone. Every operation runs the ordinary rules first and then applies the five-branch policy below. A record forbidden by either mechanism is forbidden.

### The universal restriction for non-internal users

Before anything else, a caller who is not an internal user is refused **every** message that satisfies any of:

- the message is marked employee-only;
- the message has no subtype;
- the message's subtype is marked internal.

For the read and create operations an additional condition applies: the message type must be exactly "comment". A non-internal user therefore never sees a tracking entry, a system notification or an internal note, whatever the record-level permissions say.

When a search is performed by a non-internal user, the same three conditions are pushed into the query as a filter before it runs, so the rows never leave the database.

### Read

Granted when **any** holds:

1. the acting contact is the author;
2. the acting user created the row;
3. the acting contact is among the direct recipients;
4. the acting contact has a Notification on the message;
5. the message names a model and a record, is not a user-specific notification, and the acting user may **read** that record.

### Create

Granted when **any** holds:

1. the message is free-standing (no model, or no record, or it is a user-specific notification);
2. the acting contact is among the direct recipients of the **parent** message;
3. the acting user has, on the related record, the permission named by that model's post-access attribute — `write` by default, `read` for a model that opted into it;
4. the acting contact is a Follower of the related record.

### Write

Granted when **any** holds:

1. the acting contact is the author;
2. the acting contact is among the direct recipients;
3. the acting contact has a Notification on the message;
4. the acting user may **write** the related record.

### Delete

Granted when the acting user may **write** the related record. There is no author exception: a person may not delete their own message on a record they cannot write.

### The refusal

```text
The requested operation cannot be completed due to security restrictions. Please contact your system administrator.

(Document type: Message, Operation: <operation>)

Records: <the first six identifiers>, User: <acting user identifier>
```

### Two further restrictions

| Rule | Message |
|---|---|
| Only a system administrator may change the model or the record of an existing message. | "Only administrators can modify 'model' and 'res_id' fields." |
| Only an administrator may export messages. | "Only administrators are allowed to export mail message" |

A caller who is not an internal user silently loses the author and sender fields from any create or write payload: the values are removed rather than refused, and the corresponding context defaults are stripped as well.

### Attachment checks

- Creating a message whose attachments include one that does **not** already belong to the same record checks read access on those attachments.
- Writing new attachments onto a message checks read access on them.
- Deleting a message deletes the attachments that belong to the message itself (their model is the message model and their record is one of the deleted messages, or zero).

### The single-record read path

Fetching one message by identifier for a controller uses a wider rule:

1. If the caller is a public user carrying a guest token, the ordinary access-right layer is skipped and only the five-branch policy is evaluated. This exists because the access-right layer predates guests and would wrongly refuse them.
2. Otherwise the full check runs.
3. If either fails, and the message names a model and a record, the message is granted anyway when the acting user has, on that record, the permission that the record's model maps to the requested message operation.

Unknown extra parameters passed to that path are logged as a warning, not refused, so a stale client cannot break.

---

## 2. The activity access policy

Like messages, activities layer a policy on top of the record rules.

The shipped record rule for internal users grants **write and delete only**, and only on activities the user is assigned to or created. Read and create are governed entirely by the policy below.

| Operation | Rule |
|---|---|
| Read | the record rules allow it **and** ( the activity is assigned to the acting user **or** the acting user may read the related record ) |
| Create | the record rules allow it **and** the acting user has, on the related record, the permission named by the model's post-access attribute |
| Write | the record rules allow it **or** the acting user has that same permission on the related record |
| Delete | the record rules allow it **or** the acting user has that same permission on the related record |

An activity with **no** related record is accessible only to its assignee; anybody else is refused.

For write and delete, the check short-circuits: if the caller has the permission on the model as a whole, the per-record check is skipped entirely.

Searching applies the same filter: a row assigned to the acting user always passes; any other row passes only if its related record is readable.

The refusal text is the same five-line message as for messages, with "Activity" as the document type.

---

## 3. The scheduled-message access policy

A Scheduled Message is governed by the record it will be posted on.

| Operation | Rule | Message on failure |
|---|---|---|
| Create | the acting user must have, on the target record, the permission named by the model's post-access attribute | the standard access refusal for that record |
| Read | the shipped record rule grants read to everyone who passes the access-right layer; the search override then keeps only the rows whose target record the acting user may act on | rows are silently dropped |
| Write | the same permission on the target record; additionally the record rule restricts write to rows the acting user created | the standard access refusal |
| Delete | the same permission on the target record | the standard access refusal |

Two specific refusals:

| Rule | Message |
|---|---|
| The target model and record may never be changed. | "You are not allowed to change the target record of a scheduled message." |
| Sending a scheduled message immediately is allowed only to an administrator or to the person who created it. | "You are not allowed to send this scheduled message" |

Validations:

| Rule | Message |
|---|---|
| The target model must be thread-enabled. | "A message cannot be scheduled on a model that does not have a mail thread." |
| The scheduled moment must not be in the past. | "A Scheduled Message cannot be scheduled in the past" |
| Scheduling is only offered in single-record posting mode. | "A message can only be scheduled in monocomment mode" |
| A scheduled moment must be supplied. | "A scheduled date is needed to schedule a message" |

When the scheduled job posts a message and the posting fails, the failure is swallowed: the author is notified with the subject "A scheduled message could not be sent" and the body "The message scheduled on <model>(<record identifier>) with the following content could not be sent:" followed by the original body between separator lines, and the scheduled message is deleted anyway. The notification is sent because the author may have lost access to the record in the meantime and would otherwise never learn of the loss.

Only a fixed list of notification parameters survives from the captured payload into the actual posting: whether to add a signature, the sender, the layout, the forced language, the activity type, whether to delete the mail, the relay, the message type, the model description, the reply address, whether answers start a new thread, and the subtype. Anything else stored is ignored.

---

## 4. Posting and notifying: parameter validation

### Forbidden parameters

| Operation | Forbidden | Message |
|---|---|---|
| post a message | `model`, `res_id`, `subtype` | "Those values are not supported when posting or notifying: <names>" |
| notify | `incoming_email_cc`, `incoming_email_to`, `message_id`, `message_type`, `outgoing_email_to`, `parent_id` | same |
| post or mail with a source | `body`, `composition_mode`, `incoming_email_cc`, `incoming_email_to`, `model`, `res_id`, `outgoing_email_to`, `values` | same |
| log with a view | `body`, `bodies`, `incoming_email_cc`, `incoming_email_to`, `outgoing_email_to` | same |

### The allowed message fields

Creating a message through the thread operations accepts only this set of fields; anything else raises the same message: attachments, guest author, author, body, creation date, date, add-signature flag, sender, layout, incoming carbon copy, incoming to, employee-only flag, activity type, relay, message identifier, message type, model, outgoing to, parent, direct recipients, alias domain, company, reply address, force-new-thread flag, record, subject, subtype, tracking values.

### The allowed notification parameters

A notification call accepts only: forced company, forced language, forced record name, force-send flag, automatic-deletion flag, model description, notify-author flag, notify-author-when-mentioned flag, skip-followers flag, scheduled moment, send-after-commit flag, skip-existing flag, subtitles. An internal or elevated caller may additionally pass extra headers; a portal caller may not, so a portal user cannot inject headers.

### Shape validations

| Rule | Message |
|---|---|
| Posting must target a real record. | "Posting a message should be done on a business document. Use message_notify to send a notification to an user." |
| Posting may not use the user-specific type. | "Use message_notify to send a notification to an user." |
| Inline attachments must be a list of two- or three-element groups. | "Posting a message should receive attachments as a list of list or tuples (received <value>)" / "Notification should receive attachments as a list of list or tuples (received <value>)" |
| Existing attachments must be a list of identifiers, not commands. | "Posting a message should receive attachments records as a list of IDs (received <value>)" / "Notification should receive attachments records as a list of IDs (received <value>)" |
| Recipients must be a list of identifiers. | "Posting a message should receive partners as a list of IDs (received <value>)" / "Notification should receive partners given as a list of IDs (received <value>)" |
| Notifying without recipients does nothing and logs a warning. | no user-facing message |
| A batch log may not carry attachments or tracking values for more than one record. | "Batch log cannot support attachments or tracking values on more than 1 document" |
| The source of a post-with-source must be a template or a view. | "Invalid template or view source record <value>, is <model> instead" |
| An external identifier source must resolve. | "Invalid template or view source Xml ID <reference> does not exist anymore" |
| An external identifier must resolve to a template or a view. | "Invalid template or view source reference <value>, is <model> instead" |
| A source must be a record or an external identifier. | "Invalid template or view source <value> (type <type>), should be a record or an XMLID" |
| The source must not be empty. | "Mailing or posting with a source should not be called with an empty <template or view>" |
| Rendering requires a language in context. | "At this point lang should be correctly set" |
| The composer in single-record mode needs at least one record. | "Mail composer in comment mode should run on at least one record. No records found (model <model>)." |

### Attachment handling for non-internal callers

When the caller is not an internal user, the list of existing attachments to link is **replaced** by the subset that is both attached to a composer or a scheduled message and created by that very user. This prevents a portal user from attaching somebody else's file to a message.

---

## 5. Editing and deleting a message

### The default rule

| Rule | Message |
|---|---|
| A message carrying tracking values may never be edited. | "Messages with tracking values cannot be modified" |
| Only a message of type comment may be edited. | "Only messages type comment can have their content updated" |

### The channel rule

A channel deliberately replaces the default rule rather than extending it: the tracking check does not apply, and only the type check remains.

| Rule | Message |
|---|---|
| Only a message of type comment may be edited in a channel. | "Only messages type comment can have their content updated on model 'discuss.channel'" |

### Side effects of an edit

1. When the new body is non-empty, or the message is not becoming void, an "edited" marker is appended inside the last block-level element of the body, or at the end when the body is plain text.
2. When the new body is empty and the message becomes void, the body is stored as empty.
3. Replacing the attachment list with an empty list deletes the previous attachments and notifies their removal. Passing no attachment list at all leaves them alone.
4. Every cached translation of the message is deleted.
5. When the message becomes void, its link previews are removed. In a channel its parent link is cleared as well.
6. The updated fields are broadcast: attachments sorted by identifier, body, direct recipients with their avatar and name, pinning moment, modification moment, the linked-message summaries, and — when the subject changed — the subject, plus a marker clearing the translation.

---

## 6. Follower rules

| Rule | Detail |
|---|---|
| Subscribing somebody **else** requires write access on the record. | The standard access refusal. |
| Subscribing **yourself** requires only read access, and silently does nothing when even that is missing. | No message. |
| Subscribing somebody else silently drops archived contacts. | No message. |
| Unsubscribing somebody else requires write access. | The standard access refusal. |
| Unsubscribing **yourself** requires only read access when you are not an internal user; an internal user may always unsubscribe themselves, whatever the company context. | No message. |
| A contact may follow a record at most once. | "Error, a partner cannot follow twice the same object." |
| A channel may not have followers at all. | "Adding followers on channels is not possible. Consider adding members instead." |
| A non-internal user may filter records by follower only on themselves or their commercial entity. | "Portal users can only filter threads by themselves as followers." |

Default subtypes at subscription: a contact that is a customer receives the **external** default set; anyone else receives the **full** default set.

---

## 7. Database constraints

| Record | Rule | Message |
|---|---|---|
| Follower | unique triple (model, record, contact) | "Error, a partner cannot follow twice the same object." |
| Notification | an inbox notification must name a recipient | "Customer is required for inbox notification" |
| Notification | an electronic-mail notification must have a failure type, a recipient or an address | "Customer or email is required for inbox / email notification" |
| Notification | at most one notification per (message, recipient) | database-level uniqueness |
| Message Reaction | exactly one of contact and guest | "A message reaction must be from a partner or from a guest." |
| Message Reaction | unique per (message, content, contact) and per (message, content, guest) | database-level uniqueness |
| Link Preview | unique web address | database-level uniqueness |
| Message Link Preview | unique pair (message, preview) | database-level uniqueness |
| Message Translation | unique pair (message, target language) | database-level uniqueness |
| Blacklist Entry | unique address | "Email address already exists!" |
| Role | unique name | "A role with the same name already exists." |
| Activity | a model implies a non-zero record, and no model implies no record | "Activities have to be linked to records with a not null res_id." |
| Activity | no model implies an assignee | "Activities must be assigned if not attached to a document." |
| Alias | unique pair (local part, domain), treating "no domain" as a value | see section 8 |
| Alias Domain | unique pair (bounce local part, name) | "Bounce emails should be unique" |
| Alias Domain | unique pair (catch-all local part, name) | "Catchall emails should be unique" |
| Outgoing Mail Server | unique owner | "owner_user_id must be unique" |
| Presence | exactly one of user and guest | "A mail presence must have a user or a guest." |
| Presence | at most one row per user and per guest | database-level uniqueness |
| Push Device | unique endpoint | "The endpoint must be unique !" |
| User | a shared user may not use the in-application inbox | "Only internal user can receive notifications in Odoo" |
| Channel | unique initial message | "Messages can only be linked to one sub-channel" |
| Channel | unique token | "The channel UUID must be unique" |
| Channel | an authorization group only on a channel | "Group authorization and group auto-subscription are only supported on channels." |
| Channel Member | exactly one of contact and guest | "A channel member must be a partner or a guest." |
| Channel Member | at most one row per (channel, contact) and per (channel, guest) | database-level uniqueness |
| Call Session | at most one per member | "There can only be one rtc session per channel member" |
| Call History | must name a channel | "Call history must have a channel" |
| Call History | must have a start moment | "Call history must have a start date" |
| Call History | unique starting message | "Messages can only be linked to one call history" |
| User Settings Volume | exactly one of contact and guest | "A volume setting must have a partner or a guest." |
| User Settings Volume | at most one row per (settings, contact) and per (settings, guest) | database-level uniqueness |
| Favorite Animated Image | unique per (creator, image) | "User should not have duplicated favorite GIF" |
| Live Chat Channel | the maximum session count must be strictly positive | "Concurrent session number should be greater than zero." |
| Live Chat Member History | one history per membership | "Members can only be linked to one history" |
| Live Chat Member History | one per (session, contact) and per (session, guest) | "One partner can only be linked to one history on a channel" / "One guest can only be linked to one history on a channel" |
| Live Chat Member History | not both a contact and a guest | "History should either be linked to a partner or a guest but not both" |
| Live chat session | an operator is required | "Livechat Operator ID is required for a channel of type livechat." |
| Live chat session | a closed session has no working status | "Closed Live Chat session should not have a status." |
| Expertise | unique name | database-level uniqueness |
| Conversation Tag | unique name | database-level uniqueness |
| Chatbot Message | one per message | "A mail.message can only be linked to a single chatbot message" |
| Mailing Group Member | unique per (contact, list) | "This partner is already subscribed to the group" |
| Mailing Group Moderation Rule | unique per (list, address) | "You can create only one rule for a given email address in a group." |
| Text Message | unique identifier | "UUID must be unique" |
| Text Message Tracker | unique identifier | "A record for this UUID already exists" |

---

## 8. Alias and alias-domain validations

### Alias

| Rule | Message |
|---|---|
| The local part must be a plain ASCII dot-atom. | "You cannot use anything else than unaccented latin characters in the alias address <local part>." |
| The default values must parse as a literal mapping. | "Invalid expression, it must be a literal python dictionary definition e.g. "{'field': 'value'}"" |
| The local part must not equal the bounce or the catch-all local part of its own domain. | "Aliases <names> is already used as bounce or catchall address. Please choose another alias." |
| The pair (local part, domain) must be free, and the alias in conflict names its owner when it has one. | "Alias <conflicting display name> (<current identifiers or "your alias" or "new">) is already linked with <model label> (<conflicting identifier>) and used by the <owner display name> <owner model label>. Choose another value or change it on the other document." — or, without an owner, "Alias <conflicting display name> (<current identifiers>) is already linked with <model label> (<conflicting identifier>). Choose another value or change it on the other document." |
| The same local part may not be assigned to several records in one operation. | "Email aliases <name> cannot be used on several records at the same time. Please update records one by one." |
| When the owner record has a company and the alias domain is bound to companies, they must agree. | "We could not create alias <alias display name> because domain <domain name> belongs to company <company names> while the owner document belongs to company <company name>." |
| The same rule for the forced target record. | "We could not create alias <alias display name> because domain <domain name> belongs to company <company names> while the target document belongs to company <company name>." |

The uniqueness check is skipped for aliases whose target model or owner model is not currently present in the registry, which keeps installation and test setups working.

### Alias domain

| Rule | Message |
|---|---|
| The name must not be empty. | "You cannot assign an empty domain name." |
| The name must be a plain ASCII dot-atom. | "You cannot use anything else than unaccented latin characters in the domain name <name>." |
| Two domains with the same name must not share a bounce local part. | "Bounce alias <address> is already used for another domain with same name. Use another bounce or simply use the other alias domain." |
| Two domains with the same name must not share a catch-all local part. | "Catchall alias <address> is already used for another domain with same name. Use another catchall or simply use the other alias domain." |
| No Alias may already occupy the bounce or catch-all address of the domain. | "Bounce/Catchall '<address>' is already used by <document display name>. Choose another alias or change it on the other document." — or, when no document can be named, "Bounce/Catchall '<address>' is already used. Choose another alias or change it on the linked model." |
| The default sender address must not fall inside the filter of any personal relay. | "A personal mail server is using that address, you can not use it." |

### The allowed-domain parameter

The value must be a comma-separated list of domains; it is trimmed and lower-cased before storage.

| Rule | Message |
|---|---|
| The list must not be empty after cleaning. | "Value <value> for `mail.catchall.domain.allowed` cannot be validated.\nIt should be a comma separated list of domains e.g. example.com,example.org." |

---

## 9. Incoming gateway rejections

Every rejection produces either a bounce message to the sender or a silent drop. The table gives both the internal warning text (which appears in the operator's log) and the consequence.

| Situation | Internal text | Consequence |
|---|---|---|
| The route names no model | "target model unspecified" | the route is dropped, or the call aborts when raising was requested |
| The route names an unknown model | "unknown target model <model>" | same |
| The named record no longer exists | "reply to missing document (<model>,<record>), fall back on document creation" | the record is cleared and creation is attempted |
| The model does not accept updates | "reply to model <model> that does not accept document update, fall back on document creation" | same |
| The model does not accept creation | "model <model> does not accept document creation" | the route is dropped, or the call aborts |
| The alias refuses the sender | "alias <local part>: <error>" | a bounce with the **security** body; the alias status is unchanged |
| The alias is misconfigured | "alias <local part>: incorrectly configured alias" or "… (unknown reference record)" | a bounce with the **invalid** body; the alias status becomes invalid |
| The message writes directly to the catch-all | "direct write to catchall, bounce" | a bounce with the catch-all body, referencing the loop tag, replying to the company address |
| The recipients contain the catch-all and nothing routable | "write to catchall + other unroutable emails, bounce" | same |
| No route at all and no bounce was sent | — | the call aborts with "No possible route found for incoming message from <sender> to <recipients> (Message-Id <identifier>:). Create an appropriate mail.alias or force the destination model." |
| The message identifier already exists | "found duplicated Message-Id during processing" | silently ignored |
| The advisory lock could not be taken | same | silently ignored |
| The message references one of the platform's own bounces | "reply to a bounce notification detected by headers" | silently ignored |
| The sender exceeded the loop quota | "created too many <model>" or "too much replies on same <model>" | a bounce with the "too many messages" body, referencing the loop tag; the message is ignored |
| The model does not accept incoming messages at processing time | — | the call aborts with "Undeliverable mail with Message-Id <identifier>, model <model> does not accept incoming emails" |

The generic wrapper for a routing warning that is raised is "Mailbox unavailable - <error>", deliberately short so that internal diagnostics are not exposed to the sender.

### The security bounce body

When the alias defines a custom body, that body is used. Otherwise:

```text
Dear Sender,

The message below could not be accepted by the address <alias display name>. Only <contact description> are allowed to contact it.

Please make sure you are using the correct address or contact us at <default address> instead.

Kind Regards
```

The contact description is "addresses linked to registered partners" when the policy is "Authenticated Partners", and "some specific addresses" otherwise. The default address is the company contact's formatted address, falling back to the company name. The whole body is rendered in the **author's** language when the author is known.

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
| sender | "MAILER-DAEMON" plus the company bounce address; failing that, the "to" header of the incoming message, but only when none of its addresses is a catch-all; failing that, "MAILER-DAEMON" plus the acting user's normalized address |

---

## 10. Outgoing mail rules

| Rule | Message |
|---|---|
| A mail may not name a relay its message's creator may not use. | "You may not create a message using another user's mail server." |
| A whole sending batch may not use a relay outside the allowed set. | "Unauthorized server for some of the sending mails." |
| A relay that cannot be reached, when the caller asked for an exception. | "Unable to connect to SMTP Server" |
| A delivery failure raised to the caller. | the exception text, or "Invalid text: <object>" for an encoding failure |
| Only internal users may configure a personal relay. | "Only internal users can configure a personal mail server." / "Only internal users can configure personal mail servers." |
| Configuring a personal relay requires the user to have an address. | "Please set your email before connecting your mail server." |
| A personal relay may not use an address that belongs to an alias domain. | "Your email address is used by an alias domain, and so you can not create a mail server for it." |
| Creating a personal relay for somebody else. | "You are not allowed to create a personal mail server." |
| Testing somebody else's personal relay. | "You are not allowed to test personal mail servers." |

Provisional failure states written before sending:

| Situation | Failure reason | Failure type |
|---|---|---|
| the mail has no recipient at all | the "no valid recipient" text of the relay layer | `mail_email_missing` |
| the mail has recipients | "Error without exception. Probably due to sending an email without computed recipients." | `unknown` |
| the notifications being locked | "Error without exception. Probably due to concurrent access update of notification records. Please see with an administrator." | `unknown` |

Failure classification at send time:

| Detected condition | Failure type |
|---|---|
| the relay reports no valid recipient and the sub-message had no recipient | `mail_email_missing` |
| the relay reports no valid recipient and the sub-message had recipients | `mail_email_invalid` |
| the relay reports an invalid sender | `mail_from_invalid` |
| the relay cannot determine a sender at all | `mail_from_missing` |
| the failure text mentions the corporate mail service's outbound spam exception | `mail_spam` |
| anything else | `unknown` |

Three exception classes are deliberately **not** caught and are re-raised so the job aborts rather than marking the mail failed: a memory exhaustion, a database error, and a disconnected relay session. In all three cases the transaction rolls back and the mail stays in the queue.

---

## 11. Template rules

| Rule | Message |
|---|---|
| A template may not target an abstract model. | "You may not define a template on an abstract model: <model>" |
| A contextual action's template must match the action's model. | "Mail template model of <action name> does not match action model." |
| Rendering a field the template does not define. | "Rendering of <field name> is not possible as not defined on template." |
| Rendering a field with no counterpart on the template. | "Rendering of <field name> is not possible as no counterpart on template." |
| A non-editor may not save a template containing a dynamic placeholder. | "Only members of Mail Template Editor group are allowed to edit templates containing sensible placeholders" |
| The structured engine failed. | "Failed to render template: <reference>" |
| The inline engine failed. | "Failed to render inline_template template: <text>\nError details: <error>" |
| The structured engine failed on a named field. | "Failed to render QWeb template for <field label>" |
| The rendering engine name is wrong. | "Template rendering supports only inline_template, qweb, or qweb_view (view or raw); received <engine> instead." |
| The record identifiers are not a list. | "Template rendering should only be called with a list of IDs. Received “<value>” instead." |
| An unknown rendering option was passed. | the call aborts naming the invalid options |
| An unsupported report kind is attached. | "Unsupported report type <type> found." |
| A recipient could not be found. | "No recipient found." |
| An address used as a recipient is malformed. | "Wrong email address <value>." |
| A new contact cannot be created because the address is malformed. | "<address> is not recognized as a valid email. This is required to create a new customer." |
| Creating a contact without enough information. | "You need to specify at least the partner_id or the name and the email" |

The dynamic-placeholder check runs at creation, at modification **and** at translation, so a template cannot be made dynamic through a translation.

---

## 12. Activity rules

| Rule | Message |
|---|---|
| The five protected shipped types may not change their model. | "You cannot modify <names> target model as they are are required in various apps." |
| Three of them may not be deleted. | "You cannot delete <names> as it is required in various apps." |
| The generic to-do type may not be archived. | "The 'To-Do' activity type is used to create reminders from the top bar menu and the command palette. Consequently, it cannot be archived or deleted." |
| A plan line's activity type, when it is model-specific, must match the plan's model. | "The activity type "<type name>" is not compatible with the plan "<plan name>" because it is limited to the model "<type model>"." |
| A plan line in "Default user" mode must name a user. | "When selecting "Default user" assignment, you must specify a responsible." |
| A plan line in "Ask at launch" mode must receive a user at launch. | "No responsible specified for <type name>: <summary or a hyphen>." |
| Launching a plan requires a target document. | "Plan-based scheduling are available only on documents." |
| Scheduling a personal activity requires an assignee. | "Scheduling personal activities requires an assigned user." |
| A server action of the activity kind may only target an activity-enabled model. | "A next activity can only be planned on models that use activities." |

Deleting an activity type is not refused for non-protected types: every activity of that type is first reassigned to the generic to-do type, then the type is deleted.

Archiving an activity type other than the generic to-do type is allowed and simply hides it from selection lists.

Completing an activity is permitted to the assignee even when they cannot read the related record, because the completion archives the activity and the archive itself is what the permission covers. The message posting is performed with elevated rights for the same reason.

---

## 13. Channel rules

### Creation

| Rule | Message |
|---|---|
| Contacts supplied at creation must use link or replace commands. | "Invalid value when creating a channel with members, only 4 or 6 are allowed." |
| Memberships supplied at creation must use creation commands. | "Invalid value when creating a channel with memberships, only 0 is allowed." |
| Only four member fields may be supplied at creation. | "Invalid field “<field>” when creating a channel with members." |
| A direct chat may not be created with more than two people. | "A chat should not be created with more than 2 persons. Create a group instead." |

The four allowed member fields are the contact, the guest, the unpin moment and the last-interest moment.

### Modification

| Rule | Message |
|---|---|
| The type may never change. | "Cannot change the channel type of: <names>" |
| The initial message and the parent may never change. | "Cannot change initial message nor parent channel of: <names>." |
| The authorization group of a sub-thread may not change. | "Cannot change authorized group of sub-channel: <names>." |
| The shipped whole-company group may not be deleted. | "You cannot delete those groups, as the Whole Company group is required by other modules." |

### Structure

| Rule | Message |
|---|---|
| A sub-thread's initial message must belong to the parent or to one of its sub-threads, and be a channel message. | "Cannot create <names>: initial message should belong to parent channel or one of its sub-channels." |
| A parent must not be a sub-thread, must be a channel or a group, and must share the child's type. | "Cannot create <names>: parent should not be a sub-channel and should be of type 'channel' or 'group'. The sub-channel should have the same type as the parent." |
| A direct chat may not have more than two members. | "A channel of type 'chat' cannot have more than two users." |
| Adding a member to a direct chat that already has one. | "Adding more members to this chat isn't possible; it's designed for just two people." |
| Creating a member without naming the channel. | "It appears you're trying to create a channel member, but it seems like you forgot to specify the related channel. To move forward, please make sure to provide the necessary channel information." |
| A member's channel, contact and guest may never change. | "You can not write on <field name>." |
| A public user may never be a member. | "Channel members cannot include public users." |
| The authorization group and the auto-subscription groups belong to channels only. | "For <names>, channel_type should be 'channel' to have the group-based authorization or group auto-subscription." |

### Access

The shipped record rule grants access to a channel when **either**:

- the type is **not** "channel", and the acting party is a member, or a member of its parent; **or**
- the type **is** "channel", and it has no authorization group, or the acting user belongs to that group.

A system administrator has unrestricted access.

Membership rows follow four rules, deliberately split so that "read any member of a channel I can see" and "create only myself" remain independent:

| Rule | Operations | Condition |
|---|---|---|
| own entries | write, delete | the row is the acting party's own **and** the channel is accessible by the two conditions above |
| read members | read | the channel is accessible by the two conditions above |
| join a group-restricted channel | create | the row is the acting party's own, the type is "channel", and the authorization group is empty or matches |
| invite into a group-restricted channel | create | the row is **not** the acting party's own, the type is "channel", and the authorization group is empty or matches; internal users only |
| invite into a private conversation | create | the row is **not** the acting party's own, the type is neither "channel" nor "chat", and the acting party is a member; internal users only |

The consequence is precise: a person may add **themselves** only to a public channel or to a channel whose authorization group they belong to. Into a group conversation they must be invited by an existing member. Into a direct chat nobody may be invited at all.

### Mentions and invitations

| Rule | Message |
|---|---|
| Inviting by address requires an internal user with read access on the channel. | "You don't have access to invite users to this channel." |
| Inviting by address is only allowed for a group, or a channel with no authorization group. | "Inviting by email is not allowed for this channel type (<type>)." |
| A delivery failure while inviting. | "There was an error when trying to deliver your Email, please check your configuration." — or, when the relay refused the connection, "Could not contact the mail server, please check your outgoing email server configuration." |
| Uploading an attachment where it is not allowed. | "You are not allowed to upload an attachment here." / "You are not allowed to upload attachments on this channel." |
| An attachment access token is missing. | "An access token must be provided for each attachment." |
| A token or record does not match. | "Non existing record or wrong token." |

Mentioned contacts are filtered before being stored: in a channel with an authorization group, only contacts whose users belong to that group survive; in any other type, only contacts that are already members survive. A mention of everyone expands to the full member list.

---

## 14. Guest and public-user rules

| Rule | Detail |
|---|---|
| A guest is identified solely by the pair (identifier, access token) carried in a cookie. | The token is readable only by the system group, so it never leaves the server except in the cookie itself. |
| A public user carrying a guest token posts as that guest: the author contact and the sender address are both cleared. | This is the only way a message can have a guest author. |
| A public user without a guest token may not post at all. | The posting path computes an author from the acting user, which for a public user resolves to the public contact, and the message access policy then refuses the creation. |
| The guest cookie separator character is excluded from the channel token alphabet. | So that a channel token can never be mistaken for a guest cookie. |
| Switching to another user always clears the guest from the context. | So that acting on behalf of a user can never be confused with acting as a guest. |

---

## 15. Live chat rules

| Rule | Message |
|---|---|
| Only a live chat operator may join a live chat channel. | "Only Live Chat operators can join Live Chat channels" |
| The review address must use the plain or secure hypertext transfer scheme and name a host. | "Invalid URL '<value>'. The Review Link must start with 'http://' or 'https://'." |
| The maximum concurrent session count must be strictly positive. | "Concurrent session number should be greater than zero." |
| A history may exist only on a live chat session. | "Cannot create history as it is only available for live chats: <names>." |
| A step of the question kind must have at least one answer. | "Step of type 'Question' must have answers." |
| Skills may only be linked or unlinked, never replaced wholesale. | "Write expertises: Only LINK and UNLINK commands are allowed." |
| Joining a session that needs help after somebody else already joined. | the operation returns a refusal rather than an error, so the client can show "somebody was faster" |

Additional behavioral rules:

- Changing the step type away from the question kind clears its answers.
- A triggering answer whose step is no longer earlier in the script is removed automatically.
- Duplicating a script re-links every triggering answer of the copied steps to the **copied** answers, matched by position.
- Creating a script without an operator contact creates an **archived** contact named after the script.
- A rule whose script is archived or has no step never matches.
- A session's working status must be empty once the session is closed.
- The visitor-leave message is only posted when the session already has at least one message.

---

## 16. Mailing group rules

| Rule | Message |
|---|---|
| Every moderator must have an address. | "Moderators must have an email address." |
| Automatic notification requires a text. | "The notification message is missing." |
| Automatic guidelines require a text. | "The guidelines description is missing." |
| A moderated list must have at least one moderator. | "Moderated group must have moderators." |
| Privacy "selected group of users" requires a group. | "The "Authorized Group" is missing." |
| Sending guidelines requires being an administrator or a moderator. | "Only an administrator or a moderator can send guidelines to group members." |
| Sending guidelines requires a non-empty text. | "The guidelines description is empty." |
| A closed list cannot send guidelines. | "You can not send guidelines for a closed group." |
| The guidelines template must exist. | "Template "mail_group.mail_template_guidelines" was not found. No email has been sent. Please contact an administrator to fix this issue." |
| A closed list cannot be joined. | "You can not join a closed group." |
| Joining with an unknown contact. | "The partner can not be found." |
| A post's wrapped message must belong to the list model. | "Group message can only be linked to mail group. Current model is <model>." |
| A post's wrapped message must point at this very list. | "The record of the message should be the group." |
| Relaying a post whose list does not match. | "The group of the message do not match." |
| Moderating a post that is not pending, one post. | "This message can not be moderated" |
| Moderating posts that are not pending, several posts. | "Those messages can not be moderated: <subjects>." |
| Creating a moderation rule from an invalid address. | "The email "<value>" is not valid." |
| A moderation rule address that cannot be normalized. | "Invalid email address “<value>”" |
| An unknown moderation status. | "Wrong status (<value>)" |

Behavioral rules:

- The alias security policy defaults to "everyone" for a public list and "followers" otherwise, and follows the privacy setting when it changes.
- Switching moderation on adds the acting user to the moderators.
- Guidelines are never sent to a banned address.
- A member is looked up first by contact, then by normalized address; leaving with the "all" flag removes every member with that address.
- The body of an incoming post has the platform's own list footer stripped before storage, so that a reply does not accumulate footers.
- A relayed post carries list headers: archive, subscribe, unsubscribe, one-click unsubscribe, precedence "list", and a header suppressing automatic out-of-office answers; plus, when the list has an address, the list identifier, the post address and a forge-to header. A reply carries the parent's identifier as its in-reply-to header.
- The author of a post never receives the relay of their own post.
- Relaying is batched by the configured session batch size, default 500.

---

## 17. Text message rules

| Rule | Message |
|---|---|
| Free numbers typed in the window must all format. | "Following numbers are not correctly encoded: <list>" |
| The recipient's number is invalid. | "Invalid recipient number. Please update it." |
| The recipient's name is invalid. | "Invalid recipient name." |
| Some recipients are invalid, in a batch. | "<count> invalid recipients" |
| Text messaging requires a non-transient thread-enabled model. | "Sending SMS can only be done on a not transient mail.thread model" |
| A contextual action's template must match the action's model. | "SMS template model of <action name> does not match action model." |
| An address used for account registration is invalid. | "Email <value> is invalid" |
| The sender name has the wrong shape. | "Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters." |
| The external provider account identifier has the wrong prefix. | "Invalid Twilio Account SID: must start with 'AC'" |
| The external provider account identifier has invalid characters. | "Invalid Twilio Account SID: must only contain alphanumeric characters after 'AC'" |
| No test number was supplied. | "Please set the number to which you want to send a test SMS." |
| The provider could not list the numbers. | "An error occurred while fetching the numbers." |

Provider failure explanations shown to the operator:

| Condition | Explanation |
|---|---|
| the destination country is not covered | "The destination country is not supported." |
| the content breaks the provider's rules | "The content of the message violates rules applied by our providers." |
| a trial account limitation | "Trial Account Limitation" and "Unverified recipient on Trial Account" |
| an unknown sending failure | "Unknown failure at sending, please contact Odoo support" |
| an unknown failure | "Unknown error, please contact Odoo support" |
| a duplicate was suppressed in a batch | "This SMS has been removed as the number was already used." |
| provider authentication failed | "Twilio Authentication Error" |
| the delivery-report address is wrong | "Twilio StatusCallback URL is incorrect" |

The monotonic status rule of the tracker is itself a business rule: a delivery report may never move a notification backwards. See the table in `entities.md`, section 44.3.

---

## 18. Postal mail rules

| Rule | Message |
|---|---|
| The document must be in the standard page format. | "Please use an A4 Paper format." |
| The document may not exceed the page limit. | "The document to be sent exceeds the maximum allowed limit of 8 pages." |
| The addressee's address must be complete. | "The address of the recipient is not complete" and the stored explanation "One or more required fields are empty." |
| The document could not be produced. | "The attachment could not be generated." |
| The service reported a formatting problem. | "The attachment of the letter could not be sent. Please check its content and contact the support if the problem persists." |
| The addressee's country is not served. | "The country of the partner is not covered by Snailmail." |
| There are not enough credits. | "You don't have enough credits to perform this operation.<br>Please go to your <link>iap account</link>." and the operator warning "Not enough credits for Snail Mail" |
| There is no registered account. | "You don't have an IAP account registered for this service.<br>Please go to <link>iap.odoo.com</link> to claim your free credits." |
| Anything else. | "An unknown error happened. Please contact the support." |

On success the stored explanation is "The document was correctly sent by post.<br>The tracking id is <identifier>" and the operator is informed with "Snail Mails are successfully sent".

Behavioral rules:

- The addressee's postal address is **copied** onto the letter at creation, so a later change of the contact does not rewrite what was posted.
- One Notification of the postal channel is created per letter, already marked read, so a postal send never fills anyone's inbox.
- Read access on the attachment is checked at creation and at every change of the attachment.
- When exactly one letter is re-queued, it is sent immediately; a batch is left to the job.

---

## 19. Digest rules

| Rule | Message |
|---|---|
| The periodicity must be one of the four values. | "Invalid periodicity set on digest" |
| An indicator the recipient may not read is dropped silently. | no message |

Behavioral rules:

- Only a digest in the activated state is considered by the job.
- Subscribing and unsubscribing are restricted to internal users and act only on the acting user.
- The automatic slow-down runs only for an automatic send, never for a manual one.
- Each send consumes exactly one tip, chosen among the tips the recipient has not yet received and whose group the recipient belongs to; a tip with no group is offered to everyone.
- The unsubscribe address carries a token bound to the pair (digest, recipient) and cannot be reused elsewhere.
- A delivery failure during a scheduled send is caught: the digest is simply left for the next run, with a log line.

---

## 20. Locking and concurrency rules

| Situation | Mechanism |
|---|---|
| Two transactions processing the same incoming message | A transaction-scoped advisory lock keyed on a hash of the message identifier. The loser treats the message as a duplicate. |
| Two clients marking the same conversation as fetched | The fetched-message write uses a "select for update, skipping locked rows" pattern, so a concurrent tab silently gives up instead of deadlocking. |
| The sending job and a concurrently arriving bounce | The mail is written to the failure state, and its notifications are written and **flushed immediately**, before the network call. The flush takes the row lock, so the bounce handler blocks until the send completes rather than interleaving. |
| Two sending jobs picking the same text messages | The queue selection locks the selected rows for update. |
| A relay throttle counter updated by two jobs | The counter lives on the relay row, which both jobs write, so the second waits for the first. |
| A message being sent while its transaction fails | Sending is deferred to a post-commit hook by default, executed on a fresh connection, so a rollback cannot leave a sent message behind. |
| An alias being marked invalid while the creation that failed is rolled back | The status write is performed on an **independent connection**, so it survives the rollback. |
| A tracking snapshot taken twice in one transaction | The first value wins: the store refuses to overwrite an existing snapshot entry. |
| A record deleted while its activity is being completed | The existence of the record is re-checked; a missing record makes the activity be deleted rather than archived, and its attachments removed. |
| A record deleted while a scheduled message is due | The posting fails, the author is notified, and the scheduled message is deleted. |

---

## 21. Invariants

An implementation must preserve the following at all times.

1. **A message is never notified twice to the same contact through the same channel.** Enforced by the uniqueness of (message, recipient) on the Notification.
2. **A follower row never exists twice for the same (model, record, contact).**
3. **An internal message never reaches a customer.** Enforced in three independent places: the follower query excludes a customer when the subtype is internal; the message access policy hides internal messages from non-internal users; and the reference chain of an outgoing notification prefers public ancestors.
4. **A notification status never moves backwards on the text-message channel.** Enforced by the monotonic rule.
5. **A sent electronic mail is never un-sent by a rollback.** Enforced by writing the failure state before the network call, and by deferring sending to a post-commit hook.
6. **An incoming message is processed at most once.** Enforced by the identifier lookup plus the advisory lock.
7. **A bounce never creates or updates a record.** Enforced by returning no route from the bounce branch.
8. **A reply to the platform's own bounce is never answered.** Enforced by the loop tag in the references.
9. **A tracked field change always produces a tracking entry in the same transaction as the change**, or none at all if the caller disabled tracking; there is no window in which the change is stored and the entry is not.
10. **An activity's completion date is stamped once and never rewritten.**
11. **A channel's type never changes**, so a direct chat can never become a public channel and expose its history.
12. **A channel member's channel, contact and guest never change**, so a membership can never be transferred.
13. **A live chat session always has an operator**, and a closed session never has a working status.
14. **A message keeps the company and the alias domain in force at its creation**, so the return path of an already-sent message is stable.
15. **A postal letter keeps the address in force at its creation.**
16. **Every recipient of a notification is either a contact or a bare address, never both**, so the delivery report can always be attributed.
17. **The separator of a channel member is the identifier of the first unread message**, so the unread count and the visual separator can never disagree.
18. **An author never receives a notification for their own message**, unless a caller explicitly asks for it or the author is an explicit recipient and the mention switch is on.
