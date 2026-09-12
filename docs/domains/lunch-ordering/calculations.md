# Meal Ordering — Calculations

Every formula and algorithm of the domain, with the quantities named in words, the order of
evaluation, the rounding rule that applies at each step and at least one worked numeric example
carried to the last decimal the rule produces.

Two rounding conventions are used throughout and are stated once here.

- **Currency rounding.** An amount held in a monetary field is rounded to the decimal precision of
  its currency every time it is written, both into the working set and into the table. The method is
  half away from zero: a value exactly on the half is rounded to the neighbour further from zero.
  Before the comparison the implementation adds an epsilon proportional to the magnitude of the
  value, so that a value which the binary representation places a fraction below the tie, such as
  the stored form of 0.435, still rounds upward to 0.44 rather than downward to 0.43. The examples
  below assume a currency with two decimal places and a rounding step of 0.01.
- **Fixed-digit rounding.** Where a rule rounds to a stated number of digits rather than to a
  currency, the same half-away-from-zero method with the same epsilon correction is used.

---

## 1. The total price of an order line

### 1.1 Rule

The extras of a line are the union of its three extra sets. The union is taken before the sum, so an
extra that appears in more than one of the three sets is counted once.

```formula
extras total for one unit (currency) = sum over every distinct extra of the line of (extra price (currency))

price for one unit (currency) = meal price (currency) + extras total for one unit (currency)

total price of the line (currency) = quantity (units) × price for one unit (currency)
```

### 1.2 Evaluation order and rounding

1. The distinct extras are collected from the three sets, in the order first set, second set, third
   set, keeping the first appearance of each.
2. Their prices are added in that order, each price already rounded to the currency's precision
   because the extra price is a monetary field.
3. The meal price is added. The meal price is not a monetary field: it is a decimal held with the
   accounting decimal precision, which is two places unless the installation has changed it.
4. The sum is multiplied by the quantity, which is a decimal and may be fractional.
5. The product is written into the line's total price, a monetary field, and is rounded there to the
   currency's precision, half away from zero.

Only step 5 rounds. Steps 2 to 4 are carried at full working precision.

### 1.3 Worked example — ordinary case

A meal priced at 7.20, with two extras priced at 0.30 and 1.15, ordered three times.

```formula
extras total for one unit = 0.30 + 1.15 = 1.45
price for one unit = 7.20 + 1.45 = 8.65
total price of the line = 3 × 8.65 = 25.95
```

The stored total price is **25.95**.

### 1.4 Worked example — rounding edge

A meal priced at 0.09, with one extra priced at 0.055 recorded in a currency with three decimal
places and later read in a currency with two, ordered three times. The extra price is stored at the
precision of its own currency; assume it is stored as 0.055.

```formula
extras total for one unit = 0.055
price for one unit = 0.09 + 0.055 = 0.145
total price of the line before rounding = 3 × 0.145 = 0.435
total price of the line after currency rounding to two places = 0.44
```

The stored total price is **0.44**, not 0.43: the tie is broken away from zero, and the epsilon
correction defeats the binary representation of 0.435, which is a fraction below the tie.

### 1.5 Worked example — fractional quantity

The quantity is a decimal, so a half portion is representable.

```formula
extras total for one unit = 0.00
price for one unit = 6.80 + 0.00 = 6.80
total price of the line before rounding = 0.5 × 6.80 = 3.40
total price of the line after currency rounding = 3.40
```

The stored total price is **3.40**.

### 1.6 The extras summary

```formula
extras summary (text) = the names of the distinct extras of the line, joined by the three-character separator space, plus sign, space
```

For the extras named Olives and Extra cheese, in that order, the summary is the text
"Olives + Extra cheese". The summary is stored, so it survives the later deletion of an
extra.

---

## 2. Cart totals on the ordering screen

The screen's panel shows three amounts over the employee's current lines, that is their lines dated
today or later whose state is not cancelled.

```formula
cart total (currency) = round to two places of ( sum over every current line of ( total price of the line (currency) ) )

already paid subtotal (currency) = round to two places of ( sum over every current line whose state is not the to-order state of ( total price of the line (currency) ) )

still to pay subtotal (currency) = cart total (currency) − already paid subtotal (currency)
```

The two sums are each rounded to two places before the subtraction, and the difference is not
rounded again. All three are then formatted with exactly two decimal places for display.

### 2.1 Worked example

An employee has four current lines: a received line of 8.65, a sent line of 3.40, an ordered line of
12.00 and a to-order line of 6.80.

```formula
cart total = round to two places of ( 8.65 + 3.40 + 12.00 + 6.80 ) = round to two places of 30.85 = 30.85
already paid subtotal = round to two places of ( 8.65 + 3.40 + 12.00 ) = 24.05
still to pay subtotal = 30.85 − 24.05 = 6.80
```

The panel shows "Total 30.85", "Already Paid 24.05" and "To Pay 6.80".

### 2.2 The per-line figures in the panel

Each line in the panel shows the meal and, beneath it, one row per extra. The figures are **per
line**, not per unit, and the meal row excludes the extras:

```formula
meal figure of a panel line (currency) = round to two places of ( meal price (currency) ) × quantity (units), formatted to two decimal places

extra figure of a panel row (currency) = round to two places of ( extra price (currency) ) × quantity (units), formatted to two decimal places
```

The meal price and each extra price are rounded to two places **before** the multiplication, and the
product is formatted to two decimal places without a further rounding step. For the ordinary example
of section 1.3 the panel shows the meal at 21.60, the first extra at 0.90 and the second extra at
3.45, which add to the stored total price of 25.95.

---

## 3. The internal account balance

### 3.1 Rule

```formula
statement sum of an employee (currency) = sum over every row of the account statement whose employee is that employee of ( row amount (currency) )

balance of an employee (currency) = round to two places of ( statement sum of an employee (currency) ) + permitted overdraft of the employee's company (currency)
```

The permitted overdraft is added **after** the rounding, and is itself not rounded.

When the balance is asked for without the configured allowance — which is what the ordering screen
labels "Available Balance" — the second term is omitted:

```formula
available balance of an employee (currency) = round to two places of ( statement sum of an employee (currency) )
```

### 3.2 What the rows are

The rows come from the merged statement described in [`entities.md`](entities.md#8-lunch-cash-move-report-lunchcashmovereport):

- one positive row per manually recorded movement, carrying that movement's signed amount;
- one negative row per active order whose state is ordered or received, carrying the negation of the
  order's total price.

Orders in the to-order, sent and cancelled states contribute nothing, and archived orders contribute
nothing.

### 3.3 Currency handling

The rows are added irrespective of their currency, with no conversion. See rule
[MEAL-051](business-rules.md#meal-051--amounts-of-different-currencies-are-summed-without-conversion).
The permitted overdraft is read from the company of the employee's own user record, which is the
employee's main company, not the company currently active on the request.

### 3.4 Worked example — an ordinary balance

An employee has two credits of 100.00 and 50.00, one received order of 8.65 and one ordered order of
12.00. Their company permits an overdraft of 0.00.

```formula
statement sum = 100.00 + 50.00 + ( −8.65 ) + ( −12.00 ) = 129.35
balance = round to two places of 129.35 + 0.00 = 129.35
```

The ordering screen shows an available balance of **129.35** and the spending check sees **129.35**.

### 3.5 Worked example — the permitted overdraft in use

The same employee's company permits an overdraft of 200.00. The employee has one credit of 100.00
and confirms eleven pizzas at 9.00 each, a single line of total price 99.00, then a further line of
total price 210.00.

```formula
statement sum after the first line = 100.00 + ( −99.00 ) = 1.00
balance after the first line = round to two places of 1.00 + 200.00 = 201.00
statement sum after the second line = 1.00 + ( −210.00 ) = −209.00
balance after the second line = round to two places of ( −209.00 ) + 200.00 = −9.00
```

The first confirmation succeeds because 201.00 is not below zero. The second is refused by rule
[MEAL-020](business-rules.md#meal-020--the-balance-may-not-fall-below-the-permitted-overdraft),
because −9.00 is below zero, and the whole confirmation is rolled back.

### 3.6 Worked example — rounding before the allowance

An employee has three credits of 0.335, 0.335 and 0.335, recorded in a currency with three decimal
places, and no orders. The company permits an overdraft of 0.00.

```formula
statement sum = 0.335 + 0.335 + 0.335 = 1.005
balance = round to two places of 1.005 + 0.00 = 1.01
```

The balance is **1.01**, one thousandth above the true sum, because the rounding to two places
happens on the total and breaks the tie away from zero.

### 3.7 Worked example — the sent state removing a charge

An employee has a credit of 20.00 and one order of 9.00. Immediately after the cart is confirmed the
order is in the ordered state.

```formula
statement sum while the order is in the ordered state = 20.00 + ( −9.00 ) = 11.00
```

The administrator dispatches the day's orders, which moves the line to the sent state.

```formula
statement sum while the order is in the sent state = 20.00 = 20.00
```

The balance rises from 11.00 back to 20.00 until the line is marked received, at which point it
falls to 11.00 again. This is the compatibility finding recorded in
[`entities.md`](entities.md#82-nature-and-composition).

---

## 4. Whether one more of a line is affordable

The flag that shows or hides the add control on the order dialogue, and the repeat control on the
order list, is computed per line as follows.

```formula
uncommitted total of the employee for the line's date (currency) = sum over every active line of the same employee, with the same date, whose state is the to-order state of ( total price of that line (currency) )

spendable amount (currency) = balance of the employee including the permitted overdraft (currency) − uncommitted total of the employee for the line's date (currency)

the add control is shown when: spendable amount (currency) ≥ total price of the line (currency)
```

The comparison is a plain numeric comparison with no tolerance. The line being computed is itself
part of the uncommitted total whenever its own state is the to-order state, so for a line already in
the cart the flag answers the question "could a second line just like this one be afforded", which
is exactly what the increment control needs to know.

### 4.1 Worked example

An employee has a balance including the allowance of 40.00. They already have two to-order lines for
today, of 8.65 and 12.00, and they are looking at the dialogue of a third line whose total price
would be 25.95.

```formula
uncommitted total for today = 8.65 + 12.00 = 20.65
spendable amount = 40.00 − 20.65 = 19.35
19.35 ≥ 25.95 is false
```

The add control is hidden and the dialogue shows the wallet warning.

### 4.2 The client's own pre-check

The panel's increment control uses a slightly different figure, because it works per unit rather
than per line:

```formula
unit price of the panel line (currency) = round to two places of ( meal price (currency) ) + sum over the line's extras of ( round to two places of ( extra price (currency) ) )

the increment control is enabled when: the line is not in the sent or received state, and ( balance of the employee including the permitted overdraft (currency) − still to pay subtotal (currency) ) ≥ unit price of the panel line (currency)
```

For the example of section 2.1, with an allowance-inclusive balance of 40.00 and a still-to-pay
subtotal of 6.80, and a to-order line whose unit price is 6.80, the increment is enabled because
40.00 − 6.80 = 33.20 is at least 6.80.

---

## 5. The decimal hour and its clock time

Both the vendor's cut-off hour and the notice's notification hour are decimal counts of hours
between 0 and 12 inclusive, paired with a half-day marker. They are converted into a clock time as
follows.

### 5.1 Rule

1. When the decimal hour is exactly 12 and the marker is the afternoon half, the clock time is the
   last representable instant of the day: hour 23, minute 59, second 59 and 999999 microseconds.
   This is a deliberate special case, because 12 in the afternoon half would otherwise mean hour 24.
2. Otherwise the decimal hour is split into its whole part and its fractional part.
3. When the marker is the afternoon half, twelve is added to the whole part.
4. The minute is the fractional part multiplied by sixty and rounded to zero decimal places, half
   away from zero.
5. The second is zero.

```formula
whole hours = the integer part of the decimal hour
fraction of an hour = the decimal hour − whole hours
clock hour = whole hours + ( 12 when the marker is the afternoon half, otherwise 0 )
clock minute = round to zero places of ( 60 × fraction of an hour )
clock second = 0
```

### 5.2 Worked examples

| Decimal hour | Marker | Whole hours | Fraction | Minute before rounding | Clock time |
|---|---|---|---|---|---|
| 10.0 | morning | 10 | 0.0 | 0.0 | 10:00:00 |
| 11.0 | morning | 11 | 0.0 | 0.0 | 11:00:00 |
| 12.0 | morning | 12 | 0.0 | 0.0 | 12:00:00 |
| 12.0 | afternoon | — | — | — | 23:59:59.999999 by the special case |
| 0.0 | morning | 0 | 0.0 | 0.0 | 00:00:00 |
| 0.0 | afternoon | 0 | 0.0 | 0.0 | 12:00:00 |
| 10.5 | morning | 10 | 0.5 | 30.0 | 10:30:00 |
| 9.25 | afternoon | 9 | 0.25 | 15.0 | 21:15:00 |
| 11.75 | afternoon | 11 | 0.75 | 45.0 | 23:45:00 |
| 8.4 | morning | 8 | 0.4 | 24.0 | 08:24:00 |
| 7.99 | morning | 7 | 0.99 | 59.4 | 07:59:00 |

**Compatibility finding.** A fractional part at or above 0.991666… multiplies to at least 59.5 and
rounds to sixty, which is not a valid minute, and the conversion fails. The screen's hour widget
only ever produces whole minutes, so the value cannot be reached from a screen, but it can be
written directly. A corrected behaviour would carry the overflow into the hour, or would round the
whole decimal hour to the nearest minute before splitting it.

### 5.3 The inverse conversion

The inverse, used where a clock time must be expressed as a decimal hour, is:

```formula
decimal hour = round to two places of ( clock hour + ( clock minute ÷ 60 ) + ( clock second ÷ 3600 ) )
```

For 10:30:00 this yields 10.5; for 07:59:00 it yields 7.98, which is not the 7.99 that produced it,
because the forward conversion loses information below one minute.

---

## 6. Availability on a date

```formula
the vendor serves on a date when: ( the last service date is empty, or the date is strictly before the last service date ) and the vendor's flag for that date's weekday is set
```

The weekday is taken from the date itself. The seven flags are ordered Monday, Tuesday, Wednesday,
Thursday, Friday, Saturday, Sunday, and the flag chosen is the one whose position matches the
weekday number of the date, counting Monday as zero.

"Available today" is this rule applied to today's date read in the vendor's own time zone, not in
the reader's time zone.

### 6.1 Worked example

A vendor serves Monday to Friday, not Saturday, not Sunday, with a last service date of Monday
5 November 2018. Its time zone is the zero-offset universal zone.

| Instant examined | Date in the vendor's zone | Weekday | Flag | Before the last service date | Available |
|---|---|---|---|---|---|
| Monday 29 October 2018, 01:00 | 29 October | Monday | set | yes | yes |
| Monday 29 October 2018, 20:00 | 29 October | Monday | set | yes | yes |
| Saturday 3 November 2018, 10:00 | 3 November | Saturday | clear | yes | no |
| Sunday 4 November 2018, 13:00 | 4 November | Sunday | clear | yes | no |
| Monday 5 November 2018, 09:00 | 5 November | Monday | set | no, the date equals the last service date | no |
| Tuesday 6 November 2018, 09:00 | 6 November | Tuesday | set | no | no |

The last service date is therefore the first date of unavailability, not the last date of service,
whatever its label suggests.

### 6.2 The notice's own version

The notice uses the same shape with its own show-until date and its own seven flags:

```formula
the notice is displayed on a date when: ( the show-until date is empty, or the date is strictly before the show-until date ) and the notice's flag for that date's weekday is set
```

Unlike the vendor, the notice reads today's date in the **reader's** time zone when the flag is
computed, and in the notice's own time zone only when the scheduled action's next due instant is
computed.

---

## 7. Next dispatch instant

The same algorithm computes the next due instant of a vendor's dispatch action and of a notice's
push action. It is re-run on every change to the name, the archive flag, the channel or mode, the
hour, the half-day marker or the time zone.

### 7.1 Rule

1. Take today's date, read in the record's own time zone.
2. Take the clock time from the decimal hour and the half-day marker, by the rule of section 5.
3. Combine the two and attach the record's time zone, giving a candidate instant.
4. When the action has already run at least once and the candidate instant's date is on or before
   the date of the last run, read in the record's time zone, add one day to the candidate.
5. Otherwise, when the action has never run and the candidate instant is at or before the current
   instant, add one day to the candidate.
6. Convert the candidate to the zero-offset universal zone and store it as the next due instant,
   without a zone marker.

```formula
candidate instant = today in the record's time zone, at the clock time of the decimal hour, in the record's time zone

next due instant = candidate instant + ( one day when the action has run and the candidate's date ≤ the last run's date in the record's time zone )
                                     + ( one day when the action has never run and the candidate instant ≤ the current instant )

stored next due instant = next due instant expressed in the zero-offset universal zone
```

At most one of the two day additions can apply, because they test mutually exclusive conditions.

### 7.2 Worked example — a vendor five hours behind universal time

The current instant is Friday 29 January 2021 at 12:20:00 universal time. The vendor's time zone is
five hours behind universal time. Its cut-off hour is 10.0 in the morning half. The action has never
run.

```formula
today in the vendor's zone = 29 January 2021
clock time of 10.0 morning = 10:00:00
candidate instant = 29 January 2021 10:00:00 in a zone five hours behind universal time = 29 January 2021 15:00:00 universal
the action has never run, and 15:00:00 universal is after the current 12:20:00 universal, so no day is added
stored next due instant = 29 January 2021 15:00:00
```

### 7.3 Worked example — a notice nine hours ahead of universal time

The same current instant. The notice's time zone is nine hours ahead of universal time. Its
notification hour is 8.0 in the morning half. The action has never run.

```formula
today in the notice's zone = 29 January 2021, because 12:20 universal is 21:20 local
clock time of 8.0 morning = 08:00:00
candidate instant = 29 January 2021 08:00:00 in a zone nine hours ahead of universal time = 28 January 2021 23:00:00 universal
the action has never run, and 28 January 23:00:00 universal is before the current 29 January 12:20:00 universal, so one day is added
stored next due instant = 29 January 2021 23:00:00
```

That instant is 30 January at 08:00 in the notice's own zone: the first future occurrence of the
notification hour.

### 7.4 Worked example — moving the hour backwards after the action has run

The vendor of section 7.2 has its cut-off hour lowered by five, from 10.0 to 5.0, both in the
morning half, while the action has still never run.

```formula
clock time of 5.0 morning = 05:00:00
candidate instant = 29 January 2021 05:00:00 local = 29 January 2021 10:00:00 universal
10:00:00 universal is before the current 12:20:00 universal, so one day is added
stored next due instant = 30 January 2021 10:00:00
```

The action now runs five hours earlier, one day later than the instant it previously held.

### 7.5 Worked example — moving the hour after the action has run today

The action of section 7.4 runs at 30 January 2021 10:00:00 universal, so its last-run instant is
that value and its next due instant is 31 January 2021 10:00:00. On 29 January, with the clock
frozen at 12:20 universal for the purpose of the example, the cut-off hour is then raised by seven,
from 5.0 to 12.0, still in the morning half.

```formula
today in the vendor's zone = 29 January 2021
clock time of 12.0 morning = 12:00:00
candidate instant = 29 January 2021 12:00:00 local = 29 January 2021 17:00:00 universal
the action has run; the last run, read in the vendor's zone, falls on 29 January 2021; the candidate's date, 29 January, is on or before it, so one day is added
stored next due instant = 30 January 2021 17:00:00
```

Lowering the hour by one further, from 12.0 to 11.0, yields 30 January 2021 16:00:00 by the same
path.

---

## 8. Whether the cut-off has passed

### 8.1 For the vendor

```formula
for a vendor whose channel is electronic mail:
cut-off passed = the vendor is available today and the current instant, read in the vendor's time zone, is strictly after today's date in that zone at the clock time of the cut-off hour

for a vendor whose channel is telephone:
cut-off passed = the vendor is not available today
```

A telephone vendor therefore never reports a passed cut-off while it is serving, whatever the hour,
because there is no automatic dispatch to be late for.

### 8.2 For the order line

```formula
cut-off passed for a line whose order date is before today = true
cut-off passed for a line whose order date is today = the vendor's own cut-off flag
cut-off passed for a line whose order date is after today = false
```

Today is read in the reading user's time zone here, not in the vendor's.

### 8.3 Worked example

An electronic mail vendor whose time zone is one hour ahead of universal time has a cut-off hour of
11.0 in the morning half and serves on Mondays. The reading user is in the same zone.

| Current instant, universal | Local instant | Available today | Cut-off instant, local | Cut-off passed | A line dated today may be added |
|---|---|---|---|---|---|
| Monday 09:00 | Monday 10:00 | yes | Monday 11:00 | no | yes |
| Monday 10:30 | Monday 11:30 | yes | Monday 11:00 | yes | no |
| Sunday 10:30 | Sunday 11:30 | no | — | no | yes, because the flag is false when the vendor is unavailable; the confirmation is refused later by rule MEAL-001 |

The third row is worth stating plainly: for an electronic mail vendor the cut-off flag is false on a
day the vendor does not serve, so the dialogue does **not** warn. The refusal only arrives when the
cart is confirmed.

---

## 9. The audience cut-off of a pushed notice

```formula
audience cut-off date = today − ( 1 week when the audience is the last-week audience, 4 weeks when it is the last-month audience, 52 weeks when it is the last-year audience )
```

Today is read without a time zone adjustment, as the server's own date. The everyone audience
applies no date narrowing at all.

### 9.1 Worked example

Today is Friday 29 January 2021.

| Audience | Weeks subtracted | Cut-off date | Orders considered |
|---|---|---|---|
| everyone | none | none | every order whose state is not cancelled |
| last week | 1 | 22 January 2021 | orders dated 22 January 2021 or later |
| last month | 4 | 1 January 2021 | orders dated 1 January 2021 or later |
| last year | 52 | 31 January 2020 | orders dated 31 January 2020 or later |

The last-month audience is four weeks, not a calendar month, and the last-year audience is
fifty-two weeks, which is one or two days short of a calendar year.

---

## 10. Counting the day's orders for the grouped controls

The two grouped controls on the vendor line of the order list are driven by one count.

```formula
ordered count of a vendor = the number of active order lines of that vendor dated today whose state is the ordered state
sent count of a vendor = the number of active order lines of that vendor dated today whose state is the sent state

the dispatch control is shown when: ordered count of the vendor ≥ 1
the receipt control is shown when: sent count of the vendor ≥ 1
```

Today is read in the reading user's time zone. Cancelled, to-order and received lines are counted by
neither.

### 10.1 Worked example

A vendor has, for today, three ordered lines, one sent line, two received lines and one cancelled
line, all active.

```formula
ordered count = 3, so the dispatch control is shown
sent count = 1, so the receipt control is shown
```

Both controls appear on the vendor's group header at the same time.

---

## 11. The total of the vendor order message

```formula
message total (currency) = sum over every line of the day's order for that vendor of ( total price of the line (currency) )
```

The sum is taken at full precision over values that are each already rounded to the currency's
precision, and is then formatted in the currency of the **first** collected line. The lines are
collected sorted by employee and then by meal, so the first line is the one of the alphabetically
first employee.

### 11.1 Worked example

A vendor's day holds three lines: 9.00, 9.30 and 3.00.

```formula
message total = 9.00 + 9.30 + 3.00 = 21.30
```

The message footer prints **21.30** in the currency of the first line.

---

## 12. Precision summary

| Quantity | Held as | Precision | Rounded when |
|---|---|---|---|
| Meal price | decimal with the accounting decimal precision | two places unless the installation changed it | on write |
| Extra price | monetary | the currency's decimal places | on write |
| Order total price | monetary | the currency's decimal places | on write, after the multiplication |
| Order quantity | decimal | full working precision | never |
| Movement amount | decimal | full working precision | never; only the balance rounds |
| Account balance | derived | two places, then the allowance is added unrounded | when the balance is computed |
| Cart total, already paid, still to pay | derived | two places on each of the first two, the difference unrounded | when the panel is built |
| Cut-off hour, notification hour | decimal hour | full working precision, constrained to the closed range zero to twelve | never |
| Clock minute derived from a decimal hour | whole number | zero places | during the conversion |
| Next due instant | instant | one second | never; the seconds are always zero except for the afternoon-twelve special case |
