# Workflows of the Accounts Receivable domain

Each workflow is a numbered sequence of steps with its actor, its preconditions, the records created
or updated at each step, and its postconditions. Formulas are referenced rather than repeated; see
[`calculations.md`](calculations.md).

**Actors** used throughout:

| Actor | Who | Rights |
| --- | --- | --- |
| Billing user | the person who issues invoices | member of the invoicing group |
| Accountant | the person responsible for the ledger | member of the accountant group (which implies the invoicing group) |
| Administrator | the person who configures the system | member of the accounting administration group |
| Customer | the person who receives and pays the document | portal user, or an anonymous visitor holding an access token |
| The system | a scheduled job or an automated caller | runs with elevated rights |

---

## 1. Creating a customer invoice by hand

**Actor:** billing user. **Precondition:** a sale journal exists for the company; the customer exists.

1. The billing user opens the customer invoice list and asks for a new record. A draft document is
   created in memory with the document type "Customer Invoice".
2. **Journal.** The first sale journal of the company is proposed; if the currency is already known
   and differs from the company currency, a sale journal in that currency is preferred.
3. **Customer.** The billing user picks the customer. This sets, in cascade:
   - the commercial entity on the document;
   - the delivery address, resolved from the customer;
   - the fiscal position, resolved from the customer and the delivery address;
   - the payment terms, from the customer's customer payment term;
   - the salesperson, from the customer's salesperson, else the commercial entity's, else the current
     user when internal, else the creating user;
   - the recipient bank account, resolved from the company's own bank accounts (because the document
     is inbound) by the sorting rule of [`entities.md`](entities.md) section 1.6;
   - the preferred payment method line, from the customer's preferred inbound method.

   If the customer has neither a receivable nor a payable account configured, a redirecting warning
   offers to open the accounting configuration.

4. **Dates.** The document date defaults to empty and may be typed; the accounting date follows it.
   The due date is derived from the payment terms once there are lines.
5. **Currency.** Defaults to the journal's currency, else the company currency. Choosing a foreign
   currency computes the currency rate at the document date.
6. **Lines.** For each line the billing user picks a product or types a description. Picking a
   product defaults:
   - the label (the product display name, plus the sales description on a second line, in the
     customer's language);
   - the unit (the product's reference unit);
   - the unit price (asked from the product, adjusted for tax inclusion, in the document currency, at
     the document date, through the fiscal position, converted to the line unit);
   - the taxes (the product's customer taxes filtered to the company, else the account's sale taxes,
     then mapped through the fiscal position);
   - the account (the product's income account under the fiscal position, else the most frequent
     account for this customer and document type, else the account of the last two lines when they
     agree, else the journal's default account);
   - the analytic distribution, resolved from the distribution models.

   The billing user may then change the quantity, the unit price, the discount and the taxes. Two
   display values update immediately: the line subtotal (tax-excluded) and the line total
   (tax-included).

7. **Section, subsection and note lines** may be inserted. They carry no amount and no account. A
   section may be marked to hide its composition or its prices on the printed document.
8. **Saving** runs the dynamic line synchronisation of [`calculations.md`](calculations.md) section 1.
   After it, the document holds, in addition to the typed lines:
   - the tax lines, one per tax repartition grouping key;
   - the discount allocation lines, when a discount allocation account is configured and lines carry
     a discount;
   - the cash rounding line, when a cash rounding method is set;
   - the early payment discount lines, when the payment term carries a discount in the "always" mode;
   - the payment term lines, one per distinct instalment date.
9. **Optional adjustments** before posting:
   - change the payment terms, which rebuilds the instalments and resets the due date;
   - type a due date by hand when there is no payment term;
   - attach a cash rounding method;
   - correct a tax amount directly in the totals block, which writes onto the matching tax line and
     marks it as manual so the synchroniser preserves it;
   - change the currency rate, which preserves the tax amounts in the document currency;
   - re-apply the fiscal position to the lines with the dedicated button after changing it.
10. **Warnings** may appear as banners: the credit limit warning, the tax lock date warning, the
    auto-post notices, and the duplicate list.
11. **Post.** See workflow 2.

**Postcondition:** a draft customer invoice whose lines balance, with no number.

---

## 2. Posting a customer invoice

**Actor:** billing user or accountant. **Precondition:** a draft invoice that passes every guard of
[`state-machines.md`](state-machines.md) section 1.4.

1. The user presses **Confirm**. When abnormal-invoice detection is enabled by the caller and the
   document carries an abnormal warning, a confirmation dialogue opens first listing the documents
   and asking the user to confirm.
2. All guards are evaluated and every failure is collected; if any, one message listing all of them
   is raised and nothing changes.
3. Documents dated in the future are, in the soft mode, not posted: their auto-post becomes "At Date"
   and a note is written. In the non-soft mode (the button uses the non-soft mode) they are posted.
4. Lock dates are evaluated and the accounting date is moved forward if necessary.
5. Analytic lines are created from the analytic distributions.
6. Recurring copies are produced when auto-post is monthly, quarterly or yearly.
7. Line partners are corrected to the commercial entity.
8. The status becomes posted and the "posted before" flag is set.
9. The number is assigned from the journal's sequence (see [`calculations.md`](calculations.md)
   section 8), unless the user typed one.
10. The payment reference is computed for a customer invoice that has none, using the journal's
    reference model and type, and is stamped onto the payment term line labels.
11. The customer's customer rank is increased by one.
12. If the total is zero, the "invoice paid" hook fires immediately.
13. The inalterability hash is computed when the journal is in restricted mode.

**Records created or updated:** the document (status, number, payment reference, hash); its lines
(partner, labels); the analytic lines; the customer (customer rank); possibly the next recurring
draft; possibly exchange difference and cash-basis entries that were waiting on this document.

**Postcondition:** a posted customer invoice with a number, a receivable line with a residual equal
to the total, and a payment status of "not paid".

---

## 3. Sending a customer invoice

### 3.1 One document

**Actor:** billing user. **Precondition:** the document is posted and is a sale document.

1. The user presses **Send**. The per-document constraints are checked; a failure raises immediately.
2. The single-document send wizard opens, pre-filled:
   - **Channels**: the checkbox list is built from the partner's invoice sending methods, excluding
     the "Manual" option; the channel checked by default is the partner's preferred one (falling back
     to e-mail). A channel that is not applicable to the company is not offered.
   - **Extra electronic deliveries**: every extra delivery declared by an installed extension whose
     applicability test passes, all checked.
   - **Electronic format**: the partner's electronic invoice format.
   - **Printable layout**: the partner's invoice layout, else the journal's, else the generic invoice
     layout; the selector is shown only when more than one layout is available and the document has
     no file yet.
   - **Mail template**: chosen by document type — the credit note template when every document is a
     customer credit note, the self-billing templates when every document is a self-billed vendor
     document, otherwise the invoice template.
   - **Language**: rendered from the template for this document.
   - **Recipients, subject, body**: rendered from the template in that language. Recipients are the
     template's default recipients (or its explicit "to", "carbon copy" and "partner to" fields),
     restricted to partners that have an e-mail address unless the caller allows partners without
     one.
   - **Attachments**: a placeholder for the document file that will be generated (unless one already
     exists), placeholders for the template's dynamic reports, the already-generated document file,
     and the template's static attachments. Attachments already linked are protected from deletion.
3. Alerts are computed. A "danger" alert blocks: its message is raised.
4. The user may edit the recipients, the subject, the body, add attachments, or save the body as a
   new mail template. The save-as-template path is a dialogue of its own, titled "Create a Mail
   Template", which prompts for a template name, creates the template from the current subject and
   body, selects it on the sending wizard and returns to the sending dialogue; its seven steps, its
   refusal message and its two dialogue sizes are in [`interfaces.md`](interfaces.md) section 14.2.
5. The user presses **Send**. The platform:
   1. raises any danger alert;
   2. remembers the chosen layout as the partner's default when the partner had none;
   3. generates the documents (see 3.3);
   4. sends the mails (see 3.4);
   5. when the chosen channels include the manual (download) channel, returns a download of the
      produced attachments — a single file when there is one, a compressed archive otherwise;
      otherwise the dialogue simply closes. Note that the wizard's checkbox list never offers the
      manual channel, so this branch is reached only when a caller sets the channels itself.

### 3.2 Several documents

**Actor:** billing user.

1. The user selects several documents and presses **Send**. The batch wizard opens.
2. It shows a **summary**: for each channel and each extra electronic delivery, how many of the
   selected documents would use it, using each document's own defaults. There is nothing to choose:
   every document uses its own partner's settings. The summary is a map from the channel or delivery
   key to a pair of a count and a label; each extra delivery's label is prefixed with the word "by "
   and a space, and the manual channel's ordinary label is replaced by "Manually", because in batch
   mode everything is produced asynchronously and nothing is handed to the user for download. The
   exact rules are in [`entities.md`](entities.md) section 8.2.
3. Alerts are computed over the whole selection; a danger alert blocks.
4. On confirmation, unless the caller forces the synchronous mode:
   - the sending background job must be active, otherwise the operation is refused (with a link to
     the job for an administrator);
   - each document's sending data is set to the author user and the author partner;
   - the job is triggered;
   - the user is told "Invoices are being sent in the background." under the title "Sending
     invoices".
5. The job then processes at most a fixed number of documents per run (ten by default), ordered by
   accounting date, then document date, then sequence position, then identifier, locking them so two
   runs cannot collide.

### 3.3 Generating the documents

For each document, with its resolved settings:

1. Run the pre-render hook (extensions add data here).
2. Mark whether an error is blocking: an error is blocking unless the caller allows a fallback
   document and the error is marked "continue anyway".
3. Call the external services that must run **before** rendering, for the documents without errors.
4. Render the printable documents, grouped by chosen layout, in batches (eighty documents per batch
   by default, configurable). A document that already has a file is not re-rendered. The rendered
   content is split per document; a failure to identify the documents in the output raises
   *"Cannot identify the invoices in the generated PDF: "* followed by the identifiers.
5. Run the post-render hook.
6. When a fallback is allowed and some documents failed after producing a file, report those errors
   on their threads.
7. Call the external services that must run **after** rendering.
8. Link the produced files: create the attachments, set each as the document's main attachment,
   and set the "sent" flag to true.
9. For documents that still failed and have no file, when a fallback is allowed: clear the error,
   render a *pro forma* document instead, and attach it.

### 3.4 Sending the mails

1. Generate any dynamic reports selected in the attachment widget and add them to the attachment
   list.
2. For every document whose partner has an e-mail address or whose recipient list is non-empty:
   - gather the attachments: the document's own files plus the widget's entries, skipping those the
     user unticked (unless they were added manually);
   - resolve the author and the sender address from the template, falling back to the document's own
     author computation;
   - post a comment message on the document's thread with the subject, the body, the recipients, the
     attachments, the reply-to address, the notification layout that carries the responsible's
     signature, the template's auto-delete and mail-server settings, and the document type name as
     the model description;
   - re-parent the message's attachments onto the message itself so they are not duplicated on the
     document.
3. Notify the journal's invoice subscribers, if any, with copies of the document's attachments.
   Failures here are logged and do not fail the send.
4. Push a notification to the author: on success, title "Invoices sent", body "Invoices sent
   successfully."; on failure, title "Invoices in error", body "One or more invoices couldn't be
   processed." Both carry an **Open** button that opens the documents concerned.
5. Clear each document's sending data, except when the run came from the job and the error is marked
   retryable.

**Postcondition:** the document carries a file, is marked sent, and the message thread holds the
message with its attachments.

---

## 4. The customer reads and pays the document in the portal

**Actor:** customer.

1. The customer follows the link in the e-mail, which points at the document's portal address with
   an access token, or signs in and opens **My Invoices**.
2. **The list page** (`/my/invoices`, and `/my/invoices/page/<number>` for further pages) shows every
   document that is neither draft nor cancelled and whose type is among the six invoice types, for
   the signed-in customer's commercial entity tree (enforced by a record rule). It offers:
   - sorting by date (document date descending), due date (descending), reference (number
     descending) or status (payment status);
   - filtering by all, overdue invoices, invoices, or bills;
   - a date range filter on the creation date;
   - a page counter, and a badge with the number of overdue invoices.
3. **The document page** (`/my/invoices/<identifier>`) renders the document inside the portal layout.
   It shows the amounts, the instalment table, the early payment discount notice when one applies,
   the payment history, and the buttons to download or print.
   - With the query parameter asking for the printable form, the page renders the document itself
     (as a web page, a printable file or plain text). When the document has no generated file the
     page is rendered in *pro forma* mode. When the customer's language is written right to left,
     the page language is switched so the layout follows. The layout used is the one set on the
     partner, else the generic invoice layout.
   - With the printable form **and** a download request, on a posted document, the official
     attachments are returned: a single file when there is one, otherwise a compressed archive named
     after the document.
4. **Online payment.** Available when all of the following hold:
   - the portal payment feature is enabled by system parameter;
   - the document is posted, is a customer invoice, has a non-zero residual and a non-zero total;
   - the payment status is not paid, in payment or partially paid;
   - there is no pending or authorised transaction from a real provider.

   When any of these fails, the page explains why. The explanation is built by testing six
   conditions **independently and in this order**, appending one sentence per condition that holds,
   and joining the collected sentences with line breaks. The conditions are not the negations of the
   eligibility bullets above; they are their own tests:

   | Order | Condition that appends the sentence | Sentence |
   | --- | --- | --- |
   | 1 | the system parameter that enables portal payment is off | This invoice cannot be paid online. |
   | 2 | at least one linked transaction is pending, authorised or done, **or** the residual is zero in the document currency | There is no amount to be paid. |
   | 3 | the status is not posted | This invoice isn't posted. |
   | 4 | the residual is zero in the document currency | This invoice has already been paid. |
   | 5 | the document type is not `out_invoice` (Customer Invoice) | This is not an outgoing invoice. |
   | 6 | at least one linked transaction is pending or authorised **and** comes from a provider other than the two built-in ones (the do-nothing provider and the custom provider) | There are pending transactions for this invoice. |

   Two consequences are worth stating, because they are not what the eligibility list suggests.
   First, the second condition fires as soon as **any** pending, authorised or done transaction
   exists, even on a partly paid invoice that still owes money, so a customer who paid one instalment
   online is told "There is no amount to be paid." while the residual is plainly positive. Second,
   conditions two and four both fire on a fully settled invoice, so a paid invoice collects two
   sentences, not one.

5. The payment form is built with: the compatible providers for the company, the partner and the
   amount; the compatible payment methods; the customer's saved tokens; whether the partner's company
   matches the document's company; the landing address (the document page) and the transaction
   address (`/invoice/transaction/<identifier>`).
6. The amount offered is the **next amount to pay** from the instalment computation of
   [`calculations.md`](calculations.md) section 10: the sum of the overdue instalments, or the next
   instalment, or the discounted amount when an early payment discount is running, or the whole
   residual. A custom amount may be passed in the address, protected by a signed token; a tampered
   amount redirects to the portal home.
7. The customer confirms. The payment providers domain creates a transaction, and on success creates
   an accounting payment and reconciles it with the document — see
   [`../payment-providers/workflows.md`](../payment-providers/workflows.md) for the transaction
   lifecycle and
   [`../payments-and-bank-reconciliation/workflows.md`](../payments-and-bank-reconciliation/workflows.md)
   for the payment and its reconciliation. What crosses the boundary back into this domain is
   listed in [`state-machines.md`](state-machines.md) section 1.3 (a done transaction posts a draft
   customer invoice) and in [`accounting-effects.md`](accounting-effects.md) section 6.5.
8. **Overdue batch payment.** The address `/my/invoices/overdue` collects every overdue customer
   invoice of the signed-in customer and offers one payment for the lot. All of them must share the
   partner, the company and the currency; otherwise the page fails with, respectively:
   > Overdue invoices should share the same partner.
   > Overdue invoices should share the same company.
   > Overdue invoices should share the same currency.
   The communication is the company's next batch payment communication when there is more than one
   document, and the single document's number otherwise. The transaction address is
   `/invoice/transaction/overdue`.

An invoice is **overdue** when it is neither draft nor cancelled, its type is customer invoice or
sales receipt, its payment status is none of in payment, paid, reversed, blocked or the imported-balance value, and its
due date is strictly before today.

---

## 5. Registering a payment against a customer invoice

**Actor:** billing user. **Precondition:** a posted customer invoice with a non-zero residual.

1. The user presses **Register Payment**. The register-payment wizard opens; its complete behavior
   belongs to
   [`../payments-and-bank-reconciliation/workflows.md`](../payments-and-bank-reconciliation/workflows.md).
2. The receivable-side inputs it reads from the document are:
   - the instalment list and the next amount to pay (see [`calculations.md`](calculations.md)
     section 10);
   - the early payment discount eligibility and, when eligible, the discounted amount and the
     write-off counterpart lines (section 4.5 of the same file);
   - the payment reference, used as the communication.
3. On confirmation a payment and its journal entry are created, and the payment's receivable line is
   reconciled against the document's payment term lines.
4. The document's residual, payment status and payment widgets recompute.

**Alternative — attaching an existing outstanding line.** On the document form the "outstanding
credits" block lists unreconciled counterpart lines. Pressing **Add** on one of them collects that
line plus every unreconciled line of the document on the same account and reconciles them together.
Pressing the remove icon on an already-applied line deletes that partial reconciliation.

---

## 6. Reversing a customer invoice (issuing a credit note)

**Actor:** billing user. **Precondition:** every selected document is posted and they share one
company.

1. The user selects one or more posted invoices and chooses **Reverse** (or, on the form, **Credit
   Note**). The reversal wizard opens with:
   - the selected documents;
   - the reversal date, defaulting to today;
   - the reason, hidden when the source is a plain journal entry;
   - the journal, defaulting to the first active journal among the sources' journals, restricted to
     journals of the company whose kind matches the sources' journals.
2. The user may change the date, the reason and the journal.
3. **Reverse** (the plain path):
   1. Default values are prepared per document (see
      [`accounting-effects.md`](accounting-effects.md) section 4.1).
   2. The documents are split into two batches: those that must be cancelled by their reversal —
      that is, not future-dated, and either the "reverse and create invoice" mode was chosen or the
      source is a plain journal entry — and the others.
   3. Each batch is reversed. The cancelling batch has its reconciliations removed first, its
      reverses posted immediately and reconciled with their sources.
   4. The recipient bank account of each new document is recomputed.
   5. A note is written on each source: "This entry has been **reversed**", the word being a link to
      the reverse.
4. **Reverse and Create Invoice** does everything above with the cancelling behavior and then creates
   a third document per source: a fresh draft copy carrying only the product, section, subsection and
   note lines, dated at the reversal date, with the origin copied.
5. The wizard records the produced documents and opens them: the form when there is one, the list
   otherwise.

**Postcondition:** for a customer invoice reversed plainly, a draft customer credit note the user
still has to review and post; for the cancelling paths, posted documents already reconciled.

---

## 7. Issuing a customer debit note

**Actor:** billing user. **Precondition:** every selected document is posted, is a customer invoice,
a customer credit note, a vendor bill or a vendor credit note, and is not already the source of a
debit note.

1. The user chooses **Create Debit Note**, either from the header button of a posted document or
   from the action menu of a list or kanban selection. The dialogue, titled "Create Debit Note",
   opens with the reason, the date (today), the "copy lines" option and the journal. The journal
   selector is restricted to journals whose kind is the one implied by the sources' common type —
   purchase when that common type is a vendor bill or a vendor credit note, sale in every other case,
   **including** the case where the selected sources have mixed types and therefore no common type.
   The "copy lines" checkbox is hidden when the common type is a credit note, but hiding it does not
   force it to false, and it reappears as soon as the selection has mixed types.
2. On confirmation, one document per source is created with the defaults of
   [`accounting-effects.md`](accounting-effects.md) section 5. The lines of the source are copied
   whenever the "copy lines" option is on, and cleared whenever it is off — the source's type does
   not enter into it. In particular a credit-note source with the option on **does** copy its lines;
   see the compatibility finding in [`accounting-effects.md`](accounting-effects.md) section 5.
3. A note is written on the source: "This debit note was created from: *(a link)*".
4. The produced documents are opened: the form when there is one, the list otherwise.

**Postcondition:** a draft customer invoice linked to the source, ready to be edited and posted.

---

## 8. Cancelling and resetting

**Actor:** billing user (reset), accountant (when the document is reviewed).

1. **Reset to Draft** is offered when the reset guards pass and the journal is not in restricted
   hash mode with a hash already set. It performs the steps of
   [`state-machines.md`](state-machines.md) section 1.7.
2. **Cancel** resets a posted document to draft first, then removes every reconciliation, cancels the
   attached payments, turns auto-post off and sets the status to cancelled.
3. **Request Cancellation** replaces the reset button when the document must be cancelled through an
   approved external request; the base platform never enables it.

---

## 9. The recurring invoice

**Actor:** billing user to set it up; the system to run it.

1. On a draft document the user sets auto-post to "Monthly", "Quarterly" or "Yearly" and, optionally,
   an end date.
2. When the document is posted, the platform copies it forward by one period: the accounting date and
   the document date are shifted by the period, and the copy is linked to the same chain origin. The
   copy is left in draft with the same auto-post setting.
3. Each night the auto-post job selects up to one hundred draft documents whose auto-post is not "No"
   and whose accounting date is on or before today, and posts them. It first tries the whole batch at
   once; if any document fails, it rolls back and retries one by one, locking each document, and
   writes the failure on the failing document's thread:
   > The move could not be posted for the following reason: *the error message*
4. Posting a member of the chain produces the next one, until the end date is passed.
5. Resetting a member of the chain to draft deletes the next occurrence when that next occurrence is
   still draft.

---

## 10. Mass posting and mass review

**Actor:** accountant.

1. The accountant selects several draft documents and chooses the validate action.
2. Documents that need confirmation — those whose accounting date or document date is in the future,
   and those whose journal is in restricted hash mode — are separated from the rest.
3. The rest are posted immediately in the non-soft mode.
4. For the ones needing confirmation, a wizard opens listing them and asking for an explicit
   confirmation before posting.
5. When no draft document with lines is selected at all:
   > There are no journal items in the draft state to post.
6. A separate action marks selected posted documents as reviewed; it requires the reviewer right.

---

## 11. Configuring a payment term

**Actor:** administrator.

1. Open the payment terms list and create a record. A first line is created by default: percent, one
   hundred, zero days, "Days after invoice date".
2. Add lines. Each new line's percentage defaults to one hundred minus the sum of the other percent
   lines, and its day count to the previous line's day count plus thirty.
3. Choose the delay type per line and, for "Days end of month on the", the day of the following
   month.
4. Optionally switch on the early payment discount and set the percentage, the number of days and the
   tax reduction mode (defaulted by company country).
5. The preview pane shows, for the example amount and the example date, one row per instalment date
   and, when a discount is set, the discounted amount and its deadline.
6. Saving validates: the percentages must total exactly one hundred; a discount requires a single
   line; the discount percentage and the discount days must be strictly positive; each line's
   percentage must be between zero and one hundred; the day-of-next-month value must be a number
   between zero and thirty-one.
7. Attach the term to customers (as their customer payment term) or pick it per document.

---

## 12. Configuring cash rounding

**Actor:** administrator. **Precondition:** the cash rounding feature group is granted.

1. Open the cash rounding methods list and create a record.
2. Set the rounding precision (the smallest coin), the rounding method (Up, Down or Nearest) and the
   strategy (Add a rounding line, or Modify tax amount).
3. For the "Add a rounding line" strategy, set the profit account and the loss account **for each
   company** that will use the method, because both are company-dependent. Choosing the strategy
   without a profit account for the current company shows the warning
   > **Warning for Cash Rounding Method: *the method name***
   > You must specify the Profit Account (company dependent)
4. Attach the method to a document. The rounding line is then maintained automatically on every save.

---

## 13. Managing a customer's credit limit

**Actor:** administrator, then billing user.

1. The administrator enables the credit limit feature in the accounting settings and sets a
   company-wide default limit.
2. On a customer, the billing user may switch on "Partner Limit" and set a specific limit; switching
   it off resets the limit to the company default.
3. While a draft customer invoice is being edited, the platform computes the customer's total credit
   as the receivable aggregate plus the amount awaiting invoicing plus this document's total, and
   shows the warning banner of [`entities.md`](entities.md) section 6.3 when the limit is exceeded.
4. The warning is informational; posting is not blocked.

---

## 14. Handling a detected duplicate

**Actor:** billing user.

1. While editing, the document shows the list of documents detected as duplicates by the rule of
   [`business-rules.md`](business-rules.md) section 15.
2. The user may open each duplicate, or press the action that deletes every detected duplicate of the
   current document.
3. Nothing is blocked: a duplicate may be posted.

---

## 15. Reading the receivable position of a customer

**Actor:** billing user or accountant.

1. Open the customer record. The accounting tab shows, for members of the invoicing or read-only
   accounting group: the total receivable, the total payable, the total invoiced, the days sales
   outstanding, the credit limit, the receivable account, the customer payment term and the degree of
   trust.
2. The total receivable is the sum of the residuals of the unreconciled receivable journal items of
   posted entries, across the company tree rooted at the active company's root.
3. A smart button opens the customer's documents.
4. The aged receivable report and the partner ledger give the same figures broken down by age and by
   document; they belong to [`../financial-reporting/README.md`](../financial-reporting/README.md).

---

## 16. Switching a document between invoice and credit note

**Actor:** billing user. **Precondition:** the document is draft.

The **switch to credit note / switch to invoice** action changes the document type to its counterpart
of the same direction (customer invoice ↔ customer credit note). Because the direction sign changes,
the tax synchronisation recomputes every tax line from the bases rather than preserving their
amounts, and the payment term lines are rebuilt from the new totals. Amount signs on the printed
document flip accordingly.

---

## 17. Printing

**Actor:** billing user, or customer through the portal.

1. **Print** renders the document with the layout resolved as: the partner's invoice layout, else the
   journal's, else the generic invoice layout. When no layout applies:
   > There is no template that applies to this move type.
2. The generic layout includes the payment history block; a second shipped layout omits it.
3. On a draft or unposted document, or when the document has no generated file, the printable form is
   rendered as a *pro forma* document, with the title changed accordingly ("Proforma Invoice", "Draft
   Proforma Invoice", …).
4. The rendered file is stored as the document's file attachment when it is produced by the sending
   flow; a direct print does not store it.
5. **Download** returns the stored attachments; several documents produce a compressed archive.

The section-by-section content of the printed document is in
[`interfaces.md`](interfaces.md) section 6.
