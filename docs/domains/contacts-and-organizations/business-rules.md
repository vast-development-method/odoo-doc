# Business rules of the Contacts and Organizations domain

Every validation, every constraint, every invariant, every error message, every permission check,
every locking rule and every edge-case behaviour of the domain. Messages are reproduced exactly as
the system produces them; placeholders are described in words.

---

# 1. Database-level constraints

These are enforced by the storage engine itself and cannot be bypassed by any code path, including
elevated rights.

| Entity | Constraint | Condition | Message |
|---|---|---|---|
| Party | `res_partner_check_name` | the address type is `contact` **and** the name is empty → rejected. Rows whose address type is `invoice`, `delivery` or `other` are accepted with an empty name. | `Contacts require a name` |
| Bank Account | `res_partner_bank_unique_number` | the pair (sanitised account number, account holder) must be unique | `The combination Account Number/Partner must be unique.` |
| Country | `res_country_name_uniq` | the name must be unique across all countries | `The name of the country must be unique!` |
| Country | `res_country_code_uniq` | the two-letter code must be unique across all countries | `The code of the country must be unique!` |
| Country State | `res_country_state_name_code_uniq` | the pair (country, code) must be unique | `The code of the state must be unique by country!` |
| Country Group | `res_country_group_check_code_uniq` | the code must be unique | `The country group code must be unique!` |
| Currency | `res_currency_unique_name` | the currency code must be unique | `The currency code must be unique!` |
| Currency | `res_currency_rounding_gt_zero` | the rounding factor must be strictly greater than zero | `The rounding factor must be greater than 0!` |
| Language | `res_lang_name_uniq` | the name must be unique | `The name of the language must be unique!` |
| Language | `res_lang_code_uniq` | the locale code must be unique | `The code of the language must be unique!` |
| Language | `res_lang_url_code_uniq` | the address-bar code must be unique | `The URL code of the language must be unique!` |
| Company | `res_company_name_uniq` | the company name must be unique | `The company name must be unique!` |
| Blocked Number | `phone_blacklist_unique_number` | the telephone number must be unique | `Number already exists` |

Note the asymmetry: the country name and the country code are unique **globally**, while a state
code is unique only **within its country**. Two countries may therefore both have a state coded
`CA` (California in the United States, and any state elsewhere with the same code).

---

# 2. Hierarchy rules

## 2.1 No cycle in the Party hierarchy

A validation runs on every write of the parent link. It walks the hierarchy and refuses any
configuration in which a Party is, directly or transitively, its own ancestor.

- Message: `You cannot create recursive Partner hierarchies.`
- Applies to creation and to update.
- Also applies when the parent is set indirectly, for example by the merge procedure — which is
  why the merge catches this failure and skips the parent assignment rather than aborting the
  whole merge.

## 2.2 No cycle in the Party Tag tree

- Message: `You can not create recursive tags.`
- Enforced by a validation on the parent link of the tag.

## 2.3 The Company hierarchy is immutable

Writing a parent on an existing Company is refused outright, whether or not it would create a
cycle.

- Message: `The company hierarchy cannot be changed.`
- The parent may only be supplied at creation.

## 2.4 A Company may not be duplicated

- Message: `Duplicating a company is not allowed. Please create a new company instead.`
- The duplicate operation raises immediately; nothing is created.

## 2.5 A branch must agree with its root

Every field in the root-delegated set — in the foundation package, exactly the currency — must
carry the same value on a branch as on its root.

- Message: `The <the field's label> of a subsidiary must be the same as it's root company.` The
  label is the field's own displayed name, for example `Currency`.
- Enforced by a validation on the delegated fields and on the parent.
- Kept true automatically in three places: at creation, a branch's missing delegated fields are
  copied from the parent; when a root's delegated field changes, it is copied to every branch; and
  in the form, choosing a parent copies the delegated fields immediately.
- In the form, a delegated field is displayed read-only whenever the parent is set.

---

# 3. Company-scoping rules

## 3.1 The Party's company must agree with its users

Writing a company on a Party runs this check per record:

1. If the new company is non-empty **and** the Party has at least one attached user account:
   - collect the distinct companies of those accounts;
   - if there is more than one distinct company, **or** the new company is not among them, refuse.
2. Message: `The selected company is not compatible with the companies of the related user(s)`

Setting the company to empty is always allowed, because an empty company is compatible with every
user.

## 3.2 The company cascades to children

After the check, the same company is written on every child of the Party, which recursively
cascades. A subtree therefore always shares one company value.

## 3.3 A Party that represents a Company

A validation runs on every write of the company field:

1. Select the records being written that are organizations and have a company.
2. Find the Company records whose Party is one of them.
3. For each such Company, if the Company is not equal to its Party's company value, refuse.
4. Message:
   `The company assigned to this partner does not match the company this partner represents.`

In plain terms: the Party that *is* company X must have X as its company. It may not be assigned to
company Y, and it may not be left company-less once the link exists.

## 3.4 The company-scoping domain

When another entity declares that its Party must be compatible with its own company, the test is
**"parent of"**: the Party is acceptable when it has no company, or when its company is the
entity's company or an ancestor of it. The same test applies to the Bank Account.

## 3.5 A Company with active users may not be archived

- Message:
  `The company <the company name> cannot be archived because it is still used as the default company of <the number of active users> users.`
- The count is the number of active user accounts whose own company is this one.
- Archiving a Company that passes the check also archives every branch beneath it.

---

# 4. Address rules

## 4.1 The state must belong to the country

Three mechanisms keep the state and the country consistent.

1. **The interface filter.** The state input is restricted to the states of the selected country,
   but only when a country is selected; with no country, every state is offered.
2. **On changing the country.** If a country is now set and it differs from the selected state's
   country, the state is cleared.
3. **On changing the state.** If the state has a country and it differs from the selected country,
   the country is set to the state's country.

There is **no** stored constraint. A state belonging to another country can still be written
directly through a programmatic path or an import; the import repair below is the only guard.

## 4.2 The same interlock on the Bank and the Company

The Bank and the Company both carry the same pair of interlocks, with the same direction:
clearing on a country change, back-filling on a state change. On the Bank the fields are named
`country` and `state` rather than `country_id` and `state_id`.

## 4.3 The import consistency repair

When parties are created through an import, before anything is written:

1. Collect the distinct state identifiers named in the rows.
2. Read those states' countries in one query.
3. For each row that names a state: compare the state's country with the country resolved for that
   row. If they differ, search for a state with the **same code** in the row's country; put that
   state's identifier in the row, or empty the state when no such state exists.

The repair is silent: no warning is produced, and a state that cannot be matched is simply
dropped.

## 4.4 The layout must be substitutable

A validation runs on every write of a country's address layout. It substitutes the number one for
every valid key and requires the substitution to succeed.

- Valid keys: the Party's formatting address fields, plus `state_code`, `state_name`,
  `country_code`, `country_name`, `company_name`.
- Message: `The layout contains an invalid format key`

## 4.5 Country-level address requirements

Two flags on the Country express requirements that the **interface** enforces and that this domain
does **not** enforce at the storage level:

| Flag | Default | Meaning |
|---|---|---|
| `state_required` | cleared | An address in this country should name a state. |
| `zip_required` | **set** | An address in this country should carry a postal code. |

A rebuild must reproduce the flags and their shipped values (see
[configuration.md](configuration.md) §8.1) and must apply them wherever it validates an address
entered by an external party — a storefront checkout, an electronic document import — because that
is where other domains consume them.

## 4.6 Controlled cities

When the extended-address behaviour is installed and a country has the enforcement flag set:

- the free-text city input is hidden whenever a controlled city has been chosen, or whenever the
  free-text city is empty;
- the controlled-city input is shown only for countries with the flag set;
- choosing a controlled city overwrites the free-text city, the postal code and the state;
- clearing the controlled city on a saved record clears all three;
- changing the country clears the controlled city when the city does not belong to the new country.

No storage-level constraint enforces the requirement. A rebuild that wants a hard guarantee must
add one, and must accept that existing free-text addresses in that country would then fail.

## 4.7 The address fields of a contact-type child are read-only in the form

Whenever a Party's address type is `contact` **and** it has a parent, every address input is
displayed read-only. This is an interface rule, not a storage rule: the synchronization algorithm
would happily accept a write and push it up to the parent (and it does, when a write arrives
through another path). The read-only marking exists so that an operator does not silently change
the whole organization's address while editing one person.

---

# 5. Name and identity rules

## 5.1 A contact must be named

See §1 — the database check constraint. The four consequences worth stating:

1. An operator can create a nameless invoicing, delivery or other address.
2. An operator cannot create a nameless contact.
3. Changing a nameless address's type to `contact` fails at the storage level.
4. The complete-name computation substitutes the address-type label for a missing name only for
   the three address types, never for `contact` — precisely because `contact` can never be
   nameless.

## 5.2 Renaming a Party renames its bank accounts

On every write that changes the name, and before the name is written:

- for each Party being renamed, for each of its bank accounts, if the account's holder name equals
  the Party's **current** name, the holder name is set to the new name.

Accounts whose holder name was deliberately set to something else are left alone.

## 5.3 The website link is normalised

On create and on write, a non-empty website value is normalised:

1. Parse it as a web address.
2. If it has no scheme:
   - if it also has no network location, move the path into the network location and empty the
     path;
   - add the scheme `http`.
3. Store the result.

So `example.com` becomes `http://example.com`, `www.example.com/about` becomes
`http://www.example.com/about`, and `https://example.com` is left alone.

## 5.4 Setting a parent clears the free-text company name

On create and on write, a payload containing a non-empty parent also sets the free-text company
name to empty. The two are alternatives: either the organization exists as a record, or it exists
only as text.

## 5.5 The barcode is unique

- Message: `Another partner already has this barcode`
- Enforced by a validation that counts the parties carrying the same barcode and refuses when the
  count exceeds one.
- The barcode is company-dependent, so the count is taken in the acting company's scope: two
  parties may carry the same barcode in two different companies.
- The barcode is never copied when a Party is duplicated.

## 5.6 An invalid address type supplied as a default is discarded

When a default value for the address type arrives from a menu's context and is not one of the four
valid tokens, it is replaced by an empty value rather than kept or rejected. The reason recorded is
that a stale default from another application's menu could otherwise leak onto a newly created
Party.

---

# 6. Tax registration number rules

## 6.1 When the check runs

- As the inverse of the tax registration number field and of the country field, so on every write
  of either, in raising mode.
- From the form's change handler on the same two fields, in non-raising mode, so that a partially
  typed number does not throw.
- Never during an import: the verification flag's computation is explicitly removed from the queue
  when the import flag is in the context.

## 6.2 Against which country

Against the **commercial entity's** country, not the record's own. A delivery address in another
country is therefore validated against the legal entity's country, which is correct because the
tax number belongs to the legal entity.

## 6.3 The "no tax number" marker

The single character `/` means "this counterparty deliberately has no tax number" and is always
accepted. Any *other* single character is refused in raising mode with:

```
To explicitly indicate no (valid) VAT, use '/' instead. 
```

(note the trailing space, which the system produces). In clearing mode the single character is
replaced by an empty value; with validation switched off it is returned unchanged.

## 6.4 The failure messages

With a named Party:

```

The <tax label> number [<the normalised value>] for partner [<the Party's name>] does not seem to be valid.  
Note: the expected format is <the example for the country>
```

Without a name — the case where the record label contains the text `False`:

```

The <tax label> number [<the normalised value>] does not seem to be valid.  
Note: the expected format is <the example for the country>
```

Both begin with a line feed. The note is omitted entirely when the country has no example. The tax
label is the acting company's country's own word for the number when the country being checked is
that same country and such a word is defined; otherwise the literal `VAT`.

When the union-wide retry was armed and also failed, the message is followed by a blank line and:

```
If you are trying to input a European number, this is the expected format: <the example>
```

## 6.5 The suppression flag

A reading context carrying `no_vat_validation` returns the normalised number without checking it.
The documented purpose is data pushed in from an external platform over which the business has no
control.

## 6.6 The doubled prefix

A value that begins with its own country prefix twice — `BEBE0477472701` — is treated as invalid
even if the remainder would pass, because the normalisation would otherwise silently accept a
double prefix and every downstream document would carry it.

## 6.7 Validation is per country

The full catalogue of per-country algorithms, and the fall-back to a standard numeric-identifier
library, is in [calculations.md](calculations.md) §8.4. A country for which neither a
system-defined check nor a library module exists accepts **every** value unconditionally. A rebuild
must reproduce that permissiveness: refusing unknown countries' numbers would break every
counterparty outside the covered set.

## 6.8 One check reads the Party, not the number

The Uzbek check requires nine digits for an organization and fourteen for a person. It is the only
per-country check that depends on a field of the Party. A rebuild that implements the checks as
pure functions of the number must make an exception for this one.

---

# 7. Duplicate detection

Duplicates are **detected and shown**, never refused.

## 7.1 The tax-registration twin

Computed on every Party. The full search is in [calculations.md](calculations.md) §15.1. The rules
that matter:

- The search runs with elevated rights and **includes archived parties**, deliberately, so that an
  operator is told to reactivate an archived duplicate rather than create a third copy.
- The search excludes the record itself and every descendant of it.
- A Party **with a parent never reports a twin**: its number is inherited and a match is expected.
- A number of exactly one character is never checked.
- In the prefixing group, the prefixed and unprefixed spellings are both searched, plus the two
  special prefixes.
- Candidates are limited to those whose country is the same or empty, and whose company is the same
  or empty.

## 7.2 The company-registration twin

- The search excludes the record itself and every descendant.
- A Party with a parent never reports a twin.
- Candidates are limited to those whose company is the same or empty. Unlike the tax twin, the
  country is **not** part of the filter, even though the field's own description says the number
  must be unique per country.

## 7.3 How they are shown

A warning block at the top of the Party form, visible only while editing, with four mutually
exclusive wordings:

| Condition | Wording |
|---|---|
| only a tax twin | `Potential duplicates:` then the twin's name and `(Same <the tax label>)` |
| only a registration twin | `Potential duplicates:` then the twin's name and `(Same <the registration label>)` |
| both twins and they are the same Party | `Potential duplicates:` then the twin's name and `(Same <the tax label> and <the registration label>)` |
| both twins and they are different parties | `Potential duplicates:` then the tax twin's name and `(Same <the tax label>) and`, then the registration twin's name and `(Same <the registration label>)` |

Both twins are displayed with the address and the tax number suppressed in their display names, so
the warning stays on one line.

---

# 8. Bank account rules

## 8.1 The holder must be an organization or a root

The account holder input is restricted to parties that are organizations **or** have no parent. An
address may not hold a bank account. This is an interface filter; nothing stops a programmatic
write.

## 8.2 Outgoing payments are not allowed by default

The "send money" permission is cleared on every newly created account, is never copied when an
account is duplicated, and is forced to cleared by the find-or-create procedure. Every path that
creates an account from an external document therefore produces an unpayable account until a human
allows it.

## 8.3 The sanitised number cannot be written directly

A payload containing the sanitised number has that value moved into the account number instead;
then the sanitised number is recomputed. The two can never disagree.

## 8.4 Deleting archives

See [state-machines.md](state-machines.md) §1.4.

## 8.5 The own-company guard on find-or-create

Message:

```
Please add your own bank account manually: <the account number> (<the Party's display name>)
```

Raised when the procedure would have to create an account for the Party of one of the business's
own companies and the caller did not explicitly allow it.

## 8.6 The company-scoping record rule

A Bank Account is visible when its company is one of the reader's companies or an ancestor of one,
or when it has no company. The company is a stored mirror of the holder's company, so moving a
Party between companies moves its accounts with it.

---

# 9. Archival and deletion rules

## 9.1 A Party with an active user cannot be archived

See [state-machines.md](state-machines.md) §1.3 for the two messages and the redirecting action.

## 9.2 A Party with a user cannot be deleted

The same guard, with different wording, runs on deletion. It is registered so that it does **not**
run when a software package is being uninstalled.

- With write access on user accounts, a redirecting warning:

  ```
  You cannot delete contacts linked to an active user.
  You should rather archive them after archiving their associated user.

  Linked active users : <the display names, separated by a comma and a space>
  ```

  with an action opening those accounts and a button labelled `Go to users`.

- Without write access, a plain validation failure:

  ```
  You cannot delete contacts linked to an active user.
  Ask an administrator to archive their associated user first.

  Linked active users :
  <the display names, separated by a comma and a space>
  ```

Note that both messages say "active user" while the search that produced them looks for **any**
user account, active or not. The wording is the system's; a rebuild reproduces it.

## 9.3 Countries and states cannot be deleted while referenced

The Party's country and state links are restricting links: the storage engine refuses to delete a
referenced Country or Country State.

## 9.4 Language deletion

Three guards, checked in this order for each language:

1. the base language `en_US` can never be deleted —
   `Base Language 'en_US' can not be deleted.`
2. the reader's own preferred language cannot be deleted —
   `You cannot delete the language which is the user's preferred language.`
3. an active language cannot be deleted —
   `You cannot delete the language which is Active!\nPlease de-activate the language first.`

The guard is registered so that it *does* run during uninstallation.

## 9.5 Language deactivation

Three guards, checked in this order:

1. no **active user** may have the language —
   `Cannot deactivate a language that is currently used by users.`
2. no **active Party** may have the language —
   `Cannot deactivate a language that is currently used by contacts.`
3. no user account at all, active or archived, may have the language —
   `You cannot archive the language in which the application was setup as it is used by automated processes.`
   (The message as produced names the application by its product name.)

Then the user-defined default that sets the language of new parties is discarded for that language.

## 9.6 At least one language must exist

A validation on the active flag refuses the operation when the language table would be left empty.
The check is skipped while the registry is still initialising.

- Message: `At least one language must be active.`

At start-up, if no language exists at all, an error is written to the log rather than raised.

## 9.7 Currency deactivation

A currency used by any Company cannot be deactivated.

- Message: `This currency is set on a company and therefore cannot be deactivated.`
- Two context flags suppress the check: the installation flag, because during installation a
  company's currency is not yet visible as active; and an explicit force flag used by tests.

---

# 10. Merge rules

Run in this order before anything is modified. The first five are the safety checks.

| Order | Rule | Message |
|---|---|---|
| 1 | An administrator bypasses the same-address check. | *(no message; a flag is cleared)* |
| 2 | Fewer than two surviving parties → the merge returns silently. | *(none)* |
| 3 | More than three parties → refuse. | `For safety reasons, you cannot merge more than 3 contacts together. You can re-open the wizard several times if needed.` |
| 4 | One of the parties is a descendant of another → refuse. | `You cannot merge a contact with one of his parent.` |
| 5 | More than one user account is attached across all the parties, counting archived accounts → refuse. | `You cannot merge contacts linked to more than one user even if only one is active.` |
| 6 | More than one distinct electronic mail address across the parties, unless the acting user is an administrator → refuse. | `All contacts must have the same email. Only the Administrator can merge contacts with different emails.` |

Further rules that govern the merge itself:

- The destination is the **oldest active** Party unless one is named explicitly; see
  [calculations.md](calculations.md) §11.2.
- Every user account attached to any of the parties has the destination's company linked and set,
  before any reference is rewritten.
- Bank accounts are merged before the Party references, so that no account is orphaned.
- A row that would violate a uniqueness constraint after the rewrite is **deleted**, not left
  behind — except in two-column join tables, where the colliding row is simply not moved and
  disappears with its source.
- The destination's own non-empty field values always win; a source's value is used only where the
  destination's is empty.
- Computed fields and list-valued fields are not merged by value; they are recomputed or handled by
  the reference passes.
- Fields whose copy flag is cleared — the barcode — are not merged by value; the company-dependent
  pass handles the barcode instead.
- A parent taken from a source is applied only if it is not the destination itself, and a cycle
  failure is swallowed with a log line rather than aborting the merge.
- The grouping search refuses to run with no criterion:
  `You have to specify a filter for your selection.`

---

# 11. Telephone rules

## 11.1 Normalisation failures

Every failure message from the parse operation, with the exact wording:

| Condition | Message |
|---|---|
| the parser cannot read the input at all | `Unable to parse <the number>: <the parser's own reason>` |
| the country prefix is not a real one | `Impossible number <the number>: not a valid country prefix.` |
| too few digits | `Impossible number <the number>: not enough digits.` |
| too many digits, after both repairs failed | `Impossible number <the number>: too many digits.` |
| any other impossibility | `The phone number <the number> is invalid! Let's fix it - you are not dialing aliens.` |
| possible but not a valid number under the plan | `Invalid number <the number>: probably incorrect prefix.` |

Every message names the **original** input, never the repaired one.

## 11.2 The form rewrites the number

On the Party, changing the telephone, the country or the company rewrites the telephone into
international presentation form, silently keeping the original when the rewrite fails. It is a
form-level behaviour: a write that does not go through a form does not rewrite.

## 11.3 The blocked list

- Adding a number that cannot be normalised fails with
  `<the underlying formatting error> Please correct the number and try again.`
- Writing a number on an existing blocked record normalises it first, with the same failure.
- Opening the unblock dialogue without write access on the blocked list fails with
  `You do not have the access right to unblacklist phone numbers. Please contact your administrator.`

## 11.4 Searching by number

- An entity that declares no text-typed number field fails with
  `Invalid primary phone field on model <the entity's transport name>`.
- A search with no searchable field fails with `Missing definition of phone fields.`
- A search term shorter than three characters fails with
  `Please enter at least 3 characters when searching a Phone number.`
- A term beginning with a plus sign or with two zeros matches **both** spellings.

## 11.5 The known limitation of the blocked-field flag

An entity with both a mobile field and a landline field stores only **one** sanitised number — the
first that normalises. The flag that says "the blocked number is the landline" is computed by a
loop that overwrites its own result at each step, so it ends up describing only the **last** number
field examined. A rebuild should reproduce the single sanitised number (because the blocked-list
comparison depends on it) and may correct the flag; correcting it changes no stored data.

---

# 12. Permission checks

## 12.1 Groups

| Group | Purpose |
|---|---|
| Internal user | May read parties, tags, industries, banks, bank accounts, countries, states, country groups, currencies and languages. May not create or modify any of them. |
| Contact creation | May create, read, update and delete parties, tags, bank accounts and banks; may create, read, update and delete states and country groups; may read countries. Implied by the administrator group. |
| Administrator | Implies contact creation and access-rights management. May create, read, update and delete countries, currencies, languages, industries and blocked numbers. |
| Access-rights management | May create, read, update and delete companies. |
| Portal | May read parties inside their own commercial entity's subtree, and read countries, states, country groups, currencies, languages and companies in their allowed set. |
| Public | The same read-only surface as the portal group. |
| Multi-company | Reveals the company field on the Party form and the company switcher. |
| Multi-currency | Granted automatically to every internal user when more than one currency is active; withdrawn when at most one is. |

The complete matrix is in [configuration.md](configuration.md) §3.

## 12.2 Writing the organization flag

When the organization flag is in a write payload, the acting user is not already operating with
elevated rights, **and** the user belongs to the contact-creation group, the flag is written in a
separate operation with elevated rights and removed from the payload. The purpose is to let a user
who may edit parties flip the flag even where a record rule would otherwise interfere.

## 12.3 Writing a Party that belongs to an internal user

After every write, for each record, the attached user accounts that are internal users other than
the writer are collected and the writer's **write access on the user entity** is verified. A user
who may edit parties but not user accounts therefore cannot edit another internal user's Party.

## 12.4 Elevated rights inside the synchronization

Two steps of the field-synchronization algorithm run with elevated rights: inheriting the
commercial fields from the commercial entity, and pushing the commercial fields down to the
descendants. The reason is that the commercial entity or a descendant may be invisible to the
acting user under the multi-company rule, while the values must still travel.

## 12.5 Elevated rights inside the merge

The merge uses elevated rights for: aligning the users' companies, deleting and moving bank
accounts, the whole foreign-key rewriting pass (which bypasses the object layer entirely), the
polymorphic-reference pass, the per-company writes, and the final deletion of the sources.

## 12.6 Sudo is not allowed to issue write commands on the Party or the Language

Both entities declare that command-style writes issued under elevated rights are forbidden. In
practice this prevents a caller from using elevated rights to push a nested create or delete
through a relation field.

---

# 13. Record rules

| Entity | Rule | Applies to | Filter |
|---|---|---|---|
| Party | multi-company | everybody (global) | the Party is a shared party, **or** its company is one of the reader's companies or an ancestor of one, **or** it has no company |
| Party | portal and public subtree | the portal group and the public group | the Party is the reader's commercial entity or a descendant of it; **read only** — creation, update and deletion are excluded from the rule, so they are simply not granted |
| Bank Account | multi-company | everybody (global) | the account's company is one of the reader's companies or an ancestor of one, **or** it has no company |
| Company | portal | the portal group | the company is in the reader's allowed set |
| Company | internal user | the internal-user group | the company is in the reader's allowed set |
| Company | public | the public group | the company is in the reader's allowed set |
| Company | access-rights management | that group | unrestricted |

The reason the Party's multi-company rule starts with "is a shared party" is spelled out in the
source: parties that have internal users are excluded from the company restriction so that the
restriction does not collide with the user's own company rule and make some users unselectable in
reference fields.

---

# 14. Locking and concurrency

This domain has **no** explicit locking. There is no lock date, no approval lock, no restricted
edit mode. Three concurrency-sensitive behaviours nonetheless exist and a rebuild must reproduce
them.

## 14.1 The synchronization loop guard

Before the field-synchronization algorithm runs, the write path captures each record's **previous**
values for exactly the fields being written, and then runs the synchronization with only the fields
whose value actually changed. Without this, a computation that writes back the same value would
re-enter the synchronization for ever. The source records a concrete case: a property field whose
write updates a computed field whose inverse writes the same value back onto the property field.

## 14.2 The direct address update

Pushing an address to a relative writes at the storage level, bypassing the ordinary write path and
therefore bypassing the synchronization entirely. This is what stops an address push from bouncing
back and forth between a parent and a child.

## 14.3 The merge commits per group

The automatic merge commits the transaction after each group. A failure in the tenth group
therefore leaves the first nine merged. This is deliberate — a long automatic merge must not lose
all its work — and a rebuild must reproduce it or must document the difference, because the
observable behaviour after a crash differs.

---

# 15. Edge cases a rebuild must reproduce

1. **A contact-type child with an empty name.** Unreachable through the interface but producible
   through a direct write that bypasses the check; the complete name then ends with a comma. The
   check constraint makes this unreachable in a correctly built system.
2. **A parent whose address is entirely empty.** Nothing is pushed down: the address-values helper
   returns an empty map when every address field is empty. A child keeps whatever it had.
3. **A parent whose address is partly empty.** *Everything* is pushed down, including the empty
   fields: the helper returns all six fields once any one of them is non-empty. So filling only the
   city on a parent clears the street of every contact-type child.
4. **The bulk-load path differs.** When many parties are created at once by a data load, only the
   parent's **non-empty** address fields are pushed. A rebuild that unifies the two paths changes
   observable behaviour for data loads.
5. **An organization child blocks the commercial push.** A subsidiary and its whole subtree are
   skipped by the descendant push, but the recursion still descends *into* the subsidiary's
   children — the skip applies to the write, not to the walk. In practice the subsidiary's children
   are also skipped because they are re-filtered at each level.
6. **The first-contact guard only fires on the bulk-load path.** Creating one contact through the
   ordinary path does *not* copy its address up to an address-less parent; the upstream push in the
   synchronization does that instead, and only when the child's address differs from the parent's
   after inheritance — which, for a childless parent with an empty address, it does.
7. **The company-registration number travels down but not up.** Unlike the tax number.
8. **Company-dependent commercial fields are propagated to every company.** The foundation package
   has none, so the step is a no-op; a rebuild that adds one must implement the loop over all
   companies.
9. **A Party may be its own commercial entity while not being an organization** — when it has no
   parent. Its commercial company name is then its free-text company name, which may be empty.
10. **Archived children still appear** in the children list when the caller disables the
    archived-record filter; the relation is declared with the filter disabled in its own reading
    context but with an active-only filter on the relation itself, so the two interact.
11. **The formatted electronic mail address of a nameless Party** contains the literal text
    `False` as the display name. Deliberate.
12. **The time-zone offset changes with the season.** It is computed at read time from the current
    moment.
13. **The generated avatar changes colour once**, when an unsaved Party is saved and acquires a
    creation timestamp.
14. **The street recomposition normalises.** Writing any of the three street parts rewrites the
    whole street into the canonical "name number - door" shape, even if the operator had typed
    "number name".
15. **A word without a digit cannot be a door number** and is silently lost when the street is
    recomposed.
16. **The country's two-character search shortcut** means that a country whose *name* is two
    characters long would be found by code first. No shipped country has such a name.
17. **A state search of the form `Name (Country)`** is decomposed and matched on both parts. This
    is how a display name pasted back into a search box still finds its record.
18. **The bank display-name search matches the identifier code by prefix** and the name by
    containment — a search for `BNPA` finds the institution whose code starts with it, and a search
    for `Nationale` finds every institution with that word in its name.
19. **Deleting a bank account silently archives it.** Callers that check for a deleted row will not
    find one.
20. **A currency becomes active automatically** when it is chosen as a Company's currency.
21. **Activating a language may rename another language's address-bar code.** The short-code
    reassignment is a side effect on a *different* record.
22. **The blocked-number remove operation creates a record** when the number was never blocked, so
    that the unblocking itself is on the record.
23. **The merge's group query groups by the raw value** for every field except the electronic mail
    address, the name (both lower-cased) and the tax registration number (spaces removed). Grouping
    by the organization flag therefore groups the three storage values — set, cleared and empty —
    separately.
24. **A group whose parties the operator cannot all see is narrowed, not skipped**, unless fewer
    than two remain.
