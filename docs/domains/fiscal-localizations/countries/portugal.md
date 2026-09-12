# Portugal

The Portuguese package supplies one chart of accounts template of 642 accounts and 211 account groups built on the national accounting standards system, a taxonomy code on every account that the statutory audit file requires, 42 taxes covering the mainland rates and the two autonomous region rate sets, ten tax groups, and four fiscal positions.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `pt` |
| Display name | Portugal |
| Parent template | none |
| Fiscal country | Portugal |
| Account code length | 6 characters (the framework default) |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Receivable account recorded as a Contact default | `2111` |
| Payable account recorded as a Contact default | `2211` |

The remaining company values (prefixes, exchange accounts, income and expense accounts) follow the accounts of the national plan.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 642 |
| Account groups shipped | 211 |
| Code lengths present | 2 to 6 characters |
| Translations shipped | Portuguese |

**Account type distribution.**

| Account type | Count |
|---|---|
| Bank and Cash | 3 |
| Current Asset | 98 |
| Non-current Asset | 115 |
| Receivable | 29 |
| Equity | 23 |
| Current Year Earnings | 1 |
| Expense | 134 |
| Depreciation | 18 |
| Income | 137 |
| Current Liability | 40 |
| Non-current Liability and Payable | the remainder |

**First accounts of the plan.**

| Code | Name | Type |
|---|---|---|
| `11` | Cash | Bank and Cash |
| `12` | Bank | Bank and Cash |
| `13` | Other bank deposits | Bank and Cash |
| `1411` | Potentially favourable | Current Asset |
| `1412` | Potentially unfavourable | Current Liability |
| `2436` | Value-added tax payable | Current Liability |
| `2437` | Value-added tax receivable | Current Asset |

**Account groups.** 211 groups, for example `1..` Liquid financial means, `14..` Other financial instruments, `141..` Byproducts, `142..` Financial instruments held for negotiation, `143..` Other assets and financial liabilities, `2..` Accounts receivable and payable.

### 2.1 Taxonomy code

Every account carries an integer **taxonomy code**, which is the classification the statutory audit file requires. The code is stored on the account and is what the audit export writes for each account, independently of the account's own code.

---

## 3. Taxes

42 taxes covering the three mainland rates and the rate sets of the two autonomous regions, which are lower than the mainland rates.

| Rate | Region | Scope |
|---|---|---|
| 23 percent | mainland, standard | sale and purchase |
| 13 percent | mainland, intermediate | sale and purchase |
| 6 percent | mainland, reduced | sale and purchase |
| 22 percent | one autonomous region, standard | sale and purchase |
| 12 percent | that region, intermediate | sale and purchase |
| 5 percent | that region, reduced | sale and purchase |
| 16 percent | the other autonomous region, standard | sale and purchase |
| 9 percent | that region, intermediate | sale and purchase |
| 4 percent | that region, reduced | sale and purchase |
| 0 percent | exempt | sale and purchase |

A tax carries a product applicability where the rate differs between goods and services.

---

## 4. Tax groups

Ten groups, all wired to the payable account `2436` and the receivable account `2437`.

| Group | Country set on the group |
|---|---|
| Value-added tax 0% | Portugal |
| Value-added tax 4% | none |
| Value-added tax 5% | none |
| Value-added tax 6% | Portugal |
| Value-added tax 9% | none |
| Value-added tax 12% | none |
| Value-added tax 13% | Portugal |
| Value-added tax 16% | none |
| Value-added tax 22% | none |
| Value-added tax 23% | Portugal |

The four groups that carry the country are the mainland rates; the six regional groups deliberately carry no country so that they are not proposed by the country-matching availability of the mainland return.

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country or group |
|---|---|---|---|---|
| 10 | Portugal | yes | no | Portugal |
| 20 | Inside the European Union | yes | yes | European Union group |
| 30 | Private, European Union | yes | no | European Union group |
| 40 | Outside the European Union | yes | no | none |

---

## 6. Acceptance scenarios

**Given** a company in Portugal with no accounting,
**when** the Portuguese template is loaded,
**then** 642 accounts and 211 account groups exist, every account carries a taxonomy code, ten tax groups exist and all of them point at `2436` and `2437`.

**Given** a sale of goods in the mainland,
**when** the standard tax is chosen,
**then** the rate is 23 percent.

**Given** a sale in an autonomous region whose standard rate is 22 percent,
**when** the regional tax is chosen,
**then** the rate is 22 percent and the tax group carries no country, so the mainland return does not pick it up by country matching.
