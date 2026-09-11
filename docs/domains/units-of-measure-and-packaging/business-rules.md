# Units of Measure and Packaging — Business Rules

Every validation, constraint, invariant, protection, permission check and edge-case behaviour of
the domain, with the exact user-facing message where one exists.

Message texts are reproduced verbatim, in the language they are authored in, inside fenced blocks.
Where a message contains a placeholder, the placeholder is described in words immediately below
the block. Where a message contains an abbreviation, the abbreviation is expanded immediately
below the block; the message itself is **not** rewritten, because a rebuild must produce the same
text.

---

## 1. Rules on the Unit of Measure entity

### RULE-UNIT-01 — A contained quantity may not be zero

**Kind:** stored check on the table, evaluated by the database on every insert and update.

**Condition:** the contained quantity (`relative_factor`) must not equal zero.

**Message:**

```
The conversion ratio for a unit of measure cannot be 0!
```

**Rationale:** a unit whose contained quantity is zero has an absolute quantity of zero, and every
conversion *into* it divides by zero.

**Note:** the check forbids only zero. A negative contained quantity passes. See RULE-UNIT-02.

### RULE-UNIT-02 — A negative contained quantity is not rejected by the database

**Kind:** absence of a rule, stated so that a rebuild does not add one silently.

**Behaviour:** a contained quantity of minus two is accepted by the stored check and produces a
negative absolute quantity. Conversions through such a unit change the sign of quantities.

**Industry-standard default:** a rebuild should reject a contained quantity that is not strictly
greater than zero **at the user interface layer**, while keeping the stored check exactly as
specified, so that an existing database containing such a row still loads.

### RULE-UNIT-03 — A root unit must contain exactly one

**Kind:** validation, evaluated when the contained quantity or the reference unit changes.

**Condition:** if a unit has no reference unit, its contained quantity must equal exactly one.

**Message:**

```
Reference unit of measure is missing.
```

**Rationale:** the absolute quantity of a root unit is its contained quantity, and "absolute
quantity" is defined as "how many of the root unit". A root that contained anything but one would
make that definition circular.

**Failure mode a rebuild must reproduce:** the message names the *missing reference unit*, not the
wrong contained quantity, because the expected repair is to give the unit a reference unit rather
than to reset its contained quantity to one.

### RULE-UNIT-04 — Protected units cannot be deleted

**Kind:** deletion guard, evaluated before any unit is removed, and skipped when a capability is
being uninstalled.

**Condition:** a unit is **protected** when an external identifier delivered by the units
capability points at it **and** the short part of that identifier is not on the unprotected list.

**The unprotected list, by default:**

| Short identifier | Unit |
|---|---|
| `product_uom_hour` | The working-hour unit |
| `product_uom_dozen` | The dozen |
| `product_uom_pack_6` | The pack of six |

**The unprotected list where the time-recording capability is installed:**

| Short identifier | Unit |
|---|---|
| `product_uom_dozen` | The dozen |
| `product_uom_pack_6` | The pack of six |

That is, installing time recording **protects the working-hour unit**, because timesheets are
recorded against it.

**Message:**

```
The following units of measure are used by the system and cannot be deleted: %s
You can archive them instead.
```

The placeholder is the comma-and-space separated list of the names of the protected units in the
deletion attempt.

**Consequence:** twenty-seven of the thirty shipped units can never be deleted; the dozen and the
pack of six always can; the working hour can unless time recording is installed.

### RULE-UNIT-05 — Editing a protected unit raises a warning

**Kind:** non-blocking warning, raised while editing, before saving.

**Condition:** all of the following:

1. the contained quantity is being changed;
2. the unit is protected in the sense of RULE-UNIT-04;
3. the unit was created more than one day ago.

**Title:**

```
Warning for %s
```

The placeholder is the unit's name.

**Message:**

```
Some critical fields have been modified on %s.
Note that existing data WON'T be updated by this change.

As units of measure impact the whole system, this may cause critical issues.
Therefore, changing core units of measure in a running database is not recommended.
```

The placeholder is the unit's name.

**Behaviour after the warning:** the user may proceed. The absolute quantity of the unit and of
every descendant is recomputed. **No stored quantity on any document is recomputed.**

### RULE-UNIT-06 — Deleting a unit cascades to its descendants

**Kind:** referential action.

**Behaviour:** the reference-unit link is declared to cascade, so deleting a unit deletes every
unit that names it as reference, recursively, and deletes the barcodes bound to all of them.

**Interaction with RULE-UNIT-04:** the deletion guard is evaluated on the units explicitly being
deleted. A rebuild must evaluate the guard over the full cascade closure as well, otherwise
deleting an unprotected unit could remove a protected descendant. In the shipped data no
unprotected unit has descendants, so the situation does not arise; the rule is stated for
completeness.

### RULE-UNIT-07 — A unit may not be its own ancestor

**Kind:** invariant enforced by the tree storage.

**Condition:** the reference-unit chain must be acyclic. Assigning a unit a reference unit that is
itself, or one of its own descendants, is refused by the platform's tree maintenance with its
generic recursion message.

**Industry-standard default:** a rebuild must detect the cycle before writing and refuse the write
with a message stating that the unit cannot be set as its own reference unit, directly or
indirectly.

### RULE-UNIT-08 — Units are never company scoped

**Kind:** invariant.

**Behaviour:** the entity has no company field. Every company sees every unit. A rebuild must not
add a company column, and must not filter units by company anywhere.

### RULE-UNIT-09 — Every unit shares one rounding precision

**Kind:** invariant.

**Behaviour:** the rounding precision reported by a unit is derived from the `Product Unit`
decimal precision record and nothing else. Two units never report different precisions.

**Rebuild guard:** a rebuild that stores a per-unit rounding must, at minimum, force every stored
value to the global one; otherwise the conversion matrix will not match.

### RULE-UNIT-10 — Names are not unique

**Kind:** absence of a rule, stated explicitly.

**Behaviour:** several units may bear the same name. The business relies on it: every product that
ships in a carton of a different size needs its own unit, and users commonly name them all `Box`.
Selection lists disambiguate by showing the formatted display name.

---

## 2. Rules on the Product Unit Barcode entity

### RULE-BARCODE-01 — A barcode identifies one packaging row

**Kind:** stored uniqueness constraint on the table, installation-wide.

**Message:**

```
A barcode can only be assigned to one packaging.
```

**Note:** the constraint is **not** scoped by company, although the entity has a company field.
Two companies cannot bind the same packaging barcode to different products.

### RULE-BARCODE-02 — A packaging barcode may not collide with a product barcode

**Kind:** validation on the barcode, evaluated on create and on change.

**Condition:** no product variant carries the same barcode.

**Message:**

```
A product already uses the barcode
```

### RULE-BARCODE-03 — A product barcode may not collide with a packaging barcode

**Kind:** validation on the product variant's barcode, evaluated on create and on change, per
company.

**Condition:** no Product Unit Barcode row of the same company carries the same barcode.

**Message:**

```
A packaging already uses the barcode
```

**Rationale for RULE-BARCODE-02 and RULE-BARCODE-03:** under the Global Standards One barcode conventions a
product code and a packaging code are drawn from the same pattern space, so a scanner cannot tell
them apart. The two entities therefore share one namespace.

### RULE-BARCODE-04 — A barcode is required

**Kind:** field requirement. A Product Unit Barcode row with no barcode cannot be created.

### RULE-BARCODE-05 — Barcodes are not copied

**Kind:** copy behaviour. Duplicating a product does not duplicate its packaging barcodes, because
the copy would immediately violate RULE-BARCODE-01.

---

## 3. Rules on the Package Type entity

### RULE-PACKAGE-TYPE-01 — Dimensions and maximum weight must not be negative

**Kind:** four stored checks.

| Field | Condition | Message |
|---|---|---|
| `height` | greater than or equal to zero | `Height must be positive` |
| `width` | greater than or equal to zero | `Width must be positive` |
| `packaging_length` | greater than or equal to zero | `Length must be positive` |
| `max_weight` | greater than or equal to zero | `Max Weight must be positive` |

Note that each message says "positive" while each check permits zero.

### RULE-PACKAGE-TYPE-02 — The tare weight is unchecked

**Kind:** absence of a rule. A negative tare passes the database.

**Industry-standard default:** reject a negative tare at the user interface.

### RULE-PACKAGE-TYPE-03 — A barcode identifies one package type

**Kind:** stored uniqueness constraint, installation-wide.

**Message:**

```
A barcode can only be assigned to one package type!
```

**Note:** package type barcodes are in a **different** namespace from product and packaging
barcodes; there is no cross-check between them.

### RULE-PACKAGE-TYPE-04 — The numbering sequence follows the prefix

**Kind:** invariant maintained on create and on update.

**Condition:** whenever a package type has a sequence prefix, a numbering sequence exists whose
code is that prefix, whose name is the words `Package Type Sequence` followed by a space and the
prefix, whose padding is seven digits, and whose company is the package type's company.

### RULE-PACKAGE-TYPE-05 — Duplicating renames

**Kind:** copy behaviour. The duplicate's name is the original's name followed by a space, an
opening parenthesis, the word `copy`, and a closing parenthesis. Storage capacities are copied;
the barcode and the sequence link are not.

---

## 4. Rules on the Decimal Precision entity

### RULE-PRECISION-01 — One value per usage

**Kind:** stored uniqueness constraint on the usage name.

**Message:**

```
Only one value can be defined for each given usage!
```

### RULE-PRECISION-02 — An unknown usage means two digits

**Kind:** lookup fallback.

**Behaviour:** asking for the number of digits of a usage that has no record returns two. A
rebuild must reproduce the fallback rather than failing.

### RULE-PRECISION-03 — Reducing the precision warns

**Kind:** non-blocking warning, raised while editing.

**Condition:** the new number of digits is smaller than the stored one.

**Title:**

```
Warning for %s
```

The placeholder is the usage name.

**Message:**

```
The precision has been reduced for %s.
Note that existing data WON'T be updated by this change.

As decimal precisions impact the whole system, this may cause critical issues.
E.g. reducing the precision could disturb your financial balance.

Therefore, changing decimal precisions in a running database is not recommended.
```

The placeholder is the usage name.

### RULE-PRECISION-04 — The cache must be cleared on every write

**Kind:** invariant.

**Behaviour:** the digit lookup is cached. Creating, updating or deleting any decimal precision
record clears the cache. A rebuild that caches without clearing will keep converting at the old
precision.

---

## 5. Rules on a product's units

### RULE-PRODUCT-01 — A product always has an own unit

**Kind:** field requirement plus a default.

**Behaviour:** the own unit is required. When a product is created without one, the counting unit
delivered as reference data is used. This default is applied **even when the caller explicitly
asks for no default**, which is a deliberate exception to the usual default handling.

### RULE-PRODUCT-02 — The own unit may not appear among the additional units

**Kind:** selection restriction on the additional-units field.

**Condition:** the selectable units exclude the product's own unit.

**Industry-standard default:** a rebuild should also validate the stored value, not only restrict
the selection, and should silently drop the own unit from the additional list on write rather than
failing, because the restriction exists to avoid a duplicate entry in the allowed-unit list rather
than to protect an invariant.

### RULE-PRODUCT-03 — Changing the own unit relabels, never converts

**Kind:** operation semantics.

**Behaviour:** writing a different own unit on a product does **not** convert any stored quantity.
It rewrites the unit reference on the records that hold the product, leaving the numbers alone.

**Warning shown before saving** (non-blocking), with the title:

```
What to expect ?
```

and the message:

```
Changing the unit of measure for your product will apply a conversion 1 %(old_uom_name)s = 1 %(new_uom_name)s.
All existing records (Sales orders, Purchase orders, etc.) using this product will be updated by replacing the unit name.
```

The first placeholder is the display name of the unit being replaced; the second is the display
name of the new unit.

**When the warning appears:** only when the product's variants signal that historical quantities
exist. The base catalogue never signals this. The signal is raised when at least one stock move
exists for the product, when at least one sales order line exists for it, when at least one
purchase order line exists for it, or when at least one bill of materials names it — each
capability adding its own test.

### RULE-PRODUCT-04 — Changing the own unit is refused when another unit is already in use

**Kind:** blocking validation, evaluated as part of the write.

**Condition:** for each of the record kinds below, group the existing records by the unit they
hold; if any group's unit differs from the product's *current* own unit, refuse.

| Record kind | Checked where |
|---|---|
| Stock moves | Warehouse capability |
| Stock move lines | Warehouse capability |
| Sales order lines | Sales capability |
| Purchase order lines | Purchasing capability |
| Bills of materials | Manufacturing capability |

**Message:**

```
As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one.
```

The first placeholder is the name or display name of the offending unit; the second is the name or
display name of the product's current own unit. Note the missing space before the word `If`: the
two sentences are concatenated without a separator, and a rebuild must reproduce the text as it
is.

**When the check passes:** every existing record holds the product's current own unit, and each of
them has its unit reference rewritten to the new own unit, in bulk, without touching the
quantities.

### RULE-PRODUCT-05 — Changing the own unit is refused when a posted invoice uses another unit

**Kind:** blocking validation on the product template, evaluated whenever the own unit changes,
present where the accounting capability is installed.

**Condition:** no journal item of a posted document may hold a unit different from the product
template's own unit.

**Message:**

```
This product is already being used in posted Journal Entries.
If you want to change its Unit of Measure, please archive this product and create a new one.
```

### RULE-PRODUCT-06 — Multiple units are a feature, not a data state

**Kind:** derived predicate.

**Condition:** a product is treated as having multiple units when **all** of the following hold:

1. the product's type is not a combination offer;
2. the multiple-units feature group is enabled for the installation;
3. the set formed by the own unit together with the additional units has more than one member.

**Effect:** the predicate drives whether a unit selector is shown at all on storefront and
configurator screens.

---

## 6. Rules on document lines

### RULE-LINE-01 — A line's unit must be one of the product's allowed units

**Kind:** selection restriction on every document line, expressed as a domain over a computed
allowed-unit list.

| Line kind | Allowed units |
|---|---|
| Sales order line | The product's own unit and its additional units. |
| Purchase order line | The product's own unit, its additional units, **and** the units of every vendor price list line for that product. |
| Invoice or bill line | The product's own unit and its additional units. |
| Stock move | The product's own unit, its additional units, and the units of the product's vendor price list lines. |
| Stock move line | The same list as the stock move. |

**Note:** the list is a *restriction on selection*, not a stored constraint. A value written
programmatically outside the list is accepted. A rebuild should mirror this: enforce the list in
the user interface and in the remote transport's field validation, but do not add a stored check,
because several internal flows deliberately write a unit that is outside the list — notably the
receipt of goods in a vendor unit that was later removed from the vendor's price list.

### RULE-LINE-02 — A display-only line holds no unit

**Kind:** stored check on sales order lines and purchase order lines.

**Condition:** either the line has a display type and then it must have no product, no unit, a
unit price of zero and a quantity of zero; or it has no display type and then it must have a
product and a unit (and, for purchases, a planned date), unless it is a down payment.

**Messages:** the two stored checks carry the platform's generic constraint messages naming the
offending combination. A rebuild should report that a section or note line cannot carry a product,
a unit, a price or a quantity, and that a product line must carry a product and a unit.

### RULE-LINE-03 — A unit referenced by a document line cannot be deleted

**Kind:** referential action, declared as *restrict* on the sales order line unit, the purchase
order line unit, and the invoice line unit.

**Effect:** in practice a unit that has ever been used on a commercial document cannot be deleted
at all, even if it is unprotected. Archiving is the only route.

### RULE-LINE-04 — A move's unit is frozen once the move is complete

**Kind:** blocking validation on write, bypassed only by the internal relabelling flow of
RULE-PRODUCT-04.

**Condition:** the unit of a move whose state is complete may not be changed.

**Message:**

```
You cannot change the UoM for a stock move that has been set to 'Done'.
```

The abbreviation in the message stands for *unit of measure*.

### RULE-LINE-05 — The real quantity of a move may not be written directly

**Kind:** blocking validation.

**Condition:** any attempt to write the move's real quantity — the copy held in the product's own
unit — is refused, because the intent was almost certainly to write the demand.

**Message:**

```
The requested operation cannot be processed because of a programming error setting the `product_qty` field instead of the `product_uom_qty`.
```

The first storage name in the message is the real quantity in the product's own unit; the second
is the demand in the move's unit.

### RULE-LINE-06 — A picked quantity must lie on the precision grid

**Kind:** blocking validation, evaluated when the picked quantity of a move is applied.

**Condition:** rounding the picked quantity half away from zero at the `Product Unit` precision
must leave it unchanged, as judged by the comparison operation at that same precision.

**Message**, one paragraph per offending move, joined by newline characters:

```

The quantity done for the product %(product)s doesn't respect the rounding precision defined on the system.
Please change the quantity done or the rounding precision in your settings.
```

The placeholder is the product's display name. Note the leading newline character inside the
message: each paragraph begins with a blank line.

**When it fires:** after the `Product Unit` precision has been reduced while quantities recorded
at the finer precision are still open.

### RULE-LINE-07 — A serial-tracked move line resolves to exactly one

**Kind:** blocking validation, evaluated while editing the quantity or the unit of a move line.

**Condition:** for a product tracked by serial number, the move line's quantity converted into the
product's own unit must be either zero or exactly one, as judged by the comparison and zero tests
at the `Product Unit` precision.

**Message:**

```
You can only process 1.0 %s of products with unique serial number.
```

The placeholder is the name of the product's own unit.

### RULE-LINE-08 — A conversion that rounds to zero blocks a counter-sale transfer

**Kind:** blocking validation, evaluated before a counter sale's transfer is completed.

**Condition:** for every move whose demand is non-zero and whose unit differs from the product's
own unit, converting the demand into the product's own unit half away from zero must not yield
zero.

**Message**, assembled from parts joined by newline characters:

```
Conversion Error: The following unit of measure conversions result in a zero quantity due to rounding:
 - From "%(uom_from)s" to "%(uom_to)s"

This issue occurs because the quantity becomes zero after rounding during the conversion. To fix this, adjust the conversion factors or rounding method to ensure that even the smallest quantity in the original unit does not round down to zero in the target unit.
```

The middle line repeats once per distinct offending pair; the first placeholder is the source
unit's name and the second the destination unit's name.

### RULE-LINE-09 — A storefront cart refuses an unavailable unit

**Kind:** blocking validation.

**Condition:** when a visitor adds a product to a cart naming a unit, that unit must belong to the
product's available units, unless the product does not support multiple units, in which case the
named unit is ignored and the product's own unit is used.

**Message:**

```
This product is not available (anymore) in this unit of measure.
```

### RULE-LINE-10 — A batch size must be positive in the bill's unit

**Kind:** blocking validation on a bill of materials with batching enabled.

**Condition:** the batch size, compared against zero at the `Product Unit` precision, must be
greater than zero.

---

## 7. Rules on conversion itself

### RULE-CONVERSION-01 — Zero converts to zero

A quantity of zero is returned unchanged whatever the units and whatever the rounding method. No
rounding step runs.

### RULE-CONVERSION-02 — An absent source unit converts to the input

A conversion invoked with no source unit returns the input quantity unchanged and unrounded.

### RULE-CONVERSION-03 — An absent destination unit skips rounding

A conversion invoked with no destination unit returns the quantity multiplied by the source
unit's absolute quantity, **unrounded**, even when rounding was requested.

### RULE-CONVERSION-04 — A same-unit conversion skips the factor arithmetic but not the rounding

Converting to the same unit returns the quantity rounded onto the grid with the requested method,
with no multiplication and no division.

### RULE-CONVERSION-05 — Conversion performs no tree check

The conversion operation itself never verifies that source and destination share an ancestor. The
*raise on failure* input expresses the caller's intent, and the caller is responsible for the
check. A rebuild must expose the same parameter and the same two behaviours: fail when failure is
to be raised, return the input unchanged when it is to be tolerated.

### RULE-CONVERSION-06 — Price conversion never rounds

No rounding step exists in price conversion. Callers round afterwards using a *currency* or
*price* precision, never the `Product Unit` precision.

### RULE-CONVERSION-07 — Whole-packaging rounding never uses a remainder operation

The division-round-multiply form is mandatory. A remainder-based test fails on ordinary values
because of binary representation, and the implementation notes two concrete counter-examples.

### RULE-CONVERSION-08 — Whole-packaging rounding is an identity when the units coincide

When the packaging unit and the destination unit are the same record, the input quantity is
returned untouched. Without the short-circuit, a product with no packagings whose category
requires full packagings would silently lose the fractional part of every reservation.

### RULE-CONVERSION-09 — The comparison and the zero test are not interchangeable

A rule that says two quantities are equal means the comparison returns zero. A rule that says a
quantity is zero means the zero test returns true. The two disagree when the values are close to
the grid boundary.

---

## 8. Permission checks

### RULE-SECURITY-01 — Access rights on the unit entity

| Group | Create | Read | Update | Delete |
|---|---|---|---|---|
| Settings administrators | yes | yes | yes | yes |
| Internal users | no | yes | no | no |
| Everyone else | no | no | no | no |

Read access for every internal user is required because every document line displays a unit name.

### RULE-SECURITY-02 — The multiple-units feature group

The group named `Manage Multiple Units of Measure` is a **feature flag**, not an access right. It
carries no access rights of its own. Its only effect is that fields and columns marked as
belonging to it are shown or hidden.

Consequences of the group being **off**:

- unit columns and selectors disappear from order lines, invoice lines, moves, move lines, bills
  of materials, production orders and the counter interface;
- the additional-units list on a product is hidden;
- the packaging reservation policy on a product category is hidden;
- the content description of a package omits the unit name;
- the derived predicate of RULE-PRODUCT-06 is false for every product, so storefront unit selectors
  never appear;
- **no arithmetic changes.** Conversions still happen; the user simply cannot choose a unit other
  than the product's own, so in practice every line is created in the product's own unit and the
  conversions are identities.

### RULE-SECURITY-03 — Package type and barcode permissions

Package types and product unit barcodes follow the warehouse and catalogue permission matrices
respectively; see [`../inventory-operations/`](../inventory-operations/) and
[`../products-and-catalog/`](../products-and-catalog/). Reading them requires the corresponding
application's user group; creating and editing them requires its manager group.

### RULE-SECURITY-04 — The barcode uniqueness check runs with elevated rights

The cross-entity barcode checks of RULE-BARCODE-02 and RULE-BARCODE-03 search the whole installation with
elevated rights, deliberately, so that a user who cannot see another company's products is still
prevented from re-using their barcodes. The user-facing message does not reveal which record
holds the colliding barcode.

---

## 9. Locking and concurrency rules

### RULE-LOCKING-01 — Recomputing absolute quantities locks the subtree

Changing a unit's contained quantity recomputes the absolute quantity of every descendant in the
same transaction. A rebuild must take the write locks in a deterministic order — by hierarchy
path — so that two concurrent edits in the same tree cannot deadlock.

### RULE-LOCKING-02 — The precision cache is process-wide

The digit lookup is cached beyond a single transaction. A rebuild running several processes must
invalidate the cache across all of them when a decimal precision record changes, or different
processes will round differently within the same business transaction.

### RULE-LOCKING-03 — Conversion holds no lock

Conversion reads only the two units' absolute quantities and the global precision. It takes no
lock and must remain safe to call inside any transaction.

---

## 10. Invariants a rebuild must be able to assert at any time

1. Every unit with no reference unit has a contained quantity of one and an absolute quantity of
   one.
2. Every unit with a reference unit has an absolute quantity equal to its contained quantity
   multiplied by its reference unit's stored absolute quantity.
3. No unit has a contained quantity of zero.
4. No unit is its own ancestor.
5. Every unit's hierarchy path starts at a root of the forest and ends with the unit's own
   identifier.
6. Every unit reports the same rounding precision.
7. No barcode appears twice across the product variant table and the product unit barcode table
   taken together.
8. No barcode appears twice in the package type table.
9. Every stock move's real quantity equals its demand converted into the product's own unit half
   away from zero.
10. Every stock move line's quantity in the product's unit equals its quantity converted into the
    product's own unit half away from zero.
11. Every purchase order line's total quantity equals its ordered quantity converted into the
    product's own unit, or the ordered quantity when the units coincide.
12. No product lists its own unit among its additional units.
13. Every quantity on hand is expressed in its product's own unit; no quantity-on-hand row carries
    a unit reference.
14. Every package type with a sequence prefix owns a numbering sequence whose code equals that
    prefix and whose padding is seven.

---

## 11. Edge-case behaviours, collected

| Situation | Behaviour |
|---|---|
| Converting zero | Returns zero, unrounded, whatever the units. |
| Converting between the same unit | Rounds onto the grid; performs no factor arithmetic. |
| Converting with no destination | Returns the quantity in the tree's root unit, unrounded. |
| Converting across trees, failure tolerated | Returns the input quantity unchanged. |
| Converting across trees, failure raised | Fails. |
| Converting a tiny quantity with the default method | Never yields zero; yields one step. |
| Converting a tiny quantity half away from zero | May yield zero. |
| Whole-packaging rounding with coinciding units | Returns the quantity untouched. |
| Whole-packaging rounding of zero | Returns zero. |
| Whole-packaging rounding when the packaging quantity rounds to zero | Returns the quantity untouched, because the packaging quantity is falsy. |
| Archiving a unit in use | Allowed; documents keep working. |
| Deleting a unit in use on a document | Refused by the referential restriction. |
| Deleting an unprotected unit with descendants | Cascades to the descendants. |
| Reducing the precision below what open documents use | Allowed; open transfers then fail at completion with RULE-LINE-06. |
| Setting the precision to zero digits | Allowed; every quantity becomes whole, and small conversions inflate to one or collapse to zero. |
| Removing the `Product Unit` precision record | Allowed; the precision silently becomes two digits. |
| A product with no additional units and the full-packaging policy | The reservation identity short-circuit keeps quantities exact. |
| A vendor quoting in a unit that is not among the product's additional units | Allowed; the purchase line's allowed list includes vendor units. |
| A customer ordering in a unit that is not among the product's additional units | Refused by the sales line's allowed list. |
| Two units with the same name | Allowed; disambiguated by the formatted display name. |
