# Calculations

Every formula and every algorithm of the domain, with its quantities named in words, its evaluation
order, its rounding rule and at least one worked numeric example carried to the last digit the rule
produces.

Contents:

1. [Automation rule derivations](#1-automation-rule-derivations)
2. [Webhook addresses and payloads](#2-webhook-addresses-and-payloads)
3. [Time-based automation arithmetic](#3-time-based-automation-arithmetic)
4. [Metered account balance and purchase address](#4-metered-account-balance-and-purchase-address)
5. [Data import](#5-data-import)
6. [Package archive arithmetic](#6-package-archive-arithmetic)
7. [Tokens, digests and constant-time comparison](#7-tokens-digests-and-constant-time-comparison)
8. [Sparse storage](#8-sparse-storage)
9. [Recycling thresholds and notification cadence](#9-recycling-thresholds-and-notification-cadence)
10. [Company enrichment](#10-company-enrichment)
11. [The privacy query](#11-the-privacy-query)
12. [Anonymisation](#12-anonymisation)
13. [Geocoding and address completion](#13-geocoding-and-address-completion)
14. [Signed addresses for outside storage](#14-signed-addresses-for-outside-storage)
15. [Delegated authorisation strings](#15-delegated-authorisation-strings)
16. [Translation platform links](#16-translation-platform-links)
17. [Tour addresses](#17-tour-addresses)
18. [Attachment content extraction](#18-attachment-content-extraction)

Throughout, *the base address* means the address at which this installation answers requests, as the
platform computes it.

---

# 1. Automation rule derivations

## 1.1 The field names a filter mentions

Several computations need the set of fields a stored filter refers to, without evaluating the
filter. The filter is a text form of a list of conditions, each condition being a triple of a field
path, an operator and a value.

**Procedure.**

1. Scan the filter text for every occurrence of the following shape, in order: an opening square
   bracket or an opening round bracket; any amount of white space; a quotation mark, single or
   double; a name beginning with a lower-case letter and continuing with letters, digits and
   underscores; optionally a full stop followed by further name characters and full stops, which is
   how a path across a relation is written; the same quotation mark that opened; then any run of
   characters containing exactly two commas and no comma after them, ending at a bracket of either
   kind.
2. For each occurrence take the name captured before the optional full stop. That is the field name
   on the watched record type; the part after the full stop describes a path into another record
   type and is deliberately discarded.
3. Resolve each name to a Field Definition of the watched record type. Names that resolve to nothing
   contribute nothing.
4. The result is the union of the resolved definitions; duplicates collapse.

**Why a scan and not an evaluation.** The computation runs while a form is open, on text the user is
still typing, and that text can be crafted. Scanning cannot execute anything.

**Worked example.** The filter text is

`[("stage_id.name", "=", "Won"), ("user_id", "!=", False)]`

The scan finds two occurrences. The first captures `stage_id` and discards `.name`; the second
captures `user_id`. The result is the two Field Definitions `stage_id` (Stage) and `user_id`
(Salesperson) of the watched record type.

## 1.2 The trigger-specific field

Each trigger implies one field of the watched record type, found by the first match of a filtered
search ordered by the platform default:

| Trigger | Stored value | Field sought |
|---|---|---|
| Stage is set to | `on_stage_set` | a link field named `stage_id` or `x_studio_stage_id` |
| Tag is added | `on_tag_set` | a multiple-link field named `tag_ids` or `x_studio_tag_ids` |
| Priority is set to | `on_priority_set` | a selection field named `priority` or `x_studio_priority` |
| State is set to | `on_state_set` | a selection field named `state` or `x_studio_state` |
| User is set | `on_user_set` | a single or multiple link to User named `user_id`, `user_ids`, `x_studio_user_id` or `x_studio_user_ids` |
| On archived | `on_archive` | a boolean named `active` or `x_active` |
| On unarchived | `on_unarchive` | the same boolean |
| After creation | `on_time_created` | the date-and-time field `create_date` |
| After last update | `on_time_updated` | the date-and-time field `write_date` |
| On create and edit | `on_create_or_write` | every field named in the after-filter, by §1.1 |
| every other trigger | — | none |

**Rounding and ties.** The search takes the first match in the platform's default order, which is
ascending identifier; a record type carrying both `stage_id` and `x_studio_stage_id` therefore
resolves to whichever Field Definition was created first.

## 1.3 Keeping the watched-field set in step with a filter

When the after-filter is edited, the watched-field set is not recomputed from scratch — a user may
have added fields by hand. Instead the difference is applied:

```formula
fields entering  = fields named by the new filter − fields named by the previous filter
fields leaving   = fields named by the previous filter − fields named by the new filter
new watched set  = ( current watched set + fields entering ) − fields leaving
```

Both sides use §1.1. The same arithmetic maintains the live-update field set for the live-update
trigger, using that trigger's own field set instead of the watched set.

**Worked example.** The watched set holds `stage_id` and `priority`, the latter added by hand. The
filter changes from `[("stage_id", "=", 3)]` to `[("user_id", "!=", False)]`. Entering is
`{user_id}`; leaving is `{stage_id}`. The new set is `{stage_id, priority} + {user_id} − {stage_id}`
= `{priority, user_id}`.

---

# 2. Webhook addresses and payloads

## 2.1 The secret address of an incoming webhook

```formula
web address = base address + "/web/hook/" + webhook identifier
```

The webhook identifier is a universally unique identifier in its canonical thirty-six-character
form, generated at creation and regenerated on demand. The address is computed only for the webhook
trigger; for every other trigger it is empty.

**Worked example.** Base address `https://example.test`, identifier
`0f5d1c7e-2f14-4a5f-9f2b-1c7e2f144a5f`. The address is
`https://example.test/web/hook/0f5d1c7e-2f14-4a5f-9f2b-1c7e2f144a5f`.

**Collision probability.** A version-four universally unique identifier carries 122 random bits.

```formula
expected collisions ≈ ( number of rules × ( number of rules − 1 ) ) ÷ ( 2 × 2^122 )
```

For ten thousand rules that is (10 000 × 9 999) ÷ (2 × 5.3169 × 10^36) = 9.4 × 10^−30, which is why
no uniqueness index is needed.

## 2.2 The evaluation context of the record-finding expression

The expression that turns an incoming payload into a record is evaluated with exactly these names
available, and nothing else:

| Name | Meaning |
|---|---|
| `datetime` | the date and time helpers |
| `dateutil` | the calendar arithmetic helpers |
| `time` | the clock helpers |
| `uid` | the identifier of the reader, which for a webhook is the elevated reader |
| `user` | the reader as a record |
| `model` | an empty set of the watched record type |
| `payload` | the incoming payload, present only when one was supplied |

The shipped default expression reads the record type from the payload key `_model` and the
identifier from the payload key `_id`, converts the latter to a whole number, and browses that
record.

**Compatibility finding.** The default expression reads the record type from the payload rather than
using the rule's own record type. A payload naming another record type therefore makes the rule run
against a record of a type it was not configured for; the after-filter of that rule, written against
the configured type, then usually fails and the run does nothing. A corrected default would browse
the rule's own record type and use only `_id` from the payload. The observed behaviour is reproduced
because existing integrations send both keys and rely on the default.

## 2.3 The outgoing webhook payload

The body is a structured document whose keys are sorted in ascending order of their text, and whose
values that cannot be represented directly are written as their text form.

```formula
payload = { "_model": transport name of the record type,
            "_id":    identifier of the record,
            "_action": action name + "(#" + action identifier + ")" }
          + the chosen fields of the record, read without resolving links
```

The three fixed keys are always present; a chosen field whose name collides with one of them
replaces it, because the chosen fields are merged in afterwards. Links are read as bare identifiers,
not as pairs of identifier and label.

**Worked example.** The action is named `Notify shipping` and has identifier 42; it runs on the
Transfer with identifier 7 of the record type `stock.picking`; the chosen fields are `name` and
`partner_id`. The body is, with its keys in sorted order:

| Key | Value |
|---|---|
| `_action` | `Notify shipping(#42)` |
| `_id` | 7 |
| `_model` | `stock.picking` |
| `name` | `WH/OUT/00013` |
| `partner_id` | 3814 |

**Delivery.** The body is posted with the media type declaring a structured document, after the
transaction commits, with a timeout of one second. A read timeout is logged as a warning and treated
as neither success nor failure, because the receiver may well have acted. A rollback cancels the
post and writes a warning line instead.

## 2.4 The sample payload shown on the form

The same shape as §2.3, rendered with indentation of four spaces and sorted keys. When the watched
record type has at least one record, the first record found — archived records included — supplies
the values and its identifier replaces the placeholder 1. When it has none, each chosen field
receives a sample value of its kind.

---

# 3. Time-based automation arithmetic

## 3.1 The scheduler interval

Every rule with a time trigger contributes a delay in minutes:

```formula
delay in minutes = absolute value of the delay × factor of the delay unit
```

| Delay unit | Stored value | Factor in minutes |
|---|---|---|
| Minutes | `minutes` | 1 |
| Hours | `hour` | 60 |
| Days | `day` | 1 440 |
| Months | `month` | 43 200 |
| none | — | 0 |

Delays that evaluate to zero are discarded. Then:

```formula
raw interval = smallest remaining delay in minutes ÷ 10, truncated towards zero
interval in minutes = the larger of 1 and the raw interval, then the smaller of that and 240
```

When there is no remaining delay at all, the interval is 240 minutes. Finally the unit is chosen:

```formula
When interval in minutes is an exact multiple of 60:
    interval number = interval in minutes ÷ 60 , interval unit = hours
Otherwise:
    interval number = interval in minutes , interval unit = minutes
```

**Writing the result.** The scheduled action is switched on when at least one rule has a time
trigger and off when none has. Its interval is replaced **only when the new interval is strictly
shorter than the current one**, measured as a duration with a month counted as thirty days; a rule
whose deletion would allow a longer interval therefore leaves the scheduler running as often as
before until an administrator widens it by hand.

**Worked example one.** Three rules: three days before a deadline, two hours after creation,
forty-five minutes after the last update. Delays are 3 × 1 440 = 4 320, 2 × 60 = 120 and 45 × 1 = 45
minutes. The smallest is 45. Raw interval = 45 ÷ 10 = 4 (truncated). The larger of 1 and 4 is 4; the
smaller of 4 and 240 is 4. 4 is not a multiple of 60, so the scheduler runs every **4 minutes**.

**Worked example two.** One rule: five days after creation. Delay = 7 200 minutes. Raw = 720. Capped
at 240. 240 ÷ 60 = 4, so the scheduler runs every **4 hours**, which is also the resting value.

**Worked example three.** One rule with a delay of two minutes. Delay = 2. Raw = 0 (truncated). The
larger of 1 and 0 is 1, so the scheduler runs every **1 minute**, the floor of the rule.

## 3.2 The window a time-based run examines

Let *until* be the current instant taken from the database clock at the start of the rule's turn,
and *last run* the instant stored on the rule, or the first instant of the epoch when the rule has
never run.

```formula
sign      = +1 when the delay mode is "before" , −1 when it is "after"
offset    = sign × delay × duration of one delay unit
lower end = last run + offset
upper end = until + offset
```

The duration of one delay unit is one minute, one hour, one day or one month, a month being a
calendar month as the calendar arithmetic helpers compute it.

For a date-and-time field the rule fires on records whose value is **at or after** the lower end and
**strictly before** the upper end. For a plain date field the two ends are reduced to dates and the
rule fires on records whose value is **strictly after** the lower end and **at or before** the upper
end. The asymmetry is deliberate: a plain date has no time of day, so the half-open interval is
placed on the other side to avoid firing twice on the same calendar day.

When the field is the standard automation date field and the record type also has a creation date,
records whose field is empty are considered with their creation date instead, under the same
comparison.

**Worked example.** A rule fires three days before a deadline; the delay is 3, the mode is `before`,
the unit is days. The rule last ran on 9 September 2026 at 06:00 and the scheduler starts its turn
on 12 September 2026 at 06:00.

```formula
sign = +1
offset = +1 × 3 × one day = +3 days
lower end = 9 September 2026 06:00 + 3 days = 12 September 2026 06:00
upper end = 12 September 2026 06:00 + 3 days = 15 September 2026 06:00
```

Records whose deadline falls at or after 12 September 2026 06:00 and before 15 September 2026 06:00
fire. A record with the deadline 15 September 2026 06:00 exactly does not fire in this turn; it
fires in the next one.

**Worked example, the other direction.** A rule fires two hours after creation; the delay is 2, the
mode is `after`, the unit is hours. Sign = −1, offset = −2 hours. With the same last run and now,
the window is 9 September 04:00 up to 12 September 04:00, so records created two hours or longer ago
and not yet processed fire.

## 3.3 The window when a working schedule is used

When a working schedule is chosen and the delay unit is days, the ends are not computed by adding
days but by planning working days on that schedule, leave included:

```formula
lower end = the instant reached by planning ( sign × delay ) working days from last run
upper end = the instant reached by planning ( sign × delay ) working days from until
```

Both ends are computed once per schedule and reused for every record that uses it, since a record
may name its own schedule. Records whose field lies at or after the lower end and strictly before
the upper end fire. Records with an empty field are excluded, unless the field is the standard
automation date field, in which case the creation date stands in for it.

**Worked example.** A schedule works Monday to Friday. The delay is 2 days, mode `before`, so the
sign is +1. The last run was Thursday 10 September 2026 at 06:00 and now is Friday 11 September 2026
at 06:00. Planning two working days forward from Thursday reaches Monday 14 September 06:00;
planning two working days forward from Friday reaches Tuesday 15 September 06:00. Records whose
deadline falls in that window fire, so the weekend is not counted against the notice period.

## 3.4 The order of a scheduler turn

1. Rules with a time trigger are collected, archived rules excluded.
2. Each is processed in turn. A rule that has been switched off or deleted since the collection is
   skipped.
3. The current instant is taken again for each rule, so a long run does not shorten later windows.
4. Every matching record is processed; then everything pending is written.
5. On success the rule's last-run stamp is set to the instant taken in step 3 and progress is
   recorded so that the scheduler can be interrupted and resumed.
6. On failure everything since the last commit is rolled back, the failure is remembered and the
   next rule is processed. The last remembered failure is raised at the end, which marks the turn as
   failed without having stopped the other rules.

---

# 4. Metered account balance and purchase address

## 4.1 Formatting a balance

The outside service answers with a balance as a number. It is rounded and then joined with the
service's unit name:

```formula
When the service counts in whole units:
    rounded balance = the balance rounded to a whole number, halves going to the even neighbour
Otherwise:
    rounded balance = the balance rounded to four decimal places, halves going to the even neighbour
formatted balance = text of the rounded balance + " " + unit name of the service
```

When the service has no unit name the text still carries the separating space, so a balance of 12
with no unit name is stored as `12 ` with a trailing space.

**Worked examples.**

| Service counts | Answered balance | Unit name | Stored text |
|---|---|---|---|
| whole units | 12.6 | `Credits` | `13 Credits` |
| whole units | 12.5 | `Credits` | `12 Credits` — a half goes to the even neighbour |
| whole units | 13.5 | `Credits` | `14 Credits` |
| fractions | 12.45678 | `Enrichments` | `12.4568 Enrichments` |
| fractions | 0.00005 | `Enrichments` | `0.0 Enrichments` — the fifth decimal is dropped and the trailing zeros of the text form collapse |

## 4.2 The purchase address

```formula
purchase address = service endpoint + "/iap/1/credit"
                   + "?" + encoded query
```

The query carries four entries: the database's universally unique identifier under `dbuuid`, the
service's technical name under `service_name`, the hashed account token under `account_token`, and
the fixed value 1 under `hashed`. Every entry is percent-encoded.

```formula
hashed account token = hexadecimal digest, twenty bytes, of the account token
                        truncated at the first plus sign
```

The digest is the one hundred and sixty bit secure hash, written as forty lower-case hexadecimal
characters. The truncation exists because a neutralised copy of a database appends `+disabled` to
every token; the hash is therefore of the original token and the purchase page still recognises the
account.

**Worked example.** Endpoint `https://iap.odoo.com`, service technical name `partner_autocomplete`,
token `9f2b1c7e2f144a5f9f2b1c7e2f144a5f+disabled`, database identifier
`c0ffee00-1234-5678-9abc-def012345678`. The token is truncated to
`9f2b1c7e2f144a5f9f2b1c7e2f144a5f`, hashed, and the address becomes
`https://iap.odoo.com/iap/1/credit?dbuuid=c0ffee00-1234-5678-9abc-def012345678&service_name=partner_autocomplete&account_token=<forty hexadecimal characters>&hashed=1`.

## 4.3 The alert push

When the threshold or the recipient list changes, the new configuration is pushed to the service:

| Key | Value |
|---|---|
| `account_token` | the account's token, read with elevated rights |
| `warning_threshold` | the threshold as a number |
| `warning_emails` | one entry per recipient, each carrying the recipient's address under `email` and a language code under `lang_code` |

```formula
language code of a recipient = the recipient's own language when set,
                               otherwise the reading language of the session
```

A failure here is written to the log as a warning and does **not** undo the save.

---

# 5. Data import

## 5.1 The importable-field tree

The tree is built for the target record type to a depth of **three**.

1. The first entry is always the external identifier: name `id`, label "External ID", kind
   identifier, not required.
2. When the remaining depth is zero, the tree stops there.
3. Every field of the record type is examined. The platform's bookkeeping columns are skipped, and
   so is every field declared read-only.
4. A property-holding field contributes one entry per property definition found on the records that
   define them. A definition of the separator kind is skipped, and so is a link definition whose
   target record type is unknown. The entry is named `<field>.<property>` and labelled
   "%(property_string)s (%(parent_name)s)", the placeholders being the property's own label and the
   display name of the record that defines it. Definitions are read only from records that are not
   archived, and only when the reader may read both the defining record type and the defining field.
5. A single or multiple link contributes two children: `id` labelled "External ID" and `.id`
   labelled "Database ID", both on the linked record type.
6. A one-to-many link contributes the whole tree of the linked record type at depth one less, plus,
   for a reader in the technical-features group, a child `.id` labelled "Database ID".

## 5.2 Reading a file

The reader is chosen by the media type the browser reported, and, failing that, by the file
extension:

| Media type | Reader |
|---|---|
| `text/csv` | delimited text |
| `application/vnd.ms-excel` | older office workbook |
| `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | current office workbook |
| `application/vnd.oasis.opendocument.spreadsheet` | open document spreadsheet |

**Delimited text.** The encoding is taken from the options; when absent it is detected from the
bytes. When the detected name ends in a byte-order marker variant and the bytes actually start with
that marker, the last two characters of the name are dropped so that the marker is consumed rather
than kept. The bytes are decoded; a failure raises [`business-rules.md#aut-025`](business-rules.md#aut-025).
The separator is taken from the options; when absent each of the six candidates — comma, semicolon,
tabulation, space, vertical bar and the unit-separator control character — is tried in that order
and the first that yields rows of equal width, all wider than one cell, is chosen; when none does,
the comma is used and the resulting width mismatch is reported later by
[`business-rules.md#aut-029`](business-rules.md#aut-029). The quoting character must be exactly one
character. Rows in which every cell is blank after trimming are dropped. The answer is the number of
rows kept and the rows.

**Workbooks and open documents.** Each cell is turned into text: a boolean becomes `True` or
`False`; a whole-valued number is written without its decimal part; a date or a date and time is
kept as a value and stringified later; an error cell stops the read. The list of sheet names is
written back into the options, and the sheet named in the options — or the first one — is read.

## 5.3 Inferring the kinds a column could hold

For one column, given the values of the preview rows, in this order; the first rule that matches
decides:

1. When any value is not text, skip to step 8.
2. Every value is trimmed. When the set of distinct values is exactly the empty text, the column may
   hold **any** kind.
3. When every distinct value starts with `__export__`, the column may hold an identifier, a multiple
   link, a single link or a one-to-many link.
4. When every non-empty value is made only of digits, the column may hold a whole number, a decimal
   number or a monetary amount; and when the distinct values are a subset of `0`, `1` and the empty
   text, also a boolean.
5. When every value, lower-cased, is one of `true`, `false`, `t`, `f` or empty, the column holds a
   **boolean**.
6. When every value can be read as a decimal number after the currency symbol has been removed by
   §5.9, the column may hold a decimal number or a monetary amount. While testing, the grouping and
   decimal characters are inferred and written back into the options: a value with more than one
   full stop implies a full stop as grouping and a comma as decimal; a value with more than one
   comma implies the opposite; otherwise, of the two characters, the one that appears **later** in
   the value is the decimal character and the other is the grouping character.
7. Otherwise §5.4's date test is applied; when it matches, the column holds a date, or a date and a
   time.
8. Otherwise the column may hold text of any of the text-like kinds: long text, single line text,
   binary, selection, rich text or tags.

## 5.4 Inferring a date pattern

The candidate date patterns are, in order: the pattern already in the options, then the reading
user's own date pattern when it can be turned into a matcher, then the generated list. The generated
list is every combination of

- one of the five separators: space, oblique stroke, hyphen, full stop, nothing;
- one of the eight orders: month-day-year, day-month-year, year-month-day, year-day-month, each with
  a four-digit year and with a two-digit year.

A pattern is accepted when every non-empty preview value matches it; values that are already dates
are ignored during the test. The first accepted pattern is written back into the options.

When no date pattern matches, the same test is run over the date-and-time patterns, which are every
accepted date pattern joined by a space to one of the six time patterns: hours-minutes-seconds,
hours-minutes and hours on the twenty-four hour clock, and the same three on the twelve-hour clock
followed by the meridiem indicator. Matching is case-insensitive, and the numeric parts accept both
the padded and the unpadded forms.

**Worked example.** Preview values `01/02/2026`, `15/02/2026`, `28/02/2026`. The first generated
pattern with the oblique stroke is month-day-year; `15/02/2026` fails it because 15 is not a valid
month. The next is day-month-year, which every value matches, so `%d/%m/%Y` is chosen and stored.

## 5.5 Matching a column heading to a field

For one heading, in this order:

1. **Remembered mapping.** When a Data Import Column Mapping exists for this record type with this
   heading, lower-cased, its field path is returned with the distance **−1**, which makes it
   unbeatable in §5.6.
2. **Exact match** when the heading contains no oblique stroke. The fields of the tree are examined
   in order; the first whose identifier, whose translated label, or whose label in the base language
   equals the heading — all compared case-insensitively with full case folding — is returned with
   the distance **0**.
3. **Fuzzy match.** The tree is first reduced to the fields whose kind is among the kinds §5.3
   allowed for this column. When nothing remains, there is no match. Otherwise the distance of §5.6
   is computed between the heading and each of the field's identifier, translated label and base
   label, and the smallest of the three is that field's distance. The field with the smallest
   distance overall is returned when that distance is **strictly below 0.2**; otherwise there is no
   match.
4. **A path across relations.** When the heading contains oblique strokes it is split on them, each
   part is trimmed, and steps 2 and 3 are applied to each part in turn, descending into the matched
   field's own children. A failure at any level abandons the whole heading. A path match carries no
   distance, because §5.7 leaves paths alone.

## 5.6 The distance between two texts

```formula
distance = 1 − ( 2 × number of matching characters ÷ ( length of the first text + length of the second text ) )
```

The number of matching characters is found by repeatedly taking the longest contiguous run common to
the two texts, then applying the same rule to the parts to the left of that run and to the parts to
its right, and adding the lengths of all the runs found. A distance of 0 means the texts are equal;
a distance of 1 means they share nothing. Both texts are case-folded before the comparison.

**Worked example.** Heading `custumer`, field identifier `customer`. The longest common run is
`cust`, four characters. To its right, `umer` and `omer` share the run `mer`, three characters. To
the left of `mer` there is nothing common. Total matching characters = 4 + 3 = 7.

```formula
distance = 1 − ( 2 × 7 ÷ ( 8 + 8 ) ) = 1 − ( 14 ÷ 16 ) = 1 − 0.875 = 0.125
```

0.125 is below 0.2, so the heading matches the field.

**Second worked example.** Heading `Customer`, field label `Customer Reference`. Matching characters
= 8. Lengths 8 and 18.

```formula
distance = 1 − ( 2 × 8 ÷ ( 8 + 18 ) ) = 1 − ( 16 ÷ 26 ) = 1 − 0.6154 = 0.3846
```

0.3846 is not below 0.2, so this pairing is rejected; another field may still win.

## 5.7 Removing duplicate proposals

Two columns must not be proposed for the same field.

1. Every proposal that is empty, and every proposal whose field path has more than one part, is kept
   unconditionally.
2. Among the remaining proposals, the best distance seen so far per field is tracked, starting from
   1 with no holder. A proposal replaces the holder when its distance is **strictly smaller**; ties
   therefore keep the earlier column.
3. Every proposal that is neither kept unconditionally nor the holder for its field is removed.

Because a remembered mapping carries the distance −1, it always wins, and two remembered mappings
onto the same field keep the earlier column.

**Worked example.** Column 0 `Customer` proposes `partner_id` at 0.08; column 3 `Client` proposes
`partner_id` at 0.05; column 5 `partner_id/name` proposes the path `partner_id` then `name`. Column
5 is kept unconditionally. Column 3 replaces column 0 as the holder. Column 0 is removed. The
proposal list holds columns 3 and 5.

## 5.8 Building the dense matrix

```formula
kept column positions = the positions whose mapping entry is not empty
```

When no position is kept, [`business-rules.md#aut-028`](business-rules.md#aut-028) refuses the run.
When the number of mapping entries differs from the width of the first row of the file,
[`business-rules.md#aut-029`](business-rules.md#aut-029) refuses it. The heading row is then dropped
when the options say the file has one, each remaining row is reduced to the kept positions, and rows
that became entirely empty are dropped. Finally, when the options carry a skip count, that many rows
are dropped from the **front of the reduced list**, which is why a resumed import counts rows after
the heading and after the empty ones.

## 5.9 Parsing a decimal number

For one cell, in this order:

1. Trim. An empty cell is left alone.
2. Infer the two separators for **this** value: list the characters that are neither a digit, nor a
   currency symbol, nor one of the three decorations `(`, `)`, `-` and `+`. Count them. When exactly
   two distinct such characters occur and the **last** one occurs exactly once, the grouping
   character is the more frequent of the two and the decimal character is the other. Otherwise the
   options' characters are used, defaulting to a space for grouping and a full stop for the decimal.
3. When the value contains the letter `e` in either case, replace the grouping character with a full
   stop and try to read the result as a number in scientific notation; on success write it back in
   plain notation and set the grouping character to a space.
4. Remove every grouping character and replace the decimal character with a full stop.
5. Remove the currency symbol: trim; when the value is wrapped in round brackets, strip them and
   remember that the number is negative; split the value on the run of digits, full stops, commas
   and an optional leading sign. When more than two pieces remain, the value is not a number. When
   one piece remains, it is the number itself. When two remain, the piece that is not the number is
   looked up among the currency symbols; when it is one, the other piece is the number, otherwise
   the value is not a number. A remembered negative sign is prefixed.
6. A value that is not a number raises [`business-rules.md#aut-030`](business-rules.md#aut-030).

**Worked example one.** The cell is `1.234,50 €`. The non-numeric characters are the full stop, the
comma and the space — three distinct characters, so step 2 falls back to the options; assume the
options were filled by §5.3 with a full stop for grouping and a comma for the decimal. Step 4 gives
`123450 €` — no: removing the grouping full stop gives `1234,50 €`, then replacing the comma gives
`1234.50 €`. Step 5 splits it into `1234.50` and ` €`; the second is a known currency symbol, so the
number is `1234.50`.

**Worked example two.** The cell is `(1 234.50)`. Step 2 finds the space and the full stop; the last
non-numeric character is the full stop, occurring once, so grouping is the space and the decimal is
the full stop. Step 4 gives `(1234.50)`. Step 5 strips the brackets, remembers the negative, splits
into one piece and returns `-1234.50`.

## 5.10 Parsing dates and fetching binary values

**Dates.** Each cell of a date column is read with the pattern in force. A cell that does not match
raises [`business-rules.md#aut-033`](business-rules.md#aut-033) or, when it matches no pattern at
all, [`business-rules.md#aut-034`](business-rules.md#aut-034). Cells that the reader already turned
into dates are written out with the platform's storage patterns.

**Binary values.** A cell of a binary column that looks like a web address is fetched, but only when
the reader is allowed to ([`business-rules.md#aut-031`](business-rules.md#aut-031)). The fetch is
made with a session reused across the whole import.

```formula
maximum size = the configured maximum size of an uploaded file, in bytes
```

The announced length is checked against it before reading, and the running total is checked again
after every block of 32 768 bytes; either check raises
[`business-rules.md#aut-035`](business-rules.md#aut-035). A transport failure raises
[`business-rules.md#aut-036`](business-rules.md#aut-036). The bytes are then encoded for storage. A
cell of an image column that is neither an address nor valid encoded bytes raises
[`business-rules.md#aut-032`](business-rules.md#aut-032).

**File names.** When a binary column also carries a file name, the name is extracted into a separate
answer and the cell is emptied, so that the loader stores the bytes and the caller can rename the
attachment afterwards.

## 5.11 Merging columns mapped onto the same field

The mapping is turned into an ordered register from field path to the list of column positions that
carry it. The fields of the import become the keys of that register, in the order of their first
appearance. For each row and each field:

| Kind of the target field | Separator | Extra handling |
|---|---|---|
| single line text | one space | the pieces are trimmed at the end first when the field trims |
| long text | one line break | none |
| rich text | the line-break element `<br>` | none |
| multiple link | one comma | the raw values are joined, without trimming |
| property holder | — | only the first column is used, stringified |
| anything else | — | only the first column is used |

Empty cells contribute nothing to a join. The target kind is found by walking the field path: each
part but the last retargets the record type, and the last part — with anything after a full stop
removed, so that a property is resolved to its holder — names the field whose kind decides.

**Worked example.** Two columns are both mapped onto the single line text field `name`; the row
holds `Value part 1` and `Value part 2`. The merged value is `Value part 1 Value part 2`. A third row
holds `I am Batman` and an empty cell; the merged value is `I am Batman` with no trailing space.

## 5.12 Applying fallback values

Fallback values are supplied per field as a triple of the value to fall back to, the record type the
field belongs to and the field's kind.

1. For every selection field named, the accepted values are read and lower-cased.
2. For every cell of every row whose field has a fallback:
   - a boolean field keeps its value when the value, lower-cased, is one of `0`, `1`, `true` or
     `false`, and otherwise takes the fallback;
   - a selection field keeps its value when the value, lower-cased, is among the accepted labels,
     and otherwise takes the fallback — unless the fallback is the marker `skip`, in which case the
     cell becomes empty, which makes the loader skip the row.

**Worked example.** The field `state` is a selection whose labels are "New", "In progress" and
"Done"; the fallback is `draft`. A cell holding `PENDING` does not match any label, so it becomes
`draft`. A cell holding `done` matches the label "Done" case-insensitively and is kept.

## 5.13 What the run hands to the loader

The rows are handed over with five reading-context entries: the marker that a file is being
imported, the fields allowed to create a missing linked record from its name, the fields whose
unrecognised values become empty, the fields whose unrecognised values skip the row, and the batch
size. The loader answers with the created identifiers, the failures and the position of the next row
to process.

```formula
reported next row = the loader's next row + number of rows skipped
```

```formula
answered names = as many empty texts as rows were skipped
                 + the names the loader produced
                 + as many empty texts as are needed to reach the full row count
```

The names are answered only when one of the imported fields is the display-name field of the record
type; otherwise the answer carries an empty list.

**Worked example.** A file of 1 000 data rows is imported in batches of 400. The first run skips 0
and the loader reports 400; the answer says 400. The client resubmits with a skip of 400; the loader
reports 400 again and the answer says 800. The third run reports 200 and the answer says 1 000, at
which point the client stops.

---

# 6. Package archive arithmetic

## 6.1 The size limit

```formula
maximum member size = 100 × 1 024 × 1 024 = 104 857 600 bytes
```

Every member of an uploaded archive is checked against it before anything is extracted, and every
manifest of a downloaded archive is checked again while its dependencies are examined.

## 6.2 What is extracted

Only two families of member are written to the working directory: the data files the manifest names
and every member under the package's static directory. Everything else in the archive is ignored,
which keeps an archive from writing arbitrary files. A data file whose extension the loader does not
recognise is ignored as well.

## 6.3 Dependency ordering

1. For each package in the archive, the set of names it depends on is read from its manifest.
2. Those already installed are removed.
3. Of the remainder, the names that the installation does not know at all raise
   [`business-rules.md#aut-046`](business-rules.md#aut-046).
4. The rest are installed first, in the platform's own order, before the archive's package is
   installed.
5. Packages inside the archive that depend on one another are installed in the order their manifests
   are found, which is the sorted order of their paths inside the archive.

## 6.4 Asset paths

```formula
stored asset path = the declared path when it already starts with an oblique stroke,
                    otherwise an oblique stroke followed by the declared path
```

A declared path containing a wildcard character raises
[`business-rules.md#aut-048`](business-rules.md#aut-048), because an imported package's files are
served from attachments and a wildcard cannot be resolved against them.

## 6.5 The activation-request expansion

Given the package a user asked for:

```formula
applications shown = the package itself
                     + every dependency it requires, directly or indirectly, that is itself an application
                     + every dependency required, directly or indirectly, by each of those applications
```

The set is deduplicated. The list is rendered on the reviewer's screen so that the reviewer sees
what activating one package would actually bring in.

**Worked example.** Package A depends on B and C. B is an application and depends on D. C is not an
application. The list is A, B and D: A because it was asked for, B because it is an application, D
because B requires it. C is left out because it is neither an application nor required by one of the
applications in the set.

---

# 7. Tokens, digests and constant-time comparison

## 7.1 The account token

A token is thirty-two lower-case hexadecimal characters, generated from a universally unique
identifier with its hyphens removed. The stored field accepts up to forty-three characters, which
leaves room for the suffix `+disabled` — nine characters — appended when the database is a
neutralised copy.

```formula
32 + 9 = 41 ≤ 43
```

## 7.2 The cross-site protection token

```formula
protection token = keyed digest over ( purpose , ( transport name of the record , identifier of the record ) )
```

The purpose is the fixed text `google_gmail_oauth` for the first electronic mail provider and
`microsoft_outlook_oauth` for the second. The key is the installation's own secret. The digest is
computed with elevated rights so that a reader who cannot read the record still receives a token
that the callback will accept for that record and no other.

## 7.3 Constant-time comparison

Two comparisons in this domain are made in constant time, so that the number of matching leading
characters cannot be inferred from how long the comparison took:

| Comparison | Where |
|---|---|
| The protection token returned by a provider against the recomputed one | the two delegated-authorisation callbacks |
| The token echoed by the metered service against each stored account token | the balance refresh |

## 7.4 The documentation cache key

```formula
cache key = keyed digest over ( scope , ( registry sequence , reading language , sorted group identifiers ) )
```

The scope is the fixed text `/doc/index.json`. The cached document is stored as an attachment named
`odoo-doc-index-<registry sequence>-<cache key>.json`, a reproduced name because the housekeeping
sweep matches on it. The sweep deletes every such attachment whose fourth hyphen-separated part is
not the current registry sequence.

---

# 8. Sparse storage

A field declared sparse has no column of its own. Its value lives inside a mapping stored in another
field of the record, the serialisation field, which is of the serialised kind.

```formula
stored mapping = { identifier of a sparse field : its value , for every sparse field that has one }
```

**Reading.** A key that is absent yields the field's own empty value: false for a boolean, zero for
a number, the empty text for text, an empty set for a link.

**Writing.** A value that is not empty is written under the field's identifier. A value that is
empty removes the key, so that an unset field costs nothing.

**What may not change.** The serialisation field of an existing field cannot be changed
([`business-rules.md#aut-139`](business-rules.md#aut-139)) and a sparse field cannot be renamed
([`business-rules.md#aut-140`](business-rules.md#aut-140)), because the identifier is the key.

**Worked example.** A record has the serialisation field `data` and three sparse fields: `boolean`,
`integer` and `partner`. Writing true, 42 and the Contact 3814 stores the mapping with three keys.
Writing zero into `integer` afterwards leaves a mapping with two keys, and reading `integer` yields
zero. The saving over three separate columns is one column instead of three, and no storage at all
for the records that use none of them.

---

# 9. Recycling thresholds and notification cadence

## 9.1 The age threshold

When a rule names a time field, a delta and a unit, the search gains one more condition:

```formula
threshold = now − ( delta × one unit )
condition = the time field is at or before the threshold
```

*Now* is today's date when the time field is a plain date, and the current instant when it is a date
and time. The unit is one of days, weeks, months or years, each computed as a calendar quantity, so
that a month added to 31 January lands on 28 or 29 February.

**Worked example.** The rule watches a record type on the field `write_date`, with a delta of 6 and
the unit months; the search runs on 12 September 2026 at 03:00. The threshold is 12 March 2026 at
03:00, and every record whose last write is at or before that instant becomes a candidate.

**Worked example at a month boundary.** Delta 1, unit months, the search runs on 31 March 2026. The
threshold is 28 February 2026, because February has no thirty-first day and the calendar arithmetic
clamps to the last day of the month.

## 9.2 The notification cadence

```formula
duration = frequency × one period
```

The period is days, weeks or months. A notification is sent when the rule has never notified, or
when

```formula
last notification + duration < now
```

On sending, the last-notification stamp is set to the current instant **before** the notification is
prepared, so a failure while preparing it still moves the cadence forward.

```formula
counted candidates = the number of candidates of this rule created on or after ( today − duration )
```

When that count is zero, nobody is notified, although the stamp has already moved.

**Worked example.** Frequency 1, period weeks. The last notification was on 4 September 2026 at
03:00; the search runs on 12 September 2026 at 03:05. 4 September + 7 days = 11 September, which is
before 12 September, so a notification is due. The stamp becomes 12 September 03:05. The count is
taken over candidates created on or after 5 September 2026. With 143 such candidates the body reads
"We've identified 143 records to clean with the 'Contact' recycling rule." followed by the line
offering the link.

## 9.3 Batch sizes

| Mode | Batch | Behaviour per batch |
|---|---|---|
| automatic | 5 000 | create the candidates and validate them at once; commit outside a test run |
| manual | 50 000 | create the candidates; commit outside a test run |

---

# 10. Company enrichment

## 10.1 The internet domain of a company

1. When the company has an electronic mail address, take the part after the at-sign. When that part
   is **not** one of the well-known consumer mail providers, it is the domain and the procedure
   stops.
2. Otherwise, when the company has a site address, take its host and strip a leading `www.`.
3. When the result is empty, or is `localhost`, or is `example.com`, there is no domain.

**Worked examples.**

| Electronic mail address | Site address | Domain |
|---|---|---|
| `info@proximus.be` | — | `proximus.be` |
| `someone@gmail.com` | `www.info.proximus.be` | `proximus.be` — the address is a consumer provider, so the site wins |
| `someone@gmail.com` | `http://localhost:8069` | none |
| — | — | none |

## 10.2 Translating a suggestion into records

Each suggestion arrives with codes and names; three passes turn them into links.

**Country and state.** The country is sought by code, compared case-insensitively; failing that, by
name, compared case-insensitively. When a country was found, the state is sought within it by code
and then by name, taking the first match. Each found record is written back into the suggestion as a
pair of identifier and display name, under `country_id` and `state_id`; the four original keys are
removed.

**City.** Only when the extended-address package is installed and the country enforces a city list:
the city is sought within the country, and within the state when one was found, first by postal code
and then by name compared case-insensitively. When one is found, the suggestion gains `city_id`, its
`state_id` is replaced by that city's state, and the plain city name is removed.

**Industry.** The industry code is looked up among the shipped industry records by its external
identifier `base.res_partner_industry_<code>`. When found, the suggestion gains `industry_id`.

**Language.** The preferred language code is sought first as an exact match on both the stored code
and the standard code; failing that, as a match on the first two characters of both. When found, the
suggestion gains the installed language's stored code under `lang`.

**Worked example.** A suggestion arrives with country code `BE`, state code `WBR`, postal code
`1367`, city `Ramillies`, industry code `62`, preferred language `fr_BE`. Belgium is found by code;
the state Walloon Brabant is found by code within Belgium; with the extended-address package
installed and Belgium enforcing a city list, the city Ramillies is found by postal code, so
`city_id` is set, `state_id` is replaced by that city's state and the plain city name is dropped;
the industry with the external identifier `base.res_partner_industry_62` is found; the language
`fr_BE` is found exactly.

## 10.3 Processing an enrichment answer

```formula
result = the translated data of the answer when the answer carries data, otherwise an empty result
```

Then, in this order, the first matching case adds a failure to the result:

| Case | Added |
|---|---|
| The answer carries the balance-exhausted marker | the failure flag and the message `Insufficient Credit` |
| The answer carries any other failure | the failure flag and the message "Unable to enrich company (no credit was consumed)." |
| The call itself failed | the failure flag and the failure's own text |

Finally, when the tax-number validation package is installed, the result carries a tax registration
number and the reading context carries the enriched company data, the number is checked against the
country of that data and **silently emptied** when it does not pass.

The refusals of the call itself are mapped as follows before this table is reached: a refusal because
a test is running becomes `Insufficient Credit`; a connection failure, a failing status, an access
refusal or a user-facing refusal becomes its own text; the insufficient-balance failure becomes
`Insufficient Credit`; a missing token becomes `No account token`.

## 10.4 The cross-border verification fallback

When the search by tax registration number returns nothing, the cross-border verification service is
consulted directly with the same timeout. A valid answer whose name is not the three-hyphen
placeholder `---` becomes one suggestion:

1. The answered address is split on line breaks and empty lines are dropped.
2. The first line becomes the street.
3. The first line **after the first** that begins with a digit is split once on a space; the part
   before becomes the postal code and the part after becomes the city.
4. The first line after the first that is not that postal-code line becomes the second street line.
5. The answered country code and the queried number complete the suggestion, which is then passed
   through the country and state translation of §10.2.

A failure of the verification service is logged as a warning and treated as no answer.

**Worked example.** The service answers with the name `EXAMPLE SPRL`, the country code `BE` and the
address `Chaussée de Namur 40\n1367 Ramillies\nBelgium`. The street is `Chaussée de Namur 40`; the
first following line beginning with a digit is `1367 Ramillies`, so the postal code is `1367` and the
city is `Ramillies`; the remaining line `Belgium` becomes the second street line; the country is
Belgium.

## 10.5 The automatic one-time enrichment

The enrichment of a newly created company writes back only the keys that

1. name a real field of the company's Contact,
2. carry a value, and
3. are either the main image field or a field the Contact has not already filled.

The country and the state, which arrive as pairs of identifier and display name, are reduced to
their identifiers before the write. The flag that records that the enrichment has run is set on
every company of the batch afterwards, whether or not each enrichment succeeded, which is what makes
the procedure impossible to loop.

---

# 11. The privacy query

One query is assembled and run once. It has three parts joined by union, all preserving duplicates.

**The shared sub-query.** Contacts whose normalised address equals the normalised address sought, or
whose name matches the name sought with the wildcard pattern `%name%`, case-insensitively.

**Part one — Contacts and Users.**

| Row source | Condition | Activation column reported |
|---|---|---|
| Contact | the identifier is in the shared sub-query | the Contact's own activation flag |
| User | the login matches `%address%` case-insensitively, or the User's Contact matches the address with `%address%` or the name with `%name%` | the User's own activation flag |

**Part two — messages.** Every message whose author is in the shared sub-query; the activation column
is reported as true, since a message has none.

**Part three — every other record type.** Every record type that is neither excluded nor transient
and that has a table of its own contributes one part when at least one condition applies to it:

1. **Address-like fields.** The first present and stored field among `email_normalized`, `email`,
   `email_from` and `company_email` contributes a condition. A normalised field is compared for
   equality with the normalised address; any other is compared with `%address%`
   case-insensitively. The scan of the four names stops at the first normalised one.
2. **The display-name field.** When the record type's display-name field is present, stored, of the
   single-line-text kind and not translated, a condition on `%name%` is added beside the
   address condition.
3. **Links to a Contact.** Every stored single link whose target is the Contact record type and
   whose deletion behaviour is **not** cascade contributes a condition that the link is in the
   shared sub-query. Cascading links are left out because deleting the Contact would remove those
   rows anyway.

The reported activation column is the record type's own activation flag when it has one, and true
otherwise.

**The excluded record types.**

| Record type | Reason |
|---|---|
| `res.partner` | already covered by part one |
| `res.users` | already covered by part one |
| `mail.notification` | removed by cascade with the Contact |
| `mail.followers` | removed by cascade with the Contact |
| `discuss.channel.member` | removed by cascade with the Contact |
| `mail.message` | already covered by part two |

**Order of operations.** Everything pending is written to the database first, then the query is run,
then the whole line list of the wizard is replaced by the result.

**Worked example.** Looking up "Jean-Luc Picard" with the address `jl.picard@example.org` in a small
installation returns: the Contact itself; the User whose login is that address; three messages the
Contact wrote; two sales orders whose customer link points at the Contact; and one event
registration whose own address field matches. Seven lines, grouped by record type in the list.

---

# 12. Anonymisation

The name and the address stored on a Privacy Log are masked before they are written, so that the log
proves a request was handled without repeating the personal data.

## 12.1 Masking a name

1. An empty value yields the empty text.
2. A value containing an at-sign is masked as an address instead, by §12.2.
3. Otherwise the value is split on spaces; empty pieces are dropped; each remaining piece keeps its
   first character and replaces every other character with an asterisk; the pieces are rejoined with
   single spaces.

```formula
masked piece = first character + ( length of the piece − 1 ) asterisks
```

**Worked example.** `Jean-Luc Picard` has two pieces. `Jean-Luc` is eight characters, so it becomes
`J` followed by seven asterisks: `J*******`. `Picard` is six characters, so it becomes `P*****`. The
result is `J******* P*****`.

## 12.2 Masking an address

1. An empty value, or a value with no at-sign, is handled by
   [`business-rules.md#aut-097`](business-rules.md#aut-097).
2. Otherwise the value is split once on the at-sign into a local part and a domain.
3. The local part is split on full stops; empty pieces are dropped; each keeps its first character
   and replaces the rest with asterisks; the pieces are rejoined with full stops.
4. The domain is compared against the three most common consumer domains — `gmail.com`,
   `hotmail.com` and `yahoo.com`. When it is one of them it is kept whole, because masking it would
   remove no information. Otherwise the domain is split on full stops, every label but the last is
   masked as in step 3, the last label is kept whole, and the labels are rejoined with full stops.
5. The two parts are rejoined with an at-sign.

**Worked example one.** `jean.luc.picard@enterprise.example.org`. The local part gives `j***`, `l**`
and `p*****`, rejoined as `j***.l**.p*****`. The domain is not a common one, so `enterprise` becomes
`e*********`, `example` becomes `e******` and `org` is kept: `e*********.e******.org`. The result is
`j***.l**.p*****@e*********.e******.org`.

**Worked example two.** `jlpicard@gmail.com`. The local part becomes `j*******`; the domain is kept
whole. The result is `j*******@gmail.com`.

## 12.3 The record description

For each record type found, one line:

```formula
line = display name of the record type + " (" + number of records + "): " + the identifiers
```

Each identifier is written as a hash sign followed by the number, and they are joined by a comma and
a space. For a reader in the technical-features group the record type is written as its display name,
a space, a hyphen, a space and its transport name.

**Worked example.** Two Contacts with the identifiers 3814 and 3815 and one User with the identifier
27 produce, for an ordinary administrator:

```
Contact (2): #3814, #3815
User (1): #27
```

and, for a reader in the technical-features group:

```
Contact - res.partner (2): #3814, #3815
User - res.users (1): #27
```

## 12.4 The execution details

Each acted-upon line contributes one text; the wizard's own execution details are those texts joined
by line breaks, in line order.

| Action | Text |
|---|---|
| Archive | `Archived ` + display name of the record type + ` #` + identifier |
| Unarchive | `Unarchived ` + display name of the record type + ` #` + identifier |
| Delete | `Deleted ` + display name of the record type + ` #` + identifier |

---

# 13. Geocoding and address completion

## 13.1 The default query string

```formula
query = the non-empty members of [ street , ( postal code + " " + city ) trimmed , state name , country name ]
        joined by ", "
```

The postal code and the city are combined first, each replaced by the empty text when absent, and
the pair is trimmed at both ends; when both are absent the pair is empty and drops out of the join.

**Worked example.** Street `Chaussée de Namur 40`, postal code `1367`, city `Ramillies`, no state,
country `Belgium`.

```formula
pair = ( "1367" + " " + "Ramillies" ) trimmed = "1367 Ramillies"
query = "Chaussée de Namur 40" + ", " + "1367 Ramillies" + ", " + "Belgium"
```

The query is `Chaussée de Namur 40, 1367 Ramillies, Belgium`.

**Second attempt.** When the first attempt finds nothing, the query is rebuilt from the city, the
state and the country only, so that at least the town is placed. With the same address the second
query is `Ramillies, Belgium`.

## 13.2 The second provider's country reordering

The second provider mis-parses inverted country names, so the name is reordered first:

```formula
When the country name contains a comma and ends with " of" or " of the":
    reordered name = the part after the first comma + " " + the part before the first comma
Otherwise:
    reordered name = the country name unchanged
```

The reordered name then enters §13.1 in place of the country name.

**Worked example.** `Congo, Democratic Republic of the` contains a comma and ends with ` of the`.
The part after the first comma is ` Democratic Republic of the`, retaining the space that followed
the comma, and the part before is `Congo`. The reordered name is ` Democratic Republic of the
Congo`.

**Compatibility finding.** The space that followed the comma is kept, so the reordered name begins
with a space and the joined query contains two consecutive spaces before the country. The provider
ignores the extra space, so the result is unaffected; a corrected implementation would trim the two
parts before joining.

## 13.3 Reading a place back from coordinates

The reverse lookup is used to describe where a visitor is. The city and the country code are taken
from the request's own geographic information when both are present; otherwise the first provider is
asked to reverse the coordinates, with a ten-second timeout, and the answer supplies the country
code, the postal code and the city — the city being, in order of preference, the district, the town,
the village or the city of the answer.

```formula
description = postal code
              + ( " " + city ) when a city was found and the description is not empty
              + ( city ) when a city was found and the description is empty
              + ( ", " + country name ) when a country was found and the description is not empty
              + ( country name ) when a country was found and the description is empty
```

When the description is still empty the fixed text "Unknown" is used.

**Worked example.** Postal code `1367`, city `Ramillies`, country Belgium. The description starts as
`1367`, gains ` Ramillies` and then `, Belgium`: `1367 Ramillies, Belgium`. With no postal code it
would be `Ramillies, Belgium`; with nothing at all, `Unknown`.

## 13.4 Turning answered address components into fields

Each answered component carries a long name, a short name and a list of kinds. The kinds are reduced
to one:

```formula
kind of a component = the first of its kinds that the mapping recognises,
                      or, when none is recognised, its first kind
```

The components are then sorted by the priority order below; components whose kind is not in the
order sort last, keeping their relative order.

| Priority | Kind | Fields it fills |
|---|---|---|
| 1 | `country` | country |
| 2 | `street_number` | house number |
| 3 | `locality` | city |
| 4 | `postal_town` | city |
| 5 | `route` | street |
| 6 | `postal_code` | postal code |
| 7 | `administrative_area_level_1` | state, then city |
| 8 | `administrative_area_level_2` | state, then city |
| — | `sublocality_level_1` | second street line |

They are then applied in that order, and **the first component to fill a field wins**: a later
component naming an already filled field is skipped. That is why the country is placed first — the
state lookup needs it — and why a locality beats an administrative area for the city.

- A country component is resolved to the country whose code equals its short name, upper-cased.
- A state component is resolved within the already-found country, by code equal to its short name
  upper-cased **or** by name matching its long name case-insensitively; the result is used only when
  exactly one state matches. A state component met before a country has been placed is skipped and a
  warning is logged.
- Every other component contributes its long name.

Finally, when a city name and a country are present, the city list is consulted; a found city adds
the city record and, when no state has been placed, that city's state.

**Worked example.** The answer carries, in the provider's order: `route` "Chaussée de Namur",
`street_number` "40", `locality` "Ramillies", `administrative_area_level_1` "Brabant wallon" with
the short name "WBR", `country` "Belgium" with the short name "BE", `postal_code` "1367". Sorted by
priority the order becomes country, street number, locality, route, postal code, administrative
area. Belgium is placed first; then the house number 40; then the city Ramillies; then the street;
then the postal code; finally the administrative area, which fills the state Walloon Brabant and is
skipped for the city because Ramillies already filled it.

## 13.5 Guessing the house number

When the answer carries no house number:

```formula
guess = the text the user typed
        with the postal code removed
        with the street removed
        with the city removed
        then cut at the first comma
        then trimmed
```

Each removal removes every occurrence of that text; a field that was not filled removes the empty
text, which changes nothing.

**Worked example.** The user typed `40 Chaussée de Namur, 1367 Ramillies`. Removing `1367` gives
`40 Chaussée de Namur,  Ramillies`; removing `Chaussée de Namur` gives `40 ,  Ramillies`; removing
`Ramillies` gives `40 ,  `. Cutting at the first comma gives `40 `, and trimming gives `40`.

## 13.6 The formatted street and number

When the answer carried a house number, two candidates are compared:

```formula
candidate one = the plain text of the part of the rich address before its first comma
candidate two = house number + " " + street , trimmed
formatted street and number = candidate one when its length is greater than or equal to
                              the length of candidate two , otherwise candidate two
```

The comparison exists because the provider sometimes abbreviates: it may answer `52 Hgh Rd St` where the
components say `52 High Road Street`. The longer text is assumed to be the unabbreviated one.

**Worked example.** Candidate one is `52 Hgh Rd St`, twelve characters; candidate two is
`52 High Road Street`, nineteen characters. Nineteen is greater than twelve, so candidate two
wins.

When the answer carried no house number, the formatted value is the guess of §13.5 followed by a
space and the street, trimmed.

---

# 14. Signed addresses for outside storage

## 14.1 The blob name and the plain address

```formula
blob name = identifier of the attachment + "/" + a fresh universally unique identifier + "/" + file name
```

The plain address identifies the blob and is stored on the attachment; it carries no signature and
grants no access.

| Provider | Plain address |
|---|---|
| first | `https://` + account name + `.blob.core.windows.net/` + container name + `/` + percent-encoded blob name |
| second | `https://storage.googleapis.com/` + bucket name + `/` + percent-encoded blob name |

Reading an address back checks it against the provider's own shape; a mismatch raises "%s is not a
valid Azure Blob Storage URL." or "%s is not a valid Google Cloud Storage URL.", the placeholder
being the address.

**Worked example.** Attachment 918, file name `contract.pdf`, a fresh identifier
`5f2b1c7e-2f14-4a5f-9f2b-1c7e2f144a5f`, account `examplestore`, container `attachments`. The blob
name is `918/5f2b1c7e-2f14-4a5f-9f2b-1c7e2f144a5f/contract.pdf` and the plain address is
`https://examplestore.blob.core.windows.net/attachments/918%2F5f2b1c7e-2f14-4a5f-9f2b-1c7e2f144a5f%2Fcontract.pdf`.

## 14.2 The first provider's signature

The signature is a keyed digest over a text assembled from the signed values, each followed by a
line break, in this exact order:

1. the permissions, `r` for a download and `c` for an upload;
2. the start instant, empty when none is given;
3. the expiry instant;
4. the canonical resource, which is `/blob/` + account name + `/` + container name + `/` + blob
   name;
5. the delegation key's object identifier, tenant identifier, start, expiry, service and version;
6. three empty values reserved for authorised, unauthorised and correlation identifiers;
7. the address restriction, empty;
8. the protocol restriction, empty;
9. the service version `2023-11-03`;
10. the resource kind `b`;
11. the snapshot timestamp, empty;
12. the encryption scope, empty;
13. the cache-control, disposition, encoding, language and media-type overrides.

The trailing line break is removed. The digest is the two hundred and fifty-six bit keyed secure
hash, taken with the delegation key's own value as the key, and written in the standard sixty-four
character encoding. The query string is then the signed values under their published names, with the
snapshot timestamp left out, each percent-encoded and joined by ampersands, plus the signature.

```formula
signed address = plain address + "?" + query string
```

**The delegation key.** It is fetched with the account's tenant, client and secret, valid for seven
days, and cached per database. The cache is used again only when the configuration and the key
sequence are unchanged and the key's expiry is more than one day away; a failure to obtain it is
cached too, so a wrong configuration is not retried on every request. Raising the system parameter
`cloud_storage_azure_user_delegation_key_sequence` by one invalidates the cache.

## 14.3 The second provider's signature

1. The expiry is expressed in seconds.
2. The current instant supplies a timestamp in the compact date-and-time form and a datestamp in the
   compact date form.
3. The credential scope is `<datestamp>/auto/storage/goog4_request`; the credential is the service
   account's address, an oblique stroke and the scope.
4. The headers are the host of the endpoint plus any given media type; they are canonicalised —
   lower-cased names, sorted, values trimmed — and the list of their names, joined by semicolons,
   becomes the signed-header list.
5. The query carries the algorithm `GOOG4-RSA-SHA256`, the credential, the timestamp, the expiry in
   seconds, the signed-header list, and, when asked for, the response media type and the response
   disposition. The entries are sorted by name and percent-encoded.
6. The canonical request is the request method, the resource, the canonical query, the canonical
   headers followed by an extra line break, the signed-header list and the payload marker
   `UNSIGNED-PAYLOAD`, joined by line breaks.
7. That request is hashed with the two hundred and fifty-six bit secure hash and written as
   hexadecimal.
8. The text to sign is the algorithm, the timestamp, the credential scope and that hash, joined by
   line breaks.
9. The text is signed with the service account's private key and the signature is written as
   hexadecimal.

```formula
signed address = endpoint + resource + "?" + canonical query + "&X-Goog-Signature=" + signature
```

## 14.4 Lifetimes and caching

| Quantity | Value |
|---|---|
| Lifetime of an upload address | 300 seconds |
| Lifetime of a download address | 300 seconds, overridable per request through the reading context |
| Maximum age of the cached redirection | the lifetime minus ten seconds, never below zero |
| Maximum age announced in the cross-origin rules | the download lifetime |

```formula
maximum age of the redirection = the larger of 0 and ( lifetime − 10 seconds )
```

**Worked example.** With the standard lifetime the redirection to the signed address may be cached
for 300 − 10 = 290 seconds, so a client that reuses a cached redirection still has ten seconds of
validity left when it follows it.

## 14.5 The download and upload descriptions

| Provider | Upload method | Success status | Upload headers |
|---|---|---|---|
| first | `PUT` | 201 | the blob kind `BlockBlob` under `x-ms-blob-type`, plus the media type when the attachment has one |
| second | `PUT` | 200 | the media type when the attachment has one, otherwise none |

The download description carries the signed address and the remaining lifetime. When the reading
context asks for a download rather than a display, the disposition override is added so that the
provider serves the blob as an attachment with its own file name.

## 14.6 The minimum file size

```formula
stored minimum in bytes = the minimum in megabytes × 1 000 000 , truncated to a whole number
shown minimum in megabytes = the stored minimum in bytes ÷ 1 000 000
```

The default is 20 000 000 bytes, that is 20 megabytes. The client is told the byte figure in the
session description and offers an outside upload only for files at or above it.

**Worked example.** An administrator types 12.5 megabytes. The stored value is 12 500 000 bytes. The
form shows 12.5 again on the next visit.

---

# 15. Delegated authorisation strings

## 15.1 The authentication string

```formula
authentication string = "user=" + mailbox address
                        + control character one
                        + "auth=Bearer " + access token
                        + control character one
                        + control character one
```

The control character one is the byte with the value 1. The string is then encoded in the standard
sixty-four character encoding and sent as the argument of the authentication command. Both providers
build it identically; only the token differs.

**Worked example.** Mailbox `sales@example.test`, access token `ya29.EXAMPLE`. The string is
`user=sales@example.test` followed by the control character, `auth=Bearer ya29.EXAMPLE`, and two
control characters; encoded it becomes a single sixty-four character-set text of forty-eight
characters.

## 15.2 When the access token is renewed

```formula
The access token is renewed when it is missing,
                          or its expiry instant is missing,
                          or ( expiry instant − validity threshold ) < the current instant
validity threshold = token request timeout + 5 seconds = 5 + 5 = 10 seconds
```

The instants are counted in whole seconds since the epoch. The threshold exists so that a token does
not expire between the check and the opening of the mail session.

**Worked example.** The current instant is 1 789 000 000 and the stored expiry is 1 789 000 008. The
expiry minus ten is 1 788 999 998, which is below the current instant, so the token is renewed even
though it has eight seconds left.

```formula
new expiry instant = the current instant at the moment of the exchange + the lifetime the provider returned
```

For the second provider the exchange also returns a **new refresh token**, which replaces the stored
one; for the first it does not.

## 15.3 The consent address

```formula
consent address of the first provider = a fixed consent endpoint + "?" + encoded query
consent address of the second provider = the configured base address + "authorize" + "?" + encoded query
```

| Query entry | First provider | Second provider |
|---|---|---|
| application identifier | yes | yes |
| return address | the base address + `/google_gmail/confirm` | the base address + `/microsoft_outlook/confirm` |
| response kind | `code` | `code` |
| scope | the provider scope | the base scope plus the host record's own scope |
| access kind | `offline` | — |
| prompt | `consent` | — |
| response mode | — | `query` |
| state | the record's transport name, its identifier and the protection token of §7.2 | the same three |

The access kind and the prompt on the first provider are what make it return a refresh token; the
second provider returns one because `offline_access` is part of its base scope.

---

# 16. Translation platform links

## 16.1 The project map

The map from package to translation project is read once per process from the platform configuration
files found beside the package directories, at two places for each: inside the directory and one
level above it. Each file's first section is skipped; each later section is read in one of two
shapes:

| Shape | Reading |
|---|---|
| the section name has six colon-separated parts | the fourth part is the project and the sixth is the package |
| otherwise | the section name is split on the full stop; the first part is the project and the second is the package |

The result is a mapping from package name to project name; a package met twice keeps the last
reading.

## 16.2 The address of a term on the platform

Nothing is produced when the base address is empty, when no language has a standard code, when the
map is empty, when the term has no source, when the term's language is the base language, when the
term's language has no standard code, or when the term's package is not in the map.

```formula
shortened source = the first 50 characters of the source term
                   with every line break removed
                   with every apostrophe replaced by a backslash and an apostrophe
encoded source = the shortened source, percent-encoded with spaces written as plus signs
quoted source  = "'" + encoded source + "'" when the encoded source contains a plus sign,
                 otherwise the encoded source
address = base address with any trailing oblique stroke removed
          + "/" + project
          + "/translate/#" + standard language code
          + "/" + package
          + "/42?q=text%3A" + quoted source
```

The number 42 is a fixed filler that the platform's address shape requires and that carries no
meaning.

**Worked example.** Base address `https://app.transifex.com/odoo`, project `odoo-19`, standard
language code `fr`, package `sale`, source term `Sales Order`. The shortened source is
`Sales Order`; the encoded source is `Sales+Order`; it contains a plus sign, so the quoted source is
`'Sales+Order'`. The address is
`https://app.transifex.com/odoo/odoo-19/translate/#fr/sale/42?q=text%3A'Sales+Order'`.

**Second worked example.** The source term is `Invoice`. The encoded source is `Invoice`, which
contains no plus sign, so it is not quoted, and the address ends `/42?q=text%3AInvoice`.

## 16.3 Loading the term cache

1. The cache table is locked exclusively without waiting; a failure to take the lock abandons the
   load ([`business-rules.md#aut-141`](business-rules.md#aut-141)).
2. The packages considered are the installed ones, unless the caller named some.
3. The languages considered are the installed ones except the base language, unless the caller named
   some.
4. The pairs of package and language already present in the table are read.
5. For every pair not yet present, every source term and its translation are written as one row
   carrying the source, the value, the package and the language.

Reloading first empties the table and then performs the same load, which is what the weekly job does.

---

# 17. Tour addresses

## 17.1 The sharing address

```formula
sharing address = base address + "/odoo?tour=" + tour name
```

**Worked example.** Base address `https://example.test`, tour name `sale_quote_tour`. The sharing
address is `https://example.test/odoo?tour=sale_quote_tour`.

## 17.2 The exported description

A tour exports as one text file named after the tour with the extension of a client script. Its body
declares the tour under its own name with two entries: the starting address and the list of steps.
Each step is written with its trigger, its instruction, its tooltip position — under the name
`tooltipPosition` — and its content when the content is not empty. The identifiers of the rows are
removed. The file is stored as an attachment on the tour and the browser is sent to its download
address.

## 17.3 Which tour is offered

```formula
candidate tours = tours that are not custom and that the reading user has not consumed
offered tour    = the first candidate in the order sequence, name, identifier
```

Nothing is offered unless the reading user has the tour switch on and is an internal user.

```formula
tour switch default = the user is an administrator
                      and no package with demonstration data is installed
                      and no test is running
```

Consuming a tour links the reading user to it with elevated rights and then answers with the next
tour to offer, so a client can chain them.

---

# 18. Attachment content extraction

Five families of document are turned into indexable text. The families are tried in this fixed
order, and the first that returns anything wins; when none does, the platform's own extraction is
used instead.

```formula
order = current word-processing document,
        current presentation document,
        current spreadsheet workbook,
        open document family,
        portable document
```

Every extracted text has the null character removed before it is stored.

## 18.1 The cache

One extraction is remembered at a time, keyed by the attachment's checksum. A second attachment with
the same checksum reuses the remembered text without reading the bytes. Duplicating an attachment
seeds the cache with the original's text under the original's checksum, so the copy costs nothing.

## 18.2 Whitespace cleaning

Every extractor but the word-processing one finishes by cleaning the text:

1. Every run of whitespace that contains no line break becomes a single space.
2. Every run of whitespace that contains exactly one line break becomes a single line break.
3. Every run of whitespace that contains two or more line breaks becomes exactly two line breaks.
4. The result is trimmed at both ends.

## 18.3 The five extractors

| Family | What is read | Shape of the result |
|---|---|---|
| current word-processing document | every paragraph part of the main document | the paragraph texts, one per line |
| current presentation document | every text element of every slide | the slide texts, one per line |
| current spreadsheet workbook | every sheet, values only, read row by row | one line per non-empty row, each line beginning with the escaped sheet name followed by the escaped cells, separated by commas; sheets separated by a blank line |
| open document family | the main content part; when the declared media type mentions a spreadsheet, the sheets, otherwise the text | the same comma-separated shape for a spreadsheet, the paragraph texts otherwise; parts separated by a blank line |
| portable document | every page, laid out with vertical text detection on and text-box grouping off | the page texts in order |

```formula
escaped cell = the value wrapped in double quotation marks, with every inner double quotation mark doubled,
               when the value contains a comma, a double quotation mark, a line break or a carriage return
             = the value unchanged otherwise
```

An empty value escapes to the empty text.

**Worked example.** A workbook has one sheet named `Q1, draft` with the rows `Product | Qty` and
`Bolt "M8" | 12`. The sheet name contains a comma, so it is escaped to `"Q1, draft"`. The first row
becomes `"Q1, draft",Product,Qty`. The cell `Bolt "M8"` contains double quotation marks, so it
becomes `"Bolt ""M8"""`. The second row becomes `"Q1, draft","Bolt ""M8""",12`. Rows in which every
cell is empty are skipped.

## 18.4 When an extractor is unavailable

Each extractor needs its own reading component. When the component is absent the extractor returns
the empty text and the next family is tried; the absence of the portable-document reader is written
once to the log when the package is loaded, naming where the component can be obtained. A document
that is not of the family the extractor expects — a portable document must begin with the five
characters `%PDF-` — also returns the empty text at once.

---

# 19. Where these formulas are used

- The procedures that invoke them: [`workflows.md`](workflows.md).
- The refusals they raise: [`business-rules.md`](business-rules.md).
- The fields they read and write: [`entities.md`](entities.md).
- The numbers they must produce: [`acceptance-criteria.md`](acceptance-criteria.md).
