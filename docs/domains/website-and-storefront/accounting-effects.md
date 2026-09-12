# Accounting effects

This domain posts no journal entry of its own. Nothing in the site, the content management capability,
the blog, the forum, the public directories or the tracked-link service ever touches the ledger: they
create pages, templates, visitors, posts and clicks, none of which is a financial event.

The storefront half of the domain does not post either. It produces and prices the commercial documents
that other folders turn into accounting entries, and it carries three pieces of information those entries
depend on: the tax treatment of every priced line, the delivery charge line, and the site through which
the order originated. This file states precisely where the boundary lies, what this folder writes that has
an accounting consequence, and what a rebuild must therefore get right even though it never touches a
ledger.

---

## 1. What this folder does not do

| Not done here | Owned by |
|---|---|
| Creating, posting or reversing a journal entry | [general ledger](../general-ledger/README.md) |
| Creating a customer invoice from a confirmed order, including the invoicing policy, the down payment deduction and the invoice grouping | [sales](../sales/README.md) and [accounts receivable](../accounts-receivable/README.md) |
| Selecting the revenue account, the tax account or the receivable account | [accounts receivable](../accounts-receivable/README.md) and [taxes](../taxes/README.md) |
| Computing the tax amounts stored on a line and the tax summary of an order | [taxes](../taxes/calculations.md) |
| Registering a payment, reconciling it with an invoice, and handling the intermediary account of an online payment | [payments and bank reconciliation](../payments-and-bank-reconciliation/README.md) and [payment providers](../payment-providers/README.md) |
| Valuing the goods shipped for an online order | [inventory valuation and costing](../inventory-valuation-and-costing/README.md) |
| Distributing the revenue analytically | [analytic accounting](../analytic-accounting/README.md) |
| Revaluing an unpaid online invoice held in another currency | [multi-currency](../multi-currency/README.md) |

---

## 2. What this folder writes that has an accounting consequence

### 2.1 The fiscal position of the cart

The storefront resolves a fiscal position for every visitor, including an anonymous one, from the country
detected by network location, and writes it on the cart at creation. It rewrites it whenever the customer
or the delivery address changes and, with collection in store, whenever a pickup store is chosen, using the
store address as the delivery address.

The fiscal position is what maps the product's own taxes onto the taxes actually charged and, through the
account mapping of the fiscal position, what will later determine the revenue and tax accounts of the
invoice. A rebuild that resolves a different fiscal position produces a different journal entry, even
though the storefront itself posts nothing.

Every fiscal position change on a draft storefront order is immediately followed by a recomputation of the
order taxes and, when the price list also changed, by a recomputation of the prices. Changing the country,
the tax identification number or the postal code of a contact triggers the same recomputation on every
draft storefront order of that contact and on any order delivering to it.

### 2.2 The tax display mode

The site setting that chooses between tax-excluded and tax-included display changes only what the shopper
reads, never what is stored: a line always stores a tax-excluded unit price plus its taxes. A rebuild must
not store tax-included amounts when the site is configured for tax-included display, because the invoice,
the tax report and the ledger all read the stored, tax-excluded values.

The one place where the display mode reaches a stored amount is the delivery charge: the rate displayed and
written on the delivery line is the tax-excluded rate when the site displays tax-excluded prices and the
tax-included rate when it displays tax-included prices. See §2.3.

### 2.3 The delivery charge line

A delivery method produces one ordinary Sales Order Line carrying the method's delivery product, the
quantity one and the computed rate as the unit price. That line is invoiced like any other line, therefore
it produces a revenue entry on the account of the delivery product, or of its product category, with the
taxes of that product mapped by the order's fiscal position.

Two storefront-specific facts matter.

1. **The rate is stored after tax adjustment.** The rate returned by the method is first restated as a
   tax-included unit price for the order's company, currency, date and fiscal position; then any configured
   margin is applied; then the result is rounded to the order currency precision. The value written on the
   line is that result. The arithmetic is [calculations.md](calculations.md) §17.
2. **A free-shipping threshold writes zero, it does not remove the line.** When the order total excluding
   delivery reaches the method's threshold, the unit price becomes 0.00 and the line stays. The invoice
   therefore shows a delivery line at zero, which is what makes the free shipping visible to the customer
   and auditable afterwards. The pre-override rate is kept on the rate payload as the carrier price, for
   the benefit of the shipping folder.

A cart that becomes services-only loses its delivery line entirely, and with it any delivery revenue.

### 2.4 The site stamp on the invoice

Every Journal Entry carries a site field, computed from the source orders of its lines: the site is set
when every sales order line feeding the invoice comes from one site, and left empty otherwise. It is
stored, read only and tracked in the discussion thread.

It has no effect on debits, credits or account selection. It exists for reporting: it lets the online
revenue of one site be isolated in a filter or a grouping without joining back to the orders. Transfers
carry the same stamp, related to their source order.

When the storefront capability is installed on an existing database, the column is created directly and
left empty for historical invoices rather than recomputed, so that installation on a large history stays
bounded. A rebuild may choose to fill it retrospectively; the value for a historical invoice is
deterministic and can be recomputed at any time from its lines.

### 2.5 The order confirmation trigger

The storefront decides *when* an order is confirmed, and confirmation is what makes the order invoiceable.

| Payment situation | Order state reached | Accounting consequence |
|---|---|---|
| The transaction reaches the completed or the authorised state and the confirmation amount is reached | confirmed | Delivery work is created; the order becomes invoiceable; with automatic invoicing the invoice is created, posted and sent immediately. |
| The transaction stays pending, as for a wire transfer | sent | No invoice, no stock reservation. A person confirms later, which then produces the ordinary consequences. |
| The transaction stays pending with a pay-on-site provider | confirmed | The order is confirmed and the transfer is created, but no payment has been captured: the receivable remains open until the store registers the payment. |
| The order total is zero and no transaction exists | confirmed | The order is confirmed; the resulting invoice, if any, is a zero invoice. |
| The transaction is in error | unchanged, draft | Nothing; the cart is only excluded from abandoned-cart messages. |

The automatic invoicing behaviour is governed by the platform parameter `sale.automatic_invoice`, which is
owned by the [sales](../sales/README.md) folder. The storefront only exposes the setting and relies on the
transaction post-processing to run it. With automatic invoicing enabled, the invoice is created even for a
**partial** payment, because the invoicing is attached to the transaction's orders rather than to the
confirmed ones.

### 2.6 Donations

A donation transaction is not linked to a sales order. It produces, through the payment folder, a payment
whose donation flag is copied from the transaction. The flag is informational: it lets an accountant
separate donation receipts from commercial receipts in a filter or a grouping. The accounting of the
payment itself, including the account of the payment method and the reconciliation, is unchanged.

The donation transaction also logs the donor details on the payment as a message: the company, the contact,
the donor name, the donor country and the donor address, each prefixed by its field label. This is a
message in the discussion thread, not a journal item.

### 2.7 Zero-priced products

When a site forbids zero-price sales, a line whose priced group totals zero is refused at add time and, if
it slipped through, blocks the checkout and the payment. This prevents an order, and therefore an invoice,
with a zero revenue line for a product that was meant to be priced. Exempt from the rule: delivery lines,
combo parent and item lines, which are priced as a group, promotion reward lines, and products whose
service tracking value is in the exempt list, which currently contains only the course tracking value.

### 2.8 What the site half writes that reaches a ledger indirectly

| Written here | Where it becomes an accounting entry |
|---|---|
| A lead created by a public contact form | Nowhere by itself; it becomes an order, then an invoice, through [customer relationship management](../customer-relationship-management/README.md) and [sales](../sales/README.md). |
| An event registration, a course enrolment or a job application created from a public page | The owning folder decides whether an order or an invoice follows. |
| A newsletter subscription created at checkout | Never; it is a mailing contact. |
| A page, a blog post, a forum post, a visitor, a track, a tracked link click | Never. |

---

## 3. What a rebuild must reproduce for the ledger to match

1. The same fiscal position for the same visitor, including the geolocated fiscal position of an anonymous
   visitor and the store-based fiscal position of an order collected in store.
2. The same price list for the same visitor, therefore the same tax-excluded unit prices and discounts on
   the lines.
3. The same delivery charge line, with the same unit price after tax adjustment, margin and free-shipping
   override, and the same delivery product.
4. The same confirmation moment for each payment situation, because that moment decides the invoice date
   when automatic invoicing is enabled.
5. The same site stamp on the invoices and the transfers, because online revenue reporting depends on it.
6. The refusal of zero-priced lines under the corresponding setting.

---

## 4. Reporting that reads accounting data

The online sales measure of the periodic digest sums the line subtotal of the sales analysis rows of the
period whose state is not draft, cancelled or sent and whose site is set, per company. It is a sales
measure, not a ledger measure: it reads order lines, not journal items, and it is therefore expressed tax
excluded, in the company currency of the sales analysis report. Reading it without the all-documents sales
access group raises an access error whose message is
`Do not have access, skip this data for user's digest email`, which makes the digest omit the measure for
that recipient rather than fail. The formula is in [calculations.md](calculations.md) §28.1.

---

## Reconciliation notes

1. **Two statements of the same boundary.** The site half of the folder had no accounting document at all
   and the storefront half had one. The merged statement keeps the storefront's itemised boundary and adds
   §2.8, which lists what the site half writes that another folder may later turn into an entry.
2. **Delivery method ownership.** One version placed the delivery charge line under inventory operations.
   Delivery methods and their rating belong to [delivery and shipping](../delivery-and-shipping/README.md);
   the warehouse and the quantities belong to
   [inventory operations](../inventory-operations/README.md). §2.3 is written accordingly.
