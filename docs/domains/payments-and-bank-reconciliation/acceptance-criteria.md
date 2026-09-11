# Acceptance criteria

Numbered scenarios in Given / When / Then form, with concrete numbers. A rebuilt system must reproduce every one of them.

Unless a scenario says otherwise:

- the company keeps its books in a currency written **C**, with two decimal places and a rounding step of 0.01;
- the liquidity journal *Bank* has the default account **Bank**, the suspense account **Bank Suspense**, one inbound manual method line whose payment account is **Outstanding Receipts** and one outbound manual method line whose payment account is **Outstanding Payments**;
- the customer *Northwind* has the receivable account **Accounts Receivable**; the vendor *Azure* has the payable account **Accounts Payable**;
- the full accounting capability is installed, so a document that is covered but not confirmed by the bank reaches the state `in_payment`;
- "today" is the payment date unless another date is given.

---

## A. Creating and confirming a payment

### A.1 A draft inbound payment gets its defaults

**Given** the customer *Northwind* and the journal *Bank*
**When** a user creates a Payment from the customer-payments list and fills the counterparty *Northwind*, the amount 1 000.00 and today's date
**Then** the direction is `inbound`, the counterparty kind is `customer`, the journal is *Bank*, the currency is **C**, the payment method line is the inbound manual line, the outstanding account is **Outstanding Receipts**, the destination account is **Accounts Receivable**, the state is `draft`, the number is empty and the display name is "Draft Payment"
**And** no Journal Entry exists.

### A.2 Confirming produces the entry

**Given** the draft Payment of A.1
**When** the user confirms it
**Then** the state becomes `in_process`
**And** one posted Journal Entry exists in the journal *Bank*, dated today, with two journal items:

| Account | Debit | Credit |
|---|---|---|
| Outstanding Receipts | 1 000.00 | |
| Accounts Receivable | | 1 000.00 |

**And** the Payment's number equals the entry's number
**And** the matched flag is false and the reconciled flag is false.

### A.3 A cash outstanding account settles immediately

**Given** a journal *Cash* whose inbound manual method line has the payment account **Cash** of type `asset_cash`
**And** a draft inbound Payment of 50.00 on that journal
**When** the user confirms it
**Then** the state becomes `paid`, not `in_process`.

### A.4 A payment whose liquidity account is the journal's own account is matched at once

**Given** a journal *Bank* whose inbound manual method line has the payment account **Bank**, that is, the journal's own default account
**And** a draft inbound Payment of 200.00 on that journal
**When** the user confirms it
**Then** the matched flag is true, because the journal's default account is among the liquidity accounts
**And** the Payment is counted in the *direct bank payments* figure of the dashboard, not in the outstanding figure.

### A.5 A payment cannot be negative

**Given** a new Payment
**When** the user sets the amount to −10.00 and saves
**Then** the save is refused with "The payment amount cannot be negative."

### A.6 A payment must have a method line

**Given** a journal with no payment method line for the chosen direction
**When** a user creates a Payment on it and saves
**Then** the save is refused with "Please define a payment method line on your payment."

### A.7 A method line of another journal is refused

**Given** a Payment on the journal *Bank*
**When** the user sets a payment method line that belongs to the journal *Cash* and saves
**Then** the save is refused with "The selected payment method is not available for this payment, please select the payment method again."

### A.8 A confirmed payment with an outstanding account must have an entry

**Given** a Payment with the outstanding account **Outstanding Receipts** and no Journal Entry
**When** its state is written directly to `in_process` in a way that does not create the entry
**Then** the write is refused with "A payment with an outstanding account cannot be confirmed without having a journal entry."

### A.9 A method with no payment account produces no entry

**Given** a payment method line whose payment account is empty
**And** a Payment using it
**When** the user confirms it
**Then** no Journal Entry is created
**And** the Payment's number is drawn from the payment sequence: the first one is `PAY00001`
**And** the reconciled flag is false and the matched flag equals whether the state is `paid`.

### A.10 An untrusted recipient account blocks an outbound payment that needs one

**Given** a payment method whose code is in the list of codes needing a bank account
**And** an outbound Payment to *Azure* whose recipient bank account is not trusted
**When** the user confirms it
**Then** the confirmation is refused with "To record payments with <the method name>, the recipient bank account must be manually validated. You should go on the partner bank account of Azure in order to validate it."

### A.11 The duplicate warning appears

**Given** an existing Payment of 1 000.00 to *Northwind* dated 10 March, inbound, in state `in_process`
**When** a user creates another inbound Payment of 1 000.00 to *Northwind* dated 10 March
**Then** the new Payment lists the existing one as a duplicate and the form shows "This payment has the same partner, amount and date as " with a link to it
**And** the user may nevertheless save and confirm.

### A.12 The duplicate warning ignores a different amount

**Given** the same existing Payment
**When** a user creates an inbound Payment of 1 000.01 to *Northwind* dated 10 March
**Then** no duplicate is reported.

### A.13 Cancelling a draft payment

**Given** a draft Payment with no entry
**When** the user cancels it
**Then** the state becomes `canceled` and nothing else changes.

### A.14 Cancelling a confirmed payment

**Given** a Payment in `in_process` with a posted entry
**When** the user cancels it
**Then** the state becomes `canceled` and the entry is cancelled — not deleted.

### A.15 Resetting to draft

**Given** a Payment in `in_process` with a posted entry
**When** the user resets it to draft
**Then** the state becomes `draft` and the entry returns to draft, subject to the general ledger's lock-date, hash-chain and audit-trail guards.

### A.16 Deleting a payment

**Given** a Payment in `in_process` whose entry is posted and reconciled with an invoice
**When** the user deletes the Payment
**Then** the entry is first reset to draft, then deleted; the matchings disappear with its journal items; the invoice returns to `not_paid`.

### A.17 Changing the amount of a draft payment rewrites the entry

**Given** a Payment created with explicit journal items so that it already has a draft entry of 1 000.00
**When** the user changes the amount to 1 200.00
**Then** the entry's liquidity line becomes 1 200.00 and its counterpart line becomes −1 200.00.

### A.18 Changing the amount of a posted payment does not rewrite the entry

**Given** a Payment in `in_process` with a posted entry of 1 000.00
**When** the amount is changed
**Then** the entry is untouched: a posted entry is never rewritten by the synchronisation.

### A.19 Changing the journal renumbers the entry

**Given** a Payment with a draft entry in the journal *Bank*
**When** the user changes the journal to *Cash*
**Then** the entry's journal becomes *Cash* and its number is reset to `/` so a new number is drawn at posting.

### A.20 Marking as sent

**Given** a Payment in `in_process` whose method code is `manual`
**When** the user marks it as sent
**Then** the sent flag is set, the *Reject* button appears and the *Cancel* button appears.

---

## B. Registering a payment from documents

### B.1 Nothing to pay

**Given** a fully paid customer invoice
**When** the user opens the register-payment screen on it
**Then** the screen refuses with "There's nothing left to pay for the selected journal items, so no payment registration is necessary. You've got your finances under control like a boss!"

### B.2 Mixing receivable and payable

**Given** one open customer invoice and one open vendor bill
**When** the user selects both and opens the register-payment screen
**Then** the screen refuses with "You can't register payments for both inbound and outbound moves at the same time."

### B.3 A blocked document

**Given** a customer invoice whose payment state is `blocked`
**When** the user opens the register-payment screen on it
**Then** the screen refuses with "You cannot register payments for blocked invoices."

### B.4 Two root companies

**Given** two open invoices belonging to two unrelated companies
**When** the user selects both and opens the register-payment screen
**Then** the screen refuses with "You can't create payments for entries belonging to different companies."

### B.5 The simple happy path

**Given** an open customer invoice of 1 000.00 to *Northwind*
**When** the user opens the register-payment screen and confirms with the proposed values
**Then** one Payment of 1 000.00 is created, inbound, customer, on the journal *Bank*, dated today, with the memo equal to the invoice's payment reference or number
**And** it is posted
**And** its counterpart line is matched with the invoice's receivable line for 1 000.00
**And** the invoice's payment state becomes `in_payment`
**And** the invoice is listed on the Payment's settled documents.

### B.6 A payment of 1 000 against invoices of 600 and 500, grouped

**Given** two open customer invoices of *Northwind*: invoice A of 600.00 due 10 March, invoice B of 500.00 due 20 March
**When** the user selects both, opens the register-payment screen, switches grouping on, sets the amount to 1 000.00 and confirms
**Then** exactly one Payment of 1 000.00 is created
**And** its entry is:

| Account | Debit | Credit |
|---|---|---|
| Outstanding Receipts | 1 000.00 | |
| Accounts Receivable | | 1 000.00 |

**And** two matchings are created: 600.00 between invoice A and the Payment, then 400.00 between invoice B and the Payment
**And** invoice A's payment state is `in_payment`, invoice B's is `partial` with a residual of 100.00
**And** no Full Reconciliation exists, because invoice B still has a residual
**And** all three journal items share one partial matching number.

### B.7 The same selection without grouping

**Given** the two invoices of B.6
**When** the user leaves grouping off and confirms
**Then** two Payments are created, one of 600.00 and one of 500.00
**And** invoice A and invoice B are each fully covered and both reach `in_payment`.

### B.8 The ordering follows the due date, not the amount

**Given** the two invoices of B.6 and one grouped Payment of 600.00
**When** the reconciliation runs
**Then** the whole 600.00 goes to invoice A, which is due first, and invoice B keeps 500.00 — even though a match of 500.00 against invoice B and 100.00 against invoice A would also have been possible.

### B.9 The memo of a grouped customer payment

**Given** two open customer invoices of *Northwind* and grouping on
**When** the user opens the register-payment screen
**Then** the memo is the next value of the company's group-payment sequence, for example `GROUP/2026/00001`.

### B.10 The memo of a grouped vendor payment

**Given** two open vendor bills of *Azure* whose references are `BILL-1` and `BILL-2`, and grouping on
**When** the user opens the register-payment screen
**Then** the memo is `BILL-1, BILL-2` — the distinct references, sorted, joined by a comma and a space.

### B.11 The memo of a single-document payment

**Given** one open invoice whose payment reference is `INV/2026/0007`
**When** the user opens the register-payment screen
**Then** the memo is `INV/2026/0007`.

### B.12 Batching merges the two directions of one counterparty

**Given** *Northwind* with one open invoice of 1 000.00 and one open credit note of 300.00, both on the receivable account, both in **C**, and exactly one recipient bank account per direction
**When** the user opens the register-payment screen
**Then** the two lines form **one** batch whose net balance is +700.00, so the direction is `inbound` and the proposed amount is 700.00
**And** one Payment of 700.00 is created, matched against both documents.

### B.13 Batching keeps two counterparties apart

**Given** one open invoice of *Northwind* for 1 000.00 and one of *Azure Retail* for 500.00
**When** the user selects both and opens the register-payment screen
**Then** there are two batches, the screen is not editable, the amount and the counterparty are hidden
**And** confirming creates two Payments, one per counterparty.

### B.14 A warning for payments in progress

**Given** an invoice already covered by a Payment in `in_process`
**When** the user opens the register-payment screen on that invoice again
**Then** the danger banner "There are payments in progress. Make sure you don't pay twice." is shown with a link labelled "Check them"
**And** the creation is still allowed.

### B.15 Registering on a draft document forces the difference open

**Given** a draft customer invoice with a non-zero total
**When** the user opens the register-payment screen on it, sets a lower amount and confirms
**Then** the difference handling is forced to *keep open* and no write-off line is produced.

### B.16 Untrusted accounts skip a batch

**Given** two batches for two vendors, the first with a trusted recipient account and the second with an untrusted one, and a payment method that needs a recipient account
**When** the user confirms
**Then** only the first batch produces a Payment
**And** the banner reported "1 out of 2 payments will be skipped due to untrusted bank accounts."

### B.17 Every batch skipped

**Given** the same setting with both accounts untrusted
**When** the user confirms
**Then** the creation is refused with "To record payments with <the method name>, the recipient bank account must be manually validated. You should go on the partner bank account in order to validate it."

---

## C. Installments and the early payment discount

### C.1 Two installments, none overdue

**Given** a customer invoice of 1 000.00 with two installments: 400.00 due in ten days and 600.00 due in forty days, today being before both
**When** the user opens the register-payment screen
**Then** the installment mode is `next`, the proposed amount is 400.00, the full amount is 1 000.00
**And** the sentence reads "This is the next unreconciled installment." then "Consider paying the full amount."
**And** the difference is 600.00 with the handling *keep open*.

### C.2 Switching to the full amount

**Given** the screen of C.1
**When** the user clicks the switch
**Then** the amount becomes 1 000.00, the mode becomes `full` and the difference becomes 0.00.

### C.3 One overdue installment and one future

**Given** a customer invoice of 1 000.00 with 400.00 due ten days ago and 600.00 due in thirty days
**When** the user opens the register-payment screen
**Then** the mode is `overdue`, the proposed amount is 400.00
**And** the sentence reads "This is the overdue amount." then "Consider paying the full amount."

### C.4 Two overdue installments

**Given** a customer invoice of 1 000.00 with 400.00 due twenty days ago and 600.00 due ten days ago
**When** the user opens the register-payment screen
**Then** both are collected as overdue, the proposed amount is 1 000.00 and the mode is `overdue`
**And** because the default amount equals the full amount, the mode resolves to `full` and the switch text is suppressed.

### C.5 Filtering on a next payment date

**Given** the same invoice as C.1 and a calling list whose active domain filters on a next payment date fifteen days from today
**When** the user opens the register-payment screen
**Then** the mode is `before_date`, only the 400.00 installment is collected, and the sentence reads "Total for the installments before <that date>." then "Consider paying the full amount."

### C.6 A payment taking a two percent early discount

**Given** a customer invoice of 1 000.00 with no tax and a payment term granting 2 % if paid within seven days
**And** the payment date is three days after the invoice date
**When** the user opens the register-payment screen and confirms with the proposed values
**Then** the proposed amount is 980.00, the mode is `full`, the discount mode is on, the difference is 20.00, the handling is forced to *mark as fully paid* and the write-off section is hidden
**And** the created Payment's entry is:

| Account | Debit | Credit | Label |
|---|---|---|---|
| Outstanding Receipts | 980.00 | | Manual Payment |
| Cash Discount Loss | 20.00 | | Early Payment Discount |
| Accounts Receivable | | 1 000.00 | Manual Payment |

**And** the invoice's payment state becomes `in_payment` with a residual of 0.00.

### C.7 The same discount on a vendor bill

**Given** a vendor bill of 1 000.00 with a 2 % discount within seven days, paid within the window
**When** the user confirms
**Then** the amount is 980.00 and the entry is:

| Account | Debit | Credit |
|---|---|---|
| Outstanding Payments | | 980.00 |
| Cash Discount Gain | | 20.00 |
| Accounts Payable | 1 000.00 | |

### C.8 The discount window has passed

**Given** the invoice of C.6 and a payment date eight days after the invoice date
**When** the user opens the register-payment screen
**Then** the document is not eligible, the proposed amount is 1 000.00, the discount mode is off and the handling defaults to *keep open*.

### C.9 A partially matched document is not eligible

**Given** the invoice of C.6, already matched for 100.00 by another payment, inside the discount window
**When** the user opens the register-payment screen
**Then** the document is not eligible for the discount, because one of its payment-term items is already matched.

### C.10 A discount with taxes, computation included

**Given** a customer invoice of 1 000.00 net plus 21 % tax, total 1 210.00, with a 2 % discount whose computation mode is `included`, paid within the window
**When** the user confirms
**Then** the amount proposed is 1 185.80
**And** the counterpart lines are: one base line of 20.00 on the cash-discount loss account, carrying the same taxes and tax grids as the product line, and one tax line of 4.20 labelled "Early Payment Discount (<the tax name>)" on the tax account
**And** the receivable is credited 1 210.00 in total.

### C.11 A discount with taxes, computation excluded

**Given** the same invoice with the computation mode `excluded`
**When** the user confirms
**Then** no tax line is produced and the whole 24.20 sits on the cash-discount loss account, leaving the tax report untouched.

### C.12 The discount rounding fix

**Given** an invoice with three product lines whose individual discounted deltas round to 6.67, 6.67 and 6.67, while the payment-term item's discount is 20.00
**When** the counterpart lines are produced
**Then** the sum of the base lines is 20.01 before correction; the delta of −0.01 is added to the base line with the largest foreign amount, so the counterpart lines sum to exactly 20.00.

---

## D. The payment difference

### D.1 A write-off of three hundredths

**Given** a customer invoice of 100.00 and a bank receipt of 99.97
**When** the user opens the register-payment screen, types 99.97, chooses *mark as fully paid* and the difference account *Rounding Difference* (an expense account), keeps the label `Write-Off`, and confirms
**Then** the difference is 0.03
**And** the created Payment's entry is:

| Account | Debit | Credit | Label |
|---|---|---|---|
| Outstanding Receipts | 99.97 | | Manual Payment |
| Rounding Difference | 0.03 | | Write-Off |
| Accounts Receivable | | 100.00 | Manual Payment |

**And** the invoice reaches a residual of 0.00 and the payment state `in_payment`.

### D.2 The same difference kept open

**Given** the same invoice and receipt
**When** the user chooses *keep open*
**Then** the entry has only two lines of 99.97
**And** the invoice keeps a residual of 0.03 and the payment state `partial`.

### D.3 A write-off on an outbound payment

**Given** a vendor bill of 100.00 and a transfer of 99.97, with *mark as fully paid* and an expense difference account
**When** the user confirms
**Then** the difference is 0.03 and the entry is:

| Account | Debit | Credit |
|---|---|---|
| Outstanding Payments | | 99.97 |
| Rounding Difference | | 0.03 |
| Accounts Payable | 100.00 | |

### D.4 Overpaying

**Given** a customer invoice of 100.00 and a receipt of 105.00
**When** the user types 105.00 and keeps the difference open
**Then** the difference is −5.00
**And** the Payment's counterpart line of 105.00 matches the invoice for 100.00 and keeps a residual of 5.00 on the receivable account, available for the next invoice.

### D.5 The custom-amount detector

**Given** an invoice of 1 000.00 with two installments of 400.00 and 600.00
**When** the user types 500.00
**Then** 500.00 matches none of the four totals, so it is remembered as a custom amount, the installment mode becomes `full`, the switch sentence disappears, and changing the payment date does not change the amount.

### D.6 The custom amount is converted when the currency changes

**Given** a custom amount of 500.00 typed in **C**
**When** the user changes the payment currency to **F**, whose rate at the payment date is 1 C = 0.80 F
**Then** the amount becomes 400.00 F.

### D.7 The difference section is hidden when the method has no outstanding account

**Given** a payment method line with no payment account
**When** the user types an amount different from the total
**Then** the difference section is not shown, because no write-off line could be booked.

---

## E. Multi-currency payments and reconciliation

### E.1 A payment in a foreign currency

**Given** a customer invoice of 1 500.00 **F** booked at 2 000.00 **C** (rate 1 C = 0.75 F)
**And** a Payment of 1 500.00 **F** dated when 1 C = 0.80 F
**When** the Payment is confirmed
**Then** its entry is:

| Account | Debit (C) | Credit (C) | Currency | Foreign amount (F) |
|---|---|---|---|---|
| Outstanding Receipts | 1 875.00 | | F | +1 500.00 |
| Accounts Receivable | | 1 875.00 | F | −1 500.00 |

### E.2 Reconciling it produces an exchange loss

**Given** the invoice and the Payment of E.1
**When** their receivable lines are matched
**Then** one matching is created with: amount 1 875.00 C, debit foreign amount 1 500.00 F, credit foreign amount 1 500.00 F
**And** one exchange-difference entry is created in the company's exchange journal:

| Account | Debit (C) | Credit (C) | Foreign amount (F) |
|---|---|---|---|
| Accounts Receivable | | 125.00 | 0.00 |
| Foreign Exchange Loss | 125.00 | | 0.00 |

**And** the invoice's receivable line ends with a company residual of 0.00 and a foreign residual of 0.00
**And** the exchange entry's first line is reconciled with the invoice's receivable line.

### E.3 An exchange gain

**Given** the same invoice and a Payment dated when 1 C = 0.70 F, so 1 500.00 F is worth 2 142.86 C
**When** the two are matched
**Then** the matching amount is 2 000.00 C — the smaller of the two company-currency images — and the exchange line fixes the **credit** side by −142.86, booked on the income exchange account:

| Account | Debit (C) | Credit (C) |
|---|---|---|
| Outstanding Receipts | 142.86 | |
| Foreign Exchange Gain | | 142.86 |

### E.4 The rounding-tolerance collapse

**Given** an invoice of 1 500.00 F worth 2 000.00 C and a payment of 1 500.00 F worth 1 999.99 C
**When** the two are matched
**Then** the four collapse tests all pass, so the matched amount becomes 1 999.99 C on both sides
**And** exactly one exchange line of 0.01 C is produced, for the debit side; no second, spurious line is produced for the credit side.

### E.5 A partial matching in a foreign currency keeps the ratio

**Given** an invoice of 1 000.00 F booked at 1 250.00 C and a payment of 400.00 F booked at 512.00 C
**When** the two are matched
**Then** the matching amount is the smaller company-currency image of 400.00 F
**And** an exchange line is produced for whichever side drifts, so that after the matching the ratio between the invoice's remaining company residual and its remaining foreign residual still equals 1 250.00 ÷ 1 000.00.

### E.6 A company-currency item matched against a foreign-currency item

**Given** a receivable line of 1 000.00 C with no foreign currency, on a trade account
**And** a payment line of 800.00 F, where the platform rate at the relevant date is 1 C = 0.80 F
**When** the two are matched
**Then** the company-currency item is given a mimicked foreign residual of `round_to(F, 1 000.00 × 0.80) = 800.00 F`, so the reconciliation currency is F and the two match exactly.

### E.7 The mimicking rule does not apply to a non-trade account

**Given** the same pair but with both items on a non-trade reconcilable account
**When** the two are matched
**Then** no mimicked entry is added, the reconciliation currency is the company currency, and the amounts are matched in **C** only.

### E.8 The payment's own rate prevails when the counterpart is a payment

**Given** an invoice item and a payment item, where the payment item is the payment-like one
**When** the rate is needed for the invoice item
**Then** the rate used is the **payment item's accounting rate**, that is its foreign amount divided by its balance in absolute value, not the rate table.

### E.9 The balance correction for a tiny foreign amount

**Given** a currency **B** where 1 C buys 0.01 B
**And** an invoice of 12.15 C
**When** the user registers a payment of 0.12 B against it
**Then** before posting, the Payment's balance is corrected: 0.12 B converts to 12.00 C, the documents ask for 12.15 C, the converted comparison matches, so 0.15 C is added to the first debit line and 0.15 C to the first credit line, and the invoice is settled exactly.

### E.10 A write-off on the exchange account with a foreign payment currency

**Given** an invoice of 1 000.00 F worth 1 250.00 C and a payment of 1 000.00 F worth 1 240.00 C
**When** the user chooses *mark as fully paid* with the company's expense exchange account as the difference account
**Then** no write-off line is produced; instead the Payment is created with a forced balance equal to the sum of the documents' company residuals, so the whole 10.00 C difference lands on the exchange side.

### E.11 A write-off on the exchange account with a company-currency payment

**Given** the same invoice and a payment expressed in **C**
**When** the user chooses the exchange account as the difference account
**Then** no write-off line and no forced balance are produced; a forced rate equal to the absolute value of the sum of the documents' foreign residuals divided by the payment amount is passed to the reconciliation, which pushes the difference into an exchange-difference entry.

---

## F. Bank transactions

### F.1 Creating a transaction posts its entry

**Given** the journal *Bank* with a suspense account
**When** a user creates a Bank Transaction dated 15 March, labelled "Transfer Northwind", amount +1 500.00
**Then** one Journal Entry is created **and posted** at once, with:

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| Bank | 1 500.00 | | +1 500.00 |
| Bank Suspense | | 1 500.00 | −1 500.00 |

**And** the transaction's residual is −1 500.00 and its reconciled flag is false.

### F.2 A missing suspense account

**Given** the journal *Bank* with no suspense account and a company with no suspense account either
**When** a user creates a Bank Transaction
**Then** the creation is refused with "You can't create a new statement line without a suspense account set on the Bank journal."

### F.3 A foreign currency equal to the journal currency is dropped

**Given** the journal *Bank* held in **C**
**When** a transaction is created with the foreign currency **C** and a foreign amount of 100.00
**Then** the foreign currency is cleared and the foreign amount set to 0.00 before the record is created.

### F.4 The consistency of the foreign pair

**Given** a saved transaction
**When** the user sets a foreign amount without a foreign currency
**Then** the save is refused with "You can't provide an amount in foreign currency without specifying a foreign currency."
**And** setting a foreign currency without a foreign amount is refused with "You can't provide a foreign currency without specifying an amount in 'Amount in Currency' field."
**And** setting the foreign currency to the journal currency is refused with "The foreign currency must be different than the journal one: C".

### F.5 The ordering index

**Given** two transactions on 15 March 2026: T1 with sequence 1 and identifier 42, T2 with sequence 5 and identifier 43
**Then** T1's index is `2026031521474836460000000042` and T2's is `2026031521474836420000000043`
**And** T2 sorts **before** T1, because a higher sequence yields a smaller complement.

### F.6 The residual of an unchecked transaction

**Given** a transaction of +1 500.00 whose entry has already been partially allocated, but whose entry is **not checked**
**Then** the residual is −1 500.00, the whole amount, regardless of what the journal items say.

### F.7 The residual of a checked transaction on a reconcilable suspense account

**Given** a transaction of +1 500.00, checked, whose suspense account is reconcilable, whose suspense line has been matched for 1 000.00
**Then** the residual is the suspense line's foreign residual, that is −500.00
**And** the reconciled flag is false.

### F.8 Full allocation clears the suspense group

**Given** a transaction of +1 500.00
**When** the whole amount is allocated to real accounts
**Then** the entry has no suspense line, the residual is 0.00 and the reconciled flag is true.

### F.9 A zero transaction is reconciled

**Given** a transaction created with no amount
**Then** its amount is 0.00 and its reconciled flag is true.

### F.10 Exactly one liquidity line

**Given** a transaction's entry
**When** a second line on the journal's default account is added
**Then** the write is refused with "The journal entry <the entry name> reached an invalid state regarding its related statement line.\nTo be consistent, the journal entry must always have exactly one journal item involving the bank/cash account."

### F.11 At most one suspense line

**Given** a transaction's entry
**When** a second line on the journal's suspense account is added
**Then** the write is refused with "<the entry name> reached an invalid state regarding its related statement line.\nTo be consistent, the journal entry must always have exactly one suspense line."

### F.12 Editing the amount discards the allocation

**Given** a reconciled transaction of +1 500.00 whose entry has two counterpart lines
**When** the user changes the amount to +1 600.00
**Then** the entry is rewritten with the liquidity line at 1 600.00 and one suspense line at −1 600.00; every other line is deleted and the matchings that used them disappear
**And** the transaction becomes unreconciled.

### F.13 Deleting a transaction under a restrictive audit trail

**Given** a company with a restrictive audit trail and a transaction that belongs to no statement
**When** the user deletes the transaction
**Then** the transaction record is removed and its entry is **cancelled**, not deleted.

### F.14 Deleting a transaction of a valid, complete statement

**Given** a statement that is both valid and complete
**When** the user tries to delete one of its transactions
**Then** the deletion is refused with "You can not delete a transaction from a valid statement.\nIf you want to delete it, please remove the statement first."

### F.15 The default date of a new transaction

**Given** a journal whose latest posted transaction is dated 20 March and belongs to a statement dated 31 March
**When** a user creates a new transaction on that journal
**Then** the date defaults to 31 March, the statement's date
**And** when that transaction had no statement, the date would default to 20 March instead.

---

## G. Bank statements

### G.1 A first statement

**Given** the journal *Bank* with no statement, and two posted transactions of +1 000.00 and −250.00
**When** the user groups them into a statement
**Then** the starting balance is 0.00, the computed ending balance is 750.00, the reported ending balance is initialised to 750.00
**And** the statement is complete and valid
**And** its reference is `BNK1 Statement <the date of the last posted transaction>`.

### G.2 A second statement anchored on the first

**Given** the statement of G.1 with a reported ending balance of 750.00, and a further transaction of +400.00
**When** the user groups that transaction into a second statement
**Then** the starting balance is 750.00, the computed ending balance is 1 150.00
**And** the second statement is valid.

### G.3 A transaction outside any statement

**Given** the two statements of G.2, and a transaction of −100.00 that sits between the second statement and a new third one of +50.00
**When** the third statement is created
**Then** its starting balance is `1 150.00 − 100.00 = 1 050.00` and its computed ending balance is 1 100.00.

### G.4 An incomplete statement

**Given** the second statement of G.2
**When** the user sets the reported ending balance to 1 145.00
**Then** the statement is not complete and its problem description is "The running balance (1 150.00) doesn't match the specified ending balance."

### G.5 An invalid following statement

**Given** the situation of G.4
**Then** the third statement is not valid, because its starting balance of 1 050.00 differs from `1 145.00 − 100.00 = 1 045.00`
**And** its problem description is "The starting balance doesn't match the ending balance of the previous statement, or an earlier statement is missing."
**And** the journal is flagged as having invalid statements and the dashboard card shows the *Invalid Statement(s)* warning.

### G.6 Non-contiguous selection

**Given** four consecutive posted transactions T1, T2, T3, T4
**When** the user selects T1, T2 and T4 and asks for a statement
**Then** the creation is refused with "Unable to create a statement due to missing transactions. You may want to reorder the transactions before proceeding."

### G.7 A cancelled transaction inside the range is absorbed

**Given** four transactions where T3 is cancelled
**When** the user selects T1, T2 and T4
**Then** T3 is added to the selection so the run is unbroken, and the statement is created
**And** T3 contributes nothing to the computed ending balance.

### G.8 Transactions of two journals

**Given** two transactions of two different journals
**When** the user selects both and asks for a statement
**Then** the creation is refused with "A statement should only contain lines from the same journal."

### G.9 Splitting

**Given** a run of five transactions of which the first two already belong to a statement
**When** the user asks to split at the fourth
**Then** the third and the fourth are selected — every transaction after the previous statement's last one and up to the chosen one.

### G.10 A draft transaction does not count

**Given** a statement with one posted transaction of +500.00 and one draft transaction of +200.00
**Then** the computed ending balance is the starting balance plus 500.00, and the statement's date is the date of the posted transaction.

### G.11 The running balance is anchored on statements

**Given** the three statements of G.3
**When** the running balance of the third statement's transaction is displayed
**Then** it is computed from the second statement's starting balance forwards, not from the beginning of the journal's history.

---

## H. Reconciling bank transactions

### H.1 Matching a transaction with an invoice

**Given** an open customer invoice of 1 500.00 and a transaction of +1 500.00
**When** the user matches them
**Then** the transaction's suspense line is replaced by a line on **Accounts Receivable** of 1 500.00 credit
**And** that line is matched with the invoice's receivable line
**And** the transaction is reconciled, the invoice is `paid`.

### H.2 Matching a transaction with an outstanding payment

**Given** a Payment in `in_process` of 1 000.00 whose liquidity line sits on **Outstanding Receipts**, and a transaction of +1 000.00
**When** the user matches them
**Then** the transaction's suspense line is replaced by a line on **Outstanding Receipts** of 1 000.00 credit
**And** that line is matched with the Payment's liquidity line
**And** the outstanding account returns to zero, the Payment's matched flag becomes true, its state becomes `paid`, and every document it settles moves from `in_payment` to `paid`.

### H.3 A transaction of 1 500 matched to a foreign-currency invoice at a different rate

**Given** the company keeps its books in **C**, the journal *Bank* is held in **F**
**And** a customer invoice of 1 500.00 **F** booked at 2 000.00 **C**
**And** a transaction of +1 500.00 **F** whose company amount at the transaction date is 1 875.00 **C**
**When** the user matches them
**Then** the transaction's entry becomes:

| Account | Debit (C) | Credit (C) | Currency | Foreign amount (F) |
|---|---|---|---|---|
| Bank | 1 875.00 | | F | +1 500.00 |
| Accounts Receivable | | 1 875.00 | F | −1 500.00 |

**And** one matching is created for 1 875.00 C / 1 500.00 F / 1 500.00 F
**And** one exchange-difference entry is created:

| Account | Debit (C) | Credit (C) |
|---|---|---|
| Accounts Receivable | | 125.00 |
| Foreign Exchange Loss | 125.00 | |

**And** the invoice ends with a residual of 0.00 C and 0.00 F and the payment state `paid`.

### H.4 A partial match

**Given** an open invoice of 1 500.00 and a transaction of +1 000.00
**When** the user matches them
**Then** the counterpart line is 1 000.00, the invoice keeps a residual of 500.00 and becomes `partial`
**And** the transaction is fully reconciled, because its whole amount has been explained.

### H.5 A partial match against a transfer-file payment is refused

**Given** a Payment whose payment method code starts with `iso20022`, of 1 000.00, and a transaction of +600.00
**When** the user tries to match them partially
**Then** the partial allocation is not offered: such a payment must be matched in full.

### H.6 Creating a bank account from the reported number

**Given** a transaction whose counterparty is *Northwind* and whose reported account number is `BE62 5100 0754 7061`, and no such Bank Account exists
**When** the transaction is reconciled
**Then** a Bank Account with that number is created for *Northwind* with the trust flag off.

### H.7 The creation can be disabled

**Given** the same setting, with the system parameter `account.skip_create_bank_account_on_reconcile` set to a true value
**When** the transaction is reconciled
**Then** no Bank Account is created; only a search is made for an existing one with that number, that counterparty and a compatible company.

### H.8 Creating an account for the company's own partner is refused

**Given** a transaction whose counterparty is the company's own partner and whose reported number is unknown
**When** the find-or-create algorithm runs without the company-creation flag
**Then** it fails with "Please add your own bank account manually: <the number> (<the partner display name>)".

### H.9 Undoing a reconciliation

**Given** the reconciled transaction of H.1
**When** the user undoes the reconciliation
**Then** every matching on its journal items is removed, every auto-generated Payment is deleted, the entry's lines are replaced by the default liquidity and suspense pair, the residual returns to −1 500.00 and the reconciled flag becomes false
**And** the invoice returns to `not_paid` with a residual of 1 500.00.

### H.10 Undoing on a checked transaction by a non-accountant

**Given** a checked, reconciled transaction and a user who may not review entries
**When** the user tries to undo the reconciliation
**Then** it is refused with "Validated entries can only be changed by your accountant."

### H.11 Undoing reverses the exchange entry

**Given** the reconciled pair of H.3, whose exchange entry is posted
**When** the reconciliation is undone
**Then** the exchange entry is reversed with a cancel flag, dated on its own date or on the day after the latest lock date it would violate, with the reference "Reversal of: <the exchange entry number>".

### H.12 Candidate items exclude a payment's trade line

**Given** a Payment whose counterpart line sits on **Accounts Receivable**
**When** the candidate journal items for a transaction are listed
**Then** that line is **not** offered, because a payment's trade line is excluded from the candidates; only its outstanding line is matchable.

---

## I. Reconciliation models

### I.1 A model with a percentage counterpart

**Given** the model *Line with Bank Fees*: label condition *contains* `BRT`; line 1 "Due amount" on an income account, mode `regex`, text `BRT: ([\d,.]+)`; line 2 "Bank Fees" on a finance-expense account, mode `percentage`, text `100`
**And** a transaction labelled `R:9672938 10/07 AX 9415126318 T:5L:NA BRT: 3358,07 C:` of +3 350.00
**When** the model is applied
**Then** the entry becomes:

| Account | Debit | Credit | Label |
|---|---|---|---|
| Bank | 3 350.00 | | the bank's own label |
| Income | | 3 358.07 | Due amount |
| Finance Expense | 8.07 | | Bank Fees |

**And** the open balance reaches 0.00, so the transaction is reconciled.

### I.2 Line ordering matters

**Given** the same model with the two lines in the opposite order
**When** the model is applied to the same transaction
**Then** the percentage line consumes the whole 3 350.00 first and the regular-expression line adds 3 358.07, leaving an open balance of −3 358.07 and an unreconciled transaction.

### I.3 The two percentage modes are not interchangeable

**Given** the model of I.1 with line 2 using `percentage_st_line` instead of `percentage`
**When** it is applied to the same transaction
**Then** line 2's amount is `3 350.00 × 100 ÷ 100 = 3 350.00` and the open balance ends at `8.07 − 3 350.00 = −3 341.93`, leaving the transaction unreconciled.

### I.4 A regular expression that does not match

**Given** the model of I.1 and a transaction whose label contains `BRT` but no numeric group after it
**When** the model is applied
**Then** line 1 produces nothing, line 2 produces the whole open balance, and the transaction is reconciled entirely onto the finance-expense account.

### I.5 A fixed amount

**Given** a model with one line of mode `fixed` and amount `−25`
**When** it is applied to a transaction of +100.00
**Then** the line produces a **debit** of 25.00 (a negative fixed amount counts as a debit), and 75.00 remains on the suspense account.

### I.6 Matching on the amount

**Given** a model whose amount condition is `between` with the bounds 100.00 and 200.00
**Then** it applies to a transaction of −150.00 (the absolute value is used) and not to one of 250.00.

### I.7 Matching on the label

**Given** a model whose label condition is `not_contains` with the parameter `FEE`
**Then** it applies to a transaction labelled "Transfer Northwind" and not to one labelled "Monthly fee" (the comparison ignores letter case).

### I.8 Matching on a regular expression

**Given** a model whose label condition is `match_regex` with a parameter that does not compile
**When** the model is saved
**Then** the save is refused with "The regex is not valid".

### I.9 Ordering of models

**Given** two models that both match a transaction, with sequences 5 and 10
**Then** the one with sequence 5 applies. When both have sequence 5, the one with the smaller identifier applies.

### I.10 A partner mapping

**Given** a model with a label condition *contains* `NORTHW`, exactly one line naming the counterparty *Northwind* and no account
**When** a transaction labelled "SEPA NORTHW 0012" with no counterparty is processed
**Then** the transaction's counterparty is set to *Northwind*, no counterpart line is produced, and the search continues for a counterpart model
**And** the mapping's *can be proposed* flag is false, so it is never offered as a proposal.

### I.11 A model with no condition is never proposed

**Given** the shipped *Internal Transfers* model, which has no condition and a manual trigger
**Then** its *can be proposed* flag is false: it can only be chosen explicitly by the user.

### I.12 An automated model

**Given** a model whose trigger is `auto_reconcile` and that matches a transaction
**When** the transaction is processed
**Then** the counterpart lines are written and reconciled without asking, and the entry is also marked as **checked**.

### I.13 A next activity

**Given** a model naming a next activity type
**When** it is applied
**Then** an activity of that type is scheduled on the transaction.

### I.14 Zero amounts are refused

**Given** a model line
**When** the mode is `fixed` and the amount text is `0` → "The amount is not a number"
**And** when the mode is `percentage` and the amount text is `0` → "Balance percentage can't be 0"
**And** when the mode is `percentage_st_line` and the amount text is `0` → "Statement line percentage can't be 0"
**And** when the mode is `regex` and the text does not compile → "The regex is not valid".

### I.15 Changing the mode resets the text

**Given** a line with mode `fixed` and text `250`
**When** the user changes the mode to `percentage`
**Then** the text becomes `100`
**And** changing it to `regex` sets the text to `([\d,]+)`
**And** changing it back to `fixed` clears the text.

### I.16 Duplicating a model

**Given** a model named "Bank Fees" and another already named "Bank Fees (copy)"
**When** the user duplicates the first
**Then** the copy is named "Bank Fees (copy) (copy)".

### I.17 Journal restriction

**Given** a model whose journal list contains only *Cash*
**Then** it never applies to a transaction of *Bank*
**And** a model with an empty journal list applies to every liquidity journal.

---

## J. Reconciliation mechanics

### J.1 A chain of partial reconciliations reaching full reconciliation

**Given** a customer invoice of 1 000.00 settled by three payments of 300.00, 300.00 and 400.00 on three consecutive days

**Day 1:**
**When** the first payment is matched
**Then** one matching of 300.00 is created; the invoice's residual is 700.00; no Full Reconciliation exists; the invoice and the payment share a partial matching number of the form `P` followed by the first matching's identifier; the invoice's payment state is `partial`.

**Day 2:**
**When** the second payment is matched
**Then** a second matching of 300.00 is created; the invoice's residual is 400.00; the connected group is the invoice and the two payments; the matching number is unchanged, because the component keeps the smallest matching identifier; the payment state is still `partial`.

**Day 3:**
**When** the third payment is matched
**Then** a third matching of 400.00 is created; every item of the group has a zero residual and at least one matching, so one Full Reconciliation is created, linking the three matchings and the four items
**And** every item's matching number becomes the decimal text of the Full Reconciliation's identifier
**And** the invoice's payment state becomes `in_payment`, and `paid` once all three payments are confirmed by the bank.

**Undoing day 3:**
**When** the third matching is deleted
**Then** the Full Reconciliation is deleted first; the surviving component keeps the smallest matching identifier, so the invoice and the first two payments return to that partial number; the third payment, now without any matching, ends with no matching number; the invoice returns to `partial` with a residual of 400.00.

### J.2 The matching number merges two components

**Given** items A and B matched by matching 7, and items C and D matched by matching 9
**When** a matching 12 links B and C
**Then** the two components merge into one whose number is `P7`, the smaller of the two component numbers, and all four items carry it.

### J.3 An item with no matching never closes a group

**Given** a group whose items all have a zero residual, but one of them — an exchange-difference line with a foreign amount and a zero balance — has no matching at all
**Then** no Full Reconciliation is created.

### J.4 Already reconciled

**Given** two items that are already fully reconciled with each other
**When** the user asks to reconcile them again
**Then** it is refused with "You are trying to reconcile some entries that are already reconciled."

### J.5 A cancelled entry

**Given** one open item and one item of a cancelled entry
**When** the user asks to reconcile them
**Then** it is refused with "You can not reconcile cancelled entries."

### J.6 Two accounts

**Given** one item on **Accounts Receivable** and one on **Accounts Payable**
**When** the user asks to reconcile them
**Then** it is refused with "Entries are not from the same account: Accounts Payable, Accounts Receivable".

### J.7 Two companies

**Given** two items of two unrelated companies, both on an account with the same code
**When** the user asks to reconcile them
**Then** it is refused with "Entries don't belong to the same company: <the two company names>".

### J.8 A non-reconcilable account

**Given** two items on an expense account that is not reconcilable and whose type is neither cash nor credit card
**When** the user asks to reconcile them
**Then** it is refused with "Account <the account name> does not allow reconciliation. First change the configuration of this account to allow it."

### J.9 A partly reconciled group can be reconciled again

**Given** a group of three items where two are already fully matched and the third has a residual, all sharing a partial matching number
**When** the whole group plus a new item is passed to the reconciliation
**Then** the already fully matched items are set aside by the narrowing rule and the operation proceeds without the "already reconciled" refusal.

### J.10 The currency split

**Given** four items: two in **F** and two in **C**, all on the same reconcilable account, passed as one set
**When** the reconciliation runs
**Then** the plan is split: the two **F** items are matched together first, then the two **C** items, and only afterwards is what remains matched across currencies.

### J.11 The counterparty sort

**Given** a set of items belonging to two counterparties
**When** the reconciliation runs
**Then** the items are re-sorted by counterparty identifier (items without a counterparty first) before being paired, so each counterparty's debits meet that counterparty's credits.

### J.12 The residual formula

**Given** an invoice line of +1 000.00 matched for 400.00 and then for 550.00
**Then** its company residual is `1 000.00 − 950.00 = 50.00` and it is not reconciled.

### J.13 Deferred matching on import

**Given** four journal items imported with the matching number `IMPORT-A`, on an account that is not reconcilable, and two of their entries still in draft
**When** the deferred-matching operation runs
**Then** nothing happens, because not every entry of the group is posted
**And** once every entry is posted, the account is made reconcilable and the four items are matched together, with exchange differences and cash-basis entries disabled.

### J.14 An import placeholder cannot be used in a real matching

**Given** an item whose matching number is `IMPORT-A`
**When** a real matching is created on it without clearing the number
**Then** the constraint refuses with "A temporary number can not be used in a real matching".

### J.15 A missing exchange journal

**Given** a company with no exchange-difference journal
**When** a multi-currency matching that needs an exchange entry is attempted
**Then** it fails with "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."

### J.16 A missing exchange loss account

**Given** a company with an exchange journal but no expense exchange account
**Then** the same attempt fails with "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."

### J.17 A missing exchange gain account

**Given** a company with an exchange journal and an expense exchange account but no income exchange account
**Then** the same attempt fails with "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."

### J.18 Cash-basis taxes without a journal

**Given** a company that uses cash-basis taxes and has no cash-basis journal
**When** a matching on a receivable account is attempted for a document carrying cash-basis taxes
**Then** it fails with "There is no tax cash basis journal defined for the '<the company name>' company.\nConfigure it in Accounting/Configuration/Settings"

### J.19 The exchange entry is posted only when both sides are posted

**Given** a matching between an item of a posted entry and an item of a draft entry
**When** an exchange difference is produced
**Then** the exchange entry is created but left in draft.

### J.20 Unreconciling deletes a draft exchange entry

**Given** the situation of J.19
**When** the matching is deleted
**Then** the draft exchange entry is deleted outright, not reversed.
