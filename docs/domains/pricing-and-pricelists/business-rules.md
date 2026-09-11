# Pricing and Price Lists — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behaviour of
the domain, with the exact user-facing message text where one exists.

Rules are grouped by the entity or operation they guard. Each rule states **when** it runs, **what
it tests**, **what it does when the test fails**, and **what a rebuild must do** when the platform
leaves something implicit.

---

## 1. How the rules are enforced

The domain uses four enforcement mechanisms, and the difference matters:

| Mechanism | Runs | Can be bypassed by | Failure |
|---|---|---|---|
| **Stored constraint** | On every creation and on every modification of the fields it watches, whether by a user, by an import, or by another part of the platform | nothing | The whole transaction is rolled back and an error is shown. |
| **Deletion guard** | Immediately before a delete | nothing (it runs even for an administrator) | The delete is refused and an error is shown. |
| **Form change handler** | Only inside an interactive form, when the user changes one of the fields it watches | any programmatic write, any import, any remote operation | Either a value is silently adjusted or an error is shown. |
| **Company consistency check** | Automatically on creation and modification for the fields declared company-checked | nothing | An error listing up to five inconsistencies is shown. |

A rebuild that implements the form change handlers as stored constraints will reject legitimate
imports. A rebuild that implements the stored constraints as form handlers will let corrupt data
in through the remote interface.

---

## 2. Price List

### 2.1 The name is required

**When.** Creation and modification.
**Test.** The name is not empty.
**Failure.** The standard "required field" error for the field labelled *Pricelist Name*.
**Note.** Names are **not** unique. Two price lists may have the same name; the display name
disambiguates them by appending the currency code.

### 2.2 The currency is required

**When.** Creation and modification.
**Test.** A currency is set.
**Failure.** The standard "required field" error for the field labelled *Currency*.
**Default.** The acting user's company's currency, so the field is normally filled in
automatically.

### 2.3 A price list cannot be deleted while another price list bases a rule on it

**When.** Immediately before deletion.
**Test.** Search, with elevated rights, for price list rules where all three hold:

1. the rule's base is "other price list";
2. the rule's base price list is one of the price lists being deleted;
3. the rule's own price list is **not** one of the price lists being deleted.

**Failure.** The deletion is refused with:

```
You cannot delete pricelist(s):
(<the display names of the referenced base price lists, one per line>)
They are used within pricelist(s):
<the display names of the price lists that hold those rules, one per line>
```

**Edge case — deleting a group together.** Because condition 3 excludes rules that live inside the
deleted set, a set of mutually referencing price lists can be deleted in one operation. Deleting
them one at a time fails.

**Edge case — self-reference.** A price list holding a rule based on itself is not deletable on its
own by condition 3 — it is, because the rule's own price list *is* in the deleted set. Such a
configuration cannot exist anyway; see rule 3.3.

**What a rebuild must do.** The guard must run with elevated rights, so that a user who cannot see
another company's price lists is still blocked by a reference from it.

### 2.4 A price list's rules must stay company-consistent when the company changes

**When.** Modification, when the company field is among the written values **and exactly one**
price list is being written.
**Test.** Re-run the company consistency check on every rule of the price list. Each rule's
product template, product variant and base price list must belong to the rule's (newly computed)
company or to no company.
**Failure.** The company inconsistency error of rule 7.1.
**Note.** The check is deliberately skipped when several price lists are written at once, because
a bulk company change is treated as an administrative operation. A rebuild should reproduce that,
including its consequence: a bulk write can leave inconsistent rules behind, and the next single
write on any of those price lists will then fail.

### 2.5 A price list restricted to a website must belong to that website's company

**When.** Creation and modification, only when the storefront capability is installed.
**Test.** Whenever both a website and a company are set, the website's company equals the price
list's company.
**Failure.**

```
Only the company's websites are allowed.
Leave the Company field empty or select a website from that company.
```

### 2.6 Archiving a currency archives its price lists

**When.** A currency is modified with its active flag set to false.
**Effect.** Every price list whose currency is that currency is archived. This is a side effect,
not a validation: nothing is refused.

### 2.7 Disabling the price list capability archives every price list

**When.** The settings are saved with the basic price list capability turned off.
**Effect.** **Every** price list is archived, with elevated rights. Before the save, the settings
form warns:

```
You are deactivating the pricelist feature. Every active pricelist will be archived.
```

The warning appears only when at least one active price list exists, and it is advisory: the user
may proceed.

**When the capability is turned on** and it was previously off, the automatic price list creation
of rule 2.8 runs.

### 2.8 Every company has a default price list when the capability is on

**When.** A company is created; the capability is turned on; a currency is activated for
multi-currency use.
**Effect.**

1. If the calling context asks to disable company price list creation, do nothing. (The company
   write path sets that flag while the currency is being changed, and re-runs the creation
   afterwards, so that the created price list carries the new currency.)
2. If the capability is off, do nothing.
3. Otherwise, for the companies concerned (or all companies when none is named):
   1. Find, ignoring the active flag, the price lists that have **no rules** and belong to those
      companies, and keep those whose currency equals their company's currency. Un-archive them.
   2. For every company not covered by step 3.1, create a price list with: name "Default",
      currency the company's currency, the company itself, and sequence **10**.

**What a rebuild must do.** Reproduce the un-archive-before-create order. A database where the
capability has been turned off and on again must end with the same "Default" price lists it
started with, not with duplicates.

### 2.9 Copying a price list

**When.** Duplication.
**Effect.** Unless the caller supplies a name, the copy's name becomes the original name followed
by a space and the word "(copy)" in parentheses. The rules are copied. The country groups are
copied. The website is copied.

---

## 3. Price List Rule

### 3.1 A rule based on another price list must name one

**When.** Creation and modification, watching the base and the base price list.
**Test.** No rule in the written set has a base of "other price list" without a base price list.
**Failure.**

```
A pricelist item with "Other Pricelist" as base must have a base_pricelist_id.
```

**Note.** The message names the storage field `base_pricelist_id` (the base price list) literally;
that is the text the platform produces.

**What the engine does if the state somehow exists.** The base-price computation falls through to
the catalogue price. The rule prices as if its base were the sales price. A rebuild must not crash
there.

### 3.2 The validity window must be ordered

**When.** Creation and modification, watching the start date and the end date. Also re-run
immediately in an interactive form when either date changes.
**Test.** For each rule that has **both** dates: the start date is strictly before the end date.
**Failure.**

```
<the rule's display name>: end date (<the end date, formatted for the user's language and time zone>) should be after start date (<the start date, formatted the same way>)
```

**Edge cases.**

- A rule with only a start date is valid.
- A rule with only an end date is valid.
- A rule with no dates is valid.
- Equal dates are **refused**: the test is strictly-before, so a window of zero length cannot be
  stored.

### 3.3 Price list bases must not form a cycle

**When.** Creation and modification, watching the base, the base price list and the own price
list.
**Test.** The depth-first walk of [`calculations.md`](calculations.md#86-cycle-protection).
Rules that are not based on another price list, that have no base price list, or that have no own
price list are skipped.
**Failure.**

```
Recursive pricelist rules detected: <the computed names of the rules along the cycle, joined by " ⇒ ">
```

**Why it is a write-time guard.** The price engine performs no cycle check and would recurse until
the runtime's stack is exhausted. The guard is the only protection. A rebuild **must** implement
it, and must implement the visited-edge memo, because a wide graph without the memo is
exponential.

**Worked refusals** are tabulated in
[`calculations.md`](calculations.md#86-cycle-protection).

### 3.4 The minimum margin must not exceed the maximum margin

**When.** Creation and modification, watching both margin fields.
**Test.** No rule in the written set has a minimum margin strictly greater than its maximum
margin.
**Failure.**

```
The minimum margin should be lower than the maximum margin.
```

**Edge cases.**

- Equal margins are allowed, and pin the price to exactly the base plus that margin.
- A margin of zero counts as "not configured" for the price computation but still participates in
  this comparison. A minimum margin of five with an unset (zero) maximum margin therefore
  **fails** the constraint, even though the computation would have ignored the maximum. To
  configure a floor with no ceiling, the maximum must be left at zero **and** the minimum must be
  at or below zero — or, in practice, the user must also set a maximum above the minimum.
  A rebuild must reproduce this, because it is a real and frequently encountered restriction.
- Negative margins are allowed in both fields, and mean "below the base price".

### 3.5 The rule's target must match its level

**When.** Creation and modification, watching the variant, the template and the category.
**Tests and failures.**

| Level | Test | Message on failure |
|---|---|---|
| `2_product_category` | a category is set | `Please specify the category for which this rule should be applied` |
| `1_product` | a product template is set | `Please specify the product for which this rule should be applied` |
| `0_product_variant` | a product variant is set | `Please specify the product variant for which this rule should be applied` |
| `3_global` | none | — |

### 3.6 The rounding step must be strictly positive

**When.** **Only** inside an interactive form, when the rounding step changes.
**Test.** No rule in the edited set has a rounding step that is both non-zero and strictly
negative.
**Failure.**

```
The rounding method must be strictly positive.
```

**Important.** This is a **form handler, not a stored constraint**. A negative rounding step can be
written programmatically or imported. The price engine will then call the rounding function with a
negative step, which the rounding function itself rejects as a programming error. A rebuild should
reproduce the form check and should also make the rounding function reject a non-positive step, so
that the failure mode is a loud error rather than a silently wrong price.

### 3.7 Target normalisation

**When.** Creation and modification. Not a validation — a silent normalisation.
**Effect.** Described in [`entities.md`](entities.md#25-consistency-enforced-on-write). The
irrelevant target fields are cleared according to the level, and on creation both the template and
the level are inferred when missing.

**Why it matters as a rule.** The candidate-rule search of the engine filters on the target fields.
A rule at the global level that still carried a stale product template would be excluded from the
search whenever that template is not among the products being priced, and would therefore silently
stop applying. The normalisation is what makes the search correct.

### 3.8 The markup and the discount are exact negations

**Invariant.** At all times, markup equals minus discount.

**Enforcement.** The markup is a computed, stored field whose value is the negated discount, with
an inverse that writes the negated markup into the discount. Writing either writes both.

**What a rebuild must do.** Either store one and derive the other, or store both and keep them
synchronised on every write path, including imports and remote operations. A rebuild that lets the
two drift will price cost-based rules wrongly, because the formula reads the markup when the base
is the cost and the discount otherwise.

### 3.9 Company consistency on a rule

**When.** Creation and modification.
**Test.** The rule's product template, product variant and base price list must each belong to the
rule's computed company or to no company. The rule's computed company is the price list's company
if there is one, else the product template's company.
**Failure.** The company inconsistency error of rule 7.1.

**Worked refusal.** A price list with **no** company cannot hold a rule whose base price list
belongs to a company: the rule's computed company is empty, so the base price list must also have
no company.

**Worked refusal.** A price list belonging to company one holds two rules, one based on a
company-less price list and one based on a price list of company one. Changing the price list's
company to company two fails: the second rule's base price list belongs to company one.

### 3.10 Deleting the price list deletes its rules

The link from a rule to its price list cascades on delete. Deleting a price list — when rule 2.3
allows it — removes its rules. Likewise deleting a product template, a product variant or a
product category removes the rules that target it, because all three links cascade.

**Consequence a rebuild must accept.** Deleting a product category silently deletes every pricing
rule that targeted it. There is no warning.

### 3.11 Archiving a product does not disable its rules

**Behaviour.** The price list form's rule list hides rules pointing at an archived product template
or an archived product variant. The engine's candidate search does **not** apply that filter.

**Consequence.** An archived product that is still referenced on an old order and re-priced will
still get its archived rule. This is faithful behaviour.

**Industry-standard default.** Where a rebuild wants a warning, the correct place is the archive
operation on the product, not the engine. The platform does not warn; a rebuild that adds a
warning must not change the pricing outcome.

---

## 4. The price computation

### 4.1 The engine raises no business error

**Invariant.** For any combination of stored data, the engine returns a number and a rule
identifier. It never refuses, never warns, never logs a business message.

The only failures possible are programming errors:

| Condition | Kind of failure |
|---|---|
| More than one price list supplied | Assertion: "expected singleton". |
| More than one currency supplied | Assertion: "expected singleton". |
| More than one unit supplied | Assertion: "expected singleton". |
| More than one product supplied to a per-product operation | Assertion: "expected singleton". |
| A cyclic price list graph | Unbounded recursion. Prevented by rule 3.3. |
| A negative or zero rounding step written programmatically | The rounding function's own assertion. Prevented in forms by rule 3.6. |

### 4.2 A quantity of zero

**Behaviour.** A quantity of zero is converted to zero by the quantity conversion's zero
short-circuit, and therefore fails every rule whose minimum quantity is positive. Rules with a
minimum quantity of zero still apply, because a zero minimum means "no condition".

**On a sales line.** The line substitutes **one** for a zero quantity before calling the engine,
precisely so that a line being composed prices at the single-unit price.

### 4.3 A negative quantity

**Behaviour.** A negative quantity converts to a negative quantity and is **below** every positive
minimum quantity, so quantity-break rules never apply to a negative line. Rules with a zero
minimum quantity apply, because the minimum is treated as "no condition" rather than as a
threshold of zero.

**Where negative quantities arise.** Refund-shaped lines and returns.

### 4.4 A product with no category and a category rule

**Behaviour.** The rule fails. A product with no category never matches a category rule, not even a
rule on the root category.

### 4.5 A template priced against a variant rule

**Behaviour.** A variant rule applies to a **template** only when the template has exactly one
variant and that variant is the rule's. With two or more variants the rule fails, and the engine
falls through to a less specific rule.

**Why.** Pricing a template is an aggregate question ("what does this product cost?") and a
variant rule cannot answer it when the variants differ.

### 4.6 A fixed price and a non-matching currency

**Behaviour.** A fixed price is returned in the number the rule stores, with no currency
conversion, even when the caller asked for another currency. See
[`calculations.md`](calculations.md#91-kind-fixed-price).

**What a rebuild must not do.** Convert it. Several plausible-looking rebuilds add the conversion
"for consistency" and then produce prices a hundredfold wrong in high-rate currencies.

### 4.7 A rule whose base price list is empty

**Behaviour.** The base-price computation's condition requires **both** that the base be "other
price list" **and** that a base price list be set. When the base price list is empty, the branch is
skipped and the catalogue-price branch runs.

### 4.8 A missing currency rate

**Behaviour.** The currency conversion falls back, in order, to: the most recent rate at or before
the date; the earliest rate of that currency; the value one. So a currency with no rates at all
converts one-for-one. This is the multi-currency domain's behaviour and the engine inherits it
unchanged.

### 4.9 Units in different trees

**Behaviour.** The engine's quantity conversion tolerates failure: when the document unit and the
product unit share no ancestor, the requested quantity is returned unchanged and is then compared
against the minimum quantity as if the units matched. The price conversion performs no check at
all and will produce a meaningless number.

**Industry-standard default.** A rebuild should prevent the situation upstream: a document line's
unit should be restricted to the units of the product's own unit category. The sales and purchasing
domains do restrict it; the engine does not.

---

## 5. Vendor Price

### 5.1 Required fields

| Field | Rule |
|---|---|
| Vendor | Required. |
| Product Template | Required. Filled in automatically from the variant when a variant is given. |
| Unit | Required. Defaulted from the variant's or the template's own unit, then never recomputed. |
| Quantity (minimum quantity) | Required. Defaults to zero. |
| Currency | Required. Defaults to the acting company's currency. |
| Lead Time | Required. Defaults to one. |

### 5.2 The variant must belong to the template

**When.** Interactive form, on changing the template.
**Effect.** A variant that is not among the template's variants is cleared. This is a silent
adjustment, not an error, and it does **not** run on a programmatic write: an import can store a
variant from another template, and the selection algorithm will then never match it, because it
tests the variant against the product being priced.

### 5.3 Template and variant stay synchronised

**When.** Creation and modification.
**Effect.** When a variant is written and no template is written in the same operation, the
template is filled in from the variant. There is no reverse rule: clearing the variant leaves the
template.

### 5.4 Company consistency

**When.** Creation and modification.
**Test.** The vendor, the product variant and the product template must belong to the offer's
company or to no company.
**Failure.** The company inconsistency error of rule 7.1.

### 5.5 Cascades

- Deleting the vendor deletes the offer.
- Deleting the product template deletes the offer.
- Deleting the product variant **sets the variant to nothing**, which widens the offer to all
  variants of the template. It does not delete the offer.

**A rebuild must note the asymmetry**: the variant link sets to nothing, the template link
cascades.

### 5.6 There is no uniqueness rule

Two offers from the same vendor, for the same product, with the same unit, the same minimum
quantity, the same price and overlapping validity windows are all legal. The selection algorithm
will pick one deterministically (by identifier, ultimately), and the other will simply never win.

### 5.7 There is no validity-window ordering constraint

Unlike the price list rule, a vendor price may have a start date **after** its end date. Such an
offer is rejected by both date tests and therefore never applies. No error is raised.

### 5.8 The offer's vendor must be active to be selected

**When.** During selection, not at write time.
**Effect.** An offer whose vendor is archived is silently excluded from the candidate list. The
offer is not deleted and reappears when the vendor is un-archived.

### 5.9 Learning a vendor price is capped at ten offers

**When.** A purchase order is confirmed.
**Test.** The product has **ten or fewer** offers.
**Effect when the test fails.** No offer is created. Nothing is said to the user.
**Edge case.** The test is "ten or fewer", so the creation happens when there are exactly ten,
bringing the total to eleven, and stops at eleven. A rebuild must reproduce the off-by-one to match
the shipped behaviour.

### 5.10 Learning skips vendors already recorded

**Test.** Neither the order's vendor nor that vendor's parent contact appears among the product's
existing offers' vendors.
**Effect when the test fails.** No offer is created; the existing offers are **not** updated with
the new price. Vendor price learning never overwrites.

---

## 6. The sales line discount policy

### 6.1 A discount is shown only for a percentage rule

**Test.** The discount capability is enabled **and** the selected rule's computation kind is
"percentage".
**Effect when false.** The line's discount stays zero and the whole price reduction is folded into
the unit price.

**Storefront exception.** The shop pages, the product pages and the configurator use a wider test
that also accepts a **formula** rule with a non-zero discount whose base is the sales price or
another price list. The cart and the checkout do not. So the same product can show a
struck-through price on its product page and a plain price in the cart. This is faithful
behaviour.

### 6.2 A surcharge is never shown as a negative discount

**Test.** The computed discount is kept only when its sign matches the sign of the base price
before discount: a positive discount with a positive base, or a negative discount with a negative
base.
**Effect.** A negative percentage rule on a positive base produces a higher unit price and a
discount of zero.

### 6.3 A zero base price produces a zero discount

**Test.** The base price before discount is not zero.
**Effect when it is zero.** The discount is zero. No division is attempted.

### 6.4 A combo item line copies its combo line's discount

**Effect.** The discount computation for a line carrying a combo item is short-circuited: it takes
the discount of the line it is linked to, verbatim, and stops.

### 6.5 The automatic price computation is skipped in seven cases

Listed in [`calculations.md`](calculations.md#137-when-the-automatic-computation-is-skipped). The
most important is the **manual override**: when the shadow price differs from the unit price at
the line currency's precision, the user has typed a price and the platform will not overwrite it,
until the *Update Prices* operation forces a recomputation.

**Comparison precision.** The manual-override test compares the two numbers at the **line
currency's** rounding, using the currency's precision-aware comparison. A hand-typed price that
differs from the computed one by less than half the currency's smallest unit is therefore **not**
treated as a manual override and will be silently overwritten. A rebuild that compares exactly
will treat every floating-point residue as a manual edit and will stop repricing lines.

**Currency fallback.** When the line has no currency — which happens for an unsaved line — the
comparison falls back to the line's company currency and then to the acting company's currency.

### 6.6 A line with an invoiced quantity is not repriced

**Test.** The invoiced quantity is above zero.
**Effect.** The unit price and the discount keep their values, even under a forced recomputation.

---

## 7. Access, visibility and permissions

### 7.1 Company consistency

**Message** (up to five inconsistencies are listed):

```
Uh-oh! You’ve got some company inconsistencies here:
- “<the record's display name>” belongs to company “<the company names>” while “<the field's label>” (<the field's storage name>: '<the referenced records' display names>') belongs to another company.
To avoid a mess, no company crossover is allowed!
```

For a record that is itself a company, the first line of the list instead reads:

```
- Record is company “<the company name>” while “<the field's label>” (<the field's storage name>: '<the referenced records' display names>') belongs to another company.
```

**Fields checked in this domain:**

| Entity | Company-checked fields |
|---|---|
| Price List Rule | product template, product variant, base price list |
| Vendor Price | vendor, product variant, product template |

### 7.2 Record rules

| Entity | Rule | Domain |
|---|---|---|
| Price List | "product pricelist company rule" | keep the record when its company is an **ancestor or self** of one of the acting user's allowed companies, **or** when it has no company |
| Price List Rule | "product pricelist item company rule" | the same, on the rule's computed company |
| Vendor Price | "product supplierinfo company rule" | keep the record when it has no company, **or** when its company is an ancestor or self of one of the allowed companies |

All three are non-updatable data records: reinstalling the capability does not overwrite a
customised version.

### 7.3 Access rights matrix

| Entity | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| Price List | Internal User | no | **yes** | no | no |
| Price List | Contacts Manager | no | **yes** | no | no |
| Price List | Products / Create | **yes** | **yes** | **yes** | **yes** |
| Price List | Salesperson | no | **yes** | no | no |
| Price List | Sales Manager | **yes** | **yes** | **yes** | **yes** |
| Price List Rule | Internal User | no | **yes** | no | no |
| Price List Rule | Products / Create | **yes** | **yes** | **yes** | **yes** |
| Price List Rule | Sales Manager | **yes** | **yes** | **yes** | **yes** |
| Vendor Price | Internal User | no | **yes** | no | no |
| Vendor Price | Products / Create | **yes** | **yes** | **yes** | **yes** |
| Product Margin Wizard | Accounting / User | **yes** | **yes** | **yes** | no |

**Consequences.**

- Any internal user can **read** every price list, every rule and every vendor price that the
  record rules let them see. Pricing is not confidential from internal users.
- A salesperson who is not a sales manager cannot read price list **rules** through the ordinary
  access path — only price lists. The price engine nonetheless prices their orders correctly,
  because the rule search is performed by the platform's own computation and the resulting price
  is a computed field on the line. A rebuild must make sure the engine is not blocked by the
  rule-read restriction.
- Nobody but a products manager can delete a vendor price; sales managers cannot.
- A portal or public user has **no** access entry at all and therefore cannot read price lists or
  rules directly. Storefront pricing works because the resolution and the computation are
  performed with elevated rights.

### 7.4 Field-level visibility

| Field | Restricted to |
|---|---|
| The product's cost, on the template and on the variant | Internal users |
| The sales line's cost, margin and margin percentage | Internal users |
| The price list's promotional code | Internal users |

**The cost is read with elevated rights inside the engine.** A rule whose base is the cost prices
correctly for a portal or public user even though that user cannot read the cost field. A rebuild
that omits the elevation will price such rules at zero for storefront visitors.

### 7.5 Vendor price learning runs with elevated rights

Writing the new offer onto the product template is done with elevated rights, so that a buyer who
may confirm a purchase order but may not modify products still triggers the learning.

---

## 8. Capability flags and their side effects

| Flag | Label | Effect when on | Effect when turned off |
|---|---|---|---|
| Basic price lists | "Pricelists" | Price lists appear in the interface; the automatic company default price list is created for every company; the partner price list resolution runs | **Every** price list is archived; the partner price list resolution short-circuits to "no price list"; the storefront resolution short-circuits to "no price list" |
| Discounts on lines | "Discounts" | Price list percentage rules reveal a discount on sales lines | No line ever shows a price list discount; the whole reduction is folded into the unit price |

**Coupling.** Turning on "Discounts on lines" in the settings form automatically turns on "Basic
price lists".

**How "enabled" is decided.** Both flags are tested as *feature* flags, not as *the acting user's*
group membership: the test asks whether the **superuser** has the group, which is the platform's
way of asking "is this capability switched on in this database", independently of who is acting.
A rebuild must reproduce that, or a portal visitor will see different prices from a salesperson.

---

## 9. Locking and concurrency

The domain has no locking rules of its own.

- Price lists and rules are ordinary records with ordinary optimistic concurrency: two users
  editing the same rule race, and the last write wins.
- The price engine reads a consistent snapshot within its transaction; a rule changed by another
  transaction while a quotation is being priced does not affect the running computation.
- A sales line stores its computed unit price and discount. It is therefore **not** re-priced when
  a rule changes later; existing quotations keep their prices until a user presses *Update
  Prices*. That is a deliberate business rule, not an oversight: prices quoted to a customer must
  not move under them.
- The storefront caches the resolved price list in the session. A price list that becomes
  unpublishable is detected on the next request and the resolution runs again.
- The set of price lists available to a storefront visitor is cached per website, per country
  code, per selectable flag, per set of published price lists and per contact price list. Creating,
  modifying or deleting any price list clears that cache.

---

## 10. Edge cases collected

### 10.1 Pricing with no price list at all

Legal and well defined: catalogue price, converted to the requested unit and the requested
currency. No rule is searched. See [`calculations.md`](calculations.md#11-pricing-without-a-price-list).

### 10.2 A price list with no rules

Legal. Every product prices at its catalogue price expressed in the price list's currency. This is
exactly what the automatically created "Default" price list is.

### 10.3 A rule with no price list

Storable (the link is not required at the storage level) but not reachable by the engine, because
the candidate search filters on the price list. Such rules exist only to support extension
packages that price outside a price list.

### 10.4 Two rules identical in every respect

The identifier tie-break decides; the higher identifier wins. The loser never applies.

### 10.5 A category rule and a template rule both matching

The template rule wins, because the level sorts first, regardless of minimum quantities. A
category rule with a minimum quantity of one hundred will **never** beat a template rule with a
minimum quantity of zero.

**This surprises users** and is worth stating explicitly in a rebuild's own documentation:
specificity dominates quantity.

### 10.6 A quantity break on a template rule and a bigger break on a category rule

Same answer: the template rule wins at any quantity it applies to. The category break only takes
effect for quantities below the template rule's minimum, and only if the template rule then fails.

### 10.7 A validity window in the past on the only rule

The rule is excluded by the candidate search; the engine falls back to the catalogue price. No
warning.

### 10.8 A rule whose product has been archived

Still applies. See rule 3.11.

### 10.9 A price list whose currency has been archived

The price list itself is archived by rule 2.6 and therefore disappears from selection. Documents
that already reference it keep it, and the engine still prices with it if asked.

### 10.10 A partner whose assigned price list has been archived

The specific assignment is filtered by the active flag, so the partner falls through to the
country chain as if nothing had been assigned.

### 10.11 A purchase line whose vendor has no offer for the product

The line prices from the product's cost. See
[`calculations.md`](calculations.md#157-the-purchase-order-lines-unit-price), step 4.

### 10.12 A purchase line whose vendor has an offer that is filtered out

The line prices from the product's cost **and overwrites** a manually typed price. The distinction
from 10.11 is the presence of *any* offer from that vendor, regardless of whether it applies.

### 10.13 A margin on a line with a zero subtotal

The margin percentage is zero, not undefined. The margin itself is the negative of the cost times
the quantity.

### 10.14 A margin on a line delivered but never ordered

Computed from the unit price times the delivered quantity, ignoring the discount and the taxes.
See [`calculations.md`](calculations.md#172-the-margin-and-the-margin-percentage).

### 10.15 A negative cost

Refused in an interactive form on both the template and the variant:

```
The cost of a product can't be negative.
```

This is a form handler, not a stored constraint: a negative cost can be imported, and the engine
will then price a cost-based rule at a negative number.

### 10.16 Both margin bounds configured with the minimum above the price and the maximum below it

Impossible to store (rule 3.4). If it could be stored, the maximum would win because it is applied
last.

### 10.17 A rounding step larger than the price

Legal. A price of three rounded onto a step of ten becomes zero (three divided by ten is three
tenths, which rounds half away from zero to zero). A surcharge then applies to zero. This is a
real configuration risk and a rebuild must not "protect" against it.

### 10.18 Deleting a country group

The many-to-many association rows are removed; the price lists lose that geographic restriction
and may become the country fallback for partners that previously matched it.

### 10.19 A contact with a parent company

The specific price list assignment is part of the commercial field set. Assigning a parent copies
the parent's specific assignment, **per company**, to the child, overwriting whatever the child
had. Changing the parent's specific assignment later propagates again.

### 10.20 The same physical quantity in two units must cost the same

**Invariant a rebuild should test.** For any price list, product, date and currency, pricing *n*
of unit U and pricing *m* of unit V, where *n* of U and *m* of V are the same physical quantity,
must give line totals that agree — up to the currency's rounding and up to the deliberate
non-scaling of the rule's rounding step. When they do not agree and no rounding step is
configured, the rebuild has a direction error in one of the two conversions.
