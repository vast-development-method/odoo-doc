# Identity and Access

This domain specifies **who** may use the system, **what** they may see and change, and **how** they
prove who they are. It is the foundation on which every other domain rests: no record is read, no
document is confirmed and no journal entry is posted without first passing through the checks
described here.

Three distinct mechanisms are specified, and they must not be confused with one another:

1. **Authentication** — establishing that the person or program at the other end of a connection is
   a particular user record. Password, second factor, passkey, delegated sign-in through an external
   identity provider, directory sign-in against a central directory server, application key on a
   programmatic connection, and the signed token that lets a person with no account at all read one
   specific document.
2. **Authorisation on a model** — deciding whether a user may read, create, modify or delete
   *records of a given entity at all*. This is the access-right table, keyed by entity and group.
3. **Authorisation on a record** — deciding *which* records of an entity a user may touch. This is
   the record-rule mechanism, whose combination semantics (intersection of global rules, union of
   group rules) is one of the two or three most consequential algorithms in the whole system.

On top of these sit the surrounding services that the same packages provide: the company structure
and the active-company selection that filters nearly every list in the application; the settings
mechanism that turns a checkbox on a configuration screen into a group membership, a stored
parameter, a user-defined default value or a package installation; user-defined default values
themselves; the external-party portal; the guided setup panels; the privacy search that finds every
record mentioning a person; and the recycling rules that periodically propose stale records for
deletion or merging.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| User accounts | One user record per login, backed by a contact record that carries the name, language, time zone, image and addresses. Internal, external (portal) and anonymous (public) kinds. Archival, deletion queue, self-service field access. |
| Groups and privileges | Named sets of users. A group may *imply* other groups; membership is the reflexive transitive closure of implication. Groups are presented to administrators grouped under a privilege (a named choice such as *Sales / Administrator*) inside an application category. |
| Access rights | Per entity, per group, four independent permission flags (read, create, modify, delete). Permissions are additive: a user holding any group that grants the operation on the entity may perform it. |
| Record rules | Per entity, per operation, a filter expression that a record must satisfy. Rules attached to no group are *global* and are combined by intersection; rules attached to groups the user holds are combined by union, and the union is then intersected with the global part. |
| Field-level restriction | A field may declare the groups required to read or write it; a field may also be declared permanently unreadable. |
| Privilege elevation | A well-defined escalation mode that suspends both access-right and record-rule checks for a nested piece of work, and a separate mechanism for running work *as* another user. |
| Companies | A tree of companies with branches. Each user has one default company and a set of permitted companies; the *active* set is carried on every call and drives both the default company of new records and the standard multi-company record rule. |
| Configuration settings | A non-persistent screen whose fields are reflected into group memberships, stored parameters, user-defined defaults and package installation requests when it is saved. |
| Default values | Administrator-defined or user-defined default field values, scoped by user, by company and by an optional condition, with a documented precedence order. |
| Password handling | Salted, iterated one-way hashing with automatic re-hashing on sign-in when the stored hash is out of date; self-service change; administrator-driven change; strength policy with a minimum length and a minimum estimated-guesses figure. |
| Second factor | Time-based one-time codes from an authenticator application, with a rate limiter, trusted-device registration, recovery codes, and a fallback that mails a code. |
| Passkeys | Public-key credentials registered against the server's origin; sign-in without a password, and second-factor suppression when a passkey was used. |
| Delegated sign-in | Sign-in through an external identity provider that returns a bearer token; provisioning of a user record on first use when sign-up is open. |
| Directory sign-in | Sign-in verified against a central directory server, with optional provisioning from a template user and per-company directory configurations. |
| Sign-up and password reset | Tokenised invitations attached to a contact, self-service registration, password-reset mails, and the template user that decides the groups of self-registered accounts. |
| Session lifetime | Global inactivity limits per group and per operation kind, a re-authentication dialogue that accepts several methods, and a lock screen. |
| Application keys | Long random secrets stored only as hashes, scoped, expiring, used in place of a password on non-interactive connections. |
| Devices and sessions | A record of every device that has held a session, with revocation. |
| External-party portal | The customer-facing pages, the portal mixin that gives a business document a shareable address and a signed token, the grant/revoke wizard, and the share wizard. |
| Guided setup | Reusable onboarding panels made of steps, with per-company progress tracking. |
| Privacy search | A search across every entity that declares a personal-data field, producing a log and offering anonymisation or deletion. |
| Data recycling | Rules that periodically collect records matching a filter (typically "older than N days") or duplicate groups, and propose them for deletion or merging. |

---

## 2. Entities of the domain

### 2.1 Owned by the foundation package

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| User | `res.users` | `res_users` | One account able to sign in; delegates its personal data to a Contact. |
| User Sign-in Log | `res.users.log` | `res_users_log` | One row per successful sign-in, used to derive the last sign-in date. |
| User Settings | `res.users.settings` | `res_users_settings` | Per-user client-side preferences, one row per internal user. |
| User Deletion Request | `res.users.deletion` | `res_users_deletion` | Queue of self-service account removals, processed by a scheduled job. |
| Device | `res.device` | `res_device` | One row per (session, device) pair that has been seen, with revocation. |
| Device Log | `res.device.log` | `res_device_log` | Append-only trace of device activity from which Device rows are derived. |
| Access Group | `res.groups` | `res_groups` | A named set of users; carries access rights, record rules, menus and views. |
| Group Privilege | `res.groups.privilege` | `res_groups_privilege` | A named choice grouping mutually-ordered groups for presentation. |
| Application Category | `ir.module.category` | `ir_module_category` | Top level of the presentation hierarchy above privileges. |
| Access Right | `ir.model.access` | `ir_model_access` | One (entity, group) row carrying the four permission flags. |
| Record Rule | `ir.rule` | `ir_rule` | A filter expression restricting which records of an entity an operation may touch. |
| Default Value | `ir.default` | `ir_default` | A stored default for one field, scoped by user, company and condition. |
| System Parameter | `ir.config_parameter` | `ir_config_parameter` | A global key/value setting. |
| Company | `res.company` | `res_company` | A legal or organisational unit; companies form a tree of branches. |
| Application Key | `res.users.apikeys` | `res_users_apikeys` | A hashed long-lived secret usable instead of a password on programmatic connections. |
| Application Key Description Wizard | `res.users.apikeys.description` | transient | Collects the label and validity of a new application key. |
| Application Key Display | `res.users.apikeys.show` | abstract | Presents a newly generated key once. |
| Identity Check Wizard | `res.users.identitycheck` | transient | Re-verifies the current user's credentials before a sensitive action. |
| Change Password Wizard | `change.password.wizard` | transient | Administrator-driven password change for a selection of users. |
| Change Password Line | `change.password.user` | transient | One user line of the administrator password wizard. |
| Change Own Password Wizard | `change.password.own` | transient | Self-service password change with confirmation. |
| Configuration Settings | `res.config.settings` | transient | The base of every settings screen; see the settings mechanism. |
| Configuration Item | `res.config` | transient | The base of the older step-by-step configuration items. |

### 2.2 Owned by the authentication packages

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Authentication Device | `auth_totp.device` | shares `res_users_apikeys` | A trusted browser that may skip the second factor, stored as a scoped application key. |
| Second-factor Rate Limit Log | `auth.totp.rate.limit.log` | `auth_totp_rate_limit_log` | One row per throttled event (code attempt or mail send) for rate limiting. |
| Second-factor Setup Wizard | `auth_totp.wizard` | transient | Generates the shared secret, shows the enrolment image and verifies the first code. |
| Passkey | `auth.passkey.key` | `auth_passkey_key` | A registered public-key credential for one user. |
| Passkey Creation Wizard | `auth.passkey.key.create` | transient | Collects the label of a new passkey before registration. |
| Delegated Sign-in Provider | `auth.oauth.provider` | `auth_oauth_provider` | An external identity provider configuration. |
| Directory Configuration | `res.company.ldap` | `res_company_ldap` | One directory server binding, owned by a company. |

### 2.3 Owned by the portal, onboarding, privacy and recycling packages

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Portal Document Mixin | `portal.mixin` | abstract | Gives a business document a customer-facing address and a signed access token. |
| Portal Access Wizard | `portal.wizard` | transient | Grants or revokes portal access for a set of contacts. |
| Portal Access Wizard Line | `portal.wizard.user` | transient | One contact line of the portal access wizard. |
| Portal Share Wizard | `portal.share` | transient | Sends a tokenised link to a document to a set of recipients. |
| Onboarding Panel | `onboarding.onboarding` | `onboarding_onboarding` | A named guided-setup panel made of ordered steps. |
| Onboarding Step | `onboarding.onboarding.step` | `onboarding_onboarding_step` | One action of a guided-setup panel. |
| Onboarding Progress | `onboarding.progress` | `onboarding_progress` | Per-company completion state of one panel. |
| Onboarding Step Progress | `onboarding.progress.step` | `onboarding_progress_step` | Per-company completion state of one step. |
| Privacy Log | `privacy.log` | `privacy_log` | The record of one personal-data search and what was done with the result. |
| Privacy Search Wizard | `privacy.lookup.wizard` | transient | Runs the personal-data search and offers anonymisation or deletion. |
| Privacy Search Line | `privacy.lookup.wizard.line` | transient | One found record in a personal-data search. |
| Recycling Rule | `data_recycle.model` | `data_recycle_model` | A periodic rule that collects records to delete or merge. |
| Recycling Candidate | `data_recycle.record` | `data_recycle_record` | One record proposed by a recycling rule, awaiting validation or discard. |
| Indicator Provider | `kpi.provider` | abstract | Contract for the small numeric indicators shown on the setup dashboard. |

### 2.4 Extended, not owned

The domain adds fields and behaviour to entities owned elsewhere: the Contact (`res.partner`) gains
the sign-up token and expiry, the portal access flag and the personal-data search hooks; the
Attachment (`ir.attachment`) gains the token-based read path; every business document that becomes
portal-visible gains the mixin fields; the Mail Template entity carries the invitation, reset,
second-factor and portal messages.

---

## 3. Reading order

1. **`entities.md`** — every entity, field by field. Read at least the User, Access Group, Access
   Right, Record Rule and Company sections before anything else.
2. **`business-rules.md`** — the access-checking algorithm, the rule-combination semantics, the
   privilege-elevation semantics, and every validation with its exact message. This is the heart of
   the domain.
3. **`calculations.md`** — the transitive closure of group implication, the one-time-code arithmetic,
   the password-strength estimate, the token and hash computations, the rate-limit arithmetic, the
   recycling date arithmetic.
4. **`state-machines.md`** — the sign-in state machine, the second-factor enrolment states, the
   recycling candidate states, the onboarding step states, the user-deletion queue states.
5. **`workflows.md`** — the end-to-end procedures: signing in, switching company, granting portal
   access, enrolling a second factor, resetting a password, running a privacy search, running a
   recycling rule.
6. **`configuration.md`** — the settings mechanism, the shipped groups, the access-right matrix, the
   shipped record rules, the system parameters, the scheduled jobs.
7. **`interfaces.md`** — menus, views, routes, named remote operations, mail templates.
8. **`acceptance-criteria.md`** — the numbered scenarios that a re-implementation must satisfy.
9. **`accounting-effects.md`** — short: this domain produces no journal entries.
10. **`glossary.md`** — every term.

---

## 4. Dependencies on other domains

| Depends on | For what |
|---|---|
| Messaging and activities | Mail templates and message delivery for invitations, password resets, mailed second-factor codes and portal notifications; the discussion thread on portal documents. |
| Multi-currency | The currency carried by a company. |
| Website and storefront | The anonymous user and the multi-website resolution reuse the public user and the portal routes specified here. |

| Depended on by | For what |
|---|---|
| Every other domain | The access-right rows and record rules that each domain ships are instances of the mechanisms specified here; every domain's "security groups" section is an application of the group model. |
| Sales, purchasing, accounts receivable, projects | The portal mixin, the access token and the customer-facing document pages. |
| All financial domains | The company tree, the active-company set and the standard multi-company record rule. |

---

## 5. What this domain does **not** cover

- Deployment, hosting, worker processes and network configuration.
- The message and notification machinery itself (specified in messaging and activities).
- The storefront account pages beyond the portal base (specified in website and storefront).
- Automation rules and server actions (specified in automation and integration), although the
  privilege-elevation semantics they rely on are specified here.
