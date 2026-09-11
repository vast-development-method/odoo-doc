# Multi-Currency — Calculations

This file is the arithmetic core of the domain. It specifies, with no gaps:

- the single rounding routine every monetary amount passes through, written out as arithmetic
  including its error-compensation term;
- the five rounding methods and which one money uses;
- the zero test, the three-way comparison, the exact euclidean division, the string rendering
  and the digit split, each as arithmetic;
- the accurate reciprocal used so that dividing by a small fraction never loses a digit;
- the derivation of a currency's decimal places from its rounding factor;
- the rate lookup for a currency, a company and a date, with every fallback;
- the conversion of an amount between two currencies, and the cross-rate that routes two
  foreign currencies through the company currency in a single multiplication;
- the pair of amounts every journal item carries and the arithmetic that binds them;
- the residual arithmetic in both currencies;
- the choice of reconciliation currency, the partial amounts in all three currencies, and the
  tolerance band that suppresses a difference that is only a rounding artefact;
- the exchange difference amounts in every one of the four cases that can arise;
- the formatting of an amount for a human reader under a given language;
- the rendering of an amount in words;
- the complete catalogue of the one hundred and seventy shipped currencies with their symbol,
  rounding factor, derived decimal places and symbol position.

Formulas are written as plain mathematics. Quantities are named in words. Worked numeric
examples follow every formula.

**Agreement with the quantity side.** The rounding routine, the zero test and the comparison
specified in sections 2 to 4 are the *same* routines the quantity side of the platform uses, and
they are documented there from the quantity point of view in
[../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md).
Nothing in this file contradicts that one. The differences are only in what supplies the
precision and in which rounding method is the default at the point of call:

| | Quantity side | Money side |
|---|---|---|
| Where the precision comes from | The decimal precision record named `Product Unit`, shared by every unit of measure | The `rounding` (rounding factor) field of the currency in play |
| Typical precision value | One hundredth | One hundredth for most currencies, but also one, one thousandth and one ten-thousandth |
| Can the precision be a whole number? | No, in practice it is always a fraction | **Yes** — fifteen shipped currencies round onto multiples of one |
| Default method at the point of call | Away from zero, for quantity conversion | Half away from zero, everywhere |

---

## 1. Notation and shared definitions

### 1.1 Named quantities

| Name used in formulas | Meaning |
|---|---|
| rounding factor | The `rounding` field of a currency: the multiple every amount in that currency is snapped onto. |
| decimal places | The `decimal_places` field of a currency: the number of fractional digits used when rendering. Derived from the rounding factor. |
| company currency | The currency a company keeps its books in. |
| document currency | The currency a document was agreed in; the currency of a journal item. |
| technical rate | The `rate` field of a rate record: the value against the abstract reference of rate one. |
| conversion rate from A to B | How many units of currency B one unit of currency A buys, on a date, for a company. |
| balance | The amount of a journal item in the company currency, positive for a debit. |
| foreign amount | The amount of a journal item in its own currency, with the same sign as the balance. |
| residual | What is left to reconcile on a journal item. |

### 1.2 The five rounding methods

| Method | Stored selector | Behaviour |
|---|---|---|
| Half away from zero | `HALF-UP` | Round to the nearest multiple of the factor; a value exactly halfway goes away from zero. **This is the default, and the only method money uses unless a caller names another.** |
| Half towards zero | `HALF-DOWN` | Round to the nearest multiple; a value exactly halfway goes towards zero. |
| Half to even | `HALF-EVEN` | Round to the nearest multiple; a value exactly halfway goes to the nearer even multiple. Used by the language-aware presentation routine described in section 14. |
| Away from zero | `UP` | Always round away from zero: any non-zero fractional part pushes the result to the next multiple further from zero. |
| Towards zero | `DOWN` | Always round towards zero: the fractional part is discarded. |

Every monetary rounding in this domain — the rounding applied by a currency, the rounding
applied when converting, the rounding applied when computing a residual — uses **half away from
zero**. The only place another method appears is the presentation routine of section 14, whose
own default is half to even.

### 1.3 Deriving the decimal places from the rounding factor

```formula
decimal_places = ceiling( log10( 1 ÷ rounding_factor ) )    when 0 < rounding_factor < 1
decimal_places = 0                                          otherwise
```

The rounding factor is authoritative; the decimal places are a stored consequence of it. Writing
the rounding factor rewrites the decimal places. Nothing ever writes the decimal places alone.

Worked values are tabulated in [entities.md](entities.md) §1.4.

---

## 2. The rounding routine

### 2.1 Statement

Given a value, a precision and a rounding method, produce a rounded value. The precision is
supplied **either** as a number of fractional digits **or** as a rounding factor, never both and
never neither.

```formula
rounded_value = denormalize( apply_method( normalize( value ) ) )
```

### 2.2 Algorithm

1. **Resolve the factor.**
   - If a rounding factor was supplied and a digit count was not, assert that the factor is
     strictly greater than zero and use it.
   - If a digit count was supplied and a factor was not, assert that the digit count is a whole
     number greater than or equal to zero, and set the factor to ten raised to minus that
     count.
   - If both were supplied, or neither, fail. The failure is a programming error, not a user
     error, and must be loud.

   ```formula
   rounding_factor = 10 ^ ( − precision_digits )
   ```

2. **Short-circuit.** If the factor is zero, or the value is zero, return zero.

3. **Choose the scaling direction.** Define two operations, *normalise* and *denormalise*:
   - normalise divides by the factor; denormalise multiplies by the factor;
   - **but if the factor is strictly smaller than one**, replace the factor by its accurately
     inverted value (section 2.4) and **swap** the two operations, so that normalise multiplies
     by the inverted factor and denormalise divides by it.

   The purpose is to replace a division by a small fraction — which loses accuracy — by a
   multiplication by a large whole number, which does not. With a factor of one hundredth,
   normalising multiplies by one hundred. With a factor of one (a currency with no subunit) the
   factor is not smaller than one, so no swap happens and normalising divides by one, which is a
   no-operation.

4. **Normalise.** Apply the normalise operation to the value. The result is the value counted in
   whole rounding steps, plus a fraction.

5. **Compute the compensation term.** Let the magnitude be the base-two logarithm of the
   absolute normalised value.

   ```formula
   epsilon = 2 ^ ( log2( | normalized_value | ) − 50 )
   ```

   The minimal term that repairs a single unit in the last place of a double-precision number
   would use fifty-two rather than fifty. Fifty is used deliberately, so that error accumulated
   over several prior floating-point operations is also absorbed. A rebuild that uses fifty-two,
   or omits the term altogether, produces different answers on values such as two point six
   seven five, whose binary representation lies very slightly below the tie. See the worked
   example in section 2.5.

6. **Apply the method** to the normalised value:
   - **half away from zero:** add the compensation term carrying the sign of the normalised
     value, then round half away from zero to a whole number;
   - **half to even:** take the whole part below (the floor); take the absolute difference
     between the normalised value and that floor; if that difference is within the compensation
     term of one half, the value is a tie, and the result is the floor plus one when the floor is
     odd, otherwise the floor; if it is not a tie, round half away from zero to a whole number;
   - **half towards zero:** subtract the compensation term carrying the sign of the normalised
     value, then round half away from zero to a whole number;
   - **away from zero:** add, carrying the sign of the normalised value, the quantity one minus
     the compensation term, then truncate towards zero;
   - **towards zero:** add the compensation term carrying the sign of the normalised value, then
     truncate towards zero.

   Any other method is a programming error and must fail with a message naming the unknown
   method.

7. **Denormalise.** Apply the denormalise operation to the whole-number result and return it.

### 2.3 "Round half away from zero to a whole number"

The rounding to a whole number used inside step 6 is **not** the round-half-to-even that most
language runtimes provide as their default. It is defined as:

1. Take the runtime's nearest-integer result for the value.
2. Take the runtime's nearest-integer result for the value plus one.
3. If the second minus the first is **not exactly one**, the value was a tie that the runtime
   resolved to even; return instead the value plus one half carrying the sign of the value.
4. Otherwise return the result of step 1, carrying the sign of the value.

Carrying the sign exists for two reasons: so that rounding a negative zero yields a negative zero
rather than a positive one, and so that the result is a real number rather than a whole number,
which keeps the later multiplication in the real domain.

### 2.4 The accurate reciprocal

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

For a factor not in the table, the reciprocal is computed as: render the factor in scientific
notation with fifteen digits after the point, splitting it into a coefficient and an exponent;
build the number with the same coefficient and the negated exponent; divide that by the square of
the coefficient.

```formula
inverted_factor = ( coefficient × 10 ^ ( − exponent ) ) ÷ coefficient²
```

### 2.5 Worked example — rounding two point six seven five to two decimal places

This is the canonical case and a rebuild must reproduce it exactly.

**Given** the value two point six seven five and a currency whose rounding factor is one
hundredth.

1. The factor one hundredth is strictly smaller than one, so it is inverted to exactly one
   hundred, and the two operations are swapped: normalising **multiplies** by one hundred.
2. Normalised value: two point six seven five times one hundred. In binary double precision the
   literal two point six seven five is stored as two point six seven four nine nine nine nine
   nine nine nine nine nine nine nine eight. Multiplied by one hundred this is two hundred
   sixty-seven point four nine nine nine nine nine nine nine nine nine nine nine nine seven.
3. Magnitude: the base-two logarithm of two hundred sixty-seven point five is about eight point
   zero six four. The compensation term is therefore two raised to the power of minus forty-one
   point nine three six, which is about two point three eight times ten to the minus thirteen.
4. Method half away from zero: add the compensation term, giving two hundred sixty-seven point
   five zero zero zero zero zero zero zero zero zero zero zero two four, which is now strictly
   above the tie.
5. Round half away from zero to a whole number: two hundred sixty-eight.
6. Denormalise: divide by one hundred. **Result: two point six eight.**

**Without** the compensation term, step 4 would leave two hundred sixty-seven point four nine nine
nine…, step 5 would give two hundred sixty-seven, and the result would be two point six seven —
one cent lower. Every downstream total would then be one cent out. This is why the term is not
optional.

### 2.6 Further worked examples of the rounding routine

| Value | Rounding factor | Method | Result | Why |
|---|---|---|---|---|
| 2.675 | 0.01 | half away from zero | 2.68 | Section 2.5. |
| −2.675 | 0.01 | half away from zero | −2.68 | Ties go away from zero in both directions. |
| 2.675 | 0.01 | half towards zero | 2.67 | The term is subtracted, pushing the value below the tie. |
| 2.675 | 0.01 | half to even | 2.68 | The tie is detected; the floor two hundred sixty-seven is odd, so one is added. |
| 2.665 | 0.01 | half to even | 2.66 | The tie is detected; the floor two hundred sixty-six is even, so it stands. |
| 2.671 | 0.01 | away from zero | 2.68 | Any non-zero fraction of a step pushes away from zero. |
| 2.679 | 0.01 | towards zero | 2.67 | The fractional part is discarded. |
| 1234.5 | 1 | half away from zero | 1235 | A currency with no subunit. The factor is not smaller than one, so normalising divides by one and the tie is broken away from zero. |
| 1234.4 | 1 | half away from zero | 1234 | |
| 0.12345 | 0.001 | half away from zero | 0.123 | A three-decimal currency. |
| 0.12355 | 0.001 | half away from zero | 0.124 | |
| 0.123456 | 0.0001 | half away from zero | 0.1235 | A four-decimal currency. |
| 1234.56 | 5 | half away from zero | 1235 | A hypothetical currency rounding onto multiples of five. Normalising divides by five giving two hundred forty-six point nine one two; the nearest whole number is two hundred forty-seven; denormalising multiplies by five. |
| −0.004 | 0.01 | half away from zero | −0.0 | The result is a negative zero, and the presentation routine strips the sign (section 14.4). |

---

## 3. The zero test

An amount is treated as zero when, after being rounded onto the currency's factor, its absolute
value is strictly smaller than the factor.

```formula
is_zero( amount ) = ( amount = 0 )  OR  ( | round( amount, rounding_factor ) | < rounding_factor )
```

Worked values at a factor of one hundredth:

| Amount | Rounded | Absolute value | Smaller than 0.01? | Treated as zero |
|---|---|---|---|---|
| 0 | — | — | — | yes (short-circuit) |
| 0.004 | 0.00 | 0.00 | yes | yes |
| 0.005 | 0.01 | 0.01 | no | no |
| −0.004 | −0.00 | 0.00 | yes | yes |
| 0.0049999 | 0.00 | 0.00 | yes | yes |
| 0.01 | 0.01 | 0.01 | no | no |

**The warning that matters.** Testing the difference of two amounts for zero is *not* the same as
comparing them. Six thousandths minus two thousandths is four thousandths, which the zero test
calls zero; but the comparison of section 4 calls six thousandths and two thousandths *different*,
because they round to one hundredth and zero respectively. Both behaviours are intentional and
both are used in the domain: the zero test is used to decide whether a residual is exhausted, the
comparison is used to decide whether a payment covers an invoice.

---

## 4. The three-way comparison

```formula
compare( amount_one , amount_two ) =
     0   when amount_one = amount_two exactly
     0   when round( amount_one ) − round( amount_two ) is zero at the same factor
    −1   when round( amount_one ) − round( amount_two ) < 0
    +1   otherwise
```

Algorithm:

1. Resolve the factor exactly as in section 2.2 step 1.
2. If the two amounts are bit-for-bit equal, return zero at once. (This is a short-circuit taken
   *after* the precision has been validated, so that a bad precision still fails loudly.)
3. Round each amount onto the factor, using half away from zero.
4. Take the difference of the two rounded amounts.
5. If the difference passes the zero test of section 3 at the same factor, return zero.
6. Return minus one when the difference is negative, plus one otherwise.

### 4.1 Mandatory worked example — comparison at a precision of one hundredth

| First amount | Second amount | First rounded | Second rounded | Difference | Zero? | Result | Reading |
|---|---|---|---|---|---|---|---|
| 1.432 | 1.431 | 1.43 | 1.43 | 0.00 | yes | **0** | Equal at one hundredth, even though the raw values differ by one thousandth. |
| 0.006 | 0.002 | 0.01 | 0.00 | 0.01 | no | **+1** | Different at one hundredth, even though the raw values differ by only four thousandths — less than the precision. This is the case that catches every rebuild that compares by subtracting first. |
| 0.002 | 0.006 | 0.00 | 0.01 | −0.01 | no | **−1** | The mirror of the previous row. |
| 1.005 | 1.00 | 1.01 | 1.00 | 0.01 | no | **+1** | The tie in the first amount is resolved away from zero by the compensation term. |
| 1.004 | 1.00 | 1.00 | 1.00 | 0.00 | yes | **0** | |
| 100.00 | 100.00 | — | — | — | — | **0** | Taken by the exact-equality short-circuit in step 2 without any rounding. |
| −0.006 | 0.002 | −0.01 | 0.00 | −0.01 | no | **−1** | |
| 2.675 | 2.68 | 2.68 | 2.68 | 0.00 | yes | **0** | Because the first value rounds up, as section 2.5 established. |

Ordering consequence: the comparison is a genuine three-way ordering on rounded values. A rebuild
must **not** implement it as "the sign of the difference", and must **not** implement it as
"compare after subtracting". Round first, subtract second.

---

## 5. Exact euclidean division

Some flows need to split an amount into whole multiples of another amount without the
representation error that a native division and remainder would carry. The routine returns a
whole-number quotient and a remainder.

Algorithm:

1. Resolve the factor as in section 2.2 step 1.
2. Round the dividend onto the factor, divide by the factor, and round that to the nearest whole
   number. Call the result the scaled dividend.
3. Do the same to the divisor. Call the result the scaled divisor.
4. Take the whole-number quotient and remainder of the scaled dividend by the scaled divisor.
5. Multiply the remainder by the factor and round it onto the factor.
6. Return the quotient (a whole number) and that remainder.

```formula
scaled_dividend = nearest_whole( round( dividend , factor ) ÷ factor )
scaled_divisor  = nearest_whole( round( divisor  , factor ) ÷ factor )
quotient        = whole_part( scaled_dividend ÷ scaled_divisor )
remainder       = round( ( scaled_dividend − quotient × scaled_divisor ) × factor , factor )
```

Postcondition: the dividend, rounded onto the factor, equals the quotient times the divisor plus
the remainder, exactly, at that factor.

Worked example. Dividend two hundred sixty-eight point five zero, divisor twelve point two five,
factor one hundredth. Scaled dividend twenty-six thousand eight hundred fifty; scaled divisor one
thousand two hundred twenty-five. Quotient twenty-one; twenty-one times one thousand two hundred
twenty-five is twenty-five thousand seven hundred twenty-five; remainder one thousand one hundred
twenty-five, which denormalises to eleven point two five. Check: twenty-one times twelve point
two five is two hundred fifty-seven point two five, plus eleven point two five is two hundred
sixty-eight point five zero. Correct.

---

## 6. Rendering a number as a string

```formula
string_form = decimal_string( value , decimal_places )
```

Algorithm:

1. If the value passes the zero test at the given number of digits, replace it by zero. This
   removes a negative zero and prevents a string of the form minus zero point zero zero.
2. Render the value with **exactly** that many digits after the decimal point, padding with
   zeros, using a full stop as the decimal point and no digit grouping.

The routine is a *presentation* routine, not a rounding routine: a rebuild must round first and
render second, never rely on the renderer to round. The renderer must not use a
shortest-representation conversion, because such conversions silently drop significant digits on
large values.

### 6.1 Splitting into whole and fractional parts

```formula
whole_part , fractional_part = split( value , decimal_places )
```

1. Round the value onto the given number of digits.
2. Render it as in section 6.
3. Split the rendered string at the decimal point.
4. When the number of digits is zero, the fractional part is the empty string.

The fractional part always has exactly the requested number of characters, padded with trailing
zeros. Worked values: one point four three two at two digits gives one and forty-three; one point
four nine at one digit gives one and five; one point one at three digits gives one and one
hundred; one point one two at zero digits gives one and the empty string.

A companion form returns the two parts as whole numbers instead of strings; when the number of
digits is zero the fractional part is the whole number zero.

### 6.2 Rendering for structured interchange

When an amount must be written into a structured interchange document, a variant is used that
rounds, renders to a string, and then reads the string back as a number. The point is that the
resulting number's own shortest representation is the rendered string, so that a serialiser which
cannot be told how to format numbers still emits the intended digits. The value that comes back
must **not** be used for further arithmetic.

---

## 7. The rate lookup

### 7.1 Statement

Given a set of currencies, a company and a date, produce for each currency a single number: its
technical rate in force.

### 7.2 Algorithm

For each currency:

1. **Resolve the company.** Take the **root** company of the company supplied. Rates never live
   on a branch.
2. **Primary lookup.** Among the rate records of this currency whose date is **on or before** the
   requested date and whose company is either the root company or empty, order by company
   first — records that name the company sort before records that name no company — and then by
   date **descending**. Take the first record's technical rate.
3. **Fallback lookup.** If there is no such record, consider the rate records of this currency
   whose company is either the root company or empty, **without any date restriction**. Order by
   company first, then by date **ascending**. Take the first record's technical rate. This means:
   when every rate of a currency is in the future relative to the requested date, the earliest
   future rate is used rather than nothing.
4. **Final fallback.** If the currency has no rate record at all, the rate is **one**.

The company preference in steps 2 and 3 is total: *any* company-specific record that satisfies the
date condition beats *every* shared record, even a shared record with a later date. A rebuild
must not merge the two sets and then order by date.

### 7.3 Worked example of the lookup

Rate records for the currency `USD` (United States dollar), with a root company called Northwind
and another root company called Southgate:

| Record | Date | Company | Technical rate |
|---|---|---|---|
| A | 1 January 2026 | (shared) | 1.0500 |
| B | 1 March 2026 | (shared) | 1.1000 |
| C | 15 February 2026 | Northwind | 1.0800 |
| D | 1 June 2026 | Northwind | 1.2000 |

| Requested date | Requested company | Records passing the date filter | Winner | Rate |
|---|---|---|---|---|
| 20 February 2026 | Northwind | A, C | C — it names the company | 1.0800 |
| 20 February 2026 | Southgate | A | A | 1.0500 |
| 15 March 2026 | Northwind | A, B, C | C — company-specific beats the later shared record B | 1.0800 |
| 15 March 2026 | Southgate | A, B | B — later of the two shared records | 1.1000 |
| 1 July 2026 | Northwind | A, B, C, D | D | 1.2000 |
| 1 December 2025 | Northwind | none | Fallback: earliest of A, C, D preferring company-specific → C | 1.0800 |
| 1 December 2025 | Southgate | none | Fallback: earliest shared → A | 1.0500 |

Row five of the table is the one that surprises people: in March, Northwind still uses its own
February rate, because a company-specific record is preferred unconditionally.

### 7.4 The derived current rate of a currency

The current rate exposed on a currency is computed against a **target** currency, which defaults
to the company currency:

```formula
current_rate = ( rate_of_this_currency  or  1 ) ÷ rate_of_the_target_currency
inverse_rate = 1 ÷ current_rate
```

Both rates come from the lookup of section 7.2 at the effective date. Three context values steer
the computation: the target currency, the date, and the company. The date defaults to today in
the reader's time zone.

The human-readable form is built as:

```formula
rate_string = "1 " + target_currency_code + " = " + current_rate rendered with exactly six decimal digits + " " + this_currency_code
```

and is **empty** when this currency is the effective company's currency.

---

## 8. Conversion between two currencies

### 8.1 The conversion rate

```formula
conversion_rate( from_currency , to_currency , company , date ) = 1
        when from_currency = to_currency

conversion_rate( from_currency , to_currency , company , date )
      = rate_of( to_currency , root_company , date ) ÷ rate_of( from_currency , root_company , date )
        otherwise
```

where *rate of* is the lookup of section 7.2. The date defaults to today in the reader's time
zone; the company defaults to the company the reader is acting for, and is immediately replaced
by its root.

Direction check, worth stating because it is the most commonly inverted thing in a rebuild: the
conversion rate **from the company currency to a foreign currency** is the foreign currency's
rate divided by the company currency's rate. When the company currency has no rate records — the
ordinary case — its rate is one, and the conversion rate from the company currency to a foreign
currency is simply that foreign currency's technical rate. So a technical rate of one point ten
on `USD` in a company reporting in `EUR` (the euro) means **one euro buys one point ten United
States dollars**.

### 8.2 The conversion of an amount

```formula
converted_amount = round_to( to_currency ,  from_amount × conversion_rate( from_currency , to_currency , company , date ) )
```

Algorithm:

1. If the source currency is empty, substitute the destination currency; if the destination
   currency is empty, substitute the source. If both are empty, fail — an amount cannot be
   converted from an unknown currency.
2. If the amount is exactly zero, return zero without consulting any rate.
3. Multiply the amount by the conversion rate.
4. Round the product onto the **destination** currency's rounding factor, using half away from
   zero — unless the caller explicitly asked for an unrounded result, in which case the raw
   product is returned.

**One multiplication, one rounding.** Even when neither currency is the company currency, the
conversion performs a single multiplication by the composed cross-rate, and rounds once. It does
**not** convert to the company currency, round, and convert again.

### 8.3 Mandatory worked example — conversion between two foreign currencies

**Given** a company reporting in `EUR` (the euro). On 15 March 2026 the technical rates in force
for that company are:

| Currency | Technical rate |
|---|---|
| `EUR` (euro) | 1.0000 — no rate records exist, so the final fallback of one applies |
| `USD` (United States dollar) | 1.1723 |
| `GBP` (pound sterling) | 0.8391 |

**When** five hundred United States dollars are converted to pounds sterling on that date.

Step 1 — the cross-rate is composed in a single division:

```formula
conversion_rate( USD , GBP ) = rate_of( GBP ) ÷ rate_of( USD ) = 0.8391 ÷ 1.1723 = 0.715772413204811…
```

Step 2 — one multiplication:

```formula
raw_product = 500.00 × 0.715772413204811… = 357.886206602405…
```

Step 3 — one rounding, onto the destination currency's factor of one hundredth:

**Result: three hundred fifty-seven pounds and eighty-nine pence.**

**Now the wrong way, for contrast.** A rebuild that routes through the company currency in two
rounded steps gets a different answer:

```formula
step_one  = round_to( EUR , 500.00 × ( 1 ÷ 1.1723 ) ) = round( 426.511984…) = 426.51
step_two  = round_to( GBP , 426.51 × 0.8391 )         = round( 357.884…)    = 357.88
```

Three hundred fifty-seven pounds and eighty-eight pence — one penny short. The single-step
composition is the specified behaviour; the two-step route is not. The divergence is not rare: at
these rates it appears for roughly one amount in five.

A second amount at the same rates, showing the same divergence at a larger magnitude:

| Amount in `USD` | One step (specified) | Two steps (wrong) |
|---|---|---|
| 100.00 | 71.58 | 71.58 — agree |
| 250.55 | 179.34 | 179.34 — agree |
| **500.00** | **357.89** | 357.88 — **differ** |
| 777.77 | 556.71 | 556.71 — agree |
| 1 234.56 | 883.66 | 883.66 — agree |
| **4 321.09** | **3 092.92** | 3 092.91 — **differ** |

### 8.4 Conversion when the amount is zero or the currencies are equal

| Situation | Behaviour |
|---|---|
| Source and destination are the same currency | The rate is one without any lookup. The amount is still rounded onto the currency's factor. |
| The amount is exactly zero | Zero is returned immediately. No rate lookup happens, so a conversion of zero never fails for want of a rate. |
| Rounding suppressed by the caller | The raw product is returned. Used where the result feeds a further computation that will round later. |

### 8.5 Conversions that round to zero

Because the result is rounded onto the destination factor, a small amount in a fine-grained
currency can vanish when converted into a coarse one.

| Amount | From | To | Rate | Raw product | Rounded | Note |
|---|---|---|---|---|---|---|
| 0.004 | `EUR`, factor one hundredth | `USD`, factor one hundredth | 1.1723 | 0.00468… | 0.00 | Vanishes. |
| 0.40 | `EUR` | `JPY` (Japanese yen), factor one | 163.5 | 65.4 | 65 | Does not vanish, but loses the fraction. |
| 0.002 | `EUR` | `JPY`, factor one | 163.5 | 0.327 | 0 | Vanishes. |
| 1.00 | `JPY`, factor one | `EUR` | 0.006116 | 0.006116 | 0.01 | Rounds **up** from almost nothing, because half away from zero applied to zero point six one one six of a step gives one step. |

---

## 9. The two amounts of a journal item

### 9.1 The binding formulas

```formula
foreign_amount = round_to( item_currency  , balance        × item_rate )
balance        = round_to( company_currency , foreign_amount ÷ item_rate )
```

The rate on the item, called the item rate here, is the conversion rate **from the company
currency to the item's currency**. It is derived as:

1. When the item's entry is invoice-like (a customer invoice, a credit note, a vendor bill, a
   refund or a receipt), the item rate is the entry's stored document rate; when that is zero,
   it is one.
2. Otherwise, when the item has a currency, the item rate is the conversion rate from the
   company currency to that currency, for the item's company, at the first of: the entry's
   invoice date, the entry's date, today in the reader's time zone.
3. Otherwise the item rate is one.

### 9.2 Which of the two is derived

| Caller supplied | Derived |
|---|---|
| Balance only | Foreign amount, by the first formula. |
| Foreign amount only | Balance, by the second formula. |
| Both | Both are kept as supplied, subject to the two forcings below. |
| Neither | Both remain zero, subject to the two forcings below. |

Two forcings override the above:

- When the item's currency equals the company currency **and** the entry is **not** invoice-like,
  the foreign amount is forced equal to the balance. There is then no rate arithmetic at all.
- On an invoice-like entry, a change to the foreign amount, to the item rate or to the document
  type re-derives the balance by the second formula, even when a balance was supplied.

### 9.3 Worked example — an invoice line in a foreign currency

**Given** a company reporting in `EUR`, a customer invoice in `USD` dated 15 March 2026, and a
stored document rate of one point ten (one euro buys one point ten United States dollars). A
product line has a subtotal of one thousand one hundred United States dollars.

```formula
balance = round_to( EUR , 1 100.00 ÷ 1.10 ) = round( 1 000.000000 ) = 1 000.00
```

The revenue line is therefore a credit of one thousand euros with a foreign amount of minus one
thousand one hundred United States dollars, and the receivable line a debit of one thousand euros
with a foreign amount of plus one thousand one hundred United States dollars.

At a document rate of one point one seven two three instead:

```formula
balance = round_to( EUR , 1 100.00 ÷ 1.1723 ) = round( 938.326367… ) = 938.33
```

### 9.4 The sign invariant as arithmetic

```formula
( balance ≤ 0  AND  foreign_amount ≤ 0 )   OR   ( balance ≥ 0  AND  foreign_amount ≥ 0 )
```

must hold for every item that is not a section, a subsection or a note. Because the item rate is
always strictly positive, the derivation formulas preserve the sign automatically; the invariant
bites only when both amounts are written independently.

---

## 10. Residual arithmetic

For an item on a reconcilable account, or on an account of type cash or credit card:

```formula
matched_as_debit_company   = Σ over partials where this item is the debit side  of ( partial.amount )
matched_as_debit_foreign   = round_to_places( Σ over those partials of ( partial.debit_amount_currency ) , decimal_places_of_the_debit_currency )
matched_as_credit_company  = Σ over partials where this item is the credit side of ( partial.amount )
matched_as_credit_foreign  = round_to_places( Σ over those partials of ( partial.credit_amount_currency ) , decimal_places_of_the_credit_currency )

amount_residual          = round_to( company_currency , balance        − matched_as_debit_company + matched_as_credit_company )
amount_residual_currency = round_to( item_currency    , foreign_amount − matched_as_debit_foreign + matched_as_credit_foreign )

reconciled = is_zero_in( company_currency , amount_residual )  AND  is_zero_in( item_currency , amount_residual_currency )
```

Two details a rebuild must not skip:

1. The foreign sums are rounded **inside** the aggregation, to the number of decimal places of
   the currency concerned, before being subtracted. A rebuild that sums first and rounds at the
   end can differ by one unit in the last place on a chain of many partials.
2. The item is reconciled only when **both** residuals are zero. An item can have a zero residual
   in the company currency and a non-zero residual in its own currency, or the reverse; it is not
   reconciled in either case, and this is exactly the situation the exchange difference entry
   exists to repair.

Items on accounts that are neither reconcilable nor cash-like have both residuals forced to zero
and the reconciled flag forced to false.

---

## 11. Choosing the reconciliation currency

Two items are matched in **one** currency, chosen as follows.

### 11.1 The available residuals of one item

For one item, against a counterpart whose currency is known, build a map from currency to a pair
of numbers — a residual and a rate:

1. Let the remaining company-currency amount and the remaining foreign amount be the item's two
   residuals as they stand at this point in the matching loop (they are decremented as the loop
   proceeds, not re-read from storage).
2. If the remaining company-currency amount is **not** zero, add an entry for the **company
   currency** with that residual and a rate of one.
3. If the item's currency differs from the company currency and the remaining foreign amount is
   **not** zero, add an entry for the **item's currency** with that residual and the item's
   *accounting rate* (section 11.2).
4. If the item's currency **equals** the company currency, the account is a receivable or a
   payable, the remaining company-currency amount is not zero, and the counterpart's currency is
   **not** the company currency, then the item is allowed to "mimic" the counterpart's currency:
   compute a residual in the counterpart currency as the remaining company-currency amount times
   the *platform rate* (section 11.3), rounded onto the counterpart currency; if that is not
   zero, add an entry for the counterpart currency with that residual and that rate.
5. Otherwise, if the item's currency **is** the counterpart currency and differs from the company
   currency and the remaining foreign amount is not zero, add an entry for the counterpart
   currency with that residual and the item's accounting rate.

### 11.2 The accounting rate of an item

```formula
accounting_rate = | foreign_amount ÷ balance |
```

computed from the item's own stored pair, and **undefined** when either the balance is zero at
the company currency's precision or the foreign amount is zero at the item currency's precision.
This is the rate the item was actually booked at, which may differ from any rate in the rate
table — for instance because a user overrode the document rate.

### 11.3 The platform rate for a mimicking item

Used only in step 4 above. Resolved as the first of:

1. A rate forced by the payment registration flow, when one is supplied.
2. When the counterpart is a payment or a bank statement line and this item is not, the
   counterpart's **accounting rate** in the counterpart currency. The payment's booked rate then
   governs, which is what makes a payment in a foreign currency settle an invoice in the company
   currency at the payment's own rate.
3. Otherwise the conversion rate from the company currency to the counterpart currency, for the
   item's company, at the item's **invoice date** when the item's entry is invoice-like, and at
   the item's date otherwise.

### 11.4 Selecting the currency

Let the debit item's currency and the credit item's currency be known, and let both maps be
built. Then:

1. If the **debit** currency is not the company currency **and** appears in both maps, the
   reconciliation currency is the debit currency.
2. Otherwise, if the **credit** currency is not the company currency **and** appears in both
   maps, the reconciliation currency is the credit currency.
3. Otherwise the reconciliation currency is the **company currency**.

If either map lacks the chosen currency the pairing is abandoned: the side that lacks it is
marked as having nothing left, and the loop advances to the next item on that side.

### 11.5 The exchange-line special case

A flag called here *exchange-line mode* is raised when **all three** of the following hold:

- the reconciliation currency is the company currency; **and**
- the debit item and the credit item have the **same** currency; **and**
- at least one of the two maps has **no** entry for that shared currency.

In exchange-line mode both rates are treated as undefined, so that the match consumes only
company-currency amounts and leaves the foreign amounts untouched. This is precisely the shape of
an exchange difference line being matched back against the item it repairs: the exchange line has
a company-currency amount and a foreign amount of zero.

---

## 12. The partial amounts

Let the chosen reconciliation currency be known. Write:

- *debit recon amount* — the debit item's residual in the reconciliation currency;
- *credit recon amount* — the **negated** credit item's residual in the reconciliation currency,
  so that both are positive;
- *minimum* — the smaller of the two.

```formula
comparison = compare_in( reconciliation_currency , debit_recon_amount , credit_recon_amount )
debit_fully_matched  = ( comparison ≤ 0 )
credit_fully_matched = ( comparison ≥ 0 )
minimum = min( debit_recon_amount , credit_recon_amount )
```

When the comparison is zero both sides are fully matched.

### 12.1 Case A — the reconciliation currency is the company currency

```formula
partial_amount = minimum

partial_debit_amount_currency  = min( round_to( debit_currency  , debit_rate  × minimum ) , remaining_debit_foreign  )     when a debit rate exists, else 0
partial_credit_amount_currency = min( round_to( credit_currency , credit_rate × minimum ) , − remaining_credit_foreign )   when a credit rate exists, else 0
```

where the debit rate and the credit rate are the accounting rates from each item's own map, and
both are treated as absent in exchange-line mode.

### 12.2 Case B — the reconciliation currency is a foreign currency

Here the minimum is expressed in the foreign currency and must be translated into the company
currency once for each side, using each side's own rate. Because each translated value is a
rounded number, it stands for a small interval of possible true values, and the routine computes
that interval explicitly.

```formula
interval( currency_from , currency_to , amount , rate ) =
    (   round_to( currency_to , ( amount − rounding_factor_of( currency_from ) ÷ 2 ) × rate ) ,
        round_to( currency_to ,   amount                                              × rate ) ,
        round_to( currency_to , ( amount + rounding_factor_of( currency_from ) ÷ 2 ) × rate )  )
```

with the convention that an absent rate yields the interval of three zeros. Then:

```formula
debit_interval  = interval( debit_currency  , company_currency , minimum , 1 ÷ debit_rate  )
credit_interval = interval( credit_currency , company_currency , minimum , 1 ÷ credit_rate )

partial_debit_amount  = min( middle_of( debit_interval  ) ,   remaining_debit_company  )
partial_credit_amount = min( middle_of( credit_interval ) , − remaining_credit_company )
partial_amount        = min( partial_debit_amount , partial_credit_amount )
```

**The tolerance band.** If each side's chosen company-currency amount falls inside the *other*
side's interval, the two sides disagree only by a rounding artefact, and forcing an exchange
difference would be wrong. The condition, all four parts of which must hold:

```formula
partial_debit_amount  ≤ highest_of( credit_interval )   AND
partial_debit_amount  ≥ lowest_of(  credit_interval )   AND
partial_credit_amount ≤ highest_of( debit_interval  )   AND
partial_credit_amount ≥ lowest_of(  debit_interval  )
```

each comparison being the three-way comparison of section 4 at the company currency. When it
holds, the three amounts are collapsed:

```formula
partial_amount = min( remaining_debit_company , − remaining_credit_company )
partial_debit_amount  = partial_amount
partial_credit_amount = partial_amount
```

and no exchange difference arises from the rounding.

Finally the foreign amounts of the partial:

```formula
partial_debit_amount_currency  = partial_amount   when the debit currency is the company currency, else minimum
partial_credit_amount_currency = partial_amount   when the credit currency is the company currency, else minimum
```

### 12.3 Worked example of the tolerance band

**Given** a company reporting in a currency whose factor is one hundredth, a debit item with a
balance of three hundred seventy-seven thousand five hundred fifty-four and a foreign amount of
twenty thousand, and a credit item with a balance of minus five thousand three hundred fourteen
point six two and a foreign amount of minus two hundred eighty-one point five three, both in the
same foreign currency.

The reconciliation currency is the foreign currency. The minimum is two hundred eighty-one point
five three.

- The debit item's accounting rate is twenty thousand divided by three hundred seventy-seven
  thousand five hundred fifty-four, that is zero point zero five two nine seven two five five…
  Its reciprocal times the minimum is five thousand three hundred fourteen point six four, and
  the interval around it, widened by half a hundredth of the foreign currency, runs from five
  thousand three hundred fourteen point five four to five thousand three hundred fourteen point
  seven three.
- The credit item's own booked value for the same two hundred eighty-one point five three is
  five thousand three hundred fourteen point six two, with an interval from five thousand three
  hundred fourteen point five three to five thousand three hundred fourteen point seven one.

Each side's middle value lies inside the other's interval, so the band applies: the partial is
booked at five thousand three hundred fourteen point six two in the company currency on both
sides, and **no exchange difference entry is produced**. Without the band, a two-cent difference
would be posted for no economic reason.

---

## 13. The exchange difference amounts

After the partial amounts are known, the routine decides whether either side is left with an
inconsistency, and if so by how much. There are exactly four cases.

### 13.1 Case A — reconciliation in the company currency

Only fully-matched sides are considered, and the repair is made in the **foreign** amount.

```formula
when debit_fully_matched:
    debit_exchange_amount = remaining_debit_foreign − partial_debit_amount_currency
    if not is_zero_in( debit_currency , debit_exchange_amount ):
        book an exchange line on the debit item carrying a foreign-amount repair of debit_exchange_amount
        remaining_debit_foreign = remaining_debit_foreign − debit_exchange_amount

when credit_fully_matched:
    credit_exchange_amount = remaining_credit_foreign + partial_credit_amount_currency
    if not is_zero_in( credit_currency , credit_exchange_amount ):
        book an exchange line on the credit item carrying a foreign-amount repair of credit_exchange_amount
        remaining_credit_foreign = remaining_credit_foreign + credit_exchange_amount
```

### 13.2 Case B — reconciliation in a foreign currency, side fully matched

The repair is made in the **company-currency** amount, and it clears the whole remaining balance
of that side.

```formula
debit side, fully matched:
    debit_exchange_amount = remaining_debit_company − partial_amount
    if not is_zero_in( company_currency , debit_exchange_amount ):
        book an exchange line on the debit item carrying a company-currency repair of debit_exchange_amount
        remaining_debit_company = remaining_debit_company − debit_exchange_amount
        and when the debit currency is the company currency, decrease the foreign remainder by the same amount

credit side, fully matched:
    credit_exchange_amount = remaining_credit_company + partial_amount
    if not is_zero_in( company_currency , credit_exchange_amount ):
        book an exchange line on the credit item carrying a company-currency repair of credit_exchange_amount
        remaining_credit_company = remaining_credit_company − credit_exchange_amount
        and when the credit currency is the company currency, decrease the foreign remainder by the same amount
```

### 13.3 Case C — reconciliation in a foreign currency, side only partly matched

The side is not finished, so the repair does not clear it; instead it keeps the ratio between the
two remaining amounts equal to the ratio the item was booked at, so that the *next* partial on
the same item still works.

```formula
debit side, not fully matched:
    debit_exchange_amount = partial_debit_amount − partial_amount
    if compare_in( company_currency , debit_exchange_amount , 0 ) > 0:
        book an exchange line on the debit item carrying a company-currency repair of debit_exchange_amount
        remaining_debit_company = remaining_debit_company − debit_exchange_amount

credit side, not fully matched:
    credit_exchange_amount = partial_amount − partial_credit_amount
    if compare_in( company_currency , credit_exchange_amount , 0 ) < 0:
        book an exchange line on the credit item carrying a company-currency repair of credit_exchange_amount
        remaining_credit_company = remaining_credit_company − credit_exchange_amount
```

Note the asymmetry of the two guards: the debit repair is booked only when it is **strictly
positive**, the credit repair only when it is **strictly negative**. A repair of the opposite
sign would move the residual the wrong way and is discarded.

### 13.4 Case D — no exchange difference at all

The whole of section 13 is skipped when the caller has suppressed exchange differences. Two
distinct suppressions exist and both must be honoured: one that suppresses differences for the
current operation, and one that suppresses them for the current operation **and** every
reconciliation it triggers. The second is what prevents an exchange difference entry from
generating an exchange difference entry of its own when it is matched back against the item it
repairs.

### 13.5 Decrementing the residuals

After the exchange handling, and regardless of it:

```formula
remaining_debit_company  = remaining_debit_company  − partial_amount
remaining_credit_company = remaining_credit_company + partial_amount
remaining_debit_foreign  = remaining_debit_foreign  − partial_debit_amount_currency
remaining_credit_foreign = remaining_credit_foreign + partial_credit_amount_currency
```

A side is declared finished when **both** of its remaining amounts pass the zero test at their
own currencies.

### 13.6 The partial record produced

```formula
partial.amount                  = partial_amount
partial.debit_amount_currency   = partial_debit_amount_currency
partial.credit_amount_currency  = partial_credit_amount_currency
```

all three always positive.

---

## 14. Formatting an amount for a reader

### 14.1 The language-aware number format

```formula
formatted = group( render( rounded_value , digits ) , grouping_pattern , thousands_separator ) with the decimal point replaced by the language's decimal separator
```

Algorithm:

1. **Decide the number of digits.**
   - When the rounding unit is *decimals*: if a decimal precision record was named, its digit
     count wins; otherwise, if a currency was supplied, the currency's decimal places win;
     otherwise the caller's digit count is used, defaulting to two.
   - When the rounding unit is anything else (units, thousands, lakhs or millions), the digit
     count is **zero**.
2. **Scale.** Divide the value by the rounding unit's factor: one for decimals and for units, one
   thousand for thousands, one hundred thousand for lakhs, one million for millions.
3. **Round.** Round the scaled value onto that number of digits with the requested rounding
   method, whose default here is **half to even**.
4. **Render.** Produce the number with exactly that many fractional digits.
5. **Group.** Insert the language's thousands separator according to the language's grouping
   pattern, reading the pattern from the decimal separator outwards, with a zero terminating the
   pattern and a repeated final size meaning "keep using this size".
6. **Replace the decimal point** with the language's decimal separator.
7. **Attach the symbol,** when a currency with a symbol was supplied: the formatted number, a
   **non-breaking space**, and the symbol — in that order when the currency's symbol position is
   *after*, and in the reverse order when it is *before*.

An empty value renders as the empty string rather than as zero.

### 14.2 The currency-specific format

A second, currency-first routine exists and is the one a currency uses when asked to format an
amount:

1. Build a render pattern with exactly the currency's decimal places.
2. Round the amount onto the **currency's rounding factor** (half away from zero).
3. Render and group it with the reader's language, always with grouping on.
4. Replace every ordinary space by a **non-breaking space**, and replace every minus sign by a
   minus sign followed by a **zero-width non-breaking space**. The second substitution stops a
   line break from separating the sign from the digits.
5. When trailing zeros are not wanted, strip a run of trailing zeros together with the decimal
   separator that precedes it.
6. Prefix the symbol followed by a non-breaking space when the symbol position is *before*;
   suffix a non-breaking space followed by the symbol when it is *after*. A currency with no
   symbol contributes only the space.

### 14.3 Worked formatting examples

Language conventions used below: the English convention has a full stop as the decimal separator,
a comma as the thousands separator, and groups of three; the French convention has a comma as the
decimal separator, a narrow space as the thousands separator, and groups of three; the South Asian
convention has a full stop, a comma, and one group of three followed by groups of two.

| Amount | Currency | Language convention | Result (spaces shown as ordinary spaces) |
|---|---|---|---|
| 1234567.891 | `USD`, factor 0.01, symbol before | English | `$ 1,234,567.89` |
| 1234567.891 | `EUR`, factor 0.01, symbol after | French | `1 234 567,89 €` |
| 1234567.891 | `INR` (Indian rupee), factor 0.01, symbol before | South Asian | `₹ 12,34,567.89` |
| 1234567.891 | `JPY`, factor 1, symbol before | English | `¥ 1,234,568` |
| 1234.5 | `JPY`, factor 1 | English | `¥ 1,235` — the tie is broken away from zero by the currency routine |
| 0.125 | `BHD` (Bahraini dinar), factor 0.001, symbol after | English | `0.125 BD` |
| 1000.00 | `USD`, trailing zeros suppressed | English | `$ 1,000` |
| 1000.50 | `USD`, trailing zeros suppressed | English | `$ 1,000.5` |
| −0.001 | `USD` | English | `$ 0.00` — the amount rounds to a negative zero and the sign is dropped |

### 14.4 The negative-zero rule

A currency's format operation adds a positive zero to the amount before formatting. The effect is
that an amount which rounds to a negative zero is presented as a plain zero, never as "minus
zero". A rebuild must reproduce this, because a report showing a minus sign in front of nothing is
treated as a defect.

### 14.5 The compact metric rendering

For dashboards a compact form exists:

1. While the absolute value is one thousand or more, divide by one thousand and step through the
   suffixes: none, `k`, `M`, `G`, stopping at `T`.
2. Round to one decimal place by the language runtime's own nearest-integer rule.
3. Render with the shortest form that reproduces the value, so that a whole number shows no
   decimal digits.
4. Attach the currency symbol immediately before the number when the symbol position is *before*,
   or after a single ordinary space when it is *after*.

Worked values: one hundred twenty-three thousand four hundred fifty-six point seven eight nine
renders as `123.5k`; one hundred twenty-three thousand point seven eight nine renders as `123k`;
minus one hundred twenty-three thousand four hundred fifty-six point seven eight nine renders as
`-123.5k`; zero point seven eight nine renders as `0.8`. With a currency whose symbol is a dollar
sign placed before, the first becomes `$123.5k`.

Deliberate limitation: the sequence stops at the suffix for one million million. Larger values
keep that suffix and grow the number.

---

## 15. Writing an amount in words

A currency can render an amount as words, using its unit and subunit labels.

Algorithm:

1. Render the amount with exactly the currency's decimal places, as a plain string.
2. Split it at the decimal point into a whole part and a fractional part.
3. Read the whole part as a whole number.
4. If the amount minus that whole number passes the **currency's zero test**, the result is:
   the whole number spelled out in the reader's language, in title case, a space, and the
   currency's unit label.
5. Otherwise the result is: the whole number spelled out, a space, the unit label, a space, the
   word *and*, a space, the fractional part read as a whole number and spelled out, a space, and
   the currency's subunit label.
6. When the reader's language has no spelling rules available, the spelling falls back to
   English.
7. When no spelling facility is present at all, the result is the empty string and a warning is
   logged.

Worked examples with a currency whose unit label is *Dollars*, whose subunit label is *Cents* and
whose decimal places are two, in English:

| Amount | Rendered | Whole | Fraction | Result |
|---|---|---|---|---|
| 1234.00 | `1234.00` | 1234 | 00 | *One Thousand, Two Hundred And Thirty-Four Dollars* |
| 1234.56 | `1234.56` | 1234 | 56 | *One Thousand, Two Hundred And Thirty-Four Dollars and Fifty-Six Cents* |
| 0.05 | `0.05` | 0 | 05 | *Zero Dollars and Five Cents* |
| 1234.004 | `1234.00` | 1234 | 00 | *One Thousand, Two Hundred And Thirty-Four Dollars* — the fraction vanished at step 1 |

With a currency whose decimal places are zero, step 2 produces an empty fractional part, step 4
always applies, and the subunit label is never used.

---

## 16. The catalogue of shipped currencies

One hundred and seventy currencies are delivered as reference data. **All of them ship
deactivated**; a currency becomes active when a company adopts it, when a country localization
activates it, or when a user activates it by hand. Activating a second one grants the
multi-currency permission group to every internal user, as described in
[entities.md](entities.md) §1.7.

Summary by rounding factor:

| Rounding factor | Derived decimal places | Count | Currencies |
|---|---|---|---|
| 0.0001 | 4 | 3 | `CLF`, `UYI`, `UYW` |
| 0.001 | 3 | 7 | `BHD`, `IQD`, `JOD`, `KWD`, `LYD`, `OMR`, `TND` |
| 0.01 | 2 | 142 | all others |
| 1 | 0 | 18 | `BIF`, `BYR`, `CLP`, `DJF`, `GNF`, `ISK`, `JPY`, `KMF`, `KRW`, `PYG`, `RWF`, `TWD`, `UGX`, `VND`, `VUV`, `XAF`, `XOF`, `XPF` — with `JPY`, `TWD`, `VND` and `XPF` declaring the factor as one point zero zero, which is numerically identical to one and yields the same zero decimal places |

Summary by symbol position: thirty-three currencies place the symbol **before** the amount —
`ANG`, `ARS`, `AUD`, `BND`, `BRL`, `CHF`, `CLF`, `CLP`, `CNH`, `CNY`, `COP`, `COU`, `CRC`, `DKK`,
`DOP`, `GBP`, `HKD`, `HNL`, `IDR`, `ILS`, `INR`, `JPY`, `KRW`, `MVR`, `MXN`, `MYR`, `NOK`, `NZD`,
`PEN`, `PKR`, `SGD`, `USD` and `ZAR` — and the remaining one hundred thirty-seven place it
**after**. The default for a currency that does not
declare a position is *after*.

Three further currencies are added by a country localization (Chile) and are units of account
rather than circulating money: `UF` (the indexed development unit, symbol `UF`, rounding factor
one hundredth, two decimal places, symbol after), `UTM` (the monthly tax unit, symbol `UTM`,
rounding factor one hundredth, two decimal places, symbol after) and `OTR` (a catch-all "other"
unit, symbol `OTR`, rounding factor one, zero decimal places, symbol after).

Two further activations are worth noting because they change behaviour rather than data: the
Australian, New Zealand, Hong Kong, Taiwanese and Uruguayan localizations each activate a handful
of currencies on installation, and the Taiwanese localization also overrides the symbol position
of `TWD` to **before**.

### 16.1 The complete table

Every shipped currency, alphabetically by code. *Decimal places* is derived from the rounding
factor by the formula of section 1.3 and is shown for completeness; it is not independent data.

| Code | Name | Numeric code | Symbol | Rounding factor | Decimal places | Symbol position | Unit label | Subunit label |
|---|---|---|---|---|---|---|---|---|
| `AED` | United Arab Emirates dirham | 784 | `AED` | 0.01 | 2 | after | Dirham | Fils |
| `AFN` | Afghan afghani | 971 | `Afs` | 0.01 | 2 | after | Afghani | Puls |
| `ALL` | Albanian lek | 008 | `L` | 0.01 | 2 | after | Lek | Qindarke |
| `AMD` | Armenian dram | 051 | `դր.` | 0.01 | 2 | after | Dram | Luma |
| `ANG` | Netherlands Antillean guilder | 532 | `ƒ` | 0.01 | 2 | before | Guilder | Cents |
| `AOA` | Angolan kwanza | 973 | `Kz` | 0.01 | 2 | after | Kwanza | Centimos |
| `ARS` | Argentine peso | 032 | `$` | 0.01 | 2 | before | Peso | Centavos |
| `AUD` | Australian dollar | 036 | `$` | 0.01 | 2 | before | Dollars | Cents |
| `AWG` | Aruban florin | 533 | `Afl.` | 0.01 | 2 | after | Guilder | Cents |
| `AZN` | Azerbaijani manat | 944 | `₼` | 0.01 | 2 | after | Manat | Qapik |
| `BAM` | Bosnia and Herzegovina convertible mark | 977 | `KM` | 0.01 | 2 | after | Mark | Fening |
| `BBD` | Barbados dollar | 052 | `Bds$` | 0.01 | 2 | after | Dollars | Cents |
| `BDT` | Bangladeshi taka | 050 | `৳` | 0.01 | 2 | after | Taka | Paisa |
| `BGN` | Bulgarian lev | 975 | `лв` | 0.01 | 2 | after | Lev | Stotinki |
| `BHD` | Bahraini dinar | 048 | `BD` | 0.001 | 3 | after | Dinar | Fils |
| `BIF` | Burundian franc | 108 | `FBu` | 1 | 0 | after | Franc | Centime |
| `BMD` | Bermudian dollar | 060 | `BD$` | 0.01 | 2 | after | Dollars | Cents |
| `BND` | Brunei dollar | 096 | `B$` | 0.01 | 2 | before | Dollars | Cents |
| `BOB` | Boliviano | 068 | `Bs.` | 0.01 | 2 | after | Boliviano | Centavos |
| `BRL` | Brazilian real | 986 | `R$` | 0.01 | 2 | before | Real | Centavos |
| `BSD` | Bahamian dollar | 044 | `B$` | 0.01 | 2 | after | Dollars | Cents |
| `BTN` | Bhutanese ngultrum | 064 | `Nu.` | 0.01 | 2 | after | Ngultrum | Chhertum |
| `BWP` | Botswana pula | 072 | `P` | 0.01 | 2 | after | Pula | Thebe |
| `BYN` | Belarusian ruble | 974 | `Br` | 0.01 | 2 | after | Rubles | Kopeks |
| `BYR` | Belarusian ruble | 974 | `BR` | 1 | 0 | after | Ruble BYR | Kapeyka |
| `BZD` | Belize dollar | 084 | `BZ$` | 0.01 | 2 | after | Dollars | Cents |
| `CAD` | Canadian dollar | 124 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `CDF` | Congolese franc | 976 | `Fr` | 0.01 | 2 | after | Franc | Centime |
| `CHF` | Swiss franc | 756 | `CHF` | 0.01 | 2 | before | Franc | Centimes |
| `CLF` | Unidad de Fomento | 990 | `$` | 0.0001 | 4 | before | Peso | Centavos |
| `CLP` | Chilean peso | 152 | `$` | 1 | 0 | before | Peso | Centavos |
| `CNH` | Chinese yuan - Offshore |  | `¥` | 0.01 | 2 | before | Yuan | Fen |
| `CNY` | Chinese yuan | 156 | `¥` | 0.01 | 2 | before | Yuan | Fen |
| `COP` | Colombian peso | 170 | `$` | 0.01 | 2 | before | Peso | Centavos |
| `COU` | Unidad de Valor Real | 970 | `$` | 0.01 | 2 | before | Peso | centavo |
| `CRC` | Costa Rican colón | 188 | `₡` | 0.01 | 2 | before | Colon | Centimos |
| `CUC` | Cuban convertible peso | 931 | `$` | 0.01 | 2 | after | Cuban convertible peso |  |
| `CUP` | Cuban peso | 192 | `$` | 0.01 | 2 | after | Peso | Centavos |
| `CVE` | Cape Verdean escudo | 132 | `$` | 0.01 | 2 | after | Escudo | Centavo |
| `CZK` | Czech koruna | 203 | `Kč` | 0.01 | 2 | after | Koruna | Halers |
| `DJF` | Djiboutian franc | 262 | `Fdj` | 1 | 0 | after | Franc | Centime |
| `DKK` | Danish krone | 208 | `kr` | 0.01 | 2 | before | Krone | Ore |
| `DOP` | Dominican peso | 214 | `RD$` | 0.01 | 2 | before | Pesos | Centavos |
| `DZD` | Algerian dinar | 012 | `DA` | 0.01 | 2 | after | Dinar | Centimes |
| `EGP` | Egyptian pound | 818 | `LE` | 0.01 | 2 | after | Pound | Piastres |
| `ERN` | Eritrean nakfa | 232 | `Nfk` | 0.01 | 2 | after | Nakfa | Cents |
| `ETB` | Ethiopian birr | 230 | `Br` | 0.01 | 2 | after | Birr | Cents |
| `EUR` | Euro | 978 | `€` | 0.01 | 2 | after | Euros | Cents |
| `FJD` | Fiji dollar | 242 | `FJ$` | 0.01 | 2 | after | Dollars | Cents |
| `FKP` | Falkland Islands pound | 238 | `£` | 0.01 | 2 | after | Pound | Penny |
| `GBP` | Pound sterling | 826 | `£` | 0.01 | 2 | before | Sterling | Penny |
| `GEL` | Georgian lari | 981 | `ლ` | 0.01 | 2 | after | Lari | Tetri |
| `GHS` | Ghanaian cedi | 936 | `GH¢` | 0.01 | 2 | after | Cedi | Pesewas |
| `GIP` | Gibraltar pound | 292 | `£` | 0.01 | 2 | after | Pound | Penny |
| `GMD` | Gambian dalasi | 270 | `D` | 0.01 | 2 | after | Dalasi | Butut |
| `GNF` | Guinean franc | 324 | `FG` | 1 | 0 | after | Franc | Centime |
| `GTQ` | Guatemalan Quetzal | 320 | `Q` | 0.01 | 2 | after | Quetzales | Centavo |
| `GYD` | Guyanese dollar | 328 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `HKD` | Hong Kong dollar | 344 | `$` | 0.01 | 2 | before | Dollars | Cents |
| `HNL` | Honduran lempira | 340 | `L` | 0.01 | 2 | before | Lempiras | Centavos |
| `HRK` | Croatian kuna | 191 | `kn` | 0.01 | 2 | after | Kuna | Lipa |
| `HTG` | Haitian gourde | 332 | `G` | 0.01 | 2 | after | Gourde | Centime |
| `HUF` | Hungarian forint | 348 | `Ft` | 0.01 | 2 | after | Forint | Filler |
| `IDR` | Indonesian rupiah | 360 | `Rp` | 0.01 | 2 | before | Rupiah | Sen |
| `ILS` | Israeli new shekel | 376 | `₪` | 0.01 | 2 | before | Shekel | Agorot |
| `INR` | Indian rupee | 356 | `₹` | 0.01 | 2 | before | Rupees | Paise |
| `IQD` | Iraqi dinar | 368 | `ع.د` | 0.001 | 3 | after | Dinar | Fils |
| `IRR` | Iranian rial | 364 | `﷼` | 0.01 | 2 | after | Dinar | Fils |
| `ISK` | Icelandic króna | 352 | `kr` | 1 | 0 | after | Krona | Aurar |
| `JMD` | Jamaican dollar | 388 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `JOD` | Jordanian dinar | 400 | `د.ا` | 0.001 | 3 | after | Dinar | Fils |
| `JPY` | Japanese yen | 392 | `¥` | 1.00 | 0 | before | Yen | Cen |
| `KES` | Kenyan shilling | 404 | `KSh` | 0.01 | 2 | after | Shilling | Cents |
| `KGS` | Kyrgyzstani som | 417 | `лв` | 0.01 | 2 | after | Som | Tyiyn |
| `KHR` | Cambodian riel | 116 | `៛` | 0.01 | 2 | after | Riel | Sen |
| `KMF` | Comorian franc | 174 | `CF` | 1 | 0 | after | Franc | Centime |
| `KPW` | North Korean won | 408 | `₩` | 0.01 | 2 | after | Won | Chon |
| `KRW` | South Korean won | 410 | `₩` | 1 | 0 | before | Won | Chon |
| `KWD` | Kuwaiti dinar | 414 | `د.ك` | 0.001 | 3 | after | Dinar | Fils |
| `KYD` | Cayman Islands dollar | 136 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `KZT` | Kazakhstani tenge | 398 | `₸` | 0.01 | 2 | after | Tenge | Tiin |
| `LAK` | Lao kip | 418 | `₭` | 0.01 | 2 | after | Kip | Att |
| `LBP` | Lebanese pound | 422 | `ل.ل` | 0.01 | 2 | after | Pound | Piastres |
| `LKR` | Sri Lankan rupee | 144 | `Rs` | 0.01 | 2 | after | Rupee | Cents |
| `LRD` | Liberian dollar | 430 | `L$` | 0.01 | 2 | after | Dollars | Cents |
| `LSL` | Lesotho loti | 426 | `M` | 0.01 | 2 | after | Loti | Sente |
| `LTL` | Lithuanian litas | 840 | `Lt` | 0.01 | 2 | after | Litas | Centas |
| `LVL` | Latvian lats | 840 | `Ls` | 0.01 | 2 | after | Lats | Santims |
| `LYD` | Libyan dinar | 434 | `ل.د` | 0.001 | 3 | after | Dinar | Dirham |
| `MAD` | Moroccan dirham | 504 | `DH` | 0.01 | 2 | after | Dirham | Centimes |
| `MDL` | Moldovan leu | 498 | `L` | 0.01 | 2 | after | Leu | Ban |
| `MGA` | Malagasy ariary | 969 | `Ar` | 0.01 | 2 | after | Ariary | Iraimbilanja |
| `MKD` | Macedonian denar | 807 | `ден` | 0.01 | 2 | after | Denar | Deni |
| `MMK` | Myanmar kyat | 104 | `K` | 0.01 | 2 | after | Kyat | Pya |
| `MNT` | Mongolian tögrög | 496 | `₮` | 0.01 | 2 | after | Tugrik | Mongo |
| `MOP` | Macanese pataca | 446 | `MOP$` | 0.01 | 2 | after | Pataca | Sin |
| `MRO` | Mauritanian ouguiya (old) | 478 | `UM` | 0.01 | 2 | after | Ouguiya | Khoums |
| `MRU` | Mauritanian ouguiya | 478 | `UM` | 0.01 | 2 | after | Ouguiya | Khoums |
| `MUR` | Mauritian rupee | 480 | `Rs` | 0.01 | 2 | after | Rupee | Cents |
| `MVR` | Maldivian rufiyaa | 462 | `Rf` | 0.01 | 2 | before | Rufiyaa | Laari |
| `MWK` | Malawian kwacha | 454 | `MK` | 0.01 | 2 | after | Kwacha | Tambala |
| `MXN` | Mexican peso | 484 | `$` | 0.01 | 2 | before | Pesos | Centavos |
| `MYR` | Malaysian ringgit | 458 | `RM` | 0.01 | 2 | before | Ringgit | Sen |
| `MZN` | Mozambican metical | 943 | `MT` | 0.01 | 2 | after | Metical | Centavo |
| `NAD` | Namibian dollar | 516 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `NGN` | Nigerian naira | 566 | `₦` | 0.01 | 2 | after | Naira | Kobo |
| `NIO` | Nicaraguan córdoba | 558 | `C$` | 0.01 | 2 | after | Cordoba | Centavos |
| `NOK` | Norwegian krone | 578 | `kr` | 0.01 | 2 | before | Krone | Ore |
| `NPR` | Nepalese rupee | 524 | `₨` | 0.01 | 2 | after | Rupee | Paisa |
| `NZD` | New Zealand dollar | 554 | `$` | 0.01 | 2 | before | Dollars | Cents |
| `OMR` | Omani rial | 512 | `ر.ع.` | 0.001 | 3 | after | Rial | Baisa |
| `PAB` | Panamanian balboa | 590 | `B/.` | 0.01 | 2 | after | Balboa | Centesimo |
| `PEN` | Peruvian sol | 604 | `S/` | 0.01 | 2 | before | Soles | Centimos |
| `PGK` | Papua New Guinean kina | 598 | `K` | 0.01 | 2 | after | Kina | Toea |
| `PHP` | Philippine peso | 608 | `₱` | 0.01 | 2 | after | Peso | Centavos |
| `PKR` | Pakistani rupee | 586 | `Rs.` | 0.01 | 2 | before | Rupee | Paisa |
| `PLN` | Polish złoty | 985 | `zł` | 0.01 | 2 | after | Zloty | Groszy |
| `PYG` | Paraguayan guaraní | 600 | `₲` | 1 | 0 | after | Guarani | Centimos |
| `QAR` | Qatari riyal | 634 | `QR` | 0.01 | 2 | after | Riyal | Dirham |
| `RON` | Romanian leu | 946 | `lei` | 0.01 | 2 | after | Leu | Bani |
| `RSD` | Serbian dinar | 941 | `din.` | 0.01 | 2 | after | Dinar | Para |
| `RUB` | Russian ruble | 643 | `руб` | 0.01 | 2 | after | Ruble | Kopek |
| `RWF` | Rwandan franc | 646 | `RF` | 1 | 0 | after | Franc | Santime |
| `SAR` | Saudi riyal | 682 | `SR` | 0.01 | 2 | after | Riyal | Halala |
| `SBD` | Solomon Islands dollar | 090 | `SI$` | 0.01 | 2 | after | Dollars | Cents |
| `SCR` | Seychellois rupee | 690 | `SR` | 0.01 | 2 | after | Rupee | Cents |
| `SDG` | Sudanese pound | 938 | `ج.س.` | 0.01 | 2 | after |  |  |
| `SEK` | Swedish krona | 752 | `kr` | 0.01 | 2 | after | Krona | Ore |
| `SGD` | Singapore dollar | 702 | `S$` | 0.01 | 2 | before | Dollars | Cents |
| `SHP` | Saint Helena pound | 654 | `£` | 0.01 | 2 | after | Pound | Penny |
| `SLE` | Sierra Leonean leone | 694 | `Le` | 0.01 | 2 | after | Leone | Cents |
| `SLL` | Sierra Leonean leone | 694 | `Le` | 0.01 | 2 | after | Leone | Cents |
| `SOS` | Somali shilling | 706 | `Sh.` | 0.01 | 2 | after | Shillings | Senti |
| `SRD` | Surinamese dollar | 968 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `SSP` | South Sudanese pound | 728 | `£` | 0.01 | 2 | after | Pounds | Piasters |
| `STD` | São Tomé and Príncipe dobra | 678 | `Db` | 0.01 | 2 | after | Dobra | Centimo |
| `STN` | São Tomé and Príncipe dobra | 678 | `Db` | 0.01 | 2 | after | Dobra | cêntimo |
| `SVC` | Salvadoran Colon | 222 | `¢` | 0.01 | 2 | after | Colones | Centavo |
| `SYP` | Syrian pound | 760 | `£` | 0.01 | 2 | after | Pound | Piastrp |
| `SZL` | Swazi lilangeni | 748 | `E` | 0.01 | 2 | after | Lilangeni | Cents |
| `THB` | Thai baht | 764 | `฿` | 0.01 | 2 | after | Baht | Satang |
| `TJS` | Tajikistani somoni | 972 | `TJS` | 0.01 | 2 | after | Somoni | Diram |
| `TMT` | Turkmenistan manat | 934 | `T` | 0.01 | 2 | after | Manat | Tenge |
| `TND` | Tunisian dinar | 788 | `DT` | 0.001 | 3 | after | Dinar | Millimes |
| `TOP` | Tongan paʻanga | 776 | `T$` | 0.01 | 2 | after | Paanga | Seniti |
| `TRY` | Turkish lira | 949 | `₺` | 0.01 | 2 | after | Lira | Kurus |
| `TTD` | Trinidad and Tobago dollar | 780 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `TWD` | New Taiwan dollar | 901 | `NT$` | 1.00 | 0 | after | Dollars | Cents |
| `TZS` | Tanzanian shilling | 834 | `TSh` | 0.01 | 2 | after | Shilling | Senti |
| `UAH` | Ukraine Hryvnia | 980 | `₴` | 0.01 | 2 | after | Hryvnia | Kopiyka |
| `UGX` | Ugandan shilling | 800 | `USh` | 1 | 0 | after | Shilling | Cents |
| `USD` | United States dollar | 840 | `$` | 0.01 | 2 | before | Dollars | Cents |
| `UYI` | Uruguay Peso en Unidades Indexadas | 940 | `$` | 0.0001 | 4 | after | Peso | centésimo |
| `UYU` | Uruguayan peso | 858 | `$` | 0.01 | 2 | after | Peso | Centesimos |
| `UYW` | Unidad previsional | 858 | `$` | 0.0001 | 4 | after | peso | centésimo |
| `UZS` | Uzbekistan som | 860 | `лв` | 0.01 | 2 | after | Som | Tiyin |
| `VEF` | Venezuelan bolívar fuerte | 937 | `Bs.F` | 0.01 | 2 | after | Bolivar | Centimos |
| `VES` | Venezuelan bolívar soberano | 937 | `Bs` | 0.01 | 2 | after |  |  |
| `VND` | Vietnamese đồng | 704 | `₫` | 1.00 | 0 | after | Dong | Xu |
| `VUV` | Vanuatu vatu | 548 | `VT` | 1 | 0 | after | Vatu |  |
| `WST` | Samoan tālā | 882 | `WS$` | 0.01 | 2 | after | Tala | Sene |
| `XAF` | CFA franc BEAC | 950 | `FCFA` | 1 | 0 | after | Franc | Centimes |
| `XCD` | East Caribbean dollar | 951 | `$` | 0.01 | 2 | after | Dollars | Cents |
| `XCG` | Caribbean Guilder |  | `Cg` | 0.01 | 2 | after | Guilder | Cents |
| `XOF` | CFA franc BCEAO | 952 | `CFA` | 1 | 0 | after | Franc | Centimes |
| `XPF` | CFP franc | 953 | `XPF` | 1.00 | 0 | after | Franc | Centimes |
| `YER` | Yemeni rial | 886 | `﷼` | 0.01 | 2 | after | Rial | Fils |
| `ZAR` | South African rand | 710 | `R` | 0.01 | 2 | before | Rand | Cents |
| `ZIG` | Zimbabwe Gold |  | `ZiG` | 0.01 | 2 | after | ZiGs |  |
| `ZMW` | Zambian kwacha | 967 | `ZK` | 0.01 | 2 | after | Kwacha | Ngwee |

---

## 17. Mandatory worked example — a rate of one point one zero at invoicing and one point two zero at payment, producing an exchange gain

This example walks the whole chain: document, payment, reconciliation currency choice, partial
amounts, exchange difference amount, and the exact journal items produced. Every number below is
derived by the formulas of the preceding sections.

### 17.1 The setting

| Element | Value |
|---|---|
| Company | Northwind, reporting in `EUR` (the euro) |
| Document currency | `USD` (the United States dollar) |
| Rate record on 1 March 2026 | technical rate of `USD` = **1.10** (one euro buys one point ten United States dollars) |
| Rate record on 1 April 2026 | technical rate of `USD` = **1.20** |
| `EUR` rate records | none, so the euro's rate is the final fallback of one |
| Exchange journal | Miscellaneous Operations |
| Gain Exchange Rate Account | `7760 Foreign Exchange Gain` |
| Loss Exchange Rate Account | `6560 Foreign Exchange Loss` |

The document is a **vendor bill**, because a rate that rises from one point one zero to one point
two zero means the euro strengthens against the dollar, which makes a dollar *liability* cheaper
to settle and therefore produces a **gain**. The mirror case — a customer invoice under the same
rate movement, producing a loss — is given in section 19.

### 17.2 The bill, dated 1 March 2026

The rate date is the invoice date, 1 March 2026, so the lookup of section 7 returns one point ten
and the stored document rate is **1.10**. The bill is for one thousand one hundred United States
dollars of consultancy, with no tax.

```formula
balance = round_to( EUR , 1 100.00 ÷ 1.10 ) = round( 1 000.000000 ) = 1 000.00
```

Journal entry **Bill B/2026/0004**, journal *Vendor Bills*, date 1 March 2026, document currency
`USD`, document rate 1.10:

| Line | Account | Debit (`EUR`) | Credit (`EUR`) | Foreign amount (`USD`) | Currency |
|---|---|---|---|---|---|
| 1 | `6100 Consultancy` | 1 000.00 | | +1 100.00 | `USD` |
| 2 | `4400 Accounts Payable` | | 1 000.00 | −1 100.00 | `USD` |

Check of the invariants: the balances sum to zero; the two foreign amounts have the same sign as
their balances; the payable line's residual is minus one thousand euros and minus one thousand
one hundred United States dollars.

### 17.3 The payment, dated 1 April 2026

The supplier is paid in full: one thousand one hundred United States dollars, from a bank journal
whose currency is `USD`. The rate in force on 1 April 2026 is one point twenty.

```formula
balance = round_to( EUR , 1 100.00 ÷ 1.20 ) = round( 916.666667 ) = 916.67
```

Journal entry **BNK1/2026/0011**, journal *Bank USD*, date 1 April 2026:

| Line | Account | Debit (`EUR`) | Credit (`EUR`) | Foreign amount (`USD`) | Currency |
|---|---|---|---|---|---|
| 1 | `4400 Accounts Payable` | 916.67 | | +1 100.00 | `USD` |
| 2 | `5120 Bank USD` | | 916.67 | −1 100.00 | `USD` |

### 17.4 Reconciling the two payable lines

The debit side is the payment's payable line; the credit side is the bill's payable line.

**Available residuals of the debit side** (section 11.1): the company currency with a residual of
nine hundred sixteen point sixty-seven and a rate of one; and `USD` with a residual of one
thousand one hundred and an accounting rate of

```formula
| 1 100.00 ÷ 916.67 | = 1.199995636…
```

**Available residuals of the credit side**: the company currency with a residual of minus one
thousand and a rate of one; and `USD` with a residual of minus one thousand one hundred and an
accounting rate of

```formula
| −1 100.00 ÷ −1 000.00 | = 1.10
```

**Reconciliation currency** (section 11.4): the debit currency is `USD`, it is not the company
currency, and it appears in both maps — so the reconciliation currency is **`USD`**.

**The comparison:**

```formula
debit_recon_amount  = 1 100.00
credit_recon_amount = − ( −1 100.00 ) = 1 100.00
comparison = compare_in( USD , 1 100.00 , 1 100.00 ) = 0
```

Both sides are therefore fully matched, and the minimum is one thousand one hundred.

**The partial amounts, case B** (section 12.2):

```formula
debit_interval  = interval( USD , EUR , 1 100.00 , 1 ÷ 1.199995636 )
                = ( round( 1 099.995 × 0.833336… ) , round( 1 100.00 × 0.833336… ) , round( 1 100.005 × 0.833336… ) )
                = ( 916.67 , 916.67 , 916.67 )
credit_interval = interval( USD , EUR , 1 100.00 , 1 ÷ 1.10 )
                = ( round( 1 099.995 ÷ 1.10 ) , round( 1 100.00 ÷ 1.10 ) , round( 1 100.005 ÷ 1.10 ) )
                = ( 1 000.00 , 1 000.00 , 1 000.00 )

partial_debit_amount  = min( 916.67   , 916.67   ) = 916.67
partial_credit_amount = min( 1 000.00 , 1 000.00 ) = 1 000.00
partial_amount        = min( 916.67 , 1 000.00 )   = 916.67
```

**The tolerance band** (section 12.2) does not apply: the debit amount of nine hundred sixteen
point sixty-seven is below the lowest value of the credit interval, one thousand, so the second
of the four conditions fails. The disagreement is economic, not a rounding artefact.

**The foreign amounts of the partial:** neither currency is the company currency, so both are the
minimum:

```formula
partial_debit_amount_currency  = 1 100.00
partial_credit_amount_currency = 1 100.00
```

### 17.5 The exchange difference amount

Reconciliation is in a foreign currency, so case B of section 13.2 applies to each fully-matched
side.

Debit side (the payment's payable line):

```formula
debit_exchange_amount = remaining_debit_company − partial_amount = 916.67 − 916.67 = 0.00
```

Zero at the company currency, so **no** exchange line for the payment.

Credit side (the bill's payable line):

```formula
credit_exchange_amount = remaining_credit_company + partial_amount = −1 000.00 + 916.67 = −83.33
```

Not zero, so an exchange line is booked on the bill's payable line carrying a company-currency
repair of **minus eighty-three point thirty-three**. The remaining company-currency amount of
the credit side becomes minus nine hundred sixteen point sixty-seven.

### 17.6 The exchange difference journal entry

The repair amount is negative, so the counterpart account is the **Gain Exchange Rate Account**
(section 13 and [accounting-effects.md](accounting-effects.md) §2). The entry's date is the later
of the two items' dates, 1 April 2026, subject to the exchange journal's accounting-date rules.

Journal entry **MISC/2026/0007**, journal *Miscellaneous Operations*, date 1 April 2026, always
tax-exigible:

| Line | Account | Debit (`EUR`) | Credit (`EUR`) | Foreign amount (`USD`) | Currency | Partner |
|---|---|---|---|---|---|---|
| 1 | `4400 Accounts Payable` | 83.33 | | 0.00 | `USD` | the supplier |
| 2 | `7760 Foreign Exchange Gain` | | 83.33 | 0.00 | `USD` | the supplier |

The derivation of each cell, from section 13 and the entry preparation of
[accounting-effects.md](accounting-effects.md) §2.3:

- The repair amount is minus eighty-three point thirty-three, so on line one the debit is the
  negated repair, eighty-three point thirty-three, and the credit is zero.
- The item's currency differs from the company currency, so the foreign repair amount is zero;
  line one's foreign amount is the negated zero and line two's is zero. Both lines still carry
  `USD` as their currency, so that the entry does not accidentally re-express itself.
- Line two mirrors line one on the exchange account.
- Both lines carry the partner of the repaired item.
- Line one is marked as reconciling against the bill's payable line, which triggers a second
  partial reconciliation between the two.

### 17.7 The resulting reconciliation records

| Record | Debit item | Credit item | Amount (`EUR`) | Debit amount (`USD`) | Credit amount (`USD`) |
|---|---|---|---|---|---|
| Partial 1 | Exchange entry line 1 | Bill payable line | 83.33 | 0.00 | 0.00 |
| Partial 2 | Payment payable line | Bill payable line | 916.67 | 1 100.00 | 1 100.00 |

### 17.8 The residuals after reconciliation

| Item | Balance (`EUR`) | Matched (`EUR`) | Residual (`EUR`) | Foreign amount (`USD`) | Matched (`USD`) | Residual (`USD`) | Reconciled |
|---|---|---|---|---|---|---|---|
| Bill payable line | −1 000.00 | −83.33 − 916.67 | 0.00 | −1 100.00 | −1 100.00 | 0.00 | yes |
| Payment payable line | +916.67 | +916.67 | 0.00 | +1 100.00 | +1 100.00 | 0.00 | yes |
| Exchange entry line 1 | +83.33 | +83.33 | 0.00 | 0.00 | 0.00 | 0.00 | yes |

All three residuals are zero in both currencies, so a full reconciliation record is created and
all three items receive its matching number. The economic result is recorded correctly: the
company owed one thousand euros, paid nine hundred sixteen euros and sixty-seven cents, and
recognised a gain of eighty-three euros and thirty-three cents.

### 17.9 Arithmetic check

```formula
bill in company currency      = 1 100.00 ÷ 1.10 = 1 000.00
payment in company currency   = 1 100.00 ÷ 1.20 =   916.67   (rounded from 916.666667)
gain                          = 1 000.00 − 916.67 = 83.33
```

---

## 18. Mandatory worked example — a partial payment leaving residuals in both currencies

### 18.1 The setting

Same company, same currencies, same accounts as section 17, but the document is a **customer
invoice** and the payment is partial.

| Element | Value |
|---|---|
| Invoice date | 1 March 2026, rate `USD` = 1.10 |
| Invoice amount | 1 100.00 `USD` |
| Payment date | 1 April 2026, rate `USD` = 1.20 |
| Payment amount | **600.00 `USD`** — a partial settlement |

### 18.2 The invoice

```formula
balance = round_to( EUR , 1 100.00 ÷ 1.10 ) = 1 000.00
```

Journal entry **INV/2026/0021**, journal *Customer Invoices*, date 1 March 2026:

| Line | Account | Debit (`EUR`) | Credit (`EUR`) | Foreign amount (`USD`) |
|---|---|---|---|---|
| 1 | `4000 Accounts Receivable` | 1 000.00 | | +1 100.00 |
| 2 | `7000 Product Sales` | | 1 000.00 | −1 100.00 |

### 18.3 The partial payment

```formula
balance = round_to( EUR , 600.00 ÷ 1.20 ) = 500.00
```

Journal entry **BNK1/2026/0012**, date 1 April 2026:

| Line | Account | Debit (`EUR`) | Credit (`EUR`) | Foreign amount (`USD`) |
|---|---|---|---|---|
| 1 | `5120 Bank USD` | 500.00 | | +600.00 |
| 2 | `4000 Accounts Receivable` | | 500.00 | −600.00 |

### 18.4 Matching

Debit side: the invoice's receivable line, residuals plus one thousand euros and plus one
thousand one hundred United States dollars, accounting rate one point ten.
Credit side: the payment's receivable line, residuals minus five hundred euros and minus six
hundred United States dollars, accounting rate one point twenty exactly.

Reconciliation currency: `USD`.

```formula
debit_recon_amount  = 1 100.00
credit_recon_amount = 600.00
comparison = compare_in( USD , 1 100.00 , 600.00 ) = +1
debit_fully_matched  = false
credit_fully_matched = true
minimum = 600.00
```

Partial amounts, case B:

```formula
debit_interval  = ( round( 599.995 ÷ 1.10 ) , round( 600.00 ÷ 1.10 ) , round( 600.005 ÷ 1.10 ) )
                = ( 545.45 , 545.45 , 545.46 )
credit_interval = ( round( 599.995 ÷ 1.20 ) , round( 600.00 ÷ 1.20 ) , round( 600.005 ÷ 1.20 ) )
                = ( 500.00 , 500.00 , 500.00 )

partial_debit_amount  = min( 545.45 , 1 000.00 ) = 545.45
partial_credit_amount = min( 500.00 ,   500.00 ) = 500.00
partial_amount        = min( 545.45 , 500.00 )   = 500.00
```

The tolerance band does not apply: five hundred forty-five point forty-five is above five hundred,
the highest value of the credit interval.

```formula
partial_debit_amount_currency  = 600.00
partial_credit_amount_currency = 600.00
```

### 18.5 The exchange difference — the partly-matched case

The debit side is **not** fully matched, so case C of section 13.3 applies to it:

```formula
debit_exchange_amount = partial_debit_amount − partial_amount = 545.45 − 500.00 = 45.45
compare_in( EUR , 45.45 , 0 ) = +1 > 0  →  the repair is booked
```

The credit side **is** fully matched, so case B applies to it:

```formula
credit_exchange_amount = remaining_credit_company + partial_amount = −500.00 + 500.00 = 0.00  →  nothing booked
```

The repair amount on the invoice's receivable line is **positive** forty-five point forty-five,
so the counterpart account is the **Loss Exchange Rate Account**.

Journal entry **MISC/2026/0008**, journal *Miscellaneous Operations*, date 1 April 2026:

| Line | Account | Debit (`EUR`) | Credit (`EUR`) | Foreign amount (`USD`) | Currency |
|---|---|---|---|---|---|
| 1 | `4000 Accounts Receivable` | | 45.45 | 0.00 | `USD` |
| 2 | `6560 Foreign Exchange Loss` | 45.45 | | 0.00 | `USD` |

### 18.6 The residuals — the point of the example

```formula
remaining_debit_company = 1 000.00 − 45.45 − 500.00 = 454.55
remaining_debit_foreign = 1 100.00 − 600.00         = 500.00
```

| Item | Residual in `EUR` | Residual in `USD` | Reconciled |
|---|---|---|---|
| Invoice receivable line | **454.55** | **500.00** | no — both residuals are non-zero |
| Payment receivable line | 0.00 | 0.00 | yes |
| Exchange entry line 1 | 0.00 | 0.00 | yes |

**Why the two residuals are consistent.** The invoice was booked at a rate of one point ten. The
remaining five hundred United States dollars at that same rate are

```formula
round_to( EUR , 500.00 ÷ 1.10 ) = round( 454.545455 ) = 454.55
```

which is exactly the remaining company-currency residual. That is what case C of section 13.3 is
for: without the forty-five point forty-five repair, the remaining pair would have been five
hundred United States dollars against five hundred euros, implying a rate of one, and the next
partial payment would have produced nonsense.

### 18.7 What the second, closing payment then does

Suppose the remaining five hundred United States dollars are paid on 1 May 2026 at a rate of one
point twenty-five.

```formula
balance of the second payment = round_to( EUR , 500.00 ÷ 1.25 ) = 400.00
```

Matching: reconciliation currency `USD`; debit recon amount five hundred, credit recon amount five
hundred, comparison zero, both fully matched, minimum five hundred.

```formula
partial_debit_amount  = round( 500.00 × ( 1 ÷ 1.099989… ) ) = 454.55
partial_credit_amount = round( 500.00 ÷ 1.25 )              = 400.00
partial_amount        = 400.00
debit_exchange_amount  = 454.55 − 400.00 = 54.55   (case B, debit fully matched)
credit_exchange_amount = −400.00 + 400.00 = 0.00
```

A second exchange entry debits the Loss account by fifty-four euros and fifty-five cents and
credits the receivable by the same. The invoice's receivable line ends with residuals of zero in
both currencies and the whole chain closes. Total loss recognised across the two settlements:
forty-five point forty-five plus fifty-four point fifty-five, that is exactly one hundred euros —
the difference between the one thousand euros of revenue recognised and the nine hundred euros
actually collected (five hundred plus four hundred).

---

## 19. The mirror case — the same rate movement producing a loss

For completeness, the customer-invoice version of section 17, settled in full.

| Element | Value |
|---|---|
| Invoice, 1 March 2026, rate 1.10 | receivable debit 1 000.00 `EUR` / +1 100.00 `USD` |
| Payment, 1 April 2026, rate 1.20 | receivable credit 916.67 `EUR` / −1 100.00 `USD` |

Debit side is now the **invoice**, credit side the **payment**. Reconciliation currency `USD`,
minimum one thousand one hundred, both fully matched.

```formula
partial_debit_amount  = round( 1 100.00 ÷ 1.10 )         = 1 000.00
partial_credit_amount = round( 1 100.00 ÷ 1.199995636… ) =   916.67
partial_amount        = min( 1 000.00 , 916.67 )         =   916.67

debit_exchange_amount  = remaining_debit_company − partial_amount = 1 000.00 − 916.67 = +83.33
credit_exchange_amount = remaining_credit_company + partial_amount = −916.67 + 916.67 = 0.00
```

The repair amount is **positive**, so the counterpart is the **Loss** account:

| Line | Account | Debit (`EUR`) | Credit (`EUR`) | Foreign amount (`USD`) |
|---|---|---|---|---|
| 1 | `4000 Accounts Receivable` | | 83.33 | 0.00 |
| 2 | `6560 Foreign Exchange Loss` | 83.33 | | 0.00 |

The sign of the repair amount, and nothing else, chooses the account: **positive means loss,
negative means gain**. Stated once more, because it is counter-intuitive on first reading: a
positive repair removes company-currency value from a *debit* item, which is a loss; a negative
repair removes company-currency value from a *credit* item, which is a gain.
