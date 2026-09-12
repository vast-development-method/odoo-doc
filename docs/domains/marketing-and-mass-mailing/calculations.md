# Marketing calculations

Every formula and algorithm of the domain, with its inputs, its outputs, its precision, its order of operations and at least one worked example with real numbers.

## 1. The five mailing indicators

**Inputs.** The delivery records of one mailing, grouped by status, plus two counts over non-empty moments.

```
scheduled = number of delivery records with status "outgoing"
process   = number with status "process"
pending   = number with status "pending"
canceled  = number with status "cancel"
bounced   = number with status "bounce"
failed    = number with status "error"
replied   = number with status "reply"
opened    = number with status "open"   + number with status "reply"
delivered = number with status "sent"   + number with status "open" + number with status "reply"
clicked   = number of delivery records whose last-click moment is not empty
sent      = number of delivery records whose sent moment is not empty
expected  = total number of delivery records of the mailing
```

**Denominators.** Each is replaced by 1 when it evaluates to 0, in order that an empty mailing reports zero rather than failing.

```
total         = expected − canceled                             (or 1 when that is 0)
total_no_error = expected − canceled − bounced − failed          (or 1 when that is 0)
total_sent    = expected − canceled − failed                     (or 1 when that is 0)
```

**Indicators**, each rounded to two decimal places:

```
received_ratio = round(100 × delivered ÷ total,          2)
opened_ratio   = round(100 × opened    ÷ total_no_error, 2)
replied_ratio  = round(100 × replied   ÷ total_no_error, 2)
bounced_ratio  = round(100 × bounced   ÷ total_sent,     2)
```

Note that each indicator uses a different denominator on purpose: reception is measured against everybody who was not cancelled, engagement against everybody who could actually have engaged, and bouncing against everybody who was actually handed over.

### 1.1 Worked example

A mailing has 1000 delivery records. Of these, 38 ended in a technical error, 12 bounced, and 950 were delivered; among the delivered ones 400 were opened, and 30 of those 400 also replied. No delivery record was cancelled. Every record that was handed over carries a sent moment, therefore `sent = 950 + 12 = 962`.

Status counts:

| Status | Count |
|---|---|
| `sent` (Delivered) | 950 − 400 = 550 |
| `open` (Opened) | 400 − 30 = 370 |
| `reply` (Replied) | 30 |
| `bounce` (Bounced) | 12 |
| `error` (Exception) | 38 |
| `cancel` (Cancelled) | 0 |
| **Total** | **1000** |

Derived counters:

```
delivered = 550 + 370 + 30 = 950
opened    = 370 + 30       = 400
replied   = 30
bounced   = 12
failed    = 38
canceled  = 0
expected  = 1000
sent      = 962
```

Denominators:

```
total          = 1000 − 0            = 1000
total_no_error = 1000 − 0 − 12 − 38  = 950
total_sent     = 1000 − 0 − 38       = 962
```

Indicators:

```
received_ratio = round(100 × 950 ÷ 1000, 2) = round(95.0,      2) = 95.00
opened_ratio   = round(100 × 400 ÷ 950,  2) = round(42.105263, 2) = 42.11
replied_ratio  = round(100 × 30  ÷ 950,  2) = round(3.157894,  2) = 3.16
bounced_ratio  = round(100 × 12  ÷ 962,  2) = round(1.247401,  2) = 1.25
```

The statistics message sent one day later therefore reads: *"Engagement on 1000 Emails Sent"*, `95.0%` under *"RECEIVED (950)"*, `42.11%` under *"OPENED (400)"* and `3.16%` under *"REPLIED (30)"*.

### 1.2 Second worked example: a small comparison-test version

A version addressed to 15 recipients, of which 10 opened, none bounced, none failed, none was cancelled:

```
expected = 15, canceled = 0, bounced = 0, failed = 0
total_no_error = 15
opened_ratio   = round(100 × 10 ÷ 15, 2) = round(66.666667, 2) = 66.67
```

A sibling version addressed to 30 recipients of which 15 opened:

```
opened_ratio = round(100 × 15 ÷ 30, 2) = 50.00
```

## 2. Click ratio

**Inputs.** One grouped read over the delivery records of the mailing joined to the recorded visits.

```
counted_records = delivery records of the mailing whose status is NOT in ("bounce", "cancel", "error")
clicking_records = distinct delivery records among those that have at least one recorded visit
clicks_ratio = round(100 × clicking_records ÷ counted_records, 2)
```

When the mailing has no counted record the ratio is 0.

**Worked example.** 962 counted records, 173 of which have at least one visit:

```
clicks_ratio = round(100 × 173 ÷ 962, 2) = round(17.983368, 2) = 17.98
```

A recipient who clicked five links five times each still counts once here, while the click counter of each Link Tracker counts every single visit.

## 3. Campaign indicators

**Inputs.** One grouped read over every delivery record attached to the campaign.

```
expected  = number of delivery records of the campaign
sent      = number whose sent moment is not empty
delivered_status = number with status in ("sent", "open", "reply")
open      = number with status in ("open", "reply")
reply     = number with status "reply"
bounce    = number with status "bounce"
cancel    = number with status "cancel"

total     = expected − cancel                (or 1 when that is 0)
delivered = sent − bounce

received_ratio = round(100 × delivered ÷ total, 2)
opened_ratio   = round(100 × open      ÷ total, 2)
replied_ratio  = round(100 × reply     ÷ total, 2)
bounced_ratio  = round(100 × bounce    ÷ total, 2)
```

A campaign with no delivery record reports zero for all four.

The campaign formula deliberately differs from the mailing formula: it uses one single denominator, and it derives *delivered* by subtracting bounces from the handed-over count rather than by summing statuses. A replacement must reproduce both.

**Worked example.** A campaign holding the two versions and the winner of section 4: 1000 delivery records, none cancelled, 962 handed over, 12 bounced, 550 with status `sent`, 370 `open`, 30 `reply`.

```
total          = 1000
delivered      = 962 − 12 = 950
received_ratio = round(100 × 950 ÷ 1000, 2) = 95.00
opened_ratio   = round(100 × 400 ÷ 1000, 2) = 40.00
replied_ratio  = round(100 × 30  ÷ 1000, 2) = 3.00
bounced_ratio  = round(100 × 12  ÷ 1000, 2) = 1.20
```

Note that the campaign open rate (40.00) is lower than the mailing open rate (42.11) computed over the same records, because the denominators differ.

## 4. Comparison-test sizing and winner selection

### 4.1 Audience size shown before sending

```formula
audience_size = number of records matching the recipient condition
total         = max(truncate(audience_size ÷ 100 × percentage), 1)
                when audience_size > 0 and comparison testing is enabled and percentage < 100
total         = audience_size in every other case
```

The multiplication is done as `audience_size ÷ 100.0 × percentage` and the result is truncated towards zero before the floor of 1 is applied.

### 4.2 Sample drawn when sending

```formula
audience       = keys matching the condition
audience_size  = size of the audience
sample_size    = max(truncate(audience_size ÷ 100 × percentage), 1)   when audience_size > 0, else 0
already_mailed = keys already traced for the campaign
remaining      = audience − already_mailed
sample_size    = size(remaining)
                 whenever sample_size > size(remaining),
                 and whenever size(remaining) > 0 while sample_size = 0
sample         = sample_size keys drawn at random, without replacement,
                 from the sorted remaining set
```

### 4.3 Worked example: 20 percent, winner by open rate after 24 hours

A list holds **1000** contacts. Two versions are created in one campaign, each at **20 percent**, with the criterion *Highest Open Rate* and a comparison-test moment 24 hours after creation.

**Version A is sent first.**

```
audience_size  = 1000
sample_size    = max(truncate(1000 ÷ 100 × 20), 1) = max(200, 1) = 200
already_mailed = {}                      (no trace yet for the campaign)
remaining      = 1000 keys
200 ≤ 1000, therefore sample_size stays 200
```

200 delivery records are created for version A.

**Version B is sent next.**

```
audience_size  = 1000
sample_size    = 200
already_mailed = the 200 keys traced by version A
remaining      = 800 keys
200 ≤ 800, therefore sample_size stays 200
```

200 delivery records are created for version B, drawn from the 800 untouched contacts. **400 distinct people** have now been contacted; nobody twice.

**After 24 hours**, the automatic job runs. Version A shows 96 opened out of 200 with no bounce and no error; version B shows 70 out of 200:

```
version A opened_ratio = round(100 × 96 ÷ 200, 2) = 48.00
version B opened_ratio = round(100 × 70 ÷ 200, 2) = 35.00
```

Version A wins. A copy of version A is created with `ab_testing_pc = 100` and the name `" A/B Testing V1 (final)"`, the campaign records it as winner and becomes completed, and the copy is queued immediately.

**The winner send.**

```
audience       = 1000 keys
this mailing is the campaign winner, therefore
remaining      = 1000 − (keys traced by version A or version B) = 1000 − 400 = 600
```

600 delivery records are created. Overall, each of the 1000 contacts received exactly one message: 200 got version A, 200 got version B, 600 got the winning version.

### 4.4 Worked example: percentages that do not divide evenly

A list of **150** contacts, version 1 at 10 percent and version 2 at 20 percent:

```
version 1: sample_size = max(truncate(150 ÷ 100 × 10), 1) = max(15, 1) = 15 ; remaining = 150 ; sample = 15
version 2: sample_size = max(truncate(150 ÷ 100 × 20), 1) = max(30, 1) = 30 ; remaining = 135 ; sample = 30
distinct contacts reached = 45
```

If version 1 then shows 10 opens and version 2 shows 15 opens:

```
version 1 opened_ratio = round(100 × 10 ÷ 15, 2) = 66.67
version 2 opened_ratio = round(100 × 15 ÷ 30, 2) = 50.00
winner = version 1
```

### 4.5 Worked example: a percentage too small for the audience

A list of **10** contacts, one version at **2 percent**:

```
sample_size = max(truncate(10 ÷ 100 × 2), 1) = max(truncate(0.2), 1) = max(0, 1) = 1
```

One message is sent. The floor of 1 guarantees that a test always reaches at least one person when the audience is not empty.

### 4.6 Winner selection

```
versions = the mailings of the campaign that have comparison testing enabled
           and whose mailing type matches the mailing that triggers the selection
candidates = versions whose state is "done"
if criterion = "manual": the winner is the mailing the user pressed the button on
else if candidates is empty: refuse
else: sort candidates on the criterion, descending; the winner is the first
```

Available criteria for electronic mail: `opened_ratio`, `clicks_ratio`, `replied_ratio`, and, with the matching capability packages, `crm_lead_count`, `sale_quotation_count`, `sale_invoiced_amount`. Available criteria for text messages: `clicks_ratio`, and the same three business criteria. The sort is done with elevated rights, because those business criteria read records of other domains.

## 5. Mailing list quality percentages

**Inputs.** One grouped read over the subscriptions of the list joined to the contacts and to the blocked-address register, plus one grouped read counting contacts with bounces.

```
contact_count             = number of subscriptions of the list
contact_count_email       = number of subscriptions whose contact has a normalised address,
                            that are not opted out, and whose contact is not blocked
contact_count_opt_out     = number of subscriptions that are opted out
contact_count_blacklisted = number of subscriptions whose contact is blocked
bouncing_contacts         = number of distinct contacts of the list whose bounce counter is greater than 0

contact_pct_opt_out     = 100 × contact_count_opt_out     ÷ contact_count
contact_pct_blacklisted = 100 × contact_count_blacklisted ÷ contact_count
contact_pct_bounce      = 100 × bouncing_contacts         ÷ contact_count
```

When `contact_count` is zero the three percentages are all exactly 0 and no division is performed.

With the Text Message Marketing package, `contact_count_blacklisted` additionally counts subscriptions whose contact's sanitised number is blocked, and a further counter `contact_count_sms` counts subscriptions whose contact has a sanitised number, that are not opted out, and whose contact's number is not blocked.

**Worked example.** A list with 1204 subscriptions: 37 opted out, 11 blocked addresses, 1150 with a usable address that is neither opted out nor blocked, and 64 contacts with at least one bounce.

```
contact_pct_opt_out     = 100 × 37 ÷ 1204 = 3.0731 %
contact_pct_blacklisted = 100 × 11 ÷ 1204 = 0.9136 %
contact_pct_bounce      = 100 × 64 ÷ 1204 = 5.3156 %
contact_count_email            = 1150
```

These values are not rounded at computation time; the screen shows them with its own precision.

## 6. The batching algorithm

**Inputs.** The remaining recipient keys, the batch size, the prepared composer.

```
batch_size = system parameter "mail.batch_size" read as an integer
if batch_size is absent or evaluates to 0: batch_size = 50
counter_done = 0
for each consecutive slice of at most batch_size keys:
    prepared = prepare one outgoing message per key                  (rendering, links, layout)
    prepared = apply the exclusion checks in order                   (section 7)
    prepared = drop the cancelled entries and create their delivery records
    create the outgoing messages together with their delivery records
    create the per-recipient notification records                    (empty for marketing sends)
    when the messages must be sent at once ->
        send the slice, skipping entries whose scheduled moment is in the future,
        then continue with the next slice
    otherwise ->
        counter_done = counter_done + number of kept entries
        report progress as (processed = counter_done, remaining = total − counter_done)
        drop every cached record
```

**Worked example.** 1000 remaining recipients, batch size 50, of which 38 are cancelled (blocked, opted out or duplicate) spread evenly. The loop runs 20 times. Each pass prepares 50 messages, cancels on average 1.9 of them, creates about 48 outgoing messages and 50 delivery records, commits, and reports progress. After the twentieth pass the mailing is written to `done`.

## 7. Duplicate detection inside one run

For each prepared message that passed the blocked, missing-address, invalid-address and opted-out checks:

1. When every normalised destination of this message is already in the already-contacted set, the message is cancelled with the failure type `mail_dup`.
2. Otherwise, when the number of attachments matches the composer's attachment count, **and** every normalised destination is already a key of the run's sent map, **and** at least one message already sent in this run to one of those destinations has the same subject and the same body, the message is cancelled with the failure type `mail_dup`.
3. Otherwise the message is kept, and its subject, body and attachment count are appended to the sent map under each of its normalised destinations.
4. After the batch, the keys of the sent map are added to the already-contacted set.

**Worked example.** A list contains two contacts, `Gilberte` and `Gilberte En Mieux`, both with the address `gilberte@example.com`, and the mailing body contains no placeholder that differs between them. The first is prepared and kept; the second matches the third condition (same destination, same subject, same body, same attachments) and is cancelled with `mail_dup`. If the body had contained the contact name as a placeholder, the bodies would have differed and both would have been sent.

## 8. Already-contacted set

The set is read in one grouped query over the delivery records, joined to the table of the recipient entity on the recipient key, keeping only records whose stored address is not empty.

| Comparison testing | Restriction applied to the delivery records |
|---|---|
| enabled | the campaign of the delivery record is this mailing's campaign |
| disabled | the mailing of the delivery record is this mailing **and** its recipient entity name is this mailing's recipient entity |

The result is a set of normalised addresses. The join to the recipient table means that delivery records pointing at records that have since been deleted are ignored, so a deleted contact does not permanently block its address.

## 9. Short-code generation

**Inputs.** The number of codes wanted.

1. Start with a length of 3.
2. Draw as many random strings of that length as codes are wanted, each character drawn with repetition allowed from the 62 characters `a` to `z`, `A` to `Z` and `0` to `9`.
3. When the drawn strings contain a repeat, or when any of them already exists as a Link Tracker Code, raise the length by 1 and go back to step 2.
4. Otherwise the drawn strings are the generated codes.

The alphabet has 62 characters, therefore length 3 offers 238 328 combinations, length 4 offers 14 776 336, and so on. The length grows only when a collision actually happens, which keeps short codes short in a small installation and lengthens them automatically in a large one.

## 10. Unique-name counter

Specified with its worked example in [entities.md](entities.md) section 18.2. Summary:

```
base, wanted_counter = split "<base> [<n>]" into (base, n); a plain name gives (name, 1)
used = counters already taken by the existing names equal to base or starting with "base ["
if wanted_counter is given and free: counter = wanted_counter
else: counter = the smallest integer ≥ 1 not in "used"
final = base when counter = 1, otherwise "base [counter]"
```

## 11. Text-message length estimate

The designer must show the author how many characters the links will occupy, before those links exist. The estimate builds two placeholder strings.

**Inputs.** The largest existing outgoing text-message key, the most recently created short code, the mailing key (or the largest existing mailing key plus one when the mailing is not yet saved), the base address, and the fixed code size of 3 used by the opt-out code.

```
message_key_length = max(number of digits of the largest outgoing text-message key, 5)
                     or 5 when no outgoing text message exists
code_length        = number of characters of the most recently created short code, plus 1
                     or 3 (the minimum code length) when no code exists
mailing_key_length = number of digits of this mailing's key,
                     or of the largest existing mailing key plus one, or 1 when none exists

link_placeholder        = <base>/r/<code_length copies of "x">/s/<message_key_length copies of "x">
opt_out_address         = <base>/sms/<mailing_key_length copies of "x">/<3 copies of "x">
unsubscribe_placeholder = a line break, then "STOP SMS: " followed by opt_out_address
```

**Worked example.** Base address `https://example.com`, largest outgoing text-message key 4213 (4 digits, raised to the minimum 5), most recent short code `aK7` (3 characters, plus 1 gives 4), mailing key 87 (2 digits):

```
link_placeholder        = "https://example.com/r/xxxx/s/xxxxx"          → 34 characters
opt_out_address         = "https://example.com/sms/xx/xxx"             → 30 characters
unsubscribe_placeholder = "\nSTOP SMS: https://example.com/sms/xx/xxx" → 41 characters
```

A body of 120 characters containing one link and using the opt-out option is therefore estimated at `120 − (length of the original link) + 34 + 41` characters. The operation requires write access on the mailing.

## 12. Inline image conversion

**Inputs.** A rich-text body.

1. Parse the body.
2. Walk every element and every comment:
   - An image element whose source is an inline image is collected, together with its original-file key when it carries one.
   - An element whose style contains an inline image collects every inline image found in that style, together with the exact matched text, in order that the replacement is textual.
   - A comment recognised as an outgoing-mail compatibility block collects every inline image found inside its text, and additionally, for every vector image element inside it that declares a width and a height in pixels, fetches the referenced image, crops and resizes it to those dimensions and collects the result.
3. Turn the collected images into stored files:
   - Load the existing files already attached to this mailing, keyed by checksum.
   - For each image compute its checksum. When a file with that checksum already exists, reuse it. Otherwise prepare exactly one new file per distinct checksum, named `image_mailing_<mailing key>_<running number>`, of binary kind, attached to the mailing, carrying the original-file key when one was collected.
   - Create the new files, then map every collected image to its file.
   - Generate an access token per file and build the address `/web/image/<file key>?access_token=<token>`.
4. Replace: an image element gets the new address as source; a style keeps its text with the matched inline image replaced by the new address; a comment keeps its text with the same replacement.
5. When at least one replacement happened, return the serialised body; otherwise return the original body unchanged.

**Fetch limits.** An image fetched by address is refused when the declared length exceeds the configured import maximum (*"File size exceeds configured maximum (%s bytes)"*), when the accumulated bytes exceed it during streaming (same message), or when its pixel count exceeds 42 million (*"Image size excessive, imported images must be smaller than 42 million pixel"*). Any other failure raises *"Could not retrieve URL: %s"*. Fetch failures during the compatibility-block cropping are swallowed, in order that one broken image does not break the whole save.

**Worked example.** A body containing nineteen inline images, of which the first and the last are byte-identical, produces eighteen stored files and nineteen addresses, two of which point at the same file.

## 13. Token computation

| Token | Purpose string | Input tuple | Function |
|---|---|---|---|
| Recipient token | — | (database name, mailing key, record key, address) | Keyed hash of the printed tuple with the database secret, using the strong 512-bit hash function, rendered as hexadecimal. |
| Open-tracking token | `mass_mailing-mail_mail-open` | the outgoing mail key | The platform's keyed-hash helper, run with elevated rights. |
| Statistics-message token | `mailing-report-deactivated` | the user key | The platform's keyed-hash helper, run with elevated rights. |

All three are compared in constant time.

**Worked example.** For the database `production`, mailing key 42, recipient record key 1337 and address `jane@example.com`, the recipient token is the hexadecimal keyed hash of the printed tuple `('production', 42, 1337, 'jane@example.com')`. The unsubscribe address of that recipient is then

```
<recipient base>/mailing/42/confirm_unsubscribe?document_id=1337&email=jane%40example.com&hash_token=<token>
```

and the one-click address is the same with `unsubscribe_oneclick` in place of `confirm_unsubscribe`.

## 14. Automatic blocking after repeated bounces

**Inputs.** The bounced address, the resolved contacts, the delivery records.

```
window_start = current moment − 13 weeks
qualifying = delivery records with status "bounce", last written after window_start,
             whose address matches the bounced address case-insensitively
if size(qualifying) ≥ 5
   and (no contact was resolved or every resolved contact has a bounce counter ≥ 5)
   and max(last write of qualifying) > min(last write of qualifying) + 1 week:
        add the address to the blocked-address register with the note
        "This email has been automatically added in blocklist because of too much bounced."
```

**Worked example.** An address bounced on 1 March, 3 March, 5 March, 7 March and 9 March. The count is 5 and the spread is 8 days, which is more than one week, therefore the address is blocked. The same five bounces on 1, 2, 3, 4 and 5 March give a spread of 4 days and the address is **not** blocked, because a four-day burst looks like a temporary server failure.

## 15. Marketing card geometry

```
width  = 600 pixels
height = 315 pixels
ratio  = 40 ÷ 21 ≈ 1.9048
```

These dimensions keep the file below the common five-megabyte limit of social networks while matching the near-two-to-one ratio recommended for large preview images.

## 16. Ordering rules

| Ordering | Rule |
|---|---|
| Mailing board and list | `calendar_date` descending, therefore finished mailings by sent date, queued mailings by next departure, then drafts. |
| Favorite-design gallery | `favorite_date` descending, then re-ordered so that designs with no responsible come first, then those of the current user, then the rest. Designs with a visually empty body are removed. Archived designs are kept. |
| Link Tracker list | click count descending. |
| Recent links | `newest` = creation moment then key, both descending; `most-clicked` = click count then key, both descending, only trackers with at least one click; `recently-used` = last write then key, both descending, same restriction. |
| Mailing Contact list | name ascending, then key descending. |
| Mailing List | creation moment descending. |
| Mailing Filter | creation moment descending. |
| Delivery records | creation moment descending. |
| Mailing Opt-Out Reason | sequence ascending, then creation moment descending, then key descending. |
| Campaign Stage | sequence ascending. |
| Campaign Medium and Campaign Tag | name ascending. |
| Marketing Card Campaign | key descending. |
| Campaign board columns | every stage is shown, even when it holds no campaign. |

## 17. Counting audiences for the screens

| Counter | Rule |
|---|---|
| Mailing `total` | See section 4.1. |
| Mailing `link_trackers_count` | Number of Link Tracker records whose mailing is this one. |
| Mailing `mailing_filter_count` | Number of Mailing Filter records whose recipient entity is this mailing's recipient entity. |
| List `mailing_count` | Number of rows of the list-to-mailing association table for this list. |
| Campaign `click_count` | Number of recorded visits attributed to the campaign. |
| Campaign `mailing_mail_count` and `mailing_sms_count` | Number of mailings of the campaign of that type, summed over the comparison-test grouping. |
| Campaign `ab_testing_mailings_count` and `ab_testing_mailings_sms_count` | The same restricted to mailings with comparison testing enabled. |
| Card campaign `card_count`, `card_click_count`, `card_share_count` | All cards; cards whose share status is `visited` or `shared`; cards whose share status is `shared`. |
| Mailing `card_requires_sync_count` | For a draft mailing with a card campaign: the number of recipients of the current condition, minus the number of cards of that campaign for those recipients that do **not** need regeneration. Zero for any other state. |
