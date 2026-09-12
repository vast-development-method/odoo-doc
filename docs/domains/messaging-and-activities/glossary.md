# Messaging and Activities — Glossary

Every term this folder uses, defined. Words are written in full; no abbreviation is used except inside a reproduced identifier written in code font, which always carries its full name in words the first time it appears.

Where a term names a record of this domain, the transport name (the identifier the remote interface uses) is given in code font after the definition, and the full field-by-field description is in [entities.md](entities.md).

| Term | Definition |
|---|---|
| **Access group** | A named set of permissions. Groups decide who may see a restricted Channel, who may edit a Template that contains a dynamic placeholder, who may operate live chat and who may administer a Mailing Group. Groups themselves belong to [../identity-and-access/](../identity-and-access/); this folder only uses them. |
| **Activity** | One planned action attached to one record (or free-standing), with a type, a due date, an assignee and a derived state. Completing it posts a Message and archives the Activity. `mail.activity`. |
| **Activity Plan** | A named set of activity lines launched together against one or several records of one model from a single anchor date. `mail.activity.plan`. |
| **Activity Plan Template** | One line of an Activity Plan: a type, a delay, a delay side and an assignment rule. `mail.activity.plan.template`. |
| **Activity state** | The derived value of an Activity: overdue, today or planned while the Activity is live, done once it is archived. Computed against today in the **assignee's** time zone. |
| **Activity Type** | The category of an Activity: its default summary, its default note, its default assignee, its default delay, its chaining rule, its decoration and its action class. `mail.activity.type`. |
| **Address** | Without qualification, an electronic mail address. A **normalized address** is the lower-case `local-part@domain` form with any display name removed; a **formatted address** is the `"Display Name" <local-part@domain>` form; a **raw address** is whatever was typed or received. |
| **Alias** | One inbound address bound to a model. A message arriving at it creates or updates a record of that model, under the alias's security policy. `mail.alias`. |
| **Alias Domain** | One electronic mail domain owned by the installation, with its catch-all local part, its bounce local part and its default-sender local part. `mail.alias.domain`. |
| **Already-reached addresses** | The addresses that an incoming electronic mail had already delivered to. They are stored on the Message so the notification pass does not send a second copy to the same people. |
| **Announcement service** | The publisher service contacted once a week; the announcements it returns are posted as Messages in the company-wide Channel. |
| **Assignee** | The User responsible for an Activity. |
| **Attachment** | A stored file linked either to a business record or to a Message. Attachments themselves belong to [../platform-foundation/](../platform-foundation/); this folder adds the thumbnail, the voice marker and the deletion broadcast. |
| **Author** | The Contact who wrote a Message. A Guest may be the author instead of a Contact; then the author contact and the sender address are both empty. |
| **Automated activity** | An Activity created by business code rather than typed by a person, marked with the automated flag so the same code can find, reschedule, complete or delete it later. |
| **Blacklist Entry** | One electronic mail address that must never receive a mass message. Global, keyed on the normalized address, archived rather than deleted when the address is released. `mail.blacklist`. |
| **Blocked number** | A telephone number on the global suppression list. `phone.blacklist`. |
| **Bounce** | A delivery-failure notice returned by a recipient's mail system. The incoming router recognizes it, raises the bounce counters, updates the matching Notifications, and never creates or updates a record from it. |
| **Bounce address** | The `<bounce local part>@<domain>` address of an Alias Domain, used as the return path of every outgoing electronic mail so that failures come back to the platform. |
| **Bounce counter** | The number of times a message addressed to a record's normalized address has bounced. Raised on every bounce, reset to zero as soon as a valid message arrives from that address. |
| **Broadcast channel** | The addressing unit of the event bus: a plain string, or a record, or a record plus a sub-channel name, to which payloads are pushed. Not to be confused with a conversation Channel. |
| **Canned Response** | A shortcut, typed after two colons, that expands into a longer text while composing. `mail.canned.response`. |
| **Catch-all address** | The `<catch-all local part>@<domain>` address of an Alias Domain. It collects replies to notification electronic mails when the record has no Alias of its own. Writing to it directly is bounced. |
| **Chaining** | What an Activity Type does when one of its activities is completed: either *suggest* a list of successor types, or *trigger* one successor automatically. |
| **Channel** | A multi-party conversation. Four kinds exist: a channel that may be joined, a private group, a two-person chat and a live chat session. `discuss.channel`. |
| **Channel Member** | One party inside one Channel, carrying that person's unread markers, pin state, mute setting, personal channel name and call participation. `discuss.channel.member`. |
| **Chatbot Message** | The link between one posted Message, the Chatbot Script Step that produced it and the answer the visitor gave. `chatbot.message`. |
| **Chatbot Script** | An ordered list of steps a bot plays in a live chat session before, or instead of, a human operator. `chatbot.script`. |
| **Classification of a recipient** | One of *user*, *portal* or *customer*. It decides which rendering group the recipient falls into and therefore what the notification electronic mail shows. |
| **Composer** | The window used to write a Message, a mass mailing or a text message, optionally loading a Template. `mail.compose.message` for electronic mail, `sms.composer` for text messages. |
| **Consent code** | The short-lived signed token that an electronic-mail client add-in obtains in the browser and exchanges for a durable application key. Valid for three minutes. |
| **Conversation Tag** | A qualification label placed on a finished live chat session. `im_livechat.conversation.tag`. |
| **Customer (contact)** | A Contact that has no user, or only portal and public users. A customer never receives a message whose subtype is marked internal, and receives only the external default subtypes when subscribed. |
| **Delivery channel** | How a recipient is reached: the in-application inbox, electronic mail, a text message, a postal letter or a browser push. |
| **Digest** | A periodic summary electronic mail carrying a chosen set of indicators, each shown over three time windows with a comparison against the preceding window. `digest.digest`. |
| **Digest Tip** | One rotating hint shown at the bottom of a Digest; each sending consumes one. `digest.tip`. |
| **Duration tracking** | The map from each value a tracked link field has taken to the number of whole seconds the record spent on that value. |
| **Enrichment outcome** | The single value describing how a company-enrichment attempt ended: company created, company updated, missing data, insufficient credit, no data, other, or a platform-side explanation. |
| **Event bus** | The transport that pushes payloads to connected clients. One entry is one payload on one broadcast channel. `bus.bus`. |
| **Expertise** | A skill tag used to prioritize live chat operators. `im_livechat.expertise`. |
| **Failure type** | The machine-readable reason a delivery failed: an invalid address, a missing address, insufficient credit, a rejected sender, and so on. Each Notification and each Outgoing Mail carries one. |
| **Flat thread** | A model whose messages are all attached to the first relevant ancestor message of the record, so the conversation reads as a list rather than a tree. |
| **Follower** | One Contact subscribed to one record with an explicit set of subtypes. Followers are who the notification pass reaches. `mail.followers`. |
| **Forwarding unit** | The media server that relays audio and video once a call reaches three participants, replacing direct browser-to-browser connections. |
| **Gateway** | The pipeline that turns a received electronic mail into a Message, a record creation, a record update, a bounce, or nothing. |
| **Gateway Allowed Sender** | One trusted sender address exempted from the incoming loop quota. `mail.gateway.allowed`. |
| **Guest** | A named participant of a Channel who has no user account, identified by a secret token carried in a browser cookie. `mail.guest`. |
| **Inbox** | The in-application list of Messages the reader has been notified of and has not yet marked read. |
| **Internal note** | A Message carrying the shipped internal-note subtype. Never visible to portal or public users. |
| **Layout** | The wrapper rendered around a message body when it is mailed: the preview line, the optional header with the access button, the body, the tracking lines, the signature and the optional footer. |
| **Link Preview** | The cached preview metadata of one web address, shared by every Message that mentions it. `mail.link.preview`. |
| **Live Chat Channel** | One public entry point for visitors, with its operators, its look, its capacity rule and its display rules. `im_livechat.channel`. |
| **Live Chat Member History** | The durable record of one participant of one live chat session, kept after the membership row is deleted so reporting stays correct. `im_livechat.channel.member.history`. |
| **Live chat session** | A Channel whose type is the live chat type, created when a visitor starts a conversation. |
| **Loop detection** | The protection that suppresses an incoming message when the same sender has produced too many records or too many replies inside the observation window. |
| **Mailing Group** | A public discussion list fed by electronic mail, with subscription by confirmation link, optional moderation and a public archive. `mail.group`. |
| **Mailing Group Message** | One post of a Mailing Group, wrapping a Message and adding the list thread position and the moderation status. `mail.group.message`. |
| **Main attachment** | The Attachment designated as a record's principal document, shown in the document preview pane. |
| **Mention** | Naming a Contact, a Role or everyone inside a message body, which turns the named parties into direct recipients. |
| **Message** | One entry of a conversation, or a recipient-specific notification attached to no conversation. `mail.message`. |
| **Message identifier** | The globally unique token carried by every Message, used to recognize a reply, to detect a duplicate and to detect a reply to the platform's own bounce. |
| **Message Notification Schedule** | A posted Message whose notification pass is deferred to a future moment. `mail.message.schedule`. |
| **Message Subtype** | The reason a Message exists, and the unit a Follower subscribes to. It also decides whether the Message is internal. `mail.message.subtype`. |
| **Message type** | What produced the Message: an incoming electronic mail, a comment, an outgoing mailing, a system notification, an automated targeted notification, an out-of-office answer, a user-specific notification, a text message or a postal letter. |
| **Moderation** | The approval step a Mailing Group may require before a post is relayed, with permanent allow and ban rules per address. |
| **Normalized address** | See **Address**. |
| **Notification** | The per-recipient delivery record of one Message through one channel, with its status, its read flag and its failure reason. `mail.notification`. |
| **Notification preference** | A User's choice between receiving notifications in the in-application inbox and receiving them by electronic mail. A portal or public user always receives them by electronic mail. |
| **Onboarding state** | The progress of one User through the assistant bot's guided conversation, stored on the User. |
| **Operator** | An internal user who takes live chat conversations. |
| **Out-of-office answer** | The automatic reply produced when a notified user is inside their declared absence window. |
| **Outgoing Mail** | One electronic mail waiting to be handed to a relay, wrapping one Message. `mail.mail`. |
| **Party** | A Contact used as an author, a recipient or a follower. Where the distinction between an internal user, a portal user and a bare contact matters, the text says so explicitly. |
| **Postal Letter** | One document queued for printing and posting by an external service. `snailmail.letter`. |
| **Presence** | Whether a user or a guest is online, away or offline, derived from their last interaction and possibly overridden manually. `mail.presence`. |
| **Preview line** | The hidden text placed first in a notification electronic mail so that a mail client shows it next to the subject. |
| **Push Device** | One registered browser subscription endpoint belonging to one Contact. `mail.push.device`. |
| **Push Notification** | One queued browser push payload for one Push Device. `mail.push`. |
| **Rating** | One satisfaction answer between zero and five, requested through a signed link, aggregated on the rated record and on its parent. Owned by [../learning-surveys-and-gamification/](../learning-surveys-and-gamification/); used here by live chat. |
| **Recipient (direct)** | A Contact named directly on a Message, typically by a mention or by the composer, notified whether or not they follow the record. |
| **Rendering group** | A set of recipients that share one rendering of the notification electronic mail, for example internal users who receive an access button and customers who do not. |
| **Reply address** | The address written on an outgoing electronic mail so that answers come back to the right record: a record-specific Alias, else the company catch-all, else the sender address. |
| **Role** | A named set of Users addressable by mention; mentioning it notifies every member who can access the conversation. `res.role`. |
| **Rotting** | See **Staleness**. |
| **Scheduled Message** | A Message drafted now and posted into the conversation later; until then no Message exists. `mail.scheduled.message`. |
| **Sub-thread** | A Channel opened from one message of a Channel or a group, inheriting the parent's type and authorization group. |
| **Staleness** | The condition of a record that has not left the current value of its duration-tracking field for longer than the threshold configured on that value. |
| **Suppression list** | The set of active Blacklist Entries and, for telephone numbers, the set of active blocked numbers. Mass sending consults it; a single deliberate message does not. |
| **Template** | A reusable subject, body, sender, recipient set, attachment set and report set containing placeholders, rendered against a record. `mail.template` for electronic mail, `sms.template` for text messages. |
| **Text Message** | One short message queued for delivery to a telephone number by an external sending service. `sms.sms`. |
| **Text Message Tracker** | The bridge between one provider correlation token and one Notification, kept so that a late delivery report can still be applied. `sms.tracker`. |
| **Thread** | The abstract behavior that gives any business record a conversation, followers, field change tracking and the notification machinery. `mail.thread`. |
| **Thread-enabled model** | A model that adopts the Thread behavior. **Activity-enabled model** means a model that adopts the activity behavior. |
| **Tracked field** | A field whose definition carries a tracking order number, so every change of it produces a Tracking Value and a logged Message. |
| **Tracking Value** | The old value and the new value of one tracked field, recorded on the Message that reports the change. `mail.tracking.value`. |
| **Transcript** | The printable or mailable record of one live chat conversation. |
| **Unread separator** | The message identifier at and above which a Channel Member's unread messages begin; the client draws the "new messages" line there. |
| **Visitor** | An unauthenticated website user, possibly recognized across requests by the website tracking record of [../website-and-storefront/](../website-and-storefront/). |
| **Void message** | A Message whose body has no visible content, whose subtype has no description, that has no attachment and no readable tracking value. The interface hides it. |
