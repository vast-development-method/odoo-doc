# Expenses — Accounting Effects

This file lists **every** journal entry the expense domain produces, with the journal used, each journal item, its account selection rule, its side, its amount formula, its currency handling, its date, its partner, its analytic distribution, its tax treatment and its reconciliation behaviour.

The sibling folder `../accounts-payable/accounting-effects.md` specifies the payable side of a purchase document in general. An employee-paid expense produces a **purchase receipt**, which is a purchase document; everything that folder says about the balance invariant, the payable term line, the reversal mechanics and the reconciliation of a payable line therefore applies here unchanged, and is **linked, not restated**. What this file adds is what is specific to expenses: which account is debited, how the tax is derived from a tax-inclusive receipt, how the counterpart is chosen from the payment mode, and what the company-paid path does instead.

---

## 1. Conventions

- **Debit** means a positive balance in the company currency; **credit** means a negative one. A line's amount in currency always carries the same sign as its balance.
- `round_to(currency, x)` rounds `x` to the rounding step of that currency (see `../multi-currency/calculations.md`).
- *Receipt currency* means the expense's own currency (`currency_id`); *company currency* means the currency of the expense's company.
- The **balance invariant** of `../general-ledger/accounting-effects.md` holds for every entry described here: the sum of debits equals the sum of credits in company currency, and for each currency present the amounts in that currency sum to zero.

---

## 2. What produces an entry, and what does not

| Event | Journal entry produced? |
|---|---|
| Creating, editing, submitting, approving or refusing an expense | **No.** Nothing accounting happens before posting. |
| Splitting an expense | **No.** A split is only possible while the expense is not posted. |
| Posting an **employee-paid** expense | **Yes** — one purchase receipt per employee per posting operation (§3). |
| Posting a **company-paid** expense | **Yes** — one payment, whose own entry is the accounting document, one per expense (§4). |
| Registering the reimbursement of an employee-paid expense | **Yes** — an ordinary outgoing payment, owned by `../payments-and-bank-reconciliation/` (§5). |
| Reconciling that payment against a bank statement line | **Yes** — owned by `../payments-and-bank-reconciliation/`. |
| Resetting a posted expense to draft | **Yes when the entry was posted** — a cancelling reversal (§6). **No when the entry was still draft** — the entry is simply deleted. |
| Deleting the entry or the payment by hand | **No new entry**; the expense simply loses its link (§7). |
| Rebilling an expense onto a sales order | **No.** The rebilling creates a sales order line, not an entry. The customer invoice raised later from that line is an ordinary customer invoice, specified in `../accounts-receivable/accounting-effects.md` (§8). |

---

## 3. Posting an employee-paid expense

### 3.1 The document produced

One **purchase receipt** — a journal entry whose document kind is `in_receipt` — per **employee** per posting operation. Two expenses of the same employee posted in the same operation share one receipt; two expenses of different employees posted in the same operation produce two receipts; the same expense posted alone produces its own receipt.

| Header field | Value |
|---|---|
| Number (`name`) | Forced to `/` at creation so that the numbering series assigns the number at posting. Any default number carried in the calling context is deliberately overridden. |
| Document kind (`move_type`) | `in_receipt` — purchase receipt. |
| Journal (`journal_id`) | The journal chosen in the posting dialogue; see `entities.md` §6.1 for its default resolution. The dialogue offers purchase journals only, but the entry-level check that a document kind must match the journal kind is **suspended** for entries carrying expenses, so a receipt may legitimately be written into a journal of any kind. |
| Bill date (`invoice_date`) | The accounting date chosen in the posting dialogue. |
| Accounting date (`date`) | Derived from the bill date by the general rule of `../general-ledger/` — briefly: when the bill date falls in a month earlier than the current month and the numbering series resets monthly, the accounting date becomes the **last day of the bill month**; otherwise it becomes the later of the bill date and today; lock dates push it further forward. |
| Reference (`ref`) | The description of the single expense when there is exactly one; otherwise *"Expenses of «employee name»"*. |
| Partner (`partner_id`) | The employee's **work contact**. |
| Commercial partner (`commercial_partner_id`) | The employee's **user partner**. It is also recomputed by the rule of `entities.md` §8.1: the partner's own commercial partner, unless that is the company's own partner, in which case the partner itself. An employee whose contact hangs under the company's contact record is therefore still billed to themselves, not to the company. |
| Currency (`currency_id`) | The **company currency**, always. A foreign-currency receipt produces a company-currency entry; the receipt currency appears nowhere on the entry. |
| Company (`company_id`) | The expense's company (which may be a branch). |
| Recipient bank account (`partner_bank_id`) | The employee's primary bank account, so that the later reimbursement proposes it. |
| Expenses (`expense_ids`) | Every expense of the group. |
| Attachments | A **copy** of every attachment of every expense of the group, re-owned to the entry. The originals stay on the expenses. The first such copy is forced as the entry's main attachment, without the usual filtering of structured-document files. |
| Payment term | The **supplier payment term of the employee's work contact**, applied exactly as for a vendor bill (see `../accounts-payable/calculations.md`). |

### 3.2 The lines

| # | Kind | Account | Side | Amount in currency (= balance, the entry being in company currency) |
|---|---|---|---|---|
| 1..n | product line, one per expense of the group | the expense account resolved by the algorithm of `calculations.md` §6.2 | **debit** | `+ untaxed amount of the expense in company currency` |
| n+1..m | tax line, one per (tax distribution line, expense) pair | the account of the tax distribution line, falling back to the product line's own account | **debit** for an ordinary deductible purchase tax; **credit** when the distribution factor is negative | `+ tax amount` |
| last | payment term line, one per maturity date produced by the payment term | the payable account chosen by the ordinary purchase-document rule of `../accounts-payable/calculations.md` §6.1 | **credit** | `− instalment amount` |

#### 3.2.1 The product line, field by field

| Field | Value |
|---|---|
| Label (`name`) | *"«employee name»: «first line of the expense description, truncated to 64 characters»"*. The description is split on line breaks and only the first line is used. |
| Account | `calculations.md` §6.2. |
| Quantity | The expense quantity, or **1** when the quantity is zero. |
| Unit price | The expense unit price, which is **tax-inclusive** (see §3.3). |
| Unit of measure | The expense unit. |
| Product | The expense category. |
| Taxes | The expense's tax set. |
| Analytic distribution | The expense's analytic distribution, copied verbatim. |
| Expense (`expense_id`) | The expense. |
| Partner | The employee's work contact. Then overridden, like every line of the entry, to the entry's partner — which is the same value. |
| Display type | `product`. |

#### 3.2.2 Why the debit is the untaxed amount although the unit price is the total

Because the taxes of an expense behave as **price-included**. Two mechanisms enforce this on the receipt:

1. When the entry hands a product line to the tax engine, a line whose expense is employee-paid is handed over in **forced total-included mode**, whatever the taxes themselves declare.
2. When a line's own displayed totals are computed, lines carrying an expense are computed with the price-inclusion flag forced on.

So for a product line of quantity *q* and unit price *p*:

```formula
line_total_including_tax  = round_to(company_currency, q × p)
line_subtotal             = the base returned by the tax engine in total-included mode
line_tax                  = line_total_including_tax − line_subtotal
```

and the **debit of the product line is `line_subtotal`**, not `q × p`.

The full arithmetic of the price-included tax, including the multi-rate and compounding cases, is in `calculations.md` §4.

#### 3.2.3 The tax lines

The tax engine aggregates tax amounts into one line per **accounting grouping key**. For an expense receipt that key contains: the partner, the currency, the analytic distribution, the account, the tax set, the tax distribution line, the enclosing tax group, the tax report tags — **and the expense**. The expense being part of the key is an expense-specific addition, and its consequence is:

> Two expenses on the same receipt that bear the same tax produce **two separate tax lines**, never one merged line.

Other rules, unchanged from `../taxes/`:

- The account of a tax line is the account named on the tax distribution line; when that distribution line names none, the **product line's own account** is used.
- The analytic distribution is copied onto the tax line only when the tax is flagged as analytic, or when the distribution line is not used in the tax closing; otherwise the tax line carries none.
- A tax line whose amount rounds to zero in both currencies is dropped, unless it is explicitly marked as a line to keep.
- The rounding remainder between the sum of the distribution lines and the tax total is distributed smoothly over the distribution lines, largest absolute amount first.
- For a **cash-basis** tax, the receipt being an ordinary invoice-type document, the tax line lands on the tax's **cash-basis transition account** and carries no report tags until the receipt is paid. (Contrast §4.3.)

#### 3.2.4 The payment term line

Produced by the ordinary purchase-document mechanism, with the employee's work contact as the partner:

- one line per distinct maturity date, merged when two instalments fall on the same day;
- account chosen by the payable-account rule of `../accounts-payable/calculations.md` §6.1 — in practice the work contact's payable property, falling back to its parent's;
- credit of the whole receipt total (the sum of the product and tax lines, with the sign reversed);
- label empty unless a payment reference is set on the receipt;
- maturity date from the payment term, or the bill date when there is none.

### 3.3 Reconciliation behaviour

| Line | Reconcilable? |
|---|---|
| Product line | No — expense accounts are not reconcilable accounts. |
| Tax line | No. |
| Payment term line | **Yes.** It is the line a reimbursement payment is matched against, and the line that determines the expense's *Posted* / *In Payment* / *Paid* status. |

### 3.4 Worked example A — an employee-paid meal of 60.50 including ten per cent tax

*Setting.*

| Item | Value |
|---|---|
| Company | Northwind; company currency Euro; rounding step 0.01 |
| Employee | Dana Okwu; work contact *Dana Okwu*; payable account **400000 Account Payable**; no supplier payment term |
| Expense category | *Meals*, code `FOOD`, unit cost **0.00**, unit *Units*, expense account **600300 Meals** |
| Tax | *Purchase 10 %*, tax-excluded by declaration, one hundred per cent of the tax to account **131000 Tax Paid** |
| Journal | *Expenses* (purchase) |
| Accounting date chosen in the posting dialogue | 14 March 2026 |

*Capture.* Because the category's unit cost is zero, the employee types the total.

```formula
quantity              = 1
total_amount_currency = 60.50        (typed by the employee)
price_unit            = round_to(Euro, 60.50 ÷ 1) = 60.50
currency              = Euro = company currency, so currency_rate = 1
```

*Tax derivation, in forced price-included mode.*

```formula
untaxed_amount_currency = round_to(Euro, 60.50 ÷ (1 + 10 ÷ 100)) = round_to(Euro, 55.00) = 55.00
tax_amount_currency     = 60.50 − 55.00 = 5.50
untaxed_amount          = 55.00        (mono-currency shortcut)
tax_amount              = 5.50         (mono-currency shortcut)
```

*The receipt.*

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Lunch with customer | 600300 Meals | **55.00** | |
| 2 | Purchase 10 % | 131000 Tax Paid | **5.50** | |
| 3 | *(empty)* | 400000 Account Payable | | **60.50** |

Header: kind *purchase receipt*; journal *Expenses*; bill date 14 March 2026; accounting date 14 March 2026 (the bill date is in the current month); reference *"Lunch with customer"*; partner *Dana Okwu*; currency Euro; recipient bank account the employee's.

Line 1 carries quantity 1, unit price 60.50, unit *Units*, product *Meals*, taxes {Purchase 10 %}, and the expense's analytic distribution. Line 2 carries a tax base amount of 55.00. Line 3 has a maturity date of 14 March 2026.

*Reimbursement.* A payment of 60.50 is registered from the bank journal (§5):

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | | 400000 Account Payable | **60.50** | |
| 2 | | Outstanding Payments | | **60.50** |

Line 1 of the payment is reconciled against line 3 of the receipt. The receipt's payment status becomes *in payment*; the expense's status becomes *In Payment* (or *Paid* where the in-payment hook collapses it — `state-machines.md` §2.4). When the bank statement line of −60.50 is later reconciled against the outstanding line, the payment becomes *paid*, the receipt becomes *paid*, and the expense becomes *Paid*.

### 3.5 Worked example B — an employee-paid distance claim of 120 kilometres at 0.30 each

*Setting.* Category *Mileage*, code `MIL`, unit cost **0.30** per kilometre, unit *Kilometres*, expense account **600400 Travel**, **no taxes**. Company currency Euro. Employee Dana Okwu, payable account 400000.

*Capture.* The category has a non-zero unit cost, so:

- the currency is forced to the company currency;
- the unit price is taken from the category, converted into the expense unit: `price_unit = 0.30` per kilometre;
- the employee types only the quantity: `quantity = 120`;
- the total is derived.

```formula
total_amount_currency = tax_included_total( price_unit = 0.30, quantity = 120, taxes = {} )
                      = round_to(Euro, 120 × 0.30) = 36.00
total_amount          = 36.00
untaxed_amount        = 36.00
tax_amount            = 0.00
```

*The receipt.*

| # | Label | Account | Quantity | Unit price | Debit | Credit |
|---|---|---|---|---|---|---|
| 1 | Dana Okwu: Client visit — Bruges | 600400 Travel | 120 kilometres | 0.30 | **36.00** | |
| 2 | *(empty)* | 400000 Account Payable | | | | **36.00** |

No tax line, because the category carries no tax. The quantity and the unit survive onto the journal item, which matters for two downstream consumers: the analytic line takes its unit amount from the quantity, and a rebilling line takes its own quantity from it.

*If the distance category carried a twenty-one per cent included tax instead:*

```formula
untaxed_amount_currency = round_to(Euro, 36.00 ÷ 1.21) = round_to(Euro, 29.752066…) = 29.75
tax_amount_currency     = 36.00 − 29.75 = 6.25
```

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Client visit — Bruges | 600400 Travel | **29.75** | |
| 2 | Purchase 21 % | 131000 Tax Paid | **6.25** | |
| 3 | *(empty)* | 400000 Account Payable | | **36.00** |

Note that the tax is *carved out of* the 36.00, not added to it: a distance allowance of 0.30 per kilometre costs the company 36.00 whether or not a tax is declared. This is the practical meaning of the price-included rule.

### 3.6 Worked example C — two expenses of one employee posted together

*Setting.* Dana Okwu has two approved employee-paid expenses, both taxed at fifteen per cent:

| Expense | Category | Quantity | Unit price | Total | Untaxed | Tax |
|---|---|---|---|---|---|---|
| *Office chairs* | Furniture, unit cost 800.00, account **610010 Expense Account 1** (overridden on the expense) | 2 | 800.00 | 1 600.00 | 1 391.30 | 208.70 |
| *Desk lamp* | Lamps, unit cost 160.00, account **600100 Expenses (lamps)**, **two** fifteen-per-cent purchase taxes | 1 | 160.00 | 160.00 | 123.08 | 36.92 |

```formula
1 600.00 ÷ 1.15 = 1 391.304347…  → round_to(Euro, …) = 1 391.30 ;  tax = 208.70
  160.00 ÷ 1.30 =   123.076923…  → round_to(Euro, …) =   123.08 ;  tax =  36.92
```

The second expense bears **two** fifteen-per-cent taxes, so the divisor is `1 + 0.15 + 0.15 = 1.30`, and the 36.92 of tax is split evenly between the two taxes at 18.46 each.

*The single receipt.*

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Office chairs | 610010 Expense Account 1 | **1 391.30** | |
| 2 | Purchase 15 % | 131000 Tax Paid | **208.70** | |
| 3 | Dana Okwu: Desk lamp | 600100 Expenses (lamps) | **123.08** | |
| 4 | Purchase 15 % | 131000 Tax Paid | **18.46** | |
| 5 | Purchase 15 % (second tax) | 131000 Tax Paid | **18.46** | |
| 6 | *(empty)* | 400000 Account Payable | | **1 760.00** |

Header reference: *"Expenses of Dana Okwu"* — two expenses, so the group label is used.

Three separate tax lines appear on the same account 131000: lines 2 and 4 are the **same tax** on **different expenses**, and lines 4 and 5 are **different taxes** on the same expense. Both distinctions are part of the accounting grouping key.

### 3.7 Worked example D — a foreign-currency employee expense of 100 at a rate of 1.10

*Setting.* Company currency Euro. Receipt currency United States Dollar. Expense date 4 May 2026, on which one United States Dollar buys 1.10 Euro. Category *Travel & Accommodation*, unit cost zero, account **600400 Travel**, one ten-per-cent purchase tax. Employee-paid.

*Capture.*

```formula
currency              = United States Dollar ≠ Euro, so the expense is multi-currency
total_amount_currency = 100.00                                   (typed)
currency_rate         = conversion_rate(United States Dollar → Euro, 4 May 2026) = 1.10
total_amount          = tax_included_total( price_unit = 100.00 × 1.10, quantity = 1 ) = 110.00
price_unit            = round_to(Euro, total_amount ÷ quantity) = 110.00
```

*The two tax pairs.* The tax engine is run twice, once per currency.

```formula
untaxed_amount_currency = round_to(Dollar, 100.00 ÷ 1.10) = round_to(Dollar, 90.909090…) = 90.91
tax_amount_currency     = 100.00 − 90.91 = 9.09

untaxed_amount          = round_to(Euro, 110.00 ÷ 1.10) = 100.00
tax_amount              = 110.00 − 100.00 = 10.00
```

Note that 9.09 × 1.10 = 9.999, **not** 10.00. The two currency figures are computed independently from their own totals and are not required to be exact multiples of each other. This is deliberate: the receipt shown to the tax authority of the foreign country must read 9.09, and the company's books must read 10.00.

*The receipt* — entirely in Euro; the Dollar figures are kept only on the expense record.

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Hotel Lisbon | 600400 Travel | **100.00** | |
| 2 | Purchase 10 % | 131000 Tax Paid | **10.00** | |
| 3 | *(empty)* | 400000 Account Payable | | **110.00** |

Line 1 carries quantity 1 and unit price **110.00** — the company-currency unit price, because the entry currency is the company currency.

The employee is therefore reimbursed **110.00 Euro**, and no exchange difference can ever arise on this receipt, because it holds no foreign-currency amount to revalue. The exchange risk is borne entirely at capture time, by the choice of the conversion rate (or by the manual override of §5.4 of `calculations.md`).

---

## 4. Posting a company-paid expense

### 4.1 What is created, and in what order

For **each** company-paid expense, individually:

1. Prepare the entry values and the payment values from the expense.
2. Create the **entry** first, with elevated rights, with its lines fully written out.
3. Create the **payment**, pointing at that entry.
4. Write back onto the entry: the originating payment, **and the journal identifier again** — the journal must be re-asserted because setting the originating payment triggers a recomputation chain that would otherwise void the company currency of the lines.
5. When every expense has been processed, post the **payments** (not the entries): posting the payment posts its entry at the same time.

The entry produced is a **miscellaneous entry** (document kind `entry`), not a purchase receipt. The dynamic term-line mechanism therefore never runs on it — that mechanism only applies to invoice-kind documents — and the counterpart line is written explicitly instead.

A constraint enforces one expense per entry on this path: an entry that carries a company-paid expense and more than one expense is rejected with *"Each expense paid by the company must have a distinct and dedicated journal entry."*

### 4.2 The entry header

| Header field | Value |
|---|---|
| Number (`name`) | Forced to `/`. |
| Document kind | `entry` — miscellaneous. |
| Journal | The **journal of the chosen payment method line** (a bank, cash or credit-card journal). When no payment method line is set, the posting aborts with *"You need to add a manual payment method on the journal («journal name»)"*. |
| Accounting date (`date`) | The **expense date**, or today when the expense has no date. Not adjusted by the bill-date rule of §3.1, because a miscellaneous entry is not an invoice. |
| Bill date (`invoice_date`) | Not set. |
| Reference (`ref`) | The expense description. |
| Partner (`partner_id`) | The **vendor** of the expense, which may be empty. **Not** the employee: the company paid a third party directly. |
| Currency | The **receipt currency** of the expense. Unlike the employee path, a company-paid entry is a genuine foreign-currency entry. |
| Company | The expense's company (which may be a branch). |
| Expenses | The one expense. |
| Attachments | A copy of every attachment of the expense, re-owned to the entry. |

### 4.3 The lines

Exactly three kinds of line, written explicitly, never synthesised:

| # | Kind | Account | Side | Amount in receipt currency | Balance in company currency |
|---|---|---|---|---|---|
| 1 | base line | the expense account resolved by `calculations.md` §6.2 | **debit** | `+ untaxed amount in receipt currency` as returned by the tax engine, after rounding and delta distribution | `+ (total amount in company currency − Σ tax line balances)` — see §4.3.2 |
| 2..k | tax lines | the account of the tax distribution line, falling back to the base line's account | **debit** for an ordinary deductible purchase tax; **credit** for a negative distribution factor | `+ tax amount in receipt currency` | `+ tax amount in company currency` |
| last | outstanding line | the destination account chosen by `calculations.md` §6.3 | **credit** | `round_to(receipt currency, − total amount in receipt currency)` | `− total amount in company currency` |

#### 4.3.1 How the tax engine is driven on this path

The expense is turned into **one** base line with:

| Base-line input | Value |
|---|---|
| unit price | the **receipt-currency total** of the expense |
| quantity | 1 |
| account | the resolved expense account |
| taxes | the expense's tax set |
| analytic distribution | the expense's analytic distribution |
| partner | the vendor |
| currency | the receipt currency |
| tax mode | **total included**, forced |
| rate | `abs( total amount in receipt currency ÷ total amount in company currency )`, or 0 when the company-currency total is zero |

The rate is the number of units of **receipt** currency per unit of **company** currency — the reciprocal of the expense's own conversion rate. The tax engine converts a receipt-currency figure into a company-currency figure by **dividing** by this rate:

```formula
company_amount = round_to(company_currency, receipt_amount ÷ rate)
```

With the expense's conversion rate written as *r* (company currency per unit of receipt currency), the engine's rate is `1 ÷ r`, and dividing by `1 ÷ r` is multiplying by *r*, as expected.

**Cash-basis handling.** On this path the accounting data is prepared with the cash-basis tag flag **switched on**. Two consequences, both opposite to the employee path:

1. A cash-basis tax's line goes to its **final** tax account, not to its cash-basis transition account.
2. The base line and the tax line carry their **tax report tags** immediately.

This is correct because the money has already left the company: the tax is already exigible.

#### 4.3.2 The balance override that guarantees a balanced entry

After the tax lines have been appended, the base line's **balance** is overwritten:

```formula
base_line_balance = total_amount_in_company_currency − Σ ( balance of every tax line )
```

Its **amount in receipt currency** keeps the value the tax engine produced. The outstanding line then carries exactly `− total_amount_in_company_currency` as its balance, so the three kinds of line sum to zero by construction:

```formula
( total_amount − Σ tax balances ) + ( Σ tax balances ) + ( − total_amount ) = 0
```

This override exists because the entry is a miscellaneous entry, which is subject to the automatic-balancing mechanism of `../general-ledger/`: any residual imbalance would silently create a fourth, spurious line on a suspense account. Forcing the base balance guarantees that no such line is ever created — including in the multi-currency case, where independent rounding of the base and of the taxes would otherwise leave a one-cent gap.

#### 4.3.3 The outstanding line, field by field

| Field | Value |
|---|---|
| Label | *"«employee name»: «first line of the expense description, truncated to 64 characters»"* — the same label as the base line. |
| Account | The destination account of `calculations.md` §6.3: the **dedicated outstanding account of the payment method line** when it has one, otherwise the company's outstanding-payments account for outbound payments, created on the spot if the chart does not have one yet. |
| Balance | `− total amount in company currency`. |
| Amount in currency | `round_to(receipt currency, − total amount in receipt currency)`. |
| Currency | The receipt currency. |
| Partner | The vendor. |
| Analytic distribution | None. |
| Taxes | None. |

The payable/receivable nature check that normally applies to a counterpart line is **suspended** for lines whose expense is company-paid, precisely because this account is an outstanding-payments account and not a payable one.

### 4.4 The payment-term override, and why it is inert here

The domain overrides the needed-term computation so that, for an entry carrying at least one company-paid expense, the term requirement becomes exactly one line with:

- maturity date = **today** (not the bill date, not the payment term's date);
- balance = the negated sum of the balances of every non-term line;
- amount in currency = the negated sum of their amounts in currency;
- label = the entry's payment reference, or the empty string;
- account = the destination account of `calculations.md` §6.3.

This requirement is only ever **applied** to entries of an invoice kind; the standard company-paid entry is a miscellaneous entry, where the outstanding line of §4.3.3 already plays that role, so the override has no visible effect there. It exists so that any invoice-kind entry that ends up carrying a company-paid expense still gets its counterpart on the outstanding account rather than on the partner's payable account.

### 4.5 The payment record

| Payment field | Value |
|---|---|
| Date | The **expense date**. |
| Memo | The expense description. |
| Journal | The payment method line's journal. |
| Amount | The **receipt-currency** total of the expense. |
| Direction | `outbound`. |
| Partner kind | `supplier`. |
| Partner | The vendor (possibly empty). |
| Currency | The receipt currency. |
| Payment method line | The one chosen on the expense. |
| Company | The expense's company. |
| Entry | The entry created in step 2. |
| Outstanding account | Overridden to the destination account of `calculations.md` §6.3, instead of the journal's own default. |
| Recipient bank account requirement | Forced off — a payment created from an expense never demands one, and never demands that it be a trusted account. |

Once created, the payment is **frozen**: writing its date, amount, direction, partner kind, payment reference, currency, partner, destination account, recipient bank account, journal, memo or payment method line raises *"You cannot do this modification since the payment is linked to an expense."* Other fields, such as the "sent" marker, stay writable.

### 4.6 Reconciliation behaviour

| Line | Reconcilable? |
|---|---|
| Base line | No. |
| Tax lines | No. |
| Outstanding line | **Yes** — that is the point of an outstanding account. It is matched later against the bank statement line that shows the money leaving. That matching has **no effect whatsoever on the expense**, which is already *Paid*. |

### 4.7 Worked example E — a company-paid fare of 25.00

*Setting.* Company currency Euro. Category *Travel & Accommodation*, unit cost zero, account **600400 Travel**, **no taxes**. Payment mode *Company*. Payment method line *Manual* on the journal **Bank**, whose dedicated outstanding account is **101401 Outstanding Payments**. Expense date 2 April 2026. No vendor.

*Capture.* `total_amount_currency = 25.00`; `quantity = 1`; `price_unit = 25.00`; `currency_rate = 1`; `untaxed_amount_currency = 25.00`; `tax_amount_currency = 0.00`.

*The entry.*

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Airport train | 600400 Travel | **25.00** | |
| 2 | Dana Okwu: Airport train | 101401 Outstanding Payments | | **25.00** |

Header: kind *miscellaneous entry*; journal *Bank*; accounting date 2 April 2026; reference *"Airport train"*; partner empty; currency Euro.

Line 1's balance was computed as `25.00 − 0.00 = 25.00`; its amount in currency is the tax engine's untaxed figure, also 25.00.

*The payment.* Outbound, supplier, 25.00 Euro, journal *Bank*, method *Manual*, dated 2 April 2026, memo *"Airport train"*, linked to the entry.

*The expense status* becomes **Paid** immediately.

*With a twenty per cent included tax instead:*

```formula
untaxed = round_to(Euro, 25.00 ÷ 1.20) = round_to(Euro, 20.8333…) = 20.83
tax     = 25.00 − 20.83 = 4.17
```

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Airport train | 600400 Travel | **20.83** | |
| 2 | Purchase 20 % | 131000 Tax Paid | **4.17** | |
| 3 | Dana Okwu: Airport train | 101401 Outstanding Payments | | **25.00** |

Line 1's balance is `25.00 − 4.17 = 20.83`, which happens to equal the tax engine's own figure; the override matters only when rounding makes them differ.

### 4.8 Worked example F — a company-paid foreign-currency expense of 100 at a rate of 1.10

*Setting.* As in §3.7 (Dollar receipt, Euro company, rate 1.10, ten-per-cent tax, account 600400, tax account 131000) but with payment mode *Company* and the *Manual* method on the *Bank* journal, outstanding account 101401. Vendor *Hotel Atlântico*.

*Capture.* `total_amount_currency = 100.00`; `currency_rate = 1.10`; `total_amount = 110.00`.

*Tax engine inputs.*

```formula
rate given to the engine = abs( 100.00 ÷ 110.00 ) = 0.909090909…
base line unit price     = 100.00 (Dollar), quantity 1, mode total included
```

*Tax engine outputs.*

```formula
untaxed in Dollar = round_to(Dollar, 100.00 ÷ 1.10)                 = 90.91
tax     in Dollar = 100.00 − 90.91                                  =  9.09
untaxed in Euro   = round_to(Euro,  90.909090… ÷ 0.909090909…)      = 100.00
tax     in Euro   = round_to(Euro,   9.090909… ÷ 0.909090909…)      =  10.00
```

*Balance override.* `base_line_balance = 110.00 − 10.00 = 100.00`.

*The entry* — currency United States Dollar, company currency Euro.

| # | Label | Account | Amount in Dollar | Debit (Euro) | Credit (Euro) |
|---|---|---|---|---|---|
| 1 | Dana Okwu: Hotel Lisbon | 600400 Travel | +90.91 | **100.00** | |
| 2 | Purchase 10 % | 131000 Tax Paid | +9.09 | **10.00** | |
| 3 | Dana Okwu: Hotel Lisbon | 101401 Outstanding Payments | −100.00 | | **110.00** |

Both invariants hold: the Euro column sums to zero (100.00 + 10.00 − 110.00), and the Dollar column sums to zero (90.91 + 9.09 − 100.00).

*The payment* is for **100.00 United States Dollar**, dated the expense date, on the *Bank* journal, partner *Hotel Atlântico*.

Because line 3 carries a foreign-currency amount on a reconcilable account, settling it later against a bank statement line at a different rate **does** produce an exchange difference, handled by `../multi-currency/accounting-effects.md`. This is the structural difference from the employee path, which never leaves a foreign-currency amount on the books.

---

## 5. Reimbursing an employee

Reimbursement is an ordinary outgoing payment registered against the purchase receipt, using the register-payment dialogue of `../payments-and-bank-reconciliation/`. The expense domain changes that dialogue in exactly two ways.

### 5.1 The recipient bank account proposed

When a line being paid belongs to an entry that carries an **employee-paid** expense, and the entry itself names no recipient bank account, the grouping key of the batch is extended with a recipient bank account taken from, in order:

1. the employee's **primary bank account** (read with elevated rights);
2. otherwise, the **first** bank account of the line's partner.

Because the recipient bank account is part of the grouping key, two employees are never merged into one payment, and an employee with no bank account is never merged with one who has one.

### 5.2 The expense stamped onto the payment's lines

After the payments are built, every line of a payment created from expense lines is stamped with the **first** expense found among the lines being paid. This makes the payment reachable from the expense and vice versa.

### 5.3 The entry produced

Unchanged from `../payments-and-bank-reconciliation/accounting-effects.md`:

| # | Account | Side | Amount |
|---|---|---|---|
| 1 | the destination account of the paid lines — the employee's payable account | **debit** | the amount paid |
| 2 | the journal's outstanding-payments account | **credit** | the amount paid |

Line 1 is reconciled against the receipt's payment term line, in full or partially. Partial reconciliation leaves a residual, which is what makes the expense show *In Payment* rather than *Paid* (see `state-machines.md` §4.1).

### 5.4 Effect on the expense

| Reconciliation state of the receipt | Expense status |
|---|---|
| nothing reconciled | Posted |
| partially reconciled, residual not zero | In Payment |
| fully covered by payments, bank not yet reconciled | In Payment |
| fully reconciled including the bank side | Paid |
| payments unreconciled again | Posted |

---

## 6. Resetting a posted expense

The reset operation is specified step by step in `state-machines.md` §3.5. Its accounting consequences:

### 6.1 A draft entry

Deleted outright. No accounting trace remains. Any rebilling line the entry produced has its quantities reset to zero and is unlinked (§8.3).

### 6.2 A posted entry

**Reversed in cancellation mode**, with the reversal's bill date set to **today**:

| Aspect | Rule |
|---|---|
| Journal | The same journal as the original. |
| Document kind | The reverse of the original's kind: a purchase receipt reverses into a purchase receipt of opposite sign; a miscellaneous entry reverses into a miscellaneous entry. |
| Lines | Every line of the original, with debit and credit exchanged, and with the amounts in currency negated. |
| Date | The bill date is forced to today; the accounting date follows the general rule. |
| Expenses carried | **None.** The link between the original entry and its expenses is cleared **before** the reversal runs, so neither the original nor the reversal points at the expenses afterwards. |
| Reconciliation | Cancellation mode means the reversal is immediately reconciled against the original on every reconcilable account, so the payable line of an employee receipt and its mirror net to zero. |
| Effect on a reimbursement already made | The original payable line may already be partly reconciled with a payment. The reversal reconciles against the **residual**; any payment already matched stays matched, and the difference remains outstanding on the payable account until an accountant deals with it. This is the ordinary behaviour of `../accounts-payable/workflows.md`. |

### 6.3 Worked example G — resetting the meal of example A after it was posted but before it was paid

Original receipt (§3.4): 55.00 debit to 600300, 5.50 debit to 131000, 60.50 credit to 400000.

Reversal, dated today:

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Lunch with customer | 600300 Meals | | **55.00** |
| 2 | Purchase 10 % | 131000 Tax Paid | | **5.50** |
| 3 | *(empty)* | 400000 Account Payable | **60.50** | |

Line 3 is reconciled with line 3 of the original; both become fully reconciled and the employee is owed nothing. The expense's approval state, approval date and entry reference are then cleared, and its status becomes *Draft*.

---

## 7. Removing the accounting by hand

| Operation | Accounting effect | Effect on the expense |
|---|---|---|
| The entry is reset to draft, then deleted | The entry disappears. | The entry reference becomes null; the status falls back to *Approved*. Rebilling lines are reset (§8.3). |
| The entry is **cancelled** | The entry stays in the ledger as a cancelled document. Its link to the expenses is cleared as part of the cancellation. | The status falls back to *Approved*, and the expense may be posted again. Cancelling the entry is explicitly **not** cancelling the expense. |
| The **payment** of a company-paid expense is deleted | Its entry goes with it. | The entry reference becomes null; the status falls back to *Approved*. |
| A reimbursement payment is reset to draft and unreconciled | The payment's own entry returns to draft; the receipt's payable line becomes unreconciled again. | The status returns from *Paid* or *In Payment* to *Posted*. |

---

## 8. Rebilling to a customer

### 8.1 No entry of its own

Rebilling produces **no journal entry**. It produces a **sales order line**, and the journal entry that eventually results is the customer invoice raised from that order by the ordinary sales invoicing flow (`../sales/workflows.md`, `../accounts-receivable/accounting-effects.md`).

### 8.2 Where the rebilling hooks into posting

Rebilling rides on the **analytic lines** created when the expense's journal entry is posted:

1. Posting the entry creates one analytic line per analytic account named in each line's distribution (see `../analytic-accounting/`), with

   ```formula
   analytic_line_amount     = − journal_item_balance × distribution_percentage ÷ 100
   analytic_line_unit_amount = journal_item_quantity
   ```

   An expense line being a debit, its analytic amount is **negative** — a cost.
2. Before those analytic lines are stored, each journal item is asked whether it may be rebilled. For an item that carries an expense the answer is yes exactly when **all three** hold: the expense category's rebilling policy is *at cost* or *at sales price*; the expense names a sales order; the item's display type is `product`.
3. For each rebillable item, the sales order is determined — for an expense item, simply the expense's own sales order (with the project fallback of §8.6).
4. A sales order line is created (§8.4), and the analytic line is stamped with it.
5. The sales order line's delivered quantity is then computed from the analytic lines pointing at it, summing their unit amounts over lines whose amount is at most zero.

Reversal entries are excluded from step 2: an item belonging to a reversal never creates a rebilling line.

### 8.3 Resetting a rebilling line

Whenever a posted expense entry is **reset to draft**, **reversed**, or **deleted**, every rebilling line of the expenses it carried is written back to:

- ordered quantity **0**;
- delivered quantity **0**;
- no linked expenses.

The line itself is **not** deleted: it stays on the order at zero, and a fresh posting creates a **new** line rather than reviving the old one. The write is performed with elevated rights after checking that the caller may write the expense, because an employee who may edit their expense usually may not edit a sales order.

### 8.4 The rebilling line created

| Sales order line field | Value |
|---|---|
| Order | The expense's sales order. |
| Label | The journal item's label, that is *"«employee name»: «expense description»"*. |
| Sequence | The highest sequence on the order plus one, or 100 when the order has no line. |
| Unit price | The rebilling price of §8.5. |
| Taxes | The **customer** taxes of the product, filtered to the order's company and then mapped through the order's fiscal position. |
| Discount | 0. |
| Product | The expense category. |
| Unit | The journal item's unit. |
| Ordered quantity | The journal item's quantity; then, when the policy is *at sales price* **and** the category has a non-zero unit cost, overwritten with the **expense's** quantity. |
| Is an expense line | Yes. This flag switches the line's delivered-quantity method to *analytic*. |
| Analytic distribution | The journal item's analytic distribution. |
| Expenses | The expense. |
| Expense (margin) | The expense, when the expense-margin capability is active. |

**Never merged.** For ordinary rebilling from vendor bills, a line priced at sales price on a delivery-invoiced product may be merged with an existing identical line. For expenses this reuse is **switched off**: every rebilled expense is forced onto its own new line, so that quantities can be adjusted or reset per expense without disturbing another.

### 8.5 The rebilling price

```formula
at sales price :  price = pricelist_price( product, quantity 1, unit of the item, at the order date )

at cost, same currency :
    price = round_to(company_currency, abs( ( credit − debit ) ÷ item_quantity ) )

at cost, different currencies :
    price = convert( abs( ( credit − debit ) ÷ item_quantity ),
                     from = company currency, to = order currency,
                     at the order date, falling back to today )
```

with the guard that a zero item quantity yields a price of zero.

For an expense line the credit is zero and the debit is the untaxed amount, so `abs((credit − debit) ÷ quantity)` is the **untaxed cost per unit**. Rebilling *at cost* therefore recovers the cost **net of tax**.

### 8.6 The project fallback

When the project-and-sales capability is active, the order is determined from the **project** first and from the **expense** second, the expense winning where both answer. So an expense that names no sales order but whose analytic distribution points at a project with a sales order is still rebilled to that order.

### 8.7 Guards raised while creating the rebilling line

Evaluated for each expense being posted, before any line is created:

| Condition | Message |
|---|---|
| The order is still a quotation or a sent quotation | *"The Sales Order «order number» to be reinvoiced must be validated before registering expenses."* |
| The order is cancelled | *"The Sales Order «order number» to be reinvoiced is cancelled. You cannot register an expense on a cancelled Sales Order."* |
| The order is locked | *"The Sales Order «order number» to be reinvoiced is currently locked. You cannot register an expense on a locked Sales Order."* |

### 8.8 Worked example H — an expense rebilled at cost on a sales order

*Setting.* Company currency Euro. Sales order **S00032** for customer *Deco Addict*, confirmed, in Euro, with one ordinary line for 2 units of *Consulting* at 100.00. The order's analytic account is **AA-S00032**. Category *Travel & Accommodation*: unit cost **55.00**, unit *Units*, rebilling policy **at cost**, invoicing policy *delivered quantities*, customer tax *Sales 15 %*, expense account **600400 Travel**, **no supplier tax**. Employee Dana Okwu, payable account 400000, expense paid by the employee.

*Capture.* The category has a unit cost, so the currency is forced to Euro and the employee types only the quantity.

```formula
quantity              = 11.30
price_unit            = 55.00
total_amount_currency = round_to(Euro, 11.30 × 55.00) = 621.50
untaxed_amount        = 621.50      (no tax)
```

The expense names sales order S00032 and carries the analytic distribution `{ AA-S00032 : 100 }`.

*Posting.* The receipt:

| # | Label | Account | Quantity | Unit price | Debit | Credit |
|---|---|---|---|---|---|---|
| 1 | Dana Okwu: Site visits Q1 | 600400 Travel | 11.30 | 55.00 | **621.50** | |
| 2 | *(empty)* | 400000 Account Payable | | | | **621.50** |

*The analytic line* created from line 1:

```formula
amount      = − 621.50 × 100 ÷ 100 = − 621.50
unit_amount = 11.30
```

*The rebilling price.*

```formula
credit − debit = 0 − 621.50 = − 621.50
price          = round_to(Euro, abs( − 621.50 ÷ 11.30 )) = round_to(Euro, 55.00) = 55.00
```

*The sales order line created:*

| Field | Value |
|---|---|
| Order | S00032 |
| Label | *"Dana Okwu: Site visits Q1"* |
| Product | Travel & Accommodation |
| Unit price | **55.00** |
| Ordered quantity | **11.30** (the journal item's quantity; the policy is *at cost*, so no override) |
| Delivered quantity | **11.30**, computed from the analytic line's unit amount |
| Taxes | *Sales 15 %*, after fiscal-position mapping |
| Is an expense line | yes |
| Analytic distribution | `{ AA-S00032 : 100 }` |
| Expenses | the expense |

The order's untaxed total rises by 621.50. When the order is invoiced, the customer invoice includes a line of 11.30 × 55.00 = 621.50 plus fifteen per cent sales tax, which is an ordinary customer invoice line specified in `../accounts-receivable/accounting-effects.md`.

*The margin.* With the expense-margin capability, the line's cost per unit is

```formula
line_cost_per_unit = 621.50 ÷ 11.30 = 55.00
line_margin        = ( 55.00 − 55.00 ) × 11.30 = 0.00
```

because the rebilling is at cost. Had the policy been *at sales price* with a list price of 80.00, the line would carry price 80.00, cost 55.00 and a margin of `(80.00 − 55.00) × 11.30 = 282.50`.

### 8.9 Worked example I — the same expense rebilled at sales price

Same setting, but the category's rebilling policy is **at sales price**, its list price is 80.00, and its unit cost is 55.00 (non-zero).

```formula
price = pricelist_price( Travel & Accommodation, quantity 1, unit Units, at the order date ) = 80.00
```

Because the policy is *at sales price* **and** the category has a non-zero unit cost, the ordered quantity is overwritten with the **expense's** quantity, which is also 11.30 here. The line reads 11.30 × 80.00 = 904.00 untaxed, with a delivered quantity of 11.30 and a cost per unit of 55.00.

If the category had a **zero** unit cost — the usual case for a "hotel" category where the employee types the total — the ordered quantity would stay at the journal item's quantity, which is 1, and the line would read 1 × 80.00.

---

## 9. Cross-checks with the payable domain

| Statement | Agreement |
|---|---|
| An employee-paid expense entry is a **purchase receipt**, one of the three payable document kinds listed in `../accounts-payable/entities.md`. | Yes. |
| Its payable term line is chosen by the rule of `../accounts-payable/calculations.md` §6.1, and is the counterpart of outgoing payments. | Yes — with the employee's work contact as the partner. |
| The signed amounts and the direction follow the payable convention: expense lines debit, term lines credit. | Yes. |
| Every line of a posted payable document carries the commercial partner, the accounting date and the document currency. | Yes, with the expense-specific commercial-partner rule of `entities.md` §8.1. |
| The document appears in the invoice analysis report with one row per product line and signed quantities. | Yes — with the quantity and the unit taken from the expense. |
| Reversal uses the cancelling method described in `../accounts-payable/workflows.md`. | Yes, with the bill date of the reversal forced to today. |
| Mass posting, the to-check flag, duplicate bill detection and automatic vendor posting apply. | They apply to the entry as a purchase document, but the expense domain never triggers them: an expense entry is always created already complete and posted in the same operation. |

The one place where an expense entry departs from the payable model is the **company-paid** path, which is not a payable document at all: it is a payment. It is listed here because it debits the same expense accounts and consumes the same tax configuration, but it never touches a payable account and never appears among documents awaiting payment.

---

## 10. Summary of every account used

| Account role | Chosen by | Used on |
|---|---|---|
| Expense account | `calculations.md` §6.2 — the expense's own account, else the category's, else the company's, else the payment journal's default | The product/base line of both paths, and as the fallback account of a tax line whose distribution names none |
| Tax account | The tax distribution line; its cash-basis transition account on the employee path, its final account on the company path | The tax lines |
| Employee payable account | The work contact's payable property, else its parent's | The payment term line of the employee path; the debit of the reimbursement payment |
| Outstanding payments account | The payment method line's dedicated account, else the company's outbound outstanding account | The counterpart line of the company path; the credit of the reimbursement payment |
| Bank or cash account | The bank journal's account | The bank statement side, outside this domain |
| Customer receivable, sales revenue, sales tax | The ordinary sales rules | The customer invoice raised from a rebilling line, outside this domain |
