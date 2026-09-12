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

## 4. The indirect chain: how a mailing eventually reaches the ledger

A mailing never posts anything, but it is frequently the first link of a chain that ends in a journal
entry. The chain is worth stating explicitly, because a rebuild has to know which link carries the
accounting consequence and which does not.

| Step | Record created or changed | Domain that owns the step | Ledger consequence at this step |
|---|---|---|---|
| 1 | A Mass Mailing is sent; one Mailing Trace per recipient; one Link Tracker per distinct link | this domain | none |
| 2 | A recipient clicks a tracked link; one Link Tracker Click carrying the campaign, the source and the delivery record | this domain | none |
| 3 | The visitor's browser receives the three campaign-tracking cookies | this domain | none |
| 4 | The visitor submits a form, which creates a Lead carrying the campaign, the source and the medium | [customer-relationship-management](../customer-relationship-management/) | none |
| 5 | The Lead becomes a quotation, which carries the same attribution on its lines | [sales](../sales/) | none; a quotation is not an accounting document |
| 6 | The quotation is confirmed and delivered | [sales](../sales/), [inventory-operations](../inventory-operations/) | the stock movement may post a valuation entry, which belongs to [inventory-valuation-and-costing](../inventory-valuation-and-costing/); nothing of it is attributable to the mailing |
| 7 | A customer invoice is created and posted | [accounts-receivable](../accounts-receivable/) | **the first and only journal entry of the chain**: the revenue and tax items of the invoice, exactly as that domain specifies them |
| 8 | The invoice is paid and reconciled | [payments-and-bank-reconciliation](../payments-and-bank-reconciliation/) | the payment entry and the reconciliation, again as that domain specifies them |
| 9 | The mailing reports a quotation count and an invoiced amount | this domain | none; both are read-only aggregates recomputed on demand |

Two consequences follow. First, no marketing record is ever a source document of a journal entry, so
deleting a mailing, a delivery record, a Link Tracker or a click never changes a balance. Second, the
attribution stored on the invoice is the Campaign Source of step 5, not a link to the mailing: a
rebuild that deletes the source of a finished mailing therefore loses the attribution of every
invoice behind it, which is why the source of a mailing cannot be deleted (rule `MKT-RULE-154` of
[business-rules.md](business-rules.md)).

## 5. What a bookkeeper may be asked about

| Question a bookkeeper may raise | Answer |
|---|---|
| "The campaign says we earned 42 000 but the revenue account shows something else." | The campaign figure is the untaxed signed total of the customer invoices whose Campaign Source is the mailing's source, excluding draft and cancelled invoices, summed without currency conversion. It is not an account balance and it excludes every invoice that carries no attribution. |
| "Can I reconcile against a mailing?" | No. No marketing record carries a reconciliation counterpart, an account, a journal or a tax. |
| "Does an accounting lock date block a marketing operation?" | No. No marketing operation writes into a period, so no period control applies to it. |
| "Where does the cost of the sending credit appear?" | Only as an ordinary vendor bill, entered through [accounts-payable](../accounts-payable/), with no link to any mailing, campaign or delivery record. |
| "Why does the revenue indicator show zero?" | Either no invoice carries the mailing's Campaign Source, or the invoicing capability package is not installed, in which case the indicator is simply absent from the computation and reports zero. |

## 6. Boundary summary

| Question | Answer |
|---|---|
| Does any marketing operation require a journal? | No. |
| Does any marketing operation require an account? | No. |
| Can a marketing record be locked by an accounting period? | No. |
| Does deleting a marketing record affect any ledger? | No. |
| Does the domain need the general ledger to be installed? | No. The campaign revenue indicator simply reports zero when the invoicing capability package is absent. |
