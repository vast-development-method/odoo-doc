# Marketing configuration

Everything an administrator can set, everything the domain ships as data, and everything that
governs who may do what. Six sections: the settings screen, the system parameters, the scheduled
jobs and cleanup routines, the records shipped with the capability packages, the access groups with
their rights and record rules, and the message layouts and designs.

This domain defines no numbering sequence of its own: neither a mailing, nor a list, nor a delivery
record carries a generated reference. The only generated strings are the short code of a Link
Tracker Code and the three-character opt-out code of a text-message delivery record, both specified
in [calculations.md](calculations.md).

---

## 1. The settings screen

The settings screen of the marketing application writes six values. Two of them grant a group, one
writes a stored field of the website record, and the remaining three write system parameters. The
screen is reachable only by a settings administrator.

| Setting identifier | Label shown | Kind | Where the value is kept | Effect |
|---|---|---|---|---|
| `group_mass_mailing_campaign` | Mailing Campaigns | switch | Grants or revokes the campaign-management group for every internal user | Shows the campaign field on a mailing, the campaign menu, the Campaign Stages menu and the Campaign Tags menu; also switches the comparison-test scheduled job on and off. Help text: "This is useful if your marketing campaigns are composed of several emails". |
| `mass_mailing_outgoing_mail_server` | Dedicated Server | switch | System parameter `mass_mailing.outgoing_mail_server` | Reveals the mail-server choice on every mailing and the server chooser on the settings screen. Help text: "Use a specific mail server in priority. Otherwise the system relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails." |
| `mass_mailing_mail_server_id` | Mail Server | link to an Outgoing Mail Server | System parameter `mass_mailing.mail_server_id`, holding the numeric key | The server used as the default of every newly created mailing. Cleared, and the parameter emptied, whenever the switch above is turned off. |
| `show_blacklist_buttons` | Blacklist Option when Unsubscribing | switch | System parameter `mass_mailing.show_blacklist_buttons` | Shows the "Exclude Me" and "Come Back" buttons on the public subscription-management page. Help text: "Allow the recipient to manage themselves their state in the blacklist via the unsubscription page." |
| `mass_mailing_reports` | 24H Stat Mailing Reports | switch | System parameter `mass_mailing.mass_mailing_reports` | Enables the statistics message sent to the responsible person one day after a mailing finishes. Help text: "Check how well your mailing is doing a day after it has been sent." |
| `mass_mailing_split_contact_name` | Split First and Last Name | switch | Activates or deactivates two alternative Mailing Contact screens | Splits the contact name into a first name and a last name. Reading the settings reports the current value by inspecting whether the alternative list screen is active. |

With the Checkout Newsletter package the screen gains two more controls, both scoped to the website
currently selected on the screen:

| Setting identifier | Label shown | Kind | Where the value is kept | Effect |
|---|---|---|---|---|
| `is_newsletter_enabled` | Newsletter | switch | Activates or deactivates the checkout newsletter block of the selected website | Offers the newsletter tick box on the online-shop checkout. Recomputed whenever the selected website changes, so the tick box always shows the state of the website on screen. |
| `newsletter_id` | Newsletter List | link to a Mailing List | Field `newsletter_id` on the Website record | The list a shopper is subscribed to when they tick the box. |

**Saving behaviour.** Saving the screen performs three extra actions after writing the values: the
comparison-test scheduled job is activated when the campaign switch is on and deactivated when it is
off; the two split-name screens are activated or deactivated to match the split-name switch; and,
when the dedicated-server switch has just been turned off, the stored server key is emptied so that
no mailing keeps pointing at a server the operator no longer intends to use.

**On-change behaviour.** Turning the dedicated-server switch off on the screen immediately clears
the server chooser, before the screen is saved, so that the operator sees the consequence at once.

---

## 2. System parameters

| Parameter | Default | Read by | Meaning |
|---|---|---|---|
| `mass_mailing.mass_mailing_reports` | `True`, written when the Email Marketing package is installed and never overwritten afterwards | The queue job, at the end of every run | Whether the statistics message is produced one day after a mailing finishes. |
| `mass_mailing.show_blacklist_buttons` | `True`, written at installation and never overwritten afterwards | The public subscription-management page | Whether the self-exclusion and re-inclusion buttons are offered. |
| `mass_mailing.outgoing_mail_server` | absent, which means false | Every mailing form, to decide whether the server chooser is visible | Whether a dedicated outgoing mail server is in use. |
| `mass_mailing.mail_server_id` | absent | The default of the mail-server field of a new mailing, and the owner guard on a mail server | The numeric key of the dedicated server. |
| `mass_mailing.cancelled_mails_months_limit` | 6 | The cleanup routine for cancelled outgoing mail | Age in months beyond which a cancelled outgoing mail is deleted. A value of zero or less disables the cleanup. |
| `marketing_card.card_image_cleanup_interval_days` | 60 | The card cleanup routine | Age in days beyond which a marketing card, archived ones included, is deleted. A value of zero or an empty value disables the cleanup. |
| `link_tracker.no_external_tracking` | absent, which means false | The redirection-address computation of a Link Tracker | When set, campaign parameters are not appended to a target address whose host differs from the site host. |
| `mail.batch_size` | 50 when absent, and 50 when it evaluates to zero | The batch loop of the sending algorithm | Number of recipients prepared, created and committed in one slice. Owned by the messaging domain and shared with it. |
| `web.base.url` | the installation's own base address | Link shortening, per-recipient links, marketing card addresses | The address used when no website resolves. Owned by the platform. |
| `database.secret` | generated at database creation | The recipient token of every public subscription endpoint | The key of the keyed hash. Owned by the platform; see [../../runtime/sessions-and-authentication.md](../../runtime/sessions-and-authentication.md). |

---

## 3. Scheduled jobs and cleanup routines

### 3.1 The queue job

| Property | Value |
|---|---|
| Name | *"Mail Marketing: Process queue"* |
| Acts on | Mass Mailing |
| Runs as | the system user |
| Interval | every 1 day |
| Priority | 6 |
| Active when installed | yes |
| What it does | Selects every mailing whose state is `in_queue` or `sending` and whose schedule date is empty or already past, and processes each one; then, when `mass_mailing.mass_mailing_reports` is on, sends the statistics messages that are due. The full procedure is in [workflows.md](workflows.md#6-queue-processing). |
| Woken early by | Putting a mailing in the queue, which requests a wake at the mailing's schedule date or, when it has none, at the current moment. Retrying failed deliveries therefore also wakes it, because retrying queues the mailing again. |
| Progress reporting | The number of selected mailings is reported as the remaining work before the loop; one unit of progress is reported after each mailing, which is also where the runner commits. |

### 3.2 The comparison-test job

| Property | Value |
|---|---|
| Name | *"Mail Marketing: A/B Testing"* |
| Acts on | Campaign |
| Runs as | the system user |
| Interval | every 1 day |
| Active when installed | **no**. It is activated by the campaign switch of the settings screen and deactivated when that switch is turned off. |
| What it does | Selects every campaign whose comparison-test moment has passed, whose winner criterion is not manual selection and which is not already completed, and runs the winner operation for the electronic-mail versions and then, separately, for the text-message versions with the text-message criterion. A campaign none of whose versions has reached the state `done` is skipped silently. |
| Woken early by | Creating or writing a comparison-test moment on a mailing, which requests a wake at the earliest such moment among the written records. |

### 3.3 Cleanup routines

Both routines run under the platform's periodic cleanup job rather than under a job of their own.

| Routine | What it deletes | Bound |
|---|---|---|
| Cancelled outgoing mail cleanup | Outgoing mails whose state is the cancelled state, that belong to a mailing, and whose last write is older than `mass_mailing.cancelled_mails_months_limit` months | Deletes in pages of at most 10 000 records ordered by ascending key. Disabled when the parameter is zero or less. |
| Marketing card cleanup | Marketing Cards, archived ones included, whose last write is older than `marketing_card.card_image_cleanup_interval_days` days | Social networks are expected to have cached the images by then. Disabled when the parameter is zero or empty. |

---

## 4. Records shipped with the capability packages

### 4.1 Opt-out reasons

Five Mailing Opt-Out Reason records, offered as radio buttons on the public pages, in this order.

| Sequence | Name | Asks for free text |
|---|---|---|
| 1 | *"I never subscribed to this list"* | no |
| 2 | *"I changed my mind"* | no |
| 3 | *"I receive too many emails from this list"* | no |
| 4 | *"The content of these emails is not relevant to me"* | no |
| 99 | *"Other"* | yes |

### 4.2 Campaign Mediums

Eleven Campaign Medium records. The first ten are shipped by the campaign-tracking package, the last
by the Text Message Marketing package.

| Name | Protected against deletion |
|---|---|
| `Website` | yes |
| `Phone` | no |
| `Direct` | yes |
| `Email` | yes |
| `Banner` | no |
| `X` | yes |
| `Facebook` | yes |
| `LinkedIn` | yes |
| `Television` | no |
| `Google Adwords` | no |
| `Text Message` | yes, with its own message |

The protection rules and their messages are rules `MKT-RULE-141` to `MKT-RULE-144` of
[business-rules.md](business-rules.md).

### 4.3 Campaign Sources

Eleven Campaign Source records: `Search engine`, `Lead Recall`, `Newsletter`, `Facebook`, `X`,
`LinkedIn`, `Monster`, `Glassdoor`, `Craigslist`, `Referral` and, with the Marketing Card package,
`Marketing Card`. `Referral` and `Marketing Card` are protected against deletion; the other nine are
not.

### 4.4 Campaign Stages and Campaign Tags

One Campaign Stage is shipped, named `New`, with sequence 10; it is the default stage of every new
campaign because the default is the first stage by sequence. One Campaign Tag is shipped, named
`Marketing`, with colour index 1.

### 4.5 Mailing List, Mailing Contact and Mailing Subscription

One Mailing List named `Newsletter` is shipped with the public flag set, so that it appears on the
subscription-management page and can be chosen as the newsletter list of a website. One Mailing
Contact is shipped carrying the name and address of the administrator user, and one Mailing
Subscription joins that contact to that list, opted in.

### 4.6 Marketing Card Templates

Fourteen Marketing Card Template records are shipped. Each carries a name, an optional background
image and four colours. When a template does not state a colour it takes the field default.

| Name | Background image | Primary colour | Secondary colour | Primary text colour | Secondary text colour |
|---|---|---|---|---|---|
| `Light` | none | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Dark` | none | `#161315` | `#dedede` | `#ffffff` | `#161315` |
| `Center` | a portrait background | `#161315` | `#dedede` | `#ffffff` | `#161315` |
| `World Map` | a world map | `#161315` | `#dedede` | `#ffffff` | `#161315` |
| `Lila` | a lilac pattern | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Safari` | a desert photograph | `#161315` | `#dedede` | `#ffffff` | `#161315` |
| `Waves` | a wave pattern | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Lines` | a line pattern | `#161315` | `#dedede` | `#ffffff` | `#161315` |
| `Organic` | an organic pattern | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Blur` | a blurred background | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Geometric` | a geometric pattern | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Circles` | a circle pattern | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Avatar Highlight` | a highlight background | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |
| `Drawings` | a drawing pattern | `#f9f9f9` | `#000000` | `#000000` | `#ffffff` |

The first template by key is the default design of a new Marketing Card Campaign.

### 4.7 Shipped body designs

Three body designs are shipped with the Email Marketing package and eleven more with the Mass
Mailing Themes package. Choosing one fills the editable body of the mailing; it is a starting point,
not a stored link, so a later change of a design never changes a mailing already written.

| Design | Package | Character |
|---|---|---|
| `theme_empty_template` | Email Marketing | An empty body with no structure at all. |
| `theme_basic_template` | Email Marketing | A plain body with a single text block. |
| `theme_default_template` | Email Marketing | The default themed body: header, cover, text, call to action and footer. |
| `theme_newsletter_template` | Mass Mailing Themes | A newsletter layout. |
| `theme_bignews_template` | Mass Mailing Themes | A single large announcement. |
| `theme_blogging_template` | Mass Mailing Themes | An article layout. |
| `theme_coffeebreak_template` | Mass Mailing Themes | A short informal layout. |
| `theme_coupon_template` | Mass Mailing Themes | A promotional-code layout. |
| `theme_event_template` | Mass Mailing Themes | An event announcement. |
| `theme_magazine_template` | Mass Mailing Themes | A multi-article layout. |
| `theme_promotion_template` | Mass Mailing Themes | A promotion layout. |
| `theme_roadshow1_template` | Mass Mailing Themes | A first travelling-event layout. |
| `theme_roadshow2_template` | Mass Mailing Themes | A second travelling-event layout. |
| `theme_training_template` | Mass Mailing Themes | A course announcement. |

### 4.8 Shipped images

Thirty-one public image files are shipped with the Email Marketing package as the default pictures
of the building blocks: a cover image, three media-list images, six team-member images, four
reference images, three product images, a quotation image, an image-and-text image, two event
images, two masonry-block images, a picture-block image and a text-and-image image. Each is a public
attachment whose content is a stored address rather than binary data, so that a body referring to it
stays small.

### 4.9 The guided tour

One guided tour is shipped with the Email Marketing package, sequence 200, which walks a new user
through creating and sending a first mailing and ends with the celebration message
*"Congratulations, I love your first mailing. :)"*.

---

## 5. Groups, access rights and record rules

### 5.1 Privileges and groups

Two privileges group the marketing roles inside the access-rights screen, both under the marketing
application category.

| Privilege | Sequence | Groups it contains |
|---|---|---|
| Email Marketing | 19 | User |
| Marketing Card | 100 | Marketing Card User, Marketing Card Manager |

| Group | Label | Implies | Granted at installation to |
|---|---|---|---|
| Email Marketing User | `User` | the internal-user group | the system user and the administrator |
| Campaign management | `Manage Mass Mailing Campaigns` | nothing | nobody; granted by the campaign switch of the settings screen |
| Marketing Card User | `Marketing Card User` | the internal-user group and Email Marketing User | nobody |
| Marketing Card Manager | `Marketing Card Manager` | Marketing Card User | the system user and the administrator |

The campaign-management group is not offered as a role in the access-rights screen: it is granted
and revoked only by the settings switch, for every internal user at once.

### 5.2 Access rights

Read, write, create and delete rights per entity and per group. A blank cell means the right is not
granted by that line.

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Link Tracker | internal user | yes | | | |
| Link Tracker | public user | | | | |
| Link Tracker | settings administrator | yes | yes | yes | yes |
| Link Tracker | Email Marketing User | yes | yes | yes | yes |
| Link Tracker | website designer | yes | yes | yes | yes |
| Link Tracker Code | internal user | yes | | | |
| Link Tracker Code | public user | | | | |
| Link Tracker Code | settings administrator | yes | yes | yes | yes |
| Link Tracker Code | website designer | yes | yes | yes | yes |
| Link Tracker Click | internal user | yes | | | |
| Link Tracker Click | public user | | | | |
| Link Tracker Click | settings administrator | yes | yes | yes | yes |
| Link Tracker Click | website designer | yes | yes | yes | yes |
| Mass Mailing | Email Marketing User | yes | yes | yes | yes |
| Mass Mailing | settings administrator | yes | yes | yes | yes |
| Mailing List | Email Marketing User | yes | yes | yes | yes |
| Mailing List | website designer | yes | | | |
| Mailing Contact | Email Marketing User | yes | yes | yes | yes |
| Mailing Subscription | Email Marketing User | yes | yes | yes | yes |
| Mailing Opt-Out Reason | Email Marketing User | yes | yes | yes | yes |
| Mailing Trace | Email Marketing User | yes | yes | yes | yes |
| Mailing Trace Report | Email Marketing User | yes | | | |
| Mailing Filter | Email Marketing User | yes | yes | yes | yes |
| Mailing Contact Import Wizard | Email Marketing User | yes | yes | yes | yes |
| Mailing Contact to List Wizard | Email Marketing User | yes | yes | yes | yes |
| Mailing List Merge Wizard | Email Marketing User | yes | yes | yes | |
| Mailing Test Wizard | Email Marketing User | yes | yes | yes | |
| Mailing Schedule Wizard | Email Marketing User | yes | yes | yes | yes |
| Test Text Message Mailing Wizard | Email Marketing User | yes | yes | yes | |
| Campaign | internal user | yes | yes | yes | |
| Campaign | settings administrator | yes | yes | yes | yes |
| Campaign | Email Marketing User | yes | yes | yes | yes |
| Campaign Source | internal user | yes | yes | yes | |
| Campaign Source | settings administrator | yes | yes | yes | yes |
| Campaign Source | Email Marketing User | yes | yes | yes | yes |
| Campaign Medium | internal user | yes | yes | yes | |
| Campaign Medium | settings administrator | yes | yes | yes | yes |
| Campaign Medium | Email Marketing User | yes | yes | yes | yes |
| Campaign Stage | internal user | yes | | | |
| Campaign Stage | settings administrator | yes | yes | yes | yes |
| Campaign Stage | Email Marketing User | yes | yes | yes | yes |
| Campaign Tag | internal user | yes | | | |
| Campaign Tag | settings administrator | yes | yes | yes | yes |
| Campaign Tag | campaign management | yes | yes | yes | yes |
| Marketing Card Campaign | Marketing Card User | yes | yes | yes | yes |
| Marketing Card Campaign Tag | Marketing Card User | yes | | | |
| Marketing Card Campaign Tag | Marketing Card Manager | yes | yes | yes | yes |
| Marketing Card Template | Marketing Card User | yes | | | |
| Marketing Card Template | settings administrator | yes | yes | yes | yes |
| Marketing Card | internal user | yes | yes | yes | |
| Marketing Card | portal user | yes | | | |
| Marketing Card | public user | yes | | | |
| Marketing Card | Marketing Card Manager | yes | yes | yes | yes |
| Outgoing Mail Server | Email Marketing User | yes | | | |
| Model Definition | Email Marketing User | yes | | | |
| Email Blacklist | Email Marketing User | yes | yes | yes | yes |
| Email Blacklist removal assistant | Email Marketing User | yes | yes | yes | yes |
| Phone Blacklist removal assistant | Email Marketing User | yes | yes | yes | yes |
| Text Message Tracker | Email Marketing User | yes | yes | yes | yes |
| Property definition | Email Marketing User | yes | yes | yes | yes |

Two consequences are worth stating because a rebuild that misses them changes observable behaviour.
First, every internal user may create a Campaign, a Campaign Source and a Campaign Medium but not
delete one; deletion needs a settings administrator or an Email Marketing User. Second, anonymous
visitors may read a Marketing Card, which is what makes the public card image and preview pages work
without a session, but they may read nothing else in this domain.

### 5.3 Record rules

| Rule | Entity | Applies to | Condition | Rights it governs |
|---|---|---|---|---|
| Manager may access and edit any card campaign | Marketing Card Campaign | Marketing Card Manager | always true | read, write, create, delete |
| Users may only edit their own card campaigns | Marketing Card Campaign | Marketing Card User | the responsible user is the acting user | write and delete only; reading and creating are unrestricted |
| Mailing contact property definition | Property definition | Email Marketing User | the definition is the one of the user-defined fields of Mailing Contact | read, write, create, delete |

The second rule is deliberately narrow: a Marketing Card User may read every campaign, including
other people's, and may create campaigns, but may change or delete only campaigns whose responsible
person is themselves. The third rule is what lets a marketing user define the user-defined fields of
Mailing Contact without giving them access to the property definitions of any other entity.

### 5.4 Field-level restrictions

| Field | Entity | Visible to |
|---|---|---|
| `campaign_id` | Mass Mailing | the campaign-management group only, on the form and in the search panel |
| `mailing_mail_ids`, `mailing_mail_count`, `mailing_sms_ids`, `mailing_sms_count` | Campaign | the Email Marketing User group only |
| `crm_lead_count` | Campaign | the salesperson group only |
| `quotation_count`, `invoiced_amount` | Campaign | the salesperson group only |
| `medium_id`, `source_id` and the advanced fields | Mass Mailing | the technical-features mode only; the source is always read-only |

---

## 6. Layouts, designs and message assembly

This domain ships no message template in the sense of a stored, editable template record. Every
message it produces is assembled from a layout at send time.

| Layout | Used by | Contents |
|---|---|---|
| Marketing mail layout | Every outgoing marketing message, every test message and every browser view | Wraps the rendered body together with the marketing stylesheet. It is what makes a marketing message render identically in the inbox and in the browser view. |
| Comparison-test description | The mailing form | Renders the sentence that describes the running test: the number of versions, the total percentage of the audience covered, the winner criterion and the decision moment. |
| Mobile preview frame | The mobile preview endpoint | Renders the body inside a telephone-shaped frame. |
| Subscription-management page | The public unsubscribe and preferences pages | The list of subscriptions, the suggested public lists, the feedback form and the exclusion buttons. |
| Unsubscribe confirmation pages | The confirmation endpoints | The "are you sure" page and the "successfully unsubscribed" page. |
| Browser view page | The view-in-browser endpoint | The rendered body of the mailing for one recipient. |
| Statistics message body | The queue job | The engagement block, the business block, the link-tracker table and one random tip; assembled in [workflows.md](workflows.md#20-open-tracking-view-in-browser-and-the-statistics-message). |
| Marketing card preview page | The card preview endpoint | The request title and description, the image, the post suggestion and the share buttons. |
| Marketing card crawler page | The card redirection endpoint | Only the social-preview metadata: image address, post text and target name. |
| Marketing card robots instruction | The card endpoints | Tells well-behaved crawlers which card addresses they may fetch. |
| Marketing card designs | The card rasteriser | Five shipped card layouts that the fourteen templates draw on. |

The statistics message additionally contributes one block to the periodic digest message of the
platform: a link that turns the statistics messages off, carrying the deactivation token and the
user key, shown only when a token was produced.

---

## 7. What an administrator must do to make the domain work

1. Grant the Email Marketing User group to the people who will write mailings.
2. Decide whether campaigns are used, and switch the campaign setting accordingly; this also decides
   whether the comparison-test job runs.
3. Decide whether a dedicated outgoing mail server is used. If it is, choose it; from then on a
   mailing never uses a server that has a personal owner, and a server that is the dedicated server
   can no longer be given a personal owner.
4. Decide whether the self-exclusion buttons appear on the public page, and whether the statistics
   message is sent.
5. Create at least one Mailing List, or make sure the shipped `Newsletter` list is the one wanted,
   and mark it public if visitors must be able to join it.
6. Review the five shipped opt-out reasons and add any reason the organisation needs; a reason that
   asks for free text reveals a text box on the public page.
7. For marketing cards: grant the Marketing Card User group, and, for a person who must manage other
   people's campaigns, the Marketing Card Manager group.
8. For the website subscription block: confirm that the human-verification service is configured,
   because the subscription endpoint refuses a submission whose verification token fails.

Nothing else is required. The queue job, the cleanup routines, the shipped mediums, sources, stage,
tag and card templates are installed and active without intervention.
