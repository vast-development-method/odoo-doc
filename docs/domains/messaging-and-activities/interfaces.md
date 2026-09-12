# Messaging and Activities — Interfaces

Everything this domain exposes: the menus and screens, the named operations other domains and clients call, the request routes, the event bus contract, the data contract sent to the browser, the printable documents, the shipped bodies, the external services contacted, and what can be imported or exported.

**A note on route paths.** A path is reproduced exactly, in code font, because an embedded widget, an external add-in and a delivery-report callback all address it character for character. What a rebuild must reproduce is the whole contract: the grouping of operations, the authentication level, the inputs, the outputs, the side effects and the errors.

Contents:

1. [Named operations on a thread-enabled record](#1-named-operations-on-a-thread-enabled-record)
2. [Named operations on activities](#2-named-operations-on-activities)
3. [Named operations on templates and composers](#3-named-operations-on-templates-and-composers)
4. [Named operations on channels](#4-named-operations-on-channels)
5. [Request routes](#5-request-routes)
6. [The event bus](#6-the-event-bus)
7. [The client data contract](#7-the-client-data-contract)
8. [Menus and screens](#8-menus-and-screens)
9. [Printable documents and shipped bodies](#9-printable-documents-and-shipped-bodies)
10. [External services](#10-external-services)
11. [The electronic-mail client plugin contract](#11-the-electronic-mail-client-plugin-contract)
12. [Import and export](#12-import-and-export)
13. [Errors returned to a client](#13-errors-returned-to-a-client)

---

## 1. Named operations on a thread-enabled record

These are the operations every other domain calls. The rule identifiers in the error column point at [business-rules.md](business-rules.md).

| Operation | Inputs | Result | Records changed | Errors |
|---|---|---|---|---|
| post a message | body, subject, message type, sender address, author, parent, subtype by identifier or by external identifier, direct recipients, extra outgoing addresses, already-reached "to" and carbon-copy lists, inline attachments, existing attachment identifiers, plus the notification parameters | the created Message | Message, attachments, followers, notifications, outgoing mails, broadcasts | MSG-040 to MSG-052 |
| notify parties | body, subject, author, sender address, model, record, subtype, recipients, attachments | the created Message of the user-specific type | a Message that no conversation shows, notifications, outgoing mails | MSG-046, MSG-047 |
| log an internal note | body, subject, author, attachments, tracking values | the created Message | a Message only; no notification at all | — |
| log a note on a batch | one body per record, a shared subject, an author | the created Messages | Messages only | MSG-049 |
| log a note rendered from a view | a view reference, the render values, the message type | the created Message | renders the body first, then logs | MSG-043 |
| post from a source | a Template or a view, plus the posting parameters | the created Messages | renders the body, then posts | MSG-043, MSG-098 to MSG-101 |
| mail from a source | a Template or a view, plus the mailing parameters | the created Outgoing Mails | renders per record and queues, without posting | same |
| subscribe | contacts, optional subtypes | true or false | Follower rows created or rewritten | MSG-054 |
| unsubscribe | contacts | nothing | Follower rows deleted | MSG-055 |
| read the followers | a cursor, a page size, a "recipients only" flag | follower data | none | — |
| move the conversation | the new record, optionally a new parent message | nothing | moves every Message of one record onto another and re-parents them | — |
| update a message's content | the message, body, attachments, mentioned contacts, subject, scheduled moment, a strict flag | nothing | rewrites the message, deletes its translations, broadcasts the update | MSG-025 to MSG-027 |
| cancel my delivery failures | a notification channel | true | cancels the acting user's failing notifications of that channel on this model | MSG-069 |
| default recipients | an "include carbon copy" flag | per record: contact identifiers, addresses, carbon-copy addresses | none | — |
| suggested recipients | a "consider the discussion" flag, a specific message, a "do not create contacts" flag, a new primary address, extra contacts | per record: the ordered proposal list | creates contacts when creation is allowed | — |
| find contacts from addresses | per record a list of raw addresses, plus the "avoid alias addresses" and "do not create" flags | per record the resolved contacts | creates contacts when allowed | MSG-166, MSG-167 |
| the record's main customer | — | the customer Contact | none | — |
| the record's companies | a default company | one company per record | none | — |
| receive a bounce | — | nothing | raises the record's bounce counter | — |
| reset the bounce counter | — | nothing | sets the counter back to zero | — |

**Source resolution errors**, shared by the two "from a source" operations, are quoted in [business-rules.md](business-rules.md), section 10, rules MSG-098 to MSG-101.

---

## 2. Named operations on activities

| Operation | Inputs | Result | Side effects |
|---|---|---|---|
| schedule | a type by external identifier or by identifier, a due date, a summary, a note, extra values | the created Activities | notifies and subscribes each assignee; sets the automated flag |
| schedule with a rendered note | the same plus a view reference and its render values | the created Activities | renders the note first |
| search | a list of type external identifiers, an optional assignee, an optional extra filter, an "only automated" flag defaulting to true | the matching Activities | none |
| reschedule | the same filters plus a new due date and optionally a new assignee | nothing | moves the due dates, reassigns |
| complete | the same filters plus a feedback text and optional attachments | nothing | runs the completion sequence on each match |
| remove | the same filters | nothing | deletes them, posting nothing |
| mark one done | feedback, attachments | the posted message identifier | posts the completion message, moves the attachments, archives, chains |
| mark done and schedule the next | the same | an action opening a pre-filled window, or false when a successor was created automatically | the same, then opens the window |
| cancel | — | nothing | deletes the Activity |
| reschedule to today, to tomorrow, to next week | — | nothing | moves the due date; "next week" lands on the Monday of the following week |
| send a template | a Template identifier | true | posts a message rendered from the template with the discussion subtype |
| read the activity board | model, filter, page size, offset, an "include completed" flag | the aggregated activity data per record and per type | none |

All of them do nothing at all when the "skip activity automation" switch is on.

---

## 3. Named operations on templates and composers

| Operation | Inputs | Result | Errors |
|---|---|---|---|
| render one field | the field name, the record identifiers, the engine, the options | one rendered value per record | MSG-104 to MSG-107 |
| generate the values | record identifiers, the field list, an "allow suggested recipients" flag, a "create contacts from addresses" flag | one value map per record with recipients and attachments resolved | — |
| send a template | one record identifier, a "send now" flag, a "raise on failure" flag, value overrides, a layout | the created Outgoing Mail identifier | MSG-111 |
| send a template in batch | record identifiers and the same options | the created Outgoing Mails | — |
| preview a template | the template, a record reference, a language | the rendered subject, sender, recipients, carbon copy, reply address, scheduled moment, body, reports and attachments | — |
| reset templates | the templates | nothing | MSG-114 |
| send from the composer | — (the composer carries everything) | the posted Messages or the queued Outgoing Mails | MSG-116, MSG-117 |
| schedule from the composer | — | nothing | MSG-118, MSG-119 |
| save the composer as a template | a template name | the created Template | MSG-120 |

---

## 4. Named operations on channels

| Operation | Inputs | Result |
|---|---|---|
| create a channel | name, authorization group | the Channel |
| create a group conversation | contacts, default display mode, name | the Channel |
| open or create a direct conversation | the contacts, a pin flag | the Channel |
| create a sub-thread | the source message, a name | the Channel |
| add members | contacts, guests, a "ring into the running call" flag, a "post the join notice" flag | the new memberships |
| invite by address | addresses | nothing |
| join | — | the membership |
| leave | — | nothing |
| rename, set the description, set a personal channel name | the new value | nothing |
| pin, unpin | a pin flag | nothing |
| mute | a number of minutes, or −1 meaning "until I unmute" | nothing |
| set the personal notification preference | all, mentions, nothing, or unset | nothing |
| mark read up to a message | the message identifier | nothing |
| move the unread separator | a message identifier | nothing |
| broadcast the typing indicator | a typing flag | nothing |
| pin or unpin a message | the message identifier, a pin flag | nothing |
| join a call | the session identifiers the client already knows, a camera flag | the call state, the traversal server list, the forwarding unit information |
| leave a call | a session identifier | nothing |
| cancel a call invitation | the member identifiers | nothing |
| signal to a peer | the peer notifications | nothing |
| update a call session | the session identifier and the values | nothing |
| heartbeat | the channel, the session, the sessions the client knows | the refreshed call state |
| mention suggestions | a search text, a page size | the matching contacts, roles and channels |
| invitation suggestions | a search text, the channel, a page size | the contacts that may be invited |

---

## 5. Request routes

Authentication levels: **public** (anyone, including an unauthenticated visitor, possibly identified by a guest token), **user** (an authenticated user), **none** (no session at all, used for static assets and provider callbacks), **plugin** (an electronic-mail client add-in authenticated by its own application key).

### 5.1 Conversation and message routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/mail/data` | post | public | Bulk read of messaging data with no side effect. Takes a list of fetch requests and a context; answers a data-store payload. |
| `/mail/action` | post | public | The same for requests that do have a side effect. |
| `/mail/thread/messages` | post | user | Pages through the messages of one record. |
| `/mail/message/post` | post | public | Posts a message on a record. |
| `/mail/message/update_content` | post | public | Edits a message. |
| `/mail/message/reaction` | post | public | Adds or removes a reaction; answers the whole reaction group. |
| `/mail/message/<identifier>` | get | public | Redirects to the record the message belongs to. |
| `/mail/message/translate` | post | user | Translates a message body and caches the result. |
| `/mail/thread/recipients` | post | user | The suggested recipients of a reply, creating contacts as needed. |
| `/mail/thread/recipients/fields` | post | user | Which fields of the model hold recipients. |
| `/mail/thread/recipients/get_suggested_recipients` | post | user | Recomputes the suggestions with the edits the client already made. |
| `/mail/partner/from_email` | post | user | Resolves addresses into contacts, creating them. |
| `/mail/thread/subscribe` | post | user | Adds followers. |
| `/mail/thread/unsubscribe` | post | user | Removes followers. |
| `/mail/read_subscription_data` | post | user | Which subtypes exist for the model and which the follower is subscribed to. |
| `/mail/view` | get | public | The generic access point of a notification electronic mail. Redirects, in order of availability, to a public page, to the record with an access token, to a login page, or to an explanatory page. |
| `/mail/unfollow` | get | public | Unsubscribes a contact from a record through a signed link and shows a confirmation page. |
| `/mail/inbox/messages`, `/mail/history/messages`, `/mail/starred/messages` | post | user | The three mailboxes. |
| `/mail/set_manual_im_status` | post | user | Forces the displayed presence status. |
| `/mail/link_preview`, `/mail/link_preview/hide` | post | public | Fetches or dismisses a link preview. |
| `/mail/attachment/upload` | post | public | Uploads a file onto a record or a channel. |
| `/mail/attachment/delete` | post | public | Deletes an attachment, given its access token. |
| `/mail/attachment/zip` | post | public | Downloads several attachments as one archive. |
| `/mail/attachment/pdf_first_page/<identifier>` | get | public | The first page of a portable-document attachment, as an image, for preview. |
| `/mail/attachment/update_thumbnail` | post | public | Stores a thumbnail the client generated. |
| `/mail/guest/update_name` | post | public | Renames a guest. |
| `/mail/font_to_img/...` | get | none | Renders an icon as an image, for mail clients that do not load icon fonts. |

### 5.2 Channel routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/discuss/channel/messages` | post | public | Pages through a channel's messages. |
| `/discuss/channel/pinned_messages` | post | public | The pinned messages. |
| `/discuss/channel/members` | post | public | Pages through the members. |
| `/discuss/channel/mark_as_read` | post | public | Marks read up to a message. |
| `/discuss/channel/set_new_message_separator` | post | public | Moves the unread separator, which is how a conversation is marked unread again. |
| `/discuss/channel/notify_typing` | post | public | Broadcasts the typing indicator. |
| `/discuss/channel/attachments` | post | public | Pages through the channel's attachments. |
| `/discuss/channel/join` | post | public | Joins a channel. |
| `/discuss/channel/update_avatar` | post | user | Replaces the channel picture. |
| `/discuss/channel/sub_channel/create` | post | public | Opens a sub-thread from a message. |
| `/discuss/channel/sub_channel/fetch` | post | public | Pages through the sub-threads. |
| `/discuss/channel/sub_channel/delete` | post | user | Deletes a sub-thread. |
| `/discuss/channel/ping` | post | public | The heartbeat of a call participant. |
| `/discuss/settings/mute` | post | user | Mutes one channel or every channel for a number of minutes. |
| `/discuss/settings/custom_notifications` | post | user | Sets the notification preference of one channel or of every channel. |
| `/discuss/search` | post | public | Searches across channels, messages and contacts. |
| `/discuss/channel/<identifier>` | get | public | The public page of a channel. |
| `/chat/<token>`, `/chat/<token>/<name>` | get | public | Opens or creates a conversation from a shared token. |
| `/meet/<token>`, `/meet/<token>/<name>` | get | public | The same, opening directly in full-screen video. |
| `/chat/<channel identifier>/<channel token>` | get | public | Joins a channel through its invitation link. |
| `/mail/rtc/channel/join_call`, `/mail/rtc/channel/leave_call`, `/mail/rtc/channel/upgrade_connection`, `/mail/rtc/channel/cancel_call_invitation` | post | public, except the upgrade which is user | Call control. |
| `/mail/rtc/session/update_and_broadcast`, `/mail/rtc/session/notify_call_members` | post | public | Session state and peer-to-peer signalling. |
| `/mail/rtc/audio_worklet_processor_v2`, `/discuss/voice/worklet_processor` | get | public | Audio processing helpers served as standalone scripts. |
| `/discuss/gif/search`, `/discuss/gif/categories`, `/discuss/gif/add_favorite`, `/discuss/gif/favorites`, `/discuss/gif/remove_favorite` | post | user | The animated-image picker. |
| `/websocket/update_bus_presence` | post | public | Refreshes the presence of the current session by hand. |

### 5.3 Live chat routes

Every route below exists twice: once for a visitor on the platform's own site, identified by cookies, and once in a cross-origin variant that takes an explicit guest token as its first input, for a widget embedded in an external site. The cross-origin variants deliberately ignore any user or guest cookie and refuse a token that does not belong to the addressed session.

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/im_livechat/loader/<channel identifier>` | get | public | The widget loader of one entry point. |
| `/im_livechat/support/<channel identifier>` | get | public | A standalone support page hosting the widget. |
| `/im_livechat/assets_embed.js`, `/im_livechat/emoji_bundle`, `/im_livechat/font-awesome` | get | public or none | Static assets of the widget. |
| `/im_livechat/get_session` | post | public | Opens a session, persisted or not. Inputs: the entry point, the previous operator, the chatbot script, the persistence flag. Answers the session data, or false when no operator could be resolved. |
| `/im_livechat/init` | post | public | The initial data of an embedded widget: the availability flag, the base address, the client worker version and, when available, the look settings, the welcome line, the button text, the entry point name and identifier, the review address and the default visitor name. |
| `/im_livechat/visitor_leave_session` | post | public | The visitor closed the widget; closes the session. |
| `/im_livechat/feedback` | post | public | Applies a rating with an optional comment. |
| `/im_livechat/history` | post | public | Posts the visitor's browsing history into the session. |
| `/im_livechat/email_livechat_transcript` | post | user | Mails the transcript to an address. |
| `/im_livechat/download_transcript/<channel identifier>` | get | public | Downloads the transcript as a printable document. |
| `/im_livechat/session/update_note`, `/im_livechat/session/update_status` | post | user | The internal note and the working status of a session. |
| `/im_livechat/conversation/update_tags`, `/im_livechat/conversation/write_expertises`, `/im_livechat/conversation/create_and_link_expertise` | post | user | Qualification of a conversation. |
| `/chatbot/step/trigger` | post | public | Plays the next script step. |
| `/chatbot/answer/save` | post | public | Records the answer the visitor chose. |
| `/chatbot/step/validate_email` | post | public | Validates an address answer. |
| `/chatbot/restart` | post | public | Restarts the script. |
| `/chatbot/<script identifier>/test` | get | user | Runs a script in a disposable session, for testing. |

### 5.4 Mailing group routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/groups` | get | public | The list of visible lists, with a subscription box. |
| `/groups/<list>`, `/groups/<list>/page/<number>` | get | public | The public archive of one list, paginated. |
| `/groups/<list>/<message>` | get | public | One post with its answers. |
| `/groups/<list>/<message>/get_replies` | post | public | Loads more answers. |
| `/group/subscribe`, `/group/unsubscribe` | post | public | Requests a subscription change; applied at once for a signed-in user, otherwise a confirmation message is sent. |
| `/group/subscribe-confirm`, `/group/unsubscribe-confirm` | get | public | Applies the change through the signed link. |
| `/group/<list>/unsubscribe_oneclick` | post | public | The one-click unsubscription announced in the list headers. |
| `/group/is_member` | post | public | Whether an address is a member. |

### 5.5 Digest routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/digest/<digest identifier>/unsubscribe` | get | user | Removes the acting user from the digest's recipients. |
| `/digest/<digest identifier>/unsubscribe_oneclik` | get, post | public | The one-click unsubscription announced in the headers, carrying the recipient identifier and the signed token. The path is reproduced exactly, including its spelling. |
| `/digest/<digest identifier>/set_periodicity` | get | user | Switches the periodicity from a link inside the digest. |

### 5.6 Rating routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/rate/<token>/<value>` | get | public | Applies a rating from a link inside a message and shows the feedback page. |
| `/rate/<token>/submit_feedback` | get, post | public | Submits or changes the free-text comment. |

### 5.7 Delivery report routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/sms/status` | post | public | A batch of delivery reports from the text message service: a list of entries, each with a state and the correlation tokens it applies to. Entries that resolve are applied; the others are ignored, so one bad token never loses a batch. |
| `/sms_twilio/status/<token>` | post | public | One delivery report from the external telephony provider, carrying the provider status and an error code. The request signature is verified before anything is applied. |

### 5.8 Electronic-mail client plugin routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/mail_plugin/auth`, `/mail_plugin/auth/confirm` | get, post | user | The consent screen that grants an add-in access. |
| `/mail_plugin/auth/access_token` | post | none | Exchanges a consent code for an application key. |
| `/mail_plugin/auth/check_version` | post | none | Answers the single value 1, telling the add-in that the bridge is installed. |
| `/mail_plugin/partner/get` | post | plugin | Looks a contact up by address, enriching and creating a company when possible. |
| `/mail_plugin/partner/search` | post | plugin | Searches contacts by name or address. |
| `/mail_plugin/partner/create` | post | plugin | Creates a contact with an address, a name and an optional parent company. |
| `/mail_plugin/partner/enrich_and_create_company`, `/mail_plugin/partner/enrich_and_update_company` | post | plugin | Enriches a contact from the enrichment service. |
| `/mail_plugin/log_mail_content` | post | plugin | Files the open message on a record as an internal note. |
| `/mail_plugin/get_translations` | post | plugin | The translations of the add-in's own texts. |

---

## 6. The event bus

### 6.1 Channel naming

A broadcast channel is one of three things:

1. a plain string, which must not be guessable, because anybody who knows it receives the traffic;
2. a record, in which case the name is derived from the record's model and identifier;
3. a record plus a sub-channel name, which narrows the audience of that record's channel.

Only a fixed set of record types may act as a channel: the User, the Contact, the Guest, the Channel, the Channel Member, the Message, the Presence, the Call Session, the Attachment, the Live Chat Channel and the Mailing Group. Every one of them adopts the bus sender behavior.

The channel value written on the entry is a serialized structure holding the database name and either the record reference alone or the record reference and the sub-channel name.

### 6.2 Poll contract

A client polls with three inputs: the list of channel names it listens to, the identifier of the last entry it saw, and an optional list of identifiers to ignore. The answer is every entry on those channels whose identifier is strictly greater than the last one seen, in identifier order.

Entries are written at the end of the transaction, and the list of touched channels is announced only **after** the commit, so that a client polling immediately after the commit finds the rows. When the announcement payload exceeds the transport limit it is split recursively in halves until every part fits (see [calculations.md](calculations.md), section 32).

Entries older than the retention window are deleted by the garbage collector; a client that reconnects inside that window receives everything it missed.

### 6.3 Subscription

On subscribing, a client is told which channels it may listen to: its own user or guest channel, its contact channel, the channel of every conversation it is a member of, the presence channel of every party it holds a scoped token for, and, for a live chat operator, the channels of the Live Chat Channels it operates. Presences missed while the client was away are delivered at subscription time so the interface starts with a correct status for everybody.

### 6.4 Payload catalogue

| Payload type | Channel | Content | Emitted when |
|---|---|---|---|
| `mail.message/inbox` | the recipient user | the message identifier and the message data with the follower information | an inbox Notification is created |
| `mail.message/notification_update` | the message author | the notifications of the message with their statuses | a delivery succeeds, fails or is cancelled |
| `mail.message/delete` | every affected user | the message identifiers | a Message is deleted |
| `mail.message/toggle_star` | the acting user | the message identifiers and the new starred flag | a message is starred or unstarred |
| `mail.record/insert` | various | records and fields to merge into the client store | almost every change: channel headers, memberships, presences, reactions, link previews, attachments, followers, canned responses |
| `discuss.channel/new_message` | the channel | the message data, the channel identifier, an optional temporary identifier, an optional silent flag | a message is posted in a channel |
| `discuss.channel/joined` | the new member | the channel identifier, the channel data, the membership, whether to ring, and who invited | a member is added |
| `discuss.channel/leave` | the leaving member | the channel identifier | a member leaves |
| `discuss.channel/delete` | the channel | the channel identifier | a channel is deleted |
| `discuss.channel/unpin` | the member | the channel identifier | the client must close the conversation window |
| `discuss.channel/transient_message` | the caller alone | a message body that is never stored | a command answers only its caller |
| `discuss.channel.member/typing_status` | the channel | the member and the typing flag | typing starts or stops |
| `discuss.channel.member/seen` | the channel or the member | the member and the last seen message | a member marks a conversation read |
| `discuss.channel.rtc.session/update_and_broadcast`, `.../peer_notification`, `.../ended` | the channel or one peer | session state, peer signalling, the end of a session | call events |
| `bus.bus/im_status_updated` | the presence channel of a user or a guest | the new status and, when offline, since when | a presence changes |
| `mail.activity/updated` | the assignee's user | whether an activity was created or deleted and the counter difference | an Activity due today or earlier is created, reassigned, rescheduled or deleted |
| `mail.canned.response/insert`, `.../delete` | the entitled users | the response data | a Canned Response changes |
| `ir.attachment/delete` | the channel of the record | the attachment identifier | an Attachment is deleted |

---

## 7. The client data contract

Every route that answers a client returns a **store payload**: a map from record type to a list of records, each record carrying only the fields the client needs. The client merges the payload into its local store. The contract per record type is:

| Record type | Fields sent |
|---|---|
| Message | identifier, body, date, message type, subtype with its description and internal flag, author contact or author guest, sender address, record name, model and record identifier, direct recipients with name and avatar, attachments, link previews in order, reactions grouped by content, tracking values the reader may see, pinning moment, parent, the "needs action" flag, the "has error" flag, the starred flag, the edited marker, the scheduled moment for a scheduled entry, and the notifications the client-filtering rule keeps |
| Notification | identifier, status, failure type, failure reason, recipient contact or bare address, channel |
| Follower | identifier, contact with name and avatar, the subtypes followed, the "is active" flag |
| Channel | identifier, name, display name, type, description, avatar cache key, member count, last-interest moment, the acting party's own membership, the authorization group, the default display mode, the invitation address, the token, the parent channel, the sub-thread count, whether a call is running, and, for a live chat session, the operator, the entry point, the end moment, the working status, the tags and the expertise |
| Channel Member | identifier, contact or guest, personal channel name, last seen message, fetched message, unread separator, unread counter, mute moment, personal notification preference, pin state, the participant type for a live chat session |
| Contact | identifier, name, display name, avatar cache key, presence status, offline-since moment, the "is internal" flag, the address when the reader may see it |
| Guest | identifier, name, avatar cache key, presence status |
| Activity | identifier, summary, note, due date, state, type with its icon and decoration, assignee, the "can write" flag, the chaining rule, the offered templates |
| Attachment | identifier, name, media type, thumbnail flag, voice flag, access token, owning record |
| Link Preview | identifier, source address, title, description, image address, site name, media type, sequence, hidden flag |
| Canned Response | identifier, shortcut, substitution, shared flag, editable flag |
| Presence | identifier, status, offline-since moment |
| Call Session | identifier, channel member, camera flag, microphone flag, deafened flag, screen-sharing flag |

A record the reader may not see is never included; a field the reader may not read is omitted rather than emptied.

---

## 8. Menus and screens

### 8.1 The conversation panel on a record

Present on the form of every thread-enabled record.

| Element | Content | Guard |
|---|---|---|
| Send message | An inline composer with a body, recipients and attachments; posts with the discussion subtype. | The model's post-access permission on the record. |
| Log note | The same composer, posting with the internal-note subtype. | Internal users only. |
| Activities | Opens the activity scheduling window. | The model must be activity-enabled; internal users only. |
| Attachments | A counter and a panel listing the record's files. | Internal users only. |
| Followers | A counter, the list, add and remove, and a per-follower subtype editor. | Adding or removing somebody else needs write access. |
| Follow and Unfollow | Toggles the reader's own subscription. | Read access is enough. |
| Message list | Messages newest first, with the author, the moment, the body, the attachments, the reactions, the tracking lines, the delivery-failure badge and, for a scheduled entry, its planned moment. | Internal notes are hidden from portal and public users. |
| Message actions | Reply, edit, delete, react, star, translate, copy the link, open a sub-thread, pin. | Editing and deleting follow MSG-025 to MSG-032. |
| Failure badge | Shown on a message whose notification failed, with the recipient list, the reason and a retry action. | Only the author of the message sees it. |
| Scheduled messages | A banner listing the entries waiting to be posted, with edit, send now and cancel. | Only the creator and an administrator may send now. |

### 8.2 The conversation workspace

| Screen | Content |
|---|---|
| Sidebar | Three sections: the mailboxes (Inbox, Starred, History), the channels and the direct messages. Each entry shows its unread counter; a muted entry is dimmed. Each section remembers whether it is expanded, in the user's settings. |
| Inbox | The messages the reader has an unread inbox notification for, with "mark as read" per message and "mark all as read". |
| Starred | The messages the reader starred, with "unstar all". |
| History | The messages the reader has already marked read. |
| Conversation | The message list, the composer, the member panel, the pinned-message panel, the attachment panel, the sub-thread panel and the call controls. |
| Call controls | Join, leave, camera, microphone, screen share, deafen, participant tiles and connection quality. |
| Settings dialogs | Notification settings (per channel and the global preference) and voice and video settings (the push-to-talk key, the voice activity duration, the per-correspondent volumes). |

### 8.3 The live chat back office

| Screen | Fields shown | Buttons and guards | Filters and groupings |
|---|---|---|---|
| Live Chat Channels | Name, session count, operators, availability, entry-point address | Join and Leave (live chat users only), view the sessions, view the ratings, view the chatbots | — |
| Live Chat Channel form | The look settings, the welcome line, the operator list, the capacity rule, the review address, the display rules and the embedding snippet | Test the widget | — |
| Sessions | Session name, entry point, operator, visitor, duration, rating, outcome, tags, expertise, working status | Open the conversation | Date, outcome, rating, agent, expertise, tag, country, day of the week, start hour |
| Looking for help | The sessions whose working status is "Looking for help" | Join the conversation, which succeeds only while the status is unchanged | Date |
| Member history | Participant, participant type, session, response time, session duration, call time, message count, rating, help status | Open the session | Agent, help status, rating, country, outcome, expertise, tag, start hour, day of the week, month |
| Session report | The measures of the Live Chat Session Report | Drill down to the session | Date, entry point, agent, country, language, outcome, chatbot script |
| Chatbot form | Title, bot picture, the steps with their type, message, answers and conditions | Test the script | — |
| Expertise, Conversation Tags | Name | — | — |

### 8.4 Activity screens

| Screen | Content |
|---|---|
| Activity board of a model | A grid of records against activity types; each cell shows the count and the worst state, coloured by state and by the type's decoration. Clicking a cell opens the activities. |
| My activities | The reader's activities, filtered by default to overdue and today, in list, card and calendar form. |
| Activity overview | Every activity, for administrators. |
| Other activities | The activities the reader is assigned to but whose record they cannot open. Opening one shows a read-only form of the activity itself. |
| Activity indicator | A counter in the top bar, grouped by model, split into overdue, today and planned, limited to the configured aggregation limit. |

### 8.5 Technical screens

| Screen | Purpose |
|---|---|
| Emails | The outgoing queue with the state, the failure type, the failure reason, the recipients and the scheduled moment. Buttons: Send now, Retry, Cancel. |
| Notifications | The per-recipient delivery records with their status and failure. |
| Messages | Every message, with its model, record, type, subtype and author. |
| Tracking values | The recorded field changes. |
| Subtypes | The message subtypes. |
| Followers | Every subscription. |
| Scheduled messages and deferred notifications | The two deferral records, with their moment and a "send now" action. |
| Aliases and Alias Domains | The address space, with each alias's validity status. |
| Gateway allowed senders | The senders exempt from the loop quota, with a help text explaining the two loop parameters. |
| Blacklisted addresses | The suppression list, with the removal window that asks for a reason. |
| Templates and text message templates | With the preview and reset actions, and a placeholder assistant that builds an expression from a field path and a fallback. |
| Channels, members, call sessions, call histories, traversal servers, guests, reactions, link previews, favorite animated images, user settings | Administrative lists. |
| Activity types and activity plans | The configuration of activities. |
| Mailing groups, moderation rules, posts, members | Mailing list administration, with Accept, Reject, Allow and Ban on a pending post. |
| Digests | The periodic summaries, with the indicator switches, the recipients, the periodicity and a "send now" action. |
| Postal letters | The postal queue with the state, the error code, the explanation and the re-queue and cancel actions. |

### 8.6 Menus

The domain contributes one top-level application, the conversation workspace, with the three mailboxes, the channel list and the direct-message list. Its configuration menu carries the technical screens of section 8.5. Live chat contributes its own top-level application with the conversations, the reports and the configuration of entry points, chatbots, expertise and tags. Mailing groups contribute a menu under the configuration application. Digests and postal letters appear under the general settings and technical menus respectively.

---

## 9. Printable documents and shipped bodies

| Document | Content | Grouping and totals |
|---|---|---|
| Live chat conversation | A printable transcript of one session: the entry point, the operator, the visitor, the start and end moments, and every message with its author, moment and body. An image attachment is rendered inline, any other attachment as a link. Moments are shown in the visitor's time zone when one is known, otherwise in coordinated universal time. | One document per session; no totals. |
| Live chat transcript by electronic mail | The same content wrapped in a message whose subject is "Conversation with <operator display name>". | None. |
| Live chat dashboards | Two shipped spreadsheet dashboards over the session report: an overall one and one restricted to running sessions. | Grouped by entry point, agent, outcome and period. |
| Mailing group archive | Public pages listing a list's accepted posts, newest first, each with its answers. | Paginated; the navigation groups by month. |
| Notification electronic mail | The rendered layout described below. | None. |
| Digest electronic mail | The indicator table, the tip, the preferences block and the connect button, wrapped in the digest layout. | Three columns per indicator with a margin each. |
| Postal letter | The rendered report of the source document, optionally preceded by an address cover page, with the margins whitened so the envelope window is never obstructed. | None. |

### 9.1 The standard notification layout

Rendered, in order:

1. **The preview line**, a hidden block placed first so a mail client shows it next to the subject. Its content is: when the message carries tracking values, the first triple written "<field label>: <old value> → <new value>", followed by " |..." when there is more than one triple and by " | " when a preview line also exists; then, when the subtype is internal, the literal "Internal communication: "; then the message preview line; then a long run of invisible padding characters — a combining grapheme joiner followed by a zero-width space, repeated one hundred and forty times — whose only purpose is to push the real content out of the client's preview area.
2. **The header**, shown when the header is forced, or when headers are allowed and the group has an access button. It carries the structured annotations describing the view action, then a row with the access button labelled "View <model description>" (or "View" when the description is unknown), then the subtitles with the first in bold, then a horizontal rule. When there are no subtitles the record name is shown with slashes replaced by hyphens. The button uses the company's secondary colour as background and its primary colour as text.
3. **The content**: the message body.
4. **The tracking values**, one line each, written "<field label>: <old value or "None"> → <new value or "None">".
5. **The signature**, when the message asks for one and it is not empty.
6. **The footer**, shown when the footer is forced, or when footers are allowed, the header is shown and the author is an internal user. It carries the company name, then the company's telephone number, address and site address separated by vertical bars, then the credit line and, when unfollowing is offered, a separator and an Unfollow link.

### 9.2 The layout variants

| Variant | Difference from the standard layout |
|---|---|
| Light layout | No header and no access button at all. Used for a customer-facing message such as a rating request. |
| Responsible-signature layout | Replaces the author's signature by the signature of the record's responsible. The replacement applies only when the signature switch is on, the record exists, the model has a responsible field, that field is filled, the acting identity is not the technical superuser, and the responsible's signature is not empty; otherwise no signature is rendered at all, not even the author's. |
| Follower invitation layout | The subtitles are hidden when there is no access button; the body is replaced by "<author name> (<author address>) added you as a follower of this <model description>." followed, when the caller supplied one, by that body in grey; and the unfollow block moves above the footer rule and reads "Not interested by this? Unfollow". |
| Multiple-record follower invitation layout | The same with the opening sentence "<author name> (<author address>) added you as a follower of <model description> listed below:". |

When the named layout cannot be found or renders empty, the raw message body is sent unwrapped and a warning is logged.

### 9.3 Shipped bodies posted as messages

| Body | Rendered with | Content |
|---|---|---|
| Responsible assignment notice | the record | "Dear <responsible name>," then "You have been assigned to the <model description or the word document> <the record display name>." |
| Activity assignment notice | the activity and the model description | "Dear <assignee name>," then "<the person who created the activity> has just assigned you the following activity:" followed by a list carrying the document name with the model description in brackets, the summary when there is one, and the due date. |
| Activity completion notice | the activity, the feedback text and the "different assignee" flag | The activity type icon and name followed by the word "done", then " (originally assigned to <assignee name>)" when somebody else closed it, then ": <summary>" when there is one; then, when the activity carried a note, the heading "Original note:" and the note; then, when feedback was given, the heading "Feedback:" and the feedback text with its line breaks preserved. |
| Origin link notice | the record and the source records | "This <model description in lower case> has been created from:" followed by links to the source records separated by commas; the wording becomes "has been modified from:" when the caller says the record was edited rather than created. |
| Channel invitation body | the channel, the invitation text, the signed token and the acting user | A framed block containing the invitation text and a centred "Join Channel" button linking to the channel's invitation address with the token appended. The button uses the company's secondary colour as background and its primary colour as text. |
| Account security alert | the user and the item that changed | Sent to the user when a sensitive account change happens; an address change is sent to the **previous** address. |

### 9.4 Shipped gateway bodies

| Body | Sent when | Content |
|---|---|---|
| Alias security bounce | the alias's contact-security check refuses the sender | The wrapper appends the original message. The text is the custom body when the alias defines one, otherwise the four-paragraph refusal quoted in [business-rules.md](business-rules.md), section 9. |
| Invalid alias bounce | the alias itself is misconfigured, or creation through it failed | The three-paragraph refusal quoted in the same section. |
| Catch-all bounce | a message is addressed to the catch-all mailbox only, or routing ends with a catch-all recipient and no route | States that the address is not monitored, and invites the sender to write to the company's own address, which is shown. |
| Notification limit bounce | loop detection triggered | States that too many messages were received from this address in a short period. Its references carry the loop-detection tag. |
| Closed mailing list bounce | a message is addressed to a closed list | States that the list no longer accepts messages. |

Every gateway bounce has: no author; the subject "Re: <original subject>"; the recipient taken from the return-path header of the incoming message, falling back to its sender; automatic deletion switched on; and the sender "MAILER-DAEMON" plus the company bounce address, falling back to the "to" header of the incoming message when none of its addresses is a catch-all, falling back to "MAILER-DAEMON" plus the acting user's normalized address. Bounces are created with elevated rights and sent immediately.

---

## 10. External services

| Service | Contacted for | Contract |
|---|---|---|
| Outgoing relay | Every outgoing electronic mail | Address, port, encryption, credentials and the maximum message size, all configured on the Outgoing Mail Server. A relay owned by a user is personal and is throttled. |
| Incoming mailbox | Polling for incoming messages | Address, port, encryption, credentials and the protocol kind, configured on the Incoming Mail Server. |
| Browser push service | Delivering a browser push payload | The endpoint the browser supplied, the browser's public key, the shared authentication secret and the installation's signing key pair. A device the service reports as permanently gone is deleted. |
| Text message service | Sending text messages and receiving delivery reports | The address in the system parameter, the account token, one entry per distinct body carrying the pairs of number and correlation token, and the address at which reports must be posted back. |
| External telephony provider | The same, as an alternative | The account identifier and token stored on the Company, the sending numbers per country, and a signed callback per message. |
| Postal printing service | Printing and posting a document | Per letter: the letter reference, the source model and record, the recipient's formatted postal address, the structured address, the return address of the sending company, the printable document, the page count when the call is an estimate, and the company logotype. Once per request: the account token, the installation reference, the print options with the currency of the price quote, and a flag saying the request is a batch so an insufficient-credit condition is reported per letter. |
| Company enrichment service | Enriching a contact from its address domain | The search key of the address; the answer is a document describing the company. |
| Animated image service | The animated-image picker | The service key from the system parameters. |
| Translation service | Translating a message body | The service key from the system parameters; the answer carries the detected source language and the translated body. |
| Traversal and forwarding servers | Establishing and relaying a call | Either the locally configured Interactive Connectivity Servers, at most five, or the list obtained from the external telephony provider; and, above the participant threshold, the forwarding unit addressed with a signed token. |
| Publisher announcement service | The weekly exchange | Sends the installation reference, the installation creation date, the installation name, the platform version, the number of active users, the number of active users who signed in within the last fifteen days, the number of active external users, the number of active external users who signed in within the last fifteen days, the language of the acting user, the base address, the list of installed capability packages, the subscription code and, when the acting user's contact belongs to a company, that company's name, address and telephone number. |

---

## 11. The electronic-mail client plugin contract

The plugin capability lets an add-in running inside a person's electronic-mail client read and write a small part of the platform: look up the sender of the open message, create the contact when it is unknown, enrich its company from the enrichment service, and file the open message on a record. Every call crosses an origin boundary, so the bridge has its own authentication scheme.

### 11.1 Authentication

**Leg one, consent.** The add-in opens `/mail_plugin/auth` in the browser. A caller who is not an internal user sees the error page "Access Error: Only Internal Users can link their inboxes to this database." and no code is issued. On acceptance, `/mail_plugin/auth/confirm` receives the requested scope, the add-in name, an optional extra information string, the return address and the caller's opaque state value, and builds a consent code:

```formula
grant_name   = add_in_name                                       when no extra information is given
grant_name   = add_in_name + ": " + extra_information            otherwise
payload      = the structure { scope , grant_name , timestamp in seconds since the start of 1970 , acting user identifier }
               serialized with its keys in ascending order
signature    = keyed_digest( installation_secret , purpose "mail_plugin" , payload )
consent_code = base_64( payload ) + "." + base_64( signature )
```

The browser is redirected to the return address with a success flag of 1, the consent code and the state value echoed back. On refusal the redirect carries a success flag of 0 and the state value, and no code.

**Leg two, exchange.** `/mail_plugin/auth/access_token` accepts the code from any origin, without authentication:

1. An empty code answers the structure carrying the error "Invalid code".
2. The code is split at the full stop into payload and signature, both decoded. The signature is recomputed and compared **in constant time**; a mismatch answers "Invalid code".
3. The payload timestamp is compared with the current moment. A code older than three minutes answers "Invalid code".
4. Otherwise the request switches to the identity named by the payload, an application key is generated with the plugin scope followed by the consented scope and labelled with the grant name, valid for one day, and the answer carries that key.

**Every other bridge route** requires the authorization header carrying the application key, optionally prefixed by the word "Bearer". A missing header is refused with "Access token missing"; a key whose scope is not the plugin scope is refused with "Access token invalid". On success the request runs with the identity and the context of the key's user, so all ordinary access rules apply unchanged.

### 11.2 Contact lookup

`/mail_plugin/partner/get` takes either a contact identifier, or a name together with an address.

1. Neither given: the error "You need to specify at least the partner_id or the name and the email".
2. A contact identifier given: that contact is read and returned.
3. An address that cannot be normalized: the error "Bad Email.".
4. The normalized address equals the default-sender address of any Alias Domain: the answer is a synthetic contact named "Notification" carrying that address and the outcome "a platform-side explanation" with the text "This is your notification address. Search the Contact manually to link this email to a record." Nothing is created.
5. Otherwise the first contact is taken whose stored address is the normalized or the raw address, or whose normalized address equals the normalized address.

The answer always has three parts: the contact, the identifiers of the companies the acting user belongs to, and whether the acting user may create contacts.

**The contact part** carries the identifier, the name, the address, the telephone number, whether it is a company, the small image, the job position, the enrichment outcome, and whether the acting user may modify it, tested with a real write check. A contact with no name is named from its address: the display part when there is one, otherwise the normalized address.

**The company part** describes the contact itself when it is a company, its parent when it has one, and otherwise the empty company written as identifier −1. A company the acting user may not read is returned as its identifier and the name "No Access". A readable company carries the identifier, the name, the telephone number, the address, the site address, an address block of street, city, postal code and country name, the parsed enrichment document and the large image.

**When no contact matched**, the answer carries a placeholder contact with identifier −1 and the requested address and name, and the platform tries to attach a company: first an existing enrichment record whose search key matches; failing that a company contact whose normalized address ends with the search key; failing that, and only when the acting user may create contacts, an enrichment that creates the company.

The **search key** of an address is the address domain preceded by the at sign, except when that domain is a known generic mailbox provider, in which case it is the whole address. Two people at one company therefore share one enrichment, while two people at one generic provider do not.

### 11.3 Search, creation and filing

| Route | Inputs | Behaviour |
|---|---|---|
| `/mail_plugin/partner/search` | the search term, an optional limit defaulting to thirty | When the term normalizes as an address, contacts are searched on the normalized address containing the term; otherwise on the complete name containing the term, or the reference being exactly the term, or the address containing the term. The answer is a list of contact parts. |
| `/mail_plugin/partner/create` | address, name, parent company identifier | Refused outright when the address is the default sender of an Alias Domain. Otherwise a contact is created with that name and address, and the given parent when the identifier is greater than −1. The answer is the new identifier. |
| `/mail_plugin/log_mail_content` | the model, the record, the body, an optional list of attachments as name and encoded-content pairs | Refused outright when the model is not in the allowed list, which contains Contact only unless another capability widens it. Otherwise the body is posted as an internal note on the record with the decoded attachments, through the ordinary posting operation, so followers are notified exactly as for a message typed in the platform. |
| `/mail_plugin/get_translations` | none | The translations of the add-in's own texts, as a map from source text to translated text, in the acting user's language. |

### 11.4 Company enrichment

Every enrichment answer carries exactly one outcome.

| Outcome | Meaning | Extra information |
|---|---|---|
| company created | A company contact was created from the answer. | none |
| company updated | An existing company contact was completed from the answer. | none |
| missing data | The address domain is a generic mailbox provider, so no company can be derived. | none |
| insufficient credit | The prepaid balance of the enrichment service is exhausted. | the address where credits can be bought |
| no data | The service answered but knows nothing about the domain. | "The enrichment service found no data for the email provided." |
| other | Any other failure of the call. | "Unknown reason" |
| a platform-side explanation | Used for the notification-address case of section 11.2. | the explanation text |

**Creating a company from an answer.** A company contact is created with the company flag set; the name from the answer or, failing that, the address domain; the street, the city and the postal code from the answer; the first telephone number; the site address from the answer domain; the first address; the search key; and the answer document itself. When the answer carries a logotype address, the image is downloaded with a two-second timeout and stored; a failed download is ignored. When the answer carries a country code the matching country is set, and when it also carries a region code the matching region of that country is set. Finally an internal note rendered from the answer is posted on the new company.

`/mail_plugin/partner/enrich_and_create_company` takes a contact identifier and answers the error "This partner does not exist" when it is gone, "The partner already has a company related to him" when it already has a parent, and "The email of this contact is not valid and we can not enrich it" when its address cannot be normalized. Otherwise the company is created and becomes the contact's parent.

`/mail_plugin/partner/enrich_and_update_company` takes a contact that must be a company — otherwise "Contact must be a company" — with a normalizable address. The answer then fills, and only where the field is still empty, the telephone number, the enrichment document, the image, the street, the city, the postal code and the site address. An internal note rendered from the answer is posted and the outcome is "company updated".

---

## 12. Import and export

The domain exports no data file of its own. Two rules apply:

- **Messages may not be exported** by anyone who is not an administrator — "Only administrators are allowed to export mail message".
- **Every other model of the domain follows the platform's ordinary import and export**, subject to its access rights. In practice only the configuration models are ever imported: Message Subtypes, Activity Types, Activity Plans and their lines, Templates, Alias Domains, Aliases, Live Chat Channels and their rules, Chatbot Scripts with their steps and answers, Mailing Groups and their moderation rules, Digests, and Gateway Allowed Senders.

Two contractual consequences of importing an Alias: the local part is sanitized on import exactly as on manual entry, and the uniqueness check of the pair (local part, domain) is enforced on import too, so a bulk import that would create a duplicate address fails as a whole.

---

## 13. Errors returned to a client

| Situation | What the client sees |
|---|---|
| Access denied on a Message, an Activity or a Channel | The five-line access refusal naming the document type, the operation, the first six record identifiers and the acting user identifier. |
| A validation of this domain | The exact text quoted in [business-rules.md](business-rules.md), shown as a blocking dialog. |
| A guest token that does not match the addressed session | The request is refused and the widget starts a fresh session. |
| A missing record behind a notification link | The access point falls back, in order, to a public page, to the record with an access token, to a login page, and finally to an explanatory page. |
| A malformed delivery report | The entries that resolve are applied and the others are ignored, so one bad token never loses a whole batch. |
| An unknown extra parameter on the single-message read path | Logged as a warning and ignored, so a stale client cannot break. |

---

## Reconciliation notes

1. **Route prefixes.** One source version wrote the live chat paths under a neutral prefix and the text message paths under a spelled-out prefix. The paths reproduced here are the ones an embedded widget and an external delivery-report callback actually address, because they are part of the integration contract; a rebuild that changes them must change the widget and the provider registration with them.
2. **The digest unsubscribe path.** The shipped path is spelled `/digest/<digest identifier>/unsubscribe_oneclik`, without the second letter c of the word "click". It is reproduced verbatim because the one-click header announces it to mail clients; this is recorded as a **compatibility finding**, and a corrected behaviour would serve both spellings and announce the corrected one.
3. **The bridge outcome values.** One source version noted that two shipped outcome values name the product and an abbreviation. They are described here by their function; a rebuild may keep any stable values provided the seven outcomes stay distinguishable.
