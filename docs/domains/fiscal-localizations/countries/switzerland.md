# Switzerland

The Swiss package supplies one chart of accounts template of 209 accounts built on the small and medium enterprise plan, 29 taxes covering the historical and current rate ladders, the investment and other expenses split of every input tax, the customs import tax computed as a division tax and the reverse charge on services bought abroad, eight tax groups sharing one payable and one receivable account, two fiscal positions, the Germany-style printed document layout, and the payment slip mechanism: a structured payment reference model, a dedicated account number format for payment slips, a validity check on every invoice and a wizard that handles a batch print in which some invoices cannot carry a payment slip.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `ch` |
| Display name | Switzerland |
| Parent template | none |
| Fiscal country | Switzerland |
| Account code length | 4 characters |
| Storno accounting | offered as an option, defaulting to off |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `102` |
| Cash account code prefix | `100` |
| Transfer account code prefix | `1090` |
| Default point of sale receivable account | `1101` |
| Gain exchange rate account | `3806` |
| Loss exchange rate account | `4906` |
| Cash discount write-off loss account | `4900` |
| Cash discount write-off gain account | `3800` |
| Cash difference expense account | `4991` |
| Cash difference income account | `4992` |
| Deferred expense account | `1300` |
| Deferred revenue account | `2301` |
| Default sale tax | the 8.1 percent output tax |
| Default purchase tax | the 8.1 percent input tax on goods and services |
| Printed document layout | the Germany-style layout |
| Paper format | the Germany-style paper format |
| Income account | `3200` |
| Expense account | `4200` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `1210` |
| Receivable account recorded as a Contact default | `1100` |
| Payable account recorded as a Contact default | `2000` |

The stock valuation account `1210` names its stock expense account (`4000`) and its stock variation account (`4801`).

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 209 |
| Code length | 4 characters |
| Account groups shipped | none |
| Translations shipped | German, French and Italian |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 66 |
| Receivable | 3 |
| Equity | 8 |
| Expense | 78 |
| Income | 22 |
| Other Income | 1 |
| Current Liability | 29 |
| Payable | 2 |

Key tax accounts: `2201` value-added tax payable and `1176` value-added tax receivable.

---

## 3. Taxes

29 taxes. Two rate ladders coexist: the historical one (2.5, 3.7 and 7.7 percent), shipped archived, and the current one (2.6, 3.8 and 8.1 percent), shipped active.

### 3.1 The three current rates

| Rate | Meaning | Sale tax | Purchase tax on goods and services | Purchase tax on investments and other expenses |
|---|---|---|---|---|
| 8.1 percent | Standard | 8.1% | 8.1% | 8.1% I OE |
| 3.8 percent | Accommodation | 3.8% | 3.8% | 3.8% I OE |
| 2.6 percent | Reduced | 2.6% | 2.6% | 2.6% I OE |

The **investments and other expenses** split exists because the return separates the input tax on current purchases from the input tax on capital goods and overheads, and reports them in different boxes.

### 3.2 Import tax

| Name | Printed label | Computation | Rate | Scope |
|---|---|---|---|---|
| 100% GS | 100% import | division | 100 | purchase |
| 100% I OE | 100% import, investments | division | 100 | purchase |

A customs bill states the tax amount, not the base. A **division** computation treats the typed amount as already containing the whole tax, so entering the customs tax amount produces a tax line of that amount and a base of zero, which is exactly what the return needs.

**Worked example.** A customs document charging 812.00 of import tax. The operator enters 812.00 on a line carrying the 100 percent division tax. The tax computed is `812.00 × 100 ÷ 100 = 812.00` and the taxable base is `812.00 − 812.00 = 0.00`, so the input tax box receives 812.00 and no revenue or expense is recognised on the line beyond the customs charge itself.

### 3.3 Reverse charge on services bought abroad

| Name | Printed label | Computation | Scope |
|---|---|---|---|
| 8.1% R C | 8.1% reverse | group | purchase |
| 7.7% R C | 7.7% reverse | group | purchase |

Each is a group tax whose children charge the tax and deduct it, so the amount due to the foreign supplier is unchanged while both boxes of the return receive the amount.

### 3.4 Return-only taxes

| Name | Printed label | Rate | Scope | Purpose |
|---|---|---|---|---|
| 8.1% R | 8.1% return | −8.1 | none | Reverses an output tax on a return of goods. |
| 7.7% R | 7.7% return | −7.7 | none | The same under the historical rate. |

### 3.5 Zero-rate and out-of-scope taxes

| Name | Printed label | Meaning |
|---|---|---|
| 0% EX (export) | 0% | An export sale. |
| 0% EXC | 0% excl. | A supply excluded from the tax. |
| 0% EX (import) | 0% import | A purchase whose tax is settled at customs. |
| 0% S T | 0% subventions | Subsidies and tourist taxes, reported but not taxed. |
| 0% D | 0% dons | Donations, dividends and compensation, reported but not taxed. |

---

## 4. Tax groups

| Group | Payable account | Receivable account |
|---|---|---|
| Value-added tax 0% | `2201` | `1176` |
| Value-added tax 2.5% | `2201` | `1176` |
| Value-added tax 3.7% | `2201` | `1176` |
| Value-added tax 7.7% | `2201` | `1176` |
| Value-added tax 100% | `2201` | `1176` |
| Value-added tax 2.6% | `2201` | `1176` |
| Value-added tax 3.8% | `2201` | `1176` |
| Value-added tax 8.1% | `2201` | `1176` |

---

## 5. Fiscal positions

| Name | Detected automatically | Country or group |
|---|---|---|
| Domestic | yes | Switzerland |
| Import and Export | yes | none |

---

## 6. Payment slip

### 6.1 Account number

| Entity | Field | Meaning |
|---|---|---|
| Bank Account | Payment slip account number | A dedicated form of the international account number reserved for payment slips. When the company holds one, it is used on the slip while the ordinary number stays on the account record. |
| Bank Account | Show payment slip options | Derived; reveals the field only for a Swiss company. |

### 6.2 Structured payment reference

The journal's payment reference model is extended with a Swiss form: a 27-character reference rendered in groups, for example `12 34560 00103 88500 1000 19188`, whose last character is a check computed over the preceding ones.

### 6.3 Validity check

| Field | Location | Meaning |
|---|---|---|
| Payment slip is valid | Journal Entry, derived | Whether the invoice can be printed with a payment slip: it must be a customer invoice, the company must hold a payment slip account number, the customer address must be complete and the currency must be one the slip supports. |
| Payment reference warning | Payment, derived | Warns when a payment's reference does not match the structured form the supplier expects. |

**Guard.**

```
Only customers invoices can be QR-printed.
```

### 6.4 Batch print wizard

When a batch print contains invoices that cannot carry a payment slip, a wizard reports the split.

| Field | Meaning |
|---|---|
| Number of payment slip invoices | How many will print with a slip. |
| Number of classic invoices | How many will print without one. |
| Payment slip text and classic text | The explanatory texts shown. |

**Buttons.** "Print all" prints both sets; a second button opens the list of the invoices that could not carry a slip.

**Guards.**

```
No invoice was found to be printed.
All selected invoices must belong to the same Switzerland company
```

---

## 7. Acceptance scenarios

**Given** a company in Switzerland with no accounting,
**when** the Swiss template is loaded,
**then** 209 accounts with four-character codes exist, the bank prefix is `102`, the cash prefix is `100`, the transfer prefix is `1090`, eight tax groups exist all pointing at `2201` and `1176`, the default sales tax is 8.1 percent and the report layout is the Germany-style layout.

**Given** a customs document charging 812.00 of import tax,
**when** the amount is entered on a line carrying the 100 percent division tax,
**then** the tax line is 812.00 and the taxable base is 0.00.

**Given** a purchase of capital goods at 8.1 percent,
**when** the investments and other expenses variant is chosen,
**then** the input tax is reported in the capital goods box rather than in the current purchases box.

**Given** a batch print of ten invoices of which three lack a complete customer address,
**when** the print is launched,
**then** the wizard reports seven payment slip invoices and three classic invoices and offers to open the list of the three.
