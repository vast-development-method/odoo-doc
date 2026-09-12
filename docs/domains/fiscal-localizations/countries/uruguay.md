# Uruguay

The Uruguayan package supplies one chart of accounts template of 166 accounts with eight depreciation models, 12 taxes covering the two value-added tax rates in both a tax-excluded and a tax-included form plus an exempt tax and a reduced tax, four tax groups, two fiscal positions, the document type mechanism with a two-part numbering format, and the identification type catalogue with the administration's codes. Both the sales journal and the purchase journal issue legal documents.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `uy` |
| Display name | Uruguayan Generic Chart of Accounts |
| Parent template | none |
| Fiscal country | Uruguay |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `1111` |
| Cash account code prefix | `1112` |
| Transfer account code prefix | `11120` |
| Default point of sale receivable account | `11307` |
| Gain exchange rate account | `4302` |
| Loss exchange rate account | `5302` |
| Cash discount write-off loss account | `5303` |
| Cash discount write-off gain account | `4303` |
| Default sale tax | 22 percent value-added tax, sale |
| Default purchase tax | 22 percent value-added tax, purchase |
| Deferred expense account | `11407` |
| Deferred revenue account | `21321` |
| Income account | `4102` |
| Expense account | `5100` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `11704` |
| Receivable account recorded as a Contact default | `11300` |
| Payable account recorded as a Contact default | `21100` |

The stock valuation account `11704` names its stock variation account (`5401`).

### 1.2 Journals

| Symbolic identifier | Name | Code | Uses documents | Separate credit note numbering |
|---|---|---|---|---|
| `sale` | Sales | `0001` | yes | off |
| `purchase` | Purchases | `0002` | yes | off |

### 1.3 Extra load step

After the load, the company's own contact is given the national taxpayer identification class, because that class is the one the country treats as the tax identification number.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 166 |
| Code lengths present | 2, 4 and 5 characters, padded to 6 on load |
| Account groups shipped | none |
| Translations shipped | Spanish for names and descriptions |
| Depreciation models shipped | 8 |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 38 |
| Fixed Asset | 12 |
| Receivable | 3 |
| Equity | 12 |
| Expense | 49 |
| Income | 10 |
| Current Liability | 26 |
| Payable | 16 |

Several accounts ship with a default tax already attached, so that a line posted to such an account proposes the right tax without the operator choosing it.

**Depreciation models.** Eight models, each naming the asset account, the accumulated depreciation account, the depreciation expense account, the number of periods and the period length in months.

---

## 3. Taxes

12 taxes, all carrying the tax category `vat`, which is what the statutory financial reports group by.

| Name | Printed label | Rate | Scope | Price inclusion | Tax group | Fiscal position |
|---|---|---|---|---|---|---|
| 22% | Value-added tax Sales (22%) | 22 | sale | tax excluded | Value-added tax 22% | Local, Uruguay |
| 10% | Value-added tax Sales (10%) | 10 | sale | tax excluded | Value-added tax 10% | Local, Uruguay |
| 0% EXEMPT | Value-added tax Exempt Sales | 0 | sale | tax excluded | Exempt | Export |
| 22% | Value-added tax Purchases (22%) | 22 | purchase | tax excluded | Value-added tax 22% | Local, Uruguay |
| 10% | Value-added tax Purchases (10%) | 10 | purchase | tax excluded | Value-added tax 10% | Local, Uruguay |
| 0% EXEMPT | Purchases Exempt from value-added tax | 0 | purchase | tax excluded | Exempt | Export |
| 22% included | Value-added tax Included Sales (22%) | 22 | sale | tax included | Value-added tax 22% | Local, Uruguay |
| 10% included | Value-added tax Included Sales (10%) | 10 | sale | tax included | Value-added tax 10% | Local, Uruguay |
| 22% included | Value-added tax Included Purchases (22%) | 22 | purchase | tax included | Value-added tax 22% | Local, Uruguay |
| 10% included | Value-added tax Included Purchases (10%) | 10 | purchase | tax included | Value-added tax 10% | Local, Uruguay |
| `20% Reduced VAT` | Sales Reduced Value-added tax | 20 | sale | tax excluded | Other value-added tax | Local, Uruguay |
| `20% Reduced VAT` | Purchase Reduced Value-added tax | 20 | purchase | tax excluded | Other value-added tax | Local, Uruguay |

**Worked example of the tax-included form.** A retail price of 1,220.00 carrying the 22 percent included tax.

```
base = round(1,220.00 ÷ 1.22, 2) = 1,000.00
tax  = 1,220.00 − 1,000.00       =   220.00
```

The tax-included variants exist because retail prices are quoted with the tax in them and the invoice must show the same figure the customer saw.

---

## 4. Tax groups

| Group | Country |
|---|---|
| Value-added tax 10% | Uruguay |
| Value-added tax 22% | Uruguay |
| Exempt | Uruguay |
| Other value-added tax | Uruguay |

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Country |
|---|---|---|---|
| 10 | Local, Uruguay | yes | Uruguay |
| 20 | Export | yes | none |

---

## 6. Document types and numbering

The package uses the shared Latin American document type mechanism on both the sales and the purchase journal. The document number has two parts separated by a dash: at most two letters in the first part and seven digits in the second.

```
<document number> is not a valid value for <document type>.
The document number must be entered with a maximum of 2 letters for the first part and 7 numbers for the second part.
```

---

## 7. Identification types

The package extends the shared Latin American identification type catalogue with the administration's code, so that the code the electronic document carries can be derived from the class.

| Class | Administration code |
|---|---|
| National taxpayer number | `2` |
| National identity document | `3` |
| Foreign identity document | `4` |
| Passport | `5` |
| Foreign taxpayer number | `6` |
| Other | `7` |

---

## 8. Point of sale

The point of sale package adds the document class and the document number to the receipt, so that a till issues a legal document with its own numbering rather than an internal receipt number.

---

## 9. Acceptance scenarios

**Given** a company in Uruguay with no accounting,
**when** the Uruguayan template is loaded,
**then** 166 accounts and eight depreciation models exist, the sales journal is coded `0001` and the purchase journal `0002`, both use documents, and the company's own contact carries the national taxpayer identification class.

**Given** a retail price of 1,220.00 carrying the 22 percent tax-included tax,
**when** the totals are computed,
**then** the base is 1,000.00 and the tax is 220.00.

**Given** a document number typed as `ABC-1234567`,
**when** the invoice is saved,
**then** the save is refused, because the first part may contain at most two letters.

**Given** a customer outside Uruguay,
**when** the invoice is prepared,
**then** the fiscal position "Export" is detected and the exempt tax applies.
