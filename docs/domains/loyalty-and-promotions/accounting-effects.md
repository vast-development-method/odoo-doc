# Accounting effects

The Loyalty and Promotions domain creates no journal entry of its own. It has no journal, no account field, no valuation and no reconciliation logic. Its accounting footprint is entirely indirect: it writes lines on sales documents, and those lines reach the general ledger through the ordinary customer invoicing of the [Sales Management](../sales/) domain and the ordinary session closing of the [Point of Sale](../point-of-sale/) domain. This file states exactly what those lines look like when they arrive in accounting, how the accounts are chosen, and what a replacement must and must not do.

## 1. The boundary

| Concern | Owned here | Owned elsewhere |
|---|---|---|
| The amount and the tax of a reward line | yes | |
| The product that carries the reward amount | yes (the hidden discount product of the reward) | |
| The income account used for that product | | [../general-ledger/](../general-ledger/) through the product and its category |
| The creation of the invoice from the order | | [../sales/](../sales/) |
| The posting of the invoice | | [../accounts-receivable/](../accounts-receivable/) |
| The journal entry of a closed counter session | | [../point-of-sale/](../point-of-sale/) |
| The liability represented by an unspent gift card balance | | not modelled; see section 6 |

## 2. What a reward line contributes to an invoice

A reward line is an ordinary document line. When the order is invoiced, it becomes an ordinary invoice line. Three shapes exist.

### 2.1 A discount reward

One invoice line per tax combination of the reward, each carrying:

- the reward's hidden discount product;
- quantity 1;
- a **negative** unit price;
- exactly the taxes of the document lines it compensates, mapped through the fiscal position;
- the description of the reward, possibly suffixed with the tax names.

Journal entry effect, for a discount of 76.00 split over a fifteen percent tax-excluded group of 40.00 and a ten percent tax-included group of 30.00, on a customer invoice in the company currency:

| Account | Debit | Credit |
|---|---|---|
| Product income account of the hidden discount product | 40.00 | |
| Tax account of the fifteen percent tax | 6.00 | |
| Product income account of the hidden discount product | 27.27 | |
| Tax account of the ten percent tax | 2.73 | |
| Customer receivable | | 76.00 |

The signs are the reverse of an ordinary sales line: the income account is debited and the receivable is credited, because the amounts are negative. The invoice as a whole remains balanced by the ordinary balancing rule of the general ledger.

### 2.2 A free product reward

One invoice line carrying the **real product**, its own taxes, the ordered quantity and a discount percentage of 100. The line's net amount is zero, so it contributes nothing to the ledger, but it does appear on the invoice so that the customer sees what they received and so that the stock and the cost of goods sold, when they are tracked, are driven by a real quantity of a real product.

Journal entry effect: none from the line itself. When the product is a stocked product and the deployment uses perpetual inventory valuation, the delivery of the free unit still produces the ordinary inventory output entry described in [../inventory-valuation-and-costing/](../inventory-valuation-and-costing/); the revenue side is zero, so the margin on that unit is the negative of its cost. This is the intended behavior of a give-away.

### 2.3 A payment reward (gift card, electronic wallet)

One invoice line carrying the reward's hidden discount product, quantity 1 and a negative unit price equal to the value spent.

- For an **electronic wallet** the line carries no tax at all, so the whole amount lands on the product income account.
- For a **gift card** the line carries the taxes of the gift card discount product, mapped through the fiscal position, and the price is recomputed so that the tax-included total is exactly the value spent. When the shipped configuration is used, that product carries no tax and the behavior is identical to the wallet.

### 2.4 A free shipping reward

One invoice line carrying the reward's hidden discount product, quantity 1, a negative unit price equal to the shipping amount cancelled, and the taxes of the shipping product mapped through the fiscal position.

## 3. Account selection

The account of a reward line is selected by the ordinary precedence of the general ledger, applied to the hidden discount product or, for a free product reward, to the real product:

1. the income account set on the product itself;
2. otherwise the income account set on the product's category;
3. otherwise the income account of the company's default sales settings;
4. in every case, the account is then mapped through the fiscal position of the customer.

The hidden discount product is created without an income account, so in a standard deployment the account comes from its product category. A deployment that wants discounts on a dedicated account sets that account on the product category of the hidden discount products, or on each hidden discount product individually. The hidden discount product is a service product, so no expense account or stock valuation account is involved.

**Industry-standard completion**: a replacement should place the hidden discount products in a dedicated product category mapped to a contra-revenue account (a "Sales discounts" account), so that gross revenue and discounts granted can be reported separately. The reference behavior does not force this and uses the ordinary income account; the choice is a reporting preference, not a behavioral requirement.

## 4. Taxes

1. A reward line never derives its taxes from its product. It carries exactly the tax combination it compensates, taken from the document lines and mapped through the fiscal position. This is the whole reason a single discount is split into several lines.
2. Fixed-amount taxes are never reduced by a reward that does not belong to a payment program, so the tax due on a flat levy stays intact whatever discount is granted.
3. A payment reward may reduce fixed-amount taxes, because it is a means of payment and must be able to bring the total to zero.
4. For a tax-excluded tax, the reward line carries only the base, and the tax engine recomputes the negative tax from it. For a tax-included tax, the reward line carries base plus tax, because that is what a tax-included price means. Both cases are described with numbers in section 12 of [calculations.md](calculations.md).
5. Rounding is per line. When a discount with a ceiling is split over several tax groups, the sum of the rounded lines may differ from the ceiling by up to one rounding step per line; this is accepted and is not corrected.

## 5. Analytic amounts

Reward lines carry the analytic distribution derived by the ordinary rules of [../analytic-accounting/](../analytic-accounting/): the distribution model matching the line's product, product category, customer and company. Because the amount is negative, the analytic line is negative, so a discount reduces the analytic revenue of the same account as the lines it compensates only when the distribution model matches the hidden discount product in the same way. A deployment that analyses revenue per project should therefore either set a distribution on the hidden discount products or accept that the discount lands on the default distribution.

**Industry-standard completion**: a replacement should copy the analytic distribution of the discounted lines onto the discount line in proportion to the amounts discounted, when the deployment requires discounts to reduce the same analytic accounts they were granted on. The reference behavior does not do this and applies the ordinary distribution rules to the hidden discount product.

## 6. Gift cards, electronic wallets and the revenue recognition question

This is the one place where a naive implementation produces wrong books, so the behavior is stated explicitly.

1. **Selling a gift card** produces an ordinary sales line for the gift card product at its face value. With the shipped configuration that product is a service with no tax, so selling a gift card of 50.00 credits the product income account of the gift card product by 50.00 and debits the receivable by 50.00. The card itself is created outside accounting and carries no journal entry.
2. **Spending a gift card** produces a negative sales line on the reward's hidden discount product. With the shipped configuration that line has no tax, so spending 50.00 debits the product income account of the hidden discount product by 50.00 and credits the receivable by 50.00.
3. The net effect over the two transactions is zero on the receivable and a transfer between the gift card product's income account and the hidden discount product's income account.
4. **Nothing in the reference behavior defers the revenue** of a gift card sale until the card is spent, and nothing recognizes breakage when a card expires unspent.

**Industry-standard completion**: under generally accepted accounting principles, the sale of a gift card is not revenue; it is a contract liability ("deferred revenue" or "gift card liability") that becomes revenue when the card is redeemed, and, for the portion statistically never redeemed, is recognized as breakage revenue over the redemption pattern. A replacement that must satisfy those principles should:

1. map the gift card product and the electronic wallet top-up product to a **liability account** rather than to an income account, so that selling a card credits the liability;
2. map the hidden discount product of the gift card and wallet rewards to the **same liability account**, so that spending the card debits the liability;
3. recognize revenue on the goods the card paid for, which the ordinary sales lines already do;
4. recognize breakage by a periodic manual entry that moves the estimated unredeemed portion from the liability account to a revenue account, and by a similar entry when a card reaches its expiration date with a positive balance.

Steps 1 and 2 are pure configuration and require no code. Steps 3 and 4 are outside the scope of this domain. The point that a replacement must get right is that the two products must be mapped to the **same** account, whichever it is, so that the liability or the revenue nets to zero over the life of a card. A deployment that maps the gift card product to a liability account but leaves the hidden discount product on an income account will permanently overstate both the liability and the revenue.

The same reasoning applies to an electronic wallet: a top-up is a prepayment, not revenue.

## 7. Loyalty points and the cost of the program

1. Points carried on a loyalty card are **not** recognized anywhere in the ledger. No provision, no accrual and no deferred revenue is booked when a customer earns points.
2. The cost of a loyalty program reaches the ledger only when a reward is claimed, as the negative revenue of the reward line or as the zero-revenue delivery of a free product.

**Industry-standard completion**: revenue recognition standards treat a loyalty point granted with a sale as a separate performance obligation: part of the transaction price of the original sale must be allocated to the points and deferred until they are redeemed or expire. A replacement that must satisfy those standards should compute, per sale, the stand-alone selling price of the points earned, book that portion to a contract liability instead of revenue, and release it when the points are spent or lapse. The reference behavior does none of this, and a replacement that only needs behavioral equivalence must not do it either, because it would change the amounts of the invoices produced.

## 8. Counter sessions

At a counter, reward lines are ordinary ticket lines with negative amounts. They reach the ledger through the journal entry produced when the session is closed, described in [../point-of-sale/accounting-effects.md](../point-of-sale/accounting-effects.md). This domain contributes only one classification: an invoice line whose product is one of the reward discount products of the ticket behind the invoice is flagged as a discount line for reporting.

## 9. Reversal behavior

1. Cancelling a sales order removes its reward lines and reverses the point movements, but does not touch any invoice. An invoice already posted must be reversed by the ordinary credit note workflow of [../accounts-receivable/](../accounts-receivable/).
2. A credit note produced from an invoice that carries reward lines reverses those lines like any other line, with the opposite sign. The points are **not** given back by the credit note: point movements follow the order lifecycle, not the invoice lifecycle. A replacement must not couple the two.
3. Deleting a history movement, which happens when a confirmed order is cancelled, has no accounting effect.

## 10. Reconciliation

Nothing in this domain reconciles. A gift card is not a payment method in the accounting sense: it is a negative sales line, not a payment. It therefore never appears in a bank reconciliation, never produces an open item, and never carries a payment reference. A replacement must resist the temptation to model gift cards as payments, because that would change the receivable amounts of every invoice.

## 11. Currency

1. Every reward amount is converted into the document currency before it is written, using the rate of the program's company at today's date.
2. Once written, a reward line behaves like any other line: the invoice carries it in the document currency and the ledger carries it in the company currency at the invoice's exchange rate.
3. No exchange difference is ever produced by this domain.

## 12. Worked example: the invoice of an order carrying three kinds of reward

An order in the company currency carries:

| Line | Content | Quantity | Unit price | Discount | Taxes | Tax excluded | Tax | Tax included |
|---|---|---|---|---|---|---|---|---|
| 1 | Product Alpha | 2 | 200.00 | 0 | 20 percent, tax excluded | 400.00 | 80.00 | 480.00 |
| 2 | Product Beta | 1 | 100.00 | 0 | none | 100.00 | 0.00 | 100.00 |
| 3 | Free Product Beta (free product reward) | 1 | 100.00 | 100 | none | 0.00 | 0.00 | 0.00 |
| 4 | Discount 10% on your order, tax group one | 1 | −40.00 | 0 | 20 percent, tax excluded | −40.00 | −8.00 | −48.00 |
| 5 | Discount 10% on your order, tax group two | 1 | −10.00 | 0 | none | −10.00 | 0.00 | −10.00 |
| 6 | Gift Card (payment reward) | 1 | −60.00 | 0 | none | −60.00 | 0.00 | −60.00 |
| | **Total** | | | | | **390.00** | **72.00** | **462.00** |

When the order is invoiced in full, the customer invoice carries the same six lines with the same amounts. Assume the income account of Product Alpha and Product Beta is `Product Sales`, the income account of the hidden discount product of the percentage reward is `Sales Discounts`, and the income account of the hidden discount product of the gift card reward is `Gift Card Liability` (the deployment chose the liability mapping of section 6).

| Account | Debit | Credit |
|---|---|---|
| Customer receivable | 462.00 | |
| `Product Sales` (line 1) | | 400.00 |
| `Product Sales` (line 2) | | 100.00 |
| `Product Sales` (line 3) | | 0.00 |
| Tax payable, twenty percent (line 1) | | 80.00 |
| `Sales Discounts` (line 4) | 40.00 | |
| Tax payable, twenty percent (line 4) | 8.00 | |
| `Sales Discounts` (line 5) | 10.00 | |
| `Gift Card Liability` (line 6) | 60.00 | |
| **Total** | **580.00** | **580.00** |

Observations a replacement must reproduce:

1. Line 3 produces a journal item of zero. It is still written so that the quantity invoiced of the free product is tracked and so that the printed invoice shows the gift.
2. Line 4 carries the same tax as line 1 and therefore reduces the tax payable by exactly the proportion discounted. Splitting the discount is what makes this exact; a single discount line carrying an average tax would misstate the tax payable.
3. Line 6 discharges the liability created when the gift card was sold. Had the gift card product and the hidden discount product been mapped to different accounts, the liability would never clear.
4. The receivable is the tax-included total of the order, 462.00, which is what the customer actually owes after the discounts and after the gift card has paid part of it.

## 12A. Itemisation of every journal item this domain causes

Every ledger item is stated here attribute by attribute. The domain produces four shapes of document line; the table below states, for each, every
attribute of the journal item that line becomes when the document is invoiced. Nothing else in this
domain reaches the ledger.

| Attribute | Discount reward line | Free product reward line | Payment reward line (gift card, electronic wallet) | Free shipping reward line |
|---|---|---|---|---|
| Journal | The sales journal of the invoice, chosen by the ordinary invoicing rules of [../sales/](../sales/); this domain never selects a journal. | Same. | Same. | Same. |
| Account selection rule | The income account of the reward's `discount_line_product_id`, else the income account of that product's category, else the company's default sales income account; then mapped through the customer's fiscal position. | The income account of the **real** product given away, resolved by the same precedence. | The income account of the reward's `discount_line_product_id`, resolved by the same precedence; a deployment that defers gift-card revenue points it at the same liability account as the gift-card product. | The income account of the reward's `discount_line_product_id`, resolved by the same precedence. |
| Debit or credit | Debit of the income account, because the line amount is negative; the receivable is credited by the same amount. | Neither: the net amount is zero. | Debit of the income or liability account; the receivable is credited. | Debit of the income account; the receivable is credited. |
| Amount formula | The absolute value of the line's tax-excluded subtotal, which is `discountable_per_tax` entry × `discount_factor`, rounded by the document currency. | Zero: `unit price × quantity × (1 − 100 ÷ 100)`. | The absolute value of the line's tax-excluded subtotal, which is the value spent minus the tax-included taxes adopted from the gift-card product. | The absolute value of the line's tax-excluded subtotal, at most the shipping amount and at most `discount_max_amount`. |
| Currency and rate | The document currency; the ledger carries the company-currency counter-value at the invoice's exchange rate, exactly as for any other line. The reward's own monetary parameters were already converted from the program currency at the rate of the day the line was written. | Not applicable; the amount is zero in every currency. | Same as a discount line. | Same as a discount line. |
| Date | The invoice date, as for every other line of that invoice. | Same. | Same. | Same. |
| Counterparty | The invoice's customer. | The invoice's customer. | The invoice's customer; **not** the card, which is never a counterparty. | The invoice's customer. |
| Analytic distribution | The distribution the ordinary rules of [../analytic-accounting/](../analytic-accounting/) derive for the hidden discount product; negative, because the amount is negative. | The distribution derived for the real product; the amount being zero, the analytic line is zero. | The distribution derived for the hidden discount product. | The distribution derived for the hidden discount product. |
| Tax treatment | Exactly the tax combination the line compensates, mapped through the fiscal position; one journal item per tax, of the opposite sign to the taxes it offsets. Fixed-amount taxes are never present, because they are excluded from the discountable amount. | The product's own taxes mapped through the fiscal position, applied to a base of zero, so every tax item is zero. | For an electronic wallet, no tax at all. For a gift card, the taxes of the hidden discount product mapped through the fiscal position, recomputed so that the tax-included total equals the value spent; fixed-amount taxes may be reduced. | The shipping product's taxes mapped through the fiscal position. |
| Reconciliation counterpart | None. The item is a revenue item and is never reconciled; only the receivable it contributes to is reconciled, against the customer's payment. | None. | None. A gift card is not a payment: it produces no open item, carries no payment reference and never appears in a bank reconciliation. | None. |

Two consequences follow from the table and must be reproduced:

1. **A discount never touches the account of the line it discounts.** It debits the account of its
   own hidden discount product. A deployment that wants discounts netted against the revenue they
   reduce must map the hidden discount products to the same accounts as the products sold, which is
   a configuration decision, not a behaviour of this domain.
2. **The tax items are exact, not proportional estimates.** Because the discount is split into one
   line per tax combination, each tax item is the exact negative of the proportion discounted from
   that combination, to the rounding of the document currency.

## 13. Reconciliation notes

1. **Which domain owns the ledger effects.** Both merged versions agree that this domain writes no
   journal entry of its own. One of them stated only the conclusion; the itemised journal entries of
   sections 2, 6 and 12 come from the other and are kept in full.
2. **Field and account naming.** Reward amounts are named by their reproduced field identifiers; the
   accounts are named by their role, because the account itself is a configuration decision of the
   deployment and is not shipped.
3. **Two completions are marked.** The deferral of gift-card revenue and the allocation of a portion
   of a sale to the loyalty points earned are both **industry-standard default** completions: the
   reference behaviour does neither, and a rebuild that only needs behavioural equivalence must not
   do them either, because they would change the amounts of the documents produced.
