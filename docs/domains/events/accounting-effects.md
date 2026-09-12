# Accounting effects of the Events domain

This domain creates no journal entry, no journal item, no analytic amount and no valuation layer of its own. It sells seats and booths by putting ordinary products on ordinary sales documents, and every financial consequence is produced by the domains that own those documents. This file states the boundary precisely, lists the one place where the domain *reacts* to an accounting event, and records the industry-standard completions a replacement should consider.

## 1. What this domain does not do

| Not done here | Where it happens |
|---|---|
| Creating or posting a journal entry | [General ledger](../general-ledger/README.md) |
| Creating a customer invoice from a confirmed order | [Sales](../sales/README.md) and [Accounts receivable](../accounts-receivable/README.md) |
| Choosing the income account of a ticket or a booth | [Products and catalog](../products-and-catalog/README.md) for the product and product category accounts, and the general ledger domain for the precedence between them |
| Computing the tax of a ticket or a booth line | [Taxes](../taxes/calculations.md) |
| Applying a fiscal position to the taxes and accounts of a line | [Taxes](../taxes/README.md) |
| Registering a payment or reconciling it | [Payments and bank reconciliation](../payments-and-bank-reconciliation/README.md) |
| Posting the session entries of the shop counter | [Point of sale](../point-of-sale/README.md) |
| Converting amounts at the rate of the document date | [Multi-currency](../multi-currency/README.md) |
| Valuing stock | not applicable: tickets and booths are service products and carry no stock valuation |

No record of this domain is ever written by the posting of a journal entry, with the single exception described in section 3.

## 2. How money reaches the ledger

The chain is always the same, and every link of it lives outside this domain:

1. A ticket type points at a **product** whose service tracking is the event value; a booth category points at a **product** whose service tracking is the booth value. Both products are services, and both are set to invoice on **ordered quantities** when that service tracking is chosen.
2. A sales order line carries the product, the quantity in seats (or exactly one booth line), and a unit price taken from the ticket or from the booth category rather than from the product sales price (see the price rules in [calculations.md](calculations.md#8-ticket-and-booth-prices-with-taxes-and-discounts)).
3. Confirming the order creates the attendees or confirms the booths; it does **not** create any accounting document.
4. Invoicing the order, which is a decision of the sales domain, creates the customer invoice. Because the invoicing policy is "ordered quantities", the whole ordered quantity is invoiceable as soon as the order is confirmed, without waiting for the event to take place.
5. Posting that invoice creates the journal entry. The revenue account, the tax accounts and the receivable account are chosen entirely by the general ledger and taxes domains, with the usual precedence (the product, then its product category, then the fiscal position mapping, then the company defaults).
6. Registering a payment and reconciling it is again outside this domain.

The same chain applies at the shop counter, where the counter session produces the accounting documents of the counter domain; this domain only creates the attendees attached to the counter order lines.

**Analytic amounts.** Nothing in this domain sets an analytic distribution. When an organisation wants the cost and the revenue of an event gathered on one analytic account, the standard way is to set that distribution on the sales order lines or through an analytic distribution model keyed on the product or the customer; both belong to [Analytic accounting](../analytic-accounting/README.md).

## 3. The one inbound hook: paying an invoice marks booths as paid

This is the only point at which an accounting event writes on a record of this domain.

**Triggering event:** an invoice reaches the state "paid" and the paid hook of the invoice runs.

**Procedure:**

1. From the paid invoice, collect the sales order lines behind its invoice lines.
2. Among those lines, keep the ones whose product has the booth service tracking.
3. For every such line that already has confirmed booths, write `is_paid = true` on each of those booths, with elevated rights.

**Records written:** Event Booth (`is_paid` only). No journal entry, no journal item, no reconciliation is created by this step; the flag is an operational marker used by the organiser to distinguish a reserved booth from a settled one.

**Debit and credit table:** none. This step posts nothing.

**Reversal behaviour:** reversing or unreconciling the invoice does **not** clear `is_paid`. A replacement that wants the flag to follow the payment in both directions must add the symmetric rule explicitly; that would be an extension of the observed behaviour, not a reproduction of it.

## 4. Where the domain shows money without touching the ledger

Two figures of this domain look financial and must not be mistaken for accounting figures.

| Figure | Definition | Why it is not an accounting figure |
|---|---|---|
| Total sales of an event (`sale_price_total`) | the sum of the totals including tax of the confirmed sales order lines pointing at the event, converted into the currency of the company of the event at **today's** exchange rates | It uses today's rates rather than the rates of each document date, therefore it moves over time; it counts confirmed orders, not posted invoices; it includes tax. |
| The two measures of the revenue analysis (`sale_price`, `sale_price_untaxed`) | the line total, respectively the line subtotal, divided by the order currency rate and by the ordered quantity, giving an amount per seat | They are derived from order lines, not from journal items, and ignore whether the order was ever invoiced or paid. |

Both are steering figures for the organiser. The authoritative revenue of an event is the set of posted journal items of the invoices that carry its products.

## 5. Industry-standard completions

The following rules are **not** observed in the reference behaviour. They are standard accounting practice and a replacement should decide explicitly whether to implement them; they are listed here so that the boundary is not mistaken for an omission.

1. **Deferred revenue for tickets sold before the event.** Under accrual accounting, the consideration received for a ticket is a contract liability until the event takes place, because the performance obligation is satisfied on the event date. The standard treatment is to credit a deferred revenue account when the invoice is posted and to transfer the amount to the revenue account when the event ends. The reference behaviour recognises the revenue at invoice posting, using the ordinary income account of the ticket product. A replacement that needs the accrual treatment should attach a deferred revenue account to the ticket products and post the transfer entry from the event end date, which is available on the event.
2. **Refund of a cancelled seat.** Cancelling a registration releases the seat and nothing else. When money was taken for that seat, the credit note is produced through the ordinary sales and invoicing flow. A replacement should not attempt to reverse anything from the registration itself.
3. **Booth revenue.** Booths behave exactly like tickets: the revenue is recognised when the booth invoice is posted, and the `is_paid` flag of the booth is an operational marker only.
4. **Multi-company and inter-company events.** An event may belong to no company at all, which makes it visible to every company. The documents it generates always belong to the company of the sales order or of the counter session, never to the company of the event. A replacement should keep that separation: the event is an operational object, the order is the accounting object.
5. **Tax on a price that includes tax.** The ticket and booth prices stored in this domain are entered **excluding** tax, and the tax-included figures (`price_incl`, `price_reduce_taxinc`) are derived for display only. A replacement that lets organisers enter a tax-included price must convert to a tax-excluded unit price before writing the sales order line, otherwise the invoice totals will not match the advertised price.
