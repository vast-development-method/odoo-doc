# Point of Sale — Glossary

Every term used in this domain, defined in full. Storage names are given in code font
where a term corresponds to a stored field or entity.

---

## A

**Access token (of a configuration)** — a sixteen-character value (`access_token`) that
addresses the configuration's private notification channel and authenticates the customer
display and the self-ordering pages. Rotating it invalidates every previously printed
self-ordering code.

**Access token (of an order)** — a value that lets an unauthenticated customer open the
order's portal page and claim an invoice.

**Accounting partner** — the commercial entity behind a contact: the topmost company in
the contact's parent chain, or the contact itself when it stands alone. Receivable
balances are always carried by the accounting partner, never by a child contact.

**Accounting payment** — a record of the payments domain representing money moving
through a bank or cash journal. At session closing one is created per aggregated bank
payment method and one per identified-customer bank tender.

**Activity** — a scheduled reminder attached to a record. This domain schedules one kind:
a reminder to close a session that has been open for more than seven days.

**Add-a-rounding-line strategy** — the cash rounding strategy that puts the rounding
difference on a line of its own, rather than absorbing it into the largest tax line. It is
the only strategy a counter configuration may use.

**Aggregated tender** — a tender whose payment method does not identify the customer, so
that every such tender of the session is summed into one bucket keyed by the payment
method.

**Amount in currency** — the amount of a journal item expressed in a currency other than
the company currency. Present on a journal item whenever the selling currency differs from
the company currency, except on the cost and valuation lines, which are always in company
currency.

**Asymmetric rounding** — rounding that inverts the rounding method for negative values,
so that a negative amount is rounded toward the same party as the equivalent positive
amount would be. Used for the remaining amount due and for the change.

**Authorised difference** — the largest cash difference (`amount_authorized_diff`) a
non-administrator may post when closing, when the maximum-difference flag is set.

---

## B

**Backend order sequence** — the per-configuration numbering sequence
(`order_backend_seq_id`) that supplies the running number inside the receipt number and,
reduced modulo one thousand, the tracking number.

**Balancing line** — the extra journal item, named `Difference at closing PoS session`,
added when the operator forces a close with an account and an amount.

**Bank kind** — the payment method kind derived from a bank journal. A bank tender
produces an accounting payment through the method's outstanding account at closing.

**Base line** — the intermediate representation of an order line handed to the tax engine:
the price, the quantity, the taxes after fiscal position mapping, the account, the
partner, the currency, the rate, the discount and the sign.

**Base tag** — a tax tag attached to the base (untaxed) part of a taxed line. Base tags
are part of the sales aggregation key of the closing entry.

**Basic receipt** — a second, price-free receipt, suitable as a gift receipt, printed when
the corresponding configuration flag is set.

**Bill (coin or banknote)** — see *Denomination*.

**Bucket** — a running pair of totals (an amount in the selling currency and an amount in
the company currency) accumulated under an aggregation key while the closing entry is
built.

---

## C

**Card tender** — an informal name for a bank tender taken on a payment terminal.

**Cash control** — the behavior, active when the configuration offers at least one cash
payment method, of asking for a cash count at opening and at closing.

**Cash difference** — the counted ending balance minus the theoretical closing balance. A
negative difference is a loss, a positive one a profit; either is posted as a bank
statement line in the cash journal.

**Cash journal (of a session)** — the journal (`cash_journal_id`) of the first cash-kind
payment method of the configuration. Exactly one cash register is supported per session.

**Cash kind** — the payment method kind derived from a cash journal. A cash tender
produces a bank statement line in that journal at closing.

**Cash rounding** — rounding the payable total of an order to a cash denomination, because
the smallest coin in circulation is larger than the currency's smallest representable
increment.

**Change** — the money handed back when the tenders exceed the payable amount. Recorded as
a negative cash tender with the change flag set, so that the paid amount always equals the
net cash kept.

**Closed order** — an order of a session whose state is neither unfinished nor cancelled.
The closing entry is built from the closed orders.

**Closing control** — the session state in which trading has stopped and the cash count is
being taken.

**Closing entry** — the single balanced accounting document produced when a session is
validated, summarising every uninvoiced sale, its taxes, its tenders, its rounding and its
cost of goods sold.

**Combo** — a product sold as a bundle of chosen components. Its list price is
distributed over the chosen components so that they sum exactly to it.

**Combo header line** — the order line carrying the combo product itself. It carries no
value of its own: its margin is always zero and its invoice line is a section line.

**Commercial entity** — see *Accounting partner*.

**Company currency** — the currency of the company that owns the configuration. Every
journal item's balance is expressed in it.

**Configuration** — see *Point of Sale Configuration*.

**Consolidated billing** — producing one invoice for a group of orders sharing a
configuration, a customer, an employee and a fiscal position, rather than one invoice per
order.

**Contribution date** — the date used to convert one contribution into company currency
when it is added to a bucket. It is the payment date for tenders, the order date for
sales, taxes, rounding and the invoiced-order counterweight, and the transfer completion
date for cost and valuation.

**Cost of goods sold** — the value of the goods handed over, recognised as an expense.
Written at session closing as a debit on the expense account against a credit on the stock
valuation account.

**Counted ending balance** — the cash actually counted in the drawer at closing
(`cash_register_balance_end_real`).

**Counter administrator** — a user in the point of sale administrator group.

**Course** — a grouping of restaurant order lines that must reach the kitchen together.

**Customer account kind** — see *Pay-later kind*.

**Customer display** — a second screen facing the customer, showing the order as it is
composed. Addressed by the configuration identifier, a device identifier and the
configuration's access token.

---

## D

**Deferred stock update** — the company setting under which delivery documents are created
once, at session closing, rather than per order at sale time.

**Denomination** — a coin or banknote value (`pos.bill`) offered as a one-touch button on
the payment screen.

**Device identifier** — the value a device obtains from the configuration's device
sequence when it registers. It appears inside every receipt number produced by that
device and is echoed in every change-feed message so that a device can ignore its own
echo.

**Discount amount (of a line)** — the difference between what the line would have cost at
its undiscounted price and what it actually costs, both tax-inclusive, with the line's
sign reapplied.

**Draft order** — see *Unfinished order*.

---

## E

**Edit tracking** — the configuration option that records quantity reductions and line
deletions in the order thread and lists the edited orders in the session thread at
closing.

**Extra tax data** — a per-line structured document carrying custom values for the tax
engine, such as an externally determined tax amount.

---

## F

**Fast payment** — the configuration option that shows one-touch validate buttons for
chosen payment methods on the product screen, skipping the payment screen.

**Fiscal position** — a rule set that maps taxes and accounts, applied to an order as a
whole. It is defaulted from the configuration, may be changed by the cashier, and may be
carried by a preset or by the customer.

**Floor** — a named seating area of a restaurant, holding tables.

**Forced close** — closing a session whose closing entry does not balance, by nominating
an account and an amount for a balancing line.

**Full product name** — the rendered description of the product including the chosen
attribute values, stored on the line and used verbatim on the receipt and the invoice.

---

## G

**Groupable unit** — a unit of measure marked so that lines of the same product in that
unit are merged in the cart.

---

## I

**Identify-customer method** — a payment method whose identify-customer flag
(`split_transactions`) is set. It forces a customer on every order paid with it and
produces one accounting line per tender instead of one aggregated line per method.

**Intermediary account** — the account (`receivable_account_id`) a payment method uses in
the closing entry instead of the company's default counter receivable account.

**Invoiced order** — an order whose invoice link is filled. Its revenue and taxes are
recognised by the invoice, so the closing entry cancels its tender contributions with a
counterweight credit.

---

## K

**Kiosk** — a self-ordering device at the counter, shared by successive customers. Cash
payment methods may not be offered in kiosk mode, and payment is always taken per order.

---

## L

**Last data change** — the instant (`last_data_change`) at which any setting that affects
the selling application's master data last changed. The application compares it with its
own cached copy to decide whether to reload.

**Last preparation change** — the structured document recording the last state of an order
that was sent to the preparation printers, with the server date of that state.

**Line label** — the per-line identifier (`name`) drawn from the configuration's order line
sequence.

**Loading condition** — the per-entity selection condition used when the selling
application downloads its master data.

**Lock date** — an accounting date before which no entry may be created or changed. A
session may not start before one, and a lock date may not be moved past an open session.

---

## M

**Margin** — the tax-excluded amount of a line, signed by the order, minus the line's total
cost.

**Master data** — the products, prices, taxes, partners, payment methods, categories,
fiscal positions and settings downloaded when a session opens, so that the selling
application can work without the network.

**Maximum difference** — see *Authorised difference*.

---

## O

**Offline operation** — selling with the network unavailable. Master data and orders are
held in the browser's local database; orders are transmitted when connectivity returns.

**Online payment method** — a payment method settled through a payment provider rather
than at the counter. At most one may be offered per configuration.

**Opening control** — the session state in which the record exists but trading has not
begun and the opening cash count is being taken.

**Order line sequence** — the per-configuration numbering sequence (`order_line_seq_id`)
supplying the line label.

**Order sequence** — the per-configuration numbering sequence (`order_seq_id`) supplying
the prefix and suffix of the order name and the session-unique sequence number.

**Origin (of an order)** — where the order came from (`source`): the counter, a
self-ordering mobile device or a self-ordering kiosk.

**Outstanding account** — the transit account (`outstanding_account_id`) debited when an
accounting payment is created for a bank tender, pending the bank statement.

---

## P

**Paid amount** — the sum of the amounts of an order's tenders, change included
(`amount_paid`). Always recomputed server-side on transmission; the client's value is not
trusted.

**Pay-later kind** — the payment method kind derived from having no journal, or a journal
of neither the cash nor the bank type. A pay-later tender leaves a debt on a receivable
account and produces no statement line and no accounting payment.

**Payable amount** — the amount the customer must actually hand over, after cash rounding.

**Payment term line** — the display kind marking a journal item as a settlement line. The
counter receivable lines of the closing entry carry it, and are therefore excluded from
follow-up — except the pay-later ones, which are written with the exclusion off.

**Preparation printer** — a kitchen or bar printer that receives the items to prepare,
filtered to the categories attached to it.

**Preset** — a named bundle of service settings: a pricelist, a fiscal position, an
identification requirement, a return-mode flag and optional time slots.

**Price extra** — the sum of the attribute price supplements folded into a line's unit
price (`price_extra`).

**Price type** — how a line's unit price was arrived at: from the product or a pricelist
(`original`), typed by the cashier (`manual`), or set by a programme such as loyalty
(`automatic`).

**Price-included tax** — a tax already contained in the displayed unit price. Its base is
obtained by division and its amount by subtraction, never by multiplying the rounded base.

---

## Q

**Quick response code payment** — a payment method integration that shows the customer a
scannable code carrying the bank transfer details. The method also precomputes an
amount-less code so that something can be shown while offline.

---

## R

**Receipt number** — the externally visible reference of an order (`pos_reference`),
composed of the last two digits of the year, the device identifier, the configuration
identifier and the next backend sequence value.

**Recovery session** — see *Rescue session*.

**Refund line** — a line whose unit price multiplied by its quantity is negative. Its sign
is part of the sales aggregation key, so refunds and sales of the same product never merge
into one journal item.

**Refund order** — an order whose refund flag is set, or whose total is negative. Its sign
reverses the sense of every line.

**Refunded quantity** — the positive quantity already returned against a line, computed as
minus the sum of the quantities of its refunding lines whose order is not cancelled.

**Remaining amount due** — what is still owed on an order after the tenders recorded so
far, with a residue smaller than the rounding tolerance treated as zero.

**Rescue session** — a session created automatically to receive orders belonging to a
session that was already closed. It coexists with a normal session, computes its own
counted ending balance, and is closed from the administrative interface.

**Rounding line** — the journal item, named `Rounding line`, carrying the accumulated cash
rounding difference of a session.

**Rounding step** — the multiple to which a value is rounded: the currency's smallest
representable increment for currency rounding, or the cash rounding definition's step for
cash rounding.

---

## S

**Sales details** — the printable summary of a session or a period: products sold,
products refunded, taxes, payments, cash movements, discounts, invoices and totals.

**Selling application** — the browser application the cashier works in. A client-side
replica of the pricing and tax engine, able to work offline.

**Selling currency** — the currency of the configuration (`currency_id`). Every amount
handled by the selling application is expressed in it.

**Session** — one trading period of one configuration: the unit of cash accountability,
of accounting posting and of data loading.

**Session-unique sequence number** — a strictly increasing per-session number
(`sequence_number`) required by some legal regimes, drawn from the configuration's order
sequence with the prefix and suffix stripped.

**Ship later** — selling goods now and delivering them on a later date, through the
procurement rules rather than an immediate transfer.

**Split bill** — settling one restaurant order with more than one tender, or splitting the
order itself into two orders. The two have different accounting consequences.

**Stock reference** — a grouping token linking deferred delivery documents back to their
originating counter orders.

**Storno accounting** — an accounting style in which a correction is booked as a negative
amount on the original side rather than as a positive amount on the opposite side. A
post-closing reversal entry is treated as a storno entry when the company uses it.

**Symmetric rounding** — rounding that applies the configured method unchanged to negative
values.

---

## T

**Table** — a seat group in a restaurant, carrying a number, a shape, a position, a size
and a seat count. Tables may be pushed together by giving one a parent.

**Tender** — one payment applied to an order (`pos.payment`). Change is a negative tender.

**Terminal** — a card-reading device driven by a payment terminal integration through the
five-operation contract.

**Theoretical closing balance** — the starting balance plus the cash movements plus the
cash tenders; what the drawer should contain.

**Tip** — a gratuity, recorded either as a line carrying the tip product or, in a
restaurant, as an after-payment adjustment of the terminal authorisation.

**Tracking number** — the short customer-facing number (`tracking_number`), the backend
sequence value reduced modulo one thousand.

**Transfer** — a delivery or return document of the inventory domain, created by the
counter at sale time or at session closing.

**Trusted configuration** — another configuration, in the same currency, whose open orders
this configuration may see and settle. Synchronisation notifications are relayed to it.

---

## U

**Unfinished order** — an order in the `draft` state: being composed or transmitted but
not yet fully tendered. Only an unfinished order may have its lines changed or be deleted.

**Universally unique identifier** — the client-generated key (`uuid`) that makes the
transmission protocol idempotent. Present on orders, lines, tenders and restaurant
courses, each subject to a database uniqueness constraint.

---

## V

**Validation (of a session)** — the operation that builds, checks, posts and reconciles
the closing entry and moves the session to the closed state.

---

## W

**Weighted product** — a product sold by weight, sent to the scale instead of prompting
for a quantity, or scanned with a barcode encoding its weight.

---

## Cross-domain terms used in this file

| Term | Where it is defined in full |
| --- | --- |
| Journal, Journal Entry, Journal Item, reconciliation, lock date | [General Ledger](../general-ledger/README.md) |
| Invoice, credit note, payment terms, cash rounding definition | [Accounts Receivable](../accounts-receivable/README.md) |
| Accounting payment, outstanding account, bank statement line | [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) |
| Tax, tax group, tax repartition line, tax tag, tax engine | [Taxes](../taxes/README.md) |
| Currency, rounding step, conversion rate | [Multi-Currency](../multi-currency/README.md) |
| Product, variant, attribute, combo, barcode nomenclature | [Products and Catalog](../products-and-catalog/README.md) |
| Pricelist, pricelist rule | [Pricing and Pricelists](../pricing-and-pricelists/README.md) |
| Transfer, stock move, move line, lot, operation type, warehouse | [Inventory Operations](../inventory-operations/README.md) |
| Valuation layer, cost method, stock valuation account, expense account | [Inventory Valuation and Costing](../inventory-valuation-and-costing/README.md) |
| Unit of measure, unit conversion | [Units of Measure and Packaging](../units-of-measure-and-packaging/README.md) |
| Sales order, down payment | [Sales](../sales/README.md) |
| Loyalty programme, reward, coupon, loyalty card | [Loyalty and Promotions](../loyalty-and-promotions/README.md) |
| Payment provider, payment transaction | [Payment Providers](../payment-providers/README.md) |
| Message thread, activity, email template | [Messaging and Activities](../messaging-and-activities/README.md) |
