# Germany

The German package supplies two chart of accounts templates, one for each of the two standard charts in use, with 1,274 and 1,192 accounts respectively, six tax groups each wired to a payable account, a receivable account and an advance tax payment account, six fiscal positions that separate a counterpart with a registration number from one without and goods from services, an account code that becomes immutable once it carries journal items, a four-character classification code on every account for the audit export, the tax number and the business identification number on the company, the Germany-style printed document layout, and a forced restrictive audit trail.

---

## 1. Templates

| Template code | Display name | Parent | Account code length | Accounts shipped |
|---|---|---|---|---|
| `de_skr03` | German Chart of Accounts, process-based plan | none | 4 | 1,274 |
| `de_skr04` | German Chart of Accounts, balance-sheet-based plan | none | 4 | 1,192 |

The two plans differ in how they order the accounts: one groups them by the flow of a business process, the other by the structure of the balance sheet. They are alternatives, not a chain.

### 1.1 Company-level values, process-based plan

| Company field | Value |
|---|---|
| Fiscal country | Germany |
| Bank account code prefix | `120` |
| Cash account code prefix | `100` |
| Transfer account code prefix | `1360` |
| Default point of sale receivable account | `1411` |
| Gain exchange rate account | `2660` |
| Loss exchange rate account | `2150` |
| Cash discount write-off loss account | `8736` |
| Cash discount write-off gain account | `3730` |
| Default sale tax | the 19 percent output tax |
| Default purchase tax | the 19 percent input tax |
| Income account | `8400` |
| Expense account | `3400` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `7200` |
| Receivable account recorded as a Contact default | `1410` |
| Payable account recorded as a Contact default | `1610` |
| Stock valuation account recorded as a company property | `3960` |

The balance-sheet-based plan writes the same structure against its own account codes.

### 1.2 Reconciliation models

Each template adds discount reconciliation models, one per rate and per side, for example a model named "Discount on purchases at 7 percent" pointing at the matching discount account, so that a bank statement line that is short by a discount can be reconciled in one click.

---

## 2. Chart of accounts

**Process-based plan account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 134 |
| Fixed Asset | 68 |
| Non-current Asset | 75 |
| Prepayments | 3 |
| Receivable | 13 |
| Equity | 24 |
| Expense | 424 |
| Depreciation | 37 |
| Income | 98 |
| Other Income | 171 |
| Current Liability | 163 |
| Non-current Liability and Payable | the remainder |

**Balance-sheet-based plan account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 146 |
| Fixed Asset | 63 |
| Non-current Asset | 68 |
| Receivable | 18 |
| Equity | 28 |
| Expense | 376 |
| Depreciation | 28 |
| Income | 181 |
| Other Income | 61 |
| Current Liability | 160 |
| Non-current Liability and Payable | the remainder |

Many accounts ship with report tags and with a default tax already attached.

### 2.1 Immutable account codes

An account of a German company may not have its code changed once it carries journal items:

```
You can not change the code of an account.
```

The rule fires when the fiscal country is Germany, the account belongs to the active company and at least one journal item references it. It exists because the audit export identifies an account by its code, so a changed code breaks the correspondence between an already filed period and the chart.

### 2.2 Classification code

Every account carries a four-character **classification code**, tracked in the discussion thread, which is the code the audit export writes for that account. It is separate from the account code and may be changed without breaking the rule above.

---

## 3. Taxes

The tax set covers the two current rates and the two temporarily reduced rates that were in force during a rate cut, plus the agricultural flat rates and an open rate.

| Rate | Meaning |
|---|---|
| 19 percent | Standard. |
| 7 percent | Reduced. |
| 10.7 percent | Agricultural flat rate. |
| 5.5 percent | Agricultural flat rate, reduced. |
| 0 percent | Exempt, intra-union and export. |
| Open rate | A tax whose rate the operator types, used for a rate not yet shipped. |

Each rate exists as an output tax and an input tax, and the input taxes are further split between goods and services, because the return separates them.

---

## 4. Tax groups

Six groups per template, all wired to the same three accounts of that template.

| Group | Payable account (process plan) | Receivable account (process plan) | Advance payment account (process plan) |
|---|---|---|---|
| Value-added tax 0% | `1797` | `1545` | `1780` |
| Value-added tax 7% | `1797` | `1545` | `1780` |
| Value-added tax 5.5% | `1797` | `1545` | `1780` |
| Value-added tax 10.7% | `1797` | `1545` | `1780` |
| Value-added tax open rate | `1797` | `1545` | `1780` |
| Value-added tax 19% | `1797` | `1545` | `1780` |

For the balance-sheet-based plan the three accounts are `3860`, `1421` and `3820`.

The advance payment account holds the monthly advance payments; the closing entry clears it before deciding whether the period is payable or receivable.

---

## 5. Fiscal positions

Six per template, each with account mappings.

| Sequence | Name | Detected automatically | Requires a tax number | Country or group |
|---|---|---|---|---|
| 10 | Domestic business partner | yes | no | Germany |
| 20 | Business partner in the European Union, with a registration number | yes | yes | European Union group |
| 30 | Service provider in the European Union, with a registration number | no | yes | none |
| 40 | Business partner in the European Union, without a registration number | no | no | none |
| 50 | Business partner abroad, outside the European Union | no | no | none |
| 60 | Service provider abroad, outside the European Union | no | no | none |

The split between a goods partner and a service partner exists because the place of supply, and therefore the tax treatment, differs between the two.

---

## 6. Company settings

| Field | Type | Tracked | Meaning |
|---|---|---|---|
| Tax number | text | yes | The number the tax office assigns, whose pattern encodes the office and the state. |
| Business identification number | text | yes | The national business identification number. |
| Restrictive audit trail | boolean, forced on | yes | Cannot be switched off: `Can't disable restricted audit trail: forced by localization.` |

**Validations.**

```
You cannot change the fiscal country.
Your company's SteuerNummer is not compatible with your state
Your company's SteuerNummer is not valid
```

The fiscal country of a German company is frozen because the whole audit trail, the account code immutability and the export format depend on it.

---

## 7. Printed documents

The company's report layout and paper format are the Germany-style layout: a fixed geometry with an address window positioned for a window envelope, a reference block carrying the document number, the document date, the customer number and the contact, an optional line position column, and a totals block. The layout package also adds derived fields to the document layout editor so that the preview shows a realistic invoice date, due date and delivery date.

| Setting | Location | Default | Effect |
|---|---|---|---|
| Show the position column in reports | Company | false | Prints a line number column on every document. |

The same layout is adopted by Austria, Switzerland and the packages for expenses, purchases, repairs, sales and transfers, each of which adds its own document to the layout.

---

## 8. Acceptance scenarios

**Given** a company in Germany with no accounting,
**when** the process-based template is loaded,
**then** 1,274 accounts with four-character codes exist, the bank prefix is `120`, the cash prefix is `100`, the transfer prefix is `1360`, six tax groups exist each naming an advance payment account, and the report layout is the Germany-style layout.

**Given** an account that carries journal items,
**when** its code is changed,
**then** the change is refused with `You can not change the code of an account.`

**Given** a German company,
**when** the restrictive audit trail is switched off,
**then** the change is refused with `Can't disable restricted audit trail: forced by localization.`

**Given** a counterpart in another member state that has a registration number,
**when** an invoice is prepared,
**then** the fiscal position "Business partner in the European Union, with a registration number" is detected automatically.

**Given** a period with advance payments sitting on the advance payment account,
**when** the period is closed,
**then** the closing entry clears that account before deciding the direction of the net amount.
