# Glossary of the Customer Portal

Terms used in this folder, with their meaning in this domain. Terms owned by another domain carry a pointer to it.

**Access warning.** A sentence that a document model may publish on one of its records to say that the record must not be shared. When it is not empty, the share dialog shows it and withholds the send control.

**Account details form.** The portal page at which a signed-in person edits their own Contact: name, electronic mail address, telephone number, company name, tax identification number and postal address.

**Activation link.** The address carried by the button `Activate Account` of an invitation message. It resolves a Contact through a sign-up token and lets the person choose a password. It stops resolving as soon as the Contact's outstanding-token marker, its list of users or its latest authentication moment changes, and in any case after the sign-up validity window has passed.

**Address book.** The portal page that lists the billing and the delivery addresses a person may use, and offers creation, editing and archiving of the ones they may edit.

**Address kind.** The classification of a Contact as a person (`contact`), a billing address (`invoice`), a delivery address (`delivery`) or a neutral address (`other`). It decides which list an address appears in and which badge its card shows. Owned by [Contacts and Organizations](../contacts-and-organizations/README.md).

**Addressed line.** A line of an invitation session whose typed address normalises to a non-empty address. Only addressed lines take part in the search for users that already hold one of those addresses as a login; every other line is `ko` without a query being issued.

**Always-shown card.** A home card that is revealed even when its counter is zero, because the capability package that contributed it declared it so. The two alert cards shipped with the system are of this kind.

**Anonymous public user.** The identity the platform uses for a request that carries no session. Its Contact is archived, which is why a newly created address is never attached to it. Owned by [Identity and Access](../identity-and-access/README.md).

**Application key.** A long-lived machine credential of a user, presented instead of a password by a program. External people may create one only when the operating company enables the setting and the page is opened in developer mode. Owned by [Identity and Access](../identity-and-access/README.md).

**Assigned representative.** The salesperson shown in the `Your contact` block of the account sidebar: the salesperson of the reader's Contact, or failing that the salesperson of its commercial entity.

**Base web address.** The absolute address of the installation, read from a system parameter, prefixed to a portal path to produce a link that can be copied into a message. Every share link is absolute for this reason; every link inside a rendered page is relative.

**Candidate user.** A user returned by the search that decides the address validity of the invitation lines. The search condition, the fields read of each candidate and the test that turns a candidate into an `exist` verdict are three extension points, so a capability package can narrow the detection without replacing it.

**Card (home card).** One tile of the portal home. It carries a title, a descriptive line, an optional pictogram, an optional counter and a destination. It stays hidden until its counter is known to be non-zero, unless it is declared as always shown or has no counter at all.

**Category row.** One of the four optional rows of the portal home — alerts, client documents, service documents and vendor documents — plus the common row that always carries the address book and the security cards. A category row renders only when at least one capability package contributed a card to it.

**Commercial entity.** The Contact at the top of a Contact's own branch, that is the company a person belongs to, or the person themselves when they belong to no company. The portal scopes almost everything to it: which documents a person sees, which addresses they may use and which fields they may edit. Owned by [Contacts and Organizations](../contacts-and-organizations/README.md).

**Commercial field.** A field whose value belongs to the commercial entity and is propagated down to its children, so that a child may never set it on its own record. The tax identification number is the synchronized one; the company registry number, the industry, the payable and receivable accounts, the fiscal position, the two payment terms and the credit limit are the others in the shipped configuration. Owned by [Contacts and Organizations](../contacts-and-organizations/README.md).

**Constant-time comparison.** A comparison of two secrets that takes the same time whatever the number of matching leading characters, so that a caller cannot discover a secret by measuring response times. Every secret comparison in this domain is of this kind.

**Counter.** A named number shown on a home card, computed by the domain that owns the underlying documents and fetched asynchronously by the browser after the page has rendered.

**Counter cache.** A per-session map from counter name to a boolean saying whether that counter was non-zero on the previous load. It lets a returning reader see their cards immediately.

**Deleted-account login.** The reserved login written over a person's own login when they confirm the deletion of their account. It is built from a fixed prefix, the user identifier and the current moment expressed in seconds with a fraction, which makes it unique and makes the previous login unusable.

**Developer mode.** A request state in which technical controls are rendered. Two portal sections depend on it: the application key section and certain diagnostic controls. Owned by the platform foundation; see [the request lifecycle](../../runtime/request-lifecycle.md).

**Discussion thread (on a portal page).** The list of publicly visible messages of a record, plus the composer. It shows only messages that are not internal, whose subtype is set and not internal, and whose body is non-empty or which carry at least one attachment.

**Elevated rights.** An execution mode in which the permission layers are bypassed while the acting user is kept for authorship. Every use of it in this domain is justified by a preceding validation and is bounded to what that validation covered.

**Employee.** A user that is a member of the internal user group. The portal is not hidden from employees, but an employee sees a portal page exactly as a customer does, and may not change their own name or electronic mail address through the portal address form.

**Emptied-message marker.** The body a message is left with when its author edits all the text away. A message carrying it disappears from every portal page unless it still has an attachment or, with the rating bridge installed, a rating value.

**Extension point.** An operation whose base behaviour another capability package may override, calling the inherited behaviour first. This domain declares four of them: the three that decide whether a typed address is already used as a login, and the one that receives the form values an address submission could not write onto a Contact.

**External user.** A user whose `share` characteristic is true, that is, a member of the portal group or of the public group. Redirected to the portal home on sign-in and refused entry to the back office.

**Flash message.** A value carried in the query string of a record page — an error, a warning, a success or a domain-specific message — that the page renders as a banner and then forgets. It survives exactly one rendering, because it lives in the address rather than in a record.

**Front-end layout.** The page frame every portal page is rendered inside: the document text direction taken from the active language, the navigation bar with the company logo and the user menu, the skip-to-content link, the full-width alert slot, the breadcrumb, the footer with the language selector and the credit line, and the social preview metadata.

**Front-end writable field set.** The closed list of Contact fields an external person may write from a front-end form. Anything else submitted by a form is set aside as extra form data.

**Guided tour.** A scripted walk-through the front end can run for a signed-in person. Whether tours are enabled, and which one to run, are added to the front-end session payload by this domain, which is what lets a tour run for a person who can never reach the back office.

**Home (portal home).** The page at `/my`, showing the card grid and the account sidebar.

**Identity check (re-authentication gate).** The dialog that asks for the account password again before a sensitive operation. In the portal it protects the revocation of every session and the creation and deletion of application keys. Owned by [Identity and Access](../identity-and-access/README.md).

**Internal message.** A message flagged as restricted to employees. Never shown on a portal page, whoever is reading.

**Invitation line.** One row of an invitation session: a Contact, its editable electronic mail address, the linked user, the latest authentication moment, the derived access state and the three action controls.

**Invitation session.** A short-lived record holding the Contacts in scope of an invitation, one line per Contact, and the free-text welcome message added to the invitation.

**Link signature.** A value derived from a request path and its parameters with the platform's database secret, carried in a notification link to prove that the link was produced by the platform. Used by the unfollow endpoint. Owned by [Messaging and Activities](../messaging-and-activities/README.md).

**Link-preview crawler.** A program that fetches a shared address in order to render a preview of it in a messaging client. A request identified as one must not be counted as a customer opening the document; the quotation page relies on this before it logs its "viewed by the customer" note.

**Logical parent (of a signed identity).** A record whose signed recipient identity is accepted in place of the record's own. The shipped case is a Task, whose logical parent is its Project.

**Main address.** A person's own Contact, as opposed to the billing and delivery addresses attached to their company. It cannot be archived from the portal and is edited through the account details form rather than the address form.

**Normalisation (of an address).** Trimming an electronic mail address, keeping only the part between the angle brackets when it has the display form `Name <local@domain>`, lowercasing the domain part, and yielding nothing when the result is not a single syntactically valid address. Every comparison of addresses in this domain is a comparison of normalised forms.

**Offcanvas sidebar.** The account sidebar as it is shown on a narrow screen: hidden behind the avatar control of the heading and slid over the page when that control is pressed. On a wide screen the same content is a column beside the page body.

**Page navigator.** The row of page controls under a list, computed from the total number of rows, the page size and the requested page number.

**Pending attachment.** A file uploaded from a portal composer that is not yet attached to any message: it is attached to the message-composition model with a record identifier of zero. Only such an attachment may be removed through the portal removal endpoint.

**Personal link.** A share link built for one named recipient: it carries the document's security token, the recipient's Contact identifier and the signed recipient identity for that Contact. Two recipients of the same document therefore hold two different links and post under their own names.

**Portal Access Mixin.** The contract a document model adopts to become reachable from the portal: a portal web address, a security token, an access warning and the address-building and access-action operations.

**Portal contact.** The Contact that a proof (a signed recipient identity or a security token) resolves for an anonymous visitor. It becomes the author of the messages and reactions that visitor creates, and is what makes the visitor the "owner" of their own messages.

**Portal group.** The access group that marks a user as an external person of the portal. A revoked person keeps it; the public group is never used for a revoked person.

**Portal message projection.** The reduced, front-end-oriented representation of a message produced for a portal page: author, body, date, attachments with their tokens, reactions, starred flag, private-note flag, formatted publication date, thread descriptor and, on request, rating information.

**Portal user template.** The user record duplicated to create every new external account, so that the new account starts with the portal group, the right notification preferences and the right home action. Its identifier is held in a system parameter. Owned by [Identity and Access](../identity-and-access/README.md).

**Portal web address.** The path of the portal page of a record, relative to the platform root. Owned by the document's own domain; the mixin's own value is a single hash character.

**Publishable message.** A message of a record that may appear on a public page: its kind is one of the five conversational kinds, it is not internal, its subtype is set and not internal, and it has a non-empty body, an attachment, or a rating value.

**Publisher reply.** The operating company's public answer under a customer rating, stamped automatically with the publisher Contact and the publication moment.

**Read-only endpoint.** An endpoint declared as performing no write, which lets the platform serve it from a read-only connection. The home counters, the address book, the address form and the country layout are of this kind; the address submission, the archive call and the password change are not.

**Record navigator.** The previous and next controls of a record page, computed from the identifiers the list page stored in the session.

**Record rule.** A filter expression that rows of a model must satisfy for a given operation, attached to a group. Every portal list page depends on the record rules the document domains declare for the portal group. Owned by [Identity and Access](../identity-and-access/README.md).

**Report kind.** One of the three forms in which a document's report may be served: rich text, a portable document, or plain text. Anything else is refused. Only a portable document is served with a file name and a content disposition.

**Scroll-spy sidebar.** A record-page sidebar whose entries highlight themselves as the reader scrolls past the corresponding section of the document body. A document domain opts into it by using the scroll-spy page frame instead of the plain one.

**Search bar.** The row above a list page carrying the title or the breadcrumb, the sort selector, the filter selector, the group-by selector and the scoped search input.

**Security token (of a record).** A per-record secret that grants a read of that record to whoever presents it. Generated on first demand as a version-4 random universally unique identifier, never copied when the record is duplicated, and never invalidated by being used.

**Session history key.** The name under which a list page stores the ordered identifiers of the rows it displayed, truncated to one hundred. The record page of the same document kind reads that list to compute its previous and next links.

**Share dialog.** The dialog opened from a document, offering a copy-ready link and a list of recipients who each receive a personal invitation.

**Share link.** The absolute, copy-ready address shown in the share dialog. It points at the generic redirection endpoint with the entity transport name, the record identifier and the document's security token. It is the same for every reader, unlike a personal link.

**Share split.** The decision, taken when an invitation is sent, of which recipients receive a plain token link and which receive an individual sign-up link.

**Sign-up link.** The address sent to a share recipient who has no account, when free sign-up is enabled: it carries a sign-up token for that Contact and, as its destination, the generic redirection endpoint pointing at the shared document. Following it leads to registration first and to the document afterwards.

**Sign-up scope.** The setting that decides whether anyone may create an account (`b2c`) or only a person holding a token (`b2b`). It governs the share split. Owned by [Identity and Access](../identity-and-access/README.md).

**Sign-up token.** A time-limited value produced for a Contact that proves the right to set a password. Its payload includes the Contact's latest authentication moment, its list of users and its sign-up type, so it stops working as soon as any of those change. Owned by [Identity and Access](../identity-and-access/README.md).

**Sign-up type.** The marker on a Contact that says which kind of token is outstanding for it: an invitation token or a password-reset token. Granting portal access sets it; revoking clears it.

**Signature panel.** The reusable block that collects a name and a drawn, typed or uploaded signature and posts them to a document-specific endpoint.

**Signed recipient identity.** A value derived from the database name, the record's token and a Contact identifier with the platform's database secret. Carried in a personal share link, it proves to the portal that the visitor is that Contact, and lets them post, react and edit under that name without an account.

**Skip-to-content link.** A control placed first in the front-end layout, visible only when it has the keyboard focus, that jumps past the navigation to the page body. It is rendered for everyone except an employee.

**Social preview metadata.** The title, description, site name, image, image dimensions and summary-card marker the front-end layout emits for a page that declares a preview object, so that a messaging client can render a card instead of a bare address.

**Star widget.** The row of full, half and empty stars that renders a record's average rating, followed by the number of ratings. A compressed variant renders one single icon and the numeric average instead.

**System robot user.** The non-human identity under which unattended work runs. Two operations of this domain use it: archiving a person's own account during a deletion request, which a person may not do to themselves, and the daily run that empties the deletion queue. Owned by [Identity and Access](../identity-and-access/README.md).

**Transient record.** A row created only to back a dialog, never referenced by a persistent record, and removed by the platform once it is older than the retention period for such rows. The three wizards of this domain are transient, so nothing in the domain may depend on one surviving a request.

**Trusted device (second factor).** A device on which the second factor has been remembered, so that a later sign-in from it does not ask for a code. Listed on the portal security page and revocable there. Owned by [Identity and Access](../identity-and-access/README.md).

**User Deletion Request.** A queued request to remove an external account, created when a person confirms the deletion dialog and processed by a daily scheduled job. Owned by [Identity and Access](../identity-and-access/README.md).

**Website editor.** A person allowed to change the published pages of the website. In this domain, such a person may publish a reply under a rating without having write permission on the rated record, and may switch the optional layout variants on or off.
