# Identity and Access — Business Rules

This file specifies the decisions the system takes about permission and identity: the exact
algorithms, the exact conditions under which each fails, and the exact message produced. Three
procedures are load-bearing for the whole platform and are given as fully numbered algorithms:

> **Note on reproduced texts.** Message texts, button labels and screen titles are reproduced
> **verbatim**, exactly as the system emits them, because external tests and user documentation
> depend on them. Where such a reproduced text contains an abbreviation, the abbreviation belongs to
> the text and is not this document's own prose. The abbreviations that occur are: *API* for
> application programming interface; *2FA* for two-factor authentication; *LDAP* for the central
> directory protocol; *DN* for distinguished name; *OAuth* and *UID* for the delegated sign-in
> protocol and its subject identifier; *JSON* for the structured-literal notation; *HTTP* for the
> transport protocol; *ID* for identifier. Outside such reproduced texts, every term is written in
> full.

- **the access-checking algorithm** (section 2),
- **the rule-combination semantics** (section 3),
- **the privilege-elevation semantics** (section 5).

A fourth, the **settings mechanism**, is specified in
[configuration.md](configuration.md) section 2 because it is configuration rather than a runtime
decision, but it is equally exact there.

---

## 1. The permission model in one page

Permission is decided in **four independent layers**. All four must allow an operation for it to
happen; none of them can grant what a lower layer refuses.

| Layer | Granularity | Answers | Specified in |
|---|---|---|---|
| 1. Entity | one entity, one operation | *May this user touch records of this entity at all?* | Section 2.3 |
| 2. Record | a set of records, one operation | *Which of these particular records may this user touch?* | Sections 2.4 and 3 |
| 3. Field | one field, read or write | *May this user see or change this column?* | Section 6 |
| 4. Consistency | one record and its relations | *Do the companies of the linked records agree?* | Section 7 |

The four operations are named exactly: **read**, **create**, **write** (meaning modify) and
**unlink** (meaning delete). No other operation name exists; any other value is rejected as a
programming error.

Two escape hatches exist and only two:

- **privilege elevation**, which switches layers 1, 2 and 3 off for the duration of a nested piece of
  work while leaving the acting identity unchanged (section 5);
- **the signed access token**, which lets a specific external party read a specific record without
  an account (section 12).

Everything else — every "manager sees everything", every "salesperson sees only theirs", every
multi-company isolation — is expressed with groups, access rights and record rules.

---

## 2. The access-checking algorithm

### 2.1 The four entry points

| Entry point | Behaviour | Typical use |
|---|---|---|
| **Check** | Raises on the first problem; returns nothing on success. | Called at the top of every read, create, write and delete. |
| **Has access** | Returns true or false; never raises. | Deciding whether to show a button. |
| **Filter by access** | Returns the subset of the records the user may operate on. | Listing records a user may act upon. |
| **Core check** | Returns either *nothing* (meaning permitted), or the pair (forbidden records, a way to build the corresponding refusal). | The single implementation the other three share. |

All four take exactly one argument: the operation.

### 2.2 The core procedure

**Preconditions.** An operation name in {read, create, write, unlink}; a set of records of one
entity, possibly empty, possibly containing records that do not yet exist in storage.

**Postcondition.** Either *nothing* (permitted), or a pair: the forbidden subset and a builder for
the refusal.

1. **Elevation short-circuit.** If the environment is privilege-elevated, the answer is *permitted*.
   (The three public entry points test this before even calling the core procedure: *check* does
   nothing, *has access* answers true, *filter by access* returns everything.)
2. **Entity layer.** Ask the entity-level check (section 2.3) whether the acting user holds the
   operation on this entity, in the non-raising form. If the answer is no, return the pair (**all**
   the records in the set, a builder for the entity-level refusal of section 2.5).
3. **Record layer, only for records that exist.** If **any** identifier in the set is a real stored
   identifier:
   1. Compute the combined rule filter for (entity, operation) by the procedure of section 3.
   2. If the filter is empty — meaning no rule constrains this operation for this user — skip to
      step 4.
   3. Take the same set of records **with elevation** and **with the archive filter disabled**, and
      keep only those satisfying the filter. Call the result the *permitted subset*.
   4. The *forbidden subset* is the original set minus the permitted subset.
   5. If the forbidden subset is non-empty, return the pair (forbidden subset, a builder for the
      record-level refusal of section 2.6).
4. Return *nothing*: the operation is permitted on every record in the set.

**Notes that matter for a faithful re-implementation.**

- Step 3 evaluates the filter with elevation. This is essential: evaluating it as the acting user
  would re-enter the very check being performed and would also hide records the rules are about to
  judge.
- Step 3 disables the archive filter. Without this, an archived record would be reported as
  *forbidden* rather than simply absent, producing a misleading refusal.
- Step 3 is skipped entirely when the set contains only records that do not yet exist in storage,
  because there is nothing to filter.
- A set containing a mixture of stored and unstored records is filtered as a whole; unstored records
  never satisfy a stored filter and would be reported as forbidden, so callers do not mix them.
- Calling **check** on an **empty** set is meaningful and is used deliberately: it performs step 2
  only, and therefore answers "does this user have any permission at all on this entity?".

### 2.3 The entity-level check

**Preconditions.** An entity transport name; an operation; a flag saying whether to raise.

1. If the environment is privilege-elevated, answer *permitted*.
2. Assert that the entity argument is a name, not a record set. (A programming error otherwise.)
3. If the name is not a known entity, log an error and continue — the lookup below will simply not
   find it.
4. Look up the **permitted-entity set** for (acting user, operation). This is cached; see step 5 for
   how it is built.
5. The permitted-entity set is the set of entity names *E* for which at least one access-right row
   exists such that: the row is active; the row grants the requested operation; and either the row
   names no group, or the row's group is in the acting user's **group closure** (section 4). The set
   is computed with a single grouped query over the access-right rows joined to the entity
   definitions, and is cached per (acting user, operation).
6. The answer is *permitted* when the entity name is in that set.
7. If the answer is *refused* and the caller asked to raise, build and raise the refusal of section
   2.5. Otherwise return the boolean.

**Deny by default.** An entity with no access-right row at all is forbidden to everyone who is not
elevated — including the administrator. This is why every package ships access-right rows for every
entity it defines.

**An access-right row with no group grants the operation to everyone**, including external and
anonymous users. Creating such a row logs a warning naming the row, because it is almost always a
mistake.

### 2.4 The record-level check

Given the combined filter produced by section 3, the record-level check is exactly the fourth step
of section 2.2: keep the records satisfying the filter, refuse the rest. There is no separate
procedure; the whole difficulty is in building the filter.

### 2.5 Building the entity-level refusal

**Inputs.** The entity name, the operation.

1. Log, at informational level, that access was denied by access rights, naming the operation, the
   acting user identifier and the entity.
2. Take the entity's human label (its definition's name), falling back to the transport name when
   there is none.
3. Compose part one from the operation:

   | Operation | Text |
   |---|---|
   | read | `You are not allowed to access '<label>' (<transport name>) records.` |
   | write | `You are not allowed to modify '<label>' (<transport name>) records.` |
   | create | `You are not allowed to create '<label>' (<transport name>) records.` |
   | unlink | `You are not allowed to delete '<label>' (<transport name>) records.` |

4. Build the list of groups that hold the operation on the entity (section 2.7). Compose part two:
   - when the list is non-empty: `This operation is allowed for the following groups:` followed by a
     line break and then, for each group, a tabulation, a hyphen, a space and the group's
     presentation name, one per line;
   - when the list is empty: `No group currently allows this operation.`
5. Part three is always: `Contact your administrator to request access if necessary.`
6. The final message is part one, a blank line, part two, a blank line, part three.

### 2.6 Building the record-level refusal

**Inputs.** The operation and the forbidden records.

1. Log, at informational level, that access was denied by record rules, naming the operation, up to
   the first six forbidden identifiers, the acting user identifier and the entity.
2. Re-derive the language and time zone context from the acting user, so the message is produced in
   that user's language.
3. Take the entity's human label, falling back to the transport name.
4. Translate the operation to a word: read → `read`, write → `write`, create → `create`,
   unlink → `unlink`.
5. Compose the user description as the acting user's name, a space, and `(id=<the identifier>)` in
   parentheses.
6. Compose the opening:

   ```
   Uh-oh! Looks like you have stumbled upon some top-secret records.

   Sorry, <user description> doesn't have '<operation word>' access to:
   ```

7. Compose the failing-entity line: `- <label> (<transport name>)`.
8. Compose the resolution text, beginning with:
   `If you really, really need access, perhaps you can win over your friendly administrator with a
   batch of freshly baked cookies.`
9. Determine the **failing rules** by the diagnosis procedure of section 3.6. Take the first six
   forbidden records, read with elevation.
10. Decide whether the problem looks like a company problem: it does when **any** failing rule's
    filter text mentions the company field.
11. If it looks like a company problem, ask the forbidden records for their *suggested company*
    (each entity may implement this; the default is the record's own company):
    - if there is more than one suggestion, append to the resolution text a blank line and
      `Note: this might be a multi-company issue. Switching company may help - in the system, not in
      real life!`;
    - if there is exactly one suggestion and it is among the acting user's permitted companies,
      attach to the refusal a structured hint naming that company's identifier and display name, and
      append a blank line and
      `This seems to be a multi-company issue, you might be able to access the record by switching to
      the company: <the company display name>.`;
    - if there is exactly one suggestion and it is **not** among the permitted companies, append a
      blank line and
      `This seems to be a multi-company issue, but you do not have access to the proper company to
      access the record anyhow.`
12. Choose the level of detail:
    - **Ordinary** — when the acting user does not hold the *Technical Features* group, or is not an
      internal user. The message is: the opening, a line break, the failing-entity line, a blank
      line, the resolution text.
    - **Detailed** — otherwise. In place of the failing-entity line, one line per forbidden record
      (at most six), each of the form `- <label>, <display name> (<transport name>: <identifier>)`,
      or, when the problem looks like a company problem *and* the record's company is among the
      user's permitted companies, `- <label>, <display name> (<transport name>: <identifier>,
      company=<company display name>)`. Then a blank line, then `Blame the following rules:` and one
      line per failing rule of the form `- <rule name>`, then a blank line, then the resolution text.
13. Invalidate the cached values of the forbidden records, because building the detailed message
    read their display names and evaluated filters against them.
14. Return the refusal, carrying the company hint when one was produced.

Note the deliberate asymmetry in step 12: external and anonymous users never hold the *Technical
Features* group even in developer mode, so the detailed form — which discloses record names and rule
names — is never shown to them.

### 2.7 Listing the groups that hold an operation

For an entity and an operation, select from the access-right rows joined to the entity definitions,
the group definitions and (left-joined) the privilege definitions: rows of that entity that are
active and grant that operation and whose group is set. For each, produce the presentation name:
the privilege name, a solidus, and the group name when there is a privilege; the group name alone
otherwise. Both names are taken in the acting language, falling back to the reference language.
Order by privilege name then group name, with rows lacking a privilege last.

---

## 3. The rule-combination semantics

This is the single most consequential algorithm in the domain. It converts the set of record rules
into **one filter** that the records must satisfy.

### 3.1 The intuition

- A rule with **no group** is a *global* rule. It expresses an invariant of the installation — "you
  may only see records of a company you are active in", "you may only see documents of your own
  website". Global rules are **conjunctive**: every one of them must hold.
- A rule with **groups** is a *grant*. It expresses "members of this group may additionally see
  these records" — "a salesperson sees their own orders", "a sales manager sees all orders". Grants
  that apply to the acting user are **disjunctive**: satisfying any one of them is enough.
- The two are combined by conjunction: the record must satisfy every global rule **and** at least
  one applicable grant (when any grant applies at all).

In symbols, writing *G₁ … Gₙ* for the applicable global filters and *R₁ … Rₘ* for the applicable
grant filters:

```formula
combined_filter = G₁ and G₂ and … and Gₙ and ( R₁ or R₂ or … or Rₘ )
```

with the parenthesised part omitted entirely when *m* = 0, and the whole expression being the
always-true filter when *n* = 0 and *m* = 0.

### 3.2 Selecting the applicable rules

**Inputs.** An entity transport name, an operation.

1. If the operation is not one of the four, this is a programming error and the procedure stops with
   `Invalid mode: <the value>`.
2. If the environment is privilege-elevated, the applicable-rule set is **empty**.
3. Otherwise select the identifiers of the rules such that:
   - the rule's entity is the requested entity;
   - the rule is active;
   - the rule's flag for the requested operation is set;
   - and **either** the rule is global (has no group) **or** the rule is attached to at least one
     group whose identifier is in the acting user's **group closure** (section 4).
4. Order the result by rule identifier ascending. The ordering is part of the specification: it makes
   the produced filter deterministic and therefore cacheable and comparable.

The group test in step 3 uses the *closure*, not the explicitly assigned groups: a rule attached to
a group that the user reaches only through implication applies exactly as if the group were assigned
directly.

### 3.3 Building the combined filter

**Inputs.** An entity transport name; an operation (default *read*).

1. Start with an empty list of **global filters**.
2. **Inherited entities.** For every entity this entity delegates to (the delegation targets, each
   reached through one stored link field):
   1. Skip the delegation if its link field is not stored.
   2. Recursively build the combined filter of the delegated entity for the same operation.
   3. If that filter is non-empty, append to the global filters the condition "the delegation link
      satisfies that filter".

   This is what makes a User inherit the Contact's rules: reading a user requires satisfying the
   contact rules too.
3. Select the applicable rules by section 3.2.
4. If there are none, the answer is the conjunction of the global filters accumulated in step 2,
   optimised for the entity, and the procedure ends.
5. Build the evaluation context (section 3.4).
6. Take the acting user's group closure.
7. Start with an empty list of **grant filters**.
8. For each applicable rule, read with elevation, in identifier order:
   1. If the rule has groups and none of them is in the closure, skip it. (This is a defensive
      second test; the selection already applied it.)
   2. Evaluate the rule's filter text in the evaluation context to obtain a filter. An empty filter
      text yields the always-true filter.
   3. If the rule has groups, append the filter to the **grant** list; otherwise append it to the
      **global** list.
9. If the grant list is non-empty, append the **disjunction** of the grant filters to the global
   list.
10. The answer is the **conjunction** of the global list, optimised for the entity.

### 3.4 The evaluation context

Exactly three names are in scope while a rule's filter text is evaluated:

| Name | Value | Note |
|---|---|---|
| the acting user | The acting user's record, taken **with an empty context** | The empty context is essential: it makes the filter independent of language, archive-filtering and any other contextual key, which is what makes the result safely cacheable. |
| the active company identifiers | The identifiers of the companies currently active, first one first | These are already filtered and trusted: reading them raised a refusal if the user asked for a company they are not permitted in (section 8.2). |
| the active company identifier | The identifier of the first active company | |

Nothing else is available: no arbitrary code, no other entities, no request data.

### 3.5 Caching the combined filter

The combined filter is cached under the key: acting user identifier, elevation flag, entity name,
operation, and the tuple of the values of the **cache-key context entries**.

- The only cache-key context entry is the list of active companies. A list value is converted to a
  tuple before it enters the key.
- Any create, write or delete of a rule flushes all pending work and clears every cache.
- When the developer mode for definition files is active, the cache is bypassed entirely so that a
  changed definition takes effect immediately.

### 3.6 Diagnosing which rules failed

Used only to build the refusal message. **Inputs.** The forbidden records, the operation.

1. Take an empty record set of the same entity, elevated, with the archive filter disabled. Call it
   the *probe*. Using elevation here is required: the diagnosis must be able to count records the
   acting user cannot see.
2. Build the evaluation context of section 3.4.
3. Select all applicable rules for (entity, operation) by section 3.2, read with elevation.
4. Let the *grant rules* be those applicable rules that have groups **and** whose groups intersect
   the acting user's closure.
5. Form the **disjunction** of the grant rules' filters (an empty filter text contributing the
   always-true filter).
6. Count, through the probe, the records that satisfy both that disjunction and "identifier is one
   of the forbidden identifiers". If that count equals the number of forbidden records, the grant
   rules are **not** the cause; set the grant-rule set to empty.
7. Define a global rule as *failing* when the count of forbidden records satisfying its own filter is
   **strictly less** than the number of forbidden records.
8. The failing rules are: every rule still in the grant-rule set after step 6, plus every global rule
   that is failing by step 7. They are returned attached to the acting user, so that the message can
   render their names in the right language.

The asymmetry between steps 6 and 7 mirrors the combination semantics exactly: the grants either
succeed as a group or fail as a group, whereas each global rule can fail independently.

### 3.7 Worked example — a salesperson limited to their own documents

Suppose the Sales Order entity carries three rules:

| Rule | Groups | Filter | Operations |
|---|---|---|---|
| *Sales Order: multi-company* | none (global) | the order's company is one of the active companies, or the order has no company | all four |
| *Sales Order: personal* | *Sales / User: own documents only* | the order's salesperson is the acting user | all four |
| *Sales Order: all* | *Sales / User: all documents* | always true | all four |

**User A** is in *Sales / User: own documents only*, is active in company 1 only.

- Applicable rules: the multi-company rule (global) and the personal rule (grant).
- Global list after step 8: `company is in [1] or company is empty`.
- Grant list: `salesperson = A`.
- Combined: `(company is in [1] or company is empty) and (salesperson = A)`.

Reading order number SO0042, whose salesperson is B and whose company is 1: the record satisfies the
global part but not the grant part, so it is forbidden. Diagnosis: step 6 counts 0 of 1 forbidden
records satisfying the grant disjunction, so the grant rule stays in the set; step 7 finds the
global rule satisfied by 1 of 1, so it is not failing. The refusal names *Sales Order: personal*.

**User B** is in *Sales / User: all documents*, which — being the more powerful choice of the same
privilege — also implies *Sales / User: own documents only*.

- Applicable rules: the multi-company rule and **both** grants.
- Grant list: `salesperson = B` **or** `always true`, which reduces to always true.
- Combined: `company is in [1] or company is empty`.

So the manager sees every order of their active companies. This is why the grants must be combined
by union and not by intersection: intersecting would make the more powerful group **less**
powerful, because it would also have to satisfy the narrower grant.

### 3.8 Worked example — the effect of the active-company list

Take user A above, but now permitted in companies 1 and 2 and currently active in both. The active
company identifiers are `[1, 2]`. The global filter becomes `company is in [1, 2] or company is
empty`, so orders of company 2 become readable without any change to the rules. Switching the active
set back to `[1]` immediately hides them again — and, because the active list is part of the cache
key, the previously cached filter is not reused.

---

## 4. The group closure

**Definition.** The *group closure* of a user is the smallest set *C* of groups such that:

1. every group explicitly assigned to the user is in *C*;
2. if a group *g* is in *C* and *g* implies *h*, then *h* is in *C*.

Equivalently, it is the reflexive transitive closure of the implication relation applied to the
explicitly assigned groups.

**Computation.** The implication graph is compiled once into a cached structure keyed by group
identifier, holding for each group its external references, the identifiers of the groups it implies
(its supersets) and the identifiers of the groups it is disjoint from. The closure of a set *S* is
then *S* together with the superset closure of *S*.

**Worked example.** Suppose:

- *Sales / Administrator* implies *Sales / User: all documents*;
- *Sales / User: all documents* implies *Sales / User: own documents only*;
- *Sales / User: own documents only* implies *Role / User*;
- *Role / Administrator* implies *Access Rights* and *Bypass HTML Field Sanitize*;
- *Access Rights* implies *Role / User*;
- *Role / User* implies *Technical Features*.

A user explicitly assigned {*Sales / Administrator*, *Role / Administrator*} has the closure

```
{ Sales / Administrator,
  Sales / User: all documents,
  Sales / User: own documents only,
  Role / User,
  Technical Features,
  Role / Administrator,
  Access Rights,
  Bypass HTML Field Sanitize }
```

— eight groups from two. Every access-right row and every record rule attached to any of those eight
now applies. The derived *Share User* flag is false because *Role / User* is present. The derived
*Role* field reads *Administrator* because *Role / Administrator* is present.

**Caching.** The closure of one user is cached by user identifier and is computed with an empty
context. Any write to a user's groups, or any change to the implication graph, clears the caches.

**Group membership tests.**

- *Belongs to group G* — the identifier of *G*, looked up from its external reference in the
  compiled definitions, is in the closure. For an unsaved record the closure is taken from the
  in-memory value rather than from the cache.
- The group *Technical Features* is special: the public test reports membership only when the
  current request is additionally in developer mode. The internal test ignores developer mode.
- The public test refuses to answer about **another** user unless the environment is elevated or the
  acting user is internal:

  > You can ony call user.has_group() with your current user.

  (The message is reproduced exactly, including its spelling.)
- *Satisfies a group specification* — a specification is a comma-separated list of external
  references, each optionally preceded by an exclamation mark meaning *not*. The specification `.`
  alone is never satisfied. Evaluation order: if the user belongs to **any** negated group, the
  answer is false; otherwise if the user belongs to **any** positive group, the answer is true;
  otherwise the answer is true exactly when there were no positive groups at all. So `!base.group_portal`
  alone means "anyone who is not an external user", and an empty list means "anyone".

**Derived predicates.**

| Predicate | Definition |
|---|---|
| internal | belongs to *Role / User* |
| external (portal) | belongs to *Role / Portal* |
| anonymous (public) | belongs to *Role / Public* |
| system | belongs to *Role / Administrator* |
| administrator | is the system account **or** belongs to *Access Rights* |
| superuser | is the account with identifier 1 |

Each is evaluated with elevation so that a user can ask about themselves without needing read
permission on the user entity.

---

## 5. Privilege elevation semantics

### 5.1 What elevation is

Elevation is a **flag on the environment**, not a change of identity. An elevated environment has
the same acting user, the same language, the same time zone and the same active companies; what
changes is that:

1. the entity-level check answers *permitted* immediately (section 2.3 step 1);
2. the record-level check is skipped, because the applicable-rule set is empty (section 3.2 step 2);
3. the field-level check answers *permitted* immediately (section 6);
4. the environment's active-company validation is skipped (section 8.2).

Everything else still applies: required fields, uniqueness constraints, check constraints, model
constraints, computed-field recomputation, change tracking, and the company-consistency check of
section 7. **Elevation is not a licence to write invalid data.**

### 5.2 Obtaining and leaving elevation

- *Elevate* returns the same record set attached to an elevated environment. Asking to elevate an
  already-elevated environment returns the identical object. Asking to de-elevate returns a
  non-elevated environment.
- The returned record set keeps the same pre-fetch grouping, so elevation is cheap.
- An environment whose acting user is the account with identifier 1 is **always** elevated: it is
  set at environment construction time and cannot be switched off by de-elevating.

### 5.3 Acting as another user

*With user* returns the same record set attached to an environment whose acting user is the given
user and which is **not** elevated — unless the given user is the account with identifier 1, in
which case the environment is elevated by the rule above. Passing an empty value returns the record
set unchanged.

The distinction matters: elevation keeps the identity and drops the checks; acting as another user
changes the identity and keeps the checks.

### 5.4 The protection against elevated relation commands

Some entities are too sensitive to be manipulated through a relation field from an elevated or
borrowed environment: a write of the form "add this group to that user" issued through a relation
from elevated code would silently grant permissions.

The rule: when a relation field's target entity **forbids elevated commands**, the commands are
applied in an environment that is **de-elevated** and whose acting user is reset to the
transaction's original user. The following entities forbid elevated commands:

- User, Access Group, Access Right, Record Rule, Default Value, System Parameter, External
  Identifier, Application Key.

### 5.5 Where elevation is legitimately used in this domain

| Situation | Why |
|---|---|
| Reading the group closure, the *Share User* flag, the counts of groups, rights and rules | A user must be able to learn their own permissions without having permission on the permission entities. |
| Reading or writing one's own account, limited to the self-service fields | Section 1.7 of [entities.md](entities.md). |
| Evaluating a record rule's filter | Section 3.3 step 8. |
| Filtering records during the record-level check | Section 2.2 step 3. |
| The sign-in path | The acting identity is not yet established. |
| Creating a settings row, applying a settings projection | The settings screen is administered by users who are not necessarily administrators of every affected entity. |
| Recycling candidates: reading the display name, archiving or deleting the pointed-at record | The recycling rule may target entities the operator cannot read. |
| The privacy search: reading found records, archiving or deleting them | Same reason. |
| Reading the system parameters from within a field computation | Parameters are consulted before permissions are known. |

### 5.6 Active-company narrowing

*With company* returns the same record set with the active-company list rearranged so that the given
company is **first**:

1. An empty value returns the record set unchanged.
2. If the list already begins with that company, return the record set unchanged.
3. Otherwise copy the list, remove the company from it if present, insert it at the front, and
   attach the new list to the context.

The effect is that the default company of records created through the returned set becomes that
company, while every previously active company remains active. The operation performs **no
permission check**: reading the environment's company afterwards may raise (section 8.2) unless the
environment is elevated.

---

## 6. Field-level access

### 6.1 The test

A field is accessible for an operation (read or write) when:

1. the field declares no group restriction; **or**
2. the environment is elevated; **or**
3. the field's restriction is not the *never accessible* marker, and the acting user satisfies the
   field's group specification (section 4).

The *never accessible* marker means no group can pass: the only access is through elevation.

The User entity extends the test: a field is additionally readable when the record being read is the
acting user's own record **and** the field is in the readable-by-oneself list.

### 6.2 The refusal

1. Log, at informational level, that access was denied by access rights for a field, naming the
   operation, the acting user identifier, the entity and the field name.
2. Compose:

   ```
   You do not have enough rights to access the field "<field name>" on <entity label> (<entity
   transport name>). Please contact your system administrator.

   Operation: <read or write>
   ```

3. If the acting user belongs to *Technical Features* (by the internal test, which ignores developer
   mode), append two further lines:

   ```
   User: <the acting user identifier>
   Groups: <the group explanation>
   ```

   where the group explanation is:
   - `always forbidden` when the field carries the *never accessible* marker;
   - `custom field access rules` when the field declares no groups (which can only happen when the
     test was overridden);
   - `allowed for groups '<display name>', '<display name>', …` otherwise, listing the groups of the
     specification resolved from their external references, ordered by identifier, each quoted.

### 6.3 Bulk field access

Asking for the accessible fields of an entity for an operation:

- under elevation, every field is accessible;
- with no list given, the answer is every field that passes the test;
- with a list given, each named field is tested and the first failure raises. Unknown or virtual
  field names are skipped silently, because nothing will be read from or written to them.

---

## 7. Company consistency

### 7.1 When it runs

Entities may declare that they want automatic company consistency checking. For those, the check
runs on create and on write. It may also be invoked explicitly with a list of field names.

### 7.2 The procedure

**Inputs.** A set of records; optionally a list of field names.

1. If no list is given, **or** the list mentions either company field, the list becomes *every*
   field of the entity. (Changing the company of a record can invalidate any relation, so the whole
   record must be re-checked.)
2. Partition the named fields into:
   - **plain company-checked relations** — relational fields marked as company-checked that are not
     company-dependent;
   - **company-dependent company-checked relations** — the same but company-dependent.
3. If both partitions are empty, stop.
4. For each record:
   1. **Plain relations.** Determine the record's own companies:
      - if the entity is the Company entity, the record itself;
      - else if the record has a single-company field, its value;
      - else if the record has a multi-company field, its value;
      - else log a warning naming the entity and the fields, and skip this record:

        > Skipping a company check for model *the entity name*. Its fields *the field names* are set
        > as company-dependent, but the model doesn't have a `company_id` or `company_ids` field!

      Then, for each plain relation, read the linked records with elevation; if there are any, build
      the target entity's **company-agreement filter** for the record's companies and check that
      every linked record satisfies it, with the archive filter disabled. Any that does not is an
      inconsistency, recorded as (record, field name, linked records).
   2. **Company-dependent relations.** The same, but the companies compared against are the
      **currently active company**, not the record's own — because a company-dependent value belongs
      to the company it was stored for.
5. If there are no inconsistencies, stop.

### 7.3 The company-agreement filter

The default filter for a target entity, given a set of companies:

- when the set is empty: `the target's company is empty`;
- otherwise: `the target's company is one of <the identifiers> or is empty`.

Two entities override it. The Company entity itself uses `identifier is one of <the identifiers>`
with no empty case. The User entity uses `the user's permitted companies include one of <the
identifiers>`, and the always-true filter when the set is empty — this is what allows a user whose
main company is A but who is also permitted in B to be assigned to a record of company B.

### 7.4 The refusal

The message begins with:

> Uh-oh! You've got some company inconsistencies here:

(the apostrophe is the typographic one). Then one line per inconsistency, at most five, choosing
among three forms:

| Situation | Line |
|---|---|
| The record is itself a Company | `- Record is company "<company display names>" while "<field label>" (<field storage name>: <linked display names>) belongs to another company.` |
| The record is its own link and the field is the company field | `- Only a root company can be set on "<record display name>". Currently set to "<company display name>"` |
| Otherwise | `- "<record display name>" belongs to company "<company display names>" while "<field label>" (<field storage name>: <linked display names>) belongs to another company.` |

Company display names are comma-separated; linked display names are comma-separated and each is
quoted. The message ends with a final line:

> To avoid a mess, no company crossover is allowed!

---

## 8. Company selection rules

### 8.1 The active-company list

Every call carries, in its context, an ordered list of **active company identifiers**. The first
element is the *current* company; the whole list is the set of companies whose records are visible.
The list is supplied by the client and is **validated on every read of the environment's company or
companies**.

### 8.2 Resolving the current company

1. Read the active-company list from the context.
2. If it is non-empty:
   1. If the environment is **not** elevated, take the acting user's permitted, active companies. If
      the requested list contains any identifier not among them, refuse:

      > Access to unauthorized or invalid companies.

   2. Return the company whose identifier is the **first** element.
3. If it is empty, return the acting user's default company.

### 8.3 Resolving the active companies

1. Read the active-company list from the context and the acting user's permitted, active companies.
2. If the list is non-empty: when not elevated, refuse as above if it contains anything not
   permitted; otherwise return the companies of the list, in the list's order.
3. If the list is empty, return **all** of the user's permitted companies — not just the default
   one. This is deliberate: calls made outside a client context (printing a document spanning
   several companies, serving an image referenced from a notification, following a redirect from a
   message) would otherwise fail for no good reason, and returning all permitted companies is safe
   because the user does have access to them.

### 8.4 The permitted-company list of a user

The permitted companies of a user are computed by searching for **active** companies that name the
user among their accepted users. The result is cached per user identifier, and the cache is cleared
when a company's active flag or sequence changes, when companies are created or deleted, and when a
user's companies are written.

Note that the search is used rather than reading the relation, so that the archive filter is applied
consistently: an archived company is never permitted, even if the relation still names it.

### 8.5 Switching the active company

Switching is a client-side act: the client sends a different active-company list on the next call.
The rules that make it safe are exactly sections 8.2 and 8.3. The list handed to the client at
sign-in contains:

- the **current company** — the user's default company;
- the **allowed companies** — every permitted company, each with its identifier, name, sequence, the
  identifiers of its children that are themselves in the visible hierarchy, its parent identifier
  and its currency;
- the **disallowed ancestor companies** — every ancestor of a permitted company that is not itself
  permitted, with the same fields except the currency. These exist so the client can draw the tree
  correctly: a user permitted in a branch but not in its parent still needs to see the parent as a
  grouping node, greyed out and unselectable.

### 8.6 Worked example — a user with two companies switching the active one

Company 1 *Alpha* (root, currency United States dollar) and company 2 *Beta* (root, currency euro).
User C is permitted in both; default company is *Alpha*.

1. C signs in. The session carries no active-company list yet, so the current company resolves to
   *Alpha* and the active companies resolve to {*Alpha*, *Beta*}. The client receives
   *current company* = 1 and *allowed companies* = {1, 2}.
2. The client sets the active list to `[1]`. Every subsequent call validates `{1} ⊆ {1, 2}` — fine.
   The current company is *Alpha*; the multi-company global rule becomes `company is in [1] or
   company is empty`; records of *Beta* disappear from every list.
3. C creates a Sales Order. Its company defaults to the current company, *Alpha*.
4. The client sets the active list to `[2, 1]`. Validation passes. The current company becomes
   *Beta*; the global rule becomes `company is in [2, 1] or company is empty`; records of both
   companies are visible, and new records default to *Beta*.
5. C's administrator removes *Beta* from C's permitted companies. The cached permitted list is
   cleared by the write. C's next call still carries `[2, 1]`, so validation now fails and the call
   is refused with *Access to unauthorized or invalid companies.* The client recovers by asking for
   a fresh session description, which no longer offers *Beta*.
6. If instead the administrator had set C's default company to *Beta* while removing *Beta* from
   the permitted list, the write itself would have been refused:
   *Company Beta is not in the allowed companies for user C (Alpha).*

### 8.7 Accessible branches

See [entities.md](entities.md) section 12.7. The rule exists so that an operation offered "for
company X" also covers X's branches, but only those branches the user is actually active in.

---

## 9. Authentication rules

### 9.1 The credential contract

A credential is a structured value carrying at least a **type**. The types defined across the domain
are:

| Type | Extra keys | Provided by |
|---|---|---|
| `password` | `login`, `password` | foundation |
| `totp` | `token` | second factor |
| `totp_mail` | `token` | mailed second factor |
| `webauthn` | `webauthn_response` | passkeys |
| `oauth_token` | `token` | delegated sign-in |

A successful check returns a result carrying:

| Key | Meaning |
|---|---|
| the account identifier | Which account was authenticated. |
| the method | One of `password`, `apikey`, `totp`, `totp_mail`, `passkey`, `oauth`, `ldap`, `impersonation`. |
| the second-factor policy | `default` — consult the second-factor configuration; `skip` — do not ask for a second factor; `enforce` — always ask. |

Credentials are **untrusted input** at every step.

### 9.2 Checking a password credential

**Preconditions.** A credential; an environment description carrying at least whether the connection
is interactive.

1. If the credential type is not `password`, or the password is empty, refuse immediately.
2. Read the interactivity flag; assume interactive when it is absent, and log a warning that the key
   was missing.
3. If the connection is interactive, **or** the account does not require application keys:
   1. Read the stored hash directly from the account's row, treating a null as the empty string.
   2. Verify the supplied password against the hash, obtaining both a verdict and, when the stored
      hash uses an out-of-date scheme or work factor, a **replacement hash**.
   3. If a replacement hash was produced, write it to the row. If this happened inside a request and
      the account is the acting account, flush, clear the registry caches, recompute the session
      token and store it in the session — otherwise the user would be signed out by their own
      successful sign-in.
   4. If the verdict is positive, return: the account, the method `password`, the policy `default`.
4. If the connection is **not** interactive:
   1. Try the supplied value as an application key in the global purpose. A match returns: the
      account, the method `apikey`, the policy `default`.
   2. If the account requires application keys, log that a password was attempted on a
      key-only connection.
5. Refuse.

An account **requires application keys** when it has the second factor enabled. This is the rule
that makes the second factor meaningful for programmatic connections: once it is on, a password is
no longer accepted there at all.

### 9.3 Signing in

**Inputs.** A credential carrying a login; an environment description.

1. Determine the remote network address, or the text `n/a` when there is no request.
2. Enter the **cooldown guard** keyed on the login (section 9.5).
3. Search, with elevation, for the single account matching the login, using the entity's default
   ordering. A login match is an exact equality on the login field.
4. If there is none, refuse.
5. Re-attach the account to an environment whose acting user is that account, elevated.
6. Check the credentials (section 9.2 and every extension).
7. If the request carries a time-zone cookie holding a known zone, and the account has no time zone
   **or** has never signed in, store that zone on the account.
8. Record a sign-in: insert a sign-in log row with elevation and no values.
9. On any refusal, log at informational level *Login failed for login:<login> from <address>* and
   re-raise. On success, log *Login successful for login:<login> from <address>*.
10. Return the authentication result.

### 9.4 Establishing the session

1. Compose the environment description: interactive; the base address of the request with any
   trailing solidus removed; the host header; the remote address.
2. Sign in (section 9.3) with a fresh, non-elevated, user-less environment.
3. Clear the session's account identifier; store the supplied login under the pending-login key and
   the authenticated account identifier under the pending-account key.
4. If the result's second-factor policy is `skip`, **or** the account has no second-factor address,
   **finalise** immediately (step 5). Otherwise the session stays *pending* and the client is sent to
   the second-factor address.
5. **Finalising**: pop the pending login and pending account; take the account's preference context;
   mark the session for rotation; and write into the session: the database name, the login, the
   account identifier, the preference context, and the freshly computed **session token**.
6. If the sign-in happened in the current request and on the current database, re-attach the request
   environment to the now-known account and set the request language from it.
7. After a successful interactive sign-in, when the account holds *Role / Administrator* and the
   environment description carries a base address, and the parameter `web.base.url.freeze` is not
   set, the parameter `web.base.url` is updated to that base address. Any failure here is logged and
   ignored.

### 9.5 The sign-in cooldown

The guard wraps any authentication attempt. It is keyed on the **remote network address**, not on
the login, and the counters live in the process, not in the database.

1. If there is no request, the guard does nothing.
2. Read the failure map from the registry, creating it if absent. Each entry maps a source address
   to a pair (failure count, moment of the last failure); a missing entry reads as (0, the earliest
   representable moment).
3. Read the entry for the request's remote address.
4. Decide whether the source is *on cooldown* (section 9.6). If it is:
   1. Log a warning naming the source, the login being attempted, the database, the failure count
      and the moment of the last failure, and explaining that the threshold and the duration are
      configurable and that setting the threshold to zero disables the feature.
   2. If the address is in a private range, log a second warning that a private address is being
      rate-limited and that this may indicate a mis-configured reverse proxy.
   3. Refuse with:

      > Too many login failures, please wait a bit before trying again.

5. Otherwise run the wrapped attempt.
6. If the attempt refuses, increment the failure count for the source and set the moment of the last
   failure to now, then re-raise.
7. If the attempt succeeds, **remove** the source's entry entirely.

The counters are deliberately not shared between worker processes and are not thread-safe: the
feature exists to blunt large-scale password guessing, not to be an exact quota.

### 9.6 The cooldown predicate

**Inputs.** A failure count and the moment of the last failure.

1. Read the threshold from the parameter `base.login_cooldown_after`, defaulting to **5** when the
   parameter is absent. (A normally initialised database has this parameter set to **10**.)
2. If the threshold is zero, the source is never on cooldown.
3. Read the duration in seconds from `base.login_cooldown_duration`, defaulting to **60**.
4. The source is on cooldown exactly when the failure count is **greater than or equal to** the
   threshold **and** the elapsed time since the last failure is **strictly less than** the duration.

So the cooldown is *linear*, not exponential: once the threshold is reached, every further failure
restarts a fixed window.

### 9.7 Verifying an identifier and password pair

A separate, cached entry point verifies a pair without establishing a session:

1. An empty password is refused outright.
2. The cooldown guard is entered, keyed on the account identifier.
3. An inactive account is refused.
4. The credentials are checked with the connection marked **non-interactive**.

The result is cached by (account identifier, password), which is why the entry point exists at all:
it is called on every programmatic call of a connection that authenticates on each request.

### 9.8 Changing one's own password

1. An empty old password is refused outright.
2. The old password is checked interactively against the acting account.
3. The internal password change is performed on the acting account.

The internal change:

1. Trims the new password. An empty result is refused:

   > Setting empty passwords is not allowed for security reasons!

2. Logs the change at informational level, naming the target login and identifier, the acting login
   and identifier, and the remote address.
3. Writes the password, which hashes it (and, when the strength policy package is installed, first
   validates it — section 10).

The second-factor package extends the entry point: every trusted browser of the acting account is
revoked **before** the change.

### 9.9 The session token

The session token binds a session to the state of the account. It is recomputed on every request and
compared, in constant time, with the value stored in the session.

**Computing** — see [calculations.md](calculations.md) section 6 for the exact construction.

**Checking a session.**

1. Delete expired sessions.
2. If the session carries a deletion moment that has passed, the session is invalid.
3. Compute the expected token for the session's account and session identifier.
4. If the expected token is falsy (the account row does not exist, or more than one row came back —
   in which case the registry caches are also cleared), the session is invalid.
5. If the expected token equals the stored token, in constant time, the session is valid; update the
   device trace.
6. Otherwise compute the **older** token construction and compare that. If it matches, replace the
   stored token with the current construction and treat the session as valid; update the device
   trace.
7. Otherwise the session is invalid.

An invalid session causes a sign-out that keeps the database selection, and the request continues as
an unauthenticated one.

### 9.10 Authentication modes of a route

Every route declares how it authenticates:

| Mode | Behaviour |
|---|---|
| `none` | The environment is attached to no user at all. Used for routes that must work before a database is chosen. |
| `public` | If no account is established, the environment is attached to the **anonymous account**. |
| `user` | If the established account is absent or is one of the anonymous accounts, the session is treated as expired. |
| `bearer` | Application-key authentication; see below. |

The `bearer` mode:

1. Extract the token from an `Authorization` header of the form `bearer <token>`, case-insensitively.
2. If a token is present:
   1. Verify it as an application key in the global purpose. No match:
      `Invalid apikey`, answered with a bearer authentication challenge.
   2. If the request already has an established account and it differs from the key's account:
      `Session user does not match the used apikey.`
   3. Attach the environment to the key's account and mark the session **not savable** (the call is
      stateless).
3. If no token is present and no account is established:
   `User not authenticated, use an API Key with a Bearer Authorization header.`, with a bearer
   challenge.
4. If no token is present, an account **is** established, but the request does not carry the
   browser-navigation markers (destination *document*, mode *navigate*, site *none* or
   *same-origin*, user-activated), then:
   `Missing "Authorization" or Sec-headers for interactive usage.`, with a bearer challenge.
   This is the cross-site request protection for key-authenticated routes.
5. Finally the `user` rule is applied.

**Before** the mode is applied, every request with an established account re-checks the session
token (section 9.9); a failure signs the session out, keeping the database, and continues
unauthenticated. Any unexpected error during authentication is logged and converted into a generic
refusal, so that authentication never leaks an internal error.

### 9.11 Impersonation

A route lets a holder of *Role / Administrator* replace their session account with the account of
identifier 1. The session token cache is cleared because the account changed, and a new session token
is computed. This is the only path by which a human account becomes permanently elevated, and it is
reserved to the *Role / Administrator* group.

---

## 10. Password policy rules

### 10.1 The published policy

A named operation returns the current policy so the client can show a strength indicator:

| Key | Value |
|---|---|
| minimum length | the parameter `auth_password_policy.minlength`, defaulting to 0 |

### 10.2 Enforcement

Every write of a password runs the policy check **before** hashing.

1. Read the minimum length from the parameter, defaulting to 0.
2. For each password being written, skipping empty ones: if its length is strictly less than the
   minimum, record the failure

   > Your password must contain at least *the minimum* characters and only has *the actual count*.

3. If there is at least one failure, refuse with all the failure texts joined by a blank line and a
   space.

A minimum length of 0 disables the policy. The value is clamped to zero or more when it is edited on
the settings screen.

### 10.3 Where the policy is surfaced

- The customer-facing security page exposes the minimum length so the browser can show the
  indicator.
- The sign-up and password-reset pages expose it the same way.
- The enforcement itself is in the write path, so it applies equally to a self-service change, an
  administrator-driven change, a sign-up, a password reset and a direct write.

### 10.4 Worked example — a password refused by the policy

The parameter is set to 12.

1. A user opens the self-service change dialogue, supplies the correct current password, and types
   `summer2026` (10 characters) twice.
2. The confirmation matches, so the wizard's own constraint passes.
3. The identity re-check passes (the current password was just confirmed, or the confirmation is
   within ten minutes).
4. The internal change trims the value — still 10 characters — and writes it.
5. The write runs the policy first: 10 < 12, so one failure is recorded and the write is refused
   with

   > Your password must contain at least 12 characters and only has 10.

6. Nothing is written; the stored hash is unchanged; the session is unaffected.

---

## 11. Second-factor rules

### 11.1 When a second factor is demanded

After a successful first factor:

1. If the authentication result's policy is `skip`, no second factor is demanded. A passkey
   sign-in always produces `skip`.
2. Otherwise the account is asked for its second-factor **address**. The base implementation has
   none. The second-factor package returns the second-factor page's path when the account has a
   secret. The mailed-code package additionally returns it when the account is internal and the
   mailed fallback is enabled.
3. If there is no address, the session is finalised immediately.
4. Otherwise the session stays pending and the client is sent to that address.

### 11.2 The second-factor page

**On a first display (a retrieval request):**

1. If a session account is already established, redirect to the post-sign-in destination.
2. If there is no pending account, redirect to the sign-in page.
3. Read the trusted-browser cookie. If present, verify it as a scoped key in the `browser` purpose
   for the pending account. On a match: finalise the session, re-attach the request, and redirect to
   the destination — no code is asked for.
4. Otherwise render the code form.

**On a submission carrying a code:**

1. Enter the cooldown guard keyed on the pending account identifier.
2. Build a credential of the account's second-factor type, with the code stripped of white space and
   parsed as a whole number.
3. Check the credentials interactively.
4. A refusal is rendered as the form's error. A value that will not parse as a number is rendered as
   *Invalid authentication code format.*
5. On success: finalise the session, re-attach the request, and redirect.
6. If "remember this browser" was ticked: compose a label (*browser* on *platform*, each
   capitalised, plus the city and country in parentheses when the address resolves to a city),
   generate a scoped key in the `browser` purpose with elevation whose expiry is now plus the
   trusted-browser age, and set it as a cookie with that maximum age, marked inaccessible to
   scripts, with the same-site policy *lax*.
7. The session is touched so it is saved even when nothing else changed.

### 11.3 Worked example — a sign-in with a wrong then a right code

Account D has the second factor enabled; the trusted-browser cookie is absent; the parameters are at
their defaults.

1. D posts the login and password. The password check succeeds and returns policy `default`. The
   account has a second-factor address, so the session stores the pending login and pending account
   and D is sent to the second-factor page.
2. D types `123456`, which is not the current code.
   1. The cooldown guard is entered for D's account identifier; there have been no failures, so it
      proceeds.
   2. The code-checking rate limit is consumed: a first log row is written for D with the type
      *code check*. The count before writing was 0, which is below 5, so the check proceeds.
   3. The code is matched against the secret over the window of section 7 of
      [calculations.md](calculations.md). No counter produces `123456`.
   4. A refusal is raised: *Verification failed, please double-check the 6-digit code*.
   5. The cooldown guard records one failure for D's account identifier.
   6. The page re-renders with that text as the error. The session is still pending; D is **not**
      signed in.
3. D types the code currently shown by the authenticator, `704318`.
   1. The cooldown guard proceeds: one failure is below the threshold.
   2. The rate limit is consumed again: the count before writing was 1, below 5, so it proceeds; a
      second log row is written.
   3. The match succeeds at counter *k*. The stored last counter is either absent or some earlier
      value, so the replay test passes.
   4. The last counter is set to *k*; **all** of D's code-check rate-limit rows are purged.
   5. The result is (D, method `totp`, policy `default`).
   6. The cooldown guard removes D's failure entry.
   7. The session is finalised: the pending keys are popped, the session token is computed and
      stored, the session is marked for rotation.
4. If D had submitted the same code `704318` a second time within its validity window, the match
   would again produce counter *k*, but *k* is now less than or equal to the stored last counter, so
   the refusal would be *Verification failed, please use the latest 6-digit code*.

### 11.4 Worked example — a cooldown after repeated failures

Continuing from the previous example, but D keeps typing wrong codes from the same network address.
Assume the parameters are at their initialised values: threshold 10, duration 60 seconds. Assume
also that the code-check rate limit (5 per 3600 seconds) applies per **account**, whereas the
cooldown applies per **source address**.

| Attempt | Rate-limit rows before | Cooldown failures before | Outcome |
|---|---|---|---|
| 1 | 0 | 0 | Row written; code wrong; refusal *Verification failed, please double-check the 6-digit code*; failures → 1 |
| 2 | 1 | 1 | Row written; code wrong; failures → 2 |
| 3 | 2 | 2 | Row written; code wrong; failures → 3 |
| 4 | 3 | 3 | Row written; code wrong; failures → 4 |
| 5 | 4 | 4 | Row written; code wrong; failures → 5 |
| 6 | 5 | 5 | **Rate limit reached**: refusal *You reached the limit of code verifications for your account, please try again later.* No row is written. The cooldown guard counts this refusal too; failures → 6 |
| 7…10 | 5 | 6…9 | Same rate-limit refusal each time; failures → 10 |
| 11 | 5 | 10 | **Cooldown reached**: the guard refuses before anything else with *Too many login failures, please wait a bit before trying again.* |

From attempt 11, every attempt from that address is refused for 60 seconds after the most recent
failure, and each refused attempt restarts the 60 seconds. After 60 quiet seconds, attempts resume —
but the rate-limit rows for D's account survive for a full 3600 seconds from the moment each was
written, so code checking for D stays refused until the oldest rows age out. A successful sign-in
from that address would clear the cooldown entry; a successful code check would purge the
rate-limit rows.

### 11.5 Enabling and disabling

Enabling:

1. The action is protected by the identity re-check.
2. It is refused for anyone but oneself: *Two-factor authentication can only be enabled for
   yourself*
3. It is refused when already enabled: *Two-factor authentication already enabled*
4. A secret of 160 bits is drawn from the operating system's cryptographic source, rendered in
   base 32 and grouped in blocks of four characters.
5. The setup wizard is opened; completing it performs the enrolment attempt of section 20.4 of
   [entities.md](entities.md).

Disabling is protected by the identity re-check, permitted to oneself, to an administrator or to
elevated code, and revokes every trusted browser.

---

## 12. External access through a signed token

### 12.1 The rule

A customer-facing document page may be reached by someone who is not permitted to read the document,
provided they present the document's token.

**Procedure.** Given an entity name, a record identifier and an optional token:

1. Browse the record in the acting environment; separately, browse it as the account of identifier 1
   and test existence. If it does not exist:

   > This document does not exist.

2. Try the ordinary read check as the acting user.
3. If the read check refuses:
   1. If no token was supplied, re-raise the refusal.
   2. If the document has no token of its own, re-raise the refusal.
   3. Compare the supplied token with the document's token **in constant time**. If they differ,
      re-raise the refusal.
4. Return the record **as the account of identifier 1** — that is, fully elevated.

Three properties are essential and must be preserved in any re-implementation:

- The existence test happens **before** the permission test, and produces a different error, so a
  deleted document does not look like a permission problem.
- The comparison is constant-time, so the token cannot be discovered by timing.
- What is returned is elevated. The customer-facing page therefore reads the document without any
  further permission check, and every field it shows must be chosen deliberately by the page.

### 12.2 Token life cycle

- A token does not exist until it is needed. The first time a share address is built, a version-4
  random universally unique identifier is generated and written **with elevation** (a plain write
  would leave the cached value stale and the freshly written token unreadable).
- A token is **not copied** when the document is duplicated: the copy has no token until one is
  asked for.
- There is no expiry and no revocation list. Withdrawing access means clearing or replacing the
  token.
- Before a token is attached to an address, the acting user's own **read** access to the document is
  checked — one cannot mint a share address for a document one cannot read.

### 12.3 The recipient signature

An address may additionally carry a recipient contact identifier and a signature over it (see
[calculations.md](calculations.md) section 5.3). This does not grant access — the token does that —
but it lets the document's discussion thread attribute a message to that contact instead of to an
anonymous visitor.

### 12.4 Worked example — an external party reading through a signed token

A sales order SO0042 belongs to company *Alpha*; the customer contact is *Widgets Limited*, which
has no account.

1. A salesperson opens the share dialogue on SO0042. The dialogue computes the link: because the
   document adopts the portal mixin, the absolute address is the base address plus the generic
   mail-view path plus a query carrying the entity name, the record identifier and the token. To
   obtain the token, the salesperson's **read** access to SO0042 is checked (it passes), and the
   token is generated and written with elevation.
2. Open sign-up is disabled, so every recipient receives the plain share message. The message is
   posted on SO0042 as an internal note with the subject *Invitation to access SO0042*, addressed to
   *Widgets Limited*, and rendered in that contact's language. The link embedded for that recipient
   also carries the recipient identifier and its signature.
3. The recipient opens the link. The route has authentication mode `public`, so the environment is
   attached to the anonymous account.
4. The page calls the document check with the entity name, the identifier and the token.
   1. The record exists.
   2. The read check as the anonymous account refuses: no access-right row grants the anonymous group
      read on sales orders.
   3. A token was supplied, the document has a token, and the two are equal in constant time.
   4. The elevated record is returned.
5. The page renders the order. The recipient identifier and its signature are also verified, so the
   discussion thread shows the recipient as the author of anything they post.
6. If the recipient forwards the link and a second person opens it, that person sees the same
   document: the token is the credential, and it is not bound to a person. The recipient signature,
   however, does not transfer — it is bound to the contact identifier it was signed over.

---

## 13. Sign-up and password-reset rules

### 13.1 Invitation scope

The parameter `auth_signup.invitation_scope` decides who may register:

| Value | Meaning |
|---|---|
| `b2b` (default) | Only invited contacts. A registration without a token is refused. |
| `b2c` | Anyone may register. |

### 13.2 Preparing an invitation

Preparing an invitation on a contact sets its sign-up token type to `signup` (an invitation to
create an account) or `reset` (an invitation to set a new password). **No token is stored**: the
token is a signed payload computed on demand, so the database never holds a usable credential.

Cancelling an invitation clears the type, which also invalidates any outstanding token, because the
type is part of the signed payload.

### 13.3 Registering

**Inputs.** A map of values; an optional token.

**With a token:**

1. Resolve the contact from the token (section 5.2 of [calculations.md](calculations.md)). An
   unresolvable or expired token raises

   > Signup token '*the token*' is not valid or expired

2. Clear the contact's sign-up token type, so the token cannot be used twice.
3. If the contact already has a country, a postal code or a city, drop the proposed city and country
   — geolocated guesses must not overwrite real data. If the contact already has a language, drop
   the proposed language.
4. **If the contact already has an account**: drop the proposed login and name, write the remaining
   values to that account, and — when the account had never signed in and is internal — notify the
   inviter over the live channel. Return the account's login and the proposed password.
5. **If the contact has no account**: compose the values with the contact's name, the contact
   identifier, and the electronic mail address taken from the proposal's address or, failing that,
   its login. If the contact has a company, set the account's default and permitted companies to it.
   Then create the account from the template (below).

**Without a token:** set the electronic mail address from the proposal's address or login, then
create the account from the template.

### 13.4 Creating an account from the template

1. If the proposal has no contact, and the invitation scope is not `b2c`:

   > Signup is not allowed for uninvited users

2. If the proposal carries an electronic mail address and **any** account (archived included) has
   that address:

   > Another user is already registered using this email address.

3. Read the template account identifier from the parameter `base.template_portal_user_id`. If the
   account does not exist:

   > Signup: invalid template user

4. If the proposal has no login:

   > Signup: no login given for new user

5. If the proposal has neither a contact nor a name:

   > Signup: no name or partner given for new user

6. Mark the values active and **copy the template account** with them, inside a nested savepoint and
   with the "no password mail" marker. Any failure of the copy — most often a login collision — is
   converted into a sign-up error carrying the underlying message.

Copying the template rather than creating an account is what makes the result an external user: the
template holds exactly the group *Role / Portal*, and the copy inherits it instead of going through
the internal-user defaults.

### 13.5 Resetting a password

**By login.** Search for accounts whose login matches; if none, search by electronic mail address.
No account: *No account found for this login*. More than one: *Multiple accounts found for this
login*. Otherwise proceed.

**The reset action.** The sign-up type is `signup` when the context marks the flow as account
creation, and `reset` otherwise. Then:

1. Do nothing at all when the context marks a package installation or a file import.
2. Refuse when any selected account is archived:
   *You cannot perform this action on an archived user.*
3. Prepare the invitation on every selected account's contact with the chosen type.
4. Choose the message template:
   - in creation mode, the internal-account template for internal accounts and the external-account
     template for the others;
   - otherwise, no template — the reset message is rendered directly from its view.
5. For each account: refuse when it has no electronic mail address
   (*Cannot send email: user the user name has no email address.*); then, inside a nested savepoint,
   send the message forcibly and synchronously, addressed to that account's address, with copies
   suppressed, marked for automatic deletion, typed as a user notification and not scheduled.
6. Log the send and return a non-blocking notice: *A signup link was sent by email* in creation
   mode, *A reset password link was sent by email* otherwise.
7. Delivery failures are converted into readable errors:
   - a refused connection: *Could not contact the mail server, please check your outgoing email
     server configuration*;
   - anything else: *There was an error when trying to deliver your Email, please check your
     configuration*.

### 13.6 Worked example — a sign-up through an invitation

An administrator creates an internal account for *Jean Martin*, address `jean.martin@example.test`.

1. The account is created. Because the creation is not marked "no password mail" and the account has
   an address, a sign-up invitation is prepared and sent: the contact's sign-up type becomes
   `signup`, and a signed token is computed with a validity of the parameter
   `auth_signup.signup.validity.hours` (default 144 hours, i.e. six days).
2. The token's payload is the list (contact identifier, the identifiers of the contact's accounts,
   the most recent sign-in moment of those accounts — here absent — and the sign-up type `signup`).
3. Jean opens the link. The sign-up page resolves the token: the payload is verified, then the
   contact is re-read and the payload's three trailing elements are compared with the contact's
   current state. They match, so the contact is returned, and the page is pre-filled with the
   contact's name and, since the contact has an account, its login.
4. Jean chooses a password and submits. Registration runs with the token:
   1. The contact is resolved again and the sign-up type is cleared — the token is now dead.
   2. The contact has an account, so the login and name in the proposal are dropped and only the
      password is written.
   3. The password write runs the strength policy.
   4. Because the account had never signed in and is internal, the inviter is notified over the live
      channel that Jean has connected.
5. Jean is signed in with the new password. A sign-in log row is created, so the account's status
   becomes *Confirmed* and the token could not be replayed even if the type had not been cleared,
   because the most recent sign-in moment is part of the payload and has now changed.
6. Had Jean waited seven days, the signature's expiry would have elapsed and the resolution would
   have failed with *Signup token '…' is not valid or expired*.

### 13.7 Reminding about unregistered accounts

A scheduled job mails each inviter about the accounts they created that have still never signed in.
Its parameters are: a delay of **5 days** and a batch of **100**.

1. If the reminder template is missing, log a warning and deactivate the job.
2. Compute the window: the day that is *delay* days before today, and the following day.
3. Find the internal accounts (share flag false) whose creator has an address, whose creation moment
   falls in that window, and which have no sign-in log entry. Group them by creator.
4. For each creator, send the reminder once, carrying the list of *name (login)* strings, with the
   light notification layout, queued rather than forced.
5. Progress is committed per creator; no progress marker is kept, because there is no way to know to
   whom a message has already been sent.

### 13.8 Bulk invitation

A named operation takes a list of electronic mail addresses, finds the accounts already in the
*Invited* status whose login or address is among them, creates accounts for the remaining addresses,
and re-sends the invitation to the already-invited ones in creation mode.

---

## 14. Session-lifetime rules

### 14.1 The two limits

Each group may carry two limits, both in minutes:

| Limit | Compared against | Consequence when exceeded |
|---|---|---|
| Session timeout | the moment the session was created | full sign-out |
| Inactivity timeout | the moment stored as *next identity check* | re-authentication dialogue |

Each limit may additionally demand a **second factor** at re-authentication.

### 14.2 The per-user summary

See [calculations.md](calculations.md) section 10 for the arithmetic. In outline: for each of the
two limits, take every group of the user's closure that sets it; find the smallest value among those
that demand a second factor and the smallest among those that do not; keep the second-factor one if
any, and keep the non-second-factor one only when it is strictly smaller; express both in seconds
and sort ascending.

### 14.3 Deciding whether to re-authenticate

For each limit in turn — session timeout first, then inactivity timeout — and, within a limit, for
each (duration, second-factor flag) pair in **descending** duration order:

1. Compute the threshold as the current moment minus the duration.
2. Read the session timestamp: the session creation moment for the session timeout (default 0), the
   *next identity check* moment for the inactivity timeout (default absent).
3. Subtract the *first* duration of the inactivity list from the timestamp before comparing, but only
   for the inactivity limit. (Only the shortest inactivity limit ever writes the *next identity
   check* moment, so a longer limit must offset its comparison by the shortest one to measure from
   the same origin. The session creation moment needs no such offset.)
4. If the timestamp is present and the adjusted timestamp is at or below the threshold, the answer
   is: this limit's consequence, plus whether a second factor is demanded, plus — when a second
   factor is demanded and the session records a first factor that was supplied after the threshold —
   which method that first factor used.
5. If no pair triggers, no re-authentication is needed.

### 14.4 The re-authentication exchange

1. With **no** credential supplied, the answer describes what is available: the account identifier,
   the login, and the list of available methods, with the already-used first factor removed when
   there is one.
2. The available methods of an account are, in order: `webauthn` when the account has at least one
   passkey; the account's second-factor type when it has one; and always `password` last.
3. A credential whose type is not among the available methods is refused outright.
4. A second-factor code is stripped of white space and parsed as a whole number.
5. The credential is checked interactively.
6. If a first factor was already recorded and the new method differs from it, the recorded first
   factor is discarded (the exchange is restarting).
7. Otherwise, if the result's policy is not `skip`, more than one method is available, and this
   limit demands a second factor: record (now, the method) as the first factor, remove that method
   from the available list, and answer that a second factor is required together with the remaining
   methods.
8. Otherwise the exchange succeeds: the *next identity check* moment is removed from the session and
   the *last identity check* moment is set to now.

### 14.5 Enforcement on a request

After the ordinary authentication, when the route's mode is `user` and a session account exists:

- a session-timeout trigger raises a session-expired condition, and the client sends the user back
  to the sign-in page;
- an inactivity trigger raises a re-authentication condition, unless the route opts out of identity
  checking. For a page request, the response is a redirect to the re-authentication page carrying
  the original address as the destination; for a programmatic call, the condition is returned so the
  client can show the dialogue in place.

### 14.6 Tracking inactivity

The client reports inactivity over the live channel, in milliseconds; the channel closing (last tab
closed, network lost) reports it forcibly.

1. Convert the reported period to seconds.
2. Read the user's shortest inactivity limit. If there is none, nothing happens.
3. The session is *inactive* when the report was forced, or the reported period is at least the
   limit.
4. If inactive: the *next identity check* moment becomes now plus the limit minus the reported
   period; it is only written when there is no such moment yet or the new one is earlier. The session
   is saved explicitly, because live-channel calls do not save it.
5. If active and a *next identity check* moment exists in the future, it is removed and the session
   is saved explicitly.

---

## 15. Identity re-check rules

The identity re-check (section 14 of [entities.md](entities.md)) guards these operations:

| Operation | Entity |
|---|---|
| Open the change-own-password dialogue | User |
| Apply the change-own-password dialogue | Change Own Password Wizard |
| Open the new-application-key dialogue | User |
| Produce an application key | Application Key Description Wizard |
| Remove an application key | Application Key |
| Revoke all devices | User |
| Revoke one device | Device |
| Enable the second factor (open the wizard) | User |
| Complete the second-factor enrolment | Second-factor Setup Wizard |
| Disable the second factor | User |
| Revoke all trusted browsers | User |
| Create a passkey | User |
| Register a passkey | Passkey Creation Wizard |
| Delete a passkey | Passkey |

Rules:

- The guard requires a request; without one the operation is refused with *This method can only be
  accessed over HTTP*.
- A confirmation newer than **10 minutes** satisfies the guard without a dialogue.
- The suspended call is stored with the context entries that can be serialised; entries that cannot
  (for example values that are record sets) are dropped, which means a guarded operation must not
  depend on such an entry.
- When the stored call is replayed, it is asserted to be one of the guarded operations, so the
  mechanism cannot be used to invoke an arbitrary operation.

---

## 16. Application-key rules

Summarised here; the detail is in [entities.md](entities.md) section 13.

| Rule | Statement |
|---|---|
| Storage | Only the hash and an 8-character index are stored. The clear key is returned once and never again. |
| Creation permission | Only internal users may create keys: *Only internal users can create API keys* |
| Expiry | Mandatory for non-administrators; bounded by the greatest maximum duration among the user's groups, or 1 day when none is set; must be in the future. |
| Verification | The account must be active; the scope must be empty or equal; the expiry must be empty or not past. |
| Removal permission | The acting environment is a system one, or every key belongs to the acting user. |
| Removal protection | Guarded by the identity re-check in the self-service path. |
| Non-interactive use | A key is accepted in place of a password on a non-interactive connection, and is the **only** thing accepted there once the second factor is on. |
| Transport use | A key may be presented as a bearer token on a route whose mode is `bearer` (section 9.10). |
| Programmatic management | Disabled unless the acting environment is a system one or `base.enable_programmatic_api_keys` is true; limited to `base.programmatic_api_keys_limit` unexpired keys (default 10); a scoped key may only mint keys of its own scope. |
| Housekeeping | Expired rows are deleted automatically. |

### 16.1 Worked example — an application key on the transport

An integration must read sales orders without a browser.

1. An internal user opens the new-key dialogue. The identity re-check fires (no confirmation in the
   last ten minutes), the password is confirmed, and the dialogue opens.
2. The user picks the duration *3 Months*. The expiry becomes today plus 90 days. Validation passes
   because the user's groups include *Role / User*, whose maximum duration is 90 days, and 90 ≤ 90.
3. The key is produced: 20 random bytes → 40 hexadecimal characters, say
   `9f2c1b7e4a0d6583cc21ff90ab3d7e5164280cb7`. The row stores the label, the account, no scope, the
   expiry, the hash, and the index `9f2c1b7e`.
4. The clear key is shown once.
5. The integration calls a route whose mode is `bearer`, sending the header
   `Authorization: bearer 9f2c1b7e4a0d6583cc21ff90ab3d7e5164280cb7`.
   1. The token is extracted.
   2. Verification takes the index `9f2c1b7e`, selects rows with that index joined to active
      accounts whose scope is empty or `rpc` and whose expiry is empty or not past, and verifies the
      token against each stored hash. The row matches.
   3. No account is established on the session, so there is nothing to compare against.
   4. The environment is attached to the key's account and the session is marked not savable.
   5. The `user` rule passes because the account is not anonymous.
6. Every subsequent access check in the call is performed **as that account**: the key grants the
   account's permissions, no more.
7. Ninety days later the row's expiry is past; verification no longer returns it, and the automatic
   clean-up eventually deletes it. Presenting the key then yields `Invalid apikey` with a bearer
   challenge.

---

## 17. Portal access rules

| Rule | Statement |
|---|---|
| Who may grant | The grant wizard is reachable from a contact; the acting user must be able to write contacts and create users. |
| Cannot grant twice | *The partner "the contact name" already has the portal access.* |
| Cannot grant without a valid address | *The contact "the contact name" does not have a valid email.* |
| Cannot grant a duplicated address | *The contact "the contact name" has the same email as an existing user* |
| Cannot revoke what is not external | *The partner "the contact name" has no portal access or is internal.* |
| Cannot re-invite what is not external | *You should first grant the portal access to the partner "the contact name".* |
| Never recycle an internal account | A contact whose account is internal — even archived — is reported as internal and cannot be granted external access from the wizard. |
| Revocation archives, it does not downgrade | Revoking archives the account and clears the contact's sign-up type; the account keeps *Role / Portal* rather than being moved to *Role / Public*, because the anonymous group is reserved for automated tasks and guests. |
| Self-service removal is external-only | *Only the portal users can delete their accounts. The user(s) the comma-separated names can not be deleted.* |
| Self-service removal requires the password | The customer-facing page additionally requires the user to type their own login as a confirmation; a mismatch is reported as a validation error and a wrong password as a password error. |
| Application keys on the portal | Offered only when the parameter `portal.allow_api_keys` is set. |

The customer-facing password change refuses empty values (*You cannot leave any password empty.*),
refuses a mismatched confirmation (*The new password and its confirmation must be identical.*),
reports a wrong current password as *The old password you provided is incorrect, your password was
not changed.*, and, on success, recomputes and stores the session token so the user is not signed
out.

---

## 18. Data-recycling and privacy rules

| Rule | Statement |
|---|---|
| Archive action requires an archive flag | *This model doesn't manage archived records. Only deletion is possible.* |
| Notification frequency | *The notification frequency should be greater than 0* |
| Deactivating a rule | Deletes every outstanding candidate of that rule. |
| Candidate uniqueness | A record already having a candidate — active or discarded — is never proposed again by the same rule. |
| Discarding | Clears the candidate's active flag; the row survives precisely so the record is not re-proposed. |
| Validation with elevation | Archiving or deleting the pointed-at record is done with elevation, because the operator need not have permission on the target entity. |
| Notification recipients | Restricted to accounts holding *Role / Administrator*. |
| Privacy search visibility | A found record the acting user cannot read yields an empty reference rather than an error, so a multi-company restriction does not break the list. |
| Privacy log masking | Name and address are masked before storage (section 27.1 of [entities.md](entities.md)). |
| Privacy deletion idempotence | Deleting an already-deleted line raises *The record is already unlinked.* |

---

## 19. Locking and concurrency

- **The user deletion queue** takes a row-level lock for update on each request before processing it
  and re-tests that it is still in the *To Do* state, so two concurrent runs of the job never delete
  the same account twice.
- **Sign-in log rows** are created with no values so that the insert cannot conflict with a
  concurrent transaction.
- **Device log rows** are inserted on a separate writable connection when the current one is
  read-only.
- **Recycling** commits after each batch (5 000 candidates in automatic mode, 50 000 in manual mode)
  so that a timeout does not roll back the whole collection.
- **Sign-in failure counters** are per process and are explicitly documented as not thread-safe; they
  are a rate limiter, not a quota.
- **The settings screen** flushes before installing or uninstalling packages and resets the
  transaction afterwards, because the registry changes underneath.

---

## 20. Edge cases that a re-implementation must reproduce

1. **An empty record set passed to the check** performs the entity-level check only. Code uses this
   deliberately to ask "may this user do anything with this entity?".
2. **An access-right row with no group** grants the operation to *everyone*, including anonymous
   visitors.
3. **A record rule with an empty filter text** is the always-true filter, which as a grant makes the
   whole grant disjunction always true.
4. **A rule attached to a group the user does not hold** is invisible: it neither grants nor
   restricts.
5. **Deleting a shipped rule** is not durable: reloading its package re-creates it. Deactivating is
   the supported way to switch one off.
6. **The record-level check runs with the archive filter disabled**, so an archived record inside a
   set being checked is judged on the rules, not on its archive flag.
7. **Elevation does not bypass constraints.** An elevated write that violates a check constraint or a
   model constraint still fails.
8. **Elevation does not bypass the company-consistency check**, which is a constraint, not a
   permission.
9. **Reading the environment's company can raise**, even in code that is only reading. Any code
   narrowing to a company it is unsure about must elevate.
10. **The active-company list is part of the rule cache key**, so switching company invalidates
    nothing but does select a different cached filter.
11. **A user reading their own account** bypasses access rights when, and only when, every requested
    field is in the readable-by-oneself list, or its name begins with `context_`.
12. **A self-write including a non-permitted company** silently drops that key rather than failing.
13. **A password verified against an out-of-date hash is silently re-hashed** during a successful
    sign-in, and the session token is refreshed in the same breath.
14. **Enabling the second factor invalidates every open session** of the account, because the secret
    is a session-token field — the enrolling session is explicitly repaired.
15. **Registering or deleting a passkey likewise invalidates sessions**, and the acting session is
    explicitly repaired.
16. **A passkey sign-in never asks for a second factor**, whatever the account's configuration.
17. **Directory sign-in is only consulted for logins unknown locally.** An account that exists
    locally but is archived therefore cannot sign in through the directory.
18. **A successful directory password change empties the local password column**, so the local copy
    becomes unusable.
19. **The sign-up token is not stored**; invalidating it means changing the contact's sign-up type,
    or the contact's accounts, or signing in (which changes the most recent sign-in moment).
20. **The default-value precedence** puts an unscoped default before a user-scoped one under the
    ordering actually used; see [calculations.md](calculations.md) section 9.
21. **The cooldown threshold differs** between the code's fallback (5) and the value written at
    database creation (10).
22. **The anonymous account is archived** in the shipped data, and each company can mint its own.
23. **Duplicating a company is refused outright.** Duplicating a user produces a copy whose name and
    login are suffixed and which sends no invitation when no address is supplied.
24. **The company hierarchy is immutable**: a parent can be set at creation and never changed.
