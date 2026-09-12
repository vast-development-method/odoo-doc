# Accounting effects

## 1. The short answer

**The Delivery and Shipping domain posts no Journal Entry of its own.** It owns no journal, no
account, no account-determination rule and no posting routine. Not one operation of this domain —
rating a shipment, adding a shipping charge line, validating a transfer, creating a shipment,
storing a tracking reference, cancelling a shipment, printing a return label, choosing a collection
point, batching transfers — writes a Journal Item.

What the domain does is change the *inputs* of entries that other domains post. It does so in
exactly four places, and each of them is itemised below:

| # | What the domain contributes | Where the entry is posted |
|---|---|---|
| 1 | A taxable Sales Order Line for the carriage | The customer invoice, in [`../accounts-receivable/`](../accounts-receivable/) |
| 2 | A change to that line's amount after the shipment | The same customer invoice |
| 3 | A change to the order's fiscal position when a store is chosen for collection | Every tax line of the same customer invoice, through [`../taxes/`](../taxes/) |
| 4 | A payment path that leaves the invoice open until the goods are handed over | The receipt, in [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/) |

The domain also feeds the goods movements that produce stock entries, but it does not change them:
the carrier on a Transfer has no effect on the valuation entries that
[`../inventory-valuation-and-costing/`](../inventory-valuation-and-costing/) posts when the transfer
is validated. The declared value that this domain computes for a parcel is a customs figure sent to
the carrier; it is never posted.

---

## 2. Effect one: the shipping charge line on the customer invoice

### 2.1 What the domain creates

One Sales Order Line, described in full in [entities.md](entities.md) section 6.2 and created by
the procedure of [workflows.md](workflows.md) section 2 step 8. Its accounting-relevant members
are:

| Member | Value |
|---|---|
| Product | The Delivery Method's delivery product, which is a service product |
| Quantity | Exactly one, in the delivery product's reference unit |
| Unit price | The charge produced by the pipeline of [calculations.md](calculations.md) section 7 |
| Taxes | The delivery product's sale taxes restricted to the order's company, mapped through the order's fiscal position when the order carries a customer and a fiscal position |
| Currency | The order currency |
| Analytic distribution | Whatever the order line's ordinary analytic rules produce for that product and that order; the domain adds nothing |

### 2.2 What reaches the ledger

When the order is invoiced, the charge line becomes an invoice line like any other, and
[`../accounts-receivable/`](../accounts-receivable/) posts the Journal Entry. Itemised:

| Item | Journal | Account selection rule | Debit or credit | Amount | Currency and rate | Date | Counterparty | Analytic | Tax treatment | Reconciled against |
|---|---|---|---|---|---|---|---|---|---|---|
| Carriage revenue | The sales journal of the invoice | The income account of the delivery product, failing that the income account of the delivery product's category, failing that the company's default income account, then mapped through the invoice's fiscal position account mapping | Credit | The charge line's amount excluding taxes | The invoice currency, at the invoice's rate | The invoice date | The invoice's customer | The line's analytic distribution | The line is the base of the tax items below | Nothing; it is a revenue item |
| Tax on carriage | The same sales journal | The tax account of each tax on the charge line, taken from the tax's own distribution for an invoice | Credit for a sale tax | The tax computed on the charge line's amount excluding taxes | The invoice currency, at the invoice's rate | The invoice date | The invoice's customer | The tax's analytic setting | The tax item itself | Nothing; it is a tax item |
| Receivable | The same sales journal | The customer's receivable account | Debit | The tax-inclusive amount of the charge line, added to the tax-inclusive amount of every other line of the invoice into one receivable item | The invoice currency, at the invoice's rate | The invoice date | The invoice's customer | None | None | The customer's payment |

The receivable item is not a separate item for the carriage: an invoice carries one receivable item
per due date, and the carriage merely increases it.

### 2.3 Worked example

An order carries one line of goods at 750.00 excluding taxes with a tax of fifteen per cent, and a
shipping charge line at 9.95 excluding taxes with the same tax. The invoice currency is the company
currency, whose smallest unit is one hundredth.

```formula
goods excluding taxes        = 750.00
carriage excluding taxes     =   9.95
total excluding taxes        = 759.95
tax at fifteen per cent      = 759.95 × 0.15 = 113.9925 → 113.99
total including taxes        = 873.94
```

| Item | Account | Debit | Credit |
|---|---|---|---|
| Goods revenue | Income account of the goods product | | 750.00 |
| Carriage revenue | Income account of the delivery product | | 9.95 |
| Tax collected | Tax account of the fifteen per cent tax | | 113.99 |
| Receivable | Customer receivable account | 873.94 | |

The carriage is 9.95 of the 759.95 taxable base. Nothing distinguishes it in the entry except the
account the delivery product selects, which is why sellers who want carriage revenue separated give
the delivery product its own income account or its own product category.

### 2.4 The waived charge

When the free-above-a-threshold waiver applies, the charge line is created at zero. It still
reaches the invoice, still carries its taxes and still produces a revenue item — of zero. A rebuild
must keep the line rather than suppress it, because the customer's document must show that carriage
was provided and was free.

| Item | Account | Debit | Credit |
|---|---|---|---|
| Carriage revenue | Income account of the delivery product | | 0.00 |
| Tax collected on the carriage | Tax account | | 0.00 |

### 2.5 The tax-inclusive adaptation

When a fiscal position maps the delivery product's taxes, the *charge itself* is adapted before it
is written on the line, so that the amount the customer sees on the quotation is the amount the
fiscal position implies. The adaptation happens in the pipeline, not in the ledger. Its effect on
the entry is simply that the revenue item carries the adapted amount.

Using the worked example of [calculations.md](calculations.md) section 7.6 — a ten per cent
tax-included tax mapped to a fifteen per cent tax-excluded tax, and an engine result of 10.00:

| Item | Account | Debit | Credit |
|---|---|---|---|
| Carriage revenue | Income account of the delivery product | | 9.09 |
| Tax collected on the carriage | Tax account of the fifteen per cent tax | | 1.36 |
| Receivable contribution | Customer receivable account | 10.45 | |

---

## 3. Effect two: the real carriage charge written back after the shipment

### 3.1 What the domain changes

When the Delivery Method's invoicing policy is `real`, the charge line is created at zero and the
charge the carrier actually asked for replaces it when the shipment is created
([workflows.md](workflows.md) section 10). The line's product, taxes, quantity, currency and
analytic distribution are unchanged; only the unit price and the description change.

### 3.2 When the change reaches the ledger

| Situation | Consequence |
|---|---|
| The write-back happens **before** the invoice is created | The invoice simply carries the real amount. No correcting entry exists, because no entry existed. |
| The write-back happens **after** the invoice is created but the invoice is still a draft | The invoice line is not automatically updated; the seller must regenerate or edit the draft invoice. The domain writes on the order, never on the invoice. |
| The write-back happens **after** the invoice is posted | The charge line's invoiced quantity is no longer zero, so the line is no longer the *matching* line the write-back looks for — the write-back looks for a line priced at zero. A new charge line is therefore created for the shipment's charge, and it is invoiced by the next invoice of the order. |

The domain never posts a correcting entry and never touches a posted invoice. Every correction
travels through the order and through a further invoice, which is what makes the behaviour safe on
a locked order.

### 3.3 A partial shipment and its backorder

An order shipped in two parts under the `real` policy produces two charge lines and therefore two
revenue items, one per shipment. Worked example, with a carrier charging 40.00 per shipment and a
tax of fifteen per cent:

1. The order is priced. One charge line is created at 0.00 with the estimate in brackets.
2. The first transfer is validated. The carrier charges 40.00. The zero-priced line becomes 40.00
   and its description becomes the method's name.
3. The backorder is validated. The carrier charges 40.00 again. No zero-priced line matches, so a
   second charge line is created at 40.00.
4. The invoice carries two carriage lines.

| Item | Account | Debit | Credit |
|---|---|---|---|
| Carriage revenue, first shipment | Income account of the delivery product | | 40.00 |
| Carriage revenue, second shipment | Income account of the delivery product | | 40.00 |
| Tax collected on the carriage | Tax account of the fifteen per cent tax | | 12.00 |
| Receivable contribution | Customer receivable account | 92.00 | |

### 3.4 The margin is revenue, not a separate item

The proportional margin and the fixed margin are added to the charge before it is stored, so they
are indistinguishable in the ledger from the carrier's own price. A seller who wants to see the
margin separately must post the carrier's purchase invoice against a cost account and read the
difference; the domain provides no split.

---

## 4. Effect three: the fiscal position taken from the collection store

### 4.1 What the domain changes

When the order's Delivery Method is of the in-store kind and a store has been chosen, the order's
fiscal position is recomputed with the **store's address** as the delivery address, rather than the
customer's. Every tax on every line of the order is then recomputed. The recomputation is triggered
in two places: when the store is chosen, and when the in-store method is replaced by another
method, in which case the fiscal position reverts to the one the customer's own address implies.

### 4.2 Why it matters to the ledger

A fiscal position change alters:

- which tax each line carries, and therefore which tax account each tax item selects;
- the tax amounts, and therefore the receivable;
- in a cross-border configuration, whether the sale is taxed at all.

### 4.3 Worked example

The company is in one country. A customer of a second country buys one item priced 100.00 excluding
taxes. The company's ordinary tax is twenty per cent; a fiscal position for the customer's country
maps it to a zero per cent export tax.

| Scenario | Fiscal position | Tax on the line | Entry |
|---|---|---|---|
| Ordinary delivery to the customer's address | The export position | 0.00 | Revenue 100.00 credit, tax 0.00, receivable 100.00 debit |
| Collection from a store in the company's own country | The domestic position, taken from the store's address | 20.00 | Revenue 100.00 credit, tax 20.00 credit, receivable 120.00 debit |

Choosing the store therefore changes the entry by 20.00. This is the largest accounting consequence
the domain has, and it is entirely indirect: the domain sets the fiscal position and
[`../taxes/`](../taxes/) does the rest.

### 4.4 Multi-company

When the website belongs to a second company and the chosen store belongs to that company, the
fiscal position is looked up in that company's own fiscal positions. A fiscal position of the first
company with the same country is not chosen. The entry is therefore posted in the second company's
journals with the second company's accounts.

---

## 5. Effect four: the two deferred payment paths

### 5.1 Cash on delivery

The cash-on-delivery provider is a custom provider: it creates a payment transaction in the pending
state and posts nothing. What the domain adds is:

1. the filter that hides the provider unless the order's Delivery Method allows cash on delivery;
2. the post-processing that confirms the order while the transaction is still pending.

The ledger consequence is a timing one. The order is confirmed and the goods are shipped while no
money has been received, so:

| Moment | Ledger state |
|---|---|
| Order confirmed on a pending transaction | Nothing posted |
| Goods shipped | The stock entries of [`../inventory-valuation-and-costing/`](../inventory-valuation-and-costing/) only |
| Invoice posted | Receivable debited, revenue and tax credited, as in section 2 |
| Carrier remits the collected cash | A bank or cash receipt, reconciled against the receivable by [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/) |

The receivable therefore stays open from the invoice until the carrier remits. A seller who wants
the carrier's remittance tracked separately opens a dedicated account for cash in transit; the
domain neither provides nor requires one, which is recorded as an **industry-standard default**: a
rebuild that needs an intermediary account should post the carrier's remittance to an *undeposited
funds* account and clear it when the carrier's transfer arrives.

### 5.2 Pay on site

The pay-on-site provider behaves identically: a pending transaction, an order confirmed, no
posting. The money is taken at the counter when the customer collects.

| Moment | Ledger state |
|---|---|
| Order confirmed on a pending transaction | Nothing posted |
| Goods reserved and prepared in the store | The stock entries only |
| Customer collects and pays at the counter | The receipt is recorded by whatever channel takes the money — a bank journal, a cash journal or a point-of-sale session — and reconciled against the receivable |

The carriage itself is normally free for this path, because the shipped in-store Delivery Method
carries a delivery product priced at zero. The charge line is still created, still carries its
taxes and still produces a revenue item of zero, exactly as in section 2.4.

---

## 6. What the domain deliberately does not post

| Candidate | Why nothing is posted |
|---|---|
| The carrier's own invoice | It is a vendor bill, entered by [`../accounts-payable/`](../accounts-payable/). The domain never creates one and never matches one against the shipping cost stored on a Transfer. |
| The shipping cost stored on a Transfer | It is a record of what the carrier quoted, used to price the customer's charge line under the `real` policy and to inform the warehouse. It is not a cost accrual and never reaches a Journal Item. |
| The declared value of a parcel | It is a customs figure computed from product costs and sent to the carrier. It is never posted, and its computation deliberately uses the product's cost rather than its price. |
| The insurance percentage | It is a request made to the carrier. Any premium the carrier charges arrives on the carrier's own invoice. |
| The value of a moved quantity printed on the delivery slip | It is a document figure, computed from the sales order line's tax-inclusive total. It duplicates no ledger amount and creates none. |
| Cancelling a shipment | It clears a tracking reference. Any credit the carrier owes arrives on the carrier's own invoice or credit note. |
| A return label | It is a document. The goods coming back produce stock entries and, when the seller issues one, a credit note; neither is created by this domain. |

---

## 7. Analytic accounting

The domain contributes no analytic distribution of its own. The shipping charge line inherits
whatever distribution the ordinary analytic rules of [`../analytic-accounting/`](../analytic-accounting/)
produce for the delivery product, the customer and the order. A seller who wants carriage tracked
on an analytic account does so by giving the delivery product a product category or a distribution
model that carries one.

---

## 8. Multi-currency

Every conversion this domain performs is listed in [calculations.md](calculations.md) section 20
and all of them happen **before** the charge reaches an order line. By the time the amount reaches
the ledger it is expressed in the order currency, and the invoice converts it into the company
currency exactly as it converts every other line, at the invoice's own rate. The domain therefore
creates no exchange difference of its own.

One consequence deserves stating: because the charge is rounded once, by the order currency, a
charge computed in a company currency and converted into the order currency may not convert back to
the same figure. That is the ordinary behaviour of a converted price and is handled by
[`../multi-currency/`](../multi-currency/).

---

## 9. Where to look next

| Question | File |
|---|---|
| How the amount on the charge line is computed | [calculations.md](calculations.md) section 7 |
| When the charge line is created, replaced and removed | [state-machines.md](state-machines.md) section 2 |
| How the real charge is written back | [workflows.md](workflows.md) section 10 |
| Which taxes the charge line carries | [business-rules.md](business-rules.md) rule DSH-061 |
| The entries the invoice itself produces | [`../accounts-receivable/`](../accounts-receivable/) |
| The entries the goods movement produces | [`../inventory-valuation-and-costing/`](../inventory-valuation-and-costing/) |
| The entries the receipt produces | [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/) |
