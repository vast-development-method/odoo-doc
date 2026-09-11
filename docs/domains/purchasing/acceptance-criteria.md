# Purchasing — Acceptance Criteria

Numbered scenarios in Given / When / Then form with concrete numbers. A rebuilt implementation
is behaviourally equivalent when it satisfies every one of them.

> **Reproduced text.** Status labels, button labels, subtype names and message bodies are
> reproduced exactly as the system produces them, because a rebuilt implementation must
> produce the same text. Some shipped strings contain the short form of *request for
> quotation*; that short form appears only inside such reproduced strings and never in this
> specification's own prose. See the conventions in [`README.md`](README.md).

## Common fixture

Unless a scenario says otherwise, assume:

- One company, **Northwind Supplies**, whose currency is the euro, whose tax rounding method is
  per line, whose order modification policy is *edit*, whose approval policy is *one step* and
  whose double-validation amount is 5 000.00.
- The named decimal precisions are the shipped ones: quantities to 2 digits, product prices to
  2 digits, discounts to 2 digits. The euro rounds to 0.01.
- A purchase tax **Purchase Tax 15 %**, not included in the price, fully recoverable.
- A vendor **Wood Corner**, a supplier with no parent company, no supplier currency (so orders
  are in euro), no reminder configured, and a supplier location configured.
- A storable product **Office Chair**, reference unit *Units*, control policy *on received
  quantities*, cost 45.00, vendor taxes = Purchase Tax 15 %, one vendor pricelist entry for Wood Corner:
  minimum quantity 1, price 60.00, lead time 4 days, discount 0, currency euro.
- A service product **Assembly Service**, control policy forced to *on ordered quantities*, no
  vendor pricelist entry, cost 0.00, vendor taxes = Purchase Tax 15 %.
- A buyer **Bea** holding the purchase user privilege, and an administrator **Adam** holding the
  purchase administrator privilege.
- Inventory is installed, with one warehouse receiving in one step, unless stated otherwise.
- Today is the 1st of a month; the order deadline of a newly created order is today.

---

## 1. Creating and pricing a request for quotation

**1.1 A new request for quotation gets a reference and defaults**

- **Given** Bea is signed in and the numbering series `purchase.order` stands at 30.
- **When** she creates a purchase order record with vendor Wood Corner and saves it.
- **Then** the reference is `P00031`; the status is `draft`; the buyer is Bea; the company is
  Northwind Supplies; the currency is the euro; the order deadline is today; the priority is
  normal; the billing status is *Nothing to Bill*; the locked flag is false; the acknowledged
  flag is false; and the operation type is the warehouse's incoming operation type.

**1.2 Choosing a vendor applies its defaults**

- **Given** Wood Corner has the supplier payment term *30 days*, an assigned buyer **Ben**, a
  supplier currency of United States dollars, and the reminder enabled with 3 days.
- **When** Bea sets Wood Corner as the vendor of a new draft order.
- **Then** the payment terms become *30 days*; the buyer becomes Ben; the currency becomes
  United States dollars; the reminder flag becomes true; the days before receipt becomes 3; and
  the fiscal position is the one resolved for Wood Corner.

**1.3 Adding a product suggests a quantity and a price**

- **Given** the fixture.
- **When** Bea adds a line and picks Office Chair.
- **Then** the unit becomes *Units*; the description becomes the product's display name; the
  taxes become Purchase Tax 15 %; the quantity becomes 1 (the minimum quantity of the only qualifying
  vendor entry); the unit price becomes 60.00; the discount becomes 0; and the expected arrival
  becomes the 5th (the order deadline plus 4 days).

**1.4 A vendor price break at one hundred units**

- **Given** Office Chair has a second vendor pricelist entry for Wood Corner: minimum quantity
  100, price 10.50, lead time 5, discount 0; and the first entry is minimum quantity 1, price
  12.00, lead time 5. Both are valid today.
- **When** Bea adds Office Chair to a new order.
- **Then** the quantity is 1 and the unit price is 12.00.
- **When** she changes the quantity to 100.
- **Then** the unit price becomes 10.50 and the subtotal becomes 1 050.00.
- **When** she changes the quantity to 99.
- **Then** the unit price returns to 12.00 and the subtotal becomes 1 188.00.
- **When** she instead types a unit price of 11.00 by hand and then changes the quantity to 100.
- **Then** the unit price stays 11.00, because the stored unit price differs from the technical
  unit price and the automatic computation is suppressed; the subtotal becomes 1 100.00.

**1.5 The price falls back to the product cost**

- **Given** Wood Corner has no vendor pricelist entry at all for Office Chair, and the product's
  cost is 45.00 in the euro.
- **When** Bea adds Office Chair to a new order for Wood Corner.
- **Then** the quantity becomes 1; the discount becomes 0; the unit price becomes 45.00; and the
  expected arrival becomes today, because the lead time of an absent entry is zero.

**1.6 A manual price survives when the vendor sells the product at no listed price**

- **Given** Wood Corner has no vendor pricelist entry for Office Chair and a draft line already
  carries a unit price of 52.00 typed by hand, in unit *Units*.
- **When** Bea changes the quantity from 3 to 4 without changing the unit.
- **Then** the unit price stays 52.00.
- **When** she instead changes the unit from *Units* to *Dozens*.
- **Then** the unit price is recomputed from the product's cost restated into dozens, because
  the preservation branch requires the unit to be unchanged.

**1.7 Sections, subsections and notes**

- **Given** a draft order.
- **When** Bea adds a section *Furniture* at sequence 10, a subsection *Seating* at 20, a
  product line for Office Chair at 30, and a note at 40.
- **Then** the section's parent is empty; the subsection's parent is the section; the product
  line's parent is the subsection; and the database refuses any attempt to give the section a
  product, a unit, a price other than zero, a total quantity other than zero or an expected
  arrival.

**1.8 A line that is neither a display line nor a down payment must be complete**

- **Given** a draft order.
- **When** an attempt is made to store a line with no display type, no down-payment flag and no
  product.
- **Then** the database refuses it with *Missing required fields on accountable purchase order
  line.*

**1.9 Header amounts**

- **Given** a draft order with line 1 = 7 units at 13.33, no discount, Purchase Tax 15 %; and line 2 = 3
  units at 49.99 with a 10 % discount, Purchase Tax 15 %.
- **When** the order is saved.
- **Then** line 1's subtotal is 93.31 and its tax is 14.00; line 2's subtotal is 134.97 and its
  tax is 20.25; the untaxed amount is 228.28; the tax amount is 34.25; the total is 262.53.
  (Each line's tax is rounded to the euro separately because the company rounds per line:
  93.31 × 0.15 = 13.9965 → 14.00, and 134.97 × 0.15 = 20.2455 → 20.25.)

**1.10 The header expected arrival is the earliest line arrival**

- **Given** an order with three product lines whose expected arrivals are the 12th, the 7th and
  the 20th, plus a note line with none.
- **Then** the header expected arrival is the 7th.
- **When** the buyer writes the 15th on the header.
- **Then** all three product lines receive the 15th and the header stays at the 15th; the note
  line is untouched.

---

## 2. Sending, confirming and approving

**2.1 Sending moves a draft order to sent**

- **Given** an order in the `draft` status.
- **When** Bea presses *Send RFQ* and sends the message.
- **Then** the status becomes `sent`; a message is posted under the *RFQ Sent* subtype; the
  message carries the quotation document; and the notification's portal button reads
  *View Quotation*.

**2.2 Sending an already sent order does not change the status**

- **Given** an order in the `sent` status.
- **When** Bea sends it again.
- **Then** the status stays `sent`.

**2.3 Printing the quotation also marks a draft order as sent**

- **Given** two orders, one `draft` and one `purchase`, selected together.
- **When** Bea presses *Print* on the quotation document.
- **Then** the draft order becomes `sent`; the confirmed order is unchanged; both documents are
  produced.

**2.4 Confirmation refuses a line with no product**

- **Given** a draft order with one product line and one line that has no display type, no
  down-payment flag and no product (created before the constraint could apply, for example by
  clearing the product in a client draft).
- **When** Bea presses *Confirm Order*.
- **Then** the operation is refused with *Some order lines are missing a product, you need to
  correct them before going further.* and nothing is written.

**2.5 Confirmation under a one-step policy goes straight through**

- **Given** the fixture and a draft order totalling 862.50.
- **When** Bea confirms.
- **Then** the status becomes `purchase`; the confirmation date is now; the locked flag stays
  false; a message is posted under the *RFQ Confirmed* subtype; and a receipt is created.

**2.6 An approval required above five thousand**

- **Given** the company policy is *two steps* with a double-validation amount of 5 000.00 in
  euro, and the order is in euro.
- **When** Bea confirms an order totalling 4 800.00.
- **Then** the status becomes `purchase` directly.
- **When** Bea confirms a second order totalling 5 200.00.
- **Then** the status becomes `to approve`; no confirmation date is set; no receipt is created;
  but the vendor prices have already been learned.
- **When** Bea presses *Approve Order*.
- **Then** nothing happens: she is silently filtered out because her approval test still fails.
- **When** Adam presses *Approve Order*.
- **Then** the status becomes `purchase`, the confirmation date is stamped, and the receipt is
  created.

**2.7 The threshold comparison is strict**

- **Given** the same two-step policy with a threshold of 5 000.00.
- **When** Bea confirms an order totalling exactly 5 000.00.
- **Then** the status becomes `to approve`, because the test requires the total to be strictly
  below the threshold.

**2.8 The threshold in a foreign currency**

- **Given** the two-step policy with a threshold of 5 000.00 euro, an order in United States
  dollars, and a rate on the order deadline of 1 euro = 1.0800 dollars.
- **When** Bea confirms an order totalling 5 399.99 dollars.
- **Then** the status becomes `purchase`, because 5 399.99 < 5 000.00 × 1.0800 = 5 400.00.
- **When** she confirms an order totalling 5 400.01 dollars.
- **Then** the status becomes `to approve`.

**2.9 Approval locks under the lock policy**

- **Given** the company's order modification policy is *lock*.
- **When** an order is approved.
- **Then** the locked flag becomes true, a *Locked* badge appears, the *Cancel* button
  disappears, and only Adam sees the *Unlock* button.

**2.10 Vendor price learning**

- **Given** a draft order for Wood Corner with two lines: Office Chair (Wood Corner is already a
  seller) and Assembly Service (Wood Corner is not a seller), the service line priced at 150.00
  per unit in euro with a 0 discount.
- **When** the order is confirmed.
- **Then** no new entry is created for Office Chair; and exactly one new entry is created on
  Assembly Service's template with: partner Wood Corner, sequence 1 (the product had none),
  minimum quantity 1.0, price 150.00, currency euro, discount 0, lead time 0.

**2.11 Vendor price learning on a contact address**

- **Given** the order's vendor is *Wood Corner, Springfield branch*, a contact whose parent
  company is Wood Corner; neither is a seller of the product; and the line resolved no vendor
  price.
- **When** the order is confirmed.
- **Then** the new entry is created for **Wood Corner** (the parent company), not for the
  branch.

**2.12 The ten-entry cap**

- **Given** a product that already has 11 vendor pricelist entries, and a confirmed order for a
  vendor that is not among them.
- **When** the order is confirmed.
- **Then** no new entry is created, because the cap allows learning only while the product has
  at most 10 entries.

**2.13 Price restatement when learning from a different unit**

- **Given** Office Chair's template reference unit is *Units*, the line's unit is *Dozens*
  (1 dozen = 12 units), the line price is 600.00 per dozen, and the vendor is not yet a seller.
- **When** the order is confirmed.
- **Then** the learned entry's price is 50.00, because 600.00 per dozen restated into units is
  600.00 ÷ 12.

---

## 3. Receipts

**3.1 A request for quotation of two lines becoming an order and a receipt**

- **Given** the fixture; a draft order **P00031** for Wood Corner with line 1 = 10 Office Chairs
  at 60.00 with Purchase Tax 15 %, expected the 5th; and line 2 = 1 Assembly Service at 150.00 with
  Purchase Tax 15 %, expected the 1st.
- **Then** before confirmation the untaxed amount is 750.00, the tax is 112.50, the total is
  862.50, and the header expected arrival is the 1st.
- **When** Bea sends the request and then confirms it.
- **Then** the status is `purchase`; a receipt **WH/IN/00014** exists in the ready status with
  exactly one move — 10 units of Office Chair, from the vendor location to the warehouse's
  input location, dated the 5th, unit price 60.00, linked to line 1; no move exists for the
  service; the receipt status is *Not Received*; line 1's quantity to bill is 0 and line 2's is
  1; and the billing status is *Waiting Bills*.
- **When** the inventory operator validates the receipt in full.
- **Then** line 1's received quantity becomes 10 and its quantity to bill becomes 10; the
  receipt status becomes *Fully Received*; the order's arrival date becomes the validation
  timestamp; and the order's acknowledged flag becomes true.

**3.2 An order of services creates no receipt**

- **Given** a draft order whose only line is 1 Assembly Service.
- **When** it is confirmed.
- **Then** no transfer is created; the receipt status is empty; and the billing status is
  *Waiting Bills*.

**3.3 Adding a line to a confirmed order extends the receipt**

- **Given** the confirmed order of 3.1 with its receipt still open.
- **When** Bea adds a line of 5 Office Chairs.
- **Then** a second move of 5 units is created on the same open transfer, confirmed and
  reserved, and a note *Extra line with Office Chair* is posted on the order.

**3.4 Raising a quantity on a confirmed order**

- **Given** the confirmed order of 3.1 with its receipt still open.
- **When** Bea changes line 1's quantity from 10 to 14.
- **Then** a further move of 4 units is created on the open transfer; the existing move of 10 is
  not changed; and a note recording the quantity change is posted.

**3.5 Lowering a quantity on a confirmed order raises an exception**

- **Given** the confirmed order of 3.1 with its receipt still open.
- **When** Bea changes line 1's quantity from 10 to 6.
- **Then** a warning activity is raised on the impacted transfer, rendered from the exception
  template, naming the order, stating *6.0 Units of Office Chair ordered instead of 10.0 Units*
  and listing any next impacted transfers.

**3.6 The vendor must have a supplier location**

- **Given** Wood Corner has no supplier location.
- **When** Bea confirms an order with a storable product.
- **Then** the operation is refused with *You must set a Vendor Location for this partner Wood
  Corner* and nothing is written.

**3.7 A transfer created with no move is removed**

- **Given** an order all of whose lines resolve a quantity already procured equal to the ordered
  quantity, so that no move is emitted.
- **When** the order is approved.
- **Then** the transfer that was created is deleted again and the order has no transfer.

**3.8 Changing a line's expected arrival moves the deadline**

- **Given** the confirmed order of 3.1 with its open move dated the 5th.
- **When** Bea writes the 9th as line 1's expected arrival.
- **Then** the open move's deadline becomes the 9th; the header expected arrival recomputes to
  the earliest line arrival.

**3.9 Changing a line's unit price re-prices the move**

- **Given** the confirmed order of 3.1 with its open move valued at 60.00 per unit.
- **When** Bea changes line 1's unit price to 65.00.
- **Then** the open move's unit price becomes 65.00.

---

## 4. Billing

**4.1 A partial receipt then a bill on received quantities**

- **Given** the confirmed order of 3.1, control policy *on received quantities* for Office Chair.
- **When** the operator records 6 of the 10 chairs and validates with a backorder.
- **Then** line 1's received quantity is 6; its quantity to bill is 6; the receipt status is
  *Partially Received*; a backorder transfer holds the remaining 4; a note recording the
  received-quantity change is posted.
- **When** Bea presses *Create Bill*.
- **Then** a draft vendor bill is created with vendor Wood Corner, currency euro, source
  document `P00031`, a line for Office Chair of quantity 6 at 60.00 with Purchase Tax 15 % (subtotal
  360.00), a line for Assembly Service of quantity 1 at 150.00 with Purchase Tax 15 % (subtotal 150.00),
  an untaxed total of 510.00, tax of 76.50 and a total of 586.50; and a note is posted on the
  bill reading *This vendor bill has been created from:* followed by a link to `P00031`.
- **When** the accountant posts the bill.
- **Then** line 1's billed quantity is 6 and its quantity to bill is 0; line 2's billed quantity
  is 1 and its quantity to bill is 0; the order's billing status becomes *Fully Billed*.
- **When** the operator validates the backorder for the remaining 4.
- **Then** line 1's received quantity is 10, its quantity to bill is 4, the receipt status is
  *Fully Received*, and the billing status returns to *Waiting Bills*.
- **When** Bea presses *Create Bill* again and the accountant posts it.
- **Then** the second bill carries 4 chairs at 60.00 (subtotal 240.00, tax 36.00, total 276.00)
  and a service line of quantity 0; and the order's billing status becomes *Fully Billed*.

**4.2 A bill on ordered quantities before receipt**

- **Given** the same fixture but Office Chair's control policy is *on ordered quantities*.
- **When** the order of 3.1 is confirmed.
- **Then** line 1's quantity to bill is immediately 10; the billing status is *Waiting Bills*;
  the receipt status is *Not Received*.
- **When** Bea presses *Create Bill* the same day and the accountant posts it.
- **Then** the bill carries 10 chairs at 60.00 (subtotal 600.00) and 1 service at 150.00
  (subtotal 150.00), untaxed 750.00, tax 112.50, total 862.50; line 1's billed quantity is 10
  and its quantity to bill is 0; the order's billing status is *Fully Billed* while the receipt
  status is still *Not Received*.
- **When** the goods later arrive in full.
- **Then** line 1's received quantity becomes 10 but its quantity to bill stays 0.
- **When** Bea raises line 1's quantity to 12.
- **Then** the quantity to bill becomes 2, the billing status returns to *Waiting Bills*, and 2
  more chairs are added to the receipt.
- **When** she instead lowers line 1's quantity to 8 (from 10, with 10 billed).
- **Then** the quantity to bill becomes −2; a warning activity is scheduled on the first related
  bill with the note *The quantities on your purchase order indicate less than billed. You
  should ask for a refund.*; and pressing *Create Bill* produces a document whose rounded total
  is negative, which is switched into a **vendor refund** of 2 chairs.

**4.3 A return then a refund**

- **Given** the order of 4.1 fully received (10) and fully billed (10), control policy *on
  received quantities*.
- **When** the operator returns 2 chairs to the vendor with the return flagged to be refunded,
  and validates the return.
- **Then** line 1's received quantity becomes 8 and its quantity to bill becomes −2; the order's
  billing status returns to *Waiting Bills*.
- **When** Bea presses *Create Bill*.
- **Then** the prepared document carries line 1 with quantity −2 at 60.00 and line 2 with
  quantity 0; the rounded total is −138.00, so the document is switched into a **vendor
  refund** reading 2 chairs at 60.00, untaxed 120.00, tax 18.00, total 138.00.
- **When** the accountant posts the refund.
- **Then** line 1's billed quantity becomes 8, its quantity to bill becomes 0, and the order is
  *Fully Billed* again.

**4.4 A return not flagged to be refunded changes nothing commercially**

- **Given** the same starting point as 4.3.
- **When** the operator returns 2 chairs **without** flagging the return to be refunded, and the
  return is the return of an original move.
- **Then** line 1's received quantity stays 10; its quantity to bill stays 0; the billing status
  stays *Fully Billed*; and no refund can be produced from the order.

**4.5 Grouping several orders into one bill**

- **Given** three confirmed orders: `P00040` and `P00041` for Wood Corner in euro, and `P00042`
  for Wood Corner in United States dollars, all in Northwind Supplies.
- **When** all three are selected and *Create Bill* is pressed.
- **Then** two bills are created: one carrying the lines of `P00040` and `P00041` with a source
  document of `P00040, P00041` (in the order the sets iterate), and one carrying the lines of
  `P00042`; grouping is by company, vendor and currency.

**4.6 Sections are only emitted when something follows them**

- **Given** a confirmed order whose lines in sequence order are: section *A*, product line 1,
  section *B*, section *C*, product line 2.
- **When** a bill is created.
- **Then** the bill's lines are: section *A*, product line 1, section *C*, product line 2.
  Section *B* is dropped because section *C* replaced it in the pending slot before anything was
  emitted.

**4.7 An order with only sections produces an empty bill**

- **Given** a confirmed order whose only lines are two sections.
- **When** a bill is created.
- **Then** the bill has no lines at all.

**4.8 Attachments are refused when several bills are produced**

- **Given** two confirmed orders for two different vendors.
- **When** they are selected and a bill is created with attachments supplied.
- **Then** the operation is refused with *You can only upload a bill for a single vendor at a
  time.*

**4.9 Billing status falls back when a bill is deleted**

- **Given** a confirmed order that is *Fully Billed* with one linked bill.
- **When** that bill is deleted.
- **Then** the order has no bill, every line's quantity to bill recomputes, and the billing
  status becomes *Waiting Bills* if anything is billable, otherwise *Nothing to Bill* — never
  *Fully Billed*, because that status requires a linked bill.

**4.10 A cancelled bill does not count**

- **Given** a confirmed order whose single bill of 10 chairs is posted and then cancelled.
- **Then** line 1's billed quantity returns to 0 and the quantity to bill returns to the
  received quantity (or the ordered quantity, per the policy).

**4.11 Down payments**

- **Given** a confirmed order with two product lines and no down-payment section.
- **When** a down payment of 500.00 is added through the bill-to-order assistant.
- **Then** a section line labelled *Down Payments* is created with quantity 0, the down-payment
  flag and a sequence one greater than the last line's; immediately below it a line labelled
  *Down Payment (ref: the bill line display name)* with quantity 0, the down-payment flag, the
  unit of the source bill line, a unit price of 500.00 and its taxes; and the source bill line
  now points at the created down-payment line.
- **And** confirming an order carrying such a line does not trigger the "line must have a
  product" check, because down-payment lines are exempt.

---

## 5. Cancellation, locking and deletion

**5.1 Cancelling an order with an open receipt**

- **Given** the confirmed order of 3.1 with a ready transfer.
- **When** Bea presses *Cancel*.
- **Then** the transfer is cancelled, every non-done move of every line is cancelled, and the
  order's status becomes `cancel`.

**5.2 Cancelling an order whose receipt is already done**

- **Given** the confirmed order of 3.1 with a fully validated transfer.
- **When** Bea presses *Cancel*.
- **Then** the done transfer is left alone and receives a note reading *The purchase order
  P00031 this receipt is linked to was cancelled.*; the order's status becomes `cancel`.

**5.3 A posted bill blocks cancellation**

- **Given** a confirmed order with one **posted** vendor bill.
- **When** Bea presses *Cancel*.
- **Then** the operation is refused with *Unable to cancel purchase order(s): P00031. You must
  first cancel their related vendor bills.*

**5.4 A draft bill does not block cancellation**

- **Given** a confirmed order whose only bill is a draft.
- **When** Bea presses *Cancel*.
- **Then** the order is cancelled; the draft bill is left untouched.

**5.5 A locked order cannot be cancelled**

- **Given** a confirmed, locked order.
- **When** Bea presses *Cancel* (through a client that offers the button).
- **Then** the operation is refused with *Unable to cancel purchase order(s): P00031. You must
  first unlock them.*

**5.6 Deletion requires cancellation**

- **Given** a confirmed order.
- **When** deletion is attempted.
- **Then** it is refused with *In order to delete a purchase order, you must cancel it first.*
- **When** the order is cancelled first and deletion is attempted again.
- **Then** the order and its lines are deleted.

**5.7 A line of a confirmed order cannot be deleted**

- **Given** a confirmed order with a product line and a note line.
- **When** deletion of the product line is attempted.
- **Then** it is refused with *Cannot delete a purchase order line which is in state “Purchase
  Order”.*
- **When** deletion of the note line is attempted.
- **Then** it succeeds.

**5.8 The display type cannot change**

- **Given** a note line.
- **When** an attempt is made to turn it into a section.
- **Then** it is refused with *You cannot change the type of a purchase order line. Instead you
  should delete the current line and create a new line of the proper type.*

**5.9 Resetting a confirmed order to draft unwinds nothing**

- **Given** a confirmed order with a validated receipt and a posted bill.
- **When** *Set to Draft* is applied (reaching it through the cancel-then-draft path or
  programmatically).
- **Then** the status becomes `draft`; the receipt and the bill still exist; the confirmation
  date is unchanged; the billing status becomes *Nothing to Bill*; and every line's quantity to
  bill becomes 0.

**5.10 Duplicating an order**

- **Given** the confirmed, locked, acknowledged order of 3.1, with a vendor reference, a source,
  a bill and a transfer.
- **When** Bea duplicates it.
- **Then** the copy is `draft` with a fresh reference `P00032`; no confirmation date; no arrival
  date; no vendor reference; no source; no bills; no transfers; not locked; not acknowledged;
  both lines copied; and each copied line's expected arrival recomputed from its selected vendor
  price.

---

## 6. Purchase agreements

**6.1 A blanket agreement with a fixed price used by two orders**

- **Given** the agreements capability is installed and the blanket-order series stands at 2.
- **When** Adam creates an agreement of type *Blanket Order* for Wood Corner, valid from the 1st
  to the 31st of the following quarter, with one line: Office Chair, unit *Units*, quantity
  1 000, unit price 4.25.
- **Then** its name is `BO00003`; its currency is the euro; its status is `draft`.
- **When** Adam confirms it.
- **Then** the status becomes `confirmed` and exactly one vendor pricelist entry is created:
  partner Wood Corner, product Office Chair, unit *Units*, price 4.25, currency euro, pointing
  back at the agreement line.
- **When** Bea creates a request for quotation and selects agreement `BO00003`.
- **Then** the vendor becomes Wood Corner; the currency the euro; the terms the agreement's
  description; the source contains `BO00003`; the order deadline is the later of now and the
  agreement's start date; and one line appears for Office Chair with **quantity 0** and unit
  price 4.25.
- **When** she types quantity 300 and confirms.
- **Then** the subtotal is 1 275.00 and the agreement line's ordered quantity becomes 300.
- **When** a second request is raised the same way with quantity 250 and confirmed.
- **Then** the subtotal is 1 062.50 and the agreement line's ordered quantity becomes 550.
- **When** Adam closes the agreement while no request is open.
- **Then** the status becomes `done` and the published vendor pricelist entry is deleted.
- **When** a third request had instead been left in `draft` and Adam tries to close.
- **Then** the operation is refused with *To close this purchase requisition, cancel related
  Requests for Quotation.* followed by a blank line and *Imagine the mess if someone confirms
  these duplicates: double the order, double the trouble :)*

**6.2 A purchase template copies quantities**

- **Given** an agreement of type *Purchase Template* with one line: Office Chair, quantity 25,
  unit price computed from the vendor entry as 60.00.
- **When** Bea raises a request for quotation against it.
- **Then** the order line carries **quantity 25** and unit price 60.00; no vendor pricelist
  entry was ever published.

**6.3 Confirming an empty agreement is refused**

- **Given** a draft agreement with no line.
- **When** Adam confirms it.
- **Then** it is refused with *You cannot confirm agreement 'BO00004' because it does not contain
  any product lines.*

**6.4 Confirming a blanket order with a zero price is refused**

- **Given** a draft blanket order with one line whose unit price is 0.
- **When** Adam confirms.
- **Then** it is refused with *You cannot confirm a blanket order with lines missing a price.*

**6.5 Confirming a blanket order with a zero quantity is refused**

- **Given** a draft blanket order with one line whose price is 4.25 and whose quantity is 0.
- **When** Adam confirms.
- **Then** it is refused with *You cannot confirm a blanket order with lines missing a quantity.*

**6.6 A zero price cannot be written on a confirmed blanket order**

- **Given** the confirmed agreement of 6.1.
- **When** the line's unit price is written as 0.
- **Then** it is refused with *You cannot have a negative or unit price of 0 for an already
  confirmed blanket order.*

**6.7 Changing the agreed price propagates**

- **Given** the confirmed agreement of 6.1.
- **When** Adam writes 4.50 as the line's unit price.
- **Then** the published vendor pricelist entry's price becomes 4.50, and the next request
  raised against the agreement prices the line at 4.50.

**6.8 The end date must not precede the start date**

- **Given** a draft agreement.
- **When** an end date of the 10th is written with a start date of the 20th.
- **Then** it is refused with *End date cannot be earlier than start date. Please check dates for
  agreements: BO00005*

**6.9 The type cannot change once the agreement leaves draft**

- **Given** the confirmed agreement of 6.1.
- **When** the type is written as *Purchase Template*.
- **Then** it is refused with *You cannot change the Agreement Type or Company of a not draft
  purchase agreement.*

**6.10 Changing the type of a draft agreement renumbers it**

- **Given** a draft agreement named `BO00006`.
- **When** its type is changed to *Purchase Template*.
- **Then** its name becomes the next value of the purchase-template series, for example
  `PT00002`, and both validity dates are cleared.

**6.11 Cancelling an agreement cancels its draft requests**

- **Given** the confirmed agreement of 6.1 with one `draft` request and one `purchase` order
  raised against it.
- **When** Adam cancels the agreement.
- **Then** the published entry is deleted; the draft request becomes `cancel` and receives the
  note *Cancelled by the agreement associated to this quotation.*; the confirmed order is
  untouched; and the agreement's status becomes `cancel`.

**6.12 Deleting an agreement**

- **Given** a confirmed agreement.
- **When** deletion is attempted.
- **Then** it is refused with *You can only delete draft or cancelled requisitions.*

**6.13 The duplicate blanket order warning**

- **Given** Wood Corner already has a confirmed blanket order in Northwind Supplies.
- **When** Adam selects Wood Corner on a new draft blanket order.
- **Then** a non-blocking warning appears with the title *Warning for Wood Corner* and the
  message *There is already an open blanket order for this supplier. We suggest you complete this
  open blanket order, instead of creating a new one.*, and Adam may proceed.

**6.14 An agreement's own vendor prices are exclusive**

- **Given** the confirmed agreement of 6.1 published a vendor entry for Office Chair at 4.25,
  and Wood Corner also has an ordinary entry for Office Chair at 60.00.
- **When** a line for Office Chair is priced on an order **linked to the agreement**.
- **Then** the agreement branch is used and the price is 4.25.
- **When** a line for Office Chair is priced on an order **not linked** to any agreement.
- **Then** the agreement-published entry is excluded from the candidates and the price is 60.00.

---

## 7. Calls for tenders

**7.1 A call for tenders with three alternatives where one is chosen**

- **Given** the agreements capability is installed, Bea holds the purchase-alternatives
  privilege, and a draft request `P00021` exists for vendor A with one line of 500 units.
- **When** Bea opens *Create alternative*, selects vendors B and C, leaves *Copy Products*
  ticked and confirms.
- **Then** two requests `P00022` and `P00023` are created, each with one line for the same
  product, quantity 500, in that vendor's currency; a group holds all three; and each order's
  alternatives list shows the other two.
- **When** the quotes are entered: `P00021` 9.80 per unit in euro arriving the 20th; `P00022`
  11.00 per unit in a currency whose order rate makes the company subtotal 4 800.00, arriving
  the 15th; `P00023` 9.90 per unit in euro arriving the 25th.
- **Then** the company subtotals are 4 900.00, 4 800.00 and 4 950.00; the comparison list
  highlights `P00022`'s line as best total (4 800.00), as best unit price (9.60 per unit in
  euro) and as best arrival date (the 15th).
- **When** Bea selects `P00022`'s line and presses *Choose*.
- **Then** the lines of `P00021` and `P00023` have their quantity set to 0 and both orders total
  zero.
- **When** Bea confirms `P00022`.
- **Then** the alternative warning assistant appears listing `P00021` and `P00023`.
- **When** she picks *Cancel alternatives*.
- **Then** `P00021` and `P00023` become `cancel`, and `P00022` is confirmed through the ordinary
  path: vendor prices learned, approval test applied, receipt created.
- **When** she had instead picked *Keep alternatives*.
- **Then** `P00021` and `P00023` stay `draft` with zero quantities and `P00022` is confirmed.

**7.2 Choosing a winner when a loser is already confirmed**

- **Given** a group of three alternatives one of which is already in the `purchase` status.
- **When** Bea chooses a line on another one.
- **Then** the confirmed order's line is **not** cleared and a notification appears with the
  title *Some not cleared* and the message *Some quantities were not cleared because their
  status is not a request for quotation status.*

**7.3 Choosing when there is nothing to clear**

- **Given** a group where no other order carries the chosen product with a non-zero quantity.
- **When** Bea presses *Choose*.
- **Then** a notification appears with the title *Nothing to clear* and the message *There are no
  quantities to clear.*

**7.4 A group of one deletes itself**

- **Given** a group holding exactly two orders.
- **When** one of them is removed from the other's alternatives list.
- **Then** the group is deleted and neither order belongs to a group any more.

**7.5 Ties keep every tied line**

- **Given** a group where two alternatives quote exactly the same company subtotal for the same
  product.
- **Then** both of their lines are highlighted as best total.

---

## 8. Reminders, acknowledgement and the portal

**8.1 A reminder sent two days before the planned date**

- **Given** Wood Corner is configured with the receipt reminder enabled and 2 days before
  receipt, and Bea holds the receipt-reminder privilege.
- **When** an order `P00044` is confirmed on the 1st with an expected arrival of the 10th.
- **Then** the order's reminder flag is true and its days before receipt is 2.
- **When** the daily job runs on the 1st through the 7th.
- **Then** nothing is sent, because 10 − 2 = the 8th, which is not today.
- **When** the job runs on the **8th**.
- **Then** the reminder message is posted on the order, addressed to Wood Corner, stating that
  the delivery of `P00044` is expected for the 10th and asking for confirmation, carrying an
  *Acknowledge* button pointing at the order's portal address with the acknowledgement flag, and
  attaching the purchase order document.
- **When** the job runs on the 9th.
- **Then** nothing further is sent.
- **When** the vendor presses *Acknowledge* on the 9th.
- **Then** the acknowledged flag becomes true and the order never appears in the job's selection
  again.

**8.2 Goods received before the reminder date**

- **Given** the same order.
- **When** the receipt is validated on the 6th.
- **Then** the order gains an arrival date and is excluded from the job's selection, so no
  reminder is sent on the 8th. The acknowledged flag is also set by the validation.

**8.3 An order of only services is never reminded**

- **Given** an order whose only line is 1 Assembly Service, with the reminder enabled and an
  expected arrival of the 10th.
- **When** the job runs on the send date.
- **Then** nothing is sent, because the set of line product types is exactly {service}.

**8.4 An order mixing a service and a good is reminded**

- **Given** the same order plus one line of Office Chair.
- **When** the job runs on the send date.
- **Then** the reminder is sent, because the set of line product types is {consumable, service}.

**8.5 The reminder preview**

- **Given** Bea's user record carries an email address.
- **When** she presses the preview button on an order.
- **Then** the reminder template is rendered for that order and sent immediately to **her own**
  address with no other recipients, and the interface shows the toast *A sample email has been
  sent to bea@example.com.*

**8.6 The reminder privilege gates everything**

- **Given** the receipt-reminder privilege is revoked from everyone.
- **When** the job runs, and when a user presses *Send Reminder* or the preview button.
- **Then** nothing happens in all three cases, with no error.

**8.7 The vendor updates expected arrivals through the portal**

- **Given** a confirmed order with two lines whose expected arrivals are the 10th and the 12th,
  a ready receipt, and a buyer whose time zone is 5 hours behind coordinated universal time.
- **When** the vendor opens the order's portal page in update mode and submits the 14th for the
  first line.
- **Then** an activity summarised *Date Updated* is created on the order for Bea, whose note
  reads *Wood Corner modified receipt dates for the following products:* followed by
  *- Office Chair from the 10th to the 14th* and ending with *Those dates have been updated
  accordingly on the receipt WH/IN/00014.*; the line's expected arrival becomes the 14th at
  17:00 coordinated universal time; and the open move's deadline becomes the same value.
- **When** the vendor submits a second date for the other line while the activity still exists.
- **Then** the existing activity's note is extended with a second bullet rather than a second
  activity being created, and its final sentence is rebuilt.

**8.8 The vendor updates dates when the receipt is already validated**

- **Given** the same order but with the receipt fully validated.
- **When** the vendor submits a new date.
- **Then** the line's expected arrival is **not** changed, and the activity note ends with
  *Those dates couldn’t be modified accordingly on the receipt WH/IN/00014 which had already
  been validated.*

**8.9 A malformed date is skipped**

- **Given** the same order.
- **When** the vendor submits a date that does not parse alongside a valid one.
- **Then** the invalid entry is skipped and the valid one is applied.

**8.10 A line identifier from another order redirects**

- **Given** the same order.
- **When** the submitted map contains a line identifier that does not belong to it.
- **Then** the request redirects to the order's portal address and nothing is written.

**8.11 The portal lists**

- **Given** Wood Corner's portal user, with three orders: one `sent`, one `purchase`, one
  `cancel`.
- **Then** the portal home shows a requests counter of 1 and a purchases counter of 2; the
  requests list shows the sent order; the purchases list with filter *All* shows the confirmed
  and the cancelled one; with filter *Purchase Order* only the confirmed one; with filter
  *Cancelled* only the cancelled one.

**8.12 The portal serves the right document**

- **Given** the same three orders.
- **When** each is requested with a printable report type.
- **Then** the `sent` order is served the quotation document and the other two the purchase
  order document.

---

## 9. Merging

**9.1 Two mergeable requests**

- **Given** two draft requests for Wood Corner in euro with the same operation type and no
  dropship address: `P00051` dated the 1st with a line of 30 Office Chairs at 4.10 expected the
  12th at 08:00, and `P00052` dated the 3rd with a line of 20 Office Chairs at 3.95 expected the
  12th at 23:00.
- **When** both are selected and *Merge RFQs* is pressed.
- **Then** `P00051` survives (the earliest order deadline); its line becomes 50 units at 3.95
  with a subtotal of 197.50; `P00052` is cancelled; `P00051` receives the note *RFQ merged with
  P00051 and P00052* and `P00052` receives *RFQ merged with* followed by a link to `P00051`.

**9.2 Arrivals more than a day apart do not merge**

- **Given** the same pair but `P00052`'s line is expected the 13th at 09:00 — a 25-hour gap.
- **When** they are merged.
- **Then** `P00051` ends up with two separate lines.

**9.3 Fewer than two mergeable records**

- **Given** one draft request and one confirmed order selected together.
- **When** *Merge RFQs* is pressed.
- **Then** it is refused with *Please select at least two purchase orders with state request for quotation and request for quotation
  sent to merge.*

**9.4 Records that do not share a merge key**

- **Given** two draft requests for two different vendors.
- **When** they are merged.
- **Then** it is refused with *In selected purchase order to merge these details must be
  same* followed by a line break and *Vendor, currency, destination, dropship address and
  agreement*.

**9.5 Sources and vendor references are concatenated**

- **Given** `P00051` with source *SALE-100* and vendor reference *A1*, and `P00052` with source
  *SALE-200* and vendor reference *B2*.
- **When** they merge into `P00051`.
- **Then** its source becomes *SALE-100, SALE-200* and its vendor reference becomes *A1, B2*.

---

## 10. Matching bills to orders

**10.1 Creating a bill from selected order lines**

- **Given** the matching screen shows three billable order lines of Wood Corner and no bill line
  is selected.
- **When** all three are selected and *Match* is pressed.
- **Then** a draft vendor bill is created for Wood Corner in the single shared currency, and the
  three order lines are added to it as bill lines.

**10.2 Positional matching**

- **Given** the matching screen shows, for product Office Chair, two order lines (A then B) and
  three unlinked bill lines (X, Y then Z) of the same single bill.
- **When** all five rows are selected and *Match* is pressed.
- **Then** X points at A, Y points at B, and Z — the surplus — points at **B**, the last order
  line of that product.

**10.3 Unmatched rows on a single bill**

- **Given** the matching screen shows one order line for product P, one unlinked bill line for
  product Q of a single bill, and nothing else.
- **When** both are selected and *Match* is pressed.
- **Then** no pairing is possible; the bill line for Q is **deleted**; and the order line for P
  is appended to the bill as a new line.

**10.4 No order line selected**

- **When** *Match* is pressed with only bill line rows selected.
- **Then** it is refused with *You must select at least one Purchase Order line to match or
  create bill.*

**10.5 Adding bill lines to a purchase order**

- **Given** three unlinked bill lines of Wood Corner with products, and no order is involved.
- **When** *Add to Purchase Order* is pressed, the assistant opens with the partner pre-filled
  and no order, and *Add to purchase order* is confirmed.
- **Then** a new purchase order is created for Wood Corner with three lines carrying the bill
  quantities, units, unit prices and discounts; the order is **confirmed**; and each bill line
  is linked to the purchase line built from it.

**10.6 Adding bill lines with no product**

- **Given** the selected bill lines carry no product.
- **When** *Add to purchase order* is confirmed.
- **Then** it is refused with *There are no products to add to the Purchase Order. Are these Down
  Payments?*

**10.7 Two vendors selected**

- **When** bill lines of two different commercial partners are selected and *Add to Purchase
  Order* is pressed.
- **Then** it is refused with *Please select bill lines with the same vendor.*

**10.8 Two orders involved**

- **When** the selection points at two different purchase orders.
- **Then** it is refused with *Vendor Bill lines can only be added to one Purchase Order.*

**10.9 Automatic matching, exact total**

- **Given** a confirmed order `P00061` with two lines whose remaining amounts to bill are 500.00
  and 300.00, and an incoming electronic bill quoting the reference `P00061` with a total of
  800.00.
- **When** the automatic matching runs.
- **Then** the sum 800.00 is within 0.02 of the bill total, so the result is a total match and
  the bill's lines are **replaced** by the order's lines, preceded by a section headed *From
  P00061*.

**10.10 Automatic matching, ambiguous subset**

- **Given** an order with remaining amounts 500.00, 300.00, 200.00 and 120.00, and a **scanned**
  bill quoting that order with a total of 500.01.
- **When** the automatic matching runs.
- **Then** the subset search finds both {500.00} and {300.00, 200.00}, abandons, and the result
  is an order match: the bill's lines are replaced by **all** of the order's lines with their
  full quantities.

**10.11 Automatic matching, unique subset**

- **Given** the same order and a scanned bill totalling 620.01.
- **When** the automatic matching runs.
- **Then** the unique subset {500.00, 120.00} is found; the bill keeps its own lines; the order's
  lines are appended; and the appended lines that are **not** in the subset have their quantity
  set to 0.

**10.12 Automatic matching by vendor and amount**

- **Given** an incoming bill with no recognisable reference, a recognised vendor of Wood Corner
  and a total of 862.50, and exactly one confirmed Wood Corner order whose total is 862.50.
- **When** the automatic matching runs.
- **Then** the result is a total match on that order.
- **And** when **two** Wood Corner orders both total 862.50, the result is no match and the bill
  keeps its own lines.

**10.13 Auto-completing a bill from an order**

- **Given** a draft vendor bill with no lines and a confirmed order `P00031`.
- **When** the accountant picks `P00031` in the *Auto-complete* control.
- **Then** the bill's header takes the order's vendor, currency, payment terms, fiscal position,
  bank account and narration; the order's lines are appended; the source document becomes
  `P00031`; and both transient controls are cleared.
- **When** the bill already had product lines in a different currency.
- **Then** the bill keeps its own currency and the order's lines are appended at that currency's
  restated unit prices.

---

## 11. Procurement-driven purchasing

**11.1 A reordering rule creates a request for quotation**

- **Given** inventory is installed, Office Chair has a reordering rule with a minimum of 10 and a
  maximum of 50 in the warehouse, on-hand is 0, and the buy route is active. The vendor's lead
  time is 4 days and the company's days-to-purchase is 2.
- **When** the scheduler runs on the 1st.
- **Then** a draft purchase order is created for Wood Corner, with the vendor's assigned buyer,
  the warehouse's incoming operation type, the euro, a source naming the reordering rule, and
  one line of 50 units priced from the vendor entry, whose expected arrival is the rule's
  requested date and whose reordering rule is recorded.

**11.2 No vendor found, from a reordering rule**

- **Given** the same rule but the product has no vendor at all.
- **When** the scheduler runs.
- **Then** the run raises *There is no matching vendor price to generate the purchase order for
  product Office Chair (no vendor defined, minimum quantity not reached, dates not valid, ...).
  Go on the product form and complete the list of vendors.*

**11.3 No vendor found, from a sales order**

- **Given** a make-to-order sales line for a product with no vendor.
- **When** the procurement runs.
- **Then** no error is raised; the waiting move is cancelled when it propagates cancellation or
  switched to make-to-stock otherwise; and the responsible person is notified.

**11.4 Two needs merge onto one line**

- **Given** two procurements for the same product, unit, cancellation flag, description and
  reordering rule, for 20 and 30 units.
- **When** the buy action runs.
- **Then** one line of 50 units is created, and its unit price is re-selected for the combined
  quantity of 50.

**11.5 A second need extends an existing line**

- **Given** a draft order for Wood Corner already carrying a line of 20 Office Chairs at 12.00,
  and a new procurement for 80 more with a vendor price break at 100 at 10.50.
- **When** the buy action runs.
- **Then** the line's quantity becomes 100 and its unit price becomes 10.50, because the vendor
  price is re-selected for the combined quantity.

**11.6 Daily grouping**

- **Given** Wood Corner's grouping policy is *Daily*, and two procurements requested for the same
  calendar day.
- **When** the buy action runs.
- **Then** both land on the same draft order.
- **And** when they are requested for different calendar days, two orders are created.

**11.7 Weekly grouping with a target week day**

- **Given** Wood Corner's grouping policy is *Weekly* with the target week day Friday, and a
  procurement requested for Wednesday the 14th.
- **When** the buy action runs.
- **Then** the accepted window is Friday the 16th; the created line's expected arrival is pushed
  to the 16th; and, when the order's own expected arrival is not earlier, the order deadline is
  pushed forward by 2 days.

**11.8 The order deadline is pulled back for a new line**

- **Given** an existing draft order dated the 10th, and a new line whose expected arrival is the
  14th for a vendor whose lead time is 7 days.
- **When** the line is created.
- **Then** 14 − 7 = the 7th, which is before the 10th, so the order deadline is moved back to the
  7th.

**11.9 Cancelling a procurement-driven order**

- **Given** a confirmed order created by a make-to-order procurement, with a downstream move
  waiting on it and cancellation propagation enabled.
- **When** the order is cancelled.
- **Then** the receipt and its moves are cancelled and the downstream move is cancelled too.
- **And** when propagation is disabled, the downstream move is switched to make-to-stock and its
  status recomputed instead.

---

## 12. Services purchased from a sales order

**12.1 A sold service generates a request for quotation**

- **Given** the sales-purchase bridge is installed; Assembly Service is a service with the
  subcontract flag set and one vendor pricelist entry for Wood Corner at 90.00 with a lead time
  of 3 days; and a sales order with a commitment date of the 20th carries 4 units of it.
- **When** the sales order is confirmed.
- **Then** a draft purchase order is created for Wood Corner with an order deadline of the 17th
  (the commitment date minus the 3-day lead time), a source naming the sales order, and one line
  of 4 units at 90.00 whose expected arrival is the 20th and which points back at the sales line.

**12.2 A second sold service joins the same order**

- **Given** the same sales order also carries another subcontracted service from Wood Corner.
- **Then** both purchase lines land on the same draft purchase order, because the search matches
  a draft purchase line for the same partner already serving that sales order.

**12.3 Raising the sold quantity while the purchase is still a draft**

- **Given** the purchase order of 12.1 is still `draft`.
- **When** the sold quantity rises from 4 to 6.
- **Then** the existing purchase line's quantity becomes 6; no new line is created.

**12.4 Raising the sold quantity after the purchase is confirmed**

- **Given** the purchase order of 12.1 has been confirmed.
- **When** the sold quantity rises from 4 to 6.
- **Then** a **new** purchase line for the difference of 2 is generated, following the whole
  generation path again.

**12.5 Lowering the sold quantity**

- **Given** the purchase order of 12.1 exists.
- **When** the sold quantity falls from 4 to 2, with 0 delivered.
- **Then** no purchase line changes; a warning activity is raised on the purchase order listing
  the sales line, its order and its previous quantity; and the seller sees the immediate warning
  *Ordered quantity decreased!* with the message *You are decreasing the ordered quantity! Do not
  forget to manually update the purchase order if needed.*

**12.6 No vendor for a subcontracted service**

- **Given** Assembly Service has the subcontract flag but the vendor entry has been deleted.
- **When** the sales order is confirmed.
- **Then** it is refused with *There is no vendor associated to the product Assembly Service.
  Please define a vendor for this product.*

**12.7 The subcontract flag is guarded**

- **When** the flag is set on a storable product.
- **Then** it is refused with *Product that is not a service can not create RFQ.*
- **When** the flag is set on a service with no vendor.
- **Then** it is refused with *Please define the vendor from whom you would like to purchase this
  service automatically.*

**12.8 Cancelling the purchase warns the sales order**

- **Given** the purchase order of 12.1.
- **When** it is cancelled.
- **Then** a warning activity is raised on the originating sales order listing the cancelled
  purchase order and its lines.

---

## 13. Kits

**13.1 A kit is received in components**

- **Given** the manufacturing bridge is installed and *Desk Set* is a kit of 1 desk and 2
  drawers; a confirmed purchase order carries 5 Desk Sets.
- **When** the receipt is prepared.
- **Then** the receipt carries component moves — 5 desks and 10 drawers — not a move of Desk Set.
- **When** 5 desks and 8 drawers are received.
- **Then** the purchase line's received quantity is min(5 ÷ 1, 8 ÷ 2) = **4** kits.
- **When** 2 more drawers arrive.
- **Then** the received quantity becomes 5 kits.

**13.2 Kit cost shares must total 100**

- **Given** a kit whose components carry cost shares of 70 and 20.
- **When** the structure is saved.
- **Then** it is refused with *The total cost share for a BoM's component have to be 100*.
- **And** a negative cost share is refused with *Components cost share have to be positive or
  equals to zero.*

**13.3 Cost shares split the billed value**

- **Given** the kit of 13.1 with cost shares 70 % on the desk and 30 % on the drawers, and a
  posted bill of 300.00 for 10 kits.
- **Then** the desk moves are valued at 210.00 in total and the drawer moves at 90.00.

---

## 14. Multi-currency, multi-company and rounding

**14.1 Order amounts in a foreign currency**

- **Given** Wood Corner's supplier currency is United States dollars and the rate on the order
  deadline is 1 euro = 1.25 dollars.
- **When** an order is created with one line of 10 units at 120.00 dollars, no tax.
- **Then** the order currency is dollars; the currency rate is 1.25; the untaxed amount is
  1 200.00 dollars; and the total in company currency is 960.00 euro.

**14.2 The value handed to the stock move**

- **Given** the same order, with 1 box = 12 units, the line priced at 120.00 dollars per box with
  a 5 % discount and a fully recoverable 20 % tax, and the same rate of 1.25.
- **Then** the move unit price is 7.60 euro per unit: 120.00 × 0.95 = 114.00 per box; excluding
  tax still 114.00; per unit 9.50; divided by 1.25 gives 7.60; rounded to 2 digits, 7.60.

**14.3 Comparing offers in two currencies**

- **Given** an offer of 500 units at 11.00 dollars, with an order rate of 1.1458333….
- **Then** the line subtotal is 5 500.00 dollars and the company subtotal is 4 800.00 euro.

**14.4 Global tax rounding**

- **Given** the company's tax rounding method is global rather than per line, and the two lines
  of scenario 1.9.
- **Then** the tax amount is computed once on the summed base: (93.31 + 134.973) × 0.15 =
  34.24245 → 34.24, which differs by 0.01 from the per-line result of 34.25.

**14.5 Products of another company are refused**

- **Given** a second company *Southgate Ltd* whose products are not in Northwind Supplies'
  accessible branch tree, and a product *Southgate Desk* belonging to it.
- **When** a Northwind Supplies order is saved with a line for *Southgate Desk*.
- **Then** it is refused with *Your quotation contains products from company Southgate Ltd whereas
  your quotation belongs to company Northwind Supplies. Please change the company of your
  quotation or remove the products from other companies (Southgate Desk).*

**14.6 Record rules hide other companies' orders**

- **Given** an order of Southgate Ltd and a reader whose allowed companies are Northwind Supplies
  only.
- **Then** the order is invisible to that reader in every list, search and report.

**14.7 A portal user sees only their own orders**

- **Given** Wood Corner's portal user and an order for a different vendor.
- **Then** the order is invisible and the portal page redirects to the portal home.

---

## 15. Reporting

**15.1 Purchase analysis quantities are in the product unit**

- **Given** a confirmed order with one line of 10 boxes of a product whose reference unit is
  *Units* and where 1 box = 12 units, received 6 boxes and billed 4 boxes.
- **Then** the analysis row shows a quantity ordered of 120, a quantity received of 72 and a
  quantity billed of 48, with the reference unit *Units*.
- **And** with the control policy *on received quantities*, the quantity to be billed is
  72 − 48 = 24.

**15.2 The average price is re-weighted when grouped**

- **Given** two analysis rows for the same vendor: 100 units averaging 10.00 and 5 units
  averaging 30.00.
- **When** they are grouped by vendor.
- **Then** the displayed average is 10.95, not 20.00.

**15.3 Days to confirm and days to receive**

- **Given** an order deadline of the 1st at 00:00, a confirmation date of the 3rd at 12:00 and a
  line expected the 8th at 00:00.
- **Then** the row shows 2.50 days to confirm and 7.00 days to receive.

**15.4 The on-time delivery rate**

- **Given** a vendor with three qualifying lines over the window: 100, 50 and 25 units ordered,
  with 100, 50 and 0 units delivered on time.
- **Then** the rate is 150 ÷ 175 × 100 = 85.71 %.
- **And** a vendor with no qualifying line has a rate of −1, which the interface presents as "no
  data".

**15.5 The weighted vendor delay rate**

- **Given** two vendor delay rows for the same vendor: total 100 with 80 on time, and total 10
  with 10 on time.
- **When** they are grouped.
- **Then** the rate is (80 + 10) ÷ (100 + 10) × 100 = 81.82 %, not the plain average of the two
  row rates.

**15.6 The dashboard days-to-order figure**

- **Given** three orders confirmed 2 days, 5 days and 0.5 days after creation, all within the
  last three months.
- **Then** the global days-to-order figure is 2.50.

**15.7 The dashboard on-time figure**

- **Given** four confirmed orders whose expected arrivals fall in the last three months, of which
  three have an arrival date on or before their expected arrival.
- **Then** the on-time figure is 75 %.
- **And** with no such order at all the figure is 100 %.

---

## 16. Replenishment suggestions

**16.1 A suggestion from thirty-day demand**

- **Given** the basis *thirty days*, a horizon of 7 days, a percentage of 100 %, demand of 260
  units over the last 30 days, on-hand 20 and incoming 15.
- **Then** the monthly demand is 260; the monthly ratio is 7 ÷ 30.4375 = 0.229938…; the raw need
  is 59.784 − 20 − 15 = 24.784; and the suggested quantity is 25.

**16.2 A suggestion that rounds to zero**

- **Given** the same figures but a percentage of 50 %.
- **Then** the raw need is 29.892 − 35 = −5.108 and the suggested quantity is 0.

**16.3 A suggestion from actual demand**

- **Given** the basis *actual demand*, a percentage of 100 %, and a forecast quantity at the
  horizon of −12.4.
- **Then** the suggested quantity is round-up(12.4) = 13.
- **And** with a non-negative forecast the suggestion is 0.

**16.4 Applying the suggestion collapses existing lines**

- **Given** an order carrying three lines for the same product outside any section, and a
  suggested quantity of 25.
- **When** the suggestion is applied.
- **Then** two of the three lines are deleted and the survivor is overwritten with the suggested
  quantity, unit and price.
- **And** when the suggested quantity is 0, all three are deleted.

**16.5 The suggestion parameters are remembered**

- **When** the suggestion is applied with a basis, a horizon and a percentage.
- **Then** those three values are written onto the vendor partner with elevated rights and are
  the defaults the next time the panel opens for that vendor.

---

## 17. The accrual entry

**17.1 Goods received not billed**

- **Given** a confirmed order of 10 units at 100.00, no tax, control policy *on received
  quantities*, 6 received and 0 billed at the 31st; the product's expense account is 600000; the
  chosen accrual account is 480000; the reversal date is the 1st of the next month.
- **When** the accrual entry is created.
- **Then** the posted entry debits 600000 by 600.00 with the label *P00031 - Office Chair; 0.0
  Billed, 6.0 Received at € 100.00 each* and credits 480000 by 600.00 with the label *Accrued
  total*; and a reversing entry dated the 1st mirrors it.

**17.2 Billed not received**

- **Given** the same order with 6 received and 8 billed at the 31st, and posted bill subtotals
  totalling 800.00.
- **Then** the over-billed quantity is 2, which is at least 1, so the unit price is recomputed as
  (800.00 − 600.00) ÷ 2 = 100.00; the quantity to bill at date is −2; the per-line amount is
  −200.00; 600000 is **credited** 200.00 and 480000 **debited** 200.00.

**17.3 Guards**

- **When** the reversal date is not strictly after the date.
- **Then** it is refused with *Reversal date must be posterior to date.*
- **When** the selection spans two companies.
- **Then** it is refused with *Entries can only be created for a single company at a time.*
- **When** the selection spans two currencies.
- **Then** it is refused with *Cannot create an accrual entry with orders in different
  currencies.*

**17.4 The analytic blend on the balancing item**

- **Given** two lines: one of 600.00 including tax distributed 100 % to analytic account A, and
  one of 400.00 including tax distributed 100 % to analytic account B, on a single order of
  1 000.00.
- **Then** the balancing item's analytic distribution is 60 % to A and 40 % to B.

---

## 18. The price difference

**18.1 A positive price difference**

- **Given** the company uses the recognition style in which stock is debited at bill time, the
  product's cost method is the standard-price method with a cost of 9.00, and a bill invoices 10
  units at 10.00 with no discount and no tax.
- **When** the bill is posted.
- **Then** two extra items are produced: a debit of 10.00 to the category's price-difference
  account and a credit of 10.00 to the bill line's own account, both labelled with the first 64
  characters of the line's label and carrying the line's analytic distribution.

**18.2 A discount suppresses the adjustment**

- **Given** the same bill but with a 10 % discount on the line.
- **Then** the stored unit price and the computed unit price no longer agree at the Product Price
  precision, so no adjustment is produced.

**18.3 A refund reverses the adjustment**

- **Given** the same figures on a vendor refund.
- **Then** the valuation price is negated before the subtraction and the sign of the whole
  adjustment is reversed.

---

## 19. Warnings

**19.1 The vendor warning**

- **Given** Wood Corner carries the purchase warning *Always confirm lead times*, and the reader
  holds the purchase-warnings privilege.
- **When** an order for Wood Corner is opened.
- **Then** the banner reads *Wood Corner - Always confirm lead times*.

**19.2 The parent company's warning**

- **Given** the order's vendor is a contact whose parent company carries a purchase warning.
- **Then** the banner also carries the parent's name and message.

**19.3 A product warning**

- **Given** Office Chair carries the purchase line warning *Check assembly instructions*.
- **Then** the banner also carries *Office Chair - Check assembly instructions*, on its own line.

**19.4 Duplicates are removed and the privilege gates everything**

- **Given** two lines for the same product with the same warning.
- **Then** the message appears once.
- **And** when the privilege is revoked, the banner is empty everywhere, including on bills and
  in the alternative-creation assistant.

**19.5 The duplicate order warning**

- **Given** a draft order for Wood Corner whose vendor reference is *A1*, and another
  non-cancelled Wood Corner order of the same company whose vendor reference is also *A1*.
- **Then** the draft order shows the duplicate banner listing the other order.
- **And** once the draft order is confirmed, the banner disappears, because only draft orders
  compute duplicates.

---

## 20. Product grids and structured documents

**20.1 A grid creates one line per non-empty cell**

- **Given** the grid capability is installed and a product template with two colours and three
  sizes.
- **When** the buyer fills four cells with quantities on a draft order.
- **Then** four purchase order lines are created, one per variant, each with the variant's
  attribute values and the typed quantity, and their prices are computed as for any other line.

**20.2 A grid refuses to change a product on two lines**

- **Given** an order already carrying two lines for the same variant.
- **When** the grid tries to change that variant's quantity.
- **Then** it is refused with *You cannot change the quantity of a product present in multiple
  purchase lines.*

**20.3 A grid cell set to zero**

- **Given** a draft or sent order with a line produced by a grid cell.
- **When** the cell is set to 0.
- **Then** the line is removed.
- **And** when the order is already confirmed, the line's quantity is set to 0 instead.

**20.4 Exporting a structured order**

- **Given** the structured order capability is installed and a confirmed order.
- **When** the purchase order document is produced for that single record.
- **Then** the produced file carries one embedded machine-readable attachment of content type
  `text/xml`, built from the order's non-display lines, with every line unit price forced to be
  non-negative.
- **And** the same content is served by the portal's download route with a content-disposition
  naming the file.

**20.5 Importing a structured order**

- **Given** an incoming file whose customisation identifier is exactly
  `urn:fdc:peppol.eu:poacc:trns:order:3`.
- **When** it is processed.
- **Then** it is recognised as a structured order document and a purchase order is built from it.
- **And** when some information cannot be mapped, a to-do activity is raised on the order whose
  note begins *Some information could not be imported:*.

**20.6 Grouping bill lines by tax is refused on a matched bill**

- **Given** a vendor bill at least one of whose lines points at a purchase order line.
- **When** grouping or ungrouping the lines by tax is attempted.
- **Then** it is refused with *You can only (un)group lines of an invoice not linked to a purchase
  order*.

---

## 21. Rounding edge cases

**21.1 A quantity to bill that is not exactly zero**

- **Given** a line whose ordered quantity is 10 and whose billed quantity is 9.999.
- **Then** the quantity to bill is 0.001, which **is** zero at the 2-digit product-unit
  precision, so the billing status may become *Fully Billed*.

**21.2 A quantity to bill of one hundredth**

- **Given** a line whose ordered quantity is 10 and whose billed quantity is 9.99.
- **Then** the quantity to bill is 0.01, which is **not** zero at 2 digits, so the billing status
  stays *Waiting Bills* and a bill of 0.01 units may be produced.

**21.3 A generated document whose total is exactly zero**

- **Given** an order whose every line has a quantity to bill of zero.
- **When** a bill is created.
- **Then** the document total is 0.00, which is not strictly negative, so the document stays a
  vendor bill rather than being switched to a refund.

**21.4 A move unit price rounding**

- **Given** a discounted price per product unit of 7.6049 in the company currency.
- **Then** the move unit price is 7.60.
- **And** with 7.6050 it is 7.61, because rounding is half away from zero.

**21.5 A unit conversion rounding half up**

- **Given** a received move of 7 units of a product whose line unit is a box of 12.
- **Then** the contribution to the received quantity is 7 ÷ 12 = 0.5833… rounded half up to the
  line unit's own rounding.

---

## 22. Access and privileges

**22.1 A purchase user may work with vendor bills only**

- **Given** Bea holds only the purchase user privilege.
- **Then** she may read, create, update and delete journal entries whose type is a vendor bill, a
  vendor refund or a purchase receipt, and nothing else in the ledger; she may read and update
  journal items of such documents but not delete them.

**22.2 A purchase administrator may manage vendor pricelists**

- **Given** Adam holds the purchase administrator privilege.
- **Then** he may create, read, update and delete vendor pricelist entries and pricelist rules;
  Bea may not.

**22.3 An inventory user may read orders**

- **Given** a user holding only the inventory user privilege.
- **Then** they may read purchase orders and purchase order lines but not change them.

**22.4 A portal user may not create an order**

- **Given** Wood Corner's portal user.
- **Then** they may read, update and delete their own orders through the portal rule, but not
  create one.

**22.5 The dashboard refuses a non-internal caller**

- **Given** a portal user.
- **When** the dashboard operation is called.
- **Then** access is denied.

**22.6 Agreements are readable but not writable by the administrator row alone**

- **Given** a user holding only the purchase administrator privilege and **not** the purchase
  user privilege (an unusual configuration, since the administrator privilege normally implies
  it).
- **Then** they may read purchase agreements and agreement lines but not create, update or delete
  them.
