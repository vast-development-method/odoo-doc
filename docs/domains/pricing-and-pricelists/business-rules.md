# Pricing and Price Lists — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behaviour of
the domain, with the exact user-facing text where one exists.

Each rule carries a stable identifier of the form `PR-nnn`. The identifiers are unique within this
file and are cited from the other files of the folder and from
[`acceptance-criteria.md`](acceptance-criteria.md). Section 22 maps the identifiers of the two
earlier drafts of this folder onto the scheme used here.

A rule states **when** it runs, **what** it tests, **what** happens when the test fails, and, where
the platform leaves something implicit, **what a rebuild must do**. Messages are reproduced exactly
as the platform emits them, in quotation marks; where a message embeds a storage name, that name is
part of the reproduced text and is not translated into prose.

---

## 1. How the rules are enforced

The domain uses five enforcement mechanisms, and the difference between them is behaviour, not
implementation detail.

| Mechanism | Runs | Can be bypassed by | Failure |
|---|---|---|---|
| **Stored constraint** | On every creation and on every modification of the fields it watches, whether by a user, by a data load, or by another part of the platform | nothing | The whole transaction is rolled back and an error is shown |
| **Deletion guard** | Immediately before a delete | nothing; it runs even for an administrator | The delete is refused and an error is shown |
| **Archive guard** | Immediately before the active flag is cleared | nothing | The archive is refused and an error is shown |
| **Form change handler** | Only inside an interactive form, when the user changes one of the fields it watches | any programmatic write, any data load, any remote operation | Either a value is silently adjusted, or an error is shown, or a non-blocking warning appears |
| **Company consistency check** | Automatically on creation and on modification, for the fields declared company-checked | nothing | An error listing up to five inconsistencies is shown |

A rebuild that implements the form change handlers as stored constraints will reject legitimate data
loads. A rebuild that implements the stored constraints as form handlers will let corrupt data in
through the remote interface.

---

## 2. Capabilities and feature gating

**PR-001.** The Basic Price Lists capability gates the whole domain. When it is not switched on,
every contact resolves to **no** price list, quotations carry no price list, the storefront uses no
price list, and every price is the catalogue sales price expressed in the company currency. Nothing
is refused; the resolution simply short-circuits.

**PR-002.** Switching the capability on triggers the provisioning of every company's default price
list (PR-017). It runs only on the transition from off to on.

**PR-003.** Switching the capability off archives **every active** price list, with elevated rights.
Price lists that were already archived stay archived. Before saving, a non-blocking warning is shown
when at least one active price list exists: "You are deactivating the pricelist feature. Every
active pricelist will be archived." The user may proceed. The archive guard PR-019 still applies to
each price list and can abort the whole setting change.

**PR-004.** Granting the multi-currency capability also grants Basic Price Lists to internal users
and then runs the provisioning of PR-017, because prices in more than one currency need price lists
to carry them.

**PR-005.** The Discounts capability alone decides whether a percentage rule is presented to the
customer as a discount percentage or folded into the unit price. There is no per-price-list discount
setting. Turning the Discounts setting on in the settings form also turns the Pricelists setting on.

**PR-006.** How "switched on" is decided differs by path, and a rebuild must reproduce the
difference.

| Path | Test |
|---|---|
| The contact price list resolution and the storefront resolution | whether the **superuser** holds Basic Price Lists — that is, "is the capability switched on in this database" |
| The provisioning of company default price lists | whether the **acting user** holds Basic Price Lists |
| The sales line discount policy | whether the **superuser** holds Discounts |

Unifying the three tests changes what a portal visitor is charged.

---

## 3. Price List

**PR-010.** The name is required. Creation or modification without one fails with the platform's
standard required-field error for the field labelled "Pricelist Name". Names are **not** unique: two
price lists may share a name, and the display name disambiguates them by appending the currency
code.

**PR-011.** The currency is required. Its default is the acting company's currency, so the field is
normally already filled. Deleting a currency is restricted while a price list references it.

**PR-012.** The company may be left empty, which makes the price list shared by every company. Its
default is the acting company. On deletion of the company the field is set to nothing.

**PR-013.** *(Deletion guard.)* Immediately before deletion, the platform searches, **with elevated
rights**, for price list rules that satisfy all three of:

1. the rule's base is the other price list;
2. the rule's base price list is one of the price lists being deleted;
3. the rule's own price list is **not** one of the price lists being deleted.

If any such rule exists the deletion is refused with a two-part message:

```
You cannot delete pricelist(s):
(<the display names of the referenced base price lists, one per line>)
They are used within pricelist(s):
<the display names of the price lists that hold those rules, one per line>
```

The guard runs with elevated rights so that a user who cannot see another company's price lists is
still blocked by a reference from it. Because of condition 3, a set of mutually referencing price
lists can be deleted in one operation; deleting them one at a time fails.

**PR-014.** When the company field is among the written values **and exactly one** price list is
being written, the company consistency of every one of its rules is re-verified immediately after
the write. A rule naming a product template, a product variant or a base price list of a different
company makes the write fail with the message of PR-220. The check is deliberately skipped when
several price lists are written at once, which means a bulk company change can leave inconsistent
rules behind; the next single write on any of those price lists then fails. A rebuild must reproduce
both halves.

**PR-015.** Present only with the storefront capability. Whenever both a website and a company are
set, the website's company must equal the price list's company. Otherwise:

```
Only the company's websites are allowed.
Leave the Company field empty or select a website from that company.
```

**PR-016.** Archiving a currency archives every price list denominated in it. This is a side effect,
not a validation: nothing is refused — except that PR-019 may refuse one of those archives and abort
the currency change with it.

**PR-017.** *(Provisioning.)* Triggered when a company is created, when the price list capability is
switched on, and when the multi-currency capability is granted. The step does nothing at all when
the calling context carries the flag that disables company price list creation, or when the acting
user does not hold Basic Price Lists. Otherwise, for the companies named or for every company when
none is named:

1. search, **ignoring the active flag**, for price lists that have no rules and belong to those
   companies, and keep those whose currency equals their company's currency;
2. un-archive all of them;
3. for every company not covered by step 2, create a price list named "Default", in the company's
   currency, with that company and with sequence **10**.

Reproduce the un-archive-before-create order: a database whose capability has been switched off and
on again must end with the same "Default" price lists it started with, not with duplicates.

**Compatibility finding.** When a company's currency is written, the write is performed with
provisioning suppressed and the provisioning is re-run afterwards **only if the capability was off
before the write and is on after it**. A currency change does not change capabilities, so the
re-provisioning does not in fact run, and the company keeps its existing default price list in the
**old** currency. Recorded as observed. A corrected behaviour would re-run the provisioning whenever
the currency actually changed, or re-express the untouched default price list in the new currency.

**PR-018.** Duplicating a price list copies its rules, its country groups and its website, and, when
the caller supplies no name, names the copy the original name followed by a space and the word
"(copy)" in parentheses.

**PR-019.** *(Archive guard.)* Archiving is refused when at least one **active** loyalty or promotion
programme names the price list. The message is "This pricelist may not be archived. It is being used
for active promotion programs: " followed by the comma-separated programme names. Nothing is
written.

**PR-020.** An archived price list is excluded from the contact resolution, from storefront
availability, from the point of sale and from default lists. Documents that already reference it
keep it and keep their prices, and the engine still prices with it when explicitly asked.

**PR-021.** Deleting a price list deletes its rules: the link from a rule to its price list cascades.

**PR-022.** The display name is the name, or the single word "New" when the name is empty, followed
by a space and the currency's code in parentheses. Two price lists named "Retail", one in euros and
one in United States dollars, display as `Retail (EUR)` and `Retail (USD)`. The display name depends
on the currency, so changing the currency changes the price list's appearance everywhere.

**PR-023.** The standard price list ordering is sequence ascending, then identifier ascending, then
name ascending. Every rule in this file that speaks of "the first price list" means the first in
this ordering. The identifier tie-break before the name makes "the first price list" a stable notion
for the fallback chains.

**PR-024.** A name search on a price list matches either the name or the currency, so typing a
currency code into a price list selector finds every price list in that currency.

**PR-025.** The rule list shown on a price list form hides rules whose product template is archived
or whose product variant is archived. The rules remain stored and **remain effective**, because the
engine searches rules directly and does not go through that list. Archiving a product therefore
hides its rules from one screen without disabling them.

**PR-026.** A loyalty or promotion programme that names price lists must be expressed in the same
currency as all of them. Otherwise the programme is refused with "The loyalty program's currency
must be the same as all it's pricelists ones." The constraint lives in
[loyalty and promotions](../loyalty-and-promotions/) and is restated here because it is a
restriction on which price lists may be used together with a programme.

---

## 4. Price List Rule

**PR-030.** *(Stored constraint.)* A rule whose base is the other price list must name one. Failure:

```
A pricelist item with "Other Pricelist" as base must have a base_pricelist_id.
```

The message names the storage field `base_pricelist_id` (the base price list) literally; that is the
text the platform produces and it is reproduced unchanged. If the state is nevertheless reached —
for example through a data load that bypasses the constraint — the base-price computation falls
through to the catalogue price rather than failing (PR-102).

**PR-031.** *(Stored constraint.)* The graph whose nodes are price lists and whose edges are "price
list X holds a rule based on price list Y" must be acyclic. The guard is the depth-first walk of
[`calculations.md`](calculations.md#86-cycle-protection), run whenever a rule's base, base price list
or own price list is written; rules that are not based on another price list, that have no base
price list, or that have no own price list are skipped. Failure:

```
Recursive pricelist rules detected: <the names of the price lists along the cycle, joined by " ⇒ ">
```

The names joined in the message are the **price lists'** own names, not their display names and not
the rules' computed names. A price list based on itself is the smallest cycle and is refused.

This is the **only** protection against unbounded recursion: the engine performs no cycle check and
would exhaust the runtime's stack. A rebuild must implement the guard, including its visited-edge
memo, because a wide graph without the memo is exponential.

**PR-032.** *(Stored constraint, also re-run in the form when either date changes.)* For a rule that
has **both** dates, the start instant must be strictly before the end instant. Failure:

```
<the rule's display name>: end date (<the end instant, formatted for the user's language and time zone>) should be after start date (<the start instant, formatted the same way>)
```

A rule with only a start date, only an end date, or no dates at all is valid. **Equal** instants are
refused: a window of zero length cannot be stored.

**PR-033.** *(Stored constraint.)* No rule may have a minimum margin strictly greater than its
maximum margin. Failure: "The minimum margin should be lower than the maximum margin."

Edge cases a rebuild must reproduce:

- equal margins are allowed and pin the price to exactly the base plus that margin;
- a margin of zero counts as "not configured" for the computation but **still participates in this
  comparison**, so a minimum margin of five with an unset maximum margin fails the constraint;
- negative margins are allowed in both fields and mean "below the base price".

**PR-034.** *(Stored constraint.)* The rule's target must match its applicability level.

| Level | Test | Message on failure |
|---|---|---|
| `2_product_category` | a category is set | "Please specify the category for which this rule should be applied" |
| `1_product` | a product template is set | "Please specify the product for which this rule should be applied" |
| `0_product_variant` | a product variant is set | "Please specify the product variant for which this rule should be applied" |
| `3_global` | none | — |

**PR-035.** *(Form change handler only.)* A rounding step that is non-zero and strictly negative is
refused as soon as it is typed, with "The rounding method must be strictly positive." A rounding
step of zero is legal and means "do not round".

This is **not** a stored constraint: a negative rounding step can be written programmatically or
loaded from a file. The engine would then call the rounding operation with a negative step, which
that operation rejects as a programming error. A rebuild should reproduce the form check and should
also make the rounding operation reject a non-positive step, so that the failure is loud rather than
a silently wrong price.

**PR-036.** *(Normalisation, not a validation.)* On creation and on every write that includes the
level, the target fields outside the chosen level are emptied; on creation the template is filled
from the variant when only a variant is given, and the level is inferred when it is not given. The
full table is in [`entities.md`](entities.md#25-consistency-enforced-on-write). This normalisation is
a **correctness requirement**, not a convenience: the candidate search filters on the target fields,
so a global rule still carrying a stale product template would silently stop applying to other
products.

**PR-037.** *(Invariant.)* The markup is at all times the exact negation of the discount. The markup
is computed and stored from the discount and has an inverse that writes the negated markup back into
the discount, so writing either writes both. A rebuild may store one and derive the other, or store
both and keep them synchronised on every write path including data loads and remote operations; if
the two drift, cost-based rules price wrongly, because the formula reads the markup when the base is
the cost and the discount otherwise.

**PR-038.** *(Company consistency check.)* The rule's product template, product variant and base
price list must each belong to the rule's computed company or to no company. The computed company is
the price list's company when there is one, otherwise the product template's company. Failure: the
message of PR-220.

Two worked refusals: a price list with **no** company cannot hold a rule whose base price list
belongs to a company, because the rule's computed company is empty; and a price list of company one
holding a rule based on a price list of company one cannot be moved to company two.

**PR-039.** Deleting a price list, a product template, a product variant or a product category
deletes the rules that point at it: all four links cascade. Deleting a product category therefore
silently deletes every pricing rule that targeted it, with no warning. A rebuild must accept the
consequence or add its own warning at the category deletion, never at the rule.

**PR-040.** A rule has no active flag and cannot be archived. It exists or it does not.

**PR-041.** A rule with no price list is storable — the link is not required at the storage level —
but is never selected by the engine, because the candidate search filters on the price list. Such
rules exist only to support extension packages that price outside a price list; every standard form
requires a price list.

**PR-042.** The rule's company and currency are derived and stored. The company is the price list's
company when there is one, otherwise the product template's company, otherwise nothing. The currency
is the price list's currency when there is one, otherwise the computed company's currency, otherwise
the acting company's currency.

**PR-043.** The rule's currency is used to format the monetary fields on the form and in the price
label. It is **not** used by the price computation, which takes its currency from the caller or from
the price list.

**PR-044.** There is no unique constraint and no stored check constraint on a rule. Two rules
identical in every respect may coexist; the ordering decides which one wins and the other never
applies.

**PR-045.** The form change handlers of a rule — resetting the discount and the markup when the base
changes, clearing the base price list when the kind changes, zeroing the parameters of the kinds not
chosen, aligning the level with what the user filled in — run **only** in an interactive form. A
programmatic write or a data load performs none of them, which is why the normalisation of PR-036
exists separately. The complete table is in
[`entities.md`](entities.md#26-form-change-handlers).

---

## 5. Vendor Price

**PR-050.** Required fields and defaults.

| Field | Rule |
|---|---|
| `partner_id` (vendor) | Required |
| `product_tmpl_id` (product template) | Required; filled automatically from the variant when a variant is given |
| `product_uom_id` (unit) | Required; defaulted from the variant's own unit when a variant is set and from the template's own unit otherwise, then never recomputed |
| `min_qty` (minimum quantity) | Required; defaults to zero |
| `currency_id` (currency) | Required; defaults to the acting company's currency; deleting the currency is restricted |
| `delay` (lead time) | Required; defaults to one day |
| `sequence` (sequence) | Defaults to one |
| `price` (unit price) | Defaults to zero |
| `company_id` (company) | Defaults to the acting company; may be emptied |

**PR-051.** On creation **and** on modification, when a variant is written and no template is
written in the same operation, the template is filled from the variant. There is no reverse rule:
clearing the variant leaves the template, and the offer then applies to every variant.

**PR-052.** *(Form change handler only.)* Changing the template clears a variant that does not belong
to it. This does **not** run on a programmatic write, so a data load can store a variant from another
template; the selection algorithm then never matches that offer, because it tests the variant against
the product being priced.

**PR-053.** An empty variant means the offer applies to **every** variant of the template.

**PR-054.** An empty company means the offer is valid for every company.

**PR-055.** *(Company consistency check.)* The vendor, the product variant and the product template
must belong to the offer's company or to no company. Failure: the message of PR-220.

**PR-056.** Cascades are asymmetric and a rebuild must reproduce the asymmetry: deleting the **vendor**
deletes the offer; deleting the **product template** deletes the offer; deleting the **product
variant** sets the variant to nothing, which widens the offer to all variants of the template.

**PR-057.** There is no uniqueness rule. Two offers from the same vendor, for the same product, with
the same unit, the same minimum quantity, the same price and overlapping windows are all legal. The
selection picks one deterministically and the other never wins.

**PR-058.** There is no validity-window ordering constraint. Unlike a price list rule, an offer may
have a start date **after** its end date. Such an offer is rejected by both date tests and therefore
never applies. No error is raised at any point.

**PR-059.** An offer whose vendor contact is archived is silently excluded from the candidate list at
selection time, not at write time. The offer is not deleted and reappears when the vendor is
un-archived.

**PR-060.** The stored ordering is sequence ascending, then minimum quantity descending, then unit
price ascending, then identifier ascending. This is the ordering that decides **which vendor** wins
when several vendors qualify.

**PR-061.** The discounted price is computed, never stored: the unit price converted from the
offer's unit into the **product's own** unit by price conversion, times one minus the discount over
one hundred. No currency conversion happens in the field itself; the selection converts separately,
at sort time.

**PR-062.** *(Form change handler, present only with the purchasing capability.)* Choosing a vendor
sets the currency to that vendor's preferred purchase currency when it has one, and to the acting
company's currency otherwise.

**PR-063.** The display name is the vendor's display name. With the purchasing-and-inventory bridge
installed it becomes the vendor's display name followed by, in parentheses, the minimum quantity, a
space, the unit's name, a space, a hyphen, a space and the unit price formatted in the offer's
currency — unless the calling context asks for the simplified vendor name, in which case the
vendor's display name alone is used.

**PR-064.** **Compatibility finding.** The variant is declared as a computed, stored, writable field
whose computation is meant to apply a default variant supplied by the caller, but the computation
looks the default up in the registry of entity names rather than in the calling context, so it never
finds one and never assigns. Observed effect: the variant keeps whatever was written, or stays
empty. A corrected behaviour would read the default from the calling context; a rebuild may simply
make the field an ordinary writable field with no computation, which produces the same observable
result.

**PR-065.** **Compatibility finding.** A computation exists that would default the unit price to the
product's cost, but it is not attached to the unit price field, so it never runs: a newly created
offer keeps a unit price of zero until a user types one. Recorded as observed. A corrected behaviour
would either attach the computation or delete it; a rebuild that defaults the price to the cost
changes observable behaviour and must do so deliberately.

**PR-066.** An offer has no active flag and cannot be archived. Its end of life is expressed by an
end date in the past; the list view offers filters built on that idiom.

**PR-067.** *(Vendor price learning, cap.)* When a purchase order is confirmed, an offer is created
only if the product has **ten or fewer** offers. Nothing is said to the user when the cap stops the
creation. The test is "ten or fewer", so the creation happens when there are exactly ten, bringing
the total to eleven, and stops at eleven. A rebuild must reproduce the off-by-one.

**PR-068.** *(Vendor price learning, no overwrite.)* An offer is created only when neither the order's
vendor nor that vendor's parent contact already appears among the vendors of the product's offers.
Learning never updates an existing offer with a new price.

**PR-069.** Vendor price learning writes the new offer onto the product template with **elevated
rights**, so that a buyer who may confirm a purchase order but may not modify products still triggers
it.

---

## 6. Rule selection and applicability

**PR-070.** The candidate rules of a price list, for a set of products at an instant, are the rules
that belong to that price list and satisfy all of: no category, or a category that is an
ancestor-or-self of one of the products' categories; no product template, or a template among those
involved; no product variant, or a variant among those involved; no start date, or a start date at
or before the instant; no end date, or an end date at or after the instant. Neither the minimum
quantity nor the applicability level takes part in this filter.

**PR-071.** Candidates are ordered by applicability level ascending, then minimum quantity
descending, then product category identifier descending, then rule identifier descending. The
**first applicable** candidate wins. A Price List Rule has **no** sequence field.

**PR-072.** A more specific rule always wins over a less specific one, **even when it is less
favourable to the customer** and whatever the minimum quantities. A rule on a product template beats
a rule on its category; a rule on a variant beats a rule on its template. A category rule with a
minimum quantity of one hundred will never beat a template rule with a minimum quantity of zero.

**PR-073.** Within one level, the largest minimum quantity is tried first, so the largest break the
quantity satisfies is the one that applies. A rebuild that sorts the minimum quantity ascending will
always return the smallest break and will mis-charge every bulk order.

**PR-074.** Ties beyond the level and the minimum quantity are broken by the larger category
identifier first, then by the larger rule identifier first — that is, the most recently created rule
wins. The category tie-break is an **identifier** comparison, not a depth comparison; a rebuild must
compare identifiers even though that usually, but not always, prefers the deeper category.

**PR-075.** The minimum quantity is compared against the requested quantity converted into the
**product's own** unit. The comparison is a plain arithmetic comparison with no tolerance, unlike the
vendor-side comparison of PR-149. A minimum quantity of zero disables the condition entirely rather
than acting as a threshold of zero, which matters for a negative quantity.

**PR-076.** A category rule applies when the product's category equals the rule's category or is a
descendant of it, tested by asking whether the product category's materialised path starts with the
rule category's materialised path. A product with **no** category never matches a category rule, not
even one on the root category.

**PR-077.** A variant rule applies to a **product template** only when that template has exactly one
variant and that variant is the rule's. With two or more variants the rule fails and the engine falls
through to a less specific rule.

**PR-078.** A template rule applies to a variant when the variant's template is the rule's template.

**PR-079.** When no rule applies, the suitable rule is the **empty rule**: the price is the product's
catalogue sales price including its attribute extras, converted into the requested unit and the
requested currency, and the rule identifier returned to the caller is empty.

**PR-080.** Archival takes no part in the candidate search. A rule pointing at an archived product is
still found and still applied; only the price list form hides it (PR-025). **Industry-standard
default:** where a rebuild wants a warning, the right place is the archive operation on the product,
and the warning must not change the pricing outcome.

**PR-081.** Each product gets exactly **one** rule, and only that rule. Rules never combine within a
price list. Layering is achieved only through the other-price-list base.

**PR-082.** The ordering is total, because the identifier is unique, so rule selection is fully
deterministic for a given set of rules, product, quantity, unit and date. A rebuild must implement
the two tie-breaks rather than relying on the storage order of rows.

---

## 7. The price computation

**PR-090.** For any combination of stored data the engine returns a number and a rule identifier. It
never refuses, never warns and never logs a business message. The only failures possible are
programming errors.

| Condition | Kind of failure |
|---|---|
| More than one price list supplied | Assertion: a single record was expected |
| More than one currency supplied | Assertion: a single record was expected |
| More than one unit supplied | Assertion: a single record was expected |
| More than one product supplied to a per-product operation | Assertion: a single record was expected |
| A product that is neither a template nor a variant | Assertion |
| A cyclic price list graph | Unbounded recursion; prevented by PR-031 |
| A negative or zero rounding step written programmatically | The rounding operation's own assertion; prevented in forms by PR-035 |

**PR-091.** The order of operations of a formula rule is fixed and observable: percentage, then
rounding onto the step, then the surcharge, then the minimum margin floor, then the maximum margin
ceiling. **No rounding is applied after the margins**, and the minimum is applied before the maximum,
so that if both are configured and the floor exceeds the ceiling the ceiling wins.

**PR-092.** The two margin bounds are measured from the **base price**, before the percentage, the
rounding and the surcharge — not from the running value. The floor is the base price plus the
minimum margin; the ceiling is the base price plus the maximum margin. Both bounds may be negative.

**PR-093.** The fixed price, the surcharge, the minimum margin and the maximum margin are stated per
one **product** unit and are converted into the requested unit before use. The rounding step is
**not** converted: it applies to the price as already expressed in the requested unit.

**PR-094.** The percentages — the percentage of a percentage rule, the discount and the markup of a
formula rule — are dimensionless and are never converted, neither between units nor between
currencies.

**PR-095.** The base price of a rule based on the sales price is the product's catalogue sales price
plus the extra prices of the attribute values that define it, with the extras added **before** the
unit conversion, so they scale with the unit.

**PR-096.** The base price of a rule based on the cost is the product's cost, in the product's cost
currency. For a product **template** whose own cost is zero and which has variants, the cost of the
**first** variant is used. The cost is read with **elevated rights**, because the cost field is
restricted to internal users while the resulting price is not; a rebuild that omits the elevation
prices such rules at zero for portal and public users.

**PR-097.** The base price of a rule based on another price list is the full price that price list
gives for the same product, the same quantity, the same unit and the same date, requested **in that
price list's own currency**, and then converted into the target currency. Only the price comes back;
the inner rule's identifier is discarded.

**PR-098.** Every currency conversion inside the engine is performed **unrounded**, at the pricing
date, with the rate table of the **acting** company — not the product's company and not the price
list's company.

**PR-099.** The engine never rounds its result. Rounding happens when a document line stores the
price, with that document's currency. The only rounding inside the engine is the rule's rounding
step.

**PR-100.** A fixed price is returned as the number the rule stores, scaled between units but
**never converted between currencies**, even when the caller asked for another currency. A fixed
price is a policy decision in the price list's own money. A rebuild that "helpfully" converts it
produces prices wrong by the whole rate.

**PR-101.** Zero means "not configured", not "configured to zero". This applies to the rounding step,
the surcharge, both margin bounds, and both minimum quantities.

**PR-102.** A rule whose base is the other price list but which names none does not take that branch
at all: the computation falls through to the catalogue-price branch and the rule prices as if its
base were the sales price. PR-030 normally prevents the state from being stored.

**PR-103.** A quantity of zero converts to zero and therefore fails every rule with a positive
minimum quantity; rules with a minimum quantity of zero still apply. A **negative** quantity is below
every positive minimum quantity, so quantity-break rules never apply to a negative line, while rules
with a zero minimum do. A sales line substitutes **one** for a zero quantity before calling the
engine.

**PR-104.** The engine's quantity conversion is called with failure **tolerated**: when the document
unit and the product unit belong to different unit trees, the requested quantity is returned
unchanged and is compared against the minimum quantity as if the units matched. The price conversion
performs no check at all and produces a meaningless number. **Industry-standard default:** a rebuild
should prevent the situation before the engine is called by restricting a document line's unit to
the units of the product's own unit category, as the selling and buying flows already do.

**PR-105.** An unknown computation kind — one added by an extension package — falls through to the
base price. A rebuild that adds kinds must keep that fall-through, or configurations written by an
extension will price at zero instead of at the catalogue price.

**PR-106.** The pricing date is resolved **once** for a whole call and is used for rule validity, for
the currency rate, and for every recursive call into another price list.

---

## 8. Presentation on a sales order line

**PR-110.** A discount is written on a sales order line **only** when the Discounts capability is
switched on **and** the selected rule's computation kind is percentage. A fixed rule and a formula
rule never produce a discount, whatever percentage they contain internally.

**PR-111.** When a discount is to be shown, the unit price written is the **larger** of the price
before discount and the price list price; otherwise it is the price list price. Taking the larger
value is what prevents a surcharge from being displayed as a negative discount.

**PR-112.** The discount percentage is the price before discount minus the price list price, divided
by the price before discount, times one hundred — kept **only** when its sign matches the sign of the
price before discount. A positive discount on a positive base is kept; a negative discount on a
positive base is a surcharge and is discarded; on a negative base, which a refund-shaped line
produces, the mirror rule applies.

**PR-113.** When the price before discount is zero the discount is zero and no division is attempted.

**PR-114.** The price before discount is obtained by descending from the selected rule through
chained **percentage** rules until a rule that is not a percentage rule, or no rule at all, is met,
and then computing the base price of the deepest percentage rule reached. This is what makes two
chained ten-per-cent discounts display as a single nineteen-per-cent discount.

**PR-115.** The storefront catalogue, the product pages and the product configurators use a **wider**
test: a struck-through price is shown for a percentage rule, and also for a formula rule that carries
a non-zero discount whose base is the sales price or another price list. The cart and the checkout do
**not** use the wider test; they follow PR-110. The same product can therefore show a struck-through
price on its product page and a plain price in the cart, which is faithful behaviour.

**PR-116.** The unit price is adapted to the fiscal position **only** when the product has sales
taxes, a fiscal position is set, the mapping actually changes the taxes, and **every** original tax
is price-included. A price stated excluding tax is never inflated.

**PR-117.** A line whose unit price differs from its shadow price, compared at the **line currency's**
rounding, is treated as manually priced and is not repriced by a change of product, unit or quantity.
The comparison is precision-aware: a typed price differing by less than half the currency's smallest
unit is **not** treated as a manual edit and will be silently overwritten. When the line has no
currency — an unsaved line — the comparison falls back to the line's company currency and then to the
acting company's currency. A rebuild that compares exactly will treat every floating-point residue as
a manual edit and will stop repricing lines.

**PR-118.** The automatic unit price computation is skipped entirely in seven cases: the line has no
order; the line is a down payment; the line carries a global discount marker; the price is manual
(PR-117) and the caller did not force a recomputation; the invoiced quantity is above zero; the
product's expense policy is at cost and the line is an expense line; there is no unit or no product —
in which case the unit price and the shadow price are both set to zero.

**PR-119.** *Update Prices* selects the eligible lines, discards their cached rule, recomputes their
unit price with recomputation **forced**, sets their discount to zero and recomputes it, clears the
indicator, and posts a message in the conversation: "Product prices have been recomputed according to
pricelist" followed by the price list's name as a link and a full stop, or "Product prices have been
recomputed." when the order has no price list.

**PR-120.** The price list of a **confirmed** order cannot be changed. Failure: "You cannot change the
pricelist of a confirmed order !"

**PR-121.** A sales order's currency is its price list's currency when a price list is set, and its
company's currency otherwise. Choosing a price list in another currency therefore re-expresses the
whole quotation.

**PR-122.** A sales order's price list is derived from the customer's resolved price list, read in the
order's company, and only while the order is a draft; it is cleared when there is no customer, and it
remains editable. Candidates are the price lists of the order's company or of no company.

**PR-123.** Changing the price list on an order that already has lines, or changing the order's
company, makes the *Update Prices* operation visible but reprices nothing on its own.

**PR-124.** A quantity of zero on a line under construction is read as **one** both for selecting the
rule and for computing the price, so that the line already shows a meaningful single-unit price.

**PR-125.** A combo line always displays a price of zero; each combo item line takes a prorated share
of the combo product's price, rounded to the line currency with the whole residue pushed onto the
last combo; and a combo item line's discount is copied verbatim from the combo line's discount rather
than recomputed.

**PR-126.** A line with an invoiced quantity above zero keeps its unit price and its discount, even
under a forced recomputation.

**PR-127.** The lines eligible for *Update Prices* are every line that is not a display-only line,
minus the delivery lines when the delivery capability is installed.

---

## 9. Contact price list resolution

**PR-130.** A contact's effective price list is resolved in this order: the contact's specific
assignment for the acting company **when that price list is active** (and, on a storefront request,
also publishable on that storefront); otherwise the first price list matching the base filter that
has a country group containing the contact's country; otherwise the fallback of PR-131. The base
filter is "active, and belonging to the acting company or to no company".

**PR-131.** The fallback is, in order, the first that yields a record: the first price list matching
the base filter and having **no** country group; the price list named by the configuration parameter
`res.partner.property_product_pricelist_` followed by the acting company's identifier; the price list
named by the configuration parameter `res.partner.property_product_pricelist`; the first price list
matching the base filter, whatever its country groups.

**PR-132.** A contact with no country resolves to the price list of the **context country** when the
calling context carries a country code that names an existing country, and to the fallback otherwise.
The context country is added to the set of countries being resolved but never overrides a contact's
own country.

**PR-133.** Writing a price list on a contact records a specific assignment **only** when the chosen
price list differs from what the country chain would give. Choosing exactly the country default
clears any specific assignment, which leaves the contact following the policy.

**PR-134.** Changing a contact's country never moves a specific assignment. A pinned contact stays
pinned.

**PR-135.** The specific assignment is stored **per company**: the same contact may carry a different
price list in each company.

**PR-136.** The specific assignment belongs to the synchronised commercial field set, so assigning a
parent company contact copies the parent's assignment — per company — onto the child, overwriting
whatever the child had, and later changes to the parent propagate again.

**PR-137.** The contact's price list field is never empty while at least one price list matches the
base filter. When the price list capability is off it resolves to nothing (PR-001).

**PR-138.** The specific assignment is filtered by the active flag, so archiving the assigned price
list makes the contact fall through to the country chain as if nothing had been assigned.

**PR-139.** Inside a storefront request the base filter additionally requires the price list to be
publishable on the current website, and the specific-assignment filter requires the same. Everything
else in the chain is unchanged.

**PR-140.** Each configuration parameter lookup tolerates a missing, empty or non-numeric value and
yields nothing, in which case the next step of the fallback is taken. Both are read with elevated
rights.

---

## 10. Vendor price selection

**PR-145.** Preparation keeps an offer of the product's template only when its company is empty **or
exactly equal** to the buying company, its vendor contact is active, and it names no variant or names
the product being priced. The company test is an **equality** test, stricter than the record rule's
ancestor test of PR-221: a branch company does not see its parent's company-specific offers through
this path. The offers are read with elevated rights and kept in the stored ordering of PR-060.

**PR-146.** The buying company is the company of the purchase order when the caller supplies one, and
the acting company otherwise.

**PR-147.** Each prepared candidate is then rejected when any of these holds: its start date is
strictly after the pricing date; its end date is strictly before the pricing date; the caller passes
the forced-unit option and the offer's unit is neither the requested unit nor the product's own unit;
a vendor was supplied and the offer's vendor is neither that vendor nor that vendor's parent contact;
a quantity was supplied and the quantity expressed in the offer's unit is below the offer's minimum
quantity; the offer names a variant other than the product.

**PR-148.** Passing **no quantity at all** disables the minimum-quantity filter entirely. Passing a
quantity of **zero** does not: zero is compared against the minimum like any other quantity. The
replenishment lead-time computation relies on the first behaviour to find a vendor before any
quantity is known.

**PR-149.** The minimum-quantity comparison on an offer is **precision-aware** at the `Product Unit`
precision, so at two digits a quantity of two and nine hundred ninety-nine thousandths counts as
reaching a minimum of three. The corresponding comparison on a price list rule is a plain comparison
(PR-075). The two sides of the platform differ, and a rebuild must reproduce the difference.

**PR-150.** Only the offers of the **first vendor encountered** in the prepared order survive: the
walk keeps an offer when the result is still empty or the offer's vendor equals the vendor already
kept. A cheaper offer belonging to a vendor that lost this stage is never selected.

**PR-151.** Among the survivors the selected offer is the first in ascending order of: the discounted
price converted into the acting company's currency at the pricing date, unrounded; then the sequence;
then the identifier.

**PR-152.** A caller may name a different **primary** ranking key; the discounted price then becomes
the secondary key, followed by the sequence and the identifier. Every component sorts ascending.

**PR-153.** The currency conversion in the ranking key is unrounded, so two offers whose
company-currency prices differ below the currency's rounding still order deterministically. A
consequence a rebuild must accept: a change of currency rate can swap two offers from one day to the
next with nothing else changing, so the ranking must be recomputed on every call and never cached on
the product.

**PR-154.** The quantity conversion into the offer's unit **raises** on failure: an offer quoted in a
unit from another tree aborts the computation rather than tolerating it, which is the opposite of the
selling side (PR-104).

**PR-155.** When no candidate survives the selection returns nothing, and the caller falls back as
specified in PR-161 for a purchase order line or PR-176 for a replenishment.

**PR-156.** Extension packages narrow the prepared set further and a rebuild must keep the hook: the
subcontracting capability keeps only the offers of the vendors registered as subcontractors when the
caller names them, and the purchase-agreement capability keeps only the offers with no agreement or
with the agreement of the order in context.

---

## 11. Purchase order line pricing

**PR-160.** With a selected offer, the line's unit price is computed in this order: strip from the
offer's unit price the price-included purchase taxes of the product that are not among the taxes on
the line; convert from the offer's currency into the line's currency, unrounded, at the order date or
today; convert from the offer's unit into the line's unit by price conversion. The offer's discount
percentage is copied onto the line, or zero when the offer has none. Both the unit price and the
shadow price are written.

**PR-161.** With no selected offer, the discount is set to zero and the line is priced from the
product's cost, in this order: convert the cost from the product's own unit into the line's unit, or
into the product's own unit when the line has none; strip the same taxes; convert from the product's
cost currency into the line's currency, unrounded, at the order date or today. Note that the order of
the unit conversion and the tax correction is the **reverse** of PR-160. Both orders give the same
number in exact arithmetic because all three operations are multiplicative; a rebuild should still
reproduce the stated orders so that last-digit comparisons match.

**PR-162.** Before PR-161 runs, one asymmetry applies: when the order's vendor has **no offer at all**
for this product, the line already carries a unit price, and the line's unit has not changed, the
existing price is left untouched. When an offer from that vendor exists but was filtered out — wrong
date, too small a quantity, wrong unit — the typed price **is overwritten** by the cost fallback. The
distinction is deliberate: an existing-but-inapplicable offer means the price should follow the
vendor policy, whereas no offer at all means the buyer knows best. It makes the purchase line's
manual state weaker than the sales line's.

**PR-163.** The expected arrival is recomputed when an offer is selected or when the line has none
yet: the order date plus the selected offer's lead time in days, or today plus that lead time when
the order has no date, with a lead time of zero when no offer was selected. A line with no selected
offer keeps an expected arrival it already carries.

**PR-164.** The unit of a purchase order line is restricted to the product's own unit, the product's
packaging units, and the units of the offers of that product that name no variant or name this
variant. This is why a buyer may pick the vendor's unit even when the product does not otherwise list
it — and why the forced-unit filter of PR-147 can then be satisfied.

**PR-165.** Picking a product on a purchase order line pre-fills the quantity from the order vendor's
own offers for that product that are inside their validity window at the order's calendar date: the
offer with the **smallest** minimum quantity is taken, the quantity becomes that minimum replaced by
one when it is zero, and the line adopts that offer's unit. When the vendor has no such offer the
quantity is one, in the product's own unit.

**PR-166.** The whole price computation of a purchase order line is skipped when the line has no
product, when bill lines are already attached to it, when the line has no company, when the calling
context asks for unit conversion to be skipped, or when the unit price differs from the shadow price.

**PR-167.** There is no "hide the discount" policy on the buying side: a vendor discount is always
carried onto the line and always shown.

**PR-168.** The line description is rebuilt from the product seen through the selected offer, which
substitutes the vendor's own product code and product name. A description the buyer typed by hand is
preserved; only its leading product designation is refreshed, and only when the selected offer
changed. The full rule belongs to [purchasing](../purchasing/).

**PR-169.** Three derived values follow from the line: the discounted unit price is the unit price
times one minus the discount over one hundred; the unit price per product unit is the unit price
converted from the line's unit into the product's own unit, and zero for display-only lines and down
payments; the quantity in the product unit is the line quantity converted the same way, or the line
quantity itself when the two units are equal.

**PR-170.** The gross unit price used by the amount-to-invoice computation removes the discount and
then removes any non-deductible taxes, by applying the line's taxes to the discounted price for the
line quantity or one, rounding globally, taking the void total and dividing by that same quantity.

**PR-171.** On the procurement path that creates a purchase order line from scratch, the currency
conversion of the offer's price is performed **rounded**, unlike the unrounded conversion of PR-160
used when a line is edited. Recorded as observed; a rebuild that unifies the two changes the last
hundredth of some procurement-created lines.

---

## 12. Replenishment

**PR-175.** A buy rule chooses its offer in this precedence: the offer passed explicitly in the
procurement values; otherwise the offer named on the reordering rule; otherwise the automatic
selection of section 10.

**PR-176.** When that precedence yields nothing, the **first** offer of the product whose company is
empty or equals the buying company is taken anyway, so that replenishment is never blocked. Its
quantity break may not correspond to the quantity being bought, and its price is used as it stands.

**PR-177.** When even PR-176 yields nothing, the buy rule contributes **365** days of lead time, the
delay explanation reads "No Vendor Found" with "+ 365 day(s)", and the responsible users are notified
in the conversation of the record that raised the procurement with "No supplier has been found to
replenish" followed by the product and ", this product should be manually replenished."

**PR-178.** Otherwise the buy rule's lead time contribution is the selected offer's lead time in days.

**PR-179.** When a replenishment extends an existing purchase order line, the offer is re-selected on
the **sum** of the existing quantity and the new quantity, which is what makes an order cross a
quantity break and reprice.

**PR-180.** For a new line the quantity is first converted into the product's own unit with
half-up rounding, the offer is selected on that quantity, and, when the selected offer uses another
unit and no forced unit was demanded, the quantity is converted again into the offer's unit with
half-up rounding and the line adopts the offer's unit.

**PR-181.** Setting an offer on a reordering rule assigns the first route containing a buy rule for
the rule's company or for no company when the reordering rule has none, and raises the quantity to
order to the offer's minimum quantity converted into the product's own unit whenever the quantity to
order is below it.

**PR-182.** Clearing the route of a reordering rule clears its offer.

---

## 13. Storefront

**PR-185.** A price list is publishable on a storefront when its company is empty or equals the
storefront's company **and** either its website is that storefront, or it has no website and is
either selectable or carries a promotional code.

**PR-186.** A price list with no website, not selectable and with no promotional code is a
back-office price list and never reaches a storefront.

**PR-187.** A price list is available in a country when it has no country group at all, or the
visitor's country code is among the codes of the countries of its country groups. A missing country
code makes every price list available.

**PR-188.** The promotional code is readable only by internal users.

**PR-189.** The visitor's price list is remembered in the session under the key
`website_sale_current_pl` (website sale current price list). It is reused without any search while
the stored record still exists, is publishable on this website and is available in the geolocated
country; otherwise the resolution of PR-190 runs again.

**PR-190.** Resolution, in order: if the capability is off, no price list at all; else if the session
holds a usable identifier, that one; else if the visitor has a cart, the cart's recomputed price
list; else the visitor's contact's price list, replaced by the **first** available price list when
the set of available price lists is not empty and does not contain it. The result is stored in the
session.

**PR-191.** The set of price lists available to a visitor is computed as follows: nothing when the
capability is off; otherwise, when a geolocated country code is known, every price list reachable
from a country group containing that country, keeping those publishable on this website and passing
the selectable filter; when that yields nothing, or no country code is known, the website's
publishable price lists, keeping those that pass the selectable filter **and, when a country code is
known, have no country groups at all**; then, for a signed-in visitor, the contact's price list if it
is publishable, passes the selectable filter and is available in the country; finally sorted in the
standard price list ordering. The selectable filter passes everything when the caller wants every
price list, and otherwise passes a price list only when it is marked selectable or is the one
currently in the session — which is how a promotional code stays usable for the rest of the session.

**PR-192.** The website's displayed currency is the currency of the price list resolved for the
current request, falling back to the website's company currency.

**PR-193.** **Compatibility finding.** The publishability test evaluates its conditions in an order
that applies the active-flag check only to the website-specific branch: an **archived** price list
with no website that is selectable or carries a promotional code passes the test, although the
equivalent search filter used elsewhere requires the record to be active. Observed effect: an
archived generic price list can still be resolved for a visitor whose session already names it, or
be listed among the available price lists reached through a country group. A corrected behaviour
would require the active flag in both branches, exactly as the search filter does.

---

## 14. Point of sale

**PR-195.** When a point of sale uses price lists, its default price list must be among its available
price lists. Failure: "The default pricelist must be included in the available pricelists."

**PR-196.** When a point of sale uses price lists, every available price list must be expressed in the
point of sale's currency. Failure: "All available pricelists must be in the same currency as the
company or as the Sales Journal set on this point of sale if you use the Accounting application."

**PR-197.** The default price list must belong to no company or to the company of the point of sale.
Failure: "The default pricelist must belong to no company or the company of the point of sale."

**PR-198.** Every available price list must belong to no company or to the company of the point of
sale. Failure: "The selected pricelists must belong to no company or the company of the point of
sale."

**PR-199.** The price list checks are run again when a session is opened, because the company of a
price list may have changed since the configuration was saved.

**PR-200.** Only the rules of the products loaded into the terminal are loaded, and, on a first load,
only those valid at the moment of loading. A later load brings the rules changed since the previous
load and the rules whose start instant has passed since then.

**PR-201.** The price lists loaded are the configuration's available price lists, its default price
list, the price lists named by any preset, and every price list used as a base by a rule of any of
those — so that a chained base is never missing from the terminal.

---

## 15. Margins

**PR-205.** The line cost is the product's cost converted from the product's own unit into the line's
unit by price conversion, then converted from the product's cost currency into the line's currency,
unrounded, at the order date or today, reading the cost **as the line's company** because the cost is
a per-company value.

**PR-206.** The line cost is writable. A salesperson may override it, and the override survives until
the product, the company, the currency or the unit of the line changes, because those four are the
dependencies that trigger the recomputation.

**PR-207.** The line margin is the line's subtotal minus the line cost times the ordered quantity,
and the margin percentage is that margin divided by the same subtotal, or zero when the subtotal is
zero. Neither value is rounded.

**PR-208.** On a line whose delivered quantity is non-zero **and** whose ordered quantity is zero, the
margin uses the unit price times the delivered quantity instead of the subtotal, and therefore
ignores the discount and the taxes.

**PR-209.** The line margin percentage is a **fraction**, displayed multiplied by one hundred. A
rebuild that stores forty for forty per cent will display four thousand per cent.

**PR-210.** The order margin is the sum of the line margins and the order margin percentage is that
sum divided by the order's untaxed amount, or zero when that amount is zero. The percentage
aggregates as an **average** in grouped lists, not as a sum.

**PR-211.** The line cost, the line margin, the line margin percentage and the two order fields are
readable only by internal users.

**PR-212.** With the stock margin capability installed, the line cost of a line that has valued stock
moves and whose product category cost method is **not** the standard one becomes the quantity-weighted
blend of the unit value of the delivered moves and the product's cost for the quantity not yet
delivered. Lines with no valued stock moves, and lines with a zero ordered quantity and a non-zero
delivered quantity, fall through to PR-205.

**PR-213.** The manufacturing margin capability adds no field and no formula. It only declares that
the stock margin capability and the manufacturing-sales bridge must both be installed, after which a
sales line for a manufactured product has valued stock moves and PR-212 picks up the real production
cost.

**PR-214.** With the timesheet margin capability installed, two groups are separated before anything
else: service lines that are not expenses, whose service policy is ordered-and-prepaid,
delivered-manually or delivered-by-milestones, whose order is confirmed and whose cost is already
non-zero are left **untouched**; and lines whose delivered quantity comes from timesheets and whose
product has a cost of zero take the timesheet cost. Everything else falls through to the previous
computation.

**PR-215.** The timesheet cost is the negated sum of the analytic amounts divided by the sum of the
analytic unit amounts, over the analytic lines of the line that belong to a project, or the product's
cost when there are no such lines; and when the line's unit differs from the company's project time
unit the value is converted with **quantity** conversion, not price conversion. That direction is
arithmetically wrong for an amount per unit of time and it rounds, and a rebuild must reproduce it to
match. Recorded as a **compatibility finding**; a corrected behaviour would use price conversion.

**PR-216.** The fourteen product margin measures are computed together, never stored, and read only
when the calling context supplies the date range and the invoice-state filter; thirteen of them can
be summed in grouped lists by summing the per-record values, and the average purchase unit price
cannot. The two margin **rates** are expressed out of one hundred, the opposite convention from
PR-209.

---

## 16. Access, visibility and permissions

**PR-220.** *(Company consistency message.)* Up to five inconsistencies are listed:

```
Uh-oh! You’ve got some company inconsistencies here:
- “<the record's display name>” belongs to company “<the company names>” while “<the field's label>” (<the field's storage name>: '<the referenced records' display names>') belongs to another company.
To avoid a mess, no company crossover is allowed!
```

For a record that is itself a company, the first line of the list instead reads:

```
- Record is company “<the company name>” while “<the field's label>” (<the field's storage name>: '<the referenced records' display names>') belongs to another company.
```

The fields checked in this domain are the product template, the product variant and the base price
list on a Price List Rule, and the vendor, the product variant and the product template on a Vendor
Price.

**PR-221.** *(Record rules.)* Three shipped, non-updatable record rules restrict visibility.

| Entity | Rule name | Condition kept |
|---|---|---|
| Price List | "product pricelist company rule" | the record's company is an ancestor-or-self of one of the acting user's allowed companies, or the record has no company |
| Price List Rule | "product pricelist item company rule" | the same, on the rule's computed company |
| Vendor Price | "product supplierinfo company rule" | the record has no company, or its company is an ancestor-or-self of one of the allowed companies |

Because a rule's company is computed from the price list and the product template, a rule inherits
the visibility of whichever of those two carries a company.

**PR-222.** The access rights matrix is in
[`configuration.md`](configuration.md#3-access-rights-matrix). Its consequences are rules in their
own right:

- any internal user may **read** every price list, every rule and every vendor price the record
  rules let them see; pricing is not confidential from internal users;
- a salesperson who is not a sales manager cannot read price list **rules** through the ordinary
  path, yet their orders price correctly, because the rule search happens inside the platform's own
  computation and the price is a computed field on the line;
- a portal or public user has read access to price lists and rules only where the storefront
  capability is installed, and storefront pricing works in any case because the resolution and the
  computation are performed with elevated rights;
- nobody may delete a vendor price except a holder of Products / Create or a purchase manager;
  sales managers cannot.

**PR-223.** Field-level visibility: the product's cost on the template and on the variant, the sales
line's cost, margin and margin percentage, and the price list's promotional code are readable only by
internal users.

**PR-224.** The cost is read with **elevated rights** inside the engine, so that a rule based on the
cost prices correctly for a portal or public user (see PR-096).

**PR-225.** The two configuration parameters of PR-131 are read with elevated rights.

**PR-226.** The deletion guard of PR-013 and the archive guard of PR-019 both run with elevated
rights, so a reference the acting user cannot see still blocks the operation.

**PR-227.** Vendor price learning writes with elevated rights (PR-069).

**PR-228.** Reading a price through the engine requires no right on Price List Rule beyond read
access, and the engine writes nothing, so no write right is ever needed to obtain a price.

---

## 17. Rounding, precision and currency consistency

**PR-230.** Quantity conversion between units rounds **away from zero** onto the `Product Unit`
precision. When the source and destination units are the same record the engine short-circuits and
performs no conversion and therefore no rounding — so a quantity of two and four hundred thirty-three
thousandths stays exact in the product's own unit but becomes two and forty-three hundredths when it
arrives through a conversion.

**PR-231.** Price conversion between units is **never** rounded.

**PR-232.** Rounding onto a step rounds halves **away from zero**, with the error-compensation term of
the shared rounding operation.

**PR-233.** A field declared with a *minimum display precision* is stored as a full double-precision
number. The precision record only sets how many digits are displayed and serialised for display. It
does not round the stored value, and the engine never rounds to it.

**PR-234.** The amounts of a rule — the fixed price, the surcharge and the two margin bounds — are
expressed in the rule's currency, which is the price list's currency. They are **never** converted
between currencies, so changing a price list's currency changes the meaning of every one of them.
**Industry-standard default:** a rebuild should warn the user before changing the currency of a price
list that already carries amount-bearing rules, because no automatic re-expression takes place; the
warning must not change the stored values.

**PR-235.** A vendor price's amount is expressed in the offer's own currency and is converted into the
purchase order's currency at the order date when the two differ.

**PR-236.** A product's sales price is expressed in the product's currency, which is its company's
currency when it has a company and the **main** company's currency otherwise. A product's cost is
expressed in its cost currency, which is its company's currency when it has a company and the
**acting** company's currency otherwise. The two may differ for the same product, and each is
converted into the target currency separately.

**PR-237.** The rule's rounding step and the currency's own rounding are independent and neither is
aware of the other. A rule rounding to five hundredths on a currency rounding to one hundredth
produces prices on the five-hundredth grid; a rule rounding to one on a currency whose smallest coin
is five produces whole-unit prices that the currency later rounds onto its own grid at subtotal time.

**PR-238.** Raising the `Product Price` precision changes what a user can enter, not how the vendor
ranking compares: the ranking always compares the stored values after an unrounded conversion.

**PR-239.** **Industry-standard default.** When a currency has no rate at the requested date, the
nearest earlier rate is used; a currency with no rate at all is treated as having a rate of one. This
is the conversion contract of [multi-currency](../multi-currency/), restated here because every price
in this domain depends on it. The engine inherits it unchanged and never compensates for it.

**PR-240.** *(Invariant a rebuild should test.)* For any price list, product, date and currency,
pricing *n* of unit U and *m* of unit V, where *n* of U and *m* of V are the same physical quantity,
must give line totals that agree — up to the currency's rounding and up to the deliberate
non-scaling of the rounding step (PR-093). When they disagree and no rounding step is configured, the
rebuild has a direction error in one of its two conversions.

**PR-241.** A negative cost is refused in an interactive form on both the product template and the
product variant, with "The cost of a product can't be negative." This is a form handler, not a stored
constraint: a negative cost can be loaded from a file, and a cost-based rule will then price at a
negative number.

---

## 18. Dates

**PR-245.** A rule's validity bounds are **instants**, not calendar days. A rule whose start is at
nine in the morning is not a candidate at one minute to nine on the same day.

**PR-246.** Both bounds are **inclusive**: a rule whose end instant is exactly the pricing instant is
still a candidate.

**PR-247.** The instant used for a sales order line is the **order's** date, not the current instant.
A line added today to an order dated tomorrow is priced with tomorrow's rules and tomorrow's rates,
and a line added tomorrow to an order dated today is priced with today's rules and today's rates.

**PR-248.** A vendor price's validity bounds are calendar **dates**, not instants, and both are
inclusive. The date compared against them for a purchase order line is the calendar date of the order
date, taken in the acting user's time zone.

**PR-249.** When no date is supplied to the vendor selection, today's date in the acting user's time
zone is used. When no date is supplied to the price engine, the current instant is used.

**PR-250.** The procurement path that creates a purchase order line uses, as its selection date, the
**later** of the calendar date of the order date and today, so that a back-dated order does not
select an offer that has already expired.

---

## 19. Locking and concurrency

**PR-255.** The domain has no locking rules of its own. Price lists, rules and offers are ordinary
records with ordinary optimistic concurrency: two users editing the same rule race and the last write
wins.

**PR-256.** The engine reads a consistent snapshot within its transaction, so a rule changed by
another transaction while a quotation is being priced does not affect the running computation.

**PR-257.** A document line stores its computed unit price and discount and is therefore **not**
repriced when a rule changes later. Existing quotations keep their prices until a user runs *Update
Prices*. That is a deliberate business rule, not an oversight: prices quoted to a customer must not
move under them.

**PR-258.** The storefront caches the resolved price list in the session (PR-189) and the set of
price lists available per website, per country code, per selectable flag, per set of published price
lists and per contact price list. Creating, modifying or deleting **any** price list clears that
second cache.

---

## 20. Invariants a rebuild must preserve

**PR-260.** **Industry-standard default.** The price engine is free of side effects: computing a price
creates, modifies and deletes nothing, posts no message, and is safe to call concurrently and
repeatedly.

**PR-261.** **Industry-standard default.** The engine is deterministic for a fixed set of inputs and a
fixed database state. The two tie-breaks of PR-074 exist precisely to remove the last source of
non-determinism.

**PR-262.** **Industry-standard default.** A price written on a document line is a snapshot. Changing
a price list, a rule, a product price, a cost or a currency rate afterwards must never alter a line
already written; only an explicit repricing operation may.

**PR-263.** **Industry-standard default.** The domain keeps **no** price history. There is no stored
record of the price a product had at a past date, and a past price can only be reconstructed from the
documents that used it. The only audit trail is the conversation of a price list, which records the
four tracked fields: currency, company, country groups and website. A rebuild that needs auditable
price history must add it explicitly.

**PR-264.** The engine never reads a document. It takes a product, a quantity, a unit, a currency and
a date, and returns a number and a rule identifier. Every document-shaped concern — taxes, fiscal
positions, down payments, combos, loyalty rewards, delivery charges — is applied by the caller
**after** the engine returns. A rebuild that lets document concerns leak into the engine cannot serve
the storefront, the terminal and the order line from one path.

**PR-265.** Loyalty programmes and delivery charges both compute money and neither uses this engine.
They are separate engines, specified in [loyalty and promotions](../loyalty-and-promotions/) and
[delivery and shipping](../delivery-and-shipping/), and they act on the price this domain has already
produced.

**PR-266.** The same physical quantity must cost the same whichever unit the document uses; see
PR-240.

---

## 21. Edge cases collected

**PR-270.** *Pricing with no price list at all.* Legal and well defined: the catalogue price,
converted to the requested unit and the requested currency. No rule is searched.

**PR-271.** *A price list with no rules.* Legal. Every product prices at its catalogue price expressed
in the price list's currency. This is exactly what the automatically created "Default" price list is.

**PR-272.** *A rule with no price list.* Storable, never reachable by the engine (PR-041).

**PR-273.** *Two rules identical in every respect.* The identifier tie-break decides and the higher
identifier wins; the loser never applies.

**PR-274.** *A category rule and a template rule both matching.* The template rule wins, whatever the
minimum quantities (PR-072). This surprises users and a rebuild should say so in its own
documentation.

**PR-275.** *A quantity break on a template rule and a bigger break on a category rule.* Same answer:
the template rule wins at every quantity it applies to; the category break only takes effect for
quantities below the template rule's minimum, and only if the template rule then fails.

**PR-276.** *A validity window in the past on the only rule.* The rule is excluded by the candidate
search and the engine falls back to the catalogue price. No warning.

**PR-277.** *A rule whose product has been archived.* Still applies (PR-080).

**PR-278.** *A price list whose currency has been archived.* The price list itself is archived by
PR-016 and disappears from selection. Documents that already reference it keep it, and the engine
still prices with it if asked.

**PR-279.** *A contact whose assigned price list has been archived.* The specific assignment is
filtered by the active flag, so the contact falls through to the country chain as if nothing had been
assigned (PR-138).

**PR-280.** *A purchase line whose vendor has no offer for the product.* The line prices from the
product's cost (PR-161), and a typed price is preserved (PR-162).

**PR-281.** *A purchase line whose vendor has an offer that is filtered out.* The line prices from the
product's cost **and overwrites** a typed price (PR-162).

**PR-282.** *A margin on a line with a zero subtotal.* The margin percentage is zero, not undefined;
the margin itself is the negative of the cost times the quantity.

**PR-283.** *A margin on a line delivered but never ordered.* Computed from the unit price times the
delivered quantity, ignoring the discount and the taxes (PR-208).

**PR-284.** *A rounding step larger than the price.* Legal. A price of three rounded onto a step of ten
becomes zero, because three tenths rounds half away from zero to zero, and a surcharge then applies to
zero. This is a real configuration risk and a rebuild must not "protect" against it.

**PR-285.** *Both margin bounds configured with the floor above the ceiling.* Impossible to store
(PR-033). If it could be stored, the ceiling would win, because it is applied last.

**PR-286.** *Deleting a country group.* The association rows are removed; the price lists lose that
geographic restriction and may become the country fallback for contacts that previously matched it.

**PR-287.** *A contact with a parent company.* The specific assignment is copied down per company and
overwrites whatever the child had (PR-136).

**PR-288.** *A quantity of zero on a sales line.* Read as one (PR-124). Elsewhere — for instance in a
direct call to the engine — a quantity of zero fails every positive break (PR-103).

**PR-289.** *Units in different trees.* Tolerated on the selling side, fatal on the buying side
(PR-104, PR-154).

**PR-290.** *A fixed price and a non-matching currency.* Returned unconverted (PR-100).

**PR-291.** *An offer whose start date is after its end date.* Storable and never applicable (PR-058).

**PR-292.** *A product template priced against a variant rule.* Applies only when the template has
exactly one variant and it is the rule's (PR-077); pricing a template is an aggregate question that a
variant rule cannot answer when the variants differ.

---

## 22. Mapping of the former rule identifiers

The two earlier drafts of this folder numbered their rules differently. Both schemes are mapped here
so that any external reference to either can be resolved. The first draft numbered its rules by
section — "rule 2.3" and so on; the second used identifiers of the form `PR-RULE-nnn`.

| Former identifier, first draft (section number) | Former identifier, second draft | Identifier used here |
|---|---|---|
| 2.1 The name is required | PR-RULE-011 | PR-010 |
| 2.2 The currency is required | PR-RULE-012 | PR-011 |
| — | PR-RULE-013 | PR-012 |
| 2.3 Deletion guard | PR-RULE-020 | PR-013 |
| 2.4 Company change re-checks the rules | PR-RULE-017 | PR-014 |
| 2.5 A website price list must match the company | PR-RULE-153 | PR-015 |
| 2.6 Archiving a currency | PR-RULE-019 | PR-016 |
| 2.7 Disabling the capability | PR-RULE-003 | PR-003 |
| 2.8 Every company has a default price list | PR-RULE-010, PR-RULE-002 | PR-017, PR-002 |
| 2.9 Copying a price list | PR-RULE-014 | PR-018 |
| — | PR-RULE-018 | PR-019 |
| — | PR-RULE-022 | PR-020 |
| — | PR-RULE-021 | PR-021 |
| — | PR-RULE-015 | PR-022 |
| — | PR-RULE-016 | PR-023 |
| 3.1 A rule based on another price list must name one | PR-RULE-034 | PR-030 |
| 3.3 Price list bases must not form a cycle | PR-RULE-035 | PR-031 |
| 3.2 The validity window must be ordered | PR-RULE-036 | PR-032 |
| 3.4 The minimum margin must not exceed the maximum | PR-RULE-037 | PR-033 |
| 3.5 The rule's target must match its level | PR-RULE-030 | PR-034 |
| 3.6 The rounding step must be strictly positive | PR-RULE-038 | PR-035 |
| 3.7 Target normalisation | PR-RULE-031, PR-RULE-032, PR-RULE-033 | PR-036 |
| 3.8 The markup and the discount are exact negations | PR-RULE-039 | PR-037 |
| 3.9 Company consistency on a rule | PR-RULE-041 | PR-038 |
| 3.10 Deleting the price list deletes its rules | PR-RULE-042 | PR-039, PR-021 |
| 3.11 Archiving a product does not disable its rules | — | PR-025, PR-080 |
| — | PR-RULE-040 | PR-042 |
| — | PR-RULE-043 | PR-041 |
| — | PR-RULE-044 | withdrawn; see the reconciliation notes |
| 4.1 The engine raises no business error | PR-RULE-200 | PR-090, PR-260 |
| 4.2 A quantity of zero | PR-RULE-103 | PR-103, PR-124 |
| 4.3 A negative quantity | — | PR-103 |
| 4.4 A product with no category | PR-RULE-066 | PR-076 |
| 4.5 A template priced against a variant rule | PR-RULE-067 | PR-077, PR-292 |
| 4.6 A fixed price and a non-matching currency | — | PR-100, PR-290 |
| 4.7 A rule whose base price list is empty | — | PR-102 |
| 4.8 A missing currency rate | PR-RULE-186 | PR-239 |
| 4.9 Units in different trees | — | PR-104, PR-154, PR-289 |
| 5.1 Required fields | PR-RULE-050 | PR-050 |
| 5.2 The variant must belong to the template | — | PR-052 |
| 5.3 Template and variant stay synchronised | PR-RULE-051 | PR-051 |
| 5.4 Company consistency | PR-RULE-054 | PR-055 |
| 5.5 Cascades | — | PR-056 |
| 5.6 There is no uniqueness rule | PR-RULE-057 | PR-057 |
| 5.7 There is no validity-window ordering constraint | — | PR-058 |
| 5.8 The offer's vendor must be active | PR-RULE-120 | PR-059, PR-145 |
| 5.9 Learning is capped at ten offers | PR-RULE-137 | PR-067 |
| 5.10 Learning skips vendors already recorded | PR-RULE-138 | PR-068 |
| — | PR-RULE-052 | PR-053 |
| — | PR-RULE-053 | PR-054 |
| — | PR-RULE-055 | PR-060 |
| — | PR-RULE-056 | PR-066 |
| — | PR-RULE-058 | PR-061 |
| — | PR-RULE-059 | PR-062 |
| — | PR-RULE-060 | PR-070 |
| — | PR-RULE-061 | PR-071 |
| — | PR-RULE-062 | PR-072 |
| — | PR-RULE-063 | PR-073 |
| — | PR-RULE-064 | PR-074 |
| — | PR-RULE-065 | PR-075 |
| — | PR-RULE-068 | PR-078 |
| — | PR-RULE-069 | PR-079 |
| — | PR-RULE-070 | PR-091 |
| — | PR-RULE-071 | PR-092 |
| — | PR-RULE-072 | PR-093 |
| — | PR-RULE-073 | PR-094 |
| — | PR-RULE-074 | PR-103 |
| — | PR-RULE-075 | PR-095 |
| — | PR-RULE-076 | PR-096 |
| — | PR-RULE-077 | PR-097 |
| — | PR-RULE-078 | PR-098 |
| — | PR-RULE-079 | PR-099 |
| — | PR-RULE-080 | PR-106, PR-249 |
| 6.1 A discount is shown only for a percentage rule | PR-RULE-090 | PR-110 |
| 6.2 A surcharge is never shown as a negative discount | PR-RULE-091 | PR-111, PR-112 |
| 6.3 A zero base price produces a zero discount | PR-RULE-092 | PR-113 |
| 6.4 A combo item line copies its combo line's discount | — | PR-125 |
| 6.5 The automatic computation is skipped in seven cases | PR-RULE-096, PR-RULE-097 | PR-117, PR-118 |
| 6.6 A line with an invoiced quantity is not repriced | — | PR-126 |
| — | PR-RULE-093 | PR-114 |
| — | PR-RULE-094 | PR-115 |
| — | PR-RULE-095 | PR-116 |
| — | PR-RULE-098 | PR-119 |
| — | PR-RULE-099 | PR-120 |
| — | PR-RULE-100 | PR-121 |
| — | PR-RULE-101 | PR-122 |
| — | PR-RULE-102 | PR-123 |
| — | PR-RULE-110 | PR-130 |
| — | PR-RULE-111 | PR-131 |
| — | PR-RULE-112 | PR-132 |
| — | PR-RULE-113 | PR-133 |
| — | PR-RULE-114 | PR-134 |
| — | PR-RULE-115 | PR-135 |
| — | PR-RULE-116 | PR-136 |
| — | PR-RULE-117 | PR-137 |
| — | PR-RULE-121 | PR-147 |
| — | PR-RULE-122 | PR-148 |
| — | PR-RULE-123 | PR-150 |
| — | PR-RULE-124 | PR-151, PR-152 |
| — | PR-RULE-125 | PR-150 |
| — | PR-RULE-126 | PR-155 |
| — | PR-RULE-130 | PR-160 |
| — | PR-RULE-131 | PR-161 |
| — | PR-RULE-132 | PR-162 |
| — | PR-RULE-133 | PR-163 |
| — | PR-RULE-134 | PR-164 |
| — | PR-RULE-135 | PR-165 |
| — | PR-RULE-136 | PR-166 |
| — | PR-RULE-139 | PR-069, PR-227 |
| — | PR-RULE-140 | PR-175 |
| — | PR-RULE-141 | PR-176 |
| — | PR-RULE-142 | PR-177 |
| — | PR-RULE-143 | PR-179 |
| — | PR-RULE-144 | PR-180 |
| — | PR-RULE-145 | PR-181 |
| — | PR-RULE-146 | PR-182 |
| — | PR-RULE-150 | PR-185 |
| — | PR-RULE-151 | PR-186 |
| — | PR-RULE-152 | PR-187 |
| — | PR-RULE-154 | PR-188 |
| — | PR-RULE-160 | PR-195 |
| — | PR-RULE-161 | PR-196 |
| — | PR-RULE-162 | PR-197 |
| — | PR-RULE-163 | PR-198 |
| — | PR-RULE-164 | PR-199 |
| — | PR-RULE-165 | PR-200 |
| 7.1 Company consistency | PR-RULE-041 | PR-220 |
| 7.2 Record rules | PR-RULE-173 | PR-221 |
| 7.3 Access rights matrix | PR-RULE-170, PR-RULE-171, PR-RULE-172 | PR-222 |
| 7.4 Field-level visibility | PR-RULE-174 | PR-223, PR-224 |
| 7.5 Vendor price learning runs with elevated rights | PR-RULE-139 | PR-227 |
| 8. Capability flags and their side effects | PR-RULE-001 to PR-RULE-005 | PR-001 to PR-006 |
| 9. Locking and concurrency | PR-RULE-202 | PR-255 to PR-258, PR-262 |
| 10.1 Pricing with no price list | — | PR-270 |
| 10.2 A price list with no rules | — | PR-271 |
| 10.3 A rule with no price list | PR-RULE-043 | PR-272 |
| 10.4 Two rules identical in every respect | PR-RULE-201 | PR-273, PR-082, PR-261 |
| 10.5 A category rule and a template rule both matching | — | PR-274 |
| 10.6 A quantity break on a template rule | — | PR-275 |
| 10.7 A validity window in the past | — | PR-276 |
| 10.8 A rule whose product has been archived | — | PR-277 |
| 10.9 A price list whose currency has been archived | — | PR-278 |
| 10.10 A contact whose price list has been archived | — | PR-279 |
| 10.11 A purchase line whose vendor has no offer | — | PR-280 |
| 10.12 A purchase line whose offer is filtered out | — | PR-281 |
| 10.13 A margin on a line with a zero subtotal | — | PR-282 |
| 10.14 A margin on a line delivered but never ordered | — | PR-283 |
| 10.15 A negative cost | — | PR-241 |
| 10.16 Both margin bounds impossible | — | PR-285 |
| 10.17 A rounding step larger than the price | — | PR-284 |
| 10.18 Deleting a country group | — | PR-286 |
| 10.19 A contact with a parent company | — | PR-287 |
| 10.20 The same physical quantity in two units | — | PR-240, PR-266 |
| — | PR-RULE-180 | PR-230 |
| — | PR-RULE-181 | PR-231 |
| — | PR-RULE-182 | PR-232 |
| — | PR-RULE-183 | PR-234 |
| — | PR-RULE-184 | PR-235 |
| — | PR-RULE-185 | PR-236 |
| — | PR-RULE-190 | PR-245 |
| — | PR-RULE-191 | PR-246 |
| — | PR-RULE-192 | PR-247 |
| — | PR-RULE-193 | PR-248 |
| — | PR-RULE-194 | PR-249 |
| — | PR-RULE-203 | PR-263 |
| — | PR-RULE-175 | PR-228 |

Rules that appear in neither earlier draft and are new to this consolidation: PR-024, PR-026, PR-043,
PR-044, PR-045, PR-063, PR-064, PR-065, PR-081, PR-100, PR-101, PR-102, PR-104, PR-105, PR-127,
PR-138, PR-139, PR-140, PR-145, PR-146, PR-149, PR-153, PR-154, PR-156, PR-167, PR-168, PR-169,
PR-170, PR-171, PR-178, PR-189, PR-190, PR-191, PR-192, PR-193, PR-201, PR-205 to PR-216, PR-225,
PR-226, PR-233, PR-237, PR-238, PR-250, PR-264, PR-265, PR-288 to PR-292.

---

## 23. Reconciliation notes

Where the two earlier drafts disagreed, the source of the platform's behaviour was consulted and the
correct statement kept. Each resolution is recorded here.

1. **The recursion message.** The first draft said the message joins "the computed names of the
   rules along the cycle"; the second said "the display names of the pricelists on the cycle". The
   path walked is a path of **price lists**, and the message joins their `name` values, not their
   display names and not any rule name. PR-031 states that.
2. **The base-price-list message.** The first draft reproduced the message with the storage name
   `base_pricelist_id` in it; the second rewrote that token into prose. The message is contractual
   and is reproduced verbatim, storage name included. PR-030.
3. **The access rights matrix.** The first draft listed only the internal user, the contacts manager,
   the products manager, the salesperson, the sales manager and the accounting user; the second added
   the point of sale, portal, public and purchase capabilities but omitted the inventory manager and
   the manufacturing manager. The full matrix, taken from the shipped access entries, is in
   [`configuration.md`](configuration.md#3-access-rights-matrix) and includes all of them.
4. **The buying company of the vendor selection.** The first draft said the company test always uses
   the acting company; the second said the order's company when one is supplied. The second is
   correct: the purchasing capability overrides the buying company with the order's company. PR-146.
5. **The unit recorded by vendor price learning.** The first draft said the unit is copied "from the
   offer the line had selected"; the second said the **order line's** unit. The second is correct.
   Workflow 16 of [`workflows.md`](workflows.md) and PR-067 state it.
6. **Whether a quantity may be omitted from the vendor selection.** Only the second draft recorded
   that passing no quantity at all disables the minimum-quantity filter while passing zero does not.
   It is correct and is kept as PR-148.
7. **Turning the capability off.** The first draft said "every price list is archived"; the second
   said "every price list, whether it was active or not". The search performed finds only **active**
   price lists; already archived ones are simply left alone, which has the same end state. PR-003
   states it precisely.
8. **The event-ticket warning.** The second draft carried a non-blocking form warning about a
   positive minimum quantity on a rule when the event ticketing capability is installed. No such
   warning exists in the behaviour this specification describes, and it has been **withdrawn**; that
   is the entry marked "withdrawn" in the mapping table for `PR-RULE-044`. Event ticket pricing goes
   through the ordinary engine and through the storefront display rule of PR-115.
9. **The vendor price display name.** The second draft attributed the enriched display name to the
   purchasing capability. It is contributed by the purchasing-and-inventory bridge, not by purchasing
   alone. PR-063.
10. **The export column headers.** The first draft reproduced the shortened header of the unit column
    exactly; the second expanded it into prose. Export headers are contractual and are reproduced in
    code font; see [`interfaces.md`](interfaces.md#2-request-routes).
11. **The number of product margin measures.** Both drafts spoke of "fifteen" measures. There are
    **fourteen** numeric measures plus three echoes of the calling context — the range start, the
    range end and the invoice-state filter — making seventeen fields in all; and **thirteen** of the
    fourteen can be summed in a grouped list, the average purchase unit price being the exception.
    PR-216 and [`calculations.md`](calculations.md#18-margins-on-a-product--the-analysis-measures)
    state the corrected counts.
12. **Re-provisioning after a company currency change.** Both drafts stated that changing a company's
    currency re-runs the provisioning so that the created price list carries the new currency. The
    deferral exists, but the re-run is guarded by a condition that a currency change cannot satisfy,
    so it does not in fact happen. Recorded as observed and marked a compatibility finding in PR-017.
13. **Storefront publishability and the active flag.** Neither draft recorded that the publishability
    test applies the active-flag check to one branch only. Recorded as a compatibility finding in
    PR-193.
14. **The default variant and the default price of a vendor price.** Neither draft recorded that the
    variant's computation never assigns and that the price-from-cost computation is not attached to
    its field. Both are recorded as compatibility findings in PR-064 and PR-065.
15. **How "enabled" is decided.** The first draft said both capability tests ask whether the
    superuser holds the capability. That is true of the two resolution paths and of the discount
    policy, but the **provisioning** path tests the acting user. PR-006.
