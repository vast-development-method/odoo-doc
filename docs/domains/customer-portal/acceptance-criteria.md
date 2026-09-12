# Acceptance criteria of the Customer Portal

Scenarios a replacement must pass, written as Given / When / Then with concrete values. Each scenario is independently verifiable. The rule or formula it exercises is named in the heading.

Unless a scenario says otherwise, the fixture is: one company named `Acme` whose country is Belgium; a Contact manager named `Maya`; an employee named `Ivan` who is not a Contact manager; a Contact `Testing Partner` with the address `testing_partner@example.com`; an external user `Willis` whose login is `portal_user`; and an anonymous visitor.

---

## 1. Granting, revoking and re-inviting

### PORT-AC-001 - A Contact with no user receives an account and an invitation (`PORT-RULE-004`, `PORT-RULE-005`)
- **Given** the Contact `Testing Partner` has no user
- **And** Maya opens the invitation dialog on that Contact and changes the address on the line to `first_email@example.com`
- **When** Maya presses `Grant Access`
- **Then** a user exists whose login and address are both `first_email@example.com`, which is linked to that Contact
- **And** that user is an external user
- **And** the Contact's address is now `first_email@example.com`
- **And** exactly one message has been sent, from the acting user's Contact to `Testing Partner`

### PORT-AC-002 - Granting access to a public user swaps the groups (`PORT-RULE-173`)
- **Given** a user `Public user` whose login is `public_user` and whose only group is the public group
- **And** Maya opens the invitation dialog on that user's Contact; the line shows neither an external user nor an employee
- **When** Maya sets the address to `new_email@example.com` and presses `Grant Access`
- **Then** the line shows an external user and not an employee
- **And** the user is an external user and is no longer a public user
- **And** both the Contact's address and the user's address are `new_email@example.com`
- **And** one message has been sent to that Contact

### PORT-AC-003 - Revoking archives the user and sends nothing (`PORT-RULE-006`, `PORT-RULE-007`)
- **Given** the situation at the end of `PORT-AC-002`
- **When** Maya presses `Revoke Access` on the same line
- **Then** the line still points at the same user
- **And** that user is archived
- **And** the line shows neither an external user nor an employee
- **And** the user is still a member of the portal group
- **And** no message has been sent

### PORT-AC-004 - An employee line cannot be managed (`PORT-RULE-001`, `PORT-RULE-006`)
- **Given** an employee `Internal user` whose login is `internal_user`
- **And** Maya opens the invitation dialog on that employee's Contact
- **Then** the line shows an employee
- **When** Maya presses `Grant Access`
- **Then** the operation is refused with `The partner "Internal user" already has the portal access.` and no message is sent
- **When** Maya presses `Revoke Access`
- **Then** the operation is refused with `The partner "Internal user" has no portal access or is internal.`

### PORT-AC-005 - An address already used as a login is detected (`PORT-RULE-003`)
- **Given** an external user `Willis` and an employee whose login is `test_error@example.com`
- **And** Maya opens the invitation dialog on Willis's Contact
- **When** Maya sets the line's address to `test_error@example.com`
- **Then** the line's address state is `exist`
- **And** an attempt to grant or re-invite is refused with `The contact "Willis" has the same email as an existing user`

### PORT-AC-006 - A malformed address is detected (`PORT-RULE-002`)
- **Given** the invitation dialog of the previous scenario
- **When** Maya sets the line's address to `wrong email format`
- **Then** the line's address state is `ko`
- **And** an attempt to grant or re-invite is refused with `The contact "Willis" does not have a valid email.`

### PORT-AC-007 - The dialog is reserved to a Contact manager (`PORT-RULE-031`)
- **Given** the employee `Ivan`, who is not a Contact manager
- **When** Ivan tries to create an invitation session on `Testing Partner`
- **Then** the platform's model-access refusal is raised
- **And** reading `welcome_message` on an existing session as Ivan is refused the same way
- **And** reading the address of a line of that session as Ivan is refused the same way

### PORT-AC-008 - A new external user is created in the Contact's own company (`PORT-RULE-005`)
- **Given** two companies `Acme` and `Beta`
- **And** a Contact whose company is `Beta`
- **And** Maya is working in `Acme`
- **When** Maya presses `Grant Access` on that Contact's line while working in `Acme`
- **Then** the created user's company is `Beta`
- **And** its allowed companies are exactly `Beta`

### PORT-AC-009 - Granting and revoking may be repeated (`PORT-RULE-007`)
- **Given** the Contact `Testing Partner` with no user
- **When** the sequence "grant, revoke" is executed twice in a row on the same line
- **Then** after each grant: the user is active, is an external user, the line shows an external user, and the Contact carries a sign-up type
- **And** after each revocation: the user is archived, is still an external user by group, the line shows no external user, and the Contact carries no sign-up type

### PORT-AC-010 - Re-inviting requires an existing access (`PORT-RULE-008`)
- **Given** the Contact `Testing Partner` with no user
- **When** Maya presses `Re-Invite` on its line
- **Then** the operation is refused with `You should first grant the portal access to the partner "Testing Partner".`

### PORT-AC-011 - Re-inviting invalidates the previous link
- **Given** an invited Contact whose activation link has been captured but not used
- **When** Maya presses `Re-Invite` on its line
- **Then** a second message is sent with a different activation link
- **And** following the first link no longer resolves the Contact

### PORT-AC-012 - The Contact expansion excludes places (`PORT-RULE-030`)
- **Given** a company Contact `Acme Holding` with four children: `Ann` of kind `contact`, `Ops` of kind `other`, `Billing` of kind `invoice` and `Warehouse` of kind `delivery`
- **When** Maya opens the invitation dialog on `Acme Holding`
- **Then** the dialog shows exactly three lines: `Acme Holding`, `Ann` and `Ops`

### PORT-AC-013 - The invitation message carries the welcome text
- **Given** an invitation session whose `welcome_message` is `See you on our portal.`
- **When** Maya grants access to one line
- **Then** the sent message's subject is `Your account at Acme`
- **And** its body contains the sentence `An account has been created for you with the following login: <the address>`
- **And** its body contains a button labelled `Activate Account`
- **And** its body ends with `See you on our portal.`

### PORT-AC-014 - The dialog stores its lines before any action, and every action reopens it (`PORT-RULE-032`, `PORT-RULE-033`)
- **Given** Maya has selected the Contacts `Testing Partner` and `Willis`
- **When** Maya runs the `Grant portal access` entry of the Contact action menu
- **Then** an invitation session is stored first, together with one stored line per Contact, and only then is the dialog opened on that stored session
- **And** every line carries an identifier, so its three state icons and its three action buttons are enabled rather than greyed out
- **When** Maya presses `Grant Access` on the first line
- **Then** the answer is the dialog action of the same session, so the dialog stays open
- **And** both lines are derived again, so the second line now shows the state that the first grant may have changed
- **When** Maya presses `Revoke Access`, then `Re-Invite`, then one of the state icons
- **Then** each of the four answers is again the dialog action of the same session

### PORT-AC-015 - A missing invitation template refuses the send (`PORT-RULE-009`)
- **Given** the shipped message template that carries the invitation body cannot be resolved
- **And** a line whose `email_state` is `ok` and which has no user yet
- **When** Maya presses `Grant Access`
- **Then** the operation is refused with `The template "Portal: new user" not found for sending email to the portal user.`
- **And** no message is sent
- **When** Maya presses `Re-Invite` on a line that already has portal access
- **Then** the operation is refused with the same message

### PORT-AC-016 - The address state is narrowed per website (`PORT-RULE-175`, `PORT-RULE-003`)
- **Given** the Website and Storefront capability package is installed, with two websites `Shop Europe` (identifier 1) and `Shop Asia` (identifier 2)
- **And** the request that opens the dialog is served by `Shop Europe`
- **And** three Contacts with no user: `Eve` linked to `Shop Europe`, `Frank` linked to `Shop Asia`, and `Gina` linked to no website
- **And** three existing users: user 81 with the login `eve@acme.com` on `Shop Asia`, user 82 with the login `frank@acme.com` on `Shop Asia`, and user 83 with the login `gina@acme.com` on `Shop Europe`
- **When** Maya opens the invitation dialog on the three Contacts with those three addresses typed on the lines
- **Then** the candidate search reads `id`, `login` and `website_id` of the users whose login is one of the three addresses and whose website is one of `Shop Europe`, `Shop Asia` or "no website"
- **And** Eve's line is `ok`, because user 81 is on another website
- **And** Frank's line is `exist`, because user 82 is on the same website
- **And** Gina's line is `exist`, because her Contact has no website and user 83 is on the website serving the request
- **When** Maya presses `Grant Access` on Eve's line
- **Then** a second user with the login `eve@acme.com` is created, on `Shop Europe`
- **When** Maya presses `Grant Access` on Frank's line
- **Then** the operation is refused with `The contact "Frank" has the same email as an existing user`

### PORT-AC-017 - The website that serves the request changes the state (`PORT-RULE-175`)
- **Given** exactly the data of `PORT-AC-016`
- **When** the same dialog is opened from a request served by `Shop Asia` instead
- **Then** Eve's line is still `ok` and Frank's line is still `exist`
- **And** Gina's line is now `ok`, because her Contact has no website and user 83 belongs to `Shop Europe`, which is no longer the website serving the request

### PORT-AC-018 - Without the website capability package no website is consulted (`PORT-RULE-175`)
- **Given** the same three Contacts and the same three users, on an installation where the Website and Storefront capability package is **not** installed
- **When** the dialog is opened
- **Then** the candidate search condition is the login membership term alone, and the projection reads `id` and `login` only
- **And** all three lines are `exist`

### PORT-AC-019 - A Contact manager with no permission on Users still sees every line state (`PORT-RULE-174`)
- **Given** Maya is a Contact manager who holds no read permission on Users
- **And** the Contact `Willis` whose only user is archived
- **When** Maya opens the invitation dialog on `Willis`
- **Then** the line shows the archived user, its latest authentication moment, `is_portal` false, `is_internal` false and the address state
- **And** no permission refusal is raised
- **And** the elevated read is bounded to what the state needs: the users linked to the Contacts of the dialog, and the users that already hold one of the typed addresses as a login, projected to `id` and `login` only (plus `website_id` when the website capability package is installed)

---

## 2. Sharing a document

### PORT-AC-020 - Opening the share dialog creates the token
- **Given** a sales order with an empty security token
- **When** Maya opens the share dialog on it
- **Then** the order's security token is no longer empty
- **And** the dialog's link is the absolute address of the generic redirection endpoint carrying the model, the record identifier and that token

### PORT-AC-021 - Every recipient gets a personal link (`PORT-RULE-011`, `PORT-RULE-014`)
- **Given** a sales order with a token, and two recipients `Ann` (identifier 412) and `Bob` (identifier 413)
- **When** Maya presses `Send`
- **Then** two internal notes are posted on the order, one notifying `Ann` only and one notifying `Bob` only
- **And** the link in Ann's note carries `pid=412` and a signature that verifies for Contact 412
- **And** the link in Bob's note carries `pid=413` and a different signature
- **And** both notes carry the private-note subtype, so they appear in the back-office thread of the order
- **And** neither note appears on the portal page of the order
- **And** each recipient nevertheless receives the note by electronic mail

### PORT-AC-022 - Without a token and with free sign-up, an account-less recipient gets a sign-up link (`PORT-RULE-010`)
- **Given** the sign-up scope is `b2c`
- **And** a document of a model that adopts the access mixin whose token column is still empty at the moment the split is decided
- **And** two recipients, `Ann` who has a user and `Zoe` who has none
- **When** Maya presses `Send`
- **Then** Ann's note carries the plain token link
- **And** Zoe's note carries a sign-up link whose destination is the generic redirection endpoint with the document's model and identifier

### PORT-AC-023 - Without free sign-up every recipient gets the plain link (`PORT-RULE-010`)
- **Given** the sign-up scope is `b2b`
- **And** the same document and the same two recipients
- **When** Maya presses `Send`
- **Then** both notes carry the plain token link

### PORT-AC-024 - An access warning hides the send control (`PORT-RULE-012`)
- **Given** a document whose access warning is `This document cannot be shared while it is in preparation.`
- **When** Maya opens the share dialog
- **Then** the warning banner shows that sentence
- **And** the `Send` control is not offered
- **And** a send submitted anyway is refused

### PORT-AC-025 - The invitation is rendered in the recipient's language (`PORT-RULE-015`)
- **Given** two recipients whose languages are French and Dutch
- **When** Maya presses `Send`
- **Then** the note addressed to the first is rendered in French and the note addressed to the second in Dutch

### PORT-AC-026 - Building a share address checks read permission, the record-page builder does not (`PORT-RULE-013`)
- **Given** a sales order 42 with an empty security token
- **And** Ivan, an employee who may not read that order
- **When** `_get_share_url` is called on order 42 as Ivan with the token included
- **Then** the call is refused with the platform's standard read refusal
- **And** the order's token is still empty, because the permission check runs before the token is created
- **When** `_get_share_url` is called on order 42 as Ivan with the token **not** included
- **Then** no read check is run, the returned address carries no token, and the order's token is still empty
- **When** `get_portal_url` is called on order 42 as Ivan
- **Then** no read check is run, a token is created and the returned address carries it, because the callers of that builder have already resolved the record

---

## 3. Tokens and access to a document

### PORT-AC-030 - A token grants a read to an anonymous visitor (`PORT-RULE-020`, `PORT-RULE-025`)
- **Given** a sales order with the token `3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5`
- **When** an anonymous visitor requests `/my/orders/42?access_token=3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5`
- **Then** the order page is served
- **And** the order's token is still `3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d5`
- **When** the same address is requested twice more
- **Then** the page is served both times and the token is still unchanged, because there is no use counter and no expiry

### PORT-AC-031 - A wrong token does not grant a read (`PORT-RULE-020`)
- **Given** the same order
- **When** an anonymous visitor requests the same address with the token `3f2a7c18-5be4-4d0a-9f31-6c0b2e77a1d6`
- **Then** the visitor is redirected to `/my`

### PORT-AC-032 - A missing record redirects rather than reveals (`PORT-RULE-020`)
- **Given** no record with the identifier 999999
- **When** anyone requests `/my/orders/999999?access_token=anything`
- **Then** the requester is redirected to `/my`, with no indication whether the record ever existed

### PORT-AC-033 - A token is not copied (`PORT-RULE-141`)
- **Given** a sales order whose token is set
- **When** the order is duplicated
- **Then** the copy's token is empty
- **And** the original's token is unchanged

### PORT-AC-034 - A token is created once and kept (`PORT-RULE-140`)
- **Given** a sales order whose token is empty
- **When** the token creation operation is called twice
- **Then** both calls return the same value
- **And** exactly one value is stored

### PORT-AC-035 - A token may only be filtered by membership (`PORT-RULE-142`)
- **Given** any model that adopts the access mixin
- **When** a filter uses the membership operator with a list of two values
- **Then** the filter resolves to a comparison of the stored column against those two values
- **When** a filter uses a pattern-matching operator
- **Then** the filter is rejected as an unsupported operator

### PORT-AC-036 - A shared link redirects an unauthorized reader to the portal page (`PORT-RULE-043`)
- **Given** a sales order with a token, and an anonymous visitor
- **When** the visitor follows `/mail/view?model=<the order model>&res_id=42&access_token=<the token>&pid=412&hash=<the signature for 412>`
- **Then** the visitor is redirected to `/my/orders/42` with the token, the recipient identifier and the signature carried over

### PORT-AC-037 - A shared link without a token sends an anonymous visitor to sign in (`PORT-RULE-042`)
- **Given** the same order
- **When** an anonymous visitor follows `/mail/view?model=<the order model>&res_id=42` with no token
- **Then** the visitor is sent to the sign-in page with the original address as the destination

### PORT-AC-038 - A signed-in external user with no access falls back to the portal home (`PORT-RULE-042`)
- **Given** Willis, an external user who may not read order 42
- **When** Willis follows `/mail/view?model=<the order model>&res_id=42` with no token
- **Then** Willis is redirected to `/my`

### PORT-AC-039 - A model that adopts the mixin without supplying a page has a token but no page (`PORT-RULE-143`)
- **Given** a document model that adopts the Portal Access Mixin and does not override the portal web address rule
- **And** one record of that model with the identifier 7
- **When** the record's `access_url` is read
- **Then** it is the single character `#`
- **When** the share dialog is opened on that record
- **Then** a token is created and the dialog's link is the absolute address of the generic redirection endpoint carrying the model, the identifier 7 and that token, which resolves correctly
- **When** `get_portal_url` is called on that record
- **Then** the result begins with `#`, so it addresses no page of its own
- **And** this is a valid configuration: the record is reachable only through the endpoints that its own domain provides, and the token still protects those endpoints

---

## 4. Redirection of external people

### PORT-AC-040 - Signing in lands an external user on the portal (`PORT-RULE-040`)
- **Given** the external user whose login is `portal_user` and whose password is `portal_user`
- **When** that person signs in with no requested destination
- **Then** the resulting page is `/my`

### PORT-AC-041 - An external person cannot reach the back office (`PORT-RULE-041`)
- **Given** a signed-in external user
- **When** that person requests the platform entry point with the query `?a=1`
- **Then** the response is a redirection to `/my?a=1`

### PORT-AC-042 - The access action of a document sends a readable external user to the portal page (`PORT-RULE-044`)
- **Given** a sales order 42 that adopts the access mixin, whose portal web address rule yields `/my/orders/42`
- **And** Willis, an external user who may read that order
- **When** the platform asks order 42 for the action that opens it, for Willis
- **Then** the answer is an "open web address" action in the same window, carrying the record identifier 42
- **And** its address is `/my/orders/42?access_token=<the order's token>`, a token being created if the order had none

### PORT-AC-043 - A forced front-end redirection of an unreadable record yields the bare page (`PORT-RULE-044`)
- **Given** the same order 42 and an external user who may **not** read it
- **When** the platform asks order 42 for the action that opens it, for that user, with the front-end redirection forced
- **Then** the answer is an "open web address" action in the same window, carrying the record identifier 42
- **And** its address is `/my/orders/42` with no token and no query string

### PORT-AC-044 - An unreadable record without a forced redirection falls back to the back office (`PORT-RULE-044`)
- **Given** the same order 42 and an external user who may not read it
- **When** the platform asks order 42 for the action that opens it, for that user, without forcing the front-end redirection
- **Then** the answer is the inherited back-office form action for order 42, not an "open web address" action

### PORT-AC-045 - A target user is only resolved when the acting user may read the record (`PORT-RULE-044`)
- **Given** order 42 and Ivan, an employee who may not read it
- **When** Ivan asks order 42 for the action that opens it, naming Willis as the target user
- **Then** the answer is the inherited back-office form action, and Willis's own permissions are never consulted
- **Given** the same order and Maya, who may read it
- **When** Maya asks order 42 for the action that opens it, naming Willis as the target user, and Willis may read the order
- **Then** the answer is the "open web address" action of `PORT-AC-042`, computed with Willis's identity

### PORT-AC-046 - An employee with no forced redirection keeps the back-office action (`PORT-RULE-044`)
- **Given** order 42 and Maya, an employee who may read it
- **When** the platform asks order 42 for the action that opens it, for Maya, without forcing the front-end redirection
- **Then** the answer is the inherited back-office form action, because Maya is not an external user

---

## 5. The portal home and its counters

### PORT-AC-050 - The home page issues no counting query (`PORT-RULE-054`)
- **Given** a signed-in external user with three quotations and two invoices
- **When** the home page is requested
- **Then** the rendered page contains no counter value
- **And** no counting query has been issued against the order or the invoice model

### PORT-AC-051 - Counters are fetched in batches (counter batching)
- **Given** a home page carrying seven counter names
- **When** the page loads
- **Then** exactly two counter calls are issued, the first carrying the first four names and the second carrying the last three

### PORT-AC-052 - A page with no counter issues no call (counter batching)
- **Given** a home page carrying zero counter names
- **When** the page loads
- **Then** no counter call is issued and the spinner is removed

### PORT-AC-053 - A card is revealed only when its counter is non-zero (`PORT-RULE-052`)
- **Given** a home page whose counters answer `{ quotation_count: 0, order_count: 3 }`
- **When** the answers arrive
- **Then** the quotation card stays hidden and the order card is revealed showing `3`
- **And** the address book card and the security card were visible from the start

### PORT-AC-054 - A card marked as always shown survives a zero counter (`PORT-RULE-052`)
- **Given** the overdue-invoice card, which is declared as always shown
- **And** a counter answer of zero for it
- **Then** the card is revealed nevertheless

### PORT-AC-055 - The counter cache stores presence only (`PORT-RULE-053`)
- **Given** a first load whose answers are `{ invoice_count: 12, bill_count: 0 }`
- **Then** the session cache is `{ invoice_count: true, bill_count: false }`
- **When** a second load answers `{ invoice_count: 9, bill_count: 0 }`
- **Then** the cache is unchanged and is not written back to the session

### PORT-AC-056 - A counter on a forbidden model is zero (`PORT-RULE-051`)
- **Given** an external user with no read permission on the project model
- **When** the project counter is requested
- **Then** the answer is zero and no query is issued against the project model

### PORT-AC-057 - The assigned representative falls back to the company's one (`PORT-RULE-050`)
- **Given** a Contact `Ann` with no salesperson, whose commercial entity `Acme Holding` has the salesperson `Sam`
- **When** Ann opens the portal home
- **Then** the `Your contact` block shows `Sam`

### PORT-AC-058 - No representative means no contact block (`PORT-RULE-050`)
- **Given** a Contact whose salesperson is the anonymous public user, and whose commercial entity has none either
- **When** that person opens the portal home
- **Then** the `Your contact` block is not rendered

---

## 6. The page navigator

### PORT-AC-060 - Every page is listed when there are at most five (page navigator)
- **Given** a list of 150 rows with a page size of 30, on page 3
- **When** the navigator is computed
- **Then** the page count is 5, the current page is 3, the offset is 60 and the page list is `1, 2, 3, 4, 5`

### PORT-AC-061 - The first pages are followed by one ellipsis (page navigator)
- **Given** 300 rows with a page size of 30, on page 1
- **Then** the page count is 10 and the page list is `1, 2, 3, 4, ellipsis, 10`

### PORT-AC-062 - The last pages are preceded by one ellipsis (page navigator)
- **Given** 300 rows with a page size of 30, on page 10
- **Then** the page list is `1, ellipsis, 7, 8, 9, 10` and the offset is 270

### PORT-AC-063 - A middle page is surrounded by two ellipses (page navigator)
- **Given** 300 rows with a page size of 30, on page 5
- **Then** the page list is `1, ellipsis, 4, 5, 6, ellipsis, 10` and the offset is 120

### PORT-AC-064 - A single page produces a one-entry list (page navigator)
- **Given** 20 rows with a page size of 30, on page 1
- **Then** the page count is 1, the page list is `1` and the navigator is not rendered

### PORT-AC-065 - Two pages produce a two-entry list (page navigator)
- **Given** 50 rows with a page size of 30, on page 1
- **Then** the page count is 2 and the page list is `1, 2`

### PORT-AC-066 - A page beyond the end is clamped (page navigator)
- **Given** 300 rows with a page size of 30, on page 27
- **Then** the current page is 10 and the offset is 270

### PORT-AC-067 - A non-numeric page falls back to the first one (page navigator)
- **Given** 300 rows with a page size of 30, on page `abc`
- **Then** the current page is 1 and the offset is 0

### PORT-AC-068 - An empty list produces no navigator (page navigator)
- **Given** 0 rows with a page size of 30
- **Then** the page count is 0, the current page is 1, the offset is 0 and the navigator is not rendered

---

## 7. List pages and the record navigator

### PORT-AC-070 - A list page stores its history (`PORT-RULE-144`)
- **Given** a quotation list whose first page shows 80 quotations
- **When** the page is rendered
- **Then** the session key of that page holds the 80 identifiers in the displayed order
- **And** a list of 250 rows stores only the first 100 identifiers of the current page's read, that is at most the page size

### PORT-AC-071 - The record navigator links to the neighbours with their own tokens (`PORT-RULE-145`)
- **Given** a session history of `[91, 87, 84, 80]` for the quotation list
- **When** record 87 is opened
- **Then** the previous control points at `/my/orders/91?access_token=<token of 91>`
- **And** the next control points at `/my/orders/84?access_token=<token of 84>`

### PORT-AC-072 - The navigator is disabled at the ends (`PORT-RULE-145`)
- **Given** the same history
- **When** record 91 is opened
- **Then** the previous control is disabled and the next control points at record 87
- **When** record 80 is opened
- **Then** the next control is disabled

### PORT-AC-073 - A record outside the history gets no navigator (`PORT-RULE-145`)
- **Given** the same history
- **When** record 42 is opened directly from a link
- **Then** neither a previous nor a next control is rendered

### PORT-AC-074 - Sorting keeps the page number (`PORT-RULE-148`)
- **Given** the invoice list on page 3 sorted by date
- **When** the reader chooses the sort `Due Date`
- **Then** the resulting address is the current path, that is page 3, with the sort key replaced and every other parameter kept

### PORT-AC-075 - Filtering returns to the first page (`PORT-RULE-148`)
- **Given** the invoice list on page 3 filtered by `All`
- **When** the reader chooses the filter `Overdue invoices`
- **Then** the resulting address is `/my/invoices` with the filter key replaced and every other parameter kept, that is page 1

### PORT-AC-076 - A date range is half open (`PORT-RULE-147`)
- **Given** three orders created on the first, the second and the third of March 2026
- **When** the list is requested with a range start of the first of March 2026 and a range end of the third of March 2026
- **Then** the orders of the second and the third are listed and the order of the first is not

### PORT-AC-077 - The default page size is eighty (`PORT-RULE-146`)
- **Given** 205 invoices
- **When** page 3 is requested
- **Then** the page count is 3, the offset is 160 and 45 rows are shown

---

## 8. Reports

### PORT-AC-080 - An unsupported report kind is refused (`PORT-RULE-021`)
- **Given** a sales order reachable with a token
- **When** the page is requested with the report kind `docx`
- **Then** the request is refused with `Invalid report type: docx`

### PORT-AC-081 - A multi-company render is refused (`PORT-RULE-022`)
- **Given** a record set of two timesheet lines belonging to two different companies
- **When** the task timesheet report is requested for that set
- **Then** the request is refused with `Multi company reports are not supported.`

### PORT-AC-082 - A download names the file after the record (`PORT-RULE-024`)
- **Given** an order whose report base name is `S00042 - Acme (draft)`
- **When** the portable document is requested with the download flag
- **Then** the content disposition is an attachment whose file name is `S00042_Acme_draft_` followed by a dot and the extension `pdf`
- **And** the media type is the portable-document one and the content length is set

### PORT-AC-083 - An inline render carries the same name with a different disposition (`PORT-RULE-024`)
- **Given** the same order
- **When** the portable document is requested without the download flag
- **Then** the content disposition is inline with the same file name

### PORT-AC-084 - The report is rendered in the document's company (`PORT-RULE-023`)
- **Given** an order belonging to `Beta`, read by a person whose current company is `Acme`
- **When** the report is rendered
- **Then** the layout, the logo and the paper format of `Beta` are used

---

## 9. Messaging on portal pages

### PORT-AC-090 - A message with the private-note subtype is flagged as a note (portal projection)
- **Given** three messages on a Contact: one with no subtype, one with the public comment subtype and one with the private-note subtype
- **When** the portal projection runs on each
- **Then** the "is a private note" flag is false for the first, false for the second and true for the third

### PORT-AC-091 - A message with no author projects a false author (portal projection)
- **Given** a message with no author and the body `Hello`
- **When** the portal projection runs on it
- **Then** the projected author is false, not an empty structure

### PORT-AC-092 - A portal thread never shows an internal message (`PORT-RULE-061`, `PORT-RULE-172`)
- **Given** a sales order carrying one public comment and one internal note
- **When** the thread is fetched by an anonymous visitor holding the token
- **Then** only the public comment is returned
- **When** the same thread is fetched by an employee on the portal page
- **Then** only the public comment is returned, exactly as for the anonymous visitor: a portal page applies the same conditions to an employee as to an external person

### PORT-AC-093 - An emptied message disappears from the portal thread (`PORT-RULE-061`)
- **Given** a public comment whose body has been emptied and that has no attachment
- **When** the thread is fetched
- **Then** the message is not returned
- **Given** the same message with one attachment
- **Then** the message is returned

### PORT-AC-094 - A bare star rating is returned even with an empty body (`PORT-RULE-061`)
- **Given** the rating bridge is installed
- **And** a message with an empty body, no attachment and a rating value of 4
- **When** the thread is fetched
- **Then** the message is returned

### PORT-AC-095 - An anonymous visitor with a signed link posts under the named Contact (`PORT-RULE-062`)
- **Given** a sales order and Contact 412
- **And** an anonymous visitor who opened the page with `pid=412` and the matching signature
- **When** the visitor posts the body `Please deliver on Friday`
- **Then** a public comment is created whose author is Contact 412

### PORT-AC-096 - An anonymous visitor with only a token posts under the document's customer (`PORT-RULE-062`)
- **Given** a sales order whose customer is Contact 500
- **And** an anonymous visitor who opened the page with the token only
- **When** the visitor posts a message
- **Then** a public comment is created whose author is Contact 500

### PORT-AC-097 - An anonymous visitor with no proof posts without an author (`PORT-RULE-062`)
- **Given** a page opened without a token and without a signed identity, on which posting is nevertheless permitted
- **When** a message is posted
- **Then** the message is created with no author and the projection returns a false author

### PORT-AC-098 - A signed visitor may edit their own message (`PORT-RULE-063`)
- **Given** the message posted in `PORT-AC-095`
- **When** the same visitor, with the same signed link, edits it
- **Then** the edit succeeds
- **When** a different visitor, holding a signed link for Contact 413, tries to edit it
- **Then** the edit is refused

### PORT-AC-099 - A signed visitor is recognized as the author (`PORT-RULE-065`, `PORT-RULE-073`)
- **Given** the message posted in `PORT-AC-095`
- **When** the thread is fetched with the same signed link
- **Then** the projection marks that message as written by the current reader
- **And** its attachments carry an ownership token
- **And** the attachments of other people's messages do not
- **And** every projected attachment, the reader's own and other people's alike, carries a raw-content token and a thumbnail token, so the bytes and the preview can be fetched without a session

### PORT-AC-100 - A signed identity is bound to the token (`PORT-RULE-066`)
- **Given** a signature computed for a document and Contact 412
- **When** the document's token is replaced by a new one
- **Then** the previous signature no longer verifies

### PORT-AC-101 - A signed identity may be inherited from the project (`PORT-RULE-068`)
- **Given** a project with a token, and a task of that project
- **And** a visitor holding the signature computed for the project and Contact 412
- **When** the visitor opens the task page with that signature
- **Then** the signature is accepted for the task

### PORT-AC-102 - A model without a token field cannot sign (`PORT-RULE-067`)
- **Given** a thread model whose declared external-posting token field does not exist on it
- **When** the signing operation is called
- **Then** it is refused with `Model <the model name> does not support token signature, as it does not have <the field name> field.`

### PORT-AC-103 - The customer's notification carries a working link (`PORT-RULE-069`)
- **Given** a sales order whose customer is Contact 412 and whose token is empty
- **When** an employee posts a public comment on it
- **Then** the order now has a token
- **And** the notification sent to Contact 412 carries an access button whose address contains the token, `pid=412` and the matching signature

### PORT-AC-104 - Toggling a message internal removes it from the portal thread (`PORT-RULE-061`)
- **Given** a public comment visible on the portal page
- **When** an employee marks it internal through the toggle endpoint
- **Then** the endpoint returns true
- **And** a fetch of the portal thread no longer returns it

### PORT-AC-105 - An anonymous reaction is attributed to the signed Contact (`PORT-RULE-064`)
- **Given** an anonymous visitor holding a signed link for Contact 412
- **When** the visitor reacts to a message on that document
- **Then** the reaction's author is Contact 412 and no guest author is recorded
- **And** the projection groups it under its emoji with a count of one and the name of Contact 412

### PORT-AC-106 - The author picture address follows the proof (author picture address)
- **Given** message 3175 on a document
- **When** the projection runs with a token
- **Then** the picture address is `/mail/avatar/<message entity name>/3175/author_avatar/50x50?access_token=<the token>`
- **When** it runs with a signature and a recipient identifier instead
- **Then** the address is `/mail/avatar/<message entity name>/3175/author_avatar/50x50?_hash=<signature>&pid=<contact>`
- **When** it runs with neither
- **Then** the address is `/web/image/<message entity name>/3175/author_avatar/50x50`

### PORT-AC-107 - An unresolvable proof serves the placeholder picture
- **Given** the picture endpoint called with a token that does not match the document
- **Then** the shipped placeholder image is served at the requested dimensions, not the author's picture

### PORT-AC-108 - Posting permission comes from the model's declared posting permission (`PORT-RULE-060`)
- **Given** a sales order 42 whose model declares that posting requires the read permission
- **And** an anonymous visitor holding the order's token
- **When** the visitor initializes the thread on order 42
- **Then** the thread is resolved a second time with the read permission, and the answer reports that the visitor may post
- **When** the visitor submits a message on order 42 with the same token
- **Then** the message is created
- **Given** a model that declares the write permission for posting, and a reader who may read but not write a record of it
- **When** that reader submits a message on that record
- **Then** the post is refused
- **Given** a model that declares **no** posting permission at all
- **When** any reader, with a token, with a signed identity or with direct permission, submits a message on a record of it
- **Then** the post is refused, and the thread initialization reports that the reader may neither post nor react
- **And** in every branch the three proofs accepted for the posting permission are the same three accepted for reading: direct permission, a valid security token, or a valid signed recipient identity

### PORT-AC-109 - A video attachment is typed defensively for one browser family (`PORT-RULE-074`)
- **Given** a portal message carrying two attachments, `clip.mp4` whose stored media type is `video/mp4` and `plan.png` whose stored media type is `image/png`
- **When** the thread is fetched by a browser that identifies itself as Safari
- **Then** the projected media type of `clip.mp4` is `application/octet-stream`, so the browser downloads the file instead of attempting a ranged playback
- **And** the projected media type of `plan.png` is unchanged, `image/png`
- **When** the same thread is fetched by any other browser
- **Then** the projected media type of `clip.mp4` is `video/mp4`
- **And** in both cases the stored media type of the attachment itself is never modified

---

## 10. Attachments

### PORT-AC-110 - A pending attachment may be removed (`PORT-RULE-071`)
- **Given** an attachment in the pending state uploaded from a portal composer
- **When** its owner calls the removal endpoint with the page's token
- **Then** the attachment is deleted

### PORT-AC-111 - A non-pending attachment may not be removed (`PORT-RULE-071`)
- **Given** an attachment named `contract.txt` attached to a sales order
- **When** the removal endpoint is called for it
- **Then** it is refused with `The attachment contract.txt cannot be removed because it is not in a pending state.`

### PORT-AC-112 - An attachment referenced by a message may not be removed (`PORT-RULE-072`)
- **Given** a pending attachment that a message already references
- **When** the removal endpoint is called for it
- **Then** it is refused with `The attachment <name> cannot be removed because it is linked to a message.`

### PORT-AC-113 - An unreachable attachment is refused with the generic message (`PORT-RULE-070`)
- **Given** an attachment the caller may neither read nor prove access to
- **When** the removal endpoint is called for it
- **Then** it is refused with `The attachment does not exist or you do not have the rights to access it.`

---

## 11. Ratings

### PORT-AC-120 - Star rendering rounds the fraction to one decimal place (star display)
- **Given** an average of 4.2837 over 17 ratings
- **When** the star row is rendered
- **Then** it shows four full stars, one half star, no empty star and the text `(17)`

### PORT-AC-121 - An average just below five shows a half star (star display)
- **Given** an average of 4.96
- **Then** the row shows four full stars and one half star

### PORT-AC-122 - A small fraction disappears (star display)
- **Given** an average of 2.04
- **Then** the row shows two full stars and three empty stars, with no half star

### PORT-AC-123 - The compressed variant uses three icons (star display)
- **Given** averages of 1.8, 2.05, 3.5 and 3.51
- **Then** the compressed widget shows respectively an empty star, a full star, a half star and a full star, each followed by the numeric average

### PORT-AC-124 - Rating statistics are percentages of the total (rating statistics)
- **Given** the ratings 5, 5, 4, 3 and 1 on a record
- **When** the statistics are computed
- **Then** the total is 5, the average is 3.6 and the percentages are `{1: 20, 2: 0, 3: 20, 4: 20, 5: 40}`

### PORT-AC-125 - An anonymous visitor sees the stars but not the composer (`PORT-RULE-080`)
- **Given** a page carrying the rating widget
- **When** an anonymous visitor opens it
- **Then** the star row and the count are shown and the review button is not

### PORT-AC-126 - A website editor may publish a reply (`PORT-RULE-083`)
- **Given** a rating on a record, and a user who belongs to the website editor group but has no write permission on that record
- **When** that user posts the reply `Thank you for your feedback.`
- **Then** the reply is stored, its publisher is that user's Contact and its publication moment is the current moment

### PORT-AC-127 - A person with write access on the record may publish a reply (`PORT-RULE-083`)
- **Given** a rating on a record, and a user who has write permission on that record but is not a website editor
- **When** that user posts a reply
- **Then** the reply is stored with the same stamping

### PORT-AC-128 - Anyone else is refused (`PORT-RULE-083`)
- **Given** a rating on a record, and a user who is neither a website editor nor allowed to write on the record
- **When** that user posts a reply
- **Then** the operation is refused with `Updating rating comment require write access on related record`

### PORT-AC-129 - A supplied stamp is not trusted blindly (`PORT-RULE-082`)
- **Given** a website editor who posts a reply while supplying a publisher of another Contact and a publication moment in the past
- **Then** the supplied values are kept, because the stamping only fills them when they are absent
- **And** the guard has nevertheless run, so a caller without the right is still refused

### PORT-AC-130 - An unknown rating answers an error structure (`PORT-RULE-081`)
- **Given** no rating with the identifier 999999
- **When** the publish endpoint is called for it
- **Then** the answer is `{ "error": "Invalid rating" }` and nothing is written

### PORT-AC-131 - Rating information is projected only when the caller asks for it (`PORT-RULE-084`)
- **Given** the rating capability package is installed
- **And** a sales order 42 carrying one message with a rating of four stars and a publisher reply, the record itself exposing rating statistics
- **When** the thread is fetched **without** the "include rating" option
- **Then** the projected message carries neither the rating value, nor the rating structure, nor the record's rating statistics
- **And** no Rating row is read at all
- **When** the thread is fetched **with** the "include rating" option
- **Then** the property set gains the rating and the rating value
- **And** the projected message carries the formatted rating structure with `publisher_avatar`, `publisher_comment`, `publisher_datetime`, `publisher_id` and `publisher_name`
- **And** the projected message carries the record's rating statistics, read with elevated rights because an external person holds no permission on ratings
- **Given** a message of the same thread that carries no rating
- **Then** with the option on, its rating structure is the empty structure rather than being absent

---

## 12. Account details and addresses

### PORT-AC-140 - Every mandatory field is checked (`PORT-RULE-106`)
- **Given** the external user `Willis` signed in, and the complete address values `name = Acme Farm 3`, `email = o@d.oo`, `street = Rue de Ramillies 1`, `city = Ramillies`, `zip = 1367`, `country = Belgium`, `phone = +323333333333333`
- **When** the form is submitted six times, each time with exactly one of `name`, `email`, `street`, `city`, `country` and `phone` emptied
- **Then** each response lists the emptied input among the invalid inputs
- **And** each response carries the message `Some required fields are empty.`

### PORT-AC-141 - A malformed electronic mail address is refused (`PORT-RULE-104`)
- **Given** the same person and the same values
- **When** the form is submitted with the address `hello`, then with `hello@.com`, then with `hello@oo.`
- **Then** each response lists `email` among the invalid inputs

### PORT-AC-142 - An employee may not change their own name from the portal form (`PORT-RULE-101`)
- **Given** the employee `Ivan` signed in, whose Contact carries a complete address
- **When** Ivan submits the account form with the name `My name is nobody`
- **Then** the response lists `name` among the invalid inputs
- **And** the message is `If you are ordering for an external person, please place your order via the backend. If you wish to change your name or email address, please do so in the account settings or contact your administrator.`

### PORT-AC-143 - An employee may not change their own address either (`PORT-RULE-101`)
- **Given** the same employee
- **When** Ivan submits the account form with the address `new_email@hohoho.com`
- **Then** the response lists `email` among the invalid inputs with the same message

### PORT-AC-144 - An external person may set their own tax identification number (`PORT-RULE-103`)
- **Given** `Willis`, an external user with no parent Contact and no issued document
- **When** the account form is submitted with the number `BE0926372368` and the complete address
- **Then** the response is `{ "redirectUrl": "/my/addresses" }`
- **And** the Contact now carries that number together with every submitted address value

### PORT-AC-145 - A child Contact may not set the tax identification number (`PORT-RULE-102`)
- **Given** a company Contact `Test Acme` with two children `portal_a` and `portal_b`, each linked to an external user
- **When** the user of `portal_a` submits their own address with the number `BE0926372368`
- **Then** the response lists the tax identification number among the invalid inputs
- **And** the message names the field and says it is managed on the company account

### PORT-AC-146 - Saving the main address writes every value (`PORT-RULE-096`)
- **Given** `Willis` with an empty address
- **When** the account form is submitted with the seven values of `PORT-AC-140`
- **Then** the response is `{ "redirectUrl": "/my/addresses" }`
- **And** the Contact carries exactly those seven values

### PORT-AC-147 - A supplied success address is honoured
- **Given** the same submission whose success address (`callback`) is `/my/custom/return`
- **Then** the response names `/my/custom/return` as the `redirectUrl`

### PORT-AC-148 - Creating a billing address produces a child of the requested kind (`PORT-RULE-093`)
- **Given** `Willis` with no child Contact
- **When** the address form is submitted with no Contact identifier and the address kind `billing`
- **Then** exactly one child Contact exists, its kind is `invoice`, its parent is Willis's commercial entity and it carries every submitted value
- **And** the response is `{ "redirectUrl": "/my/addresses" }`

### PORT-AC-149 - Creating a delivery address produces the delivery kind (`PORT-RULE-093`)
- **Given** the same starting point
- **When** the address form is submitted with the address kind `delivery`
- **Then** exactly one child Contact exists whose kind is `delivery`

### PORT-AC-150 - Using the delivery address as the billing address produces the neutral kind (`PORT-RULE-093`)
- **Given** the same starting point
- **When** the address form is submitted with the address kind `delivery` and the "use delivery as billing" flag set
- **Then** exactly one child Contact exists whose kind is `other`

### PORT-AC-151 - An unchanged name does not touch the bank account holder name (`PORT-RULE-094`)
- **Given** an external user `Partner A` with a bank account whose holder name is `Partner A Holder`
- **When** the account form is submitted with the name unchanged and a complete address
- **Then** the response status is successful
- **And** the bank account's holder name is still `Partner A Holder`

### PORT-AC-152 - An anonymous child address may be renamed (`PORT-RULE-101`)
- **Given** an external user, and a child Contact of that person's commercial entity with the kind `invoice`, no name and no user
- **When** the person submits the address form for that child with the name `Secret Name` and a complete address
- **Then** the child's name is `Secret Name`

### PORT-AC-153 - A colleague's personal Contact may not be edited (`PORT-RULE-091`)
- **Given** the company `Test Acme` with the children `portal_a` and `portal_b`, each of kind `contact` and each linked to a user
- **When** the user of `portal_a` requests the address form for `portal_b`
- **Then** the response is forbidden

### PORT-AC-154 - A child address may be archived (`PORT-RULE-091`, `PORT-RULE-097`)
- **Given** the user of `portal_a`, and a delivery address `Nobody` whose parent is `portal_a`
- **When** that user calls the archive endpoint for `Nobody`
- **Then** the address is archived

### PORT-AC-155 - The main address may not be archived (`PORT-RULE-097`)
- **Given** the same user
- **When** that user calls the archive endpoint for their own Contact
- **Then** the call is refused with `You cannot archive your main address`

### PORT-AC-156 - An address outside the company tree may not be archived (`PORT-RULE-091`)
- **Given** the same user and the unrelated external user `Willis`
- **When** the user of `portal_a` calls the archive endpoint for Willis's Contact
- **Then** the call is refused

### PORT-AC-157 - A colleague's personal Contact may not be archived (`PORT-RULE-091`)
- **Given** the same user
- **When** that user calls the archive endpoint for `portal_b`
- **Then** the call is refused

### PORT-AC-158 - The country is frozen once a document has been issued (`PORT-RULE-100`)
- **Given** an external person whose Contact has the country Belgium and for whom one posted customer invoice exists
- **When** the account form is submitted with the country France
- **Then** the response lists `country_id` among the invalid inputs
- **And** the message is `Changing your country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`

### PORT-AC-159 - The tax identification number is frozen once a document has been issued (`PORT-RULE-103`)
- **Given** an external person who is their own commercial entity, whose Contact carries a number, and for whom one confirmed sales order exists
- **When** the account form is submitted with a different number
- **Then** the response lists the number among the invalid inputs
- **And** the message is `Changing the tax identification number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`
- **And** the form renders the number input as read-only with the same sentence underneath

### PORT-AC-160 - The country layout drives the form (`PORT-RULE-107`)
- **Given** the address form with the country changed to a country whose layout puts the postal code before the city and which requires a subdivision
- **When** the country information is fetched
- **Then** the answer says the postal code precedes the city, lists the subdivisions and lists the subdivision among the mandatory fields
- **And** the form places the postal code before the city, fills and shows the subdivision selector and marks the subdivision as required

### PORT-AC-161 - A country without subdivisions hides the selector (`PORT-RULE-107`)
- **Given** the same form with a country that has no subdivision
- **Then** the subdivision selector is emptied and hidden and is not required

### PORT-AC-162 - The postal code accepts the alternative input name (`PORT-RULE-092`)
- **Given** a submission that carries `zipcode = 12345` and no `zip`
- **Then** the Contact's postal code becomes `12345`

### PORT-AC-163 - A field outside the writable set is not written (`PORT-RULE-092`)
- **Given** a submission that carries `credit_limit = 999999`
- **Then** the Contact's credit limit is unchanged and the value is offered to the extension point as extra form data

### PORT-AC-164 - A half-typed address is completed rather than stored (`PORT-RULE-106`)
- **Given** a page that does not need a postal address, so that only the name and the address are mandatory
- **When** a submission carries a street but no city and no country
- **Then** the response lists `city` and `country_id` among the missing inputs with the message `Some required fields are empty.`

### PORT-AC-165 - The company address is hidden from its children when incomplete (`PORT-RULE-090`)
- **Given** a company Contact with a name, an address, a postal code, a city and a country but no telephone number
- **And** a child external person
- **When** that person opens the address book
- **Then** the company address is not offered in either list

### PORT-AC-166 - The address book merges the two lists when no billing address exists (`PORT-RULE-090`)
- **Given** an external person whose addresses are all of the kind `delivery` or `other`
- **When** the address book is opened
- **Then** the `Same as delivery address` switch is checked and the billing section is hidden

### PORT-AC-167 - A company name typed on a child renames the company (`PORT-RULE-095`)
- **Given** a company Contact `Old Name` with a child address
- **When** the child address is submitted with `company_name = New Name`
- **Then** the company Contact is renamed `New Name`
- **And** the child's own company-name field is empty

### PORT-AC-168 - The tax identification number is checked against the country rules (`PORT-RULE-105`)
- **Given** the accounting capability package is installed
- **And** an external person editing their own Contact, which is its own commercial entity and may still edit the number
- **When** the form is submitted with the country Belgium and the tax identification number `BE0999999999`, which the accounting domain's checker rejects
- **Then** a throwaway Contact carrying only that country and that number is built and submitted to the checker
- **And** the tax identification number input is marked invalid
- **And** the checker's own message is added to the response verbatim, with no rewording and no wrapping sentence
- **And** nothing is written on the Contact
- **When** the form is resubmitted with the same country and the valid number `BE0477472701`
- **Then** the checker raises nothing, the number is not marked invalid, and the Contact is written
- **Given** the same form on an installation where the accounting capability package is **not** installed
- **When** the form is submitted with `BE0999999999`
- **Then** no check is run and the number is written as typed
- **Given** the same form where the number was already marked invalid by `PORT-RULE-102` or `PORT-RULE-103`
- **When** the form is submitted
- **Then** the country check is skipped, so a single input never carries two messages

---

## 13. Password, second factor, passkeys, sessions and application keys

### PORT-AC-170 - An empty password input is refused (`PORT-RULE-110`)
- **Given** a signed-in external person on the security page
- **When** the password form is submitted with an empty new password
- **Then** the response shows `You cannot leave any password empty.` against that input and nothing is changed

### PORT-AC-171 - A mismatched confirmation is refused (`PORT-RULE-110`)
- **When** the form is submitted with `new1 = Sunflower42` and `new2 = Sunflower43`
- **Then** the response shows `The new password and its confirmation must be identical.` against the confirmation input

### PORT-AC-172 - A wrong old password is refused with a specific message (`PORT-RULE-110`)
- **When** the form is submitted with a wrong old password and two identical new ones
- **Then** the response shows `The old password you provided is incorrect, your password was not changed.` against the old-password input

### PORT-AC-173 - A successful change keeps the session alive (`PORT-RULE-111`)
- **Given** a signed-in external person
- **When** the password is changed successfully
- **Then** the banner `Password Updated!` is shown
- **And** the next request in the same session is still authenticated

### PORT-AC-174 - The security page may not be framed by a third party (`PORT-RULE-112`)
- **When** the security page or the account page is served
- **Then** the response carries a frame-options header restricted to the same origin and a content-security policy restricting frame ancestors to the same origin

### PORT-AC-175 - The password minimum length reaches the form (`PORT-RULE-114`)
- **Given** the password policy bridge is installed with a minimum length of 12
- **When** the security page is rendered
- **Then** the new-password input carries a client-side minimum of 12 and a strength meter is rendered next to it

### PORT-AC-176 - An external person may enroll a second factor from the portal (`PORT-RULE-115`, `PORT-RULE-116`)
- **Given** the second-factor bridge for the portal is installed and the person has no second factor
- **When** the person opens `/my/security`
- **Then** the section shows `Two-factor authentication not enabled` and the button `Enable two-factor authentication`
- **When** the person completes the enrollment with a valid code
- **Then** the section shows `Two-factor authentication enabled`
- **And** the person's next sign-in asks for a code

### PORT-AC-177 - Disabling the second factor also drops the trusted devices
- **Given** an external person with a second factor and two trusted devices
- **When** the person disables the second factor
- **Then** the trusted devices are revoked and the section returns to the "not enabled" state

### PORT-AC-178 - An external person may create, rename and delete their own passkey
- **Given** the passkey bridge for the portal is installed
- **When** the person presses `Add Passkey`, confirms their password, names the passkey and completes the browser ceremony
- **Then** the passkey is listed with its name, its creation moment and its last-use moment
- **When** the person renames it and then deletes it
- **Then** both operations succeed

### PORT-AC-179 - An external person may not touch another person's passkey
- **Given** a passkey created by the administrator
- **When** an external person tries to rename it
- **Then** the operation is refused with a permission refusal

### PORT-AC-180 - Revoking every session asks for the password first (`PORT-RULE-113`)
- **Given** a signed-in external person with three devices, one of which is the current one
- **When** the person presses `Log out from all devices`
- **Then** the dialog `Security Control` appears
- **When** the person enters a wrong password
- **Then** the input shows `Check failed` and nothing is revoked
- **When** the person enters the correct password
- **Then** the two other devices are revoked and the current one is not

### PORT-AC-181 - The application key section is hidden without the setting (`PORT-RULE-120`)
- **Given** the setting `Customer API Keys` is off
- **When** an external person opens `/my/security` in developer mode
- **Then** the application key section is not rendered

### PORT-AC-182 - The application key section is hidden outside developer mode (`PORT-RULE-120`)
- **Given** the setting is on
- **When** an external person opens `/my/security` outside developer mode
- **Then** the application key section is not rendered

### PORT-AC-183 - An external person may create a key when the setting allows it (`PORT-RULE-121`, `PORT-RULE-123`)
- **Given** the setting is on and developer mode is active
- **When** an external person creates a key with the description `Data extraction` and a duration
- **Then** the password dialog is shown first
- **And** the key is shown once in the dialog `Application Key Ready`
- **And** reopening the page lists the key with its description, its scope, its creation moment and its expiration date, without the key value
- **When** the person opens the creation dialog again, and then deletes the listed key
- **Then** each of the three operations (opening the dialog, generating the key, deleting the key) answers with an identity-check request first, shows the password dialog, and only replays the operation once the password is accepted

### PORT-AC-184 - The custom expiration is not offered to an external person (`PORT-RULE-122`)
- **Given** the same conditions
- **When** the creation dialog is built
- **Then** the duration selector contains every shipped duration except the custom-date one

### PORT-AC-185 - A person who is neither an employee nor an external user is refused (`PORT-RULE-121`)
- **Given** the setting is on
- **When** an anonymous public user attempts to create a key
- **Then** the attempt is refused with `Only internal and portal users can create API keys`

### PORT-AC-186 - The second-factor invitation points a non-employee at the portal (`PORT-RULE-117`)
- **Given** the external user `Willis`
- **When** an administrator sends Willis the "please enable a second factor" invitation
- **Then** the address carried by the invitation is `/my/security`
- **Given** the employee `Ivan`
- **When** an administrator sends Ivan the same invitation
- **Then** the address carried by the invitation is the back-office one, unchanged from the inherited behavior

---

## 14. Account deletion

### PORT-AC-190 - A wrong confirmation is refused (`PORT-RULE-130`)
- **Given** an external person whose login is `portal_user`
- **When** the deletion form is submitted with `validation = portal-user` and the correct password
- **Then** nothing is written, the security page re-renders with the dialog open and the message `You should enter "portal_user" to validate your action.` is shown under the confirmation input

### PORT-AC-191 - A wrong password is refused (`PORT-RULE-131`)
- **When** the deletion form is submitted with the correct login and a wrong password
- **Then** nothing is written and `Wrong password.` is shown under the password input

### PORT-AC-192 - A confirmed deletion makes the account unusable and queues the removal (`PORT-RULE-133`)
- **Given** an external person whose login is `portal_user` and who owns one application key
- **When** the deletion form is submitted with the correct login and password
- **Then** the user is archived
- **And** the login starts with `__deleted_user_`
- **And** the stored password verifies against the empty string, which no credential check can satisfy
- **And** the user owns no application key
- **And** exactly one User Deletion Request exists for that user, in the state `todo`
- **And** the browser is redirected to the sign-in page with the message `Account deleted!`
- **And** the session is closed

### PORT-AC-193 - The queued removal deletes the account (`PORT-RULE-135`)
- **Given** the situation at the end of `PORT-AC-192`
- **When** the scheduled job is run
- **Then** the user no longer exists

### PORT-AC-194 - An employee may not delete their account this way (`PORT-RULE-132`)
- **Given** an employee
- **When** the deactivation operation is called for that employee
- **Then** it is refused with `Only the portal users can delete their accounts. The user(s) <the employee's name> can not be deleted.`

### PORT-AC-195 - The block-list choice is honoured (`PORT-RULE-134`)
- **Given** an external person whose address is `willis@example.com` and whose telephone number formats successfully
- **When** the deletion is confirmed with the block-list checkbox ticked
- **Then** `willis@example.com` is present in the electronic mail block list with the reason naming the deleted account and the acting user
- **And** the telephone number is present in the telephone block list with a note carrying the same reason

### PORT-AC-196 - The block-list choice may be declined (`PORT-RULE-134`)
- **When** the deletion is confirmed with the checkbox unticked
- **Then** neither block list is changed

### PORT-AC-197 - A note explains the archiving (`PORT-RULE-133`)
- **When** a deletion is confirmed
- **Then** the person's Contact carries a note whose body is `Archived because <the acting user's name> (#<the acting user's identifier>) deleted the portal account`

### PORT-AC-198 - A Contact that a document references survives archived (`PORT-RULE-137`)
- **Given** a queued deletion whose Contact is referenced by a posted invoice
- **When** the scheduled job runs
- **Then** the user is deleted, the request is `done` and the Contact still exists, archived
- **And** a warning line has been written

### PORT-AC-199 - A failed user deletion is marked and not retried (`PORT-RULE-136`)
- **Given** a queued deletion whose user cannot be deleted
- **When** the scheduled job runs
- **Then** the request is `fail`
- **When** the job runs again
- **Then** that request is not picked up

### PORT-AC-200 - A request whose user vanished is closed immediately (`PORT-RULE-135`)
- **Given** a request in `todo` whose user has already been removed by other means
- **When** the job runs
- **Then** the request becomes `done` without any deletion attempt

### PORT-AC-201 - The queue is processed fifty at a time (`PORT-RULE-135`)
- **Given** one hundred and twenty requests in `todo` whose users all exist
- **When** the job runs once
- **Then** at most fifty are processed

---

## 15. Permissions of the portal

### PORT-AC-210 - An external person sees only their commercial tree's orders (`PORT-RULE-171`)
- **Given** a company `Acme Holding` with the children `Ann` and `Bob`, an order addressed to `Ann` and an order addressed to an unrelated Contact
- **When** Bob opens `/my/orders`
- **Then** only the order addressed to `Ann` is listed

### PORT-AC-211 - An external person may only read (`PORT-RULE-171`)
- **Given** Bob and an order he may read
- **When** a write is attempted on that order as Bob
- **Then** it is refused, because the model permissions grant only reading even though the record rule covers writing

### PORT-AC-212 - An external person sees only posted invoices of their tree (`PORT-RULE-171`)
- **Given** a draft invoice and a posted invoice, both addressed to `Ann`
- **When** Bob opens `/my/invoices`
- **Then** only the posted one is listed

### PORT-AC-213 - A vendor sees the purchase orders addressed to them (`PORT-RULE-171`)
- **Given** a purchase order whose vendor is `Ann`
- **When** Bob, a child of the same commercial entity, opens `/my/purchase`
- **Then** the order is listed

### PORT-AC-214 - The three wizards are closed to everyone but a Contact manager (`PORT-RULE-170`)
- **Given** the employee `Ivan`
- **When** Ivan tries to read the share wizard, the invitation wizard or an invitation line
- **Then** each attempt is refused
- **And** no group at all may delete a row of any of the three

---

## 16. Presentation

### PORT-AC-220 - The portal home loads for an external person
- **Given** the external person `Willis` whose Contact carries a city, a company name, a country, a telephone number, a street, a postal code and a country subdivision
- **When** Willis signs in and reaches the portal home
- **Then** the page renders with the sidebar showing the name, the company name and the address, and with the `Addresses` and `Connection & Security` cards

### PORT-AC-221 - A frozen tax identification number is explained on the form
- **Given** `Willis` for whom documents have been issued
- **When** Willis opens `/my/account`
- **Then** the tax identification number input is read-only
- **And** the sentence `Changing the tax identification number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.` is shown underneath
- **And** editing the telephone number and saving still succeeds

### PORT-AC-222 - The skip-to-content control is offered to non-employees
- **Given** an external person
- **When** any portal page is rendered
- **Then** the first focusable element of the page is a control labelled `Skip to Content` that moves the focus to the page content
- **And** that control is not rendered for an employee

### PORT-AC-223 - The language selector appears only with a choice (`PORT-RULE-149`)
- **Given** one published front-end language
- **Then** no language selector is rendered
- **Given** two published front-end languages
- **Then** the selector is rendered with two entries, the active one marked, and each entry rewriting the current address for that language

### PORT-AC-224 - The user menu shortens a long name (user menu name)
- **Given** a person named `Marie-Christine Delacroix-Fontaine`, which is thirty-four characters long
- **Then** the menu shows `Marie-Christine Delacr...`
- **Given** a person named `Marie-Christine Delacroix`, which is twenty-five characters long
- **Then** the menu shows the name in full

### PORT-AC-225 - The due-date label is computed from whole days (due-date label)
- **Given** the current date is the eleventh of September 2026
- **Then** a due date of the eleventh renders `Due today`, a due date of the fifteenth renders `Due in 4 days`, and a due date of the ninth renders `2 days overdue`

### PORT-AC-226 - The unfollow page uses the portal layout
- **Given** a notification carrying an unfollow link for a record
- **When** the recipient follows it
- **Then** the Contact is removed from the record's followers
- **And** the confirmation page is rendered inside the portal layout with the record's display name and the human name of its model
- **When** the link's signature is altered
- **Then** the request is refused with `Non existing record or wrong token.`

---

## 17. Signing and paying

### PORT-AC-230 - Signing a quotation records the signature and confirms it
- **Given** a quotation that requires a signature and no payment, opened with a valid token
- **When** the signature endpoint receives the name `Ann Smith` and a signature image
- **Then** the order carries `signed_by = Ann Smith`, a signing moment and the image
- **And** the order is confirmed
- **And** a public comment `Order signed by Ann Smith` exists on the order with the signed report attached
- **And** the response asks the browser to reload at the order page with the flash value `sign_ok`
- **And** the reloaded page shows `Your order has been confirmed.`

### PORT-AC-231 - Signing a quotation that also needs a payment does not confirm it
- **Given** a quotation that requires both a signature and a payment
- **When** the signature is recorded
- **Then** the order is not confirmed
- **And** the response asks the browser to reload with the flash value `sign_ok` and the payment invitation
- **And** the page shows `Your order has been signed but still needs to be paid to be confirmed.`

### PORT-AC-232 - A missing signature is refused
- **When** the signature endpoint is called with a name and no image
- **Then** the answer is `{ "error": "Signature is missing." }` and nothing is written

### PORT-AC-233 - A malformed signature is refused
- **When** the signature endpoint is called with an image that is not decodable
- **Then** the answer is `{ "error": "Invalid signature data." }` and nothing is written

### PORT-AC-234 - A document that does not need a signature refuses one
- **Given** a confirmed order
- **When** the signature endpoint is called
- **Then** the answer is `{ "error": "The order is not in a state requiring customer signature." }`

### PORT-AC-235 - Rejecting requires a reason
- **Given** a quotation that requires a signature, opened with a token
- **When** the rejection endpoint is called with no reason
- **Then** the browser is redirected to the order page with the flash value `cant_reject` and the order is unchanged
- **When** it is called with the reason `Too expensive`
- **Then** the order is cancelled and a public comment carrying that text exists on it

### PORT-AC-236 - A payment below the required prepayment is refused
- **Given** an order of 1,200.00 with a required prepayment share of 30 percent, not yet confirmed
- **When** the page is opened with a suggested amount of 200.00
- **Then** the request is refused with `The amount is lower than the prepayment amount.`

### PORT-AC-237 - A down payment is proposed when the prepayment share is partial (payment amount)
- **Given** the same order, opened with no suggested amount and no explicit choice
- **Then** the payment amount offered is 360.00
- **And** the sidebar shows the heading `Down payment`, the amount 360.00 and the share `30%`

### PORT-AC-238 - A company mismatch hides the payment form
- **Given** an order belonging to `Beta` and a signed-in person whose Contact belongs to `Acme`
- **When** the order page is opened
- **Then** the payment form is replaced by the "switch company" warning

### PORT-AC-239 - An invalid token on the transaction endpoint is refused
- **When** the transaction endpoint of an order is called with a token that does not match
- **Then** it is refused with `The access token is invalid.`

### PORT-AC-240 - A quotation view is logged once per day per session
- **Given** a quotation in the sent state and an anonymous visitor holding a token
- **When** the visitor opens the page twice on the same day in the same session
- **Then** exactly one note `Quotation viewed by customer <name>` exists on the order
- **And** that note is written in the salesperson's language
- **When** the request carries the link-preview marker
- **Then** no note is written
