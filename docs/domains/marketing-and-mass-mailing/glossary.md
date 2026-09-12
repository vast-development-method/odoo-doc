# Marketing glossary

Every term this domain uses in a sense a reader could not guess, and every term whose everyday
meaning differs from the meaning here. Terms are grouped by subject; within a group they are
alphabetical. A term written in **bold** inside a definition has its own entry.

---

## 1. The message and its life

**Mailing.** One designed message together with the audience it is addressed to, the moment it
leaves, the state of that departure and the measurement of what happened afterwards. The canonical
entity name is Mass Mailing. A mailing is written once and delivered to many recipients; it is not a
conversation.

**Mailing type.** The channel a mailing uses: electronic mail, stored as `mail`, or text message,
stored as `sms`. The type decides which body field is sent, which **medium** is derived, which
exclusion checks run and which winner criteria are offered.

**Draft.** The state in which a mailing is being written. Nothing has been handed to any transport
and every field is editable.

**In queue.** The state in which a mailing has been released by its author and waits for the **queue
job**. It may still be cancelled back to draft.

**Sending.** The state in which the queue job has started, or resumed, the **batch loop**. Delivery
records exist for the recipients already processed.

**Sent (state).** The state, stored as `done`, in which a mailing has no remaining recipient. It
does not mean every message arrived: arrival is measured per recipient by the **delivery status**.

**Departure.** The moment at which a queued mailing may be picked up: the later of its schedule date
and the current moment. A mailing scheduled in the past departs at the next run of the queue job,
never retroactively.

**Calendar moment.** The single moment shown for a mailing in the calendar view: the sent moment when
it is finished, the next departure when it is queued, the current moment while it is sending, and
nothing while it is a draft.

**Favorite design.** A mailing marked as a reusable body. The gallery of designs lists every favorite
whose body is not visually empty, archived ones included.

**Preview sentence.** A short line displayed by most inboxes next to the subject. It is prepended to
the body inside a hidden block of zero size and zero opacity so that the inbox reads it and the
reader does not see it twice.

**Archive keeping.** The choice to keep the outgoing message records after sending instead of letting
them delete themselves. It changes the exclusion outcome of a missing or unusable address from
cancellation to failure.

---

## 2. The audience

**Recipient entity.** The kind of record a mailing addresses. It is chosen from the entities that
declare themselves **mailing enabled**. When the chosen entity is Mailing List, the records actually
addressed are Mailing Contacts.

**Mailing enabled.** A flag an entity declares to say that a mailing may address it. Contact, Mailing
Contact and Mailing List always declare it; Lead, Sales Order, Event Registration and Event Track
declare it when the matching capability package is installed.

**Recipient condition.** The stored expression that selects the records to address, evaluated against
the recipient entity without elevated rights. An expression that cannot be parsed selects nothing,
which is what prevents an accidental send to everybody.

**Saved filter.** A named, reusable recipient condition bound to one recipient entity. Loading one
replaces the stored condition; editing the condition afterwards does not unload the filter, so the
two values can differ and the form shows both.

**Default condition.** The condition an entity publishes for mailings, applied whenever the recipient
entity changes and no saved filter is loaded. Mailing List publishes membership of the chosen lists;
Sales Order excludes cancelled orders; Event Registration excludes cancelled and draft registrations;
Event Track excludes cancelled talks.

**Audience.** The set of recipient records matching the condition, before any exclusion and before
any **sample**.

**Remaining recipients.** The audience minus every record that already has a **delivery record** for
this mailing, or, for a **comparison test**, for this campaign. This is what makes an interrupted
send resumable and a comparison test collision-free.

**Mailing List.** A named audience of Mailing Contacts. Membership is carried by a **subscription**,
never by a bare link, because the opt-out state belongs to the pair and not to the contact.

**Public list.** A Mailing List marked as shown in preferences. Only a public list is offered on the
subscription-management page, may be joined from that page and may be named in an unsubscribe
confirmation. A private list appears there as `Mailing List #<key>`.

**Mailing Contact.** A lightweight addressee, deliberately separate from the general Contact entity,
so that very large audiences cost little. Two Mailing Contacts may share one address on purpose, in
order to hold different per-list preferences.

**Subscription.** The membership of one Mailing Contact in one Mailing List, carrying the opt-out
flag, the moment of the opt-out and its reason. A contact may be a member of a list and opted out of
it at the same time.

**Opt out.** A recipient's refusal of further messages from one list. It is recorded on the
subscription, never on the contact, and it is reversible.

**Opt-out reason.** A selectable sentence explaining why a person left a list or blocked themselves.
A reason may ask for free text, which reveals a text box on the public page.

**Blocked address, blocked number.** A global refusal held in the registers of the messaging domain.
It applies to every mailing at once and is what the **exclusion list** flag consults.

**Exclusion list.** The switch that decides whether the blocked-address register is consulted for a
mailing. Switching it off removes only that check; the opt-out, duplicate and address-validity checks
still run.

**Already-contacted set.** The set of addresses this mailing, or this campaign when a comparison test
is running, has already reached. It is what suppresses a second message to the same person.

**Same-run duplicate.** A message suppressed because an identical message — same subject, same body,
same attachment count — has already been prepared for the same address inside the current run. A body
that differs between two recipients, for instance because it prints their names, is not a duplicate.

---

## 3. Sending

**Sending algorithm.** The procedure that turns a mailing and its remaining recipients into outgoing
messages and delivery records, in fixed-size slices with a commit between them.

**Batch loop.** The heart of that algorithm: prepare a slice, apply the **exclusion checks**, drop
the cancelled entries after creating their delivery records, create the outgoing messages, then
either send at once or report progress, commit and drop the caches.

**Batch size.** The number of recipients handled in one slice, taken from a shared system parameter
and defaulting to 50 when the parameter is absent or zero.

**Exclusion checks.** The ordered list of six reasons to suppress a prepared message. The first
matching reason applies and no other; each produces a specific failure type.

**Failure type.** The code stored on a delivery record explaining why it did not succeed.
Twenty-nine values exist across the two channels, plus four contributed by the alternative telephony
provider.

**Queue job.** The scheduled process that drains the queue: it selects the mailings that are due,
sends them, closes the exhausted ones and, at the end, sends the **statistics message** for the
mailings that are due one.

**Wake request.** An instruction that makes a scheduled job run earlier than its interval. Queueing a
mailing wakes the queue job; writing a comparison-test moment wakes the comparison-test job.

**Direct send.** The path that sends a mailing in one pass without queueing. It is used by the
comparison-test winner and by the message composer when it created the mailing itself.

**Dedicated server.** An outgoing mail server chosen for marketing alone. When one is configured it
becomes the default of every new mailing, it may no longer be given a personal owner, and a mailing
never falls back to a server that has one.

---

## 4. Measurement

**Delivery record.** One record per recipient record of a mailing, holding the whole history of that
one delivery: its status, its moments, its failure code and its clicks. The canonical entity name is
Mailing Trace. Delivery records live in their own table so that the measurement survives the deletion
of the outgoing message.

**Delivery status.** The state of one delivery record. Nine values exist. Two of the labels are
shifted with respect to their stored values: the stored value `pending` is displayed as "Sent" and
the stored value `sent` is displayed as "Delivered"; every formula uses the stored values.

**Test trace.** A delivery record created by a test send, flagged so that it is excluded from the
real measurement screens.

**Expected.** The number of delivery records of a mailing, whatever their status. It is the base from
which the three percentage denominators are derived by subtraction.

**Received ratio.** The share of delivered records among everybody who was not cancelled.

**Opened ratio.** The share of opened or replied records among everybody who could actually have
engaged, that is, excluding the cancelled, the bounced and the failed.

**Replied ratio.** The same denominator, counting only the replied.

**Bounced ratio.** The share of bounced records among everybody who was actually handed over, that
is, excluding the cancelled and the failed.

**Click ratio.** The share of delivery records with at least one recorded visit among the records
whose status is not bounced, cancelled or failed. A person who clicks five links five times each
counts once here and twenty-five times in the click counters of the trackers.

**Statistics message.** The electronic mail sent to the responsible person one day after a mailing
finishes, carrying an engagement block, a business block, the table of link trackers and one tip. It
can be turned off for the whole installation from a link inside it.

**Analytical view.** The read-only aggregation of delivery records by mailing, campaign, source,
state and creation moment. Note that it counts an opened delivery as the status `open` alone, whereas
a mailing counts `open` and `reply` together.

**Bounce.** A notification from a receiving system that a message could not be delivered. Five
bounces of one address inside thirteen weeks, spread over more than one week, add that address to the
blocked-address register automatically.

---

## 5. Comparison testing

**Comparison test.** Two or more versions of one message, each addressed to a random percentage of
the audience, compared on a chosen indicator, after which the winning version is sent to everybody
who has not been reached. It is the feature usually called split testing.

**Version.** One mailing of a comparison test. Every version belongs to the same **campaign**, which
is what makes them share an audience without contacting anybody twice.

**Comparison-test percentage.** The share of the audience one version is addressed to, between 0 and
100 inclusive.

**Sample.** The random subset of the remaining audience a version actually reaches. Its size is the
percentage of the whole audience, truncated towards zero, raised to at least one whenever the
audience is not empty, and reduced to the size of the remainder when it exceeds it.

**Decision moment.** The moment at which the winner is chosen automatically. It defaults to one day
after creation and is deliberately carried over when a version is duplicated.

**Winner criterion.** The indicator on which the versions are ranked: manual selection, highest open
rate, highest click rate, highest reply rate, and, with the matching packages, the lead count, the
quotation count or the invoiced amount. The text-message channel offers manual selection, highest
click rate and the three business criteria.

**Winner mailing.** A copy of the winning version at 100 percent, named after the original with the
suffix `(final)`. Recording it marks the campaign completed, which permanently prevents any further
winner operation.

**Completed campaign.** A campaign whose winner has been recorded. The condition is stored and
derived; it is never turned off.

---

## 6. Campaign tracking

**Campaign.** A named marketing effort with a stage, tags, a responsible person and aggregated
indicators. It is the unit of attribution and the parent of a comparison test. Its technical name is
unique across the installation and is what appears in a link parameter.

**Campaign Source.** Where a contact or a click came from. Every mailing owns exactly one Campaign
Source, whose name doubles as the mailing's own name, which is why mailing names are unique.

**Campaign Medium.** How a message was delivered: electronic mail, text message, a banner, a
telephone call and so on. Seven mediums are protected against deletion because other flows name them.

**Campaign Stage.** A column of the campaign board. Every stage is displayed even when it holds no
campaign.

**Campaign Tag.** A coloured label on a campaign. A colour index of zero means the tag is not shown
on a card.

**Campaign tracking parameters.** The three query parameters a marketing link carries, named
`utm_campaign`, `utm_source` and `utm_medium`. They are captured into three cookies after every
request and stamped on records created afterwards.

**Automatically generated campaign.** A campaign created implicitly because a visitor's link named a
campaign that did not exist. It is flagged so that it stays out of the campaign screens.

**Unique-name counter.** The bracketed number appended to a name to make it unique. Counter 1 is
written as the bare name; the algorithm always fills the lowest free counter, reusing holes left by
deleted records.

**Attribution.** The act of recording, on a business record, which campaign, source and medium
produced it. Attribution is what lets a mailing report the leads, quotations and revenue it caused.

---

## 7. Links

**Link Tracker.** A target web address together with its campaign attribution, exposed through one or
more short codes and counting the visits it receives. Two links with the same target but different
labels are two trackers, therefore two codes, therefore counted separately.

**Short code.** The short random string that identifies a tracker in a shortened address. Codes start
at three characters and grow only when a collision actually happens.

**Short address.** The address a recipient actually clicks: the short host, the path segment that
marks a redirection, and the code. When the link was inside a message it carries the delivery record
key or the outgoing text message key as well, which is what identifies the exact recipient.

**Redirection address.** The address the visitor is finally sent to: the target address with the three
campaign tracking parameters appended, unless external tracking is suppressed and the target host
differs from the site host.

**Link label.** The clickable text of a link, trimmed and truncated to forty characters, captured
when the link was shortened. For a link whose only content is an image, the label is the alternative
text or the last path segment of the image source, prefixed to mark it as media.

**Skip list.** The addresses that are never shortened inside a mailing body: the unsubscribe
placeholder, the view-in-browser placeholder and the marketing card paths.

**Recorded visit.** One click of a short address, holding the network address of the visitor, the
resolved country, the campaign, and, when the click came from a message, the delivery record and the
mailing. The canonical entity name is Link Tracker Click.

**Automated agent.** A caller recognised as a preview fetcher or a crawler by its agent string.
Visits by such callers are not recorded on a plain short link, but they are recorded on a link that
carries a delivery record key, because that key makes the visit attributable.

---

## 8. Public pages

**Recipient token.** The keyed hash of the database name, the mailing key, the recipient record key
and the address, computed with the database secret. It is what lets an anonymous recipient act on
their own subscription without an account, and it is always compared in constant time.

**Subscription-management page.** The public page a recipient reaches from a message: it lists the
lists they belong to, the public lists they may join, and offers the exclusion buttons and the
feedback form.

**One-click unsubscribe.** The address advertised in the unsubscribe header of every marketing
message, which a mail client may call without a session and without the cross-site request token.

**Self exclusion.** A recipient adding their own address to the blocked-address register from the
public page. Its opposite is re-inclusion, which archives the register entry.

**Feedback window.** The ten minutes after an opt-out during which a submitted reason is attached to
the subscriptions that were just opted out. A later submission touches nothing.

**View in browser.** The page that renders the body of a mailing for one recipient, reached from a
link inside the message.

**Open tracking image.** The one-pixel transparent image appended to every outgoing marketing
message. Fetching it marks the delivery record opened.

**Placeholder link.** An address written into a body by the designer that a real message never
contains, because it is replaced per recipient at send time. Two exist: the unsubscribe placeholder
and the view-in-browser placeholder.

---

## 9. Text messages

**Text message mailing.** A mailing whose channel is the text message. It carries a plain-text body,
sanitised destination numbers, an optional opt-out sentence and delivery reports from the sending
service.

**Sanitised number.** The strict international form of a telephone number, computed against the
country of the record, of the visitor or of the company. Comparison of numbers is always done on the
sanitised form.

**Opt-out code.** The three random characters written on a text-message delivery record and into the
opt-out address. Uniqueness is not required, because the triple of code, mailing and number is what
the page verifies.

**Delivery report.** The message the sending service sends back to say what happened to one text
message. It drives the delivery status and closes the mailing when nothing is left being processed.

**Length estimate.** The calculation that tells the author how many characters the shortened links
and the opt-out sentence will occupy, before those links exist, by building two placeholder strings
of the right length.

---

## 10. Marketing cards

**Marketing card.** A personalised image rendered for one record, which the person concerned can
share on a social network. Its dimensions are 600 by 315 pixels, the ratio recommended for a large
preview image.

**Card campaign.** The design, the content mapping, the target entity and the sharing text from which
cards are produced. Its canonical entity name is Marketing Card Campaign.

**Content slot.** One place in a card design that receives a value: a header, a sub-header, a
section, two sub-sections, two images, a background, a button. A textual slot is either static or
dynamic; a dynamic slot names a field path walked on the record.

**Requires synchronisation.** The flag on a card saying that its image no longer matches the current
design. Writing any design field of the campaign sets it on every card, archived ones included, and a
mailing cannot be queued while any recipient's card carries it.

**Share status.** The evidence a card has been used: empty when nobody opened its page, `visited`
when the person opened it, `shared` when a recognised crawler fetched the image. It is never
downgraded.

**Crawler page.** The answer served to a recognised crawler at the card redirection address: a page
carrying only the social-preview metadata, so that the network shows the card when the person posts
the link.

---

## 11. Words used in a specific sense

**Assistant.** A short-lived form that collects a few values and performs one operation, holding no
lasting record. Six exist in this domain.

**Elevated rights.** Reading or writing performed with the permissions of the system rather than of
the caller. This domain uses it for the public pages, for the card status writes, for the winner
comparison and for creating the delivery records of cancelled messages.

**Normalised address.** The lower-cased, comment-free, single-address form of an electronic mail
address. Comparison of addresses is always done on the normalised form.

**Rendered.** Evaluated as a template against one recipient record, so that placeholders become
values. The subject, the preview sentence, both bodies and the answer address are rendered per
recipient.

**Post-processing.** The step after rendering that converts local links into absolute addresses and
then shortens every link that is not in the skip list.

**Marketing layout.** The wrapper applied to every rendered body before it is sent, carrying the
marketing stylesheet. It is what makes a message look the same in the inbox and in the browser view.

**Building block.** One reusable piece of a body offered by the designer: a header, a cover, a text
block, an image-and-text block, a media list, a product list, an event block, a call-to-action block,
a badge, a quotation, a rating, a separator, a features grid, a masonry block, a company-team block,
a reference block, a promotional-code block or a footer.

**Design.** A complete starting body offered when a mailing is created. Fourteen are shipped; a
design is copied into the mailing, not linked, so a later change never rewrites a mailing already
written.

**Progress reporting.** The pair of numbers a long-running job publishes so that the runner can show
how far it has come and decide when to stop. The sending pass reports it after every committed batch.
