# Sales — Workflows

End-to-end operational procedures, expressed as numbered steps with preconditions, postconditions
and failure conditions. The two critical procedures are the **confirmation algorithm** (section 4)
and the **invoicing algorithm** (section 8); everything else supports them.

Roles used below:

| Role | Meaning |
|---|---|
| Salesperson | A user in the group "User: Own Documents Only" or "User: All Documents" of the sales organisation. |
| Sales Manager | A user in the group "Administrator" of the sales organisation. |
| Invoicing user | A user in the invoicing or accounting group of the financial domain. |
| Customer | A portal user, or an anonymous visitor holding the access token of the document. |
| System | An unattended process: a scheduled job, a payment post-processing, an incoming message. |

---

## 1. Creating a quotation

**Performed by**: Salesperson. **Precondition**: at least one customer record exists.

1. A new record is opened. The company defaults to the session company; the order date defaults to
   the current instant; the status is *Quotation*; the reference is the placeholder word `New`.
2. The salesperson chooses the customer. This one choice triggers, in this order, the computation
   of: the invoice address (the customer's invoicing contact), the delivery address (the customer's
   delivery contact), the fiscal position (from customer, delivery address and company), the
   payment term (the customer's sale payment term), the preferred payment method line, the price
   list (the customer's price list), the currency (the price list's currency, else the company
   currency), the currency rate (for the order date), the salesperson, the sales team and the terms
   text.
   - The salesperson is only computed when the order is unsaved or has no salesperson yet, and is
     then the customer's own salesperson, else the commercial parent's salesperson, else the
     current user when that user is a salesperson.
   - The sales team is derived from the salesperson by the team-selection heuristic of section 14.
3. If the company declares a default quotation template — or if the salesperson selects one — the
   template is applied: every existing line is removed and one line is created per template line
   (display type, product, quantity, unit, optional flag, sequence, and the template description
   when present). The first created line receives the sequence −99. The template also overrides the
   terms text, the signature requirement, the payment requirement, the prepayment percentage, the
   expiration date and the invoicing journal.
   - If the customer is changed afterwards while the order is still unsaved and while the lines
     still match the template exactly (same product, unit, quantity and display type, line by
     line), the template is re-applied so that translated descriptions follow the new customer's
     language.
4. Lines are added. For each line the salesperson picks a product; the description, the unit, the
   taxes, the unit price and the discount are computed as specified in
   [calculations.md](calculations.md). A configurable product opens the configurator; a combo
   product opens the combo configurator and produces one item line per choice.
5. Sections, subsections and notes may be inserted; sections may be marked "collapse prices",
   "collapse composition" or "optional".
6. The record is saved. At that moment the reference is drawn from the order sequence, in the
   order's company, dated with the order date expressed in the reader's time zone.

**Postcondition**: a numbered quotation in status *Quotation*, with computed totals and a validity
date.

**Failure conditions**:

- Clearing the company raises "The company is required, please select one before making any other
  changes to the sale order."
- A product belonging to another company raises the mixed-company message of
  [business-rules.md](business-rules.md).
- A combo line whose selected item count does not match the number of choices raises "The number of
  selected combo items must match the number of available combo choices."

---

## 2. Revising prices and taxes after a change of context

**Performed by**: Salesperson. **Precondition**: the order has lines.

### 2.1 Update prices

Triggered by the "Update Prices" operation, normally offered after the price list changed.

1. Collect the lines to recompute: every line that is not a display line.
2. Invalidate the cached price-list rule of those lines.
3. Recompute the unit price with the *force recomputation* instruction, which disables the
   manual-price suppression rule, but **not** the suppression caused by an already-invoiced
   quantity nor by an at-cost expense line.
4. Set the discount of those lines to zero, then recompute the discount. Resetting first is
   deliberate: a rule that no longer grants a discount must clear the old one.
5. Lower the "price list changed" screen flag.
6. Post a note on the thread: "Product prices have been recomputed according to pricelist *link to
   the price list*." — or "Product prices have been recomputed." when the order has no price list.

### 2.2 Update taxes

Triggered by the "Update Taxes" operation, normally offered after the fiscal position changed.

1. Recompute the tax set of every line that is not a display line.
2. Lower the "fiscal position changed" screen flag.
3. Post a note: "Product taxes have been recomputed according to fiscal position *link to the
   fiscal position*." — with an empty link when there is no fiscal position.

---

## 3. Sending the quotation to the customer

**Performed by**: Salesperson. **Precondition**: status *Quotation* or *Quotation Sent*.

1. The analytic distribution of every line of every order in status *Quotation* or *Quotation Sent*
   is validated. An invalid distribution aborts the operation.
2. A message composer opens, pre-filled with:
   - the order entity and the selected records;
   - composition mode *comment* for a single order, *mass mail* for several;
   - the notification layout that carries the responsible person's signature;
   - the instruction that the footer may be shown;
   - the instruction to hide the template-management options.
3. For a single order the composer additionally forces email sending and, unless the default
   template is explicitly suppressed, loads the template chosen by the template-selection rule
   (section 3.1) together with the "mark as sent" instruction. When the default template is
   suppressed, an access token is created for each order instead.
4. When the reader is an administrator, the company has no external document layout configured and
   the layout check was requested, a document-layout configuration dialogue is inserted in front of
   the composer.
5. Sending posts the message. Because the "mark as sent" instruction is active, every selected
   order still in status *Quotation* moves to *Quotation Sent* with change tracking suppressed.

### 3.1 Template-selection rule

```
if the pro-forma instruction is active      → the pro-forma quotation template
else if the status is not 'sale'            → the quotation template
else                                        → the confirmation template (section 3.2)
```

### 3.2 Confirmation-template rule

```
if the system parameter 'sale.default_confirmation_template' names an existing template
                                            → that template
else                                        → the shipped order-confirmation template
```

With the quotation-template capability installed, a quotation template that names its own
confirmation message overrides both.

### 3.3 Marking as sent without an email

The "Mark as Sent" operation writes the status directly. Every selected order must be in status
*Quotation*, otherwise the whole operation fails with "Only draft orders can be marked as sent
directly."

---

## 4. Confirming the order — the confirmation algorithm

**Performed by**: Salesperson (back office), Customer (portal signature or payment) or System
(payment post-processing). **Precondition**: status *Quotation* or *Quotation Sent*.

This is the single algorithm through which every confirmation passes, whatever triggered it.

### 4.1 Phase A — guards

For each selected order, in order:

1. **Status guard.** The status must be `draft` or `sent`. Otherwise the entire operation fails
   with "Some orders are not in a state requiring confirmation."
2. **Product guard.** Every line that is neither a display line nor an advance-invoice line must
   carry a product. Otherwise the entire operation fails with "Some order lines are missing a
   product, you need to correct them before going further."

A failure in phase A leaves every selected order untouched.

### 4.2 Phase B — analytic validation

3. For every line of every selected order whose status is `draft` or `sent` and which is not a
   display line, the analytic distribution is validated against the applicability rules of the
   analytic domain, using the business domain *sale order*, the line's product and the line's
   company. An invalid or missing mandatory distribution aborts the confirmation with the analytic
   domain's message.

### 4.3 Phase C — writing the confirmation values

4. Every selected order is written with:
   - status `sale`;
   - order date = the current instant.

   This single write is what makes the order date mean "confirmation date" from now on. It also
   triggers, through ordinary recomputation: the per-line status mirror, the quantity to invoice,
   the per-line invoice status, the order invoice status, and the expected date.
5. Change tracking posts a status note on the thread with the subtype "Sales Order Confirmed"; the
   same note is mirrored on the sales team's thread.

### 4.4 Phase D — context cleanup

6. Two context values that may have leaked from the calling screen are removed before any
   downstream record is created: the default record name and the default salesperson. Leaving them
   in place would give generated documents the wrong name or the wrong responsible person.

### 4.5 Phase E — downstream document creation

7. The downstream-creation extension point runs with the cleaned context. Every coupling contributes
   one step; the order below is the order in which the couplings wrap each other, from the
   outermost (running first) to the innermost.

   **E1 — Pickup location materialisation** (delivery coupling). For each order that carries pickup
   location data: build or reuse a delivery-type contact under the customer with the pickup
   address (street, city, postal code, state, country, telephone, electronic mail address) marked
   as a pickup location, and set it as the order's delivery address, with the instruction that
   existing transfers must follow the new address.

   **E2 — Service document generation** (project coupling). Skipped when the context asks to
   disable task generation. For each order (grouped by company so that each generation runs in the
   right company) the service lines are processed with elevated rights:

   a. Select the service lines whose service tracking is *project only*, *task in project* or
      *task in a global project*, excluding optional lines whose quantity is zero.
   b. For the lines that need a **new project** (*project only* and *task in project*), sorted by
      sequence then identifier:
      - if the line already has a project, reuse it;
      - otherwise, if no project has yet been created for this order with the same project template
        (or with no template), create one. The project's analytic account is the order's project
        analytic account when it has one, otherwise a new analytic account created with: name = the
        order reference (prefixed when a prefix is supplied, as "*prefix*: *order reference*"),
        code = the customer reference, company = the order's company, plan = the root project plan,
        customer = the order's customer;
      - if the order has no project yet, the newly created project becomes the order's project;
      - otherwise, attach the line to the project already created for this order (matching on the
        project template when the product declares one).
   c. For a *task in project* line without a task, and whose product's task template has not
      already been used in this run, create the task in that project and post on it: "This task has
      been created from: *link to the order* (*product name*)".
   d. Handle milestones: when the product's service policy is *delivered milestones*, enable
      milestones on the project if needed; attach the project's unassigned milestones to this line,
      splitting the ordered quantity equally between them; if there is none, create one milestone
      named after the line with a quantity percentage of one, and point the generated task at it.
   e. For *task in a global project* lines, sorted by sequence then identifier: if the order has no
      project, adopt the product's global project. Then, for each such line without a task, create
      the task in the product's global project (or in the order's project), provided the ordered
      quantity is strictly positive and the product's task template has not already been used.
      When no project can be found the confirmation fails with: "A project must be defined on the
      quotation *order reference* or on the form of products creating a task on order. The
      following product need a project in which to put its task: *product name*".
   f. If the order ends with exactly one project and that project came from a template, the
      project's company is aligned with the template's company.

   **E3 — Procurement launch** (inventory coupling). For every line, with the line's company
   active:
   - skip the line when the order status is not `sale`, or the order is locked, or the product is
     not a goods product;
   - compute the quantity already procured (section 4.6); skip the line when it already equals the
     ordered quantity at the `Product Unit` precision;
   - create the order's procurement reference when the order has none;
   - build the procurement values (section 4.7);
   - the quantity to procure is the ordered quantity minus the quantity already procured, adjusted
     between the line unit and the product's reference unit;
   - collect one procurement request per line: product, quantity, unit, final location (the
     delivery address's customer location), the product display name, the order reference, the
     order's company and the values.

   All collected requests are then run at once by the replenishment engine, which matches a rule
   and executes a pull, a buy or a manufacture action. Finally, every transfer of the order that is
   neither cancelled nor completed is confirmed, which schedules it.

   **E4 — Service-to-purchase generation** (purchasing coupling). With elevated rights, for every
   line whose product declares "buy on sale" and which has not already generated a purchase line:
   - find the vendor: the first seller of the product matching the requested quantity and unit; if
     there is none, fail with "There is no vendor associated to the product *product name*. Please
     define a vendor for this product.";
   - find or create a draft purchase request for that vendor, in the line's company, additionally
     restricted to purchase lines already linked to the same order;
   - when creating one, its values are: vendor, vendor reference, company, currency (the vendor's
     purchase currency, else the company currency), no destination address, source document = the
     order reference, vendor payment term, order date = the promised delivery date (or now) minus
     the vendor's delay in days, and the vendor's fiscal position;
   - append the order reference to the purchase request's source-document list when it is not
     already there;
   - create one purchase line with: the product, the quantity converted from the line unit to the
     vendor's unit, the vendor's unit, the vendor price corrected for tax inclusion and converted
     into the purchase currency at today's rate, the planned date = the purchase order date plus
     the vendor's delay, the mapped vendor taxes, the vendor's discount, the label built from the
     product display name plus the purchase description plus the line's variant suffix, a link back
     to this order line, and the line's analytic distribution when it has one.

   **E5 — Loyalty settlement** (promotions coupling). Before the status is even written, the
   promotions coupling verifies that no coupon would end with a negative point balance — failing
   with "One or more rewards on the sale order is invalid. Please check them." — then updates the
   programs and rewards, records the loyalty history lines, and deletes the coupons of *current*
   programs that claim no reward.

   **E6 — Opportunity revenue update** (customer-relationship coupling). After confirmation, the
   linked opportunity's revenue figures are refreshed from the order.

   **E7 — Event registration confirmation** (events coupling). Registrations sold by the order are
   confirmed.

   **E8 — External fulfilment** (print-on-demand coupling). The order is transmitted to the
   external production service.

### 4.6 Quantity already procured

```formula
already_procured = Σ over outgoing moves of convert( move quantity when completed,
                                                     else move demanded quantity ,
                                                     from = move unit , to = line unit ,
                                                     rounding = half up )
                 − Σ over incoming moves of the same converted quantity
```

The move classification is the *non-strict* one: a move counts as outgoing when it was created by
the rule that started this line's chain and its final (or destination) location is an outgoing
location. This makes multi-step delivery routes count once, not once per step.

### 4.7 Procurement values produced per line

| Procurement value | Source |
|---|---|
| Source document | the order reference |
| References | the order's procurement references |
| Originating order line | this line |
| Planned date | the deadline minus the company's security lead time, in days |
| Deadline | the order's promised delivery date, else the line's expected date |
| Routes | the line's routes |
| Warehouse | the line's warehouse |
| Partner | the order's delivery address |
| Final location | the delivery address's customer location |
| Product description variants | the line's variant suffix, in the customer's language, trimmed |
| Company | the order's company |
| Sequence | the line's sequence |
| Attribute values that never create variants | the line's extra values |
| Packaging unit | the line's unit |
| Project | the order's project, when the project coupling is installed and the order has one |

### 4.8 Phase F — automatic locking

8. Each confirmed order is asked whether it should be locked. The answer is yes when the feature
   group "Lock Confirmed Sales" is enabled. The check is deliberately made on the *feature* rather
   than on the acting user's groups, because a public user can confirm an order from the portal.
9. The orders that answered yes are locked.

### 4.9 Phase G — confirmation message

10. When the caller asked for it (the "send email" instruction — set by the portal signature, by
    the payment post-processing and by the explicit "confirm and send" path), the confirmation
    message is sent to each order's customer with the confirmation template of section 3.2.
11. Independently, with the quotation-template capability, when the caller did **not** ask for the
    message and the order's quotation template names its own confirmation message, that message is
    sent anyway, because a template-specific message is assumed to carry information the customer
    needs.

### 4.10 Sending the message synchronously or asynchronously

The message-sending helper behaves as follows:

1. If there is no template, nothing happens.
2. If the caller runs with elevated rights, the sending is re-attributed to the system user so the
   notification is not attributed to the public visitor.
3. Read the system parameter `sale.async_emails` and the active state of the pending-email job.
   - If asynchronous sending is enabled, the job is active and deferral is allowed: store the
     template on the order's pending-template field and trigger the job. Nothing is sent now.
   - Otherwise: post the message immediately, forcing the send, with the notification layout that
     carries the responsible person's signature and the comment subtype.

### 4.11 Postconditions of the confirmation

- Status `sale`; order date equal to the confirmation instant.
- Per-line and order-level invoice statuses recomputed.
- Transfers created (goods lines), projects, tasks and milestones created (service lines), purchase
  requests created (buy-on-sale service lines), manufacturing orders requested where the rules say
  so.
- The lock flag possibly raised.
- Zero, one or two messages posted: the tracking note and, conditionally, the confirmation message.

---

## 5. Customer acceptance in the portal

**Performed by**: Customer. **Precondition**: the customer holds the address of the order, either
as an authenticated portal user whose commercial parent owns the order, or as an anonymous visitor
carrying the access token.

### 5.1 Opening the page

1. The document access check runs; failure redirects to the portal home.
2. If a payment amount was supplied in the address and it is smaller than the required prepayment
   amount while the order is not yet confirmed, the request fails with "The amount is lower than
   the prepayment amount."
3. If a report type was requested, the printable document is returned instead of the page.
4. **View logging.** When the reader is a shared (portal or public) user, an access token was used,
   the request is not a link preview, and the order status is `draft` or `sent`: if the session has
   not already recorded a view of this order today, record today's date in the session and post a
   note on the thread, as the system user, with the subtype "Quotation Viewed", reading "Quotation
   viewed by customer *name*" in the salesperson's (or the company's) language. The author is the
   order's customer for an anonymous visitor, the reader's contact otherwise.
5. The page is rendered with: the order, the visible product documents, the message flag, the
   back-office address, the company (for the logo) and the payment amount.
6. Payment values are added when the order has to be paid, or when a payment amount was supplied
   and the order is not expired: the compatible providers, methods and tokens for the computed
   amount, the transaction address, the landing address and a freshly ensured access token.
   - The amount offered is: the supplied amount when it is a down payment smaller than the total;
     else the required prepayment amount for a down payment; else the supplied amount or the total
     for a confirmed order; else the total.
   - Whether the payment is treated as a down payment follows the explicit choice in the address
     (`down_payment` or `full_amount`), or, absent a choice, "the prepayment percentage is smaller
     than one" when no amount was supplied, or "the supplied amount is smaller than the total"
     when one was.

### 5.2 Accepting by signature

1. The access check runs; failure answers "Invalid order."
2. The "has to be signed" predicate must hold; otherwise the answer is "The order is not in a state
   requiring customer signature."
3. A signature image must be supplied; otherwise the answer is "Signature is missing."
4. The signer name, the signature instant (now) and the signature image are written, then flushed
   immediately so that the printable document can embed the signature.
   - A malformed image answers "Invalid signature data."
5. **If the order does not also have to be paid**, the confirmation algorithm runs with the "send
   email" instruction and with the signature included in the rendered document.
6. The printable document is rendered as a portable-document-format file with the signature
   included.
7. A comment is posted on the thread with that file attached, named after the order, whose body is
   "Order signed by *name*". The author is the order's customer for an anonymous visitor, the
   reader's contact otherwise.
8. The answer instructs the browser to reload the page at the portal address with the query
   `&message=sign_ok`, plus `&allow_payment=yes` when the order still has to be paid.

### 5.3 Declining

1. The access check runs; failure redirects to the portal home.
2. The "has to be signed" predicate must hold **and** a decline message must be supplied. If either
   fails, the customer is redirected back with the query `&message=cant_reject` and nothing
   changes.
3. Otherwise the cancellation path runs directly — note that this bypasses the lock check of the
   ordinary cancel operation.
4. The decline text is posted as a customer comment on the thread, attributed as in section 5.2.
5. The customer is redirected to the portal address of the order.

### 5.4 Paying

1. The transaction endpoint checks the access token; an invalid token answers "The access token is
   invalid."
2. A draft transaction is created, linked to the order, with the reader's contact as partner for an
   authenticated user and the order's invoice address otherwise, the order currency, and the order
   identifier so that recurring-billing couplings can tokenise it.
3. The processing values are returned to the payment form.
4. When the provider reports the outcome, the post-processing of section 6 runs.

### 5.5 Downloading a product document

1. The access check runs; failure redirects to the portal home.
2. The document must exist, be active, and be one of the documents the order exposes — that is, its
   sale visibility is *on quote*, or it is *on confirmed order* and the order status is `sale`.
3. The attachment is streamed back as a download.

### 5.6 Downloading the structured order document

1. The access check runs.
2. The list of structured-document builders is taken; when it is empty the customer is redirected
   to the portal home.
3. The first builder exports the order; the result is returned with the extensible-markup-language
   content type, its computed length and a download file name.

---

## 6. Payment post-processing

**Performed by**: System, when a transaction changes state.

### 6.1 Pending transactions

For each transaction that reached *pending*:

1. The generic post-processing of the payment domain runs first.
2. Every linked order still in status `draft` or `sent` is collected; those in `draft` are moved to
   `sent` with change tracking suppressed.
3. If the provider is the manual one, each linked order's payment reference is filled with the
   computed communication: the order reference when the provider communicates by document
   reference; the literal `CUST/` followed by the customer identifier modulo 97, padded to two
   digits with a leading zero, when it communicates by customer identifier; nothing when the
   provider declares no communication kind. The result is then passed through the sale journal's
   reference-formatting rule of the company.
4. Unless the transaction is a card-validation operation, a "payment initiated" message is sent to
   the customer of each collected order.

### 6.2 Authorized transactions

For each transaction that reached *authorized*:

1. The generic post-processing runs.
2. The confirmation check of section 6.4 runs; it may confirm the order.
3. Unless the transaction is a card-validation operation, a "payment initiated" message is sent to
   the linked orders that were **not** confirmed by step 2.

### 6.3 Completed transactions

For each transaction that reached *done*:

1. Unless it is a card-validation operation, the confirmation check of section 6.4 runs, and the
   orders it did **not** confirm receive the "payment initiated" message.
2. Read the system parameter `sale.automatic_invoice`.
3. If automatic invoicing is on, the invoices are created now (section 6.5) — deliberately for the
   orders of every completed transaction, not only for the orders the transaction just confirmed,
   so that a partial payment still produces its advance invoice.
4. The generic post-processing runs, which posts the invoices.
5. If automatic invoicing is on and the caller did not ask to skip sending:
   - when asynchronous sending is enabled and the invoice-sending job exists, trigger that job;
   - otherwise send the invoices immediately (section 6.6).

### 6.4 Confirmation check driven by the paid amount

For each transaction:

1. Only the single-order case is supported: if the transaction is linked to zero or to several
   orders, nothing happens.
2. Take the linked order when its status is `draft` or `sent`.
3. Confirm it when **both**: it does not still have to be signed, **and** the confirmation amount
   has been reached (the required prepayment amount is not greater than the accumulated paid
   amount).
4. Confirmation runs with the "send email" and "include signature" instructions.

**Consequence.** A single payment of half the total confirms the order when the prepayment
percentage is fifty percent or less, and does not confirm it when the percentage is higher. Several
partial payments accumulate, because the paid amount is the sum over all authorized and completed
transactions.

### 6.5 Automatic invoicing after payment

For each completed transaction that has linked orders, with the transaction's company active:

1. Keep the linked orders whose status is `sale`.
2. Split them into *fully paid* orders (accumulated paid amount not less than the total) and the
   rest.
3. For the rest, generate one advance invoice each, using the advance-invoice wizard with the
   *fixed amount* method and an amount equal to the transaction amount (or the amount supplied in
   the context).
4. For the fully paid orders:
   a. force every line in status `sale` to behave as if its invoicing policy were *ordered
      quantities*, by writing the quantity to invoice as the ordered quantity minus the invoiced
      quantity — this is what makes a fully prepaid order invoice in full even when its products
      are invoiced on delivery;
   b. create the invoices with the *final* switch on and with the instruction not to raise when
      there is nothing to invoice.
5. Ensure an access token on every produced invoice, so that a concurrent structured-document
   post-processing cannot collide with the portal display.
6. Link the produced invoices to the transaction.

### 6.6 Sending the invoices automatically

Acting as the system user, for each transaction, in the transaction's company:

1. Select the invoices that are not yet marked as sent, are posted, and are ready to be sent.
2. Mark them as sent.
3. Send them with the instruction not to raise on failure and to fall back to a plain document
   when the structured document cannot be produced. The template is the one named by the system
   parameter `sale.default_invoice_email_template` when it names an existing template.

### 6.7 The retry job

A daily job, inactive by default and switched on together with automatic invoicing, re-sends the
invoices that were not ready at the time:

1. Stop immediately when automatic invoicing is off.
2. Select the transactions that are *done*, already post-processed, whose linked orders are
   confirmed, whose last state change is at most two days old, and which own at least one posted,
   not-yet-sent invoice.
3. Run the sending procedure of section 6.6 on them.

---

## 7. Cancelling and resetting

### 7.1 Cancel

**Performed by**: Salesperson. **Precondition**: none beyond the lock check.

1. If any selected order is locked, the whole operation fails with "You cannot cancel a locked
   order. Please unlock it first."
2. The cancellation path runs:
   a. With the inventory coupling: for each confirmed order with lines, the impacted downstream
      documents are collected as if every line's quantity dropped to zero; every transfer that is
      not completed is cancelled with the instruction not to raise its own activity; then a warning
      activity describing the decrease is raised on each impacted document that is not itself
      cancelled.
   b. With the purchasing coupling: a warning activity is raised, with elevated rights, on every
      non-cancelled purchase request line generated by a buy-on-sale service line of the cancelled
      orders, grouped so that one activity is raised per purchase request.
   c. Every draft invoice linked to the order is cancelled.
   d. The status is written to `cancel`.

### 7.2 Mass cancel

**Performed by**: Salesperson, from a list selection.

1. The dialogue reports how many orders were selected and whether at least one of them is
   confirmed, so the user can see that confirmed orders are about to be cancelled.
2. Confirming runs the ordinary cancel operation on the whole selection, with the same lock guard.

### 7.3 Back to quotation

**Performed by**: Salesperson.

1. Only the orders whose status is `cancel` or `sent` are affected; the others are silently
   ignored.
2. Those orders are written with: status `draft`, signature cleared, signer name cleared, signature
   instant cleared.
3. Nothing downstream is recreated or reversed; transfers cancelled by a previous cancellation stay
   cancelled. Re-confirming afterwards re-runs the confirmation algorithm, which creates fresh
   transfers for the quantities that are not yet procured, and which deliberately does **not**
   recreate projects, tasks or purchase requests that already exist for the same lines.

---

## 8. Invoicing — the invoicing algorithm

**Performed by**: Salesperson or Invoicing user, through the advance-payment dialogue; or System,
after a payment.

### 8.1 Entering the dialogue

The dialogue is opened on one or several orders. It offers three methods.

| Method | Behaviour |
|---|---|
| Regular invoice | Runs the invoice-creation algorithm of section 8.3 with `final` = the "deduct down payments" switch and `grouped` = the negation of the "consolidated billing" switch. |
| Down payment (percentage) | Requires exactly one order. Runs the advance-invoice procedure of section 8.6 with the amount kind *percentage*. |
| Down payment (fixed amount) | Requires exactly one order. Runs the advance-invoice procedure with the amount kind *fixed*. |

Before anything runs, the amount validation of [entities.md](entities.md), section 9.3, is
applied.

### 8.2 Access rule

The invoice-creation algorithm first checks whether the reader may create invoices. If not, it
checks whether the reader may write on the orders; if that also fails, the algorithm returns
nothing at all instead of raising. When the reader may write on the orders but may not create
invoices, the invoices are created with elevated rights — a salesperson can therefore invoice an
order without being allowed to create an invoice from scratch.

### 8.3 The invoice-creation algorithm

**Input**: a set of orders, a `grouped` switch, a `final` switch.
**Output**: the created invoices.

**Step 1 — build the invoice values, order by order.**

A running sequence counter starts at zero and is shared across all orders of the run.

For each order:

1. If the invoice address declares a language, the whole computation runs in that language.
2. The order's company is made active.
3. The invoice header values are prepared (the mapping of [calculations.md](calculations.md),
   section 8.1).
4. The invoiceable lines are selected (section 8.4).
5. If every selected line is a display line, the order is skipped entirely — an invoice made only
   of sections and notes is never created.
6. For each selected line, in the order produced by section 8.4:
   - the first time an advance line is met, insert a section line with the label "Down Payments",
     no product, no unit, quantity zero, discount zero, unit price zero, no account, at the current
     sequence counter; increment the counter;
   - prepare the optional values: the sequence counter; and, for an advance line, quantity −1 and
     the reversed extra tax data;
   - produce the invoice line values (the mapping of [calculations.md](calculations.md),
     section 8.2), and append them;
   - increment the counter.
7. Append the produced invoice lines to the header values and keep the whole thing for step 2.

**Step 2 — nothing to invoice.**

If no header values were produced at all and the caller did not ask to stay silent, the operation
fails with:

```
Cannot create an invoice. No items are available to invoice.

To resolve this issue, please ensure that:
   • The products have been delivered before attempting to invoice them.
   • The invoicing policy of the product is configured correctly.

If you want to invoice based on ordered quantities instead:
   • For consumable or storable products, open the product, go to the 'General Information' tab and change the 'Invoicing Policy' from 'Delivered Quantities' to 'Ordered Quantities'.
   • For services (and other products), change the 'Invoicing Policy' to 'Prepaid/Fixed Price'.
```

**Step 3 — grouping.**

When `grouped` is false (which is the *consolidated billing* case), the header values are grouped:

1. The grouping key is the tuple: company, customer, delivery address, currency, fiscal position.
2. The header values are sorted by that key, then consecutive runs with the same key are merged:
   - the first header of the run becomes the reference header;
   - every subsequent header's invoice lines are appended to it;
   - the set of source documents, the set of payment communications and the set of references are
     collected;
   - the merged header's reference becomes the comma-and-space-joined references, truncated to
     2000 characters; its source document becomes the comma-and-space-joined source documents; its
     payment communication is kept only when all merged headers agreed on a single one, otherwise
     it is cleared.

When `grouped` is true, one invoice is produced per order.

**Step 4 — re-sequencing.**

If fewer invoice headers remain than there were orders — that is, some grouping happened — the
lines of every header are renumbered from 1 upwards in their current order (see
[calculations.md](calculations.md), section 8.3).

**Step 5 — creation.**

The invoices are created with elevated rights, with the customer-invoice document type forced as
the default.

**Step 6 — negative invoices become credit notes.**

When `final` is true, every created invoice whose total is strictly negative is switched to the
credit-note document type. The team field is protected during the switch so that it is not
recomputed. Each switched document is then registered as the reversal of the order's existing
invoices.

**Step 7 — origin note.**

On every created invoice, a note is posted linking back to the orders behind its lines.

### 8.4 Selecting the invoiceable lines

The selection walks the order's lines in their natural order (sequence, then identifier) and keeps
a small amount of state so that a section or a subsection is only carried onto the invoice when at
least one of its children is invoiceable.

```
section_buffer   = empty
subsection_buffer= empty
advance_lines    = empty list
result           = empty list
precision        = the decimal precision named 'Product Unit'

for each line L of the order, in order:
    if L is a section:
        section_buffer    = [L]          # start a new section
        subsection_buffer = empty
        continue
    if L is a subsection:
        subsection_buffer = [L]          # start a new subsection
        continue
    if L is not a note and its quantity to invoice is zero at 'precision':
        continue
    if ( quantity to invoice > 0 )
       or ( quantity to invoice < 0 and final )
       or ( L is a note ):
        if L is an advance line:
            append L to advance_lines     # advance lines are grouped at the end
            continue
        if subsection_buffer is not empty:
            if L is a display line:
                append L to subsection_buffer
                continue
            append section_buffer then subsection_buffer to result
            subsection_buffer = empty
            section_buffer    = empty
        else if section_buffer is not empty:
            if L is a display line:
                append L to section_buffer
                continue
            append section_buffer to result
            section_buffer    = empty
            subsection_buffer = empty
        append L to result

return result followed by advance_lines
```

Three consequences worth stating explicitly:

- A note is always invoiceable; its quantity to invoice is zero but the zero test is skipped for
  notes.
- A section whose children are all fully invoiced never appears on the new invoice.
- A line with a **negative** quantity to invoice is only picked up when `final` is true. This is
  what confines credit-note generation to the final invoicing run.

### 8.5 What "final" means, in one sentence

`final` = "this is the closing invoice of the order": deduct the advance invoices already issued,
pick up negative quantities so that over-invoicing is corrected, and switch a negative result to a
credit note.

### 8.6 Creating an advance invoice

**Precondition**: exactly one order; the amount is strictly positive.

1. The dialogue's company is made active.
2. The order's priced lines are converted into base lines; tax details are added and rounded.
3. The advance base lines are produced by the *reduce to target amount* routine
   ([calculations.md](calculations.md), section 7), with the amount kind and amount taken from the
   dialogue and the computation key `down_payment,<dialogue identifier>`.
4. The advance section line is created on the order if there is not already one.
5. One advance order line is created per produced base line ([calculations.md](calculations.md),
   section 7.6).
6. The account to use per advance line is resolved as: the account carried by the base line, else
   the product's advance-invoice account, else the product's income account; and, overriding all
   of them, the company's advance-invoice account mapped through the order's fiscal position when
   the company declares one.
7. The invoice header is prepared exactly as an ordinary invoice, and its lines are the advance
   lines with quantity 1 and the label of [calculations.md](calculations.md), section 7.7.
8. The invoice is created with elevated rights and then de-escalated to the reader's rights.
9. An origin note linking the invoice to the order is posted on the invoice, authored by the
   reader when the reader is an internal user and by the system user otherwise.
10. A note is posted on the order: "*link labelled "Down payment invoice"* has been created".
11. The dialogue closes onto the created invoice.

### 8.7 Postconditions of an invoicing run

- One or more draft invoices (or credit notes) exist, each line linked to its order line.
- The invoiced quantity of every touched order line has risen (draft invoices already count).
- The per-line and order-level invoice statuses have been recomputed.
- The order's invoice count and invoice list include the new documents.
- Nothing is posted: the created documents are drafts, and posting is a separate act of the
  receivables domain.

---

## 9. Crediting and refunding

**Performed by**: Invoicing user.

There are two distinct paths, and the difference matters.

### 9.1 Credit note created from the invoice

The reversal facilities of the receivables domain produce a credit note whose lines are **not**
linked to the order lines. Consequently the order's invoiced quantity does not fall, and the order
does not become invoiceable again. This is deliberate: an arbitrary refund must not silently
re-open an order for re-invoicing.

### 9.2 Credit note created from the order

Running the invoicing algorithm with `final` true on an order whose quantity to invoice has become
negative produces a document with a negative total, which step 6 switches to a credit note and
registers as the reversal of the order's invoices. Its lines **are** linked to the order lines, so
the invoiced quantity falls and the order's status returns to *To Invoice* or *Fully Invoiced* as
appropriate.

### 9.3 Return-driven refund

With the inventory coupling, a customer return whose moves are flagged as refundable decreases the
delivered quantity of the order lines ([calculations.md](calculations.md), section 5.4). For a
product invoiced on delivered quantities this makes the quantity to invoice negative, and the
credit note of section 9.2 follows.

---

## 10. Handling an upsell

**Performed by**: System (detection) and Salesperson (resolution).

1. A product invoiced on *ordered quantities* is delivered beyond the ordered quantity — for
   example because a project took longer than planned.
2. The line's invoice status becomes *Upselling Opportunity*; when every remaining line is
   *invoiced* or *upselling*, so does the order's.
3. The transition into the order-level upselling state schedules the to-do activity described in
   [state-machines.md](state-machines.md), section 4.5.
4. The salesperson resolves it by raising the ordered quantity on the line. That write:
   - posts the quantity-change note on the thread;
   - makes the quantity to invoice positive again, so the line returns to *To Invoice*;
   - with the inventory coupling, launches procurement for the additional quantity;
   - with the purchasing coupling, increases the vendor's draft purchase line, or creates a new
     purchase request for the difference when the existing purchase line is already confirmed or
     cancelled.
5. Alternatively the salesperson decides not to charge the excess and leaves the line as it is; the
   order then stays in the upselling state, which is a reportable condition rather than an error.

---

## 11. Applying a discount

**Performed by**: Salesperson.

1. The discount dialogue opens on one order.
2. The user chooses one of three kinds and enters a percentage (as a fraction) or an amount.
3. *On all order lines*: the discount percentage, multiplied by one hundred, is written on every
   line of the order.
4. *Global discount* or *Fixed amount*: the company's discount product is resolved (creating it if
   allowed), the reduce-to-target routine produces one negative base line per tax group, and one
   new order line per base line is appended at sequence 999
   ([calculations.md](calculations.md), section 9).

---

## 12. Producing a payment link

**Performed by**: Salesperson or Invoicing user.

1. The dialogue is opened on the order and pre-filled with: the order currency, the invoice
   address, the suggested amount, the maximum amount (the remaining balance), the amount already
   paid and the required prepayment amount
   ([calculations.md](calculations.md), section 1.8).
2. The dialogue produces an address that points at the portal page of the order with the payment
   amount and the access token embedded.
3. Following that address opens the portal page with the payment form pre-loaded for that amount.

---

## 13. Period-end revenue accrual

**Performed by**: Invoicing user. **Precondition**: the selected orders belong to a single company
and share a single currency.

1. The dialogue defaults the accrual date to the last day of the previous month, the reversal date
   to the day after the accrual date, and the journal to the first general journal of the company.
2. Failure conditions: "Entries can only be created for a single company at a time."; "Cannot
   create an accrual entry with orders in different currencies."; "Reversal date must be posterior
   to date."
3. The amounts are computed as in [calculations.md](calculations.md), section 12.
4. The entry and its reversal are created and posted; a note is placed on the orders that received
   entries; the dialogue closes onto the two entries.

The journal items themselves are specified in [accounting-effects.md](accounting-effects.md).

---

## 14. Assigning the sales team

**Performed by**: System, whenever the salesperson changes.

The team is chosen by this heuristic, evaluated in order. Let *valid companies* be the empty
company plus the companies of the target user that are enabled in the reader's session.

1. Search the teams whose company is among the valid companies and of which the target user is
   either the leader or a member.
   - If that set is non-empty and an extra filter was supplied, apply the filter; if the context
     default team is in the filtered set, take it; otherwise take the first of the filtered set in
     the entity's own ordering.
2. If nothing was chosen: if the context default team is in the unfiltered set, take it; otherwise
   take the first of the unfiltered set.
3. If nothing was chosen and a context default team exists, take it.
4. Otherwise search all teams whose company is among the valid companies; if an extra filter was
   supplied, take the first team matching it.
5. Otherwise take the first team of that search.

The entity's own ordering is sequence ascending, then creation date descending, then identifier
descending.

---

## 15. Managing team membership

**Performed by**: Sales Manager.

1. Adding a user to the member list of a team creates a membership record for every user not
   already a member, and re-activates or archives the existing memberships so that they match the
   list exactly.
2. In single-membership mode (The system parameter `sales_team.membership_multi` is false),
   creating a membership — or re-activating one — archives every other active membership of the
   same user in a different team. The screen warns beforehand: "*user names* already in other teams
   (*team names*)."
3. The uniqueness validation and the company validation of [entities.md](entities.md), section 6.4,
   apply.
4. Changing a membership's user or team directly is not supported; the prescribed procedure is to
   archive the membership and create a new one.

---

## 16. Creating orders from an incoming structured document

**Performed by**: Salesperson (upload) or System (incoming message).

1. One or more attachments are supplied. An empty set fails with "No attachment was provided".
2. Each attachment is decoded by the document-import mixin, which produces one order per
   attachment, with the reader's own contact as the default customer.
3. The decoding contract — which fields of the structured document map onto which order fields — is
   specified in
   [../electronic-invoicing-and-document-exchange/README.md](../electronic-invoicing-and-document-exchange/README.md).
4. The user is redirected to the generated orders under the title "Generated Orders".

---

## 17. Adding products from the catalogue

**Performed by**: Salesperson.

1. The catalogue view opens with: the order's currency, the price precision of the unit-price
   field, and whether sections should be shown (only for saved orders).
2. Only products flagged as sellable are listed. For each product the price shown is the price-list
   price for a quantity of one, in the order currency, at the order date, and — for readers in the
   sale-warning group — the product's sale warning.
3. Setting a quantity on a product:
   - if a line already exists for that product in the selected section and the quantity is not
     zero, that line's quantity is written;
   - if such a line exists, the quantity is zero and the order is still a quotation, the line is
     deleted and the price-list price for a quantity of one is returned;
   - if such a line exists, the quantity is zero and the order is confirmed, the line's quantity is
     set to zero instead of deleting it;
   - if no such line exists and the quantity is positive, a line is created in the selected
     section;
   - if no such line exists and the quantity is zero, nothing is written and the price-list price
     for a quantity of one is returned.
4. The value returned to the screen is the line's discounted unit price.
5. Throughout the catalogue interaction, field-change tracking on the order is suppressed while the
   order is still a quotation, so the thread is not polluted with one note per click.

---

## 18. The asynchronous message job

**Performed by**: System. **Frequency**: daily, inactive by default.

1. Search the orders whose pending-template field is set.
2. Report the number found as the remaining work.
3. For each order, one at a time:
   - send the stored template synchronously (deferral is explicitly disallowed inside the job);
   - clear the pending-template field;
   - report one unit of progress and commit; stop early when the job runs out of time.

The job's active state is kept in step with the system parameter `sale.async_emails`: writing,
creating or deleting that parameter switches the job on or off accordingly.

---

## 19. Configuring a product on a line

**Performed by**: Salesperson (back office) or Customer (storefront).

### 19.1 Opening the configurator

The configurator opens when the chosen product template reports that it is configurable — it has
configurable attributes, or it has optional products, or it is a combo.

The client asks for the configuration values, supplying: the product template, the quantity, the
currency, the order date (so that prices are computed at the right rate), optionally the unit, the
company, the price list and a starting combination of attribute values.

The server answers with two lists.

**The main product list** contains one entry describing the product template with:

1. the combination to start from, resolved as follows:
   - keep the supplied attribute values that belong to this template;
   - for every attribute line of the template that the supplied values do not cover and whose
     display kind is not *multiple choice*, add the first active value of that line;
   - if nothing at all was supplied, take the template's first possible combination;
2. the resulting product variant when one exists;
3. the display name and the customer-facing description;
4. the price computed through the configurator price hook, for the given quantity, date, currency,
   price list and unit;
5. the price-list rule that produced it — which is stripped from the answer of the update call, so
   that it never leaves the server on that path;
6. the attribute lines, their values, each value's extra price, and whether each value is
   available given the template's exclusion rules.

**The optional product list** contains one entry per optional product of the template that passes
the display check, each built with that optional product's first possible combination computed
against the main product's combination, and carrying the identifier of the parent template. The
list is empty when the caller asked for the main product only.

The answer also repeats the currency identifier.

### 19.2 Changing the combination

When the customer changes an attribute value, the client asks for the updated combination,
supplying the template, the full list of chosen attribute values, the currency, the order date, the
quantity, and optionally the unit, the company and the price list. The server resolves the variant
for that exact combination — or falls back to the template when no variant exists — and returns the
basic product information: name, description, price and availability. The price-list rule is
removed from the answer.

### 19.3 Creating a variant on the fly

When the chosen combination involves a dynamically created attribute, the client asks the server to
create the variant, supplying the template and the combination. The server creates the variant and
returns its identifier.

### 19.4 Refreshing the optional products

When the main combination changes, the client asks for the optional products again, supplying the
template, its combination, the parent combination, the currency, the order date and optionally the
company and the price list.

### 19.5 Applying the result

The client returns to the order: the chosen variant, the chosen values of the attributes that
create no variant, the custom texts, the quantity, and the list of optional products the customer
accepted. The order then creates:

- one line for the main product, carrying the extra values and the custom values;
- one line per accepted optional product, linked to the main line, whose description gains the
  suffix "Option for: *main product display name*".

### 19.6 The combo configurator

For a combo product the client asks for the combo data, supplying the same context. The server
returns the combo choices, their items, each item's product and extra price, and whether each item
is itself configurable. A second call returns the price of a chosen configuration.

When the customer confirms, the client writes the chosen items into the transient
"selected combo items" payload of the combo line. The line-collection change handler then:

1. checks that the number of selected items equals the number of choices, failing with "The number
   of selected combo items must match the number of available combo choices." otherwise;
2. deletes the existing combo item lines of that combo line;
3. creates one line per selected item with: the item's product, the combo line's quantity, the
   combo item, the chosen values of attributes that create no variant, the custom values, a
   sequence equal to the combo line's sequence plus the item's index plus one, and a link to the
   combo line (through the real identifier when the combo line is saved, through the provisional
   identifier otherwise);
4. shifts the sequence of every line that came after the combo line by the number of selected
   items, so that the item lines sit directly under their combo;
5. clears the transient payload so the same change is not applied twice.

### 19.7 Keeping item lines in step

When the combo line's quantity or discount changes and its choices have not changed, the existing
item lines are updated with the same quantity and the same discount. When the combo line's product
is replaced by a product that is not a combo, the item lines are deleted.

---

## 20. The customer adding optional products from the portal

**Performed by**: Customer. **Precondition**: the quotation-template capability is installed, the
order is in status `draft` or `sent`, and the line belongs to an optional section.

1. The customer opens the order's customer page and sees the optional block: the lines that sit
   under a section flagged optional, or under a subsection whose section is flagged optional.
2. The customer presses the plus button, the minus button, or types a quantity. The client calls
   the update endpoint with the order identifier, the line identifier, the access token, and
   either a removal flag or an explicit quantity.
3. The server checks document access; failure redirects to the portal home.
4. The server checks that the **order** may be edited on the portal — its status must be `draft` or
   `sent`. Otherwise nothing happens.
5. The server checks that the **line** exists, belongs to that order, and may be edited on the
   portal — the order may be edited, the line is not a combo item, its product is not the company's
   discount product, and the line is optional. Otherwise nothing happens.
6. The new quantity is computed:
   - when an explicit quantity was supplied, the maximum of that quantity and zero;
   - otherwise the current quantity plus one (or minus one when the removal flag is set), floored
     at zero.
7. When the line's product is a combo, the same quantity is written on every linked combo item
   line first.
8. The line's quantity is written **while protecting the discount and the unit price**, so that the
   salesperson's manual pricing is not overwritten by the recomputation the write would otherwise
   trigger.
9. Unless the parameter that disables portal updates is set, and provided the order's company has
   at least one active price list, the line's unit price is recomputed — this is what makes a
   quantity-based price break apply when the customer increases the quantity.
10. The page is refreshed with the new totals.

---

## 21. Working the invoicing and upselling lists

**Performed by**: Salesperson.

### 21.1 Orders to invoice

1. The list shows every order whose order-level invoice status is "To Invoice"; creation is
   disabled on this list.
2. The salesperson selects one or several orders and presses "Create Invoice".
3. The advance-payment dialogue opens with the regular method pre-selected; consolidated billing is
   on by default, so orders of the same customer, address, currency and fiscal position are merged.
4. The created invoices open; the salesperson reviews and posts them through the receivables
   domain.

### 21.2 Orders to upsell

1. The list shows every order whose order-level invoice status is "Upselling Opportunity";
   creation is disabled.
2. Each of these orders also carries a to-do activity assigned to its salesperson.
3. The salesperson raises the ordered quantity of the over-delivered line, or leaves the order as
   it is.

---

## 22. Reporting

**Performed by**: Sales Manager.

1. The analysis entity is opened from the Reporting menu in one of four pre-grouped variants: all
   dimensions, by salesperson, by product, by customer.
2. Rows are restricted by the multi-company rule and by the ownership rules, so a salesperson in
   the "own documents" group only sees their own rows.
3. Measures are already converted into the reading company's currency and into each product's
   reference unit, so no further conversion is needed.
4. Drilling from a row opens the source order.
5. The sales-team dashboard offers the same analysis filtered to one team through its primary
   button, which is labelled "Sales Analysis" inside the sales application.
