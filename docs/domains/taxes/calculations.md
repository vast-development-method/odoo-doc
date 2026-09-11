# Taxes — Calculations

This document specifies the tax computation engine completely: the data shapes it consumes and
produces, the rounding primitives, the ordering and grouping of taxes, the amount formula of every
computation kind, the propagation of one tax's amount into another tax's base, the document-wide
rounding passes, the derivation of accounting data, the totals block, the fiscal position mapping,
the cash basis percentages, the withholding amounts and the arithmetic of every country's tax
identification number check.

Read the sections in order: each one depends on the previous ones.

Two properties of the engine must be understood before anything else:

- **The engine is a pure function of a list of base lines and a company.** It never reads the
  document. Sales orders, purchase orders, invoices, bills, payments, point-of-sale orders and
  expense reports all convert their own lines into base lines and call the same engine, which is
  why they agree to the cent.
- **The engine is duplicated on the client side.** A second implementation, in the language of the
  browser, mirrors every step marked below as *mirrored*. The two must agree exactly; any
  divergence shows up as a total that changes when a document is saved. Steps marked
  *server-only* exist only where accounting records are produced.

---

## 1. Data shapes

### 1.1 The base line

A **base line** is the engine's representation of one taxable amount. It is built from any record
or plain set of values by the preparation step of section 1.3. It contains:

| Key | Meaning |
|---|---|
| `record` | The originating record, kept only so that the caller can find its way back. |
| `id` | The originating record's identifier, or zero. |
| `product_id` | The product, used by custom-formula taxes and to collect product tags. |
| `product_uom_id` | The unit of measure, used by custom-formula taxes. |
| `tax_ids` | The taxes to apply. |
| `price_unit` | The unit price. |
| `quantity` | The quantity. |
| `discount` | The discount percentage applied to the unit price. |
| `currency_id` | The currency the price is expressed in. |
| `special_mode` | `false`, `total_excluded` or `total_included` — see section 1.2. |
| `special_type` | `false`, `early_payment`, `cash_rounding`, `non_deductible`, `global_discount` or `down_payment`. Marks lines that need extra treatment outside the engine proper. |
| `rate` | The number of units of the line currency per unit of the company currency. |
| `filter_tax_function` | An optional predicate removing some taxes from the evaluation without unlinking them from their parent group. |
| `sign` | The sign to apply when turning amounts into accounting balances (plus one or minus one). |
| `is_refund` | Whether the refund distribution must be used instead of the invoice distribution. |
| `partner_id`, `account_id`, `analytic_distribution` | Carried to the produced tax entries. |
| `computation_key` | Splits one document into independent computation subsets (section 9.2). |
| `manual_total_excluded_currency`, `manual_total_excluded` | Forced untaxed amounts (section 9.1). |
| `manual_tax_amounts` | Forced per-tax base and tax amounts (section 9.1). |

### 1.2 The special mode

`special_mode` overrides what the supplied price means.

| Value | Meaning |
|---|---|
| `false` | The price means what the taxes say: a price-included tax is extracted from it, a price-excluded tax is added on top. |
| `total_excluded` | The supplied price is the amount **without** any tax. Every tax, including those flagged price-included, is added on top. Supplying one hundred with a twenty-one percent price-included tax under this mode gives the same result as supplying one hundred twenty-one with no mode. |
| `total_included` | The supplied price is the amount **with** all taxes. Every tax, including those flagged price-excluded, is extracted from it. Supplying one hundred twenty-one with a twenty-one percent price-excluded tax under this mode gives the same result as supplying one hundred with no mode. |

Symmetry between the two modes is only guaranteed when the price is not pre-rounded **and** the
document-wide rounding method is "round per tax". With "round per line" the two directions can
differ by one unit of the last decimal place.

### 1.3 Preparing a base line from a record

*Mirrored.* Each key is resolved by the following rule, in this order:

1. If the caller passed the key explicitly, use the caller's value; if that value is falsy, use
   the fallback.
2. Otherwise, if the source is a record and has a field with that name (and the caller did not
   say the value must come from the base line), use the record's field value.
3. Otherwise, if the source is a plain set of values, use the value stored under that key, or the
   fallback.
4. Otherwise, use the fallback.

When the fallback is a record set, the resolved value is reduced to its stored original (so that
an unsaved edit does not leak into the computation).

The currency is resolved as: the line currency, else the company currency of the line, else the
currency of the line's company, else nothing.

Defaults: product empty, unit of measure empty, taxes empty, unit price zero, quantity zero,
discount zero, rate one, sign plus one, refund flag false, partner empty, account empty, analytic
distribution empty.

After the basic keys are resolved, the stored extra tax data is merged in (section 9.3), and every
key of the source whose name starts with an underscore and which is not already present is copied
across unchanged, so that callers can smuggle their own annotations through the engine.

### 1.4 The tax line

A **tax line** is the engine's representation of an *existing* tax accounting entry, supplied so
that the engine can decide which entries to keep, update, delete or create. It carries: the
originating record and identifier, the distribution line that produced it, the originating group
of taxes, the base taxes, the report tags, the currency, the partner, the account, the analytic
distribution, the sign, the amount in the line currency and the amount in the company currency.

### 1.5 The tax details block

After the computation, each base line carries a **tax details** block:

| Key | Meaning |
|---|---|
| `raw_total_excluded_currency` / `raw_total_excluded` | Untaxed total, unrounded, in the line currency and in the company currency. |
| `raw_total_included_currency` / `raw_total_included` | Total with taxes, unrounded, in both currencies. |
| `total_excluded_currency` / `total_excluded` | The same, rounded. |
| `total_included_currency` / `total_included` | The same, rounded. |
| `delta_total_excluded_currency` / `delta_total_excluded` | The share of the document-wide rounding correction allocated to this line. **The accounting balance of the base line is the rounded untaxed total plus this delta**, never the rounded total alone. |
| `taxes_data` | One entry per produced tax result. |

Each entry of `taxes_data` carries: the tax, the taxes it affects downstream, the parent group if
any, the batch it belongs to, whether it was treated as price-included, whether it is the negative
half of a reverse charge, and four pairs of amounts — raw and rounded, base and tax, in the line
currency and in the company currency.

---

## 2. Rounding primitives

Every monetary rounding in the engine uses one primitive. It is defined here in arithmetic because
every downstream number depends on its exact tie-breaking.

### 2.1 Rounding to a step

Let *v* be the value and *s* the rounding step (for a currency with two decimal places, *s* is one
hundredth; for a currency with no decimal places, *s* is one; for a cash rounding to the nearest
five cents, *s* is five hundredths).

```formula
round_to_step( v , s , method ) :
    if s = 0 or v = 0 :  result = 0
    otherwise :
        if s < 1 :
            n = v × invert(s)                         (normalize by multiplying)
        else :
            n = v ÷ s                                 (normalize by dividing)
        e = 2 ^ ( log2( |n| ) − 50 )                  (the tie-correction epsilon)
        method 'half away from zero' :  r = nearest_integer_away_from_zero( n + sign(n) × e )
        method 'half toward zero'    :  r = nearest_integer_away_from_zero( n − sign(n) × e )
        method 'half to even'        :  let i = floor(n), f = |n − i| ;
                                        if |0.5 − f| < e then r = i + (i modulo 2)
                                        else r = nearest_integer_away_from_zero( n )
        method 'away from zero'      :  r = truncate( n + sign(n) × (1 − e) )
        method 'toward zero'         :  r = truncate( n + sign(n) × e )
        if s < 1 :  result = r ÷ invert(s)
        else     :  result = r × s
```

Notes that matter for equivalence:

- **`nearest_integer_away_from_zero`** is not the usual "round half to even" of most standard
  libraries. It is defined as: take the library's nearest-integer value; if adding one to the
  input and taking the nearest integer does not increase the result by exactly one, the input was
  exactly on a tie and the correct answer is the input plus one half carrying the input's sign;
  otherwise the library's value with the input's sign restored. The second half of that rule is
  what keeps the sign of a negative zero.
- **`invert(s)`** is a higher-accuracy reciprocal. Write *s* in scientific notation with fifteen
  significant decimal digits as *c* times ten to the power *x*. Then
  `invert(s) = (c × 10^(−x)) ÷ c²`. For *s* equal to one hundredth this yields exactly one
  hundred. Using this instead of `1 ÷ s` removes a family of one-unit-in-the-last-place errors.
- The epsilon is deliberately larger than the smallest representable step (fifty rather than
  fifty-two binary places below the value) so that a value that has already gone through several
  floating-point operations still breaks ties in the intended direction.
- The default method everywhere in the tax engine is **half away from zero**. The other methods
  are only reachable through the cash rounding feature.

**Worked example.** Round two point six seven five to two decimal places. The stored double is
slightly below the tie (two point six seven four nine nine nine nine …). Normalizing gives two
hundred sixty-seven point four nine nine nine …; the epsilon at that magnitude is about two to the
power minus forty-two, which lifts the value just above two hundred sixty-seven point five; the
nearest integer away from zero is two hundred sixty-eight; denormalizing gives two point six eight.

### 2.2 Zero test

```formula
is_zero( v , s ) =  ( v = 0 )  or  ( | round_to_step( v , s , half away from zero ) | < s )
```

### 2.3 Comparison

```formula
compare( a , b , s ) :
    d = round_to_step( a , s , half away ) − round_to_step( b , s , half away )
    result = −1 if d < −s ÷ 2 ,  +1 if d > s ÷ 2 ,  0 otherwise
```

Comparison rounds **before** subtracting; the zero test rounds **after**. The two therefore
disagree on values such as six thousandths against two thousandths at two decimal places.

### 2.4 Currency rounding

A currency carries a rounding step and a number of decimal places. "Round to the line currency"
means `round_to_step( value , line currency step , half away from zero )`. "Round to the company
currency" is the same with the company currency's step.

---

## 3. Flattening, ordering and batching

### 3.1 Flatten and sort

*Mirrored.* Input: a set of taxes. Output: an ordered list of non-group taxes, plus a map from
each produced tax to the group it came from.

1. Sort the input by (sequence ascending, identifier ascending). An unsaved record sorts as if its
   identifier were absent, which places it first among equal sequences.
2. Walk the sorted input. For a Group of Taxes, sort its children by the same key and append them
   in that order, recording the group for each child. For any other tax, append the tax itself.
3. Duplicates are not appended twice.

Consequence: **a group is evaluated at the position of the group's own sequence, but its children
keep their relative order inside that position.** Writing the taxes as letters ordered
alphabetically, the input *G*, *B* containing (*A*, *D*, *F*), *E*, *C* is evaluated as
*A*, *D*, *F*, *C*, *E*, *G*.

Nested groups are forbidden, so one pass is enough.

### 3.2 Filtering

If the caller supplied a filter predicate, the flattened list is reduced to the taxes that satisfy
it. Filtering happens **after** flattening, so a child removed by the filter still remembers its
parent group and the surviving children of the same group still produce entries carrying that
group.

### 3.3 Batching

*Mirrored.* A **batch** is a maximal run of consecutive taxes, taken in reverse evaluation order,
that must be solved together. Batching matters because the extraction of several price-included
percentage taxes from one price is a single division, not a chain of divisions.

1. Start with an empty batch and a flag "previous tax accepts being affected" set to false.
2. Walk the flattened, filtered list **backwards**. For each tax:
   a. If the batch is not empty, the tax joins it only when **all four** of the following hold:
      - it has the same computation kind as the batch's first member;
      - either a special mode is active, or it has the same price-inclusion flag as the batch's
        first member;
      - it has the same "affects base of subsequent taxes" flag as the batch's first member;
      - either it does not affect the base of subsequent taxes, or it does affect it **and** the
        previously examined tax does not accept being affected.
   b. If any of those fails, close the current batch (assign it to each of its members) and start
      a new one.
   c. Set the flag "previous tax accepts being affected" to this tax's own "base affected by
      previous taxes" flag, then append the tax to the batch.
3. Close the last batch.

Note that step 2c sets the flag from the tax being examined *before* the next iteration reads it,
and because the walk is backwards, "the previously examined tax" is the tax that comes **after**
in evaluation order.

**Worked example.** Two price-included percentage taxes of ten percent each, neither affecting the
base of the other, sequence one and two. Walking backwards: the second forms a batch; the first
has the same kind, the same inclusion flag, the same "affects base" flag (false), so it joins. One
batch of two. Their combined extraction divides by one plus zero point two, not twice by one plus
zero point one.

**Counter-example.** The same two taxes, but the first one affects the base of subsequent taxes
and the second accepts being affected. Walking backwards: the second forms a batch and sets the
flag to true (it accepts being affected). The first affects the base and the flag is true, so the
fourth condition fails: two batches of one each, evaluated in cascade.

---

## 4. The amount of one tax

All formulas below take a **raw base** — the line's raw base plus whatever extra base has been
propagated to this tax (section 5) — and return an unrounded tax amount. The batch is the batch of
section 3.3.

Define, for a batch *B*:

```formula
batch_percentage( B ) = ( sum over t in B of t.amount ) ÷ 100
```

### 4.1 Fixed amount

*Mirrored.* Evaluated in the **first** pass, before any other kind, because a fixed tax can change
the base a price-included batch must be extracted from.

```formula
tax_amount = sign_of_price × quantity × tax.amount
where sign_of_price = −1 when the unit price is strictly negative, +1 otherwise
```

The raw base is ignored. The unit price used for the sign test is the price **after** the discount
has been applied.

**Worked example (mandatory example 3).** A line of seven units at fifteen, with a fixed
environmental levy of five hundredths per unit (sequence one) and a twenty percent tax
(sequence two), price-excluded, company rounding "round per tax", currency with two decimals.

```formula
raw_base       = 7 × 15.00 = 105.00
levy_amount    = +1 × 7 × 0.05 = 0.35
twenty_amount  = 105.00 × 20 ÷ 100 = 21.00
untaxed total  = 105.00
tax total      = 0.35 + 21.00 = 21.35
grand total    = 126.35
```

Both taxes have the same base of one hundred five, because the levy does **not** affect the base
of subsequent taxes. If the levy is flagged "affect base of subsequent taxes" and the twenty
percent tax accepts being affected, the second tax's base becomes one hundred five point three
five and its amount twenty-one point zero seven, giving a tax total of twenty-one point four two
and a grand total of one hundred twenty-six point four two.

### 4.2 Percentage, price-excluded

*Mirrored.* Evaluated in the **third** pass, walking forward.

```formula
tax_amount = raw_base × tax.amount ÷ 100
```

**Worked example (mandatory example 1).** One unit at one hundred, one tax of twenty-one percent,
price-excluded.

```formula
raw_base      = 1 × 100.00 = 100.00
tax_amount    = 100.00 × 21 ÷ 100 = 21.00
base_amount   = 100.00
untaxed total = 100.00
tax total     = 21.00
grand total   = 121.00
```

### 4.3 Percentage, price-included

*Mirrored.* Evaluated in the **second** pass, walking backwards.

```formula
total_percentage       = batch_percentage( batch )
to_price_excluded      = 1 ÷ ( 1 + total_percentage )    , or 0 when total_percentage = −1
tax_amount             = raw_base × to_price_excluded × tax.amount ÷ 100
```

The guard against a total percentage of exactly minus one hundred percent prevents a division by
zero; the amount is then zero.

**Worked example (mandatory example 2).** One unit at one hundred twenty-one, one tax of twenty-one
percent, price-included.

```formula
raw_base          = 121.00
total_percentage  = 0.21
to_price_excluded = 1 ÷ 1.21 = 0.826446280991735…
tax_amount        = 121.00 × 0.826446280991735 × 0.21 = 21.00
base_amount       = 121.00 − 21.00 = 100.00
untaxed total     = 100.00 ; tax total = 21.00 ; grand total = 121.00
```

**Worked example with a residue.** One unit at twenty-one point five three, one tax of twenty-one
percent, price-included, "round per tax".

```formula
raw tax  = 21.53 ÷ 1.21 × 0.21 = 3.736611570247934…
raw base = 21.53 − 3.736611570247934 = 17.793388429752066…
rounded tax = 3.74 ; rounded base = 17.79 ; 17.79 + 3.74 = 21.53
```

**Two price-included taxes in one batch.** One unit at one hundred, two price-included taxes of ten
percent each in the same batch:

```formula
total_percentage  = 0.20
to_price_excluded = 1 ÷ 1.20 = 0.8333333333…
each tax_amount   = 100 × 0.8333333333 × 0.10 = 8.3333333333…
base              = 100 − ( 8.3333333333 + 8.3333333333 ) = 83.3333333334…
```

Not two successive divisions by one point one, which would have given a different base.

### 4.4 Division ("percentage tax included"), price-excluded

*Mirrored.* Evaluated in the third pass.

```formula
total_percentage        = batch_percentage( batch )
included_base_multiplier = 1                      when total_percentage = 1
                         = 1 − total_percentage   otherwise
tax_amount = raw_base × tax.amount ÷ 100 ÷ included_base_multiplier
```

**Worked example (mandatory example 4).** One unit at one hundred eighty, one division tax of ten
percent, price-excluded.

```formula
total_percentage         = 0.10
included_base_multiplier = 0.90
tax_amount               = 180.00 × 0.10 ÷ 0.90 = 20.00
base_amount              = 180.00
untaxed total            = 180.00 ; tax total = 20.00 ; grand total = 200.00
```

The meaning of a division tax is "the tax is that percentage **of the total including the tax**":
twenty is ten percent of two hundred. The guard for a total percentage of exactly one hundred
percent prevents a division by zero and makes the tax equal to the base.

### 4.5 Division, price-included

*Mirrored.* Evaluated in the second pass.

```formula
tax_amount = raw_base × tax.amount ÷ 100
```

**Worked example.** One unit at two hundred, one division tax of ten percent, price-included.

```formula
tax_amount    = 200.00 × 0.10 = 20.00
base_amount   = 200.00 − 20.00 = 180.00
untaxed total = 180.00 ; tax total = 20.00 ; grand total = 200.00
```

The price-included and the price-excluded forms are exact inverses: one hundred eighty excluded
and two hundred included describe the same transaction.

### 4.6 Custom formula

*Mirrored.* Available only when the Custom Formula Taxes capability is installed. Evaluated in the
**first** pass, alongside fixed taxes, and therefore able to influence a price-included batch.

The formula is a single arithmetic expression evaluated against a context containing exactly five
names:

| Name | Value |
|---|---|
| `price_unit` | The unit price after the discount. |
| `quantity` | The quantity. |
| `base` | The raw base handed to this tax, including any propagated extra base. |
| `product` | A flat table of the product's non-relational field values. |
| `uom` | A flat table of the unit of measure's non-relational field values. |

Grammar, enforced when the tax is saved and again at every evaluation:

1. The text must parse as one expression. Otherwise: *"Invalid formula"*.
2. Before validation, every attribute read of the form `product.<field>` or `uom.<field>` is
   rewritten as an index read `product['<field>']` / `uom['<field>']`, and the field name is
   recorded. Index reads written directly are recorded too.
3. Every recorded field must exist on the corresponding entity and must not be a relation.
   Otherwise: *"Field '<field name>' is not accessible"*.
4. The only permitted syntactic constructs are: the expression itself, a name, a call, an index
   read, a constant, the four arithmetic operators addition, subtraction, multiplication and
   division, the boolean operators "and" and "or", the four comparisons less-than,
   less-than-or-equal, greater-than, greater-than-or-equal, and the unary plus and minus. Anything
   else raises *"Invalid AST node: <construct name>"*.
5. Constants must be integers, decimals or the empty value. Otherwise:
   *"Only int, float or None are allowed as constant values"*.
6. Names must be one of the five listed above, read-only. Otherwise: *"Unknown identifier: <name>"*
   or *"Only read access to identifiers is allowed"*.
7. Calls may only be to the two functions "minimum" and "maximum", and only with positional
   arguments. Otherwise: *"Unknown function call"* or *"Kwargs are not allowed"*.
8. An index read is only allowed on `product` or `uom` and only with a text constant index.
   Otherwise: *"Only product['string'] or uom['string'] read-access is allowed"*.
9. At evaluation, the whole context is round-tripped through a neutral text encoding; if any value
   cannot be encoded, the evaluation raises
   `Only primitive types are allowed in python tax formula context.`
10. A division by zero during evaluation yields zero rather than an error.

The default formula of a new custom-formula tax is the unit price multiplied by one tenth.

Because a custom-formula tax is evaluated in the fixed-amount pass, **it behaves exactly like a
fixed tax with respect to batching, price inclusion and base propagation**: it is computed first
and its result is never divided out of a price-included batch.

### 4.7 Group of taxes

A Group of Taxes never produces an amount of its own. It disappears during flattening (section
3.1) and is replaced by its children; each child's result records the group so that the produced
accounting entry can name it.

**Worked example (mandatory example 5).** One unit at one hundred, one Group of Taxes of sequence
one whose children are a ten percent tax of sequence one and a five percent tax of sequence two,
all price-excluded.

```formula
flattened order  = [ child 10% , child 5% ]
raw_base         = 100.00
child 10% amount = 100.00 × 0.10 = 10.00 , base 100.00
child  5% amount = 100.00 × 0.05 =  5.00 , base 100.00
untaxed total    = 100.00 ; tax total = 15.00 ; grand total = 115.00
```

Two tax accounting entries are produced, one per child, each naming the group as its originator
group. The totals block shows them under their own tax groups, which may or may not be the same.

---

## 5. Propagating one tax's amount into another tax's base

*Mirrored.* This is the most intricate part of the engine. After a tax's amount is known, the
engine may add or subtract that amount from the base of other taxes. Two separate accumulators are
maintained per tax:

- **extra base for tax** — added to the base used to *compute the amount* of that tax. It is only
  updated while that tax's amount is still unknown, because once an amount is computed it must not
  change.
- **extra base for base** — added to the base *reported* for that tax. It is always updated.

Let *T* be the tax whose amount has just been computed, *A* its amount, *before(T)* the taxes that
appear strictly before *T*'s batch in evaluation order, and *after(T)* the taxes that appear
strictly after *T*'s batch in evaluation order.

### 5.1 The complete case table

| Price inclusion of *T* | Special mode | "Affects base" of *T* | Action |
|---|---|---|---|
| included | `false` or `total_included` | yes | For each tax in *after(T)* that does **not** accept being affected: subtract *A*. For each tax in *before(T)*: subtract *A*. |
| included | `false` or `total_included` | no | For each tax in *after(T)*: subtract *A*. For each tax in *before(T)*: subtract *A*. |
| included | `total_excluded` | yes | For each tax in *after(T)* that **does** accept being affected: add *A*. |
| included | `total_excluded` | no | Nothing. |
| excluded | `false` or `total_excluded` | yes | For each tax in *after(T)* that **does** accept being affected: add *A*. |
| excluded | `false` or `total_excluded` | no | Nothing. |
| excluded | `total_included` | yes | For each tax in *before(T)*: subtract *A*. |
| excluded | `total_included` | no | For each tax in *after(T)*: subtract *A*. For each tax in *before(T)*: subtract *A*. |

Note that the price-inclusion tested here is the tax's **own configured** flag, not the effective
flag forced by the special mode.

### 5.2 Why each row exists

- **Included, no special mode, not affecting base.** The tax is inside the price, so every other
  tax must work on a price from which it has been removed. Hence the subtraction on both sides.
- **Included, no special mode, affecting base.** The tax is inside the price, but it is meant to
  swell the base of the taxes that accept it. Those taxes therefore keep it; only the taxes that
  refuse to be affected get it removed, and the earlier taxes always get it removed.
  *Illustration.* A price-excluded fixed tax of one that affects the base, followed by a
  price-included ten percent tax, on a price of one hundred twenty. The fixed tax is computed
  first because it can move the price. The percentage tax is then extracted from one hundred
  twenty plus one, that is one hundred twenty-one. But the fixed tax is itself price-excluded, so
  its own base must be the price with the percentage tax removed.
- **Included, mode `total_excluded`, affecting base.** The caller already handed a price with the
  included taxes removed. To reconstruct the base of the taxes this one is supposed to swell, its
  amount must be added back.
  *Illustration.* A price-included fixed tax of one that affects the base, followed by a
  price-included ten percent tax. With no mode and a price of one hundred twenty-one, the base of
  the second tax is one hundred twenty-one. Under `total_excluded` the caller supplies one hundred
  nine; the first tax's amount of one must be added back before the second tax is computed.
- **Excluded, mode `false` or `total_excluded`, affecting base.** The straightforward cascade: the
  tax is added on top and swells the base of the taxes that accept it.
- **Excluded, mode `total_included`.** The caller handed the grand total, so every tax is being
  extracted. A tax that does **not** affect the base has to be removed from everybody's base; a
  tax that **does** affect the base has already been included in the later taxes' base by
  construction, so only the earlier taxes need it removed.

### 5.3 Worked example (mandatory example 6): a chain using base-amount inclusion

Configuration: tax *A*, ten percent, sequence one, price-excluded, **affects base of subsequent
taxes**; tax *B*, five percent, sequence two, price-excluded, **base affected by previous taxes**.
One unit at one hundred, currency with two decimals, "round per tax".

Batching (walking backwards): *B* opens a batch and sets the "accepts being affected" flag to true.
*A* has the same kind and inclusion but affects the base while the flag is true, so it cannot join.
Two batches of one.

Evaluation:

1. Fixed pass: nothing.
2. Price-included pass: nothing (both are price-excluded).
3. Price-excluded pass, forward order.
   - *A*: raw base one hundred plus extra base zero, amount ten. Propagation: *A* is excluded, no
     special mode, affects base, so for every tax after *A*'s batch that accepts being affected —
     that is *B* — add ten to both accumulators.
   - *B*: raw base one hundred plus extra base ten equals one hundred ten, amount five point five.
4. Base pass, walking backwards.
   - *B*: base equals one hundred plus extra-base-for-base ten equals one hundred ten. *B* does not
     affect the base, so its downstream tax list is empty. *B* accepts being affected, so it is
     pushed on the "subsequent taxes" stack.
   - *A*: base equals one hundred plus zero equals one hundred. *A* affects the base, so its
     downstream tax list is the current stack, that is *B*.
5. Untaxed total is the base of the **first** entry in evaluation order, one hundred. Tax total is
   ten plus five point five equals fifteen point five. Grand total one hundred fifteen point five.

```formula
A: base = 100.00 , amount = 100.00 × 0.10 = 10.00
B: base = 100.00 + 10.00 = 110.00 , amount = 110.00 × 0.05 = 5.50
untaxed = 100.00 , tax = 15.50 , total = 115.50
```

The downstream tax list matters for accounting: the tax entry produced by *A* carries *B* as one of
its own base taxes, so that *B* is re-applied to *A*'s amount if the entry is ever recomputed, and
it also inherits *B*'s base report tags (section 8.4).

**The same chain with *A* price-included.** Price one hundred ten instead of one hundred.

1. Price-included pass, backwards: *A* is reached (its batch is the only price-included one). Raw
   base one hundred ten, single-member batch, total percentage zero point one, factor one divided
   by one point one, amount ten. Propagation: included, no special mode, affects base, so for each
   tax after *A*'s batch that does **not** accept being affected — none — subtract; and for each
   tax before *A*'s batch — none — subtract. Nothing happens.
2. Price-excluded pass, forward: *B*, raw base one hundred ten plus extra zero, amount five point
   five.
3. Base pass: *B*'s base is one hundred ten; *A*'s base is one hundred ten minus its own batch
   total of ten, that is one hundred.
4. Untaxed total is the base of the first entry, *A*, that is one hundred. Tax total fifteen point
   five, grand total one hundred fifteen point five — identical to the price-excluded variant, as
   expected.

---

## 6. Computing one base line

*Mirrored.* Inputs: the taxes, the unit price, the quantity, the currency step, the rounding
method, the product, the unit of measure, the special mode and the optional filter.

1. **Flatten, filter and batch** (section 3).
2. **Initialise one record per tax** with: the tax, the effective price-inclusion flag, two
   zeroed extra-base accumulators, the parent group and the batch. The effective price-inclusion
   flag is:
   - `false` when the tax has a negative distribution factor (a reverse-charge tax is always
     treated as price-excluded, whatever its own flag says);
   - otherwise `true` under mode `total_included`, `false` under mode `total_excluded`;
   - otherwise the tax's own price-inclusion flag.
   For a tax with a negative distribution factor, a **second, mirror record** is created, marked
   as the reverse-charge half.
3. **Compute the raw base**: quantity multiplied by the unit price (the price already carries the
   discount, see section 6.1). Under "round per line", round it to the currency step immediately.
4. **First pass — fixed and custom formula**, walking the flattened list **backwards**. For each
   tax whose amount is not yet known, evaluate the fixed-amount rule (section 4.1) or the
   custom-formula rule (section 4.6). Only those two kinds return a value here; the others return
   nothing and are skipped.
5. **Second pass — price-included**, walking **backwards**. For each tax whose effective
   price-inclusion flag is true and whose amount is not yet known, evaluate section 4.3 or 4.5.
6. **Third pass — price-excluded**, walking **forwards**. For each tax whose effective
   price-inclusion flag is false and whose amount is not yet known, evaluate section 4.2 or 4.4.
7. Whenever an amount is produced: under "round per line", round it to the currency step
   immediately; mirror it negated onto the reverse-charge half if there is one; then run the
   propagation of section 5.
8. **Base pass**, walking **backwards**, maintaining a stack of "subsequent taxes" that is
   initially empty. For each tax whose amount is known:
   ```formula
   batch_total = sum of the amounts of every tax in this tax's batch
               + sum of the reverse-charge halves of those of them that have one
   base        = raw_base + extra_base_for_base
   base        = base − batch_total      when the effective price-inclusion flag is true
                                          and the special mode is false or 'total_included'
   ```
   Record the base. The tax's downstream tax list is the current stack if the tax affects the base
   of subsequent taxes, empty otherwise. Copy both onto the reverse-charge half. If the tax accepts
   being affected by previous taxes, push it on the stack.
9. **Assemble the result list** in the order the records were created (that is, the flattened
   order), inserting each reverse-charge half immediately after its positive half.
10. **Totals**:
    ```formula
    total_excluded = base of the first entry of the result list
    total_included = total_excluded + sum of every entry's tax amount
    ```
    When the result list is empty, both totals equal the raw base.

The choice of "the base of the first entry" rather than "the raw base" is what makes price-included
taxes work: the first entry's base has already had the included taxes removed.

### 6.1 The discount

```formula
price_unit_after_discount = price_unit × ( 1 − discount ÷ 100 )
```

The discount is applied **before** the engine is called; the engine only sees the discounted price.
No rounding is applied to the discounted price: the multiplication by the quantity happens on the
full-precision value.

### 6.2 The two currencies

*Mirrored.* Immediately after the single-line computation, every amount is duplicated into the
company currency using the base line's rate. The rate is expressed as *units of line currency per
unit of company currency*, so the conversion is a **division**.

```formula
amount_in_company_currency = amount_in_line_currency ÷ rate      , or 0 when rate = 0
```

Under "round per line" each converted amount is immediately rounded to the company currency. Under
"round per tax" they are left raw and rounded later by section 7.

The four totals and, for each tax entry, the base and the tax amount are all duplicated this way.

### 6.3 Non-deductible lines

*Server-only.* When the base line is marked with the special type `non_deductible`, every
reverse-charge half is dropped from the result list and its amount is subtracted from the total
with taxes. This is what lets a partially deductible purchase show only the non-recoverable share.

---

## 7. Document-wide rounding

*Mirrored.* Rounding one line at a time and adding up is not the same as adding up and rounding
once. The engine therefore computes everything unrounded, then performs four passes that both
round and redistribute the difference, so that every visible subtotal is consistent with every
visible total.

Everything in this section operates on a **list** of base lines that belong to the same document.

### 7.1 Smooth distribution of a delta

*Mirrored.* Given a delta to distribute and a list of weights, allocate the delta in whole units of
the last decimal place, proportionally to the weights, biggest weight first.

1. If the list of weights is empty, return an empty allocation.
2. Let the step be ten to the power minus the number of decimal places. If the delta is zero at
   that precision (section 2.2), return an all-zero allocation.
3. Let the sign be minus one when the delta is negative, plus one otherwise. Let the number of
   units be the nearest integer to the absolute delta divided by the step. Let the remaining units
   equal that number.
4. **Normalise the weights**: take the absolute value of each weight, keep its original position,
   sort by absolute value descending, and divide each by the sum of all absolute values. When the
   sum is zero, every normalised weight is one divided by the number of weights.
5. Walk the normalised weights in that sorted order. For each, allocate the smaller of
   (the nearest integer to the normalised weight multiplied by the number of units) and
   (the remaining units); subtract what was allocated from the remaining units; add
   sign × allocated × step to the allocation at that weight's original position. Stop early when
   nothing remains.
6. Distribute the units still remaining one at a time, in the same sorted order, one unit to each.
   Because the weights are normalised, the leftover can never exceed the number of weights.

**Worked example.** Three weights of zero point four, zero point three and zero point three, a
delta of three hundredths, three decimal places. The step is one thousandth, so there are thirty
units to place. Sorted order is the first weight, then the two others. Allocation: twelve units to
the first (zero point zero one two), nine to each of the others (zero point zero zero nine). Total
thirty. Result: twelve thousandths, nine thousandths, nine thousandths.

**Worked example with a leftover.** Two equal weights, a delta of one hundredth, two decimal
places. One unit to place; normalised weights one half each; nearest integer to one half of one is
zero (the nearest-integer rule used here is the language's, which for exactly one half rounds to
the even neighbour, hence zero); so the loop allocates nothing and one unit remains; step six gives
it to the first weight in sorted order. Result: one hundredth to the first, nothing to the second.

### 7.2 Pass one — plain rounding

For each base line and for each of the two currencies:

```formula
total_excluded            = round_to_currency( raw_total_excluded )
per tax entry: base_amount = round_to_currency( raw_base_amount )
per tax entry: tax_amount  = round_to_currency( raw_tax_amount )
```

### 7.3 Pass two — apply manual amounts

For each base line, for each of the two currencies:

- If a manual untaxed total is set for that currency, the rounded untaxed total is **replaced** by
  it. When the manual value was given in the line currency and a rate exists, the company-currency
  untaxed total is recomputed as the manual value divided by the rate, rounded to the company
  currency.
- For each tax entry, for the base and for the tax amount independently: if a manual value exists
  for that tax, the rounded amount is replaced by that value multiplied by the reverse-charge sign
  (minus one on a reverse-charge half, plus one otherwise) and rounded to the currency. When the
  manual value was given in the line currency and a rate exists, the company-currency amount is
  recomputed as that value divided by the rate, rounded to the company currency.

### 7.4 Pass three — totals with taxes and zeroed deltas

For each base line and each currency:

```formula
delta_total_excluded = 0
total_included       = total_excluded + sum of the rounded tax amounts of every tax entry
```

### 7.5 Pass four — redistribute the tax-amount and base-amount differences

Group every tax entry of every base line by the key

```
( tax , currency , refund flag , reverse-charge flag , effective price inclusion , computation key )
```

For each group and for each of the two currencies:

1. Let the **target tax amount** be the sum over the group of: the manual tax amount if one is set
   for that tax on that line, otherwise the raw tax amount. (For the company currency, when only a
   line-currency manual amount was set, the already-rounded company amount is used instead.)
2. Compute
   ```formula
   delta_tax = round_to_currency( target_tax_amount ) − sum_of_rounded_tax_amounts
   ```
   If that delta is not zero at the currency's precision, distribute it (section 7.1) over the
   group's tax entries, weighted by each entry's **raw** tax amount, and add each allocation to the
   corresponding rounded tax amount.
3. Let the **target base amount** be the analogous sum for bases. Then:
   - in *included* mode — that is, when the mode is "mixed" and the group is price-included, or
     when the mode is explicitly "included":
     ```formula
     delta_base = round_to_currency( target_base_amount + target_tax_amount )
                  − ( sum_of_rounded_base_amounts + sum_of_rounded_tax_amounts + delta_tax )
     ```
   - in *excluded* mode:
     ```formula
     delta_base = round_to_currency( target_base_amount ) − sum_of_rounded_base_amounts
     ```
   If that delta is not zero, distribute it over the group's tax entries weighted by each entry's
   **raw** base amount, and add each allocation to the corresponding rounded base amount.

The engine always calls this pass in "mixed" mode.

### 7.6 Pass five — redistribute the untaxed-total difference

Group every base line by the key `( currency , refund flag , computation key )`. For each group:

1. Decide the mode. In "mixed" mode, start from *included* and switch to *excluded* as soon as one
   base line in the group carries a tax entry that is **not** price-included **and** whose tax
   amount is non-zero in at least one of the two currencies.
2. For each of the two currencies:
   - *excluded* mode:
     ```formula
     delta = round_to_currency( target_total_excluded ) − sum_of_rounded_totals_excluded
     ```
     with the weights being each line's **raw** untaxed total. Skip the group entirely when the
     target is exactly zero.
   - *included* mode:
     ```formula
     delta = round_to_currency( target_total_excluded + target_tax_amount )
             − ( sum_of_rounded_totals_excluded + sum_of_rounded_tax_amounts )
     ```
     with the weights being each line's **raw** total with taxes. Skip the group when the target
     total with taxes is exactly zero.
3. Distribute the delta (section 7.1) and add each allocation to that line's
   `delta_total_excluded` for that currency.

This is the only pass that writes the delta field. The accounting balance of a base line is always
its rounded untaxed total **plus** this delta.

### 7.7 Pass six — realign on existing tax entries

*Server-only.* When the caller supplied existing tax lines (section 1.4) — which happens whenever
an already-saved document is recomputed and the user may have typed a tax amount by hand — the
totals are pulled back onto those existing amounts.

1. Total the supplied tax lines by the key `( tax , currency , the distribution line is a refund one )`,
   summing the sign multiplied by the amount, separately in each currency.
2. Group the computed tax entries by the key `( tax , currency , refund flag )`.
3. For each group that has a counterpart in step 1 and whose current total is non-zero, compute
   `delta = supplied total − current total` and distribute it (section 7.1) over the group's tax
   entries weighted by their **rounded** tax amounts.

### 7.8 Worked example (mandatory example 7): round per line against round per tax

A document with three identical lines: quantity twelve point one two, unit price twelve point one
two, one price-excluded tax of twenty-three percent. Currency with two decimal places. Company
currency equals document currency, so the rate is one.

**Round per line.** The raw base of each line is rounded before anything else:

```formula
raw_base per line  = round( 12.12 × 12.12 ) = round( 146.8944 ) = 146.89
tax per line       = round( 146.89 × 0.23 ) = round( 33.7847 )  = 33.78
```

Pass four finds a target tax amount of three times thirty-three point seven eight — the raw amounts
are already rounded — equal to one hundred one point three four, which matches the sum of the
rounded amounts: no delta. Pass six likewise finds no delta.

| Line | Untaxed | Tax | Total |
|---|---|---|---|
| 1 | 146.89 | 33.78 | 180.67 |
| 2 | 146.89 | 33.78 | 180.67 |
| 3 | 146.89 | 33.78 | 180.67 |
| **Document** | **440.67** | **101.34** | **542.01** |

**Round per tax.** Nothing is rounded during the single-line computation:

```formula
raw_base per line = 12.12 × 12.12 = 146.8944
raw_tax per line  = 146.8944 × 0.23 = 33.785712
```

Pass two rounds each to one hundred forty-six point eight nine and thirty-three point seven nine.
Pass four, tax amounts: the target is three times thirty-three point seven eight five seven one
two equals one hundred one point three five seven one three six, rounded to one hundred one point
three six; the sum of the rounded amounts is one hundred one point three seven; the delta is minus
one hundredth. The three weights are equal, so the sorted order is the original order and the
single unit goes to the first entry: line one's tax becomes thirty-three point seven eight.
Pass four, base amounts, excluded mode: the target is three times one hundred forty-six point eight
nine four four equals four hundred forty point six eight three two, rounded to four hundred forty
point six eight; the sum of the rounded bases is four hundred forty point six seven; the delta is
plus one hundredth, allocated to line one, whose reported base becomes one hundred forty-six point
nine zero.
Pass six, excluded mode (the tax is not price-included and its amount is non-zero): the same delta
of plus one hundredth is allocated to line one's untaxed delta.

| Line | Rounded untaxed | Delta | Balance | Tax | Total |
|---|---|---|---|---|---|
| 1 | 146.89 | +0.01 | 146.90 | 33.78 | 180.68 |
| 2 | 146.89 | 0.00 | 146.89 | 33.79 | 180.68 |
| 3 | 146.89 | 0.00 | 146.89 | 33.79 | 180.68 |
| **Document** | | | **440.68** | **101.36** | **542.04** |

**The difference to the cent.** Untaxed four hundred forty point six seven against four hundred
forty point six eight (one hundredth); tax one hundred one point three four against one hundred one
point three six (two hundredths); grand total five hundred forty-two point zero one against five
hundred forty-two point zero four (three hundredths). Under "round per tax" the document total is
the correctly rounded value of the exact arithmetic; under "round per line" it is the sum of three
independently rounded line totals.

**The price-included variant.** Three lines of one unit at twenty-one point five three with a
twenty-one percent price-included tax:

- Round per line: each line yields untaxed seventeen point seven nine and tax three point seven
  four; document untaxed fifty-three point three seven, tax eleven point two two, total sixty-four
  point five nine.
- Round per tax: the raw untaxed per line is seventeen point seven nine three three eight eight
  four two nine and the raw tax three point seven three six six one one five seven. Pass four, tax
  amounts: target eleven point two zero nine eight three four seven, rounded eleven point two one;
  sum of rounded eleven point two two; delta minus one hundredth to line one, whose tax becomes
  three point seven three. Pass four, base amounts, included mode: target base plus target tax
  equals sixty-four point five nine, rounded sixty-four point five nine; current sum is fifty-three
  point three seven plus eleven point two two plus the delta of minus one hundredth equals
  sixty-four point five eight; delta plus one hundredth to line one, whose reported base becomes
  seventeen point eight zero. Pass six, included mode: the same delta of plus one hundredth lands
  on line one's untaxed delta.

| Line | Balance | Tax | Total |
|---|---|---|---|
| 1 | 17.80 | 3.73 | 21.53 |
| 2 | 17.79 | 3.74 | 21.53 |
| 3 | 17.79 | 3.74 | 21.53 |
| **Document** | **53.38** | **11.21** | **64.59** |

Both methods reach the same grand total here — a price-included tax pins it — but they disagree on
the split: fifty-three point three seven against fifty-three point three eight untaxed, and eleven
point two two against eleven point two one of tax. The document totals under "round per tax" are
globally correct: fifty-three point three eight multiplied by zero point two one is eleven point
two one nine eight, which rounds to the reported eleven point two one to within the same cent.

### 7.9 Multi-currency worked example

Three lines of one unit at twenty-one point five three in a foreign currency, one price-excluded
tax of twenty percent, a rate of one point two five foreign units per company unit, "round per
tax".

```formula
raw untaxed per line, foreign  = 21.53
raw untaxed per line, company  = 21.53 ÷ 1.25 = 17.224
raw tax per line, foreign      = 21.53 × 0.20 = 4.306
raw tax per line, company      = 4.306 ÷ 1.25 = 3.4448
```

Foreign currency: target tax twelve point nine one eight rounds to twelve point nine two; the sum
of three rounded four point three one is twelve point nine three; delta minus one hundredth to line
one, whose tax becomes four point three zero. Target untaxed sixty-four point five nine equals the
sum of the rounded values, so no untaxed delta.

Company currency: target tax ten point three three four four rounds to ten point three three; the
sum of three rounded three point four four is ten point three two; delta plus one hundredth to line
one, whose company tax becomes three point four five. Target untaxed fifty-one point six seven two
rounds to fifty-one point six seven; the sum of three rounded seventeen point two two is fifty-one
point six six; delta plus one hundredth to line one's company untaxed delta.

| Line | Foreign balance | Foreign tax | Company balance | Company tax |
|---|---|---|---|---|
| 1 | 21.53 | 4.30 | 17.23 | 3.45 |
| 2 | 21.53 | 4.31 | 17.22 | 3.44 |
| 3 | 21.53 | 4.31 | 17.22 | 3.44 |
| **Document** | **64.59** | **12.92** | **51.67** | **10.33** |

The two currencies are rounded and redistributed independently; the allocations do not have to land
on the same line.

---

## 8. Deriving the accounting data

*Server-only.* After the amounts are final, the engine turns them into the data an accounting entry
needs: which distribution line produced what, on which account, with which report tags, and under
which grouping key.

### 8.1 Choosing the distribution

```formula
distribution_used =  tax.refund_repartition_line_ids     when base_line.is_refund is true
                     tax.invoice_repartition_line_ids    otherwise
```

The refund flag of a base line coming from a document is set as follows:

1. When the document kind is a customer credit note or a vendor refund, the flag is **true**.
2. When the document kind is a miscellaneous entry:
   - on a line that **is** a tax entry, the flag is true when its distribution line's document kind
     is `refund`;
   - otherwise, when the line's taxes contain both a sales tax and a purchase tax, the flag is
     true when the line has no credit (that is, the line is a debit or zero);
   - otherwise, taking the first tax only: the flag is true when that tax is a sales tax and the
     line has no credit, or when that tax is a purchase tax and the line has no debit;
   - and finally, if the line carries taxes and the entry is itself the reversal of another entry,
     the flag is **inverted**.
3. In every other case the flag is false.

### 8.2 Splitting the amount over the distribution lines

For each tax entry:

1. Select the distribution lines of kind `tax` from the chosen distribution:
   - for the **positive** half, those whose factor is greater than or equal to zero, with a sign of
     plus one;
   - for the **reverse-charge** half, those whose factor is strictly negative, with a sign of minus
     one.
2. For each selected distribution line:
   ```formula
   share_in_line_currency   = round_to_line_currency( tax_amount_in_line_currency  × factor × sign )
   share_in_company_currency= round_to_company_currency( tax_amount_in_company_currency × factor × sign )
   account                  = target account of the distribution line (entities, section 2.5),
                              falling back to the base line's own account
   ```
3. Accumulate the shares. Because each share is rounded independently, the accumulated total may
   differ from the tax amount by a unit of the last decimal place. The difference is therefore
   redistributed: sort the shares by decreasing absolute amount in the line currency, then by
   decreasing absolute amount in the company currency; compute
   `delta = tax_amount − accumulated total`; distribute it with the smooth allocation of section
   7.1 weighted by each share's own amount; add the allocations back.

The sign convention means that a distribution of plus one hundred and minus one hundred produces
two shares: `+tax_amount` for the positive line and `−(−tax_amount)` — that is `+tax_amount` — on
the reverse-charge half, because the reverse-charge half already carries the negated amount. The
net effect on the ledger is zero while both report tags are stamped.

### 8.3 Tags on the base entry

The base entry's tag set is built as:

1. The product's own account tags (only those attached to the product), if there is a product.
2. Plus, for every tax entry that is **not** a reverse-charge half and whose tax is exigible on
   the invoice — or, when the caller explicitly asked for cash basis tags to be included, for every
   such tax regardless of exigibility — the tags of the `base` distribution line of the chosen
   distribution.

### 8.4 Tags on a tax entry

Walking the tax entries **backwards** and maintaining, per tax, the set of base tags of the taxes
seen so far:

1. Start the tax entry's tag set with the product's account tags.
2. Add the distribution line's own tags, but only when the tax is exigible on the invoice or the
   caller asked for cash basis tags to be included.
3. If the tax affects the base of subsequent taxes, also add the **base tags of every other tax**
   already recorded in the running set. This is what makes a tax that swells the base of a later
   tax report its own amount inside that later tax's base grid.
4. After processing the tax's entries, if the tax accepts being affected by previous taxes, record
   its own base tags in the running set (again, only when exigible on invoice or when cash basis
   tags were requested).

### 8.5 Grouping keys

Two tax entries are merged into one accounting entry when they share a grouping key.

**Base part of the key** (from the base line):

```
( partner , currency , analytic distribution , account , the set of base taxes )
```

**Full key of a tax entry**: the base part, overridden and extended with

```
( distribution line , partner , currency , originator group of taxes ,
  analytic distribution — kept only when the tax is analytic or the distribution line is not used
  in the tax settlement, otherwise cleared ,
  account of the distribution line, falling back to the base line's account ,
  the set of downstream taxes the entry itself is subject to ,
  the set of report tags ,
  a technical "keep zero line" marker, false by default )
```

**Key of an existing tax entry** (used to decide whether it can be updated instead of recreated):

```
( distribution line , partner , currency , originator group of taxes ,
  analytic distribution , account , set of base taxes , set of report tags )
```

The two keys are deliberately built from the same fields in the same order so that they can be
compared directly.

### 8.6 Producing the accounting entries

1. For each base line, record the update it needs: its tag set, and its balance in each currency
   computed as
   ```formula
   amount_in_line_currency    = sign × ( total_excluded_currency + delta_total_excluded_currency )
   amount_in_company_currency = sign × ( total_excluded         + delta_total_excluded )
   ```
2. For each tax entry share, accumulate under its grouping key:
   ```formula
   name            = the manual tax line name if the base line carries one, else the tax name
   tax_base_amount = tax_base_amount + sign × the tax entry's base amount in company currency
   amount_currency = amount_currency + sign × the share in line currency
   balance         = balance         + sign × the share in company currency
   ```
3. Drop every accumulated entry whose amount is zero in **both** currencies, unless its key carries
   the "keep zero line" marker. Then strip every technical key (those whose name begins with a
   double underscore) from the keys.
4. Match against the supplied existing tax entries: an existing entry whose key is present in the
   accumulation, and which has not already been matched, is scheduled for **update** with the
   accumulated amounts; every other existing entry is scheduled for **deletion**; every
   accumulated key left unmatched is scheduled for **creation**.

### 8.7 Worked example (mandatory example 10): a refund and its tag signs

Configuration. A sales tax of twenty-one percent whose invoice distribution is: base line tagged
`+base 21`, tax line of one hundred percent on the tax account tagged `+tax 21`. Its refund
distribution is: base line tagged `-base 21`, tax line of one hundred percent on the same tax
account tagged `-tax 21`. The four tags are the tags created by four report expressions whose
formulas are, respectively, `base 21`, `tax 21`, `-base 21` and `-tax 21` — in practice the same
two tags are reused with the minus prefix, so the report negates them.

**The invoice.** One customer invoice line, one unit at one thousand, sign minus one (a customer
invoice's product line is a credit).

| Entry | Account | Amount in line currency | Report tags |
|---|---|---|---|
| Base | revenue | −1 000.00 (credit) | `base 21` |
| Tax | tax payable | −210.00 (credit) | `tax 21` |
| Counterpart | receivable | +1 210.00 (debit) | none |

The tax return line whose expression is `base 21` sums the balances carrying that tag: minus one
thousand. The line whose expression is `tax 21` sums minus two hundred ten. Because a sales tax
return is conventionally presented as a positive figure, the shipped report expressions for a sales
grid use the **negating** form: their formula is written with a leading minus sign, the tag's
"negate balance" flag is therefore true, and the reported figures are plus one thousand and plus
two hundred ten.

**The credit note for the whole invoice.** The same line, but the refund flag is true and the sign
is plus one.

| Entry | Account | Amount in line currency | Report tags |
|---|---|---|---|
| Base | revenue | +1 000.00 (debit) | `-base 21` → the tag named `base 21` with negation |
| Tax | tax payable | +210.00 (debit) | `-tax 21` → the tag named `tax 21` with negation |
| Counterpart | receivable | −1 210.00 (credit) | none |

Two independent sign flips happen and they cancel:

1. The **balance** flips because the document is a refund: plus one thousand instead of minus one
   thousand.
2. The **tag** changes to the refund distribution's tag, which in the shipped charts is the same
   tag name carrying the opposite report sign.

```formula
reported_amount_on_the_grid = report_sign × sum of the balances of the tagged journal items
where report_sign = −1 when the expression's formula starts with a minus sign, +1 otherwise
```

Invoice contribution to the sales base grid: report sign minus one times minus one thousand equals
plus one thousand. Credit note contribution: report sign plus one times plus one thousand equals
plus one thousand — **the wrong direction**, unless the refund distribution points at the
oppositely signed expression, which is exactly why the refund distribution exists as a separate,
independently taggable list. With the shipped configuration the credit note points at the
expression whose formula is the negated one, so its contribution is minus one thousand and the
period's net base is zero.

**The rule to implement.** The refund distribution is not a sign convention applied by the engine;
it is a second, fully independent mapping. The engine's only refund-specific behaviour is: pick
the refund list instead of the invoice list. Everything else — which account, which factor, which
tag, and therefore which sign appears on the return — is data.

**Consistency guard.** The two lists must have the same length, the same kinds in the same order
and the same percentages (entities, section 2.4), so that a refund can never redistribute an amount
differently from the invoice it reverses. Only the accounts and the tags may differ.

### 8.8 Re-deriving the tags of existing entries

*Server-only.* A dedicated maintenance operation rebuilds the tag links of already-posted journal
items after the distribution of a tax has been edited. It is specified in `workflows.md` section 9;
its selection rule for the distribution to use on a miscellaneous entry is:

| Tax type | Line balance | Distribution used |
|---|---|---|
| sales | less than or equal to zero | invoice |
| sales | greater than zero | refund |
| purchase | greater than or equal to zero | invoice |
| purchase | less than zero | refund |

For invoice-kind documents the invoice distribution is used, for refund-kind documents the refund
distribution, and for a cash basis entry the kind of its originating document decides.

---

## 9. Manual amounts, computation keys and stored extra data

### 9.1 Manual amounts

Three optional inputs let a caller pin the result instead of letting the engine derive it:

| Input | Effect |
|---|---|
| `manual_total_excluded_currency` | Replaces the rounded untaxed total in the line currency (pass three of section 7). |
| `manual_total_excluded` | Replaces the rounded untaxed total in the company currency. |
| `manual_tax_amounts` | A table keyed by tax identifier; each entry may carry any of `base_amount_currency`, `base_amount`, `tax_amount_currency`, `tax_amount`, each replacing the corresponding rounded amount. |

Manual amounts do not change the *raw* amounts, so the "target" amounts used by the redistribution
passes of sections 7.5 and 7.6 fall back to the manual values when they exist and to the raw values
otherwise. That is what keeps the document total exactly on the pinned figure.

A helper "freeze the current result" writes the current rounded amounts into the manual inputs:
for each base line with at least one tax entry, the manual untaxed totals become the rounded totals
plus their deltas, and the manual per-tax amounts become the current rounded base and tax amounts —
skipping reverse-charge halves, and, when a filter is supplied, writing an empty entry (rather than
no entry) for the taxes the filter rejects.

### 9.2 Computation keys

A computation key partitions one document into independent rounding subsets. Both redistribution
passes (sections 7.5 and 7.6) include the key in their grouping, so lines carrying different keys
never lend cents to each other.

The canonical use is a final invoice that deducts a previous down payment: the original lines carry
no key and total one thousand, while the negative down-payment lines carry the key `down_payment`
and total minus three hundred. Each subset is rounded correctly on its own, so the invoice shows
exactly seven hundred and the down payment is neither inflated nor deflated by a stray cent.

Keys in use: `global_discount` and `down_payment`, each optionally suffixed. A base line whose
stored key begins with `global_discount` is additionally marked with the special type
`global_discount`; one beginning with `down_payment` is marked `down_payment`.

### 9.3 Stored extra tax data

*Mirrored.* Manual amounts and computation keys survive a save in a structured document stored on
the journal item.

**Export.** The stored document contains the computation key when there is one. When at least one
manual value exists, it also contains those manual values **and** a snapshot of the inputs they
were captured against: currency, unit price, discount, quantity and rate.

**Import.** The computation key is always restored. The manual values are restored only when
**every** one of the following holds:

1. a stored manual per-tax table exists;
2. the base line's currency is the stored currency;
3. the base line's unit price equals the stored unit price, compared at the currency's precision;
4. the base line's discount equals the stored discount, compared at the currency's precision;
5. the base line's quantity equals the stored quantity, compared at the currency's precision;
6. the number of flattened taxes on the base line equals the number of taxes in the stored table;
7. every flattened tax of the base line is present in the stored table.

When they are restored, the stored unit price replaces the base line's unit price, and every
company-currency amount is re-expressed for the current rate:

```formula
rate_change = current_rate ÷ stored_rate            , or 1 when either rate is absent
restored_company_amount = stored_company_amount ÷ rate_change
```

Line-currency amounts are restored unchanged.

**Reversal.** When a document is reversed, the stored data is deep-copied and the quantity, both
manual untaxed totals and every manual base and tax amount (in both currencies) are negated.

### 9.4 Turning a refund base line back into a normal one

*Server-only.* A helper produces, from a base line marked as a refund, an equivalent line that is
not: the quantity is negated, the refund flag cleared, and every amount in the tax details — the
four totals, both deltas, and each tax entry's four base and tax amounts, raw and rounded — is
negated. This is used where a consumer needs all lines pointing the same way regardless of document
kind.

---

## 10. The totals block

*Mirrored (except the non-deductible part, which is server-only).* The totals block is the
structured value a document exposes so that a screen or a printed page can show "untaxed amount",
one line per tax group, optional intermediate subtotals, the cash rounding delta and the grand
total — in the document currency and, optionally, in the company currency.

### 10.1 Inputs and output shape

Inputs: the rounded base lines, the document currency, the company, and an optional cash rounding
configuration.

Output keys: the two currencies and their rounding steps; a flag saying whether any tax group is
involved; a flag saying whether every tax group shares the same displayed base; the untaxed amount,
the tax amount and the grand total in both currencies; the cash rounding delta in both currencies
when there is one; and an ordered list of subtotals, each with a name, a base amount, a tax amount
and an ordered list of tax groups. Each tax group carries its identifier, its name, its
point-of-sale label, the identifiers of the taxes aggregated into it, its base and tax amounts in
both currencies, the base amount **to display** in both currencies, and, when applicable, the
non-deductible tax amount in both currencies.

### 10.2 Algorithm

1. **Global amounts.** Aggregate every base line under a single key that is present only when the
   line has at least one tax entry. Sum, over all keys: the untaxed totals including their deltas
   into the block's untaxed amount, and the tax amounts into the block's tax amount, in both
   currencies. Set "has tax groups" when at least one key was non-empty.
2. **Per tax group.** Aggregate every base line by the tax group of each tax entry. Sort the
   resulting groups by (group sequence, group identifier).
3. For each group in that order:
   a. Collect the distinct taxes that contributed to it.
   b. **Displayed base.**
      - If every contributing tax is of the fixed kind, there is **no** displayed base (the value
        is explicitly absent, and the group shows no base at all).
      - Else if every contributing tax is of the division kind **and** every one of them is
        price-included, the displayed base is, summed over the contributing base lines, the untaxed
        total plus its delta plus the tax amount of every division tax entry of that line. (For a
        price-included division tax the meaningful base to show the customer is the tax-inclusive
        amount.)
      - Otherwise the displayed base is the group's plain base amount.
   c. If a displayed base exists, record its textual representation at the currency's precision in
      a set; that set is later used for the "same base" flag.
   d. Determine the subtotal the group belongs to: the group's "preceding subtotal" label when set,
      otherwise the label *"Untaxed Amount"*. The first group that names a given subtotal fixes
      that subtotal's position in the output order.
   e. Append the group to that subtotal.
4. **Subtotals.** If no group at all was produced, create the single subtotal *"Untaxed Amount"*
   with no groups. Order the subtotals by the position recorded in step 3d. Then walk them in
   order, keeping a running accumulated tax amount that starts at zero:
   ```formula
   subtotal.base_amount = block.base_amount + accumulated_tax_amount
   for each group in the subtotal:
       subtotal.tax_amount      = subtotal.tax_amount      + group.tax_amount
       accumulated_tax_amount   = accumulated_tax_amount   + group.tax_amount
   ```
   This is what makes a "preceding subtotal" behave like a cascading base: the second subtotal's
   base is the untaxed amount plus the taxes of the first subtotal.
5. **Cash rounding** — section 10.3.
6. **Subtract the cash rounding from the untaxed amounts.** The cash rounding delta (zero when
   there is none) is subtracted from the block's untaxed amount and from every subtotal's base
   amount, in both currencies. Then the block's untaxed amount, as text at the currency's
   precision, is added to the set of encountered bases, and the "same base" flag is set when that
   set has exactly one member.
7. **Non-deductible amounts** — section 10.4.
8. **Grand total.**
   ```formula
   total_amount = base_amount + tax_amount + cash_rounding_delta      (per currency)
   ```

### 10.3 Cash rounding inside the totals block

Two situations:

**A. Cash rounding lines already exist among the base lines** (the document has been saved and the
rounding line materialised). The block's cash rounding delta is the sum of those lines' rounded
untaxed totals, in both currencies. Nothing else is computed.

**B. No cash rounding line exists but a cash rounding configuration was supplied.** Let the
configuration provide a rounding step, a rounding method and a strategy.

```formula
total_in_line_currency     = round_to_line_currency( base_amount_currency + tax_amount_currency )
total_in_company_currency  = round_to_company_currency( base_amount + tax_amount )
target_total               = round_to_step( total_in_line_currency , cash_rounding_step , cash_rounding_method )
delta_in_line_currency     = target_total − total_in_line_currency
rate                       = | total_in_line_currency ÷ total_in_company_currency |   , 0 when the company total is 0
delta_in_company_currency  = round_to_company_currency( delta_in_line_currency ÷ rate )   , 0 when rate = 0
```

The two totals are rounded **first** so that a sub-unit floating residue cannot be magnified by the
step rounding.

If the delta is zero at the currency's precision, nothing happens. Otherwise:

- **Strategy "add a rounding line".** The delta is recorded as the block's cash rounding delta and
  added to the block's untaxed amount and to the *"Untaxed Amount"* subtotal's base amount, in both
  currencies. (Step 6 then subtracts it again from the displayed untaxed amounts, so that the
  rounding appears as its own figure rather than distorting the net.)
- **Strategy "adjust the biggest tax".** Among every (subtotal, tax group) pair, pick the one whose
  tax amount in the line currency is the largest. Add the delta to that group's tax amount, to that
  subtotal's tax amount and to the block's tax amount, in both currencies. If there is no tax group
  at all, the cash rounding silently does nothing and the delta is reset to zero.

### 10.4 Non-deductible amounts

*Server-only.* When at least one base line is marked with the special type `non_deductible` **and**
carries taxes, those lines are aggregated by tax group and, for every group of every subtotal:

```formula
group.non_deductible_tax_amount = non_deductible tax amount of that group
group.tax_amount    = group.tax_amount    − non_deductible tax amount
group.base_amount   = group.base_amount   − non_deductible base amount
subtotal.tax_amount = subtotal.tax_amount − non_deductible tax amount
block.tax_amount    = block.tax_amount    − non_deductible tax amount
```

### 10.5 Excluding tax groups from the block

A post-processing helper folds selected tax groups into the base: for each excluded group, its tax
amount is added to the enclosing subtotal's base amount and to the block's base amount and
subtracted from both tax amounts; the group itself disappears; a subtotal left with no group at all
disappears too. Localizations use it to present a tax as part of the price.

### 10.6 Worked example: two subtotals

Two lines, each one unit at one hundred. Line one carries a tax *X* of ten percent whose tax group
has no preceding subtotal. Line two carries a tax *Y* of five percent whose tax group has the
preceding subtotal label "Total excluding surcharge".

- Block untaxed amount: two hundred. Block tax amount: fifteen.
- Group of *X* is met first (lower sequence), so the subtotal *"Untaxed Amount"* is registered at
  position zero; group of *Y* registers "Total excluding surcharge" at position one.
- Walking the subtotals:
  - *"Untaxed Amount"*: base two hundred plus accumulated zero equals two hundred; tax ten;
    accumulated becomes ten.
  - *"Total excluding surcharge"*: base two hundred plus accumulated ten equals two hundred ten;
    tax five; accumulated becomes fifteen.
- Grand total: two hundred plus fifteen equals two hundred fifteen.

### 10.7 Worked example: cash rounding to the nearest five cents

One line, one unit at nine point nine nine, one tax of twenty-one percent price-excluded. Untaxed
nine point nine nine, tax two point one zero, total twelve point zero nine. Cash rounding step five
hundredths, method "half away from zero".

```formula
total              = 12.09
target_total       = round_to_step( 12.09 , 0.05 , half away ) = 12.10
delta              = 12.10 − 12.09 = 0.01
```

- Strategy "add a rounding line": the block reports untaxed nine point nine nine, tax two point one
  zero, cash rounding one hundredth, grand total twelve point one zero.
- Strategy "adjust the biggest tax": the single tax group's tax becomes two point one one; the block
  reports untaxed nine point nine nine, tax two point one one, grand total twelve point one zero.

---

## 11. Fiscal positions

### 11.1 Mapping taxes

*Mirrored in spirit; the lookup table is server-computed.*

```formula
map_tax( fiscal_position , taxes ) :
    when there is no fiscal position:            result = taxes unchanged
    when the fiscal position has no taxes at all: result = the taxes that belong to no fiscal position
    otherwise:  for each tax in taxes, in order, look it up in the fiscal position's tax table;
                if found, append every replacement it maps to;
                if not found, append the tax itself;
                remove duplicates, keeping the first occurrence
```

The tax table of a fiscal position is built by walking the taxes attached to that fiscal position
and, for each tax the attached tax declares as "replaces", recording the attached tax as a
replacement of it. One replaced tax may therefore expand into several replacements — this is how a
single domestic tax becomes a pair of reverse-charge taxes.

### 11.2 Mapping accounts

```formula
map_account( fiscal_position , account ) = the mapped destination when the account appears as a
                                           source in the fiscal position's account table,
                                           otherwise the account itself
```

### 11.3 Automatic detection

```
Detecting the fiscal position of a partner and a delivery address:
 1. If there is no partner, there is no fiscal position.
 2. Determine whether the transaction is inside the same economic union and whether both parties
    carry a number issued by the same country:
       both_numbers_present = the company has a tax identification number
                              and the partner has one
       inside_union = both_numbers_present
                      and the first two characters of the company's number are the code of a member
                          country of the union group
                      and the first two characters of the partner's number are the code of a member
                          country of the union group
       same_country_prefix = both_numbers_present
                      and the first two characters of the two numbers are equal
 3. If no delivery address was supplied, or if inside_union and same_country_prefix and the
    partner's country equals the company's country, then the delivery address is the partner
    itself.
 4. If the delivery address carries a manually chosen fiscal position, return it.
    Otherwise, if the partner carries one, return it.
 5. If the partner has no country, there is no fiscal position.
 6. Search every fiscal position of the acting company whose "detect automatically" flag is set,
    and return the first match according to section 11.4, evaluated against the delivery address.
```

### 11.4 Precedence and the match test

Candidates are sorted by:

1. the number of ancestors of their company, **descending** — a fiscal position defined on a branch
   wins over one defined on its parent;
2. then by their sequence, ascending.

The first candidate that satisfies **all five** of the following predicates wins:

| Predicate | Satisfied when |
|---|---|
| tax registration | the fiscal position does not require one, **or** the counterpart's registration is valid for the acting company |
| postal code | the fiscal position has no complete range, **or** the counterpart's postal code lies between the two bounds inclusive, compared as text |
| state | the fiscal position lists no state, **or** the counterpart's state is one of them |
| country | the fiscal position names no country, **or** the counterpart's country is that country |
| country group | the fiscal position names no group, **or** the counterpart's country is a member of the group **and** (the counterpart has no state, or that state is not one of the group's excluded states) |

"The counterpart's registration is valid" means: the counterpart has a non-empty tax identification
number, and — when the cross-border verification applies, that is when the counterpart's number is
checked against the union register for this company and either the company's country is in the
union or the counterpart's country hosts a foreign registration of the company — the number is also
marked valid by that register.

### 11.5 Adapting the unit price when taxes are substituted

*Mirrored.* Substituting a price-included tax by another changes what the stored unit price means.

```
Adapting a unit price from an original set of taxes to a new set:
 1. If the two sets are identical, return the price unchanged.
 2. If at least one tax of the original set is not price-included, return the price unchanged.
 3. Compute the original taxes on a quantity of one with the "round per tax" method and no special
    mode; take the resulting untaxed total as the new price.
 4. Compute the new taxes on that price, quantity one, "round per tax", special mode
    'total_excluded'; let the delta be the sum of the tax amounts of those new taxes that are
    themselves price-included.
 5. Return the price from step 3 plus that delta.
```

Step 2 is deliberate: a mapping from a price-**excluded** tax to a price-**included** one leaves the
number alone, so that a rule mapping fifteen percent excluded to six percent included turns a price
of one hundred into one hundred divided by one point zero six rather than into one hundred six.

**Worked example (mandatory example 9): a fiscal position substitution.**

Configuration. Domestic sales tax *D*: twenty-one percent, price-included, invoice distribution
base tagged `base 21` and one tax line of one hundred percent on "tax payable" tagged `tax 21`.
Intra-union reverse-charge tax *R*: twenty-one percent, price-excluded, invoice distribution base
tagged `base intra-union` and two tax lines — plus one hundred percent on "tax payable" tagged
`tax due` and minus one hundred percent on "tax deductible" tagged `tax deductible`. A fiscal
position "Intra-union business" with "detect automatically" on, no country, the union country group,
"tax registration required" on, and one tax: *R*, declaring that it replaces *D*. A fiscal position
account mapping sends the domestic revenue account to the export revenue account.

A product priced one hundred twenty-one with tax *D*, sold to a business established in another
member country whose registration number is valid.

1. **Detection.** The partner has a country in the union group, a valid registration, no postal
   range and no state restriction; "Intra-union business" matches.
2. **Account mapping.** The line's revenue account becomes the export revenue account.
3. **Tax mapping.** The line's taxes are `[D]`; the tax table maps *D* to `[R]`; the line's taxes
   become `[R]`.
4. **Unit price adaptation.** The original set `[D]` is entirely price-included, so the adaptation
   runs. Step 3: computing *D* on one hundred twenty-one gives an untaxed total of one hundred.
   Step 4: computing *R* on one hundred with the mode `total_excluded` gives a tax amount of
   twenty-one, but *R* is price-excluded, so the delta is zero. The new unit price is one hundred.
5. **Computation.** Base one hundred; *R* produces two halves of plus twenty-one and minus
   twenty-one.

| Entry | Account | Amount | Report tags |
|---|---|---|---|
| Base | export revenue | −100.00 (credit) | `base intra-union` |
| Tax, positive half | tax payable | −21.00 (credit) | `tax due` |
| Tax, reverse-charge half | tax deductible | +21.00 (debit) | `tax deductible` |
| Counterpart | receivable | +100.00 (debit) | none |

The customer is invoiced one hundred, not one hundred twenty-one; the ledger nets to zero tax; the
return reports a base of one hundred, a tax due of twenty-one and a deductible tax of twenty-one.

### 11.6 The price hint shown next to a product price

*Server-only.* For a product price *p* and the product's sales taxes restricted to the acting
company:

1. Run the engine once on quantity one at price *p*, giving a total with taxes and an untaxed
   total.
2. Build the hint from at most three fragments, in this order:
   - *"<total with taxes> Incl. Taxes"*, only when the total with taxes differs from *p* at the
     currency's precision;
   - *"<untaxed total> Excl. Taxes"*, only when the untaxed total differs from *p*;
   - when the withholding capability is installed and the withheld amount is non-zero,
     *"<withheld amount> Tax Withheld"*, where the withheld amount is computed by a second run of
     the engine with withholding taxes enabled, summing the negated tax amount of every withholding
     tax.
3. The hint is the fragments joined by a comma and a space, wrapped as *"(= …)"*. When there is no
   fragment, the hint is a single space.

A related helper computes the public price of a product: if the product has no sales tax, the price
is unchanged; otherwise the taxes are computed on the given price and, when the resulting total
with taxes equals the given price, that total is returned (the taxes are already included);
otherwise the taxes are recomputed forcing the "price included" interpretation and the untaxed total
is returned.

### 11.7 Removing an inapplicable price-included tax from a price

*Server-only.* When a product proposes taxes that the document does not use, any of the product's
taxes that is price-included and absent from the document's tax set is removed from the price:
the removed taxes are computed on the price and the untaxed total replaces it. When a company is
given, both tax sets are first restricted to that company.

---

## 12. Cash basis: making a tax exigible at payment

*Server-only.* A tax whose exigibility is "based on payment" is posted, at invoice time, on its
**transition account** instead of its real tax account. When the invoice is reconciled, one
additional journal entry per reconciliation moves the proportional share from the transition
account to the real tax account and stamps the report tags.

### 12.1 Deciding whether an entry is needed

For a candidate document:

1. Walk its journal items. A line whose account kind is receivable or payable is a **term line**;
   accumulate, with a sign of plus one when its balance is positive and minus one otherwise, its
   balance, its residual, its amount in document currency and its residual in document currency.
2. A line that **is** a tax entry of a tax exigible on payment is collected with the treatment
   `tax`.
3. Otherwise, a line whose flattened base taxes contain at least one tax exigible on payment is
   collected with the treatment `base`. (A line can therefore be collected as `base` even though it
   is itself a tax entry of a different, invoice-exigible tax — only its base part is deferred.)
4. If nothing was collected, or if there is no term line at all, no cash basis entry is ever
   produced for this document.
5. Collect the distinct currencies of the term lines and of the collected lines. If there is more
   than one, no cash basis entry is produced — the mechanism does not support a document mixing
   currencies.
6. The document is **fully paid** when the total residual is zero in the company currency or the
   total residual in document currency is zero in that currency.

### 12.2 The paid percentage of one partial reconciliation

For each partial reconciliation and for each of its two sides, let *M* be the document on that side
and *counterpart* the document on the other side.

```formula
partial_amount            = the partial's amount                    (company currency)
partial_amount_currency   = the partial's amount on M's side        (document currency)
```

When *M* is the debit side, the rate reference amounts are the negated balance and negated
document-currency amount of the credit line; when *M* is the credit side, they are the balance and
document-currency amount of the debit line. When both sides are themselves invoices — the case of a
credit note reconciled against an invoice — the rate reference amounts are instead *M*'s own line
amounts, the payment date is *M*'s own date and the settlement date is the later of the two
reconciled lines' dates. Otherwise the payment date and the settlement date are both the
counterpart line's date.

```formula
when the document currency is the company currency:
    skip this side entirely if the partial amount is zero in the company currency
    percentage = partial_amount ÷ total_balance
otherwise:
    skip this side entirely if the partial amount in document currency is zero in that currency
    percentage = partial_amount_currency ÷ total_amount_currency
```

where `total_balance` and `total_amount_currency` are the signed totals of the term lines from
section 12.1.

The **payment rate**, used to convert each cash basis line back into the company currency, is:

```formula
when the two reconciled lines have different currencies:
    payment_rate = the conversion rate from the company currency to the document currency,
                   for the company, at the payment date
                   (or, when the caller forced a rate at payment registration, that forced rate)
when they share a currency and the rate reference balance is non-zero:
    payment_rate = rate_reference_amount_currency ÷ rate_reference_balance
otherwise:
    payment_rate = 0
```

### 12.3 Composing the entry

For each partial, one journal entry is created:

- **Journal**: the company's cash basis journal. If the company has none:
  *"There is no tax cash basis journal defined for the '<company name>' company.\nConfigure it in Accounting/Configuration/Settings"*
- **Date**: the later of the settlement date and the day after the user's effective fiscal lock
  date for that journal.
- **Reference**: the originating document's number.
- **Fiscal position**: the originating document's fiscal position.
- Links: the partial reconciliation and the originating document.

Then, for each collected line:

```formula
amount_currency = round_to_line_currency( line.amount_currency × percentage )
balance         = amount_currency ÷ payment_rate          , or 0 when payment_rate = 0
```

with one correction: for a line collected as `tax`, when the document is now fully paid **or** the
line's remaining residual in document currency is smaller in absolute value than the computed
share, **and** this is the last partial being processed for that document, the share is replaced by
the line's whole remaining residual. A running residual per tax line is decremented by each share
so that successive partials never over-allocate. This is what guarantees that the sum of the cash
basis entries of a fully paid invoice matches the invoice's tax to the cent.

The shares are grouped so that as few journal items as possible are produced:

- a `base` share's grouping key is (currency, partner, account, the set of taxes exigible on
  payment, analytic distribution);
- a `tax` share's grouping key is the same plus the distribution line.

Shares sharing a key are added together (debit and credit are re-derived from the summed balance,
and for tax shares the base amounts are summed too).

Finally, for each grouped share, **two** journal items are created in a fixed alternating order —
the counterpart first, then the share itself, with consecutive sequence numbers two apart — whose
content is given in `accounting-effects.md` section 6.

An entry is posted immediately when both reconciled documents are posted; otherwise it is left in
draft until they are.

### 12.4 Reconciling the transition account

For every `tax` share whose original tax entry sits on a reconcilable account, the newly created
counterpart item is reconciled with the original tax entry, so that the transition account empties
as the invoice is paid. Items already reconciled, and counterparts whose amount rounded to zero,
are skipped.

### 12.5 Worked example (mandatory example 8): a payment of forty percent under deferred exigibility

Configuration. A customer invoice in the company currency, one line of one thousand, one sales tax
of twenty-one percent whose exigibility is "based on payment", transition account "tax to receive",
real tax account "tax payable", cash basis journal "Cash Basis", company base account
"cash basis base".

**The invoice, posted.**

| Journal item | Account | Debit | Credit | Base taxes | Tags |
|---|---|---|---|---|---|
| Product | revenue | | 1 000.00 | the cash basis tax | none (the base tags are withheld) |
| Tax | tax to receive | | 210.00 | | none (the tax tags are withheld) |
| Term | receivable | 1 210.00 | | | |

No report tag is stamped, because at invoice time the tax is not yet exigible.

**A payment of four hundred eighty-four is received and reconciled.**

```formula
total_amount_currency = 1 210.00        (the single term line)
partial_amount        =   484.00
percentage            = 484.00 ÷ 1 210.00 = 0.40
payment_rate          = 1                (same currency, the reference amounts cancel)
```

Shares:

```formula
base share   = round( −1 000.00 × 0.40 ) = −400.00     (the product line is a credit)
tax share    = round(   −210.00 × 0.40 ) =  −84.00
```

**The cash basis entry, dated on the payment date, in the cash basis journal:**

| Sequence | Journal item | Account | Debit | Credit | Base taxes | Tags |
|---|---|---|---|---|---|---|
| 0 | base counterpart | cash basis base | 400.00 | | none | none |
| 1 | base | cash basis base | | 400.00 | the cash basis tax | the base tags of the invoice distribution |
| 2 | tax counterpart | tax to receive | 84.00 | | none | none |
| 3 | tax | tax payable | | 84.00 | the cash basis tax | the tax tags of the invoice distribution |

The two base items cancel each other on the same account; their only purpose is to carry the base
amount into the tax return. The two tax items move eighty-four from the transition account to the
real tax account. The counterpart at sequence two is then reconciled against the invoice's own tax
item, leaving one hundred twenty-six on the transition account.

**The remaining sixty percent, paid later.** Percentage zero point six; base share minus six
hundred; tax share, since this is the last partial of a now fully paid invoice, is forced to the
tax line's remaining residual of one hundred twenty-six rather than the computed one hundred
twenty-six exactly — identical here, but the rule is what prevents a one-cent drift when the
percentages do not divide evenly. The transition account reaches zero and the two tax items become
fully reconciled.

**A three-way split that needs the correction.** The same invoice paid in three instalments of four
hundred three point three three, four hundred three point three three and four hundred three point
three four. The percentages are zero point three three three three zero five seven eight,
the same, and the remainder. The tax shares computed by percentage would be sixty-nine point
nine nine, sixty-nine point nine nine and seventy point zero one, which sum to two hundred nine
point nine nine. The last-partial correction replaces the third share by the residual of seventy
point zero two, so the three shares sum to exactly two hundred ten.

---

## 13. Withholding at payment

*Server-only.* A withholding tax is a tax with a **negative** amount and the flag "withhold on
payment". Two rules keep it out of the ordinary flow:

1. Whenever tax details are added to a base line, unless the caller explicitly asked for withholding
   taxes to be calculated, a filter is installed that removes every withholding tax from the
   evaluation. The tax therefore never appears on an invoice total.
2. The tax label of a withholding tax is its invoice label only, never its name, so leaving the
   label empty hides it from printed documents.

### 13.1 Deriving the withholding lines of a payment

Given the base lines of the documents about to be paid:

1. Rebuild each base line with withholding calculation **enabled** and no filter, then run the
   engine and the document-wide rounding on the rebuilt lines.
2. Aggregate the results by the key
   ```
   ( name , analytic distribution , account , tax , skip flag , currency )
   ```
   where the name is the base line's manual tax line name if it has one, otherwise the tax's name;
   the account is the company's withholding tax base account if set, otherwise the base line's
   account; and the skip flag is true for every tax that is **not** a withholding tax.
3. Ignore every group whose skip flag is true.
4. Compare with the withholding lines that already exist, grouped by the same key:
   - more than one existing line on a key: keep the first and delete the others;
   - an existing line: update its four source amounts — base in document currency, base in company
     currency, tax in document currency **negated**, tax in company currency **negated**;
   - no existing line: create one with those four source amounts, the tax, the analytic
     distribution, the account, the name, the source tax and the source currency;
   - a key that exists only among the existing lines: delete those lines.

The negation in step 4 is what turns the engine's negative tax amount into a positive "amount
withheld".

### 13.2 The net amount

```formula
withholding_net_amount = payment_amount − sum of the amounts of the withholding lines
```

Registering a payment whose net amount is negative is refused:
*"The withholding net amount cannot be negative."*

### 13.3 Turning the withholding lines into journal items

1. **Before consuming any sequence value**, verify that every line has either a number or a tax
   with a withholding sequence; otherwise refuse with
   *"Please enter the withholding number for the tax <tax name>"*.
2. For each line without a number, draw the next value of its tax's withholding sequence.
3. Convert each line into a base line with:
   ```formula
   conversion_rate = rate from the company currency to the payment currency, at the payment date
   sign            = +1 for an incoming payment, −1 for an outgoing one
   price_unit      = the line's withholding base
   quantity        = 1
   account         = the line's account
   computation_key = the line's own identifier (each line rounds independently)
   manual_total_excluded_currency = the withholding base
   manual_total_excluded          = round_to_company_currency( withholding base ÷ conversion_rate )
   manual_tax_amounts for the line's tax:
        base_amount_currency = the withholding base
        base_amount          = round_to_company_currency( withholding base ÷ conversion_rate )
        tax_amount_currency  = − the withholding amount
        tax_amount           = round_to_company_currency( − withholding amount ÷ conversion_rate )
   is_refund       = the rule of entities section 8.5
   ```
4. Run the engine, the document-wide rounding, the accounting derivation and the entry production
   of section 8.6 on those base lines.
5. Emit **one journal item per produced tax entry**, with its amounts **negated** and the payment's
   partner, named *"WH Tax: <tax name>"*.
6. Aggregate the base updates by their base grouping key extended with the tag set, summing the
   amounts and collecting the line names. For each aggregate emit **two** journal items with the
   payment's partner:
   - *"WH Base: <names>"* with the aggregated amounts, **no** taxes and **no** tags;
   - *"WH Base Counterpart: <names>"* with the negated amounts, no analytic distribution, and
     keeping the taxes and the tags of the grouping key.

### 13.4 Worked example (mandatory example 11): a withholding

Configuration. A sales tax of fifteen percent, ordinary. A withholding tax of **minus one percent**
with "withhold on payment" on, a withholding sequence with four-digit padding, and a distribution of
one base line and one tax line of one hundred percent on the account "withholding tax credit". The
company's withholding tax base account is "withholding base". Everything in the company currency.

**The invoice.** One customer invoice line, one unit at one thousand, taxes: the fifteen percent tax
only (the withholding tax is not put on the invoice).

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Product | revenue | | 1 000.00 |
| Tax 15% | tax payable | | 150.00 |
| Term | receivable | 1 150.00 | |

**Registering the payment.** The user switches "withhold tax amounts" on and adds a line with the
withholding tax and a withholding base of one thousand.

```formula
original_base_amount = 1 000.00
tax computation on 1 000.00 with the −1% tax  →  tax amount = −10.00
original_tax_amount  = − ( −10.00 ) = 10.00
base_amount          = 1 000.00        (typed by hand, so no paid factor is applied)
amount               = round( 10.00 × 1 000.00 ÷ 1 000.00 ) = 10.00
withholding_net_amount = 1 150.00 − 10.00 = 1 140.00
```

**The payment.** Its amount stays one thousand one hundred fifty — the customer's debt is settled in
full — but only one thousand one hundred forty reaches the bank.

| Journal item | Account | Debit | Credit | Base taxes |
|---|---|---|---|---|
| Liquidity | outstanding receipts | 1 140.00 | | |
| Counterpart | receivable | | 1 150.00 | |
| WH Tax: withholding | withholding tax credit | 10.00 | | |
| WH Base | withholding base | 1 000.00 | | none |
| WH Base Counterpart | withholding base | | 1 000.00 | the withholding tax |

The two base items cancel on the withholding base account and exist only so that the withholding
base reaches the tax return through the base tags of the withholding tax's distribution. The tax
item records the amount the customer retained and paid to the authorities on the company's behalf,
as a claim against the authorities.

**With an instalment.** If the same invoice is paid in two instalments of five hundred seventy-five
and the withholding line was derived from the documents rather than typed, the paid factor applies:

```formula
full_amount                = 1 150.00       (total still to pay for the batch)
moves_total                = 1 150.00
split_factor               = 1 150.00 ÷ 1 150.00 = 1
percentage_paid_factor     = 575.00 ÷ 1 150.00 × 1 = 0.5
base_amount                = round( 1 000.00 × 0.5 ) = 500.00
amount                     = round( 10.00 × 500.00 ÷ 1 000.00 ) = 5.00
withholding_net_amount     = 575.00 − 5.00 = 570.00
```

### 13.5 Constraints specific to withholding taxes

- A withholding tax may not use the "group of taxes" or the "percentage tax included" computations:
  *"Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations."*
- Switching "withhold on payment" on forces the exigibility back to "based on invoice" and the
  price-inclusion override to "tax excluded".
- Setting the amount to zero or above clears the flag.
- The withholding feature is offered only when the acting company owns at least one withholding tax
  matching the payment direction, and — on the register-payment wizard — only when the wizard will
  create a single journal entry (a wizard that cannot be edited, or one that will split into one
  payment per document, hides it).

---

## 14. Document-level helpers built on the engine

*Mirrored.* These helpers all reduce or reshape a set of base lines while guaranteeing that no
amount is lost.

### 14.1 Which taxes can be discounted

```formula
can_be_discounted( tax ) = tax.amount_type is neither 'fixed' nor 'code'
```

A fixed tax and a custom-formula tax are per-unit charges; scaling them by a discount percentage
would be wrong, so they are excluded from every proportional operation.

### 14.2 Splitting a base line

Given a list of weights, a base line is split into that many base lines such that computing taxes
on the pieces gives exactly the same result as on the whole.

1. Normalise the weights (section 7.1 step 4).
2. Multiply every **raw** amount — the four totals, and each tax entry's four raw amounts — by the
   normalised weight of the piece.
3. Distribute every **rounded** amount with the smooth allocation of section 7.1, using the delta to
   distribute equal to the whole's rounded amount and the weights equal to the pieces' weights. This
   guarantees the pieces add back exactly to the whole.
4. Set each piece's total with taxes to its untaxed total plus the sum of its tax amounts.
5. Build each piece's base line with a unit price equal to the whole's unit price times the
   normalised weight, and the piece's tax details attached.

### 14.3 Merging two tax details

The four totals and the two deltas are added. Tax entries are merged by tax, adding all eight
amounts. Then, for every tax present in the first block but absent from the second — typically a
fixed tax, whose base is an artefact — the second block's untaxed totals are added to that tax's
base amounts and the second block's untaxed deltas are added to its rounded base amounts, so that
the merged base stays meaningful.

### 14.4 Reducing many base lines to few

To minimise the number of lines a discount or a down payment has to create:

1. Turn each base line into an equivalent line of quantity one whose unit price is the quantity
   times the discounted unit price and whose discount is zero.
2. Group by the set of taxes, extended by the caller's own grouping keys.
3. Within a group, add the unit prices and merge the tax details (section 14.3).
4. Drop groups whose unit price is zero at the currency's precision.
5. Recompute the analytic distribution of each group as a weighted average of the members'
   distributions, weighted by each member's raw untaxed total:
   ```formula
   weight_of_account = sum over members of ( member_distribution_percentage × member_raw_untaxed ÷ 100 )
   new_percentage    = weight_of_account × 100 ÷ sum of member_raw_untaxed
   new_percentage    = 100  when the sum of member raw untaxed totals is zero
   ```
   *Illustration.* A line of one thousand distributed one hundred percent to an account and a line
   of minus one hundred distributed fifty percent to the same account give
   `((1000 × 1) + (−100 × 0.5)) ÷ (1000 − 100) = 1.0555…`, that is one hundred five point five six
   percent.

### 14.5 Reaching a target amount

Used by the global discount and by the down payment. Inputs: the base lines, an amount kind
(`fixed` or `percent`) and an amount.

1. Compute the current grand total in both currencies, and the current base and tax amounts per
   tax.
2. Turn the request into a percentage and a target grand total:
   ```formula
   sign       = −1 when the requested amount is negative, +1 otherwise
   'fixed'  : percentage = |amount| ÷ current_total_in_line_currency   (0 when that total is 0)
              target_total_currency = round_to_line_currency( amount )
              target_total          = round_to_company_currency( target_total_currency ÷ rate )
   'percent': percentage = |amount| ÷ 100
              target_total_currency = round_to_line_currency( current_total_currency × sign × percentage )
              target_total          = round_to_company_currency( current_total × sign × percentage )
   ```
3. The target base and tax amount of each tax are the current ones multiplied by the sign and the
   percentage, rounded. The target untaxed total is the target grand total minus the sum of the
   target tax amounts.
4. Reduce the lines (section 14.4), scale each reduced line's unit price by the sign and the
   percentage, and run the engine and the document-wide rounding on the result.
5. Sort the new lines by (whether they carry a special type, untaxed total descending) and, per tax
   and per currency, distribute the difference between the target and the achieved tax amount, and
   likewise for the base amount, using the smooth allocation weighted by each entry's share.
6. Distribute the remaining difference on the untaxed totals into the lines' untaxed deltas — and,
   in the line currency, also into their unit prices.
7. Freeze the result into manual amounts (section 9.1), so that recomputing the document cannot
   move it.

A **global discount** calls this with the negated amount after dropping every non-discountable tax;
a **down payment** calls it with the amount as given after wrapping every non-discountable tax into
the base.

### 14.6 Dispatching taxes into separate base lines

To exclude some taxes from a proportional operation without losing their amounts, each base line is
partitioned into the taxes to keep and the taxes to exclude, and the excluded taxes' amounts are
moved into new base lines that carry no tax at all. The resulting set has the same grand total as
the original and can be scaled safely. Symmetric helpers exist to squash those extra lines back and
to dispatch global-discount lines and return-of-merchandise lines.

---

## 15. Tax identification numbers

*Server-only.* Every partner, and every fiscal position carrying a foreign registration, stores a
tax identification number. The number is normalised and checked whenever it is written. This
section gives the pipeline, the shared check primitives as arithmetic, and then one entry per
country.

### 15.1 The pipeline

Inputs: a country, the number as typed, a label for the error message, and a validation mode which
is one of *off*, *error* or *blank-on-failure*.

1. If there is no country or no number, return the number unchanged and report no validated
   country.
2. If the number is exactly one character long:
   - if it is a solidus, or the mode is *off*, return it unchanged;
   - if the mode is *blank-on-failure*, return an empty number;
   - if the mode is *error*, refuse with
     *"To explicitly indicate no (valid) VAT, use '/' instead. "*.
3. **Split the prefix.** The prefix is the first two characters uppercased when both are letters,
   and empty otherwise. The remainder is everything after the first two characters with spaces
   removed; when there is no prefix the remainder is the whole number.
4. If the prefix is the two letters of the economic union itself and the country is not a member of
   that union, return the number unchanged — a company outside the union that trades with
   non-businesses inside it may carry such a number.
5. Translate the prefix to a country code: two prefixes differ from the country code they denote —
   the prefix used for Greece maps to the Greek country code, and the prefix used for Northern
   Ireland maps to the United Kingdom country code. Otherwise the prefix is already the code.
6. If that country code belongs to the union-prefix country group:
   - when the stored country is itself in that group **and** a prefix was present, strip the prefix
     from the number and remember the prefix as the "prefixed country";
   - otherwise remember that a second, union-wide attempt may be needed.
7. The **code to check** is the prefixed country when there is one, otherwise the stored country's
   code.
8. **Normalise** the number with the country's formatting routine (section 15.3).
9. If the prefixed country is the Greek country code, replace it by the Greek union prefix.
10. The **number to return** is the prefixed country concatenated with the normalised number.
11. If the mode is *off*, or the caller asked for validation to be skipped, return now.
12. Detect a **doubled prefix**: a prefixed country is present and the number to return starts with
    that prefix twice.
13. Run the country check (section 15.4) on the normalised number. If it fails, or the prefix is
    doubled:
    - when a union-wide attempt is pending, retry the whole pipeline against the country denoted by
      the prefix with the original prefixed number; if that retry also fails, refuse with the
      standard message followed by a blank line and
      *"If you are trying to input a European number, this is the expected format: "* and the
      country's example number;
    - when the mode is *error*, refuse with the standard message;
    - when the mode is *blank-on-failure*, return an empty number.

**The standard message.** Let the label be *"VAT"*, replaced by the country's own label for the
number when the checked country is the acting company's country and that country defines one.

When the record label does not contain the text "False":

> The **&lt;label&gt;** number [&lt;the number&gt;] for &lt;the record label&gt; does not seem to be valid.
> Note: the expected format is &lt;the example&gt;

Otherwise (the record has no name, as for the anonymous storefront user):

> The **&lt;label&gt;** number [&lt;the number&gt;] does not seem to be valid.
> Note: the expected format is &lt;the example&gt;

The record label is *"partner [&lt;partner name&gt;]"* for a partner and
*"fiscal position [&lt;fiscal position name&gt;]"* for a fiscal position. The note is omitted when the
country has no example.

**When no routine exists for a country the number is accepted unchanged.**

### 15.2 Shared check primitives

Throughout, `d1 … dn` are the characters of the number read left to right, `value(c)` is the
position of a character in the stated alphabet (zero-based), and `mod` is the remainder of a
Euclidean division whose result always has the sign of the divisor (so `−3 mod 11 = 8`).

#### 15.2.1 Weighted modulus

```formula
weighted_sum( number , weights ) = sum over i of ( weight_i × digit_i )
```

The weights are paired with the digits from the left unless stated otherwise; when the weight list
is shorter than the number, the surplus digits are ignored.

#### 15.2.2 The doubling checksum (commonly called the Luhn checksum)

Over an alphabet of size *n* (ten for plain digits):

```formula
v1 … vm = the alphabet positions of the characters, read RIGHT to LEFT
checksum = (   sum of v at the odd positions 1, 3, 5, …
             + sum over v at the even positions 2, 4, 6, … of
                   ( floor( v × 2 ÷ n ) + ( v × 2 mod n ) )
           ) mod n
valid when checksum = 0
check digit to append = alphabet[ ( n − checksum( number with alphabet[0] appended ) ) mod n ]
```

For plain digits, `floor(v × 2 ÷ 10) + (v × 2 mod 10)` is the familiar "double it and add the two
digits together".

#### 15.2.3 The recursive modulus eleven over ten

```formula
check = 5
for each digit d, left to right:
    check = ( ( ( check , or 10 when check is 0 ) × 2 ) mod 11 + d ) mod 10
valid when check = 1
check digit to append = ( 1 − ( ( check , or 10 when check is 0 ) × 2 ) mod 11 ) mod 10
```

#### 15.2.4 The recursive modulus thirty-seven over thirty-six

The same shape over an alphabet of thirty-six characters (digits then letters):

```formula
check = 18
for each character c, left to right:
    check = ( ( ( check , or 36 when check is 0 ) × 2 ) mod 37 + value(c) ) mod 36
valid when check = 1
```

#### 15.2.5 The modulus ninety-seven over ten

```formula
expand:   replace each character by the decimal writing of its value in base thirty-six
          (digits become themselves, A becomes 10, B becomes 11, …, Z becomes 35)
          and concatenate the results
checksum = the resulting integer mod 97
valid when checksum = 1
check digits to append = 98 − ( checksum of the number with "00" appended )   , written on two digits
```

#### 15.2.6 Cleaning

"Cleaning with a character set" means removing every character of that set from the number.
"Trimming" removes leading and trailing white space. Unless stated otherwise, numbers are also
uppercased.

### 15.3 Normalisation routines

Before the check, the number is passed through a country-specific normaliser. The default
normaliser is the country's own "compact" routine listed with the check in section 15.4. Seven
countries have an application-level normaliser that overrides it:

| Country | Normalisation |
|---|---|
| Albania | Split the prefix off, compact the remainder (remove spaces, uppercase, drop a leading two-letter country code or the same code in parentheses), then re-join prefix and remainder. |
| Economic union prefix | Return the number unchanged. |
| Switzerland | Prepend the two-letter country code, apply the library's formatting for that country, then drop the two leading characters again. The result is the enterprise identifier written with a hyphen and two full stops followed by a space and the three-letter tax-regime suffix. |
| Chile | Remove full stops, the two-letter country code, spaces and hyphens; uppercase; then, when more than two characters remain, re-insert a hyphen before the last character. |
| Colombia | Apply the library formatting, then remove full stops and hyphens, then re-insert a hyphen before the last character when more than two characters remain. |
| Vietnam | Apply the library formatting only when the number matches the ten-digit (optionally plus a three-digit branch) company pattern; otherwise leave it alone. |
| Hungary | Compact; then, when the number matches the eight-digit plus one-digit plus two-digit company pattern, re-insert hyphens as `########-#-##`. |
| Iceland | Split the prefix off, compact the remainder, re-join. |
| San Marino | Prepend the two-letter country code, compact, then drop the two leading characters. |

### 15.4 Per-country checks

Each entry states: the accepted written forms, the cleaning applied, the structural checks, and
the check-digit arithmetic. A number that passes all stated checks is valid. The example is the
one the error message quotes.

---

**Albania** — example `ALJ91402501L`.
Clean by removing spaces and uppercasing; drop a leading two-letter country code or that code in
parentheses. The result must be exactly ten characters and must match: one letter among J, K, L, M;
eight digits; one uppercase letter. No check digit.

**Andorra** — clean by removing spaces, hyphens and full stops; uppercase; trim. Eight characters.
The first and last must be letters and the six in between digits. The first letter must be one of
A, C, D, E, F, G, L, O, P, U. When the first letter is F the six digits must not exceed `699999`;
when it is A or L they must lie strictly between `699999` and `800000`. No check digit.

**Argentina** — example `20055361682`.
Clean by removing spaces and hyphens. Eleven digits. The first two must be one of
20, 23, 24, 27, 30, 33, 34, 50, 51, 55.

```formula
weights = ( 5 , 4 , 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d10
s = weighted_sum mod 11
check_digit = the character at position ( 11 − s ) of the string "012345678990"
valid when check_digit = d11
```

The odd trailing string makes both the remainder ten and the remainder eleven produce a zero and a
nine respectively, as the national rule requires.

**Australia** — example `83 914 571 673`.
Clean by removing spaces. Eleven digits.

```formula
weights = ( 3 , 5 , 7 , 9 , 11 , 13 , 15 , 17 , 19 )  applied to d3 … d11
s = − weighted_sum
expected_first_two = 11 + ( ( s − 1 ) mod 89 )
valid when the decimal writing of expected_first_two equals d1 d2
```

**Austria** — example `ATU12345675`.
Clean by removing spaces, hyphens, solidi and full stops; uppercase; drop a leading country code.
Nine characters: the letter U followed by eight digits.

```formula
check_digit = ( 6 − doubling_checksum( d2 … d8 ) ) mod 10
valid when check_digit = d9
```

**Azerbaijan** — clean by removing spaces; left-pad with a zero when nine characters long. Ten
digits. The last digit must be one or two.

```formula
weights = ( 4 , 1 , 8 , 6 , 2 , 7 , 5 , 3 )   applied to d1 … d8
check = weighted_sum mod 11
valid when check = d9
```

**Belarus** — clean by removing spaces, uppercasing, dropping a leading three-letter national
prefix in either alphabet, and transliterating the ten Cyrillic letters that look like Latin ones
into their Latin counterparts. Nine characters. Characters three to nine must be digits. The first
two must be either both digits or both letters from the set A, B, C, E, H, K, M, O, P, T. The first
character must be one of the digits one to seven or one of the letters A, B, C, E, H, K, M.

```formula
if the number is not all digits:
    replace the second character by the index of that letter in "ABCEHKMOPT"
alphabet = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
weights  = ( 29 , 23 , 19 , 17 , 13 , 7 , 5 , 3 )   applied to the first eight characters
check    = ( sum of weight × value(character) ) mod 11
invalid when check > 9
valid when check = d9
```

**Belgium** — example `BE0477472701`.
Clean by removing spaces, hyphens, solidi and full stops; uppercase; drop a leading country code;
replace a leading `(0)` by a zero; left-pad with a zero when nine characters long. Ten digits,
strictly positive, first digit zero or one.

```formula
checksum = ( the integer formed by d1 … d8 + the integer formed by d9 d10 ) mod 97
valid when checksum = 0
```

**Bulgaria** — example `BG1234567892`.
Clean by removing spaces, hyphens and full stops; uppercase; drop a leading country code. All
digits.

- **Nine digits** (legal persons):
  ```formula
  check = ( sum over i = 1..8 of i × di ) mod 11
  if check = 10 :  check = ( sum over i = 1..8 of ( i + 2 ) × di ) mod 11
  check_digit = check mod 10
  valid when check_digit = d9
  ```
- **Ten digits**: valid when **any** of the following holds — the number is a valid national
  personal number, or a valid foreigner's number, or the following check succeeds:
  ```formula
  weights = ( 4 , 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d9
  check_digit = ( 11 − weighted_sum ) mod 11
  valid when check_digit = d10
  ```
  *National personal number*: ten digits; the first six encode a date of birth where the month is
  increased by forty for a year after 1999 and by twenty for a year before 1900;
  ```formula
  weights = ( 2 , 4 , 8 , 5 , 10 , 9 , 7 , 3 , 6 )  applied to d1 … d9
  check_digit = ( weighted_sum mod 11 ) mod 10
  ```
  *Foreigner's number*: ten digits;
  ```formula
  weights = ( 21 , 19 , 17 , 13 , 11 , 9 , 7 , 3 , 1 )  applied to d1 … d9
  check_digit = weighted_sum mod 10
  ```
- Any other length is invalid.

**Brazil** — example: either eleven digits for a natural person or fourteen characters for a legal
person. The number is valid when **either** check succeeds.

*Natural person, eleven digits*, strictly positive:

```formula
d_check1 = ( 11 − sum over i = 1..9 of ( 10 − i + 1 ) × di ) mod 11 mod 10
           that is, weights ( 10 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )
d_check2 = ( 11 − ( sum over i = 1..9 of ( 11 − i + 1 ) × di + 2 × d_check1 ) ) mod 11 mod 10
           that is, weights ( 11 , 10 , 9 , 8 , 7 , 6 , 5 , 4 , 3 ) plus twice the first check digit
valid when d_check1 d_check2 = d10 d11
```

*Legal person, fourteen characters*: clean by removing spaces, hyphens, full stops and solidi;
uppercase. Must match "one or more digits or capital letters" and must not start with twelve zeros.

```formula
value(c) = the character's code point minus 48   (so '0'→0 … '9'→9, 'A'→17, … 'Z'→42)
weights1 = ( 5 , 4 , 3 , 2 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to the first twelve characters
d_check1 = ( 11 − weighted_sum1 ) mod 11 mod 10
weights2 = ( 6 , 5 , 4 , 3 , 2 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 ) applied to those twelve values plus d_check1
d_check2 = ( 11 − weighted_sum2 ) mod 11 mod 10
valid when d_check1 d_check2 = the last two characters
```

**Canada** — clean by removing hyphens and spaces. Nine or fifteen digits. The first nine must be
all digits and satisfy the doubling checksum. When fifteen characters long, characters ten and
eleven must be one of the four registered programme codes (a capital R followed by C, M, P or T)
and characters twelve to fifteen must be digits.

**Switzerland** — example `CHE-123.456.788 TVA` (also written with the Italian or German suffix).
The application supplies its own check and accepts, ignoring spaces:

```
E followed by nine digits, or E followed by a hyphen and three groups of three digits
separated by full stops, then an optional space, then one of the three-letter tax-regime
suffixes MWST, TVA or IVA
```

The three-letter suffix that spells the English abbreviation of the tax is **not** accepted.

```formula
take the nine digits of the enterprise identifier as n1 … n9
weights = ( 5 , 4 , 3 , 2 , 7 , 6 , 5 , 4 )   applied to n1 … n8
check_digit = ( 11 − ( weighted_sum mod 11 ) ) mod 11
valid when check_digit = n9
```

The library form the normaliser produces additionally allows a fourth suffix and requires a
three-letter country prefix; the application's own check is the one that decides validity.

**Chile** — example `76086428-5`.
Clean by removing spaces, hyphens and full stops; uppercase; drop a leading country code. Eight or
nine characters; all but the last must be digits.

```formula
read the digits of the number without its last character from RIGHT to LEFT as v0 , v1 , v2 , …
weight of v_i = 4 + ( ( 5 − i ) mod 6 )       (this cycles 2,3,4,5,6,7,2,3,…)
s = sum of weight × value
check_character = the character at position ( s mod 11 ) of the string "0123456789K"
valid when check_character = the last character
```

**China** — clean by removing spaces and hyphens; uppercase. Eighteen characters; the first eight
must be digits; every character must belong to the thirty-one-character alphabet
`0123456789ABCDEFGHJKLMNPQRTUWXY` (the letters I, O, S, V, Z are excluded).

```formula
weights = ( 1 , 3 , 9 , 27 , 19 , 26 , 16 , 17 , 20 , 29 , 25 , 13 , 8 , 24 , 10 , 30 , 28 )
          applied to the first seventeen characters
total   = sum of weight × value(character)
check_character = alphabet[ ( 31 − total ) mod 31 ]
valid when check_character = the eighteenth character
```

**Colombia** — example `213123432-1`.
Clean by removing full stops, commas, hyphens and spaces; uppercase. Between eight and sixteen
digits.

```formula
weights = ( 3 , 7 , 13 , 17 , 19 , 23 , 29 , 37 , 41 , 43 , 47 , 53 , 59 , 67 , 71 )
          applied to the digits BEFORE the last one, read RIGHT to LEFT
s = weighted_sum mod 11
check_digit = the character at position s of the string "01987654321"
valid when check_digit = the last digit
```

**Costa Rica** — example `3101012009`.
The application supplies its own check: the number must match one of
- nine digits whose first is not zero (natural person),
- ten digits (legal person or a particular residence permit class),
- eleven or twelve digits whose first is not zero (another residence permit class).

No check digit.

**Cyprus** — example `CY10259033P`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Nine characters; the
first eight must be digits. The first two digits may not be one followed by two.

```formula
translate = { 0→1 , 1→0 , 2→5 , 3→7 , 4→9 , 5→13 , 6→15 , 7→17 , 8→19 , 9→21 }
s = sum of translate(d) over the digits at ODD positions 1, 3, 5, 7
  + sum of d          over the digits at EVEN positions 2, 4, 6, 8
check_letter = the letter at position ( s mod 26 ) of the plain Latin alphabet
valid when check_letter = the ninth character
```

**Czechia** — example `CZ12345679`.
Clean by removing spaces and solidi; uppercase; drop a leading country code. All digits.

- **Eight digits**, not starting with nine:
  ```formula
  weights = ( 8 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d7
  check = ( 11 − weighted_sum ) mod 11
  check_digit = ( check , or 1 when check is 0 ) mod 10
  valid when check_digit = d8
  ```
- **Nine digits starting with six**:
  ```formula
  weights = ( 8 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d2 … d8
  check = weighted_sum mod 11
  check_digit = ( 8 − ( 10 − check ) mod 11 ) mod 10
  valid when check_digit = d9
  ```
- **Nine or ten digits** otherwise: the number must be a valid national personal number — the first
  six digits encode a date of birth in which the month carries an offset of fifty for the second
  sex and a further offset of twenty for a duplicate registration; for a ten-digit number the whole
  number taken as an integer must satisfy `(integer without the last digit) mod 11 mod 10 = last digit`.
- Any other length is invalid.

**Germany** — example `DE123456788` or `12/345/67890`. Valid when **either** check succeeds.

*Tax registration number*: clean by removing spaces, hyphens, full stops, solidi and commas;
uppercase; drop a leading country code. Nine digits, first digit not zero, satisfying the recursive
modulus eleven over ten (section 15.2.3).

*Regional tax number*: clean the same way. Ten, eleven or thirteen digits, matching one of the two
patterns of at least one of the sixteen regions. Each region defines a regional pattern and a
country-wide pattern, written with the placeholder letters F for the tax-office code, B for the
district, U for the serial and P for the check position; for example one region's regional pattern
is two office digits, three district digits, four serial digits and one check digit, and its
country-wide pattern prefixes a fixed two-digit region code and a zero. No check digit is verified.

**Denmark** — example `DK12345674`.
Clean by removing spaces, hyphens, full stops, commas, solidi and colons; uppercase; drop a leading
country code. Eight digits, first digit not zero.

```formula
weights = ( 2 , 7 , 6 , 5 , 4 , 3 , 2 , 1 )   applied to d1 … d8
valid when weighted_sum mod 11 = 0
```

**Dominican Republic** — example `1-01-85004-3` or `101850043`. Valid when **either** check
succeeds.

*Taxpayer number*: clean by removing spaces and hyphens. All digits. A published list of
twenty-four historical numbers is accepted unconditionally. Otherwise nine digits and

```formula
weights = ( 7 , 9 , 8 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d8
check = weighted_sum mod 11
check_digit = ( ( 10 − check ) mod 9 ) + 1
valid when check_digit = d9
```

*Identity card number*: clean the same way. A published list of several hundred historical numbers
is accepted unconditionally. Otherwise eleven digits satisfying the doubling checksum.

**Algeria** — clean by removing spaces. Fifteen or twenty digits. No check digit.

**Ecuador** — example `1792060346001` or `1792060346`.
The application supplies its own check: clean by removing spaces, hyphens and full stops;
uppercase; trim. The number is valid when it is ten or thirteen characters long and entirely
decimal. No check digit is verified.

(The library check, which the application replaces, additionally validates the province code, the
third digit's class and one of three weighted modulus-eleven checksums.)

**Estonia** — example `EE123456780`.
Clean by removing spaces; uppercase; drop a leading country code. Nine digits.

```formula
weights = ( 3 , 7 , 1 , 3 , 7 , 1 , 3 , 7 , 1 )   applied to d1 … d9
valid when weighted_sum mod 10 = 0
```

**Egypt** — clean by removing spaces, hyphens and solidi, and mapping the two families of
Arabic-Indic digit characters to plain digits. Nine digits. No check digit.

**Spain** — example `ESA12345674`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Nine characters;
characters two to eight must be digits.

- First character K, L or M: the last character must equal the natural-person check letter
  computed on characters two to eight.
- First character a digit: the whole number is a natural-person number —
  ```formula
  check_letter = the letter at position ( the integer formed by d1 … d8 mod 23 )
                 of the string "TRWAGMYFPDXBNJZSQVHLCKE"
  valid when check_letter = d9
  ```
- First character X, Y or Z: a foreigner's number — replace the first character by its index in
  "XYZ" (zero, one or two) and apply the natural-person rule.
- Otherwise the first character must be one of A, B, C, D, E, F, G, H, J, N, P, Q, R, S, U, V and
  the number is a legal-person number:
  ```formula
  c = doubling_check_digit( d2 … d8 )          (a digit)
  the last character must be either c or the letter at position c of "JABCDEFGHI"
  ```

**Economic-union-wide number** — a number carried by a trader established outside the union. The
prefix is either the two letters of the union or the two letters of the Isle of Man; the number is
eleven characters long in the first case and twelve in the second; everything after the prefix must
be digits; and characters three to five must be one of the twenty-eight numeric member-state codes.

**Finland** — example `FI12345671`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Eight digits.

```formula
weights = ( 7 , 9 , 10 , 5 , 8 , 4 , 2 , 1 )   applied to d1 … d8
valid when weighted_sum mod 11 = 0
```

**Faroe Islands** — clean by removing spaces, hyphens and full stops; uppercase; drop a leading
country code. Six digits. No check digit.

**France** — example `FR23334175221`.
Clean by removing spaces, hyphens and full stops; uppercase; drop a leading country code. Eleven
characters. The first two must belong to the thirty-three-character alphabet
`0123456789ABCDEFGHJKLMNPQRSTUVWXYZ` (the letters I and O are excluded); characters three to eleven
must be digits. When characters three to five are not three zeros, the last nine digits must
themselves be a valid company registration number (nine digits satisfying the doubling checksum).

```formula
when the whole number is digits:
    valid when the integer formed by d1 d2
              = ( the integer formed by d3 … d11 followed by the digits "12" ) mod 97

otherwise, let A = value of d1 and B = value of d2 in the thirty-three-character alphabet:
    when d1 is a digit :  check = A × 24 + B − 10
    when d1 is a letter:  check = A × 34 + B − 100
    valid when ( the integer formed by d3 … d11 + 1 + floor( check ÷ 11 ) ) mod 11 = check mod 11
```

**United Kingdom** — example `GB123456782` (or the Northern Ireland prefix).
Clean by removing spaces, hyphens and full stops; uppercase; drop a leading country code or the
Northern Ireland prefix.

- **Five characters**: the last three must be digits; the number must start with the
  government-department prefix and be below five hundred, or with the health-authority prefix and
  be five hundred or more. No check digit.
- **Eleven characters** whose first six are one of the two eight-eight-eight department forms:
  characters seven to eleven must be digits; the department rule above applies to characters seven
  to nine; and
  ```formula
  valid when ( the integer formed by characters 7..9 ) mod 97 = the integer formed by characters 10..11
  ```
- **Nine or twelve digits**:
  ```formula
  weights  = ( 8 , 7 , 6 , 5 , 4 , 3 , 2 , 10 , 1 )   applied to d1 … d9
  checksum = weighted_sum mod 97
  when the integer formed by d1 d2 d3 is 100 or more:  valid when checksum is 0, 42 or 55
  otherwise:                                           valid when checksum is 0
  ```
- Any other length is invalid.

**Ghana** — clean by removing spaces; uppercase. Eleven characters matching: one letter among
P, C, G, Q, V; two zeros; eight characters that are digits or capital letters.

```formula
check = ( sum over i = 1..9 of i × value of the character at position i + 1 ) mod 11
check_character = "X" when check = 10 , otherwise the decimal digit check
valid when check_character = the eleventh character
```

**Guinea** — clean by removing spaces and hyphens. Nine digits satisfying the doubling checksum.

**Greece** — example `EL123456783`.
Clean by removing spaces, hyphens, full stops, solidi and colons; uppercase; drop a leading union
prefix or country code; left-pad with a zero when eight digits long. Five test numbers are accepted
unconditionally: `047747270`, `047747210`, `047747220`, `117747270`, `127747270`. Otherwise nine
digits and

```formula
checksum = 0
for each of d1 … d8, left to right:  checksum = checksum × 2 + d
check_digit = ( checksum × 2 mod 11 ) mod 10
valid when check_digit = d9
```

**Guatemala** — clean by removing spaces and hyphens; uppercase; trim; remove leading zeros. Two
test numbers, `11201220K` and `11201350K`, and every number matching "nine eight followed by ten
digits followed by the letter K" are accepted unconditionally. Otherwise: between two and twelve
characters; all but the last must be digits; the last must be a digit or the letter K.

```formula
read d1 … d(n−1) from RIGHT to LEFT with the weights 2, 3, 4, … increasing by one
c = ( − weighted_sum ) mod 11
check_character = "K" when c = 10 , otherwise the decimal digit c
valid when check_character = the last character
```

**Croatia** — example `HR01234567896`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Eleven digits
satisfying the recursive modulus eleven over ten (section 15.2.3).

**Hungary** — example `HU12345676`, `12345678-1-11` or `8071592153`.
Valid when **any** of the following holds:

- the number matches eight digits, an optional hyphen, one digit between one and five, an optional
  hyphen and two digits (a company number);
- the number matches an eight followed by nine digits (a natural person's number);
- the number is exactly eight digits (the union form);
- the library check succeeds: clean by removing spaces and hyphens, uppercase, drop a leading
  country code; eight digits and
  ```formula
  weights = ( 9 , 7 , 3 , 1 , 9 , 7 , 3 , 1 )   applied to d1 … d8
  valid when weighted_sum mod 10 = 0
  ```

The union form of a company number is built as the country code followed by the first eight digits.

**Indonesia** — example `1234567890123456`.
The application supplies its own check: clean by removing spaces, hyphens and full stops; trim.
The number must be fifteen or sixteen digits. A sixteen-digit number whose first digit is not zero
is accepted with no further check. Otherwise the doubling checksum must hold over characters one to
nine of a fifteen-digit number, or characters two to ten of a sixteen-digit number.

**Ireland** — example `IE1234567FA`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Eight or nine
characters. Character one and characters three to seven must be digits; characters eight onwards
must belong to the twenty-three-letter alphabet `WABCDEFGHIJKLMNOPQRSTUV`.

```formula
check_letter( seven characters ) :
    left-pad to eight characters with zeros
    s = sum over i = 1..7 of ( 8 − i + 1 ) × digit_i            (weights 8,7,6,5,4,3,2)
      + 9 × value of the eighth character in the alphabet above
    result = the alphabet character at position ( s mod 23 )

when d1 … d7 are all digits:
    valid when d8 = check_letter( d1 … d7 followed by anything after d8 )
when d2 is a letter or a plus sign or an asterisk:
    valid when d8 = check_letter( d3 … d7 followed by d1 )
otherwise invalid
```

**Israel** — example: nine digits satisfying the doubling checksum.
Clean by removing spaces and hyphens; left-pad with zeros to nine characters. At most nine
characters; all digits; strictly positive; and the doubling checksum must be zero.

**India** — example `12AAAAA1234AAZA`.
The application supplies its own check: the number must be exactly fifteen characters and match at
least one of six patterns:

| Pattern | Shape |
|---|---|
| ordinary, composition, casual taxpayer | two digits, five letters, four digits, one letter, one character that is a digit one to nine or a letter, one character among Z, z or the digits one to nine or the letters A to J in either case, one alphanumeric |
| United Nations or other body | four digits, three capitals, five digits, the letter U or O, the letter N, one capital or digit |
| revised non-resident | four digits, three capitals, five digits, three capitals |
| non-resident | four digits, three letters, five digits, the letters N and R, one alphanumeric |
| tax deducted at source | two digits, four letters, one alphanumeric, four digits, one letter, one digit one to nine or letter, the letter D or K, one alphanumeric |
| tax collected at source | two digits, five letters, four digits, one letter, one digit one to nine or letter, the letter C, one alphanumeric |

No check digit is verified by the application's check. (The library check the application replaces
additionally validates the state code, the embedded permanent account number and a doubling
checksum over the thirty-six-character alphabet.)

**Iceland** — example `IS062199`.
Clean by removing spaces; uppercase; drop a leading country code. Five or six digits. No check
digit.

**Italy** — example `IT12345670017`.
Clean by removing spaces, hyphens and colons; uppercase; drop a leading country code. Eleven
digits; the first seven may not all be zero; the office code formed by digits eight to ten must lie
between `001` and `100` inclusive or be one of `120`, `121`, `888`, `999`; and the doubling
checksum over the eleven digits must be zero.

**Japan** — example `T7000012050002`.
The application first drops a leading capital T, then applies the library check: clean by removing
hyphens and spaces; thirteen digits;

```formula
weights = ( 1 , 2 , 1 , 2 , 1 , 2 , 1 , 2 , 1 , 2 , 1 , 2 )
          applied to d2 … d13 read from RIGHT to LEFT
s = weighted_sum mod 9
check_digit = 9 − s
valid when check_digit = d1
```

**Kenya** — clean by removing spaces and hyphens; uppercase. Eleven characters matching: the letter
A or P, nine digits, one capital letter. No check digit.

**Korea** — example `123-45-67890` or `1234567890`.
Clean by removing spaces and hyphens. Ten digits. The first three read as a number must be at least
`101`; digits four and five may not both be zero; digits six to nine may not all be zero. No check
digit.

**Lithuania** — example `LT123456715`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. All digits. Nine
digits with a one in position eight, or twelve digits with a one in position eleven.

```formula
check = ( sum over i = 1..n−1 of ( 1 + ( ( i − 1 ) mod 9 ) ) × di ) mod 11
if check = 10 :
    check = sum over i = 1..n−1 of ( 1 + ( ( i + 1 ) mod 9 ) ) × di
check_digit = ( check mod 11 ) mod 10
valid when check_digit = dn
```

**Luxembourg** — example `LU12345613`.
Clean by removing spaces, colons, full stops and hyphens; uppercase; drop a leading country code.
Eight digits.

```formula
check_digits = ( the integer formed by d1 … d6 ) mod 89 , written on two digits
valid when check_digits = d7 d8
```

**Latvia** — example `LV41234567891`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Eleven digits.

- First digit greater than three (a legal person):
  ```formula
  weights = ( 9 , 1 , 4 , 8 , 3 , 10 , 2 , 5 , 7 , 6 , 1 )   applied to d1 … d11
  valid when weighted_sum mod 11 = 3
  ```
- Otherwise (a natural person; when the number does not start with three two, the first six digits
  must also form a valid date of birth whose century is given by the seventh digit as eighteen
  hundred plus one hundred times that digit):
  ```formula
  weights = ( 10 , 5 , 8 , 4 , 2 , 1 , 6 , 3 , 7 , 9 )   applied to d1 … d10
  check_digit = ( ( 1 + weighted_sum ) mod 11 ) mod 10
  valid when check_digit = d11
  ```

**Morocco** — example `12345678`.
The application supplies its own check: the number must be exactly eight digits. No check digit.
(The library check the application replaces requires fifteen digits satisfying the modulus
ninety-seven over ten.)

**Monaco** — the French rules apply, with the extra requirement that characters three to five be
three zeros; the normalised number carries the French country prefix.

**Montenegro** — clean by removing spaces. Eight digits.

```formula
weights = ( 8 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d7
check_digit = ( ( − weighted_sum ) mod 11 ) mod 10
valid when check_digit = d8
```

**North Macedonia** — clean by removing spaces and hyphens; uppercase; drop a leading country code
written in either alphabet. Thirteen digits.

```formula
weights = ( 7 , 6 , 5 , 4 , 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d12
check_digit = ( ( − weighted_sum ) mod 11 ) mod 10
valid when check_digit = d13
```

**Malta** — example `MT12345634`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Eight digits, first
digit not zero.

```formula
weights = ( 3 , 4 , 6 , 7 , 8 , 9 , 10 , 1 )   applied to d1 … d8
valid when weighted_sum mod 37 = 0
```

**Mexico** — example `GODE561231GR8`.
The application supplies its own check. The number must match, in full:

```
three or four letters (capital or small, including the Spanish n with a tilde and the ampersand)
optional space, hyphen or underscore
two digits for the year, two digits for the month (first digit zero or one),
two digits for the day (first digit zero to three)
optional space, hyphen or underscore
three characters that are letters, digits, the ampersand or the Spanish n with a tilde
```

The year is interpreted as nineteen hundred plus the two digits when they exceed thirty, and two
thousand plus the two digits otherwise; the resulting year, month and day must form a real calendar
date. No check digit is verified.

**Mozambique** — clean by removing spaces, hyphens and full stops. Nine digits.

```formula
weights = ( 8 , 9 , 4 , 5 , 6 , 7 , 8 , 9 )   applied to d1 … d8
check = weighted_sum mod 11
check_digit = the character at position check of the string "01234567891"
valid when check_digit = d9
```

**Netherlands** — example `NL123456782B90`.
Clean by removing spaces, hyphens and full stops; uppercase; drop a leading country code; left-pad
the part before the last three characters with zeros to nine digits. Twelve characters. Characters
one to nine must be digits forming a strictly positive number; character ten must be the letter B;
characters eleven and twelve must be digits forming a strictly positive number.

Valid when **either**:

```formula
the citizen service number check on d1 … d9 succeeds:
    ( sum over i = 1..8 of ( 9 − i + 1 ) × di ) − d9   is a multiple of 11
        that is, weights ( 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 ) then subtract the last digit
or the modulus ninety-seven over ten of the two-letter country code followed by the whole
   twelve-character number equals one
```

**Norway** — example `NO123456785`.
The application supplies its own check: an optional three-letter tax-regime suffix is dropped when
the number is twelve characters long and ends with it; the remainder must be exactly nine digits.

```formula
weights = ( 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d8
check = 11 − ( weighted_sum mod 11 )
if check = 11 :  check = 0
if check = 10 :  the number is invalid
valid when check = d9
```

**New Zealand** — example `49-098-576` or `49098576`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Eight or nine digits;
the number read as an integer must lie strictly between ten million and one hundred fifty million.

```formula
left-pad the digits before the last one to eight characters with zeros
primary_weights   = ( 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )
s = ( − weighted_sum with the primary weights ) mod 11
if s ≠ 10 :  check_digit = s
otherwise :
    secondary_weights = ( 7 , 4 , 3 , 2 , 5 , 2 , 7 , 6 )
    check_digit = ( − weighted_sum with the secondary weights ) mod 11
valid when check_digit = the last digit
```

**Peru** — example `10XXXXXXXXY`, `20…`, `15…`, `16…` or `17…`.
The application supplies its own check: exactly eleven digits.

```formula
weights = the digits of the string "5432765432" , that is ( 5 , 4 , 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )
          applied to d1 … d10
check = 11 − ( weighted_sum mod 11 )
if check = 10 :  check = 0
if check = 11 :  check = 1
valid when check = d11
```

(The library check the application replaces additionally restricts the first two digits to 10, 15,
17 or 20 and folds the remainder differently.)

**Philippines** — example `123-456-789-123`.
The application supplies its own check: between eleven and seventeen characters, matching three
groups of three digits separated by hyphens, optionally followed by a hyphen and a branch code of
three to five digits. No check digit.

**Poland** — example `PL1234567883`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Ten digits.

```formula
weights = ( 6 , 5 , 7 , 2 , 3 , 4 , 5 , 6 , 7 , −1 )   applied to d1 … d10
valid when weighted_sum mod 11 = 0
```

**Portugal** — example `PT123456789`.
Clean by removing spaces, hyphens and full stops; uppercase; drop a leading country code. Nine
digits, first digit not zero.

```formula
weights = ( 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d8
check_digit = ( ( 11 − weighted_sum ) mod 11 ) mod 10
valid when check_digit = d9
```

**Paraguay** — clean by removing spaces and hyphens; uppercase. At most nine digits.

```formula
read d1 … d(n−1) from RIGHT to LEFT with the weights 2, 3, 4, … increasing by one
check_digit = ( ( − weighted_sum ) mod 11 ) mod 10
valid when check_digit = dn
```

**Romania** — example `RO1234567897`, a thirteen-digit personal number, or a nine-digit code
prefixed by nine thousand.
Valid when **any** of the following holds:

- the number matches: one digit one to nine, two digits for the year, a month between `01` and
  `12`, a day between `01` and `31`, six further digits (a natural person's number);
- the number matches four thousand-nine hundred, that is the four characters `9000`, followed by
  nine digits;
- the library check succeeds: clean by removing spaces and hyphens, uppercase, drop a leading
  country code. A thirteen-digit number must be a valid national personal number; a number of two
  to ten characters must satisfy
  ```formula
  left-pad the digits before the last one to nine characters with zeros
  weights = ( 7 , 5 , 3 , 2 , 1 , 7 , 5 , 3 , 2 )
  check_digit = ( ( 10 × weighted_sum ) mod 11 ) mod 10
  valid when check_digit = the last digit
  ```
  with the additional requirement that the first character is not a zero.

The national personal number is thirteen digits whose first digit is one to nine, whose digits two
to seven form a valid date of birth (the century coming from the first digit: one and two mean the
nineteen hundreds, three and four the eighteen hundreds, five and six the two thousands), whose
digits eight and nine name a recognised county, and whose check digit is

```formula
weights = ( 2 , 7 , 9 , 1 , 4 , 6 , 3 , 5 , 8 , 2 , 7 , 9 )   applied to d1 … d12
check = weighted_sum mod 11
check_digit = 1 when check = 10 , otherwise check
```

**Serbia** — example `RS101134702`.
The application first drops a leading country code, then applies the library check: clean by
removing spaces, hyphens and full stops; nine digits satisfying the recursive modulus eleven over
ten (section 15.2.3).

**Russia** — example `123456789047`.
The application supplies its own check: ten or twelve digits.

- **Ten digits**:
  ```formula
  weights = ( 2 , 4 , 10 , 3 , 5 , 9 , 4 , 6 , 8 )   applied to d1 … d9
  valid when ( weighted_sum mod 11 ) mod 10 = d10
  ```
- **Twelve digits**:
  ```formula
  weights1 = ( 7 , 2 , 4 , 10 , 3 , 5 , 9 , 4 , 6 , 8 )   applied to d1 … d10
  valid so far when weighted_sum1 mod 11 = d11
  weights2 = ( 3 , 7 , 2 , 4 , 10 , 3 , 5 , 9 , 4 , 6 , 8 )   applied to d1 … d11
  valid when weighted_sum2 mod 11 = d12
  ```
  Note that the application's check compares the raw remainder, not the remainder folded to a
  single digit; a remainder of ten therefore fails.

**Sweden** — example `SE123456789701`.
Clean by removing spaces, hyphens and full stops; uppercase; drop a leading country code. All
digits; the last two must be `01`; and the first ten must be exactly ten digits satisfying the
doubling checksum.

**Singapore** — clean by removing white space; uppercase. Nine or ten characters.

- **Nine characters** (a business): the first eight must be digits, the ninth a letter;
  ```formula
  weights = ( 10 , 4 , 9 , 3 , 8 , 2 , 7 , 1 )   applied to d1 … d8
  check_letter = the character at position ( weighted_sum mod 11 ) of "XMKECAWLJDB"
  valid when check_letter = the ninth character
  ```
- **Ten characters beginning with a digit** (a locally incorporated company): the first nine must
  be digits, the first four may not exceed the current year;
  ```formula
  weights = ( 10 , 8 , 6 , 4 , 9 , 7 , 5 , 3 , 1 )   applied to d1 … d9
  check_letter = the character at position ( weighted_sum mod 11 ) of "ZKCMDNERGWH"
  valid when check_letter = the tenth character
  ```
- **Ten characters beginning with a letter** (another entity): the first must be R, S or T;
  characters two and three must be digits and, when the first is T, may not exceed the last two
  digits of the current year; characters four and five must name one of the thirty-eight recognised
  entity kinds; characters six to nine must be digits;
  ```formula
  alphabet = "ABCDEFGHJKLMNPQRSTUVWX0123456789"
  weights  = ( 4 , 3 , 5 , 3 , 10 , 2 , 2 , 5 , 7 )   applied to the first nine characters
  check_character = alphabet[ ( weighted_sum − 5 ) mod 11 ]
  valid when check_character = the tenth character
  ```

**Slovenia** — example `SI12345679`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Eight digits, not
starting with zero.

```formula
weights = ( 8 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d7
check = 11 − ( weighted_sum mod 11 )
check_digit = 0 when check = 10 , otherwise check
valid when check_digit = d8
```

**Slovakia** — example `SK2022749619`.
Clean by removing spaces and hyphens; uppercase; drop a leading country code. Ten digits. Valid
when the number is a valid national personal number, or when the first digit is not zero, the third
digit is one of two, three, four, seven, eight or nine, and

```formula
valid when ( the integer formed by the ten digits ) mod 11 = 0
```

**San Marino** — example `SM24165`.
Clean by removing full stops; trim; remove leading zeros. One to five digits. When fewer than three
digits remain, the number read as an integer must belong to a published list of sixty-five
historical low numbers. No check digit.

**Senegal** — clean by removing spaces, hyphens, solidi and commas; uppercase. When more than nine
characters, the last three form a tax-regime suffix and are removed for the length and checksum
tests; the remainder must be seven or nine digits. A suffix, when present, must be: a first
character among zero, one and two; a second character among the twenty-two capital letters
excluding I, O, X and Y; and a third character that is a digit.

```formula
left-pad the number (without suffix) to nine digits with zeros
weights = ( 1 , 2 , 1 , 2 , 1 , 2 , 1 , 2 , 1 )
valid when weighted_sum mod 10 = 0
```

**El Salvador** — clean by removing spaces and hyphens; uppercase; drop a leading country code.
Fourteen digits; the first must be zero, one or nine.

```formula
when the three characters at positions 11..13 are lexicographically at most "100" :
    weights = ( 14 , 13 , 12 , 11 , 10 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )
    check_digit = ( weighted_sum mod 11 ) mod 10
otherwise :
    weights = ( 2 , 7 , 6 , 5 , 4 , 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )
    check_digit = ( ( − weighted_sum ) mod 11 ) mod 10
valid when check_digit = d14
```

**Thailand** — example `1234545678781`.
The check accepts either of two thirteen-digit forms, trying them in order:

- *the taxpayer form*: thirteen digits, first digit zero;
- *the personal form*: thirteen digits, first digit neither zero nor nine.

Both use the same check digit:

```formula
s = ( sum over i = 1..12 of ( 2 − i + 1 ) × di ) mod 11
    that is, weights 13, 12, 11, …, 2 applied to d1 … d12
check_digit = ( 1 − s ) mod 10
valid when check_digit = d13
```

**Tunisia** — clean by removing spaces, solidi, full stops and hyphens; uppercase; left-pad the
leading run of digits to seven characters with zeros. Eight or thirteen characters. The first seven
must be digits; the eighth must be one of the twenty-three permitted control letters (the Latin
alphabet without I, O and U). For a thirteen-character number: the ninth character must be one of
A, P, B, D, N; the tenth one of M, P, C, N, E; characters eleven to thirteen must be digits; and
those three digits must be three zeros unless the tenth character is E. No check digit.

**Turkey** — example: eleven digits for a natural person, or ten digits for a company.
Valid when **either** check succeeds.

*Natural person, eleven digits*, first digit not zero:

```formula
check1 = ( 10 − ( sum over i = 1..9 of ( 3 when i is odd , 1 when i is even ) × di ) ) mod 10
check2 = ( check1 + sum over i = 1..9 of di ) mod 10
valid when check1 check2 = d10 d11
```

*Company, ten digits*:

```formula
s = 0
for i = 1 … 9 , where n is the i-th digit counted from the RIGHT of d1 … d9 :
    c1 = ( n + i ) mod 10
    when c1 is not zero :
        c2 = ( c1 × 2^i ) mod 9 , replaced by 9 when that remainder is zero
        s = s + c2
check_digit = ( 10 − s ) mod 10
valid when check_digit = d10
```

**Taiwan** — the application supplies its own check, updated for the current national rule: clean
by removing spaces and hyphens. Exactly eight digits.

```formula
multipliers = ( 1 , 2 , 1 , 2 , 1 , 2 , 4 , 1 )
products    = for each i , the decimal writing of multiplier_i × di
digit_sum( S ) = the sum of the individual digits of every product in S

when d7 ≠ 7 :
    valid when digit_sum( all eight products ) mod 5 = 0
when d7 = 7 :
    base = digit_sum( the products of positions 1..6 and 8 )
    valid when ( base + 1 ) mod 5 = 0  or  base mod 5 = 0
```

The divisor is five, not ten: the national authority relaxed the rule when the number space neared
exhaustion, so numbers that were previously invalid are now valid.

**Ukraine** — example: eight digits, eight digits with the country prefix, ten digits, or twelve
digits.
The application supplies its own check: drop a leading country code, then accept when the remaining
length is eight, ten or twelve. No check digit.

**Uruguay** — example `219999830019`.
The application supplies its own check: clean by removing spaces and hyphens; uppercase; trim; drop
a leading country code. Twelve digits. Characters one and two must lie between `01` and `22`
inclusive; characters three to eight must not all be zero; characters nine to eleven must be `001`.

```formula
weights = ( 4 , 3 , 2 , 9 , 8 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to d1 … d11
check_digit = ( − weighted_sum ) mod 11
valid when check_digit = d12
```

**Uzbekistan** — example: nine digits for a company, fourteen for an individual.
The application supplies its own check: the number must be all digits, and its length must be
exactly nine when the partner is a company and exactly fourteen otherwise. No check digit.

**Venezuela** — example `V-12345678-1`, `V123456781` or `V-12.345.678-1`.
The application supplies its own check. The number must match, in full and ignoring letter case:
one kind letter among V, E, C, J, P, G; then an eight-digit identifier written either plainly, or
as two digits, three digits and three digits separated by full stops, the whole optionally wrapped
in hyphens (the separators must be used consistently: a leading hyphen requires a trailing hyphen,
and a first full stop requires a second); then one check digit.

```formula
kind_digit =  1 for V (citizen) , 2 for E (foreigner) , 3 for C or J (council or legal entity) ,
              4 for P (passport) , 5 for G (government)
multipliers = ( 3 , 2 , 7 , 6 , 5 , 4 , 3 , 2 )   applied to the eight identifier digits
checksum    = kind_digit × 4 + weighted_sum
check_digit = 11 − ( checksum mod 11 )
if check_digit > 9 :  check_digit = 0
valid when check_digit = the last digit
```

**Vietnam** — the application supplies its own check: trim, then accept a ten-digit number,
optionally followed by a three-digit branch code with or without a hyphen, or a twelve-digit
personal identity number. No check digit.

(The library check the application replaces requires ten or thirteen digits, forbids seven zeros in
positions three to nine and a three-zero branch code, and verifies
`check_digit = 10 − ( weighted_sum with weights 31, 29, 23, 19, 17, 13, 7, 5, 3 over d1 … d9 ) mod 11`
against the tenth digit.)

**Northern Ireland** — example `XI123456782`. The United Kingdom rules apply; the prefix is
retained so that the number is recognised as belonging to the union register.

**Saudi Arabia** — example `310175397400003`.
The application supplies its own check: the number must match exactly fifteen digits beginning and
ending with a three. No check digit.

### 15.5 The cross-border verification service

*Server-only.* Beyond the syntactic check, a partner's number may be verified against the union's
central register through a relay service.

**Configuration.** A company-level switch turns it on. The relay endpoint is either a production
address or a test address; which one is the default depends on whether the validation capability
was installed with demonstration data; an administrator may override it through a system parameter
but only with one of those two values, otherwise: `Invalid IAP VIES endpoint`.

**Credentials.** The database identifies itself with a pair (identifier, token). If none is stored,
a random universally unique identifier and a random token are generated and stored in their own
transaction, so that an error later in the current transaction cannot lose them. When the periodic
job does not exist, or the run is a test run, a fixed placeholder pair is used and the relay
ignores it.

**When the check runs.** A stored flag "intra-community valid" is recomputed whenever the number
changes. If no company at all has the switch on, the flag is set to false without any call. A
partner whose parent carries the same number inherits the parent's flag. Otherwise one request is
sent.

**The request.** A form-encoded request to the relay's validity endpoint carrying: the number, the
database's unique identifier, the client identifier, the client token, a callback address formed as
the base address of this installation followed by the path
`/base_vat/1/webhook_update_vies`, and a signed callback token built from the fixed text
`vies_check` and the number, valid for seven days. The request times out after twenty seconds. A
transport failure, or a response without a status, yields the status *fault*.

**The statuses and what they mean.**

| Status | Flag | Message logged on the partner |
|---|---|---|
| `valid` | true | *"The Intra-Community validity has been updated to: valid."* |
| `unassigned` | false | *"The Intra-Community validity has been updated to: unassigned."* |
| `pending` | false | `The VIES check is pending. The status will be updated soon.` |
| `fault` | false | `The VIES check failed. Please check the Tax ID manually.` |

**The callback.** The relay may later call the callback address with the number and a status; the
receiving route re-verifies the signed token before applying the status.

**The periodic job.** A scheduled job asks the relay for updates on numbers previously reported as
pending, receives a table of number to status, groups the partners by number and applies each
status.

**Suppression during import.** When records are created or written as part of a file import, the
recomputation of the flag is cancelled, so that importing ten thousand partners does not issue ten
thousand requests.

**Effect on fiscal positions.** A fiscal position that requires a tax registration accepts a
partner only when the partner has a number **and**, when cross-border verification applies to that
partner for the acting company, the flag is true. Verification applies when the company has a
country, the partner's number does not start with the company's fiscal country code, the company's
switch is on, and either the company's country belongs to the union or the partner's country is one
in which some company holds a foreign registration.

### 15.6 Worked verifications

Each walk-through takes the example number the error message quotes, shows the cleaning, the
arithmetic and the comparison. They are the fastest way to confirm an implementation of section
15.4.

**Argentina — `20055361682`.** Eleven digits; the first two are `20`, which is in the permitted
list.

```formula
5×2 + 4×0 + 3×0 + 2×5 + 7×5 + 6×3 + 5×6 + 4×1 + 3×6 + 2×8
  = 10 + 0 + 0 + 10 + 35 + 18 + 30 + 4 + 18 + 16 = 141
141 mod 11 = 9
character at position 11 − 9 = 2 of "012345678990" is "2"
last digit = 2  →  valid
```

**Austria — `ATU12345675`.** The prefix is dropped, leaving `U12345675`: the letter U followed by
eight digits.

```formula
doubling_checksum( "1234567" ) = 1
check_digit = ( 6 − 1 ) mod 10 = 5
last digit = 5  →  valid
```

**Belgium — `BE0477472701`.** The prefix is dropped, leaving ten digits beginning with zero.

```formula
first eight digits as an integer = 4 774 727
last two digits as an integer    =         1
( 4 774 727 + 1 ) mod 97 = 4 774 728 mod 97 = 0  →  valid
```

**Bulgaria — `BG1234567892`.** Ten digits, so the three alternatives are tried; the third succeeds.

```formula
4×1 + 3×2 + 2×3 + 7×4 + 6×5 + 5×6 + 4×7 + 3×8 + 2×9
  = 4 + 6 + 6 + 28 + 30 + 30 + 28 + 24 + 18 = 174
174 mod 11 = 9
check_digit = ( 11 − 174 ) mod 11 = ( −163 ) mod 11 = 2
last digit = 2  →  valid
```

**Chile — `76086428-5`.** The hyphen is removed and re-inserted by the normaliser; the body is
`76086428`.

```formula
reading the body right to left with the cycling weights 2,3,4,5,6,7,2,3 …
  the actual weights used are, from the rightmost digit: 9,8,7,6,5,4,9,8
  (the expression 4 + ((5 − i) mod 6) for i = 0,1,2,… gives 9,8,7,6,5,4,9,8)
9×8 + 8×2 + 7×4 + 6×6 + 5×8 + 4×0 + 9×6 + 8×7
  = 72 + 16 + 28 + 36 + 40 + 0 + 54 + 56 = 302
302 mod 11 = 5
character at position 5 of "0123456789K" is "5"
check character = 5  →  valid
```

**Colombia — `213123432-1`.** The body is `213123432`.

```formula
weights 3,7,13,17,19,23,29,37,41 applied right to left
3×2 + 7×3 + 13×4 + 17×3 + 19×2 + 23×1 + 29×3 + 37×1 + 41×2
  = 6 + 21 + 52 + 51 + 38 + 23 + 87 + 37 + 82 = 397
397 mod 11 = 1
character at position 1 of "01987654321" is "1"
check digit = 1  →  valid
```

**Cyprus — `CY10259033P`.** The body is `10259033`, the check character `P`.

```formula
translate the digits at the odd positions 1,3,5,7 → 1,2,9,3 become 0,5,21,7
sum of translated odd positions = 0 + 5 + 21 + 7 = 33
sum of the even positions 0,5,0,3                =  8
total = 41 ;  41 mod 26 = 15
letter at position 15 of the alphabet = "P"  →  valid
```

**Czechia — `CZ12345679`.** Eight digits, not starting with nine.

```formula
8×1 + 7×2 + 6×3 + 5×4 + 4×5 + 3×6 + 2×7 = 8 + 14 + 18 + 20 + 20 + 18 + 14 = 112
112 mod 11 = 2
check = ( 11 − 112 ) mod 11 = 9 ;  ( 9 or 1 ) mod 10 = 9
last digit = 9  →  valid
```

**Denmark — `DK12345674`.**

```formula
2×1 + 7×2 + 6×3 + 5×4 + 4×5 + 3×6 + 2×7 + 1×4
  = 2 + 14 + 18 + 20 + 20 + 18 + 14 + 4 = 110
110 mod 11 = 0  →  valid
```

**Estonia — `EE123456780`.**

```formula
3×1 + 7×2 + 1×3 + 3×4 + 7×5 + 1×6 + 3×7 + 7×8 + 1×0
  = 3 + 14 + 3 + 12 + 35 + 6 + 21 + 56 + 0 = 150
150 mod 10 = 0  →  valid
```

**Finland — `FI12345671`.**

```formula
7×1 + 9×2 + 10×3 + 5×4 + 8×5 + 4×6 + 2×7 + 1×1
  = 7 + 18 + 30 + 20 + 40 + 24 + 14 + 1 = 154
154 mod 11 = 0  →  valid
```

**France — `FR23334175221`.** Both leading characters are digits, so the numeric branch applies.

```formula
digits 3..11 = 334175221 ;  append "12" → 33417522112
33 417 522 112 mod 97 = 23
first two digits = 23  →  valid
and, because digits 3..5 are not "000", 334175221 must also satisfy
the doubling checksum as a company registration number, which it does
```

**Germany — `DE123456788`.** Nine digits, first digit not zero; the recursive modulus eleven over
ten.

```formula
check starts at 5
after "1":  ((5 × 2) mod 11 + 1) mod 10 = (10 + 1) mod 10 = 1
after "2":  ((1 × 2) mod 11 + 2) mod 10 = (2 + 2) mod 10 = 4
after "3":  ((4 × 2) mod 11 + 3) mod 10 = (8 + 3) mod 10 = 1
after "4":  ((1 × 2) mod 11 + 4) mod 10 = (2 + 4) mod 10 = 6
after "5":  ((6 × 2) mod 11 + 5) mod 10 = (1 + 5) mod 10 = 6
after "6":  ((6 × 2) mod 11 + 6) mod 10 = (1 + 6) mod 10 = 7
after "7":  ((7 × 2) mod 11 + 7) mod 10 = (3 + 7) mod 10 = 0
after "8":  ((10 × 2) mod 11 + 8) mod 10 = (9 + 8) mod 10 = 7
after "8":  ((7 × 2) mod 11 + 8) mod 10 = (3 + 8) mod 10 = 1
final = 1  →  valid
```

Note the "or ten" rule in the third line from the end: when the running value is zero it is read
as ten before doubling.

**Greece — `EL123456783`.** The union prefix is dropped, leaving nine digits.

```formula
running = 0
for each of 1,2,3,4,5,6,7,8 :  running = running × 2 + digit
  → 1, 4, 11, 26, 57, 120, 247, 502
check_digit = ( 502 × 2 mod 11 ) mod 10 = ( 1004 mod 11 ) mod 10 = 3 mod 10 = 3
last digit = 3  →  valid
```

**Hungary — `HU12345676`.** Eight digits.

```formula
9×1 + 7×2 + 3×3 + 1×4 + 9×5 + 7×6 + 3×7 + 1×6
  = 9 + 14 + 9 + 4 + 45 + 42 + 21 + 6 = 150
150 mod 10 = 0  →  valid
```

**Ireland — `IE1234567FA`.** Nine characters; the first seven are digits, so the first branch
applies and the check character is computed on the seven digits followed by the ninth character.

```formula
the value fed to the check routine is "1234567" + "A" = "1234567A", already eight characters
8×1 + 7×2 + 6×3 + 5×4 + 4×5 + 3×6 + 2×7 = 8 + 14 + 18 + 20 + 20 + 18 + 14 = 112
plus 9 × the position of "A" in "WABCDEFGHIJKLMNOPQRSTUV" = 9 × 1 = 9
total = 121 ;  121 mod 23 = 6
character at position 6 of that alphabet = "F"
eighth character = "F"  →  valid
```

**Italy — `IT12345670017`.** Eleven digits; the office code `001` lies in the permitted range; the
doubling checksum over the eleven digits is zero.

**Latvia — `LV41234567891`.** Eleven digits whose first digit is four, greater than three, so the
legal-person branch applies.

```formula
9×4 + 1×1 + 4×2 + 8×3 + 3×4 + 10×5 + 2×6 + 5×7 + 7×8 + 6×9 + 1×1
  = 36 + 1 + 8 + 24 + 12 + 50 + 12 + 35 + 56 + 54 + 1 = 289
289 mod 11 = 3  →  valid (the required remainder for a legal person is three, not zero)
```

**Lithuania — `LT123456715`.** Nine digits with a one in position eight.

```formula
weights 1,2,3,4,5,6,7,8,9 cycling every nine, applied to "12345671"
1×1 + 2×2 + 3×3 + 4×4 + 5×5 + 6×6 + 7×7 + 8×1
  = 1 + 4 + 9 + 16 + 25 + 36 + 49 + 8 = 148
148 mod 11 = 5 , which is not ten, so no second pass
check_digit = ( 5 mod 11 ) mod 10 = 5
last digit = 5  →  valid
```

**Luxembourg — `LU12345613`.**

```formula
first six digits as an integer = 123 456
123 456 mod 89 = 13 , written on two digits as "13"
last two digits = "13"  →  valid
```

**Malta — `MT12345634`.**

```formula
3×1 + 4×2 + 6×3 + 7×4 + 8×5 + 9×6 + 10×3 + 1×4
  = 3 + 8 + 18 + 28 + 40 + 54 + 30 + 4 = 185
185 mod 37 = 0  →  valid
```

**Netherlands — `NL123456782B90`.** Twelve characters; the tenth is the letter B; the first nine
form a strictly positive number; the last two form a strictly positive number.

```formula
the citizen-service-number branch on "123456782" :
9×1 + 8×2 + 7×3 + 6×4 + 5×5 + 4×6 + 3×7 + 2×8 = 9 + 16 + 21 + 24 + 25 + 24 + 21 + 16 = 156
156 − 2 = 154 ;  154 mod 11 = 0  →  valid
```

**Norway — `NO123456785`.** Nine digits after any three-letter suffix is dropped.

```formula
3×1 + 2×2 + 7×3 + 6×4 + 5×5 + 4×6 + 3×7 + 2×8
  = 3 + 4 + 21 + 24 + 25 + 24 + 21 + 16 = 138
138 mod 11 = 6 ;  check = 11 − 6 = 5 , neither eleven nor ten
last digit = 5  →  valid
```

**Peru — a constructed example.** Eleven digits beginning with `20`.

```formula
body = 2010006660
5×2 + 4×0 + 3×1 + 2×0 + 7×0 + 6×0 + 5×6 + 4×6 + 3×6 + 2×0
  = 10 + 0 + 3 + 0 + 0 + 0 + 30 + 24 + 18 + 0 = 85
85 mod 11 = 8 ;  check = 11 − 8 = 3 , neither ten nor eleven
the complete number is 20100066603
```

**Poland — `PL1234567883`.**

```formula
6×1 + 5×2 + 7×3 + 2×4 + 3×5 + 4×6 + 5×7 + 6×8 + 7×8 + (−1)×3
  = 6 + 10 + 21 + 8 + 15 + 24 + 35 + 48 + 56 − 3 = 220
220 mod 11 = 0  →  valid
```

The negative weight on the last digit is what turns the usual "compare to a check digit" into a
"the whole thing is a multiple of eleven" test.

**Portugal — `PT123456789`.**

```formula
9×1 + 8×2 + 7×3 + 6×4 + 5×5 + 4×6 + 3×7 + 2×8
  = 9 + 16 + 21 + 24 + 25 + 24 + 21 + 16 = 156
156 mod 11 = 2 ;  check = ( ( 11 − 156 ) mod 11 ) mod 10 = ( (−145) mod 11 ) mod 10 = 9
last digit = 9  →  valid
```

**Romania — `RO1234567897`.** Nine-digit body, so the company branch applies.

```formula
left-pad "12345678" to nine characters → "012345678"
weights 7,5,3,2,1,7,5,3,2
7×0 + 5×1 + 3×2 + 2×3 + 1×4 + 7×5 + 5×6 + 3×7 + 2×8
  = 0 + 5 + 6 + 6 + 4 + 35 + 30 + 21 + 16 = 123
check = ( ( 10 × 123 ) mod 11 ) mod 10 = ( 1230 mod 11 ) mod 10 = 9 mod 10 = 9
```

The example quoted by the message is `RO1234567897`, whose body is `123456789` and whose check
digit is `7`; running the same arithmetic on that body gives:

```formula
left-pad "123456789" to nine characters → already nine
7×1 + 5×2 + 3×3 + 2×4 + 1×5 + 7×6 + 5×7 + 3×8 + 2×9
  = 7 + 10 + 9 + 8 + 5 + 42 + 35 + 24 + 18 = 158
check = ( ( 10 × 158 ) mod 11 ) mod 10 = ( 1580 mod 11 ) mod 10 = 7  →  valid
```

**Russia — `123456789047`.** Twelve digits, so both check digits are verified.

```formula
weights1 = 7,2,4,10,3,5,9,4,6,8 over "1234567890"
7×1 + 2×2 + 4×3 + 10×4 + 3×5 + 5×6 + 9×7 + 4×8 + 6×9 + 8×0
  = 7 + 4 + 12 + 40 + 15 + 30 + 63 + 32 + 54 + 0 = 257
257 mod 11 = 4  =  the eleventh digit  →  first check passes

weights2 = 3,7,2,4,10,3,5,9,4,6,8 over "12345678904"
3×1 + 7×2 + 2×3 + 4×4 + 10×5 + 3×6 + 5×7 + 9×8 + 4×9 + 6×0 + 8×4
  = 3 + 14 + 6 + 16 + 50 + 18 + 35 + 72 + 36 + 0 + 32 = 282
282 mod 11 = 7  =  the twelfth digit  →  valid
```

**Serbia — `RS101134702`.** The country prefix is dropped, leaving nine digits; the recursive
modulus eleven over ten gives a final value of one.

**Slovakia — `SK2022749619`.** Ten digits, first digit not zero, third digit is two which is in the
permitted set.

```formula
2 022 749 619 mod 11 = 0  →  valid
```

**Slovenia — `SI12345679`.**

```formula
8×1 + 7×2 + 6×3 + 5×4 + 4×5 + 3×6 + 2×7 = 112
112 mod 11 = 2 ;  check = 11 − 2 = 9 , not ten
last digit = 9  →  valid
```

**Spain — `ESA12345674`.** First character `A`, so the legal-person branch applies.

```formula
doubling_check_digit( "1234567" ) = 4
the last character must be "4" or the letter at position 4 of "JABCDEFGHI", which is "D"
last character = "4"  →  valid
```

**Sweden — `SE123456789701`.** The last two characters are `01`; the first ten digits must satisfy
the doubling checksum, and `1234567897` does.

**Switzerland — `CHE-123.456.788 TVA`.** The application's own check extracts the nine digits
`123456788`.

```formula
5×1 + 4×2 + 3×3 + 2×4 + 7×5 + 6×6 + 5×7 + 4×8
  = 5 + 8 + 9 + 8 + 35 + 36 + 35 + 32 = 168
168 mod 11 = 3 ;  check = ( 11 − 3 ) mod 11 = 8
ninth digit = 8  →  valid
```

**United Kingdom — `GB123456782`.** Nine digits; the first three read as `123`, which is at least
one hundred, so the relaxed acceptance applies.

```formula
8×1 + 7×2 + 6×3 + 5×4 + 4×5 + 3×6 + 2×7 + 10×8 + 1×2
  = 8 + 14 + 18 + 20 + 20 + 18 + 14 + 80 + 2 = 194
194 mod 97 = 0 , which is one of the three accepted remainders  →  valid
```

**Japan — `T7000012050002`.** The leading `T` is dropped, leaving thirteen digits.

```formula
weights 1,2,1,2,… applied to digits 2..13 read RIGHT to LEFT:
digits 2..13 = 000012050002 ; read right to left: 2,0,0,0,5,0,2,1,0,0,0,0
1×2 + 2×0 + 1×0 + 2×0 + 1×5 + 2×0 + 1×2 + 2×1 + 1×0 + 2×0 + 1×0 + 2×0
  = 2 + 0 + 0 + 0 + 5 + 0 + 2 + 2 = 11
11 mod 9 = 2 ;  check = 9 − 2 = 7
first digit = 7  →  valid
```

**Venezuela — `V-12345678-1`.** The kind letter V gives a kind digit of one.

```formula
checksum = 1 × 4 + ( 3×1 + 2×2 + 7×3 + 6×4 + 5×5 + 4×6 + 3×7 + 2×8 )
         = 4 + ( 3 + 4 + 21 + 24 + 25 + 24 + 21 + 16 ) = 4 + 138 = 142
142 mod 11 = 10 ;  check = 11 − 10 = 1 , not greater than nine
last digit = 1  →  valid
```

**Uruguay — `219999830019`.** Twelve digits; the first two lie between `01` and `22`; characters
three to eight are not all zero; characters nine to eleven are `001`.

```formula
weights 4,3,2,9,8,7,6,5,4,3,2 over "21999983001"
4×2 + 3×1 + 2×9 + 9×9 + 8×9 + 7×9 + 6×8 + 5×3 + 4×0 + 3×0 + 2×1
  = 8 + 3 + 18 + 81 + 72 + 63 + 48 + 15 + 0 + 0 + 2 = 310
check = ( − 310 ) mod 11 = 9
last digit = 9  →  valid
```

**Brazil, legal person — a constructed example.**

```formula
body = 112223330001
first check digit:
  5×1 + 4×1 + 3×2 + 2×2 + 9×2 + 8×3 + 7×3 + 6×3 + 5×0 + 4×0 + 3×0 + 2×1
    = 5 + 4 + 6 + 4 + 18 + 24 + 21 + 18 + 0 + 0 + 0 + 2 = 102
  ( 11 − 102 ) mod 11 mod 10 = ( −91 ) mod 11 mod 10 = 8 mod 10 = 8
second check digit, over the twelve values plus the first check digit:
  6×1 + 5×1 + 4×2 + 3×2 + 2×2 + 9×3 + 8×3 + 7×3 + 6×0 + 5×0 + 4×0 + 3×1 + 2×8
    = 6 + 5 + 8 + 6 + 4 + 27 + 24 + 21 + 0 + 0 + 0 + 3 + 16 = 120
  ( 11 − 120 ) mod 11 mod 10 = ( −109 ) mod 11 mod 10 = 1
the complete number is 11222333000181
```

**Taiwan — `04595257`.** The seventh digit is five, not seven, so the simple branch applies.

```formula
multipliers 1,2,1,2,1,2,4,1 against 0,4,5,9,5,2,5,7
products = 0, 8, 5, 18, 5, 4, 20, 7
digit sum = 0 + 8 + 5 + (1+8) + 5 + 4 + (2+0) + 7 = 40
40 mod 5 = 0  →  valid
```

Under the older rule the test was a division by ten, and forty would have failed.

**Turkey, natural person — a constructed example.**

```formula
body = 123456789
check1 = ( 10 − ( 3×1 + 1×2 + 3×3 + 1×4 + 3×5 + 1×6 + 3×7 + 1×8 + 3×9 ) ) mod 10
       = ( 10 − ( 3 + 2 + 9 + 4 + 15 + 6 + 21 + 8 + 27 ) ) mod 10
       = ( 10 − 95 ) mod 10 = 5
check2 = ( 5 + ( 1+2+3+4+5+6+7+8+9 ) ) mod 10 = ( 5 + 45 ) mod 10 = 0
the complete number is 12345678950
```

**Mozambique — a constructed example.**

```formula
body = 12345678
8×1 + 9×2 + 4×3 + 5×4 + 6×5 + 7×6 + 8×7 + 9×8
  = 8 + 18 + 12 + 20 + 30 + 42 + 56 + 72 = 258
258 mod 11 = 5
character at position 5 of "01234567891" is "5"
the complete number is 123456785
```

**Montenegro — a constructed example.**

```formula
body = 1234567
8×1 + 7×2 + 6×3 + 5×4 + 4×5 + 3×6 + 2×7 = 8 + 14 + 18 + 20 + 20 + 18 + 14 = 112
check = ( ( − 112 ) mod 11 ) mod 10 = 9 mod 10 = 9
the complete number is 12345679
```

**North Macedonia — a constructed example.**

```formula
body = 123456789012
7×1 + 6×2 + 5×3 + 4×4 + 3×5 + 2×6 + 7×7 + 6×8 + 5×9 + 4×0 + 3×1 + 2×2
  = 7 + 12 + 15 + 16 + 15 + 12 + 49 + 48 + 45 + 0 + 3 + 4 = 226
check = ( ( − 226 ) mod 11 ) mod 10 = 5
the complete number is 1234567890125
```

**Paraguay — a constructed example.**

```formula
body = 80000000 , read right to left with the weights 2,3,4,5,6,7,8,9
2×0 + 3×0 + 4×0 + 5×0 + 6×0 + 7×0 + 8×0 + 9×8 = 72
check = ( ( − 72 ) mod 11 ) mod 10 = 5
the complete number is 800000005
```

**Guatemala — a constructed example.**

```formula
body = 1120122 , read right to left with the weights 2,3,4,5,6,7,8
c = ( − ( 2×2 + 3×2 + 4×1 + 5×0 + 6×2 + 7×1 + 8×1 ) ) mod 11
  = ( − ( 4 + 6 + 4 + 0 + 12 + 7 + 8 ) ) mod 11 = ( − 41 ) mod 11 = 3
check character = "3"
the complete number is 11201223
```

**Israel — a constructed example.** Nine digits beginning with five and satisfying the doubling
checksum: taking `500000009`, the checksum is four, so the number is **not** valid; the smallest
valid nine-digit number beginning with five is obtained by adjusting the last digit so that the
checksum reaches zero.

### 15.7 The example quoted for each country

The error message quotes one example per country. Where the example is a sentence rather than a
number, the sentence is reproduced.

| Country | Example quoted |
|---|---|
| Albania | `ALJ91402501L` |
| Argentina | `20055361682` |
| Austria | `ATU12345675` |
| Australia | `83 914 571 673` |
| Belgium | `BE0477472701` |
| Bulgaria | `BG1234567892` |
| Brazil | either eleven digits for a natural person or fourteen characters for a legal person |
| Costa Rica | `3101012009` |
| Switzerland | `CHE-123.456.788 TVA` or `CHE-123.456.788 MWST` or `CHE-123.456.788 IVA` |
| Chile | `76086428-5` |
| Colombia | `213123432-1` |
| Cyprus | `CY10259033P` |
| Czechia | `CZ12345679` |
| Germany | `DE123456788` or `12/345/67890` |
| Denmark | `DK12345674` |
| Dominican Republic | `1-01-85004-3` or `101850043` |
| Ecuador | `1792060346001` or `1792060346` |
| Estonia | `EE123456780` |
| Spain | `ESA12345674` |
| Finland | `FI12345671` |
| France | `FR23334175221` |
| United Kingdom | `GB123456782` or `XI123456782` |
| Greece | `EL123456783` |
| Hungary | `HU12345676` or `12345678-1-11` or `8071592153` |
| Croatia | `HR01234567896` |
| Indonesia | `1234567890123456` |
| Ireland | `IE1234567FA` |
| Israel | nine digits respecting the doubling checksum |
| India | `12AAAAA1234AAZA` |
| Iceland | `IS062199` |
| Italy | `IT12345670017` |
| Japan | `T7000012050002` |
| Korea | `123-45-67890` or `1234567890` |
| Lithuania | `LT123456715` |
| Luxembourg | `LU12345613` |
| Latvia | `LV41234567891` |
| Morocco | `12345678` |
| Monaco | `FR53000004605` |
| Malta | `MT12345634` |
| Mexico | `GODE561231GR8` |
| Netherlands | `NL123456782B90` |
| Norway | `NO123456785` |
| New Zealand | `49-098-576` or `49098576` |
| Peru | `10XXXXXXXXY` or `20XXXXXXXXY` or `15XXXXXXXXY` or `16XXXXXXXXY` or `17XXXXXXXXY` |
| Philippines | `123-456-789-123` |
| Poland | `PL1234567883` |
| Portugal | `PT123456789` |
| Romania | `RO1234567897` or `8001011234567` or `9000123456789` |
| Serbia | `RS101134702` |
| Russia | `123456789047` |
| Sweden | `SE123456789701` |
| Slovenia | `SI12345679` |
| Slovakia | `SK2022749619` |
| San Marino | `SM24165` |
| Thailand | `1234545678781` |
| Turkey | eleven digits for a natural person or ten digits for a company |
| Ukraine | `12345678` or `UA12345678`, `1234567890`, or `123456789012` |
| Uruguay | twelve digits, all numbers, valid check digit, for example `219999830019` |
| Uzbekistan | `123456789` for a company or `12345678901234` for an individual |
| Venezuela | `V-12345678-1`, `V123456781` or `V-12.345.678-1` |
| Northern Ireland | `XI123456782` |
| Saudi Arabia | fifteen digits, the first and the last being a three |

### 15.8 Implementation checklist for the number checks

1. Implement the six shared primitives of section 15.2 exactly, including the "or ten" and "or
   thirty-six" folds of the recursive schemes and the base-thirty-six expansion of the modulus
   ninety-seven over ten.
2. Implement the cleaning rules **per country**: they differ in which characters are removed and
   whether the country prefix is dropped.
3. Implement the pipeline of section 15.1 before any country routine; in particular the prefix
   translation for Greece and Northern Ireland, the union retry and the doubled-prefix test.
4. Where a country accepts several kinds of number, try them in the stated order and accept when
   any of them succeeds.
5. Where the specification says "no check digit", do **not** invent one: the number is accepted on
   its shape alone.
6. Where a country has no routine at all, accept the number unchanged.
7. Normalise **before** checking, and store the normalised form.
8. Return both the normalised number and the country code the number was validated for; the caller
   uses the second value to decide whether a fiscal position requiring a registration matches.

---

## 16. Reconstructing the base-to-tax mapping from posted journal items

*Server-only.* Reports, audits and structured document formats need to know, for an already posted
entry, **which base journal item contributed how much to which tax journal item**. The link is not
stored: the entry only records, per base item, the taxes it carries, and per tax item, the
distribution line that produced it. This section specifies the algorithm that rebuilds the mapping.

The result is one row per triple (tax item, base item, source item), carrying a base amount and a
tax amount in both currencies. The **source item** is either the base item itself, or — when the
contribution comes from a tax that swelled the base — the tax item that produced that extra base.

Throughout, the illustrating entry is:

| Item | Originator tax | Base taxes | Debit | Credit |
|---|---|---|---|---|
| base 1 | — | *ten affecting base*, *twenty* | 1 000 | |
| base 2 | — | *ten affecting base*, *five* | 2 000 | |
| base 3 | — | *ten affecting base*, *five* | 3 000 | |
| tax 1 | *ten affecting base* | *twenty* | | 100 |
| tax 2 | *twenty* | — | | 220 |
| tax 3 | *ten affecting base* | *five* | | 500 |
| tax 4 | *five* | — | | 275 |

### 16.1 Step one — the raw mapping

Pair every tax item with every base item of the same entry that satisfies **all** of the following.
Let *T* be the tax item, *B* a candidate base item, *X* the tax that produced *T* and *R* its
distribution line.

1. *B* is not itself a tax item (it has no distribution line).
2. *B* belongs to the same entry as *T*.
3. *B* carries, among its base taxes, the tax recorded as *T*'s originator group when there is one,
   and otherwise *T*'s originator tax.
4. Either the entry is not a miscellaneous entry; **or** *X* is exigible on payment and has a
   transition account; **or** the sign of *T*'s balance equals the sign of
   `B.balance × X.amount × R.factor_percent`. (On a miscellaneous entry the sign is the only
   evidence of which side of the transaction a line belongs to.)
5. *B*'s partner equals *T*'s partner, treating the absence of a partner as equal to the absence of
   a partner.
6. *B*'s currency equals *T*'s currency.
7. Either the distribution line's account, falling back to *B*'s own account, equals *T*'s account;
   **or** *X* is exigible on payment and has a transition account.
8. Either (*X* is not analytic **and** the distribution line is used in the tax settlement); **or**
   both *B* and *T* have no analytic distribution; **or** the two analytic distributions are equal.
9. When *X* affects the base of subsequent taxes, a further test described in 16.2 must also hold.

Each surviving pair contributes a row whose base amount is *B*'s own balance and whose base amount
in document currency is *B*'s own amount in document currency.

In the illustration this yields:

| base item | tax item | base amount |
|---|---|---|
| base 1 | tax 1 | 1 000 |
| base 1 | tax 2 | 1 000 |
| base 2 | tax 3 | 2 000 |
| base 2 | tax 4 | 2 000 |
| base 3 | tax 3 | 3 000 |
| base 3 | tax 4 | 3 000 |

### 16.2 The tail test for a tax that affects the base

Condition 9 exists because a tax item produced by a base-affecting tax must be paired only with
the base items that carry **exactly the same downstream taxes** as that tax item does, not merely
the same affecting tax.

Build, for a journal item, its **affecting-tax list**: expand every Group of Taxes into its
children, drop every tax that does not accept being affected by previous taxes, and sort what
remains by (sequence, identifier).

Then the test is:

```formula
let tail = the affecting-tax list of B , restricted to its last
           ( 1 + length of the affecting-tax list of T ) entries
the pair survives when
    tail = [ T's originator tax ] followed by T's own affecting-tax list
```

In the illustration, *tax 1* is produced by the affecting tax and itself carries the tax *twenty*.
Its affecting-tax list is therefore `[twenty]`, and the required tail is
`[ten affecting base, twenty]`. Base item one's list is exactly that, so the pair survives; base
items two and three have `[ten affecting base, five]`, so they do not.

### 16.3 Step two — the extra base contributed by a base-affecting tax

A tax item produced by a base-affecting tax is itself part of the base of the taxes that come
after it. Those contributions must appear as their own rows, with the tax item as the **source**.

For each tax item *S* whose originator tax affects the base of subsequent taxes, and for each tax
item *T* that shares a base item with *S* and whose originator tax is one of *S*'s own base taxes:

1. Enumerate the base items *B* that step one paired with *S*.
2. Build a running total over those base items, ordered by (the originator tax of *T*, the base
   item's identifier):
   ```formula
   contribution_of_B =  | B.quantity |, carrying the sign of B's balance      when the tax is fixed
                        B.balance                                            otherwise
   cumulated( B ) = the running sum of contribution over the ordering
   total          = the sum over every B
   ```
3. Allocate *S*'s whole amount over those base items by the running-total difference, which
   guarantees that the allocations add back exactly:
   ```formula
   allocated_up_to( B ) = round_to_company_currency(
                              sign( cumulated(B) ) × S.balance × | cumulated(B) | ÷ total )
   extra_base_for( B )  = allocated_up_to( B ) − allocated_up_to( the previous B in the ordering )
   ```
   with the previous value taken as zero for the first base item, and the whole expression taken
   as zero when the total is zero. The same is computed independently in the document currency.
4. Emit one row per base item, with that extra base amount, with *T* as the tax item, *B* as the
   base item and *S* as the source item.

In the illustration:

| base item | tax item | source item | base amount |
|---|---|---|---|
| base 1 | tax 2 | tax 1 | 100 |
| base 2 | tax 4 | tax 3 | 200 |
| base 3 | tax 4 | tax 3 | 300 |

The five hundred of *tax 3* is split two fifths and three fifths, because base items two and three
contribute two thousand and three thousand.

### 16.4 Step three — the fallback mapping

When step one paired a tax item with **no** base item at all — which happens when the
configuration changed after the entry was posted, or when the entry was imported — an approximate
mapping is added. It pairs the orphan tax item with every base item of the same entry and the same
currency that carries, among its base taxes, the tax item's originator group or originator tax. No
other condition is applied. The rows carry the base item's whole balance.

The fallback can be switched off by the caller, in which case an orphan tax item simply produces no
row.

### 16.5 Step four — allocating the tax amounts

Every row so far carries a base amount. The tax amount is allocated by the same running-total
technique, this time over all the rows of one tax item.

1. Order the rows of a tax item by (its originator tax, the base item's identifier, the source
   item's identifier).
2. For each row:
   ```formula
   contribution =  | B.quantity |, carrying the sign of B's balance     when the tax is fixed
                   the row's own base amount                            otherwise
   cumulated    = the running sum of contribution over the ordering
   total        = the sum over every row of the tax item
   allocated_up_to = round_to_company_currency(
                         sign( cumulated ) × T.balance × | cumulated | ÷ total )
   tax_amount   = allocated_up_to − the previous row's allocated_up_to
   ```
   with the previous value taken as zero for the first row, the whole expression taken as zero when
   the total is zero, and the same computed independently in the document currency.

In the illustration:

| base item | tax item | source item | base amount | tax amount |
|---|---|---|---|---|
| base 1 | tax 1 | base 1 | 1 000 | 100 |
| base 1 | tax 2 | base 1 | 1 000 | 1 000 ÷ 1 100 × 220 = 200 |
| base 1 | tax 2 | tax 1 | 100 | 100 ÷ 1 100 × 220 = 20 |
| base 2 | tax 3 | base 2 | 2 000 | 2 000 ÷ 5 000 × 500 = 200 |
| base 2 | tax 4 | base 2 | 2 000 | 2 000 ÷ 5 500 × 275 = 100 |
| base 2 | tax 4 | tax 3 | 200 | 200 ÷ 5 500 × 275 = 10 |
| base 3 | tax 3 | base 3 | 3 000 | 3 000 ÷ 5 000 × 500 = 300 |
| base 3 | tax 4 | base 3 | 3 000 | 3 000 ÷ 5 500 × 275 = 150 |
| base 3 | tax 4 | tax 3 | 300 | 300 ÷ 5 500 × 275 = 15 |

Each tax item's allocations add back exactly: one hundred; two hundred plus twenty equals two
hundred twenty; two hundred plus three hundred equals five hundred; one hundred plus ten plus one
hundred fifty plus fifteen equals two hundred seventy-five.

### 16.6 The other columns of a row

| Column | Value |
|---|---|
| identifier | the three item identifiers joined by hyphens, in the order tax item, base item, source item |
| base item, tax item, source item | as above |
| display kind | the tax item's display kind |
| tax | the tax item's originator tax |
| originator group of taxes | the tax item's |
| distribution line | the tax item's |
| base account | the base item's account |
| exigible | true when the tax is **not** exigible on payment, **or** the tax item's entry is itself a cash basis entry, **or** the entry is marked as always exigible |
| company, company currency and its decimal places, currency and its decimal places | from the tax item |

The *exigible* column is what a tax return filters on when it is configured to show only exigible
lines.

### 16.7 Why the running-total technique

Allocating a total over several parts by multiplying each part's share and rounding each product
independently loses or gains units of the last decimal place. The running-total technique rounds
the **cumulative** allocation and takes differences, so the last row absorbs whatever the earlier
roundings left over and the allocations always add back to the total exactly. It is the same idea
as the smooth distribution of section 7.1, applied where an ordering rather than a weight list is
the natural input.
