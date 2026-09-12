# Multi-Currency — Calculations

This file is the arithmetic core of the domain. It specifies, with no gaps:

- the three notions of precision that coexist in the platform and must not be confused;
- the derivation of a currency's decimal places from its rounding factor;
- the single rounding routine every monetary amount passes through, written out as arithmetic
  including its error-compensation term, and the five rounding methods;
- the three-way comparison, the zero test, the exact euclidean division and the string rendering;
- the accurate reciprocal used so that dividing by a small fraction never loses a digit;
- the rate lookup for a currency, a company and a date, with every fallback;
- the conversion of an amount between two currencies, and the cross-rate that composes two rates
  into a single multiplication;
- the three interchangeable representations of a rate row and the arithmetic that ties them;
- the document rate of an invoice-like document and the derivation of every line balance from it;
- the pair of amounts every journal item carries and the arithmetic that binds them;
- the residuals an item offers for matching, per currency, and the rate attached to each;
- the choice of reconciliation currency, the partial amounts in all three currencies, and the
  tolerance band that suppresses a difference that is only a rounding artefact;
- the exchange difference amounts in every case that can arise, the account choice, the entry date
  and the reversal date;
- the recomputation of the two residual amounts and the closure test of a matched group;
- the current, historical and average factors of a reporting rate table;
- the three-currency arithmetic of a bank transaction and the bank-implied rates;
- the amount proposed by the payment registration screen;
- the formatting of an amount for a human reader and the rendering of an amount in words;
- three end-to-end worked examples that carry a document, a payment, a matching, an exchange
  difference and the resulting residuals through in full.

Formulas are written as plain arithmetic over quantities named in words. Worked numeric examples
follow every formula, carried to the last decimal the rule produces.

Throughout this file:

- *the company currency* is the main currency of the company that owns the record;
- *the document currency* is the currency written on the document, and *the item currency* the
  currency of a journal item;
- *round onto a currency* means the rounding routine of section 3 applied with that currency's
  rounding factor and the method half away from zero;
- *the rate of a currency* means the technical rate in force, produced by the lookup of section 7.

**Agreement with the quantity side.** The rounding routine, the zero test and the comparison
specified in sections 3 to 5 are the *same* routines the quantity side of the platform uses, and
they are documented there from the quantity point of view in
[../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md).
Nothing in this file contradicts that one. The differences are only in what supplies the precision
and in which method is the default at the point of call:

| | Quantity side | Money side |
|---|---|---|
| Where the precision comes from | The decimal precision record named `Product Unit`, shared by every unit of measure | The rounding factor of the currency in play |
| Typical precision value | One hundredth | One hundredth for most currencies, but also one, one thousandth and one ten-thousandth |
| Can the precision be a whole number? | No; in practice it is always a fraction | **Yes** — eighteen shipped currencies round onto multiples of one |
| Default method at the point of call | Away from zero, for a quantity conversion | Half away from zero, everywhere |

Contents:

1. [Notation and the three notions of precision](#1-notation-and-the-three-notions-of-precision)
2. [Decimal places from the rounding factor](#2-decimal-places-from-the-rounding-factor)
3. [The rounding routine](#3-the-rounding-routine)
4. [The three-way comparison](#4-the-three-way-comparison)
5. [The zero test](#5-the-zero-test)
6. [Exact euclidean division](#6-exact-euclidean-division)
7. [The rate lookup](#7-the-rate-lookup)
8. [Conversion between two currencies](#8-conversion-between-two-currencies)
9. [The three rate representations of a rate row](#9-the-three-rate-representations-of-a-rate-row)
10. [The document rate](#10-the-document-rate)
11. [The two amounts of a journal item](#11-the-two-amounts-of-a-journal-item)
12. [The residuals a journal item offers, per currency](#12-the-residuals-a-journal-item-offers-per-currency)
13. [Computing one partial matching](#13-computing-one-partial-matching)
14. [The exchange difference amounts](#14-the-exchange-difference-amounts)
15. [Choosing the exchange journal and the gain or loss account](#15-choosing-the-exchange-journal-and-the-gain-or-loss-account)
16. [The date of an exchange difference entry](#16-the-date-of-an-exchange-difference-entry)
17. [The date of the reversal of an exchange difference entry](#17-the-date-of-the-reversal-of-an-exchange-difference-entry)
18. [Recomputing the residual amounts of a journal item](#18-recomputing-the-residual-amounts-of-a-journal-item)
19. [Deciding that a group of matched items is fully reconciled](#19-deciding-that-a-group-of-matched-items-is-fully-reconciled)
20. [Reporting rate tables](#20-reporting-rate-tables)
21. [A bank transaction in up to three currencies](#21-a-bank-transaction-in-up-to-three-currencies)
22. [The amount proposed by the payment registration screen](#22-the-amount-proposed-by-the-payment-registration-screen)
23. [Rendering a number as a string](#23-rendering-a-number-as-a-string)
24. [Formatting an amount for a reader](#24-formatting-an-amount-for-a-reader)
25. [Writing an amount in words](#25-writing-an-amount-in-words)
26. [Worked example: a bill settled after the rate rose, producing a gain](#26-worked-example-a-bill-settled-after-the-rate-rose-producing-a-gain)
27. [Worked example: a partial payment leaving residuals in both currencies](#27-worked-example-a-partial-payment-leaving-residuals-in-both-currencies)
28. [Worked example: the mirror case, producing a loss](#28-worked-example-the-mirror-case-producing-a-loss)
29. [Reconciliation notes](#29-reconciliation-notes)

---

## 1. Notation and the three notions of precision

### 1.1 Named quantities

| Name used in formulas | Meaning |
|---|---|
| rounding factor | The rounding factor of a currency: the multiple every amount in that currency is snapped onto. |
| decimal places | The number of fractional digits used when an amount in that currency is stored, rendered or spelled. Derived from the rounding factor. |
| company currency | The currency a company keeps its books in. |
| document currency | The currency a document was agreed in; the currency of a journal item. |
| technical rate | The rate stored on a rate row: the value against the abstract reference of rate one. |
| conversion factor from one currency to another | How many units of the second currency one unit of the first buys, on a date, for a company. |
| balance | The amount of a journal item in the company currency, positive for a debit. |
| amount in currency | The amount of a journal item in its own currency, with the same sign as the balance. |
| residual | What is left to consume by matching on a journal item. |
| implied rate | The rate recovered from a posted item by dividing its amount in currency by its balance. |

### 1.2 Three notions of precision

They must not be confused.

| Notion | Where it lives | What it controls |
|---|---|---|
| Rounding factor | One value per currency | The grid onto which every amount in that currency is snapped. It need not be a power of ten: a rounding factor of five hundredths snaps amounts onto five-cent multiples. |
| Decimal places | Derived from the rounding factor and stored on the currency | The number of fractional digits used when a monetary value is written to storage, formatted for display, or spelled out in words. |
| Decimal precision registry entries | Named usages held in a separate registry, each with a digit count | The precision of non-monetary decimals such as quantities and discount percentages. A monetary amount never consults this registry: it uses its currency's own rounding factor. |

The decimal precision registry works as follows. Each entry has a unique usage name and a digit
count, defaulting to two digits. A lookup for a usage name that is not registered returns two.
Lookups are cached, and the cache is invalidated on every creation, write and deletion of an
entry. Two usages behave specially while an internal recursion guard is active during accounting
computations: the usage named `Discount` returns thirteen digits and the usage named
`Product Unit` returns ten digits, so that an intermediate result is not prematurely truncated.
Lowering the digit count of a registry entry produces a non-blocking warning titled "Warning for
*the usage name*" with the body "The precision has been reduced for *the usage name*.⏎Note that
existing data WON'T be updated by this change.⏎⏎As decimal precisions impact the whole system, this
may cause critical issues.⏎E.g. reducing the precision could disturb your financial
balance.⏎⏎Therefore, changing decimal precisions in a running database is not recommended."

### 1.3 Monetary storage precision

A monetary field is a decimal that names a companion field holding its currency. Two rules apply.

1. **On assignment into the in-memory record**, the value is snapped with the rounding routine of
   section 3 using the rounding factor of the currency held by the companion field at that moment.
   Assigning a monetary value while the companion currency field holds more than one currency is
   an error condition, because a single value cannot belong to two currencies at once.
2. **On writing to storage**, the value is snapped the same way and then rendered as a fixed-point
   decimal with exactly the decimal places of that currency. When the companion currency is empty
   the raw value is stored unchanged.

A replacement implementation must apply the rounding at both moments. Applying it only at write
time would let an unrounded intermediate value be read back within the same transaction; applying
it only at assignment would let an unrounded computed value reach storage.

---

## 2. Decimal places from the rounding factor

**Inputs:** the rounding factor of a currency.
**Output:** the decimal places, a whole number greater than or equal to zero.
**Precision:** exact whole-number arithmetic on the ceiling of a base-ten logarithm.

```formula
decimal places = ceiling( log10( 1 ÷ rounding factor ) )    when 0 < rounding factor < 1
decimal places = 0                                          otherwise
```

The rounding factor is authoritative; the decimal places are a stored consequence of it. Writing
the rounding factor rewrites the decimal places. Nothing ever writes the decimal places alone.

Worked values:

| Rounding factor | One divided by it | Base-ten logarithm | Ceiling | Decimal places |
|---|---|---|---|---|
| 0.01 | 100 | 2 | 2 | 2 |
| 0.001 | 1 000 | 3 | 3 | 3 |
| 0.0001 | 10 000 | 4 | 4 | 4 |
| 0.05 | 20 | 1.30103… | 2 | 2 |
| 0.5 | 2 | 0.30103… | 1 | 1 |
| 1 | not applicable, the branch condition is false | — | — | 0 |
| 1.00 | not applicable | — | — | 0 |
| 5 | not applicable | — | — | 0 |

Note the case of a rounding factor of five hundredths: the amount is snapped onto five-cent
multiples, yet it is stored and displayed with two fractional digits, because two digits are
needed to write a five-cent multiple. The rounding factor, not the decimal place count, decides
which values are reachable. A rounding factor of zero or a negative rounding factor is refused by
the stored check of [business-rules.md](business-rules.md) `MCUR-003`.

---

## 3. The rounding routine

### 3.1 Statement and algorithm

Given a value, a precision and a rounding method, produce a rounded value. The precision is
supplied **either** as a number of fractional digits **or** as a rounding factor, never both and
never neither (`MCUR-049`).

```formula
rounded value = denormalise( apply the method to ( normalise( value ) ) )
```

1. **Resolve the factor.**
   - When a rounding factor was supplied and a digit count was not, assert that the factor is
     strictly greater than zero and use it.
   - When a digit count was supplied and a factor was not, assert that the digit count is a whole
     number greater than or equal to zero, and set the factor to ten raised to minus that count.

     ```formula
     rounding factor = 10 ^ ( − precision digits )
     ```

   - When both were supplied, or neither, fail. The failure is a programming error, not a user
     error, and must be loud.

2. **Short-circuit.** When the factor is zero, or the value is zero, the result is zero.

3. **Choose the scaling direction.** Define two operations, *normalise* and *denormalise*:
   normalise divides by the factor and denormalise multiplies by it; **but when the factor is
   strictly smaller than one**, replace the factor by its accurately inverted value (section 3.2)
   and **swap** the two operations, so that normalise multiplies by the inverted factor and
   denormalise divides by it.

   The purpose is to replace a division by a small fraction — which loses accuracy — by a
   multiplication by a large whole number, which does not. With a factor of one hundredth,
   normalising multiplies by one hundred. With a factor of one, the factor is not smaller than one,
   so no swap happens and normalising divides by one, which changes nothing.

4. **Normalise.** Apply the normalise operation to the value. The result is the value counted in
   whole rounding steps, plus a fraction.

5. **Compute the compensation term.** Let the magnitude be the base-two logarithm of the absolute
   normalised value.

   ```formula
   compensation term = 2 ^ ( log2( | normalised value | ) − 50 )
   ```

   The minimal term that repairs a single unit in the last place of a double-precision number
   would use fifty-two rather than fifty. Fifty is used deliberately, so that error accumulated
   over several prior floating-point operations is also absorbed, while remaining far too small to
   change a genuine result. A rebuild that uses fifty-two, or omits the term altogether, produces
   different answers on values such as two point six seven five, whose binary representation lies
   very slightly below the tie. See the worked example in section 3.3.

6. **Apply the method** to the normalised value:
   - **half away from zero:** add the compensation term carrying the sign of the normalised value,
     then round half away from zero to a whole number;
   - **half to even:** take the whole part below the value, its floor; take the absolute difference
     between the normalised value and that floor; when that difference is within the compensation
     term of one half the value is a tie, and the result is the floor plus one when the floor is
     odd and the floor itself when it is even; when it is not a tie, round half away from zero to a
     whole number;
   - **half towards zero:** subtract the compensation term carrying the sign of the normalised
     value, then round half away from zero to a whole number;
   - **away from zero:** add, carrying the sign of the normalised value, the quantity one minus the
     compensation term, then truncate towards zero;
   - **towards zero:** add the compensation term carrying the sign of the normalised value, then
     truncate towards zero.

   Any other method is a programming error and must fail with a message naming the unknown method.

7. **Denormalise.** Apply the denormalise operation to the whole-number result, and that is the
   answer.

**"Round half away from zero to a whole number"** inside step 6 is **not** the round-half-to-even
that most language runtimes provide as their default. It is defined as:

1. Take the runtime's nearest-whole-number result for the value.
2. Take the runtime's nearest-whole-number result for the value plus one.
3. When the second minus the first is **not exactly one**, the value was a tie that the runtime
   resolved to even; the answer is instead the value plus one half carrying the sign of the value.
4. Otherwise the answer is the result of step 1, carrying the sign of the value.

Carrying the sign exists for two reasons: so that rounding a negative zero yields a negative zero
rather than a positive one, and so that the result is a real number rather than a whole number,
which keeps the later multiplication in the real domain.

### 3.2 The accurate reciprocal

Inverting a rounding factor must not itself introduce error. A lookup table holds the exact
reciprocals of the thirty factors most commonly met:

| Factor | Reciprocal | Factor | Reciprocal | Factor | Reciprocal |
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

Three of the four rounding factors in use by shipped currencies are in the table: one hundredth,
one thousandth and one ten-thousandth. The fourth, one, never reaches the inverter because it is
not smaller than one.

For a factor not in the table, the reciprocal is computed as follows: render the factor in
scientific notation with fifteen digits after the point, splitting it into a coefficient and an
exponent; build the number with the same coefficient and the negated exponent; divide that by the
square of the coefficient.

```formula
inverted factor = ( coefficient × 10 ^ ( − exponent ) ) ÷ coefficient²
```

For the common factors this produces exactly one hundred, one thousand and ten thousand.

### 3.3 Mandatory worked example — rounding two point six seven five onto one hundredth

This is the canonical case and a rebuild must reproduce it exactly.

**Given** the value two point six seven five and a currency whose rounding factor is one
hundredth.

1. The factor one hundredth is strictly smaller than one, so it is inverted to exactly one
   hundred, and the two operations are swapped: normalising **multiplies** by one hundred.
2. Normalised value: two point six seven five times one hundred. In binary double precision the
   literal two point six seven five is stored as two point six seven four nine nine nine nine nine
   nine nine nine nine nine nine eight; multiplied by one hundred this is two hundred sixty-seven
   point four nine nine nine nine nine nine nine nine nine nine nine nine four.
3. Magnitude: the base-two logarithm of two hundred sixty-seven point five is about eight point
   zero six three seven. The compensation term is therefore two raised to the power of minus
   forty-one point nine three six three, which is about three point four times ten to the minus
   thirteen.
4. Method half away from zero: add the compensation term, giving two hundred sixty-seven point
   five zero zero zero zero zero zero zero zero zero zero zero two eight, which is now strictly
   above the tie.
5. Round half away from zero to a whole number: two hundred sixty-eight.
6. Denormalise: divide by one hundred. **Result: two point six eight.**

**Without** the compensation term, step 4 would leave two hundred sixty-seven point four nine nine
nine…, step 5 would give two hundred sixty-seven, and the result would be two point six seven —
one cent lower. Every downstream total would then be one cent out. This is why the term is not
optional.

### 3.4 Further worked examples of the rounding routine

| Value | Rounding factor | Method | Result | Why |
|---|---|---|---|---|
| 2.675 | 0.01 | half away from zero | 2.68 | Section 3.3. |
| −2.675 | 0.01 | half away from zero | −2.68 | The compensation term carries the sign; ties go away from zero in both directions. |
| 2.675 | 0.01 | half towards zero | 2.67 | The term is subtracted, pushing the value below the tie. |
| 2.675 | 0.01 | half to even | 2.68 | The tie is detected; the floor two hundred sixty-seven is odd, so one is added. |
| 2.665 | 0.01 | half to even | 2.66 | The tie is detected; the floor two hundred sixty-six is even, so it stands. |
| 2.671 | 0.01 | away from zero | 2.68 | Any non-zero fraction of a step pushes away from zero. |
| 2.679 | 0.01 | towards zero | 2.67 | The fraction of a step is discarded. |
| 1.435 | 0.01 | half away from zero | 1.44 | The scaled value one hundred forty-three point five becomes one hundred forty-three point five zero zero zero zero zero zero zero zero zero zero zero zero nine; the nearest whole number is one hundred forty-four. |
| 1.005 | 0.01 | half away from zero | 1.01 | The same mechanism. |
| 1.33 | 0.05 | half away from zero | 1.35 | The reciprocal is twenty; the scaled value is twenty-six point six; the nearest whole number is twenty-seven; twenty-seven divided by twenty is one point three five. |
| 1.32 | 0.05 | half away from zero | 1.30 | Scaled value twenty-six point four; nearest whole number twenty-six; result one point three zero. |
| 1234.5 | 1 | half away from zero | 1235 | A currency with no subunit. The factor is not smaller than one, so normalising divides by one and the tie is broken away from zero. |
| 1234.4 | 1 | half away from zero | 1234 | |
| 1234.49 | 1 | half away from zero | 1234 | |
| 1234.56 | 1 | half away from zero | 1235 | |
| 0.12345 | 0.001 | half away from zero | 0.123 | A three-decimal currency. |
| 0.12355 | 0.001 | half away from zero | 0.124 | |
| 0.123456 | 0.0001 | half away from zero | 0.1235 | A four-decimal currency. |
| 1234.56 | 5 | half away from zero | 1235 | A hypothetical currency rounding onto multiples of five. Normalising divides by five giving two hundred forty-six point nine one two; the nearest whole number is two hundred forty-seven; denormalising multiplies by five. |
| −0.004 | 0.01 | half away from zero | −0.0 | The result is a negative zero; the presentation routine strips the sign (section 24.3). |

---

## 4. The three-way comparison

**Inputs:** two amounts and a precision.
**Output:** minus one, zero or plus one.
**Meaning:** two amounts are equal when their *rounded* values are equal. This is deliberately not
the same as their difference being negligible.

```formula
compare( first amount , second amount ) =
     0   when the two amounts are exactly equal
     0   when round( first amount ) − round( second amount ) passes the zero test at the same factor
    −1   when round( first amount ) − round( second amount ) is negative
    +1   otherwise
```

Algorithm:

1. Resolve the factor exactly as in section 3.1 step 1.
2. When the two amounts are bit-for-bit equal, the answer is zero at once. The short-circuit is
   taken *after* the precision has been validated, so that a bad precision still fails loudly.
3. Round each amount onto the factor, using half away from zero.
4. Take the difference of the two rounded amounts.
5. When the difference passes the zero test of section 5 at the same factor, the answer is zero.
6. Otherwise the answer is minus one when the difference is negative and plus one when it is
   positive.

### 4.1 Mandatory worked example — comparison at a rounding factor of one hundredth

| First amount | Second amount | First rounded | Second rounded | Difference | Zero? | Result | Reading |
|---|---|---|---|---|---|---|---|
| 1.432 | 1.431 | 1.43 | 1.43 | 0.00 | yes | **0** | Equal at one hundredth, even though the raw values differ by one thousandth. |
| 0.006 | 0.002 | 0.01 | 0.00 | 0.01 | no | **+1** | Different at one hundredth, even though the raw values differ by only four thousandths — less than the precision. This is the case that catches every rebuild that subtracts before rounding. |
| 0.002 | 0.006 | 0.00 | 0.01 | −0.01 | no | **−1** | The mirror of the previous row. |
| 1.005 | 1.00 | 1.01 | 1.00 | 0.01 | no | **+1** | The tie in the first amount is resolved away from zero by the compensation term. |
| 1.004 | 1.00 | 1.00 | 1.00 | 0.00 | yes | **0** | |
| 100.004 | 100.0 | 100.00 | 100.00 | 0.00 | yes | **0** | |
| 100.00 | 100.00 | — | — | — | — | **0** | Taken by the exact-equality short-circuit of step 2 without any rounding. |
| −0.006 | 0.002 | −0.01 | 0.00 | −0.01 | no | **−1** | |
| −0.006 | −0.002 | −0.01 | −0.00 | −0.01 | no | **−1** | |
| 1052.63 | 1086.96 | 1052.63 | 1086.96 | −34.33 | no | **−1** | |
| 2.675 | 2.68 | 2.68 | 2.68 | 0.00 | yes | **0** | Because the first value rounds up, as section 3.3 established. |

Ordering consequence: the comparison is a genuine three-way ordering on rounded values. A rebuild
must **not** implement it as "the sign of the difference", and must **not** implement it as
"compare after subtracting". Round first, subtract second.

---

## 5. The zero test

An amount is treated as zero when it is exactly zero, or when, after being rounded onto the
currency's factor, its absolute value is strictly smaller than the factor.

```formula
is zero( amount ) = ( amount = 0 )  OR  ( | round( amount , rounding factor ) | < rounding factor )
```

Worked values at a rounding factor of one hundredth:

| Amount | Rounded | Absolute value | Strictly below 0.01? | Treated as zero |
|---|---|---|---|---|
| 0 | not evaluated | not evaluated | not evaluated | yes, by the short-circuit |
| 0.004 | 0.00 | 0.00 | yes | yes |
| 0.0049999 | 0.00 | 0.00 | yes | yes |
| 0.005 | 0.01 | 0.01 | no | no |
| 0.006 | 0.01 | 0.01 | no | no |
| −0.004 | −0.00 | 0.00 | yes | yes |
| 0.01 | 0.01 | 0.01 | no | no |

### 5.1 The asymmetry that matters

Testing the difference of two amounts for zero is **not** the same as comparing them. The zero
test rounds *after* subtracting; the comparison rounds *before* subtracting. With six thousandths,
two thousandths and a rounding factor of one hundredth:

```formula
is zero( 0.006 − 0.002 ) = is zero( 0.004 ) = true
compare( 0.006 , 0.002 ) = compare( 0.01 , 0.00 ) = +1
```

Both behaviours are intentional and both are used in this domain: the zero test decides whether a
residual is materially exhausted, the comparison decides which of two items fully covers the
other.

---

## 6. Exact euclidean division

Some flows need to split an amount into whole multiples of another amount without the
representation error that a native division and remainder would carry. The routine returns a
whole-number quotient and a remainder.

Algorithm:

1. Resolve the factor as in section 3.1 step 1.
2. Round the dividend onto the factor, divide by the factor, and round that to the nearest whole
   number. Call the result the scaled dividend.
3. Do the same to the divisor. Call the result the scaled divisor.
4. Take the whole-number quotient and the remainder of the scaled dividend by the scaled divisor.
5. Multiply the remainder by the factor and round the product onto the factor.
6. The answer is that quotient, a whole number, and that remainder.

```formula
scaled dividend = nearest whole number( round( dividend , factor ) ÷ factor )
scaled divisor  = nearest whole number( round( divisor  , factor ) ÷ factor )
quotient        = whole part( scaled dividend ÷ scaled divisor )
remainder       = round( ( scaled dividend − quotient × scaled divisor ) × factor , factor )
```

Postcondition: the dividend, rounded onto the factor, equals the quotient times the divisor plus
the remainder, exactly, at that factor.

**Worked example.** Dividend two hundred sixty-eight point five zero, divisor twelve point two
five, factor one hundredth. The scaled dividend is twenty-six thousand eight hundred fifty; the
scaled divisor is one thousand two hundred twenty-five. The quotient is twenty-one; twenty-one
times one thousand two hundred twenty-five is twenty-five thousand seven hundred twenty-five; the
remainder is one thousand one hundred twenty-five, which denormalises to eleven point two five.
Check: twenty-one times twelve point two five is two hundred fifty-seven point two five, plus
eleven point two five is two hundred sixty-eight point five zero. Correct.

---

## 7. The rate lookup

### 7.1 Statement

Given a set of currencies, a company and a date, produce for each currency a single number: its
technical rate in force. The result carries full precision; no rounding is ever applied to a rate.

### 7.2 Algorithm

For each currency:

1. **Resolve the company.** Take the **root** company of the company supplied. Rates never live on
   a branch.
2. **Primary lookup.** Among the rate rows of this currency whose rate date is **on or before** the
   requested date and whose company is either the root company or empty, order by company first —
   rows that name the company sort before rows that name no company — and then by rate date
   **descending**. Take the first row's technical rate.
3. **Fallback lookup.** When there is no such row, consider the rate rows of this currency whose
   company is either the root company or empty, **without any date restriction**. Order by company
   first, then by rate date **ascending**. Take the first row's technical rate. When every rate of
   a currency is later than the requested date, the earliest one is therefore used rather than
   nothing.
4. **Last resort.** When the currency has no rate row at all, the rate is **one**.

The company preference in steps 2 and 3 is total: *any* company-specific row that satisfies the
date condition beats *every* shared row, even a shared row with a later date. A rebuild must not
merge the two sets and then order by date.

The lookup is evaluated for several currencies at once when a conversion is requested, and its
result is cached; the cache is invalidated whenever any rate row is created or modified.

### 7.3 Worked example — a step function of time

A company whose main currency is `USD` (the United States dollar) records the following rows for
`EUR` (the euro), all scoped to the root company. No rate row exists for `USD`.

| Rate date | Technical rate |
|---|---|
| 2024-01-01 | 0.9000 |
| 2024-07-01 | 0.9250 |
| 2025-01-01 | 0.9500 |

| Requested date | Rows on or before the date | Winner | Rate of `EUR` | Reason |
|---|---|---|---|---|
| 2023-12-31 | none | fallback: the earliest row, 2024-01-01 | 0.9000 | Before the first rate, the first rate is projected backwards. |
| 2024-01-01 | 2024-01-01 | 2024-01-01 | 0.9000 | The boundary date is included. |
| 2024-06-30 | 2024-01-01 | 2024-01-01 | 0.9000 | The row of 2024-07-01 is not yet in force. |
| 2024-07-01 | 2024-01-01, 2024-07-01 | 2024-07-01 | 0.9250 | The latest wins. |
| 2024-12-31 | 2024-01-01, 2024-07-01 | 2024-07-01 | 0.9250 | |
| 2025-01-01 | all three | 2025-01-01 | 0.9500 | |
| 2030-05-05 | all three | 2025-01-01 | 0.9500 | The last rate stays in force indefinitely. |

The rate of `USD` is one for every one of those dates, by the last-resort rule, because the dollar
carries no rate row.

### 7.4 Worked example — company scope beats date

Rate rows for `USD` in a platform with a root company called Northwind and another root company
called Southgate:

| Row | Rate date | Company | Technical rate |
|---|---|---|---|
| A | 2026-01-01 | shared | 1.0500 |
| B | 2026-03-01 | shared | 1.1000 |
| C | 2026-02-15 | Northwind | 1.0800 |
| D | 2026-06-01 | Northwind | 1.2000 |

| Requested date | Requested company | Rows passing the date filter | Winner | Rate |
|---|---|---|---|---|
| 2026-02-20 | Northwind | A, C | C — it names the company | 1.0800 |
| 2026-02-20 | Southgate | A | A | 1.0500 |
| 2026-03-15 | Northwind | A, B, C | C — a company-specific row beats the later shared row B | 1.0800 |
| 2026-03-15 | Southgate | A, B | B — the later of the two shared rows | 1.1000 |
| 2026-07-01 | Northwind | A, B, C, D | D | 1.2000 |
| 2025-12-01 | Northwind | none | fallback: the earliest among A, C, D preferring the company-specific row → C | 1.0800 |
| 2025-12-01 | Southgate | none | fallback: the earliest shared row → A | 1.0500 |

Row three of the table is the one that surprises people: in March, Northwind still uses its own
February rate, because a company-specific row is preferred unconditionally.

A second illustration with a single company: a currency has a shared row dated 2025-01-01 with
rate zero point nine five and a row scoped to the root company dated 2024-01-01 with rate zero
point nine three. A lookup at 2025-06-01 finds both in force and answers zero point nine three.

### 7.5 The derived current rate of a currency

The current rate exposed on a currency is computed against a **target** currency, which defaults to
the main currency of the company in context:

```formula
current rate  = ( rate of this currency  or  1 ) ÷ rate of the target currency
inverse rate  = 1 ÷ current rate
```

Both rates come from the lookup of section 7.2 at the effective date. Four context values steer
the computation: the target currency, the date, the company and the company identifier. The date
defaults to today in the reader's time zone.

The human-readable form is built as:

```formula
rate as text = "1 " + target currency code + " = " + current rate rendered with exactly six decimal digits + " " + this currency code
```

and is **empty** when this currency is the main currency of the company in context.

---

## 8. Conversion between two currencies

### 8.1 The conversion factor

```formula
conversion factor( from currency , to currency , company , date ) = 1
        when the two currencies are the same

conversion factor( from currency , to currency , company , date )
      = rate of( to currency , root company , date ) ÷ rate of( from currency , root company , date )
        otherwise
```

where *rate of* is the lookup of section 7.2. The date defaults to today in the reader's time
zone; the company defaults to the company the reader is working in, and is immediately replaced by
its root.

Direction check, worth stating because it is the most commonly inverted thing in a rebuild: the
conversion factor **from the company currency to a foreign currency** is the foreign currency's
rate divided by the company currency's rate. When the company currency has no rate rows — the
ordinary case — its rate is one, and the factor is simply the foreign currency's technical rate.
So a technical rate of one point ten on `USD` in a company reporting in `EUR` means **one euro buys
one point ten United States dollars**.

Stated as the two directions a document needs:

- The rate of the document currency divided by the rate of the company currency is **the rate from
  the company currency to the document currency**: how many units of the document currency one
  unit of the company currency buys. This is the quantity stored on a document as its document
  rate and derived on a journal item as its item rate.
- The rate of the company currency divided by the rate of the document currency is **the rate from
  the document currency to the company currency**, the reciprocal of the previous one, which is
  what the inverse rate of a currency returns.

### 8.2 The conversion of an amount

```formula
converted amount = round onto the to currency ( amount × conversion factor( from currency , to currency , company , date ) )
```

Algorithm:

1. When the source currency is empty, substitute the destination currency; when the destination
   currency is empty, substitute the source. When both are empty, fail — an amount cannot be
   converted from an unknown currency.
2. When the amount is exactly zero, the answer is zero, without consulting any rate.
3. When the two currencies are the same, the factor is exactly one, without any lookup.
4. Multiply the amount by the conversion factor.
5. Round the product onto the **destination** currency's rounding factor, using half away from
   zero — unless the caller explicitly asked for an unrounded result, in which case the raw
   product is the answer.

**One multiplication, one rounding.** Even when neither currency is the company currency, the
conversion performs a single multiplication by the composed cross-rate and rounds once. It does
**not** convert into the company currency, round, and convert again.

### 8.3 Mandatory worked example — conversion between two foreign currencies

**Given** a company reporting in `EUR`. On 15 March 2026 the technical rates in force for that
company are:

| Currency | Technical rate |
|---|---|
| `EUR` | 1.0000 — no rate rows exist, so the last resort of one applies |
| `USD` | 1.1723 |
| `GBP` (pound sterling) | 0.8391 |

**When** five hundred United States dollars are converted into pounds sterling on that date.

Step 1 — the cross-rate is composed in a single division:

```formula
conversion factor( USD , GBP ) = 0.8391 ÷ 1.1723 = 0.715772413204811…
```

Step 2 — one multiplication:

```formula
raw product = 500.00 × 0.715772413204811… = 357.886206602405…
```

Step 3 — one rounding, onto the destination currency's factor of one hundredth:

**Result: three hundred fifty-seven pounds and eighty-nine pence.**

**Now the wrong way, for contrast.** A rebuild that routes through the company currency in two
rounded steps gets a different answer:

```formula
step one = round onto EUR( 500.00 × ( 1 ÷ 1.1723 ) ) = round( 426.511984… ) = 426.51
step two = round onto GBP( 426.51 × 0.8391 )         = round( 357.884… )   = 357.88
```

Three hundred fifty-seven pounds and eighty-eight pence — one penny short. The single-step
composition is the specified behaviour; the two-step route is not. The divergence is not rare: at
these rates it appears for roughly one amount in five.

| Amount in `USD` | One step, specified | Two steps, wrong |
|---|---|---|
| 100.00 | 71.58 | 71.58 — agree |
| 250.55 | 179.34 | 179.34 — agree |
| **500.00** | **357.89** | 357.88 — **differ** |
| 777.77 | 556.71 | 556.71 — agree |
| 1 234.56 | 883.66 | 883.66 — agree |
| **4 321.09** | **3 092.92** | 3 092.91 — **differ** |

### 8.4 Worked example — the two directions at a rate of one point zero eight five zero

A company's main currency is `EUR`, which carries no rate row. `USD` has a rate row dated
2026-02-01 with a technical rate of one point zero eight five zero, meaning one euro is worth one
dollar and eight and a half cents. Both currencies have a rounding factor of one hundredth.

Company currency into foreign currency, on 2026-02-10:

```formula
conversion factor = 1.0850 ÷ 1.0 = 1.0850
raw product       = 1 000.00 × 1.0850 = 1 085.00
result            = 1 085.00
```

Foreign currency into company currency, on the same date:

```formula
conversion factor = 1.0 ÷ 1.0850 = 0.9216589861751152
raw product       = 1 000.00 × 0.9216589861751152 = 921.6589861751152
normalised        = 921.6589861751152 × 100 = 92 165.89861751152
compensation term = 2 ^ ( log2( 92 165.9 ) − 50 ) = 2 ^ ( 16.4918 − 50 ) = 7.9 × 10 ^ −11
normalised + term = 92 165.89861751160
nearest whole     = 92 166
result            = 92 166 ÷ 100 = 921.66
```

A currency with no fractional part. `JPY` (the Japanese yen) has a rate row of one hundred
sixty-five point two zero on the same date and a rounding factor of one:

```formula
1 000.00 EUR into JPY : 1 000.00 × ( 165.20 ÷ 1.0 ) = 165 200.00 → 165 200
1 000 JPY into EUR    : 1 000 × ( 1.0 ÷ 165.20 ) = 6.053268765133172 → 6.05
```

### 8.5 Conversions that round to zero, and round trips

Because the result is rounded onto the destination factor, a small amount in a fine-grained
currency can vanish when converted into a coarse one.

| Amount | From | To | Factor used | Raw product | Rounded | Note |
|---|---|---|---|---|---|---|
| 0.004 | `EUR`, factor one hundredth | `USD`, factor one hundredth | 1.1723 | 0.00468… | 0.00 | Vanishes. |
| 0.40 | `EUR` | `JPY`, factor one | 163.5 | 65.4 | 65 | Does not vanish, but loses the fraction. |
| 0.002 | `EUR` | `JPY`, factor one | 163.5 | 0.327 | 0 | Vanishes. |
| 1.00 | `JPY`, factor one | `EUR` | 0.006116 | 0.006116 | 0.01 | Rounds **up** from almost nothing, because half away from zero applied to zero point six one one six of a step gives one step. |

Converting an amount into another currency and back does not in general return the starting
amount, because each leg rounds. Converting one hundred euro into dollars at one point zero eight
five zero gives one hundred eight point five zero, and converting that back gives exactly one
hundred; converting one hundred euro into yen at one hundred sixty-five point two zero gives
sixteen thousand five hundred twenty, and converting that back gives exactly one hundred. But
converting seven hundredths of a euro into yen gives round( 11.564 ) = 12, and converting twelve
yen back gives round( 0.0726… ) = 0.07, while converting eight hundredths gives round( 13.216 ) =
13 and back gives 0.08. A replacement must never rely on a round trip being the identity, and must
never re-derive a stored foreign amount by converting a stored company-currency amount.

---

## 9. The three rate representations of a rate row

**Inputs:** the technical rate of the row, the rate of the company's main currency in force at the
same moment, and the company scope.
**Outputs:** the company rate and the inverse company rate.

Let *the company's own rate* be, for a company: among the rate rows of that company's main currency
whose technical rate is non-zero and whose company is that company or is empty, the technical rate
of the row with the greatest rate date; or one when there is no such row.

```formula
company rate          = ( technical rate  or  carried-forward rate  or  1 ) ÷ the company's own rate
inverse company rate  = 1 ÷ company rate
technical rate        = company rate × the company's own rate          when the company rate is written
company rate          = 1 ÷ inverse company rate                       when the inverse company rate is written
```

with the guards of [business-rules.md](business-rules.md) `MCUR-035`: a company rate that is zero
or unset is forced to one before its reciprocal is taken, and an inverse company rate that is zero
or unset is forced to one before its reciprocal is taken. The company used is the row's own company
when it has one, otherwise the root company of the company the reader is working in.

The *carried-forward rate* is the technical rate of the latest row of the same currency and the
same company scope whose rate date is strictly earlier than this row's, and one when there is none.

### 9.1 Worked example — a platform whose main currency carries no rate

A company's main currency is `USD`, which carries no rate row, so the company's own rate is one. An
accountant creates a rate row for `EUR` dated 2026-02-01 and types zero point nine two zero zero
into the column headed `EUR` per `USD`, which is the company rate.

```formula
technical rate       = 0.9200 × 1.0 = 0.9200
inverse company rate = 1 ÷ 0.9200 = 1.0869565217391304
```

The list then shows the company rate as 0.920000000000 and the inverse company rate as
1.086956521739, each displayed with twelve fractional digits.

### 9.2 Worked example — a platform whose main currency carries a rate

The same platform also keeps rate rows for the dollar itself, because the reference of rate one is
a third unit of account. The dollar carries a row dated 2026-01-15 with a technical rate of one
point two five zero zero, so the company's own rate is one point two five zero zero. The
accountant again types zero point nine two zero zero into `EUR` per `USD`:

```formula
technical rate = 0.9200 × 1.2500 = 1.1500
```

The stored technical rate is one point one five zero zero, and a conversion from dollars into euro
still yields 1.1500 ÷ 1.2500 = 0.9200 as intended. This is why the technical rate and the rate an
accountant reads are separate fields.

### 9.3 The plausibility warning

When the company rate is edited in a form, let the previous technical rate be the technical rate
of the latest strictly earlier row of the same currency and the same company scope. When such a row
exists:

```formula
relative movement = ( previous technical rate − new technical rate ) ÷ previous technical rate
warn when | relative movement | > 0.2
```

where the new technical rate is the one implied by the entered company rate.

**Worked example.** The previous technical rate is zero point nine two zero zero. The accountant
enters a company rate of zero point seven zero zero zero in a platform whose company's own rate is
one, giving a new technical rate of zero point seven zero zero zero. The relative movement is
( 0.9200 − 0.7000 ) ÷ 0.9200 = 0.2391, whose absolute value exceeds zero point two, so the warning
of `MCUR-021` is shown. Entering zero point seven five zero zero instead gives ( 0.9200 − 0.7500 )
÷ 0.9200 = 0.1848, which does not exceed zero point two, and no warning appears.

---

## 10. The document rate

**Inputs:** the document currency, the company currency, the company and the document's rate date.
**Output:** the document rate — how many units of the document currency correspond to one unit of
the company currency.

### 10.1 The rate date of a document

```formula
document rate date = the invoice date            when one is set
document rate date = today in the reader's time zone   otherwise
```

### 10.2 The expected rate and the applied rate

```formula
expected rate = rate of( document currency , company , document rate date ) ÷ rate of( company currency , company , document rate date )
expected rate = 1   when the document has no currency
```

The applied rate, stored on the document, is set to the expected rate whenever the document
currency, the company currency, the company, the invoice date or the taxable supply date changes,
and only for an invoice, a bill, a credit note, a debit note or a receipt. A user may then
overwrite it; the overwritten value is what every line balance is derived from. The rate refresh
operation resets the applied rate to the expected rate.

### 10.3 Deriving a line balance from a line amount

For every line of such a document:

```formula
balance = round onto the company currency( amount in currency ÷ document rate )
```

and conversely, when a balance is known and an amount in currency is not:

```formula
amount in currency = round onto the document currency( balance × item rate )
```

where the item rate equals the document's applied rate, defaulting to one when that is zero.

### 10.4 Worked example — a manually fixed rate

A company's main currency is the United States dollar. A customer invoice is issued in a foreign
currency whose expected rate on the invoice date is two point zero. The accountant overrides the
applied rate to five point zero because the sales contract fixes it. The invoice carries one
product line of two thousand in the document currency and one tax line at fifteen percent.

| Line | Amount in currency | Computation | Balance |
|---|---|---|---|
| Product | −2 000.00 | −2 000.00 ÷ 5.0 | −400.00 |
| Tax at fifteen percent | −300.00 | −300.00 ÷ 5.0 | −60.00 |
| Receivable | +2 300.00 | +2 300.00 ÷ 5.0 | +460.00 |

The entry balances in both columns: the document currency column sums to zero, and the company
currency column sums to zero. Posting the document changes neither value, because the rate was
manually fixed (`MCUR-082`).

### 10.5 Worked example — changing the invoice date changes every balance

A company's main currency is the United States dollar. A foreign currency carries rates of zero
point five from 2025-01-01 and zero point four from 2025-02-01. A customer invoice dated
2025-01-01 has one product line of one thousand in the document currency with a fifteen percent
tax. At the invoice date 2025-01-01 the applied rate is zero point five:

| Line | Amount in currency | Balance |
|---|---|---|
| Product | −1 000.00 | −2 000.00 |
| Tax | −150.00 | −300.00 |
| Receivable | +1 150.00 | +2 300.00 |

The user changes the invoice date to 2025-02-01. The applied rate becomes zero point four and every
balance is re-derived while the amounts in the document currency are preserved:

| Line | Amount in currency | Balance |
|---|---|---|
| Product | −1 000.00 | −2 500.00 |
| Tax | −150.00 | −375.00 |
| Receivable | +1 150.00 | +2 875.00 |

Changing the applied rate never changes an amount in the document currency; it only re-derives the
company currency column. This is why the operation is a reapplication of the rate and not a
recomputation of the document.

### 10.6 Worked example — a fractional rate under per-tax rounding

A company whose tax rounding method is round per tax issues an invoice in a foreign currency with a
manually applied rate of one divided by one thousand one hundred eighty-nine point five. The single
line has a quantity of zero point eight zero and a unit price of eight hundred ninety-four point
three four, giving seven hundred fifteen point four seven two in the document currency.

```formula
balance = round onto the company currency( −715.472 ÷ ( 1 ÷ 1 189.5 ) )
        = round( −715.472 × 1 189.5 )
        = round( −851 053.9440 )
        = −851 053.94
```

The counterpart receivable line carries plus eight hundred fifty-one thousand fifty-three point
nine four. Because the rounding is applied once to the whole tax group rather than to each line,
the two columns balance exactly.

### 10.7 The rate on a line of an entry that is not a document

For a miscellaneous journal entry, a payment entry or a bank transaction entry there is no stored
document rate. The item rate is computed directly:

```formula
item rate       = rate of( item currency , company , effective date ) ÷ rate of( company currency , company , effective date )
effective date  = the entry's invoice date      when set
effective date  = the entry's accounting date   when set and no invoice date
effective date  = today                         otherwise
item rate       = 1                             when the item has no currency
```

### 10.8 The implied rate carried by a posted journal item

Once a journal item is written, the rate at which it was recorded is no longer read from the rate
table; it is read back from the item itself:

```formula
implied rate = | amount in currency ÷ balance |      when neither amount is zero at its own precision
implied rate = undefined                             otherwise
```

This is the quantity the reconciliation algorithm calls the accounting rate. It is expressed in
units of the document currency per one unit of the company currency, the same direction as the
document rate.

**Worked example.** An invoice line carries an amount in currency of one thousand and a balance of
one thousand eighty-six point nine six. Its implied rate is 1 000.00 ÷ 1 086.96 = 0.9200007359…,
which is the rate zero point nine two zero zero as recovered from two rounded amounts. The
recovered rate is not exactly the original rate, because both amounts were rounded; the
reconciliation algorithm is built to tolerate exactly that discrepancy (section 13.4).

---

## 11. The two amounts of a journal item

### 11.1 The binding formulas

```formula
amount in currency = round onto the item currency   ( balance            × item rate )
balance            = round onto the company currency( amount in currency ÷ item rate )
```

The item rate is the conversion factor **from the company currency to the item currency**. It is
derived as:

1. When the item's entry is invoice-like — a customer invoice, a credit note, a vendor bill, a
   refund or a receipt — the item rate is the entry's stored document rate; when that is zero, it
   is one.
2. Otherwise, when the item has a currency, the item rate is the conversion factor from the
   company currency to that currency, for the item's company, at the effective date of section
   10.7.
3. Otherwise the item rate is one.

### 11.2 Which of the two is derived

| Caller supplied | Derived |
|---|---|
| The balance only | The amount in currency, by the first formula. |
| The amount in currency only | The balance, by the second formula. |
| Both | Both are kept as supplied, subject to the two forcings below. |
| Neither | Both remain zero, subject to the two forcings below. |

Two forcings override the table:

- When the item currency equals the company currency **and** the entry is **not** invoice-like,
  the amount in currency is forced equal to the balance. There is then no rate arithmetic at all.
- On an invoice-like entry, a change to the amount in currency, to the item rate or to the document
  type re-derives the balance by the second formula, even when a balance was supplied.

### 11.3 Worked example — an invoice line in a foreign currency

**Given** a company reporting in `EUR`, a customer invoice in `USD` dated 15 March 2026, and a
stored document rate of one point ten, meaning one euro buys one point ten United States dollars.
A product line has a subtotal of one thousand one hundred United States dollars.

```formula
balance = round onto EUR( 1 100.00 ÷ 1.10 ) = round( 1 000.000000 ) = 1 000.00
```

The revenue line is therefore a credit of one thousand euro with an amount in currency of minus one
thousand one hundred United States dollars, and the receivable line a debit of one thousand euro
with an amount in currency of plus one thousand one hundred United States dollars.

At a document rate of one point one seven two three instead:

```formula
balance = round onto EUR( 1 100.00 ÷ 1.1723 ) = round( 938.326367… ) = 938.33
```

### 11.4 The sign invariant as arithmetic

```formula
( balance ≤ 0  AND  amount in currency ≤ 0 )   OR   ( balance ≥ 0  AND  amount in currency ≥ 0 )
```

must hold for every item that is not a section, a subsection or a note. Because the item rate is
always strictly positive, the derivation formulas preserve the sign automatically; the invariant
bites only when both amounts are written independently.

---

## 12. The residuals a journal item offers, per currency

Before two journal items can be matched, the algorithm asks each of them: in which currencies can
you offer a residual, how much, and at what rate. The answer is a map from currency to a pair — a
residual and a rate.

**Inputs:** the item, its current residual amount and residual amount in currency as they stand at
this point in the matching loop (they are decremented as the loop proceeds, not re-read from
storage), the currency of the item it is about to be matched against — the counterpart currency —
and the counterpart item itself.

### 12.1 Algorithm

1. Let the company currency, the item currency and the account be those of the item.
2. Let *zero in the company currency* be true when the residual amount is zero at the company
   currency's precision, and *zero in the item currency* be true when the residual amount in
   currency is zero at the item currency's precision.
3. Start with an empty map.
4. When *zero in the company currency* is false, add an entry for the **company currency** with
   that residual and a rate of one.
5. When the item currency differs from the company currency and *zero in the item currency* is
   false, add an entry for the **item currency** with that residual and the item's **accounting
   rate**, which is the implied rate of section 10.8.
6. **Mirroring a company-currency item into the counterpart currency.** When all of the following
   hold — the item currency equals the company currency; the account is a receivable or a payable
   account; *zero in the company currency* is false; and the counterpart currency differs from the
   company currency — compute a mirror rate by the rule of section 12.2 and add an entry for the
   **counterpart currency** whose residual is the residual amount multiplied by that mirror rate
   and rounded onto the counterpart currency, and whose rate is that mirror rate; unless that
   rounded residual is zero at the counterpart currency's precision, in which case no entry is
   added. This step is what allows an invoice issued in the company currency to be settled by a
   payment made in a foreign currency.
7. **Offering a foreign residual to a counterpart in the same foreign currency.** Otherwise, when
   the item currency equals the counterpart currency, differs from the company currency, and *zero
   in the item currency* is false, add or overwrite the entry for the counterpart currency with
   that residual and the item's accounting rate.
8. The map is the answer.

### 12.2 The mirror rate

The mirror rate used in step 6 is chosen by the first matching clause:

1. A rate forced by the payment registration flow, when the operation context carries one.
2. When the counterpart item originates from a payment or from a bank transaction and this item
   does not, the counterpart item's **accounting rate** in the counterpart currency. **The rate of
   the payment always wins**, which is what makes the settlement value of an invoice equal to the
   value the bank actually moved.
3. Otherwise the conversion factor from the company currency to the counterpart currency, for the
   item's company, evaluated at the item's own rate date: the invoice date when the item belongs to
   an invoice-like entry, and the item's accounting date otherwise.

---

## 13. Computing one partial matching

**Inputs:** the debit item with its running residuals in both currencies, the credit item with its
running residuals in both currencies, and the two maps of section 12.
**Outputs:** a matched amount in the company currency, a matched amount in the debit item's
currency, a matched amount in the credit item's currency, and, when needed, an exchange difference
instruction for one or both items.

### 13.1 Choosing the currency the matching is measured in

1. When the debit currency is not the company currency **and** appears in both maps, the
   reconciliation currency is the debit currency.
2. Otherwise, when the credit currency is not the company currency **and** appears in both maps,
   the reconciliation currency is the credit currency.
3. Otherwise the reconciliation currency is the **company currency**.

When either map lacks the chosen currency, the pairing is abandoned: the side that lacks it is
marked as having nothing left, and the loop advances to the next item on that side.

### 13.2 Preliminary quantities and the exchange-line mode

```formula
debit recon amount    =   the residual the debit item offers in the reconciliation currency
credit recon amount   = − the residual the credit item offers in the reconciliation currency
comparison            = compare at the reconciliation currency( debit recon amount , credit recon amount )
minimum               = the smaller of the two recon amounts
debit fully matched   = ( comparison ≤ 0 )
credit fully matched  = ( comparison ≥ 0 )
```

Both recon amounts are positive by construction. When the comparison is zero both flags are true:
the two items cover each other exactly in the reconciliation currency.

A special mode, *exchange-line mode*, is raised when **all three** of the following hold:

- the reconciliation currency is the company currency; **and**
- the debit item and the credit item have the **same** currency; **and**
- at least one of the two maps has **no** entry for that shared currency.

This describes two items sharing a foreign currency where at least one has no residual left in it,
which is exactly the shape of an exchange difference correction line: it carries a company currency
amount and an amount in currency of zero. In this mode both rates are treated as absent, so that
the matched amounts in the document currencies are zero: the correction reduces only the company
currency residual and must not consume any foreign currency residual.

### 13.3 Branch A — the reconciliation currency is the company currency

```formula
debit rate  = the rate the debit map offers for the debit currency,   absent in exchange-line mode
credit rate = the rate the credit map offers for the credit currency, absent in exchange-line mode

matched amount = minimum

matched amount in the debit currency  = the smaller of ( round onto the debit currency ( debit rate  × minimum ) ,   the remaining debit residual in currency )    when the debit rate exists
matched amount in the debit currency  = 0                                                                                                                          when it is absent
matched amount in the credit currency = the smaller of ( round onto the credit currency( credit rate × minimum ) , − the remaining credit residual in currency )   when the credit rate exists
matched amount in the credit currency = 0                                                                                                                          when it is absent
```

### 13.4 Branch B — the reconciliation currency is a foreign currency

Here the minimum is expressed in the foreign currency and must be translated into the company
currency once for each side, using each side's own rate. Because each translated value is itself a
rounded number, it stands for a small interval of possible true values, and the routine computes
that interval explicitly.

```formula
interval( currency from , currency to , amount , rate ) =
    (   round onto currency to( ( amount − rounding factor of currency from ÷ 2 ) × rate ) ,
        round onto currency to(   amount                                          × rate ) ,
        round onto currency to( ( amount + rounding factor of currency from ÷ 2 ) × rate )  )
```

with the convention that an absent rate yields the interval of three zeros. Then:

```formula
debit interval  = interval( debit currency  , company currency , minimum , 1 ÷ debit rate  )
credit interval = interval( credit currency , company currency , minimum , 1 ÷ credit rate )

partial debit amount  = the smaller of ( the middle of the debit interval  ,   the remaining debit residual )
partial credit amount = the smaller of ( the middle of the credit interval , − the remaining credit residual )
matched amount        = the smaller of ( partial debit amount , partial credit amount )
```

**The tolerance band.** When each side's chosen company-currency amount falls inside the *other*
side's interval, the two sides disagree only by a rounding artefact, and forcing an exchange
difference would be wrong. The condition, all four parts of which must hold, each comparison being
the three-way comparison of section 4 at the company currency:

```formula
partial debit amount  ≤ the highest of the credit interval   AND
partial debit amount  ≥ the lowest  of the credit interval   AND
partial credit amount ≤ the highest of the debit interval    AND
partial credit amount ≥ the lowest  of the debit interval
```

When it holds, the three amounts are collapsed onto the smaller of the two outstanding
company-currency residuals:

```formula
matched amount        = the smaller of ( the remaining debit residual , − the remaining credit residual )
partial debit amount  = matched amount
partial credit amount = matched amount
```

and no exchange difference arises from the rounding.

Finally the matched amounts in each side's own currency:

```formula
matched amount in the debit currency  = matched amount   when the debit currency is the company currency,  otherwise the minimum
matched amount in the credit currency = matched amount   when the credit currency is the company currency, otherwise the minimum
```

### 13.5 Updating the running residuals

After the exchange handling of section 14, and regardless of it:

```formula
remaining debit residual              = remaining debit residual              − matched amount
remaining credit residual             = remaining credit residual             + matched amount
remaining debit residual in currency  = remaining debit residual in currency  − matched amount in the debit currency
remaining credit residual in currency = remaining credit residual in currency + matched amount in the credit currency
```

A side is declared finished, and dropped from the matching loop, when **both** of its remaining
amounts pass the zero test at their own currencies.

The matching record produced carries the matched amount, the matched amount in the debit currency
and the matched amount in the credit currency; all three are always positive.

### 13.6 Worked example — two items in the same foreign currency recorded at different rates

A company's main currency is the United States dollar. A foreign currency has a rounding factor of
one thousandth and rates of three point zero from 2016-01-01 and two point zero from 2017-01-01.
Two journal items sit on the same receivable account:

| Item | Date | Balance | Amount in currency | Implied rate |
|---|---|---|---|---|
| Debit item | 2017-01-01 | +60.00 | +120.000 | 120 ÷ 60 = 2.0 |
| Credit item | 2016-01-01 | −80.00 | −240.000 | 240 ÷ 80 = 3.0 |

```formula
debit map  : company currency → ( 60.00 , 1 ) ; foreign currency → ( 120.000 , 2.0 )
credit map : company currency → ( −80.00 , 1 ) ; foreign currency → ( −240.000 , 3.0 )
reconciliation currency = the foreign currency, the debit currency, present in both maps
debit recon amount  = 120.000
credit recon amount = 240.000
comparison          = compare( 120.000 , 240.000 ) = −1   →  the debit side is fully matched, the credit side is not
minimum             = 120.000

debit interval  : rate 1 ÷ 2.0 = 0.5 , half a step = 0.0005
                  lowest  = round( 119.9995 × 0.5 ) = round( 59.99975 ) = 60.00
                  middle  = round( 120.0000 × 0.5 ) = 60.00
                  highest = round( 120.0005 × 0.5 ) = round( 60.00025 ) = 60.00
credit interval : rate 1 ÷ 3.0 = 0.333333… , half a step = 0.0005
                  lowest  = round( 119.9995 × 0.333333… ) = 40.00
                  middle  = round( 120.0000 × 0.333333… ) = 40.00
                  highest = round( 120.0005 × 0.333333… ) = 40.00

partial debit amount  = the smaller of ( 60.00 , 60.00 ) = 60.00
partial credit amount = the smaller of ( 40.00 , 80.00 ) = 40.00
matched amount        = the smaller of ( 60.00 , 40.00 ) = 40.00

tolerance band : compare( 60.00 , the highest of the credit interval = 40.00 ) = +1, which is not ≤ 0  →  no collapse

matched amount in the debit currency  = the minimum = 120.000
matched amount in the credit currency = the minimum = 120.000
```

The matching records a matched amount of forty, a matched amount in the debit currency of one
hundred twenty and a matched amount in the credit currency of one hundred twenty. The exchange
difference of section 14 then produces a second matching of twenty in the company currency with
zero in both document currencies, and the final residuals are: the debit item fully reconciled at
zero and zero, the credit item left at minus forty and minus one hundred twenty.

### 13.7 Worked example — the tolerance band preventing a spurious difference

A company's main currency is the United States dollar. A foreign currency with a rounding factor of
one hundredth has a single rate row of zero point zero five two nine seven two five five four nine
one nine. Two items sit on the same reconcilable account, both dated at that rate's date:

| Item | Balance | Amount in currency | Currency |
|---|---|---|---|
| Debit item | +377 554.00 | +20 000.00 | the foreign currency |
| Credit item | −372 239.38 | −372 239.38 | the company currency |

The credit item is in the company currency, sits on a receivable account and faces a counterpart in
a foreign currency, so step 6 of section 12.1 mirrors it:

```formula
mirror rate       = 0.052972554919          from the rate table; neither item is a payment
mirrored residual = round onto the foreign currency( −372 239.38 × 0.052972554919 ) = round( −19 718.471066 ) = −19 718.47

debit map  : company currency → ( 377 554.00 , 1 ) ; foreign currency → ( 20 000.00 , 20 000 ÷ 377 554 = 0.05297255492 )
credit map : company currency → ( −372 239.38 , 1 ) ; foreign currency → ( −19 718.47 , 0.052972554919 )
reconciliation currency = the foreign currency
debit recon amount  = 20 000.00
credit recon amount = 19 718.47
comparison          = +1   →  the credit side is fully matched, the debit side is not
minimum             = 19 718.47

debit interval  : rate 1 ÷ 0.05297255492 = 18.87770000 , half a step = 0.005
                  lowest = round( 19 718.465 × 18.8777 ) = 372 239.27
                  middle = round( 19 718.470 × 18.8777 ) = 372 239.36
                  highest= round( 19 718.475 × 18.8777 ) = 372 239.46
credit interval : rate 1 ÷ 0.052972554919 = 18.87770207 , half a step = 0.005
                  lowest = 372 239.31 , middle = 372 239.40 , highest = 372 239.50

partial debit amount  = the smaller of ( 372 239.36 , 377 554.00 ) = 372 239.36
partial credit amount = the smaller of ( 372 239.40 , 372 239.38 ) = 372 239.38
matched amount        = the smaller of ( 372 239.36 , 372 239.38 ) = 372 239.36

tolerance band : 372 239.36 ≤ 372 239.50 and 372 239.36 ≥ 372 239.31
                 372 239.38 ≤ 372 239.46 and 372 239.38 ≥ 372 239.27      all four hold
        →  matched amount = the smaller of ( 377 554.00 , 372 239.38 ) = 372 239.38
           partial debit amount = partial credit amount = 372 239.38

matched amount in the debit currency  = the minimum = 19 718.47
matched amount in the credit currency = matched amount = 372 239.38    the credit item is in the company currency
```

Because the tolerance band collapsed the two translations onto a single value, **no exchange
difference is produced**: the debit item is left with five thousand three hundred fourteen point
six two in the company currency and two hundred eighty-one point five three in the foreign
currency, which is the genuine unpaid remainder rather than a rounding artefact. Without the band,
a two-cent difference would be posted for no economic reason.

### 13.8 Worked example — a company-currency item mirrored at a very small rate

A company's main currency is the United States dollar. A foreign currency has a rounding factor of
one thousandth and a single rate of zero point zero zero zero zero one. Two items sit on the same
reconcilable account:

| Item | Balance | Amount in currency | Currency |
|---|---|---|---|
| Credit item | −10.00 | −10.00 | the company currency |
| Debit item | +1 000 000.00 | +100.000 | the foreign currency |

```formula
credit map before mirroring : company currency → ( −10.00 , 1 )
mirrored residual = round onto the foreign currency( −10.00 × 0.00001 ) = round( −0.0001 ) = −0.000
```

The mirrored residual is zero at the foreign currency's precision, so no foreign entry is added and
the reconciliation currency falls back to the company currency. The minimum is ten, the matched
amount is ten, and the matched amounts in the document currencies are:

```formula
debit rate  = the implied rate of the debit item = 100 ÷ 1 000 000 = 0.0001
matched amount in the debit currency  = the smaller of ( round onto the foreign currency( 0.0001 × 10.00 ) , 100.000 ) = the smaller of ( 0.001 , 100.000 ) = 0.001
credit rate = the implied rate of the credit item = 1, because it is in the company currency
matched amount in the credit currency = the smaller of ( round onto the company currency( 1 × 10.00 ) , 10.00 ) = 10.00
```

No exchange difference is produced, and the debit item is left with nine hundred ninety-nine
thousand nine hundred ninety in the company currency and ninety-nine point nine nine nine in the
foreign currency.

---

## 14. The exchange difference amounts

**Inputs:** the results of section 13 before the running residuals are updated.
**Output:** for the debit item, for the credit item, or for both, an instruction naming which
column must be written off and by how much.

The whole computation is skipped when the operation context suppresses exchange differences
(`MCUR-101`).

### 14.1 Branch A — the matching is measured in the company currency

The company currency column is already exactly consumed; what may be left over is a residual in a
document currency, so the repair is expressed in the **amount in currency** column.

```formula
when the debit side is fully matched:
        debit exchange amount = remaining debit residual in currency − matched amount in the debit currency
        and, when that is not zero at the debit currency's precision:
                instruct the debit item, column amount in currency, amount debit exchange amount
                remaining debit residual in currency = remaining debit residual in currency − debit exchange amount

when the credit side is fully matched:
        credit exchange amount = remaining credit residual in currency + matched amount in the credit currency
        and, when that is not zero at the credit currency's precision:
                instruct the credit item, column amount in currency, amount credit exchange amount
                remaining credit residual in currency = remaining credit residual in currency + credit exchange amount
```

### 14.2 Branch B — the matching is measured in a foreign currency, side fully matched

The document currency column is already exactly consumed; what may be left over is a residual in
the company currency, and that residual is precisely the effect of the rate having moved. The
repair is expressed in the **balance** column and clears the whole remaining balance of that side.

```formula
when the debit side is fully matched:
        debit exchange amount = remaining debit residual − matched amount
        and, when that is not zero at the company currency's precision:
                instruct the debit item, column balance, amount debit exchange amount
                remaining debit residual = remaining debit residual − debit exchange amount
                and, when the debit currency is the company currency,
                        remaining debit residual in currency = remaining debit residual in currency − debit exchange amount

when the credit side is fully matched:
        credit exchange amount = remaining credit residual + matched amount
        and, when that is not zero at the company currency's precision:
                instruct the credit item, column balance, amount credit exchange amount
                remaining credit residual = remaining credit residual − credit exchange amount
                and, when the credit currency is the company currency,
                        remaining credit residual in currency = remaining credit residual in currency − credit exchange amount
```

### 14.3 Branch C — the matching is measured in a foreign currency, side only partly matched

The side is not finished, so the repair does not clear it; instead it keeps the ratio between the
two remaining amounts equal to the ratio the item was recorded at, so that the *next* matching on
the same item still works.

```formula
when the debit side is not fully matched:
        debit exchange amount = partial debit amount − matched amount
        and, when compare at the company currency( debit exchange amount , 0 ) > 0:
                instruct the debit item, column balance, amount debit exchange amount
                remaining debit residual = remaining debit residual − debit exchange amount
                and the same adjustment to the remaining debit residual in currency when the debit currency is the company currency

when the credit side is not fully matched:
        credit exchange amount = matched amount − partial credit amount
        and, when compare at the company currency( credit exchange amount , 0 ) < 0:
                instruct the credit item, column balance, amount credit exchange amount
                remaining credit residual = remaining credit residual − credit exchange amount
                and the same adjustment to the remaining credit residual in currency when the credit currency is the company currency
```

Note the asymmetry of the two guards: the debit repair is booked only when it is **strictly
positive**, the credit repair only when it is **strictly negative**. A repair of the opposite sign
would move the residual the wrong way and is discarded. Without this branch, a partial settlement
at a different rate would leave a residual whose implied rate was neither the recording rate nor
the settlement rate.

### 14.4 Branch D — no exchange difference at all

The whole of section 14 is skipped when the caller has suppressed exchange differences. Two
distinct suppressions exist and both must be honoured: one that suppresses differences for the
current operation, and one that suppresses them for the current operation **and** every
reconciliation it triggers. The second is what prevents an exchange difference entry from
generating an exchange difference entry of its own when its correction line is matched back against
the item it repairs.

### 14.5 The sign convention of the instruction

An instruction whose amount is **positive** means the item has too much debit left and the excess
must be written off, which is a **loss** when the item is an asset. An instruction whose amount is
**zero or negative** means the item has too much credit left, which is a **gain**. Section 15 turns
the sign into an account choice.

### 14.6 Worked example — a customer invoice collected after the rate rose, producing a loss

A company's main currency is `USD`. `EUR` carries rates of zero point nine two zero zero from
2026-01-01 and zero point nine five zero zero from 2026-03-01. Both currencies have a rounding
factor of one hundredth.

**The invoice.** A customer invoice of one thousand euro is issued on 2026-01-15 at the applied
rate zero point nine two zero zero:

```formula
receivable balance = round onto USD( 1 000.00 ÷ 0.9200 ) = round( 1 086.9565217391305 ) = 1 086.96
```

The receivable item carries a balance of plus one thousand eighty-six point nine six and an amount
in currency of plus one thousand.

**The payment.** The customer pays one thousand euro on 2026-03-10 at the applied rate zero point
nine five zero zero:

```formula
liquidity balance = round onto USD( 1 000.00 ÷ 0.9500 ) = round( 1 052.6315789473683 ) = 1 052.63
```

The payment's receivable item carries a balance of minus one thousand fifty-two point six three and
an amount in currency of minus one thousand.

**The matching.**

```formula
debit map  : USD → ( 1 086.96 , 1 ) ; EUR → ( 1 000.00 , 1 000 ÷ 1 086.96 = 0.920000736 )
credit map : USD → ( −1 052.63 , 1 ) ; EUR → ( −1 000.00 , 1 000 ÷ 1 052.63 = 0.950000475 )
reconciliation currency = EUR
debit recon amount = 1 000.00 , credit recon amount = 1 000.00 , comparison = 0  →  both sides fully matched
minimum = 1 000.00

debit interval  : rate 1 086.96 ÷ 1 000 = 1.08696 , half a step = 0.005
                  lowest = round( 999.995 × 1.08696 ) = 1 086.95 , middle = 1 086.96 , highest = 1 086.97
credit interval : rate 1 052.63 ÷ 1 000 = 1.05263 , half a step = 0.005
                  lowest = 1 052.62 , middle = 1 052.63 , highest = 1 052.64

partial debit amount  = the smaller of ( 1 086.96 , 1 086.96 ) = 1 086.96
partial credit amount = the smaller of ( 1 052.63 , 1 052.63 ) = 1 052.63
matched amount        = the smaller of ( 1 086.96 , 1 052.63 ) = 1 052.63

tolerance band : compare( 1 086.96 , 1 052.64 ) = +1, not ≤ 0  →  no collapse
matched amount in the debit currency  = 1 000.00
matched amount in the credit currency = 1 000.00
```

**The exchange difference**, branch B:

```formula
the debit side is fully matched  :  1 086.96 − 1 052.63 = +34.33  →  instruction on the invoice receivable item, balance column, +34.33
the credit side is fully matched : −1 052.63 + 1 052.63 =   0.00  →  nothing
```

The company invoiced one thousand euro when they were worth one thousand eighty-six point nine six
dollars and collected one thousand euro when they were worth one thousand fifty-two point six three
dollars: a realised **loss** of thirty-four point three three dollars. The entry produced is in
[accounting-effects.md](accounting-effects.md) section 5.1.

### 14.7 Worked example — the same invoice collected after the rate fell, producing a gain

The same invoice, balance plus one thousand eighty-six point nine six and amount in currency plus
one thousand. The euro rate on the payment date is zero point eight nine zero zero instead:

```formula
liquidity balance = round onto USD( 1 000.00 ÷ 0.8900 ) = round( 1 123.5955056179776 ) = 1 123.60
the credit item   : balance = −1 123.60 , amount in currency = −1 000.00

reconciliation currency = EUR , debit recon amount = credit recon amount = 1 000.00 , comparison = 0
partial debit amount  = 1 086.96
partial credit amount = 1 123.60
matched amount        = 1 086.96
tolerance band : compare( 1 086.96 , the lowest of the credit interval = 1 123.59 ) = −1, not ≥ 0  →  no collapse

the debit side is fully matched  :  1 086.96 − 1 086.96 =   0.00  →  nothing
the credit side is fully matched : −1 123.60 + 1 086.96 = −36.64  →  instruction on the payment receivable item, balance column, −36.64
```

A realised **gain** of thirty-six point six four dollars.

---

## 15. Choosing the exchange journal and the gain or loss account

**Inputs:** the company of the matching and the signed amount of the exchange instruction.
**Outputs:** a journal and an account.

```formula
exchange journal = the company's exchange difference journal

exchange account = the company's loss exchange account    when the instruction amount > 0     a loss
exchange account = the company's gain exchange account    when the instruction amount ≤ 0     a gain
```

The company used is the company of the invoice, bill, credit note, debit note or receipt among the
matched entries when there is one, and otherwise the company of the matched items. When no company
can be determined, nothing is produced.

The three configuration values are mandatory as soon as any exchange difference is needed; see
`MCUR-102` for the exchange journal, `MCUR-103` for the loss account and `MCUR-104` for the gain
account, each with its exact message.

---

## 16. The date of an exchange difference entry

**Inputs:** the accounting dates of the two matched items, the exchange journal, the lock dates of
the company, the numbering reset period of the exchange journal's sequence, and today's date.
**Output:** the accounting date of the exchange difference entry.

Algorithm:

1. Let the base date be the greater of the two matched items' accounting dates.
2. Pass the base date through the journal's accounting-date rule:
   - Let the violated lock dates be the lock dates of the company that the base date violates,
     taking into account whether the entry affects the tax report. When the list is not empty,
     replace the base date with the latest violated lock date plus one day.
   - Let the reset period be the numbering reset period deduced from the highest entry number
     already used in that journal: monthly, yearly, or none when the journal has no entry yet.
   - For a journal that is not a sale journal — and the exchange journal never is, because it is of
     type general:
     - When the journal has no entry yet, or the reset period is monthly: when today's year and
       month are later than the base date's year and month, the result is the last day of the base
       date's month; otherwise the result is the later of the base date and today.
     - When the reset period is yearly: when today's year is later than the base date's year, the
       result is the thirty-first of December of the base date's year; otherwise the result is the
       later of the base date and today.
     - Otherwise the result is the base date unchanged.
3. Let the candidate be the result of step 2. For **every** item submitted with the batch of
   instructions — including an item whose instruction is afterwards skipped for being zero at its
   own precision — replace the candidate with the later of the candidate and that item's accounting
   date.
4. The entry's date is that candidate.

### 16.1 Worked examples

A company whose exchange journal numbers entries with a monthly reset. Two items are matched, dated
2017-01-01 and 2016-01-01, so the base date is 2017-01-01.

| Today | Result of step 2 | Result of step 3 | Entry date |
|---|---|---|---|
| 2019-01-01 | Today's year and month are later than January 2017, so the last day of January 2017 | the later of 2017-01-31, 2017-01-01 and 2016-01-01 | **2017-01-31** |
| 2017-01-15 | Today's year and month equal January 2017, so the later of 2017-01-01 and 2017-01-15 | the later of 2017-01-15, 2017-01-01 and 2016-01-01 | **2017-01-15** |
| 2016-12-01 | Today is earlier, so the later of 2017-01-01 and 2016-12-01 | the later of 2017-01-01, 2017-01-01 and 2016-01-01 | **2017-01-01** |

With a yearly reset instead and today at 2019-01-01, step 2 yields the thirty-first of December
2017 and the entry date is **2017-12-31**.

---

## 17. The date of the reversal of an exchange difference entry

When a reconciliation is undone, a posted exchange difference entry is reversed rather than
deleted.

```formula
reversal date = the original entry's date
violated lock dates = the lock dates of the company that the original entry's date violates,
                      taking into account whether the entry affects the tax report
reversal date = the latest violated lock date + one day     when that list is not empty
```

The reversal's internal reference is set to "Reversal of: *the entry number of the original
entry*". A draft exchange difference entry is deleted outright rather than reversed.

### 17.1 Worked examples

A posted exchange difference entry numbered `EXCH/2026/03/0004`, dated 2026-03-31, carrying no tax
and therefore not affecting the tax report:

| Lock dates of the company | Violated by 2026-03-31 | Latest violated | Reversal date |
|---|---|---|---|
| None set | none | none | 2026-03-31, the original date unchanged |
| Fiscal year lock date 2025-12-31 | none, because 2026-03-31 is later | none | 2026-03-31 |
| Fiscal year lock date 2026-03-31 and hard lock date 2026-02-28 | both, because 2026-03-31 falls on or before each of them | 2026-03-31 | 2026-04-01 |
| Fiscal year lock date 2026-03-31 and hard lock date 2026-04-30 | both | 2026-04-30 | 2026-05-01 |

In the third row the displacement is one day only, which is the smallest date that clears the lock.
The reversal therefore lands in the following month and carries the number of that month's
sequence; the original entry keeps its own date and number, which leaves both visible in the audit
trail.

Contrast with section 16: the date of the *original* exchange difference entry is additionally
pushed by the journal's numbering reset period and by today's date, while the date of a *reversal*
is pushed only by lock dates.

---

## 18. Recomputing the residual amounts of a journal item

**Inputs:** the item's balance and amount in currency, and every partial matching the item takes
part in.
**Outputs:** the residual amount, the residual amount in currency and the reconciled flag.

Algorithm:

1. When the item's account does not allow reconciliation and is neither a cash account nor a credit
   card account, both residuals are zero and the reconciled flag is false.
2. Otherwise sum over the matchings in which the item is the debit side: the matched amounts give
   the matched total in the company currency, and the matched amounts in the debit currency give
   the matched total in currency, **rounded inside the aggregation** to the number of decimal
   places of the debit currency.
3. Sum over the matchings in which the item is the credit side in the same way, rounding to the
   decimal places of the credit currency.
4. Compute:

```formula
residual amount             = round onto the company currency( balance            − matched as debit in company currency + matched as credit in company currency )
residual amount in currency = round onto the item currency   ( amount in currency − matched as debit in currency         + matched as credit in currency )
is reconciled               = the residual amount is zero at the company currency AND the residual amount in currency is zero at the item currency
```

where the item currency falls back to the company currency when the item has no currency.

Two details a rebuild must not skip: the foreign sums are rounded **inside** the aggregation, and
an item is reconciled only when **both** residuals are zero. An item can have a zero residual in
the company currency and a non-zero residual in its own currency, or the reverse; it is not
reconciled in either case, and that is exactly the situation the exchange difference entry exists
to repair.

**Worked example.** The invoice receivable item of section 14.6 carries a balance of one thousand
eighty-six point nine six and an amount in currency of one thousand. It takes part in two matchings
as the debit side: the first with a matched amount of one thousand fifty-two point six three and a
matched amount in the debit currency of one thousand, the second — against the exchange difference
correction line — with a matched amount of thirty-four point three three and a matched amount in
the debit currency of zero.

```formula
residual amount             = round( 1 086.96 − ( 1 052.63 + 34.33 ) + 0 ) = round( 0.00 ) = 0.00
residual amount in currency = round( 1 000.00 − ( 1 000.00 +  0.00 ) + 0 ) = round( 0.00 ) = 0.00
is reconciled               = true
```

---

## 19. Deciding that a group of matched items is fully reconciled

**Inputs:** the set of items connected by matchings, after the matchings and the exchange
differences have been created.
**Output:** whether a Full Reconciliation record is created for the group.

1. Let *several currencies* be true when the set of distinct currencies among the items of the
   group has more than one member.
2. For each item of the group:
   - an item whose reconciled flag is true counts as reconciled;
   - otherwise an item that takes part in no matching at all does not count as reconciled;
   - otherwise, when *several currencies* is true, the item counts as reconciled when its residual
     amount is zero at the company currency's precision;
   - otherwise the item counts as reconciled when its residual amount in currency is zero at the
     item currency's precision.
3. The group is fully reconciled when every item counts as reconciled.

The second clause exists for an item whose balance is zero while its amount in currency is not,
such as a bare exchange difference line: without a matching it must not be declared reconciled
merely because its company currency residual is nil. The third and fourth clauses differ
deliberately: when several currencies are involved the company currency is the only common
denominator and is the one that must close; when a single currency is involved the document
currency column is the one that must close, because the company currency column may legitimately
retain a difference that a further exchange difference entry will clear.

**Worked example.** Two items in the same foreign currency with a rounding factor of one
thousandth:

| Item | Balance | Amount in currency |
|---|---|---|
| First | 0.00 | −0.020 |
| Second | 0.00 | +0.010 |

Matching them consumes ten thousandths in the foreign currency and nothing in the company currency.
Afterwards the first item has a residual amount in currency of minus ten thousandths, which is not
zero at a rounding factor of one thousandth. Only one currency is involved, so the document
currency test applies and the group is not fully reconciled. Adding a third item with an amount in
currency of plus ten thousandths and matching it against the first closes the group, and a single
Full Reconciliation record then covers all three items.

---

## 20. Reporting rate tables

To consolidate figures belonging to companies whose main currencies differ, a rate table is built
for the duration of a report run. It maps, for each company and each reporting period, a factor to
apply directly to an amount expressed in that company's main currency in order to express it in the
main currency of the company the report is run for.

The table has one row per combination of company, period key and rate type, with the columns:
company, period key, valid-from date, next-change date, rate type and factor. The rate type is one
of `historical`, `current` and `average`.

### 20.1 The trivial case

When every company involved shares a single main currency, no table is built. A synthetic table is
used in which every factor is one, with no date bounds, so that the consolidation query is written
the same way in both cases and no temporary storage is created.

### 20.2 The domestic rows

Every company whose main currency equals the reporting company's main currency receives a factor of
one for every requested rate type, with no date bounds.

### 20.3 The current factor

One row per foreign-currency company per period, with no valid-from and no next-change date.

```formula
reference factor( period ) = rate of( the reporting company's currency , the reporting root company , the period's end date )

current factor( company , period ) = reference factor( period ) ÷ rate of( that company's currency , the reporting root company , the period's end date )
        when a rate row exists for that company's currency on or before the period's end date
current factor( company , period ) = 1
        otherwise
```

#### Worked example

The reporting company's main currency is `USD`, rounding factor one hundredth. A subsidiary's main
currency is `EUR`, with two rate rows: zero point nine zero zero zero from 2024-01-01 and zero
point nine five zero zero from 2024-07-01. The reporting period ends on 2024-12-31.

```formula
case 1 : the reporting currency carries no rate row
         reference factor = rate of( USD , reporting root , 2024-12-31 ) = 1
         rate of( EUR , reporting root , 2024-12-31 ) = 0.9500
         current factor   = 1 ÷ 0.9500 = 1.052632

case 2 : the reporting currency itself carries a rate row of 1.1000 from 2024-01-01
         reference factor = 1.1000
         current factor   = 1.1000 ÷ 0.9500 = 1.157895

case 3 : the subsidiary's currency has no rate row dated on or before 2024-12-31
         current factor   = 1
```

A subsidiary balance of ten thousand euro consolidates in case 1 as
round( 10 000.00 × 1.052632 ) = 10 526.32 dollars, and in case 2 as
round( 10 000.00 × 1.157895 ) = 11 578.95 dollars. In case 3 it consolidates unchanged as ten
thousand, which is deliberate: an unknown rate must not silently scale a balance towards zero.

### 20.4 The historical factor

One row per foreign-currency company per rate change of that company's currency, valid from the
rate's date until the day before the next rate's date. The numerator is the reporting currency's
own rate in force **on the same date**, so that a movement of the reporting currency is reflected
as well:

```formula
historical factor( rate row ) = rate of( the reporting company's currency , the reporting root company , the rate row's date ) ÷ the rate row's technical rate
```

Rows dated after the period's end date are excluded; when a previous period has already been
produced, rows dated on or before that previous period's end date are excluded as well, so that the
table is not populated twice for the same span.

#### Worked example

The reporting currency is `USD` with no rate row, so its rate is one at every date; the
subsidiary's currency is `EUR` with zero point nine zero zero zero from 2024-01-01 and zero point
nine five zero zero from 2024-07-01; a third row of zero point nine seven zero zero is dated
2025-02-01. The reporting period runs from 2024-01-01 to 2024-12-31 and no previous period has been
produced.

| Rate row | Included | Valid from | Next change | Factor |
|---|---|---|---|---|
| 2024-01-01, rate 0.9000 | yes | 2024-01-01 | 2024-07-01 | 1 ÷ 0.9000 = 1.111111 |
| 2024-07-01, rate 0.9500 | yes | 2024-07-01 | none | 1 ÷ 0.9500 = 1.052632 |
| 2025-02-01, rate 0.9700 | no, dated after the period's end date | not applicable | not applicable | not applicable |

A fixed asset acquired on 2024-03-15 for five thousand euro falls in the window that opens on
2024-01-01 and closes on 2024-06-30, and consolidates as round( 5 000.00 × 1.111111 ) = 5 555.56
dollars, keeping that value in every later report, which is the purpose of a historical rate. A
second asset acquired on 2024-09-10 for five thousand euro falls in the window that opens on
2024-07-01 and consolidates as round( 5 000.00 × 1.052632 ) = 5 263.16 dollars.

When a previous period ending 2024-06-30 has already been produced, the row of 2024-01-01 is
excluded from this period's table and only the row of 2024-07-01 is emitted.

Now suppose the reporting currency also carries rate rows, one point zero zero zero zero from
2024-01-01 and one point zero four zero zero from 2024-07-01. The factors become
1.0000 ÷ 0.9000 = 1.111111 for the first window and 1.0400 ÷ 0.9500 = 1.094737 for the second,
because the numerator is read on the same date as the row it divides.

### 20.5 The average factor

One row per foreign-currency company per period. The period is cut into segments at every rate
change — of either the foreign currency or the reporting currency — that falls strictly inside the
period; within a segment both rates are constant. The factor is the day-weighted mean of the
per-segment factors:

```formula
for each segment, with its start date and the start of the next segment
        ( the period's end date plus one day, for the last segment ):
    days of the segment       = the next start − this start, in whole days
    foreign rate of the segment  = the rate of the foreign currency in force at this start,   or 1 when none
    domestic rate of the segment = the rate of the reporting currency in force at this start, or 1 when none
    factor of the segment     = domestic rate of the segment ÷ foreign rate of the segment

average factor = ( sum over the segments of factor × days ) ÷ ( sum over the segments of days )
```

Segments of zero days are discarded. When the period has no start date, the start of the calendar
year containing the period's end date is used.

#### Worked example

The reporting company's main currency is `USD` with no rate rows, so the domestic rate is one
throughout. A subsidiary's main currency has rates of zero point nine zero zero zero from
2024-01-01 and zero point nine five zero zero from 2024-07-01. The reporting period runs from
2024-01-01 to 2024-12-31.

```formula
breakpoints : 2024-01-01, the period start, and 2024-07-01, a rate change inside the period
segment 1   : 2024-01-01 up to 2024-07-01 exclusive  →  182 days, foreign rate 0.9000, factor 1 ÷ 0.9000 = 1.111111
segment 2   : 2024-07-01 up to 2025-01-01 exclusive  →  184 days, foreign rate 0.9500, factor 1 ÷ 0.9500 = 1.052632

average factor = ( 1.111111 × 182 + 1.052632 × 184 ) ÷ 366
               = ( 202.222222 + 193.684211 ) ÷ 366
               = 395.906433 ÷ 366
               = 1.081712
```

An amount of ten thousand in the subsidiary's currency consolidates as
10 000.00 × 1.081712 = 10 817.12 dollars at the average rate.

---

## 21. A bank transaction in up to three currencies

**Inputs:** the statement line's amount in the bank account currency, its optional transacted
currency and amount in currency, the journal, the company and the line's date.
**Outputs:** the two journal items generated for the transaction.

Let the company currency be the main currency of the company; let the **journal currency** be the
journal's currency when it has one and the company currency otherwise — this is the bank account's
own currency; and let the **transacted currency** be the statement line's transacted currency when
it has one and the journal currency otherwise. The *journal amount* is the statement line's amount,
expressed in the journal currency.

### 21.1 The transaction amount and the company amount

```formula
transaction amount = journal amount        when the transacted currency = the journal currency
transaction amount = the amount in currency of the statement line   otherwise

company amount = journal amount                                              when the journal currency = the company currency
company amount = transaction amount                                          when the transacted currency = the company currency
company amount = convert( journal amount , from the journal currency to the company currency , the company , the line's date )   otherwise
```

The third branch deliberately converts the *journal* amount, not the transaction amount: the bank
moved a known quantity of the bank account currency, and that quantity is what the ledger must
reflect (`MCUR-156`).

### 21.2 The two journal items

| Item | Account | Item currency | Amount in currency | Debit | Credit |
|---|---|---|---|---|---|
| Liquidity | The journal's default account | the journal currency | the journal amount | the company amount when it is positive, else zero | the negated company amount when it is negative, else zero |
| Counterpart | The supplied counterpart account, defaulting to the journal's suspense account | the transacted currency | the negated transaction amount | the negated company amount when it is negative, else zero | the company amount when it is positive, else zero |

When no counterpart account is supplied and the journal has no suspense account, the operation
fails with the message of `MCUR-155`.

### 21.3 Worked example — three distinct currencies

A company's main currency is `USD`. A bank account is held in `EUR`, so the journal currency is the
euro. A customer pays in `GBP`. The bank reports that eight hundred fifty euro arrived and states
that the transaction was seven hundred thirty pounds. The statement line is dated 2026-04-10, and
the euro rate on that date is zero point nine two zero zero, the dollar carrying no rate row.

```formula
the company currency = USD , the journal currency = EUR , the transacted currency = GBP
journal amount     = 850.00 EUR
transaction amount = 730.00 GBP                 the transacted currency differs from the journal currency
company amount     = convert( 850.00 , EUR into USD ) = round onto USD( 850.00 × ( 1.0 ÷ 0.9200 ) )
                   = round( 923.9130434782609 ) = 923.91 USD
```

| Item | Account | Item currency | Amount in currency | Debit | Credit |
|---|---|---|---|---|---|
| Liquidity | Bank | `EUR` | +850.00 | 923.91 | 0.00 |
| Counterpart | Suspense | `GBP` | −730.00 | 0.00 | 923.91 |

The entry balances in the company currency column. The two document currency columns do not balance
against each other, and are not required to: each item carries its own document currency, and the
balancing requirement applies to the company currency column only.

### 21.4 Deriving a counterpart amount at the bank's own rates

When an item is matched against a bank transaction, the amounts of the counterpart item are derived
from the rates the bank actually applied rather than from the rate table, so that the transaction
reconciles exactly.

Let the posted transaction supply the transaction amount and its currency — taken from the suspense
line when that is the only counterpart, and from the statement line otherwise — the journal amount
and its currency, being the sum of the liquidity items and their currency, and the company amount,
being the sum of the liquidity balances. Then:

```formula
rate from the journal currency to the transaction currency = | transaction amount | ÷ | journal amount |     zero when the journal amount is zero
rate from the company currency to the journal currency     = | journal amount |     ÷ | company amount |     zero when the company amount is zero
```

Given an amount to be settled, expressed in some currency with a value in that currency and, when
that currency is neither the transaction currency nor the journal currency, a balance in the
company currency:

1. **When the currency is the transaction currency:** the resulting amount in currency is the
   supplied amount; an intermediate value is the supplied amount divided by the rate from the
   journal currency to the transaction currency, rounded onto the journal currency, or zero when
   that rate is zero; the resulting balance is that intermediate value divided by the rate from the
   company currency to the journal currency, rounded onto the company currency, or zero when that
   rate is zero.
2. **When the currency is the journal currency:** the resulting amount in currency is the supplied
   amount multiplied by the rate from the journal currency to the transaction currency, rounded
   onto the transaction currency; the resulting balance is the supplied amount divided by the rate
   from the company currency to the journal currency, rounded onto the company currency, or zero
   when that rate is zero.
3. **Otherwise:** an intermediate value is the supplied balance multiplied by the rate from the
   company currency to the journal currency, rounded onto the journal currency; the resulting
   amount in currency is that intermediate value multiplied by the rate from the journal currency
   to the transaction currency, rounded onto the transaction currency; the resulting balance is the
   supplied balance unchanged.

**Worked example.** Continuing section 21.3, the bank's implied rates are:

```formula
rate from the journal currency to the transaction currency = 730.00 ÷ 850.00 = 0.858823529
rate from the company currency to the journal currency     = 850.00 ÷ 923.91 = 0.920002
```

An invoice of seven hundred thirty pounds is being settled, so the first branch applies:

```formula
resulting amount in currency = 730.00 GBP
intermediate                 = round onto EUR( 730.00 ÷ 0.858823529 ) = round( 850.0000 ) = 850.00
resulting balance            = round onto USD( 850.00 ÷ 0.920002 )    = round( 923.9082… ) = 923.91
```

The counterpart item therefore carries seven hundred thirty pounds and nine hundred twenty-three
point nine one dollars, matching the liquidity side exactly and leaving no rounding residue.

---

## 22. The amount proposed by the payment registration screen

When several documents are settled at once by a single payment, the screen shows a total expressed
in the payment's currency.

**Inputs:** the residual amounts of the selected items, grouped by the currency of the item; the
payment currency; the company currency; the payment date.
**Output:** a single total expressed in the payment currency.

For each currency group, with its accumulated residual in the company currency and its accumulated
residual in the document currency:

1. When the group's currency **is** the payment currency, add the group's residual in the document
   currency.
2. Otherwise, when the group's currency is not the company currency and the payment currency **is**
   the company currency, add the conversion of the group's residual in the document currency from
   the group's currency into the company currency at the payment date.
3. Otherwise add the conversion of the group's residual in the **company** currency from the
   company currency into the payment currency at the payment date.

Branches two and three have the same effect whenever the payment currency is not the currency of
the item: the company currency residual is the quantity converted, because it is the only value
both sides agree on.

**Worked example.** A company's main currency is `USD`. Two open invoices are selected: one for one
thousand euro whose receivable item carries a balance of one thousand eighty-six point nine six,
and one for five hundred dollars. The payment is made in euro on a date whose euro rate is zero
point nine five zero zero.

```formula
the euro group   : the group's currency is the payment currency  →  total = 1 000.00
the dollar group : the group's currency is the company currency and the payment currency is not
                   →  convert( 500.00 , USD into EUR ) = round onto EUR( 500.00 × 0.9500 ) = 475.00
total = 1 000.00 + 475.00 = 1 475.00 euro
```

When the payment currency is changed on the screen after an amount has been typed by hand, the
typed amount is itself converted from the previously selected currency into the new one at the
payment date, so that the accountant does not have to retype it.

---

## 23. Rendering a number as a string

```formula
string form = decimal string( value , decimal places )
```

Algorithm:

1. When the value passes the zero test at the given number of digits, replace it by zero. This
   removes a negative zero and prevents a string of the form minus zero point zero zero.
2. Render the value with **exactly** that many digits after the decimal point, padding with zeros,
   using a full stop as the decimal point and no digit grouping.

The routine is a *presentation* routine, not a rounding routine: a rebuild must round first and
render second, and must never rely on the renderer to round. The renderer must not use a
shortest-representation conversion, because such a conversion silently drops significant digits on
large values.

### 23.1 Splitting into a whole part and a fractional part

```formula
whole part , fractional part = split( value , decimal places )
```

1. Round the value onto the given number of digits.
2. Render it as in section 23.
3. Split the rendered string at the decimal point.
4. When the number of digits is zero, the fractional part is the empty string.

The fractional part always has exactly the requested number of characters, padded with trailing
zeros. Worked values: one point four three two at two digits gives one and forty-three; one point
four nine at one digit gives one and five; one point one at three digits gives one and one hundred;
one point one two at zero digits gives one and the empty string. A companion form returns the two
parts as whole numbers instead of strings; when the number of digits is zero the fractional part is
the whole number zero.

### 23.2 Rendering for structured interchange

When an amount must be written into a structured interchange document, a variant is used that
rounds, renders to a string, and then reads the string back as a number. The point is that the
resulting number's own shortest representation is the rendered string, so that a serialiser which
cannot be told how to format numbers still emits the intended digits. The value that comes back
must **not** be used for further arithmetic.

---

## 24. Formatting an amount for a reader

### 24.1 The language-aware number format

**Inputs:** a value, a number of digits or a source of digits, a rounding method, a rounding unit,
a currency, and the reader's language.
**Output:** a display string.

1. **Decide the number of digits.**
   - When the rounding unit is *decimals*: when a decimal precision record was named, its digit
     count wins; otherwise, when a currency was supplied, the currency's decimal places win;
     otherwise the caller's digit count is used, defaulting to two.
   - When the rounding unit is anything else — units, thousands, lakhs or millions — the digit
     count is **zero**.
2. **Scale.** Divide the value by the rounding unit's factor: one for decimals and for units, one
   thousand for thousands, one hundred thousand for lakhs, one million for millions.
3. **Round.** Round the scaled value onto that number of digits with the requested rounding method,
   whose default here is **half to even**.
4. **Render.** Produce the number with exactly that many fractional digits.
5. **Group.** Insert the language's thousands separator according to the language's grouping
   pattern, read from the decimal separator outwards by the rule of [entities.md](entities.md)
   section 14.1.
6. **Replace the decimal point** with the language's decimal separator.
7. **Attach the symbol,** when a currency with a symbol was supplied: the formatted number, a
   **non-breaking space**, and the symbol — in that order when the symbol position is `after`, and
   in the reverse order when it is `before`.

An empty value renders as the empty string rather than as zero.

### 24.2 The currency-specific format

A second, currency-first routine exists and is the one a currency uses when asked to format an
amount:

1. Build a render pattern with exactly the currency's decimal places.
2. Round the amount onto the **currency's rounding factor**, half away from zero.
3. Render and group it with the reader's language, always with grouping on.
4. Replace every ordinary space by a **non-breaking space**, and replace every minus sign by a
   minus sign **followed by** a **zero-width non-breaking space**. The second substitution stops a
   line break from separating the sign from the digits.
5. When trailing zeros are not wanted, strip a run of trailing zeros together with the decimal
   separator that precedes it.
6. Prefix the symbol followed by a non-breaking space when the symbol position is `before`; suffix
   a non-breaking space followed by the symbol when it is `after`. A currency with no symbol
   contributes only the space.

### 24.3 The negative-zero rule

A currency's format operation adds a positive zero to the amount before formatting. The effect is
that an amount which rounds to a negative zero is presented as a plain zero, never as a minus sign
in front of nothing. A rebuild must reproduce this, because a report showing a minus sign in front
of nothing is treated as a defect.

### 24.4 Worked formatting examples

Language conventions used below: the English convention has a full stop as the decimal separator, a
comma as the thousands separator and the pattern `[3,0]`; the French convention has a comma as the
decimal separator, a narrow space as the thousands separator and the pattern `[3,0]`; the German
convention has a comma as the decimal separator and a full stop as the thousands separator; the
South Asian convention has a full stop, a comma and the pattern `[3,2,0]`.

| Amount | Currency | Language convention | Result, spaces shown as ordinary spaces |
|---|---|---|---|
| 1 234 567.891 | `USD`, factor 0.01, symbol before | English | `$ 1,234,567.89` |
| 1 234 567.891 | `EUR`, factor 0.01, symbol after | French | `1 234 567,89 €` |
| 1 234 567.891 | `INR` (the Indian rupee), factor 0.01, symbol before | South Asian | `₹ 12,34,567.89` |
| 1 234 567.891 | `JPY`, factor 1, symbol before | English | `¥ 1,234,568` |
| 1 234.5 | `USD`, factor 0.01, symbol before | English | `$ 1,234.50` |
| 1 234.5 | `EUR`, factor 0.01, symbol after | German | `1.234,50 €` |
| 1 234.5 | `JPY`, factor 1, symbol before | English | `¥ 1,235` — the tie is broken away from zero by the currency routine |
| 1 234.56 | `JPY`, factor 1, symbol before | English | `¥ 1,235` |
| 0.125 | `BHD` (the Bahraini dinar), factor 0.001, symbol after | English | `0.125 BD` |
| 1 000.00 | `USD`, trailing zeros suppressed | English | `$ 1,000` |
| 1 000.50 | `USD`, trailing zeros suppressed | English | `$ 1,000.5` |
| −1 234.5 | `USD`, factor 0.01, symbol before | English | `$ -1,234.50`, with a zero-width non-breaking space after the minus sign |
| −0.001 | `USD` | English | `$ 0.00` — the amount rounds to a negative zero and the sign is dropped |
| −0.0 | `USD` | English | `$ 0.00` |

### 24.5 The compact metric rendering

For dashboards a compact form exists:

1. While the absolute value is one thousand or more, divide by one thousand and step through the
   suffixes: none, `k`, `M`, `G`, stopping at `T`.
2. Round to one decimal place by the language runtime's own nearest-number rule.
3. Render with the shortest form that reproduces the value, so that a whole number shows no
   decimal digits.
4. Attach the currency symbol immediately before the number when the symbol position is `before`,
   or after a single ordinary space when it is `after`.

Worked values: one hundred twenty-three thousand four hundred fifty-six point seven eight nine
renders as `123.5k`; one hundred twenty-three thousand point seven eight nine renders as `123k`;
minus one hundred twenty-three thousand four hundred fifty-six point seven eight nine renders as
`-123.5k`; zero point seven eight nine renders as `0.8`. With a currency whose symbol is a dollar
sign placed before, the first becomes `$123.5k`.

Deliberate limitation: the sequence stops at the suffix for one million million. Larger values keep
that suffix and grow the number.

---

## 25. Writing an amount in words

**Inputs:** an amount, a currency and the reader's language.
**Output:** a string naming the amount in words, used on printed documents when the company setting
for spelling the total is on, and by legal document formats that require an amount in letters.

1. Render the amount as a fixed-point decimal string with exactly the currency's decimal places.
   This rendering rounds half to even at that digit count, unlike the currency rounding of section
   3, which rounds half away from zero; the difference can show only at an exact tie in the digit
   beyond the last one printed.
2. Split the rendered string at the decimal point into a whole part and a fractional part. When the
   decimal places are zero there is no separator and the fractional part is empty.
3. Read the whole part as a whole number.
4. When the amount minus that whole number passes the **currency's zero test**, the result is: the
   whole number spelled out in the reader's language, in title case, a space, and the currency's
   unit label.
5. Otherwise the result is: the whole number spelled out, a space, the unit label, a space, the
   word "and", a space, the fractional digits read as a whole number and spelled out, a space, and
   the currency's subunit label. An empty fractional string is read as zero.
6. When the reader's language has no spelling rules available, the spelling falls back to English.
7. When no spelling facility is present at all, the result is the empty string and a warning is
   logged.

**The fractional part is read as a whole number of subunits, not as a fraction.** A fractional part
of `05` yields the words for five, not for five hundredths, because the subunit label already
supplies the scale.

Worked examples with a currency whose unit label is "Dollars", whose subunit label is "Cents" and
whose decimal places are two, in English:

| Amount | Rendered | Whole | Fraction | Result |
|---|---|---|---|---|
| 1234.00 | `1234.00` | 1234 | `00` | "One Thousand, Two Hundred And Thirty-Four Dollars" |
| 1234.56 | `1234.56` | 1234 | `56` | "One Thousand, Two Hundred And Thirty-Four Dollars and Fifty-Six Cents" |
| 1234.50 | `1234.50` | 1234 | `50` | "One Thousand, Two Hundred And Thirty-Four Dollars and Fifty Cents" |
| 1234.05 | `1234.05` | 1234 | `05` | "One Thousand, Two Hundred And Thirty-Four Dollars and Five Cents" |
| 0.99 | `0.99` | 0 | `99` | "Zero Dollars and Ninety-Nine Cents" |
| 0.05 | `0.05` | 0 | `05` | "Zero Dollars and Five Cents" |
| 1234.004 | `1234.00` | 1234 | `00` | "One Thousand, Two Hundred And Thirty-Four Dollars" — the fraction vanished at step 1 |

With a currency whose decimal places are zero, step 2 produces an empty fractional part, step 4
always applies, and the subunit label is never used: for `JPY` with the unit label "Yen", an amount
of one thousand two hundred thirty-four point five six renders as `1235`, the whole number is one
thousand two hundred thirty-five, the difference of minus zero point four four is zero at a
rounding factor of one, and the result is "One Thousand, Two Hundred And Thirty-Five Yen".

---

## 26. Worked example: a bill settled after the rate rose, producing a gain

This example walks the whole chain: document, payment, reconciliation currency choice, partial
amounts, exchange difference amount, and the exact journal items produced. Every number is derived
by the formulas of the preceding sections.

### 26.1 The setting

| Element | Value |
|---|---|
| Company | Northwind, reporting in `EUR` |
| Document currency | `USD` |
| Rate row on 1 March 2026 | technical rate of `USD` = **1.10** — one euro buys one point ten United States dollars |
| Rate row on 1 April 2026 | technical rate of `USD` = **1.20** |
| Rate rows for `EUR` | none, so the euro's rate is the last resort of one |
| Exchange journal | Miscellaneous Operations |
| Gain exchange account | `7760 Foreign Exchange Gain` |
| Loss exchange account | `6560 Foreign Exchange Loss` |

The document is a **vendor bill**, because a rate that rises from one point one zero to one point
two zero means the euro strengthens against the dollar, which makes a dollar *liability* cheaper to
settle and therefore produces a **gain**. The mirror case — a customer invoice under the same rate
movement, producing a loss — is section 28.

### 26.2 The bill, dated 1 March 2026

The rate date is the invoice date, so the lookup of section 7 returns one point ten and the stored
document rate is 1.10. The bill is for one thousand one hundred United States dollars of
consultancy, with no tax.

```formula
balance = round onto EUR( 1 100.00 ÷ 1.10 ) = round( 1 000.000000 ) = 1 000.00
```

Journal entry **B/2026/0004**, journal *Vendor Bills*, date 1 March 2026, document currency `USD`,
document rate 1.10:

| Line | Account | Debit `EUR` | Credit `EUR` | Amount in currency `USD` | Item currency |
|---|---|---|---|---|---|
| 1 | `6100 Consultancy` | 1 000.00 | | +1 100.00 | `USD` |
| 2 | `4400 Accounts Payable` | | 1 000.00 | −1 100.00 | `USD` |

Check of the invariants: the balances sum to zero; the two amounts in currency have the same sign
as their balances; the payable line's residuals are minus one thousand euro and minus one thousand
one hundred United States dollars.

### 26.3 The payment, dated 1 April 2026

The supplier is paid in full: one thousand one hundred United States dollars, from a bank journal
whose currency is `USD`. The rate in force on 1 April 2026 is one point twenty.

```formula
balance = round onto EUR( 1 100.00 ÷ 1.20 ) = round( 916.666667 ) = 916.67
```

Journal entry **BNK1/2026/0011**, journal *Bank United States dollar*, date 1 April 2026:

| Line | Account | Debit `EUR` | Credit `EUR` | Amount in currency `USD` | Item currency |
|---|---|---|---|---|---|
| 1 | `4400 Accounts Payable` | 916.67 | | +1 100.00 | `USD` |
| 2 | `5120 Bank United States dollar` | | 916.67 | −1 100.00 | `USD` |

### 26.4 Reconciling the two payable items

The debit side is the payment's payable item; the credit side is the bill's payable item.

**The residuals the debit side offers** (section 12.1): the company currency with a residual of
nine hundred sixteen point sixty-seven and a rate of one; and `USD` with a residual of one thousand
one hundred and an accounting rate of

```formula
| 1 100.00 ÷ 916.67 | = 1.199995636…
```

**The residuals the credit side offers**: the company currency with a residual of minus one
thousand and a rate of one; and `USD` with a residual of minus one thousand one hundred and an
accounting rate of

```formula
| −1 100.00 ÷ −1 000.00 | = 1.10
```

**The reconciliation currency** (section 13.1): the debit currency is `USD`, it is not the company
currency, and it appears in both maps — so the reconciliation currency is `USD`.

**The comparison:**

```formula
debit recon amount  = 1 100.00
credit recon amount = − ( −1 100.00 ) = 1 100.00
comparison          = compare at USD( 1 100.00 , 1 100.00 ) = 0
```

Both sides are therefore fully matched, and the minimum is one thousand one hundred.

**The partial amounts, branch B** (section 13.4):

```formula
debit interval  = interval( USD , EUR , 1 100.00 , 1 ÷ 1.199995636 )
                = ( round( 1 099.995 × 0.833336… ) , round( 1 100.000 × 0.833336… ) , round( 1 100.005 × 0.833336… ) )
                = ( 916.67 , 916.67 , 916.67 )
credit interval = interval( USD , EUR , 1 100.00 , 1 ÷ 1.10 )
                = ( round( 1 099.995 ÷ 1.10 ) , round( 1 100.000 ÷ 1.10 ) , round( 1 100.005 ÷ 1.10 ) )
                = ( 1 000.00 , 1 000.00 , 1 000.00 )

partial debit amount  = the smaller of ( 916.67   , 916.67   ) = 916.67
partial credit amount = the smaller of ( 1 000.00 , 1 000.00 ) = 1 000.00
matched amount        = the smaller of ( 916.67 , 1 000.00 )   = 916.67
```

**The tolerance band** does not apply: the partial debit amount of nine hundred sixteen point
sixty-seven is below the lowest value of the credit interval, one thousand, so the second of the
four conditions fails. The disagreement is economic, not a rounding artefact.

**The matched amounts in each side's currency:** neither currency is the company currency, so both
are the minimum:

```formula
matched amount in the debit currency  = 1 100.00
matched amount in the credit currency = 1 100.00
```

### 26.5 The exchange difference amount

The matching is measured in a foreign currency, so branch B of section 14.2 applies to each
fully-matched side.

Debit side, the payment's payable item:

```formula
debit exchange amount = remaining debit residual − matched amount = 916.67 − 916.67 = 0.00
```

Zero at the company currency, so **no** exchange line for the payment.

Credit side, the bill's payable item:

```formula
credit exchange amount = remaining credit residual + matched amount = −1 000.00 + 916.67 = −83.33
```

Not zero, so an exchange line is booked on the bill's payable item carrying a company-currency
repair of **minus eighty-three point thirty-three**. The remaining company-currency residual of the
credit side becomes minus nine hundred sixteen point sixty-seven.

### 26.6 The exchange difference journal entry

The repair amount is negative, so the counterpart account is the **gain exchange account** (section
15). The entry's date is the later of the two items' dates, 1 April 2026, subject to the exchange
journal's accounting-date rule of section 16.

Journal entry **MISC/2026/0007**, journal *Miscellaneous Operations*, date 1 April 2026, always
tax-exigible:

| Line | Account | Debit `EUR` | Credit `EUR` | Amount in currency `USD` | Item currency | Counterparty |
|---|---|---|---|---|---|---|
| 1 | `4400 Accounts Payable` | 83.33 | | 0.00 | `USD` | the supplier |
| 2 | `7760 Foreign Exchange Gain` | | 83.33 | 0.00 | `USD` | the supplier |

The derivation of each cell, from section 14 and the entry preparation of
[accounting-effects.md](accounting-effects.md) section 5:

- The repair amount is minus eighty-three point thirty-three, so on line one the debit is the
  negated repair, eighty-three point thirty-three, and the credit is zero.
- The item's currency differs from the company currency, so the repair in the document currency
  column is zero; line one's amount in currency is the negated zero and line two's is zero. Both
  lines still carry `USD` as their item currency, so that the entry does not accidentally
  re-express itself.
- Line two mirrors line one on the exchange account.
- Both lines carry the counterparty of the repaired item.
- Line one is marked as reconciling against the bill's payable item, which triggers a second
  matching between the two.

### 26.7 The resulting matching records

| Record | Debit item | Credit item | Matched amount `EUR` | In the debit currency `USD` | In the credit currency `USD` |
|---|---|---|---|---|---|
| Matching 1 | Exchange entry line 1 | Bill payable item | 83.33 | 0.00 | 0.00 |
| Matching 2 | Payment payable item | Bill payable item | 916.67 | 1 100.00 | 1 100.00 |

### 26.8 The residuals after reconciliation

| Item | Balance `EUR` | Matched `EUR` | Residual `EUR` | Amount in currency `USD` | Matched `USD` | Residual `USD` | Reconciled |
|---|---|---|---|---|---|---|---|
| Bill payable item | −1 000.00 | −83.33 − 916.67 | 0.00 | −1 100.00 | −1 100.00 | 0.00 | yes |
| Payment payable item | +916.67 | +916.67 | 0.00 | +1 100.00 | +1 100.00 | 0.00 | yes |
| Exchange entry line 1 | +83.33 | +83.33 | 0.00 | 0.00 | 0.00 | 0.00 | yes |

All three residuals are zero in both currencies, so a Full Reconciliation record is created and all
three items receive its matching number. The economic result is recorded correctly: the company
owed one thousand euro, paid nine hundred sixteen euro and sixty-seven cents, and recognised a gain
of eighty-three euro and thirty-three cents.

### 26.9 Arithmetic check

```formula
the bill in the company currency    = 1 100.00 ÷ 1.10 = 1 000.00
the payment in the company currency = 1 100.00 ÷ 1.20 =   916.67    rounded from 916.666667
the gain                            = 1 000.00 − 916.67 = 83.33
```

---

## 27. Worked example: a partial payment leaving residuals in both currencies

### 27.1 The setting

The same company, the same currencies and the same accounts as section 26, but the document is a
**customer invoice** and the payment is partial.

| Element | Value |
|---|---|
| Invoice date | 1 March 2026, rate of `USD` = 1.10 |
| Invoice amount | 1 100.00 `USD` |
| Payment date | 1 April 2026, rate of `USD` = 1.20 |
| Payment amount | **600.00 `USD`** — a partial settlement |

### 27.2 The invoice

```formula
balance = round onto EUR( 1 100.00 ÷ 1.10 ) = 1 000.00
```

Journal entry **INV/2026/0021**, journal *Customer Invoices*, date 1 March 2026:

| Line | Account | Debit `EUR` | Credit `EUR` | Amount in currency `USD` |
|---|---|---|---|---|
| 1 | `4000 Accounts Receivable` | 1 000.00 | | +1 100.00 |
| 2 | `7000 Product Sales` | | 1 000.00 | −1 100.00 |

### 27.3 The partial payment

```formula
balance = round onto EUR( 600.00 ÷ 1.20 ) = 500.00
```

Journal entry **BNK1/2026/0012**, date 1 April 2026:

| Line | Account | Debit `EUR` | Credit `EUR` | Amount in currency `USD` |
|---|---|---|---|---|
| 1 | `5120 Bank United States dollar` | 500.00 | | +600.00 |
| 2 | `4000 Accounts Receivable` | | 500.00 | −600.00 |

### 27.4 Matching

Debit side: the invoice's receivable item, residuals plus one thousand euro and plus one thousand
one hundred United States dollars, accounting rate one point ten. Credit side: the payment's
receivable item, residuals minus five hundred euro and minus six hundred United States dollars,
accounting rate one point twenty exactly.

Reconciliation currency: `USD`.

```formula
debit recon amount   = 1 100.00
credit recon amount  = 600.00
comparison           = compare at USD( 1 100.00 , 600.00 ) = +1
debit fully matched  = false
credit fully matched = true
minimum              = 600.00
```

Partial amounts, branch B:

```formula
debit interval  = ( round( 599.995 ÷ 1.10 ) , round( 600.000 ÷ 1.10 ) , round( 600.005 ÷ 1.10 ) )
                = ( 545.45 , 545.45 , 545.46 )
credit interval = ( round( 599.995 ÷ 1.20 ) , round( 600.000 ÷ 1.20 ) , round( 600.005 ÷ 1.20 ) )
                = ( 500.00 , 500.00 , 500.00 )

partial debit amount  = the smaller of ( 545.45 , 1 000.00 ) = 545.45
partial credit amount = the smaller of ( 500.00 ,   500.00 ) = 500.00
matched amount        = the smaller of ( 545.45 , 500.00 )   = 500.00
```

The tolerance band does not apply: five hundred forty-five point forty-five is above five hundred,
the highest value of the credit interval.

```formula
matched amount in the debit currency  = 600.00
matched amount in the credit currency = 600.00
```

### 27.5 The exchange difference — the partly-matched case

The debit side is **not** fully matched, so branch C of section 14.3 applies to it:

```formula
debit exchange amount = partial debit amount − matched amount = 545.45 − 500.00 = +45.45
compare at EUR( 45.45 , 0 ) = +1 > 0   →  the repair is booked
```

The credit side **is** fully matched, so branch B applies to it:

```formula
credit exchange amount = remaining credit residual + matched amount = −500.00 + 500.00 = 0.00   →  nothing booked
```

The repair amount on the invoice's receivable item is **positive** forty-five point forty-five, so
the counterpart account is the **loss exchange account**.

Journal entry **MISC/2026/0008**, journal *Miscellaneous Operations*, date 1 April 2026:

| Line | Account | Debit `EUR` | Credit `EUR` | Amount in currency `USD` | Item currency |
|---|---|---|---|---|---|
| 1 | `4000 Accounts Receivable` | | 45.45 | 0.00 | `USD` |
| 2 | `6560 Foreign Exchange Loss` | 45.45 | | 0.00 | `USD` |

### 27.6 The residuals — the point of the example

```formula
remaining debit residual             = 1 000.00 − 45.45 − 500.00 = 454.55
remaining debit residual in currency = 1 100.00 − 600.00         = 500.00
```

| Item | Residual `EUR` | Residual `USD` | Reconciled |
|---|---|---|---|
| Invoice receivable item | **454.55** | **500.00** | no — both residuals are non-zero |
| Payment receivable item | 0.00 | 0.00 | yes |
| Exchange entry line 1 | 0.00 | 0.00 | yes |

**Why the two residuals are consistent.** The invoice was recorded at a rate of one point ten. The
remaining five hundred United States dollars at that same rate are

```formula
round onto EUR( 500.00 ÷ 1.10 ) = round( 454.545455 ) = 454.55
```

which is exactly the remaining company-currency residual. That is what branch C of section 14.3 is
for: without the forty-five point forty-five repair, the remaining pair would have been five
hundred United States dollars against five hundred euro, implying a rate of one, and the next
partial payment would have produced nonsense.

### 27.7 What the second, closing payment then does

Suppose the remaining five hundred United States dollars are paid on 1 May 2026 at a rate of one
point twenty-five.

```formula
the balance of the second payment = round onto EUR( 500.00 ÷ 1.25 ) = 400.00
```

Matching: reconciliation currency `USD`; debit recon amount five hundred, credit recon amount five
hundred, comparison zero, both sides fully matched, minimum five hundred.

```formula
partial debit amount   = round( 500.00 × ( 1 ÷ 1.099989… ) ) = 454.55
partial credit amount  = round( 500.00 ÷ 1.25 )              = 400.00
matched amount         = 400.00
debit exchange amount  = 454.55 − 400.00 = +54.55    branch B, the debit side fully matched
credit exchange amount = −400.00 + 400.00 = 0.00
```

A second exchange entry debits the loss account by fifty-four euro and fifty-five cents and credits
the receivable by the same. The invoice's receivable item ends with residuals of zero in both
currencies and the whole chain closes. The total loss recognised across the two settlements is
forty-five point forty-five plus fifty-four point fifty-five, that is exactly one hundred euro —
the difference between the one thousand euro of revenue recognised and the nine hundred euro
actually collected, five hundred plus four hundred.

---

## 28. Worked example: the mirror case, producing a loss

For completeness, the customer-invoice version of section 26, settled in full.

| Element | Value |
|---|---|
| Invoice, 1 March 2026, rate 1.10 | receivable debit 1 000.00 `EUR` / +1 100.00 `USD` |
| Payment, 1 April 2026, rate 1.20 | receivable credit 916.67 `EUR` / −1 100.00 `USD` |

The debit side is now the **invoice**, the credit side the **payment**. Reconciliation currency
`USD`, minimum one thousand one hundred, both sides fully matched.

```formula
partial debit amount  = round( 1 100.00 ÷ 1.10 )          = 1 000.00
partial credit amount = round( 1 100.00 ÷ 1.199995636… )  =   916.67
matched amount        = the smaller of ( 1 000.00 , 916.67 ) = 916.67

debit exchange amount  = remaining debit residual − matched amount  = 1 000.00 − 916.67 = +83.33
credit exchange amount = remaining credit residual + matched amount =  −916.67 + 916.67 =   0.00
```

The repair amount is **positive**, so the counterpart is the **loss exchange account**:

| Line | Account | Debit `EUR` | Credit `EUR` | Amount in currency `USD` |
|---|---|---|---|---|
| 1 | `4000 Accounts Receivable` | | 83.33 | 0.00 |
| 2 | `6560 Foreign Exchange Loss` | 83.33 | | 0.00 |

The sign of the repair amount, and nothing else, chooses the account: **positive means loss,
negative means gain**.

---

## 29. Reconciliation notes

These notes record where the two independently written drafts of this document disagreed and which
statement was kept, after checking the behaviour against the system itself.

1. **The number of currencies whose rounding factor is one.** One draft said fifteen in its
   introduction and eighteen in its catalogue. Eighteen is correct: fourteen shipped currencies
   declare the factor as one and four declare it as one point zero zero, which is numerically
   identical. The comparison table at the head of this file and the count in
   [configuration.md](configuration.md) section 4.1 both say eighteen.
2. **The placement of the zero-width space beside a minus sign.** One draft placed it before the
   minus sign, the other after it. It follows the minus sign: the substitution replaces a minus
   sign by a minus sign and then the zero-width non-breaking space, so that the sign stays attached
   to the digits that follow. Section 24.2 states the corrected rule.
3. **Where the shipped currency catalogue lives.** Both drafts carried a complete table of the one
   hundred and seventy shipped currencies, one ordered alphabetically and one in load order, and
   the two agreed on every value. The catalogue is reference data rather than arithmetic, so it is
   published once, in load order and with the derived decimal places added, in
   [configuration.md](configuration.md) section 4.2; this file keeps only the summary by rounding
   factor it needs.
4. **Pseudo-code against procedures.** One draft expressed several algorithms as conditional
   blocks. Every algorithm in this file is now a numbered procedure or a formula block, with no
   construct borrowed from a programming language.
5. **The date of an exchange difference entry.** One draft raised the entry date only for the items
   that produce lines. The date is raised for every item submitted with the batch, including one
   whose instruction is afterwards skipped for being zero; section 16 step 3 states this, and
   [business-rules.md](business-rules.md) `MCUR-117` records it as a rule.
6. **The worked examples.** Both drafts carried end-to-end examples: one a bill settled at a higher
   rate with its mirror invoice case, the other an invoice collected at both a weaker and a
   stronger rate, plus the tolerance-band and tiny-rate cases. All of them are kept, in sections
   13.6 to 13.8, 14.6, 14.7 and 26 to 28, because each exercises a different branch of the
   algorithm.
