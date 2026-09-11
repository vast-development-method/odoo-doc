# Accounts Payable — Accounting Effects

This file lists every journal entry that the payable domain produces, with the journal used, each journal item, its account selection rule, its side, its amount, its currency handling, its date, its partner, its analytic distribution, its tax treatment and its reconciliation behaviour.

Conventions used throughout:

- **Debit** means a positive balance in company currency; **credit** means a negative balance. The amount in document currency always carries the same sign as the balance.
- A line's **amount in currency** is the amount in the document currency; its **balance** is the amount in the company currency.
- Every accountable line of a purchase document carries the document's **commercial partner** (forced at posting), the document's **accounting date**, and the document's **currency**.
- An expense line carries its own **analytic distribution**; the tax engine copies that distribution onto the tax lines of taxes flagged as analytic.

---

## 1. The balance invariant

Every posted journal entry satisfies

```formula
Σ debit over all lines = Σ credit over all lines
```

in company currency, and, for each currency present on the entry,

```formula
Σ amount_in_currency over the lines of that currency = 0
```

For a vendor bill this means: expense lines plus tax lines plus non-deductible lines (which net to zero among themselves except for the tax shift) equal, in absolute value, the payable term lines.

---

## 2. Posting a vendor bill

### 2.1 Journal

The **purchase journal** chosen on the document. Its numbering series gives the document its number.

### 2.2 Lines produced

| # | Display type | Account | Side | Amount in document currency | Balance |
|---|---|---|---|---|---|
| 1..n | `product` | the expense account chosen by `calculations.md` §6.2 | **debit** | `+ price_subtotal` of the line, as returned by the tax engine after rounding and delta distribution | `+ price_subtotal ÷ document_rate`, rounded to the company currency |
| n+1..m | `tax` | the account of the tax distribution line that produced it (the **purchase** distribution for a bill, the **refund** distribution for a vendor credit note) | **debit** for an ordinary deductible purchase tax; credit when the distribution factor is negative (reverse charge) | `+ tax amount` aggregated per accounting grouping key | the same, converted |
| optional | `rounding` | the cash rounding rule's **loss** account when the adjustment reduces what is owed, its **profit** account otherwise | either | the cash rounding difference | the same, converted |
| optional | `non_deductible_product` | the **same expense account as the product line it derives from** | **credit** | `− non_deductible_base` (see `calculations.md` §10) | the same, converted |
| optional | `non_deductible_product_total` | the journal's **private share account**, falling back to its default account | **debit** | the sum of the private bases | the same, converted |
| optional | `non_deductible_tax` | the journal's **private share account**, falling back to its default account (or the account already on an existing such line) | **debit** | the tax on the private bases | the same, converted |
| last | `payment_term` | the payable account chosen by `calculations.md` §6.1 | **credit** | `− instalment amount` in document currency | `− instalment amount` in company currency |

There is **one payable term line per distinct maturity date** produced by the payment term, merged when two instalments fall on the same day. Each carries its own **maturity date**, and, when the term grants an early discount, its **discount date**, **discounted amount in currency** and **discounted balance**.

Section, subsection and note lines carry no account and no amount and are not part of the entry in the accounting sense.

### 2.3 Sequence of the lines

Presentation and product lines keep sequence 100 (or whatever the user ordered them to); tax lines get 10000; rounding lines 11000; payable term lines 12000. The private-share product lines get *(product line sequence + 1)*; the private-share totals get *(highest sequence + 1)*.

### 2.4 Reconciliation behaviour

- Expense, tax, rounding and private-share lines are **not** reconcilable (their accounts normally are not).
- The payable term lines are on a **reconcilable** account and become the counterpart of outgoing payments, of vendor credit notes and of miscellaneous entries.
- A term line with a zero amount is still produced when the document total is zero; such a document is immediately treated as paid.

### 2.5 Worked example — a bill of two expense lines with a purchase tax

*Setting.*

| Item | Value |
|---|---|
| Company | Northwind; company currency Euro; rounding factor 0.01 |
| Journal | Vendor Bills (purchase); default expense account **600000 Expenses** |
| Vendor | Papeterie Lambert; payable account **400000 Account Payable**; vendor payment term *Immediate* (one line, percent 100, `days_after` 0) |
| Tax | **Purchase 15 %**, tax-excluded, one hundred per cent distribution to the account **131000 Tax Paid** |
| Product A | "Office chair"; expense account 600000; supplier tax Purchase 15 %; purchase price 800.00 per unit |
| Product B | "Desk lamp"; expense account **600100 Expenses (lamps)**; supplier taxes Purchase 15 % **and** a second purchase tax **Purchase 15 % (b)**, also tax-excluded, distributing to 131000; purchase price 160.00 per dozen |
| Bill date and accounting date | 1 January 2026 |
| Vendor reference | `LAM-9912` |

*Lines typed.* One unit of Product A at 800.00; one dozen of Product B at 160.00.

*Step 1 — line defaulting.*

- Line 1: label *Office chair*; unit *Units*; quantity 1; unit price 800.00; taxes {Purchase 15 %}; account 600000.
- Line 2: label *Desk lamp*; unit *Dozens* (the product's reference unit, and also the supplier unit); quantity 1; unit price 160.00; taxes {Purchase 15 %, Purchase 15 % (b)}; account 600100.

*Step 2 — tax computation* (round globally, the industry-standard default of this system; see `../taxes/calculations.md`).

```formula
line 1 subtotal = round_to(Euro, 1 × 800.00 × (1 − 0 ÷ 100)) = 800.00
line 1 total    = 800.00 + round_to(Euro, 800.00 × 15 ÷ 100) = 800.00 + 120.00 = 920.00

line 2 subtotal = round_to(Euro, 1 × 160.00 × (1 − 0 ÷ 100)) = 160.00
line 2 tax a    = round_to(Euro, 160.00 × 15 ÷ 100) = 24.00
line 2 tax b    = round_to(Euro, 160.00 × 15 ÷ 100) = 24.00
line 2 total    = 160.00 + 24.00 + 24.00 = 208.00
```

Aggregating per tax:

```formula
tax "Purchase 15 %"     = 120.00 + 24.00 = 144.00
tax "Purchase 15 % (b)" =            24.00 =  24.00
```

Because the two taxes distribute to the **same** account 131000 but are **different taxes**, the grouping key differs (the originator tax is part of the key), so **two** tax lines are produced.

*Step 3 — document totals.*

```formula
amount_untaxed = 800.00 + 160.00 = 960.00
amount_tax     = 144.00 + 24.00  = 168.00
amount_total   = 1 128.00
```

*Step 4 — the payment term.* One line, percent 100, zero days, so one instalment of 1 128.00 due **1 January 2026**. The document's due date is 1 January 2026.

*Step 5 — the payable account.* No other payable term line exists on the document, so rule 2 applies: the vendor's payable account property, **400000**.

*Step 6 — the label of the term line.* No payment reference; the document is a bill with a vendor reference, so the label is `LAM-9912`. The term has one line, so no instalment suffix.

*Step 7 — the posted journal entry.*

| Line | Display type | Account | Label | Partner | Debit | Credit | Amount in currency | Maturity |
|---|---|---|---|---|---|---|---|---|
| 1 | `product` | 600000 Expenses | Office chair | Papeterie Lambert | 800.00 | — | +800.00 | — |
| 2 | `product` | 600100 Expenses (lamps) | Desk lamp | Papeterie Lambert | 160.00 | — | +160.00 | — |
| 3 | `tax` | 131000 Tax Paid | Purchase 15 % | Papeterie Lambert | 144.00 | — | +144.00 | — |
| 4 | `tax` | 131000 Tax Paid | Purchase 15 % (b) | Papeterie Lambert | 24.00 | — | +24.00 | — |
| 5 | `payment_term` | 400000 Account Payable | LAM-9912 | Papeterie Lambert | — | 1 128.00 | −1 128.00 | 1 January 2026 |

Totals: debit 1 128.00, credit 1 128.00. ✓

Each tax line carries the reporting grids (tax tags) of its distribution line; each expense line carries the **base** grids of the taxes applied to it. Line 3's tax base amount is 960.00 (the sum of the bases it taxes); line 4's is 160.00.

*Step 8 — the signed totals.*

```formula
amount_untaxed_signed = − (800.00 + 160.00) = − 960.00
amount_tax_signed     = − (144.00 + 24.00)  = − 168.00
amount_total_signed   = − 1 128.00
amount_residual       = − direction_sign × (− 1 128.00) = − (+1) × (−1 128.00) = + 1 128.00
amount_residual_signed= − 1 128.00
```

*Variant — the bill in a foreign currency.* Same bill written in United States dollars with a document rate of 1.20 dollars per euro. The commercial amounts stay 800.00, 160.00, 144.00, 24.00 and 1 128.00 **dollars**; every balance becomes the dollar amount divided by 1.20 and rounded to the euro: 666.67; 133.33; 120.00; 20.00; and the term line −940.00. Note 666.67 + 133.33 + 120.00 + 20.00 = 940.00 exactly, because the tax engine distributes the rounding delta onto the first line of each group; when it does not, the term line, being the balance, absorbs the residue so that the entry still balances.

---

## 3. Posting a vendor credit note

Identical to §2 with every side reversed, because the direction sign is −1:

| Display type | Account | Side |
|---|---|---|
| `product` | the expense account | **credit** |
| `tax` | the **refund** distribution line's account of the tax | **credit** |
| `payment_term` | the payable account | **debit** |

The tax distribution used is the **refund** distribution rather than the invoice distribution, which may point at a different account and carries different reporting grids — this is what lets a country report purchases and purchase returns on distinct lines of its tax return.

The payable term line of a vendor credit note is a **debit**, so it reconciles naturally against the credit of a vendor bill.

---

## 4. Posting a purchase receipt

Identical to §2. A purchase receipt is an outbound document like a bill; the only differences are:

- its fiscal position defaults to the company's **default purchase receipt fiscal position** rather than being detected from the vendor;
- it is excluded from the **exact duplicate** predicate (it can still be a probable duplicate);
- some views and filters treat it separately.

Its journal is a purchase journal and its numbering shares the bill series unless the journal defines otherwise.

---

## 5. Reversing a bill

### 5.1 The three reversal shapes

| Operation | What is created | Is the original neutralised? |
|---|---|---|
| **Reverse** (called *Add Credit Note* on an invoice, *Reverse* on an entry) | one vendor credit note per bill, posted or scheduled | No — the credit note is an independent document that the user may reconcile or not |
| **Reverse and cancel** | one vendor credit note per bill, posted immediately and **reconciled** against the bill | Yes |
| **Reverse and modify** (*Modify*) | the cancelling reversal above, **plus** a new draft bill that copies only the product, section, subsection and note lines of the original | Yes, and the replacement is ready to edit |

### 5.2 The reversal document's defaults

| Field | Value |
|---|---|
| type | the mirror type: a bill reverses into a vendor credit note, a vendor credit note into a bill, a purchase receipt into a vendor credit note |
| reversed entry | the original |
| partner | the original's partner |
| vendor reference | *Reversal of: «original number»* — or *Reversal of: «original number», «reason»* when a reason was typed — rendered in the partner's language |
| accounting date | the reversal date chosen in the dialogue |
| bill date | the same date, for an invoice-like document |
| due date | the same date |
| journal | the journal chosen in the dialogue, which must be of the **same type** as the original's journal, otherwise *Journal should be the same type as the reversed entry.* |
| payment term | **cleared**, unless the original's term computes its early discount in the `mixed` mode, in which case the original term is kept (so that the tax reduction already booked is reproduced) |
| salesperson | copied |
| origin | copied |
| automatic posting | `at_date` when the reversal date is in the future, `no` otherwise |
| recipient bank account | cleared and then recomputed |
| main attachment | for a purchase document in *modify* mode, the original's main attachment is **copied** onto the replacement so that the supplier's file follows |

The lines are copied and their amounts are **negated only for miscellaneous entries and cost-of-goods lines**. For an invoice-like document the lines are copied as they are and the mirror **type** does the sign inversion, because the direction sign of the mirror type is the opposite of the original's.

When the company enables storno accounting, copied lines of a miscellaneous entry also have their storno flag inverted.

### 5.3 The reconciliation performed by a cancelling reversal

1. Before copying, every reconciliation of every line of the original is **removed**.
2. The reversal is created and posted.
3. Then, for the pair *(original, reversal)*: take all their lines that are not reconciled, sort them so that receivable and payable lines come **first**, group them by *(account, currency)*, and, for each group where no line is yet reconciled and the account is reconcilable **or** its type is cash or credit card, reconcile the group.

### 5.4 Refusals

| Condition | Message |
|---|---|
| the selected documents belong to more than one company | *All selected moves for reversal must belong to the same company.* |
| any selected document is not posted | *To reverse a journal entry, it has to be posted first.* |
| the chosen journal's type differs from the original's | *Journal should be the same type as the reversed entry.* |

### 5.5 The chatter

The original receives *This entry has been reversed* with a link to the reversal; the reversal receives *This entry has been reversed from «link to the original»*.

### 5.6 Worked example — a refund of a partly paid bill

*Setting.* Continue from §2.5. The bill `BILL/2026/01/0003` of 1 128.00 Euro, vendor reference `LAM-9912`, posted on 1 January 2026 with a single payable term line of 1 128.00 credit due the same day.

*Step 1 — a partial payment.* On 20 January 2026 an outgoing payment of **400.00** Euro is registered from the bank journal and reconciled against the bill. The payment's own entry is (see `../payments-and-bank-reconciliation/accounting-effects.md`):

| Account | Debit | Credit |
|---|---|---|
| 400000 Account Payable | 400.00 | — |
| 101401 Outstanding Payments | — | 400.00 |

A partial reconciliation of 400.00 now links the bill's term line (credit 1 128.00) to the payment's payable line (debit 400.00).

State of the bill:

```formula
amount_residual = 1 128.00 − 400.00 = 728.00
payment_state   = partial
```

*Step 2 — the supplier issues a credit note for the two desk lamps.* The lamps line was 160.00 plus 24.00 plus 24.00 = 208.00. The user selects the bill and asks for a reversal with the reversal date **5 February 2026** and the reason *Lamps returned*.

Because a **partial** reversal is wanted, the user takes the plain *Reverse* route (not *Reverse and cancel*), producing a draft vendor credit note that copies every line, and then deletes the chair line from it. (The cancelling route would reverse the whole 1 128.00 and reconcile it, which is not what happened commercially.)

*Step 3 — the credit note as posted.*

| Field | Value |
|---|---|
| type | `in_refund` |
| number | `RBILL/2026/02/0001` (the purchase journal has a separate refund sequence) |
| reversed entry | `BILL/2026/01/0003` |
| vendor reference | *Reversal of: BILL/2026/01/0003, Lamps returned* |
| bill date, accounting date, due date | 5 February 2026 |
| payment term | cleared |

| Line | Display type | Account | Debit | Credit | Amount in currency |
|---|---|---|---|---|---|
| 1 | `product` | 600100 Expenses (lamps) | — | 160.00 | −160.00 |
| 2 | `tax` (refund distribution of Purchase 15 %) | 131000 Tax Paid | — | 24.00 | −24.00 |
| 3 | `tax` (refund distribution of Purchase 15 % (b)) | 131000 Tax Paid | — | 24.00 | −24.00 |
| 4 | `payment_term` | 400000 Account Payable | 208.00 | — | +208.00 |

Totals: debit 208.00, credit 208.00. ✓ The credit note's own totals are `amount_untaxed` 160.00, `amount_tax` 48.00, `amount_total` 208.00, and its `amount_total_signed` is **+208.00** (the mirror of the bill's −1 128.00).

*Step 4 — reconciling the credit note against the bill.* The two payable term lines are on the same account 400000 and the same currency, so they may be reconciled. A second partial reconciliation of 208.00 is created.

State of the bill afterwards:

```formula
reconciled so far = 400.00 (payment) + 208.00 (credit note) = 608.00
amount_residual   = 1 128.00 − 608.00 = 520.00
payment_state     = partial
```

State of the credit note: residual 0, and the counterpart types seen from it are `{in_invoice}`, which is **not** one of the reversal patterns for a credit note (which require exactly `{entry}`), so its payment status is **`paid`**, not `reversed`.

*Step 5 — settling the rest.* A second outgoing payment of 520.00 on 28 February 2026 clears the bill:

| Account | Debit | Credit |
|---|---|---|
| 400000 Account Payable | 520.00 | — |
| 101401 Outstanding Payments | — | 520.00 |

Now the bill's residual is zero. Its counterparts include payments, so the status depends on whether every settling payment is matched with a bank statement: while at least one is not, the in-payment hook is consulted, which in the base package answers **`paid`**; an accounting package would answer `in_payment` until the bank statements are reconciled. It is **not** `reversed`, because payments are among the counterparts.

*Step 6 — the full-reversal variant.* Had the user instead chosen *Reverse and cancel* on the partly paid bill:

1. Every reconciliation of the bill's lines is first **removed** — the 400.00 partial disappears, the payment becomes unreconciled again and its own status falls back from `paid` to `in_process`.
2. A vendor credit note of the full 1 128.00 is created and posted on the reversal date.
3. The two payable term lines (bill 1 128.00 credit, credit note 1 128.00 debit) are reconciled, clearing both.
4. The bill's counterpart set is exactly `{in_refund}`, so its payment status becomes **`reversed`**.
5. The 400.00 payment is now floating: it must be re-reconciled against something else, or refunded.

This is why a partly paid bill should be reversed with the plain *Reverse* route and then reconciled by hand, unless the intent really is to unwind the payment as well.

---

## 6. Cancelling a bill

Cancelling produces **no new journal entry**. It:

1. resets the document to draft (with all the guards);
2. **removes every reconciliation** of every line — which changes the payment status of every counterpart document and payment;
3. cancels every payment whose entry this is;
4. sets the automatic posting mode to `no` and the status to `cancel`.

The journal items remain in the database but belong to a cancelled entry and are therefore excluded from the ledger and from every report.

---

## 7. Paying a bill by printed cheque

The cheque itself produces the ordinary outgoing payment entry, specified in `../payments-and-bank-reconciliation/accounting-effects.md`. The payable domain adds only this:

| Aspect | Effect |
|---|---|
| Journal | the **bank journal** the cheque is drawn on |
| Liquidity line | the journal's outstanding payments account (or its default account when no outstanding account is configured), **credit**, for the payment amount |
| Counterpart line | the vendor's payable account, **debit**, for the payment amount |
| Label of both lines | *Checks - «cheque number»*, extended with *: «memo»* when the payment carries a memo. Without a cheque number the generic payment label applies |
| Reconciliation | the counterpart line is reconciled against the bill's payable term line, wholly or partly |
| Cheque number | consumed from the journal's cheque sequence at posting when the journal uses manual numbering; written after the fact by the pre-numbered wizard otherwise |

**Voiding** a cheque (see `state-machines.md` §5.3) resets the payment to draft and cancels it: the draft entry is **deleted** if it was still draft, and otherwise cancelled. Either way the reconciliation against the bill is removed and the bill's payment status reverts.

**Worked example.** A cheque of 1 234.56 United States dollars drawn on the bank journal *First National*, whose outstanding payments account is **101402 Outstanding Payments** and whose cheque sequence stands at 000101.

At posting (manual numbering):

| Account | Label | Debit | Credit |
|---|---|---|---|
| 400000 Account Payable | Checks - 00101 | 1 234.56 | — |
| 101402 Outstanding Payments | Checks - 00101 | — | 1 234.56 |

and `check_amount_in_words` is set to *One Thousand, Two Hundred And Thirty-Four Dollars and Fifty-Six Cents*.

At bank reconciliation, the statement line moves the amount from the outstanding account to the bank account and the payment becomes `paid`.

---

## 8. Intercompany clearing

*(Intercompany Payment Clearing package.)* When a document of company **A** is paid by a payment transaction that belongs to company **B** of the same group, posting the document produces **two** additional journal entries so that neither company is left with a dangling balance.

### 8.1 When it fires

At the end of posting, a document qualifies when **all** of:

1. it is invoice-like;
2. its payment status is `not_paid`, `partial` or `in_payment`;
3. its company has an **intercompany clearing journal**;
4. it has at least one payment transaction carrying a payment;
5. the document's company is **not** the company of that payment;
6. the direction and the accounts are complete:
   - for an **inbound** document: the document's company has an intercompany **receivable** account and the payer's company has an intercompany **payable** account;
   - for an **outbound** document (a vendor bill): the document's company has an intercompany **payable** account and the payer's company has an intercompany **receivable** account;
7. at least one such payment is `in_process` or `paid`, belongs to another company, and that company has an intercompany clearing journal.

Only transactions in the `authorized` or `done` state are processed.

### 8.2 The payer's clearing entry (one per settling payment)

Created **in the payer's company**, in that company's intercompany clearing journal, with the payment's memo as reference.

Let the payment's counterpart lines be the lines of its own entry on the receivable or payable account, and

```formula
clearing_balance         = − Σ balance over those counterpart lines
clearing_amount_currency = − Σ amount_in_currency over those counterpart lines
```

| Line | Account | Partner | Currency | Amount in currency | Balance |
|---|---|---|---|---|---|
| 1 | the **payment's own counterpart account** | the document's partner | the payment's currency | `clearing_amount_currency` | `clearing_balance` |
| 2 | the other company's intercompany **payable** account when the document is a sale document, its intercompany **receivable** account otherwise | the **document's company partner** | the payment's currency | `− clearing_amount_currency` | `− clearing_balance` |

The entry is posted, and line 1 is reconciled against the payment's counterpart lines — clearing the payment in the payer's books.

*Label of both lines*: `«document partner display name» / «document number»`.

### 8.3 The biller's settlement entry (one per document)

Created **in the document's company**, in that company's intercompany clearing journal, with reference *Interco Settlement - «the comma-joined memos of the settling payments»*.

Let the document's receivable and payable lines be its term lines, and

```formula
total_invoice_balance        = Σ balance over those lines
clearing_counterpart_balance = − total_invoice_balance
```

| Line | Account | Partner | Currency | Balance |
|---|---|---|---|---|
| 1 | the document company's intercompany **receivable** account for a sale document, its intercompany **payable** account otherwise | the **payer's company partner** | the document company's currency | `− clearing_counterpart_balance` |
| 2 | the **document's own term-line account** (its payable or receivable account) | the payment's partner | the document company's currency | `clearing_counterpart_balance` |

The entry is posted, and line 2 is reconciled against the document's term lines — clearing the document in the biller's books.

*Label of both lines*: `«document partner display name» / «document number»`.

### 8.4 Net effect on a vendor bill

Company A owes its supplier; company B paid on A's behalf. After the two entries:

- in **B**'s books the payment's payable line is cleared and B carries a **receivable from A** on its intercompany receivable account;
- in **A**'s books the bill's payable line is cleared and A carries a **payable to B** on its intercompany payable account;
- the two intercompany balances mirror each other and net to zero on consolidation.

---

## 9. Debit notes

*(Debit Notes package.)* A debit note produces **no special entry**: it is an ordinary document of the same type as the source (a bill for a bill; a **bill** for a vendor credit note, since a debit note against a credit note is a charge), created by copying the source with these overrides:

| Field | Value |
|---|---|
| type | the source type, except that `in_refund` becomes `in_invoice` and `out_refund` becomes `out_invoice` |
| vendor reference | the source's number, or `«source number», «reason»` when a reason was typed |
| accounting date | the chosen date, or the source's accounting date |
| bill date | the same, for an invoice-like source |
| journal | the chosen journal, or the source's journal |
| payment term | **cleared** |
| debit origin | the source |
| lines | **removed** unless the *Copy Lines* switch was set; never copied when the source is a credit note |

When the journal has a **dedicated debit note sequence** and the resulting document is a bill or a customer invoice, its numbering series is prefixed with the letter `D` — that is, the starting sequence becomes `D` followed by the journal's normal starting sequence — and the "last number" search is restricted to documents that likewise do or do not have a debit origin, so the two series never interfere.

The chatter of the source records *This debit note was created from: «link»*.

Its posting produces the ordinary lines of §2.

---

## 10. The private-share entries in detail

Given a bill line of 100.00 with a 20 % purchase tax and 75 % deductibility (the worked example of `calculations.md` §10), the **posted** entry is:

| Display type | Account | Debit | Credit |
|---|---|---|---|
| `product` | 600000 Expenses | 100.00 | — |
| `non_deductible_product` | 600000 Expenses | — | 25.00 |
| `non_deductible_product_total` | 610000 Private Share | 25.00 | — |
| `tax` | 131000 Tax Paid | 15.00 | — |
| `non_deductible_tax` | 610000 Private Share | 5.00 | — |
| `payment_term` | 400000 Account Payable | — | 120.00 |

Totals: debit 145.00, credit 145.00. ✓

Reading it: the expense account keeps 75.00 net (100.00 debit less 25.00 credit); the private share account carries 25.00 of expense plus 5.00 of non-reclaimable tax; the tax authority is owed 15.00 rather than 20.00; and the supplier is still owed the full 120.00.

The labels of the two aggregate lines become *«document number» - private part* and *«document number» - private part (taxes)* at posting.

---

## 11. Effects on other domains

| Domain | Effect |
|---|---|
| `../general-ledger/` | The purchase document is a journal entry: it participates in numbering, in gap detection, in the hash chain, in lock dates, in the trial balance and in the general ledger |
| `../taxes/` | Every expense line feeds the tax engine and carries base reporting grids; every tax line carries tax reporting grids; cash-basis taxes defer their recognition to the reconciliation of the payable term line |
| `../payments-and-bank-reconciliation/` | The payable term lines are the reconciliation targets of outgoing payments; the payment register wizard reads the residual and the early discount data from them |
| `../analytic-accounting/` | Each expense line's analytic distribution creates analytic lines at posting and deletes them on reset to draft |
| `../multi-currency/` | Reconciling a foreign-currency payable line against a payment at a different rate creates an exchange difference entry; that entry can never be reset to draft |
| `../inventory-valuation-and-costing/` | On a bill line for a stockable product under perpetual valuation, the expense account is replaced by the stock input account and the difference to the standard price goes to the company's price difference account; specified in that domain |
| `../purchasing/` | Posting a bill updates the billed quantities of the matched purchase order lines through the matching hook |
| `../financial-reporting/` | Posted purchase documents feed the aged payable report, the partner ledger and the tax return |

---

## 12. Entries this domain does **not** produce

- It produces no entry when a document is **cancelled**, **reset to draft**, **uploaded**, **decoded**, **marked reviewed**, or **detected as a duplicate**.
- It produces no entry when a cheque is **printed**, **unmarked as sent** or **renumbered**; only posting and voiding move the ledger.
- The **analysis report** is a read-only view and produces nothing.
