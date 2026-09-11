# Point of Sale — Accounting Effects

This file specifies every accounting document the point of sale domain produces: which
journal is used, which lines are written, how each account is selected, which side each
amount lands on, the exact amount formula, the currency handling, the date, the partner,
the tax handling and the reconciliation performed.

The centrepiece is the **session closing entry**, specified line by line in section 3.

Throughout this file:

- **Selling currency** means the currency of the point of sale configuration; every amount
  carried by an order, a line or a payment is expressed in it.
- **Company currency** means the currency of the company that owns the configuration.
- A journal item is written either with an explicit debit and credit pair, or with a
  signed balance and a signed amount in currency. The two notations are equivalent: a
  positive balance is a debit and a negative balance is a credit. Which notation a given
  line uses is stated, because it determines whether a foreign-currency amount is carried.

---

## 1. Overview of the documents produced

| Document | Journal | When | Section |
| --- | --- | --- | --- |
| Session closing entry | The configuration's point of sale journal | Session validation | 3 |
| Cash statement line, per aggregated cash method | The cash method's journal | Session validation | 3.9 |
| Cash statement line, per identified-customer cash payment | The cash method's journal | Session validation | 3.9 |
| Cash statement line, cash difference at closing | The session's cash journal | Session validation, after the closing entry balances | 4 |
| Cash statement line, manual cash in or cash out | The session's cash journal | Any time during the session | 5 |
| Accounting payment, per aggregated bank method | The bank method's journal | Session validation | 3.8 |
| Accounting payment, per identified-customer bank payment | The bank method's journal | Session validation | 3.8 |
| Closing-difference entry for an identified-customer bank method | That method's journal | Session validation | 3.8.4 |
| Customer invoice or credit note | The configuration's invoice journal | When an order is invoiced | 6 |
| Invoice payment entry, one per tender | The configuration's point of sale journal | When an order is invoiced | 7 |
| Reversal entry removing an order from the closing entry | The configuration's point of sale journal | When an order is invoiced after its session closed | 8 |
| Real-time cost of goods sold entry | Determined by the inventory valuation domain | When a delivery is completed and the product is valued in real time | 9 |

---

## 2. Accounts used and how they are selected

| Role | Selection rule |
| --- | --- |
| **Income account of a sale** | The income account configured on the product (falling back, through the product category, to the category's income account). When the product resolves to no income account at all: *"Please define income account for this product: '<product name>' (id:<product identifier>)."* When the product-derived account is empty, the default account of the configuration's point of sale journal is used instead. The result is then mapped through the order's fiscal position. |
| **Tax account** | The account of the tax repartition line that produced the tax amount, as returned by the tax engine. When a repartition line has no account the whole closing is refused (see section 3.7). |
| **Point of sale receivable account** | The intermediary account of the payment method when it has one, otherwise the company's default point of sale receivable account. |
| **Customer receivable account** | The receivable account of the accounting partner (the commercial entity of the selected customer), read in the context of the order's company. |
| **Outstanding account of a bank method** | The outstanding account configured on the payment method. When the payment state model distinguishes in-payment from paid and the method has none, the account is derived from the payment direction by the payments domain. |
| **Cash account** | The default account of the cash method's journal. |
| **Cash profit and cash loss accounts** | The profit account and the loss account of the cash journal. |
| **Rounding profit and rounding loss accounts** | The profit account and the loss account of the configuration's cash rounding definition. |
| **Expense account (cost of goods sold)** | The expense account resolved for the product by the inventory valuation domain, then mapped through the order's fiscal position when the move belongs to a transfer of an order that has one. |
| **Stock valuation account** | The stock valuation account resolved for the product by the inventory valuation domain. Never mapped by a fiscal position. |
| **Balancing account** | Chosen by the operator in the forced-close wizard; defaulted to the company's default point of sale receivable account, falling back to the fallback customer receivable account. |

---

## 3. The session closing entry

### 3.1 Header

| Attribute | Value |
| --- | --- |
| Journal | The point of sale journal of the session's configuration. |
| Date | Today in the acting time zone — **not** the session's opening or closing instant. |
| Reference | The session identifier (the session's name). |
| Entry kind | An ordinary entry (not an invoice). |
| Link | Stored on the session as its journal entry, and reachable from every order of the session as its session journal entry. |
| Tax exigibility | Forced to always exigible: the closing entry never produces cash-basis tax entries of its own, because the tax values are written directly on it. |

The entry is created empty, then populated in seven passes, in this exact order:

1. Accumulate amounts (section 3.2).
2. Create the non-reconcilable lines: taxes, sales, cost of goods sold, rounding
   (sections 3.5 to 3.7 and 3.12).
3. Create the bank receivable lines and their accounting payments (section 3.8).
4. Create the pay-later receivable lines (section 3.10).
5. Create the cash statement lines and the cash receivable lines (section 3.9).
6. Create the invoice receivable lines (section 3.11).
7. Create the stock valuation lines (section 3.12).
8. Optionally create the balancing line (section 3.13).

The order matters: the sales lines are created after the tax lines, and the identifier of
each created sales line is kept so that other capabilities (loyalty, events) can attach to
it.

### 3.2 Which orders are aggregated

Let the **closed orders** of the session be the orders whose state is neither unfinished
nor cancelled — that is, the paid, posted and invoiced ones.

| Contribution | Orders considered |
| --- | --- |
| Payments (all receivable aggregations) | Every closed order, **invoiced ones included** |
| Sales lines, tax lines, rounding difference | Every closed order that is **not** invoiced |
| Invoice receivable lines | Every closed order that **is** invoiced |
| Cost of goods sold and stock valuation lines | Every order of the session — whatever its state — that is not invoiced and has no shipping date, through its transfers; plus every transfer attached to the session that belongs to no single order (the deferred ones created at closing) |

An **invoiced order** is an order whose invoice link is filled. Its revenue and its taxes
are recognised by its own invoice, so they must not be recognised a second time by the
closing entry. Its tenders, however, really were received at the counter, so they *are*
aggregated, and a counterbalancing credit is written for them (section 3.11). The net
effect on the closing entry of an invoiced order is therefore exactly zero, while the
money actually collected still reaches the bank or the cash drawer.

### 3.3 The amount accumulator

Every aggregation bucket holds a pair of running totals:

- **amount** — in the selling currency;
- **amount converted** — in the company currency.

Tax buckets hold two further totals, the base amount and the converted base amount.

Adding a contribution to a bucket:

```formula
new_amount = old_amount + contribution_amount
```

```formula
new_amount_converted = old_amount_converted + converted_contribution
```

where

```formula
converted_contribution = contribution_amount                                  when the session currency is the company currency,
                                                                              or the contribution is declared to be already in company currency

converted_contribution = convert( contribution_amount ,
                                  from = selling currency ,
                                  to = company currency ,
                                  company = the session's company ,
                                  at = the contribution date )                otherwise
```

The conversion rounds to the company currency by default. The contribution date differs
per bucket and is stated in each section below. Base amounts are added without any
conversion at all: the converted base amount receives the same figure as the base amount.

Cost of goods sold and stock valuation contributions are always declared to be already in
company currency, because inventory valuation is kept in company currency.

### 3.4 Debit and credit conventions

Two helpers are used throughout.

**Debiting** a line with an amount and a converted amount:

```formula
debit  = converted_amount   when converted_amount > 0 , otherwise 0
credit = − converted_amount when converted_amount < 0 , otherwise 0
```

and, when the selling currency differs from the company currency and the amount is not
declared to be in company currency, the line also carries an amount in currency equal to
the amount, with the selling currency as its currency.

**Crediting** a line with an amount and a converted amount:

```formula
debit  = − converted_amount when converted_amount < 0 , otherwise 0
credit =   converted_amount when converted_amount > 0 , otherwise 0
```

and, in the same condition, an amount in currency equal to **minus** the amount.

So "debit with 25.00" means a debit of 25.00 when the figure is positive and a credit of
25.00 when it is negative. Sign is carried by the amount, never by the caller.

### 3.5 Sales lines

#### 3.5.1 Building the base lines

For every non-invoiced closed order, in the order's own accounting context:

1. Convert each order line into a tax base line. The base line carries: the line record,
   the commercial partner of the order's customer, the selling currency, the order's
   currency rate, the product, the taxes after fiscal position mapping, the unit price,
   the quantity `line quantity × (−1 when the order is a refund order, +1 otherwise)`, the
   discount percentage, the income account after fiscal position mapping, whether the line
   itself is a refund line, the unit of measure and the rendered product name.
2. A line is a **refund line** when `unit price × quantity < 0`. An order is a **refund
   order** when its refund flag is set or its total is negative.
3. The base line's sign is `+1` when the order is a refund order and `−1` otherwise. This
   sign is what turns an ordinary sale into a credit.
4. Add the tax details to the base lines, round the tax details per base line, and add the
   accounting data including cash-basis tags.
5. Ask the tax engine for the tax lines to add and the base line updates.

The rendered product name is built as follows: the product's display name in the
customer's language; if the line carries a full product name, that name prefixed by the
product's internal reference in square brackets and a space when the product has one; then,
when the product has a sales description, a newline and that description.

#### 3.5.2 The aggregation key

Each updated base line is placed in a bucket keyed by:

| Key part | Value |
| --- | --- |
| Account | The income account of the base line, after fiscal position mapping. |
| Sign | `−1` when the base line is a refund line, `+1` otherwise. |
| Taxes | The identifiers of the line's taxes after fiscal position mapping, with tax hierarchies flattened, as an ordered tuple. |
| Base tags | The identifiers of the base line's tax tags, as an ordered tuple. |
| Product | The product identifier when the configuration asks for a per-product closing entry, otherwise nothing. |

Two lines with the same key are added together. The contribution is the base line update's
amount in currency (as the amount) and its balance (as the converted amount). Because the
converted amount is supplied directly by the tax engine, no further conversion is applied
to sales lines — the engine has already used the order's own currency rate.

The contribution date is the order date, but it is only used when the engine did not
supply a balance.

When the configuration asks for a per-product closing entry, the bucket also accumulates
the quantity: the base line quantity is added to a running quantity that starts at zero.

#### 3.5.3 The resulting journal item

| Attribute | Value |
| --- | --- |
| Name | `Sales untaxed` when the sign is `+1` and the line carries no tax; `Refund untaxed` when the sign is `−1` and the line carries no tax; otherwise `Sales <product name> with <tax names>` or `Refund <product name> with <tax names>`, where the product name is empty unless the configuration asks for a per-product closing entry, and the tax names are the names of the applied taxes joined by a comma and a space. |
| Account | The account of the key. |
| Taxes | The taxes of the key, set on the line so that the line is recognised as a taxed base. |
| Tax tags | The base tags of the key. |
| Product | The product of the key, or empty. |
| Product unit | The product's reference unit when a product is set, otherwise empty. |
| Display kind | Product line. |
| Currency | Always the selling currency. |
| Amount in currency | The accumulated amount. |
| Balance | The accumulated converted amount. |
| Quantity | The accumulated quantity (defaulting to 1.00 when quantities are not tracked) multiplied by the key's sign. |

Note that this line is written with an explicit balance and amount in currency rather than
with the debit-and-credit helper, and that it always carries a currency — even when the
selling currency is the company currency.

For an ordinary sale the balance is **negative**, so the line is a credit: revenue.

#### 3.5.4 Worked example

Two uninvoiced orders in a session whose selling currency is the company currency:

- Order A, one line, product P, quantity 1, unit price 25.00 including a 21 percent
  price-included tax, income account 400000, no fiscal position.
- Order B, one line, product Q, quantity 1, unit price 40.50 including the same tax, same
  income account.

The tax engine returns, for order A, a base amount of 20.66 and a tax amount of 4.34; for
order B, a base amount of 33.47 and a tax amount of 7.03. Both base lines have the same
account, the same sign (+1), the same tax set and the same base tag set, so they fall into
one bucket:

```formula
sales_amount = − ( 20.66 + 33.47 ) = − 54.13
```

One journal item results: account 400000, name `Sales with 21%`, amount in currency
−54.13, balance −54.13, that is a **credit of 54.13**, carrying the 21 percent tax and the
base tags of that tax, quantity 1.00.

### 3.6 Refund lines

A refund line is not a separate line kind; it is an ordinary base line whose quantity is
negative. Two consequences:

1. Its key sign is `−1`, so it lands in a different bucket than the sales of the same
   product and tax set. The bucket produces a line whose name begins with the word
   `Refund`.
2. Its amounts are the negatives of the corresponding sale amounts, so the resulting
   journal item is a **debit** on the income account.

A refund of the whole of order A above produces a bucket with amount +25.00 split as a
base of +20.66 and a tax of +4.34: a debit of 20.66 on account 400000 named
`Refund with 21%`, and a debit of 4.34 on the tax account.

### 3.7 Tax lines

#### 3.7.1 The aggregation key

Each tax line returned by the engine is placed in a bucket keyed by the triple:

| Key part | Value |
| --- | --- |
| Account | The tax line's account. |
| Tax repartition line | The identifier of the repartition line that produced the amount. |
| Tax tags | The identifiers of the tax tags of the tax line, as an ordered tuple. |

The contribution adds the tax line's amount in currency as the amount, its balance as the
converted amount, and its tax base amount as the base amount. The contribution date is the
order date.

#### 3.7.2 The resulting journal item

| Attribute | Value |
| --- | --- |
| Name | The name of the tax that owns the repartition line. |
| Account | The account of the key. |
| Tax base amount | The accumulated converted base amount. |
| Tax repartition line | The repartition line of the key. |
| Tax tags | The tags of the key. |
| Display kind | Tax line. |
| Currency | Always the selling currency. |
| Amount in currency | The accumulated amount. |
| Balance | The accumulated converted amount. |

#### 3.7.3 Missing tax account

Before anything is written, every prepared tax line is checked for an account. If any has
none, the whole closing is refused with:

> Unable to close and validate the session.
> Please set corresponding tax account in each repartition line of the following taxes:
> *<the names of the offending taxes, joined by a comma and a space>*

#### 3.7.4 Worked example, continued

The two orders above yield one tax bucket: account 251000 (the tax payable account), the
sale repartition line of the 21 percent tax, its tags.

```formula
tax_amount = − ( 4.34 + 7.03 ) = − 11.37
```

```formula
tax_base_amount = 20.66 + 33.47 = 54.13
```

One journal item results: account 251000, name `21%`, amount in currency −11.37, balance
−11.37, that is a **credit of 11.37**, with a tax base amount of 54.13.

### 3.8 Bank tenders

#### 3.8.1 Aggregated bank tenders

For every payment of every closed order whose method is of the bank kind and whose
identify-customer flag is **false**, the amount is accumulated in a bucket keyed by the
payment method. The contribution date is the payment date. Payments whose amount is zero
at the selling currency's precision are skipped entirely.

Two things are then created per bucket.

**(a) A receivable line in the closing entry**

| Attribute | Value |
| --- | --- |
| Account | The intermediary account of the payment method, or the company's default point of sale receivable account. |
| Name | The session identifier, a space, a hyphen, a space, the payment method name. |
| Display kind | Payment term line — so that the line is recognised as a settlement line and excluded from follow-up. |
| Partner | None. |
| Amounts | **Debited** with the accumulated amount and the accumulated converted amount. |

**(b) An accounting payment**

| Attribute | Value |
| --- | --- |
| Amount | The absolute value of the accumulated amount. |
| Direction | Inbound when the accumulated amount is greater than or equal to zero at the selling currency's precision, outbound when it is less. |
| Journal | The journal of the payment method. |
| Forced outstanding account | The outstanding account of the payment method. |
| Destination account | The intermediary account of the payment method, or the company's default point of sale receivable account. |
| Memo | `Combine <payment method name> POS payments from <session identifier>` |
| Payment method link | The counter payment method. |
| Session link | The session. |
| Company | The session's company. |

The accounting payment is posted immediately. Its own entry, written in the payment
method's journal by the payments domain, contains a **debit on the outstanding account**
and a **credit on the destination account** for an inbound payment, and the mirror for an
outbound one.

The receivable line of the closing entry and the destination line of the payment entry are
then reconciled with each other (section 3.14).

When the payment state model distinguishes in-payment from paid and the created payment
has no outstanding account, the outstanding account is derived from the direction before
posting.

#### 3.8.2 Identified-customer bank tenders

For every payment whose method is of the bank kind and whose identify-customer flag is
**true**, the amount is accumulated in a bucket keyed by the payment itself — one bucket
per tender, never merged.

**(a) A receivable line in the closing entry**

| Attribute | Value |
| --- | --- |
| Account | The receivable account of the **accounting partner** of the payment's customer. |
| Partner | That accounting partner. |
| Name | The session identifier, a space, a hyphen, a space, the payment method name. |
| Amounts | **Debited** with the payment amount and its converted value. |

If the payment has no customer at all the closing is refused:

> You have enabled the "Identify Customer" option for *<payment method name>* payment
> method,but the order *<order name>* does not contain a customer.

(The message is reproduced exactly as the system emits it, including the missing space
after the comma.)

**(b) An accounting payment, one per tender**

| Attribute | Value |
| --- | --- |
| Amount | The absolute value of the payment amount. |
| Partner | The accounting partner. |
| Direction | Inbound when the amount is greater than or equal to zero, outbound otherwise. |
| Journal | The journal of the payment method. |
| Forced outstanding account | The outstanding account of the payment method. |
| Destination account | The receivable account of the accounting partner. |
| Memo | `<payment method name> POS payment of <customer display name> in <session identifier>` |
| Payment method link, session link | As above. |

Payments whose method has no journal at all are skipped.

#### 3.8.3 Per-method closing differences

The closing procedure may be given a map from payment method to a difference amount — the
amount the operator says was actually received compared with what the till recorded. For
an **aggregated** bank method the difference is folded into the accounting payment:

1. Compute the source and destination line values (section 3.8.4). If there is nothing to
   post, stop.
2. Find the line of the payment entry that sits on the source account (the outstanding
   account).
3. Compute the new balance of that line:

```formula
new_balance = current_outstanding_balance + convert( difference , from = selling currency , to = company currency , at = the session closing instant )
```

4. Put the payment entry back into the draft state, add the destination line, and rewrite
   the outstanding line so that it carries a debit of the new balance when that balance is
   positive and a credit of its absolute value when it is negative.
5. Set the payment amount to the absolute value of the new balance and post the entry
   again.

#### 3.8.4 Closing differences for identified-customer bank methods

For every bank method of the till whose identify-customer flag is set, a **separate
miscellaneous entry** is created, whatever the number of payments:

| Attribute | Value |
| --- | --- |
| Journal | The journal of the payment method. |
| Date | Today in the acting time zone. |
| Reference | `Closing difference in <payment method name> (<session identifier>)` |
| Lines | The source line and the destination line below. |

The two lines are computed as follows. Let `difference` be the amount given for that
method, and compare it with zero at the selling currency's precision.

- The **source account** is the outstanding account of the payment method (or, when the
  difference is being folded into an accounting payment, that payment's outstanding
  account).
- The **destination account** is the profit account of the method's journal when the
  difference is positive, and its loss account when the difference is negative.
- When the difference is zero, or there is no source account, nothing is created at all.

```formula
converted_difference = convert( difference , from = selling currency , to = company currency , at = the session closing instant )
```

- The source line **debits** the source account with the difference and the converted
  difference.
- The destination line **credits** the destination account with the same pair.

A positive difference therefore debits the outstanding account (more money arrived than
recorded) and credits the profit account. A negative difference does the mirror.

These entries are posted immediately and are found again later by searching journal items
whose reference matches the closing-difference reference string.

### 3.9 Cash tenders

Cash tenders produce **two** documents each: a bank statement line in the cash journal
(which carries its own entry: the cash account against the receivable account) and a
counterpart receivable line in the closing entry.

#### 3.9.1 Aggregated cash tenders

Accumulated per payment method, contribution date the payment date. A bucket whose amount
is zero at the selling currency's precision is skipped entirely.

**Statement line**

| Attribute | Value |
| --- | --- |
| Date | Today in the acting time zone. |
| Label | The session identifier. |
| Session link | The session. |
| Journal | The journal of the payment method. |
| Counterpart account | The intermediary account of the payment method, or the company's default point of sale receivable account. |
| Amounts | See the currency rule below. |

**Receivable line in the closing entry** — identical in shape to the aggregated bank
receivable line of section 3.8.1(a): same account, same name, payment-term display kind,
no partner, **debited** with the accumulated amount and converted amount.

#### 3.9.2 Identified-customer cash tenders

Accumulated per payment.

**Statement line**

| Attribute | Value |
| --- | --- |
| Date | The payment date expressed as a date in the acting time zone. |
| Label | The payment's own label. |
| Session link | The session. |
| Journal | The journal of the payment method. |
| Counterpart account | The receivable account of the accounting partner. |
| Partner | The accounting partner. |

**Receivable line in the closing entry** — identical in shape to section 3.8.2(a).

#### 3.9.3 Currency of a statement line

```formula
journal_currency = the journal's own currency, or the company currency when it has none
```

- When the journal currency equals the selling currency, the statement line carries only
  an amount, equal to the bucket amount.
- Otherwise the statement line carries an amount equal to
  `convert( bucket amount , from = selling currency , to = journal currency , company = the session's company , at = the session closing instant )`,
  an amount in currency equal to the bucket amount, and the selling currency as its
  foreign currency.

#### 3.9.4 The entry behind a statement line

The bank statement line model writes, in the journal of the statement line:

- a **liquidity line** on the journal's default account for the statement amount, and
- a **counterpart line** on the counterpart account for the opposite amount.

For an aggregated cash bucket of 25.00 that means: **debit the cash account 25.00**,
**credit the point of sale receivable account 25.00**.

### 3.10 Pay-later tenders

A tender whose method is of the customer-account kind is aggregated **only when the order
is not invoiced**, because an invoiced order settles the customer account through the
invoice itself and no payment entry is created for it.

- Not identifying the customer: accumulated per payment method, contribution date the
  payment date; produces one aggregated receivable line exactly as in section 3.8.1(a).
- Identifying the customer: accumulated per payment; produces one receivable line per
  tender exactly as in section 3.8.2(a).

Every pay-later receivable line is written with the follow-up exclusion **turned off** —
that is, the line *is* subject to dunning, unlike the other point of sale receivable
lines, because it represents a real debt of a real customer.

No accounting payment and no statement line are created: the money has not arrived. The
debt sits on the receivable account until it is settled through the ordinary receivable
flow (see section 10).

### 3.11 The invoice receivable counterweight

For every closed order that **is** invoiced, and for every one of its tenders that is not
of the pay-later kind, two things are recorded:

1. The receivable lines of the tender's own invoice payment entry (section 7) that sit on
   the company's default point of sale receivable account are remembered, keyed by
   payment method for aggregated tenders and by payment for identified-customer tenders.
2. The tender amount is accumulated in a parallel bucket, with the **order date** as the
   contribution date (not the payment date).

Each such bucket produces one journal item in the closing entry:

| Attribute | Value |
| --- | --- |
| Account | The company's default point of sale receivable account — never the method's intermediary account. |
| Name | `From invoice payments` |
| Display kind | Payment term line. |
| Amounts | **Credited** with the accumulated amount and converted amount. |

This credit exactly cancels the debit that the same tender produced in section 3.8 or
3.9, so the invoiced order contributes nothing net to the closing entry while its money
still reaches the bank or the drawer. The two sides are then reconciled against the
invoice payment entry (section 3.14).

### 3.12 Cost of goods sold and stock valuation

#### 3.12.1 Which moves are considered

Collect the identifiers of:

- the transfers of every order of the session — whatever its state — that is not invoiced
  and has no shipping date; and
- the transfers attached to the session that belong to no single order (the deferred ones
  created at closing).

Then take every stock move of those transfers whose product is storable and whose
valuation is real time. Moves of products valued periodically produce nothing here.

#### 3.12.2 The amount of a move

```formula
signed_quantity = convert_unrounded( move_quantity , from = move_unit , to = product_reference_unit )
```

```formula
signed_quantity = − signed_quantity        when the move is an incoming move
```

```formula
move_amount = signed_quantity × move_unit_price
```

The move unit price is the valuation unit price of the move as determined by the inventory
valuation domain. The contribution date is the completion date of the transfer, and the
contribution is declared to be **already in company currency**, so no conversion is
applied.

#### 3.12.3 The three buckets

| Bucket | Key | Fed by |
| --- | --- | --- |
| Cost of goods sold | The expense account of the product, mapped through the fiscal position of the transfer's order when it has one | Every considered move |
| Stock returned | The stock valuation account of the product (never mapped) | Incoming moves only |
| Stock delivered | The stock valuation account of the product (never mapped) | Outgoing moves only |

#### 3.12.4 The resulting journal items

- **Cost of goods sold line**: account = the expense account of the key, **debited** with
  the bucket amount, declared to be in company currency (so no amount in currency is
  carried even in a foreign-currency session).
- **Stock valuation line**: account = the stock valuation account of the key, **credited**
  with the bucket amount, also in company currency. One line is produced for the delivered
  bucket and one for the returned bucket, even when both land on the same account; both
  are created together for that account.

An outgoing move of a product costing 12.00 therefore produces a debit of 12.00 on the
expense account and a credit of 12.00 on the stock valuation account. An incoming move
(a return) of the same product produces a credit of 12.00 on the expense account and a
debit of 12.00 on the stock valuation account, because its amount is negative.

### 3.13 The rounding line

When and only when the configuration has cash rounding enabled, a rounding difference is
accumulated over the non-invoiced closed orders:

```formula
order_rounding_difference = order_amount_paid + order_total_amount_currency
```

where `order_total_amount_currency` is the sum of the amounts in currency of every base
line update and every tax line of that order — a **negative** number for an ordinary sale.
The contribution date is the order date.

The accumulated difference produces at most one journal item:

| Condition | Account | Amounts |
| --- | --- | --- |
| The accumulated amount is negative (a rounding loss) | The loss account of the configuration's rounding method | **Debited** with the negated amount and the negated converted amount |
| The accumulated amount is positive (a rounding gain) | The profit account of the configuration's rounding method | **Credited** with the amount and the converted amount |
| The accumulated amount is zero in both currencies | — | No line at all |

The line is always named `Rounding line`.

**Worked example.** A single order whose untaxed base is 10.00 and whose 21 percent tax is
2.10, giving 12.10, with a cash rounding step of 0.05 and the round-half-up method. The
payable amount becomes 12.10 (already a multiple of 0.05), so the difference is zero and
no line is produced. With a base of 10.03 and a tax of 2.11 the total is 12.14; the
payable amount becomes 12.15 and the customer hands over 12.15. Then
`order_rounding_difference = 12.15 + (−12.14) = +0.01`, a gain, credited 0.01 to the
rounding profit account.

### 3.14 Reconciliation performed at closing

Reconciliation is assembled as a single plan and executed once, with cash-basis tax
generation suppressed. The plan is built in this order.

**Step 1 — post the cash side.** Collect the receivable lines of the created cash
statement entries (the lines whose account is of the receivable kind), together with the
cash receivable lines of the closing entry. Post, without the soft option, every entry
among them that is not yet posted.

**Step 2 — cash reconciliation.** For each distinct account carried by those lines, if the
account is reconcilable, add to the plan the subset of those lines that sit on that
account and are not yet reconciled. This matches, per account, the credit produced by the
cash statement against the debit produced by the closing entry.

**Step 3 — aggregated bank and cash-method reconciliation.** For each payment method that
produced an aggregated bank bucket, if the receivable account of that method is
reconcilable, add to the plan the pair {closing-entry receivable line, accounting-payment
destination line}, restricted to the lines that are not yet reconciled.

**Step 4 — identified-customer bank reconciliation.** For each identified-customer bank
payment, if the receivable account of its customer is reconcilable, add to the plan the
pair {closing-entry receivable line, accounting-payment destination line}, restricted to
the unreconciled ones.

**Step 5 — invoiced-order reconciliation.** Only when the company's default point of sale
receivable account is reconcilable:

- for each payment method with an aggregated invoiced-order bucket, add the union of the
  remembered receivable lines of the invoice payment entries and the `From invoice
  payments` line of the closing entry;
- for each identified-customer payment of an invoiced order, add the same union computed
  per payment.

**Step 6 — execute.** If the plan is not empty, run it. The plan is executed entry by
entry in order, which is equivalent to reconciling each group separately, but the
recomputation cascade triggered by the created partial reconciliations runs once instead
of once per group.

### 3.15 The balancing line

When the closing entry does not balance, the whole transaction is rolled back and the
operator is offered the forced-close wizard. If the operator confirms, the closing is
retried with a balancing account and an amount to balance, and one extra line is created
at the very end:

| Attribute | Value |
| --- | --- |
| Name | `Difference at closing PoS session` |
| Account | The account chosen in the wizard. |
| Partner | None. |
| Amounts | **Credited** with the amount expressed in the selling currency and the amount expressed in company currency. |

The amount to balance is already in company currency. The selling-currency figure is
computed as
`convert( amount to balance , from = company currency , to = selling currency , company = the session's company , at = today )`
and is zero when the session already works in company currency.

Nothing is created when the amount to balance is zero at the company currency's precision.

The message shown in the wizard is:

> There is a difference between the amounts to post and the amounts of the orders, it is
> probably caused by taxes or accounting configurations changes.

### 3.16 Disposal of an empty closing entry

If, after all passes, the closing entry has no line at all, it is deleted instead of being
posted, and the orders of the session are **not** moved to the posted state. This happens
for a session whose only activity was cancelled orders.

---

## 4. The cash difference at closing

After the closing entry has been found to balance, the cash difference captured *before*
the payment statement lines were created is posted as one more bank statement line.

Let `difference = counted ending balance − theoretical closing balance`. Nothing is
created when it is zero.

| Attribute | Value |
| --- | --- |
| Journal | The session's cash journal. |
| Amount | The difference, signed. |
| Date | The date of the most recent existing cash statement line of the session, or today when there is none. |
| Session link | The session. |
| Label | `Cash difference observed during the counting (Loss) - closing` when the difference is negative, `Cash difference observed during the counting (Profit) - closing` when it is positive. |
| Counterpart account | The loss account of the cash journal for a negative difference, the profit account for a positive one. |

If the required account is missing the closing is refused:

> Please go on the *<cash journal name>* journal and define a Loss Account. This account
> will be used to record cash difference.

> Please go on the *<cash journal name>* journal and define a Profit Account. This account
> will be used to record cash difference.

After the statement line is created, a message is posted on its entry:
`Related Session: <a link to the session>`.

### 4.1 When the counterpart account itself carries taxes

Some jurisdictions require the cash difference to be taxed. When the loss or profit
account carries taxes, the statement line is created with explicit lines instead of a
counterpart account.

```formula
sign = +1 when the difference is positive, −1 when it is negative
```

The taxes of the counterpart account are evaluated over the **absolute value** of the
difference, with the price-included behavior forced on, in the journal's currency, for a
quantity of one.

```formula
cash_amount = sign × tax_inclusive_total
```

```formula
base_amount = − sign × tax_exclusive_total
```

```formula
tax_line_amount = − sign × tax_amount        for each tax produced by the engine
```

Conversion to company currency, where the journal currency differs from it, is done at the
statement line's date.

| Line | Account | Amount in currency | Balance |
| --- | --- | --- | --- |
| Liquidity | The default account of the cash journal | `cash_amount` | The converted `cash_amount` |
| Base | The counterpart (loss or profit) account, carrying the taxes and the base tags | `base_amount` | The converted `base_amount` |
| Tax, one per tax produced | The tax's own account, falling back to the counterpart account | `tax_line_amount` | The converted `tax_line_amount` |

Every line is named with the statement label, except the tax lines which are named with
the tax name. The tax lines carry the tax base amount (the converted tax-exclusive total),
the repartition line and the tax tags, and are marked as tax lines.

---

## 5. Manual cash in and cash out

At any moment during a session a permitted user may record a cash movement.

| Attribute | Value |
| --- | --- |
| Journal | The session's cash journal. |
| Amount | `+1 × amount` for a cash in, `−1 × amount` for a cash out. |
| Date | Today in the acting time zone. |
| Session link | The session. |
| Label | The session identifier, a hyphen, the translated movement kind, a hyphen, the reason typed by the user. |
| Partner | The partner recorded as the person performing the movement. |

The statement line produces the ordinary two-line entry of the statement model: the cash
account against the counterpart the operator's configuration determines. These lines feed
the theoretical closing balance (they are part of the sum of the session's statement
lines).

Deleting a cash movement is a separate permission; the deletion posts
`Cash move deleted: <cashier name>: <amount>` in the session thread.

A cash movement can only be attempted when the session has a cash journal; otherwise
*"There is no cash payment method for this PoS Session"*.

---

## 6. The customer invoice of a counter order

### 6.1 Header

| Attribute | Value |
| --- | --- |
| Journal | The invoice journal of the configuration. |
| Document kind | `out_invoice` (customer invoice) when the total of the group is greater than zero; `out_refund` (credit note) when it is less than zero; when the total is exactly zero, `out_refund` if every order of the group is a refund order, otherwise `out_invoice`. |
| Origin | The receipt numbers of the orders, joined by a comma and a space. |
| Reference | The order name for a single order; empty for a group. For a single order that refunds an invoiced order: `Reversal of: <the refunded order's invoice number>`, and the invoice is linked as the reversal of that invoice. |
| Counter orders | The orders. |
| Refunded invoices | The invoices of the orders refunded by the lines of these orders. |
| Customer | The invoice address of the order's customer. |
| Delivery address | The delivery address of the order's customer. |
| Bank account | See section 6.2. |
| Currency | The selling currency. |
| Invoice date | For a single order whose session is not closed, the order date; otherwise now. Expressed as a date in the acting time zone. |
| Salesperson | The order's employee. |
| Fiscal position | The order's fiscal position. |
| Payment terms | The customer's payment terms, but **only** when at least one tender of the order is of the pay-later kind; otherwise none. |
| Cash rounding definition | The configuration's rounding method, when the configuration has cash rounding and either rounding is not restricted to cash or at least one cash tender is present. |
| Narration | The floating order names of the orders that have one, joined by a comma and a space. |

### 6.2 Bank account selection

```
1. When the summed total is less than or equal to zero and the customer has bank accounts,
   take the first customer bank account that allows outgoing payments.
2. Otherwise, when the summed total is greater than or equal to zero and the order has at
   least one tender, take the bank account of the journal of the first tender's payment
   method, but only when that account allows outgoing payments.
3. When nothing was found and the summed total is greater than or equal to zero, take the
   first bank account of the company's own partner that allows outgoing payments.
4. Otherwise leave it empty.
```

### 6.3 Invoice lines

For each order in the group, each base line is converted into an invoice line.

Quantity sign:

```formula
is_refund_order = the order's refund flag is set, or its total is negative
```

```formula
quantity_sign = −1  when ( the document is a customer invoice and the order is a refund order )
                    or ( the document is a credit note and the order is not a refund order )
                +1  otherwise
```

| Product kind | Line produced |
| --- | --- |
| A combo header | A section line whose name is `<product name> x <quantity>` (the quantity rendered as a whole number when it is one), carrying the signed quantity and the product unit, and no amount. |
| Anything else | A product line carrying the product, the signed quantity, the discount percentage, the unit price, the rendered name, the taxes after fiscal position mapping, the product unit and the exported extra tax data. |

Two kinds of note lines are inserted:

- immediately after a product line, when the order's pricelist contains at least one
  percentage rule and the line's unit price is strictly below the product's list price, a
  note reading `Price discount from <list price> to <unit price>`, both rendered with the
  selling currency's decimal places;
- immediately after a product line, when the line carries a customer note, that note.

At the end of each order's lines, when the order has a general customer note, that note is
inserted as a note line.

### 6.4 Cash rounding on the invoice

When the configuration has cash rounding and the invoice carries a rounding definition, the
invoice's own rounding line is adjusted so that the invoice total matches what the customer
actually paid.

```formula
amount_paid = ( −1 when the summed order total is negative, +1 otherwise ) × the summed paid amount
```

```formula
difference_in_currency = invoice_direction_sign × ( amount_paid − invoice_total )
```

```formula
difference_in_balance = round_to_company_currency( difference_in_currency ÷ invoice_currency_rate )
```

(The balance difference is zero when the rate is zero.)

Nothing happens when the currency difference is zero. Otherwise:

- If the invoice already has a rounding line that is not a tax line, its amount in
  currency and its balance are each increased by the corresponding difference.
- Otherwise a new rounding line is created: account = the loss account of the rounding
  definition when the currency difference is positive, the profit account otherwise; name
  = the rounding definition's name; display kind = rounding; carrying the difference in
  currency and in balance.
- In both cases the payment-term line with the largest absolute amount in currency is
  reduced by the same difference, so that the invoice stays balanced.

### 6.5 Posting and messaging

The invoice is created with elevated rights, in the order's company, with invoice
synchronisation suppressed, and is posted immediately. A message is posted on it:
`This invoice has been created from the point of sale session:` followed by links to the
orders. A message is posted on each order thread by the platform's ordinary linking.

The rendered invoice document is generated and sent unless the acting context switches
generation off.

---

## 7. The invoice payment entry

One entry is created per tender of an invoiced order, in the **point of sale journal** of
the configuration — not in the bank or cash journal.

Tenders skipped entirely: those of the pay-later kind, and those whose amount is zero at
the selling currency's precision.

**Merging of change.** When the order has a negative cash change payment and at least one
positive cash payment, the change payment is not given an entry of its own; instead the
first positive cash payment's entry is written for the **sum** of the two amounts and is
linked to both payments. This makes the entry reflect the net cash actually kept.

| Attribute | Value |
| --- | --- |
| Journal | The point of sale journal of the session's configuration. |
| Date | The order date, expressed as a date in the order's time zone. |
| Reference | `Invoice payment for <order name> (<invoice number>) using <payment method name>` |
| Counter payments | The payment, plus the change payment when they were merged. |

Two lines:

| Line | Account | Partner | Side |
| --- | --- | --- | --- |
| Customer side | The receivable account of the accounting partner, read in the order's company | The accounting partner | **Credited** with the payment amount and its converted value |
| Counter side | See below | See below | **Debited** with the same pair |

The counter side account depends on whether the entry is being written during ordinary
invoicing or during the post-closing reversal path:

| Situation | Account | Partner on the line |
| --- | --- | --- |
| The session is still open (ordinary invoicing) | The company's default point of sale receivable account | None |
| The session is already closed and the method identifies the customer | The receivable account of the accounting partner | The accounting partner |
| The session is already closed and the method does not identify the customer | The intermediary account of the payment method, falling back to the company's default point of sale receivable account | None |

Both lines are written with the follow-up exclusion turned off. The entry is posted
immediately. The payment record keeps a link to it.

### 7.1 Reconciliation against the invoice

The receivable account of the invoice's accounting partner is taken. When that account is
not reconcilable, nothing happens. Otherwise the receivable lines of the payment entries
are selected by sign (section 6.4 of [`entities.md`](entities.md)): for a positive tender
only lines with a negative balance, for a negative tender only lines with a positive
balance. Those lines and the unreconciled receivable lines of the invoice are reconciled
together, with elevated rights, in the invoice's company.

---

## 8. Invoicing an order after its session has closed

The order's revenue, taxes, cost of goods sold and settlement are already inside a posted
closing entry. Issuing an invoice now would recognise them twice. The system therefore
creates a **reversal entry** that removes exactly that order's contribution from the
closing entry.

### 8.1 Building the mirror values

The order's own accounting values are rebuilt, per nature, as if the order were being
posted alone. Let

```formula
sign = +1 when the order total is negative, −1 otherwise
```

```formula
rate = conversion rate from the selling currency to the company currency at the order date
```

**Tax nature.** The tax lines the engine produces for the order, each marked as a tax line.

**Product nature.** For each base line update:

| Attribute | Value |
| --- | --- |
| Name | The line's full product name. |
| Product | The line's product. |
| Quantity | `line quantity × sign` |
| Account | The base line's account. |
| Partner | The base line's partner. |
| Currency | The base line's currency. |
| Taxes, tax tags | From the base line and its update. |
| Amount in currency | The update's amount in currency. |
| Balance | `round_to_company_currency( amount in currency × rate )` |
| Follow-up exclusion | Off. |

**Cash rounding nature.** Applied when the configuration has cash rounding, a rounding
method exists, and either rounding is not restricted to cash or a cash tender is present.

```formula
amount_currency = sign × round_to_currency( round_to_currency( running_total ) + total_payment_amount )
```
when rounding is restricted to cash and at least one non-cash tender is present, and
otherwise

```formula
amount_currency = rounding_definition_difference( selling currency , running_total )
```

where `running_total` is the sum, so far, of the tax and product amounts in currency. When
the result is zero nothing is added. Otherwise:

- with the **biggest-tax** strategy, the amount is folded into the tax line whose amount,
  multiplied by minus the sign, is the largest;
- with the **add-a-rounding-line** strategy, a line is added: account = the loss account
  when minus the sign times the amount is positive and a loss account exists, otherwise
  the profit account; name = the rounding definition's name; partner = the commercial
  partner; display kind = rounding.

**Stock nature.** For every move of the order's transfers whose product is valued in real
time, two lines are added, both in company currency and both named
`Stock variation for <product name>`:

```formula
balance = move_value      when the move is an outgoing move
balance = − move_value    otherwise
```

- the expense account with that balance,
- the stock valuation account with its negation.

**Payment-term nature.** For each tender:

- the receivable account is the customer's receivable account when the method identifies
  the customer, otherwise the method's intermediary account falling back to the company's
  default point of sale receivable account;
- tenders that do not identify the customer and land on the same account are merged into
  one line;
- each line carries the partner only when the method identifies the customer, a name made
  of the account code written twice separated by a space, the amount in currency equal to
  the tender amount, and a balance equal to the converted tender amount at the order date;
- the display kind is payment term.

**Residual correction.** The other balances were converted and rounded line by line, so in
a foreign-currency session the converted tender amounts can drift by a few cents. When the
amounts in currency already balance:

```formula
residual = round_to_company_currency( − running_total_balance − sum of payment term balances )
```

and, when that residual is not zero, it is added to the **last** payment-term line.

### 8.2 The reversal entry

Every value above is negated — balance, amount in currency and, where present, tax base
amount — and the whole set is written as a single entry:

| Attribute | Value |
| --- | --- |
| Journal | The point of sale journal of the configuration. |
| Date | Today in the acting time zone. |
| Reference | `Reversal of POS closing entry <closing entry number> for order <order name> from session <session identifier>` |
| Reversed counter order | The order. |
| Invoice synchronisation | Suppressed, for the entry and for its lines. |

The entry is posted immediately. Its signed total is negated when it is displayed, and it
is treated as a storno entry when the company uses storno accounting.

### 8.3 Reconciliation after the reversal

Let the accounting partner be the commercial entity of the order's customer, and let the
relevant accounts be the union of: the company's default point of sale receivable account,
the intermediary accounts of the payment methods of the order's tenders, and the partner's
receivable account.

Candidate lines are the lines of the reversal entry plus either:

- the lines of the invoice payment entries created for this order, when there are any; or
- the lines of the closing entry that carry this partner and sit on the partner's
  receivable account, when there are none.

Group the candidates by account, keeping only those on a relevant account that are not yet
reconciled, and reconcile each group.

---

## 9. Real-time cost of goods sold

When the company updates stock in real time, the delivery is completed at sale time and
the inventory valuation domain writes the ordinary stock entry then. The closing entry
still picks those moves up (section 3.12), because it aggregates moves rather than
accounting entries, and the entries the valuation domain wrote are separate documents.

For an invoiced order in a company using the cost-of-goods-at-invoicing model, the cost is
recognised on the invoice instead. The unit cost used there is obtained as follows:

1. Take the valued real-time moves of the order's transfers for that product, sorted by
   date; if there are any, use their valuation unit price.
2. Otherwise, when the product's cost method is standard or average, use its standard
   price.
3. Otherwise consume the first-in-first-out layers for the quantity and divide the
   resulting value by the quantity; use zero when the quantity is zero.

---

## 10. Settling a pay-later balance afterwards

A pay-later tender leaves a debit on a receivable account. Nothing further happens inside
this domain. The balance is settled through the ordinary receivable flow of the
[payments domain](../payments-and-bank-reconciliation/README.md): an accounting payment is
registered against the customer, and its receivable line is reconciled with the
closing-entry receivable line of the pay-later tender (or, when the order was invoiced,
with the invoice's receivable line).

Because the pay-later line of the closing entry is written with the follow-up exclusion
turned off, the outstanding amount appears in the customer's ageing and in dunning, unlike
the other point of sale receivable lines, which are settlement lines and are excluded.

---

## 11. Fully worked closing entry — the mandatory scenario

### 11.1 Given

A configuration whose selling currency is the company currency. One cash payment method
("Cash", cash journal with default account 570000, profit account 758000, loss account
658000), one bank payment method ("Card", bank journal, outstanding account 101401,
intermediary account left empty so the company's default point of sale receivable account
101300 is used), no cash rounding, no identify-customer flags.

Three orders, all in one session whose opening cash count was 0.00 and which had no manual
cash movement:

| Order | Content | Tender | Invoiced |
| --- | --- | --- | --- |
| A | One unit of product P at 25.00 including 21 percent tax; product cost 12.00; product valued in real time, expense account 600000, stock valuation account 140000; income account 400000 | Cash 25.00 | No |
| B | One unit of product Q at 40.50 including 21 percent tax; product cost 18.00; same accounts | Card 40.50 | No |
| C | One unit of product R at 100.00 including 21 percent tax; same accounts | Card 100.00 | **Yes**, customer Acme, receivable account 121000 |

The session is closed with a counted cash amount of 24.50.

### 11.2 Intermediate figures

Tax split, rounding each order separately to the currency:

```formula
base_A = round_to_currency( 25.00 ÷ 1.21 ) = 20.66      tax_A = 25.00 − 20.66 = 4.34
```

```formula
base_B = round_to_currency( 40.50 ÷ 1.21 ) = 33.47      tax_B = 40.50 − 33.47 = 7.03
```

Order C is invoiced, so its base and tax do not enter the closing entry at all.

Sales bucket (account 400000, sign +1, tax set {21 percent}, same base tags):

```formula
sales_amount = − ( 20.66 + 33.47 ) = − 54.13
```

Tax bucket (account 251000, the sale repartition line of the 21 percent tax):

```formula
tax_amount = − ( 4.34 + 7.03 ) = − 11.37        tax_base_amount = 20.66 + 33.47 = 54.13
```

Receivable buckets:

```formula
aggregated_cash[ Cash ]  = 25.00
```

```formula
aggregated_bank[ Card ]  = 40.50 + 100.00 = 140.50
```

```formula
invoiced_bank[ Card ]    = 100.00
```

Cost buckets (orders A and B only; order C is invoiced and excluded):

```formula
cost_of_goods_sold[ 600000 ] = 12.00 + 18.00 = 30.00
```

```formula
stock_delivered[ 140000 ]    = 12.00 + 18.00 = 30.00
```

### 11.3 The closing entry, line by line

Journal: the point of sale journal. Date: today. Reference: the session identifier.

| # | Line name | Account | Debit | Credit | Notes |
| --- | --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 tax payable | | 11.37 | Tax line; tax base amount 54.13; repartition line of the 21 percent tax; its tax tags |
| 2 | `Sales with 21%` | 400000 product sales | | 54.13 | Product line; carries the 21 percent tax and the base tags; quantity 1.00 |
| 3 | `<session> - Card` | 101300 point of sale receivable | 140.50 | | Payment-term line, no partner |
| 4 | `<session> - Cash` | 101300 point of sale receivable | 25.00 | | Payment-term line, no partner |
| 5 | `From invoice payments` | 101300 point of sale receivable | | 100.00 | Payment-term line, no partner |
| 6 | *(no name)* | 600000 cost of goods sold | 30.00 | | Written in company currency only |
| 7 | *(no name)* | 140000 stock valuation | | 30.00 | Written in company currency only |
| | **Totals** | | **195.50** | **195.50** | |

Note that line 3 carries the **whole** 140.50 taken on the card — the 40.50 of the
uninvoiced order B and the 100.00 of the invoiced order C — while line 5 gives back the
100.00 that belongs to the invoice. The net contribution of order C to this entry is zero.

There is no rounding line, because the configuration has no cash rounding.

### 11.4 The satellite documents

**Cash statement line** (cash journal, date today, label = the session identifier,
counterpart account 101300):

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| Liquidity | 570000 cash | 25.00 | |
| Counterpart | 101300 point of sale receivable | | 25.00 |

**Accounting payment for the card method** (bank journal, amount 140.50, inbound, forced
outstanding account 101401, destination 101300, memo
`Combine Card POS payments from <session>`):

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| Outstanding | 101401 outstanding receipts | 140.50 | |
| Destination | 101300 point of sale receivable | | 140.50 |

**Invoice of order C** (invoice journal, customer Acme, total 100.00 including 17.36 tax
on a base of 82.64):

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| Product | 400000 product sales | | 82.64 |
| Tax | 251000 tax payable | | 17.36 |
| Receivable | 121000 customer receivable | 100.00 | |

**Invoice payment entry of order C** (point of sale journal, date = order C's date,
reference `Invoice payment for <order C name> (<invoice number>) using Card`):

| Line | Account | Partner | Debit | Credit |
| --- | --- | --- | --- | --- |
| Counter side | 101300 point of sale receivable | none | 100.00 | |
| Customer side | 121000 customer receivable | Acme | | 100.00 |

**Cash difference** — the theoretical closing balance is
`0.00 + 0.00 + 25.00 = 25.00`; the counted amount is 24.50; the difference is −0.50, a
loss:

| Line | Account | Debit | Credit |
| --- | --- | --- | --- |
| Counterpart | 658000 cash difference loss | 0.50 | |
| Liquidity | 570000 cash | | 0.50 |

The statement label is
`Cash difference observed during the counting (Loss) - closing`.

### 11.5 What is reconciled against what

| Group | Lines reconciled | Account | Amount |
| --- | --- | --- | --- |
| Cash | Closing-entry line 4 (debit 25.00) against the counterpart line of the cash statement (credit 25.00) | 101300 | 25.00 |
| Card | Closing-entry line 3 (debit 140.50) against the destination line of the accounting payment (credit 140.50) | 101300 | 140.50 |
| Invoiced order | Closing-entry line 5 (credit 100.00) against the counter-side line of the invoice payment entry of order C (debit 100.00) | 101300 | 100.00 |
| Invoice settlement | The customer-side line of the invoice payment entry of order C (credit 100.00) against the receivable line of the invoice (debit 100.00) | 121000 | 100.00 |

The invoice settlement group is reconciled at invoicing time, not at closing time. The
other three are part of the closing reconciliation plan.

After closing, account 101300 carries no residual balance from this session, account
121000 carries no residual balance for Acme, account 570000 carries 24.50 (the counted
cash), account 101401 carries 140.50 awaiting the bank statement, and the loss on the cash
count of 0.50 sits in account 658000.

---

## 12. Worked entry — a refund of one line

### 12.1 Given

The same configuration. Order A of section 11 was paid in cash and delivered. The customer
returns the single unit of product P. The cashier opens the paid order, refunds the one
line, and gives 25.00 back in cash. No invoice is involved. The product's cost at the time
of the return is 12.00.

### 12.2 The refund order

A new order is created in the current session, copied from order A with:

- name = `<order A name> REFUND`;
- refund flag set;
- one line copied from order A's line with quantity `−( 1 − 0 ) = −1`, pointing at order
  A's line as the refunded line, cost not yet computed;
- one cash payment of −25.00.

### 12.3 The contribution to the closing entry

Base and tax of the refund line: base −20.66, tax −4.34. The base line is a refund line
(unit price × quantity is negative), so its key sign is −1 and it lands in its own bucket:

| # | Line name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | 4.34 | |
| 2 | `Refund with 21%` | 400000 | 20.66 | |
| 3 | `<session> - Cash` | 101300 | | 25.00 |
| 4 | *(no name)* | 600000 cost of goods sold | | 12.00 |
| 5 | *(no name)* | 140000 stock valuation | 12.00 | |

The cash bucket for the method is −25.00, so the debit helper produces a **credit** of
25.00. The statement line for the cash method is likewise −25.00, producing a credit on
the cash account and a debit on the receivable account, which reconciles against line 3.

The incoming stock move makes the cost bucket negative, so the cost-of-goods-sold line
becomes a credit and the stock valuation line a debit.

Note that if the original sale and the refund are in the **same** session, the buckets of
sections 11 and 12 are added together: the sales bucket keeps its −20.66 for product P and
a separate refund bucket carries +20.66, they are **not** netted into one line, because
the sign is part of the aggregation key. The cash bucket, in contrast, *is* netted,
because it is keyed only by the payment method: 25.00 − 25.00 = 0.00, and a zero cash
bucket produces neither a statement line nor a receivable line.

---

## 13. Worked entry — a price-included tax of twenty-one percent on twelve point one zero

### 13.1 Given

One order, one line, product with a 21 percent price-included tax, unit price 12.10,
quantity 1, no discount, no cash rounding, selling currency equal to company currency.

### 13.2 The split

```formula
tax_excluded = round_to_currency( 12.10 ÷ ( 1 + 21 ÷ 100 ) ) = round_to_currency( 10.0000000 ) = 10.00
```

```formula
tax_amount = 12.10 − 10.00 = 2.10
```

The tax amount is obtained by subtraction from the inclusive price, never by multiplying
the rounded base by the rate: `10.00 × 0.21 = 2.10` happens to agree here, but on a price
of 12.13 the base is 10.02 and the subtraction gives 2.11 while the multiplication would
give 2.10, and the subtraction is the one that keeps the total exact.

### 13.3 The contribution to the closing entry

| # | Line name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | 2.10 |
| 2 | `Sales with 21%` | 400000 | | 10.00 |
| 3 | `<session> - <method>` | 101300 | 12.10 | |

Line 2 carries the 21 percent tax in its tax set and the base tags of that tax's base
repartition line; line 1 carries the tax repartition line and its tags, and a tax base
amount of 10.00.

---

## 14. Worked entry — cash rounding to five hundredths

### 14.1 Given

The configuration has cash rounding with a rounding step of 0.05, the round-half-up
method, the add-a-rounding-line strategy, a rounding profit account 758100 and a rounding
loss account 658100. Rounding is **not** restricted to cash. One order, one line, unit
price 12.13 including 21 percent tax, quantity 1, paid in cash.

### 14.2 The amounts

```formula
tax_excluded = round_to_currency( 12.13 ÷ 1.21 ) = 10.02
```

```formula
tax_amount = 12.13 − 10.02 = 2.11
```

```formula
payable = round( 12.13 , step = 0.05 , method = round-half-up ) = 12.15
```

The customer hands over 12.15. The order's paid amount is 12.15; the order's total, as
reported by the tax totals summary with cash rounding applied, is 12.15 as well.

### 14.3 The rounding difference

```formula
order_total_amount_currency = ( − 10.02 ) + ( − 2.11 ) = − 12.13
```

```formula
order_rounding_difference = 12.15 + ( − 12.13 ) = + 0.02
```

A positive difference is a gain and is credited to the rounding profit account.

### 14.4 The contribution to the closing entry

| # | Line name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | 2.11 |
| 2 | `Sales with 21%` | 400000 | | 10.02 |
| 3 | `Rounding line` | 758100 rounding profit | | 0.02 |
| 4 | `<session> - Cash` | 101300 | 12.15 | |

Totals: debit 12.15, credit 12.15.

### 14.5 The mirror case

With a unit price of 12.12, the payable amount becomes 12.10; the customer hands over
12.10 while the untaxed base is 10.02 and the tax is 2.10, summing to 12.12. Then
`order_rounding_difference = 12.10 − 12.12 = − 0.02`, a loss, and line 3 becomes a
**debit** of 0.02 on account 658100.

### 14.6 When rounding is restricted to cash

With the restriction on, the rounding is applied only to the portion actually settled in
cash:

```formula
non_cash_amount = the sum of the amounts of the tenders whose method is not a cash method
```

```formula
payable = non_cash_amount + round( total − non_cash_amount , step , method )
```

For a total of 12.13 settled with 10.00 on a card and the rest in cash, the payable amount
is `10.00 + round( 2.13 , 0.05 , half up ) = 10.00 + 2.15 = 12.15`. The rounding
difference is again +0.02 and the entry is the same, except that the receivable side is
split between the card bucket (10.00) and the cash bucket (2.15).

---

## 15. Worked entry — a ten percent loyalty reward after one hundred points

### 15.1 Given

A loyalty programme grants one point per unit of currency spent and offers, at one hundred
points, a reward of ten percent off the order. The reward is realised as an ordinary order
line carrying the programme's discount product, with a negative amount. The discount
product's income account is 400000 and it carries the same 21 percent price-included tax
as the goods (the reward inherits the taxation of the lines it discounts).

The customer has 100 points and buys goods for 60.50 including tax. The reward line is
worth ten percent of the eligible amount.

### 15.2 The amounts

```formula
goods_included = 60.50        goods_excluded = round_to_currency( 60.50 ÷ 1.21 ) = 50.00        goods_tax = 10.50
```

```formula
reward_included = − round_to_currency( 60.50 × 10 ÷ 100 ) = − 6.05
```

```formula
reward_excluded = round_to_currency( − 6.05 ÷ 1.21 ) = − 5.00        reward_tax = − 6.05 − ( − 5.00 ) = − 1.05
```

```formula
order_total = 60.50 − 6.05 = 54.45
```

### 15.3 The contribution to the closing entry

The goods line and the reward line share the same account, the same tax set and the same
base tags, and both are ordinary (non-refund) lines of a non-refund order, so they fall
into the **same** bucket and are netted:

```formula
sales_amount = − ( 50.00 − 5.00 ) = − 45.00        tax_amount = − ( 10.50 − 1.05 ) = − 9.45
```

| # | Line name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | 9.45 |
| 2 | `Sales with 21%` | 400000 | | 45.00 |
| 3 | `<session> - <method>` | 101300 | 54.45 | |

If the reward product's income account differs from the goods' income account — which is
the usual configuration when discounts are tracked separately — the reward produces its
own bucket and its own line: a **debit** of 5.00 on the discount account and a **debit**
of 1.05 on the tax account, against credits of 50.00 and 10.50 on the goods accounts.

The loyalty card's point balance is reduced by one hundred when the order is transmitted;
that movement has no accounting effect. A gift card or an electronic wallet, in contrast,
is a liability and is treated by the
[loyalty and promotions domain](../loyalty-and-promotions/README.md).

---

## 16. Worked entry — a split bill

### 16.1 Given

A restaurant order totalling 100.00 including 21 percent tax. Two guests each settle
50.00: one in cash, one by card. The configuration has bill splitting enabled. Neither
payment method identifies the customer.

Splitting the bill does **not** split the order: one order carries two tenders.

### 16.2 The contribution to the closing entry

```formula
base = round_to_currency( 100.00 ÷ 1.21 ) = 82.64        tax = 100.00 − 82.64 = 17.36
```

| # | Line name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | 17.36 |
| 2 | `Sales with 21%` | 400000 | | 82.64 |
| 3 | `<session> - Cash` | 101300 | 50.00 | |
| 4 | `<session> - Card` | 101300 | 50.00 | |

One cash statement line of 50.00 and one accounting payment of 50.00 are produced, exactly
as in section 11.

### 16.3 The alternative: splitting into two orders

When the cashier instead moves some lines onto a second order, two independent orders
result, each with its own receipt number, its own tax split and its own tender. The
closing entry then aggregates their base lines into the same sales bucket (same account,
same sign, same tax set) but keeps the receivable contributions in the buckets of their
respective methods. Because each order's tax is rounded separately, the sum of the two tax
amounts may differ by one hundredth from the tax of the undivided order; this is the
expected behavior and the closing entry balances either way, since the receivable side is
the sum of what was actually tendered.

---

## 17. Worked entry — a self-ordered sale paid online

### 17.1 Given

A mobile self-ordering configuration with pay-after set to each order. The customer orders
goods for 30.25 including 21 percent tax and pays through a payment provider. The online
payment method is of the bank kind; its journal is the provider's journal and its
outstanding account is 101402.

### 17.2 What happens before closing

1. The order is created unfinished from the public self-ordering route, with its origin set
   to the mobile self-ordering value.
2. A payment transaction is created for the order's total, in the selling currency, through
   the [payment providers domain](../payment-providers/README.md).
3. When the provider confirms, the transaction creates a counter payment of 30.25 on the
   order with the online payment method, the order becomes paid, and the order-state and
   synchronisation notifications are broadcast.
4. Depending on the provider's own settlement model, the transaction may also create an
   accounting payment of its own; when it does, that payment is the one the closing entry
   reconciles against, and the online payment method's outstanding account is the
   provider's.

### 17.3 The contribution to the closing entry

```formula
base = round_to_currency( 30.25 ÷ 1.21 ) = 25.00        tax = 30.25 − 25.00 = 5.25
```

| # | Line name | Account | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | 5.25 |
| 2 | `Sales with 21%` | 400000 | | 25.00 |
| 3 | `<session> - <online method>` | 101300 | 30.25 | |

plus an accounting payment of 30.25 in the provider's journal: debit 101402, credit 101300.
Line 3 and the destination line of that payment are reconciled.

---

## 18. Worked entry — a pay-later order settled afterwards

### 18.1 Given

A customer-account payment method with the identify-customer flag set (the usual
configuration, since a debt must belong to someone). Customer Acme, receivable account
121000. An order for 60.50 including 21 percent tax, wholly tendered on the customer
account, not invoiced.

### 18.2 The contribution to the closing entry

```formula
base = round_to_currency( 60.50 ÷ 1.21 ) = 50.00        tax = 60.50 − 50.00 = 10.50
```

| # | Line name | Account | Partner | Debit | Credit |
| --- | --- | --- | --- | --- | --- |
| 1 | `21%` | 251000 | | | 10.50 |
| 2 | `Sales with 21%` | 400000 | | | 50.00 |
| 3 | `<session> - Customer Account` | 121000 customer receivable | Acme | 60.50 | |

No statement line, no accounting payment: nothing was received. Line 3 is written with the
follow-up exclusion turned off, so Acme's 60.50 appears in the ageing report and in
dunning.

### 18.3 Settlement afterwards

Later, Acme pays 60.50 by bank transfer. The payments domain registers an inbound
accounting payment of 60.50 whose destination account is 121000. Its receivable line
(a credit of 60.50 on 121000, partner Acme) is reconciled against line 3 above (a debit of
60.50 on 121000, partner Acme). Account 121000 for Acme returns to zero and the bank
journal carries the receipt.

### 18.4 If the pay-later order is invoiced instead

When the same order is invoiced, the pay-later tender produces **no** closing-entry line at
all (section 3.10 excludes invoiced orders) and **no** invoice payment entry (section 7
skips pay-later tenders). The invoice alone carries the receivable, with the customer's
payment terms applied, because the presence of a pay-later tender is precisely the
condition under which the order's invoice inherits the customer's payment terms. Settlement
then follows the ordinary accounts receivable flow.
