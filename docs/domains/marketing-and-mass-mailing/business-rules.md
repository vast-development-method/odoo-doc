# Marketing business rules

The complete rule catalogue of the domain. Each rule has a stable number so that other documents can cite it. Messages are reproduced exactly as they are shown to the user. Where a message contains a placeholder it is written in the form `%(name)s` or `%s` exactly as the system substitutes it.

## 1. Mass Mailing structure and validation

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-001` | The subject is required on every mailing. | Standard required-field refusal. |
| `MKT-RULE-002` | A mailing of type `mail` must have a sender address. Enforced as a database check together with the type. | *"email from is required for mailing"* |
| `MKT-RULE-003` | The comparison-test percentage must be between 0 and 100 inclusive. Enforced as a database check. | *"The A/B Testing Percentage needs to be between 0 and 100%"* |
| `MKT-RULE-004` | The recipient entity is required and must be an entity that declares itself mailable. | Standard domain refusal on the field. |
| `MKT-RULE-005` | A loaded saved filter must target the same recipient entity as the mailing. Checked on create and on write of either field. | *"The saved filter targets different recipients and is incompatible with this mailing."* |
| `MKT-RULE-006` | Clearing the campaign of a mailing that keeps comparison testing enabled is refused. | *"A campaign should be set when A/B test is enabled"* |
| `MKT-RULE-007` | When a marketing card campaign is set on a mailing, the recipient entity must be that campaign's target entity. | *"Card Campaign Mailing should target model %(model_name)s"* |
| `MKT-RULE-008` | The answer address is required in the form when the answer mode is `new`. | Standard required-field refusal. |
| `MKT-RULE-009` | The medium is required in the form. | Standard required-field refusal. |
| `MKT-RULE-010` | The body is required in the form when the editable body is filled, in order that the inline body is never lost. | Standard required-field refusal. |
| `MKT-RULE-011` | The name of a mailing is the name of its owned Campaign Source and is therefore unique across every source in the system; a duplicate name is silently made unique by appending a bracketed counter. | — |
| `MKT-RULE-012` | A name supplied only through screen defaults is ignored when creating a mailing, in order to avoid a uniqueness violation. | — |
| `MKT-RULE-013` | Writing the subject or the name of more than one mailing at once is refused. | *"You cannot update multiple records with the same name. The name should be unique!"* |

## 2. Editing guards

| Rule | Statement |
|---|---|
| `MKT-RULE-020` | While the state is `sending` or `done`, the following fields are read-only in the form: subject, preview, sender address, answer mode, answer address, attachments, recipient entity, mailing lists, saved filter, condition, body, campaign, medium, name, mail server, archive-keeping flag. |
| `MKT-RULE-021` | While the state is not `draft`, the comparison-test flag, the comparison-test percentage, the comparison-test criterion, the comparison-test moment and the exclusion-list flag are read-only. |
| `MKT-RULE-022` | The schedule date remains editable in the board while the state is `draft` or `in_queue`, and is read-only afterwards. |
| `MKT-RULE-023` | The exclusion-list control is hidden when no mail server is set and the flag is still true, in order that only an operator who has deliberately chosen a dedicated server can switch it off. |
| `MKT-RULE-024` | The mail server control is hidden unless the dedicated-server setting is on, and its choices exclude servers that have a personal owner. |
| `MKT-RULE-025` | The campaign field and the campaign search field are visible only to the campaign-management group. |
| `MKT-RULE-026` | The medium, source and advanced fields are visible only in the technical-features mode; the source is always read-only. |

## 3. Sending guards

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-030` | Sending with an empty remaining audience is refused. | *"There are no recipients selected."* |
| `MKT-RULE-031` | Queueing or sending a mailing that carries a marketing card campaign with at least one recipient lacking an up-to-date card is refused. | *"You should update all the cards for %(mailing)s before scheduling a mailing."* |
| `MKT-RULE-032` | Pressing Send writes the schedule type to `now`, which clears any previously chosen moment. |
| `MKT-RULE-033` | Pressing Schedule with a moment already strictly in the future queues directly; otherwise the schedule assistant opens. |
| `MKT-RULE-034` | Cancelling is only offered while the state is `in_queue`. |
| `MKT-RULE-035` | Retrying is only offered while the state is `done` and at least one delivery is in error. |
| `MKT-RULE-036` | Duplicating from the header is only offered while the state is `done`. |
| `MKT-RULE-037` | The queue job selects only mailings in state `in_queue` or `sending` whose schedule date is empty or strictly earlier than the current moment. |
| `MKT-RULE-038` | The queue job acts with the identity of the mailing's responsible user, falling back to its last writer, falling back to the job's own user; the recipient rendering therefore uses that person's language and time zone. |
| `MKT-RULE-039` | A mailing whose remaining audience is empty when the queue job reaches it is closed as `done` rather than refused. |

## 4. Audience and exclusion

| Rule | Statement |
|---|---|
| `MKT-RULE-050` | The audience is the set of records of `mailing_model_real` matching the stored condition. An unparsable condition yields the impossible condition `id IN ()`, therefore an empty audience, therefore a refusal or an immediate close, never an accidental send to everybody. |
| `MKT-RULE-051` | When the recipient entity is Mailing List, the records actually addressed are Mailing Contacts, and the default condition is `list_ids IN (the mailing's lists)`. |
| `MKT-RULE-052` | Choosing a new recipient entity resets the condition to that entity's default condition and clears the loaded saved filter. |
| `MKT-RULE-053` | Editing the condition by hand does **not** clear the loaded saved filter; the two values can therefore differ, and the form shows both. |
| `MKT-RULE-054` | Deleting a saved filter clears the reference on the mailings that used it but leaves their stored condition untouched. |
| `MKT-RULE-055` | With a marketing card campaign the audience is additionally restricted to records that already have a card. |
| `MKT-RULE-056` | A record that already has a trace for this mailing is never contacted again by it. |
| `MKT-RULE-057` | With comparison testing enabled, a record already traced for the **campaign** is never contacted again by any version of that campaign. |
| `MKT-RULE-058` | The campaign winner mailing excludes every record traced by any version of the campaign, which is what makes the final send reach only the untouched remainder. |
| `MKT-RULE-059` | The sample size of a comparison-test version is `max(floor(audience × percentage ÷ 100), 1)`, then reduced to the number of remaining records when it exceeds it; a non-empty remainder with a computed size of zero is raised to the whole remainder. |
| `MKT-RULE-060` | The sample is drawn at random from the sorted remainder, therefore the draw is reproducible only in its size, not in its membership. |

| Rule | Exclusion, evaluated in this order, first match wins | Resulting delivery record |
|---|---|---|
| `MKT-RULE-061` | The recipient is in the blocked-address register and the exclusion-list flag is on. | status `cancel`, failure `mail_bl` |
| `MKT-RULE-062` | The recipient has no destination address at all. | status `cancel` when archives are not kept, otherwise `error`; failure `mail_email_missing` |
| `MKT-RULE-063` | The recipient has no destination address that normalises. | same; failure `mail_email_invalid` |
| `MKT-RULE-064` | Every normalised destination of the recipient is in the opted-out set. | status `cancel`, failure `mail_optout` |
| `MKT-RULE-065` | Every normalised destination is in the already-contacted set. | status `cancel`, failure `mail_dup` |
| `MKT-RULE-066` | Every normalised destination already received, in this same run, a message with the same subject, the same body and the same attachment count. | status `cancel`, failure `mail_dup` |

| Rule | Statement |
|---|---|
| `MKT-RULE-067` | Switching the exclusion-list flag off removes rule `MKT-RULE-061` only; opt-out, duplicate and address-validity rules still apply. |
| `MKT-RULE-068` | For a list-based mailing, a contact is opted out only when it is opted out of at least one of the mailing's lists **and** opted in to none of them. Two contacts sharing one address, one opted in and one opted out, therefore still receive the message. |
| `MKT-RULE-069` | With a marketing card campaign the already-contacted set is forced empty, because each recipient receives a different image. |
| `MKT-RULE-070` | For a text-message mailing the equivalent exclusions are, in order: blocked number (`sms_blacklist`), opted-out record (`sms_optout`), already-used number in this run (`sms_duplicate`), unusable number (`sms_number_format`), missing number (`sms_number_missing`); all produce a cancelled message. |
| `MKT-RULE-071` | A text-message mailing on an entity that exposes neither a telephone field nor a linked contact is refused. Message: *"Unsupported %s for mass SMS"* |

## 5. Batching, resumability and transactions

| Rule | Statement |
|---|---|
| `MKT-RULE-080` | The batch size is the system parameter `mail.batch_size`; when it is absent or zero the value 50 is used. |
| `MKT-RULE-081` | Outside automated tests, the sending pass commits after each batch and reports its progress; the record caches are dropped between batches. |
| `MKT-RULE-082` | After the pass, the state, the sent date and the statistics flag are written and committed immediately outside automated tests. |
| `MKT-RULE-083` | A pass interrupted after a commit is resumable: the already-created traces remove their recipients from the remaining audience of the next pass. |
| `MKT-RULE-084` | The card-refresh routine commits after each batch of 100 rendered cards and drops the cached images, in order to bound memory use. |
| `MKT-RULE-085` | The removal of failed deliveries during a retry is done in pages of at most 1000 outgoing messages. |
| `MKT-RULE-086` | The removal of old cancelled outgoing mail is done in pages of at most 10 000 records, ordered by key ascending. |

## 6. Dates and moments

| Rule | Statement |
|---|---|
| `MKT-RULE-090` | Every stored moment is in coordinated universal time and is displayed in the reader's time zone. |
| `MKT-RULE-091` | The schedule date is cleared whenever the schedule type is `now`. |
| `MKT-RULE-092` | The next departure is the later of the schedule date and the current moment; with no schedule date it is the current moment. A mailing scheduled in the past is therefore sent at the next run of the queue job, never retroactively. |
| `MKT-RULE-093` | The calendar moment is the sent date when the state is `done`, the next departure when it is `in_queue`, the current moment when it is `sending`, and empty when it is `draft`. |
| `MKT-RULE-094` | The favorite moment is set once, when the design is first marked as a favorite, and cleared when the mark is removed. |
| `MKT-RULE-095` | The opt-out moment is set to the current database moment when a subscription becomes opted out and cleared when it becomes opted in. |
| `MKT-RULE-096` | Writing an opt-out moment or an opt-out reason on a subscription forces the opt-out flag to true. |
| `MKT-RULE-097` | Feedback is attached to the subscriptions opted out within the last ten minutes; older opt-outs are not touched by a late feedback submission. |
| `MKT-RULE-098` | The comparison-test moment defaults to the current moment plus one day and is explicitly carried over when a version is duplicated. |
| `MKT-RULE-099` | The statistics message is sent for mailings whose sent date is between one day and five days before the current moment, once, because the flag is cleared for the whole selection before the messages are built. |
| `MKT-RULE-100` | The automatic blocking after repeated bounces only triggers when the newest qualifying bounce is more than one week after the oldest, within a window of thirteen weeks. |
| `MKT-RULE-101` | Creating a mailing from a calendar day only pre-fills the schedule when that day is strictly in the future. |

## 7. Rounding and numeric presentation

| Rule | Statement |
|---|---|
| `MKT-RULE-110` | All five mailing percentages and the campaign percentages are rounded to two decimal places, half away from zero. |
| `MKT-RULE-111` | Every percentage denominator that would be zero is replaced by 1, therefore an empty mailing reports 0.00 and never fails. |
| `MKT-RULE-112` | The list quality percentages are not rounded when computed; they are rounded only for display. When a list has no contact all three are exactly zero. |
| `MKT-RULE-113` | The sample size uses truncation towards zero, then a floor of 1 whenever the audience is not empty. |
| `MKT-RULE-114` | Tag colors are a random integer between 1 and 11 inclusive; a color of zero means the tag is not shown on cards. |

## 8. Uniqueness

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-120` | Campaign names are unique. | *"The name must be unique"* |
| `MKT-RULE-121` | Campaign Source names are unique. | *"The name must be unique"* |
| `MKT-RULE-122` | Campaign Medium names are unique. | *"The name must be unique"* |
| `MKT-RULE-123` | Campaign Tag names are unique. | *"Tag name already exists!"* |
| `MKT-RULE-124` | Marketing Card Campaign Tag names are unique. | *"Tags may not reuse existing names."* |
| `MKT-RULE-125` | A contact subscribes at most once to a list. | *"A mailing contact cannot subscribe to the same mailing list multiple times."* |
| `MKT-RULE-126` | A marketing card exists at most once per (campaign, record). | *"Each record should be unique for a campaign"* |
| `MKT-RULE-127` | Link Tracker Codes are unique across the system. | *"Code must be unique."* |
| `MKT-RULE-128` | The combination of target address, campaign, medium, source and label of a Link Tracker is unique; empty and empty-string labels are the same value. | *"Combinations of Link Tracker values (URL, campaign, medium, source, and label) must be unique."* then a line break, then *"The following combinations are already used: "* then a line break and one `- <tuple>` per duplicate. |
| `MKT-RULE-129` | Mailing Contact addresses are deliberately **not** unique; two contacts may share an address in order to hold different per-list preferences. |
| `MKT-RULE-130` | Mailing List names are deliberately **not** unique. |

## 9. Deletion and archiving guards

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-140` | A mailing list used by at least one mailing whose state is not `done` cannot be archived. | *"At least one of the mailing list you are trying to archive is used in an ongoing mailing campaign."* |
| `MKT-RULE-141` | A Campaign Medium linked to at least one mailing cannot be deleted. | *"You cannot delete these UTM Mediums as they are linked to the following mailings in Mass Mailing:"* then a line break and the comma-separated quoted subjects. |
| `MKT-RULE-142` | A Campaign Source linked to at least one mailing cannot be deleted. | *"You cannot delete these UTM Sources as they are linked to the following mailings in Mass Mailing:"* then a line break and the comma-separated quoted subjects. |
| `MKT-RULE-143` | The six protected mediums (`Email`, `Direct`, `Website`, `X`, `Facebook`, `LinkedIn`) cannot be deleted. | *"Oops, you can't delete the Medium '%s'."* then a line break and *"Doing so would be like tearing down a load-bearing wall — not the best idea."* |
| `MKT-RULE-144` | With the Text Message Marketing package, the `Text Message` medium cannot be deleted. | *"The UTM medium '%s' cannot be deleted as it is used in some main functional flows, such as the SMS Marketing."* |
| `MKT-RULE-145` | The `Referral` source cannot be deleted. | *"You cannot delete the 'Referral' UTM source record."* |
| `MKT-RULE-146` | The `Marketing Card` source cannot be deleted. | *"The UTM source '%s' cannot be deleted as it is used to promote marketing cards campaigns."* |
| `MKT-RULE-147` | A campaign used by the recruitment process cannot be deleted. | *"The UTM campaign '%s' cannot be deleted as it is used in the recruitment process."* |
| `MKT-RULE-148` | Removing an entity from the registry deletes the marketing card campaigns targeting it. | — |
| `MKT-RULE-149` | Changing the target entity of a marketing card campaign that already has cards is refused. | *"Model of campaign %(campaign)s may not be changed as it already has cards"* |
| `MKT-RULE-150` | Deleting a mailing deletes its traces; the Link Tracker records survive with an empty mailing reference and keep their clicks. | — |
| `MKT-RULE-151` | Deleting a mailing list deletes its subscriptions but not its contacts. Deleting a contact deletes its subscriptions. | — |
| `MKT-RULE-152` | A Mailing Opt-Out Reason referenced by a subscription or by a blocked address cannot be deleted. | Standard restricted-deletion refusal. |
| `MKT-RULE-153` | A Campaign Stage referenced by a campaign cannot be deleted. | Standard restricted-deletion refusal. |
| `MKT-RULE-154` | A Campaign Source owned by a mailing or by another source-owning record cannot be deleted. | Standard restricted-deletion refusal. |
| `MKT-RULE-155` | A Link Tracker referenced by a marketing card campaign cannot be deleted. | Standard restricted-deletion refusal. |
| `MKT-RULE-156` | The Marketing Card Campaign board and list keep archived campaigns out of sight; a campaign is archived, never deleted, when it has produced cards. | — |

## 10. Comparison testing

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-170` | Enabling comparison testing without a campaign creates one, named `A/B Test: <subject>`, owned by the mailing's responsible, carrying the mailing's comparison-test moment and criterion. | — |
| `MKT-RULE-171` | Enabling comparison testing on a mailing that already has a campaign never replaces that campaign. | — |
| `MKT-RULE-172` | The winner operation requires all selected mailings to share exactly one campaign. | *"To send the winner mailing the same campaign should be used by the mailings"* |
| `MKT-RULE-173` | The winner operation requires that the campaign is not already completed. | *"To send the winner mailing the campaign should not have been completed."* |
| `MKT-RULE-174` | With a non-manual criterion, at least one version must have reached `done`. | *"No mailing for this A/B testing campaign has been sent yet! Send one first and try again later."* |
| `MKT-RULE-175` | Promoting a version manually requires comparison testing to be enabled on it. | *"A/B test option has not been enabled"* |
| `MKT-RULE-176` | The winner is the version in state `done` with the highest value of the criterion; the comparison is made with elevated rights so that criteria depending on other domains are readable. |
| `MKT-RULE-177` | Ties are resolved by the order in which the versions are read, which is the campaign's mailing order; no explicit tie-break exists. This is an industry-standard completion: a replacement should make the tie-break explicit and deterministic, for example the lowest mailing key first, and document it. | — |
| `MKT-RULE-178` | The winner send is a **copy** at 100 percent named `" <name> (final)"`; the original versions are never re-sent. | — |
| `MKT-RULE-179` | Recording the winner marks the campaign completed, which permanently blocks any further winner operation on it. | — |
| `MKT-RULE-180` | Opening the version comparison requires a campaign. | *"No mailing campaign has been found"* |
| `MKT-RULE-181` | The comparison-test scheduled job is inactive by default and is switched on and off together with the campaign setting. | — |
| `MKT-RULE-182` | Creating or writing a comparison-test moment asks the comparison-test job to wake at the earliest such moment among the written records. | — |
| `MKT-RULE-183` | For a text-message mailing the criterion is the text-message criterion of the campaign and the versions considered are the campaign's text-message mailings. | — |

## 11. Link tracking

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-190` | Creating a Link Tracker without a target address is refused. | *"Creating a Link Tracker without URL is not possible"* |
| `MKT-RULE-191` | A target address starting with `?` or `#` is refused, because it would loop on the current page. | *"“%s” is not a valid link, links cannot redirect to the current page."* |
| `MKT-RULE-192` | A bare host is completed with the `http` scheme at creation. | — |
| `MKT-RULE-193` | Campaign, source and medium coming from visitor cookies are discarded when creating a Link Tracker; only explicit values are kept. | — |
| `MKT-RULE-194` | When no title is supplied, the target page's preview title is fetched once at creation; on failure the address itself becomes the title. | — |
| `MKT-RULE-195` | Links are never shortened when they use the electronic-mail, telephone or text-message protocols, when they already point at the short prefix, or when they match one of the skipped addresses. | — |
| `MKT-RULE-196` | The skip list used when shortening a mailing body is `/unsubscribe_from_list`, `/view`, `/cards`. A skipped address matches when it is followed by `#`, `?`, `/` or the end of the address. | — |
| `MKT-RULE-197` | The stored label of a shortened link is the trimmed anchor text truncated to 40 characters, or, for an image anchor, `[media] ` followed by the alternative text or the last path segment of the image source. | — |
| `MKT-RULE-198` | Two links with the same target but different labels are two different trackers, therefore two different short codes, therefore separately counted. | — |
| `MKT-RULE-199` | A visit through a plain short link is not recorded when the caller is recognised as an automated agent. | — |
| `MKT-RULE-200` | A visit through a short link carrying a recipient key is always recorded, and marks that recipient's delivery record opened and clicked. | — |
| `MKT-RULE-201` | A visit whose code matches nothing records nothing and answers "not found". | — |
| `MKT-RULE-202` | The redirection is a permanent redirection to an external address. | — |
| `MKT-RULE-203` | Campaign parameters are appended to the redirection address, unless the system parameter `link_tracker.no_external_tracking` is set and the target host differs from the site host. | — |
| `MKT-RULE-204` | Three consecutive dots in a campaign parameter value are escaped in the redirection address, because some reverse proxies reject that sequence. | — |
| `MKT-RULE-205` | Searching the short address strips the site base address and the `/r/` prefix, therefore both a full short address and a bare code match. | — |
| `MKT-RULE-206` | The short-address host is the current website's base address when the website belongs to the current company, otherwise the company's base address. | — |
| `MKT-RULE-207` | The recent-links query accepts only `newest`, `most-clicked` and `recently-used`; any other value answers *"This filter doesn't exist."* | — |

## 12. Campaign tracking capture

| Rule | Statement |
|---|---|
| `MKT-RULE-210` | The three tracked request parameters are `utm_campaign`, `utm_source` and `utm_medium`, stored in the cookies `odoo_utm_campaign`, `odoo_utm_source` and `odoo_utm_medium`. |
| `MKT-RULE-211` | A cookie is written only when the parameter is present in the request and its value differs from the current cookie value. |
| `MKT-RULE-212` | Cookies are written on the request host, classified as optional (subject to the visitor's consent choices), with a lifetime of 31 days. |
| `MKT-RULE-213` | A record being created inherits the cookie values only when the creating identity is the system identity or is not a salesperson. A salesperson creating a record by hand never inherits a visitor's attribution. |
| `MKT-RULE-214` | A cookie value that is a name rather than a key is resolved by a case-insensitive search that includes archived records, and creates the record when nothing matches. |
| `MKT-RULE-215` | A campaign created implicitly this way is flagged as automatically generated and is excluded from the campaign screens. |
| `MKT-RULE-216` | The unique-name algorithm fills the lowest free counter, reusing holes left by deleted records. |

## 13. Public pages, tokens and permissions

| Rule | Statement | Message or answer |
|---|---|---|
| `MKT-RULE-220` | Every public subscription endpoint requires either a valid token or a signed-in identity. | `bad request` for an anonymous caller without a token. |
| `MKT-RULE-221` | A signed-in caller who is not a Marketing User may not open a mailing-specific page without a token. | `bad request` |
| `MKT-RULE-222` | A token is only accepted together with a mailing key, an address and a record key. | `bad request` |
| `MKT-RULE-223` | The token is the keyed hash of the tuple (database name, mailing key, record key, address) computed with the database secret and the strong hash function, compared in constant time. | `unauthorized` on mismatch. |
| `MKT-RULE-224` | A page endpoint converts "not found" into "unauthorized", in order not to reveal which mailing keys exist. | `unauthorized` |
| `MKT-RULE-225` | Structured-data endpoints answer the strings `error` for a bad request and `unauthorized` for a missing or wrong token, instead of raising. | — |
| `MKT-RULE-226` | The one-click unsubscribe endpoint accepts only the post method and is exempt from the cross-site request token, because the calling mail client has no session. | — |
| `MKT-RULE-227` | The open-tracking endpoint requires a token equal to the keyed hash of the outgoing mail key under the purpose `mass_mailing-mail_mail-open`. | `unauthorized` |
| `MKT-RULE-228` | The statistics-message deactivation endpoint requires a token equal to the keyed hash of the user key under the purpose `mailing-report-deactivated`, and requires that user to be a Marketing User. | `unauthorized` |
| `MKT-RULE-229` | A signed-in caller without a token acts on their **own** normalised address; the address supplied in the request is ignored. | — |
| `MKT-RULE-230` | Opting in through the public page is possible only for lists that are public or that the person already belongs to. | — |
| `MKT-RULE-231` | A private list the person is opted out of is not displayed on the public page. | — |
| `MKT-RULE-232` | Feedback without a chosen reason is refused. | `error` |
| `MKT-RULE-233` | The self-exclusion and re-inclusion buttons appear only when the system parameter `mass_mailing.show_blacklist_buttons` is true. | — |
| `MKT-RULE-234` | A website form submitting to Mailing Contact must name at least one list. | *"Mailing List(s) not found!"* |
| `MKT-RULE-235` | A website form naming a non-public list is refused. | *"You cannot subscribe to the following list anymore : %s"* |
| `MKT-RULE-236` | The website subscription endpoint verifies a human-verification token for the action `website_mass_mailing_subscribe` before doing anything. | The verification error text, shown as a danger notice. |
| `MKT-RULE-237` | The text-message opt-out page accepts a number only when its sanitised form matches the number stored on a trace of that mailing carrying that code. | *"Oops! Number not found"* |
| `MKT-RULE-238` | A number that cannot be sanitised is refused on the same page. | *"Oops! The phone number seems to be incorrect. Please make sure to include the country code."* |
| `MKT-RULE-239` | A text-message opt-out page whose mailing or trace code does not resolve redirects the visitor to the back office rather than revealing the failure. | — |

## 14. Marketing card rules

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-250` | The target entity of a campaign is derived from the preview record and is read-only. | — |
| `MKT-RULE-251` | Allowed target entities are the intersection of the entities present in the system with Contact, Event Track, Event Booth and Event Registration. | — |
| `MKT-RULE-252` | Writing any rendering-relevant field marks every card of the campaign, archived ones included, as needing regeneration. | — |
| `MKT-RULE-253` | Writing the target address updates the campaign's Link Tracker; an empty address falls back to the site base address. | — |
| `MKT-RULE-254` | A rasterisation failure outside automated tests refuses the operation. | *"An error occured while rendering a card for %(record_name)s. Try again or check the server logs for more details."* |
| `MKT-RULE-255` | A preview card is archived on purpose, in order that it is regenerated before a real send. | — |
| `MKT-RULE-256` | The preview page sets the share status to `visited` only when it was empty. | — |
| `MKT-RULE-257` | The image endpoint sets the share status to `shared` only when the caller is a recognised crawler and the status is not already `shared`. | — |
| `MKT-RULE-258` | A card with no image answers "not found" on the image endpoint. | — |
| `MKT-RULE-259` | The redirection endpoint answers a preview-metadata page to a recognised crawler and a redirection to everybody else. | — |
| `MKT-RULE-260` | Clicks from the preview are not counted: the redirection goes through the campaign's short address only when the card is active. | — |
| `MKT-RULE-261` | A card whose last write is older than the retention period is deleted by the cleanup routine; a retention of zero disables the cleanup. | — |
| `MKT-RULE-262` | Rendering a marketing card body is unrestricted, because the rendered field is a read-only related field of a shipped template that only a settings administrator may change. | — |
| `MKT-RULE-263` | Campaign counters treat a shared card as also visited. | — |

## 15. Mailing lists, contacts and subscriptions

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-270` | Creating a contact with both the membership view and the subscription list filled is refused. | *"You should give either list_ids, either subscription_ids to create new contacts."* |
| `MKT-RULE-271` | The default lists of the calling screen are merged into the created subscriptions, never duplicated. | — |
| `MKT-RULE-272` | Duplicating a contact from inside a list screen does not add that screen's lists again. | — |
| `MKT-RULE-273` | The contact opt-out flag is meaningful only when exactly one list is named by the calling screen; outside that situation it is false and searching on it returns nothing. | — |
| `MKT-RULE-274` | The first and last name fields are not searchable unless the split-name setting is on. | — |
| `MKT-RULE-275` | Creating a contact from a typed string parses it as `"Name" <address>`. | — |
| `MKT-RULE-276` | The merge assistant can only be opened from a Mailing List screen. | *"You can only apply this action from Mailing Lists."* |
| `MKT-RULE-277` | The merge inserts only contacts that are not opted out of their source list and are not blocked, and only one contact per address. | — |
| `MKT-RULE-278` | The merge never creates a duplicate in the destination: a contact whose raw address already exists there is skipped. | — |
| `MKT-RULE-279` | The paste-import refuses above 5000 parsed addresses and proposes the file import instead. | *"You have to much emails, please upload a file."* |
| `MKT-RULE-280` | The paste-import warns when nothing parses. | *"No valid email address found."* |
| `MKT-RULE-281` | The paste-import warns when every parsed address is already a member. | *"No contacts were imported. All email addresses are already in the mailing list."* |
| `MKT-RULE-282` | The paste-import keeps the first non-empty name seen for a repeated address. | — |
| `MKT-RULE-283` | The paste-import adds the chosen lists to an existing contact instead of creating a second contact with the same address, when that contact is already a member of at least one chosen list. | — |
| `MKT-RULE-284` | The add-to-list assistant adds only the contacts that are not already members. | *"%s Mailing Contacts have been added. "* |
| `MKT-RULE-285` | A list marked as shown in preferences is offered on the public page and its name may appear in unsubscribe confirmations; a private list is referred to as *"Mailing List #<key>"*. | — |

## 16. Permissions per operation

| Rule | Operation | Who may perform it |
|---|---|---|
| `MKT-RULE-290` | Create, read, update and delete Mass Mailing, Mailing List, Mailing Contact, Mailing Subscription, Mailing Opt-Out Reason, Mailing Trace, Mailing Filter | Marketing User; settings administrators also have full rights on Mass Mailing. |
| `MKT-RULE-291` | Read the analytical view | Marketing User. |
| `MKT-RULE-292` | Create, read, update and delete Campaign, Campaign Source, Campaign Medium, Campaign Stage, Campaign Tag | Marketing User (and, through the platform, every internal user except for deletion, which requires a settings administrator). |
| `MKT-RULE-293` | Read Link Tracker, Link Tracker Code, Link Tracker Click | Every internal user. Full rights: settings administrators, Marketing Users (on the tracker) and website designers. |
| `MKT-RULE-294` | Read the outgoing mail servers and the entity registry | Marketing User (read only). |
| `MKT-RULE-295` | Manage blocked addresses and the unblocking assistant | Marketing User. |
| `MKT-RULE-296` | Manage blocked numbers and the unblocking assistant | Marketing User, with the Text Message Marketing package. |
| `MKT-RULE-297` | Create, read, update and delete their **own** Marketing Card Campaign | Marketing Card User; the record rule restricts write and delete to campaigns whose responsible is the acting user. |
| `MKT-RULE-298` | Access and edit **any** Marketing Card Campaign, and manage Marketing Card Campaign Tags and Marketing Cards | Marketing Card Manager. |
| `MKT-RULE-299` | Read Marketing Card Templates | Marketing Card User. Full rights: settings administrator only. |
| `MKT-RULE-300` | Read Marketing Card records | Every internal user (also create and update), portal users and anonymous visitors (read only). |
| `MKT-RULE-301` | Change the user-defined field definitions of Mailing Contact | Marketing User, through a record rule limited to the Mailing Contact property definition. |
| `MKT-RULE-302` | Open the Marketing settings screen | Settings administrator. |
| `MKT-RULE-303` | Create a shortened link from the public links page | Any signed-in user who has the right to create Link Tracker records; the page itself reports whether they may. |
| `MKT-RULE-304` | Estimate the length of a text-message body from the designer | The acting user must have write access on the mailing. |

## 17. Consistency rules

| Rule | Statement |
|---|---|
| `MKT-RULE-310` | There is no company-consistency rule inside this domain: a mailing, a list, a contact and a trace are not company-scoped. Campaigns carry a company for reporting only and are not filtered by it. |
| `MKT-RULE-311` | There is no currency handling in this domain except the display currency of the campaign revenue indicator, which is the currency of the campaign's company. |
| `MKT-RULE-312` | The per-recipient links of a message are built on the base address of the **recipient record**, in order that a multi-site installation keeps each recipient on their own site. |
| `MKT-RULE-313` | The context value naming the recipient record is only honoured when it is an actual record, never a plain value, in order that a crafted request cannot redirect the links of a message to another host. |
| `MKT-RULE-314` | An outgoing marketing message is never sent through a server that has a personal owner. |
| `MKT-RULE-315` | The trace keeps the normalised address and the plain integer keys of the outgoing message, in order that the measurement survives the deletion of the outgoing message. |
| `MKT-RULE-316` | The report entity counts an opened delivery as the status `open` only, while the mailing counts `open` and `reply` together; both definitions must be reproduced. |

## 18. Delivery reporting and status ordering

| Rule | Statement | Message |
|---|---|---|
| `MKT-RULE-320` | Marking a delivery record opened never downgrades a record already in `open` or `reply`; those two statuses are explicitly skipped. | — |
| `MKT-RULE-321` | Marking a delivery record clicked never changes its status; it only stamps the last-click moment, and it always overwrites it, so the stored moment is the latest click. | — |
| `MKT-RULE-322` | Marking a delivery record sent clears its failure type, so a successful retry leaves no stale failure code. | — |
| `MKT-RULE-323` | A text-message delivery report is ignored when the record is already in a status at least as advanced; the ignore sets are listed in [entities.md](entities.md#2811-text-message-tracker-messaging-and-activities). | — |
| `MKT-RULE-324` | A text-message delivery report reporting that messages are being processed writes the mailing back to `sending` even when it had already reached `done`. | — |
| `MKT-RULE-325` | A mailing closed by a delivery report gets the sent moment and, only when it had no previous sent moment, the statistics flag. | — |
| `MKT-RULE-326` | When a delivery report arrives from an unauthenticated provider callback, the change entries it causes are attributed to the system contact rather than to a person. | — |
| `MKT-RULE-327` | A delivery record is never deleted by a status change; it is deleted only by a retry or by the deletion of its mailing. | — |

## 19. Reproduced text irregularities

The strings below contain irregularities. They are reproduced exactly, because support procedures and
automated tests key on them, and a rebuild that tidies them changes observable behaviour.

| Rule | Where | Irregularity |
|---|---|---|
| `MKT-RULE-330` | The notice shown after adding contacts to a list | *"%s Mailing Contacts have been added. "* ends with a space. |
| `MKT-RULE-331` | The discard button of the schedule assistant | Its label *"Discard "* ends with a space. |
| `MKT-RULE-332` | The name of a comparison-test winner copy | It begins with a space: *" <original name> (final)"*. |
| `MKT-RULE-333` | The label of the failure type `twilio_authentication` | It ends with a stray double quotation mark: the stored label is exactly `Authentication Error"`. **Compatibility finding**: a corrected behaviour would display the label without the trailing mark. |
| `MKT-RULE-334` | The refusal shown by the paste import above its limit | *"You have to much emails, please upload a file."* contains a grammatical error. |
| `MKT-RULE-335` | The refusal shown by an invalid saved condition | *"The filter domain is not valid for this recipients."* contains a grammatical error. |
| `MKT-RULE-336` | The refusal shown when both membership views are supplied | *"You should give either list_ids, either subscription_ids to create new contacts."* names two stored identifiers inside the sentence and uses "either … either". |
| `MKT-RULE-337` | The refusal shown when a website form names a non-public list | *"You cannot subscribe to the following list anymore : %s"* has a space before the colon. |
| `MKT-RULE-338` | The rendering-failure message of a marketing card | *"An error occured while rendering a card for %(record_name)s. Try again or check the server logs for more details."* misspells "occurred". |

## 20. Compatibility obligations

| Rule | Statement |
|---|---|
| `MKT-RULE-340` | The older unsubscribe path, which uses a different prefix and different parameter spellings, must keep answering, because addresses of that shape are inside messages already delivered. It performs the same unsubscription as the current path. |
| `MKT-RULE-341` | The two shifted labels of the delivery status must be reproduced: the stored value `pending` is displayed as "Sent" and the stored value `sent` is displayed as "Delivered". Every formula uses the stored values. |
| `MKT-RULE-342` | The analytical view counts an opened delivery as the status `open` alone, while a mailing counts `open` and `reply` together. Both definitions must be reproduced. |
| `MKT-RULE-343` | The campaign indicators use one single denominator and derive the delivered figure by subtracting bounces from the handed-over count, while the mailing indicators use three denominators and derive the delivered figure by summing statuses. Both must be reproduced. |
| `MKT-RULE-344` | The status value written on an outgoing text message that is suppressed is spelled `canceled` with one letter L, while the delivery-record status is spelled `cancel`. Both spellings are contractual. |
| `MKT-RULE-345` | The three campaign-tracking cookies keep the names they have today, because a visitor may arrive carrying cookies written by an earlier visit. |
| `MKT-RULE-346` | The marketing card image is served under the fixed download name `card.jpg`, and both the readable-key form and the numeric-key form of the three card addresses must answer, because social networks cache whichever form they first saw. |

## 21. Rule identifier index

| Range | Topic | Section |
|---|---|---|
| `MKT-RULE-001` – `MKT-RULE-013` | Mass Mailing structure and validation | 1 |
| `MKT-RULE-020` – `MKT-RULE-026` | Editing guards | 2 |
| `MKT-RULE-030` – `MKT-RULE-039` | Sending guards | 3 |
| `MKT-RULE-050` – `MKT-RULE-071` | Audience and exclusion | 4 |
| `MKT-RULE-080` – `MKT-RULE-086` | Batching, resumability and transactions | 5 |
| `MKT-RULE-090` – `MKT-RULE-101` | Dates and moments | 6 |
| `MKT-RULE-110` – `MKT-RULE-114` | Rounding and numeric presentation | 7 |
| `MKT-RULE-120` – `MKT-RULE-130` | Uniqueness | 8 |
| `MKT-RULE-140` – `MKT-RULE-156` | Deletion and archiving guards | 9 |
| `MKT-RULE-170` – `MKT-RULE-183` | Comparison testing | 10 |
| `MKT-RULE-190` – `MKT-RULE-207` | Link tracking | 11 |
| `MKT-RULE-210` – `MKT-RULE-216` | Campaign tracking capture | 12 |
| `MKT-RULE-220` – `MKT-RULE-239` | Public pages, tokens and permissions | 13 |
| `MKT-RULE-250` – `MKT-RULE-263` | Marketing card rules | 14 |
| `MKT-RULE-270` – `MKT-RULE-285` | Mailing lists, contacts and subscriptions | 15 |
| `MKT-RULE-290` – `MKT-RULE-304` | Permissions per operation | 16 |
| `MKT-RULE-310` – `MKT-RULE-316` | Consistency rules | 17 |
| `MKT-RULE-320` – `MKT-RULE-327` | Delivery reporting and status ordering | 18 |
| `MKT-RULE-330` – `MKT-RULE-338` | Reproduced text irregularities | 19 |
| `MKT-RULE-340` – `MKT-RULE-346` | Compatibility obligations | 20 |

Numbers not listed inside a range are unused; every number listed inside a range is defined. The
ranges leave room so that a later rule can be added to a topic without renumbering the rest.
