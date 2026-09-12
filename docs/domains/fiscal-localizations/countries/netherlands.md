# Netherlands

The Dutch package supplies one chart of accounts template built on the national reference classification, 31 taxes covering the three rates and the reverse-charge, intra-union and distance-selling regimes, three tax groups sharing a single tax liability account, six fiscal positions including a distance-selling position that is excluded from the domestic return, and a pair of rounding difference accounts used when a document total is rounded to whole units.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `nl` |
| Display name | Netherlands |
| Parent template | none |
| Fiscal country | Netherlands |
| Account code length | 6 characters |
| Cost accounting flag | true |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `103` |
| Cash account code prefix | `101` |
| Transfer account code prefix | `1060` |
| Default point of sale receivable account | the point of sale receivable account |
| Gain exchange rate account | `8920` |
| Loss exchange rate account | `4920` |
| Cash discount write-off loss account | `7065` |
| Cash discount write-off gain account | `8065` |
| Rounding difference loss account | `4960` |
| Rounding difference profit account | `4950` |
| Default sale tax | 21 percent sales tax |
| Default purchase tax | 21 percent purchase tax |
| Income account | `8001` |
| Expense account | `7001` |
| Deferred expense account | `1205` |
| Deferred revenue account | `1405` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `3001` |
| Receivable account recorded as a Contact default | the trade receivable account |
| Payable account recorded as a Contact default | the trade payable account |
| Stock valuation account recorded as a company property | `3200` |

The stock valuation account `3001` names its stock expense account (`7000`) and its stock variation account (`7090`).

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | the national reference classification, with codes of 4 and 5 characters padded to 6 |
| Account groups shipped | none |
| Translations shipped | Dutch and German |

**Account type distribution.**

| Account type | Count |
|---|---|
| Bank and Cash | 1 |
| Current Asset | 15 |
| Fixed Asset | 48 |
| Prepayments | 1 |
| Receivable | 2 |
| Equity | 6 |
| Current Year Earnings | 1 |
| Expense | 118 |
| Depreciation | 5 |
| Cost of Revenue | 29 |
| Income | 24 |
| Other Income and the liability types | the remainder |

---

## 3. Taxes

31 taxes. The names follow the boxes of the periodic return: the sales boxes `1a`, `1b`, `1e`, `2a`, `3a`, `3b`, `3c`, `4a`, `4b` and the input box `5b`.

| Family | Rates | Scope |
|---|---|---|
| Domestic sales | 21, 9, 0 percent | sale |
| Domestic purchases | 21, 9, 0 percent | purchase |
| Reverse charge, domestic | 21, 9 percent | sale and purchase |
| Intra-union supply of goods | 0 percent | sale |
| Intra-union acquisition of goods | 21, 9 percent | purchase |
| Intra-union service supplied | 0 percent | sale |
| Intra-union service received | 21, 9 percent | purchase |
| Export outside the union | 0 percent | sale |
| Import from outside the union | 21, 9 percent | purchase |
| Distance selling | 21, 9 percent | sale |

A reverse-charge tax charges the tax and immediately deducts it, so the base is reported in both the output box and the input box while the amount due is unchanged.

**Worked example.** An intra-union acquisition of 1,000.00 at 21 percent.

```
tax charged in box 4b   =  210.00
tax deducted in box 5b  = −210.00
net amount due          =    0.00
base reported in box 4b = 1,000.00
```

---

## 4. Tax groups

| Group | Payable account | Receivable account |
|---|---|---|
| Value-added tax 0% | value-added tax liabilities | value-added tax liabilities |
| Value-added tax 9% | value-added tax liabilities | value-added tax liabilities |
| Value-added tax 21% | value-added tax liabilities | value-added tax liabilities |

All three point at the same liability account, which is the account the periodic return settles.

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country or group |
|---|---|---|---|---|
| 10 | Netherlands Domestic | yes | no | the mainland Netherlands country group |
| 30 | European Union, intra-community business to business | yes | yes | European Union group |
| 40 | European Union, business to consumer | yes | no | European Union group |
| 50 | European Union, non-intra | yes | no | none |
| (no ordering) | Value-added tax reverse charge | no | no | none |
| (no ordering) | Installation and Distance Selling | no | no | none |

Each position carries account mappings so that intra-union and export revenue is separated in the ledger. The distance-selling position exists because those supplies are declared in the destination country's return, not in the domestic one, and must therefore be excluded from the domestic report.

---

## 6. Rounding difference accounts

| Setting | Type | Effect |
|---|---|---|
| Rounding difference loss account | many_to_one to Account, company-checked | Receives a negative difference when a document total is rounded down. |
| Rounding difference profit account | many_to_one to Account, company-checked | Receives a positive difference when a document total is rounded up. |

**Worked example.** A total of 118.456 rounded to 118.46 posts a difference of 0.004 to the profit account; a total of 118.454 rounded to 118.45 posts −0.004 to the loss account.

---

## 7. Acceptance scenarios

**Given** a company in the Netherlands with no accounting,
**when** the Dutch template is loaded,
**then** the bank prefix is `103`, the cash prefix is `101`, the transfer prefix is `1060`, the rounding difference accounts are `4960` and `4950`, three tax groups exist and all three point at the value-added tax liabilities account.

**Given** an intra-union acquisition of 1,000.00 at 21 percent,
**when** the totals are computed,
**then** 210.00 is charged and 210.00 is deducted, the net amount due is 0.00 and the base of 1,000.00 is reported.

**Given** a distance sale to a consumer in another member state,
**when** the "Installation and Distance Selling" position is applied,
**then** the supply is excluded from the domestic periodic return.
