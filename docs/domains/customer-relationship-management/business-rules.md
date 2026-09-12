# Business rules

The complete rule catalogue of the Customer Relationship Management domain: validations, guards, permissions per operation, company and currency consistency, uniqueness, rounding, date rules, and the exact user-facing messages. Each rule carries a stable identifier so that the other documents of this domain and of neighbouring domains can cite it. A message written between backticks is reproduced literally; a placeholder written as `<name>` is substituted at run time.

Rules are grouped by subject and numbered in one single scheme, a short domain prefix followed by a three-digit sequence number, running from `LEAD-001` to `LEAD-189` in the order in which they appear below. Section 12 maps every identifier onto the identifier the rule carried in the description this file was consolidated from.

## 1. Lead identity, typing and required data

### LEAD-001 Title is mandatory

A Lead cannot be stored without a `name`. When the field is empty at creation and a customer with a name is set, the system fills it with `<customer name>'s opportunity` before the record is stored. When neither is available, the storage layer refuses the record.

### LEAD-002 Type is mandatory and has two values only

`type` is required and is one of `lead` or `opportunity`. The default at creation is `lead` when the acting user belongs to the access group **Show Lead Menu**, and `opportunity` otherwise.

### LEAD-003 Type never regresses automatically

No automatic operation writes `type = "lead"` on a record that is already an opportunity. Conversion, merging with an opportunity and automatic assignment all move a record forward to `opportunity`. A user with write access may still write the field directly.

### LEAD-004 Priority values

`priority` takes exactly one of `0` (Low), `1` (Medium), `2` (High), `3` (Very High). The default is `0`. Any other value is refused by the storage layer.

### LEAD-005 Probability bounds

A check constraint enforces `probability` is greater than or equal to zero and less than or equal to one hundred. Violation message:

> "The probability of closing the deal should be between 0% and 100%!"

### LEAD-006 A record in a won stage must have a probability of one hundred

For every Lead: `stage.is_won = true` implies `probability` equal to one hundred. The rule is validated after every create and every write that touches `probability` or `stage_id`. Violation message:

> "A lead in a Won stage cannot be lost. Move it to another stage first."

This single message covers three situations: marking lost a record that sits in a won stage, writing a probability below one hundred while the record sits in a won stage, and moving a record whose probability is below one hundred into a won stage without the write path that forces the probability (a direct write of `stage_id` does force it, see `LEAD-041`).

### LEAD-007 A record cannot be won and lost at the same time

During the won and lost bookkeeping that runs after every create and after every write touching `active`, `stage_id` or `probability`, a record whose new state is simultaneously won and lost is refused:

> "The lead <record> cannot be won and lost at the same time."

Because "won" requires a probability of one hundred and "lost" requires a probability of zero, this state is unreachable through the supported operations; the rule is a safety net for direct writes.

### LEAD-008 Salesperson must be an internal user

`user_id` may only reference a user that is not a shared (portal or public) user.

### LEAD-009 Company coherence of the salesperson, the team and the customer

The automatic company check applies to `user_id`, `team_id` and `partner_id`: when the Lead has a company, each of those records must either have no company or have the same company. The derivation of `company_id` described in [entities.md](entities.md) sections 1.4.1 to 1.4.3 keeps the four values coherent; a manual write that breaks the coherence is refused by the platform's company check with the platform's standard message.

### LEAD-010 Company may be empty

`company_id` may be empty. An empty company means the record belongs to no single company and is visible to every company (see `LEAD-135`). The derivation never invents a company when the record has no salesperson, no team and no customer.

### LEAD-011 Currency follows the company

`company_currency` is not writable. It is the currency of `company_id`, or the currency of the company active in the session when `company_id` is empty. All monetary fields of the Lead (`expected_revenue`, `prorated_revenue`, `recurring_revenue`, `recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue_prorated`, `sale_amount_total`) are expressed in that currency. No currency conversion happens inside the Lead; conversions only happen when reading amounts from Sales Orders (see `LEAD-124`).

### LEAD-012 Web address is normalized on write

Whenever `website` is present in the values of a create or a write, the value is passed through the contact domain's web address cleaning rule before being stored, so that `mycompany.example`, `www.mycompany.example` and `http://mycompany.example` are stored in the same canonical form. See [../contacts-and-organizations/business-rules.md](../contacts-and-organizations/business-rules.md).

### LEAD-013 Stage must belong to the team of the record

The selectable stages of a Lead are the stages with no team plus the stages whose team list contains the team of the Lead. The derivation replaces the stage when the current one is not selectable (see [entities.md](entities.md) section 1.4.3). A manual write of a stage outside that set is stored; the interface simply does not offer it.

### LEAD-014 Stage deletion is restricted

A Pipeline Stage referenced by at least one Lead cannot be deleted. The reference from Lead to Pipeline Stage is restrictive.

### LEAD-015 Lost reason deletion is restricted

A Lost Reason referenced by at least one Lead, including archived ones, cannot be deleted. The reference from Lead to Lost Reason is restrictive.

### LEAD-016 Recurring plan month count is non-negative

A check constraint enforces `number_of_months >= 0` on Recurring Revenue Plan. Violation message:

> "The number of month can't be negative."

### LEAD-017 Tag names are unique

A uniqueness constraint on the `name` of Sales Tag. Violation message:

> "Tag name already exists!"

### LEAD-018 Recurring revenue is only kept for users of the recurring revenue group

When a Lead is duplicated by a user who does not belong to the access group **Show Recurring Revenues Menu**, the copy receives `recurring_revenue = 0` and an empty `recurring_plan`.

### LEAD-019 Properties follow the team

`lead_properties` holds values whose definition lives on `lead_properties_definition` of the Sales Team. Changing the team of a Lead therefore changes which properties exist on it: properties whose definition does not exist on the new team are dropped.

## 2. Contact data, synchronization and communication quality

### LEAD-020 The address block travels as a whole

When the customer of a Lead changes, the six address fields `street`, `street2`, `city`, `zip`, `state_id`, `country_id` are rewritten together or not at all. If the customer has at least one non-empty address field, all six are copied from the customer, including the empty ones. If the customer has no address field filled, all six keep the values they had on the Lead. A mixed address made of parts coming from two sources is never produced.

### LEAD-021 Contact name derivation

When the customer changes: if there is no customer, `contact_name` is cleared. Otherwise the candidate is the customer's own name when the customer is not a company, and nothing when the customer is a company; if the candidate is empty, the previous value of `contact_name` is kept.

### LEAD-022 Company name derivation

When the customer changes: if there is no customer, `partner_name` is cleared. Otherwise the candidate is, in this order, the name of the customer's parent, then the customer's own name when the customer is a company, then the customer's stored company name; if the candidate is empty, the previous value of `partner_name` is kept.

### LEAD-023 Job position and web address derivation

When the customer changes, `function` and `website` are overwritten by the customer's values when the Lead value is empty **or** the customer has a value. A non-empty Lead value survives only when the customer has none.

### LEAD-024 Language derivation

When the customer changes, `lang_id` is set to the language matching the customer's language code, for every Lead that has a customer. When the customer has no language, the Lead language is cleared. This is deliberately unconditional so that the customer's language always wins.

### LEAD-025 When the email of the Lead is written onto the customer

The Lead writes its email onto the customer only when all of the following hold, evaluated with "void values are not propagated":

1. `partner_id` (contact) is set;
2. **and** `email_from` (electronic mail address) is not empty;
3. **and** the raw value of `email_from` differs from the raw address of the Contact;
4. **and** the normalised form of `email_from` differs from the normalised form of the Contact's
   address.

`normalized(x)` is the normalized bare address of `x`, or `x` itself when it cannot be normalized, or empty when `x` is empty. A change that is only a change of display form (`Robert <robert@example.com>` versus `robert@example.com`) therefore does not write on the customer.

### LEAD-026 When the telephone number of the Lead is written onto the customer

Identical to `LEAD-025` with `phone` and the international telephone format instead of the normalized address:

1. `partner_id` is set;
2. **and** `phone` (telephone) is not empty;
3. **and** the raw value of `phone` differs from the raw telephone of the Contact;
4. **and** the international form of `phone` differs from the international form of the Contact's
   telephone.

### LEAD-027 When the email or the telephone of the customer is copied onto the Lead

The reverse direction uses the same two tests but with "void values are propagated": `email_from` is overwritten by `partner.email` when the customer has an email and the test of `LEAD-025` passes with the void guard removed; the same applies to `phone` with `LEAD-026`.

### LEAD-028 Warning flags before saving

`partner_email_update` and `partner_phone_update` evaluate `LEAD-025` and `LEAD-026` with the void guard active, so that the interface can warn the user that saving will also modify the customer record.

### LEAD-029 A portal user never overwrites the email of a customer that has a salesperson

When the acting user is a portal user and the customer of the Lead already has a salesperson, the email propagation of `LEAD-025` is suppressed. This protects customer records owned by the internal organization from edits made through the reseller portal.

### LEAD-030 Telephone quality

`phone_state` is derived from `phone` and the country code of `country_id`: empty when `phone` is empty; `correct` when the number can be parsed with that country code; `incorrect` when parsing raises. When `country_id` is empty, the parse runs without a country hint, which succeeds only for numbers written in the international form.

### LEAD-031 Email quality

`email_state` is derived from `email_from`: empty when `email_from` is empty; `correct` when at least one of the addresses contained in the field passes address validation; `incorrect` when none does. A field holding several addresses is therefore `correct` as soon as one of them is valid.

### LEAD-032 Telephone reformatting on change

When the user edits `phone`, `country_id` or `company_id` in a form, the telephone number is rewritten in the international format if it can be parsed with the country of the record; otherwise it is left exactly as typed.

### LEAD-033 Email domain criterion

`email_domain_criterion` is derived from `email_normalized`:

1. When `email_normalized` is empty, the criterion is empty.
2. When the normalised address contains no at sign, the criterion is the whole normalised address.
3. Otherwise the domain is the part of the normalised address after the last at sign.
4. When that domain is a generic electronic mail provider domain, the criterion is the whole
   normalised address.
5. Otherwise the criterion is an at sign followed by the domain.

The list of generic provider domains is maintained by the platform. A generic domain yields the whole address so that two unrelated private individuals sharing a public provider are never grouped.

### LEAD-034 Blacklists

The email blacklist and the telephone blacklist of the messaging domain apply to Leads. `is_blacklisted` is true when the normalized email is on the email blacklist; `phone_blacklisted` is true when the sanitized telephone number is on the telephone blacklist; `partner_is_blacklisted` reflects the customer's flag. A blacklisted address or number is excluded from mass mailing and from mass text messaging. See [../messaging-and-activities/business-rules.md](../messaging-and-activities/business-rules.md).

### LEAD-035 Customer visibility on a lead form

The customer field is shown on a record of type `lead` only when a customer is already set or when the reader belongs to the technical group. On a record of type `opportunity` it is always shown. This is a presentation rule only; it never blocks a write.

### LEAD-036 Choosing a commercial entity that contradicts the customer clears the customer

In a form, when the user sets `commercial_partner_id` to a company that is not the commercial entity of the currently selected customer, the system clears `partner_id`, `email_from` and `phone` and restores the chosen company into `commercial_partner_id`, so that the user can pick a contact under the new company. When `name` is still empty, it becomes `<company name>'s opportunity`.

### LEAD-037 Commercial entity derivation

`commercial_partner_id` is derived and not stored. For a Lead with a customer, it is the commercial entity of that customer when this entity is a company and is not the customer itself, and empty otherwise. For a Lead with no customer but a `partner_name`, it is the first company contact whose name equals `partner_name`, or empty when no such company exists.

### LEAD-038 Posting a message that names a recipient can attach the customer

When a message is posted on a Lead that has an email and no customer, and the message explicitly names a recipient whose address equals the Lead address, that contact is written as the customer on **every** Lead that has no customer, matches that address (on the normalized address when available, otherwise on the raw address) and whose stage is not folded.

### LEAD-039 Replies are addressed to the team alias

Notification emails sent about a Lead carry a reply address equal to the alias of the team of the Lead. A Lead with no team falls back on the platform default reply address. Notification emails also carry the expected closing date as a subtitle when `date_deadline` is set, rendered as `Deadline: <date in the reader's date format>`.

## 3. Pipeline, probability, won, lost and dates

### LEAD-040 Won status definition

`won_status` is derived and never written directly:

| Value | Definition |
|---|---|
| `won` | `probability` is exactly one hundred **and** the stage carries `is_won` |
| `lost` | `active` is false **and** `probability` is exactly zero |
| `pending` | every other combination |

A probability of one hundred alone does not make a record won. Archiving alone does not make a record lost.

### LEAD-041 Writing a won stage forces the record open and won

A write that changes `stage_id` to a stage flagged as won also writes, in the same operation, `active = true`, `probability` equal to one hundred and `automated_probability = 100`. This applies to every write path, including the mass edit of a list and the drag of a card onto the won column.

### LEAD-042 A stage change between two won stages keeps the closing date

When a record already sits in a won stage and is written into another won stage, `date_closed` is not rewritten. Records that were not already in a won stage receive `date_closed = now`.

### LEAD-043 Closing date rules

On every write, in this order of evaluation:

| Condition on the values being written | Effect on `date_closed` |
|---|---|
| `probability` greater than or equal to one hundred or `active = false` | set to the current instant |
| otherwise, `probability` strictly above zero | cleared |
| otherwise, the stage actually changes, the new stage is not a won stage, and `probability` is not part of the write | cleared |
| otherwise | untouched |

At creation, a record that lands directly in a won stage and has no closing date receives `date_closed = now`.

### LEAD-044 Assignment date rules

| Condition on the values being written | Effect on `date_open` |
|---|---|
| `user_id` is written empty | cleared |
| `user_id` is written non-empty and at least one record in scope has a different salesperson | set to the current instant |
| otherwise | untouched |

Writing the same salesperson again does not move the date.

### LEAD-045 Last stage update date

`date_last_stage_update` is set to the current instant at creation and on every write of `stage_id` in which at least one record in scope actually changes stage. Writing the same stage does not move the date.

### LEAD-046 Days to assign and days to close are whole days

`day_open` is the absolute number of whole days between `create_date` and `date_open`, and is empty when either is missing. `day_close` is the absolute number of whole days between `create_date` and `date_closed`, and is empty when either is missing. Both use the whole-day difference of the two instants, not a rounded fraction.

### LEAD-047 Rotting

A Lead rots when all of the following hold: `won_status = "pending"`, `type = "opportunity"`, the stage defines a strictly positive rotting threshold in days, and the number of whole days since the last stage change (or since creation when there was none) is greater than that threshold. Changing the threshold on a stage does not retroactively recompute the rotting status of records whose last update predates the change.

### LEAD-048 Marking a record lost

The Mark Lost operation performs, in this order: archive the records (`active = false`), then write `probability` equal to zero, `automated_probability = 0` and the chosen lost reason. The closing date follows `LEAD-043`. A record whose stage is flagged as won is refused by `LEAD-006`.

### LEAD-049 Marking a record won

The Mark Won operation performs, in this order: unarchive the records, choose a won stage per record (see [calculations.md](calculations.md), section 3.4), then write `stage_id` and `probability` equal to one hundred grouped by target stage. A record with no reachable won stage keeps the whole set of won stages as the write target, which the storage layer resolves to the first one.

### LEAD-050 Restoring a lost record

Restore performs: unarchive, clear `lost_reason_id`, recompute `automated_probability`, then write `probability` set equal to `automated_probability`. The record becomes automatic again.

### LEAD-051 Unarchiving without restoring

Unarchive alone performs: `active = true`, clear `lost_reason_id`, recompute `automated_probability`. It does **not** realign `probability`, so a manually typed probability survives an unarchive.

### LEAD-052 Archiving a won record keeps it won

Archiving a record whose probability is one hundred and whose stage is a won stage leaves `won_status = "won"`, because lost requires a probability of zero. No frequency counter moves.

### LEAD-053 Probability is automatic while it equals the automated probability

`is_automated_probability` is true when `probability` and `automated_probability` are equal when compared to two decimal places. The scoring model overwrites `probability` only for records that are active and automatic at the moment of the recomputation; it always overwrites `automated_probability`.

### LEAD-054 Explicitly aligning the probability

The operation "use the automated value" recomputes `automated_probability` for the single record and then writes that value into `probability`, making the record automatic again.

### LEAD-055 Changing the won flag of a stage rewrites its records

Writing `is_won = true` on a Pipeline Stage writes `probability` equal to one hundred and `automated_probability = 100` on every Lead currently in that stage. Writing `is_won = false` recomputes `automated_probability` for those records and realigns `probability` for the records that were still automatic. A manual probability typed just before is lost in the second case.

### LEAD-056 Warning when toggling the won flag

Changing `is_won` on an existing Pipeline Stage returns a warning before saving:

- Title: `Do you really want to update this stage?`
- Body: `Changing the value of 'Is Won Stage' may induce a large number of operations, as the probabilities of opportunities in this stage will be recomputed on saving.`

The warning is not shown while creating a new stage.

### LEAD-057 Tracking subtypes

The subtype attached to a tracking message on a Lead is chosen in this order, the first match winning:

| Order | Condition on the change | Subtype |
|---|---|---|
| 1 | `stage_id` changed and the record is now won | Opportunity Won |
| 2 | `lost_reason_id` changed and is now set | Opportunity Lost |
| 3 | `stage_id` changed | Stage Changed |
| 4 | `won_status` changed and is not `lost` | Opportunity Restored |
| 5 | `won_status` changed and is `lost` | Opportunity Lost |
| 6 | none of the above | the platform default |

### LEAD-058 Creation message

The creation of a Lead posts a message with the subtype Opportunity Created whose body is `A new lead has been created for the team "<team name>".`, or `A new lead has been created and is not assigned to any team.` when the record has no team.

### LEAD-059 Deleting a Lead detaches its meetings

Deleting a Lead first clears the generic document reference of every Calendar Event that points at it, so that no meeting keeps a link to a record that no longer exists.

## 4. Duplicates, conversion and merging

### LEAD-060 Potential duplicate detection

Duplicates are detected, never prevented. The potential duplicates of a Lead are computed with elevated rights, across company visibility rules and including archived records, from three criteria combined by union:

1. Start from an empty result set. Every search below excludes the record itself.
2. When `email_domain_criterion` (electronic mail domain criterion) is not empty, search the Leads
   whose criterion is exactly equal, apply the discriminating test of step 5, and add the outcome to
   the result set.
3. When `partner_id` (contact) is set and that Contact has a commercial entity, search the Leads
   whose Contact is that commercial entity or one of its descendants, and add the outcome to the
   result set without applying the discriminating test.
4. When `phone_sanitized` (sanitised telephone) is not empty, search the Leads whose sanitised
   number is exactly equal, apply the discriminating test, and add the outcome to the result set.
5. **Discriminating test.** The search is run with a limit of twenty-one records. Its outcome is
   kept only when strictly fewer than twenty-one records came back; twenty-one or more means the
   criterion is not discriminating and the whole outcome is discarded.
6. `duplicate_lead_ids` (potential duplicates) is the result set plus the record itself;
   `duplicate_lead_count` (potential duplicate count) is the number of records in the result set,
   without the record itself.

A criterion that matches twenty-one records or more is judged not discriminating and contributes nothing. The commercial entity criterion has no such cap.

### LEAD-061 Duplicate search used by conversion, mass conversion and assignment

A second, simpler search is used when the system needs the records that should be merged with a given one:

**Inputs**: an optional Contact, an optional electronic mail address, and a flag "include lost".

1. When neither a Contact nor an address is supplied, the answer is the empty set.
2. Build the alternatives. When the supplied address yields at least one normalised address, the
   first alternative is "`email_normalized` (normalised electronic mail address) is one of the
   normalised addresses contained in the supplied value". When a Contact is supplied, the second
   alternative is "`partner_id` is that Contact".
3. When no alternative could be built, the answer is the empty set.
4. Combine the alternatives with a logical **or** to form the base condition.
5. Add the outcome restriction. With "include lost" set, the restriction is "`won_status` is not
   `won`, and either `type` is `opportunity` or `active` is true". With "include lost" clear, the
   restriction is "`won_status` is exactly `pending` and `active` is true".
6. Run the search with archived records included and return the outcome.

The conversion dialogue calls it with "include lost" set, so an archived lost opportunity is proposed for merging. The mass conversion dialogue and the assignment job call it with "include lost" clear.

### LEAD-062 Conversion is refused on a closed record

Opening the conversion dialogue on a Lead whose probability is exactly one hundred is refused:

> "Closed/Dead leads cannot be converted into opportunities."

### LEAD-063 Conversion skips archived and won records

The conversion operation walks the selected records and silently skips every record that is archived or whose won status is `won`. The remaining records receive `type = "opportunity"`, `date_conversion = now`, the chosen customer when it differs from the current one, and a stage computed from the target team when they had none.

### LEAD-064 Customer handling during conversion

| Chosen action | Effect per record |
|---|---|
| `create` | When the record already has a customer, nothing. Otherwise a new contact is created from the record data (see [entities.md](entities.md), section 8.8) and linked. |
| `exist` | The chosen contact is written on the record, replacing any existing one. |
| `each_exist_or_create` (mass mode only) | Per record: the contact matching the record email is looked up without creating one; when found it is written, otherwise a new contact is created from the record data. |

### LEAD-065 Salesperson allocation during conversion

After conversion, the chosen salespeople are distributed over the converted records round robin. When `force_assignment` is false, records that already have a salesperson are excluded from the distribution. The single conversion dialogue defaults `force_assignment` to true; the mass conversion dialogue defaults it to false.

### LEAD-066 Merge requires at least two records

> "Select at least two Leads/Opportunities from the list to merge them."

### LEAD-067 Manual merge is limited to five records

A merge started by a user is refused above five records:

> "To prevent data loss, Leads and Opportunities can only be merged by groups of 5."

The limit is expressed as a parameter in the message, so a deployment that changes the limit changes the number shown. The automatic assignment job merges without any limit, because it runs with system rights.

### LEAD-068 Survivor selection

The records to merge are ordered by decreasing confidence and the first one survives. The ordering key, compared in this order, is:

1. the record is an opportunity **or** is active (an archived lead ranks last; an archived opportunity still ranks as valid);
2. the record is an opportunity;
3. the sequence of its stage, higher first;
4. its probability, higher first;
5. its identifier, higher first (a newer record is considered more reliable).

### LEAD-069 Merged type and priority

The merged record is an opportunity when at least one of the merged records is an opportunity, otherwise a lead. The merged priority is the highest priority among the records that have one, and empty when none has one.

### LEAD-070 Merged description

The non-empty descriptions of the merged records are concatenated in confidence order, separated by a blank line. A description that contains only empty rich text markup counts as empty and is skipped.

### LEAD-071 Merged address

The address block is taken as a whole from the record with the greatest number of non-empty address fields among `street`, `street2`, `city`, `zip`, `state_id`, `country_id`. Ties are broken in favour of the record that comes first in the confidence order.

### LEAD-072 Merged lost reason

The merged lost reason is empty when the survivor has a strictly positive probability. Otherwise it is the first non-empty lost reason in confidence order.

### LEAD-073 Merged tags

The merged tag set is the union of the tags of every merged record.

### LEAD-074 Every other merged scalar takes the first non-empty value

The merged field list, in the order in which it is built, is: `campaign_id`, `medium_id`, `source_id`, `email_cc`, `name`, `user_id`, `color`, `company_id`, `lang_id`, `team_id`, `referred`, `stage_id`, `expected_revenue`, `recurring_plan`, `recurring_revenue`, `create_date`, `date_automation_last`, `date_deadline`, `partner_id`, `partner_name`, `contact_name`, `email_from`, `function`, `phone`, `website`, plus the fields handled specially by `LEAD-069` to `LEAD-073`, plus the six address fields of `LEAD-071`, plus every field added by an installed bridge. For each of the plain fields above, the merged value is the first non-empty value in confidence order.

A name in the merged field list that does not correspond to a field of the Lead is silently skipped, which lets a bridge add a name to the list without checking whether the matching capability package is installed.

### LEAD-075 Merged flags and collections added by bridges

| Field | Merge rule |
|---|---|
| Orders (sales bridge) | The orders of every merged record are attached to the survivor. |
| Website visitors (website bridge) | The union of the visitors of every merged record. |
| Enrichment done flag (enrichment bridge) | True when at least one merged record had it true. |
| Event, registrations, external company identifier, mining request, website identification fields, reseller fields (latitude, longitude, assigned partner, forwarding date) | First non-empty value in confidence order. |
| Every other many-to-many and one-to-many field | Not merged. |

### LEAD-076 Probability is not merged

The survivor keeps its own `probability` and its own `automated_probability`, and therefore keeps its automatic or manual nature. The probability is not part of the merged field list.

### LEAD-077 Forced salesperson and team during a merge

When the caller supplies a salesperson or a team, that value replaces the merged one. A value equal to what the survivor already carries is removed from the write, so that no recomputation is triggered for nothing.

### LEAD-078 Stage repair after a forced team

When a team is forced and the merged stage is not available to that team, the merged stage becomes the first stage available to that team ordered by sequence then identifier, or empty when the team has none available.

### LEAD-079 History, activities, attachments and meetings move to the survivor

Every message of every merged-away record is re-pointed at the survivor and its subject becomes `From *the record's title*: <original subject>`, or `From *the record's title*` when the message had no subject. Every activity is re-pointed at the survivor. Every attachment is re-pointed and renamed `<attachment name> (from <first twenty characters of the record name>)`. Every meeting is re-pointed both through its opportunity link and through its generic document reference.

### LEAD-080 Followers move only when recently active

A follower of a merged-away record is added to the survivor only when the related contact posted a message on that record within the last thirty days and is not already a follower of the survivor. Each contact is moved at most once, even when it followed several merged-away records.

### LEAD-081 Merge summary note

Before the history is moved, an internal note is posted on the survivor listing, for every merged-away record, its main fields, its properties formatted for reading, and the followers that were carried over.

### LEAD-082 Merged-away records are deleted with elevated rights

The merged-away records are deleted with system rights, because the acting user had the right to read them and the deletion is a consequence of an operation they were allowed to perform. A caller may ask to keep them (the conversion dialogue does, and deletes them itself once it has re-pointed itself at the survivor).

### LEAD-083 Deduplication during mass conversion

When the mass conversion dialogue has deduplication enabled, each selected record is, before conversion, merged with the records returned by `LEAD-061` for its customer and its email, with lost records excluded, whenever that search returns strictly more than one record. Records already consumed by an earlier merge are skipped. The final list to convert keeps the original selection order, with the merge survivors appended at the end when they were not part of the selection.

## 5. Teams, memberships and assignment

### LEAD-084 A team assignment condition must be a valid search condition

The assignment condition of a Sales Team must parse and must be usable as a search condition on Lead. Violation message:

> "Assignment domain for team <team name> is incorrectly formatted"

### LEAD-085 A member assignment condition must be a valid search condition

> "Member assignment domain for user <user name> and team <team name> is incorrectly formatted"

### LEAD-086 A member preference condition must be a valid search condition

> "Member preferred assignment domain for user <user name> and team <team name> is incorrectly formatted"

### LEAD-087 Membership uniqueness in single-membership mode

When multi-team membership is disabled, the pair (team, user) must be unique among **active** memberships. Archived duplicates are tolerated, which is why no database uniqueness constraint is used. Violation message:

> "You are trying to create duplicate membership(s). We found that <user name> (<team name>), ... already exist(s)."

### LEAD-088 A member must be allowed in the company of the team

When the team has a company, every member must have that company among its allowed companies. Violation message on the membership:

> "User '<user name>' is not allowed in the company '<company name>' of the Sales Team '<team name>'."

Violation message when the check is run from the team after a company change:

> "The following team members are not allowed in company '<company name>' of the Sales Team '<team name>': <user names>"

### LEAD-089 Single-membership mode archives the other memberships

In single-membership mode, creating a membership, or activating an archived one, archives every other **active** membership of the same user in a different team. Creating a membership that already exists as archived leaves the archived one alone and creates a new one.

### LEAD-090 Archiving a user archives its memberships

Archiving a user archives every Sales Team Member record of that user.

### LEAD-091 The main team of a user is the oldest active membership

`sale_team_id` on a user is the team of the oldest active membership, memberships being ordered by creation instant ascending then identifier.

### LEAD-092 Writing the member list of a team

Writing the member list of a Sales Team creates a membership for every user newly present, activates the existing membership of every user present, and archives the membership of every user absent. Users written into the member list are also added to the favourite users of the team.

### LEAD-093 Two shipped teams cannot be deleted

> "Cannot delete default team "<team name>""

applies to the shipped website team and the shipped point of sale team.

### LEAD-094 A team actively used by sales cannot be deleted

A Sales Team whose count of active Sales Orders is **five or more** cannot be deleted:

> "Team <team name> has <count> active sale orders. Consider cancelling them or archiving the team instead."

### LEAD-095 Deleting a team folds its scoring statistics into the cross-team statistics

See [calculations.md](calculations.md), section 14, for the exact arithmetic. The team's own frequency rows are removed by cascade.

### LEAD-096 Clearing both usage flags clears the alias

In a form, clearing both `use_leads` and `use_opportunities` clears the alias name of the team, because the team no longer accepts incoming requests.

### LEAD-097 Writing a usage flag rewrites the alias

Writing `use_leads` or `use_opportunities` rewrites the alias of the team: the alias name is cleared when both flags are false; the alias target is the Lead entity; the alias defaults become `type` (equal to `lead` when the writing user belongs to the access group **Show Lead Menu** and the team uses leads, otherwise `opportunity`) and `team_id` (this team).

### LEAD-098 Automatic assignment requires an administrator

Running the assignment, manually or from the scheduled job, is refused for a user who is neither a sales administrator nor the system:

> "Lead/Opportunities automatic assignment is limited to managers or administrators"

### LEAD-099 Which leads the team allocation considers

A Lead is a candidate for allocation to a team when all of the following hold: it has no team and no salesperson; its won status is not `won`; its creation instant is at most the current instant minus the configured assignment delay in hours; when a creation window is in force, its creation instant is strictly after the current instant minus the window in days; and it matches the assignment condition of the team. Teams with a capacity of zero are not considered at all.

### LEAD-100 Which leads the member distribution considers

A Lead is a candidate for distribution to a member of a team when it has no salesperson, no assignment date, and belongs to that team. Note that the absence of an assignment date is part of the condition: a record that once had a salesperson and lost it is not redistributed.

### LEAD-101 Member eligibility

A member is eligible when it is not paused (`assignment_optout = false`) and its remaining quota is strictly positive. Archived memberships never receive leads. A capacity of zero yields a quota of zero and therefore excludes the member.

### LEAD-102 A member only receives leads its condition accepts

A lead is given to the first eligible member, in the current order of the member list, whose assignment condition accepts it. When the member also has a preference condition, the leads matching both conditions are distributed first, in a separate pass.

### LEAD-103 Every assigned lead becomes an opportunity

The member distribution converts each lead it assigns into an opportunity, keeping the customer the record already had, and suppressing the automatic subscription notifications.

### LEAD-104 Deduplication during allocation

Before a lead is attached to a team, the duplicate search of `LEAD-061` runs on its email. When it returns more than one record, the whole group is merged without a size limit and without forcing a salesperson or a team, and the survivor is the record attached to the team. The team is written on the candidate record **before** the merge, so that a survivor elected among the duplicates that already had a team keeps its own team and salesperson.

### LEAD-105 Manual assignment forces the quota and ignores the creation window

A manual run uses the full daily quota of every member regardless of how many leads they already received in the last twenty-four hours, and considers leads of any age. A scheduled run subtracts the leads of the last twenty-four hours from the quota and, by default, only considers leads created in the last seven days.

### LEAD-106 Assignment from the settings screen skips opted-out teams

The assignment button of the settings screen runs the manual assignment over every team whose `assignment_optout` is false.

### LEAD-107 Leads created without a salesperson are given to the team leader when rule-based assignment is off

When rule-based assignment is not activated, a Lead created by the incoming mail gateway or by a live chat step and having no salesperson receives the leader of its team, and a note is logged on it: `This new lead created by <creation source> was automatically assigned to team leader <user name>`, where the creation source is `incoming email` or `livechat discussion`.

## 6. Predictive lead scoring

### LEAD-108 Only an administrator may change the scoring configuration

Confirming the probability update dialogue does nothing at all for a user who is not an administrator: no parameter is written and no rebuild runs. No message is shown.

### LEAD-109 The scoring start date must be a real date

The stored start date is read as a text value. When it cannot be parsed as a date, the scoring model treats the configuration as absent and performs no computation and no frequency update. The settings screen displays, in that case, the date eight days before today.

### LEAD-110 Only fields that exist on the Lead entity are used as scoring variables

The stored list of scoring variables is a comma-separated list of field names. Names that do not correspond to a field of the Lead entity are ignored.

### LEAD-111 The stage is always a scoring variable

`stage_id` and `team_id` are always part of the values read for a record, in addition to the configured variables. `stage_id` always participates in the probability computation; `team_id` selects which statistics are used and is not itself a scored variable.

### LEAD-112 Rebuilding the frequency table requires deletion rights on the Lead entity

The full rebuild first checks that the acting identity may delete Leads; otherwise:

> "You don't have the access needed to run this cron."

### LEAD-113 Changing the scoring variable list reloads the Lead entity

Creating, writing or deleting the system parameter that holds the scoring variable list forces the Lead entity to be reloaded, because the set of fields the probability derivation depends on is dynamic.

### LEAD-114 A record with no stage has a probability of zero

The scoring model returns zero for a record whose stage is empty and performs no further computation for it.

### LEAD-115 A record in a won stage has a probability of one hundred

The scoring model returns one hundred for a record whose stage is flagged as won, without further computation.

### LEAD-116 A pending record is always strictly between zero and one hundred

The computed probability of a pending record is clamped to the range `[0.01, 99.99]` after rounding to two decimal places. Exactly zero and exactly one hundred are reserved for lost and won.

### LEAD-117 Statistics are never negative

A frequency counter that a decrement would take to zero or below is stored as `0.1`.

## 7. Acquisition channels

### LEAD-118 Website contact form

A lead created from the website contact form receives: the medium submitted, else the default medium of the entity, else the medium named `website` (created when missing); the team submitted, else the default team of the website; the salesperson submitted, else the default salesperson of the website; and a type equal to `lead` when the resolved team uses leads and `opportunity` when it does not, falling back, when there is no team, on the reader's membership of the access group **Show Lead Menu**. When the form imposes a customer, the submitted email and telephone are dropped from the values, because they would be resynchronized from the customer.

### LEAD-119 Live chat lead creation and channel access

A lead created from a live chat conversation stores the conversation in `origin_channel_id`. A sales user may read a conversation that produced a lead, and may read and create its membership records, but may not write, create or delete the conversation itself. Creating or writing a Lead that points at a conversation the acting user may not read is refused:

> "You cannot create leads linked to channels you don't have access to."

> "You cannot update a lead and link it to a channel you don't have access to."

### LEAD-120 Event lead rules do not run during data import

Rules attached to event registrations do not run when registrations are created or written as part of a data import.

### LEAD-121 One lead per registration and per rule

A registration that already produced a lead for a given rule never produces a second one for that rule, even when the triggering status changes again. Archived leads count for this exclusion.

### LEAD-122 One generation request per event

> "You can only have one generation request per event at a time."

### LEAD-123 Only event managers may regenerate the leads of an event

> "Only Event Managers are allowed to re-generate all leads."

### LEAD-124 Confirming a sales order may raise the expected revenue

Confirming a Sales Order linked to an opportunity writes the untaxed amount of the order into `expected_revenue` of that opportunity when **both** of the following hold: the untaxed amount is strictly greater than the current expected revenue, and the currency of the order equals the company currency of the opportunity. The change is tracked with the log message `Expected revenue has been updated based on the linked Sales Orders.`

### LEAD-125 Lead mining errors

| Service answer | State after the run | `error_type` | Message |
|---|---|---|---|
| Not enough credits | `error` | `credits` | none; the state carries the information |
| Empty result | `draft` | `no_result` | none |
| Transport failure | unchanged | unchanged | `Your request could not be executed: <reason>` |
| Data returned | `done` | cleared | none |

### LEAD-126 Enrichment eligibility

The manual enrichment button is offered only when the record is active, has an email whose quality is not `incorrect`, has not been enriched yet, did not come from the website identification service and does not have a probability of one hundred. The scheduled enrichment selects records that have not been enriched, whose probability is below one hundred or empty, that have an email, that did not come from the website identification service, that were created within the last twenty-four hours and that are active.

### LEAD-127 Enrichment marks the record even when nothing is found

`iap_enrich_done` is set to true for every processed record, whatever the outcome, so that the same record is never charged twice.

### LEAD-128 Enrichment only fills empty fields

The values returned by the enrichment service are written only where the Lead field is empty. `partner_name`, `reveal_id`, `street`, `city`, `zip`, `phone`, `country_id` and `state_id` are candidates; none of them is ever overwritten.

### LEAD-129 Enrichment notifications

| Outcome | Message shown to the acting user |
|---|---|
| Success | `The leads/opportunities have successfully been enriched` |
| Not enough credits | `Not enough credits for Lead Enrichment` |
| Any other failure | `An error occurred during lead enrichment` |

Notifications are suppressed when the run comes from the scheduled job.

### LEAD-130 Lead Generation Rule pattern must compile

> "Enter Valid Regex."

### LEAD-131 Lead Generation Rule contact count bounds

A check constraint enforces `extra_contacts >= 1 AND extra_contacts <= 5`:

> "Maximum 5 contacts are allowed!"

### LEAD-132 Reveal View retention

A Reveal View whose address already produced a lead within the retention window is deleted before processing. Records of that entity older than one month are deleted by the cleanup.

### LEAD-133 Mail plugin lead creation requires a known contact

When the contact identifier passed by the mail plugin does not resolve, the answer is the error code `partner_not_found` and no record is created.

### LEAD-134 Survey leads are always opportunities

A lead created from a survey participation is created with `type = "opportunity"`, on the assumption that the answers already qualify the interest.

## 8. Resellers, partnerships and the partner portal

### LEAD-135 Lead visibility

| Access group | Visible Leads |
|---|---|
| Salesperson (own documents only) | `user_id` is the reading user or is empty |
| Salesperson (all documents) | every Lead |
| Portal user | `partner_assigned_id` is the reader's commercial entity or one of its descendants, read only |

In addition, every reader is restricted by the company rule `company_id` is among the reader's allowed companies or is empty.

### LEAD-136 A portal user may only write on a lead assigned to its own family

> "Only users with commercial partner which is a parent of the assigned partner can edit this lead."

### LEAD-137 A portal user may only point a link field at a record it may read

Every many-to-one value written by a portal user is checked for read access on the target record before the write proceeds.

### LEAD-138 A portal user may only update a closed list of contact fields

The portal operation that updates the contact details accepts only `partner_name`, `phone`, `email_from`, `street`, `street2`, `city`, `zip`, `state_id` and `country_id`. Any other field is refused:

> "Not allowed to update the following field(s): <field names>."

### LEAD-139 Creating an opportunity from the portal requires a grade

The portal operation that creates an opportunity is refused when neither the acting user's contact nor its commercial entity carries a grade. The refusal is an access denial with no message body. When the grade is present but a field is missing, the answer is:

> "All fields are required!"

### LEAD-140 Geographic assignment needs a country

The partner assignment operation skips every Lead with no country and warns:

> "There is no country set in addresses for <lead names>."

### LEAD-141 Only weighted partners receive forwarded leads

Every candidate search is restricted to contacts with `partner_weight > 0`. A weight of zero means the partner never receives a forwarded lead.

### LEAD-142 A partner that declined a lead is never proposed again for it

Every candidate search excludes the contacts listed in `partner_declined_ids` of the Lead.

### LEAD-143 No partner found

When no candidate is found at all, the shipped tag `No more partner available` is added to the Lead and no partner is assigned.

### LEAD-144 Forwarding requires an email address on the recipient

| Mode | Message when a recipient has no email address |
|---|---|
| Automatic | `Set an email address for the partner(s): <names>` |
| Single | `Set an email address for the partner <name>` |

A missing forwarding template is reported as:

> "The Forward Email Template is not in the database"

### LEAD-145 Declining a lead unsubscribes the whole partner family

Declining removes every contact of the declining partner's commercial family from the followers, clears `partner_assigned_id`, and adds every contact of that family to `partner_declined_ids`. When the refusal is marked as spam, the shipped tag `Spam` is added.

### LEAD-146 Accepting a lead converts it

Accepting posts `I am interested by this lead.` followed by the optional comment, then converts the record into an opportunity keeping its current customer.

### LEAD-147 Declining messages

| Contacted flag | Message posted |
|---|---|
| true | `I am not interested by this lead. I contacted the lead.` |
| false | `I am not interested by this lead. I have not contacted the lead.` |

The optional comment is appended as a separate paragraph.

### LEAD-148 A grade with a default pricelist writes that pricelist on the contact

Writing a grade whose default pricelist is set also writes that pricelist on the contact. Writing a different pricelist in the same operation is refused:

> "You are trying to assign two different pricelists (one directly and one from grade (<grade name>))."

### LEAD-149 A sales order cannot grant two different grades

> "You cannot confirm Sale Order <order name> because there are products assigning different grades."

### LEAD-150 Forwarding a lead assigns the salesperson of the recipient

When the chosen partner has a salesperson, that user becomes the salesperson of the forwarded Lead. The rule only applies to records that are active and whose probability is below one hundred.

### LEAD-151 Opening a forwarded lead as an external user goes to the portal

A shared user, or any user when the caller asks for the public form, is redirected to the portal page of the record instead of the internal form, provided the user may read the record.

### LEAD-152 Publishing a reseller page requires write access to the contact

The control that publishes or unpublishes the public page of a reseller is offered only to a reader who may write on the Contact record behind that page. Write access on Contact is granted by the salesperson group and by the contact manager group (`LEAD-155`); website editing rights alone do not grant it. A reader who may not write on the contact sees the page without the publication control and cannot change the publication flag by any other route, because the write is refused by the access rules of the contact.

### LEAD-153 The directory falls back to all countries

When the public directory is requested without an explicit country and without the explicit "all countries" choice, the country of the visitor is inferred from the network address of the request. When the inferred country holds no listable reseller while other countries do, the country filter is dropped for that request: the directory lists the resellers of every country and the country selector shows **All Countries** as the active entry. This prevents a visitor located in a country with no reseller from seeing an empty directory.

### LEAD-154 Grade publication filters the directory

A reseller is listable in the public directory only when it is a company contact, carries a grade, is published, and its grade is not archived. For a reader who is not a website editor the grade must be published as well; a website editor also sees the resellers of an unpublished grade, so that a grade can be prepared before it is announced. The same condition governs the detail page of one reseller: an unpublished reseller page answers only to a website editor.

## 9. Access, concurrency and consistency

### LEAD-155 Access rights per entity

| Entity | Access group | Read | Create | Update | Delete |
|---|---|---|---|---|---|
| Lead | Salesperson (own documents only) | yes | yes | yes | no |
| Lead | Sales administrator | yes | yes | yes | yes |
| Lead | Portal user | yes | no | no | no |
| Pipeline Stage | Internal user | yes | no | no | no |
| Pipeline Stage | Sales administrator | yes | yes | yes | yes |
| Pipeline Stage | Portal user | yes | no | no | no |
| Sales Team | Internal user | yes | no | no | no |
| Sales Team | Salesperson (own documents only) | yes | no | no | no |
| Sales Team | Sales administrator | yes | yes | yes | yes |
| Sales Team Member | Internal user | yes | no | no | no |
| Sales Team Member | Sales administrator | yes | yes | yes | yes |
| Sales Tag | Salesperson (own documents only) | yes | yes | yes | no |
| Sales Tag | Sales administrator | yes | yes | yes | yes |
| Lost Reason | Internal user | yes | no | no | no |
| Lost Reason | Salesperson (own documents only) | yes | no | no | no |
| Lost Reason | Sales administrator | yes | yes | yes | yes |
| Recurring Revenue Plan | Salesperson (own documents only) | yes | no | no | no |
| Recurring Revenue Plan | Sales administrator | yes | yes | yes | yes |
| Lead Scoring Frequency and Lead Scoring Frequency Field | Salesperson (own documents only) | yes | no | no | no |
| Lead Scoring Frequency and Lead Scoring Frequency Field | System administrator | yes | no | no | no |
| Activity Analysis Report | Salesperson (own documents only) | yes | no | no | no |
| Predictive Lead Scoring Update Wizard | Platform administrator | yes | yes | yes | yes |
| Lead Lost Wizard, Conversion wizards, Merge wizard, Forwarding wizards | Salesperson (own documents only) | yes | yes | yes | no |
| Event Lead Rules | Event registration desk, Event user, Salesperson | yes | no | no | no |
| Event Lead Rules | Event manager | yes | yes | yes | yes |
| Event Lead Request | System administrator | yes | yes | yes | yes |
| Lead Mining entities | Sales administrator | yes | yes | yes | yes |
| Lead Generation Rule and Reveal View | Salesperson (own documents only) | yes | no | no | no |
| Lead Generation Rule and Reveal View | Sales administrator | yes | yes | yes | yes |
| Partner Grade | Internal user, Portal user, Public user, Accounting reader, Invoicing user | yes | no | no | no |
| Partner Grade | Salesperson (own documents only) | yes | yes | yes | no |
| Partner Grade | Sales administrator, System administrator | yes | yes | yes | yes |
| Partner Activation | Internal user | yes | no | no | no |
| Partner Activation | Contact manager | yes | yes | yes | yes |
| Partner Assignment Analysis | Salesperson (own documents only) | yes | no | no | no |

The domain also grants access to entities owned by other domains, because a salesperson cannot work a pipeline without them. These grants are part of this domain and a replacement must reproduce them:

| Entity | Owning domain | Access group | Read | Create | Update | Delete |
|---|---|---|---|---|---|---|
| Contact | Contacts and Organizations | Salesperson (own documents only) | yes | yes | yes | no |
| Contact | Contacts and Organizations | Sales administrator | yes | no | no | no |
| Contact Tag | Contacts and Organizations | Salesperson (own documents only) | yes | yes | yes | no |
| Contact Tag | Contacts and Organizations | Sales administrator | yes | no | no | no |
| Calendar Event | Calendar and Scheduling | Salesperson (own documents only) | yes | yes | yes | no |
| Calendar Event | Calendar and Scheduling | Sales administrator | yes | yes | yes | yes |
| Calendar Event Tag | Calendar and Scheduling | Internal user | yes | no | no | no |
| Calendar Event Tag | Calendar and Scheduling | Salesperson (own documents only) | yes | no | no | no |
| Calendar Event Tag | Calendar and Scheduling | Sales administrator | yes | yes | yes | no |
| Activity Type | Messaging, Activities and Collaboration | Sales administrator | yes | yes | yes | yes |
| Activity Plan | Messaging, Activities and Collaboration | Sales administrator | yes | yes | yes | yes |
| Activity Plan Template | Messaging, Activities and Collaboration | Sales administrator | yes | yes | yes | yes |
| Text Message Template | Messaging, Activities and Collaboration | Sales administrator | yes | yes | yes | yes |
| Website Visitor | Website and Content Management | Salesperson (own documents only) | yes | no | no | no |
| Website Visit Track | Website and Content Management | Salesperson (own documents only) | yes | no | no | no |

The two Contact rows are not a mistake: the sales administrator is granted reading only by this domain, and receives writing and creation through the salesperson group it contains. The record-visibility conditions that narrow these grants further (the Activity Plan, Activity Plan Template and Text Message Template records restricted to the Lead and Contact models) are listed in [configuration.md](configuration.md) section 6.

No access group may delete a Lead except the sales administrator. Deleting is deliberately restricted because a lost record is archived, not deleted.

### LEAD-156 The scoring frequency table is written with elevated rights

Frequency rows are created and updated with system rights, because they are cross-company statistics that a salesperson may read but must never write.

### LEAD-157 Duplicate counting crosses company boundaries

The potential duplicate computation runs with elevated rights and includes archived records, because a high duplicate count is a signal to escalate to a manager and must not be hidden by the company rule.

### LEAD-158 Long-running jobs commit in batches

The assignment job, the scoring recomputation and the enrichment job commit their work in batches so that a failure late in the run does not discard the work already done, and so that no single transaction holds a write lock on the Lead table for a long time. The batch sizes are configurable; their defaults are in [configuration.md](configuration.md).

### LEAD-159 The enrichment job locks each batch

Each enrichment batch is locked before processing. When the lock cannot be taken, the job reschedules itself five minutes later instead of processing the batch. Each successful batch is committed on its own, so that consumed external credits are never rolled back.

### LEAD-160 A concurrent update during the scoring recomputation is retried, not lost

The scoring recomputation writes in batches of records grouped by identical probability. A batch whose write fails is logged and skipped; the remaining batches proceed. The next run recomputes the skipped records.

### LEAD-161 Rounding of monetary derivations

`prorated_revenue` is rounded to two decimal places. The three recurring derivations (`recurring_revenue_monthly`, `recurring_revenue_monthly_prorated`, `recurring_revenue_prorated`) are not rounded by the derivation itself; they are stored with the precision of the currency of the record and rounded by the storage layer accordingly.

### LEAD-162 Rounding of the probability

The probability produced by the scoring model is rounded to two decimal places with half-away-from-zero rounding before being clamped by `LEAD-116`. The comparison that decides whether a probability is automatic uses two decimal places as well.

### LEAD-163 Rounding of the daily quota

The daily quota of a member is the monthly capacity divided by thirty, rounded to a whole number with half-away-from-zero rounding. See [calculations.md](calculations.md).

### LEAD-164 Rounding when folding team statistics

When a team is deleted, the counters it contributes are rounded to whole numbers with half-away-from-zero rounding before being added to the cross-team counters. A contribution of `0.1` therefore adds nothing.

### LEAD-165 Dates are stored in coordinated universal time

Every datetime field of this domain is stored in coordinated universal time and displayed in the reader's time zone. `date_deadline` and `date_partner_assign` are plain dates with no time zone.

### LEAD-166 The celebration message uses a time zone-aware midnight

The day boundaries used to choose a celebration message are computed from midnight in the time zone of the reader, falling back on the time zone of the salesperson, falling back on coordinated universal time.

### LEAD-167 Ordering by the reader's own activity deadline

When a list of Leads is ordered by the reader's own activity deadline, the result is built in two passes: first the records on which the reader has at least one open activity, ordered by their earliest deadline (ascending or descending as requested) and filtered by the requested condition; then, if the requested window is not yet full, the remaining records in the normal ordering, excluding the records already returned. Counting is not affected by this rule.

### LEAD-168 Default ordering of a Lead list

`priority` descending, then `id` descending. A partial index supports this ordering for non-archived records.

### LEAD-169 Kanban column expansion

When Leads are grouped by stage, the displayed columns are the stages present in the grouping, plus every stage with no team, plus the stages of the team imposed by the context when there is one, plus the stages of every team of the reading user when the screen asks for them, ordered by the stage ordering.

## 10. Suggested recipients, text messaging and stage timing

### LEAD-170 Suggested recipients of a Lead

When a message is composed on a Lead, the platform proposes recipients. The Lead contributes, in this order:

1. its customer, when one is set;
2. every address extracted from `email_from`, in the order in which they appear in the field, the first one being the candidate customer of the record;
3. every address extracted from `email_cc`;
4. the salesperson of the record, unless that salesperson is the acting user.

A proposal that matches an existing contact carries that contact and no creation values. A proposal that matches no contact carries the creation values of `LEAD-171`, except for the addresses beyond the first one of `email_from` and for the carbon copy addresses, which carry no creation values because they are not the candidate customer of the record. Followers of the record are never proposed again, the platform's own aliases are never proposed, and the system identity is never proposed. A contact with no address is still proposed, with an empty address.

### LEAD-171 Creation values carried by a suggested recipient

The values that would create the contact behind a suggested recipient are the contact values of [entities.md](entities.md) section 8.7, with three differences:

1. the address itself is not repeated in the values, because it is the key of the proposal;
2. no parent company is ever created for the proposal, even when `partner_name` is set; when the Lead already has a commercial entity, that entity is set as the parent and the company name is dropped, and otherwise `partner_name` is carried as the company name;
3. the proposal is marked as a company only when the derived contact name is exactly equal to `partner_name`.

The derived contact name is `contact_name` when it is set, otherwise the display name carried by the address, otherwise the address itself. Empty values are not carried. The language of the Lead is carried only when that language is active; an archived language is dropped and the created contact then keeps the default customer language of the platform (see [entities.md](entities.md) section 8.7).

### LEAD-172 Text message target of a Lead

A text message composed on a Lead is sent to the sanitized international form of `phone`. A number on the telephone blacklist is skipped (`LEAD-034`). In batch mode the option that keeps a log is enabled by default, so that every record receives a note recording what was sent. Writing `phone` propagates to the customer under `LEAD-026` and refreshes the sanitized number, which in turn refreshes the potential duplicate count of `LEAD-060`.

### LEAD-173 Text message target of a website visitor

The text message composer opened from a website visitor targets the customer of the visitor when the visitor has one, taking the number from the telephone field of that contact. When the visitor has no customer but has at least one Lead whose telephone number equals the number of the visitor, the composer targets the most confident of those Leads, in the confidence order of `LEAD-068`, taking the number from the telephone field of the Lead. When neither applies, the composer is not offered.

### LEAD-174 Time spent in each stage

Every Lead exposes a map from stage to the whole number of seconds the record spent in that stage, rebuilt at every read from the recorded stage changes as described in [entities.md](entities.md) section 1.3.2. The map is never stored. Reading it for several records at once reads the recorded changes of the whole batch once, so the cost does not grow with the number of records read.

## 11. Catalogue, attribution and telephone-validation rules

### LEAD-175 A Recurring Revenue Plan is required as soon as a recurring revenue is entered

On a record of type `opportunity` edited by a user who holds the group **Show Recurring Revenues
Menu**, `recurring_plan` (recurring plan) becomes required as soon as `recurring_revenue` (recurring
revenue) is different from zero. The record cannot be saved until a plan is chosen.

### LEAD-176 A Stage that holds at least one Lead cannot be deleted

The reference from `stage_id` (stage) to the Stage entity is restrictive. The same applies to the
reference from a Stage to a Sales Team through `team_ids` (sales teams): a team that restricts at
least one stage cannot be deleted while that restriction exists.

### LEAD-177 A Sales Team always owns an alias

`alias_id` (alias) is **required** on a Sales Team and is created automatically with the team. The
reference is restrictive on delete and is not copied when a team is duplicated.

### LEAD-178 What a duplicated Lead does not carry over

Duplicating a Lead drops `stage_id`, `date_closed` (closed date), `date_open` (assignment date),
`probability`, `automated_probability`, `won_status`, `duplicate_lead_ids`, `duplicate_lead_count`,
`email_domain_criterion`, `email_normalized`, `phone_sanitized`, `date_last_stage_update` and the
whole discussion thread. It **does** carry over `lead_properties` (properties). The type and the
team of the source are re-imposed explicitly. The assignment date of the copy is the current instant
when the source is an opportunity whose salesperson is an active user, and empty otherwise; a
salesperson who is not an active user is dropped from the copy.

### LEAD-179 Campaign identifiers, source names and medium names are made unique

The identifier of a Campaign, the name of a Source and the name of a Medium are unique. Uniqueness
is achieved by appending a bracketed counter rather than by rejecting the input; the algorithm is in
[calculations.md](calculations.md), section 10. A Source additionally carries a stored uniqueness
rule whose message is "The name must be unique".

### LEAD-180 A Campaign needs both an identifier and a title

`name` (campaign identifier) and `title` (campaign name) are both required on a Campaign. A campaign
created with an identifier but no title takes the identifier as its title. The identifier is not
translatable, the title is. A Campaign also requires `user_id` (responsible) and `stage_id`
(campaign stage); the stage is not copied on duplication and its reference is restrictive.

### LEAD-181 Delivered attribution records that may not be deleted

| Record | Message |
|---|---|
| The delivered Source named "Referral" | "You cannot delete the 'Referral' UTM source record." |
| Each of the six delivered Media — Email, Direct, Website, X, Facebook and LinkedIn | "Oops, you can't delete the Medium '*the name*'. Doing so would be like tearing down a load-bearing wall — not the best idea." |

Both messages are reproduced verbatim, including the wording of the second.

### LEAD-182 Campaign Tag names are unique

A stored uniqueness rule on the name of a Campaign Tag, with the message "Tag name already exists!".
It is a different entity from the Lead Tag of `LEAD-017`, and the two catalogues are
independent even though their message is worded identically.

### LEAD-183 The lead generation catalogues are unique by name

| Entity | Message |
|---|---|
| Industry | "Industry name already exists!" |
| Role | "Role name already exists!" |
| Seniority | "Name already exists!" |

The display name of a Role and of a Seniority replaces underscores by spaces and applies title case,
so that a stored service value reads as a label.

### LEAD-184 A blacklisted telephone number is unique and is archived rather than deleted

A stored uniqueness rule on the number of a Telephone Blacklist record, with the message "Number
already exists". Creating a number that already exists as an archived record reactivates that
record; creating one that already exists and is active returns the existing record. A number that
cannot be sanitised is refused with "*the parsing error* Please correct the number and try again."

### LEAD-185 Searching by telephone needs three characters

A search over the telephone search field with fewer than three characters is refused with "Please
enter at least 3 characters when searching a Phone number." A record whose entity declares no
primary telephone field is refused with "Missing definition of phone fields."

### LEAD-186 Removing a number from the telephone blacklist is a privileged operation

A user without the right to un-blacklist is refused with "You do not have the access right to
unblacklist phone numbers. Please contact your administrator."

### LEAD-187 The two digest figures are skipped rather than refused

Reading either digest figure without the salesperson group raises the access error "Do not have
access, skip this data for user's digest email". The digest machinery treats that error as "omit
this figure for this recipient" rather than as a failure of the whole digest.

### LEAD-188 The Probability Rebuild Wizard writes both parameters and reloads the entity

Confirming the wizard writes the scoring variable list as a comma-separated list of field names — an
empty selection stores an empty text — and the scoring start date as a date written as year, month
and day separated by hyphens. Writing the variable list reloads the Lead entity, because the set of
fields the probability derivation depends on is dynamic.

### LEAD-189 Only a stage available to the team may be offered

The stage selector of a Lead offers the stages with no team restriction plus the stages whose team
list contains the record's team. The pipeline column expansion of `LEAD-169` uses the same
condition. A direct write of a stage outside that set is stored; only the interface restricts it.

---

## 12. Rule identifier mapping

This domain was consolidated from two independently written descriptions. One of them carried a
numbered rule catalogue whose identifiers are listed in the last column below; the other stated its
rules inside its entity, calculation, state machine and workflow documents and used **no** rule
identifiers at all, so it has no former identifier to map. Every rule of both descriptions is in the
catalogue above under one single scheme, a short domain prefix followed by a three-digit sequence
number.

The rules whose last column is empty are the ones that only the second description carried, or that
the consolidation added from the behaviour of the system; the summary of where each of them was
stated is:

| Rules | Where the second description stated them |
|---|---|
| `LEAD-175` to `LEAD-189` | its entity document, in the sections on the Recurring Plan, the Stage, the Sales Team alias, the lead generation catalogues, the campaign attribution entities and the periodic digest; its calculation document, in the section on the unique-name counter; its state machine document, in the section on the pipeline |

The full mapping follows.

| Identifier in this file | Rule | Former identifier of the numbered catalogue |
|---|---|---|
| `LEAD-001` | Title is mandatory | LEAD-RULE-001 |
| `LEAD-002` | Type is mandatory and has two values only | LEAD-RULE-002 |
| `LEAD-003` | Type never regresses automatically | LEAD-RULE-003 |
| `LEAD-004` | Priority values | LEAD-RULE-004 |
| `LEAD-005` | Probability bounds | LEAD-RULE-005 |
| `LEAD-006` | A record in a won stage must have a probability of one hundred | LEAD-RULE-006 |
| `LEAD-007` | A record cannot be won and lost at the same time | LEAD-RULE-007 |
| `LEAD-008` | Salesperson must be an internal user | LEAD-RULE-008 |
| `LEAD-009` | Company coherence of the salesperson, the team and the customer | LEAD-RULE-009 |
| `LEAD-010` | Company may be empty | LEAD-RULE-010 |
| `LEAD-011` | Currency follows the company | LEAD-RULE-011 |
| `LEAD-012` | Web address is normalized on write | LEAD-RULE-012 |
| `LEAD-013` | Stage must belong to the team of the record | LEAD-RULE-013 |
| `LEAD-014` | Stage deletion is restricted | LEAD-RULE-014 |
| `LEAD-015` | Lost reason deletion is restricted | LEAD-RULE-015 |
| `LEAD-016` | Recurring plan month count is non-negative | LEAD-RULE-016 |
| `LEAD-017` | Tag names are unique | LEAD-RULE-017 |
| `LEAD-018` | Recurring revenue is only kept for users of the recurring revenue group | LEAD-RULE-018 |
| `LEAD-019` | Properties follow the team | LEAD-RULE-019 |
| `LEAD-020` | The address block travels as a whole | LEAD-RULE-020 |
| `LEAD-021` | Contact name derivation | LEAD-RULE-021 |
| `LEAD-022` | Company name derivation | LEAD-RULE-022 |
| `LEAD-023` | Job position and web address derivation | LEAD-RULE-023 |
| `LEAD-024` | Language derivation | LEAD-RULE-024 |
| `LEAD-025` | When the email of the Lead is written onto the customer | LEAD-RULE-025 |
| `LEAD-026` | When the telephone number of the Lead is written onto the customer | LEAD-RULE-026 |
| `LEAD-027` | When the email or the telephone of the customer is copied onto the Lead | LEAD-RULE-027 |
| `LEAD-028` | Warning flags before saving | LEAD-RULE-028 |
| `LEAD-029` | A portal user never overwrites the email of a customer that has a salesperson | LEAD-RULE-029 |
| `LEAD-030` | Telephone quality | LEAD-RULE-030 |
| `LEAD-031` | Email quality | LEAD-RULE-031 |
| `LEAD-032` | Telephone reformatting on change | LEAD-RULE-032 |
| `LEAD-033` | Email domain criterion | LEAD-RULE-033 |
| `LEAD-034` | Blacklists | LEAD-RULE-034 |
| `LEAD-035` | Customer visibility on a lead form | LEAD-RULE-035 |
| `LEAD-036` | Choosing a commercial entity that contradicts the customer clears the customer | LEAD-RULE-036 |
| `LEAD-037` | Commercial entity derivation | LEAD-RULE-037 |
| `LEAD-038` | Posting a message that names a recipient can attach the customer | LEAD-RULE-038 |
| `LEAD-039` | Replies are addressed to the team alias | LEAD-RULE-039 |
| `LEAD-040` | Won status definition | LEAD-RULE-040 |
| `LEAD-041` | Writing a won stage forces the record open and won | LEAD-RULE-041 |
| `LEAD-042` | A stage change between two won stages keeps the closing date | LEAD-RULE-042 |
| `LEAD-043` | Closing date rules | LEAD-RULE-043 |
| `LEAD-044` | Assignment date rules | LEAD-RULE-044 |
| `LEAD-045` | Last stage update date | LEAD-RULE-045 |
| `LEAD-046` | Days to assign and days to close are whole days | LEAD-RULE-046 |
| `LEAD-047` | Rotting | LEAD-RULE-047 |
| `LEAD-048` | Marking a record lost | LEAD-RULE-048 |
| `LEAD-049` | Marking a record won | LEAD-RULE-049 |
| `LEAD-050` | Restoring a lost record | LEAD-RULE-050 |
| `LEAD-051` | Unarchiving without restoring | LEAD-RULE-051 |
| `LEAD-052` | Archiving a won record keeps it won | LEAD-RULE-052 |
| `LEAD-053` | Probability is automatic while it equals the automated probability | LEAD-RULE-053 |
| `LEAD-054` | Explicitly aligning the probability | LEAD-RULE-054 |
| `LEAD-055` | Changing the won flag of a stage rewrites its records | LEAD-RULE-055 |
| `LEAD-056` | Warning when toggling the won flag | LEAD-RULE-056 |
| `LEAD-057` | Tracking subtypes | LEAD-RULE-057 |
| `LEAD-058` | Creation message | LEAD-RULE-058 |
| `LEAD-059` | Deleting a Lead detaches its meetings | LEAD-RULE-059 |
| `LEAD-060` | Potential duplicate detection | LEAD-RULE-070 |
| `LEAD-061` | Duplicate search used by conversion, mass conversion and assignment | LEAD-RULE-071 |
| `LEAD-062` | Conversion is refused on a closed record | LEAD-RULE-072 |
| `LEAD-063` | Conversion skips archived and won records | LEAD-RULE-073 |
| `LEAD-064` | Customer handling during conversion | LEAD-RULE-074 |
| `LEAD-065` | Salesperson allocation during conversion | LEAD-RULE-075 |
| `LEAD-066` | Merge requires at least two records | LEAD-RULE-076 |
| `LEAD-067` | Manual merge is limited to five records | LEAD-RULE-077 |
| `LEAD-068` | Survivor selection | LEAD-RULE-078 |
| `LEAD-069` | Merged type and priority | LEAD-RULE-079 |
| `LEAD-070` | Merged description | LEAD-RULE-080 |
| `LEAD-071` | Merged address | LEAD-RULE-081 |
| `LEAD-072` | Merged lost reason | LEAD-RULE-082 |
| `LEAD-073` | Merged tags | LEAD-RULE-083 |
| `LEAD-074` | Every other merged scalar takes the first non-empty value | LEAD-RULE-084 |
| `LEAD-075` | Merged flags and collections added by bridges | LEAD-RULE-085 |
| `LEAD-076` | Probability is not merged | LEAD-RULE-086 |
| `LEAD-077` | Forced salesperson and team during a merge | LEAD-RULE-087 |
| `LEAD-078` | Stage repair after a forced team | LEAD-RULE-088 |
| `LEAD-079` | History, activities, attachments and meetings move to the survivor | LEAD-RULE-089 |
| `LEAD-080` | Followers move only when recently active | LEAD-RULE-090 |
| `LEAD-081` | Merge summary note | LEAD-RULE-091 |
| `LEAD-082` | Merged-away records are deleted with elevated rights | LEAD-RULE-092 |
| `LEAD-083` | Deduplication during mass conversion | LEAD-RULE-093 |
| `LEAD-084` | A team assignment condition must be a valid search condition | LEAD-RULE-100 |
| `LEAD-085` | A member assignment condition must be a valid search condition | LEAD-RULE-101 |
| `LEAD-086` | A member preference condition must be a valid search condition | LEAD-RULE-102 |
| `LEAD-087` | Membership uniqueness in single-membership mode | LEAD-RULE-103 |
| `LEAD-088` | A member must be allowed in the company of the team | LEAD-RULE-104 |
| `LEAD-089` | Single-membership mode archives the other memberships | LEAD-RULE-105 |
| `LEAD-090` | Archiving a user archives its memberships | LEAD-RULE-106 |
| `LEAD-091` | The main team of a user is the oldest active membership | LEAD-RULE-107 |
| `LEAD-092` | Writing the member list of a team | LEAD-RULE-108 |
| `LEAD-093` | Two shipped teams cannot be deleted | LEAD-RULE-109 |
| `LEAD-094` | A team actively used by sales cannot be deleted | LEAD-RULE-110 |
| `LEAD-095` | Deleting a team folds its scoring statistics into the cross-team statistics | LEAD-RULE-111 |
| `LEAD-096` | Clearing both usage flags clears the alias | LEAD-RULE-112 |
| `LEAD-097` | Writing a usage flag rewrites the alias | LEAD-RULE-113 |
| `LEAD-098` | Automatic assignment requires an administrator | LEAD-RULE-114 |
| `LEAD-099` | Which leads the team allocation considers | LEAD-RULE-115 |
| `LEAD-100` | Which leads the member distribution considers | LEAD-RULE-116 |
| `LEAD-101` | Member eligibility | LEAD-RULE-117 |
| `LEAD-102` | A member only receives leads its condition accepts | LEAD-RULE-118 |
| `LEAD-103` | Every assigned lead becomes an opportunity | LEAD-RULE-119 |
| `LEAD-104` | Deduplication during allocation | LEAD-RULE-120 |
| `LEAD-105` | Manual assignment forces the quota and ignores the creation window | LEAD-RULE-121 |
| `LEAD-106` | Assignment from the settings screen skips opted-out teams | LEAD-RULE-122 |
| `LEAD-107` | Leads created without a salesperson are given to the team leader when rule-based assignment is off | LEAD-RULE-123 |
| `LEAD-108` | Only an administrator may change the scoring configuration | LEAD-RULE-130 |
| `LEAD-109` | The scoring start date must be a real date | LEAD-RULE-131 |
| `LEAD-110` | Only fields that exist on the Lead entity are used as scoring variables | LEAD-RULE-132 |
| `LEAD-111` | The stage is always a scoring variable | LEAD-RULE-133 |
| `LEAD-112` | Rebuilding the frequency table requires deletion rights on the Lead entity | LEAD-RULE-134 |
| `LEAD-113` | Changing the scoring variable list reloads the Lead entity | LEAD-RULE-135 |
| `LEAD-114` | A record with no stage has a probability of zero | LEAD-RULE-136 |
| `LEAD-115` | A record in a won stage has a probability of one hundred | LEAD-RULE-137 |
| `LEAD-116` | A pending record is always strictly between zero and one hundred | LEAD-RULE-138 |
| `LEAD-117` | Statistics are never negative | LEAD-RULE-139 |
| `LEAD-118` | Website contact form | LEAD-RULE-140 |
| `LEAD-119` | Live chat lead creation and channel access | LEAD-RULE-141 |
| `LEAD-120` | Event lead rules do not run during data import | LEAD-RULE-142 |
| `LEAD-121` | One lead per registration and per rule | LEAD-RULE-143 |
| `LEAD-122` | One generation request per event | LEAD-RULE-144 |
| `LEAD-123` | Only event managers may regenerate the leads of an event | LEAD-RULE-145 |
| `LEAD-124` | Confirming a sales order may raise the expected revenue | LEAD-RULE-146 |
| `LEAD-125` | Lead mining errors | LEAD-RULE-147 |
| `LEAD-126` | Enrichment eligibility | LEAD-RULE-148 |
| `LEAD-127` | Enrichment marks the record even when nothing is found | LEAD-RULE-149 |
| `LEAD-128` | Enrichment only fills empty fields | LEAD-RULE-150 |
| `LEAD-129` | Enrichment notifications | LEAD-RULE-151 |
| `LEAD-130` | Lead Generation Rule pattern must compile | LEAD-RULE-152 |
| `LEAD-131` | Lead Generation Rule contact count bounds | LEAD-RULE-153 |
| `LEAD-132` | Reveal View retention | LEAD-RULE-154 |
| `LEAD-133` | Mail plugin lead creation requires a known contact | LEAD-RULE-155 |
| `LEAD-134` | Survey leads are always opportunities | LEAD-RULE-156 |
| `LEAD-135` | Lead visibility | LEAD-RULE-160 |
| `LEAD-136` | A portal user may only write on a lead assigned to its own family | LEAD-RULE-161 |
| `LEAD-137` | A portal user may only point a link field at a record it may read | LEAD-RULE-162 |
| `LEAD-138` | A portal user may only update a closed list of contact fields | LEAD-RULE-163 |
| `LEAD-139` | Creating an opportunity from the portal requires a grade | LEAD-RULE-164 |
| `LEAD-140` | Geographic assignment needs a country | LEAD-RULE-165 |
| `LEAD-141` | Only weighted partners receive forwarded leads | LEAD-RULE-166 |
| `LEAD-142` | A partner that declined a lead is never proposed again for it | LEAD-RULE-167 |
| `LEAD-143` | No partner found | LEAD-RULE-168 |
| `LEAD-144` | Forwarding requires an email address on the recipient | LEAD-RULE-169 |
| `LEAD-145` | Declining a lead unsubscribes the whole partner family | LEAD-RULE-170 |
| `LEAD-146` | Accepting a lead converts it | LEAD-RULE-171 |
| `LEAD-147` | Declining messages | LEAD-RULE-172 |
| `LEAD-148` | A grade with a default pricelist writes that pricelist on the contact | LEAD-RULE-173 |
| `LEAD-149` | A sales order cannot grant two different grades | LEAD-RULE-174 |
| `LEAD-150` | Forwarding a lead assigns the salesperson of the recipient | LEAD-RULE-175 |
| `LEAD-151` | Opening a forwarded lead as an external user goes to the portal | LEAD-RULE-176 |
| `LEAD-152` | Publishing a reseller page requires write access to the contact | LEAD-RULE-177 |
| `LEAD-153` | The directory falls back to all countries | LEAD-RULE-178 |
| `LEAD-154` | Grade publication filters the directory | LEAD-RULE-179 |
| `LEAD-155` | Access rights per entity | LEAD-RULE-180 |
| `LEAD-156` | The scoring frequency table is written with elevated rights | LEAD-RULE-181 |
| `LEAD-157` | Duplicate counting crosses company boundaries | LEAD-RULE-182 |
| `LEAD-158` | Long-running jobs commit in batches | LEAD-RULE-183 |
| `LEAD-159` | The enrichment job locks each batch | LEAD-RULE-184 |
| `LEAD-160` | A concurrent update during the scoring recomputation is retried, not lost | LEAD-RULE-185 |
| `LEAD-161` | Rounding of monetary derivations | LEAD-RULE-186 |
| `LEAD-162` | Rounding of the probability | LEAD-RULE-187 |
| `LEAD-163` | Rounding of the daily quota | LEAD-RULE-188 |
| `LEAD-164` | Rounding when folding team statistics | LEAD-RULE-189 |
| `LEAD-165` | Dates are stored in coordinated universal time | LEAD-RULE-190 |
| `LEAD-166` | The celebration message uses a time zone-aware midnight | LEAD-RULE-191 |
| `LEAD-167` | Ordering by the reader's own activity deadline | LEAD-RULE-192 |
| `LEAD-168` | Default ordering of a Lead list | LEAD-RULE-193 |
| `LEAD-169` | Kanban column expansion | LEAD-RULE-194 |
| `LEAD-170` | Suggested recipients of a Lead | LEAD-RULE-195 |
| `LEAD-171` | Creation values carried by a suggested recipient | LEAD-RULE-196 |
| `LEAD-172` | Text message target of a Lead | LEAD-RULE-197 |
| `LEAD-173` | Text message target of a website visitor | LEAD-RULE-198 |
| `LEAD-174` | Time spent in each stage | LEAD-RULE-199 |
| `LEAD-175` | A Recurring Revenue Plan is required as soon as a recurring revenue is entered | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-176` | A Stage that holds at least one Lead cannot be deleted | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-177` | A Sales Team always owns an alias | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-178` | What a duplicated Lead does not carry over | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-179` | Campaign identifiers, source names and medium names are made unique | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-180` | A Campaign needs both an identifier and a title | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-181` | Delivered attribution records that may not be deleted | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-182` | Campaign Tag names are unique | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-183` | The lead generation catalogues are unique by name | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-184` | A blacklisted telephone number is unique and is archived rather than deleted | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-185` | Searching by telephone needs three characters | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-186` | Removing a number from the telephone blacklist is a privileged operation | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-187` | The two digest figures are skipped rather than refused | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-188` | The Probability Rebuild Wizard writes both parameters and reloads the entity | *(none: the rule was not in the numbered catalogue)* |
| `LEAD-189` | Only a stage available to the team may be offered | *(none: the rule was not in the numbered catalogue)* |

Two properties of the scheme are worth stating. First, the numbers run consecutively in the order in
which the rules appear, so a reader can find a rule by its number without a search; a rule added
later takes the next free number at the end of its group's section and is appended to this table.
Second, every identifier in this file is stable: the other documents of this folder, and the
acceptance criteria, cite these identifiers and no other.

---

## 13. Reconciliation notes

| Subject | The two statements | Resolution |
|---|---|---|
| Rule identifiers | One description numbered its rules; the other did not. | The numbered scheme is kept and extended. Section 12 maps the former identifiers and names where the unnumbered statements came from. |
| Deleting a Sales Team that carries sales orders | One description said "more than five active sales orders"; the source refuses at **five or more**. | `LEAD-094` now states five or more. The message itself is unchanged. |
| The electronic mail domain criterion | One description said the criterion is empty at a free public provider; the other said it is the whole address. | The whole address is correct, and the whole text when the value carries no at sign. `LEAD-033` carries the correct rule, and the entity and calculation documents were corrected to match it. |
| Enrichment eligibility | One description tied the manual enrichment control to the enrichment mode setting; the other listed six properties of the record. | The six properties are correct. `LEAD-126` carries them, and the entity document was corrected: the mode decides whether the scheduled job runs, nothing else. |
| Field naming | One description used descriptive canonical names. | Every rule in this file names fields by their storage name, in code font, with the full name in words on first use, because the storage names are contractual. |
