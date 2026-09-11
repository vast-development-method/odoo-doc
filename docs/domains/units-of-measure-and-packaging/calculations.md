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

### 5.1 The trees the shipped units form

```mermaid
flowchart TD
    subgraph Counting
      U["Units — absolute 1"] --> P6["Pack of 6 — absolute 6"]
      U --> DZ["Dozens — absolute 12"]
      DZ -. configured .-> BX["Box of 12 Dozens — absolute 144"]
      BX -. configured .-> PL["Pallet of 40 Boxes — absolute 5760"]
    end
    subgraph WorkingTime["Working time"]
      H["Hours — absolute 1"] --> D["Days — absolute 8"]
      H --> MIN["Minutes — absolute 0.0166667"]
    end
    subgraph Mass
      G["g — absolute 1"] --> KG["kg — absolute 1000"]
      KG --> TON["Ton — absolute 1000000"]
      G --> OZ["oz — absolute 28.3495"]
      OZ --> LB["lb — absolute 453.592"]
    end
    subgraph Volume
      ML["ml — absolute 1"] --> LT["L — absolute 1000"]
      LT --> M3["m³ — absolute 1000000"]
      LT --> FLOZ["fl oz — absolute 29.5735"]
      FLOZ --> QT["qt — absolute 946.352"]
      QT --> GAL["gal — absolute 3785.408"]
      LT --> IN3["in³ — absolute 16.3871"]
      IN3 --> FT3["ft³ — absolute 28316.9088"]
    end
    subgraph Length
      MM["mm — absolute 1"] --> CM["cm — absolute 10"]
      CM --> MT["m — absolute 1000"]
      MT --> KM["km — absolute 1000000"]
      CM --> IN["in — absolute 25.4"]
      IN --> FT["ft — absolute 304.8"]
      FT --> YD["yd — absolute 914.4"]
      YD --> MI["mi — absolute 1609344"]
    end
    subgraph Surface
      M2["m² — absolute 1"] --> FT2["ft² — absolute 0.092903"]
    end
    subgraph Energy
      KWH["KWH — absolute 1"]
    end
```

Seven trees exist as delivered. Two of them hold a single unit (surface has two, energy has one).
No conversion is possible between trees; see section 9.

### 5.2 The configured packaging units used throughout this file

Two units beyond the shipped set are referenced repeatedly in the worked examples and in the
conversion matrix, because the client scenario this specification must serve is stated in boxes
and pallets. They are **not** delivered as reference data; they are the units a user creates to
model cartons and pallets, and they are specified here so the arithmetic is unambiguous.

| Name | Reference unit | Contains | Absolute quantity | Meaning |
|---|---|---|---|---|
| `Box of 12 Dozens` | `Dozens` | 12 | 144 | A carton holding twelve dozens, that is one hundred forty-four items. |
| `Pallet of 40 Boxes` | `Box of 12 Dozens` | 40 | 5760 | A pallet holding forty such cartons, that is five thousand seven hundred sixty items. |

Their sequences, by the derivation in [`entities.md`](entities.md), are both one thousand (twelve
multiplied by one hundred and forty multiplied by one hundred both exceed the cap).

A shorter carton is also used in one worked example and must not be confused with the box above:

| Name | Reference unit | Contains | Absolute quantity | Meaning |
|---|---|---|---|---|
| `Box of 12` | `Units` | 12 | 12 | A carton holding twelve items. Arithmetically identical to a dozen; a separate unit only so that the document prints the word `Box`. |

### 5.3 The complete quantity ladder of the counting tree

| Unit | Absolute quantity | In units | In packs of six | In dozens | In boxes of twelve dozens | In pallets |
|---|---|---|---|---|---|---|
| One unit | 1 | 1 | one sixth | one twelfth | one one-hundred-forty-fourth | one five-thousand-seven-hundred-sixtieth |
| One pack of six | 6 | 6 | 1 | one half | one twenty-fourth | one nine-hundred-sixtieth |
| One dozen | 12 | 12 | 2 | 1 | one twelfth | one four-hundred-eightieth |
| One box of twelve dozens | 144 | 144 | 24 | 12 | 1 | one fortieth |
| One pallet of forty boxes | 5760 | 5760 | 960 | 480 | 40 | 1 |

Read the other way: a pallet is five thousand seven hundred sixty items, four hundred eighty
dozens, nine hundred sixty packs of six, or forty boxes. A box is one hundred forty-four items,
twelve dozens or twenty-four packs of six.

---

## 6. The quantity conversion algorithm

This is the operation every document boundary calls.

### 6.1 Inputs and outputs

| Input | Meaning | Default |
|---|---|---|
| source unit | The unit the quantity is expressed in. Exactly one unit, or none. | — |
| quantity | The number to convert. | — |
| destination unit | The unit to convert to. May be none. | — |
| round | Whether to round the result. | yes |
| rounding method | Which of the five methods to use. | **away from zero** |
| raise on failure | Whether an impossible conversion must fail or return the input unchanged. | yes |

Output: a number expressed in the destination unit.

### 6.2 Algorithm

1. **Empty short-circuit.** If there is no source unit, or the quantity is zero (or otherwise
   falsy), return the quantity unchanged. No rounding, no unit check. A conversion of zero
   therefore always yields exactly zero, whatever the units and whatever the rounding method.
2. **Single-record precondition.** Assert that the source is exactly one unit. More than one, or
   an unloaded set, is a programming error.
3. **Identity short-circuit.** If the source unit and the destination unit are the same record,
   the working amount is the quantity itself, with **no multiplication and no division**. This
   matters: converting a quantity to its own unit never introduces a rounding step through the
   factors, though the rounding in step 5 still applies.
4. **Scale.** Otherwise:

   ```formula
   amount = quantity × absolute_quantity( source_unit )
   ```

   and then, **only if a destination unit was supplied**:

   ```formula
   amount = amount ÷ absolute_quantity( destination_unit )
   ```

   The order is fixed: multiply first, divide second. A rebuild that computes the ratio of the two
   absolute quantities first and multiplies once will produce different last digits on many pairs.
   When no destination unit is supplied, the result is the quantity expressed in the *root* unit
   of the source unit's tree.
5. **Round.** If a destination unit was supplied **and** rounding was requested, round the amount
   onto the destination unit's rounding precision, using the supplied rounding method:

   ```formula
   result = round_to_step( amount , rounding_precision( destination_unit ) , rounding_method )
   ```

   Since every unit shares the same rounding precision, this is always the `Product Unit` step.
   When no destination unit was supplied, **no rounding happens at all**, even if rounding was
   requested.
6. **Return** the amount.

### 6.3 The four short-circuit cases, stated explicitly

| Case | Result |
|---|---|
| No source unit | The quantity, unchanged, unrounded. |
| Quantity is zero | Zero, unchanged, unrounded. |
| Source unit equals destination unit | The quantity, rounded onto the grid with the requested method. No factor arithmetic. |
| Destination unit is absent | The quantity multiplied by the source unit's absolute quantity, **unrounded**. |

The third case is subtle. Converting twenty-two and forty-three hundredths units into units with
away-from-zero rounding returns twenty-two and forty-three hundredths, because the factor
arithmetic is skipped and the value is already on the grid. Converting twenty-two and four
hundred thirty-three thousandths units into units with away-from-zero rounding returns twenty-two
and forty-four hundredths, because the rounding step still runs.

### 6.4 Cross-tree conversions

The conversion algorithm as stated performs no tree check at all: it will happily multiply a
quantity by the source absolute quantity and divide by the destination absolute quantity even if
the two units are in different trees, producing a number with no physical meaning. The check is
the caller's responsibility, and the *raise on failure* input exists for callers that want the
input returned unchanged rather than a meaningless number. Callers that must be safe use the
shared-ancestor test of section 9 before converting.

**Industry-standard default.** A rebuild should treat a conversion between units with no common
ancestor as an error when the caller has asked for failure to be raised, and should return the
input quantity unchanged when the caller has asked for failure to be tolerated. That is the
contract the input parameter names, and callers rely on it — several component-quantity
computations pass "tolerate failure" precisely so that a mis-configured bill of materials yields
the raw number instead of aborting a whole production plan.

### 6.5 Round trips are not identities

Converting a quantity to another unit and back does not in general return the original quantity,
because each direction rounds.

Worked example, at the shipped precision of two digits and the default rounding method:

```formula
7 Units → Dozens  = round_away_from_zero( 7 × 1 ÷ 12 ) = round_away_from_zero( 0.58333… ) = 0.59
0.59 Dozens → Units = round_away_from_zero( 0.59 × 12 ÷ 1 ) = round_away_from_zero( 7.08 ) = 7.08
```

Seven units become seven and eight hundredths units after a round trip through dozens. A rebuild
must not "optimise" a round trip away, and a business process that converts back and forth
repeatedly will drift upwards under the default method. This is the reason the reservation
arithmetic of the warehouse deliberately converts **down** and then back **half away from zero**,
so that it can never reserve more than is available; see section 12.4.

---

## 7. The complete conversion matrix

Each table gives, for one tree, every ordered pair of distinct units. The columns are:

- **Exact ratio** — one source unit expressed in destination units, before any rounding. This is
  the quotient of the two stored absolute quantities, shown to the full precision the stored
  values support.
- **Result, default rounding away from zero** — what the conversion operation returns for a
  quantity of one with its default rounding method, at the shipped precision of two digits.
- **Result, half-up rounding** — what it returns when the caller asks for half away from zero,
  which is what the stock-move and move-line boundaries do.
- **Result, rounding towards zero** — what it returns when the caller asks for towards zero,
  which is what the reservation arithmetic does in its first leg.

To convert a quantity other than one, multiply the quantity by the exact ratio and then round;
do **not** multiply the quantity by a rounded ratio. Section 8 works this through.

#### Counting and packaging tree

| Source unit | Destination unit | Exact ratio (one source unit in destination units) | Result, default rounding away from zero | Result, half-up rounding | Result, rounding towards zero |
|---|---|---|---|---|---|
| Units | Pack of 6 | 0.16666666666666666 | 0.17 | 0.17 | 0.16 |
| Units | Dozens | 0.08333333333333333 | 0.09 | 0.08 | 0.08 |
| Units | Box of 12 Dozens | 0.006944444444444444 | 0.01 | 0.01 | 0 |
| Units | Pallet of 40 Boxes | 0.00017361111111111112 | 0.01 | 0 | 0 |
| Pack of 6 | Units | 6 | 6 | 6 | 6 |
| Pack of 6 | Dozens | 0.5 | 0.5 | 0.5 | 0.5 |
| Pack of 6 | Box of 12 Dozens | 0.041666666666666664 | 0.05 | 0.04 | 0.04 |
| Pack of 6 | Pallet of 40 Boxes | 0.0010416666666666667 | 0.01 | 0 | 0 |
| Dozens | Units | 12 | 12 | 12 | 12 |
| Dozens | Pack of 6 | 2 | 2 | 2 | 2 |
| Dozens | Box of 12 Dozens | 0.08333333333333333 | 0.09 | 0.08 | 0.08 |
| Dozens | Pallet of 40 Boxes | 0.0020833333333333333 | 0.01 | 0 | 0 |
| Box of 12 Dozens | Units | 144 | 144 | 144 | 144 |
| Box of 12 Dozens | Pack of 6 | 24 | 24 | 24 | 24 |
| Box of 12 Dozens | Dozens | 12 | 12 | 12 | 12 |
| Box of 12 Dozens | Pallet of 40 Boxes | 0.025 | 0.03 | 0.03 | 0.02 |
| Pallet of 40 Boxes | Units | 5760 | 5760 | 5760 | 5760 |
| Pallet of 40 Boxes | Pack of 6 | 960 | 960 | 960 | 960 |
| Pallet of 40 Boxes | Dozens | 480 | 480 | 480 | 480 |
| Pallet of 40 Boxes | Box of 12 Dozens | 40 | 40 | 40 | 40 |

#### Mass tree

| Source unit | Destination unit | Exact ratio (one source unit in destination units) | Result, default rounding away from zero | Result, half-up rounding | Result, rounding towards zero |
|---|---|---|---|---|---|
| g | kg | 0.001 | 0.01 | 0 | 0 |
| g | Ton | 1e-06 | 0.01 | 0 | 0 |
| g | oz | 0.03527399072294044 | 0.04 | 0.04 | 0.03 |
| g | lb | 0.0022046244201837776 | 0.01 | 0 | 0 |
| kg | g | 1000 | 1000 | 1000 | 1000 |
| kg | Ton | 0.001 | 0.01 | 0 | 0 |
| kg | oz | 35.27399072294044 | 35.28 | 35.27 | 35.27 |
| kg | lb | 2.2046244201837775 | 2.21 | 2.2 | 2.2 |
| Ton | g | 1000000 | 1000000 | 1000000 | 1000000 |
| Ton | kg | 1000 | 1000 | 1000 | 1000 |
| Ton | oz | 35273.99072294044 | 35274 | 35273.99 | 35273.99 |
| Ton | lb | 2204.6244201837776 | 2204.63 | 2204.62 | 2204.62 |
| oz | g | 28.3495 | 28.35 | 28.35 | 28.34 |
| oz | kg | 0.0283495 | 0.03 | 0.03 | 0.02 |
| oz | Ton | 2.83495e-05 | 0.01 | 0 | 0 |
| oz | lb | 0.0625 | 0.07 | 0.06 | 0.06 |
| lb | g | 453.592 | 453.6 | 453.59 | 453.59 |
| lb | kg | 0.453592 | 0.46 | 0.45 | 0.45 |
| lb | Ton | 0.000453592 | 0.01 | 0 | 0 |
| lb | oz | 16 | 16 | 16 | 16 |

#### Volume tree

| Source unit | Destination unit | Exact ratio (one source unit in destination units) | Result, default rounding away from zero | Result, half-up rounding | Result, rounding towards zero |
|---|---|---|---|---|---|
| ml | L | 0.001 | 0.01 | 0 | 0 |
| ml | m³ | 1e-06 | 0.01 | 0 | 0 |
| ml | fl oz (US) | 0.03381405650328842 | 0.04 | 0.03 | 0.03 |
| ml | qt (US) | 0.0010566892657277631 | 0.01 | 0 | 0 |
| ml | gal (US) | 0.0002641723164319408 | 0.01 | 0 | 0 |
| ml | in³ | 0.06102361003472243 | 0.07 | 0.06 | 0.06 |
| ml | ft³ | 3.531458914046437e-05 | 0.01 | 0 | 0 |
| L | ml | 1000 | 1000 | 1000 | 1000 |
| L | m³ | 0.001 | 0.01 | 0 | 0 |
| L | fl oz (US) | 33.81405650328842 | 33.82 | 33.81 | 33.81 |
| L | qt (US) | 1.0566892657277631 | 1.06 | 1.06 | 1.05 |
| L | gal (US) | 0.2641723164319408 | 0.27 | 0.26 | 0.26 |
| L | in³ | 61.02361003472243 | 61.03 | 61.02 | 61.02 |
| L | ft³ | 0.03531458914046437 | 0.04 | 0.04 | 0.03 |
| m³ | ml | 1000000 | 1000000 | 1000000 | 1000000 |
| m³ | L | 1000 | 1000 | 1000 | 1000 |
| m³ | fl oz (US) | 33814.05650328842 | 33814.06 | 33814.06 | 33814.05 |
| m³ | qt (US) | 1056.689265727763 | 1056.69 | 1056.69 | 1056.68 |
| m³ | gal (US) | 264.17231643194077 | 264.18 | 264.17 | 264.17 |
| m³ | in³ | 61023.610034722435 | 61023.62 | 61023.61 | 61023.61 |
| m³ | ft³ | 35.31458914046437 | 35.32 | 35.31 | 35.31 |
| fl oz (US) | ml | 29.5735 | 29.58 | 29.57 | 29.57 |
| fl oz (US) | L | 0.0295735 | 0.03 | 0.03 | 0.02 |
| fl oz (US) | m³ | 2.95735e-05 | 0.01 | 0 | 0 |
| fl oz (US) | qt (US) | 0.03125 | 0.04 | 0.03 | 0.03 |
| fl oz (US) | gal (US) | 0.0078125 | 0.01 | 0.01 | 0 |
| fl oz (US) | in³ | 1.804681731361864 | 1.81 | 1.8 | 1.8 |
| fl oz (US) | ft³ | 0.001044376001945523 | 0.01 | 0 | 0 |
| qt (US) | ml | 946.352 | 946.36 | 946.35 | 946.35 |
| qt (US) | L | 0.946352 | 0.95 | 0.95 | 0.94 |
| qt (US) | m³ | 0.000946352 | 0.01 | 0 | 0 |
| qt (US) | fl oz (US) | 32 | 32 | 32 | 32 |
| qt (US) | gal (US) | 0.25 | 0.25 | 0.25 | 0.25 |
| qt (US) | in³ | 57.749815403579646 | 57.75 | 57.75 | 57.74 |
| qt (US) | ft³ | 0.03342003206225674 | 0.04 | 0.03 | 0.03 |
| gal (US) | ml | 3785.408 | 3785.41 | 3785.41 | 3785.4 |
| gal (US) | L | 3.785408 | 3.79 | 3.79 | 3.78 |
| gal (US) | m³ | 0.003785408 | 0.01 | 0 | 0 |
| gal (US) | fl oz (US) | 128 | 128 | 128 | 128 |
| gal (US) | qt (US) | 4 | 4 | 4 | 4 |
| gal (US) | in³ | 230.99926161431858 | 231 | 231 | 230.99 |
| gal (US) | ft³ | 0.13368012824902695 | 0.14 | 0.13 | 0.13 |
| in³ | ml | 16.3871 | 16.39 | 16.39 | 16.38 |
| in³ | L | 0.0163871 | 0.02 | 0.02 | 0.01 |
| in³ | m³ | 1.63871e-05 | 0.01 | 0 | 0 |
| in³ | fl oz (US) | 0.5541143253250377 | 0.56 | 0.55 | 0.55 |
| in³ | qt (US) | 0.017316072666407428 | 0.02 | 0.02 | 0.01 |
| in³ | gal (US) | 0.004329018166601857 | 0.01 | 0 | 0 |
| in³ | ft³ | 0.0005787037037037037 | 0.01 | 0 | 0 |
| ft³ | ml | 28316.9088 | 28316.91 | 28316.91 | 28316.9 |
| ft³ | L | 28.3169088 | 28.32 | 28.32 | 28.31 |
| ft³ | m³ | 0.028316908800000002 | 0.03 | 0.03 | 0.02 |
| ft³ | fl oz (US) | 957.509554161665 | 957.51 | 957.51 | 957.5 |
| ft³ | qt (US) | 29.922173567552033 | 29.93 | 29.92 | 29.92 |
| ft³ | gal (US) | 7.480543391888008 | 7.49 | 7.48 | 7.48 |
| ft³ | in³ | 1728 | 1728 | 1728 | 1728 |

#### Working time tree

| Source unit | Destination unit | Exact ratio (one source unit in destination units) | Result, default rounding away from zero | Result, half-up rounding | Result, rounding towards zero |
|---|---|---|---|---|---|
| Minutes | Hours | 0.0166667 | 0.02 | 0.02 | 0.01 |
| Minutes | Days | 0.0020833375 | 0.01 | 0 | 0 |
| Hours | Minutes | 59.999880000240005 | 60 | 60 | 59.99 |
| Hours | Days | 0.125 | 0.13 | 0.13 | 0.12 |
| Days | Minutes | 479.99904000192004 | 480 | 480 | 479.99 |
| Days | Hours | 8 | 8 | 8 | 8 |

#### Length tree

| Source unit | Destination unit | Exact ratio (one source unit in destination units) | Result, default rounding away from zero | Result, half-up rounding | Result, rounding towards zero |
|---|---|---|---|---|---|
| mm | cm | 0.1 | 0.1 | 0.1 | 0.1 |
| mm | m | 0.001 | 0.01 | 0 | 0 |
| mm | km | 1e-06 | 0.01 | 0 | 0 |
| mm | in | 0.03937007874015748 | 0.04 | 0.04 | 0.03 |
| mm | ft | 0.0032808398950131237 | 0.01 | 0 | 0 |
| mm | yd | 0.0010936132983377078 | 0.01 | 0 | 0 |
| mm | mi | 6.213711922373341e-07 | 0.01 | 0 | 0 |
| cm | mm | 10 | 10 | 10 | 10 |
| cm | m | 0.01 | 0.01 | 0.01 | 0.01 |
| cm | km | 1e-05 | 0.01 | 0 | 0 |
| cm | in | 0.3937007874015748 | 0.4 | 0.39 | 0.39 |
| cm | ft | 0.03280839895013124 | 0.04 | 0.03 | 0.03 |
| cm | yd | 0.010936132983377079 | 0.02 | 0.01 | 0.01 |
| cm | mi | 6.213711922373341e-06 | 0.01 | 0 | 0 |
| m | mm | 1000 | 1000 | 1000 | 1000 |
| m | cm | 100 | 100 | 100 | 100 |
| m | km | 0.001 | 0.01 | 0 | 0 |
| m | in | 39.37007874015748 | 39.38 | 39.37 | 39.37 |
| m | ft | 3.280839895013124 | 3.29 | 3.28 | 3.28 |
| m | yd | 1.093613298337708 | 1.1 | 1.09 | 1.09 |
| m | mi | 0.000621371192237334 | 0.01 | 0 | 0 |
| km | mm | 1000000 | 1000000 | 1000000 | 1000000 |
| km | cm | 100000 | 100000 | 100000 | 100000 |
| km | m | 1000 | 1000 | 1000 | 1000 |
| km | in | 39370.078740157485 | 39370.08 | 39370.08 | 39370.07 |
| km | ft | 3280.8398950131236 | 3280.84 | 3280.84 | 3280.83 |
| km | yd | 1093.6132983377079 | 1093.62 | 1093.61 | 1093.61 |
| km | mi | 0.6213711922373341 | 0.63 | 0.62 | 0.62 |
| in | mm | 25.4 | 25.4 | 25.4 | 25.4 |
| in | cm | 2.54 | 2.54 | 2.54 | 2.54 |
| in | m | 0.0254 | 0.03 | 0.03 | 0.02 |
| in | km | 2.5399999999999997e-05 | 0.01 | 0 | 0 |
| in | ft | 0.08333333333333334 | 0.09 | 0.08 | 0.08 |
| in | yd | 0.02777777777777778 | 0.03 | 0.03 | 0.02 |
| in | mi | 1.5782828282828283e-05 | 0.01 | 0 | 0 |
| ft | mm | 304.79999999999995 | 304.8 | 304.8 | 304.8 |
| ft | cm | 30.479999999999997 | 30.48 | 30.48 | 30.48 |
| ft | m | 0.30479999999999996 | 0.31 | 0.3 | 0.3 |
| ft | km | 0.00030479999999999993 | 0.01 | 0 | 0 |
| ft | in | 11.999999999999998 | 12 | 12 | 12 |
| ft | yd | 0.3333333333333333 | 0.34 | 0.33 | 0.33 |
| ft | mi | 0.0001893939393939394 | 0.01 | 0 | 0 |
| yd | mm | 914.3999999999999 | 914.4 | 914.4 | 914.4 |
| yd | cm | 91.43999999999998 | 91.44 | 91.44 | 91.44 |
| yd | m | 0.9143999999999999 | 0.92 | 0.91 | 0.91 |
| yd | km | 0.0009143999999999999 | 0.01 | 0 | 0 |
| yd | in | 36 | 36 | 36 | 36 |
| yd | ft | 3 | 3 | 3 | 3 |
| yd | mi | 0.0005681818181818182 | 0.01 | 0 | 0 |
| mi | mm | 1609343.9999999998 | 1609344 | 1609344 | 1609344 |
| mi | cm | 160934.39999999997 | 160934.4 | 160934.4 | 160934.4 |
| mi | m | 1609.3439999999998 | 1609.35 | 1609.34 | 1609.34 |
| mi | km | 1.6093439999999997 | 1.61 | 1.61 | 1.6 |
| mi | in | 63359.99999999999 | 63360 | 63360 | 63360 |
| mi | ft | 5280 | 5280 | 5280 | 5280 |
| mi | yd | 1760 | 1760 | 1760 | 1760 |

#### Surface tree

| Source unit | Destination unit | Exact ratio (one source unit in destination units) | Result, default rounding away from zero | Result, half-up rounding | Result, rounding towards zero |
|---|---|---|---|---|---|
| m² | ft² | 10.763915051182416 | 10.77 | 10.76 | 10.76 |
| ft² | m² | 0.092903 | 0.1 | 0.09 | 0.09 |
