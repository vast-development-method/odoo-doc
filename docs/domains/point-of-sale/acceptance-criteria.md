# Point of Sale — Acceptance Criteria

Numbered Given / When / Then scenarios with concrete numbers. A conforming
implementation must reproduce every stated figure, every state value, every record
created and every message text.

Unless a scenario says otherwise, the following **standard setting** applies.

| Element | Value |
| --- | --- |
| Company | One company, currency with rounding step 0.01 and two decimal places, tax rounding per line |
| Configuration | Named "Shop", identifier 3, point of sale journal `POSS`, invoice journal a sale journal, no cash rounding, no category restriction, price control not restricted, edit tracking off, stock updated in real time |
| Payment methods | "Cash" (cash journal, default account 570000, profit account 758000, loss account 658000, does not identify the customer), "Card" (bank journal, outstanding account 101401, no intermediary account, does not identify the customer), "Customer Account" (no journal, identifies the customer) |
| Company default counter receivable account | 101300 |
| Tax | A single 21 percent price-included sale tax, tax account 251000 |
| Accounts | Income 400000, expense 600000, stock valuation 140000, customer receivable 121000 |
| Sequences | Session sequence prefix `/`, padding 5; the four configuration sequences with padding 6 and the no-gap implementation |
| Device identifier | `0` |
| Year | 2026 |

---

## Group A — Session lifecycle

### A1 — Opening a session, happy path

**Given** configuration "Shop" with no session, and a previous session closed with a
counted ending balance of 120.00,
**When** a cashier opens the selling application for "Shop" and confirms the opening
control with a counted amount of 120.00 and no notes,
**Then**
1. a session is created in the `opening_control` state with a starting balance of 120.00
   and a name of `/`;
2. after confirmation the state is `opened`, the opening instant is stamped, the starting
   balance is 120.00 and the name is `Shop/00001`;
3. the session thread carries a message reading
   `Opening cash difference: 0.00`, `Opening cash expected: 120.00`,
   `Opening cash counted: 120.00`, each on its own line;
4. no accounting record is created.

### A2 — Opening with a counted amount differing from the previous closing

**Given** the same session pre-filled with 120.00,
**When** the cashier counts 118.50 and confirms with the note `short by two coins`,
**Then**
1. the thread reads `Opening cash difference: -1.50`, `Opening cash expected: 120.00`,
   `Opening cash counted: 118.50`, followed by
   `Opening control message: short by two coins`;
2. the starting balance becomes 118.50;
3. **no** accounting entry is produced for the 1.50: an opening discrepancy is recorded in
   the thread only.

### A3 — A second session on the same configuration is refused

**Given** configuration "Shop" with a session in the `opened` state,
**When** a second session is created for "Shop" without the onboarding marker,
**Then** the creation is refused with
*"Another session is already opened for this point of sale."*

### A4 — A recovery session may coexist

**Given** the same open session,
**When** a session is created for "Shop" with the recovery flag set,
**Then** the creation succeeds, the configuration's current session is still the first
one, the configuration's has-active-session flag is true and its rescue session count is
one.

### A5 — A session may not start before a lock date

**Given** the company's fiscal-year lock date is 31 December 2025 and today is
15 December 2025,
**When** a session is created,
**Then** the creation is refused with
*"You cannot create a session starting before: <the formatted lock dates>"*.

### A6 — Cancelling an unused session

**Given** a session in `opening_control` with no order,
**When** the cashier cancels it,
**Then** the session and its cash statement lines are deleted and the answer reports
success.

### A7 — Cancelling a used session is refused

**Given** a session in `opening_control` with one order, or a session in `opened`,
**When** the cancel operation is invoked,
**Then** it is refused with *"You can only cancel a session that is in opening control
state and has no orders."*

### A8 — Closing is blocked by an unfinished order

**Given** an opened session with one order in the `draft` state whose scheduled time is
empty,
**When** the cashier asks to close,
**Then** the answer is a refusal carrying
*"You cannot close the POS while there are still draft orders for the day."*, the
redirect flag false, and the identifiers of the unfinished orders.

### A9 — An order scheduled in the future does not block the closing

**Given** an opened session with one paid order and one `draft` order whose scheduled time
is tomorrow at 12:00,
**When** the cashier closes the session from the selling application,
**Then**
1. the future order's session link is emptied, so it survives into the next session;
2. the closing proceeds normally;
3. the session reaches the `closed` state.

### A10 — Closing a session with no activity

**Given** an opened session with no order and no cash movement,
**When** the session is validated,
**Then**
1. no closing entry is created;
2. the cash difference, if any, is still posted as a statement line;
3. the session state becomes `closed`.

### A11 — A closing entry with no line is discarded

**Given** a session whose only order was cancelled,
**When** the session is validated,
**Then** the closing entry is created, found to have no line, and deleted; the orders are
**not** moved to the `done` state; the session state becomes `closed`.

### A12 — Concurrent closing

**Given** two cashiers closing the same session at the same time, the first having
succeeded,
**When** the second asks to close,
**Then** the answer is an alert titled *Session already closed* reading *"The session has
been already closed by another User. All sales completed in the meantime have been saved
in a Rescue Session, which can be reviewed anytime and posted to Accounting from Point of
Sale's dashboard."* with the redirect flag true.

---

## Group B — The mandatory closing scenario

### B1 — A session with a cash order, a card order and an invoiced order, closed short

**Given** the standard setting, a session opened with a counted opening amount of 0.00 and
no manual cash movement, and three orders:

| Order | Content | Cost of the goods | Tender | Invoiced |
| --- | --- | --- | --- | --- |
| A | One unit of product P, unit price 25.00 including 21 percent tax | 12.00 | Cash 25.00 | No |
| B | One unit of product Q, unit price 40.50 including 21 percent tax | 18.00 | Card 40.50 | No |
| C | One unit of product R, unit price 100.00 including 21 percent tax | — | Card 100.00 | Yes, customer Acme |

**When** the cashier closes the session with a counted cash amount of 24.50,

**Then** the following figures are computed:

```formula
base_A = round_to_currency( 25.00 ÷ 1.21 ) = 20.66       tax_A = 25.00 − 20.66 = 4.34
base_B = round_to_currency( 40.50 ÷ 1.21 ) = 33.47       tax_B = 40.50 − 33.47 = 7.03
sales_bucket = − ( 20.66 + 33.47 ) = − 54.13
tax_bucket   = − ( 4.34 + 7.03 )   = − 11.37             tax_base = 54.13
cash_bucket  = 25.00
card_bucket  = 40.50 + 100.00 = 140.50
invoiced_card_bucket = 100.00
cost_bucket  = 12.00 + 18.00 = 30.00
theoretical_cash = 0.00 + 0.00 + 25.00 = 25.00
cash_difference  = 24.50 − 25.00 = − 0.50
```

**And** the closing entry, in the point of sale journal, dated today, referenced with the
session identifier, contains exactly seven lines:

| # | Name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | 11.37 |
| 2 | `Sales with 21%` | 400000 | | 54.13 |
| 3 | `<session> - Card` | 101300 | 140.50 | |
| 4 | `<session> - Cash` | 101300 | 25.00 | |
| 5 | `From invoice payments` | 101300 | | 100.00 |
| 6 | *(unnamed)* | 600000 | 30.00 | |
| 7 | *(unnamed)* | 140000 | | 30.00 |
| | **Totals** | | **195.50** | **195.50** |

Line 1 carries the tax repartition line of the 21 percent tax, its tax tags and a tax base
amount of 54.13. Line 2 carries the 21 percent tax in its tax set, the base tags of that
tax, a quantity of 1.00 and the product line display kind. Lines 3, 4 and 5 carry the
payment-term display kind and no partner. Lines 6 and 7 are written in company currency
only.

**And** the satellite documents are:

1. **A cash statement line** in the cash journal, dated today, labelled with the session
   identifier, counterpart account 101300, amount 25.00 — producing a debit of 25.00 on
   570000 and a credit of 25.00 on 101300.
2. **An accounting payment** in the bank journal, inbound, amount 140.50, forced
   outstanding account 101401, destination 101300, memo `Combine Card point of sale payments from
   <session identifier>` — producing a debit of 140.50 on 101401 and a credit of 140.50 on
   101300.
3. **The invoice of order C**, in the invoice journal, customer Acme, total 100.00: a
   credit of 82.64 on 400000, a credit of 17.36 on 251000, a debit of 100.00 on 121000.
4. **The invoice payment entry of order C**, in the point of sale journal, dated order C's
   date, referenced `Invoice payment for <order C name> (<invoice number>) using Card`: a
   debit of 100.00 on 101300 with no partner and a credit of 100.00 on 121000 with partner
   Acme.
5. **The cash difference statement line** in the cash journal, amount −0.50, labelled
   `Cash difference observed during the counting (Loss) - closing`, counterpart account
   658000 — producing a debit of 0.50 on 658000 and a credit of 0.50 on 570000; its entry
   carries a message `Related Session: <link>`.

**And** the reconciliations performed are:

| Group | Lines | Account | Amount |
| --- | --- | --- | --- |
| Cash | Closing line 4 against the counterpart line of the cash statement | 101300 | 25.00 |
| Card | Closing line 3 against the destination line of the accounting payment | 101300 | 140.50 |
| Invoiced order | Closing line 5 against the counter-side line of the invoice payment entry | 101300 | 100.00 |
| Invoice settlement (at invoicing time, not at closing) | The customer-side line of the invoice payment entry against the invoice's receivable line | 121000 | 100.00 |

**And** the residual balances are: 101300 zero for this session; 121000 zero for Acme;
570000 24.50; 101401 140.50 awaiting the bank statement; 658000 0.50.

**And** orders A and B move from `paid` to `done`; order C was already `done` when it was
invoiced; the session state becomes `closed`.

### B2 — The same scenario closed exactly

**Given** B1 but with a counted cash amount of 25.00,
**Then** the closing entry is identical, and **no** cash difference statement line is
created at all.

### B3 — The same scenario closed long

**Given** B1 but with a counted cash amount of 26.00,
**Then** the cash difference is +1.00, a statement line labelled
`Cash difference observed during the counting (Profit) - closing` is created with
counterpart account 758000, producing a debit of 1.00 on 570000 and a credit of 1.00 on
758000.

### B4 — The same scenario with the cash journal missing its loss account

**Given** B1 with the cash journal's loss account cleared,
**When** the session is validated,
**Then** the validation is refused with *"Please go on the Cash journal and define a Loss
Account. This account will be used to record cash difference."*

### B5 — The same scenario with the tax lacking an account

**Given** B1 with the 21 percent tax's sale repartition line carrying no account,
**When** the session is validated,
**Then** the validation is refused with *"Unable to close and validate the session.\nPlease
set corresponding tax account in each repartition line of the following taxes: \n21%"* and
nothing is posted.

### B6 — The same scenario with the maximum difference set

**Given** B1 with the configuration's maximum-difference flag set and an authorised
difference of 0.20, and a cashier who is **not** a counter administrator,
**When** the counted amount of 24.50 gives a difference of −0.50,
**Then** the selling application refuses the closing and asks for an administrator; with
an administrator, the closing proceeds exactly as in B1.

### B7 — The same scenario in a foreign selling currency

**Given** B1 but the configuration's journal carries a currency other than the company
currency, with a rate such that one unit of the selling currency equals 0.80 units of the
company currency at every relevant date,
**Then**
1. lines 1 to 5 each carry an amount in currency equal to the figures of B1 and a balance
   equal to those figures multiplied by 0.80 and rounded to the company currency: 9.10,
   43.30, 112.40, 20.00 and 80.00 respectively;
2. lines 6 and 7 carry **no** amount in currency and a balance of 30.00, because cost and
   valuation are always in company currency;
3. the cash statement line carries an amount of 20.00 in the journal currency, an amount
   in currency of 25.00 and the selling currency as its foreign currency, because the cash
   journal is in the company currency;
4. the closing entry still balances; if per-line conversion rounding leaves a residue, the
   entry does not balance and the forced-close wizard is offered instead of a silent
   imbalance.

---

## Group C — Orders and payments

### C1 — A simple cash sale

**Given** an opened session,
**When** the cashier adds one unit of a product priced 12.10 including 21 percent tax and
tenders 12.10 in cash,
**Then**
1. the line's tax-included amount is 12.10 and its tax-excluded amount is 10.00;
2. the order's tax amount is 2.10 and its total is 12.10;
3. the paid amount is 12.10, the returned amount is 0.00, the difference is 0.00;
4. the order state becomes `paid`;
5. the receipt number is `260-3-<the next backend sequence value>`, the tracking number is
   that value modulo one thousand, and the order name is
   `Shop - <the last part of the receipt number>`.

### C2 — Change is a negative cash tender

**Given** the same order,
**When** the customer hands over 20.00,
**Then**
1. the change is 7.90;
2. on transmission the server creates a second tender named `return`, amount −7.90, on the
   cash method, with the change flag set;
3. the paid amount recorded server-side is `20.00 − 7.90 = 12.10`;
4. the returned amount is 7.90.

### C3 — Change without a cash method is refused

**Given** a configuration with only a card method,
**When** an order is transmitted with a non-zero change,
**Then** the transmission is refused with *"No cash statement found for this session.
Unable to record returned cash."*

### C4 — An underpaid order is refused

**Given** an order totalling 12.10 with tenders summing 12.00 and no cash rounding,
**When** the order is marked paid,
**Then** the transition is refused with *"Order <order name> is not fully paid."*

### C5 — An overpaid order produces a warning, not a refusal

**Given** a paid order totalling 12.10,
**When** a further tender of 1.00 is added,
**Then** the write succeeds and the payment-changes message ends with
*"Warning, the paid amount is higher than the total amount. (Difference: 1.00)"*.

### C6 — Changing the tender of a printed order is refused

**Given** a paid order whose print count is 1,
**When** a tender is added or changed with a payment status other than `cancelled`,
**Then** the write is refused with *"You cannot change the payment of a printed order."*

### C7 — A tender of a method not offered on the till is refused

**Given** a payment method that is not among the configuration's methods,
**When** a tender using it is written,
**Then** the write is refused with *"The payment method selected is not allowed in the
config of the point of sale session."*

### C8 — Editing a tender of a posted order is refused

**Given** an order in the `done` state,
**When** the amount of one of its tenders is written,
**Then** the write is refused with *"You cannot edit a payment for a posted order."*

### C9 — A transmission replay is idempotent

**Given** an unfinished order transmitted once, with two lines,
**When** the identical payload is transmitted again,
**Then**
1. the existing order is found by its universally unique identifier;
2. the two create commands for the lines are converted into updates of the existing lines;
3. the order still has exactly two lines;
4. no new order is created.

### C10 — A transmission of an already paid order is a no-operation

**Given** a paid order,
**When** the same payload is transmitted again,
**Then** the existing order is returned unchanged, the server log records the event, and
nothing is written.

### C11 — A transmission against a closed session is re-homed

**Given** session S1 closed and session S2 opened on the same configuration,
**When** an order naming S1 is transmitted,
**Then**
1. a warning is logged naming S1, its identifier, the order's universally unique
   identifier and its total;
2. a second warning names S2 as the session used;
3. the order is created in S2.

### C12 — A transmission against a closed session with no open session is refused

**Given** session S1 closed and no other session on the configuration,
**When** an order naming S1 is transmitted,
**Then** the transmission is refused with *"No open session available. Please open a new
session to capture the order."*

### C13 — Deleting an order

**Given** an unfinished order,
**When** it is deleted,
**Then** it is first written to the `cancel` state (so that the change feed carries the
cancellation) and then deleted. For an order in `paid` or `done`, the deletion is refused
with *"In order to delete a sale, it must be new or cancelled."*

### C14 — Cancelling a future-dated order from the administrative interface

**Given** a selection containing one order whose scheduled time is tomorrow,
**When** the cancel action is invoked,
**Then** it is refused with *"The order delivery / pickup date is in the future. You
cannot cancel it."*

### C15 — Edit tracking

**Given** a configuration with edit tracking on and an unfinished order with a line of
3 units of product P,
**When** the quantity is reduced to 2 and then a second line is deleted,
**Then**
1. the first line's edited flag becomes true and the order thread carries
   `P: Ordered quantity: 3.0→2`;
2. the order's deleted-line flag becomes true and the thread carries
   `<product name>: Deleted line (quantity: <quantity>)`;
3. the order's edited flag is true;
4. after the session closes, the session thread carries
   `Edited order(s) during the session:` followed by a bulleted list containing a link to
   this order;
5. writing the deleted-line flag back to false has no effect.

---

## Group D — The mandatory refund scenario

### D1 — Refunding one line of a three-line order

**Given** the standard setting and a paid order O with three lines:

| Line | Product | Quantity | Unit price including tax | Line including tax | Tax-excluded | Tax | Unit cost |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | P | 3 | 12.10 | 36.30 | 30.00 | 6.30 | 6.00 |
| 2 | Q | 1 | 40.50 | 40.50 | 33.47 | 7.03 | 18.00 |
| 3 | R | 2 | 5.00 | 10.00 | 8.26 | 1.74 | 2.00 |
| | | | **Totals** | **86.80** | **71.73** | **15.07** | |

The tax-excluded figures are `round_to_currency( 36.30 ÷ 1.21 ) = 30.00`,
`round_to_currency( 40.50 ÷ 1.21 ) = 33.47` and
`round_to_currency( 10.00 ÷ 1.21 ) = 8.26`; each tax is the line total minus its base.
Order O was tendered 86.80 in cash and delivered in real time.

**When** the cashier refunds **one unit of line 1** and hands back 12.10 in cash,

**Then**
1. a new order R is created in the current session with the refund flag set and the name
   `<order O name> REFUND`; it has its own receipt number and tracking number, its
   own session-unique sequence number, and its cost is flagged as not computed;
2. it has exactly one line: product P, quantity −1, unit price 12.10, refunded line =
   line 1 of order O, with a copy of every lot of the original line;
3. the refund guard passes, because
   `| 3 | − ( | 0 | + | −1 | ) = 2 ≥ 0`;
4. the refund line's tax-excluded amount is −10.00 and its tax-included amount is −12.10;
5. order R's tax amount is −2.10 and its total is −12.10;
6. one cash tender of −12.10 is recorded;
7. order O's line 1 now reports an already-refunded quantity of 1
   (`− ( −1 ) = 1`), and order O still has refundable lines because `3 > 1`;
8. order O's refund-orders count becomes 1 and order R's refunded-order link points at O;
9. a transfer of the return operation type is created — the goods were delivered — moving
   one unit of P back, linked to the original outgoing move as its returned origin, and
   completed.

**And** when the sale and the refund fall in the **same** session, the closing entry
contains:

| # | Name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | 12.97 |
| 2 | `Sales with 21%` | 400000 | | 71.73 |
| 3 | `Refund with 21%` | 400000 | 10.00 | |
| 4 | `<session> - Cash` | 101300 | 74.70 | |
| 5 | *(unnamed)* | 600000 | 34.00 | |
| 6 | *(unnamed)* | 140000 | | 34.00 |
| | **Totals** | | **118.70** | **118.70** |

with

```formula
tax_bucket   = − 15.07 + 2.10 = − 12.97
sales_bucket = − ( 30.00 + 33.47 + 8.26 ) = − 71.73
refund_bucket = + 10.00
cash_bucket  = 86.80 − 12.10 = 74.70
cost_bucket  = ( 3 × 6.00 + 1 × 18.00 + 2 × 2.00 ) − ( 1 × 6.00 ) = 40.00 − 6.00 = 34.00
```

Note carefully that the **sales** contributions are **not** netted — rows 2 and 3 are
separate, because the sign is part of the sales aggregation key — while the **tax**
contributions **are** netted into row 1, because the tax aggregation key is (account,
repartition line, tags) and carries no sign. The cash contributions are also netted,
because the cash bucket is keyed only by payment method.

**And** one cash statement line of 74.70 is produced, debiting 570000 and crediting
101300; it is reconciled against row 4.

### D2 — The two tax contributions really are netted

**Given** D1,
**Then** the closing entry contains exactly **one** row on account 251000, because the tax
aggregation key is (account, repartition line, tags) and does not carry a sign, while it
contains **two** rows on account 400000, because the sale aggregation key does carry a
sign.

### D3 — The cash bucket nets to zero

**Given** a session containing exactly one sale of 25.00 in cash and one full refund of
25.00 in cash,
**Then** the cash bucket is 0.00, **no** cash statement line is created for the method and
**no** cash receivable line is created, while the sale row (credit 20.66 base, credit 4.34
tax) and the refund row (debit 20.66 base) still both exist and the tax row nets to zero
and is therefore also absent — leaving a sale credit of 20.66 and a refund debit of 20.66
that balance each other.

### D4 — Refunding more than the outstanding quantity is refused

**Given** order O of D1 with one unit of line 1 already refunded,
**When** a refund line for line 1 with quantity −3 is proposed,
**Then** it is refused with *"You cannot refund more than the outstanding quantity for
this product."* because `| 3 | − ( 1 + 3 ) = −1 < 0`.

### D5 — Refunding lines of two different orders in one transmission

**Given** a transmission whose lines refund lines of two distinct orders,
**Then** the transmission is refused with *"You can only refund products from the same
order."*

### D6 — Refunding when no session is open

**Given** a paid order whose configuration has no open session,
**When** the refund action is invoked from the administrative interface,
**Then** it is refused with *"To return product(s), you need to open a session in the point of sale
Shop"*.

### D7 — A full refund before delivery cancels the delivery

**Given** a ship-later order whose transfer is reserved but not completed, and a full
refund of every line,
**Then** the original transfer is cancelled and **no** return transfer is created.

### D8 — A partial refund before delivery reduces the delivery

**Given** the same order with 3 units demanded and a refund of 1 unit,
**Then** the matching move's demanded quantity becomes 2, the move is re-reserved, and
**no** return transfer is created.

### D9 — A refund of a line whose move is already completed creates a return

**Given** a delivered order and a refund of 1 unit,
**Then** a transfer of the return operation type is created (falling back to the outgoing
type with the locations swapped), one move of 1 unit is created with the original outgoing
move recorded as its returned origin, and the transfer is completed.

---

## Group E — The mandatory tax scenario

### E1 — A price-included tax of twenty-one percent on twelve point one zero

**Given** one line, one unit, unit price 12.10, a 21 percent price-included tax, no
discount,
**Then**
1. `tax_excluded = round_to_currency( 12.10 ÷ 1.21 ) = 10.00`;
2. `tax_amount = 12.10 − 10.00 = 2.10`;
3. the line's tax-excluded amount is 10.00 and its tax-included amount is 12.10;
4. the closing entry receives a credit of 10.00 on 400000 named `Sales with 21%`, a credit
   of 2.10 on 251000 named `21%` with a tax base amount of 10.00, and a debit of 12.10 on
   101300.

### E2 — The tax is obtained by subtraction, not by multiplication

**Given** a unit price of 12.13 with the same tax,
**Then** `tax_excluded = round_to_currency( 12.13 ÷ 1.21 ) = 10.02` and
`tax_amount = 12.13 − 10.02 = 2.11`, **not** 2.10; the two amounts sum exactly to 12.13.

### E3 — Quantity and discount

**Given** a unit price of 12.10 with the same tax, quantity 3, discount 10 percent,
**Then**
1. `price_after_discount = 12.10 × 0.90 = 10.89`;
2. the tax-included amount is `round_to_currency( 10.89 × 3 ) = 32.67`;
3. the tax-excluded amount is `round_to_currency( 32.67 ÷ 1.21 ) = 27.00`;
4. the tax is `32.67 − 27.00 = 5.67`;
5. the discount amount of the line is
   `+1 × ( | 36.30 | − | 32.67 | ) = 3.63`.

### E4 — A fiscal position mapping the tax

**Given** a fiscal position mapping the 21 percent tax to a 0 percent tax, applied to the
order,
**Then**
1. the line's taxes-to-apply are the 0 percent tax;
2. the tax-excluded and tax-included amounts are both 12.10;
3. the closing entry receives a credit of 12.10 on the income account mapped by the same
   fiscal position, named `Sales with 0%`, and no tax row for the 21 percent tax.

### E5 — A fiscal position with no mapping at all

**Given** a fiscal position with an empty tax map,
**Then** the taxes pass through unchanged — unless one of the taxes is itself attached to
a fiscal position, in which case the mapped result is the empty set and the line is
untaxed, producing a line named `Sales untaxed`.

### E6 — Global tax rounding

**Given** the company set to round taxes globally and three lines of 1.11 each with a
21 percent price-excluded tax,
**Then** the total tax is `round_to_currency( 1.11 × 0.21 × 3 ) = 0.70`, whereas per-line
rounding would give `0.23 × 3 = 0.69`; the closing entry and the receipt both show 0.70.

---

## Group F — The mandatory cash rounding scenario

### F1 — Rounding to five hundredths, gain

**Given** the configuration with cash rounding on, rounding step 0.05, method round half
away from zero, strategy add-a-rounding-line, rounding profit account 758100, rounding
loss account 658100, rounding **not** restricted to cash; one order of one unit priced
12.13 including 21 percent tax, paid in cash,
**Then**
1. `tax_excluded = 10.02`, `tax_amount = 2.11`;
2. `payable = round( 12.13 , 0.05 , half away from zero ) = 12.15`;
3. the customer hands over 12.15 and the change is 0.00;
4. `order_rounding_difference = 12.15 + ( −12.13 ) = +0.02`;
5. the closing entry contains a credit of 10.02 on 400000, a credit of 2.11 on 251000, a
   credit of 0.02 on 758100 named `Rounding line`, and a debit of 12.15 on 101300; totals
   12.15 against 12.15.

### F2 — Rounding to five hundredths, loss

**Given** F1 with a unit price of 12.12,
**Then** `tax_excluded = 10.02`, `tax_amount = 2.10`, `payable = 12.10`,
`order_rounding_difference = 12.10 − 12.12 = −0.02`, and the rounding line is a **debit**
of 0.02 on 658100.

### F3 — Rounding with change

**Given** F1 and the customer handing over 20.00,
**Then**
1. `applied_rounding = 12.15 − 12.13 = 0.02`;
2. `gross = | 12.13 | − | 20.00 | + 0.02 = −7.85`;
3. the change is 7.85, itself a multiple of 0.05, so the asymmetric rounding leaves it
   unchanged;
4. `20.00 − 7.85 = 12.15`, the payable amount.

### F4 — Rounding restricted to cash, mixed tenders

**Given** F1 with the restriction on, an order totalling 12.13, 10.00 taken on a card and
the rest in cash,
**Then**
1. `payable = 10.00 + round( 2.13 , 0.05 , half away from zero ) = 12.15`;
2. the cash tender is 2.15;
3. the rounding difference is +0.02 and the rounding line is a credit of 0.02 on 758100;
4. the card bucket is 10.00 and the cash bucket is 2.15.

### F5 — Rounding restricted to cash, no cash tender

**Given** F1 with the restriction on and the whole 12.13 taken on a card,
**Then** no rounding is applied: the payable amount is 12.13, the order total is 12.13,
the rounding difference is 0.00 and **no** rounding line is created.

### F6 — The tolerance when an order is marked paid

**Given** F1 with an order totalling 12.13 and a tender of 12.10,
**Then**
1. the rounded total is 12.15, so `12.15 − 12.10 = 0.05 ≠ 0`;
2. the tolerance for the half-away-from-zero method is
   `round_to_currency( 0.05 ÷ 2 ) = 0.03` (0.025 rounded to the currency);
3. the raw difference is `12.13 − 12.10 = 0.03`, which is not greater than the tolerance,
   so the order **is** accepted as paid.

With a tender of 12.09 the raw difference is 0.04, which exceeds the tolerance, and the
order is refused with *"Order <order name> is not fully paid."*

### F7 — The rounding strategy must be add-a-rounding-line

**Given** a configuration with cash rounding on whose rounding method uses the
biggest-tax strategy,
**Then** the configuration is refused with *"The cash rounding strategy of the point of
sale Shop must be: 'Add a rounding line'"*.

### F8 — The rounding definition is frozen during a session

**Given** an open session using a rounding definition,
**When** that definition's step, method or strategy is changed,
**Then** the change is refused with *"You are not allowed to change the cash rounding
configuration while a pos session using it is already opened."*

### F9 — Rounding on an invoice

**Given** F1 with the order invoiced,
**Then** the invoice carries the rounding definition, and its rounding line is adjusted so
that the invoice total equals the 12.15 actually paid: a rounding line of 0.02 is added
(or an existing one increased) and the payment-term line with the largest absolute amount
is reduced by the same 0.02.

---

## Group G — The mandatory loyalty scenario

### G1 — A ten percent reward claimed after one hundred points

**Given** the standard setting plus a loyalty programme granting one point per unit of
currency spent, with one reward: a ten percent discount on the order, costing one hundred
points; the discount product's income account is 400000 and it inherits the 21 percent tax
of the discounted lines; a customer holding exactly 100 points,

**When** the cashier selects that customer, adds goods for 60.50 including tax, and claims
the reward,

**Then**
1. `goods_excluded = round_to_currency( 60.50 ÷ 1.21 ) = 50.00`, `goods_tax = 10.50`;
2. `reward_included = − round_to_currency( 60.50 × 10 ÷ 100 ) = − 6.05`;
3. `reward_excluded = round_to_currency( −6.05 ÷ 1.21 ) = − 5.00`,
   `reward_tax = −6.05 − (−5.00) = −1.05`;
4. the order total is `60.50 − 6.05 = 54.45`;
5. a reward line exists carrying the discount product, a negative amount, the reward and
   the coupon consumed, and a price type of `automatic`;
6. on transmission the card's balance drops by 100 and the coupon is marked used;
7. the closing entry, when the reward product shares the income account of the goods,
   contains one sale row crediting `50.00 − 5.00 = 45.00` on 400000, one tax row crediting
   `10.50 − 1.05 = 9.45` on 251000, and a debit of 54.45 on 101300; totals 54.45 against
   54.45.

### G2 — The reward on its own income account

**Given** G1 with the discount product's income account set to 708000,
**Then** the closing entry contains a credit of 50.00 on 400000, a **debit** of 5.00 on
708000, a credit of 10.50 and a debit of 1.05 netting to a credit of 9.45 on 251000, and a
debit of 54.45 on 101300.

### G3 — The reward is withdrawn when the order changes

**Given** G1 and the cashier removing goods so that the order no longer qualifies,
**Then** the reward line is removed automatically and the acting user is told *"Some
coupons are invalid. The applied coupons have been updated. Please check the order."*

### G4 — Not enough points

**Given** a customer holding 90 points,
**When** the reward is claimed,
**Then** it is refused with *"There are not enough points for the coupon: <coupon>."*

### G5 — A coupon outside its validity window

**Given** a coupon whose validity starts tomorrow,
**Then** entering it is refused with *"This coupon is not yet valid (<date>)."*; a coupon
whose validity ended yesterday is refused with *"This coupon is expired (<date>)."*

### G6 — A reward product not available at the counter

**Given** a programme whose reward product is not marked available at the counter,
**Then** the operator is told *"To continue, make the following reward products available
in Point of Sale."*

---

## Group H — The mandatory split-bill scenario

### H1 — Splitting the payment of one order

**Given** the restaurant capability, a configuration with bill splitting on, and one order
totalling 100.00 including 21 percent tax on a table,
**When** two guests each settle 50.00, one in cash and one by card,
**Then**
1. the order keeps one identity, one receipt number and one tax computation:
   `base = round_to_currency( 100.00 ÷ 1.21 ) = 82.64`, `tax = 17.36`;
2. two tenders exist: cash 50.00 and card 50.00;
3. the closing entry contains a credit of 17.36 on 251000, a credit of 82.64 on 400000, a
   debit of 50.00 on 101300 named `<session> - Cash` and a debit of 50.00 on 101300 named
   `<session> - Card`;
4. one cash statement line of 50.00 and one accounting payment of 50.00 are produced;
5. totals 100.00 against 100.00.

### H2 — Splitting into two orders

**Given** the same order, and the waiter moving half the lines onto a second order,
**When** each order is settled separately,
**Then**
1. two orders exist, each with its own receipt number, tracking number and tax
   computation;
2. their base lines land in the **same** sales bucket (same account, same sign, same tax
   set), so one sale row is written;
3. their tenders land in the buckets of their own methods;
4. because each order's tax is rounded separately, the sum of the two tax amounts may
   differ by 0.01 from the tax of the undivided order; the entry still balances, because
   the receivable side is the sum of what was actually tendered.

### H3 — Splitting when one guest pays on a customer account

**Given** H1 with the second guest settling 50.00 on the customer-account method (which
identifies the customer) and the order carrying customer Bee,
**Then** the closing entry contains a debit of 50.00 on 101300 named `<session> - Cash`
and a debit of 50.00 on Bee's receivable account 121000, carrying partner Bee, with the
follow-up exclusion off; no accounting payment and no statement line are produced for the
second half.

### H4 — Splitting with no customer on an identifying method

**Given** H3 without a customer on the order,
**Then** the closing is refused with *"You have enabled the "Identify Customer" option for
Customer Account payment method,but the order <order name> does not contain a customer."*

---

## Group I — The mandatory self-ordering scenario

### I1 — A self-ordered sale paid online

**Given** configuration "Shop" with the self-ordering mode set to mobile ordering, the
service mode set to table service, the restaurant capability installed, pay-after set to
each order, and one online payment method of the bank kind whose journal is the provider's
and whose outstanding account is 101402,

**When** a customer scans the code of table 4, orders goods for 30.25 including 21 percent
tax, submits and pays through the provider, and the provider confirms,

**Then**
1. an order is created in the `draft` state with the origin `mobile`, the self-ordering
   table set to table 4, a receipt number prefixed with the letter `S` and the
   configuration's next backend sequence value;
2. every submitted line is sanitised: the quantity must be a finite number strictly
   greater than zero, the attribute values must belong to the product's template, the
   taxes are recomputed server-side and the line amounts are recomputed server-side;
3. a payment transaction is created for 30.25 in the selling currency;
4. on confirmation a counter payment of 30.25 is created with the online payment method
   and the order becomes `paid`;
5. an order-state-changed notification and a synchronisation notification are broadcast,
   followed by a payment-status notification carrying the result and the order and line
   data;
6. `base = round_to_currency( 30.25 ÷ 1.21 ) = 25.00`, `tax = 5.25`;
7. the closing entry contains a credit of 5.25 on 251000, a credit of 25.00 on 400000 and
   a debit of 30.25 on 101300 named `<session> - <online method name>`;
8. an accounting payment of 30.25 is created in the provider's journal, debiting 101402 and
   crediting 101300, and is reconciled against the closing-entry line.

### I2 — A self order whose payment fails

**Given** I1 with the provider reporting failure,
**Then** the order stays `draft`, a payment-status notification carrying the failure is
broadcast, and removing the order from the interface sets it to `cancel`.

### I3 — A kiosk configuration may not offer cash

**Given** a configuration with the self-ordering mode set to kiosk,
**When** a cash payment method is added,
**Then** the change is refused with *"You cannot add cash payment methods in kiosk mode."*

### I4 — Pay-after is forced

**Given** a configuration set to kiosk mode, or to mobile mode with pickup-zone service,
or to mobile mode without the restaurant capability,
**Then** pay-after is forced to each order on every write. Conversely, choosing pay-after
at meal end in mobile mode forces the service mode to table service.

### I5 — An invalid preset is refused

**Given** a configuration using presets,
**When** an order is submitted naming no preset, a preset that is not available in
self-ordering and is not the default, or a preset outside the configuration's available
list,
**Then** it is refused with, respectively, *"Invalid preset"*, *"Preset is not available
in self-ordering"* and *"Preset is not available in this configuration"*.

### I6 — A quantity that is not a positive finite number is refused

**Given** a submitted line with a quantity of zero, a negative quantity, a boolean, or a
non-finite value,
**Then** it is refused with *"Invalid quantity"*. Returns cannot be performed through the
public route.

### I7 — An attribute value from another product is refused

**Given** a submitted line carrying an attribute value belonging to a different product
template,
**Then** it is refused with *"Invalid product attribute"*.

### I8 — Rotating the access token invalidates printed codes

**Given** printed table codes,
**When** the configuration's access token is rotated,
**Then** every table identifier is regenerated and every previously printed code stops
working.

### I9 — A self order paid at the counter

**Given** a configuration with pay-after at meal end and a submitted order held on table
4,
**Then** the order appears among the unfinished orders of the session, the cashier opens
it and takes payment as for any other order, and the closing entry treats it as an
ordinary order.

---

## Group J — The mandatory pay-later scenario

### J1 — A pay-later order settled afterwards

**Given** the standard setting, customer Acme with receivable account 121000, and an order
of 60.50 including 21 percent tax settled entirely on the "Customer Account" method
(which identifies the customer), not invoiced,

**When** the session is closed,

**Then**
1. `base = round_to_currency( 60.50 ÷ 1.21 ) = 50.00`, `tax = 10.50`;
2. the closing entry contains a credit of 10.50 on 251000, a credit of 50.00 on 400000 and
   a **debit of 60.50 on 121000 carrying partner Acme**, named `<session> - Customer
   Account`, with the follow-up exclusion **off**;
3. no statement line and no accounting payment are created;
4. Acme's ageing shows 60.50 outstanding and the amount is subject to dunning.

**When** Acme later pays 60.50 by bank transfer,

**Then** an inbound accounting payment of 60.50 is registered with destination account
121000, its receivable line (credit 60.50, partner Acme) is reconciled against the
closing-entry line (debit 60.50, partner Acme), and account 121000 returns to zero for
Acme.

### J2 — A pay-later order that is invoiced

**Given** J1 with the order invoiced,
**Then**
1. the pay-later tender produces **no** closing-entry line, because pay-later tenders of
   invoiced orders are excluded;
2. **no** invoice payment entry is produced, because pay-later tenders are skipped;
3. the invoice carries the receivable of 60.50 on 121000 and, because a pay-later tender
   is present, the customer's payment terms are applied to it;
4. settlement follows the ordinary receivable flow.

### J3 — A pay-later method that does not identify the customer

**Given** a customer-account method with the identify-customer flag cleared, and an order
of 60.50 settled on it,
**Then** the closing entry contains one aggregated debit of 60.50 on the method's
intermediary account (falling back to 101300) with **no** partner, still with the
follow-up exclusion off.

### J4 — Mixed pay-later and cash

**Given** an order of 60.50 with 20.00 in cash and 40.50 on Acme's customer account,
**Then** the closing entry contains a debit of 20.00 on 101300 named `<session> - Cash`
and a debit of 40.50 on 121000 with partner Acme; one cash statement line of 20.00 is
produced.

---

## Group K — Invoicing

### K1 — Invoicing at the counter

**Given** an order of 100.00 including 21 percent tax with customer Acme and the invoice
request turned on, paid by card, in an open session,
**When** the order is transmitted,
**Then**
1. the order reaches the `paid` state, then immediately the `done` state;
2. an invoice is created in the invoice journal, dated the order date (because the session
   is not closed), referenced with the order name, with the order's receipt number as its
   origin, the customer's invoice address, the order's fiscal position, and no payment
   terms (because no pay-later tender is present);
3. the invoice is posted; it carries a credit of 82.64 on 400000, a credit of 17.36 on
   251000 and a debit of 100.00 on 121000;
4. an invoice payment entry is created in the point of sale journal, debiting 100.00 on
   101300 and crediting 100.00 on 121000 with partner Acme;
5. the two 121000 lines are reconciled;
6. a message is posted on the invoice: *"This invoice has been created from the point of
   sale session:"* followed by a link to the order.

### K2 — Invoicing after the session closed

**Given** order C of scenario B1, invoiced **after** the session was closed,
**Then**
1. the order is already `done`;
2. an invoice and an invoice payment entry are created as in K1, except that the invoice
   date is now and the payment entry's counter-side account is the method's intermediary
   account (falling back to 101300) for a non-identifying method, or the partner's
   receivable for an identifying one;
3. a **reversal entry** is created in the point of sale journal, dated today, referenced
   *"Reversal of point of sale closing entry <closing entry number> for order <order name> from
   session <session identifier>"*, containing the negation of the order's own accounting
   values: a debit of 82.64 on 400000, a debit of 17.36 on 251000, a credit of 100.00 on
   the receivable account of the tender, plus the negated stock pair;
4. the reversal entry is posted and its lines are reconciled, per account, against the
   payment entry's lines (or, when there are none, against the closing entry's lines
   carrying this partner on the partner's receivable account);
5. the reversal entry's displayed signed total is negated, and the entry is treated as a
   storno entry when the company uses storno accounting.

### K3 — Consolidated invoicing

**Given** three paid orders of customer Acme on configuration "Shop", all with the same
employee and the same fiscal position, none of them invoiced,
**When** the invoice wizard is confirmed with consolidated billing,
**Then** one invoice is produced whose origin lists the three receipt numbers joined by a
comma and a space, whose reference is empty (because there is more than one order), and
whose lines are the concatenation of the three orders' lines.

### K4 — Consolidated invoicing refused for a refund of an invoiced order

**Given** a selection of two orders, one of which refunds an invoiced order,
**When** consolidated billing is asked for,
**Then** it is refused with *"The following refund orders can't be part of a consolidated
invoice because they refunded invoiced orders. Each refund order should be handled
separately.\n\n<order name> (<receipt number>)"*.

### K5 — Consolidated invoicing with a missing customer

**Given** a selection spanning two configurations where one order has no customer,
**Then** the grouping refuses with *"Kindly ensure that each order contains a customer."*

### K6 — Consolidated invoicing with one customer and some orders missing it

**Given** a selection on one configuration where exactly one customer appears and some
orders carry none,
**Then** the confirmation wizard is offered, reading *"It seems that the point of sale order(s)
<names> do not have a customer.\n\nWould you like to set <customer name> as the customer
for the selected point of sale order(s)?"*; confirming writes that customer onto every order and
reopens the invoice wizard.

### K7 — No invoiceable order

**Given** a selection whose orders are all already invoiced, unfinished or cancelled,
**Then** the wizard refuses with *"No valid orders were selected. No new invoices could be
generated"*.

### K8 — A credit note for a refund of an invoiced order

**Given** an invoiced order O and a refund order R of the whole of O, itself invoiced,
**Then** the document produced for R is a credit note, referenced *"Reversal of: <O's
invoice number>"*, linked as the reversal of O's invoice, and O's invoice appears among
R's invoice's refunded invoices.

### K9 — Resetting a counter invoice to draft while the session is open

**Given** an invoice of an order whose session is still open,
**When** the invoice is reset to draft,
**Then** nothing happens and the acting user receives a sticky danger notification reading
*"You can't reset this invoice to draft because the point of sale session is still open. Please
close the ongoing session first, then try again."*

### K10 — The customer claims the invoice from the receipt

**Given** an order with the receipt code `A1B2C` and the receipt number `260-3-000127`,
dated 3 March 2026,
**When** the customer opens the request page and submits the receipt number `260-3-000127`,
the date 2026-03-03 and the code `A1B2C`,
**Then** the order is found — the receipt number is matched by suffix and the date window
is 2 March 2026 inclusive to 5 March 2026 exclusive — and the customer is redirected to
the validation page with the order's access token.

Submitting a receipt number of eleven characters is refused with *"The Ticket Number
should be at least 12 characters long."*; omitting any field is refused with *"Please fill
all the required fields."*; a lookup that finds nothing is refused with *"No sale order
found."*

---

## Group L — Inventory

### L1 — Real-time delivery

**Given** the company set to update stock in real time and an order of 2 units of a
storable product,
**When** the order becomes paid,
**Then** a transfer of the configuration's operation type is created from its source
location to the customer location of the order's partner (falling back to the operation
type's default destination), with one move of 2 units marked picked, and the transfer is
completed.

### L2 — Deferred delivery

**Given** the company set to update stock at session closing and three orders,
**When** the session is validated,
**Then** one transfer per destination location is created covering the lines of every
closed order that does not force real-time creation and has no shipping date; the
transfers carry the session and the session name as their origin; the transfer is
completed.

### L3 — The deferred flag is frozen

**Given** a session opened while the company updates stock at closing,
**When** the company setting is changed to real time mid-session,
**Then** the session keeps its deferred behavior for its whole life.

### L4 — Real-time creation is forced for an invoiced order under cost-at-invoicing

**Given** the company using cost-at-invoicing accounting, the session deferring stock, and
an order asking to be invoiced,
**Then** the transfer is created immediately, and that order's transfers are excluded from
the deferred creation at closing.

### L5 — A failed delivery does not block the sale

**Given** insufficient stock,
**Then** the completion is attempted, refused and swallowed; the transfer remains
incomplete; the order still becomes paid; the session's failed-transfer flag is true and
the transfer count button is highlighted.

### L6 — Ship later

**Given** the ship-later flag on and an order with a shipping date of 20 April 2026,
**Then**
1. a stock reference named after the order is created and attached to the order;
2. every line of a consumable or storable product raises a procurement for its quantity to
   the customer location, with the shipping date converted from the acting time zone to
   universal time as both the planned date and the deadline, carrying the deferred route,
   the warehouse, the partner and the reference;
3. the resulting transfers are confirmed;
4. for each tracked product the move lines are rebuilt from the order's lots **without**
   being marked done.

### L7 — Lot capture

**Given** a serial-tracked product and an operation type that uses existing lots, and an
order line carrying two scanned serial numbers, one of which already exists,
**Then**
1. the existing serial is matched;
2. the missing one is created when the operation type also creates lots;
3. one unit is assigned per serial;
4. the matched serial's quantity is spread over the stock quantities holding it in the
   source location, newest first; any remainder becomes a move line naming the lot
   directly.

### L8 — A unit conversion that rounds to zero

**Given** a move whose demanded quantity converts to zero in the product's reference unit,
**Then** the completion is refused with a message beginning *"Conversion Error: The
following unit of measure conversions result in a zero quantity due to rounding:"*,
listing *" - From "<source unit>" to "<target unit>""* and ending with the explanation of
how to fix it.

### L9 — No delivery notification

**Given** any counter transfer,
**Then** the ordinary transfer confirmation email and text message are **not** sent.

### L10 — Cost computation at closing

**Given** the session deferring stock and a line of a first-in-first-out product,
**When** the session is validated,
**Then** the deferred transfers are created first, then the cost of every not-yet-costed
closed order is computed from those transfers' moves, and only then is the closing entry
built — so the cost-of-goods-sold rows carry the correct figures.

---

## Group M — Multi-currency and multi-company

### M1 — A foreign selling currency

**Given** a configuration whose journal carries a currency other than the company currency,
with a rate of 0.80 company units per selling unit,
**Then**
1. every closing-entry line except the cost and valuation rows carries an amount in
   currency in the selling currency and a balance in the company currency;
2. the cost and valuation rows carry **no** amount in currency;
3. the sales and tax balances are produced by the tax engine using the **order's** rate at
   the order date;
4. the tender balances are produced by converting at the **payment's** date;
5. when a rate change occurs mid-session, those two can legitimately differ, and the
   resulting imbalance is surfaced by the balance check rather than silently absorbed.

### M2 — A cash journal in the company currency with a foreign selling currency

**Given** M1 and a cash bucket of 25.00 in the selling currency,
**Then** the statement line carries an amount of
`convert( 25.00 , selling → journal currency , at the session closing instant ) = 20.00`,
an amount in currency of 25.00 and the selling currency as its foreign currency.

### M3 — The balancing line in a foreign currency

**Given** a forced close with an amount to balance of 3.00 in company currency,
**Then** the balancing line credits 3.00 in company currency and
`convert( 3.00 , company → selling , at today )` in the selling currency; when the session
already works in company currency, the selling-currency figure is 0.00.

### M4 — Multi-company visibility

**Given** two companies each with a configuration,
**When** a user whose active company set contains only the first company lists sessions,
orders, payments, methods, configurations or analysis rows,
**Then** only the records of the first company are visible, enforced by the record rules.

### M5 — A payment method of the wrong company

**Given** a configuration of company A,
**When** a payment method of company B is added,
**Then** the change is refused with *"The payment methods for the point of sale Shop must
belong to its company."*

### M6 — The selling application is locked to one company

**Given** a user with two active companies,
**When** the selling application is opened,
**Then** the acting company set is forced to the single company of the session, and the
allowed-companies list transmitted to the browser contains only that company.

---

## Group N — Rounding edge cases

### N1 — Half exactly, away from zero

**Given** a rounding step of 0.05 and the half-away-from-zero method,
**Then** 12.125 rounds to 12.15 and −12.125 rounds to −12.15.

### N2 — Half exactly, toward zero

**Given** the half-toward-zero method,
**Then** 12.125 rounds to 12.10 and −12.125 rounds to −12.10.

### N3 — Asymmetric rounding on a negative amount

**Given** a rounding step of 0.05 and the round-away-from-zero method,
**Then** symmetric rounding of −12.12 gives −12.15, while **asymmetric** rounding gives
−12.10, because the method is inverted for negative values. The remaining amount due and
the change use asymmetric rounding; the payable amount uses symmetric rounding.

### N4 — The double zero test

**Given** two values whose rounded forms differ but whose difference rounds to zero,
**Then** the comparison reports them equal, because the test checks both the raw
difference and the rounded difference.

### N5 — Combo distribution residual

**Given** a menu priced 11.99 with two combos of base prices 8.00 and 4.00, one component
chosen from each,
**Then**
1. `original_total = 12.00`;
2. the first component takes `round( 8.00 × 11.99 ÷ 12.00 ) = 7.99`, leaving a remainder of
   `11.99 − 7.99 = 4.00`;
3. the second, being the last, takes `round( 4.00 × 11.99 ÷ 12.00 ) = 4.00` and then
   absorbs the remaining 0.00;
4. the components sum exactly to 11.99.

### N6 — Combo distribution when the last component has a quantity above one

**Given** a menu whose last chosen component has a quantity of 3 and a parent coefficient
of 1,
**Then** before the distribution the component is split into one entry of quantity 2 and
one of quantity 1, so that the residual can be placed on a single unit.

### N7 — Tax on an inclusive price that does not divide evenly

**Given** inclusive prices of 0.01 through 0.10 with a 21 percent price-included tax,
**Then** the base is `round_to_currency( price ÷ 1.21 )` and the tax is the remainder;
for 0.01 the base is 0.01 and the tax is 0.00; for 0.10 the base is 0.08 and the tax is
0.02; in every case base plus tax equals the price exactly.

---

## Group O — Data loading and offline operation

### O1 — The session opens with the master data

**Given** a session in the `opening_control` state,
**When** the selling application loads,
**Then** it receives, per entity, the loading field set, the relation descriptions and the
records matching that entity's condition; a payment method that is archived is still
included; only the acting user is included among the users.

### O2 — Incremental loading

**Given** the application supplying a last server date,
**Then** every entity's condition is narrowed to records written after that date, except
the session, the configuration and the user, which are always sent in full.

### O3 — Stale local records

**Given** the application holding identifiers that were deleted, archived, or became
unreadable,
**When** it asks the server to filter its local data,
**Then** the server answers, per entity, the union of the non-existent identifiers and the
irrelevant ones.

### O4 — The product load limit

**Given** the product load limit set to 5000 and 6200 matching products,
**When** the application asks for the loading information,
**Then** it receives a count of 6200 and a limit of 5000, and warns the operator before
triggering a full synchronisation.

### O5 — On-demand partner fetch

**Given** the partner load limit of 100 and an empty search condition with an offset of
100,
**Then** the next hundred partners of the bounded selection (ordered by counter order count
descending, then by name) are returned together with their fiscal positions. With a
non-empty search condition, the whole partner set is searched, one hundred at a time from
the offset.

### O6 — Offline selling and reconnection

**Given** the network unavailable after the master data was loaded,
**When** the cashier sells three orders and the network returns,
**Then** the three orders are transmitted, each identified by its universally unique
identifier; a replay of any of them is harmless; the local store is cleared of the
transmitted orders.

### O7 — The data-change signal

**Given** the configuration's default pricelist changed while a session is open,
**Then** the configuration's last-data-change instant is updated, the application notices
the difference against its cached copy and reloads the master data.

### O8 — The change feed ignores a device's own echo

**Given** two devices on the same configuration,
**When** device 1 transmits an order,
**Then** a synchronisation notification carrying device identifier 1 is broadcast; device
1 ignores it and device 2 applies it.

---

## Group P — Restaurant service

### P1 — Matching an order by table

**Given** the restaurant capability and an unfinished order on table 4,
**When** a payload with a different universally unique identifier but the same table and
the `draft` state is transmitted,
**Then** the existing order is matched and updated rather than a new one being created.

### P2 — A default floor is created

**Given** a configuration created with the restaurant capability and no floor,
**Then** a floor named after the company is created with one square table numbered 1,
one seat, at horizontal 100 and vertical 100, sized 130 by 130.

### P3 — Turning the restaurant capability off

**Given** a restaurant configuration,
**When** the capability flag is cleared,
**Then** the floor list is emptied and the tip-after-payment flag is forced off.

### P4 — A floor may not be removed while in use

**Given** a floor attached to a configuration with an open session,
**When** it is deleted,
**Then** it is refused with *"You cannot remove a floor that is used in a PoS session,
close the session(s) first: \n"* followed by *"Floor: <floor name> - PoS Config:
<configuration name> \n"*.

### P5 — A table may not be removed while orders sit on it

**Given** a table with an unfinished order,
**When** it is deleted or deactivated,
**Then** it is refused with *"You cannot delete a table when orders are still in draft for
this table."*

### P6 — Table grouping cannot create a cycle

**Given** table A whose parent is table B,
**When** table B's parent is set to table A,
**Then** the assignment is silently reverted to table B's previous parent.

### P7 — Courses

**Given** an order with two courses, index 0 and index 1,
**When** course 0 is fired,
**Then** its fired flag becomes true, its fired instant is stamped, and only its lines are
sent to the preparation printers.

### P8 — Preparation delta and the out-of-order guard

**Given** an order whose stored last-preparation-change document carries a server date of
12:05,
**When** a payload carrying a server date of 12:03 arrives,
**Then** the stored document is kept, the payload's is discarded, and the server log
records an outdated preparation change. When the payload carries 12:07, it is accepted and
its server date is replaced by the current server time.

---

## Group Q — Permissions

### Q1 — A cashier may not record a cash movement without accounting rights

**Given** a user in the point of sale user group only,
**When** a cash in is attempted,
**Then** it is refused with *"You don't have the access rights to perform a cash in/out."*

### Q2 — A counter administrator may record one

**Given** a user in the point of sale administrator group,
**Then** the cash in succeeds and a statement line is created labelled
`<session identifier>-<translated movement kind>-<reason>`.

### Q3 — Deleting a cash movement of another session

**Given** a permitted user and a statement line belonging to a different session,
**When** the deletion is attempted,
**Then** it is refused with *"You cannot delete a cash move that is not linked to this
session."*

### Q4 — A cashier sees only counter statement lines

**Given** a user in the point of sale user group and a mixture of counter and ordinary
bank statement lines,
**Then** only the lines carrying a session are visible.

### Q5 — A cashier sees only counter invoices

**Given** the same user and a mixture of counter invoices and ordinary invoices,
**Then** only the entries carrying at least one counter order, and their items, are
visible.

### Q6 — The receipt header is administrator-only

**Given** a counter administrator who is not a platform administrator,
**When** the receipt header is written,
**Then** it is refused as an access error with *"Only administrators can edit receipt
headers and footers"*.

### Q7 — The selling application refuses a non-internal user

**Given** a portal or public user,
**When** the selling application address is opened,
**Then** the answer is not-found.

### Q8 — The selling application refuses the superuser

**Given** the system superuser outside a test environment,
**When** the application is opened,
**Then** it is refused with *"You do not have permission to open a point of sale session. Please try
opening a session with a different user"*.

### Q9 — An unreadable entity does not block the session

**Given** a user who may not read one of the loaded entities,
**When** the session loads its data,
**Then** that entity comes back empty, the failure is logged, and the session still opens.

---

## Group R — Protections on shared data

### R1 — The tax is frozen during a session

**Given** an open session containing an order line carrying the 21 percent tax,
**When** the tax's amount is changed,
**Then** it is refused with *"It is forbidden to modify a tax used in a point of sale order not
posted. You must close the point of sale sessions before modifying the tax."*

### R2 — The lock date is frozen during a session

**Given** an open session that started on 5 January 2026,
**When** the company's tax lock date is set to 31 January 2026,
**Then** it is refused with *"Please close all the point of sale sessions in this period
before closing it. Open sessions are: <session names> "*.

### R3 — A journal attached to a method is protected

**Given** a bank journal attached to a payment method,
**When** its type is changed, or it is archived or deleted,
**Then** it is refused with, respectively, *"This journal is associated with a payment
method. You cannot modify its type"* and *"You can not archive this journal because it is
set on the following payment method : Card."*

### R4 — A sequence used by a configuration is protected

**When** one of the four configuration sequences is deleted,
**Then** it is refused with *"You cannot delete a sequence used in an active point of sale config:
<sequence names>"*.

### R5 — A category is protected while any session is open

**Given** any open session anywhere,
**When** a counter category is deleted,
**Then** it is refused with *"You cannot delete a point of sale category while a session is
still opened."*

### R6 — A customer with counter orders cannot be deleted

**When** such a partner is deleted,
**Then** it is refused with *"You cannot delete a customer that has point of sales orders.
You can archive it instead."*

### R7 — A payment method is frozen during a session

**Given** a method offered on a configuration with an open session,
**When** anything but its sequence is written,
**Then** it is refused with *"Please close and validate the following open PoS Sessions
before modifying this payment method.\nOpen sessions: <session names>"*.

### R8 — A cash method belongs to one shop

**When** a cash method is attached to a second configuration,
**Then** it is refused with *"Validation Error: You cannot assign the same Cash payment
method to multiple point of sale Shops. Please create a separate Cash payment method for each
shop."*

### R9 — A cash journal carries one method

**When** a second payment method is put on the same cash journal,
**Then** the configuration check refuses with *"You cannot use the same journal on
multiples cash payment methods."*

### R10 — A preset attached to a configuration cannot be deleted

**Then** it is refused with *"You cannot delete a preset that is linked to a point of sale
configuration."*

---

## Group S — Reporting

### S1 — The sales details document for one session

**Given** the session of scenario B1,
**When** the session report is rendered,
**Then**
1. the products section lists P and Q with quantity 1 each, their unit prices, their
   totals paid and their base amounts, grouped by counter category (or under "Not
   Categorized");
2. the taxes section lists the 21 percent tax with a base amount of 54.13 and a tax amount
   of 11.37 for the sold section, using the base amounts of every line of the selected
   orders;
3. the payments section lists Cash with a total of 25.00, an expected count of 25.00, a
   counted amount of 24.50 and a difference of −0.50, with a movement list, and Card with a
   total of 140.50;
4. the invoice list contains order C's invoice with its signed total;
5. the grand total paid is the sum of the captured payments of the session.

### S2 — The sales details document for a date range

**Given** a range from 1 March 2026 to 3 March 2026 and two configurations,
**Then** the orders selected are the paid and posted orders of those configurations whose
date lies in the range, the per-method totals across sessions are shown (they are hidden
when specific sessions were requested), and the state reported is the literal value
"multiple".

### S3 — Default period

**Given** neither a start nor a stop instant,
**Then** the start is today at midnight in the acting time zone expressed in universal
time and the stop is the start plus one day minus one second. A stop earlier than the
start is replaced by the start plus one day minus one second.

### S4 — Reporting currency

**Given** two configurations with different currencies,
**Then** the reporting currency is the company currency; with a single shared currency, it
is that currency, and every order total is converted into it at the order date when the
order's pricelist currency differs.

### S5 — Cash rounding total

**Given** a period containing three orders whose paid amounts exceed their totals by 0.02,
−0.01 and 0.03,
**Then** the cash rounding total is `round_to_reporting_currency( 0.02 − 0.01 + 0.03 ) =
0.04`.

### S6 — A session with no cash method

**Given** a session whose configuration offers no cash method,
**Then** a synthetic cash row named `Cash <session identifier>` is inserted first, with a
total of zero, an expected count equal to the previous closed session's counted ending
balance plus this session's frozen cash transactions, the counted ending balance, and a
movement list that drops its last entry when the difference is not zero (because that
entry is the difference itself).

### S7 — The order analysis rows

**Given** the session of B1,
**Then** the analysis view exposes one row per order line, each with its date, order,
employee, customer, product, product category, counter category, company, configuration,
session, state, pricelist, signed quantity, totals with and without tax, average price,
total cost, margin, line count, invoiced flag and validation delay in days between the
order date and the invoice date.

### S8 — The digest indicator

**Given** a period containing the three orders of B1,
**Then** the "POS Sales" indicator reports `25.00 + 40.50 + 100.00 = 165.50` in the company
currency; a recipient who is not in the point of sale user group has the indicator skipped
because the computation raises *"Do not have access, skip this data for user's digest
email"*.

---

## Group T — Forced close and recovery

### T1 — A forced close

**Given** a session whose closing entry does not balance by 3.00 in company currency,
**When** the validation is attempted,
**Then**
1. the whole transaction is rolled back — the deferred transfers, the statement lines and
   the accounting payments created during the attempt are all undone;
2. the forced-close wizard is offered, carrying the amount to balance, the default
   balancing account (the company's default counter receivable account, falling back to
   the fallback customer receivable account) and the message *"There is a difference
   between the amounts to post and the amounts of the orders, it is probably caused by
   taxes or accounting configurations changes."*;
3. confirming re-runs the validation with that account and amount, adding one line named
   `Difference at closing PoS session` crediting 3.00 on the chosen account;
4. the entry then balances and is posted.

### T2 — The account is read-only for a user without accounting rights

**Given** a counter administrator without accounting read rights,
**Then** the wizard's account field is read-only and the default is used.

### T3 — A rescue session closes without a cash difference

**Given** a rescue session with cash control containing two cash tenders of 10.00 and 5.00
and a starting balance of 0.00,
**When** the closing control is requested,
**Then** the counted ending balance is computed automatically as `0.00 + 15.00 = 15.00`,
the difference is 0.00, no cash difference line is created, and the session is validated
immediately.

### T4 — A rescue session cannot be closed from the selling application

**Given** a rescue session,
**Then** the selling application offers no closing path for it; it is closed from the
administrative interface.

---

## Group U — Presets and scheduling

### U1 — A preset switches the pricelist and the fiscal position

**Given** a preset carrying a take-away pricelist and a take-away fiscal position,
**When** it is chosen on an order,
**Then** every line's price and taxes are recomputed with that pricelist and that fiscal
position.

### U2 — A return preset negates the cart

**Given** a preset with the return-mode flag set,
**Then** every quantity added to the cart is negative and the order behaves as a refund
order.

### U3 — Slot capacity

**Given** a preset with a capacity of 5, an interval of 20 minutes and a schedule from
11:30 to 14:00, and four orders already booked at 12:30,
**Then** the slot at 12:30 accepts a fifth order and refuses a sixth; the generated
instants are 11:30, 11:50, 12:10, 12:30, 12:50, 13:10, 13:30 and 13:50.

### U4 — Attendance sanity

**Given** an attendance line from 15.0 to 9.0,
**Then** the preset is refused with *"The start time must be before the end time."*

---

## Group V — Barcodes

### V1 — A weighted product barcode

**Given** a rule of the weighted kind and a scanned code encoding product P and a weight of
1.234 kilograms,
**Then** a line for product P is added with a quantity of 1.234.

### V2 — A priced product barcode

**Given** the shipped rule `23.....{NNNDD}` on the thirteen-digit European article number
encoding, and a code encoding 12.34,
**Then** the product is added with a unit price of 12.34 and a price type of `manual`.

### V3 — A discount barcode

**Given** the shipped rule `22{NN}` and a code encoding 15,
**Then** a discount of 15 percent is applied to the selected line.

### V4 — A customer barcode

**Given** the shipped rule with pattern `042`,
**Then** the matching partner becomes the order's customer.

### V5 — A cashier barcode

**Given** the shipped rule with pattern `041`,
**Then** the matching operator is logged in when the employee capability is installed.

### V6 — Fallback order

**Given** a code matching no rule,
**Then** it is tried as a product barcode, then as a product-unit barcode, then as a lot
name, and finally reported as unknown.

---

## Group W — The customer display

### W1 — A correct token renders the display

**Given** a configuration with access token `abc123`,
**When** the display address is opened with the parameter `access_token=abc123`,
**Then** the page is rendered with the configuration identifier, the token, whether a
background image exists, the company identifier, the proxy address and the device
identifier.

### W2 — A wrong token is refused

**When** the address is opened with any other token, or with none,
**Then** the answer is not-found. The comparison is made in constant time.

### W3 — The order is pushed to the display

**When** the cashier changes the order,
**Then** a display-update message addressed to that device identifier carries the order.

---

## Group X — Housekeeping

### X1 — The stale-session reminder

**Given** a session opened eight days ago, still not closed, with no activity,
**When** the replenishment scheduler runs,
**Then** an activity of the shipped old-session type is scheduled for the session's
opening user with the note *"Your PoS Session is open since <opening instant>, we advise
you to close it and to create a new one."* A session that already has an activity receives
no second one.

### X2 — The replenishment trigger

**When** a session is validated,
**Then** the scheduler is triggered on the moves of the session's transfers, with elevated
rights, so that reordering rules see the consumption immediately.

### X3 — The settings screen does not fire the frozen-field guard spuriously

**Given** an open session and the settings screen re-saved without changing any frozen
field,
**Then** the write succeeds, because unchanged values are dropped from the write before the
guard runs.

### X4 — The settings screen unlinks what it omits

**Given** a configuration with three available pricelists and a settings save carrying link
commands for only two of them,
**Then** the third is unlinked, because the omitted links are turned into explicit unlink
commands.
