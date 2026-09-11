# Workflows of the Contacts and Organizations domain

End-to-end operational procedures, step by step, with the role that performs each one, its
preconditions, the records it creates or updates at each step, and its failure conditions. Every
algorithm referenced here is specified in full in [calculations.md](calculations.md); every
validation in [business-rules.md](business-rules.md).

**Roles used in this file.**

| Role | Membership |
|---|---|
| Internal user | the internal-user group; may read the directory, may not change it |
| Directory maintainer | the contact-creation group; may create, update and delete parties, tags, banks and bank accounts |
| Administrator | the administrator group; additionally may edit countries, currencies, languages, industries and blocked numbers, and may merge parties whose electronic mail addresses differ |
| Accounts manager | the access-rights-management group; additionally may create and edit companies |
| Portal visitor | a signed-in external counterparty; may read their own commercial entity's subtree |
| Public visitor | not signed in; may read published public pages |

---

# 1. Create an independent organization

**Role**: directory maintainer.
**Preconditions**: none.

1. Open the contacts list and start a new record. The window action supplies the default
   organization flag **set**, so the new record starts as an organization.
2. Type the name. The form shows the organization-oriented placeholder.
3. Optionally fill the electronic mail address, the telephone, the website, the language, the tax
   registration number and the tags.
4. Fill the address block. Its field order follows the layout of the **acting company's** country,
   not of the country being typed (see [calculations.md](calculations.md) §6.5).
5. Save.

**What happens on save.**

| Step | Effect |
|---|---|
| Website normalisation | a value with no scheme acquires `http://` |
| Language computation | the record has no parent and no language was given → the language becomes the user-defined default, or failing that the session language |
| Insert | the row is written |
| Complete name | computed: the record is an organization with no parent and no free-text company name, so the complete name is the bare name |
| Commercial entity | computed: the record is an organization → itself |
| Commercial company name | computed: the commercial entity is an organization → its name |
| Shared party flag | computed: no user accounts → set |
| Field synchronization | runs with the whole creation payload; the record has no parent and no children, so every branch is skipped |
| Tax number check | if a number was typed, the normalise-and-validate routine runs against the commercial entity's country, which is this record's country |
| Telephone rewrite | if the form was used and a telephone was typed, the value is already in international form because the form's change handler rewrote it |

**Failure conditions.**

- an unnamed record → `Contacts require a name`
- an invalid tax registration number → the message of
  [business-rules.md](business-rules.md) §6.4
- a barcode already in use → `Another partner already has this barcode`

**Warnings, not failures.** A twin with the same tax registration number or the same company
registration number produces the warning block described in
[business-rules.md](business-rules.md) §7.3.

---

# 2. Create an independent person

**Role**: directory maintainer.

Identical to §1 except that the operator switches the organization/person selector to the person
value before typing the name. The switch sets the organization flag to cleared, which:

- changes the placeholder to a person-oriented example;
- reveals the job-position input;
- hides the industry and the company registration number, which are only shown for organizations;
- makes the address block's label read from the computed address-type description rather than the
  fixed word `Address`.

**Edge case.** The person may be given a **free-text company name** instead of a parent. Doing so
makes the complete name `<the free-text name>, <the person's name>` and reveals a `Create` button
next to the free-text name — see §4.

---

# 3. Create a bare address with no parent

**Role**: directory maintainer.
**Preconditions**: none.

1. Create a Party, clear the organization flag, and set the address type to `invoice`, `delivery`
   or `other`.
2. Leave the name empty if desired — the database check constraint permits it for these three
   types.
3. Fill the address.
4. Save.

The record is its own commercial entity (it has no parent). Its complete name is empty unless it
has a free-text company name, in which case the address type's label is substituted for the
missing name and prefixed: for example `Acme Holdings, Delivery`.

---

# 4. Turn a free-text employer into a real organization

**Role**: directory maintainer.
**Preconditions**: the Party is a person, has no parent, and has a non-empty free-text company
name.

1. Open the Party's form. Next to the free-text company name a `Create` button is shown.
2. Press it. The "create the parent organization" operation runs.

**What it does, in order.**

1. A new Party is created with:
   - the name taken from the free-text company name;
   - the organization flag **set**;
   - the tax registration number copied from this Party;
   - a copy of all six address fields of this Party.
2. This Party is written with:
   - the new organization as its parent;
   - every one of its existing children re-parented onto the new organization.
3. Because the parent is now set, the write also clears the free-text company name.

**What the synchronization then does.** The write carries a parent, so:

- the commercial fields are inherited from the new commercial entity — which already carries the
  same tax registration number, so nothing changes;
- the parent's address is inherited — the parent's address is a copy of this Party's, so nothing
  changes;
- the upward address push finds no difference and is skipped;
- the downstream push runs on the children, which were just re-parented away and are therefore no
  longer children of this Party.

**Result.** A three-record structure where there was one: the new organization, the person under
it, and the person's former children now siblings of the person.

**Failure condition.** An empty free-text company name makes the operation a no-op; the button is
not shown in that case.

---

# 5. Attach a contact and an address to an organization

**Role**: directory maintainer.
**Preconditions**: the organization exists.

1. Open the organization's form and go to the contacts tab.
2. Press the button that adds a contact. The embedded form opens with these defaults, supplied by
   the tab's context:
   - the parent, set to the organization;
   - all six address fields, copied from the organization;
   - the language, copied from the organization;
   - the salesperson, copied from the organization;
   - the address type, set to `contact` when the parent is an organization and to `other`
     otherwise.
3. Choose the address type with the radio selector: `Contact`, `Invoice`, `Delivery` or `Other`.
4. For a `contact`, type the name (required), the electronic mail address, the telephone and the
   job position; the address block is hidden because it is the organization's.
5. For the three address types, the address block is shown and is editable.
6. Save the embedded record, then save the organization.

**What happens for a contact-type child.** See the worked example in
[calculations.md](calculations.md) §3.9 step 2. In summary: the commercial fields are inherited,
the parent's address is inherited, nothing is pushed back up.

**What happens for an address-type child.** The commercial fields are inherited; the address is
**not** touched in either direction.

**Failure conditions.**

- a `contact`-type child with no name → `Contacts require a name`
- a parent that is a descendant of the child → `You cannot create recursive Partner hierarchies.`
- a company on the child incompatible with an attached user → the message of
  [business-rules.md](business-rules.md) §3.1

---

# 6. The organization moves

**Role**: directory maintainer.
**Preconditions**: the organization has at least one contact-type child.

1. Open the organization's form.
2. Change the address fields.
3. Save.

**What happens.** Exactly the sequence traced in [calculations.md](calculations.md) §3.9 under
"The company moves". The changed address fields are pushed to every `contact`-type child through
the direct address update; addresses of type `invoice`, `delivery` and `other` are untouched.

**The reverse direction.** Editing a contact-type child's address pushes it up to the
organization, which then pushes it down to every other contact-type child. This is why the form
marks those inputs read-only.

**Side effect outside this domain.** When the geolocation behaviour is installed, any write that
touches the street, the postal code, the city, the state or the country and that does not itself
set both coordinates forces both coordinates to zero. The child records receive their address
through the direct storage update, which bypasses the ordinary write path — so a rebuild must
decide whether the reset applies to them too, and must document its choice. The observed behaviour
is that the reset applies to the record whose ordinary write carried the address change, and not
to the relatives updated through the direct path.

---

# 7. Record a tax registration number

**Role**: directory maintainer.

1. Open any Party of the organization — the organization itself or any of its contacts.
2. Type the number into the tax registration input. The input's label is the acting company's
   country's own word for it, when that country defines one.
3. Leave the field. The form's change handler runs the normalise-and-validate routine in
   **non-raising** mode, so a partially typed number does not throw; an invalid number is silently
   cleared in that mode only if the mode is the clearing one, which it is not — in non-raising mode
   the routine returns the value normalised but unchecked.
4. Save. The routine now runs in raising mode as the field's inverse.

**What happens on save.**

1. The number is normalised for the country being checked (see
   [calculations.md](calculations.md) §8.5).
2. It is validated (see §8.4 of the same file).
3. If it differs from what was typed, the normalised form is written back.
4. The field synchronization pushes the number **up** to the parent and, from there, **down** to
   every non-organization descendant.
5. Where the verification behaviour is installed and at least one Company has verification switched
   on, the intra-community validity flag is recomputed: for a Party whose parent carries the same
   number the flag is copied from the parent; otherwise the external service is called and a
   history entry is written.

**Failure conditions**: the two messages of [business-rules.md](business-rules.md) §6.4, plus the
single-character message of §6.3.

**Worked case.** Typing `be 0477.472.701` on a Belgian organization stores `BE0477472701`; see
[calculations.md](calculations.md) §8.7.

---

# 8. Archive and unarchive a Party

**Role**: directory maintainer.

**Archive.**

1. Select one or more parties and choose the archive action, or clear the active flag on the form.
2. The archive guard runs: attached **active** user accounts block the operation, with a message
   that either offers a link to those accounts or asks for an administrator, depending on the
   operator's access to user accounts. See [state-machines.md](state-machines.md) §1.3.
3. On success, the active flag is cleared. The Party disappears from every default search, from
   every reference-field dropdown and from every embedded children list (which filters on the
   active flag).
4. Nothing else changes: every document keeps its reference, every balance still aggregates it.

**Unarchive.**

1. Find the Party with the `Archived` filter in the search panel.
2. Choose the unarchive action.
3. The active flag is set. No guard applies.

**Delete.** Only when no reference exists. The same user guard applies with different wording; see
[business-rules.md](business-rules.md) §9.2.

---

# 9. Register a bank account for a Party

**Role**: directory maintainer.
**Preconditions**: the Party is an organization or has no parent.

1. Open the bank-accounts list, or the accounts section of the Party's form.
2. Create an account. The account holder defaults to the Party being edited.
3. Type the account number in any spelling. The sanitised form is computed and stored alongside.
4. Optionally choose the institution. Choosing one that does not yet exist creates it; typing into
   the mirrored name or identifier-code inputs edits the institution itself.
5. Optionally set the currency, the clearing number, the sequence and the notes.
6. Leave the "send money" permission **cleared** unless outgoing payments to this account are
   intended.
7. Save.

**What happens on save.**

| Step | Effect |
|---|---|
| Sanitisation | a payload naming the sanitised number has it moved into the account number; the sanitised number is then recomputed |
| Uniqueness | the pair (sanitised number, holder) is checked; a duplicate fails with `The combination Account Number/Partner must be unique.` |
| Holder name | defaults to the holder's name |
| Company | mirrored from the holder |
| Account type | derived from the number; the foundation package always answers `bank` |
| Colour | ten when outgoing payments are allowed, one otherwise |

**Automatic creation from an incoming document.** When another domain finds an account number in a
received document it calls the find-or-create procedure of [calculations.md](calculations.md) §7.3
rather than creating an account directly. That procedure searches the holder's whole commercial
subtree, refuses to create an account for one of the business's own companies unless explicitly
allowed, and always creates with outgoing payments **disallowed**.

**Archive an account.** Use the archive action. Deleting an account archives it instead; see
[business-rules.md](business-rules.md) §8.4.

---

# 10. Merge duplicate parties

## 10.1 From a selection

**Role**: directory maintainer (administrator for parties with different electronic mail
addresses).

1. In the contacts list or card view, select two or three parties.
2. Choose the `Merge` action from the action menu. The action is bound to the list and card views
   of the Party entity and opens the merge wizard as a modal, with the archived-record filter
   switched off.
3. The wizard opens directly in the `selection` state with the selected parties listed and the
   destination defaulted to the **oldest active** one. The list shows, per Party: the internal
   identifier, the display name, the electronic mail address, the organization flag, the tax
   registration number and the country.
4. Optionally change the destination. The choice is restricted to the parties in the list, and the
   dropdown shows each one with its internal identifier appended so that two identically named
   records can be told apart.
5. Optionally remove a Party from the list to exclude it from the merge.
6. Press `Merge Contacts`.

**What happens**: the six safety checks, then the six passes of
[calculations.md](calculations.md) §11.3 to §11.9. On success the wizard moves to `finished` — or
to the next group, when the wizard was driven from a duplicate search.

**Worked example**: [calculations.md](calculations.md) §11.10 traces two parties carrying two
invoices, two sales orders, two bank accounts and a tag set through the whole procedure.

## 10.2 From a duplicate search

**Role**: directory maintainer.

1. Open the deduplication action from the contacts configuration menu. The wizard opens in the
   `option` state.
2. Tick the grouping criteria: any combination of the electronic mail address, the name, the
   organization flag, the tax registration number and the parent. At least one is required.
   Several criteria are combined with "and": two parties are candidates only when **all** the
   ticked fields match.
3. Optionally tick the exclusion filters: exclude parties that have a user account; exclude
   parties that have accounting entries (offered only where the accounting behaviour is installed).
4. Optionally set the maximum number of groups.
5. Choose one of three buttons.

| Button | Behaviour |
|---|---|
| `Merge with Manual Check` | runs the grouping query, creates one group per candidate set, and steps the operator through them one at a time |
| `Merge Automatically` | confirmed with `Are you sure to execute the automatic merge of your contacts?`; runs the query and then merges every group without asking, committing after each |
| `Merge Automatically all process` | confirmed with `Are you sure to execute the list of automatic merges of your contacts?`; runs the parent-migration pass, then a fresh automatic merge on the three criteria "tax registration number, electronic mail address and name", then clears the organization flag on every Party that has a parent |

6. In the manual mode, for each group the operator may press `Merge Contacts` or
   `Skip these contacts`. Either way the group is removed and the next one is loaded.
7. When no group remains, the wizard shows `There are no more contacts to merge for this request`
   and offers a button that reopens the deduplication action from scratch.

**Failure condition**: no criterion ticked →
`You have to specify a filter for your selection.`

---

# 11. Import parties from a tabular file

**Role**: directory maintainer.

1. Open the contacts list and choose the import action.
2. Optionally download the shipped template, offered under the label
   `Import Template for Contacts`.
3. Map the columns to field storage names. Relational columns may be matched by display name.
4. Run the import.

**What the domain does differently during an import.**

| Behaviour | Reason |
|---|---|
| The state/country consistency of every row is repaired before insertion (see [business-rules.md](business-rules.md) §4.3) | the import resolves each column independently, so a state may be matched from a different country |
| The intra-community verification is removed from the computation queue on both create and write | so that importing ten thousand rows does not make ten thousand external calls |
| Geocoding is suppressed | the geocoding operation returns immediately when the import flag is in the context |
| The country-package installation is suppressed | creating a Company during an import does not trigger a package installation |

**What the domain does the same.** Every validation still runs, the field synchronization still
runs, and the tax-number check still runs in raising mode — so a file containing an invalid tax
registration number fails on that row.

**The bulk-load path.** A data load — as opposed to a user import — takes the batched path of
[calculations.md](calculations.md) §3.8, which suppresses the per-record synchronization, writes
one value map per (commercial entity, parent) group, and then runs the downstream push and the
first-contact guard.

---

# 12. Block and unblock a telephone number

## 12.1 Block a number

**Role**: administrator (the blocked list is restricted to the administrator group).

1. Open the blocked-numbers list and create a record, **or** call the add operation from another
   flow — for example, a recipient asking to be removed from a text-message campaign.
2. Supply the number in any spelling, and optionally a reason.

**What happens.**

1. The number is normalised into the strict international form using the acting user's own
   country resolution. A number that cannot be normalised aborts with
   `<the formatting error> Please correct the number and try again.`
2. Duplicates within the batch are collapsed.
3. Existing records with the same number, active or archived, are found.
4. Archived ones are reactivated — unless the caller explicitly asked for them to stay inactive.
5. Only genuinely new numbers are inserted.
6. A reason supplied for an existing record becomes the log message of its tracking entry; a reason
   supplied for a new record is posted on it as an internal note.
7. The records are returned in the order the caller listed the numbers.

## 12.2 Unblock a number

**Role**: administrator, or any user with write access on the blocked list.

1. From the blocked-number record, or from a record carrying a blocked number, press the unblock
   button. The access check runs first: without write access the operation fails with
   `You do not have the access right to unblacklist phone numbers. Please contact your administrator.`
2. The unblock dialogue opens as a medium modal, showing the number read-only and an editable
   reason.
3. Press apply.

**What happens.**

1. The matching records are found, including archived ones.
2. Numbers that have **never** been blocked are created with the active flag **cleared**, so that
   the unblocking itself is recorded.
3. A supplied reason becomes the log message of the tracking entry for existing records, and is
   posted as the internal note `Unblock Reason: <the reason>` on newly created ones.
4. Existing records are archived.

## 12.3 The effect of blocking

Every automated messaging domain compares its recipient's **sanitised** number against the blocked
list and suppresses delivery on a match. The comparison is on the strict international form, so a
number blocked as `+32485112233` also suppresses a recipient stored as `0485 11 22 33` in Belgium,
because both sanitise to the same value.

## 12.4 Blocking on portal-account deletion

When a portal visitor deletes their own account and asks for their number to be blocked, every
number field of that user is formatted and collected before the deletion, then blocked afterwards,
and each blocked record receives the log entry:

```
Blocked by deletion of portal account <the deleted user's name> by <the acting user's name> (#<the acting user's identifier>)
```

---

# 13. Enrich a Party from an external company directory

**Role**: directory maintainer (the enrichment behaviour must be installed and the service account
must hold credit).

## 13.1 Suggest parties while typing

1. The operator types into the name input, the tax registration input or the global-business-
   identifier input of a Party form. Those three inputs carry the enrichment widget.
2. The widget calls one of two lookups:

| Lookup | Input | Behaviour |
|---|---|---|
| by name | the typed text and a country identifier | when the country identifier is the boolean false the acting company's country is used; when it is zero the country filter is deliberately omitted. The country's code is sent with the query. |
| by tax registration number | the typed number and a country identifier | an empty country identifier falls back to the acting company's country |

3. Each suggestion returned is normalised (see §13.3) and offered in a dropdown.
4. Choosing a suggestion fills the form's fields.

**The fall-back when the service fails a tax-number lookup.** The system calls the customs-union
verification service directly and, if that service returns a valid record whose name is not the
placeholder `---`, builds one suggestion from it:

1. Split the returned address on line feeds, dropping empty lines.
2. The first line becomes the street.
3. The first remaining line that begins with a digit becomes the postal-code-and-city line; it is
   split on the first space into the postal code and the city.
4. The first remaining line that is not that one becomes the second street line.
5. The country code returned by the service is used.

## 13.2 Enrich an existing record

Three lookups exist, each taking one identifier: the global business identifier, the national
goods-and-services-tax number, and the internet domain. Each returns one record, which is
normalised and returned together with an error indication.

**Error indications.**

| Condition | Returned error text |
|---|---|
| the service reports insufficient credit | `Insufficient Credit` |
| the service reports any other error | `Unable to enrich company (no credit was consumed).` |
| the transport failed | the transport's own message |
| no account token is configured | `No account token` |

## 13.3 Normalising a suggestion

Applied to every suggestion before it reaches the form.

1. **Location codes.** Remove the country code and country name from the payload and resolve a
   Country: first by a case-insensitive code match, then by a case-insensitive name match. If a
   Country was found, resolve a State inside it: first by a case-insensitive code match, then by a
   case-insensitive name match. Put the resolved records back into the payload as pairs of
   identifier and display name.
2. **Controlled city.** When the extended-address behaviour is installed, the resolved country
   enforces cities, and the payload carries a postal code or a city name: search the controlled
   city list within that country (and within the state when one was resolved), first by postal
   code and then by a case-insensitive name match. On a hit, put the city and **its** state into
   the payload and remove the free-text city.
3. **Industry.** Remove the industry code from the payload and look up the shipped industry record
   whose single-letter section matches it; on a hit, put it into the payload.
4. **Language.** Remove the preferred-language code from the payload and look for an installed
   language whose locale code and language code both equal it; failing that, for one whose locale
   code and language code both begin with its first two characters. On a hit, put the locale code
   into the payload.
5. **Tax number.** When the tax-number behaviour is installed, the payload carries a tax
   registration number, and the reading context carries the enriched company data: run the
   normalise-and-validate routine in **clearing** mode against the resolved country, so an invalid
   number is silently dropped rather than blocking the suggestion.

## 13.4 Enrich a new Company automatically

**Role**: administrator; runs by itself.

1. When a Company is created, and the acting user is a system user, and the registry is ready, and
   the demonstration-data flag is absent, the automatic enrichment runs once per Company and then
   marks itself done.
2. It derives the Company's internet domain: first from the electronic mail address, rejecting the
   well-known free mail providers; failing that from the website, rejecting the loopback name and
   the reserved example name. With no domain, it stops.
3. It calls the domain lookup with a five-second budget.
4. From the returned payload it keeps only the entries that (a) name a field that exists on the
   Party, (b) have a non-empty value, and (c) either name the stored image or correspond to a field
   that is currently **empty** on the Company's Party. So enrichment never overwrites a value a
   human typed, except for the image.
5. The state and country entries are reduced from a pair of identifier and display name to the
   identifier alone.
6. The result is written onto the Company's Party.

The interface is told, through the session information, whether the acting administrator's own
company still needs enriching.

## 13.5 Post the enrichment result on the record

A helper posts an internal note rendered from a shipped template, carrying: the telephone, the
name, the electronic mail address, the entity type returned by the service, the tax registration
number **or**, failing that, the company registration number, the website, the stored image, the
street, the second street line, the postal code, the city, the country name, the state code, and
the classification codes returned by the service.

---

# 14. Geocode a Party's address

**Role**: directory maintainer (the geolocation behaviour must be installed).

## 14.1 Configure the provider

**Role**: administrator.

1. Open the general settings. The geolocation section shows a provider selector and, when the
   second provider is chosen, a key input.
2. Choose `Open Street Map` or `Google Place Map`. The choice is stored in the system parameter
   `base_geolocalize.geo_provider`.
3. For the second provider, paste the service key. It is stored in the system parameter
   `base_geolocalize.google_map_api_key`.

## 14.2 Resolve the coordinates

1. Open a Party's form and go to the assignment tab. It shows the latitude, the longitude, the date
   of the last resolution, and one of two buttons: `Compute based on address` when both coordinates
   are zero, or `Refresh` when either is non-zero.
2. Press the button.

**What happens.**

1. The operation returns immediately — doing nothing — when the reading context does not carry the
   force flag **and** any of the following holds: an import is running, a test is running, the
   registry is not ready, or demonstration data is being installed.
2. The Party is read in the base language, deliberately, because the providers expect country names
   in that language.
3. A query string is built from the street, the postal code, the city, the state name and the
   country name (see [calculations.md](calculations.md) §14.4).
4. The provider is called. On a hit, the first result's latitude and longitude are taken.
5. On a miss, a **second** attempt is made with a query string built from the city, the state and
   the country only — dropping the street.
6. On a hit, the Party is written with the two coordinates and with today's date in the acting
   user's time zone.
7. On a miss, the Party is collected into a failure list.
8. When the failure list is non-empty, a danger-level notification is pushed to the acting user:
   title `Warning`, message
   `No match found for <the display names of the failed parties, separated by a comma and a space> address(es).`

**Provider failures.**

| Condition | Message |
|---|---|
| the configured provider has no implementation | `Provider <the technical name> is not implemented for geolocation service.` |
| the transport fails | `Error with geolocation server: <the underlying error>` |
| the second provider has no key | a message stating that a key for the geocoding service is required, followed by a line feed and an invitation to visit the provider's key documentation address for more information |
| the second provider returns an error status | a message reading `Unable to geolocate, received the error:` followed by the provider's own message, then a blank line, then three sentences telling the operator that the provider has made the feature chargeable, that billing must be enabled on the provider account, and that the geocoding, static-map and browser-map services must be switched on in the provider's developer console |
| the reverse lookup is attempted while tests run | `OpenStreetMap calls disabled in testing environment.` |

A provider returning "no results" is not a failure: the operation returns nothing and the two-stage
retry applies.

## 14.3 Coordinates are invalidated when the address moves

Any write that touches the street, the postal code, the city, the state or the country and that
does not itself set **both** coordinates forces both to zero. The date of the last resolution is
**not** cleared, which means a Party can show a resolution date with zero coordinates; a rebuild
reproduces this.

---

# 15. Publish a Party as a public page

**Role**: a user with the restricted-editor right on the public site (the public-page behaviour
must be installed).

1. Open the Party's form. A publish control appears in the button box.
2. Optionally fill the short description and the full description. The full description is a rich
   text whose inline styles are stripped and whose sanitisation may be overridden by a privileged
   user; both are translated.
3. Toggle the publish control.

**What happens.**

- The published flag is written. Because it is tracked, an entry appears in the Party's message
  history under one of two subtypes: `Partner published` or `Partner unpublished`, both described
  as `Partner Published` and `Partner Unpublished` respectively, and neither subscribed by
  default.
- The Party's public address is computed as `/partners/` followed by a readable slug built from
  the Party's identifier and name.

**Serving the page.**

| Step | Behaviour |
|---|---|
| Route | `/partners/<the slug>`, served over the hypertext transfer protocol, public authentication, rendered inside the site layout |
| Identifier extraction | the slug is decomposed back into an internal identifier |
| Access | the Party is read with elevated rights; the page is served when the Party exists **and** either it is published **or** the visitor holds the restricted-editor right |
| Canonical address | when the slug the visitor used differs from the canonical slug, the visitor is redirected to the canonical one |
| Not found | any other case returns the not-found response |

**What the page shows.** The Party's stored image (falling back to the largest avatar), a contact
block rendering the address, the website, the telephone and the electronic mail address with
machine-readable markup, the display name as the page heading, the full description, and two
editable regions above and below into which site blocks may be dropped so that they appear on
**every** partner page.

---

# 16. Maintain the geography reference data

**Role**: administrator for countries and currencies; directory maintainer for states and country
groups.

## 16.1 Countries

The countries list is **not** creatable or deletable through the interface: the list view and the
form view both disable creation and deletion. An administrator may edit an existing country's
layout, input view, currency, calling code, tax label, name position, state requirement and postal
code requirement, and may manage its states from an embedded editable list on the form.

Editing the layout runs the substitutability validation. Editing the layout, the input view or the
tax label clears the relevant caches so that every open form picks up the change.

## 16.2 States

A directory maintainer may create, edit and delete states, from the dedicated menu or from the
embedded list on a country's form. The pair (country, code) must be unique.

## 16.3 Country groups

A directory maintainer may create, edit and delete groups and change their membership. The code
must be unique and is upper-cased on write. Changing the membership of the prefixing group changes
the behaviour of the tax-number algorithm for every Party in the affected countries — a rebuild
should treat this as a privileged operation even though the shipped access rights do not.

## 16.4 Cities

Where the extended-address behaviour is installed, a directory maintainer may create, edit and
delete controlled cities; every internal user may read them. A country's form gains a button that
opens its cities, with the country defaulted and pre-filtered.

## 16.5 Currencies and languages

Both are administrator-only. Activating a language loads the translations of every installed
package and may shorten its address-bar code at the expense of another language's. Activating a
currency grants the multi-currency group to every internal user once more than one currency is
active.

---

# 17. Create a Company

**Role**: accounts manager.

1. Open the companies list and create a record.
2. Type the name. Optionally choose a parent — **this is the only moment at which a parent may be
   chosen**.
3. Choose the currency. When a parent was chosen, the currency and every other root-delegated field
   are copied from the parent and shown read-only.
4. Fill the address, the electronic mail address, the telephone, the website, the tax registration
   number and the company registration number. All of them are stored on the Company's Party.
5. Save.

**What happens on save.**

1. A Party is created first, carrying the name, the organization flag **set**, the logo as its
   image, the electronic mail address, the telephone, the website, the tax registration number and
   the country. The parent default is explicitly suppressed so that a default parent from the
   context cannot attach the company's Party to something.
2. Those parties are flushed so that their stored computed fields exist before the Company rows are
   written.
3. Root-delegated fields missing from a branch are filled from its parent.
4. The whole cache is cleared.
5. The Company rows are written.
6. The creating user and the built-in super-user are added to the accepted-users list of every new
   Company.
7. Every inactive currency used by a new Company is activated.
8. Every new Company with a country triggers the installation of its country-specific packages —
   unless tests are running, the registry is initialising, an installation is in progress, or an
   import is running.
9. Where the enrichment behaviour is installed, the automatic enrichment of §13.4 runs.

**Failure conditions.**

- a duplicate name → `The company name must be unique!`
- a branch whose currency differs from its root's →
  `The <the field's label> of a subsidiary must be the same as it's root company.`

**Afterwards.** The Company's address fields are computed from its Party by resolving the Party's
`contact` address; writing any of them writes straight back onto the Company's own Party. The
hierarchy can never be changed; archiving the Company archives every branch and is refused while
active users name it as their own company.

---

# 18. Change the acting company

**Role**: any internal user belonging to more than one company.

1. Use the company switcher. It lists the user's allowed companies ordered by sequence and then by
   name, each shown with the colour index computed in [calculations.md](calculations.md) §14.2.
2. The choice changes: which parties are visible under the multi-company record rule; which bank
   accounts are visible; which company-dependent values are read and written; which country drives
   the address-block reordering and the tax-registration label; and which country drives the
   telephone-number country fall-back.

**Accessible branches.** Where an operation needs "this company and the branches of it that I may
use", the accessible-branch walk of [entities.md](entities.md) §12.6 is used. It is cached per
(allowed companies, company, user), so a rebuild must invalidate that cache when a user's allowed
companies change — which is why writing a Company's active flag or sequence clears the whole cache.

---

# 19. Find a Party from an inbound message

**Role**: automatic; performed by the messaging domain.

1. The sender's address line arrives as a combined name-and-address string.
2. The find-or-create procedure of [calculations.md](calculations.md) §4.2 runs:
   - the string is split into a name and a normalised address;
   - a Party whose electronic mail address matches the normalised address case-insensitively is
     searched for and returned if found;
   - otherwise a Party is created with the parsed name, or with the address as its name when no
     name was parsed.
3. The caller may demand a valid address, in which case a string with no address fails with
   `A valid email is required for find_or_create to work properly.`
4. An empty string always fails with `An email is required for find_or_create to work`.

**Worked examples**: [calculations.md](calculations.md) §4.1.

---

# 20. Resolve the addresses of a counterparty for a document

**Role**: automatic; performed by every document-producing domain.

1. The document names a Party.
2. The document asks for the address types it needs — typically `invoice` and `delivery`.
3. The address-resolution algorithm of [calculations.md](calculations.md) §5 runs and returns one
   Party identifier per requested type, always including a `contact` entry.
4. The document stores the resolved identifiers in its own columns, so that a later change to the
   hierarchy does not silently move a posted document's addresses.

**Worked examples**: [calculations.md](calculations.md) §5.2, including the case where a
subsidiary's boundary stops the walk and the group's invoicing address is *not* used.

---

# 21. Render a Party on a printed document

**Role**: automatic.

1. The document reads the Party's **commercial company name** for the legal name line, or the
   Party's display name where the individual is meant.
2. It reads the rendered address from the address-rendering algorithm of
   [calculations.md](calculations.md) §6, with or without the company line depending on whether the
   legal name is already printed separately.
3. It places the counterparty's name before or after the address block according to the country's
   name-position setting.
4. It reads the tax registration number, relabelled by the country's own word for it.
5. It renders every amount with the reader's language formatting
   ([calculations.md](calculations.md) §10) and the currency's symbol and symbol position.
6. It renders the document in the Party's language.
