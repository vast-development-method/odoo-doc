# Meal Ordering — Acceptance criteria

Numbered scenarios in Given, When, Then form. Every scenario states concrete starting records, a
concrete operation with concrete inputs, and the exact resulting records, amounts, quantities and
states. Amounts are in the currency named in the fixture, which has two decimal places and a
rounding step of 0.01, unless the scenario says otherwise.

Scenario numbers are stable. A scenario keeps its number even when the surrounding text is
rewritten.

---

## 0. The fixture

Unless a scenario overrides part of it, every scenario below starts from this state.

### 0.1 Companies and currencies

| Record | Value |
|---|---|
| Company N | Named "Northern Office". Its currency has two decimal places and a rounding step of 0.01. Its permitted overdraft is 0.00. Its delivery notice message is the packaged default, the two lines "Your lunch has been delivered." and "Enjoy your meal!". |
| Company S | Named "Southern Office". Its currency has two decimal places and a rounding step of 0.01, and is a different currency from Company N's. Its permitted overdraft is 0.00. |

### 0.2 People

| Record | Value |
|---|---|
| Dana | An internal user of Company N holding the ordering privilege. Time zone: the zero-offset universal zone. Language: the installation's base language. |
| Erin | An internal user of Company N holding the ordering privilege. Time zone: the zero-offset universal zone. Language: a second installed language. |
| Mark | An internal user of Company N holding the administration privilege, which implies the ordering privilege. Time zone: the zero-offset universal zone. |
| Sam | An internal user of Company S holding the ordering privilege. |

### 0.3 Locations

| Record | Name | Address | Company |
|---|---|---|---|
| Location F1 | "Farm 1" | "1 Mill Lane" | empty |
| Location F2 | "Farm 2" | "2 Mill Lane" | empty |
| Location SS | "Southern Site" | "9 Harbour Road" | Company S |

Dana's and Erin's current location is Location F1. Mark's current location is Location F2. Sam's
current location is Location SS.

### 0.4 Vendors

| Record | Contact | Channel | Cut-off hour | Half-day | Time zone | Weekdays served | Last service date | Served locations | Company |
|---|---|---|---|---|---|---|---|---|---|
| Vendor P, "Pizza Inn" | Contact "Pizza Inn" | electronic mail | 11.0 | morning | zero-offset universal | Monday to Friday | none | Location F1 and Location F2 | empty |
| Vendor K, "Kothai" | Contact "Kothai" | electronic mail | 10.0 | morning | five hours behind universal | Monday to Friday | none | none declared | empty |
| Vendor C, "Coin Gourmand" | Contact "Coin Gourmand" | telephone | 12.0 | morning | zero-offset universal | Monday to Friday | none | Location F1 and Location F2 | empty |

Vendor P's three extra groups are labelled "Extras", "Beverages" and "Extra Label 3", all three with
the none-or-more discipline. Mark is the responsible on all three vendors.

### 0.5 Catalogue

| Record | Name | Category | Vendor | Price |
|---|---|---|---|---|
| Meal PZ | "Pizza" | Category "Pizza" | Vendor P | 9.00 |
| Meal TS | "Tuna Sandwich" | Category "Sandwich" | Vendor C | 3.00 |
| Meal KC | "Kothai Curry" | Category "Sandwich" | Vendor K | 12.00 |

| Record | Name | Vendor | Group | Price |
|---|---|---|---|---|
| Extra OL | "Olives" | Vendor P | 1 | 0.30 |
| Extra CH | "Extra cheese" | Vendor P | 1 | 1.15 |
| Extra WA | "Water" | Vendor P | 2 | 1.00 |

### 0.6 Accounts

Dana has one manual movement of 100.00 dated 26 October 2018. Erin has one manual movement of 20.00
dated 26 October 2018. Sam has one manual movement of 50.00 in Company S's currency.

### 0.7 Clocks

Two reference instants are used.

| Clock | Instant | Notes |
|---|---|---|
| Clock A | Monday 29 October 2018 at 10:00:00 universal time | Before Vendor P's cut-off of 11:00 and before Vendor C's of 12:00. |
| Clock B | Friday 29 January 2021 at 12:20:00 universal time | Used for the scheduled-action scenarios. |

Scenarios say which clock they use.

---

## 1. Catalogue and availability

### Scenario 1.1 — The catalogue shows the meals available at the employee's location

- **Given** the fixture and Clock A, and Dana's current location is Location F1.
- **When** Dana opens the ordering screen.
- **Then** the endpoint at the route `/lunch/user_location_get` returns Location F1, and the
  catalogue lists Meal PZ, whose vendor serves Location F1, and Meal TS, whose vendor serves
  Location F1, and Meal KC, whose vendor declares no location at all.
- **And** the panel shows "Available Balance 100.00".

### Scenario 1.2 — A vendor that serves no location serves every location

- **Given** the fixture and Clock A, and Sam's current location is Location SS.
- **When** Sam opens the ordering screen.
- **Then** the catalogue lists Meal KC, because Vendor K declares no served location, and does not
  list Meal PZ or Meal TS, whose vendors serve only Location F1 and Location F2.

### Scenario 1.3 — No location at all

- **Given** the fixture with Location F1, Location F2 and Location SS all deleted, and Clock A.
- **When** Dana opens the ordering screen.
- **Then** the catalogue lists nothing and shows the two lines "No location found" and "Please
  create a location to start ordering."
- **And** the panel shows "No lunch location available." in place of the location selector.

### Scenario 1.4 — The availability filter on a served weekday

- **Given** the fixture and Clock A, which falls on a Monday.
- **When** Dana applies the filter "Available Today".
- **Then** Meal PZ, Meal TS and Meal KC are all listed, because all three vendors serve Mondays.

### Scenario 1.5 — The availability filter on an unserved weekday

- **Given** the fixture, with the clock at Saturday 3 November 2018 at 10:00:00 universal time.
- **When** Dana applies the filter "Available Today".
- **Then** no meal is listed, because none of the three vendors serves Saturdays.

### Scenario 1.6 — Choosing a date switches the weekday filter

- **Given** the fixture and Clock A, with the filter "Monday" active because today is a Monday.
- **When** Dana picks Saturday 3 November 2018 in the panel's date picker.
- **Then** the filter "Monday" is switched off and the filter "Saturday" is switched on, and the
  catalogue lists nothing.

### Scenario 1.7 — The last service date excludes its own day

- **Given** the fixture with Vendor P's last service date set to Monday 5 November 2018, and the
  clock at Monday 5 November 2018 at 09:00:00 universal time.
- **When** the availability of Vendor P is read.
- **Then** it is false, because the date is on the last service date, even though Monday is a served
  weekday.
- **And** with the clock at Friday 2 November 2018 at 09:00:00 universal time the availability is
  true.

### Scenario 1.8 — The novelty ribbon

- **Given** the fixture and Clock A, with Meal PZ's novelty date set to Monday 29 October 2018.
- **When** Dana views the catalogue.
- **Then** Meal PZ carries the pill "New", because today is on the novelty date and the comparison is
  inclusive.
- **And** with the clock at Tuesday 30 October 2018 the pill is absent.

### Scenario 1.9 — Marking a favourite writes on the employee, not on the meal

- **Given** the fixture and Clock A.
- **When** Dana clicks the favourite marker on Meal PZ.
- **Then** Dana appears in Meal PZ's favourite list and Meal PZ appears in Dana's favourite meal
  list, and the two are the same association row in the table `lunch_product_favorite_user_rel`.
- **And** Erin, viewing the same catalogue, sees the marker unset on Meal PZ, because the flag is
  computed per reading user.

### Scenario 1.10 — The card picture falls back to the category picture

- **Given** the fixture with Meal TS carrying no picture and Category "Sandwich" carrying one.
- **When** Dana views the catalogue.
- **Then** Meal TS's card shows the category's thumbnail.
- **And** when Category "Sandwich" also carries no picture, the card shows the packaged default meal
  picture.

---

## 2. Building the cart

### Scenario 2.1 — Adding a plain line

- **Given** the fixture and Clock A.
- **When** Dana opens Meal PZ, chooses no extras, writes no note and presses "Add To Cart".
- **Then** one Lunch Order exists with the meal Meal PZ, the employee Dana, the date Monday
  29 October 2018, the quantity 1, the location Location F1, the vendor Vendor P, the category
  "Pizza", the state `new`, the archive flag true, the delivery-notice flag false and the total
  price 9.00.
- **And** the extras summary is empty.
- **And** the account statement for Dana still shows one row of 100.00, because a to-order line
  contributes nothing.

### Scenario 2.2 — Adding a line with extras

- **Given** the fixture and Clock A.
- **When** Dana opens Meal PZ, ticks Extra OL and Extra CH in the first block, and presses "Add To
  Cart".
- **Then** one Lunch Order exists with the quantity 1 and the total price 10.45, because
  9.00 + 0.30 + 1.15 = 10.45.
- **And** the extras summary reads "Olives + Extra cheese".

### Scenario 2.3 — Adding the same line twice merges it

- **Given** the result of Scenario 2.1: one to-order line of Meal PZ, quantity 1, price 9.00.
- **When** Dana opens Meal PZ again, chooses no extras, writes no note and presses "Add To Cart".
- **Then** **no** second Lunch Order is created. The existing line's quantity becomes 2 and its total
  price becomes 18.00.

### Scenario 2.4 — A requested quantity is ignored by the merge

- **Given** the result of Scenario 2.1: one to-order line of Meal PZ, quantity 1.
- **When** an administrator creates a line for Dana, Meal PZ, the same date, no note, Location F1
  and the quantity **5**.
- **Then** no second line is created, and the existing line's quantity becomes **2**, not 6: the
  merge always adds exactly one.
- **And** the total price becomes 18.00.

### Scenario 2.5 — A different note prevents the merge

- **Given** the result of Scenario 2.1.
- **When** Dana adds Meal PZ again with the note "No chilli".
- **Then** a second Lunch Order is created with the quantity 1, the note "No chilli" and the total
  price 9.00, and the first line keeps the quantity 1.

### Scenario 2.6 — A different set of extras prevents the merge

- **Given** the result of Scenario 2.1.
- **When** Dana adds Meal PZ again with Extra OL ticked.
- **Then** a second Lunch Order is created with the quantity 1 and the total price 9.30.

### Scenario 2.7 — A different location prevents the merge

- **Given** the result of Scenario 2.1, with Dana's line at Location F1.
- **When** Dana changes her current location to Location F2 and adds Meal PZ again with no extras
  and no note.
- **Then** a second Lunch Order is created at Location F2 with the quantity 1.

### Scenario 2.8 — A different date prevents the merge

- **Given** the result of Scenario 2.1, dated Monday 29 October 2018.
- **When** Dana picks Tuesday 30 October 2018 in the date picker and adds Meal PZ again.
- **Then** a second Lunch Order is created dated Tuesday 30 October 2018 with the quantity 1.

### Scenario 2.9 — An ordered line is never a merge target on creation

- **Given** the fixture and Clock A. Dana creates one line of Meal PZ, quantity 1, and confirms her
  cart, so the line is in the `ordered` state with the quantity 1.
- **When** Dana adds Meal PZ again with the same extras, note and location.
- **Then** a **second** Lunch Order is created in the `new` state with the quantity 1, and the
  ordered line keeps the quantity 1.
- **And** Dana's account statement shows two rows: 100.00 and −9.00. The to-order line contributes
  nothing.

### Scenario 2.10 — A received line is never a merge target on creation

- **Given** the fixture and Clock A. Dana has one line of Meal PZ with the note "Pizza" in the
  `confirmed` state with the quantity 1.
- **When** Dana creates a second line of Meal PZ with the note "Pizza", the same location and the
  same date.
- **Then** both lines exist, both have the archive flag true and both have the quantity 1.

### Scenario 2.11 — Incrementing from the panel

- **Given** the result of Scenario 2.1: one to-order line of quantity 1 and price 9.00, and Dana's
  balance is 100.00.
- **When** Dana presses the increment control on that line.
- **Then** the quantity becomes 2 and the total price becomes 18.00.
- **And** the balance is still 100.00, because a to-order line contributes nothing to the statement.

### Scenario 2.12 — Decrementing from two to one

- **Given** the result of Scenario 2.11: one to-order line of quantity 2 and price 18.00.
- **When** Dana presses the decrement control.
- **Then** the quantity becomes 1 and the total price becomes 9.00, and the archive flag is still
  true.

### Scenario 2.13 — Decrementing from one archives the line

- **Given** the result of Scenario 2.12: one to-order line of quantity 1.
- **When** Dana presses the decrement control.
- **Then** the line's archive flag becomes false and its quantity stays at **1**, not zero.
- **And** the line disappears from the panel and from the account statement.

### Scenario 2.14 — Decrementing a sent line does nothing

- **Given** one line of Meal PZ in the `sent` state with the quantity 3.
- **When** the quantity operation is invoked on it with a step of minus one.
- **Then** the quantity is still 3 and the archive flag is still true, because the operation skips
  lines in the sent and received states.

### Scenario 2.15 — Emptying the cart

- **Given** the fixture and Clock A. Dana has three current lines: one in the `new` state of 9.00,
  one in the `ordered` state of 3.00 and one in the `sent` state of 12.00.
- **When** Dana presses "Clear Order", which calls the endpoint at the route `/lunch/trash`.
- **Then** the `new` line and the `ordered` line are first moved to the `cancelled` state and then
  deleted. The `sent` line is untouched.
- **And** Dana's account statement shows 100.00 alone, because the ordered charge has gone and the
  sent charge never appeared.

---

## 3. The extras discipline

### Scenario 3.1 — None or more accepts none

- **Given** the fixture, where Vendor P's first group has the none-or-more discipline and holds
  Extra OL and Extra CH.
- **When** Dana adds Meal PZ with no extras.
- **Then** the line is created with the total price 9.00 and no refusal is raised.

### Scenario 3.2 — One or more refuses none

- **Given** the fixture with Vendor P's first group discipline changed to one-or-more and its label
  left as "Extras".
- **When** Dana adds Meal PZ with no extras.
- **Then** the creation is refused with the message "You should order at least one Extras", the
  placeholder having been replaced by the vendor's first extra label.

### Scenario 3.3 — One or more accepts one

- **Given** the state of Scenario 3.2.
- **When** Dana adds Meal PZ with Extra OL ticked.
- **Then** the line is created with the total price 9.30.

### Scenario 3.4 — One or more accepts two

- **Given** the state of Scenario 3.2.
- **When** Dana adds Meal PZ with Extra OL and Extra CH ticked.
- **Then** the line is created with the total price 10.45.

### Scenario 3.5 — Exactly one refuses none

- **Given** the fixture with Vendor P's first group discipline changed to exactly-one and its label
  changed to "Sauce".
- **When** Dana adds Meal PZ with no extras.
- **Then** the creation is refused with the message "You have to order one and only one Sauce".

### Scenario 3.6 — Exactly one refuses two

- **Given** the state of Scenario 3.5.
- **When** Dana adds Meal PZ with Extra OL and Extra CH ticked.
- **Then** the creation is refused with the same message "You have to order one and only one Sauce".

### Scenario 3.7 — A discipline on an empty group is suspended

- **Given** the fixture with Vendor P's **third** group discipline set to exactly-one and its label
  set to "Dessert", and no extra existing in that group.
- **When** Dana adds Meal PZ with no extras of any group.
- **Then** the line is created and no refusal is raised, because the offered-block flag for group
  three is false.

### Scenario 3.8 — Two disciplines fire in group order

- **Given** the fixture with Vendor P's first group discipline set to one-or-more with the label
  "Extras", and its second group discipline set to exactly-one with the label "Beverages" and
  holding Extra WA.
- **When** Dana adds Meal PZ with no extras at all.
- **Then** the refusal names the **first** group: "You should order at least one Extras", because
  the groups are evaluated in ascending order and the first failure ends the evaluation.

### Scenario 3.9 — An extra written through the second list takes group two

- **Given** the fixture.
- **When** Mark adds an extra named "Soda" priced 1.20 to Vendor P's **second** list.
- **Then** the created Lunch Topping carries the group number 2, whatever number the incoming values
  carried.

### Scenario 3.10 — Deleting an extra from the second list

- **Given** a vendor whose second list holds one extra named "salt" priced 7.00 and whose third list
  holds one extra named "sugar" priced 10.00.
- **When** Mark deletes the second list's extra.
- **Then** the vendor's second list is empty and the third list still holds "sugar". The deletion
  command carries no values, so the group-forcing rule leaves it alone.
- **And** deleting the third list's extra empties that list too.

---

## 4. The internal account and the permitted overdraft

### Scenario 4.1 — An ordinary balance

- **Given** the fixture and Clock A. Dana has one credit of 100.00, one received line of 8.65 and
  one ordered line of 12.00.
- **When** the balance is read with the permitted overdraft included, Company N permitting 0.00.
- **Then** it is 79.35, because 100.00 − 8.65 − 12.00 = 79.35 and the allowance adds nothing.

### Scenario 4.2 — Confirming within the balance

- **Given** the fixture and Clock A. Dana's balance is 100.00 and she has one to-order line of Meal
  PZ, quantity 1, price 9.00.
- **When** Dana presses "Order Now".
- **Then** the line moves to the `ordered` state, the statement gains a row of −9.00 and the balance
  becomes 91.00.

### Scenario 4.3 — Confirming beyond the balance is refused

- **Given** the fixture and Clock A, with Erin's balance at 20.00 and Company N permitting 0.00.
  Erin has one to-order line of Meal KC, quantity 2, price 24.00.
- **When** Erin presses "Order Now".
- **Then** the confirmation is refused with the message "Oh no! You don’t have enough money in your
  wallet to order your selected lunch! Contact your lunch manager to add some money to your wallet."
- **And** the line is still in the `new` state, the statement is unchanged and the balance is still
  20.00, because the whole transaction is rolled back.

### Scenario 4.4 — The permitted overdraft lets the confirmation through

- **Given** the state of Scenario 4.3, with Company N's permitted overdraft raised to 200.00.
- **When** Erin presses "Order Now".
- **Then** the line moves to the `ordered` state. The statement sum is 20.00 − 24.00 = −4.00 and the
  balance is −4.00 + 200.00 = 196.00, which is not below zero.

### Scenario 4.5 — Eleven pizzas against a two-hundred allowance

- **Given** the fixture and Clock A, with Company N's permitted overdraft set to 200.00 and Dana's
  only credit removed so that her statement sum is 0.00 before the order. Dana has a to-order line
  of Meal PZ with the quantity 11, total price 99.00.
- **When** the balance-allows-one-more flag is read on that line.
- **Then** it is true: the uncommitted total for the date is 99.00, the balance including the
  allowance is 200.00, the spendable amount is 200.00 − 99.00 = 101.00, and 101.00 is at least
  99.00.
- **When** Dana presses "Order Now".
- **Then** the line moves to the `ordered` state, the statement sum becomes −99.00 and the balance
  becomes 101.00.

### Scenario 4.6 — The add control hides before the balance is exhausted

- **Given** the fixture and Clock A. Dana's balance including the allowance is 40.00 and she already
  has two to-order lines for today, of 8.65 and 12.00.
- **When** Dana opens Meal PZ with Extra OL and Extra CH and a quantity that would give a total price
  of 25.95.
- **Then** the add control is hidden and the dialogue shows "Your wallet does not contain enough
  money to order that. To add some money to your wallet, please contact your lunch manager.",
  because the spendable amount is 40.00 − 20.65 = 19.35 and 19.35 is below 25.95.

### Scenario 4.7 — The increment control's own pre-check

- **Given** the fixture and Clock A. Dana's balance including the allowance is 40.00, her still-to-
  pay subtotal is 6.80, and she has a to-order line of a meal priced 6.80 with no extras.
- **When** the panel decides whether the increment control is enabled.
- **Then** it is enabled, because 40.00 − 6.80 = 33.20 is at least the unit price of 6.80.

### Scenario 4.8 — The increment control refuses at the boundary

- **Given** the same state with the balance including the allowance at 12.00.
- **When** the panel decides whether the increment control is enabled.
- **Then** the control is disabled, because the spendable figure is 12.00 − 6.80 = 5.20 and 5.20 is
  below the unit price of 6.80.

### Scenario 4.9 — Exactly at zero is allowed

- **Given** the fixture and Clock A, with Erin's statement sum at 9.00 and Company N permitting
  0.00, and one to-order line of Meal PZ priced 9.00.
- **When** Erin presses "Order Now".
- **Then** the confirmation succeeds. The statement sum becomes 0.00 and the balance is 0.00, which
  is not strictly below zero.

### Scenario 4.10 — One hundredth below zero is refused

- **Given** the same state with Erin's statement sum at 8.99.
- **When** Erin presses "Order Now".
- **Then** the confirmation is refused with the wallet message, because the statement sum would
  become −0.01 and the balance −0.01 is below zero.

### Scenario 4.11 — The sent state returns the money

- **Given** the fixture and Clock A. Dana has a credit of 20.00 and one line of Meal PZ in the
  `ordered` state of 9.00, so her balance is 11.00.
- **When** Mark dispatches Vendor P's day and the line moves to the `sent` state.
- **Then** Dana's balance is 20.00 again, because the statement counts only the ordered and received
  states.
- **When** Mark marks the line received.
- **Then** Dana's balance is 11.00 again.

### Scenario 4.12 — A credit and a correction

- **Given** the fixture and Clock A, with Dana's statement holding one credit of 100.00.
- **When** Mark records a movement for Dana of −15.00 with the description "Correction, cash short".
- **Then** Dana's statement holds two rows, 100.00 and −15.00, and her balance is 85.00.

### Scenario 4.13 — Rounding before the allowance

- **Given** an employee with three credits of 0.335 each, recorded in a currency with three decimal
  places, no orders, and a company permitting 0.00.
- **When** the balance is read.
- **Then** it is 1.01, because the sum 1.005 is rounded to two places away from zero before the
  allowance of 0.00 is added.

### Scenario 4.14 — Currencies are added without conversion

- **Given** an employee with one credit of 100.00 in Company N's currency and one credit of 100.00
  in Company S's currency, and a company permitting 0.00.
- **When** the balance is read.
- **Then** it is 200.00. No conversion is performed and no currency is attached to the result. This
  is the observed behaviour recorded as rule MEAL-051.

### Scenario 4.15 — The statement description of an order charge

- **Given** the fixture and Clock A. Dana has one line of Meal PZ in the `ordered` state, quantity 2,
  with Extra OL and Extra CH.
- **When** Dana's statement is read.
- **Then** it holds a row whose amount is −20.90 and whose description is "Order: 2 x Pizza Olives +
  Extra cheese": the fixed word "Order:", a space, the quantity, " x ", the meal name in the
  installation's base language, a space, and the stored extras summary.

### Scenario 4.16 — An archived line leaves the statement

- **Given** the state of Scenario 4.15, where Dana's balance is 100.00 − 20.90 = 79.10.
- **When** the line's archive flag is set to false.
- **Then** Dana's balance returns to 100.00.

---

## 5. The order pipeline

### Scenario 5.1 — The whole ordinary path

- **Given** the fixture and Clock A.
- **When** Dana adds Meal PZ with Extra OL, quantity 1, and presses "Order Now"; then Mark presses
  "Send Orders" on Vendor P's group; then Mark presses "Confirm Orders" on the same group; then Mark
  presses the notify control on the line.
- **Then** the line passes through the states `new`, `ordered`, `sent` and `confirmed` in that
  order, its total price stays 9.30 throughout, its delivery-notice flag becomes true at the last
  step, and Dana's balance moves 100.00, 90.70, 100.00, 90.70, 90.70.

### Scenario 5.2 — Confirming when the vendor does not serve the order date

- **Given** the fixture, with the clock at Friday 2 November 2018 at 10:00:00 universal time, and
  Dana holding one to-order line of Meal PZ dated **Saturday 3 November 2018**.
- **When** Dana presses "Order Now".
- **Then** the confirmation is refused with the message "The vendor related to this order is not
  available at the selected date.", and the line stays in the `new` state.

### Scenario 5.3 — Confirming an archived meal

- **Given** the fixture and Clock A, with Dana holding one to-order line of Meal PZ and Meal PZ
  archived afterwards by Mark.
- **When** Dana presses "Order Now".
- **Then** the confirmation is refused with the message "Product is no longer available.", and the
  line stays in the `new` state.

### Scenario 5.4 — One bad line refuses the whole cart

- **Given** the fixture, with the clock at Friday 2 November 2018 at 10:00:00 universal time, and
  Dana holding two to-order lines: one of Meal PZ dated Friday 2 November 2018 and one of Meal PZ
  dated Saturday 3 November 2018.
- **When** Dana presses "Order Now".
- **Then** the confirmation is refused with the date message and **both** lines stay in the `new`
  state.

### Scenario 5.5 — Confirming merges into an existing ordered line

- **Given** the fixture and Clock A. Dana has one line of Meal TS in the `ordered` state with the
  quantity 1 and no extras and no note, and one line of Meal TS in the `new` state with the
  quantity 2, the same location, no extras and no note.
- **When** Dana presses "Order Now".
- **Then** the to-order line is archived and the ordered line's quantity becomes 3, with the total
  price 9.00. Only one active ordered line of Meal TS remains.

### Scenario 5.6 — Confirming beside a sent line loses the quantity

- **Given** the fixture and Clock A. Dana has one line of Meal PZ in the `sent` state with the
  quantity 1, and one line of Meal PZ in the `new` state with the quantity 2, the same location,
  extras and note.
- **When** Dana presses "Order Now".
- **Then** the to-order line is archived with its state left at `new`, and the sent line's quantity
  stays at **1**. The two pizzas are lost. This is the compatibility finding recorded in
  [`state-machines.md`](state-machines.md#18-merge-on-write-and-what-it-does-to-a-state-change).
- **And** a corrected implementation would leave the to-order line active and move it to `ordered`,
  giving one sent line of quantity 1 and one ordered line of quantity 2.

### Scenario 5.7 — Receiving a line twice does not archive it

- **Given** the fixture and Clock A, with Dana holding one line of Meal PZ in the `sent` state,
  quantity 1, note "Pizza".
- **When** Mark presses the receipt control, and then presses it again.
- **Then** the line is in the `confirmed` state, its archive flag is true and its quantity is 1. The
  merge search excludes the line itself, so the second write finds nothing to merge into.

### Scenario 5.8 — Cancelling and resetting

- **Given** the fixture and Clock A, with Dana holding one line of Meal PZ in the `ordered` state of
  9.00, so her balance is 91.00.
- **When** Mark presses the cancel control.
- **Then** the line is in the `cancelled` state and Dana's balance is 100.00.
- **When** Mark presses the reset control.
- **Then** the line is in the `ordered` state again and Dana's balance is 91.00.

### Scenario 5.9 — Repeating a received line

- **Given** the fixture and Clock A, with Dana holding one line of Meal PZ in the `confirmed` state
  dated Friday 26 October 2018, quantity 2, with Extra OL, note "No chilli", location Location F1,
  total price 18.60.
- **When** Dana presses the repeat control.
- **Then** a new Lunch Order is created dated Monday 29 October 2018, already in the `ordered`
  state, quantity 2, with Extra OL, note "No chilli", location Location F1 and total price 18.60.
  The original line is unchanged.
- **And** the screen navigates to the personal order list.

### Scenario 5.10 — Repeating when the vendor does not serve today

- **Given** the fixture with the clock at Saturday 3 November 2018 at 10:00:00 universal time, and
  Dana holding one received line of Meal PZ.
- **When** Dana presses the repeat control.
- **Then** it is refused with the message "The vendor related to this order is not available today."
  and no copy is made.

### Scenario 5.11 — A repeat inherits the delivery-notice flag

- **Given** the state of Scenario 5.9 with the original line's delivery-notice flag already true.
- **When** Dana presses the repeat control and Mark later marks the copy received and presses the
  notify control.
- **Then** nothing is pushed, because the copy's delivery-notice flag was copied as true and the
  notify operation drops every line whose flag is set. This is rule MEAL-045.

### Scenario 5.12 — Clearing a weekday cancels future orders

- **Given** the fixture and Clock A. Dana holds four lines of Meal PZ: one dated Monday 29 October
  2018 in the `new` state; one dated Wednesday 31 October 2018 in the `ordered` state; one dated
  Wednesday 31 October 2018 in the `sent` state; one dated Wednesday 24 October 2018 in the
  `ordered` state.
- **When** Mark clears Vendor P's Wednesday flag.
- **Then** the second line moves to the `cancelled` state. The first is untouched because it falls
  on a Monday, the third because it is already sent, and the fourth because its date is in the past.

### Scenario 5.13 — An employee may not change a received line

- **Given** the fixture and Clock A, with Dana holding one line in the `confirmed` state.
- **When** Dana tries to change its note.
- **Then** the write is refused by the record rule of MEAL-031, because the state is the received
  state.
- **And** Mark, holding the administration privilege, may make the same change.

### Scenario 5.14 — An employee may not change another employee's line

- **Given** the fixture and Clock A, with Erin holding one line in the `new` state.
- **When** Dana tries to change its quantity.
- **Then** the write is refused by the record rule of MEAL-031, because the line's employee is not
  Dana.

### Scenario 5.15 — An employee may delete only to-order and cancelled lines

- **Given** the fixture and Clock A, with Dana holding four lines, one in each of the states `new`,
  `ordered`, `sent` and `confirmed`, and one in the `cancelled` state.
- **When** Dana deletes each in turn.
- **Then** the `new` line and the `cancelled` line are deleted, and the other three are refused by
  the record rule of MEAL-032.
- **And** Mark, holding the administration privilege, is refused on the same three, because the rule
  is attached to the ordering privilege which the administration privilege implies.

### Scenario 5.16 — The cart's collapsed state

- **Given** the fixture and Clock A, with Dana holding three current lines in the states
  `confirmed`, `confirmed` and `new`.
- **When** the panel computes the collapsed state.
- **Then** it is the to-order state, because that state has the lowest rank, and the control "Order
  Now" is shown.

### Scenario 5.17 — Confirming a cart with nothing to confirm

- **Given** the fixture and Clock A, with Dana holding one current line in the `ordered` state and
  none in the `new` state.
- **When** the endpoint at the route `/lunch/pay` is called for Dana.
- **Then** it answers true, because Dana has at least one current line, and nothing changes: the
  filtered set of to-order lines is empty and the confirm-cart operation does nothing.

### Scenario 5.18 — Confirming a cart for an employee with no current line

- **Given** the fixture and Clock A, with Dana holding no line dated today or later.
- **When** the endpoint at the route `/lunch/pay` is called for Dana.
- **Then** it answers false and nothing changes.

---

## 6. Dispatch and receipt

### Scenario 6.1 — The grouped dispatch control appears

- **Given** the fixture and Clock A, with Vendor P holding three active `ordered` lines dated today,
  one active `sent` line dated today, two `confirmed` lines and one `cancelled` line.
- **When** the order list is grouped by vendor.
- **Then** Vendor P's group header shows both "Send Orders", because the ordered count is 3, and
  "Confirm Orders", because the sent count is 1.

### Scenario 6.2 — Dispatching an electronic mail vendor

- **Given** the fixture, with the clock at Monday 29 October 2018 at 13:00:00 universal time, and
  Vendor P holding one `ordered` line of Meal PZ dated today for Dana, total price 9.00.
- **When** Mark presses "Send Orders" on Vendor P's group.
- **Then** one message is queued to the Contact "Pizza Inn", from Mark's formatted electronic mail
  address, with the subject "Orders for Northern Office".
- **And** the line moves to the `sent` state.
- **And** a success notice reads "The orders have been sent!".

### Scenario 6.3 — The message body

- **Given** the state of Scenario 6.2 with three lines instead of one: Meal PZ for Dana at 9.00,
  Meal PZ with Extra OL for Dana at 9.30, and Meal TS for Erin at 3.00 — the last belonging to
  Vendor C and therefore excluded.
- **When** Vendor P's dispatch runs.
- **Then** the message table holds two rows, for the two Vendor P lines, and the total row reads
  18.30.
- **And** the second row's "Comments" cell holds the extras summary "Olives" and, beneath it in
  grey, the note when one is set.
- **And** the "Site" cell of both rows holds "Farm 1", Dana's current location name.
- **And** the location block does not render at all, because of the compatibility finding in
  [`interfaces.md`](interfaces.md#7-the-vendor-order-message).

### Scenario 6.4 — Dispatching leaves other vendors alone

- **Given** the state of Scenario 6.3.
- **When** Vendor P's dispatch runs.
- **Then** the Meal TS line of Vendor C is still in the `ordered` state.

### Scenario 6.5 — Dispatching twice in a day

- **Given** the state after Scenario 6.2, where Vendor P has one `sent` line. Dana then adds another
  Meal PZ with a **different** note and confirms it, giving one more `ordered` line.
- **When** Mark presses "Send Orders" again.
- **Then** the new line moves to the `sent` state and a second message is queued holding that one
  line alone, because the dispatch collects only the lines in the `ordered` state.

### Scenario 6.6 — Dispatching with nothing to dispatch

- **Given** the fixture and Clock A, with Vendor P holding no `ordered` line dated today.
- **When** Vendor P's dispatch operation runs.
- **Then** it ends silently: no message is queued, no state changes and no refusal is raised.

### Scenario 6.7 — Dispatching a vendor that does not serve today

- **Given** the fixture with the clock at Saturday 3 November 2018 at 10:00:00 universal time, and
  Vendor P holding one `ordered` line dated Saturday 3 November 2018.
- **When** Vendor P's dispatch operation runs.
- **Then** it ends silently and the line stays in the `ordered` state.

### Scenario 6.8 — Dispatching a telephone vendor directly

- **Given** the fixture and Clock A.
- **When** Vendor C's automatic dispatch operation is invoked directly.
- **Then** it is refused with the message "Cannot send an email to this supplier!".

### Scenario 6.9 — Dispatching a telephone vendor through the grouped control

- **Given** the fixture and Clock A, with Vendor C holding two `ordered` lines dated today.
- **When** Mark presses "Send Orders" on Vendor C's group.
- **Then** both lines move to the `sent` state, no message is queued and the success notice reads
  "The orders have been sent!". No refusal is raised.

### Scenario 6.10 — Receiving the day

- **Given** the state after Scenario 6.9, with two `sent` lines of Vendor C dated today.
- **When** Mark presses "Confirm Orders" on Vendor C's group.
- **Then** both lines move to the `confirmed` state and the success notice reads "The orders have
  been confirmed!".
- **And** the charges re-enter the employees' statements.

### Scenario 6.11 — Receiving a line that is not sent

- **Given** the fixture and Clock A, with one line of Meal PZ in the `new` state.
- **When** the receipt operation is invoked on it directly.
- **Then** the line moves to the `confirmed` state with no refusal, because the operation has no
  guard. The controls that would have offered the operation are hidden on a to-order line.

### Scenario 6.12 — Pushing the delivery notice

- **Given** the fixture and Clock A, with Dana holding two `confirmed` lines and Erin holding one
  `confirmed` line, all three with the delivery-notice flag false. Company N's delivery notice
  message is the packaged default.
- **When** Mark selects all three and presses the notify control.
- **Then** exactly **two** notifications are posted, one to Dana's Contact and one to Erin's,
  because the operation sends once per distinct employee.
- **And** each carries the subject "Lunch notification" and the body "Your lunch has been delivered.
  Enjoy your meal!", rendered in that employee's language: Dana's in the base language and Erin's in
  her own.
- **And** all three lines have the delivery-notice flag set to true.

### Scenario 6.13 — Pushing the delivery notice twice

- **Given** the state after Scenario 6.12.
- **When** Mark selects the same three lines and presses the notify control again.
- **Then** nothing is posted and nothing changes, because every selected line already has the flag
  set and the operation ends as soon as the selection is empty.

### Scenario 6.14 — The bound server actions

- **Given** the fixture and Clock A, with three lines of Vendor P in the `sent` state.
- **When** Mark selects all three in the list and runs the action named "Lunch: Receive meals".
- **Then** all three move to the `confirmed` state.
- **When** Mark runs "Lunch: Cancel meals" on them.
- **Then** all three move to the `cancelled` state.
- **When** Mark runs "Lunch: Send notifications" on them.
- **Then** notifications are posted for each distinct employee, even though the lines are cancelled,
  because the operation checks only the delivery-notice flag.

---

## 7. Notices

### Scenario 7.1 — A banner notice reaches the right employee

- **Given** the fixture and Clock A, with one Lunch Alert named "Order before 11" whose mode is the
  banner mode, whose weekday flags are all set, whose show-until date is empty and whose location
  list holds Location F1 alone.
- **When** Dana, whose current location is Location F1, opens the ordering screen.
- **Then** the panel shows one warning block holding the notice's message.
- **And** Mark, whose current location is Location F2, sees no warning block.

### Scenario 7.2 — A banner notice on a day it does not apply

- **Given** the state of Scenario 7.1 with the notice's Monday flag cleared, and Clock A, which
  falls on a Monday.
- **When** Dana opens the ordering screen.
- **Then** no warning block is shown.

### Scenario 7.3 — A banner notice on its show-until date

- **Given** the state of Scenario 7.1 with the show-until date set to Monday 29 October 2018, and
  Clock A.
- **When** Dana opens the ordering screen.
- **Then** no warning block is shown, because the displayability rule requires the show-until date
  to be strictly later than today.

### Scenario 7.4 — A pushed notice reaches the employees who ordered

- **Given** the fixture and Clock A, with one Lunch Alert whose mode is the pushed mode, whose
  audience is everyone, whose location list holds Location F1 and Location F2, whose message is
  "Today's delivery is at noon." and whose weekday flags are all set.
- **And** Dana has one order in the `ordered` state and Erin has one order in the `cancelled` state,
  and both are currently at Location F1.
- **When** the notice's scheduled action runs.
- **Then** one notification is posted on the notice record with the subject "Your Lunch Order" and
  the body "Today's delivery is at noon.", addressed to Dana's Contact alone: Erin is excluded
  because her only order is cancelled.

### Scenario 7.5 — A pushed notice with a last-week audience

- **Given** the state of Scenario 7.4 with the audience set to the last-week audience, Clock A being
  Monday 29 October 2018, Dana's only order dated Friday 26 October 2018 and Erin's only order
  dated Monday 15 October 2018, both in the `ordered` state.
- **When** the notice's scheduled action runs.
- **Then** the audience cut-off date is 22 October 2018, so Dana is included and Erin is not, and
  one notification is posted to Dana's Contact.

### Scenario 7.6 — A pushed notice with a last-month audience

- **Given** the same state with the audience set to the last-month audience.
- **Then** the cut-off date is 1 October 2018, four weeks before Monday 29 October 2018, so both
  Dana and Erin are included and one notification is posted to both Contacts.

### Scenario 7.7 — A pushed notice narrowed by location

- **Given** the state of Scenario 7.4 with the notice's location list holding Location F2 alone.
- **When** the notice's scheduled action runs.
- **Then** no notification is posted, because neither Dana nor Erin is currently at Location F2.

### Scenario 7.8 — A pushed notice on a day it does not apply

- **Given** the state of Scenario 7.4 with the notice's Monday flag cleared.
- **When** the notice's scheduled action runs on Monday 29 October 2018.
- **Then** nothing is posted. The scheduled action is not deleted, because the show-until date is
  empty.

### Scenario 7.9 — A pushed notice whose show-until date has passed

- **Given** the state of Scenario 7.4 with the show-until date set to Friday 26 October 2018, and
  the clock at Monday 29 October 2018.
- **When** the notice's scheduled action runs.
- **Then** nothing is posted, the scheduled action is deleted and the notice's reference to it is
  cleared.

### Scenario 7.10 — A pushed notice on its show-until date

- **Given** the state of Scenario 7.4 with the show-until date set to Monday 29 October 2018, and
  Clock A.
- **Then** the scheduled action is active, because today is on or before the show-until date.
- **When** it runs.
- **Then** nothing is posted, because the displayability rule requires the date to be strictly
  later, and the action is not deleted either, because today is not strictly later than the
  show-until date. This is the gap recorded in
  [`state-machines.md`](state-machines.md#54-the-gap-on-the-show-until-date-itself).

### Scenario 7.11 — Pushing a banner notice directly

- **Given** the state of Scenario 7.1, where the notice's mode is the banner mode.
- **When** its push operation is invoked directly.
- **Then** it is refused with the message "Cannot send a chat notification in the current state".

### Scenario 7.12 — A notice with no location is pushed to everyone

- **Given** the fixture and Clock A, with one Lunch Alert in the pushed mode, the everyone audience
  and an **empty** location list, created without the form.
- **When** its scheduled action runs.
- **Then** every employee with a non-cancelled order is notified, because the location narrowing is
  applied only when the list is non-empty. The same notice would never appear as a banner, because
  the banner query requires the employee's location to be in the list.

---

## 8. Scheduled actions and time zones

### Scenario 8.1 — A vendor's action is created inactive and then synchronised

- **Given** Clock B, Friday 29 January 2021 at 12:20:00 universal time.
- **When** Mark creates Vendor K: channel electronic mail, cut-off hour 10.0 in the morning half,
  time zone five hours behind universal time.
- **Then** exactly one scheduled action exists for Vendor K. It is active, its name is "Lunch: send
  automatic email to Kothai", its interval is once every one day, its owner is the platform's root
  user and its next due instant is 29 January 2021 at 15:00:00 universal time.
- **And** its body holds two comment lines stating that it is controlled by the Lunch Vendor entity
  and must not be edited directly, and one line that invokes the dispatch operation on Vendor K
  alone.

### Scenario 8.2 — A notice's action nine hours ahead is postponed

- **Given** Clock B.
- **When** Mark creates a Lunch Alert named "Tokyo UTC+9" in the pushed mode, with the notification
  hour 8.0 in the morning half and a time zone nine hours ahead of universal time.
- **Then** the scheduled action's next due instant is 29 January 2021 at 23:00:00 universal time,
  which is 30 January at 08:00 in the notice's own zone, because the candidate instant of 28 January
  at 23:00 universal had already passed.

### Scenario 8.3 — A notice's action five hours behind is not postponed

- **Given** Clock B.
- **When** Mark creates a Lunch Alert named "New York UTC-5" in the pushed mode, with the
  notification hour 10.0 in the morning half and a time zone five hours behind universal time.
- **Then** the scheduled action's next due instant is 29 January 2021 at 15:00:00 universal time,
  its name is "Lunch: alert chat notification (New York UTC-5)" and it is active.

### Scenario 8.4 — Archiving deactivates, un-archiving reactivates

- **Given** the state of Scenario 8.1.
- **When** Mark archives Vendor K.
- **Then** the scheduled action becomes inactive.
- **When** Mark un-archives Vendor K.
- **Then** the scheduled action becomes active again.

### Scenario 8.5 — Changing the channel deactivates the action

- **Given** the state of Scenario 8.1.
- **When** Mark changes Vendor K's channel to telephone.
- **Then** the scheduled action becomes inactive.
- **When** Mark changes it back to electronic mail.
- **Then** the scheduled action becomes active again.

### Scenario 8.6 — Archiving a notice, switching its mode and passing its show-until date

- **Given** the state of Scenario 8.3, whose action is active.
- **When** Mark archives the notice, the action becomes inactive; when he un-archives it, active.
- **When** Mark switches the mode to the banner mode, the action becomes inactive; when he switches
  it back to the pushed mode, active.
- **When** Mark sets the show-until date to one day before today in the notice's zone, the action
  becomes inactive; when he sets it to two days after, active; when he clears it, active.

### Scenario 8.7 — Lowering the hour before the first run

- **Given** the state of Scenario 8.1, whose next due instant is 29 January 2021 at 15:00:00
  universal time and whose action has never run, with the clock still at Clock B.
- **When** Mark lowers Vendor K's cut-off hour by five, from 10.0 to 5.0.
- **Then** the next due instant becomes 30 January 2021 at 10:00:00 universal time: the previous
  instant less five hours plus one day, because 10:00 universal on 29 January is already past.

### Scenario 8.8 — Raising the hour after a run today

- **Given** the state of Scenario 8.7, with the action's last-run instant set to 29 January 2021 at
  10:00:00 universal time and its next due instant advanced to 31 January 2021 at 10:00:00 universal
  time, the clock still at Clock B.
- **When** Mark raises the cut-off hour by seven, from 5.0 to 12.0.
- **Then** the next due instant becomes 30 January 2021 at 17:00:00 universal time: the original
  15:00 instant plus one day and two hours.
- **When** Mark then lowers it by one, from 12.0 to 11.0.
- **Then** the next due instant becomes 30 January 2021 at 16:00:00 universal time.

### Scenario 8.9 — Deleting the vendor deletes the action

- **Given** the state of Scenario 8.1.
- **When** Mark deletes Vendor K, which has no meals pointing at it.
- **Then** the vendor, its scheduled action and the server action behind that action are all gone.

### Scenario 8.10 — The cut-off hour range constraint

- **Given** the fixture.
- **When** Mark sets Vendor P's cut-off hour to 13.0.
- **Then** the write is refused with the message "Automatic Email Sending Time should be between 0
  and 12", and the scheduled action's next due instant is unchanged, because the constraint is
  checked before the synchronisation.
- **And** setting it to −1.0 is refused with the same message, while 0.0 and 12.0 are both accepted.

### Scenario 8.11 — The notification hour range constraint

- **Given** the fixture.
- **When** Mark sets a notice's notification hour to 12.5.
- **Then** the write is refused with the message "Notification time must be between 0 and 12".

### Scenario 8.12 — Twelve in the afternoon half

- **Given** a vendor whose cut-off hour is 12.0 and whose half-day marker is the afternoon half, in
  the zero-offset universal zone.
- **When** the clock time of that cut-off is computed.
- **Then** it is 23:59:59 and 999999 microseconds, the last representable instant of the day, by the
  special case of the conversion rule.
- **And** the cut-off flag is therefore false for practically the whole day.

### Scenario 8.13 — Availability is read in the vendor's zone, not the reader's

- **Given** a vendor whose time zone is nine hours ahead of universal time and which serves Monday
  to Friday, and a reader in the zero-offset universal zone.
- **And** the clock at Friday 29 January 2021 at 16:00:00 universal time, which is Saturday
  30 January at 01:00 in the vendor's zone.
- **When** the vendor's availability today is read.
- **Then** it is false, because the vendor's own date is a Saturday, even though the reader's date is
  still a Friday.

---

## 9. Access, impersonation and several companies

### Scenario 9.1 — An orderer cannot see another employee's account movements

- **Given** the fixture, with one movement for Dana of 100.00 and one for Erin of 20.00.
- **When** Dana lists the Lunch Cash Move records.
- **Then** she sees her own movement alone.
- **And** Mark, holding the administration privilege, sees both.

### Scenario 9.2 — An orderer cannot create an account movement

- **Given** the fixture.
- **When** Dana tries to create a Lunch Cash Move.
- **Then** the creation is refused by the access matrix, which grants the ordering group read alone
  on that entity.

### Scenario 9.3 — The statement is readable by every internal user

- **Given** the fixture.
- **When** Dana queries the Lunch Cash Move Report without a filter.
- **Then** she receives Erin's rows as well as her own, because the report carries no record rule.
  This is the compatibility finding recorded as rule MEAL-054.
- **And** the screen "My Account History" nevertheless shows her own rows alone, because it applies
  a filter on the reading user.

### Scenario 9.4 — Impersonation by an administrator

- **Given** the fixture and Clock A.
- **When** Mark selects Dana in the panel's employee selector and the screen calls the endpoint at
  the route `/lunch/infos` naming Dana.
- **Then** the endpoint answers with Dana's name, picture, balance, location and lines.
- **And** the administrator flag in the answer is true, because it reports whether the **requesting**
  user is an administrator, which is Mark.

### Scenario 9.5 — Impersonation by an orderer is refused

- **Given** the fixture and Clock A.
- **When** Dana calls the endpoint at the route `/lunch/infos` naming Erin.
- **Then** it is refused with the message "You are trying to impersonate another user, but this can
  only be done by a lunch manager".
- **And** the same refusal is raised for the routes `/lunch/trash`, `/lunch/pay`,
  `/lunch/user_location_set` and `/lunch/user_location_get`.

### Scenario 9.6 — Setting a location for an impersonated employee

- **Given** the fixture and Clock A.
- **When** Mark, having selected Dana, calls the endpoint at the route `/lunch/user_location_set`
  with Location F2.
- **Then** Dana's current location becomes Location F2, because the write is performed with elevated
  rights.

### Scenario 9.7 — A location belonging to another company is replaced

- **Given** the fixture, with Dana's current location set to Location SS, which belongs to
  Company S, and Dana's active companies holding Company N alone.
- **When** Dana calls the endpoint at the route `/lunch/user_location_get`.
- **Then** the endpoint returns the record identifier of the oldest location whose company is empty
  or among Dana's active companies, which is Location F1.
- **And** the endpoint at the route `/lunch/infos` additionally writes that location onto Dana.

### Scenario 9.8 — A meal of another company is invisible

- **Given** the fixture with a meal of Company S, and Dana's active companies holding Company N
  alone.
- **When** Dana lists the catalogue.
- **Then** the Company S meal is absent, by the global company rule on the meal.

### Scenario 9.9 — Changing a vendor's company rewrites its orders

- **Given** the fixture, with Vendor P's company empty and three orders of Vendor P: one in the
  `new` state, one in the `confirmed` state dated last month and one archived.
- **When** Mark sets Vendor P's company to Company S.
- **Then** all three orders have their company rewritten to Company S, including the archived one and
  the historical one, and all three disappear from Dana's lists. This is rule MEAL-048.

### Scenario 9.10 — An extra of another company is visible

- **Given** the fixture with an extra belonging to Company S, attached to a vendor of Company S.
- **When** Dana queries the Lunch Topping records directly.
- **Then** the Company S extra is returned, because no company rule exists on that entity. This is
  part of rule MEAL-034.

### Scenario 9.11 — A user without the ordering privilege cannot read the personal fields

- **Given** an internal user holding neither meal ordering privilege.
- **When** that user reads their own last ordering location.
- **Then** the read is refused, because the field belongs to the ordering privilege.

### Scenario 9.12 — An orderer may update but not create a location

- **Given** the fixture.
- **When** Dana renames Location F1 to "Farm One".
- **Then** the write succeeds, because the access matrix grants the ordering group the update right
  on the location.
- **When** Dana tries to create a new location.
- **Then** the creation is refused.

---

## 10. Catalogue archival

### Scenario 10.1 — Archiving a vendor archives its meals

- **Given** the fixture.
- **When** Mark archives Vendor P.
- **Then** Meal PZ is archived and Vendor P's scheduled action becomes inactive.

### Scenario 10.2 — Un-archiving a vendor un-archives its meals

- **Given** the state of Scenario 10.1.
- **When** Mark un-archives Vendor P.
- **Then** Meal PZ is active again and the scheduled action becomes active again.

### Scenario 10.3 — Un-archiving a vendor whose meal sits in an archived category fails

- **Given** the fixture with Category "Pizza" archived, which archived Meal PZ, and Vendor P then
  archived as well.
- **When** Mark un-archives Vendor P.
- **Then** the whole operation is refused with the message "The following product categories are
  archived. You should either unarchive the categories or change the category of the product."
  followed by a line break and the single line "Pizza".
- **And** Vendor P is still archived. This is the compatibility finding in
  [`entities.md`](entities.md#16-archival-and-company-behaviour).

### Scenario 10.4 — Archiving a category archives its meals

- **Given** the fixture.
- **When** Mark archives Category "Pizza".
- **Then** Meal PZ is archived.

### Scenario 10.5 — Un-archiving a category leaves meals of archived vendors archived

- **Given** the fixture with Category "Pizza" archived and Vendor P archived.
- **When** Mark un-archives Category "Pizza".
- **Then** Meal PZ stays archived, because its vendor is still archived, and no refusal is raised.

### Scenario 10.6 — Un-archiving a meal under an archived vendor is refused

- **Given** the fixture with Vendor P archived, which archived Meal PZ.
- **When** Mark tries to un-archive Meal PZ alone.
- **Then** it is refused with the message "The following suppliers are archived. You should either
  unarchive the suppliers or change the supplier of the product." followed by a line break and the
  single line "Pizza Inn".

### Scenario 10.7 — Creating an active meal under an archived category is refused

- **Given** the fixture with Category "Sandwich" archived.
- **When** Mark creates a meal named "Club" in Category "Sandwich" for Vendor C at 3.40.
- **Then** the creation is refused with the archived-category message followed by a line break and
  the single line "Sandwich".

### Scenario 10.8 — Two offending meals list two names

- **Given** the fixture with Category "Pizza" and Category "Sandwich" both archived and Vendor P and
  Vendor C both archived, so that Meal PZ and Meal TS are both archived.
- **When** Mark selects both meals and un-archives them in one operation.
- **Then** the refusal's trailing block holds two lines, "Pizza" and "Sandwich", one per archived
  category of the offending meals.

### Scenario 10.9 — A category counter

- **Given** the fixture, where Category "Sandwich" holds Meal TS and Meal KC.
- **When** Mark opens Category "Sandwich".
- **Then** the counter control reads 2 and opens the meal screen pre-filtered and pre-defaulted on
  that category.
- **And** for a category with no meal the counter control is hidden.

---

## 11. Rounding, precision and several currencies

### Scenario 11.1 — An ordinary total

- **Given** the fixture and Clock A.
- **When** Dana adds Meal PZ with Extra OL and Extra CH and raises the quantity to 3.
- **Then** the price for one unit is 9.00 + 0.30 + 1.15 = 10.45 and the stored total price is
  3 × 10.45 = **31.35**.

### Scenario 11.2 — A half-way total rounds away from zero

- **Given** a meal priced 0.09, one extra whose stored price is 0.055 because the extra's own
  currency has three decimal places, a quantity of 3, and an order currency with two decimal
  places.
- **When** the total price is computed.
- **Then** the unrounded product is 0.435 and the stored total price is **0.44**, because currency
  rounding breaks the tie away from zero and the epsilon correction defeats the binary
  representation of 0.435.

### Scenario 11.3 — A fractional quantity

- **Given** a meal priced 6.80, no extras and a quantity of 0.5.
- **When** the total price is computed.
- **Then** it is **3.40**.

### Scenario 11.4 — The panel figures add up to the line total

- **Given** the state of Scenario 11.1: quantity 3, meal 9.00, extras 0.30 and 1.15.
- **When** the panel is built.
- **Then** the meal figure reads 27.00, the first extra figure reads 0.90 and the second reads 3.45,
  and 27.00 + 0.90 + 3.45 = 31.35, which is the stored total price.

### Scenario 11.5 — The cart totals

- **Given** the fixture and Clock A, with Dana holding four current lines: a `confirmed` line of
  8.65, a `sent` line of 3.40, an `ordered` line of 12.00 and a `new` line of 6.80.
- **When** the panel is built.
- **Then** it shows "Total 30.85", "Already Paid 24.05" and "To Pay 6.80".

### Scenario 11.6 — A vendor message total

- **Given** the fixture and Clock A, with Vendor P holding three `ordered` lines of 9.00, 9.30 and
  3.00 dated today.
- **When** the dispatch runs.
- **Then** the message's total row reads 21.30, formatted in the currency of the first collected
  line, and the lines are listed sorted by employee and then by meal.

### Scenario 11.7 — The message currency comes from the first line

- **Given** a vendor with two `ordered` lines dated today, one for an employee of Company N and one
  for an employee of Company S, whose companies use different currencies, the Company N employee
  sorting first by name.
- **When** the dispatch runs.
- **Then** every amount in the message, including the amount of the Company S line, is formatted in
  Company N's currency. No conversion is performed.
- **And** the subject reads "Orders for Northern Office", the company name of that same first line.

### Scenario 11.8 — A clock time from a decimal hour

- **Given** the conversion rule.
- **Then** 10.0 in the morning half gives 10:00:00; 10.5 in the morning half gives 10:30:00; 9.25 in
  the afternoon half gives 21:15:00; 0.0 in the afternoon half gives 12:00:00; 7.99 in the morning
  half gives 07:59:00; and 12.0 in the afternoon half gives 23:59:59 and 999999 microseconds.

### Scenario 11.9 — A decimal hour that overflows the minute

- **Given** a vendor whose cut-off hour is written directly as 10.9917.
- **When** the clock time is computed.
- **Then** the minute rounds to 60 and the conversion fails, so the scheduled action's next due
  instant cannot be computed and the write fails. The value cannot be produced from a screen,
  because the hour control only ever emits whole minutes. This is the compatibility finding in
  [`calculations.md`](calculations.md#52-worked-examples).

---

## 12. Concurrency

### Scenario 12.1 — Two simultaneous confirmations may both pass the balance check

- **Given** the fixture and Clock A, with Erin's balance at 9.00 and Company N permitting 0.00, and
  two separate to-order lines of Meal PZ each priced 9.00, each in its own session.
- **When** both sessions confirm at the same instant, each flushing its own write and reading the
  balance before the other commits.
- **Then** both checks see a balance of 0.00 and both commit, leaving Erin at −9.00, beyond the
  permitted overdraft. This is the compatibility finding recorded as rule MEAL-044.
- **And** a corrected implementation would take a lock on Erin's account before reading the balance,
  so that the second confirmation would be refused with the wallet message.

### Scenario 12.2 — Two simultaneous additions may both create a line

- **Given** the fixture and Clock A, with Dana holding no line of Meal PZ.
- **When** two sessions each add Meal PZ with the same extras, note, date and location at the same
  instant.
- **Then** each search finds no match and each creates a line, leaving two to-order lines of
  quantity 1 where one line of quantity 2 was intended. This is rule MEAL-055.

---

## 13. Coverage map

| Area | Scenarios |
|---|---|
| Catalogue, availability and favourites | 1.1 to 1.10 |
| Cart building, merging and quantity | 2.1 to 2.15 |
| Extras discipline and group forcing | 3.1 to 3.10 |
| Balance, permitted overdraft and statement rows | 4.1 to 4.16 |
| Pipeline transitions and record rules on orders | 5.1 to 5.18 |
| Dispatch, receipt and delivery notice | 6.1 to 6.14 |
| Notices, banner and pushed | 7.1 to 7.12 |
| Scheduled actions, time zones and hour constraints | 8.1 to 8.13 |
| Access, impersonation and several companies | 9.1 to 9.12 |
| Catalogue archival | 10.1 to 10.9 |
| Rounding, precision and several currencies | 11.1 to 11.9 |
| Concurrency | 12.1 to 12.2 |

| Refusal message | Scenario that raises it |
|---|---|
| "The vendor related to this order is not available at the selected date." | 5.2, 5.4 |
| "Product is no longer available." | 5.3 |
| "The vendor related to this order is not available today." | 5.10 |
| "Cannot send an email to this supplier!" | 6.8 |
| "Oh no! You don’t have enough money in your wallet to order your selected lunch! Contact your lunch manager to add some money to your wallet." | 4.3, 4.10 |
| "You should order at least one %s" | 3.2, 3.8 |
| "You have to order one and only one %s" | 3.5, 3.6 |
| "The following product categories are archived. You should either unarchive the categories or change the category of the product." | 10.3, 10.7, 10.8 |
| "The following suppliers are archived. You should either unarchive the suppliers or change the supplier of the product." | 10.6 |
| "Cannot send a chat notification in the current state" | 7.11 |
| "Automatic Email Sending Time should be between 0 and 12" | 8.10 |
| "Notification time must be between 0 and 12" | 8.11 |
| "You are trying to impersonate another user, but this can only be done by a lunch manager" | 9.5 |
| The record-rule refusal on writing an order | 5.13, 5.14 |
| The record-rule refusal on deleting an order | 5.15 |
