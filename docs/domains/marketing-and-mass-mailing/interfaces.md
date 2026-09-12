# Marketing interfaces

Everything through which a person, another system or a scheduled process reaches this domain: the
menu tree, the screens, the named operations offered on each entity, the public and internal
endpoints, the documents and messages produced, the import and export paths, and the external
services contacted.

Conventions: a route path, a stored value or an identifier is reproduced in code font; a verbatim
label or message is reproduced between quotation marks; an endpoint marked *public* answers without
a session, an endpoint marked *signed in* refuses an anonymous caller.

---

## 1. The menu tree

Two application menus belong to this domain, plus two technical menus and one entry placed on the
website editor.

### 1.1 Email Marketing

| Level | Menu | Opens | Visible to |
|---|---|---|---|
| 1 | Email Marketing (sequence 115) | — | Email Marketing User |
| 2 | Mailings (sequence 1) | The mailing screens, restricted to the electronic-mail type, with "assigned to me" pre-selected and the responsible defaulted to the acting user | Email Marketing User |
| 2 | Mailing Lists (sequence 2) | — | Email Marketing User |
| 3 | Mailing Lists (sequence 1) | The Mailing List screens | Email Marketing User |
| 3 | Mailing List Contacts (sequence 2) | The Mailing Contact screens, with the "address not blocked" filter pre-selected | Email Marketing User |
| 2 | Campaigns (sequence 3) | The campaign screens, excluding automatically generated campaigns | campaign management |
| 2 | Reporting (sequence 90) | — | Email Marketing User |
| 3 | Mass Mailing Analysis (sequence 1) | The analytical view, restricted to the electronic-mail type | Email Marketing User |
| 3 | Opt-Out Report (sequence 2) | The subscriptions that are opted out, grouped by reason | Email Marketing User |
| 2 | Configuration (sequence 100) | — | Email Marketing User |
| 3 | Settings (sequence 0) | The settings screen | settings administrator |
| 3 | Campaign Stages (sequence 1) | The Campaign Stage screens | campaign management |
| 3 | Campaign Tags (sequence 2) | The Campaign Tag screens | campaign management |
| 3 | Link Tracker (sequence 10) | The Link Tracker screens | Email Marketing User |
| 3 | Blacklisted Email Addresses (sequence 20) | The blocked-address register of the messaging domain | Email Marketing User |
| 3 | Optout Reasons (sequence 21) | The Mailing Opt-Out Reason screens | Email Marketing User |
| 3 | Favorite Filters (sequence 30) | The Mailing Filter screens, with "saved by me" pre-selected | Email Marketing User |

### 1.2 Text Message Marketing

| Level | Menu | Opens | Visible to |
|---|---|---|---|
| 1 | Text Message Marketing (sequence 120) | — | Email Marketing User |
| 2 | Text Message Marketing (sequence 1) | The mailing screens restricted to the text-message type, with the text-message context switched on | Email Marketing User |
| 2 | Mailing Lists (sequence 2) | — | Email Marketing User |
| 3 | Mailing Lists (sequence 1) | The Mailing List screens in the text-message context, which shows the number-based counters | Email Marketing User |
| 3 | Mailing List Contacts (sequence 2) | The Mailing Contact list in the text-message context, with the "number not blocked" filter pre-selected | Email Marketing User |
| 2 | Campaigns (sequence 5) | The same campaign screens as the electronic-mail application | campaign management |
| 2 | Reporting (sequence 80) | The analytical view restricted to the text-message type | Email Marketing User |
| 2 | Configuration (sequence 100) | — | Email Marketing User |
| 3 | Blacklisted Phone Numbers (sequence 1) | The blocked-number register of the messaging domain | Email Marketing User |
| 3 | Link Tracker (sequence 2) | The Link Tracker screens | Email Marketing User |

### 1.3 Marketing Card

| Level | Menu | Opens | Visible to |
|---|---|---|---|
| 1 | Marketing Card (sequence 270) | — | Marketing Card User |
| 2 | Campaigns (sequence 0) | The Marketing Card Campaign screens | Marketing Card User |

### 1.4 Technical menus

| Menu | Parent | Opens | Visible to |
|---|---|---|---|
| Link Tracker (sequence 270) | application root | — | the technical-features group |
| Campaign tracking (sequence 99) | Link Tracker | — | the technical-features group |
| Campaigns (sequence 1) | Campaign tracking | Every campaign, including automatically generated ones | the technical-features group |
| Mediums (sequence 5) | Campaign tracking | The Campaign Medium screens | the technical-features group |
| Sources (sequence 10) | Campaign tracking | The Campaign Source screens | the technical-features group |
| Link Tracker | Link Tracker root | The Link Tracker screens | the technical-features group |
| Mass Mailing (sequence 4) | the platform's technical menu | — | the technical-features group |
| Mailing Traces (sequence 2) | Mass Mailing technical menu | The delivery-record screens | the technical-features group |
| Marketing Card (sequence 4) | the platform's technical menu | — | Marketing Card Manager and the technical-features group |
| Card Template | Marketing Card technical menu | The Marketing Card Template screens | Marketing Card Manager and the technical-features group |
| Link Tracker (sequence 25) | the website editor's current-page menu | The shortened-link creation page | website designer |

---

## 2. Screens

Screens are described by what they let a person do; the exact widget arrangement is not part of the
contract, but the presence of a control, its visibility condition and its read-only condition are.

### 2.1 Mass Mailing

| Screen | Purpose | Notable controls and behaviour |
|---|---|---|
| Board | The working view, grouped by state with all four columns always shown | A card shows the subject, the responsible, the schedule or sent moment and the five indicators. Empty state groups are displayed. |
| List | Bulk review | Shows subject, responsible, state, sent date and the indicators. The schedule date stays editable while the state is `draft` or `in_queue`. |
| Calendar | Planning | Positions each mailing on its `calendar_date`. Creating from a future day pre-fills the schedule type as `scheduled` and the schedule date as that day. |
| Form | Design and send | Header buttons: Send, Schedule, Test, Cancel, Retry, Duplicate, Create an Alternative Version, Compare Version, Send this as winner, Update cards, Add to Templates, Remove from Templates. Statistic buttons open the filtered delivery records and the filtered recipient documents. |
| Form, Mail Body page | The designer | Design gallery, building blocks, mobile preview, code view and the dynamic-placeholder helper. |
| Form, Settings page | Sender and audience settings | Sender address, answer mode and address, attachments, campaign, responsible, archive-keeping flag, mail server, exclusion-list flag. |
| Form, comparison-test page | The test settings | Enable switch, percentage, winner criterion, decision moment, and the rendered description sentence. |
| Search panel | Finding a mailing | Search on subject and campaign; filters *"My Mailings"*, *"Sent Period"*, *"A/B Tests"*, *"A/B Tests to review"* and *"Archived"*; grouping by state, by sender and by mailing list. |

The read-only conditions of every field on the form are rules `MKT-RULE-020` to `MKT-RULE-026` of
[business-rules.md](business-rules.md).

### 2.2 The other entities

| Entity | Screens | Notable behaviour |
|---|---|---|
| Mailing List | board, list, form, simplified form | The board card shows the four quality counters and the three percentages. The simplified form is used when a list is created from inside another screen. |
| Mailing Contact | list, board, form, chart, matrix, plus a split-name list and a split-name form activated by the setting | The list shows the opt-out column only when it was opened from one list. |
| Mailing Subscription | list, form, chart, matrix, search | The opt-out report groups by reason. |
| Mailing Opt-Out Reason | list, form, search | Ordered by sequence. |
| Mailing Trace | list, list for electronic mail, list for text messages, form, form for text messages, chart, search | Reachable from the technical menu and from every statistic button of a mailing. |
| Mailing Trace Report | chart, matrix, list | One action per channel. |
| Mailing Filter | list, form, search | The default filter is *"saved by me"*. |
| Link Tracker | list, form, chart, search | The form offers "Visit Page" and the statistics button. Grouping by campaign, medium and source. |
| Link Tracker Click | list, form, chart, search | The statistics action groups by country by default. |
| Campaign | board, list, form, quick-create form, search | The board shows every stage as a column. The form carries the statistic buttons for mailings, text-message mailings, clicks, leads, quotations and revenue. |
| Campaign Source, Campaign Medium, Campaign Stage, Campaign Tag | list and form each; the medium also has a search panel | Reached from the technical menus and from the marketing configuration menu. |
| Marketing Card Campaign | board, list, form, search | The form has a Card Layout page and a Recipient Message page, and header buttons Share, Preview, and statistic buttons for mailings, cards, clicked cards and shared cards. Search filters: *"my campaigns"*, *"archived"*, grouping by responsible and by tag. |
| Marketing Card | list, search | Filters *"shared"* and *"visited"*, grouping by campaign. The default action pre-selects grouping by campaign and the visited filter. |
| Marketing Card Template | form | Settings administrator only. |
| Email Blacklist | list, form, search | Extended by this domain with the opt-out reason. |

### 2.3 Assistants

| Assistant | Opened from | Buttons |
|---|---|---|
| Mailing Test Wizard | the Test button of a mailing | *"Send test"* and *"Cancel"* |
| Test Text Message Mailing Wizard | the Test button of a text-message mailing | Send and cancel |
| Mailing Schedule Wizard | the Schedule button, titled *"When do you want to send your mailing?"* | *"Schedule"* and *"Discard "* — the discard label carries a trailing space, which is reproduced |
| Mailing Contact Import Wizard | the Import button of a list or of the contact screen, titled *"Import Mailing Contacts"* | *"Import"*, *"Discard"* and a button that opens the general file-import assistant |
| Mailing Contact to List Wizard | the contact list selection, titled *"Add Selected Contacts to a Mailing List"* | *"Add"*, *"Add and Send Mailing"* and *"Cancel"* |
| Mailing List Merge Wizard | the list selection, titled *"Merge"*, bound to the list and board views | *"Merge"* and *"Cancel"* |

---

## 3. Named operations

Operations are grouped by the entity they are invoked on. Each one states what it does and what it
returns; the procedures are in [workflows.md](workflows.md).

### 3.1 On a Mass Mailing

| Operation | Effect |
|---|---|
| Send | Writes the schedule type to `now` and queues the mailing. |
| Schedule | Queues directly when a future schedule date is already stored, otherwise opens the schedule assistant. |
| Cancel | Returns the mailing to `draft` and clears the schedule. |
| Test | Opens the test assistant, or the text-message test assistant for a text-message mailing. |
| Retry | Deletes the failed outgoing messages and their delivery records, then queues the mailing again. |
| Retry text messages | The same for outgoing text messages in error. |
| Duplicate | Creates a copy in `draft` and opens it. |
| Reload | Refreshes the form; used by the "will be sent as soon as possible" notice. |
| Add to Templates / Remove from Templates | Sets or clears the favorite mark and its moment, and shows the confirmation notice. |
| Fetch favorite designs | Returns the favorite designs for the gallery, in the order of [calculations.md](calculations.md#16-ordering-rules). |
| Compare Version | Opens the list of the campaign's comparison-test versions of the same mailing type. |
| Send this as winner | Selects and sends the winning version. |
| Select as winner | Marks this version as the winner and opens the resulting mailing. |
| Update cards | Produces or refreshes the marketing cards of every draft mailing of the campaign. |
| View delivery records | Six operations open the delivery records filtered on scheduled, cancelled, failed, processing, sent and, for text messages, the equivalent filters. |
| View documents | Five operations open the recipient records that opened, clicked, replied, bounced or were delivered. |
| View mailing contacts | Opens the Mailing Contacts that belong to the lists selected on this mailing. |
| View link trackers | Opens the Link Tracker records created for this mailing. |
| Redirect to leads and opportunities | Opens the leads attributed to this mailing's source. |
| Redirect to quotations, redirect to invoiced | Open the sales orders and the customer invoices attributed to this mailing's source. |
| Buy text-message credit | Opens the external purchase page of the sending service. |
| Text-message length placeholders | Returns the two placeholder strings that let the designer estimate the length of a text message; requires write access on the mailing. |

### 3.2 On the other entities

| Entity | Operation | Effect |
|---|---|---|
| Mailing List | Open import | Opens the paste-import assistant with this list pre-selected. |
| Mailing List | Send mailing | Opens a new mailing addressed to this list. |
| Mailing List | Send text message | The same for the text-message channel. |
| Mailing List | View contacts, view contacts with an address, view opted-out, view blocked, view bouncing, view contacts with a number | Six operations opening the Mailing Contacts of the list filtered accordingly. |
| Mailing List | View mailings | Opens the mailings that address this list. |
| Mailing List | Merge | Inserts the qualifying contacts of the source lists into the destination and optionally archives the sources. |
| Mailing Contact | Import | Opens the paste-import assistant. |
| Mailing Contact | Add to mailing list | Opens the add-to-list assistant with the selection. |
| Mailing Contact | Add to list by name | Creates a contact from a typed string and links it to a list. |
| Mailing Subscription | Open mailing contact | Opens the contacts behind the selected subscriptions. |
| Mailing Trace | View contact | Opens the recipient record of the delivery record. |
| Mailing Trace | Seven status operations | Mark a set of delivery records sent, opened, clicked, replied, bounced, failed or cancelled; specified in [entities.md](entities.md#66-status-operations). |
| Link Tracker | Visit page | Opens the target address. |
| Link Tracker | View statistics | Opens the recorded visits of the tracker. |
| Link Tracker | Visit statistics page | Opens the public statistics page of the short code. |
| Link Tracker | Recent links | Returns the trackers in one of three orders. |
| Link Tracker | Find or create | Returns one tracker per requested value set, reusing existing ones. |
| Link Tracker | Address from code | Returns the redirection address of a code. |
| Campaign | Create mass mailing, create text-message mailing | Open a new mailing already attributed to the campaign. |
| Campaign | Redirect to mailings, to text-message mailings, to leads, to quotations, to invoiced | Open the attributed records. |
| Campaign | Click statistics | Opens the recorded visits attributed to the campaign. |
| Campaign | Recipients of the campaign | Returns the recipient records already addressed by the campaign, per recipient entity. |
| Campaign Medium | Fetch or create by name | Returns the medium registered under a normalised external key, creating it when missing. |
| Campaign Tracking Mixin | Find or create by name | Returns the campaign, source or medium of a given name, creating it when missing. |
| Marketing Card Campaign | Preview | Creates or refreshes the card of the preview record and opens its page. |
| Marketing Card Campaign | Share | Opens a new mailing pre-filled with the campaign and a default body. |
| Marketing Card Campaign | View cards, view clicked, view shared, view mailings | Open the filtered cards and the mailings. |
| Marketing Card Campaign | Refresh cards | Produces or refreshes the cards matching a condition, in batches of 100. |
| Model Definition | Mailing-enabled search | Resolves a search on the mailing-enabled flag by listing the entities that declare it. |

---

## 4. Endpoints

### 4.1 Click tracking and redirection

| Path | Method | Access | Behaviour |
|---|---|---|---|
| `/r/<code>` | read | public | Records a visit unless the caller is a recognised automated agent, then redirects permanently to the redirection address of the code. An unknown code answers not found. |
| `/r/<code>/m/<trace key>` | read | public | The same, plus the named delivery record; the visit is always recorded, and the delivery record is marked opened and clicked. |
| `/r/<code>/s/<outgoing text message key>` | read | public | The same, resolving the delivery record from the plain integer text-message key; automated agents are skipped. |
| `/r/<code>+` | read | signed in | Renders the statistics page of the tracker behind the code; an unknown code redirects permanently to the site root. |
| `/r` | read | signed in | Renders the shortened-link creation page and tells it whether the caller may create trackers and codes. |
| `/website_links/new` | structured data | signed in | Creates or reuses a tracker. An empty address answers `{"error": "empty_url"}`. |
| `/website_links/add_code` | structured data | signed in | Adds a further code to an existing tracker when that code does not already point at it. |
| `/website_links/recent_links` | structured data | signed in | Returns the trackers in one of the three orders; any other order answers *"This filter doesn't exist."* |

### 4.2 Subscription management

| Path | Method | Access | Behaviour |
|---|---|---|---|
| `/mailing/<mailing key>/unsubscribe` | read | public | Unsubscribes and renders the subscription-management page. |
| `/mailing/<mailing key>/unsubscribe_oneclick` | write | public, exempt from the cross-site request token | Performs the same unsubscription and answers an empty success. This is the address advertised in the unsubscribe header. |
| `/mailing/<mailing key>/confirm_unsubscribe` | read | public | Renders the confirmation page instead of acting. |
| `/mailing/confirm_unsubscribe` | write | public | Performs the unsubscription from the confirmation page and renders *"Successfully unsubscribed!"* |
| `/mailing/my` | read | signed in | Renders the subscription-management page for the signed-in user's own address, with the feedback form disabled. |
| `/mailing/list/update` | structured data | public | Applies the checkbox changes; answers the number of lists opted out of. |
| `/mailing/feedback` | structured data | public | Records the chosen reason and the free text; a missing reason answers `error`. |
| `/mailing/blocklist/add` | structured data | public | Adds the address to the blocked-address register; answers true. |
| `/mailing/blocklist/remove` | structured data | public | Removes the address from the register; answers true. |
| `/unsubscribe_from_list` | read | public | The placeholder that a real message never contains; redirects permanently to `/mailing/my`. Deliberately excluded from language prefixing. |

### 4.3 Message viewing and tracking

| Path | Method | Access | Behaviour |
|---|---|---|---|
| `/mailing/<mailing key>/view` | read | public | Renders the body of the mailing for the named recipient inside the browser-view page. |
| `/view` | read | signed in | Renders a generic explanation page; used by the designer's preview of the placeholder link. |
| `/mail/track/<outgoing mail key>/<token>/blank.gif` | read | public | Marks every delivery record carrying that plain integer mail key as opened and answers a one-pixel transparent image. |
| `/mailing/mobile/preview` | read | signed in | Renders the mobile preview frame. |
| `/mailing/report/unsubscribe` | read | public | Turns the statistics message off for the whole installation and renders a confirmation page. |

### 4.4 Text message opt-out

| Path | Method | Access | Behaviour |
|---|---|---|---|
| `/sms/<mailing key>/<code>` | read | public | The opt-out entry page: asks for the number, sanitises it and, on a match, redirects to the confirmation page. |
| `/sms/<mailing key>/unsubscribe/<code>` | read | public | Performs the opt-out or the number blocking and renders the confirmation. |

### 4.5 Website subscription

| Path | Method | Access | Behaviour |
|---|---|---|---|
| `/website_mass_mailing/is_subscriber` | structured data | public | Reports whether the visitor is already subscribed to the named list, and echoes the value used. |
| `/website_mass_mailing/subscribe` | structured data | public | Verifies the human-verification token, then subscribes; answers a success notice reading *"Thanks for subscribing!"* |

### 4.6 Marketing cards

| Path | Method | Access | Behaviour |
|---|---|---|---|
| `/cards/<slug>/card.jpg` and `/cards/<card key>/card.jpg` | read | public | Answers the card image with the image content type and the download name `card.jpg`; sets the share status to `shared` when the caller is a recognised crawler. A card with no image answers not found. |
| `/cards/<slug>/preview` and `/cards/<card key>/preview` | read | public | Renders the sharing page and sets the share status to `visited` when it was empty. |
| `/cards/<slug>/redirect` and `/cards/<card key>/redirect` | read | public | Answers a preview-metadata page to a recognised crawler; redirects everybody else to the campaign target, through the campaign's short address when the card is active. |

### 4.7 Legacy path kept for compatibility

| Path | Method | Access | Behaviour |
|---|---|---|---|
| `/mail/mailing/<mailing key>/unsubscribe` | read | public | Accepts the older parameter spellings and performs the same unsubscription as `/mailing/<mailing key>/unsubscribe`. It exists because addresses of this shape are inside messages already delivered, and a rebuild must keep answering them. |

The credential check that every subscription endpoint runs first, and the exact answers it produces,
are in [workflows.md](workflows.md#18-credential-check-for-the-public-pages).

---

## 5. Documents produced

This domain prints nothing. It produces no Portable Document Format file, no spreadsheet and no
printable form. Three artefacts leave the system nevertheless:

| Artefact | Format | Produced by | Notes |
|---|---|---|---|
| The outgoing message | An electronic mail or a text message | The sending algorithm | Carries the tracking image, the per-recipient unsubscribe and view links and the four unsubscribe headers for electronic mail; carries the shortened links and the opt-out sentence for a text message. |
| The marketing card image | A raster image of 600 by 315 pixels | The card rasteriser | Served by the card image endpoint and fetched by social-network crawlers. |
| The statistics message | An electronic mail | The queue job, one day after a mailing finishes | Assembled from the engagement block, the business block, the link-tracker table and one tip. |

---

## 6. Headers added to an outgoing marketing message

| Header | Value | Purpose |
|---|---|---|
| `List-Unsubscribe` | the one-click unsubscribe address of this recipient, between angle brackets | Lets the reader's mail client offer an unsubscribe button. |
| `List-Unsubscribe-Post` | `List-Unsubscribe=One-Click` | Tells the client it may unsubscribe without opening a page. |
| `Precedence` | `list` | Marks the message as bulk. |
| `X-Auto-Response-Suppress` | `OOF` | Suppresses out-of-office answers. |

---

## 7. Analytical screens

| Screen | Grain | Columns |
|---|---|---|
| Mass Mailing Analysis | One row per combination of trace creation moment, source name, campaign name, mailing type, mailing state and sender address | The counts listed in [entities.md](entities.md#7-mailing-trace-report). Available as a chart, a matrix and a list, once for the electronic-mail type and once for the text-message type. |
| Opt-Out Report | One row per opted-out subscription | Grouped by opt-out reason by default; available as a chart, a matrix, a list and a form. |
| Click Statistics | One row per recorded visit | Grouped by country by default. |
| Statistics of Clicks per campaign | One row per Link Tracker | Opened from a campaign, filtered on that campaign. |
| Mailing List quality | Per list, on the board card | The four counters and the three percentages of [calculations.md](calculations.md#5-mailing-list-quality-percentages). |

---

## 8. Import and export

| Path | Direction | Behaviour |
|---|---|---|
| Paste import | in | The Mailing Contact Import Wizard accepts up to 5000 pasted addresses; above that it refuses and offers the file import. Specified in [workflows.md](workflows.md#8-import-contacts-by-pasting-addresses). |
| File import | in | The general file-import assistant of the platform, opened on Mailing Contact. The domain supplies two import templates, one for a plain contact list and one for a list carrying tags. |
| Website form | in | A public website form may submit to Mailing Contact under the form key `create_mailing_contact`, labelled *"Subscribe to Newsletter"*. It must name at least one list and every named list must be public. |
| Export | out | The platform's ordinary export of any list screen. The domain adds no export of its own. |

---

## 9. External services contacted

| Service | Why | Contract |
|---|---|---|
| The outgoing mail relay | To deliver electronic mail | Owned by the messaging domain. This domain only chooses the server and forbids servers that have a personal owner. |
| The text-message sending service | To deliver text messages and to report delivery | Owned by the messaging domain. This domain supplies the delivery-report address `<base>/sms/status` when it sends a test batch, and reads the failure codes listed in [entities.md](entities.md#64-failure-types). |
| The telephony provider alternative | To deliver text messages through a third-party telephony account instead of the credit-based service | Contributes four further failure codes, all prefixed `twilio_`. |
| Social-network crawlers | To fetch the preview image of a shared marketing card | Recognised by their agent string; the recognised values are listed in [state-machines.md](state-machines.md#3-the-share-status-of-a-marketing-card). |
| The human-verification service | To protect the public subscription endpoint | The action name checked is `website_mass_mailing_subscribe`; a failed verification answers a danger notice carrying the verification error text. |
| The target page of a shortened link | To read its title once, at creation | Only the preview metadata of the page is read; a failure makes the address itself the title. |
| The address of a fetched image | To convert an inline image of a body into a stored file | Limited by the configured import maximum and by 42 million pixels; the refusal messages are in [calculations.md](calculations.md#12-inline-image-conversion). |

Every address of an external service that appears in this specification is reproduced only where it
is part of the integration contract; the installation's own base address is read from
`web.base.url` or resolved from the website of the recipient.

---

## 10. Notifications produced inside the system

| Notification | Where it appears | Text |
|---|---|---|
| Test result | A note on the discussion thread of the mailing | *"Test mailing successfully sent to <address>"*, or *"Test mailing could not be sent to <address>:"* followed by the reason, plus *"Mailing addresses incorrect: <comma-separated list>"* when some lines were unusable. |
| Text-message test result | A note on the discussion thread of the mailing | *"Test SMS successfully sent to <number>"*, *"Test SMS could not be sent to <number>: <explanation>"* and *"Test SMS skipped those numbers as they appear invalid: <comma-separated list>"*. |
| Unsubscription | A note on the discussion thread of each affected Mailing Contact | *"<contact display name> unsubscribed from the following mailing list(s)"* followed by the bulleted list names. |
| Re-subscription | The same | *"<contact display name> subscribed to the following mailing list(s)"* followed by the bulleted list names. |
| Feedback | A message on the contact, on the blocked-address record or on the recipient record | *"Feedback from <author>"* followed by a line break and the text. |
| Blocking request | A note on the blocked-address record | One of the four *"Blocklist request…"* or *"Blocklist removal request…"* sentences of [workflows.md](workflows.md#19-update-subscriptions-give-feedback-exclude-and-re-include). |
| Automatic blocking | A note on the blocked-address record | *"This email has been automatically added in blocklist because of too much bounced."* |
| Number blocking | A note on the blocked-number record | *"Blacklist through SMS Marketing unsubscribe (mailing ID: <mailing key> - model: <recipient entity display name>)"*. |
| Design added or removed | A transient notice on the screen | *"Design added to the <recipient entity names> Templates!"* and *"Design removed from the <recipient entity names> Templates!"* |
| Contacts added to a list | A transient notice on the screen | *"<n> Mailing Contacts have been added. "* — the trailing space is reproduced. |
| Import result | A transient notice on the screen | *"Contacts successfully imported. Number of contacts imported: <n>"*, optionally followed by *". Number of duplicates ignored: <d>"*. |
| Activity tray | The platform's activity tray | The mailing group is renamed *"Email Marketing"*; with the Text Message Marketing package it is split into two groups, one per mailing type. |

---

## 11. What another domain may call

| Called by | Operation | Contract |
|---|---|---|
| Any domain that wants its entity to be mailable | Declare the mailing-enabled flag on the entity | The entity then appears in the recipient-entity chooser. It may also publish a default recipient condition and an opted-out set. |
| Events | Three buttons on an event | Open a prepared mailing addressed to attendees, to invited contacts or to speakers, with the condition and the subject already filled. |
| Learning and courses | One button on a course | Opens a prepared mailing addressed to the contacts of the course members. |
| Any domain rendering a message | Link shortening | Two operations shorten the links of a rich-text body and of a plain-text body, creating Link Tracker records attributed to the supplied campaign, source, medium and mailing. |
| Any domain creating records from a web visit | Campaign tracking | The abstract behaviour stamps campaign, source and medium from the visitor's cookies onto the created record. |
| The messaging domain | Delivery result propagation | After an attempt, the delivery records of a mailing are marked sent or failed; after a bounce or an answer, they are marked bounced, opened or replied. |
