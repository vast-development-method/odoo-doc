# Messaging and Activities — Calculations

Every formula and every algorithm of the domain, with its rounding rule, its precision, its date arithmetic and at least one worked numeric example.

Formulas are written in plain mathematics. Algorithms are numbered steps with preconditions, postconditions and failure conditions.

Contents:

1. [Address normalization and formatting](#1-address-normalization-and-formatting)
2. [The recipient computation](#2-the-recipient-computation)
3. [Recipient grouping for notification rendering](#3-recipient-grouping-for-notification-rendering)
4. [The reply address](#4-the-reply-address)
5. [The message identifier and the reference chain](#5-the-message-identifier-and-the-reference-chain)
6. [Parent selection for a posted message](#6-parent-selection-for-a-posted-message)
7. [Field change tracking](#7-field-change-tracking)
8. [Tracking display order](#8-tracking-display-order)
9. [Duration tracking and staleness](#9-duration-tracking-and-staleness)
10. [Automatic subscription](#10-automatic-subscription)
11. [Default and suggested recipients](#11-default-and-suggested-recipients)
12. [Finding or creating a contact from an address](#12-finding-or-creating-a-contact-from-an-address)
13. [Activity deadlines](#13-activity-deadlines)
14. [Activity plan deadlines](#14-activity-plan-deadlines)
15. [The activity state](#15-the-activity-state)
16. [Incoming routing](#16-incoming-routing)
17. [Loop detection](#17-loop-detection)
18. [Bounce detection and the bounce counter](#18-bounce-detection-and-the-bounce-counter)
19. [Outgoing batching and relay selection](#19-outgoing-batching-and-relay-selection)
20. [The personal relay throttle](#20-the-personal-relay-throttle)
21. [Attachment size and the link fallback](#21-attachment-size-and-the-link-fallback)
22. [Template rendering](#22-template-rendering)
23. [Channel unread counters](#23-channel-unread-counters)
24. [Channel display name](#24-channel-display-name)
25. [Live chat operator assignment](#25-live-chat-operator-assignment)
26. [Live chat capacity and durations](#26-live-chat-capacity-and-durations)
27. [Digest indicators](#27-digest-indicators)
28. [Text message number sanitizing and suppression](#28-text-message-number-sanitizing-and-suppression)
29. [Browser push payload truncation](#29-browser-push-payload-truncation)
30. [Miscellaneous derived values](#30-miscellaneous-derived-values)

---

## 1. Address normalization and formatting

Three distinct forms of an electronic mail address are used throughout the domain and must never be confused.

| Form | Shape | Where used |
|---|---|---|
| raw | whatever the user typed or the header contained | stored in the sender field of a message |
| normalized | `local-part@domain`, no display name | every comparison, every uniqueness rule, every blacklist lookup |
| formatted | `"Display Name" <local-part@domain>`, or the bare address when there is no name | outgoing headers |

### Normalizing

1. Split the input into (display name, address) pairs. A pair whose name is empty and whose address contains a space is re-split by replacing spaces with commas, so that an unquoted `Name email@domain` is recovered as a name and an address.
2. In **strict** mode, if the input yields more than one address, the result is "no address". In non-strict mode the first candidate is used.
3. The domain part is always lower-cased. A domain containing non-ASCII characters is encoded to its ASCII-compatible form.
4. The local part is lower-cased **only when it is pure ASCII**. A local part containing non-ASCII characters is kept as it is, because the international transport extension exists precisely to carry it.
5. The result must contain a local part, an at-sign and a domain. The domain need not contain a dot.

Worked example. Input `"  Name  <NaMe@DoMaIn.CoM>"` gives the normalized form `name@domain.com` and the formatted form `"Name" <name@domain.com>`. Input `tony@e.com, "Tony2" <tony2@e.com>` gives "no address" in strict mode and `tony@e.com` in non-strict mode.

### Formatting

1. Split the address into local part and domain at the last at-sign.
2. If the domain cannot be encoded in the target character set, encode it to its ASCII-compatible form.
3. If there is no display name, return the bare address.
4. Otherwise return the display name, quoted when necessary, followed by the address in angle brackets. A display name that cannot be encoded in the target character set is encoded as a transport-safe word.

### Sanitizing an alias local part

Used for alias names, for alias-domain names, and for the three special local parts of an alias domain.

1. Trim leading and trailing whitespace.
2. When the value is being treated as a whole address, remember the part after the first at-sign, lower-cased.
3. Remove accents; lower-case; keep only the part before the first at-sign.
4. Remove every leading dot, every trailing dot, and every dot that is immediately followed by another dot.
5. Replace every run of characters outside the allowed set by a single hyphen. The allowed set is: letters, digits, underscore, and the punctuation characters exclamation mark, number sign, dollar sign, percent sign, ampersand, apostrophe, asterisk, plus sign, hyphen, slash, equals sign, question mark, circumflex, underscore, grave accent, opening brace, vertical bar, closing brace, tilde and full stop.
6. Encode to plain ASCII, replacing anything that still does not fit.
7. If nothing is left, the result is "no name".
8. If a whole address was requested and a right part was remembered, re-join them with an at-sign.

Worked example. Input `  Jöbs & Careers @Example.COM ` treated as a local part gives `jobs-careers-`; treated as a whole address it gives `jobs-careers-@example.com`.

### Validity of an alias local part

Independently of sanitizing, a validation refuses any local part that is not a plain ASCII dot-atom: one or more allowed characters, optionally followed by groups of a dot plus one or more allowed characters. The error is "You cannot use anything else than unaccented latin characters in the alias address <name>."

---

## 2. The recipient computation

This is the central algorithm of the domain: given a message and the record it was posted on, decide who must be told and through which channel.

### Inputs

| Input | Source |
|---|---|
| the record | the thread the message was posted on, or nothing for a free-standing notification |
| the message type | the message |
| the subtype | the message |
| the direct recipients | the message's explicit recipient list |
| the extra outgoing addresses | the message's outgoing address list |
| the already-emailed addresses | the message's incoming "to" and "carbon copy" lists, normalized |
| the switches | notify the author; notify the author when mentioned; skip followers; skip existing notifications |

### Step 1 — decide whether followers participate

- If the "skip followers" switch is on, treat the message type as "user specific notification" for the rest of the computation. This makes the algorithm consider the direct recipients only.
- Otherwise use the real message type.

### Step 2 — fetch the candidate set

Three cases, depending on what is available.

**Case A — a record, a subtype, and a message type that is not "user specific notification".**

The candidate set is the union of:

- every Follower of the record **that follows the given subtype**; and
- every contact in the direct recipient list, whether or not they follow anything.

A follower is excluded from the union when the subtype is marked internal **and** the follower's contact is a customer. That single condition is what keeps internal notes away from customers.

**Case B — a record and direct recipients, but no subtype (or a user-specific notification).**

The candidate set is the direct recipient list. Each candidate is additionally flagged as a follower of the record when a Follower row exists, because the flag drives the notification grouping.

**Case C — direct recipients only, with no record.**

The candidate set is the direct recipient list, all flagged as non-followers.

### Step 3 — enrich each candidate

For each candidate contact, the following is gathered in the same query:

| Piece | Rule |
|---|---|
| active | the contact's active flag |
| normalized address | the contact's normalized address |
| name | the contact's name |
| language | the contact's language |
| is a customer | the contact's customer flag (no user, or only portal or public users) |
| user | the contact's **first active user**, ordered by sharing flag ascending with unknown first, then by identifier ascending. In other words: an internal user wins over a portal user, and among equals the lowest identifier wins. |
| users are shared | the sharing flag of that user |
| notification preference | that user's preference, defaulting to electronic mail when the contact has no user |
| groups | that user's groups, **extended with the transitive closure of implied groups** |
| is a follower | true in case A for a follower row, false for a pure direct recipient |

The candidate is then typed:

```formula
type = "portal"    when the found user is a shared user
type = "customer"  when there is no user and the contact is a customer
type = "user"      otherwise
```

### Step 4 — remove the author

Unless the "notify the author" switch is on:

1. Compute the **real author**. If the acting user is active, the real author is the acting user's contact — because an active user performing an action is the one who did it. If the acting user is inactive (the system user running the incoming gateway, for example), the real author is the message author, provided that author is active and is not the system contact.
2. If the "notify the author when mentioned" switch is on **and** the real author is among the direct recipients, do not remove them.
3. Otherwise remove the real author's contact from the candidate set.

### Step 5 — remove the inapplicable

Remove a candidate when any of the following holds:

- the contact is archived;
- the contact's normalized address appears among the already-emailed addresses, that is, among the normalized "to" and "carbon copy" addresses of the incoming message that produced this one. This is what prevents notifying by electronic mail somebody who was already a recipient of the original.

### Step 6 — add the address-only recipients

For each entry of the extra outgoing address list, add a synthetic candidate: active, no identifier, not a follower, name equal to the display name found in the entry or to the address itself, no language, no groups, delivery by electronic mail, flagged as a customer, typed "customer".

### Step 7 — optionally drop already-notified contacts

When the "skip existing notifications" switch is on, look up the existing Notifications of the message for the candidate contacts and drop the candidates that already have one. This is off by default because it costs an extra query.

### Output

A list of records, one per recipient, each carrying: active flag, normalized address, contact identifier (or none), follower flag, name, language, group identifiers, notification channel, customer flag, type, user identifier and shared-users flag.

### Worked example — a message with two followers on different subtypes

Given:

- a record of a thread-enabled model, with two followers:
  - Ana, an internal user whose notification preference is the in-application inbox, following the subtypes "Discussions" and "Note";
  - Ben, a customer contact with no user, following only "Discussions";
- the shipped subtypes: "Discussions" is not internal, "Note" is internal;
- the acting user is Carla, an internal user who is **not** a follower.

**When** Carla posts a message with the subtype "Discussions":

1. Candidate set: Ana (follows the subtype), Ben (follows the subtype). Carla is not a follower and is not a direct recipient.
2. The subtype is not internal, so Ben survives the customer filter.
3. Carla is the real author, but she is not a candidate, so nothing is removed.
4. Result: two recipients. Ana with channel "inbox", type "user", follower flag true. Ben with channel "email", type "customer", follower flag true.
5. Delivery: one Notification of channel "inbox" in status "delivered" for Ana, plus a broadcast to Ana's client; one Notification of channel "email" in status "ready" for Ben, plus one Outgoing Mail addressed to Ben.

**When** Carla posts a message with the subtype "Note" instead:

1. Candidate set from followers: Ana (follows "Note"), Ben (does **not** follow "Note").
2. Result: one recipient, Ana, channel "inbox".
3. Even if Ben had followed "Note", he would have been removed at step 2 because the subtype is internal and he is a customer.

**When** Carla posts with the subtype "Discussions" and explicitly adds Ben as a direct recipient:

The result is the same two recipients; Ben appears once, because the union deduplicates by contact.

**When** Carla posts with the subtype "Discussions" and the message carries an incoming "to" list containing Ben's address (the case of an incoming electronic mail that already copied Ben):

Ben is removed at step 5 and only Ana is notified.

---

## 3. Recipient grouping for notification rendering

Notification electronic mails are not rendered once per recipient; they are rendered once per (language, group) pair. This is what makes a thousand-follower notification affordable.

### Step 1 — group by language

Each recipient is placed in the bucket of their own language, when that language is installed; otherwise in the bucket of the forced language when the caller supplied one; otherwise in the bucket of the acting language.

### Step 2 — classify inside each language bucket

Four groups are evaluated **in order**; the first whose predicate matches wins, and the recipient joins only that group.

| Order | Group | Predicate | Active by default | Shows the access button by default |
|---|---|---|---|---|
| 1 | user | the recipient's type is "user" | yes | yes, when the message is attached to a record |
| 2 | portal | the recipient's type is "portal" | **no** | no |
| 3 | follower | the recipient follows the record | **no** | no |
| 4 | customer | anything else | yes | no |

A group that is not active is skipped entirely: its recipients fall through to the next matching group. Capabilities that grant portal access switch the portal group on; capabilities that want followers to receive a link switch the follower group on.

Groups with no recipient at the end are dropped.

### Step 3 — build the rendering context per language bucket

| Value | Rule |
|---|---|
| author's user | the first user of the author contact |
| signature | when the author has a user and the message asks for a signature, the author user's signature wrapped in a separator line; otherwise empty |
| company | the forced company when the caller supplied one; else the company of the record when the model has a company field and it is set; else the acting company |
| site address | the company's site address, prefixed with the plain hypertext transfer scheme when it carries no scheme; false when the company has none |
| model description | the forced description when supplied; otherwise the translated label of the model, resolved in the bucket's language |
| record name | the forced name when supplied; otherwise the display name of the record, resolved in the bucket's language |
| tracking values | the readable tracking values of the message, filtered by field access and by the model's display filter, each rendered as a triple (field label, old value, new value) |
| is a discussion | true when the subtype is the shipped "Discussions" subtype |
| show the unfollow link | true when the model allows unfollowing, or when any recipient of the group is a follower with an active non-shared user |

### Step 4 — render and create the outgoing mails

For each (language, group) pair:

1. Render the layout named by the message's layout identifier, defaulting to the standard notification layout. If the layout does not exist or renders empty, fall back to the raw message body and log a warning.
2. Split the group's contact recipients into chunks of the configured generation batch size (default 50) and create one Outgoing Mail per chunk, with the rendered body and the chunk as recipients.
3. If the group also has address-only recipients, create one more Outgoing Mail whose "to" field is the comma-joined list of those addresses.
4. Create one Notification per recipient, pointing at the Outgoing Mail that will carry them, with the author, the status "ready", the channel "email" and the read flag already set.

### Step 5 — decide whether to send now

```formula
send_now = force_send_requested and ( number_of_outgoing_mails < force_send_limit )
```

The force-send limit is a system parameter, default 100. When the acting context forces sending, that overrides the caller's choice. When sending now, the mails are sent **after the transaction commits** unless the caller asked otherwise, so that a failed transaction cannot leave sent electronic mails behind.

---

## 4. The reply address

The reply address decides where an answer lands. It is computed once per record.

### Algorithm

1. Group the records by company. A record's company is its company field when the model has one and it is set, else the default company supplied by the caller, else the acting company.
2. **Aliases first.** Search every Alias that has a domain, whose parent model is this model, whose parent record is one of the records, and that has a local part. For each record take the **first** such alias found and use its full address. This is how a container record (a project, a list) captures the answers of its children.
3. **Catch-all second.** For every record still without an address, use the catch-all address of its company's alias domain, when there is one.
4. **Default last.** A record still without an address keeps the default supplied by the caller, which is normally the sender address of the message.
5. Format the result.

### Formatting the reply address

The total length of the formatted value must stay under 68 characters, which is 78 (the line limit of the transport format) minus the length of the header name and its separator. The rule is:

1. If the bare address alone is already 68 characters or longer, return the bare address and log a warning that the header may be folded incorrectly.
2. Otherwise try the author's name plus the address. If that is at most 68 characters, use it.
3. Otherwise try the acting user's name plus the address.
4. Otherwise return the bare address.

Worked example. Catch-all `catchall@example.com` (20 characters) and author "Ana Fernández de la Torre" gives the candidate `"Ana Fernández de la Torre" <catchall@example.com>` which is 49 characters — accepted. With the catch-all `catchall@a-very-long-subdomain.example-company-group.com` (54 characters) the same candidate would be 83 characters, so the acting user's name is tried; if that is also too long, the bare address is used.

---

## 5. The message identifier and the reference chain

### The message identifier

Every Message receives a globally unique identifier at creation if none was supplied. The grammar is:

```formula
message_identifier = "<" + random_fraction + "." + epoch_seconds + fixed_marker + tag + "@" + host_name + ">"
```

where:

- `random_fraction` is the decimal expansion of a random number in the half-open interval from zero to one, with the leading "0." removed, taken from the operating system's cryptographic source when available;
- `epoch_seconds` is the current moment as seconds since the start of 1970, written with 15 decimal places;
- `fixed_marker` is the reproduced literal `-openerp-`, a contractual string that the incoming router matches on and that a compatible rebuild must emit unchanged;
- `tag` depends on the case;
- `host_name` is the name of the machine that generated it.

| Case | Tag |
|---|---|
| The message forces answers to be treated as new | `reply_to` |
| The message is attached to a record | `<the record identifier>-<model transport name>` |
| Anything else | `private` |
| A notification produced by the notify operation | `message-notify` |
| A batch log entry | `message-notify` |
| A loop-detection bounce | `loop-detection-bounce-email` |

The tag is what makes the incoming router able to recognize an answer without a database lookup in the common case, and what makes the loop detector able to recognize a reply to its own bounce.

Worked example: a message on record 42 of the model whose transport name is `crm.lead` produces an identifier of the shape `<374561902345678.1719483022.123456789012345-openerp-42-crm.lead@mail-01>`.

### The reference chain of a notification

When a notification electronic mail is built, the chain placed in its references header is computed as follows:

1. Take the last 32 messages of the same record, excluding the message itself, that have a subtype (which excludes pure logs) and have a message identifier, ordered by identifier descending.
2. Sort them by three keys, all descending: first, "is a public message" (neither the message nor its subtype is internal); second, "is an outgoing kind" (comment, automated notification, incoming or outgoing electronic mail); third, "is not a per-recipient kind" (not a user-specific notification and not an out-of-office answer).
3. Keep the first three of that sorted list and re-sort them by identifier ascending, oldest first.
4. The chain is those three identifiers followed by the identifier of the message being sent.

The purpose of the sort is that mail clients thread on the **public** conversation rather than on internal notes, and that the chain never leaks the existence of an internal note to an external recipient.

---

## 6. Parent selection for a posted message

The rule depends on the model's flat-thread attribute.

### Algorithm

1. If a parent was supplied, verify it: it must be a Message of the same model and the same record. If it is not, treat it as absent.
2. If the model is **not** flat-threaded, stop: the verified parent (possibly none) is the answer.
3. If the model **is** flat-threaded and no valid parent was found, find the ancestor:
   1. Take the last 200 messages of the record that are not user-specific notifications, ordered by date descending then identifier descending. The limit exists because a record flooded by an automated sender must not make posting quadratic.
   2. Re-sort them by three keys, all descending: "is a conversational message" (its type is comment or incoming electronic mail); then the message date, falling back to the creation moment, falling back to the earliest representable moment; then the identifier.
   3. The first of that sorted list is the parent.
4. If the record has no message at all, there is no parent.

Worked example. A record has, in creation order: a log entry (type "system notification"), a customer electronic mail (type "incoming email"), and a tracking entry (type "system notification"). A new comment on a flat-threaded model is parented to the **customer electronic mail**, because conversational messages sort first, even though the tracking entry is more recent.

---

## 7. Field change tracking

### Which fields are tracked

A field is tracked when its definition carries a tracking marker. The marker is either a positive integer, which is also the display order, or the value "true", which means "order 100". A dynamic-property field is additionally tracked when the field that defines its schema is itself tracked and the property field is not explicitly excluded.

The set is computed once per model and cached per acting user and per elevation state, because the set is filtered by field-level access.

### When the comparison happens

1. **Before** a modification, the current value of every tracked field of the record set is snapshotted into a per-transaction store, keyed by model and record. A value already present in the store is **not** overwritten, so the snapshot always holds the value as it was at the *first* write of the transaction.
2. A computed stored field that is about to be recomputed triggers the same snapshot for the fields it computes.
3. At the very end of the transaction, before committing, the comparison runs for every record that has a snapshot.
4. A record whose snapshot is the explicit "discard" marker is skipped entirely. That marker is set at creation, at duplication, and when a caller asks for no tracking.

### The comparison

For each record and for each tracked field, in the order described in the next section:

1. If the field is absent from the snapshot, skip it.
2. Read the current value. For a dynamic-property field, read the full definition-plus-value form rather than the bare value.
3. If the new value equals the old value, skip. If **both** are falsy, skip as well — this is deliberate, so that an empty text becoming a false value does not produce a tracking entry.
4. For a dynamic-property field, additionally skip unless the defining field itself changed; then produce one tracking entry per property that has a value, in reverse definition order, skipping separators and rich-text properties.
5. Otherwise produce one tracking entry.
6. Record the field name as changed.

### Building a tracking entry

The pair (old value, new value) is written into the column pair that matches the field type. See the table in `entities.md`, section 6. Two cases deserve emphasis:

- a **selection** stores the translated label, not the stored key, falling back to the key when no label exists; an empty value stores the empty string;
- a **link** stores the identifier in the integer columns and the display name in the text columns; an empty value stores identifier 0 and an empty name.

An unsupported field type aborts with a programming error naming the field and the type.

### What happens with the result

1. Ask the model for the subtype that these changes trigger. The default is "no subtype".
2. Determine the author: the explicitly registered tracking author when one was set for this record in this transaction, otherwise none (which makes the posting operation compute it).
3. Determine the body: the explicitly registered log message when one was set (even an empty one), otherwise the model's default log message for this set of changes, which is empty by default.
4. **If a subtype was returned**, post a message with that body, that author, that subtype and the tracking entries. This produces a visible entry that notifies the followers of that subtype.
5. **If no subtype was returned but there are tracking entries**, log an internal note with the same body, author and entries. This produces a visible entry that notifies nobody.
6. If there are neither, do nothing.
7. Independently, ask the model for a template to post for these changes, and post or mass-mail it. The default composition mode is "post on a document" for a single record and "mass mail" for several; the default message type is "automated targeted notification"; and the author is notified when mentioned.

### Worked example — a tracked field change producing a tracking entry

Given a model with a link field `stage_id` (the pipeline stage) marked tracked with order 1, a decimal field `expected_revenue` marked tracked with order 20, and a model rule that returns the subtype "Stage Changed" when the stage field is among the changes.

A record currently has stage "New" (identifier 3) and expected revenue 1 500.00.

**When** a user writes stage "Qualified" (identifier 5) and expected revenue 2 000.00 in one operation:

1. Before the write, the snapshot records stage = the record for identifier 3 and expected revenue = 1 500.00.
2. The write happens.
3. At the end of the transaction the comparison runs. Ordering is by descending sequence then descending name, so the insertion order is: expected revenue (order 20) first, then stage (order 1). Display is by ascending sequence, and the table is read newest first, which puts stage before expected revenue on screen.
4. Two tracking entries are produced:
   - for `stage_id`: old integer 3, old text "New", new integer 5, new text "Qualified";
   - for `expected_revenue`: old decimal 1 500.00, new decimal 2 000.00, currency taken from the record's currency companion field if the field is monetary.
5. The model rule returns the "Stage Changed" subtype, so a message is posted carrying both entries with that subtype.
6. The followers subscribed to "Stage Changed" are notified; followers not subscribed to it are not.
7. On screen the message shows: "Stage: New → Qualified" then "Expected Revenue: 1,500.00 → 2,000.00".

**When** the same user writes only the expected revenue:

The model rule returns no subtype (the stage did not change), so an internal note carrying the single entry is logged instead, and nobody is notified.

---

## 8. Tracking display order

Entries must appear in ascending order of the display sequence. The storage table is read newest-identifier-first. Therefore the entries are **created in the reverse order** of their display.

### The sequence of a field

```formula
sequence = the tracking marker of the field, when it is an integer
sequence = 100, when the marker is the value "true"
sequence = 100, when the field is unknown to the model
```

For a dynamic-property field whose own marker is "true", the sequence is inherited from the field that defines its schema, again falling back to 100.

### The insertion sort

The fields are sorted by three keys, **all descending**:

1. the sequence;
2. "is not a dynamic-property field" (so that properties come after ordinary fields of the same sequence);
3. the field name.

### The display sort

When the stored entries are rendered, they are sorted by three keys, **all ascending**:

1. the sequence, defaulting to 100 for an entry whose field no longer exists and whose snapshot carries none;
2. "is a dynamic-property field";
3. the field name, falling back to the name in the snapshot, falling back to the literal `unknown`.

### Worked example

Three tracked fields: `stage_id` with sequence 1, `user_id` with sequence 1, `priority` with sequence 30. All three change at once.

Insertion order (descending sequence, then descending name): `priority` (30), then `user_id` (1, name "user_id" sorts after "stage_id"), then `stage_id` (1).

Stored identifiers are therefore, say, 101 for priority, 102 for the responsible and 103 for the stage.

Reading the table newest-first gives 103, 102, 101 — stage, responsible, priority. The display sort then confirms: stage (1, "stage_id"), responsible (1, "user_id"), priority (30).

---

## 9. Duration tracking and staleness

### Time spent in each value

A model that declares a tracked link field as its *duration field* exposes a map from the identifier of each value that field has taken to the number of seconds the record spent on it.

Precondition: the named field must exist, must be a link field and must be tracked; otherwise the computation aborts with "Field “<field>” on model “<model>” must be of type Many2one and have tracking=True for the computation of duration."

Algorithm:

1. Fetch every tracking entry of that field for the records, joined to its message, ordered by entry identifier ascending. Each yields the moment of the change and the **old** value identifier.
2. Start the clock at the record's creation moment, falling back to the current moment.
3. If a change of that field is currently pending in the transaction store and its pending new value differs from the stored value, append a synthetic entry at the current moment carrying the pending **old** value. This prevents the time spent in the previous value from being credited to the new one during the same transaction.
4. Append a synthetic final entry at the current moment carrying the **current** value, so that the ongoing stay is counted.
5. For each entry in order, add to the accumulator of its old value the whole number of seconds between the previous moment and this entry's moment, then move the previous moment to this entry's moment.

```formula
seconds_in_value_v = Σ over consecutive tracking entries e with old value v of
                     truncate_to_integer( moment(e) − moment(previous entry) )
```

Worked example. A record is created on the first of March at 09:00:00 in stage "New". On the third of March at 09:00:00 it moves to "Qualified". On the fourth of March at 21:00:00 it moves to "Won". The map is computed on the fifth of March at 09:00:00.

- entry 1: moment = 3 March 09:00:00, old value = New. Adds 2 × 86 400 = 172 800 seconds to "New". Previous moment becomes 3 March 09:00:00.
- entry 2: moment = 4 March 21:00:00, old value = Qualified. Adds 129 600 seconds to "Qualified". Previous moment becomes 4 March 21:00:00.
- synthetic final entry: moment = 5 March 09:00:00, old value = Won (the current value). Adds 43 200 seconds to "Won".

Result: New 172 800, Qualified 129 600, Won 43 200. Total 345 600 seconds, which is exactly the four days since creation.

### Staleness

The staleness feature is active only when: the value model of the duration field has a threshold field expressed in days; the record model has a field recording the moment of the last change of that field; and at least one value in play has a non-zero threshold. A zero threshold means "never stale".

```formula
is_stale     = ( last_change_moment + threshold_days ) < now
stale_days   = whole days between now and last_change_moment
```

The moment used is the recorded last-change moment, falling back to the record's creation moment, falling back to the current moment. A record that is not stale reports zero days.

The searchable form restricts the scan to records whose last-change moment is within a configurable number of months (default 12), computes the rotting moment as the last-change moment plus the threshold expressed in days, and returns the records whose rotting moment has passed. Only the two set operators are accepted; anything else aborts with "For performance reasons, use "=" operators on rotting fields." A model that has not configured the feature aborts with "Model configuration does not support the rotting feature".

Worked example. A record entered its current stage on the tenth of May at 14:00. The stage's threshold is 7 days. On the eighteenth of May at 09:00 the record is stale, because 10 May 14:00 plus 7 days is 17 May 14:00, which is before 18 May 09:00. The reported staleness is the whole number of days between 18 May 09:00 and 10 May 14:00, which is 7.

---

## 10. Automatic subscription

Automatic subscription runs at creation and at every modification of a thread-enabled record. It has two independent mechanisms.

### Mechanism one — propagation through the subtype parent relation

Purpose: following a container should subscribe you to its children.

1. Load the automatic-subscription map of the model: the child subtype identifiers, the default subtype identifiers, all internal subtype identifiers, the parent map (child subtype of the container's subtype) and the relation map (container model to the set of field names on this model that point at it).
2. Intersect the relation map with the values actually being written. Only a field that **received a value** in this operation counts.
3. If nothing matched, this mechanism contributes nothing.
4. For each matched container model, fetch the follower rows of the container records named by the written values, together with each follower's customer flag and active flag.
5. For each such follower row, compute the set of subtypes to grant on **this** record:
   - for every subtype the follower has on the container that appears in the parent map, take the corresponding child subtype;
   - plus every subtype the follower has that is not in the parent map but is one of this model's own subtypes.
6. Skip a follower whose contact is archived.
7. If the follower's contact is a customer, remove every internal subtype from the computed set.
8. Record the pair (contact, subtype set).

### Mechanism two — the responsible

Purpose: the person a record is assigned to should follow it and be told.

The default implementation:

1. Look for a field named `user_id` on the model. It must exist, must point at the user model, must be tracked, and must have received a value in this operation.
2. Read that user. If the user is active, return one entry: that user's contact, the **default** subtype set of the model, and, when the user is not the acting user, the shipped assignment notification template.
3. Otherwise return nothing.

Models override this to add their own responsible-like fields.

### Merging and inserting

1. For each entry produced by mechanism two, add the pair only when mechanism one did not already produce one for the same contact — mechanism one wins, because it carries a more precise subtype set.
2. Insert the followers with the existing-follower policy the caller chose: `update` at creation (add missing subtypes, remove none) and `skip` at modification (leave existing followers entirely alone).
3. For each (template, language) pair collected from mechanism two, send the assignment notification to the contacts concerned, rendered in the contact's language.

### The assignment notification

Subject: "You have been assigned to <the record display name>". Body: the shipped assignment template rendered with the access link, the company, the model description and the record; local links in the rendered body are made absolute. Layout: the standard notification layout. Delivered through the notify operation, so it lands in the inbox or in the mailbox according to the recipient's preference.

It is suppressed when: the caller set the silent switch; the platform is still installing; or demonstration data is being loaded.

---

## 11. Default and suggested recipients

Two related computations answer two different questions.

### Default recipients — "who should this template be sent to"

1. Collect, per record: the customer contacts (from the model's contact fields, in order: the single-contact field then the multiple-contact field, deduplicated); the primary address; and the carbon-copy address when the caller asked for it.
2. The primary address is the value of the model's declared primary address field; when that is empty, the first non-empty value among the fields named `email_from`, `x_email_from`, `email`, `x_email`, `partner_email`, `email_normalized`, in that order.
3. Build the ban list: the system contact's address, unless another contact shares it; plus every address of the collection that belongs to the installation (any alias, catch-all, bounce or default-sender address).
4. Remove public contacts and banned addresses from the contact list; remove banned addresses from the address lists.
5. Choose between contacts and addresses:
   - if the model does **not** prioritize addresses, or there is no address at all:
     - if there are contacts with a usable address, use the contacts and no address;
     - else if the contacts without a usable address are exactly as many as the addresses and every address is one of their raw addresses, use those contacts and no address;
     - else use the addresses when there are any, and otherwise the contacts;
   - if the model prioritizes addresses and there are addresses:
     - if the number of addresses equals the number of usable contacts and every address normalizes to one of their normalized addresses, use the contacts;
     - otherwise use the addresses.
6. The result per record is a triple: the carbon-copy list, the comma-joined address list and the contact identifier list.

### Suggested recipients — "who might the user want to add"

1. Start from the default collection above.
2. Add the responsible user's contact, unless that is the acting user.
3. Add the customer contacts.
4. Add the primary address, or a forced address when the caller supplied one.
5. When asked to consider the discussion, find the **last relevant message**: among the record's messages, keep only those whose type is comment or incoming electronic mail **and** whose subtype is the creation subtype of the model or the shipped "Discussions" subtype; sort by date then identifier, newest first; take the first. If none qualifies, no message is used at all — replying-to-all must not be offered on a record that has only logs.
6. From that message add: its direct recipients and its author, when active; its incoming "to" list, its incoming carbon-copy list and its sender address; and the sender address again when it normalizes to something different from the author's normalized address.
7. Remove from the address list everything that already matches a follower or an already-collected contact, comparing both the normalized and the raw address.
8. Resolve the remaining addresses to contacts without creating any, banning the system contact's address and every installation address.
9. Filter the resulting contact list: drop followers, **except** a follower that is also a default customer of the record and is a customer contact; drop banned addresses; drop public contacts.
10. Filter the remaining address list: drop banned addresses and drop anything that matches an address already covered by a follower or a kept contact.
11. Produce one entry per contact (display name when the contact has no name, address, name, contact identifier, empty creation values) and one per address (address, name taken from the model's customer information for that address or from the parsed display name, no contact identifier, the model's customer information as creation values).

---

## 12. Finding or creating a contact from an address

Used by the incoming gateway, by the composer and by the suggested-recipient computation.

### Algorithm

1. For each record and each of its addresses, compute the key: the non-strict normalized form, falling back to the raw input when normalization fails — so that two different malformed inputs stay distinguishable.
2. Determine the company of each record.
3. Gather the model's customer information: a map from normalized address to a set of initial contact values. Merge the caller's additional values on top.
4. Unless the caller disabled it, add every installation address found among the keys to the ban list.
5. Ask the contact model to find or create one contact per input address, passing: the ban list, the per-address initial values (each including the record's company), an optional filter on found contacts, whether creation is allowed, and a preference order.
6. The preference order, applied as a descending sort over found candidates, is a tuple of seven boolean keys, most significant first:
   1. the candidate is the acting user's contact;
   2. the candidate is a follower of the record;
   3. the candidate is **not** a customer (internal users win);
   4. the candidate has a user (portal users beat bare contacts);
   5. the candidate's company equals the record's company;
   6. the candidate's formatted address is exactly one of the inputs;
   7. the candidate has **no** company (company-agnostic contacts are preferred to contacts of a different company).
   The underlying search order, used to break remaining ties, is ascending identifier: the oldest matching contact wins.
7. Map the results back: for each input address, the found or created contact is added to every record that supplied it, deduplicated.

Precondition: when invoked on a record set, the set must match the per-record address map; otherwise the call aborts with "Invoke with either self maching records_emails, either on a void recordset."

---

## 13. Activity deadlines

### From an activity type

```formula
due_date = base_date + delay_count × one_unit( delay_unit )
```

| Quantity | Rule |
|---|---|
| `base_date` | the due date of the activity being completed, when the type's delay origin is "after previous activity deadline" **and** the caller supplied such a date; otherwise today in the acting user's time zone |
| `delay_count` | the type's delay count; may be zero, which makes the due date equal the base date |
| `one_unit` | one calendar day, one calendar week (seven calendar days) or one calendar month |

Month arithmetic is calendar arithmetic and clamps to the last valid day of the target month.

Worked examples, with the acting user in a time zone where today is the 31st of January 2026:

| Delay | Origin | Previous due date supplied | Result |
|---|---|---|---|
| 3 days | after previous activity deadline | none | 3 February 2026 |
| 3 days | after previous activity deadline | 10 February 2026 | 13 February 2026 |
| 3 days | after previous activity completion date | 10 February 2026 (ignored) | 3 February 2026 |
| 2 weeks | after previous activity completion date | — | 14 February 2026 |
| 1 month | after previous activity completion date | — | 28 February 2026 (clamped) |
| 0 days | either | — | 31 January 2026 |

### Worked example — an activity due in three days, completed, chaining its successor

Given:

- an activity type "Call" with delay 3 days, origin "after previous activity deadline", chaining "trigger", and a trigger successor "Follow-up" with delay 7 days, origin "after previous activity deadline";
- a record of a thread-enabled, activity-enabled model;
- the acting user Dana, in a time zone where today is Monday 2 March 2026.

**Scheduling.** Dana schedules an activity of type "Call" on the record. The due date is computed from today, because no previous due date is in play: 2 March plus 3 days = **Thursday 5 March 2026**. Dana is the assignee, so no assignment notification is sent. Dana's contact is subscribed to the record. The due date is after today, so the state is "planned" and no counter is broadcast.

**On the due date.** On 5 March the state becomes "today" for Dana (whose time zone decides). On 6 March it becomes "overdue". The record's activity indicator follows.

**Completion.** On Friday 6 March Dana completes the activity with the feedback "Spoke to the customer, will call back".

1. The chaining is "trigger", so the successor values are prepared **first**, with 5 March (the due date being completed) placed in the context.
2. The successor's type is "Follow-up", whose origin is "after previous activity deadline", so its base date is 5 March, not today. Its due date is 5 March plus 7 days = **Thursday 12 March 2026**.
3. A message is posted on the record from the shipped completion template, authored by Dana, carrying the activity type "Call" and the "Activities" subtype. The rendering shows the type, the summary, the feedback, and — because Dana is the assignee — no "assigned to" line.
4. Any attachment of the activity is moved onto that message.
5. The successor activity is created: type "Follow-up", due 12 March, assignee taken from the type's default user or from the defaults, automated flag inherited from the preparation, previous type "Call".
6. The original activity is archived; its completion date is stamped with the current moment (6 March); its feedback is stored.
7. Dana's live-activity counter drops by one for the completed activity (it was due today or earlier) and the new activity, being due in the future, does not raise it.

Had the successor's origin been "after previous activity completion date", its due date would have been 6 March plus 7 days = 13 March.

Had the chaining been "suggest" instead of "trigger", no successor would have been created; instead the scheduling window would have reopened with "Follow-up" offered among the suggestions, carrying the previous type and the previous due date in its context.

---

## 14. Activity plan deadlines

Each line of a plan computes its own due date from the single plan date.

```formula
line_due_date = plan_date − delay_count × one_unit( delay_unit )   when the trigger is "before plan date"
line_due_date = plan_date + delay_count × one_unit( delay_unit )   when the trigger is "after plan date"
```

The plan date defaults to today in the acting user's time zone. **The delay configured on the activity type is ignored**; the line's own delay is used.

Worked example. A plan "Employee onboarding" with plan date Monday 1 June 2026 and three lines:

| Line | Delay | Trigger | Due date |
|---|---|---|---|
| Prepare the workstation | 5 days | before plan date | 27 May 2026 |
| Welcome meeting | 0 days | after plan date | 1 June 2026 |
| First review | 1 month | after plan date | 1 July 2026 |

The assignee of each line is resolved as follows: when the line's assignment mode is "Default user", the line's fixed user; when it is "Ask at launch", the user chosen in the launching window. A line in "ask at launch" mode with no user chosen produces the blocking error "No responsible specified for <type name>: <summary>." — the summary falling back to a hyphen — and the whole plan refuses to launch.

---

## 15. The activity state

```formula
difference_in_days = due_date − today_in_the_assignee_time_zone

state = "today"    when difference_in_days = 0
state = "overdue"  when difference_in_days < 0
state = "planned"  when difference_in_days > 0
state = "done"     when the activity is archived, whatever the difference
```

"Today in the assignee's time zone" is obtained by taking the current moment in coordinated universal time, converting it to the assignee's time zone when one is set, and keeping only the calendar date. When the assignee has no time zone, the machine's local calendar date is used.

The aggregate form used for searching and grouping computes, per record, the minimum over its live activities of the sign of the difference, using the assignee's time zone stored on each activity and falling back to coordinated universal time. The sign is −1, 0 or 1, and maps to overdue, today and planned. Taking the minimum reproduces the precedence overdue > today > planned exactly.

Worked example. An activity is due on 5 March. Its assignee is in a time zone 13 hours ahead of coordinated universal time. At 20:00 on 4 March in coordinated universal time, it is already 09:00 on 5 March for the assignee, so the state is "today" — even though a reader in coordinated universal time would still call it "tomorrow".

---

## 16. Incoming routing

The complete algorithm that turns a received electronic mail into a record creation, a record update, a bounce, or nothing.

### Phase A — parse

1. Decode the raw bytes into a message structure using the transport policy.
2. Read the message identifier. If the header is absent, generate one of the shape `<timestamp@localhost>` and log it.
3. Read the subject, decoding any encoded words.
4. Read the sender: the first address of the "from" header. Keep the whole header when no address could be parsed.
5. Read the carbon-copy list.
6. Compute the recipient list as the union of the "delivered to", "to", "carbon copy", "resent to" and "resent carbon copy" headers, in formatted form, deduplicated.
7. Compute the "to" list as the union of the "delivered to" and "to" headers only.
8. Compute the **filtered** "to" and carbon-copy lists by removing every address that belongs to the installation. Those filtered lists are what gets stored on the message, so that a later notification does not re-address the platform's own aliases.
9. Read the references and the in-reply-to headers.
10. Read the date. Parse it leniently; a value without a time-zone offset is understood as coordinated universal time; any other value is converted to coordinated universal time. An unparsable date falls back to the current moment with a log line.
11. Find the parent message: first by exact match on the in-reply-to header; if that fails, by matching any of the **last 32** references; newest identifier first in both cases.
12. Derive from the parent: the parent identifier, and an "is internal" flag that is true when the parent's subtype is internal **and** the parent is not an automated notification.
13. Extract the bounce information (phase B).
14. Extract the body and the attachments (phase C).

### Phase B — bounce extraction

A message is a bounce when **any** of the following holds:

- one of its "to" addresses equals a bounce address of one of the installation's alias domains;
- the local part of its sender address, lower-cased, is exactly `mailer-daemon`;
- its content type is the multipart report type, or its content type declares a delivery-status report.

When it is a bounce:

1. Find the embedded original: the first part whose content type is the embedded-message type or the header-only message type; failing that, the first multipart report part.
2. Find the delivery-status part. When it exists and has more than one sub-part, read the final-recipient field of its second sub-part; the value is of the form `<address type>;<address>`, so the address is the part after the semicolon, normalized. Look up the contact with that normalized address.
3. From the embedded original, collect candidate references: every identifier found in its message-identifier header and in the original-message-identifier header of the common corporate mail service. Search for a Message with one of those identifiers, newest creation date first. If none is found, extend the candidate list with the in-reply-to and references headers of the bounce itself, and use the already-found parent when there is one.
4. If a bounced message was found and no contact was resolved, and that message has exactly one notified contact, take that contact and its address as the bounced party.

### Phase C — body and attachment extraction

For a message whose main type is text:

- a plain-text body is wrapped in a preformatted block;
- a rich-text body is sanitized for class attributes only; the full sanitizing happens later when the message body field is written.

For a multipart message, walking the parts in order:

1. Stop early when the message is a bounce and a body has already been found, so that the quoted original does not overwrite the bounce explanation.
2. Replace three malformed content types (`binary/octet-stream`, `*/*`, `bin/plain`) by the generic binary type, with a warning.
3. Note whether an alternative container or a mixed container has been seen.
4. Skip container parts.
5. For a textual part with no declared character set, assume the eight-bit Unicode transformation format rather than plain ASCII.
6. Correct a content type that begins with the three letters of the portable document format to the proper document type.
7. Classify the part:
   - a part with a file name **and** a content identifier becomes an inline attachment, remembering the identifier;
   - a part with a file name, or an explicit attachment disposition, becomes an attachment (named "attachment" when it has no name);
   - a plain-text part becomes the body, appended, unless an alternative container was seen and a body already exists;
   - a rich-text part becomes the body: inside an alternative container it **replaces** the body (keeping the rich version), unless both a rich part and a mixed container have been seen, in which case it is appended;
   - anything else becomes an attachment.

Post-processing of the body:

1. Remove every element carrying the platform's own notification marker, as a class or as a summary attribute. This strips the headers and footers the platform itself added to the message being replied to.
2. For every image whose source is a content identifier that matches one of the extracted attachments, record the attachment's file name on the image so it can be re-linked.

### Phase D — duplicate and loop guards

1. Search for an existing Message with the same message identifier. If one exists, ignore the incoming message and log it.
2. If none exists and the identifier is non-empty, take a transaction-scoped advisory lock keyed on a hash of the identifier. If the lock cannot be taken, another transaction is already processing the same message, so treat it as a duplicate.
3. If any reference of the message contains the loop-detection tag, the message is a reply to one of the platform's own bounces; ignore it.

### Phase E — routing

1. **Bounce.** If the message is a bounce, handle it (see section 18) and return no route.
2. **Reset bounce counters.** Otherwise, because a message did arrive from this sender, reset the bounce counter of every blacklist-enabled record whose normalized address equals the sender's.
3. **Compute the allowed domains.** Read the allowed-domain parameter; when it is non-empty, add every alias-domain name to it. A recipient address whose domain is outside this list can never contribute a local part to alias matching.
4. **Find the replied-to message.** Take the references, or the in-reply-to when there are none; drop any reference containing the reply-to tag; keep the last 32; search for a Message with one of those identifiers, newest identifier first. Its model and record define the candidate reply target.
5. **Forward detection.** If a reply target was found, search for aliases of a **different** model among the "to" addresses (by full address, or by local part for local-part-based aliases). If any exists, the message is a forward, not a reply: discard the reply target, and restrict the valid recipient list to the addresses that matched those aliases.
6. **Reply route.** If a reply target survives, find an alias of the **same** model among the valid recipients, if any; find the user to act as (see below); check the route (see phase F). If the check passes, that single route is the answer. If the check returns an explicit rejection, return no route at all.
7. **Direct write to the catch-all.** If the message has recipients, forget any parent (the reference did not resolve to a route), then test whether it writes directly to the catch-all: with the strict rule, **all** of its "to" addresses must be catch-all addresses; with the relaxed rule, **any** of them. On a match, determine the company: the acting company, unless a matched catch-all belongs to an alias domain bound to companies that do not include it, in which case the first of those. Send the shipped catch-all bounce rendered for that company, with a reference carrying the loop-detection tag and the company address as reply address. Return no route.
8. **Alias routes.** Search every alias matching a valid recipient, by full address or by local part for local-part-based aliases. For each match build a route from the alias: its target model, its forced record (or none), its evaluated default values, the acting user and the alias itself. Check each route, raising on failure. Return every route that passed — one incoming message may legitimately create several records.
9. **Fallback route.** If a fallback model was supplied, forget any parent, find the user to act as, and check a route made of that model, the supplied record, the supplied custom values and no alias. Raising on failure. Return it if it passed.
10. **Unroutable with a catch-all in the recipients.** If there are recipients and the relaxed catch-all test matches, send the catch-all bounce and return no route.
11. **Otherwise** raise: "No possible route found for incoming message from <sender> to <recipients> (Message-Id <identifier>:). Create an appropriate mail.alias or force the destination model."

### Phase F — checking one route

A route is the tuple (model, record, default values, acting user, alias).

1. The model must be named, and must exist. Otherwise warn "target model unspecified" or "unknown target model <model>" and drop the route.
2. If a record is named: it must exist, otherwise warn "reply to missing document (<model>,<record>), fall back on document creation" and clear the record; and the model must accept updates, otherwise warn "reply to model <model> that does not accept document update, fall back on document creation" and clear the record.
3. If there is no record, the model must accept creation; otherwise warn "model <model> does not accept document creation" and drop the route.
4. If there is an alias:
   1. Resolve the author, when not already resolved: look the sender address up among the contacts, without creating one, using the target record as context — or, failing that, the alias's owner record.
   2. Choose the object on which the contact-security check runs: the target record when there is one; else the alias's owner record when the alias names one; else the bare model.
   3. Run the check (phase G). On error, warn "alias <name>: <error>", send the appropriate bounce and reject the route.
5. Return the route.

### Phase G — the contact-security check

| Policy | Rule | Error when it fails |
|---|---|---|
| `everyone` | always passes | — |
| `partners` | the author must have resolved to a contact | "restricted to known authors" |
| `followers` | the object must be a real record — otherwise "incorrectly configured alias (unknown reference record)", flagged as a configuration error — and must have followers — otherwise "incorrectly configured alias", flagged as a configuration error — and the author must be one of them | "restricted to followers" |

A failure flagged as a configuration error sets the alias to invalid and sends the "invalid alias" bounce; a plain rejection leaves the alias status alone and sends the "security" bounce.

### Phase H — finding the user to act as

1. Normalize the sender address. If it cannot be normalized, there is no user.
2. Choose the context record: the alias's owner record when the alias names both a parent model and a parent record; otherwise nothing.
3. Look the address up among contacts in that context, **filtering to contacts that have a user**, without creating anything.
4. The answer is that contact's main user, or nothing.

The acting user defaults to the current one when no user was found.

### Phase I — processing the routes

For each route:

1. Browse the model with automatic subscription and automatic logging both suppressed.
2. Verify once more that the model accepts the operation; otherwise abort with "Undeliverable mail with Message-Id <identifier>, model <model> does not accept incoming emails".
3. When the route came from an alias: if the acting user is a system user, switch to the resolved user, then elevate. This is what makes the created record owned by the right person while the gateway itself runs with system rights.
4. If there is a record and the model accepts updates, call the update operation with the parsed values.
5. Otherwise:
   1. forget the parent, because a newly created record starts a new conversation;
   2. call the creation operation with the parsed values and the route's default values;
   3. on failure, and only when the route came from an alias, open an **independent transaction** and mark the alias invalid with a bounce, then re-raise;
   4. on success, if the alias was not already valid, mark it valid;
   5. the created record becomes the target and the model's creation subtype becomes the subtype of the message to post.
6. Switch to the system user for the posting, so that the real author is computed correctly.
7. Determine the subtype when the creation did not supply one: the internal-note subtype when the message was derived as internal, the "Discussions" subtype otherwise.
8. Determine the additional recipients: when there is a parent message with an author, ping that author if the incoming message is internal, or if that author is an external contact — so a private answer reaches the person it answers.
9. Build the posting parameters from the parsed values, adding the filtered "to" and carbon-copy lists, the subtype and the additional recipients, and removing the purely computational keys (sender, recipients, raw carbon copy, raw "to", references, in-reply-to, the platform's own message identifier header, and the five bounce keys).
10. Post. When the target is not a real record — a message whose parent is not attached to anything — use the notify operation instead of the post operation. Otherwise post, with author subscription skipped when the message has no author.
11. After posting, write the originally parsed direct recipients onto the message. This is deliberately done **after** the notification pass, so that people already addressed by the original electronic mail are not notified twice.

### Worked example — an inbound message on an alias creating a record

Given:

- an alias with local part `jobs`, domain `example.com`, target model a thread-enabled recruitment model whose primary address field is `email_from` and whose display field is `name`, default values that set a department, security policy "everyone", and status "not tested";
- a message arriving with sender `"Erik Nilsson" <erik@candidate.example>`, "to" `jobs@example.com`, subject `Application for the developer position`, no references, one attachment.

Processing:

1. Parsing yields: identifier from the header; subject as above; sender formatted; recipients `jobs@example.com`; filtered "to" empty, because the only recipient belongs to the installation; no parent.
2. The message is not a bounce, has no duplicate and no loop reference.
3. No reply target, so the reply branch is skipped. The recipient is not a catch-all address.
4. The alias search matches the alias by full address. One route is built: the recruitment model, no forced record, the department default, the acting user (no contact matched `erik@candidate.example`, so the gateway user), and the alias.
5. The route check passes: the model exists, accepts creation, and the policy is "everyone".
6. Processing: the record is created with the display field set to the subject and the primary address field set to the sender; the department default is applied. The alias status becomes **valid**.
7. The creation subtype of the model, if it defines one, becomes the message subtype; otherwise the "Discussions" subtype is used.
8. A Message of type "incoming email" is posted on the new record, with the parsed body, the attachment, the sender address, the empty filtered "to" and carbon-copy lists, and the message identifier from the header.
9. The notification pass runs with the new record's followers, which at this point are whoever automatic subscription added — typically nobody, because creation ran with subscription suppressed, plus the responsible if the default values set one.

Had the sender been unknown and the policy been "Authenticated Partners", step 5 would have failed with "restricted to known authors", the alias status would have stayed "not tested", and a bounce with the security body would have been returned to the sender.

---

## 17. Loop detection

Purpose: an automatic replier answering the platform's own notifications must not be able to create an unbounded number of records or messages.

### Parameters

| Parameter | Default | Meaning |
|---|---|---|
| `mail.gateway.loop.minutes` | 120 | The width of the observation window, in minutes. |
| `mail.gateway.loop.threshold` | 20 | The number of records or messages above which the sender is considered to be looping. |

### Algorithm

1. Read the sender address. With no sender there is no loop.
2. If the normalized sender is listed as an allowed gateway sender, stop: no loop.
3. Compute the window start as the current moment minus the configured number of minutes.
4. Group the candidate routes by model. A route with a record identifier of zero means "create"; a route with a real identifier means "update".
5. For each model that declares a loop-detection filter:
   1. **Creation side.** If any route for this model creates, ask the model for its loop-detection filter for the normalized sender. The default filter is "the model's primary address field contains the normalized sender"; a model without a primary address field declares none and logs it. Count the records of the model created inside the window that match the filter. A count at or above the threshold means a creation loop.
   2. **Update side.** If there are routes updating records of this model, and no creation loop was already found, count, **per record**, the messages of type "incoming email" created inside the window on those records, restricted to the resolved author when one exists and to the raw or normalized sender address otherwise. A count at or above the threshold for any single record means an update loop.
6. On either loop: log which kind was detected, render the shipped "too many messages" body with the recipient list, and send a bounce whose references are the incoming identifier followed by a freshly generated identifier carrying the loop-detection tag. Return "ignore the message".

The second identifier is what makes step 3 of phase D recognize a reply to this very bounce and drop it silently, so the bounce itself cannot start a new loop.

Worked example. An automatic replier sends 25 messages in 90 minutes to `jobs@example.com`. The first 20 create records. On the 21st, the creation count inside the 120-minute window is 20, which is at or above the threshold, so the message is ignored and a single bounce is sent. If the replier answers that bounce, the answer carries the loop-detection tag in its references and is dropped without any further bounce.

---

## 18. Bounce detection and the bounce counter

### Propagating a bounce

Given a parsed bounce carrying a bounced address, a bounced contact, a bounced message and its references:

1. If there is no bounced address, log that no routing is done and stop.
2. Determine the originally addressed record from the bounced message's model and record, when both are set and the model exists.
3. For **every** blacklist-enabled model except the behavior itself, search its records whose normalized address equals the bounced address, and call the bounce-received hook on them. Remember whether the originally addressed record was among them.
4. If the originally addressed record exists, was not already handled in step 3, and is thread-enabled, call the bounce-received hook on it as well.
5. If a bounced message was found and either an address or a contact was resolved, update the matching Notifications of that message — those whose recipient contact is the bounced contact, or whose recipient address equals the bounced address — setting the failure reason to the plain-text body of the bounce, the failure type to "bounce" and the status to "bounced".
6. Log which case applied.

### The counter

The default implementation of the bounce-received hook on the blacklist behavior increments the record's bounce counter by one. The reset hook sets it back to zero.

```formula
new_bounce_counter = old_bounce_counter + 1
```

The counter is reset to zero for every blacklist-enabled record whose normalized address equals the sender of **any** successfully received, non-bounce incoming message. The reasoning is that receiving something from an address proves the address works.

A channel treats a member whose bounce counter has reached **ten** as undeliverable and removes them from the channel.

### Worked example — a bounce raising the party's counter

Given a contact Fatima whose normalized address is `fatima@example.org` and whose bounce counter is 2; a notification electronic mail was sent to her for message *M* on record *R*.

1. The recipient's system returns the message. The bounce arrives at `bounce@example.com`.
2. The detection recognizes it, because a "to" address matches a bounce address of an alias domain.
3. The delivery-status part yields the final recipient `rfc822;fatima@example.org`, normalized to `fatima@example.org`; the contact Fatima is found.
4. The embedded original yields the message identifier of *M*; the Message *M* is found; its model and record give *R*.
5. Every blacklist-enabled model is scanned for the normalized address. The contact model is blacklist-enabled, so Fatima's bounce counter goes from 2 to **3**. Any other blacklist-enabled record with the same address — an applicant record, a lead — also has its counter raised.
6. *R* itself, if it is blacklist-enabled and was not already caught, has its counter raised too.
7. The Notification of *M* for Fatima becomes: status "bounced", failure type "bounce", failure reason the plain-text body of the bounce message.
8. Message *M* is re-broadcast to its author's client, so the delivery-error badge appears immediately.
9. No route is produced: the bounce itself never creates or updates a record.

Later, Fatima replies to something from a working address. The reset pass finds her normalized address among the blacklist-enabled records with a positive counter and sets every such counter back to **0**.

---

## 19. Outgoing batching and relay selection

Sending is batched so that one connection carries many messages.

### Grouping

1. Read, without prefetching, the identifier, the sender, the chosen relay and the alias domain of every mail in the set.
2. Normalize the sender: split and re-format the sender value and take the first result; fall back to the raw value when nothing parses. This protects against a sender that was formatted twice.
3. Build the first grouping key: (chosen relay, alias domain, normalized sender, the set of relays this mail's creator is allowed to use).
4. For each first-level group, resolve the relay:
   - when the mail already names a relay, keep it and use the normalized sender as the transport sender;
   - otherwise ask the relay model to find one for the normalized sender among the allowed set, contextualized with the alias domain's default sender address and bounce address. The answer is a relay and a transport sender, which may differ from the message sender when the relay's filter does not accept it.
5. Build the second grouping key: (resolved relay, alias domain, transport sender).
6. Split each second-level group into batches of the configured session batch size (default 1000).

### The allowed relay set

```formula
allowed = relays without an owner
          when the batch has several distinct creators
          or when personal relays are disabled by system parameter

allowed = relays without an owner, plus relays owned by the creator of the message
          otherwise
```

A mail that names a relay outside its allowed set is refused at creation with "You may not create a message using another user's mail server.", and the sending pass refuses the whole batch with "Unauthorized server for some of the sending mails."

### The queue selection

```formula
queue = mails whose state is "outgoing"
        and ( scheduled moment is empty or scheduled moment ≤ now )
```

ordered by identifier, limited to the batch size (default 1000, configurable). When a specific set of identifiers is requested, the limit is ten times the batch size and the result is intersected with that set. The identifiers are then sorted ascending before sending, so that processing order is deterministic.

Progress is reported to the scheduling framework after each mail when auto-commit is on, and once at the end otherwise. The total reported is the number selected when that is below the batch size, and a fresh count of the whole queue otherwise.

---

## 20. The personal relay throttle

A relay owned by a user is throttled so that the user's own account is not flagged as a bulk sender.

### Per-minute accounting

Two fields on the relay hold the state: the minute currently being counted, and the number of recipients already sent in it.

1. Compute the current minute by truncating the current moment to the minute.
2. If the stored minute is **older** than the current minute, the accounting is stale: set the stored minute to the current minute and the count to zero.
3. If the stored minute is **newer** than the current minute, something wrote the field by hand: reset it the same way and log an error.

### Splitting

Process the mails oldest first (by creation moment, then identifier). For each mail, let *r* be the number of contact recipients, or one when there is none:

| Condition | Action |
|---|---|
| the count is already at or above the maximum | delay the whole mail |
| the count plus *r* would exceed the maximum | **split**: copy the mail, giving the copy the first (maximum − count) recipients and keeping the rest on the original; move the notifications of those recipients onto the copy; add (maximum − count), or one when that is zero, to the count; send the copy; delay the original |
| otherwise | send it and add *r* to the count |

Because the first sub-mail of a record also carries the free "to" and carbon-copy addresses, a split may cause two transport messages where one was intended; the original is therefore stripped of its free "to" and carbon-copy values when it is split.

### Delaying

Walk the delayed mails in order, keeping a running count that starts from the relay's current count:

1. If the running count is below the maximum, add this mail's recipient count to it.
2. Otherwise reset the running count to this mail's recipient count and advance the target minute by one.
3. Stamp the mail's scheduled moment with the target minute.

Finally, wake the sending job at the earliest delayed moment plus 59 seconds, so that it runs once that minute has fully elapsed.

Worked example. The maximum is 10 per minute. The relay's count for the current minute is 7. Three mails are queued: *A* with 2 recipients, *B* with 5 recipients, *C* with 1 recipient.

- *A*: 7 + 2 = 9, at most 10, so *A* is sent and the count becomes 9.
- *B*: 9 is below 10 but 9 + 5 = 14 exceeds it. *B* is split: a copy takes the first 1 recipient (10 − 9), the original keeps 4 and loses its free addresses; the count becomes 10; the copy is sent; the original is delayed.
- *C*: the count is 10, at or above the maximum, so *C* is delayed.
- Delaying: the running count starts at 10. For the original *B* (4 recipients) the running count is not below the maximum, so it resets to 4 and the target minute advances by one; *B* is scheduled for the next minute. For *C* (1 recipient) the running count 4 is below 10, so it becomes 5 and *C* is scheduled for the same next minute.
- The job is woken at the next minute plus 59 seconds.

---

## 21. Attachment size and the link fallback

An electronic mail that would be too large has its attachments replaced by download links.

### The size estimate

```formula
estimated_size = length_in_bytes( serialized_headers )
               + length_in_bytes( body )
               + Σ over attachments of ( attachment_size × 8 ÷ 6 )
               + 10240
```

The factor 8 ÷ 6 accounts for the transport encoding, which represents six bits of data in eight bits of output. The constant 10 240 bytes is a safety margin.

### The rule

1. Remove from the attachment set every attachment that the body already references by a content or image link, because it is displayed inline and must not be duplicated.
2. Convert every attachment that is a pure external link — it has an address, no stored bytes, and the address uses the plain, secure or file-transfer scheme — into a link: generate an access token for it, render the shipped attachment-link block, append it to the body, and drop the attachment.
3. Among the remaining attachments, consider those that belong to a **business record** (they have a model and a record, and that model is not the message model). If the estimated size exceeds the relay's maximum message size expressed in bytes, generate access tokens for them, render the link block, append it to the body and drop them.
4. Read the remaining attachments' bytes in one pass, sorted by identifier ascending so that the order matches what the user saw when uploading, and skip any whose content is absent.

```formula
maximum_size_in_bytes = relay_maximum_size_in_megabytes × 1024 × 1024
```

Worked example. A relay allows 25 megabytes, that is 26 214 400 bytes. The headers serialize to 800 bytes, the body is 12 000 bytes, and there are two attachments of 9 000 000 and 11 000 000 bytes belonging to the source record.

```formula
estimated_size = 800 + 12000 + (9000000 + 11000000) × 8 ÷ 6 + 10240
               = 800 + 12000 + 26666666.67 + 10240
               = 26689706.67 bytes
```

which exceeds 26 214 400, so both attachments are replaced by links.

---

## 22. Template rendering

### The two engines

| Engine | Syntax | Used for |
|---|---|---|
| inline placeholder | `{{ expression }}` with an optional default after a separator | subjects, text-message bodies, sender, recipients, reply address, scheduled date |
| structured template | a markup template evaluated against the record | rich-text bodies, notification layouts, digest bodies |

A third mode, "structured template by reference", renders a stored view instead of a raw string.

Calling the renderer with anything other than these three engines aborts with "Template rendering supports only inline_template, qweb, or qweb_view (view or raw); received <engine> instead." Calling it with something other than a list of record identifiers aborts with "Template rendering should only be called with a list of IDs. Received “<value>” instead."

### The evaluation context

Both engines receive the same base context:

| Name | Content |
|---|---|
| `ctx` | the acting context |
| `object` | the record being rendered against |
| `user` | the acting user |
| `env` | the acting environment |
| `format_addr` | the address-formatting helper |
| `format_date`, `format_datetime`, `format_time` | locale-aware date and time formatting; an unknown locale falls back to the raw value |
| `format_amount` | locale-aware monetary formatting |
| `format_duration` | duration formatting |
| `is_html_empty` | the emptiness test for rich text |
| `slug` | the address-fragment builder for a record |

plus the shared template globals. The caller may add or override entries.

### The safety rule

A template is *unsafe* when it contains an expression that is not in the allowed list. The allowed list, for a caller who is not a template editor, is exactly:

`object.name`, `object.contact_name`, `object.partner_id`, `object.partner_id.name`, `object.user_id`, `object.user_id.name`, `object.user_id.signature`.

1. If the template contains **no** unsafe expression, it is rendered by direct field lookup, without evaluating anything. Each expression is split on dots, the first segment is discarded (it is the record itself) and the remaining segments are followed field by field; a missing key yields the declared default. An expression that is not allowed in this mode aborts as a syntax error.
2. If the template **does** contain an unsafe expression and the acting caller is restricted, rendering is refused with "Only members of <group name> group are allowed to edit templates containing sensible placeholders".
3. Otherwise the full engine runs.

A caller is *restricted* when the model allows unrestricted rendering, the acting user is not a template editor, and the restriction setting is switched on.

The same check runs at creation, at modification and at translation of a template, so an unprivileged user can never store a dynamic template.

### Post-processing

When the post-processing option is set, every local link in the result is made absolute against the base address of the record:

- the source attribute of images and of the two vector fill elements, when it starts with a single slash;
- the target attribute of links, when it starts with a single slash;
- addresses used in style declarations and in background attributes.

A link already carrying a scheme, an address starting with two slashes, and inline data are all left untouched. The base address is that of the record when a model was given; otherwise the configured base address.

### Failures

A failure of the structured engine aborts with "Failed to render template: <reference>". A failure of the inline engine aborts with "Failed to render inline_template template: <text>\nError details: <error>".

### The preview line

A preview line is prepended to an outgoing body as a hidden block: a container with no display, a one-pixel font, zero height, zero width and zero opacity. The preview text itself is trimmed and converted from inline-placeholder syntax to the structured syntax before insertion, so it can contain placeholders too.

---

## 23. Channel unread counters

### The counter

```formula
unread_count = count of messages M such that
                   M.model  = the channel model
               and M.record = this channel
               and M.type ∉ { "system notification", "user specific notification" }
               and M.identifier ≥ member.new_message_separator
```

The separator is an identifier, not a date. Its meaning is "the first message I have not read".

### Marking read up to a message

Given a target identifier *t*:

1. Find the newest message of the channel whose identifier is at most *t*. If there is none, do nothing.
2. Let *m* be that message. If the member's current last-seen identifier is strictly lower than *m*, write: the fetched message becomes the maximum of its current value and *m*; the last-seen message becomes *m*; the last-seen moment becomes now.
3. Set the separator to *m* + 1.

If the separator is already *m* + 1, nothing is written; the member is simply re-broadcast with the current counters so that an out-of-step client re-synchronizes.

### Posting your own message

After a member posts message *m* in a channel, the posting hook sets their last-seen message to *m* without notifying, and their separator to *m* + 1. An author never sees their own message as unread.

### Worked example — a channel of three members and the unread counters

Given a channel of type "channel" with three members: Gita, Hugo and Iris. No message exists yet, so all three separators are 0 and all three counters are 0.

1. **Gita posts message 101** (type comment).
   - Gita's separator becomes 102 by the posting hook. Her counter is the number of messages with identifier ≥ 102, that is 0.
   - Hugo's separator is still 0. His counter is the number of qualifying messages with identifier ≥ 0, that is 1.
   - Iris: likewise 1.
   - The channel's last-interest moment is updated, which re-pins the channel for anyone who had unpinned it.
2. **Hugo opens the channel and marks it read up to 101.**
   - The newest message at or below 101 is 101. His last-seen identifier 0 is below 101, so his last-seen message becomes 101, his fetched message becomes 101 and his last-seen moment is stamped.
   - His separator becomes 102. His counter becomes 0.
   - The channel type is "channel", which is **not** in the set of types that broadcast read receipts, so only Hugo's own client is told. In a direct chat or a group, the whole channel would have been told.
3. **Iris posts message 102 and a system notification 103.**
   - Iris's separator becomes 103 by the posting hook; but message 103 is a system notification, which the counter excludes, so her counter is 0.
   - Gita's separator is 102; qualifying messages with identifier ≥ 102 are {102}; her counter is 1.
   - Hugo's separator is 102; his counter is likewise 1.
4. **Gita mutes the channel until tomorrow.** Her counter is unaffected — muting suppresses notifications, not counting. She simply receives no browser push and no inbox entry for new messages until the mute expires.
5. **Hugo unpins the channel.** His unpin moment is stamped. His counter is unaffected. The moment anyone posts a non-notification message, the channel's last-interest moment rises above his unpin moment and the channel re-pins itself for him.

---

## 24. Channel display name

```formula
display_name = channel.name                              when the name is set
display_name = list( first three member names )          when the name is empty
             + "1 other" or "<n> others" when member_count > 3
```

The member names are taken from the first three members **by identifier**, using the contact name or, for a guest, the guest name. The count of extras is the member count minus three. The pieces are joined with the locale's list separator, which is not necessarily a comma.

Member-based naming applies only to the types that declare it, which is the group type.

Worked example. A group with five members created in the order Jonas, Kira, Liam, Mira, Noor and no name displays "Jonas, Kira, Liam and 2 others". With exactly four members it displays "Jonas, Kira, Liam and 1 other". With three or fewer it displays "Jonas, Kira and Liam".

---

## 25. Live chat operator assignment

The algorithm that answers "who should take this visitor".

### Inputs

| Input | Meaning |
|---|---|
| the live chat channel | supplies the candidate operators and the capacity rule |
| the previous operator | the contact the visitor last spoke to, when the visitor is returning |
| the visitor's language | for language matching |
| the visitor's country | for country matching |
| the requested skills | supplied by a chatbot forwarding step, empty otherwise |
| an explicit candidate list | supplied by the caller, otherwise the channel's available operators |

### Step 0 — clean up

Delete the call sessions that have gone stale, so that an operator who crashed out of a call is not wrongly treated as busy.

### Step 1 — the candidate set

The candidates are the explicit list when the caller supplied one, otherwise the channel's **available** operators. Availability requires all three of:

1. the operator's presence status is exactly "online";
2. the channel's session mode is unlimited, **or** the operator's ongoing-session count for that channel is strictly below the maximum;
3. the channel does not block assignment during calls, **or** the operator is not in a call.

If the candidate set is empty, there is no operator.

### Step 2 — the load table

One query produces, for every candidate contact that currently has at least one live session, three values:

| Value | Definition |
|---|---|
| count | the number of **distinct** live chat sessions in which this contact appears in the participant history, restricted to sessions whose end moment is empty and whose last-interest moment is within the last **30 minutes** |
| in a call | true when the contact has at least one call session anywhere |
| contact | the contact |

The rows are ordered by three keys:

1. descending on the composite condition "the count is below 2 **or** the operator is not in a call" — which places, first, everyone who is either lightly loaded or free, and last, the operators that are both in a call and already carrying two or more chats;
2. ascending on the count;
3. descending on "is not in a call".

A candidate absent from the table has **no** live session at all in the last 30 minutes.

### Step 3 — the returning visitor

If the previous operator is among the candidates:

- if they have no row in the load table, return them;
- if their count is below 2, return them;
- if they are not in a call, return them;
- otherwise fall through to the preference search.

So a returning visitor gets the same person unless that person is simultaneously in a call and already handling two or more chats.

### Step 4 — the buffer

Collect the candidates that were assigned a **still-open** session within the last **120 seconds**. These are "failing the buffer": they have just been given work and should not be given more, unless they are the only ones who fit the preference being evaluated.

### Step 5 — the preference ladder

Eight preference lines are tried in order. Each line is a set of conditions that must **all** hold. The first line that leaves at least one candidate wins.

| Order | Conditions |
|---|---|
| 1 | same language **and** holds every requested skill |
| 2 | same language **and** holds at least one requested skill |
| 3 | same language |
| 4 | same country **and** holds every requested skill |
| 5 | same country **and** holds at least one requested skill |
| 6 | same country |
| 7 | holds every requested skill |
| 8 | holds at least one requested skill |

"Same language" means the operator's contact language equals the visitor's language, **or** the visitor's language is among the operator's declared additional live chat languages. "Same country" means the operator's contact country equals the visitor's country. With no requested skills, "holds every requested skill" is vacuously true for everyone, so line 1 degenerates to line 3.

Inside the winning line: if any surviving candidate respects the buffer, restrict to those; otherwise keep them all. Then apply step 6.

If **no** line leaves a candidate, apply step 6 to the whole candidate set.

### Step 6 — the least busy

1. Restrict the load table to the candidates under consideration.
2. If any candidate has **no** row in the load table — no live session in the last 30 minutes — pick one of those **at random**. Randomness spreads the load between equally idle operators.
3. Otherwise take the first row of the load table (the one the ordering placed first), read its pair (count, in a call), collect every candidate whose row has exactly that same pair, and pick one of them at random.

### Worked example — a live chat routed to the available operator with the fewest sessions

Given a live chat channel with the session mode "Limited", a maximum of 3 sessions per operator, no blocking during calls, and four operators: Priya, Quentin, Rosa and Sam. A visitor arrives with language `fr_FR` and country France. No chatbot is involved, so no skills are requested and there is no previous operator.

Current facts:

| Operator | Presence | Live sessions in the last 30 minutes | In a call | Contact language | Contact country | Additional live chat languages |
|---|---|---|---|---|---|---|
| Priya | online | 3 | no | `en_GB` | United Kingdom | — |
| Quentin | online | 1 | yes | `fr_FR` | France | — |
| Rosa | online | 2 | no | `fr_FR` | Belgium | — |
| Sam | away | 0 | no | `fr_FR` | France | — |

1. **Availability.** Priya's count is 3, which is not below the maximum of 3, so she is **not** available. Sam's presence is "away", not "online", so he is **not** available. Quentin and Rosa are available. The candidate set is {Quentin, Rosa}.
2. **Load table.** Quentin: count 1, in a call true. Rosa: count 2, in a call false. Ordering: the first key is "count below 2 or not in a call" — true for Quentin (count 1 < 2) and true for Rosa (not in a call), so it does not separate them. The second key is the ascending count: Quentin 1, Rosa 2. Quentin sorts first.
3. **Returning visitor.** None.
4. **Buffer.** Suppose neither was given a session in the last 120 seconds, so nobody fails the buffer.
5. **Preference ladder.** No skills are requested, so line 1 reduces to "same language". Quentin's contact language is `fr_FR` — a match. Rosa's is also `fr_FR` — a match. Line 1 leaves {Quentin, Rosa}.
6. **Least busy.** Both have a row in the load table, so the random-idle branch does not apply. The first row is Quentin's, with the pair (1, true). Rosa's pair is (2, false), which differs. Only Quentin matches. **Quentin is chosen.**

Variant A — the channel blocks assignment during calls. Quentin is in a call, so he is not available at step 1; the candidate set is {Rosa} and Rosa is chosen.

Variant B — Rosa has 0 live sessions instead of 2. She then has no row in the load table, so step 6 picks at random among the candidates with no row, which is {Rosa}. Rosa is chosen. This is the "fewest sessions" case in its purest form.

Variant C — Quentin was given a session 30 seconds ago. He fails the buffer. Line 1 leaves {Quentin, Rosa}; restricting to those respecting the buffer leaves {Rosa}; Rosa is chosen even though Quentin is less loaded.

Variant D — the visitor's language is `de_DE`. Lines 1 to 3 leave nobody. Line 4 is "same country": Quentin is in France, Rosa in Belgium, so line 4 leaves {Quentin}; Quentin is chosen. Had neither been in France, lines 7 and 8 would also leave nobody (no skills requested makes line 7 vacuous and therefore non-empty — line 7 would leave both), and the least-busy rule would decide.

### What happens when nobody is found

No message is posted at all, so a chatbot script can continue with fallback steps. The session's failure marker becomes "no one available", which makes the outcome "No one Available".

### What happens when someone is found

1. The current step's message is posted, if the step has one.
2. The human is added as a member with the participant type "agent" and, when the forwarding step names skills, with those skills; the skills are also recorded on the session.
3. The bot is removed from the session, without a leave message.
4. The session is renamed to "<visitor display name> <operator live chat name or real name>".
5. The failure marker is reset to "never answered".
6. The session is broadcast to the new operator and pinned for them.

---

## 26. Live chat capacity and durations

### Ongoing sessions per operator and channel

```formula
ongoing_sessions( operator , channel ) =
    count of channel-member rows M such that
        M.contact = operator's contact
    and M.session belongs to this live chat channel
    and M.session has no end moment
    and M.session last-interest moment ≥ now − 15 minutes
```

### Remaining capacity of a channel

```formula
eligible          = operators, minus those in a call when the channel blocks assignment during calls
total_capacity    = max_sessions × count( eligible )
used_capacity     = Σ over eligible operators of ongoing_sessions( operator , channel )
remaining         = max( 0 , total_capacity − used_capacity )
```

Worked example. A channel allows 5 sessions per operator and has 3 operators; one of them is in a call and the channel blocks assignment during calls. The other two carry 4 and 1 ongoing sessions.

```formula
eligible        = 2
total_capacity  = 5 × 2 = 10
used_capacity   = 4 + 1 = 5
remaining       = max( 0 , 10 − 5 ) = 5
```

### Session duration for a participant

```formula
session_duration_hours = ( session_end_moment − participant_arrival_moment ) ÷ 3600
```

where the session end moment is the recorded end when the session is closed; otherwise the creation moment of the last message of the session; otherwise the current moment.

### Response time of an operator

The first time an operator posts a message in a session, the response time is stamped:

```formula
response_time_hours = ( moment_of_first_message − operator_arrival_moment ) ÷ 3600
```

It is written once and never overwritten.

### Message count per participant

Incremented by one for every message the participant posts whose type is neither "system notification" nor "user specific notification".

### Call metrics per participant

```formula
has_call            = 1 when the participant took part in at least one call, else 0
call_duration_hours = Σ over the participant's calls of the call duration in hours
```

The "has call" value is stored as a number precisely so that reports can both sum it (giving the number of sessions with calls) and average it (giving the share of sessions with calls).

---

## 27. Digest indicators

### The three windows

Let *now* be the current moment, expressed in the time zone of the company's working calendar when one is configured.

| Column | Current window | Comparison window |
|---|---|---|
| Last 24 hours | [ now − 1 day , now ] | [ now − 2 days , now − 1 day ] |
| Last 7 Days | [ now − 1 week , now ] | [ now − 2 weeks , now − 1 week ] |
| Last 30 Days | [ now − 1 month , now ] | [ now − 2 months , now − 1 month ] |

Every window is half-open: the start is included and the end is excluded, because every indicator filters on "date ≥ start and date < end".

### The margin

```formula
margin = round( ( current_value − previous_value ) ÷ previous_value × 100 , 2 decimals )
```

with two exceptions that both yield exactly zero: the two values are equal; or either of them is zero. The second exception avoids a division by zero and avoids reporting an infinite improvement from nothing.

Worked example. Current value 27, previous value 20.

```formula
margin = round( ( 27 − 20 ) ÷ 20 × 100 , 2 ) = round( 35.0 , 2 ) = 35.00
```

Current value 27, previous value 0 gives a margin of 0. Current value 0, previous value 20 gives a margin of 0 as well.

### Formatting a value

| Value type | Rule |
|---|---|
| whole number | shown as is |
| decimal | shown with exactly two decimals |
| monetary | abbreviated to a compact form (thousands, millions) and then prefixed or suffixed with the currency symbol according to the currency's symbol position |

### The generic company-scoped indicator

Most indicators are computed by one shared routine:

1. Read the window bounds and the set of companies. The set is the digest's company; when the digest has no company, the acting company is added.
2. Determine the company field of the target model: the multiple-company field for the user model, the single-company field for everything else.
3. Build the filter: the company field is in the set; the chosen date field is at or after the start; the chosen date field is strictly before the end; plus any extra filter.
4. Group by company and either count the rows or sum a chosen field.
5. Assign to the digest the value of its own company, defaulting to zero.

### The two indicators of this domain

| Indicator | Target model | Date field | Extra filter | Aggregation | Company scoped |
|---|---|---|---|---|---|
| Connected Users | the user model | the last-login moment | none | count | yes, on the multiple-company field |
| Messages Sent | the message model | the creation moment | subtype is the shipped "Discussions" subtype **and** type is one of comment, incoming electronic mail, outgoing electronic mail | count | **no** |

The second is deliberately not company-scoped: it counts every conversational message of the installation in the window.

### The three live chat indicators

| Indicator | Definition |
|---|---|
| Percentage of Happiness | the share of live chat ratings in the window that are the top rating, expressed as a percentage with two decimals |
| Conversations handled | the number of live chat sessions of the window |
| Time to answer (sec) | the average first-answer time of the window, expressed in seconds with two decimals |

### Worked example — a digest computing a weekly indicator

Given a digest with periodicity "weekly", one company, the indicator "Messages Sent" switched on and the indicator "Connected Users" switched off. The digest is sent on Monday 8 June 2026 at 07:00 in the company's calendar time zone.

The three columns are computed for the indicator "Messages Sent":

| Column | Current window | Current count | Comparison window | Previous count | Margin |
|---|---|---|---|---|---|
| Last 24 hours | 7 June 07:00 → 8 June 07:00 | 4 | 6 June 07:00 → 7 June 07:00 | 5 | round((4−5)÷5×100, 2) = −20.00 |
| Last 7 Days | 1 June 07:00 → 8 June 07:00 | 143 | 25 May 07:00 → 1 June 07:00 | 110 | round((143−110)÷110×100, 2) = 30.00 |
| Last 30 Days | 8 May 07:00 → 8 June 07:00 | 602 | 8 April 07:00 → 8 May 07:00 | 655 | round((602−655)÷655×100, 2) = −8.09 |

Each count is the number of Messages created in the window whose subtype is "Discussions" and whose type is comment, incoming electronic mail or outgoing electronic mail. The switched-off indicator contributes no column at all. An indicator the recipient may not read is dropped from that recipient's digest.

After sending:

1. The slow-down check runs, because the sending is automatic. The look-back for a weekly digest is 7 days. If **no** recipient logged in between 1 June and 8 June, the digest carries the line "We have noticed you did not connect these last few days. We have automatically switched your preference to monthly Digests." and its periodicity becomes "monthly".
2. The next run date becomes today plus the (possibly new) interval: 15 June if it stayed weekly, 8 July if it became monthly.
3. One rotating tip is consumed: the first tip the recipient has not yet received and whose group they belong to is rendered into the message and the recipient is added to that tip's "already received" list.
4. The electronic mail is created with the subject "<company name>: <digest name>", the one-click unsubscribe headers, and the automatic-deletion flag.

---

## 28. Text message number sanitizing and suppression

### Finding the number of a record

1. Determine the candidate fields: the forced field when the caller named one, otherwise the model's declared number fields (conventionally the mobile field then the landline field).
2. For each candidate field in order, format the value to the international form. The first that formats successfully wins; the answer is (the record's own customer contact, the formatted number, the raw value, "not from the contact", the field name).
3. If none formats and the record has a customer contact, try the contact's own number fields in order. The answer is (that contact, the formatted number or nothing, the contact's value, "from the contact", the field name, defaulting to the landline field name when nothing formatted).
4. If there is no contact either, the answer is (no contact, nothing, the first non-empty candidate value or nothing, the first candidate field name).

### Sanitizing free numbers typed in the window

Each comma-separated entry is trimmed and formatted against the target record, or against the acting user when there is no single target. If **any** entry fails to format, the whole operation aborts with "Following numbers are not correctly encoded: <list>".

### Suppression at batch preparation

For each target record, in this exact order of precedence:

| Order | Condition | Resulting state | Failure type |
|---|---|---|---|
| 1 | the number formats **and** the exclusion list is in use **and** the formatted number is on the suppression list | `canceled` | `sms_blacklist` |
| 2 | the number formats **and** the record is opted out | `canceled` | `sms_optout` |
| 3 | the number formats **and** the same formatted number already occurred earlier in this batch | `canceled` | `sms_duplicate` |
| 4 | the number does not format **and** a raw value exists | `canceled` | `sms_number_format` |
| 5 | the number does not format **and** no raw value exists | `canceled` | `sms_number_missing` |
| 6 | otherwise | `outgoing` | none |

The stored number is the formatted one when it exists, otherwise the raw value, so that the operator can see what was attempted. Duplicate detection uses the **order of the record set**, so the first occurrence is kept and every later one is cancelled.

### Worked example — a text message refused for a suppressed number

Given a suppression list containing `+32470112233`, and three contacts targeted in one batch: Tomas with the mobile number `0470 11 22 33` in Belgium, Ulla with the mobile number `0470/11.22.33` in Belgium, and Viktor with the mobile number `not a number`.

1. Tomas's number formats to `+32470112233`. It is on the suppression list and the exclusion list is in use, so his row is created with the state **cancelled** and the failure type **blacklisted**. No provider call is made for him. His notification, when the batch keeps a log, is written with the status "cancelled".
2. Ulla's number formats to the same `+32470112233`. Rule 1 fires again for her as well — the suppression check is evaluated per record and does not depend on order — so she is also cancelled as blacklisted. Had the number not been suppressed, rule 3 would have cancelled her as a duplicate, because Tomas came first in the record set.
3. Viktor's number does not format. A raw value exists, so rule 4 fires: state **cancelled**, failure type **wrong number format**.
4. Nothing at all is handed to the provider. Each cancelled row records exactly why, so the operator sees three distinct explanations rather than one generic failure.
5. Because the rows never reach the provider, no tracker is created for them and no delivery report can ever arrive; their notifications stay at "cancelled" permanently — the monotonic rule would ignore a later attempt to move a cancelled notification back to "ready" from a report.

---

## 29. Browser push payload truncation

```formula
maximum_payload_length = maximum_transport_payload
                       − encryption_header_size
                       − encryption_block_overhead
```

All three quantities are fixed by the transport and the encryption scheme and do not depend on the payload.

```formula
body_maximum_length = max( 0 , maximum_payload_length
                              − serialized_payload_length
                              + serialized_body_length )
```

Both lengths are measured on the **serialized** form, in which every non-ASCII character is written as a six-character escape sequence. Truncating the serialized body must therefore never cut an escape in half:

1. Cut the serialized body to the computed maximum and strip any trailing backslash.
2. Try to decode the result. If it decodes, use it.
3. If it does not, the decoding failure reports the position of the offending escape; cut instead to two characters before that position, which removes the escape marker as well, and decode again.

Worked example. The body is `BØDY` and three bytes must be removed. Serialized, the body is `BØDY`, nine characters. Cutting to six gives `B\u00d`, which does not decode. The failure position is at the escape, so the cut moves back to before the marker and the body becomes `B`.

---

## 30. Miscellaneous derived values

### Message preview

The plain text of the body, shortened to at most 190 characters on a word boundary, with an ellipsis marker when it was shortened.

### Emptiness of a message

A message is empty when **all** of the following hold: the body is absent or contains no visible content; the subtype has no description; there is no attachment; and either the reader may not see tracking values or there are none. An empty message is hidden by the interface and, when it becomes empty through an edit, its link previews are removed.

### Needs-action counters on a record

```formula
needaction_counter = count of messages on this record, excluding user-specific notifications,
                     that have an unread notification for the acting contact

error_counter      = count of messages on this record, excluding user-specific notifications,
                     authored by the acting contact,
                     that have at least one notification in status "bounced" or "exception"
```

The error counter is deliberately restricted to the acting contact's **own** messages: a delivery error is shown to the person who caused it, not to everyone who can read the record.

### Tracking summary in a push notification

For a message, the summary is:

1. the subtype description, surrounded by line breaks, when the subtype has one;
2. then, for each tracking value whose field still exists and carries **no** access group: the field label, a colon and a space, the old value, then — only when the two differ — an arrow and the new value, then a line break.

For a boolean field the values are rendered as the words for true and false; for anything else the text column is used, falling back to the integer column.

### Alias status reset

Any change to the security policy, the default values or the target model resets the alias status to "not tested". No other field does.

### The channel token

Ten characters drawn uniformly from a 56-character alphabet: the lower-case letters without the letter that resembles the digit one, the upper-case letters without the letter that resembles the digit zero, and the digits from two to nine. The resulting space is 56 to the tenth power, which is above 3 × 10^17 — large enough that guessing an invitation link is not practical, while the link stays short enough to be pasted into a chat.

### The unsubscribe token of a digest

A keyed digest computed over the pair (digest identifier, recipient identifier) with the fixed purpose string for digest unsubscription, using the installation secret. It cannot be transferred to another digest or another recipient.

### The action-link token

For every action link other than "view":

```formula
token = keyed_digest( installation_secret ,
                      base_link + "?" + join( "<key>=<value>" for each key in ascending key order ) )
```

The token is then added to the parameters and the final address is the base link followed by the parameters, again sorted by key. Sorting is what makes the token reproducible.

---

## 31. Telephone number parsing and formatting

The text-message channel depends entirely on this. It is specified here because two other computations — the recipient resolution of section 28 and the staleness of a telephone field — read its output.

### Parsing

1. Parse the raw number with the interpretation country as the default region, keeping the raw input.
2. Reformat the parsed value to the international form and parse it again, so that country-specific corrections are applied. Several countries have changed their numbering plan; the correction adds a leading digit for Brazilian mobile numbers, removes one for Mexican mobile numbers, and applies the published equivalents for Ivory Coast, Colombia, Israel, Morocco, Mauritius, Kenya, Panama and Senegal.
3. A value that cannot be parsed at all aborts with "Unable to parse <number>: <details>".

### Possibility and validity

| Condition | Message |
|---|---|
| the country prefix is not a valid one | "Impossible number <number>: not a valid country prefix." |
| the number is too short | "Impossible number <number>: not enough digits." |
| the number is too long and neither repair works | "Impossible number <number>: too many digits." |
| the number is impossible for another reason | "The phone number <number> is invalid! Let's fix it - you are not dialing aliens." |
| the number is possible but not valid for its region | "Invalid number <number>: probably incorrect prefix." |

Two repairs are attempted for a number that is too long, in this order: a value beginning with a double zero is retried with that prefix replaced by a plus sign; a value with no leading plus sign is retried with one added. Both repairs keep the original value in the final message, so the person recognizes what they typed.

### Formatting

| Requested form | Result |
|---|---|
| strict international | the country prefix followed by the national number with no separator; this is what is stored and what is handed to the sending service |
| international | the country prefix followed by the national number with readable separators |
| national | the national number without the country prefix; used only when the number's country matches the interpretation country |
| address form | the form used inside a structured link |

When the requested form is international, **or** when the number's country code differs from the interpretation country's dialling code, the international form is used even if the national form was asked for. When the caller asks not to raise, an unformattable value is returned unchanged.

### Interpretation country of a record

In order: the record's own country field when the model declares one; otherwise the country of the first contact found through the model's customer fields; otherwise the acting company's country.

### Worked example

A contact in Belgium, whose dialling code is 32, stores the number `0470 12 34 56`. Formatting in the strict international form gives `+32470123456`. Formatting in the national form with Belgium as the interpretation country gives `0470 12 34 56`. The same raw value with France as the interpretation country parses as a French number and is rejected as invalid, because `0470123456` is not a valid French mobile number; the operation then either aborts or, when the caller asked not to raise, returns the raw input unchanged.

---

## 32. Event bus announcement splitting

At the end of a transaction the list of touched broadcast channels is serialized and announced. When the serialized form exceeds the transport limit it is split.

```formula
payload = serialize( channels )

emit payload                                        when count(channels) = 1
                                                     or length_in_bytes(payload) < maximum_announcement_length

otherwise split channels at ceiling( count ÷ 2 ) and recurse on each half
```

The recursion terminates because a single channel is always emitted, whatever its length.

Worked example. A message posted in a channel of 900 members touches 901 broadcast channels: the channel itself plus each member's personal channel. The serialized list is 12 000 bytes and the maximum is 8 000. The list is split into 451 and 450 entries, each serializing to roughly 6 000 bytes, and two announcements are emitted.

---

## 33. Rating aggregation as live chat uses it

### The grade of one value

| Condition on the value | Grade | Label |
|---|---|---|
| at least 4 | `top` | Happy |
| at least 3 and below 4 | `ok` | Neutral |
| at least 1 and below 3 | `ko` | Unhappy |
| below 1 | `none` | Not Rated yet |

A three-value grouping is used for the satisfaction percentage: great from 4, okay from 3 to below 4, bad below 3.

### The grade of an average

Compared at two decimals:

| Condition on the average | Grade |
|---|---|
| at least 3.66 | `top` |
| at least 2.33 and below 3.66 | `ok` |
| at least 1 and below 2.33 | `ko` |
| below 1 | `none` |

### Count, average and percentage

```formula
considered = the ratings of the record whose value ≥ 1
count      = number of considered ratings
average    = ( Σ of their values ) ÷ count
percentage = great_count × 100 ÷ count           when count > 0
percentage = −1                                   when count = 0
```

A rating stored with the value zero is an unanswered request and is excluded from all three. The value −1 is deliberately distinguishable from zero: it means "never rated", while zero means "rated, and nobody was satisfied".

### The parent aggregate

The same three formulas are applied over the consumed ratings whose **parent** record is the aggregating record. One extra measure exists:

```formula
average_on_a_hundred_point_scale = average × 20
```

### Worked example

A live chat entry point received the session ratings 5, 5, 4, 3, 1, 1 and one unanswered request stored as 0.

- Considered ratings: 5, 5, 4, 3, 1, 1. Count 6.
- Average = (5 + 5 + 4 + 3 + 1 + 1) ÷ 6 = 19 ÷ 6 = 3.166666…, displayed as 3.17. Its grade is `ok`, because 3.17 is below 3.66 and at or above 2.33.
- Grades: great, great, great, okay, bad, bad. Great count 3.
- Percentage = 3 × 100 ÷ 6 = 50.0.
- With an average of 4.20 over 55 consumed ratings, the hundred-point form is 4.20 × 20 = 84.0.

---

## 34. Further live chat reporting measures

Section 26 gives the capacity and the duration arithmetic. The reporting view adds the following.

### Duration in minutes

```formula
duration_minutes = ( ( session_end_moment or now ) − session_start_moment ) in seconds ÷ 60
```

Two decimals, averaged across sessions.

### Response time in hours

```formula
response_time_hours = undefined
    when the session is closed and its only operator-side message is later than the end moment

response_time_hours = ( first_operator_message_moment − last_bot_message_moment ) ÷ 3600
    when the session had both a bot and a human operator

response_time_hours = ( first_operator_message_moment − session_start_moment ) ÷ 3600
    when the session had a human operator and no bot

response_time_hours = ( first_operator_side_message_moment − session_start_moment ) ÷ 3600
    otherwise
```

Six decimals, averaged across sessions. The undefined branch removes a session whose only operator message arrived after the session was closed, which would otherwise report a negative or meaningless time.

Worked example. A session starts at 10:00:00, a bot posts its last line at 10:00:12, a human answers at 10:01:42 and the session closes at 10:09:00. The session had both a bot and a human, so the response time is (10:01:42 − 10:00:12) ÷ 3600 = 90 ÷ 3600 = 0.025000 hours. The duration is (10:09:00 − 10:00:00) ÷ 60 = 540 ÷ 60 = 9.00 minutes.

### Call measures

```formula
call_duration_hours  = Σ over the session's finished calls of ( end_moment − start_moment ) in seconds ÷ 3600
has_call             = 1 when call_duration_hours is defined, otherwise 0
sessions_with_calls  = Σ of has_call
share_of_calls       = average of has_call
```

Worked example. Over 40 sessions, 6 had at least one call: the sum is 6 and the average is 6 ÷ 40 = 0.15, displayed as 15 percent.

### Satisfaction of a session

A last rating value of 1 reports Unhappy, 3 reports Neutral and 5 reports Happy; anything else, the value zero included, reports no rating at all.

### Handling flags

```formula
handled_by_agent = 1 when at least one participant history is of the operator type, otherwise 0
handled_by_bot   = 1 when at least one participant history is of the bot type
                     and none is of the operator type, otherwise 0
```

A session escalated from a bot to a human therefore counts as handled by an operator only.

### The chatbot answer path

The identifiers of the answers the visitor chose, in the order they were recorded, joined by a space, a hyphen and a space; and the same list rendered with the answer labels in the reader's language, falling back to the reference language, then to any stored translation, then to the literal label "Unknown" for an answer that no longer exists. Grouping on the path therefore still groups the sessions whose answers were deleted.

---

## 35. Batch sizes and their effect

| Quantity | Parameter | Default | Effect |
|---|---|---|---|
| Recipients per generated notification electronic mail | `mail.batch_size` | 50 | The recipients of one rendering group are split into chunks of this size and each chunk becomes one Outgoing Mail. A stored value of zero is replaced by 50 so the loop always progresses. The same size splits the records of one template rendering pass. |
| Outgoing mails per queue run | `mail.mail.queue.batch.size` | 1000 | The upper bound of one scheduled run. With an explicit list of identifiers the search limit is ten times this value and the result is intersected with that list. |
| Immediate-sending limit | `mail.mail.force.send.limit` | 100 | Above this number of produced mails, immediate sending is refused and the queue takes over. |
| Rows per transport session | `mail.session.batch.size` | 1000 | One connection carries at most this many mails before a new one is opened. |
| Members per mailing-list relay chunk | `mail.session.batch.size` | 500 in the relay path | The member list is processed in chunks of this size. |
| Text messages per queue run and per provider call | `sms.session.batch.size` | 500 | The upper bound of one scheduled run, also used to split one run into provider calls. |
| Text messages per call to the external telephony provider | `sms_twilio.session.batch.size` | 10 | The provider accepts smaller calls. |
| Records per composer rendering pass | fixed | 50 | The composer renders and creates in passes of this size. |
| Records per activity scheduling pass | fixed | 500 | The scheduling window creates in passes of this size. |
| Scheduled messages posted per run | fixed | 50 | More are left for the next wake-up, which is triggered immediately when any remain. |
| Activities purged per run | fixed | 10000 | More are left for the next run. |

Worked example. A message is posted on a record followed by 320 people, of whom 260 prefer electronic mail and are all classified as customers. With the generation batch size at 50, the customer group produces ceiling(260 ÷ 50) = 6 Outgoing Mails: five carrying 50 recipients and one carrying 10, plus 260 Notifications. Because 6 is below the immediate-sending limit of 100, the six mails are handed to the relay right after the transaction commits. Had those 260 recipients been spread over four rendering groups in different languages, each pair of language and group would produce its own chunks and the total would be the sum of the per-pair chunk counts.

---

## 36. The message preview line

```formula
plain_text = the body with every markup element removed and every run of whitespace collapsed to one space, trimmed
preview    = plain_text                                       when its length ≤ 190 characters
preview    = plain_text shortened to 190 characters on a word boundary, with the marker " [...]" appended
                                                              otherwise
```

The 190 characters include the marker: the shortening keeps whole words and reserves room for the marker, so the result never exceeds 190 characters.

Worked example. A body whose plain text is 300 characters long produces the first whole words that fit in 190 characters minus the six characters of the marker, followed by that marker.

**Compatibility finding.** The explanatory comment beside the implementation states a limit of 100 characters and calls it the longest preview line a widely used mail client shows; the constant actually applied is 190. A rebuild that must produce byte-identical previews uses 190. A corrected behaviour would make the comment and the constant agree, and, if the shorter value were chosen, would shorten to 100.

---

## 37. The consent code of the electronic-mail client plugin

```formula
grant_name   = add_in_name                                    when no extra information was supplied
grant_name   = add_in_name + ": " + extra_information         otherwise

payload      = the structure { scope , grant_name , issue_moment_in_seconds_since_1970 , acting_user_identifier }
               serialized with its keys in ascending order

signature    = keyed_digest( installation_secret , purpose "mail_plugin" , payload )

consent_code = base_64( payload ) + "." + base_64( signature )
```

Validation recomputes the signature over the decoded payload and compares it **in constant time**; a mismatch and an issue moment more than three minutes old both answer the invalid-code error. The application key issued in exchange lives for one day.

Worked example. A code issued at 09:00:00 and exchanged at 09:02:59 is accepted; the same code exchanged at 09:03:01 is refused, and no key is created.

---

## 38. Reconciliation notes

1. **The preview length.** One source version computed the preview as the first 100 characters plus a six-character marker; the other as at most 190 characters including the marker. 190 is the observable behaviour and is what section 36 states, with the discrepancy recorded there as a compatibility finding. Section 30 refers to the same rule.
2. **The unread counter.** One source version subtracted the member's own messages from the counter. The observable counter excludes the two notification message types and counts everything else at or above the separator; the author's own messages never appear because the posting hook advances the author's separator. Section 23 states the observable rule and the worked example counts accordingly.
3. **The reply address length rule.** One source version described a middle step in which the display name is "simplified". The observable rule tries three candidates in order — the author's name plus the address, the acting user's name plus the address, then the bare address — and section 4 states those three.
4. **The message identifier grammar.** One source version described the identifier as "a random token, a context string, a host name"; the other gave the full grammar including the fixed marker. Section 5 keeps the full grammar, because the incoming router matches on the marker.
5. **The personal relay throttle.** One source version presented the throttle as a simple allowance of whole mails; the observable behaviour also **splits** a mail whose recipient count would cross the limit and strips the free addresses from the remainder. Section 20 keeps the split.
6. **The loop-detection worked example.** One source version counted the twentieth message as still creating a record and the twenty-first as suppressed; the threshold is compared with "at or above", so the message that finds twenty existing records is the one suppressed. Section 17 states that comparison.
