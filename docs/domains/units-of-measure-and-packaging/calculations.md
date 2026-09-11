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
| 2.675 | two digits, that is a step of one hundredth | half away from zero | 2.68 | The binary value is slightly below the tie; the compensation term lifts it over. Without the term the answer would be 2.67. |
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

The values in the name column are the delivered names, reproduced exactly because documents and
exchanged files print them. Several are conventional short forms; their full names in words are:
`mm` millimetre, `cm` centimetre, `m` metre, `km` kilometre, `in` inch, `ft` foot, `yd` yard,
`mi` mile, `m²` square metre, `ft²` square foot, `ml` millilitre, `L` litre, `m³` cubic metre,
`fl oz (US)` United States fluid ounce, `qt (US)` United States quart, `gal (US)` United States
gallon, `in³` cubic inch, `ft³` cubic foot, `g` gram, `kg` kilogram, `Ton` metric tonne of one
thousand kilograms, `oz` ounce, `lb` pound, `KWH` kilowatt hour.

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

Seven trees exist as delivered. The smallest are surface, with two units, and energy, with one.
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

The unit names in the source and destination columns are the delivered names, whose full names
in words are listed at the head of section 5.

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

### 7.1 Cross-tree pairs

Every pair of units drawn from two different trees is **not convertible**. The table of
convertible tree pairs is the diagonal and nothing else:

| | Counting | Working time | Length | Surface | Volume | Mass | Energy |
|---|---|---|---|---|---|---|---|
| **Counting** | convertible | no | no | no | no | no | no |
| **Working time** | no | convertible | no | no | no | no | no |
| **Length** | no | no | convertible | no | no | no | no |
| **Surface** | no | no | no | convertible | no | no | no |
| **Volume** | no | no | no | no | convertible | no | no |
| **Mass** | no | no | no | no | no | convertible | no |
| **Energy** | no | no | no | no | no | no | convertible |

In particular: **there is no relationship between length and volume**, even though the shipped
volume tree contains a cubic inch and the shipped length tree contains an inch. A rebuild must not
attempt to derive one from the other. Likewise mass and volume are unrelated; a density is not
part of this model.

---

## 8. The mandatory worked examples

Each example is worked from the stored data, through the algorithm of section 6, to the stored
result. Unless stated, the precision is the shipped two digits and the rounding method is the
conversion default, away from zero.

### 8.1 Two and a half dozens into units

**Given.** A quantity of two and one half, expressed in `Dozens` (absolute quantity twelve). The
destination is `Units` (absolute quantity one).

**Step 1 — short-circuits.** The source unit exists; the quantity is not zero; source and
destination differ; a destination is supplied. No short-circuit applies.

**Step 2 — scale.**

```formula
amount = 2.5 × 12 = 30
amount = 30 ÷ 1 = 30
```

**Step 3 — round.**

```formula
result = round_away_from_zero( 30 , step = 0.01 ) = 30
```

Thirty is already a whole multiple of one hundredth, so no rounding occurs. The compensation term
at a normalised magnitude of three thousand is about three thousand divided by two raised to the
fiftieth, far below one step, so it does not push the value to the next step.

**Result: 30 units.**

**Under every rounding method** the answer is thirty, because the exact value lands on the grid.
At a precision of zero digits the answer is still thirty, for the same reason.

**Interpretation.** Two and a half dozens is a legitimate quantity to enter: nothing forbids a
fractional number of packagings. The system will happily plan a move of thirty items described as
"two and a half dozens". If the business requires whole packagings, that is enforced by the
reservation policy of section 11, not by the conversion.

### 8.2 Seven units into dozens at a rounding of one hundredth

**Given.** A quantity of seven, expressed in `Units` (absolute quantity one). The destination is
`Dozens` (absolute quantity twelve). The precision is two digits, so the step is one hundredth.

**Step 1 — short-circuits.** None applies.

**Step 2 — scale.**

```formula
amount = 7 × 1 = 7
amount = 7 ÷ 12 = 0.5833333333333334
```

**Step 3 — round, away from zero (the default).** Normalise by multiplying by one hundred:
fifty-eight and one third. The compensation term at that magnitude is about fifty-eight divided
by two raised to the fiftieth. Add one minus the compensation term, with the sign of the value:
fifty-nine and one third minus a negligible amount. Truncate towards zero: fifty-nine.
Denormalise by dividing by one hundred.

```formula
result = 0.59 dozens
```

**Step 3 alternative — round, half away from zero.** Normalise: fifty-eight and one third. Add
the compensation term. Round half away from zero to a whole number: fifty-eight. Denormalise.

```formula
result = 0.58 dozens
```

**Step 3 alternative — round, towards zero.** Fifty-eight. Result: **0.58 dozens.**

**Results at other precisions.**

| Precision digits | Step | Away from zero | Half away from zero | Towards zero |
|---|---|---|---|---|
| 0 | one | 1 | 1 | 0 |
| 2 | one hundredth | 0.59 | 0.58 | 0.58 |
| 3 | one thousandth | 0.584 | 0.583 | 0.583 |
| 4 | one ten-thousandth | 0.5834 | 0.5833 | 0.5833 |

**Interpretation.** At the shipped precision, seven items cannot be described exactly in dozens.
The default conversion over-states by one hundredth of a dozen, which is twelve hundredths of an
item. This is intentional: a system that under-stated would allow a customer to be shipped less
than they ordered after a unit change.

### 8.3 One pound into kilograms

**Given.** A quantity of one, expressed in `lb` (absolute quantity four hundred fifty-three and
five hundred ninety-two thousandths). The destination is `kg` (absolute quantity one thousand).

**Step 1 — short-circuits.** None applies.

**Step 2 — scale.**

```formula
amount = 1 × 453.592 = 453.592
amount = 453.592 ÷ 1000 = 0.453592
```

**Step 3 — round.**

| Precision digits | Away from zero | Half away from zero | Towards zero |
|---|---|---|---|
| 0 | 1 | 0 | 0 |
| 2 (shipped) | **0.46** | 0.45 | 0.45 |
| 3 | 0.454 | 0.454 | 0.453 |
| 6 | 0.453592 | 0.453592 | 0.453592 |

**Result at the shipped precision: 0.46 kilograms.**

**Where the four hundred fifty-three and five hundred ninety-two thousandths comes from.** The
pound is not defined against the gram directly. It is defined as sixteen ounces, and the ounce as
twenty-eight and three thousand four hundred ninety-five ten-thousandths grams. The chain is:

```formula
absolute_quantity(oz) = 28.3495 × 1 = 28.3495
absolute_quantity(lb) = 16 × 28.3495 = 453.592
```

This is a rounded international pound: the exact value is four hundred fifty-three and fifty-nine
thousand two hundred thirty-seven ten-millionths grams. The shipped data is therefore accurate to
seven significant figures, not exact. A rebuild must use the shipped chain, not the exact
definition, or the conversion matrix will not match.

**The reverse direction.** One kilogram into pounds:

```formula
amount = 1 × 1000 ÷ 453.592 = 2.2046244201837775
result (away from zero)      = 2.21
result (half away from zero) = 2.2
result (towards zero)        = 2.2
```

### 8.4 One hundred grams into kilograms at a rounding of one thousandth

**Given.** A quantity of one hundred, expressed in `g` (absolute quantity one). The destination is
`kg` (absolute quantity one thousand). The precision is three digits, so the step is one
thousandth.

**Step 1 — short-circuits.** None applies.

**Step 2 — scale.**

```formula
amount = 100 × 1 = 100
amount = 100 ÷ 1000 = 0.1
```

**Step 3 — round.** Normalise by multiplying by one thousand (the accurately inverted step, taken
from the lookup table, is exactly one thousand): one hundred. One hundred is already whole, so
every rounding method returns one hundred. Denormalise by dividing by one thousand.

```formula
result = 0.1 kilograms
```

**Result: 0.1 kilograms**, under every rounding method, and also at precisions of one and two
digits. At a precision of zero digits the results diverge: away from zero gives one kilogram,
half away from zero gives zero, towards zero gives zero.

**Contrast with one gram into kilograms.** One gram is one thousandth of a kilogram, which is
exactly representable at three digits but **not** at the shipped two digits:

| Precision digits | Exact value | Away from zero | Half away from zero | Towards zero |
|---|---|---|---|---|
| 2 (shipped) | 0.001 | **0.01** | 0 | 0 |
| 3 | 0.001 | 0.001 | 0.001 | 0.001 |
| 4 | 0.001 | 0.001 | 0.001 | 0.001 |

At the shipped precision one gram becomes **ten** grams' worth of kilogram under the default
method, and **nothing at all** under half away from zero. This is the single most dangerous
configuration in the domain: a business that keeps stock in grams and sells in kilograms must
raise the `Product Unit` precision to at least three digits, or accept a ten-fold rounding error
on single-gram quantities.

### 8.5 Ordering five boxes of twelve when stock is kept in units

**Given.**

- A product whose own unit is `Units`.
- An additional trading unit `Box of 12`: reference unit `Units`, contains twelve, absolute
  quantity twelve.
- A customer order line for a quantity of five in `Box of 12`.
- Stock is counted, reserved and moved in `Units`.

**Step 1 — the order line.** The line stores a quantity of five and a unit of `Box of 12`. The
allowed-unit list for the line is the product's own unit together with its additional units, so
`Box of 12` is selectable. The price on the line is a price per box (see 8.6).

**Step 2 — the stock move created on confirmation.** The move inherits the line's quantity and
the line's unit:

- the move's demand is five;
- the move's unit is `Box of 12`;
- the move's packaging unit is set from the originating document's unit, which is `Box of 12`;
- the move's packaging quantity is the demand converted from the move's unit into the packaging
  unit, which is an identity here and gives five.

**Step 3 — the move's real quantity.** The move stores a second copy of the quantity in the
product's own unit, rounded **half away from zero**:

```formula
real_quantity = round_half_away_from_zero( 5 × 12 ÷ 1 , step = 0.01 ) = 60
```

**Step 4 — reservation.** The warehouse reserves against quantities on hand, which are held in
the product's own unit. It needs sixty units. Suppose ninety-two units are on hand and
unreserved.

Because the move's unit is not the product's own unit, the reservation applies a protective
double conversion (section 12.4): the available quantity of sixty (the smaller of the wanted
sixty and the available ninety-two) is converted **towards zero** into the move's unit, then back
**half away from zero** into the product's unit:

```formula
quantity_in_move_unit  = round_towards_zero( 60 × 1 ÷ 12 , step = 0.01 ) = 5
quantity_in_own_unit   = round_half_away_from_zero( 5 × 12 ÷ 1 , step = 0.01 ) = 60
```

Sixty units are reserved. The purpose of the double conversion is visible when only fifty-eight
units are available: fifty-eight units is four and eighty-three hundredths boxes towards zero,
which is fifty-seven and ninety-six hundredths units back — so fifty-seven and ninety-six
hundredths are reserved rather than fifty-eight, guaranteeing the reservation is expressible in
the move's unit.

**Step 5 — the move line.** A move line is created with a quantity in the operator's unit. If the
operator picks in `Units`, the move line's unit is `Units` and its quantity is sixty; its
quantity in the product's unit is sixty. If the operator picks in `Box of 12`, the move line's
unit is `Box of 12`, its quantity is five, and its quantity in the product's unit is

```formula
round_half_away_from_zero( 5 × 12 ÷ 1 ) = 60
```

**Step 6 — the move's picked quantity.** The move's picked quantity is the sum of its move lines'
quantities, each converted into the **move's** unit **without rounding**. With one move line of
sixty units:

```formula
picked_in_move_unit = 60 × 1 ÷ 12 = 5     (unrounded)
```

**Step 7 — quantities on hand.** Sixty units leave stock. Quantities on hand never mention boxes.

**Step 8 — the invoice.** The invoice line copies the order line's unit and quantity: five `Box of
12`. The invoiced quantity reported back onto the order line is converted from the invoice line's
unit into the order line's unit, which is an identity here.

**Summary table of the same five boxes, seen from each document.**

| Record | Unit stored | Quantity stored | Second copy in product's unit | Rounding used |
|---|---|---|---|---|
| Sales order line | `Box of 12` | 5 | none | — |
| Stock move | `Box of 12` | 5 (demand) | 60 (real quantity) | half away from zero |
| Stock move, packaging fields | `Box of 12` | 5 (packaging quantity) | — | away from zero |
| Stock move line, picked in units | `Units` | 60 | 60 | half away from zero |
| Stock move line, picked in boxes | `Box of 12` | 5 | 60 | half away from zero |
| Quantity on hand | implied `Units` | −60 | — | — |
| Invoice line | `Box of 12` | 5 | — | — |

**The same scenario with a box of twelve dozens.** If the additional unit is instead `Box of 12
Dozens` (absolute quantity one hundred forty-four), five boxes are seven hundred twenty units,
the move's real quantity is seven hundred twenty, and every other row of the table scales by
twelve. The arithmetic is identical; only the absolute quantity changes.

### 8.6 A price of twenty-four per dozen converted to a price per unit

**Given.** A price of twenty-four, expressed **per one `Dozens`**. The destination is a price
**per one `Units`**.

Price conversion is the *inverse* of quantity conversion, because a price is a quantity in the
denominator. The formula is in section 10; applied here:

```formula
price_per_destination = price_per_source × absolute_quantity( destination_unit ) ÷ absolute_quantity( source_unit )
price_per_unit        = 24 × 1 ÷ 12 = 2
```

**Result: 2 per unit.** No rounding is applied: price conversion never rounds.

**Check by multiplication.** Twelve units at two each is twenty-four, which is one dozen at
twenty-four. The conversion is consistent.

**The reverse direction.** A price of two per unit converted to a price per dozen:

```formula
price_per_dozen = 2 × 12 ÷ 1 = 24
```

**A price that does not divide evenly.** Twenty-four per box of twelve dozens, converted to a
price per unit:

```formula
price_per_unit = 24 × 1 ÷ 144 = 0.16666666666666666
```

The result is **not** rounded to sixteen hundredths or seventeen hundredths by this operation. It
is returned at full precision, and the caller — a price list rule, an order line, an invoice
line — applies its own currency rounding afterwards. A rebuild that rounds inside the price
conversion will produce line totals that differ by fractions of a currency unit on large
quantities. One hundred forty-four units at the unrounded price is exactly twenty-four; at a price
rounded to seventeen hundredths it would be twenty-four and forty-eight hundredths.

**A price per unit converted to a price per pack of six.** One and one half per unit:

```formula
price_per_pack = 1.5 × 6 ÷ 1 = 9
```

**A price per kilogram converted to a price per gram.** One hundred per kilogram:

```formula
price_per_gram = 100 × 1 ÷ 1000 = 0.1
```

**A realistic retail price per unit converted to a price per dozen.** Nineteen and ninety-nine
hundredths per unit:

```formula
price_per_dozen = 19.99 × 12 ÷ 1 = 239.88
```

---

## 9. The shared-ancestor test

### 9.1 Purpose

Decides whether two units can meaningfully be converted. It is the only tree-membership test in
the domain, and it works purely on the materialised paths, so it costs no database access beyond
the two records.

### 9.2 Algorithm

1. Assert each side is exactly one unit.
2. Split each unit's hierarchy path on the solidus character, producing an ordered list of
   ancestor identifiers from the root downwards.
3. Walk the two lists in parallel from the beginning. While the elements are equal, collect them.
   Stop at the first difference or when either list is exhausted.
4. Return true when at least one element was collected, false otherwise.

### 9.3 Consequences

- Two units in the same tree always share at least the root element and therefore return true,
  whatever their depth or their relative position.
- Two units in different trees differ at the first element and therefore return false.
- A unit shares an ancestor with itself.
- The test is symmetric.
- The test says nothing about *how far apart* two units are, and nothing about whether the
  conversion will round to zero.

### 9.4 Where the test is applied

| Caller | What it guards |
|---|---|
| Service product configuration | Whether a product's own unit belongs to the working-time tree, which decides whether the product can be sold as time and delivered from recorded time. |
| Sales line delivered-quantity recording | Whether the line's unit belongs to the working-time tree, and whether the company's time unit can be converted into the line's unit. |
| Label printing for picked goods | Whether the picked unit belongs to the counting tree, which decides whether one label per item is meaningful. |
| Lot and serial number label printing | The same test on the move line's unit. |
| Incoming electronic document processing | Whether the unit named on the incoming document belongs to the same tree as the matched product's own unit; if not, the document's unit is discarded and the product's own unit is used instead. |

### 9.5 Worked examples

| First unit | Path | Second unit | Path | Common prefix | Result |
|---|---|---|---|---|---|
| `Dozens` | root `Units`, then `Dozens` | `Pack of 6` | root `Units`, then `Pack of 6` | the root | share an ancestor |
| `Units` | root `Units` | `Pallet of 40 Boxes` | root `Units`, `Dozens`, `Box`, `Pallet` | the root | share an ancestor |
| `kg` | root `g`, then `kg` | `L` | root `ml`, then `L` | none | do not share an ancestor |
| `Hours` | root `Hours` | `Hours` | root `Hours` | the root | share an ancestor |
| `in³` | root `ml`, `L`, `in³` | `in` | root `mm`, `cm`, `in` | none | do not share an ancestor |

---

## 10. Price conversion

### 10.1 Why it differs from quantity conversion

A quantity has the unit in the numerator: *thirty items*. A price has the unit in the
denominator: *two currency units **per** item*. Converting the denominator therefore inverts the
ratio. Converting a price per dozen into a price per unit **divides** by twelve, whereas
converting a quantity in dozens into units **multiplies** by twelve.

### 10.2 Statement

```formula
price_in_destination_unit = price_in_source_unit × absolute_quantity( destination_unit ) ÷ absolute_quantity( source_unit )
```

### 10.3 Algorithm

1. Assert the source is exactly one unit.
2. **Short-circuit.** If there is no source unit, or the price is zero (or otherwise falsy), or
   there is no destination unit, or the source and destination units are the same record, return
   the price unchanged.
3. Multiply the price by the destination unit's absolute quantity.
4. Divide by the source unit's absolute quantity.
5. **Return without rounding.**

### 10.4 Properties a rebuild must preserve

- **No rounding, ever.** Neither to the `Product Unit` precision nor to any currency precision.
  Rounding is the caller's business.
- **No tree check.** As with quantity conversion, the operation will convert across trees and
  produce nonsense; callers guard it.
- **Exactly invertible in exact arithmetic**, and *almost* invertible in floating point:
  converting a price from dozens to units and back yields the original price for every pair in
  the counting tree, but not necessarily for pairs in the length tree whose absolute quantities
  are not exactly representable.
- **The multiplication comes first.** Multiplying by the destination absolute quantity and then
  dividing by the source keeps more significant digits than computing the ratio first, in the
  same way as quantity conversion.

### 10.5 Worked examples

| Price | Per source unit | To destination unit | Arithmetic | Result |
|---|---|---|---|---|
| 24 | `Dozens` | `Units` | 24 × 1 ÷ 12 | 2 |
| 2 | `Units` | `Dozens` | 2 × 12 ÷ 1 | 24 |
| 24 | `Box of 12 Dozens` | `Units` | 24 × 1 ÷ 144 | 0.16666666666666666 |
| 24 | `Box of 12` | `Units` | 24 × 1 ÷ 12 | 2 |
| 1.5 | `Units` | `Pack of 6` | 1.5 × 6 ÷ 1 | 9 |
| 19.99 | `Units` | `Dozens` | 19.99 × 12 ÷ 1 | 239.88 |
| 2 | `g` | `Ton` | 2 × 1000000 ÷ 1 | 2000000 |
| 100 | `kg` | `g` | 100 × 1 ÷ 1000 | 0.1 |
| 0 | anything | anything | short-circuit | 0 |
| any | `Units` | `Units` | short-circuit | unchanged |

### 10.6 Where price conversion is used

| Caller | From | To | Purpose |
|---|---|---|---|
| Product price computation | The product's own unit | The unit asked for by the caller | Express the catalogue price, or the cost, per the unit the document uses. Applied **before** currency conversion. |
| Price list rule, fixed price | The product's own unit | The document's unit | A fixed price entered on a rule is understood as a price per the product's own unit and is scaled to the document's unit. |
| Price list rule, surcharge | The product's own unit | The document's unit | The surcharge added by a formula rule is scaled the same way. |
| Price list rule, minimum margin | The product's own unit | The document's unit | Scaled before being used as a floor. |
| Price list rule, maximum margin | The product's own unit | The document's unit | Scaled before being used as a ceiling. |
| Vendor price list line, discounted price | The vendor's unit | The product's own unit | Convert the vendor's quoted price into a price per the product's own unit, then apply the discount. |
| Purchase order line, price per product unit | The line's unit | The product's own unit | A read-only display of the line price in the product's own unit. |
| Purchase order line, price from the standard cost | The product's own unit | The line's unit | When no vendor price applies, the cost is scaled to the line's unit before tax-inclusion correction and currency conversion. |
| Purchase order line, price from a vendor | The vendor's unit | The line's unit | The vendor's price is scaled to whatever unit the buyer chose. |
| Storefront list price | The product's own unit | The unit chosen by the visitor | The displayed price per the chosen packaging. |
| Bill of materials cost report | The component's own unit | The bill line's unit | The component cost per bill-line unit, then multiplied by the line quantity. |
| Production overview report | The product's own unit | The move's unit | The cost per move unit. |

### 10.7 The price list rule formula with unit conversion in place

The complete formula for a rule of the formula kind, with the unit scaling shown explicitly, is:

```formula
base_price   = price_of_the_chosen_base , expressed per destination_unit , converted to the target currency , unrounded
price        = base_price − ( base_price × discount_percentage ÷ 100 )
price        = round_to_step( price , price_rounding_step )                         if a rounding step is configured
price        = price + convert_price( surcharge , product_unit → destination_unit ) if a surcharge is configured
price        = max( price , base_price + convert_price( minimum_margin , product_unit → destination_unit ) )   if a minimum margin is configured
price        = min( price , base_price + convert_price( maximum_margin , product_unit → destination_unit ) )   if a maximum margin is configured
```

with the discount replaced by the negated markup when the base is the cost.

The **quantity** used to select the rule is converted the other way, into the product's own unit,
because a rule's minimum quantity is always expressed in the product's own unit:

```formula
quantity_for_rule_matching = convert_quantity( quantity , document_unit → product_unit , tolerate_failure = yes )
```

and a rule applies when its minimum quantity is zero, or the converted quantity is not below it.

**Worked example.** A product whose own unit is `Units`, catalogue price fifty. A rule says: for a
minimum quantity of twenty-four, a ten per cent discount. A customer orders three `Dozens`.

```formula
quantity_for_rule_matching = convert_quantity( 3 , Dozens → Units , away from zero ) = 36
36 ≥ 24 , so the rule applies
base_price  = convert_price( 50 , Units → Dozens ) = 50 × 12 ÷ 1 = 600
price       = 600 − ( 600 × 10 ÷ 100 ) = 540
```

The line shows five hundred forty per dozen, three dozens, one thousand six hundred twenty in
total, which is thirty-six items at forty-five each.

---

## 11. Whole-packaging rounding

### 11.1 Purpose

Snap a quantity, expressed in a product's own unit, onto a whole multiple of a packaging
quantity. Used when a business refuses to break a carton.

### 11.2 Inputs

| Input | Meaning |
|---|---|
| the packaging unit | The unit whose whole multiples the result must be. This is the unit the operation is invoked on. |
| product quantity | The quantity to snap, expressed in the destination unit below. |
| destination unit | The unit the quantity is expressed in — in practice always the product's own unit. |
| rounding method | Away from zero, half away from zero (the default) or towards zero. |

### 11.3 Algorithm

1. Assert the packaging unit is exactly one unit.
2. Compute the **packaging quantity**: convert a quantity of one from the packaging unit into the
   destination unit, with the *default* conversion rounding, that is away from zero, at the
   `Product Unit` precision.

   ```formula
   packaging_quantity = convert_quantity( 1 , packaging_unit → destination_unit )
   ```

3. **Identity short-circuit.** If the packaging unit and the destination unit are the same record,
   return the product quantity **unchanged**. This test happens *after* the packaging quantity is
   computed and before it is used.
4. If both the product quantity and the packaging quantity are non-zero:

   ```formula
   result = round_to_step( product_quantity ÷ packaging_quantity , step = 1 , rounding_method ) × packaging_quantity
   ```

   Note the step of **one**: the division is rounded to a whole number of packagings, and the
   whole number is then multiplied back.
5. Otherwise return the product quantity unchanged.

### 11.4 Why a remainder operation is forbidden

The obvious implementation — test whether the remainder of the quantity divided by the packaging
quantity is zero — is wrong in floating point and must not be used. Two counter-examples that the
implementation notes explicitly:

- eight remainder one and six tenths evaluates to one and five thousand nine hundred ninety-nine
  ten-thousandths and change, not zero, although eight is exactly five packs of one and six
  tenths;
- five and four tenths remainder one and eight tenths evaluates to about two ten-thousandths of a
  trillionth, not zero.

The division-round-multiply form is immune because the rounding step absorbs the error.

### 11.5 Worked examples

All at the shipped precision of two digits. The destination unit is `Units` throughout unless
stated.

| Packaging unit | Packaging quantity | Product quantity | Method | Quotient before rounding | Rounded quotient | Result |
|---|---|---|---|---|---|---|
| `Box of 12 Dozens` | 144 | 1600 | towards zero | 11.111… | 11 | 1584 |
| `Box of 12 Dozens` | 144 | 1600 | half away from zero | 11.111… | 11 | 1584 |
| `Box of 12 Dozens` | 144 | 1600 | away from zero | 11.111… | 12 | 1728 |
| `Pallet of 40 Boxes` | 5760 | 1600 | towards zero | 0.277… | 0 | 0 |
| `Pallet of 40 Boxes` | 5760 | 1600 | half away from zero | 0.277… | 0 | 0 |
| `Pallet of 40 Boxes` | 5760 | 1600 | away from zero | 0.277… | 1 | 5760 |
| `Dozens` | 12 | 20 | towards zero | 1.666… | 1 | 12 |
| `Dozens` | 12 | 20 | half away from zero | 1.666… | 2 | 24 |
| `Dozens` | 12 | 20 | away from zero | 1.666… | 2 | 24 |
| `Units` | 1 | 22.43 | towards zero | — | — | **22.43** (identity short-circuit) |
| `Pack of 6` | 6 | 0 | any | — | — | 0 |

The `Units` row is the reason the identity short-circuit exists. Without it, twenty-two and
forty-three hundredths divided by one, rounded to a whole number towards zero, times one, would
be twenty-two — the reservation would silently lose forty-three hundredths of an item every time
the policy was set to full packagings on a product with no packagings at all.

The `Pallet of 40 Boxes` rows with the away-from-zero method are why that method is never used by
the reservation caller: rounding *up* to a whole packaging would reserve five thousand seven
hundred sixty units when only one thousand six hundred exist.

### 11.6 The only caller, and its conditions

Whole-packaging rounding runs during reservation, and only when **both** of these hold:

1. the reservation call carries a packaging unit in its context — which happens when the move
   being reserved has a packaging unit, that is when it came from a sales or purchase document
   line; and
2. the product's category has its reservation policy set to `full`.

When both hold, the quantity that may be reserved is recomputed as:

```formula
available = whole_packaging_round( min( wanted_quantity , available_quantity ) , packaging_unit , product_unit , towards zero )
```

with the **towards zero** method, so a partial packaging is never reserved.

**Worked example.** A customer orders two `Pallet of 1000` — a pallet unit containing one thousand
units. One thousand six hundred units are on hand.

- Policy `partial`: one thousand six hundred are reserved.
- Policy `full`: the wanted quantity is two thousand, the available is one thousand six hundred,
  the smaller is one thousand six hundred; one thousand six hundred divided by one thousand is one
  and six tenths, rounded towards zero is one, multiplied back is one thousand. **One thousand are
  reserved** and six hundred stay free.

---

## 12. How a quantity crosses every document boundary

This section is the complete inventory of conversions performed by the rest of the system. For
each crossing it gives the source unit, the destination unit, the rounding method and whether the
result is stored.

### 12.1 The master table

| # | Crossing | Source unit | Destination unit | Rounding method | Stored? |
|---|---|---|---|---|---|
| 1 | Product catalogue price to a document line | The product's own unit | The line's unit | none (price conversion) | no |
| 2 | Product cost to a document line | The product's own unit | The line's unit | none (price conversion) | no |
| 3 | Price list rule matching | The document's unit | The product's own unit | away from zero, failure tolerated | no |
| 4 | Price list fixed price, surcharge, margins | The product's own unit | The document's unit | none (price conversion) | no |
| 5 | Vendor price list line, discounted price | The vendor's unit | The product's own unit | none (price conversion) | no |
| 6 | Vendor selection by minimum quantity | The document's unit | The vendor's unit | away from zero | no |
| 7 | Sales order line to stock move | — | — | the unit is copied, not converted | yes |
| 8 | Stock move demand to real quantity | The move's unit | The product's own unit | **half away from zero** | yes |
| 9 | Stock move demand to packaging quantity | The move's unit | The packaging unit | away from zero | yes |
| 10 | Stock move to move line (creation) | — | — | the unit is copied from the move | yes |
| 11 | Move line quantity to quantity in the product's unit | The move line's unit | The product's own unit | **half away from zero** | yes |
| 12 | Move lines summed into the move's picked quantity | Each move line's unit | The move's unit | **no rounding** | yes |
| 13 | Reservation, first leg | The product's own unit | The move's unit | **towards zero** | no |
| 14 | Reservation, second leg | The move's unit | The product's own unit | **half away from zero** | no |
| 15 | Reservation, whole-packaging snap | The packaging unit | The product's own unit | away from zero for the packaging quantity, **towards zero** for the snap | no |
| 16 | Move line quantity suggested from a quantity on hand | The product's own unit | The move line's unit | **half away from zero** | yes |
| 17 | Move demand shown against a move line | The move's unit | The move line's unit | **half away from zero** | no |
| 18 | Forecast availability | The move's unit | The product's own unit | **half away from zero** | no |
| 19 | Procurement to purchase order line | The procurement's unit | The product's own unit | **half away from zero** | no |
| 20 | Purchase order line quantity to total quantity | The line's unit | The product's own unit | away from zero | yes |
| 21 | Purchase order line to stock move | The line's unit | The product's own unit, unless propagation is enabled | **half away from zero** | yes |
| 22 | Received quantity reported onto a purchase order line | The move's unit | The line's unit | away from zero | yes |
| 23 | Invoiced quantity reported onto a purchase order line | The invoice line's unit | The order line's unit | away from zero | yes |
| 24 | Delivered quantity reported onto a sales order line | The move's unit | The line's unit | **half away from zero** | yes |
| 25 | Invoiced quantity reported onto a sales order line | The invoice line's unit | The order line's unit | away from zero, and **unrounded** for the down-payment comparison | yes |
| 26 | Recorded time reported onto a sales order line | The time record's unit | The line's unit | **half away from zero** | yes |
| 27 | Sales order line to procurement | The line's unit | The product's own unit, unless propagation is enabled | **half away from zero** | no |
| 28 | Bill of materials line to component quantity | The bill line's unit | The bill's unit | **no rounding** | no |
| 29 | Bill of materials component quantity, final step | The bill line's unit | itself | **away from zero** | no |
| 30 | Production order quantity to the bill's unit | The order's unit | The bill's unit | away from zero, and **unrounded** where a ratio is formed | no |
| 31 | Production order produced quantity | The order's unit | The product's own unit | **half away from zero** | no |
| 32 | Kit component quantity per kit | The bill line's unit | The component's own unit | **no rounding**, failure tolerated | no |
| 33 | Reordering rule quantity to order | The product's own unit | The rule's unit | **no rounding** | no |
| 34 | Picking weight from move lines | The move line's unit | The product's own unit | away from zero | no |
| 35 | Picking volume from moves | The move's unit | The product's own unit | away from zero | no |
| 36 | Package content weight | The move line's unit | The product's own unit | away from zero | no |
| 37 | Delivery method weight check | The line's unit | The product's own unit | away from zero | no |
| 38 | Counter sale to stock move | The move's unit | The product's own unit | **half away from zero**, and the transfer is refused if the result is zero | no |
| 39 | Storefront cart availability check | The product's own unit | The cart line's unit | away from zero, and **unrounded** for the available quantity | no |
| 40 | Catalogue quantity aggregation across lines | Each line's unit | The product's own unit | away from zero | no |

### 12.2 Reading the table

Three patterns account for nearly all of it.

- **Half away from zero** is used whenever the result is a *physical* quantity that will be
  compared with, or subtracted from, a quantity on hand. The reasoning is that a physical count
  should be the nearest representable number, not a systematically inflated one.
- **Away from zero** is used whenever the result is a *commercial* quantity that must not
  under-serve the counterparty: a packaging count on a delivery note, a received quantity
  reported back to a buyer, a rule-matching quantity.
- **No rounding at all** is used whenever the result is an intermediate value that will itself be
  converted or summed: the move's picked quantity, the component quantity inside a bill of
  materials expansion, the quantity to order on a reordering rule.

### 12.3 The unit propagation parameter

Crossings 21 and 27 depend on a system parameter.

| Parameter key | Value | Effect |
|---|---|---|
| `stock.propagate_uom` | `1` | The document line's unit is carried onto the stock move unchanged, and the quantity is converted from the line's unit into **the same line's unit** with half-away-from-zero rounding — that is, it is merely re-rounded onto the grid. |
| `stock.propagate_uom` | anything else, or absent | The move is created in the **product's own unit**, and the quantity is converted from the line's unit into the product's own unit with half-away-from-zero rounding. |

The default is therefore to *not* propagate: a purchase for five boxes becomes a receipt move for
sixty units, unless the parameter is set.

**Worked example, propagation off (the default).** A purchase line for five `Box of 12`:

```formula
move_quantity = round_half_away_from_zero( 5 × 12 ÷ 1 ) = 60
move_unit     = Units
```

**Worked example, propagation on.** The same line:

```formula
move_quantity = round_half_away_from_zero( 5 × 12 ÷ 12 ) = 5
move_unit     = Box of 12
```

### 12.4 The reservation double conversion, in full

Crossings 13 and 14 form a single protective idiom that a rebuild must copy exactly.

**Precondition.** The reservation is not in strict mode, a move unit was supplied, and that unit
differs from the product's own unit.

**Algorithm.**

1. Let the wanted quantity be the smaller of the quantity asked for and the quantity available,
   both in the product's own unit.
2. Convert it into the move's unit **towards zero**.
3. Convert the result back into the product's own unit **half away from zero**.
4. Reserve that.

**Why.** The move's unit may not be able to express the available quantity. If fifty-eight units
are available and the move is in boxes of twelve, reserving fifty-eight units would leave the move
holding four and eighty-three hundredths boxes, a quantity that cannot be picked as whole boxes
and that will not survive its own round trip. Converting down first guarantees that whatever is
reserved is exactly expressible in the move's unit.

**Worked examples.**

| Available in units | Move unit | Step 2, towards zero | Step 3, half away from zero | Reserved |
|---|---|---|---|---|
| 60 | `Box of 12` | 5 | 60 | 60 |
| 58 | `Box of 12` | 4.83 | 57.96 | 57.96 |
| 11 | `Box of 12` | 0.91 | 10.92 | 10.92 |
| 5 | `Box of 12` | 0.41 | 4.92 | 4.92 |
| 1600 | `Box of 12 Dozens` | 11.11 | 1599.84 | 1599.84 |

A further rule applies to serial-tracked products: after the double conversion, if the quantity
is not a whole number at the `Product Unit` precision, the reservation is reduced to zero.

### 12.5 Crossing a unit change on the product

If the product's own unit is changed while documents exist, **nothing is converted**. The unit
reference is replaced on the impacted records and every stored number stays as it was. A move for
sixty whose product changed from `Units` to `kg` becomes a move for sixty kilograms. This is
stated to the user before the change as "a conversion of one old unit equals one new unit".

The consequence for a rebuild: the unit change operation is a *relabelling*, not a conversion, and
must not recompute the real quantity of existing moves, the quantity in the product's unit of
existing move lines, or any quantity on hand.

---

## 13. Every other derived computation in the domain

### 13.1 The sequence of a unit

```formula
sequence = minimum( 1000 , truncate_towards_zero( contained_quantity × 100 ) )
```

applied only when the record is new or its sequence is zero.

Worked values: a contained quantity of one gives one hundred; six gives six hundred; twelve gives
one thousand (capped from one thousand two hundred); two and fifty-four hundredths gives two
hundred fifty-four; nine hundred twenty-nine ten-thousandths gives nine; one hundred sixty-six
ten-thousandths gives one; twenty-eight and three thousand four hundred ninety-five ten-thousandths
gives one thousand (capped from two thousand eight hundred thirty-four).

### 13.2 The rounding precision of a unit

```formula
rounding_precision = 10 ^ ( − digits_of( "Product Unit" ) )
```

### 13.3 The discounted vendor price

```formula
discounted_price = convert_price( vendor_price , vendor_unit → product_own_unit ) × ( 1 − discount_percentage ÷ 100 )
```

Worked example. A vendor quotes one hundred twenty per `Box of 12 Dozens` with a five per cent
discount, for a product whose own unit is `Units`:

```formula
converted = 120 × 1 ÷ 144 = 0.8333333333333334
discounted = 0.8333333333333334 × ( 1 − 5 ÷ 100 ) = 0.7916666666666667
```

### 13.4 The purchase line total quantity

```formula
total_quantity = convert_quantity( ordered_quantity , line_unit → product_own_unit )   if the units differ
total_quantity = ordered_quantity                                                       if they are the same
```

with the default away-from-zero rounding. Worked example: seven `Box of 12` gives eighty-four
units.

### 13.5 The purchase line gross unit price in the product's unit

Used when an accrual must be valued.

```formula
price = price_unit
price = price × ( 1 − discount_percentage ÷ 100 )                                  if a discount applies
price = tax_exclusive_total( price , quantity ) ÷ quantity                          if taxes apply
price = price × absolute_quantity( product_own_unit ) ÷ absolute_quantity( line_unit )   if the units differ
```

Note that the last step performs the price conversion **inline**, using the ratio of the two
absolute quantities directly rather than calling the price conversion operation. The arithmetic is
identical.

### 13.6 The packaging quantity on a move

```formula
packaging_quantity = convert_quantity( demand , move_unit → packaging_unit )
```

with the default away-from-zero rounding, and only when a packaging unit is set.

Worked examples, for a move whose unit is `Units`:

| Demand in units | Packaging unit | Packaging quantity |
|---|---|---|
| 60 | `Box of 12` | 5 |
| 58 | `Box of 12` | 4.84 |
| 1600 | `Box of 12 Dozens` | 11.12 |
| 1600 | `Pallet of 40 Boxes` | 0.28 |

### 13.7 The bulk weight of a transfer

```formula
bulk_weight = Σ over groups of ( line_count × convert_quantity( grouped_quantity , move_line_unit → product_own_unit ) × product_weight )
```

The grouping is by transfer, product, move line unit and quantity; the count of lines in each
group multiplies the converted quantity. Only move lines with no destination package contribute.

Worked example. A transfer holds three move lines of five `Box of 12` each for a product weighing
two (interpreted as kilograms or pounds by the system parameter). The group is (this transfer,
this product, `Box of 12`, five) with a count of three:

```formula
converted   = convert_quantity( 5 , Box of 12 → Units ) = 60
bulk_weight = 3 × 60 × 2 = 360
```

### 13.8 The shipping weight of a transfer

```formula
shipping_weight = bulk_weight + Σ over outermost destination packages of ( declared_shipping_weight or computed_package_weight )
```

with the computed package weight being the package type's tare plus the same content-weight sum
restricted to the move lines whose destination package is that package.

### 13.9 The shipping volume of a transfer

```formula
shipping_volume = Σ over moves of ( convert_quantity( picked_quantity , move_unit → product_own_unit ) × product_volume )
```

### 13.10 The content description of a package

For display, the contents of a package are grouped by (unit, product) and each group is rendered
as: the quantity — printed without a decimal part when it is a whole number — then, **only if the
reader holds the multiple-units group**, a space and the unit's name, then a space and the
product's name. A reader without the group sees only the quantity and the product name.

### 13.11 Component quantity for one unit of a kit

```formula
quantity_per_kit = Σ over bill lines of convert_quantity( line_quantity ÷ bill_quantity , bill_line_unit → component_own_unit , no rounding , tolerate failure )
```

The division by the bill's produced quantity happens **before** the conversion, and the conversion
is unrounded so that fractional components survive. Failure is tolerated so that a component whose
unit is in a different tree from the bill line's unit contributes its raw number rather than
aborting.

### 13.12 Component quantity for a production order

```formula
factor           = convert_quantity( order_quantity , order_unit → bill_unit , no rounding ) ÷ bill_quantity
line_quantity    = component_line_quantity × factor
line_quantity    = round_away_from_zero( line_quantity )
```

The final rounding is **away from zero** at the `Product Unit` precision: a production run never
plans to consume less of a component than the ratio demands.

Worked example. A bill produces one `Box of 12 Dozens` from three `kg` of material. A production
order asks for two and a half boxes.

```formula
factor        = 2.5 ÷ 1 = 2.5
line_quantity = 3 × 2.5 = 7.5 kg
rounded       = 7.5 kg
```

A second example where the rounding bites: the same bill consumes one `Units` of a fastener per
box, and the order is for one third of a box.

```formula
factor        = 0.33 ÷ 1 = 0.33
line_quantity = 1 × 0.33 = 0.33
rounded away from zero at two digits = 0.33
```

and with a fastener quantity of seven per box:

```formula
line_quantity = 7 × 0.33 = 2.31
rounded away from zero = 2.31
```

### 13.13 The unbuild ratio

```formula
ratio = unbuild_quantity ÷ convert_quantity( produced_quantity , production_unit → unbuild_unit )
```

when the unbuild is linked to a production order, and

```formula
ratio = convert_quantity( unbuild_quantity , unbuild_unit → bill_unit ) ÷ bill_quantity
```

when it is linked only to a bill of materials.

### 13.14 The catalogue aggregated quantity

When several lines of the same document hold the same product in different units, the catalogue
panel shows a single figure:

```formula
aggregated_quantity = Σ over lines of convert_quantity( line_quantity , line_unit → product_own_unit )
```

with the default away-from-zero rounding, so the aggregate may exceed the true total by up to one
hundredth per line.

---

## 14. Rounding edges and pathologies

### 14.1 Conversions that round to zero

A conversion rounds to zero when the exact value is smaller than half a step under half-away-from-
zero rounding, or smaller than a full step under towards-zero rounding. Under the **default**
away-from-zero rounding a non-zero quantity **never** rounds to zero: any positive value, however
small, is lifted to one step.

| Quantity | From | To | Exact | Away from zero | Half away from zero | Towards zero |
|---|---|---|---|---|---|---|
| 1 | `g` | `Ton` | 0.000001 | 0.01 | 0 | 0 |
| 1 | `Units` | `Pallet of 40 Boxes` | 0.000173… | 0.01 | 0 | 0 |
| 1 | `ml` | `m³` | 0.000001 | 0.01 | 0 | 0 |
| 1 | `mm` | `mi` | 0.000000621… | 0.01 | 0 | 0 |
| 1 | `Minutes` | `Days` | 0.00208… | 0.01 | 0 | 0 |

The practical rule: **the default conversion is safe against silent loss and unsafe against silent
inflation; the half-away-from-zero conversion is the reverse.** Both are used, deliberately, at
different boundaries.

### 14.2 The one place where rounding to zero is detected and refused

When a counter sale is turned into a stock transfer, every move whose unit differs from the
product's own unit has its demand converted half away from zero. If any such conversion yields
zero, the whole transfer is refused before any quantity is written. The user sees a message that
begins by stating that a conversion error occurred and that the following unit of measure
conversions result in a zero quantity due to rounding; then one line per offending pair, each
beginning with a space, a hyphen-minus, a space, then the word `From`, the source unit's name in
double quotation marks, the word `to`, and the destination unit's name in double quotation marks;
then a closing paragraph explaining that the issue occurs because the quantity becomes zero after
rounding during the conversion, and that to fix it the conversion factors or the rounding method
must be adjusted so that even the smallest quantity in the original unit does not round down to
zero in the target unit. The exact text is reproduced in
[`business-rules.md`](business-rules.md).

### 14.3 Conversions that inflate

| Quantity | From | To | Exact | Away from zero | Inflation |
|---|---|---|---|---|---|
| 1 | `Units` | `Dozens` | 0.08333… | 0.09 | eight per cent |
| 1 | `g` | `kg` | 0.001 | 0.01 | nine hundred per cent |
| 1 | `Units` | `Pallet of 40 Boxes` | 0.000173… | 0.01 | more than fifty-six fold |
| 7 | `Units` | `Dozens` | 0.58333… | 0.59 | one per cent |
| 1600 | `Units` | `Box of 12 Dozens` | 11.111… | 11.12 | eight hundredths of a per cent |

Inflation is proportionally worst for small quantities of large units. A rebuild that reports a
packaging quantity on a delivery note must expect that "nought point nought one pallets" is the
system's honest answer for one item.

### 14.4 Precision of zero digits

Setting the `Product Unit` precision to zero digits makes the step one, so every quantity in every
unit becomes a whole number.

| Quantity | From | To | Exact | Away from zero | Half away from zero | Towards zero |
|---|---|---|---|---|---|---|
| 2 | `Units` | a unit containing 20 | 0.1 | **1** | 0 | 0 |
| 7 | `Units` | `Dozens` | 0.583… | 1 | 1 | 0 |
| 1 | `lb` | `kg` | 0.4536 | 1 | 0 | 0 |
| 100 | `g` | `kg` | 0.1 | 1 | 0 | 0 |
| 2.5 | `Dozens` | `Units` | 30 | 30 | 30 | 30 |

The first row is the documented regression case: at zero digits, two units convert *up* to one
score.

### 14.5 Precision higher than the shipped value

Raising the precision reduces every inflation but does not remove it, because the ratios are
generally irrational in base ten.

| Quantity | From | To | 2 digits | 3 digits | 4 digits | 6 digits |
|---|---|---|---|---|---|---|
| 7 `Units` | | `Dozens` | 0.59 | 0.584 | 0.5834 | 0.583334 |
| 1 `lb` | | `kg` | 0.46 | 0.454 | 0.4536 | 0.453592 |
| 1 `g` | | `kg` | 0.01 | 0.001 | 0.001 | 0.001 |

### 14.6 The comparison-versus-difference trap

Two quantities that differ by less than one step may still compare as different, and two
quantities that compare as equal may have a non-zero difference. Every rule in this domain that
says "equal" means *compares as zero*, and every rule that says "is zero" means *the zero test
returns true*. A rebuild must not interchange them. The canonical counter-example, at two digits:
six thousandths and two thousandths compare as different (one hundredth against zero) while their
difference of four thousandths tests as zero.

### 14.7 Negative quantities

Every rounding method is defined symmetrically about zero: away from zero moves a negative value
further negative, towards zero moves it towards zero, and half away from zero sends a negative tie
to the more negative value. Negative quantities occur legitimately — a return move, an unreserve
operation, a negative quantity on hand — and convert by the same formulas with no special case.

| Quantity | From | To | Exact | Away from zero | Half away from zero | Towards zero |
|---|---|---|---|---|---|---|
| −7 | `Units` | `Dozens` | −0.58333… | −0.59 | −0.58 | −0.58 |
| −1 | `lb` | `kg` | −0.453592 | −0.46 | −0.45 | −0.45 |

### 14.8 A contained quantity that is not exactly one on a root unit

Forbidden by validation; see [`business-rules.md`](business-rules.md). Were it allowed, the
absolute quantity of the root would not be one and every conversion in the tree would be scaled by
a constant that cancels out — the arithmetic would still be self-consistent, which is why the rule
is a validation rather than an arithmetic necessity. The rule exists so that "absolute quantity"
always means "quantity of the root unit".

### 14.9 A contained quantity of zero

Forbidden by a stored check. Were it allowed, the absolute quantity would be zero and every
conversion *into* that unit would divide by zero.

### 14.10 A negative contained quantity

Not rejected by the stored check, which only forbids zero. A unit containing minus two would have
a negative absolute quantity and would convert quantities into negative numbers. **Industry-
standard default**: a rebuild should reject a contained quantity that is not strictly positive at
the user interface, while keeping the stored check exactly as specified so that existing data
loads unchanged.

---

## 15. Effects of changing the global precision

| Change | Immediate effect | Effect on stored data | Risk |
|---|---|---|---|
| Increase digits | Every unit's rounding precision shrinks; new conversions keep more digits. | None; stored quantities remain valid and are already on the coarser grid, which is a subset of the finer one. | Low. |
| Decrease digits | Every unit's rounding precision grows; new conversions keep fewer digits. | None; stored quantities are **not** rewritten and may now be off-grid. | High: an off-grid picked quantity is refused at completion time with the rounding message in [`business-rules.md`](business-rules.md), blocking the transfer until the quantity is edited or the precision restored. |
| Delete the `Product Unit` record | The lookup falls back to two digits. | None. | Moderate: silently restores the shipped behaviour. |

The lookup is cached, and the cache is cleared whenever any decimal precision record is created,
updated or deleted. A rebuild must clear its own cache on the same three events, or conversions
will keep using the old precision until the process restarts.

### 15.1 The off-grid refusal, in full

Before a move's picked quantity is applied, it is re-rounded at the `Product Unit` precision with
half-away-from-zero rounding and compared with itself:

```formula
rounded = round_half_away_from_zero( picked_quantity , precision_digits )
if compare( rounded , picked_quantity , precision_digits ) ≠ 0 then refuse
```

Every offending move contributes one paragraph to the error; the paragraphs are joined by newline
characters and raised together. The paragraph states that the quantity done for the named product
does not respect the rounding precision defined on the system, and asks the user to change the
quantity done or the rounding precision in the settings.
