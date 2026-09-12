# Messaging and Activities

## Scope

This domain specifies the communication backbone of the system: the mechanism by which **any** business record acquires a conversation history, a list of interested parties, a change log, a set of scheduled follow-up tasks and a set of outbound and inbound electronic messages.

Almost every other domain in this repository depends on it. A customer invoice logs its validation in a thread; a sales order notifies its salesperson when it is confirmed; a manufacturing order carries an activity "check the components"; a purchase agreement is created from an electronic mail sent to a mailbox address; a payment failure raises a delivery-error counter on a contact. All of that behavior is defined here, once, and inherited by the other domains through a small number of reusable behaviors (called *mixins* in this document, and always named in full).

The domain covers six connected subjects:

1. **The thread behavior** — posting, the computation of who must be notified, the delivery of those notifications by in-application inbox, electronic mail and browser push, message subtypes, automatic subscription, field change tracking, duration tracking, and the access policy that governs who may read or write a message.
2. **Messages and their parts** — the message record itself, its author, its recipients, its attachments, its reactions, its link previews, its translations, its scheduling, its pinning, its starring and its delivery notifications.
3. **Activities** — the to-do items attached to records, their types, their chaining, their plans, their deadline state computation, and the wizard that schedules a whole plan at once.
4. **Electronic mail plumbing** — the outgoing queue, the outgoing mail servers, the alias domains (catch-all, bounce and default-from local parts), the incoming servers, the complete incoming routing algorithm with bounce handling and loop prevention, the templates and the rendering contract, the blacklist and the opt-out.
5. **Real-time conversation** — channels and their members, the unread model, guests, typing indicators, call sessions, commands, mentions, presence, the event bus that pushes all of it to connected clients, and the data contracts the client loads.
6. **Adjacent delivery channels and satellites** — live chat with operator assignment and chatbot scripts, the assistant bot script, mailing groups with moderation, the electronic-mail client plugin contract, text messages with their own queue and blacklist, periodic digests with every shipped indicator, and postal mail.

## What is in scope and what is not

In scope:

| Subject | Where specified |
|---|---|
| Thread behavior: posting, logging, notifying, the recipient computation query, the notification groups, the electronic mail layout rendering, the reply-to computation, the message identifier grammar | `entities.md`, `workflows.md`, `calculations.md`, `business-rules.md` |
| Subtypes, their parent relationship, automatic subscription, default and internal and external subtype sets | `entities.md`, `calculations.md`, `workflows.md` |
| Followers: creation, subtype editing, removal, the four existing-follower policies | `entities.md`, `workflows.md`, `business-rules.md` |
| Field change tracking: which fields, the comparison, the stored old and new values per type, the ordering, the subtype selection, the template posting, the duration tracking and the rotting computation | `calculations.md`, `entities.md`, `workflows.md` |
| Messages: every field, the message types, the access policy with its five branches, editing, deleting, reactions, link previews, translations, pinning, starring, scheduling | `entities.md`, `business-rules.md`, `state-machines.md` |
| Notifications: the two base delivery types and the two added by satellites, their statuses, their failure types, the failure reporting to the author | `entities.md`, `state-machines.md` |
| Activities: types, categories, chaining, deadline computation, the state derivation, completion and its message, plans and plan templates, the scheduling wizard | `entities.md`, `state-machines.md`, `calculations.md`, `workflows.md` |
| Outgoing electronic mail: the queue record, the send algorithm, the batching, the failure classification, the automatic deletion, the scheduled date parsing | `state-machines.md`, `workflows.md`, `business-rules.md` |
| Alias domains, aliases, the alias behavior mixins, the catch-all, the bounce address, the default-from address | `entities.md`, `configuration.md`, `workflows.md` |
| Incoming routing: parsing, bounce detection and propagation, reply detection, alias matching, the contact-security check, loop detection, record creation and update, the exact bounce bodies | `workflows.md`, `calculations.md`, `business-rules.md` |
| Templates: fields, the rendering engines, the placeholder contract, the recipient generation, the report attachment generation, the editor restriction, the reset behavior | `entities.md`, `calculations.md`, `business-rules.md`, `interfaces.md` |
| The composer: its two modes, every computed default, the batch behavior, the send algorithm | `entities.md`, `workflows.md` |
| Blacklist, opt-out, bounce counter, the normalized address computation | `entities.md`, `calculations.md`, `business-rules.md` |
| Channels: the three base kinds plus the live chat kind, membership, the unread counters, pinning, muting, sub-channels, invitations, the authorized group, the automatic subscription group | `entities.md`, `calculations.md`, `workflows.md`, `business-rules.md` |
| Guests, presence, typing, call sessions, call history, the selective-forwarding-unit contract | `entities.md`, `interfaces.md`, `configuration.md` |
| The event bus: channel naming, the poll contract, the garbage collection, the websocket subscription | `interfaces.md`, `configuration.md` |
| The client data contract (the store): what each record type sends to the browser | `interfaces.md` |
| Live chat: channels, rules, operator assignment with its exact ranking, chatbot scripts and steps and answers, expertise, the session outcome, the reports | `entities.md`, `calculations.md`, `workflows.md`, `state-machines.md` |
| The assistant bot conversation script and its onboarding state machine | `state-machines.md`, `workflows.md` |
| Mailing groups: members, messages, moderation rules, the moderation state machine, the public pages | `entities.md`, `state-machines.md`, `workflows.md` |
| The electronic-mail client plugin contract: authentication, enrichment, record creation and logging | `interfaces.md`, `workflows.md` |
| Text messages: the queue, the templates, the composer, the trackers, the delivery reports, the number sanitizing, the blacklist, the two providers | `entities.md`, `state-machines.md`, `workflows.md`, `calculations.md` |
| Digests: periodicity, the indicator set, the computation of each indicator, the comparison with the previous period, the sending algorithm, the unsubscribe | `entities.md`, `calculations.md`, `workflows.md` |
| Postal mail: the letter record, its state machine, the address validation, the pricing call, the error codes | `entities.md`, `state-machines.md`, `workflows.md` |
| User notification settings, canned responses, roles and role mentions, voice metadata, favorite animated images | `entities.md`, `configuration.md` |
| Every access right, record rule, security group, system parameter, scheduled job and route of the domain | `configuration.md`, `interfaces.md` |

Out of scope, specified in a neighbouring folder:

| Subject | Folder |
|---|---|
| Contacts themselves (their names, addresses, company hierarchy, commercial entity) | `../contacts-and-organizations/` |
| Users, groups as an access-control concept, authentication, the portal access token | `../identity-and-access/` |
| Automated actions and server actions in general (this folder describes only the four message- and activity-related server-action kinds) | `../automation-and-integration/` |
| Meetings and calendar events, including the meeting activity category | `../calendar-and-scheduling/` |
| Ratings as a generic behavior (this folder describes only how live chat uses them) | `../learning-surveys-and-gamification/` |
| Mass mailing campaigns, mailing lists used for marketing, and their statistics | `../marketing-and-mass-mailing/` |
| The accounting and stock consequences of the documents that use this domain | the respective accounting and supply-chain folders |

## Entities

The domain defines the following records. Names are given in full words; the transport name (the identifier used by the remote interface) and the storage name (the database table) follow.

### The thread and its parts

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Thread (abstract behavior) | `mail.thread` | none | Gives a record a conversation, followers, tracking and notification behavior |
| Message | `mail.message` | `mail_message` | One entry of a conversation: body, author, recipients, type, subtype |
| Message Subtype | `mail.message.subtype` | `mail_message_subtype` | A named category of message used to filter who gets notified |
| Follower | `mail.followers` | `mail_followers` | One party subscribed to one record, with the subtypes they follow |
| Notification | `mail.notification` | `mail_notification` | One delivery of one message to one party, with its channel and its status |
| Tracking Value | `mail.tracking.value` | `mail_tracking_value` | One recorded change of one tracked field, attached to a message |
| Message Reaction | `mail.message.reaction` | `mail_message_reaction` | One emoji placed on one message by one party |
| Link Preview | `mail.link.preview` | `mail_link_preview` | The cached preview metadata of one web address |
| Message Link Preview | `mail.message.link.preview` | `mail_message_link_preview` | The attachment of one link preview to one message, with its order and hidden flag |
| Message Translation | `mail.message.translation` | `mail_message_translation` | The cached machine translation of one message body into one language |
| Message Notification Schedule | `mail.message.schedule` | `mail_message_schedule` | A posted message whose notifications are delayed until a date |
| Scheduled Message | `mail.scheduled.message` | `mail_scheduled_message` | A message not yet posted, held until a date with its full posting payload |
| Blacklist Entry | `mail.blacklist` | `mail_blacklist` | One electronic mail address that must never receive mass messages |
| Gateway Allowed Sender | `mail.gateway.allowed` | `mail_gateway_allowed` | One sender address exempted from the incoming loop quota |
| Canned Response | `mail.canned.response` | `mail_canned_response` | A shortcut that expands into a longer text while composing |
| Role | `res.role` | `res_role` | A named set of users that can be mentioned as a group |
| Blacklist behavior (abstract) | `mail.thread.blacklist` | none | Adds the normalized address, the blacklist flag and the bounce counter to a thread |
| Carbon-copy behavior (abstract) | `mail.thread.cc` | none | Keeps the carbon-copy list of an incoming conversation on the record |
| Main attachment behavior (abstract) | `mail.thread.main.attachment` | none | Designates one attachment of the thread as the one to preview |
| Duration tracking behavior (abstract) | `mail.tracking.duration.mixin` | none | Computes how long a record stayed in each value of one link field, and whether it is stale |

### Activities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Activity | `mail.activity` | `mail_activity` | One to-do attached to one record, with a deadline and an assignee |
| Activity Type | `mail.activity.type` | `mail_activity_type` | The category of an activity: default delay, chaining, category, templates |
| Activity Plan | `mail.activity.plan` | `mail_activity_plan` | A named set of activities to launch together on one record |
| Activity Plan Template | `mail.activity.plan.template` | `mail_activity_plan_template` | One line of a plan: type, delay, assignee rule |
| Activity behavior (abstract) | `mail.activity.mixin` | none | Gives a record its activity list and its derived activity state |
| Activity Schedule wizard | `mail.activity.schedule` | transient | Collects the parameters and launches one activity or one plan |
| Activity Schedule Line | `mail.activity.schedule.line` | transient | One preview row of the wizard |

### Electronic mail plumbing

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Outgoing Mail | `mail.mail` | `mail_mail` | One queued electronic mail to send, wrapping one message |
| Alias | `mail.alias` | `mail_alias` | One inbound address that routes to a model, with its security policy |
| Alias Domain | `mail.alias.domain` | `mail_alias_domain` | One electronic mail domain with its catch-all, bounce and default-from local parts |
| Alias behavior, required (abstract) | `mail.alias.mixin` | none | Gives a record exactly one alias, created with it |
| Alias behavior, optional (abstract) | `mail.alias.mixin.optional` | none | Gives a record an alias created on demand when a name is set |
| Incoming Mail Server | `fetchmail.server` | `fetchmail_server` | One mailbox to poll for incoming messages |
| Outgoing Mail Server | `ir.mail_server` | `ir_mail_server` | One relay used to send, with its owner restriction (extended here) |
| Template | `mail.template` | `mail_template` | A reusable message with placeholders, recipients and attached reports |
| Render behavior (abstract) | `mail.render.mixin` | none | The placeholder rendering contract and its two engines |
| Composer behavior (abstract) | `mail.composer.mixin` | none | Subject and body derived from a template, with the editor restriction |
| Template reset behavior (abstract) | `template.reset.mixin` | none | Restores a template to its shipped definition |
| Composer | `mail.compose.message` | transient | The message composition window in its two modes |
| Template Preview | `mail.template.preview` | transient | Renders a template against a chosen record |
| Template Reset wizard | `mail.template.reset` | transient | Resets a selection of templates |
| Followers Edit wizard | `mail.followers.edit` | transient | Adds or removes followers in bulk, optionally notifying them |
| Blacklist Removal wizard | `mail.blacklist.remove` | transient | Removes an address from the blacklist with a reason |

### Push and presence

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Push Device | `mail.push.device` | `mail_push_device` | One browser subscription endpoint of one party |
| Push Notification | `mail.push` | `mail_push` | One queued browser push payload for one device |
| Presence | `mail.presence` | `mail_presence` | The online, away or offline status of one user or one guest |
| Event Bus Entry | `bus.bus` | `bus_bus` | One broadcast payload on one named channel |
| Bus sender behavior (abstract) | `bus.listener.mixin` | none | Lets a record be used as a broadcast channel |

### Conversation channels

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Channel | `discuss.channel` | `discuss_channel` | A multi-party conversation: channel, group, direct chat or live chat session |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | One party inside one channel, with the unread markers and the mute settings |
| Call Session | `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | One live audio or video participation of one member |
| Call History | `discuss.call.history` | `discuss_call_history` | One completed or ongoing call in one channel with its duration |
| Guest | `mail.guest` | `mail_guest` | An unauthenticated participant identified by a token |
| Voice Metadata | `discuss.voice.metadata` | `discuss_voice_metadata` | Marks one attachment as a voice recording |
| Favorite Animated Image | `discuss.gif.favorite` | `discuss_gif_favorite` | One saved animated image of one user |
| Interactive Connectivity Server | `mail.ice.server` | `mail_ice_server` | One traversal server used to establish calls |
| User Settings | `res.users.settings` | `res_users_settings` | Per-user conversation preferences (extended here) |
| User Settings Volume | `res.users.settings.volumes` | `res_users_settings_volumes` | The playback volume of one other party for one user |

### Live chat

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Live Chat Channel | `im_livechat.channel` | `im_livechat_channel` | One public entry point with its operators, its look and its capacity |
| Live Chat Rule | `im_livechat.channel.rule` | `im_livechat_channel_rule` | One condition that decides how the button behaves on one page |
| Live Chat Member History | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | The permanent trace of one participant of one session, for reporting |
| Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise` | One skill tag used to prioritize operators |
| Live Chat Conversation Tag | `im_livechat.conversation.tag` | `im_livechat_conversation_tag` | One label placed on a finished session |
| Chatbot Script | `chatbot.script` | `chatbot_script` | One automated conversation with its operator identity |
| Chatbot Script Step | `chatbot.script.step` | `chatbot_script_step` | One message or question of a script |
| Chatbot Script Answer | `chatbot.script.answer` | `chatbot_script_answer` | One proposed answer of a question step |
| Chatbot Message | `chatbot.message` | `chatbot_message` | The link between one posted message, one step and the answer given |
| Live Chat Session Report | `im_livechat.report.channel` | database view | The aggregated reporting rows over sessions |

### Mailing groups

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Mailing Group | `mail.group` | `mail_group` | A public discussion list fed by an inbound address |
| Mailing Group Member | `mail.group.member` | `mail_group_member` | One subscriber of a list, by contact or by bare address |
| Mailing Group Message | `mail.group.message` | `mail_group_message` | One post of a list, with its moderation status and its thread position |
| Mailing Group Moderation Rule | `mail.group.moderation` | `mail_group_moderation` | A permanent allow or ban decision for one address in one list |
| Mailing Group Rejection wizard | `mail.group.message.reject` | transient | Rejects or bans a pending post with an explanation |

### Text messages

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Text Message | `sms.sms` | `sms_sms` | One queued text message with its number, its body and its state |
| Text Message Template | `sms.template` | `sms_template` | A reusable text message body with placeholders |
| Text Message Tracker | `sms.tracker` | `sms_tracker` | The link between one provider identifier and one notification |
| Text Message Composer | `sms.composer` | transient | The text message composition window in its three modes |
| Text Message Template Preview | `sms.template.preview` | transient | Renders a text message template against a record |
| Text Message Template Reset | `sms.template.reset` | transient | Restores shipped text message templates |
| Provider Number | `sms.twilio.number` | `sms_twilio_number` | One sending number of the external provider, per country |
| Provider Account wizards | `sms.account.code`, `sms.account.phone`, `sms.account.sender`, `sms.twilio.account.manage` | transient | Registration and verification of the sending account |

### Digests and postal mail

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Digest | `digest.digest` | `digest_digest` | A periodic summary with a selected set of indicators and a recipient list |
| Digest Tip | `digest.tip` | `digest_tip` | One rotating hint shown at the bottom of a digest |
| Postal Letter | `snailmail.letter` | `snailmail_letter` | One document to print and post, with its address and its state |

### Satellite behaviors on existing records

| Entity | Transport name | Purpose in this domain |
|---|---|---|
| Company | `res.company` | The alias domain, the bounce and catch-all addresses, the notification colors, the postal options, the text-message provider |
| Contact | `res.partner` | The notification preferences, the tracked fields, the presence status, the channels, the bounce counter |
| User | `res.users` | The notification channel choice, the out-of-office responder, the manual presence, the assistant-bot state, the live chat identity and skills |
| Model registry entry | `ir.model` | The three flags that say whether a model has a thread, activities or a blacklist |
| Field registry entry | `ir.model.fields` | The tracking order number of a field |
| Server Action | `ir.actions.server` | The four message- and activity-related action kinds |
| Scheduled Job | `ir.cron` | Made a thread with activities so failures are logged and assigned |
| Attachment | `ir.attachment` | The thumbnail, the voice marker and the broadcast on deletion |
| Websocket handler | `ir.websocket` | The list of broadcast channels a client may subscribe to, the presence update and the presence clearing on disconnection |
| Report definition | `ir.actions.report` | The retrieval of a generated document and the page-format check used by postal mail |
| User Settings | `res.users.settings` | The conversation preferences, the push-to-talk key, the per-correspondent volumes and the live chat identity |

### Records used here but owned elsewhere

| Record | Transport name | Owning folder | What this folder relies on |
|---|---|---|---|
| Rating | `rating.rating` | [../learning-surveys-and-gamification/](../learning-surveys-and-gamification/) | The satisfaction answer a live chat session collects, its token, its value, its grade and its roll-up to the entry point. Specified here only to that extent, in `entities.md`, section 48.3. |
| Contact | `res.partner` | [../contacts-and-organizations/](../contacts-and-organizations/) | The universal author, recipient and follower. |
| User, Access Group | `res.users` | [../identity-and-access/](../identity-and-access/) | The identity behind a contact and the groups every access rule is written against. |
| Attachment, Model registry entry, Field registry entry, Server Action, Scheduled Job, Report definition, Websocket handler, Window action | various | [../platform-foundation/](../platform-foundation/) | The files, the registry flags, the automation kinds and the transport this folder extends. |
| Website, Website Visitor, Website Page | various | [../website-and-storefront/](../website-and-storefront/) | The site that serves the live chat widget and the visitor a session is attached to. |
| Calendar Event | `calendar.event` | [../calendar-and-scheduling/](../calendar-and-scheduling/) | The meeting an activity of the meeting category opens. |

Two records have no other home and are therefore owned here even though a neighbouring folder also reads them: the **Blocked Number** (`phone.blacklist`), which the text-message channel cannot work without and which [../marketing-and-mass-mailing/](../marketing-and-mass-mailing/) also consults, and the **Contact Enrichment** side table (`res.partner.iap`), which exists only for the electronic-mail client plugin of this folder.

Entities whose transport name begins with `ir.`, `base.` or `report.` belong to the platform foundation; this folder specifies only the fields it adds to them, and the reader is sent to [../platform-foundation/](../platform-foundation/) for the rest.

## Actors

| Actor | What they do in this domain |
|---|---|
| Internal user | Posts messages and internal notes, schedules activities, follows records, manages the canned responses and templates they created, operates channels. |
| Portal user | An external party with a login. Sees only the non-internal messages of the records shared with them, and may comment on a record they follow. |
| Public visitor | An unauthenticated web visitor. May open a live chat session and act as a Guest, and may read a public mailing-list archive. |
| Guest | A named, token-identified participant of a Channel with no user account. |
| Live chat operator | An internal user who is a member of one or more Live Chat Channels and is assigned conversations. |
| Live chat administrator | Configures entry points, chatbot scripts, expertise, tags and the reports. |
| Moderator | Accepts, rejects, whitelists and bans on a moderated Mailing Group. |
| Settings administrator | Configures alias domains, incoming and outgoing relays, activity types, message subtypes, the template-rendering restriction, the traversal and forwarding servers, the suppression lists and the sending accounts. |
| The platform's scheduler | Drains the outgoing queue, the text-message queue and the postal queue; releases deferred notifications; posts scheduled messages; ships digests; notifies moderators; collects what has expired. |
| External services | The relay, the mailbox, the browser push service, the text-message service, the telephony provider, the printing service, the enrichment service, the animated-image service, the translation service, the traversal servers, the forwarding unit and the publisher announcement service. |

## Capability packages

The domain ships as seventeen capability packages, listed with what each one adds in [configuration.md](configuration.md), section 1. The core is the notification bus plus the discussion and messaging package; everything else — the assistant bot, mailing groups and their public pages, follow buttons on public pages, live chat and its website widget, text messages and the external telephony provider, text messages on calendar events, postal mail and its accounting bridge, prepaid service notices, the periodic digest and the electronic-mail client plugin — is optional and is marked as such wherever it appears.

## Reading order

1. [glossary.md](glossary.md) — the vocabulary: thread, subtype, follower, notification, tracking value, alias, catch-all, bounce, activity state, channel member, unread separator, operator assignment.
2. [entities.md](entities.md) — the data model, field by field, entity by entity, with a link to the generated reference page of every entity.
3. [state-machines.md](state-machines.md) — the outgoing mail states, the notification states, the activity state derivation, the moderation states, the postal letter states, the live chat session states, the assistant-bot onboarding states, the suppression-list activation, the rating consumption and the sending-account registration.
4. [calculations.md](calculations.md) — the recipient computation, the tracking comparison, the deadline arithmetic, the unread counters, the operator ranking, the digest indicators, the number formatting, the rating aggregation, the rendering.
5. [business-rules.md](business-rules.md) — every constraint, every access check, every exact error message, numbered `MSG-nnn`, with the mapping from the identifiers the two source versions used.
6. [workflows.md](workflows.md) — posting, notifying, sending, receiving, bouncing, scheduling an activity, running a plan, opening a channel, routing a live chat visitor, moderating a list post, shipping a digest, registering a sending account, exchanging with the announcement service.
7. [accounting-effects.md](accounting-effects.md) — why this domain posts nothing to the ledger, and what it does instead.
8. [configuration.md](configuration.md) — capability packages, settings, system parameters, fixed constants, shipped records, scheduled jobs, groups, access rights, record rules and the declarations a thread-enabled model must make.
9. [interfaces.md](interfaces.md) — named operations, routes, the event bus, the client data contract, menus and screens, printable documents, shipped bodies, external services, the electronic-mail client plugin contract, import and export.
10. [acceptance-criteria.md](acceptance-criteria.md) — the numbered scenarios an implementation must pass.

## Files in this folder

| File | Contents |
|---|---|
| [README.md](README.md) | This page: scope, entities, actors, reading order, dependencies and conventions. |
| [entities.md](entities.md) | Every entity in full: purpose, lifecycle, complete field table, relations, uniqueness, ordering, display name, archival, company behaviour and the extensions other capability packages contribute. |
| [state-machines.md](state-machines.md) | Every state field with its stored values, its transition table, its guards and a diagram. |
| [workflows.md](workflows.md) | The end-to-end operational procedures, step by step, with the records each step touches and the failures it can produce. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with the exact refusal texts, plus the identifier mapping. |
| [calculations.md](calculations.md) | Every formula and algorithm with its rounding, its precision and at least one worked numeric example. |
| [accounting-effects.md](accounting-effects.md) | The reasoned statement that the domain posts nothing to the ledger, and the seven points at which it touches the accounting folders. |
| [configuration.md](configuration.md) | Settings, parameters, constants, master data, jobs, groups, rights and rules. |
| [interfaces.md](interfaces.md) | Operations, routes, the event bus, the client data contract, screens, documents and external services. |
| [acceptance-criteria.md](acceptance-criteria.md) | The Given, When and Then scenarios. |
| [glossary.md](glossary.md) | Every term of the domain, defined. |

## Dependencies on other domains

| Domain | Dependency |
|---|---|
| `../contacts-and-organizations/` | Every recipient, author and follower is a Contact. The normalized address, the formatted address, the commercial entity and the language of a Contact are used throughout. |
| `../identity-and-access/` | Users, their groups, their companies and their language. The notification channel choice is a field of the User. The access rules of this domain are written against the shipped groups. |
| `../automation-and-integration/` | Server actions may post a message, add followers or schedule an activity. Automated rules fire on tracked field changes. |
| `../calendar-and-scheduling/` | An activity whose type category is "meeting" opens a calendar view and is completed when the meeting is over. |
| `../learning-surveys-and-gamification/` | A live chat session asks the visitor for a rating; the rating record belongs to that domain. |
| All business domains | Every document model that carries a conversation inherits the thread behavior specified here, and every tracked field on such a model produces the tracking entries specified here. |

## Conventions used in this folder

- **Party** means a Contact record used as an author, a recipient or a follower. Where a distinction matters between an internal user, a portal user and a bare contact, the document says so explicitly.
- **Address** without qualification means an electronic mail address. A *normalized address* is the lower-case `local-part@domain` form with any display name removed; a *formatted address* is the `"Display Name" <local-part@domain>` form.
- **Thread-enabled model** means a model that inherits the thread behavior; **activity-enabled model** means a model that inherits the activity behavior.
- Dates and times are stored without a time zone and are understood as coordinated universal time unless the text says otherwise. Deadlines of activities are calendar dates without a time component and are compared in the time zone of the assignee.
- Identifiers reproduced in code font (for example `partner_id`, `mail.message`, `mail.mt_note`) are exact contract names that an implementation must preserve because external callers, stored data or shipped records depend on them. Each is given with its full name in words on first use in a document.
- Wherever the behavior of the code leaves a detail unspecified and this document fills it with the common practice of messaging software, the sentence is marked with the phrase **industry-standard default**.
- A reproduced route path, stored selection value, sequence code or verbatim message is written exactly as the system produces it, because an embedded widget, an external callback or a support procedure depends on it character for character.

## How the two source drafts were merged

This folder was consolidated from two independently written drafts of the same domain. The merge kept every entity, field, state value, transition, rule, message, formula, worked example, algorithm step, workflow, setting, access rule, interface contract, report, scenario and glossary term that either draft carried. Where both stated the same fact, the more precise wording was kept once. Where they contradicted each other, the source tree decided, and the decision is recorded in a "Reconciliation notes" section at the end of the affected file. Those sections exist in [entities.md](entities.md), [state-machines.md](state-machines.md), [workflows.md](workflows.md), [business-rules.md](business-rules.md), [calculations.md](calculations.md), [accounting-effects.md](accounting-effects.md), [configuration.md](configuration.md), [interfaces.md](interfaces.md) and [acceptance-criteria.md](acceptance-criteria.md).

Two of the drafts organised the same material differently: one kept the incoming gateway, the activities, the channels with live chat, and the text messages with postal mail in four separate topic files. Their content is preserved here inside the eleven documents — the gateway algorithm in [calculations.md](calculations.md), section 16, and its procedures in [workflows.md](workflows.md), sections 10, 11 and 28 to 30; the activities across [entities.md](entities.md), sections 17 to 20, [calculations.md](calculations.md), sections 13 to 15, and [workflows.md](workflows.md), sections 13, 14 and 31; the channels and live chat across [entities.md](entities.md), sections 34 to 42, [calculations.md](calculations.md), sections 23 to 26 and 34, and [workflows.md](workflows.md), sections 15 to 17 and 32; and the text messages and postal mail across [entities.md](entities.md), sections 44 and 46, [calculations.md](calculations.md), sections 28 and 31, and [workflows.md](workflows.md), sections 19, 21, 26 and 27. No extra topic file is therefore needed, and nothing was dropped.
