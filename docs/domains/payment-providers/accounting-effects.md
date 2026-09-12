# Accounting effects

What a Payment Transaction causes in the ledger, when it causes it, how every account is selected, how the resulting Payment is reconciled, and where the boundary with the Payments and Bank Reconciliation domain lies.

---

## 1. The boundary

This domain never writes a journal item itself. It creates a **Payment**, and the Payment domain turns that Payment into a balanced Journal Entry according to its own rules ([../payments-and-bank-reconciliation/accounting-effects.md](../payments-and-bank-reconciliation/accounting-effects.md)). What belongs here is:

- **when** a Payment is created (the triggering transaction state and the guards),
- **with which values** it is created (direction, amount, currency, contact, journal, payment method line, memo, destination account, write-off lines),
- **against what** it is reconciled,
- and what happens when the transaction is cancelled or refunded.

Everything that concerns the layout of the journal entry, the outstanding accounts, the exchange differences and the matching algorithm belongs to the Payments and Bank Reconciliation domain.

When the Accounting Payments capability package is not installed, this domain has **no accounting effect at all**: a confirmed transaction only flags itself as post-processed and lets the sales and point of sale packages act on their own documents.

---

## 2. Triggering events

| Event | Condition | Accounting consequence |
|---|---|---|
| A transaction reaches `done` | Always | Every linked invoice that is still a draft is posted. |
| A transaction reaches `done` | The operation is not `validation`, the transaction has no Payment yet, and no child of it is in state `done` or `cancel` | A Payment is created and posted (section 3), then reconciled (section 5). |
| A transaction reaches `done` | A Payment exists after the step above | The message `The payment related to transaction <transaction link> has been posted: <payment link>` is logged on every linked document. |
| A transaction reaches `cancel` | The transaction has a Payment | That Payment is cancelled. |
| A transaction reaches `pending`, `authorized` or `error` | always | No accounting consequence. An authorized amount is a reservation at the provider; it is not money and it is not recorded. |
| A refund transaction reaches `done` | Same guards as any confirmed transaction | A Payment in the opposite direction is created and posted. |
| A capture child transaction reaches `done` | Same guards | A Payment is created for the captured amount and reconciled against the invoices of the **source** transaction. |
| A void child transaction reaches `cancel` | always | Nothing: a void releases a reservation, it moves no money. |

The guard "no child of it is in state `done` or `cancel`" is what prevents a double entry when an authorization has been split: the source transaction of a split authorization never produces a Payment of its own; its capture children do.

The guard "the operation is not `validation`" reflects that a validation amount is never included in a payout: it is either zero, or charged and immediately voided or refunded by the connector.

---

## 3. The Payment created from a transaction

**Inputs**: the confirmed transaction.

| Payment field | Value |
|---|---|
| `amount` | The absolute value of the transaction amount. A Payment amount is always zero or positive; the direction carries the sign. |
| `payment_type` | `inbound` when the transaction amount is strictly positive, `outbound` when it is negative. A refund transaction therefore always produces an outbound payment. |
| `currency_id` | The transaction currency. |
| `partner_id` | The **commercial contact** of the transaction's contact, in order that a payment made by a delivery address settles the invoices of the customer company. |
| `partner_type` | `customer`. |
| `journal_id` | The provider's `journal_id`. |
| `company_id` | The provider's company. |
| `payment_method_line_id` | The inbound payment method line of that journal whose `payment_provider_id` is the transaction's provider. |
| `payment_token_id` | The transaction's token, when it has one. |
| `payment_transaction_id` | The transaction. |
| `memo` | The transaction reference, then a space, a hyphen and a space, then the provider reference, or nothing when there is no provider reference. |
| `invoice_ids` | The invoices linked to the transaction. |
| `write_off_line_vals` | Empty, except for the early payment discount case of section 4. |
| `destination_account_id` | The account of the first payment-term journal item found on the linked invoices, when there is one; otherwise the Payment domain's own default (the receivable account of the contact). |

The Payment is created and then posted at once. Its identifier is written back on the transaction's `payment_id` field, which makes the relation one-to-one in practice.

### 3.1 The resulting journal entry, for an inbound payment

| Account | Debit | Credit |
|---|---|---|
| The outstanding receipts account of the payment method line (see 3.3) | amount | |
| The destination account (the receivable account of the customer, or the account of the invoice's payment-term item) | | amount |

### 3.2 The resulting journal entry, for an outbound payment (a refund)

| Account | Debit | Credit |
|---|---|---|
| The destination account (the receivable account of the customer) | amount | |
| The outstanding receipts account of the payment method line | | amount |

### 3.3 How the outstanding account is chosen

The outstanding account sits on the payment method line, not on the provider. It is set once, when the line is created for the provider, by this rule:

1. When the provider's code is `custom`, no outstanding account is set at all. A custom provider (wire transfer, cash on delivery, pay on site) does not collect money itself, therefore the Payment it would produce must use the journal's own default rather than a provider-specific holding account. In practice a custom provider never confirms a transaction by itself, therefore this branch is only reached when an employee confirms it manually.
2. Otherwise, when another payment method line of the same company already uses the same code, the new line reuses that line's outstanding account, in order that all the lines of one provider share one holding account.
3. Otherwise the account is the chart of accounts' outstanding receipts account when the accounting payment method is inbound, or its outstanding payments account when it is outbound.
4. When the chart of accounts defines neither, the company's internal transfer account is used.

**Precedence, from strongest to weakest**: an account already chosen by an existing line of the same code; the chart of accounts' outstanding account for the direction; the company's internal transfer account. Nothing on the product, the product category or the fiscal position takes part in this choice, because a payment is not a sale.

An administrator may change the outstanding account of the line afterwards; from then on it is the line's own value that is used, because the rule above only runs when the line is created.

---

## 4. Early payment discount

When a confirmed transaction pays exactly the discounted amount of an invoice that is still eligible for its early payment discount, the Payment must settle the whole receivable, not only the amount received, and the difference must be recognised as a discount.

1. For each posted invoice linked to the transaction, read the invoice's next-payment values.
2. When the installment state of those values is the early payment discount state **and** the transaction amount equals the amount due of those values exactly:
   1. take the journal item that carries the discount;
   2. build one counterpart line for it whose amount in currency is the opposite of that item's residual amount in currency, and whose balance is the opposite of that item's balance;
   3. ask the invoicing domain to build the counterpart items for an early payment discount with that line and with an open balance equal to the discount amount of the next-payment values;
   4. set the contact of the first resulting line to the invoice's contact;
   5. add those lines to the Payment's write-off lines;
   6. stop looking at further invoices.

The resulting journal entry therefore carries, besides the two lines of section 3.1, the discount lines built by the invoicing domain (a discount expense or a reduction of income, and the matching tax adjustment when the discount changes the tax base). The exact shape of those lines is owned by [../accounts-receivable/](../accounts-receivable/README.md).

**Worked example.** An invoice of 100.00 euro with a 2 percent early payment discount valid until the 10th. The customer pays 98.00 on the 8th. The transaction amount equals the discounted amount due, therefore the Payment is created with 98.00 and with write-off lines totalling 2.00. After posting, the invoice's receivable line of 100.00 is fully settled: 98.00 by the payment line and 2.00 by the discount line.

---

## 5. Reconciliation

1. Choose the invoices to reconcile against:
   - when the transaction's operation equals the operation of its source transaction (that is, for a capture or a void child), take the invoices of the **source** transaction;
   - otherwise (a normal payment, or a refund child, whose operation is `refund`) take the invoices of the transaction itself.
2. Drop the cancelled invoices.
3. Post the remaining invoices that are still drafts.
4. Take the journal items of the Payment's entry and of those invoices, keep those whose account is the Payment's destination account and that are not already reconciled, and reconcile them.

**Consequences.**

- A normal payment of a customer invoice settles that invoice and moves its payment state to "in payment" or "paid", according to the rules of [../accounts-receivable/](../accounts-receivable/README.md).
- A capture child settles the invoice of the original authorization, which is the desired behaviour: the customer only ever sees one invoice.
- A refund child has no invoices of its own, therefore nothing is reconciled automatically. An accountant matches the refund payment against a credit note afterwards. This is deliberate: the platform does not know which credit note, if any, the refund corresponds to.
- A Payment produced by a transaction may never be **partially** reconciled against a bank transaction; the bank reconciliation refuses the partial match and requires the whole payment to be matched at once.

---

## 6. Cancellation and reversal

| Situation | Effect |
|---|---|
| A transaction that had produced a Payment reaches `cancel` | The Payment is cancelled. Cancelling a posted Payment is the Payment domain's own operation: it reverses or resets the journal entry according to that domain's rules, and undoes the reconciliation. |
| A transaction reaches `error` after having produced a Payment | Nothing happens automatically. The only path into `error` from `done` is the Stripe refund reversal, and it concerns a refund transaction; the accountant must then reverse the refund Payment by hand. The connector's message states this explicitly: `The refund did not go through. Please log into your Stripe Dashboard to get more information on that matter, and address any accounting discrepancies.` |
| A void child reaches `cancel` | No Payment exists for it, therefore the cancellation branch finds nothing to cancel. |
| A refund is issued | A new outbound Payment is created; the original Payment is untouched, and the link between the two is kept through `source_payment_id` on the refund Payment. |

---

## 7. Analytic accounting

A Payment produced by this domain carries no analytic distribution. Money movements are not analysed; the analytic consequences of the sale or the purchase were recorded on the invoice. This is the behaviour of the Payment entity itself and is specified in [../analytic-accounting/](../analytic-accounting/README.md).

---

## 8. Currency handling

- The Payment is created in the **transaction's** currency, which is the currency the customer was charged in.
- When that currency differs from the company's accounting currency, the Payment domain converts the amount at the Payment's own date and records the difference between the invoice rate and the payment rate as an exchange difference when the two are reconciled. None of that is decided here.
- The provider's own settlement currency is invisible to the ledger: what the provider pays out, and in which currency, appears later as a bank transaction that is matched against the outstanding receipts account.
- The `maximum_amount` of a provider is expressed in the **company's** currency, therefore the payment amount is converted into the company currency before it is compared (see `calculations.md`, section 8).

---

## 9. Point of sale online payments

When a transaction linked to a point of sale order reaches `authorized` or `done`, the Payment is created by the same rule as any other transaction, and then:

1. a payment line is added to the point of sale order with the transaction amount, the moment of the last state change as its date, the configuration's online payment method, and the created Payment as its online payment;
2. the point of sale payment method, the order and the session are written on the Payment, in order that the point of sale session closing entry can account for it.

The session closing entry itself belongs to [../point-of-sale/accounting-effects.md](../point-of-sale/accounting-effects.md).

---

## 10. Summary of the ledger positions used

| Position | Role | Chosen by |
|---|---|---|
| Outstanding receipts account | Holds the money between the customer's payment and the provider's payout. | The payment method line of the provider (section 3.3). |
| Outstanding payments account | Holds the money between a refund being issued and the provider's debit. | The same rule, for an outbound accounting payment method. |
| Receivable account of the customer | The debt being settled. | The invoice's payment-term item when one exists, otherwise the Payment domain's default. |
| Bank account of the journal | Reached only later, when the provider's payout is matched with the outstanding account during bank reconciliation. | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md). |
| Early payment discount accounts | Recognise the discount granted. | [../accounts-receivable/](../accounts-receivable/README.md). |
| Internal transfer account | Last-resort outstanding account when the chart of accounts defines none. | Section 3.3, step 4. |

**Industry-standard completion.** The reference behaviour does not record the provider's own fee anywhere: the amount held in the outstanding receipts account is the gross amount the customer paid, and the difference with the net payout appears when the bank transaction is reconciled, where the accountant books the fee. A replacement should follow the same principle: never derive a fee from the transaction, because the fee is only known with certainty on the provider's settlement report.
