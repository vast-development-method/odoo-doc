# Messaging and Activities — Configuration

Everything an operator or an administrator sets before and while the domain runs: the capability packages that switch parts of it on, the settings shown in the configuration screen, the system parameters, the fixed constants a rebuild must reproduce, the shipped master data, the scheduled jobs, the security groups, the access rights, the record rules, and the declarations a model must make in order to become thread-enabled.

Values reproduced in code font (parameter keys, stored selection values, external identifiers) are contractual: a rebuild that must read an existing database or honour an existing integration depends on them character for character. Each carries its full name in words the first time it appears.

Contents:

1. [Capability packages](#1-capability-packages)
2. [Settings of the configuration screen](#2-settings-of-the-configuration-screen)
3. [System parameters](#3-system-parameters)
4. [Fixed constants](#4-fixed-constants)
5. [Shipped master data](#5-shipped-master-data)
6. [Scheduled jobs](#6-scheduled-jobs)
7. [Security groups](#7-security-groups)
8. [Access rights](#8-access-rights)
9. [Record rules](#9-record-rules)
10. [Master-data prerequisites](#10-master-data-prerequisites)
11. [What a thread-enabled model must declare](#11-what-a-thread-enabled-model-must-declare)

---

## 1. Capability packages

The domain ships as a set of capability packages. Each is named here by its business name; the machine-readable catalogues under `schemas/` hold the technical keys that tooling needs.

| Capability package | What it adds |
|---|---|
| Notification bus | The event bus: the broadcast entry, the bus sender behavior and the websocket subscription contract. Everything else depends on it. |
| Discussion and messaging | The core: the Thread behavior, Messages, Subtypes, Followers, Notifications, Tracking Values, Outgoing Mail, Aliases and Alias Domains, the incoming gateway, Templates and the rendering contract, Activities and Plans, Channels, Guests, calls, Canned Responses, Roles, browser push, the Blacklist Entry, the composer and the wizards. |
| Assistant bot | The guided onboarding conversation of the built-in bot contact. |
| Mailing groups | Public discussion lists fed by electronic mail, their members, their posts, their moderation rules and the moderation wizard. |
| Mailing groups on the website | The public archive, the subscription page and the one-click unsubscribe route of a mailing group. |
| Follow buttons on public pages | The follow and unfollow controls shown on a public page of a thread-enabled record. |
| Live chat | Live chat entry points, display rules, operators, chatbot scripts, expertise, conversation tags, the participation history and the session report. |
| Live chat on the website | The chat widget served by a website, the linkage between a website visitor and a session, and operator-initiated chat requests. |
| Text messages | Text messages, their templates, their composer, their trackers, their queue and the text-message notification channel. |
| Text messages through the external telephony provider | An alternative sending service with its account credentials and its sending numbers. |
| Text messages on calendar events | Reminders of a meeting sent as text messages. |
| Postal mail | Postal Letters, their queue and the postal notification channel. |
| Postal mail for accounting documents | The "by post" sending method offered on a customer invoice and on a follow-up report. |
| Prepaid service notices for messaging | Turns the credit notices of the external prepaid services into messages addressed to the operator. |
| Periodic digest | Digests, their indicators, their tips and their sending job. |
| Electronic-mail client plugin | The bridge an add-in running inside a person's electronic-mail client uses to look up, create and enrich contacts and to file the open message on a record. |

---

## 2. Settings of the configuration screen

Each setting is stored either as a field of the Company, as a field of the Website, or as a system parameter, as the "Stored as" column says.

### 2.1 Electronic mail

| Setting | Type | Default | Stored as | Effect |
|---|---|---|---|---|
| Alias Domain | Link to Alias Domain | the first Alias Domain by ordering | Company field `alias_domain_id` (alias domain) | The address space of the company: its catch-all, its bounce address and its default sender. |
| Use Custom Email Servers | Boolean | off | screen switch only | Reveals the incoming-server and outgoing-server configuration. Switching it off deletes nothing. |
| Restrict Template Rendering | Boolean | on | system parameter `mail.restrict.template.rendering` | When on, only a member of the template-editor group may create or modify a Template containing anything other than a plain field path. Everyone may still render an existing template. |
| Email Button Text colour | Text | `#FFFFFF` | Company field `email_primary_color` (primary colour of notification electronic mail) | The text colour of the access button in a notification electronic mail. |
| Email Button colour | Text | `#875A7B` | Company field `email_secondary_color` (secondary colour of notification electronic mail) | The background colour of that button and of the links in the footer. |
| Failed emails | Integer, read-only | computed | screen indicator | The number of Outgoing Mails in the failure state, shown with a link to the failure list. |

Two further switches install an authentication package for a hosted mailbox provider so that an incoming or outgoing server may authenticate with a delegated token instead of a password. They add no field of this domain.

### 2.2 Real-time conversation

| Setting | Type | Default | Stored as | Effect |
|---|---|---|---|---|
| Use external traversal servers | Boolean | off | system parameter `mail.use_twilio_rtc_servers` | Ask the external telephony provider for the traversal server list instead of using the locally configured Interactive Connectivity Servers. |
| External telephony account identifier | Text | empty | system parameter `mail.twilio_account_sid` | Credential of that provider. |
| External telephony account token | Text | empty | system parameter `mail.twilio_account_token` | Credential of that provider. |
| Use a forwarding unit | Boolean | off | system parameter `mail.use_sfu_server` | Route calls through a media forwarding unit once the participant threshold is reached. |
| Forwarding unit address | Text | empty | system parameter `mail.sfu_server_url` | The address of that unit. |
| Forwarding unit key | Text | empty | system parameter `mail.sfu_server_key` | The shared key, stored encoded, used to sign the per-channel token. |

### 2.3 Content services

| Setting | Type | Default | Stored as | Effect |
|---|---|---|---|---|
| Animated image service key | Text | empty | system parameter `discuss.klipy_api_key` | Enables the animated-image picker in the conversation composer. |
| Message translation service key | Text | empty | system parameter `mail.google_translate_api_key` | Enables the "translate this message" action and the Message Translation cache. |

### 2.4 Postal mail

| Setting | Type | Default | Stored as | Effect |
|---|---|---|---|---|
| Print in colour | Boolean | on | Company field `snailmail_color` (postal colour printing) | The default colour option of a new Postal Letter. |
| Add a cover page | Boolean | off | Company field `snailmail_cover` (postal cover page) | The default cover-page option. Forced on for a document layout that carries no address block of its own. |
| Print both sides | Boolean | off | Company field `snailmail_duplex` (postal double-sided printing) | The default double-sided option. |

### 2.5 Text messages

| Setting | Type | Default | Stored as | Effect |
|---|---|---|---|---|
| Text message provider | Selection | `odoo` (the platform's own sending service) | Company field `sms_provider` (text message provider) | Which sending service is used: the platform's own service or the external telephony provider. The stored value `twilio` selects the external provider. |
| Telephony account identifier | Text | empty | Company field `sms_twilio_account_sid` | Must start with the two letters `AC`, be thirty-four characters long and contain only letters and digits after that prefix. |
| Telephony account token | Text | empty | Company field `sms_twilio_auth_token` | Readable only by the system group. |
| Sending numbers | Sub-records: Provider Number | empty | Company field `sms_twilio_number_ids` | One row per sending number, each with its country and its preference order. |

### 2.6 Live chat

| Setting | Type | Default | Stored as | Effect |
|---|---|---|---|---|
| Website live chat channel | Link to Live Chat Channel | empty | Website field `channel_id` (live chat channel of the website) | The entry point whose widget is served on that site. |

### 2.7 Digest

| Setting | Type | Default | Stored as | Effect |
|---|---|---|---|---|
| Digest Emails | Boolean | on when a Digest exists for the company | screen switch backed by the Digest activation state | Switching it off deactivates the company's default Digest; switching it on activates it. |
| Default digest | Link to Digest | the shipped default Digest of the company | screen field | Which Digest the switch acts on. |

---

## 3. System parameters

| Key | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `mail.restrict.template.rendering` | restrict template rendering | Boolean as text | `1`, shipped | See section 2.1. The stored value is normalized to a boolean on write. |
| `mail.catchall.domain.allowed` | allowed catch-all domains | comma-separated domains | empty | Extra right-hand sides accepted when matching an Alias by local part alone. Validated as a comma-separated list of domains; an empty result after cleaning is refused. |
| `mail.gateway.loop.minutes` | gateway loop window in minutes | Integer | 120 | The observation window of the incoming loop detection. |
| `mail.gateway.loop.threshold` | gateway loop threshold | Integer | 20 | The number of records created, or messages received on one record, inside the window that marks the sender as looping. |
| `mail.batch_size` | generation batch size | Integer | 50 | Recipients per generated notification electronic mail, and records per template rendering chunk. A stored value of zero is treated as 50 so the loop always progresses. |
| `mail.mail.queue.batch.size` | outgoing queue batch size | Integer | 1000 | Outgoing Mails processed by one queue run. |
| `mail.mail.force.send.limit` | immediate sending limit | Integer | 100 | Above this number of produced Outgoing Mails, immediate sending is refused and the queue takes over. |
| `mail.session.batch.size` | session batch size | Integer | 1000 for the sending session, 500 for a mailing-group relay | Rows handled inside one transport session, and members per relay chunk of a Mailing Group. |
| `mail.server.personal.limit.minutes` | personal relay limit per minute | Integer | 30 when unset or zero | Recipients per minute allowed through one personal relay. |
| `mail.disable_personal_mail_servers` | disable personal relays | Boolean as text | off | When on, a personal relay is never selected and only relays without an owner are allowed. |
| `mail.link_preview_throttle` | link preview throttle | Integer | 99 | The maximum number of previews fetched from one host inside the throttling window. |
| `mail.activity.gc.delete_overdue_years` | overdue activity retention in years | Integer | `3`, shipped | Age above which an overdue Activity is deleted. Zero or a missing value disables the routine with a warning; a negative value is rejected with a different warning. |
| `mail.activity.systray.limit` | activity indicator limit | Integer | 1000 | The maximum number of activities aggregated in the top-bar activity indicator. |
| `mail.web_push_vapid_public_key` | browser push public key | Text | generated at first use | The public half of the signing key pair announced to browsers. |
| `mail.web_push_vapid_private_key` | browser push private key | Text | generated at first use | The private half used to authenticate a push delivery. |
| `mail.chat_from_token` | allow chat from a token | Boolean as text | off | Allows opening or creating a Channel from a shared token link. |
| `mail.use_twilio_rtc_servers`, `mail.twilio_account_sid`, `mail.twilio_account_token` | external traversal server settings | see section 2.2 | empty | See section 2.2. |
| `mail.use_sfu_server`, `mail.sfu_server_url`, `mail.sfu_server_key` | forwarding unit settings | see section 2.2 | empty | See section 2.2. |
| `discuss.klipy_api_key` | animated image service key | Text | empty | See section 2.3. |
| `mail.google_translate_api_key` | translation service key | Text | empty | See section 2.3. |
| `bus.gc_retention_seconds` | event bus retention in seconds | Integer | 86400, that is one day | How long an event bus entry is kept before the garbage collector deletes it. |
| `sms.endpoint` | text message service address | Text | the platform's text message service address | The address of the sending service. Reproduced as part of the integration contract. |
| `sms.session.batch.size` | text message queue batch size | Integer | 500 | Text Messages processed by one queue run and split into one service call each. |
| `sms_twilio.session.batch.size` | external telephony batch size | Integer | 10 | Text Messages per call when the external telephony provider is used. |
| `snailmail.endpoint` | postal service address | Text | the platform's postal service address | The address of the printing service. |
| `snailmail.timeout` | postal service timeout | Integer | the service default in seconds | The timeout of one call to the printing service. |
| `web.base.url` | base address | Text | the address the installation answers on | Used to make a local link absolute and to build every access link. |

Four historic address parameters — the bounce local part, the catch-all local part, the catch-all domain and the default sender — are read exactly once, when the messaging package is installed on a database that previously carried only the platform foundation, in order to seed the first Alias Domain. They are never read again; the Alias Domain record is the single source of truth afterwards.

---

## 4. Fixed constants

These are not configurable. A rebuild must reproduce the values.

| Constant | Value | Meaning |
|---|---|---|
| Away threshold | 1800 seconds | Inactivity after which a Presence becomes "away". |
| Forwarding unit threshold | 3 participants | The number of call sessions at which a Channel switches from direct browser-to-browser connections to a forwarding unit. |
| Forwarding unit request token lifetime | 30 seconds | The lifetime of the token used to request a channel from the forwarding unit. |
| Forwarding unit session token lifetime | 8 hours | The lifetime of the token handed to each call session. |
| Channel bounce limit | 10 | The bounce counter at which a Contact is removed from a Channel. |
| External recipient header limit | 50 | Above this number of external addresses, the reply-to-all header is omitted. |
| Reply address length limit | 68 characters | 78, the header line limit, minus 10 for the header name and its separator. |
| Message preview length | 190 characters | The maximum length of the preview line, the ellipsis marker included. |
| Notification reference count | 3 ancestors plus the message itself | The size of the reference chain of a notification electronic mail. |
| Reference scan depth | 32 | The number of threading references examined when looking for a parent. |
| Flat-thread ancestor scan depth | 200 messages | The number of messages examined to find the ancestor of a flat thread. |
| Electronic mail size safety margin | 10240 bytes | Added to the estimated message size. |
| Transfer-encoding expansion factor | 8 ÷ 6 | Applied to every attachment size when estimating the message size. |
| Browser push direct-send threshold | 5 devices | Below it, payloads are pushed directly; from it, they are queued. |
| Out-of-office answer window | 4 days | A user does not answer the same correspondent twice inside this window. |
| Live chat assignment buffer | 120 seconds | The minimum gap between two conversations handed to one operator, relaxed when it would leave no candidate. |
| Live chat activity window | 30 minutes | The age below which a session counts towards an operator's load. |
| Live chat capacity window | 15 minutes | The age below which a session counts towards an operator's concurrent limit. |
| Live chat rating satisfaction window | 14 days | The period over which the satisfaction of a Live Chat Channel is computed. |
| Activity purge batch | 10000 | Activities deleted per run of the purge routine. |
| Notification retention | 180 days | The age above which a read, delivered or cancelled notification of a non-customer contact is deleted. |
| Sub-thread unpin idle period | 2 days | The idle period after which a member of a sub-thread is unpinned. |
| Guest call-invitation poll window | 12 hours | A guest is rung only if they polled inside this window. |
| Channel token length and alphabet | 10 characters from a 56-character alphabet | The lower-case letters without the one that resembles the digit one, the upper-case letters without the one that resembles the digit zero, and the digits two to nine. |
| Composer generation batch | 50 records | Records processed per rendering pass of the composer. |
| Activity schedule wizard batch | 500 records | Records processed per pass of the activity scheduling window. |
| Scheduled-message posting batch | 50 | Scheduled Messages posted per run. |
| Consent code lifetime | 3 minutes | The lifetime of the electronic-mail client consent code. |
| Application key lifetime | 1 day | The lifetime of the key exchanged for a consent code. |
| Enrichment logotype download timeout | 2 seconds | A failed download never fails the enrichment. |
| Postal page limit | 8 pages | The maximum number of pages the printing service accepts. |

---

## 5. Shipped master data

### 5.1 Message subtypes

| External identifier | Name | Default | Internal | Sequence | Track recipients | Purpose |
|---|---|---|---|---|---|---|
| `mail.mt_comment` | Discussions | yes | no | 0 | yes | The subtype of a public discussion message. Notifying the followers of a record uses it. |
| `mail.mt_note` | Note | no | yes | 100 | yes | The subtype of an internal log. Posting without an explicit subtype falls back to it. |
| `mail.mt_activities` | Activities | no | yes | 90 | no | The subtype of the message logged when an Activity is completed. |

None of the three carries a description, so a message that has one of them and an empty body is still void.

### 5.2 Activity types

| External identifier | Name | Default summary | Icon | Sequence | Delay | Category | Protection |
|---|---|---|---|---|---|---|---|
| `mail.mail_activity_data_todo` | To-Do | To-Do | check mark | 2 | 5 days | none | May never be archived or deleted; receives the activities of any deleted type; used by the reminder bar and the command palette. |
| `mail.mail_activity_data_email` | Email | Email | envelope | 3 | 0 days | none | — |
| `mail.mail_activity_data_call` | Call | Call | telephone | 6 | 2 days | phone call | May not be deleted; may not change its model. |
| `mail.mail_activity_data_meeting` | Meeting | Meeting | people | 9 | 0 days | none, extended to the meeting category when the calendar capability is present | May not be deleted; may not change its model. |
| `mail.mail_activity_data_upload_document` | Document | Document | upload | 25 | 5 days | upload document | May not change its model; may be deleted. |
| `mail.mail_activity_data_warning` | Exception | none | warning | 99 | 0 days | none | Shipped archived; decoration "Alert"; used by business flows to raise an alert on a record. May not change its model; may be deleted. |

### 5.3 Electronic mail templates

| External identifier | Name | Model | Purpose |
|---|---|---|---|
| `mail.mail_template_data_test` | Mail: Test Mail Template | User | Verifies the outgoing configuration: sent to the acting user's formatted address, from the company name and address, in the user's language, carrying a small test attachment. |
| `mail_group.mail_template_guidelines` | Mail Group: Send Guidelines | Mailing Group Member | Sent to a new subscriber of a list that publishes guidelines. Subject "Guidelines of group <list name>", default recipients, automatic deletion. |
| `mail_group.mail_template_list_subscribe` | Mail Group: Mailing List Subscription | Mailing Group | Confirms a subscription request. Subject "Confirm subscription to <list name>", automatic deletion. |
| `mail_group.mail_template_list_unsubscribe` | Mail Group: Mailing List Unsubscription | Mailing Group | Confirms an unsubscription request. Subject "Confirm unsubscription to <list name>", automatic deletion. |

One text-message template is shipped as a starting point, bound to the Contact model.

### 5.4 Notification layouts and shipped bodies

| External identifier | Kind | Content |
|---|---|---|
| `mail.mail_notification_layout` | layout | The standard notification layout: preview line, optional header with the access button and the subtitles, the message body, the tracking value list, the signature, and the optional footer with the company block, the credit line and the unfollow link. |
| `mail.mail_notification_light` | layout | The same without the header and without the access button. Used for customer-facing messages such as a rating request. |
| `mail.mail_notification_layout_with_responsible_signature` | layout | The standard layout, forcing the signature of the record's responsible instead of the author's. |
| `mail.mail_notification_invite` | layout | The follower-invitation layout: the subtitles are hidden when there is no access button, the body is replaced by the invitation sentence, and the unfollow block is moved above the footer rule. |
| `mail.message_activity_assigned` | body | The activity assignment notice. |
| `mail.message_activity_done` | body | The activity completion notice. |
| `mail.message_user_assigned` | body | The responsible assignment notice. |
| `mail.message_origin_link` | body | The "created from" notice linking a record to the records it came from. |
| `mail.message_notification_email` | wrapper | The wrapper that carries the preview line and the padding characters. |
| `mail.mail_bounce_catchall` | body | The bounce returned when a message is addressed to the catch-all mailbox. |
| `mail.mail_bounce_alias_security` | body | The wrapper of the two alias rejections; it appends the original message for reference. |
| `mail.mail_bounce_notification_limit` | body | The bounce returned when loop detection triggers. |
| `mail_group.mail_group_mail_rejected_not_open` | body | The bounce returned when a message is addressed to a closed mailing list. |
| `mail.mail_channel_invite` | body | The channel invitation body with the join button. |

The exact content of every one of these is specified in [interfaces.md](interfaces.md), section 7.

### 5.5 Shipped conversation records

| Record | Content |
|---|---|
| Channel `general` | Name "general", description "General announcements for all employees."; the internal-user group is its auto-subscription group, so every active internal user becomes a member. |
| Channel for administrators | Name "Administrators", description "General channel for administrators."; its authorization group and its auto-subscription group are both the settings-administrator group. |
| Welcome message | Posted in the general channel with the subject "Welcome to the system!" and a body inviting everybody to share company information there. |
| Assistant bot contact | The built-in bot contact with its picture. The root user's notification preference is the in-application inbox and the root user's onboarding state is `disabled`. |
| Canned Response `hello` | Substitution "Hello, how may I help you?", shared with the internal-user group. |

### 5.6 Shipped live chat records

| Record | Content |
|---|---|
| Live Chat Channel | Name taken from the site, welcome line "How may I help you?", linked to the default website. |
| Chatbot Script "Welcome Bot" | Nine steps: a welcome line, a question with three answers, and three branches. |

The shipped script, in sequence order:

| Sequence | Step type | Message | Shown only if |
|---|---|---|---|
| 1 | text | "Welcome to CompanyName! 👋" | — |
| 2 | question | "What are you looking for?" with the answers "I have a pricing question", "I am looking for your documentation" (which redirects to the site root) and "I am just looking around" | — |
| 3 | text | "Hmmm, let me check if I can find someone that could help you with that..." | the pricing answer |
| 4 | forward to operator | (no message) | the pricing answer |
| 5 | text | "Hu-ho, it looks like none of our operators are available 🙁" | the pricing answer |
| 6 | address question | "Would you mind leaving your email address so that we can reach you back?" | the pricing answer |
| 7 | text | "And tadaaaa here you go! 🌟" | the documentation answer |
| 8 | text | "If you need anything else, feel free to get back in touch" | the documentation answer |
| 9 | text | "Please do! If there is anything we can help with, let us know" | the "just looking around" answer |

### 5.7 Shipped digest records

One Digest is shipped per company, named after the company, with the periodicity "daily", the recipients left empty and the two indicators of this domain switched on. The live chat capability switches on its three additional indicators. Digest Tips are shipped as ordinary data records and may be added by any capability package.

### 5.8 External prepaid service registrations

| Service | Unit shown to the operator | Balance kind | Description shown |
|---|---|---|---|
| Text messages | Credits | fractional | "Send SMS Text Message to your contacts" |
| Postal mail | Stamps | whole units | "Send your invoices and follow-up reports by post" |

### 5.9 Sequences

This domain defines **no** numbering sequence. Every identifier it needs is either a database identifier, a random token (the channel token, the guest access token, the text message correlation token, the rating access token) or a generated electronic mail message identifier whose grammar is in [calculations.md](calculations.md), section 5. A rebuild therefore needs no sequence configuration for this folder.

---

## 6. Scheduled jobs

| Displayed name | Acts on | Interval | Priority | Purpose |
|---|---|---|---|---|
| "Mail: Email Queue Manager" | Outgoing Mail | every hour | 6 | Drains the outgoing queue in batches of one thousand. |
| "Mail: Fetchmail Service" | Incoming Mail Server | every 5 minutes, **inactive when shipped** | default | Polls every confirmed incoming server. Activated automatically the first time an incoming server is confirmed. |
| "Mail: Post scheduled messages" | Scheduled Message | every day | default | Posts every Scheduled Message whose moment has passed, acting as its creator. |
| "Notification: Notify scheduled messages" | Message Notification Schedule | every hour | default | Releases every deferred notification pass whose moment has passed. Also woken on demand for the exact moment of a new deferral. |
| "Notification: Delete Notifications older than 6 Months" | Notification | every month | default | Deletes read, delivered or cancelled notifications of non-customer contacts older than 180 days, in batches, reporting whether more remain. |
| "Mail: send web push notification" | Push Notification | every day | default | Delivers every queued browser push payload and deletes the rows. Also woken on demand when rows are created. |
| "Discuss: channel member unmute" | Channel Member | every day | default | Clears every mute moment that has passed and re-broadcasts the affected members. |
| "Mail List: Notify group moderators" | Mailing Group | every day | 1000 | Notifies every moderator of every moderated list that has pending posts, with the subject "Messages are pending moderation". |
| "SMS: SMS Queue Manager" | Text Message | every 24 hours | default | Drains the text message queue. |
| "Snailmail: process letters queue" | Postal Letter | every 24 hours | default | Drains the postal letter queue. |
| "Digest Emails" | Digest | every day, first run two hours after installation | default | Sends every activated Digest whose next run date has passed. |
| "Publisher: Update Notification" | the announcement exchange | every week, first run one week after installation | 1000 | Contacts the publisher announcement service and posts the announcements it returns in the company-wide channel. |

In addition, the platform's maintenance job runs the following collection routines of this domain: expired event bus entries; expired presence rows; call sessions whose heartbeat lapsed; old Message Translations; composer attachments that never became a message; Link Previews no message refers to; Text Messages marked for deletion; overdue Activities older than the configured retention; empty and bot-only live chat sessions; sub-thread pins that went idle; and read live chat sessions still pinned in an operator's sidebar.

---

## 7. Security groups

| Group | Implied by | What it allows |
|---|---|---|
| Mail Template Editor | the settings-administrator group | Creating and modifying a Template that contains anything other than a plain field path while the rendering restriction is on; resetting a shipped Template. |
| Canned Response Administrator | the settings-administrator group and the live-chat-administrator group | Reading and modifying every shared Canned Response. Belongs to the privilege category "Canned Responses". |
| Receive notifications in the inbox | — | Marks the users whose notification preference is the in-application inbox. The preference field and this group are kept consistent in both directions. |
| Live Chat / User | — | Joining a Live Chat Channel, taking conversations, reading every live chat session and every live chat membership, inviting anybody into a live chat session, managing conversation tags and display rules, reading the expertise list, using canned responses. Belongs to the privilege category "Live Chat". |
| Live Chat / Administrator | implies Live Chat / User and Canned Response Administrator | Everything above, plus creating, modifying and deleting entry points, chatbot scripts, steps, answers, expertise and reports, and deleting sessions. |
| Mailing Group Administrator | the settings-administrator group | Reading, creating, modifying and deleting every Mailing Group, member, post and moderation rule. |

The groups this domain leans on but does not define are the internal-user group, the portal group, the public group, the settings-administrator group and the configuration-manager group, all of which belong to [../identity-and-access/](../identity-and-access/).

---

## 8. Access rights

Rights are given per model and per group as the four operations read, write, create and delete. "Own" in the table means the rows a record rule narrows to; the record rules are in section 9.

| Model | Public | Portal | Internal user | Settings administrator |
|---|---|---|---|---|
| Message (`mail.message`) | read | read, write, create, delete | read, write, create, delete | all four |
| Message Subtype (`mail.message.subtype`) | read | read | read | all four |
| Follower (`mail.followers`) | — | — | read | all four |
| Notification (`mail.notification`) | — | read | read, write, create | all four |
| Tracking Value (`mail.tracking.value`) | — | — | — | all four (system group) |
| Message Reaction (`mail.message.reaction`) | — | — | — | all four (system group) |
| Link Preview and Message Link Preview | — | — | — | all four (configuration-manager group) |
| Message Translation (`mail.message.translation`) | — | — | — | all four |
| Message Notification Schedule (`mail.message.schedule`) | — | — | — | all four |
| Scheduled Message (`mail.scheduled.message`) | — | — | read, write, create, delete (own) | all four |
| Outgoing Mail (`mail.mail`) | — | — | — | all four |
| Alias (`mail.alias`) | — | — | read | all four |
| Alias Domain (`mail.alias.domain`) | — | — | read | all four (configuration-manager group) |
| Gateway Allowed Sender (`mail.gateway.allowed`) | — | — | — | all four |
| Blacklist Entry (`mail.blacklist`) | — | — | — | all four |
| Incoming Mail Server (`fetchmail.server`) | — | — | — | all four |
| Template (`mail.template`) | — | — | read, write, create, delete (own or unowned) | all four |
| Activity (`mail.activity`) | — | — | read, write, create, delete | all four |
| Activity Type (`mail.activity.type`) | — | — | read | all four |
| Activity Plan and Activity Plan Template | — | — | read | all four |
| Channel (`discuss.channel`) | read | read | read, write, create | all four |
| Channel Member (`discuss.channel.member`) | read, write, create, delete | read, write, create, delete | read, write, create, delete | all four |
| Call Session (`discuss.channel.rtc.session`) | — | — | — | all four |
| Call History (`discuss.call.history`) | read | read | read | all four |
| Guest (`mail.guest`) | — | — | read | all four |
| Presence (`mail.presence`) | — | — | — | all four |
| Interactive Connectivity Server (`mail.ice.server`) | — | — | — | all four |
| Voice Metadata (`discuss.voice.metadata`) | — | — | — | all four |
| Favorite Animated Image (`discuss.gif.favorite`) | — | — | read, write, create, delete (own) | all four (configuration-manager group) |
| Canned Response (`mail.canned.response`) | — | — | read, write, create, delete (own or shared) | all four |
| Role (`res.role`) | — | — | read | all four (configuration-manager group) |
| Push Device and Push Notification | — | — | — | all four |
| Event Bus Entry (`bus.bus`) | — | — | — | all four |
| Text Message (`sms.sms`) | — | — | — | all four |
| Text Message Template (`sms.template`) | — | — | read | all four |
| Text Message Tracker (`sms.tracker`) | — | — | — | all four |
| Provider Number (`sms.twilio.number`) | — | — | read | all four |
| Postal Letter (`snailmail.letter`) | — | — | read, write, create | all four |
| Digest (`digest.digest`) | — | — | read | all four |
| Digest Tip (`digest.tip`) | — | — | read | all four |
| Mailing Group (`mail.group`) | read | read | read, write, create, delete (within the visibility rule) | all four (mailing-group administrator) |
| Mailing Group Member (`mail.group.member`) | — | — | all four for a moderator of the list | all four |
| Mailing Group Message (`mail.group.message`) | read (accepted only) | read (accepted only) | read, write, create, delete (within the visibility rule) | all four |
| Mailing Group Moderation Rule (`mail.group.moderation`) | — | — | all four for a moderator of the list | all four |
| Live Chat Channel (`im_livechat.channel`) | — | — | read for a live chat user | all four for a live chat administrator |
| Live Chat Rule (`im_livechat.channel.rule`) | — | — | read, write, create for a live chat user | all four for a live chat administrator |
| Live Chat Expertise (`im_livechat.expertise`) | — | — | read | all four for a live chat administrator |
| Conversation Tag (`im_livechat.conversation.tag`) | — | — | read, write, create for a live chat user | all four for a live chat administrator |
| Live Chat Member History (`im_livechat.channel.member.history`) | — | — | read for a live chat user | all four for a live chat administrator |
| Live Chat Session Report (`im_livechat.report.channel`) | — | — | — | read for a live chat administrator |
| Chatbot Script, Step, Answer and Message | — | — | — | all four for a live chat administrator |

Every wizard is readable, writable and creatable by the group that uses it: internal users for the two composers, the follower editor, the template previews and the activity scheduling window; settings administrators for the suppression-list removals and the text-message account windows; template editors for the two template reset windows; mailing-group administrators and moderators for the rejection window.

---

## 9. Record rules

| Model | Rule | Groups | Condition |
|---|---|---|---|
| Message | the five-branch policy | all | Not a record rule but a policy applied on top of the ordinary layer; see [business-rules.md](business-rules.md), section 1. |
| Activity | own activities | internal user | Write and delete only, on activities assigned to the acting user or created by them. Read and create are governed entirely by the policy of [business-rules.md](business-rules.md), section 2. |
| Scheduled Message | own entries | internal user | Read is granted broadly and narrowed by the search override to the rows whose target record the acting user may act on; write is restricted to the rows the acting user created. |
| Channel | accessible channels | internal, portal and public users | The type is not the plain channel type **and** the acting party is a member or a member of the parent channel; **or** the type is the plain channel type **and** the channel has no authorization group, or the acting user belongs to that group. |
| Channel | full access | system group | Everything. |
| Channel | live chat supervision | live chat user | Read-only access to every live chat session. |
| Channel Member | own entries | internal, portal and public users | Write and delete on the acting party's own row, under the same visibility condition as the channel. |
| Channel Member | read members | internal, portal and public users | Read on the rows of a channel the acting party can access. |
| Channel Member | join a channel | internal, portal and public users | Create the acting party's own row when the type is the plain channel type and the authorization group is empty or satisfied. |
| Channel Member | invite into a channel | internal user | Create somebody else's row when the type is the plain channel type and the authorization group is empty or satisfied. |
| Channel Member | invite into a private conversation | internal user | Create somebody else's row when the type is neither the plain channel type nor the chat type and the acting party is a member. |
| Channel Member | live chat | live chat user | Read every live chat membership and create any membership in a live chat session. |
| Call History | accessible calls | internal, portal and public users | Read the call history of a channel the acting party can access. |
| Call History | live chat | live chat user | Read every live chat call history. |
| Template | own templates | internal user | Write and delete on templates the acting user created or that name them as owner; a template editor and an administrator may write all of them. |
| Canned Response | own and shared | internal user | Read the responses the acting user created plus those shared with a group they belong to; modify and delete only their own. |
| Favorite Animated Image | own entries | internal user | Only the rows the acting user created. |
| Mailing Group | visible groups | all | Public lists to everyone; member-only lists to their members; group-restricted lists to the members of the authorization group. |
| Mailing Group Message | visible posts | public and portal users | Accepted posts of a visible list only. A moderator additionally reads the pending and rejected posts of the lists they moderate. |
| Rating | internal only | internal user | Read, write and create; public and portal users reach a rating only through its access token. |

---

## 10. Master-data prerequisites

Before the domain can operate, all of the following must exist.

1. **At least one Alias Domain**, with a domain name, a bounce local part and a catch-all local part. Without one, an outgoing electronic mail has no return path, a reply cannot be threaded back, and the gateway cannot bounce.
2. **A Company bound to that Alias Domain.** A company without one falls back to the first Alias Domain by ordering; a database with none breaks reply addressing.
3. **At least one Outgoing Mail Server**, or a relay configured outside the platform. Without one, the queue moves every row to the failure state with the relay-failure type.
4. **An Incoming Mail Server, or a relay that pipes messages into the gateway**, when incoming electronic mail is wanted.
5. **The three shipped Message Subtypes.** They are referenced by external identifier throughout the domain and posting breaks without them.
6. **The six shipped Activity Types**, of which three may not be deleted and one may not even be archived.
7. **The assistant bot contact and the root user's inbox preference**, so that a message created by the gateway has a coherent author.
8. For text messages: a registered sending account with the platform's own service, or telephony credentials and at least one sending number with the external provider.
9. For postal mail: a registered postal account with credits, and a document layout whose paper format the printing service accepts.
10. For live chat: at least one Live Chat Channel with either one operator or one Chatbot Script on one of its rules.
11. For browser push: the signing key pair, generated at first use and stored in the two system parameters.

---

## 11. What a thread-enabled model must declare

A model that adopts the Thread behavior chooses the following. Everything has a default, so a model may adopt the behavior and declare nothing.

| Declaration | Default | Effect |
|---|---|---|
| flat thread | true | A message posted without a parent is attached to the first relevant ancestor of the record. |
| post access | `write` | The permission required on the record before a Message may be created on it. |
| customer tie | false | The record's main customer is subscribed when it appears among the direct recipients of a posted message. |
| primary address field | `email` | The field holding the main electronic mail address, used by the suppression list, by record creation from an incoming message and by loop detection. |
| external recipient limit | 50 | Above it, external addresses are not listed in the reply-to-all header. |
| number fields | none | The fields scanned for a telephone number, in preference order. |
| country field | `country_id` when it exists | The country used to interpret a telephone number. |
| customer fields | the fields linking to Contact | Where the record's customer is found for default recipients and for country inference. |
| creation subtype | none | The subtype of the message logged at creation. When there is none, the creation sentence is logged as an internal note. |
| creation sentence | "<document type name> created" | The body of that message. |
| tracked fields | none | The fields whose change produces a Tracking Value, each with its display order number. |
| tracking subtype rule | returns nothing | Maps a set of changed values to a subtype, so a stage change notifies the right followers. |
| tracking template rule | returns nothing | Maps a set of changed values to a Template to post automatically. |
| responsible field | `user_id` when it is tracked | Drives the automatic subscription and the assignment notification. |
| duration-tracking field | none | The tracked link field whose per-value durations are measured. |
| staleness condition | none | The condition a record must satisfy before staleness is evaluated. |
| loop-detection filter | the primary address field contains the sender | How the gateway counts the records this sender already created. |
| customer information map | empty | Maps a normalized address to the initial values of a Contact created for it. |

The three flags on the model registry entry (`is_mail_thread`, `is_mail_activity` and `is_mail_blacklist`) record which behaviors a model adopted. Only a custom model may have them changed — "Only custom models can be modified." — and none of the three may be switched off once it is on: "Field "Mail Thread" cannot be changed to "False"." for the thread flag, and the equivalent sentences naming "Mail Activity" and "Mail Blacklist" for the other two.

---

## Reconciliation notes

1. **The name of the text message queue job.** One source version named the job "Text Message: Text Message Queue Manager"; the shipped record is labelled "SMS: SMS Queue Manager". The shipped label is reproduced here, in quotation marks, because an operator searching the job list keys on it.
2. **The digest sending job.** One source version listed no digest job at all and the other named it differently; the shipped record is labelled "Digest Emails", runs every day and is first scheduled two hours after installation. That is what is stated.
3. **The default text-message provider value.** One source version wrote the stored value as `platform_service`; the value written to the database is `odoo`, with `twilio` selecting the external provider. The stored values are reproduced, since an integration reads them.
4. **The message preview length.** One source version stated 100 characters. The implementation shortens to 190 characters including the ellipsis marker, while an explanatory comment in the same place still says 100. 190 is the observable behaviour and is what section 4 states; the discrepancy is recorded as a **compatibility finding** in [calculations.md](calculations.md), section 30.
5. **The mailing-group relay batch size.** One source version gave the parameter a mailing-group-specific default. The single parameter `mail.session.batch.size` serves both the transport session (default 1000) and the mailing-group relay (default 500 in the relay code path); both defaults are stated in section 3.
