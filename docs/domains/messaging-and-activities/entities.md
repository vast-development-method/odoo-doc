# Messaging and Activities — Entities

This document describes every entity of the messaging and activities domain: its purpose, its lifecycle, every field with its type and rules, its relations, its uniqueness rules, its defaults, its computed fields and the rule behind each, its ordering, its display rule, its archival behavior and its multi-company behavior.

Field tables use three columns: the storage name of the field in code font, its type, and the rules that govern it. When a field is computed the table states what it is computed from and whether the computed value is stored in the database; when it is stored and editable, the table says so, because such a field is recomputed only while its sources change and the user has not overridden it.

Several entities of this domain are **abstract behaviors**: they define fields and methods but have no table of their own. A concrete model that adopts a behavior gets a copy of its fields in its own table. Those are marked "abstract behavior" and have no storage name.

Contents:

1. [Thread (abstract behavior)](#1-thread-abstract-behavior)
2. [Message](#2-message)
3. [Message Subtype](#3-message-subtype)
4. [Follower](#4-follower)
5. [Notification](#5-notification)
6. [Tracking Value](#6-tracking-value)
7. [Message Reaction](#7-message-reaction)
8. [Link Preview and Message Link Preview](#8-link-preview-and-message-link-preview)
9. [Message Translation](#9-message-translation)
10. [Message Notification Schedule](#10-message-notification-schedule)
11. [Scheduled Message](#11-scheduled-message)
12. [Blacklist Entry](#12-blacklist-entry)
13. [Gateway Allowed Sender](#13-gateway-allowed-sender)
14. [Canned Response](#14-canned-response)
15. [Role](#15-role)
16. [Thread satellite behaviors](#16-thread-satellite-behaviors)
17. [Activity](#17-activity)
18. [Activity Type](#18-activity-type)
19. [Activity Plan and Activity Plan Template](#19-activity-plan-and-activity-plan-template)
20. [Activity behavior (abstract)](#20-activity-behavior-abstract)
21. [Outgoing Mail](#21-outgoing-mail)
22. [Alias](#22-alias)
23. [Alias Domain](#23-alias-domain)
24. [Alias behaviors (abstract)](#24-alias-behaviors-abstract)
25. [Incoming Mail Server](#25-incoming-mail-server)
26. [Outgoing Mail Server — fields added here](#26-outgoing-mail-server--fields-added-here)
27. [Template](#27-template)
28. [Render behavior, Composer behavior, Template reset behavior](#28-render-behavior-composer-behavior-template-reset-behavior)
29. [Composer](#29-composer)
30. [Wizards of the electronic mail area](#30-wizards-of-the-electronic-mail-area)

Entities 31 and beyond (push, presence, the event bus, channels, guests, live chat, mailing groups, text messages, digests, postal mail and the satellite fields on shared records) continue in the second half of this document.

31. [Push Device and Push Notification](#31-push-device-and-push-notification)
32. [Presence](#32-presence)
33. [Event Bus Entry and the bus sender behavior](#33-event-bus-entry-and-the-bus-sender-behavior)
34. [Channel](#34-channel)
35. [Channel Member](#35-channel-member)
36. [Call Session and Call History](#36-call-session-and-call-history)
37. [Guest](#37-guest)
38. [Conversation side records](#38-conversation-side-records)
39. [Live Chat Channel and Live Chat Rule](#39-live-chat-channel-and-live-chat-rule)
40. [Live Chat Member History, Expertise and Conversation Tag](#40-live-chat-member-history-expertise-and-conversation-tag)
41. [Chatbot Script, Step, Answer and Message](#41-chatbot-script-step-answer-and-message)
42. [Live Chat Session Report](#42-live-chat-session-report)
43. [Mailing Group entities](#43-mailing-group-entities)
44. [Text Message entities](#44-text-message-entities)
45. [Digest and Digest Tip](#45-digest-and-digest-tip)
46. [Postal Letter](#46-postal-letter)
47. [Satellite fields on shared records](#47-satellite-fields-on-shared-records)

---

## 1. Thread (abstract behavior)

Thread (`mail.thread`, abstract behavior, no table of its own).

### Purpose

The Thread behavior turns any business record into a discussion topic. A model that adopts it gains:

- a conversation history (a list of Messages pointing at the record by model name and record identifier);
- a list of Followers, each with the set of subtypes they are subscribed to;
- automatic subscription of the creator and of the newly assigned responsible;
- field change tracking, which turns a modification of a marked field into a Tracking Value attached to a logged Message;
- the whole notification machinery: computing who must be told about a new message, and delivering that to the in-application inbox, to electronic mail and to browser push;
- the incoming routing hooks that let an electronic mail create or update a record of the model;
- a set of counters used by list and form views (unread count, delivery-error count, attachment count).

### Behavior attributes

A model adopting the Thread behavior may set the following class-level attributes. They are part of the contract and an implementation must expose the same switches.

| Attribute | Default | Meaning |
|---|---|---|
| `_mail_flat_thread` | true | When true, a posted message that is given no parent is automatically attached to the **first relevant ancestor message** of the record, producing a single flat conversation. When false, messages without a parent stay at the root and genuine threads are possible. |
| `_mail_post_access` | `write` | The permission required **on the document** to be allowed to create a Message on it. The two useful values are `write` (editing the record is required to talk about it) and `read` (reading is enough). |
| `_mail_thread_customer` | false | When true, the model has a strong tie with one main customer; that customer is automatically subscribed when it appears among the direct recipients of a posted message. |
| `_primary_email` | `email` | The name of the field holding the main electronic mail address of the record. Used when an incoming message creates a record, when computing default recipients, and when detecting an incoming loop. |
| `_CUSTOMER_HEADERS_LIMIT_COUNT` | 50 | Above this number of external addresses on one notification, the list of external addresses is **not** added to the outgoing header that enables reply-to-all, to avoid leaking a large audience. |

### Context switches

The behavior reacts to a set of per-call switches. They are part of the contract because callers in other domains rely on them.

| Switch | Default | Effect |
|---|---|---|
| `mail_create_nosubscribe` | off | At creation, do not subscribe the acting user to the new record. Also implies not subscribing the author of any message posted during the same call, unless that is separately overridden. |
| `mail_create_nolog` | off | At creation, do not log the automatic "<document type> created" message. |
| `mail_notrack` | off | At creation and modification, perform no field change tracking. |
| `tracking_disable` | off | Disable **all** thread features for the call: no automatic subscription, no tracking, no automatic log. |
| `mail_notify_force_send` | on | Try to send notification electronic mails immediately rather than leaving them in the queue, as long as the count stays below the force-send limit. |
| `mail_notify_author` | off | Notify the author of their own message. |
| `mail_notify_author_mention` | off | Notify the author when the author is among the direct recipients of the message. |
| `mail_auto_subscribe_no_notify` | off | Perform automatic subscription silently, without the assignment notification. |
| `mail_post_autofollow` | off | Subscribe the direct recipients of a posted message. |
| `mail_post_autofollow_author_skip` | off | Do not subscribe the author of a posted message. |

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `message_is_follower` | Boolean, computed, not stored | True when the contact of the acting user follows this record. Computed from the Follower list with elevated rights. Searchable: the search resolves to the set of record identifiers followed by the acting user's contact. |
| `message_follower_ids` | Sub-records: Follower | All Follower rows pointing at this record. Readable only by internal users. Not a database column: the link is by model name plus record identifier, so no foreign key exists. |
| `message_partner_ids` | Multiple links to Contact, computed, not stored, writable | The contacts extracted from the Follower list. Writing it computes the difference: contacts added are subscribed, contacts removed are unsubscribed (the removals are executed after all additions, because unsubscribing deletes rows and invalidates the cache). Readable only by internal users. Searching it is restricted: a non-internal user may only filter on themselves or on their commercial entity, otherwise the search is refused with "Portal users can only filter threads by themselves as followers." |
| `message_ids` | Sub-records: Message | All Messages pointing at this record **except** those of type "user specific notification". Access checks on the sub-records are bypassed at query level because the Message model applies its own policy. |
| `has_message` | Boolean, computed, not stored | True when at least one Message exists for the record, including user-specific notifications. Searchable. |
| `message_needaction` | Boolean, computed, not stored | True when the count below is non-zero. |
| `message_needaction_counter` | Integer, computed, not stored | Number of Messages on this record for which the acting user's contact has an unread Notification, excluding user-specific notifications. |
| `message_has_error` | Boolean, computed, not stored | True when the count below is non-zero. |
| `message_has_error_counter` | Integer, computed, not stored | Number of Messages on this record **authored by the acting user's contact** that carry at least one Notification in status "bounced" or "exception", excluding user-specific notifications. |
| `message_attachment_count` | Integer, computed, not stored | Number of attachments whose owning model and record identifier point at this record. Readable only by internal users. |

### Lifecycle hooks on the record

**At creation** the behavior performs, in order:

1. If the "no subscribe" switch is on and the "skip author subscription" switch is not explicitly set, turn the latter on as well for the whole call.
2. If the "disable everything" switch is on: create the records, mark them as never-tracked, return.
3. Create the records through the normal creation path.
4. Unless the "no subscribe" switch is on, and provided the acting user is active and is not a portal or public user, insert the acting user's contact as a Follower of every created record, with the default subtypes of the model, without checking for existing rows.
5. For each created record, build a value map from the explicit creation values plus every context default (a context key named `default_<field>` contributes `<field>` when the field is not already in the values), then run automatic subscription on that map with the existing-follower policy "update".
6. Unless the "no log" switch is on, log the creation:
   - if the model defines a creation subtype, post a message with that subtype, authored by the acting user, whose body is the creation sentence wrapped in a marker that hides it in the interface;
   - otherwise log the creation sentence as a plain internal note, in one batch for all records.
   The creation sentence is "<document type name> created", where the document type name is the translated description of the model.
7. Discard any tracking accumulated so far, then, unless the "no track" switch is on, register for each record the list of tracked fields that received a non-empty value at creation, and schedule the "post the template linked to those changes" step to run at the end of the transaction. A field set to a falsy value at creation is **not** considered a change.

**At modification** the behavior performs:

1. If the "disable everything" switch is on, write and return.
2. Unless the "no track" switch is on, snapshot the current value of every tracked field of the record set **before** the write (see [calculations.md](calculations.md), tracking section).
3. Write.
4. Run automatic subscription on the written values.

**At deletion** the behavior performs, in order: discard pending tracking; delete every Message pointing at the records; delete the records; delete every Follower pointing at the records; delete every Scheduled Message pointing at the records. Messages and Followers cannot cascade at database level because the link is by model name plus identifier rather than by foreign key.

**At duplication** tracking is disabled for the duration of the copy, so the many intermediate values written during a copy do not produce tracking entries.

### Multi-company behavior

The Thread behavior itself has no company field. It derives the company of a record through the following rule, used for the reply address, the alias domain of a message and the branding of notification electronic mails:

1. If the model has a field named `company_id` (the company link), use its value.
2. If that value is empty, or the model has no such field, use the company given by the caller, defaulting to the active company of the acting user.

The alias domain of a record is then the alias domain of that company; if the company has none, the alias domain of the default company; if that is also empty, the first alias domain by ordering.

---

## 2. Message

Message (`mail.message`, table `mail_message`).

### Purpose

One entry of a conversation. A Message may be attached to a record (the usual case), or free-standing (used for private notifications). It carries the body, the author, the direct recipients, the type, the subtype, the attachments, the parent link, the identifiers needed for electronic mail threading, and the list of per-recipient Notifications.

### Lifecycle

1. **Created** by the thread posting operation, by the logging operation, by the notification operation, by the incoming gateway, by the composer, or directly through the remote interface by an authorized caller.
2. **Notified**: zero or more Notification rows are created, one per recipient and per delivery channel, and the corresponding Outgoing Mail rows are produced for the electronic mail channel.
3. **Edited**: only a message of type "comment" that carries no Tracking Value may have its body changed; the edit stamps an "edited" marker at the end of the body and discards every cached translation.
4. **Emptied**: a message whose body becomes empty, that has no attachment, no subtype description and no readable tracking value, is considered void; the caller then removes its link previews. (The record itself stays; the interface hides it.)
5. **Deleted**: allowed only to a caller with write-like access on the related record. Deleting cascades to the attachments that belong to the message itself, and broadcasts a deletion event to every recipient contact that has a user and to the authors of its notifications.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `subject` | Text | Free subject. Often empty for in-application comments; filled for electronic mail. |
| `date` | Date and time | Default: the current moment. For an incoming electronic mail, the parsed value of the message date header converted to coordinated universal time. |
| `body` | Rich text | Default empty. Style attributes are sanitized. At creation, every embedded image encoded inline in the body is extracted into a real attachment, given an access token, linked to the message, and the inline data is replaced by a reference to that attachment with an added marker so the interface does not display the image twice. |
| `preview` | Text, computed, not stored | The plain-text beginning of the body, shortened to at most 190 characters with an ellipsis. Used as the preview line of a notification electronic mail. |
| `linked_message_ids` | Multiple links to Message, computed, not stored | The messages referenced from inside the body through a redirect link carrying the message model and identifier. The result is filtered by the access rights of the acting user, never elevated, because the body is user input. |
| `message_link_preview_ids` | Sub-records: Message Link Preview | The link previews attached to this message. Readable only by the configuration-manager group. |
| `reaction_ids` | Sub-records: Message Reaction | The emoji reactions placed on this message. Readable only by the system group. |
| `attachment_ids` | Multiple links to Attachment | The files attached to the message. Attachments are also linked to the *document* through their own model and record fields; this relation links them to the message. Access checks on this relation are bypassed at query level. At creation and at modification, any attachment that does not already belong to the same document is checked for read access. |
| `parent_id` | Link to Message | The message this one answers. Indexed when not empty. On deletion of the parent, set to empty. |
| `child_ids` | Sub-records: Message | The answers to this message. |
| `model` | Text | The transport name of the model of the related record. Empty for a free-standing message. Only a system administrator may change it after creation. |
| `res_id` | Record reference | The identifier of the related record, interpreted against `model`. Only a system administrator may change it after creation. |
| `record_name` | Text, computed, not stored | The display name of the related record, read with elevated rights. Empty when the record no longer exists or the model is unknown. |
| `record_alias_domain_id` | Link to Alias Domain | The alias domain that was in force for the related record when the message was created. Used to compute the return path of outgoing notifications. On deletion of the domain, set to empty. |
| `record_company_id` | Link to Company | The company that was in force for the related record when the message was created. On deletion, set to empty. |
| `message_type` | Selection | Required, default "comment". See the table of message types below. |
| `subtype_id` | Link to Message Subtype | Indexed. Decides which followers are notified. On deletion of the subtype, set to empty. |
| `mail_activity_type_id` | Link to Activity Type | Set on the message that records the completion of an activity, so the history can be filtered by activity type. Indexed when not empty. On deletion, set to empty. |
| `is_internal` | Boolean | "Employee only". When true the message is hidden from portal and public users **regardless** of the subtype. |
| `email_from` | Text | The sender address in formatted form. Kept even when an author contact is found, and used as the displayed sender when no author contact matched. A non-internal caller may not set or change it. |
| `author_id` | Link to Contact | Indexed. The author. On deletion of the contact, set to empty. A non-internal caller may not set or change it. |
| `author_avatar` | Image, related to the author | Read-through to the author's small avatar. |
| `author_guest_id` | Link to Guest | Set instead of the author when a public visitor identified only by a guest token posts the message. |
| `is_current_user_or_guest_author` | Boolean, computed, not stored | True when the acting user's contact is the author, or the acting guest is the author guest. |
| `partner_ids` | Multiple links to Contact | The **direct recipients**: the parties explicitly addressed in addition to the followers selected by the subtype. Archived contacts are kept visible in the relation. |
| `incoming_email_to` | Long text | For an incoming electronic mail, the comma-separated list of addresses that were in the "to" header, with the alias addresses of the installation removed. Used to avoid notifying by electronic mail someone who already received the original. |
| `incoming_email_cc` | Text | Same for the carbon-copy header. |
| `outgoing_email_to` | Text | A comma-separated list of extra addresses that must receive the notification even though they match no contact. Each becomes a recipient entry of type "customer" with delivery by electronic mail. |
| `notified_partner_ids` | Multiple links to Contact through the notification table | The contacts that have a Notification for this message. The set changes over time because old notifications are garbage-collected. |
| `needaction` | Boolean, computed, not stored | True when the acting user's contact has an unread Notification on this message. Searchable. |
| `has_error` | Boolean, computed, not stored | True when any Notification of this message is in status "bounced" or "exception". Searchable. |
| `notification_ids` | Sub-records: Notification | One row per recipient and channel. Access checks bypassed at query level. |
| `starred_partner_ids` | Multiple links to Contact | The contacts that marked the message as a favourite. |
| `pinned_at` | Date and time | When set, the message is pinned in its channel; the value is the moment of pinning. |
| `starred` | Boolean, computed, not stored, per user | True when the acting user's contact is in the favourite list. Searchable. |
| `tracking_value_ids` | Sub-records: Tracking Value | The recorded field changes carried by this message. Readable only by the system group; individual rows are then filtered again by field-level access (see [business-rules.md](business-rules.md)). |
| `reply_to_force_new` | Boolean | When true, answers to this message must **not** be threaded back onto the same record. This changes the generated message identifier so the incoming router does not match it. |
| `message_id` | Text | The globally unique electronic mail identifier of the message. Indexed, read-only, not copied. Generated at creation if not supplied (see the grammar in [calculations.md](calculations.md)). |
| `reply_to` | Text | The reply address, in formatted form. Computed at creation when not supplied. |
| `mail_server_id` | Link to Outgoing Mail Server | The relay to prefer for this message. |
| `email_layout_xmlid` | Text | The external identifier of the layout used to wrap the body in notification electronic mails. Not copied. |
| `email_add_signature` | Boolean | Default true. When true and the author has a user, that user's signature is appended to notification electronic mails. |
| `mail_ids` | Sub-records: Outgoing Mail | The outgoing mails built from this message. Readable only by the system group. |

Ordering: newest first (descending identifier). Display name: the subject.

Indexes: one on the pair (model, record identifier) and one on the triple (model, record identifier, identifier).

### Message types

| Value | Label | Meaning |
|---|---|---|
| `email` | Incoming Email | Produced by the incoming gateway from a received electronic mail. |
| `comment` | Comment | Produced by a person: the composer, the conversation window, the portal. |
| `email_outgoing` | Outgoing Email | Produced by a mass mailing. |
| `notification` | System notification | Produced by the system, for example a tracking message. |
| `auto_comment` | Automated Targeted Notification | Produced by an automatic notification mechanism, for example an acknowledgement or a stage-change template. |
| `out_of_office` | Out-of-office Message | The automatic answer of an absent user. |
| `user_notification` | User Specific Notification | A notification addressed to specific recipients that must **not** appear in the conversation of the record. |

The type "user specific notification" is reserved: the posting operation refuses it and directs the caller to the notification operation instead. Conversely the notification operation always produces that type.

### Access policy

The Message model does **not** use the ordinary record-rule mechanism alone. It layers a five-branch policy on top. The full rules with their exact error text are in [business-rules.md](business-rules.md); the summary is:

- **Read** is granted when the acting contact is the author, or the acting user created the row, or the acting contact is a direct recipient, or the acting contact has a Notification on the message, or the acting user may read the related record.
- **Create** is granted when the message is free-standing, or the acting contact follows the related record, or the acting contact is a direct recipient of the parent message, or the acting user has the permission named by the model's post-access attribute on the related record.
- **Write** is granted when the acting contact is the author, or is a direct recipient, or has a Notification, or the acting user may write the related record.
- **Delete** is granted when the acting user may write the related record.
- **Non-internal users** additionally never see a message that is marked employee-only, that has no subtype, or whose subtype is marked internal; and for read and create the message must additionally be of type "comment".

Searching applies the same filter: the ordinary query runs first, then every returned identifier is checked against the five branches and the forbidden ones are dropped silently.

Exporting messages is restricted to administrators, with the message "Only administrators are allowed to export mail message".

---

## 3. Message Subtype

Message Subtype (`mail.message.subtype`, table `mail_message_subtype`).

### Purpose

A subtype is the *reason* a message exists, and it is the unit a follower subscribes to. A follower of a record subscribes to a set of subtypes; a message whose subtype is in that set notifies that follower, a message whose subtype is not in that set does not. A subtype may be global (applying to all models) or bound to one model.

Subtypes also carry the **parent relationship**, which is the mechanism by which following a container record (say a project) subscribes you to messages of its children (say tasks).

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. The label shown in the follower subscription editor. |
| `description` | Long text, translatable | Optional. When set, this text is prepended to the displayed body of every message carrying the subtype. When empty, nothing is prepended. A message with an empty body but a subtype that has a description is **not** considered void. |
| `internal` | Boolean | When true, messages with this subtype are visible only to internal users. |
| `parent_id` | Link to Message Subtype | The *parent* subtype in the automatic-subscription sense. Despite the name, the relation points from the container-model subtype to the child-model subtype: a subtype defined on the container model names, through this link, the subtype of the child model that must be added to the follower. On deletion, set to empty. |
| `relation_field` | Text | The name of the field on the **child** model that points back to the container record. Used to detect that the container changed and therefore that the container's followers must be propagated. |
| `res_model` | Text | The transport name of the model this subtype applies to. Empty means "all models". |
| `default` | Boolean | Default true. When true the subtype is part of the default subscription of a new follower. |
| `sequence` | Integer | Default 1. Ordering key. |
| `hidden` | Boolean | When true the subtype is not offered in the follower subscription editor. |
| `track_recipients` | Boolean | When true, the interface shows **all** recipients of a message with this subtype rather than only the noteworthy ones (external recipients and failures). |

Ordering: by sequence then identifier.

### Cache invalidation

Creating, modifying or deleting any subtype clears the cached automatic-subscription map for every model. An implementation must invalidate the equivalent cache.

### Derived sets

Two derived sets are computed and cached per model. Both are part of the contract.

**Default subtypes of a model.** The subtypes marked default whose model is that model or empty. They are split into the *internal* ones (marked internal) and the *external* ones (the rest). A new follower that is a customer (a contact without a user, or with only portal or public users) receives the **external** set; any other follower receives the **full** set.

**Automatic-subscription map of a model.** Built from all subtypes whose model is empty, or equals the model, or whose parent subtype's model equals the model. It yields five pieces of data:

| Piece | Content |
|---|---|
| child identifiers | Every subtype whose model is empty or equals the model. |
| default identifiers | Among those, the ones marked default. |
| all internal identifiers | Every subtype in the whole selection that is marked internal. |
| parent map | For every subtype that names a relation field, the pair (that subtype, its parent subtype). |
| relation map | For every subtype that names a relation field, its model mapped to the set of relation field names. |

### Shipped subtypes

Three subtypes are shipped with the platform and referenced by external identifier throughout the system.

| External identifier | Name | Default | Internal | Description | Purpose |
|---|---|---|---|---|---|
| `mail.mt_note` | Note | no | yes | empty | The subtype of an internal log. Posting without an explicit subtype falls back to this one. |
| `mail.mt_comment` | Discussions | yes | no | empty | The subtype of a public discussion message. Notifying the followers of a record uses this one. |
| `mail.mt_activities` | Activities | no | yes | empty | The subtype of the message logged when an activity is completed. |

---

## 4. Follower

Follower (`mail.followers`, table `mail_followers`).

### Purpose

One subscription: a contact follows one record of one model, with an explicit set of subtypes. The absence of a row means the contact does not follow the record; a row with an empty subtype set means the contact follows the record but receives no notification from subtype-driven messages (they may still be reached as a direct recipient).

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `res_model` | Text | Required, indexed. The transport name of the followed model. **No integrity check** is performed on the value for performance; rows of deleted models are cleaned by the model registry. |
| `res_id` | Record reference | Indexed. The identifier of the followed record, interpreted against `res_model`. |
| `partner_id` | Link to Contact | Required, indexed. On deletion of the contact, the row is deleted. |
| `subtype_ids` | Multiple links to Message Subtype | The subtypes this follower is subscribed to. |
| `name` | Text, related to the contact | The contact name. |
| `email` | Text, related to the contact | The contact address. |
| `is_active` | Boolean, related to the contact | Whether the contact is active. |

The model keeps no creation or modification stamps.

Uniqueness: the triple (model, record identifier, contact) is unique; the violation message is "Error, a partner cannot follow twice the same object."

Display name: the display name of the contact, read with elevated rights so that a portal contact can still be named across companies.

### Insertion policies

Inserting followers is always done through one operation that takes a policy for rows that already exist:

| Policy | Behavior for an existing row |
|---|---|
| `skip` | Leave it untouched. This is the default when no explicit subtypes are given. |
| `force` | Delete every existing row for the given records and contacts first, then create fresh rows with the given subtypes. |
| `replace` | Keep the row, add the subtypes that are missing, remove the subtypes that are no longer wanted. |
| `update` | Keep the row, add the subtypes that are missing, remove nothing. |

When no explicit subtype set is given, the subtypes are computed per contact: the **external** default set for a contact that is a customer, the **full** default set otherwise. The caller may pass the list of known customer contacts to avoid the lookup.

---

## 5. Notification

Notification (`mail.notification`, table `mail_notification`).

### Purpose

One delivery of one Message to one recipient through one channel. It is the record that answers "was this person told, how, and did it work".

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `author_id` | Link to Contact | The author of the message, denormalized so that the "my messages that failed" query is a single index scan. On deletion, set to empty. |
| `mail_message_id` | Link to Message | Required, indexed. On deletion of the message, the row is deleted. |
| `mail_mail_id` | Link to Outgoing Mail | Indexed. The outgoing mail that carries this notification, when the channel is electronic mail. |
| `res_partner_id` | Link to Contact | Indexed. The recipient. On deletion of the contact, the row is deleted. May be empty only for an electronic-mail notification addressed to a bare address. |
| `mail_email_address` | Text | The recipient address when no contact matched. Expected to be normalized, except when the row records a failure caused by an invalid address. |
| `notification_type` | Selection | Required, indexed, default "inbox". Base values `inbox` and `email`; the text-message capability adds `sms`, the postal capability adds `snail`. |
| `notification_status` | Selection, indexed | Default "ready". See the state table below. |
| `is_read` | Boolean, indexed | Whether the recipient has acknowledged the in-application notification. Electronic-mail notifications are created already marked read, because there is no read-back channel. |
| `read_date` | Date and time | Stamped automatically the moment the read flag is set to true, at creation or at modification. Not copied. |
| `failure_type` | Selection | The machine-readable failure cause. See the table below. |
| `failure_reason` | Long text | The human-readable failure cause, typically the text returned by the relay. Not copied. |

The model keeps no creation or modification stamps.

### Statuses

| Value | Label | Meaning |
|---|---|---|
| `ready` | Ready to Send | Created, not yet handed to the channel. |
| `process` | Processing | Handed to an intermediary that has not yet confirmed (used by the text-message channel). |
| `pending` | Sent | Handed over successfully; delivery not yet confirmed (used by the text-message channel; electronic mail does not distinguish). |
| `sent` | Delivered | Delivered. |
| `bounce` | Bounced | The recipient's system returned the message. |
| `exception` | Exception | The attempt failed. |
| `canceled` | Cancelled | The attempt was abandoned deliberately. |

### Failure types

Base set:

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `mail_bounce` | Bounce |
| `mail_spam` | Detected As Spam |
| `mail_email_invalid` | Invalid email address |
| `mail_email_missing` | Missing email address |
| `mail_from_invalid` | Invalid from address |
| `mail_from_missing` | Missing from address |
| `mail_smtp` | Connection failed (outgoing mail server problem) |
| `mail_bl` | Blacklisted Address |
| `mail_optout` | Opted Out |
| `mail_dup` | Duplicated Email |

The text-message capability adds: missing number, wrong number format, insufficient credit, country not supported, registration needed, and the provider-specific authentication, callback, missing sending number and identical sender-and-recipient failures. The postal capability adds: credit error, trial error, no price available, missing required fields, format error and generic error.

### Constraints and indexes

| Rule | Condition | Message |
|---|---|---|
| Recipient required for inbox | A row of type "inbox" must have a recipient contact. | "Customer is required for inbox notification" |
| Recipient or address required for electronic mail | A row of type "email" must have a failure type, or a recipient contact, or a non-empty address. | "Customer or email is required for inbox / email notification" |
| One notification per message and contact | The pair (message, recipient contact) is unique when the contact is set. | database-level uniqueness |

Indexes: (recipient, read flag, status, message) for the inbox counters; (author, status) restricted to the failing statuses for the "my failures" query.

### Modification restrictions

Changing the message or the recipient of an existing notification is refused for anyone who is not an administrator, with the message "Can not update the message or recipient of a notification."

Creating notifications requires read access on the referenced messages.

### Garbage collection

A maintenance routine deletes notifications that are read, whose read date is older than a retention window (180 days by default), whose recipient is **not** a customer, and whose status is "delivered" or "cancelled". It deletes at most one batch per run and reports whether more remain.

### Client filtering

When the list of notifications of a message is sent to the interface, only the *noteworthy* ones are included: those in status bounced, exception or cancelled; those whose recipient is a customer; those addressed to a bare address; and, for the rest, only when the subtype of the message is marked "track recipients".

---

## 6. Tracking Value

Tracking Value (`mail.tracking.value`, table `mail_tracking_value`).

### Purpose

One recorded change of one tracked field: the old value, the new value, and enough type information to render both. Tracking Values are always attached to a Message, and that message is what makes the change visible in the conversation.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `field_id` | Link to Field definition | Indexed, read-only. The field whose value changed. On deletion of the field definition, set to empty; the row then relies on the snapshot below. |
| `field_info` | Structured value | A snapshot of the field description, used when the field no longer exists and for dynamic properties. Holds a label, a name and a type. |
| `old_value_integer` | Integer, read-only | The old value for integer, boolean and link fields (for a link, the identifier). |
| `old_value_float` | Decimal, read-only | The old value for decimal and monetary fields. |
| `old_value_char` | Text, read-only | The old value for text, selection, link (the display name) and list fields (the comma-joined display names). |
| `old_value_text` | Long text, read-only | The old value for long-text fields. |
| `old_value_datetime` | Date and time, read-only | The old value for date and date-and-time fields (a plain date is stored at midnight). |
| `new_value_integer`, `new_value_float`, `new_value_char`, `new_value_text`, `new_value_datetime` | same types | The corresponding new values. |
| `currency_id` | Link to Currency, read-only | Set for a monetary field, so the amount can be rendered with its currency. On deletion, set to empty. |
| `mail_message_id` | Link to Message | Required, indexed. On deletion of the message, the row is deleted. |

Ordering: newest first (descending identifier). Because display is by ascending sequence, the creation order is deliberately reversed (see [calculations.md](calculations.md)).

Display name: the field.

### Which storage column holds which type

| Field type | Old and new columns used | Extra |
|---|---|---|
| integer | integer columns | — |
| decimal | decimal columns | — |
| text | text columns | — |
| long text | long-text columns | — |
| date and time | date-and-time columns | — |
| monetary | decimal columns | the currency link is filled from the field's currency companion field |
| date | date-and-time columns | the date is combined with midnight before storing |
| boolean | integer columns | stored as 0 or 1, displayed as false or true |
| selection | text columns | the **label** of the value is stored, not the key; a missing label falls back to the key; an empty value stores the empty string |
| link | integer columns hold the identifiers, text columns hold the display names | an empty value stores identifier 0 and an empty name |
| list (one-to-many, many-to-many, tag set) | text columns hold the comma-joined display names | an element with no display name is rendered as "Unnamed <model description> (<identifier>)" |

Any other field type is rejected at creation time as unsupported.

### Access to a tracking value

Tracking Values are readable only by the system group as a whole; individual rows are then filtered:

- a row **linked to a field** is visible when the acting user may read that field on its model;
- a row **not linked to a field** (the field was deleted) is visible only to a system administrator.

A second, stricter filter is applied when a tracking summary is embedded in a notification: only rows linked to an existing field that has **no access group at all** are included, because the notification is rendered once for many recipients.

---

## 7. Message Reaction

Message Reaction (`mail.message.reaction`, table `mail_message_reaction`).

### Purpose

One emoji placed on one message by one party. The party is either a contact or a guest, never both.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `message_id` | Link to Message | Required, read-only, indexed. On deletion of the message, the row is deleted. |
| `content` | Text | Required, read-only. The emoji itself. |
| `partner_id` | Link to Contact | Read-only. On deletion, the row is deleted. |
| `guest_id` | Link to Guest | Read-only. On deletion, the row is deleted. |

The model keeps no creation or modification stamps. Ordering: newest first.

Uniqueness: (message, content, contact) when the contact is set; (message, content, guest) when the guest is set.

Constraint: exactly one of contact and guest must be set — "A message reaction must be from a partner or from a guest."

### Behavior

Adding a reaction that already exists is a no-op; removing one that does not exist is a no-op. After either operation the **whole group** for that message and that emoji is recomputed and broadcast on the message's broadcast channel: either the full list of reactions of that group, or a deletion marker naming the message and the content when the group became empty.

---

## 8. Link Preview and Message Link Preview

Two records model link previews: one caches the preview of a web address (shared by every message that mentions it), the other attaches a cached preview to one message with an order and a hidden flag.

### Link Preview

Link Preview (`mail.link.preview`, table `mail_link_preview`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `source_url` | Text | Required. The web address the preview describes. Unique. |
| `og_type` | Text | The declared type of the target page. |
| `og_title` | Text | The declared title. |
| `og_site_name` | Text | The declared site name. |
| `og_image` | Text | The address of the declared preview image. |
| `og_description` | Long text | The declared description. |
| `og_mimetype` | Text | The declared content type of the target. |
| `image_mimetype` | Text | The content type of the preview image. |
| `create_date` | Date and time | Indexed, so stale previews can be expired efficiently. |
| `message_link_preview_ids` | Sub-records: Message Link Preview | The attachments of this preview to messages. Readable only by the configuration-manager group. |

Display name: the web address.

### Message Link Preview

Message Link Preview (`mail.message.link.preview`, table `mail_message_link_preview`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `message_id` | Link to Message | Required, indexed. On deletion, the row is deleted. |
| `link_preview_id` | Link to Link Preview | Required, indexed. On deletion, the row is deleted. |
| `sequence` | Integer | Display order inside the message. |
| `is_hidden` | Boolean | When true the preview is suppressed for that message (the reader dismissed it). |
| `author_id` | Link to Contact, related to the message | Used to decide who may dismiss the preview. |

Ordering: by sequence then identifier. Uniqueness: (message, link preview).

When the previews of a message are sent to the interface, the hidden ones are removed and the rest are sorted by sequence then identifier.

---

## 9. Message Translation

Message Translation (`mail.message.translation`, table `mail_message_translation`).

### Purpose

The cached machine translation of one message body into one target language, so the same translation is not requested twice.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `message_id` | Link to Message | Required. On deletion, the row is deleted. |
| `source_lang` | Text | Required. The language the translation service detected in the source body. |
| `target_lang` | Text | Required. The shortened language code requested. |
| `body` | Rich text | Required. The translated body as returned by the service. Style attributes are sanitized. |
| `create_date` | Date and time | Indexed, so the cache can be expired by age. |

Uniqueness: (message, target language).

Editing the body of a message deletes every translation of that message.

---

## 10. Message Notification Schedule

Message Notification Schedule (`mail.message.schedule`, table `mail_message_schedule`).

### Purpose

A message that has **already been posted** but whose notifications must not go out yet. The message exists and is visible in the conversation; only the delivery is deferred. This is distinct from a Scheduled Message, which is not posted at all.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `mail_message_id` | Link to Message | Required. On deletion, the row is deleted. |
| `notification_parameters` | Long text | The serialized set of notification parameters captured at posting time, replayed when the notification finally runs. |
| `scheduled_datetime` | Date and time | Required. The moment at which the notification must be produced. |

Ordering: by scheduled moment descending then identifier descending. Display name: the message.

---

## 11. Scheduled Message

Scheduled Message (`mail.scheduled.message`, table `mail_scheduled_message`).

### Purpose

A message that has **not** been posted. The composer captured everything needed to post it and stored it for later. Until the moment arrives the conversation shows nothing; the author sees a pending entry and may edit or cancel it.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `subject` | Text | The subject to use when posting. |
| `body` | Rich text | The body to use. Style attributes sanitized. |
| `scheduled_date` | Date and time | Required. When to post. |
| `attachment_ids` | Multiple links to Attachment | The files to attach. Access checks bypassed at query level. |
| `composition_comment_option` | Selection | Either "reply all" or "forward", remembered from the composer. |
| `model` | Text | Required. The model of the target record. |
| `res_id` | Record reference | Required. The target record. |
| `author_id` | Link to Contact | Required. Who will appear as the author. |
| `partner_ids` | Multiple links to Contact | The direct recipients to use. |
| `is_note` | Boolean | Default false. When true the message will be posted as an internal note, otherwise as a discussion message. |
| `notification_parameters` | Long text | The serialized notification parameters to replay. |
| `send_context` | Structured value | Extra sending context captured by the composer. |

---

## 12. Blacklist Entry

Blacklist Entry (`mail.blacklist`, table `mail_blacklist`). Adopts the Thread behavior.

### Purpose

One address that must never receive a mass message. The blacklist is global (not per company, not per list) and is keyed on the **normalized** address.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `email` | Text | Required, indexed for partial-word search. Case-insensitive. Tracked with order 1. Stored normalized. |
| `active` | Boolean | Default true. Tracked with order 2. Archiving the row is how an address is removed from the blacklist while keeping the history. |

Uniqueness: the address — "Email address already exists!". Display name: the address.

Because the record adopts the Thread behavior, every change of the address and every archive or unarchive produces a tracked log entry, which is the audit trail of the opt-out.

---

## 13. Gateway Allowed Sender

Gateway Allowed Sender (`mail.gateway.allowed`, table `mail_gateway_allowed`).

### Purpose

One trusted sender address exempted from the incoming-loop quota. Automated senders that legitimately produce many messages are listed here so that the loop detector does not bounce them.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `email` | Text | Required. The address as typed. |
| `email_normalized` | Text, computed from the address, stored, indexed | The normalized form used for matching. |

---

## 14. Canned Response

Canned Response (`mail.canned.response`, table `mail_canned_response`).

### Purpose

A shortcut that expands into a longer text while composing. Typing two colons followed by the shortcut inserts the substitution, which the author may then edit before sending.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `source` | Text | Required, indexed for partial-word search. The shortcut. |
| `substitution` | Long text | Required. The text inserted in place of the shortcut. |
| `last_used` | Date and time | The moment the response was last inserted, used to order proposals. |
| `group_ids` | Multiple links to Group | The groups allowed to use the response. The selection is restricted to groups the acting user belongs to, so a user cannot share a response with a group they are not in. |
| `is_shared` | Boolean, computed, stored | True when the response is visible to users other than its creator. |
| `is_editable` | Boolean, computed, not stored | True when the acting user may edit the response. |

Ordering: newest first. Display name: the shortcut.

---

## 15. Role

Role (`res.role`, table `res_role`).

### Purpose

A named set of users that can be mentioned as a group inside a conversation. Mentioning the role notifies every user attached to it.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. Unique — "A role with the same name already exists." |
| `user_ids` | Multiple links to User | The members. |

---

## 16. Thread satellite behaviors

Four abstract behaviors extend the Thread behavior. They are adopted by concrete models and add fields to them.

### 16.1 Blacklist behavior

Blacklist behavior (`mail.thread.blacklist`, abstract). Adopts the Thread behavior.

Adopted by models whose records hold a mailable address and that must honour the opt-out. It normalizes the address of the record and exposes the blacklist status and the bounce counter.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `email_normalized` | Text, computed from the primary address field, stored, computed with elevated rights | The normalized form of the address: lower case, no display name, `local-part@domain`. Used for every address comparison. |
| `is_blacklisted` | Boolean, computed, not stored, computed with elevated rights | True when the normalized address has an active Blacklist Entry. Readable only by internal users. Searchable. |
| `message_bounce` | Integer | Default 0. The number of times a message to this address has bounced. |

The behavior also fixes the primary address field name to `email` unless the adopting model overrides it.

### 16.2 Carbon-copy behavior

Carbon-copy behavior (`mail.thread.cc`, abstract). Adopts the Thread behavior.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `email_cc` | Text | The accumulated carbon-copy list of the conversation, taken from incoming messages and reused on outgoing ones. |

### 16.3 Main attachment behavior

Main attachment behavior (`mail.thread.main.attachment`, abstract). Adopts the Thread behavior.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `message_main_attachment_id` | Link to Attachment | Not copied, indexed when not empty. The attachment shown in the document preview pane. |

### 16.4 Duration tracking behavior

Duration tracking behavior (`mail.tracking.duration.mixin`, abstract). Adopts the Thread behavior.

Adopted by models that have a pipeline-like link field (a stage, a status) and want to know how long each record spent in each value.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `duration_tracking` | Structured value, computed, not stored | A map from the identifier of each value the link field has taken to the number of seconds the record spent on that value. |
| `rotting_days` | Integer, computed, not stored | The number of days since the record last moved. |
| `is_rotting` | Boolean, computed, not stored | True when the record has been idle longer than the threshold configured for its current value. Searchable. |

The exact arithmetic of both is given in [calculations.md](calculations.md).

---

## 17. Activity

Activity (`mail.activity`, table `mail_activity`).

### Purpose

One thing to do, attached to one record (or free-standing), with a due date and an assignee. Completing an activity archives it and posts a message on the record; if the type says so, it also creates the next activity in the chain.

### Lifecycle

1. **Created** manually from the record, by a plan, by an automated rule, or by another domain's business code.
2. **Live**: the derived state is "overdue", "today" or "planned" depending on the due date compared with today in the assignee's time zone.
3. **Completed**: the activity is archived (its active flag becomes false), its completion date is stamped, a message is posted on the record with the activities subtype, its attachments are moved onto that message, and if the type chains by trigger the successor activity is created.
4. **Cancelled**: the activity is deleted outright. No message is posted.
5. **Purged**: a maintenance routine deletes very old overdue activities when the operator has configured a retention period.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `res_model_id` | Link to Model definition | Indexed. The model of the related record. On deletion of the model definition, the row is deleted. Not required: a free-standing activity has none. |
| `res_model` | Text, related to the model definition, stored, read-only, precomputed | The transport name, denormalized and indexed so that per-model queries do not join. |
| `res_id` | Record reference | Indexed. The related record. |
| `res_name` | Text, computed from the model and record, stored, read-only, computed with elevated rights | The display name of the related record at the time of computation. |
| `activity_type_id` | Link to Activity Type | The category. Default: the first type, by ordering, whose model is the related model or empty. On deletion of the type, the deletion is refused while activities reference it — but the type deletion path reassigns them to the generic "to-do" type first. Selection restricted to types whose model is empty or equals the related model. |
| `activity_category` | Selection, related to the type | Read-only. The action class of the type. |
| `activity_decoration` | Selection, related to the type | Read-only. The colour class of the type. |
| `icon` | Text, related to the type | Read-only. |
| `summary` | Text | The one-line title. Defaults from the type's default summary when the type is chosen. |
| `note` | Rich text | The detail. Style attributes sanitized. Defaults from the type's default note. |
| `date_deadline` | Date | Required, indexed. Default: today in the acting user's time zone. |
| `date_done` | Date, computed from the active flag, stored | Empty while the activity is live. Stamped with the current moment the first time the activity is archived; a later re-archive does not overwrite it. |
| `feedback` | Long text | The free text the person typed when completing the activity. |
| `automated` | Boolean, read-only | True when the activity was created by the system rather than typed by a person. The helper that schedules activities from business code sets it; the manual form does not. |
| `attachment_ids` | Multiple links to Attachment | Files attached to the activity. Access checks bypassed at query level. On completion they are moved onto the posted message. |
| `user_id` | Link to User | Indexed. The assignee. On deletion of the user, the row is deleted. Not required, but see the constraint below. |
| `user_tz` | Selection, related to the assignee's time zone, stored | Denormalized so the state can be computed in one database query. |
| `state` | Selection, computed, not stored | One of "overdue", "today", "planned", "done". See [state-machines.md](state-machines.md). |
| `recommended_activity_type_id` | Link to Activity Type | While scheduling the successor of a completed activity, the type the person picked among the suggestions. Selecting it copies it into the real type. |
| `previous_activity_type_id` | Link to Activity Type | Read-only. The type of the activity this one succeeds. |
| `has_recommended_activities` | Boolean, computed, not stored | True when the previous type has at least one suggested successor. |
| `mail_template_ids` | Multiple links to Template, related to the type | Read-only. The templates offered as one-click actions on the activity. |
| `chaining_type` | Selection, related to the type | Read-only. Either "suggest" or "trigger". |
| `can_write` | Boolean, computed, not stored | True when the acting user may modify the activity. Used to hide buttons. |
| `active` | Boolean | Default true. False means completed. |

Ordering: by due date ascending then identifier ascending. Display name: the summary, falling back to the display name of the type.

### Constraints

| Rule | Condition | Message |
|---|---|---|
| Record required with a model | Either both the model and a non-zero record identifier are set, or neither is. | "Activities have to be linked to records with a not null res_id." |
| Assignee required without a model | An activity with no model must have an assignee. | "Activities must be assigned if not attached to a document." |

### Access policy

Like messages, activities layer their own policy:

- **Read**: allowed by the ordinary rules **and** (the activity is assigned to the acting user **or** the acting user may read the related record).
- **Create**: allowed by the ordinary rules **and** the acting user has, on the related record, the permission named by that model's post-access attribute (defaulting to write).
- **Modify** and **Delete**: allowed by the ordinary rules **or** the acting user has that same permission on the related record.
- A free-standing activity (no model) is accessible only to its assignee.

Searching applies the same filter: rows assigned to the acting user always pass; the others are kept only if the related record is readable.

### Side effects at creation

1. Determine which assignee contacts the acting user may read, so that the notification can be sent with ordinary rights where possible and with elevated rights otherwise.
2. For every created activity whose assignee is **not** the acting user, and unless the quick-update switch is on, send the assignment notification (see below).
3. Subscribe each assignee's contact as a Follower of the related record, batched by model and by user.
4. For every created activity that is live, has an assignee and is due today or earlier, broadcast a counter increment to that user.

### The assignment notification

For each activity, rendered in the **assignee's** language:

- body: the shipped assignment template, which shows the activity type, the summary, the note and a button leading to the record;
- subject: `"<record name>: <summary>" assigned to you`, where the summary falls back to the type name and then to the empty string;
- subtitles: `Activity: <type name>` (falling back to "Todo") and `Deadline: <due date formatted in the recipient's date format>`;
- layout: the standard notification layout;
- delivered through the notification operation, so it lands in the inbox or in the mailbox according to the recipient's preference.

### Side effects at modification

- When the assignee changes and the new assignee is not the acting user, and the quick-update switch is off, the new assignee is notified as above.
- When the assignee changes, the new assignee's contact is subscribed to the related records.
- When the due date, the active flag or the assignee changes, the per-user count of live activities due today or earlier is recomputed before and after, and the difference is broadcast to each affected user as an increment or a decrement.

### Side effects at deletion

The same counter decrement is broadcast for every deleted activity that was live, assigned and due today or earlier.

### Completion

Completion is described step by step in [workflows.md](workflows.md). In summary, for each activity:

1. If the type chains by trigger, the values of the successor are prepared **before** anything is destroyed, with the current due date placed in the context so a "after previous activity deadline" delay can use it.
2. If the related record still exists, post a message on it from the shipped "activity done" template, authored by the acting user, carrying the activity type and the activities subtype. The rendering receives the activity, the feedback text and a flag saying whether the assignee differs from the acting user.
3. Move the attachments of the activity onto that message.
4. If the related record no longer exists, no message is posted, the attachments are deleted and the activity is deleted rather than archived.
5. Create the successor activities, if any.
6. Archive the remaining activities and store the feedback on them.

---

## 18. Activity Type

Activity Type (`mail.activity.type`, table `mail_activity_type`).

### Purpose

The category of an activity. It supplies the default summary, the default note, the default assignee, the default delay, the chaining rule, the visual decoration, the action class and the list of one-click templates.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. |
| `summary` | Text, translatable | The default summary copied onto a new activity of this type. |
| `sequence` | Integer | Default 10. Ordering key; also decides which type is the default for a model (the first one). |
| `active` | Boolean | Default true. |
| `create_uid` | Link to User | Indexed. |
| `delay_count` | Integer | Default 0. The number of delay units used to compute the due date. |
| `delay_unit` | Selection | Required, default "days". One of `days`, `weeks`, `months`. |
| `delay_label` | Text, computed, not stored | The delay rendered as "<count> <unit label>". |
| `delay_from` | Selection | Required, default `previous_activity`. `current_date` means "after previous activity completion date" (count from today); `previous_activity` means "after previous activity deadline" (count from the due date of the activity being completed, when one is known). |
| `icon` | Text | The icon name. |
| `decoration_type` | Selection | Empty, `warning` ("Alert") or `danger` ("Error"). Colours the activity and drives the record-level exception indicator. |
| `res_model` | Selection over thread-enabled, non-transient models | Empty means the type is generic. |
| `triggered_next_type_id` | Link to Activity Type, computed, stored, editable | The successor created automatically on completion. Cleared automatically when the chaining rule is "suggest". Setting it switches the chaining rule to "trigger"; clearing it switches back to "suggest". Deletion is refused while referenced. Selection restricted to types whose model is empty or equal. |
| `chaining_type` | Selection | Required, default `suggest`. `suggest` proposes successors; `trigger` creates one automatically. |
| `suggested_next_type_ids` | Multiple links to Activity Type, computed, stored, editable | The successors proposed on completion. Cleared automatically when the chaining rule is "trigger". Setting any switches the chaining rule to "suggest". |
| `previous_type_ids` | Multiple links to Activity Type | The reverse side of the suggestion relation: the types that propose this one. |
| `category` | Selection | Default `default`. `default` = None; `upload_file` = "Upload Document" (the activity is completed automatically when a document is attached); `phonecall` = "Phonecall". Other capabilities add more values, for example a meeting category that opens the calendar. |
| `mail_template_ids` | Multiple links to Template | Templates offered as one-click "send" buttons on activities of this type. When the model of the type changes, templates of another model are dropped. |
| `default_user_id` | Link to User | The default assignee. |
| `default_note` | Rich text, translatable | The default note. |
| `initial_res_model` | Selection, computed, not stored | The model at the start of an edit, used only to warn the user that the model changed. |
| `res_model_change` | Boolean, not stored | Set while editing when the model changed. |

Ordering: by sequence then identifier. Display name: the name.

### Protected shipped types

Five types are shipped and referenced by external identifier. Two properties are protected:

| External identifier | Model | May be deleted |
|---|---|---|
| `mail.mail_activity_data_call` | generic | no |
| `mail.mail_activity_data_meeting` | generic | no |
| `mail.mail_activity_data_todo` | generic | no |
| `mail.mail_activity_data_upload_document` | generic | yes |
| `mail.mail_activity_data_warning` | generic | yes |

Changing the model of any of the five is refused with "You cannot modify <names> target model as they are are required in various apps."

Deleting one of the three non-deletable ones is refused with "You cannot delete <names> as it is required in various apps."

Archiving the generic to-do type is refused with "The 'To-Do' activity type is used to create reminders from the top bar menu and the command palette. Consequently, it cannot be archived or deleted."

Deleting any other type first reassigns every activity of that type to the generic to-do type, then deletes.

### Deadline computation

```formula
due_date = base_date + delay_count × one_unit(delay_unit)
```

where `base_date` is:

- the due date of the activity being completed, when the delay origin is "after previous activity deadline" **and** such a previous due date is supplied by the caller;
- today in the acting user's time zone otherwise.

`one_unit` is one calendar day, one calendar week or one calendar month. Month arithmetic clamps to the last day of the target month. Worked examples are in [calculations.md](calculations.md).

---

## 19. Activity Plan and Activity Plan Template

### 19.1 Activity Plan

Activity Plan (`mail.activity.plan`, table `mail_activity_plan`).

A named set of activities launched together on one or several records of one model.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. |
| `company_id` | Link to Company | Default: the active company. Empty means the plan is shared by all companies. |
| `template_ids` | Sub-records: Activity Plan Template | The lines. Copied when the plan is duplicated. |
| `active` | Boolean | Default true. |
| `res_model_id` | Link to Model definition, computed from the model name, stored, editable, required, precomputed, computed with elevated rights | The model the plan applies to. On deletion of the model definition, the plan is deleted. |
| `res_model` | Selection over activity-enabled, non-transient models | Required. |
| `steps_count` | Integer, computed, not stored | The number of lines. |
| `has_user_on_demand` | Boolean, computed, not stored | True when at least one line asks for the assignee at launch. |

Ordering: newest first. Duplicating a plan appends " (copy)" to the name unless a name is supplied.

Constraint: changing the model of a plan re-runs the compatibility check of all its lines.

### 19.2 Activity Plan Template

Activity Plan Template (`mail.activity.plan.template`, table `mail_activity_plan_template`).

One line of a plan.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `plan_id` | Link to Activity Plan | Required, indexed. On deletion, the row is deleted. |
| `res_model` | Selection, related to the plan | — |
| `company_id` | Link to Company, related to the plan | — |
| `sequence` | Integer | Default 10. |
| `activity_type_id` | Link to Activity Type | Required, default the generic to-do type. Deletion of the type is refused while referenced. Selection restricted to generic types and to types of the plan's model. |
| `delay_count` | Integer | Default 0. **The delay of the type is ignored**; this one is used. |
| `delay_unit` | Selection | Required, default "days". Days, weeks or months. |
| `delay_from` | Selection | Required, default `before_plan_date`. Either "Before Plan Date" or "After Plan Date". |
| `icon` | Text, related to the type | Read-only. |
| `summary` | Text, computed from the type's default summary, stored, editable | — |
| `responsible_type` | Selection, computed from the type, stored, editable | Required, default `on_demand`. `on_demand` = "Ask at launch"; `other` = "Default user". Computed: "other" when the type has a default user, "on demand" otherwise. |
| `responsible_id` | Link to User, computed, stored, editable | The fixed assignee. Computed: the type's default user, cleared when the assignment mode is not "other". Checked against the company of the plan. |
| `note` | Rich text, computed from the type's default note, stored, editable | — |
| `next_activity_ids` | Multiple links to Activity Type, computed from the type, stored, editable | Informational: the successors the type would chain to. Recomputed only when the type changes, so later changes to the type do not silently alter the plan. |

Ordering: by sequence then identifier. Display name: the summary.

Constraints:

| Rule | Message |
|---|---|
| The type's model, when set, must equal the plan's model. | "The activity type "<type name>" is not compatible with the plan "<plan name>" because it is limited to the model "<type model>"." |
| When the assignment mode is "Default user", an assignee must be set. | "When selecting "Default user" assignment, you must specify a responsible." |

### Line deadline

```formula
line_due_date = plan_date − delay_count × one_unit(delay_unit)      when trigger = before plan date
line_due_date = plan_date + delay_count × one_unit(delay_unit)      when trigger = after plan date
```

`plan_date` defaults to today in the acting user's time zone.

---

## 20. Activity behavior (abstract)

Activity behavior (`mail.activity.mixin`, abstract).

Adopted by a model to give its records an activity list and the derived indicators used by list, kanban and the dedicated activity view.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `activity_ids` | Sub-records: Activity | The **live** activities of the record (archived ones are filtered out by the default active filter). Access checks bypassed at query level. Readable only by internal users. |
| `activity_state` | Selection, computed, not stored | The worst state among the live activities: "overdue" wins over "today", which wins over "planned"; empty when there is none. Readable only by internal users. Searchable through a dedicated aggregate query that computes, per record, the minimum of the sign of (due date − today in the assignee's time zone) and maps −1, 0, 1 to overdue, today, planned. |
| `activity_user_id` | Link to User, computed, not stored, read-only | The assignee of the **first** live activity in due-date order. Searchable. Readable only by internal users. |
| `activity_type_id` | Link to Activity Type, related to the activity list, writable | The type of the first live activity. Writable, so the interface can retype it. Searchable. |
| `activity_type_icon` | Text, related to the activity list | — |
| `activity_date_deadline` | Date, computed, not stored, read-only | The due date of the first live activity. Searchable; searching for an empty value also matches records with no activity at all. |
| `my_activity_date_deadline` | Date, computed, not stored, read-only, per user | The due date of the first live activity assigned to the acting user. Searchable. |
| `activity_summary` | Text, related to the activity list, writable | The summary of the first live activity. Searchable. |
| `activity_exception_decoration` | Selection, computed, not stored | The decoration of the "worst" activity type present on the record: an "Error" type wins immediately; otherwise the last "Alert" type found. Searchable. |
| `activity_exception_icon` | Text, computed, not stored | The icon of that same type. |

Grouping a list by the activity state is supported through the same aggregate query, so the grouping is done in the database rather than per record.

### Helper operations

The behavior exposes a small set of operations used by every other domain:

| Operation | Effect |
|---|---|
| schedule | Create one activity per record of the set, with a type given by external identifier or by identifier, a due date defaulting to today in the acting user's time zone, a summary and note defaulting to the type's, the automated flag set, and the type's default user as assignee when none is given. If the named type does not exist or belongs to another model, fall back to the model's default type. |
| schedule with a rendered note | The same, with the note produced by rendering a template for each record. |
| search | Find the activities of the set whose type is among a list of external identifiers, optionally restricted to one assignee, optionally restricted to automated ones (the default). |
| reschedule | Find them and change their due date and/or assignee. |
| complete | Find them and complete them with a feedback text. |
| remove | Find them and delete them. |
| send a template | Post a message on the record from a template, with the discussion subtype. |

All of them do nothing at all when the "skip activity automation" switch is on.

---

## 21. Outgoing Mail

Outgoing Mail (`mail.mail`, table `mail_mail`). Delegates to Message: every Outgoing Mail carries a mandatory link to a Message and exposes that message's fields as if they were its own.

### Purpose

One electronic mail waiting to be handed to a relay. It exists because sending is asynchronous, batched, retriable and subject to per-relay throttling.

### Field table (own fields)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `mail_message_id` | Link to Message | Required, indexed. The delegation target: reading any Message field through an Outgoing Mail reads it from here. On deletion of the message, the row is deleted. Access checks bypassed at query level. |
| `mail_message_id_int` | Integer, computed with elevated rights, not stored | The plain identifier of the message, exposed for callers that may not read the message. |
| `message_type` | Selection, delegated | Default overridden to "outgoing email". |
| `body_html` | Long text | The rendered body actually sent. Distinct from the message body, which is what the conversation shows. |
| `body_content` | Rich text, computed from the body, searchable | A sanitized read-only view of the body for the interface. |
| `references` | Long text, read-only | The chain of ancestor message identifiers placed in the references header. |
| `headers` | Long text | Extra headers as a serialized map. Not copied. Malformed content is logged and ignored rather than aborting the queue. |
| `restricted_attachment_count` | Integer, computed, not stored | How many of the attachments the acting user may not read. |
| `unrestricted_attachment_ids` | Multiple links to Attachment, computed, not stored, writable | The attachments the acting user may read. Writing it keeps the unreadable ones and replaces the readable ones. |
| `is_notification` | Boolean | True when the mail was created to notify recipients of an existing message. Set automatically at creation when a message link is supplied. Decides whether deleting the mail also deletes the message. |
| `email_to` | Long text | Free recipient addresses. |
| `email_cc` | Text | Free carbon-copy addresses. |
| `recipient_ids` | Multiple links to Contact | Recipients that are contacts. Archived contacts remain visible. |
| `state` | Selection, read-only, not copied | Default `outgoing`. See [state-machines.md](state-machines.md). |
| `failure_type` | Selection | Same vocabulary as the Notification failure type, minus the bounce value. |
| `failure_reason` | Long text, read-only, not copied | The exception text. |
| `auto_delete` | Boolean | When true the mail is deleted after a successful send (and after a send that failed only because the address was missing or invalid). |
| `scheduled_date` | Date and time | When set, the queue skips the mail until that moment. Parsed leniently at creation and modification (see below). |
| `fetchmail_server_id` | Link to Incoming Mail Server, read-only, indexed when not empty | Set when the mail originates from a polled mailbox. |

Ordering: newest first. Display name: the subject (delegated).

### Scheduled date parsing

A supplied scheduled date may be a date, a date and time, or a string. The rule is:

1. A date and time is taken as is.
2. A plain date is taken at midnight.
3. A string is parsed with the year-first preference, so that an ambiguous three-number date is read as year, month, day. A string that cannot be parsed yields "no schedule".
4. The microseconds are dropped.
5. If the result carries no time-zone information it is understood as coordinated universal time; otherwise it is converted to coordinated universal time.
6. The stored value is the naive coordinated-universal-time form.

An empty string is stored as "no schedule" rather than failing.

### Constraint

An Outgoing Mail may not name a relay that the creator of its message is not allowed to use — "You may not create a message using another user's mail server."

The allowed set is computed as follows: if the message has several creators (a batch) or the operator has disabled personal relays through a system parameter, only relays **without** an owner are allowed; otherwise relays without an owner plus the relays owned by the creator of the message.

### Deletion

Deleting an Outgoing Mail also deletes its Message when the mail is **not** a notification mail, because in that case the message exists only to carry the mail.

---

## 22. Alias

Alias (`mail.alias`, table `mail_alias`).

### Purpose

One inbound address. When a received electronic mail is addressed to it, the router uses the alias to decide which model to create or update, with which default values, on behalf of which user, and under which security policy.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `alias_name` | Text | The local part, for example `jobs`. Not copied. Sanitized at creation and modification (see below). An empty value means the alias is inactive. |
| `alias_full_name` | Text, computed, stored, indexed when not empty | `<local part>@<domain name>` when both are set; the local part alone when there is no domain; empty when there is no local part. |
| `alias_domain_id` | Link to Alias Domain | Default: the alias domain of the active company. On deletion of the domain, the deletion is refused while aliases reference it. |
| `alias_domain` | Text, related to the domain name | — |
| `alias_model_id` | Link to Model definition | Required. The model to create or update. On deletion of the model definition, the alias is deleted. The selection is restricted to models that have a message list, that is, thread-enabled models. |
| `alias_defaults` | Long text | Required, default an empty map. A literal mapping evaluated to provide default field values when a record is created from this alias. |
| `alias_force_thread_id` | Integer | When set, **all** incoming messages are attached to that record, and creation of new records through this alias is disabled entirely. |
| `alias_parent_model_id` | Link to Model definition | The model of the record that *owns* the alias, which is not necessarily the model the alias creates. |
| `alias_parent_thread_id` | Integer | The identifier of that owner record. |
| `alias_contact` | Selection | Required, default `everyone`. The security policy: `everyone` (anyone may post), `partners` (only a sender matching a known contact), `followers` (only a follower of the target record, or of the owner record when the alias creates records). |
| `alias_incoming_local` | Boolean | Default false. When true the alias matches on the **local part only**, so any domain is accepted (subject to the allowed-domain list). When false the full address must match. |
| `alias_bounced_content` | Rich text, translatable | When set, this content replaces the default rejection body sent to an unauthorized sender. |
| `alias_status` | Selection, computed, stored | `not_tested`, `valid` or `invalid`. Reset to "not tested" whenever the security policy, the default values or the target model change. Set to "valid" the first time a record is successfully created. Set to "invalid" when a configuration error made an incoming message bounce. |

Ordering: by target model then local part. Display name: `<local part>@<domain>`; the local part alone when there is no domain; "Inactive Alias" when there is no local part.

Uniqueness: the pair (local part, domain) is unique, treating "no domain" as a distinct value.

### Sanitizing a local part

Applied at creation and at modification, and reused for the alias-domain local parts:

1. Trim.
2. If the value is being treated as a full address, remember the part after the first at-sign, lower-cased.
3. Remove accents, lower-case, and keep only what precedes the first at-sign.
4. Remove leading dots, trailing dots and any dot immediately followed by another dot.
5. Replace every run of characters outside the allowed set (letters, digits, underscore and the punctuation `! # $ % & ' * + - / = ? ^ _ \` { | } ~ .`) by a single hyphen.
6. Encode to plain ASCII, replacing anything that does not fit.
7. If nothing is left, the result is "no name".
8. If a full address was requested and a right part was remembered, re-join them with an at-sign.

### Constraints

| Rule | Message |
|---|---|
| The local part must be a plain ASCII dot-atom. | "You cannot use anything else than unaccented latin characters in the alias address <local part>." |
| The default values must be a literal mapping. | "Invalid expression, it must be a literal python dictionary definition e.g. "{'field': 'value'}"" |
| The local part must not equal the bounce or catch-all local part of its own domain. | "Aliases <names> is already used as bounce or catchall address. Please choose another alias." |
| When the owner record has a company, that company's alias domain must match the alias's domain, provided the alias domain is bound to companies. | "We could not create alias <alias>, because domain <domain> belongs to company <companies> while the owner document belongs to company <company>." |
| The same rule for the forced target record. | "We could not create alias <alias> because domain <domain> belongs to company <companies> while the target document belongs to company <company>." |
| The pair (local part, domain) must be free. | See the two long messages in [business-rules.md](business-rules.md); they name the conflicting alias, the model it serves and, when applicable, the owner record. |
| The same local part must not be requested twice in one batch. | "Email aliases <name> cannot be used on several records at the same time. Please update records one by one." |

---

## 23. Alias Domain

Alias Domain (`mail.alias.domain`, table `mail_alias_domain`).

### Purpose

One electronic mail domain owned by the installation, with the three special local parts that make the gateway work: the catch-all that receives replies, the bounce address that receives delivery failures, and the default sender used when the configured sender does not match a relay's filter.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. The domain, for example `example.com`. Sanitized like a local part. |
| `company_ids` | Sub-records: Company | The companies that use this domain as their default. |
| `sequence` | Integer | Default 10. Ordering key; the first domain by ordering is the global fallback. |
| `bounce_alias` | Text | Required, default `bounce`. The local part of the return path. |
| `bounce_email` | Text, computed, not stored | `<bounce local part>@<domain>`; empty when there is no bounce local part. |
| `catchall_alias` | Text | Required, default `catchall`. The local part of the reply address. |
| `catchall_email` | Text, computed, not stored | `<catch-all local part>@<domain>`; empty when there is no catch-all local part. |
| `default_from` | Text | Default `notifications`. Either a local part or a complete address. |
| `default_from_email` | Text, computed, not stored | The value itself when it already contains an at-sign; otherwise `<value>@<domain>`. |

Ordering: by sequence ascending then identifier ascending.

Uniqueness: (bounce local part, domain) and (catch-all local part, domain).

### Constraints

| Rule | Message |
|---|---|
| The domain name must not be empty. | "You cannot assign an empty domain name." |
| The domain name must be a plain ASCII dot-atom. | "You cannot use anything else than unaccented latin characters in the domain name <name>." |
| Two domains with the same name must not share a bounce local part. | "Bounce alias <address> is already used for another domain with same name. Use another bounce or simply use the other alias domain." |
| Two domains with the same name must not share a catch-all local part. | "Catchall alias <address> is already used for another domain with same name. Use another catchall or simply use the other alias domain." |
| No Alias may already occupy the bounce or catch-all address. | "Bounce/Catchall '<address>' is already used by <document>. Choose another alias or change it on the other document." when an owner or target record can be named, otherwise "Bounce/Catchall '<address>' is already used. Choose another alias or change it on the linked model." |
| The default sender address must not fall inside the filter of a personal relay. | "A personal mail server is using that address, you can not use it." |

### First-domain initialisation

When the **first** alias domain of the installation is created (detected because the total count equals the number just created), it is assigned to every company that has none — including archived companies — and to every Alias that has none.

### Finding aliases in a list of addresses

A shared helper answers "which of these addresses belong to the installation". Given a list of normalized addresses:

1. Keep only entries containing an at-sign.
2. Build the set of all bounce, catch-all and default-sender addresses of all alias domains.
3. Read the allowed-domain list from the system parameters; if it is non-empty, add every alias-domain name to it. Local-part matching is then restricted to addresses whose domain is in that list.
4. Search the Alias table for rows whose full address is in the list, or whose local part is in the list of candidate local parts **and** that are marked local-part based.
5. Add the full addresses of the matched non-local aliases to the set.
6. Return every input address that is in the set, plus every input address whose local part matches a local-part-based alias and whose domain passes the allowed-domain check.

This helper is used to avoid creating contacts for the installation's own addresses, to filter the recipient list of an incoming message, and to keep alias addresses out of suggested recipients.

---

## 24. Alias behaviors (abstract)

### 24.1 Optional alias behavior

Optional alias behavior (`mail.alias.mixin.optional`, abstract).

A model adopting it may have an alias but does not need one; the alias record is created on demand the first time a local part is set.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `alias_id` | Link to Alias | Not required, not copied. On deletion of the alias, the deletion is refused while records reference it. |
| `alias_name` | Text, related to the alias, writable | Setting it on a record with no alias creates the alias. |
| `alias_domain_id` | Link to Alias Domain, related to the alias, writable | — |
| `alias_domain` | Text, related to the alias | — |
| `alias_defaults` | Long text, related to the alias | — |
| `alias_email` | Text, computed, not stored | The full address. Searchable. |

### 24.2 Required alias behavior

Required alias behavior (`mail.alias.mixin`, abstract). Adopts the optional behavior and, in addition, delegates to Alias.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `alias_id` | Link to Alias | **Required.** Created together with the record. |
| `alias_name`, `alias_defaults` | delegated | Exposed as if they were fields of the record. |

A model adopting this behavior always has exactly one alias, created with the record and deleted with it.

---

## 25. Incoming Mail Server

Incoming Mail Server (`fetchmail.server`, table `fetchmail_server`).

### Purpose

One mailbox that the platform polls for incoming messages.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. |
| `active` | Boolean | Default true. |
| `state` | Selection, indexed, read-only, not copied | Default `draft` ("Not Confirmed"); `done` ("Confirmed") after a successful connection test. |
| `server` | Text | Host name or network address. |
| `port` | Integer | — |
| `server_type` | Selection, required, indexed | Default `imap`. Values: `imap` (internet message access protocol server), `pop` (post office protocol server), `local` (a local mailbox file). |
| `server_type_info` | Long text, computed, not stored | Human-readable guidance for the selected type. |
| `is_ssl` | Boolean | Connections are encrypted on a dedicated port (the industry-standard defaults are port 993 for the encrypted access protocol and port 995 for the encrypted post office protocol). |
| `attach` | Boolean | Default true. Keep attachments; when false, incoming messages are stripped of attachments before processing. |
| `original` | Boolean | Keep a full copy of each original message as an attachment. Roughly doubles storage. |
| `date` | Date and time, read-only | The moment of the last successful fetch. |
| `error_date` | Date and time, read-only | The moment of the last failure; cleared on success. |
| `error_message` | Long text, read-only | The last failure text. |
| `user` | Text | The account name. |
| `password` | Text | The account secret. |
| `object_id` | Link to Model definition | The fallback model: each incoming message that matches no alias and no reply is processed as a record of this model. |
| `priority` | Integer | Default 5. Lower values are polled first. |
| `message_ids` | Sub-records: Outgoing Mail, read-only | The messages that came from this server. |
| `configuration` | Long text, read-only | Generated instructions for configuring the external side. |
| `script` | Text, read-only | The path of the shipped gateway script used when the external mail transfer agent pipes messages in directly. |

Ordering: by priority.

---

## 26. Outgoing Mail Server — fields added here

Outgoing Mail Server (`ir.mail_server`, table `ir_mail_server`). The base record belongs to the platform layer; this domain adds the ownership restriction that allows a user to send through their own account.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `mail_template_ids` | Sub-records: Template, read-only | The templates that prefer this relay. |
| `owner_user_id` | Link to User, not copied | When set, the relay is *personal*: only that user's messages may use it. Unique across relays. |
| `owner_limit_time` | Date and time, not copied | The minute currently being counted for the throttle. |
| `owner_limit_count` | Integer, not copied | How many recipients have already been sent in that minute. |

Constraint: the owner is unique — "owner_user_id must be unique".

The throttle is described in [calculations.md](calculations.md) and [workflows.md](workflows.md).

---

## 27. Template

Template (`mail.template`, table `mail_template`). Adopts the render behavior and the template-reset behavior.

### Purpose

A reusable message with placeholders. A template supplies the subject, the body, the sender, the recipients, the reply address, the attachments, the reports to generate, the layout and the schedule.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | — |
| `description` | Long text, translatable | Internal description of when to use the template. |
| `active` | Boolean | Default true. |
| `template_category` | Selection, computed, not stored, searchable | `base_template` (shipped and unmodified), `hidden_template` (shipped and meant to stay out of the picker), `custom_template` (created or modified by a user). |
| `model_id` | Link to Model definition | The model the template renders against. On deletion of the model definition, the template is deleted. Selection restricted to non-abstract models. |
| `model` | Text, related, stored, read-only, indexed | The transport name. |
| `subject` | Text, translatable | Placeholders allowed. |
| `email_from` | Text | The sender. Placeholders allowed. When empty the sender falls back to the author's alias address if configured, then to the author's own address. |
| `user_id` | Link to User | The owner. Restricted to non-portal users. A template with an owner is offered only to that owner. |
| `use_default_to` | Boolean | Default true. When true the recipients are computed from the record rather than taken from the three fields below. |
| `email_to` | Text | Comma-separated recipient addresses. Placeholders allowed. |
| `partner_to` | Text | Comma-separated recipient contact identifiers. Placeholders allowed. |
| `email_cc` | Text | Comma-separated carbon-copy addresses. Placeholders allowed. |
| `reply_to` | Text | The reply address to force. Only used when the reply is not threaded back into the record's conversation. |
| `body_html` | Rich text, translatable | Rendered through the structured-template engine with post-processing. Sanitized for outgoing electronic mail. |
| `attachment_ids` | Multiple links to Attachment | Static files attached to every message produced. Access checks bypassed at query level. |
| `report_template_ids` | Multiple links to Report definition | Reports generated per record and attached. Restricted to reports of the same model. |
| `email_layout_xmlid` | Text, not copied | The external identifier of the notification layout to wrap the body in. |
| `mail_server_id` | Link to Outgoing Mail Server, indexed when not empty | The preferred relay. When empty, the highest-priority matching relay is used. |
| `scheduled_date` | Text | A placeholder expression producing the moment at which the queue may send. |
| `auto_delete` | Boolean | Default true. Delete the outgoing mail after sending. |
| `ref_ir_act_window` | Link to Window action, read-only, not copied | The contextual action that makes the template available from the record's action menu. |
| `can_write` | Boolean, computed, not stored | Whether the acting user may edit the template. |
| `is_template_editor` | Boolean, computed, not stored | Whether the acting user belongs to the template-editor group. |
| `has_dynamic_reports` | Boolean, computed, not stored | Whether at least one report is attached. |
| `has_mail_server` | Boolean, computed, not stored | Whether a relay is configured anywhere. |

Ordering: by owner, then name, then identifier.

The template model is marked as **allowed to render unrestricted expressions**: a template body may contain arbitrary placeholder expressions, subject to the editor restriction described in [business-rules.md](business-rules.md).

---

## 28. Render behavior, Composer behavior, Template reset behavior

### 28.1 Render behavior

Render behavior (`mail.render.mixin`, abstract).

The contract for turning a text containing placeholders into a final text for one record.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `lang` | Text | An optional language code, usually itself a placeholder expression, used to pick the translation of the rendered fields. When empty, the language of the main recipient contact is used. |
| `render_model` | Text, computed, not stored | The model the rendering runs against. |

Class attribute: a flag saying whether the model is allowed to render unrestricted expressions. It is false by default and true for Template and for the text-message template.

Two engines exist:

| Engine | Syntax | Where used |
|---|---|---|
| inline placeholder | `{{ expression }}` inside plain text | subjects, text-message bodies, sender and recipient fields, the scheduled date |
| structured template | a markup template language evaluated against the record | rich-text bodies |

The rendering contract, the evaluation context, the safety restrictions and the post-processing are specified in [calculations.md](calculations.md).

### 28.2 Composer behavior

Composer behavior (`mail.composer.mixin`, abstract). Adopts the render behavior.

Used by every window that lets a person edit a subject and a body derived from a template.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `subject` | Text, computed from the template, stored, editable, computed without elevation | Reset to the template's subject when the template changes; cleared when the template is removed. |
| `body` | Rich text, computed from the template, stored, editable, computed without elevation | Same rule. Rendered through the structured-template engine with post-processing. Sanitized for outgoing electronic mail. |
| `body_has_template_value` | Boolean, computed, not stored | True when the body still equals the template's body, so the caller can tell an untouched body from an edited one. |
| `template_id` | Link to Template | Restricted to templates of the rendering model. |
| `lang` | Text, computed, stored, editable, precomputed, computed without elevation | — |
| `is_mail_template_editor` | Boolean, computed, not stored | Whether the acting user belongs to the template-editor group. |
| `can_edit_body` | Boolean, computed, not stored | Whether the acting user may edit the body. False for a non-editor when the body still contains template markup that could execute expressions. |

### 28.3 Template reset behavior

Template reset behavior (`template.reset.mixin`, abstract).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `template_fs` | Text, not copied | The shipped definition file the template came from. A template with this value can be restored to its original content; a template without it cannot. |

---

## 29. Composer

Composer (`mail.compose.message`, transient). Adopts the composer behavior.

### Purpose

The message composition window. It has two modes:

- **comment**: post on one record (or on a small batch), producing one Message per record and the notifications that follow;
- **mass mail**: send an electronic mail per record of a possibly large set, rendering the template per record, producing Outgoing Mails but no conversation entry unless asked.

Records are processed in batches of 50.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `subject` | Text, computed, stored, editable | From the template, else from the record. |
| `body` | Rich text, computed, stored, editable | From the template. Rendered with the structured engine and post-processed. |
| `parent_id` | Link to Message | The message being answered. On deletion, set to empty. |
| `template_id` | Link to Template | Restricted to templates of the target model that are either unowned or owned by the acting user. |
| `attachment_ids` | Multiple links to Attachment, computed, stored, editable | The files to attach. Access checks bypassed at query level. |
| `email_layout_xmlid` | Text, computed, stored, editable, not copied, computed without elevation | The notification layout. |
| `email_add_signature` | Boolean, computed, stored, editable | Whether to append the author's signature. |
| `email_from` | Text, computed, stored, editable, computed without elevation | The sender address. |
| `author_id` | Link to Contact, computed, stored, editable, computed without elevation | The author. |
| `composition_mode` | Selection | Default `comment`. Either "Post on a document" or "Email Mass Mailing". |
| `composition_batch` | Boolean, computed, not stored | True when more than one record is targeted. |
| `composition_comment_option` | Selection | `reply_all` or `forward`. Drives the default recipient list when answering. |
| `model` | Text, computed, stored, editable | The target model. |
| `model_is_thread` | Boolean, computed, not stored | Whether the target model has a conversation. |
| `res_ids` | Long text, computed, stored, editable | The serialized list of target record identifiers. |
| `res_domain` | Long text | An alternative to the list: a filter evaluated at send time. |
| `res_domain_user_id` | Link to User | The user whose rights are used to evaluate that filter. |
| `record_alias_domain_id` | Link to Alias Domain, computed, stored, editable | The alias domain of the target records. |
| `record_company_id` | Link to Company, computed, stored, editable | The company of the target records. |
| `message_type` | Selection | Required, default `comment`. One of automated targeted notification, comment, system notification. |
| `subtype_id` | Link to Message Subtype, computed, stored, editable | On deletion, set to empty. |
| `subtype_is_log` | Boolean, computed, not stored | True when the chosen subtype is the internal note. |
| `mail_activity_type_id` | Link to Activity Type | On deletion, set to empty. |
| `reply_to` | Text, computed, stored, editable, computed without elevation | Setting it bypasses the automatic threading of replies. |
| `reply_to_force_new` | Boolean, computed, stored, editable | Treat answers as new incoming messages rather than replies. |
| `reply_to_mode` | Selection, computed, with an inverse | `update` = "Store email and replies in the chatter of each record"; `new` = "Collect replies on a specific email address". Writing it sets the flag above. |
| `partner_ids` | Multiple links to Contact, computed, stored, editable | Additional recipients. |
| `partner_ids_all_have_email` | Boolean, computed, not stored | Warns when a chosen recipient has no address. |
| `notified_bcc_contains_share` | Boolean, computed, not stored | True when an external contact follows the document and will therefore be silently copied. |
| `auto_delete` | Boolean, computed, stored, editable, computed without elevation | Delete the outgoing mails after sending. |
| `auto_delete_keep_log` | Boolean, computed, stored, editable | In mass mode, keep a copy of the body even when the mails are deleted. |
| `force_send` | Boolean, computed, stored, editable | Send immediately instead of queueing. |
| `mail_server_id` | Link to Outgoing Mail Server, computed, stored, editable, computed without elevation | — |
| `notify_author` | Boolean, computed, stored, editable | Notify the author of their own message. |
| `notify_author_mention` | Boolean, computed, stored, editable | Notify the author when the author is an explicit recipient. |
| `notify_skip_followers` | Boolean, computed, stored, editable | Ignore the followers entirely and notify only the explicit recipients. |
| `scheduled_date` | Text, computed, stored, editable, computed without elevation | In comment mode, postpone the notifications; in mass mode, postpone the sending. Understood as coordinated universal time. |
| `use_exclusion_list` | Boolean | Default true, not copied. Honour the blacklist. |
| `template_name` | Text | The name to use when saving the current content as a new template. |

---

## 30. Wizards of the electronic mail area

### 30.1 Template Preview

Template Preview (`mail.template.preview`, transient). Renders one template against one chosen record and shows every rendered field: subject, sender, recipients, carbon copy, reply address, scheduled date, body and attachments. It also exposes an error message field so a broken placeholder is reported rather than raised. The list of previewed fields is fixed: attachments, body, subject, carbon copy, sender, recipient addresses, recipient contacts, reports, reply address and scheduled date.

### 30.2 Template Reset

Template Reset (`mail.template.reset`, transient). Holds a set of templates and restores each one that has a shipped definition.

### 30.3 Followers Edit

Followers Edit (`mail.followers.edit`, transient). Adds or removes a set of contacts as followers of a set of records of one model, optionally posting a message to notify them.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `res_model` | Text | Required. The followed model. |
| `res_ids` | Text | The followed record identifiers. |
| `operation` | Selection | Required, default `add`. Either "Add" or "Remove". |
| `partner_ids` | Multiple links to Contact | Required. |
| `message` | Rich text | The optional notification body. |
| `notify` | Boolean | Default false. |

### 30.4 Blacklist Removal

Blacklist Removal (`mail.blacklist.remove`, transient). Holds the address (read-only, required) and a reason, and archives the corresponding Blacklist Entry with the reason logged in its conversation.

### 30.5 Activity Schedule

Activity Schedule (`mail.activity.schedule`, transient). Launches either one activity or a whole plan on a set of records. Records are processed in batches of 500.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `res_model_id` | Link to Model definition, computed, stored, editable, precomputed | On deletion, the row is deleted. |
| `res_model` | Text, editable | — |
| `res_ids` | Long text, computed, stored, editable, precomputed | The serialized target identifiers. |
| `is_batch_mode` | Boolean, computed, not stored | True when more than one record is targeted. |
| `company_id` | Link to Company, computed, not stored | The company of the targets. |
| `error`, `has_error`, `warning`, `has_warning` | Rich text and Booleans, computed, not stored | The blocking and non-blocking problems detected before launching, rendered as a list. |
| `plan_available_ids` | Multiple links to Activity Plan, computed, stored, computed with elevated rights | The plans applicable to the target model and company. |
| `plan_id` | Link to Activity Plan, computed, stored, editable | Restricted to the available set. |
| `plan_has_user_on_demand` | Boolean, related to the plan | — |
| `plan_schedule_line_ids` | Sub-records: Activity Schedule Line, computed | The preview of what will be created. |
| `plan_on_demand_user_id` | Link to User | Default: the acting user. The assignee used for every line that asks at launch. |
| `plan_date` | Date, computed, stored, editable | The anchor date of the plan. |
| `activity_type_id` | Link to Activity Type, computed, stored, editable | Used in single-activity mode. On deletion, set to empty. |
| `activity_category`, `chaining_type` | Selections, related to the type, read-only | — |
| `date_deadline` | Date, computed, stored, editable | Used in single-activity mode. |
| `summary` | Text, computed, stored, editable | — |
| `note` | Rich text, computed, stored, editable | Style attributes sanitized. |
| `activity_user_id` | Link to User, computed, stored, editable | — |

### 30.6 Activity Schedule Line

Activity Schedule Line (`mail.activity.schedule.line`, transient). One preview row: the owning wizard, a description, a due date and a responsible user. Ordered by due date ascending then identifier ascending.

---

## 31. Push Device and Push Notification

### 31.1 Push Device

Push Device (`mail.push.device`, table `mail_push_device`).

One browser subscription endpoint belonging to one contact. It is what allows the platform to wake a closed browser tab.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `partner_id` | Link to Contact | Required, indexed. Default: the contact of the acting user. |
| `endpoint` | Text | Required. The address the push service exposes for this browser. Unique — "The endpoint must be unique !". |
| `keys` | Text | Required. The browser-supplied key material: the subscription public key and the authentication secret, kept as a structured value. The private half never leaves the browser. |
| `expiration_time` | Date and time | When the subscription stops being valid, as declared by the browser. |

A device that the push service reports as permanently unreachable is deleted.

### 31.2 Push Notification

Push Notification (`mail.push`, table `mail_push`).

One queued payload for one device, used when the number of devices to reach at once exceeds the direct-send threshold of five.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `mail_push_device_id` | Link to Push Device | Required. On deletion, the row is deleted. |
| `payload` | Long text | The serialized notification payload. |

Creating rows here also wakes the dedicated scheduled job immediately.

### 31.3 Payload shape and size limit

The payload is a structured value with a title and an options block holding the body, the icon, optional vibration pattern, an optional "requires interaction" flag, an optional grouping tag, an optional list of action buttons, and a data block carrying the model, the record identifier and the client action to open.

The encrypted payload must stay under the transport limit. The usable length is

```formula
maximum_payload_length = maximum_transport_payload − encryption_header_size − encryption_block_overhead
```

When the serialized payload exceeds it, only the body is shortened:

```formula
body_maximum_length = max( 0 , maximum_payload_length − serialized_payload_length + serialized_body_length )
```

The body is measured in its escaped serialized form, because a non-ASCII character is serialized as a six-character escape. Truncation must not cut an escape sequence in half: a trailing backslash is removed first, and if the result is still not decodable the cut is moved back to just before the offending escape marker.

---

## 32. Presence

Presence (`mail.presence`, table `mail_presence`). Adopts the bus sender behavior.

### Purpose

The online status of one user or one guest. It is deliberately a separate record rather than a field on the user, so that the very frequent status writes do not contend with ordinary user updates.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `user_id` | Link to User | On deletion, the row is deleted. |
| `guest_id` | Link to Guest | On deletion, the row is deleted. |
| `last_poll` | Date and time | Default: now. Updated every time the party contacts the server. |
| `last_presence` | Date and time | Default: now. Updated every time the party actually interacts (a keystroke, a click), not merely polls. |
| `status` | Selection | Default `offline`. One of `online`, `away`, `offline`. |

The model keeps no creation or modification stamps.

Uniqueness: at most one row per user; at most one row per guest.

Constraint: exactly one of user and guest must be set — "A mail presence must have a user or a guest."

The derived status shown to others combines this record with the user's manual override (see [calculations.md](calculations.md)).

---

## 33. Event Bus Entry and the bus sender behavior

### 33.1 Event Bus Entry

Event Bus Entry (`bus.bus`, table `bus_bus`).

One broadcast payload placed on one named channel. Clients subscribe to a set of channels and receive every entry whose identifier is greater than the last one they saw.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `channel` | Text | The serialized channel name. |
| `message` | Text | The serialized payload: a type and a body. |
| `create_date` | Date and time | Indexed, used by the garbage collector. |

A maintenance routine deletes entries older than a retention window.

Reading is by *polling*: given a list of channel names, the last identifier the client has seen and an optional list of identifiers to ignore, the entries strictly newer than that identifier on those channels are returned in identifier order.

Channel names are not free text. A channel is either a plain string (which must not be guessable by an attacker, because anyone who knows the name receives the traffic) or a record, in which case the name is derived from the record's model and identifier. The set of record types that may act as a channel is fixed and enumerated in [interfaces.md](interfaces.md).

### 33.2 Bus sender behavior

Bus sender behavior (`bus.listener.mixin`, abstract).

Adopted by every record that may be used as a broadcast target. It provides a single operation: send a payload of a given type to the channel derived from this record. Sending goes through a transaction hook so that nothing is broadcast if the transaction ultimately fails.

---

## 34. Channel

Channel (`discuss.channel`, table `discuss_channel`). Adopts the Thread behavior and the bus sender behavior.

### Purpose

A multi-party conversation. Unlike an ordinary thread, a channel has **members** rather than followers, and the member record carries the per-person state (unread marker, mute, pin, custom name).

### Behavior attributes

| Attribute | Value | Consequence |
|---|---|---|
| flat thread | false | Messages are not force-parented; genuine reply threads are possible. |
| post access | `read` | Reading the channel is enough to post in it. |
| maximum bounce limit | 10 | A member whose bounce counter reaches ten is removed from the channel. |

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. May be empty in practice for a group, in which case the display name is built from the members. |
| `active` | Boolean | Default true. False hides the channel without deleting it. |
| `channel_type` | Selection, required, read-only | Default `channel`. `chat` = a private one-to-one conversation; `group` = a private conversation among invited people; `channel` = a conversation that may be joined freely subject to its authorization group. The live chat capability adds `livechat`. **The type can never be changed after creation.** |
| `is_editable` | Boolean, computed, not stored, per user | True when the acting user may modify the channel. |
| `default_display_mode` | Selection | Empty or `video_full_screen`. Decides whether opening the channel from its invitation link starts in full-screen video. |
| `description` | Long text | The topic. |
| `image_128` | Image | The uploaded picture. |
| `avatar_128` | Image, computed, not stored | The uploaded picture when there is one, otherwise a generated picture: a fixed glyph for a channel, another for a group, tinted with a colour derived from the channel token. A conversation of type chat has no generated picture. |
| `avatar_cache_key` | Text, computed, not stored | A digest of the picture, or the literal `no-avatar`, used as a cache-busting key. |
| `channel_partner_ids` | Multiple links to Contact, computed, writable, searchable | The contacts among the members. Writing it creates the missing members and deletes the surplus ones. |
| `channel_member_ids` | Sub-records: Channel Member | The membership rows. |
| `parent_channel_id` | Link to Channel, read-only, indexed | The parent when this channel is a sub-thread. On deletion of the parent, the row is deleted. Access checks bypassed at query level. |
| `sub_channel_ids` | Sub-records: Channel, read-only | The sub-threads. |
| `from_message_id` | Link to Message, read-only | The message a sub-thread was started from. Unique — "Messages can only be linked to one sub-channel". |
| `pinned_message_ids` | Sub-records: Message | The messages of this channel that carry a pinning moment. |
| `sfu_channel_uuid` | Text | Readable only by the system group. The identifier of the channel on the selective forwarding unit, when one is used for calls. |
| `sfu_server_url` | Text | Readable only by the system group. The address of that unit. |
| `rtc_session_ids` | Sub-records: Call Session | Readable only by the system group. |
| `call_history_ids` | Sub-records: Call History | — |
| `is_member` | Boolean, computed with elevated rights, not stored, searchable | True when the acting party has a member row. |
| `self_member_id` | Link to Channel Member, computed with elevated rights, not stored | The acting party's own member row. |
| `invited_member_ids` | Sub-records: Channel Member, computed with elevated rights, not stored | The members currently being rung for a call. |
| `member_count` | Integer, computed with elevated rights, not stored | How many members. |
| `message_count` | Integer, computed, not stored, read-only | How many messages excluding the two notification types. |
| `last_interest_dt` | Date and time, indexed | Default: one second before the creation moment. Updated to the current moment whenever a message that is not a notification is posted. Used together with the member's own marker to decide pinning. |
| `group_ids` | Multiple links to Group | Auto-subscription groups: every active user of those groups is added as a member. Members may still leave afterwards. |
| `uuid` | Text, at most 50 characters, not copied | Default: a random ten-character token drawn from an unambiguous alphabet (no zero, no one, no letter that looks like a digit) and never containing the guest cookie separator. Unique — "The channel UUID must be unique". |
| `group_public_id` | Link to Group, computed, stored, editable, recursive | The authorization group: only its members may see and join the channel. Computed: a sub-thread inherits its parent's group; a top-level channel with no explicit group defaults to the internal-user group; a chat or a group conversation has none. |
| `invitation_url` | Text, computed, not stored | The path `/chat/<channel identifier>/<channel token>`. |
| `channel_name_member_ids` | Sub-records: Channel Member, computed, not stored | The first three members by identifier, used to build the display name of a conversation that has no name. Only computed for the types that use member-based naming, which is the group type. |

### Constraints

| Rule | Message |
|---|---|
| A sub-thread's initial message must belong to the parent channel or to one of its sub-threads, and must be a channel message. | "Cannot create <names>: initial message should belong to parent channel or one of its sub-channels." |
| A parent must not itself be a sub-thread, must be of type channel or group, and must have the same type as the child. | "Cannot create <names>: parent should not be a sub-channel and should be of type 'channel' or 'group'. The sub-channel should have the same type as the parent." |
| A conversation of type chat may not have more than two members. | "A channel of type 'chat' cannot have more than two users." |
| The authorization group and the auto-subscription groups are only allowed on a channel. | at database level: "Group authorization and group auto-subscription are only supported on channels."; at validation level: "For <names>, channel_type should be 'channel' to have the group-based authorization or group auto-subscription." |
| The type may not be changed. | "Cannot change the channel type of: <names>" |
| The initial message and the parent may not be changed. | "Cannot change initial message nor parent channel of: <names>." |
| The authorization group of a sub-thread may not be changed. | "Cannot change authorized group of sub-channel: <names>." |
| A channel may not have followers. | "Adding followers on channels is not possible. Consider adding members instead." |
| The shipped whole-company group may not be deleted. | "You cannot delete those groups, as the Whole Company group is required by other modules." |
| Only the member fields listed below may be supplied while creating a channel with members. | "Invalid field “<field>” when creating a channel with members." |
| Members supplied at creation must be supplied as creation commands. | "Invalid value when creating a channel with memberships, only 0 is allowed." |
| Contacts supplied at creation must be supplied as link or replace commands. | "Invalid value when creating a channel with members, only 4 or 6 are allowed." |

The member fields allowed at channel creation are: the contact, the guest, the unpin moment and the last-interest moment.

### Display name

- When the channel has a name, that name.
- Otherwise, the names of the first three members (contact name, or guest name), joined with the locale's list separator; when there are more than three members, a final element "1 other" or "<n> others" is appended.

### Creation side effects

1. The list of contacts to add is collected from the contact field and from the explicit membership commands.
2. Unless the platform is installing or the acting user is public, the acting user's contact is always added, so the creator can see the channel and has a correct pin state.
3. The channel is created with automatic logging and automatic subscription suppressed.
4. Every active user of every auto-subscription group who is not already a member is added as a member, and each new member is informed on their own broadcast channel.
5. The new channel is pushed to the creator's own broadcast channel.

### Deletion side effect

A deletion event naming the channel is broadcast on the channel's own broadcast channel.

### Modification side effect

A fixed list of fields is watched; when any of them actually changes, the new values are broadcast to the channel. The watched list is: the avatar cache key (channels and groups only), the type, the creator, the default display mode, the description (channels and groups only), the auto-subscription groups (channels only), the authorization group (channels only), the last-interest moment, the member count, the name and the token.

### Type-dependent capabilities

| Capability | Types allowing it |
|---|---|
| Leaving (as opposed to merely unpinning) | channel, group |
| Broadcasting read receipts to the whole channel | chat, group |
| Naming from the member list | group |
| Loading the member list lazily | channel, group |
| Inviting by electronic mail | group; channel when it has no authorization group |

### Recipient computation override

A channel is not an ordinary thread: it has no followers. The recipient computation is therefore replaced entirely.

1. If the message type is not one of comment, incoming electronic mail, outgoing electronic mail or an equivalent conversational type, there are **no** recipients at all.
2. For every contact explicitly mentioned in the message, excluding the author and excluding any contact whose address equals the sender address, and only when the contact is active: add a recipient entry with the contact's language, name, sharing flag and the notification preference of its first active user (internal users first, then by identifier), typed "user" when the contact is internal and has a preference, "customer" otherwise.
3. For every **member** of the channel other than the author, whose contact is active, who is not muted, whose user has not manually set the do-not-disturb status, and who passes the notification-preference test below: add a recipient entry whose delivery channel is browser push.

The notification-preference test is:

- for a conversation that is not of type channel: always pass;
- for a channel: pass when the member's own setting is "all messages"; or the member has no own setting and the user's global channel setting is "all messages"; or the member's own setting is "mentions only" and the contact is explicitly mentioned; or the member has no own setting, the user has no global setting, and the contact is explicitly mentioned.

All recipients of a channel message are then forced into the "customer" notification group, so the notification electronic mail is the minimal one without a link into the application.

Browser push is delivered only to the entries whose channel is browser push; inbox recipients are deliberately excluded so that a user who disabled push still gets the inbox entry and no duplicate.

The push title depends on the type: for a chat, the author's name; for a channel, `#<channel name> - <author name>`; for a group, `<channel name or joined member names> - <author name>`; otherwise `#<channel name>`.

### Posting overrides

- Posting a message that is not a notification updates the channel's last-interest moment.
- A special mention of everyone expands into the full member contact list.
- Every explicitly mentioned contact is filtered: in a channel with an authorization group, only contacts whose users belong to that group survive; in any other type, only contacts that are already members survive.
- Automatic subscription of the author and of the recipients is always disabled, because channels use membership, not followership.
- After posting, the author's own member row is advanced: the message becomes the last seen message and the new-message separator moves to just after it.
- After posting in a sub-thread, every mentioned contact that is a member of the parent and wants channel notifications is added to the sub-thread; in a sub-thread of a channel, mentioned contacts that are not yet members of the parent are added as well.
- Only a message of type comment may have its content updated — "Only messages type comment can have their content updated on model 'discuss.channel'".
- An attachment flagged as a voice recording gets its voice metadata row created.

### Bounce handling

When a bounce is received for a member whose bounce counter has reached ten, that member is removed from the channel.

---

## 35. Channel Member

Channel Member (`discuss.channel.member`, table `discuss_channel_member`). Adopts the bus sender behavior.

### Purpose

One party inside one channel, with all the per-person state: what they have read, what they have fetched, whether the channel is pinned in their sidebar, whether they muted it, what they renamed it to, and their call session.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `partner_id` | Link to Contact | Indexed. On deletion, the row is deleted. |
| `guest_id` | Link to Guest | Indexed. On deletion, the row is deleted. |
| `is_self` | Boolean, computed, not stored, searchable, per acting party | True when the row belongs to the acting contact or the acting guest. |
| `channel_id` | Link to Channel | Required. On deletion, the row is deleted. Access checks bypassed at query level. |
| `custom_channel_name` | Text | A name this member gave the channel, shown only to them. |
| `fetched_message_id` | Link to Message, indexed when not empty | The last message this member's client has downloaded. |
| `seen_message_id` | Link to Message, indexed when not empty | The last message this member has actually read. |
| `new_message_separator` | Integer | Required, default 0. The identifier **before which** the "new messages" line is drawn; equivalently, the first unread identifier. |
| `message_unread_counter` | Integer, computed with elevated rights, not stored | The number of unread messages. See the formula below. |
| `custom_notifications` | Selection | Empty (use the user's global setting), `all` ("All Messages"), `mentions` ("Mentions Only") or `no_notif` ("Nothing"). Applies only to channels. |
| `mute_until_dt` | Date and time | When set and in the future, no notification reaches this member. |
| `is_pinned` | Boolean, computed, not stored, searchable | Whether the channel appears in this member's sidebar. See the formula below. |
| `unpin_dt` | Date and time, indexed | The moment the member removed the channel from their sidebar. |
| `last_interest_dt` | Date and time, indexed | Default: one second before the creation moment. Updated on creating, joining and pinning. |
| `last_seen_dt` | Date and time | The moment the member last read something. |
| `rtc_session_ids` | Sub-records: Call Session | This member's live call participation. |
| `rtc_inviting_session_id` | Link to Call Session | Set while this member is being rung by that session. |

Index: (channel, contact, last seen message).

Uniqueness: (channel, contact) when the contact is set; (channel, guest) when the guest is set.

Constraints:

| Rule | Message |
|---|---|
| Exactly one of contact and guest must be set. | "A channel member must be a partner or a guest." |
| A public user may never be a member. | "Channel members cannot include public users." |
| The channel must be supplied at creation. | "It appears you're trying to create a channel member, but it seems like you forgot to specify the related channel. To move forward, please make sure to provide the necessary channel information." |
| A conversation of type chat that already has a member may not receive another. | "Adding more members to this chat isn't possible; it's designed for just two people." |
| The channel, the contact and the guest may never be changed. | "You can not write on <field name>." |

Display name: `“<member name>” in “<channel display name>”`, where the member name is the contact name or the guest name.

### The unread counter

```formula
unread_count = number of messages M such that
               M.model = channel model
           and M.record = this channel
           and M.type is neither "system notification" nor "user specific notification"
           and M.identifier ≥ member.new_message_separator
```

Two consequences follow from the use of `≥` on the identifier rather than a date:

- the separator value is the identifier of the **first unread** message, so marking everything read sets it to the last message identifier plus one;
- a message inserted with a lower identifier than an already-read one (which cannot happen in normal operation, identifiers being monotonic) would be counted as read.

### The pin rule

```formula
is_pinned = ( unpin_dt is empty )
            or ( member.last_interest_dt ≥ unpin_dt )
            or ( channel.last_interest_dt ≥ unpin_dt )
```

A channel therefore re-pins itself automatically as soon as anything interesting happens in it after the member unpinned it.

### Marking as read

Given a target message identifier:

1. Find the newest message of the channel whose identifier is at most the target. If there is none, do nothing.
2. Set the last seen message to it, provided the current last seen identifier is strictly lower. Doing so also raises the fetched message to the maximum of its current value and the new one, and stamps the last-seen moment.
3. Move the new-message separator to that identifier plus one.

When the channel type allows read receipts (chat and group), the new last-seen value is broadcast to the whole channel; otherwise only to the member.

### Creation and deletion side effects

- Creating a member of a sub-thread also adds the same party to the parent channel, so the member lists stay consistent.
- Creating or deleting a member of a type that names itself from its members re-broadcasts the naming member list.
- Deleting a member deletes its call sessions first, then removes the same party from every sub-thread of the channel (with the usual leave message), then deletes.

### Leaving a channel

Leaving is not simply deleting the row:

1. The party is unsubscribed as a follower (a no-op for channels, kept for safety).
2. The member's client is told to close the conversation window and to treat the channel as not locally pinned.
3. If there is no member row, stop.
4. For any type other than a plain channel, a notification message "left the channel" is posted, authored by the leaving contact.
5. The member row is deleted and the channel broadcasts the removal and the new member count.

### Muting

Setting a mute moment schedules the un-mute job for that moment. A periodic job clears every mute moment that is in the past and re-broadcasts the member.

### Typing indicator

A member may broadcast a transient typing flag to the whole channel. The broadcast carries the flag and the moment; nothing is stored.

### Garbage collection of sub-thread pins

A maintenance routine unpins a member of a **sub-thread** when all of the following hold: the member currently counts as pinned; both the member's and the channel's last-interest moments are older than two days; and there is no message in the sub-thread at or after the member's new-message separator that is not a notification. Each unpinned member is told to close the window.

---

## 36. Call Session and Call History

### 36.1 Call Session

Call Session (`discuss.channel.rtc.session`, table `discuss_channel_rtc_session`). Adopts the bus sender behavior.

One live audio or video participation.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `channel_member_id` | Link to Channel Member | Required. On deletion, the row is deleted. Unique — "There can only be one rtc session per channel member". |
| `channel_id` | Link to Channel, related, stored, read-only, indexed when not empty | Denormalized for querying. |
| `partner_id` | Link to Contact, related, stored, indexed | — |
| `guest_id` | Link to Guest, related | — |
| `write_date` | Date and time, indexed | Used as the heartbeat: a session that has not been touched recently is considered dead and deleted. |
| `is_screen_sharing_on` | Boolean | — |
| `is_camera_on` | Boolean | — |
| `is_muted` | Boolean | — |
| `is_deaf` | Boolean | Incoming sound disabled. |

Display name: the channel member.

**Peer-to-peer versus forwarding unit.** While fewer than three sessions exist in a channel, the call runs directly between browsers and the channel's forwarding-unit fields are cleared. From the third session on, if a forwarding unit is configured, a channel is requested from it and its identifier and address are stored on the channel; every existing session is then told to switch over. Each session receives a signed token carrying the forwarding-unit channel identifier, its own session identifier and the list of traversal servers; the token is valid for eight hours. The token used to *request* the channel is valid for thirty seconds. If the unit cannot be reached, the call silently stays peer-to-peer.

**Ringing.** Inviting members to a call sets their ringing-session link, broadcasts the invited list to the channel, and pushes a browser notification titled "Incoming call" with the body "Conference: <channel display name>", a vibration pattern, the "requires interaction" flag, a grouping tag of the form `call_<channel identifier>`, and two action buttons labelled "Decline" and "Accept". Cancelling the invitations clears the links, broadcasts the removal and pushes a cancellation payload with the same grouping tag.

Members eligible to be rung are those of the channel that are not already ringing, have no session, whose user has not manually set the do-not-disturb status, and, for guests, who have polled within the last twelve hours.

### 36.2 Call History

Call History (`discuss.call.history`, table `discuss_call_history`).

One call in one channel, from the moment the first session appeared to the moment the last one left.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `channel_id` | Link to Channel | Required, indexed. On deletion, the row is deleted. Also enforced by a check — "Call history must have a channel". |
| `duration_hour` | Decimal, computed, not stored | The elapsed time in hours. |
| `start_dt` | Date and time | Required, indexed. Also enforced by a check — "Call history must have a start date". |
| `end_dt` | Date and time | Empty while the call is running. |
| `start_call_message_id` | Link to Message, indexed | The notification message that announced the call. Unique — "Messages can only be linked to one call history". |

Ordering: newest start first, then newest identifier.

Index: (channel, end moment) restricted to rows with no end moment, so "is a call running in this channel" is a single lookup.

---

## 37. Guest

Guest (`mail.guest`, table `mail_guest`). Adopts the avatar behavior and the bus sender behavior.

### Purpose

An unauthenticated participant. A guest exists so that a visitor who has not logged in can hold a conversation, be addressed, react, and be recognized across requests.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. The display name the visitor gave. |
| `access_token` | Text | Required, read-only, not copied, readable only by the system group. Default: a freshly generated universally unique identifier. This is the whole credential of the guest. |
| `country_id` | Link to Country | Guessed from the request. |
| `email` | Text | Optional, collected by a chatbot step or a form. |
| `lang` | Selection over installed languages | — |
| `timezone` | Selection over time zones | — |
| `channel_ids` | Multiple links to Channel through the membership table, not copied | The channels the guest is a member of. |
| `presence_ids` | Sub-records: Presence | Readable only by the system group. |
| `im_status` | Text, computed with elevated rights, not stored | The derived online status. |
| `offline_since` | Date and time, computed with elevated rights, not stored | When the guest went offline. |

The guest identity is carried in a cookie named `dgid` whose value is the guest identifier and the access token joined by a separator character; the channel token alphabet deliberately excludes that separator.

---

## 38. Conversation side records

### 38.1 Voice Metadata

Voice Metadata (`discuss.voice.metadata`, table `discuss_voice_metadata`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `attachment_id` | Link to Attachment | Indexed, not copied. On deletion, the row is deleted. Access checks bypassed at query level. |

The mere existence of the row marks the attachment as a voice recording, which changes how it is rendered and how it is named in a push notification ("Voice Message" instead of the file name).

### 38.2 Favorite Animated Image

Favorite Animated Image (`discuss.gif.favorite`, table `discuss_gif_favorite`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `tenor_gif_id` | Text | Required. The identifier of the image in the external animated-image catalogue. |

Uniqueness: (creator, image identifier) — "User should not have duplicated favorite GIF".

### 38.3 Interactive Connectivity Server

Interactive Connectivity Server (`mail.ice.server`, table `mail_ice_server`).

One traversal server used to establish a direct connection between two browsers.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `server_type` | Selection | Required, default `stun`. Either a session-traversal server (`stun:`) or a relay server (`turn:`). |
| `uri` | Text | Required. The server address. |
| `username` | Text | Credential, used by relay servers. |
| `credential` | Text | Credential, used by relay servers. |

Display name: the address.

When the operator prefers an external provider, the traversal server list is fetched from that provider instead, using the account identifier and token stored in the system parameters.

### 38.4 User Settings — conversation fields

User Settings (`res.users.settings`, table `res_users_settings`). The base record belongs to the identity domain; this domain adds:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `is_discuss_sidebar_category_channel_open` | Boolean | Default true. Whether the channel section of the sidebar is expanded. |
| `is_discuss_sidebar_category_chat_open` | Boolean | Default true. Whether the direct-message section is expanded. |
| `push_to_talk_key` | Text | The key combination, encoded as `shift.control.alt.key`. |
| `use_push_to_talk` | Boolean | Default false. |
| `voice_active_duration` | Integer | Default 200. How long, in milliseconds, the microphone stays open after the volume drops below the threshold. |
| `volume_settings_ids` | Sub-records: User Settings Volume | Per-correspondent playback volumes. |
| `channel_notifications` | Selection | Empty (mentions only), `all` ("All Messages") or `no_notif` ("Nothing"). Applies to channels only. |

### 38.5 User Settings Volume

User Settings Volume (`res.users.settings.volumes`, table `res_users_settings_volumes`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `user_setting_id` | Link to User Settings | Required, indexed. On deletion, the row is deleted. |
| `partner_id` | Link to Contact | Indexed. On deletion, the row is deleted. |
| `guest_id` | Link to Guest | Indexed. On deletion, the row is deleted. |
| `volume` | Decimal | Default 0.5. Between 0 and 1; the scale is interpreted by the browser. |

Uniqueness: (settings, contact) when the contact is set; (settings, guest) when the guest is set.

Constraint: exactly one of contact and guest must be set — "A volume setting must have a partner or a guest."

---

## 39. Live Chat Channel and Live Chat Rule

### 39.1 Live Chat Channel

Live Chat Channel (`im_livechat.channel`, table `im_livechat_channel`). Adopts the rating-parent behavior.

#### Purpose

One public entry point for visitors. It carries the list of operators, the look of the button and of the conversation window, the capacity limit, the welcome text and the rules that decide how the button behaves on each page.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. |
| `button_text` | Text, translatable | Default "Need help? Chat with us.". |
| `default_message` | Text, translatable | Default "How may I help you?". The automated welcome line the visitor sees when a conversation starts. |
| `header_background_color` | Text | Default `#875A7B`. |
| `title_color` | Text | Default `#FFFFFF`. |
| `button_background_color` | Text | Default `#875A7B`. |
| `button_text_color` | Text | Default `#FFFFFF`. |
| `max_sessions_mode` | Selection | Default `unlimited`. Either "Unlimited" or "Limited". |
| `max_sessions` | Integer | Default 10. The maximum number of concurrent sessions per operator when the mode is limited. Must be strictly positive — "Concurrent session number should be greater than zero." |
| `block_assignment_during_call` | Boolean | When true, an operator who is in a call receives no new conversation. |
| `review_link` | Text | An optional address a satisfied visitor is redirected to. Must start with the plain or secure hypertext transfer scheme and have a host — "Invalid URL '<value>'. The Review Link must start with 'http://' or 'https://'." |
| `web_page` | Text, computed, not stored, read-only | `<base address>/im_livechat/support/<channel identifier>`, a ready-made public page. |
| `are_you_inside` | Boolean, computed, not stored, read-only | True when the acting user is one of the operators. |
| `available_operator_ids` | Multiple links to User, computed, not stored | The operators currently able to take a conversation. See the availability rule below. |
| `script_external` | Rich text, computed, not stored, read-only, not sanitized | The snippet to paste into an external site. |
| `nbr_channel` | Integer, computed, not stored, read-only | How many sessions the channel has ever had. |
| `user_ids` | Multiple links to User | The operators. Default: the acting user. |
| `channel_ids` | Sub-records: Channel | The sessions. |
| `chatbot_script_count` | Integer, computed, not stored | How many distinct scripts the rules reference. |
| `rule_ids` | Sub-records: Live Chat Rule | — |
| `ongoing_session_count` | Integer, computed, not stored | The number of ongoing sessions across all operators. |
| `remaining_session_capacity` | Integer, computed, not stored | See the formula below. |

Rating satisfaction is computed over the last 14 days.

#### Availability of an operator

An operator of a live chat channel is *available* when all of the following hold:

1. their presence status is "online";
2. either the channel's session mode is unlimited, or their current ongoing-session count for that channel is strictly below the maximum;
3. either the channel does not block assignment during calls, or the operator is not currently in a call.

The **ongoing-session count** of an operator for a channel is the number of channel-member rows of that operator in sessions of that live chat channel whose session end moment is empty and whose last-interest moment is within the last fifteen minutes.

#### Remaining capacity

```formula
eligible_operators = operators, minus those in a call when the channel blocks assignment during calls
total_capacity     = max_sessions × count(eligible_operators)
used_capacity      = sum over eligible operators of their ongoing-session count for this channel
remaining_capacity = max( 0 , total_capacity − used_capacity )
```

#### Availability of the channel as a whole

The channel can serve a visitor when it has at least one chatbot script configured on a rule, or at least one available operator.

### 39.2 Live Chat Rule

Live Chat Rule (`im_livechat.channel.rule`, table `im_livechat_channel_rule`).

One condition deciding how the chat button behaves on a page.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `regex_url` | Text | A pattern matched against the page address. An empty pattern matches every page. |
| `action` | Selection | Required, default `display_button`. `display_button` = "Show"; `display_button_and_text` = "Show with notification" (the button plus a floating line of text); `auto_popup` = "Open automatically" (on a large screen; on a small screen it behaves like "Show"); `hide_button` = "Hide". |
| `auto_popup_timer` | Integer | Default 0. The delay in seconds before the window opens by itself. Only meaningful with the automatic-opening action. |
| `chatbot_script_id` | Link to Chatbot Script | The script to run instead of routing to a human. |
| `chatbot_enabled_condition` | Selection | Required, default `always`. `always`; `only_if_no_operator` ("Only when no operator is available"); `only_if_operator` ("Only when an operator is available"). |
| `channel_id` | Link to Live Chat Channel, indexed when not empty | — |
| `country_ids` | Multiple links to Country | When set, the rule applies only to visitors located in those countries. Requires location lookup to be available. |
| `sequence` | Integer | Default 10. Lower wins when several rules match. |

Ordering: by sequence ascending.

#### Matching algorithm

Given a channel, a page address and an optional visitor country:

1. If a country is known, take the rules of the channel that list that country, ordered by sequence, and return the first one that passes the filter below.
2. Otherwise, or if none matched, take the rules of the channel that list **no** country, ordered by sequence, and return the first one that passes the filter.
3. If none matches, there is no rule.

The filter rejects a rule when:

- the address does not match the pattern (an empty pattern always matches; an unknown address is treated as the empty string);
- the rule names a script that is archived or has no step;
- the rule requires an available operator and there is none, or requires **no** available operator and there is one.

---

## 40. Live Chat Member History, Expertise and Conversation Tag

### 40.1 Live Chat Member History

Live Chat Member History (`im_livechat.channel.member.history`, table `im_livechat_channel_member_history`).

The permanent trace of one participant of one live chat session. It exists because the membership row is deleted when a participant leaves, while reporting must keep the facts.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `member_id` | Link to Channel Member, indexed when not empty | The live membership, while it exists. Unique — "Members can only be linked to one history". |
| `livechat_member_type` | Selection, computed from the member, stored | `agent`, `visitor` or `bot`. Once stored it is never recomputed away, because the computation keeps the existing value when there is one. |
| `channel_id` | Link to Channel, computed from the member, stored, indexed | On deletion, the row is deleted. |
| `guest_id` | Link to Guest, computed, stored, indexed when not empty | — |
| `partner_id` | Link to Contact, computed, stored, indexed when not empty | — |
| `chatbot_script_id` | Link to Chatbot Script, computed, stored, indexed when not empty | — |
| `agent_expertise_ids` | Multiple links to Expertise, computed, stored | The skills the operator had at the time. |
| `conversation_tag_ids` | Multiple links to Conversation Tag, related to the session | — |
| `avatar_128` | Image, computed, not stored | The contact's or the guest's avatar. |
| `session_country_id` | Link to Country, related to the session | — |
| `session_livechat_channel_id` | Link to Live Chat Channel, related to the session | — |
| `session_outcome` | Selection, related to the session | — |
| `session_start_hour` | Decimal, related to the session | — |
| `session_week_day` | Selection, related to the session | — |
| `session_duration_hour` | Decimal, computed, stored, averaged in reports | The hours between this participant's arrival and the end of the session. |
| `rating_id` | Link to Rating, computed, stored | For an operator or a bot, the rating of the session that targets that party. A session allows at most one rating. |
| `rating` | Decimal, related | — |
| `rating_text` | Selection, related | — |
| `call_history_ids` | Multiple links to Call History | The calls this participant took part in. |
| `has_call` | Decimal, computed, stored | 1 when the participant had at least one call, 0 otherwise. Stored as a number so it can be both summed and averaged. |
| `call_count` | Decimal, related to the flag, summed in reports | — |
| `call_percentage` | Decimal, related to the flag, averaged in reports | — |
| `call_duration_hour` | Decimal, computed, stored, summed in reports | The total hours of the participant's calls. |
| `message_count` | Integer, averaged in reports | The number of messages this participant sent in the session. |
| `help_status` | Selection, computed, stored | For an operator: `requested` when this history is the session's help-requesting operator, `provided` when it is the helping operator, otherwise empty. |
| `response_time_hour` | Decimal, averaged in reports | The time the participant took to answer. |

Uniqueness: one history per membership; one per (session, contact); one per (session, guest).

Constraints: a history may not carry both a contact and a guest — "History should either be linked to a partner or a guest but not both"; a history may exist only on a live chat session — "Cannot create history as it is only available for live chats: <names>."

Display name: the contact name, or the guest name; for a visitor that is a contact, the contact's full display name; "Unknown" when neither exists.

Session duration:

```formula
session_duration_hour = ( session_end_moment − participant_arrival_moment ) ÷ 3600 seconds
```

where the session end moment is the recorded end when the session is closed, otherwise the creation moment of the last message of the session, otherwise the current moment.

### 40.2 Expertise

Expertise (`im_livechat.expertise`, table `im_livechat_expertise`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. Unique. |
| `user_ids` | Multiple links to User, computed, not stored, writable | The operators holding this skill. The link is physically stored on the user's settings record; this field reads and writes it. |

### 40.3 Conversation Tag

Conversation Tag (`im_livechat.conversation.tag`, table `im_livechat_conversation_tag`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | Required. Unique. |
| `color` | Integer | Default: a random value between 1 and 11. |
| `conversation_ids` | Multiple links to Channel | The sessions carrying the tag. |

Ordering: by name. Deleting a tag first removes it from every session, so the sessions broadcast their new tag list.

---

## 41. Chatbot Script, Step, Answer and Message

### 41.1 Chatbot Script

Chatbot Script (`chatbot.script`, table `chatbot_script`). Adopts the image behavior and the campaign-source behavior.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `title` | Text, translatable | Required, default "Chatbot". Kept separate from the campaign-source name, which is manipulated by the campaign behavior. |
| `active` | Boolean | Default true. |
| `image_1920` | Image, related to the operator contact, writable | The bot's picture is the picture of its operator contact. |
| `script_step_ids` | Sub-records: Chatbot Script Step | Copied when the script is duplicated. |
| `operator_partner_id` | Link to Contact | Required, indexed, not copied. Deletion of the contact is refused while a script references it. The bot appears in the conversation as this contact. When a script is created without one, an **archived** contact is created automatically with the script title as name and the script image. |
| `livechat_channel_count` | Integer, computed, not stored | How many live chat channels reference the script through a rule. |
| `first_step_warning` | Selection, computed, not stored | `first_step_operator` when the last welcome step forwards to an operator; `first_step_invalid` when the last welcome step is not one of the question types (selection, address, telephone number, free input, multi-line free input); empty otherwise. |

Ordering: by title then identifier. Display name: the title. Duplicating appends " (copy)" to the title and re-links every triggering answer of the copied steps to the copied answers.

Constraint: a step of type "Question" must have at least one answer — "Step of type 'Question' must have answers."

### 41.2 Chatbot Script Step

Chatbot Script Step (`chatbot.script.step`, table `chatbot_script_step`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, computed, not stored | `<script title> - Step <sequence>`. |
| `message` | Rich text, translatable | What the bot says at this step. |
| `sequence` | Integer | Assigned automatically at creation: steps created for the same script are numbered consecutively starting at the current maximum plus one (or at zero when the script has no step). An explicitly supplied sequence wins and resets the counter. |
| `chatbot_script_id` | Link to Chatbot Script | Required, indexed. On deletion, the row is deleted. |
| `step_type` | Selection | Required, default `text`. See the table below. |
| `answer_ids` | Sub-records: Chatbot Script Answer | Copied on duplication. Cleared automatically when the step type stops being "Question". |
| `triggering_answer_ids` | Multiple links to Chatbot Script Answer, computed, stored, editable, not copied | "Only If": the step is shown only when **all** of these answers have been selected earlier. Restricted to answers of earlier steps of the same script; any answer whose step is no longer earlier is removed automatically. |
| `is_forward_operator` | Boolean, computed, not stored | True when the type is "Forward to Operator". |
| `is_forward_operator_child` | Boolean, computed, not stored | True when, walking back through the chain of parent steps (the nearest earlier forwarding or question step that this step depends on), a forwarding step is reached. Used to know that a step runs after the human has taken over. |
| `operator_expertise_ids` | Multiple links to Expertise | When forwarding, prefer operators holding these skills. |

Ordering: by sequence then identifier.

#### Step types

| Value | Label | Behavior |
|---|---|---|
| `text` | Text | The bot says the message and moves on. |
| `question_selection` | Question | The bot says the message and offers the answers as buttons. |
| `question_email` | Email | The bot asks for an address and validates it. |
| `question_phone` | Phone | The bot asks for a telephone number. |
| `forward_operator` | Forward to Operator | The bot hands the conversation to a human, preferring the skills listed on the step. |
| `free_input_single` | Free Input | The bot asks for a single-line answer. |
| `free_input_multi` | Free Input (Multi-Line) | The bot asks for a multi-line answer. |

### 41.3 Chatbot Script Answer

Chatbot Script Answer (`chatbot.script.answer`, table `chatbot_script_answer`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. The button label. |
| `sequence` | Integer | Default 1. |
| `redirect_link` | Text | When set, clicking the answer sends the visitor to this address. If the address leaves the site, the script ends. |
| `script_step_id` | Link to Chatbot Script Step | Required, indexed. On deletion, the row is deleted. |
| `chatbot_script_id` | Link to Chatbot Script, related | — |

Ordering: by step, then sequence, then identifier.

Display name: the step message shortened to 26 characters with an ellipsis, a colon, then the answer label. Searching the display name searches both the answer label and the step message.

### 41.4 Chatbot Message

Chatbot Message (`chatbot.message`, table `chatbot_message`).

The link between one posted message, the script step that produced it and the answer the visitor gave.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `mail_message_id` | Link to Message | Unique — "A mail.message can only be linked to a single chatbot message". |
| `discuss_channel_id` | Link to Channel | Required, indexed. On deletion, the row is deleted. |
| `script_step_id` | Link to Chatbot Script Step, indexed when not empty | — |
| `user_script_answer_id` | Link to Chatbot Script Answer | On deletion, set to empty. |
| `user_raw_script_answer_id` | Integer | The identifier of the chosen answer, kept as a plain number so statistics survive the deletion of the answer. |
| `user_raw_answer` | Rich text | What the visitor actually typed, for the free-input types. |

Ordering: newest creation first, then newest identifier. Display name: the session.

Index: (session, raw answer identifier) restricted to rows where the raw answer identifier is set.

---

## 42. Live Chat Session Report

Live Chat Session Report (`im_livechat.report.channel`, a read-only database view).

One row per live chat session, pre-aggregated for reporting. Because it is a view, it has no constraints, no defaults and no write path.

| Field (storage name) | Type | Meaning |
|---|---|---|
| `uuid` | Text | The session token. |
| `channel_id` | Link to Channel | The session. |
| `channel_name` | Text | Its name. |
| `livechat_channel_id` | Link to Live Chat Channel | The entry point. |
| `start_date` | Date and time | When the session started. |
| `start_hour` | Text | The hour of the start. |
| `start_date_minutes` | Text | The start truncated to the minute. |
| `day_number` | Selection | The day of the week, 0 for Sunday through 6 for Saturday. |
| `time_to_answer` | Decimal, six decimals, averaged | The hours until the first answer to the visitor. |
| `start_date_hour` | Text | The hour part of the start. |
| `duration` | Decimal, two decimals, averaged | The length of the session in minutes. |
| `nbr_message` | Integer, averaged | The number of messages. |
| `country_id` | Link to Country | The visitor's country. |
| `lang_id` | Link to Language, related | The visitor's language. |
| `rating` | Integer, averaged | The numeric rating. |
| `rating_text` | Text | The satisfaction label. |
| `partner_id` | Link to Contact | The operator. |
| `handled_by_bot` | Integer, summed | 1 when a bot handled the session. |
| `handled_by_agent` | Integer, summed | 1 when a human handled the session. |
| `visitor_partner_id` | Link to Contact | The visitor when identified. |
| `call_duration_hour` | Decimal, two decimals, averaged | — |
| `has_call` | Decimal | 1 when the session had a call. |
| `number_of_calls` | Decimal, related, summed | — |
| `percentage_of_calls` | Decimal, related, averaged | — |
| `session_outcome` | Selection | `no_answer` ("Never Answered"), `no_agent` ("No one Available"), `no_failure` ("Success"), `escalated` ("Escalated"). |
| `chatbot_script_id` | Link to Chatbot Script | — |
| `chatbot_answers_path` | Text | The identifiers of the answers the visitor chose, in order. |
| `chatbot_answers_path_str` | Text | The same as readable labels. |
| `session_expertises` | Text | The skills used in the session, as text. |
| `session_expertise_ids` | Multiple links to Expertise, related | — |
| `conversation_tag_ids` | Multiple links to Conversation Tag, related | — |
| `agent_requesting_help_history` | Link to Member History, related | — |
| `agent_providing_help_history` | Link to Member History, related | — |

Ordering: by start date, then live chat channel, then session.

### Live chat fields added to the Channel record

The live chat capability adds one selection value, `livechat` ("Livechat Conversation"), to the channel type, and the following fields:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `duration` | Decimal, computed, not stored | The length of the session in hours. |
| `livechat_lang_id` | Link to Language | The visitor's language. |
| `livechat_end_dt` | Date and time | Set when either the visitor or the last operator leaves. An empty value means the session is running. |
| `livechat_channel_id` | Link to Live Chat Channel, indexed when not empty | — |
| `livechat_operator_id` | Link to Contact, indexed when not empty | The operator (a human's contact, or a bot's operator contact). Required for a live chat session — "Livechat Operator ID is required for a channel of type livechat." |
| `livechat_channel_member_history_ids` | Sub-records: Member History | — |
| `livechat_expertise_ids` | Multiple links to Expertise, stored | The skills used. |
| `livechat_agent_history_ids`, `livechat_bot_history_ids`, `livechat_customer_history_ids` | Sub-records: Member History, computed, searchable | The histories split by participant type. |
| `livechat_agent_partner_ids`, `livechat_bot_partner_ids`, `livechat_customer_partner_ids` | Multiple links to Contact, computed, stored | The same split at contact level. |
| `livechat_customer_guest_ids` | Multiple links to Guest, computed | — |
| `livechat_agent_requesting_help_history` | Link to Member History, computed, stored | The operator who asked for help. |
| `livechat_agent_providing_help_history` | Link to Member History, computed, stored | The operator who came to help. |
| `livechat_note` | Rich text | An internal note about the session, visible to internal users. |
| `livechat_status` | Selection, computed, stored, editable | `in_progress` ("In progress"), `waiting` ("Waiting for customer"), `need_help` ("Looking for help"). Must be empty once the session is closed — "Closed Live Chat session should not have a status." |
| `livechat_outcome` | Selection, computed, stored | `no_answer`, `no_agent`, `no_failure`, `escalated`. |
| `livechat_conversation_tag_ids` | Multiple links to Conversation Tag | Visible to live chat operators. |
| `livechat_start_hour` | Decimal, computed, stored | — |
| `livechat_week_day` | Selection, computed, stored | 0 for Monday through 6 for Sunday. |
| `livechat_matches_self_lang`, `livechat_matches_self_expertise` | Booleans, computed, searchable | Whether the session matches the acting operator's languages and skills, used to filter the queue. |
| `chatbot_current_step_id` | Link to Chatbot Script Step | Where the script currently stands. |
| `chatbot_message_ids` | Sub-records: Chatbot Message | Visible to live chat managers. |
| `country_id` | Link to Country | The visitor's country. |
| `livechat_failure` | Selection | `no_answer`, `no_agent`, `no_failure`. Set to "never answered" the moment a human is assigned, and cleared when the human actually answers. |
| `livechat_is_escalated` | Boolean, computed, stored | — |
| `rating_last_text` | Selection, stored | The last satisfaction label, made stored for reporting. |

Indexes: on the end moment restricted to running sessions; on the failure restricted to the two failure values; on the escalation flag restricted to true; and on (type, creation date) restricted to live chat sessions.

The live chat capability also adds to the Channel Member: the history rows, the participant type (`agent`, `visitor`, `bot`, computed with an inverse), the script the member runs when it is a bot, and the skills the member had as an operator.
