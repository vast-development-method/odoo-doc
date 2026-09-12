# Mexico

The Mexican package supplies one chart of accounts template aligned with the tax administration's account grouping code, a set of taxes covering the value-added tax, the excise tax and the income tax withholdings, fourteen tax groups each wired to its own payable and receivable account, five fiscal positions, two extra journals, a balance-direction tagging rule applied to every account, and the classification fields that an electronic invoice payload requires: the factor type and the administration tax type on every tax, and the banking institution codes on bank records.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `mx` |
| Display name | Mexico |
| Parent template | none |
| Fiscal country | Mexico |
| Account code length | 9 characters |
| Contributing package category | Accounting localizations, charts of accounts |
| Automatically installed with the accounting package | yes |

### 1.1 Company-level values written by the template

| Company field | Value written |
|---|---|
| Fiscal country | Mexico |
| Account code length used for padding | 9 |
| Cost accounting flag (expense recognised at the customer invoice) | true |
| Print the invoice total in words | true |
| Bank account code prefix | `102.01.0` |
| Cash account code prefix | `101.01.0` |
| Transfer account code prefix | `102.01.01` |
| Default point of sale receivable account | `105.02.01` Foreign Customers |
| Gain exchange rate account | `702.01` |
| Loss exchange rate account | `701.01` |
| Deferred expense account | `173.01` |
| Cash discount write-off loss account | `402.01` |
| Cash discount write-off gain account | `503.01` |
| Cash basis journal | the journal created under the symbolic identifier `cbmx` |
| Cash basis base account | `801.01.99` |
| Default sale tax | the 16 percent sales tax |
| Default purchase tax | the 16 percent purchase tax |
| Income account | `401.01` |
| Expense account | `601.84` |
| Income account for returns and discounts | `402.01` |
| Income account for re-invoicing | `402.04` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `115.01` |
| Receivable account recorded as a Contact default | `105.01.01` Domestic customers |
| Payable account recorded as a Contact default | `201.01` |
| Stock valuation account recorded as a company property | `115.01` |
| Cash basis base account recorded as a company property | `801.01.99` |

### 1.2 Utility accounts overridden by the package

| Company field | Name | Code |
|---|---|---|
| Cash difference income account | Other Income | `403.01.01` |
| Cash difference expense account | Cash Difference Loss | `601.84.02` |

The four other utility accounts (bank suspense, the two cash discount accounts, the liquidity transfer account and the two outstanding accounts) follow the framework rules of [../configuration.md](../configuration.md) section 5, numbered under the prefixes above.

### 1.3 Journals

Beyond the six journals every template creates, the package adds:

| Symbolic identifier | Name | Type | Code | Default account | Shown on dashboard |
|---|---|---|---|---|---|
| `cbmx` | Effectively Paid | Miscellaneous | `CBMX` | `118.01` | yes |
| `cash` | Cash | Cash | assigned from the cash prefix | yes |

The `cbmx` journal is the cash basis journal: every tax that becomes exigible on payment posts its transfer entry there.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 140 |
| Account groups shipped | 1,079 |
| Code length | 9 characters, in the three-part form `NNN.NN.NN` |
| Translations shipped | Spanish for every account name and description |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 25 |
| Fixed Asset | 15 |
| Non-current Asset | 2 |
| Receivable | 7 |
| Equity | 2 |
| Current Year Earnings | 1 |
| Expense | 29 |
| Cost of Revenue | 23 |
| Income | 3 |
| Other Income | 2 |
| Current Liability | 25 |
| Payable | 6 |

**Account groups.** The 1,079 groups reproduce the administration's grouping code at three levels: a one-character level (for example `1` Active), a three-character level (for example `101` Cash) and a six-character level (for example `101.01` Cash in hand). Each group carries a starting prefix only; the ending prefix defaults to it.

**Examples of shipped accounts.**

| Code | Name | Type |
|---|---|---|
| `102.02.01` | Foreign currency bank transfers | Current Asset |
| `105.01.01` | Domestic customers | Receivable |
| `105.01.02` | National customers (point of sale) | Receivable |
| `105.02.01` | Foreign Customers | Receivable |
| `107.01.01` | Officers and Employees | Receivable |
| `107.05.01` | Goods Shipped, No Invoices | Current Asset |
| `108.01.01` | Allowance for doubtful accounts national | Current Asset |
| `108.02.01` | Allowance for doubtful accounts foreign | Current Asset |
| `110.01.01` | Employment subsidy to apply | Current Asset |
| `113.01.01` | Value-added tax credit | Receivable |
| `113.02.01` | Income tax credit | Receivable |
| `113.02.02` | Income tax withheld | Current Asset |
| `115.01` | Inventory | Current Asset |
| `119.01` | Value-added tax pending on purchases | Current Asset |
| `119.02` | Value-added tax on imports | Current Asset |
| `119.03` | Excise tax pending on purchases | Current Asset |
| `209.01` | Value-added tax pending on sales | Current Liability |
| `209.02` | Excise tax pending on sales | Current Liability |
| `213.01` | Value-added tax payable | Current Liability |
| `213.02` | Excise tax payable | Current Liability |
| `213.03` | Income tax withheld payable | Current Liability |
| `216.04` | Income tax withholdings payable | Current Liability |
| `216.10` | Value-added tax withholdings payable | Current Liability |
| `401.01` | Sales income | Income |
| `402.01` | Returns and discounts on sales | Income |
| `402.04` | Re-invoicing | Income |
| `403.01.01` | Other Income | Other Income |
| `601.84` | Other expenses | Expense |
| `601.84.02` | Cash Difference Loss | Expense |
| `801.01.99` | Cash basis base account | off-balance |

### 2.1 Automatic balance-direction tagging

Every account created for a Mexican company that carries neither the debit-balance tag nor the credit-balance tag receives one of them automatically at creation:

```
tag = debit_balance_tag   when the first character of the code is "1", "5", "6" or "7"
tag = credit_balance_tag  otherwise
```

The rule is applied on creation only, and only when both tags exist in the database. It is deliberately approximate: it is a starting classification that the accountant may change, and it exists so that the statutory chart export always carries a direction.

**Worked example.** Account `115.01` starts with `1` and is tagged debit balance. Account `209.01` starts with `2` and is tagged credit balance. Account `601.84` starts with `6` and is tagged debit balance.

### 2.2 Credit note account substitution

When a customer credit note line of a Mexican company is a product line and the company has an income account for returns and discounts, the line's account is forced to that account instead of the ordinary revenue account.

```
country = "MX" AND entry_kind = "customer_credit_note" AND line_kind = "product"
AND company.income_account_for_returns_and_discounts IS SET
    → line.account = company.income_account_for_returns_and_discounts
```

**Worked example.** A credit note of 1,000.00 for goods sold. Without the rule the line would credit `401.01` Sales income with a negative amount; with the rule it debits `402.01` Returns and discounts on sales with 1,000.00, so the gross sales figure of the income statement is preserved and the returns figure is visible on its own line.

---

## 3. Taxes

Thirty-three taxes are shipped. Every one carries a factor type and, except the two exempt taxes, an administration tax type. Nineteen of them are shipped archived and must be activated by the accountant when the company's regime needs them.

### 3.1 Value-added tax

| Name | Printed label | Rate | Scope | Tax group | Exigibility | Cash basis transition account | Active on install | Fiscal position |
|---|---|---|---|---|---|---|---|---|
| 0% | Value-added tax (0%) | 0 | sale | Value-added tax 0% | on payment | `209.01` | yes | Foreign Customer; replaces the 16 percent and the 8 percent sales taxes |
| 16% | Value-added tax (16%) | 16 | sale | Value-added tax 16% | on payment | `209.01` | yes | Domestic |
| 8% | Value-added tax (8%) | 8 | sale | Value-added tax 8% | on payment | `209.01` | yes | Domestic |
| Exento | 0% Exento | 0 | sale | Exento | on invoice | `209.01` | yes | Domestic |
| 0% | Value-added tax (0%) | 0 | purchase | Value-added tax 0% | on payment | `119.01` | yes | Foreign Customer; replaces the 16 percent and the 8 percent purchase taxes |
| 16% | Value-added tax (16%) | 16 | purchase | Value-added tax 16% | on payment | `119.01` | yes | Domestic |
| 8% | Value-added tax (8%) | 8 | purchase | Value-added tax 8% | on payment | `119.01` | yes | Domestic |
| 8% S. | Value-added tax (8%) S. | 8 | purchase | Value-added tax 8% | on payment | `119.01` | yes | Domestic |
| 16% NC | Value-added tax (16%) NC | 16 | purchase | Value-added tax 16% | on payment | `119.01` | yes | Domestic |
| 16% IMP | Value-added tax (16%) Imports | 16 | purchase | Value-added tax 16% | on payment | `119.02` | yes | Domestic |
| 16% IMP INT | Value-added tax (16%) Non-Tangible Imports | 16 | purchase | Value-added tax 16% | on payment | `119.02` | yes | Domestic |
| Exento | 0% Exento | 0 | purchase | Exento | on invoice | `119.01` | yes | Domestic |

Three purchase taxes exist only to be substituted by a fiscal position and all carry the 16 percent rate:

| Name | Printed label | Fiscal position | Replaces |
|---|---|---|---|
| `16% VAT T` | Value-added tax T (16%) | Freight | the 16 percent purchase tax |
| `16% VAT 2/3 H` | Value-added tax 2/3 H (16%) | Honorarium | the 16 percent purchase tax |
| `16% VAT 2/3 L` | Value-added tax 2/3 L (16%) | Lease | the 16 percent purchase tax |

### 3.2 Value-added tax withholdings

All of them carry a negative rate, the purchase scope and the on-payment exigibility, and all post to the transition account `216.10`.

| Name | Printed label | Rate | Tax group |
|---|---|---|---|
| 4% WH | Value-added tax withholding (−4%) | −4 | Value-added tax Retention 4% |
| 10% WH L | Value-added tax withholding (−10%) | −10 | Value-added tax Retention 10% |
| 10.67% WH L | Value-added tax withholding (−10.67%) | −10.666666666667 | Value-added tax Retention 10.67% |
| 10.67% WH | Value-added tax withholding (−10.67%) | −10.666666666667 | Value-added tax Retention 10.67% |

The two 10.666666666667 rates express two thirds of the 16 percent rate exactly: `16 × 2 ÷ 3 = 10.6666…`. The stored precision is twelve decimal places, which is what makes the withheld amount agree with the administration's own rounding.

**Worked example.** A lease invoice with a base of 30,000.00 and a 16 percent value-added tax of 4,800.00. The two-thirds withholding is `round(30,000.00 × 0.10666666666667, 2) = 3,200.00`, which is exactly two thirds of 4,800.00.

### 3.3 Income tax withholdings

| Name | Printed label | Rate | Scope | Tax group | Exigibility | Transition account | Active on install |
|---|---|---|---|---|---|---|---|
| 10% WH L I | Withholding income (−10%) | −10 | purchase | Income tax Retention 10% | on invoice | none | yes |
| 10% WH I S | Withholding income salaries (−10%) | −10 | purchase | Income tax Retention 10% | on invoice | none | yes |
| 1.25% WH | Withholding income tax (−1.25%) | −1.25 | purchase | Income tax Retention 1.25% | on invoice | `216.04` | no |
| 1.25% WH | Withholding income tax (−1.25%) | −1.25 | sale | Income tax Retention 1.25% | on invoice | `113.02.01` | no |

The 1.25 percent pair implements the simplified regime for individuals; both are shipped archived.

### 3.4 Excise tax

Ten excise taxes are shipped, five for sales and five for purchases, all with the on-payment exigibility, all with "include in base amount" set to true so that the value-added tax is computed on the price plus the excise tax, and all posting to `209.02` for sales and `119.03` for purchases. Every sale-side excise tax is shipped archived.

| Rate | Printed label | Sale tax active | Purchase tax active |
|---|---|---|---|
| 8 percent | Excise tax 8% | no | yes |
| 25 percent | Excise tax 25% | no | yes |
| 26.5 percent | Excise tax 26.5% | no | yes |
| 30 percent | Excise tax 30% | no | yes |
| 53 percent | Excise tax 53% | no | yes |

**Worked example of the base inclusion.** A product priced 1,000.00 carrying the 30 percent excise tax and the 16 percent value-added tax.

```
excise            = round(1,000.00 × 0.30, 2) = 300.00
value-added base  = 1,000.00 + 300.00        = 1,300.00
value-added tax   = round(1,300.00 × 0.16, 2) =  208.00
invoice total     = 1,000.00 + 300.00 + 208.00 = 1,508.00
```

### 3.5 Tax classification fields

| Field on Tax | Values | Default | Meaning |
|---|---|---|---|
| Factor type | `Tasa` (rate), `Cuota` (fixed amount per unit), `Exento` (exempt) | `Tasa` | How the tax is expressed on the electronic document. A rate tax transmits a percentage; a fixed-amount tax transmits an amount per unit; an exempt tax transmits no amount at all. |
| Administration tax type | `isr` (income tax), `iva` (value-added tax), `ieps` (excise tax), `local` (local tax); derived from the tax and editable, stored | derived | Which family the tax belongs to. |

**Two kinds of zero.** The country distinguishes a supply taxed at zero percent, which carries the factor type `Tasa` and a rate of zero, from an exempt supply, which carries the factor type `Exento` and no amount. The package ships both: the taxes named "0%" carry `Tasa` and the taxes named "Exento" carry `Exento`.

**Fixed-amount taxes.** A tax whose factor type is `Cuota` cannot be expressed as a percentage. It is configured as a formula tax computing `quantity × quota_value`. Only a per-unit quota is supported.

**Worked example.** A quota of 6.455 per unit on a line of 40 units gives `round(40 × 6.455, 2) = 258.20`.

**Local taxes.** A tax whose administration tax type is `local` is reported in a separate section of the electronic document and is not validated by the certification provider. A negative rate on a local tax is reported as a withholding; a positive rate is reported as a charge.

---

## 4. Tax groups

Fourteen groups, all with the country Mexico. Every group names a payable account and a receivable account, which are the two counterparts the closing entry uses.

| Group | Payable account | Receivable account |
|---|---|---|
| Value-added tax 0% | `213.01` | `113.01` |
| Value-added tax 16% | `213.01` | `113.01` |
| Value-added tax 8% | `213.01` | `113.01` |
| Exento | `213.01` | `113.01` |
| Value-added tax Retention 4% | `213.01` | `113.01` |
| Value-added tax Retention 10% | `213.01` | `113.01` |
| Value-added tax Retention 10.67% | `213.01` | `113.01` |
| Income tax Retention 1.25% | `213.03` | `113.02.01` |
| Income tax Retention 10% | `213.03` | `113.02.01` |
| Excise tax 8% | `213.02` | `113.08` |
| Excise tax 25% | `213.02` | `113.08` |
| Excise tax 26.5% | `213.02` | `113.08` |
| Excise tax 30% | `213.02` | `113.08` |
| Excise tax 53% | `213.02` | `113.08` |

---

## 5. Fiscal positions

| Sequence | Name | Country | Purpose |
|---|---|---|---|
| 10 | Mexico Domestic | Mexico | The domestic position. Every ordinary tax names it, so a domestic document keeps its taxes unchanged. |
| 20 | Foreign Customer | none | Substitutes the zero percent taxes for the 16 percent and 8 percent taxes on both sides. |
| 30 | Freight | Mexico | Substitutes the freight variant of the 16 percent purchase tax. |
| 40 | Honorarium | Mexico | Substitutes the professional fees variant of the 16 percent purchase tax. |
| 50 | Lease | Mexico | Substitutes the lease variant of the 16 percent purchase tax. |

**Worked example of the Foreign Customer position.** An invoice line carries the 16 percent sales tax. The customer's fiscal position is Foreign Customer. The substitution replaces the 16 percent tax by the zero percent sales tax, because the zero percent tax names the 16 percent tax as a source tax. The document total therefore carries no tax and the base is reported on the export line of the return.

---

## 6. Fields added

### 6.1 Company

| Field | Type | Meaning |
|---|---|---|
| Income account for returns and discounts | many_to_one to Account of an income type | Where a customer credit note line posts instead of the revenue account. |
| Income account for re-invoicing | many_to_one to Account | Where a re-issued invoice posts. |

### 6.2 Tax

| Field | Type | Meaning |
|---|---|---|
| Factor type | selection `Tasa`, `Cuota`, `Exento`, default `Tasa` | How the tax is expressed on the electronic document. |
| Administration tax type | selection `isr`, `iva`, `ieps`, `local`, derived and editable, stored | Which family the tax belongs to. |

### 6.3 Bank

| Field | Type | Meaning |
|---|---|---|
| Banking association code | text of 3 digits | The three-digit number the banking association assigns to identify an institution. |

### 6.4 Bank Account

| Field | Type | Meaning |
|---|---|---|
| Standardised banking cipher | text | The national standardised account number, required on payment instructions. |
| Fiscal country codes | text, not stored | Technical helper that reveals the two fields above only for a Mexican company. |

---

## 7. Asset models shipped

The package ships nine depreciation models, each naming its accumulated depreciation account, its depreciation expense account, its period in months and its number of periods.

| Model | Accumulated depreciation account | Depreciation expense account | Period, months | Number of periods |
|---|---|---|---|---|
| Technology | `171.05.01` | `613.05.01` | 12 | 3 |
| Renewable Energy Technology Installation | `171.16.01` | `613.16.01` | 12 | 10 |
| Machinery and equipment | `171.02.01` | `613.02.01` | 12 | 5 |
| Other Machines and Equipment | `171.18.01` | `613.18.01` | 12 | 5 |
| Furniture and Office Equipment | `171.04.01` | `613.04.01` | 12 | 5 |
| Vehicles | `171.03.01` | `613.03.01` | 12 | 5 |
| Upgrades and Retrofits | `171.17.01` | `613.17.01` | 12 | 3 |
| Brands and Patents | `183.07.01` | `614.07.01` | 1 | 80 |
| Deferred Expenses | `183.01.01` | `614.01.01` | 12 | 1 |

**Worked example.** A vehicle capitalised at 480,000.00 under the Vehicles model depreciates over 5 periods of 12 months, that is 5 years, at `round(480,000.00 ÷ 5, 2) = 96,000.00` per year, debiting `613.03.01` and crediting `171.03.01`.

---

## 8. Reports

| Report | Content |
|---|---|
| Third-party operations declaration | One line per supplier and per operation type, with the supplier's national taxpayer number, the country when foreign, the base subject to each value-added tax rate, the withheld amounts and the exempt base. The report is built as a financial report with one line per declaration box, fed by report tags carried by the purchase taxes. |
| Chart of accounts export | Every account with its code, its name, its grouping code and its balance direction tag. |
| Trial balance | Opening balance, period debits, period credits and closing balance per account, grouped by grouping code. |

---

## 9. Currency rule

The country's legal currency is the Mexican peso and a Mexican company must keep its books in it. The template writes the peso as the company currency at load time, because the currency comes from the fiscal country. A company that prices in another currency uses a price list rather than a second company currency.

---

## 10. Numbering and documents

The package does not install the Latin American document type mechanism. Invoices are numbered by the ordinary journal numbering series. The legal number an electronic document carries is the invoice number produced by that series, and the registration identifier the administration returns is stored separately.

---

## 11. Validation rules specific to this country

| Rule | Condition | Behavior |
|---|---|---|
| Balance direction tag | an account is created for a company whose countries include Mexico and it carries neither direction tag | The debit or credit tag is added according to the first character of the code. |
| Credit note account | a customer credit note product line and the company has an income account for returns and discounts | The line's account is forced to that account. |
| Company currency | a Mexican company | The peso is written at template load; changing it later is refused once journal items exist. |
| Factor type on a quota tax | the factor type is `Cuota` | The tax must use a formula computation; a percentage computation cannot express a per-unit amount. |
| Exempt versus zero | the factor type is `Exento` | The tax transmits no amount; the base is still reported. |

---

## 12. Acceptance scenarios

**Given** a company with no accounting entries and the country Mexico,
**when** the Mexican template is loaded,
**then** the company currency is the Mexican peso, the account code length is 9, the bank prefix is `102.01.0`, the cash prefix is `101.01.0`, the transfer prefix is `102.01.01`, 140 accounts and 1,079 account groups exist, 33 taxes and 14 tax groups exist, the cash difference income account is `403.01.01` named "Other Income", the cash difference expense account is `601.84.02`, and the cash basis journal is the one coded `CBMX`.

**Given** the Mexican chart is loaded,
**when** an account with the code `700100` is created by hand,
**then** it receives the debit-balance tag, because its first character is `7`.

**Given** a posted customer invoice of 1,000.00 for goods,
**when** a credit note is created from it,
**then** the product line of the credit note debits `402.01` Returns and discounts on sales rather than crediting `401.01` Sales income with a negative amount.

**Given** an invoice line of 1,000.00 carrying the 30 percent excise tax and the 16 percent value-added tax,
**when** the totals are computed,
**then** the excise tax is 300.00, the value-added tax is 208.00 and the total is 1,508.00.

**Given** a lease bill with a base of 30,000.00,
**when** the two-thirds value-added tax withholding is applied,
**then** the withheld amount is 3,200.00 and the tax group is Value-added tax Retention 10.67%.

**Given** a customer whose fiscal position is Foreign Customer,
**when** an invoice line carrying the 16 percent sales tax is prepared,
**then** the line carries the zero percent sales tax instead and the invoice total contains no tax.
