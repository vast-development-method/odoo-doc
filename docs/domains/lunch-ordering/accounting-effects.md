# Meal Ordering — Accounting effects

## 1. The domain produces no journal entries

No operation of this domain writes to the ledger. No entity of this domain carries a journal, an
account, a tax, an analytic distribution or a reconciliation marker. Confirming a cart, dispatching
a vendor's day, marking a delivery received, cancelling a line, archiving a line and recording an
account movement all leave the general ledger untouched.

This is a deliberate design, not an omission, and the reasoning is worth stating because a rebuild
will be tempted to post something.

1. **The money never belongs to the employer in this domain.** An employee hands cash to an
   administrator so that the administrator can pay a vendor on their behalf. Both the receipt and
   the payment are the employer's, but the intent of the arrangement is that they cancel out. The
   domain records only the part that has no accounting counterpart on its own: how much of the
   employee's money is still unspent.
2. **The charge is not a sale and not an expense.** The employer neither sells the meal to the
   employee nor consumes it. There is no revenue to recognise, no cost of sales to match, no tax to
   account for and no receivable to age.
3. **The amounts are too small and too numerous to post.** A working population of two hundred
   people generates roughly forty thousand meal charges a year. Posting each one would swamp the
   ledger with entries that net to nothing.
4. **The entities have no accounting fields at all.** The account movement has an employee, a date,
   an amount, a currency and a description, and nothing else. There is no journal reference, no
   account reference and no posting state. A rebuild that wanted to post would have to add every one
   of those.

The consequence for a rebuild: **implement the internal account exactly as specified in
[`calculations.md`](calculations.md#3-the-internal-account-balance) and post nothing.** A rebuild
that posts journal entries from this domain will not be compatible, because reports, reconciliations
and trial balances will differ from the system being replaced.

---

## 2. What the domain keeps instead: the internal account

The domain keeps a private, single-entry running total per employee. It is not a ledger:

| Property of a ledger | Present here? |
|---|---|
| Double entry | No. Each row is a single signed amount with no counterpart. |
| Accounts | No. There is one implicit account per employee and nothing else. |
| Journals | No. |
| Posting state | No. A movement is effective the instant it is created. |
| Period closing | No. Rows are never locked, never closed and never carried forward. |
| Reconciliation | No. |
| Currency conversion | No. Amounts of different currencies are added as if they were the same — see rule [MEAL-051](business-rules.md#meal-051--amounts-of-different-currencies-are-summed-without-conversion). |
| Audit trail on change | Only the platform's ordinary creation and write stamps on the movement record. Deleting a movement removes the row from the statement with no trace in the statement itself. |

The two row kinds and their signs are specified in
[`entities.md`](entities.md#82-nature-and-composition). The balance formula, its rounding and the
permitted overdraft are specified in
[`calculations.md`](calculations.md#3-the-internal-account-balance).

---

## 3. The ledger effects this domain triggers elsewhere

Four real financial events surround the arrangement. None of them is produced by this domain; each
is produced by another domain, from a document that a person creates there. They are listed here so
that a rebuild can see the whole picture and can wire the domains together correctly.

### 3.1 The vendor's invoice

**Event.** The vendor invoices the employer for the meals delivered over a period.

**Produced by.** [Accounts Payable](../accounts-payable/), from a vendor bill whose partner is the
same Contact the meal vendor stands for.

| Item | Journal | Account selection | Side | Amount | Currency and rate | Date | Counterparty | Analytic | Tax | Reconciled against |
|---|---|---|---|---|---|---|---|---|---|---|
| Expense or suspense item | The purchase journal of the company | The expense account of the bill line's product, or the account chosen manually; employers commonly use a staff-catering expense account or a suspense account that the employee repayments clear | Debit | The bill line's amount excluding tax | The bill's currency at the bill's rate | The bill date | The vendor's Contact | The analytic distribution of the bill line, when one is set | The purchase tax on the bill line, when any | Nothing |
| Tax item | The same journal | The tax account of the applied tax | Debit | The tax amount computed by the tax engine | As above | As above | The vendor's Contact | As above | The tax itself | The tax report |
| Payable item | The same journal | The payable account of the vendor's Contact | Credit | The bill total including tax | As above | As above | The vendor's Contact | None | None | The outgoing payment that settles the bill |

The link back to this domain is informational only: the meal vendor record names the same Contact,
so an accountant can see which vendor a bill belongs to. No order line, no account movement and no
statement row is touched by the bill, and no bill line is created from an order.

### 3.2 The employee's payment to the employer

**Event.** An employee hands cash or makes a transfer so that their internal account can be
credited.

**Produced by.** [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/) when the
employer chooses to record it at all, from a bank or cash statement line or from a miscellaneous
entry.

| Item | Journal | Account selection | Side | Amount | Currency and rate | Date | Counterparty | Analytic | Tax | Reconciled against |
|---|---|---|---|---|---|---|---|---|---|---|
| Cash or bank item | The cash or bank journal the money arrived in | The outstanding-receipts or liquidity account of that journal | Debit | The amount handed over | The journal's currency at the statement rate | The date the money arrived | The employee's Contact, when the employer records one | None | None | The bank statement line |
| Counterpart item | The same journal | The same expense or suspense account used on the vendor bill, or a staff-advance account when the employer tracks these per employee | Credit | The same amount | As above | As above | The employee's Contact | As above | None | The vendor bill's expense item, when a suspense account is used |

**Compatibility note for a rebuild.** There is no automatic link. Creating a Lunch Cash Move does
not create a payment, and recording a payment does not create a Lunch Cash Move. The administrator
records both, or only the account movement, according to how the employer chooses to run the
arrangement.

### 3.3 The employer's subsidy

**Event.** The employer pays part of the meal cost, either by crediting accounts without collecting
money or by setting a permitted overdraft that is never recovered.

**Produced by.** [General Ledger](../general-ledger/), from a miscellaneous entry the accountant
writes at the period end, sized from this domain's own figures.

| Item | Journal | Account selection | Side | Amount | Currency and rate | Date | Counterparty | Analytic | Tax | Reconciled against |
|---|---|---|---|---|---|---|---|---|---|---|
| Staff benefit expense | The miscellaneous journal | The staff-benefit expense account chosen by the accountant | Debit | The subsidised part, read from the statement as the credits granted without a matching receipt | The company currency | The last day of the period | Usually none | The cost centre of the workforce, when the employer allocates staff benefits | Usually none; some jurisdictions treat a meal subsidy as a taxable benefit in kind, which is handled in [Taxes](../taxes/) | The suspense account balance |
| Suspense clearing | The same journal | The same suspense account the vendor bills were charged to | Credit | The same amount | As above | As above | None | As above | None | The vendor bills' expense items |

### 3.4 Payroll recovery

**Event.** The employer recovers what employees owe through the payroll instead of collecting cash.

**Produced by.** [Work Entries](../work-entries/) and the payroll capability it feeds, from a
deduction line on the payslip, sized from this domain's statement.

The deduction reaches the ledger as an ordinary payslip item: a debit to the net-payable account and
a credit to the same suspense or staff-advance account used above. This domain supplies only the
figure; it neither creates the deduction nor learns that it happened, so an administrator who
recovers through payroll must also record a compensating Lunch Cash Move for each employee, or the
internal balances will never return to zero.

---

## 4. Reporting the arrangement

Because nothing is posted, the only report of the arrangement is the domain's own statement:

- **Per employee.** The personal statement screen, filtered to the reading employee, with a summed
  total column. It answers "what do I still have".
- **Across employees.** The administrator's control screen, grouped by employee, with a summed total
  per group. It answers "who owes us what", and its grand total is the net position of the whole
  arrangement: positive when employees have paid in advance more than they have consumed, negative
  when they have consumed more than they have paid.

An accountant who needs the figure for the entries of sections 3.2 to 3.4 reads it from the grand
total of that screen at the period end.

**Industry-standard default.** The system offers no period cut-off on the statement, so the grand
total is always the position as of now, not as of a date. A rebuild that needs a dated position
should add a date filter on the statement rather than a posting mechanism; filtering by date on the
existing rows reproduces the position at that date exactly, because no row is ever back-dated
automatically and no row is ever revalued.

---

## 5. Summary for a rebuild

| Question | Answer |
|---|---|
| Does confirming a cart post anything? | No. |
| Does dispatching or receiving post anything? | No. |
| Does recording an account movement post anything? | No. |
| Does the domain hold any account, journal or tax reference? | No. |
| Where does the vendor's cost enter the books? | On a vendor bill in [Accounts Payable](../accounts-payable/), created by hand. |
| Where does the employee's money enter the books? | On a bank or cash statement line in [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/), created by hand, when the employer records it at all. |
| Where does the employer's subsidy enter the books? | On a miscellaneous entry in [General Ledger](../general-ledger/), sized from this domain's statement. |
| What must a rebuild reproduce exactly? | The statement rows, their signs, their inclusion rules and the balance arithmetic. Nothing else has an accounting consequence. |
