# Accounting effects

## 1. The position of this domain

**This domain writes nothing to the ledger.** No operation of any of its sixteen capability packages creates, changes, posts, reverses or reconciles a Journal Entry or a Journal Item, and none of them creates a Journal, an Account, an Account Tag, a Tax, an Analytic Line or a Currency Rate.

The statement is not a summary; it is exhaustive, and it is derived as follows.

| Entity of the domain | Every operation it exposes | Ledger effect |
|---|---|---|
| Spreadsheet Document mixin | Validate a stored workbook; compute the workbook text from the stored file and write it back; compute the download file name; produce the empty workbook; resolve display names for a workbook; package workbook-file parts into one archive; read a file's content | none |
| Spreadsheet Dashboard | Toggle the favourite mark; build the reading payload; load the sample workbook; decide whether the dashboard is empty; read the translation namespace; duplicate | none |
| Dashboard Group | Guard deletion | none |
| Dashboard Share | Compute the shared address; create a share and answer with the address; compare a token; check access | none |
| Dashboard Board | Create, which stores nothing; read the layout; preprocess the layout | none |

The six operations this domain adds to Account are the reading operations of [`entities.md`](entities.md) §8.1. Four of them aggregate Journal Items — total debit and credit, residual amount, partner balance, tagged balance — one lists them in a window action, and one groups Accounts by type to collect their codes. Every one of them is declared as a read-only operation, and none of them writes.

The operation this domain adds to Company reads the fiscal-year end day and end month and answers two dates. The operations it adds to Currency and Currency Rate read a rate. None writes.

**Why this matters even so.** A dashboard is what a controller looks at before deciding that the ledger is right. A rebuild that produced the same journal entries but different dashboard numbers would be reporting on a ledger it was not describing, and the error would surface only when someone reconciled the dashboard against a statutory report. The obligations of §4 exist for that reason.

## 2. The ledger data the domain reads

| Datum | Read by | Where the reading rule is |
|---|---|---|
| Journal Item debit amount | The total-debit formula and the total-balance formula | [`calculations.md`](calculations.md) §7 |
| Journal Item credit amount | The total-credit formula and the total-balance formula | [`calculations.md`](calculations.md) §7 |
| Journal Item balance | The partner-balance formula and the tagged-balance formula | [`calculations.md`](calculations.md) §9, §11 |
| Journal Item residual amount | The residual formula | [`calculations.md`](calculations.md) §8 |
| Journal Item date | The period condition of every accounting formula | [`calculations.md`](calculations.md) §6 |
| Journal Item company | The company condition of every accounting formula | [`calculations.md`](calculations.md) §6 |
| Journal Item partner | The partner condition of the partner-balance formula and of the cell audit action | [`calculations.md`](calculations.md) §6 |
| Journal Entry state | The posted condition of every accounting formula | [`calculations.md`](calculations.md) §6 |
| Account code | The prefix condition of every code-based formula | [`calculations.md`](calculations.md) §6 |
| Account type | The payable-and-receivable fallback, and the account-group formula | [`calculations.md`](calculations.md) §6, §12.2 |
| Account initial-balance flag | The split between cumulative and period-confined selection | [`calculations.md`](calculations.md) §6 |
| Account tags | The tagged-balance formula | [`calculations.md`](calculations.md) §11 |
| Company fiscal-year last day and last month | The fiscal-year formulas and every yearly or daily period | [`calculations.md`](calculations.md) §5, §12.1 |
| Currency code, symbol, decimal places, symbol position | The currency number format | [`calculations.md`](calculations.md) §13.3 |
| Currency rate on a date for a company | The currency-rate formula | [`calculations.md`](calculations.md) §13.1 |

Every one of these is owned elsewhere: the first eleven by [`../general-ledger/`](../general-ledger/), the twelfth by [`../contacts-and-organizations/`](../contacts-and-organizations/), the last two by [`../multi-currency/`](../multi-currency/). This folder specifies how they are *selected and combined*, never how they come to hold the values they hold.

## 3. Reading rules that a rebuild must not soften

Four properties of the reading rule change the numbers a controller sees. They are restated here because they are accounting decisions, not presentation decisions.

### 3.1 Balance-sheet accounts are cumulative, profit-and-loss accounts are not

The period condition of [`calculations.md`](calculations.md) §6 is a disjunction, not a conjunction of one window. An Account that includes the initial balance contributes every Journal Item dated on or before the window's last date, whatever the window's first date is. An Account that does not contributes only the items inside the window.

The consequence to hold on to: a "balance for 2022" on a receivable account is a **position at 31 December 2022**, and a "balance for 2022" on a revenue account is a **flow over 2022**. The same formula produces both, and which one it produces is decided by the Account, not by the formula.

### 3.2 Cancelled entries are never counted

Asking for unposted entries widens the selection to every entry state except `cancel`; it never widens it to every state. There is no argument, anywhere in the domain, that makes a cancelled entry contribute to a figure.

### 3.3 The company is a hard filter, not a context

Every accounting formula names exactly one company — the argument's, or the acting one — and the selection carries a condition on the Journal Item's own company. A workbook read by a user with three active companies does not aggregate across them unless three formulas, one per company, are written. This is the opposite of the way the record rules of [`configuration.md`](configuration.md) §5 treat the dashboards themselves, which are visible across the active companies; the two must not be conflated.

### 3.4 The fiscal year, not the calendar year

A year in a period argument is a fiscal year. For a company whose fiscal year ends on the thirty-first of December the two coincide; for any other company they do not, and the mapping of [`calculations.md`](calculations.md) §5.3 applies. A rebuild that read a year as a calendar year would produce figures that differ from the statutory reports of the same installation by the length of the shift.

## 4. Correctness obligations

| Obligation | Statement |
|---|---|
| Agreement with the general ledger | For any window and any account selection, the total debit and total credit that this domain reports must equal the totals that the general ledger reports for the same window, the same accounts, the same company and the same posted-entry choice. There is no rounding step in this domain that could make the two differ |
| Agreement with the aged balances | The residual amount this domain reports for the payable and receivable accounts of a company at a date must equal the total outstanding that the receivables and payables reporting shows at the same date. See [`../financial-reporting/`](../financial-reporting/) |
| Agreement with partner statements | The partner balance this domain reports for one partner must equal the partner's balance on the same accounts over the same window. See [`../accounts-receivable/`](../accounts-receivable/) and [`../accounts-payable/`](../accounts-payable/) |
| No conversion | The domain never converts a ledger amount between currencies. A workbook that mixes companies with different currencies in one figure is adding unlike quantities, and the domain does not prevent it. A rebuild must reproduce the absence of the conversion, because introducing one would change every such figure |
| No rounding | The domain inserts no rounding step between the stored amounts and the cell. A cell showing two decimal places is showing a rounded *presentation* of an exact sum; a second cell that adds ten such cells adds the exact sums, not the presented ones |

## 5. Ledger effects triggered indirectly

The domain triggers none. Reading a dashboard, setting a filter, drilling through to a record, sharing a dashboard, downloading a workbook file and pinning a view onto the personal board all leave the ledger untouched.

Two paths lead a reader *from* this domain to a place where a ledger effect can be produced, but the effect is produced there and is specified there:

| Path | Where it leads | Where the effect is specified |
|---|---|---|
| The cell audit action of [`workflows.md`](workflows.md) §6.5 opens a list of Journal Items | A reader with the right may open one and act on its entry | [`../general-ledger/`](../general-ledger/) |
| A drill-through from a list, a pivot or a chart opens the records the element measures | A reader with the right may confirm an order, validate an invoice, post an entry from there | [`../sales/`](../sales/), [`../accounts-receivable/`](../accounts-receivable/), [`../general-ledger/`](../general-ledger/), [`../inventory-operations/`](../inventory-operations/), [`../point-of-sale/`](../point-of-sale/), [`../expenses/`](../expenses/), [`../timesheets/`](../timesheets/) according to the entity |

In both cases the opening is a window action and nothing more: no record is created, changed or posted by the act of opening it.

## 6. Analytic accounting

The domain produces no analytic distribution and reads none. An accounting formula's selection carries no analytic condition, and the cell audit action's window action carries none either. A workbook that wants analytic figures uses a data-bound pivot or list over the analytic entity, which is an ordinary data-bound element with no special treatment; the entity and its fields belong to [`../analytic-accounting/`](../analytic-accounting/).

## 7. Taxes

The domain applies no tax treatment. An accounting formula sums stored amounts, and a stored amount is already whatever the tax computation of [`../taxes/`](../taxes/) made it. A workbook that reports tax figures does so by selecting the tax accounts by code, or by tag, exactly as it selects any other account; the domain has no notion of a tax line.

## 8. Reconciliation

The domain reconciles nothing and unreconciles nothing. It reads one consequence of reconciliation — the residual amount of a Journal Item — and that reading is the whole of its involvement. The reconciliation itself is specified in [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/).

The consequence worth stating: a residual figure is a **current** property. A residual asked for a window that ends in the past reflects every reconciliation performed since, including reconciliations dated after the window's last date. The worked example of [`calculations.md`](calculations.md) §8.2 shows the case: a receivable item dated 2 February 2022 reports a residual of 1350.00 for a window ending on 2 February 2022, because a payment dated 10 February 2022 has already been reconciled against it. A rebuild that computed a residual "as at" the window's last date would produce 1500.00 instead and would disagree with this domain on every reconciled document.

**Compatibility finding.** The behaviour above is the observed one and is reproduced. A corrected behaviour would offer both readings — the current residual and the residual as at the window's last date — under two distinct formulas, so that an author can state which of the two they mean. The single existing formula answers the current residual.
