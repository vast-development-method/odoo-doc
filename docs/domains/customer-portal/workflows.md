# Workflows of the Customer Portal

Every operational workflow of the Customer Portal end to end: the actors, the preconditions, the numbered steps, the branches, the records written with their field values, the messages sent and the postconditions. Rules cited as `PORT-RULE-nnn` are defined in [business-rules.md](business-rules.md); formulas cited by name are defined in [calculations.md](calculations.md); the state machines the steps move are specified in [state-machines.md](state-machines.md).

Throughout this file, "elevated rights" means that the operation reads or writes while bypassing the permission layers, keeping the acting user for authorship; "acting user" means the identity that issued the request; "external user" means a user whose `share` characteristic is true.

## Table of contents

1. Granting portal access to a Contact
2. Granting portal access to several Contacts at once
3. Revoking portal access
4. Re-sending an invitation
5. Activating an invited account
6. Sharing a document by link
7. Opening a shared document from a link
8. Signing in as an external person
9. Loading the portal home and its counters
10. Browsing a document list page
11. Opening a document record page
12. Rendering and downloading a document report
13. Loading the discussion thread of a portal page
14. Posting a message from a portal page
15. Attaching a file to a portal message and removing a pending attachment
16. Reacting to a message from a portal page
17. Marking a message internal or public
18. Rating a record from the portal
19. Publishing a reply under a rating
20. Signing a document from the portal
21. Paying a document from the portal
22. Viewing and editing the account details
23. Managing the address book
24. Changing the password
25. Managing the second authentication factor
26. Managing passkeys
27. Revoking all other sessions
28. Managing application keys
29. Requesting the deletion of the account
30. Processing the account deletion queue
31. Switching the front-end language
32. Unfollowing a document from a notification link
33. State tables

---

## 1. Granting portal access to a Contact

**Actors**: Contact manager (back office), invited person (recipient of the message).

**Preconditions**: a Contact record exists. The portal user template exists and its identifier is held in the sign-up template system parameter. The shipped invitation message template exists.

**Steps**

1. The Contact manager selects one Contact in the Contact list or opens a Contact form, and triggers the action `Grant portal access`.
2. The server action creates one Portal Access Wizard row. Creating it fills `partner_ids` by expansion (`PORT-RULE-030`): the selected Contact plus its child Contacts whose address kind is `contact` or `other`. Creating it also derives one Portal Access Wizard User line per Contact in `partner_ids`, each carrying `partner_id` and `email` copied from the Contact.
3. The server action returns a dialog titled `Portal Access Management` showing:
   - an explanatory paragraph: `Select which contacts should belong to the portal in the list below. The email address of each selected contact must be valid and unique. If necessary, you can fix any contact's email address directly in the list.`
   - the `welcome_message` input with the placeholder `This text is included at the end of the email sent to new portal users.`
   - the editable line list with the columns Contact, Email, a state icon, Latest Authentication and the action buttons.
4. For each line the state is derived: `user_id` (the Contact's first user, archived ones included), `is_internal`, `is_portal`, `email_state` (computed for the whole line list in one pass through the three extension points of section 4.2 of [entities.md](entities.md), narrowed per website by `PORT-RULE-175` when the website capability package is installed) and `login_date`. The state icon shows a green check with the tooltip `Valid Email Address` for `ok`, a red cross with the tooltip `Invalid Email Address` for `ko`, and a red struck-through person with the tooltip `Email Address already taken by another user` for `exist`.
5. The Contact manager may correct the electronic mail address in the line. Doing so re-derives `email_state` for the whole line list.
6. The Contact manager presses **Grant Access** on a line. The button is visible only when `is_portal` is false, `is_internal` is false and `email_state` is `ok`.
7. The server asserts the address is usable (`PORT-RULE-002`, `PORT-RULE-003`).
8. The server asserts the line does not already have access (`PORT-RULE-001`).
9. **Write on the Contact**: when the normalized address of the line differs from the normalized address of the Contact, `email` is set to the normalized address (`PORT-RULE-004`).
10. **Branch A, the Contact has no user**: the company is chosen as the Contact's company when set, otherwise the acting user's current company (`PORT-RULE-005`). A user is created by duplicating the portal user template with `login` and `email` set to the normalized address, `partner_id` set to the Contact, `company_id` set to the chosen company, `company_ids` set to exactly that company, `active` set to true, and the "do not send a password reset message" marker set.
    **Branch B, the Contact already has a user** (public, or archived external): that user is reused.
11. **Write on the user** (with elevated rights): `active = true`; the portal group is added; the public group is removed.
12. **Write on the Contact** (with elevated rights): `signup_type = signup`.
13. The invitation message is sent (section 1.1). The line is evaluated with archived records excluded so that only the live user is considered.
14. The dialog is reopened, which re-derives every line.

**Records written**

| Record | Fields written |
|---|---|
| Contact | `email` (when corrected), `signup_type = signup` |
| User (created or reused) | `login`, `email`, `partner_id`, `company_id`, `company_ids`, `active = true`, group membership: portal added, public removed |
| Message queue | one outgoing message to the invited person |

**Postconditions**: the line shows `is_portal = true`; the Latest Authentication column is still empty until the person signs in; the invited person holds a link that lets them choose a password.

**Failure paths**

| Condition | Outcome |
|---|---|
| `email_state` is `ko` | `The contact "<Contact name>" does not have a valid email.` Nothing is written. |
| `email_state` is `exist` | `The contact "<Contact name>" has the same email as an existing user` Nothing is written. |
| The line is already an external user or is an employee | `The partner "<Contact name>" already has the portal access.` Nothing is written, no message sent. |
| The portal user template does not resolve | `Signup: invalid template user` |
| The invitation template does not resolve | `The template "Portal: new user" not found for sending email to the portal user.` The user has already been created and put in the portal group at that point; the operation is rolled back with the surrounding transaction. |

### 1.1 Sending the invitation message

1. Resolve the shipped template `Settings: New Portal User Invite`, raising the refusal above when it is absent.
2. Take the language and the Contact of the linked user.
3. Prepare that Contact for sign-up again, which regenerates the token material.
4. Render the template against the **user** record with the rendering context carrying: the database name, the language, the wizard's `welcome_message` and the tracking medium `portalinvite`.
5. Send immediately (not queued).

The rendered message reads:

- Subject: `Your account at <company name>`
- Sender: the company's formatted address when set, otherwise the acting user's formatted address
- Recipient: the user's formatted address
- Header: the caption `Your Account` above the person's name, and the company logo when the company does not use the default logo
- Body: `Dear <name>,` / `Welcome to <company name>'s Portal!` / `An account has been created for you with the following login: <login>` / `Click on the button below to pick a password and activate your account.` / a button labelled `Activate Account` pointing at the Contact's sign-up web address / the welcome message
- Footer: the company name, telephone, electronic mail address and website, and a "powered by" credit line.

---

## 2. Granting portal access to several Contacts at once

**Actors**: Contact manager.

**Preconditions**: several Contacts are selected, or one company Contact with children is selected.

**Steps**

1. The Contact manager selects several Contacts (or the company Contact) and triggers `Grant portal access`.
2. The expansion of `partner_ids` unions, for every selected Contact, that Contact and its child Contacts of kind `contact` and `other`. Duplicates are removed, so a person who is a child of two selected companies appears once.
3. One line per resulting Contact is shown.
4. The Contact manager presses **Grant Access** on each line that should get access. Each press runs the complete section 1 workflow for that line only, then reopens the dialog.
5. Lines whose Contact is an employee show a disabled button labelled `Internal User` with the tooltip `This partner is linked to an internal User and already has access to the Portal.` and their Email cell is read-only.
6. The Contact manager presses `Close` to leave the dialog.

**Postconditions**: each granted line has its own user, its own sign-up token and has received its own message. Lines left untouched are unchanged.

---

## 3. Revoking portal access

**Actors**: Contact manager.

**Preconditions**: the line shows `is_portal = true`.

**Steps**

1. The Contact manager presses **Revoke Access**. The button is visible only when `is_portal` is true and `is_internal` is false.
2. The server checks the guard (`PORT-RULE-006`).
3. **Write on the Contact**: the normalized address when it differs (`PORT-RULE-004`), and `signup_type` cleared.
4. **Write on the user** (with elevated rights), when the linked user exists and is an external user: `active = false`.
5. The dialog is reopened. The line now shows `is_portal = false` and keeps its `user_id`, which is the archived one.

**Postconditions**: the person can no longer sign in; their outstanding activation or reset link no longer resolves, because the token payload includes the Contact's sign-up type and the list of its users; the user keeps the portal group so that a later grant restores access without touching group membership semantics (`PORT-RULE-007`); no message is sent.

**Failure paths**

| Condition | Outcome |
|---|---|
| `is_portal` is false (no user, public user, archived user, or employee) | `The partner "<Contact name>" has no portal access or is internal.` |

---

## 4. Re-sending an invitation

**Actors**: Contact manager.

**Preconditions**: the line has `is_portal` true and its `email_state` is `ok`.

**Steps**

1. The Contact manager presses **Re-Invite**.
2. The address assertion runs (`PORT-RULE-002`, `PORT-RULE-003`).
3. The guard runs (`PORT-RULE-008`).
4. The normalized address is written back onto the Contact when it differs.
5. The invitation message is sent again (section 1.1), which prepares the Contact for sign-up once more.
6. The dialog is reopened.

**Postconditions**: a fresh activation link has been sent; the previous link stops working, because the token payload is regenerated and the previous payload no longer matches the Contact's state.

---

## 5. Activating an invited account

**Actors**: invited person.

**Preconditions**: the person holds an invitation message with an `Activate Account` link.

**Steps**

1. The person opens the link, which carries the database name and the sign-up token.
2. The sign-up flow of [Identity and Access](../identity-and-access/README.md) validates the token: it resolves the Contact, and checks that the Contact's latest authentication moment, the list of the Contact's users and the Contact's sign-up type still match the values captured when the token was produced, and that the token has not expired (the sign-up validity window, one hundred and forty-four hours by default; the reset validity window, four hours by default, for a reset token).
3. The person chooses a password.
4. The sign-up flow writes the password on the user and clears the Contact's sign-up type, which invalidates the token.
5. The person is signed in and redirected.
6. Because the person is not an employee, the post-authentication redirection target is forced to `/my` (`PORT-RULE-040`).

**Postconditions**: the person is an active external user with a password, sees the portal home, and the Latest Authentication column of the wizard now shows a value.

---

## 6. Sharing a document by link

**Actors**: any employee who may read the document and open the share dialog.

**Preconditions**: the document's model adopts the Portal Access Mixin.

**Steps**

1. The employee opens the document and triggers `Share Document`. The action carries the active model and the active record identifier in its context.
2. A Portal Share Wizard row is created. Its defaults set `res_model`, `res_id` and `share_link`.
3. The dialog shows:
   - a warning banner with `access_warning` when it is not empty;
   - the `share_link` with a copy-to-clipboard control labelled `Copy Link`;
   - the recipient selector with the placeholder `Add contacts to share the document...`;
   - the note input with the placeholder `Add a note`;
   - the **Send** button (hidden when `access_warning` is not empty) and a **Cancel** button.
4. Computing `share_link` calls `_portal_ensure_token` on the document, so **the token is created at dialog opening**, before anything is sent. From that moment the plain link is usable.
5. The employee picks recipients and presses **Send**.
6. The wizard reads the `auth_signup.invitation_scope` system parameter.
7. It splits the recipients (`PORT-RULE-010`):
   - when the document has a token **or** the scope is `b2b`: all recipients take the plain-link path;
   - otherwise: only recipients that already have a user take the plain-link path.
8. For each plain-link recipient: the share address is rebuilt with that recipient's identifier and the signed identity for that recipient; the rendering language is switched to the recipient's language; an internal note is posted on the document carrying the invitation body, the subject `Invitation to access <document display name>`, the light notification layout and that single recipient; the language is restored.
9. For each remaining recipient: the Contact is prepared for sign-up, a sign-up web address pointing at the generic redirection endpoint with the document model and identifier is built, and the same internal note is posted with that address.
10. The dialog closes.

**Records written**

| Record | Fields written |
|---|---|
| Document | `access_token` (when it was empty) |
| Contact (sign-up path only) | `signup_type = signup` |
| Message | one internal note per recipient on the document, with that recipient as the only notified party |

**Postconditions**: every recipient has received a personal link; the plain-link recipients can read the document and post messages under their own Contact identity without an account; the sign-up recipients must create an account first and then land on the document.

---

## 7. Opening a shared document from a link

**Actors**: anonymous visitor, external user, or employee.

**Preconditions**: a link of one of two shapes:
- the **direct** shape `<portal web address>?access_token=<token>[&pid=<contact>&hash=<signed identity>]`;
- the **redirected** shape `/mail/view?model=<model>&res_id=<identifier>&access_token=<token>[&pid=...&hash=...][&auth_signup_token=... | &auth_login=...]`.

### 7.1 The redirected shape

1. The request reaches the generic redirection endpoint, which is open to anonymous visitors.
2. When neither the model nor the identifier is usable, or the model is unknown, the generic fallback applies (step 8).
3. When the model adopts the Portal Access Mixin:
   - resolve the acting identity as the session user, or the anonymous public user when there is no session;
   - browse the record with elevated rights;
   - check read permission for that identity;
   - when the check fails **and** the record has a token **and** the supplied token matches it under a constant-time comparison: compute the access action with the front-end redirection forced; when that action is an "open web address" action, append the recipient identifier and the signed identity to its address when both were supplied, and redirect there. **The visitor lands on the portal page.**
4. Otherwise the generic redirection logic applies: for a signed-in identity that may read the record, the access action is computed for that identity; an external user gets the portal address with the token appended, an employee gets the back-office form.
5. For a signed-in identity that may not read the record, the generic fallback applies.
6. For an anonymous visitor whose access action is an "open web address" action that is not explicitly marked public, the visitor is sent to the sign-in page with the original redirection address preserved, so that they land on the document after signing in.
7. When the record does not exist, the generic fallback applies.
8. **Generic fallback**: an anonymous visitor is sent to the sign-in page with the original redirection address preserved; a signed-in external user is sent to `/my` (`PORT-RULE-041`); a signed-in employee is sent to the messaging screen.

### 7.2 The direct shape

1. The request reaches the document page endpoint of the owning domain, which is open to anonymous visitors.
2. The endpoint resolves the document through the shared access check (`PORT-RULE-020`):
   - browse the record with elevated rights; when it does not exist, refuse with `This document does not exist.`;
   - check read permission for the acting identity; when it succeeds, return the record read with elevated rights;
   - when it fails, compare the supplied token with the record's token under a constant-time comparison; when they match, return the record read with elevated rights; otherwise re-raise the permission refusal.
3. On a permission refusal or a missing record, the page endpoint of every shipped document domain redirects to `/my`.
4. The page renders with the token carried in the rendering values, so that every link the page produces (report, download, payment, signature, discussion thread) carries it forward.

**Postconditions**: a visitor holding the token reads the document without an account; a visitor holding a signed identity in addition may also post messages, react and edit their own messages.

---

## 8. Signing in as an external person

**Actors**: external user.

**Steps**

1. The person submits the sign-in form.
2. On success, the redirection target is computed. When no explicit target was requested and the authenticated user is not an employee, the target is forced to `/my` (`PORT-RULE-040`).
3. Any later attempt by that person to open the back-office entry point or the web client is answered with a redirection to `/my`, preserving the query string (`PORT-RULE-041`).

**Postconditions**: an external person never reaches the back office, even by typing its address.

---

## 9. Loading the portal home and its counters

**Actors**: external user or employee, signed in.

**Preconditions**: none beyond a session.

**Steps**

1. The browser requests `/my` (or `/my/home`).
2. The server prepares the shared layout values:
   - resolve the **assigned representative**: the salesperson of the acting user's Contact when that Contact has one and it is not the anonymous public user; otherwise the salesperson of the Contact's commercial entity when that one is not the anonymous public user; otherwise none (`PORT-RULE-050`);
   - set the page name to `home`.
3. The server calls the counter preparation with an **empty** list of requested counters, which every contributing domain interprets as "compute nothing". The home page is therefore rendered without any counter value.
4. The page renders the card grid: four optional category rows (alert, client, service, vendor), each enabled only when at least one capability package contributed to it, then the common category containing two cards that are always visible:
   - `Addresses` / `Add, remove or modify your addresses` → `/my/addresses`
   - `Connection & Security` / `Configure your connection parameters` → `/my/security`
   - and a spinner placeholder.
5. Each contributed card is rendered hidden, unless the session remembers that its counter was non-zero on a previous load, or the card is marked as always shown.
6. The front end collects the counter names of every card on the page and issues at most three parallel calls to `/my/counters`, each carrying a slice of the names (`counter batching`, see [calculations.md](calculations.md)).
7. Each call runs the counter preparation of every contributing domain for the requested names only, then updates the session cache: for every returned name ending in `_count`, the cache stores whether the value is non-zero. The cache is written back only when it changed.
8. The front end writes each returned number into its card and reveals the card when the number is not zero or when the card is in the always-shown list.
9. The spinner is removed once every call has answered.

**Counters contributed by the shipped domains**

| Counter | Contributing domain | Population |
|---|---|---|
| `quotation_count` | Sales | Sales orders of the commercial entity's tree in state "sent". |
| `order_count` | Sales | Sales orders of the commercial entity's tree in state "confirmed"; counted with a limit of one, so the card shows `1` whenever at least one exists. |
| `invoice_count` | Accounts Receivable | Customer invoices, credit notes and receipts that are neither cancelled nor draft; counted with a limit of one. |
| `bill_count` | Accounts Payable | Vendor bills, refunds and receipts that are neither cancelled nor draft; counted with a limit of one. |
| `overdue_invoice_count` | Accounts Receivable | Customer invoices and receipts of the acting Contact that are neither cancelled nor draft, whose payment state is not among in-payment, paid, reversed, blocked or legacy, and whose due date is before today. |
| `rfq_count` | Purchasing | Purchase orders in state "sent". |
| `purchase_count` | Purchasing | Purchase orders in state "confirmed" or "cancelled". |
| `project_count` | Projects and Tasks | Projects readable by the acting user. |
| `task_count` | Projects and Tasks | Tasks readable by the acting user that belong to a project. |
| `timesheet_count` | Timesheets | Timesheet lines matching the portal filter of the Timesheets domain, counted with elevated rights. |
| `production_count` | Manufacturing | Subcontracting productions of the acting vendor. |
| `lead_count`, `opp_count` | Customer Relationship Management | Assigned leads and opportunities of a partnership. |

Every counter is computed only when the acting user actually has read permission on the underlying model; otherwise the value is zero (`PORT-RULE-051`).

**Postconditions**: the page shows only the cards that have content, and the session remembers which cards to show immediately next time.

---

## 10. Browsing a document list page

**Actors**: external user or employee, signed in. Every list page requires a session; none of them accepts a token.

**Preconditions**: the acting user has read permission on the listed model, or the page shows an empty list.

**Generic steps** (each contributing domain fills in the specific filter, sort and search definitions)

1. The browser requests the page address, optionally suffixed with `/page/<number>`, and optionally carrying `sortby`, `filterby`, `groupby`, `search_in`, `search`, `date_begin` and `date_end`.
2. The server prepares the shared layout values (section 9, step 2).
3. The server builds the base condition of the page. Every shipped list page scopes to the acting person: either through the record rules of the listed model (which restrict to the commercial entity's tree), or through an explicit condition.
4. The sort key defaults to the domain's default when absent, and selects an ordering expression from the domain's sort table.
5. The filter key defaults to `all` when absent, and its condition is intersected into the base condition.
6. When both `date_begin` and `date_end` are given, the condition is intersected with `create_date > date_begin AND create_date <= date_end`. Note the asymmetry of the bounds: the lower bound is exclusive and the upper bound is inclusive.
7. When `search` and `search_in` are both given, the search condition of the domain for that scope is intersected.
8. The total number of matching rows is counted; when the acting user has no read permission the total is zero.
9. The page navigator is computed (`page navigator`, see [calculations.md](calculations.md)) with a page size of eighty rows, except on the timesheet page where it is one hundred.
10. The rows of the current page are read with the chosen ordering, the page size as the limit and the navigator's offset.
11. The identifiers of the rows are stored in the session under the page's history key, truncated to the first one hundred.
12. The page renders the search bar (title or breadcrumb, sort selector, filter selector, group-by selector, scoped search input), the table, and the navigator.

**The search bar contract**

| Element | Shown when | Behavior |
|---|---|---|
| Breadcrumb | the page sets the breadcrumb flag | Replaces the title with the portal breadcrumb. |
| Title | otherwise | The page title, or `No title`. |
| `Sort By:` | the page supplies a sort table | A menu; choosing an entry reloads the **current** path with every existing query parameter preserved and `sortby` replaced, so the page number is kept. |
| `Filter By:` | the page supplies a filter table | A menu; choosing an entry reloads the page's **default** address with every existing query parameter preserved and `filterby` replaced, so the reader returns to the first page. |
| `Group By:` | the page supplies a group table | A menu; choosing an entry reloads the page's default address with `groupby` replaced, so the reader returns to the first page. |
| Search box | the page supplies a scope table | A menu of scopes plus a text input. Submitting sets `search_in` to the selected scope and `search` to the text in the query string of the current address. The placeholder of the input mirrors the label of the selected scope. |
| Toggle | any selector exists | On a narrow screen, collapses the selectors behind a button labelled `Toggle filters`. |

**The page navigator contract**

The navigator is shown only when there is more than one page. It renders a previous control, the computed page list (with a non-clickable ellipsis entry where the list is elided), and a next control. The previous control is disabled on page one and the next control on the last page. Every entry keeps the page's query parameters.

**Shipped list pages**

| Page | Address | Listed entity | Default sort | Sorts | Filters | Group-by | Search scopes |
|---|---|---|---|---|---|---|---|
| Quotations | `/my/quotes` | Sales Order in state "sent" | `date` | `Order Date` | none | none | none |
| Sales orders | `/my/orders` | Sales Order in state "confirmed" | `date` | `Order Date` | none | none | none |
| Invoices and bills | `/my/invoices` | Journal Entry, invoice kinds, not cancelled and not draft | `date` | `Date`, `Due Date`, `Reference`, `Status` | `All`, `Overdue invoices`, `Invoices`, `Bills` (presented in alphabetical order of their keys) | none | none |
| Requests for quotation | `/my/rfq` (`rfq` stands for request for quotation) | Purchase Order in state "sent" | `date` | `Newest`, `Name`, `Total` | none | none | none |
| Purchase orders | `/my/purchase` | Purchase Order | `date` | `Newest`, `Name`, `Total` | `All` (confirmed or cancelled), `Purchase Order`, `Cancelled` | none | none |
| Projects | `/my/projects` | Project, excluding templates | `name` | `Newest`, `Name` | none | none | none |
| Tasks | `/my/tasks` | Task with a project, excluding template descendants | first entry of the sort table | `Newest`, `Title`, `Project`, `Stage`, `Status`, `Milestone`, `Priority`, `Deadline`, `Last Stage Update` | `All` plus one entry per readable project | `None`, `Stage`, `Project`, `Status`, `Milestone`, `Priority`, `Customer` | tasks, assignees, stages, status, project, priority, milestone, customer |
| Timesheets | `/my/timesheets` | Analytic Line matching the timesheet portal filter | `date desc` | `Newest`, `Employee`, `Project`, `Task`, `Description` | `All`, `Last Year`, `Last Quarter`, `Last Month`, `Last Week`, `Today`, `This Week`, `This Month`, `This Quarter`, `This Year` | `None`, `Date`, `Project`, `Parent Task`, `Task`, `Employee` | description, employee, project, task, parent task |

**Postconditions**: the session holds the ordered identifiers of the last viewed list, which powers the previous and next controls on the record pages of that list.

---

## 11. Opening a document record page

**Actors**: anonymous visitor with a token, external user, or employee.

**Steps**

1. The browser requests the record address, optionally carrying `access_token`, `pid`, `hash`, `report_type`, `download`, `message`, `error`, `warning` or `success`.
2. The endpoint resolves the record through the shared access check (`PORT-RULE-020`). On failure it redirects to `/my`.
3. When `report_type` names a supported form, the report path is taken instead (section 12).
4. The owning domain computes its own page values (amounts, lines, actions, payment context).
5. The shared record-page values are added:
   - the record itself under the name `object`;
   - when a token was supplied: the token under two names (`access_token` for the page links and `token` for the discussion thread) and the "hide the breadcrumb" flag, which the caller may override to force the breadcrumb so that an anonymous reader is invited to register;
   - the flash values `error`, `warning` and `success` when present;
   - the recipient identifier `pid` and the signed identity `hash` when present, which the discussion thread uses to establish the poster's identity;
   - the previous and next record links (`record pager`, see [calculations.md](calculations.md)), computed from the session history of the list the reader came from.
6. The page renders inside the portal layout with the record sidebar, the document body and the discussion thread anchor.

**The record-page contract**

| Element | Supplied by | Behavior |
|---|---|---|
| Breadcrumb | this domain, extended per document domain | Home icon, then the list page of the document kind, then the document reference. |
| Previous / next controls | this domain | Shown only when the session history contains the current record and the record exposes a portal web address; each link carries the neighbour's own token. |
| Sidebar | the document domain, using the shared sidebar frame | A title block (usually the total amount), an entry block (action buttons, a scroll-spy navigation, domain-specific blocks), and a "powered by" credit. |
| Action buttons | the document domain | For example `Sign & Pay`, `Accept & Sign`, `Accept & Pay`, `Pay Now`, `Reject`, `View Details`, `Download`. |
| Report links | this domain's address builder | `get_portal_url(report_type = <one of the three kinds>, download = <true or false>)`. |
| Discussion thread | this domain | An anchor carrying the model, the record identifier, the token, the recipient identifier, the signed identity, the page size (ten messages by default), whether the composer is allowed and whether the two-column layout is used. |
| Signature panel | this domain | Rendered when the document domain asks for it (section 20). |
| Flash banners | this domain | Rendered from `error`, `warning`, `success` and the domain-specific `message` value. |

---

## 12. Rendering and downloading a document report

**Actors**: anyone who reached the record page.

**Steps**

1. The reader follows a link carrying `report_type` and optionally `download=true`.
2. The endpoint resolves the record (`PORT-RULE-020`).
3. The endpoint validates the report kind against the three supported values; anything else is refused with `Invalid report type: <value>` (`PORT-RULE-021`).
4. When the record carries a company field and the set spans more than one company, the render is refused with `Multi company reports are not supported.` (`PORT-RULE-022`).
5. When the record carries a company field with exactly one company, the rendering company is switched to it, so that the layout, the logo and the paper format of that company are used.
6. The report action is rendered with elevated rights for the requested kind, passing the record identifiers and the report kind as render data.
7. The response headers are built (`report headers`, see [calculations.md](calculations.md)):
   - media type: the portable-document media type (`application/pdf`) when `report_type` is the portable-document kind, and the rich-text media type (`text/html`) otherwise;
   - content length;
   - for a portable document only, a content disposition naming the file `<sanitized report base name>.pdf`, as an attachment when `download` is true and inline otherwise.

**Domain-specific variations**

| Domain | Variation |
|---|---|
| Accounts Receivable | When the request asks for a downloadable portable document and the invoice is posted, the **legal** documents are served instead of a fresh render: one document is served directly, several are packed into a compressed archive named after the invoice. Otherwise, when no generated document exists yet, the render is switched to the pro-forma variant. When the customer's language is written right to left, the rendering language is switched to it. The report definition of the customer's preferred template is used when set. |
| Purchasing | The quotation report is used while the order is a request for quotation, and the order report once it is confirmed. |
| Projects and Tasks | The task report is the timesheet report of the task when the timesheet capability is installed; otherwise the request is refused with `There is nothing to report.` |

---

## 13. Loading the discussion thread of a portal page

**Actors**: anonymous visitor with a token or a signed identity, external user, or employee.

**Steps**

1. The page anchor triggers an initialization call to `/portal/chatter_init` with the model, the record identifier and whichever of `token`, `pid` and `hash` the page carries.
2. The server seeds the store with the reader's own user data, and marks the reader as a publisher when they belong to the website editor group.
3. The server resolves the thread (`_get_thread_with_access`). When it cannot, the response contains only the seed.
4. The server resolves whether the reader may post (`PORT-RULE-060`): the thread is re-resolved with the permission that the model declares for posting.
5. For an anonymous reader, the portal Contact is resolved (`get_portal_partner`); when found, it is published to the store with its active flag, its picture, its main user's Contact and share characteristic, and its name. Reacting is allowed only when both the posting permission and a resolved Contact are present.
6. The response reports whether the reader may react, whether the reader has direct read access on the thread, and the thread's display name.
7. The page then calls `/mail/chatter_fetch` with the model, the record identifier, the token and the paging parameters.
8. The server resolves the thread by token only for this call. When it cannot, the response is "not found".
9. When the token resolves a portal Contact, that Contact and the thread are put into the rendering context so that the reader is recognized as the author of their own messages.
10. The server intersects three conditions (`PORT-RULE-061`):
    - the extra condition of the page (empty by default; the rating bridge adds a filter on the rating value when the caller asks for one);
    - the publishable-message condition: the filter of `website_message_ids` for this model, restricted to this record, intersected with the share condition `is_internal = false AND subtype IS SET AND subtype.internal = false`;
    - the non-empty condition: `body NOT IN (empty, the marker of an emptied edited message) OR attachments IS NOT EMPTY`. The rating bridge widens it with `OR rating_value IS SET`, so that a bare star rating with no text is still shown.
11. The messages are fetched with elevated rights, justified by the fact that thread access was validated above and the condition is restricted to shareable messages only.
12. The messages are projected with the portal projection, passing the call parameters as options so that the author picture addresses carry the token or the signed identity.

**Postconditions**: every reader of a portal page, signed in or not, sees exactly the public messages of the record and never an internal note, including employees reading the portal page.

---

## 14. Posting a message from a portal page

**Actors**: anonymous visitor with a signed identity or a token, external user, or employee.

**Preconditions**: the discussion thread anchor was rendered with the composer allowed.

**Steps**

1. The reader types a body, optionally attaches files (section 15), and submits.
2. The front end posts to the shared message-posting endpoint with the model, the record identifier, the body, the attachment identifiers and whichever of `token`, `pid` and `hash` the page carries.
3. The server resolves the thread for the posting permission (`PORT-RULE-060`).
4. **Author resolution** (`PORT-RULE-062`): when the acting user is the anonymous public user, the portal Contact is resolved from the signed identity first and from the token second; when one is found, the message author is forced to that Contact. When no Contact is found, the post proceeds with no author, which the projection renders as a message without an author.
5. The message is created as a comment with the public comment subtype, so that it satisfies the share condition and is visible on the page.
6. The page reloads the thread.

**Editing an own message**

1. The reader presses edit on a message.
2. The server decides whether the edit is allowed (`PORT-RULE-063`): when the acting user is the anonymous public user and the message names a model and a record, the portal Contact is resolved from the supplied signed identity, recipient identifier and token; the edit is allowed when that Contact equals the message author. Otherwise the inherited decision applies.
3. On success the body is replaced; a message whose body is emptied and that has no attachment disappears from the portal list because of the non-empty condition.

---

## 15. Attaching a file to a portal message and removing a pending attachment

**Actors**: the same as section 14.

**Steps**

1. The reader picks files in the composer. Each file is uploaded through the shared attachment upload endpoint and is created in a **pending** state: it is attached to the message-composition model with a record identifier of zero.
2. The composer shows each pending attachment with a delete control.
3. Pressing delete calls `/portal/attachment/remove` with the attachment identifier and, when the page has one, the token.
4. The server resolves the attachment through the shared access check. When it fails, it refuses with `The attachment does not exist or you do not have the rights to access it.` (`PORT-RULE-070`).
5. The server verifies the pending state: the attachment must belong to the message-composition model and its record identifier must be zero; otherwise it refuses with `The attachment %s cannot be removed because it is not in a pending state.` (`PORT-RULE-071`).
6. The server verifies that no message references the attachment; otherwise it refuses with `The attachment %s cannot be removed because it is linked to a message.` (`PORT-RULE-072`).
7. The attachment row is deleted.
8. When the message is finally posted, the pending attachments are re-parented onto the message by the messaging domain and stop being removable through this endpoint.

---

## 16. Reacting to a message from a portal page

**Actors**: the same as section 14.

**Steps**

1. The reader picks an emoji on a message.
2. The shared reaction endpoint resolves the reaction author: the inherited resolution first; when it yields nothing and the message names a model and a record, the portal Contact is resolved from the signed identity, the recipient identifier and the token, and when one is found it becomes the reaction author and the guest author is cleared (`PORT-RULE-064`).
3. The reaction is stored and the thread is refreshed; the projection groups reactions by content and reports the counts and the names of the reacting Contacts and guests.

---

## 17. Marking a message internal or public

**Actors**: signed-in user (the endpoint requires a session).

**Steps**

1. The reader toggles the internal marker on a message shown in a portal thread.
2. The server writes the new value of the internal flag on the message, subject to the message write permission of the messaging domain.
3. The server returns the resulting value.

**Postcondition**: a message marked internal disappears from every portal thread, because the share condition excludes internal messages, and reappears in the back-office thread only.

---

## 18. Rating a record from the portal

**Actors**: external user or employee. Anonymous visitors do not see the composer.

**Preconditions**: the rating bridge is installed and the page embeds the star widget.

**Steps**

1. The page renders the star widget with the record's rating average and rating count, plus the composer button. The button label is `Review` in the full-screen variant, `Edit Review` when the reader already posted a rating on the record, and `Add Review` otherwise.
2. The composer is offered only when the page does not disable it and the reader is not the anonymous public user (`PORT-RULE-080`).
3. The reader picks a star value and optionally types a body and attaches files.
4. Submitting posts a message on the record carrying the rating value, through the messaging domain's posting operation. The messaging domain creates the Rating row with elevated rights, because an external reader has no permission on ratings.
5. The page reloads the widget from the response: the new average, the new count and the reader's own rating value; the submit label becomes `Update review`.
6. The projected messages of the thread carry the rating value when the caller asks for it, and the per-record rating statistics: the average, the total and the percentage of ratings at each of the five levels.

---

## 19. Publishing a reply under a rating

**Actors**: website editor, or any user with write permission on the rated record.

**Steps**

1. The reader opens a public page showing a rating and types a reply.
2. The front end calls `/website/rating/comment` with the rating identifier and the comment text. The endpoint requires a session.
3. The server fetches the rating; when it does not exist it answers `{ "error": "Invalid rating" }` (`PORT-RULE-081`).
4. The server writes the comment. Before the write reaches storage, the stamping rule runs (`PORT-RULE-082`): the guard is evaluated, the publication moment is set to now when not supplied and the publisher is set to the acting user's Contact when not supplied.
5. The guard (`PORT-RULE-083`): when the website editor group exists and the acting user belongs to it, the write is allowed; otherwise the acting user must have write permission on every rated record, and the refusal is `Updating rating comment require write access on related record`.
6. The server returns the formatted reply: the publisher picture address, the comment, the formatted publication moment, the publisher identifier and the publisher name.

---

## 20. Signing a document from the portal

**Actors**: anonymous visitor with a token, or external user.

**Preconditions**: the document domain declares that the document must be signed.

**Steps (the shared part)**

1. The record page renders the signature panel with the address to call, a default name, the drawing mode (draw by hand, generate from the typed name, or upload an image), the button label, the drawing area ratio, the kind (full signature or initials) and the pen colour.
2. The reader types their name and draws, types or uploads a signature, then presses the button.
3. The panel posts the name and the signature image to the address supplied by the document domain, which is `get_portal_url(suffix = "/accept")` for a sales order.

**Steps (the sales order case, owned by [Sales](../sales/README.md), described here because it is the contract every other domain follows)**

4. The endpoint resolves the order (`PORT-RULE-020`); on failure it answers `{ "error": "Invalid order." }`.
5. When the order does not require a signature, it answers `{ "error": "The order is not in a state requiring customer signature." }`.
6. When no signature image was supplied, it answers `{ "error": "Signature is missing." }`.
7. It writes `signed_by = <name>`, `signed_on = <now>` and `signature = <image>` on the order, then flushes so that the stored signature is visible to the report renderer. A malformed image answers `{ "error": "Invalid signature data." }`.
8. When the order does not additionally require a payment, the order is confirmed.
9. The order report is rendered as a portable document including the signature and posted as an attachment on a public comment with the body `Order signed by <name>`, authored by the order's Contact for an anonymous signer and by the acting user's Contact otherwise.
10. The response asks the page to reload at `get_portal_url(query_string = "&message=sign_ok")`, with `&allow_payment=yes` appended when a payment is still required.
11. The reloaded page shows one of three banners: `Your order has been confirmed.` when the order is now confirmed, `Your order has been signed but still needs to be paid to be confirmed.` when a payment remains, or `Your order has been signed.` otherwise.

**Rejecting instead of signing (sales order)**

1. The reader presses `Reject` and types a reason in the required text area.
2. The endpoint resolves the order, then, when the order still requires a signature **and** a reason was typed, cancels the order and posts the reason as a public comment authored by the order's Contact for an anonymous visitor and by the acting user's Contact otherwise, and redirects to the order page.
3. Otherwise it redirects to `get_portal_url(query_string = "&message=cant_reject")`, which renders the "cannot reject" banner.

---

## 21. Paying a document from the portal

**Actors**: anonymous visitor with a token, or external user.

This workflow is owned by [Payment Providers](../payment-providers/README.md); the portal contributes the page frame and the address contract. The record page of a payable document computes the payment context and passes it to the embedded payment form:

| Value | Meaning |
|---|---|
| amount | The amount to collect, decided by the document domain (full amount, remaining amount, or the required prepayment). |
| currency | The document currency. |
| paying Contact | The acting user's Contact when signed in, otherwise the document's own Contact. |
| compatible providers, methods and saved methods | Resolved with elevated rights for that company, Contact, amount and currency. |
| company mismatch | Whether the paying Contact's company differs from the document's company; when it does, the page shows the "switch company" warning instead of the form. |
| transaction address | `get_portal_url(suffix = "/transaction")`. |
| landing address | `get_portal_url()`. |
| token | The document token, created if needed. |

The transaction endpoint resolves the document through the shared access check and answers `The access token is invalid.` when the token does not match.

---

## 22. Viewing and editing the account details

**Actors**: signed-in user (external or employee).

**Steps**

1. The browser requests `/my/account`, optionally with `redirect=<address>`.
2. The server builds the rendering values: the shared layout values, plus the address-form values computed for the acting user's **own** Contact with the "this address serves as both billing and delivery" flag set to true and the success address set to the `redirect` parameter (default `/my`), plus the page name `my_details`.
3. The response carries two protective headers: framing restricted to the same origin, expressed both as a frame-options header and as a content-security policy.
4. The page renders inside the account layout, which shows the heading `My account`, the form in the main column and the account sidebar in the side column (an offcanvas panel on a narrow screen).
5. The form shows: `Your name`, `Email`, `Phone`; then, because the address serves as a billing address, `Company Name` and the tax identification number under its reproduced label `VAT`; then `Street and Number`, `Apartment, suite, etc.`, `City`, `Zip Code`, `Country` and `State / Province`. The postal code is rendered before or after the city according to the country's address layout.
6. Hidden inputs carry the request-forgery token, the address kind, the "use delivery as billing" flag, the parent identifier, the Contact identifier, the success address and the base list of required inputs (`name,email`).
7. On loading, and again whenever the country changes, the form calls `/my/address/country_info` (section 23.3) and rewrites its inputs.
8. Pressing `Save Address` runs the browser's own required-field validation, then posts the whole form to `/my/address/submit` (section 23.4).
9. On success the browser is redirected to the success address; on failure the offending inputs are marked and the error messages are listed above the form.

**The account sidebar**

| Block | Content |
|---|---|
| Identity | The person's picture, name and company name. |
| Contact details | The Contact rendered with its electronic mail address, telephone and postal address. |
| `Edit information` | A link to `/my/account`. |
| `Your contact` | Shown only when an assigned representative was resolved: the representative's name, electronic mail address, telephone and city. |
| Loyalty cards | Contributed by [Loyalty and Promotions](../loyalty-and-promotions/README.md): one card per programme with the balance, and the note `A reward is waiting for you` when a reward is reachable. |

---

## 23. Managing the address book

### 23.1 Listing the addresses

1. The browser requests `/my/addresses`.
2. The server collects (`PORT-RULE-090`):
   - **billing addresses**: the Contacts that are descendants of the acting person's commercial entity and whose address kind is `invoice` or `other`, plus the commercial entity itself, plus the acting person's own Contact;
   - **delivery addresses**: the Contacts matching the commercial entity's delivery-address filter (descendants of the commercial entity whose kind is `delivery` or `other`, plus the commercial entity itself), plus the acting person's own Contact.
   Both lists are read with elevated rights and ordered by identifier descending.
3. When the acting person is **not** their own commercial entity (they are a child of a company), the commercial entity is removed from whichever of the two lists it does not qualify for: it is removed from the billing list when its mandatory billing fields are not all filled, and from the delivery list when its mandatory delivery fields are not all filled. The reason is that a child may not edit the company's address, so an incomplete company address would be a dead end.
4. The page decides whether the two lists are merged: the "same as delivery address" switch is checked when **no** address in the billing list has the kind `invoice`.
5. The page renders two sections, `Delivery address` and `Billing address`, each with an `Add Address` control, and one card per address.

**The address card**

| Element | Rule |
|---|---|
| Body | The Contact rendered with its name and postal address. |
| Badge | `Main Address` for the acting person's own Contact, else `Delivery Address` for kind `delivery`, else `Billing Address` for kind `invoice`. |
| Edit control | Shown when `_can_be_edited_by_current_customer` is true. For the person's own Contact it points at `/my/account?redirect=/my/addresses`; for any other address it points at the address form with the address kind and the Contact identifier. |
| Remove control | Shown when the address is editable, is not the currently selected one and is not the person's own Contact. |

6. Toggling the "same as delivery address" switch hides or shows the billing section and rewrites the delivery `Add Address` link so that the flag is carried into the form.

### 23.2 Opening the address form

1. The browser requests `/my/address` with `partner_id` (optional), `address_type` (`billing` by default, or `delivery`) and `use_delivery_as_billing` (`false` by default).
2. When a Contact identifier is supplied, the Contact is read with elevated rights and `_can_be_edited_by_current_customer` is evaluated; a false result is answered with a forbidden response (`PORT-RULE-091`).
3. The form values are computed:
   - when a Contact was supplied: the country and country subdivision come from that Contact, and the "may edit the tax identification number" flag is that Contact's predicate;
   - when no Contact was supplied (creation): the country comes from the current customer's country, or from the acting user's country when the customer has none; the country subdivision comes from the current customer; the "may edit the tax identification number" flag is true when there is no current customer, and otherwise is the current customer's predicate evaluated only when the form targets the current customer itself;
   - `is_commercial_address` is true when there is no current customer or when the edited Contact is the commercial entity;
   - `is_main_address` is true when there is no current customer or when the edited Contact is the current customer;
   - the "update the commercial address elsewhere" link is offered only when the current customer **is** its own commercial entity, and then points at `/my/account?redirect=/my/addresses`;
   - `_can_edit_country` is true when the Contact has no country yet, or when its predicate is true;
   - the country list is every country, read with elevated rights;
   - the country subdivision list is the chosen country's subdivisions;
   - the postal code is placed before the city when the country's address layout puts it there;
   - the discard address is the success address when one was supplied, otherwise `/my/addresses`.
4. The page renders `Billing address` or `Delivery address` as its heading, the same field set as the account form, and the footer with `Discard` and `Save Address`.
5. The tax identification number input is read-only when the edited address is not the commercial address, or when the current customer already has a number and may not edit it; below it the page explains why: `The tax identification number can only be updated on the company account.` when the commercial entity is a company and this is the main address, or the "documents have been issued" sentence when the number is frozen. On a child address that is neither the commercial address nor the main address, the page adds `The company name and tax identification number can only be updated on your main account.` followed, when the link is available, by `If you want to modify it, update your account details.`
6. The country selector is disabled and greyed when the country may not be edited, and the page explains: `Changing country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`

### 23.3 Reacting to a country change

1. The browser calls `/my/address/country_info/<country>` with the address kind. The endpoint is open to anonymous visitors.
2. The server answers with: the ordered list of address field names of that country's layout, whether the postal code precedes the city, the list of the country's subdivisions as triples (identifier, name, code), the country's telephone prefix, and the list of mandatory field names for the requested address kind.
3. The form sets the telephone placeholder to `+<prefix>` when the prefix is not zero, and empties it otherwise.
4. The form rebuilds the subdivision selector: when the country has subdivisions it fills and shows it; otherwise it empties and hides it. On the very first render the selector is left alone unless it is empty, because the server already rendered it.
5. The form reorders the postal code and the city, and shows or hides the street, postal code and city inputs according to the country's layout.
6. The form clears the required marker on inputs that are no longer mandatory (unless they are in the form's own base required list) and sets it on the newly mandatory ones. A label whose input is not required carries the "optional" styling.

### 23.4 Submitting an address

**Actors**: signed-in user.

1. The browser posts the whole form to `/my/address/submit` with `partner_id` when updating.
2. When a Contact identifier is supplied, the Contact is read with elevated rights and `_can_be_edited_by_current_customer` is evaluated; a false result is answered with a forbidden response (`PORT-RULE-091`).
3. **Parse** (`PORT-RULE-092`): every submitted value is trimmed when it is text. A value whose input name is both a Contact field and a member of the front-end writable set is converted to the field's internal representation and kept, **even when it is empty**, so that a field can be cleared. A relation to one record given as a run of digits is converted to that identifier. Every other submitted value is kept aside as extra form data. Finally, when `zipcode` was submitted and `zip` was not supplied or was empty, the postal code takes the value of `zipcode`.
4. **Validate** (section 23.5). When any error message was produced, the response is `{ "invalid_fields": [...], "messages": [...] }` and nothing is written.
5. **Branch A, creation** (no Contact identifier):
   - complete the values (`PORT-RULE-093`): the language is the request language; the company is the current customer's company; the address kind is `other` when the form asks to use the delivery address as the billing address, `invoice` when the kind is billing and `delivery` when the kind is delivery; the parent is the commercial entity when that entity is active (so that a new address is never attached to the archived anonymous public Contact);
   - create the Contact with elevated rights, with change tracking disabled and with the tax-identification-number check suppressed because it was already run during validation;
   - when the telephone validation capability is installed, run its formatting rule on the new Contact.
6. **Branch B, update** (a Contact identifier was supplied) and the submitted values differ from the stored ones (compared field by field in the internal representation, treating two empty values as equal):
   - when the submitted name, trimmed, equals the stored name, trimmed, the name is removed from the values before writing. The reason is that writing the name has a side effect on the holder name of the person's bank accounts, which must not fire when the name did not actually change (`PORT-RULE-094`);
   - write the values;
   - when the telephone was among the written values and the telephone validation capability is installed, run its formatting rule.
7. **Company name propagation** (`PORT-RULE-095`): when a company name was submitted, the Contact is not its own commercial entity and the commercial entity is a company, then the Contact's own company-name field is cleared and, when a non-empty name was submitted that differs from the commercial entity's name, the commercial entity is renamed.
8. The extra form data is handed to the extension point, which the shipped behavior leaves empty and other domains use (for example the electronic-invoicing preferences on the account page).
9. The response is `{ "redirectUrl": <success address> }`, default `/my/addresses`.

### 23.5 Validating an address

The validation produces three results: the set of invalid input names, the set of missing input names, and the ordered list of messages. The response merges the two sets into `invalid_fields`.

**When a Contact is being updated**

1. Detect the three changes: a name change (a name was submitted, the Contact already had one and the submitted value differs from the trimmed stored one), a country change, an electronic-mail-address change.
2. **Country guard** (`PORT-RULE-100`): when the country changed and `_can_edit_country` is false, mark the country invalid and add:
   `Changing your country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`
3. **Employee guard** (`PORT-RULE-101`): when the name or the electronic mail address changed and **not every** user of the Contact is an external user, mark the changed inputs invalid and add:
   `If you are ordering for an external person, please place your order via the backend. If you wish to change your name or email address, please do so in the account settings or contact your administrator.`
4. **Child-address commercial guard** (`PORT-RULE-102`): when the Contact is not its own commercial entity, then for every commercial field that was submitted:
   - when the submitted value differs from the stored one and at least one of the two is non-empty, mark that field invalid and add `The %(field_name)s is managed on your company account.` when the commercial entity is a company, or `The %(field_name)s is managed on your main account address.` otherwise, where the placeholder is the human label of the field;
   - otherwise remove the field from the values so that it is not written.
   In addition, when the Contact is not the current customer, the company name is removed from the values.
5. **Commercial-entity tax identification guard** (`PORT-RULE-103`): when the Contact **is** its own commercial entity and a tax identification number was submitted, the Contact already had one, the submitted value differs and `can_edit_vat` is false, mark it invalid and add:
   `Changing the tax identification number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`

**When a Contact is being created**: `is_commercial_address` is true exactly when there is no current customer.

**Always**

6. **Electronic mail format** (`PORT-RULE-104`): when an address was submitted and it does not match the single-address pattern, mark it invalid and add `Invalid Email! Please enter a valid email address.`
7. **Tax identification number check** (`PORT-RULE-105`): when a number was submitted, the accounting capability is installed and the number is not already marked invalid, build a throwaway Contact carrying only the country and the number and run the accounting domain's number check; a failure marks the number invalid and adds the checker's own message verbatim.
8. **Required set** (`PORT-RULE-106`): start from the comma-separated list submitted by the form. Add the mandatory delivery fields when the kind is delivery or the form uses the delivery address as the billing address. Add the mandatory billing fields when the kind is billing or the form uses the delivery address as the billing address; in that case, when the address is not the commercial address, remove from the required set every commercial field that was not submitted. Then, when **any** of the common address fields was filled, add all of them, so that a partially typed address is completed rather than stored half filled.
9. **Missing check**: every required input whose value is empty is added to the missing set; when the set is not empty, add `Some required fields are empty.`

**The mandatory field sets**

```
common_address_fields(country) = { street, city, country }
                               ∪ { state } when country.state_required
                               ∪ { zip }   when country.zip_required

mandatory_billing_fields(country)  = { name, email }
                                   ∪ { phone } ∪ common_address_fields(country)   when the page needs an address
mandatory_delivery_fields(country) = { name, email }
                                   ∪ { phone } ∪ common_address_fields(country)   when the page needs an address
```

The "page needs an address" switch is true for every shipped portal page; a storefront checkout of intangible goods may turn it off, in which case only the name and the electronic mail address are mandatory.

### 23.6 Archiving an address

1. The browser calls `/my/address/archive` with the Contact identifier. The endpoint requires a session.
2. The Contact is read with elevated rights and must exist and satisfy `_can_be_edited_by_current_customer`; otherwise the response is forbidden (`PORT-RULE-091`).
3. When the Contact **is** the acting user's own Contact, the call is refused with `You cannot archive your main address` (`PORT-RULE-097`).
4. Otherwise the Contact is archived.
5. The browser reloads the page.

Note the combined effect: a personal Contact of a colleague (kind `contact`) fails the editability predicate and is therefore refused; the acting person's own Contact passes the predicate but is refused by the explicit guard; only `invoice`, `delivery` and `other` addresses of the company tree can be archived.

---

## 24. Changing the password

**Actors**: signed-in user.

**Steps**

1. The browser requests `/my/security`. The server prepares the shared layout values, adds the error-lookup helper, adds whether application keys are allowed (read from the system parameter with elevated rights) and sets the "deletion dialog open" flag to false.
2. The page renders the sections described in [interfaces.md](interfaces.md). The response carries the same two framing-protection headers as the account page.
3. The person fills the current password, the new password and its confirmation and submits the form to the same address with the method that carries a body.
4. The server trims the three values and runs the change (`PORT-RULE-110`):
   - any empty value answers `You cannot leave any password empty.` against that input;
   - a confirmation that differs from the new password answers `The new password and its confirmation must be identical.` against the confirmation input;
   - the change operation of the identity domain is called with the old and the new password; a credential refusal whose message is the generic one is replaced by `The old password you provided is incorrect, your password was not changed.` and is reported against the old-password input; any other credential refusal is reported verbatim against the old-password input; a business refusal (for example `Setting empty passwords is not allowed for security reasons!` or a password-policy refusal) is reported as a section-level message.
5. On success the server recomputes the session token from the new credentials and writes it into the session, so that the person is **not** signed out by their own password change (`PORT-RULE-111`).
6. The page re-renders with the banner `Password Updated!`.

**Password policy**: when the password-policy bridge is installed, the shared layout values additionally carry the configured minimum length; the new-password input receives that minimum as its client-side constraint and a strength meter is rendered next to it. The server-side enforcement belongs to [Identity and Access](../identity-and-access/README.md).

---

## 25. Managing the second authentication factor

**Actors**: external user (this section describes the portal variant; the back-office variant belongs to the identity domain).

**Preconditions**: the second-factor bridge for the portal is installed, which grants the portal group read, write, create and delete on the enrollment wizard (still bounded by the global rule that a person may only touch their own).

**Steps**

1. The security page renders the second-factor section. When the person has no second factor, it shows the status box `Two-factor authentication not enabled` and the button `Enable two-factor authentication`; otherwise it shows `Two-factor authentication enabled`, the link `(Disable two-factor authentication)` and, when trusted devices exist, a table with the columns `Trusted Device` and `Added On`, a delete control per row and a `Revoke All` button.
2. Because an external person may not open back-office screens, the page embeds the enrollment wizard's combined layout as a hidden data island read with elevated rights, and the front end builds the dialog from it.
3. Pressing the enable button opens the enrollment dialog, which runs the identity domain's enrollment: generate a secret, show the provisioning address and its machine-readable image, collect a verification code, verify it and store the secret.
4. Pressing the disable link runs the identity domain's disabling operation, which also revokes every trusted device.
5. Pressing `Revoke All` revokes every trusted device of the person.
6. The invitation that an administrator can send to ask a person to enable a second factor points at `/my/security` for a non-employee.

---

## 26. Managing passkeys

**Actors**: external user.

**Preconditions**: the passkey bridge for the portal is installed.

**Steps**

1. The security page renders the passkey section above the session-revocation section, with a table whose columns are `Name`, `Created`, `Last Used`, a rename control and a delete control, and a button `Add Passkey`.
2. Pressing `Add Passkey` asks for the account password first (the re-authentication gate), then for a name, then runs the browser's registration ceremony against the platform origin, and stores the resulting credential.
3. The rename control renames a passkey the person owns.
4. The delete control deletes a passkey the person owns.
5. A person may only act on their own passkeys: writing on another person's passkey is refused by the identity domain's permission layer.
6. Signing in with a passkey does not additionally ask for a second factor.

---

## 27. Revoking all other sessions

**Actors**: signed-in user.

**Steps**

1. The person presses `Log out from all devices` in the section `Revoke All Sessions`.
2. The front end calls the identity domain's revoke-all operation for the person's own user.
3. That operation is protected by the re-authentication gate: it answers with an identity-check request instead of doing the work.
4. The front end opens the dialog `Security Control` showing `Please enter your password to confirm you own this account`, a password input and a `Forgot password?` link pointing at the password-reset page, with the confirmation button labelled `Confirm Password`.
5. The person types their password; the front end records the chosen check method as the password and runs the check, which replays the original operation on success and reports `Check failed` on the input on failure.
6. Every device of the person except the current one is revoked.
7. The page reloads.

---

## 28. Managing application keys

**Actors**: external user, when the setting allows it and the page is opened in developer mode.

**Preconditions**: the section is rendered only when the request is in developer mode **and** the system parameter allows external application keys (`PORT-RULE-120`).

**Steps**

1. The page renders the section `Developer Application Keys` with a documentation link, a table with the columns `Description`, `Scope`, `Added On` and `Expiration Date`, a delete control per row and a `New Application Key` button.
2. Pressing the button first calls the identity domain's key wizard operation purely to trigger the re-authentication gate, so that the portal behaves like the back office and asks for the password before showing the form.
3. The front end loads the duration choices of the key wizard and removes the "custom date" choice, because an external person may not pick an arbitrary expiration.
4. The dialog `New Application Key` asks for a description (`What's this key for?`, with the explanation that the description is the only way to identify the key later) and a duration, with the note `The key will be deleted once this period has elapsed.`
5. Confirming creates the wizard row with the description and the duration, then calls the key-generation operation, which passes through the re-authentication gate again.
6. The generated key is shown once in the dialog `Application Key Ready` with the text `Here is your new application key, use it instead of a password for programmatic access. Your login is still necessary for interactive usage.` and the warning `Important: The key cannot be retrieved later and provides full access to your user account, it is very important to store it securely.`
7. Closing the dialog reloads the page.
8. The delete control calls the key-removal operation through the same gate and reloads the page.
9. The permission relaxation is what makes all of this possible: an external person's key creation is allowed only when the setting is on (`PORT-RULE-121`).

---

## 29. Requesting the deletion of the account

**Actors**: external user. The section is rendered only for members of the portal group.

**Steps**

1. The person presses `Delete Account`, which opens the dialog `Are you sure you want to do this?` containing:
   - the explanation `Disable your account, preventing any further login.` and the warning `This action cannot be undone.`;
   - step one: `Enter your password to confirm you own this account` with a required password input;
   - step two: `Confirm you want to delete your account by copying down your login (<login>).` with a required text input;
   - a checkbox, checked by default, labelled `Put my email and phone in a block list to make sure I'm never contacted again`;
   - the buttons `Delete Account` and `Cancel`.
2. Submitting posts the password, the typed login and the block-list choice to `/my/deactivate_account`.
3. The server prepares the security page values with the "deletion dialog open" flag set to true, so that a failed attempt re-renders the page with the dialog open.
4. **Confirmation check** (`PORT-RULE-130`): when the typed value differs from the person's login, the error `validation` is recorded and the page re-renders with the message `You should enter "<login>" to validate your action.` under the text input. Nothing else happens.
5. **Credential check** (`PORT-RULE-131`): the password is verified as an interactive credential. A refusal records the error `password` and the page re-renders with `Wrong password.` under the password input.
6. **Deactivation** (with elevated rights):
   1. A note is logged on the person's Contact: `Archived because <acting user name> (#<acting user identifier>) deleted the portal account`.
   2. When the block-list choice is on, the electronic mail addresses that normalize successfully are remembered, and every formatted telephone number of the person is remembered.
   3. **Refusal guard** (`PORT-RULE-132`): every user in the set must be an external user; otherwise the operation is refused with `Only the portal users can delete their accounts. The user(s) <names> can not be deleted.`
   4. For each user: the login is replaced by `__deleted_user_<identifier>_<timestamp>` and the password is set to the empty string, which no credential check can ever satisfy; every application key of the user is revoked.
   5. The user is archived, running as the system robot user because a person may not deactivate themselves; a failure to archive is swallowed.
   6. The Contact is archived; a failure to archive is swallowed.
   7. One User Deletion Request is created per user with `state = todo`.
   8. The remembered electronic mail addresses are added to the electronic mail block list with the reason `Blocked by deletion of portal account <person name> by <acting user name> (#<acting user identifier>)`.
   9. The remembered telephone numbers are added to the telephone block list, each with a note carrying the same reason.
7. The session is closed.
8. The browser is redirected to `/web/login?message=Account deleted!`, with the message value percent-encoded.

**A business refusal** raised anywhere in step 6 (for example the non-external-user guard) records the error under the key `other` and re-renders the page with that message in a red banner inside the dialog.

**Postconditions**: the person can no longer sign in with any credential; their identity is still present in the database but archived and renamed; a deletion request is queued.

---

## 30. Processing the account deletion queue

**Actors**: the scheduler, running as the system robot user.

**Preconditions**: the scheduled action `Base: Portal Users Deletion` is active. It runs once a day with priority eight and processes fifty requests per run.

**Steps**

1. Search every User Deletion Request in state `todo`.
2. Split off the requests whose user no longer exists and set them to `done` immediately; report that many as progress, with the remaining count as the work still to do.
3. For each of the first fifty remaining requests, in order:
   1. Take an exclusive row lock on the request and re-read its state; skip it when another run already took it.
   2. Remember the user, the user's name, the user's Contact and the name of the person who asked for the deletion.
   3. **Delete the user.** On success, write an audit line `User #<identifier> '<name>', deleted. Original request from '<requester>'.`, set the request to `done` and report one unit of progress.
      On failure, roll back the current work, write an error line `User #<identifier> '<name>' could not be deleted. Original request from '<requester>'. Related error: <error>`, set the request to `fail`, report one unit of progress, and continue with the next request when the run still has budget, or stop when it does not.
   4. **Delete the Contact.** On success, write an audit line `Partner #<identifier> '<name>', deleted. Original request from '<requester>'.` and stop the run when the budget is exhausted.
      On failure (for example the Contact is referenced by an invoice), roll back the current work, write a warning line `Partner #<identifier> '<name>' could not be deleted. Original request from '<requester>'. Related error: <error>`, and stop the run when the budget is exhausted. The request stays `done`: the account is gone and only the Contact remains, archived.

**Postconditions**: the user row is removed; the Contact row is removed when nothing references it and stays archived otherwise; the request records the outcome.

---

## 31. Switching the front-end language

**Actors**: any visitor.

**Steps**

1. The layout renders the language selector only when more than one language is published on the front end.
2. The selector shows the active language's name (the part after the last slash of its full name) or, in the compact variant, the uppercase first segment of its address code.
3. Each entry links to the current page's address rewritten for that language, with the language prefetch flag and the "force the default language" flag set, so that a link to the default language is produced without a prefix.
4. Following an entry reloads the page in that language. The rendering language of every value computed by the server, including the labels of filters and sorts, follows.

The layout also sets the text direction of the whole document from the active language's direction, and adds a right-to-left marker class to the page wrapper when the direction is right to left.

---

## 32. Unfollowing a document from a notification link

**Actors**: the recipient of a notification message.

**Steps**

1. The recipient follows the unfollow link contained in a notification. The link carries the model, the record identifier, the Contact identifier and a link signature. The endpoint accepts anonymous requests and is exempt from request-forgery protection, because it is followed directly from a mail client with an unpredictable session.
2. The signature is recomputed from the request path and its parameters minus the signature itself, and compared in constant time. A mismatch, or a record that does not exist, is refused with `Non existing record or wrong token.`
3. The Contact is removed from the record's followers, with elevated rights.
4. The confirmation page is rendered inside the **portal** layout (this domain replaces the plain public layout with the portal one), showing the record's display name and the human name of its model, and a link back to the record only when the reader is signed in and may read it.

---

## 33. State tables

The complete machines — every state with its stored value, label and meaning, every transition with
its guards and its exact refusal message, and a diagram for each — are in
[state-machines.md](state-machines.md). The tables below are the short form used by the procedures of
this file.

### 33.1 Address validity of an invitation line (`email_state`)

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| any | the address on the line is edited | the address does not normalize | `ko` | The Grant Access and Re-Invite buttons disappear; the red cross icon is shown. |
| any | the address on the line is edited | the address normalizes and another user already has it as a login, the website test of `PORT-RULE-175` included when the website capability package is installed | `exist` | The Grant Access and Re-Invite buttons disappear; the struck-through person icon is shown. |
| any | the address on the line is edited | the address normalizes and no other user has it, or every other user that has it belongs to an incompatible website under `PORT-RULE-175` | `ok` | The Grant Access or Re-Invite button appears; the green check icon is shown. |

The detection itself runs once for the whole line list through three extension points (`_get_similar_users_domain`, `_get_similar_users_fields` and `_is_portal_similar_than_user`); their base behavior is in sections 4.3.9 to 4.3.11 of [entities.md](entities.md), the per-website override in section 4.6 of the same file, and the worked numbers in section 7 of [calculations.md](calculations.md).

### 33.2 Access state of an invitation line

See section 4.5 of [entities.md](entities.md) for the full transition table of (`is_portal`, `is_internal`).

### 33.3 User Deletion Request state

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| (none) | the person confirms the deletion dialog | the person is an external user | `todo` | The user is renamed and emptied of credentials, its application keys are revoked, the user and the Contact are archived, the block lists are updated. |
| `todo` | the scheduled run starts | the referenced user no longer exists | `done` | None. |
| `todo` | the scheduled run processes the request | the user deletion succeeds | `done` | The user row is removed; the Contact removal is attempted afterwards. |
| `todo` | the scheduled run processes the request | the user deletion fails | `fail` | The work is rolled back; an error line is written; the user stays archived. |
| `done` | the Contact deletion fails | not applicable | `done` | A warning line is written; the Contact stays archived. |

### 33.4 Sharing decision

| Document has a token | Sign-up scope | Recipient has a user | Path taken |
|---|---|---|---|
| yes | any | any | Plain token link. |
| no | `b2b` | any | Plain token link (created on the fly by the address builder). |
| no | `b2c` | yes | Plain token link. |
| no | `b2c` | no | Individual sign-up link pointing at the generic redirection endpoint with the document model and identifier. |
