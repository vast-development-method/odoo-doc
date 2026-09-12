# Chile

The Chilean package supplies one chart of accounts template of 195 accounts, 27 taxes covering the value-added tax, the professional fees withholding scale, the alcoholic beverages tax and the fuel excise taxes, five tax groups, thirteen fiscal positions that rewrite the accounts according to the deductibility of the input tax, a taxpayer type on every counterpart that governs which document class may be issued, the document type mechanism shared across Latin America with a strict numbering format, and customs and currency codes used on export documents.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `cl` |
| Display name | Chile |
| Parent template | none |
| Fiscal country | Chile |
| Account code length | 6 characters |
| Rounding method written | Round per Tax |
| Cost accounting flag | true |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `1101` |
| Cash account code prefix | `1101` |
| Transfer account code prefix | `117` |
| Default point of sale receivable account | `110421` |
| Gain exchange rate account | `320265` |
| Loss exchange rate account | `410195` |
| Default sale tax | 19 percent value-added tax, sale |
| Default purchase tax | 19 percent value-added tax, purchase |
| Income account | `310115` |
| Expense account | `410235` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `110612` |
| Receivable account recorded as a Contact default | `110310` |
| Payable account recorded as a Contact default | `210210` |
| Stock valuation account recorded as a company property | `110610` |

The stock valuation account `110612` names its stock expense account (`410230`) and its stock variation account (`603100`).

### 1.2 Journals

Beyond the six journals every template creates, the package adds one:

| Symbolic identifier | Name | Type | Code | Ordering | Uses documents | Default account |
|---|---|---|---|---|---|---|
| `domestic_purchase` | Domestic Purchases | Purchase | `DMP` | 2 | yes | `410235` |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 195 |
| Code length | 6 characters |
| Account groups shipped | none |
| Translations shipped | Spanish |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 40 |
| Fixed Asset | 8 |
| Non-current Asset | 13 |
| Prepayments | 3 |
| Receivable | 8 |
| Equity | 16 |
| Expense | 35 |
| Cost of Revenue | 4 |
| Income | 17 |
| Other Income | 4 |
| Current Liability | 34 |
| Non-current Liability | 6 |
| Payable | 5 |
| Off-balance | 2 |

Key tax accounts: `110720` recoverable value-added tax and `210760` value-added tax payable, which are the receivable and payable counterparts of every tax group.

---

## 3. Taxes

27 taxes. Every ordinary tax carries an administration code, which is an integer identifying the tax family in the statutory declaration.

### 3.1 Value-added tax

| Name | Printed label | Rate | Scope | Administration code | Tax group | Fiscal position |
|---|---|---|---|---|---|---|
| `19% VAT` | Value-added tax 19% Sale | 19 | sale | 14 | Value-added tax 19% | Chile Domestic |
| `19% VAT` | Value-added tax 19% Purchase | 19 | purchase | 14 | Value-added tax 19% | Chile Domestic |
| 19% F A | Value-added tax 19% Fixed Assets | 19 | purchase | 14 | Value-added tax 19% | Purchases, Fixed Assets |
| 19% F A C | Value-added tax 19% Fixed Assets Common Use | 19 | purchase | 14 | Value-added tax 19% | Purchases, Fixed Assets |
| 19% F A NR | Value-added tax 19% Fixed Assets Not Recoverable | 19 | purchase | 14 | Value-added tax 19% | the five non-recoverable positions |
| 19% NR | Value-added tax 19% Non-recoverable | 19 | purchase | 14 | Value-added tax 19% | the five non-recoverable positions |
| 19% C | Value-added tax 19% Common Use | 19 | purchase | 14 | Value-added tax 19% | Chile Domestic |
| 19% SM | Value-added tax 19% Supermarket | 19 | purchase | 14 | Value-added tax 19% | Shopping, Supermarket |

The six purchase variants all charge the same 19 percent but are routed to different accounts by their fiscal positions, so that the recoverable share, the common-use share and the non-deductible share are separated in the ledger without any manual reclassification.

### 3.2 Professional fees withholding

A dated ladder of rates, all with the purchase scope, the administration code 15 and the tax group "Second Category Withholding Tax". Each year's rate is a separate tax, so that a bill dated in an earlier year keeps the rate that applied then.

| Name | Printed label | Rate | Active on install |
|---|---|---|---|
| 10.75% WH | Withholding second category 2020 | −10.75 | no |
| 11.5% WH | Withholding second category 2021 | −11.5 | no |
| 12.25% WH | Withholding second category 2022 | −12.25 | yes |
| 13% WH | Withholding second category 2023 | −13 | yes |
| 13.75% WH | Withholding second category 2024 | −13.75 | yes |
| 14.5% WH | Withholding second category 2025 | −14.5 | yes |
| 15.25% WH | Withholding second category 2026 | −15.25 | yes |

**Worked example.** A professional fees bill of 1,000,000 local units dated in 2025 carries the 14.5 percent withholding: `round(1,000,000 × 0.145, 0) = 145,000`. The supplier is paid 855,000 and the company remits 145,000.

### 3.3 Total value-added tax withholding

| Name | Rate | Scope | Administration code | Tax group |
|---|---|---|---|---|
| 19% WH | −19 | purchase | 15 | Withholdings |

Used when the buyer must account for the whole tax instead of the seller.

### 3.4 Alcoholic and non-alcoholic beverages tax

Eight taxes, four for purchases and four for sales, all with the tax group "Beverages tax".

| Rate | Meaning | Administration code |
|---|---|---|
| 10 percent | Non-alcoholic beverages | 27 |
| 18 percent | Non-alcoholic beverages, high sugar | 26 |
| 20.5 percent | Wines | 25 |
| 31.5 percent | Spirits | 24 |

### 3.5 Specific taxes

| Name | Computation | Scope | Administration code | Tax group |
|---|---|---|---|---|
| 63% Spec | 63 percent | purchase | 29 | Specific Taxes |
| Fuel excise, petrol | fixed amount of 1 unit, typed per document | purchase | 35 | Specific Taxes |
| Fuel excise, diesel | fixed amount of 1 unit, typed per document | purchase | 28 | Specific Taxes |

The two fuel excise taxes are fixed-amount taxes whose amount the operator types, because the published rate is an amount per cubic metre that changes weekly.

---

## 4. Tax groups

| Group | Payable account | Receivable account |
|---|---|---|
| Value-added tax 19% | `210760` | `110720` |
| Specific Taxes | `210760` | `210760` |
| Beverages tax | `210760` | `210760` |
| Second Category Withholding Tax | `210760` | `210760` |
| Withholdings | `210760` | `210760` |

---

## 5. Fiscal positions

Thirteen positions. Only the domestic one names a country; the others are chosen by hand to express how the input tax must be treated. Each carries account mappings that redirect the input tax to the correct account.

| Name | Purpose |
|---|---|
| Chile Domestic | The domestic position; every ordinary tax names it. |
| Purchases, intended to generate non-taxable or exempt transactions | Input tax not recoverable because the output is exempt. |
| Purchases, invoices from suppliers registered after the due date | Input tax not recoverable because the supplier registered late. |
| Purchases, rejected expenses | Input tax not recoverable because the expense is not deductible. |
| Purchases, free deliveries received | Input tax not recoverable on goods received free. |
| Purchases, others | Any other non-recoverable case. |
| Purchases, fixed assets | Input tax on capital goods, reported separately. |
| Purchases, exempt | Purchases carrying no tax. |
| Shopping, supermarket | Supermarket purchases, whose deductibility is capped. |
| Sales, exempt | Exempt sales. |
| Sales, exports | Export sales. |
| Purchases, imports | Imports, whose tax is paid at customs. |
| Purchases, fees | Professional fees, which carry the withholding rather than the value-added tax. |

---

## 6. Counterparts

| Field | Values | Meaning |
|---|---|---|
| Taxpayer type | `1` (value-added tax affected, first category), `2` (fees receipt issuer, second category), `3` (end consumer), `4` (foreigner) | Decides which document classes may be issued to the counterpart and which taxes are allowed. |
| Activity description | text | The economic activity, printed on documents. |

**National identification validation.**

```
The format of your RUN is not valid. It should be like 76086428-5.
```

**Company.** The company mirrors the activity description from its own contact.

---

## 7. Document types

The package uses the shared Latin American mechanism and extends the internal type list with four values: purchase invoices, receipt invoices, stock deliveries and the base four. Each document type carries an "active in the localization" marker; only marked types are offered on invoices.

**Number format.** The number, called the folio, must contain only digits:

```
The DTE document number (folio) must contain only digits.
```

**Posting constraints.**

| Rule | Message |
|---|---|
| A document class that requires an identified counterpart needs a taxpayer type and a number. | `Tax payer type and vat number are mandatory for this type of document` |
| A foreign counterpart requires an export document class or a receipt to an end consumer. | `Document types for foreign customers must be export type (codes 110, 111 or 112) or you should define the customer as an end consumer and use receipts (codes 39 or 41)` |
| The import declaration class may only be used with the customs authority's own identification number. | `The DIN document is intended to be used only with RUT 60805000-0` |
| The supplier's taxpayer type must match the document class. | `The tax payer type of this supplier is incorrect for the selected type of document.` |
| A supplier of the wrong taxpayer type may not issue that class. | `The tax payer type of this supplier is not entitled to deliver this type of document.` |
| A foreign vendor bill needs a journal that does not use documents. | `You need a journal without the use of documents for foreign suppliers.` |

---

## 8. Reference data on platform entities

| Entity | Field | Meaning |
|---|---|---|
| Country | Customs code, customs name, customs abbreviation | The customs authority's designation of the country, printed on export documents. |
| Currency | Currency code, short name | The customs authority's designation of the currency. |
| Unit of Measure | Administration code | The unit code used on electronic documents. |
| Tax | Administration code | The integer that identifies the tax family in the declaration. |
| Bank Account | Supervisory authority code, at most 10 characters | The banking supervisor's code for the institution. |

---

## 9. Acceptance scenarios

**Given** a company in Chile with no accounting,
**when** the Chilean template is loaded,
**then** 195 accounts exist, the account code length is 6, both the bank and the cash prefix are `1101`, the transfer prefix is `117`, five tax groups exist and a purchase journal named "Domestic Purchases" with the code `DMP` exists and uses documents.

**Given** a supplier bill for professional fees dated in 2025,
**when** the 14.5 percent second-category withholding is applied to a base of 1,000,000,
**then** the withheld amount is 145,000.

**Given** a purchase invoice whose fiscal position is "Purchases, rejected expenses",
**when** the line carrying the 19 percent purchase tax is prepared,
**then** the tax is replaced by the non-recoverable 19 percent tax and the amount is booked as an expense rather than as recoverable tax.

**Given** a foreign customer and a domestic invoice document class,
**when** the invoice is posted,
**then** the posting is refused with the export-document message.

**Given** a document number containing a letter,
**when** the invoice is saved,
**then** the save is refused with `The DTE document number (folio) must contain only digits.`
