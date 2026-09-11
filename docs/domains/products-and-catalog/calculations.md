# Products and Catalog — Calculations and Algorithms

Every formula and every algorithm of the domain, with its rounding rule, its precision, its
currency handling, its unit handling, its date handling, its evaluation order, and at least one
worked numeric example. Read the sections in order: later sections use definitions introduced
earlier.

Notation used in the `formula` blocks:

- Named quantities are written in words with underscores, for example `template_sales_price`.
- The four arithmetic symbols are `+`, `−`, `×` and `÷`, and `=` is assignment or equality.
- `round_to(value, precision)` means "round `value` to `precision` decimal places, half away from
  zero".
- `round_to_currency(value, currency)` means "round `value` to the rounding multiple of `currency`,
  half away from zero"; for a currency whose rounding multiple is one hundredth this is rounding to
  two decimal places.
- `floor_div(a, b)` is integer division discarding the remainder; `a mod b` is the remainder.

Named decimal precisions used here are `Product Price`, `Product Unit`, `Volume`, `Stock Weight`
and `Discount`; their default digit counts are listed in [configuration.md](configuration.md).

**A note on quoted system messages.** Text shown inside a block quote or between quotation marks in
this file is the message the system itself emits, reproduced character for character so that an
implementation can match it. A few of those messages contain abbreviations the system prints:
`URL` for uniform resource locator, `FNC1` for the function code one separator, `GS1` for Global
Standards One, and `FEFO` for first expiry first out. They are reproduced because the text is a
contract; everywhere outside a quoted message this folder writes such terms in full.

---

## 1. The variant combination

### 1.1 What a combination is

A **combination** is a set of Template Attribute Values, at most one per attribute line, drawn from
the attribute lines of one template. Three properties of a combination matter everywhere:

1. **Its identifier string** — the comma-joined list of the identifiers of its members, sorted
   ascending numerically. This is stored on a variant as its combination indices and is the key by
   which a variant is found from a combination.
2. **Its restriction to variant-creating attributes** — the members whose attribute's
   variant-creation mode is not "never". Only these members are stored on a variant.
3. **Its name** — the comma-and-space-joined names of its members after removing no-variant
   members and single-value-line members.

```formula
combination_identifier_string = join_with_comma( sort_ascending( identifiers_of_members ) )
```

The empty combination has the empty identifier string. A template with no attribute lines has
exactly one variant, whose combination indices are empty.

### 1.2 Worked example — a template with a size attribute and a colour attribute

Take a template named "Office Chair" with:

- attribute **Size**, variant creation "instantly", display type radio, sequence 10, offering the
  values **S** (sequence 1), **M** (sequence 2) and **L** (sequence 3);
- attribute **Colour**, variant creation "instantly", display type colour, sequence 20, offering
  the values **Black** (sequence 1) and **White** (sequence 2).

Two attribute lines are created on the template, in sequence order Size then Colour. Materialising
them produces six Template Attribute Values:

| Template Attribute Value | Line | Underlying value | Identifier (illustrative) |
|---|---|---|---|
| Size: S | Size line | S | 101 |
| Size: M | Size line | M | 102 |
| Size: L | Size line | L | 103 |
| Colour: Black | Colour line | Black | 104 |
| Colour: White | Colour line | White | 105 |

The six possible combinations and their identifier strings are:

| Combination | Members | Identifier string |
|---|---|---|
| S / Black | 101, 104 | `101,104` |
| S / White | 101, 105 | `101,105` |
| M / Black | 102, 104 | `102,104` |
| M / White | 102, 105 | `102,105` |
| L / Black | 103, 104 | `103,104` |
| L / White | 103, 105 | `103,105` |

---

## 2. Materialising template attribute values

Whenever a template attribute line is created or written (unless the caller suppresses the step),
the set of Template Attribute Values of that line is brought into agreement with its offered value
list. The algorithm, per line:

**Preconditions.** The line exists and its offered value list is the desired state.

1. Let `remaining` be the set of identifiers of the offered values.
2. For each existing Template Attribute Value of the line:
   - if its underlying attribute value is **not** in `remaining`: mark it for deletion, but only if
     it is currently active. An already-archived value is left alone, because being archived means
     an earlier deletion attempt already failed;
   - otherwise: remove that identifier from `remaining`, and if the Template Attribute Value is
     archived, mark it for reactivation.
3. Search for archived Template Attribute Values that belong to the same template and the same
   attribute, whose underlying value is still in `remaining`, and that are **not** attached to this
   line. Group them by underlying value and take the first of each group. For each: set it active,
   re-attach it to this line, remove it from the deletion set if it was there, and remove its
   underlying value from `remaining`. This is the step that makes a value survive being moved from
   one line to another, and therefore keeps the variants that reference it.
4. The identifiers left in `remaining`, taken in ascending order, become new Template Attribute
   Values. Each is created with the line, the underlying value, and an extra price copied from the
   underlying value's default extra price.
5. Apply the reactivations, then apply the archivals. Both are done *per line*, before moving to the
   next line, so that a later line can reuse a value a previous line has just released.
6. After every line has been processed: delete the values still marked for deletion, then create the
   queued new values.
7. Unless variant creation is suppressed, run the variant generation algorithm on the templates
   involved.

**Postconditions.** Each offered value has exactly one active Template Attribute Value on the line;
no non-offered value has an active one.

**Failure conditions.** Deleting a Template Attribute Value may fail because a document references
it; the deletion path catches the failure per record and archives instead (see
[entities.md](entities.md), section 8.5).

---

## 3. Variant generation

This is the algorithm that keeps the set of a template's variants in step with its attribute
configuration. It runs on template creation, on any write that touches the attribute lines, on
reactivation of a template that has no variants, after value materialisation, after exclusion
changes, and after a template attribute line is deleted.

### 3.1 The algorithm

**Input.** A set of templates. **Output.** For each template: variants created, variants
reactivated, variants archived or deleted.

1. If the input set is empty, stop.
2. Flush every pending change to storage, so that the queries below see a consistent state.
3. Initialise three accumulators: *to create* (a list of value sets), *to activate* (a set of
   variants), *to unlink* (a set of variants).
4. For each template:
   1. Let **lines** be the template's valid attribute lines (those with at least one offered value)
      restricted to attributes whose variant-creation mode is not "never".
   2. Let **all variants** be every variant of the template, archived ones included, sorted by
      activity flag ascending then identifier descending. (The sort matters only for the
      deterministic choice of which duplicate survives.)
   3. **Single-value repair.** Let **single lines** be the lines among **lines** whose active
      materialised values number exactly one. If there are any, then for each variant: form the
      union of the variant's current combination and the active values of the single lines. If that
      union has exactly as many members as there are **lines**, and the set of lines it covers is
      exactly **lines**, and it differs from the variant's current combination, write it onto the
      variant. The effect is that adding an attribute that offers only one value enriches the
      existing variants instead of replacing them.
   4. Build the lookup **existing** from each variant's combination to that variant.
   5. **If the template has no dynamic attribute:**
      - Enumerate the full cartesian product of the active materialised values of each line in
        **lines**, one factor per line.
      - Filter that enumeration with the configuration filter of section 4.1, ignoring no-variant
        attributes.
      - For each surviving combination: if it is in **existing**, add that variant to the
        per-template activation set; otherwise queue a creation record holding the template, the
        combination and the template's own activity flag, and then check the ceiling: if the number
        of queued creations for this template exceeds the variant limit, raise

        > The number of variants to generate is above allowed limit. You should either not generate
        > variants for each combination or generate them on demand from the sales order. To do so,
        > open the form view of attributes and change the mode of *Create Variants*.

        The limit is read from the system parameter `product.dynamic_variant_limit` and defaults to
        one thousand. The comparison is strictly greater than the limit, so exactly the limit is
        allowed.
      - Add the per-template activation set to *to activate* and the queued creations to *to
        create*.
   6. **Otherwise (the template has at least one dynamic attribute), and only if it already has
      variants:** take the combinations of the existing variants, run them through the same
      configuration filter, and add the corresponding variants to the activation set. Nothing is
      created: a dynamic template's variants come into existence one at a time.
   7. Add to *to unlink* every variant of **all variants** that is not in this template's activation
      set.
5. Activate the variants in *to activate* whose template is active. (A variant of an archived
   template stays archived.)
6. Create the variants in *to create*.
7. If *to unlink* is non-empty:
   - Put them through the delete-or-archive procedure.
   - Then check whether every input template still exists. If any template has disappeared —
     because deleting its last variant deleted it — raise

     > This configuration of product attributes, values, and exclusions would lead to no possible
     > variant. Please archive or delete your product directly if intended.

8. For each variant that was in *to unlink*, delete every combo item that names it.
9. Flush and invalidate every cache, because the collections that were read with archived records
   included are now stale.

### 3.2 Worked example — six variants with extra prices

Continuing the Office Chair of section 1.2. The template's sales price is 100.00 in the company
currency, whose rounding multiple is one hundredth.

Extra prices are set on the Template Attribute Values:

| Template Attribute Value | Extra price |
|---|---|
| Size: S | 0.00 |
| Size: M | 0.00 |
| Size: L | 5.00 |
| Colour: Black | 0.00 |
| Colour: White | 2.00 |

Running the generation algorithm:

- **lines** = [Size line, Colour line]; neither attribute is dynamic, neither is "never".
- No single-value line, so the repair step does nothing.
- The cartesian product has 3 × 2 = 6 elements.
- No exclusions exist, so all six survive the configuration filter.
- **existing** is empty (the template was just created with a single attribute-less variant, which
  has an empty combination and therefore is not in the enumeration), so six creations are queued.
- Six is below the limit of one thousand.
- The one attribute-less variant is in *to unlink*; it is deleted.
- Six variants are created.

The resulting variants and their prices:

```formula
variant_price_extra = sum_of( extra_price of each member of the combination )
variant_sales_price = template_sales_price + variant_price_extra
```

| Variant | Combination indices | Price extra | Sales price |
|---|---|---|---|
| Office Chair (S, Black) | `101,104` | 0.00 + 0.00 = 0.00 | 100.00 |
| Office Chair (S, White) | `101,105` | 0.00 + 2.00 = 2.00 | 102.00 |
| Office Chair (M, Black) | `102,104` | 0.00 + 0.00 = 0.00 | 100.00 |
| Office Chair (M, White) | `102,105` | 0.00 + 2.00 = 2.00 | 102.00 |
| Office Chair (L, Black) | `103,104` | 5.00 + 0.00 = 5.00 | 105.00 |
| Office Chair (L, White) | `103,105` | 5.00 + 2.00 = 7.00 | 107.00 |

The display names come from section 12.2: because both lines offer more than one value, both values
appear, joined by a comma and a space, inside parentheses after the template name.

### 3.3 Worked example — an excluded combination

Now add one exclusion to the Office Chair: **Colour: White excludes Size: L**. Concretely, an
exclusion record is created whose owning value is *Colour: White*, whose template is the Office
Chair, and whose excluded value list is [*Size: L*].

Creating the exclusion re-runs variant generation:

- The cartesian product still has six elements.
- The configuration filter now computes the own-exclusion map (section 4.2). It is:

  | Value | Excludes |
  |---|---|
  | Size: S | (none) |
  | Size: M | (none) |
  | Size: L | (none) |
  | Colour: Black | (none) |
  | Colour: White | Size: L |

- For the combination {Size: L, Colour: White} the union of the values excluded by its members is
  {Size: L}; that set intersects the combination itself, so the combination is skipped.
- The other five combinations survive.
- Five combinations are in **existing**; they are activated (they already are active, so nothing
  changes).
- The variant *Office Chair (L, White)* is in *to unlink*. It is deleted if nothing references it,
  and archived otherwise.

After the pass the template has five active variants. Asking whether {Size: L, Colour: White} is a
possible combination now returns false at the very first test, the configuration filter.

Note the **inversion** used by the configurator: when the exclusion map is handed to a client, it is
completed with the inverse pairs (section 4.3), so the client also learns that *Size: L* excludes
*Colour: White*. The inversion is purely presentational; the filter itself is symmetric already,
because it takes the union of what every member excludes and intersects it with the whole
combination.

### 3.4 Archival versus deletion

A variant that must disappear is first offered for deletion. The delete-or-archive procedure
(specified in [entities.md](entities.md), section 2.6) tries the whole batch, halves it on failure
and finally archives the individual records that cannot be deleted. The practical rule is:

- a variant never referenced by any document is deleted;
- a variant referenced by any document — a sales order line, a stock move, a valuation layer — is
  archived;
- deleting the last variant of a non-dynamic template deletes the template as well, which is why the
  generation algorithm re-checks the existence of its input templates afterwards.

---

## 4. Deciding whether a combination is possible

Four increasingly strict predicates exist. They are used in this order.

### 4.1 The configuration filter

**Input.** A stream of candidate combinations, and a flag saying whether no-variant attributes
should be ignored. **Output.** A stream of the combinations that satisfy the template's attribute
configuration.

Preparation, done once for the whole stream:

1. Let **lines** be the template's valid attribute lines.
2. Let **all active values** be the active materialised values of those lines.
3. If no-variant attributes are to be ignored, remove from **lines** the lines whose attribute never
   creates variants.
4. Let **lines without multi** be the lines whose attribute's display type is not multi-checkbox.
5. Compute the own-exclusion map (section 4.2).

Then, for each candidate combination:

1. Let **candidate without multi** be the members whose attribute's display type is not
   multi-checkbox.
2. If the number of members of **candidate without multi** differs from the number of **lines
   without multi**, reject. (The candidate does not name exactly one value per non-multi attribute.)
3. If the set of lines covered by **candidate without multi** is not exactly **lines without multi**,
   reject. (The candidate names a value of an attribute the template does not use, or omits one it
   does.)
4. If the candidate is not a subset of **all active values**, reject. (The candidate names a value
   the template no longer offers, or an archived one.)
5. If the own-exclusion map is non-empty: form the union of the values excluded by each member of
   the candidate. If that union intersects the candidate, reject.
6. Otherwise accept.

The **empty candidate on a template with no attribute lines** is accepted: there are zero lines, the
candidate has zero members, the covered-line sets are both empty, and the subset test on an empty
set holds.

The **single-combination form** of this filter answers "is this one combination allowed by the
configuration?" by running the filter over a one-element stream and asking whether anything came
out.

### 4.2 The own-exclusion map

**Input.** Optionally, the list of Template Attribute Value identifiers making up a combination of
interest. **Output.** A map from Template Attribute Value identifier to the list of Template
Attribute Value identifiers it excludes.

1. Let **values** be every materialised value of the template's valid attribute lines.
2. Build the value filter: "the value is active", widened to "or the value is one of the
   combination of interest" when such a combination was given; then narrowed to "and the value is
   one of **values**". The widening is what lets a configurator explain why an *already chosen*,
   now archived value is refused.
3. Group the template's exclusion records by their owning value, keeping only exclusions whose
   owning value passes the filter and whose template is this template.
4. For each value of **values** that is active, or that belongs to the combination of interest:
   - if it owns exclusions, map it to the identifiers of the excluded values that are **active**;
   - otherwise map it to the empty list.

Note that the map always contains a key for every eligible value, even when the list is empty. The
filter in section 4.1 relies on this: it looks up every member of the candidate without guarding
against a missing key.

### 4.3 Completing the inverse exclusions

The map handed to a configurator is completed so that exclusion is visibly symmetric:

1. Start from a copy of the own-exclusion map.
2. For each key and each value in its list:
   - if that value is already a key of the result and the original key is not already in its list,
     append the original key;
   - otherwise set the result's entry for that value to a one-element list containing the original
     key.

**Worked example.** Suppose *Colour: Black* (identifier 104) excludes *Size: L* (103) and *Size: M*
(102), and no other exclusion exists. The own map is:

```
101 → []        (Size: S)
102 → []        (Size: M)
103 → []        (Size: L)
104 → [103,102] (Colour: Black)
105 → []        (Colour: White)
```

Completion visits key 104, value 103: 103 is already a key with an empty list not containing 104, so
103 becomes `[104]`. Then key 104, value 102: 102 becomes `[104]`. The result is:

```
101 → []
102 → [104]
103 → [104]
104 → [103,102]
105 → []
```

Note the second branch of the rule overwrites rather than appends when the value is not yet a key;
in practice every value of the template *is* a key, so the overwrite branch is reached only for
values belonging to another template, which cannot happen for own exclusions.

### 4.4 The parent-exclusion map

**Input.** A parent combination — the combination of the product from which this template is offered
as an option or an accessory. **Output.** A map from parent value identifier to the list of this
template's value identifiers that the parent value forbids.

1. If the parent combination is empty, the map is empty.
2. For each value of the parent combination, for each exclusion record it owns whose template is
   *this* template:
   - if the exclusion names values, map the parent value to those value identifiers;
   - if the exclusion names no value, map the parent value to **every** materialised value of every
     attribute line of this template. An exclusion with an empty value list therefore means "this
     whole product is incompatible with that parent value".

### 4.5 The full possibility predicate

**Input.** A combination, optionally a parent combination, and the ignore-no-variant flag.
**Output.** True or false.

1. If the configuration filter rejects the combination, return false.
2. Find the variant for the combination (section 5.1).
3. If the template has at least one dynamic attribute:
   - if a variant was found and it is archived, return false. (A dynamic combination that was
     explicitly archived stays refused.)
   - if no variant was found, continue: it can be created on demand.
4. Otherwise (no dynamic attribute):
   - if no variant was found, or the variant is archived, return false.
5. Compute the parent-exclusion map. If any value listed anywhere in it is a member of the
   combination, return false.
6. Return true.

The **variant form** of the predicate asks whether an existing variant is possible: it applies the
full predicate to the variant's own combination, with no-variant attributes ignored.

### 4.6 The attribute-exclusion payload

The payload a configurator or a storefront receives for one template, optionally in the context of a
parent combination and optionally for a specific combination of interest, is a five-entry record:

| Entry | Content |
|---|---|
| `exclusions` | The own-exclusion map of section 4.2, completed with its inverses as in section 4.3. |
| `archived_combinations` | The list of identifier tuples of the template's archived variants, keeping only those whose values are all active (or are in the combination of interest), minus the set of identifier tuples of the active variants. |
| `parent_exclusions` | The parent-exclusion map of section 4.4. |
| `parent_combination` | The identifiers of the parent combination. |
| `parent_product_name` | The display name of the parent product, supplied by the caller, used in explanations such as "Not available with Customizable Desk (Legs: Steel)". |
| `mapped_attribute_names` | A map from value identifier to display name, covering every materialised value of the template's valid attribute lines plus every value of the parent combination. Used to build explanations such as "Not available with Color: Black". |

The subtraction in `archived_combinations` matters: a combination may have both an archived variant
and an active one (the uniqueness index only constrains active variants), and in that case it is
orderable and must not be reported as archived.

---

## 5. Finding and creating a variant for a combination

### 5.1 Finding

**Input.** A template and a combination. **Output.** At most one variant.

1. Remove the no-variant members from the combination; call the result the **filtered combination**.
2. Compute the filtered combination's identifier string.
3. Search the variants of the template, archived ones included, whose combination indices equal that
   string — or, when the string is empty, whose combination indices are empty or unset — ordered by
   activity flag descending, keeping the first.
4. Return it, or nothing.

The ordering by activity flag descending means that when both an active and an archived variant
carry the same combination, the active one wins.

The lookup is cached on the pair (template identifier, frozen set of filtered combination
identifiers) and is performed as a privileged reader, so that the cache is shared by all users. It
is invalidated whenever a variant is created, whenever a variant's combination is written, and
whenever a variant is activated or archived.

### 5.2 Creating on demand

**Input.** A template, a combination that must contain **all** its members including no-variant
ones (they are needed to evaluate possibility even though they are not stored), and a flag saying
whether a failure should be recorded in the technical log.

1. Find the variant for the combination.
2. If one exists:
   - if it is archived **and** the template has a dynamic attribute **and** the full possibility
     predicate accepts the combination, reactivate it;
   - return it.
3. If the template has no dynamic attribute, return nothing. (Optionally log: a user tried to create
   a variant for a non-dynamic product.)
4. If the full possibility predicate rejects the combination, return nothing. (Optionally log: a
   user tried to create an invalid variant.)
5. Otherwise create a variant, as a privileged writer, with the template and with the combination
   restricted to variant-creating attributes.

**Creating the first variant** of a template is this same procedure applied to the first possible
combination (section 6.2).

### 5.3 Worked example — a dynamically created variant

Take a template "Engraved Pen" whose attribute **Nib** has variant creation **dynamically** and
offers **Fine**, **Medium** and **Broad**, and whose attribute **Barrel** also has variant creation
**dynamically** and offers **Brass** and **Steel**. Template sales price 40.00. Extra prices: Broad
3.00, Steel 6.00; the rest zero.

**State after configuration.** Materialisation creates five Template Attribute Values. Variant
generation runs, finds the template has a dynamic attribute, finds it has one variant (the
attribute-less one created with the template), runs that variant's empty combination through the
configuration filter — which rejects it, because the combination covers zero lines while the
template has two — and therefore puts it in *to unlink*. The attribute-less variant is deleted. The
template now has **zero** variants and is not deleted, because it has a dynamic attribute.

**A buyer configures Broad + Steel.** The consuming domain calls the create-on-demand procedure with
the combination {Nib: Broad, Barrel: Steel}:

1. Find: the filtered combination identifier string is, say, `203,205`; no variant has it; nothing
   is found.
2. The template has a dynamic attribute, so creation is allowed.
3. The full possibility predicate:
   - configuration filter: two non-multi members, two non-multi lines, covered lines match, both
     values active, no exclusions → accepted;
   - find the variant → none; template is dynamic → continue;
   - no parent combination → no parent exclusions;
   - accepted.
4. A variant is created with the template and the combination `{Nib: Broad, Barrel: Steel}`.

Its price extra is 3.00 + 6.00 = 9.00 and its sales price is 40.00 + 9.00 = **49.00**.

**The same buyer orders Fine + Brass later.** A second variant is created the same way, with price
extra 0.00 and sales price 40.00. The template now has two variants out of six possible
combinations; the other four have never been created and never will be until somebody orders them.

**Archiving one.** If the Broad + Steel variant is archived, the create-on-demand procedure on that
same combination finds it, sees it is archived, sees the template is dynamic and the combination is
still possible — except that the full possibility predicate returns **false** for an archived
variant of a dynamic template. So the reactivation branch is not taken and the archived variant is
returned as-is. The consuming domain must therefore check possibility before using the returned
variant.

---

## 6. Enumerating possible combinations

### 6.1 The pruned cartesian product

The plain cartesian product of a template's values is unusable when the template has many attributes
and exclusions: filtering after the fact enumerates combinations that could have been ruled out
after the second value. The domain therefore uses a backtracking enumeration that tests exclusions
as values are added.

**Input.** A list of value lists, one per attribute line (a multi-checkbox line contributes an empty
list), and a parent combination. **Output.** A stream of combinations.

1. If the input list is empty, stop with no output.
2. Drop the empty value lists **except** that if every list was empty, emit the empty combination
   once and stop. (This is the only place the empty combination is emitted.)
3. Build the own-exclusion map, expressed as a map from value to the set of values it excludes.
4. Build a **rejection counter**: a map from value to a whole number, initially zero everywhere.
   Increase it by one for every value named as a key of the parent-exclusion map. A value whose
   counter is greater than zero cannot enter the partial combination.
5. Let the **partial combination** be empty, let the **chosen index** of each line be −1 (meaning
   "no value chosen for this line yet"), and let the **line cursor** be the first line.
6. Loop:
   1. Let **current values** be the value list of the line at the cursor and **current index** its
      chosen index.
   2. If **current values** is empty — which can only happen for a multi-checkbox line — then: if the
      cursor is on the last line, emit the partial combination; otherwise advance the cursor to the
      next line and continue the loop.
   3. Otherwise let **current value** be the value at **current index**.
   4. If **current index** is at least zero, the current value is leaving the partial combination:
      decrease the rejection counter of every value it excludes, and remove it from the partial
      combination.
   5. If **current index** is not the last index of **current values**: increase the chosen index of
      this line by one and re-read the current value.
   6. Else, if the cursor is not on the first line: reset this line's chosen index to −1, move the
      cursor to the previous line and continue the loop.
   7. Else: the first line has been exhausted; stop.
   8. The new current value is entering the partial combination: increase the rejection counter of
      every value it excludes, and add it to the partial combination.
   9. If the new current value's own rejection counter is greater than zero, or any value it excludes
      is already in the partial combination, continue the loop (this value is refused; the next
      iteration will try the following value of the same line).
   10. If the cursor is on the last line, emit the partial combination; otherwise advance the cursor
       to the next line.

The stream yields the **same recordset object** each time, mutated in place; a consumer that wants to
keep a combination must copy it. Consumers in this domain immediately either test it or add it to
another set, so the aliasing is not observable.

### 6.2 Possible combinations, in sequence order

**Input.** A template, optionally a parent combination, optionally a set of **necessary values** that
must appear in every result. **Output.** A stream of possible combinations.

1. If the template is archived, stop with no output. (The generator returns the explanatory string
   "The product template is archived so no combination is possible.", which callers treat as
   exhaustion.)
2. Let **necessary lines** be the attribute lines of the necessary values. Let **lines** be the
   template's valid attribute lines minus the necessary lines.
3. If **lines** is empty and the full possibility predicate accepts the necessary values with the
   parent combination, emit the necessary values.
4. For each line in **lines**, in the template's attribute-line order: if its attribute's display
   type is not multi-checkbox, contribute the line's active materialised values (in their own
   order); otherwise contribute an empty list.
5. Run the pruned cartesian product over those lists with the parent combination.
6. For each partial combination it emits, form the union with the necessary values, and if the full
   possibility predicate accepts it with the parent combination, emit it.
7. When the product is exhausted, stop. (The generator returns the string "There are no remaining
   possible combination.")

Because step 4 walks the lines in the template's order and each list is in the values' order, the
stream is ordered by attribute sequence and, within an attribute, by value sequence. The **first
possible combination** is simply the first element of this stream, or the empty combination when the
stream is empty — which is why a caller that needs to distinguish "no combination is possible" from
"the empty combination is the answer" must re-test the result with the possibility predicate.

### 6.3 Closest possible combinations

**Input.** A template and a desired combination that may be incomplete, may be impossible, or both.
**Output.** A stream of possible combinations that keep as much of the desired combination as
possible.

1. Loop:
   1. Ask for the possible combinations with the current desired combination as necessary values.
   2. If that stream yields at least one element, emit that element and then every further element of
      the stream, and stop. (The generator returns the string "There are no remaining closest
      combination.")
   3. Otherwise, if the desired combination is empty, stop. (The generator returns the string "There
      are no possible combination.")
   4. Otherwise drop the **last** member of the desired combination and repeat.

Because the members of a combination are ordered by attribute line then attribute value, dropping the
last member drops the value of the attribute that appears last in the configurator — the most
recently answered question. The **closest possible combination** is the first element of this
stream.

**Worked example.** Office Chair with the exclusion "White excludes L". A buyer arrives with a
pre-filled link asking for {Size: L, Colour: White}.

- First iteration: necessary values = {L, White}; **lines** is empty (both lines are necessary
  lines); the possibility predicate rejects {L, White}; the pruned product over an empty list of
  lists emits nothing. Stream empty.
- Drop the last member. Members are ordered by line then value: the Size line has sequence 10 and
  the Colour line sequence 20, so the order is [Size: L, Colour: White] and the last member is
  *Colour: White*. Desired becomes {Size: L}.
- Second iteration: necessary values = {L}; **lines** = [Colour line]; the pruned product yields
  {Black} then {White}; union with {L} gives {L, Black} — accepted — and {L, White} — rejected.
  Stream yields {Size: L, Colour: Black}.

The closest possible combination is therefore **L / Black**: the buyer keeps the size they asked for
and loses the colour.

---

## 7. Extra prices

### 7.1 On a variant

```formula
variant_price_extra = sum_of( price_extra of each Template Attribute Value in the variant combination )
```

The summands are in the template's currency. No rounding is applied to the sum itself; the result is
a decimal shown with the `Product Price` precision.

```formula
variant_sales_price_in_product_unit = template_sales_price + variant_price_extra
```

When the caller names a unit of measure in the context, the **template** price is converted first
and the extra is added afterwards:

```formula
variant_sales_price_in_named_unit
    = ( template_sales_price × named_unit_absolute_factor ÷ product_unit_absolute_factor )
      + variant_price_extra
```

This is deliberate and is a real asymmetry: the attribute extra is **not** converted between units.

Writing the variant sales price inverts the computation:

```formula
new_template_sales_price
    = ( written_price × product_unit_absolute_factor ÷ named_unit_absolute_factor )
      − variant_price_extra
```

with the unit conversion applied only when a unit was named in the context.

**Worked example.** The Office Chair's default unit is "Units". A caller reads the sales price of
*Office Chair (L, White)* in the unit "Dozens", whose absolute factor relative to "Units" is one
twelfth (one dozen contains twelve units, so the dozen's absolute quantity is 1 ÷ 12 when the unit
is the reference — see `../units-of-measure-and-packaging/` for the exact factor convention). With a
template price of 100.00 and a price extra of 7.00:

```
converted template price = 100.00 × 12 = 1200.00
variant sales price      = 1200.00 + 7.00 = 1207.00
```

If the caller then writes 1300.00 back:

```
new template price = 1300.00 ÷ 12 − 7.00 = 108.3333… − 7.00 = 101.3333…
```

stored at the `Product Price` precision.

### 7.2 Extra prices of values that never create a variant

A value of a "never create variants" attribute is not stored on any variant, so its extra price
cannot come from the variant's own price extra. It travels through the **price context** instead.

**On a variant**, given the full combination chosen by the buyer:

```formula
no_variant_attributes_price_extra
    = sum_of( price_extra of each Template Attribute Value V in the chosen combination
              such that  V.price_extra ≠ 0
                    and  V belongs to this variant's template
                    and  V is not one of this variant's own combination members )
```

The last condition — rather than "V's attribute never creates variants" — is deliberate: an
attribute whose mode was changed after the variants were built may still have its values stored on a
variant, and those must not be counted twice.

```formula
variant_attributes_extra_price = variant_price_extra + no_variant_attributes_price_extra
```

**On a template** (used when no variant exists yet, for instance while configuring):

```formula
current_attributes_price_extra
    = sum_of( price_extra of each Template Attribute Value V in the chosen combination
              such that  V.price_extra ≠ 0
                    and  V belongs to this template )
template_attributes_extra_price = current_attributes_price_extra
```

Both quantities are passed as context entries — `no_variant_attributes_price_extra` and
`current_attributes_price_extra` respectively — to the price computation of section 7.3.

**Worked example.** A template "Business Card Pack" priced 25.00 has:

- attribute **Paper**, variant creation instantly, values *Matte* (extra 0.00) and *Glossy* (extra
  4.00);
- attribute **Engraving**, variant creation **never**, display type radio, values *None* (extra
  0.00) and *Gold foil* (extra 12.00), with *Gold foil* additionally flagged as free text.

Two variants exist: Matte and Glossy. A buyer picks **Glossy** and **Gold foil**, typing the text
"Acme Ltd".

- The variant is *Business Card Pack (Glossy)*; its own price extra is 4.00.
- The chosen combination is {Paper: Glossy, Engraving: Gold foil}. The no-variant extra is the sum
  over the members with a non-zero extra, belonging to this template, not in the variant's own
  combination: *Gold foil* qualifies (12.00); *Glossy* does not (it is in the variant's combination).
  So the no-variant extra is 12.00.
- The attributes extra price is 4.00 + 12.00 = 16.00.
- The unit price before any pricelist rule is 25.00 + 16.00 = **41.00**.
- Because *Gold foil* is flagged as free text, an Attribute Custom Value record is created holding
  the Template Attribute Value *Engraving: Gold foil* and the text "Acme Ltd". Its computed name is
  "Engraving: Gold foil: Acme Ltd".

### 7.3 The price computation of a template or a variant

**Input.** A price kind — either `list_price` ("sales price") or `standard_price` ("cost") — an
optional target unit, an optional target currency, an optional company, an optional date.
**Output.** One amount per record.

1. The company defaults to the acting company; the date defaults to today in the reader's time zone.
2. The records are re-read in the context of that company. For the cost, they are additionally
   re-read as a privileged reader, because the cost is readable only by internal users and the sales
   price may have to be derived from it for others.
3. For each record:
   1. Take the raw amount: the record's value for the chosen price kind, or zero when it is unset.
   2. Choose the source currency: for the cost, the record's cost currency; for anything else, the
      record's currency.
   3. **On a template only**, for the cost: when the template's own cost came back as zero and the
      template has variants, take the first variant's cost instead.
   4. For the sales price only, add the attributes extra price of section 7.2.
   5. If a target unit was given, convert:

      ```formula
      converted = amount × target_unit_absolute_factor ÷ source_unit_absolute_factor
      ```

      where the source unit is the product's default unit. The conversion is skipped when the two
      units are the same, when the amount is zero, or when either unit is missing.
   6. If a target currency was given, convert from the source currency to it, for the given company
      and date, using the currency conversion of `../multi-currency/`.

**Evaluation order matters**: the extra price is added *before* the unit conversion, so for the
sales price the extra **is** converted when a unit is given here — the opposite of the variant's own
sales-price field of section 7.1, which converts first and adds afterwards. Both behaviours exist in
the system and an implementation must reproduce both.

---

## 8. Combos

### 8.1 The base price of a choice group

```formula
choice_base_price = minimum over items of
    convert( item_variant_sales_price,
             from = item_variant_currency,
             to   = choice_currency,
             company = choice_company or acting_company,
             date = current_moment )
```

with the base price equal to zero when the group has no items. The item's sales price is the
variant's sales price **including its own attribute extras**, that is, `lst_price`. The conversion
date is the current moment, not the order date; the base price is therefore a live figure and is not
stored.

### 8.2 Prorating a combo product's price over the chosen items

**Input.** A document line carrying a combo product, and one child line per chosen combo item.
**Output.** A unit price for each child line.

1. Let **combo product price** be the price the combo product itself would have had if it were an
   ordinary product — the pricelist price, or, when the applicable rule displays a discount, the
   larger of the pre-discount base price and the pricelist price. Combo logic is deliberately
   ignored at this step.
2. For each choice group of the combo product, convert its base price into the document currency,
   for the document company, at the document date:

   ```formula
   converted_base_price(g) = convert( base_price(g), choice_currency(g), document_currency,
                                      document_company, document_date )
   ```

3. Let **total base price** be the sum of the converted base prices.
4. If the total base price is non-zero, the prorated share of each group is

   ```formula
   share(g) = round_to_currency( converted_base_price(g) × combo_product_price
                                 ÷ total_base_price,
                                 document_currency )
   ```

5. Otherwise — every group has a zero base price — the shares are equal:

   ```formula
   share(g) = round_to_currency( combo_product_price ÷ number_of_groups, document_currency )
   ```

   This special case exists because prorating by zero weights would concentrate the whole price on
   one group through the remainder correction below.
6. Compute the rounding remainder and give it to the **last** choice group of the combo product, in
   the groups' own order (sequence ascending, then identifier):

   ```formula
   remainder = combo_product_price − sum over g of share(g)
   share(last_group) = share(last_group) + remainder
   ```

7. The unit price of the child line for a chosen item is its group's share plus that item's extra
   price and the extra prices of any never-create-variant attribute values chosen on it, both
   converted into the document currency for the document company at the document date:

   ```formula
   item_unit_price = share( group_of(item) )
                   + convert( item_extra_price + no_variant_attributes_price_extra,
                              item_currency, document_currency,
                              document_company, document_date )
   ```

8. The combo product's own line always displays a unit price of **zero**: the whole price is carried
   by the child lines.

**Postcondition.** The sum of the child lines' unit prices, before the per-item extra prices, equals
the combo product price exactly, in the document currency.

### 8.3 Worked example — a combination product's price

A restaurant sells a combo product **"Lunch Menu"** at a catalogue price of 15.00 in euro, the
document currency, whose rounding multiple is one hundredth. No pricelist rule applies, so the combo
product price is 15.00.

The Lunch Menu references three choice groups:

| Group | Sequence | Items (sales price incl. attribute extras) | Item extra price | Base price |
|---|---|---|---|---|
| Main | 10 | Burger 9.00, Veggie Wrap 8.00, Steak 14.00 | Burger 0.00, Veggie Wrap 0.00, Steak 3.00 | min(9.00, 8.00, 14.00) = **8.00** |
| Side | 20 | Fries 3.50, Salad 4.00 | both 0.00 | min(3.50, 4.00) = **3.50** |
| Drink | 30 | Water 2.00, Soda 2.50 | Water 0.00, Soda 0.50 | min(2.00, 2.50) = **2.00** |

All amounts are already in euro, so the conversions are identities.

```
total base price = 8.00 + 3.50 + 2.00 = 13.50

share(Main)  = round_to_currency( 8.00 × 15.00 ÷ 13.50 ) = round_to_currency( 8.888888… ) = 8.89
share(Side)  = round_to_currency( 3.50 × 15.00 ÷ 13.50 ) = round_to_currency( 3.888888… ) = 3.89
share(Drink) = round_to_currency( 2.00 × 15.00 ÷ 13.50 ) = round_to_currency( 2.222222… ) = 2.22

sum of shares = 8.89 + 3.89 + 2.22 = 15.00
remainder     = 15.00 − 15.00 = 0.00
```

The remainder is zero here, so no correction is applied.

A customer orders **Steak**, **Salad** and **Soda**. The child line unit prices are:

```
Steak line = share(Main)  + 3.00 = 8.89 + 3.00 = 11.89
Salad line = share(Side)  + 0.00 = 3.89
Soda  line = share(Drink) + 0.50 = 2.22 + 0.50 = 2.72
Lunch Menu line = 0.00

order total = 0.00 + 11.89 + 3.89 + 2.72 = 18.50
```

which is the menu price of 15.00 plus the 3.00 steak supplement and the 0.50 soda supplement, as
intended.

**A case with a remainder.** Change the menu price to 10.00 and keep the same base prices:

```
share(Main)  = round_to_currency( 8.00 × 10.00 ÷ 13.50 ) = round_to_currency( 5.925925… ) = 5.93
share(Side)  = round_to_currency( 3.50 × 10.00 ÷ 13.50 ) = round_to_currency( 2.592592… ) = 2.59
share(Drink) = round_to_currency( 2.00 × 10.00 ÷ 13.50 ) = round_to_currency( 1.481481… ) = 1.48

sum of shares = 5.93 + 2.59 + 1.48 = 10.00
remainder     = 0.00
```

Still zero. Now make the base prices 1.00, 1.00 and 1.00 with a menu price of 10.00:

```
share(g) = round_to_currency( 1.00 × 10.00 ÷ 3.00 ) = round_to_currency( 3.333333… ) = 3.33  (each)
sum      = 9.99
remainder = 10.00 − 9.99 = 0.01
share(Drink) = 3.33 + 0.01 = 3.34
```

The last group in order — Drink, sequence 30 — absorbs the cent. The three child lines are 3.33,
3.33 and 3.34, summing exactly to 10.00.

**The all-zero case.** If the three groups' base prices were all 0.00, the total would be zero and
the even-share branch would apply: each share would be round_to_currency(10.00 ÷ 3) = 3.33, the
remainder 0.01 would again go to the last group, and the result would be the same 3.33 / 3.33 /
3.34.

---

## 9. The product matrix

**Input.** A template, optionally a company, optionally a target currency, and a flag saying whether
extra prices should be displayed (they are hidden on purchase documents). **Output.** A header row
and a grid of cells.

1. The company defaults to the template's company, and failing that to the acting company. The
   currency defaults to the template's currency.
2. Let **lines** be the template's valid attribute lines, in their order.
3. Let **first-line values** be the active materialised values of the first line, in their order.
   Let **values per line** be, for each line, the list of identifiers of its active materialised
   values.
4. The **header** is a list whose first element is a cell holding the template's display name, and
   whose remaining elements are one header cell per first-line value.
5. Build the enumeration: start from a list containing one empty tuple; for each line's value list in
   order, replace the enumeration by the list formed by taking each value `y` of that line, in order,
   and each partial tuple `x` of the current enumeration, in order, and appending `y` to `x`. This
   produces the cartesian product in **first-line-fastest** order: consecutive entries differ in the
   first line's value.
6. Cut the enumeration into consecutive chunks of as many entries as there are first-line values.
   Each chunk is one **row**.
7. For each row:
   1. The row's header cell is built from the members of the first entry **after the first one** —
      that is, the values of every line except the first — which are the values shared by the whole
      row.
   2. For each entry in the row, produce a cell holding: the identifiers of the entry sorted
      ascending (`ptav_ids`), a quantity of zero (`qty`), and whether the full possibility predicate
      accepts the entry (`is_possible_combination`).
8. Return the header and the list of rows.

A **header cell** is built from a set of values as follows:

```formula
header_cell_name = join_with( " • ", names_of_the_values )     when the set is non-empty
header_cell_name = " "                                         when the set is empty
```

The single space for the empty set exists so that a template with only one attribute line renders a
blank row header rather than an empty one, which the client would otherwise label "Not available".

```formula
header_cell_extra_price = sum_of( price_extra of the values )        when extras are displayed
header_cell_extra_price = 0                                          otherwise
```

When the extra price is non-zero, the cell also carries the target currency and the converted
amount:

```formula
header_cell_price = convert( header_cell_extra_price,
                             from = template_currency, to = target_currency,
                             company = company, date = today )
```

When the extra price is zero, neither the currency nor the price appears in the cell.

**Worked example.** The Office Chair of section 3.2, with the exclusion of section 3.3, displayed in
the company currency with extras shown.

- **lines** = [Size, Colour]; first-line values = [S, M, L]; values per line =
  [[101,102,103], [104,105]].
- Header = [ {name: "Office Chair"}, {name: "S"}, {name: "M"}, {name: "L", price: 5.00,
  currency: …} ]. The L cell carries a price because *Size: L* has a non-zero extra.
- Enumeration: start `[()]`; after the Size line it is `[(101),(102),(103)]`; after the Colour line
  it is `[(101,104),(102,104),(103,104),(101,105),(102,105),(103,105)]`.
- Chunks of three: row one is `[(101,104),(102,104),(103,104)]`, row two is
  `[(101,105),(102,105),(103,105)]`.
- Row one's header cell is built from the members of `(101,104)` after the first, that is `{104}` =
  *Black*: `{name: "Black"}` with no price, because Black's extra is zero.
- Row two's header cell is `{name: "White", price: 2.00, currency: …}`.
- The cells:

| | S | M | L |
|---|---|---|---|
| **Black** | `101,104`, qty 0, possible | `102,104`, qty 0, possible | `103,104`, qty 0, possible |
| **White** | `101,105`, qty 0, possible | `102,105`, qty 0, possible | `103,105`, qty 0, **not possible** |

The bottom-right cell is marked impossible because of the exclusion, and the client renders it as
unavailable rather than as an input box.

---

## 10. Barcodes — the classic nomenclature

### 10.1 The check digit

The check digit is the last digit of a fixed-length numeric barcode. It is computed from the other
digits so that any single-digit error and most transpositions are detected. The same arithmetic
serves the eight-digit and thirteen-digit European Article Numbers, the twelve-digit Universal
Product Code, the fourteen-digit Global Trade Item Number and the eighteen-digit Serial Shipping
Container Code.

**Input.** A numeric string whose last character is the (possibly wrong, possibly placeholder) check
digit. **Output.** A digit from zero to nine.

1. Drop the last character and reverse what remains. Call the resulting sequence `d[0], d[1], …`
   (so `d[0]` is the digit immediately to the left of the check digit).
2. Sum the digits at even positions into `even_sum` and the digits at odd positions into `odd_sum`.
3. Compute:

```formula
weighted_total = even_sum × 3 + odd_sum
check_digit    = ( 10 − ( weighted_total mod 10 ) ) mod 10
```

Reversing first is what makes the arithmetic length-independent: the digit adjacent to the check
digit always gets the weight three, whatever the total length.

**Worked example.** Compute the check digit of the twelve-digit prefix `213456701250`, that is, of
the thirteen-character string `2134567012500` with a placeholder zero.

Dropping the last character gives `213456701250`; reversing gives
`0, 5, 2, 1, 0, 7, 6, 5, 4, 3, 1, 2`.

| Position | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Digit | 0 | 5 | 2 | 1 | 0 | 7 | 6 | 5 | 4 | 3 | 1 | 2 |
| Parity | even | odd | even | odd | even | odd | even | odd | even | odd | even | odd |

```
even_sum = 0 + 2 + 0 + 6 + 4 + 1 = 13
odd_sum  = 5 + 1 + 7 + 5 + 3 + 2 = 23
weighted_total = 13 × 3 + 23 = 39 + 23 = 62
check_digit = ( 10 − ( 62 mod 10 ) ) mod 10 = ( 10 − 2 ) mod 10 = 8
```

The complete thirteen-digit barcode is therefore `2134567012508`.

### 10.2 The encoding check

**Input.** A string and an encoding name. **Output.** True or false.

1. If the encoding is `any`, return true.
2. Look up the required length:

| Encoding | Full name | Required length |
|---|---|---|
| `ean8` | eight-digit European Article Number | 8 |
| `ean13` | thirteen-digit European Article Number | 13 |
| `gtin14` | fourteen-digit Global Trade Item Number | 14 |
| `upca` | twelve-digit Universal Product Code | 12 |
| `sscc` | eighteen-digit Serial Shipping Container Code | 18 |

3. Return true when **all** of the following hold:
   - the encoding is not `ean13`, **or** the first character is not the digit zero (a thirteen-digit
     code beginning with zero is a Universal Product Code in disguise and must be treated as one);
   - the length equals the required length;
   - every character is a digit;
   - the check digit computed from the string equals the string's last digit.

An encoding name not in the table is a programming error and produces a lookup failure.

### 10.3 Sanitising

```formula
sanitised_thirteen_digit_code = first_thirteen_characters_left_padded_with_zeros_to_thirteen
                                with its last character replaced by
                                check_digit( that padded string )
```

```formula
sanitised_twelve_digit_code = sanitised_thirteen_digit_code( "0" followed by the input )
                              with its leading character removed
```

The twelve-digit sanitiser therefore prefixes a zero, sanitises as thirteen digits, and strips the
zero again — which is exactly the equivalence between the two schemes.

### 10.4 Matching a pattern

**Input.** A barcode string and a pattern. **Output.** A numeric value, a base code and a match
verdict.

1. Initialise the value to zero, the base code to the barcode, and the verdict to false.
2. Escape the barcode: replace every backslash by two backslashes, every opening brace by a
   backslash and an opening brace, every closing brace by a backslash and a closing brace, and every
   full stop by a backslash and a full stop. From here on, "the barcode" means the escaped string.
   The purpose is that the barcode is about to be used as the *subject* of a match against a pattern
   built partly from itself, and its own metacharacters must not be interpreted.
3. Search the pattern for the first occurrence of an opening brace, then zero or more letters N, then
   zero or more letters D, then a closing brace. Call its start position **num start** and its end
   position (one past the closing brace) **num end**.
4. If such a group exists:
   1. The **value string** is the barcode from **num start** up to but excluding **num end** minus
      two. The subtraction of two accounts for the two brace characters, which occupy positions in
      the pattern but not in the barcode.
   2. Within the group, find the **whole part**: an opening brace, then zero or more letters N, then
      one character that is either a letter D or a closing brace. Its length minus two is the number
      of whole digits.
   3. Within the group, find the **decimal part**: one character that is either an opening brace or a
      letter N, then zero or more letters D, then a closing brace. Its start offset and its end
      offset minus one delimit the decimal digits inside the value string.
   4. The whole digits are the value string's first *whole-part* characters; if that is empty it is
      read as the single digit zero. The decimal digits are the value string between the decimal
      part's start offset and its end offset minus one, read as the fractional part of a number whose
      whole part is zero.
   5. If the whole digits are all digits:

      ```formula
      value = whole_digits_as_integer + decimal_digits_as_fraction
      ```

      and the **base code** becomes the barcode with the positions from **num start** to **num end**
      minus two replaced by that many digit-zero characters, then unescaped (the four escapes of step
      2 reversed, in the same order). The **pattern** likewise becomes the pattern with the whole
      brace group replaced by that many digit-zero characters.
5. Attempt the pattern match of the (possibly rewritten) pattern against the first *pattern-length*
   characters of the base code, anchored at the start. The verdict is whether that match succeeded.

### 10.5 Parsing a barcode against a nomenclature

**Input.** A scanned string and a nomenclature. **Output.** Either a single typed result, or — for a
uniform resource identifier — a list of typed results, or — for a Global Standards One nomenclature
— the list produced by the decomposition of section 11.

1. If the scanned string begins with `urn:`, decode it as a uniform resource identifier
   (section 10.7) and return the result.
2. If the nomenclature is a Global Standards One nomenclature, run the decomposition of section 11.
3. Otherwise, initialise the result to: encoding empty, type `error`, code equal to the scanned
   string, base code equal to the scanned string, value zero.
4. For each rule of the nomenclature, in ascending sequence then ascending identifier:
   1. Let **current barcode** be the scanned string.
   2. If the rule demands the thirteen-digit European Article Number, the scanned string is a valid
      twelve-digit Universal Product Code, and the conversion policy is `upc2ean` or `always`, prefix
      a zero to **current barcode**.
   3. Else if the rule demands the twelve-digit Universal Product Code, the scanned string is a valid
      thirteen-digit European Article Number, its first character is a zero, and the conversion
      policy is `ean2upc` or `always`, drop the first character of **current barcode**.
   4. If the **scanned string** — not the converted one — does not satisfy the rule's encoding, skip
      this rule. (This is the order the system uses: the conversion is prepared before the check, but
      the check is applied to the original string. In practice the check therefore rejects any rule
      whose encoding differs from the scanned string's, and the conversion only takes effect for the
      `any` encoding or when the two coincide.)
   5. Match **current barcode** against the rule's pattern (section 10.4).
   6. If it matched:
      - if the rule's type is `alias`, replace the scanned string by the rule's alias, set the
        result's code to that alias, and **continue with the next rule** — the alias is not a final
        answer;
      - otherwise set the result's encoding, type and value from the rule and the match, set the
        result's code to **current barcode**, and set the result's base code to: the sanitised
        thirteen-digit form of the match's base code when the rule's encoding is `ean13`, the
        sanitised twelve-digit form when it is `upca`, and the match's base code unchanged otherwise.
        Return the result.
5. If no rule matched, return the initialised error result.

### 10.6 Worked example — parsing a weight barcode

A grocery prints weight-embedded labels. The nomenclature has one rule:

| Field | Value |
|---|---|
| Rule Name | Weighted Product |
| Sequence | 10 |
| Encoding | `ean13` |
| Type | `product` |
| Pattern | `21.....{NNDDD}.` |

The pattern reads: the two literal digits 2 and 1; five arbitrary characters (the item reference);
a numeric field of two whole digits and three decimal digits (the weight in kilograms); one
arbitrary character (the check digit).

A scale prints the label for 1.250 kilograms of item reference `34567`. It composes the twelve-digit
prefix `21` + `34567` + `01250` = `213456701250` and computes the check digit 8 (section 10.1),
giving the printed barcode **`2134567012508`**.

**Parsing that scan:**

1. The string does not begin with `urn:`; the nomenclature is not a Global Standards One
   nomenclature.
2. The single rule demands `ean13`. The encoding check on `2134567012508`: the first character is
   `2`, not zero; the length is thirteen; every character is a digit; the check digit computed from
   the string is 8 and the last digit is 8. The check passes.
3. Pattern matching:
   - The barcode contains no backslash, brace or full stop, so escaping leaves it unchanged.
   - The brace group `{NNDDD}` occupies pattern positions 7 through 13 inclusive, so **num start** is
     7 and **num end** is 14.
   - The value string is the barcode from position 7 to position 12 exclusive: characters at
     positions 7, 8, 9, 10 and 11, which are `0`, `1`, `2`, `5`, `0` → **`01250`**.
   - The whole part inside the group matches `{NND`, of length 4, so the number of whole digits is
     4 − 2 = 2. The whole digits are the first two characters of the value string: **`01`**.
   - The decimal part inside the group matches from offset 2 to offset 7 (`NDDD}`), so the decimal
     digits run from offset 2 to offset 6 of the value string: characters at offsets 2, 3 and 4 →
     **`250`**.
   - Value = 1 + 0.250 = **1.25**.
   - The base code is the barcode with positions 7 through 11 replaced by five zeros:
     `2134567` + `00000` + `8` = **`2134567000008`**.
   - The pattern becomes `21.....` + `00000` + `.` = `21.....00000.`.
   - Matching `21.....00000.` against the first thirteen characters of `2134567000008`: `21` matches
     `21`, the five dots match `34567`, the five zeros match `00000`, the final dot matches `8`.
     **Match.**
4. The rule's type is `product`, not `alias`, so the result is assembled:
   - encoding `ean13`, type `product`, value **1.25**, code **`2134567012508`**;
   - because the encoding is `ean13`, the base code is sanitised: `2134567000008` truncated and
     padded to thirteen characters is itself; its recomputed check digit is 0 (even sum
     0+0+0+6+4+1 = 11, odd sum 0+0+7+5+3+2 = 17, weighted total 11 × 3 + 17 = 50, check digit
     (10 − 0) mod 10 = 0); so the base code is **`2134567000000`**.

The consuming domain then looks up the product whose stored barcode is `2134567000000` and records a
quantity of 1.25 kilograms.

### 10.7 Decoding a uniform resource identifier

Electronic product codes may be scanned as uniform resource identifiers. They are decoded before any
rule is consulted.

**Input.** A string beginning with `urn:`. **Output.** A list of typed results.

1. Split the string on colons. Take the last two pieces and trim the whitespace from each: the first
   is the **identifier kind**, the second is the **data**.
2. Split the data on full stops into a list of fields.
3. Dispatch on the identifier kind:

| Kind | Meaning | Handling |
|---|---|---|
| `lgtin` | lot-level Global Trade Item Number | trade-item decoding on all fields |
| `sgtin` | serialised Global Trade Item Number | trade-item decoding on all fields |
| `sgtin-96` | ninety-six-bit serialised trade item | trade-item decoding on the fields **after the first**, which is a filter value |
| `sgtin-198` | one-hundred-ninety-eight-bit serialised trade item | same as `sgtin-96` |
| `sscc` | Serial Shipping Container Code | logistic-unit decoding on all fields |
| `sscc-96` | ninety-six-bit Serial Shipping Container Code | logistic-unit decoding on the fields after the first |

Any other kind leaves the string unchanged, and it then goes through ordinary parsing.

**Trade-item decoding.** The three fields are the company prefix, the item reference with its
indicator digit in front, and the tracking number.

```formula
product_barcode_without_check = indicator_digit
                              + company_prefix
                              + item_reference_without_indicator
product_barcode = product_barcode_without_check
                + check_digit( product_barcode_without_check followed by "0" )
```

Two results are produced: one of type `product` whose code and value are the product barcode, and
one of type `lot` whose code and value are the tracking number. Both carry the original uniform
resource identifier as their base code and an empty encoding.

**Logistic-unit decoding.** The two fields are the company prefix and the serial reference with its
extension digit in front.

```formula
container_code_without_check = extension_digit + company_prefix + serial_reference_without_extension
container_code = container_code_without_check
               + check_digit( container_code_without_check followed by "0" )
```

One result is produced, of type `package`, whose code and value are the container code.

**Worked examples.**

| Scanned string | Fields | Result |
|---|---|---|
| `urn:epc:class:lgtin : 4012345.012345.998877` | prefix `4012345`, item `012345`, tracking `998877` | indicator `0`, so the code before the check digit is `0` + `4012345` + `12345` = `0401234512345`; the check digit of `04012345123450` is 6; product code **`04012345123456`**, lot **`998877`** |
| `urn:epc:id:sgtin:9521141.012345.4711` | prefix `9521141`, item `012345`, serial `4711` | code before check `0` + `9521141` + `12345` = `0952114112345`; check digit 4; product **`09521141123454`**, lot **`4711`** |
| `urn:epc:tag:sgtin-96 : 1.358378.0728089.620776` | filter `1` dropped; prefix `358378`, item `0728089`, serial `620776` | indicator `0`, code before check `0` + `358378` + `728089` = `0358378728089`; check digit 8; product **`03583787280898`**, lot **`620776`** |
| `urn:epc:id:sscc:952656789012.03456` | prefix `952656789012`, serial reference `03456` | extension `0`, code before check `0` + `952656789012` + `3456` = `09526567890123456`; check digit 8; package **`095265678901234568`** |

Note that the last two pieces of the colon-split are taken, so the number of colon-separated
namespace segments before them does not matter, and that the trimming allows the whitespace some
scanners insert around the final colon.

---

## 11. Barcodes — the Global Standards One nomenclature

### 11.1 What changes

Under a Global Standards One nomenclature a scanned string is not one code but a concatenation of
*(application identifier, value)* pairs. Fixed-length values run straight into the next pair;
variable-length values are terminated by a separator. Parsing is therefore a decomposition loop, and
the result is an ordered list rather than a single record.

Only rules whose encoding is `gs1-128` participate.

### 11.2 The separator

```formula
separator_group = "(?:" + configured_separator_expression + ")?"
```

when the nomenclature configures one, and the optional single non-printing group-separator character
otherwise. The shipped default expression is `(Alt029|#|\x1D)`, which accepts the literal text
`Alt029`, the number sign, or the non-printing group-separator character. In every case the group is
**optional**, so a fixed-length field that is not followed by a separator still matches.

### 11.3 Stripping the symbology identifier

Some scanners prefix the data with a symbology identifier. Before decomposing, if the string starts
with any one of `]C1`, `]e0`, `]d2`, `]Q3`, `]J1`, or the non-printing group-separator character,
that prefix is removed — the **first** matching prefix only, and only once.

### 11.4 Interpreting one matched pair

**Input.** A rule and a successful match whose first group is the application identifier and whose
second group is the value string. **Output.** A record with the rule, the rule's type, the
application identifier, the value string and the interpreted value — or nothing, meaning "this rule
does not really apply, try the next".

By content type:

**Measure.**

1. Let the **decimal position** be zero.
2. If the rule's decimal-usage flag is set, the decimal position is the **last character of the
   matched application identifier**, read as a digit.
3. If the decimal position is greater than zero:

   ```formula
   value = value_string_without_its_last_decimal_position_characters
         + "."
         + the_last_decimal_position_characters_of_value_string
   ```

   read as a decimal number.
4. Otherwise the value is the value string read as a whole number.
5. Any failure in the above raises:

   > There is something wrong with the barcode rule "*the rule name*" pattern.
   > If this rule uses decimal, check it can't get sometime else than a digit as last char for the
   > Application Identifier.
   > Check also the possible matched values can only be digits, otherwise the value can't be casted
   > as a measure.

**Numeric identifier.**

1. Left-pad the value string with zeros to eighteen characters and compute the check digit of the
   padded string.
2. If that digit is not equal to the value string's last character, return nothing — the rule does
   not apply, and the loop will try the next rule.
3. Otherwise the value is the value string, kept as a string including its check digit.

**Date.**

1. If the value string is not exactly six characters long, return nothing.
2. Otherwise the value is the date obtained by the conversion of section 11.5.

**Alphanumeric name.** The value is the value string unchanged.

### 11.5 Converting a six-digit date

**Input.** Six digits, two for the year within its century, two for the month, two for the day.
**Output.** A calendar date.

1. Let **current year** be the current calendar year and **current century** be `floor_div(current
   year, 100)`.
2. Let **difference** be the two-digit year minus `current_year mod 100`.
3. Choose the century:

```formula
century = current_century − 1    when   51 ≤ difference ≤ 99
century = current_century + 1    when  −99 ≤ difference ≤ −50
century = current_century        otherwise
year    = century × 100 + two_digit_year
```

   This is the standard century-determination rule: a two-digit year more than fifty years in the
   future is read as belonging to the previous century, and one more than fifty years in the past as
   belonging to the next.
4. If the day digits are `00`, the day is not given: the date is the **last day of that month** in
   that year, accounting for leap years.
5. Otherwise the date is that year, month and day. If the combination is not a valid calendar date,
   raise:

   > A Global Standards One barcode nomenclature pattern was matched. However, the barcode failed to be converted to
   > a valid date: '*the underlying parser's message*'

**Worked examples**, evaluated with a current year of 2026 (so current century 20, current year
within century 26):

| Six digits | Difference | Century | Result |
|---|---|---|---|
| `151020` | 15 − 26 = −11 | 20 | 20 October 2015 |
| `520300` | 52 − 26 = 26 | 20 | day digits are `00` → last day of March 2052 → 31 March 2052 |
| `200200` | 20 − 26 = −6 | 20 | day digits are `00` → last day of February 2020, a leap year → 29 February 2020 |
| `410551` | 41 − 26 = 15 | 20 | month 05, day 51 → not a valid date → the conversion error above |
| `800101` | 80 − 26 = 54 | 20 − 1 = 19 | 1 January 1980 |
| `700101` | 70 − 26 = 44 | 20 | 1 January 2070 |

### 11.6 The decomposition loop

**Input.** A scanned string and a Global Standards One nomenclature. **Output.** An ordered list of
records, or nothing at all when the string cannot be fully decomposed.

1. Build the separator group (section 11.2).
2. Strip the symbology identifier (section 11.3).
3. Let **results** be empty.
4. While the remaining string is non-empty:
   1. Find the next rule: for each `gs1-128` rule of the nomenclature, in ascending sequence then
      identifier, attempt to match, anchored at the start of the remaining string, the rule's pattern
      followed by the separator group. Accept the first rule whose match succeeds **and** yields at
      least two capturing groups **and** whose interpretation (section 11.4) returns a record. The
      new remaining string is what follows the end of the match, separator included.
   2. If no rule was found, or the remaining string did not shrink, **abandon the whole
      decomposition** and return nothing. A partially decomposable string is not partially accepted.
   3. Append the record to **results** and continue.
5. Return **results**.

Two consequences deserve emphasis. First, a rule whose interpretation returns nothing — a numeric
identifier with a bad check digit, a date with the wrong length — does not stop the loop; the loop
simply tries the next rule at the same position. Second, because matching is anchored and greedy,
the ordering of the rules by sequence is what disambiguates application identifiers that are
prefixes of one another.

### 11.7 Worked example — a barcode carrying the product, batch and expiry application identifiers

The label on a case of vaccine vials carries, in Global Standards One one-hundred-twenty-eight
symbology:

```
(01) 94019097685457        Global Trade Item Number
(10) LOT-A42               batch or lot number          [variable length → separator follows]
(17) 261231                expiration date, 31 December 2026
```

encoded as the character sequence

```
<GS> 0 1 9 4 0 1 9 0 9 7 6 8 5 4 5 7 1 0 L O T - A 4 2 <GS> 1 7 2 6 1 2 3 1
```

where `<GS>` is the non-printing group-separator character. Written as one string with the
separators shown as `<GS>`:

```
<GS>019401909768545710LOT-A42<GS>17261231
```

**Decomposition, using the shipped rule set** (rules in sequence order: `00` at 100, `01` at 101,
`02` at 102, `410` at 110, `413` at 113, `414` at 114, `10` at 125, `21` at 126, `13` at 137, `15`
at 138, `17` at 139, `30` at 300, `37` at 305, then the measure rules):

**Step 0 — strip the symbology identifier.** The string begins with the group-separator character,
which is in the strip list, so it is removed once. The remaining string is
`019401909768545710LOT-A42<GS>17261231`.

**Step 1.**

- Rule `00`, pattern `(00)(\d{18})`, anchored: the string starts `01`, not `00`. No match.
- Rule `01`, pattern `(01)(\d{14})`, anchored: matches `01` then the fourteen digits
  `94019097685457`. Two groups. Interpretation: content type numeric identifier. Left-pad
  `94019097685457` with four zeros to eighteen characters: `000094019097685457`. Its check digit:
  drop the last character and reverse → `5,4,5,8,6,7,9,0,9,1,0,4,9,0,0,0,0`; even positions
  5+5+6+9+9+0+9+0+0 = 43, odd positions 4+8+7+0+1+4+0+0 = 24; weighted total 43 × 3 + 24 = 153;
  check digit (10 − 3) mod 10 = **7**. The value string's last character is `7`. They agree, so the
  record is produced.
- Record: application identifier `01`, type `product`, value `94019097685457`.
- The separator group matches the empty string (the next character is `1`, not a separator), so the
  match ends right after the fourteenth digit. Remaining string: `10LOT-A42<GS>17261231`.

**Step 2.**

- Rules `00`, `01`, `02`, `410`, `413`, `414` do not match the leading `10`.
- Rule `10`, pattern `(10)([!"%-/0-9:-?A-Z_a-z]{0,20})`, anchored: matches `10`, then the character
  class greedily consumes `LOT-A42` — the hyphen, the uppercase letters and the digits are all in the
  class — and stops at the group-separator character, which is not. The optional separator group then
  consumes that character.
- Interpretation: content type alphanumeric name, so the value is the value string unchanged.
- Record: application identifier `10`, type `lot`, value `LOT-A42`.
- Remaining string: `17261231`.

**Step 3.**

- Rules up to `15` do not match the leading `17`. (Rule `21` does not; rule `13` does not; rule `15`
  does not.)
- Rule `17`, pattern `(17)(\d{6})`: matches `17` then `261231`.
- Interpretation: content type date, value string length six. Conversion with a current year of
  2026: difference is 26 − 26 = 0, so the century is 20 and the year is 2026; the day digits are
  `31`, not `00`; 31 December 2026 is a valid date.
- Record: application identifier `17`, type `expiration_date`, value **31 December 2026**.
- Remaining string is empty; the loop ends.

**Result** — an ordered list of three records:

| Order | Application identifier | Type | Value string | Interpreted value |
|---|---|---|---|---|
| 1 | `01` | `product` | `94019097685457` | `94019097685457` |
| 2 | `10` | `lot` | `LOT-A42` | `LOT-A42` |
| 3 | `17` | `expiration_date` | `261231` | 31 December 2026 |

The consuming domain resolves the first record to a product — see section 11.9 for the unpadding
that makes a fourteen-digit trade item number find a thirteen-digit stored barcode — creates or
finds the lot named `LOT-A42`, and sets its expiration date to 31 December 2026.

**What happens without the separator.** Had the label omitted the separator after the lot number,
the string after step 1 would have been `10LOT-A4217261231`. The character class of rule `10`
accepts digits, so the greedy match of up to twenty characters would have consumed
`LOT-A4217261231` — fifteen characters — and the lot would have been read as `LOT-A4217261231` with
no expiry at all. This is exactly the failure the separator prevents, and it is why the nomenclature
warns that the separator expression must not match the beginning or the end of any rule's pattern.

**An alternative encoding that needs no separator.** Placing the fixed-length field before the
variable-length one — `(01)94019097685457(17)261231(10)LOT-A42` — decomposes correctly without any
separator, because after the trade item number the remaining string begins `17`, which rule `10`
cannot match, and after the date the remaining string is `10LOT-A42`, whose greedy match reaches the
end of the string anyway.

### 11.8 Worked example — the four-element label from the standard's own illustration

The string `<GS>01940190976854571033650100138<GS>310200200415131018` decomposes to:

| Order | Application identifier | Rule | Value string | Interpreted value |
|---|---|---|---|---|
| 1 | `01` | Global Trade Item Number | `94019097685457` | `94019097685457` |
| 2 | `10` | Batch or lot number | `33650100138` | `33650100138` |
| 3 | `3102` | Net weight, kilograms | `002004` | **20.04** |
| 4 | `15` | Best before date | `131018` | 18 October 2013 |

The third record is the interesting one. Its rule's pattern is `(310[0-5])(\d{6})` and its
decimal-usage flag is set, so the decimal position is the last character of the matched application
identifier `3102`, that is **2**. The value is therefore the value string `002004` split two digits
from the right:

```formula
value = "0020" + "." + "04"  =  20.04
```

and its unit is the rule's associated unit, kilograms.

Two further decimal examples with a rule whose pattern is `(304)(\d{5,8})` and whose decimal-usage
flag is set — so the decimal position is always 4:

| Scan | Value string | Split | Value |
|---|---|---|---|
| `30400000` | `00000` | `0` and `0000` | 0.0 |
| `30418789` | `18789` | `1` and `8789` | 1.8789 |
| `3041515000` | `1515000` | `151` and `5000` | 151.5 |

and with the decimal-usage flag cleared on a rule whose pattern is `(300)(\d{5,8})`:

| Scan | Value string | Value |
|---|---|---|
| `30000000` | `00000` | 0 |
| `30018789` | `18789` | 18789 |
| `3001515000` | `1515000` | 1515000 |

### 11.9 Search preprocessing — unpadding

A stored product barcode is usually thirteen digits, while a scanned Global Standards One trade item
number is fourteen with a leading zero. Searching for the scan verbatim would find nothing. The
domain therefore rewrites barcode searches when the acting company's nomenclature is a Global
Standards One nomenclature.

**Input.** A search condition set, a list of the result types the caller cares about, and the name of
the field being searched (the barcode field by default). **Output.** A rewritten condition set.

The rewrite is skipped entirely when the caller sets the suppression flag `skip_preprocess_gs1`
("skip preprocess Global Standards One"). Each individual condition is rewritten as follows:

1. If the condition is not on the named field, or the company's nomenclature is not a Global
   Standards One nomenclature, leave it alone.
2. If the condition's value is empty, leave it alone.
3. If the comparison is membership or non-membership over more than one value, rewrite each value
   independently as an equality condition, combine them with "or", and negate the combination for
   non-membership.
4. If the comparison is membership or non-membership over exactly one value, reduce it to equality
   or inequality on that value.
5. If the comparison is not one of contains, does-not-contain, equals or differs, leave the condition
   alone.
6. Parse the value as a barcode. Any parsing or validation failure yields an empty parse.
7. Let the **replacing comparison** be "contains" when the original comparison was contains or
   equals, and "does not contain" otherwise.
8. For each record of the parse, in order:
   - if its type is not in the caller's list of interesting types, skip it;
   - if its type is `lot`, return a condition comparing the field to the record's value with the
     **original** comparison (lot names are not numeric and must not be unpadded);
   - otherwise, if the record's value, read as text, consists of zero or more leading zeros followed
     by at least one digit, return a condition comparing the field to those digits without the
     leading zeros, using the replacing comparison;
   - otherwise stop looking at further records.
9. If the parse produced nothing at all, and the raw value consists of **at least one** leading zero
   followed by at least one digit, return a condition comparing the field to the digits without the
   leading zeros, using the replacing comparison.
10. Otherwise leave the condition alone.

Note the difference between steps 8 and 9: a parsed value may have **zero** leading zeros and is
still switched to a contains comparison, while an unparsed value is only rewritten when it actually
has a leading zero to strip.

**Worked example.** The company uses the shipped Global Standards One nomenclature. A user scans
`0194019097685457` into a product search box, producing the condition "barcode equals
`0194019097685457`".

1. The condition is on the barcode field and the nomenclature is a Global Standards One one.
2. The comparison is equality, which is in the allowed set.
3. Parsing yields one record: application identifier `01`, type `product`, value `94019097685457`.
4. The replacing comparison is "contains".
5. The record's type `product` is in the caller's list. Its value `94019097685457` matches "zero or
   more leading zeros then digits" with zero leading zeros, so the unpadded digits are
   `94019097685457`.
6. The condition becomes "barcode contains `94019097685457`".

A product stored with the barcode `9401909768545` — the thirteen-digit European Article Number whose
fourteen-digit form is `09401909768545…` — would **not** be found by this particular rewrite, since
the unpadded string still has fourteen digits. The rewrite is designed for the opposite direction:
a scan of `00000012345670` unpads to `12345670` and finds the eight-digit stored barcode.

---

## 12. Names and displays

### 12.1 The complete name of a category

```formula
complete_name(category) = complete_name(parent) + " / " + name(category)      when a parent exists
complete_name(category) = name(category)                                      otherwise
```

The computation is recursive and stored, so renaming a category rewrites the complete name of every
descendant.

**Worked example.** Categories "All" (no parent), "Saleable" (parent All), "Office Furniture"
(parent Saleable) give complete names `All`, `All / Saleable`, `All / Saleable / Office Furniture`.
Renaming "Saleable" to "Sellable" rewrites the second to `All / Sellable` and the third to
`All / Sellable / Office Furniture`.

### 12.2 The combination name and the variant display name

```formula
combination_name = join_with( ", ",
    names of the combination members, after removing
      (a) members whose attribute never creates variants, and
      (b) members that come from a single-value line )
```

Rule (b) needs the active-state subtlety: if **every** member of the set is active, a line counts as
single-value when it has exactly one *active* materialised value; if **any** member is archived, the
count includes archived values. This makes an archived variant keep the name it had when it was
created, even after the line acquired more values.

```formula
variant_base_name = template_name                                    when combination_name is empty
variant_base_name = template_name + " (" + combination_name + ")"    otherwise
```

**Worked example.** Office Chair with Size {S, M, L} and Colour {Black, White}:

- *Office Chair (S, Black)* — both lines have more than one value, so both members show.

Now add a third attribute **Material** offering only **Oak**. Variant generation's single-value
repair adds *Material: Oak* to every existing variant without recreating anything. The display names
are unchanged: *Material: Oak* comes from a single-value line and is filtered out. The variant is
still *Office Chair (S, Black)*.

Now add **Walnut** to the Material line. The line is no longer single-value, so six new variants are
created (three sizes × two colours × two materials, minus the six already existing which get the
Walnut ones added), and the names become *Office Chair (S, Black, Oak)*, *Office Chair (S, Black,
Walnut)* and so on. The previously archived variants, if any, keep their two-value names because
their members were archived at the time.

### 12.3 The variant reference and the vendor-specific display

The **reference** of a variant, in the context of a partner:

1. Start from the internal reference.
2. If the reader may read vendor price lines, walk the variant's vendor lines that belong to the
   acting companies. For each line whose partner is the context partner:
   - skip the line if it names a different variant;
   - otherwise set the reference to the line's vendor product code if it has one, or back to the
     internal reference if it has none;
   - stop walking as soon as a line that names **this exact variant** has been used.

The consequence is that a variant-specific vendor line wins over a template-wide one, and the last
template-wide line of that partner wins over earlier ones.

The **customer reference** of a variant, in the context of a partner: for the first vendor line whose
partner matches, the vendor's product name or, failing that, the internal reference or, failing that,
the product name, prefixed by the bracketed reference when there is one. If no vendor line matches,
the plain display name.

### 12.4 Searching for a product by name

A name search on variants is deliberately staged, from most specific to least, so that scanning a
barcode or typing a code finds one record rather than a hundred:

1. If the search term is empty, fall back to the framework's default.
2. If the comparison is one of equals, contains, contains-with-wildcards, matches or
   matches-with-wildcards:
   - search on internal reference **exactly equal** to the term; if that finds nothing,
   - search on barcode **exactly equal** to the term.
3. If nothing was found and the comparison is positive:
   - search on internal reference with the caller's comparison;
   - then, if the limit is not yet reached, search on name with the caller's comparison, excluding
     the identifiers already found. (The two searches are kept separate deliberately: combining them
     with "or" is markedly slower on a large catalogue because the name lives in a translation
     structure.)
4. If nothing was found and the comparison is negative: search for records whose name satisfies the
   negative comparison **and** whose internal reference either satisfies it or is unset.
5. If nothing was found and the comparison is positive and the term contains a bracketed section,
   search on internal reference exactly equal to the text inside the first pair of brackets. This is
   what makes pasting a whole display name such as `[FURN-001] Office Chair` work.
6. If nothing was found and a partner is named in the context, search on the vendor lines of that
   partner whose vendor product code or vendor product name satisfies the comparison.
7. Return the identifiers and display names of what was found, read as a privileged reader so that
   vendor-derived display names are complete.

The **display-name search condition** used by generic searches is broader: it matches on the
template name, on the internal reference, on the barcode (for membership comparisons and for
positive contains-style comparisons), on the text inside brackets of each membership value, and on
the vendor product code and name when a partner is in the context. For positive comparisons the
result is the union of separate searches; for negative comparisons it is the conjunction of the
individual negative conditions.

### 12.5 The zoomability flag

```formula
can_image_1024_be_zoomed = ( the original image exists )
                       and ( the original image is strictly larger, in pixels,
                             than the one-thousand-and-twenty-four-pixel derivative )
```

evaluated with binary field contents actually loaded rather than substituted by their size. On a
variant the flag is the variant's own when the variant has its own image, and the template's
otherwise.

---

## 13. Expiry dates

### 13.1 The four day counts and the four dates

A product may declare four whole-number day counts. Three of them are measured **backwards from the
expiration date**; only the first is measured forwards from a receipt.

| Day count on the product | Date on the lot | Relation |
|---|---|---|
| Expiration day count | Expiration date | receipt moment **plus** the count |
| Best-before day count | Best-before date | expiration date **minus** the count |
| Removal day count | Removal date | expiration date **minus** the count |
| Alert day count | Alert date | expiration date **minus** the count |

### 13.2 Computing the expiration date of a lot

The expiration date is computed from the product and is stored, but may be overwritten by a user.

```formula
expiration_date = current_moment + expiration_day_count days
```

applied only when the product uses expiration dates **and** the lot has no expiration date yet. When
the product does not use expiration dates, the field is cleared.

The "current moment" here is the server's current local moment at the instant of the computation,
not a date-only value: the resulting expiration date carries a time of day.

### 13.3 Computing the three derived dates of a lot

The computation depends on the three fields, the product and the previous stored state, and is
applied per lot:

1. If the product does not use expiration dates: clear the best-before date, the removal date and
   the alert date.
2. Otherwise, if the lot has an expiration date:
   - **Initialisation branch** — taken when the product has changed since the last stored state,
     **or** none of the three derived dates is set, **or** the expiration date has just been set
     where there was none before:

     ```formula
     best_before_date = expiration_date − best_before_day_count days
     removal_date     = expiration_date − removal_day_count days
     alert_date       = expiration_date − alert_day_count days
     ```

   - **Shift branch** — taken when the expiration date is being changed and there was a previous
     expiration date:

     ```formula
     shift            = new_expiration_date − previous_expiration_date
     best_before_date = previous_best_before_date + shift      (only if it was set)
     removal_date     = previous_removal_date     + shift      (only if it was set)
     alert_date       = previous_alert_date       + shift      (only if it was set)
     ```

     A derived date that was empty stays empty.
3. If the lot has no expiration date and the product does use expiration dates, none of the three
   derived dates is touched.

The shift branch preserves manual adjustments: if a warehouse clerk moved the removal date closer to
the expiration date than the product's default, postponing the expiration keeps that relative
position.

### 13.4 Worked example — an expiry date computation

A product **"Fresh Yoghurt 500 g"** declares:

| Setting | Value |
|---|---|
| Use expiration date | yes |
| Expiration day count | 30 |
| Best-before day count | 5 |
| Removal day count | 3 |
| Alert day count | 10 |

**Receipt.** A goods receipt is validated on **11 September 2026 at 09:30**, creating the lot
`YOG-2609-A`. At the moment the lot record is created:

```
expiration_date  = 11 September 2026 09:30  +  30 days  =  11 October 2026 09:30
```

The derived-date computation then runs. The product has just been set, so the initialisation branch
applies:

```
best_before_date = 11 October 2026 09:30  −  5 days  =   6 October 2026 09:30
removal_date     = 11 October 2026 09:30  −  3 days  =   8 October 2026 09:30
alert_date       = 11 October 2026 09:30  − 10 days  =   1 October 2026 09:30
```

The four dates on the lot are therefore:

| Date | Value |
|---|---|
| Alert date | 1 October 2026 09:30 |
| Best-before date | 6 October 2026 09:30 |
| Removal date | 8 October 2026 09:30 |
| Expiration date | 11 October 2026 09:30 |

**A manual adjustment.** A quality inspector shortens the removal date by one day, to **7 October
2026 09:30**. Nothing else changes.

**A supplier certificate extends the shelf life.** The expiration date is written to **21 October
2026 09:30**. The shift branch applies, because there was a previous expiration date:

```
shift            = 21 October 2026 09:30 − 11 October 2026 09:30 = 10 days
best_before_date =  6 October 2026 09:30 + 10 days = 16 October 2026 09:30
removal_date     =  7 October 2026 09:30 + 10 days = 17 October 2026 09:30
alert_date       =  1 October 2026 09:30 + 10 days = 11 October 2026 09:30
```

The inspector's one-day tightening is preserved: the removal date is still four days before
expiration rather than the product default of three.

**The expiry alert flag.**

```formula
expiry_alert = ( expiration_date exists ) and ( expiration_date ≤ current_moment )
```

On **11 October 2026 at 09:31** with the original dates, the flag would have been true; with the
extended dates it becomes true only on 21 October 2026 at 09:30.

**Availability.** From the removal date onwards the lot stops counting as fresh:

```formula
available_quantity(quant) = 0
    when the product uses expiration dates
     and the quant's removal date exists
     and the quant's removal date ≤ current_moment
```

otherwise the ordinary available-quantity formula of `../inventory-operations/` applies. On **17
October 2026 at 09:31** (with the extended dates) the quantity held under lot `YOG-2609-A` stops
being available for new reservations.

### 13.5 Dates on a move line

A move line computes its own expiration date, because the lot may not exist yet at the moment the
line is prepared.

```
if the line's quant has a lot, or the line has a lot:
    expiration_date = that lot's expiration date
else if the line's operation type creates lots:
    if the product uses expiration dates:
        if the line has no expiration date yet:
            expiration_date = ( the transfer's scheduled moment, or today if there is none )
                              + expiration_day_count days
    else:
        expiration_date = empty
```

and, for the removal date:

```
if the line's lot has a removal date:
    removal_date = that lot's removal date
else if the line's operation type creates lots:
    if the product uses expiration dates and the line has an expiration date:
        removal_date = expiration_date − removal_day_count days
    else:
        removal_date = empty
```

When the line eventually creates its lot, the line's expiration date is copied onto the new lot.

### 13.6 Generating a batch of lot lines

When a user asks for a run of serial numbers on a move, the generated line values receive a default
expiration date:

```formula
default_expiration_date = ( the transfer's scheduled moment, or today's date if there is none )
                        + expiration_day_count days
```

and for the serial-number command generator the base is today's date rather than the transfer's
scheduled moment. A value already present in the prepared line is never overwritten.

### 13.7 Reading a typed date out of free text

When a barcode-driven screen receives a string it cannot otherwise classify, it tries to read it as a
date, and if that succeeds it becomes the line's expiration date — unless the move's product does
not use expiration dates, in which case the string is ignored rather than rejected.

The day-month-year ambiguity is resolved by examining the first two components of the string, split
on hyphens, slashes and spaces:

1. If fewer than two components are found, no option is inferred from this string; try the next.
2. If the first component contains a letter, it is a month name: no option is set (month-first is the
   default reading) and the search stops.
3. If the first component, read as a number, is greater than 31, it must be a year: set year-first
   and stop.
4. If the first component, read as a number, is greater than 12, and the second component either
   contains a letter or is at most 12, the first must be a day: set day-first and stop.
5. Otherwise the string is ambiguous. Fall back to the reader's language date format: if it begins
   with a month placeholder, set nothing and return immediately; if it begins with a day placeholder,
   set day-first and stop; if it begins with a year placeholder, set year-first and stop.

### 13.8 The expiry reminder

Run as part of the scheduled replenishment work (see [configuration.md](configuration.md)):

1. Select every lot whose alert date is at or before **today's date** and whose reminded flag is
   false.
2. Restrict to the lots that have at least one stock quantity record with a strictly positive
   quantity in a location whose usage is internal.
3. For each remaining lot, schedule a to-do activity with the summary "Alert Date Reached" and the
   note "The alert date has been reached for this lot/serial number", assigned to: the product's
   responsible user as seen from the lot's company, or failing that the product's responsible user
   without a company restriction, or failing that the system superuser.
4. Set the reminded flag on **every lot selected in step 1** — including those filtered out in step 2
   — so that a lot never produces two reminders even if its alert date is later moved backwards.

Note the asymmetry in step 4: lots with no internal stock are marked as reminded without receiving an
activity.

---

## 14. Miscellaneous computations

### 14.1 The product count of a category

```formula
product_count(category) = sum over d in descendants(category) including itself of
                              number_of_templates_whose_category_is_exactly_d
```

The grouping is performed once over the whole descendant set of the categories being computed, then
each category re-reads the descendants of itself and sums. Archived templates are excluded, because
the grouping runs with the default active-record filter.

### 14.2 The label sheet page count

```formula
labels_per_page = columns × rows
total_labels    = sum over products of the number of copies requested for that product
page_count      = floor_div( total_labels − 1 , labels_per_page ) + 1
```

with the grid dimensions derived from the format string as described in [entities.md](entities.md),
section 19.

**Worked example.** Format `4x12xprice` gives four columns and twelve rows, so forty-eight labels per
page. Printing three copies each of twenty products gives sixty labels:
`floor_div(60 − 1, 48) + 1 = floor_div(59, 48) + 1 = 1 + 1 = 2` pages.

Printing exactly forty-eight labels gives `floor_div(47, 48) + 1 = 0 + 1 = 1` page; forty-nine gives
`floor_div(48, 48) + 1 = 1 + 1 = 2`.

### 14.3 Selecting a vendor for a product

Although vendor pricelists belong to `../pricing-and-pricelists/`, the selection is invoked from the
catalog and is specified here for completeness.

**Preparation.** Take the variant's vendor lines as a privileged reader, keep those whose company is
empty or is the acting company, whose partner is active, and which either name no variant or name
this variant. Sort them by sequence ascending, then by minimum quantity **descending**, then by
price ascending, then by identifier ascending.

**Filtering**, given a partner, a quantity, a date, a unit and optional parameters:

1. The date defaults to today in the reader's time zone. The comparison precision is the `Product
   Unit` precision.
2. For each prepared line, in order:
   - convert the quantity into the line's unit when a unit was given and it differs from the line's
     unit;
   - skip the line if its start date is after the date;
   - skip the line if its end date is before the date;
   - skip the line if the caller demanded a forced unit and the line's unit is neither that unit nor
     the product's own unit;
   - skip the line if a partner was given and the line's partner is neither that partner nor that
     partner's parent;
   - skip the line if the converted quantity is strictly less than the line's minimum quantity, at
     the `Product Unit` precision;
   - skip the line if it names a variant other than this one;
   - otherwise keep it.

**Selection.** From the kept lines, keep the first one and then every subsequent line belonging to
the **same partner** as the first. Sort that subset by the requested ordering — by default the
discounted price converted into the acting company's currency at the given date without rounding,
then sequence, then identifier; when the caller names another field, that field first, then the
discounted price, then sequence, then identifier — and return the first.

---

## 15. Rounding and precision summary

| Quantity | Precision | Rounding |
|---|---|---|
| Sales price, cost, extra price, combo base price, combo item extra price | `Product Price`, two digits by default | Half away from zero at the precision |
| Prorated combo share | The document currency's rounding multiple | Half away from zero at the multiple; the remainder is given to the last choice group |
| Quantity comparisons in vendor selection | `Product Unit`, two digits by default | Comparison only |
| Volume | `Volume`, two digits by default | Half away from zero |
| Weight | `Stock Weight`, two digits by default | Half away from zero |
| Discount percentages | `Discount`, two digits by default | Half away from zero |
| Barcode-derived measures | None; the value is exactly the digits read, with the decimal point inserted | No rounding |
| Expiry dates | Whole days added to or subtracted from a moment; the time of day is preserved | None |

The currency conversions used in this domain (combo base price, combo proration, matrix header
cells, vendor selection ordering) all delegate to `../multi-currency/`; the combo base price and the
matrix header cell convert **at the current moment**, the combo proration and the vendor ordering
convert **at the document date**, and the vendor ordering converts **without rounding**.
