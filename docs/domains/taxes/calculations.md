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
   *"Only primitive types are allowed in python tax formula context."*
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
