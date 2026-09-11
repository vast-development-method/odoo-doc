# Identity and Access — Glossary

Every term used in this domain, defined in full. Terms are listed alphabetically. Reproduced
identifiers appear in code font with their full name in words.

---

**Access check** — The four-layer decision that determines whether an operation may be performed:
the entity layer, the record layer, the field layer and the company-consistency layer. Specified as
a numbered procedure in [business-rules.md](business-rules.md) section 2.

**Access group** — See *group*.

**Access right** — One row of the table `ir.model.access` (the model-access entity), stating that
members of one group may perform some of the four operations on records of one entity. Rows are
additive; absence of a row means refusal. Transport name `ir.model.access`, storage
`ir_model_access`.

**Access token** — The random value stored on a customer-facing document that lets a person with no
account read that one document. A version-4 random universally unique identifier, generated on
first use, compared in constant time, never copied on duplication. Field `access_token` (the
security-token field) of the portal document mixin.

**Accessible branches** — For a company and the currently active companies, the set of companies
obtained by walking the company's subtree and keeping those that are also active; falling back to
the company itself when the acting account is the account of identifier 1 and the intersection is
empty.

**Account** — See *user*.

**Acting user** — The account on whose behalf the current piece of work runs. It is the account
whose groups, companies, language and time zone are consulted. Privilege elevation does not change
it; acting as another user does.

**Active companies** — The ordered list of company identifiers carried in the call's context under
the key `allowed_company_ids` (the permitted-company-identifiers key). Its first element is the
*current company*; the whole list is what the standard multi-company record rule compares against.
It is validated against the acting user's permitted companies on every read.

**Administrator** — An account that is either the account of identifier 1 or a member of the
*Access Rights* group. Distinct from a *system user*, which is a member of *Role / Administrator*.

**Anonymous account** — The account under which a visitor with no session browses public pages. It
holds *Role / Public*. The shipped one is archived; each company can mint its own.

**Application category** — The top level of the presentation hierarchy above privileges, used to
group the selectors on the account form. Transport name `ir.module.category`.

**Application key** — A long random secret usable instead of a password on a programmatic
connection. Stored only as a hash plus an 8-character index. May be scoped and normally expires.
Transport name `res.users.apikeys`, storage `res_users_apikeys`.

**Archive filter** — The default restriction that hides records whose active flag is clear. It is
deliberately **disabled** during the record-level access check, so that an archived record is
reported as absent rather than as forbidden.

**Authentication** — Establishing which account is at the other end of a connection. Distinct from
*authorisation*.

**Authentication result** — The structured value a successful credential check returns: the account
identifier, the method used, and the second-factor policy (`default`, `skip` or `enforce`).

**Authorisation** — Deciding what an established account may do. Split into the entity layer, the
record layer and the field layer.

**Bearer mode** — The authentication mode of a path that accepts an application key in an
`Authorization` header of the form `bearer <token>`, and that additionally requires
browser-navigation markers when no header is present.

**Branch** — A company whose parent is another company. Branches inherit the *root-delegated
fields* of their root.

**Closure** — See *group closure*.

**Company** — A legal or organisational unit. Companies form a tree. Transport name `res.company`,
storage `res_company`.

**Company-agreement filter** — The filter an entity supplies to say which of its records agree with
a given set of companies. The default is "the target's company is one of these, or empty"; the
Company entity and the User entity override it.

**Company consistency** — The check that the companies of a record's linked records agree with the
record's own company. Not a permission check: it is a constraint, and privilege elevation does not
suspend it.

**Constant-time comparison** — A comparison of two values whose duration does not depend on the
position of the first differing byte. Used for the session token, the signed-payload signature, the
access token and the recipient signature.

**Cooldown** — The linear rate limit applied to sign-in attempts, keyed on the remote network
address and held in the process rather than the database.

**Credential** — An untrusted structured value presented at authentication, carrying a type and the
data that type needs. The defined types are `password`, `totp` (a one-time code from an
authenticator application), `totp_mail` (a mailed one-time code), `webauthn` (a passkey assertion)
and `oauth_token` (a token from an external identity provider).

**Current company** — The first element of the active-company list, or the acting user's default
company when the list is empty. The company of records created in that call.

**Default value** — A stored proposal for one field of one entity, scoped by user, company and an
optional condition. Transport name `ir.default`, storage `ir_default`.

**Delegated sign-in** — Signing in through an external identity provider that returns a bearer
token, which the server exchanges for a subject identity.

**Delegation** — The mechanism by which one entity presents another entity's fields as its own. The
User delegates to the Contact; the Company delegates to the Contact. Record rules on the delegated
entity apply to the delegating one.

**Deny by default** — The property that an entity with no access-right row is forbidden to everyone
who is not privilege-elevated.

**Device** — A (session, platform, browser) combination that has held a session for an account.
Derived as a view over the device log. Transport names `res.device` and `res.device.log`.

**Directory sign-in** — Verifying credentials against a central directory server, and optionally
provisioning an account from a template. Configured by the Directory Configuration entity,
transport name `res.company.ldap`.

**Disjoint groups** — Groups that no single account's closure may contain together. The three kind
groups — *Role / User*, *Role / Portal*, *Role / Public* — are mutually disjoint.

**Elevation** — See *privilege elevation*.

**Entity-level check** — The first layer of the access check: whether the acting user holds the
operation on the entity at all.

**Evaluation context of a rule** — The exactly three names available while a record rule's filter
is evaluated: the acting user (with an **empty** context), the active company identifiers, and the
first of them.

**External user** — An account holding *Role / Portal*. Sees only the customer-facing pages and only
its own documents. Its derived *Share User* flag is true.

**Failing rule** — A rule identified by the diagnosis procedure as a cause of a record-level
refusal. Grants fail as a group; global rules fail individually.

**Field-level restriction** — A group requirement attached to a field, or the *never accessible*
marker. Checked on read and on write.

**Filter** — A structured condition over an entity's fields. Record rules carry filters; the
combination of the applicable rules' filters is itself a filter.

**Finalising a session** — Converting a *pending* session into an *established* one: popping the
pending login and account, computing the preference context and the session token, and marking the
session for rotation.

**Global rule** — A record rule with no group. Global rules are combined by **intersection**: every
one of them must be satisfied.

**Grant rule** — A record rule with at least one group. Grants whose groups the acting user holds
are combined by **union**: satisfying any one suffices.

**Group** — A named set of users; the only currency in which permission is expressed. Groups form a
directed graph through implication. Transport name `res.groups`, storage `res_groups`.

**Group closure** — The reflexive transitive closure of implication applied to an account's
explicitly assigned groups. Every access decision is taken against the closure, never against the
explicit groups.

**Group definition** — The compiled, cached representation of the implication graph, answering
identifier lookup, superset closure and subset closure.

**Group expression** — The coarse summary of who may perform an operation on an entity: the empty
expression, the universe expression, or an expression built from the granting rows' groups.

**Guarded operation** — An operation wrapped by the identity re-check.

**Hashing context** — The configured set of password schemes and their parameters: the iterated
derivation scheme with at least 600 000 iterations first, the plain-text scheme second and
deprecated.

**Identity re-check** — The requirement to re-confirm one's credentials before a sensitive
operation, unless the last confirmation is newer than ten minutes. The suspended call is serialised
into a wizard record and replayed on success. Transport name `res.users.identitycheck`.

**Implication** — The relation between two groups meaning "every member of the first is also a
member of the second". Written as a set inclusion: the first group is a subset of the second.

**Implied group** — In the settings mechanism, the group that a checkbox adds to or removes from the
implications of its holder groups.

**Inactivity timeout** — The number of minutes of inactivity after which re-authentication is
demanded, configured per group and summarised per user.

**Index (of an application key)** — The first 8 hexadecimal characters of a clear key, stored in
clear so verification can narrow to a handful of rows.

**Internal user** — An account holding *Role / User*. Sees the back office. Its derived *Share User*
flag is false.

**Invitation** — An outstanding sign-up or reset offer, represented by the contact's sign-up type
plus a signed token computed on demand.

**Invitation scope** — The parameter `auth_signup.invitation_scope` (the registration-openness
parameter), with the values `b2b` (invited contacts only) and `b2c` (anyone).

**Kind group** — One of the three mutually disjoint groups *Role / User*, *Role / Portal* and
*Role / Public*, which determine whether an account is internal, external or anonymous.

**Last accepted counter** — The one-time-code counter most recently accepted for an account. A code
whose counter is not strictly greater is refused as a replay.

**Mailed second factor** — A second-factor method in which the code is derived from a key that
depends on the account and its last sign-in and is mailed to the account, with a one-hour step and
a one-hour acceptance window.

**Never accessible marker** — A field restriction that no group can pass; the field is reachable
only through privilege elevation.

**Onboarding panel** — A guided-setup panel made of ordered steps, with per-company progress
tracking. Transport names `onboarding.onboarding`, `onboarding.onboarding.step`,
`onboarding.progress`, `onboarding.progress.step`.

**Operation** — One of exactly four names: read, create, write (meaning modify) and unlink (meaning
delete).

**Passkey** — A registered public-key credential bound to the server's origin. Signing in with one
skips the second factor. Transport name `auth.passkey.key`, storage `auth_passkey_key`.

**Password policy** — The strength requirement applied on every password write. The foundation
policy is a minimum length; a length of 0 disables it.

**Pending session** — A session in which the first factor succeeded but the second has not. It holds
a pending login and a pending account and no account identifier.

**Permitted companies** — The set of active companies an account may work in, computed by search and
cached per account.

**Permitted-entity set** — For an acting user and an operation, the set of entity names at least one
active access-right row permits.

**Personal-data search** — The composite search that finds every record mentioning a person by name
or electronic mail address, across every entity that exposes such a field or links to contacts.
Transport names `privacy.lookup.wizard`, `privacy.lookup.wizard.line`, `privacy.log`.

**Portal** — The customer-facing area, reached at `/my`. Also used as an adjective for external
users.

**Portal document mixin** — The contract that gives a business document a customer-facing address,
an access token and a share-address builder. Transport name `portal.mixin`.

**Preference context** — The pair (language, time zone) plus the acting account identifier, stored
in the session at finalisation and re-derived when needed.

**Privilege** — A named choice grouping mutually-ordered groups for presentation, sitting under an
application category. Transport name `res.groups.privilege`, storage `res_groups_privilege`.

**Privilege elevation** — The mode in which the entity, record and field layers of the access check
are suspended while the acting identity is unchanged. It does **not** suspend constraints.

**Programmatic key management** — Minting and revoking application keys without a browser, gated by
a switch, a per-user limit and a scope-compatibility rule.

**Rate limiter (second factor)** — Two counters, one for mailing a code and one for verifying a
code, each allowing 5 events per 3 600 seconds per account, purged entirely on a successful
verification.

**Recipient signature** — A keyed hash over (database name, document access token, recipient contact
identifier) that lets a customer-facing page attribute a posted message to a known contact. It does
not grant access.

**Record-level check** — The second layer of the access check: which of the given records satisfy
the combined rule filter.

**Record rule** — A filter restricting which records of an entity an operation may touch, attached
to zero groups (global) or to some groups (a grant). Transport name `ir.rule`, storage `ir_rule`.

**Recycling candidate** — One record proposed by a recycling rule, awaiting validation or discard.
Transport name `data_recycle.record`.

**Recycling rule** — A periodic rule collecting records that match a filter and an optional age
condition, and proposing them for archival or deletion. Transport name `data_recycle.model`.

**Registration** — Creating an account for oneself, with or without an invitation token.

**Relation-command protection** — The rule that commands on a relation whose target entity forbids
elevated commands are applied in a de-elevated environment attached to the transaction's original
user.

**Root company** — The company at the top of a company tree, that is, the one with no parent.

**Root-delegated field** — A field that must hold the same value on every company of a tree. The
currency is the only one in the foundation.

**Scope (of an application key)** — A text restricting a key to one purpose. An empty scope means a
global key. The purpose `browser` marks a trusted-browser key; the purpose used for programmatic
calls is `rpc` (the remote-call purpose).

**Second factor** — An additional proof of identity demanded after a successful first factor, unless
the result's policy is `skip`.

**Self-service fields** — The two lists of fields a user may read and write on their own account
without any administration group.

**Session** — The server-side state associated with a browser, holding at least the database name,
the account identifier, the login, the preference context and the session token.

**Session token** — A keyed hash binding a session to the state of its account, recomputed on every
request and compared in constant time. Its key material is the *session-token fields*.

**Session-token fields** — The account fields whose values form the key of the session token:
identifier, login, password, active, plus the second-factor secret, the delegated-sign-in token and
the list of passkey identifiers.

**Session timeout** — The number of minutes after which a session must be re-established regardless
of activity, configured per group and summarised per user.

**Settings mechanism** — The procedure that projects a settings screen's fields into user-defined
defaults, group implications, system parameters and package installation requests, and back.

**Share flag** — The derived boolean on an account, true when the closure does not contain
*Role / User*. Every "only my own documents" rule keys off it.

**Sign-up token** — A signed payload carrying the contact identifier, the contact's account
identifiers, the most recent sign-in moment of those accounts and the sign-up type, with an expiry.
Not stored anywhere.

**Signed payload** — A self-contained token packing a message, an expiry and a keyed-hash signature
into one address-safe string, specified in [calculations.md](calculations.md) section 4.2.

**Subset closure** — For a set of groups, that set together with every group that reaches it by
implication. Used to rewrite searches "through" the closure.

**Superset closure** — For a set of groups, that set together with every group it reaches by
implication. The group closure of an account is the superset closure of its explicit groups.

**System account** — The account of identifier 1. Its environments are permanently
privilege-elevated. It cannot be deleted or re-activated.

**System parameter** — A global key/value setting. Transport name `ir.config_parameter`, storage
`ir_config_parameter`.

**System user** — An account whose closure contains *Role / Administrator*. Distinct from an
*administrator*, which also includes members of *Access Rights* and the account of identifier 1.

**Template account (external users)** — The archived account, referenced by the parameter
`base.template_portal_user_id` (the external-user-template parameter), that is copied whenever a
self-registered or portal-granted account is created. It holds exactly *Role / Portal*.

**Template account (directory)** — The account a directory binding copies when provisioning.

**Trusted browser** — A browser that may skip the second factor, represented by a scoped application
key with the purpose `browser` plus a cookie holding the clear key.

**Two-factor authentication** — See *second factor*.

**Universe expression** — The group expression meaning "every user", produced when an access-right
row grants an operation with no group.

**User** — An account able to sign in. Transport name `res.users`, storage `res_users`. It delegates
its personal data to a Contact.

**User settings** — The one-row-per-account store of client-side preferences. Transport name
`res.users.settings`, storage `res_users_settings`.

**Verification (of a key or password)** — Comparing a supplied secret against a stored hash,
possibly producing a replacement hash when the stored one is out of date.
