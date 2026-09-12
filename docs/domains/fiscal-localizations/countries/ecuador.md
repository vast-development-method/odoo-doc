# Ecuador

The Ecuadorian package supplies one chart of accounts template of 501 accounts with a hierarchical group tree of 112 nodes, 182 taxes covering six value-added tax rates, the special consumption tax, the plastic bottle tax, the value-added tax and income tax withholding ladders and the exchange outflow tax, 17 tax groups each carrying an Ecuadorian subtype, two fiscal positions, the emission entity and emission point carried on every journal, the administration payment method on every invoice, and the document type mechanism with a strict three-part numbering format.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `ec` |
| Display name | Ecuador |
| Parent template | none |
| Fiscal country | Ecuador |
| Account code length | 4 characters (the shipped codes are 2 to 10 characters; only the shortest are padded) |
| Rounding method written | Round per Line |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `11010201` |
| Cash account code prefix | `1101010` |
| Transfer account code prefix | `1101030` |
| Default point of sale receivable account | `1102050103` |
| Gain exchange rate account | `430501` |
| Loss exchange rate account | `520304` |
| Cash discount write-off loss account | the early payment discount loss account |
| Cash discount write-off gain account | the early payment discount gain account |
| Cash difference income account | the income cash difference account |
| Cash difference expense account | the expense cash difference account |
| Default sale tax | 15 percent value-added tax on goods |
| Default purchase tax | 15 percent value-added tax on purchases |
| Income account | `410101` |
| Expense account | `110307` |
| Tax calculation rounding method | Round per Line |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `110306` |
| Receivable account recorded as a Contact default | `1102050101` |
| Payable account recorded as a Contact default | `210301` |
| Stock valuation account recorded as a company property | `110306` |
| Stock loss account recorded as a company property | `510112` |
| Production stock valuation account recorded as a company property | `110302` |

The stock valuation account `110306` names its stock expense account (`510106`) and its stock variation account (`110310`).

### 1.2 Sales journal override

| Field | Value |
|---|---|
| Name | `001-001 Facturas de cliente` |
| Emission entity | `001` |
| Emission point | `001` |
| Emission address | the company's own contact |

### 1.3 Extra post-load step

After the ordinary post-processing, the purchase journal's default account is set to the account named by the template key for the purchase journal expense category, which is `52022816`.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 501 |
| Code lengths present | 2, 4, 6, 8 and 10 characters |
| Account groups shipped | 112, arranged as a tree with explicit parent links |
| Translations shipped | Spanish |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 54 |
| Fixed Asset | 35 |
| Non-current Asset | 10 |
| Prepayments | 4 |
| Receivable | 13 |
| Equity | 15 |
| Current Year Earnings | 1 |
| Expense | 172 |
| Depreciation | 32 |
| Cost of Revenue | 68 |
| Income | 19 |
| Current Liability | 48 |
| Non-current Liability | 4 |
| Payable | 26 |

**Group tree.** Unlike most packages, the Ecuadorian groups carry an explicit parent link as well as a prefix, so the hierarchy is stated rather than derived. Examples: `1` Active, `11` Current assets, `1101` Cash and cash equivalents, `110101` Cash, `110102` Banks, `110103` Transfers, `110104` Securities in custody, `1102` Financial assets, `110205` Unrelated notes and accounts receivable.

---

## 3. Taxes

182 taxes. Every tax carries three declaration codes:

| Field | Meaning |
|---|---|
| Base declaration code | The box of the return that receives the base, before the tax is computed. |
| Applied declaration code | The box that receives the computed tax. |
| Annexed transaction code | Whether a purchase invoice supports a tax credit, a cost or an expense, according to the annex's classification table. |

### 3.1 Value-added tax rates

| Rate | Tax group | Ecuadorian subtype |
|---|---|---|
| 5 percent | Value-added tax 5% | `vat05` |
| 8 percent | Value-added tax 8% | `vat08` |
| 12 percent | Value-added tax 12% | `vat12` |
| 13 percent | Value-added tax 13% | `vat13` |
| 14 percent | Value-added tax 14% | `vat14` |
| 15 percent | Value-added tax 15% | `vat15` |
| 0 percent | Value-added tax 0% | `zero_vat` |
| not subject | Non-subject value-added tax | `not_charged_vat` |
| exempt | Exempt value-added tax | `exempt_vat` |

Six rates coexist because the standard rate has been changed several times and a document dated in an earlier period must keep the rate that applied then.

### 3.2 Other tax families

| Family | Tax group | Ecuadorian subtype | Payable account | Receivable account |
|---|---|---|---|---|
| Special consumption tax | Special Consumptions | `ice` | special consumption tax deduction | special consumption tax deduction |
| Plastic bottle tax | Plastic Bottles | `irbpnr` | plastic bottle tax deduction | plastic bottle tax deduction |
| Value-added tax withholding on sales | Sales Withholding Value-added tax | `withhold_vat_sale` | value-added tax deduction | withholding tax credit |
| Value-added tax withholding on purchases | Purchase Withholding Value-added tax | `withhold_vat_purchase` | value-added tax deduction | withholding tax credit |
| Income tax withholding on sales | Sale Profit Withhold | `withhold_income_sale` | profit tax deduction | profit tax credit |
| Income tax withholding on purchases | Purchase Profit Withhold | `withhold_income_purchase` | profit tax deduction | profit tax credit |
| Exchange outflow tax | Exchange Outflows | `outflows_tax` | other tax deduction | other tax credit |
| Others | Others | `other` | other tax deduction | other tax credit |

The value-added tax groups all share the same pair: the value-added tax deduction account on the payable side and the value-added tax credit account on the receivable side.

### 3.3 Withholding ladders

The income tax withholding family carries one tax per published rate and per nature of payment, because the rate depends on what is being paid. Each carries its own applied declaration code, which is the box of the withholding return.

**Worked example.** A service bill of 1,000.00 carrying the 15 percent value-added tax and a 10 percent income tax withholding and a 70 percent value-added tax withholding.

```
value-added tax        = round(1,000.00 × 0.15, 2) = 150.00
income tax withheld    = round(1,000.00 × 0.10, 2) = 100.00
value-added tax withheld = round(150.00 × 0.70, 2) = 105.00
amount paid to supplier = 1,000.00 + 150.00 − 100.00 − 105.00 = 945.00
```

---

## 4. Tax groups

17 groups, each with an ordering number that fixes the order of the subtotals on the printed document.

| Ordering | Group |
|---|---|
| 3 | Value-added tax 5% |
| 5 | Value-added tax 8% |
| 10 | Value-added tax 12% |
| 15 | Value-added tax 13% |
| 20 | Value-added tax 14% |
| 25 | Value-added tax 15% |
| 30 | Value-added tax 0% |
| 40 | Non-subject value-added tax |
| 50 | Exempt value-added tax |
| 60 | Special Consumptions |
| 70 | Plastic Bottles |
| 80 | Sales Withholding Value-added tax |
| 80 | Purchase Withholding Value-added tax |
| 90 | Sale Profit Withhold |
| 95 | Purchase Profit Withhold |
| 100 | Exchange Outflows |
| 110 | Others |

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax identification number | Country |
|---|---|---|---|---|
| 10 | National regime | yes | no | Ecuador |
| 20 | Foreign regime | yes | yes | none |

---

## 6. Emission entity and emission point

Every journal that issues legal documents carries the two numbers the administration assigns, which together form the first two parts of the document number.

| Field | Type | Meaning |
|---|---|---|
| Requires emission data | boolean, derived | True when the journal issues legal documents and therefore needs the two numbers. |
| Emission entity | text of exactly 3 characters, not copied | The establishment number. |
| Emission point | text of exactly 3 characters, not copied | The issuing point number. |
| Emission address | many_to_one to Contact restricted to the company's own addresses | The address printed on documents issued here. |

**Document number format.** Three parts: the entity, the emission point and a nine-digit counter.

```
Ecuadorian Document <number> must be like 001-001-123456789
```

---

## 7. Payment method

Every invoice carries a payment method chosen from a shared catalogue that the administration publishes.

| Entity | Fields |
|---|---|
| Ecuador Payment Method | sequence, name (translatable), code, active |

The default on a new invoice is the first entry of the catalogue in sequence order.

---

## 8. Document types

The package extends the shared Latin American internal type list with two values:

| Value | Meaning |
|---|---|
| Purchase liquidation | A document the buyer issues on behalf of a seller who cannot issue one. |
| Withhold | A withholding certificate, which is a legal document with its own class and numbering. |

Each document type also carries a marker that switches the three-part number check on.

---

## 9. Counterpart validation

The identification number is validated against its identification class. A class that must be ten digits long raises:

```
If your identification type is <class>, it must be 10 digits
```

The contact form shows the reason inline in a derived field rather than blocking the save, so an incomplete contact can be saved and completed later.

---

## 10. Stock

The stock package adds the carrier and the transport document data to the delivery note, so that the printed note carries the information the administration requires when goods move.

---

## 11. Acceptance scenarios

**Given** a company in Ecuador with no accounting,
**when** the Ecuadorian template is loaded,
**then** 501 accounts and 112 account groups exist, the rounding method is Round per Line, the sales journal is named `001-001 Facturas de cliente` with the emission entity `001` and the emission point `001`, and the purchase journal's default account is `52022816`.

**Given** a service bill of 1,000.00 with a 15 percent value-added tax, a 10 percent income tax withholding and a 70 percent value-added tax withholding,
**when** the totals are computed,
**then** the value-added tax is 150.00, the income tax withheld is 100.00, the value-added tax withheld is 105.00 and the amount paid to the supplier is 945.00.

**Given** a document number typed as `1-1-123`,
**when** the invoice is saved,
**then** the save is refused with the three-part format message.

**Given** a contact whose identification class must be ten digits and whose number is nine digits,
**when** the contact is opened,
**then** the validation message is shown inline and the contact can still be saved.
