# Glossary of the Customer Portal

Terms used in this folder, with their meaning in this domain. Terms owned by another domain carry a pointer to it.

**Access warning.** A sentence that a document model may publish on one of its records to say that the record must not be shared. When it is not empty, the share dialog shows it and withholds the send control.

**Account details form.** The portal page at which a signed-in person edits their own Contact: name, electronic mail address, telephone number, company name, tax identification number and postal address.

**Address book.** The portal page that lists the billing and the delivery addresses a person may use, and offers creation, editing and archiving of the ones they may edit.

**Address kind.** The classification of a Contact as a person (`contact`), a billing address (`invoice`), a delivery address (`delivery`) or a neutral address (`other`). It decides which list an address appears in and which badge its card shows. Owned by [Contacts and Organizations](../contacts-and-organizations/README.md).

**Anonymous public user.** The identity the platform uses for a request that carries no session. Its Contact is archived, which is why a newly created address is never attached to it. Owned by [Identity and Access](../identity-and-access/README.md).

**Application key.** A long-lived machine credential of a user, presented instead of a password by a program. External people may create one only when the operating company enables the setting and the page is opened in developer mode. Owned by [Identity and Access](../identity-and-access/README.md).

**Assigned representative.** The salesperson shown in the `Your contact` block of the account sidebar: the salesperson of the reader's Contact, or failing that the salesperson of its commercial entity.

**Card (home card).** One tile of the portal home. It carries a title, a descriptive line, an optional pictogram, an optional counter and a destination. It stays hidden until its counter is known to be non-zero, unless it is declared as always shown or has no counter at all.

**Commercial entity.** The Contact at the top of a Contact's own branch, that is the company a person belongs to, or the person themselves when they belong to no company. The portal scopes almost everything to it: which documents a person sees, which addresses they may use and which fields they may edit. Owned by [Contacts and Organizations](../contacts-and-organizations/README.md).

**Commercial field.** A field whose value belongs to the commercial entity and is propagated down to its children, so that a child may never set it on its own record. The tax identification number is the synchronized one; the company registry number, the industry, the payable and receivable accounts, the fiscal position, the two payment terms and the credit limit are the others in the shipped configuration. Owned by [Contacts and Organizations](../contacts-and-organizations/README.md).

**Constant-time comparison.** A comparison of two secrets that takes the same time whatever the number of matching leading characters, so that a caller cannot discover a secret by measuring response times. Every secret comparison in this domain is of this kind.

**Counter.** A named number shown on a home card, computed by the domain that owns the underlying documents and fetched asynchronously by the browser after the page has rendered.

**Counter cache.** A per-session map from counter name to a boolean saying whether that counter was non-zero on the previous load. It lets a returning reader see their cards immediately.

**Developer mode.** A request state in which technical controls are rendered. Two portal sections depend on it: the application key section and certain diagnostic controls. Owned by the platform foundation; see [the request lifecycle](../../runtime/request-lifecycle.md).

**Discussion thread (on a portal page).** The list of publicly visible messages of a record, plus the composer. It shows only messages that are not internal, whose subtype is set and not internal, and whose body is non-empty or which carry at least one attachment.

**Elevated rights.** An execution mode in which the permission layers are bypassed while the acting user is kept for authorship. Every use of it in this domain is justified by a preceding validation and is bounded to what that validation covered.

**Employee.** A user that is a member of the internal user group. The portal is not hidden from employees, but an employee sees a portal page exactly as a customer does, and may not change their own name or electronic mail address through the portal address form.

**External user.** A user whose `share` characteristic is true, that is, a member of the portal group or of the public group. Redirected to the portal home on sign-in and refused entry to the back office.

**Front-end writable field set.** The closed list of Contact fields an external person may write from a front-end form. Anything else submitted by a form is set aside as extra form data.

**Home (portal home).** The page at `/my`, showing the card grid and the account sidebar.

**Identity check (re-authentication gate).** The dialog that asks for the account password again before a sensitive operation. In the portal it protects the revocation of every session and the creation and deletion of application keys. Owned by [Identity and Access](../identity-and-access/README.md).

**Internal message.** A message flagged as restricted to employees. Never shown on a portal page, whoever is reading.

**Invitation session.** A short-lived record holding the Contacts in scope of an invitation, one line per Contact, and the free-text welcome message added to the invitation.

**Invitation line.** One row of an invitation session: a Contact, its editable electronic mail address, the linked user, the latest authentication moment, the derived access state and the three action controls.

**Link signature.** A value derived from a request path and its parameters with the platform's database secret, carried in a notification link to prove that the link was produced by the platform. Used by the unfollow endpoint. Owned by [Messaging and Activities](../messaging-and-activities/README.md).

**Logical parent (of a signed identity).** A record whose signed recipient identity is accepted in place of the record's own. The shipped case is a Task, whose logical parent is its Project.

**Main address.** A person's own Contact, as opposed to the billing and delivery addresses attached to their company. It cannot be archived from the portal and is edited through the account details form rather than the address form.

**Page navigator.** The row of page controls under a list, computed from the total number of rows, the page size and the requested page number.

**Pending attachment.** A file uploaded from a portal composer that is not yet attached to any message: it is attached to the message-composition model with a record identifier of zero. Only such an attachment may be removed through the portal removal endpoint.

**Portal Access Mixin.** The contract a document model adopts to become reachable from the portal: a portal web address, a security token, an access warning and the address-building and access-action operations.

**Portal contact.** The Contact that a proof (a signed recipient identity or a security token) resolves for an anonymous visitor. It becomes the author of the messages and reactions that visitor creates, and is what makes the visitor the "owner" of their own messages.

**Portal group.** The access group that marks a user as an external person of the portal. A revoked person keeps it; the public group is never used for a revoked person.

**Portal message projection.** The reduced, front-end-oriented representation of a message produced for a portal page: author, body, date, attachments with their tokens, reactions, starred flag, private-note flag, formatted publication date, thread descriptor and, on request, rating information.

**Portal web address.** The path of the portal page of a record, relative to the platform root. Owned by the document's own domain; the mixin's own value is a single hash character.

**Publisher reply.** The operating company's public answer under a customer rating, stamped automatically with the publisher Contact and the publication moment.

**Record navigator.** The previous and next controls of a record page, computed from the identifiers the list page stored in the session.

**Record rule.** A filter expression that rows of a model must satisfy for a given operation, attached to a group. Every portal list page depends on the record rules the document domains declare for the portal group. Owned by [Identity and Access](../identity-and-access/README.md).

**Search bar.** The row above a list page carrying the title or the breadcrumb, the sort selector, the filter selector, the group-by selector and the scoped search input.

**Security token (of a record).** A per-record secret that grants a read of that record to whoever presents it. Generated on first demand as a version-4 random universally unique identifier, never copied when the record is duplicated, and never invalidated by being used.

**Share dialog.** The dialog opened from a document, offering a copy-ready link and a list of recipients who each receive a personal invitation.

**Share split.** The decision, taken when an invitation is sent, of which recipients receive a plain token link and which receive an individual sign-up link.

**Signature panel.** The reusable block that collects a name and a drawn, typed or uploaded signature and posts them to a document-specific endpoint.

**Signed recipient identity.** A value derived from the database name, the record's token and a Contact identifier with the platform's database secret. Carried in a personal share link, it proves to the portal that the visitor is that Contact, and lets them post, react and edit under that name without an account.

**Sign-up scope.** The setting that decides whether anyone may create an account (`b2c`) or only a person holding a token (`b2b`). It governs the share split. Owned by [Identity and Access](../identity-and-access/README.md).

**Sign-up token.** A time-limited value produced for a Contact that proves the right to set a password. Its payload includes the Contact's latest authentication moment, its list of users and its sign-up type, so it stops working as soon as any of those change. Owned by [Identity and Access](../identity-and-access/README.md).

**Sign-up type.** The marker on a Contact that says which kind of token is outstanding for it: an invitation token or a password-reset token. Granting portal access sets it; revoking clears it.

**Trusted device (second factor).** A device on which the second factor has been remembered, so that a later sign-in from it does not ask for a code. Listed on the portal security page and revocable there. Owned by [Identity and Access](../identity-and-access/README.md).

**User Deletion Request.** A queued request to remove an external account, created when a person confirms the deletion dialog and processed by a daily scheduled job. Owned by [Identity and Access](../identity-and-access/README.md).

**Website editor.** A person allowed to change the published pages of the website. In this domain, such a person may publish a reply under a rating without having write permission on the rated record, and may switch the optional layout variants on or off.
