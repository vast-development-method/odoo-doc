# Calculations and algorithms of the Contacts and Organizations domain

Every formula and every algorithm of the domain, with its inputs, its evaluation order, its
rounding where rounding applies, its failure conditions, and at least one worked example with
concrete values.

Notation used in the `formula` blocks:

- named quantities are written in words, joined by underscores;
- `+` between two pieces of text means concatenation;
- `×`, `÷`, `−`, `=` have their ordinary arithmetic meanings;
- `remainder_of(a ÷ b)` is the non-negative remainder of the integer division;
- `floor(x)` is the largest whole number not greater than `x`;
- `round_to_whole(x)` rounds to the nearest whole number and, on an exact half, to the even
  neighbour (this is the platform's fixed-point text formatting behaviour, and it matters in §12);
- `uppercase(...)`, `lowercase(...)`, `trim(...)` do what their names say, `trim` removing
  whitespace from both ends.

---

# 1. Naming

## 1.1 The complete name

The complete name is stored, indexed, and is the first sort key of the Party. It is recomputed
whenever the organization flag, the name, the parent's name, the address type, the free-text
company name or the commercial company name changes. It is always computed with an **empty
context**, so that no reading-context switch can leak into the stored value.

**Inputs**: the Party's own name; its organization flag; its parent; its free-text company name;
its commercial company name; its address type.

**Algorithm.**

1. Let *displayed types* be the three address types whose label is substituted for a missing name:
   `invoice`, `delivery`, `other`. The type `contact` is deliberately **not** in this set.
2. Let *name* be the Party's own name, or the empty string when it has none.
3. If the Party has **neither** a free-text company name **nor** a parent, the complete name is
   *name* trimmed. Stop.
4. Otherwise (the Party belongs to an organization, whether that organization exists as a record or
   only as free text):
   1. If *name* is empty and the address type is one of the displayed types, set *name* to the
      label of that address type: `Invoice`, `Delivery` or `Other`.
   2. If the Party is **not** an organization, and the reading context does not carry the
      suppression flag `partner_display_name_hide_company`, prefix the name:

```formula
name = organization_label + ", " + name
```

where

```formula
organization_label = commercial_company_name                        if it is non-empty
                   = name_of_the_parent (read with elevated rights)  otherwise
```

5. The complete name is *name* trimmed.

**Failure conditions**: none. The computation never raises.

**Worked example 1.1-A — an organization.**

| Input | Value |
|---|---|
| name | `Deco Addict` |
| organization flag | set |
| parent | none |
| address type | `contact` |

Step 3 applies (no parent, no free-text company name) → complete name = `Deco Addict`.

**Worked example 1.1-B — a person inside an organization.**

| Input | Value |
|---|---|
| name | `Douglas Fletcher` |
| organization flag | clear |
| parent | `Deco Addict` (an organization) |
| commercial company name | `Deco Addict` |
| address type | `contact` |

Step 4 applies; the name is non-empty so 4.1 does nothing; 4.2 applies because the Party is not an
organization → complete name = `Deco Addict, Douglas Fletcher`.

**Worked example 1.1-C — an unnamed delivery address.**

| Input | Value |
|---|---|
| name | empty |
| organization flag | clear |
| parent | `Deco Addict` |
| commercial company name | `Deco Addict` |
| address type | `delivery` |

Step 4.1 substitutes the label `Delivery`; step 4.2 prefixes → complete name =
`Deco Addict, Delivery`.

**Worked example 1.1-D — an unnamed contact-type child.**

Same as 1.1-C but with address type `contact`. Step 4.1 does **not** apply because `contact` is not
a displayed type, so the name stays empty; step 4.2 prefixes → complete name = `Deco Addict, `
which, after trimming, is `Deco Addict,` — with a trailing comma. This is the observed behaviour.
In practice it cannot be reached through the interface because a database check constraint refuses
a `contact`-type Party with an empty name.

**Worked example 1.1-E — a subsidiary organization.**

| Input | Value |
|---|---|
| name | `Deco Addict Belgium` |
| organization flag | **set** |
| parent | `Deco Addict` |

Step 4 is entered because a parent exists, but 4.2 is skipped because the Party is an organization
→ complete name = `Deco Addict Belgium`, with no prefix. A subsidiary keeps its own bare name.

**Worked example 1.1-F — a person with a free-text employer and no parent record.**

| Input | Value |
|---|---|
| name | `Marc Demo` |
| organization flag | clear |
| parent | none |
| free-text company name | `Azure Interior` |
| commercial company name | `Azure Interior` (derived from the free-text name, see §1.2) |

Step 4 is entered because the free-text company name is set; 4.2 prefixes with the commercial
company name → complete name = `Azure Interior, Marc Demo`.

## 1.2 The commercial company name

```formula
commercial_company_name = name_of_the_commercial_entity        if the commercial entity is an organization
                        = own_free_text_company_name           otherwise
```

Stored, recomputed when the free-text company name, the parent's organization flag or the
commercial entity's name changes.

**Worked example.** A delivery address under `Deco Addict` has no free-text company name of its
own. Its commercial entity is `Deco Addict`, which is an organization, so the commercial company
name is `Deco Addict`. If instead the delivery address stood alone with a free-text company name
`Deco Addict` and no parent, its commercial entity would be itself, which is not an organization,
so the commercial company name would fall through to the free-text value — again `Deco Addict`.

## 1.3 The display name

The display name is computed, never stored, and depends on the reading context. It is recomputed
when the complete name, the electronic mail address, the tax registration number, the state, the
country or the commercial company name changes, and it depends on six context keys:
`show_address`, `partner_show_db_id`, `show_email`, `show_vat`, `lang` and
`formatted_display_name`.

**Algorithm.**

1. **Branch A — the two-part formatted display** (`formatted_display_name` is present in the
   context):
   1. Let *name* be the Party's own name, or the empty string.
   2. If the Party has a parent **or** a free-text company name, replace *name* by

```formula
name = organization_part + " \t " + "--" + own_part + "--"
```

      where *organization_part* is the free-text company name when set, otherwise the parent's
      name; and *own_part* is the Party's own name when set, otherwise the label of its address
      type (`Contact`, `Invoice`, `Delivery`, `Other`), otherwise the empty string. The separator
      is a space, a horizontal tab character, and a space.
   3. If `show_email` is present and the Party has an electronic mail address, append
      ` \t --` + the address + `--`.
   4. Otherwise, if `partner_show_db_id` is present, append ` \t --` + the internal identifier +
      `--`.
2. **Branch B — the ordinary display** (no `formatted_display_name`):
   1. Let *name* be the complete-name computation of §1.1 run in the reading language.
   2. If `partner_show_db_id` is present, append a space and the internal identifier in
      parentheses.
   3. If `show_email` is present and the Party has an electronic mail address, append a space and
      the address between angle brackets.
   4. If `show_address` is present, append a line feed and the postal address rendered **without**
      the company line (§6).
   5. If `show_vat` is present and the Party has a tax registration number, append: when
      `show_address` was also present, a space, a line feed, a space and the number; otherwise a
      space, a hyphen, a space and the number.
3. **Both branches**: collapse every run of whitespace that immediately precedes a line feed into
   the line feed alone, then trim the result.

**Worked example 1.3-A (mandatory example: a display name built from parent and address flags).**

Given:

| Party | Value |
|---|---|
| own name | empty |
| organization flag | clear |
| address type | `delivery` |
| parent | `Deco Addict` (organization) |
| commercial company name | `Deco Addict` |
| street | `77 Santa Barbara Rd` |
| street 2 | empty |
| city | `Pleasant Hill` |
| state | `California`, code `CA` |
| postal code | `94523` |
| country | `United States`, layout `%(street)s\n%(street2)s\n%(city)s %(state_code)s %(zip)s\n%(country_name)s` |
| tax registration number | `US12345678` |
| internal identifier | `41` |

Reading context: `show_address` present, `show_vat` present, `formatted_display_name` absent,
`partner_show_db_id` absent, `show_email` absent.

Step B.1 — the complete name. The Party has a parent, its own name is empty and its address type
`delivery` is a displayed type, so the name becomes `Delivery`; the Party is not an organization,
so it is prefixed: `Deco Addict, Delivery`.

Step B.4 — the address without the company line, rendered by the country layout with
`street = "77 Santa Barbara Rd"`, `street2 = ""`, `city = "Pleasant Hill"`, `state_code = "CA"`,
`zip = "94523"`, `country_name = "United States"`:

```
77 Santa Barbara Rd

Pleasant Hill CA 94523
United States
```

(the second line is empty because the second street line is empty).

Intermediate value after step B.4:

```
Deco Addict, Delivery
77 Santa Barbara Rd

Pleasant Hill CA 94523
United States
```

Step B.5 — `show_address` was present, so the tax number is appended as a space, a line feed, a
space and the number:

```
Deco Addict, Delivery
77 Santa Barbara Rd

Pleasant Hill CA 94523
United States 
 US12345678
```

Step 3 — every run of whitespace directly before a line feed collapses. The trailing space on the
line `United States ` is removed. The final display name is:

```
Deco Addict, Delivery
77 Santa Barbara Rd

Pleasant Hill CA 94523
United States
 US12345678
```

with no leading or trailing whitespace overall. Note the single leading space on the last line: it
belongs to the ` ` before the number, not to a whitespace run before a line feed, so it survives.

**Worked example 1.3-B — the same Party read in the two-part formatted mode.**

Reading context: `formatted_display_name` present, `show_email` present, the Party's electronic
mail address being `deliveries@deco.example`.

Step A.2: the Party has a parent, its own name is empty, so *own_part* is the address-type label
`Delivery` and *organization_part* is `Deco Addict` (the parent's name, because there is no
free-text company name):

```
Deco Addict 	 --Delivery--
```

Step A.3 appends the address:

```
Deco Addict 	 --Delivery-- 	 --deliveries@deco.example--
```

Note that in branch A the address, the tax number and the identifier-when-mail-is-shown are never
appended: only one of the electronic mail address or the identifier appears, and the postal address
never does.

**Worked example 1.3-C — an organization read with the identifier switch.**

Party `Deco Addict`, organization, no parent, internal identifier `14`; context
`partner_show_db_id` present, everything else absent. Branch B: complete name `Deco Addict`, then
step B.2 appends ` (14)` → `Deco Addict (14)`.

## 1.4 The formatted electronic mail address

**Algorithm.**

1. Start with the value empty for every record.
2. Normalise the stored electronic mail address into a list of normalised addresses. Normalisation
   lower-cases a purely-ASCII local part, leaves a non-ASCII local part untouched, always
   lower-cases the domain, and strips any surrounding display name.
3. If the list is non-empty, the result is the display form built from the Party's name and the
   normalised addresses joined by commas.
4. Otherwise, if the stored value is non-empty (it could not be normalised — it is malformed), the
   result is the display form built from the Party's name and the raw stored value. The malformed
   value is deliberately preserved so that delivery failures name the real value.
5. Otherwise the result stays empty.

The **display form** of a (name, address) pair is:

```formula
display_form = address                                   if the name is empty
             = quotation_mark + name + quotation_mark + " <" + address + ">"   otherwise
```

with the name quoted, and encoded when it contains characters outside the target character set, and
the domain part converted to its ASCII-compatible encoding when it contains non-ASCII characters.

When the Party has no name, the literal four-character text `False` is used as the name part. This
is deliberate.

**Worked example.** Name `Deco Addict`, stored address `Sales@Deco.Example` → normalised
`sales@deco.example` → formatted `"Deco Addict" <sales@deco.example>`.

**Worked example — several addresses.** Name `Deco Addict`, stored value
`sales@deco.example, orders@deco.example` → both normalise → formatted
`"Deco Addict" <sales@deco.example,orders@deco.example>`. This is not a valid single address; it is
accepted because some mail servers tolerate it and because the alternative — silently dropping one
address — is worse.

**Worked example — malformed.** Name empty, stored value `not-an-address` → normalisation yields
nothing, the stored value is non-empty → formatted `"False" <not-an-address>`.

## 1.5 The ordering key

```formula
order_key = ( complete_name ascending , internal_identifier descending )
```

Two parties whose complete names are equal are therefore listed newest first.

---

# 2. Commercial-entity resolution

## 2.1 The rule

```formula
commercial_entity(P) = P                                  if P is an organization
                     = P                                  if P has no parent
                     = commercial_entity(parent_of(P))    otherwise
```

The field is **stored** and **recursive**: the platform recomputes it on a record and on every
descendant whenever the organization flag or the parent of any record in the chain changes.
Recursion terminates because the parent link is guaranteed acyclic by a validation.

## 2.2 Worked example (mandatory example: commercial-party resolution across three levels)

Consider this hierarchy.

| Level | Party | Internal identifier | Organization flag | Parent | Address type |
|---|---|---|---|---|---|
| 1 | `Azure Interior` | 10 | **set** | none | `contact` |
| 2 | `Azure Interior Benelux` | 11 | **set** | 10 | `contact` |
| 3 | `Brandon Freeman` | 12 | clear | 11 | `contact` |
| 3 | *(unnamed)* delivery address | 13 | clear | 11 | `delivery` |
| 4 | *(unnamed)* invoice address | 14 | clear | 12 | `invoice` |

Resolution, record by record:

| Party | Rule applied | Commercial entity |
|---|---|---|
| 10 `Azure Interior` | organization → itself | 10 |
| 11 `Azure Interior Benelux` | organization → itself (the parent is **not** followed) | 11 |
| 12 `Brandon Freeman` | not an organization, has a parent → the commercial entity of 11 | 11 |
| 13 delivery address | not an organization, has a parent → the commercial entity of 11 | 11 |
| 14 invoice address | not an organization, has a parent → the commercial entity of 12, which is 11 | 11 |

So the invoice address at level four resolves, through two hops, to the **subsidiary** `Azure
Interior Benelux`, not to the group parent `Azure Interior`. This is the single most important
consequence of the rule: an invoice raised on party 14 is legally raised on party 11, and its
receivable balance, its credit limit and its tax registration number are those of party 11.

**What changes if the subsidiary stops being an organization.** Clear the organization flag on
party 11. The recursive recomputation then gives:

| Party | Commercial entity |
|---|---|
| 10 | 10 |
| 11 | 10 (not an organization, has a parent → the commercial entity of 10) |
| 12 | 10 |
| 13 | 10 |
| 14 | 10 |

Every descendant now resolves to the group parent. The commercial company name of each descendant
changes from `Azure Interior Benelux` to `Azure Interior`, and so does every stored complete name.

**What changes if the subsidiary's parent is cleared.** Set the parent of party 11 to nothing while
leaving its organization flag clear. Party 11 now has no parent, so it is its own commercial entity
by the second clause, and parties 12, 13 and 14 resolve to 11 again — but now the commercial
company name of party 11 is not its own name (it is not an organization), it is party 11's
free-text company name, which is empty. So the complete name of party 12 becomes the prefix of an
empty organization label, that is `, Brandon Freeman`. This is why clearing a parent also clears
nothing else and why the free-text company name exists.

## 2.3 The "open the commercial entity" operation

Requires exactly one Party. Returns a window action of kind `ir.actions.act_window` (a window
action) on the Party entity, view mode `form`, record identifier equal to the commercial entity's
identifier, target `current` (the main content area, not a dialogue).

---

# 3. Field synchronization through the hierarchy

This is the defining algorithm of the domain. It keeps a Party, its parent and its descendants
consistent on two named field sets.

## 3.1 The two field sets

```formula
address_fields = { street , street2 , zip , city , state_id , country_id }
                 ∪ { city_id }      when the extended-address behaviour is installed

synced_commercial_fields = { vat }

commercial_fields = synced_commercial_fields ∪ { company_registry , industry_id }
```

The *formatting* address fields — the set used when rendering an address — is by default the same
as the address field set, and behaviours may narrow or widen it independently.

## 3.2 Helper computations

**Address values of a record** — used when pushing an address to a relative:

1. If **every** address field of the record is empty, the result is the empty map. An empty address
   is never pushed: a relative keeps whatever it already had.
2. Otherwise the result is the write-form of all the address fields, including the empty ones. So a
   record whose city is filled but whose state is empty pushes *both*, clearing the relative's
   state.

**Commercial values of a record** — used when pushing commercial fields down:

1. Collect only the commercial fields whose value on the record is non-empty.
2. If none are non-empty, the result is the empty map.
3. Otherwise the result is the write-form of exactly those fields. Empty commercial fields are
   **never** pushed: an organization with no industry does not erase a child's industry.

**Synchronised commercial values of a record** — the same, restricted to the synchronised subset.

**Updating an address** on a set of records writes the address-typed subset of a value map
directly at the storage level, bypassing the ordinary write path. This is essential: it prevents
the synchronization from re-triggering itself and looping.

## 3.3 The synchronization algorithm

**Precondition**: the record's new values are already in memory (the algorithm runs *after* the
write, not before). The input is the map of values that actually changed.

**Steps.**

1. Load the record's parent, address type and commercial entity.
2. **Upward, part one — inherit from the parent.** If the changed values contain a parent, or set
   the address type to `contact`:
   1. If the changed values contain a parent, synchronise the commercial fields *from* the
      commercial entity (see §3.4).
   2. If the record now has a parent **and** its address type is `contact`, take the parent's
      address values; if they are non-empty, write them onto the record with the direct address
      update.
3. **Upward, part two — push the address up.** Push the record's address to its parent when
   **all three** of the following hold:
   - the record has a parent and its address type is `contact`;
   - the changed values touch at least one address field, **or** contain a parent;
   - at least one address field of the record actually differs from the parent's.

   When they hold, the record's address values are written onto the parent with an ordinary write —
   which re-enters this same algorithm for the parent and therefore pushes the address down to the
   parent's other `contact`-type children.
4. **Upward, part three — push the synchronised commercial fields up.** Push when **all three**
   hold:
   - the record has a parent **and** the record is not itself the commercial entity;
   - the changed values touch at least one synchronised commercial field, **or** contain a parent;
   - at least one synchronised commercial field actually differs from the parent's.

   When they hold, the record's *non-empty* synchronised commercial values are written onto the
   parent with an ordinary write.
5. **Downstream — push to the children** (see §3.5).

**Postcondition**: every `contact`-type descendant of the record's parent shares the parent's
address; every non-organization descendant of the commercial entity carries the commercial
entity's non-empty commercial fields.

**Failure conditions**: an attempt to synchronise a reverse-list field raises
`One2Many fields cannot be synchronized as part of 'commercial_fields' or 'address fields'`. This
can only happen if a behaviour adds a reverse list to one of the two sets.

## 3.4 Inheriting the commercial fields from the commercial entity

1. Let *commercial entity* be the record's commercial entity.
2. If it is the record itself, do nothing.
3. Otherwise take the commercial entity's non-empty commercial values; if there are any, write them
   onto the record and then push them on to the record's own descendants (§3.6).
4. Then propagate the **company-dependent** commercial fields to every other company: for every
   company other than the acting one, read the commercial entity's values for the commercial fields
   that are company-dependent while acting for that company, and write them onto the record while
   acting for that company.

## 3.5 Pushing to the children

Input: the map of values that changed.

1. If the record has no children, stop.
2. **Commercial fields.** If the record *is* its own commercial entity, take the intersection of the
   changed field names with the commercial field set and push exactly those to the descendants
   (§3.6).
3. **Address fields.** If the changed values touch at least one address field, select the children
   whose address type is `contact` and write the *changed address values* onto them with the direct
   address update.

Note the asymmetry: the address push uses the **changed** values (so only the fields that moved are
overwritten), while the commercial push uses the commercial entity's **current** values.

## 3.6 Pushing the commercial fields to the descendants

Input: the set of field names to push (defaulting to the whole commercial field set).

1. Let *sync values* be the write-form of those fields read from the commercial entity —
   **including empty ones** this time, because the caller has already decided which fields to push.
2. Select the children that are **not** organizations. An organization child is its own commercial
   entity and is skipped, together with its whole subtree for these fields.
3. For each such child: if any of the fields to push differs between the child and the sync values,
   remember the child. Then, whether or not it differed, recurse into that child's own descendants
   with the same field set.
4. Write the sync values onto all the remembered children in one operation.

The "did it actually differ" test is what stops the recursion from writing over and over.

## 3.7 The first-contact guard

Run after a record is created through the bulk-load path (§3.8), and only there.

Push the child's address **up** to the parent when **all four** hold:

1. the parent is an organization, **or** the parent itself has no parent;
2. the child has at least one address field filled;
3. the parent has **no** address field filled;
4. the parent has exactly one child.

## 3.8 The bulk-load variant

When many parties are created at once by a data load, the per-record algorithm would issue one
write per record. The bulk path instead:

1. Creates every record with the synchronization suppressed (context flag
   `_partners_skip_fields_sync`).
2. Groups the created records by the pair (commercial entity identifier when a parent was supplied
   and the record is not its own commercial entity, parent identifier when the record has a parent
   and its address type is `contact`).
3. For each group, builds one value map: the commercial entity's whole commercial field set, then
   every **non-empty** address field of the parent; and writes that map once onto the whole group
   with elevated rights.
4. Then runs, per record, the downstream push (§3.5) and the first-contact guard (§3.7).

Note the difference from the per-record path: here the parent's **empty** address fields are not
pushed, whereas §3.2 pushes them.

## 3.9 Worked example (mandatory example: a company, a contact child, an invoicing child, and the company moves)

### Initial state

Three records are created in this order.

**Step 1 — the organization.**

| Field | Value |
|---|---|
| name | `Deco Addict` |
| organization flag | set |
| parent | none |
| street | `77 Santa Barbara Rd` |
| street 2 | empty |
| city | `Pleasant Hill` |
| state | `California` (code `CA`) |
| postal code | `94523` |
| country | `United States` |
| tax registration number | `US11223344` |

Synchronization runs on create. Step 2 does not apply (no parent in the values, the address type is
`contact` but the record has no parent, so 2.2 finds no parent). Step 3 does not apply (no parent).
Step 4 does not apply. Step 5 finds no children. Nothing happens. Identifier: 100.

**Step 2 — the contact child.**

| Field | Value |
|---|---|
| name | `Douglas Fletcher` |
| organization flag | clear |
| parent | 100 |
| address type | `contact` |
| every address field | left empty by the operator |

Synchronization on create:

- Step 2.1: the values contain a parent → inherit the commercial fields. The commercial entity is
  100, which is not the record, so its non-empty commercial values — here only the tax registration
  number `US11223344`, because the company registration number and the industry are empty — are
  written onto the child. The child then pushes them to its own descendants (it has none). Then the
  company-dependent commercial fields are propagated to every other company; the commercial field
  set contains no company-dependent field in the foundation package, so nothing happens.
- Step 2.2: the child has a parent and its address type is `contact` → take the parent's address
  values. The parent has at least one filled address field, so the map is non-empty and contains
  **all six** address fields. They are written directly onto the child.
- Step 3: the child has a parent, its type is `contact`, the values contain a parent → the third
  condition is evaluated: does any address field of the child differ from the parent's? After step
  2.2 they are all equal, so **no**. The upward push is skipped. (This ordering is essential:
  inheriting first is what prevents the empty child address from wiping out the parent's.)
- Step 4: the child has a parent and is not the commercial entity; the values contain a parent; does
  any synchronised commercial field differ from the parent's? After step 2.1 the tax number is
  equal, so **no**. Skipped.
- Step 5: no children.

Resulting child, identifier 101:

| Field | Value |
|---|---|
| street | `77 Santa Barbara Rd` |
| city | `Pleasant Hill` |
| state | `California` |
| postal code | `94523` |
| country | `United States` |
| tax registration number | `US11223344` |
| commercial entity | 100 |
| commercial company name | `Deco Addict` |
| complete name | `Deco Addict, Douglas Fletcher` |

**Step 3 — the invoicing child.**

| Field | Value |
|---|---|
| name | empty |
| organization flag | clear |
| parent | 100 |
| address type | `invoice` |
| street | `PO Box 1201` |
| city | `Pleasant Hill` |
| state | `California` |
| postal code | `94523` |
| country | `United States` |

Synchronization on create:

- Step 2.1: the values contain a parent → the commercial fields are inherited; the tax registration
  number `US11223344` is written onto the invoicing child too. (Commercial fields are inherited by
  **every** non-organization descendant, whatever its address type.)
- Step 2.2: the address type is `invoice`, **not** `contact` → the parent's address is **not**
  inherited. The operator's own street `PO Box 1201` survives.
- Step 3: the first condition requires the address type to be `contact` → skipped. The invoicing
  address is never pushed up.
- Step 4: the values contain a parent; the tax numbers are equal after 2.1 → skipped.
- Step 5: no children.

Resulting child, identifier 102: street `PO Box 1201`, the rest as typed, commercial entity 100,
complete name `Deco Addict, Invoice`.

### The company moves

The operator opens party 100 and changes the address to a new building:

| Field | Old value | New value |
|---|---|---|
| street | `77 Santa Barbara Rd` | `215 Vine Street` |
| street 2 | empty | `Suite 400` |
| city | `Pleasant Hill` | `Walnut Creek` |
| state | `California` | `California` (unchanged) |
| postal code | `94523` | `94596` |
| country | `United States` | `United States` (unchanged) |

The write path first captures the previous values, writes the four changed fields, and then runs
the synchronization with the **changed subset only**: street, second street line, city, postal
code. The state and country are not in the changed map because their values did not move.

- Step 2: the changed values contain neither a parent nor an address type of `contact` → skipped.
- Step 3: party 100 has no parent → skipped.
- Step 4: party 100 has no parent → skipped.
- Step 5 — push to children:
  - 5.2: party 100 *is* its own commercial entity, so the intersection of the changed field names
    with the commercial field set is computed. The changed names are four address fields; the
    intersection is empty; the descendant push runs with an empty field set, which writes nothing.
  - 5.3: the changed values touch address fields → select the children whose address type is
    `contact`. That is party 101 only. Party 102 is of type `invoice` and is **not** selected.
    Write the four changed address values directly onto party 101.

Final state:

| Party | Type | street | street 2 | city | state | postal code | country |
|---|---|---|---|---|---|---|---|
| 100 `Deco Addict` | `contact` | `215 Vine Street` | `Suite 400` | `Walnut Creek` | `California` | `94596` | `United States` |
| 101 `Douglas Fletcher` | `contact` | `215 Vine Street` | `Suite 400` | `Walnut Creek` | `California` | `94596` | `United States` |
| 102 invoicing address | `invoice` | `PO Box 1201` | empty | `Pleasant Hill` | `California` | `94523` | `United States` |

The contact child followed the move; the invoicing address did not. This is the intended behaviour:
a person's location is the organization's location, but a billing address is an independent fact.

Note also that party 101's address was updated through the **direct address update**, which writes
at the storage level. No further synchronization pass runs for party 101, so the move does not
bounce back up to party 100.

### The reverse direction

Now the operator edits party 101's street to `215 Vine Street, Building B` (keeping everything else).

- The write happens on party 101; the changed map contains the street only.
- Step 2: the changed values contain neither a parent nor an address-type change to `contact` →
  skipped. Note that step 2 is entered only on a *parent change* or a *type change*, not on an
  address change.
- Step 3: party 101 has a parent; its type is `contact`; the changed values touch an address field;
  and the street now differs from the parent's → **all three hold**. Party 101's address values
  (all six fields) are written onto party 100 with an ordinary write.
- That write re-enters the algorithm for party 100 with the changed map `{street}`. Step 5.3 pushes
  the street down to every `contact`-type child of party 100 — which is party 101 again, now
  already equal, and any other contact-type children, which now follow.
- Step 4: the synchronised commercial fields are unchanged → skipped.

So editing a contact's address edits the organization's address and every sibling contact's
address. This is deliberate and is the reason the interface marks the address fields of a
`contact`-type child read-only while it has a parent.

## 3.10 Worked example — the tax registration number typed on a child

Party 101 is given the tax registration number `US99887766` (it previously held `US11223344`
inherited from party 100).

- The write happens; the changed map is `{vat}`.
- Step 2: no parent change, no type change to `contact` → skipped.
- Step 3: the changed values touch no address field and contain no parent → skipped.
- Step 4: party 101 has a parent and is not the commercial entity; the changed values touch a
  synchronised commercial field; the value now differs from the parent's → **all three hold**. The
  non-empty synchronised commercial values of party 101 — `{vat: "US99887766"}` — are written onto
  party 100 with an ordinary write.
- That write re-enters the algorithm on party 100 with the changed map `{vat}`. Step 5.2 applies:
  party 100 is its own commercial entity, the intersection of `{vat}` with the commercial field set
  is `{vat}`, so the value is pushed to every non-organization descendant. Parties 101 and 102 both
  receive `US99887766`.

Net effect: typing a tax number on any contact of an organization sets it for the whole
organization. Typing a **company registration number** on a contact does *not*, because the
registration number is in the commercial field set but not in its synchronised subset: it travels
down only.

---

# 4. Parsing a name from a combined name and address

## 4.1 Creating a Party from a text label

Used whenever an operator types a value into a reference field and asks for it to be created on the
spot, and whenever an inbound message names a sender that does not yet exist.

**Algorithm.**

1. If the creation context carries a default address type that is not one of the four valid tokens,
   remove that default from the context before going further.
2. Split the input text into a name and a normalised electronic mail address (§4.3).
3. If the context carries the flag `force_email` and no address was found, fail with
   `Couldn't create contact without email address!`
4. Build the creation values: the name field is set to the parsed name if there is one, otherwise to
   the normalised address. When an address was found, it is also set on the electronic mail field.
5. Create the Party.
6. Return its internal identifier and its display name.

**Worked example (mandatory example: a name parsed from a combined name and address).**

Input text: `"Raoul le Grand" <raoul@grosbedon.example>`

Step 2 splits it into the name `Raoul le Grand` and the address `raoul@grosbedon.example`. Step 4
produces the creation values `{name: "Raoul le Grand", email: "raoul@grosbedon.example"}`. Step 6
returns the new identifier and the display name `Raoul le Grand`.

**Worked example — no quotes.**

Input text: `Raoul <raoul@grosbedon.example>` → name `Raoul`, address `raoul@grosbedon.example`.

**Worked example — no angle brackets (the space-separated fall-back).**

Input text: `Raoul raoul@grosbedon.example`

The standard address parser returns one pair whose name is empty and whose address is the whole
string `Raoul raoul@grosbedon.example`. The space-separated fall-back then applies (§4.3 step 3):
the string is re-parsed with every space replaced by a comma, giving the pieces `Raoul` and
`raoul@grosbedon.example`; the pieces without an at-sign become the name, the piece with an at-sign
becomes the address. Result: name `Raoul`, address `raoul@grosbedon.example`.

**Worked example — a bare address.**

Input text: `raoul@grosbedon.example` → name empty, address `raoul@grosbedon.example`. Step 4 then
sets the **name** field to `raoul@grosbedon.example` as well as the address field, because the
parsed name is empty.

**Worked example — free text with no address at all.**

Input text: `Deco Addict` → the address parser finds no pair with an at-sign, so the name becomes
the whole input text and the address stays empty. Creation values: `{name: "Deco Addict"}`.

**Worked example — an upper-cased address.**

Input text: `Raoul <Raoul@GrosBedon.Example>` → the address is normalised to
`raoul@grosbedon.example` (the local part is pure the basic character encoding so it is lower-cased; the domain is always
lower-cased). Name `Raoul`.

**Worked example — two addresses.**

Input text: `tony@e.example, "Tony2" <tony2@e.example>` → the parser returns two pairs; only the
**first** is used, and the normalisation runs in non-strict mode so a multi-address input still
yields a value. Result: name empty, address `tony@e.example`, and the created Party's name is
`tony@e.example`.

## 4.2 Finding or creating a Party from an address

**Algorithm.**

1. If the input is empty, fail with `An email is required for find_or_create to work`.
2. Split the input into a name and a normalised address (§4.3).
3. If no address was found and the caller asked for a valid address, fail with
   `A valid email is required for find_or_create to work properly.`
4. If an address was found, search for the first Party whose electronic mail address equals the
   normalised address under a case-insensitive comparison. If one is found, return it and stop.
5. Otherwise create a Party exactly as in §4.1 steps 4 to 5 and return it.

Note the difference from §4.1: this operation returns the **record**, not the pair of identifier
and display name, and it never raises for a missing address unless asked to.

The search in step 4 is a plain case-insensitive equality on the stored electronic mail address. A
Party whose stored value is `Sales@Deco.Example` is therefore found by a search for
`sales@deco.example`, but a Party whose stored value is
`sales@deco.example, orders@deco.example` is **not** found by a search for either component.

## 4.3 The splitting grammar

1. If the input is empty or contains only whitespace, the result is the pair (empty, empty).
2. Parse the input with the standard address-list grammar. Keep only the pairs whose address part
   is non-empty **and** contains an at-sign; the grammar returns an empty address when it fails,
   and sometimes returns a fragment with no at-sign, neither of which is a usable address.
3. **The domain-only correction.** If any surviving pair's address begins with an at-sign — which
   happens for inputs such as `@example.com` — discard the parsed pairs entirely and re-extract
   addresses with a plain pattern match, keeping only those that do not begin with an at-sign; each
   becomes a pair with an empty name.
4. **The space-separated fall-back**, applied to every surviving pair: when the pair has no name but
   its address contains a space, re-parse the address with every space replaced by a comma; among
   the resulting pieces, those without an at-sign are joined by single spaces to form the name and
   the first piece with an at-sign becomes the address. If no piece contains an at-sign, the pair is
   left as it was.
5. Take the **first** surviving pair. If it has an address, normalise it in non-strict mode; when
   normalisation fails, keep the raw address. If there is no pair at all, the name becomes the whole
   input text and the address stays empty.

**Normalisation** of an address:

1. Locate the local part and the domain part around the last at-sign.
2. If the local part is purely the basic character encoding, lower-case it; otherwise leave it as it is, because the
   internationalised-mail extension makes non-ASCII local parts meaningful.
3. Lower-case the domain part always.
4. In strict mode, refuse to return anything when the input contained more than one address; in
   non-strict mode, return the first candidate.

---

# 5. Address resolution

## 5.1 The operation

Given a Party and a set of requested address types, return a map from each requested type to a
Party identifier.

**Algorithm.**

1. Let *requested* be the set of requested types. Add `contact` to it if it is not already there.
   (The caller therefore always gets a `contact` entry, even when it did not ask for one.)
2. Let *result* be an empty map and *visited* be an empty set.
3. For each Party in the input set:
   1. Let *current* be that Party.
   2. **Loop A (walk up the ancestors).** While *current* exists:
      1. Let *to scan* be the one-element list containing *current*.
      2. **Loop B (depth-first descent).** While *to scan* is not empty:
         1. Remove the **first** element and call it *record*; add it to *visited*.
         2. If *record*'s address type is in *requested* and *result* has no entry for that type
            yet, record `result[type] = identifier of record`.
         3. If *result* now has an entry for every requested type, return *result* immediately.
         4. Prepend *record*'s children — those not already visited and **not** organizations — to
            *to scan*, keeping their natural order and placing them before whatever was already
            queued. This makes the traversal depth-first.
      3. If *current* is an organization, or has no parent, leave loop A.
      4. Otherwise set *current* to its parent and repeat loop A.
4. Let *fall-back* be `result["contact"]` if present; otherwise the identifier of the input Party;
   otherwise the boolean false.
5. For every requested type with no entry, set the entry to *fall-back*.
6. Return *result*.

**Key properties.**

- The descent **stops at organizations**: a subsidiary's addresses are never offered as the parent's
  addresses. The boundary is the organization flag, not the address type.
- The ascent **also stops at organizations**: once the walk reaches an organization it does not go
  further up.
- The first match wins per type, and the traversal order is: the starting Party itself, then its
  descendants depth-first, then the parent and *its* descendants, and so on.
- The `contact` type is always resolved, and it is the fall-back for every other type.

## 5.2 Worked example

Hierarchy:

| Identifier | Name | Organization | Parent | Address type |
|---|---|---|---|---|
| 100 | `Deco Addict` | yes | — | `contact` |
| 101 | `Douglas Fletcher` | no | 100 | `contact` |
| 102 | *(unnamed)* | no | 100 | `invoice` |
| 103 | *(unnamed)* | no | 100 | `delivery` |
| 104 | `Deco Addict Belgium` | **yes** | 100 | `contact` |
| 105 | *(unnamed)* | no | 104 | `delivery` |

**Resolve for party 101, requesting `invoice` and `delivery`.**

Requested becomes `{invoice, delivery, contact}`.

- Loop A, current = 101.
  - Loop B: scan 101 → its type `contact` is requested and unset → `result[contact] = 101`. Result
    has one of three entries. Children of 101: none. To-scan empty.
  - 101 is not an organization and has a parent → current = 100.
  - Loop B: scan 100 → type `contact` already set, skip. Children of 100 that are not organizations
    and not visited: 102, 103. (104 is an organization and is excluded.) To-scan = [102, 103].
    - Scan 102 → `result[invoice] = 102`. Two of three.
      Children of 102: none. To-scan = [103].
    - Scan 103 → `result[delivery] = 103`. Three of three → **return immediately**.
- Result: `{contact: 101, invoice: 102, delivery: 103}`.

**Resolve for party 105, requesting `invoice`.**

Requested becomes `{invoice, contact}`.

- Loop A, current = 105.
  - Loop B: scan 105 → type `delivery`, not requested. Children: none.
  - 105 is not an organization, has a parent → current = 104.
  - Loop B: scan 104 → type `contact` requested and unset → `result[contact] = 104`. Children of 104
    that are not organizations: 105, already visited → excluded. To-scan empty.
  - 104 **is** an organization → leave loop A.
- Fall-back = `result[contact]` = 104. `result[invoice]` is unset → set to 104.
- Result: `{contact: 104, invoice: 104}`.

The group's invoicing address 102 is **not** reached, because the walk up stopped at the subsidiary.

**Resolve for party 100, requesting `delivery`.**

Requested becomes `{delivery, contact}`.

- Loop A, current = 100.
  - Loop B: scan 100 → `result[contact] = 100`. Children that are not organizations: 101, 102, 103.
    To-scan = [101, 102, 103].
    - Scan 101 → type `contact`, already set. Children of 101: none. To-scan = [102, 103].
    - Scan 102 → type `invoice`, not requested. To-scan = [103].
    - Scan 103 → `result[delivery] = 103`. Two of two → return.
- Result: `{contact: 100, delivery: 103}`.

**Resolve for a Party with no relatives at all**, party 200, requesting `invoice`: loop B records
`result[contact] = 200`; loop A stops because there is no parent; the fall-back is 200; the result
is `{contact: 200, invoice: 200}`. A lone Party is its own every address.

## 5.3 Use by the Company

A Company's displayed address is resolved with the single preference `contact` against the
Company's Party. So a Company whose Party has a `contact`-type child inherits that child's address
for display, while writing any address field on the Company writes it back onto the Company's own
Party — not onto the resolved child. Because the two are kept equal by the address synchronization,
the asymmetry is invisible in practice; it becomes visible only when the Party's `contact`-type
child was deliberately given a divergent address through the direct storage path.

---

# 6. Rendering a postal address

## 6.1 Choosing the layout

```formula
layout = country_layout          when the Party has a country and that country has a layout
       = default_layout          otherwise
```

```formula
default_layout = "%(street)s" + newline
               + "%(street2)s" + newline
               + "%(city)s %(state_code)s %(zip)s" + newline
               + "%(country_name)s"
```

The default layout is also the shipped default value of the country's own layout field, so most
countries render identically to the default unless they were given something else.

## 6.2 Building the substitution map

The map defaults every missing key to the empty string. It is filled with:

| Key | Value |
|---|---|
| `state_code` | the state's code, or the empty string |
| `state_name` | the state's name, or the empty string |
| `country_code` | the country's two-letter code, or the empty string |
| `country_name` | the country's name, or the empty string |
| `company_name` | the Party's commercial company name, or the empty string |
| every formatting address field | that field's value on the Party, or the empty string |

Then:

- If the caller asked for the address **without** the company, the `company_name` entry is forced to
  the empty string.
- Otherwise, if the commercial company name is non-empty, the layout is **prefixed** with
  `%(company_name)s` and a line feed.

## 6.3 Substituting

The rendered address is the layout with every key replaced by its value. A key present in the
layout but absent from the map substitutes to the empty string rather than failing, because the map
defaults.

Empty values leave empty lines and doubled spaces behind. The renderer does **not** collapse them.
Consumers that need a tidy address collapse the whitespace themselves — the display-name
computation, for example, collapses every whitespace run that precedes a line feed.

## 6.4 Worked example (mandatory example: an address rendered by a country format)

**Party.**

| Field | Value |
|---|---|
| commercial company name | `Deco Addict` |
| street | `Chaussée de Namur 40` |
| street 2 | empty |
| postal code | `1367` |
| city | `Ramillies` |
| state | none |
| country | `Belgium` |

**Country layout for Belgium** (as shipped):

```
%(street)s
%(street2)s
%(zip)s %(city)s
%(country_name)s
```

Note that Belgium puts the postal code **before** the city, unlike the default layout which puts
the city first and inserts the state code between them.

**Substitution map:** `street = "Chaussée de Namur 40"`, `street2 = ""`, `zip = "1367"`,
`city = "Ramillies"`, `state_code = ""`, `state_name = ""`, `country_code = "BE"`,
`country_name = "Belgium"`, `company_name = "Deco Addict"`.

**With the company line** (the default): the layout is prefixed, giving

```
%(company_name)s
%(street)s
%(street2)s
%(zip)s %(city)s
%(country_name)s
```

and the result is

```
Deco Addict
Chaussée de Namur 40

1367 Ramillies
Belgium
```

— five lines, the third of which is empty because the second street line is empty.

**Without the company line**: the result is

```
Chaussée de Namur 40

1367 Ramillies
Belgium
```

**The same Party with the United States layout** (to show the difference), keeping the Belgian
postal code and city but setting the country to `United States` and the state to `California`
(code `CA`):

```
Chaussée de Namur 40

Ramillies CA 1367
United States
```

**The same Party in China**, whose shipped layout is
`%(country_name)s, %(zip)s\n%(state_name)s %(city)s %(street)s %(street2)s`:

```
China, 1367
 Ramillies Chaussée de Namur 40 
```

— two lines; the second begins with a space because the state name is empty and ends with a space
because the second street line is empty. This is exactly what the substitution produces.

## 6.5 The input-form reordering

The **form** is reordered to match the layout as well, but by a different mechanism and against the
country of the **acting company**, not the Party. The procedure is in
[entities.md](entities.md) §18.3. Worked example, for a company whose country is Belgium:

1. The Belgian layout's first line containing the key `city` is `%(zip)s %(city)s`.
2. The keys extracted, in order of appearance, are `zip`, `city`.
3. The anchor is the postal-code input. The set still to place is {city, state} (postal code
   removed).
4. Walking the remaining keys: `city` → move the city input immediately after the postal-code
   input; the set becomes {state}.
5. The leftover state input is moved immediately after the city input.

Resulting input order inside the address block: street, second street line, **postal code, city,
state**, country. For a company whose country is the United States, whose layout's city line is
`%(city)s %(state_code)s %(zip)s`, the extracted keys are `city`, `state_code`, `zip`; the
`state_code` key maps onto the state input; the resulting order is **city, state, postal code**.

## 6.6 The layout's address-field extraction

A country can report the list of keys its layout mentions by extracting every parenthesised group
from the layout string. For the Belgian layout this yields `street`, `street2`, `zip`, `city`,
`country_name` — in the order they appear. Note that this extraction takes *everything* between
parentheses, including keys that are not address fields such as `country_name`.

---

# 7. Bank account numbers

## 7.1 Sanitising an account number

```formula
sanitized_account_number = uppercase( delete_every_non_word_character( account_number ) )
```

where a *word character* is a letter, a digit or the underscore. Every other character — spaces,
hyphens, full stops, solidi, parentheses — is removed. An empty input yields the boolean false.

**Worked examples.**

| Raw account number | Sanitised |
|---|---|
| `BE71 0961 2345 6769` | `BE71096123456769` |
| `be71-0961-2345-6769` | `BE71096123456769` |
| `123-4567890-02` | `123456789002` |
| `GB33BUKB20201555555555` | `GB33BUKB20201555555555` |

Because the underscore survives, `A_1` sanitises to `A_1`, not to `A1`. This is a consequence of
the character class and is stated here so a rebuild reproduces it exactly.

## 7.2 Searching by account number

A search on the account number is rewritten into a search on the sanitised number, with the search
term sanitised first — element by element for the "is in" and "is not in" operators, and as a whole
otherwise. A user searching for `BE71 0961 2345 6769` therefore finds the account whatever spelling
it was stored in.

## 7.3 Find or create a bank account

**Inputs**: an account number; a Party; a Company; two options — whether creating an account for one
of the business's own companies is allowed, and extra values to use only when creating.

**Algorithm.**

1. Search, with elevated rights and with archived records included, for a bank account whose
   account number matches the input (the match goes through the sanitising search of §7.2) **and**
   whose holder is the given Party's commercial entity or any descendant of it.
2. If none is found:
   1. If creating an account for one of the business's own companies is **not** allowed, and the
      given Party is the Party of one of the companies in the database, fail with
      `Please add your own bank account manually: <the account number> (<the Party's display name>)`.
   2. Otherwise create the account with a cleaned context, using the extra creation values, the
      account number, the Party as holder, and the outgoing-payment permission explicitly
      **cleared**.
3. From the accounts now in hand, keep those that satisfy the company-scoping domain for the given
   Company (their company is that Company or an ancestor of it, or they have no company) **and**
   that are active.
4. Sort the survivors so that accounts whose holder is exactly the given Party come first — the sort
   key is the boolean "the holder is not the given Party", ascending, so false (the holder *is* the
   Party) sorts before true.
5. Return the first survivor, dropping the elevated rights. When nothing survives, return nothing.

**Why the guard in 2.1 exists.** An incoming electronic invoice carries the supplier's account
number. If the invoice was in fact issued to the business by itself, or if the number in the
document is the business's own, silently creating a bank account for the business's own Party would
make that number payable. The guard forces a human to do it.

**Worked example.** An incoming bill from `Deco Addict` (party 100, whose commercial entity is
itself) carries the account number `BE71 0961 2345 6769`. Company `My Company` is acting.

- Step 1 searches for an account whose sanitised number is `BE71096123456769` held by party 100 or
  any descendant. Suppose party 101 (a contact of 100) already holds it.
- Step 2 is skipped.
- Step 3 keeps it if its company is `My Company`, an ancestor of `My Company`, or empty, and if it
  is active.
- Step 4 sorts: the holder is 101, not 100, so the key is true; but it is the only candidate.
- The account held by party 101 is returned. The bill will be paid to it.

If instead both party 100 and party 101 held an account with that number, step 4 would put party
100's account first and it would be returned.

---

# 8. Tax registration numbers

## 8.1 Where the checks live

The foundation package stores the tax registration number and offers two extension points that do
nothing: a per-record check, and a normalise-and-validate routine that returns the value unchanged
together with the country's code. The tax-number behaviour replaces the second with the real
algorithm; the accounting behaviour replaces the first so that the check runs against the
**commercial entity's** country rather than the record's own.

## 8.2 The per-record check

For each Party:

1. Run the normalise-and-validate routine with the **commercial entity's** country, the Party's
   tax registration number, the Party's name (for the message) and the requested validation mode.
2. If the returned number differs from the stored one, write the returned number back. The
   comparison before writing is deliberate: it avoids a write, and therefore a whole
   synchronization pass, when nothing changed.

Three validation modes exist:

| Mode | Behaviour on an invalid number |
|---|---|
| `error` | raise |
| `setnull` | return the empty string, silently clearing the field |
| `False` (no validation) | return the number normalised but unchecked |

The check runs as the **inverse** of the tax-number field and of the country field, so it fires on
every write of either; and it runs in non-raising mode from the form's change handler so that
typing an incomplete number does not throw while the operator is still typing.

## 8.3 The normalise-and-validate routine

**Inputs**: a country, a number, a name for the message, a validation mode.

**Steps.**

1. If either the country or the number is empty, return the number unchanged and no country code.
2. **The single-character escape.** If the number is exactly one character long:
   - if it is `/`, or if validation is switched off, return it unchanged and no country code — `/`
     is the agreed marker for "this counterparty has no tax number";
   - if the mode is `setnull`, return the empty string;
   - if the mode is `error`, fail with
     `To explicitly indicate no (valid) VAT, use '/' instead. `
3. **Split the prefix.** Take the first two characters, upper-cased, as a candidate prefix and the
   rest with spaces removed as the number body. If the candidate prefix is not purely alphabetic,
   there is no prefix: the prefix is the empty string and the body is the whole number.
4. **The union-wide prefix escape.** If the prefix is `EU` and the country is not a member of the
   union's country group, return the number unchanged and no country code. Companies outside the
   union that trade with non-businesses inside it are issued numbers prefixed `EU`.
5. **Decide which country to validate against.**
   1. Let *mapped code* be the country code that corresponds to the prefix: the prefix itself,
      unless the prefix is one of the two special prefixes, in which case it is the country code it
      stands for (`EL` stands for `GR`, `XI` stands for `GB`).
   2. If *mapped code* is the code of a country in the prefixing group:
      - if the **given** country is itself in the prefixing group and a prefix was present, then the
        number body alone is kept and the prefix is remembered as the *prefixed country*;
      - otherwise a second, union-wide attempt is armed.
   3. The *code to check* is the prefixed country if one was remembered, otherwise the given
      country's code.
6. **Normalise.** Run the country-specific compaction for the *code to check* (§8.5). This removes
   punctuation and applies any country-specific re-spacing.
7. If the prefixed country is `GR`, replace it by `EL` — the country code and the tax prefix differ
   for Greece.
8. The **value to return** is the prefixed country (possibly empty) followed by the normalised body.
9. If validation is switched off, or the reading context carries the suppression flag
   `no_vat_validation`, return that value together with the *code to check*. The suppression flag
   exists so that numbers pushed in from an external platform, where the sender controls nothing,
   can be stored as they are.
10. **Detect a doubled prefix.** The value is invalid when a prefixed country was remembered and the
    value begins with that prefix twice — `BEBE0477472701`.
11. **Validate.** Run the country-specific check for the *code to check* on the normalised body
    (§8.4). If it passes and the prefix is not doubled, return the value and the *code to check*.
12. **On failure**:
    - If the union-wide attempt was armed in step 5.2, retry the whole routine with the country
      whose code is *mapped code* and with the original prefix and body rejoined. If that retry also
      fails, fail with the standard message followed by a blank line and
      `If you are trying to input a European number, this is the expected format: ` and the example
      for that country (§8.6).
    - Else if the mode is `error`, fail with the standard message.
    - Else return the empty string together with the *code to check*.

**The standard failure message.**

```
The <tax label> number [<the value>] for <the record label> does not seem to be valid. 
Note: the expected format is <the example>
```

where the tax label is the acting company's country's own word for the tax number when the code
being checked is that company's country code and such a word is defined, and otherwise the literal
`VAT`; the record label is `partner [<the Party's name>]`; and the note is omitted entirely when no
example exists for the country. When the record label contains the text `False` — which happens for
the public visitor's nameless Party — the phrase `for <the record label>` is dropped:

```
The <tax label> number [<the value>] does not seem to be valid. 
Note: the expected format is <the example>
```

Both messages begin with a line feed.

## 8.4 The per-country check algorithms

Every country falls back to a standard numeric-identifier library unless the system defines its own
check. The library is consulted by looking up a module for the country code and calling its
validity test; when neither a system-defined check nor a library module exists, the number is
accepted unconditionally. The system-defined checks follow.

### 8.4.1 Albania

```formula
valid = length(compacted_number) = 10
        AND compacted_number matches:  one of J K L M , then 8 digits , then one uppercase letter
```

The compaction is the library's national-identifier compaction.

### 8.4.2 Switzerland

The accepted spellings, ignoring spaces, are `CHE#########MWST`, `CHE#########TVA`,
`CHE#########IVA`, `CHE-###.###.### MWST`, `CHE-###.###.### TVA`, `CHE-###.###.### IVA`. The
English abbreviation for value-added tax is deliberately **not** accepted.

The pattern applied to the number *after* the country prefix has been stripped is: the letter `E`,
then either nine digits or a hyphen and three groups of three digits separated by full stops, then
an optional space, then one of the three national abbreviations, anchored at the end.

Check digit, over the nine digits:

```formula
weights = ( 5 , 4 , 3 , 2 , 7 , 6 , 5 , 4 )
checksum = sum over i from 1 to 8 of ( digit_i × weight_i )
check = remainder_of( ( 11 − remainder_of( checksum ÷ 11 ) ) ÷ 11 )
valid = ( check = digit_9 )
```

**Worked example.** Number `CHE-123.456.788 TVA`. Digits: 1 2 3 4 5 6 7 8 8.
checksum = 1×5 + 2×4 + 3×3 + 4×2 + 5×7 + 6×6 + 7×5 + 8×4 = 5 + 8 + 9 + 8 + 35 + 36 + 35 + 32 = 168.
168 ÷ 11 = 15 remainder 3. 11 − 3 = 8. 8 ÷ 11 = 0 remainder 8. check = 8 = the ninth digit → valid.

### 8.4.3 Norway

1. A twelve-character number whose last three characters, upper-cased, are `MVA` has those three
   removed.
2. The remainder must be exactly nine characters and must be entirely numeric.

```formula
weights = ( 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )
checksum = sum over i from 1 to 8 of ( digit_i × weight_i )
check = 11 − remainder_of( checksum ÷ 11 )
if check = 11 then check = 0
if check = 10 then the number is invalid
valid = ( check = digit_9 )
```

**Worked example.** `123456785`. checksum = 1×3 + 2×2 + 3×7 + 4×6 + 5×5 + 6×4 + 7×3 + 8×2 = 3 + 4 +
21 + 24 + 25 + 24 + 21 + 16 = 138. 138 ÷ 11 = 12 remainder 6. check = 11 − 6 = 5 = the ninth digit →
valid.

### 8.4.4 Peru

Exactly eleven digits.

```formula
weights = ( 5 , 4 , 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )
checksum = sum over i from 1 to 10 of ( digit_i × weight_i )
check = 11 − remainder_of( checksum ÷ 11 )
if check = 10 then check = 0
if check = 11 then check = 1
valid = ( check = digit_11 )
```

### 8.4.5 Russia

Either ten or twelve digits, all numeric.

For ten digits:

```formula
weights = ( 2 , 4 , 10 , 3 , 5 , 9 , 4 , 6 , 8 )
checksum = sum over i from 1 to 9 of ( digit_i × weight_i )
check = remainder_of( checksum ÷ 11 )
valid = ( remainder_of( check ÷ 10 ) = digit_10 )
```

For twelve digits, two checks in sequence:

```formula
weights_1 = ( 7 , 2 , 4 , 10 , 3 , 5 , 9 , 4 , 6 , 8 )
checksum_1 = sum over i from 1 to 10 of ( digit_i × weight_1_i )
first_check = remainder_of( checksum_1 ÷ 11 )
must hold:  first_check = digit_11

weights_2 = ( 3 , 7 , 2 , 4 , 10 , 3 , 5 , 9 , 4 , 6 , 8 )
checksum_2 = sum over i from 1 to 11 of ( digit_i × weight_2_i )
second_check = remainder_of( checksum_2 ÷ 11 )
must hold:  second_check = digit_12
```

Note that the ten-digit form takes the remainder modulo ten of the check before comparing, while
the twelve-digit form does not.

### 8.4.6 Uruguay

1. Clean the value of spaces and hyphens, upper-case it, trim it, and remove a leading `UY`.
2. It must be entirely numeric and exactly twelve digits.
3. The first two digits, read as text, must lie between `01` and `22` inclusive.
4. Digits three to eight must not all be zero.
5. Digits nine to eleven must be exactly `001`.
6. The check digit:

```formula
weights = ( 4 , 3 , 2 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )
total = sum over i from 1 to 11 of ( digit_i × weight_i )
check = remainder_of( ( − total ) ÷ 11 )      taking the non-negative remainder
valid = ( check = digit_12 )
```

**Worked example.** `219999830019`. The first eleven digits are 2 1 9 9 9 9 8 3 0 0 1.
total = 2×4 + 1×3 + 9×2 + 9×9 + 9×8 + 9×7 + 8×6 + 3×5 + 0×4 + 0×3 + 1×2
= 8 + 3 + 18 + 81 + 72 + 63 + 48 + 15 + 0 + 0 + 2 = 310.
−310 modulo 11: 310 ÷ 11 = 28 remainder 2, so −310 ≡ −2 ≡ 9 (mod 11). check = 9 = the twelfth digit
→ valid. The first two digits `21` are within `01`..`22`; digits three to eight `999983` are not all
zero; digits nine to eleven are `001`. Valid.

### 8.4.7 Venezuela

The accepted spellings, case-insensitively, are: a kind letter, then either a bare eight-digit
number or a punctuated one, then a single check digit. The punctuation is all-or-nothing: either
the form is `X-##.###.###-#` (hyphen after the letter, full stops inside, hyphen before the check
digit) or `X-########-#` (hyphens but no full stops) or `X#########` (nothing at all). A mixture is
refused.

Kind letters and their kind digits:

| Letter | Meaning | Kind digit |
|---|---|---|
| `V` | citizen | 1 |
| `E` | foreigner | 2 |
| `C` | township or communal council | 3 |
| `J` | legal entity | 3 |
| `P` | passport | 4 |
| `G` | government | 5 |

```formula
weights = ( 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )
checksum = kind_digit × 4 + sum over i from 1 to 8 of ( identifier_digit_i × weight_i )
check = 11 − remainder_of( checksum ÷ 11 )
if check > 9 then check = 0
valid = ( check = stated_check_digit )
```

**Worked example.** `V-12.345.678-1`. Kind `V` → kind digit 1. Identifier digits 1 2 3 4 5 6 7 8.
checksum = 1×4 + (1×3 + 2×2 + 3×7 + 4×6 + 5×5 + 6×4 + 7×3 + 8×2)
= 4 + (3 + 4 + 21 + 24 + 25 + 24 + 21 + 16) = 4 + 138 = 142.
142 ÷ 11 = 12 remainder 10. check = 11 − 10 = 1 = the stated check digit → valid.

### 8.4.8 Brazil

Either a valid natural-person number (checked by the standard library) **or** a valid legal-entity
number, checked here:

1. Clean the value of spaces, hyphens, full stops and solidi; trim; upper-case.
2. Refuse it if it starts with twelve zeros, or if it is not exactly fourteen characters.
3. Refuse it if it contains anything other than digits and upper-case letters.
4. Map each of the first twelve characters to a value by subtracting forty-eight from its character
   code. For a digit this yields the digit; for `A` it yields seventeen, for `B` eighteen, and so on.
5. First check digit:

```formula
weights_1 = ( 5 , 4 , 3 , 2 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )
d1 = remainder_of( remainder_of( ( 11 − sum over i from 1 to 12 of ( value_i × weight_1_i ) ) ÷ 11 ) ÷ 10 )
```

6. Append `d1` to the value list. Second check digit:

```formula
weights_2 = ( 6 , 5 , 4 , 3 , 2 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )
d2 = remainder_of( remainder_of( ( 11 − sum over i from 1 to 13 of ( value_i × weight_2_i ) ) ÷ 11 ) ÷ 10 )
```

7. The last two characters of the cleaned value must be `d1` followed by `d2`.

Both remainders are taken as non-negative remainders, which is what makes the doubled modulo
meaningful: the inner one maps the arithmetic into the range zero to ten and the outer one folds ten
onto zero.

### 8.4.9 Taiwan

1. Compact the value with the standard library.
2. It must be exactly eight digits.
3. Multiply each digit by the corresponding element of the multiplier
   `( 1 , 2 , 1 , 2 , 1 , 2 , 4 , 1 )`, and for each product add the **decimal digits of the
   product** to the running sum — so a product of twenty contributes two plus zero.
4. If the seventh digit is **not** `7`:

```formula
valid = ( remainder_of( digit_sum ÷ 5 ) = 0 )
```

5. If the seventh digit **is** `7`, compute the digit sum over every position except the seventh,
   and accept when either that sum, or that sum plus one, is divisible by five:

```formula
base = digit_sum_over_positions_1_to_6_and_8
valid = ( remainder_of( ( base + 1 ) ÷ 5 ) = 0 )  OR  ( remainder_of( base ÷ 5 ) = 0 )
```

The divisor is five, not ten: the national authority changed the rule when the numbering space
neared exhaustion.

### 8.4.10 Indonesia

1. Clean the value of spaces, hyphens and full stops; trim.
2. It must be fifteen or sixteen digits and entirely numeric.
3. A sixteen-digit value that does **not** begin with `0` is accepted with no check digit — the new
   numbering plan has none.
4. Otherwise validate the standard Luhn check over: the first nine digits when the value is fifteen
   digits long, or digits two to ten when the value is sixteen digits long beginning with `0`.

### 8.4.11 Mexico

The value must match: three or four letters (the tilde-n in either case is allowed, as is an
ampersand), an optional separator (space, hyphen or underscore), two digits for the year, two digits
for the month with a leading zero or one, two digits for the day with a leading zero, one, two or
three, an optional separator, and three characters from letters, digits, the tilde-n and the
ampersand — matching the whole string.

Then the embedded date must be a real calendar date:

```formula
year = 1900 + two_digit_year      when two_digit_year > 30
     = 2000 + two_digit_year      otherwise
```

and the triple (year, month, day) must be a date that exists. A number carrying the thirty-first of
February is refused.

### 8.4.12 Saudi Arabia

```formula
valid = the number matches:  the digit 3 , then 13 digits , then the digit 3
```

Fifteen digits, first and last both `3`.

### 8.4.13 India

The number must be exactly fifteen characters and must match at least one of six patterns:

| Pattern | Shape |
|---|---|
| ordinary, composite and casual | 2 digits, 5 letters, 4 digits, 1 letter, 1 character from digits one-to-nine and letters, 1 character from `Z`, `z` and the digits one to nine and the letters `A` to `J` in either case, 1 letter or digit |
| union or other body | 4 digits, 3 upper-case letters, 5 digits, one of `U` or `O`, the letter `N`, 1 upper-case letter or digit |
| revised non-resident | 4 digits, 3 upper-case letters, 5 digits, 3 upper-case letters |
| non-resident | 4 digits, 3 letters, 5 digits, the letters `N` then `R`, 1 letter or digit |
| deduction at source | 2 digits, 4 letters, 1 letter or digit, 4 digits, 1 letter, 1 character from digits one-to-nine and letters, one of `D` or `K`, 1 letter or digit |
| collection at source | 2 digits, 5 letters, 4 digits, 1 letter, 1 character from digits one-to-nine and letters, the letter `C`, 1 letter or digit |

### 8.4.14 Romania

Accepted if **any** of the three holds:

1. It matches a natural-person identifier: a digit from one to nine, two digits (the year), a month
   as `0` and one-to-nine or `1` and zero-to-two, a day as `0` and one-to-nine or `1`/`2` and any
   digit or `3` and `0`/`1`, then six digits.
2. It matches `9000` followed by nine digits.
3. It is a valid company identifier under the standard library.

### 8.4.15 Hungary

Accepted if **any** of the four holds:

1. It matches the company form: eight digits, an optional hyphen, one digit from one to five, an
   optional hyphen, two digits.
2. It matches the individual form: the digit `8` followed by nine digits.
3. It matches the union form: exactly eight digits.
4. It is valid under the standard library.

A helper also converts a local number to its union form: when the value matches the company form or
the union form, the union number is `HU` followed by the first eight digits.

### 8.4.16 Philippines

```formula
valid = length(number) is between 11 and 17 inclusive
        AND number matches:  3 digits , hyphen , 3 digits , hyphen , 3 digits ,
                             optionally ( hyphen , 3 to 5 digits ) , anchored at the end
```

### 8.4.17 Costa Rica

```formula
valid = number matches one of:
          a digit one-to-nine followed by 8 digits        (natural person, 9 digits)
          10 digits                                       (legal entity or foreign resident)
          a digit one-to-nine followed by 10 or 11 digits (migrant identifier, 11 or 12 digits)
```

### 8.4.18 Vietnam

```formula
valid = number (trimmed) matches one of:
          10 digits optionally followed by an optional hyphen and 3 digits
          12 digits
```

The ten-digit form is the enterprise identifier, the thirteen-character form adds a branch suffix,
and the twelve-digit form is the personal identifier introduced for individuals.

### 8.4.19 Ukraine

```formula
body = number with a leading "UA" removed
valid = length(body) is 8 , 10 or 12
```

Eight for the register of enterprises, ten for the natural-person taxpayer number, twelve for the
individual identifier.

### 8.4.20 Uzbekistan

The number must be entirely numeric, and then:

```formula
valid = ( length = 9 )    when the Party is an organization
      = ( length = 14 )   when the Party is a person
```

This is the only per-country check that reads a field of the Party rather than the number alone.

### 8.4.21 Morocco

```formula
valid = number is entirely numeric AND length = 8
```

### 8.4.22 Ecuador

Clean the value of spaces, hyphens and full stops, upper-case it and trim it; then:

```formula
valid = length is 10 or 13 , and every character is a decimal digit
```

### 8.4.23 Dominican Republic

Accepted when the value is valid as either the national tax number or the national identity card
number under the standard library.

### 8.4.24 Japan

A leading `T` is removed, then the standard library check is applied.

### 8.4.25 Greece

Five specific numbers are accepted unconditionally so that document-exchange testing can proceed:
`047747270`, `047747210`, `047747220`, `117747270`, `127747270`. Any other value is compacted and
checked by the standard library.

### 8.4.26 Guatemala

Two specific numbers are accepted unconditionally — `11201220K` and `11201350K` — as is any value
matching `98` followed by ten digits and the letter `K`. Any other value goes to the standard
library.

### 8.4.27 Germany

Accepted when the value is valid as either the value-added tax identifier or the national tax
number under the standard library.

### 8.4.28 Israel

The standard library's identity-number check, which is the Luhn check over nine digits.

### 8.4.29 Thailand and Turkey

Thailand uses the standard library's taxpayer identification check. Turkey accepts a value that is
valid as either the national identity number or the tax identification number.

### 8.4.30 Serbia

A leading `RS` is removed and the standard library check is applied.

### 8.4.31 Ireland

A helper computes the check character over an eight-character value, left-padded with zeros:

```formula
extra = 0                                    when the 8th character is a space or the letter W
      = 9 × ( character_code(8th) − 64 )     when the 8th character is a letter
      = invalid                              otherwise
checksum = extra + sum over i from 1 to 7 of ( ( 8 − i ) × digit_i )
check_character = the character at position remainder_of( checksum ÷ 23 ) of "WABCDEFGHIJKLMNOPQRSTUV"
```

The positions of the alphabet string are counted from zero. The validity test itself delegates to
the standard library; the helper is used by document-exchange behaviours.

## 8.5 The per-country normalisation

Normalisation runs before validation. The default is the standard library's compaction for the
country, which strips punctuation and spaces. The system defines its own for eight countries.

| Country | Normalisation |
|---|---|
| Albania | Split off the two-letter prefix, compact the body with the national-identifier compaction, and re-join. |
| Union-wide prefix `EU` | Return the value unchanged. |
| Switzerland | Prepend `CH`, run the library's formatting, then drop the two leading characters — so the punctuated form is restored. |
| Chile | Remove full stops, the country prefix, spaces and hyphens; upper-case; then, when more than two characters remain, insert a hyphen before the last character. |
| Colombia | Run the library's formatting, remove full stops and hyphens, then, when more than two characters remain, insert a hyphen before the last character. |
| Vietnam | Run the library's formatting only when the value matches the enterprise pattern; otherwise leave it alone. |
| Hungary | Compact with the library; then, if the value matches the company form, re-insert the separators as eight digits, a hyphen, one digit, a hyphen, two digits. |
| Iceland | Split off the two-letter prefix, compact the body with the national tax-number compaction, re-join. |
| San Marino | Prepend `SM`, compact with the library, then drop the two leading characters. |

**Worked example — Chile.** Input `CL 76.086.428-5` → remove full stops → `CL 76086428-5` → remove
the prefix `CL` → ` 76086428-5` → remove spaces → `76086428-5` → remove hyphens → `760864285` →
upper-case → `760864285` → more than two characters → insert a hyphen before the last → `76086428-5`.

**Worked example — Hungary.** Input `12345678111` compacts to `12345678111`; it matches the company
form (eight digits, one digit from one to five, two digits) → re-inserted as `12345678-1-11`.

## 8.6 The expected-format examples

These strings are appended to the failure message as the note. They are the exact values the system
carries.

| Country code | Example shown |
|---|---|
| `al` | `ALJ91402501L` |
| `ar` | `20055361682` |
| `at` | `ATU12345675` |
| `au` | `83 914 571 673` |
| `be` | `BE0477472701` |
| `bg` | `BG1234567892` |
| `br` | `either 11 digits for CPF or 14 characters for CNPJ` |
| `cr` | `3101012009` |
| `ch` | `CHE-123.456.788 TVA or CHE-123.456.788 MWST or CHE-123.456.788 IVA` |
| `cl` | `76086428-5` |
| `co` | `213123432-1` |
| `cy` | `CY10259033P` |
| `cz` | `CZ12345679` |
| `de` | `DE123456788 or 12/345/67890` |
| `dk` | `DK12345674` |
| `do` | `1-01-85004-3 or 101850043` |
| `ec` | `1792060346001 or 1792060346` |
| `ee` | `EE123456780` |
| `es` | `ESA12345674` |
| `fi` | `FI12345671` |
| `fr` | `FR23334175221` |
| `gb` | `GB123456782 or XI123456782` |
| `gr` | `EL123456783` |
| `hu` | `HU12345676 or 12345678-1-11 or 8071592153` |
| `hr` | `HR01234567896` |
| `id` | `1234567890123456` |
| `ie` | `IE1234567FA` |
| `il` | `XXXXXXXXX [9 digits] and it should respect the Luhn algorithm checksum` |
| `in` | `12AAAAA1234AAZA` |
| `is` | `IS062199` |
| `it` | `IT12345670017` |
| `jp` | `T7000012050002` |
| `kr` | `123-45-67890 or 1234567890` |
| `lt` | `LT123456715` |
| `lu` | `LU12345613` |
| `lv` | `LV41234567891` |
| `ma` | `12345678` |
| `mc` | `FR53000004605` |
| `mt` | `MT12345634` |
| `mx` | `GODE561231GR8` |
| `nl` | `NL123456782B90` |
| `no` | `NO123456785` |
| `nz` | `49-098-576 or 49098576` |
| `pe` | `10XXXXXXXXY or 20XXXXXXXXY or 15XXXXXXXXY or 16XXXXXXXXY or 17XXXXXXXXY` |
| `ph` | `123-456-789-123` |
| `pl` | `PL1234567883` |
| `pt` | `PT123456789` |
| `ro` | `RO1234567897 or 8001011234567 or 9000123456789` |
| `rs` | `RS101134702` |
| `ru` | `123456789047` |
| `se` | `SE123456789701` |
| `si` | `SI12345679` |
| `sk` | `SK2022749619` |
| `sm` | `SM24165` |
| `th` | `1234545678781` |
| `tr` | `11111111111 (NIN) or 2222222222 (VKN)` |
| `ua` | `12345678 or UA12345678 (EDRPOU), 1234567890 (RNOPP) or 123456789012 (IPN)` |
| `uy` | `Example: '219999830019' (format: 12 digits, all numbers, valid check digit)` |
| `uz` | `123456789 (TIN) or 12345678901234 (PINFL)` |
| `ve` | `V-12345678-1, V123456781, V-12.345.678-1` |
| `xi` | `XI123456782` |
| `sa` | `310175397400003 [Fifteen digits, first and last digits should be "3"]` |

The abbreviations inside these strings are reproduced exactly because they are the text the system
shows; they name national document types and check algorithms.

## 8.7 Worked example of the whole routine

**Input**: country `Belgium` (a member of the prefixing group), number `be 0477.472.701`, mode
`error`.

1. Neither input is empty.
2. The number is longer than one character.
3. Prefix candidate = `BE` (the first two characters upper-cased); it is alphabetic, so the prefix
   is `BE` and the body is `0477.472.701` (spaces removed — there are none inside).
4. The prefix is not `EU`.
5. The mapped code is `BE`; `BE` is in the prefixing group; the given country Belgium is also in the
   prefixing group and a prefix was present → the body alone is kept and the prefixed country is
   `BE`. The code to check is `BE`.
6. Normalisation for `BE` is the library compaction → `0477472701`.
7. The prefixed country is not `GR`.
8. The value to return is `BE` + `0477472701` = `BE0477472701`.
9. Validation is on.
10. The value does not start with `BEBE`.
11. The library check for Belgium passes.
12. Return `BE0477472701` and the code `BE`.

**Input**: country `France`, number `BE0477472701`, mode `error`.

3. Prefix `BE`, body `0477472701`.
5. The mapped code `BE` is in the prefixing group; the given country France is also in the prefixing
   group and a prefix was present → the prefixed country is `BE`, the body alone is kept, and the
   code to check is `BE`. The number is therefore validated as a Belgian number even though the
   Party's country is France, which is the intended behaviour for a French-registered Party trading
   under a Belgian registration.

**Input**: country `Switzerland` (not in the prefixing group), number `BE0477472701`, mode `error`.

5. The mapped code `BE` **is** in the prefixing group, but the given country is not → the union-wide
   retry is armed; no prefixed country is remembered; the code to check is `CH`.
6. Swiss normalisation is applied to `BE0477472701`.
11. The Swiss check fails.
12. The retry is armed → the routine restarts with the country whose code is `BE` and the number
    `BE0477472701`, which succeeds and returns `BE0477472701`.

---

# 9. Telephone numbers

## 9.1 The parse operation

**Inputs**: a number as text, and a two-letter country code that tells the parser which national
plan to assume when the number carries no international prefix.

**Steps.**

1. Parse the number once with the given region, keeping the raw input.
2. Format the parsed result in international presentation form. This step exists solely to apply
   the shipped numbering-plan corrections (§9.5), which only take effect through a format-and-reparse
   cycle.
3. Parse the formatted value a second time with the same region. The result of this second parse is
   the value the rest of the algorithm uses.
4. If either parse fails, fail with `Unable to parse <the number>: <the parser's own reason>`.
5. **Possibility test.** If the parsed number is not a possible number, look at why:
   - the country prefix is not a real one → fail with
     `Impossible number <the number>: not a valid country prefix.`
   - too short → fail with `Impossible number <the number>: not enough digits.`
   - too long → attempt two repairs before giving up:
     - if the raw input began with `00`, strip those two characters, prepend a plus sign and re-run
       the whole parse; if that also fails, fail with
       `Impossible number <the number>: too many digits.`
     - else if the raw input did not begin with a plus sign, prepend one and re-run the whole parse;
       if that also fails, fail with the same message;
     - else fail with the same message.
   - any other reason → fail with
     `The phone number <the number> is invalid! Let's fix it - you are not dialing aliens.`
6. **Validity test.** If the parsed number is possible but not valid, fail with
   `Invalid number <the number>: probably incorrect prefix.`
7. Return the parsed number.

Every message names the **original** input, not the repaired one, so that a log entry can be traced
back to what the operator typed.

## 9.2 The format operation

**Inputs**: a number as text; the two-letter country code of the assumed country; the calling code
of the assumed country; the requested output form; whether to raise on failure.

**Steps.**

1. Parse the number (§9.1). On failure: raise if asked to, otherwise return the input unchanged.
2. Choose the output form:

| Requested | Form used |
|---|---|
| `E164` | the strict international form |
| `RFC3966` | the address-bar form |
| `INTERNATIONAL` | the international presentation form |
| `NATIONAL` | the national presentation form — **but only if** the parsed number's country calling code equals the assumed country's calling code; when they differ, the international presentation form is used instead |

3. Format and return.

The override in the last row is the important rule: a number that belongs to another country is
never rendered in national form, because a national-form foreign number is undiallable.

## 9.3 Resolving which country to assume

A record formats its own number through a helper that resolves the country in this order:

1. an explicitly supplied country;
2. otherwise, when the record is a single record, the record's own country resolution:
   1. the record's country field if it has one and it is filled,
   2. otherwise the country of the first party found in any of the record's party-reference fields;
3. otherwise the acting company's country.

The number itself is taken from an explicitly supplied value, or from the named field, or from the
first non-empty field in the record's number-field list (`mobile` then `phone`).

When no number can be found, the helper returns false without raising.

## 9.4 Worked example (mandatory example: a national telephone number rendered internationally)

**Given** a Party whose country is `Belgium` (two-letter code `BE`, calling code thirty-two) and
whose telephone field holds the national spelling `02 290 34 90`.

**Formatting to the international presentation form.**

1. Parse `02 290 34 90` assuming region `BE`. Every separator is discarded, leaving the digits
   `0229034 90`, that is `022903490`. The leading zero is the Belgian national trunk prefix; the
   parser strips it, so the national significant number is the eight digits `22903490` and the
   country calling code is thirty-two.
2. Format internationally: `+32 2 290 34 90`.
3. Re-parse that value: same result.
4. It is possible and valid.
5. The requested form is the international presentation form → the output is `+32 2 290 34 90`.

**Formatting the same number to the strict international form**: `+3222903490` — a plus sign, the
calling code, the national significant number, no separators. This is the value stored in the
sanitised-number field and compared against the blocked list.

**Formatting the same number to the national presentation form**: the parsed country calling code
(thirty-two) equals the assumed country's calling code (thirty-two), so the national form is used:
`02 290 34 90`.

**Formatting the same number while assuming France** (code `FR`, calling code thirty-three): the
parser reads `02 290 34 90` as a French number; the digits form a possible and valid French
geographic number, and the output in international presentation form is `+33 2 29 03 49 0`… — in
practice the French plan requires nine significant digits after the trunk zero and this input has
eight, so the possibility test reports "too short" and the operation fails with
`Impossible number 02 290 34 90: not enough digits.` This illustrates why the assumed country
matters: the same digits are a valid Belgian number and an impossible French one.

**Formatting a number that already carries a foreign prefix while assuming Belgium.** Input
`+33 1 42 68 53 00`, assumed country `BE`, assumed calling code thirty-two, requested form
`NATIONAL`. The parsed country calling code is thirty-three, which differs from thirty-two, so the
override applies and the international presentation form is produced: `+33 1 42 68 53 00`.

**Formatting a number typed with a double-zero prefix.** Input `0032 2 290 34 90`, assumed country
`BE`. The first parse treats the leading `0032` as a trunk prefix followed by digits and the
possibility test reports "too long". The first repair applies because the raw input begins with
`00`: the two zeros are stripped and a plus sign is prepended, giving `+32 2 290 34 90`, which
parses, is possible and is valid. Output in international presentation form: `+32 2 290 34 90`.

**Formatting a number typed with no prefix at all but with a country code.** Input
`32 2 290 34 90`, assumed country `BE`. The possibility test reports "too long"; the second repair
applies because the raw input does not begin with a plus sign: a plus sign is prepended, giving
`+32 2 290 34 90`, which is valid.

## 9.5 The shipped numbering-plan corrections

Eight national numbering plans carry corrections, applied by replacing the shipped plan metadata at
start-up. Each correction exists because the underlying library's plan is out of date or too narrow.
They take effect only because the parse operation formats and re-parses (§9.1 step 2).

| Country | Correction |
|---|---|
| Ivory Coast | Revised mobile and fixed-line patterns. |
| Kenya | Revised patterns. |
| Colombia | Revised patterns. |
| Brazil | Revised patterns. |
| Morocco | Revised patterns. |
| Senegal | Revised patterns. |
| Mauritius | Revised patterns. |
| Panama | Revised patterns. |
| Israel | Revised patterns. |

A rebuild that uses a current numbering-plan library will not need these corrections; what it must
reproduce is the *behaviour* — that a number valid under the current national plan is accepted.

## 9.6 When no numbering-plan library is available

The platform degrades gracefully: parsing returns false, formatting returns the input unchanged
after logging one informational line, and the country-for-a-number lookup returns a triple of empty
strings. Every dependent behaviour therefore continues to work with unformatted numbers. A rebuild
may choose to make the library mandatory instead, but must then accept that numbers that used to be
stored unformatted are now refused.

## 9.7 Deriving a country from a number

Given a number with no assumed country:

1. Parse it with no region.
2. On failure return the triple (empty code, empty national number, empty calling code).
3. Otherwise return the region code the parser assigns to the number, the national significant
   number as text, and the country calling code as text.

---

# 10. Formatting a number for a language

## 10.1 The operation

**Inputs**: a fixed-point or integer format specification beginning with a percent sign; a value;
and a flag saying whether digit grouping is wanted.

**Steps.**

1. If the specification does not begin with a percent sign, fail with
   `format() must be given exactly one %char format specifier`.
2. Apply the specification to the value, producing a text with a full stop as the decimal mark and
   no grouping. (This is the neutral, locale-independent rendering.)
3. Look up the language's cached data. If the language is not active, fail with
   `The language <the language's name> is not installed.`
4. **If grouping is wanted:**
   1. Read the grouping specification and the thousands separator (an absent separator counts as
      the empty string).
   2. If the specification's conversion letter is one of `e`, `E`, `f`, `F`, `g`, `G` — a
      floating-point conversion — split the neutral text on the full stop; intersperse the separator
      into the first piece using the grouping specification (§10.2); then re-join the pieces with the
      language's **decimal separator**.
   3. Otherwise, if the conversion letter is one of `d`, `i`, `u` — an integer conversion —
      intersperse the separator into the whole text.
5. **If grouping is not wanted**, and the conversion letter is a floating-point one, and the neutral
   text contains a full stop, replace that full stop by the language's decimal separator. Otherwise
   leave the text as it is.
6. Return the text.

Note what step 5 does **not** do: with grouping switched off, an integer conversion is returned
completely untouched.

## 10.2 The interspersion procedure

**Inputs**: a text, a grouping specification (a list of whole numbers), a separator.

**Steps.**

1. Decompose the text with the pattern "any number of non-digit characters, then any number of
   non-space characters, then anything": the first piece is the **left part** (a sign, a currency
   symbol, an opening parenthesis), the second is the **body**, the third is the **right part**.
2. Reverse the body.
3. Split the reversed body according to the grouping specification (§10.3).
4. Reverse the list of pieces, reverse each piece, and join them with the separator.
5. Return the left part, the joined body and the right part concatenated, together with the number
   of separators inserted (one fewer than the number of pieces, or zero).

## 10.3 The split procedure

**Inputs**: a text (already reversed), a list of counts.

**Steps.**

1. Let *saved count* be the length of the text.
2. For each count in the list, in order:
   1. If the text is exhausted, stop.
   2. If the count is minus one, stop — this means "no further grouping".
   3. If the count is zero, repeat the *saved count* forever: take *saved count* characters, emit
      them, and continue until the text is exhausted; then stop.
   4. Otherwise take that many characters, emit them, set *saved count* to that count, and continue
      with the rest.
3. If any text remains, emit it as a final piece.

So `[3,0]` means "three, then three forever"; `[3,2,0]` means "three, then two, then two forever";
`[2,-1,3]` means "two, then stop" — the rest becomes one final piece.

**Worked trace** for the reversed body `7654321` and the grouping `[3,0]`:

- count 3 → emit `765`, remainder `4321`, saved count 3;
- count 0 → repeat 3 forever: emit `432`, remainder `1`; emit `1`, remainder empty; stop.
- Pieces: `765`, `432`, `1`.

Reverse the list: `1`, `432`, `765`. Reverse each piece: `1`, `234`, `567`. Join with the separator.

## 10.4 Worked example (mandatory example: a number formatted in a language that groups by three and uses a comma decimal mark)

**Language**: German, locale code `de_DE` — grouping `[3,0]`, decimal separator the comma,
thousands separator the full stop.

**Specification** `%.2f`, **value** one million two hundred and thirty-four thousand five hundred
and sixty-seven point eight nine one, **grouping wanted**.

1. The specification begins with a percent sign.
2. Neutral rendering: `1234567.89` (the fixed-point conversion rounds the third decimal away).
3. The language is active.
4. Grouping is wanted; the conversion letter is `f`, a floating-point one:
   1. Split on the full stop → `1234567` and `89`.
   2. Intersperse the first piece with grouping `[3,0]` and separator `.`:
      - decompose: left part empty, body `1234567`, right part empty;
      - reverse the body → `7654321`;
      - split → `765`, `432`, `1`;
      - reverse the list and each piece → `1`, `234`, `567`;
      - join with `.` → `1.234.567`.
   3. Re-join with the decimal separator `,` → `1.234.567,89`.
5. Result: **`1.234.567,89`**.

**The same value in English (United States)** — grouping `[3,0]`, decimal separator the full stop,
thousands separator the comma: `1,234,567.89`.

**The same value in French** — grouping `[3,0]`, decimal separator the comma, thousands separator a
non-breaking space: `1 234 567,89`, where each gap is one non-breaking space character.

**The same value in Hindi** — grouping `[3,2,0]`, decimal separator the full stop, thousands
separator the comma. Value twelve million three hundred and forty-five thousand six hundred and
seventy-eight point nine:

- neutral rendering `12345678.90`;
- body `12345678` reversed → `87654321`;
- split with `[3,2,0]`: count 3 → `876`, remainder `54321`, saved 3; count 2 → `54`, remainder
  `321`, saved 2; count 0 → repeat 2 forever: `32`, then `1`;
- pieces `876`, `54`, `32`, `1`; reversed list `1`, `32`, `54`, `876`; each reversed `1`, `23`,
  `45`, `678`;
- joined with `,` → `1,23,45,678`;
- re-joined with the decimal separator → **`1,23,45,678.90`**.

**A negative value in German**, minus one million two hundred and thirty-four thousand five hundred
and sixty-seven point eight nine one: the neutral rendering is `-1234567.89`; the decomposition
gives left part `-`, body `1234567`, right part empty; the body is interspersed exactly as before
and the left part is put back → **`-1.234.567,89`**.

**A small value in German**, twelve point five: neutral rendering `12.50`; the body `12` reversed is
`21`; count 3 takes all two characters → one piece `21`; reversed → `12`; no separator inserted;
re-joined → **`12,50`**.

**An integer conversion in German**, specification `%d`, value one million two hundred and thirty-
four thousand five hundred and sixty-seven, grouping wanted: neutral rendering `1234567`; the
conversion letter is `d` so the whole text is interspersed → **`1.234.567`**. No decimal separator
appears.

**The same floating-point value in German with grouping switched off**: step 5 applies; the neutral
text `1234567.89` has its full stop replaced by the comma → **`1234567,89`**.

## 10.5 Where the formatting is used

The operation is the primitive underneath every rendering of a monetary amount, a quantity or a
percentage in a printed document, in a message body and in an export. Domains that render amounts
add the currency symbol before or after according to the currency's symbol position, and choose the
number of decimal places from the currency's decimal places; the grouping and the separators always
come from the reader's language, never from the currency.

---

# 11. Merging parties

## 11.1 Overview

The merge takes two or three parties, chooses one as the destination, rewrites every reference in
the database from the sources to the destination, merges the field values into the destination, and
deletes the sources.

## 11.2 Choosing the destination

```formula
ordering_key(P) = ( P is not active , creation_timestamp(P) )
```

with a missing creation timestamp replaced by the first instant of the first of January nineteen
seventy. The parties are sorted by this key **descending**. Because "is not active" is false for an
active Party and true for an archived one, descending order puts **archived parties first** and,
within each activity group, **newest first**. The destination is the **last** element — that is, the
**oldest active** Party. When every Party is archived, it is the oldest archived one.

**Worked example.** Three parties:

| Party | Active | Created |
|---|---|---|
| A | yes | 2023-04-11 |
| B | yes | 2021-09-02 |
| C | no | 2024-01-30 |

Keys: A → (false, 2023-04-11); B → (false, 2021-09-02); C → (true, 2024-01-30). Sorted descending:
C (true beats false), then A (2023 beats 2021), then B. The destination is **B**, the oldest active
Party. The sources are C and A.

When the caller supplies a destination explicitly and it is among the parties, that one is used and
the others become the sources.

## 11.3 The safety checks

Run in this order, before anything is modified.

1. If the acting user is an administrator, the "extra checks" flag is switched off, which disables
   check 5 only.
2. Non-existent identifiers are dropped. If fewer than two parties remain, the merge returns
   silently — it is not an error to ask to merge one Party.
3. If more than three parties remain, fail with
   `For safety reasons, you cannot merge more than 3 contacts together. You can re-open the wizard several times if needed.`
4. Build the set of all descendants of all the parties, excluding the parties themselves. If any of
   the parties to merge is a descendant of another, fail with
   `You cannot merge a contact with one of his parent.`
5. Collect every user account attached to any of the parties, including archived accounts. If there
   is more than one, fail with
   `You cannot merge contacts linked to more than one user even if only one is active.`
6. If the extra checks are on, collect the distinct electronic mail addresses of the parties. If
   there is more than one distinct value, fail with
   `All contacts must have the same email. Only the Administrator can merge contacts with different emails.`

## 11.4 Aligning the users' company

If the destination has a company, every user account attached to **any** of the parties has that
company linked into its allowed companies and set as its own company, with elevated rights. This
must happen before the references are rewritten, because otherwise the company consistency check on
the Party would refuse the later writes.

## 11.5 Merging the bank accounts

For each bank account of the sources:

1. Look for an account of the **destination** whose sanitised number is the same.
2. If one exists: rewrite every foreign key and every polymorphic reference from the source account
   to the destination account (using the same two passes as §11.6 and §11.7 but on the bank-account
   entity), then delete the source account with elevated rights.
3. Otherwise: move the source account to the destination by writing its holder.

This runs **before** the Party references are rewritten, so that the accounts are already in their
final place when the Party rows disappear.

## 11.6 Rewriting the foreign keys

1. Ask the database for every (table, column) pair where the column is the first column of a foreign
   key constraint pointing at the Party table, in the current schema.
2. Invalidate the whole in-memory cache, so that nothing stale survives the direct manipulation.
3. For each such (table, column), skipping any table whose name contains `base_partner_merge_`:
   1. List the table's columns except the foreign-key column; call the first of them the *witness
      column*.
   2. If no row of the table points at any source, skip the table.
   3. **If the table has at most one other column** — it is a join table — update, for each source
      separately, every row pointing at the source to point at the destination, **but only where no
      row already exists that points at the destination and has the same witness-column value**.
      Rows that would collide are simply left behind pointing at a Party that is about to be deleted,
      and are removed by the database's own cascade when the source is deleted.
   4. **Else, if the table has no check constraint and no uniqueness constraint involving the
      column**, update every row pointing at any source to point at the destination in one
      statement.
   5. **Else**, attempt the same single statement inside a save point with logging muted. If the
      database refuses — almost always a uniqueness violation — roll the save point back and
      **delete** every row pointing at any source instead. The justification recorded in the code is
      that a row whose Party no longer exists is useless.

**Worked consequence.** A Journal Item (`account.move.line`) has a Party column and many other
columns; its table carries check constraints, so branch 3.5 applies: the update is attempted, and
because no uniqueness constraint involves the Party column it succeeds. Every posted accounting
entry of the sources now points at the destination.

**Worked consequence.** The Party-to-Tag join table has exactly two columns; branch 3.3 applies: a
source's tag link is moved to the destination unless the destination already carries that tag, in
which case the duplicate link is left behind and vanishes with the source row.

## 11.7 Rewriting the polymorphic references

Records that point at *any* entity by a pair of (entity name, record identifier) columns must be
rewritten too.

1. For each source, and for each of the five well-known holders:

| Holder | Entity-name column | Identifier column |
|---|---|---|
| Attachments (`ir.attachment`) | `res_model` | `res_id` |
| Followers (`mail.followers`) | `res_model` | `res_id` |
| Activities (`mail.activity`) | `res_model` | `res_id` |
| Messages (`mail.message`) | `model` | `res_id` |
| External identifiers (`ir.model.data`) | `model` | `res_id` |

   search for the rows naming the Party entity and the source identifier; if the identifier column
   carries no check or uniqueness constraint, write the destination identifier straight away and
   flush; otherwise attempt the write inside a save point and, on refusal, delete the rows.
2. The same treatment is applied to an additional holder list. For the Party merge that list names
   the calendar entity with the entity-name column `model_id.model`; the entity is usually absent
   from the registry, in which case the step does nothing.
3. For every **stored reference-typed field** in the whole registry whose entity is concrete and
   whose field is not computed: for each source, find the records whose field value is the text
   "the Party entity name, a comma, the source identifier" and write the corresponding text for the
   destination.
4. For every **company-dependent link field** anywhere in the registry that points at the Party:
   rewrite the per-company map in place, replacing any value equal to a source identifier with the
   destination identifier.
5. In the stored default values, for every company-dependent link field whose target is the Party
   entity and whose stored value is a plain number, replace a source identifier with the
   destination identifier.
6. Flush.
7. For every **company-dependent field of the Party itself** — the barcode in the foundation package
   — merge the per-company maps: aggregate the sources' maps in ascending identifier order, so that
   a later source wins a conflict between sources; then combine that aggregate with the
   destination's own map so that the **destination's own entries win**; and write the result on the
   destination.

## 11.8 Merging the field values

1. Take every field of the Party.
2. Skip every field whose copy flag is cleared — in the foundation package, the barcode.
3. For every remaining field that is **not** a list-valued field (neither many-to-many nor a reverse
   list) and is **not computed**:
   - walk the sources in order and then the destination last;
   - whenever the field's value on the walked record is non-empty, remember it, overwriting any
     previously remembered value.

   Because the destination comes last, **the destination's own non-empty value always wins**; when
   the destination's value is empty, the **last source with a non-empty value** wins. Sources are
   walked in the record set's order, which is ascending identifier.
4. A reference-typed field is remembered as the whole reference, not as an identifier.
5. A field that the calling behaviour has declared **summable** and that is not company-dependent is
   accumulated instead of overwritten. The foundation package declares no summable fields; other
   domains declare counters.
6. A field that is company-dependent **and** summable is summed per company, over the sources and
   the destination, with elevated rights so that companies the user cannot see are included.
7. Remove the internal identifier from the value map.
8. Remove the parent from the value map and keep it aside.
9. Write the value map onto the destination.
10. Write the per-company sums onto the destination, once per company, with elevated rights.
11. If a parent was kept aside and it is not the destination itself, try to write it. If the write
    fails the cycle validation, skip it silently and log an informational line.

## 11.9 Finishing

1. Mark the destination's shared-party flag for recomputation.
2. Log the operation: the acting user, the source identifiers and the destination identifier.
3. Delete the sources with elevated rights.

## 11.10 Worked example (mandatory example: a merge of two parties referenced by invoices and orders)

### Before

| | Party A | Party B |
|---|---|---|
| internal identifier | 501 | 502 |
| name | `Deco Addict` | `Deco Addict SA` |
| active | yes | yes |
| created | 2021-03-04 | 2023-08-19 |
| electronic mail address | `contact@deco.example` | `contact@deco.example` |
| telephone | empty | `+32 2 290 34 90` |
| street | `Chaussée de Namur 40` | empty |
| city | `Ramillies` | `Ramillies` |
| tax registration number | empty | `BE0477472701` |
| reference | `C0042` | `C0087` |
| language | `fr_FR` | `en_US` |
| tags | `Prospect` | `Prospect`, `Wholesaler` |
| bank accounts | `BE71 0961 2345 6769` | `BE71 0961 2345 6769`, `BE68 5390 0754 7034` |
| parent | none | none |
| user accounts | none | none |

Other records:

- Customer invoice `INV/2022/00013`, posted, with its Party column pointing at 501 and three Journal
  Items also pointing at 501.
- Customer invoice `INV/2024/00007`, posted, pointing at 502, with two Journal Items pointing at 502.
- Sales order `S00021`, confirmed, pointing at 501, with its invoicing address and shipping address
  columns also pointing at 501.
- Sales order `S00038`, confirmed, pointing at 502.
- Twelve messages in the message history of 501 and four in that of 502.
- Two attachments on 501.

### The merge

**Destination.** Keys: A → (false, 2021-03-04); B → (false, 2023-08-19). Sorted descending: B first
(2023 beats 2021), then A. The last element is **A**, identifier 501. So the destination is
`Deco Addict` and the source is `Deco Addict SA`.

**Safety checks.** Two parties — within the limit of three. Neither is a descendant of the other. No
user accounts at all, so the one-user rule passes. Both electronic mail addresses are
`contact@deco.example`, so the same-address rule passes even for a non-administrator.

**User companies.** The destination has no company, so nothing is done.

**Bank accounts.** The source holds two.

- `BE71 0961 2345 6769` sanitises to `BE71096123456769`; the destination already holds an account
  with that sanitised number → every reference to the source's account is rewritten onto the
  destination's account, and the source's account row is deleted.
- `BE68 5390 0754 7034` has no twin → it is moved to the destination by writing its holder.

The destination now holds both accounts. Note that the moved account keeps its own outgoing-payment
permission, its sequence and its notes.

**Foreign keys.** Every table with a foreign key to the Party table is visited:

- the invoice table (`account_move`): many columns, has check constraints → branch 3.5; the single
  update succeeds; `INV/2024/00007` now points at 501.
- the Journal Item table (`account_move_line`): same branch; the two items of `INV/2024/00007` now
  point at 501.
- the sales order table (`sale_order`): its Party column, its invoicing-address column and its
  shipping-address column are three separate foreign keys and are each visited in turn; `S00038`
  now points at 501 in all three.
- the Party-to-Tag join table: exactly two columns → branch 3.3. For the tag `Prospect`, a row
  already exists for the destination with the same witness value, so the source's row is **not**
  moved and is left to disappear with the source. For the tag `Wholesaler`, no such row exists, so
  it is moved. The destination ends with both tags.
- the Party table's own parent column: a self-referencing foreign key. No row points at 502 as a
  parent, so the table is skipped at step 3.2.

**Polymorphic references.** The four messages of 502 are moved to 501, as are its followers and its
activities; the destination's message history now holds sixteen messages. No attachments were on
502. External identifiers naming 502 are rewritten. No stored reference-typed field in the registry
points at 502. No company-dependent link field points at 502.

**Company-dependent fields of the Party.** The barcode is company-dependent. Party 502's map is
`{company 1: "DA-502"}` and party 501's is empty. The aggregate of the sources is
`{company 1: "DA-502"}`; combining it with the destination's empty map leaves
`{company 1: "DA-502"}`, which is written on 501. Had 501 already carried a barcode for company 1,
its own value would have won.

**Field values.** The walk is: source 502, then destination 501.

| Field | 502 | 501 | Remembered | Reason |
|---|---|---|---|---|
| name | `Deco Addict SA` | `Deco Addict` | `Deco Addict` | destination non-empty, walked last |
| electronic mail address | `contact@deco.example` | `contact@deco.example` | `contact@deco.example` | both non-empty |
| telephone | `+32 2 290 34 90` | empty | `+32 2 290 34 90` | destination empty, source wins |
| street | empty | `Chaussée de Namur 40` | `Chaussée de Namur 40` | destination non-empty |
| city | `Ramillies` | `Ramillies` | `Ramillies` | |
| tax registration number | `BE0477472701` | empty | `BE0477472701` | destination empty, source wins |
| reference | `C0087` | `C0042` | `C0042` | destination non-empty |
| language | `en_US` | `fr_FR` | `fr_FR` | destination non-empty |
| active | true | true | true | |
| barcode | *(skipped: copy flag cleared)* | | | handled by the company-dependent pass |
| complete name | *(skipped: computed)* | | | recomputed after the write |
| commercial entity | *(skipped: computed)* | | | recomputed after the write |
| tags | *(skipped: list-valued)* | | | handled by the foreign-key pass |
| bank accounts | *(skipped: list-valued)* | | | handled by the bank-account pass |
| parent | none | none | none | nothing to set aside |

The value map is written onto 501. The stored complete name is recomputed from the surviving name
and remains `Deco Addict`.

**Finishing.** The shared-party flag is queued for recomputation; the operation is logged; party 502
is deleted.

### After

| Field | Party 501 |
|---|---|
| name | `Deco Addict` |
| electronic mail address | `contact@deco.example` |
| telephone | `+32 2 290 34 90` |
| street | `Chaussée de Namur 40` |
| city | `Ramillies` |
| tax registration number | `BE0477472701` |
| reference | `C0042` |
| language | `fr_FR` |
| barcode (company 1) | `DA-502` |
| tags | `Prospect`, `Wholesaler` |
| bank accounts | `BE71 0961 2345 6769`, `BE68 5390 0754 7034` |
| message history | sixteen messages |

And the documents:

| Document | Party before | Party after |
|---|---|---|
| `INV/2022/00013` and its three Journal Items | 501 | 501 |
| `INV/2024/00007` and its two Journal Items | 502 | **501** |
| `S00021` (three Party columns) | 501 | 501 |
| `S00038` (three Party columns) | 502 | **501** |

The accounting balance of party 501 is now the sum of the two former balances, and both sales
orders appear on one Party. No accounting entry was created, modified in amount, or reversed: only
the Party column changed. See [accounting-effects.md](accounting-effects.md) §3.

## 11.11 Finding candidate duplicates

**The grouping query.** Given a list of field names and a maximum number of groups:

1. Build the list of grouping expressions: the electronic mail address and the name are grouped by
   their lower-cased value; the tax registration number is grouped by its value with every space
   removed; every other field is grouped by its raw value.
2. Build the filter: the electronic mail address, the name and the tax registration number, when
   selected, each contribute the condition "this column is not empty". Other selected fields
   contribute nothing.
3. The query selects the lowest identifier and the array of identifiers, from the Party table, with
   the filter when there is one, grouped by the grouping expressions, keeping only groups of two or
   more rows, ordered by the lowest identifier, limited to the maximum number of groups when one is
   given.

**Processing the result.** For each group returned:

1. Re-read the parties through the ordinary access path, so that parties the operator may not see
   are dropped.
2. If fewer than two survive, skip the group.
3. If any exclusion filter is on and any Party of the group is used by the corresponding entity,
   skip the group. The filters are: "has a user account" → the user entity through its Party
   column; "has accounting entries" → the Journal Item entity through its Party column, offered only
   when the accounting behaviour is installed.
4. Otherwise create a Merge Group holding the lowest identifier and the surviving identifiers.

Finally the wizard's state becomes `selection` and the group count is recorded.

If no grouping criterion was selected at all, the operation fails with
`You have to specify a filter for your selection.`

**Worked example.** Grouping by the electronic mail address and by the name, with a maximum of one
hundred groups. The query groups the Party table by the lower-cased address and the lower-cased
name, keeping only groups with two or more rows and only rows where both columns are non-empty. Two
parties named `Deco Addict` and `deco addict`, both with the address `contact@deco.example`, fall
in the same group; a third named `Deco Addict` with no address does not, because the filter excludes
rows with an empty address.

## 11.12 The parent-migration pass

A separate, one-off procedure that merges a Party with its own parent when they are plainly the same
entity:

1. Run a query that joins the Party table to itself on equal electronic mail addresses **and** equal
   names **and** a parent relationship in either direction, grouping by the address, the name and
   the identifier of whichever of the two is the parent, keeping groups of two or more, ordered by
   the lowest identifier.
2. Process the result exactly as in §11.11.
3. Merge every group, deleting each group as it is treated and committing after each one.
4. Set the wizard's state to `finished`.
5. Finally, repair any row that ended up being its own parent by clearing both its organization flag
   and its parent.

Note that step 4 of §11.3 — the "cannot merge a contact with its parent" check — would refuse every
one of these groups. The pass is therefore only meaningful where that check does not bite, which is
when the two rows are the same record after the earlier passes.

## 11.13 The full-cleanup pass

1. Run the parent-migration pass.
2. Create a **new** wizard configured to group by the tax registration number, the electronic mail
   address and the name, and run its automatic process end to end.
3. Clear the organization flag on every row that has a parent and an organization flag set.
4. Return the next-screen action.

---

# 12. Generating an initials avatar

## 12.1 The colour

**Inputs**: the Party's name; the Party's creation timestamp.

```formula
seed = name + creation_timestamp_as_text
```

where the creation timestamp is expressed as seconds since the first instant of the first of
January nineteen seventy, written with its fractional part, and replaced by the empty string when
the record has no creation timestamp.

```formula
digest = hexadecimal_text_of( SHA-512( seed encoded as bytes ) )

hue        = value_of_hex_pair( digest characters 1 and 2 ) × 360 ÷ 255
saturation = value_of_hex_pair( digest characters 3 and 4 ) × 30 ÷ 255 + 40
lightness  = 45
```

The saturation formula is written in the source as a multiplication by the difference between
seventy and forty divided by two hundred and fifty-five, then an addition of forty — so saturation
ranges over forty to seventy per cent. Lightness is fixed at forty-five per cent so that white text
is always legible on the result.

The colour is then written as text:

```formula
colour_text = "hsl(" + round_to_whole(hue) + ", " + round_to_whole(saturation) + "%, 45%)"
```

with both numbers rendered with no decimal places. The rounding is to the nearest whole number with
exact halves going to the even neighbour.

## 12.2 The image

```
<?xml version="1.0" encoding="UTF-8" ?><svg height="180" width="180" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><rect fill="COLOUR" height="180" width="180"/><text fill="#ffffff" font-size="96" text-anchor="middle" x="90" y="125" font-family="sans-serif">INITIAL</text></svg>
```

with `COLOUR` replaced by the colour text and `INITIAL` replaced by the initial. The whole document
is a single line with no whitespace between elements, and the result is encoded for transport with
the standard base-sixty-four encoding.

```formula
initial = escape_for_markup( uppercase( first_character_of( name ) ) )
```

The escaping turns the ampersand, the less-than sign, the greater-than sign, the double quotation
mark and the single quotation mark into their markup entities, so a Party named `&Co` yields the
initial `&amp;`.

## 12.3 Worked example (mandatory example: an initials avatar)

**Party.** Name `Deco Addict`; created on the fifteenth of March two thousand and twenty-four at
nine hours thirty minutes zero seconds, Coordinated Universal Time.

1. The creation timestamp in seconds is one billion seven hundred and ten million four hundred and
   ninety-five thousand, written with its fractional part as `1710495000.0`.
2. The seed is `Deco Addict1710495000.0`.
3. The digest of that seed begins with the hexadecimal characters `4f1f`.
4. The first pair `4f` has the value seventy-nine.
   hue = 79 × 360 ÷ 255 = 28440 ÷ 255 = 111.5294117647…
   Rounded to a whole number: **112**.
5. The second pair `1f` has the value thirty-one.
   saturation = 31 × 30 ÷ 255 + 40 = 930 ÷ 255 + 40 = 3.647058823… + 40 = 43.647058823…
   Rounded to a whole number: **44**.
6. Lightness is **45**.
7. The colour text is `hsl(112, 44%, 45%)` — a mid-green.
8. The initial is the upper-cased first character of `Deco Addict`, that is `D`; it needs no
   escaping.
9. The image document is:

```
<?xml version="1.0" encoding="UTF-8" ?><svg height="180" width="180" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><rect fill="hsl(112, 44%, 45%)" height="180" width="180"/><text fill="#ffffff" font-size="96" text-anchor="middle" x="90" y="125" font-family="sans-serif">D</text></svg>
```

10. That document is encoded for transport; the encoding begins
    `PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiID8+PHN2ZyBoZWlnaHQ9IjE4MCIgd2lk…`.

**Why the timestamp is in the seed.** Two parties with the same name created at different moments
get different colours, so a list of three people all called `Smith` is still visually
distinguishable. Conversely the colour of one Party never changes, because its creation timestamp
never changes.

**A Party with no creation timestamp** — a record being previewed before it is saved — seeds on the
name alone. For `Deco Addict` alone the seed is `Deco Addict` and the resulting colour differs from
the saved record's colour. This is visible in practice as a colour that changes once when a new
Party is saved.

## 12.4 When the generated avatar is *not* used

See [entities.md](entities.md) §1.11. In short: the generated avatar is produced only for a Party
that has a name, has no stored image, and either has a non-shared user account or is of address type
`contact`. Every other imageless Party gets one of the five shipped placeholder pictures.

---

# 13. Decomposing and recomposing a street

Supplied by the extended-address behaviour; also available as a helper in the foundation package.

## 13.1 The grammar

Two patterns are tried **in order**. The first that matches and produces a non-empty group map wins.
Both are applied from the beginning of the text, both treat the full stop as matching a line feed,
and both ignore whitespace and comments inside the pattern itself.

**Pattern one — the number comes first.** Anchored at both ends.

1. The **house number**: a digit, then any number of letters, digits, solidi and hyphens.
2. One or more characters from {comma, whitespace}.
3. The **street name**: anything, matched as little as possible.
4. Optionally: any whitespace, one character from the range comma-to-solidus (that is, one of the
   comma, the hyphen, the full stop or the solidus), any whitespace, then the **door number** — one
   or more characters, required to contain at least one digit somewhere, and forbidden from starting
   a run that looks like a comma surrounded by optional spaces or a hyphen surrounded by at least
   one space on each side.
5. End of text.

**Pattern two — the number comes between the name and the door.** Anchored at the start only.

1. The **street name**: anything, matched as little as possible, then any whitespace.
2. Optionally a **comma** — remembered, because it changes what follows.
3. Any whitespace.
4. Optionally the **house number**: a digit, then any number of letters, digits, solidi and hyphens.
5. A conditional: **if** the comma was captured, nothing more is required here; **otherwise** the
   text at this point must be followed by either whitespace and one of the comma, hyphen, full stop
   or solidus, or by optional whitespace and the end of the text. (This is a look-ahead: it consumes
   nothing.)
6. Optionally the **door number**, exactly as in pattern one, step 4.

The three results are trimmed of surrounding whitespace; a group that did not participate becomes
the empty string. When neither pattern matches, all three results are empty.

## 13.2 The recomposition

```formula
street = trim( street_name + " " + house_number )
street = street + " - " + door_number        when the door number is non-empty
```

The recomposition is **not** the inverse of the decomposition: it always produces the
"name number - door" shape, whatever shape the input had.

## 13.3 Worked decomposition and recomposition table

| Input street | Pattern that matched | Street name | House number | Door number | Street after recomposition |
|---|---|---|---|---|---|
| *(empty)* | two | *(empty)* | *(empty)* | *(empty)* | *(empty)* |
| `Place Royale` | two | `Place Royale` | *(empty)* | *(empty)* | `Place Royale` |
| `Chaussee de Namur 40a - 2b` | two | `Chaussee de Namur` | `40a` | `2b` | `Chaussee de Namur 40a - 2b` |
| `Chaussee de Namur 1` | two | `Chaussee de Namur` | `1` | *(empty)* | `Chaussee de Namur 1` |
| `40 Chaussee de Namur` | **one** | `Chaussee de Namur` | `40` | *(empty)* | `Chaussee de Namur 40` |
| `Chaussee de Namur, 40 - Apt 2b` | two | `Chaussee de Namur` | `40` | `Apt 2b` | `Chaussee de Namur 40 - Apt 2b` |
| `header Chaussee de Namur, 40 trailer ` | two | `header Chaussee de Namur` | `40` | *(empty)* | `header Chaussee de Namur 40` |
| `Cl 53` then a line feed then ` # 43 - 81`, the whole preceded by a line feed | two | `Cl 53`, a line feed, ` #` | `43` | `81` | `Cl 53`, a line feed, ` # 43 - 81` |
| `Street Line 1` then a line feed then `Number Line 2 44 76` | two | `Street Line 1`, a line feed, `Number Line 2 44` | `76` | *(empty)* | the same text |
| `1600 Pennsylvania Ave NW, Apt 4B` | **one** | `Pennsylvania Ave NW` | `1600` | `Apt 4B` | `Pennsylvania Ave NW 1600 - Apt 4B` |
| `10, Rue de la Paix` | **one** | `Rue de la Paix` | `10` | *(empty)* | `Rue de la Paix 10` |
| `Calle Gran Vía, 42, 3º Dcha` | two | `Calle Gran Vía` | `42` | `3º Dcha` | `Calle Gran Vía 42 - 3º Dcha` |
| `Jean-Baptiste-Lebas 12 - A-3` | two | `Jean-Baptiste-Lebas` | `12` | `A-3` | `Jean-Baptiste-Lebas 12 - A-3` |
| `Jean-Baptiste-Lebas, 12 / A-3` | two | `Jean-Baptiste-Lebas` | `12` | `A-3` | `Jean-Baptiste-Lebas 12 - A-3` |
| `1-7-1 Nagatacho, Chiyoda-ku, Apt 3` | **one** | `Nagatacho, Chiyoda-ku` | `1-7-1` | `Apt 3` | `Nagatacho, Chiyoda-ku 1-7-1 - Apt 3` |

Three observations a rebuild must reproduce:

- The recomposition normalises: `10, Rue de la Paix` becomes `Rue de la Paix 10`, and
  `Jean-Baptiste-Lebas, 12 / A-3` becomes `Jean-Baptiste-Lebas 12 - A-3`. Writing the three parts
  therefore rewrites the street even when nothing changed.
- The door number must contain at least one digit: in `header Chaussee de Namur, 40 trailer ` the
  word `trailer` cannot be a door number, so it is absorbed into nothing and silently lost. The
  recomposition yields `header Chaussee de Namur 40`, and the trailing word is gone.
- A house number may contain hyphens and solidi, which is how the Japanese block-lot-house form
  `1-7-1` is captured whole.

## 13.4 The write direction

Writing any of the three parts recomputes the street through §13.2. Writing the street recomputes
all three parts through §13.1. The two computations are wired as a computed-and-stored field with
an inverse, so a single write of either side is enough.

---

# 14. Small derived values

## 14.1 The time-zone offset

```formula
offset_text = the current offset of the Party's time zone from Coordinated Universal Time,
              rendered as a sign, two digits of hours and two digits of minutes
```

with the zone `GMT` used when the Party has no time zone, giving `+0000`. The value depends on the
moment at which it is computed, because it accounts for daylight saving.

**Worked example.** A Party whose time zone is `Europe/Brussels`, read on the fifteenth of March:
Central European Time is one hour ahead, so the offset is `+0100`. Read on the fifteenth of July,
Central European Summer Time is two hours ahead, so the offset is `+0200`.

## 14.2 The company colour index

```formula
colour_index = colour_index_of_the_root_company's_party      when that value is non-zero
             = remainder_of( root_company_identifier ÷ 12 )   otherwise
```

**Worked example.** A root company with identifier seven whose Party's colour index is zero yields
a colour index of seven. A root company with identifier fourteen whose Party's colour index is zero
yields two. Writing the colour index on any branch writes it on the root company's Party.

## 14.3 The bank-account colour

```formula
colour = 10    when outgoing payments are allowed
       = 1     otherwise
```

## 14.4 The geocoding query string

```formula
query = join_with_", "( the non-empty elements of [ street , trim(zip + " " + city) , state , country ] )
```

**Worked example.** Street `Chaussée de Namur 40`, postal code `1367`, city `Ramillies`, no state,
country `Belgium` → the second element is `1367 Ramillies`, the state is dropped as empty, and the
query is `Chaussée de Namur 40, 1367 Ramillies, Belgium`.

For the second provider, a country name containing a comma and ending in ` of` or ` of the` is
rewritten first: `Congo, Democratic Republic of the` becomes `Democratic Republic of the Congo`.

## 14.5 The reverse-geocoding description

Given a latitude and a longitude, the description is built as follows.

1. Take the city name and the country code from the request's own geographic lookup, if the request
   carries one.
2. If either is missing, call the second provider's reverse lookup and take from its address block:
   the country code; the city, preferring in order the city district, the town, the village and the
   city; and the postal code.
3. Resolve the country from the upper-cased country code.
4. Assemble: start with the postal code or the empty string; append the city, preceded by a space if
   anything is already there; append the country name, preceded by a comma and a space if anything
   is already there.
5. If the result is empty, return the word `Unknown`.

**Worked example.** Postal code `1367`, city `Ramillies`, country `Belgium` → `1367 Ramillies,
Belgium`. With no postal code → `Ramillies, Belgium`. With nothing at all → `Unknown`.

---

# 15. Duplicate detection queries

## 15.1 The tax-registration twin

Computed on every Party, never stored, depending on the tax registration number, the company, the
company registration number and the country.

**Algorithm.**

1. Let *identifier* be the record's own identifier, taken from the persisted record so that an
   unsaved edit compares against the saved state.
2. Search with elevated rights and **with archived records included** — deliberately, so that an
   operator is told to reactivate an archived duplicate rather than create a third copy.
3. Build the list of numbers to look for:
   1. the record's own tax registration number;
   2. the check is skipped entirely when the number is empty or exactly one character long;
   3. if the record has a country and that country is in the prefixing group:
      - if the number's first two characters are alphabetic, also look for the number **without**
        those two characters;
      - otherwise also look for the country code followed by the number, and, when the country code
        has a special tax prefix (`GR` → `EL`, `GB` → `XI`), also the special prefix followed by the
        number.
4. The filter is: the tax registration number is one of the numbers to look for; **and**, when the
   record has a country, the candidate's country is that country or empty; **and**, when the record
   has a company, the candidate's company is that company or empty; **and**, when the record is
   persisted, the candidate is neither the record itself nor any of its descendants.
5. The twin is the first match — but only when the number is worth checking **and the record has no
   parent**. A child never reports a twin, because its number is inherited.

**Worked example.** A Party with country `Greece` and tax registration number `123456783`. Greece is
in the prefixing group and the number's first two characters are not alphabetic, so the numbers to
look for are `123456783`, `GR123456783` and `EL123456783`. A second Party stored with `EL123456783`
is therefore detected.

**Worked example.** A Party with country `Belgium` and number `BE0477472701`. The first two
characters are alphabetic, so the numbers to look for are `BE0477472701` and `0477472701`. A second
Party stored with the bare `0477472701` is detected.

## 15.2 The company-registration twin

1. The filter is: the company registration number equals the record's; **and** the candidate's
   company is the record's company or empty; **and**, when the record is persisted, the candidate is
   neither the record itself nor any of its descendants.
2. The twin is the first match, but only when the registration number is non-empty **and the record
   has no parent**.

Note the asymmetry with §15.1: the registration twin does not filter on the country, even though the
field's own description says the number must be unique across all parties of the same country.

## 15.3 Where the twins are shown

Both are shown as warnings on the Party form, never as refusals. The system does not prevent two
parties from sharing a tax registration number; it tells the operator and lets them decide, because
legitimate cases exist — a branch registered under the head office's number, or a data set being
migrated in stages.
