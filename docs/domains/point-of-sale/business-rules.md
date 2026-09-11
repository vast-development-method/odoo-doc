# Point of Sale — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case
behavior of the domain, with the exact user-facing message the system produces.

Placeholders in messages are written as words in angle brackets, for example
*<session identifier>*. Where the system's own text contains an irregularity (a missing
space, a double word, an abbreviation), the irregularity is reproduced exactly, because a
reimplementation must produce the same text: these strings are part of the observable
behavior, not prose.

---

## 1. Invariants

These properties must hold at all times; every rule in the rest of this file exists to
protect one of them.

| # | Invariant |
| --- | --- |
| I1 | At most one non-closed, non-recovery session exists per configuration. |
| I2 | A session that is not closed has no closing entry; a closed session has either a posted closing entry or none at all. |
| I3 | The closing entry balances: the sum of the balances of its journal items is zero. |
| I4 | An order in the paid or posted state has been fully tendered, within the cash rounding tolerance. |
| I5 | The paid amount of an order equals the sum of the amounts of its tenders, change included. |
| I6 | Every order of a closed session is in the posted state or the cancelled state. |
| I7 | An invoiced order contributes nothing net to the closing entry. |
| I8 | The quantity refunded against a line never exceeds the quantity sold on that line. |
| I9 | Every tender's payment method is one of the payment methods of the session's configuration. |
| I10 | Every universally unique identifier of an order, an order line and a payment is unique across the whole database. |
| I11 | Every amount handled by the selling application is expressed in the configuration currency, and every payment method, available pricelist and invoice journal agrees with that currency. |
| I12 | Exactly one cash register exists per session: the journal of the first cash-kind payment method of the configuration. |
| I13 | A cash journal carries at most one payment method, and a cash payment method is attached to at most one configuration. |
| I14 | The session's starting balance equals the counted amount entered at opening, so an opening discrepancy never becomes an accounting entry. |

---

## 2. Configuration rules

### 2.1 Currency agreement

Checked whenever the default pricelist, the use-pricelist flag, the available pricelists,
the point of sale journal, the invoice journal or the payment methods change.

| Condition | Message |
| --- | --- |
| The configuration uses a pricelist, has a default pricelist, and that pricelist is not among the available ones | The default pricelist must be included in the available pricelists. |
| A payment method has a journal, that journal has its own currency, and that currency is not the configuration currency | All payment methods must be in the same currency as the Sales Journal or the company currency if that is not set. |
| The configuration uses a pricelist and any available pricelist has a currency other than the configuration currency | All available pricelists must be in the same currency as the company or as the Sales Journal set on this point of sale if you use the Accounting application. |
| The invoice journal has its own currency and it is not the configuration currency | The invoice journal must be in the same currency as the Sales Journal or the company currency if that is not set. |

### 2.2 Company agreement

| Condition | Message |
| --- | --- |
| Any payment method belongs to a company other than the configuration's | The payment methods for the point of sale <configuration name> must belong to its company. |
| The default pricelist belongs to a company other than the configuration's | The default pricelist must belong to no company or the company of the point of sale. |
| Any available pricelist belongs to a company other than the configuration's | The selected pricelists must belong to no company or the company of the point of sale. |

### 2.3 Cash rules

| Condition | Message |
| --- | --- |
| Cash control is on and a cash payment method's journal has no loss account or no profit account | You need a loss and profit account on your cash journal. |
| A cash payment method of this configuration is already used by another configuration | This cash payment method is already used in another Point of Sale.\\nA new cash payment method should be created for this Point of Sale. |
| A cash payment method's journal already carries more than one payment method | You cannot use the same journal on multiples cash payment methods. |

### 2.4 Cash rounding

| Condition | Message |
| --- | --- |
| Cash rounding is on and the rounding method's strategy is not the add-a-rounding-line one | The cash rounding strategy of the point of sale <configuration name> must be: '<the label of the add-a-rounding-line strategy>' |

### 2.5 Trusted configurations

| Condition | Message |
| --- | --- |
| A trusted configuration uses a different currency | You cannot share open orders with configuration that does not use the same currency. |

### 2.6 Before a session may be opened

Checked in this order, and only when no session is open yet:

1. The company must have a chart of accounts, otherwise *"No chart of account configured,
   go to the "configuration / settings" menu, and install one from the Invoicing tab."*
2. The pricelist company rules of section 2.2 must hold.
3. The payment method company rule of section 2.2 must hold.
4. The currency rules of section 2.1 must hold.
5. The cash profit and loss rule of section 2.3 must hold.
6. At least one payment method must be configured, otherwise *"You must have at least one
   payment method configured to launch a session."*

Then, whether or not a session already exists:

7. Every stored field of the configuration is re-validated.
8. The company must have a fiscal country, otherwise *"The company must have a fiscal
   country set."*
9. The acting user must not be the system superuser, otherwise *"You do not have
   permission to open a point of sale session. Please try opening a session with a different
   user"*. (This check is skipped in a test environment.)

### 2.7 Fields frozen while a session is open

The frozen set is: the restaurant capability flag, the payment method list and the active
flag. The restaurant capability adds the floor list. Attempting to change any of them
while a non-closed session exists raises:

> Unable to modify this PoS Configuration because you can't modify *<the labels of the
> offending fields, joined by a comma and a space>* while a session is open.

Two exceptions:

- Setting the active flag to true is always allowed (a configuration may be reactivated
  with an open session; it may not be deactivated).
- The payment method list may be changed when the acting context explicitly bypasses the
  restriction, which is what the payment method form does when it rewrites its own
  attachments.

### 2.8 Receipt header and footer

Only an administrator of the platform may write the custom-header flag, the receipt
header or the receipt footer. Otherwise the write is refused as an access error:
*"Only administrators can edit receipt headers and footers"*.

### 2.9 Tip product

Clearing the tip product while turning tipping on restores the shipped tip product. When
that product does not exist the write is refused: *"The default tip product is missing.
Please manually specify the tip product. (See Tips field.)"*

A product used as a special product by a configuration cannot be archived: *"You cannot
archive a product that is set as a special product in a Point of Sale configuration.
Please change the configuration first."*

### 2.10 Operation type

An operation type used by a configuration cannot be archived:

> You cannot archive '*<operation type name>*' as it is used by point of sale configuration
> '*<configuration name>*'.

### 2.11 Consequential writes

| Change | Consequence |
| --- | --- |
| Order printing turned off | The printer list is cleared. |
| Tax regime selection turned on with a default fiscal position not in the list | The default fiscal position is added to the list. |
| Tax regime selection turned off | The fiscal position list is emptied. |
| A default preset set that is not among the available presets | It is added to the available presets. |
| The payment method list changed | The fast payment method list is filtered down to methods still offered; when it becomes empty, fast payment validation is turned off. |
| Order printing turned on or off | The preparation printers menu visibility is recomputed. |
| A direct printer address entered without a dot | It is converted into a certificate-bearing hostname before being stored. |
| A fiscal position archived | Every configuration using it as the default has that field cleared. |

### 2.12 Writes arriving from the settings screen

The settings screen always sends link commands for its many-valued fields, never unlink
commands, so unlinking would never happen. Two corrections are applied when the acting
context marks the write as coming from that screen:

1. For every many-valued field present in the write, the records currently linked but not
   mentioned by a link command are turned into explicit unlink commands, prepended to the
   command list.
2. Every field whose written value would not actually change the record is dropped from
   the write, so that the frozen-field check of section 2.7 does not fire on a value that
   is not really changing.

The second correction also means that re-saving the settings screen with an open session
is harmless as long as nothing frozen actually changed.

---

## 3. Session rules

### 3.1 Creation

| Condition | Message |
| --- | --- |
| No configuration supplied, directly or through the acting context | You should assign a Point of Sale to your session. |
| More than one non-closed, non-recovery session exists for the configuration | Another session is already opened for this point of sale. |
| The opening date violates a lock date of the point of sale journal's company | You cannot create a session starting before: <the formatted list of violated lock dates> |

The one-open-session check is skipped when the acting context marks the creation as an
onboarding creation.

A session created by a user in the point of sale user group is created with elevated
rights, so that a cashier without accounting rights can still open a till.

### 3.2 Cancelling an unused session

| Condition | Message |
| --- | --- |
| The session is not in opening control, or it already has at least one order | You can only cancel a session that is in opening control state and has no orders. |

### 3.3 Closing

| Condition | Message | Redirects the cashier to the administrative interface |
| --- | --- | --- |
| An order of the session due now or earlier is unfinished | You cannot close the point of sale while there are still draft orders for the day. | No |
| The session is already closed | The session has been already closed by another User. All sales completed in the meantime have been saved in a Rescue Session, which can be reviewed anytime and posted to Accounting from Point of Sale's dashboard. (Shown as an alert titled *Session already closed*.) | Yes |
| The session is already closed, as seen from the administrative interface | This session is already closed. | — |
| A bank payment method was given a negative difference and its journal has no loss account | Need loss account for the following journals to post the lost amount: <journal names>\\n | No |
| A bank payment method was given a positive difference and its journal has no profit account | Need profit account for the following journals to post the gained amount: <journal names> | No |
| Storing the counted cash on a session with no cash register | There is no cash register in this session. | — |

### 3.4 Validation

| Condition | Message |
| --- | --- |
| An order of the session is unfinished | There are still orders in draft state in the session. Pay or cancel the following orders to validate the session:\\n<order names, joined by a comma and a space> |
| An invoice of a closed order is not posted | You cannot close the point of sale when invoices are not posted.\\nInvoices: <one line per invoice, each the invoice number, a space, a hyphen, a space, and the invoice state> |
| A prepared tax line has no account | Unable to close and validate the session.\\nPlease set corresponding tax account in each repartition line of the following taxes: \\n<tax names, joined by a comma and a space> |
| A negative cash difference must be posted and the cash journal has no loss account | Please go on the <cash journal name> journal and define a Loss Account. This account will be used to record cash difference. |
| A positive cash difference must be posted and the cash journal has no profit account | Please go on the <cash journal name> journal and define a Profit Account. This account will be used to record cash difference. |
| The closing entry does not balance | Not an error: the transaction is rolled back and the forced-close wizard is offered, carrying the message *There is a difference between the amounts to post and the amounts of the orders, it is probably caused by taxes or accounting configurations changes.* |

### 3.5 The authorised cash difference

When the configuration sets a maximum difference, the selling application refuses a
closing whose absolute cash difference exceeds the configured limit unless the acting user
is a counter administrator. The limit and the role are transmitted with the closing
control data; the server does not repeat the check, so a reimplementation must enforce it
in the selling application.

### 3.6 Locking rules protecting sessions

- **Accounting lock dates.** A company's fiscal-year, tax, sale or hard lock date cannot
  be moved while a session that would violate it is open:
  *"Please close all the point of sale sessions in this period before closing it. Open
  sessions are: <session names> "* (note the trailing space). The sessions considered are
  the non-closed sessions of the company or of a descendant company whose opening instant
  is at or before the greater of the fiscal-year and hard lock dates, or at or before the
  tax lock date, or — when the configuration's point of sale journal is a sale journal —
  at or before the sale lock date.
- **Row-level lock on the configuration.** Opening the selling application takes a
  no-wait row-level lock on the configuration record before creating a session, so that
  two simultaneous openings cannot create two sessions.
- **Invoicing lock.** Generating an invoice takes an exclusive lock on the orders. A
  second attempt is refused with *"Some orders are already being invoiced. Please try
  again later."*

### 3.7 Deletion

Deleting a session first deletes its cash statement lines. There is no guard beyond the
ordinary access rights, so deletion is effectively reserved to administrators; the
supported path for an unused session is the cancel operation of section 3.2.

---

## 4. Order rules

### 4.1 Creation and transmission

| Condition | Message |
| --- | --- |
| A transmission contains lines refunding more than one distinct order | You can only refund products from the same order. |
| The named session is closing or closed and no open session exists for the configuration | No open session available. Please open a new session to capture the order. |
| Change must be recorded and the session has no cash payment method | No cash statement found for this session. Unable to record returned cash. |
| The order asks to be invoiced and the configuration has no invoice journal | No invoice journal configured for this point of sale session. |
| The order has no currency when its amounts are recomputed | You can't: create a pos order from the backend interface, or unset the pricelist, or create a pos.order in a python test with Form tool, or edit the form view in studio if no PoS order exist |

A customer identifier that no longer exists is silently dropped, together with the
to-invoice flag.

### 4.2 Payment completeness

| Condition | Message |
| --- | --- |
| The order is being marked paid and the tendered amount does not cover the total, beyond the cash rounding tolerance | Order <order name> is not fully paid. |
| A write touches the tenders, the paid amount is below the total and the order is paid or posted | The paid amount is different from the total amount of the order. |
| A write touches the tenders, the paid amount is above the total and the order is paid | Not an error: the line *Warning, the paid amount is higher than the total amount. (Difference: <amount>)* is appended to the payment-changes message. |

### 4.3 Editing a paid order

| Condition | Message |
| --- | --- |
| Writing a state other than paid, posted or invoiced onto an order already in one of those states | This order has already been paid. You cannot set it back to draft or edit it. |
| Cancelling from the administrative interface when no selected order is unfinished | This order has already been paid. You cannot set it back to draft or edit it. |
| Cancelling from the administrative interface when a selected order is scheduled for a future date | The order delivery / pickup date is in the future. You cannot cancel it. |
| Adding or changing a tender whose status is anything other than cancelled on an order printed at least once | You cannot change the payment of a printed order. |
| Deleting an order that is neither unfinished nor cancelled | In order to delete a sale, it must be new or cancelled. |
| Deleting a line whose order is neither unfinished nor cancelled | You can only unlink PoS order lines that are related to orders in new or cancelled state. |

### 4.4 The deleted-line flag

Once an order's deleted-line flag is true, an ordinary write can never set it back to
false: the key is silently dropped from the write values. Only the creation of a fresh
order clears it.

### 4.5 Refunds

| Condition | Message |
| --- | --- |
| A refund line's quantity would exceed the outstanding quantity of the refunded line | You cannot refund more than the outstanding quantity for this product. |
| Refunding from the administrative interface when the order's configuration has no open session | To return product(s), you need to open a session in the point of sale <configuration display name> |

### 4.6 Invoicing

| Condition | Message |
| --- | --- |
| No selected order is invoiceable | No valid orders were selected. No new invoices could be generated |
| A consolidated invoice is asked for a selection containing a refund of an invoiced order, and the selection holds more than one order | The following refund orders can't be part of a consolidated invoice because they refunded invoiced orders. Each refund order should be handled separately.\\n\\n<one line per order, each the order name followed by its receipt number in parentheses> |
| A consolidated group has no customer | Kindly ensure that each order contains a customer. |
| Another invoicing attempt holds the lock | Some orders are already being invoiced. Please try again later. |
| An invoice of an order whose session is still open is reset to draft | Not an error: a notification is pushed to the acting user reading *You can't reset this invoice to draft because the POS session is still open. Please close the ongoing session first, then try again.* and the reset does not happen. |

### 4.7 Deleting a customer

A partner with counter orders cannot be deleted: *"You cannot delete a customer that has
point of sales orders. You can archive it instead."*

### 4.8 The uniqueness constraints

| Entity | Constraint | Message |
| --- | --- | --- |
| Order | Unique universally unique identifier | An order with this uuid already exists |
| Order line | Unique universally unique identifier | An order line with this uuid already exists |
| Payment | Unique universally unique identifier | A payment with this uuid already exists |
| Predefined note | Unique name | A note with this name already exists |

### 4.9 Idempotency of transmission

The transmission protocol is idempotent on the order's universally unique identifier and,
inside an order, on each line's and each tender's universally unique identifier:

- an order already present and not unfinished is returned unchanged, with the event
  recorded in the server log;
- an order already present and unfinished is updated;
- a create command for a line whose universally unique identifier already exists on that
  order is converted into an update of the existing line before the write, so a replayed
  transmission never duplicates lines.

With the restaurant capability, the match is widened: an unfinished order on the same
table of the same configuration is treated as the same order even when the identifiers
differ.

### 4.10 The preparation-change ordering rule

When the transmitted last-preparation-change document and the stored one both carry a
metadata block, the one with the later server date wins. When the stored one wins, the
event is recorded in the server log as an outdated preparation change probably caused by a
synchronisation problem. When the transmitted one wins, its server date is replaced by the
current server time before storing.

---

## 5. Order line rules

| Condition | Behavior |
| --- | --- |
| A line is created without a label | The label is taken from the configuration's order line sequence, falling back to the generic order line sequence. |
| A product resolves to no income account and the configuration's point of sale journal has no default account | Refused with *"Please define income account for this product: '<product name>' (id:<product identifier>)."* |
| The quantity is reduced while edit tracking is on | The line's edited flag is set and *"<product name>: Ordered quantity: <old quantity>→<new quantity>"* is posted on the order thread. |
| A line is deleted while edit tracking is on | The order's deleted-line flag is set and *"<product name>: Deleted line (quantity: <quantity>)"* is posted on the order thread. |
| A line is transmitted with a combo parent expressed as a universally unique identifier | The identifier is resolved to the real parent and the temporary key is removed from the values. |
| A field that the line model refuses to accept from the wire is transmitted | The combo parent and the combo children are never accepted directly from a transmission; they are resolved through the identifier mechanism instead. |

---

## 6. Payment rules

| Condition | Message |
| --- | --- |
| The amount is written on a tender of an order that is posted or already invoiced | You cannot edit a payment for a posted order. |
| A tender's method is not among the payment methods of the session's configuration | The payment method selected is not allowed in the config of the point of sale session. |
| A tender of an identify-customer method is aggregated at closing and its order has no customer | You have enabled the "Identify Customer" option for <payment method name> payment method,but the order <order name> does not contain a customer. |
| The administrative payment wizard is confirmed with an identify-customer method and the order has no customer | Customer is required for <payment method name> payment method. |

A tender whose amount is zero at the selling currency's precision is skipped by every
closing aggregation.

---

## 7. Payment method rules

### 7.1 Write restriction

Any write is refused when the method is offered on a configuration with a non-closed
session, unless every written field is `sequence`:

> Please close and validate the following open PoS Sessions before modifying this payment
> method.
> Open sessions: *<session names, joined by a space>*

### 7.2 Journal rules

| Condition | Message |
| --- | --- |
| A journal of a type other than cash or bank is chosen | Only journals of type 'Cash' or 'Bank' could be used with payment methods. |
| A journal attached to a payment method has its type changed | This journal is associated with a payment method. You cannot modify its type |
| A journal attached to a payment method is archived or deleted | You can not archive this journal because it is set on the following payment method : <method name>. |

When the chart of accounts is swapped, the payment methods of the deleted journals and the
configurations using those journals are deleted with them.

### 7.3 Quick response code rules

| Condition | Message |
| --- | --- |
| The integration is quick response code and the journal is not a bank journal with a bank account | At least one bank account must be defined on the journal to allow registering quick response code payments with Bank apps. |
| The integration is quick response code and no format is chosen | You must select a QR-code method to generate QR-codes for this payment method. |
| The bank account cannot produce the chosen format for the company currency | The format's own error message. |
| A code is requested on a method that is not configured for it | This payment method is not configured to generate quick response codes. |

### 7.4 Company and shop rules

| Condition | Message |
| --- | --- |
| A configuration offering the method belongs to another company | The points of sale for the payment method <method name> must belong to its company. |
| A cash method is attached to more than one configuration | Validation Error: You cannot assign the same Cash payment method to multiple point of sale Shops. Please create a separate Cash payment method for each shop. |

### 7.5 Duplication

Duplicating a method clears its configuration list. When the original sits on a cash
journal and the duplicate would keep that journal, the journal is cleared on the
duplicate, because a cash journal may carry only one method.

### 7.6 Integration exclusivity

Choosing an integration clears the fields of the other integrations, as described in
[`entities.md`](entities.md) section 7.4.

---

## 8. Category, note, denomination, preset and printer rules

| Entity | Condition | Message |
| --- | --- | --- |
| Category | A category would become its own ancestor | Error! You cannot create recursive categories. |
| Category | An availability hour is outside the range from zero to twenty-four | The Availability Until must be set between 00:00 and 24:00 / The Availability After must be set between 00:00 and 24:00 |
| Category | The until hour is smaller than the after hour | The Availability Until must be greater than Availability After. |
| Category | A category is deleted while any session anywhere is open | You cannot delete a point of sale category while a session is still opened. |
| Denomination | A denomination is created by typing a non-numeric name | The name of the Coins/Bills must be a number. |
| Preset | An attendance line's start hour, modulo twenty-four, is not strictly smaller than its end hour modulo twenty-four | The start time must be before the end time. |
| Preset | A preset attached to a configuration is deleted | You cannot delete a preset that is linked to a point of sale configuration. |
| Preset | A shipped master preset is deleted | You cannot delete the master preset(s). |
| Printer | The printer type is the direct one and no address is given | Epson Printer IP Address cannot be empty. |

---

## 9. Tax, journal, sequence and rounding protections

| Condition | Message |
| --- | --- |
| A tax's kind, amount, scope, group, price-included behavior, base-inclusion behavior or base-affected behavior is changed while a counter line carrying it belongs to an open session | It is forbidden to modify a tax used in a point of sale order not posted. You must close the point of sale sessions before modifying the tax. |
| A numbering sequence used by a configuration is deleted | You cannot delete a sequence used in an active point of sale config: <the names of the order sequences of the affected configurations> |
| A cash rounding definition used by a configuration is deleted | You cannot delete a rounding method that is used in a Point of Sale configuration. |
| A cash rounding definition's step, method or strategy is changed while a session using it is open | You are not allowed to change the cash rounding configuration while a pos session using it is already opened. |

A tax counts as **used** — and therefore becomes harder to change — as soon as any counter
order line carries it, even a line of an unfinished order.

---

## 10. Inventory rules

| Condition | Message or behavior |
| --- | --- |
| A unit conversion on a move would round the demanded quantity down to zero | Conversion Error: The following unit of measure conversions result in a zero quantity due to rounding:\\n - From "<source unit>" to "<target unit>"\\n…\\n\\nThis issue occurs because the quantity becomes zero after rounding during the conversion. To fix this, adjust the conversion factors or rounding method to ensure that even the smallest quantity in the original unit does not round down to zero in the target unit. |
| Completing a counter transfer fails (insufficient stock, missing lot, a validation of the inventory domain) | The failure is swallowed: the transfer is left incomplete, the sale still completes, and the session reports a failed transfer. |
| A refund of goods never delivered, in full | The original transfer is cancelled and no return transfer is created. |
| A refund of goods never delivered, in part | The matching moves' demanded quantities are reduced, a move reaching zero is cancelled and deleted, and the remaining ones are re-reserved. No return transfer is created. |
| A counter transfer would trigger the ordinary delivery confirmation email or text message | It is suppressed: transfers whose operation type is the warehouse's point-of-sale type are excluded. |

---

## 11. Permission checks

### 11.1 Security groups

| Group | Implies |
| --- | --- |
| Point of Sale / User | — |
| Point of Sale / Administrator | Point of Sale / User and Inventory / User |
| Preset Menu | — (a visibility group that reveals the preset menu) |

### 11.2 Checks written into the operations

| Operation | Requirement | Message on failure |
| --- | --- | --- |
| Reading the cash-in and cash-out list | The acting user is in the point of sale user group | You don't have the access rights to get the cash in/out list. |
| Reading the closing control data | The acting user is in the point of sale user group | You don't have the access rights to get the point of sale closing control data. |
| Recording a cash in or cash out | The acting user is a counter administrator **or** holds invoice-level accounting rights | You don't have the access rights to perform a cash in/out. |
| Deleting a cash movement | The acting user is a counter administrator **or** holds basic accounting rights | You don't have the access rights to delete a cash in/out. |
| Deleting a cash movement | The movement belongs to this session | You cannot delete a cash move that is not linked to this session. |
| Editing the receipt header or footer | The acting user is a platform administrator | Only administrators can edit receipt headers and footers |
| Opening the selling application | The acting user is an internal user | The page is reported as not found. |
| Opening the selling application | The acting user is not the system superuser | You do not have permission to open a point of sale session. Please try opening a session with a different user |
| Changing a unit price when price control is restricted | The acting user is a counter administrator | Enforced by the selling application; the server does not repeat the check. |
| Seeing cost and margin in the product information panel | The margins-and-costs flag is set, or the acting user is a counter administrator | Enforced by the selling application. |
| Posting a cash difference above the authorised limit | The acting user is a counter administrator | Enforced by the selling application. |

### 11.3 Elevated execution

Several operations run with elevated rights so that a cashier without accounting rights
can still work:

- creating a session, when the acting user is in the point of sale user group;
- validating a session, for the same reason;
- reading and summing the session's cash statement lines, which the accounting record
  rules would otherwise hide;
- creating, reading and deleting cash movements;
- creating and posting the invoice of a counter order;
- creating the invoice payment entries;
- searching the stock moves that feed the cost aggregation;
- reading the counter orders behind an invoice when computing the invoiced lots.

### 11.4 Failures while loading data

When the acting user may not read one of the entities loaded at session opening, that
entity is returned empty and the event is recorded in the server log as an access failure
for that entity. The session still opens. Likewise, a record the user may not read is
reported as no longer relevant, so the selling application drops it from its cache.

---

## 12. Self-ordering rules

| Condition | Message |
| --- | --- |
| Self-ordering is enabled and the default user is missing or is neither a counter user nor a counter administrator | The Self-Order default user must be a point of sale user |
| The self-ordering mode is kiosk and a cash payment method is offered | You cannot add cash payment methods in kiosk mode. |
| A submitted order names a preset while the configuration uses presets and the preset does not exist | Invalid preset |
| The preset is not available in self-ordering and is not the configuration default | Preset is not available in self-ordering |
| The preset is not among the configuration's available presets | Preset is not available in this configuration |
| A submitted line carries a quantity that is not a finite number strictly greater than zero | Invalid quantity |
| A submitted line carries an attribute value that does not exist or does not belong to the ordered product's template | Invalid product attribute |
| A submitted line carries a combo structure that does not match the product | Invalid combo line |
| Codes are requested in a mode other than mobile or browse-only | quick response codes can only be generated in mobile or consultation mode. |
| Codes are requested in table-service mode with no table | In Self-Order mode, you must have at least one table to generate quick response codes |
| A pay-after value other than each order is chosen in kiosk mode | Only pay after each is available with kiosk mode. |

Server-side sanitisation of a submitted order, beyond the messages above:

- a delete or unlink command is honoured only for a line that already belongs to the order
  being submitted; otherwise it is dropped;
- an update command is honoured only for a line that already belongs to that order;
  otherwise it is dropped, so an arbitrary line cannot be re-parented onto this order;
- the taxes are recomputed from the product and the fiscal position, ignoring anything the
  payload claims;
- the line amounts are written as zero and recomputed server-side, so the payload cannot
  dictate a price;
- only whole-number attribute value identifiers and whole-number combo child identifiers
  are accepted.

Forced settings, applied on every write:

- kiosk mode forces pay-after to each order;
- mobile mode with pickup-zone service, or without the restaurant capability, forces
  pay-after to each order;
- mobile mode with pay-after set to meal forces the service mode to table.

---

## 13. Online payment rules

| Condition | Message |
| --- | --- |
| More than one online payment method is offered on one configuration | A point of sale config cannot have more than one online payment method. |
| An online payment method's providers do not all use the configuration currency | All payment providers configured for an online payment method must use the same currency as the Sales Journal, or the company currency if that is not set, of the point of sale config. |
| An online payment method has no published provider supporting the configuration currency | To use an online payment method in a point of sale config, it must have at least one published payment provider supporting the currency of that point of sale config. |
| An online tender is created with no accounting payment | Cannot create a point of sale online payment without an accounting payment. |
| An accounting payment marked online is attached to a tender whose method is not an online method | Cannot create a point of sale payment with a not online payment method and an online accounting payment. |
| The essential data of an online tender is edited | Cannot edit a point of sale online payment essential data. |
| An order already carries an online payment and another is attempted | The <order name> already has one online payment. |
| A payment transaction carries a negative amount | The payment transaction (<transaction identifier>) has a negative amount. |
| The partner of an online tender cannot be resolved | The partner of the point of sale online payment (id=<identifier>) could not be found |
| An online tender cannot be saved | The point of sale online payment (tx.id=<transaction identifier>) could not be saved correctly |
| An online tender cannot be saved because the method is missing | The point of sale online payment (tx.id=<transaction identifier>) could not be saved correctly because the online payment method could not be found |

---

## 14. Restaurant rules

| Condition | Message |
| --- | --- |
| A floor used by a configuration with an open session is deleted | You cannot remove a floor that is used in a PoS session, close the session(s) first: \\n followed by one line per pair, each *Floor: <floor name> - PoS Config: <configuration name> \\n* |
| The configuration list or the active flag of a floor is written while a configuration using it has an active session | Please close and validate the following open PoS Session before modifying this floor.\\nOpen session: <configuration names, joined by a space> |
| A floor is deactivated while an unfinished order sits on one of its tables | You cannot delete a floor when orders are still in draft for this floor. |
| A table used by a configuration with an open session is deleted | You cannot remove a table that is used in a PoS session, close the session(s) first. |
| A table with unfinished orders is removed | You cannot delete a table when orders are still in draft for this table. |
| A table's parent would create a cycle | Not an error: the previous parent is silently restored. |

Creating a configuration with the restaurant capability and no floor creates a default
floor named after the company, with one square table numbered 1, one seat, positioned at
horizontal 100 and vertical 100, sized 130 by 130. Turning the capability off clears the
floor list and forces the tip-after-payment flag off.

---

## 15. Employee rules

| Condition | Message |
| --- | --- |
| An employee that may be used in an open session is deleted | You cannot delete an employee that may be used in an active PoS session, close the session(s) first: \\n followed by one line per pair, each *Employee: <employee name> - PoS Config(s): <configuration names> \\n* |
| A user chosen for a self-ordering or employee role is not a counter user | The user must be a point of sale user |

---

## 16. Loyalty rules at the counter

| Condition | Message |
| --- | --- |
| A coupon is entered that grants no claimable reward | No reward can be claimed with this coupon. |
| A coupon is entered outside its validity window, before it starts | This coupon is not yet valid (<date>). |
| A coupon is entered after it expired | This coupon is expired (<date>). |
| A coupon is entered that does not match the order's pricelist | This coupon is not available with the current pricelist. |
| A coupon is entered that is not valid at all | This coupon is invalid (<code>). |
| A programme requiring a code is applied without one | This programs requires a code to be applied. |
| A coupon does not hold enough points for the reward | There are not enough points for the coupon: <coupon>. |
| Some applied coupons became invalid after the order changed | Some coupons are invalid. The applied coupons have been updated. Please check the order. |
| A reward product is not available at the counter | To continue, make the following reward products available in Point of Sale. |
| A gift card programme is set to print cards but has no printable document | There is no print report on the gift card program and your pos is set to print them. |
| A gift card programme is set to send cards but has no email template | There is no email template on the gift card program and your pos is set to print them. |
| Generated gift card codes collide with existing ones | The following codes already exist in the database, perhaps they were already sold?\\n<the offending codes> |

---

## 17. Combo rules

| Condition | Message |
| --- | --- |
| A combo's free quantity is negative | The free quantity of a combo must be greater or equal to 0. |
| A combo's maximum quantity is below one | The maximum quantity of a combo must be greater or equal to 1. |
| A combo's free quantity exceeds its maximum quantity | The free quantity must be smaller or equal to the maximum quantity. |
| A product that belongs to a combo is removed from the counter catalogue | You must first remove this product from the <combo name> combo |

---

## 18. Portal invoice request rules

| Condition | Message |
| --- | --- |
| A required field of the request form is empty | Please fill all the required fields. |
| The supplied receipt number is shorter than twelve characters | The Ticket Number should be at least 12 characters long. |
| No order matches the receipt number, date window and code | No sale order found. |
| The validation page is reached with no access token, or with a token matching no order | The page is reported as not found. |
| A required detail of the signed-in visitor's own record is missing | The <field label> must be filled in your details. |
| A required extra field of the fiscal country is missing | The field <field label in lower case> must be filled. |
| The submitted form is invalid | Message: <messages>\\nPlease update the <invalid fields> |

The date window is: the supplied date minus one day, inclusive, to the supplied date plus
two days, exclusive. The receipt number is matched by suffix, with the percent and
underscore characters escaped so that they are treated literally.

A visitor may edit a partner only when that partner is their own, or is a child contact of
their commercial entity of the invoice, delivery or other kind.

---

## 19. Edge cases

| Situation | Behavior |
| --- | --- |
| A session is validated with no order and no cash movement | Only the cash difference is posted, and the session is marked closed. No closing entry is created. |
| A closing entry ends up with no line at all | It is deleted instead of posted, and the paid orders are **not** moved to posted. |
| An order is transmitted twice | The second transmission is a no-operation for a non-unfinished order, and an idempotent update for an unfinished one. |
| An order is transmitted against a closed session | It is re-homed to the open session of the same configuration, with a warning in the server log; without one, the transmission is refused. |
| An order scheduled for a future time exists when the session closes from the selling application | Its session link is emptied so that it survives into the next session; it does not block the closing. |
| A refund and its original sale are in the same session | The sales buckets are **not** netted (the sign is part of the key) but the tender buckets are (they are keyed only by method). A cash bucket that nets to zero produces neither a statement line nor a receivable line. |
| A payment method is archived | It is still loaded into the selling application, so historical orders remain readable. |
| The selling currency is a foreign currency | Every accounting line except the cost and valuation lines carries an amount in currency and the selling currency; the cost and valuation lines are written in company currency only. Rounding drift on the converted tender amounts is absorbed by the residual correction on the last payment-term line of the reversal entry, and by the balancing line of the closing entry when the operator forces a close. |
| The tax configuration changes mid-session | Protected by the tax rule of section 9; when it happens anyway (for example through a data change that bypasses the constraint), the closing entry may not balance and the forced-close wizard is offered. |
| Two cashiers close the same session simultaneously | The second one is told the session is already closed and is redirected to the administrative interface; the orders it still held are captured by a rescue session. |
| A product is sold that is not loaded in the selling application | It is fetched on demand by barcode or by search; a product that cannot be fetched cannot be sold. |
| A combo's components sum to a value different from the combo price because of rounding | The residual is always added to the last component, so the components sum exactly to the combo price. |
| A line's cost cannot be established at sale time because stock is updated at closing | The cost stays uncomputed; it is computed at closing from the deferred transfer's moves. |
| An order is invoiced after its session closed | A reversal entry removes the order's contribution from the closing entry, and the invoice and its payment entries take its place. |
| A pay-later tender is on an invoiced order | It produces neither a closing-entry line nor a payment entry; the invoice carries the receivable, with the customer's payment terms applied. |
