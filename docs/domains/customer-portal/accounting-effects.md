# Accounting effects of the Customer Portal

The Customer Portal creates no journal entry, changes no journal item, computes no valuation and writes no analytic amount. Several portal actions are nevertheless financially meaningful: a customer signs a quotation, pays an invoice, acknowledges a purchase order or deletes an account from a portal page. Each of those hands its financial consequence to the domain that owns the document, and the boundary between the portal action and that consequence is drawn action by action in the tables that follow.

## 1. What this domain does and does not write

| The portal does | The portal does not |
|---|---|
| Render a document and its report for a reader who holds a security token. | Change any amount, tax, currency or account on that document. |
| Collect a customer signature and store it on the document. | Post anything as a result of the signature. |
| Hand the payment context (amount, currency, paying Contact, compatible providers, transaction address) to the embedded payment form. | Create the payment transaction, the payment, or the journal entry that records it. |
| Redirect the reader back to the document after a payment attempt. | Reconcile anything. |
| Show an overdue alert card and an overdue filter. | Compute the due amounts; it reads the payment state and the due date computed elsewhere. |
| Offer a download of the legal electronic document of an invoice. | Produce that document. |
| Show a loyalty balance in the account sidebar. | Award, spend or expire loyalty points. |

## 2. Where each financially meaningful portal action produces its effect

| Portal action | Effect produced by | Reference |
|---|---|---|
| Accepting and signing a quotation | The order confirmation of the sales domain, which may create a delivery, a down-payment invoice and analytic entries according to its own rules. | [Sales](../sales/README.md) |
| Rejecting a quotation | The order cancellation of the sales domain. No accounting effect unless the order had already produced documents, in which case the sales domain's own cancellation rules apply. | [Sales](../sales/README.md) |
| Paying an order or an invoice | The transaction state machine of the payment domain, and then the payment and reconciliation rules of the payments domain. The portal only supplies the amount, the currency, the paying Contact and the return addresses. | [Payment Providers](../payment-providers/README.md), [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) |
| Downloading an invoice | Nothing. The rendering of a portable document does not change the invoice; the pro-forma variant is used when no legal document exists yet, and it is not stored. | [Accounts Receivable](../accounts-receivable/README.md) |
| Acknowledging a purchase order on the vendor side | The acknowledgement operation of the purchasing domain. | [Purchasing](../purchasing/README.md) |
| Proposing new dates on a purchase order line | The date update of the purchasing domain, which may re-plan the related procurement. No accounting effect. | [Purchasing](../purchasing/README.md) |
| Editing a task in a shared project | The task write rules of the projects domain; the analytic effect of a timesheet, if any, belongs to the timesheets domain. | [Projects and Tasks](../projects-and-tasks/README.md), [Timesheets](../timesheets/README.md) |
| Deleting a portal account | Nothing financial. The deletion queue refuses to remove a Contact that a document still references, and leaves it archived. | [Identity and Access](../identity-and-access/README.md) |

## 3. Currency in the portal

The portal never converts an amount. Every amount shown on a record page is read from the document in the document's own currency, and is formatted with the document's currency for display. The payment context is handed to the payment form with the document's currency, not the reader's. When the reader's company differs from the document's company, the page refuses to show the payment form and shows the "switch company" warning instead, precisely so that no payment is ever created in the wrong company.

## 4. One consequence a replacement must preserve

Because signing a quotation may confirm the order, and confirming an order may produce an invoice and its journal entry, a replacement must run the signature endpoint inside the same transaction boundary as the confirmation it triggers. The shipped behavior writes the signature, flushes so that the stored signature is visible to the report renderer, confirms the order when no payment is required, renders the report including the signature, and posts it as an attachment on a public comment. A failure at any of those steps must leave the order unsigned and unconfirmed, not signed and unconfirmed.
