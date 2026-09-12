# Peru

The Peruvian package supplies one chart of accounts template of 1,228 accounts built on the general business accounting plan, 83 account groups, 18 taxes covering the general sales tax, the excise tax, the plastic bag tax, the export and exemption categories and the free-of-charge mechanism, 15 tax groups, two fiscal positions, the district as a third administrative level of every address, and the document type mechanism shared across Latin America.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `pe` |
| Display name | Peru |
| Parent template | none |
| Fiscal country | Peru |
| Account code length | 7 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `1041` |
| Cash account code prefix | `1031` |
| Transfer account code prefix | `1051` |
| Default point of sale receivable account | `1215` |
| Gain exchange rate account | `776` |
| Loss exchange rate account | `676` |
| Cash discount write-off loss account | `675` |
| Cash discount write-off gain account | `775` |
| Default sale tax | 18 percent general sales tax, sale |
| Default purchase tax | 18 percent general sales tax, purchase |
| Income account | `70121` |
| Expense account | `6329` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `20111` |
| Receivable account recorded as a Contact default | `1213` |
| Payable account recorded as a Contact default | `4212` |
| Stock valuation account recorded as a company property | `20111` |

The stock valuation account `20111` names its stock expense account (`6111`) and its stock variation account (`69121`).

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 1,228 |
| Code lengths present | 3, 4, 5 and 6 characters, padded to 7 on load |
| Account groups shipped | 83 |
| Translations shipped | Spanish |

**Account type distribution.**

| Account type | Count |
|---|---|
| Bank and Cash | 12 |
| Current Asset | 169 |
| Fixed Asset | 180 |
| Non-current Asset | 145 |
| Prepayments | 6 |
| Receivable | 99 |
| Equity | 47 |
| Expense | 165 |
| Depreciation | 104 |
| Cost of Revenue | 28 |
| Income | 34 |
| Other Income | 79 |
| Current Liability | 77 |
| Non-current Liability | 18 |
| Payable | the remainder |

**Account groups.** The 83 groups are the ten classes of the national plan and their first sub-levels.

| Prefix | Name |
|---|---|
| `0` | Memorandum accounts |
| `1` | Available and receivable assets |
| `2` | Realisable assets |
| `3` | Available and payable assets |
| `4` | Liabilities |
| `5` | Net worth |
| `6` | Expenses by nature |
| `7` | Income |
| `8` | Intermediate management balances |
| `9` | Analytical cost accounting |

**Padding example.** The account whose template code is `1213` is stored as `1213000`, because the template pads to seven characters.

---

## 3. Taxes

18 taxes. Every tax carries an administration tax code and an international category code, which are what the electronic document transmits.

### 3.1 Administration tax codes

| Code | Meaning |
|---|---|
| `1000` | General sales tax |
| `1016` | Tax on the sale of paddy rice |
| `2000` | Selective excise tax |
| `7152` | Plastic bag tax |
| `9995` | Export |
| `9996` | Free of charge |
| `9997` | Exonerated |
| `9998` | Unaffected |
| `9999` | Other taxes |

### 3.2 International category codes

| Code | Meaning |
|---|---|
| `E` | Exempt from tax |
| `G` | Free export item, tax not charged |
| `O` | Services outside the scope of tax |
| `S` | Standard rate |
| `Z` | Zero-rated goods |

### 3.3 Shipped taxes

| Name | Printed label | Rate | Scope | Tax code | Category | Tax group | Fiscal position |
|---|---|---|---|---|---|---|---|
| `VAT 18%` | General sales tax 18% | 18 | sale | `1000` | `S` | General sales tax | Local Peru |
| `VAT 18%` | General sales tax 18% | 18 | purchase | `1000` | `S` | General sales tax | Local Peru |
| `VAT 18% G NG` | General sales tax 18%, taxed and untaxed | 18 | purchase | `1000` | `S` | General sales tax, taxed and untaxed | Local Peru |
| `VAT 18% NG` | General sales tax 18%, untaxed | 18 | purchase | `1000` | `S` | General sales tax, untaxed | Local Peru |
| 3% IGV Withholding | 3% general sales tax withholding | −3 | sale | none | none | General sales tax withholding | Local Peru |
| 0% Exo | Exonerated 0% | 0 | sale and purchase | `9997` | `E` | Exonerated | Local Peru |
| 0% Ina | Unaffected 0% | 0 | sale and purchase | `9998` | `Z` | Unaffected | Local Peru |
| 0% Gra | Free of charge 0% | 0 | sale and purchase | `9996` | `E` | Free of charge | Local Peru |
| 0% Exp | Export 0% | 0 | sale and purchase | `9995` | `S` | Export | Foreign, Export |
| 0% ISC | Selective excise 0% | 0 | sale | `2000` | `S` | Selective excise | Local Peru |

### 3.4 The free-of-charge mechanism

A good given away free must still be declared with its notional value and its notional tax, and the two must cancel each other in the accounts. Three taxes and one group tax implement this.

| Name | Rate | Scope | Purpose |
|---|---|---|---|
| −Base | −100 | none | Removes the notional base from the amount due. |
| 18% Free | 18 | none | Charges the notional tax. |
| −18% Free | −18 | none | Removes the notional tax from the amount due, booking it as an expense. |
| 18% Free Group | group | sale | The tax the user chooses; its children are the three above. |

**Worked example.** A free sample whose notional value is 500.00.

```
notional base          =  500.00   reported on the declaration
notional tax           =   90.00   reported on the declaration
base removed           = −500.00
tax removed            =  −90.00   booked as an expense
amount due from customer = 0.00
```

### 3.5 The excise tax computation kind

The excise tax carries a computation kind that names how the administration wants it computed:

| Value | Meaning |
|---|---|
| `01` | System to value: a percentage of the price. |
| `02` | Application of a fixed amount: an amount per unit. |
| `03` | Retail price system: a percentage of a published retail price rather than of the invoice price. |

---

## 4. Tax groups

15 groups, all with the country Peru. Two of them carry an ordering number that pushes them to the end of the document subtotals.

| Group | Ordering |
|---|---|
| General sales tax | 0 |
| General sales tax withholding | 0 |
| General sales tax, taxed and untaxed | 0 |
| General sales tax, untaxed | 0 |
| Tax on the sale of paddy rice | 0 |
| Selective excise tax | 0 |
| Export | 0 |
| Free of charge | 0 |
| Exonerated | 0 |
| Unaffected | 0 |
| Others | 0 |
| Plastic bag tax | 0 |
| Detraction | 100 |
| Retention | 100 |
| Free invoice | 200 |

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Country |
|---|---|---|---|
| 10 | Local Peru | yes | Peru |
| 20 | Foreign, Export | yes | none |

---

## 6. Addresses

A Peruvian address has four levels: the country, the country subdivision, the city and the **district**.

| Entity | Field | Meaning |
|---|---|---|
| City | Code | The administration's code for the city. |
| Peru District | name, city, code, derived country, derived subdivision | The third administrative level. |
| Contact | District | The district of the address. |
| Contact | District name | Derived from the district. |

The checkout form and the contact form load the cities of a subdivision and the districts of a city through the two request endpoints listed in [../interfaces.md](../interfaces.md) section 4.

---

## 7. Reference data

| Entity | Field | Meaning |
|---|---|---|
| Bank | Administration code | The code the administration assigns to a banking institution. |
| Identification type | Administration tax code | The administration's code for the identification class. |

---

## 8. Documents

The package uses the shared Latin American document type mechanism together with the debit note capability, because a Peruvian seller corrects upwards with a debit note. The classes include the invoice, the receipt, the credit note, the debit note and the export invoice, each with its own numbering series per journal.

---

## 9. Acceptance scenarios

**Given** a company in Peru with no accounting,
**when** the Peruvian template is loaded,
**then** 1,228 accounts and 83 account groups exist, the account code length is 7, the bank prefix is `1041`, the cash prefix is `1031`, the transfer prefix is `1051`, and 15 tax groups exist.

**Given** an account whose template code is `1213`,
**when** the template is loaded,
**then** the stored code is `1213000`.

**Given** a free sample whose notional value is 500.00 carrying the free-of-charge group tax,
**when** the totals are computed,
**then** the notional base of 500.00 and the notional tax of 90.00 are reported, the amount due from the customer is 0.00 and 90.00 is booked as an expense.

**Given** a customer whose address is in Peru,
**when** the invoice is prepared,
**then** the fiscal position "Local Peru" is detected and the general sales tax at 18 percent applies.

**Given** a customer outside Peru,
**when** the invoice is prepared,
**then** the fiscal position "Foreign, Export" is detected and the export tax at 0 percent applies, carrying the tax code `9995`.
