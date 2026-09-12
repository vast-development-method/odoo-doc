# Marketing accounting effects

This domain creates no journal entry, changes no valuation, and writes no analytic amount. Nothing in it debits or credits an account, reconciles anything, or produces a document that a bookkeeper must post. This file states the boundary precisely, because the domain does display monetary figures and does consume a paid external service.

## 1. What the domain never does

| Event | Accounting consequence |
|---|---|
| A mailing is created, tested, scheduled, sent, cancelled, retried or archived. | None. |
| A delivery record is created or changes status. | None. |
| A contact subscribes, unsubscribes, is blocked or is unblocked. | None. |
| A short link is created or visited. | None. |
| A campaign, source, medium, stage or tag is created or deleted. | None. |
| A marketing card is rendered, previewed, shared or deleted. | None. |
| A text message is sent and consumes credit of the messaging service. | None **in this domain**. See section 3. |

No entity of this domain carries an account field, a journal field, a tax field, an analytic distribution or a posted state. No operation of this domain opens, posts, reverses or reconciles anything.

## 2. Monetary figures the domain reads but never writes

Three figures of other domains are displayed inside marketing screens. They are read-only aggregates computed on demand; the domain never writes them back and never creates the records behind them.

| Figure | Where it appears | How it is obtained | Owning domain |
|---|---|---|---|
| Quotation count | On a mailing, on a campaign, and as a winner criterion | Number of sales orders having at least one line and whose Campaign Source is the mailing's source | [sales](../sales/) |
| Invoiced amount | On a mailing, on a campaign, as a winner criterion, and in the statistics message | Sum of the untaxed signed amounts of the customer invoices whose Campaign Source is the mailing's source and whose state is neither draft nor cancelled | [accounts-receivable](../accounts-receivable/) and [general-ledger](../general-ledger/) |
| Lead count | On a mailing, on a campaign, and as a winner criterion | Number of leads and opportunities, archived ones included, whose Campaign Source is the mailing's source | [customer-relationship-management](../customer-relationship-management/) |

The currency used to format the invoiced amount is the currency of the company of the mailing's responsible user, or of the campaign's company on a campaign. No currency conversion is performed: the amounts are read in the signed company currency of the invoices, which is the amount the accounting domain already normalised. When several companies are involved, the figure is an unconverted sum and must be read as an indicator, not as a ledger balance. **Industry-standard completion:** a replacement that needs a comparable figure across companies should convert each invoice amount into the campaign's currency at the invoice date rate, using the rule of [../multi-currency/calculations.md](../multi-currency/calculations.md), and state that it does so; the reference behavior does not convert.

Because these figures are used as comparison-test winner criteria, they are read with elevated rights during that selection, which means a marketing user can indirectly cause a sort on amounts they may not read directly. The sorted values are never displayed to them; only the resulting winner is.

## 3. The cost of sending

Sending text messages consumes credit of an external, prepaid service, and sending electronic mail may consume a paid relay. Neither consumption produces any record in this system:

- No purchase order, vendor bill, expense or journal entry is created when credit is consumed.
- The only visible consequence of exhausted credit is the failure type `sms_credit` on the delivery records and the derived flag on the mailing, which offers a button leading to the external purchase page of the service.
- The only visible consequence of an unregistered sending account is the failure type `sms_acc` and its own flag.

The purchase of that credit, when it is invoiced to the organisation, enters the system through the ordinary vendor-bill path of [accounts-payable](../accounts-payable/), with no link whatsoever to any mailing, campaign or delivery record.

## 4. Boundary summary

| Question | Answer |
|---|---|
| Does any marketing operation require a journal? | No. |
| Does any marketing operation require an account? | No. |
| Can a marketing record be locked by an accounting period? | No. |
| Does deleting a marketing record affect any ledger? | No. |
| Does the domain need the general ledger to be installed? | No. The campaign revenue indicator simply reports zero when the invoicing capability package is absent. |
