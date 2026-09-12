# Multi-company

One installation serves several legal entities at once. This document specifies what a company is, how a tree of companies is formed, how a session decides which companies it is working in, how records are scoped to companies and filtered by the record-rule layer, how the platform prevents a document of one company from pointing at master data of another, how a value can differ per company on a shared record, how currency and branches behave, what changes at the instant a person switches company, and how the cross-company flows work.

Company scoping is not a gate of its own. It is an application of the record-rule layer specified in [the security model](security-model.md), plus one validation — the company consistency check — that is not a permission at all. [The security model, section 12](security-model.md#12-company-scoping-and-company-consistency) states the part the gates need; everything else is here.

**Naming used throughout this document.** The single-company link of an entity is its **company field**, whose storage name is `company_id`; the multi-company link of the few entities that have one is the **company list field**, whose storage name is `company_ids`. On the User entity the single link is the **default company** and the list is the **allowed companies**; they use the same two storage names. The ordered list a session is currently working in is the **activated companies**, carried in the environment context under the key `allowed_company_ids`, and its first element is the **current company**. Inside a record-rule filter, `company_id` denotes the identifier of the current company and `company_ids` denotes the list of identifiers of the activated companies.

---

## Table of contents

1. [The company tree](#1-the-company-tree)
2. [Allowed companies, activated companies, current company](#2-allowed-companies-activated-companies-current-company)
3. [How the activation travels](#3-how-the-activation-travels)
4. [Company fields on entities](#4-company-fields-on-entities)
5. [Company scoping through record rules](#5-company-scoping-through-record-rules)
6. [The company consistency check](#6-the-company-consistency-check)
7. [Values that differ per company](#7-values-that-differ-per-company)
8. [Currency](#8-currency)
9. [Working with branches](#9-working-with-branches)
10. [Elevated rights and companies](#10-elevated-rights-and-companies)
11. [Cross-company flows](#11-cross-company-flows)
12. [Companies and identities](#12-companies-and-identities)
13. [What changes at the instant of a switch](#13-what-changes-at-the-instant-of-a-switch)
14. [Caching keyed by company](#14-caching-keyed-by-company)
15. [Messages](#15-messages)
16. [Acceptance criteria](#16-acceptance-criteria)
17. [Invariants a rebuild must preserve](#17-invariants-a-rebuild-must-preserve)
18. [Reconciliation notes](#18-reconciliation-notes)

---

## 1. The company tree

### 1.1 What a company is

A Company is a legal or organisational entity that owns documents. It carries its own identity data through a Contact record — name, address, tax identification number, registry number, electronic mail address, telephone number, web site address, logo — its own document presentation — paper format, document template, font, primary and secondary colours, tagline, footer — its own currency, and the set of users allowed to work in it.

| Property | Rule |
|---|---|
| Uniqueness | The name is unique across all companies; the violation is refused with **"The company name must be unique!"** |
| Ordering | By the presentation sequence, whose default is 10, then by name; this ordering drives the company switcher |
| Identity record | Every company has exactly one Contact. Creating a company from a name creates that Contact, marked as an organisation, carrying the electronic mail address, telephone number, web site address, tax identification number and country given for the company |
| Reverse consistency | A Contact that represents an organisation and carries a company link must be the identity record of that same company, otherwise **"The company assigned to this partner does not match the company this partner represents."** |
| Duplication | Forbidden: **"Duplicating a company is not allowed. Please create a new company instead."** |
| Hierarchy | A company may have a parent company; the set of companies therefore forms a forest |
| Hierarchy changes | Forbidden after creation: writing the parent link is refused with **"The company hierarchy cannot be changed."** |
| Archiving | Allowed only when no **active** user has it as their default company, otherwise **"The company "** the company name **" cannot be archived because it is still used as the default company of "** the number of such users **" users."**; archiving a company also archives its branches |

The Company entity's own field catalogue and the shipped company records are specified in [identity and access](../domains/identity-and-access/entities.md).

### 1.2 Root companies and branches

A company with no parent is a **root company**. A company with a parent is a **branch**. Branches exist so that one legal entity can be subdivided — per establishment, per activity, per country registration — while sharing the parts of the configuration that must legally be identical.

The derived values are all computed from the materialised ancestor path of the tree:

| Value | Definition |
|---|---|
| `parent_path`, the materialised ancestor path | The identifiers of the company's ancestors in order, ending with the company's own, each followed by a separator |
| Ancestors | The companies of the path, from the root down to and including the company itself; for a root company this is the company alone |
| Root | The first of the ancestors |
| Branches | The direct branches of the company that are active |
| All branches | The direct branches, archived ones included |

**Worked example.** Companies 1 (root), 2 (branch of 1), 3 (branch of 2) and 4 (unrelated root).

| Company | Materialised ancestor path | Ancestors | Root |
|---|---|---|---|
| 1 | `1/` | 1 | 1 |
| 2 | `1/2/` | 1, 2 | 1 |
| 3 | `1/2/3/` | 1, 2, 3 | 1 |
| 4 | `4/` | 4 | 4 |

### 1.3 Values delegated from the root to the branches

Some values must be identical on every company of one tree. They are the **root-delegated values**. The foundation delegates exactly one, the **currency**; capability packages may extend the list.

| Moment | Rule |
|---|---|
| Creating a branch | Every root-delegated value absent from the creation values is copied from the parent |
| Editing a branch in a form | Changing the parent link copies every root-delegated value from the new parent into the form |
| Writing a root-delegated value on a root company | The same value is written on **every** descendant of that company, found with elevated rights, in one write per root |
| Validation | On any write of a root-delegated value or of the parent link, a branch whose value differs from its parent's is refused with **"The "** the value's label **" of a subsidiary must be the same as it's root company."** |
| Presentation | In a form view, every root-delegated field is served with its read-only marker bound to the condition "this company has a parent" |

**Worked example.** Company 1 uses the euro; branches 2 and 3 were created from it and therefore also use the euro. An administrator sets the currency of company 1 to the Swiss franc: the write propagates to 2 and 3 in the same transaction. An attempt to set the currency of company 2 alone to the United States dollar is refused with "The Currency of a subsidiary must be the same as it's root company."

### 1.4 Creating a company

**Preconditions.** One or more sets of creation values, each carrying at least a name or an identity record.

1. For every entry that has no identity record but has a name, create the Contact: mark it as an organisation and carry over the name, logo, electronic mail address, telephone number, web site address, tax identification number and country of the entry; flush the unit of work so that the Contact's computed address values exist; link it as the company's identity record.
2. For every entry naming a parent, copy the root-delegated values from the parent where they are not given.
3. Clear the registry caches.
4. Insert the companies.
5. Add every created company to the allowed companies of **both** the acting user and the root identity.
6. Activate the currency of every created company whose currency was archived.
7. For every created company that names a country, install the localisation packages marked automatic for that country, unless the platform is running its self-checks, is importing a file, or is in the middle of an installation.

Step 5 is what makes a newly created company immediately usable: without it the creator could not activate it, because a session may only activate companies the user is allowed ([section 2.2](#22-deriving-the-current-company-and-the-activated-companies)).

Capability packages extend creation with their own per-company master data. The inventory capability, for instance, creates the company's locations, sequences, operation types and rules, and links the other companies' identity contacts to the shared inter-company transit location ([section 11.3](#113-inter-company-goods-movements)).

---

## 2. Allowed companies, activated companies, current company

### 2.1 The three sets

| Notion | Where it lives | Definition |
|---|---|---|
| Allowed companies of a user | The User record's company list field, `company_ids` | The **active** companies linked to that user, through the association table `res_company_users_rel`; cached per user and recomputed when companies or the link change |
| Default company of a user | The User record's company field, `company_id` | The company a session starts in, and the company written on records the user creates when nothing else applies |
| Activated companies of an environment | The context key `allowed_company_ids` | The ordered list of companies the session is currently working in |
| Current company of an environment | The first activated company | The company used for defaults, for per-company values, for numbering and as the target of "create here" |

The default company must be one of the allowed companies of an **active** user. Violating this is refused with **"Company "** the company name **" is not in the allowed companies for user "** the user name **" ("** the allowed company names **")."** The constraint is re-evaluated when the default company, the allowed companies or the active flag change, which is why unarchiving a user whose company was archived in the meantime fails with the same message.

### 2.2 Deriving the current company and the activated companies

**The current company.**

1. Take the context key `allowed_company_ids`, or the empty list when it is absent.
2. If the list is not empty: in a **restricted** environment, if any identifier in it is not among the acting user's allowed companies, refuse with **"Access to unauthorized or invalid companies."**; in an unrestricted environment skip that check. Return the company of the **first** identifier of the list.
3. Otherwise return the acting user's default company.

**The activated companies.**

1. Take the context key `allowed_company_ids`, or the empty list when it is absent.
2. If the list is not empty: perform the same authorisation check as above, then return the companies of the list, in that order.
3. Otherwise return **all** the allowed companies of the acting user.

Two deliberate asymmetries, both of which a rebuild must reproduce:

1. **Without the context key, the current company is the user's default company but the activated companies are *all* the user's companies.** A request that arrives without an explicit activation — a printed document, an image served by address, a link followed from a notification, a scheduled job — therefore sees everything the user is entitled to see, rather than one company's slice. This is safe, because the user is entitled to all of them, and it avoids spurious refusals on flows that have no screen to carry an activation. Reading the empty list as the main company alone would silently hide records the user may see.
2. **With elevated rights, the authorisation of step 2 is skipped.** An elevated flow may activate any company, including one the acting user is not allowed. This is what makes cross-company automation possible ([section 10](#10-elevated-rights-and-companies)).

### 2.3 Changing the activated companies from inside a flow

Switching a record set to a company:

1. If the company is empty, return the record set unchanged.
2. Take the current context list, or the empty list.
3. If the list is not empty and its first element is already that company, return the record set unchanged.
4. Build a new list: the company, then the previous list with that company removed.
5. Return the record set in an environment whose `allowed_company_ids` is that new list.

| Starting activation | Switch to | Resulting activation |
|---|---|---|
| Absent | 3 | 3 |
| 1, 2 | 1 | 1, 2 (unchanged) |
| 1, 2 | 2 | 2, 1 |
| 1, 2 | 3 | 3, 1, 2 |

The first row is the one to be careful with: switching company from an environment that had **no** activation narrows the activated set to that single company; it does not add to the user's companies.

The switch validates nothing by itself. The authorisation of [section 2.2](#22-deriving-the-current-company-and-the-activated-companies) happens later, when the current company or the activated companies are read in a restricted environment. A flow that switches to a company the acting user is not allowed and then reads company-scoped data without elevation is refused at that point.

### 2.4 What reads the current company

| Consumer | Use |
|---|---|
| Default values | A company field declared with the standard default takes the current company |
| Per-company values | The entry read and written is the current company's ([section 7.1](#71-per-company-field-values)) |
| Record rules | The filter receives the current company and the activated companies ([section 5](#5-company-scoping-through-record-rules)) |
| Company consistency | Per-company relation fields are validated against the **current** company, ordinary ones against the record's own company ([section 6](#6-the-company-consistency-check)) |
| Currency | The reference currency of amounts presented without an explicit currency is the current company's ([section 8](#8-currency)) |
| Numbering sequences | When several sequences share a code, the current company's wins over the shared one ([section 7.3](#73-sequences)) |
| User default values | A default defined for the current company wins over one defined for no company ([section 7.2](#72-user-default-values)) |
| Cache keys | Every value that depends on the company is cached under the current company's identifier ([section 14](#14-caching-keyed-by-company)) |

### 2.5 The three mechanisms that express company scope

| Mechanism | What it does |
|---|---|
| A company field on the record | Says which company owns it. Empty means shared by every company. |
| A record rule per entity, usually global | Restricts visibility to records whose company is among the activated companies, or is shared |
| The company consistency check | Prevents records of incompatible companies from being linked |

The first two are visibility; the third is integrity. They are independent: a user may perfectly well see two companies' records at once and must still be prevented from linking them.

---

## 3. How the activation travels

### 3.1 The chain

The activated companies travel from the browser to the filter of a record rule along one chain:

1. The activated-companies cookie held by the browser.
2. The client's own state, initialised from that cookie at session bootstrap.
3. The context key `allowed_company_ids` sent with every call the client makes.
4. The environment of the request built from that context.
5. Record-rule filters, default values, per-company values, numbering and currency.

The session bootstrap sends the client, for an internal user: the user's default company; the map of allowed companies with, for each, its identifier, name, presentation sequence, branch identifiers, parent identifier and currency identifier; and the map of **disallowed ancestor companies**, being the ancestors of the allowed companies that the user is *not* allowed. The second map exists only so that the switcher can draw the tree correctly; those companies can never be activated.

### 3.2 Reading and normalising the cookie

1. Read the identifiers in the cookie, in order; the cookie joins them with a separator, and separators are normalised on the way in.
2. Resolve each requested identifier against the user's allowed companies.
3. If nothing resolved, or if any requested identifier did not resolve, replace the whole list by the user's default company alone.
4. Keep the first element in place and sort the remaining elements by identifier.
5. Write the cookie back and publish the list as `allowed_company_ids`.

Step 3 is the fail-safe: a cookie naming a company the user has lost access to resets the activation instead of refusing the request. Step 4 reduces the number of distinct activation lists, because the list is part of the cache key of nearly every call; only the first element carries meaning in the ordering, so the rest is normalised.

### 3.3 The company switcher

The switcher shows the tree of allowed companies, plus the disallowed ancestors needed to draw it, sorted by presentation sequence at the root level and indented by depth. Two gestures exist:

| Gesture | Effect |
|---|---|
| **Toggle** a company | Add it to the selection together with **all its branches recursively**, or remove it together with all its branches recursively |
| **Log into** a company | When the selection is currently a single whole company, replace the selection entirely; otherwise move that company to the front of the selection. Either way, apply immediately |

A company that is not allowed — a disallowed ancestor — can be neither toggled nor logged into; it is drawn for structure only.

"The selection is currently a single whole company" means that every selected company has the same highest selected ancestor and every descendant of that ancestor is also selected: the person is working in exactly one company with all of its branches. In that case "log into" replaces the whole selection. Otherwise the person is deliberately working across several companies, and "log into" only changes which one is current.

Applying a selection:

1. If the selection is empty, use the current company alone.
2. Publish the selection as the activated companies, without adding branches again: the toggle already did that.
3. Write the cookie.
4. If a record is open in the client, ask the platform whether the acting user may still read that record with the new activation; if not, drop that record from the navigation history so that the reload does not land on a refused page.
5. Reload.

Step 4 is the reason a person who switches away from a document's company lands on the list rather than on a refusal.

### 3.4 Activation outside the client

| Caller | Activation |
|---|---|
| A scheduled job | None: the activated companies are all the allowed companies of the job's identity, and the current company is that identity's default company |
| A printed document | The activation of the request that asked for it; a document renderer explicitly switches to the company of the document it renders |
| A shared link | The cookie, widened by the suggested company when the record is otherwise unreachable ([section 11.4](#114-following-a-link-into-another-company)) |
| A machine-to-machine call | Whatever the caller puts in the context; the authorisation of [section 2.2](#22-deriving-the-current-company-and-the-activated-companies) applies |
| An elevated internal flow | Whatever it switched to, unvalidated |

---

## 4. Company fields on entities

### 4.1 The three shapes

| Shape | Meaning | Count in the shipped catalogue |
|---|---|---|
| A single company link, `company_id` | The record belongs to exactly that company; an **empty** value means the record is shared by every company | 222 entities |
| A list of companies, `company_ids` | The record belongs to each of those companies; an empty list means shared | 6 entities: Account, User, Electronic Mail Alias Domain, In-App Purchase Account, Spreadsheet Dashboard, and the account-merge wizard line |
| No company field | The entity is global and its records are identical for everybody: languages, countries, currencies, units of measure, entity definitions, views, translations | The remaining entities |

The "empty means shared" convention is the single most important multi-company rule of the data model. It lets one product, one tax group, one payment term or one contact be used by every company, while a journal entry, a payment or a stock move belongs to exactly one.

### 4.2 The default

A company field declared with the standard default takes the **current company** at creation time. A few entities deliberately declare no default so that their records are shared unless someone sets a company explicitly; Contact is the most important of them, because a contact that belongs to one company cannot be used by the others. A child contact inherits its parent's company at creation.

### 4.3 Company-checked relations

A relational field may be marked **company-checked**. The mark has two effects: one at validation time ([section 6](#6-the-company-consistency-check)) and one in the client.

The **candidate filter** sent to the client for a company-checked relation is built as follows, and is the reason a person never sees another company's journal in a selector:

| Case | Candidate filter |
|---|---|
| The field is a per-company value | The company filter of the target entity for the **first activated company**, conjoined with the field's own declared filter |
| The entity is the Company entity itself | The company filter of the target entity for **this record**, conjoined with the own filter |
| The entity has a single company link | The company filter for that company when it is set, otherwise the company filter for "no company", conjoined with the own filter |
| The entity has a list of companies | The company filter for that list when it is set, otherwise the company filter for "no company", conjoined with the own filter |
| The entity has neither, and the field is not a per-company value | No company filter is added, and a warning is written to the technical log naming the field |

The **company filter of the target entity** is the same function that the consistency check uses ([section 6.2](#62-the-company-filter-of-a-target-entity)), so the client's selector and the server's validation agree by construction. The selector is guidance, not enforcement ([the security model, section 20](security-model.md#20-what-is-not-enforcement)); the validation is the guarantee.

### 4.4 Changing the company of a record

Nothing in the platform forbids writing a different company on a record; the write is governed by the ordinary gates plus the consistency check.

1. The record-rule gate decides whether the user may write the record **as it stands before the write** ([the security model, section 8.6](security-model.md#86-write)). There is no post-write record check on a write, therefore moving a record to a company the user cannot see succeeds and the record then disappears from their view.
2. The consistency check of [section 6](#6-the-company-consistency-check) runs after the write and refuses the move when it would leave the record pointing at master data of the old company.
3. Individual entities add their own guards where a company change would corrupt history. The warehouse entity, for instance, refuses it outright with **"Changing the company of this record is forbidden at this point, you should rather archive it and create a new one."**

---

## 5. Company scoping through record rules

### 5.1 The mechanism

Company scoping is **not** a special layer. It is ordinary record-rule evaluation ([the security model, section 6](security-model.md#6-record-rules)) using the two company values of the evaluation context:

| Name in the filter | Value |
|---|---|
| `company_id` | The identifier of the current company |
| `company_ids` | The list of identifiers of the activated companies |

Both are already validated against the acting user's allowed companies by the time the filter runs ([section 2.2](#22-deriving-the-current-company-and-the-activated-companies)), therefore a rule may trust them and does not re-check them.

### 5.2 The five canonical shapes

Of the 576 record rules shipped with the platform, 160 reference the company. They take five shapes.

| Shape | Condition | Meaning | Used for |
|---|---|---|---|
| A. Own company or shared | The company is among the activated companies, or is empty | Records of an activated company, plus shared records | Operational documents that may also exist without a company |
| B. Own company only | The company is among the activated companies | Records of an activated company only | Documents that always have a company: journal entries, payments, stock moves |
| C. Ancestors, shared allowed | The company is empty, or is an ancestor of, or equal to, an activated company | Records of an activated company, of any ancestor of an activated company, or shared | Master data a branch inherits from its root and that may also be shared by all: products, product documents, price lists, price list rules, vendor price lines, payment terms, bank accounts, currency rates, contacts |
| D. Ancestors only | The company is an ancestor of, or equal to, an activated company — or, on the entities scoped by a list, the company list contains such a company | Records of an activated company or of any of its ancestors | Master data that always carries a company: journals, taxes, accounts |
| E. Identity | The record's own identifier is among the activated companies | The activated companies themselves | The Company entity, one group rule per user kind |

The ancestor test is evaluated by expanding the materialised ancestor paths of the activated companies into the set of all their ancestor identifiers, the activated companies themselves included, and testing membership of that set. It is therefore strictly wider than shape B whenever a branch is activated, and identical to it when only root companies are activated.

**Worked evaluation.** Companies 1 (root), 2 and 3 (branches of 1) and 4 (unrelated root). The user is allowed 1, 2, 3 and 4.

| Activation | Shape A matches records whose company is | Shape B | Shape C | Shape D |
|---|---|---|---|---|
| 1 | 1, empty | 1 | 1, empty | 1 |
| 2 | 2, empty | 2 | 1, 2, empty | 1, 2 |
| 3, 2 | 3, 2, empty | 3, 2 | 1, 2, 3, empty | 1, 2, 3 |
| 1, 2, 3 | 1, 2, 3, empty | 1, 2, 3 | 1, 2, 3, empty | 1, 2, 3 |
| 4 | 4, empty | 4 | 4, empty | 4 |
| 1, 4 | 1, 4, empty | 1, 4 | 1, 4, empty | 1, 4 |

Read the second row carefully: working in branch 2 alone shows the **root's** master data (shapes C and D) but **not** the root's documents (shapes A and B). That is exactly the intent of branches — shared configuration, separate books.

### 5.3 The rules shipped by the foundation

| Entity | Kind | Condition |
|---|---|---|
| Company | One group rule for internal users, one for portal users, one for public users | The record's own identifier is among the activated companies |
| Company | Group rule for access-rights administrators | Always true |
| User | Global | The user is not an external user, or their allowed companies intersect the activated companies |
| Contact | Global | The contact is not shared-only, or its company is an ancestor of an activated company, or its company is empty |
| Bank Account | Global | The company is an ancestor of an activated company, or is empty |
| Currency Rate | Global | The company is an ancestor of an activated company, or is empty |

Two of them are worth explaining.

**The Company rule is a group rule, not a global rule.** Each user kind gets its own rule with the same condition, and administrators of access rights get an "always true" rule. Because group rules combine by disjunction ([the security model, section 6.4](security-model.md#64-the-combination-rule)), an access-rights administrator sees every company while everybody else sees only the companies they have activated. A consequence to reproduce: a user who is allowed three companies but has activated one **cannot read the other two** until they activate them; anything that needs them anyway must read with elevated rights, which is what the session bootstrap and the switcher do.

**The Contact rule exempts the contacts of internal users.** A contact is shared-only exactly when it has no internal user; contacts that do have one are visible whatever the company. Without that exemption a user of company A could not be selected as the salesperson, the responsible person or a follower of a document in company B, and relation selectors would show holes.

### 5.4 Interaction with the composition laws

Company rules are almost always **global** rules (shapes A to D), which means they conjoin with everything else: no group rule can ever widen them. That property is what makes company isolation trustworthy. The exception is the Company entity itself, where the rules are group rules and the access-rights administrator's rule genuinely widens the set.

The composed filter is cached per acting identity, unrestricted flag, entity, operation and **activated companies**; the activated companies are part of the cache key by default, and the list is converted into a fixed sequence for the key ([the security model, section 6.4](security-model.md#64-the-combination-rule)). Switching company therefore changes the cache entry rather than invalidating anything.

### 5.5 The multi-company hint on a refusal

When the record gate refuses and at least one blamed rule mentions the company anywhere in its filter, the refusal explains which company would have helped. The full algorithm and the three message variants are in [the security model, section 6.7](security-model.md#67-the-refusal-message). The suggested company is determined as follows:

1. If the entity has a single company link, the record's own company.
2. Otherwise, if the entity has a list of companies, the first company of the intersection of the record's list with the acting user's allowed companies.
3. Otherwise, none, and no hint is added.

Entities whose visibility is decided by another record's company override this rule so that the hint names the company the **rule** tests rather than the record's own: a time-off request, for instance, suggests the company of its time-off type.

---

## 6. The company consistency check

### 6.1 What it protects

Record rules keep a user from **seeing** another company's records. They do not keep a document of company A from **pointing at** a record of company B: a user allowed in both companies may legitimately see both, and a selector filtered on the client can be bypassed by any call that writes the field directly. The consistency check is the server-side guarantee.

It is a **validation, not a permission**: it runs whatever the acting identity and **also with elevated rights**, because a cross-company link corrupts the books regardless of who created it.

### 6.2 The company filter of a target entity

Every entity answers the question "given these companies, which of your records may they use?". The answer is a filter.

| Entity family | Filter for a set of companies | Filter for the empty set |
|---|---|---|
| Default | The company is in the set, or is empty | The company is empty |
| Shared down a hierarchy: taxes, tax groups, journals, accounts, payment terms, payment methods, payment providers, products, product templates, analytic accounts, analytic plans, analytic distribution models, bank accounts, currencies, contacts | The company is empty, or is an ancestor of, or equal to, a company of the set — expanded into "the company is among the ancestors of the set, or is empty" | The company is empty |
| Scoped by a list of companies: Account | The company list contains one of the ancestors of the set | Always true |
| User | The user's allowed companies intersect the set | Always true |

The second family is the branch rule again: a branch may use its root's master data. The fourth is what lets a user of company A be assigned to a document of company B when that user is allowed both.

An entity may override the rule; the User entity's row above is precisely such an override, checking membership of the allowed companies rather than equality of the default company.

```formula
compatible( linked record , owning companies ) =
    ( company of linked record is empty )
    OR ( company of linked record ∈ owning companies )
```

An empty company on the linked record means "shared by all companies", which is why shared reference data may be linked from any company's documents.

### 6.3 When the check runs

| Moment | Fields checked |
|---|---|
| After a creation, on an entity that enables the automatic check | **Every** field of the entity |
| After a write, on an entity that enables the automatic check | The written fields only, **unless** the write touched the company link or the company list, in which case every field |
| Explicitly, from an operation that changes links without going through a write | The fields the operation names |

87 entities enable the automatic check in the shipped catalogue, and 420 relational fields across the catalogue are marked company-checked.

### 6.4 The algorithm

**Preconditions.** A record set, and optionally the names of the fields to check.

1. If no field names are given, or the names include the company link or the company list, check **every** field of the entity; otherwise check only the named ones.
2. Split the named fields into two lists: the **ordinary** ones — relational, company-checked and not per-company values — and the **per-company** ones — relational, company-checked and per-company values.
3. If both lists are empty, stop: there is nothing to check.
4. Start an empty list of inconsistencies.
5. For each record in the set:
   1. **Ordinary fields, against the record's own companies.** If the ordinary list is not empty, determine the record's companies: the record itself when the entity is the Company entity; otherwise its company field when it has one; otherwise its company list when it has one; otherwise write a technical warning naming the entity and the fields and skip this record. For each ordinary field, read its value **with elevated rights**; if the value is not empty, build the company filter of the target entity for those companies and, if the filter is not empty, record the triple of record, field and targets for every target that does not satisfy it, evaluated with archived records visible.
   2. **Per-company fields, against the current company.** For each per-company field, read its value with elevated rights; if the value is not empty, build the company filter of the target entity for the current company alone and record the triple for every target that does not satisfy it.
6. If the list of inconsistencies is not empty, refuse with the message of [section 6.5](#65-the-message).

Three details change the outcome and must be reproduced:

- The targets are read **with elevated rights**. Without that, a user who may not read the target record would get a permission refusal instead of a consistency refusal, and the message would be wrong.
- The evaluation makes **archived records visible**, so that archiving a target does not turn a consistency violation into a silent pass.
- Step 5.2 compares against the **current company**, not the record's. That is correct by construction: a per-company value is being written *for* the current company, so the record it points at must be usable *by* the current company.

### 6.5 The message

The first line is always:

> "Uh-oh! You’ve got some company inconsistencies here:"

Then at most five lines, one per inconsistency, in one of three shapes:

| Situation | Line |
|---|---|
| The record is itself a company | "- Record is company “\<company name\>” while “\<field label\>” (\<field name\>: \<target display names\>) belongs to another company." |
| The record points at itself through its own company link | "- Only a root company can be set on “\<the record's display name\>”. Currently set to “\<company name\>”" |
| Any other case | "- “\<the record's display name\>” belongs to company “\<company names\>” while “\<field label\>” (\<field name\>: \<target display names\>) belongs to another company." |

The last line is always:

> "To avoid a mess, no company crossover is allowed!"

The company names are the record's company, or the comma-separated list when the entity has a company list. The target display names are the comma-separated list of the offending targets, each quoted. Only the first five inconsistencies are rendered; any remainder is counted but not shown.

### 6.6 Worked examples

**Setup.** Companies 1 (root), 2 (branch of 1) and 4 (unrelated root). Journal J1 belongs to company 1; account A1 is available to companies 1 and 2; tax T4 belongs to company 4; product P has no company.

| Attempt | Outcome |
|---|---|
| Create a journal entry in company 1 on journal J1 with a line on account A1 | Allowed: the journal's company is 1, and the account's company list contains 1 |
| Create a journal entry in company 2 on journal J1 | Allowed: J1's company 1 is an ancestor of 2, and Journal belongs to the hierarchy-sharing family |
| Create a journal entry in company 4 on journal J1 | Refused: "- “INV/2026/0001” belongs to company “Company 4” while “Journal” (journal_id: 'J1') belongs to another company." |
| Add a line with tax T4 to a journal entry of company 1 | Refused, naming the tax field and T4 |
| Add a line with product P to a journal entry of any company | Allowed: a shared record satisfies every company filter |
| Set, while company 4 is current, the per-company receivable account of a contact to A1 | Refused by step 5.2: the current company is 4 and A1 is not available to 4, even though the contact itself is shared |
| Set the same per-company value while company 1 is current | Allowed, and the value is stored for company 1 only |
| Move a journal entry of company 1 to company 4 by writing its company | The write touches the company link, therefore **every** field is re-checked; the journal, the accounts and the taxes of the existing lines now belong to the wrong company, and the move is refused |

### 6.7 What the check does not do

It does not check non-relational values, it does not check relations that are not marked company-checked, and it does not walk transitively: if document D of company 1 points at record R of company 1, and R points at record S of company 4, the check on D says nothing about S. Each entity is responsible for marking its own links. A rebuild that omits a mark loses the guarantee silently, which is why the mark belongs in the entity specification of every domain.

---

## 7. Values that differ per company

Three distinct mechanisms let one installation hold different values per company. They are not interchangeable.

### 7.1 Per-company field values

A field declared per-company is stored once per record but holds one entry per company. Reading returns the entry of the **current company**, falling back to the company's user default value and then to the type's empty value; writing sets the entry of the current company only. The complete storage, reading, writing, searching and invalidation rules are in [the entity and field system, section 11](entity-and-field-system.md#11-company-dependent-values). What matters here:

| Question | Answer |
|---|---|
| Which company decides the value | The current company of the environment, never the record's own company |
| What happens on a company switch | The value read changes with no write and no database access when both entries are already in the cache; the cache holds one entry per current company and record |
| What happens to the other companies' entries when one is written | Nothing |
| How it interacts with the consistency check | A per-company relation marked company-checked is validated against the current company ([section 6.4](#64-the-algorithm), step 5.2) |
| How the client filters its candidates | Against the first activated company ([section 4.3](#43-company-checked-relations), first row) |

Typical uses: the receivable and payable accounts of a contact, the stock locations used for a contact's deliveries and receipts, the price list of a contact, the fiscal position of a contact. All of them are properties of the relationship between a company and a shared record, not of the shared record itself. This is orthogonal to scoping: the record is shared, only the value differs.

### 7.2 User default values

A User Default Value proposes a value for a field of an entity at creation time. It is scoped by **user** and by **company**, both optional:

1. Select every default of that entity whose user is empty or is the acting user, whose company is empty or is the current company, and whose recorded condition matches.
2. Order them by user, then by company, then by identifier.
3. Keep, for each field, the **first** row seen.

Ordering a nullable column ascending places the rows that carry a value **before** the rows that do not, therefore the precedence is:

1. a default for this user and this company;
2. a default for this user, any company;
3. a default for this company, any user;
4. a default for anybody and any company.

The same table is the fallback source of per-company field values ([section 7.1](#71-per-company-field-values)). Creating, writing or deleting any User Default Value clears the whole cache, because any per-company fallback may have changed.

### 7.3 Sequences

Several numbering sequences may share one code. The one used is chosen by company:

1. Check the read permission on the Sequence entity for the acting user.
2. Take the sequences whose code matches and whose company is the **current** company or is empty, ordered by company.
3. If there is none, write a technical note and return nothing.
4. Use the first candidate.

Ordering by a nullable company ascending puts the company-specific sequence before the shared one, so a company that defines its own numbering wins and every other company falls back to the shared sequence. Two companies therefore keep independent counters for the same document kind as soon as each has its own sequence record.

---

## 8. Currency

### 8.1 One currency per company, shared down the tree

Every company has exactly one currency, and it is a root-delegated value ([section 1.3](#13-values-delegated-from-the-root-to-the-branches)): a branch always has its root's currency. Amounts of a document are expressed in the document's own currency and are also stored converted into the **company currency** of the document's company, which is what makes the books of one company internally consistent.

Two guards:

| Rule | Message |
|---|---|
| A currency used by a company cannot be archived | **"This currency is set on a company and therefore cannot be deactivated."** |
| Setting a company's currency activates that currency when it was archived | None; the activation is silent |

When a company is created and its country implies a currency, the form proposes that country's currency.

### 8.2 Rates belong to the root company

A Currency Rate record carries a date, a currency, a rate and a company.

| Rule | Detail |
|---|---|
| Default company | The **root** of the current company |
| Constraint | A rate may not be attached to a branch: **"Currency rates should only be created for main companies"** |
| Uniqueness | One rate per date, currency and company: **"Only one currency rate per day allowed!"** |
| Positivity | **"The currency rate must be strictly positive."** |
| Shared rates | A rate record with no company applies to every company that has no rate of its own for that currency and date |
| Visibility | The shipped global rule is shape C of [section 5.2](#52-the-five-canonical-shapes): a rate is visible when its company is empty or is an ancestor of an activated company |

### 8.3 Resolving a rate

**Preconditions.** A set of currencies, a company and a date.

1. Take the root of the given company.
2. For each currency, find the **latest** rate record of that currency whose date is on or before the given date and whose company is either empty or that root, ordered by company with the records that carry a company first, then by date descending, keeping one.
3. For the same currency, find the **earliest** rate record whose company is either empty or that root, ordered by company with the records that carry a company first, then by date ascending, keeping one.
4. The rate is the latest record's rate; when there is no rate at or before the given date, it is the earliest record's rate; when the currency has no rate at all, it is 1.
5. Return the rate of each currency.

Two properties follow: a company-specific rate always wins over a shared rate of the same date, and a date **before** the first known rate uses that first rate rather than 1, so back-dated documents are converted with the oldest known rate instead of at parity.

### 8.4 Converting

```formula
conversion rate = rate of the target currency ÷ rate of the source currency
```

where both rates come from [section 8.3](#83-resolving-a-rate), evaluated for the **root** of the given company — or of the current company when none is given — at the given date, or today in the environment's time zone when no date is given. When the source and target currencies are the same, the conversion rate is 1 and no rate is read.

```formula
converted amount = source amount × conversion rate
```

The converted amount is then rounded to the number of decimal places of the target currency, by half-up rounding, when rounding is requested; otherwise it is returned unrounded. A source amount of zero converts to zero without reading any rate.

**Worked example.** Company 1 has the euro as its currency, which therefore has the rate 1 by definition. The rate records are, with the currency codes reproduced as stored values (euro `EUR`, United States dollar `USD`, Swiss franc `CHF`):

| Currency | Date | Rate | Company |
|---|---|---|---|
| `USD` | 1 March 2026 | 1.0850 | Company 1 |
| `USD` | 15 March 2026 | 1.1000 | Company 1 |
| `CHF` | 1 March 2026 | 0.9500 | Company 1 |
| `USD` | 10 March 2026 | 1.2000 | Shared, no company |

| Conversion | Date | Rates used | Result |
|---|---|---|---|
| 100.00 euro to United States dollars | 12 March 2026 | Euro 1, `USD` 1.0850 — the company's own record of 1 March; the shared record of 10 March loses because a company record exists | 100.00 × 1.0850 ÷ 1 = 108.50 |
| 100.00 euro to United States dollars | 20 March 2026 | Euro 1, `USD` 1.1000 | 100.00 × 1.1000 ÷ 1 = 110.00 |
| 100.00 United States dollars to euro | 20 March 2026 | `USD` 1.1000, euro 1 | 100.00 × 1 ÷ 1.1000 = 90.909090…, rounded to two decimal places = 90.91 |
| 100.00 United States dollars to Swiss francs | 20 March 2026 | `USD` 1.1000, `CHF` 0.9500 | 100.00 × 0.9500 ÷ 1.1000 = 86.363636…, rounded to two decimal places = 86.36 |
| 100.00 euro to United States dollars | 1 February 2026, before every rate | The earliest `USD` record, 1.0850 | 100.00 × 1.0850 ÷ 1 = 108.50 |

For company 4, whose currency is the United States dollar, the same rate records are read only when they carry no company or belong to company 4; the rate of the United States dollar for company 4 is 1 by definition, and the company-specific records above are not visible to it.

The exchange-rate model itself, the rate providers and the accounting treatment of exchange differences are specified in [multi-currency](../domains/multi-currency/README.md); the conversion of accounting documents belongs to [the general ledger](../domains/general-ledger/README.md).

### 8.5 Presentation

The label of a rate column is rendered against the current company's currency — "Unit per" the currency name, and the currency name "per Unit" — and the view cache key therefore includes the current company's currency name, so two companies with different currencies do not share a cached view. The group `base.group_multi_currency`, Multi Currency, gates the display of currency fields and rate menus; it grants no permission of its own.

---

## 9. Working with branches

### 9.1 Accessible branches

**Preconditions.** A company, and an environment with its activated companies.

1. Let the accessible set be the activated companies.
2. Let the current level be the given company, read with elevated rights.
3. While the current level is not empty: append the intersection of the current level with the accessible set to the result, then set the current level to the direct branches of the current level.
4. If the result is empty and the acting identity is the root identity, return the given company itself.
5. Return the result.

The result is the part of the subtree of that company that the session is currently working in. It is cached per activated companies, company and acting identity. The special case at step 4 exists because a scheduled job has no activation of its own, so the intersection would otherwise be empty and every branch-aware computation would silently produce nothing.

### 9.2 Whole-company selection

A set of companies is a whole-company selection when it equals the set of every company that is a descendant of, or equal to, the roots of that set. It answers the question "is the session working on complete companies, branches included?". Operations that only make sense for a complete legal entity — statutory reports, legal exports, tax returns — use it to refuse or to warn when only part of a tree is activated.

### 9.3 The branch list

A company exposes an action listing its direct branches, with archived branches visible and the parent preset for creation. Because the hierarchy cannot be changed afterwards ([section 1.1](#11-what-a-company-is)), the parent chosen at creation is permanent; correcting a mistake means archiving the branch and creating another.

---

## 10. Elevated rights and companies

| Aspect | Restricted | Elevated |
|---|---|---|
| Activating a company the user is not allowed | Refused with **"Access to unauthorized or invalid companies."** when the current or activated companies are read | Permitted, silently |
| Reading records of another company | Refused by the company record rules | Permitted: rules are skipped entirely |
| Writing records of another company | Refused by the company record rules, evaluated on the records before the write | Permitted |
| The company consistency check | Enforced | **Still enforced**: it is a validation, not a permission |
| Per-company values | Read and written for the current company | Identical, and the current company may be one the user is not allowed |
| Authorship | The acting user | The acting user, unchanged |

A deliberate cross-company operation therefore always has the same shape:

1. Compute the target company.
2. Build an environment switched to that company.
3. Elevate.
4. Create or write the records.
5. Leave the elevation as soon as the write is done.

The three failure modes to watch for are: omitting step 2, which creates the records in the caller's company; omitting step 5, so that the rest of the flow silently sees every company; and assuming that step 3 disables the consistency check, which it does not — the master data of the target company must genuinely be reachable from that company.

---

## 11. Cross-company flows

### 11.1 Shared master data

The cheapest cross-company mechanism is the empty company link: one record usable by all. It is used for the reference data every company agrees on — units of measure, countries, currencies, languages — for the catalogue when the group sells the same goods everywhere — products, product categories, attributes — and for the shared address book, being contacts without a company.

Consequences to reproduce:

1. A shared record satisfies the company filter of every company ([section 6.2](#62-the-company-filter-of-a-target-entity)), so it can be referenced from any company's documents.
2. A shared record is visible under rule shapes A, C and E but **not** under shapes B and D. An entity whose rule is shape B therefore cannot have shared records in practice, and its company field is effectively required.
3. Creating a shared record means writing an empty company, which the client only offers when the person holds `base.group_multi_company`, Multi Company; the field is otherwise hidden and takes the current company by default.
4. Per-company values ([section 7.1](#71-per-company-field-values)) are the standard way to keep a record shared while letting each company parameterise it: one product, one receivable account per company.

### 11.2 Inter-company settlement of a payment

When a document of company A is settled by a payment recorded in company B, neither company's books may simply absorb the other's amount: each needs a balanced entry and an inter-company position. The platform performs the settlement automatically when all of the following hold:

1. the document is an invoice or a credit note, and its payment state is "not paid", "partially paid" or "in payment";
2. it carries at least one payment transaction whose payment belongs to a **different** company;
3. both companies define an inter-company clearing journal;
4. the document's company defines the inter-company receivable account for a sale document, or the inter-company payable account for a purchase document, and the payment's company defines the opposite one;
5. the payment is in process or paid.

**The procedure.**

1. For each payment:
   1. Take the receivable or payable lines of the payment as the counterpart.
   2. The balance is the negated sum of the balances of the counterpart lines; the currency amount is the negated sum of their currency amounts.
   3. Create a journal entry **in the payment's company**, in its inter-company clearing journal, referenced by the payment's memo, with two lines, both labelled with the document's contact, a slash and the document's number. The first line uses the payment's own counterpart account, carries the document's contact, the currency amount and the balance. The second line uses the payment company's inter-company payable account when the document is a sale document, or its inter-company receivable account otherwise, carries the identity contact of the document's company, and the negated currency amount and balance.
   4. Post that entry and reconcile the counterpart lines with its first line.
2. Take the receivable or payable lines of the document; their total is the sum of their balances.
3. Create a journal entry **in the document's company**, in its inter-company clearing journal, referenced "Interco Settlement - " followed by the payment memos separated by commas, with two lines. The first line uses the document company's inter-company receivable account when the document is a sale document, or its inter-company payable account otherwise, carries the identity contact of the payment's company and the total as its balance. The second line uses the document's own receivable or payable account, carries the payment's contact and the negated total as its balance.
4. Post that entry and reconcile the document's lines with its second line.

**Worked example.** Company A issues a customer invoice of 1,000.00 euro to customer C. The money is received through a payment recorded in company B, which has already booked bank 1,000.00 debit against outstanding receipts 1,000.00 credit.

Entry created in company B, in its inter-company clearing journal:

| Account | Contact | Debit | Credit |
|---|---|---|---|
| Outstanding receipts | C | 1,000.00 | |
| Inter-company payable | Identity contact of A | | 1,000.00 |

Entry created in company A, in its inter-company clearing journal:

| Account | Contact | Debit | Credit |
|---|---|---|---|
| Inter-company receivable | Identity contact of B | 1,000.00 | |
| Customer receivable | C | | 1,000.00 |

Afterwards the payment's outstanding-receipts line is reconciled and disappears from company B's outstanding items; company B owes 1,000.00 to company A; company A is owed 1,000.00 by company B; the customer invoice of company A is fully reconciled and its payment state becomes "paid". Every entry is balanced inside its own company, and the two inter-company positions mirror each other and are eliminated on consolidation. For a purchase document the two inter-company accounts swap: the document's company uses its inter-company **payable** account and the payment's company its inter-company **receivable** account. The account selection rules and the reconciliation semantics are specified in [payments and bank reconciliation](../domains/payments-and-bank-reconciliation/README.md) and [the general ledger](../domains/general-ledger/README.md).

### 11.3 Inter-company goods movements

Goods that leave one company and enter another cannot be moved directly, because a stock move belongs to one company and each company must value its own stock. The platform uses a **shared transit location**.

| Element | Rule |
|---|---|
| The internal transit location | One per company, used between two warehouses of the **same** company |
| The inter-company transit location | One for the whole installation, with **no company**, therefore usable by all; it is activated as soon as a second company is created, and it may be archived but never deleted: **"The "** the location name **" location is required by the Inventory app and cannot be deleted, but you can archive it."** |
| Which one a resupply route uses | The internal transit location when the supplying warehouse belongs to the same company, the inter-company transit location otherwise |
| Direction accounting | A move whose source is a transit location **without** a company counts as an incoming move, and a move whose destination is such a location counts as an outgoing move; a transit location **with** a company is neither |
| Contact locations | When a company is created and the acting user holds Multi Company, the identity contact of every other company receives the inter-company transit location as its per-company delivery and receipt location, and the new company's identity contact receives the same for every other company |
| Rule selection | When a procurement looks for a rule delivering to the inter-company transit location, the rules delivering to the generic customer location are considered too, so that the ordinary delivery rules do not have to be duplicated |
| Elevated procurement | When a procurement runs with elevated rights, the rule search is explicitly restricted to rules whose company is empty or is a descendant of the procurement's company, because the record rules that would normally do it are skipped |

The two halves of the transfer are therefore two separate chains of documents, one per company, meeting at a location that belongs to neither. Valuation, ownership and the accounting consequences of each half are specified in [inventory operations](../domains/inventory-operations/README.md) and [inventory valuation and costing](../domains/inventory-valuation-and-costing/README.md).

### 11.4 Following a link into another company

A person who follows a link to a document of a company they have not activated would be refused by the company rule. The shared-link endpoint therefore widens the activation before giving up. The full algorithm is in [the security model, section 16.9](security-model.md#169-the-shared-link-redirection); its company part is:

1. Take the activated companies from the browser cookie, or the user's default company when the cookie is absent.
2. Try the read check on the record with those companies.
3. On refusal, take the record's suggested company ([section 5.5](#55-the-multi-company-hint-on-a-refusal)). If there is none, give up and redirect to the fallback page.
4. Retry the read check with the companies plus the suggested one. On success, write the widened list back into the cookie so that the page the person lands on is rendered with that company active. On refusal, give up and redirect to the fallback page.

The widening is silent and permanent for the session: the person arrives with one more company activated than before. It never grants access, because the suggested company must already be one of the user's allowed companies for the retry to pass the authorisation of [section 2.2](#22-deriving-the-current-company-and-the-activated-companies).

### 11.5 Documents printed across companies

A printed document is rendered with the company of the record switched in, so that the layout, the logo, the tagline, the footer, the paper format and the currency are the issuing company's. An external print endpoint refuses a set of records spanning several companies with **"Multi company reports are not supported."** rather than picking one.

### 11.6 People across companies

| Case | Rule |
|---|---|
| A user works in several companies | The user is linked to each of them; the default company decides where their new records land |
| A user of company A is named on a document of company B | Allowed when the user is allowed B as well: the company filter of the User entity is "the user's allowed companies intersect the given companies" |
| A contact is used by several companies | The contact must have no company, or a company that is an ancestor of the using company |
| The identity contact of a company | It belongs to that company; the list of all companies' identity contacts is cached, because many flows must recognise that a contact is one of the installation's own companies |
| An employee record | It belongs to one company; a person working for two companies has one employee record per company, and the link to the user is what ties them together |

---

## 12. Companies and identities

### 12.1 Effects of changing a user's companies

| Change | Effect |
|---|---|
| Adding or removing an allowed company | The automatic group adjustment runs: a user with at most one allowed company loses Multi Company, a user with more than one gains it. The adjustment runs on creation, on every write that touches the list, and on every unsaved record built in a form |
| Adding or removing an allowed company | Every environment of the current transaction whose acting user is among the modified users has its cached current company and activated companies reset |
| Changing the default company | The identity contact of the user follows, unless that contact is shared and has no company |
| Either of the two | The registry caches are cleared, because the allowed-companies set of a user is cached |
| Archiving a company | It silently leaves the allowed set of every user, because that set is defined as the **active** companies linked to the user |

### 12.2 The automatic groups

Two groups grant no permission at all and only control what the client shows:

| Group | Managed | Effect |
|---|---|---|
| `base.group_multi_company`, Multi Company | Automatically, from the number of a user's allowed companies | Shows the company switcher, every company field and every company grouping in the views |
| `base.group_multi_currency`, Multi Currency | Manually, from the settings | Shows the currency fields and the rate menus |

Because they grant nothing, hiding them protects nothing: a user without Multi Company who calls an operation naming a company field is subject to exactly the same gates as anybody else.

### 12.3 The public identity of a company

**Preconditions.** A company.

1. Read every user holding the public-user group, with elevated rights, archived ones included.
2. Take the first of them whose default company is the given company; if there is one, return it.
3. Otherwise duplicate the platform's public user, setting the name to "Public user for " followed by the company name, a per-company dead sign-in name, the default company to the given company, and the allowed companies to that company alone.

The elevated read at step 1 is required: the company rules would otherwise hide the public users attached to other companies. Each site of the multi-site capability also names its own public user, which is how one installation serves several public front ends, each anchored in its own company.

---

## 13. What changes at the instant of a switch

The switch itself writes nothing in the database. It changes the activated companies of every subsequent request, and through them:

| Observable | Effect of the switch |
|---|---|
| Records returned by a search | Recomputed: the company rule contributes a different condition ([section 5](#5-company-scoping-through-record-rules)) |
| A refusal on a record that is still open | The client checks the read permission before reloading and drops the record from the history when it is lost ([section 3.3](#33-the-company-switcher)) |
| Default values of a company field | The new current company |
| Per-company field values | The entries of the new current company, fallback included ([section 7.1](#71-per-company-field-values)) |
| User default values | The defaults of the new current company take precedence ([section 7.2](#72-user-default-values)) |
| Numbering sequences | The numbering of the new current company ([section 7.3](#73-sequences)) |
| The currency shown as "the company currency" | The new current company's currency; amounts already stored are untouched |
| Candidate lists of company-checked relations | Rebuilt against the new company ([section 4.3](#43-company-checked-relations)) |
| Composed record-rule filters | A different cache entry; nothing is invalidated ([section 5.4](#54-interaction-with-the-composition-laws)) |
| Per-company cached field values | A different cache entry; nothing is invalidated ([section 14](#14-caching-keyed-by-company)) |
| Records already created | Unchanged: a record keeps the company it was created in |
| Permissions | Unchanged: groups, access rights and field restrictions do not depend on the company |
| The user's allowed companies | Unchanged: an activation is a subset of them, never an extension |

---

## 14. Caching keyed by company

| Cache | Key | Cleared by |
|---|---|---|
| Composed record-rule filter | Acting identity, unrestricted flag, entity, operation, **activated companies** | Any change to a record rule |
| Per-company field value | Record, field, **current company** | Writing the field, or any change to a User Default Value |
| Allowed companies of a user | The user | Creating or deleting a company; changing a company's active flag or presentation sequence; changing a user's companies |
| Accessible branches | Activated companies, company, acting identity | Creating or deleting a company; changing the active flag or the presentation sequence |
| User default values of an entity | Acting identity, **current company**, entity, condition | Any change to a User Default Value |
| The identity contacts of all companies | None; the cache is global | Creating or deleting a company |
| The assembled view of a currency-sensitive entity | The view key plus the **current company's currency name** | Any view change |
| Presentation assets | The asset key | Writing a company's font, primary colour, secondary colour or document template |

Creating or deleting a company clears the registry caches unconditionally; writing a company clears them when the active flag or the presentation sequence changed. Every one of these invalidations is signalled to the other workers by the mechanism of [the architecture, section 12](architecture.md#12-cross-process-coherence).

---

## 15. Messages

| Situation | Message |
|---|---|
| Activating a company the acting user is not allowed | "Access to unauthorized or invalid companies." |
| A default company not among the allowed companies | "Company \<company name\> is not in the allowed companies for user \<user name\> (\<allowed company names\>)." |
| Archiving a company that is still someone's default | "The company \<company name\> cannot be archived because it is still used as the default company of \<number\> users." |
| Duplicating a company | "Duplicating a company is not allowed. Please create a new company instead." |
| Changing the parent of a company | "The company hierarchy cannot be changed." |
| A root-delegated value differing between a branch and its parent | "The \<value label\> of a subsidiary must be the same as it's root company." |
| Two companies with the same name | "The company name must be unique!" |
| A contact representing a company but attached to another | "The company assigned to this partner does not match the company this partner represents." |
| Company inconsistency, header | "Uh-oh! You’ve got some company inconsistencies here:" |
| Company inconsistency, ordinary line | "- “\<the record's display name\>” belongs to company “\<company names\>” while “\<field label\>” (\<field name\>: \<target display names\>) belongs to another company." |
| Company inconsistency, the record is a company | "- Record is company “\<company name\>” while “\<field label\>” (\<field name\>: \<target display names\>) belongs to another company." |
| Company inconsistency, a non-root company where a root is required | "- Only a root company can be set on “\<the record's display name\>”. Currently set to “\<company name\>”" |
| Company inconsistency, footer | "To avoid a mess, no company crossover is allowed!" |
| A currency rate attached to a branch | "Currency rates should only be created for main companies" |
| Archiving a currency used by a company | "This currency is set on a company and therefore cannot be deactivated." |
| A rate that is not strictly positive | "The currency rate must be strictly positive." |
| Two rates for the same day, currency and company | "Only one currency rate per day allowed!" |
| Printing an external document for records of several companies | "Multi company reports are not supported." |
| Changing the company of a warehouse | "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |
| Deleting the shared inter-company transit location | "The \<location name\> location is required by the Inventory app and cannot be deleted, but you can archive it." |
| A record gate refusal with several candidate companies | "Note: this might be a multi-company issue. Switching company may help." |
| A record gate refusal with one reachable company | "This seems to be a multi-company issue, you might be able to access the record by switching to the company: \<company name\>." |
| A record gate refusal with one unreachable company | "This seems to be a multi-company issue, but you do not have access to the proper company to access the record anyhow." |

---

## 16. Acceptance criteria

Companies 1 (root), 2 and 3 (branches of 1) and 4 (unrelated root). Unless stated otherwise the acting user is an internal user allowed 1, 2, 3 and 4, whose default company is 1.

### Activation

**AC-MC-1.** *Given* no activation value in the context, *when* the current company is read, *then* it is the user's default company; and *when* the activated companies are read, *then* they are **all four** allowed companies, not the default one alone.

**AC-MC-2.** *Given* the context activates companies 2 and 1 in that order, *when* the current company is read, *then* it is company 2.

**AC-MC-3.** *Given* the context activates company 5, which the user is not allowed, *when* the current company or the activated companies are read in a restricted environment, *then* the operation is refused with "Access to unauthorized or invalid companies."; and *when* the same is read in an unrestricted environment, *then* company 5 is returned without refusal.

**AC-MC-4.** *Given* the context activates companies 1 and 2, *when* a flow switches to company 2, *then* the activation becomes 2 then 1; and *when* it switches to company 1, *then* the activation is unchanged.

**AC-MC-5.** *Given* no activation value, *when* a flow switches to company 3, *then* the activation becomes company 3 alone, not the user's four companies with 3 first.

**AC-MC-6.** *Given* a cookie naming a company the user has lost access to, *when* the session starts, *then* the activation falls back to the user's default company alone and the cookie is rewritten.

**AC-MC-7.** *Given* the switcher, *when* the person toggles company 1, *then* companies 2 and 3 are toggled with it; and *when* the person then toggles company 2 off, *then* company 1 stays selected.

**AC-MC-8.** *Given* the selection is exactly companies 1, 2 and 3, *when* the person logs into company 4, *then* the selection becomes company 4 alone; and *given* the selection is companies 1 and 4, *when* the person logs into company 4, *then* the selection stays 1 and 4 with 4 first.

**AC-MC-9.** *Given* a document of company 4 is open and the person deactivates company 4, *when* the selection is applied, *then* the document is dropped from the navigation history and the reload does not land on a refusal.

### Scoping

**AC-MC-10.** *Given* an entity whose rule is shape A, with records in companies 1 and 4 and one shared record, *when* the user activates company 1, *then* a search returns the record of company 1 and the shared record.

**AC-MC-11.** *Given* an entity whose rule is shape B, *when* the user activates company 2, *then* records of company 1 are **not** returned.

**AC-MC-12.** *Given* an entity whose rule is shape C, *when* the user activates company 2, *then* records of company 1, records of company 2 and shared records are all returned.

**AC-MC-13.** *Given* the user activates company 1, *when* the Company entity is read, *then* only company 1 comes back although the user is allowed four; and *given* the user also holds the access-rights administrator group, *then* all companies come back.

**AC-MC-14.** *Given* a contact with no company and a contact of company 4, *when* the user activates company 1, *then* the first is visible and the second is not; and *given* the contact of company 4 is the identity contact of an internal user, *then* it is visible too.

**AC-MC-15.** *Given* an entity with a company field and the standard global rule, *when* a user whose selection is company 1 searches, *then* only records whose company is company 1 or empty are returned.

**AC-MC-16.** *Given* a record the user may write, *when* the user writes another company on it and that company is not activated, *then* the write succeeds and the record disappears from the user's searches.

### Consistency

**AC-MC-17.** *Given* a document of company 4 and a journal of company 1, *when* the journal is written on the document, *then* the write is refused with a line naming the record, its company, the field label and the journal, beginning "Uh-oh! You’ve got some company inconsistencies here:" and ending "To avoid a mess, no company crossover is allowed!".

**AC-MC-18.** *Given* a document of company 2 and a journal of company 1, *when* the journal is written on the document, *then* it is allowed, because Journal is shared down the hierarchy.

**AC-MC-19.** *Given* a shared product with an empty company, *when* it is used on documents of companies 1 and 4, *then* both are allowed.

**AC-MC-20.** *Given* a per-company relational field marked company-checked, *when* it is written while company 4 is current with a target of company 1, *then* it is refused; and *when* the same value is written while company 1 is current, *then* it is allowed and only company 1's entry changes.

**AC-MC-21.** *Given* an elevated flow, *when* it writes a cross-company link, *then* the consistency check still refuses it, because it is a validation and not a permission.

**AC-MC-22.** *Given* a write that changes the company link of a record, *when* it runs, *then* every company-checked field of the record is re-validated, not only the written ones.

**AC-MC-23.** *Given* a target record of the wrong company that has been archived, *when* the check runs, *then* it is still caught, because the check makes archived records visible.

**AC-MC-24.** *Given* a target record the acting user may not read, *when* the check runs, *then* the refusal is the consistency refusal and not a permission refusal, because the targets are read with elevated rights.

### Branches

**AC-MC-25.** *Given* company 1 with the euro as its currency and branches 2 and 3, *when* the currency of company 1 is set to the Swiss franc, *then* companies 2 and 3 are set to the Swiss franc in the same transaction.

**AC-MC-26.** *Given* branch 2, *when* its currency alone is changed, *then* the change is refused with "The Currency of a subsidiary must be the same as it's root company."

**AC-MC-27.** *Given* branch 2, *when* its parent is written, *then* the write is refused with "The company hierarchy cannot be changed."

**AC-MC-28.** *Given* company 1 is archived, *when* its branches are read, *then* they are archived too.

**AC-MC-29.** *Given* company 1 is still the default company of one active user, *when* it is archived, *then* the archive is refused with "The company Company 1 cannot be archived because it is still used as the default company of 1 users."

**AC-MC-30.** *Given* a currency rate created while branch 2 is current, *when* it is saved, *then* its company is company 1; and *when* a rate is explicitly attached to branch 2, *then* it is refused with "Currency rates should only be created for main companies".

### Currency

**AC-MC-31.** *Given* the rate records of [section 8.4](#84-converting), *when* 100.00 euro is converted to United States dollars on 12 March 2026 for company 1, *then* the result is 108.50.

**AC-MC-32.** *Given* the same rates, *when* 100.00 United States dollars is converted to euro on 20 March 2026, *then* the result is 90.91, rounded from 90.909090… to the euro's two decimal places.

**AC-MC-33.** *Given* a date earlier than every rate record of a currency, *when* a conversion is performed, *then* the earliest known rate is used and not a rate of one.

**AC-MC-34.** *Given* a shared rate record and a company-specific rate record for the same currency at an earlier date, *when* a conversion is performed for that company at a date after both, *then* the company-specific record is used.

### Per-company values and defaults

**AC-MC-35.** *Given* a per-company field holding 5.00 for company 1 and nothing for company 4, *when* it is read while company 4 is current, *then* the fallback is returned and no write has happened.

**AC-MC-36.** *Given* a user default value defined for company 1 and another defined for no company, *when* a record is created while company 1 is current, *then* the first wins; and while company 4 is current, *then* the second wins.

**AC-MC-37.** *Given* two sequences sharing one code, one for company 1 and one shared, *when* a number is drawn while company 1 is current, *then* the company's sequence is used; and while company 4 is current, *then* the shared one is used.

### Cross-company flows

**AC-MC-38.** *Given* an invoice of 1,000.00 euro in company A settled by a payment recorded in company B, both companies having their inter-company clearing journal and accounts, *when* the settlement runs, *then* two balanced entries are created, one per company; the payment's outstanding line and the invoice's receivable line are reconciled; the invoice becomes fully paid; and company B carries an inter-company payable of 1,000.00 while company A carries an inter-company receivable of 1,000.00.

**AC-MC-39.** *Given* warehouses in two different companies, *when* a resupply route is created between them, *then* the route passes through the shared inter-company transit location; and *given* two warehouses of the same company, *then* the route passes through that company's internal transit location.

**AC-MC-40.** *Given* a notification link to a document of a company the person has not activated, *when* the person follows it, *then* the activation is widened with the document's suggested company and the document opens; and *given* the person is not allowed that company, *then* they are redirected to the fallback page and no company is added.

**AC-MC-41.** *Given* an external print request covering records of two companies, *when* it runs, *then* it is refused with "Multi company reports are not supported."

### Identities

**AC-MC-42.** *Given* a user with one allowed company, *when* a second company is added, *then* the user gains Multi Company; and *when* it is removed again, *then* the user loses it.

**AC-MC-43.** *Given* a company is created by a user, *when* the creation completes, *then* the company is in the allowed companies of that user and of the root identity.

**AC-MC-44.** *Given* a company with no public identity of its own, *when* one is requested, *then* the platform's public user is duplicated with that company as its only allowed company and as its default company.

**AC-MC-45.** *Given* a user whose default company is archived, *when* the user is unarchived, *then* it is refused with "Company \<company name\> is not in the allowed companies for user \<user name\> (\<allowed company names\>)."

---

## 17. Invariants a rebuild must preserve

1. An empty company on a record means "shared by every company", both for visibility and for compatibility.
2. An empty activation list means all of the acting user's companies, and the current company is the user's default company.
3. A restricted environment may not activate a company the user is not allowed; an unrestricted one may.
4. Company scoping is expressed as record rules and nothing else; the rules are global wherever isolation must be guaranteed.
5. A branch sees its root's master data and not its root's documents.
6. Root-delegated values are identical across a whole tree, and writing one on a root writes it on every descendant in the same transaction.
7. The company hierarchy cannot be changed after creation.
8. The company consistency check is a validation: it runs with elevated rights and it makes archived records visible.
9. A per-company value is read and validated against the current company, never against the record's own company.
10. Switching company changes cache keys; it never invalidates a cache and never writes anything.
11. Groups, access rights and field restrictions never depend on the company.
12. The two company-visibility groups grant no permission; hiding a field from them protects nothing.

---

## 18. Reconciliation notes

Four behaviours in this document contradict the reading a careful person would most naturally arrive at, and two decisions about naming and ownership are recorded with them. Each was verified against the running system.

1. **The meaning of an empty activation list.** When the context carries no activation, the environment does **not** fall back to the user's default company for both values. The behaviour is asymmetric: the current company is the default company, but the activated companies are **all** the user's companies. [Section 2.2](#22-deriving-the-current-company-and-the-activated-companies) states the asymmetry and explains why it is deliberate; criterion AC-MC-1 asserts it.
2. **The shape of the company rule.** "The record's company is empty or among the allowed companies" is one canonical shape out of five, not the canonical shape. Shapes B, C, D and E are all in use, and the difference between them is what makes branches work. [Section 5.2](#52-the-five-canonical-shapes) enumerates all five with a worked evaluation, and keeps the one-shape statement as shape A.
3. **The compatibility rule of the consistency check.** Equality of the linked record's company with the owning company, or emptiness, is the default family only. The hierarchy-sharing family accepts any ancestor, the list-scoped family tests the company list, and the User entity tests the intersection with the allowed companies. [Section 6.2](#62-the-company-filter-of-a-target-entity) gives all four families, and the simple form is kept as the formula of the default family.
4. **Where per-company values are checked.** A per-company field is **not** checked against the record's own company. It is checked against the **current** company, which is a different company whenever the record is shared. [Section 6.4](#64-the-algorithm), step 5.2, states it, and criterion AC-MC-20 asserts it.
5. **Naming of the company fields.** The link fields are reproduced by their storage names, `company_id` and `company_ids`, because an integration and a data import depend on them; every occurrence carries its full name in words.
6. **Where this material lives.** Company scoping and company consistency could have stayed inside the security model. They are here, because scoping is an application of record rules rather than a gate, and because the company tree, per-company values, currency, branches and cross-company flows are far larger than the part the gates need. [The security model, section 12](security-model.md#12-company-scoping-and-company-consistency) keeps that part and links here. The scenarios of this document are numbered in one series with the prefix `AC-MC` in [section 16](#16-acceptance-criteria).

---

## Related documents

- [The security model](security-model.md) — the four data gates, the record-rule layer that company scoping uses, and the refusal messages that carry the multi-company hint.
- [The entity and field system](entity-and-field-system.md) — per-company field values, the company consistency declaration on a field, and the filter grammar the rules are written in.
- [Architecture](architecture.md) — the environment that carries the company selection, and the cross-worker signalling behind the invalidations of [section 14](#14-caching-keyed-by-company).
- [Views and actions](views-and-actions.md) — the candidate filters and company groupings that the client renders.
- [Client architecture](client-architecture.md) — the company switcher and the activation the client sends with every call.
- [Design principles](design-principles.md) — multi-company by record, and why the empty company means shared.
- [Multi-currency](../domains/multi-currency/README.md) — currencies, rate providers and exchange differences.
- [The general ledger](../domains/general-ledger/README.md) — the accounting treatment of company currency and of inter-company positions.
- [Payments and bank reconciliation](../domains/payments-and-bank-reconciliation/README.md) — the settlement flow of [section 11.2](#112-inter-company-settlement-of-a-payment).
- [Inventory operations](../domains/inventory-operations/README.md) and [inventory valuation and costing](../domains/inventory-valuation-and-costing/README.md) — the transit-location flow of [section 11.3](#113-inter-company-goods-movements).
- [Identity and access](../domains/identity-and-access/entities.md) — the Company and User field catalogues and the shipped company records.
