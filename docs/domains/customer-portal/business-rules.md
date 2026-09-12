# Business rules of the Customer Portal

The numbered rule catalogue of the Customer Portal. Every rule states the condition, the consequence and, when the system emits one, the exact message. Rules are cited from [workflows.md](workflows.md), [entities.md](entities.md), [state-machines.md](state-machines.md), [interfaces.md](interfaces.md) and [acceptance-criteria.md](acceptance-criteria.md).

**About the messages.** Messages are reproduced as the system emits them. Two substitutions are applied throughout this specification:
- the words that name the tax identification number are written in full as "tax identification number"; the stored identifier of the field is reproduced as `vat` and the label the address pages render is the reproduced single word `VAT` (see section 0.3 of [entities.md](entities.md));
- the words that name an application key are written in full as "application key".
Placeholders written as `%s` or `%(name)s` are the system's own substitution markers and are filled with the value named in the rule.

**About the comparisons.** Every comparison of a supplied secret with a stored secret in this domain (security token, signed recipient identity, link signature) is a constant-time comparison with respect to the number of matching leading characters. A replacement that uses an ordinary short-circuiting string comparison leaks the secret through response timing and does not satisfy this specification.

---

## 1. Granting, revoking and re-inviting portal access

### PORT-RULE-001 - A Contact that already has access cannot be granted access again
**Condition**: the invitation line has `is_portal` true or `is_internal` true.
**Consequence**: the grant is refused and nothing is written.
**Message**: `The partner "%s" already has the portal access.` where `%s` is the Contact's name.
**Note**: the rule covers employees deliberately: a person who is an employee, even an archived one, must be managed from the user administration screens, never from the invitation dialog.

### PORT-RULE-002 - An invitation requires a usable electronic mail address
**Condition**: the invitation line has `email_state` equal to `ko`, that is, the address is empty or does not normalize to a single valid address.
**Consequence**: the grant or the re-invitation is refused and nothing is written.
**Message**: `The contact "%s" does not have a valid email.` where `%s` is the Contact's name.

### PORT-RULE-003 - An invitation requires an address that no other user already uses as a login
**Condition**: the invitation line has `email_state` equal to `exist`, that is, a user other than the line's own linked user already has the normalized address as its login (archived users included).
**Consequence**: the grant or the re-invitation is refused and nothing is written.
**Message**: `The contact "%s" has the same email as an existing user` where `%s` is the Contact's name. (The message carries no final period; reproduce it as written.)
**Narrowing**: on an installation that serves several websites the condition is narrowed per website by `PORT-RULE-175`.

### PORT-RULE-004 - A corrected address is written back onto the Contact
**Condition**: the invitation line has `email_state` equal to `ok` and the normalized form of the line's address differs from the normalized form of the Contact's address.
**Consequence**: the Contact's electronic mail address is replaced by the normalized form. The rule runs at the start of the grant, the revocation and the re-invitation, so even a revocation may correct the address.

### PORT-RULE-005 - A new external user is created in the Contact's own company
**Condition**: the grant must create a user.
**Consequence**: the company used for the creation is the Contact's company when it has one, and the acting user's current company otherwise. The created user's allowed companies are exactly that one company.
**Rationale**: an external person must never be allowed into a company they do not belong to, regardless of the company the granting employee happens to be working in.

### PORT-RULE-006 - Only a live external user can be revoked
**Condition**: the invitation line has `is_portal` false.
**Consequence**: the revocation is refused and nothing is written.
**Message**: `The partner "%s" has no portal access or is internal.` where `%s` is the Contact's name.

### PORT-RULE-007 - A revoked user keeps the portal group
**Condition**: a revocation succeeds.
**Consequence**: the user is archived; it is **not** moved to the public group. The public group is reserved for automated tasks and anonymous visitors, so reusing it for a revoked person would grant that person the anonymous visitor's record rules.

### PORT-RULE-008 - Only a live external user can be re-invited
**Condition**: the invitation line has `is_portal` false.
**Consequence**: the re-invitation is refused and nothing is written.
**Message**: `You should first grant the portal access to the partner "%s".` where `%s` is the Contact's name.

### PORT-RULE-009 - The invitation message template must exist
**Condition**: the shipped template that carries the invitation body cannot be resolved.
**Consequence**: the send is refused.
**Message**: `The template "Portal: new user" not found for sending email to the portal user.`

### PORT-RULE-175 - The already-used-as-a-login detection is narrowed per website
**Condition**: the Website and Storefront capability package is installed, so that a login is unique per website instead of globally, and `email_state` is being computed for the lines of an invitation dialog.
**Consequence**: the three extension points of the detection are overridden, and `PORT-RULE-003` therefore fires on fewer lines:
- the candidate search is narrowed by the term `website_id IN W`, where `W` is built by walking the addressed lines in order, appending the Contact's website when it is set and not yet present, and appending both "no website" and the website serving the current request the first time a line's Contact has no website;
- the projection reads the candidate's `website_id` in addition to its `id` and its `login`;
- a candidate only counts as an existing registration when, additionally: the line's Contact has a website and the candidate has the same one; or the line's Contact has no website and the candidate either has no website or belongs to the website serving the current request.
**Consequence when the package is not installed**: the base behavior applies unchanged and no website is consulted.
**Note**: a replacement that omits this narrowing computes `email_state` wrongly on an installation that serves several websites, refusing grants for addresses that are free on the website the Contact belongs to. The three extension points, their base behavior and the override are specified in [entities.md](entities.md) sections 4.3.9 to 4.3.11 and 4.6, with worked numbers in [calculations.md](calculations.md) section 7.1.

---

## 2. Sharing a document

### PORT-RULE-010 - The share split decides between a plain link and a sign-up link
**Condition**: the Send action of the share dialog.
**Consequence**:
- when the shared document carries a non-empty security token, **or** the `auth_signup.invitation_scope` system parameter is `b2b`: every recipient receives the plain token link;
- otherwise: recipients that already have a user receive the plain token link, and the remaining recipients receive an individual sign-up link.

### PORT-RULE-011 - A share link is personal
**Condition**: a plain token link is built for a named recipient.
**Consequence**: the address carries the recipient's Contact identifier **and** the signed recipient identity for that Contact, in addition to the security token. Two recipients of the same document therefore hold two different addresses, and each of them posts messages under their own name.

### PORT-RULE-012 - A share with an access warning must not be sent
**Condition**: the shared document's `access_warning` is not the empty string.
**Consequence**: the Send button is hidden and the warning is shown in a banner instead. **Industry-standard completion**: a replacement must additionally refuse the send server-side when the warning is not empty, because a control that exists only in the browser is not a control.

### PORT-RULE-013 - Building a share address requires read permission
**Condition**: a share address is built with the token included.
**Consequence**: the acting user's read permission on the record is checked before the token is created or read; a failure raises the platform's standard read refusal. The address builder used by the record pages (`get_portal_url`) deliberately skips this check, because its callers have already resolved the record.
**Message**: the read variant of the standard access refusal composed in `PORT-RULE-031`, that is `You are not allowed to access '%(document_kind)s' (%(document_model)s) records.` followed by the allowed-groups sentence and the closing sentence. When the refusal comes from a record rule rather than from the access-right table, the platform appends its own record-level explanation; this domain adds nothing to it.

### PORT-RULE-014 - The invitation note is an internal note
**Condition**: an invitation is sent from the share dialog.
**Consequence**: the message posted on the document uses the private-note subtype, so it is visible to employees in the back-office thread and is **never** shown on the portal page of the document, while the recipient still receives it by electronic mail.

### PORT-RULE-015 - The invitation is rendered in the recipient's language
**Condition**: an invitation is sent from the share dialog.
**Consequence**: the rendering language is switched to the recipient's language for the duration of that one send and restored afterwards, so a list of recipients with different languages produces one message per language.

---

## 3. Reaching a document from the portal

### PORT-RULE-020 - The shared document access check
**Condition**: any portal endpoint that resolves a document by its identifier.
**Consequence**, in this order:
1. Browse the record with elevated rights. When it does not exist, refuse with `This document does not exist.` as a missing-record refusal.
2. Check the acting identity's read permission on the record. When it succeeds, return the record read with elevated rights.
3. When it fails: when a token was supplied, the record's token is not empty and the two are equal under a constant-time comparison, return the record read with elevated rights.
4. Otherwise re-raise the permission refusal.
**Consequence at the page level**: every shipped document page answers a missing-record or permission refusal with a redirection to `/my`, never with an error page, so that a stale link does not leak whether a record exists.

### PORT-RULE-021 - Only three report kinds are served
**Condition**: a report request whose kind is not one of the rich-text, portable-document or plain-text kinds.
**Consequence**: refused.
**Message**: `Invalid report type: %s` where `%s` is the requested value.

### PORT-RULE-022 - A report is rendered for one company at a time
**Condition**: the record set to render carries a company field and spans more than one company.
**Consequence**: refused.
**Message**: `Multi company reports are not supported.`

### PORT-RULE-023 - A report is rendered in the document's company
**Condition**: the record set carries a company field with exactly one company.
**Consequence**: the rendering company is switched to that company before the render, so the layout, the logo, the paper format and the company-scoped settings of the document's own company are used, not those of the reader.

### PORT-RULE-024 - A downloaded portable document is named after the record
**Condition**: a portable document is served.
**Consequence**: the content disposition names the file with the record's report base name, in which every run of characters that are neither letters, digits nor the underscore is replaced by a single underscore, followed by a dot and the extension `pdf`. The disposition is an attachment when the request asked to download and inline otherwise.

### PORT-RULE-025 - A token is never invalidated by reading
**Condition**: any read of a document through a token.
**Consequence**: the token is unchanged. A token is invalidated only when the owning domain clears or replaces it; there is no use counter and no expiry.

---

## 4. The invitation wizard itself

### PORT-RULE-030 - The Contact expansion of an invitation session
**Condition**: an invitation session is created.
**Consequence**: the Contacts in scope are the union, over every Contact given in the context, of that Contact and of its child Contacts whose address kind is `contact` or `other`. Duplicates are removed. Addresses of kind `invoice` and `delivery` are deliberately excluded, because they are places rather than people.

### PORT-RULE-031 - Only a Contact manager may open the invitation session
**Condition**: any read, write or create on the invitation session or on its lines.
**Consequence**: allowed only for members of the Contact manager group `base.group_partner_manager`; refused for every other user. Deletion is granted to nobody, so a delete is refused even for a Contact manager.
**Message**: the platform's standard model-access refusal, which is composed of three reproduced parts: a header that depends on the operation — `You are not allowed to access '%(document_kind)s' (%(document_model)s) records.` for a read, `You are not allowed to modify '%(document_kind)s' (%(document_model)s) records.` for a write, `You are not allowed to create '%(document_kind)s' (%(document_model)s) records.` for a create and `You are not allowed to delete '%(document_kind)s' (%(document_model)s) records.` for a delete, where the first placeholder is the human name of the entity (`Portal Sharing`, `Grant Portal Access` or `Portal User Config`) and the second is its transport name; then either `This operation is allowed for the following groups:` followed by the list of groups that do allow it, or `No group currently allows this operation.` when none does; then `Contact your administrator to request access if necessary.`
**Worked message**: a read attempted by an employee who is not a Contact manager on the invitation session produces `You are not allowed to access 'Grant Portal Access' (portal.wizard) records.`, then the group list naming the Contact manager group, then the closing sentence.

### PORT-RULE-032 - Invitation lines must exist before they can be acted upon
**Condition**: the dialog is opened.
**Consequence**: the session and its lines are created first and the dialog is opened on the stored session, because a line that has no identifier renders its buttons as disabled.

### PORT-RULE-033 - Every per-line action reopens the dialog
**Condition**: a grant, a revocation, a re-invitation or a state-icon press.
**Consequence**: the action returns the dialog action for the parent session, so the dialog stays open and every line is re-derived.

---

## 5. Redirection of external people

### PORT-RULE-040 - Signing in without an explicit destination sends a non-employee to the portal
**Condition**: a successful sign-in with no requested destination, by a user that is not an employee.
**Consequence**: the destination is forced to `/my`.

### PORT-RULE-041 - An external person may not reach the back office
**Condition**: a signed-in session whose user is not an employee requests the platform entry point or the web client entry point.
**Consequence**: the request is answered with a redirection to `/my`, carrying the original query parameters unchanged.

### PORT-RULE-042 - The generic fallback of the redirection endpoint depends on the identity
**Condition**: the generic redirection endpoint cannot resolve a usable destination.
**Consequence**:
- an anonymous visitor is sent to the sign-in page with the original redirection address preserved as the destination;
- a signed-in external user is sent to `/my`;
- a signed-in employee is sent to the messaging screen.

### PORT-RULE-043 - A token in a redirection link overrides missing permissions
**Condition**: the generic redirection endpoint receives a model that adopts the Portal Access Mixin, the identity cannot read the record, and the supplied token equals the record's token.
**Consequence**: the access action is computed with the front-end redirection forced; when it is an "open web address" action, the recipient identifier and the signed identity are appended when both were supplied, and the visitor is redirected there.

### PORT-RULE-044 - The access action of a portal-enabled document
**Condition**: the platform asks a record that adopts the Portal Access Mixin for the action that opens it.
**Consequence**:
- when a target user is named and the acting user cannot read the record, the inherited back-office action is returned unchanged;
- when the resolved user is an external user or the front-end redirection is forced:
  - and the record is readable by that user: an "open web address" action pointing at the portal page with the token appended;
  - and the record is not readable and the front-end redirection is forced: an "open web address" action pointing at the bare portal page with no token;
  - and the record is not readable and the redirection is not forced: fall through to the inherited back-office action;
- otherwise the inherited back-office action.

---

## 6. The portal home

### PORT-RULE-050 - Resolving the assigned representative
**Condition**: any portal page is rendered for a signed-in user.
**Consequence**: the representative shown in the sidebar is, in order: the salesperson of the acting user's own Contact when that Contact has one and that salesperson is not the anonymous public user; otherwise the salesperson of the Contact's commercial entity when it is not the anonymous public user; otherwise none, in which case the contact block is not rendered.

### PORT-RULE-051 - A counter is zero when the reader may not read the model
**Condition**: a counter is requested for a model on which the acting user has no read permission.
**Consequence**: the counter is zero, and no query is issued against that model.

### PORT-RULE-052 - A card is hidden until its counter is known to be non-zero
**Condition**: the portal home is rendered.
**Consequence**: every card that has a counter is rendered hidden, unless the session remembers that this counter was non-zero on a previous load, or the card is declared as always shown. Once the counters arrive, a card is revealed when its value is not zero or when it is in the always-shown list. Cards without a counter (the address book and the security page) are always visible.

### PORT-RULE-053 - The counter cache stores presence, not values
**Condition**: a counter call answers.
**Consequence**: for every returned name that ends in `_count`, the session cache stores a boolean saying whether the value was non-zero. The cache is written back to the session only when it differs from the stored one, so that a reader who always sees the same cards does not rewrite their session on every page load.

### PORT-RULE-054 - The home page itself computes no counter
**Condition**: the home page is rendered.
**Consequence**: the counter preparation is called with an empty list of requested names, so the initial rendering issues no counting query. Every number is fetched afterwards by the browser.

---

## 7. Messaging on portal pages

### PORT-RULE-060 - Posting permission is resolved from the model's declared posting permission
**Condition**: a post, a reaction or the initialization of a portal discussion thread.
**Consequence**: the model declares which permission a person must have on the record in order to post on it. The thread is re-resolved with that permission; when the model declares no permission, posting is refused. The resolution accepts the same three proofs as reading: direct permission, a valid security token, or a valid signed recipient identity.
**Message**: none. The refusal is silent by design: the thread simply does not resolve, the initialization response carries no thread and therefore no composer, and a post attempted anyway is answered with the transport's not-found response rather than with a sentence. A replacement must not invent an error text here, because the absence of one is what prevents a caller from probing which records exist.

### PORT-RULE-061 - What a portal discussion thread shows
**Condition**: any fetch of messages for a portal page.
**Consequence**: the condition applied is the intersection of:
1. the page's own extra condition (empty by default; the rating bridge adds `rating_value = <requested value>` when the caller asks for one);
2. `model = <the entity transport name> AND message_type IN ("comment", "email", "email_outgoing", "auto_comment", "out_of_office") AND res_id = <the record> AND is_internal = false AND subtype IS SET AND subtype.internal = false`;
3. `body NOT IN (empty, the emptied-message marker) OR attachments IS NOT EMPTY`, widened by the rating bridge to `body NOT IN (empty, the emptied-message marker) OR attachments IS NOT EMPTY OR rating_value IS SET`.
**Consequence for employees**: the restriction is not relaxed for employees. An employee reading a portal page sees exactly what the customer sees.

### PORT-RULE-062 - The author of a message posted by an anonymous visitor
**Condition**: a post whose acting user is the anonymous public user.
**Consequence**: the portal Contact is resolved as follows and, when found, becomes the message author:
1. when the supplied signed identity matches the signature of the supplied recipient identifier for this record, the Contact named by that identifier;
2. otherwise, when the supplied token matches the record's token, the first Contact among the record's mail recipients;
3. otherwise nothing, and the message is stored without an author.

### PORT-RULE-063 - An anonymous visitor may edit their own messages
**Condition**: an edit request whose acting user is the anonymous public user, on a message that names a model and a record.
**Consequence**: the portal Contact is resolved with the supplied signed identity, recipient identifier and token; the edit is allowed when that Contact equals the message author, and otherwise the inherited decision applies.

### PORT-RULE-064 - The author of a reaction by an anonymous visitor
**Condition**: a reaction whose inherited author resolution yields nothing, on a message that names a model and a record.
**Consequence**: the portal Contact is resolved as in `PORT-RULE-062`; when one is found it becomes the reaction author and the guest author is cleared.

### PORT-RULE-065 - A reader is recognized as the author of their own messages
**Condition**: the portal projection runs with a portal Contact and a portal thread in the rendering context.
**Consequence**: every message whose author is that Contact and whose record is that thread is flagged as written by the current reader, which is what enables the edit and delete controls and what adds the attachment ownership token.

### PORT-RULE-066 - The signature that proves a recipient identity
**Condition**: a signed recipient identity is produced or verified.
**Consequence**: the value is the keyed-hash message authentication code of the textual representation of the triple (database name, value of the model's declared external-posting token field, recipient Contact identifier), using the platform's database secret as the key and the secure hash algorithm producing a 256-bit digest as the hash function, rendered as lowercase hexadecimal.
**Corollaries**: clearing or replacing the document's token invalidates every previously issued signed identity for that document; a link cannot be replayed against another installation; the signature carries no expiry of its own.

### PORT-RULE-067 - A model without a token field cannot sign a recipient identity
**Condition**: the signing operation is called on a model whose declared external-posting token field is not one of its fields.
**Consequence**: refused.
**Message**: `Model %(model_name)s does not support token signature, as it does not have %(field_name)s field.`

### PORT-RULE-068 - A signed identity may be inherited from a logical parent
**Condition**: the direct signature check fails.
**Consequence**: the model's logical-parent signature is computed and compared; when it matches, the identity is accepted. The shipped use is a Task, whose logical parent is its Project: a person invited to a shared project may post on any of its tasks.

### PORT-RULE-069 - The notification of a document customer carries a working link
**Condition**: a message is notified on a record whose model adopts both the Discussion Thread Mixin and the Portal Access Mixin, and the record has a customer Contact.
**Consequence**: the record's token is created if needed (with elevated rights, justified by the fact that a person with read access who posts must be able to give the other recipients a working link); a dedicated recipient group matching that Contact is placed before every other group, carrying an access button whose address is the generic redirection endpoint with the token, the Contact identifier, the signed identity for that Contact and the Contact's sign-up parameters. In addition, the generic external-user recipient group is forced to be active and to show its access button.

---

## 8. Attachments on portal pages

### PORT-RULE-070 - Removing an attachment requires access to it
**Condition**: a removal request whose attachment cannot be resolved through the shared access check (`PORT-RULE-020`) with the supplied token.
**Consequence**: refused.
**Message**: `The attachment does not exist or you do not have the rights to access it.`

### PORT-RULE-071 - Only a pending attachment may be removed through the portal
**Condition**: the attachment is not attached to the message-composition model, or its record identifier is not zero.
**Consequence**: refused.
**Message**: `The attachment %s cannot be removed because it is not in a pending state.` where `%s` is the attachment name.

### PORT-RULE-072 - An attachment referenced by a message may not be removed
**Condition**: at least one message references the attachment.
**Consequence**: refused.
**Message**: `The attachment %s cannot be removed because it is linked to a message.` where `%s` is the attachment name.

### PORT-RULE-073 - Portal attachments are served with their own tokens
**Condition**: the portal projection formats an attachment.
**Consequence**: the projected attachment carries a raw-content token and a thumbnail token, so the browser can fetch the bytes and the preview without a session; it carries an ownership token only when the reader is recognized as the author of the message (`PORT-RULE-065`), which is what permits deletion.

### PORT-RULE-074 - Video attachments are typed defensively for one browser family
**Condition**: the requesting browser is Safari and the attachment's media type contains `video`.
**Consequence**: the projected media type is replaced by the generic binary media type, so that the browser downloads the file instead of attempting a ranged playback that it mishandles in this context.

---

## 9. Ratings

### PORT-RULE-080 - The rating composer is offered only to identified readers
**Condition**: the rating widget is rendered.
**Consequence**: the composer is shown when the page does not explicitly disable it **and** the reader is not the anonymous public user. The star average and the count are shown to everybody.

### PORT-RULE-081 - A publisher reply needs an existing rating
**Condition**: the publish endpoint receives an identifier that resolves to no rating.
**Consequence**: the response is the structure `{ "error": "Invalid rating" }` with a successful transport status; nothing is written.

### PORT-RULE-082 - A publisher reply is stamped automatically
**Condition**: a create or a write that carries a non-empty publisher comment.
**Consequence**, before the values reach storage: the guard of `PORT-RULE-083` runs; when no publication moment was supplied it is set to the current moment; when no publisher was supplied it is set to the Contact of the acting user. On creation the guard runs a second time after the rows exist, for the rows that ended with a non-empty comment.
**Rationale**: the two stamped fields are declared read-only precisely so that they behave like a tracking stamp; a client that supplies them is not trusted to be honest, but is not refused either, which is why the guard is what protects the operation.

### PORT-RULE-083 - Who may publish a reply under a rating
**Condition**: a create or a write that carries a non-empty publisher comment.
**Consequence**:
- when the website editor group exists in the installation and the acting user belongs to it, the write is allowed;
- otherwise, the acting user must have write permission on every record that the affected ratings rate; a failure refuses the whole operation.
**Message**: `Updating rating comment require write access on related record` (reproduce the wording as written, including its grammar).

### PORT-RULE-084 - Rating information is added to the projection only on request
**Condition**: the portal projection runs.
**Consequence**: the rating value and the rating link are projected only when the caller passes the "include rating" option. When they are projected, the per-record rating statistics are added as well for every record that exposes them, read with elevated rights.

---

## 10. Addresses: structure and permissions

### PORT-RULE-090 - What the address book lists
**Condition**: the address book page is rendered for a signed-in person.
**Consequence**:
- the billing list is the set of Contacts that are descendants of the person's commercial entity whose address kind is `invoice` or `other`, plus the commercial entity itself, plus the person's own Contact;
- the delivery list is the set of Contacts that are descendants of the commercial entity whose address kind is `delivery` or `other`, plus the commercial entity itself, plus the person's own Contact;
- both lists are read with elevated rights and ordered by identifier descending;
- when the person is not their own commercial entity, the commercial entity is removed from the billing list when its mandatory billing fields are not all filled, and from the delivery list when its mandatory delivery fields are not all filled.
**Rationale for the last point**: a child of a company may not edit the company address, so offering an incomplete company address would be a dead end.

### PORT-RULE-091 - Which addresses a person may edit
**Condition**: any address form load, address submission or address archiving that names a Contact.
**Consequence**: the Contact must satisfy the editability predicate, that is, it must be the acting person's own Contact, or a descendant of the acting person's commercial entity whose address kind is `invoice`, `delivery` or `other`. A failure is answered with a forbidden response.
**Message**: none of this domain's own. The response is the transport's forbidden response, status code 403, carrying the platform's standard refusal page. No sentence naming the Contact is produced, deliberately: an address form that said which Contact it refused would confirm the existence of a Contact the reader may not see. A replacement must keep the refusal contentless for the same reason.
**Corollary**: the personal Contact of a colleague (kind `contact`) can never be edited from the portal.

### PORT-RULE-092 - What the address form is allowed to write
**Condition**: an address submission.
**Consequence**: only inputs whose name is both a Contact field and a member of the front-end writable set are converted and kept; every other input is set aside as extra form data and offered to the extension point. Text values are trimmed. A value is kept even when it is empty, so that a field can be cleared. A relation to one record given as a run of digits is converted to that identifier. After parsing, when `zipcode` was submitted and `zip` was absent or empty, the postal code takes the value of `zipcode`.
**The writable set** is: name, telephone, electronic mail address, street, street second line, city, country subdivision, country, postal code (under both input names), the tax identification number and the company name. Other capability packages extend it; the electronic-invoicing capability adds the preferred sending method and the preferred interchange format on the account page.

### PORT-RULE-093 - What a newly created address is completed with
**Condition**: an address submission with no Contact identifier.
**Consequence**: before creation the values receive: the request language; the company of the current customer; the address kind, which is `other` when the form uses the delivery address as the billing address, `invoice` when the requested kind is billing and `delivery` when the requested kind is delivery; and the parent, set to the commercial entity only when that entity is active. The creation runs with elevated rights, with change tracking disabled, and with the tax-identification-number check suppressed because validation already performed it.

### PORT-RULE-094 - An unchanged name is not written
**Condition**: an address update whose submitted name, trimmed, equals the stored name, trimmed.
**Consequence**: the name is removed from the values before the write.
**Rationale**: writing the name propagates to the account holder name of the person's bank accounts; that propagation must not fire when nothing actually changed.

### PORT-RULE-095 - The company name typed on a child address renames the company
**Condition**: an address submission carries a company name, the edited Contact is not its own commercial entity, and the commercial entity is a company.
**Consequence**: the edited Contact's own company-name field is cleared; when the submitted name is non-empty and differs from the commercial entity's name, the commercial entity is renamed.

### PORT-RULE-096 - An unchanged address is not written at all
**Condition**: an address update whose parsed values are all equal, field by field in the internal representation, to the stored values, treating two empty values as equal.
**Consequence**: no write is issued; the response is nevertheless the success response with the redirection address.

### PORT-RULE-097 - A person may not archive their own main address
**Condition**: an archive request naming the acting user's own Contact.
**Consequence**: refused.
**Message**: `You cannot archive your main address`

---

## 11. Addresses: validation

The validation produces three results: the invalid input names, the missing input names, and an ordered list of messages. The response merges the first two into one list. When the message list is not empty, nothing is written.

### PORT-RULE-100 - The country is frozen once documents have been issued
**Condition**: an update in which a country was submitted, the Contact already had a country, the submitted country differs, and the "may edit the country" predicate is false.
**Consequence**: the country input is marked invalid and the message is added.
**Message**: `Changing your country is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`
**The predicate** is true when: no posted customer invoice or customer credit note exists whose Contact is exactly this Contact, **and** no sales order in state "sent" or "confirmed" exists whose order Contact or invoicing Contact is exactly this Contact. Domains that are not installed contribute no condition.

### PORT-RULE-101 - An employee may not change their name or address from the portal form
**Condition**: an update in which the name or the electronic mail address changed, and **not every** user of the edited Contact is an external user.
**Consequence**: the changed inputs are marked invalid and the message is added.
**Message**: `If you are ordering for an external person, please place your order via the backend. If you wish to change your name or email address, please do so in the account settings or contact your administrator.`
**Note**: a Contact with no user at all satisfies "every user is external" vacuously, so an anonymous address created by an employee for a customer can still be renamed from the portal.

### PORT-RULE-102 - Commercial fields are managed on the commercial entity
**Condition**: an update of a Contact that is not its own commercial entity, in which a commercial field was submitted.
**Consequence**, per commercial field:
- when the submitted value differs from the stored one and at least one of the two is non-empty: the field is marked invalid and a message is added, which is `The %(field_name)s is managed on your company account.` when the commercial entity is a company, and `The %(field_name)s is managed on your main account address.` otherwise, where the placeholder is the human label of the field;
- otherwise the field is removed from the values so that it is never written.
In addition, when the edited Contact is not the current customer, the company name is removed from the values.
**The commercial field set** is the synchronized commercial fields (the tax identification number) plus the company registry number and the industry, extended by the accounting capability with the payable account, the receivable account, the fiscal position, the customer payment terms, the vendor payment terms and the credit limit, and further extended by country packages with their own identification fields.

### PORT-RULE-103 - The tax identification number is frozen once documents have been issued
**Condition**: an update of a Contact that **is** its own commercial entity, in which a tax identification number was submitted, the Contact already had one, the submitted value differs, and the "may edit the tax identification number" predicate is false.
**Consequence**: the input is marked invalid and the message is added.
**Message**: `Changing the tax identification number is not allowed once document(s) have been issued for your account. Please contact us directly for this operation.`
**The predicate** is true when: the Contact has no parent, **and** no posted customer invoice or customer credit note exists for any Contact under the commercial entity, **and** no sales order in state "sent" or "confirmed" exists for any Contact under the commercial entity.

### PORT-RULE-104 - The electronic mail address must be a single valid address
**Condition**: an address was submitted and it does not match the single-address pattern.
**Consequence**: the input is marked invalid and the message is added.
**Message**: `Invalid Email! Please enter a valid email address.`
**Examples of rejected values**: `hello`, `hello@.com`, `hello@oo.`

### PORT-RULE-105 - The tax identification number is checked against the country rules
**Condition**: a number was submitted, the accounting capability is installed, and the number is not already marked invalid.
**Consequence**: a throwaway Contact carrying only the country and the number is built and submitted to the accounting domain's number check; a failure marks the input invalid and adds the checker's own message verbatim. See [Taxes](../taxes/README.md) for the checking rules.

### PORT-RULE-106 - Building the required set
**Condition**: every address submission.
**Consequence**, in this order:
1. Start from the comma-separated list of input names submitted by the form (the account and address forms submit `name,email`).
2. When the requested kind is delivery or the form uses the delivery address as the billing address, add the mandatory delivery fields.
3. When the requested kind is billing or the form uses the delivery address as the billing address, add the mandatory billing fields; then, when the edited address is not the commercial address, remove from the required set every commercial field that was **not** submitted.
4. When any of the common address fields was filled, add all of the common address fields.
5. Every required input whose value is empty is added to the missing set.
6. When the missing set is not empty, add the message.
**Message**: `Some required fields are empty.`

```
common_address_fields(country) = { street, city, country }
                               ∪ { state } when country.state_required is true
                               ∪ { zip }   when country.zip_required is true

mandatory_billing_fields(country)  = { name, email } ∪ ( { phone } ∪ common_address_fields(country)  when the page needs an address )
mandatory_delivery_fields(country) = { name, email } ∪ ( { phone } ∪ common_address_fields(country)  when the page needs an address )
```

### PORT-RULE-107 - The country layout drives the form
**Condition**: the country of the address form changes, or the form loads.
**Consequence**: the server returns the country's ordered address field names, whether the postal code precedes the city, the country's subdivisions, the country's telephone prefix and the mandatory field names for the requested address kind. The form then: sets the telephone placeholder to the prefix or empties it; fills and shows the subdivision selector when the country has subdivisions and empties and hides it otherwise; reorders the postal code and the city; shows or hides the street, the postal code and the city; and moves the required marker so that exactly the newly mandatory inputs plus the form's own base required list are required.

---

## 12. Password and session security

### PORT-RULE-110 - The three password inputs are validated before anything is attempted
**Condition**: the password change form is submitted.
**Consequence**, in this order:
1. Each of the three values is trimmed. Any empty one answers `You cannot leave any password empty.` against that input and stops.
2. A confirmation that differs from the new password answers `The new password and its confirmation must be identical.` against the confirmation input and stops.
3. The identity domain's change operation is called. A credential refusal whose message is the generic credential refusal is replaced by `The old password you provided is incorrect, your password was not changed.` and reported against the old-password input; any other credential refusal is reported verbatim against the old-password input; a business refusal (for example the empty-password refusal or a password-policy refusal) is reported as a section-level message.

### PORT-RULE-111 - Changing the password does not sign the person out
**Condition**: the password change succeeds.
**Consequence**: the session token is recomputed from the new credentials and written into the session, so the current session survives while every other session of that person is invalidated by the credential change.

### PORT-RULE-112 - The account and security pages may only be framed by the platform itself
**Condition**: the account page or the security page is served.
**Consequence**: the response carries both a frame-options header restricted to the same origin and a content-security policy restricting frame ancestors to the same origin.

### PORT-RULE-113 - Revoking every session requires re-authentication
**Condition**: the revoke-all-sessions operation is invoked.
**Consequence**: the operation first answers with an identity-check request; the browser shows the dialog `Security Control` with the text `Please enter your password to confirm you own this account`; the check is run with the password as the chosen method; only on success is the original operation replayed. Every device of the person except the current one is then revoked.

### PORT-RULE-114 - A password-policy minimum length reaches the portal form
**Condition**: the password-policy bridge is installed.
**Consequence**: the configured minimum length is added to the shared layout values of every portal page, and the new-password input of the security page carries it as a client-side minimum, next to a strength meter. Server-side enforcement remains with the identity domain.

### PORT-RULE-115 - The second-factor enrollment layout is embedded, not fetched
**Condition**: the security page is rendered and the second-factor bridge for the portal is installed.
**Consequence**: the combined layout of the enrollment wizard is read with elevated rights and embedded in the page as a hidden data island, because an external person has no permission to read back-office layouts.

### PORT-RULE-116 - An external person may use the enrollment wizard
**Condition**: the second-factor bridge for the portal is installed.
**Consequence**: the portal group is granted read, write, create and delete on the enrollment wizard; the platform's global rule that a person may only act on their own enrollment still applies.

### PORT-RULE-117 - The second-factor invitation points at the portal for a non-employee
**Condition**: an administrator sends the "please enable a second factor" invitation to a user that is not an employee.
**Consequence**: the address in the invitation is `/my/security`, not the back-office one.

---

## 13. Application keys for external people

### PORT-RULE-120 - The application key section is shown only in developer mode and only when allowed
**Condition**: the security page is rendered.
**Consequence**: the section is rendered only when the request runs in developer mode **and** the `portal.allow_api_keys` system parameter (the "customers may create application keys" switch) is set.

### PORT-RULE-121 - An external person may create an application key only when the setting allows it
**Condition**: the permission check that guards key creation refuses the acting user.
**Consequence**:
- when the `portal.allow_api_keys` system parameter is set and the acting user is an external user: allowed;
- when it is set and the acting user is neither an employee nor an external user: refused with `Only internal and portal users can create API keys`;
- when it is not set: the inherited refusal is re-raised unchanged.

### PORT-RULE-122 - An external person may not choose an arbitrary expiration
**Condition**: the key creation dialog is built on the portal.
**Consequence**: the "custom date" duration choice is removed from the list offered, so the key always receives one of the predefined durations.

### PORT-RULE-123 - Key creation and deletion pass through the re-authentication gate
**Condition**: the key wizard is opened, a key is generated or a key is deleted from the portal.
**Consequence**: each of the three operations answers with an identity-check request first; the password dialog is shown; only on success is the operation replayed.

---

## 14. Account deletion

### PORT-RULE-130 - The login must be typed back exactly
**Condition**: the deletion form is submitted with a confirmation value that differs from the acting user's login.
**Consequence**: nothing is written; the security page is re-rendered with the deletion dialog open and the message `You should enter "<login>" to validate your action.` under the confirmation input.

### PORT-RULE-131 - The current password must be supplied
**Condition**: the credential check of the deletion form fails.
**Consequence**: nothing is written; the security page is re-rendered with the deletion dialog open and the message `Wrong password.` under the password input.

### PORT-RULE-132 - Only external people may delete their own account
**Condition**: the deactivation operation is called on a set that contains at least one user that is not an external user.
**Consequence**: the whole operation is refused as a credential refusal.
**Message**: `Only the portal users can delete their accounts. The user(s) %s can not be deleted.` where `%s` is the comma-separated list of the offending users' names.

### PORT-RULE-133 - Deletion first makes the account unusable, then queues the removal
**Condition**: a successful deactivation.
**Consequence**, in this order, for each user:
1. A note is logged on the Contact: `Archived because %(user_name)s (#%(user_id)s) deleted the portal account`.
2. The login is replaced by `__deleted_user_<user identifier>_<current time as a fractional number of seconds>` and the password is set to the empty string. An empty stored password can never be satisfied by a credential check, so the account is immediately unusable even before it is archived.
3. Every application key of the user is revoked.
4. The user is archived, running as the system robot user because a person may not deactivate themselves; a failure is swallowed.
5. The Contact is archived; a failure is swallowed.
6. One User Deletion Request is created with `state = todo`.
**Rationale for swallowing the archiving failures**: the account must end up unusable and queued even when a constraint prevents the archiving; the queue processing will retry the actual removal.

### PORT-RULE-134 - The block-list request is honoured per channel
**Condition**: the deletion form carried the block-list choice.
**Consequence**:
- every electronic mail address of the deleted users that normalizes successfully is added to the electronic mail block list with the reason `Blocked by deletion of portal account %(portal_user_name)s by %(user_name)s (#%(user_id)s)`;
- every telephone number of the deleted users that formats successfully is added to the telephone block list, each carrying a note with the same reason.
When the choice was not made, neither list is touched.

### PORT-RULE-135 - The queue is processed in bounded batches
**Condition**: the daily scheduled run.
**Consequence**: requests whose user no longer exists are set to `done` immediately; the first fifty of the remaining ones are processed; each is locked before being processed, and a request already taken by a concurrent run is skipped; the run stops early when its time budget is exhausted, leaving the remaining requests in `todo` for the next run.

### PORT-RULE-136 - A failed user deletion is recorded, not retried in the same run
**Condition**: the removal of a user raises.
**Consequence**: the work of that request is rolled back, an error line is written and the request is set to `fail`. A request in `fail` is never picked up again by the scheduled run, because the run only searches for `todo`.

### PORT-RULE-137 - A failed Contact deletion does not fail the request
**Condition**: the user was removed but the Contact cannot be removed, typically because a document references it.
**Consequence**: the work of that step is rolled back, a warning line is written, and the request stays `done`. The Contact stays archived.

---

## 15. Tokens, addresses and presentation

### PORT-RULE-140 - A security token is created lazily and exactly once
**Condition**: any operation that needs a document's token.
**Consequence**: when the column is empty, a version-4 random universally unique identifier is written into it with elevated rights (a write, not an in-memory assignment, so that the cache is invalidated); the stored value is then returned. A record that already has a token keeps it.

### PORT-RULE-141 - A security token is never copied
**Condition**: a record that adopts the Portal Access Mixin is duplicated.
**Consequence**: the copy has an empty token. Every link issued for the original keeps pointing at the original.

### PORT-RULE-142 - A security token may only be filtered by membership
**Condition**: a filter on the token column.
**Consequence**: only the two membership operators, *is in* and *is not in*, are supported; every other operator is refused, which forbids pattern matching and ordering comparisons on a secret. The field's own search rule answers "not implemented" for any other operator, and the platform then raises the refusal below.
**Message**: `Unsupported operator on %(field_label)s %(model_label)s in %(domain)s`, where the first placeholder is the label of the field (`Security Token`), the second is the human name of the entity followed by its transport name in parentheses, and the third is the filter expression that was submitted.

### PORT-RULE-143 - The portal web address of a document is owned by the document's domain
**Condition**: the portal web address is read.
**Consequence**: the mixin's own value is the single character `#`; each adopting model replaces it. A model that adopts the mixin without replacing it therefore has a token but no page, which is a valid configuration: such a document is reachable only through the endpoints its own domain provides.

### PORT-RULE-144 - Every list page keeps the last one hundred identifiers in the session
**Condition**: a list page renders its rows.
**Consequence**: the ordered identifiers of the rows are stored in the session under the page's history key, truncated to the first one hundred. The record page reads that history to compute the previous and next links.

### PORT-RULE-145 - The previous and next links carry their own tokens
**Condition**: a record page computes its previous and next links.
**Consequence**: when the neighbouring record exposes a portal web address, the link is that address with the neighbour's own token appended; when it exposes only a website address, the link is that address as is; when it exposes neither, there is no link and the control is disabled.

### PORT-RULE-146 - The default page size is eighty rows
**Condition**: any list page that does not override it.
**Consequence**: eighty rows per page. The timesheet page overrides it to one hundred; the discussion thread on a record page shows ten messages per page by default.

### PORT-RULE-147 - A date range on a list page is half open
**Condition**: a list page receives both a range start and a range end.
**Consequence**: the condition added is `create_date > <range start> AND create_date <= <range end>`. The lower bound is exclusive and the upper bound is inclusive.

### PORT-RULE-148 - Sorting keeps the page, filtering and grouping do not
**Condition**: a selector of the search bar is used.
**Consequence**: choosing a sort reloads the **current** path with every existing query parameter preserved and the sort key replaced, so the reader stays on the same page number; choosing a filter or a grouping reloads the page's **default** address with the parameters preserved and the key replaced, so the reader returns to the first page. The reason is that a filter or a grouping changes the number of rows, which can make the current page number meaningless.

### PORT-RULE-149 - The language selector appears only when there is a choice
**Condition**: the front-end layout renders.
**Consequence**: the selector is rendered only when more than one language is published on the front end. Each entry rewrites the current address for that language with the "force the default language" flag, so that the default language is produced without a prefix. The document's text direction and the right-to-left marker on the page wrapper follow the active language.

---

## 16. Permissions

### PORT-RULE-170 - The three wizards are reserved to the Contact manager
| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Portal Share Wizard | Contact manager | yes | yes | yes | no |
| Portal Access Wizard | Contact manager | yes | yes | yes | no |
| Portal Access Wizard User | Contact manager | yes | yes | yes | no |

No other group is granted anything on these three entities.

### PORT-RULE-171 - The record rules that decide what an external person sees
The Customer Portal itself declares no record rule. Each document domain declares its own; the shipped ones are listed here because every portal list page depends on them. In each expression, `commercial_partner_id` denotes the commercial entity of the acting user's Contact.

| Entity | Model permissions for the portal group | Record rule for the portal group |
|---|---|---|
| Sales Order (`sale.order`) | read only | `partner_id IS DESCENDANT OF commercial_partner_id`, with read, write and delete permitted by the rule and create denied |
| Sales Order Line (`sale.order.line`) | read only | `order_id.partner_id IS DESCENDANT OF commercial_partner_id` |
| Journal Entry (`account.move`) | read only (granted by the accounting domain) | `state NOT IN ("cancel", "draft") AND move_type IN ("out_invoice", "out_refund", "in_invoice", "in_refund") AND partner_id IS DESCENDANT OF commercial_partner_id` |
| Journal Item (`account.move.line`) | read only | `parent_state NOT IN ("cancel", "draft") AND move_id.move_type IN ("out_invoice", "out_refund", "in_invoice", "in_refund") AND move_id.partner_id IS DESCENDANT OF commercial_partner_id` |
| Purchase Order (`purchase.order`) | read only | `partner_id IS DESCENDANT OF commercial_partner_id`, with read, write and delete permitted by the rule and create denied |
| Purchase Order Line (`purchase.order.line`) | read only | `order_id.partner_id IS DESCENDANT OF commercial_partner_id` |
| Project (`project.project`) | read only | `privacy_visibility IN ("invited_users", "portal") AND message_partner_ids IS DESCENDANT OF commercial_partner_id` |
| Project Collaborator (`project.collaborator`) | read only | `project_id.privacy_visibility IN ("invited_users", "portal") AND partner_id = the acting Contact` |
| Task (`project.task`), read | read only | `project_id.privacy_visibility IN ("invited_users", "portal") AND active = true AND ( message_partner_ids IS DESCENDANT OF commercial_partner_id OR project_id.collaborator_ids ANY ( partner_id = the acting Contact AND limited_access = false ) )`, with read permitted and write, create and delete denied by the rule |
| Task (`project.task`), write and create | granted only by this rule | `project_id.privacy_visibility IN ("invited_users", "portal") AND active = true AND ( ( message_partner_ids IS DESCENDANT OF commercial_partner_id AND project_id.collaborator_ids.partner_id IN (the acting Contact) ) OR project_id.collaborator_ids ANY ( partner_id = the acting Contact AND limited_access = false ) )`, with write and create permitted and read and delete denied by the rule. The rule is shipped inactive and is switched on when the project sharing feature is enabled. |

Note the two asymmetries that a replacement must reproduce: the model permissions of a sales order and of a purchase order grant only reading to the portal group, while their record rules mark read, write and delete as covered; the effective permission is the intersection, so an external person can only read. The purchase order rule uses the commercial entity of the **vendor** side, which is how a vendor sees the orders addressed to them.

### PORT-RULE-172 - A portal page never relaxes a record rule for an employee
**Condition**: an employee opens a portal page.
**Consequence**: the page applies the same conditions as for an external person; in particular the discussion thread shows no internal message. The purpose is that an employee previewing the portal sees what the customer sees.

### PORT-RULE-173 - The portal group and the public group are mutually exclusive
**Condition**: a grant of portal access.
**Consequence**: the portal group is added and the public group is removed in the same write. The user-type groups are mutually exclusive by the rules of [Identity and Access](../identity-and-access/README.md); this domain relies on that exclusivity and never leaves a user in both.

### PORT-RULE-174 - Elevated reads in this domain are bounded
Every use of elevated rights in this domain is justified by a preceding validation, and reads no more than the validation covered:

| Operation | Justification |
|---|---|
| Creating a security token | The caller has already checked read permission (`PORT-RULE-013`), or is the owning domain's own page code. |
| Returning a document after a token match | The token match is the proof of access. |
| Fetching messages for a portal page | Thread access was validated first, and the condition is restricted to publishable messages only. |
| Reading the linked user, and the users that already use one of the submitted addresses as a login, in the invitation wizard | The Contact manager needs to see the state of every line without holding permissions on Users. |
| Reading the country list and the address lists | These are reference data and Contacts inside the reader's own company tree. |
| Creating or writing an address | Editability was validated first (`PORT-RULE-091`). |
| Reading the application key setting | A system parameter is never readable by an external person. |
| Reading the enrollment wizard layout | An external person may not read back-office layouts. |
| Reading rating rows in the projection | An external person has no permission on ratings. |
| The account deactivation | A person may not archive themselves, so the archiving runs as the system robot user. |
