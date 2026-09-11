# Identity and Access — Entities

This file specifies every entity owned by the identity-and-access domain, and every field this
domain adds to entities owned elsewhere. For each entity it gives the purpose, the life cycle, the
complete field table, the relations, the uniqueness rules, the ordering, the display-name rule, the
archival behaviour and the multi-company behaviour.

> **Note on reproduced texts.** Message texts, button labels and screen titles are reproduced
> **verbatim**, exactly as the system emits them, because external tests and user documentation
> depend on them. Where such a reproduced text contains an abbreviation, the abbreviation belongs to
> the text and is not this document's own prose. The abbreviations that occur are: *API* for
> application programming interface; *2FA* for two-factor authentication; *LDAP* for the central
> directory protocol; *DN* for distinguished name; *OAuth* and *UID* for the delegated sign-in
> protocol and its subject identifier; *JSON* for the structured-literal notation; *HTTP* for the
> transport protocol; *ID* for identifier. Outside such reproduced texts, every term is written in
> full.

Field tables use three columns:

- **Field (storage name)** — the human name followed by the reproduced storage name in code font.
- **Type** — the storage type. `Text` is a single-line string; `Long text` is multi-line; `Rich
  text` is markup; `Selection` lists its values; `Link to X` is a many-to-one foreign key; `Collection
  of X` is a one-to-many back-reference; `Set of X` is a many-to-many relation, with its relation
  table named where it matters; `Binary` is an opaque byte payload; `Structured value` is a
  serialised object.
- **Meaning and rules** — required, default, computed and from what, stored or not, readonly, copy
  behaviour on duplication, change tracking, company scoping, indexing, delete behaviour, selection
  labels.

Unless a field row says otherwise: the field is optional, is stored, is writable, is copied when the
record is duplicated, and is not tracked.

Two conventions recur throughout this domain and are defined once here:

- **Restricted field** — a field that carries a list of groups. Reading or writing it is refused
  unless the acting user belongs to one of those groups. A field may also be marked *never
  accessible*, in which case no group can read or write it through the ordinary field path and only
  privilege-elevated internal code may touch it.
- **Privilege elevation** — a mode in which access-right and record-rule checks are suspended while
  the identity of the acting user is unchanged. It is written in this document as "elevated". See
  [business-rules.md](business-rules.md) section 5.

---

## 1. User

**Transport name** `res.users` — **table** `res_users`.

### 1.1 Purpose

A User is one account able to sign in. It holds only the technical identity: the login, the secret,
the groups, the companies, the preferences and the credentials. Everything personal — the name, the
electronic mail address, the telephone number, the language, the time zone, the image, the postal
addresses — lives on a Contact record, and the User *delegates* to it: every field of the Contact
is readable and writable through the User as if it were its own, and the two records are created and
archived together.

Three kinds of user exist, distinguished only by which of three mutually exclusive groups the user
holds:

| Kind | Group held | Meaning |
|---|---|---|
| Internal | *Role / User* (`base.group_user`) | A member of the organisation. Sees the back office. |
| External | *Role / Portal* (`base.group_portal`) | A customer, supplier or partner. Sees only the customer-facing pages and only their own documents. |
| Anonymous | *Role / Public* (`base.group_public`) | The identity under which a visitor with no account browses public pages. There is exactly one such account per company. |

A user that is not internal is a **shared** user; the derived flag `share` (Share User) records this
and is used in hundreds of record rules across the system.

Two accounts are special and are shipped with the system:

- The **system account** (identifier 1). Every environment created for this account is permanently
  privilege-elevated, so it bypasses all access rights and record rules. It is the account under
  which package installation and scheduled jobs run. It cannot be deleted, cannot be re-activated
  through the ordinary path, and holds the *Role / Administrator* group.
- The **administrator account** (identifier 2). An ordinary human account that holds *Role /
  Administrator*. It cannot be deleted, only archived.

### 1.2 Life cycle

1. **Created.** A Contact is created or supplied; the account is attached to it. Default groups are
   applied (section 1.6). A per-user settings row is created if the account is internal. If the
   account has an electronic mail address and the creation was not marked "no password mail", an
   invitation with a sign-up token is sent (see the sign-up package, section 24).
2. **Invited.** The account exists but has never signed in. The derived status is *Invited*.
3. **Confirmed.** After the first successful sign-in a sign-in log row exists and the derived status
   becomes *Confirmed*.
4. **Archived.** The active flag is cleared. The linked Contact is archived with it. A user cannot
   archive the account they are currently signed in as, and the system account can never be
   re-activated.
5. **Queued for deletion.** An external user may ask for their own account to be removed: the login
   is replaced by a reserved unusable value, the secret is cleared, every application key is
   revoked, the account and Contact are archived, and a deletion request row is created. A scheduled
   job performs the actual removal later.
6. **Deleted.** Only through the deletion queue, or by an administrator directly. Deleting the
   system account, the administrator account, the anonymous account or the external-user template is
   refused.

### 1.3 Field table — identity and secret

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Related Contact (`partner_id`) | Link to Contact | **Required.** The record that carries all personal data. Delete behaviour *restrict*: the Contact cannot be deleted while a user points at it. Indexed. Search-access is bypassed on this link, so a user readable to you does not require its Contact to be readable to you. |
| Login (`login`) | Text | **Required.** The identifier typed at sign-in. Globally unique; the uniqueness constraint message is *"You can not have two users with the same login!"*. When the login looks like a single electronic mail address, changing it also sets the electronic mail address (form-level assistance only). |
| Password (`password`) | Text | **Not stored as typed.** Reading always yields the empty string. Writing runs the value through the one-way hashing context and stores the resulting encoded hash in the column; the plain value never reaches the column. Not copied on duplication. Empty means the account cannot sign in with a password. |
| Set Password (`new_password`) | Text | Write-only assistance field. Reading yields the empty string. Writing a non-empty value sets the password **of another user**; writing it for oneself is refused with *"Please use the change password wizard (in User Preferences or User menu) to change your own password."* An empty value is silently ignored. Not stored. |
| Active (`active`) | Boolean | Default true. Clearing it archives the account. Setting it true on the system account is refused with *"You cannot activate the superuser."* Clearing it on the account currently signed in is refused with *"You cannot deactivate the user you're currently logged in as."* Setting it true also un-archives the linked Contact. |
| Contact is Active (`active_partner`) | Boolean | Mirror of the Contact's active flag. Read-only. |

### 1.4 Field table — groups, role and companies

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Groups (`group_ids`) | Set of Access Group, relation table `res_groups_users_rel` (columns `uid`, `gid`) | The groups **explicitly** assigned. Default: the *Role / User* group plus every group implied by the *Default access for new users* group (section 1.6). Changing it clears the access caches. |
| Groups and implied groups (`all_group_ids`) | Set of Access Group | Computed, not stored, computed with elevation. The reflexive transitive closure of implication over `group_ids` — that is, the explicit groups plus every group they imply, directly or indirectly. This is the set every access decision is taken against. Searchable through the closure. |
| Role (`role`) | Selection | Computed from the groups, writable. Values: `group_user` (*User*), `group_system` (*Administrator*). It is *Administrator* when the closure contains the *Role / Administrator* group, *User* when it contains *Role / User*, otherwise empty. Writing it on the form swaps the two role groups inside `group_ids` while leaving every other group untouched, and only when the account is already internal. |
| Share User (`share`) | Boolean | Computed from the closure, **stored**, computed with elevation. False when the closure contains *Role / User*; true otherwise. Every "only my own documents" rule keys off this flag. |
| Default Company (`company_id`) | Link to Company | **Required.** Default: the company active in the creating environment. The company used when the user creates a record and no company is otherwise imposed. Must be an element of the permitted companies (section 1.9). Carries the marker that makes company pickers list only the user's permitted companies. |
| Companies (`company_ids`) | Set of Company, relation table `res_company_users_rel` (columns `user_id`, `cid`) | The companies the user is permitted to work in. Default: the company active in the creating environment. |
| Number of Companies (`companies_count`) | Integer | Computed, not stored: the total number of companies in the database (used to decide whether to show the company switcher). |
| # Groups (`groups_count`) | Integer | Computed with elevation: the size of the closure. |
| # Access Rights (`accesses_count`) | Integer | Computed with elevation: the number of access-right rows attached to any group of the closure. |
| # Record Rules (`rules_count`) | Integer | Computed with elevation: the number of record rules attached to any group of the closure. |
| Group hierarchy snapshot (`view_group_hierarchy`) | Structured value | Not stored, not copied. A snapshot of all groups, privileges and categories with their implication sets, handed to the client so the group editor can be rendered without further calls. |

### 1.5 Field table — preferences, logs, devices, keys

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Electronic Mail Signature (`signature`) | Rich text | Computed from the name, writable, stored. When the name is set and the signature is empty, it becomes the name wrapped in a block element. |
| Home Action (`action_id`) | Link to Action | The screen opened right after sign-in, in addition to the standard menu. Three actions are refused (section 1.10). |
| Sign-in log entries (`log_ids`) | Collection of User Sign-in Log | Every successful sign-in. |
| Latest Sign-in (`login_date`) | Date and time | Mirror of the newest sign-in log entry's creation date. |
| Devices (`device_ids`) | Collection of Device | Every device that has held a session for this account and has not been revoked. |
| Application Keys (`api_key_ids`) | Collection of Application Key | The long-lived secrets this account may use instead of a password on programmatic connections. |
| Settings (`res_users_settings_ids`) | Collection of User Settings | At most one row. |
| Settings (single) (`res_users_settings_id`) | Link to User Settings | Computed, not stored: the first (and only) settings row. Exists so that other entities can relate to a single-valued target. |
| Time zone offset (`tz_offset`) | Text | Computed from the time zone: the current offset from coordinated universal time, formatted as a sign followed by four digits (for example `+0200`). |
| Electronic mail domain placeholder (`email_domain_placeholder`) | Text | Computed from the acting user's own electronic mail address; used only as a form hint. |

### 1.6 Default groups of a new account

1. Start with the group *Role / User*.
2. Add every group **implied by** the group named *Default access for new users*. (That group exists
   precisely so that an administrator can define, once, the set of application groups every new
   internal account should receive; the group itself is not given to the user, only the groups it
   implies.)
3. The result is written to the explicit groups of the new account.

An account created as a copy of the external-user template (sign-up, portal grant, directory
provisioning) does **not** go through this default: it receives exactly the template's groups, which
is the single group *Role / Portal*.

### 1.7 Self-service field access

A user may read and write a restricted set of fields on **their own** account without holding any
administration group. Two lists govern this; both are extended by other packages and the union is
cached.

**Readable by oneself.** Electronic mail signature, default company, login, electronic mail address,
name, every image size (`image_1920`, `image_1024`, `image_512`, `image_256`, `image_128`), language,
time zone, time-zone offset, groups, related contact, last modification date, home action, every
avatar size (`avatar_1920`, `avatar_1024`, `avatar_512`, `avatar_256`, `avatar_128`), share flag,
devices, application keys, telephone number, display name. Packages add: the second-factor enabled
flag and trusted devices; the passkey collection; the delegated-sign-in token presence flag.

**Writable by oneself.** Electronic mail signature, home action, default company, electronic mail
address, name, largest image, language, time zone, application keys, telephone number.

Rules:

1. A read of one's own account that touches only readable-by-oneself fields (or fields whose name
   begins with `context_`) is performed with elevation, so ordinary access rights do not block it.
2. A write to one's own account that touches only writable-by-oneself fields is performed with
   elevation. If such a write includes the default company and the proposed company is **not** among
   the user's permitted companies, that key is silently dropped rather than refused.
3. The field-access check treats a readable-by-oneself field as readable when the record being read
   is the acting user themselves.
4. The field catalogue returned to the client is augmented with these fields, marked read-only when
   they are readable but not writable, and marked not searchable.

### 1.8 Ordering, display name, uniqueness, archival

- **Default ordering**: name ascending, then login ascending.
- **Display name**: the Contact's display name. Name search first tries an exact match on the login;
  if exactly one account matches, that single result is returned and no further search is done.
  Otherwise the ordinary name search runs, extended so that a value matching a login also matches.
- **Uniqueness**: login is globally unique.
- **Duplication**: the copy receives name and login suffixed with the word *(copy)* unless those
  were supplied explicitly. The secret, the sign-up token type, the application keys and the
  delegated-sign-in identifier are not copied.
- **Archival**: supported through the active flag; archiving the account archives the Contact.

### 1.9 Multi-company behaviour

- The default company must be an element of the permitted companies. Violation raises:

  > Company *the company name* is not in the allowed companies for user *the user name* (*the
  > comma-separated list of permitted company names*).

  The check runs only for active accounts.
- Writing the default company also moves the Contact to that company, **unless** the Contact has no
  company (a company-less Contact stays global).
- Writing either company field resets the cached company properties of every open environment whose
  acting user is among the written accounts.
- When the *Multi Companies* group exists, creating or writing an account recomputes membership of
  that group mechanically: an account with **two or more** permitted companies is put into the group;
  an account with **one or zero** is taken out of it. This happens on creation, on every write that
  touches the permitted companies, and on the in-memory record used by the form.

### 1.10 Validation rules on the User

| Rule | Condition | Message |
|---|---|---|
| Default company must be permitted | Active account whose default company is not in its permitted companies | *Company … is not in the allowed companies for user … (…).* |
| Home action may not be the application switcher | Home action equals the shipped "open website" action | *The "App Switcher" action cannot be selected as home action.* |
| Home action may not be a reload | Home action is a client action whose tag is the reload tag | *The "…" action cannot be selected as home action.* |
| Home action may not need a pre-selected record | Home action is a window action whose context mentions the active record key | *The action "…" cannot be set as the home action because it requires a record to be selected beforehand.* |
| Mutually exclusive kinds | The closure contains more than one of *Role / User*, *Role / Portal*, *Role / Public* | *User 'the user name' cannot be at the same time in exclusive groups 'first group', 'second group'.* |
| At least one administrator | After the change, no account at all holds *Role / Administrator* | *You must have at least an administrator user.* (Suspended while the foundation package itself is being installed or updated.) |

### 1.11 Deletion guards

| Condition | Message |
|---|---|
| The system account is in the selection | *You can not remove the admin user as it is used internally for resources created by the system (updates, module installation, ...)* |
| The administrator account is in the selection | *You cannot delete the admin user because it is utilized in various places (such as security configurations,...). Instead, archive it.* |
| The external-user template is in the selection | *Deleting the template users is not allowed. Deleting this profile will compromise critical functionalities.* |
| The anonymous account is in the selection | *Deleting the public user is not allowed. Deleting this profile will compromise critical functionalities.* |

### 1.12 Session-token fields

A small set of fields is designated as **session-token fields**. Their values are the key material
from which the session token is derived (see [calculations.md](calculations.md) section 6), so
changing any of them invalidates every open session of that account. The base set is: identifier,
login, password, active. Packages extend it: the second-factor secret; the delegated-sign-in token;
the set of registered passkeys.

A larger set is designated as **cache-invalidating fields**: the session-token fields plus groups,
active, language, time zone, default company and permitted companies. A write touching any of them
clears the registry caches.

---

## 2. User Sign-in Log

**Transport name** `res.users.log` — **table** `res_users_log`.

### 2.1 Purpose

One row is inserted on every successful sign-in. The row carries no payload of its own: the acting
user and the timestamp come from the automatic authorship columns. The newest row of an account is
what the *Latest Sign-in* field of the User shows, and the existence of any row is what makes an
account *Confirmed* rather than *Invited*.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Created by (`create_uid`) | Link to User | Read-only, indexed. The account that signed in. |
| Creation date (`create_date`) | Date and time | Read-only. The moment of the sign-in. |

- **Default ordering**: identifier descending (newest first).
- **Housekeeping**: an automatic clean-up deletes every row of an account except its newest one.
- Rows are created with elevation and with no values, so that a concurrent transaction is never
  affected.

---

## 3. User Settings

**Transport name** `res.users.settings` — **table** `res_users_settings`.

### 3.1 Purpose

A single row per internal account, holding client-side preferences that other packages extend
(notification preferences, panel widths, voice settings and so on). The foundation defines only the
link; the value of the entity is the guaranteed one-row-per-user contract and the formatting
protocol used to hand the row to the client.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| User (`user_id`) | Link to User | **Required.** Delete behaviour *cascade*. Not indexed. Restricted in pickers to accounts that do not already have a settings row. |

- **Uniqueness**: one row per account. Violation message: *"One user should only have one user
  settings."*
- **Display name**: the linked account.
- **Find-or-create**: a helper returns the existing row of an account or creates one with elevation.
- **Formatting for the client**: every field except the display name is returned; the account link is
  returned as an object carrying only the identifier.
- **Partial update**: a named operation accepts a map of field names to values, writes only those
  that differ from the current value, and returns the changed subset plus the identifier.

---

## 4. User Deletion Request

**Transport name** `res.users.deletion` — **table** `res_users_deletion`.

### 4.1 Purpose

Deleting an account is expensive, because every entity in the database carries authorship columns
pointing at it and those columns are not always indexed. Self-service deletions are therefore queued
and executed by a scheduled job, in batches, with a commit between records.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| User (`user_id`) | Link to User | Delete behaviour *set to empty*, so the request survives the deletion it caused. |
| User identifier (`user_id_int`) | Integer | Computed from the link, **stored**. Keeps the numeric identity after the account row is gone. |
| State (`state`) | Selection | **Required**, default `todo`. Values: `todo` (*To Do*), `done` (*Done*), `fail` (*Failed*). |

- **Display name**: the linked account.
- **Processing**: see [workflows.md](workflows.md) section 12.

---

## 5. Device Log and Device

**Transport names** `res.device.log` — **table** `res_device_log`; `res.device` — database view
`res_device`.

### 5.1 Purpose

Every session that reaches the server leaves a trace: which platform, which browser, which network
address, when it was first and last seen. The log is append-only and one row is written at most once
per hour per (session, platform, browser, address) combination. The Device entity is a read-only
view over the log that keeps, for each (account, session, platform, browser) combination, only the
newest non-revoked row — that is, the list a user sees under "my devices".

### 5.2 Field table (shared by both)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Session Identifier (`session_identifier`) | Text | **Required**, indexed. The stored prefix of the session identifier, not the identifier itself. |
| Platform (`platform`) | Text | The operating system reported by the client. |
| Browser (`browser`) | Text | The browser reported by the client. |
| Network Address (`ip_address`) | Text | The remote network address of the request. |
| Country (`country`) | Text | Resolved from the network address. |
| City (`city`) | Text | Resolved from the network address. |
| Device Type (`device_type`) | Selection | `computer` (*Computer*) or `mobile` (*Mobile*). Set to *Mobile* when the reported platform, lower-cased, is one of: android, iphone, ipad, ipod, blackberry, windows phone, webos. Otherwise *Computer*. |
| User (`user_id`) | Link to User | Indexed. The account the session belonged to. |
| First Activity (`first_activity`) | Date and time | When this combination was first seen in this session. |
| Last Activity (`last_activity`) | Date and time | When it was last seen. Indexed. |
| Revoked (`revoked`) | Boolean | True once the session store no longer holds the session — either because it was explicitly revoked or because it expired. |
| Current Device (`is_current`) | Boolean | Computed, not stored: true when the current request's session identifier begins with this row's stored prefix. |
| Linked Network Addresses (`linked_ip_addresses`) | Long text | Computed, not stored: every distinct network address recorded for the same (session, platform, browser) combination, one per line, in first-seen order. |

- **Display name**: the platform and the browser, each with its first letter capitalised, separated
  by a space; the word *Unknown* stands in for a missing part.
- **Default ordering of the Device view**: last activity descending. Sorting by *Current Device*
  is translated into "rows whose session identifier equals the current one first".
- **Indexes on the log**: a composite index on (account, session identifier, platform, browser, last
  activity, identifier) restricted to non-revoked rows, and an index on the revoked flag restricted
  to non-revoked rows.

### 5.3 Write rule for the log

A row is written only when the session's own trace says something changed:

1. If the session carries the "tracing disabled" marker, nothing is written at all. (Only elevated
   internal code can set that marker; it exists for automated technical sessions.)
2. Otherwise the session holds a list of traces, each a (platform, browser, network address,
   first-seen, last-seen) tuple. If a trace matches the current request's platform, browser and
   address:
   - and its last-seen value is **3600 seconds or more** in the past, the last-seen value is
     advanced to now and a log row is written;
   - otherwise nothing is written.
3. If no trace matches, a new trace is appended with first-seen and last-seen both set to now, and a
   log row is written.
4. The log row is written on a writable connection even when the current request is read-only.

### 5.4 Housekeeping

- **De-duplication.** Rows of the same (session, platform, browser, network address) except the one
  with the greatest last-activity are deleted. When the job knows the time of its own previous run,
  only groups whose surviving row is newer than that time are considered.
- **Revocation sweep.** Rows that are not revoked and whose last activity is older than the
  configured maximum session inactivity are checked against the session store in batches of 100 000;
  those whose session no longer exists are marked revoked.

### 5.5 Revoking

Revoking one or more devices deletes the corresponding sessions from the session store and marks
every log row with those session identifiers as revoked. If the current device is among them, the
current session is signed out. The action is protected by the identity re-check (section 13).

Revoking **all** devices of the acting account revokes every device except the current one.

---

## 6. Access Group

**Transport name** `res.groups` — **table** `res_groups`.

### 6.1 Purpose

A group is a named set of users. It is the only currency in which permission is expressed: access
rights name a group, record rules name groups, menus name groups, views name groups, restricted
fields name groups.

Groups form a directed graph through **implication**. If group *A* implies group *B*, then every
user of *A* is also, for all purposes, a user of *B*. Implication is what lets a manager group carry
the whole user group's permissions without restating them. The relation is interpreted as set
inclusion: *A* implies *B* means the set *A* is a subset of the set *B*.

### 6.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Text | **Required**, translatable. May not begin with a hyphen: writing such a name is refused with *"The name of the group can not start with \"-\""*. |
| Users (`user_ids`) | Set of User, relation table `res_groups_users_rel` (columns `gid`, `uid`) | The accounts **explicitly** in this group. |
| Users and implied users (`all_user_ids`) | Set of User | Computed, not stored: every account explicitly in this group **or in any group that implies it**. Writable: see the inverse rule below. Archived accounts are included. |
| # Users (`all_users_count`) | Integer | Computed with elevation: the size of the above. |
| Access Controls (`model_access`) | Collection of Access Right | The per-entity permission rows attached to this group. Copied when the group is duplicated. |
| Rules (`rule_groups`) | Set of Record Rule, relation table `rule_group_rel` (columns `group_id`, `rule_group_id`) | The non-global record rules attached to this group. |
| Access Menu (`menu_access`) | Set of Menu, relation table `ir_ui_menu_group_rel` | Menus visible to this group. |
| Views (`view_access`) | Set of View, relation table `ir_ui_view_group_rel` | Views restricted to this group. |
| Comment (`comment`) | Long text | Translatable. Explanatory text shown next to the group in the administration screens. |
| Group Name (`full_name`) | Text | Computed, not stored: when the group has a privilege and the reading context does not ask for the short form, the privilege name, a space, a solidus, a space, then the group name; otherwise just the group name. Searchable (section 6.5). |
| Share Group (`share`) | Boolean | Marks a group created for the purpose of sharing data with external users. Purely descriptive. |
| Application key maximum duration in days (`api_key_duration`) | Decimal | The longest validity, in days, of an application key created by a member of this group. Must be zero or positive; the check message is *"The api key duration cannot be a negative value."* |
| Sequence (`sequence`) | Integer | Presentation order inside a privilege. |
| Privilege (`privilege_id`) | Link to Group Privilege | Indexed. The named choice this group belongs to. |
| Implied Groups (`implied_ids`) | Set of Access Group, relation table `res_groups_implied_rel` (columns `gid`, `hid`) | The groups this group's users also belong to. |
| Transitively Implied Groups (`all_implied_ids`) | Set of Access Group | Computed with elevation, recursive, not stored: **the group itself plus every group reachable by following implication**, to any depth. |
| Implying Groups (`implied_by_ids`) | Set of Access Group, same relation table with the columns exchanged | The inverse of implication: groups whose users are also in this group. |
| Transitively Implying Groups (`all_implied_by_ids`) | Set of Access Group | Computed with elevation, recursive, not stored: the group itself plus every group that reaches it by following implication. |
| Disjoint Groups (`disjoint_ids`) | Set of Access Group | Computed, not stored: for the three kind groups (*Role / User*, *Role / Portal*, *Role / Public*), the other two; empty for every other group. |
| Group hierarchy snapshot (`view_group_hierarchy`) | Structured value | Computed, not stored. Same snapshot as on the User. |

### 6.3 Uniqueness and ordering

- **Uniqueness**: (privilege, name) is unique. Violation message: *"The name of the group must be
  unique within a group privilege!"*
- **Default ordering**: privilege, then sequence, then name, then identifier.
- **Sorting by full name** is handled specially: the whole matching set is fetched, sorted in memory
  on the computed full name, then paged.
- **Display name**: the full name.
- **Duplication**: the copy's name is the original's name followed by the word *(copy)*, unless a
  name was supplied. Access-right rows are copied with the group.

### 6.4 Writing the implied users

Writing *Users and implied users* on a group is interpreted as follows:

1. Let *target* be the written set and *current* be the accounts reachable today (explicit accounts
   of this group and of every group that implies it).
2. Accounts in *target* but not in *current* are **added to the explicit accounts** of this group.
3. Accounts in *current* but not in *target* are **removed from the explicit accounts** of this
   group.
4. If any account that was to be removed is still reachable through an implying group, the write is
   refused:

   > It is not possible to remove implied group *'the group name'* from users *the comma-separated
   > list of names*

### 6.5 Searching the full name

A search on the full name with a positive operator is expanded into a disjunction:

1. The plain condition on the group name.
2. For each searched value: the condition on the group name.
3. If the value contains a solidus, the part before it is the privilege name and the part after it
   the group name (both trimmed); the disjunct is "privilege name matches" **and** "group name
   matches". If the value contains no solidus, it is treated as a privilege name alone and the
   disjunct is "privilege name matches".

Negative operators are not supported and fall back to the generic behaviour.

### 6.6 Validation rules on the group

| Rule | Condition | Effect |
|---|---|---|
| Disjoint kinds preserved | Implication was changed in a way that could give some user two of the three kind groups | The group cache is cleared and a search is made for a single offending active user; if found, that user's own disjointness check is run and raises. |
| Inherited view groups | The set of restricted views changed | Each affected view re-checks its own group consistency. |
| Not a settings group | The group is the implied group of a settings checkbox field | *You cannot delete a group linked with a settings field.* |

The disjointness search is deliberately written as a search rather than a per-user loop: the domain
looks for one active user who is explicitly in any of the changed groups and whose closure contains
two different kind groups.

### 6.7 Cache behaviour

- Creating, deleting, or writing implication clears the **group cache** (the cache that holds the
  compiled implication graph).
- Any write at all first clears the access caches, because the derived *Share User* flag on users
  depends on group membership.

### 6.8 The compiled group definition

The implication graph is compiled once and cached. Its inputs are, for every group ordered by
identifier: the group's external references (or its numeric identifier as a string when it has
none), the identifiers of the groups it implies (its *supersets*), and the identifiers of the groups
it is disjoint from. The compiled structure answers three questions used everywhere:

- the identifier of a group given an external reference;
- the **superset closure** of a set of groups — every group reachable by implication;
- the **subset closure** of a set of groups — every group that reaches them.

It also exposes an *empty* expression (no user) and a *universe* expression (every user), used by the
access-right summary of section 8.6.

### 6.9 Applying and removing an implied group

Two operations exist, used by the settings mechanism:

- **Apply** group *G* to a set of groups *S*: for every group of *S* whose transitive closure does
  **not** already contain *G*, add *G* to its direct implied groups.
- **Remove** group *G* from a set of groups *S*: take the union of the transitive closures of *S*;
  for every group in that union that directly implies *G*, remove *G* from its direct implied
  groups.

The asymmetry is deliberate. Applying only touches the named groups; removing must reach into the
whole closure, otherwise the permission would survive through an intermediate group.

### 6.10 Ensuring external references

Groups that have no external reference receive one on demand, of the form
`__custom__.group_<identifier>`. This is needed because several mechanisms address groups by
reference rather than by identifier.

---

## 7. Group Privilege

**Transport name** `res.groups.privilege` — **table** `res_groups_privilege`.

### 7.1 Purpose

A privilege is the unit the administrator actually manipulates: on the user form, an application
category shows a list of privileges, and each privilege offers a single choice among its groups
(plus "no access"). *Sales / Administrator*, *Sales / User: own documents only* and *Sales / User:
all documents* are three groups of one privilege.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Text | **Required**, translatable. |
| Description (`description`) | Long text | Help text shown under the privilege. |
| Placeholder (`placeholder`) | Text | Default `No`. The label of the "no group" choice in the selector. |
| Sequence (`sequence`) | Integer | Default 100. Presentation order within the category. |
| Category (`category_id`) | Link to Application Category | Indexed. The application heading this privilege sits under. |
| Groups (`group_ids`) | Collection of Access Group | The mutually-ordered choices. |

- **Default ordering**: sequence, then name, then identifier.
- **Ordering of the choices inside a privilege** (used when rendering the selector): each group is
  ranked by the number of groups of *this same privilege* contained in its transitive closure
  (groups without a privilege rank zero), then by sequence, then by identifier. The effect is that
  the least powerful choice is offered first and the most powerful last.

---

## 8. Access Right

**Transport name** `ir.model.access` — **table** `ir_model_access`.

### 8.1 Purpose

An access right is a single sentence of the form *"members of group G may do operations O on entity
E"*. Four independent flags carry the operations. Rows are **additive**: a user may perform an
operation on an entity if **any** row grants it for **any** group in the user's closure, or if any
row grants it with **no group at all** (such a row grants the operation to everyone; the system logs
a warning when one is created, because it is an anti-pattern).

Absence of any granting row means the operation is refused — the model is deny-by-default.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Text | **Required**, indexed. A free label, conventionally *entity name.group suffix*. |
| Active (`active`) | Boolean | Default true. An inactive row grants nothing. |
| Entity (`model_id`) | Link to Entity definition | **Required**, indexed, delete behaviour *cascade*. |
| Group (`group_id`) | Link to Access Group | Indexed, delete behaviour *restrict* — a group cannot be deleted while a row names it. Empty means "everyone". |
| Read Access (`perm_read`) | Boolean | Grants reading records of the entity. |
| Create Access (`perm_create`) | Boolean | Grants creating records of the entity. |
| Write Access (`perm_write`) | Boolean | Grants modifying records of the entity. |
| Delete Access (`perm_unlink`) | Boolean | Grants deleting records of the entity. |

- **Default ordering**: entity, then group, then name, then identifier.
- **Cache**: creating, writing or deleting a row invalidates everything and clears the stable cache
  (which holds, per user and operation, the set of permitted entities).

### 8.2 The permitted-entity set

For one acting user and one operation, the set of entity names the user may operate on is computed
once and cached per (user, operation). It is the set of entity names for which at least one active
row grants that operation and either names no group or names a group in the user's closure.

### 8.3 Groups with access to an entity

For the error message and for administration screens, the system can list the names of the groups
that hold an operation on an entity: every active row of that entity granting that operation whose
group is set, rendered as *privilege name/group name* when the group has a privilege and as the
group name otherwise, ordered by privilege name then group name with unnamed privileges last.

### 8.4 The refusal message

When the entity-level check fails the following message is produced. It has three parts joined by
blank lines.

Part one depends on the operation:

| Operation | Text |
|---|---|
| read | You are not allowed to access '*the entity label*' (*the entity transport name*) records. |
| write | You are not allowed to modify '*the entity label*' (*the entity transport name*) records. |
| create | You are not allowed to create '*the entity label*' (*the entity transport name*) records. |
| delete | You are not allowed to delete '*the entity label*' (*the entity transport name*) records. |

Part two is either

> This operation is allowed for the following groups:
> *a tabulated list, one group per line, each preceded by a tabulation, a hyphen and a space*

when at least one group holds the operation, or

> No group currently allows this operation.

when none does.

Part three is always

> Contact your administrator to request access if necessary.

### 8.5 Elevation

Under privilege elevation the entity-level check returns *permitted* immediately, without consulting
any row.

### 8.6 The group expression of an entity

A second, coarser summary is cached per (entity, operation): a *group expression* describing who may
perform the operation. It is:

- the **empty** expression when no active row grants the operation;
- the **universe** expression when at least one granting row has no group;
- otherwise the expression built from the identifiers of the granting rows' groups.

This is what other mechanisms consult when they need to know, statically, which groups can see an
entity — for instance when deciding whether a menu should be shown.

---

## 9. Record Rule

**Transport name** `ir.rule` — **table** `ir_rule`.

### 9.1 Purpose

A record rule narrows an operation from "all records of the entity" to "the records satisfying this
filter". A rule carries one filter expression, the entity it applies to, the operations it applies
to, and the groups it applies to.

The single most important property is the **combination semantics**, specified exhaustively in
[business-rules.md](business-rules.md) section 3:

- rules with **no group** are *global*; they are combined by **intersection** (every one of them must
  be satisfied);
- rules **with** groups that the acting user holds are combined by **union** (satisfying any one of
  them suffices);
- the union of the group rules is then **intersected** with the global part.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Text | A free label, shown in the diagnostic message when debugging is on. |
| Active (`active`) | Boolean | Default true. Clearing it disables the rule without deleting it — important because a shipped rule that is deleted is re-created when its package is reloaded. |
| Entity (`model_id`) | Link to Entity definition | **Required**, indexed, delete behaviour *cascade*. |
| Groups (`groups`) | Set of Access Group, relation table `rule_group_rel` (columns `rule_group_id`, `group_id`) | Delete behaviour *restrict*. Empty means the rule is global. |
| Filter (`domain_force`) | Long text | The filter expression, written in the filter language, evaluated in the evaluation context of section 9.3. Empty means "always true". |
| Global (`global`) | Boolean | Computed from the groups, **stored**: true exactly when the rule has no group. |
| Read (`perm_read`) | Boolean | Default true. The rule constrains reading. |
| Write (`perm_write`) | Boolean | Default true. The rule constrains modification. |
| Create (`perm_create`) | Boolean | Default true. The rule constrains creation. |
| Delete (`perm_unlink`) | Boolean | Default true. The rule constrains deletion. |

### 9.2 Constraints

| Rule | Condition | Message |
|---|---|---|
| At least one operation | All four operation flags are false | *Rule must have at least one checked access right!* |
| Not on the rule entity itself | The entity of the rule is the record-rule entity | *Rules can not be applied on the Record Rules model.* |
| Filter must be valid | The rule is active and has a filter that fails to parse or does not validate against the entity | *Invalid domain: the parser's own message* |

### 9.3 Evaluation context

The filter expression is evaluated with exactly three names in scope:

| Name | Value |
|---|---|
| the acting user | The acting user's record, taken with an **empty context**, so that the evaluation cannot depend on language, active-test or any other contextual key. |
| the active company identifiers | The identifiers of the companies currently active in the environment, in order, first one first. |
| the active company identifier | The identifier of the first active company. |

Because the acting user is exposed as a record, a filter can walk any relation from it — for
example "documents whose salesperson is the acting user", or "documents whose company is one of the
active companies".

### 9.4 Caching

The combined filter for one (entity, operation) is cached per acting user, per elevation state, per
entity, per operation, and per the values of the **cache-key context entries**. The only cache-key
context entry is the list of active companies; a list value is converted to a tuple for the key.
Creating, writing or deleting any rule flushes everything and clears the caches. When the developer
mode for definition files is on, the cache is bypassed entirely.

### 9.5 Ordering and display

- **Default ordering**: entity descending, then identifier.
- Rules are always fetched in ascending identifier order when they are being combined, so the
  combination is deterministic.

---

## 10. Default Value

**Transport name** `ir.default` — **table** `ir_default`.

### 10.1 Purpose

A default value is a stored proposal for one field of one entity, used when a new record is being
prepared and the field has no other value. Defaults are scoped, and the scope determines precedence.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Field (`field_id`) | Link to Field definition | **Required**, indexed, delete behaviour *cascade*. |
| User (`user_id`) | Link to User | Indexed, delete behaviour *cascade*. Empty means the default applies to every user. |
| Company (`company_id`) | Link to Company | Indexed, delete behaviour *cascade*. Empty means the default applies in every company. |
| Condition (`condition`) | Text | An opaque string restricting applicability. The client writes single-field conditions of the form *key=value*. Empty means unconditional. |
| Default Value (`json_value`) | Text | **Required.** The value, serialised as a structured literal. |

- **Display name**: the field.
- **Uniqueness**: enforced procedurally rather than by constraint — setting a default searches for
  an existing row with the same (field, user, company, condition) and rewrites it.

### 10.2 Validation

| Rule | Condition | Message |
|---|---|---|
| Well-formed serialisation | The stored value is not a valid structured literal | *Invalid JSON format in Default Value field.* |
| Convertible to the field | The parsed value cannot be converted to the field's type | *Invalid value in Default Value field. Expected type 'the field type' for 'the entity name.the field name'.* |
| Field must be writable by the acting user | Not elevated, and the acting user cannot write the target field | The field-access refusal of section 26.2. Checked after creation and after every write. |

Additional checks are applied when a default is **set** through the named operation rather than
written directly:

| Condition | Message |
|---|---|
| The entity or field does not exist | *Invalid field the entity name.the field name* |
| The value cannot be converted | *Invalid value for the entity name.the field name: the value* |
| An integer value outside the 32-bit signed range | *Invalid value for the entity name.the field name: the value is out of bounds (integers should be between -2,147,483,648 and 2,147,483,647)* |

### 10.3 Precedence

See [calculations.md](calculations.md) section 9 for the exact ordering. In summary: for one entity
and one condition, rows whose user is empty or the acting user and whose company is empty or the
acting company are read, ordered by user, then company, then identifier, and the **first** row seen
for each field wins. Because an empty value sorts before a set value in the ordering used, this
makes a global default win over a per-user one — see the calculation section for the precise
consequence and the worked example.

### 10.4 Cache and side effects

- Creating, writing or deleting a default invalidates the whole environment and clears the registry
  caches, because company-dependent fields take their fallback value from defaults.
- The per-(user, company, entity, condition) map of defaults is cached.
- Deleting records of an entity discards every default of a link field pointing at any of them.
- Discarding by value removes every default of a given field whose serialised value is one of a
  given list.

---

## 11. System Parameter

**Transport name** `ir.config_parameter` — **table** `ir_config_parameter`.

### 11.1 Purpose

A global key/value store for database-wide settings. Values are always text; the settings mechanism
converts to and from the declared type of the settings field.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Key (`key`) | Text | **Required**, unique. Violation message: *Key must be unique.* |
| Value (`value`) | Long text | **Required.** |

- **Display name**: the key. **Default ordering**: key.
- **Reading** is cached per key in the stable cache and bypasses the object layer entirely, because
  parameters are consulted from within field computations.
- **Writing** a value of *false* or *absent* **deletes** the parameter; writing any other value
  creates or updates it and returns the previous value.
- **Renaming** a parameter that is one of the initialised defaults is refused:
  *You cannot rename config parameters with keys the comma-separated list of keys*
- Creating, writing or deleting clears the stable cache.

### 11.2 Parameters initialised when the database is created

| Key | Initial value |
|---|---|
| `database.secret` | A freshly generated random universally unique identifier (version 4). This is the key material of every signed token in the system. |
| `database.uuid` | A freshly generated time-based universally unique identifier (version 1). |
| `database.create_date` | The current date and time. |
| `web.base.url` | The address the application answers on, initialised to the loop-back host and the configured port. |
| `base.login_cooldown_after` | 10 |
| `base.login_cooldown_duration` | 60 |

One further parameter is shipped as data: `base.default_max_email_size` with value 10.

Note the deliberate discrepancy: the sign-in cooldown code falls back to **5** failures when the
parameter is absent, while the initialisation writes **10**. A database created normally therefore
behaves with a threshold of 10; a database where the parameter has been deleted behaves with a
threshold of 5. Both figures are specified in [calculations.md](calculations.md) section 8.

---

## 12. Company

**Transport name** `res.company` — **table** `res_company`.

### 12.1 Purpose

A company is a legal or organisational unit. Companies form a **tree**: a company may have a parent,
and the descendants of a company are called its *branches*. The root of a tree is the company with
no parent. Certain fields are **delegated to the root** — they must have the same value on every
company of a tree — and the currency is the only such field in the foundation.

Like the User, a Company delegates its address, its electronic mail address, its telephone number,
its tax identifiers and its image to a Contact.

### 12.2 Field table — identity and hierarchy

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Company Name (`name`) | Text | **Required**, stored, mirrors the Contact's name and writes through to it. Globally unique; violation message *The company name must be unique!* |
| Active (`active`) | Boolean | Default true. Archiving a company archives every branch beneath it. |
| Sequence (`sequence`) | Integer | Default 10. Orders companies in the company switcher. |
| Parent Company (`parent_id`) | Link to Company | Indexed, delete behaviour *restrict*. **Immutable after creation**: any write touching it is refused with *The company hierarchy cannot be changed.* |
| Branches (`child_ids`) | Collection of Company | The direct children. |
| All branches (`all_child_ids`) | Collection of Company | The direct children including archived ones. |
| Hierarchy path (`parent_path`) | Text | Indexed. The materialised path of ancestor identifiers. |
| Ancestors (`parent_ids`) | Set of Company | Computed with elevation, not stored: the companies named in the hierarchy path, root first; for a company with no path, itself. |
| Root Company (`root_id`) | Link to Company | Computed with elevation, not stored: the first ancestor, i.e. the root of the tree. |
| Contact (`partner_id`) | Link to Contact | **Required**, indexed. |
| Accepted Users (`user_ids`) | Set of User, relation table `res_company_users_rel` | The accounts permitted to work in this company (the inverse of the User's permitted companies). |

### 12.3 Field table — currency, address and presentation

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Currency (`currency_id`) | Link to Currency | **Required.** Default: the currency of the creating user's default company. **Delegated to the root**: a branch must carry the same currency as its root. Writing a currency that is archived activates it. |
| Street, Street 2, Postal code, City (`street`, `street2`, `zip`, `city`) | Text | Computed from the Contact's contact-type address, each with its own write-through. |
| Federated State (`state_id`) | Link to State | Computed from the Contact, writes through. Restricted in pickers to states of the selected country. |
| Country (`country_id`) | Link to Country | Computed from the Contact, writes through. Selecting a country proposes that country's currency. |
| Country code (`country_code`) | Text | Mirror of the country's code. |
| Electronic mail address (`email`), Telephone (`phone`) | Text | Mirrors of the Contact, stored, writable. |
| Website (`website`) | Text | Mirror of the Contact, writable. |
| Tax identifier (`vat`) | Text | Mirror of the Contact, writable, labelled *Tax ID*. |
| Company registry (`company_registry`) | Text | Mirror of the Contact, writable, labelled *Company ID*. |
| Bank accounts (`bank_ids`) | Collection of Bank Account | Mirror of the Contact's bank accounts, writable. |
| Company Tagline (`report_header`) | Rich text | Translatable. Placed in the header or footer of printed documents depending on the chosen layout. |
| Report Footer (`report_footer`) | Rich text | Translatable. Bottom of every printed document. |
| Company Details (`company_details`) | Rich text | Translatable. Top of every printed document. |
| Company details are empty (`is_company_details_empty`) | Boolean | Computed: true when the details, reduced to plain text, are empty. Exists because an "empty" rich-text field still contains an empty paragraph. |
| Company Logo (`logo`) | Binary | Mirror of the Contact's largest image; defaults to the shipped placeholder logo. |
| Logo for the web (`logo_web`) | Binary | Computed from the logo, **stored**, not kept as an attachment: the logo resized to 180 points wide with the height derived. Stored in the row itself because it is fetched directly for performance. |
| Uses the default logo (`uses_default_logo`) | Boolean | Computed, stored: true when there is no logo or the logo equals the shipped placeholder. |
| Paper format (`paperformat_id`) | Link to Paper Format | Default: the shipped European format. |
| Document Template (`external_report_layout_id`) | Link to View | The document skin used for printed documents. |
| Font (`font`) | Selection | Default *Lato*. Values: `Lato`, `Roboto`, `Open_Sans` (*Open Sans*), `Montserrat`, `Oswald`, `Raleway`, `Tajawal`, `Fira_Mono` (*Fira Mono*). |
| Primary colour (`primary_color`), Secondary colour (`secondary_color`) | Text | Colours of the document skin. |
| Colour (`color`) | Integer | Computed from the root: the root's Contact colour if set, otherwise the root's identifier modulo 12. Writing it writes the root's Contact colour. |
| Layout background (`layout_background`) | Selection | **Required**, default `Blank`. Values: `Blank` (*Blank*), `Demo logo` (*Demo logo*), `Custom` (*Custom*). |
| Layout background image (`layout_background_image`) | Binary | Used when the layout background is *Custom*. |
| Uninstalled localisation packages (`uninstalled_l10n_module_ids`) | Set of Package | Computed from the country: the automatically-installable packages bound to that country whose dependencies are all satisfied and which are not yet installed. |

### 12.4 Creation

1. For every proposed company that has a name but no Contact, a Contact is created first, marked as
   an organisation, carrying the proposed name, image, electronic mail address, telephone number,
   website, tax identifier and country. The created Contacts are flushed so that their computed
   address fields exist, then attached.
2. For every proposed company with a parent, each **root-delegated field** absent from the proposal
   is filled from the parent.
3. The registry caches are cleared, then the rows are created.
4. Both the creating user and the system account gain the new companies in their permitted
   companies.
5. Any currency used by a new company that is archived is activated.
6. Companies that have a country trigger installation of that country's localisation packages,
   unless the environment is a test run, an import, or a package-installation pass.

Duplication is refused outright: *Duplicating a company is not allowed. Please create a new company
instead.*

### 12.5 Write behaviour

1. A write touching the parent is refused (section 12.2).
2. A write setting an archived currency activates that currency first.
3. After the write: if the written keys intersect {active, sequence}, the registry caches are
   cleared (those fields feed the cached list of a user's companies). If the written keys intersect
   {font, primary colour, secondary colour, document template}, the compiled-asset cache is cleared.
4. Clearing the active flag archives every branch.
5. For every written **root** company, each written root-delegated field is copied down to every
   branch beneath it, in sorted field order.
6. A company that gains a country for the first time triggers localisation installation.
7. If any address field was written, the address fields of the whole entity are invalidated so that
   they are recomputed from the Contact.

### 12.6 Constraints

| Rule | Condition | Message |
|---|---|---|
| Cannot archive a company still used as a default | Archiving a company that is the default company of at least one active account | *The company the company name cannot be archived because it is still used as the default company of the number of users users.* |
| Root-delegated fields must match the root | A branch whose delegated field differs from its parent's | *The the field label of a subsidiary must be the same as it's root company.* |

### 12.7 Accessible branches

For a given company and the currently active companies, the **accessible branches** are computed as
follows:

1. Start with an empty result and with *current* set to the company itself.
2. While *current* is non-empty: add to the result those companies of *current* that are also among
   the active companies; then set *current* to the direct children of *current*.
3. If the result is empty **and** the acting account is the system account, return the company
   itself instead. (The system account bypasses record rules, so the intersection may legitimately
   be empty in a scheduled job.)

The result is cached per (active companies, company, acting user).

A related predicate answers whether a set of companies is exactly the set of all companies in the
trees rooted at their roots — used by operations that only make sense for whole companies.

### 12.8 The anonymous account of a company

Each company can produce an anonymous account: the first account holding *Role / Public* whose
default company is this company; if none exists, the shipped anonymous account is copied with the
name *Public user for the company name*, the login `public-user@company-<identifier>.com`, and this
company as its only permitted company.

### 12.9 Ordering, display, archival

- **Default ordering**: sequence, then name.
- **Name search** honours a context marker meaning "this picker is a user preference": in that case
  the search is elevated and restricted to the acting user's permitted companies, so that a user
  sees all their own companies even if record rules would hide some.
- **Archival**: supported; cascades to branches; blocked by active users (section 12.6).
- **Main company**: the shipped company is named *My Company*, with the United States dollar as
  currency. If it is missing, the company with the lowest identifier takes its place.

---

## 13. Application Key

**Transport name** `res.users.apikeys` — **table** `res_users_apikeys` (created explicitly, not by
the object layer, so that it can hold a secret column the object layer never exposes).

### 13.1 Purpose

An application key is a long random secret that stands in for a password on non-interactive
connections. It is stored **only as a hash**; the clear value is shown exactly once, at creation.
A key may be scoped, so that it is accepted only for one purpose, and it normally expires.

### 13.2 Columns

The table is created with the following columns. Only the first five are exposed as fields.

| Column | Exposed as | Type | Meaning |
|---|---|---|---|
| `id` | identifier | Serial | Primary key. |
| `name` | Description (`name`) | Text | **Required, read-only.** The label shown in the key list. |
| `user_id` | User (`user_id`) | Link to User | **Required, read-only**, indexed, delete behaviour *cascade*. |
| `scope` | Scope (`scope`) | Text | **Read-only.** Empty means a global key, accepted for any purpose. A non-empty value restricts the key to that purpose. |
| `expiration_date` | Expiration Date (`expiration_date`) | Date and time | **Read-only.** Empty means the key never expires. |
| `create_date` | Creation Date (`create_date`) | Date and time | **Read-only.** Defaults to the current moment in coordinated universal time. |
| `index` | not exposed | Text of exactly 8 characters | The first 8 hexadecimal characters of the clear key, used to find candidate rows cheaply. A check constraint enforces the length. |
| `key` | not exposed | Text | The hash of the clear key. |

An index exists on (user, index). When the generated index name would exceed 63 characters, a
deterministic shortened name is used instead: the first 50 characters of the table name, the
characters `_idx_`, and the first 8 hexadecimal characters of the hash of the table name.

### 13.3 Generation

1. The requested expiry is validated (section 13.4).
2. 20 random bytes are drawn from the operating system's cryptographic source and rendered as 40
   hexadecimal characters. This is the clear key.
3. A row is inserted with the label, the acting user, the scope, the expiry, the **hash** of the
   clear key, and the first 8 characters of the clear key as the index.
4. The clear key is returned to the caller and never stored.

The hashing context used for keys is deliberately cheaper than the one used for passwords (6 000
iterations rather than at least 600 000), because the key is 160 bits of uniform randomness and is
not subject to dictionary attack.

### 13.4 Expiry validation

| Condition | Outcome |
|---|---|
| The acting user holds the *Role / Administrator* group | No check at all; a key with no expiry is allowed. |
| No expiry given | *The API key must have an expiration date* |
| Expiry later than now plus the user's maximum duration | *You cannot exceed the maximum number of days days.* |
| Expiry at or before now | *You cannot set an expiration date in the past.* |

The user's maximum duration is the **greatest** *Application key maximum duration in days* among all
groups of the user's closure; if every such value is zero or absent, it is **1.0** day.

### 13.5 Verifying a key

Given a scope and a candidate key:

1. Assert that both scope and key are non-empty.
2. Take the first 8 characters of the candidate as the index.
3. Select every row joined to an **active** account whose index equals that value, whose scope is
   either empty or equal to the requested scope, and whose expiry is either empty or not before the
   current moment.
4. For each candidate row in turn, verify the candidate key against the stored hash; the first match
   returns the account identifier.
5. If no row matches, the result is "no account".

### 13.6 Removal

- **Self-service removal** is protected by the identity re-check.
- Removal is permitted when the acting environment is that of a system user **or** when every key in
  the selection belongs to the acting user. Otherwise:
  *You can not remove API keys unless they're yours or you are a system user*
- Removal deletes the rows with elevation and clears the registry caches (the verification path
  caches negative results).

### 13.7 Programmatic key management

Two named operations allow a program holding a valid key to mint and revoke keys without a browser.
Both are gated:

> Programmatic application programming interface keys are not enabled

unless the acting environment is that of a system user **or** the parameter
`base.enable_programmatic_api_keys` is set to a true value. (Administrators are exempt on purpose:
otherwise an administrator would simply switch the parameter on, use it and switch it off, leaving a
window in which everyone could do the same.)

**Minting** takes an existing key, a scope, a label and an expiry:

1. The whole operation runs inside the sign-in cooldown guard, keyed on the first 8 characters of
   the supplied key.
2. A string expiry is parsed; both a date and a date-with-time are accepted.
3. The number of the acting user's keys that are unexpired is counted. If it is at least the limit,
   the operation is refused:
   *Limit of the limit API keys is reached for programmatic creation*
   The limit comes from `base.programmatic_api_keys_limit`, default 10; a non-numeric value falls
   back to 10 with a warning.
4. The supplied key is verified against the requested scope (or the global purpose when no scope is
   requested). A global key may mint a key of any scope; a scoped key may only mint keys of its own
   scope. If verification fails, or succeeds for a different account:
   *The provided API key is invalid or does not belong to the current user.*
5. A new key is generated and returned.

**Revoking** takes a key:

1. The operation runs inside the sign-in cooldown guard.
2. Unexpired rows with the matching index are selected and the key is verified against each; the
   first match is removed and the operation reports success.
3. If no row matches: *The provided API key is invalid.*

### 13.8 Housekeeping

An automatic clean-up deletes every row whose expiry is set and is in the past.

### 13.9 Application Key Description Wizard

**Transport name** `res.users.apikeys.description` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Description (`name`) | Text | **Required.** |
| Duration (`duration`) | Selection | **Required.** Computed choice list (below). Default: the first available choice. |
| Expiration Date (`expiration_date`) | Date and time | Computed from the duration, stored, writable. |

The choice list is built as follows. The base choices are, as a number of days rendered as text:
`1` (*1 Day*), `7` (*1 Week*), `30` (*1 Month*), `90` (*3 Months*), `180` (*6 Months*), `365`
(*1 Year*). A system user additionally gets `0` (*Persistent Key*, meaning no expiry) and `-1`
(*Custom Date*). A non-system user gets only those base choices whose number of days is at most the
user's maximum duration, plus `-1` (*Custom Date*).

The expiry is derived from the duration: for a duration of zero or more days other than zero, today
plus that many days; for a duration of exactly zero, no expiry; for the custom choice (−1), the
expiry is left to the user. Changing the expiry by hand validates it immediately and, if invalid,
raises a non-blocking notice titled *The API key duration is not correct.* carrying the validation
message. Creating the wizard record validates the expiry outright.

Producing the key is protected by the identity re-check and additionally refuses non-internal users:

> Only internal users can create application programming interface keys

On success the clear key is handed to a display record.

### 13.10 Application Key Display

**Transport name** `res.users.apikeys.show` — abstract, no table.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Identifier (`id`) | Identifier | Present only so the client can receive the computed key. |
| Key (`key`) | Text | Read-only. The clear key, shown once. |

---

## 14. Identity Check Wizard

**Transport name** `res.users.identitycheck` — transient.

### 14.1 Purpose

Certain operations are too sensitive to allow on an unattended, already-signed-in browser: creating
an application key, changing one's own password, disabling the second factor, revoking devices,
deleting a passkey. Those operations are wrapped so that, unless the identity was already confirmed
recently, the operation is *suspended*: its full call — context, entity, records, operation name,
positional and keyword arguments — is serialised into a wizard record and a dialogue is opened. When
the dialogue succeeds, the stored call is replayed.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Request (`request`) | Text | **Read-only** and **never accessible** as a field: no group can read it. Holds the serialised suspended call. |
| Authentication method (`auth_method`) | Selection | Default `password` (*Password*). Extended by other packages. |
| Password (`password`) | Text | Not stored. |

### 14.2 The suspension rule

1. If there is no request in progress, the operation is refused outright: *This method can only be
   accessed over HTTP*.
2. If the session records an identity confirmation newer than **10 minutes** ago, the operation runs
   immediately.
3. Otherwise a wizard record is created with elevation, holding a serialised list of six elements:
   the context entries that can be serialised (unserialisable entries are dropped), the entity's
   transport name, the identifiers of the records, the operation's name, the positional arguments
   and the keyword arguments. A medium-sized dialogue on that record is returned, titled *Access
   Control*.

### 14.3 Running the check

1. A request must exist.
2. The credentials are verified interactively, using the acting user's login and the password taken
   **from the context** under the key `password`. Failure raises
   *Incorrect Password, try again or click on Forgot Password to reset your password.*
3. The session's identity-confirmation timestamp is set to now.
4. The stored call is deserialised, the operation is looked up on the recorded records in the
   recorded context, it is asserted that the operation really is one of the protected ones, and it
   is invoked with the stored arguments; its result is returned to the client.

---

## 15. Change Password Wizards

### 15.1 Change Password Wizard (administrator-driven)

**Transport name** `change.password.wizard` — transient, discarded after 0.2 hours.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Users (`user_ids`) | Collection of Change Password Line | Default: one line per account in the acting selection. |

Applying the wizard writes each line's new password, then: if the acting user is among the affected
accounts the client is told to reload; otherwise the dialogue simply closes.

### 15.2 Change Password Line

**Transport name** `change.password.user` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Wizard (`wizard_id`) | Link to Change Password Wizard | **Required**, delete behaviour *cascade*. |
| User (`user_id`) | Link to User | **Required**, delete behaviour *cascade*. |
| User Login (`user_login`) | Text | Read-only. Copied from the account when the line is defaulted. |
| New Password (`new_passwd`) | Text | Default empty. |

Applying a line with a non-empty new password performs the internal password change on that account
(which logs the change, including the acting user and the network address). Immediately afterwards
every line's new password is blanked, so that clear passwords do not linger in the transient table.

### 15.3 Change Own Password Wizard

**Transport name** `change.password.own` — transient, discarded after 0.1 hours.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| New Password (`new_password`) | Text | |
| New Password (Confirmation) (`confirm_password`) | Text | |

Constraint: the two must be identical, otherwise
*The new password and its confirmation must be identical.*

Applying is protected by the identity re-check. It performs the internal password change on the
acting account, deletes the wizard record, and tells the client to reload (so that the session token,
which depends on the password, is refreshed rather than rejected).

---

## 16. Configuration Settings

**Transport name** `res.config.settings` — transient. **Transport name** `res.config` — transient.

### 16.1 Purpose

A settings screen is a transient record whose fields are *not* the settings themselves. Saving the
record **projects** each field onto the thing it actually configures — a user-defined default, a
group implication, a stored parameter, or a package installation — according to a naming convention.
The full mechanism is specified in [configuration.md](configuration.md) section 2; this section
specifies the entity.

The base entity declares no business fields. It declares that a settings field may additionally
carry these attributes:

| Attribute | Applies to | Meaning |
|---|---|---|
| target entity | fields whose name begins with `default_` | The entity whose field receives a user-defined default. |
| stored parameter key | any field | The key of the system parameter the value is stored under. |
| holder groups | boolean and selection fields whose name begins with `group_` | Comma-separated external references of the groups that gain or lose the implication. Defaults to the single group *Role / User*. |
| implied group | boolean and selection fields whose name begins with `group_` | The external reference of the group that is implied or un-implied. |

Duplication is refused: *Cannot duplicate configuration!*

The display name of a settings record is the name of the window action that opens it, or the entity
name when there is no such action.

### 16.2 Classification of the fields

Given a list of field names (or all of them), each is placed in exactly one bucket:

| Bucket | Test | Recorded as |
|---|---|---|
| default | name begins with `default_` | (field name, target entity, the field name with the first 8 characters removed) |
| group | name begins with `group_` | (field name, the holder groups resolved to records, the implied group resolved to a record) |
| package | name begins with `module_` | the package whose name is the field name with the first 7 characters removed |
| parameter | the field carries a stored parameter key | (field name, the key) |
| other | anything else | the field name |

Declaration errors are structural and are raised as such:

- a `default_` field without a target entity;
- a `group_` field that is not boolean or selection;
- a `group_` field without an implied group;
- a `module_` field that is not boolean or selection;
- a parameter field whose type is not one of boolean, integer, decimal, text, selection, link or
  date-and-time.

### 16.3 Reading the current state

Producing the default values of a settings screen:

1. The acting user must be able to read the settings entity; otherwise the read-access refusal is
   raised.
2. For each **default** field: read the stored user-defined default of (target entity, target
   field) with no user scope, no company scope and no condition. If one exists, use it.
3. For each **group** field: the value is *true* exactly when the implied group is in the transitive
   closure of **every** holder group. For a selection field the boolean is rendered as the text `1`
   or `0`.
4. For each **package** field: true when the package's state is *installed*, *to install* or *to
   upgrade*.
5. For each **parameter** field: read the parameter; if it is absent use the field's own default.
   Then convert: a link field parses the value as an identifier and keeps it only if such a record
   still exists (a deleted target yields empty rather than an error); an integer field parses an
   integer, falling back to 0 with a logged warning; a decimal field parses a decimal, falling back
   to 0 with a logged warning; a boolean field parses a truth value, falling back to the truthiness
   of the raw value.
6. Finally, the hook for non-conventional fields contributes its own map, overriding anything above.

### 16.4 Writing the state

Saving the record performs, in this order:

1. **Non-conventional projection** — the entity-specific hook, which by default does the three
   projections below.
2. **Defaults.** For each default field whose value differs from the currently-read state, write a
   user-defined default of (target entity, target field) with the new value. A record-valued field
   contributes its identifier (single link) or the list of identifiers (multiple link). Writing is
   elevated.
3. **Groups.** The group fields are processed **in ascending order of their value**, so that every
   removal happens before every addition. For each field whose value differs from the currently-read
   state: a true value applies the implied group to the holder groups; a false value removes it.
   Both operations are elevated.
4. **Parameters.** For each parameter field: a text value is trimmed and an empty result is stored as
   *false* (which deletes the parameter) — this is deliberate, because keys pasted with surrounding
   spaces are a recurring source of failures; an integer or decimal value is stored as its literal
   representation, or *false* when it is zero; a link value is stored as its identifier. If the
   stored value already equals the new one (compared both as text and directly), nothing is written.
5. **Packages.** Packages whose field is true and which are not installed are collected as *to
   install*; packages whose field is false and which are installed or awaiting upgrade are collected
   as *to uninstall*. If either set is non-empty, everything is flushed first. If there is anything
   to uninstall, the uninstall confirmation dialogue is returned and **nothing further happens in
   this call**. Otherwise the installations are performed. If anything was installed, the transaction
   is reset because the registry has changed.
6. The next configuration step is asked for; if it is not a plain "close", it is returned. Otherwise
   the client is told to reload, so the menus and the current screen pick up the new state.

Saving is refused unless the acting user is an administrator:

> Only administrators can change the settings

### 16.5 Creation optimisation

Saving a settings screen without changing anything would otherwise write every mirrored field and
trigger long recomputation chains. Therefore, at creation, every proposed value of a **mirrored**
writable field is compared with the value the mirror currently holds, computed by walking the mirror
path from the value proposed for the first segment; if they are equal, the key is dropped from the
proposal.

### 16.6 Message helpers

Two helpers turn structured placeholders inside a message into readable text:

- a placeholder naming a menu is replaced by that menu's full path, and the message becomes a
  redirecting warning offering to open the corresponding screen under the label *Go to the
  configuration panel*;
- a placeholder naming a field is replaced by that field's label.

### 16.7 The older configuration item

The `res.config` entity is the base of step-by-step configuration items. It declares four handlers —
next, execute, cancel, skip — and the convention that an execute or cancel returning an action
dictionary replaces the default "go to the next item" behaviour. Its default "next" tells the client
to reload. Implementing `execute` is mandatory; not doing so is a structural error.

---

## 17. Authentication Device (trusted browser)

**Transport name** `auth_totp.device` — shares the application-key table.

### 17.1 Purpose

When a user completes a second-factor challenge and ticks "remember this browser", a scoped
application key is generated with the scope `browser` and stored in a cookie. On the next sign-in,
presenting that cookie satisfies the second factor without a code. The entity reuses the application
key table (and therefore its secret column and its hashing) but is a separate entity so the two
concepts do not mix.

Fields are exactly those of the Application Key. The scope is always `browser`. The label is
composed at creation as *the browser on the platform*, each capitalised, followed — when the
network address resolves to a city — by a space and the city and country in parentheses.

### 17.2 Trusted-device age

The validity of a trusted-browser key, in seconds, is the value of the parameter
`auth_totp.trusted_device_age` interpreted as a whole number of **days** and multiplied by 86 400.
A value that is zero, negative or unparseable falls back, with a logged warning, to **90 days**.

### 17.3 Verification for a specific account

A helper answers "does this key, in this scope, belong to this account?" — it runs the ordinary key
verification and compares the resulting account identifier with the expected one. This is what the
second-factor page calls with the cookie value and the pending account.

---

## 18. Second-factor Rate Limit Log

**Transport name** `auth.totp.rate.limit.log` — transient, table `auth_totp_rate_limit_log`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| User (`user_id`) | Link to User | **Required, read-only.** |
| Network address (`ip`) | Text | Read-only. |
| Limit type (`limit_type`) | Selection | Read-only. `send_email` (*Send Email*) or `code_check` (*Code Checking*). |

An index exists on (user, limit type, creation date).

The limits are: **5 events per 3600 seconds** for mailing a code, and **5 events per 3600 seconds**
for verifying a code. The arithmetic is in [calculations.md](calculations.md) section 7.

---

## 19. Second-factor Setup Wizard

**Transport name** `auth_totp.wizard` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| User (`user_id`) | Link to User | **Required, read-only.** |
| Secret (`secret`) | Text | **Required, read-only.** The shared secret, rendered in base-32 and grouped into blocks of four characters separated by spaces for legibility. |
| Enrolment address (`url`) | Text | Computed from the account's login, the account's company display name and the secret; stored; read-only. |
| Enrolment image (`qrcode`) | Binary | Computed with the address; stored, not kept as an attachment; read-only. A quick response code encoding the enrolment address, drawn with a box size of 4, encoded as a portable network graphic. |
| Verification Code (`code`) | Text | Not stored. At most 7 characters. |

The enrolment address is built as a uniform resource locator with the scheme `otpauth`, the host
`totp`, the path *issuer*, a colon, *login*, and a query carrying: the compressed secret (spaces
removed), the issuer, the algorithm name in capitals, the number of digits, and the period in
seconds. The issuer is the request's host name with any port removed; when there is no request it is
the account's company display name.

Enabling is protected by the identity re-check, and then:

1. The verification code taken from the context is stripped of white space and parsed as a whole
   number. A non-numeric value raises *The verification code should only contain numbers*.
2. The enrolment attempt is made (section 20.4). On failure:
   *Verification failed, please double-check the 6-digit code*
3. On success the secret is blanked on the wizard record and a success notice is shown:
   *2-Factor authentication is now enabled.*

---

## 20. Second factor on the User

The second-factor package adds the following to the User entity.

### 20.1 Fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Second-factor secret (`totp_secret`) | Text | **Never accessible** as a field. Not copied. Computed and written through direct column access, because the column exists but is deliberately not a managed field: reading fetches the column for the row; writing updates the column and clears the counter. |
| Last accepted counter (`totp_last_counter`) | Integer | **Never accessible** as a field. Not copied. The counter value of the most recently accepted code. |
| Two-factor authentication (`totp_enabled`) | Boolean | Computed from the secret (elevated read): true when a secret exists. Searchable: a search for enabled accounts becomes the raw condition "the secret column is not the empty string". |
| Trusted Devices (`totp_trusted_device_ids`) | Collection of Authentication Device | The scoped keys of section 17. |

Both *Two-factor authentication* and *Trusted Devices* are added to the readable-by-oneself list.

### 20.2 Consequences for other credentials

- The secret is added to the session-token fields, so enabling or disabling the second factor
  invalidates every open session of the account.
- An account with the second factor enabled may **not** authenticate a non-interactive connection
  with a password; only an application key is accepted.
- Changing one's own password first revokes every trusted browser.

### 20.3 Verifying a code

When the credential type is the second-factor type:

1. The code-checking rate limit is consumed; exceeding it raises
   *You reached the limit of code verifications for your account, please try again later.*
2. The stored secret is decoded from base-32 and matched against the supplied code (see
   [calculations.md](calculations.md) section 7 for the exact arithmetic).
3. No match: *Verification failed, please double-check the 6-digit code*
4. A match whose counter is **less than or equal to** the last accepted counter: the code is a
   replay, and the refusal is *Verification failed, please use the latest 6-digit code*
5. Otherwise the last accepted counter is advanced to the matched counter, the code-checking rate
   limit rows for this account are purged, and the result reports the account, the method *second
   factor*, and the default multi-factor policy.

### 20.4 Enrolling

1. Refused, silently, if the second factor is already enabled or if the target account is not the
   acting account.
2. The proposed secret is stripped of white space and upper-cased.
3. The supplied code is matched against it; no match means failure.
4. On success the secret and the matched counter are stored, and — when there is a request — the
   session token is recomputed and replaced so the user is not signed out by their own change.

### 20.5 Disabling

Protected by the identity re-check. Permitted when the target is the acting account, or the acting
user is an administrator, or the environment is elevated; otherwise the operation silently reports
failure. It revokes every trusted browser, clears the secret, refreshes the session token when the
target is the acting account, and shows a warning notice:

> Two-factor authentication disabled for the following user(s): *the comma-separated list of names*

---

## 21. Passkey

**Transport name** `auth.passkey.key` — **table** `auth_passkey_key`.

### 21.1 Purpose

A passkey is a public-key credential held by the user's device or password manager. Signing in with
one proves possession of a private key bound to this server's origin, and no shared secret is ever
transmitted. A successful passkey sign-in **skips** the second factor entirely.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Text | **Required.** The label the user gives the credential. |
| Credential identifier (`credential_identifier`) | Text | **Required.** Restricted to the *Role / Administrator* group. Unique; violation message *The credential identifier should be unique.* |
| Public key (`public_key`) | Text | **Required.** Restricted to the *Role / Administrator* group. Stored in a column the object layer does not manage: reading fetches the column directly, writing through the field does nothing and the value is written by direct column update at registration. |
| Signature counter (`sign_count`) | Integer | Default 0. Restricted to the *Role / Administrator* group. Advanced on every successful authentication to detect cloned authenticators. |
| Owner (`create_uid`) | Link to User | Indexed. The account the credential belongs to. |

- **Default ordering**: identifier descending (newest first).
- Deleting a passkey is logged with the passkey identifier, the acting user and the network address.

### 21.2 The collection on the User

The User gains *Passkeys* (`auth_passkey_key_ids`), the collection of credentials owned by the
account, added to the readable-by-oneself list and to the session-token fields. Because the
credentials live in another table, the session-token query is extended with a left join and an
aggregate of the credential identifiers in descending order, so that registering or deleting a
passkey invalidates the account's sessions.

### 21.3 Registration

1. The acting user asks to create a passkey; the operation is protected by the identity re-check.
2. Registration options are generated for the relying party identified by the **host** of the
   configured base address, with the relying-party name, the account identifier as the user handle,
   the login as the user name, and the requirements that the credential be **resident** (discoverable)
   and that **user verification** be required.
3. The challenge from those options is stored in the session under the key `webauthn_challenge`.
4. A dialogue collects a label. Submitting it is again protected by the identity re-check.
5. The authenticator's registration response is verified against: the challenge popped from the
   session, the expected origins (the base address with its path removed, plus the list of accepted
   mobile application signing-key hashes), the expected relying-party identifier (the host), and the
   requirement that user verification took place.
6. A credential row is created **through the account's passkey collection** (so that the session
   token cache is invalidated), with the label and the credential identifier rendered in
   base-64 with the address-safe alphabet.
7. The public key is written by direct column update, encoded in base-64 with the address-safe
   alphabet.
8. The creation is logged and the session token is refreshed.

If no challenge is found in the session, the operation is refused with *Cannot find a challenge for
this session*.

### 21.4 Authentication

**Finding the account.** When the credential type is the passkey type, the response's credential
identifier is looked up directly in the credential table joined to the accounts, and the found
login is put into the credential dictionary so the ordinary sign-in path can proceed. An unknown
identifier raises *Unknown passkey*.

**Verifying.** The credential row of the acting account with that identifier is fetched with
elevation; if there is none, *Unknown passkey*. The authenticator's assertion is verified against
the stored public key, the stored signature counter, the challenge, the expected origins, the
relying-party identifier and the user-verification requirement. A verification failure is turned
into a refusal carrying the verifier's own message. On success the stored signature counter is
replaced by the new one and the result reports the account, the method *passkey*, and the
multi-factor policy **skip**.

### 21.5 Deletion

Protected by the identity re-check. For each selected credential: if it belongs to the acting
account, it is deleted through the account's passkey collection and the session token is refreshed;
otherwise nothing happens and an attempt is logged with both accounts and the network address.

### 21.6 Passkey Creation Wizard

**Transport name** `auth.passkey.key.create` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Text | **Required.** |

---

## 22. Delegated Sign-in Provider

**Transport name** `auth.oauth.provider` — **table** `auth_oauth_provider`.

### 22.1 Purpose

A configuration of an external identity provider that issues bearer tokens. The user is sent to the
provider, comes back with a token, and the server exchanges the token for a subject identity at the
provider's information endpoint.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Provider name (`name`) | Text | **Required.** |
| Client identifier (`client_id`) | Text | The identifier this installation is known by at the provider. |
| Authorisation address (`auth_endpoint`) | Text | **Required.** Where the user is sent. |
| Scope (`scope`) | Text | Default `openid profile email`. What is requested from the provider. |
| User information address (`validation_endpoint`) | Text | **Required.** Where the token is exchanged for the subject identity. |
| Extra data address (`data_endpoint`) | Text | Optional second address merged into the identity. |
| Allowed (`enabled`) | Boolean | Only enabled providers are offered on the sign-in page. |
| Presentation class (`css_class`) | Text | Default `fa fa-fw fa-sign-in text-primary`. The icon class of the button. |
| Sign-in button label (`body`) | Text | **Required**, translatable. |
| Sequence (`sequence`) | Integer | Default 10. |

- **Default ordering**: sequence, then name.

### 22.2 Fields added to the User

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Provider (`oauth_provider_id`) | Link to Delegated Sign-in Provider | |
| Provider user identifier (`oauth_uid`) | Text | Not copied. The subject identity at the provider. |
| Token store (`oauth_access_token`) | Text | Read-only, not copied, never pre-fetched, **never accessible** as a field. |
| Has a token (`has_oauth_access_token`) | Boolean | Computed from the token (elevated read). Restricted to the *Access Rights* group. Added to the readable-by-oneself list. |

Uniqueness: (provider, provider user identifier) is unique. Violation message: *OAuth UID must be
unique per provider*.

The token is added to the session-token fields.

Removing the token is permitted to a holder of the *Access Rights* group or to the owner; otherwise:
*You do not have permissions to remove the access token*

### 22.3 Validating a token

1. The provider's information address is called with the token — either as a bearer authorisation
   header, when the parameter `auth_oauth.authorization_header` is set, or as a query parameter
   otherwise, with a 10-second timeout.
2. A successful response is parsed as a structured document. An unsuccessful one is inspected for an
   authentication challenge of the bearer kind carrying an error, which is returned as the result;
   failing that, the result is the generic invalid-request error.
3. If the result carries an error, validation fails with that error.
4. If the provider has an extra data address, it is called the same way and the result is merged in.
5. The subject identity is unified: the first non-empty value among the keys `sub`, `id` and
   `user_id` is taken, removed from the payload, and re-inserted under `user_id`. If none is
   present: *Missing subject identity*

### 22.4 Signing in

1. Search for an account with that provider and that subject identity. If found, its stored token is
   replaced with the new one and its login is returned.
2. If not found and the environment forbids account creation, the attempt yields nothing.
3. Otherwise provisioning values are composed: the name from the payload's `name`, falling back to
   the electronic mail address; the login and the electronic mail address from the payload's
   `email`, falling back to the synthetic value `provider_<provider identifier>_user_<subject>`; the
   provider; the subject; the token; active.
4. The sign-up path is invoked with those values and with the token carried in the provider's opaque
   state under the key `t`. Any sign-up refusal is converted back into the original access refusal.

### 22.5 The token as a credential

The token can also be presented as a credential. The ordinary credential check runs first; if it
refuses and the credential is a provider token, then — provided the connection is interactive or the
account does not require application keys, and the account is active — an account with that
identifier and that stored token is searched for. A match reports the account, the method *delegated
sign-in*, and the default multi-factor policy.

---

## 23. Directory Configuration

**Transport name** `res.company.ldap` — **table** `res_company_ldap`.

### 23.1 Purpose

A binding to a central directory server against which sign-in credentials can be verified, and from
which accounts can be provisioned. Several bindings may exist; they are tried in sequence order.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sequence (`sequence`) | Integer | Default 10. The order in which bindings are tried. |
| Company (`company`) | Link to Company | **Required**, delete behaviour *cascade*. The company provisioned accounts are attached to. |
| Directory server address (`ldap_server`) | Text | **Required**, default `127.0.0.1`. |
| Directory server port (`ldap_server_port`) | Integer | **Required**, default 389. |
| Query account distinguished name (`ldap_binddn`) | Text | The account used to search the directory. Empty means an anonymous search. |
| Query account password (`ldap_password`) | Text | |
| Search filter (`ldap_filter`) | Text | **Required.** A directory filter in string form. Every `%s` placeholder is replaced by the supplied login; the filter should contain at least one. The filter **must** match exactly one entry, otherwise the sign-in is refused. |
| Search base (`ldap_base`) | Text | **Required.** The distinguished name whose whole subtree is searched. |
| Template User (`user`) | Link to User | The account copied when provisioning. |
| Create accounts (`create_user`) | Boolean | Default true. Whether to provision an account for a directory user who has none locally. |
| Use transport security (`ldap_tls`) | Boolean | Request encrypted transport after connecting. Requires a server that offers the upgrade; otherwise every attempt fails. |

- **Default ordering**: sequence.
- **Display name**: the directory server address.

### 23.2 Connecting

The connection address is built as `ldap://` followed by the server address, a colon and the port.
Unless the parameter `auth_ldap.disable_chase_ref` is set to a false value (it defaults to true),
referral chasing is switched off. When transport security is requested, the upgrade is performed
immediately after connecting.

### 23.3 Finding an entry

1. Count the placeholders in the filter; a filter with none is logged as a warning.
2. Substitute the login into every placeholder, escaping it for the filter language.
3. Search the subtree of the search base with that filter, with a 60-second timeout, binding first
   with the query account (or anonymously when it is empty).
4. Discard results with no distinguished name.
5. Exactly one remaining result yields that result; zero or more than one yields nothing.

### 23.4 Verifying credentials

1. An empty password fails immediately. (This is deliberate: an anonymous bind with a valid
   distinguished name and an empty password would otherwise succeed and be mistaken for a valid
   sign-in.)
2. The entry is found; if there is none, failure.
3. A fresh connection is opened and a simple bind is attempted with the entry's distinguished name
   and the supplied password. Invalid credentials and any other directory error both yield failure
   (the latter is logged).
4. Success yields the directory entry.

### 23.5 Provisioning

1. The login is lower-cased and trimmed.
2. An account with that lower-cased login is looked up directly. If it exists and is active, it is
   used. If it exists and is archived, provisioning does **not** occur and the flow falls through to
   the refusal below.
3. Otherwise, if the binding permits creation, values are composed: the name from the entry's common
   name attribute, the login, the binding's company; and, when the login is a single valid electronic
   mail address, that address. If the binding names a template account, the template is copied with
   those values and forced active; otherwise a plain account is created. Creation is elevated and
   marked "no password mail".
4. If neither branch applies: *No local user found for LDAP login and not configured to create one*

### 23.6 Sign-in and password change

**Sign-in.** The ordinary sign-in runs first. If it refuses, and no account exists with that
lower-cased login at all, each binding in sequence order verifies the credentials; the first success
provisions or finds the account and reports the method *directory* with the default multi-factor
policy. If an account with that login does exist locally, the original refusal stands — the
directory is never consulted for a login that is already known locally.

**Credential check.** Similarly: the ordinary check runs first; on refusal, and provided the
credential is a password, the connection is interactive or the account does not require application
keys, and the account is active, each binding verifies the acting account's login and password.

**Password change.** Before the ordinary change, each binding is asked to change the password in the
directory: the entry is found, a bind is made with the old password, and the directory's password
change is invoked. The first binding that reports success causes the **local** password column to be
set to empty (so that the local copy cannot be used) and the change ends there. If no binding
succeeds, the ordinary local change proceeds.

### 23.7 Connection test

An action tests the configured binding by connecting and binding with the query account, and
reports:

| Outcome | Title | Message |
|---|---|---|
| Success | Connection Test Successful! | Successfully connected to directory access protocol server at *server*:*port* |
| Server unreachable | Connection Test Failed! | Cannot contact directory access protocol server at *server*:*port* |
| Invalid credentials | Connection Test Failed! | Invalid credentials for bind DN *the query account distinguished name* |
| Timeout | Connection Test Failed! | Connection to directory access protocol server at *server*:*port* timed out |
| Any other error | Connection Test Failed! | An error occurred: *the error* |

---

## 24. Sign-up fields on the Contact and the User

### 24.1 On the Contact

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sign-up token type (`signup_type`) | Text | Not copied. Restricted to the *Access Rights* group. Either `signup` (an invitation to create an account) or `reset` (an invitation to set a new password), or empty (no invitation outstanding). |

There is **no stored token**. The token is a signed payload computed on demand (see
[calculations.md](calculations.md) section 5), so it cannot be stolen from the database and it
expires by construction.

### 24.2 On the User

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Status (`state`) | Selection | Computed, not stored; searchable. `new` (*Invited*) when there is no sign-in date, `active` (*Confirmed*) otherwise. The search translates a query for *Confirmed* into "has at least one sign-in log entry" and a query for *Invited* into "has none". |

### 24.3 Side effects on the ordinary life cycle

- **Creation.** Unless the creation is marked "no password mail", every created account that has an
  electronic mail address is sent a sign-up invitation. If delivery fails, the invitation is
  cancelled on the Contact rather than left dangling.
- **Archiving.** Writing the active flag to false cancels any outstanding invitation on the Contact.
- **Deleting.** Deleting an account cancels the invitation on its Contact.
- **Duplication.** Duplicating an account without supplying an electronic mail address is done with
  the "no password mail" marker, so the copy does not mail the original address.

---

## 25. Portal entities

### 25.1 Portal Document Mixin

**Transport name** `portal.mixin` — abstract; its fields materialise on every entity that adopts it.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Portal Access Address (`access_url`) | Text | Computed, not stored. The customer-facing path of this document. The base implementation yields `#`; every adopting entity overrides it. |
| Security Token (`access_token`) | Text | Not copied. A random universally unique identifier, created on demand. Searchable only with the *is in* and *is not in* operators. |
| Access warning (`access_warning`) | Long text | Computed, not stored. A message shown to the recipient when the document is visible but something about the access is unusual. Empty in the base implementation. |

**Ensuring a token.** If the document has no token, one is generated as a random universally unique
identifier of version 4 and written with elevation (a plain assignment would not clear the cache
and the freshly written value would not be readable back). The token is returned.

**Building a share address.** Given optional flags:

1. If the *redirect* flag is set, the parameters carry the entity transport name and the record
   identifier, and the address is the generic mail-view path; otherwise the parameters start empty
   and the address is the document's own portal path.
2. If a token is wanted and the entity has the token field, the acting user's **read** access to the
   document is checked first, then the token is ensured and added to the parameters.
3. If a recipient contact identifier is given, it is added, together with a signature over that
   identifier (see [calculations.md](calculations.md) section 5.3), so the recipient can be
   recognised in the document's discussion thread.
4. If a sign-up invitation is wanted and the document has a contact, the contact's sign-up
   parameters are merged in.
5. The result is the address, a question mark, and the parameters encoded as a query string.

**Portal address helper.** A second helper composes an address from the document's portal path, an
optional suffix, the ensured token, an optional report kind, an optional download marker, an
optional extra query string and an optional fragment. The order is fixed: suffix, then the token,
then the report kind, then the download marker, then the extra query string, then the fragment.

**Access action.** When a document is opened on behalf of someone, the mixin decides between the
back-office form and the customer-facing page:

1. If an account is named and that account cannot read the document, the generic behaviour applies.
2. Otherwise the document is re-read as that account.
3. If the account is a shared user, or the caller demanded the customer-facing page: if the document
   cannot be read, then either an address action on the bare portal path is returned (when the
   customer-facing page was demanded) or the generic behaviour applies; if it can be read, an
   address action on the full share address is returned.
4. Otherwise the generic behaviour applies.

### 25.2 Portal Access Wizard

**Transport name** `portal.wizard` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Contacts (`partner_ids`) | Set of Contact | Default: the contacts of the acting selection, **expanded** — for each selected contact, itself plus every child of kind *contact* or *other*. |
| Users (`user_ids`) | Collection of Portal Access Wizard Line | Computed from the contacts, stored, writable: one line per contact, carrying that contact's electronic mail address. |
| Invitation Message (`welcome_message`) | Long text | Included in the invitation sent to newly granted contacts. |

### 25.3 Portal Access Wizard Line

**Transport name** `portal.wizard.user` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Wizard (`wizard_id`) | Link to Portal Access Wizard | **Required**, delete behaviour *cascade*. |
| Contact (`partner_id`) | Link to Contact | **Required, read-only**, delete behaviour *cascade*. |
| Electronic mail address (`email`) | Text | Editable; used both to create the account and to correct the contact. |
| User (`user_id`) | Link to User | Computed with elevation: the first account of the contact, archived ones included. |
| Latest Authentication (`login_date`) | Date and time | Mirror of the account's last sign-in. |
| Is Portal (`is_portal`) | Boolean | Computed: the account exists, is active, and is an external user. |
| Is Internal (`is_internal`) | Boolean | Computed: the account exists and is an internal user — **even if archived**, because an account that was internal must not be silently recycled as an external one. |
| Status (`email_state`) | Selection | Computed, default `ok`. `ok` (*Valid*), `ko` (*Invalid*), `exist` (*Already Registered*). |

**Status computation.** A line whose address does not normalise to a valid address is *Invalid*. For
the remainder, all accounts (archived included) whose login is one of the normalised addresses are
fetched; a line is *Already Registered* when one of them has exactly that login and is a **different**
account from the line's own; otherwise *Valid*.

**Granting.** See [workflows.md](workflows.md) section 9.

### 25.4 Portal Share Wizard

**Transport name** `portal.share` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Related Document Entity (`res_model`) | Text | **Required.** Defaulted from the acting context. |
| Related Document Identifier (`res_id`) | Integer | **Required.** Defaulted from the acting context. |
| Related Document (`resource_ref`) | Reference | Computed from the two above: a reference value combining them. |
| Recipients (`partner_ids`) | Set of Contact | **Required.** |
| Note (`note`) | Long text | Extra content included in the message. |
| Link (`share_link`) | Text | Computed: the absolute share address of the document, built with the *redirect* flag; empty when the document's entity does not adopt the portal mixin. |
| Access warning (`access_warning`) | Long text | Computed: the document's own access warning. |

**Sending.** The parameter `auth_signup.invitation_scope` decides the split:

1. If the document already has a token, or open sign-up is **not** enabled, every recipient gets the
   plain share message.
2. Otherwise only recipients that already have an account get the plain share message.
3. The remaining recipients each get an individual message carrying a **sign-up** address that
   redirects, after registration, to the document.

Each message is composed in the recipient's language, posted on the document as an internal note
with the light notification layout, with the subject *Invitation to access the document's display
name*, and addressed to that single recipient.

---

## 26. Onboarding entities

### 26.1 Onboarding Panel

**Transport name** `onboarding.onboarding` — **table** `onboarding_onboarding`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name of the onboarding (`name`) | Text | Translatable. |
| One word name (`route_name`) | Text | **Required**, unique. Defines the panel's path, `/onboarding/<one word name>`. Violation message: *Onboarding alias must be unique.* |
| Onboarding steps (`step_ids`) | Set of Onboarding Step | The ordered steps. |
| Message at completion (`text_completed`) | Text | Default *Nice work! Your configuration is done.* |
| Should be done per company? (`is_per_company`) | Boolean | Computed, **not stored**: true when any progress row carries a company **or** any step is per-company. Once true it stays true in practice, because existing per-company progress keeps it true even if the last per-company step is removed — this avoids having to merge progress rows. |
| Closing action (`panel_close_action_name`) | Text | The name of the operation to run when the panel is dismissed. |
| Onboarding Progress (`current_progress_id`) | Link to Onboarding Progress | Computed for the acting company. |
| Completion State (`current_onboarding_state`) | Selection | Computed, read-only. `not_done` (*Not done*), `just_done` (*Just done*), `done` (*Done*). |
| Was panel closed? (`is_onboarding_closed`) | Boolean | Computed from the current progress. |
| Progress rows (`progress_ids`) | Collection of Onboarding Progress | Read-only. All progress rows across companies. |
| Sequence (`sequence`) | Integer | Default 10. |

- **Default ordering**: sequence ascending, then identifier descending.
- **Current progress**: the progress row whose company is empty or the acting company. When there is
  none, the state is *Not done*, the link is empty and the panel counts as not closed.
- **Write**: if the set of steps changed, every progress row recomputes its step-progress links.

### 26.2 Onboarding Step

**Transport name** `onboarding.onboarding.step` — **table** `onboarding_onboarding_step`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Onboardings (`onboarding_ids`) | Set of Onboarding Panel | The panels this step appears in. |
| Title (`title`) | Text | Translatable. Also the display name. |
| Description (`description`) | Text | Translatable. |
| Button text (`button_text`) | Text | **Required**, translatable, default *Let's do it*. |
| Icon when completed (`done_icon`) | Text | Default `fa-star`. |
| Text when completed (`done_text`) | Text | Translatable, default *Step Completed!* |
| Step Image (`step_image`) | Binary | Falls back to a shipped placeholder image. |
| Step Image Filename (`step_image_filename`) | Text | |
| Alternative text for the image (`step_image_alt`) | Text | Translatable, default *Onboarding Step Image*. |
| Opening action (`panel_step_open_action_name`) | Text | The operation run when the step is started. |
| Step Progress (`current_progress_step_id`) | Link to Onboarding Step Progress | Computed for the acting company. |
| Completion State (`current_step_state`) | Selection | Computed. Same three values. |
| Progress rows (`progress_ids`) | Collection of Onboarding Step Progress | Read-only. |
| Is per company (`is_per_company`) | Boolean | Default true. |
| Sequence (`sequence`) | Integer | Default 10. |

- **Default ordering**: sequence ascending, then identifier ascending.
- **Constraint**: a step linked to any panel must have an opening action, otherwise

  > An "Opening Action" is required for the following steps to be linked to an onboarding panel:
  > *the list of titles*

- **Write**: changing the per-company flag deletes every progress row of the affected steps; then
  every linked panel refreshes its progress rows; then, if new panels were linked, their progress
  rows recompute their step links.

### 26.3 Onboarding Progress

**Transport name** `onboarding.progress` — **table** `onboarding_progress`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Onboarding progress (`onboarding_state`) | Selection | Computed, **stored**. `done` when the number of step-progress rows in state *just done* or *done* equals the number of steps of the panel; `not_done` otherwise. |
| Was panel closed? (`is_onboarding_closed`) | Boolean | |
| Company (`company_id`) | Link to Company | Delete behaviour *cascade*. Empty for a panel that is not per-company. |
| Related onboarding (`onboarding_id`) | Link to Onboarding Panel | **Required**, indexed, delete behaviour *cascade*. |
| Progress Steps (`progress_step_ids`) | Set of Onboarding Step Progress | |

- **Uniqueness**: a unique index on (panel, company treated as 0 when empty). A plain constraint
  cannot express this, hence the index.

### 26.4 Onboarding Step Progress

**Transport name** `onboarding.progress.step` — **table** `onboarding_progress_step`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Related progress trackers (`progress_ids`) | Set of Onboarding Progress | |
| Step Progress (`step_state`) | Selection | Default `not_done`. Same three values. |
| Step (`step_id`) | Link to Onboarding Step | **Required**, indexed, delete behaviour *cascade*. |
| Company (`company_id`) | Link to Company | Delete behaviour *cascade*. |

- **Uniqueness**: a unique index on (step, company treated as 0 when empty).
- **Display name**: the step.

The three-state cycle is specified in [state-machines.md](state-machines.md) section 5.

---

## 27. Privacy entities

### 27.1 Privacy Log

**Transport name** `privacy.log` — **table** `privacy_log`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Date (`date`) | Date and time | **Required**, default now. |
| Anonymised name (`anonymized_name`) | Text | **Required.** Stored already masked — see below. |
| Anonymised address (`anonymized_email`) | Text | **Required.** Stored already masked. |
| Handled By (`user_id`) | Link to User | **Required**, default the acting user. |
| Execution details (`execution_details`) | Long text | One line per action taken on a found record. |
| Found Records (`records_description`) | Long text | One line per entity, giving the entity label, the number of records and their identifiers. |
| Additional note (`additional_note`) | Long text | Free text. |

- **Display name**: the handling account.

**Masking on creation.** The log exists to prove that a request was handled; it must not itself
become a second copy of the personal data. Both values are therefore masked before being stored.

- A name that contains an at-sign is masked as an address (below). Otherwise each space-separated
  word is reduced to its first character followed by as many asterisks as the remaining characters,
  and the words are rejoined with spaces. Empty words are dropped.
- An address is split at the at-sign. The local part is split on full stops and each piece is
  reduced to its first character plus asterisks, rejoined with full stops. The domain is left
  untouched when it is one of the three most common consumer domains (`gmail.com`, `hotmail.com`,
  `yahoo.com`); otherwise every label except the last is reduced the same way and the last label is
  kept intact.
- A value with no at-sign presented as an address yields an error value carrying
  *This email address is not valid (the value)*.

Worked example: the name `Marie Claire Dubois` becomes `M***** C****** D*****`; the address
`marie.dubois@acme-corp.example` becomes `m****.d*****@a*********.e******`; the address
`marie@gmail.com` becomes `m****@gmail.com`.

### 27.2 Privacy Search Wizard

**Transport name** `privacy.lookup.wizard` — transient, never discarded by count, discarded after 24
hours.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Text | **Required.** The person's name. |
| Electronic mail address (`email`) | Text | **Required.** |
| Lines (`line_ids`) | Collection of Privacy Search Line | The found records. |
| Execution details (`execution_details`) | Long text | Computed from the lines, **stored**: the non-empty execution details of the lines, one per line. Computing it also writes the log (below). |
| Log (`log_id`) | Link to Privacy Log | |
| Found Records (`records_description`) | Long text | Computed: for each entity, its label (followed by a space, a hyphen, a space and the transport name when the acting user holds the technical-features group), the number of found records in parentheses, a colon, and the identifiers each preceded by a number sign, comma-separated. One entity per line. |
| Number of lines (`line_count`) | Integer | Computed. |

- **Display name**: the fixed text *Privacy Lookup*.
- **Log writing**: whenever the execution details are recomputed, a log is created if there is none
  and the details are non-empty; otherwise the existing log's details and record description are
  overwritten.

The search itself is specified in [calculations.md](calculations.md) section 11.

### 27.3 Privacy Search Line

**Transport name** `privacy.lookup.wizard.line` — transient, never discarded by count, discarded
after 24 hours.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Wizard (`wizard_id`) | Link to Privacy Search Wizard | |
| Resource identifier (`res_id`) | Integer | **Required.** |
| Resource name (`res_name`) | Text | Computed, **stored**: the found record's display name, or *the entity label/the identifier* when the display name is empty. Skipped when the record no longer exists. |
| Related Document Entity (`res_model_id`) | Link to Entity definition | Delete behaviour *cascade*. |
| Document Entity (`res_model`) | Text | Mirror of the entity's transport name, stored, read-only. |
| Record (`resource_ref`) | Reference | Computed, writable: a reference to the found record, but **only if the acting user may read it** — a record hidden by a multi-company rule yields an empty reference rather than an error. Empty once the record has been deleted. |
| Has an archive flag (`has_active`) | Boolean | Computed, stored: whether the found entity supports archiving. |
| Is active (`is_active`) | Boolean | The current archive state; changing it archives or un-archives the found record. |
| Is deleted (`is_unlinked`) | Boolean | Set once the found record has been deleted through this line. |
| Execution details (`execution_details`) | Text | Default empty. One sentence describing what was done. |

**Actions.**

- Toggling *is active* writes the flag on the found record with elevation and records the detail as
  *Archived the entity label #the identifier* or *Unarchived the entity label #the identifier*.
- Deleting a line's record deletes it with elevation, records *Deleted the entity label #the
  identifier*, and marks the line deleted. Deleting an already-deleted line raises *The record is
  already unlinked.*
- Bulk archive skips lines with no archive flag and lines already archived.
- Bulk delete skips lines already deleted.

---

## 28. Data-recycling entities

### 28.1 Recycling Rule

**Transport name** `data_recycle.model` — **table** `data_recycle_model`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | Boolean | Default true. Deactivating a rule deletes every outstanding candidate it produced. |
| Name (`name`) | Text | **Required**, computed from the entity, stored, writable, copied. Defaults to the entity's label when empty. |
| Entity (`res_model_id`) | Link to Entity definition | **Required**, delete behaviour *cascade*. |
| Entity name (`res_model_name`) | Text | Mirror of the entity's transport name, stored, read-only. |
| Candidates (`recycle_record_ids`) | Collection of Recycling Candidate | |
| Recycle Mode (`recycle_mode`) | Selection | **Required**, default `manual`. `manual` (*Manual*) — candidates are listed for a human to validate; `automatic` (*Automatic*) — candidates are validated as soon as they are collected. |
| Recycle Action (`recycle_action`) | Selection | **Required**, default `unlink`. `archive` (*Archive*) or `unlink` (*Delete*). |
| Filter (`domain`) | Text | Computed from the entity, stored, writable; reset to the empty filter when the entity changes. |
| Time Field (`time_field_id`) | Link to Field definition | Restricted in pickers to stored date or date-and-time fields of the chosen entity. Delete behaviour *cascade*. |
| Delta (`time_field_delta`) | Integer | Default 1. |
| Delta Unit (`time_field_delta_unit`) | Selection | Default `months`. `days` (*Days*), `weeks` (*Weeks*), `months` (*Months*), `years` (*Years*). |
| Include archived (`include_archived`) | Boolean | When set, archived records are considered too. |
| Records To Recycle (`records_to_recycle_count`) | Integer | Computed: the number of outstanding candidates. |
| Notify Users (`notify_user_ids`) | Set of User | Default the acting user. Restricted in pickers to accounts whose closure contains the *Role / Administrator* group. |
| Notify (`notify_frequency`) | Integer | Default 1. Must be strictly positive; violation message *The notification frequency should be greater than 0*. |
| Notify Frequency Period (`notify_frequency_period`) | Selection | Default `weeks`. `days` (*Days*), `weeks` (*Weeks*), `months` (*Months*). |
| Last notification (`last_notification`) | Date and time | Read-only. |

- **Default ordering**: name.
- **Constraint**: choosing the *Archive* action on an entity that has no archive flag is refused:
  *This model doesn't manage archived records. Only deletion is possible.*

The collection algorithm is in [calculations.md](calculations.md) section 12 and the whole flow in
[workflows.md](workflows.md) section 14.

### 28.2 Recycling Candidate

**Transport name** `data_recycle.record` — **table** `data_recycle_record`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | Boolean | Default true. Discarding a candidate clears it, so the candidate is remembered and not proposed again. |
| Record Name (`name`) | Text | Computed with elevation: the display name of the pointed-at record, or *Undefined Name* when the display name is empty, or *\*\*Record Deleted\*\** when the record no longer exists. |
| Recycle Rule (`recycle_model_id`) | Link to Recycling Rule | Indexed when not empty, delete behaviour *cascade*. |
| Record identifier (`res_id`) | Integer | Indexed, never aggregated. |
| Entity (`res_model_id`) | Link to Entity definition | Mirror of the rule's entity, stored, read-only. |
| Entity name (`res_model_name`) | Text | Mirror of the rule's entity name, stored, read-only. |
| Company (`company_id`) | Link to Company | Computed from the pointed-at record, **stored**: that record's company when it has one, otherwise empty. |

**Validating** a set of candidates: the pointed-at records are grouped by entity; those whose rule
says *Archive* are archived with elevation, those whose rule says *Delete* are deleted with
elevation; candidates whose record no longer exists are simply consumed; finally every processed
candidate row is deleted.

**Discarding** clears the active flag.

---

## 29. Indicator Provider

**Transport name** `kpi.provider` — abstract.

A contract, not a table. An implementation returns a list of indicator descriptions, each carrying
an identifier, a label, a numeric value and optionally a threshold and a target screen. The setup
dashboard aggregates every implementation's list. It exists in this domain because the first
implementation counts users and companies.

---

## 30. Fields this domain adds to entities owned elsewhere

| Entity | Field | Added by | Purpose |
|---|---|---|---|
| Contact (`res.partner`) | Sign-up token type (`signup_type`) | sign-up | Section 24.1. |
| Access Group (`res.groups`) | Session timeout (`lock_timeout`), Require second factor on session timeout (`lock_timeout_mfa`), Inactivity timeout (`lock_timeout_inactivity`), Require second factor on inactivity timeout (`lock_timeout_inactivity_mfa`) and six presentation companions | session timeout | Section 31. |
| User (`res.users`) | Second-factor fields | second factor | Section 20. |
| User (`res.users`) | Passkeys | passkeys | Section 21.2. |
| User (`res.users`) | Delegated-sign-in fields | delegated sign-in | Section 22.2. |
| User (`res.users`) | Status (`state`) | sign-up | Section 24.2. |
| Company (`res.company`) | Directory configurations (`ldaps`) | directory sign-in | The collection of bindings owned by the company. |
| Attachment (`ir.attachment`) | — | portal | The token-based read path, specified in [interfaces.md](interfaces.md). |

---

## 31. Session-timeout fields on the Access Group

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Session timeout (`lock_timeout`) | Integer | Minutes after which re-authentication is required regardless of activity. Zero or empty means no such limit. |
| Require second factor on session timeout (`lock_timeout_mfa`) | Boolean | |
| Inactivity timeout (`lock_timeout_inactivity`) | Integer | Minutes of inactivity after which re-authentication is required. Zero or empty means no such limit. |
| Require second factor on inactivity timeout (`lock_timeout_inactivity_mfa`) | Boolean | |
| Has a session timeout (`has_lock_timeout`) | Boolean | Computed, writable, not stored. Presentation only. Turning it on proposes 1440 minutes with the second factor **required**; turning it off clears both. |
| Session timeout unit (`lock_timeout_delay_unit`) | Selection | Computed, writable, not stored. `minutes`, `hours`, `days`. |
| Session timeout in that unit (`lock_timeout_delay_in_unit`) | Integer | Computed, writable, not stored. |
| Session timeout behaviour (`lock_timeout_2fa_selection`) | Selection | Computed with an inverse. `without_2fa` (*Logout*), `with_2fa` (*Logout with two-factor authentication*). |
| Has an inactivity timeout (`has_lock_timeout_inactivity`) | Boolean | Computed, writable, not stored. Turning it on proposes 15 minutes with the second factor **not** required; turning it off clears both. |
| Inactivity timeout unit (`lock_timeout_inactivity_delay_unit`) | Selection | As above. |
| Inactivity timeout in that unit (`lock_timeout_inactivity_delay_in_unit`) | Integer | As above. |
| Inactivity timeout behaviour (`lock_timeout_inactivity_2fa_selection`) | Selection | `without_2fa` (*Screen lock*), `with_2fa` (*Screen lock with two-factor authentication*). |

**Unit rendering.** A number of minutes is rendered in the largest exact unit: a multiple of 1440
becomes that many *days*; otherwise a multiple of 60 becomes that many *hours*; otherwise it stays
in *minutes*. Zero or empty renders as zero *minutes*. The reverse multiplies by 1440, by 60, or by
1.

**Cache.** Creating, writing or deleting a group whose timeout fields are involved clears the
registry caches, because the per-user timeout summary is cached.

**Per-user summary.** See [calculations.md](calculations.md) section 10.
