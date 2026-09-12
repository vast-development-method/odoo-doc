# Australia

The Australian package supplies one chart of accounts template, 130 tax rows covering the goods and
services tax and its exemptions, four tax groups, four fiscal positions, a registration flag and a
trading name on the company, and a family of fourteen statutory activity statements whose figures
are rounded down to whole units.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `au` |
| Display name | Australia |
| Parent template | none |
| Fiscal country | Australia |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Australian dollar |
| Bank account code prefix | the liquidity prefix declared by the template |
| Cash account code prefix | the liquidity prefix declared by the template |
| Transfer account code prefix | the transfer prefix declared by the template |
| Default sale tax | the 10 percent goods and services tax on sales |
| Default purchase tax | the 10 percent goods and services tax on purchases |
| Tax computation rounding method | Round per Tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 114 |
| Account group rows shipped | none; the chart is flat |
| Tax rows shipped | 130 |
| Tax group rows shipped | 4 |
| Fiscal position rows shipped | 4 |

The 130 tax rows are the tax records together with their repartition instructions and their report
tags; the number of user-selectable taxes is smaller, because each tax carries several repartition
rows.

---

## 3. Taxes

| Family | Rate | Scope | Reported in |
|---|---|---|---|
| Goods and services tax on sales | 10 percent | sale | the sales boxes of the activity statement |
| Goods and services tax on purchases | 10 percent | purchase | the acquisition boxes |
| Goods and services tax free sales | 0 percent | sale | the tax-free sales box |
| Input taxed sales | 0 percent | sale | the input-taxed box |
| Goods and services tax on capital purchases | 10 percent | purchase | the capital acquisition box |
| Not reportable | 0 percent | sale and purchase | nothing |

**Worked example.** A sale of 1,000.00 at 10 percent produces a tax of round(1,000.00 × 0.10, 2) =
100.00 and an invoice total of 1,100.00. The base of 1,000.00 is reported in the sales box and the
tax of 100.00 in the tax-on-sales box.

---

## 4. Fiscal positions

| Name | Detected automatically | Country or group | Effect |
|---|---|---|---|
| Domestic | yes | Australia | keeps the domestic taxes |
| Export | yes | none | maps the domestic taxes to the goods and services tax free sales tax |
| Import | yes | none | maps the purchase taxes to the import treatment |
| Not registered | no | none | removes the goods and services tax from every line |

---

## 5. Fields added

### 5.1 Company

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_au_is_gst_registered` | Australian goods and services tax registered | boolean | Whether the company is registered for the goods and services tax. When false, the taxes are removed from documents and the activity statement is not offered. |
| `l10n_au_trading_name` | Trading name | text | The name the business trades under, printed on invoices when it differs from the legal name. |

### 5.2 Payment

The Australian package adds the payment fields shared with New Zealand for the local direct-debit
and direct-credit formats: the account number and the branch number of the counterpart bank
account, validated by the country's own format checker.

---

## 6. Statutory reporting

Fourteen report structures are shipped. One is the activity statement itself and the other thirteen
are the sections that feed it, each of which can also be run on its own.

| Report | Purpose |
|---|---|
| Business activity statement | The periodic return, whose integer rounding setting is **Down**, so every figure is truncated towards zero. |
| Master activity statement | The consolidated statement for a tax unit. |
| Section A | Amounts the company owes the administration. |
| Section C | Total sales. |
| Section D | Export sales. |
| Section F | Other goods and services tax free sales. |
| Section G | Capital purchases. |
| Section U | Amounts the administration owes the company. |
| Section V | Net amount. |
| Section W | Withholding amounts. |
| Section X | Credits from withholding. |
| Section Y | Instalment amounts. |
| Section balance and section totals | The remaining two sections aggregate the above into the two figures the administration expects. |

Every one of them allows a foreign registration, groups several companies through a tax unit, and
defaults its period to the previous return period. Only tax-exigible journal items are considered,
so a cash basis tax appears only once it has been paid.

**Worked example of the integer rounding.** A net amount of 1,234.56 is reported as 1,234, because
the statement's integer rounding setting is Down. A net amount of −1,234.56 is reported as −1,234,
because Down means towards zero.

---

## 7. International invoice profile

The Australian and New Zealand invoice payload profile is a separate package that builds the
international payload both countries have adopted. It is described in
[country-packages.md](../country-packages.md); the payload itself is built by the shared builders of
[Electronic Invoicing and Document Exchange](../../electronic-invoicing-and-document-exchange/README.md).

---

## 8. Acceptance scenarios

**Given** a company in Australia with no accounting,
**when** the Australian template is loaded,
**then** 114 accounts exist, four tax groups exist, four fiscal positions exist and the default
sales tax is the 10 percent goods and services tax.

**Given** a company whose registration flag is false,
**when** an invoice is created,
**then** no goods and services tax is applied and the activity statement is not offered.

**Given** an invoice line of 1,000.00 with the 10 percent tax,
**when** the totals are computed,
**then** the tax is 100.00 and the total is 1,100.00.

**Given** an activity statement whose net amount computes to 1,234.56,
**when** the statement is rendered,
**then** the figure shown is 1,234, because the integer rounding setting is Down.
