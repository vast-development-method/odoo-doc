# Point of Sale — Workflows

End-to-end operational procedures, step by step, with the role that performs each one,
the preconditions, the records created or updated at each step, and the failure
conditions.

Roles used in this file:

| Role | Meaning |
| --- | --- |
| **Cashier** | A user in the point of sale user group. May open and close sessions, sell, refund and invoice. |
| **Counter administrator** | A user in the point of sale administrator group, which implies the cashier group and the inventory user group. May change prices when price control is restricted, post a cash difference above the authorised limit, and manage configurations, categories, presets and payment methods. |
| **Accountant** | A user with accounting rights. Sees the closing entry, the statement lines and the reconciliations. |
| **Customer** | An unauthenticated visitor, acting through the receipt's portal link or the self-ordering pages. |
| **System** | The platform acting on its own: scheduled jobs, automatic creation, notifications. |

---

## 1. Preparing a point of sale

**Role**: counter administrator. **Precondition**: the company has a chart of accounts.

1. Create a Point of Sale Configuration, giving it a name. When the acting company has no
   warehouse at all, one is created first, coded with the first three characters of the
   configuration name.
2. The system creates four numbering sequences for the configuration: the order sequence,
   the backend order sequence, the order line sequence and the device sequence, all with
   the code `pos.order`, six digits of padding (zero for the device sequence) and the
   no-gap implementation.
3. The system installs any capability requested by a `module_` flag that is set, and
   extends the implied security groups required by any `group_` flag.
4. The system refreshes the visibility of the preparation printers menu: it is visible
   when at least one configuration has order printing enabled.
5. The administrator sets the journals. When nothing is chosen, the point of sale journal
   defaults to the miscellaneous journal coded with the four letters `POSS` of the acting
   company (created if missing) and the invoice journal defaults to the first sale
   journal.
6. The administrator sets the payment methods. When none are chosen, the default set is
   computed as described in [`entities.md`](entities.md) section 1.13, creating a cash
   method, a card method and a customer-account method if nothing usable exists.
7. Optionally: pricelists, fiscal positions, category restriction, presets, printers,
   receipt header and footer, cash rounding, tips, ship later, trusted configurations.

**Failure conditions** are listed in [`business-rules.md`](business-rules.md) section 2.

### 1.1 Loading a shipped scenario

**Role**: counter administrator. Four scenarios are shipped: clothes shop, bakery shop,
furniture shop and plain retail; the restaurant capability adds a bar scenario and a
restaurant scenario.

1. Create the journal and the standard payment methods for the scenario, giving the cash
   journal a scenario-specific name and hiding it from the accounting dashboard.
2. Create the configuration with that journal and those payment methods, plus the
   scenario's own settings (for example bill splitting and the restaurant capability for
   the bar scenario).
3. Register a stable external reference for the configuration, suffixed with the company
   identifier when the acting company is not the main one.
4. Load the scenario's category data, and, when demonstration data is requested, its
   product data — first loading the generic product demonstration data when the product
   capability itself carries none.
5. Restrict the configuration to the scenario's categories.
6. For the restaurant and bar scenarios, load the floor data if it is not already there
   and attach the main floor and the patio floor to the configuration.
7. For the furniture and restaurant scenarios, when demonstration data is requested, the
   acting company is the main company and the demonstration session does not already
   exist, load a set of demonstration orders and a closed demonstration session.

---

## 2. Opening a session

**Role**: cashier. **Precondition**: the configuration is active and correctly
configured.

1. The cashier opens the selling application for a configuration. Before anything else the
   system validates the configuration:
   - the company has a chart of accounts;
   - the default pricelist belongs to no company or to the configuration's company, and
     every available pricelist does likewise;
   - every payment method belongs to the configuration's company;
   - the currencies of the payment methods, of the available pricelists and of the invoice
     journal all agree with the configuration currency;
   - when cash control is on, every cash payment method's journal has both a profit and a
     loss account;
   - at least one payment method is configured;
   - every stored field passes its own validation;
   - the company has a fiscal country.
2. When no session is open for the configuration, a session is created. Creating it
   requires a configuration and checks the one-open-session rule and the lock dates.
3. The system pre-fills the starting balance with the counted closing balance of the
   previous session (zero when there is none) and generates a change-feed access token.
4. The browser is redirected to the selling application for that configuration.
5. The selling application downloads the master data (section 3) and, when cash control is
   on and the session is still in opening control, shows the opening count screen.
6. The cashier counts the drawer, enters the amount and optional notes, and confirms.
7. The system stamps the opening instant, writes the notes, posts the opening difference
   in the session thread as three lines (difference, expected, counted), overwrites the
   starting balance with the counted amount, moves the session to the opened state and
   assigns the session name from the session sequence.

**Concurrency.** Opening the selling application takes a row-level lock on the
configuration without waiting, so two simultaneous openings cannot create two sessions.
When the configuration already has an active session that the acting request cannot see,
the browser is redirected to the dashboard instead.

**Cancelling an unused session.** A session still in opening control with no order may be
cancelled; the session and its cash statement lines are deleted.

---

## 3. Loading the data the selling application needs

**Role**: system, at session opening and on demand.

The application asks for two things: the **parameters** (which fields and which relations
each entity exposes) and the **data** (the records themselves).

### 3.1 The entity list

The following entities are loaded, in this order, and each one may use the records already
loaded to compute its own selection condition:

the session; the configuration; presets; working-schedule attendances; orders; order
lines; lots and serial numbers on lines; payments; payment methods; printers; counter
categories; coin and banknote denominations; the company; product templates; product
variants; product attributes; taxes; tax groups; custom attribute values; template
attribute lines; template attribute values; template attribute exclusions; combos; combo
items; users; partners; product units; decimal precisions; units of measure; countries;
country states; languages; product categories; pricelists; pricelist rules; cash rounding
definitions; fiscal positions; the operation type; currencies; predefined notes; product
tags; installed capabilities; accounting entries; accounts; removal strategies.

Extension capabilities append to this list (restaurant floors, tables and courses;
employees; loyalty programmes, rules, rewards and cards; and so on).

### 3.2 The selection conditions

| Entity | Condition |
| --- | --- |
| Session | This session. |
| Configuration | This configuration. |
| Presets | The available presets plus the default preset. |
| Orders | Unfinished orders of this configuration. |
| Order lines | Lines of the loaded orders whose product is active. |
| Lots on lines | Lots of the loaded lines. |
| Payments | Payments of the loaded orders. |
| Payment methods | Every method, **active and archived alike**. |
| Printers | The printers of this configuration. |
| Counter categories | Everything when the configuration does not restrict categories; otherwise the configuration's available categories plus every category routed to a preparation printer. |
| Coin and banknote denominations | Those attached to this configuration, plus those attached to no configuration at all. |
| Company | The configuration's company. |
| Product templates | Templates of the configuration's company that are available at the counter and sellable; restricted to the configuration's available categories when the configuration restricts categories. Limited in number (see section 3.4). |
| Taxes | Every tax of the configuration's company. |
| Partners | A bounded selection (see section 3.4), plus the acting user's own partner, plus the customers of the loaded orders. |
| Fiscal positions | The configuration's fiscal positions, plus those of the loaded presets, plus those of the loaded partners. |
| Currencies | The company currency, the configuration currency and the currencies of the loaded pricelists. |
| Cash rounding | The configuration's rounding method. |
| Operation type | The configuration's operation type. |
| Predefined notes | The configuration's notes, or everything when it has none. |
| Users | Only the acting user. |
| Accounting entries | Nothing is loaded by the generic condition; entries reach the application only as part of an order's own data. |

### 3.3 Incremental loading

Every condition may be narrowed by a **last server date**. When the application supplies
one, the condition receives an extra clause requiring the record's last write instant to
be later than that date, so that only what changed since the last load is transmitted.
Three entities are exempt from this narrowing and are always transmitted in full: the
session, the configuration and the user.

The application also asks the server which of its locally cached record identifiers are no
longer relevant. The server answers, per entity, the union of the identifiers that no
longer exist and the identifiers of the records that are archived or that the acting user
may no longer read.

### 3.4 Bounded selections

Two entities are too numerous to load in full.

**Products.** The limit is the system parameter for the product load limit, defaulting to
five thousand. Before triggering a full synchronisation the application asks how many
products match the configuration's condition and what the limit is, and warns the operator
when the count exceeds the limit or crosses a dangerous threshold of twenty thousand.
Products not loaded are fetched on demand by barcode or by search.

**Partners.** The limit is the system parameter for the customer load limit, defaulting to
one hundred. The selection is ordered by the number of counter orders the partner has,
descending, then by name, and is paged by an offset. Additional partners are fetched on
demand: with an empty search condition the next page of the same ordered selection is
returned; with a non-empty search condition the whole partner set is searched, paged by
the offset, one hundred at a time. Every fetch also returns the fiscal positions of the
returned partners.

### 3.5 Prices in another currency

When a loaded record carries a price in a currency other than the configuration currency,
that price is converted at load time using today's rate. A product carries two prices in
two potentially different currencies (its sale price and its cost), and each is converted
from its own currency.

### 3.6 The change feed

The configuration owns a private notification channel addressed by its access token. The
server broadcasts on it:

| Notification | When |
| --- | --- |
| Synchronisation | After orders are transmitted, cancelled or removed; after a self-ordered transaction changes. Carries the changed record identifiers, the records themselves for the static entities, the session identifier and the identifier of the device that caused the change (so that device can ignore its own echo). |
| Closing session | When a session is written to the closed state. |
| Order state changed | When a self-ordered transaction changes state. |
| Payment status | When a self-order payment resolves, carrying the result and the order and line data. |
| Customer display update | Addressed to one display device, carrying the order to show. |

A configuration also relays its synchronisation notifications to every trusted
configuration, so that shared open orders stay in step.

---

## 4. Selling

**Role**: cashier. **Precondition**: an opened session.

1. The cashier selects products, by touching a button, by scanning a barcode, or by
   searching. Each selection creates or updates an order line in the browser.
   - A product of the combo kind opens the combo configurator; confirming it creates one
     header line and one line per chosen component, priced as described in
     [`calculations.md`](calculations.md) section 3.4.
   - A configurable product opens the attribute configurator; the chosen values become the
     line's selected attributes and their supplements are folded into the line's price
     extra.
   - A product to be weighed opens the scale screen and the weight becomes the quantity.
   - A lot-tracked or serial-tracked product opens the lot capture screen; the captured
     names become lot records on the line.
   - Lines of the same product, unit, price, discount and notes are merged when the
     product's unit is groupable.
2. The cashier may change the quantity, the unit price (only when price control is not
   restricted, or the cashier is an administrator), the discount percentage, the line note
   and the customer note.
3. The cashier may select a customer. Doing so copies the customer's pricelist onto the
   order and, in the interface, their fiscal position.
4. The cashier may change the pricelist, the fiscal position or the preset, each of which
   causes every line's price and taxes to be recomputed.
5. Totals are recomputed after every change, as described in
   [`calculations.md`](calculations.md) section 6.2.
6. When preparation printing is on, the cashier sends the order to the kitchen; the delta
   against the last printed state is computed and printed, and the last printed state is
   updated.
7. The cashier opens the payment screen.

Throughout, the order is saved to the browser's local storage after every change, and
transmitted to the server opportunistically. An order that cannot be transmitted stays in
local storage and is retried.

---

## 5. Taking payment

**Role**: cashier.

1. The payment screen proposes the remaining amount due for the first method touched; the
   proposal is rounded to the cash denomination when the method is a cash method and cash
   rounding is enabled, and falls back to the change when the remaining due is zero.
2. Touching a method creates a tender in the browser.
   - **Cash**: the cashier may type the amount, or touch one of the configured coin and
     banknote denominations.
   - **Bank without integration**: the amount is accepted as typed.
   - **Bank with a terminal**: the terminal flow of section 6 runs.
   - **Bank with a quick response code**: the code is generated for the amount and shown;
     the cashier confirms manually when the customer has paid.
   - **Customer account**: the order must carry a customer; the amount stays owed.
3. The cashier may add a second tender, remove a tender or change one. Changing a tender
   is refused when the order has already been printed at least once and the new status is
   anything other than cancelled.
4. When the tenders cover the total, the remaining due becomes zero and the change is
   computed. The cashier validates.
5. The browser stamps the order with its computed amounts, sets the order state to paid
   and transmits it.
6. The server processes the transmission (section 7).
7. The receipt is printed (automatically when the configuration asks for it), or the
   preview screen is shown.
8. When the customer asks for it, the receipt is sent by email, or as a text message when
   that capability is enabled.

### 5.1 Fast payment

When fast payment validation is enabled, the product screen shows one button per fast
payment method. Touching it creates a tender for the whole remaining amount with that
method and validates the order in one step, skipping the payment screen.

---

## 6. The payment terminal contract

**Role**: cashier and terminal.

A terminal integration must implement five operations. The selling application calls them
and reacts to their outcome; it never talks to the terminal hardware directly.

| Operation | Input | Expected outcome |
| --- | --- | --- |
| **Send payment request** | The tender (amount, currency, order reference, terminal-specific identifiers) | The tender's payment status moves to a waiting value. Eventually the terminal reports success, failure or cancellation. |
| **Send payment cancel** | The tender | The terminal abandons the request; the tender's status becomes cancelled or retry. |
| **Send payment reversal** | The tender | A completed tender is reversed on the terminal; the status becomes reversed. |
| **Set payment status** | A status value | Records the terminal's answer on the tender. |
| **Handle terminal answer** | The terminal's payload | Writes the card data onto the tender: card type, card brand, last four digits, cardholder name, payment reference number, approval code, issuer bank, payment mode, transaction identifier, and the text block to print on the receipt. |

Rules that hold for every integration:

- A tender in a waiting state blocks validation of the order.
- When the configuration validates terminal payments automatically, a success answer
  immediately validates the order.
- A failure leaves the tender in a retry state; the cashier may retry, cancel, or declare
  the tender successful without a terminal answer (the forced-success status).
- The terminal's answer is recorded on the tender even when the payment failed, so that
  the receipt can show the refusal.
- The terminal never creates accounting records; the tender is an ordinary bank tender and
  is treated as such at closing.

---

## 7. Transmitting an order to the server

**Role**: system, on every transmission.

The transmission carries a list of orders. Each is processed in turn.

1. Log the transmission with a random token so that two concurrent transmissions can be
   told apart in the log. When the order-data logging parameter is on, log the whole
   payload.
2. Collect the orders refunded by the lines of this order. More than one is refused with
   *"You can only refund products from the same order."* Exactly one is remembered so that
   it is returned to the application with fresh data.
3. Look for an existing order with the same universally unique identifier. With the
   restaurant capability on, an unfinished order for the same table on the same
   configuration also matches.
4. **No existing order** — create it (step 6).
   **Existing and unfinished** — update it (step 7).
   **Existing and not unfinished** — do nothing and return the existing order. This
   happens legitimately when a tip is added later.
5. Resolve the session: when the named session is in closing control or closed, find the
   open session of the same configuration and re-home the order to it; when there is none,
   refuse with *"No open session available. Please open a new session to capture the
   order."*
6. **Creation.** Drop a customer identifier that no longer exists, and with it the
   to-invoice flag. Default the company from the configuration. When the order being
   transmitted is the one currently in progress (its identifier matches the one named in
   the context), replace its date with the server clock. Create the order, which assigns
   the receipt number, the tracking number and the session-unique sequence number and
   defaults the pricelist, the fiscal position and the preset.
7. **Update.** Move the order to the named session when it changed. Write the lines and
   the tenders first, converting a create command whose universally unique identifier
   already exists on the order into an update of that line — this is what makes the
   protocol idempotent. Log the identifiers actually added. Then write the remaining
   fields, without the universally unique identifier and without the access token, keeping
   the existing state when the transmitted state is paid (the paid state is assigned
   later, deliberately).
8. Resolve any cross-references expressed as universally unique identifiers into real
   identifiers.
9. Recompute the paid amount from the tenders and write it back. When the order is not
   unfinished and the change is not zero, create the change tender: a payment named
   "return", with the change amount, dated now, on the first cash method of the session,
   flagged as change. Without a cash method this fails with *"No cash statement found for
   this session. Unable to record returned cash."* Then recompute the amounts.
10. When the order is not unfinished and not cancelled: run the paid check (section 7.1),
    create the delivery document (section 8) and compute the line costs where possible.
11. When the order asks to be invoiced, is paid and the configuration has an invoice
    journal, generate the invoice (section 10). When the configuration has no invoice
    journal, refuse with *"No invoice journal configured for this POS session."*
12. Ensure the order has a portal access token, and broadcast a synchronisation
    notification on the configuration's change feed unless the transmission is a
    preparation-only one.
13. Return the orders together with their lines, lots, custom attribute values, tenders
    and related accounting entries.

### 7.1 The paid check

Computed as described in [`state-machines.md`](state-machines.md) section 2.3. On failure
the message is *"Order <order name> is not fully paid."* On success the order state becomes
paid, which assigns its name.

A separate guard runs on every write that touches the tenders: when the paid amount is
**less** than the total and the order is paid or posted, the write is refused with *"The
paid amount is different from the total amount of the order."* When the paid amount is
**greater** than the total and the order is paid, a warning line is appended to the
payment-changes message: *"Warning, the paid amount is higher than the total amount.
(Difference: <amount>)"*.

### 7.2 The payment-changes message

Every write that touches the tenders composes a message listing the changes, and posts it
on the order thread under the heading `Payment changes:` as a bulleted list:

| Change | Line |
| --- | --- |
| A tender added | `Added <method> with <amount>` |
| A tender's method and amount changed | `<old method> changed to <new method> and from <old amount> to <new amount>` |
| Only the method changed | `<old method> changed to <new method> for <old amount>` |
| Only the amount changed | `Amount for <method> changed from <old amount> to <new amount>` |
| A tender removed | `Removed <method> with <amount>` |

---

## 8. Creating the delivery document

**Role**: system, when an order becomes paid (real-time mode) or at session closing
(deferred mode).

### 8.1 Deciding when

```
create_now ⟺ the session does not defer stock updates
             or the company uses cost-at-invoicing accounting and the order asks to be invoiced
             or the order refunds an order that had a shipping date
```

### 8.2 Ship later

When the order carries a shipping date:

- for a refund of a ship-later order, the ordinary transfer path of section 8.3 is used,
  because it knows how to cancel or reduce the original transfer;
- otherwise the procurement rules are launched. A stock reference named after the order is
  created when the order has none, and every line of a storable or consumable product
  raises a procurement for its quantity, to the customer location of the order's partner,
  with the shipping date converted from the acting time zone to universal time as both the
  planned date and the deadline, carrying the configuration's deferred route, the
  configuration's warehouse, the partner and the reference. Then the resulting transfers
  are confirmed, and for every tracked product the move lines are rebuilt from the order's
  lots without being marked done.

### 8.3 The ordinary transfer path

1. Take the destination location: the customer location of the order's partner when it has
   one, otherwise the default destination of the operation type (and, at session closing,
   the partner location of the warehouse when the operation type has none).
2. Keep only the lines of consumable or storable products whose quantity is not zero at the
   product unit's precision.
3. Split them into positive lines and negative lines.
4. **Positive lines.** Create a transfer from the operation type's source location to the
   destination, with direct move type and no responsible user. Create one move per
   (product, set of selected attribute values) group, with the summed absolute quantity,
   the product's reference unit, the transfer's locations and company, the non-variant
   attribute values, and, when the group refunds a completed outgoing move of the same
   product, a link to that move as the returned origin. Confirm the moves, build their move
   lines from the order lines (marking the quantities done), mark them picked, then attempt
   to complete the transfer. A refusal — insufficient stock, a missing lot — is swallowed
   and the transfer is left incomplete.
5. **Negative lines.** When they all refund lines of one single order:
   - if every refundable line of that order is now fully refunded, and the original order
     still has transfers that are neither completed nor cancelled, and none of its transfers
     is completed, cancel those transfers and stop;
   - if the refund is partial under the same conditions, reduce the demanded quantity of the
     matching moves by the refunded quantity, deleting a move whose remaining quantity
     reaches zero and re-reserving the others, then stop.
6. Otherwise create a return transfer: the return operation type of the configuration's
   operation type when it has one (destination = that type's default destination), falling
   back to the same operation type with the source and destination swapped. Create its
   moves the same way and attempt to complete it.
7. Attach the created transfers to the session and, for a per-order transfer, to the order,
   setting their origin to the order name.

### 8.4 Lot handling on the moves

For each move whose product is tracked and whose operation type uses lots:

1. Search for existing lots of the company (or of no company) matching the product and the
   name captured on the line, when the operation type uses existing lots.
2. Create the missing ones, when the operation type creates lots.
3. For each captured lot, the quantity assigned is one for a serial-tracked product and the
   absolute line quantity for a lot-tracked one.
4. When an existing lot was matched, the quantity is spread over the stock quantities
   holding that lot in the source location, newest first, and any remainder becomes a move
   line naming the lot directly.
5. For every other move (untracked products, or products whose operation type neither uses
   nor creates lots) the done quantity is simply set to the demanded quantity. Before doing
   so, any unit conversion that would round the demanded quantity down to zero is detected
   and refused with a message beginning *"Conversion Error: The following unit of measure
   conversions result in a zero quantity due to rounding:"*, listing each offending pair as
   *" - From "<source unit>" to "<target unit>""* and ending with an explanation of how to
   fix it.

### 8.5 Owner propagation on a return

When the returned goods came from a transfer that recorded an owner, the owner is copied
onto the return's move lines, quantity by quantity, until the returnable quantity per
(product, owner) pair is exhausted.

### 8.6 Notifications

Counter deliveries never send the ordinary transfer confirmation email or text message:
transfers whose operation type is the warehouse's point-of-sale type are excluded from
that mechanism.

---

## 9. Refunding

**Role**: cashier.

### 9.1 From the selling application

1. The cashier opens the list of past orders and selects the order to refund. The list
   covers the orders of this configuration and of its trusted configurations, in the same
   currency, whose state is not unfinished and not cancelled — or which are refunds that
   are not unfinished.
2. The cashier selects the lines and the quantities to return. Each selection creates a
   refund line in a new order in the current session, with a negative quantity, pointing at
   the original line.
3. The guard of [`calculations.md`](calculations.md) section 10.3 prevents returning more
   than the outstanding quantity.
4. The cashier validates and hands the money back; the refund order is transmitted like any
   other.

### 9.2 From the administrative interface

1. The user opens a paid order and asks to return products.
2. For each order, the system finds the current session of the order's configuration. When
   there is none, it refuses with *"To return product(s), you need to open a session in
   the POS <configuration display name>"*.
3. A copy of the order is created in that session with: the name being the original name
   followed by ` REFUND`, a fresh receipt number and tracking number, the current instant
   as its date, no lines, a paid amount of zero, the cost flagged as not computed, the
   refund flag set.
4. For every line of the original order whose already-refunded quantity is below its
   quantity, a refund line is copied with quantity `−( quantity − already refunded )`, a
   copy of each lot, the cost flagged as not computed and a link to the original line.
5. The amounts of the refund order are recomputed and a synchronisation notification is
   broadcast.
6. The user is taken to the refund order to record the repayment.

### 9.3 What the refund does downstream

- The original line's already-refunded quantity rises, and the original order's
  refund-orders count rises.
- The refund order contributes a refund bucket to the closing entry
  ([`accounting-effects.md`](accounting-effects.md) section 12).
- The delivery path of section 8.3 either cancels, reduces or reverses the original
  transfer.
- When the original order was invoiced, invoicing the refund produces a credit note linked
  as the reversal of the original invoice, and the original invoice is listed among the
  refunded invoices.

---

## 10. Invoicing

### 10.1 At the counter

**Role**: cashier.

1. The cashier selects a customer on the order — mandatory, since an invoice needs one —
   and turns on the invoice request.
2. On transmission, once the order is paid, the invoice is generated.

### 10.2 Generating the invoice

**Role**: system.

1. Take an exclusive lock on the orders. When another invoicing attempt holds it, refuse
   with *"Some orders are already being invoiced. Please try again later."*
2. Set the order state to posted.
3. Build the invoice values (see [`accounting-effects.md`](accounting-effects.md)
   section 6) and create the invoice with elevated rights in the order's company.
4. Adjust the invoice's cash rounding line so that the invoice total equals what was paid.
5. Post a message on the invoice linking back to the orders.
6. Post the invoice.
7. For each session of the group, note whether it is already closed. For each order,
   create the invoice payment entries for its tenders. Remember the entries of the orders
   whose session was already closed.
8. Reconcile the invoice against the payment entries.
9. For each order whose session was already closed, create the reversal entry that removes
   it from the closing entry, and reconcile the result.
10. Unless generation is switched off in the acting context, render and send the invoice
    document.

### 10.3 Invoicing later from the administrative interface

**Role**: cashier or accountant.

1. Select one or more paid orders and ask to create invoices.
2. The wizard asks for the customer (defaulted from the first selected order) and whether
   to produce one consolidated invoice or one invoice per order.
3. With consolidated billing, one invoice covers the whole selection; without it, each
   order is invoiced separately.
4. The procedure of section 10.2 then runs.

### 10.4 The customer claiming an invoice from the receipt

**Role**: customer.

1. The receipt carries a link to the invoice request page — as a quick response code, as a
   printed address, or both, according to the company setting — and, when the company asks
   for it, a five-character code.
2. The customer opens the invoice request page. Reaching it without a token shows a form
   asking for the receipt number, the order date and the code.
3. Submitting the form validates that all three fields are present, that the receipt number
   is at least twelve characters long (otherwise *"The Ticket Number should be at least 12
   characters long."*) and, when anything is missing, reports *"Please fill all the
   required fields."*
4. The order is looked up by a receipt number ending with the supplied value, an order date
   within one day before and two days after the supplied date, and an exact code match.
   When nothing is found: *"No sale order found."*
5. On success the customer is redirected to the validation page carrying the order's access
   token.
6. The validation page:
   - returns not-found when there is no token or no matching order;
   - redirects straight to the existing invoice when the order already has one;
   - stops when another process holds the invoicing lock;
   - asks for the billing address, plus any extra partner fields and invoice fields the
     company's fiscal country requires;
   - for a signed-in visitor whose own details are already complete, invoices immediately;
   - for an order that already carries a customer and an unauthenticated visitor, invoices
     immediately with that customer;
   - otherwise renders the address form, pre-filled from the visitor's partner where
     possible, defaulting the country to the company's fiscal country.
7. Submitting the form creates or updates the address, validates the extra fields
   (*"The field <field label> must be filled."* for each missing one) and, when everything
   is valid, sets the customer on the order, invoices it and redirects to the invoice.
   Otherwise it raises a message of the form *"Message: <messages>\nPlease update the
   <invalid fields>"*.

---

## 11. Tipping

**Role**: cashier.

Two mechanisms exist.

**Tip as a product.** The tip product is added as an ordinary line with the tip amount as
its price. It is sold, taxed and posted like any other line.

**Tip after payment (restaurant).** When the configuration allows adjusting the terminal
authorisation after the customer has left, the order is validated for the goods only, and
the tip is recorded later: the order's tip amount is set, the already-tipped flag is
raised, and the tender authorised on the terminal is increased. Because the order is
already paid, the transmission takes the "existing and not unfinished" branch and simply
returns the order; the tip is applied by a dedicated operation rather than by the ordinary
order write.

---

## 12. Cash movements during the session

**Role**: cashier for a cash in or out, counter administrator or accountant for both; a
user with basic accounting rights or a counter administrator to delete one.

1. The cashier chooses cash in or cash out, types an amount and a reason, and confirms.
2. The permission is checked: the acting user must be a counter administrator or hold
   invoice-level accounting rights, otherwise *"You don't have the access rights to
   perform a cash in/out."*
3. The session must have a cash journal, otherwise *"There is no cash payment method for
   this PoS Session"*.
4. A statement line is created as described in
   [`accounting-effects.md`](accounting-effects.md) section 5.
5. Deleting one requires a counter administrator or basic accounting rights, otherwise
   *"You don't have the access rights to delete a cash in/out."* The line must belong to
   this session, otherwise *"You cannot delete a cash move that is not linked to this
   session."* The deletion posts `Cash move deleted: <cashier name>: <amount>` in the
   session thread.

---

## 13. Closing a session

**Role**: cashier, escalating to counter administrator when the difference exceeds the
authorised limit.

### 13.1 From the selling application

1. The cashier asks to close. The application first checks that no order due now or earlier
   is unfinished; the server repeats the check and answers, when it fails, *"You cannot
   close the POS while there are still draft orders for the day."* with a flag telling the
   application not to redirect.
2. The server reports whether the session was already closed by somebody else. If it was,
   the answer is an alert titled "Session already closed" reading *"The session has been
   already closed by another User. All sales completed in the meantime have been saved in
   a Rescue Session, which can be reviewed anytime and posted to Accounting from Point of
   Sale's dashboard."* with a redirect to the administrative interface.
3. The server validates any per-method bank differences supplied: each concerned journal
   must have the account matching the sign of its difference. The failure message is
   *"Need loss account for the following journals to post the lost amount: <journals>"*
   and/or *"Need profit account for the following journals to post the gained amount:
   <journals>"*.
4. The application shows the closing control screen, built from the closing control data:
   the number and total of the closed orders, the opening notes, the expected cash
   (starting balance plus cash tenders plus cash movements) with the movement list, one row
   per non-cash method with its total and its count, whether the acting user is an
   administrator, and the authorised difference when the maximum-difference flag is set.
5. The cashier counts the drawer and enters the amount. The server stores it as the counted
   ending balance, after repeating the not-closed and no-draft-orders checks and verifying
   that the session has a cash register, otherwise *"There is no cash register in this
   session."*
6. The application computes the difference and, when the maximum-difference flag is set and
   the acting user is not an administrator, refuses a difference beyond the limit and asks
   for an administrator.
7. Orders scheduled for a future time are detached from the session (their session link is
   emptied) so that they survive into the next session.
8. The closing control transition and then the validation run (section 13.3).
9. When validation returns a wizard instead of completing, the application reports failure
   with the wizard's title as the message and redirects to the administrative interface.
10. When the acting user has an email address, `Closed Register` is posted in the session
    thread.

### 13.2 From the administrative interface

1. The user opens the session and asks for the closing control, then for validation.
2. When the configuration has no cash control, the closing control transition goes straight
   to validation.
3. The user may supply a balancing account and an amount when the first attempt refused.

### 13.3 Validation

The full guard list and the full side-effect list are in
[`state-machines.md`](state-machines.md) sections 1.3 and 1.4; the resulting accounting
document is specified line by line in [`accounting-effects.md`](accounting-effects.md)
section 3. In outline:

1. Freeze the cash transaction total and capture the cash difference.
2. Create the deferred delivery documents and compute the outstanding line costs, in
   deferred mode.
3. Build the closing entry.
4. Verify that it balances; on failure roll everything back and offer the forced close.
5. Post the cash difference statement line.
6. Post the closing entry and move the paid orders to posted; or delete the entry when it
   has no line.
7. Execute the reconciliation plan.
8. Post the edited-orders message when edit tracking is on.
9. Trigger the replenishment scheduler on the moves of the session's transfers.
10. Mark the session closed and broadcast the closing notification.

### 13.4 Forced close

**Role**: counter administrator, with an accountant choosing the account when the
administrator may not read accounting data.

1. The wizard opens carrying the unbalanced amount, the default balancing account and the
   explanation *"There is a difference between the amounts to post and the amounts of the
   orders, it is probably caused by taxes or accounting configurations changes."*
2. The user picks an account — or cannot, when they lack accounting read rights, in which
   case the default is used.
3. Confirming re-runs the validation with the account and the amount, and one balancing
   line named `Difference at closing PoS session` is added at the end.

---

## 14. The rescue session

**Role**: system, then counter administrator.

1. An order reaches the server naming a session that is in closing control or closed.
2. The server logs a warning naming the closed session, its identifier, the order's
   universally unique identifier and its total.
3. The server looks for an opened session of the same configuration. If there is one, the
   order is re-homed to it and the warning names the session used.
4. If there is none, the transmission is refused with *"No open session available. Please
   open a new session to capture the order."*
5. A rescue session — a session whose recovery flag is set — may be created to receive such
   orders. It coexists with a normal session, is counted in the configuration's rescue
   session counter, and is shown on the dashboard.
6. A rescue session cannot be closed from the selling application. An administrator opens
   it from the dashboard and validates it. Its counted ending balance is computed
   automatically as the starting balance plus its cash tenders, so it closes with no cash
   difference.

---

## 15. Restaurant service

**Role**: waiter (a cashier), with the restaurant capability installed.

### 15.1 Seating and ordering

1. The waiter opens the floor plan (the default screen, unless the configuration defaults
   to the register) and touches a table.
2. Touching an empty table starts a new order bound to that table; touching an occupied
   table opens its unfinished order.
3. The waiter records the number of guests.
4. Lines are added as in section 4. Lines may be grouped into courses; each course is a
   separate record with an index and a fired flag.
5. Firing a course sends its lines to the preparation printers and stamps the course's
   fired instant.

### 15.2 Transferring an order

1. The waiter selects the order and touches the destination table.
2. The order's table link is rewritten. When the order came from self-ordering, its
   self-ordering table link is realigned to the new table at the same time.
3. Tables pushed together are modelled by setting one table's parent to another. Setting a
   parent that would create a cycle is silently reverted.

### 15.3 Printing the bill before payment

When bill printing is enabled, the waiter may print a bill at any time. Printing the bill
does not change the order state and does not increment the receipt print counter used by
the payment-change guard.

### 15.4 Splitting the bill

**Role**: waiter. **Precondition**: bill splitting enabled.

Two mechanisms exist, and they have different accounting consequences.

**Split the payment.** The order stays whole; the waiter records two or more tenders on it,
one per payer. The closing entry sees one sale and two receivable contributions
([`accounting-effects.md`](accounting-effects.md) section 16.2).

**Split the order.** The waiter moves a subset of the lines (or a fractional quantity of a
line) onto a new order. Two independent orders result, each with its own receipt number,
its own tax computation and its own tenders
([`accounting-effects.md`](accounting-effects.md) section 16.3).

### 15.5 Closing a table

An unfinished order on a table blocks deactivating the table (*"You cannot delete a table
when orders are still in draft for this table."*) and deactivating its floor (*"You cannot
delete a floor when orders are still in draft for this floor."*).

---

## 16. Self-ordering

### 16.1 Modes

| Mode | Who orders | Payment timing |
| --- | --- | --- |
| Disabled | — | — |
| Browse only | The customer reads the menu on their own device | No ordering |
| Mobile ordering | The customer orders from their own device | Each order (forced) unless the service mode is table service with the restaurant capability, in which case meal-end payment is possible |
| Kiosk | The customer orders on a shared device at the counter | Each order (forced); cash methods are forbidden |

### 16.2 Reaching the self-ordering pages

1. The configuration publishes a shortened address. For table service with the restaurant
   capability, one address per active table is published, each carrying the table's
   identifier; otherwise a generic address is published, six times over, so that a sheet of
   codes can be printed.
2. The address carries the configuration's access token, and for a table the table
   identifier. Rotating the access token also regenerates every table identifier, which
   invalidates every printed code.
3. When no session is open, the pages are served with the access rights of the
   configuration's default user, which must be a counter user or administrator.

### 16.3 Placing a self order

1. The customer browses the categories and products. A category is only offered between its
   availability-after hour and its availability-until hour.
2. The customer chooses a preset when the configuration uses presets. The preset must exist
   (*"Invalid preset"*), must be available in self-ordering or be the configuration default
   (*"Preset is not available in self-ordering"*) and must be among the configuration's
   available presets (*"Preset is not available in this configuration"*).
3. The customer submits. Every line of the payload is validated before it is written:
   - a delete or unlink command is honoured only for a line that belongs to the order being
     submitted;
   - an update command is honoured only for a line that belongs to that order — otherwise
     the command is dropped, so that an arbitrary line cannot be re-parented;
   - a quantity must be a finite number strictly greater than zero, otherwise *"Invalid
     quantity"*; returns are not possible through the public route;
   - every attribute value must exist and must belong to the ordered product's template,
     otherwise *"Invalid product attribute"*;
   - the taxes are recomputed server-side from the product and the fiscal position, and the
     line amounts are set to zero and recomputed server-side, so the payload cannot dictate
     a price.
4. The order is created with a receipt number prefixed for the device: the letter `K`, the
   configuration identifier and a hyphen for a kiosk, the letter `S` for a mobile order.
   A kiosk order is named after the table stand number when one was taken, and after the
   tracking number otherwise.
5. Pay-each: a payment flow starts (section 16.4). Pay-at-meal-end: the order is held
   unfinished on the table and sent to preparation.
6. An order-state-changed notification and a synchronisation notification are broadcast.

### 16.4 Paying a self order online

1. A payment transaction is created for the order total in the selling currency through the
   payment providers domain.
2. The customer is redirected to, or shown inline, the provider's form.
3. On the provider's callback the transaction is confirmed; it creates a counter payment on
   the order with the online payment method and the order becomes paid.
4. The result is broadcast on the change feed as a payment-status notification carrying the
   result and the order and line data; a success also triggers the receipt processing and
   the send-to-preparation hook.
5. A failure leaves the order unfinished; removing it from the interface sets it to
   cancelled and broadcasts the notifications.

### 16.5 Paying a self order at the counter

The order appears among the unfinished orders of the session. The cashier opens it and
takes payment as in section 5.

---

## 17. Employee login

**Role**: cashier, with the employee capability installed.

1. The configuration lists the employees allowed to use the till.
2. On opening the selling application, a login screen asks for the employee: by touching a
   name and typing a personal identification number, or by scanning a badge.
3. The chosen employee is recorded on every order created and on every tender taken, and
   on every cash movement.
4. Cash-in and cash-out permissions are evaluated against the employee rather than the
   platform user.
5. The cashier may lock the screen, which returns to the login screen without closing the
   session.

---

## 18. Settling a sales order at the counter

**Role**: cashier, with the sales-order capability installed.

1. The cashier searches the sales orders that may be settled and selects one.
2. The system offers three treatments: settle the whole order, settle a down payment, or
   add the remaining lines to the current counter order.
3. Settling the whole order copies its lines onto the counter order, each line keeping a
   link back to its sales order line so that the delivered and invoiced quantities advance.
4. Settling a down payment adds one line carrying the down payment product for the chosen
   amount or percentage.
5. The counter order is then paid as usual. The delivery documents it creates are attached
   to the sales order too.

---

## 19. Applying loyalty at the counter

**Role**: cashier, with the loyalty capability installed.

1. Loyalty programmes, their rules, their rewards and the customer's cards are loaded into
   the selling application with the rest of the master data.
2. When a customer is selected, or a code is typed, or a coupon barcode is scanned, the
   applicable programmes are re-evaluated over the current order.
3. Claiming a reward adds one or more ordinary order lines carrying the reward's product
   with a negative amount, marked with the reward and the coupon consumed. Their price type
   is the automatic one.
4. Every change to the order re-evaluates the programmes, so a reward that no longer
   applies is removed and a newly earned one is offered.
5. On transmission, the point balances are updated and the coupons are marked as used.
6. The reward lines are aggregated into the closing entry exactly like any other line
   ([`accounting-effects.md`](accounting-effects.md) section 15).

---

## 20. The global discount button

**Role**: cashier, with the discount capability installed.

1. The cashier touches the discount button and types a percentage.
2. One line is added carrying the configured discount product, with a negative amount equal
   to the percentage of the current order total, and with the taxes needed so that the
   discount is spread over the tax groups already present.
3. The line is an ordinary line and is aggregated like any other.

---

## 21. Printing the daily report

**Role**: cashier or counter administrator.

- **From the selling application**, at closing: the document is rendered for the session
  just closed, optionally including the opening and closing notes.
- **From the administrative interface**, for a session: the same document for that session.
- **From the administrative interface**, for a period: the wizard asks for a start date, an
  end date and a set of configurations, all required; moving the start date past the end
  date drags the end date along and vice versa. The document is rendered for that range.
- **Through a direct address**: the document is rendered for a start and stop instant passed
  as parameters, for a signed-in user.

The content is specified in [`calculations.md`](calculations.md) section 14.

---

## 22. Working offline

**Role**: cashier and system.

1. A service worker is registered for the selling application and serves the cached
   application resources. The list of addresses to cache is the asset links of the
   application bundle plus the two entry addresses of the configuration.
2. Master data and orders are held in a local database in the browser.
3. Every change to an order is written to local storage immediately.
4. Transmission is attempted opportunistically. A failure leaves the order in local storage
   and schedules a retry.
5. The application periodically checks connectivity with a lightweight round trip that
   answers a fixed token.
6. When connectivity returns, every pending order is transmitted. The idempotency key makes
   a duplicate transmission harmless: an order already present and not unfinished is
   returned unchanged; an order already present and unfinished is updated, and a line whose
   universally unique identifier already exists is updated rather than duplicated.
7. The application compares the configuration's last-data-change instant with its own
   cached copy and reloads the master data when they differ.
8. While offline the application cannot: look up a customer that was not loaded, look up a
   product that was not loaded, generate a quick response code for an amount (it falls back
   to the amount-less code cached at loading), reach a payment terminal that needs the
   server, or close the session.

---

## 23. Barcode behaviors

**Role**: cashier.

Scanned codes are interpreted through the company's barcode nomenclature, falling back to
the configuration's fallback nomenclature. Five rule kinds are contributed by this domain,
in addition to the generic product and lot rules:

| Rule kind | Label | Effect |
| --- | --- | --- |
| `weight` | Weighted Product | The code carries a product reference and a weight; the product is added with that weight as its quantity. |
| `price` | Priced Product | The code carries a product reference and a price; the product is added with that price as its unit price and the price type set to manual. |
| `discount` | Discounted Product | The code carries a discount percentage; it is applied to the selected line. |
| `client` | Client | The code identifies a partner; that partner becomes the order's customer. |
| `cashier` | Cashier | The code identifies the operator; used by the employee login. |

A code matching no rule is tried as a product barcode, then as a product-unit barcode, then
as a lot name, and finally reported as unknown.

---

## 24. The customer display

**Role**: system and customer.

1. The configuration exposes a display page addressed by the configuration identifier and a
   device identifier, protected by the configuration's access token compared in constant
   time. A wrong or missing token returns not-found.
2. The page is rendered for an unauthenticated visitor, with the language of the acting
   user falling back to the language of the company's partner, and with the configuration's
   display data: the configuration identifier, the access token, whether a background image
   is set, the company identifier, and the proxy address.
3. The selling application pushes the current order to the display over the change feed,
   addressed to that device identifier.

---

## 25. Housekeeping

**Role**: system.

### 25.1 The stale session reminder

A scheduled task, chained to the replenishment scheduler, looks for sessions whose opening
instant is more than seven days in the past and whose state is not closed. For each such
session that has no activity yet, it schedules an activity of the old-session kind for the
session's opening user, with the note *"Your PoS Session is open since <opening instant>,
we advise you to close it and to create a new one."*

### 25.2 Lock-date protection

Changing a company's fiscal-year, tax, sale or hard lock date is refused while a session
that would violate it is still open:

> Please close all the point of sale sessions in this period before closing it. Open
> sessions are: *<session names>*

The sessions considered are the non-closed sessions of the company or of a descendant
company whose opening instant is at or before the greater of the fiscal-year and hard lock
dates, or at or before the tax lock date, or — when the configuration's point of sale
journal is a sale journal — at or before the sale lock date.

### 25.3 Tax protection

A tax may not have its kind, amount, scope, tax group, price-included behavior,
base-inclusion behavior or base-affected behavior changed while any counter order line
carrying it belongs to a session that is not closed:

> It is forbidden to modify a tax used in a POS order not posted. You must close the POS
> sessions before modifying the tax.

### 25.4 Journal protection

- A journal attached to a counter payment method may not change its type: *"This journal is
  associated with a payment method. You cannot modify its type"*.
- Such a journal may not be archived or deleted: *"You can not archive this journal because
  it is set on the following payment method : <method name>."*
- When the chart of accounts is swapped, the payment methods of the deleted journals and the
  configurations using them are deleted with them.

### 25.5 Sequence protection

A numbering sequence used by an active configuration may not be deleted: *"You cannot
delete a sequence used in an active POS config: <sequence names>"*.

### 25.6 Rounding definition protection

- A cash rounding definition used by a configuration may not be deleted: *"You cannot delete
  a rounding method that is used in a Point of Sale configuration."*
- Its rounding step, method or strategy may not be changed while a session using it is open:
  *"You are not allowed to change the cash rounding configuration while a pos session using
  it is already opened."*
