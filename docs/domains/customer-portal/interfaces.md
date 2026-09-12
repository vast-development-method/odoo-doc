# Interfaces of the Customer Portal

The service operations a client or an integration invokes, the request endpoints with their authentication level and their payloads, the screens described as workflows on views, the reports, the notifications and the scheduled job.

---

## 1. Service operations

### 1.1 On a document that adopts the Portal Access Mixin

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| `_portal_ensure_token` | exactly one record | the token text | Writes a freshly generated token with elevated rights when the column is empty. | none |
| `_get_share_url` | `redirect` (boolean, default false), `signup_partner` (boolean, default false), `pid` (a recipient Contact identifier, optional), `share_token` (boolean, default true) | a path with a query string | May create the token; may prepare the customer Contact for sign-up. | The standard read refusal when the acting user may not read the record and the token is requested. The signature refusal `Model %(model_name)s does not support token signature, as it does not have %(field_name)s field.` when a recipient is passed on a model without a token field. |
| `get_portal_url` | `suffix`, `report_type`, `download`, `query_string`, `anchor`, all optional | a path with a query string | May create the token. No permission check. | none |
| `_get_access_action` | `access_uid` (a user identifier, optional), `force_website` (boolean, default false) | an action description | May create the token. | none |
| `action_share` | the active model and the active record identifier taken from the calling context | a dialog action | none | none |

### 1.2 On a discussion thread

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| `_sign_token` | exactly one record, `contact_identifier` | a sixty-four character hexadecimal signature | none | `Model %(model_name)s does not support token signature, as it does not have %(field_name)s field.` |
| `_portal_get_parent_hash_token` | exactly one record, `contact_identifier` | a signature or nothing | none | none |
| `_get_thread_with_access` | `thread_identifier`, `mode`, and any of `hash`, `pid`, `token` | the record, or an empty set | none | Unknown parameter names are logged and ignored. |
| `get_portal_partner` | a thread, `hash`, `pid`, `token` | a Contact, or an empty set | none | none |

### 1.3 On messages

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| `portal_message_format` | a set of messages, `options` (a map that may carry `token`, `hash`, `pid` and the "include rating" flag) | one structure per message, plus one lightweight structure per linked message | none | The standard read refusal when the acting user may not read the messages. |
| `_portal_message_format` | the property names, `options` | the same | none | none; the caller must have validated access already |

### 1.4 On the invitation wizard

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| `action_open_wizard` | the active record identifiers of the calling context | a dialog action | Creates one wizard row and its lines. | The standard model-access refusal for anyone who is not a Contact manager. |
| `action_grant_access` | exactly one line | the reopen-dialog action | Writes the Contact address, creates or reuses a user, sets the groups, prepares the sign-up token, sends the invitation. | `PORT-RULE-001`, `PORT-RULE-002`, `PORT-RULE-003`, `PORT-RULE-009` and the sign-up creation refusals. |
| `action_revoke_access` | exactly one line | the reopen-dialog action | Writes the Contact address, clears the sign-up type, archives the user. | `PORT-RULE-006` |
| `action_invite_again` | exactly one line | the reopen-dialog action | Writes the Contact address, regenerates the sign-up token, sends the invitation. | `PORT-RULE-002`, `PORT-RULE-003`, `PORT-RULE-008`, `PORT-RULE-009` |
| `action_refresh_modal` | exactly one line | the reopen-dialog action | none | none |
| `_get_similar_users_domain` | the addressed lines of the dialog | a condition over User | none | none. **Extension point**; see [entities.md](entities.md) section 4.3.9 and `PORT-RULE-175`. |
| `_get_similar_users_fields` | none | the list of User field names the candidate search reads | none | none. **Extension point**; see [entities.md](entities.md) section 4.3.10. |
| `_is_portal_similar_than_user` | one projected candidate row, one line | boolean | none | none. **Extension point**; see [entities.md](entities.md) section 4.3.11. |

### 1.5 On the share wizard

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| `action_send_mail` | the wizard row | a close-dialog action | Posts one internal note per recipient on the shared document, each notifying that recipient only; may create the document token; may prepare recipients for sign-up. | The standard required-field refusal when no recipient was picked. |
| `_send_public_link` | a set of recipients | none | As above, for the plain-link path. | none |
| `_send_signup_link` | a set of recipients | none | As above, for the sign-up path. | none |

### 1.6 On Contacts

| Operation | Inputs | Output |
|---|---|---|
| `_get_frontend_writable_fields` | none | the closed set of field names an external person may write from a front-end form |
| `_can_edit_country` | exactly one Contact | boolean |
| `can_edit_vat` | exactly one Contact | boolean |
| `_can_be_edited_by_current_customer` | exactly one Contact | boolean |
| `_get_current_partner` | none | the acting person's Contact, or an empty set for the anonymous public user |
| `_get_delivery_address_domain` | a set of Contacts | a condition |

---

## 2. Request endpoints

Authentication levels: **public** means the endpoint answers a request with no session; **user** means it requires a session of any kind. "Front-end" marks an endpoint whose response is rendered inside the front-end layout with the active language resolved from the address.

### 2.1 Endpoints owned by this domain

| Path | Methods | Authentication | Purpose | Request | Response |
|---|---|---|---|---|---|
| `/my`, `/my/home` | GET | user | The portal home. | none | The home page rendered in the front-end layout. Listed as front-end content under the name `User Dashboard`. |
| `/my/counters` | structured call | user, read-only | Compute the card counters. | `counters`: the list of counter names to compute. | A map from counter name to number. Also updates the session counter cache. |
| `/my/account` | GET | user | The account details form for the person's own Contact. | `redirect`: the address to return to after a successful save, default `/my`. | The form page. Headers: framing restricted to the same origin, expressed both as a frame-options header and as a content-security policy. |
| `/my/addresses` | GET | user, read-only | The address book. | none | The address book page. |
| `/my/address` | GET | user, read-only | The address creation or update form. | `partner_id` (optional), `address_type` (`billing` or `delivery`, default `billing`), `use_delivery_as_billing` (`true` or `false`, default `false`). | The form page, or a forbidden response when the Contact may not be edited. Excluded from the sitemap. |
| `/my/address/submit` | POST | user | Create or update an address. | `partner_id` (optional) plus the whole form: `name`, `email`, `phone`, `company_name`, `vat` (abbreviating value-added tax, carrying the tax identification number), `street`, `street2`, `city`, `zip` (or `zipcode`), `country_id`, `state_id`, `address_type`, `use_delivery_as_billing`, `parent_id`, `callback`, `required_fields`, and the request-forgery token. | A structured body: `{ "redirectUrl": <address> }` on success, or `{ "invalid_fields": [names], "messages": [texts] }` on failure. Excluded from the sitemap. |
| `/my/address/country_info/<country>` | POST, structured call | **public**, read-only | The address layout of a country. | `address_type`. | `{ fields, zip_before_city, states, phone_code, required_fields }`, where `states` is a list of triples (identifier, name, code). |
| `/my/address/archive` | POST, structured call | user | Archive an address. | `partner_id`. | Nothing on success. A forbidden response when the Contact may not be edited; the refusal `You cannot archive your main address` when it is the person's own Contact. |
| `/my/security` | GET, POST | user | The connection and security page, and the password change. | On POST: `old`, `new1`, `new2` and the request-forgery token. | The page, with success or error state. Same framing headers as the account page. |
| `/my/deactivate_account` | POST | user | Request the deletion of the account. | `validation` (the login typed back), `password`, `request_blacklist` (present when the checkbox is ticked) and the request-forgery token. | On success a redirection to `/web/login?message=Account deleted!` with the message percent-encoded and the session closed. On failure the security page re-rendered with the dialog open and the error placed on the offending input. |
| `/portal/attachment/remove` | structured call | **public** | Remove a pending attachment. | `attachment_id`, `access_token` (optional). | Nothing on success; the three refusals of `PORT-RULE-070`, `PORT-RULE-071` and `PORT-RULE-072`. |
| `/mail/avatar/<message entity name>/<message>/author_avatar/<width>x<height>` | GET | **public** | The author picture of a message in a portal thread. | `access_token`, or `_hash` and `pid`. | The picture stream at the requested dimensions, or the shipped placeholder when the proof does not resolve the thread, or when neither a token nor a signed identity was supplied. |
| `/portal/chatter_init` | structured call | **public** | Initialize a portal discussion thread. | `thread_model`, `thread_id`, and any of `token`, `hash`, `pid`. | The store seed: the reader's own user data, the publisher marker, the resolved portal Contact (active flag, picture, main user, name), whether the reader may react, whether the reader has direct read access, and the thread display name. |
| `/mail/chatter_fetch` | structured call | **public** | Fetch a page of portal messages. | `thread_model`, `thread_id`, `fetch_params` (the paging parameters), `token`, and, with the rating bridge, `rating_value` and the "include rating" flag. | The paging result, the projected messages and their identifiers. A "not found" response when the thread does not resolve. |
| `/mail/update_is_internal` | structured call | user | Toggle a message between public and internal. | `message_id`, `is_internal`. | The resulting value. |
| `/mail/unfollow` | GET | **public**, front-end, exempt from request-forgery protection | Remove a Contact from a record's followers from a notification link. | `model`, `res_id`, `pid`, `token` (the link signature). | The confirmation page rendered inside the portal layout, with a link back to the record only when the reader is signed in and may read it. The refusal `Non existing record or wrong token.` otherwise. |
| `/website/rating/comment` | POST, structured call | user | Publish a reply under a rating. | `rating_id`, `publisher_comment`. | The formatted reply (`publisher_avatar`, `publisher_comment`, `publisher_datetime`, `publisher_id`, `publisher_name`), or `{ "error": "Invalid rating" }`. |

### 2.2 Endpoints this domain redirects

| Path | Change |
|---|---|
| The platform entry point | A signed-in session whose user is not an employee is redirected to `/my`, carrying the original query parameters. |
| The web client entry point | Same redirection. |
| The post-sign-in destination | When no destination was requested and the authenticated user is not an employee, the destination becomes `/my`. |
| `/mail/view` | When the target model adopts the Portal Access Mixin, a valid token turns a permission refusal into a redirection to the portal page, with the recipient identifier and the signed identity appended when both were supplied. The generic fallback sends a signed-in external user to `/my`. |

### 2.3 Endpoints contributed by the document domains

These are specified in their own folders; they are listed here because they form the portal's visible surface and because each of them follows the contract of section 14 of [configuration.md](configuration.md).

| Path | Methods | Authentication | Purpose | Domain |
|---|---|---|---|---|
| `/my/quotes`, `/my/quotes/page/<page>` | GET | user | The quotation list. | Sales |
| `/my/orders`, `/my/orders/page/<page>` | GET | user | The sales order list. | Sales |
| `/my/orders/<order>` | GET | public | The order page, the report and the download. | Sales |
| `/my/orders/<order>/accept` | structured call | public | Record the signature. | Sales |
| `/my/orders/<order>/decline` | POST | public | Reject the quotation with a reason. | Sales |
| `/my/orders/<order>/transaction` | structured call | public | Create the payment transaction. | Sales, Payment Providers |
| `/my/orders/<order>/document/<document>` | GET | public, read-only | Download a product document attached to the order. | Sales |
| `/my/orders/<order>/download_edi` | GET | public | Download the electronic-interchange representation of the order. | Sales, Electronic Invoicing and Document Exchange |
| `/my/invoices`, `/my/invoices/page/<page>` | GET | user | The invoice and bill list. | Accounts Receivable, Accounts Payable |
| `/my/invoices/<invoice>` | GET | public | The invoice page, the report, the legal document download and the payment form. | Accounts Receivable |
| `/my/journal/<journal>/unsubscribe` | GET, POST | public | Unsubscribe an address from the electronic-invoice notifications of a journal, proved by a signed value. | General Ledger |
| `/my/rfq`, `/my/rfq/page/<page>` | GET | user | The request-for-quotation list on the vendor side; the path segment `rfq` abbreviates request for quotation. | Purchasing |
| `/my/purchase`, `/my/purchase/page/<page>` | GET | user | The purchase order list on the vendor side. | Purchasing |
| `/my/purchase/<order>` | GET | public | The purchase order page, the report, the download and the acknowledgement. | Purchasing |
| `/my/purchase/<order>/update` | structured call | public | Propose new dates on the order lines. | Purchasing |
| `/my/purchase/<order>/download_edi` | GET | public | Download the electronic-interchange representation. | Purchasing |
| `/my/projects`, `/my/projects/page/<page>` | GET | user | The project list. | Projects and Tasks |
| `/my/projects/<project>`, `/my/projects/<project>/page/<page>` | GET | public | The project page with its task list. | Projects and Tasks |
| `/my/projects/<project>/project_sharing`, and any sub-path | GET | user | The collaborator editing screen. | Projects and Tasks |
| `/my/projects/<project>/task/<task>` | GET | public | A task page inside a project. | Projects and Tasks |
| `/my/projects/<project>/task/<task>/subtasks` | GET | user | The sub-task list of a task. | Projects and Tasks |
| `/my/projects/<project>/task/<task>/recurrent_tasks` | GET | user | The recurring occurrences of a task. | Projects and Tasks |
| `/my/tasks`, `/my/tasks/page/<page>` | GET | user | The task list across projects. | Projects and Tasks |
| `/my/tasks/<task>` | GET | public | A task page, and its timesheet report. | Projects and Tasks |
| `/project_sharing/attachment/add_image` | POST | user | Upload an image into a shared task description. | Projects and Tasks |
| `/my/timesheets`, `/my/timesheets/page/<page>` | GET | user | The timesheet list. | Timesheets |
| `/my/payment_method` | GET | user | The saved payment methods page. | Payment Providers |
| `/my/loyalty_card/<card>/history`, `/my/loyalty_card/<card>/values` | GET, structured call | user | The point history of a loyalty card. | Loyalty and Promotions |
| `/my/productions`, `/my/productions/<production>`, `/my/productions/<production>/subcontracting_portal` | GET | user | The subcontracting production pages. | Manufacturing |
| `/my/leads`, `/my/opportunities` and their record pages | GET | user | The assigned lead and opportunity pages of a partnership. | Customer Relationship Management |
| `/event/<event>/my_tickets` | GET | public | The attendee tickets of an event, proved by a signed value over the registration identifiers. | Events |

---

## 3. Screens described as workflows on views

### 3.1 Back office: the invitation dialog

**View kind**: form, opened as a dialog.

| Element | Behavior |
|---|---|
| Explanatory paragraph | `Select which contacts should belong to the portal in the list below. The email address of each selected contact must be valid and unique. If necessary, you can fix any contact's email address directly in the list.` |
| `welcome_message` | A free-text input with the placeholder `This text is included at the end of the email sent to new portal users.` |
| Line list | Editable at the bottom, creation and deletion disabled. |
| Column `Contact` | Read-only, always saved with the line. |
| Column `Email` | Editable, read-only when the line is an employee. |
| State icon | Three mutually exclusive buttons, each of which simply reopens the dialog: a green check, shown when `email_state` is `ok`, titled `Valid Email Address`; a red cross, shown when it is `ko`, titled `Invalid Email Address`; a struck-through person, shown when it is `exist`, titled `Email Address already taken by another user`. |
| Column `Latest Authentication` | Read-only, empty until the person signs in. |
| Button `Grant Access` | Shown when the line is neither an external user nor an employee and the address state is `ok`. |
| Button `Revoke Access` | Shown when the line is an external user and not an employee. |
| Button `Re-Invite` | Shown when the line is an external user, is not an employee and the address state is `ok`. |
| Marker `Internal User` | Shown, disabled, when the line is an employee, titled `This partner is linked to an internal User and already has access to the Portal.` |
| Footer button `Close` | Saves the dialog and closes it. |

### 3.2 Back office: the share dialog

**View kind**: form, opened as a medium dialog.

| Element | Behavior |
|---|---|
| Warning banner | Shown when `access_warning` is not empty; carries the warning text. |
| `share_link` | Read-only with a copy-to-clipboard control labelled `Copy Link`. |
| `partner_ids` | A tag selector that accepts electronic mail addresses, with the placeholder `Add contacts to share the document...`. Required. |
| `note` | A free-text input with the placeholder `Add a note`. |
| Button `Send` | Hidden when `access_warning` is not empty. |
| Button `Cancel` | Discards the dialog. |

### 3.3 Portal: the home page

**View kind**: a card grid inside the account layout.

| Element | Behavior |
|---|---|
| Heading | `My account`, with the avatar button that opens the offcanvas sidebar on a narrow screen. |
| Alert row | Rendered only when a package contributed to it. Shipped contributions: `Quotations to review` (always shown, coloured, pointing at `/my/quotes`) and `Invoices to pay` (always shown, coloured, pointing at `/my/invoices?filterby=overdue_invoices`). |
| Client row | Shipped contributions: `Your Orders` / `Follow, view or pay your orders` → `/my/orders`; `Your Invoices` / `Follow, download or pay your invoices` → `/my/invoices?filterby=invoices`. |
| Service row | Shipped contributions: `Projects` / `Follow the evolution of your projects` → `/my/projects`; `Tasks` / `Follow and comment on tasks in your projects` → `/my/tasks`; `Timesheets` / `Review all timesheets related to your projects` → `/my/timesheets`. |
| Vendor row | Shipped contributions: `Requests for Quotation` / `Follow your Requests for Quotation` → `/my/rfq` (path segment `rfq`); `Our Orders` / `Follow orders you have to fulfill` → `/my/purchase`; `Our Invoices` / `Follow, download or pay our invoices` → `/my/invoices?filterby=bills`. |
| Common row | `Addresses` / `Add, remove or modify your addresses` → `/my/addresses`; `Connection & Security` / `Configure your connection parameters` → `/my/security`. Always visible. |
| Spinner | Removed once every counter call has answered. |
| Sidebar | The account sidebar described in section 22 of [workflows.md](workflows.md). |

### 3.4 Portal: a document list page

**View kind**: a search bar, a table and a navigator.

Described as a contract in section 10 of [workflows.md](workflows.md); the table columns are chosen by each document domain.

### 3.5 Portal: a document record page

**View kind**: a breadcrumb row with the record navigator, a sidebar column and a content column.

Described as a contract in section 11 of [workflows.md](workflows.md).

### 3.6 Portal: the account details form

| Input | Label | Required | Notes |
|---|---|---|---|
| `name` | `Your name` | yes | Refused for an employee's own Contact. |
| `email` | `Email` | yes | Format checked. Refused for an employee's own Contact. |
| `phone` | `Phone` | when the page needs an address | The placeholder becomes the country's telephone prefix. |
| `company_name` | `Company Name` | no | Shown only on a billing address; read-only when the edited address is not the main address. |
| `vat` (the input name abbreviates value-added tax) | the tax identification number label | no | Shown only on a billing address; read-only when the address is not the commercial address or when the number is frozen; the reason is explained underneath. |
| `street` | `Street and Number` | when the page needs an address | |
| `street2` | `Apartment, suite, etc.` | no | |
| `zip` | `Zip Code` | when the country requires it | Rendered before or after the city according to the country's layout. |
| `city` | `City` | when the page needs an address | |
| `country_id` | `Country` | when the page needs an address | Disabled and explained when the country is frozen; the first entry is `Country...`. |
| `state_id` | `State / Province` | when the country requires it | Hidden when the country has no subdivision; the first entry is `State / Province...`. |
| Footer | `Discard` and `Save Address` | | `Discard` goes to the success address, or to `/my/`. |

### 3.7 Portal: the connection and security page

| Section | Content | Shown when |
|---|---|---|
| Heading | A back control to `/my/` and the title `Connection & Security`. | always |
| `Change Password` | The three inputs `Password:`, `New Password:` and `Verify New Password:`, each with a show-password control; the success banner `Password Updated!`; the error banner; the button `Change Password`. A capitals-lock warning is attached to the new-password block. | always |
| `Two-factor authentication` | Either the status box `Two-factor authentication not enabled` with the button `Enable two-factor authentication`, or the line `Two-factor authentication enabled` with the link `(Disable two-factor authentication)`; plus, when trusted devices exist, the table `Trusted Device` / `Added On` with a delete control per row and the button `Revoke All`. A documentation link is offered next to the heading. | the second-factor bridge is installed |
| `Passkeys` | The table `Name` / `Created` / `Last Used` with a rename and a delete control per row, and the button `Add Passkey`. | the passkey bridge is installed |
| `Revoke All Sessions` | The button `Log out from all devices`. | always |
| `Developer Application Keys` | The table `Description` / `Scope` / `Added On` / `Expiration Date` with a delete control per row, and the button `New Application Key`; plus a documentation link next to the heading. | developer mode **and** the setting is on |
| `Delete Account` | The button `Delete Account` and the confirmation dialog. | the reader is a member of the portal group |

The confirmation dialog contains: the title `Are you sure you want to do this?`; the sentences `Disable your account, preventing any further login.` and `This action cannot be undone.`; step one `1. Enter your password to confirm you own this account` with a required password input; step two `2. Confirm you want to delete your account by copying down your login (<login>).` with a required text input; the checkbox `Put my email and phone in a block list to make sure I'm never contacted again`, ticked by default; and the buttons `Delete Account` and `Cancel`.

### 3.8 Portal: the address book

| Element | Behavior |
|---|---|
| Section `Delivery address` | Heading plus an `Add Address` control carrying the delivery kind and the "use as billing" flag. |
| Delivery card list | One card per delivery address. |
| Section `Billing address` | Heading plus an `Add Address` control carrying the billing kind, hidden while the "same as delivery" switch is on. |
| Switch `Same as delivery address` | Checked when no address in the billing list has the billing kind; toggling it hides or shows the billing section and rewrites the delivery `Add Address` link. |
| Billing card list | One card per billing address, hidden while the switch is on. |
| Card badge | `Main Address`, `Delivery Address` or `Billing Address`. |
| Card edit control | Titled `Edit this address`; points at `/my/account?redirect=/my/addresses` for the person's own Contact, and at the address form otherwise. |
| Card remove control | Titled `Remove this address`; shown only when the address is editable, is not the selected one and is not the person's own Contact. |

### 3.9 Portal: the discussion thread

| Element | Behavior |
|---|---|
| Message list | Public messages only, newest-first paging of ten by default, each with the author name, the author picture, the formatted publication date, the body, the attachments and the reactions. |
| Composer | Shown when the anchor allows it. For a reader with no proven identity, it shows `Leave a comment` and `You must be logged in to post a comment.` with a sign-in link that returns to the thread anchor. Otherwise it shows the avatar, the text area with the placeholder `Write a message...`, the attachment control titled `Add attachment` and the submit control. |
| Pending attachments | Listed under the text area, each with a delete control titled `Delete` and a download link. |
| Rating widget | When the rating bridge is installed and the page asks for it: the star row, the average, the count and the review button. |
| Publisher reply | Shown under a rating, with the publisher picture, name, publication moment and comment. |

---

## 4. Reports and printed documents

This domain renders no report of its own. It provides the rendering path and the file naming for the reports owned by the document domains.

| Aspect | Rule |
|---|---|
| Kinds served | rich text, portable document and plain text; anything else is refused (`PORT-RULE-021`). |
| Company | The document's own company, single-company only (`PORT-RULE-022`, `PORT-RULE-023`). |
| Media type | The portable-document media type for a portable document, the rich-text media type otherwise. |
| Content length | Always set. |
| Disposition | Only for a portable document: attachment when the reader asked to download, inline otherwise; the file name is derived from the record's report base name (`PORT-RULE-024`). |
| Language | The reader's language, except where a document domain overrides it (the invoice page switches to the customer's language when that language is written right to left). |

---

## 5. Exported files

| Export | Endpoint | Content |
|---|---|---|
| Invoice legal documents | The invoice page with a downloadable portable document requested on a posted invoice | One document when there is a single one; otherwise a compressed archive named after the invoice, containing every legal document. |
| Order interchange document | The order download endpoints | The interchange representation of the order produced by the first available builder, served as a rich-structured text document with an attachment disposition and a file name produced by the builder. |
| Product documents of an order | The order document endpoint | The attachment behind the product document, served as an attachment, only when the document is active and belongs to the order's product document set. |
| Attendee tickets | The event ticket endpoint | The ticket or badge document for a set of registrations, proved by a signed value over the registration identifiers. |

---

## 6. Notifications

| Notification | Trigger | Channel | Recipients | Content |
|---|---|---|---|---|
| Portal invitation | `action_grant_access` or `action_invite_again` | electronic mail, sent immediately | the invited person | The shipped template `Settings: New Portal User Invite`, carrying the activation link and the invitation session's welcome message. |
| Share invitation | `action_send_mail` | electronic mail, through an internal note | one named Contact per message | The invitation body with a button `Open <document display name>` pointing at that recipient's personal link, plus the note. |
| Customer access button on every notification | Any message notified on a record that adopts both the thread mixin and the access mixin, when the record has a customer | electronic mail | the record's customer | An access button pointing at the generic redirection endpoint with the token, the recipient identifier, the signed identity and the sign-up parameters. |
| Access button for other external recipients | The same trigger | electronic mail | every other external recipient | The generic external-user access button, forced to be active. |
| `Quotation viewed by customer <name>` | An external or anonymous reader opens a quotation in the draft or sent state with a token, when the request is not a link preview and the note was not already logged for that order today in this session | a note on the order | the salesperson through the follower rules | Logged in the salesperson's language, not the reader's. |
| `Order signed by <name>` | The signature endpoint | a public comment on the order, with the signed report attached | the order's followers | Authored by the order's Contact for an anonymous signer, by the acting user's Contact otherwise. |
| The rejection reason | The rejection endpoint | a public comment on the order | the order's followers | The text typed by the customer. |
| `Archived because <name> (#<identifier>) deleted the portal account` | The account deletion request | a note on the Contact | the Contact's followers | Explains why the Contact is archived. |
| `Blocked by deletion of portal account <person> by <name> (#<identifier>)` | The account deletion request with the block-list choice | the reason stored on the electronic mail block list entry, and a note on the telephone block list entry | not applicable | Explains why the address and the number are blocked. |

---

## 7. Scheduled job

| Job | Schedule | Work |
|---|---|---|
| `Base: Portal Users Deletion` | once a day, priority 8, as the system robot user | Processes the User Deletion Request queue in batches of fifty; see section 30 of [workflows.md](workflows.md) for the full algorithm and section 33.3 for the state table. |

---

## 8. Front-end session payload

When a request carries a session, the front-end session payload produced by the routing layer is extended with:

| Key | Content |
|---|---|
| tours enabled | Whether guided tours are enabled for the signed-in user. |
| current tour | The tour to run, when tours are enabled. |

This is what allows the portal onboarding tour to run for an external person, who cannot reach the back office where tours normally live.
