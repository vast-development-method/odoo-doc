# Workflows

Every operational flow of the payments and bank reconciliation domain, step by step, with the role that performs it, the preconditions, and the records created or updated at each step.

## Roles used in this file

| Role | Meaning |
|---|---|
| Billing user | Can create and edit invoices, bills and payments, and can see basic payment reporting. Cannot see journal entries, reports or the reconciliation screens. |
| Basic accounting user | Everything a billing user can do, plus the basic bank reconciliation features. |
| Accountant | Everything, including journal entries, reconciliation, reports and advanced configuration except a few settings. |
| Accounting administrator | Everything, including the configuration a plain accountant cannot change. |
| Bank-account validator | A separate permission that allows setting or clearing the *trusted for outgoing payments* flag on a Bank Account. |

The precise permission matrix is in `configuration.md`.

---

## 1. Record a standalone payment

**Who:** billing user. **Precondition:** at least one journal of type `bank`, `cash` or `credit` exists in the company, with at least one payment method line in the wanted direction and a payment account on it.

1. The user opens the customer-payments list, the vendor-payments list or the payments list of a liquidity journal, and creates a record.
   - The screen presets the direction and the counterparty kind from the list it was opened from: inbound and customer, or outbound and supplier.
2. The system computes, in this order, the defaults of the new Payment:
   1. **Journal** — when the counterparty or the direction is already known, the journal of the counterparty's default payment method line for that direction; otherwise the first liquidity journal of the company.
   2. **Company** — from the journal, narrowed to an accessible branch.
   3. **Currency** — the journal's currency, or the company's currency.
   4. **Available payment method lines** — the journal's lines for the direction, minus the excluded codes.
   5. **Payment method line** — the counterparty's default line for the direction when it is available; otherwise the current one when still available; otherwise the first available; otherwise none.
   6. **Outstanding account** — the payment account of that line.
   7. **Destination account** — the counterparty's receivable account for a customer payment, its payable account for a vendor payment; or the company's first account of that type when there is no counterparty.
   8. **Available recipient bank accounts** — the journal's own bank account for an inbound payment; the counterparty's bank accounts for an outbound one.
   9. **Recipient bank account** — the first available one.
   10. **Date** — today.
   11. **State** — `draft`.
3. The user fills the counterparty, the amount, the date, the memo and, for an outbound payment, the recipient bank account. Every one of these recomputes the dependent defaults above.
4. **Duplicate warning.** While the counterparty, the amount, the date and the direction are set, the system looks for other Payments of the same company with the same four values whose state is `draft` or `in_process`, and lists them as possible duplicates. The user may proceed anyway.
5. **Quick response code.** For an outbound manual payment with a trusted recipient bank account and a currency, the system attempts to build a payable code and shows it with the caption "Scan me with your banking app."
6. The user saves. Validations run: the payment method line must exist and must belong to this journal; the amount must not be negative.
7. The user confirms.
   1. If the method requires a recipient bank account, the direction is outbound and the account is not trusted, the operation is refused.
   2. Payments whose outstanding account is of type `asset_cash` go straight to `paid`; the others go to `in_process`.
   3. Because the state write does not carry an entry, the system generates the Journal Entry (`accounting-effects.md`, section 1) and posts it.
   4. The Payment's number is computed from the entry's number.
8. **Records after the flow:** one Payment in `in_process` or `paid`; one posted Journal Entry with two lines; the liquidity line unmatched on the outstanding account.

---

## 2. Register a payment for one open document

**Who:** billing user. **Precondition:** the document is posted, has a residual, and is not blocked.

1. From the document, or from a selection of its journal items, the user opens the register-payment screen.
2. **Collect the lines.** From the active records:
   - when the active model is the entry model, take every journal item of the selected entries;
   - when it is the journal item model, take the selected items;
   - otherwise: "The register payment wizard should only be called on account.move or account.move.line records."
3. **Filter.** Keep only the items whose account type is `asset_receivable` or `liability_payable` and whose residual is non-zero — the item currency residual when the item has a currency, the company currency residual otherwise.
4. **Refuse impossible selections**, in this order:
   1. nothing left: "There's nothing left to pay for the selected journal items, so no payment registration is necessary. You've got your finances under control like a boss!"
   2. more than one root company: "You can't create payments for entries belonging to different companies."
   3. items from sibling companies whose root company the user may not act for: "You can't create payments for entries belonging to different branches without access to parent company."
   4. both receivable and payable account types in the selection: "You can't register payments for both inbound and outbound moves at the same time."
   5. any selected document whose payment state is `blocked`: "You cannot register payments for blocked invoices."
   6. a default journal inherited from the calling list that is not a liquidity journal of the right company is silently dropped rather than refused.
5. **Build the batches** (`calculations.md`, section 12.2). One document yields one batch, so the screen is editable.
6. **Compute the proposal**: the counterparty, the counterparty kind, the direction, the source currency and the two source amounts from the batch; then the journal, the currency, the payment method line, the recipient bank account, the amount, the installment mode, the memo and the grouping switch.
7. **Warn.** When any document already has a Payment in `in_process`, the screen shows, in the danger style, "There are payments in progress. Make sure you don't pay twice." with a link labelled "Check them".
8. The user adjusts the date, the amount, the journal, the method, the recipient account and the memo.
9. **Create.** The user confirms:
   1. When the screen was opened on a draft document, the difference handling is forced to `open`.
   2. Skip the batches for which a recipient bank account is required but missing or untrusted. If no batch survives: "To record payments with <the method name>, the recipient bank account must be manually validated. You should go on the partner bank account in order to validate it."
   3. Because there is one batch and either one item or grouping is on, the *edit mode* applies: one Payment is created from the screen's own values (`calculations.md`, section 12.12 decides the write-off).
   4. Create the Payment, apply the balance correction of `calculations.md`, section 12.16 when the currencies differ.
   5. Post the Payment.
   6. Reconcile: for each account present among the Payment's receivable or payable lines that are posted and unmatched, reconcile those lines together with the selected journal items of the same account.
   7. Record the Payment on each settled document's matched-payments relation.
10. **Records after the flow:** one Payment; one posted Journal Entry; one or more Partial Reconciliations; possibly a Full Reconciliation; possibly an exchange-difference entry; possibly cash-basis tax entries; the document's payment state recomputed.
11. The screen closes onto the created Payment, or onto the list of created Payments when there is more than one.

---

## 3. Register payments for many documents

**Who:** billing user.

The first eight steps are those of section 2. The difference is the number of batches and the grouping switch.

### 3.1 One batch, grouping on

The screen is editable. One Payment is created from the screen's values, for the whole batch. Its amount is what the user entered; its memo is the shared communication; its destination account is the account of the first selected item.

### 3.2 One batch, grouping off, more than one document

The screen is editable but the *edit mode* does not apply, because the batch holds more than one item and grouping is off. Therefore:

1. Determine the items actually to be paid: when the installment mode is `next`, `overdue` or `before_date`, the items behind the common and by-default buckets; otherwise every selected item.
2. Split the batch into sub-batches, one per document, keeping only the items to be paid. Each sub-batch's direction is recomputed from the sign of its own item's balance.
3. For each sub-batch, prepare the Payment from the **batch** values rather than the screen values: its amount is the batch's source amount in the source currency, its memo is derived from that batch's documents, its direction and counterparty kind come from the batch.
4. Where a discount applies to a sub-batch, the amount becomes the discounted default amount and the discount counterpart lines are attached as write-off lines.
5. Create, post and reconcile all the Payments in one operation.

### 3.3 Several batches

The screen is **not** editable: the amount, the counterparty and the recipient account are hidden and cannot be set. The journal and the method may still be chosen. Each batch produces one Payment (grouping on) or one Payment per document (grouping off), always from the batch values.

When a batch's direction differs from the direction of the method line chosen on the screen, the first available method line of the journal for the batch's own direction is used instead.

### 3.4 The memo

```formula
if the selected items belong to exactly one document:
    memo = that document's payment reference, or its own reference, or its number
else if any of the documents is an outbound document:
    memo = the distinct values of ( payment reference, or reference, or number ) of every document,
           sorted, joined by ", "
else:
    memo = the next value of the company's group-payment sequence
```

The group-payment sequence is a per-company, gapless, year-ranged sequence with the prefix `GROUP/<the year>/` and five digits of padding.

---

## 4. Register a payment with installments or an early discount

**Who:** billing user.

1. Steps 1 to 6 of section 2.
2. The screen reads the installments of each selected document (`calculations.md`, section 12.4) and computes the four totals (section 12.6).
3. The proposed amount is the **default** total, which is:
   - the discounted amount when a discount applies;
   - the sum of the overdue installments when some are overdue;
   - the next installment when none is overdue;
   - the sum of the installments due before the next payment date when the calling list filtered on that date;
   - otherwise the full amount.
4. Under the amount, a sentence explains what the amount is and offers one click to switch to the other candidate: the full amount, or the installment or discounted amount.
5. **Discount path.** When a discount applies and the amount equals either the default or the full total, the screen enters discount mode: the difference handling is forced to *mark as fully paid* and the write-off section disappears. On confirmation the difference becomes the discount counterpart lines of `calculations.md`, section 12.13, which are attached to the Payment as write-off lines.
6. **Installment path.** When the amount is an installment amount, the difference is the rest of the document and the difference handling defaults to *keep open*: the document stays partially paid.
7. **Custom amount.** As soon as the user types an amount that differs from all four totals, that amount is remembered as a custom amount, the automatic recomputation stops overwriting it, the switch sentence disappears, and the installment mode becomes `full`. Changing the currency converts the custom amount at the payment date; changing the payment date restores it unchanged.

---

## 5. Register a payment with a difference

**Who:** billing user.

1. Steps 1 to 8 of section 2, with an amount that differs from the total.
2. The screen computes the difference and, when it is non-zero, the screen is editable, grouping is on or unavailable, the wizard is not in discount mode and the method line has an outstanding account, shows the difference section.
3. The user chooses:
   - **Keep open** — no write-off. The reconciliation matches what it can; the document keeps a residual and its payment state becomes `partial`.
   - **Mark as fully paid** — the user picks a difference account and may change the write-off label. On confirmation:
     - when the chosen account is one of the company's two exchange-difference accounts and the payment currency differs from the source currency, no write-off line is produced: instead the payment's balance is forced (payment currency ≠ company currency) or a forced rate is passed to the reconciliation (payment currency = company currency), so the whole difference lands in an exchange-difference entry;
     - otherwise one write-off line is produced on the chosen account, signed by the direction.
4. **Records after the flow:** as in section 2, plus either a write-off journal item inside the Payment's entry or an exchange-difference entry.

---

## 6. Confirm, validate, reject, cancel and reset a payment

**Who:** billing user for confirm, validate, reject and cancel; accountant for reset when the entry is posted.

| Operation | Steps |
|---|---|
| **Confirm** | 1. For each Payment: if the method requires a recipient account, the direction is outbound and the account is untrusted, refuse. 2. Payments whose outstanding account is of type `asset_cash` become `paid`. 3. Payments in no state, `draft` or `in_process` become `in_process`. 4. The write of the state generates the Journal Entry for Payments that have none, and posts every generated entry still in draft. |
| **Validate** | Sets the state to `paid` directly. Used when the confirmation of the money does not come from a reconciliation. |
| **Reject** | Sets the state to `rejected`. The Journal Entry is left as it is; reversing or cancelling it is a separate decision. |
| **Cancel** | 1. Sets the state to `canceled`. 2. Deletes the Payment's entries that are still draft. 3. Cancels the remaining entries through the general ledger's cancel operation, with all its guards. |
| **Request cancellation** | Delegates to the entry's cancellation-request operation, used where an entry cannot be cancelled directly. |
| **Reset to draft** | 1. Sets the state to `draft`. 2. Resets the Journal Entry to draft through the general ledger's reset operation, with its lock-date, hash-chain and audit-trail guards. |
| **Mark as sent / unmark as sent** | Sets or clears the *is sent* flag. Used by check printing and by transfer-file generation. |

---

## 7. Delete a payment

**Who:** billing user, subject to the entry's own deletion guards.

1. Every Journal Entry of the Payments that is not in draft is reset to draft.
2. Every Journal Entry is deleted. Deleting the entry cascades to its journal items, which removes the matchings that involved them.
3. The Payments are deleted.
4. The payment state of every document that was reconciled with the Payments is forced to recompute.

---

## 8. Move money between two liquidity journals

**Who:** billing user.

1. From the dashboard card of a liquidity journal, the user opens the internal-transfer list. The screen presets the direction to outbound, the counterparty to the company's own partner and the internal-transfer context flag, and filters the list on transfers.
2. The user creates a first Payment: outbound, on the source journal, counterparty = the company partner, destination account = the company's **inter-bank transfer account**, amount and date as wanted.
3. The user creates the mirror Payment: inbound, on the destination journal, same counterparty, same destination account, same amount.
4. The two Payments are cross-referenced through the paired-transfer field.
5. Both are confirmed, producing the two entries of `accounting-effects.md`, section 2.
6. The two counterpart lines on the transfer account, being on the same reconcilable account with opposite signs, are reconciled with each other.
7. Each liquidity line is confirmed independently when the corresponding bank or cash transaction is reconciled.

**Precondition:** the company must have an inter-bank transfer account, reconcilable and of type `asset_current`. The shipped chart template creates it.

---

## 9. Record what the bank reports

**Who:** basic accounting user.

Three ways to bring transactions in.

### 9.1 Manually

1. From the dashboard card of a liquidity journal, or from the bank-statement list, the user creates a transaction.
2. Defaults: the journal from the context or the first liquidity journal of the company; and, when the journal is known, the date copied from the latest posted transaction of that journal, or from that transaction's statement when it has one, so that a whole statement can be typed without re-entering the date.
3. The user types the date, the label, the counterparty (optional), the amount and, when the bank converted a foreign amount, the foreign currency and the foreign amount.
4. On save the creation flow of `entities.md`, section 5.5 runs: the journal is taken from the statement when needed, a foreign currency equal to the journal currency is dropped, the entry type is forced, the default liquidity and suspense lines are created, and the entry is **posted immediately**.

### 9.2 By import

A capability package that imports statement files adds values to the journal's *bank feed* selection and creates the transactions and, usually, a Bank Statement carrying the file's reported balances and a reference to the file. The transaction fields it fills are: the date, the label, the amount, the foreign currency and amount when the file carries them, the counterparty's account number, the counterparty's name, the transaction type code and the raw payload.

### 9.3 By synchronisation

A capability package that synchronises with a bank provider does the same on a schedule and additionally records the provider's own reference on the Bank Statement.

---

## 10. Group transactions into a statement

**Who:** basic accounting user.

### 10.1 From a selection

1. The user selects contiguous transactions of one journal and asks to create a statement.
2. The defaults are computed (`entities.md`, section 4.4):
   - a single active record selects that transaction alone;
   - several active records select them all, after checking that they share a journal and that the run is contiguous once cancelled transactions are set aside; cancelled transactions inside the range are added to the selection.
3. The statement's starting balance is computed by the anchoring algorithm; its computed ending balance follows; its reported ending balance is initialised to the computed one.
4. The user overwrites the reported ending balance with what the bank says and saves.
5. The statement's name, date and first-line index are computed; the statement's completeness and validity follow.

### 10.2 By splitting

1. The user asks to split at a given transaction.
2. The system finds the last transaction before it that belongs to a different, non-empty statement, and selects every transaction of the journal strictly after that boundary and up to and including the chosen one.
3. The rest of the flow is as above.

### 10.3 Consequences

- A statement that is not complete shows "The running balance (<the computed balance>) doesn't match the specified ending balance."
- A statement whose starting balance does not equal the previous statement's reported ending balance shows "The starting balance doesn't match the ending balance of the previous statement, or an earlier statement is missing.", and the journal is flagged as having invalid statements, which surfaces on the dashboard card.
- A transaction that belongs to a statement that is both valid and complete can no longer be deleted: "You can not delete a transaction from a valid statement.\nIf you want to delete it, please remove the statement first."

---

## 11. Reconcile a bank transaction

**Who:** basic accounting user. The full ordering is in `calculations.md`, section 9.2; this section states it as an operational flow with the records touched.

1. **Open the reconciliation screen** on a liquidity journal. It lists the journal's transactions that are posted and not reconciled, most recent first.
2. **Pick a transaction.** The screen shows: the transaction's date, label, amount, running balance, counterparty and open amount; the candidate journal items (`calculations.md`, section 9.1); and the applicable reconciliation model's proposal when there is one.
3. **The counterparty is detected** when the transaction has none: from the reported account number through a Bank Account lookup, then from a partner-mapping reconciliation model, then — *industry-standard default* — from the reported third-party name.
4. **A reconciliation model is applied** when one matches (`calculations.md`, section 10). An automated model applies without asking; a manual model proposes.
5. **The user allocates.** Any combination of:
   - ticking candidate journal items to match;
   - adding a manual counterpart line with an account, a label and an amount;
   - choosing a different model;
   - changing the counterparty;
   - asking for the transaction to be recorded as a Payment.
6. **The amounts are converted** at the bank's own implied rates (`calculations.md`, section 8.5), so the bank's conversion prevails.
7. **Partial matching** is applied when the open amount is smaller than a matched item's residual, except against a Payment whose method is a transfer-file method or that has an online transaction, which must be matched in full.
8. **The entry is rewritten:** the suspense line is replaced by the counterpart lines; the single liquidity line is untouched.
9. **The reconciliation runs**, account by account, matching the new counterpart lines with the matched journal items.
10. **Side effects:**
    - a bank account is created or found for the reported account number when a counterparty is known and the creation is not disabled by the system parameter;
    - the transaction's residual recomputes and its reconciled flag follows;
    - an automated model also marks the entry as checked;
    - a model naming a next activity type schedules an activity on the transaction;
    - each created journal item records the model that produced it;
    - a Payment created on the fly is added to the transaction's auto-generated payments.
11. **Downstream:** every settled document's payment state recomputes; every settled Payment's matched flag and state recompute; exchange-difference and cash-basis entries are created as needed.

### 11.1 Creating or finding the counterparty's bank account

When the transaction reports an account number and has a counterparty:

1. If the system parameter `account.skip_create_bank_account_on_reconcile` is set to a true value, only **search**: a Bank Account with that number, that counterparty, and a company that is empty or the transaction's company. Return it or nothing.
2. Otherwise run the find-or-create algorithm:
   1. Search, ignoring the archived flag and with elevated privileges, for a Bank Account with that number whose holder is the counterparty's commercial partner or one of its children.
   2. If none is found and the counterparty is one of the database's own company partners and company-account creation was not explicitly allowed: "Please add your own bank account manually: <the account number> (<the counterparty display name>)".
   3. Otherwise create the Bank Account with that number, that counterparty and the trust flag **off**.
   4. From the resulting set keep those that are active and visible from the transaction's company, prefer the one whose holder is exactly the counterparty rather than a child, and return the first.

---

## 12. Undo a reconciliation

**Who:** basic accounting user; an accountant is required when the transaction is checked.

1. For each transaction that is checked **and** reconciled, if the current user may not review the entry: "Validated entries can only be changed by your accountant."
2. Remove every matching on every journal item of the transactions. This deletes the Partial Reconciliations, then the Full Reconciliations they belonged to, then reverses or deletes the exchange-difference and cash-basis entries, then recomputes the matching numbers, and finally pushes back to `in_process` the Payments that were `paid` only because of those matchings.
3. Delete every Payment auto-generated from those transactions.
4. For each transaction, clear the entry's journal items and recreate the default liquidity and suspense pair; set the checked flag to whether the current user may review entries.
5. Downstream: the residual returns to the full amount, the reconciled flag becomes false, the settled documents return to `partial` or `not_paid`.

---

## 13. Unreconcile journal items directly

**Who:** accountant.

1. From the journal items list, the user selects items and asks to unmatch.
2. The system extends the selection to the whole connected matching group through the matching numbers.
3. Every matching of that group is deleted, with the side effects of step 2 of section 12.

A single item can also be unmatched from its own screen, which removes every matching it takes part in.

---

## 14. Create and maintain a reconciliation model

**Who:** accountant (create, update); billing user can read and update but not create or delete.

1. The user opens the reconciliation-model list, ordered by sequence, and creates a model.
2. The user names it, sets its sequence by dragging, and fills the conditions: the journals it applies to (empty means all liquidity journals), the counterparties (empty means all), the amount condition with its one or two bounds, and the label condition with its parameter.
3. **Validation:** when the label condition is *match regex*, the parameter must compile as a regular expression, otherwise "The regex is not valid".
4. The user adds counterpart lines, in the order they must be evaluated: for each, the counterparty or the account, the amount mode, the amount text, the taxes, the analytic distribution and the label.
5. **Validation of each line's amount text:** not zero for the fixed mode ("The amount is not a number"), not zero for the two percentage modes ("Statement line percentage can't be 0" / "Balance percentage can't be 0"), and compilable for the regular-expression mode ("The regex is not valid").
6. The user chooses the trigger: *Set Manual* leaves the model as a proposal; *Automate* makes it reconcile matching transactions by itself.
7. Optionally the user picks a next activity type to be scheduled whenever the model is applied.
8. The system recomputes two derived flags: whether the model can be proposed spontaneously, and whether it is a partner mapping.
9. **Statistics.** The *Journal Entries* button lists every entry that contains at least one journal item created by this model.
10. **Duplicating** a model appends " (copy)" to its name, repeatedly until the name is unique, and copies its lines.
11. **Archiving** a model removes it from every proposal without deleting it or its history.

---

## 15. Set up a bank account for the company

**Who:** accounting administrator.

1. From the accounting dashboard, the user adds a bank account.
2. The bank setup screen opens. It delegates to a Bank Account whose holder is forced to the active company's partner.
3. The user types the account number. The journal name is initialised to the account number, then may be changed.
4. The user may name a bank, or only give a bank identifier code: in that case a Bank is searched by that code and created with that code as both its name and its code when not found.
5. The journal is proposed: the journal already pointing at this account, or the first journal of the wanted type (`bank` by default, `credit` when the screen was opened for a credit-card account) that has no bank account **and** has never carried a journal entry.
6. On save:
   - when no journal was selected, a new journal is created with the entered name, the next free sequence prefix for that type in the company, that type, the company, this Bank Account and the bank feed source `undefined`;
   - when a journal was selected, its bank account is set to this Bank Account and its name replaced by the entered name.
7. Validating reloads the screen. Capability packages that import statements extend this last step to offer an import straight away.

**Constraints:** a Bank Account may back at most one journal ("A bank account can belong to only one journal."); a `bank` journal's bank account must belong to the company's own partner and must not belong to another company.

---

## 16. Register and trust a counterparty's bank account

**Who:** billing user to create; bank-account validator to trust.

1. From the counterparty's record, or from a vendor bill, the user adds a bank account: the account number, optionally a bank, a clearing number, a currency and a holder name.
2. On save:
   - the number is sanitized (non-alphanumeric characters removed, the rest upper-cased) and, when it validates as an international number, normalised and rewritten in groups of four;
   - the type is inferred (international or plain);
   - the trust flag is forced to **false**, whatever was asked, and only set afterwards when the current user is a bank-account validator;
   - an archived account with the same number and holder blocks the creation: "A bank account with Account Number <the number> already exists for Partner <the partner name>, but is archived. Please unarchive it instead.";
   - a note is posted on the counterparty's message thread: "Bank Account <a link> created".
3. **Warnings.** For an untrusted international account, the screen warns when the account's country prefix differs from the holder's country, and when the institution code (characters five to seven of the sanitized number) is one of the known money-transfer services, naming the service.
4. **Duplicates.** The screen lists the holders of the other active accounts that carry the same number in the same company or in no company.
5. **Trusting.** A bank-account validator sets the *Send Money* flag. From then on:
   - the number, the sanitized number, the holder and the type are locked; changing any of them is refused with "You cannot modify the account number or partner of an account that has been trusted." unless the same change also clears the trust flag;
   - changing the trust flag at all requires the permission: "You do not have the rights to trust or un-trust accounts." and, on the constraint side, "You do not have the right to trust or un-trust a bank account."
6. **Deletion** never deletes: the account is archived and a note is posted, "Bank Account <a link> with number <the number> archived".

---

## 17. Send a payment receipt

**Who:** billing user.

1. From a Payment, the user asks to send the receipt by electronic mail.
2. A message composer opens, pre-filled from the payment-receipt template, with the payment receipt attached as a printable document.
3. The printable document shows the title returned by the receipt-title hook (by default "Payment Receipt"), the payment's own data, and — unless a capability package says otherwise — the payment method and the table of documents the payment settles.
4. Attachments created by the sending are re-pointed at the Payment when it has no main attachment yet.

---

## 18. Read the liquidity dashboard and act on it

**Who:** billing user (limited), basic accounting user (full).

For each liquidity journal, the dashboard card shows the figures of `calculations.md`, section 17, and offers these operations:

| Figure or button | Operation |
|---|---|
| Number to reconcile | Opens the reconciliation screen on the journal's unreconciled, checked, posted transactions. |
| Number to check / amount to check | Opens the list of the journal's posted transactions whose entry is not checked. |
| Account balance | Opens the difference screen: the journal items on the journal's default account since the last statement date (or the fiscal-year lock date). |
| Outstanding payments balance | Opens the list of the journal's Payments that are posted but not yet matched. |
| Last statement | Opens that Bank Statement. |
| Invalid statements warning | Opens the statement list of the journal. |
| New transaction | Creates a Bank Transaction on the journal. |
| New statement | Creates a Bank Statement on the journal. |
| Payments | Opens the journal's Payments, optionally filtered by direction or restricted to internal transfers. |
| Drop zone | Accepts a dropped file to import transactions, when an import capability is installed. |

---

## 19. The order in which everything settles

For a single customer invoice paid by bank transfer, the complete chain is:

1. The invoice is posted. Its receivable line has a residual. Payment state: `not_paid`.
2. A Payment is registered. Its entry debits the outstanding-receipts account and credits the receivable account. Payment state of the invoice: `paid` or `in_payment` according to the in-payment hook; Payment state: `in_process`; the Payment's *reconciled* flag is true, its *matched* flag false.
3. The bank reports the receipt. A Bank Transaction is created: its entry debits the bank account and credits the suspense account.
4. The transaction is reconciled against the Payment. The suspense line moves onto the outstanding-receipts account and is matched with the Payment's liquidity line.
5. The outstanding-receipts account returns to zero. The Payment's *matched* flag becomes true and its state becomes `paid`. The invoice's payment state becomes `paid`.
6. Later, the transaction is grouped into a Bank Statement whose reported ending balance agrees with the computed one; the statement is complete and valid.

Every one of those six steps is independent and reversible: undoing step 4 returns the Payment to `in_process` and the invoice to `in_payment`; deleting the Payment in step 2 returns the invoice to `not_paid`.
