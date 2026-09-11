# Identity and Access — Interfaces

This file specifies everything the domain exposes: the navigation an administrator or a user sees,
the screens and what each shows, the named remote operations and their inputs and outputs, the
addressable paths with their method and authentication mode, the messages the domain sends, and the
external services it talks to.

> **Note on reproduced texts.** Message texts, button labels and screen titles are reproduced
> **verbatim**, exactly as the system emits them, because external tests and user documentation
> depend on them. Where such a reproduced text contains an abbreviation, the abbreviation belongs to
> the text and is not this document's own prose. The abbreviations that occur are: *API* for
> application programming interface; *2FA* for two-factor authentication; *LDAP* for the central
> directory protocol; *DN* for distinguished name; *OAuth* and *UID* for the delegated sign-in
> protocol and its subject identifier; *JSON* for the structured-literal notation; *HTTP* for the
> transport protocol; *ID* for identifier. Outside such reproduced texts, every term is written in
> full.

Reproduced identifiers (paths, transport names, storage names, parameter keys) are given in code
font, each accompanied by its full name in words on first use.

---

## 1. Navigation

### 1.1 The administration area

| Menu | Parent | Visible to | Opens |
|---|---|---|---|
| Settings | top level | holders of *Role / Administrator* (the whole area) | the administration area |
| General Settings | Settings | *Role / Administrator* | the settings screen, reachable at the path fragment `settings` |
| Users & Companies | Settings | the area's own restriction | a grouping menu |
| Users | Users & Companies (first) | — | the account list |
| Companies | Users & Companies | — | the company list |
| Groups | Users & Companies | *Technical Features* | the group list |
| Privileges | Users & Companies | *Technical Features* | the privilege list |
| Technical | Settings | *Technical Features* | a grouping menu |
| Security | Technical | — | a grouping menu |
| Record Rules | Security | — | the record-rule list |
| Access Rights | Security | — | the access-right list |
| Parameters | Technical | — | a grouping menu |
| System Parameters | Parameters | — | the parameter list |
| Actions → User-defined Defaults | Technical → Actions | — | the default-value list |
| Sequences & Identifiers → External Identifiers | Technical | *Technical Features* | the external-identifier list |
| Privacy | Technical | — | a grouping menu |
| Privacy Logs | Privacy | — | the privacy-log list, with creation disabled |

### 1.2 The data-cleaning area

| Menu | Parent | Opens |
|---|---|---|
| Data Cleaning | top level (sequence 250) | the area |
| Recycle Records | Data Cleaning (sequence 10) | the outstanding candidate list |
| Configuration | Data Cleaning (sequence 100) | a grouping menu |
| Configuration → Rules | Configuration (sequence 1) | a grouping menu |
| Configuration → Rules → Recycle Records | Rules (sequence 10) | the recycling-rule list |

### 1.3 The user menu

The user menu in the top bar carries the preferences entry (*My Profile*) and the sign-out entry.

### 1.4 The customer-facing area

Reached at `/my` (the customer dashboard). The dashboard shows a card per document kind with a
count; the domain itself contributes the account card, the addresses card and the security card.

---

## 2. Screens

### 2.1 The account form (full)

Shown to administrators. Structure:

**Buttons along the top of the record**, each visible only to holders of *Technical Features*:

| Button | Shows | Opens |
|---|---|---|
| Groups | the group count | the closure, read-only, creation and deletion disabled |
| Access Rights | the access-right count | every access-right row attached to the closure, read-only |
| Record Rules | the record-rule count | every record rule attached to the closure, read-only |

**Ribbons**: *External user* when the share flag is set; *Archived* when the account is archived.

**Banner**: when the account is archived but its contact is not, a notice *The contact linked to
this user is still active* with a link to archive the contact.

**Header block**: the image; the name (placeholder *e.g. John Doe*, required); the login, preceded
by an envelope icon when it equals the electronic mail address and by a key icon otherwise; the
electronic mail address, shown only when it differs from the login; the telephone number.

**Related contact**: shown read-only to holders of *Technical Features*, and only on a saved record.

**Tab — Access Rights**

- A *Roles* group, hidden when the account is external and there is at most one company. It holds:
  the role as a horizontal pair of radio choices (hidden for external accounts); the permitted
  companies as removable tags, coloured, with creation disabled, shown only when more than one
  company exists; the default company, shown under the same condition, with the user-preference
  marker switched off so that every company is listed.
- The group editor: for holders of *Technical Features*, always; for everyone else, only when the
  account is internal. It renders the hierarchy snapshot as one selector per privilege, grouped by
  application category, each offering the privilege's groups in the order of section 14.8 of
  [calculations.md](calculations.md) plus the privilege's "no access" placeholder.

**Tab — Preferences**: the language (required) with a shortcut to install a language for holders of
*Role / Administrator*; the electronic mail signature.

**Tab — Calendar**: the time zone, with a mismatch indicator fed by the time-zone offset; hidden for
external accounts.

**Tab — Security** (only on a saved record):

| Block | Content |
|---|---|
| Change Password | *Update if compromised.* For one's own account, the self-service dialogue; for anyone else, the administrator dialogue. |
| Application Keys | Only on one's own account. *Connect external services.* A card list of the account's keys and an **Add API Key** button. |
| Devices | Only on one's own account. *Check if they are yours.* A card list of the account's devices and a **Log out from all devices** button. |
| Two-factor authentication | Contributed by the second-factor package: the enabled indicator, the enable or disable button, and the trusted-device list. |
| Passkeys | Contributed by the passkey package: the credential list with rename and delete, and a create button. |

### 2.2 The account list

Columns: avatar (24 by 24 points, rounded), name (read-only), login (read-only, shown by default),
language (hidden by default), last sign-in (read-only, hidden by default), role as a badge.
Multi-record editing is enabled.

### 2.3 The account preferences form

A reduced form used by the *My Profile* entry, restricted to the self-service fields. Its
post-processing removes the group markers from the fields that are in the readable-by-oneself list,
so those fields render even for a user with no administration group.

### 2.4 The group form

Shows the name, the privilege, the sequence, the comment, the share flag, the maximum
application-key duration, the directly implied groups, the transitively implied groups, the
directly implying groups, the users, the access-right rows, the record rules, the menus and the
views. A button *Users and implied users of …* opens every account that reaches the group.

A dedicated narrow form is used for the *Default access for new users* group, showing only its
implied groups.

### 2.5 The company form

Shows the name, the parent (read-only after creation), the branches, the contact-derived address
block, the currency (read-only on a branch, because it is delegated to the root), the tax
identifier, the company registry, the logo, the document presentation block (tagline, footer,
details, template, paper format, font, colours, background) and the directory configurations for
holders of *Role / Administrator*. A button opens the branches.

Every root-delegated field is rendered read-only when the record has a parent.

### 2.6 The record-rule form

Shows the name, the entity, the filter, the groups, the four operation flags and the derived global
flag. The group picker is restricted to non-global rules.

### 2.7 The access-right list

Shows the name, the entity, the group and the four flags; it is the natural place to audit who may
do what.

### 2.8 The settings screen

One page per application, each a block of the fields described in section 2.8 of
[configuration.md](configuration.md). The domain's own blocks are:

| Block | Fields |
|---|---|
| Users | the invitation scope; the password-reset toggle; the external-user template; the minimum password length; the second-factor enforcement toggle and policy; the customer application-key toggle; a button to edit the default groups of new users |
| Companies | the selected company, its name, its information block, a button to open it; the multi-currency toggle |
| Integrations | the external-provider toggle and its client identifier and return address; the directory package toggle and the directory configuration list |
| Statistics | the number of companies, the number of active internal accounts, the number of installed languages |

### 2.9 The recycling screens

- **Rule list and form**: the entity, the name, the mode, the action, the filter, the time field and
  its delta and unit, the archived-inclusion flag, the notification recipients and cadence, the
  outstanding count, and buttons to run the rule and to open its candidates.
- **Candidate list**: grouped by rule in a side panel, showing the record name, the entity and the
  company, with per-row and bulk **Validate** and **Discard**.

### 2.10 The privacy screens

- **Search form**: the name and the electronic mail address, and a **Look up** button.
- **Result list**: grouped by entity by default, creation disabled; each row shows the record name,
  the entity, a link to the record when readable, the archive toggle, the deletion button and the
  execution detail.
- **Log form**: the date, the handler, the masked name and address, the found records, the execution
  details and the free note. The list is opened with creation disabled.

### 2.11 The customer-facing pages

| Page | Path | Content |
|---|---|---|
| Dashboard | `/my`, `/my/home` | One card per document kind with a count. |
| Account details | `/my/account` | The contact's own details, editable within the permitted subset. |
| Addresses | `/my/addresses` | The contact's addresses, with creation, editing and archiving. |
| One address | `/my/address` | The address form. |
| Security | `/my/security` | The password-change block; the application-key block, shown only when `portal.allow_api_keys` (the customer application-key parameter) is set; the second-factor block and the passkey block when those packages are installed; the account-deletion block. Served with framing restricted to the same origin. |

---

## 3. Named remote operations

Each entry gives the entity, the operation, its inputs and its output. All of them are subject to
the ordinary access checks unless stated.

### 3.1 On the User (`res.users`)

| Operation | Inputs | Output | Notes |
|---|---|---|---|
| `authenticate` | a credential; an environment description | the authentication result | Establishes nothing by itself; the session layer calls it. |
| `change_password` | the old password; the new password | true | Raises on a wrong old password or an empty new one. |
| `has_group` | one fully-qualified group reference | true or false | Refuses to answer about another account unless the caller is internal or elevated. |
| `has_groups` | a comma-separated specification with optional negations | true or false | Section 4 of [business-rules.md](business-rules.md). |
| `context_get` | — | the preference context (language, time zone, acting account) | Cached per account. |
| `get_company_currency_id` | — | the identifier of the acting company's currency | |
| `action_get` | — | the description of the *Change My Preferences* screen | |
| `preference_save` | — | an instruction to reload the client context | |
| `preference_change_password` | — | the self-service password dialogue | Guarded by the identity re-check. |
| `action_change_password_wizard` | — | the administrator password dialogue | |
| `api_key_wizard` | — | the new-key dialogue | Guarded by the identity re-check. |
| `action_revoke_all_devices` | — | an instruction to reload | Guarded by the identity re-check. |
| `action_show_groups` / `action_show_accesses` / `action_show_rules` | — | a read-only list of the closure, of its access rights, of its rules | |
| `get_password_policy` | — | a map carrying the minimum length | Contributed by the password-policy package. |
| `signup` | a map of values; an optional token | the pair (login, password) | Contributed by the sign-up package. |
| `reset_password` | a login or address | the notice action | Contributed by the sign-up package. |
| `web_create_users` | a list of addresses | the created accounts | Creates accounts and re-invites already-invited ones. |
| `action_totp_enable_wizard` | — | the enrolment dialogue | Guarded by the identity re-check. |
| `action_totp_disable` | — | a warning notice, or false | Guarded by the identity re-check. |
| `revoke_all_devices` | — | — | Revokes every trusted browser. Guarded by the identity re-check. |
| `action_totp_invite` | — | an informational notice | Mails the enrolment invitation to the selected accounts. |
| `get_totp_invite_url` | — | the path of the enrolment screen | |
| `action_create_passkey` | — | the passkey creation dialogue, carrying the registration options | Guarded by the identity re-check. |
| `remove_oauth_access_token` | — | — | Permitted to a holder of *Access Rights* or to the owner. |
| `auth_oauth` | the provider identifier; the provider's parameters | the triple (database name, login, token) | Runs as the account of identifier 1. |

### 3.2 On the Access Group (`res.groups`)

| Operation | Inputs | Output |
|---|---|---|
| `action_show_all_users` | — | a read-only list of every account that reaches the group |

### 3.3 On the Application Key (`res.users.apikeys`)

| Operation | Inputs | Output | Notes |
|---|---|---|---|
| `remove` | — | a close instruction | Guarded by the identity re-check and by the ownership test. |
| `generate` | an existing key; a scope; a label; an expiry | the new clear key | Gated by the programmatic-management switch; runs inside the cooldown guard. |
| `revoke` | a key | true | Same gating; raises *The provided API key is invalid.* when nothing matches. |

### 3.4 On the User Settings (`res.users.settings`)

| Operation | Inputs | Output |
|---|---|---|
| `set_res_users_settings` | a map of field names to values | the changed subset, formatted, plus the identifier |

### 3.5 On the Configuration Settings (`res.config.settings`)

| Operation | Inputs | Output |
|---|---|---|
| `get_values` | — | the map of non-conventional values |
| `set_values` | — | — |
| `execute` | — | the next action (an uninstall dialogue, a configuration step, or a reload instruction) |
| `cancel` | — | the action that re-opens the screen |
| `get_option_path` | a menu reference | the pair (full menu path, action identifier) |
| `get_option_name` | an entity-and-field name | the field's label |
| `get_config_warning` | a message with placeholders | a warning, redirecting when a menu placeholder was present |
| `open_company` | — | the acting company's form |
| `open_new_user_default_groups` | — | the default-group dialogue, creating the group if it is missing |
| `edit_external_header` | — | the document template's view, or false |
| `action_open_template_user` | — | the external-user template's form |

### 3.6 On the Onboarding entities

| Operation | Inputs | Output |
|---|---|---|
| `action_close_panel` (on the panel) | a panel reference | — (silently does nothing when the reference is unknown) |
| `action_close` / `action_toggle_visibility` (on the panel) | — | — |
| `action_refresh_progress_ids` (on the panel) | — | — |
| `action_set_just_done` (on the step) | — | the steps that actually moved |
| `action_validate_step` (on the step) | a step reference | `NOT_FOUND`, `JUST_DONE` or `WAS_DONE` |

### 3.7 On the Privacy and Recycling entities

| Operation | Inputs | Output |
|---|---|---|
| `action_lookup` (search wizard) | — | the result list action |
| `action_open_lines` (search wizard) | — | the result list action |
| `action_unlink` / `action_archive_all` / `action_unlink_all` (search line) | — | — |
| `action_open_record` (search line) | — | the found record's form |
| `action_recycle_records` (rule) | — | the candidate list for a manual rule; nothing for an automatic one |
| `open_records` (rule) | — | the candidate list filtered to this rule |
| `action_validate` / `action_discard` (candidate) | — | — |

### 3.8 On the Directory Configuration (`res.company.ldap`)

| Operation | Inputs | Output |
|---|---|---|
| `test_ldap_connection` | — | a notice describing success or the precise failure (section 23.7 of [entities.md](entities.md)) |

---

## 4. Addressable paths

The **mode** column is the authentication mode of section 9.10 of
[business-rules.md](business-rules.md).

### 4.1 Sign-in and session

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/` | retrieve | none | Redirects a signed-in non-internal account to the customer landing page; otherwise to the back office. |
| `/web`, the application alias path and its sub-paths, and `/scoped_app/<any sub-path>` | retrieve | none | The back office. Ensures a database, redirects to the sign-in page when there is no session, re-checks the session token, redirects a non-internal account to the customer landing page, refreshes the session lifetime, and renders the client. The response forbids framing entirely and forbids storage. On a permission failure it redirects to the sign-in page with the marker `error=access`. |
| `/web/login` | retrieve, submit | none | The sign-in page (section 1 of [workflows.md](workflows.md)). |
| `/web/login_successful` | retrieve | user | The landing page for external accounts. |
| `/web/become` | retrieve | user | Replaces the session account with the account of identifier 1; only for holders of *Role / Administrator*. |
| `/web/webclient/load_menus` | retrieve | user | The menu tree. Opted out of identity checking, because the client fetches it outside its ordinary call machinery. |
| `/web/health` | retrieve | none | A liveness answer; optionally probes the storage server. The session is not saved. |
| `/robots.txt` | retrieve | none | Disallows everything except an explicitly allowed list of paths. |

### 4.2 Second factor

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/web/login/totp` | retrieve, submit | public | The second-factor page (section 2 of [workflows.md](workflows.md)). Rendered inside the site layout, without language negotiation. |
| `/auth-timeout/send-totp-mail-code` | call | user | Mails a code on demand during re-authentication. Opted out of identity checking. |

### 4.3 Passkeys

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/auth/passkey/start-auth` | call | public | Returns the authentication options and stores the challenge in the session. |
| `/.well-known/assetlinks.json` | retrieve | public | Returns the accepted mobile application signing-key statements, so a companion application counts as the same origin. |

### 4.4 Delegated sign-in

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/auth_oauth/signin` | retrieve | none | The provider's return address. Reads the opaque state, checks the database name against the filter and answers with a bad-request condition when it does not pass, validates the token as the account of identifier 1, **commits** so the possibly-created account is visible to the following authentication, establishes the session, and redirects. The destination is the state's explicit destination, or an action path, or a menu path, or the back office; a non-internal account that would land on the back office is sent to the root instead. |
| `/auth_oauth/oea` | retrieve | none | Starts the flow against the shipped provider for a named database. |

Failures of the return address are turned into a redirect back to the sign-in page carrying one of
three markers, which the page renders as:

| Marker | Message |
|---|---|
| 1 | Sign up is not allowed on this database. |
| 2 | Access Denied |
| 3 | You do not have access to this database or your invitation has expired. Please ask for an invitation and be sure to follow the link in your invitation email. |

### 4.5 Registration and password reset

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/web/signup` | retrieve, submit | public | Registration (workflow 11). Protected by a challenge-response test under the name `signup`. Does not exist without a token when self-registration is closed. Framing restricted to the same origin. |
| `/web/reset_password` | retrieve, submit | public | Password reset (workflow 12). Protected by a challenge-response test under the name `password_reset`. Does not exist without a token when reset is disabled. Framing restricted to the same origin. |
| `/.well-known/change-password` | retrieve | public | Redirects to the password-reset page, so a password manager can offer a change-password shortcut. |

### 4.6 Session lifetime

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/auth-timeout/check-identity` | retrieve | user | Renders the re-authentication form as a page, carrying the original destination. Opted out of identity checking. |
| `/auth-timeout/session/check-identity` | call | user | Receives the re-authentication form. Not read-only, because checking can write (the mailed-code rate limiter). Opted out of identity checking. |

### 4.7 The customer-facing area

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/my`, `/my/home` | retrieve | user | The dashboard. |
| `/my/counters` | call | user | The per-card counts; read-only. |
| `/my/account` | retrieve, submit | user | The account details. |
| `/my/addresses` | retrieve | user | The address list; read-only. |
| `/my/address` | retrieve | user | One address form; read-only. |
| `/my/address/submit` | submit | user | Creates or updates an address. |
| `/my/address/country_info/<a country>` | call, submit | public | The address fields and the mandatory fields of a country; read-only. |
| `/my/address/archive` | call, submit | user | Archives an address; refuses the main address with *You cannot archive your main address* |
| `/my/security` | retrieve, submit | user | The security page and the password change. Framing restricted to the same origin. |
| `/my/deactivate_account` | submit | user | Self-service account removal (workflow 13). |
| `/portal/attachment/remove` | call | public | Removes an attachment the visitor added; refuses one already linked to a message with *The attachment … cannot be removed because it is linked to a message.* |
| `/mail/unfollow` | retrieve | — | Unsubscribes a recipient from a document's notifications. |
| `/portal/chatter_init`, `/mail/chatter_fetch` | call | public | The discussion thread data of a customer-facing document. |
| `/mail/update_is_internal` | call | user | Marks a message internal. |
| `/mail/avatar/mail.message/<a message identifier>/author_avatar/<width>x<height>` | retrieve | public | The author image shown in a customer-facing thread. |

### 4.8 The setup dashboard

| Path | Method | Mode | Purpose |
|---|---|---|---|
| `/base_setup/data` | call | user | Returns the number of active internal accounts, the number of internal accounts that have never signed in, the ten newest such accounts as pairs of identifier and login, and an action opening them. Refused to anyone without *Access Rights* with the message *Access Denied*. |
| `/base_setup/demo_active` | call | user | Whether any package carries demonstration data. |
| `/kpi/summary` | call | — | Aggregates the indicator providers declared by the installed packages; authenticated with an application key verified directly against the key table. |

### 4.9 The customer-facing document path

Any entity adopting the portal document mixin defines its own path, returned by the mixin's address
field. The common contract is:

- the path accepts an `access_token` (the security-token parameter);
- it may accept `report_type` (the report kind: `html`, `pdf` or `text`), `download` (a marker
  requesting a file rather than a page), and a fragment;
- it may accept `pid` (the recipient contact identifier) and `hash` (the recipient signature);
- the generic redirecting path `/mail/view` accepts `model` (the entity transport name) and `res_id`
  (the record identifier) together with the token.

The access decision on those paths is the document check of section 12.1 of
[business-rules.md](business-rules.md). Rendering a printable document from such a path is refused
for `report_type` outside the three accepted values (*Invalid report type: …*) and for documents
spanning several companies (*Multi company reports are not supported.*). The response carries the
content type of the chosen kind, its length, and — for the printable kind — a file name built by
replacing every run of non-word characters in the document's base file name with a single low line,
and a disposition of *attachment* when a download was requested and *inline* otherwise.

---

## 5. Messages sent

| Message | Sent when | To | Content |
|---|---|---|---|
| Account created — internal | An internal account is created, or is re-invited in creation mode | the account's address | The registration link. |
| Account created — external | An external account is created or granted customer-facing access | the account's address | The registration link, plus the wizard's invitation message when one was typed, marked with the medium *portal invitation*. |
| Password reset | A reset is requested | the account's address | The reset link. Rendered directly from its view rather than from a template. Subject *Password reset*, sent from the company's address or the account's own. |
| Registration confirmed | A registration completes | the new account | A confirmation. |
| Unregistered-account reminder | The reminder job finds accounts created 5 days ago that have never signed in | each inviter | The list of *name (login)* strings. Sent with the light notification layout, queued. |
| Second-factor invitation | An administrator invites accounts to enrol | each invited account | The enrolment link. Sent forcibly, from the inviting user's address and authorship, with the light notification layout. |
| Second-factor code | The second-factor page opens for an account whose method is the mailed one, or the code is requested during re-authentication | the account's address | The six-digit code and the validity, rendered as a human-readable duration in the account's language; plus, when a request exists, the device, the browser, the network address and — when resolvable — the city and country. Sent forcibly, copies suppressed, marked for automatic deletion, with the light notification layout. |
| Security alert — new connection | A sign-in succeeds on an account with a second factor, from a browser that is not trusted | the account's address | Subject *New Connection to your Account*; body *A new device was used to sign in to your account.* |
| Security alert — second factor activated | The second-factor secret is set | the account | Subject *Security Update: 2FA Activated*; body *Two-factor authentication has been activated on your account*. The message does **not** suggest enabling the second factor. |
| Security alert — second factor deactivated | The secret is cleared | the account | Subject *Security Update: 2FA Deactivated*; body *Two-factor authentication has been deactivated on your account*. |
| Security alert — trusted device removed | A trusted browser key is deleted | the account | Subject *Security Update: Device Removed*; body *A trusted device has just been removed from your account: <the device labels, comma-separated>*. |
| Document share | The share wizard sends | each recipient | Posted on the document as an internal note with the light notification layout; subject *Invitation to access <the document display name>*; body carrying the recipient, the note, the record, the share link and the entity's label in lower case. |
| Recycling notice | A manual rule's cadence elapses and there is something to review | the rule's notified users | Subject *Data to Recycle*; body carrying the number of candidates created within the period, the entity's label, the rule and the area's menu. |

All security alerts share one rendering: they optionally end with a suggestion to enable the second
factor, and that suggestion is suppressed both when the account already has it and when the message
is itself about the second factor.

---

## 6. External services

| Service | Used by | Contract |
|---|---|---|
| An external identity provider | delegated sign-in | The visitor is sent to the provider's authorisation address with the client identifier, the scope and an opaque state. The provider returns a bearer token. The server calls the information address, either with a bearer header (when `auth_oauth.authorization_header` is set) or with the token as a query parameter, with a 10-second timeout, and expects a structured document carrying a subject under one of `sub`, `id` or `user_id`. An optional second address is merged in. An unsuccessful response is inspected for a bearer challenge carrying an error. |
| A central directory server | directory sign-in | A connection is opened to `ldap://<server>:<port>`; referral chasing is switched off unless `auth_ldap.disable_chase_ref` says otherwise; transport security is negotiated when requested. The query account binds, the subtree of the search base is searched with the formatted filter and a 60-second timeout, exactly one entry must remain, and that entry's distinguished name is bound with the supplied password. A password change binds with the old password and invokes the directory's password change. |
| An authenticator application | the second factor | The enrolment address of section 7.6 of [calculations.md](calculations.md), delivered as a quick response code. |
| A public-key authenticator | passkeys | The registration and authentication option sets, the challenge, the relying-party identifier (the host of the base address), the expected origins (the base address, plus the accepted mobile application signing-key hashes), and the requirements that the credential be discoverable and that user verification take place. |
| A geolocation source | device logging and notices | Resolves a network address to a country and a city. When it cannot, the fields are simply empty. |
| A challenge-response test | registration and password reset, and optionally sign-in | Verified before the submission is processed, under the names `signup`, `password_reset` and `login`. |

---

## 7. Import and export

The domain defines no import or export format of its own. Two mechanisms of the domain govern
exporting in general:

| Mechanism | Effect |
|---|---|
| The group *Allowed* under the privilege *Export* | Holding it permits exporting records at all. The group is implied by *Role / Administrator* and is held by the system account. |
| The access-right rows on the export entities | The export-definition entity is writable only by holders of *Allowed*; the export-line entity is writable by every internal user. |

Accounts, groups, companies, access rights, record rules and defaults are all ordinary entities and
can therefore be imported and exported like any other, subject to those rules. Two cautions apply
and must be reproduced:

1. Importing an account with a password writes the password through the hashing path, so the
   imported value must be a clear password, not a hash. A value that is already an encoded hash is
   recognised at start-up and left alone.
2. Importing groups or access rights does **not** clear the caches synchronously in every path;
   the create, write and delete operations of those entities do, which is why bulk loading should go
   through them rather than through raw insertion.

---

## 8. Printable documents

The domain produces **no printable document of its own**. What it contributes to every other
domain's printable documents is the company block: the logo, the tagline, the details, the footer,
the paper format, the document template, the font and the two colours, all specified in section 12
of [entities.md](entities.md). Changing the font, either colour or the document template clears the
compiled-asset cache, because those values are woven into the generated presentation rules.
