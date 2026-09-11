# Identity and Access — Workflows

This file specifies the end-to-end operational procedures of the domain. Each workflow gives the
role that performs it, the preconditions, the numbered steps, what is created or updated at each
step, and the failure points with their exact messages.

> **Note on reproduced texts.** Message texts, button labels and screen titles are reproduced
> **verbatim**, exactly as the system emits them, because external tests and user documentation
> depend on them. Where such a reproduced text contains an abbreviation, the abbreviation belongs to
> the text and is not this document's own prose. The abbreviations that occur are: *API* for
> application programming interface; *2FA* for two-factor authentication; *LDAP* for the central
> directory protocol; *DN* for distinguished name; *OAuth* and *UID* for the delegated sign-in
> protocol and its subject identifier; *JSON* for the structured-literal notation; *HTTP* for the
> transport protocol; *ID* for identifier. Outside such reproduced texts, every term is written in
> full.

Cross-references: the algorithms invoked here are specified in
[business-rules.md](business-rules.md); the arithmetic in
[calculations.md](calculations.md); the states in [state-machines.md](state-machines.md).

---

## 1. Signing in with a login and a password

**Role**: anyone. **Precondition**: an account with that login exists and is active.

1. The visitor opens the sign-in page. The page is served with the caching directive *no cache*,
   with framing restricted to the same origin, and with a content policy allowing only same-origin
   framing ancestors.
2. The page is rendered with: the request parameters that belong to the sign-in parameter set; the
   list of databases when listing is permitted; whether self-registration is open; whether password
   reset is offered; and, when the session remembers a pending login, that login pre-filled.
3. The visitor submits the login and the password.
4. The controller assembles a credential from the parameters `login`, `password` and `type`, keeping
   only non-empty values, and defaults the type to `password`.
5. If the installation demands a challenge-response test for this sign-in, the test is verified
   first; a failure ends the flow.
6. The session's authentication procedure runs (section 9.4 of
   [business-rules.md](business-rules.md)).
7. On success the visitor is redirected to the post-sign-in destination, which is the requested
   destination when one was given, and otherwise the back office for an internal account or the
   customer-facing landing page for any other.
8. On failure the page is re-rendered with an error: *Wrong login/password* when the refusal carries
   no message of its own, and the refusal's own text otherwise (which is how the cooldown message and
   the directory messages reach the user).
9. A signed-in user who opens the sign-in page with a destination parameter is redirected to that
   destination instead of seeing the form.

**What is created or updated**: one sign-in log row; possibly the account's time zone; possibly a
re-hashed password; the session's account identifier, login, database, preference context and
session token; possibly the system parameter `web.base.url`.

---

## 2. Signing in when a second factor is required

**Role**: anyone with the second factor enabled, or anyone at all when the enforcement policy
demands it.

1. Steps 1 to 6 of workflow 1 run. The first factor succeeds; the result's second-factor policy is
   `default`; the account has a second-factor address.
2. The session becomes *Pending*: the pending login and the pending account are stored and the
   account identifier is cleared.
3. The visitor is sent to the second-factor page.
4. **Trusted-browser shortcut.** On the initial display the page reads the trusted-browser cookie.
   If it verifies as a key in the `browser` purpose for the pending account, the session is
   finalised immediately and the visitor is redirected. No code is asked for.
5. **Mailed code.** When the account's second-factor type is the mailed one, opening the page also
   mails a code. This is done inside a nested savepoint; a rate-limit refusal, a missing address or
   any other failure is shown as the page's error instead of aborting. Sending is refused outright
   unless the session is genuinely in the pending phase.
6. The visitor types the code and submits.
7. The cooldown guard is entered, keyed on the pending account identifier.
8. A credential of the account's second-factor type is assembled with the code stripped of white
   space and parsed as a whole number. A non-numeric value produces *Invalid authentication code
   format.*
9. The credential is checked interactively. The rate limiter is consumed first; see section 11 of
   [business-rules.md](business-rules.md) for the full sequence and the two distinct refusal
   messages.
10. On success the session is finalised, the request environment is re-attached to the now-known
    account, and the visitor is redirected.
11. If "remember this browser" was ticked, a scoped key is generated with elevation, expiring at the
    current moment plus the trusted-browser age, and set as a cookie with that maximum age, marked
    inaccessible to scripts, with the same-site policy *lax*. The key's label is composed from the
    browser, the platform and, when resolvable, the city and country.
12. The session is touched in every branch so that it is persisted.

**Security notice.** When the second factor is enabled and the sign-in did **not** come from a
trusted browser, a notice is mailed to the account: subject *New Connection to your Account*, body
*A new device was used to sign in to your account.*, rendered in the account's language.

---

## 3. Signing in with a passkey

**Role**: anyone who has registered a passkey.

1. The sign-in page offers the passkey option. The client asks the server for authentication
   options.
2. The server generates them for the relying party identified by the **host** of the configured base
   address, demanding user verification, and stores the challenge in the session under the
   passkey-challenge key. The options are returned to the client.
3. The browser or password manager produces an assertion and the client submits it as a credential
   of the passkey type.
4. The sign-in procedure resolves the account: the assertion's credential identifier is looked up
   directly in the credential table joined to the accounts, and the found login is placed into the
   credential. An unknown identifier raises *Unknown passkey*.
5. The ordinary sign-in path continues with that login: the cooldown guard, the account search, the
   credential check.
6. The credential check fetches the credential row for the acting account and that identifier with
   elevation. None found: *Unknown passkey*.
7. The assertion is verified against the challenge popped from the session, the expected origins
   (the base address with its path removed, plus the accepted mobile application signing-key
   hashes), the relying-party identifier, the stored public key, the stored signature counter, and
   the user-verification requirement. A failure surfaces as a refusal carrying the verifier's own
   message. A missing challenge produces *Cannot find a challenge for this session*.
8. The stored signature counter is replaced by the new one.
9. The result reports the method *passkey* and the policy **skip**, so the session is finalised
   immediately: **no second factor is asked for, whatever the account's configuration**.

---

## 4. Signing in through an external identity provider

**Role**: anyone, when at least one provider is enabled.

1. The sign-in page lists the enabled providers, each rendered with its label and its presentation
   class, ordered by sequence then name.
2. The visitor is sent to the provider's authorisation address with this installation's client
   identifier, the configured scope, and an opaque state that carries, among other things, any
   sign-up token under the key `t`.
3. The provider returns the visitor with a bearer token.
4. The server validates the token (section 22.3 of [entities.md](entities.md)): the information
   address is called, the response is parsed, an extra data address is merged when configured, and
   the subject identity is unified into one key. A missing subject produces *Missing subject
   identity*.
5. The server looks for an account with that provider and that subject.
   - **Found**: the stored token is replaced and the login is returned.
   - **Not found and account creation is forbidden in this context**: the flow yields nothing and
     the sign-in is refused.
   - **Not found otherwise**: provisioning values are composed (name, login and address from the
     payload, with a synthetic address when none is supplied; the provider; the subject; the token;
     active) and the registration path is invoked with the token taken from the opaque state. A
     registration refusal is converted back into the original access refusal.
6. The session is established with the returned login and the token as the credential.

**Note.** The provider token can also be presented later as a credential in its own right, which is
what lets a program that holds it act as the account (section 22.5 of
[entities.md](entities.md)).

---

## 5. Signing in through the central directory

**Role**: anyone whose account lives only in the directory.

1. The ordinary sign-in runs and refuses.
2. The system checks whether an account with that login, lower-cased, exists locally. **If one
   does, the original refusal stands** — the directory is never consulted for a login already known
   locally, so an archived local account cannot be resurrected through the directory.
3. Otherwise each directory binding, in sequence order, is asked to verify the credentials:
   1. An empty password fails immediately.
   2. The search filter is formatted with the login substituted into every placeholder, escaped for
      the filter language, and the subtree of the search base is searched with a 60-second timeout,
      binding first with the query account or anonymously.
   3. Entries without a distinguished name are discarded. Exactly one remaining entry is required.
   4. A fresh connection binds with that distinguished name and the supplied password.
4. The first binding that verifies provisions or finds the account (section 23.5 of
   [entities.md](entities.md)) and the result reports the method *directory* and the policy
   `default`.
5. If no binding verifies, the original refusal stands.
6. When no local account exists and the binding forbids creation, the refusal is *No local user
   found for LDAP login and not configured to create one*.

---

## 6. Calling the system with an application key

**Role**: an integration.

1. An internal user creates a key (workflow 17) and stores the clear value in the integration's
   configuration.
2. The integration sends a request to a route whose authentication mode is `bearer`, with the header
   `Authorization: bearer <the clear key>`.
3. The token is extracted case-insensitively and verified in the global purpose. A failure answers
   `Invalid apikey` with a bearer challenge.
4. If the request also carries a session with a different account, the call is refused with
   `Session user does not match the used apikey.`
5. The environment is attached to the key's account and the session is marked **not savable**,
   because the call is stateless.
6. The `user` rule is applied, so an anonymous account is refused.
7. Every access decision in the call is taken as that account.

**Alternative.** On a non-interactive connection that authenticates with a login and a password on
each call, the key may be supplied **in place of the password**; the credential check tries the
password first and the key second. When the account has the second factor enabled, only the key is
accepted.

---

## 7. Switching the active company

**Role**: any internal user permitted in more than one company.

1. At sign-in the client receives the company description: the current company (the user's default
   company), the permitted companies with their identifiers, names, sequences, parents, visible
   children and currencies, and the disallowed ancestor companies with the same fields minus the
   currency.
2. The user selects a different set in the company switcher. The client sends the new ordered list
   of active company identifiers with every subsequent call.
3. On each call, the first read of the environment's company or companies validates the list: every
   identifier must be among the user's permitted, active companies, otherwise the call is refused
   with *Access to unauthorized or invalid companies.*
4. The first element becomes the **current** company: it is the default company of every record
   created in that call, and the company used to resolve company-dependent values.
5. The whole list becomes the **active** set: the standard multi-company record rule uses it, so
   lists and searches immediately change.
6. Because the list is part of the record-rule cache key, a different list selects a different
   cached filter rather than invalidating anything.
7. Operations that need whole companies rather than a selection use the accessible-branch
   computation (section 12.7 of [entities.md](entities.md)) or the all-branches predicate.

A fully worked example, including the failure modes, is in section 8.6 of
[business-rules.md](business-rules.md).

---

## 8. Creating an internal user and granting permissions

**Role**: a holder of *Role / Administrator* (to reach the settings screens) or of *Access Rights*.

1. The administrator opens the user list and creates a record, supplying at least a name and a
   login.
2. The default groups are applied: *Role / User* plus every group directly implied by *Default
   access for new users*.
3. A contact is created or attached; the contact's company follows the account's default company
   unless the contact is global; the contact's active flag follows the account's.
4. When the account is internal and has no image, an initials avatar is generated from the name.
5. A settings row is created for the account, with elevation, when the account is internal.
6. The multi-company group membership is adjusted mechanically from the number of permitted
   companies.
7. The administrator sets the permitted companies and the default company. The default must be among
   the permitted, otherwise *Company … is not in the allowed companies for user … (…).*
8. The administrator sets the application permissions. On the form these are presented as one
   selector per **privilege**, grouped by application category; choosing a value writes the
   corresponding group into the explicit groups. The *Role* selector swaps *Role / User* and *Role /
   Administrator* without touching anything else.
9. The system validates that the closure contains at most one of the three kind groups, otherwise
   *User '…' cannot be at the same time in exclusive groups '…', '…'.*
10. The system validates that at least one account still holds *Role / Administrator*, otherwise
    *You must have at least an administrator user.*
11. Writing the groups clears the access caches for everyone.
12. Because the account has an electronic mail address and the creation was not marked "no password
    mail", a sign-up invitation is prepared on the contact and mailed. A delivery failure cancels
    the invitation rather than leaving it dangling.
13. Five days later, if the account has still never signed in, the reminder job mails its creator.

---

## 9. Granting a contact access to the customer-facing pages

**Role**: an internal user who can write contacts and create users.

1. The user selects one or more contacts and opens the portal access wizard. The selection is
   **expanded**: for each selected contact, the contact itself plus every child of kind *contact* or
   *other*.
2. One line is produced per contact, pre-filled with that contact's electronic mail address.
3. Each line computes:
   - the contact's first account, archived ones included;
   - whether that account is internal (**even if archived**) or an active external one;
   - the address status: *Invalid* when the address does not normalise; *Already Registered* when
     another account already has that address as its login; *Valid* otherwise.
4. The user may correct an address in place.
5. The user presses **Grant access** on a line.
   1. The address is validated: *The contact "…" does not have a valid email.* or *The contact "…"
      has the same email as an existing user*.
   2. Granting to a contact that is already external or internal is refused: *The partner "…"
      already has the portal access.*
   3. If the typed address is valid and differs from the contact's, the contact's address is
      updated.
   4. If the contact has no account, one is created **from the external-user template** in the
      contact's company or, failing that, the acting company, with the normalised address as both
      login and address, with elevation and the "no password mail" marker.
   5. The account is written active, given *Role / Portal* and stripped of *Role / Public*.
   6. A sign-up invitation is prepared on the contact.
   7. The invitation message is sent forcibly and synchronously, in the account's language, carrying
      the wizard's invitation message and marked with the medium *portal invitation*. A missing
      template raises *The template "Portal: new user" not found for sending email to the portal
      user.*
6. The user may press **Revoke access**: refused unless the contact is currently external (*The
   partner "…" has no portal access or is internal.*); otherwise the contact's sign-up type is
   cleared and the account is archived. The account keeps *Role / Portal*; it is **not** moved to
   *Role / Public*, which is reserved for automated tasks and guests.
7. The user may press **Invite again**: refused unless the contact is currently external (*You
   should first grant the portal access to the partner "…".*); otherwise a fresh invitation is
   prepared and sent.
8. After each action the wizard re-opens so several contacts can be handled in one sitting.

---

## 10. Sharing a document with an external party

**Role**: any internal user who can read the document.

1. The user opens the share dialogue from the document. The dialogue is seeded with the document's
   entity and identifier from the acting context, and computes the share link.
2. Computing the link requires the document's entity to adopt the portal mixin; otherwise the link
   is empty.
3. The link is the absolute base address plus the generic mail-view path plus a query carrying the
   entity name, the record identifier and the access token. Obtaining the token first checks the
   acting user's **read** access to the document, then generates and writes the token with elevation
   if there is none.
4. The user selects recipients and optionally adds a note.
5. On sending, the recipients are split:
   - if the document already has a token, **or** self-registration is not open, every recipient gets
     the plain share message;
   - otherwise only recipients that already have an account get the plain share message;
   - the rest each get a message carrying a **registration** link that, after registration, redirects
     to the document.
6. Each message is composed in the recipient's language and posted on the document as an internal
   note with the light notification layout, addressed to that single recipient, with the subject
   *Invitation to access <the document's display name>*. The link embedded for that recipient also
   carries the recipient's contact identifier and its signature.
7. The recipient opens the link. The route's authentication mode is `public`, so the environment is
   the anonymous account. The document check (section 12.1 of
   [business-rules.md](business-rules.md)) either raises *This document does not exist.*, or
   re-raises the read refusal when the token is absent or wrong, or returns the document fully
   elevated.
8. The page renders the document. When the recipient signature verifies, messages posted in the
   discussion thread are attributed to that contact rather than to an anonymous visitor.

---

## 11. Registering through an invitation

**Role**: an invited contact.

1. The contact receives the invitation message containing a registration link with a token.
2. Opening the link renders the registration page. The page first resolves the token to pre-fill the
   form: the contact's name, and either the existing account's login or the contact's address as the
   proposed login. A token that will not resolve sets the page's error to *Invalid signup token* and
   marks the token invalid.
3. If there is **no** token **and** self-registration is not open, the page does not exist at all.
4. The visitor fills in the login, the name and the password twice, and submits.
5. The submitted values are prepared: login, name and password are taken; the form being empty
   raises *The form was not properly filled in.*; a mismatch raises *Passwords do not match; please
   retype them.*; the acting language is carried over when it is installed.
6. Registration runs (section 13.3 of [business-rules.md](business-rules.md)):
   - the contact is resolved from the token and the token type is cleared;
   - geolocated city, country and language are dropped when the contact already has them;
   - if the contact already has an account, the login and name are dropped and the remaining values
     — in practice the password — are written to it; a never-signed-in internal account additionally
     notifies its inviter over the live channel;
   - otherwise an account is created from the external-user template.
7. The work is committed, because what follows may fail and the account must survive.
8. The visitor is signed in with the new credentials unless a second factor intervenes; in that
   case the environment is switched to the anonymous account so the page can still be rendered.
9. An account-created confirmation message is sent to the new account, forcibly and synchronously,
   when the template exists.
10. The sign-in flow resumes and the visitor lands on the post-sign-in destination. A non-internal
    account that has just been created lands on the customer-facing landing page with a marker
    saying the account was created.
11. Failures surface as the page's error: the message of a user-facing refusal; *Another user is
    already registered using this email address.* when a registration error coincides with an
    existing login; and otherwise *Could not create a new account.* followed by a line break and the
    underlying message, with the underlying message also logged as a warning.
12. A visitor who arrives with only a proposed address and whose address already belongs to a
    confirmed account is redirected to the sign-in page with that login pre-filled.

A fully worked example, including how the token dies, is in section 13.6 of
[business-rules.md](business-rules.md).

---

## 12. Resetting a password

**Role**: anyone with an account.

1. The visitor opens the password-reset page. If there is no token **and** password reset is not
   enabled (the parameter `auth_signup.reset_password` is not the text `True`), the page does not
   exist.
2. **Requesting.** The visitor types a login and submits. The attempt is logged with the login, the
   acting account and the remote address. The reset procedure searches by login, then by electronic
   mail address; *No account found for this login* and *Multiple accounts found for this login* are
   the two refusals. On success the page shows *Password reset instructions sent to your email
   address.*
3. The reset action prepares an invitation of type `reset` on the account's contact, computes a
   token with the reset validity (default 4 hours) and mails it. An account with no address raises
   *Cannot send email: user … has no email address.*; an archived account raises *You cannot perform
   this action on an archived user.*
4. **Using.** The visitor opens the link. The token resolves and the form is pre-filled.
5. The visitor types a new password twice and submits. The registration path runs **without**
   signing the visitor in, and the page shows *Your password has been reset successfully.*
6. The password write runs the strength policy; a failure surfaces as the page's error.
7. Delivery failures are converted into readable messages: *Could not contact the mail server, please
   check your outgoing email server configuration* for a refused connection and *There was an error
   when trying to deliver your Email, please check your configuration* otherwise.
8. A well-known path exists that simply redirects to the password-reset page, so that password
   managers can offer a change-password shortcut.

---

## 13. Removing one's own external account

**Role**: an external user.

1. On the customer-facing security page the user opens the deactivation dialogue, types their own
   login as a confirmation and their password.
2. A confirmation that does not equal the login is reported as a validation error.
3. The password is checked interactively; a failure is reported as a password error.
4. The removal runs:
   1. Every selected account must be external, otherwise *Only the portal users can delete their
      accounts. The user(s) … can not be deleted.*
   2. The request is logged with the login, the identifier and the remote address.
   3. The login is replaced by `__deleted_user_<identifier>_<the current moment as a decimal number
      of seconds>` and the secret is cleared.
   4. Every application key of the account is removed.
   5. A deletion request in state *To Do* is created.
   6. The account is archived as the account of identifier 1 — a user cannot archive themselves —
      and the contact is archived; both failures are swallowed, because the queue entry is what
      matters.
5. The session is signed out and the visitor is redirected to the sign-in page with the message
   *Account deleted!*
6. The scheduled job later performs the real deletion (workflow 14).

---

## 14. Processing the account deletion queue

**Role**: a scheduled job.

1. Select every request in state *To Do*.
2. Requests whose account link is empty — the account is already gone — are set to *Done* in bulk
   and counted as progress.
3. The remaining requests are processed in batches of **50**.
4. For each request: take a row-level lock for update and re-test that the state is still *To Do*;
   skip it if not.
5. Remember the account's name, its contact and the name of the original requester.
6. **Delete the account.** On success, log the deletion naming the account identifier, the account
   name and the requester; set the request to *Done*; count one unit of progress.
7. On failure, roll the transaction back, log an error naming the same three things and the
   underlying error, set the request to *Failed*, count one unit of progress, and continue to the
   next request if the progress budget allows, otherwise stop.
8. **Delete the contact.** On success, log it and continue while the budget allows. On failure, roll
   back, log a warning, and continue while the budget allows. A contact that is referenced by a
   document — an invoice, for example — cannot be deleted, and this is expected: the account is gone
   and the contact remains as a historical reference.

---

## 15. Saving a settings screen

**Role**: a holder of *Role / Administrator* or *Access Rights*.

1. The administrator opens a settings screen. The record's default values are produced (section 16.3
   of [entities.md](entities.md)): user-defined defaults are read; group checkboxes are computed by
   testing whether the implied group is in the closure of **every** holder group; package checkboxes
   are computed from the package state; parameter fields are read and converted.
2. The administrator changes some values and presses save.
3. The save is refused unless the acting user is an administrator: *Only administrators can change
   the settings*.
4. The archive filter is disabled for the whole save, so archived groups and packages are still
   found.
5. The fields are classified (section 16.2 of [entities.md](entities.md)) and the current state is
   re-read for comparison.
6. **Defaults** are written for the fields that changed.
7. **Groups** are processed in ascending order of their value, so every removal happens before every
   addition. A true value applies the implied group to the holder groups; a false value removes it
   from the whole closure of the holder groups.
8. **Parameters** are written for the fields that changed, with the type conversions of section 16.4
   of [entities.md](entities.md). Note in particular that a text value is trimmed and that an empty
   result **deletes** the parameter.
9. **Packages**: those to install and those to uninstall are collected. If either set is non-empty,
   everything is flushed.
10. If anything must be uninstalled, the uninstall confirmation dialogue is returned and the flow
    ends here. The installations are **not** performed in this call.
11. Otherwise the installations run. If anything was installed, the transaction is reset because the
    registry has changed.
12. The next configuration step is asked for; unless it is a plain close, it is returned. Otherwise
    the client is told to reload so that menus and the current screen pick up the new state.

**Why the ordering of step 7 matters.** Consider two checkboxes of the same holder group, one
switching off *Sales / User: all documents* and one switching on *Sales / Administrator*. If the
addition ran first, the removal — which reaches into the whole closure — would strip the group that
the addition had just implied. Processing false before true avoids this.

---

## 16. Enrolling a second factor

**Role**: any user, for their own account.

1. The user opens their preferences and presses the enable action.
2. The identity re-check fires unless the identity was confirmed within the last ten minutes. The
   suspended call is stored and the *Access Control* dialogue opens; the user types their password;
   the confirmation timestamp is set; the suspended call is replayed.
3. Enabling for anyone but oneself is refused: *Two-factor authentication can only be enabled for
   yourself*. Enabling when already enabled is refused: *Two-factor authentication already enabled*.
4. A 160-bit secret is drawn, rendered in base 32 and grouped in blocks of four.
5. A setup wizard record is created carrying the account and the secret; the enrolment address and
   the enrolment image are computed from them.
6. The dialogue shows the image and the secret. The user adds the account to an authenticator
   application and types the code it shows.
7. The identity re-check fires again on the confirm action.
8. The code is stripped of white space and parsed; a non-numeric value raises *The verification code
   should only contain numbers*.
9. The enrolment attempt runs: the secret is compressed and upper-cased, the code is matched, and on
   success the secret and the matched counter are stored.
10. The session token is recomputed and written into the current session, so the enrolling user is
    not signed out by their own change. Every **other** session of the account becomes invalid.
11. The wizard's copy of the secret is blanked.
12. A success notice is shown: *2-Factor authentication is now enabled.*
13. A security notice is mailed: subject *Security Update: 2FA Activated*, body *Two-factor
    authentication has been activated on your account*.
14. A failed code raises *Verification failed, please double-check the 6-digit code* and the secret
    is not stored.

**Disabling** is the mirror image: the identity re-check; the permission test (self, administrator
or elevated); revoke every trusted browser; clear the secret; refresh the session token when
disabling one's own; show the warning notice naming the affected accounts; mail the notice *Security
Update: 2FA Deactivated*.

**Inviting** users to enable it: an administrator selects accounts and triggers the invitation,
which mails the invitation template — forcibly, with the light notification layout, from the
administrator's own address and authorship — to every selected account that has no secret, and
shows an informational notice naming them.

---

## 17. Creating an application key

**Role**: an internal user.

1. The user opens their preferences and presses the new-key action.
2. The identity re-check fires; on success the description dialogue opens.
3. The duration choices are computed from the user's maximum duration (section 3.3 of
   [calculations.md](calculations.md)); a system user additionally sees the persistent choice.
4. The user types a label and picks a duration. The expiry is derived; picking the custom choice
   lets the user type a date, which is validated immediately and, when invalid, produces a
   non-blocking notice titled *The API key duration is not correct.* carrying the validation
   message.
5. Creating the dialogue record validates the expiry outright.
6. The user confirms. The identity re-check fires again.
7. Non-internal users are refused: *Only internal users can create API keys*.
8. The key is generated (section 3.1 of [calculations.md](calculations.md)); the dialogue record is
   deleted; the generation is logged with the purpose, the login, the identifier and the remote
   address.
9. The clear key is shown once in a display dialogue titled *API Key Ready*. It cannot be retrieved
   afterwards.

**Removal**: the identity re-check fires; removal is permitted to a system user or to the owner,
otherwise *You can not remove API keys unless they're yours or you are a system user*; the rows are
deleted with elevation and the caches are cleared.

---

## 18. Re-authenticating after inactivity

**Role**: any user whose groups set an inactivity limit.

1. The client reports inactivity over the live channel, or the channel closes.
2. The *next identity check* moment is written into the session (section 14.6 of
   [business-rules.md](business-rules.md)) and the session is saved explicitly.
3. The user returns and issues a request to a route whose mode is `user`.
4. The re-authentication predicate fires. For a page request the response is a redirect to the
   re-authentication page carrying the original address as the destination; for a programmatic call
   the condition is raised so the client shows the dialogue in place.
5. The dialogue asks the server what methods are available with no credential supplied; the answer
   carries the account identifier, the login and the list of methods, with the already-used first
   factor removed when one is recorded.
6. The user supplies a credential. A type that is not among the available methods is refused
   outright.
7. The credential is checked interactively. For the mailed second factor, the user may first ask
   for a code to be sent; that request goes through the send rate limiter.
8. If the limit demands a second factor and more than one method is available and the result's
   policy is not `skip`, the first factor is recorded in the session, its method is removed from the
   list, and the dialogue asks for the second.
9. When the exchange completes, the *next identity check* moment is removed and the *last identity
   check* moment is set to now. The original request is retried.
10. Some routes opt out of identity checking so they can still be served while the screen is
    locked — notably the menu-loading route, which is fetched by the client outside its ordinary
    call machinery and therefore cannot handle the re-authentication condition.

When the **session** limit rather than the inactivity limit fires, there is no dialogue: the session
is treated as expired and the user is sent back to the sign-in page.

---

## 19. Revoking devices

**Role**: any user, for their own devices.

1. The user opens their security preferences and sees the device list: one row per (session,
   platform, browser) combination, newest activity first, with the current device marked.
2. The user presses **Revoke** on a row, or **Revoke all** to clear every device but the current
   one.
3. The identity re-check fires.
4. The sessions with those session identifiers are deleted from the session store.
5. Every device-log row with those identifiers is marked revoked, with elevation.
6. The action is logged naming the acting user and the identifiers.
7. If the current device was among them, the current session is signed out.

Separately, a housekeeping pass marks as revoked every non-revoked row whose last activity is older
than the maximum session inactivity and whose session no longer exists in the store, in batches of
100 000 with a commit between batches; and a de-duplication pass deletes superseded rows.

---

## 20. Running a recycling rule

**Role**: an administrator, or the scheduled job.

1. **Collection.** For every active rule (or for the selected rule when run by hand):
   1. Build the filter: the rule's stored filter, intersected with the date condition when a time
      field, a non-zero delta and a delta unit are all set (section 12.1 of
      [calculations.md](calculations.md)).
   2. Search the rule's entity, with the archive filter disabled when the rule includes archived
      records.
   3. Exclude records that already have a candidate row for this rule, active or discarded.
   4. In **automatic** mode, create and immediately validate the candidates in batches of 5 000,
      committing after each batch outside tests.
   5. In **manual** mode, accumulate the candidates and create them in batches of 50 000,
      committing after each batch outside tests.
2. **Notification.** For every manual rule with notified users and a non-zero frequency: if the
   last-notification moment is absent or older than the frequency, set it to now and send a notice
   counting the candidates created within the last frequency period. When the count is zero, no
   notice is sent.
3. **Validation by an operator.** The operator opens the candidate list, reviews the record names
   and companies, and presses validate on a selection: the pointed-at records are archived or
   deleted with elevation according to the rule's action, records that no longer exist are skipped,
   and the candidate rows are deleted.
4. **Discarding.** The operator presses discard: the candidate's active flag is cleared and the
   record is never proposed again by this rule.
5. **Deactivating a rule** deletes every candidate it produced, without touching the pointed-at
   records.

A fully worked example with numbers, including the ninety-day case, is in section 12.3 of
[calculations.md](calculations.md).

---

## 21. Running a personal-data search

**Role**: a user who can reach the privacy menu.

1. The operator opens the search, either from the menu or from a contact (in which case the name and
   address are pre-filled from that contact).
2. The operator supplies a name and an electronic mail address and presses search.
3. The address is normalised; a value that will not normalise raises *Invalid email address "…"*.
4. The composite query of section 11.2 of [calculations.md](calculations.md) is built and run, and
   its rows become the wizard's lines.
5. The line list opens. For each line: the record name is resolved with elevation; the reference is
   resolved only when the acting user may read the record, so a record hidden by a multi-company
   rule shows as a line without a link rather than raising; whether the entity supports archiving is
   computed.
6. The operator acts per line or in bulk:
   - toggling the active flag archives or un-archives the pointed-at record with elevation and
     records *Archived <entity label> #<identifier>* or *Unarchived <entity label> #<identifier>*;
   - deleting removes the pointed-at record with elevation, records *Deleted <entity label>
     #<identifier>* and marks the line deleted; a second attempt raises *The record is already
     unlinked.*;
   - the bulk actions skip lines that are already in the target state.
7. Every change to the execution details recomputes them on the wizard, and that recomputation
   writes the log: a log is created the first time there is anything to record, with the name and
   address **masked** (section 13 of [calculations.md](calculations.md)); afterwards the existing
   log's details and record description are overwritten.
8. The operator may add a free note to the log afterwards.

---

## 22. Completing a guided-setup panel

**Role**: any user who sees the panel.

1. The panel is rendered for the acting company. Rendering asks the current progress row for the
   per-step states and the overall state.
2. If no progress row exists for the acting company, one is created — with the company when the
   panel is per-company, without it otherwise — linked to the step-progress rows that already exist
   for that company.
3. Each step shows its title, description, image and button. Pressing the button runs the step's
   opening action.
4. When the action completes, the step is marked as completed: a progress row is created if needed
   (linked to every progress row of every panel containing the step and matching the company), and
   the state becomes *Just done*.
5. The next rendering consolidates: every *Just done* row of that panel becomes *Done*, and the
   overall state is reported as *Just done* once — so the completion animation is shown exactly
   once — and as *Done* thereafter.
6. The panel's own state is stored on the progress row and recomputes whenever the step states or
   the panel's step set change: *Done* when the number of steps in *Just done* or *Done* equals the
   number of steps of the panel, *Not done* otherwise.
7. Dismissing the panel sets the closed flag on the current progress row; the render-time overall
   state becomes *closed*. Toggling visibility inverts the flag.
8. Changing a step's per-company flag deletes the step's progress rows and refreshes the panels'
   progress rows, so the completion state restarts cleanly at the new granularity.

---

## 23. Creating a company or a branch

**Role**: a holder of *Role / Administrator*.

1. The administrator creates a company, supplying at least a name, and optionally a parent.
2. A contact is created for it when none was supplied, marked as an organisation and carrying the
   name, image, address, telephone number, website, tax identifier and country; the contacts are
   flushed so their computed address fields exist.
3. For a branch, every root-delegated field absent from the proposal is copied from the parent. The
   currency is the only such field in the foundation.
4. The registry caches are cleared and the rows are created.
5. The creating user and the system account gain the new company in their permitted companies.
6. Any archived currency used by a new company is activated.
7. A company with a country triggers the installation of that country's localisation packages,
   unless the environment is a test run, an import or a package-installation pass.
8. Constraints: the name must be unique (*The company name must be unique!*); a branch's delegated
   fields must equal the root's (*The … of a subsidiary must be the same as it's root company.*).
9. The parent can never be changed afterwards (*The company hierarchy cannot be changed.*) and the
   company cannot be duplicated (*Duplicating a company is not allowed. Please create a new company
   instead.*).
10. Writing a root-delegated field on a root copies it down to every branch.
11. Archiving a company archives its branches and is refused while any active account has it as its
    default company.

---

## 24. Changing a password

Three paths exist and they are deliberately different.

### 24.1 A user changing their own password from the back office

1. Preferences → change password. The identity re-check fires.
2. The dialogue asks for the new password twice; a mismatch raises *The new password and its
   confirmation must be identical.*
3. The identity re-check fires again on confirm.
4. Every trusted browser of the account is revoked.
5. The internal change runs: trim, refuse empty (*Setting empty passwords is not allowed for
   security reasons!*), log, write.
6. The write runs the strength policy and then hashes.
7. The wizard record is deleted and the client is told to reload, so the session picks up the new
   token instead of being rejected.

### 24.2 A user changing their own password from the customer-facing pages

1. The security page posts the old password and the new one twice.
2. Any empty value: *You cannot leave any password empty.*
3. A mismatch: *The new password and its confirmation must be identical.*
4. The old password is verified. A generic refusal becomes *The old password you provided is
   incorrect, your password was not changed.*; a specific one is shown as-is.
5. A policy failure is shown as-is.
6. On success the session token is recomputed and written into the session, so the user stays signed
   in.
7. When a directory binding accepts the change, the change is made **in the directory** and the
   local password column is emptied.

### 24.3 An administrator changing someone else's password

1. Select accounts, open the change-password wizard. One line per account, pre-filled with the
   login.
2. Type the new passwords and apply. Each non-empty line performs the internal change on its
   account.
3. Immediately afterwards every line's new password is blanked, so clear values do not linger.
4. If the acting user is among the affected accounts, the client is told to reload; otherwise the
   dialogue simply closes.
5. Writing the *Set Password* field directly on one's **own** account is refused: *Please use the
   change password wizard (in User Preferences or User menu) to change your own password.*

---

## 25. Diagnosing a refused operation

**Role**: an administrator helping a user.

1. Reproduce the refusal and read the message. An entity-level refusal begins *You are not allowed
   to …* and lists the groups that hold the operation. A record-level refusal begins *Uh-oh! Looks
   like you have stumbled upon some top-secret records.*
2. For an entity-level refusal, grant one of the listed groups, or create an access-right row. The
   caches clear automatically.
3. For a record-level refusal, switch the acting user's session into developer mode and reproduce:
   the detailed form of the message then names the failing records and the failing rules.
4. If the message suggests a company problem, check the acting user's active-company list and
   permitted companies: the message itself distinguishes the three cases (several candidate
   companies; one candidate the user may switch to; one candidate the user may not reach).
5. On the user form, the counters *# Groups*, *# Access Rights* and *# Record Rules* and their three
   actions open, respectively, the closure, every access-right row attached to it and every rule
   attached to it — all read-only.
6. On a group's form, the action *Users and implied users of …* lists every account that reaches the
   group, directly or through implication.
