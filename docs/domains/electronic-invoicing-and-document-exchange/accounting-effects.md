# Accounting effects

This domain creates no journal entry of its own, posts nothing, reverses nothing and holds no account. It is a transport and translation layer: it turns an accounting document that already exists into a file, and it turns a file into an accounting document that is left in draft for a human or for the automatic posting rule to accept. Everything that follows describes the boundary precisely, because a replacement must not accidentally give this domain a ledger of its own, and must not accidentally lose the several places where it does change what the ledger will later contain.

---

# 1. What this domain never does

| It never | Reason |
|---|---|
| creates a journal entry or a journal item | the only records it creates on the accounting side are draft documents built from an inbound file, and their lines |
| posts an accounting document | posting is always an act of the general ledger, triggered by a user or by the automatic posting rule of the accounts payable domain |
| computes a tax amount | every amount comes from the tax engine of the [taxes](../taxes/calculations.md) domain; this domain only chooses grouping keys, signs and presentation precision |
| selects an account from a product, a product category, a fiscal position or a company default | the account of an imported line is predicted by the accounts payable domain from previous documents, and is otherwise left empty so that the ordinary rules of the general ledger apply |
| creates an analytic amount or an analytic distribution | none of its records carries one |
| reconciles anything | it only reads the reconciliation of a credit note in order to write the preceding invoice reference, and reads the reconciled payments in order to write the payment means code |
| reverses a document | a cancellation withdrawn from the network cancels the accounting document; it never reverses it |
| holds a currency of its own | every amount is expressed in the currency of the accounting document or in the company currency, both read from that document |

---

# 2. Where this domain changes what the ledger will contain

## 2.1 It refuses a posting

When an accounting document is posted, one delivery record is created for every applicable format of its journal, and the formats that need no remote call are generated immediately. When a generation produces a validation message whose blocking level is the error level, the posting itself is refused. The document therefore never reaches the ledger, and no journal entry is written. The complete list of those validations, with their exact messages, is in [business-rules.md](business-rules.md).

This is the only way in which this domain prevents an accounting fact from being recorded. A replacement that moved those validations after the posting would let unposted-by-intention documents into the ledger.

## 2.2 It cancels an accounting document

When a cancellation requested from the remote service succeeds, the accounting document is cancelled. Cancelling a posted document is an act of the general ledger, with the consequences the general ledger defines: the journal items are dropped or the entry is marked cancelled according to the rules of that domain. This domain only supplies the trigger.

A forced cancellation does the same without waiting for the remote service, and is offered only when the format allows it.

## 2.3 It blocks a reset to draft and a resequencing

An accounting document that counts as sent over the exchange network may not be reset to draft and may not be resequenced. Both guards protect the ledger from a change that the trading partner and the network already saw. The resequencing refusal names the affected documents: `The following documents have already been sent and cannot be resequenced: %s`.

## 2.4 It creates draft accounting documents from inbound files

An inbound file becomes a draft vendor bill, a draft vendor credit note, a draft customer invoice or a draft customer credit note. The draft carries:

| Written by this domain | Consequence once the document is posted |
|---|---|
| the partner | decides the receivable or payable counterpart account of the journal entry, through the ordinary rules of the general ledger |
| the currency and the conversion rate read for the invoice date | decides the company currency amounts of every journal item |
| the invoice date and the due date | decide the accounting date and the payment term lines |
| the lines with their quantity, unit price, discount and taxes | decide the revenue or expense amounts and the tax amounts |
| the account of a line, when the prediction resolved one | decides the revenue or expense account of that journal item |
| the taxes of a line, when the matching resolved one | decide the tax journal items and their tax grid tags |
| the recipient bank account | is carried onto the payment |
| the delivery terms | carried as information |
| the extra line labelled `Rounding` produced by the untaxed amount correction | becomes an ordinary journal item on the account the ordinary rules resolve for a line without a product |

Nothing is posted. The document waits for a user, or for the automatic posting rule of the accounts payable domain, which posts a vendor bill when the same vendor has sent the same shape of document three times in a row without a manual change.

## 2.5 It rewrites the tax amounts of a draft document

After an inbound file has been written onto a draft document, two correction passes run.

1. The **tax amount correction** redistributes the computed tax amount of each tax group so that it equals the amount stated in the file, and updates the tax lines of the draft document accordingly. It runs only when every tax group of the file resolved a real tax and the discrepancy is at most three hundredths of the currency unit.
2. The **untaxed amount correction** appends one line labelled `Rounding` carrying the difference between the tax exclusive amount stated in the file, increased by the payable rounding amount stated in the file, and the untaxed amount the draft document computed.

Both passes change the future journal entry. Both are specified with worked examples in sections 14 and 15 of [calculations.md](calculations.md). Both run inside a scope in which the balance check, the discount precision limit and the dynamic line synchronisation are suspended, so that an intermediate unbalanced state is never validated; the document is balanced again when the scope closes.

## 2.6 It creates master data that the ledger then uses

| Record created | When | Effect on later accounting |
|---|---|---|
| a partner | an inbound file names a partner that could not be matched and carries both a name and a tax identification number | every later document of that partner uses it |
| a bank account on a partner or on the company | an inbound file names a payee account number that is not yet known | a payment registered later may use it |
| a tax identification number written onto an existing partner that had none | an inbound file supplies one and the partner had none | the fiscal position rules of the [taxes](../taxes/calculations.md) domain may resolve differently afterwards |

## 2.7 It reacts to two accounting events

| Event | Reaction |
|---|---|
| a vendor bill that may still be answered is posted | an approval response is sent to the document originator over the exchange network |
| a vendor bill that may still be answered is cancelled | the rejection form is opened; on send, a rejection response is sent and the result of the cancellation is returned so that the cancellation completes |

Neither reaction changes the accounting document; both are acts of communication that happen to be triggered by an accounting event.

---

# 3. What it reads from the ledger

The export mapping reads, from the accounting document and from the tax engine: the document name, the dates, the currency, the partner, the shipping address, the company, the journal, the fiscal position, the payment terms, the recipient bank account, the payment reference, the terms and conditions, the delivery date, the deferred dates of the lines, the reconciled payments, the reconciled preceding invoices of a credit note, the total, the residual amount, the signed company currency total, the signed company currency residual amount, and the complete set of base lines with their tax details. It writes none of them.

The residual amount and the total are what produce the prepaid amount and the payable amount of the exported file, which is the only place where a payment already registered in the ledger shows up in a structured file.

---

# 4. Cross domain boundary

| Domain | What it owns that this domain depends on |
|---|---|
| [general ledger](../general-ledger/README.md) | the accounting document, its posting, its cancellation, its reset to draft, its sequence and its resequencing, and the journal |
| [taxes](../taxes/calculations.md) | every tax amount, every base amount, every rounding of those amounts, and the aggregation helpers |
| [accounts receivable](../accounts-receivable/README.md) | the sending service, the printed invoice report, the sending method selection and the electronic mail message |
| [accounts payable](../accounts-payable/README.md) | the account prediction, the tax prediction, the partner matching strategies and the automatic posting rule |
| [payments and bank reconciliation](../payments-and-bank-reconciliation/README.md) | the bank account record, the direct debit mandate that decides the payment means code, and the reconciliation that decides the residual amount |
| [multi-currency](../multi-currency/README.md) | the conversion rate read at import and the company currency amounts of the export |
