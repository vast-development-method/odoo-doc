# Products and Catalog — Business Rules

Every validation, constraint, invariant, guard, permission check and edge-case behaviour of the
domain, with the exact user-facing message the system produces. Placeholders in messages are written
here in italics as the quantity they stand for; the surrounding text is reproduced verbatim,
including punctuation, capitalisation and embedded line breaks.

Validations are grouped by the entity they protect. Within each group they are listed in the order
in which they fire during a write.

**A note on quoted system messages.** Text shown inside a block quote or between quotation marks in
this file is the message the system itself emits, reproduced character for character so that an
implementation can match it. A few of those messages contain abbreviations the system prints:
`URL` for uniform resource locator, `FNC1` for the function code one separator, `GS1` for Global
Standards One, and `FEFO` for first expiry first out. They are reproduced because the text is a
contract; everywhere outside a quoted message this folder writes such terms in full.

---

## 1. How validations are classified

| Class | When it fires | What it does on failure |
|---|---|---|
| **Database constraint** | At the moment the row is written | Refuses the write and reports the constraint's message |
| **Record validation** | After the write, before the transaction is confirmed, on the records whose watched fields changed | Refuses the whole transaction and reports the message |
| **Write guard** | Inside the write path, before anything is stored | Refuses the whole operation and reports the message |
| **Deletion guard** | Inside the deletion path, before anything is removed | Refuses the whole operation and reports the message |
| **Interface warning** | While a user edits a form, before saving | Shows a dialog; does not block saving |
| **Silent correction** | Inside the write or computation path | Changes the value without telling the user |

An implementation must reproduce the class as well as the message: an interface warning that becomes
a blocking error changes the user's workflow, and a silent correction that becomes an error breaks
imports.

---

## 2. Product Template

### 2.1 A combo product must have at least one choice group

**Class** record validation, watching the type and the choice group list.
**Condition** the type is `combo` ("Combo") and the choice group list is empty.
**Message**

> A combo product must contain at least 1 combo choice.

### 2.2 A sellable combo product may only contain sellable products

**Class** record validation, watching the type, the choice group list and the sellable flag.
**Condition** the type is `combo`, the sellable flag is true, and at least one product reachable
through the choice groups' items has its own sellable flag false.
**Message**

> A sellable combo product can only contain sellable products.

### 2.3 Changing the type to combo is refused when attributes exist

**Class** interface warning, raised as a blocking error from the type field's change handler.
**Condition** the type is being changed to `combo` and the template has at least one attribute line.
**Message**

> Combo products can't have attributes.

### 2.4 Changing the type to combo is refused when the product is itself inside a combo

**Class** interface warning, raised as a blocking error from the type field's change handler.
**Condition** the type is being changed to `combo` and at least one combo item names a variant of
this template.
**Message**

> This product is part of a combo, so its type can't be changed to "combo".

### 2.5 Silent corrections attached to the type

- Changing the type to `combo` sets the purchasable flag to false.
- Writing any type other than `combo` empties the choice group list.
- A type other than `service` forces the create-on-order field back to `no` ("Nothing").

### 2.6 The cost may not be negative

**Class** interface warning, raised as a blocking error from the cost field's change handler on both
the template and the variant.
**Condition** the cost being typed is strictly less than zero.
**Message**

> The cost of a product can't be negative.

This is **not** a record validation: a negative cost written through an import or a remote call is
accepted.

### 2.7 A duplicate internal reference is warned about, not refused

**Class** interface warning.
**Condition** an internal reference is typed and at least one other record already has it. On a
template the search is over templates; on a variant it is over variants. When the record already
exists, it excludes itself.
**Message on a template** title "Note:", body

> The Internal Reference '*the typed reference*' already exists.

**Message on a variant** title "Note:", body

> The Reference '*the typed reference*' already exists.

Internal references are deliberately **not** unique.

### 2.8 Changing the unit of measure is warned about

**Class** interface warning, and only when the consuming domain says the product is in use (the base
catalog always says it is not, so the warning is silent unless an inventory or sales capability is
installed).
**Condition** the unit is being changed from a different one.
**Message** title "What to expect ?", body

> Changing the unit of measure for your product will apply a conversion 1 *the previous unit's
> display name* = 1 *the new unit's display name*.
> All existing records (Sales orders, Purchase orders, etc.) using this product will be updated by
> replacing the unit name.

The behaviour behind the warning is a **replacement, not a conversion**: existing documents keep
their numbers and change their unit label. The write path calls the variants' unit-replacement hook
with conversion suppressed before storing the new unit.

### 2.9 Labels cannot be printed for services

**Class** write guard on the label-printing action, on both the template and the variant.
**Condition** any selected record's type is `service`.
**Message**

> Labels cannot be printed for products of service type

### 2.10 The import-only field may not be written

**Class** write guard.
**Condition** the product-values field is written outside an import.
**Message** a programming-level error carrying the text "This field can only be used to import
products." (template) — the variant's counterpart raises the same kind of error.

### 2.11 Company scoping

- A template with an empty company is visible to every company.
- A template with a company is visible to that company and to every company below it in the company
  tree. The record rule is: the company is a parent of one of the acting companies, or the company
  is empty.
- Company consistency between a template and the records it points at is enforced automatically for
  every link declared as company-checked; the check uses the same parent-of relation.

---

## 3. Product Variant

### 3.1 Barcode uniqueness

**Class** record validation, watching the barcode, and additionally re-run on the template when the
template's company changes.

The check runs **within a company**. The records being checked are grouped by their company, and for
each group two searches are made with the Global Standards One search preprocessing suppressed:

**Against other products.** Search the products whose barcode is one of the group's barcodes and
whose company is either empty or the group's company (when the group has a company; when the group's
company is empty, no company restriction is applied). Group them by barcode and keep the barcodes
that occur more than once. For each such barcode, build one line:

> - Barcode "*the barcode*" already assigned to product(s): *the comma-separated display names of
>   the products the reader may read*

Join the lines with newlines. If any line was produced, append:

> (blank line)
> Note: products that you don't have access to will not be shown above.

and raise:

> Barcode(s) already assigned:
> (blank line)
> *the joined lines*

**Against packaging barcodes.** Search the packaging barcodes with the same barcode-and-company
filter. If at least one exists, raise:

> A packaging already uses the barcode

The mirror check on the packaging side is in section 8.2. Both exist because, under a Global
Standards One nomenclature, a product barcode and a packaging barcode share the same application
identifier and the same pattern, so a collision would make a scan ambiguous.

### 3.2 One active variant per combination

**Class** database constraint: a uniqueness index over the pair (template, combination indices)
restricted to rows whose activity flag is true.
**Effect** a template may never have two active variants with the same combination. Archived
variants are exempt, which is what makes archiving and later reactivating a combination possible.
**Message** the database's own duplicate-key message; the domain never triggers it in normal
operation, because variant generation and the create-on-demand procedure both look up the existing
variant first.

### 3.3 Company consistency of combo items

**Class** record validation on the variant, watching the company.
**Condition** a variant's company changes while combo items name it.
**Effect** the company consistency of every such combo item's product link is re-checked, producing
the framework's standard company-mismatch message naming the item and the field.

### 3.4 Duplicating a variant duplicates its template

**Class** silent behaviour.
Copying a variant is not possible as such: the template is copied instead and the copy's first
variant is returned, creating it when the copy is dynamic and has none. See
[entities.md](entities.md), section 2.6.

### 3.5 The import-only field on a variant

Two guards apply during an import that carries the product-values column.

**Reconciling with an existing variant.** When the import matches an existing variant and supplies
product values, the supplied set of `attribute:value` pairs must equal the variant's own set,
compared as unordered sets of trimmed pairs. Otherwise:

> The exitings product has different attribute value. "*the imported values*" is not equivalent to
> "*the existing values*" for "*the external identifier*", "*the internal identifier*"

(The misspelling of "existing" in the first word is reproduced exactly as the system emits it.)

When the sets do match, every empty column of the imported row is dropped, together with the
product-values column itself, so that an import that only asserts the combination does not blank
other fields.

**Creating new variants.** Before anything is created:

- a row with product values but no name and no template reference raises

  > Unable to import products with attribute values but without name of product set

- a product value with no attribute part raises

  > Unable to import products with attribute value without attribute name (defined as:
  > attribute:value): *the whole product-values cell*

- two product values naming the same attribute in one row raise

  > It is not possible to import different values for the same attribute: *the whole product-values
  > cell*

- the same attribute-and-value pair twice in one row raises

  > Duplicate values in attribute values are not allowed: *the whole product-values cell*

### 3.6 Deleting a variant may delete its template

Deleting the last remaining variant of a template — counting archived variants — deletes the
template as well, **unless** the template uses at least one dynamic attribute, in which case a
template with no variants is a legitimate state.

Before the deletion, a variant image is moved up to the template when the template has none, so
that the picture is not lost.

### 3.7 Archival cascades

| Operation | Effect |
|---|---|
| Archive a template | Archives every variant, archived ones included (a no-op for those) |
| Archive a variant | Archives the template when that template is active and now has no active variant |
| Unarchive a variant | Unarchives the template when that template is archived and now has an active variant |
| Unarchive a template that has no variants | Re-runs variant generation |

---

## 4. Product Attribute

### 4.1 Multi-checkbox attributes may not create variants

**Class** database constraint.
**Condition** the display type is `multi` ("Multi-checkbox") and the variant-creation mode is
anything other than `no_variant` ("Never").
**Message**

> Multi-checkbox display type is not compatible with the creation of variants

**Silent correction** choosing the multi-checkbox display type on an attribute with no related
products sets the variant-creation mode to `no_variant` automatically.

### 4.2 The variant-creation mode is frozen once the attribute is in use

**Class** write guard.
**Condition** the variant-creation mode is being changed to a different value on an attribute whose
count of related **active** products is non-zero.
**Message**

> You cannot change the Variants Creation Mode of the attribute *the attribute display name* because
> it is used on the following products:
> *the comma-separated display names of the related templates*

The rationale is that the change would silently invalidate every existing combination, and
recomputing them all could take arbitrarily long.

### 4.3 An attribute in use may not be deleted

**Class** deletion guard (skipped when the whole capability is being removed).
**Condition** the attribute's count of related active products is non-zero.
**Message**

> You cannot delete the attribute *the attribute display name* because it is used on the following
> products:
> *the comma-separated display names of the related templates*

### 4.4 An attribute in use may not be archived

**Class** write guard on the archiving action.
**Condition** the attribute's count of related active products is non-zero.
**Message**

> You cannot archive this attribute as there are still products linked to it

### 4.5 Resequencing invalidates caches

Writing a new sequence onto an attribute, when at least one record's sequence actually changes,
flushes every pending write and invalidates every cached collection. This is necessary because the
order of a template's attribute lines derives from the attributes' sequences, and a stale cached
collection would produce combinations in the wrong order and therefore wrong variant names.

---

## 5. Attribute Value

### 5.1 The attribute of a value in use may not be changed

**Class** write guard.
**Condition** the attribute is being changed on a value whose "used on products" flag is true — that
is, at least one attribute line offering it belongs to an active template.
**Message**

> You cannot change the attribute of the value *the value display name* because it is used on the
> following products: *the comma-separated display names of the templates*

### 5.2 A value in use may not be deleted

**Class** deletion guard (skipped when the whole capability is being removed).
**Condition** the value's "used on products" flag is true.
**Message**

> You cannot delete the value *the value display name* because it is used on the following products:
> *the newline-separated display names of the templates*
> (a trailing newline)

### 5.3 A value used only on archived variants is archived instead of deleted

**Class** silent correction inside the deletion path.
**Condition** for a value being deleted, gather every variant — archived ones included — reachable
through the materialised Template Attribute Values of that value. If **none** of them is active but
**at least one exists**, the value is archived rather than deleted. The remaining values proceed to
deletion normally.

### 5.4 Resequencing invalidates caches

As for the attribute, and for the same reason: the order of values inside a template attribute line
derives from the values' sequences.

---

## 6. Template Attribute Line

### 6.1 An active line must offer at least one value

**Class** record validation, watching the activity flag, the value list and the attribute.
**Condition** the line is active and its value list is empty.
**Message**

> The attribute *the attribute display name* must have at least one value for the product *the
> product display name*.

Archiving a line **clears** its value list as part of the same write, so archiving never trips this
rule: the activity flag becomes false in the same operation.

### 6.2 Every offered value must belong to the line's attribute

**Class** record validation.
**Condition** any offered value's attribute differs from the line's attribute.
**Message**

> On the product *the product display name* you cannot associate the value *the value display name*
> with the attribute *the attribute display name* because they do not match.

### 6.3 A line may not be moved to another template

**Class** write guard.
**Condition** a different template is written.
**Message**

> You cannot move the attribute *the attribute display name* from the product *the source product
> display name* to the product *the target product identifier*.

Note that the target appears as its raw identifier, not as a display name, because the guard runs
before the new value is resolved.

### 6.4 A line's attribute may not be changed

**Class** write guard.
**Condition** a different attribute is written.
**Message**

> On the product *the product display name* you cannot transform the attribute *the source attribute
> display name* into the attribute *the target attribute identifier*.

### 6.5 Creating a line reuses an archived one

**Class** silent behaviour. See [calculations.md](calculations.md), section 2, and
[entities.md](entities.md), section 7.3. The consequence a user sees is that removing an attribute
from a product and adding it back restores the previous variants rather than creating new ones —
provided the values are the same.

### 6.6 Deleting a line falls back to archiving

**Class** silent behaviour. The line's active materialised values are deleted first to release
references; then each line is deleted individually inside a savepoint, and any line whose deletion
fails is archived instead.

---

## 7. Template Attribute Value

### 7.1 One record per value per line

**Class** database constraint over (attribute line, attribute value).
**Message**

> Each value should be defined only once per attribute per product.

### 7.2 An active record's value must be offered by its line

**Class** record validation.
**Condition** the record is active and its underlying attribute value is not in its line's offered
value list.
**Message**

> The value *the value display name* is not defined for the attribute *the attribute display name*
> on the product *the product display name*.

### 7.3 The variant link may not be written from this side

**Class** write guard, on both creation and modification.
**Condition** the related-variant list appears in the values being written.
**Message**

> You cannot update related variants from the values. Please update related values from the
> variants.

The reason is mechanical: the combination index that identifies a variant is computed from the
variant's side of the link, and writing the link from this side would not trigger that computation.

### 7.4 The underlying value may not be changed

**Class** write guard.
**Message**

> You cannot change the value of the value *the record's display name* set on product *the product
> display name*.

### 7.5 The template may not be changed

**Class** write guard.
**Message**

> You cannot change the product of the value *the record's display name* set on product *the product
> display name*.

### 7.6 Writing the exclusion list regenerates variants

**Class** side effect. Any write that includes the exclusion list re-runs variant generation for the
templates involved.

### 7.7 Deletion falls back to archiving

**Class** silent behaviour. See [entities.md](entities.md), section 8.5. In particular, a value
belonging to a line that has exactly one materialised value is first **removed from its variants**,
which leaves the variants intact with one fewer attribute value rather than deleting them.

---

## 8. Packaging Barcode

### 8.1 A barcode identifies exactly one packaging

**Class** database constraint on the barcode alone — globally, not per company.
**Message**

> A barcode can only be assigned to one packaging.

### 8.2 A packaging barcode may not collide with a product barcode

**Class** record validation, watching the barcode.
**Condition** at least one product has the same barcode. The search is **not** company-restricted on
this side.
**Message**

> A product already uses the barcode

### 8.3 Required fields

The unit, the product and the barcode are all required; the barcode is not copied when the record is
duplicated, so duplicating a packaging barcode fails on the required-field check until a new barcode
is supplied.

---

## 9. Product Combo and Combo Item

### 9.1 A choice group must contain at least one item

**Class** record validation, watching the item list.
**Message**

> A combo choice must contain at least 1 product.

### 9.2 A choice group may not offer the same product twice

**Class** record validation, watching the item list.
**Condition** the number of distinct products among the items is smaller than the number of items.
**Message**

> A combo choice can't contain duplicate products.

### 9.3 A choice group may not contain combo products

**Class** record validation on the item, watching the product.
**Condition** the named variant's type is `combo`.
**Message**

> A combo choice can't contain products of type "combo".

The item's product link additionally restricts the selectable set to products whose type is not
`combo`, so the interface never offers one; the validation catches remote calls and imports.

### 9.4 Company consistency

**Class** record validation on the choice group, watching the company.
**Effect** the templates that reference this group must be company-compatible with it, and the
items' products must be company-compatible with it. Failure produces the framework's standard
company-mismatch message naming the record and the field.

### 9.5 Removing a variant removes the combo items naming it

**Class** side effect of variant generation. Every combo item that names a variant being unlinked is
deleted in the same pass. There is no warning.

---

## 10. Product Category

### 10.1 No cycles

**Class** record validation, watching the parent.
**Condition** the parent chain revisits the category.
**Message**

> You cannot create recursive categories.

### 10.2 Deleting a parent deletes its children

The parent link cascades on deletion, so deleting a category deletes the whole subtree. Templates
pointing at a deleted category are left with an empty category — the template's category link is not
required.

---

## 11. Product Tag

### 11.1 Tag names are globally unique

**Class** database constraint.
**Message**

> Tag name already exists!

Duplicating a tag therefore appends " (copy)" to the name automatically.

### 11.2 A variant may not be tagged directly with a tag its template already carries

**Class** interface restriction on the variant-tag link: the selectable set excludes tags already on
the template. There is no record validation, so a remote call may create the overlap; the merged tag
set is computed as a union, so the overlap is harmless.

---

## 12. Product Document

### 12.1 A web address must be well formed

**Class** interface warning, raised as a blocking error from the address field's change handler.
**Condition** the document's kind is a web address, an address is present, and it does not begin
with `https://`, `http://` or `ftp://`.
**Message** — four lines, the second of which carries a fixed example address that the system
supplies as a constant. In the reproduced text below, the abbreviation the system prints on the
first and last lines stands for "uniform resource locator", and *the example address* stands for
the constant the system substitutes there:

> Please enter a valid uniform resource locator.
> Example: *the example address*
> (blank line)
> Invalid uniform resource locator: *the entered address*

### 12.2 Upload route protections

The upload route accepts a file only when:

1. the owner model is `product.product` or `product.template` — any other model returns an empty
   response;
2. the named record exists;
3. the caller has write access on that model.

Each uploaded file is stored as a document whose name is the file name, whose owner is the named
record, whose company is the record's company, and whose content type is the one the browser
declared. A failure on one file is caught, recorded in the technical log and reported in the
response as an error text; the remaining files are still attempted. The successful response carries
the text "All files uploaded".

### 12.3 Automatic document creation

Any attachment created with an owner model of `product.product` or `product.template`, and which is
not the storage behind a specific field, automatically gets a document — unless the creation carries
the suppression flag. Creating a document itself sets that flag, so no loop occurs.

---

## 13. Barcode Nomenclature and Barcode Rule

### 13.1 The shipped default nomenclature may not be deleted

**Class** deletion guard (skipped when the whole capability is being removed).
**Message**

> You cannot delete '*the nomenclature display name*' because it's the default barcode nomenclature.

### 13.2 The separator expression must be a valid expression

**Class** record validation on the nomenclature, watching the separator.
**Condition** the nomenclature is a Global Standards One nomenclature, the separator is set, and
wrapping it in an optional non-capturing group does not compile.
**Message**

> The FNC1 Separator Alternative is not a valid Regex: *the compiler's message*

### 13.3 Global Standards One rule patterns need exactly two groups

**Class** record validation on the rule, watching the pattern, applied to rules whose encoding is
`gs1-128`.

First, the pattern must compile:

> The rule pattern '*the rule name*' is not a valid Regex: *the compiler's message*

Then, counting the parenthesised groups by finding every run from an opening parenthesis to the next
closing parenthesis, there must be exactly two:

> The rule pattern "*the rule name*" is not valid, it needs two groups:
> 	- A first one for the Application Identifier (usually 2 to 4 digits);
> 	- A second one to catch the value.

(The two continuation lines each begin with a tab character followed by a hyphen and a space.)

### 13.4 Classic rule patterns and the brace grammar

**Class** record validation on the rule, watching the pattern, applied to every rule whose encoding
is not `gs1-128`.

Compute the **reduced pattern** by replacing every occurrence of an escaped backslash, an escaped
opening brace and an escaped closing brace with the letter X, in that order. Then:

| Situation | Message |
|---|---|
| Exactly two braces, and no group matching an opening brace, zero or more N, zero or more D, a closing brace | There is a syntax error in the barcode pattern *the pattern*: braces can only contain N's followed by D's. |
| Exactly two braces, and an empty brace pair is present | There is a syntax error in the barcode pattern *the pattern*: empty braces. |
| Any other non-zero number of braces | There is a syntax error in the barcode pattern *the pattern*: a rule can only contain one pair of braces. |
| No braces, and the reduced pattern is exactly one asterisk | ` '*' is not a valid Regex Barcode Pattern. Did you mean '.*'?` (the message begins with a space) |
| After removing any group of an opening brace, one or more N, zero or more D and a closing brace, the remainder does not compile | The barcode pattern *the pattern* does not lead to a valid regular expression. |

**Worked cases.** With these rules:

| Pattern | Verdict |
|---|---|
| `........` | accepted — no braces |
| `{NNNNNNNN}` | accepted |
| `......{}..` | refused — empty braces |
| `......{DN}` | refused — braces can only contain N's followed by D's |
| `....{NN}{DD}` | refused — a rule can only contain one pair of braces |
| `*` | refused — not a valid pattern, did you mean `.*` |
| `**>>>{ND}` | refused — after removing the brace group, `**>>>` does not compile |
| `..>>>{ND}` | accepted — after removing the brace group, `..>>>` compiles |

Note that the braces-count test reaches the "braces can only contain N's followed by D's" branch
before the "empty braces" branch, so `......{}..` — which has exactly two braces and contains no
`{N*D*}` group other than the empty one — is caught by whichever branch the engine evaluates first;
in practice the `{N*D*}` search **does** match the empty pair, because both quantifiers accept zero
repetitions, so the empty-braces branch is the one that fires.

### 13.5 A pattern is matched against a prefix, not the whole barcode

The classic matcher anchors at the start and compares only the first *pattern-length* characters of
the base code. A pattern shorter than the barcode therefore matches a prefix. This is documented to
users on the nomenclature form:

> Barcodes Nomenclatures define how barcodes are recognized and categorized. When a barcode is
> scanned it is associated to the first rule with a matching pattern. The pattern syntax is that of
> regular expression, and a barcode is matched if the regular expression matches a prefix of the
> barcode.
>
> Patterns can also define how numerical values, such as weight or price, can be encoded into the
> barcode. They are indicated by {NNN} where the N's define where the number's digits are encoded.
> Floats are also supported with the decimals indicated with D's, such as {NNNDD}. In these cases,
> the barcode field on the associated records must show these digits as zeroes.

The last sentence is a real invariant an implementation must respect: the barcode stored on a product
matched by a value-carrying rule **must** have zeros in the value positions, because the parser
compares against the zeroed base code.

### 13.6 A measure rule with a badly configured application identifier fails loudly

**Class** record validation raised during parsing, not during configuration.
**Condition** a Global Standards One measure rule whose decimal-usage flag is set matches, but the
last character of the matched application identifier is not a digit, or the captured value is not
numeric.
**Message**

> There is something wrong with the barcode rule "*the rule name*" pattern.
> If this rule uses decimal, check it can't get sometime else than a digit as last char for the
> Application Identifier.
> Check also the possible matched values can only be digits, otherwise the value can't be casted as
> a measure.

### 13.7 A date that is not a real date fails loudly

**Class** record validation raised during parsing.
**Condition** a six-digit date whose day digits are not `00` does not form a valid calendar date.
**Message**

> A Global Standards One barcode nomenclature pattern was matched. However, the barcode failed to be converted to a
> valid date: '*the underlying parser's message*'

### 13.8 A partially decomposable Global Standards One barcode is rejected entirely

If, at any point in the decomposition loop, no rule matches the remaining string, or a matching rule
consumes nothing, the whole decomposition returns nothing. There is no partial result and no error
message: the caller sees "this is not a Global Standards One barcode" and falls back to whatever it
does with unrecognised scans.

### 13.9 A numeric identifier with a wrong check digit is not an error

When a Global Standards One rule of content type "numeric identifier" matches but the check digit
does not verify, the rule is simply skipped and the next rule is tried at the same position. This is
deliberate: the same digit prefix can legitimately belong to two different application identifiers.

---

## 14. Variant generation

### 14.1 The generation ceiling

**Class** write guard raised inside the generation algorithm.
**Condition** the number of variants queued for creation for **one template** exceeds the limit. The
limit is the system parameter `product.dynamic_variant_limit` read as a whole number, defaulting to
one thousand. The comparison is strictly greater, so exactly the limit is allowed.
**Message**

> The number of variants to generate is above allowed limit. You should either not generate variants
> for each combination or generate them on demand from the sales order. To do so, open the form view
> of attributes and change the mode of *Create Variants*.

### 14.2 A configuration that leaves no possible variant is refused

**Class** write guard raised at the end of the generation algorithm.
**Condition** after the unlink pass, at least one of the input templates no longer exists — which
happens when deleting its last variant cascaded into deleting the template.
**Message**

> This configuration of product attributes, values, and exclusions would lead to no possible
> variant. Please archive or delete your product directly if intended.

This does **not** fire for a template with a dynamic attribute, because such a template legitimately
has zero variants and is therefore never deleted by its last variant going away.

### 14.3 An archived template's variants are not reactivated

The activation step of variant generation applies only to variants whose template is active. A
combination that becomes possible again on an archived template stays archived until the template is
unarchived.

### 14.4 Adding a single-value attribute does not recreate variants

The single-value repair step enriches the existing variants with the new value in place, provided
the resulting combination has exactly one value per line and covers exactly the template's lines.
This is why adding a "Material: Oak" attribute to an existing catalogue does not orphan every
historical order line.

### 14.5 Generation is suppressible

Every entry point honours a suppression flag named `create_product_product` ("create product
variant"). When the flag is present and false, template creation, template modification and value
materialisation all skip the generation step. This is used by the multi-step product import, which
must create templates, attributes, lines and variants in a specific order.

---

## 15. Combination possibility — the complete rule set

A combination is **possible** exactly when all of the following hold. Each row names the check and
the reason it exists.

| # | Check | Why |
|---|---|---|
| 1 | The template is not archived (when enumerating) | An archived product cannot be ordered |
| 2 | The combination names exactly one value per non-multi-checkbox attribute line of the template | A combination must be complete and unambiguous |
| 3 | The lines covered by the combination are exactly the template's non-multi-checkbox lines | The combination must not name an attribute the template does not use |
| 4 | Every member is an active materialised value of the template | A withdrawn value cannot be chosen |
| 5 | No member is excluded by another member | Own exclusions |
| 6 | For a template with a dynamic attribute: either no variant exists for the combination, or the existing variant is active | An explicitly archived dynamic combination stays refused |
| 7 | For a template with no dynamic attribute: a variant exists for the combination and is active | A deleted or archived combination is not orderable |
| 8 | No value of the combination appears anywhere in the parent-exclusion map | Parent exclusions |

Checks 2 to 5 make up the **configuration filter**, used on its own by variant generation. Checks 1
to 8 make up the **full possibility predicate**, used by the configurator, the matrix and the
create-on-demand procedure.

When no-variant attributes are to be ignored — as they are during variant generation and when
testing an existing variant — the template's lines in checks 2 and 3 exclude the lines whose
attribute never creates variants.

---

## 16. Expiry

### 16.1 Expiry dates require tracking

**Class** silent correction. Writing a product's tracking mode to `none` ("no tracking") forces the
use-expiration-date flag to false in the same write.

### 16.2 Expired goods block a delivery until confirmed

**Class** interactive guard, run before a transfer is completed.
**Condition** the caller has not already confirmed (the confirmation is carried by a flag named
`skip_expired`, "skip expired"), and at least one move line of the transfer either has a lot whose
expiry alert flag is true, or has a removal date at or before the current moment.
**Effect** instead of completing, the system opens the Expiry Delivery Confirmation dialog listing
the offending lots. The dialog offers two outcomes:

- **Confirm** — the transfer is completed again with the confirmation flag set, so the check is
  skipped;
- **Discard expired products** — every move line of the listed transfers that uses expiration dates
  and whose removal date is strictly before the current moment is deleted, and the transfer is then
  completed with a cleaned context.

Note the asymmetry: the guard triggers on "removal date at or before now", while the discard action
removes lines whose removal date is strictly before now. A line whose removal date is exactly the
current moment triggers the dialog but is not discarded by the discard action.

**Dialog text** see [entities.md](entities.md), section 21.

### 16.3 Expired stock is not available

A stock quantity record whose product uses expiration dates and whose removal date is at or before
the current moment has an available quantity of **zero**, regardless of what is physically on hand
and unreserved. Quantities already reserved stay reserved.

### 16.4 Reservation and availability use the move's own date

For a product that uses expiration dates, the reservation and available-quantity computations are
performed in a context whose "as of" moment is the **move's date**, not the current moment. So a
move scheduled in the future will not reserve stock that will have passed its removal date by then.

### 16.5 A lot is reminded only once

The scheduled expiry reminder marks every lot it selected as reminded, including the lots it filtered
out for having no internal stock. Moving an alert date backwards after the reminder has fired does
not produce a second reminder.

### 16.6 First-expiry-first-out ordering

When the removal strategy is first expiry first out, the candidate stock quantity records are ordered
by removal date ascending, then by entry date ascending, then by identifier ascending. Records with
no removal date sort according to the database's ordering of missing values.

---

## 17. Permissions and access

### 17.1 Groups

| Group | Full name | Effect |
|---|---|---|
| `product.group_product_manager` | Products — Create | Full create, read, update and delete on every catalog entity. Implied by the system administration group; granted to the root user and to the administrator user. |
| `product.group_product_variant` | Manage Product Variants | Reveals the attribute and variant parts of the interface. Granted to every internal user as soon as the product matrix capability is installed. |
| `product.group_product_pricelist` | Basic Pricelists | Reveals pricelists. Belongs to `../pricing-and-pricelists/` but is bootstrapped from here. |
| `product_expiry.group_expiry_date_on_delivery_slip` | Include expiration dates on delivery slip | Adds the expiry columns to the printed delivery document. |
| `uom.group_uom` | Units of Measure and Packagings | Reveals the unit fields. Belongs to `../units-of-measure-and-packaging/`. |

### 17.2 Access rights

The full matrix is in [configuration.md](configuration.md). The shape is uniform: every internal
user may **read** every catalog entity and may do nothing else; the product manager group may create,
read, update and delete. Three exceptions:

- the label layout dialog is fully writable by every internal user, because it is transient;
- the attribute-value bulk update dialog is create-read-update for the product manager group and
  cannot be deleted;
- the unit of measure entity gains full rights for the product manager group, so that a product
  manager can define a packaging unit without holding the unit-of-measure administration right.

### 17.3 Record rules

| Entity | Rule |
|---|---|
| Product Template | The record's company is a parent of one of the acting companies, or the record has no company |
| Product Document | Same |
| Product Combo | The record has no company, or its company is a parent of one of the acting companies |

Product Variants are governed through their template: the variant entity declares that its company
scoping follows the parent-of relation, and reading a variant requires reading its template.

### 17.4 Reads performed as a privileged reader

Several computations deliberately read more than the acting user may:

- the variant display name reads vendor price lines as a privileged reader, after checking that the
  user may read the variant itself, so that the vendor-specific name is complete;
- the barcode uniqueness check searches as a privileged reader, then filters the reported product
  names down to those the user may read and appends the note that inaccessible products are not
  shown;
- the combination-to-variant lookup is performed as a privileged reader so that its cache can be
  shared between users;
- the create-on-demand procedure creates the variant as a privileged writer, so that a salesperson
  who may not create products may still configure one;
- the automatic creation of a document from an attachment is performed as a privileged writer;
- the "default extra price changed" flag searches Template Attribute Values as a privileged reader
  precisely in order to know which products the user cannot see.

### 17.5 Reference checks on the variant's reference field

The variant's computed reference field first asks whether the reader has read access on vendor price
lines. If not, the internal reference is returned unchanged and no vendor lookup happens at all.

---

## 18. Ordering invariants

| Entity | Order |
|---|---|
| Product Template | favourite descending, then name |
| Product Variant | internal reference, then name, then identifier |
| Product Category | complete name |
| Product Tag | sequence, then identifier |
| Product Attribute | sequence, then identifier |
| Attribute Value | attribute, then sequence, then identifier |
| Template Attribute Line | sequence, then attribute, then identifier |
| Template Attribute Value | attribute line, then attribute value, then identifier |
| Template Attribute Exclusion | template, then identifier |
| Attribute Custom Value | the referenced template attribute value, then identifier |
| Product Combo | sequence, then identifier |
| Product Document | sequence, then name |
| Barcode Rule | sequence ascending, then identifier |
| Vendor Pricelist Line | sequence, then minimum quantity **descending**, then price, then identifier |

The ordering of Template Attribute Values is what makes a combination's member order deterministic,
which in turn makes the variant display name and the closest-possible-combination algorithm
deterministic.

---

## 19. Edge cases worth stating explicitly

1. **A template with no attributes** has exactly one variant, whose combination is empty and whose
   combination indices are the empty string. The uniqueness index treats the empty string as a
   value, so a second active empty-combination variant is refused.
2. **The empty combination is possible** on a template with no attribute lines, and impossible on a
   template with attribute lines.
3. **A multi-checkbox line contributes nothing to the cartesian product**: its value list is
   deliberately empty in the enumeration, so the enumeration produces combinations with no value
   from that line, and the configuration filter ignores multi-checkbox lines when counting.
4. **A no-variant attribute's value never reaches a variant**, but it does reach the possibility
   predicate: the create-on-demand procedure requires the *complete* combination including
   no-variant members, precisely so that a parent exclusion on a no-variant value is honoured.
5. **An attribute whose mode changed after variants existed** may leave no-variant values stored on
   variants. The no-variant extra price formula guards against double-counting by excluding values
   already in the variant's own combination, rather than by looking at the attribute's mode.
6. **Archiving the last active variant archives the template**, which then archives the variant
   again — a harmless no-op, but implementations must not loop.
7. **Copying a template does not copy the materialised attribute values**; they are regenerated, and
   the extra prices are then re-applied one by one, guarded by a check that the attribute and the
   underlying value match.
8. **The combo base price is computed at the current moment**, so a combo choice's base price may
   change between two reads of the same record when a currency rate changes.
9. **Prorating a zero-weight combo** distributes the price evenly rather than concentrating it, and
   in both branches the rounding remainder goes to the last choice group in sequence order.
10. **A barcode search under a Global Standards One nomenclature is rewritten to a contains
    comparison**, so a search that used to be exact becomes fuzzy. Callers that need exactness must
    set the suppression flag — as the barcode uniqueness check does.
11. **The classic parser applies the encoding check to the original scan, not the converted one**,
    so the conversion policy has an effect only for rules with the `any` encoding or when the scan
    already satisfies the rule's encoding.
12. **An alias rule does not restart the rule list**; it replaces the working barcode and the loop
    continues with the *next* rule. A chain of aliases therefore only resolves forwards.
13. **The parse result's code field for an alias** is set to the alias even when no later rule
    matches, so an unmatched scan that hit an alias reports the alias as its code and `error` as its
    type.
14. **A product document's deletion deletes its attachment**, and an attachment's deletion deletes
    its document through the cascading link, so the two always disappear together.
15. **The category product count includes descendants** despite the help text saying otherwise.
