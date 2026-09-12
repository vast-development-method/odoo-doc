# Pricing and Price Lists — Acceptance Criteria

Numbered Given / When / Then scenarios with concrete records, concrete inputs and exact results. A
rebuild that passes all of them prices, presents, selects and reports exactly as this specification
requires.

Each scenario carries a stable identifier of the form `PR-AC-nnn` and, where it verifies a numbered
rule, cites that rule of [`business-rules.md`](business-rules.md). Section 24 maps the identifiers of
the earlier draft of this folder onto the scheme used here, and section 25 records the resolutions
where the two source descriptions disagreed.

---

## 1. The standing fixture

Unless a scenario says otherwise, every scenario assumes the following starting state. Nothing else
is assumed; anything a scenario needs beyond this is stated in its Given.

| Element | Value |
|---|---|
| Company "Alpha" | currency the dollar, rounding one hundredth, the acting company |
| Company "Beta" | currency the euro, rounding one hundredth, a sibling of Alpha, not an ancestor of it |
| Currency rate on every pricing date used | one dollar is 0.80 euro; equivalently one euro is 1.25 dollars |
| `Product Price` precision (product price) | two decimal places |
| `Product Unit` precision (product unit) | two decimal places |
| `Discount` precision (discount) | two decimal places |
| Units of measure | `Units` with absolute factor one; `Dozens` with absolute factor twelve; `Box of 12 Dozens` with absolute factor one hundred forty-four; all three in the same unit tree, whose reference unit is `Units` |
| Weight units | `Tonnes` with absolute factor one, the reference of its own tree; `Kilograms` with absolute factor one thousandth |
| Capabilities | Basic Price Lists granted; Discounts granted; multi-currency granted |
| Product "Desk lamp" | own unit `Units`, catalogue price 23.45, cost 12.00, category "Furniture", one variant |
| Product "Office chair" | own unit `Units`, catalogue price 100.00, cost 60.00, category "Furniture / Seating", one variant |
| Product "Steel bracket" | own unit `Units`, catalogue price 5.00, cost 4.80, category "Hardware", one variant |
| Product "Sofa" | own unit `Units`, catalogue price 800.00, cost 500.00, category "Furniture", three variants "Sofa, Red", "Sofa, Blue", "Sofa, Green" |
| Category tree | "Furniture", with the child "Furniture / Seating"; "Hardware", with no child |

Where a scenario says "the price is requested", it means the engine is called with a price list, a
product, a quantity, a unit and a date, and returns a number and a rule identifier, as specified in
[`calculations.md`](calculations.md#41-the-four-entry-points). Where a scenario says "a line is
created", it means a document line is added and the platform's own computations run.

---

## 2. Feature gating and provisioning

**PR-AC-001** (PR-001). **Given** the Basic Price Lists capability is not granted, **when** the price
list of the contact "Nadia Petrov" is resolved, **then** the result is empty; **and when** a
quotation is created for her, **then** it carries no price list, its currency is the dollar, and a
line for one "Office chair" is priced at 100.00 with a discount of zero.

**PR-AC-002** (PR-002, PR-017). **Given** the database holds the two companies Alpha and Beta and no
price list at all, and the Basic Price Lists capability is not granted, **when** the capability is
granted, **then** exactly two price lists exist, both named "Default", both active, both with
sequence ten, one with company Alpha and currency the dollar and one with company Beta and currency
the euro.

**PR-AC-003** (PR-017). **Given** company Alpha already holds an **archived** price list named
"Default" with no rules and currency the dollar, **when** the provisioning step runs, **then** that
price list is un-archived, it is not renamed, and no second price list is created for Alpha.

**PR-AC-004** (PR-017). **Given** company Alpha holds one **active** price list with one rule, in the
dollar, **when** the provisioning step runs, **then** a new price list named "Default" is created for
Alpha with sequence ten, because a price list that carries rules is not an untouched default.

**PR-AC-005** (PR-017). **Given** company Alpha holds one active rule-less price list in the **euro**
while Alpha's currency is the dollar, **when** the provisioning step runs, **then** that price list
is left alone and a new "Default" in the dollar is created.

**PR-AC-006** (PR-017, PR-006). **Given** the acting user does not hold Basic Price Lists, **when**
the provisioning step runs, **then** nothing is created and nothing is un-archived.

**PR-AC-007** (PR-003). **Given** three active price lists exist and none is named by an active
promotion programme, **when** the Basic Price Lists capability is withdrawn, **then** all three are
archived, and the contact resolution of every contact returns nothing.

**PR-AC-008** (PR-003). **Given** two active price lists and one already archived price list exist,
**when** the capability is withdrawn, **then** the two active ones become archived and the third is
untouched — no write at all is performed on it.

**PR-AC-009** (PR-003). **Given** at least one active price list exists, **when** the user clears the
Pricelists setting in the settings screen, **then**, before saving, the non-blocking warning "You are
deactivating the pricelist feature. Every active pricelist will be archived." is shown and the user
may still save.

**PR-AC-010** (PR-004). **Given** neither multi-currency nor Basic Price Lists is granted, **when**
multi-currency is granted, **then** Basic Price Lists is granted to internal users and each of Alpha
and Beta receives its "Default" price list.

**PR-AC-011** (PR-005). **Given** the Discounts setting is off, **when** the user turns it on in the
settings screen, **then** the Pricelists setting is turned on as well.

**PR-AC-012** (PR-017, compatibility finding). **Given** company Alpha has the currency the dollar
and its "Default" price list in the dollar, **when** Alpha's currency is changed to the euro,
**then** the existing "Default" price list still carries the **dollar** and no new price list is
created, because the re-provisioning is guarded by a capability transition that a currency change
does not produce. A rebuild that corrects this must do so deliberately.

---

## 3. Price list master data

**PR-AC-020** (PR-010, PR-011). **Given** a new price list form, **when** the user saves without a
name, **then** the save is refused by the required-field check on the field labelled "Pricelist
Name"; **and when** the user supplies the name "Retail" and does not touch the currency, **then** the
saved record carries the dollar, Alpha's currency.

**PR-AC-021** (PR-010). **Given** a price list named "Retail" in the dollar already exists, **when** a
second price list named "Retail" in the euro is saved, **then** the save succeeds: names are not
unique.

**PR-AC-022** (PR-022). **Given** the two price lists of PR-AC-021, **then** their display names are
`Retail (USD)` and `Retail (EUR)`. **Given** an unsaved price list with no name in the dollar,
**then** its display name is `New (USD)`.

**PR-AC-023** (PR-018). **Given** a price list "Retail" with three rules, one country group and a
website, **when** it is duplicated without a name being supplied, **then** the copy is named "Retail
(copy)", carries three rules that are copies of the originals, and carries the same country group and
the same website.

**PR-AC-024** (PR-023). **Given** every price list is archived and then "First Pricelist" is created
with sequence sixteen, **when** a contact with no country and no specific assignment resolves its
price list, **then** the result is "First Pricelist". **Given** "Second Pricelist" is then created,
also with sequence sixteen, **when** the same contact resolves again, **then** the result is still
"First Pricelist", because the identifier tie-break prefers the older record.

**PR-AC-025** (PR-023). **Given** "Second Pricelist" is given sequence five, **when** the same contact
resolves again, **then** the result is "Second Pricelist": the sequence outranks the identifier.

**PR-AC-026** (PR-024). **Given** the two price lists of PR-AC-021, **when** the user types `EUR` into
a price list selection field, **then** `Retail (EUR)` is offered and `Retail (USD)` is not.

**PR-AC-027** (PR-013). **Given** the price list "Base" and the price list "Outer", where one rule of
"Outer" has the base "other price list" naming "Base", **when** "Base" alone is deleted, **then** the
deletion is refused with a message whose first line is "You cannot delete pricelist(s):", whose
second line names `Base (USD)`, whose third line is "They are used within pricelist(s):" and whose
fourth line names `Outer (USD)`.

**PR-AC-028** (PR-013). **Given** the same two price lists, **when** both are deleted in **one**
operation, **then** the deletion succeeds, because the only reference lies entirely inside the
deleted set.

**PR-AC-029** (PR-013, PR-226). **Given** "Base" belongs to Alpha and "Outer" belongs to Beta, and the
acting user is allowed in Alpha only, **when** the user deletes "Base", **then** the deletion is
still refused, because the guard runs with elevated rights and sees Beta's rule.

**PR-AC-030** (PR-021). **Given** a price list with four rules, **when** the price list is deleted,
**then** its four rules no longer exist.

**PR-AC-031** (PR-019). **Given** an active promotion programme "Spring Sale" names the price list
"Retail", **when** "Retail" is archived, **then** the archive is refused with "This pricelist may not
be archived. It is being used for active promotion programs: Spring Sale" and "Retail" stays active.

**PR-AC-032** (PR-016). **Given** two active price lists in the euro and no promotion programme,
**when** the euro is archived, **then** both price lists become archived.

**PR-AC-033** (PR-016, PR-019). **Given** one of those two euro price lists is named by an active
promotion programme, **when** the euro is archived, **then** the archive of that price list is
refused, the whole currency change fails with it, and the other price list stays active.

**PR-AC-034** (PR-020). **Given** a confirmed sales order priced with "Retail" whose only line reads
90.00, **when** "Retail" is archived, **then** the order still names "Retail", the line still reads
90.00, and a direct price request against "Retail" still returns a price.

**PR-AC-035** (PR-014, PR-220). **Given** a price list of company Alpha holding one rule based on
another price list of Alpha, **when** the price list's company is changed to Beta in a write that
touches this record alone, **then** the write is refused with the company inconsistency message,
whose first line is "Uh-oh! You've got some company inconsistencies here:".

**PR-AC-036** (PR-014). **Given** two price lists of company Alpha, each holding one rule based on
another price list of Alpha, **when** both companies are changed to Beta in **one** write of two
records, **then** the write succeeds and leaves two inconsistent rules behind; **and when** either
price list is afterwards written on its own, **then** that write fails with the same message.

**PR-AC-037** (PR-015). **Given** a price list of company Alpha and a website of company Beta,
**when** that website is set on the price list, **then** the save is refused with "Only the company's
websites are allowed." followed by a line break and "Leave the Company field empty or select a
website from that company."

---

## 4. Price list rule validation and normalisation

**PR-AC-040** (PR-034). **Given** a rule form with the level "Product Category" and no category
chosen, **when** the user saves, **then** the save is refused with "Please specify the category for
which this rule should be applied".

**PR-AC-041** (PR-034). **Given** a rule created with the level `1_product` and no product template,
**then** the creation is refused with "Please specify the product for which this rule should be
applied". **Given** the level `0_product_variant` and no variant, **then** it is refused with "Please
specify the product variant for which this rule should be applied".

**PR-AC-042** (PR-036). **Given** a rule is created with the variant "Sofa, Red", no level and no
template, **then** the created rule has the level `0_product_variant`, the variant "Sofa, Red" and
the template "Sofa".

**PR-AC-043** (PR-036). **Given** a rule is created with only a category, **then** its level is
`2_product_category`. **Given** a rule created with only a template, **then** its level is
`1_product`. **Given** a rule created with none of the three, **then** its level is `3_global`.

**PR-AC-044** (PR-036). **Given** a rule with the level `0_product_variant`, a template and a variant,
**when** its level is written as `3_global`, **then** its variant, its template and its category are
all emptied in the same write.

**PR-AC-045** (PR-036). **Given** a rule with the level `2_product_category` and a category, **when**
its level is written as `1_product` and a template is supplied in the same write, **then** the
category is emptied and the template is kept.

**PR-AC-046** (PR-036). **Given** a rule with the level `1_product`, a template and a variant, **when**
the level is rewritten as `1_product`, **then** the variant is emptied and the template is kept.

**PR-AC-047** (PR-030). **Given** a rule with the computation kind `formula` and the base `pricelist`
and no base price list, **when** it is saved, **then** the save is refused with the message
reproduced exactly as `A pricelist item with "Other Pricelist" as base must have a
base_pricelist_id.`, storage name included.

**PR-AC-048** (PR-031). **Given** four price lists "A", "B", "C" and "D" and the chain "A" based on
"B", "B" based on "C", "C" based on "D", **when** a rule of "D" based on "D" is created, **then** the
creation is refused with "Recursive pricelist rules detected: " followed by the names of the price
lists along the cycle joined by the arrow character; the same holds for a rule of "D" based on "A"
and for a rule of "C" based on "B".

**PR-AC-049** (PR-031). **Given** the same four price lists with the rule of "B" removed, leaving "A"
based on "B" and "C" based on "D", **when** a rule of "D" based on "A" is created, **then** it
succeeds; **when** a rule of "C" based on "B" is then created, **then** it also succeeds; **when** a
rule of "A" based on "C" is then created, **then** it is refused; **when** instead a rule of "B" based
on "D" is created, **then** it is refused.

**PR-AC-050** (PR-032). **Given** a rule whose start instant and end instant are both the first of
March at nine in the morning, **when** it is saved, **then** the save is refused with the rule's
display name, a colon, " end date (", the end instant formatted for the user's language and time
zone, ") should be after start date (", the start instant, and a closing bracket.

**PR-AC-051** (PR-032). **Given** a rule with only a start instant, **then** it saves. **Given** a rule
with only an end instant, **then** it saves. **Given** a rule with neither, **then** it saves.

**PR-AC-052** (PR-033). **Given** a rule with a minimum margin of 10.00 and a maximum margin of 5.00,
**when** it is saved, **then** the save is refused with "The minimum margin should be lower than the
maximum margin."

**PR-AC-053** (PR-033). **Given** a rule with a minimum margin of 5.00 and **no** maximum margin,
**when** it is saved, **then** the save is refused with the same message, because an unset maximum
margin is stored as zero and still takes part in the comparison.

**PR-AC-054** (PR-033). **Given** a rule with a minimum margin of 5.00 and a maximum margin of 5.00,
**when** it is saved, **then** it saves, and the price it produces is exactly the base price plus
five.

**PR-AC-055** (PR-033). **Given** a rule with a minimum margin of −5.00 and a maximum margin of −1.00,
**when** it is saved, **then** it saves: negative margins mean "below the base price".

**PR-AC-056** (PR-035). **Given** a rule form, **when** the user types minus five hundredths into the
rounding step, **then** the value is refused at once with "The rounding method must be strictly
positive."

**PR-AC-057** (PR-035). **Given** a data load that writes a rounding step of minus five hundredths,
**then** the record is stored, because the check is a form handler and not a stored constraint.

**PR-AC-058** (PR-037). **Given** a rule created with a discount of ten, **then** its markup reads
minus ten. **When** the discount is then written as minus twenty and two hundredths, **then** the
markup reads twenty and two hundredths. **When** the markup is then written as minus one half,
**then** the discount reads one half.

**PR-AC-059** (PR-042). **Given** a price list of company Alpha in the dollar, **when** a rule is
created in it, **then** the rule's company is Alpha and its currency is the dollar. **Given** a rule
with no price list whose product template belongs to Beta, **then** the rule's company is Beta and
its currency is the euro.

**PR-AC-060** (PR-038). **Given** a price list with **no** company, **when** a rule of it is created
with the base "other price list" naming a price list of Alpha, **then** the creation is refused with
the company inconsistency message, because the rule's computed company is empty.

**PR-AC-061** (PR-045). **Given** a rule form with the kind `percentage`, a percentage of twenty and a
base price list chosen, **when** the kind is changed to `formula` and back to `percentage` and the
percentage is retyped as twenty, **then** the base price list is empty, because changing the kind
cleared it.

**PR-AC-062** (PR-045). **Given** a rule form with the kind `formula`, the base `list_price` and a
discount of fifteen, **when** the base is changed to `standard_price`, **then** the discount and the
markup are both zero.

**PR-AC-063** (PR-045). **Given** a rule form with the kind `formula`, a discount of fifteen, a
rounding step of five hundredths, a surcharge of minus one hundredth, a minimum margin of 1.00 and a
maximum margin of 2.00, **when** the kind is changed to `fixed`, **then** the base is `list_price`
and the discount, the markup, the surcharge, the rounding step, both margins and the percentage are
all zero.

**PR-AC-064** (PR-045). **Given** a rule form opened on a price list with the template "Sofa" chosen
and no variant, **when** the user picks the variant "Sofa, Red", **then** the level becomes
`0_product_variant`; **when** the user then clears the variant, **then** the level becomes
`1_product` and the variant field is empty.

**PR-AC-065** (PR-046). **Given** the event-ticketing capability is installed and a rule form with the
level `3_global`, **when** the user sets the minimum quantity to five, **then** the non-blocking
warning titled "Warning" appears with the text "A pricelist item with a positive min. quantity will
not be applied to the event tickets products.", and the minimum quantity keeps the value five.

**PR-AC-066** (PR-046). **Given** the same capability and a rule form with the level `1_product` whose
template is an event ticket product, **when** the user sets the minimum quantity to five, **then** the
warning text is "A pricelist item with a positive min. quantity cannot be applied to this event
tickets product." **Given** the template is an ordinary product, **then** no warning appears.

**PR-AC-067** (PR-039). **Given** the product template "Sofa" with two rules scoped to it, **when**
"Sofa" is deleted, **then** both rules no longer exist. **Given** the category "Hardware" with one
rule scoped to it, **when** "Hardware" is deleted, **then** that rule no longer exists and no warning
was shown.

**PR-AC-068** (PR-044). **Given** two rules on the same price list, the same level, the same target,
the same minimum quantity, the same window and the same fixed price, **when** both are saved, **then**
both are stored: there is no uniqueness rule.

**PR-AC-069** (entities section 4.2). The variant's own rule list holds the rules of its template
that target no variant or target this variant, and writing it preserves the rules that target other
variants. **Given** the template "Sofa" with one template-scoped rule and two rules scoped to the
variant "Sofa, Red", **when** the rule list of "Sofa, Red" is read, **then** it holds all three;
**when** one of the two variant rules is removed and the other's fixed price is set to 79.90 through
the variant's own rule list, **then** the template keeps two rules in all and the surviving variant
rule reads 79.90. **Given** instead the template "Sofa" with one template-scoped rule and one rule
scoped to "Sofa, Blue", **when** the rule list of "Sofa, Red" is read, **then** it holds the
template-scoped rule only; **when** that rule is deleted from the form of "Sofa, Red", **then** it no
longer exists and the rule scoped to "Sofa, Blue" is untouched.

---

## 5. Rule selection and applicability

**PR-AC-070** (PR-071, PR-072). **Given** a price list with a rule on the category "Furniture" giving
ten per cent off and a rule on the template "Office chair" giving five per cent off, **when** the
price of "Office chair" is requested for one unit, **then** the applied rule is the template rule and
the price is 95.00, not 90.00.

**PR-AC-071** (PR-072). **Given** a price list with a rule on the template "Sofa" giving ten per cent
off and a rule on the variant "Sofa, Red" giving five per cent off, **when** the price of "Sofa, Red"
is requested, **then** the applied rule is the variant rule and the price is 760.00.

**PR-AC-072** (PR-072). **Given** a price list with a rule on the category "Furniture" with minimum
quantity one hundred giving thirty per cent off, and a rule on the template "Office chair" with
minimum quantity zero giving five per cent off, **when** the price of "Office chair" is requested for
two hundred units, **then** the applied rule is the template rule and the price is 95.00 per unit: the
category break never wins.

**PR-AC-073** (PR-073). **Given** a price list with a rule on the template "Office chair" with minimum
quantity zero and a fixed price of 100.00, and a second rule on the same template with minimum
quantity ten and a fixed price of 90.00, **when** the price is requested for four units, **then** the
result is 100.00; for ten units, 90.00; for eleven units, 90.00.

**PR-AC-074** (PR-074). **Given** two rules on the same template, the same minimum quantity and no
category, the first created with a fixed price of 99.90 and the second with a fixed price of 89.90,
**when** the price is requested, **then** the result is 89.90.

**PR-AC-075** (PR-074). **Given** two category rules matching the same product, one on "Furniture"
created first and one on "Furniture / Seating" created second, **when** the price of "Office chair"
is requested, **then** the rule with the **larger category identifier** applies — here the child
category, because it was created later. **Given** the child category had been created **before** the
parent, **then** the parent's rule would apply, because the comparison is on identifiers and not on
depth.

**PR-AC-076** (PR-075). **Given** a rule with minimum quantity four giving ten per cent off on a
product whose own unit is `Units`, **when** the price is requested for three `Units`, **then** the rule
does not apply; for four `Units`, it applies; for five `Units`, it applies; for one `Dozens`, it
applies because twelve units are reached; for four tenths of a `Dozens`, it applies because four and
eight tenths units are reached; for three tenths of a `Dozens`, it does not apply because only three
and six tenths units are reached.

**PR-AC-077** (PR-075, PR-093). **Given** a **formula** rule on the sales price with a minimum
quantity of three, no discount, no rounding step, no margins and a surcharge of minus ten, on a
product whose own unit is `Tonnes` and whose catalogue price is 100.00 per tonne, **when** the price
is requested for two `Kilograms`, **then** the matching quantity is two thousandths of a tonne, the
rule fails and the result is 0.10 per kilogram; for two thousand `Kilograms`, the matching quantity
is two tonnes, the rule fails and the result is 0.10; for three thousand five hundred `Kilograms`,
the matching quantity is three and a half tonnes, the rule applies, the surcharge scales to minus one
hundredth per kilogram and the result is **0.09**; for two `Tonnes`, 100.00; for three `Tonnes`,
**90.00**.

**PR-AC-078** (PR-076). **Given** a rule on the category "Furniture", **when** the price of "Office
chair", whose category is "Furniture / Seating", is requested, **then** the rule applies. **Given**
"Steel bracket", whose category is "Hardware", **then** it does not. **Given** a product with **no**
category, **then** it does not, even if the rule names the root category.

**PR-AC-079** (PR-077). **Given** a rule scoped to the variant of "Desk lamp", which has exactly one
variant, **when** the price of the **template** "Desk lamp" is requested, **then** the rule applies.
**Given** a rule scoped to "Sofa, Red" and the template "Sofa", which has three variants, **when** the
price of the template "Sofa" is requested, **then** the rule does not apply and the engine falls
through.

**PR-AC-080** (PR-078). **Given** a rule scoped to the template "Sofa", **when** the price of "Sofa,
Blue" is requested, **then** the rule applies.

**PR-AC-081** (PR-079). **Given** a price list with no rule at all, **when** the price of "Office
chair" is requested in the dollar, **then** the result is 100.00 and the returned rule identifier is
empty.

**PR-AC-082** (PR-070, PR-245, PR-246). **Given** a rule valid from the first of March at eight in the
morning to the first of March at eleven at night, **when** the price is requested at eight exactly,
**then** the rule is a candidate; at eleven at night exactly, it is still a candidate; one second
after eleven at night, it is not; at one minute to eight, it is not.

**PR-AC-083** (PR-036, PR-070). **Given** a rule created with the level `3_global` and a product
template supplied in the same call, **when** the price of a **different** product is requested,
**then** the global rule is still a candidate, because the normalisation emptied the template on
write.

**PR-AC-084** (PR-080, PR-025). **Given** a rule scoped to the template "Desk lamp" and "Desk lamp"
archived, **when** the price of "Desk lamp" is requested, **then** the rule still applies; **and when**
the price list form is opened, **then** the rule is not listed.

**PR-AC-085** (PR-081). **Given** a price list with a global rule giving ten per cent off and a
template rule giving a fixed price of 50.00 on "Office chair", **when** the price of "Office chair" is
requested, **then** the result is 50.00 and **not** 45.00: the two rules do not combine.

**PR-AC-086** (PR-082). **Given** any fixed database state, **when** the same price is requested one
hundred times, **then** the same rule identifier and the same number are returned every time.

---

## 6. The price computation

**PR-AC-090** (PR-091, PR-092). **Given** the price list "Retail" in the dollar with one global
formula rule on the sales price with a discount of ten, a rounding step of five hundredths, a
surcharge of one half, a minimum margin of minus one and a half and no maximum margin, and the
product "Desk lamp" at 23.45, **when** the price is requested for one `Units`, **then** the
intermediate values are 21.105 after the discount, 21.10 after the rounding, 21.60 after the
surcharge and a floor of 21.95, and the result is **21.95**.

**PR-AC-091** (PR-092). **Given** the same rule with the minimum margin changed to minus three,
**when** the price is requested, **then** the floor is 20.45, it does not bite, and the result is
**21.60**.

**PR-AC-092** (PR-093). **Given** the rule of PR-AC-090, **when** the price is requested for one
`Dozens`, **then** the base is 281.40, the discounted price is 253.26, the rounded price is 253.25,
the surcharge scales to six and gives 259.25, the floor is 281.40 minus eighteen, that is 263.40, and
the result is **263.40 per dozen**, which is 21.95 per unit.

**PR-AC-093** (PR-093). **Given** the rule of PR-AC-090 with the minimum margin removed, **when** the
price is requested for one `Units` and then for one `Dozens`, **then** the results are 21.60 and
259.25, and 259.25 is **not** twelve times 21.60, because the rounding step is not scaled between
units. This divergence is correct behaviour.

**PR-AC-094** (PR-091). **Given** a formula rule on the sales price with a discount of twenty, a
rounding step of ten and a surcharge of minus one hundredth, and a product at 100.00, **when** the
price is requested, **then** the result is **79.99**.

**PR-AC-095** (PR-091, PR-096). **Given** a formula rule on the **cost** with a markup of ninety-nine
and ninety-nine hundredths, no rounding step and a surcharge of one half, and a product whose cost is
21.00, **when** the price is requested, **then** the result is **42.4979**, and a sales line in a
currency rounded to hundredths stores 42.50.

**PR-AC-096** (PR-091). **Given** the same rule with a rounding step of one and a surcharge of minus
one hundredth, **when** the price is requested, **then** the result is **41.99**.

**PR-AC-097** (PR-092). **Given** a formula rule on the sales price with no discount, a surcharge of
one hundred, a minimum margin of ten and a maximum margin of one hundred, on a base price of ten
thousand, **when** the price is requested, **then** the floor gives ten thousand and ten, the ceiling
gives ten thousand one hundred, and the result is **10010** — the minimum is applied first and the
maximum then leaves it alone.

**PR-AC-098** (PR-091). **Given** the same rule with a surcharge of one hundred and a maximum margin
of ninety, **when** the price is requested, **then** the result is **10090**: the ceiling removes ten
of the surcharge.

**PR-AC-099** (PR-100). **Given** a fixed-price rule of 25.00 in a price list expressed in the dollar,
**when** the price is requested **in the euro**, **then** the result is **25.00**, unconverted.

**PR-AC-100** (PR-093). **Given** the same fixed-price rule on a product whose own unit is `Units`,
**when** the price is requested for one `Dozens`, **then** the result is **300.00**.

**PR-AC-101** (PR-094, PR-091). **Given** a percentage rule with a percentage of minus ten on a
product at 20.00, **when** the price is requested, **then** the result is **22.00**.

**PR-AC-102** (PR-095). **Given** a product template at 75.00 with three attribute values that create
no variant, whose extra prices are 5.00, zero and 25.00, **when** the price is requested with each
value chosen in turn and no rule applies, **then** the results are 80.00, 75.00 and 100.00.

**PR-AC-103** (PR-095). **Given** the same template and the value whose extra price is 5.00, **when**
the price is requested for one `Dozens`, **then** the result is **960.00**, because the extra price is
added before the unit conversion and therefore scales.

**PR-AC-104** (PR-096). **Given** a product template whose own cost is zero and whose first variant
has a cost of 30.00, and a formula rule on the cost with a markup of zero, **when** the price of the
**template** is requested, **then** the result is **30.00**.

**PR-AC-105** (PR-096, PR-224). **Given** a public visitor with no right to read the product cost and
a price list whose only rule is based on the cost, **when** a price is requested for that visitor,
**then** the price is computed from the real cost and is not zero.

**PR-AC-106** (PR-097). **Given** the price list "Wholesale dollars" in the dollar with one global
percentage rule of ten on the sales price, and the price list "Wholesale euros" in the euro with one
global formula rule based on "Wholesale dollars" with a discount of five, a surcharge of two and no
rounding step, and the product "Office chair" at 100.00 in the dollar, **when** the price is requested
from "Wholesale euros" with no explicit currency, **then** the inner hop returns 90.00 dollars, the
conversion gives 72.00 euros, the discount gives 68.40, the surcharge gives **70.40 euros**, and the
returned rule identifier is the **outer** rule's.

**PR-AC-107** (PR-097). **Given** price list "A" with a global fixed price of three quarters at
minimum quantity zero and a global fixed price of one half at minimum quantity one thousand, and
price list "B" with a global percentage rule of minus ten based on "A", **when** the price is
requested through "B" for one unit, **then** the result is 0.825; for one thousand units, 0.55.

**PR-AC-108** (PR-097). **Given** price list "Cost Plus" with a global formula rule on the cost with a
markup of sixty, and price list "Reseller" with a global percentage rule of twelve based on "Cost
Plus", and a product whose cost is 25.00, **when** the price is requested through "Reseller", **then**
the result is **35.20**.

**PR-AC-109** (PR-102). **Given** a rule whose base is `pricelist` and whose base price list was
emptied by a data load, **when** the price is requested, **then** the rule prices from the catalogue
price as if its base were `list_price`, and nothing is raised.

**PR-AC-110** (PR-101, PR-284). **Given** a formula rule on the sales price with no discount, a
rounding step of ten and a surcharge of one, on a product at 3.00, **when** the price is requested,
**then** three tenths rounds half away from zero to zero, the rounded price is 0.00, and the result is
**1.00**.

**PR-AC-111** (PR-103). **Given** a rule with a minimum quantity of ten, **when** the price is
requested for a quantity of zero, **then** the rule does not apply; **when** it is requested for a
quantity of minus five, **then** it does not apply either; **and given** a second rule with a minimum
quantity of zero, **then** that second rule applies in both cases.

**PR-AC-112** (PR-090). **Given** any combination of stored data, **when** a price is requested,
**then** the call returns a number and a rule identifier and raises no business error, posts no
message and writes no record.

**PR-AC-113** (PR-099). **Given** a price list in a currency rounded to hundredths and a rule that
produces 42.4979, **when** the price is requested, **then** the engine returns 42.4979 unrounded; the
rounding to 42.50 is observable only after the number is written on a document line.

**PR-AC-114** (PR-106). **Given** a rule valid only until the first of March and a price list chain
two levels deep, **when** the price is requested with the date the first of March, **then** the same
date is used for the rule validity of both levels and for the currency rate of both levels.

---

## 7. Units and currencies

**PR-AC-120** (PR-093). **Given** a product at 20.00 per `Units` and a price list with no rule,
**when** the price is requested for one `Dozens`, **then** the result is **240.00**.

**PR-AC-121** (PR-230). **Given** a quantity of three thousand three hundred thirty-three
ten-thousandths of a `Units` requested in `Units`, **then** the matching quantity is unchanged at
0.3333, because the identity conversion short-circuits. **Given** one `Units` converted into
`Dozens` at the `Product Unit` precision of two decimal places, **then** the matching quantity is
**0.09**, rounded away from zero.

**PR-AC-122** (PR-098). **Given** a product at 100.00 in the dollar, a price list in the euro, a rate
of 0.80 euro per dollar and a global percentage rule of ten, **when** a sales line for one unit is
created, **then** the line's unit price is **80.00 euro** and its discount is ten, so the subtotal is
72.00 euro.

**PR-AC-123** (PR-098). **Given** the same setting and a unit `Box of 12 Dozens` of absolute factor
one hundred forty-four, **when** the line's unit is changed to that unit for a quantity of one,
**then** the unit price is 11520.00 euro, the discount is ten, and the untaxed subtotal is 10368.00
euro.

**PR-AC-124** (PR-236). **Given** a product with **no** company, a main company in the dollar, a
second company Beta in the euro, a rate under which one dollar is 0.50 euro, a catalogue price of
100.00 read in the main company's currency and a cost of 10.00 recorded in Beta and read in the euro,
and a price list with no company carrying a twenty per cent percentage rule on the sales price for
the first product and a ten per cent percentage rule on the cost for a second product, **when** lines
are created in Beta, **then**:

| Price list currency | Product | Unit price written | Discount |
|---|---|---|---|
| euro | the first | 50.00 | 20 |
| euro | the second | 10.00 | 10 |
| dollar | the first | 100.00 | 20 |
| dollar | the second | 20.00 | 10 |

**PR-AC-125** (PR-234). **Given** a price list in the dollar with a fixed-price rule of 25.00, **when**
the price list's currency is changed to the euro, **then** the rule still reads 25.00 and now means
twenty-five euros: no amount is converted and no warning is shown by the platform.

**PR-AC-126** (PR-231). **Given** a price conversion from `Units` to `Dozens` of a price of 13.37,
**then** the result is 160.44 exactly, with no rounding at any precision.

**PR-AC-127** (PR-240, PR-266). **Given** the price list "Trade" with a global percentage rule of
fifteen at minimum quantity ten and a global percentage rule of zero at minimum quantity zero, and
"Steel bracket" at 5.00 per `Units`, **when** a line of one `Dozens` is priced, **then** the unit
price is 51.00 and the line total is 51.00; **when** a line of twelve `Units` is priced, **then** the
unit price is 4.25 and the line total is 51.00. The two totals must agree.

**PR-AC-128** (PR-075). **Given** the same price list, **when** a line of three quarters of a `Dozens`
is priced, **then** the matching quantity is nine and the break fails, so the unit price is 60.00 per
dozen and the line total is 45.00; **when** a line of eighty-four hundredths of a `Dozens` is priced,
**then** the matching quantity is ten and eight hundredths, the break succeeds, and the unit price is
51.00 per dozen.

**PR-AC-129** (PR-075, PR-230). **Given** the same price list and the unit `Box of 12 Dozens`, **when**
a line of seven hundredths of a box is priced, **then** the matching quantity is ten and eight
hundredths units, the break succeeds, the base is 720.00 per box and the price is **612.00 per box**.

**PR-AC-130** (PR-104). **Given** a product whose own unit is `Units` and a document line expressed in
`Kilograms`, which is in another unit tree, **when** the price is requested, **then** the quantity is
compared against the minimum quantity **unconverted** and the price conversion produces a meaningless
number; nothing is raised. A rebuild should prevent the situation by restricting the line's unit
choice, as the selling and buying flows already do.

**PR-AC-131** (PR-239). **Given** a currency with no rate at the requested date but a rate at an
earlier date, **when** a price is converted, **then** the nearest earlier rate is used; **given** a
currency with no rate at all, **then** the rate is taken as one.

---

## 8. Presentation on a sales order line

**PR-AC-140** (PR-110, PR-111). **Given** the Discounts capability granted, a product at 100.00 and a
percentage rule of ten on that product, **when** a sales line for one unit is created, **then** the
unit price is 100.00, the discount is ten and the subtotal is 90.00.

**PR-AC-141** (PR-110). **Given** the same setting but a **formula** rule with a discount of ten on
the sales price, **when** a sales line for one unit is created, **then** the unit price is 90.00, the
discount is zero and the subtotal is 90.00.

**PR-AC-142** (PR-110). **Given** a percentage rule of ten and the Discounts capability **withdrawn**,
**when** a sales line for one unit of a product at 100.00 is created, **then** the unit price is 90.00
and the discount is zero.

**PR-AC-143** (PR-114). **Given** the price list "First" with a percentage rule of ten on the sales
price of a product at 100.00, and the price list "Second" with a percentage rule of ten based on
"First", **when** a sales line priced with "Second" is created for one unit, **then** the unit price
is 100.00, the discount is **nineteen** and the subtotal is 81.00.

**PR-AC-144** (PR-111, PR-112). **Given** a percentage rule of minus ten and a product at 20.00,
**when** a sales line is created, **then** the unit price is 22.00 and the discount is zero, because
the candidate discount of minus ten is rejected by the sign guard.

**PR-AC-145** (PR-112). **Given** a refund-shaped line whose price before discount is minus 100.00 and
whose price list price is minus 81.00, **when** the discount is computed, **then** the candidate is
nineteen with a negative base, the mirror of the sign guard applies, and the discount shown is
nineteen.

**PR-AC-146** (PR-113). **Given** a product at zero and a percentage rule of ten, **when** a sales line
is created, **then** the price before discount is zero, no division is attempted and the discount is
zero.

**PR-AC-147** (PR-110). **Given** a price list with a percentage rule of ten on product "X" and a
formula rule with a discount of ten on product "Y", both at 100.00, **when** an order with one line of
each is repriced, **then** line "X" reads unit price 100.00, discount ten, subtotal 90.00, and line
"Y" reads unit price 90.00, discount zero, subtotal 90.00.

**PR-AC-148** (PR-124). **Given** a line under construction with a quantity of zero and a rule with a
minimum quantity of zero, **when** the rule is selected and the price computed, **then** the quantity
used is **one** and a price is shown.

**PR-AC-149** (PR-073, PR-110). **Given** a percentage rule of ten with a minimum quantity of four on a
product at 100.00 per `Units`, **when** an order carries six lines of three `Units`, four `Units`,
five `Units`, one `Dozens`, four tenths of a `Dozens` and three tenths of a `Dozens`, **then** exactly
the second, third, fourth and fifth lines carry the rule and a discount of ten, the unit prices of
those four lines are 100.00, 100.00, 1200.00 and 1200.00, and the first and sixth carry no discount.
**When** the second line's quantity is reduced to three, **then** its discount becomes zero.

**PR-AC-150** (PR-247). **Given** a percentage rule of ten valid from one hour before now until
twenty-three hours after now, and a product at 100.00, **when** a line is added today to an order
dated today, **then** the discount is ten; added today to an order dated tomorrow, zero; added
tomorrow to an order dated tomorrow, zero; added tomorrow to an order dated today, ten. With four such
lines of one unit each the untaxed total is **380.00**.

**PR-AC-151** (PR-121). **Given** a quotation whose price list is in the euro, **then** the quotation's
currency is the euro. **When** the price list is cleared, **then** the currency becomes the dollar,
the company's currency.

**PR-AC-152** (PR-122). **Given** a contact whose resolved price list is "Wholesale", **when** a
quotation is created for that contact, **then** its price list is "Wholesale".

**PR-AC-153** (PR-120). **Given** that quotation confirmed, **when** the price list is changed, **then**
the change is refused with "You cannot change the pricelist of a confirmed order !".

**PR-AC-154** (PR-118). **Given** a line with no unit of measure and no product, **when** the
computation runs, **then** both the unit price and the shadow price are set to zero.

**PR-AC-155** (PR-118). **Given** a down payment line and a global discount line, **when** the product
or the quantity of either changes, **then** neither is repriced.

**PR-AC-156** (PR-125). **Given** a combo product at 120.00 with three combos whose base prices are
60.00, 40.00 and 20.00, all in the line currency, **when** the lines are created, **then** the combo
line displays zero, the three combo item lines display 60.00, 40.00 and 20.00, and the three shares
add up to exactly 120.00.

**PR-AC-157** (PR-125). **Given** a combo product at 100.00 with three combos of equal base price,
**when** the lines are created, **then** the three shares are 33.33, 33.33 and **33.34**, the whole
rounding residue being pushed onto the last combo.

**PR-AC-158** (PR-125). **Given** a combo line carrying a discount of ten, **when** the combo item
lines are computed, **then** each carries a discount of ten, copied verbatim rather than recomputed.

**PR-AC-159** (PR-127). **Given** an order carrying one ordinary line, one display-only section line
and one delivery line, **when** *Update Prices* runs, **then** the ordinary line is repriced and the
other two are not.

---

## 9. Update prices and manual prices

**PR-AC-165** (PR-119). **Given** a quotation whose untaxed total is 1000.00 and whose price list has
no rule, **when** *Update Prices* runs, **then** the total is unchanged and the message "Product
prices have been recomputed according to pricelist" followed by the price list's display name as a
link and a full stop is posted in the order's conversation.

**PR-AC-166** (PR-119). **Given** a quotation with no price list at all, **when** *Update Prices* runs,
**then** the message posted is "Product prices have been recomputed."

**PR-AC-167** (PR-119). **Given** the quotation of PR-AC-165, **when** a global percentage rule of five
is added to its price list and *Update Prices* runs, **then** every line carries a discount of five,
the undiscounted total is 1000.00 and the total is 950.00.

**PR-AC-168** (PR-119). **Given** the same quotation, **when** that rule is replaced by a global
formula rule with a discount of five and *Update Prices* runs, **then** every line carries a discount
of zero, the undiscounted total is 1000.00 and the total is 950.00.

**PR-AC-169** (PR-119). **Given** the same quotation, **when** the price list is removed and *Update
Prices* runs, **then** every line carries a discount of zero and the total returns to 1000.00.

**PR-AC-170** (PR-117). **Given** a line whose computed unit price is 20.00, **when** the user types
100.00 and then changes the quantity to ten, **then** the unit price stays 100.00.

**PR-AC-171** (PR-117). **Given** a product at zero and a line whose unit price and shadow price are
both zero, **when** the user types 10.00 and then changes the quantity to two, **then** the unit price
stays 10.00.

**PR-AC-172** (PR-117). **Given** a line in a currency rounded to hundredths whose shadow price is
20.00, **when** the user types 20.002 and then changes the quantity, **then** the line is **not**
treated as manually priced and the price is recomputed, because the difference is below half of the
currency's smallest unit.

**PR-AC-173** (PR-119, PR-117). **Given** a line with a manual unit price of 20.50 on a product at
20.00 and no price list, **when** *Update Prices* runs, **then** the unit price becomes 20.00 and the
shadow price follows it.

**PR-AC-174** (PR-126). **Given** a line with an invoiced quantity of two, **when** the line's quantity
is changed, **then** neither the unit price nor the discount is recomputed; **and when** *Update
Prices* runs with recomputation forced, **then** they are still not recomputed.

**PR-AC-175** (PR-123). **Given** a quotation with lines and the price list "Retail", **when** the price
list is changed to "Wholesale", **then** the existing lines keep their prices and the *Update Prices*
operation becomes visible. **When** the order's company is changed instead, **then** it also becomes
visible.

**PR-AC-176** (PR-119). **Given** a quotation on which *Update Prices* has just run, **then** the
indicator that made the operation visible is cleared.

---

## 10. Tax adaptation

**PR-AC-180** (PR-116). **Given** a product at 115.00 carrying a fifteen per cent price-included sales
tax, a fiscal position mapping that tax onto a six per cent tax stated excluding tax, and a price list
with a global percentage rule of fifty-four, **when** a sales line is created, **then** the discount
is fifty-four, the unit price is 100.00, the subtotal is 46.00 and the line carries the six per cent
tax.

**PR-AC-181** (PR-116). **Given** a product whose sales tax is stated **excluding** tax and a fiscal
position mapping it onto a price-included tax, **when** a sales line is created, **then** the unit
price is the price list price unchanged: no inflation occurs.

**PR-AC-182** (PR-116). **Given** a product with **no** sales tax and a fiscal position set, **when** a
sales line is created, **then** the unit price is the price list price unchanged.

**PR-AC-183** (PR-116). **Given** a product with a price-included tax and a fiscal position whose
mapping leaves that tax unchanged, **when** a sales line is created, **then** the unit price is the
price list price unchanged, because the mapping produced the same taxes.

---

## 11. Contact price list resolution

**PR-AC-190** (PR-130, PR-131). **Given** three active price lists — "Default" with sequence ten and no
country group, "Europe Pricelist" with sequence sixteen and the country group "Europe", and
"Wholesale" with sequence sixteen and no country group — **when** the contact's country is set to
Belgium, which is in "Europe", **then** the resolved price list is "Europe Pricelist" and **no**
specific assignment is recorded. **When** the country is then set to Kiribati, **then** the resolved
price list is "Default" and still no specific assignment is recorded.

**PR-AC-191** (PR-133). **Given** the same contact, **when** "Wholesale" is chosen on the contact form,
**then** the resolved price list is "Wholesale" and the specific assignment is "Wholesale".

**PR-AC-192** (PR-133). **Given** the same contact located in Belgium with no specific assignment,
**when** "Europe Pricelist" — exactly what the country chain gives — is chosen on the form, **then** no
specific assignment is recorded.

**PR-AC-193** (PR-134). **Given** the contact of PR-AC-191 with the specific assignment "Wholesale",
**when** its country is changed back to Belgium, **then** the resolved price list stays "Wholesale"
and the specific assignment stays "Wholesale".

**PR-AC-194** (PR-131, PR-140). **Given** no price list matches the base filter and the configuration
parameter `res.partner.property_product_pricelist` (the fallback customer price list) names
"Wholesale", **when** a contact form is opened, **then** the field reads "Wholesale". **When**
"Default" is chosen instead, **then** the specific assignment becomes "Default".

**PR-AC-195** (PR-131). **Given** the company-scoped parameter
`res.partner.property_product_pricelist_` followed by Alpha's identifier names "Alpha Wholesale" and
the global parameter names "Wholesale", **when** a contact of Alpha resolves, **then** the result is
"Alpha Wholesale": the company-scoped parameter is read first.

**PR-AC-196** (PR-140). **Given** the global parameter holds the text `not-a-number`, **when** a
contact resolves, **then** the parameter is ignored and the next step of the fallback is taken.

**PR-AC-197** (PR-137). **Given** a new contact form with no country typed and at least one price list
matching the base filter, **when** it is opened, **then** the price list field already shows a price
list and is not empty.

**PR-AC-198** (PR-136, PR-135). **Given** a company contact whose specific assignment is "Business
Pricelist One" in Alpha and "Business Pricelist Two" in Beta, **when** a contact is attached to it as
a child, **then** the child's specific assignment becomes "Business Pricelist One" in Alpha and
"Business Pricelist Two" in Beta.

**PR-AC-199** (PR-136). **Given** the same parent and child, **when** the parent's specific assignment
in Alpha is changed to "Retail", **then** the child's assignment in Alpha becomes "Retail" and its
assignment in Beta is unchanged.

**PR-AC-200** (PR-138). **Given** a contact whose specific assignment is "Wholesale" and whose country
is Belgium, **when** "Wholesale" is archived, **then** the contact resolves to "Europe Pricelist" as
if nothing had been assigned, and the stored assignment is not cleared.

**PR-AC-201** (PR-132). **Given** a contact with no country and a calling context carrying the country
code of Belgium, **when** the contact resolves, **then** the result is "Europe Pricelist". **Given** a
contact whose own country is Kiribati and the same context, **then** the result is "Default": the
context never overrides a contact's own country.

---

## 12. Vendor price master data

**PR-AC-210** (PR-050). **Given** a new vendor price with only a vendor and a product template
supplied, **then** its minimum quantity is zero, its unit price is zero, its lead time is one day, its
sequence is one, its currency is the dollar, its company is Alpha, and its unit is the product
template's own unit.

**PR-AC-211** (PR-051). **Given** a vendor price created with a product variant and no product
template, **then** its product template is the variant's template. **When** the variant of an existing
vendor price is cleared, **then** the template is unchanged and the offer now applies to every
variant.

**PR-AC-212** (PR-061). **Given** a vendor price of 120.00 per `Dozens` with a discount of ten on a
product whose own unit is `Units`, **then** its discounted price is **9.00**, being 120.00 divided by
twelve and then reduced by a tenth.

**PR-AC-213** (PR-062). **Given** the purchasing capability installed and a vendor whose preferred
purchase currency is the euro, **when** that vendor is chosen on a vendor price form, **then** the
currency becomes the euro. **Given** a vendor with no preferred purchase currency, **then** the
currency becomes the dollar.

**PR-AC-214** (PR-063). **Given** the purchasing-and-inventory bridge installed and an offer for
"Wood Corner" with a minimum quantity of three `Units` and a unit price of 785.00 dollars, **then**
its display name is `Wood Corner (3.0 Units - $ 785.00)`. **Given** the bridge is not installed,
**then** the display name is `Wood Corner`.

**PR-AC-215** (PR-057). **Given** two offers from the same vendor for the same product with the same
unit, the same minimum quantity, the same price and overlapping windows, **when** both are saved,
**then** both exist and only one will ever be selected.

**PR-AC-216** (PR-058). **Given** an offer whose start date is the thirty-first of March and whose end
date is the first of March, **when** it is saved, **then** it saves without error and is rejected by
both date tests at every date, so it never applies.

**PR-AC-217** (PR-056). **Given** an offer naming the vendor "Wood Corner", the template "Sofa" and
the variant "Sofa, Red", **when** "Wood Corner" is deleted, **then** the offer is deleted. **Given** a
fresh such offer, **when** "Sofa" is deleted, **then** the offer is deleted. **Given** a fresh such
offer, **when** "Sofa, Red" is deleted, **then** the offer survives with an empty variant and now
covers every variant of "Sofa".

**PR-AC-218** (PR-052). **Given** a vendor price form naming the template "Sofa" and the variant
"Sofa, Red", **when** the template is changed to "Desk lamp", **then** the variant is cleared.
**Given** the same change performed by a data load, **then** the variant is **not** cleared and the
offer never matches anything.

**PR-AC-219** (PR-050). **Given** the `Product Unit` precision raised to three decimal places and an
offer whose minimum quantity is typed as one and two hundred thirty-four thousandths, **when** it is
saved and read back, **then** it reads 1.234.

**PR-AC-220** (PR-066). **Given** an offer with an end date in the past, **then** it is still stored,
still visible and still readable; only the selection rejects it.

**PR-AC-221** (PR-065, compatibility finding). **Given** a product whose cost is 40.00 and a newly
created offer for it on which no unit price was typed, **then** the offer's unit price is **zero**,
not 40.00.

**PR-AC-222** (PR-064, compatibility finding). **Given** a calling context that supplies a default
product variant, **when** an offer is created without naming a variant, **then** the offer's variant
is **empty**: the default is not applied.

**PR-AC-223** (entities section 3.3). **Given** a product whose internal reference is `DEFCODE`, with
an offer for "Wood Corner" carrying the vendor product code `ASUCODE` and an offer for "Azure
Interior" carrying the vendor product code `C2CCODE`, **when** the product's reference is read with no
vendor in the calling context, **then** it is `DEFCODE`; with "Azure Interior" in the context, **then**
it is `C2CCODE`. **Given** an offer whose vendor product code is empty, **then** the reference falls
back to `DEFCODE`. **Given** the reader has no read access to Vendor Price at all, **then** the
reference is always `DEFCODE`.

**PR-AC-224** (entities section 3.3, PR-221). **Given** a product with three offers for the same
vendor — one whose company is Alpha carrying the code `A`, one whose company is Beta carrying the code
`B`, and one with no company carrying the code `NO` — and a reader allowed in both companies, **when**
the product's reference is read with that vendor in the context and no company restriction, **then**
all three offers are considered; with Alpha alone allowed, **then** only `A` and `NO` are; with Beta
alone, **then** only `B` and `NO` are. An offer naming another variant is skipped, and an offer naming
**this** variant ends the search, so a variant-specific code wins over a template-wide one.

---

## 13. Vendor price selection

**PR-AC-230** (PR-060, PR-150, PR-151). **Given** three offers, all with sequence one, all in the
dollar, none with dates, none naming a variant — "V1" for "Wood Corner" with minimum quantity one at
750.00, "V2" for "Azure Interior" with minimum quantity one at 790.00, and "V3" for "Azure Interior"
with minimum quantity three at 785.00 — **then** the prepared order is V3, V1, V2, and:

| Requested vendor | Requested quantity | Selected offer | Selected price |
|---|---|---|---|
| Azure Interior | 1 | V2 | 790.00 |
| Azure Interior | 3 | V3 | 785.00 |
| none | 1 | V1 | 750.00 |
| none | 3 | V3 | 785.00 |

The last row is the important one: the cheaper 750.00 is **not** selected, because the grouping step
committed to Azure Interior before any price was compared.

**PR-AC-231** (PR-151). **Given** one vendor with three offers at twenty-five, twenty-two and twenty
thousandths, all with minimum quantity zero, and the `Product Price` precision set to three decimal
places, **when** an offer is selected for a quantity of two hundred and one, **then** the price is
0.020.

**PR-AC-232** (PR-147, PR-248). **Given** an offer valid from the first of March to the thirty-first of
March, **when** a selection is made for the twenty-eighth of February, **then** it is not a candidate;
for the first of March, it is; for the thirty-first of March, it is; for the first of April, it is
not.

**PR-AC-233** (PR-148). **Given** a single offer with a minimum quantity of one hundred, **when** a
selection is made with a quantity of five, **then** nothing is selected; **when** a selection is made
with **no quantity at all**, **then** that offer is selected.

**PR-AC-234** (PR-149). **Given** an offer with a minimum quantity of three and the `Product Unit`
precision at two decimal places, **when** a selection is made for a quantity of two and nine hundred
ninety-nine thousandths, **then** the offer **is** a candidate. **Given** the precision raised to three
decimal places, **then** it is not.

**PR-AC-235** (PR-147). **Given** an offer restricted to the variant "Sofa, Red", **when** a selection
is made for "Sofa, Blue", **then** it is not a candidate.

**PR-AC-236** (PR-145). **Given** an offer whose company is Beta, **when** a selection is made for a
purchase order of Alpha, **then** it is not a candidate. **Given** an offer with no company, **then**
it is a candidate for both. **Given** an offer whose company is the **parent** of Alpha, **then** it is
still not a candidate, because the test is an equality test.

**PR-AC-237** (PR-059). **Given** an offer whose vendor contact is archived, **when** a selection is
made, **then** it is not a candidate; **when** the vendor is un-archived, **then** it is again.

**PR-AC-238** (PR-147, PR-164). **Given** an offer stated in `Box of 12 Dozens`, a product whose own
unit is `Units` and a purchase line in `Units` with the forced-unit option, **when** a selection is
made, **then** the offer is not a candidate.

**PR-AC-239** (PR-147). **Given** the vendor contact "Wood Corner, Warehouse" whose parent is "Wood
Corner", and an offer recorded on "Wood Corner", **when** a selection is made for "Wood Corner,
Warehouse", **then** the offer is a candidate.

**PR-AC-240** (PR-151, PR-153). **Given** vendor "Alpha Supplies" offering 90.00 euro with no discount
and vendor "Beta Supplies" offering 100.00 dollars with a five per cent discount, both with sequence
one and minimum quantity zero, the company currency the dollar and a rate of 1.25 dollars per euro,
**when** a selection is made with no vendor supplied, **then** the preparation orders by the gross
price, 90.00 before 100.00, the grouping commits to "Alpha Supplies", and its offer is selected at
90.00 euro — even though ninety euros is 112.50 dollars, dearer than the ninety-five dollars the
other vendor would have charged. **Given** "Beta Supplies" is given sequence zero, **then** it wins
instead.

**PR-AC-241** (PR-154). **Given** an offer quoted in `Kilograms` for a product whose own unit is
`Units`, **when** a purchase line selection is made, **then** the quantity conversion fails and the
computation is aborted, unlike the selling side, which tolerates the same failure.

**PR-AC-242** (PR-155). **Given** a product with no offer that survives the filters, **when** a
selection is made, **then** nothing is returned and the caller falls back as PR-161 or PR-176
prescribes.

**PR-AC-243** (PR-156). **Given** the subcontracting capability and a caller naming the subcontractors
of the product, **when** a selection is made, **then** only the offers of the vendors registered as
subcontractors are considered. **Given** the purchase-agreement capability and an order carrying an
agreement, **then** only the offers with no agreement line or with that agreement are considered.

---

## 14. Purchase order line pricing

**PR-AC-250** (PR-160, PR-163). **Given** an offer of 785.00 dollars per `Units` with a discount of
three and a lead time of three days, and a purchase order in the dollar dated the tenth of a month,
**when** a line is created for five `Units`, **then** the unit price is 785.00, the discount is three,
the expected arrival is the thirteenth, and the untaxed subtotal is
5 × 785.00 × ( 1 − 3 ÷ 100 ) = 5 × 761.45 = **3807.25**.

**PR-AC-251** (PR-160). **Given** an offer of 9000.00 per `Dozens` and a purchase line expressed in
`Dozens`, **when** the line is priced, **then** the unit price is 9000.00. **Given** instead an offer
of 750.00 per `Units` and a line expressed in `Dozens`, **then** the unit price is 9000.00.

**PR-AC-252** (PR-160, PR-235). **Given** an offer of 100.00 euro and a purchase order in the dollar
with a rate of 1.25 dollars per euro on the order date, **when** a line is priced, **then** the unit
price is **125.00 dollars**.

**PR-AC-253** (PR-160). **Given** a product carrying a price-included purchase tax of ten per cent
that the order's fiscal position removes from the line, and an offer of 100.00, **when** a line is
priced, **then** the tax is stripped, giving 100.00 ÷ 1.10 = 90.909090..., which is what the line
carries and what a screen shows as **90.91** at the `Product Price` precision of two decimal
places.

**PR-AC-254** (PR-161). **Given** a product whose cost is 800.00 in the dollar, a purchase order in
the dollar and **no** offer at all for the product, **when** a line is created, **then** the unit
price is 800.00 and the discount is zero.

**PR-AC-255** (PR-162). **Given** a product with no offer for the order's vendor and a line that
already carries a typed unit price of 42.00, **when** the quantity is changed and the unit is not,
**then** the unit price stays 42.00.

**PR-AC-256** (PR-162). **Given** a product with an offer for the order's vendor whose minimum
quantity is one hundred, and a line for five units carrying a typed unit price of 42.00, **when** the
line is recomputed, **then** the typed price is **overwritten** by the cost fallback.

**PR-AC-257** (PR-163). **Given** a purchase order with **no** order date and an offer with a lead time
of four days, **when** a line is created, **then** the expected arrival is today plus four days.

**PR-AC-258** (PR-163). **Given** a line with no selected offer that already carries an expected
arrival, **when** the line is recomputed, **then** the expected arrival is left untouched.

**PR-AC-259** (PR-164). **Given** a product whose own unit is `Units`, with the packaging unit
`Dozens` and an offer stated in `Box of 12 Dozens`, **when** the unit field of a purchase line is
opened, **then** the candidate units are `Units`, `Dozens` and `Box of 12 Dozens`.

**PR-AC-260** (PR-165). **Given** a vendor with two offers for a product, one with minimum quantity
zero and one with minimum quantity ten, both in `Units`, **when** the product is chosen on a line,
**then** the quantity is pre-filled with one `Units`. **Given** instead a single offer with a minimum
quantity of five `Dozens`, **then** the quantity is pre-filled with five `Dozens` and the line adopts
`Dozens`.

**PR-AC-261** (PR-166). **Given** a purchase line whose unit price was typed by hand, **when** the
quantity is changed, **then** neither the unit price nor the discount is recomputed. **Given** a line
that already carries bill lines, **then** the same holds.

**PR-AC-262** (PR-169). **Given** a purchase line of six `Dozens` at 54.00 per dozen with a discount of
ten on a product whose own unit is `Units`, **then** the discounted unit price is 48.60, the unit price
per product unit is 4.50, and the quantity in the product unit is seventy-two.

**PR-AC-263** (PR-167). **Given** an offer carrying a discount of ten, **when** a line is priced,
**then** the discount of ten is written on the line and shown; there is no policy that folds it into
the unit price.

---

## 15. Vendor price learning

**PR-AC-270** (PR-067). **Given** a purchase order for "Wood Corner" with one line for a product that
has **no** offer for "Wood Corner", at a unit price of 750.00 in the order currency with a discount of
five, **when** the order is confirmed, **then** an offer is created with the vendor "Wood Corner", a
minimum quantity of one, a unit price of 750.00, the **line's** currency, a discount of five, a lead
time of **zero** and sequence one.

**PR-AC-271** (PR-067). **Given** the same product already carries one offer with sequence four,
**when** the same order is confirmed, **then** the created offer has sequence **five**.

**PR-AC-272** (PR-067). **Given** a product that already has **eleven** offers, **when** an order is
confirmed with a new vendor, **then** no offer is created. **Given** a product with exactly ten,
**then** an eleventh **is** created.

**PR-AC-273** (PR-068). **Given** a product that already has an offer from the order's vendor, **when**
the order is confirmed, **then** no offer is created and the existing one is not updated.

**PR-AC-274** (PR-068). **Given** a purchase order whose vendor is the address "Wood Corner,
Warehouse" whose parent is "Wood Corner", **when** the order is confirmed, **then** the created offer
names **"Wood Corner"**, the parent.

**PR-AC-275** (PR-067). **Given** a line expressed in `Dozens` at 9000.00 for a product whose template
unit is `Units`, **when** the order is confirmed, **then** the created offer has a unit price of
**750.00**, converted into the template's own unit, and no currency conversion is performed.

**PR-AC-276** (PR-067, reconciliation note six). **Given** a line that had selected an offer carrying
the vendor product code `ASU-77` and the vendor product name "Chaise haute", and expressed in
`Dozens`, **when** the order is confirmed, **then** the created offer carries the code `ASU-77`, the
name "Chaise haute" and the unit **`Dozens`**, the line's unit.

**PR-AC-277** (PR-069). **Given** a buyer who may confirm a purchase order but may not modify
products, **when** the order is confirmed, **then** the offer is created all the same.

---

## 16. Replenishment

**PR-AC-285** (PR-175). **Given** a reordering rule naming the offer "V2" and a procurement carrying no
explicit offer, **when** the buy rule runs, **then** "V2" is used even when the automatic selection
would have picked another.

**PR-AC-286** (PR-175). **Given** a replenishment wizard on which the user picked the offer "V3" and a
reordering rule naming "V2", **when** the buy rule runs, **then** "V3" is used: the explicit offer
outranks the pinned one.

**PR-AC-287** (PR-176). **Given** a product whose only offer has a minimum quantity of one hundred,
**when** a replenishment of five units runs, **then** that offer is used anyway and a purchase order
line is created at its price.

**PR-AC-288** (PR-177). **Given** a product with no offer at all and a buy rule, **when** the lead time
is computed, **then** it includes **365** days, the delay explanation reads "No Vendor Found" with
"+ 365 day(s)", and the responsible users are notified with "No supplier has been found to replenish"
followed by the product's display name and ", this product should be manually replenished."

**PR-AC-289** (PR-179). **Given** a vendor offering 750.00 above quantity one and 700.00 above quantity
one hundred, and an open purchase order carrying a line for sixty units at 750.00, **when** a
replenishment adds fifty units, **then** the line becomes one hundred and ten units at **700.00**.

**PR-AC-290** (PR-180). **Given** a selected offer stated in `Dozens` and a procurement of
twenty-four `Units` with no forced unit, **when** the purchase order line is created, **then** it is
expressed in `Dozens` with a quantity of two.

**PR-AC-291** (PR-181). **Given** a reordering rule with no route and a quantity to order of two,
**when** an offer with a minimum quantity of five `Units` is set on it, **then** the rule receives the
first route containing a buy rule and the quantity to order becomes five.

**PR-AC-292** (PR-182). **Given** a reordering rule with a route and a pinned offer, **when** the route
is cleared, **then** the pinned offer is cleared.

**PR-AC-293** (PR-250). **Given** a purchase order dated three days ago and an offer that expired
yesterday, **when** the procurement path selects an offer, **then** the selection date is **today**,
the later of the order date and today, and the expired offer is not selected.

**PR-AC-294** (calculations section 15.12). **Given** the product "Screw" with two offers from one
vendor, "A" with minimum quantity fifty at 10.00 and "B" with minimum quantity zero at 12.00, and a
product cost of 15.00, **when** the suggested quantity is sixty, **then** the estimated unit price is
10.00 and the estimated total is 600.00; **when** it is thirty, **then** they are 12.00 and 360.00;
**when** the minimum quantities are raised to fifty on "A" and forty on "B" and the suggested
quantity stays thirty, **then** the second selection ranks by minimum quantity, picks "B", and the
estimated total is 360.00; **when** the product has no offer at all, **then** the estimated unit price
is 15.00 and the estimated total is 450.00.

---

## 17. Margins

**PR-AC-300** (PR-205, PR-207). **Given** the product "Office chair" with a cost of 60.00 and a sales
line of one `Units` at a unit price of 100.00 with no discount and no tax, **then** the line cost is
60.00, the subtotal is 100.00, the margin is **40.00** and the margin percentage is **four tenths**,
displayed as forty per cent.

**PR-AC-301** (PR-207). **Given** the same product and a line of ten units at 100.00 with a discount of
five, **then** the subtotal is 950.00, the margin is **350.00** and the margin percentage is
350.00 ÷ 950.00 = 0.368421052631578 to fifteen decimal places, displayed as about thirty-six point
eight per cent.

**PR-AC-302** (PR-205). **Given** the same product with its cost of 60.00 in the dollar and an order in
the euro at a rate of 0.80 euro per dollar, and a line of one unit at 100.00 euro, **then** the line
cost is 48.00, the margin is **52.00** and the margin percentage is 0.52.

**PR-AC-303** (PR-205). **Given** the same product and a line of one `Dozens` at 1200.00, **then** the
line cost is 720.00, the margin is **480.00** and the margin percentage is four tenths — the same
fraction as PR-AC-300, as it must be.

**PR-AC-304** (PR-208). **Given** a line whose ordered quantity is zero, whose delivered quantity is
three, whose unit price is 100.00 and whose product costs 60.00, **then** the calculated subtotal is
300.00, the margin is **120.00** and the margin percentage is four tenths; the discount and the taxes
are ignored.

**PR-AC-305** (PR-282). **Given** a line whose subtotal is zero and whose cost is 60.00 for one unit,
**then** the margin is minus 60.00 and the margin percentage is **zero**, not undefined.

**PR-AC-306** (PR-209). **Given** the line of PR-AC-300, **then** the stored margin percentage is 0.4
and the screen shows forty per cent. A rebuild that stores forty shows four thousand per cent.

**PR-AC-307** (PR-210). **Given** an order with two lines whose margins are 40.00 and 350.00 and whose
untaxed total is 1050.00, **then** the order margin is **390.00** and the order margin percentage is
390.00 ÷ 1050.00 = 0.3714285714285714 to sixteen decimal places. **Given** a grouped list of two such orders, **then** the margin column shows the
**sum** and the margin percentage column shows the **average**.

**PR-AC-308** (PR-210). **Given** an order whose untaxed amount is zero, **then** the order margin
percentage is zero.

**PR-AC-309** (PR-206). **Given** a line whose computed cost is 60.00, **when** the salesperson types
75.00 into the cost, **then** the cost stays 75.00 while the quantity changes; **when** the product is
changed, **then** the cost is recomputed and the override is lost.

**PR-AC-310** (PR-211). **Given** a portal user reading a sales order, **then** the cost, the margin and
the margin percentage of every line and of the order are not returned.

**PR-AC-311** (PR-212). **Given** the stock margin capability, a product whose category cost method is
not the standard one, a line of ten units of which four are delivered with a stock move unit value of
55.00, and a product cost of 60.00, **then** the line cost is
( 4 × 55.00 + 6 × 60.00 ) ÷ 10 = **58.00** per unit.

**PR-AC-312** (PR-212). **Given** the same capability but a line with **no** valued stock move, **then**
the line cost falls back to the catalogue cost of 60.00.

**PR-AC-313** (PR-212). **Given** the same capability, a line with valued stock moves, an ordered
quantity of zero and a delivered quantity of three, **then** the line cost falls back to the catalogue
cost and the margin uses the delivered-but-never-ordered branch of PR-AC-304.

**PR-AC-314** (PR-214). **Given** the timesheet margin capability and a confirmed order line for a
service whose service policy is delivered by milestones and whose cost is already 80.00, **then** the
line cost is left at 80.00 and no other computation touches it.

**PR-AC-315** (PR-215, compatibility finding). **Given** the timesheet margin capability, a service line
whose product cost is zero, whose delivered quantity comes from timesheets, and whose analytic lines
sum to minus 450.00 over ten unit amounts, **then** the average cost is 45.00 per hour; **and given**
the line's unit is `Days` while the company's project time unit is `Hours` with eight hours to the day,
**then** the value is converted with **quantity** conversion, giving 45.00 × 8 = 360.00 rounded onto
the `Product Unit` step, which is the arithmetically wrong direction and must be reproduced.

**PR-AC-316** (PR-213). **Given** the manufacturing margin capability alone, **then** no field and no
formula is added; **and given** the stock margin capability and the manufacturing-sales bridge are
both installed, **then** a line for a manufactured product has valued stock moves and PR-AC-311
applies to it.

---

## 18. The product margin analysis

**PR-AC-320** (PR-216). **Given** a product with a catalogue price of 100.00 and a cost of 60.00, and,
inside the range, one customer invoice of ten units with a subtotal of 950.00 and a balance of minus
950.00, one customer credit note of two units with a subtotal of 190.00 and a balance of plus 190.00,
and one vendor bill of twelve units with a subtotal of 720.00 and a balance of plus 720.00, **then**
the measures are: invoiced quantity on the sales side eight, average sale unit price 95.00, turnover
760.00, expected sale 800.00, sales gap 40.00, invoiced quantity on the purchase side twelve, average
purchase unit price 60.00, total cost 720.00, normal cost 720.00, purchase gap 0.00, total margin
40.00, expected margin 80.00, total margin rate 40.00 × 100 ÷ 760.00 = 5.263157894736842 to fifteen
decimal places, and expected margin rate 10.00 exactly.

**PR-AC-321** (PR-216). **Given** the same data and the filter `paid` (paid only), **then** only posted
documents whose payment state is in payment, paid or reversed are counted. **Given** `open_paid` (open
and paid), **then** posted documents in every payment state are counted. **Given** `draft_open_paid`
(draft, open and paid), **then** draft documents are counted as well.

**PR-AC-322** (PR-216). **Given** no calling context at all, **then** the range runs from the first of
January to the thirty-first of December of the current year and the filter is `open_paid`.

**PR-AC-323** (PR-216). **Given** a grouped list of two products whose turnovers are 760.00 and 240.00,
**then** the turnover column of the group shows 1000.00, computed by summing the two per-product
values; **and** the average purchase unit price column shows no group total at all.

**PR-AC-324** (PR-216). **Given** a product whose turnover is zero, **then** its total margin rate is
zero rather than a division by zero; **given** its expected sale is zero, **then** its expected margin
rate is zero.

**PR-AC-325** (PR-216). **Given** documents in two currencies inside the range, **then** the average
unit prices divide by the document rate of each document while the turnover and the total cost use
the balance, which is already in the company currency.

**PR-AC-326** (PR-263). **Given** a product margin analysis run today for a range in the past, **when**
the product's catalogue price and cost are changed and the analysis is run again for the same range,
**then** the expected sale and the normal cost change, because they are valued at the **current**
catalogue price and cost and no price history is kept.

---

## 19. Storefront

**PR-AC-330** (PR-186). **Given** a price list with no website, not selectable and with no promotional
code, **when** storefront publishability is evaluated, **then** it is not publishable.

**PR-AC-331** (PR-185). **Given** a price list with no website and the selectable flag set, **then** it
is publishable on every storefront of its company. **Given** a price list with no website, not
selectable, but carrying a promotional code, **then** it is publishable too.

**PR-AC-332** (PR-185). **Given** a price list bound to the storefront "Shop One" and a visitor on
"Shop Two", **then** it is not publishable there.

**PR-AC-333** (PR-187). **Given** a price list carrying the country group "Europe", **when**
availability in Belgium is evaluated, **then** it is available; in Kiribati, it is not. **Given** a
price list with no country group, **then** it is available everywhere. **Given** no country code at
all, **then** every price list is available.

**PR-AC-334** (PR-189). **Given** a visitor whose session names a price list that still exists, is
publishable here and is available in the visitor's country, **when** a page is requested, **then** the
stored identifier is reused and no search is performed.

**PR-AC-335** (PR-189, PR-190). **Given** a visitor whose session names a price list that has since
been bound to another website, **when** a page is requested, **then** the resolution runs again and
the session is rewritten.

**PR-AC-336** (PR-190). **Given** the Basic Price Lists capability withdrawn, **when** a visitor
requests a page, **then** no price list is used at all and catalogue prices are shown.

**PR-AC-337** (PR-190). **Given** a signed-in visitor with a cart, **when** the resolution runs,
**then** the cart's own price list is taken before the contact's.

**PR-AC-338** (PR-190). **Given** a signed-in visitor whose contact price list is not among the price
lists available to them and the available set is not empty, **when** the resolution runs, **then** the
**first** available price list is used.

**PR-AC-339** (PR-191). **Given** a visitor whose geolocated country is Belgium, a price list reachable
from the country group "Europe" that is publishable and selectable, and a second publishable
selectable price list with no country group, **when** the available set is computed, **then** it holds
the first one; **and given** no price list is reachable from "Europe", **then** the set holds the
second one, because a price list with country groups is excluded from that branch when a country code
is known.

**PR-AC-340** (PR-191). **Given** a visitor whose session names a price list that is publishable but
**not** selectable, **when** the available set is computed for the chooser, **then** that price list is
still listed, which is how a promotional code stays usable for the rest of the session.

**PR-AC-341** (PR-115). **Given** a formula rule with a discount of ten on the sales price, **when** a
product page is rendered, **then** a struck-through price is shown; **when** the same product is added
to the cart, **then** the line shows a net unit price and no discount.

**PR-AC-342** (PR-192, PR-193). **Given** the resolved price list is expressed in the euro, **then** the
website's displayed currency is the euro. **Given** an **archived** price list with no website that is
selectable, **then** the publishability test still passes it, which is the recorded compatibility
finding; a corrected rebuild refuses it in both branches.

---

## 20. Point of sale

**PR-AC-350** (PR-195). **Given** a terminal that uses price lists with the available price lists "A"
and "B" and the default price list "C", **when** it is saved, **then** the save is refused with "The
default pricelist must be included in the available pricelists."

**PR-AC-351** (PR-196). **Given** a terminal in the dollar that uses price lists, with an available
price list expressed in the euro, **when** it is saved, **then** the save is refused with "All
available pricelists must be in the same currency as the company or as the Sales Journal set on this
point of sale if you use the Accounting application."

**PR-AC-352** (PR-197). **Given** a terminal of Alpha and a price list of Beta set as its default,
**when** it is saved, **then** the save is refused with "The default pricelist must belong to no
company or the company of the point of sale."

**PR-AC-353** (PR-198). **Given** the same terminal and that price list added to the available price
lists, **when** it is saved, **then** the save is refused with "The selected pricelists must belong to
no company or the company of the point of sale."

**PR-AC-354** (PR-199). **Given** a valid terminal configuration whose default price list's company is
afterwards changed to another company, **when** a session is opened, **then** the checks run again and
the opening is refused with the message of PR-AC-352.

**PR-AC-355** (PR-200). **Given** a rule that becomes valid tomorrow, **when** the terminal loads its
data today, **then** that rule is not loaded; **when** the terminal loads again tomorrow, **then** it
is.

**PR-AC-356** (PR-201). **Given** a terminal whose available price list "Outer" holds a rule based on
"Base", where "Base" is neither available nor the default, **when** the terminal loads its data,
**then** "Base" and its rules are loaded as well.

**PR-AC-357** (PR-200). **Given** a terminal that has already loaded once, **when** it loads again,
**then** it receives the rules changed since the previous load and the rules whose start instant has
passed since then, and nothing else.

---

## 21. The price grid and the exports

**PR-AC-365** (interfaces section 3.1). **Given** the price list "Retail" with a global fixed-price
rule of 8.00 at minimum quantity one hundred and no other rule, and a product at 10.00 per `Units`,
**when** the grid is produced for the quantities one, ten and one hundred, **then** the row reads
10.00, 10.00, 8.00.

**PR-AC-366** (interfaces section 3.1). **Given** a product template with three variants and a rule
scoped to one of them giving a fixed price of 5.00, **when** the grid is produced for that template,
**then** the template row is followed by three indented variant rows and the row of the scoped variant
shows 5.00.

**PR-AC-367** (interfaces section 3.1). **Given** a grid request naming a price list that no longer
exists, **when** the grid is produced, **then** the first price list in the standard ordering is used
and no error is raised.

**PR-AC-368** (interfaces section 3.1). **Given** a grid request with no selected identifiers, **when**
the grid is produced, **then** it is empty and no error is raised.

**PR-AC-369** (interfaces section 2). **Given** a grid request for two products and the quantities one
and five, **when** the delimited-text export is produced, **then** the header row reads `Product`,
`UOM`, `Quantity (1 UoM)`, `Quantity (5 UoM)` — reproduced exactly — and two data rows follow.

**PR-AC-370** (interfaces section 2). **Given** the same request, **when** the spreadsheet export is
produced, **then** the file is named "Pricelist - " followed by the price list's name and the
workbook extension, and every column is at least as wide as its longest value, header included.

**PR-AC-371** (interfaces section 2). **Given** a grid request whose selection is a template with three
variants, **when** either export is produced, **then** the file holds **three** rows, one per variant,
and **no** row for the template itself.

---

## 22. Access, companies and record rules

**PR-AC-380** (PR-222). **Given** a user holding only the internal user capability, **when** a price
list is read, **then** the read succeeds; **when** a price list is created, **then** the creation is
refused.

**PR-AC-381** (PR-222). **Given** an anonymous storefront visitor with the storefront capability
installed, **when** a price list rule is read, **then** the read succeeds and a storefront price can be
computed.

**PR-AC-382** (PR-222). **Given** a purchase administrator, **when** a vendor price is created, edited
or deleted, **then** each operation succeeds; **when** a price list is created, **then** the creation
is refused.

**PR-AC-383** (PR-222). **Given** a sales manager, **when** a vendor price is deleted, **then** the
deletion is refused.

**PR-AC-384** (PR-221). **Given** a user whose allowed companies are Alpha only, **when** the price list
list is read, **then** it holds the price lists of Alpha and the price lists with no company, and not
those of Beta.

**PR-AC-385** (PR-221). **Given** Alpha is the parent of a branch company Gamma and the user is allowed
in Gamma, **when** the price list list is read, **then** the price lists of Alpha **are** visible,
because the record rule tests the ancestor relation.

**PR-AC-386** (PR-145, PR-221). **Given** the same user and an offer whose company is Alpha, **when** a
vendor price is selected for a purchase in Gamma, **then** the offer is **not** a candidate, because
the selection's company test is an equality test although the record rule shows the record.

**PR-AC-387** (PR-223, PR-188). **Given** a portal user, **when** a price list is read, **then** the
promotional code is not returned; **when** a product is read, **then** the cost is not returned.

**PR-AC-388** (PR-228). **Given** a user with read access to price lists and rules and no write access
anywhere, **when** a price is requested, **then** it is returned and nothing is written.

---

## 23. Determinism, snapshots and concurrency

**PR-AC-390** (PR-261, PR-273). **Given** two rules identical in every respect but their identifier,
**when** the same price is requested one hundred times, **then** the rule with the **larger**
identifier applies every time.

**PR-AC-391** (PR-262). **Given** a confirmed sales order line stored at 90.00, **when** the rule that
produced it is deleted, the product's catalogue price is doubled and the currency rate is halved,
**then** the line still reads 90.00.

**PR-AC-392** (PR-263). **Given** a price computed for a past date from a rule based on the cost,
**when** the product's cost is changed and the same computation is repeated with the same past date,
**then** the result changes, because no cost history is kept.

**PR-AC-393** (PR-260). **Given** any price request, **when** it completes, **then** no record has been
created, modified or deleted and no message has been posted.

**PR-AC-394** (PR-256). **Given** a quotation being priced in one transaction and a rule of its price
list changed and committed by another transaction meanwhile, **then** the running computation is
unaffected.

**PR-AC-395** (PR-257). **Given** a quotation priced yesterday, **when** a rule of its price list is
changed today, **then** the quotation's lines are unchanged until *Update Prices* is run.

**PR-AC-396** (PR-258). **Given** the storefront cache of the price lists available per website,
**when** any price list is created, modified or deleted, **then** that cache is cleared.

---

## 24. Mapping of the former scenario identifiers

An earlier draft of this folder numbered its scenarios in the same `PR-AC-nnn` form but with a
different allocation, and cited rules under the earlier `PR-RULE-nnn` scheme. Both were renumbered
here: the rules into the `PR-nnn` scheme of
[`business-rules.md`](business-rules.md#22-index-of-rule-identifiers), and the scenarios as below, so
that any external reference to the earlier draft can be resolved.

| Former scenario | Scenario here | Former scenario | Scenario here |
|---|---|---|---|
| PR-AC-001 | PR-AC-001 | PR-AC-155 | PR-AC-190 |
| PR-AC-002 | PR-AC-002 | PR-AC-156 | PR-AC-191 |
| PR-AC-003 | PR-AC-003 | PR-AC-157 | PR-AC-193 |
| PR-AC-004 | PR-AC-004 | PR-AC-158 | PR-AC-194 |
| PR-AC-005 | PR-AC-007 | PR-AC-159 | PR-AC-197 |
| PR-AC-006 | PR-AC-009 | PR-AC-160 | PR-AC-198 |
| PR-AC-007 | PR-AC-010 | PR-AC-161 | PR-AC-199 |
| PR-AC-008 | PR-AC-012 | PR-AC-170 | PR-AC-210 |
| PR-AC-009 | PR-AC-011 | PR-AC-171 | PR-AC-211 |
| PR-AC-010 | PR-AC-020 | PR-AC-172 | PR-AC-212 |
| PR-AC-011 | PR-AC-022 | PR-AC-173 | PR-AC-213 |
| PR-AC-012 | PR-AC-023 | PR-AC-174 | PR-AC-219 |
| PR-AC-013 | PR-AC-024 | PR-AC-175 | PR-AC-214 |
| PR-AC-014 | PR-AC-060 | PR-AC-176 | PR-AC-223 |
| PR-AC-015 | PR-AC-035 | PR-AC-177 | PR-AC-224 |
| PR-AC-016 | PR-AC-031 | PR-AC-185 | PR-AC-230 |
| PR-AC-017 | PR-AC-032 | PR-AC-186 | PR-AC-231 |
| PR-AC-018 | PR-AC-027 | PR-AC-187 | PR-AC-232 |
| PR-AC-019 | PR-AC-028 | PR-AC-188 | PR-AC-233 |
| PR-AC-020 | PR-AC-030 | PR-AC-189 | PR-AC-235 |
| PR-AC-021 | PR-AC-034 | PR-AC-190 | PR-AC-236 |
| PR-AC-030 | PR-AC-040 | PR-AC-191 | PR-AC-237 |
| PR-AC-031 | PR-AC-041 | PR-AC-192 | PR-AC-238 |
| PR-AC-032 | PR-AC-042 | PR-AC-193 | PR-AC-239 |
| PR-AC-033 | PR-AC-043 | PR-AC-200 | PR-AC-250 |
| PR-AC-034 | PR-AC-044 | PR-AC-201 | PR-AC-251 |
| PR-AC-035 | PR-AC-045 | PR-AC-202 | PR-AC-254 |
| PR-AC-036 | PR-AC-047 | PR-AC-203 | PR-AC-255 |
| PR-AC-037 | PR-AC-048 | PR-AC-204 | PR-AC-257 |
| PR-AC-038 | PR-AC-049 | PR-AC-205 | PR-AC-259 |
| PR-AC-039 | PR-AC-050 | PR-AC-206 | PR-AC-260 |
| PR-AC-040 | PR-AC-052 | PR-AC-207 | PR-AC-261 |
| PR-AC-041 | PR-AC-056 | PR-AC-208 | PR-AC-252 |
| PR-AC-042 | PR-AC-058 | PR-AC-209 | PR-AC-253 |
| PR-AC-043 | PR-AC-059 | PR-AC-215 | PR-AC-270 |
| PR-AC-044 | PR-AC-061 | PR-AC-216 | PR-AC-271 |
| PR-AC-045 | PR-AC-062 | PR-AC-217 | PR-AC-272 |
| PR-AC-046 | PR-AC-063 | PR-AC-218 | PR-AC-274 |
| PR-AC-047 | PR-AC-064 | PR-AC-219 | PR-AC-275 |
| PR-AC-048 | PR-AC-064 | PR-AC-220 | PR-AC-273 |
| PR-AC-049 | PR-AC-067 | PR-AC-225 | PR-AC-285 |
| PR-AC-050 | PR-AC-069 | PR-AC-226 | PR-AC-287 |
| PR-AC-051 | PR-AC-069 | PR-AC-227 | PR-AC-288 |
| PR-AC-052 | PR-AC-065 | PR-AC-228 | PR-AC-289 |
| PR-AC-060 | PR-AC-070 | PR-AC-229 | PR-AC-290 |
| PR-AC-061 | PR-AC-071 | PR-AC-230 | PR-AC-291 |
| PR-AC-062 | PR-AC-071 | PR-AC-231 | PR-AC-292 |
| PR-AC-063 | PR-AC-073 | PR-AC-232 | PR-AC-294 |
| PR-AC-064 | PR-AC-076 | PR-AC-240 | PR-AC-365 |
| PR-AC-065 | PR-AC-077 | PR-AC-241 | PR-AC-366 |
| PR-AC-066 | PR-AC-078 | PR-AC-242 | PR-AC-367 |
| PR-AC-067 | PR-AC-079 | PR-AC-243 | PR-AC-369 |
| PR-AC-068 | PR-AC-081 | PR-AC-244 | PR-AC-370 |
| PR-AC-069 | PR-AC-082 | PR-AC-250 | PR-AC-330 |
| PR-AC-070 | PR-AC-083 | PR-AC-251 | PR-AC-331 |
| PR-AC-080 | PR-AC-090 | PR-AC-252 | PR-AC-332 |
| PR-AC-081 | PR-AC-091 | PR-AC-253 | PR-AC-037 |
| PR-AC-082 | PR-AC-092 | PR-AC-254 | PR-AC-333 |
| PR-AC-083 | PR-AC-095 | PR-AC-255 | PR-AC-341 |
| PR-AC-084 | PR-AC-096 | PR-AC-256 | PR-AC-350 |
| PR-AC-085 | PR-AC-094 | PR-AC-257 | PR-AC-351 |
| PR-AC-086 | PR-AC-100 | PR-AC-258 | PR-AC-352 and PR-AC-353 |
| PR-AC-087 | PR-AC-101 | PR-AC-259 | PR-AC-354 |
| PR-AC-088 | PR-AC-102 | PR-AC-260 | PR-AC-355 |
| PR-AC-089 | PR-AC-103 | PR-AC-270 | PR-AC-380 |
| PR-AC-090 | PR-AC-104 | PR-AC-271 | PR-AC-381 |
| PR-AC-091 | PR-AC-105 | PR-AC-272 | PR-AC-382 |
| PR-AC-092 | PR-AC-107 | PR-AC-273 | PR-AC-384 |
| PR-AC-093 | PR-AC-108 | PR-AC-274 | PR-AC-387 |
| PR-AC-094 | PR-AC-113 | PR-AC-280 | PR-AC-390 |
| PR-AC-095 | PR-AC-112 | PR-AC-281 | PR-AC-391 |
| PR-AC-100 | PR-AC-120 | PR-AC-282 | PR-AC-392 |
| PR-AC-101 | PR-AC-122 | PR-AC-110 | PR-AC-140 |
| PR-AC-102 | PR-AC-123 | PR-AC-111 | PR-AC-141 |
| PR-AC-103 | PR-AC-124 | PR-AC-112 | PR-AC-142 |
| PR-AC-104 | PR-AC-121 | PR-AC-113 | PR-AC-143 |
| PR-AC-105 | PR-AC-125 | PR-AC-114 | PR-AC-144 |
| PR-AC-130 | PR-AC-165 | PR-AC-115 | PR-AC-147 |
| PR-AC-131 | PR-AC-167 | PR-AC-116 | PR-AC-148 |
| PR-AC-132 | PR-AC-168 | PR-AC-117 | PR-AC-149 |
| PR-AC-133 | PR-AC-169 | PR-AC-118 | PR-AC-150 |
| PR-AC-134 | PR-AC-170 | PR-AC-119 | PR-AC-151 |
| PR-AC-135 | PR-AC-171 | PR-AC-120 | PR-AC-152 and PR-AC-153 |
| PR-AC-136 | PR-AC-173 | PR-AC-145 | PR-AC-180 |
| PR-AC-137 | PR-AC-174 | PR-AC-146 | PR-AC-181 |
| PR-AC-138 | PR-AC-175 | PR-AC-147 | PR-AC-182 |

Scenarios that appear in no earlier draft and are new to this consolidation: PR-AC-005, PR-AC-006,
PR-AC-008, PR-AC-021, PR-AC-025, PR-AC-026, PR-AC-029, PR-AC-033, PR-AC-036, PR-AC-046, PR-AC-051,
PR-AC-053 to PR-AC-055, PR-AC-057, PR-AC-066, PR-AC-068, PR-AC-072, PR-AC-074, PR-AC-075, PR-AC-080,
PR-AC-084 to PR-AC-086, PR-AC-093, PR-AC-097 to PR-AC-099, PR-AC-106, PR-AC-109 to PR-AC-111,
PR-AC-114, PR-AC-126 to PR-AC-131, PR-AC-145, PR-AC-146, PR-AC-154 to PR-AC-159, PR-AC-166,
PR-AC-172, PR-AC-176, PR-AC-183, PR-AC-192, PR-AC-195, PR-AC-196, PR-AC-200, PR-AC-201, PR-AC-215
to PR-AC-218, PR-AC-220 to PR-AC-222, PR-AC-234, PR-AC-240 to PR-AC-243, PR-AC-256, PR-AC-258,
PR-AC-262, PR-AC-263, PR-AC-276, PR-AC-277, PR-AC-286, PR-AC-293, PR-AC-300 to PR-AC-326, PR-AC-334
to PR-AC-340, PR-AC-342, PR-AC-356, PR-AC-357, PR-AC-368, PR-AC-371, PR-AC-383, PR-AC-385,
PR-AC-386, PR-AC-388, PR-AC-393 to PR-AC-396.

---

## 25. Reconciliation notes

1. **The scenario numbering.** Only one of the two source descriptions carried acceptance scenarios,
   under a numbering with wide gaps between its blocks. Those gaps have been reused for the scenarios
   the other description's material demands — the margins, the product margin analysis, the storefront
   resolution, the terminal payload, the price grid exports and the concurrency invariants — so the
   blocks stay contiguous by subject. Section 24 maps every former identifier.

2. **The rules cited.** The earlier scenarios cited rules of the form `PR-RULE-nnn`. Those identifiers
   no longer exist: the rule catalogue was renumbered into one scheme, mapped in
   [`business-rules.md`](business-rules.md#23-mapping-of-the-former-rule-identifiers). Every citation
   here uses the new identifier.

3. **The provisioning after a currency change.** The earlier scenario asserted that changing a
   company's currency makes the provisioning run again so that the company's default price list
   carries the new currency. It does not: the re-run is guarded by a capability transition that a
   currency change cannot produce. PR-AC-012 asserts the **observed** behaviour and names the
   corrected one.

4. **The event-ticket warning.** The earlier scenario asserted the warning with the words "minimum
   quantity" spelled out. The shipped text uses the shortened form, and the two branches differ.
   PR-AC-065 and PR-AC-066 reproduce both texts exactly.

5. **The vendor price learned from a confirmed order.** The earlier scenario asserted that the created
   offer takes the unit of the order line. That is true only when the line had selected an offer, and
   in that case the vendor's product name and code come from the **offer** while the unit comes from
   the **line**. PR-AC-276 asserts all three.

6. **The price grid export header.** The earlier scenario expanded the column headers into words. They
   are contractual and are asserted verbatim in PR-AC-369.

7. **The count of margin measures.** The earlier description spoke of fifteen measures. PR-AC-320 lists
   the fourteen numeric measures with concrete values, and PR-AC-323 asserts that thirteen of them sum
   in a grouped list while the average purchase unit price does not.

8. **The subtotal of the worked purchase line.** The earlier scenario asserted a subtotal of 3806.25
   for five units at 785.00 with a discount of three per cent. The arithmetic gives
   5 × 785.00 × 0.97 = 3807.25, and PR-AC-250 asserts that figure with its intermediate value. A
   rebuild tested against the old number would be tuned to an error of one unit of currency.

9. **The unit of the surcharge in the tonne example.** The earlier scenario applied a surcharge to a
   rule without saying which computation kind carries one. Only a formula rule has a surcharge, so
   PR-AC-077 states the kind, and it also states the scaled value of the surcharge per kilogram, which
   is what makes the result 0.09 rather than 0.10 minus ten.
