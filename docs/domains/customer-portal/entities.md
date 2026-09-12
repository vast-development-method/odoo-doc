# Entities of the Customer Portal

Every field of the four entities the Customer Portal owns, and every field or behavior it adds to entities owned by other domains, with data types, defaults, derivation rules, copy behavior, constraints, validation messages, on-change behavior and record lifecycle. The Customer Portal owns no persistent entity of its own: its four entities are one abstract contract that document models adopt and three short-lived wizards. The persistent consequences of the domain are written on Contacts, Users, Messages, Ratings and on the documents that adopt the access mixin.

## 0. Conventions

### 0.1 Shared fields

Every persistent entity in the system carries `id` (the surrogate integer primary key), `create_date` (creation moment), `create_uid` (creating user), `write_date` (last modification moment) and `write_uid` (last modifying user). Entities that support archiving carry `active` (boolean, default true; archived rows are excluded from default queries). These fields are not repeated in the tables below.

Transient entities (the three wizards of this domain) also carry those five fields, but their rows are short lived: a transient row is created when the dialog opens, is never referenced by a persistent record, and is removed by the platform's garbage collection once it is older than the transient retention period. Nothing in this domain may depend on a transient row surviving a request.

### 0.2 Kinds of entity used here

- **abstract**: a contract with fields and operations that is never stored on its own; a persistent entity adopts it and then owns the columns.
- **transient**: a row that exists only to back a dialog.

### 0.3 Naming of the tax identification number label

The label shown next to the tax identification number field, and the words used inside the messages that mention it, come from the country configuration of the operating company (the label is a per-country text, for example the local name of the value-added tax registration number). In this specification the placeholder is always written in full as "tax identification number". A replacement must substitute the configured label at render time.

---

## 1. Portal Access Mixin (`portal.mixin`)

**Kind**: abstract. **Adopted by**: every document model that must be reachable from the portal (see section 15).

**Purpose**: give a document a stable public web address, a per-record security token that grants read access to whoever holds it, an optional warning that blocks sharing, and a redirection rule that sends external people to the portal page instead of the back-office form.

### 1.1 Fields

| Canonical name | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Visible or editable to | Meaning |
|---|---|---|---|---|---|---|---|---|
| `access_url` | text | no | `#` | derived, not stored; recomputed on every read | not applicable (not stored) | no | readable by anyone who may read the record | The path of the portal page of this record, relative to the platform root, for example `/my/orders/42`. The mixin's own rule returns the single character `#`; every adopting model overrides the rule with its own path (section 15). |
| `access_token` | text | no | empty | stored | **no** (explicitly excluded from duplication) | no | readable by anyone who may read the record; written only with elevated rights | The per-record security token. A request that carries a value equal to this one is allowed to read the record even when the requester's permissions would refuse it. Generated on first demand as a version-4 random universally unique identifier in its canonical thirty-six character text form. |
| `access_warning` | long_text | no | empty string | derived, not stored | not applicable | no | readable by anyone who may read the record | A message that the adopting model may fill to explain why this record must not be shared. When it is non-empty, the share wizard shows it and hides the send button. The mixin's own rule always returns the empty string. |

### 1.2 Derivation rules

**`access_url`**: for each record in the set, assign the constant `#`. Every adopting model overrides this rule, calls the inherited rule first and then assigns its own path. The field has no declared dependencies, so it is evaluated once per record per environment and is invalidated whenever the cache is cleared.

**`access_warning`**: for each record in the set, assign the empty string. Adopting models override this rule; for example a model may return a sentence explaining that the document is in a state in which sharing would expose data the recipient must not see.

### 1.3 Searching on the token

Filtering on `access_token` is supported only with the two membership operators `IN` and `NOT IN`. Any other operator is rejected as unsupported, which makes the platform fall back to its generic handling and ultimately raise an unsupported-operator error. The supported form translates directly to a comparison of the stored column:

```
search(access_token IN [t1, t2, ...])  ->  access_token IN (t1, t2, ...)
search(access_token NOT IN [t1, ...])  ->  access_token NOT IN (t1, ...)
```

The purpose of the restriction is to forbid pattern matching (`LIKE`, `ILIKE`) and ordering comparisons on a secret.

### 1.4 Operations

#### 1.4.1 `_portal_ensure_token`

Input: exactly one record. Output: the token text.

1. Read `access_token`.
2. If it is empty, write a freshly generated version-4 random universally unique identifier into `access_token` **with elevated rights**. The write is mandatory (not a mere in-memory assignment) because it must invalidate the cache, otherwise the subsequent read would still return the empty value.
3. Return `access_token`.

The operation is idempotent for a record that already has a token. Two concurrent first calls on the same record are serialized by the row lock taken by the write; the loser re-reads the winner's token only if it re-reads after commit, so a replacement must treat the token as "whatever value is committed", never assume uniqueness of generation order.

#### 1.4.2 `_get_share_url`

Inputs: `redirect` (boolean, default false), `signup_partner` (boolean, default false), `recipient_contact` (identifier or empty, default empty), `include_token` (boolean, default true). Applies to exactly one record. Output: a path with a query string.

1. If `redirect` is true, start the parameter set with the record's model name and the record's identifier, so that the generic redirection endpoint can resolve the record; otherwise start with an empty parameter set.
2. If `include_token` is true and the model carries the `access_token` field: check that the acting user may read the record (raising the standard read-refusal error if not), then call `_portal_ensure_token` and add the token to the parameter set.
3. If `recipient_contact` is given: add it to the parameter set and add the signed recipient identity produced by `_sign_token` (section 5.3) for that Contact.
4. If `signup_partner` is true and the model has a customer Contact field with a value: add the sign-up parameters of that Contact (either a sign-up token, or the login of the existing user; see [Identity and Access](../identity-and-access/README.md)).
5. Return `"/mail/view" + "?" + encoded_parameters` when `redirect` is true, and `access_url + "?" + encoded_parameters` otherwise.

#### 1.4.3 `get_portal_url`

Inputs: `suffix` (text, optional), `report_type` (optional, one of `html` for rich text, `pdf` for a portable document, `text` for plain text), `download` (boolean, optional), `query_string` (text, optional), `anchor` (text, optional). Applies to exactly one record. Output: a path with a query string.

```
portal_web_address =
    access_url
  + suffix                                     (empty when not given)
  + "?access_token=" + _portal_ensure_token()
  + ("&report_type=" + report_type  if report_type given else "")
  + ("&download=true"               if download is true  else "")
  + query_string                               (empty when not given; the caller supplies its own leading "&")
  + ("#" + anchor                   if anchor given      else "")
```

The endpoint that serves `access_url` decides whether it honours `report_type` and `download`; the mixin only assembles the address. Unlike `_get_share_url`, this operation does **not** check read permission before creating the token, because it is called by code that has already resolved the record.

#### 1.4.4 `_get_access_action`

Inputs: `access_uid` (the identifier of the user for whom the action is computed, optional), `force_website` (the "force the front end" flag, boolean, default false). Applies to exactly one record. Output: an action description.

1. Let the *resolved identity* be the acting user and the *resolved record* be this record.
2. If `access_uid` is given:
   - Check that the acting user may read the record. If that check fails, delegate to the inherited behavior immediately (the back-office form action) and stop.
   - Set the resolved identity to the user named by `access_uid`, read with elevated rights, and re-bind the resolved record to that identity.
3. If the resolved identity is an external user (its `share` characteristic is true) **or** `force_website` is true:
   - Check that the resolved record may be read by the resolved identity.
   - If the check succeeds, return an "open web address" action whose address is `_get_share_url()` (that is, the portal page plus the token), whose target is the same window, and which carries the record identifier.
   - If the check fails and `force_website` is true, return an "open web address" action whose address is the bare `access_url` (no token), same window, carrying the record identifier.
   - If the check fails and `force_website` is false, fall through to step 4.
4. Delegate to the inherited behavior, which returns the back-office form action for the record.

#### 1.4.5 `action_share`

A model-level operation. It loads the shipped "Share Document" window action, merges into its context the active model and active record identifier taken from the calling context, and returns the action. The result opens the Portal Share Wizard in a medium dialog.

### 1.5 Lifecycle

The mixin adds no lifecycle of its own. The token is created lazily and never rotated automatically: once a document has a token, every link ever sent for that document keeps working until the token column is cleared or overwritten by the owning domain. Duplicating a record produces a record with an empty token, so the copy has no working share link until one is requested.

---

## 2. Portal Share Wizard (`portal.share`)

**Kind**: transient. **Opened from**: the "Share Document" action bound to this wizard, reachable from any record of a model that adopts the Portal Access Mixin.

### 2.1 Fields

| Canonical name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `res_model` | text | yes | the active model of the calling context | stored | The technical name of the model of the document being shared. |
| `res_id` | integer | yes | the active record identifier of the calling context | stored | The identifier of the document being shared. |
| `resource_ref` | reference (model name plus identifier) | no | none | derived from `res_model` and `res_id` | A single-value handle on the shared document, used to read its display name, its base web address and its discussion thread. The selectable models are every model registered in the platform. |
| `partner_ids` | many_to_many to Contact | **yes** | empty | stored | The recipients of the invitation. At least one is required; the dialog cannot be sent with an empty list. |
| `note` | long_text | no | empty | stored | Extra content added to the invitation body, rendered with line breaks preserved. |
| `share_link` | text | no | computed at dialog opening and recomputed on change | derived from `res_model` and `res_id` | The absolute, copy-ready share web address. |
| `access_warning` | long_text | no | empty | derived from `res_model` and `res_id` | Mirrors the shared document's access warning. When non-empty, the send button is hidden and a warning banner is shown. |

### 2.2 Default values at dialog opening

1. Take the inherited defaults.
2. Set `res_model` to the active model of the context (empty when absent).
3. Set `res_id` to the active record identifier of the context (empty when absent).
4. If both are set, browse the record and set `share_link` to `get_base_url() + _get_share_url(redirect = true)` for that record. Note that this default is computed for **any** model, even one that does not adopt the access mixin, which is why the derivation in section 2.3 repeats the check.

### 2.3 Derivation rules

**`resource_ref`** (depends on `res_model`, `res_id`): for each row, if `res_model` is set and names a model known to the platform, assign the reference `"<model name>,<identifier or 0>"`; otherwise assign nothing.

**`share_link`** (depends on `res_model`, `res_id`): for each row, assign empty; then, if `res_model` is set, resolve the model; if that model adopts the Portal Access Mixin **and** `res_id` is set, assign the record's `get_base_url()` followed by its `_get_share_url(redirect = true)`.

The resulting link therefore always points at the generic redirection endpoint with the model, the record identifier and the token, for example:

```
https://<platform host>/mail/view?model=<entity transport name>&res_id=<identifier of the record>&access_token=<security token>
```

**`access_warning`** (depends on `res_model`, `res_id`): for each row, assign empty; then, if `res_model` is set and names a model that adopts the Portal Access Mixin and `res_id` is set, assign the document's `access_warning`.

### 2.4 Operations

#### 2.4.1 `_send_public_link(recipients)`

For each recipient Contact, in order:

1. Compute the share link as the document's `get_base_url()` followed by its `_get_share_url(redirect = true, pid = <the recipient's identifier>)`. Because a recipient is passed, the address carries the recipient identifier **and** the signed recipient identity, which is what later lets that person post messages on the document without an account.
2. Remember the current rendering language, then switch the rendering language to the recipient's language.
3. Post a message on the document, built from the shipped invitation body (section 2.5), with:
   - render values: the recipient Contact, the note, the document record, the share link, and the lowercased human name of the document's model;
   - subject: `Invitation to access <display name of the document>`;
   - subtype: internal note;
   - notification layout: the light notification layout;
   - recipients: this one Contact.
4. Restore the previous rendering language.

#### 2.4.2 `_send_signup_link(recipients)`

Default recipient set when none is passed: the recipients that have no user at all.

For each recipient Contact, in order:

1. Prepare the Contact for sign-up and read its authentication parameters (this writes a sign-up token on the Contact when free sign-up is enabled).
2. Compute `share_link` as the Contact's sign-up web address for the generic redirection endpoint, carrying the document model and identifier as redirection parameters. Following that link takes the person to the sign-up page; after they choose a password, they land on the document.
3. Steps 2 to 4 of section 2.4.1 (language switch, message post with the same body, subject, subtype, layout and single recipient, language restore).

#### 2.4.3 `action_send_mail` (the **Send** button)

1. Read whether free sign-up is enabled: the `auth_signup.invitation_scope` system parameter equals `b2c` rather than `b2b` (the setting is owned by [Identity and Access](../identity-and-access/README.md)).
2. Decide the split:
   - If the document carries a non-empty `access_token`, **or** free sign-up is disabled: every recipient receives the plain token link. Reason: with a token the recipient does not need an account, and without free sign-up no sign-up link can be produced.
   - Otherwise: only the recipients that already have a user receive the plain token link.
3. Call `_send_public_link` with the token-link recipients.
4. Call `_send_signup_link` with the remaining recipients (the difference between the full recipient list and the token-link recipients).
5. Close the dialog.

Note the asymmetry: when a document has a token, recipients without an account still receive the plain link and the token alone proves their right to read; when the document has no token yet, `_get_share_url` inside `_send_public_link` creates one, so in practice step 2 only matters for models whose token column is still empty at the moment the decision is taken.

### 2.5 The invitation body

The shipped body renders as:

```
Dear <recipient name>,

<name of the person sending the invitation> has invited you to access the following <lowercased model name>:

[ Open <document display name> ]      <- a button linking to the share link

<note, when given, with line breaks preserved>
```

### 2.6 Validation and guards

- `res_model`, `res_id` and `partner_ids` are required; a send with an empty recipient list is refused by the required-field check with the platform's standard missing-required-field message.
- The **Send** button is hidden whenever `access_warning` is not the empty string, and the warning banner is shown in its place. There is no server-side re-check: a replacement must enforce the same hiding, and should additionally refuse the send server-side when the warning is non-empty (**industry-standard completion**, because a client-side-only guard is not a guard).

---

## 3. Portal Access Wizard (`portal.wizard`)

**Kind**: transient. **Opened from**: the "Grant portal access" action bound to Contacts, or the server action that creates the wizard first and then opens it.

### 3.1 Fields

| Canonical name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `partner_ids` | many_to_many to Contact | no | the expansion described in section 3.2 | stored | The Contacts in scope for this invitation session. |
| `user_ids` | one_to_many to Portal Access Wizard User (inverse field `wizard_id`) | no | derived from `partner_ids`, stored, and editable afterwards | stored **and** derived | One line per Contact in scope. Recomputed whenever `partner_ids` changes; the user may then edit the electronic mail address on each line. |
| `welcome_message` | long_text | no | empty | stored | Free text appended at the end of the invitation message sent to each new external user. |

### 3.2 Default expansion of `partner_ids`

Input: the `default_partners` context key when present, otherwise the active record identifiers of the calling context.

For each Contact in that input, add to the result:
- its child Contacts whose address kind is `contact` or `other`, and
- the selected Contact record.

The result is a set (duplicates removed). Consequence: opening the wizard on a company Contact brings in all of its people and its "other" addresses, but **not** its `invoice` and `delivery` addresses, which are places rather than people.

### 3.3 Derivation of `user_ids`

Depends on `partner_ids`. For each wizard row, replace the whole line list with one newly created line per Contact in `partner_ids`, carrying:
- `partner_id` = the Contact,
- `email` = the Contact's electronic mail address (possibly empty).

Because the field is stored and writable, a later manual edit of a line's electronic mail address survives until `partner_ids` changes again.

### 3.4 Operations

#### 3.4.1 `action_open_wizard` (model level)

Creates an empty wizard row (which triggers the default expansion of `partner_ids` and the derivation of `user_ids`) and returns the dialog action for it. A server action is used rather than a plain window action because the lines must already exist in the database before the dialog is shown: the per-line buttons are disabled for lines that have no identifier.

#### 3.4.2 `_action_open_modal`

Returns a dialog action for this wizard row, titled `Portal Access Management`, in form mode, targeting a new dialog. Every per-line action returns this so that the dialog stays open and refreshes after each grant, revoke or re-invitation.

### 3.5 Model access

Only the Contact manager group may read, write and create this entity. Deletion is not granted to anybody. An employee who is not a Contact manager receives the standard model-access refusal both when creating the wizard and when reading any of its fields, including the fields of its lines.

---

## 4. Portal Access Wizard User (`portal.wizard.user`)

**Kind**: transient. One line of an invitation session.

### 4.1 Fields

| Canonical name | Type | Required | Default | Stored or derived | Deletion behavior of the relation | Meaning |
|---|---|---|---|---|---|---|
| `wizard_id` | many_to_one to Portal Access Wizard | yes | set by the parent | stored | cascade (deleting the wizard deletes the line) | The invitation session this line belongs to. |
| `partner_id` | many_to_one to Contact | yes | set by the parent | stored, read-only in the dialog | cascade (deleting the Contact deletes the line) | The Contact whose access is being managed. Labelled `Contact`. |
| `email` | text | no | the Contact's electronic mail address | stored, editable | not applicable | The address that will become the login of the external user and, when valid and different, is written back onto the Contact. Read-only in the dialog when the line represents an employee. |
| `user_id` | many_to_one to User | no | derived | derived from `partner_id`, evaluated with elevated rights, not stored | not applicable | The user currently linked to the Contact, archived ones included. |
| `login_date` | datetime | no | none | related to `user_id.login_date`, read-only | not applicable | Labelled `Latest Authentication`. Empty when the person has never signed in. |
| `is_portal` | boolean | no | false | derived from `user_id`, `user_id.active` and `user_id.group_ids` | not applicable | True when the linked user exists, is active and is an external user. |
| `is_internal` | boolean | no | false | derived from `user_id`, `user_id.active` and `user_id.group_ids` | not applicable | True when the linked user exists and is an employee, **whether active or archived**. |
| `email_state` | selection | no | `ok` | derived from `email`, evaluated with elevated rights | not applicable | The validity of the address: see section 4.2. |

`email_state` values, with their labels:

| Value | Label | Meaning |
|---|---|---|
| `ok` | `Valid` | The address normalizes to a non-empty address and no other user already uses it as a login. |
| `ko` | `Invalid` | The address is empty or does not normalize to a valid address. |
| `exist` | `Already Registered` | The address normalizes correctly but another user (a different user from the one linked to this line) already has it as a login. |

### 4.2 Derivation rules

**`user_id`** (depends on `partner_id`): for each line, search the Contact's users **including archived ones** and take the first one, or nothing when the Contact has no user. Evaluated with elevated rights so that a Contact manager who has no permission on Users can still see the state.

**`is_portal` and `is_internal`** (depend on `user_id`, `user_id.active`, `user_id.group_ids`): for each line, in this order:
1. If a user exists and that user is an employee: `is_internal = true`, `is_portal = false`. The archived state is deliberately ignored here, because a user who was an employee when archived must be reactivated from the user administration screens, not from this wizard.
2. Else, if a user exists, is active and is an external user: `is_internal = false`, `is_portal = true`.
3. Else: both false.

**`email_state`** (depends on `email`): for the whole set of lines at once:
1. Split the set into the lines whose address normalizes to something non-empty, called the *addressed lines*, and the rest. Assign `ko` to the rest.
2. Build the search condition by calling `_get_similar_users_domain` (section 4.3.9) with the addressed lines, and the field list by calling `_get_similar_users_fields` (section 4.3.10). Search users **including archived ones** with that condition, with elevated rights, and read exactly the projected fields of each match. Each row read is called a *candidate*.
3. For each addressed line, assign `exist` when at least one candidate satisfies `_is_portal_similar_than_user` (section 4.3.11) for that line; otherwise assign `ok`. The first satisfying candidate decides; the others are not examined.

The three operations used in steps 2 and 3 are **extension points**: a capability package may override any of them to widen or narrow the detection. The base behavior is in sections 4.3.9 to 4.3.11; the narrowing that the Website and Storefront capability package installs, which is mandatory for a correct `email_state` on an installation that serves several websites, is in section 4.6.

Normalization of an address means: trim, take the address part of a `Display Name <address@example.com>` form, lowercase the domain part, and return nothing when the result is not a syntactically valid single address.

### 4.3 Operations

#### 4.3.1 `action_grant_access` (the **Grant Access** button)

Applies to exactly one line.

1. Run the address uniqueness assertion (section 4.3.5). Raises when `email_state` is `ko` or `exist`.
2. If `is_portal` or `is_internal` is true, refuse with: `The partner "<Contact name>" already has the portal access.`
3. Write the normalized address back onto the Contact when it differs (section 4.3.6).
4. Take the linked user with elevated rights.
5. If there is no linked user: choose the company as the Contact's company when set, otherwise the acting user's current company; then create a user from the portal user template in that company (section 4.3.7).
6. Write on the user: `active = true`, add the portal group, remove the public group. The removal of the public group matters because a person who signed up through a public form starts in the public group.
7. Prepare the Contact for sign-up: set its `signup_type` to `signup`. This is what makes the invitation link produce a token.
8. Send the invitation message (section 4.3.8), evaluating the line with archived records excluded.
9. Return the reopen-dialog action.

#### 4.3.2 `action_revoke_access` (the **Revoke Access** button)

Applies to exactly one line.

1. If `is_portal` is false, refuse with: `The partner "<Contact name>" has no portal access or is internal.`
2. Write the normalized address back onto the Contact when it differs (section 4.3.6).
3. Clear the Contact's `signup_type`, which invalidates any outstanding invitation or reset token for that Contact.
4. If the linked user exists and is an external user, archive it (`active = false`). The user keeps the portal group: the public group is reserved for automated tasks and anonymous visitors and must not be reused for a revoked person.
5. Return the reopen-dialog action.

No message is sent on revocation.

#### 4.3.3 `action_invite_again` (the **Re-Invite** button)

Applies to exactly one line.

1. Run the address uniqueness assertion (section 4.3.5).
2. If `is_portal` is false, refuse with: `You should first grant the portal access to the partner "<Contact name>".`
3. Write the normalized address back onto the Contact when it differs.
4. Send the invitation message again (which regenerates the sign-up token, so the previous link stops working).
5. Return the reopen-dialog action.

#### 4.3.4 `action_refresh_modal`

Returns the reopen-dialog action without doing anything else. It exists because the three state icons in the line must be real, enabled buttons in order to show their tooltips; pressing one simply reloads the dialog.

#### 4.3.5 `_assert_user_email_uniqueness`

Applies to exactly one line.
- When `email_state` is `ko`, refuse with: `The contact "<Contact name>" does not have a valid email.`
- When `email_state` is `exist`, refuse with: `The contact "<Contact name>" has the same email as an existing user`

#### 4.3.6 `_update_partner_email`

Applies to exactly one line. When `email_state` is `ok` **and** the normalized form of the line's address differs from the normalized form of the Contact's address, write the normalized address onto the Contact.

#### 4.3.7 `_create_user`

Applies to exactly one line. Creates a user by duplicating the portal user template (whose identifier is held in a system parameter) with:
- `email` = the normalized address,
- `login` = the normalized address,
- `partner_id` = the line's Contact,
- `company` = the acting company (which step 5 of `action_grant_access` has already switched to the Contact's company when the Contact has one),
- `allowed_companies` = exactly that one company,
- `active` = true,
- the "do not send a password reset message" marker set, because this operation sends its own invitation.

Failures: when the template identifier does not resolve to an existing user, the creation fails with `Signup: invalid template user`; when the login is missing, with `Signup: no login given for new user`; when neither a Contact nor a name is given, with `Signup: no name or partner given for new user`; when the login is already taken, the underlying uniqueness violation is re-raised as a sign-up error.

#### 4.3.8 `_send_email`

Applies to exactly one line.
1. Resolve the shipped message template named `Settings: New Portal User Invite`. If it cannot be resolved, refuse with: `The template "Portal: new user" not found for sending email to the portal user.`
2. Take the language of the linked user and the Contact of the linked user.
3. Prepare that Contact for sign-up again (regenerating the token material).
4. Render and send the template to the linked user, forcing immediate delivery, with the database name, the language, the wizard's welcome message and the tracking medium `portalinvite` passed as rendering context.

The rendered message is described in [configuration.md](configuration.md).

#### 4.3.9 `_get_similar_users_domain(addressed_lines)`

A set-level operation. Input: the addressed lines, that is the lines whose typed address normalizes to a non-empty address. Output: a condition over User.

The base behavior returns exactly one term:

```
login IN { normalize(line.email) : line ∈ addressed_lines }
```

The search that uses this condition always runs with archived users included and with elevated rights.

This is an extension point. An override calls the inherited operation first, then appends further terms to the condition; it never removes the login term. Section 4.6 gives the shipped override.

#### 4.3.10 `_get_similar_users_fields()`

A set-level operation with no input. Output: the list of User fields that the candidate search reads.

The base behavior returns the two names `id` and `login`.

This is an extension point. An override calls the inherited operation first and appends the field names its own matching test needs. Every field named here is read with elevated rights, so an override must not add a field that would leak information the Contact manager may not see.

#### 4.3.11 `_is_portal_similar_than_user(candidate, line)`

Inputs: one candidate row as projected by `_get_similar_users_fields`, and one addressed line. Output: boolean.

The base behavior returns true when **both** of the following hold:
- `candidate.login = normalize(line.email)`,
- `candidate.identifier ≠ line.user.identifier` (which is satisfied when the line has no linked user, because the comparison is then against "no identifier").

This is an extension point. An override calls the inherited operation first; when the inherited result is false the override returns false, and when it is true the override may still return false because of its own additional test. An override therefore never turns an `ok` line into an `exist` line that the base behavior would not have flagged; it may only turn an `exist` line back into `ok`. Section 4.6 gives the shipped override.

### 4.4 Model access

Same as the parent wizard: read, write and create for the Contact manager group only; no deletion for anybody.

### 4.5 Line state table

| From state (`is_portal`, `is_internal`, `email_state`) | Operation | Guard | To state | Side effects |
|---|---|---|---|---|
| (false, false, `ok`), no linked user | `action_grant_access` | address valid and unused | (true, false, `ok`) | A user is created from the template in the Contact's company; portal group added; public group removed; Contact's `signup_type` set to `signup`; invitation message sent. |
| (false, false, `ok`), linked user exists and is public or archived-external | `action_grant_access` | address valid and unused | (true, false, `ok`) | Existing user reactivated; portal group added; public group removed; `signup_type` set; invitation message sent. |
| (true, false, any) | `action_grant_access` | fails | unchanged | Refusal `The partner "<name>" already has the portal access.` |
| (false, true, any) | `action_grant_access` | fails | unchanged | Same refusal as above; no message sent. |
| (any, any, `ko`) | `action_grant_access` | fails | unchanged | Refusal `The contact "<name>" does not have a valid email.` |
| (any, any, `exist`) | `action_grant_access` | fails | unchanged | Refusal `The contact "<name>" has the same email as an existing user` |
| (true, false, any) | `action_revoke_access` | passes | (false, false, unchanged) | User archived; `signup_type` cleared; portal group kept; no message. |
| (false, any, any) | `action_revoke_access` | fails | unchanged | Refusal `The partner "<name>" has no portal access or is internal.` |
| (false, true, any) | `action_revoke_access` | fails | unchanged | Same refusal. |
| (true, false, `ok`) | `action_invite_again` | passes | unchanged | New sign-up token; invitation message sent again; the previous link stops working. |
| (false, any, `ok`) | `action_invite_again` | fails | unchanged | Refusal `You should first grant the portal access to the partner "<name>".` |
| any | `action_refresh_modal` | always passes | unchanged | Dialog reopened. |

### 4.6 Multi-website narrowing of the address-state detection

**Installed by**: the Website and Storefront capability package (see [Website and Storefront](../website-and-storefront/README.md)). **Applies to**: the three extension points of sections 4.3.9 to 4.3.11. **Reason**: when the platform serves several public websites, a login is unique per website rather than globally, so two people may legitimately hold the same address on two different websites. Without this narrowing, `email_state` is computed wrongly on such an installation: lines would be marked `exist` and refused although the address is free on the website the person belongs to.

**Prerequisite field**: the [Website and Storefront](../website-and-storefront/README.md) domain adds a `website_id` link to Contact (through its published-on-website contract) and a `website_id` link to User, the latter related to the Contact's one and stored, and it makes the pair (`login`, `website_id`) the uniqueness rule of a User instead of `login` alone. The value is empty when the record belongs to no particular website, which means it belongs to all of them.

**Override of `_get_similar_users_domain(addressed_lines)`**

1. Take the inherited condition (the login membership term).
2. Build the website list `W`, starting empty, by walking the addressed lines in order:
   - let `w` be the website of the line's Contact;
   - when `w` is set and `w` is not already in `W`, append `w`;
   - when `w` is empty and the value "no website" is not already in `W`, append two values: "no website", and the identifier of the website that is serving the current request.
3. Return the inherited condition with the added term `website_id IN W`.

When `addressed_lines` is empty, `W` is empty and the added term matches nothing, which is consistent: there is no line to decide.

**Override of `_get_similar_users_fields()`**

Return the inherited list (`id`, `login`) with `website_id` appended.

**Override of `_is_portal_similar_than_user(candidate, line)`**

1. Evaluate the inherited test. When it is false, return false.
2. When the line's Contact **has** a website (`website_id` is set): return true only when the candidate has a website **and** that website is the same one. A candidate that belongs to no website does not block a Contact that belongs to one.
3. When the line's Contact has **no** website (so the person will be able to sign in on any website): return true when the candidate has no website either, **or** when the candidate's website is the website serving the current request. The reason for the second branch is that such a Contact is redirected to the current website when the account is created, so a clash on that website is a real clash.

**Combined effect table** (the inherited test has already passed, that is, some user already holds the normalized address as a login):

| Website of the line's Contact | Website of the candidate user | Resulting `email_state` |
|---|---|---|
| none | none | `exist` |
| none | the website serving the current request | `exist` |
| none | another website | `ok` |
| website A | website A | `exist` |
| website A | website B | `ok` |
| website A | none | `ok` |

The filter and the test agree: a candidate that the filter would not have returned can never satisfy the test, so the two may be reasoned about independently. See [calculations.md](calculations.md) section 7 for the worked example, and `PORT-RULE-175`.

---

## 5. Fields and behavior added to Discussion Thread Mixin (`mail.thread`)

Owned by [Messaging and Activities](../messaging-and-activities/README.md). The Customer Portal adds the following.

### 5.1 Field

| Canonical name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `website_message_ids` | one_to_many to Message, keyed on the message's related record identifier | no | empty | derived list, not a stored column | The messages of this record that may appear on a public page. The list is restricted by a filter, is not subject to the message record rules when read through the list (it bypasses the search-level access filter and relies on the caller having validated thread access), and is labelled `Website Messages` with the help text `Website communication history`. |

The filter is:

```
model = <this model's transport name>
AND message_type IN ("comment", "email", "email_outgoing", "auto_comment", "out_of_office")
```

Message kinds excluded by that filter are the purely technical ones (notifications, user notifications and the like), which must never appear on a public page.

### 5.2 The external-posting token field name

Every model that adopts the Discussion Thread Mixin declares which of its fields holds the secret used to authorize external posting. The default is `access_token`, that is, the field contributed by the Portal Access Mixin. A model may point it at another field; the only use of this indirection in the shipped behavior is the project sharing screen, which reads the project's token through the same name.

### 5.3 `_sign_token(contact_identifier)`

Applies to exactly one record. Output: a hexadecimal digest.

1. If the declared token field name is not a field of this model, refuse with: `Model <model name> does not support token signature, as it does not have <field name> field.`
2. Read the platform's database secret from the system parameters.
3. Build the tuple `(database name, value of the token field, contact identifier)` and render it as its canonical textual representation.
4. Return the keyed-hash message authentication code of that text, using the database secret as the key and the secure hash algorithm producing a 256-bit digest as the hash function, encoded as lowercase hexadecimal.

Properties that a replacement must preserve:
- The digest changes when the token changes, so clearing or rotating a document's token invalidates every signed identity previously issued for it.
- The digest is bound to the database, so a link cannot be replayed against another installation.
- Comparison of a supplied digest with the computed one must be done in constant time with respect to the number of matching leading characters, to avoid leaking the digest through response timing.

### 5.4 `_portal_get_parent_hash_token(contact_identifier)`

Returns nothing by default. A model whose records are shared indirectly through a parent record overrides it and returns the parent's signed identity. The shipped override is Task, which returns the signed identity of its Project: a person who received a link to a shared project may therefore post on any task of that project without a task-specific link.

### 5.5 `_get_allowed_access_params`

The set of query-string parameter names that the thread-resolution operations accept is extended with `hash` (the signed recipient identity), `pid` (the recipient Contact identifier) and `token` (the document security token). Any other parameter name passed to thread resolution is logged as invalid and ignored.

### 5.6 `_get_thread_with_access(thread_identifier, hash, pid, token, mode)`

1. Try the inherited resolution first: browse the record and return it when it exists and the acting user has the requested permission on it with the company restriction lifted.
2. When that fails, browse the record with elevated rights and return it if **either**:
   - `validate_thread_with_hash_pid(record, hash, pid)` succeeds (section 5.7), **or**
   - `validate_thread_with_token(record, token)` succeeds (section 5.8).
3. Otherwise return nothing.

### 5.7 `validate_thread_with_hash_pid(record, hash, pid)`

1. Return false when either `hash` or `pid` is missing.
2. Convert `pid` to an integer.
3. Return true when `hash` equals `record._sign_token(pid)` under a constant-time comparison.
4. Otherwise compute `record._portal_get_parent_hash_token(pid)`; return true when it is non-empty and equals `hash` under a constant-time comparison.
5. Otherwise return false.

### 5.8 `validate_thread_with_token(record, token)`

Return true when `token` is non-empty and equals, under a constant-time comparison, the value of the field named by the model's external-posting token field name.

### 5.9 `get_portal_partner(record, hash, pid, token)`

1. If `validate_thread_with_hash_pid(record, hash, pid)` succeeds, return the Contact whose identifier is `pid`, read with elevated rights.
2. Else if `validate_thread_with_token(record, token)` succeeds, return the first Contact among the record's mail recipients, if any.
3. Else return nothing.

This resolution is what gives an anonymous visitor an author identity: a message posted from a page opened with a signed link is attributed to the Contact named in the link, and a message posted from a page opened with a bare token is attributed to the document's own customer.

### 5.10 Notification recipient groups

When a message is sent on a record whose model adopts **both** the Discussion Thread Mixin and the Portal Access Mixin, the recipient grouping used to build the notification message is changed as follows.

1. Compute the inherited groups.
2. Stop here when the record set is empty or when the model does not adopt the Portal Access Mixin.
3. Resolve the record's customer Contact (the first mail recipient).
4. When there is a customer:
   - ensure the record has a token (created with elevated rights if needed, justified by the fact that a person with read access who posts must be able to give the other recipients a working link);
   - build the action link for the generic redirection endpoint with the token, the customer's identifier, the signed identity for that customer and the customer's sign-up parameters;
   - prepend a recipient group named `portal_customer` that matches exactly that Contact, is active, and shows an access button pointing at that link.
5. Whatever happens in step 4, find the inherited `portal` recipient group and force it to be active and to show its access button, so that every other external recipient also receives a direct link (their own permissions, or the token in the link, then decide what they may see).

---

## 6. Behavior added to Message (`mail.message`)

Owned by [Messaging and Activities](../messaging-and-activities/README.md). No field is added; three behaviors are.

### 6.1 Authorship recognition for a signed recipient

The derived flag that tells the front end "the current reader wrote this message" is extended: after the inherited computation, if the rendering context carries a portal Contact and a portal thread, then for every message whose author equals that Contact and whose related record is exactly that thread, the flag is forced to true. This is what lets an anonymous visitor who opened a signed link edit or delete their own messages.

### 6.2 `portal_message_format(options)` (public)

1. Check that the acting user may read the messages in the set (raising the standard read refusal otherwise).
2. Compute the default property set (section 6.3) and delegate to the private projection (section 6.4).

The access check is deliberately separate from the projection, because the private projection is also called from endpoints that have already validated thread access and then read with elevated rights.

### 6.3 Default property set

| Projected property | Meaning |
|---|---|
| `attachments` | The list of attachments of the message, each formatted as described in section 6.5. |
| `author_avatar_web_address` | The address at which the author's picture may be fetched, chosen as described in section 6.4. |
| `author` | The author Contact, reduced to its identifier and its name, or false. |
| `author_guest` | The anonymous guest identity that authored the message, when there is one. |
| `body` | The message body, transported as rich text. |
| `date` | The publication moment. |
| `id` | The message identifier. |
| `is_internal` | Whether the message is restricted to employees. |
| `is_message_subtype_note` | Whether the message subtype is the shipped private-note subtype. |
| `message_type` | The kind of message. |
| `model` | The transport name of the entity of the discussed record. |
| `published_date_str` | The publication moment formatted for the reader. |
| `res_id` | The identifier of the discussed record. |
| `starred` | Whether the reader has starred the message. |
| `subtype` | The message subtype. |

When the rating capability is installed and the caller passes the option "include rating", the set additionally contains the rating link and the rating value.

### 6.4 `_portal_message_format(property_names, options)`

1. **Attachments.** If `attachment_ids` is in the property set, remove it from the set and, with elevated rights, read for every attachment of every message in the set: checksum, thumbnail availability, identifier, media type, name, related record identifier and related record model. Then build, per message, the ordered list of its formatted attachments (section 6.5).
2. **Plain fields.** Keep only the property names that are real fields and read them in the platform's read-format representation.
3. For every message, in order:
   - Wrap the body so that it is transported as rich text rather than as escaped plain text.
   - Attach the formatted attachment list when attachments were requested.
   - Compute the author avatar web address:
     - when the options carry a token: `/mail/avatar/message/<message identifier>/author_avatar/50x50?access_token=<token>`;
     - else when the options carry a signed identity and a Contact identifier: `/mail/avatar/message/<message identifier>/author_avatar/50x50?_hash=<hash>&pid=<pid>`;
     - else: `/web/image/message/<message identifier>/author_avatar/50x50`.
   - Compute the "is a private note" flag as `subtype = <the shipped "note" subtype>`; a message with no subtype yields false.
   - Compute the formatted publication date from the message date in the reader's language and time zone; empty when the message has no date.
   - Build the reaction groups: for each distinct reaction content, the content, the number of reactions, the list of reacting guests (identifier and name) and the list of reacting Contacts (identifier and name), plus the message identifier.
   - Replace the author with either the pair (identifier, name) of the author Contact, or false when the message has no author.
   - Add a thread descriptor: whether the related model adopts the Discussion Thread Mixin, the related record identifier and the related record model.
4. **Linked messages.** For every message linked from the projected messages but not itself in the set, add a lightweight entry with its identifier, related model and related record identifier, plus a thread descriptor carrying the display name of the linked record (read with elevated rights) or false when the record cannot be resolved.

### 6.5 `_portal_message_format_attachments(attachment_values)`

1. Copy the attachment name into a `filename` property.
2. When the requesting browser is Safari and the media type contains `video`, replace the media type with `application/octet-stream`. This is a workaround for that browser's handling of ranged video requests; a replacement should keep it, because without it the video download fails in that browser.
3. Add the attachment's raw-content access token and its thumbnail access token.
4. When the reader is recognized as the author of the message (section 6.1), also add the attachment's ownership token, which is what permits deletion.

### 6.6 Rating additions to the projection (rating capability installed)

When `rating` is part of the property set:
1. With elevated rights, read every Rating linked to the projected messages: identifier, publisher comment, publisher, publication moment and message.
2. For each message, attach the formatted rating of that message (or an empty structure when it has none).
3. For each message, resolve its related record; when that record exposes rating statistics, attach the statistics computed with elevated rights.

`_portal_message_format_rating(rating_values)` produces:
- `publisher_avatar`: `/web/image/contact/<publisher identifier>/avatar_128/50x50` when there is a publisher, empty text otherwise;
- `publisher_comment`: the comment, or empty text when absent;
- `publisher_datetime`: the publication moment formatted in the reader's language and time zone;
- `publisher_id`: the publisher identifier or false;
- `publisher_name`: the publisher name or empty text.

---

## 7. Fields added to Rating (`rating.rating`)

Owned by [Messaging and Activities](../messaging-and-activities/README.md).

| Canonical name | Type | Required | Default | Stored or derived | Copied on duplicate | Deletion behavior | Meaning |
|---|---|---|---|---|---|---|---|
| `publisher_comment` | long_text | no | empty | stored | yes | not applicable | The operating company's public reply to this rating, shown under the rating on the public page. Labelled `Publisher comment`. |
| `publisher_id` | many_to_one to Contact | no | empty | stored, read-only | yes | set null (deleting the Contact leaves the comment without a publisher) | Who wrote the reply. Labelled `Commented by`. Indexed with a partial index that skips empty values. |
| `publisher_datetime` | datetime | no | empty | stored, read-only | yes | not applicable | When the reply was written. Labelled `Commented on`. |

### 7.1 Automatic stamping

Both on creation and on modification, before the values reach storage:

1. If the incoming values contain a non-empty `publisher_comment`:
   - run the write guard (section 7.2);
   - when `publisher_datetime` is not supplied, set it to the current moment;
   - when `publisher_id` is not supplied, set it to the Contact of the acting user.
2. On creation only, after the rows exist, run the write guard again for the rows that ended up with a non-empty comment.

The two stamped fields are declared read-only precisely because they are meant to behave like a tracking stamp rather than like data a person types.

### 7.2 Write guard

1. Resolve the website editor group. When that group exists **and** the acting user belongs to it, allow and stop.
2. Otherwise, group the ratings by the model of the record they rate; for each group, check that the acting user has write permission on those records. If any check fails, refuse with:

```
Updating rating comment require write access on related record
```

The refusal is raised as a permission refusal, chained from the underlying one.

---

## 8. Behavior added to Contact (`res.partner`)

Owned by [Contacts and Organizations](../contacts-and-organizations/README.md). No field is added; six predicates and helpers are.

### 8.1 `_get_frontend_writable_fields` (model level)

The closed set of Contact fields that a person may write through a front-end form:

| Identifier | Full name | Form input name | Note |
|---|---|---|---|
| `name` | Name | `name` | |
| `phone` | Telephone | `phone` | |
| `email` | Electronic mail address | `email` | |
| `street` | Street | `street` | |
| `street2` | Street second line | `street2` | Second address line. |
| `city` | City | `city` | |
| `state_id` | Country subdivision | `state_id` | |
| `country_id` | Country | `country_id` | |
| `zip` | Postal code | `zip` | |
| `zip` | Postal code | `zipcode` | Alternative input name accepted from the form; mapped onto the postal code when the primary name is absent or empty. Note that `zipcode` is named in the writable set although it is not a field of Contact, which is what makes the fall-back mapping possible. |
| `vat` | Tax identification number | `vat` | The reproduced identifier `vat` abbreviates value-added tax; the value carried is the tax identification number of the Contact, whatever the country calls it. |
| `company_name` | Company name | `company_name` | |

Any other form field is not written onto the Contact; it is carried aside as "extra form data" and offered to the extension points described in [workflows.md](workflows.md).

Other capability packages extend this set; for example an electronic-invoicing capability adds the preferred sending method and the preferred interchange format to the account form.

### 8.2 `_can_edit_country`

Applies to exactly one Contact. Returns true in the base behavior. Extended by other domains:

| Extending domain | Additional condition |
|---|---|
| Accounts Receivable | No posted customer invoice or customer credit note exists whose Contact is exactly this Contact. |
| Sales | No sales order in state "sent" or "confirmed" exists whose order Contact **or** invoicing Contact is exactly this Contact. |

The predicate is therefore "no document has been issued under this country yet".

### 8.3 `can_edit_vat`

Applies to exactly one Contact. Base behavior: true only when the Contact has **no** parent. Reason: the tax identification number is a commercial field synchronized from the commercial entity down to its children, so only the commercial entity may own it. Extended by other domains:

| Extending domain | Additional condition |
|---|---|
| Accounts Receivable | No posted customer invoice or customer credit note exists for any Contact under this Contact's commercial entity. |
| Sales | No sales order in state "sent" or "confirmed" exists for any Contact under this Contact's commercial entity. |

### 8.4 `_can_be_edited_by_current_customer`

Applies to exactly one Contact. Output: boolean.

1. Resolve the current customer (section 8.5).
2. Return true when this Contact **is** the current customer.
3. Otherwise, search the Contacts that are descendants of the current customer's commercial entity **and** whose address kind is `invoice`, `delivery` or `other`, and return whether this Contact is among them.

Consequence: a person may edit their own Contact and any address of their company, but not the personal Contact of a colleague (kind `contact`), and not a Contact outside their company tree.

### 8.5 `_get_current_partner` (model level)

Returns nothing when the acting user is the anonymous public user; otherwise returns the Contact of the acting user. Extensions may resolve it differently (for example from an anonymous shopping session).

### 8.6 `_get_delivery_address_domain`

Applies to a set of Contacts. Returns the condition:

```
id IS DESCENDANT OF <these Contacts>
AND ( type IN ("delivery", "other") OR id IN <these Contacts> )
```

That is: the commercial entity itself plus its delivery and "other" addresses.

---

## 9. Field added to Configuration Settings (`res.config.settings`)

Owned by [Identity and Access](../identity-and-access/README.md); the settings mechanism itself is described in [the security model](../../overview/security-model.md).

| Canonical name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `portal_allow_api_keys` | boolean | no | false | derived from a system parameter, with a write-back rule | Labelled `Customer API Keys` ("Customer Application Programming Interface Keys"), shown next to the checkbox caption `Customers can generate API keys`, inside the developer-only block of the general settings page. |

- **Read rule**: the value of the system parameter `portal.allow_api_keys` (the "customers may create application keys" switch), interpreted as a boolean.
- **Write rule**: writes the boolean into that system parameter with elevated rights.
- The settings page loader also puts the same value into the page payload on every load, so that the checkbox reflects the stored parameter even before the derivation runs.

---

## 10. Field added to View Definition (`ir.ui.view`)

Owned by the platform foundation; see [views and actions](../../overview/views-and-actions.md) and [inheritance and extension](../../overview/inheritance-and-extension.md).

| Canonical name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|
| `customize_show` | boolean | no | false | yes | Labelled `Show As Optional Inherit`. Marks an inheriting layout fragment as an optional variant that a website editor may switch on or off from the editor's customization panel, instead of a fragment that is always applied. |

Layout fragments shipped with this flag set to true: the pictogram variant of the portal home cards, and the per-domain home-page card blocks contributed by Sales, Accounts Receivable, Purchasing, Projects and Tasks, Timesheets, Manufacturing subcontracting and the payment capability.

---

## 11. Behavior added to Request Routing (`ir.http`)

Owned by the platform foundation; see [the request lifecycle](../../runtime/request-lifecycle.md) and [translation](../../runtime/translation.md).

1. **Front-end translation catalogues.** The list of capability packages whose translations are shipped to the front end is extended with the portal package, and, when the rating bridge is installed, with the portal rating package. Two further bridges add the password-policy package (so that the strength meter is translated) when that bridge is installed.
2. **Front-end session payload.** When the request carries a signed-in session, the payload is extended with whether guided tours are enabled for that user and, when they are, with the current tour to run. This is what allows the portal onboarding tour to run for an external user.

---

## 12. Behavior added to Template Engine (`ir.qweb`)

Owned by the platform foundation; see [views and actions](../../overview/views-and-actions.md).

When a front-end rendering environment is prepared, the value set is extended with:
- a helper that tells whether a rich-text value is empty (used by the layout to hide empty blocks),
- a lazily evaluated list of the languages published on the front end (used by the language selector; lazy so that the query is not run for pages that do not show the selector),
- and every key of the rendering environment's context that is not already present in the value set, so that context keys set by a controller reach the template.

---

## 13. Behavior added to User Application Key Creation Wizard (`res.users.apikeys.description`)

Owned by [Identity and Access](../identity-and-access/README.md).

The permission check that guards key creation is relaxed:

1. Run the inherited check. If it passes, allow.
2. If it refuses:
   - read the system parameter `portal.allow_api_keys` with elevated rights;
   - when it is set and the acting user is an external user, allow;
   - when it is set and the acting user is neither an employee nor an external user, refuse with: `Only internal and portal users can create API keys` (reproduced message; the three letters abbreviate "application programming interface");
   - when it is not set, re-raise the inherited refusal unchanged.

---

## 14. Behavior added to User (`res.users`)

Owned by [Identity and Access](../identity-and-access/README.md).

`get_totp_invite_url` (the second-factor invitation web address): when the target user is not an employee, return `/my/security`; otherwise return the inherited back-office address. This makes the "invite to enable a second factor" action produce a link an external person can actually open.

---

## 15. Document models that adopt the Portal Access Mixin

Each adopting model overrides `access_url`. The table lists every adoption shipped with the system and the rule each one applies. The models themselves are specified in their own domains.

| Entity | Owning domain | Portal web address rule | Notes |
|---|---|---|---|
| Sales Order (`sale.order`) | [Sales](../sales/README.md) | `/my/orders/<identifier>` | Serves both quotations and confirmed orders; the page decides which actions to offer from the order state. |
| Journal Entry (`account.move`) | [Accounts Receivable](../accounts-receivable/README.md) and [Accounts Payable](../accounts-payable/README.md) | `/my/invoices/<identifier>`, applied only to rows that are invoices (customer invoice, customer credit note, customer receipt, vendor bill, vendor refund, vendor receipt) | Non-invoice journal entries keep the mixin's default address `#` and are therefore never shareable. |
| Journal (`account.journal`) | [General Ledger](../general-ledger/README.md) | inherited default | Adopts the mixin without overriding the address rule, so it has a security token and a discussion thread that can be shared by link, but no portal page of its own. |
| Purchase Order (`purchase.order`) | [Purchasing](../purchasing/README.md) | `/my/purchase/<identifier>` | Serves both requests for quotation and confirmed purchase orders. |
| Project (`project.project`) | [Projects and Tasks](../projects-and-tasks/README.md) | `/my/projects/<identifier>` | Also acts as the logical parent for its tasks' signed identities. |
| Task (`project.task`) | [Projects and Tasks](../projects-and-tasks/README.md) | `/my/tasks/<identifier>` | Its logical-parent signed identity is the project's. |
| Point of Sale Order (`pos.order`) | [Point of Sale](../point-of-sale/README.md) | inherited default | Adopts the mixin for the token, used by the digital receipt link. |
| Electronic Waybill (`l10n.in.ewaybill`) | [Fiscal Localizations](../fiscal-localizations/README.md) | inherited default | Country-specific document; adopts the mixin for the token. |

---

## 16. Identity, uniqueness, ordering and company scoping

| Entity | Default ordering | Display name rule | Uniqueness | Company scoping | Archiving |
|---|---|---|---|---|---|
| Portal Access Mixin | not applicable (abstract) | not applicable | The `access_token` column carries no uniqueness constraint; collisions are prevented by the randomness of a version-4 universally unique identifier (**industry-standard completion**: a replacement should add a unique index on the column of each adopting model, since a collision would grant a reader access to the wrong document). | inherited from the adopting model | inherited from the adopting model |
| Portal Share Wizard | by identifier ascending | the identifier | none | none; the wizard reads the shared document with the acting user's own permissions, so company rules apply through the document | not archivable |
| Portal Access Wizard | by identifier ascending | the identifier | none | none; the company used to create a new user is taken per line from the line's Contact | not archivable |
| Portal Access Wizard User | by identifier ascending | the Contact's display name | none | none | not archivable |

---

## 17. Lifecycle of an external user, seen from this domain

| Stage | Trigger | Resulting state |
|---|---|---|
| Prospect | A Contact exists with or without an electronic mail address. | No user. `is_portal` false, `is_internal` false. |
| Invited | Contact manager presses **Grant Access**. | User exists, active, in the portal group, not in the public group; the Contact's `signup_type` is `signup`; an invitation message with an activation link has been sent. The person still has no password. |
| Active | The person opens the activation link and sets a password. | The sign-up token is consumed; the Contact's `signup_type` is cleared by the sign-up flow; the user's latest authentication moment starts to be recorded. |
| Re-invited | Contact manager presses **Re-Invite**. | A new sign-up token replaces the previous one; the previous activation link stops working; a new invitation message is sent. |
| Revoked | Contact manager presses **Revoke Access**. | User archived, still in the portal group; `signup_type` cleared; existing sessions stop working at the next request because the user is inactive. |
| Restored | Contact manager presses **Grant Access** on a revoked line. | User reactivated, portal group re-asserted, public group removed, new sign-up token, new invitation message. |
| Self-deleted | The person confirms the account deletion dialog. | Login replaced by a reserved deleted-account login, password emptied, application keys removed, user and Contact archived, a note logged on the Contact, and a deletion request queued. |
| Removed | The daily scheduled job processes the queue. | The user row is deleted; the Contact row is deleted too when nothing references it, and is left archived otherwise. |
