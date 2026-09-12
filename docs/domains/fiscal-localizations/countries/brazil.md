# Brazil

The Brazilian package supplies one chart of accounts template of 1,085 accounts built on the national standard plan, 323 taxes covering the seventeen Brazilian tax families of both the current and the reformed regimes, 160 tax groups, four fiscal positions that select the applicable rate from the geography of the transaction, a city catalogue whose members are identified by postal code intervals, the three national registration numbers on every contact, a document series carried on every sales journal, and the instant payment key types needed to print a payment visual code.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `br` |
| Display name | Brazil |
| Parent template | none |
| Fiscal country | Brazil |
| Account code length | 6 characters, but the shipped codes are already 13 or 16 characters and are therefore not padded |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `1.01.01.02.00` |
| Cash account code prefix | `1.01.01.01.00` |
| Transfer account code prefix | `1.01.01.12.00` |
| Default point of sale receivable account | `1.01.01.04.02` Cash in Transit (point of sale) |
| Gain exchange rate account | Exchange variation income |
| Loss exchange rate account | Exchange variation expense |
| Cash discount write-off loss account | Discounts granted |
| Cash discount write-off gain account | Discounts obtained |
| Default sale tax | 17 percent state circulation tax, internal sale |
| Default purchase tax | 17 percent state circulation tax, internal purchase |
| Income account | Revenue from sales of goods |
| Expense account | Cost of goods sold |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `1.01.03.04.01` Inventory of goods for resale |
| Receivable account recorded as a Contact default | `1.01.01.04.01` Cash in Transit |
| Payable account recorded as a Contact default | `2.01.01.03.01` Suppliers |

The inventory account names its stock expense account (Cost of goods sold) and its stock variation account (Variation of inventory of goods).

### 1.2 Sales journal override

| Field | Value |
|---|---|
| Series number | `1` |
| Separate credit note numbering | off |

A second series requires a second sales journal: the series is a property of the journal, not of the document.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 1,085 |
| Code lengths | 13 characters (`N.NN.NN.NN.NN`) and 16 characters (`N.NN.NN.NN.NN.NN`) |
| Account groups shipped | none; the hierarchy is expressed by the dotted code itself |
| Translations shipped | Portuguese for every account name |

**Account type distribution.**

| Account type | Count |
|---|---|
| Bank and Cash | 2 |
| Current Asset | 192 |
| Fixed Asset | 41 |
| Non-current Asset | 233 |
| Prepayments | 4 |
| Receivable | 3 |
| Equity | 46 |
| Expense | 208 |
| Depreciation | 9 |
| Cost of Revenue | 7 |
| Income | 17 |
| Other Income | 100 |
| Current Liability | 115 |
| Non-current Liability | 99 |
| Payable | 9 |

**First accounts of the plan.**

| Code | Name | Type |
|---|---|---|
| `1.01.01.01.01` | Cash, Head Office | Bank and Cash |
| `1.01.01.01.02` | Cash, Branches | Bank and Cash |
| `1.01.01.02.01` | Bank accounts, in the country | Current Asset |
| `1.01.01.02.02` | Bank accounts, abroad | Current Asset |
| `1.01.01.04.01` | Cash in Transit | Receivable |
| `1.01.01.04.02` | Cash in Transit (point of sale) | Receivable |
| `1.01.01.05.01` | Securities for trading, measured at fair value | Current Asset |
| `1.01.03.04.01` | Inventory of goods for resale | Current Asset |
| `1.02.01.08.02` | Recoverable state circulation tax | Current Asset |
| `2.02.01.10.03` | Taxes payable | Current Liability |

The two accounts `2.02.01.10.03` and `1.02.01.08.02` are the payable and receivable counterparts of **every one** of the 160 tax groups, so the closing entry consolidates all seventeen tax families into one payable line and one receivable line.

---

## 3. Taxes

323 taxes are shipped, always in pairs: every tax exists once for sales and once for purchases, and, for most families, once with the tax excluded from the price and once with it included.

### 3.1 Tax families

| Family | Full name in words | Level | Taxes shipped |
|---|---|---|---|
| State circulation tax | Tax on the circulation of goods and on transport and communication services | state | 82 |
| Contribution on goods and services | The reformed federal consumption contribution | federal | 40 |
| Tax on goods and services | The reformed shared consumption tax, in its state and municipal shares | state and municipal | 72 |
| Social contribution on turnover | Contribution for the financing of social security | federal | 20 |
| Social integration contribution | Contribution to the social integration programme | federal | 20 |
| Manufactured products tax | Tax on manufactured products | federal | 12 |
| Approximate tax burden | The informative breakdown of the tax burden printed on a consumer receipt, split into federal, state and municipal shares | informative | 12 |
| Social security on turnover | Employer contribution on gross turnover | federal | 8 |
| Social contribution on net profit | Contribution on net profit | federal | 9 |
| Selective tax | The reformed excise tax on goods harmful to health or the environment | federal | 8 |
| Service tax | Municipal tax on services | municipal | 10 |
| Income tax withheld | Income tax withheld at source | federal | 9 |
| Import tax | Tax on imports | federal | 6 |
| Social security contribution | Employer and employee social security | federal | 10 |
| Financial operations tax | Tax on credit, exchange and insurance operations | federal | 4 |
| Corporate income tax | Corporate income tax | federal | 1 |

### 3.2 Tax fields specific to the country

| Field on Tax | Type | Default | Meaning |
|---|---|---|---|
| Discount this tax in the price | boolean | false | The tax is included in the price the customer sees and is therefore deducted from the base of the taxes that follow. Set for the state circulation tax and the two social contributions; not set for the manufactured products tax, which is added on top. |
| Base reduction | decimal, no decimal places declared | 0 | A decimal fraction between 0 and 1 by which the base is reduced before the rate is applied. |
| Substitution margin | decimal, no decimal places declared | 0 | A decimal fraction by which the base is increased to estimate the downstream retail price, used by the tax-substitution regime. |

**Base reduction formula.**

```
reduced_base = round(base × (1 − base_reduction), 2)
tax          = round(reduced_base × rate ÷ 100, 2)
```

**Worked example.** A base of 1,000.00, a base reduction of 0.2657 and a rate of 18 percent.

```
reduced_base = round(1,000.00 × 0.7343, 2) = 734.30
tax          = round(734.30 × 0.18, 2)     = 132.17
```

**Substitution margin formula.**

```
substitution_base = round(base × (1 + substitution_margin), 2)
substituted_tax   = round(substitution_base × rate ÷ 100, 2) − tax_already_charged
```

**Worked example.** A base of 1,000.00, a substitution margin of 0.40, a rate of 18 percent, tax already charged 180.00.

```
substitution_base = round(1,000.00 × 1.40, 2) = 1,400.00
substituted_tax   = round(1,400.00 × 0.18, 2) − 180.00 = 252.00 − 180.00 = 72.00
```

**Tax included in its own base.** Several Brazilian taxes are legally computed on a base that already contains them. Such a tax is shipped with the price inclusion set to "tax included", and the base is grossed up:

```
grossed_base = round(base ÷ (1 − rate ÷ 100), 2)
tax          = round(grossed_base × rate ÷ 100, 2)
```

**Worked example.** A base of 1,000.00 and a rate of 18 percent included.

```
grossed_base = round(1,000.00 ÷ 0.82, 2) = 1,219.51
tax          = round(1,219.51 × 0.18, 2) =   219.51
```

### 3.3 The state circulation tax and the four rates

The state circulation tax is the only family whose rate depends on the geography of the transaction, and the four fiscal positions exist to select it.

| Rate | When it applies | Tax names | Fiscal position that selects it |
|---|---|---|---|
| 17 percent | Both parties in the same state | 17% ICMS I (internal) and 17% ICMS E (external) | Internal, within one state |
| 7 percent | A seller in the South or Southeast selling to the North, Northeast or Midwest | 7% ICMS E | South and Southeast to North, Northeast and Midwest |
| 12 percent | Any other interstate transaction | 12% ICMS E | Interstate |
| 0 percent | A foreign counterpart | 0% ICMS I and 0% ICMS E | Foreign |

A further pair, 0% ICMS S, expresses the substitution regime, in which the tax on the whole downstream chain is charged once at the top; those two taxes carry the price-discount marker set to false.

**Worked example.** A seller in São Paulo (Southeast) invoices a buyer in Bahia (Northeast) for 10,000.00.

```
fiscal position = South and Southeast to North, Northeast and Midwest
tax             = 7% ICMS E
amount          = round(10,000.00 × 0.07, 2) = 700.00
```

The same invoice to a buyer in São Paulo gives the Internal position, the 17 percent tax and 1,700.00.

### 3.4 Manufactured products tax

| Rate | Tax names | Fiscal position |
|---|---|---|
| 10 percent | 10% IPI, sale and purchase | Internal |
| 0 percent | 0% IPI, sale and purchase | Foreign |

The manufactured products tax carries the price-discount marker false: it is added on top of the price rather than being included in it.

**Worked example.** A price of 1,000.00 carrying the 17 percent circulation tax (included, discounted in price) and the 10 percent manufactured products tax (added).

```
circulation tax = round(1,000.00 × 0.17, 2) =   170.00   (already inside the 1,000.00)
manufactured products tax = round(1,000.00 × 0.10, 2) = 100.00
invoice total = 1,000.00 + 100.00 = 1,100.00
```

### 3.5 The reformed taxes

The reformed regime replaces the circulation tax, the service tax and the two federal contributions by two taxes and one excise. The package ships all three, each in a wide matrix of situations, so that a company can run the old and the new regimes in parallel during the transition.

| Reformed tax | Situations shipped, each as an excluded and an included variant |
|---|---|
| Contribution on goods and services | ordinary, deferred, government compensation, presumed credit, presumed credit with social contribution, presumed withholding, reduced rate, returned, social works, transfer of credit |
| Tax on goods and services | the same ten situations, twice: once for the state share and once for the municipal share |
| Selective tax | ordinary and reduced, excluded and included |

Each situation has its own tax group, which is why 160 groups exist.

### 3.6 Approximate tax burden

Twelve informative taxes with a zero effect on the amount due, whose only purpose is to print, on a consumer receipt, the estimated share of the price that is tax. They are split into a federal, a state and a municipal share, each for goods and for services, each excluded and included.

---

## 4. Tax groups

160 groups, all with the country Brazil, all wired to the payable account `2.02.01.10.03` and the receivable account `1.02.01.08.02`. The group name always states the family and either a rate (for the current-regime taxes) or a situation (for the reformed taxes), for example "ICMS 17%", "PIS 0.65%", "COFINS 3%", "Contribution on goods and services, deferred, excluded".

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax identification number | Country | Interstate type |
|---|---|---|---|---|---|
| 1 | Internal, within one state | yes | yes | Brazil | Internal |
| 2 | Foreign | yes | no | none | none |
| 3 | South and Southeast to North, Northeast and Midwest | yes | yes | Brazil | South or Southeast selling to North, Northeast or Midwest |
| 4 | Interstate | yes | yes | Brazil | Other interstate |

Each position also carries a list of account mappings, so that a foreign sale is booked to the export revenue account rather than to the domestic revenue account.

**Interstate type field.** The fiscal position carries an extra selection with the three values above. It is what a rate table or a report reads to decide which published rate matrix applies, without having to compare the two states again.

**Detection.** The four positions rely on the framework predicates of [../workflows.md](../workflows.md) section 8: the Internal position names Brazil and requires a tax identification number, and the two interstate positions are distinguished by the country subdivision lists attached to them.

---

## 6. Cities and postal code ranges

A Brazilian address names a city rather than typing one, and the city is found from the postal code.

| Entity | Fields |
|---|---|
| City | the platform's city, extended with a list of postal code ranges and a derived text rendering of them for the checkout form |
| Brazil City Postal Code Range | city, start, end |

**Uniqueness.** The start of a range is unique and the end of a range is unique across the whole table, so no two cities may claim overlapping intervals.

```
The "from" zip must be unique
The "to" zip must be unique.
```

**Format and ordering validation.**

```
Invalid zip range format: <start> <end>. It should follow this format: 01000-001
Start should be less than end: <start> <end>
```

**Worked example.** A postal code of `01310-100` falls inside the range `01000-001` to `01599-999`, which belongs to the city of São Paulo, so the address is completed with that city and its state.

---

## 7. Registration numbers

| Location | Field | Meaning |
|---|---|---|
| Contact | State tax identification number | The state registration, 9 to 14 digits. Each state publishes its own format and check rule; the package stores the number without imposing one national rule. |
| Contact | Municipal tax identification number | The municipal registration. |
| Contact | Free trade zone registration number | The registration with the free trade zone authority, which grants an exemption. |
| Company | State tax identification number | Mirrored from the company contact. |
| Company | Municipal tax identification number | Mirrored from the company contact. |

The national taxpayer number itself is the platform's tax identification number field, validated by the national checker for both the individual and the legal-entity forms.

---

## 8. Instant payment keys

A bank account carries a key type used to build the instant payment visual code printed on the invoice.

| Key type | Validation | Message on failure |
|---|---|---|
| Electronic mail address | must be a valid address | `<value> is not a valid email.` |
| Mobile number | must start with the country calling code, then a two-digit area code, then a nine-digit number | `The mobile number <value> is invalid. It must start with +55, contain a 2 digit territory or state code followed by a 9 digit number.` |
| National taxpayer number, individual or legal entity | must be a valid number without punctuation | `<value> is not a valid CPF or CNPJ (don't include periods or dashes).` |
| Random key | must match the thirty-six character grouped form | `The random key <value> is invalid, the format looks like this: 71d6c6e1-64ea-4a11-9560-a10870c40ca2` |
| Any other type | refused | `The proxy type must be Email Address, Mobile Number, CPF/CNPJ (BR) or Random Key (BR) for Pix code generation.` |

---

## 9. Documents and numbering

| Element | Behavior |
|---|---|
| Series | Carried on the sales journal. Printed on the document and quoted in every declaration. |
| Number | Produced by the journal's own numbering series. |
| Separate credit note numbering | Switched off by the template, so credit notes consume the same series as invoices. |
| Reversal wizard | Extended so that a credit note produced from an invoice inherits the series. |

---

## 10. Online checkout and sales

| Element | Behavior |
|---|---|
| Checkout address form | Adds the city selector driven by the postal code, the state selector, and the three registration numbers. A postal code that matches no range leaves the city empty and the customer chooses. |
| Sales order | Carries the fiscal position derived from the delivery address, so that the quotation already shows the correct rate. |
| Printed quotation and invoice | Show the tax breakdown per family and the approximate tax burden line. |

---

## 11. Acceptance scenarios

**Given** a company in Brazil with no accounting,
**when** the Brazilian template is loaded,
**then** 1,085 accounts exist, the bank prefix is `1.01.01.02.00`, the default sales tax is the 17 percent internal circulation tax, 160 tax groups exist and all of them point at the same payable and receivable accounts.

**Given** a seller in São Paulo and a buyer in Bahia,
**when** an invoice is prepared,
**then** the fiscal position "South and Southeast to North, Northeast and Midwest" is detected and the circulation tax on the line is the 7 percent external tax.

**Given** a base of 1,000.00, a base reduction of 0.2657 and a rate of 18 percent,
**when** the tax is computed,
**then** the reduced base is 734.30 and the tax is 132.17.

**Given** a base of 1,000.00 and a rate of 18 percent declared as included in its own base,
**when** the tax is computed,
**then** the grossed base is 1,219.51 and the tax is 219.51.

**Given** a city whose postal code range is `01000-001` to `01599-999`,
**when** a second range starting at `01000-001` is created for another city,
**then** the creation is refused with `The "from" zip must be unique`.

**Given** a bank account whose key type is the mobile number and whose key is `11987654321`,
**when** the account is saved,
**then** the save is refused with the mobile number message, because the country calling code is missing.

**Given** a sales journal carrying the series `1`,
**when** a second series is needed,
**then** a second sales journal is created and given the second series, because the series belongs to the journal.
