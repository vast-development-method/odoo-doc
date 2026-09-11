# Products and Catalog — Acceptance Criteria

Numbered scenarios in Given / When / Then form with concrete values. An implementation is
behaviourally equivalent when every scenario below holds. Scenarios are grouped by subject; within
a group the happy path comes first, then each validation failure, then the edge cases.

Unless stated otherwise, the acting user is a product manager, a single company named **Main
Company** is in play, its currency is the euro whose rounding multiple is one hundredth, the
`Product Price` precision is two digits, the `Product Unit` precision is two digits, and the
barcode nomenclature in force is the shipped classic Default Nomenclature.

**A note on quoted system messages.** Text shown inside a block quote or between quotation marks in
this file is the message the system itself emits, reproduced character for character so that an
implementation can match it. A few of those messages contain abbreviations the system prints:
`URL` for uniform resource locator, `FNC1` for the function code one separator, `GS1` for Global
Standards One, and `FEFO` for first expiry first out. They are reproduced because the text is a
contract; everywhere outside a quoted message this folder writes such terms in full.

---

## A. Creating products and variants

### A-1 — Creating a product with no attributes gives exactly one variant

**Given** no product named "Desk Pad" exists.
**When** a product manager creates a Product Template with name "Desk Pad", type `consu`, sales
price 12.00, no attribute line.
**Then** one Product Template exists with sales price 12.00; exactly one Product Variant exists for
it; that variant's combination is empty; its combination indices are the empty string; its sales
price is 12.00 and its price extra is 0.00; the template's variant count is 1.

### A-2 — Values supplied at creation reach the variant

**Given** the same conditions as A-1.
**When** the manager creates the template supplying, in the same operation, a barcode of
`5410011234567`, an internal reference of `DESKPAD-01`, a cost of 7.50, a weight of 0.30 and a
volume of 0.001.
**Then** the created variant carries barcode `5410011234567`, internal reference `DESKPAD-01`, cost
7.50, weight 0.30 and volume 0.001; the template's computed mirrors of those five fields read back
the same values.

### A-3 — Adding two attributes generates six variants

**Given** the template "Office Chair" from A-1-style creation with sales price 100.00 and no
attribute lines; an attribute **Size** with variant creation `always`, sequence 10, offering values
**S** (sequence 1), **M** (sequence 2), **L** (sequence 3); an attribute **Colour** with variant
creation `always`, sequence 20, offering **Black** (sequence 1) and **White** (sequence 2).
**When** the manager adds a Size line offering S, M and L, and a Colour line offering Black and
White, in one save.
**Then** five Template Attribute Values exist (three for Size, two for Colour), each with extra
price 0.00; six Product Variants exist; the original attribute-less variant no longer exists; the
six variants' display names are "Office Chair (S, Black)", "Office Chair (S, White)", "Office Chair
(M, Black)", "Office Chair (M, White)", "Office Chair (L, Black)" and "Office Chair (L, White)";
every variant's sales price is 100.00.

### A-4 — Extra prices change variant prices without changing the variant set

**Given** the state after A-3.
**When** the manager sets the extra price of *Size: L* to 5.00 and of *Colour: White* to 2.00.
**Then** the variant set is unchanged — still six variants, none archived, none created — and the
six sales prices are 100.00, 102.00, 100.00, 102.00, 105.00 and 107.00 in the order of A-3.

### A-5 — An exclusion removes one variant

**Given** the state after A-4.
**When** the manager creates an exclusion whose owning value is *Colour: White*, whose template is
"Office Chair", and whose excluded value list is [*Size: L*].
**Then** variant generation runs; the variant "Office Chair (L, White)" is deleted (nothing
references it); five active variants remain; asking whether the combination {Size: L, Colour: White}
is possible returns false; asking whether {Size: L, Colour: Black} is possible returns true.

### A-6 — Removing the exclusion brings the variant back

**Given** the state after A-5.
**When** the manager deletes the exclusion.
**Then** variant generation runs and a variant for {Size: L, Colour: White} exists again, active,
with price extra 7.00 and sales price 107.00. Its identifier may differ from the deleted one.

### A-7 — An exclusion whose variant is referenced archives instead of deleting

**Given** the state after A-4, and a confirmed sales order line referencing the variant "Office
Chair (L, White)".
**When** the manager creates the exclusion of A-5.
**Then** the variant "Office Chair (L, White)" is **archived**, not deleted; the sales order line
still resolves to it; the template has five active variants and one archived variant; the payload's
archived-combination list contains the tuple of that combination's value identifiers.

### A-8 — Adding a single-value attribute does not recreate the variants

**Given** the state after A-3, with the six variants carrying the internal references `CH-01`
through `CH-06`.
**When** the manager adds an attribute line for **Material** offering only **Oak**.
**Then** the six variants are **not** recreated: their identifiers and internal references are
unchanged; each variant's combination now additionally contains *Material: Oak*; the display names
are unchanged, because *Material: Oak* comes from a single-value line and is filtered out of the
combination name.

### A-9 — Making the single-value attribute multi-value creates the new variants

**Given** the state after A-8.
**When** the manager adds **Walnut** to the Material line.
**Then** twelve variants exist; the six original ones keep their identifiers and now display as
"Office Chair (S, Black, Oak)" and so on; six new variants are created for the Walnut combinations.

### A-10 — The generation ceiling is enforced

**Given** the system parameter `product.dynamic_variant_limit` is unset, so the limit is one
thousand; a template with attributes whose cartesian product has 1001 combinations, none excluded,
all attributes with variant creation `always`; the template currently has no variants.
**When** the manager saves the attribute configuration.
**Then** the operation is refused with

> The number of variants to generate is above allowed limit. You should either not generate
> variants for each combination or generate them on demand from the sales order. To do so, open the
> form view of attributes and change the mode of *Create Variants*.

and no variant is created.

### A-11 — Exactly the limit is allowed

**Given** the same, with a cartesian product of exactly 1000 combinations.
**When** the manager saves.
**Then** 1000 variants are created and no error is raised.

### A-12 — A configuration that leaves no possible variant is refused

**Given** a template with one attribute **Colour** offering **Black** and **White**, and exclusions
making both impossible: *Black* excludes *Black* and *White* excludes *White*.
**When** the configuration is saved.
**Then** every candidate combination is filtered out, every variant goes to the unlink set, the last
variant's deletion deletes the template, and the operation is refused with

> This configuration of product attributes, values, and exclusions would lead to no possible
> variant. Please archive or delete your product directly if intended.

---

## B. Dynamic variants

### B-1 — A dynamic template starts with no variants

**Given** an attribute **Nib** with variant creation `dynamic` offering **Fine**, **Medium** and
**Broad**, and an attribute **Barrel** with variant creation `dynamic` offering **Brass** and
**Steel**.
**When** the manager creates a template "Engraved Pen" with sales price 40.00 and adds both
attribute lines.
**Then** the template has **zero** variants; the template is **not** deleted; the template's
dynamically-created flag is true; the template's configurable flag is true.

### B-2 — Ordering a combination creates exactly one variant

**Given** the state after B-1, with extra prices *Broad* 3.00 and *Steel* 6.00 and all others 0.00.
**When** a salesperson configures {Nib: Broad, Barrel: Steel} and the consuming domain calls the
create-on-demand procedure with that combination.
**Then** one variant is created; its combination indices are the sorted identifier string of those
two values; its price extra is 9.00 and its sales price is 49.00; the template now has one variant;
the other five combinations still have none.

### B-3 — Ordering the same combination again reuses the variant

**Given** the state after B-2.
**When** the create-on-demand procedure is called again with {Nib: Broad, Barrel: Steel}.
**Then** the existing variant is returned; no second variant is created; the uniqueness index is not
violated.

### B-4 — A second combination creates a second variant

**Given** the state after B-2.
**When** the create-on-demand procedure is called with {Nib: Fine, Barrel: Brass}.
**Then** a second variant is created with price extra 0.00 and sales price 40.00; the template has
two variants.

### B-5 — An archived dynamic variant is not silently revived

**Given** the state after B-2 with that variant archived.
**When** the create-on-demand procedure is called with {Nib: Broad, Barrel: Steel}.
**Then** the archived variant is found and returned **as it is**, still archived; the full
possibility predicate returns false for that combination, because for a dynamic template an
archived variant makes the combination impossible; the consuming domain must therefore refuse the
line.

### B-6 — A non-dynamic template refuses on-demand creation

**Given** the "Office Chair" of A-3, whose attributes both have variant creation `always`, and
whose variant for {Size: L, Colour: White} has been deleted by an exclusion.
**When** the create-on-demand procedure is called with {Size: L, Colour: White}.
**Then** nothing is returned; no variant is created; a warning is written to the technical log when
the caller asked for logging.

### B-7 — Deleting the last variant of a dynamic template does not delete the template

**Given** the state after B-2 (one variant).
**When** that variant is deleted.
**Then** the template still exists with zero variants.

### B-8 — Deleting the last variant of a non-dynamic template deletes the template

**Given** a template "Desk Pad" with one variant and no dynamic attribute.
**When** that variant is deleted.
**Then** both the variant and the template are gone.

---

## C. Combination possibility

### C-1 — The empty combination on an attribute-less template is possible

**Given** the template "Desk Pad" with no attribute lines and one variant.
**When** the full possibility predicate is asked about the empty combination.
**Then** it returns true.

### C-2 — The empty combination on a template with attributes is impossible

**Given** the "Office Chair" of A-3.
**When** the full possibility predicate is asked about the empty combination.
**Then** it returns false, because the configuration filter finds zero members against two lines.

### C-3 — A combination missing one attribute is impossible

**Given** the "Office Chair" of A-3.
**When** the predicate is asked about {Size: M}.
**Then** it returns false.

### C-4 — A combination naming a foreign attribute is impossible

**Given** the "Office Chair" of A-3 and a Template Attribute Value *Finish: Matte* belonging to
another template.
**When** the predicate is asked about {Size: M, Colour: Black, Finish: Matte}.
**Then** it returns false: the number of members does not match the number of lines, and the covered
lines do not match.

### C-5 — A combination naming an archived value is impossible

**Given** the "Office Chair" of A-3 with *Colour: White* archived.
**When** the predicate is asked about {Size: M, Colour: White}.
**Then** it returns false: the subset test against the active materialised values fails.

### C-6 — A multi-checkbox attribute may contribute zero values

**Given** a template "Sandwich" with an attribute **Bread** (variant creation `always`, display type
radio, values **White** and **Rye**) and an attribute **Extras** (display type `multi`, variant
creation `no_variant`, values **Pickles**, **Onions**, **Cheese**).
**When** the predicate is asked about {Bread: White}, with no-variant attributes **not** ignored.
**Then** it returns true: the multi-checkbox line is excluded from both the member count and the
covered-line comparison, so a combination with only the bread value is complete.
**And when** it is asked about {Bread: White, Extras: Pickles, Extras: Cheese}, it also returns
true.

### C-7 — A parent exclusion makes a child value impossible

**Given** a template "Desk" with attribute **Legs** offering **Steel** and **Wood**; a template
"Desk Lamp" with attribute **Shade** offering **Black** and **Chrome**; and an exclusion whose
owning value is *Legs: Steel*, whose template is "Desk Lamp", and whose excluded value list is
[*Shade: Chrome*].
**When** the predicate is asked whether {Shade: Chrome} is possible on "Desk Lamp" with the parent
combination {Legs: Steel}.
**Then** it returns false.
**And when** it is asked about {Shade: Black} with the same parent, it returns true.
**And when** it is asked about {Shade: Chrome} with parent {Legs: Wood}, it returns true.

### C-8 — A parent exclusion with no values excludes the whole product

**Given** the same two templates, and an exclusion whose owning value is *Legs: Steel*, whose
template is "Desk Lamp", and whose excluded value list is **empty**.
**When** the parent-exclusion map is computed for "Desk Lamp" with parent {Legs: Steel}.
**Then** the map maps *Legs: Steel* to **every** materialised value of every attribute line of
"Desk Lamp", so every combination of "Desk Lamp" is impossible under that parent.

### C-9 — Exclusions are completed with their inverses in the payload

**Given** a template with values identified 101 (*Size: S*), 102 (*Size: M*), 103 (*Size: L*), 104
(*Colour: Black*), 105 (*Colour: White*), and one exclusion in which 104 excludes 103 and 102.
**When** the attribute-exclusion payload is built.
**Then** the exclusions entry is exactly `{101: [], 102: [104], 103: [104], 104: [103, 102],
105: []}`.

### C-10 — The archived-combination list excludes combinations that also have an active variant

**Given** a template with an archived variant for combination *C* and an active variant for the same
combination *C* (possible because the uniqueness index only constrains active rows).
**When** the payload is built.
**Then** the archived-combination list does **not** contain *C*.

### C-11 — The closest possible combination drops the last value first

**Given** the "Office Chair" of A-4 with the exclusion of A-5 ("White excludes L"); the Size line
has sequence 10 and the Colour line sequence 20.
**When** the closest possible combination is requested for {Size: L, Colour: White}.
**Then** the result is {Size: L, Colour: Black}: the last member in line-then-value order is *Colour:
White*, so it is dropped first, and the enumeration then completes the colour with Black.

### C-12 — The first possible combination follows the sequences

**Given** the "Office Chair" of A-4 with the exclusion of A-5.
**When** the first possible combination is requested with no necessary values.
**Then** the result is {Size: S, Colour: Black}.

### C-13 — An archived template yields no combination

**Given** the "Office Chair" archived.
**When** the possible combinations are enumerated.
**Then** the enumeration yields nothing.

---

## D. Extra prices

### D-1 — Variant price extra is the sum of its members

**Given** the "Office Chair" of A-4.
**When** the price extra of "Office Chair (L, White)" is read.
**Then** it is 7.00 (5.00 + 2.00).

### D-2 — A never-create-variant value contributes through the context

**Given** the "Business Card Pack" of [calculations.md](calculations.md), section 7.2: sales price
25.00; attribute **Paper** (`always`) with *Matte* extra 0.00 and *Glossy* extra 4.00; attribute
**Engraving** (`no_variant`) with *None* extra 0.00 and *Gold foil* extra 12.00.
**When** a line is created for the variant "Business Card Pack (Glossy)" with the chosen combination
{Paper: Glossy, Engraving: Gold foil}.
**Then** the variant's own price extra is 4.00; the no-variant extra computed from the combination
is 12.00; the attributes extra price is 16.00; the unit price before any pricelist rule is 41.00.

### D-3 — A value already on the variant is not double-counted

**Given** the same, and suppose the Paper attribute's mode was later changed to `no_variant` while
the variants still carry *Paper: Glossy*.
**When** the no-variant extra is computed for the same chosen combination.
**Then** it is still 12.00, not 16.00, because *Paper: Glossy* is excluded for being a member of the
variant's own combination.

### D-4 — Unit conversion on the variant sales price converts the base only

**Given** "Office Chair (L, White)" with template sales price 100.00, price extra 7.00, default unit
"Units", and a unit "Dozens" for which one dozen equals twelve units.
**When** the variant's sales price is read with "Dozens" named in the context.
**Then** the value is 1207.00: the template price is converted (100.00 × 12 = 1200.00) and the extra
is added unconverted.

### D-5 — Writing the variant sales price inverts that computation

**Given** the same context.
**When** 1300.00 is written to the variant's sales price.
**Then** the template's sales price becomes 1300.00 ÷ 12 − 7.00 = 101.3333…, stored at the
`Product Price` precision as 101.33.

### D-6 — The general price computation converts extras too

**Given** the same product.
**When** the general price computation is invoked for the sales price with target unit "Dozens".
**Then** the result is (100.00 + 7.00) × 12 = 1284.00, because that computation adds the extra
**before** converting. Both D-4 and D-6 must hold: they are different entry points.

### D-7 — The cost falls back to the first variant on a template

**Given** a template with three variants whose costs are 7.00, 7.00 and 7.00, so the template's
computed cost mirror reads 0.00 (more than one variant).
**When** the general price computation is invoked for the cost on the **template**.
**Then** it takes the first variant's cost, 7.00.

---

## E. Combos

### E-1 — A choice group's base price is the minimum item price

**Given** a choice group "Main" in euro with items Burger (sales price 9.00), Veggie Wrap (8.00) and
Steak (14.00), all priced in euro.
**When** the base price is read.
**Then** it is 8.00.

### E-2 — An empty choice group has a base price of zero and is refused on save

**Given** a new choice group with no items.
**When** it is saved.
**Then** the base price reads 0.00 and the save is refused with "A combo choice must contain at
least 1 product."

### E-3 — Prorating a combo price

**Given** a combo product "Lunch Menu" at 15.00 in euro with choice groups Main (sequence 10, base
price 8.00), Side (sequence 20, base price 3.50) and Drink (sequence 30, base price 2.00); item
extra prices Steak 3.00 and Soda 0.50, everything else 0.00; no pricelist rule applies; the document
currency is euro.
**When** a customer orders the Lunch Menu choosing Steak, Salad and Soda.
**Then** the total base price is 13.50; the shares are 8.89, 3.89 and 2.22; their sum is 15.00 so
the remainder is 0.00; the combo product's line unit price is 0.00; the Steak line is 11.89, the
Salad line 3.89 and the Soda line 2.72; the order total is 18.50.

### E-4 — The rounding remainder goes to the last choice group

**Given** the same combo product priced at 10.00 with all three base prices equal to 1.00.
**When** the proration runs.
**Then** each share rounds to 3.33, their sum is 9.99, the remainder is 0.01, and the **Drink**
group — the last in sequence order — receives it, giving shares 3.33, 3.33 and 3.34 summing exactly
to 10.00.

### E-5 — Zero base prices distribute evenly

**Given** the same combo product priced at 10.00 with all three base prices equal to 0.00.
**When** the proration runs.
**Then** the total base price is zero, so the even-share branch applies: each share is
round_to_currency(10.00 ÷ 3) = 3.33; the remainder 0.01 goes to the Drink group; the shares are
3.33, 3.33 and 3.34.

### E-6 — A combo may not contain a combo

**Given** a combo product "Family Menu".
**When** a choice group item is created naming a variant of "Family Menu".
**Then** the save is refused with "A combo choice can't contain products of type \"combo\"."

### E-7 — A choice group may not repeat a product

**Given** a choice group with items Burger and Burger.
**When** it is saved.
**Then** the save is refused with "A combo choice can't contain duplicate products."

### E-8 — A combo product may not have attributes

**Given** a template with one attribute line.
**When** its type is changed to `combo` on the form.
**Then** the change is refused with "Combo products can't have attributes."

### E-9 — A product inside a combo may not become a combo

**Given** a variant of the template "Burger" is an item of some choice group.
**When** the "Burger" template's type is changed to `combo` on the form.
**Then** the change is refused with "This product is part of a combo, so its type can't be changed
to \"combo\"."

### E-10 — A combo product with no choice group is refused

**Given** a template with type `combo` and an empty choice group list.
**When** it is saved.
**Then** the save is refused with "A combo product must contain at least 1 combo choice."

### E-11 — A sellable combo may not contain an unsellable product

**Given** a combo product with the sellable flag true, one of whose items names a variant whose
template has the sellable flag false.
**When** it is saved.
**Then** the save is refused with "A sellable combo product can only contain sellable products."

### E-12 — Changing the type away from combo empties the choice groups

**Given** a combo product with three choice groups.
**When** its type is written to `consu`.
**Then** the choice group list becomes empty; the three Product Combo records themselves still
exist.

### E-13 — Setting the type to combo clears the purchasable flag

**Given** a product with the purchasable flag true and no attribute line and not itself inside a
combo.
**When** its type is changed to `combo` on the form.
**Then** the purchasable flag becomes false.

### E-14 — Deleting a variant deletes the combo items naming it

**Given** a choice group with two items, one of which names the variant "Office Chair (L, White)".
**When** an exclusion removes that variant and variant generation unlinks it.
**Then** that combo item is deleted; the choice group now has one item; no warning is shown.

---

## F. Barcodes — the classic nomenclature

### F-1 — The check digit of a twelve-digit prefix

**When** the check digit of `2134567012500` is computed (the last character being a placeholder).
**Then** the reversed digits after dropping the last character are
`0,5,2,1,0,7,6,5,4,3,1,2`; the even-position sum is 13; the odd-position sum is 23; the weighted
total is 62; the check digit is 8; the complete barcode is `2134567012508`.

### F-2 — The encoding check

| Input | Encoding | Result |
|---|---|---|
| `0002` | `ean8` | false — wrong length |
| `12345678` | `ean8` | false — the check digit of `12345678` is 0, not 8 |
| `12345670` | `ean8` | true |
| `02003405` | `ean8` | true |
| `0401234512345` | `ean13` | false — the first character is zero |
| `2134567012508` | `ean13` | true |
| `anything` | `any` | true |

### F-3 — Parsing a weight barcode

**Given** a nomenclature with one rule: encoding `ean13`, type `product`, pattern `21.....{NNDDD}.`
**When** `2134567012508` is parsed.
**Then** the result is: type `product`, encoding `ean13`, code `2134567012508`, value **1.25**, base
code **`2134567000000`**.

### F-4 — The product must be stored under the zeroed base code

**Given** the state of F-3 and a product whose stored barcode is `2134567012508`.
**When** the scan `2134567012508` is used to find the product by base code.
**Then** nothing is found: the base code is `2134567000000`, not the stored value. The product must
be stored with the zeros.

### F-5 — No rule matching gives the error outcome

**Given** a nomenclature with one rule: encoding `ean8`, pattern `........`
**When** `0002` is parsed.
**Then** the result is: type `error`, encoding empty, code `0002`, base code `0002`, value 0.
**And when** `12345678` is parsed (wrong check digit), the same error outcome is produced.

### F-6 — A whole-barcode numeric rule

**Given** a nomenclature with one rule: encoding `ean8`, pattern `{NNNNNNNN}`
**When** `12345670` is parsed.
**Then** type `product`, encoding `ean8`, code `12345670`, base code `00000000`, value
**12345670.0**.
**And when** `02003405` is parsed, base code `00000000` and value **2003405.0**.

### F-7 — Rule order decides between competing rules

**Given** a nomenclature with two `ean8` rules: rule A with pattern `11.....{N}` and rule B with
pattern `66{NN}....`, both with the same sequence so that identifier order applies with A first.
**When** `11012344` is parsed, **then** base code `11012340` and value 4.
**When** `66012344` is parsed, **then** base code `66002344` and value 1.
**When** `16012344` is parsed, **then** the error outcome with base code `16012344` and value 0.

### F-8 — Sequence beats creation order

**Given** a nomenclature with two `ean13` rules: rule A (created first) pattern `.....{NNNDDDD}.`
sequence 3; rule B pattern `22......{NNDD}.` sequence 2.
**When** `2012345610255` is parsed — which only rule A can match — **then** base code
`2012300000008` and value **456.1025**.
**When** `2212345610259` is parsed — which both can match — **then** rule B wins on sequence: base
code `2212345600007` and value **10.25**.
**When** rule A's sequence is then set to 1 and `2212345610259` is parsed again, **then** rule A
wins: base code `2212300000002` and value **456.1025**.

### F-9 — A thirteen-digit rule with a partial numeric field

**Given** a nomenclature with one `ean13` rule, pattern `1........{NND}.`
**When** `1020034051259` is parsed.
**Then** type `product`, base code `1020034050009`, value **12.5**.

### F-10 — Pattern validation

| Pattern written on a rule | Outcome |
|---|---|
| `......{}..` | refused, "There is a syntax error in the barcode pattern ......{}..: empty braces." |
| `......{DN}` | refused, "…: braces can only contain N's followed by D's." |
| `....{NN}{DD}` | refused, "…: a rule can only contain one pair of braces." |
| `*` | refused, " '*' is not a valid Regex Barcode Pattern. Did you mean '.*'?" |
| `**>>>{ND}` | refused, "The barcode pattern **>>>{ND} does not lead to a valid regular expression." |
| `..>>>{ND}` | accepted |

### F-11 — A Global Standards One rule pattern needs two groups

**Given** a nomenclature in Global Standards One mode.
**When** a rule is created with pattern `01\d{14}` (no groups) and name "Trade item".
**Then** the save is refused with

> The rule pattern "Trade item" is not valid, it needs two groups:
> 	- A first one for the Application Identifier (usually 2 to 4 digits);
> 	- A second one to catch the value.

### F-12 — Decoding a lot-level trade item uniform resource identifier

**When** `urn:epc:class:lgtin : 4012345.012345.998877` is parsed.
**Then** two records are produced: a `product` record with value `04012345123456` and a `lot` record
with value `998877`; both carry the original string as their base code.

### F-13 — Decoding a serialised trade item uniform resource identifier

**When** `urn:epc:id:sgtin:9521141.012345.4711` is parsed.
**Then** a `product` record with value `09521141123454` and a `lot` record with value `4711`.

### F-14 — Decoding a ninety-six-bit serialised trade item drops the filter

**When** `urn:epc:tag:sgtin-96 : 1.358378.0728089.620776` is parsed.
**Then** the leading `1` is dropped as a filter; a `product` record with value `03583787280898` and
a `lot` record with value `620776` are produced.

### F-15 — Decoding a logistic unit uniform resource identifier

**When** `urn:epc:id:sscc:952656789012.03456` is parsed.
**Then** one `package` record with value `095265678901234568`.

### F-16 — The shipped default nomenclature may not be deleted

**When** a system administrator deletes the shipped Default Nomenclature.
**Then** the deletion is refused with "You cannot delete 'Default Nomenclature' because it's the
default barcode nomenclature."

---

## G. Barcodes — the Global Standards One nomenclature

### G-1 — A four-element label

**Given** the shipped Global Standards One nomenclature; the current year is 2026.
**When** the string `<GS>01940190976854571033650100138<GS>310200200415131018` is decomposed, where
`<GS>` is the non-printing group-separator character.
**Then** four records are produced, in this order:

| Order | Application identifier | Type | Value |
|---|---|---|---|
| 1 | `01` | `product` | `94019097685457` |
| 2 | `10` | `lot` | `33650100138` |
| 3 | `3102` | `quantity` | 20.04 |
| 4 | `15` | `use_date` | 18 October 2013 |

### G-2 — Product, batch and expiry with a separator

**Given** the shipped Global Standards One nomenclature; the current year is 2026.
**When** the string `<GS>019401909768545710LOT-A42<GS>17261231` is decomposed.
**Then** three records are produced:

| Order | Application identifier | Type | Value |
|---|---|---|---|
| 1 | `01` | `product` | `94019097685457` |
| 2 | `10` | `lot` | `LOT-A42` |
| 3 | `17` | `expiration_date` | 31 December 2026 |

### G-3 — The same data without the separator is misread

**Given** the same nomenclature.
**When** the string `019401909768545710LOT-A4217261231` is decomposed.
**Then** two records are produced: the trade item number, and a `lot` record whose value is
`LOT-A4217261231` — the greedy alphanumeric field swallows the expiry. No error is raised. This is
the expected behaviour and an implementation must reproduce it.

### G-4 — Placing the fixed-length field first removes the need for a separator

**Given** the same nomenclature.
**When** the string `019401909768545717261231` followed by `10LOT-A42` is decomposed — that is,
`01940190976854571726123110LOT-A42`.
**Then** three records are produced: the trade item number, an `expiration_date` of 31 December
2026, and a `lot` of `LOT-A42`.

### G-5 — A bad check digit skips the rule rather than failing

**Given** the shipped nomenclature.
**When** a string begins with `01` followed by fourteen digits whose last digit is not the correct
check digit.
**Then** the trade item rule produces nothing, the loop tries the following rules at the same
position, and — finding none that matches — abandons the whole decomposition and returns nothing.

### G-6 — A partially decomposable string returns nothing

**Given** the shipped nomenclature.
**When** the string `0194019097685457XYZ` is decomposed.
**Then** the first record would be the trade item number, but `XYZ` matches no rule, so the whole
decomposition returns nothing. There is no partial result.

### G-7 — The decimal position

**Given** a Global Standards One nomenclature with a measure rule of pattern `(300)(\d{5,8})` and
decimal usage off, and a measure rule of pattern `(304)(\d{5,8})` and decimal usage on.

| Scan | Value string | Value |
|---|---|---|
| `30000000` | `00000` | 0 |
| `30018789` | `18789` | 18789 |
| `3001515000` | `1515000` | 1515000 |
| `30400000` | `00000` | 0.0 |
| `30418789` | `18789` | 1.8789 |
| `3041515000` | `1515000` | 151.5 |

### G-8 — A decimal rule whose application identifier has no digit fails loudly

**Given** the decimal rule of G-7 with its pattern changed to `()(\d{0,4})`, so the first group
captures nothing.
**When** `1234` is decomposed.
**Then** the parse raises

> There is something wrong with the barcode rule "GS1 Rule Test - Four Decimals" pattern.
> If this rule uses decimal, check it can't get sometime else than a digit as last char for the
> Application Identifier.
> Check also the possible matched values can only be digits, otherwise the value can't be casted as
> a measure.

### G-9 — A measure rule that matches non-digits fails loudly

**Given** the non-decimal rule of G-7 with its pattern changed to `(300)(.*)`.
**When** `300bilou4000` is decomposed.
**Then** the same class of validation error is raised, naming that rule.

### G-10 — Date conversion

**Given** the current year is 2026.

| Six digits | Result |
|---|---|
| `151020` | 20 October 2015 |
| `520300` | 31 March 2052 (day digits `00` → last day of the month) |
| `200200` | 29 February 2020 (leap year) |
| `800101` | 1 January 1980 (difference 54, so the previous century) |
| `700101` | 1 January 2070 (difference 44, so the current century) |
| `410551` | refused: "A GS1 barcode nomenclature pattern was matched. However, the barcode failed to be converted to a valid date: …" |

### G-11 — A date field of the wrong length skips the rule

**Given** a date rule whose value pattern can capture fewer than six digits.
**When** it matches five digits.
**Then** the rule produces nothing and the loop tries the next rule.

### G-12 — Symbology identifiers are stripped

**Given** the shipped nomenclature.
**When** the string `]C10194019097685457` is decomposed.
**Then** the prefix `]C1` is removed and one `product` record with value `94019097685457` is
produced. The same holds for the prefixes `]e0`, `]d2`, `]Q3`, `]J1` and the non-printing
group-separator character. Only the **first** matching prefix is removed, and only once.

### G-13 — Search unpadding

**Given** the acting company's nomenclature is the shipped Global Standards One nomenclature.
**When** a barcode search is made with the condition "barcode equals `0194019097685457`" and the
caller declares interest in the `product` type.
**Then** the condition is rewritten to "barcode **contains** `94019097685457`".

### G-14 — Search unpadding of a lot keeps the original comparison

**Given** the same nomenclature, and a search whose parsed first interesting record has type `lot`.
**When** the rewrite runs.
**Then** the condition compares the field to the lot value with the **original** comparison, not
with a contains comparison, and no leading zeros are stripped.

### G-15 — Unpadding an unparseable value

**Given** the same nomenclature.
**When** the condition is "barcode equals `0000012345670`" and the value parses to nothing.
**Then** the condition becomes "barcode contains `12345670`".
**And when** the condition is "barcode equals `12345670`" and the value parses to nothing, the
condition is left unchanged, because there is no leading zero to strip.

### G-16 — Membership conditions are expanded

**Given** the same nomenclature.
**When** the condition is "barcode is one of [`0194019097685457`, `0209521141123454`]".
**Then** it is rewritten as the disjunction of the two individually rewritten equality conditions.
**And when** the original comparison was "is none of", the disjunction is negated.

### G-17 — An empty membership list is untouched

**When** the condition is "barcode is one of []".
**Then** it is left exactly as it is.

### G-18 — The uniqueness check suppresses the rewrite

**Given** the same nomenclature.
**When** the barcode uniqueness validation runs.
**Then** it searches with the suppression flag set, so the search is exact and a product whose
barcode is a substring of another product's barcode is not reported as a duplicate.

---

## H. Barcode uniqueness

### H-1 — Two products may not share a barcode in one company

**Given** a product A of Main Company with barcode `5410011234567`.
**When** a product B of Main Company is saved with the same barcode.
**Then** the save is refused with

> Barcode(s) already assigned:
> (blank line)
> - Barcode "5410011234567" already assigned to product(s): *the display names of A and B*

### H-2 — Products of different companies may share a barcode

**Given** a product A of Company One with barcode `5410011234567` and a product B of Company Two.
**When** B is saved with the same barcode.
**Then** the save succeeds, because the check is grouped by company and a company's group only
searches products of that company or of no company.

### H-3 — A shared product collides with every company

**Given** a product A with **no** company and barcode `5410011234567`.
**When** a product B of Company One is saved with the same barcode.
**Then** the save is refused, because Company One's search includes products with no company.

### H-4 — Inaccessible duplicates are noted

**Given** a duplicate barcode on a product the acting user may not read.
**When** the validation reports.
**Then** the inaccessible product's name is omitted from the list and the message ends with

> (blank line)
> Note: products that you don't have access to will not be shown above.

### H-5 — A product barcode may not collide with a packaging barcode

**Given** a packaging barcode `5410011234567`.
**When** a product is saved with that barcode.
**Then** the save is refused with "A packaging already uses the barcode".

### H-6 — A packaging barcode may not collide with a product barcode

**Given** a product with barcode `5410011234567`.
**When** a packaging barcode with that value is saved.
**Then** the save is refused with "A product already uses the barcode".

### H-7 — Two packaging barcodes may not share a value, even across companies

**Given** a packaging barcode `5410011234567` of Company One.
**When** a packaging barcode with the same value is created for Company Two.
**Then** the save is refused by the database constraint with "A barcode can only be assigned to one
packaging."

### H-8 — Duplicating a product does not duplicate its barcode

**Given** a product with barcode `5410011234567`.
**When** it is duplicated.
**Then** the copy's barcode is empty and no uniqueness error is raised; the copy's name is the
original's followed by " (copy)".

---

## I. Attributes and values — guards

### I-1 — The variant-creation mode is frozen once used

**Given** the attribute **Size** used on the active template "Office Chair".
**When** its variant-creation mode is changed from `always` to `dynamic`.
**Then** the change is refused with "You cannot change the Variants Creation Mode of the attribute
Size because it is used on the following products:\nOffice Chair".

### I-2 — The mode may be changed when only archived templates use it

**Given** the attribute **Size** used only on archived templates.
**When** its mode is changed.
**Then** the change succeeds, because the guard counts only attribute lines on **active** templates.

### I-3 — An attribute in use may not be deleted

**Given** the same as I-1.
**When** the attribute is deleted.
**Then** the deletion is refused with "You cannot delete the attribute Size because it is used on
the following products:\nOffice Chair".

### I-4 — An attribute in use may not be archived

**Given** the same as I-1.
**When** the attribute is archived.
**Then** the archival is refused with "You cannot archive this attribute as there are still products
linked to it".

### I-5 — A multi-checkbox attribute may not create variants

**When** an attribute is saved with display type `multi` and variant creation `always`.
**Then** the save is refused by the database check with "Multi-checkbox display type is not
compatible with the creation of variants".

### I-6 — Choosing multi-checkbox on an unused attribute sets the mode

**Given** an attribute with variant creation `always` and no related products.
**When** its display type is changed to `multi` on the form.
**Then** the variant-creation mode becomes `no_variant` automatically, and the save succeeds.

### I-7 — A value's attribute is frozen once the value is used

**Given** the value **L** of the attribute **Size**, offered by an active template.
**When** its attribute is changed.
**Then** the change is refused with "You cannot change the attribute of the value Size: L because it
is used on the following products: Office Chair".

### I-8 — A value used on active products may not be deleted

**Given** the same.
**When** the value is deleted.
**Then** the deletion is refused with "You cannot delete the value Size: L because it is used on the
following products:\nOffice Chair\n".

### I-9 — A value used only on archived variants is archived

**Given** the value **L**, whose materialised Template Attribute Values are linked only to archived
variants, and no active template offers it.
**When** the value is deleted.
**Then** the value is **archived** instead; its archived variants keep resolving.

### I-10 — A value's display name includes the attribute

**Given** the value **L** of attribute **Size**.
**Then** its display name is "Size: L" by default, and "L" when the caller sets the
show-attribute context flag to false.

### I-11 — Selecting a never-create-variant attribute pre-selects all its values

**Given** an attribute **Engraving** with variant creation `no_variant` offering three values.
**When** a new attribute line is created and that attribute is chosen on the form.
**Then** all three values are pre-selected.

### I-12 — Selecting any other attribute filters the existing selection

**Given** a new attribute line with values of attribute **Size** already selected.
**When** the attribute is changed to **Colour**.
**Then** the value selection becomes empty, because no selected value belongs to Colour.

---

## J. Template attribute lines and values

### J-1 — An active line must offer a value

**When** a line is saved active with no value.
**Then** the save is refused with "The attribute Size must have at least one value for the product
Office Chair."

### J-2 — A value must belong to the line's attribute

**When** a line for attribute **Size** is saved offering the value **Black** of attribute
**Colour**.
**Then** the save is refused with "On the product Office Chair you cannot associate the value
Colour: Black with the attribute Size because they do not match."

### J-3 — A line may not be moved to another template

**When** a line's template is written to another template's identifier.
**Then** the write is refused with "You cannot move the attribute Size from the product Office Chair
to the product *the target identifier*."

### J-4 — A line's attribute may not be changed

**When** a line's attribute is written to another attribute's identifier.
**Then** the write is refused with "On the product Office Chair you cannot transform the attribute
Size into the attribute *the target identifier*."

### J-5 — Removing an attribute and adding it back restores the variants

**Given** the "Office Chair" of A-3 with a confirmed sales order line referencing "Office Chair (L,
Black)".
**When** the manager deletes the Colour attribute line, and then adds a Colour line again offering
Black and White.
**Then** the deletion archives the line and its materialised values (because the order line blocks
the deletion); the addition revives the same archived line, reactivates the same materialised
values and therefore reactivates the same variants, so "Office Chair (L, Black)" is the same record
as before and the order line still points at it.

### J-6 — Archiving a line clears its values

**When** a line is archived.
**Then** its value list becomes empty in the same write, so the "an active line must offer a value"
validation is not tripped; value materialisation then archives or deletes the line's materialised
values.

### J-7 — A value may be moved between lines of the same attribute

**Given** an archived Template Attribute Value for (template T, attribute A, value V) attached to
line L1, and a new line L2 for the same template and attribute offering V.
**When** value materialisation runs for L2.
**Then** the archived record is reactivated and **re-attached to L2** rather than a new record being
created.

### J-8 — Each value appears once per line

**When** two Template Attribute Values are created for the same line and the same underlying value.
**Then** the second is refused with "Each value should be defined only once per attribute per
product."

### J-9 — The variant link may not be written from the value side

**When** a Template Attribute Value is written with a related-variant list.
**Then** the write is refused with "You cannot update related variants from the values. Please
update related values from the variants."

### J-10 — The underlying value and the template are frozen

**When** a Template Attribute Value's underlying attribute value is written to a different one,
**then** "You cannot change the value of the value Size: L set on product Office Chair."
**When** its template is written to a different one, **then** "You cannot change the product of the
value Size: L set on product Office Chair."

### J-11 — Deleting a single-value line's value does not delete the variants

**Given** the "Office Chair" of A-8: six variants each carrying *Material: Oak*, the Material line
offering only Oak.
**When** the Material line's materialised value is deleted.
**Then** *Material: Oak* is removed from each of the six variants directly, and the six variants
survive with one fewer attribute value.

---

## K. Naming and display

### K-1 — The template display name with an internal reference

**Given** a template named "Office Chair" with internal reference `FURN-001`.
**Then** its display name is `[FURN-001] Office Chair`; with the two-column formatting context it is
`Office Chair` followed by a tab and `--FURN-001--`; with the internal-reference display switched
off it is `Office Chair`.

### K-2 — The variant display name with a multi-value combination

**Given** "Office Chair" with Size {S, M, L} and Colour {Black, White}.
**Then** the variant for {S, Black} displays as "Office Chair (S, Black)".

### K-3 — Single-value lines are hidden from the name

**Given** the state after A-8 (a single-value Material line).
**Then** the same variant still displays as "Office Chair (S, Black)".

### K-4 — No-variant values are hidden from the name

**Given** a template with a `no_variant` attribute whose value is somehow stored on a variant.
**Then** that value does not appear in the variant's display name.

### K-5 — An archived variant keeps its historical name

**Given** a variant whose combination contains an archived value, on a line that now offers three
values.
**Then** the single-value filter counts archived values too, so the name is computed as it was when
the variant existed — an archived value from a then-single-value line is still hidden.

### K-6 — The vendor-specific display name

**Given** a variant "Office Chair (S, Black)" with internal reference `CH-01`, and a vendor line for
partner **Acme** with vendor product name "Acme Chair" and vendor product code `AC-77`.
**When** the display name is read with Acme in the partner context.
**Then** it is `[AC-77] Acme Chair (S, Black)`.

### K-7 — A variant-specific vendor line wins

**Given** the same, plus a second Acme vendor line naming this exact variant, with vendor product
code `AC-77-S-BK` and no vendor product name.
**Then** the reference resolves to `AC-77-S-BK` and the display name is
`[AC-77-S-BK] Office Chair (S, Black)`.

### K-8 — Several vendor lines produce a joined name

**Given** two vendor lines for the same partner, neither naming a variant, with different vendor
product names.
**Then** the display name is the comma-and-space-joined list of the distinct per-line names.

### K-9 — The category complete name

**Given** categories "All" (no parent), "Saleable" (parent All) and "Office Furniture" (parent
Saleable).
**Then** the complete names are `All`, `All / Saleable` and `All / Saleable / Office Furniture`.
**When** "Saleable" is renamed "Sellable", **then** the complete names become `All`, `All / Sellable`
and `All / Sellable / Office Furniture`.

### K-10 — Category cycles are refused

**When** "All" is given "Office Furniture" as its parent.
**Then** the save is refused with "You cannot create recursive categories."

### K-11 — Staged name search

**Given** a variant with internal reference `CH-01`, name "Office Chair (S, Black)" and barcode
`5410011234567`; and twenty other variants whose names contain "Chair".
**When** a name search is run for `CH-01`, **then** exactly that one variant is returned, because
the exact-internal-reference stage matched.
**When** a name search is run for `5410011234567`, **then** exactly that one variant is returned,
because the exact-barcode stage matched.
**When** a name search is run for `Chair`, **then** the internal-reference stage returns nothing and
the name stage returns the twenty-one matching variants.
**When** a name search is run for `[CH-01] Office Chair`, **then** the earlier stages return nothing
and the bracketed-code stage returns the one variant.

### K-12 — The template name search covers the variants

**Given** a template "Office Chair" whose variant has internal reference `CH-01`.
**When** templates are searched by display name for `CH-01` with a positive comparison.
**Then** the template is returned, because the search unions the direct matches with the
variant matches.
**When** the same search is made with a negative comparison, **then** the template is **not**
returned, because the negative search intersects the two conditions.

---

## L. Images

### L-1 — A variant with no image falls back to the template

**Given** a template with an image and a variant with none.
**Then** every one of the variant's five image fields reads the template's corresponding derivative,
and the variant's zoomability flag reads the template's.

### L-2 — A variant with its own image does not fall back

**Given** a template with image T and a variant with image V.
**Then** the variant's image fields read V's derivatives and its zoomability flag is its own.

### L-3 — Writing an image on the only variant writes it on the template

**Given** a template with exactly one active variant and no image.
**When** an image is written on the variant's fallback image field.
**Then** the variant's own image field stays empty and the template's image is set.

### L-4 — Writing an image on one of several variants writes it on the variant

**Given** a template with three active variants and an image already on the template.
**When** an image is written on one variant's fallback image field.
**Then** that variant's own image field is set and the template's image is unchanged.

### L-5 — Clearing an already-empty image clears the template

**Given** a template with an image and a variant with none, among several variants.
**When** the variant's fallback image field is cleared.
**Then** the first branch applies — the template's image is being set to the variant's empty value —
so the **template's** image is cleared.

### L-6 — Deleting the last variant moves its image up

**Given** a template with no image and one variant carrying an image.
**When** that variant is deleted.
**Then** the image is written onto the template before the deletion. (In practice the template is
then deleted too, unless it is dynamic.)

### L-7 — Changing the template image invalidates the variants' caches

**Given** a template with three variants, none of which has its own image.
**When** the template's original image is written.
**Then** the five image fields and the zoomability flag are invalidated on every variant, and each
variant's last-write moment becomes at least the template's, so a browser cache holding a variant
image expires.

### L-8 — Zoomability

**Given** a template whose original image is 1600 by 900 pixels.
**Then** the one-thousand-and-twenty-four-pixel derivative is 1024 by 576, which is strictly
smaller, so the zoomability flag is true.
**Given** instead an original of 800 by 600.
**Then** the derivative is also 800 by 600, so the flag is false.

---

## M. Documents

### M-1 — Uploading through the route

**Given** a product manager and a template with identifier 42.
**When** two files are posted to the upload route with owner model `product.template` and owner
identifier 42.
**Then** two documents are created, each with the file's name, the template as owner, the template's
company and the browser-declared content type; the response carries "All files uploaded".

### M-2 — An invalid owner model is rejected

**When** the upload route is called with owner model `res.partner`.
**Then** the response is empty and nothing is created.

### M-3 — A caller without write access is rejected

**Given** a user with read-only access on products.
**When** the upload route is called.
**Then** the response is empty and nothing is created.

### M-4 — One failing file does not stop the others

**Given** three files, the second of which fails to store.
**Then** documents are created for the first and the third; the response carries the failure text of
the second under the error key.

### M-5 — A file in the message thread becomes a document

**When** a user attaches a file to a message on a product variant.
**Then** an attachment is created with that variant as owner and no field name, and a document is
created for it automatically.

### M-6 — A bad web address is refused

**When** a document of kind "web address" is given the address `www.example.com`.
**Then** the form refuses it with

> Please enter a valid URL.
> Example: *the constant example address*
> (blank line)
> Invalid URL: www.example.com

### M-7 — Deleting a document deletes its attachment

**When** a document is deleted.
**Then** its underlying attachment is deleted too.

### M-8 — The template's document count includes the variants'

**Given** a template with two documents of its own and a variant with one.
**Then** the template's document count is 3 and the variant's is 1.

---

## N. Labels

### N-1 — Printing labels for a service is refused

**When** the label action is run on a selection containing a product of type `service`.
**Then** it is refused with "Labels cannot be printed for products of service type".

### N-2 — A non-positive copy count is refused

**When** the dialog is confirmed with a copy count of 0.
**Then** it is refused with "You need to set a positive quantity."

### N-3 — An empty selection is refused

**When** the dialog is confirmed with neither templates nor variants filled.
**Then** it is refused with "No product to print, if the product is archived please unarchive it
before printing its label."

### N-4 — Grid dimensions and page counts

| Format | Columns | Rows | Labels per page | Labels printed | Pages |
|---|---|---|---|---|---|
| `dymo` | 1 | 1 | 1 | 5 | 5 |
| `2x7xprice` | 2 | 7 | 14 | 14 | 1 |
| `2x7xprice` | 2 | 7 | 14 | 15 | 2 |
| `4x12xprice` | 4 | 12 | 48 | 47 | 1 |
| `4x12xprice` | 4 | 12 | 48 | 48 | 1 |
| `4x12xprice` | 4 | 12 | 48 | 49 | 2 |
| `4x12xprice` | 4 | 12 | 48 | 60 | 2 |

### N-5 — The no-price document is chosen for the no-price format

**When** the format is `4x12`.
**Then** the printable document named for four columns, twelve rows and no price is used.
**When** the format is `4x12xprice`, **then** the price-bearing document is used.

---

## O. Expiry

### O-1 — Receiving creates the four dates

**Given** a product "Fresh Yoghurt 500 g" with the expiry flag set, expiration day count 30,
best-before day count 5, removal day count 3 and alert day count 10.
**When** a lot is created on 11 September 2026 at 09:30.
**Then** its expiration date is 11 October 2026 09:30; its best-before date is 6 October 2026 09:30;
its removal date is 8 October 2026 09:30; its alert date is 1 October 2026 09:30.

### O-2 — Postponing the expiration shifts the other three

**Given** the state after O-1, with the removal date manually set to 7 October 2026 09:30.
**When** the expiration date is written to 21 October 2026 09:30.
**Then** the shift is 10 days: the best-before date becomes 16 October 2026 09:30, the removal date
becomes 17 October 2026 09:30 (preserving the manual one-day tightening), and the alert date becomes
11 October 2026 09:30.

### O-3 — An empty derived date stays empty under a shift

**Given** a lot whose alert date was manually cleared.
**When** the expiration date is postponed.
**Then** the alert date stays empty.

### O-4 — Clearing the product flag clears the derived dates

**Given** the lot of O-1.
**When** the product's use-expiration-date flag is cleared.
**Then** the lot's best-before, removal and alert dates are cleared. (The expiration date itself is
cleared only when its own computation re-runs with the flag off.)

### O-5 — Turning off tracking turns off expiry

**When** a product's tracking mode is written to `none`.
**Then** its use-expiration-date flag is written to false in the same operation.

### O-6 — The expiry alert flag

**Given** the lot of O-1 with its original dates.
**Then** at 11 October 2026 09:29 the expiry alert flag is false; at 11 October 2026 09:30 it is
true.

### O-7 — Expired stock is not available

**Given** the lot of O-1 with a stock quantity record of 24 units, none reserved.
**Then** at 8 October 2026 09:29 the available quantity is 24; at 8 October 2026 09:30 — the removal
date — it is 0.

### O-8 — The reminder fires once

**Given** the lot of O-1 with 24 units in an internal location, and today's date is 1 October 2026.
**When** the scheduled reminder runs.
**Then** a to-do activity is created on the lot with the summary "Alert Date Reached" and the note
"The alert date has been reached for this lot/serial number", assigned to the product's responsible
user for the lot's company, or the product's responsible user, or the superuser; the lot's reminded
flag becomes true.
**When** the reminder runs again the next day, **then** no second activity is created.
**When** the alert date is moved back to 30 September 2026 and the reminder runs, **then** still no
second activity is created.

### O-9 — A lot with no internal stock is marked reminded without an activity

**Given** a lot whose alert date has passed and which has no stock in an internal location.
**When** the reminder runs.
**Then** no activity is created **and** the lot's reminded flag is set to true.

### O-10 — Delivering an expired lot opens the dialog

**Given** an outgoing transfer with one move line whose lot's expiry alert flag is true.
**When** the operator asks to complete it.
**Then** the completion does not happen; the confirmation dialog opens with the description

> You are going to deliver the product Fresh Yoghurt 500 g, YOG-2609-A which is expired or should at
> least be removed from stock.
> Do you confirm you want to proceed?

and the lot list hidden, because only one lot is listed.

### O-11 — Several expired lots give the generic message

**Given** two offending lots.
**Then** the description is

> You are going to deliver some product expired lots.
> Do you confirm you want to proceed?

and the lot list is shown.

### O-12 — Confirming proceeds

**When** the operator confirms.
**Then** the completion runs again with the expiry check suppressed and the transfer completes.

### O-13 — Discarding removes the expired lines

**Given** a transfer with three move lines, two of which use expiration dates and have a removal
date strictly before the current moment.
**When** the operator chooses to discard the expired products.
**Then** those two move lines are deleted and the transfer completes with the remaining line.

### O-14 — A line whose removal date is exactly now triggers the dialog but is not discarded

**Given** a move line whose removal date equals the current moment exactly.
**Then** the pre-completion check fires (its comparison is "at or before"), but the discard action
does not remove the line (its comparison is "strictly before").

### O-15 — Move-line date defaults

**Given** the product of O-1 and an incoming transfer whose scheduled moment is 20 September 2026
08:00, on an operation type that creates lots.
**Then** a move line with no lot gets an expiration date of 20 October 2026 08:00 and a removal date
of 17 October 2026 08:00.
**And when** the line is given a lot with an expiration date of 25 October 2026 12:00, the line's
expiration date becomes 25 October 2026 12:00 and, if that lot has a removal date, the line's
removal date becomes the lot's.

### O-16 — First expiry first out ordering

**Given** three stock quantity records of the same product with removal dates 5 October, 3 October
and 7 October 2026.
**When** the removal strategy is first expiry first out.
**Then** they are consumed in the order 3 October, 5 October, 7 October; ties are broken by entry
date, then by identifier.

### O-17 — The forecast "to remove" figure

**Given** a product with 100 on hand, 20 incoming, 30 outgoing and a forecast of 80.
**Then** the "to remove" figure is 100 + 20 − 30 − 80 = 10.

---

## P. Import and export

### P-1 — A combined import creates templates, attributes, values and variants

**Given** an import file for the template entity with the columns Name, Sales Price and Product
Values, and these rows:

| Name | Sales Price | Product Values |
|---|---|---|
| Desk Pad | 12.00 | *(empty)* |
| T-Shirt | 20.00 | Size:S,Colour:Black |
| T-Shirt | 20.00 | Size:M,Colour:Black |
| T-Shirt | 20.00 | Size:S,Colour:White |

**When** the import runs and neither the Size nor the Colour attribute exists.
**Then**: the first row is imported as a plain template with one variant; the attributes **Size**
and **Colour** are created with variant creation `dynamic` and display type `radio`; the values
**S**, **M**, **Black** and **White** are created; one template "T-Shirt" is created with sales
price 20.00; the placeholder variant it received on creation is removed; two attribute lines are
created, Size offering S and M and Colour offering Black and White; three variants are created for
the three listed combinations, and the fourth combination (M, White) does **not** exist.

### P-2 — Empty cells are filled from the template's first variant

**Given** the same import with a Cost column filled only on the first T-Shirt row, with the value
9.00.
**Then** the other two T-Shirt variants take the cost captured from the template's first variant at
the moment the template was created, which is the default cost, not 9.00. (The fill uses the
**captured defaults**, not the preceding row.)

### P-3 — A row with product values but no name is refused

**When** a row has product values and neither a name nor a template reference.
**Then** the import raises "Unable to import products with attribute values but without name of
product set".

### P-4 — A product value with no attribute part is refused

**When** a cell reads `Large`.
**Then** the import raises "Unable to import products with attribute value without attribute name
(defined as: attribute:value): Large".

### P-5 — The same attribute twice in one row is refused

**When** a cell reads `Size:S,Size:M`.
**Then** the import raises "It is not possible to import different values for the same attribute:
Size:S,Size:M".

### P-6 — The same pair twice in one row is refused

**When** a cell reads `Size:S,Size:S`.
**Then** the import raises "Duplicate values in attribute values are not allowed: Size:S,Size:S".

### P-7 — Re-importing an existing variant with a different combination is refused

**Given** a variant with an external identifier and combination `Colour:Black,Size:S`.
**When** an import row with that external identifier carries `Colour:White,Size:S`.
**Then** the import raises

> The exitings product has different attribute value. "Colour:White,Size:S" is not equivalent to
> "Colour:Black,Size:S" for "*the external identifier*", "*the internal identifier*"

### P-8 — Re-importing the same combination asserts without blanking

**Given** the same variant.
**When** an import row with that external identifier carries `Colour:Black,Size:S` and leaves every
other column empty.
**Then** nothing is changed: the empty columns are dropped together with the product-values column.

### P-9 — Exporting a combination round-trips

**Given** the variant of P-7.
**When** its product-values field is exported.
**Then** the value is `Colour:Black,Size:S` — the pairs joined by commas and sorted alphabetically —
which is exactly what the import accepts.

### P-10 — A direct variant import with product values is redirected

**When** an import is run against the variant entity with a product-values column and without the
from-template flag.
**Then** it is redirected to the template entity, and the returned identifiers are the variants of
the resulting templates.

---

## Q. The catalog contract

### Q-1 — The default product filter

**Given** a document of Main Company implementing the catalog contract.
**When** the catalog action is opened.
**Then** the filter is: the product's company is empty, or is a parent of Main Company; **and** the
product's type is not `combo`.

### Q-2 — Products already on the document report their line data

**Given** a document with a line for product P of quantity 3 and unit price 12.00.
**When** the order-lines-information route is called for the page containing P.
**Then** the payload for P carries quantity 3, the price 12.00, P's type, P's reference and a unit
display name — filled from P's own default unit when the line did not supply one.

### Q-3 — Products not on the document report the defaults

**Given** the same document and a product Q with no line.
**Then** the payload for Q carries quantity 0, the document's read-only verdict, Q's type, Q's unit
display name and Q's reference. A product that has a line is never overwritten by this pass.

### Q-4 — Typing a quantity calls the update route

**When** a user types 5 on Q's card.
**Then** the update route is called with the document's transport name, the document identifier, Q's
identifier and the quantity 5; the document creates or changes the line; the route returns the
resulting unit price.

### Q-5 — Both routes act as the document's company

**Given** a document of Company Two while the user's acting company is Company One.
**Then** both routes execute with Company Two as the acting company, so company-dependent values —
the cost in particular — are read for Company Two.

---

## R. The matrix

### R-1 — The grid shape

**Given** the "Office Chair" of A-4 with the exclusion of A-5, displayed in euro with extras shown.
**Then** the header is: a cell with "Office Chair", then cells "S", "M" and "L", the last carrying a
price of 5.00 in euro; there are two rows, headed "Black" (no price) and "White" (price 2.00 in
euro); the six cells carry the sorted value identifiers, a quantity of 0, and possibility flags that
are true everywhere except the White-and-L cell.

### R-2 — The enumeration order

**Given** value lists [101, 102, 103] for the first line and [104, 105] for the second.
**Then** the enumeration is `(101,104), (102,104), (103,104), (101,105), (102,105), (103,105)`, cut
into two rows of three.

### R-3 — A row header for several remaining lines

**Given** a template with three attribute lines, the second and third contributing to the rows.
**Then** the row header cell's name joins those values with a bullet separator surrounded by spaces,
and its price is the sum of their extra prices, converted.

### R-4 — A template with one attribute line

**Given** a template with exactly one attribute line offering three values.
**Then** there is one row whose header cell's name is a single space, so the client renders a blank
row header rather than treating it as unavailable.

### R-5 — Extras hidden

**When** the matrix is requested with extra prices hidden — as on a purchase document.
**Then** no header cell carries a price or a currency, even for values whose extra price is
non-zero.

---

## S. Multi-company and currency

### S-1 — A shared template uses the main company's currency

**Given** a template with no company, in an installation whose main company's currency is the euro,
while the acting company's currency is the United States dollar.
**Then** the template's currency is the euro and its **cost** currency is the United States dollar.

### S-2 — A company-scoped template uses its own company's currency

**Given** a template of Company Two whose currency is the pound sterling.
**Then** both its currency and its cost currency are the pound sterling, whatever the acting
company is.

### S-3 — The cost is per company

**Given** a variant whose cost is 7.00 for Company One and 6.20 for Company Two.
**When** it is read with Company One acting, **then** 7.00; with Company Two acting, **then** 6.20.

### S-4 — Writing the cost on a multi-variant template does nothing

**Given** a template with three variants.
**When** 8.00 is written to the template's cost.
**Then** no variant's cost changes, because the template's mirror falls back to zero and the
write-through only fires for a single-variant template.

### S-5 — The company record rule

**Given** Company One with child Company Two, and a template of Company One.
**When** a user whose acting companies are {Company Two} reads products.
**Then** the template **is** visible, because the rule is "the record's company is a parent of one of
the acting companies".
**And given** a template of Company Two, a user acting as {Company One} does **not** see it.

### S-6 — A new company gets the default nomenclature

**When** a company is created.
**Then** its nomenclature is the shipped Default Nomenclature.

### S-7 — Installing the barcode capability back-fills nomenclatures

**Given** three existing companies with no nomenclature.
**When** the barcode capability is installed.
**Then** all three are written with the shipped Default Nomenclature.

---

## T. Settings and parameters

### T-1 — The weight unit

**Given** the system parameter `product.weight_in_lbs` unset.
**Then** the weight unit label is the display name of the kilogram unit.
**When** the parameter is set to the string `1`, **then** it is the display name of the pound unit.
**When** it is set to the string `0` or to any other string, **then** it is the kilogram again.

### T-2 — One parameter governs volume and length

**When** `product.volume_in_cubic_feet` is set to `1`.
**Then** the volume unit becomes the cubic foot **and** the length unit becomes the foot.
**When** it is unset, **then** they are the cubic metre and the millimetre.

### T-3 — Turning off pricelists archives them

**Given** two active pricelists.
**When** the pricelist setting is turned off.
**Then** the form shows the warning "You are deactivating the pricelist feature. Every active
pricelist will be archived.", and on saving both pricelists are archived.

### T-4 — Turning on pricelists bootstraps a default per company

**Given** two companies, neither with a rule-free pricelist in its own currency.
**When** the pricelist setting is turned on.
**Then** one pricelist named "Default" is created per company, each with that company's currency and
sequence 10.

### T-5 — An existing rule-free default is revived rather than duplicated

**Given** a company with an archived pricelist that has no rules and whose currency equals the
company's currency.
**When** the setting is turned on.
**Then** that pricelist is unarchived and no new one is created.

### T-6 — Changing a company's currency creates the pricelist with the new currency

**Given** a company whose currency is being changed, and the pricelist group being granted in the
same operation.
**When** the write happens.
**Then** the bootstrap is suppressed during the write and re-run afterwards, so the created
pricelist carries the **new** currency.

### T-7 — Archiving a currency archives its pricelists

**When** a currency is archived.
**Then** every pricelist using it is archived.

---

## U. Access and visibility

### U-1 — An internal user may read but not change

**Given** a user holding only the internal-user group.
**When** they open a product.
**Then** they may read every field they have field-level access to; any attempt to create, change or
delete a template, a variant, a category, a tag, an attribute, a value, a line, a materialised
value, an exclusion, a combo, a combo item, a document or a packaging barcode is refused by the
access rights.

### U-2 — An internal user may use the label dialog

**Given** the same user.
**Then** they may create, read, change and delete Label Layout records, so the label dialog works
for them.

### U-3 — The cost is hidden from non-internal readers

**Given** a reader outside the internal-user group.
**Then** the cost field is not readable. Computations that need it — deriving a sales price from a
cost — re-read it as a privileged reader so that the price is still correct.

### U-4 — The bulk-update dialog cannot be deleted

**Given** a product manager.
**Then** they may create, read and change Attribute Value Bulk Update records but not delete them.

### U-5 — Nomenclatures are administered by the system administration group

**Given** a product manager who is not a system administrator.
**Then** they may read barcode nomenclatures and rules but not change them.

### U-6 — The variant group reveals the variant interface

**Given** a user without the variant group.
**Then** the attribute configuration, the variant list, the attribute-value tags on the catalog
cards and the attribute search fields are hidden. Installing the product matrix capability grants
the group to every internal user, making them visible.

---

## V. Cross-cutting invariants

### V-1 — Every active combination has at most one active variant

At all times, for every template, no two active variants share the same combination indices.

### V-2 — A template always has at least one variant unless it is dynamic or archived

For every active, non-dynamic template, the variant count is at least one.

### V-3 — The sum of prorated combo shares equals the combo price

For every combo line, the sum over its choice groups of the prorated share equals the combo
product's price exactly, in the document currency, and the combo product's own line unit price is
zero.

### V-4 — A barcode identifies at most one thing per company

Within one company, no barcode is carried by two products, and no barcode is carried by both a
product and a packaging binding. Globally, no barcode is carried by two packaging bindings.

### V-5 — A parse is total or nothing under the Global Standards One nomenclature

A Global Standards One decomposition either consumes the entire string or returns nothing.

### V-6 — Derived expiry dates keep their offsets under a postponement

For any lot, postponing the expiration date by *d* changes each non-empty derived date by exactly
*d*, so the offsets between the four dates are preserved.

### V-7 — The catalog posts nothing

No operation specified in this folder creates, changes, posts, reverses or reconciles a Journal
Entry or a Journal Item.
