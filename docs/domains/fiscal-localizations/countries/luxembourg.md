# Luxembourg

The Luxembourgish package supplies one chart of accounts template of 973 account groups built on the standard chart of accounts, twelve tax groups covering every rate the country has published, each wired to a payable account, a receivable account **and an advance tax payment account**, five fiscal positions, and a bank fee account resolved by an exact code, type and name match.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `lu` |
| Display name | Luxembourg |
| Parent template | none |
| Fiscal country | Luxembourg |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `513` |
| Cash account code prefix | `516` |
| Transfer account code prefix | `517` |
| Default point of sale receivable account | `40111` |
| Gain exchange rate account | `7561` |
| Loss exchange rate account | `6561` |
| Bank suspense account | `484` |
| Cash discount write-off loss account | `65562` |
| Cash discount write-off gain account | `75562` |
| Default sale tax | 17 percent output tax |
| Default purchase tax | 17 percent input tax |
| Income account | `703001` |
| Expense account | `6061` |
| Receivable account recorded as a Contact default | `4011` |
| Payable account recorded as a Contact default | `44111` |
| Stock valuation account recorded as a company property | `301` |

### 1.2 Journals

Both the sales journal and the purchase journal are created with separate credit note numbering switched on.

### 1.3 Reconciliation model

| Symbolic identifier | Name | Line |
|---|---|---|
| `cash_discount_template` | Cash Discount | one line, 100 percent, account `65562`, labelled "Cash Discount" |

### 1.4 Bank fee account override

The bank fee reconciliation model points at the account whose code is `613330`, whose type is Expense and whose name is exactly "Bank account charges and bank commissions (included custody fees on securities)". The match is deliberately exact so that a renamed or re-coded account is not silently used instead; when it is not found, the framework's generic search applies.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account groups shipped | 973, each with a starting and an ending prefix |
| Code lengths present | 2 to 6 characters |
| Translations shipped | French and German |

**Account type distribution.**

| Account type | Count |
|---|---|
| Bank and Cash | 5 |
| Current Asset | 87 |
| Fixed Asset | 65 |
| Prepayments | 1 |
| Receivable | 4 |
| Equity | 81 |
| Current Year Earnings | 1 |
| Expense | 260 |
| Income | 133 |
| Other Income | 26 |
| Current Liability | 73 |
| Non-current Liability and Payable | the remainder |

---

## 3. Tax groups

Twelve groups, one per rate the country has published, all sharing the same three accounts.

| Group | Payable account | Receivable account | Advance payment account |
|---|---|---|---|
| 0% value-added tax | `461412` | `421612` | `421613` |
| 3% value-added tax | `461412` | `421612` | `421613` |
| 6% value-added tax | `461412` | `421612` | `421613` |
| 7% value-added tax | `461412` | `421612` | `421613` |
| 8% value-added tax | `461412` | `421612` | `421613` |
| 10% value-added tax | `461412` | `421612` | `421613` |
| 12% value-added tax | `461412` | `421612` | `421613` |
| 13% value-added tax | `461412` | `421612` | `421613` |
| 14% value-added tax | `461412` | `421612` | `421613` |
| 15% value-added tax | `461412` | `421612` | `421613` |
| 16% value-added tax | `461412` | `421612` | `421613` |
| 17% value-added tax | `461412` | `421612` | `421613` |

The **advance payment account** is unusual: the closing entry clears whatever sits on it before deciding whether the period is payable or receivable, which is how the country's monthly advance payments are reconciled with the annual return.

**Worked example of a closing with an advance payment.** A period with 17,000.00 collected, 5,000.00 deductible and 8,000.00 already paid in advance.

| Line | Account | Debit | Credit |
|---|---|---|---|
| Clear collected tax | `461412` | 17,000.00 | |
| Clear deductible tax | `421612` | | 5,000.00 |
| Clear advance payment | `421613` | | 8,000.00 |
| Net due | `461412` | | 4,000.00 |
| **Totals** | | **17,000.00** | **17,000.00** |

---

## 4. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country or group |
|---|---|---|---|---|
| 10 | Not liable to value-added tax | no | no | none |
| 20 | Luxembourgish Taxable Person | yes | no | Luxembourg |
| 30 | Intra-Community Taxable Person | yes | yes | European Union group |
| 40 | European Union private | yes | no | European Union group |
| 50 | Extra-Community Taxable Person | yes | no | none |

---

## 5. Acceptance scenarios

**Given** a company in Luxembourg with no accounting,
**when** the Luxembourgish template is loaded,
**then** the bank prefix is `513`, the cash prefix is `516`, the transfer prefix is `517`, 973 account groups exist, twelve tax groups exist and each one names an advance payment account.

**Given** a period with 17,000.00 collected, 5,000.00 deductible and 8,000.00 on the advance payment account,
**when** the period is closed,
**then** the closing entry clears all three and posts a net payable of 4,000.00.

**Given** the account whose code is `613330` with its exact shipped name,
**when** the template is loaded,
**then** the bank fee reconciliation model points at it.
