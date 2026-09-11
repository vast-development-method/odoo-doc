# Units of Measure and Packaging — Calculations

This file is the arithmetic core of the domain. It specifies, with no gaps:

- the rounding function the whole platform uses, in full, including its error-compensation term;
- the comparison and zero-test built on that function;
- how a unit's absolute quantity is derived from its declared contained quantity;
- the absolute quantity of every shipped unit, to the last representable digit;
- the quantity conversion algorithm, step by step, with every short-circuit;
- the **complete conversion matrix** for every tree, in both directions, under three rounding
  methods;
- the price conversion algorithm and why it differs;
- the whole-packaging rounding algorithm;
- the shared-ancestor test that decides whether a conversion is possible at all;
- the rounding method used at **every document boundary** a quantity crosses;
- every derived computation in the domain (weights, volumes, discounted vendor prices, packaging
  quantities, component scaling);
- the behaviour at the rounding edges, including conversions that round to zero and conversions
  that round up from almost nothing.

Formulas are written as plain mathematics. Quantities are named in words. Worked numeric examples
follow every formula.

---

## 1. Notation and shared definitions

### 1.1 Named quantities

| Name used in formulas | Meaning |
|---|---|
| contained quantity | The `relative_factor` of a unit: how many of its reference unit one of it contains. |
| absolute quantity | The `factor` of a unit: how many of the root unit of its tree one of it contains. |
| rounding precision | The `rounding` of a unit: ten raised to the power of minus the number of digits of the `Product Unit` decimal precision record. Identical for every unit. |
| precision digits | The number of digits of the `Product Unit` decimal precision record. Shipped value: two. |
| source unit | The unit a quantity is currently expressed in. |
| destination unit | The unit a quantity is being converted to. |
| rounding method | One of: away from zero, towards zero, half away from zero, half towards zero, half to even. |

### 1.2 The five rounding methods

| Method | Stored selector | Behaviour |
|---|---|---|
| Half away from zero | `HALF-UP` | Round to the nearest step; a value exactly halfway goes away from zero. **This is the default of the rounding function itself.** |
| Half towards zero | `HALF-DOWN` | Round to the nearest step; a value exactly halfway goes towards zero. |
| Half to even | `HALF-EVEN` | Round to the nearest step; a value exactly halfway goes to the nearer even multiple of the step. |
| Away from zero | `UP` | Always round away from zero: any non-zero fractional part pushes the result to the next step further from zero. **This is the default of the quantity conversion operation.** |
| Towards zero | `DOWN` | Always round towards zero: the fractional part is discarded. |

The two defaults differ, and this is the single most common source of mistaken rebuilds. The
*rounding function* defaults to half away from zero. The *quantity conversion* defaults to away
from zero. A conversion that produces nine hundredths of a dozen from one unit is therefore not a
bug: away-from-zero rounding of eight and one third hundredths gives nine hundredths.

### 1.3 The precision grid

A precision may be given either as a number of digits or as a step.

```formula
step = 10 ^ ( − precision_digits )
```

With two digits the step is one hundredth. The rounding function accepts **exactly one** of the
two forms; supplying both, or neither, is a programming error and must fail loudly. Supplying a
step requires the step to be strictly greater than zero. Supplying a number of digits requires it
to be a whole number greater than or equal to zero.

The step need not be a power of ten. The function is written to round onto arbitrary steps, so a
step of one half or one quarter is legal and rounds onto halves or quarters. The domain uses
exactly two steps in practice: the `Product Unit` step (one hundredth as shipped) for quantity
conversion, and a step of **one** for whole-packaging rounding.

---

## 2. The rounding function

Every quantity in this domain is rounded by one function. A rebuild must reproduce it exactly,
including the error-compensation term, because several documented results depend on it.

### 2.1 Statement

Given a value, a precision (as digits or as a step) and a rounding method, produce a rounded
value.

```formula
rounded_value = denormalize( apply_method( normalize( value ) ) )
```

### 2.2 Algorithm

1. **Resolve the step.** If a step was supplied, assert it is strictly positive and use it. If a
   number of digits was supplied, assert it is a non-negative whole number and set the step to ten
   raised to minus that number. If both or neither were supplied, fail.
2. **Short-circuit.** If the step is zero, or the value is zero, return zero. (The step cannot be
   zero when supplied directly, because of the assertion in step 1; the test exists for the
   degenerate case where the digit count produced a step that underflowed to zero.)
3. **Choose the scaling direction.** Define two operations, *normalise* and *denormalise*:
   - normalise divides by the step, denormalise multiplies by the step;
   - **but if the step is smaller than one**, replace the step by its *accurately inverted* value
     (see 2.3) and **swap** the two operations, so that normalise now multiplies by the inverted
     step and denormalise divides by it.
   The purpose is to turn a division by a small fraction — which loses accuracy — into a
   multiplication by a large integer, which does not. With a step of one hundredth, normalising a
   value multiplies it by one hundred.
4. **Normalise.** Apply the normalise operation to the value. The result is the value expressed in
   whole steps, plus a fraction.
5. **Compute the compensation term.** Let the magnitude be the base-two logarithm of the absolute
   normalised value. The compensation term is two raised to the power of that magnitude minus
   fifty.

   ```formula
   epsilon = 2 ^ ( log2( | normalized_value | ) − 50 )
   ```

   The minimal term that repairs a single unit in the last place would use fifty-two rather than
   fifty; fifty is used deliberately so that error accumulated over several prior floating-point
   operations is also absorbed. A rebuild that uses fifty-two, or omits the term, will produce
   different results on values such as two and six hundred seventy-five thousandths, whose binary
   representation is very slightly below the tie.
6. **Apply the method** to the normalised value:
   - **half away from zero:** add the compensation term with the sign of the normalised value,
     then round half away from zero to a whole number;
   - **half to even:** take the whole part below (the floor); take the absolute difference between
     the normalised value and that floor; if that difference is within the compensation term of
     one half, the value is a tie, and the result is the floor plus one if the floor is odd, else
     the floor; otherwise round half away from zero to a whole number;
   - **half towards zero:** subtract the compensation term with the sign of the normalised value,
     then round half away from zero to a whole number;
   - **away from zero:** add, with the sign of the normalised value, the quantity one minus the
     compensation term, then truncate towards zero;
   - **towards zero:** add the compensation term with the sign of the normalised value, then
     truncate towards zero.
   Any other method is a programming error and must fail with a message naming the unknown method.
7. **Denormalise.** Apply the denormalise operation to the whole-number result and return it.

The "round half away from zero to a whole number" used inside step 6 is itself specified: it is
**not** the round-half-to-even rounding that most language runtimes provide by default. It is
defined as: take the runtime's nearest-integer result; if adding one to the value and taking the
runtime's nearest-integer result does not increase the answer by exactly one, the value was a tie
that the runtime resolved to even, so instead return the value plus one half carrying the sign of
the value; otherwise return the runtime's result carrying the sign of the value. The sign carrying
exists so that rounding negative zero yields negative zero and so that the result is a real number
rather than a whole number.

### 2.3 Accurate inversion of a step

Inverting a step must not itself introduce error. A lookup table holds the exact inverses of the
thirty steps most commonly used:

| Step | Inverse | Step | Inverse | Step | Inverse |
|---|---|---|---|---|---|
| one tenth | ten | two tenths | five | five tenths | two |
| one hundredth | one hundred | two hundredths | fifty | five hundredths | twenty |
| one thousandth | one thousand | two thousandths | five hundred | five thousandths | two hundred |
| one ten-thousandth | ten thousand | two ten-thousandths | five thousand | five ten-thousandths | two thousand |
| one hundred-thousandth | one hundred thousand | two hundred-thousandths | fifty thousand | five hundred-thousandths | twenty thousand |
| one millionth | one million | two millionths | five hundred thousand | five millionths | two hundred thousand |
| one ten-millionth | ten million | two ten-millionths | five million | five ten-millionths | two million |
| one hundred-millionth | one hundred million | two hundred-millionths | fifty million | five hundred-millionths | twenty million |
| one billionth | one billion | two billionths | five hundred million | five billionths | two hundred million |
| one ten-billionth | ten billion | two ten-billionths | five billion | five ten-billionths | two billion |

For a step not in the table, the inverse is computed as follows: render the step in scientific
notation with fifteen digits after the point, splitting it into a coefficient and an exponent;
build the number whose coefficient is the same and whose exponent is negated; divide that by the
square of the coefficient.

```formula
inverted_step = ( coefficient × 10 ^ ( − exponent ) ) ÷ coefficient²
```

Worked example. For a step of one eighth, the scientific rendering is a coefficient of one and two
hundred fifty thousandths with an exponent of minus one. The number with the negated exponent is
twelve and one half. Divided by the square of the coefficient, one and five thousand six hundred
twenty-five ten-thousandths, the result is exactly eight.

### 2.4 Worked examples of the rounding function

| Value | Step | Method | Result | Why |
|---|---|---|---|---|
| 2.675 | one thousandth of a hundred, i.e. two digits | half away from zero | 2.68 | The binary value is slightly below the tie; the compensation term lifts it over. Without the term the answer would be 2.67. |
| 0.0833333… | two digits | away from zero | 0.09 | Any non-zero fraction of a step pushes away from zero. |
| 0.0833333… | two digits | half away from zero | 0.08 | Eight and one third steps is nearer eight than nine. |
| 0.0833333… | two digits | towards zero | 0.08 | The fractional part is discarded. |
| 1.234 | two digits | away from zero | 1.24 | |
| 1.234 | two digits | half away from zero | 1.23 | |
| −0.5 | zero digits | half away from zero | −1 | Ties go away from zero, so the negative tie goes to minus one. |
| −0.5 | zero digits | half towards zero | 0 | Ties go towards zero. |
| 0.5 | zero digits | half to even | 0 | Zero is the nearer even whole number. |
| 1.5 | zero digits | half to even | 2 | Two is even. |
| 2.5 | zero digits | half to even | 2 | Two is even; this is where half-to-even differs from half away from zero. |
| 1.0000000001 | two digits | away from zero | 1.01 | A fraction of a step smaller than the compensation term would be absorbed; this one is not. |
| 1.000000000000001 | two digits | away from zero | 1.0 | The excess is below the compensation term at this magnitude and is absorbed. |

The last two rows matter. The compensation term is scaled to the magnitude of the *normalised*
value, so at a normalised magnitude of about one hundred the term is roughly one hundred divided
by two raised to the fiftieth power, which is about nine hundredths of a quadrillionth. Any excess
smaller than that is treated as floating-point noise and does not trigger an away-from-zero step.

---

## 3. Comparison and zero test

Three operations are offered on every unit, and all three use the `Product Unit` precision, never
a per-unit precision.

### 3.1 Rounding a value in a unit

```formula
rounded = round_to_precision( value , precision_digits , rounding_method )
```

with the rounding method defaulting to **half away from zero**. Note that this per-unit rounding
operation has a *different* default from the conversion operation. A rebuild must keep them
separate.

### 3.2 Comparing two values in a unit

```formula
comparison = compare( value_one , value_two , precision_digits )
```

Result is minus one, zero or plus one according to whether the first value is lower than, equal
to, or greater than the second, **after both have been rounded onto the precision grid**.

Algorithm:

1. If the two values are identical as stored, return zero immediately. (This short-circuit exists
   so that the parameter checks still run but no rounding is performed.)
2. Round each value onto the grid, half away from zero.
3. Take the difference of the rounded values.
4. If the difference is zero at the same precision, return zero.
5. Otherwise return minus one if the difference is negative and plus one if it is positive.

The important consequence: **two values may differ by less than the step and still compare as
different**, because each is rounded *before* the subtraction. Six thousandths and two thousandths
differ by four thousandths, which is below a hundredth, yet they round to one hundredth and zero
respectively and therefore compare as different.

### 3.3 Testing a value for zero in a unit

```formula
is_zero = ( value = 0 ) or ( | round_to_precision( value , step ) | < step )
```

Algorithm: return true when the value is exactly zero, or when its absolute value, after rounding
onto the grid half away from zero, is strictly smaller than the step.

**The zero test and the comparison do not agree in general.** Testing whether the difference of
two values is zero rounds *after* subtracting; comparing them rounds *before*. The pair six
thousandths and two thousandths is zero by the difference test and different by the comparison
test. A rebuild that implements one in terms of the other will diverge.

### 3.4 Euclidean division at a precision

A division that yields a whole quotient and a remainder, both free of representation error, is
also available and is used where a quantity must be split into whole packs and a leftover.

Algorithm:

1. Resolve the step as in 2.2 step 1.
2. Round each operand onto the grid and scale it to an exact whole number by dividing by the step
   and taking the runtime's nearest whole number.
3. Take the whole quotient and remainder of the two scaled whole numbers.
4. Return the quotient unchanged, and the remainder multiplied by the step and rounded onto the
   grid.

The postcondition is that the dividend, rounded onto the grid, equals the quotient multiplied by
the divisor plus the remainder, with the quotient a whole number.

---

## 4. Deriving the absolute quantity

### 4.1 The formula

```formula
absolute_quantity(unit) =
    contained_quantity(unit) × absolute_quantity( reference_unit(unit) )      if the unit has a reference unit
    contained_quantity(unit)                                                  if it does not
```

The computation is **recursive and stored**. Changing the contained quantity of any unit
recomputes the absolute quantity of that unit and of every unit below it in the tree, in the same
transaction.

Because the reference unit's *stored* absolute quantity is used, and not a freshly evaluated
product of the whole chain, the arithmetic is a left-to-right chain of multiplications from the
root downwards. This matters for values that are not exactly representable: the foot is computed
as twelve multiplied by the inch's stored absolute quantity of twenty-five and four tenths, which
gives three hundred four and seven hundred ninety-nine thousandths and change rather than exactly
three hundred four and eight tenths. A rebuild that computes the foot as twelve multiplied by two
and fifty-four hundredths multiplied by ten will get a different last digit.

### 4.2 Worked derivations

**A root unit.** The counting unit has no reference unit and a contained quantity of one. Its
absolute quantity is one.

```formula
absolute_quantity(Units) = 1
```

**One level down.** The dozen has the counting unit as reference and contains twelve.

```formula
absolute_quantity(Dozens) = 12 × 1 = 12
```

**Two levels down.** A box containing twelve dozens:

```formula
absolute_quantity(Box) = 12 × absolute_quantity(Dozens) = 12 × 12 = 144
```

**Three levels down.** A pallet containing forty boxes:

```formula
absolute_quantity(Pallet) = 40 × absolute_quantity(Box) = 40 × 144 = 5760
```

**A chain that is not exactly representable.** The metre is one hundred centimetres, the
centimetre is ten millimetres, the millimetre is the root:

```formula
absolute_quantity(mm)  = 1
absolute_quantity(cm)  = 10 × 1   = 10
absolute_quantity(m)   = 100 × 10 = 1000
absolute_quantity(km)  = 1000 × 1000 = 1000000
absolute_quantity(in)  = 2.54 × 10 = 25.4
absolute_quantity(ft)  = 12 × 25.4 = 304.79999999999995
absolute_quantity(yd)  = 3 × 304.79999999999995 = 914.3999999999999
absolute_quantity(mi)  = 1760 × 914.3999999999999 = 1609343.9999999998
```

The last three values are the exact binary-floating-point results of the stated multiplication
order. They are reproduced deliberately: a rebuild using exact decimal arithmetic will produce
three hundred four and eight tenths, nine hundred fourteen and four tenths, and one million six
hundred nine thousand three hundred forty-four, and will therefore round differently on some
conversions. Where a rebuild chooses exact decimal arithmetic — which is defensible — it must
expect those specific discrepancies in the last displayed digit and must document them; it must
not expect the conversion matrix in section 7 to match digit for digit for the length tree.

**A contained quantity smaller than one.** The minute contains sixteen thousand six hundred
sixty-seven millionths of an hour:

```formula
absolute_quantity(Minutes) = 0.0166667 × 1 = 0.0166667
```

Note that this is not exactly one sixtieth. The shipped value is one sixtieth rounded up at the
seventh decimal digit. The consequence is visible in the conversion matrix: one hour is fifty-nine
and nine hundred ninety-nine thousandths minutes before rounding, not sixty.

### 4.3 Inverting the derivation

Occasionally a rebuild must go the other way: given a desired absolute quantity, what contained
quantity must be typed?

```formula
contained_quantity(unit) = absolute_quantity(unit) ÷ absolute_quantity( reference_unit(unit) )
```

Worked example. To create a unit "Gross" of one hundred forty-four items whose reference unit is
the dozen, the contained quantity to type is one hundred forty-four divided by twelve, that is
twelve.

---

## 5. The absolute quantity of every shipped unit

The table below is the complete reference data as delivered, with the derived absolute quantity
and the derived sequence. The rounding precision column is the value every unit reports at the
shipped precision of two digits.

| External identifier | Name (`name`) | Tree | Reference unit (`relative_uom_id`) | Contains (`relative_factor`) | Absolute quantity (`factor`) | Sequence (`sequence`) | Active on delivery | Rounding precision (`rounding`) |
|---|---|---|---|---|---|---|---|---|
| `uom.product_uom_unit` | `Units` | Counting | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_pack_6` | `Pack of 6` | Counting | `Units` | 6 | 6 | 600 | yes | 0.01 |
| `uom.product_uom_dozen` | `Dozens` | Counting | `Units` | 12 | 12 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_hour` | `Hours` | Working time | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_day` | `Days` | Working time | `Hours` | 8 | 8 | 800 | yes | 0.01 |
| `uom.product_uom_minute` | `Minutes` | Working time | `Hours` | 0.0166667 | 0.0166667 | 1 | yes | 0.01 |
| `uom.product_uom_millimeter` | `mm` | Length | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_cm` | `cm` | Length | `mm` | 10 | 10 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_meter` | `m` | Length | `cm` | 100 | 1000 | 1000 | yes | 0.01 |
| `uom.product_uom_km` | `km` | Length | `m` | 1000 | 1000000 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_inch` | `in` | Length | `cm` | 2.54 | 25.4 | 254 | no (archived) | 0.01 |
| `uom.product_uom_foot` | `ft` | Length | `in` | 12 | 304.79999999999995 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_yard` | `yd` | Length | `ft` | 3 | 914.3999999999999 | 300 | no (archived) | 0.01 |
| `uom.product_uom_mile` | `mi` | Length | `yd` | 1760 | 1609343.9999999998 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_square_meter` | `m2` | Surface | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_square_foot` | `ft2` | Surface | `m2` | 0.092903 | 0.092903 | 9 | no (archived) | 0.01 |
| `uom.product_uom_milliliter` | `ml` | Volume | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_litre` | `L` | Volume | `ml` | 1000 | 1000 | 1000 | yes | 0.01 |
| `uom.product_uom_cubic_meter` | `m3` | Volume | `L` | 1000 | 1000000 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_floz` | `floz` | Volume | `L` | 0.0295735 | 29.5735 | 2 | no (archived) | 0.01 |
| `uom.product_uom_qt` | `qt` | Volume | `floz` | 32 | 946.352 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_gal` | `gal` | Volume | `qt` | 4 | 3785.408 | 400 | no (archived) | 0.01 |
| `uom.product_uom_cubic_inch` | `in3` | Volume | `L` | 0.0163871 | 16.3871 | 1 | no (archived) | 0.01 |
| `uom.product_uom_cubic_foot` | `ft3` | Volume | `in3` | 1728 | 28316.9088 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_gram` | `g` | Mass | — (root) | 1 | 1 | 100 | yes | 0.01 |
| `uom.product_uom_kgm` | `kg` | Mass | `g` | 1000 | 1000 | 1000 | yes | 0.01 |
| `uom.product_uom_ton` | `Ton` | Mass | `kg` | 1000 | 1000000 | 1000 | yes | 0.01 |
| `uom.product_uom_oz` | `oz` | Mass | `g` | 28.3495 | 28.3495 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_lb` | `lb` | Mass | `oz` | 16 | 453.592 | 1000 | no (archived) | 0.01 |
| `uom.product_uom_kwh` | `KWH` | Energy | — (root) | 1 | 1 | 100 | yes | 0.01 |
