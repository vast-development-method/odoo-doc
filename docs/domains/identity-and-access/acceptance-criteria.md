# Identity and Access — Acceptance Criteria

Numbered scenarios in Given / When / Then form, with concrete values. A re-implementation that
satisfies all of them reproduces the domain's behaviour.

> **Note on reproduced texts.** Message texts, button labels and screen titles are reproduced
> **verbatim**, exactly as the system emits them, because external tests and user documentation
> depend on them. Where such a reproduced text contains an abbreviation, the abbreviation belongs to
> the text and is not this document's own prose. The abbreviations that occur are: *API* for
> application programming interface; *2FA* for two-factor authentication; *LDAP* for the central
> directory protocol; *DN* for distinguished name; *OAuth* and *UID* for the delegated sign-in
> protocol and its subject identifier; *JSON* for the structured-literal notation; *HTTP* for the
> transport protocol; *ID* for identifier. Outside such reproduced texts, every term is written in
> full.

**Shared fixture.** Unless a scenario says otherwise:

| Object | Value |
|---|---|
| Company 1 | *Alpha*, root, currency the United States dollar, sequence 10 |
| Company 2 | *Beta*, root, currency the euro, sequence 20 |
| Company 3 | *Alpha North*, parent company 1, currency the United States dollar |
| Account 1 | the system account, permanently elevated |
| Account 2 | the administrator, login `admin`, holds *Role / Administrator* |
| Account 40 | *Sofia Lang*, login `sofia@alpha.test`, internal, permitted in company 1, default company 1 |
| Account 41 | *Marc Ruiz*, login `marc@alpha.test*`, internal, permitted in companies 1 and 2, default company 1 |
| Account 42 | *Widgets Limited*, login `contact@widgets.test`, external (*Role / Portal*), permitted in company 1 |
| Group 10 | *Sales / User: own documents only*, implies *Role / User* |
| Group 11 | *Sales / User: all documents*, implies group 10 |
| Group 12 | *Sales / Administrator*, implies group 11 |
| Parameters | `base.login_cooldown_after` = 10, `base.login_cooldown_duration` = 60, `auth_password_policy.minlength` = 0, `auth_signup.invitation_scope` = `b2c`, `auth_signup.reset_password` = `True` |

---

## 1. Group closure

### Scenario 1.1 — A group with implied groups and the resulting closure

**Given** the implication graph

| Group | Directly implies |
|---|---|
| 1 *Role / User* | 5 *Technical Features* |
| 2 *Role / Administrator* | 3 *Access Rights*, 4 *Bypass HTML Field Sanitize* |
| 3 *Access Rights* | 1 |
| 10 *Sales / User: own documents only* | 1 |
| 11 *Sales / User: all documents* | 10 |
| 12 *Sales / Administrator* | 11 |

**And** account 43 whose **explicit** groups are exactly {12, 2}

**When** the closure of account 43 is computed

**Then** the closure is exactly the eight groups {1, 2, 3, 4, 5, 10, 11, 12}

**And** the group count shown on the account form is 8

**And** the derived *Share User* flag is false, because group 1 is present

**And** the derived *Role* field reads *Administrator*, because group 2 is present

**And** searching for accounts whose closure contains group 1 finds account 43, because its explicit
group 12 is in the subset closure of {1}

**And** removing group 2 from the explicit groups reduces the closure to {1, 5, 10, 11, 12} — five
groups — while the *Share User* flag stays false and the *Role* field becomes *User*.

### Scenario 1.2 — Implication cycles are harmless

**Given** two groups 20 and 21 where 20 implies 21 and 21 implies 20

**When** the closure of an account whose explicit groups are {20} is computed

**Then** the closure is {20, 21} and the computation terminates.

### Scenario 1.3 — A disjointness violation is refused

**Given** account 40 holds *Role / User*

**When** an administrator adds *Role / Portal* to account 40's explicit groups

**Then** the write is refused with

> User 'Sofia Lang' cannot be at the same time in exclusive groups 'Role / User', 'Role / Portal'.

**And** nothing is written.

### Scenario 1.4 — A disjointness violation created through implication is refused

**Given** a group 30 that implies *Role / Portal*, and account 40 holding *Role / User*

**When** an administrator adds group 30 to account 40

**Then** the write is refused with the same message, because the test is made on the closure and not
on the explicit groups.

### Scenario 1.5 — The last administrator cannot be demoted

**Given** account 2 is the only account whose closure contains *Role / Administrator*

**When** an administrator removes that group from account 2

**Then** the write is refused with

> You must have at least an administrator user.

### Scenario 1.6 — Removing an implied group from the whole closure

**Given** group 12 implies group 11, which implies group 10

**When** the settings mechanism removes group 10 from the holder group 12

**Then** the implication *group 11 implies group 10* is deleted, because the removal walks the whole
closure of the holder group and deletes every direct implication of group 10 found there

**And** a subsequent closure of an account holding group 12 no longer contains group 10.

### Scenario 1.7 — Applying an implied group is idempotent

**Given** group 12 already reaches group 10 through group 11

**When** the settings mechanism applies group 10 to holder group 12

**Then** nothing is written, because group 10 is already in group 12's transitive closure.

---

## 2. The access check

### Scenario 2.1 — An access check failing on the entity, with its exact message

**Given** the entity *Journal Entry* (`account.move`, the accounting-document entity) whose label is
*Journal Entry*

**And** access-right rows granting read on it to the groups *Accounting / Billing* and
*Accounting / Accountant* only, both with a privilege named *Accounting*

**And** account 40, whose closure contains neither

**When** account 40 reads any Journal Entry

**Then** the read is refused with exactly:

```
You are not allowed to access 'Journal Entry' (account.move) records.

This operation is allowed for the following groups:
	- Accounting/Billing
	- Accounting/Accountant

Contact your administrator to request access if necessary.
```

(the two group lines each begin with one tabulation, a hyphen and a space; the three parts are
separated by blank lines)

**And** an informational log entry records the refusal with the operation, the account identifier
and the entity.

### Scenario 2.2 — The same check when no group at all holds the operation

**Given** the same entity but with no access-right row granting *delete* to any group

**When** account 40 deletes a Journal Entry

**Then** the refusal is

```
You are not allowed to delete 'Journal Entry' (account.move) records.

No group currently allows this operation.

Contact your administrator to request access if necessary.
```

### Scenario 2.3 — Deny by default

**Given** a newly defined entity with no access-right row at all

**When** account 2 (an administrator, not elevated) reads it

**Then** the read is refused, because the model is deny-by-default and administration groups carry
no implicit permission.

### Scenario 2.4 — An access-right row with no group

**Given** a row granting read on an entity with no group set

**When** the anonymous account reads it

**Then** the read is permitted

**And** creating such a row emitted a warning naming the row.

### Scenario 2.5 — The empty-set check

**Given** account 40 holds no group granting write on the Sales Order entity

**When** the check is invoked with an **empty** set of records and the operation *write*

**Then** the entity-level refusal is raised, and the record layer is not consulted at all.

### Scenario 2.6 — Elevation short-circuits the whole check

**Given** account 40 cannot read Journal Entries

**When** code running as account 40 elevates and reads a Journal Entry

**Then** the read succeeds and no access-right row and no record rule is consulted.

### Scenario 2.7 — Elevation does not suspend constraints

**Given** a Company named *Alpha* already exists

**When** elevated code creates a second Company named *Alpha*

**Then** the creation is refused with

> The company name must be unique!

---

## 3. Record rules

### Scenario 3.1 — A rule limiting a salesperson to their own documents

**Given** the Sales Order entity carries three rules:

| Rule | Groups | Filter | Operations |
|---|---|---|---|
| *Sales Order: multi-company* | none (global) | the order's company is one of the active companies, or the order has no company | all four |
| *Sales Order: personal* | group 10 | the order's salesperson is the acting user | all four |
| *Sales Order: all* | group 11 | always true | all four |

**And** account 40 holds group 10 and is active in company 1 only

**And** three orders exist: SO0001 (salesperson 40, company 1), SO0002 (salesperson 41, company 1),
SO0003 (salesperson 40, company 2)

**When** account 40 lists Sales Orders

**Then** the combined filter is

```
(company is in [1] or company is empty) and (salesperson = 40)
```

**And** the list contains SO0001 only

**And** reading SO0002 directly is refused with the record-level message

**And** the failing-rule diagnosis names *Sales Order: personal*, because 0 of the 1 forbidden
records satisfy the grant disjunction while 1 of 1 satisfies the global rule.

### Scenario 3.2 — The manager sees everything through the union

**Given** the same three rules, and account 41 holding group 11 (which implies group 10), active in
companies 1 and 2

**When** account 41 lists Sales Orders

**Then** the grant disjunction is `salesperson = 41 or always true`, which reduces to always true

**And** the combined filter is `company is in [1, 2] or company is empty`

**And** all three orders are listed.

This is the criterion that distinguishes union from intersection: under intersection, account 41
would see fewer orders than account 40's rule alone allows, which is the opposite of the intent.

### Scenario 3.3 — A global rule cannot be widened

**Given** account 41 as above but active in company 1 only

**When** account 41 lists Sales Orders

**Then** SO0003 (company 2) is absent, even though the *Sales Order: all* grant is always true,
because the global rule is combined by intersection.

### Scenario 3.4 — Rules on a delegated entity apply

**Given** the User entity delegates to the Contact entity through a stored link

**And** a global rule on Contacts restricting to non-shared contacts or contacts of an active
company

**When** account 40 reads a User

**Then** the combined filter for Users contains, in addition to the User rules, the condition that
the delegation link satisfies the Contact filter.

### Scenario 3.5 — An inactive rule constrains nothing

**Given** *Sales Order: multi-company* is deactivated

**When** account 40 lists Sales Orders

**Then** the combined filter is `salesperson = 40` alone, and SO0003 (company 2) becomes visible.

### Scenario 3.6 — A rule with an empty filter is always true

**Given** a grant rule attached to group 10 with an empty filter

**When** account 40 lists Sales Orders

**Then** that grant contributes the always-true filter, so the whole grant disjunction is always
true and every order of the active companies is listed.

### Scenario 3.7 — A rule on the record-rule entity is refused

**When** an administrator creates a record rule whose entity is the record-rule entity itself

**Then** the creation is refused with

> Rules can not be applied on the Record Rules model.

### Scenario 3.8 — A rule with no operation is refused

**When** an administrator clears all four operation flags on a rule

**Then** the write is refused with

> Rule must have at least one checked access right!

### Scenario 3.9 — An invalid filter is refused

**Given** a rule whose filter text refers to a field that does not exist on the entity

**When** the rule is saved while active

**Then** the save is refused with

> Invalid domain: *the parser's own message*

### Scenario 3.10 — The archive filter is disabled during the record check

**Given** SO0001 is archived and account 40 may read it under the rules

**When** account 40 checks read access on SO0001

**Then** the check passes, because the filtering step disables the archive filter; the record is
simply absent from ordinary lists rather than reported as forbidden.

### Scenario 3.11 — The detailed refusal is shown only in developer mode

**Given** account 40 is internal and holds *Technical Features*, and the request is in developer
mode

**When** account 40 reads SO0002

**Then** the message names SO0002 by display name and identifier, and lists *Sales Order: personal*
under *Blame the following rules:*

**And** when the same read is performed by account 42 (external), the message contains only the
entity line and the resolution text, whatever the developer mode.

### Scenario 3.12 — The multi-company hint

**Given** SO0003 belongs to company 2, account 41 is permitted in companies 1 and 2, and the active
list is `[1]`

**When** account 41 reads SO0003

**Then** the refusal carries a structured hint naming company 2

**And** the resolution text ends with

> This seems to be a multi-company issue, you might be able to access the record by switching to the
> company: Beta.

**And** when account 41 is permitted only in company 1, the same read instead ends with

> This seems to be a multi-company issue, but you do not have access to the proper company to access
> the record anyhow.

---

## 4. Companies

### Scenario 4.1 — A user with two companies switching the active one

**Given** account 41, permitted in companies 1 and 2, default company 1

**When** account 41 signs in

**Then** the client receives *current company* = 1 and *allowed companies* = {1, 2}

**And** with no active list in the context, the current company resolves to *Alpha* and the active
companies resolve to {*Alpha*, *Beta*}.

**When** the client sends the active list `[1]`

**Then** validation passes, the current company is *Alpha*, and a Sales Order created in that call
receives company 1.

**When** the client sends the active list `[2, 1]`

**Then** validation passes, the current company becomes *Beta*, records of both companies are
visible, and a Sales Order created in that call receives company 2.

**When** an administrator removes company 2 from account 41's permitted companies, and account 41's
next call still carries `[2, 1]`

**Then** the call is refused with

> Access to unauthorized or invalid companies.

**And** after the client asks for a fresh session description, *Beta* is no longer offered.

### Scenario 4.2 — The default company must be permitted

**Given** account 41 permitted in companies 1 and 2, default company 2

**When** an administrator sets the permitted companies to {1} alone

**Then** the write is refused with

> Company Beta is not in the allowed companies for user Marc Ruiz (Alpha).

### Scenario 4.3 — The multi-company group is maintained mechanically

**Given** account 40 permitted in company 1 only, not holding *Multi Companies*

**When** an administrator adds company 2 to its permitted companies

**Then** *Multi Companies* is added to its explicit groups.

**When** the administrator removes company 2 again

**Then** *Multi Companies* is removed.

### Scenario 4.4 — Company consistency is enforced

**Given** a Sales Order of company 1 and a Payment Term belonging to company 2

**When** the payment term is set on the order and the order's entity requires company consistency

**Then** the write is refused with a message beginning

> Uh-oh! You've got some company inconsistencies here:

**And** containing a line of the form

> - "SO0001" belongs to company "Alpha" while "Payment Terms" (invoice_payment_term_id: '30 Days')
> belongs to another company.

**And** ending with

> To avoid a mess, no company crossover is allowed!

### Scenario 4.5 — A branch must share its root's currency

**Given** company 3 is a branch of company 1 whose currency is the United States dollar

**When** an administrator sets company 3's currency to the euro

**Then** the write is refused with

> The Currency of a subsidiary must be the same as it's root company.

### Scenario 4.6 — Writing a delegated field on the root cascades

**When** an administrator sets company 1's currency to the Swiss franc

**Then** company 3's currency becomes the Swiss franc as well

**And** any currency that was archived is activated.

### Scenario 4.7 — The hierarchy is immutable

**When** an administrator writes a parent on company 2

**Then** the write is refused with

> The company hierarchy cannot be changed.

### Scenario 4.8 — Archiving cascades and is guarded

**Given** account 40's default company is company 1 and account 40 is active

**When** an administrator archives company 1

**Then** the archival is refused with

> The company Alpha cannot be archived because it is still used as the default company of 1 users.

**When** account 40 is first archived and company 1 is archived again

**Then** company 1 and company 3 are both archived.

### Scenario 4.9 — Duplication is refused

**When** an administrator duplicates company 1

**Then** the operation is refused with

> Duplicating a company is not allowed. Please create a new company instead.

### Scenario 4.10 — Accessible branches

**Given** account 41 is permitted in companies 1 and 3, and the active list is `[1, 3]`

**When** the accessible branches of company 1 are computed

**Then** the result is {1, 3}

**And** when the active list is `[1]`, the result is {1}

**And** when the computation runs as the account of identifier 1 inside a scheduled job with an
empty intersection, the result is the company itself.

---

## 5. Authentication

### Scenario 5.1 — A successful password sign-in

**Given** account 40 with the password `pa55word-alpha`

**When** account 40 submits its login and that password

**Then** the credential check verifies the stored hash and reports (account 40, method `password`,
policy `default`)

**And** a sign-in log row is created, so the account's status becomes *Confirmed*

**And** the session records the database name, the login, the account identifier, the preference
context and the session token

**And** the cooldown entry for the source address is removed.

### Scenario 5.2 — A silent re-hash

**Given** account 40's stored hash uses 600 000 iterations, and `password.hashing.rounds` has since
been set to 1 000 000

**When** account 40 signs in successfully

**Then** a replacement hash with 1 000 000 iterations is written to the row

**And** the registry caches are cleared

**And** the session token is recomputed and written into the session, so the user is **not** signed
out.

### Scenario 5.3 — A cooldown after repeated failures

**Given** `base.login_cooldown_after` = 10 and `base.login_cooldown_duration` = 60, and no previous
failures from the source address

**When** ten sign-in attempts with a wrong password are made from that address

**Then** each is refused with *Wrong login/password* and the failure counter reaches 10

**When** an eleventh attempt is made within 60 seconds of the tenth failure

**Then** the attempt is refused **before the password is even examined**, with

> Too many login failures, please wait a bit before trying again.

**And** a warning naming the source, the attempted login, the database, the failure count and the
moment of the last failure is logged

**And** when the source address is in a private range, a second warning about a possibly
mis-configured reverse proxy is logged

**And** the failure counter becomes 11 and the last-failure moment is reset, so the window restarts.

**When** 61 seconds pass with no attempt and a correct password is then supplied

**Then** the attempt proceeds, succeeds, and the counter entry is removed entirely.

**And** with the parameter deleted, the same scenario triggers the cooldown at the fifth failure
instead of the tenth, because the code's fallback threshold is 5.

### Scenario 5.4 — A disabled cooldown

**Given** `base.login_cooldown_after` = 0

**When** a hundred failed attempts are made from one address

**Then** none of them is ever refused by the cooldown.

### Scenario 5.5 — A session token invalidated by a change elsewhere

**Given** account 40 is signed in on two browsers

**When** an administrator changes account 40's password

**Then** the next request from each browser recomputes the token, finds a mismatch, signs that
session out keeping the database, and continues unauthenticated.

### Scenario 5.6 — The older token construction is accepted once

**Given** a session whose stored token matches the older construction

**When** a request arrives

**Then** the session is accepted, and the stored token is replaced by the current construction.

### Scenario 5.7 — An application key in place of a password on a non-interactive connection

**Given** account 40 has a valid unscoped key

**When** a non-interactive call presents the login and the key as the password

**Then** the password verification fails, the key verification succeeds, and the result reports
(account 40, method `apikey`, policy `default`).

### Scenario 5.8 — A password is refused on a non-interactive connection once the second factor is on

**Given** account 40 has the second factor enabled

**When** a non-interactive call presents the login and the correct password

**Then** the call is refused, and an informational log entry records that a password was attempted
on a connection that requires a key.

### Scenario 5.9 — Impersonation

**Given** account 2 holds *Role / Administrator* and is signed in

**When** account 2 requests impersonation

**Then** the session's account becomes the account of identifier 1, the registry caches are cleared
and a new session token is computed

**And** the same request made by account 40 leaves the session unchanged.

---

## 6. The second factor

### Scenario 6.1 — A two-factor sign-in with a wrong then a right code

**Given** account 40 has the second factor enabled with a secret *K*, a last accepted counter of
59 599 999, no trusted-browser cookie, and the rate-limit table empty for that account

**And** the current moment is 1 788 000 041 seconds, so the window contains the counters 59 600 000,
59 600 001 and 59 600 002, and the code at 59 600 001 is `704318`

**When** account 40 signs in with the correct password

**Then** the result's policy is `default`, the account has a second-factor address, the session
becomes *Pending* and the visitor is sent to the second-factor page.

**When** account 40 submits the code `123456`

**Then** the cooldown guard permits the attempt (no failures yet)

**And** the code-check rate limit is consumed: the count before the write is 0, which is below 5, so
a row is written

**And** no counter in the window produces `123456`

**And** the refusal is

> Verification failed, please double-check the 6-digit code

**And** the cooldown failure counter for the source becomes 1

**And** the session is still *Pending*: the visitor is **not** signed in.

**When** account 40 submits the code `704318`

**Then** the rate limit is consumed again (count 1, below 5, a second row is written)

**And** the match is counter 59 600 001, which is strictly greater than 59 599 999, so the code is
accepted

**And** the last accepted counter becomes 59 600 001

**And** **all** the account's code-check rate-limit rows are deleted

**And** the result is (account 40, method `totp`, policy `default`)

**And** the session is finalised: the pending keys are popped, the token is computed and stored, the
session is marked for rotation

**And** the cooldown entry for the source is removed.

**When** account 40 submits `704318` again five seconds later

**Then** the match is again 59 600 001, which is not strictly greater than the stored last counter,
and the refusal is

> Verification failed, please use the latest 6-digit code

### Scenario 6.2 — The code rate limit

**Given** account 40 with the second factor enabled

**When** six wrong codes are submitted within one hour

**Then** the first five each write a rate-limit row and are refused with the wrong-code message

**And** the sixth is refused, **before the code is examined and without writing a row**, with

> You reached the limit of code verifications for your account, please try again later.

**And** the five rows expire 3 600 seconds after each was written.

### Scenario 6.3 — Trusted browser

**Given** account 40 completed a second factor and ticked "remember this browser", and
`auth_totp.trusted_device_age` is absent

**Then** a scoped key with the purpose `browser` was generated with an expiry of the current moment
plus 90 × 86 400 = 7 776 000 seconds

**And** a cookie holding the clear key was set with that maximum age, marked inaccessible to scripts,
with the same-site policy *lax*

**And** its label is, for example, `Firefox on Linux (Lyon, France)`.

**When** account 40 signs in again from that browser

**Then** the second-factor page verifies the cookie and finalises the session without asking for a
code.

**When** account 40 changes its own password

**Then** every trusted browser is revoked first, so the next sign-in asks for a code again

**And** a notice with the subject *Security Update: Device Removed* is mailed, naming the removed
device labels.

### Scenario 6.4 — Enrolment

**Given** account 40 has no secret

**When** account 40 opens the enable action and the identity was last confirmed 15 minutes ago

**Then** the action is suspended and the *Access Control* dialogue opens; after the password is
confirmed, the suspended call is replayed.

**When** account 40 scans the enrolment image and types the code shown

**Then** the secret and the matched counter are stored

**And** the session token of the enrolling session is recomputed and stored

**And** every **other** session of account 40 becomes invalid at its next request

**And** the notice *2-Factor authentication is now enabled.* is shown

**And** a message with the subject *Security Update: 2FA Activated* is mailed.

**When** account 41 attempts to enable the second factor on account 40

**Then** the operation is refused with

> Two-factor authentication can only be enabled for yourself

### Scenario 6.5 — A passkey sign-in skips the second factor

**Given** account 40 has the second factor enabled **and** a registered passkey

**When** account 40 signs in with the passkey

**Then** the credential check reports policy `skip`, the session is finalised immediately, and no
code is asked for.

### Scenario 6.6 — A new-device notice

**Given** account 40 has the second factor enabled and signs in from a browser with no trusted
cookie

**Then** a message with the subject *New Connection to your Account* and the body *A new device was
used to sign in to your account.* is mailed to the account's address, in the account's language.

---

## 7. Passwords

### Scenario 7.1 — A password refused by the policy

**Given** `auth_password_policy.minlength` = 12

**When** account 40 changes its own password to `summer2026` (10 characters)

**Then** the confirmation matches and the identity re-check passes

**And** the write is refused with

> Your password must contain at least 12 characters and only has 10.

**And** the stored hash is unchanged and the session is unaffected.

**When** account 40 instead supplies `summer-2026-!` (13 characters)

**Then** the write succeeds and the session token is recomputed.

### Scenario 7.2 — Several failures are reported together

**Given** the administrator password wizard with two lines, both carrying passwords shorter than the
minimum

**When** the wizard is applied

**Then** the refusal text contains both failure sentences, joined by a blank line and a space.

### Scenario 7.3 — An empty password is refused

**When** a password is changed to whitespace only

**Then** the change is refused with

> Setting empty passwords is not allowed for security reasons!

### Scenario 7.4 — Changing one's own password through the wrong field

**When** account 40 writes the *Set Password* field on its own account

**Then** the write is refused with

> Please use the change password wizard (in User Preferences or User menu) to change your own password.

### Scenario 7.5 — The confirmation must match

**When** the self-service dialogue is submitted with two different values

**Then** the save is refused with

> The new password and its confirmation must be identical.

### Scenario 7.6 — The customer-facing password change

**Given** account 42 on the customer-facing security page

**When** it submits an empty new password

**Then** the page reports *You cannot leave any password empty.*

**When** it submits a wrong current password

**Then** the page reports *The old password you provided is incorrect, your password was not
changed.*

**When** it submits a correct current password and a matching new one

**Then** the change succeeds and the session token is recomputed, so the user stays signed in.

### Scenario 7.7 — Administrator-driven change does not leave clear values behind

**When** an administrator changes two accounts' passwords through the wizard

**Then** after the operation every wizard line's new-password field is blank

**And** if the acting administrator was among the affected accounts, the client is told to reload.

---

## 8. The customer-facing token

### Scenario 8.1 — An external party reading through a signed token

**Given** sales order SO0042 of company 1, customer contact *Widgets Limited* (contact 1904), which
has **no** account

**And** self-registration is closed (`auth_signup.invitation_scope` = `b2b`)

**When** account 40 opens the share dialogue on SO0042

**Then** account 40's **read** access to SO0042 is checked and passes

**And** because SO0042 has no token, a version-4 random universally unique identifier is generated
and written with elevation

**And** the computed link is the base address, the generic mail-view path, and a query carrying the
entity transport name, the record identifier and the token.

**When** account 40 sends the share to *Widgets Limited*

**Then** because self-registration is closed, the recipient receives the **plain** share message

**And** the message is posted on SO0042 as an internal note with the light notification layout,
subject *Invitation to access SO0042*, in the recipient's language

**And** the link embedded for that recipient additionally carries the recipient contact identifier
and the recipient signature computed over (database name, access token, 1904).

**When** the recipient opens the link

**Then** the route's mode is `public`, so the environment is the anonymous account

**And** the document exists, so no missing-document error is raised

**And** the read check as the anonymous account refuses

**And** a token was supplied, the document has a token, and they are equal in constant time, so the
refusal is swallowed

**And** the document is returned **fully elevated**, and the page renders it

**And** the recipient signature verifies, so a message posted in the thread is attributed to
*Widgets Limited*.

**When** the recipient forwards the link to a third person who opens it

**Then** that person sees the same document, because the token is the credential and is not bound to
a person

**And** the recipient signature does **not** transfer, because it is bound to contact 1904.

### Scenario 8.2 — A wrong token

**When** the link is opened with the token altered by one character

**Then** the comparison fails and the original read refusal is raised.

### Scenario 8.3 — A deleted document

**When** the link is opened after SO0042 has been deleted

**Then** the answer is

> This document does not exist.

and **not** a permission refusal.

### Scenario 8.4 — A duplicate carries no token

**When** SO0042 is duplicated

**Then** the copy has no access token, and the old link does not open it.

### Scenario 8.5 — Sharing with self-registration open

**Given** `auth_signup.invitation_scope` = `b2c` and a document with **no** token yet

**When** the share wizard sends to two recipients, one with an account and one without

**Then** the one with an account receives the plain share message

**And** the one without receives an individual message carrying a registration link that redirects,
after registration, to the document.

---

## 9. Registration and invitation

### Scenario 9.1 — A sign-up through an invitation

**Given** `auth_signup.signup.validity.hours` is absent, so the validity is 144 hours

**When** an administrator creates an internal account for *Jean Martin* with the address
`jean.martin@example.test`

**Then** the account is created with the default groups (*Role / User* plus the groups implied by
*Default access for new users*)

**And** a settings row is created for it

**And** the contact's sign-up type becomes `signup`

**And** a token is computed over the payload (the contact identifier, the list of the contact's
account identifiers, the most recent sign-in moment — absent — and the type `signup`), signed under
the scope `signup`, expiring 144 hours from now

**And** an invitation carrying the registration link is mailed, forcibly and synchronously.

**When** Jean opens the link

**Then** the page resolves the token, verifies the signature and the expiry, re-reads the contact,
and finds that the three recorded elements still match the contact's current state

**And** the form is pre-filled with the contact's name and the existing account's login.

**When** Jean submits a password twice

**Then** the two must match, otherwise *Passwords do not match; please retype them.*

**And** the contact's sign-up type is cleared, so the token is dead

**And** because the contact already has an account, the proposed login and name are dropped and only
the password is written

**And** the password write runs the strength policy

**And** because the account had never signed in and is internal, the inviter is notified over the
live channel

**And** the work is committed

**And** Jean is signed in, so a sign-in log row is created and the status becomes *Confirmed*.

**When** Jean opens the same link again

**Then** the token no longer resolves — both because the type was cleared and because the most
recent sign-in moment has changed — and the page reports

> Invalid signup token

**And** had Jean waited seven days instead, the signature's expiry would have elapsed and the
resolution would have failed the same way.

### Scenario 9.2 — Registration without an invitation when self-registration is closed

**Given** `auth_signup.invitation_scope` = `b2b`

**When** a visitor opens the registration page without a token

**Then** the page does not exist.

**When** a registration is attempted programmatically with no contact and no token

**Then** it is refused with

> Signup is not allowed for uninvited users

### Scenario 9.3 — A duplicated address

**Given** an account already exists with the login `jean.martin@example.test`

**When** a registration proposes that address

**Then** it is refused with

> Another user is already registered using this email address.

### Scenario 9.4 — A missing template

**Given** `base.template_portal_user_id` points at a deleted account

**When** a registration is attempted

**Then** it is refused with

> Signup: invalid template user

### Scenario 9.5 — Geolocated values do not overwrite real ones

**Given** the invited contact already has a country and a city

**When** the registration form submits a guessed city and country

**Then** both are dropped and the contact's existing values survive.

### Scenario 9.6 — A password reset

**Given** `auth_signup.reset_password.validity.hours` is absent, so the validity is 4 hours

**When** account 40 requests a reset by login

**Then** the contact's sign-up type becomes `reset`, a token valid 4 hours is computed, and the reset
message is mailed

**And** the page reports *Password reset instructions sent to your email address.*

**When** the link is used within 4 hours

**Then** the new password is written **without** signing the visitor in, and the page reports *Your
password has been reset successfully.*

**When** an unknown login is submitted

**Then** the page reports *No account found for this login*.

### Scenario 9.7 — Reset on an archived account

**When** a reset is requested for an archived account

**Then** it is refused with

> You cannot perform this action on an archived user.

### Scenario 9.8 — The unregistered-account reminder

**Given** account 44 was created 5 days ago by account 2, has an address, and has never signed in

**When** the reminder job runs today

**Then** account 2 receives one message listing `Name (login)` for account 44

**And** account 44, if it had signed in once, would not appear.

---

## 10. Portal access

### Scenario 10.1 — Granting

**Given** contact 1904 *Widgets Limited* with the address `contact@widgets.test` and no account

**When** an internal user opens the portal access wizard on contact 1904 and presses grant

**Then** the line's status was *Valid*, so no refusal occurs

**And** an account is created from the external-user template in contact 1904's company, with
`contact@widgets.test` as both login and address

**And** the account is active, holds *Role / Portal* and does not hold *Role / Public*

**And** a sign-up invitation is prepared and the invitation message is sent forcibly.

### Scenario 10.2 — Granting twice

**When** the grant is pressed again

**Then** it is refused with

> The partner "Widgets Limited" already has the portal access.

### Scenario 10.3 — An invalid address

**Given** the line's address is `not-an-address`

**When** grant is pressed

**Then** it is refused with

> The contact "Widgets Limited" does not have a valid email.

### Scenario 10.4 — A duplicated address

**Given** another account already has the login `contact@widgets.test`

**When** grant is pressed on contact 1904

**Then** it is refused with

> The contact "Widgets Limited" has the same email as an existing user

### Scenario 10.5 — Revoking

**When** revoke is pressed on an external contact

**Then** the contact's sign-up type is cleared and the account is archived

**And** the account still holds *Role / Portal* — it is **not** moved to *Role / Public*.

### Scenario 10.6 — An internal account is never recycled

**Given** contact 1905 has an **archived** account that holds *Role / User*

**When** the wizard is opened on contact 1905

**Then** the line reports *Is Internal* and not *Is Portal*

**And** grant is refused with

> The partner "…" already has the portal access.

### Scenario 10.7 — Self-service removal

**Given** account 42 on the customer-facing security page

**When** it types its own login as the confirmation and its correct password

**Then** the login becomes `__deleted_user_42_<the current moment as a decimal number of seconds>`,
the secret is cleared, its application keys are removed, a deletion request in state *To Do* is
created, and the account and its contact are archived

**And** the session is signed out and the visitor is redirected with the message *Account deleted!*

**When** the deletion job next runs

**Then** it takes the row lock, re-tests the state, deletes the account, sets the request to *Done*,
and then attempts to delete the contact; a referential failure there is logged as a warning and the
contact survives.

**When** account 40 (internal) attempts the same

**Then** it is refused with

> Only the portal users can delete their accounts. The user(s) Sofia Lang can not be deleted.

---

## 11. Application keys

### Scenario 11.1 — An application key on the transport

**Given** account 40 is internal and its closure contains *Role / User*, whose maximum
application-key duration is 90 days

**When** account 40 opens the new-key dialogue

**Then** the identity re-check fires, and after confirmation the duration choices offered are
1 Day, 1 Week, 1 Month, 3 Months and Custom Date — the persistent choice is absent because account
40 is not an administrator, and 6 Months and 1 Year are absent because 180 > 90 and 365 > 90.

**When** account 40 picks *3 Months* and confirms with the label `Nightly export`

**Then** the expiry is today plus 90 days, which passes validation with equality

**And** 20 random bytes are drawn and rendered as 40 hexadecimal characters, say
`9f2c1b7e4a0d6583cc21ff90ab3d7e5164280cb7`

**And** the stored row carries the label, the account, no scope, the expiry, the hash and the index
`9f2c1b7e`

**And** the clear key is shown once in a dialogue titled *API Key Ready*.

**When** an integration calls a route whose mode is `bearer` with the header
`Authorization: bearer 9f2c1b7e4a0d6583cc21ff90ab3d7e5164280cb7`

**Then** verification selects the rows whose index is `9f2c1b7e`, joined to active accounts, whose
scope is empty or `rpc` and whose expiry is empty or not past, and verifies the candidate against
each stored hash

**And** the match yields account 40; the environment is attached to account 40 and the session is
marked not savable

**And** every access decision in the call is taken as account 40 — the key grants no more than the
account's own permissions.

**When** the same route is called with no header at all, from a browser navigation carrying the
markers *destination: document*, *mode: navigate*, *site: same-origin* and *user-activated*, on an
established session

**Then** the call proceeds as that session's account.

**When** the same route is called with no header from a call that lacks those markers

**Then** the call is refused with

> Missing "Authorization" or Sec-headers for interactive usage.

**When** the key has expired

**Then** verification does not return it, and the answer is

> Invalid apikey

with a bearer authentication challenge.

### Scenario 11.2 — Exceeding the maximum duration

**When** account 40 picks *Custom Date* and types a date 120 days away

**Then** the validation refuses with

> You cannot exceed 90.0 days.

**And** a date in the past refuses with

> You cannot set an expiration date in the past.

**And** leaving the date empty refuses with

> The application programming interface key must have an expiration date

### Scenario 11.3 — An administrator may mint a persistent key

**Given** account 2 holds *Role / Administrator*

**Then** the duration choices include *Persistent Key*, and choosing it produces a key with no
expiry; no duration validation is performed at all.

### Scenario 11.4 — Removal permissions

**When** account 40 removes its own key

**Then** the identity re-check fires and the removal succeeds.

**When** account 40 removes account 41's key

**Then** it is refused with

> You can not remove application programming interface keys unless they're yours or you are a system user

### Scenario 11.5 — Only internal users may create keys

**Given** `portal.allow_api_keys` is set and account 42 is external

**When** account 42 confirms the key dialogue

**Then** it is refused with

> Only internal users can create application programming interface keys

### Scenario 11.6 — Programmatic minting

**Given** `base.enable_programmatic_api_keys` is not set and the caller is not a system user

**When** the programmatic minting operation is called

**Then** it is refused with

> Programmatic application programming interface keys are not enabled

**Given** the switch is set and account 40 already has 10 unexpired keys

**When** minting is attempted

**Then** it is refused with

> Limit of 10 application programming interface keys is reached for programmatic creation

**Given** the caller presents a key scoped to `reporting`

**When** it attempts to mint a key with no scope (a global key)

**Then** the verification of the presented key against the global purpose fails and the answer is

> The provided application programming interface key is invalid or does not belong to the current user.

---

## 12. Session lifetime

### Scenario 12.1 — Two limits combined

**Given** account 40's closure contains groups setting: session timeout 1440 minutes with a second
factor; session timeout 720 minutes without; session timeout 2880 minutes without; inactivity 15
minutes without; inactivity 30 minutes with

**When** the per-user summary is computed

**Then** the session list is `[(43200, false), (86400, true)]`

**And** the inactivity list is `[(900, false), (1800, true)]`.

### Scenario 12.2 — Inactivity re-authentication

**Given** the lists above, and account 40 reports 900 seconds of inactivity

**Then** the *next identity check* moment is written as the current moment

**When** 1 800 seconds later account 40 issues a request to a route whose mode is `user`

**Then** the entries are examined in descending duration order, so (1800, true) is tested first and
fires

**And** a re-authentication demand is raised, requiring **two** factors

**And** for a page request the response is a redirect to the re-authentication page carrying the
original address.

**When** account 40 supplies its password

**Then** the first factor is recorded in the session and the answer asks for the remaining methods.

**When** account 40 supplies a valid second-factor code

**Then** the *next identity check* moment is removed, the *last identity check* moment is set to
now, and the original request is retried.

### Scenario 12.3 — Session timeout is a sign-out, not a dialogue

**Given** the session was created 43 300 seconds ago and the session list is
`[(43200, false), (86400, true)]`

**When** a request arrives

**Then** the (43200, false) entry fires and a session-expired condition is raised; the user is
returned to the sign-in page with no dialogue.

### Scenario 12.4 — A route that opts out

**Given** the screen is locked

**When** the client fetches the menu tree

**Then** the request is served, because that route opts out of identity checking.

---

## 13. Defaults and settings

### Scenario 13.1 — Default precedence

**Given** four defaults on the Sales Order payment-term field:

| Identifier | User | Company | Value |
|---|---|---|---|
| 31 | — | — | payment term 1 |
| 32 | — | 2 | payment term 4 |
| 33 | 88 | — | payment term 7 |
| 34 | 88 | 2 | payment term 9 |

**And** the acting user is 88 and the acting company is 2

**When** the defaults of the Sales Order entity are read

**Then** the applied value is **payment term 9**, because the ordering is ascending by user, then
company, then identifier, with absent values placed **last**, so the most specific row wins

**And** deleting row 34 makes the winner payment term 7

**And** deleting rows 34 and 33 makes the winner payment term 4

**And** deleting rows 34, 33 and 32 makes the winner payment term 1.

### Scenario 13.2 — Setting a default validates the value

**When** a default is set for a field that does not exist

**Then** it is refused with

> Invalid field sale.order.nonexistent

**When** a default of 3 000 000 000 is set on an integer field

**Then** it is refused with

> Invalid value for sale.order.some_int: 3000000000 is out of bounds (integers should be between -2,147,483,648 and 2,147,483,647)

### Scenario 13.3 — Setting a default to its current value writes nothing

**When** the same value is set twice

**Then** the second call finds the serialised value unchanged and does not write, so the caches are
not cleared.

### Scenario 13.4 — A default on a field the user cannot write

**Given** account 40 cannot write the field *Salesperson* on Sales Orders

**When** account 40 creates a default for that field

**Then** the creation is refused with the field-access message naming *Salesperson*, the entity and
the operation *write*.

### Scenario 13.5 — Saving a settings screen

**Given** an administrator opens the settings screen and switches the multi-currency toggle on and
the minimum password length to 12

**When** the screen is saved

**Then** *Multi Currencies* is applied to *Role / User* — added as a direct implication because the
closure did not contain it

**And** the parameter `auth_password_policy.minlength` is set to the text `12`

**And** the client is told to reload.

**When** the administrator switches the multi-currency toggle off and saves

**Then** the implication is removed from every group in the closure of *Role / User* that directly
implies *Multi Currencies*.

### Scenario 13.6 — Removal is processed before addition

**Given** two group checkboxes on the same holder group: one switching off group 11 and one
switching on group 12 (which implies group 11)

**When** both are changed in one save

**Then** the removal is processed first and the addition second, so the holder group ends up
reaching group 11 through group 12.

### Scenario 13.7 — A text parameter is trimmed and an empty result deletes the parameter

**Given** a text parameter field whose value is `"  "` (two spaces)

**When** the screen is saved

**Then** the trimmed value is empty, so the value written is *false*, which **deletes** the
parameter.

### Scenario 13.8 — Packages to uninstall divert the save

**Given** a save that both installs one package and uninstalls another

**When** the screen is saved

**Then** the defaults, groups and parameters are written, everything is flushed, and the uninstall
confirmation dialogue is returned

**And** the installation does **not** happen in that call.

### Scenario 13.9 — A non-administrator cannot save

**When** account 40, holding neither *Role / Administrator* nor *Access Rights*, saves the settings
screen

**Then** it is refused with

> Only administrators can change the settings

### Scenario 13.10 — A dangling link parameter does not break the screen

**Given** a link-valued parameter field whose stored value points at a deleted record

**When** the settings screen is opened

**Then** the field reads as empty and the screen opens normally.

---

## 14. Data recycling

### Scenario 14.1 — A recycling rule collecting records older than ninety days

**Given** a rule on the Lead entity with: time field *Creation Date* (a date and time), delta 90,
delta unit *Days*, filter "the archive flag is false", *include archived* set, mode *manual*, action
*Delete*, notify every 1 week, notified user account 2

**And** the current moment is 11 September 2026 at 06:00

**When** the collection runs

**Then** the limit is 11 September 2026 06:00 minus 90 days = 13 June 2026 06:00

**And** the search filter is "the archive flag is false **and** the creation moment is at or before
13 June 2026 06:00", run with the archive filter disabled

**And** suppose it returns 61 342 records, of which 1 200 already have candidate rows

**Then** 60 142 new candidate rows are created, in batches of 50 000 and 10 142, with a commit after
each batch

**And** nothing is deleted, because the mode is manual.

**When** an operator validates 500 of them

**Then** the 500 pointed-at leads are deleted with elevation, any that no longer exist are skipped,
and the 500 candidate rows are deleted.

**When** the operator discards 200 of them

**Then** those 200 candidate rows have their active flag cleared and are **never proposed again by
this rule**.

**When** the collection runs again the next day

**Then** neither the 500 consumed nor the 200 discarded records reappear — the consumed ones because
they no longer exist, the discarded ones because a candidate row still names them.

### Scenario 14.2 — Automatic mode

**Given** the same rule but in mode *Automatic*

**When** the collection runs

**Then** the candidates are created and validated in batches of 5 000, with a commit after each, and
no candidate is ever visible to an operator.

### Scenario 14.3 — Calendar arithmetic

**Given** a rule with delta 1 and unit *Months*, and the current date is 31 March 2026

**Then** the limit is 28 February 2026, not 3 March 2026.

### Scenario 14.4 — Archive action on an entity without an archive flag

**When** an administrator sets the action to *Archive* on such an entity

**Then** it is refused with

> This model doesn't manage archived records. Only deletion is possible.

### Scenario 14.5 — Notification cadence

**Given** frequency 2, period *Weeks*, last notification 20 August 2026, the current moment
11 September 2026

**When** the notification pass runs

**Then** 20 August plus 14 days = 3 September, which is before 11 September, so the notice is due

**And** the last-notification moment becomes 11 September 2026

**And** the counted candidates are those created at or after 28 August 2026

**And** when that count is zero, **no notice is sent at all**.

### Scenario 14.6 — Deactivating a rule

**When** the rule's active flag is cleared

**Then** every candidate row of that rule is deleted, and no pointed-at record is touched.

### Scenario 14.7 — A guarded record survives

**Given** a recycling rule with the action *Delete* pointed at posted journal entries

**When** validation runs

**Then** the deletion is attempted with elevation and the financial domain's own guard refuses it;
the candidate stays and the batch continues.

---

## 15. Privacy

### Scenario 15.1 — A personal-data search

**Given** the name *Marie Dubois* and the address `marie.dubois@acme.test`, and contact 1904 whose
normalised address matches

**When** the search runs

**Then** the indirect references resolve to {1904}

**And** the result contains: contact 1904; every message authored by 1904; every sales order whose
customer, invoice address or delivery address is 1904; every invoice whose contact is 1904; and, for
the mailing-trace entity, rows whose address **equals** the normalised address rather than matching
a pattern

**And** notifications, followers and channel memberships are **not** searched, because they are
deleted by cascade

**And** the description reads, one entity per line, `<entity label> (<count>): #<identifier>, …`,
with the transport name appended after a hyphen when the operator holds *Technical Features*.

### Scenario 15.2 — An unreadable record

**Given** one found sales order belongs to a company the operator is not active in

**When** the result list is rendered

**Then** that line shows the record name but **no** link, and no refusal is raised.

### Scenario 15.3 — Archiving and deleting

**When** the operator clears the active flag on a line

**Then** the pointed-at record is archived with elevation and the line's detail becomes
`Archived Sales Order #6120`.

**When** the operator deletes a line

**Then** the record is deleted with elevation, the detail becomes `Deleted Sales Order #6120`, and
the line is marked deleted.

**When** the operator deletes the same line again

**Then** it is refused with

> The record is already unlinked.

### Scenario 15.4 — The log is masked

**When** the first action is taken

**Then** a privacy log is created carrying `M***** D*****` as the name and
`m****.d*****@a***.t***` as the address, together with the execution details and the record
description

**And** subsequent actions overwrite the same log's details rather than creating another.

### Scenario 15.5 — Masking of a common consumer domain

**Given** the address `marie@gmail.com`

**Then** the masked form is `m****@gmail.com` — the domain is left intact.

### Scenario 15.6 — An invalid address

**When** the search is run with an address that does not normalise

**Then** it is refused with

> Invalid email address "the value"

---

## 16. Identity re-check

### Scenario 16.1 — A fresh confirmation skips the dialogue

**Given** the session's identity confirmation is 4 minutes old

**When** account 40 presses *Add API Key*

**Then** the dialogue opens directly, with no password prompt.

### Scenario 16.2 — A stale confirmation suspends the call

**Given** the confirmation is 11 minutes old

**When** account 40 presses *Add API Key*

**Then** a wizard record is created with elevation holding the serialised call — the serialisable
context entries, the entity transport name, the record identifiers, the operation name, the
positional arguments and the keyword arguments — and the *Access Control* dialogue is returned.

**When** account 40 types the wrong password

**Then** it is refused with

> Incorrect Password, try again or click on Forgot Password to reset your password.

**When** account 40 types the right password

**Then** the confirmation timestamp is set to now, the stored call is deserialised, the operation is
asserted to be a guarded one, and it is invoked with the stored arguments; its result is returned.

### Scenario 16.3 — No request means no guarded operation

**When** a guarded operation is invoked outside a request (from a scheduled job, say)

**Then** it is refused with

> This method can only be accessed over Hypertext Transfer Protocol

---

## 17. Onboarding

### Scenario 17.1 — Completing a panel

**Given** a per-company panel with three steps, and no progress rows for company 1

**When** the panel is rendered for company 1

**Then** a progress row is created for company 1 and every step reads *Not done*.

**When** step 1 is completed

**Then** a step-progress row for company 1 is created in state *Just done*, linked to the panel's
progress row

**And** the panel's stored state stays *Not done*, because 1 ≠ 3.

**When** the panel is rendered again

**Then** step 1 is reported as *Just done* once and is then consolidated to *Done*

**And** a further rendering reports it as *Done*.

**When** steps 2 and 3 are completed and the panel is rendered

**Then** the stored state becomes *Done* and the overall render-time value is *Just done* for that
one rendering, then *Done*.

### Scenario 17.2 — Adding a step reopens the panel

**When** a fourth step is added to the panel

**Then** the progress rows recompute their step links and the stored state returns to *Not done*.

### Scenario 17.3 — Dismissing

**When** the panel is dismissed

**Then** the progress row's closed flag is set and the render-time overall value is *closed*

**And** toggling visibility inverts the flag.

### Scenario 17.4 — A step without an opening action

**When** a step with no opening action is linked to a panel

**Then** the save is refused with

> An "Opening Action" is required for the following steps to be linked to an onboarding panel:
> ['the step title']

### Scenario 17.5 — Changing the per-company flag

**When** a step's per-company flag is changed

**Then** every progress row of that step is deleted and the linked panels refresh their progress
rows, so completion restarts cleanly at the new granularity.

---

## 18. Self-service access to one's own account

### Scenario 18.1 — Reading one's own safe fields

**Given** account 42 is external and has no permission to read the User entity beyond the
external-user rows

**When** account 42 reads its own name, language, time zone, image and application keys

**Then** the read is performed with elevation and succeeds.

### Scenario 18.2 — Reading an unsafe field

**When** account 42 reads its own *Groups and implied groups* together with a field that is not in
the safe list

**Then** the read is performed **without** elevation and is subject to the ordinary rules.

### Scenario 18.3 — Writing one's own safe fields

**When** account 40 writes its own signature, language and time zone

**Then** the write is performed with elevation and succeeds.

### Scenario 18.4 — Writing a company one is not permitted in

**Given** account 40 is permitted in company 1 only

**When** account 40 writes its own default company as company 2, together with only safe fields

**Then** the company key is **silently dropped** and the remaining fields are written.

### Scenario 18.5 — Asking about another account's groups

**When** account 42 (external) asks whether account 40 belongs to a group

**Then** it is refused with

> You can ony call user.has_group() with your current user.

**And** the same question asked by account 40 (internal) about account 41 is answered.

---

## 19. Devices

### Scenario 19.1 — One log row per hour

**Given** account 40's session has a trace whose last activity is 1 200 seconds old

**When** a request arrives from the same platform, browser and address

**Then** no log row is written.

**When** a request arrives 3 600 seconds after the trace's last activity

**Then** the trace's last activity is advanced and one log row is written.

### Scenario 19.2 — A new device

**When** account 40 signs in from a different browser

**Then** a new trace is appended with first and last activity set to now and a log row is written

**And** the device list shows two rows.

### Scenario 19.3 — Revoking

**When** account 40 presses *Log out from all devices*

**Then** the identity re-check fires, and every device except the current one has its session
deleted from the store and its log rows marked revoked

**And** the current session survives.

**When** account 40 revokes the current device explicitly

**Then** the current session is signed out.

### Scenario 19.4 — Tracing disabled

**Given** a session carrying the tracing-disabled marker

**When** requests arrive

**Then** no device log row is ever written.

---

## 20. Directory and delegated sign-in

### Scenario 20.1 — Directory provisioning

**Given** no local account has the login `k.tanaka`, and a binding whose filter matches exactly one
directory entry for it, with account creation permitted and a template account set

**When** `k.tanaka` signs in with the correct directory password

**Then** the ordinary sign-in refuses, no local account exists with that login, the binding verifies,
and the template account is copied with the name taken from the entry's common-name attribute, the
login, the binding's company and, when the login is a valid address, that address

**And** the result reports the method *directory*.

### Scenario 20.2 — An archived local account blocks the directory

**Given** a local archived account with the login `k.tanaka`

**When** `k.tanaka` signs in with the correct directory password

**Then** the original refusal stands and the directory is never consulted.

### Scenario 20.3 — An ambiguous filter

**Given** the filter matches two directory entries

**Then** verification fails and the binding is skipped.

### Scenario 20.4 — An empty password

**Given** the directory would accept an anonymous bind with a valid distinguished name

**When** an empty password is supplied

**Then** the binding fails immediately, before any bind is attempted.

### Scenario 20.5 — A directory password change

**When** a user changes their password and a binding accepts the change

**Then** the change is made in the directory and the **local** password column is emptied, so the
local copy cannot be used.

### Scenario 20.6 — Delegated sign-in provisioning

**Given** a provider whose information address returns a payload with a subject under `sub` and an
address

**And** no account with that provider and subject

**And** self-registration open

**When** the visitor returns with a valid token

**Then** an account is provisioned with the payload's name, address and login, the provider, the
subject and the token, and is signed in.

**When** the payload carries no subject under any of `sub`, `id` or `user_id`

**Then** the flow is refused with

> Missing subject identity

**And** when self-registration is closed, the return address redirects to the sign-in page with the
marker that renders as *Sign up is not allowed on this database.*

### Scenario 20.7 — Provider-subject uniqueness

**When** two accounts are given the same provider and the same subject

**Then** the second write is refused with

> OAuth UID must be unique per provider
