# Argentina

The Argentine package supplies three chained chart of accounts templates, one per tax responsibility regime, the value-added tax scale with its administration codes, the twenty-four provincial turnover tax perceptions and withholdings, the responsibility type catalogue that decides which document letter may be issued to whom, the point of sale numbering carried on the journal, the document type mechanism shared with the rest of Latin America, a withholding at payment framework with progressive scales and per-counterpart authorisations, a delivery guide numbering mechanism with an authorisation code and a numbering range, and a checkout extension that shows prices with and without national taxes.

---

## 1. Templates

Three templates form a chain. The most specific one, the registered taxpayer chart, inherits from the exempt chart, which inherits from the base chart.

| Template code | Display name | Parent | Ordering | Account code length |
|---|---|---|---|---|
| `ar_base` | Generic Chart of Accounts Argentina Single Taxpayer / Basis | none | 1 | 12 |
| `ar_ex` | Argentine Generic Chart of Accounts for Exempt Individuals | `ar_base` | 2 | 12 |
| `ar_ri` | Argentine Generic Chart of Accounts for Registered Accountants | `ar_ex` | 0 | 12 |

Because the registered taxpayer chart has the lowest ordering number, it is the first entry of the selection list for Argentina and is therefore the guessed template.

### 1.1 Company-level values

All three templates write the same liquidity prefixes and exchange accounts.

| Company field | Value |
|---|---|
| Fiscal country | Argentina |
| Account code length used for padding | 12 |
| Bank account code prefix | `1.1.1.02.` |
| Cash account code prefix | `1.1.1.01.` |
| Transfer account code prefix | `6.0.00.00.` |
| Default point of sale receivable account | `1.1.3.01.020` Sales receivables (point of sale) |
| Gain exchange rate account | Exchange differences |
| Loss exchange rate account | Exchange differences |

The base template additionally writes:

| Company field | Value |
|---|---|
| Income account | Sales of goods |
| Expense account | Purchases of goods |
| Stock journal | the inventory valuation journal |
| Stock valuation account | Goods for resale |
| Receivable account recorded as a Contact default | Trade receivables |
| Payable account recorded as a Contact default | Trade payables |

The registered taxpayer template additionally writes:

| Company field | Value |
|---|---|
| Default sale tax | Value-added tax 21% (sale) |
| Default purchase tax | Value-added tax 21% (purchase) |

The account `Goods for resale` names its stock expense account (Purchases of goods) and its stock variation account (Variation of goods for resale), so that perpetual inventory posts correctly.

### 1.2 Sales journal override

The base template overrides the sales journal that every template creates:

| Field | Value |
|---|---|
| Name | Ventas Preimpreso (pre-printed sales) |
| Code | `0001` |
| Point of sale number | 1 |
| Point of sale address | the company's own contact |
| Point of sale system | `II_IM` (pre-printed or imported invoice) |
| Separate credit note numbering | off |

---

## 2. Chart of accounts

| Template | Accounts contributed | Cumulative total when loaded |
|---|---|---|
| `ar_base` | 228 | 228 |
| `ar_ex` | 63 | 291 |
| `ar_ri` | 8 | 299 |

Because delimited files are merged generic first and child last, loading `ar_ri` reads the base file, then the exempt file, then the registered taxpayer file, and the union is 299 accounts.

**Account groups.** 58 groups, contributed by the base template only, each with a starting and an ending prefix. Examples: `1..` Active, `1.1..` Current Assets, `1.1.1..` Banks and savings, `1.1.4..` Tax credits, `1.1.4.04..` Value-added tax credits, `1.1.4.05..` Earnings tax credits.

**Account type distribution of the base template.**

| Account type | Count |
|---|---|
| Bank and Cash | 1 |
| Current Asset | 87 |
| Fixed Asset | 10 |
| Receivable | 3 |
| Equity | 9 |
| Current Year Earnings | 1 |
| Expense | 78 |
| Depreciation | 5 |
| Income | 2 |
| Other Income | 6 |
| Current Liability | 9 |
| Non-current Liability | 3 |
| Payable | 14 |

**Structure of the tax accounts.** The chart has one account pair per province for the turnover tax, one for the value-added tax, one for the earnings tax and one for the social security tax, on both the incurred and the applied side.

| Code | Name | Contributed by |
|---|---|---|
| `1.1.1.02.010` | Third Party Checks | base |
| `1.1.1.02.020` | Rejected Third Party Checks | base |
| `1.1.3.01.010` | Sales receivables | base |
| `1.1.3.01.020` | Sales receivables (point of sale) | base |
| `1.1.4.01.010` | Withholding and perception, municipal tax | base |
| `1.1.4.02.010` | Balance in favour, turnover tax, City of Buenos Aires | base |
| `1.1.4.02.020` | Turnover tax withholding incurred, City of Buenos Aires | base |
| `1.1.4.02.030` | Turnover tax perception incurred, City of Buenos Aires | base |
| `1.1.4.04.010` | Value-added tax credit | registered taxpayer |
| `1.1.4.04.020` | Value-added tax perception incurred | registered taxpayer |
| `1.1.4.04.030` | Value-added tax withholding incurred | registered taxpayer |
| `1.1.4.04.040` | Value-added tax technical balance | registered taxpayer |
| `1.1.4.04.050` | Value-added tax unrestricted balance | registered taxpayer |
| `1.1.4.05.010` | Earnings advance | exempt |
| `1.1.4.05.020` | Earnings perception incurred | exempt |
| `1.1.4.05.030` | Earnings withholding incurred | exempt |
| `1.1.4.05.040` | Earnings balance | exempt |
| `2.1.3.02.020` | Withholding return to be paid | exempt |
| `2.1.3.02.030` | Turnover tax withholding applied, City of Buenos Aires | exempt |
| `2.1.3.03.010` | Value-added tax debit | registered taxpayer |
| `2.1.3.03.020` | Value-added tax balance payable | registered taxpayer |
| `2.1.3.03.050` | Value-added tax instalment plan to be paid | registered taxpayer |

The exempt template contributes 54 current liability accounts, which are the per-province applied withholding and perception accounts; a company under the exempt regime withholds on behalf of the administration but does not charge value-added tax.

---

## 3. Taxes

### 3.1 Value-added tax, registered taxpayer chart

Twenty taxes. Every ordinary tax names the domestic fiscal position, so a domestic document keeps its tax.

| Name | Printed label | Rate | Scope | Tax group | Active on install |
|---|---|---|---|---|---|
| `VAT 21%` | Value-added tax 21% | 21 | sale and purchase | Value-added tax 21% | yes |
| `VAT 10.5%` | Value-added tax 10.5% | 10.5 | sale and purchase | Value-added tax 10.5% | yes |
| `VAT 27%` | Value-added tax 27% | 27 | sale and purchase | Value-added tax 27% | yes |
| `VAT 2.5%` | Value-added tax 2.5% | 2.5 | sale and purchase | Value-added tax 2.5% | no |
| `VAT 5%` | Value-added tax 5% | 5 | sale and purchase | Value-added tax 5% | no |
| `VAT 0%` | Value-added tax 0% | 0 | sale and purchase | Value-added tax 0% | yes |
| 0% EXEMPT | Value-added tax Exempt | 0 | sale and purchase | Value-added tax Exempt | yes |
| 0% NT | Value-added tax Not Taxed | 0 | sale and purchase | Value-added tax Untaxed | yes |
| 0% NA | Value-added tax Not Applicable | 0 | sale (archived) and purchase | Value-added tax Not Applicable | purchase only |
| `Perc VAT` | Value-added tax Perception | fixed amount, 1 unit | purchase | Value-added tax Perception | yes |
| `VAT 20%` | Value-added tax Additional 20% | 20 | purchase | Value-added tax Perception | yes |

**Fiscal position substitutions.**

| Fiscal position | Source taxes | Replacement |
|---|---|---|
| Purchases / Sales abroad | the 21, 10.5, 0, 27 percent purchase taxes and the exempt and untaxed purchase taxes | Value-added tax Not Applicable (purchase) |
| Purchases / Sales abroad | the 10.5, 0, 27 and 21 percent sales taxes | Value-added tax Exempt (sale) |
| Purchases / Sales Free Trade Zone | the 21, 10.5, 0 and 27 percent purchase taxes | Value-added tax Exempt (purchase) |
| Purchases / Sales Free Trade Zone | the 10.5, 0, 27 and 21 percent sales taxes | Value-added tax Exempt (sale) |
| Purchases value-added tax in the correspondent | the 21, 10.5, 0, 27 percent purchase taxes and the exempt and untaxed purchase taxes | Value-added tax Not Applicable (purchase) |

### 3.2 Turnover tax perceptions

The base template ships twenty-four **incurred** perception taxes, one per province plus the city of Buenos Aires, all with the purchase scope, the fixed-amount computation and an amount of 1 unit. They are entered by hand on a vendor bill: the supplier printed a perception amount on the bill and the operator types it.

The exempt template ships the matching twenty-four **applied** perception taxes, all with the sale scope, the percentage computation and a rate of zero, all shipped archived. An accountant who must charge a perception activates the province's tax and sets its rate to the one the province publishes for that counterpart.

| Province or city | Incurred tax name | Applied tax name | Tax group |
|---|---|---|---|
| City of Buenos Aires | P. IIBB CABA | P. IIBB CABA 0% | Perception turnover tax City of Buenos Aires |
| Buenos Aires province | P. IIBB PBA | P. IIBB PBA 0% | Perception turnover tax Buenos Aires |
| Catamarca | P. IIBB C | P. IIBB C 0% | Perception turnover tax Catamarca |
| Córdoba | P. IIBB CBA | P. IIBB CBA 0% | Perception turnover tax Córdoba |
| Corrientes | P. IIBB CTS | P. IIBB CTS 0% | Perception turnover tax Corrientes |
| Entre Ríos | P. IIBB ER | P. IIBB ER 0% | Perception turnover tax Entre Ríos |
| Jujuy | P. IIBB J | P. IIBB J 0% | Perception turnover tax Jujuy |
| Mendoza | P. IIBB MZA | P. IIBB MZA 0% | Perception turnover tax Mendoza |
| La Rioja | P. IIBB LR | P. IIBB LR 0% | Perception turnover tax La Rioja |
| Salta | P. IIBB S | P. IIBB S 0% | Perception turnover tax Salta |
| San Juan | P. IIBB SJ | P. IIBB SJ 0% | Perception turnover tax San Juan |
| San Luis | P. IIBB SL | P. IIBB SL 0% | Perception turnover tax San Luis |
| Santa Fe | P. IIBB SF | P. IIBB SF 0% | Perception turnover tax Santa Fe |
| Santiago del Estero | P. IIBB SE | P. IIBB SE 0% | Perception turnover tax Santiago del Estero |
| Tucumán | P. IIBB T | P. IIBB T 0% | Perception turnover tax Tucumán |
| Chaco | P. IIBB CHO | P. IIBB CHO 0% | Perception turnover tax Chaco |
| Chubut | P. IIBB CHT | P. IIBB CHT 0% | Perception turnover tax Chubut |
| Formosa | P. IIBB F | P. IIBB F 0% | Perception turnover tax Formosa |
| Misiones | P. IIBB MS | P. IIBB MS 0% | Perception turnover tax Misiones |
| Neuquén | P. IIBB N | P. IIBB N 0% | Perception turnover tax Neuquén |
| La Pampa | P. IIBB LP | P. IIBB LP 0% | Perception turnover tax La Pampa |
| Río Negro | P. IIBB RN | P. IIBB RN 0% | Perception turnover tax Río Negro |
| Santa Cruz | P. IIBB SC | P. IIBB SC 0% | Perception turnover tax Santa Cruz |
| Tierra del Fuego | P. IIBB TAIS | P. IIBB TAIS 0% | Perception turnover tax Tierra del Fuego |

Several provinces prescribe a specific printed wording, which is stored as the tax's printed label rather than as its name: Entre Ríos prints "Imp. Pciales o IIBB o Profesiones Liberales Entre Ríos", Mendoza prints "Alícuota IBMza - Ley N° 9655", Chubut prints "VALOR APROXIMADO DEL ISIB CHUBUT" and the City of Buenos Aires prints "ALÍCUOTA ISIB CABA".

### 3.3 Other perceptions

| Name | Rate | Scope | Tax group | Active |
|---|---|---|---|---|
| Perc Profits | fixed amount, 1 unit | purchase | Profit Perceptions | yes |
| Perc Profits 0% | 0 percent | sale | Profit Perceptions | no |
| `Perc VAT 0%` | 0 percent | sale | Value-added tax Perception | no |
| Other taxes | fixed amount, 1 unit | purchase | Other Taxes | yes |
| Internal taxes | fixed amount, 1 unit | purchase | Internal Taxes | yes |

### 3.4 Withholdings at payment

The withholding package adds withholding taxes whose scope is "none", so that they cannot be chosen on an invoice line, and whose payment direction decides whether they apply to a vendor payment or to a customer payment.

| Family | Direction | Count | Computation kind |
|---|---|---|---|
| Turnover tax withholding incurred, per province | customer payment | 24 | untaxed base, or total amount for Córdoba |
| Turnover tax withholding applied, per province | vendor payment | 24 | untaxed base, or total amount for Córdoba |
| Social security withholding incurred | customer payment | 1 | percentage |
| Social security withholding applied | vendor payment | 1 | percentage |
| Earnings withholding incurred | customer payment | 1 | percentage |
| Earnings withholding applied, per regime | vendor payment | several, keyed by the administration's regime code | earnings, or earnings on a progressive scale |

An **incurred** withholding records an amount another party withheld from the company; an **applied** withholding records an amount the company withheld from another party.

---

## 4. Tax groups

Forty groups in the base template, two more in the exempt template and twenty-five in the withholding package. Every group carries the country Argentina, a payable account and a receivable account, and one of the two administration codes.

**Value-added tax rate codes.**

| Group | Rate code |
|---|---|
| Value-added tax Not Applicable | `0` |
| Value-added tax Untaxed | `1` |
| Value-added tax Exempt | `2` |
| Value-added tax 0% | `3` |
| Value-added tax 10.5% | `4` |
| Value-added tax 21% | `5` |
| Value-added tax 27% | `6` |
| Value-added tax 5% | `8` |
| Value-added tax 2.5% | `9` |

**Tribute codes.**

| Code | Meaning | Groups carrying it |
|---|---|---|
| `01` | National Taxes | National Taxes, Profit Withholdings |
| `04` | Internal Taxes | Internal Taxes |
| `06` | Value-added tax perception | Value-added tax Perception, Value-added tax Withholding |
| `07` | Turnover tax perception | the twenty-four provincial perception groups |
| `08` | Municipal perceptions | Municipal Taxes Perceptions |
| `09` | Other perceptions | Profit Perceptions, Other Perceptions |
| `99` | Others | Other Taxes |

**Deletion guard.** A tax group the country requires cannot be deleted:

```
The tax group '<name>' can't be removed, since it is required in the Argentinian localization.
```

---

## 5. Fiscal positions

| Sequence | Name | Applies automatically to responsibility types | Country |
|---|---|---|---|
| 10 | Argentina Domestic | none (it is the domestic position) | Argentina |
| 20 | Purchases / Sales abroad | Foreign counterpart | none |
| 30 | Purchases / Sales Free Trade Zone | Value-added tax liberated | none |
| 40 | Purchases value-added tax in the correspondent | Exempt taxpayer and simplified regime taxpayer | none |

**Automatic application.** Beyond the five framework predicates, an Argentine fiscal position may name a list of responsibility types. It is applied when the counterpart's responsibility type is in that list. This is an additional predicate specific to this country.

---

## 6. Responsibility types

The responsibility type decides, together with the company's own responsibility type, which document letter may be issued. The catalogue is a shared reference table with a unique name and a unique code.

| Code | Meaning |
|---|---|
| 1 | Registered taxpayer |
| 3 | Non-registered taxpayer |
| 4 | Exempt taxpayer |
| 5 | Final consumer |
| 6 | Simplified regime taxpayer |
| 7 | Non-categorised or non-registered person |
| 8 | Value-added tax liberated, promotional law |
| 9 | Foreign counterpart |
| 10 | Foreign counterpart, simplified regime |
| 13 | Simplified regime, small taxpayer |

The company's own responsibility type is restricted to codes 1, 4 and 6, because only those three may issue documents.

**Immutability.** The company's responsibility type cannot change once accounting exists:

```
Could not change the ARCA Responsibility of this company because there are already accounting entries.
```

---

## 7. Document types and numbering

The package uses the shared Latin American document type mechanism. A sales journal that issues legal documents carries:

| Field | Type | Meaning |
|---|---|---|
| Uses documents | boolean | Marks the journal as issuing legal documents. |
| Is a point of sale | boolean, derived and editable, stored | True when the country is Argentina, the journal type is sale and the journal uses documents. |
| Point of sale number | integer | The number the administration assigned. Written into the journal code as five digits with leading zeros. |
| Point of sale system | selection | Which issuing system the point of sale uses. |
| Point of sale address | many_to_one to Contact restricted to the company's own addresses | Printed on documents issued from the point of sale. |

**Point of sale systems.**

| Code | Meaning |
|---|---|
| `II_IM` | Pre-printed or imported invoice |
| `RLI_RLM` | Pre-printed or imported receipt |
| `RAW_MAW` | Web receipt or web invoice |
| `CPERCIBIDO` | Perceived receipt |
| `BFERCEL` | Electronic tax bond |
| `FEERCEL`, `FEEWS`, `FEERCELP` | Export invoice, three variants |
| `FERCEL` | Electronic invoice |
| `FERINI` | Initial electronic invoice |

**Validation rules.**

| Rule | Message |
|---|---|
| A point of sale journal must have a number. | `Please define an ARCA POS number` |
| The number may not exceed five digits. | `Please define a valid ARCA POS number (5 digits max)` |
| A purchase journal may only use the pre-printed, receipt or web systems. | `The pos system <system> can not be used on a purchase journal (id <identifier>)` |
| The type, the point of sale system, the point of sale number and the documents marker may not change once validated invoices exist. | `You can not change <journal> journal's configuration if it already has validated invoices` |

**Journal code.** Setting the point of sale number on a sales journal rewrites the journal code as the number in five digits with leading zeros, for example point of sale 3 gives the code `00003`.

**Document letters.** The document type carries a letter selection built from the counterpart's and the issuer's responsibility types. The letters are `A`, `B`, `C`, `E`, `M`, `T`, `R`, `X` and `I`.

**Document number format.** A dash, at most five characters before it and at most eight after it, for example `0001-00000001`. An import dispatch document requires exactly sixteen characters.

```
<value> is not a valid value for <field>.
The number of import Dispatch must be 16 characters.

<value> is not a valid value for <field>.
The document number must be entered with a dash (-) and a maximum of 5 characters for the first part and 8 for the second. The following are examples of valid numbers:
* 1-1
* 0001-00000001
* 00001-00000001
```

---

## 8. Posting constraints

| Rule | Condition | Message |
|---|---|---|
| A journal that uses documents accepts only invoice-class entries. | a miscellaneous entry is posted in such a journal | `The selected Journal can't be used in this transaction, please select one that doesn't use documents as these are just for Invoices.` |
| Exactly one value-added tax per line. | a line carries zero or more than one tax whose group belongs to the value-added tax family | `There should be a single tax from the "VAT" tax group per line, but this is not the case for line "<line>". Please add a tax to this line or check the tax configuration's advanced options for the corresponding field "Tax Group".` |
| A document type whose purchase aliquot marker is `zero` requires the Not Applicable tax on every line. | a bill carries another value-added tax | `On invoice id "<number>" you must use VAT Not Applicable on every line.` |
| A document type whose purchase aliquot marker is `not_zero` forbids the Not Applicable tax. | a bill carries the Not Applicable tax | `On invoice id "<number>" you must use a VAT tax that is not VAT Not Applicable` |
| The document number of an automatically numbered journal may only be changed in its point of sale part. | the user edits the counter part | `The document number can not be changed for this journal, you can only modify the point of sale part.` |

---

## 9. Identification and validation of counterparts

The package uses the shared Latin American identification type mechanism, and adds the administration code to each type. The company and every counterpart carry a responsibility type.

**Tax identification number validation.** The national taxpayer number is eleven digits with a check digit. Validation raises:

| Condition | Message |
|---|---|
| No number configured. | `No VAT configured for partner [<identifier>] <name>` |
| The check digit does not match. | `The validation digit is not valid for "<value>"` |
| The length is not eleven. | `Invalid length for "<value>"` |
| A non-digit character is present. | `Only numbers allowed for "<value>"` |
| The two leading digits are not one of the allowed person categories. | `CUIT number must be prefixed with one of the following: <list>` |

**Derived presentation fields on a contact.**

| Field | Rule |
|---|---|
| Tax identification number digits | The number, or nothing when the identification type is not the tax number. |
| Formatted tax identification number | The number rendered as two digits, a hyphen, ten digits, a hyphen and the check digit. |

**Generic numbers per country.** The Country record carries three generic numbers the administration publishes for counterparts of that country: one for natural persons, one for legal entities and one for others, each eleven characters, together with a three-character country code used on electronic documents. The Currency record carries a four-character administration code.

**Unit of measure.** Each unit of measure carries the administration's unit code.

---

## 10. Withholding at payment

### 10.1 Configuration on the tax

| Field | Values | Meaning |
|---|---|---|
| Argentine tax scope | `sale`, `purchase`, `none`, `supplier` (vendor payment withholding), `customer` (customer payment withholding) | Extends the ordinary scope with the two withholding scopes. |
| Withholding payment type | `supplier`, `customer` | Which direction the withholding applies to. |
| Withholding tax kind | `earnings`, `earnings_scale`, `iibb_untaxed`, `iibb_total` | Which computation applies. |
| Withholding numbering series | many_to_one to Sequence | Produces the certificate number when none is typed. |
| Administration code | text | The regime code, used to group withholdings of the same regime for the period accumulation. |
| Non-taxable amount | decimal | The part of the base that is never withheld on. |
| Minimum threshold | decimal | A computed amount below this threshold becomes zero. |
| Jurisdiction | many_to_one to Country Subdivision, deletion restricted | The province a turnover tax withholding belongs to. |
| Scale | many_to_one to Argentina Earnings Scale | The bracket table for a scale withholding. |

### 10.2 Base amount

```
base_amount = payment_amount                                                                when kind = "iibb_total"
base_amount = payment_amount × Σ invoice.untaxed_total ÷ Σ invoice.total                    otherwise
```

The second form strips the taxes out of the payment, because a turnover tax withholding applies to the untaxed base and a payment settles a tax-inclusive total.

**Worked example.** A payment of 121,000.00 settles one invoice whose untaxed total is 100,000.00 and whose total is 121,000.00.

```
base_amount = 121,000.00 × 100,000.00 ÷ 121,000.00 = 100,000.00
```

### 10.3 Period accumulation for earnings withholdings

For a withholding whose kind is `earnings` or `earnings_scale`, the system accumulates, over the calendar month containing the payment date, every posted journal item of the same company tree, the same administration regime code, the same counterpart (taken at the commercial level) and a withholding kind of `earnings` or `earnings_scale`:

```
same_period_base        = |Σ balance of the base items matching the criteria|
same_period_withheld    = |Σ balance of the tax items matching the criteria|
net_amount              = max(0, base_amount + same_period_base − non_taxable_amount)
```

For every other kind:

```
net_amount = max(0, base_amount − non_taxable_amount)
```

### 10.4 Computed amount

```
tax_amount = the ordinary tax computation applied to net_amount, rounded per line
```

Then, for an earnings scale withholding, the bracket is selected and the amount is replaced:

```
bracket    = the bracket of the tax's scale whose excess_amount ≤ net_amount < to_amount
tax_amount = (net_amount − bracket.excess_amount) × bracket.percentage ÷ 100 + bracket.fixed_amount
```

Then, for either earnings kind:

```
tax_amount = tax_amount − same_period_withheld
```

Finally, for every kind:

```
tax_amount = 0   when minimum_threshold > tax_amount
```

**Worked example, plain percentage.** A vendor payment of 100,000.00 with a 3 percent turnover tax withholding on the untaxed base, a non-taxable amount of 10,000.00, no minimum threshold. The invoice is untaxed 100,000.00 and total 121,000.00, the payment is 121,000.00.

```
base_amount = 100,000.00
net_amount  = max(0, 100,000.00 − 10,000.00) = 90,000.00
tax_amount  = round(90,000.00 × 0.03, 2)     =  2,700.00
```

**Worked example, earnings scale with accumulation.** The counterpart has already received 250,000.00 this month under the same regime and 20,000.00 was withheld. A new payment brings a base of 100,000.00. The non-taxable amount is 0. The scale is:

| Excess amount from | Up to | Fixed amount | Percentage |
|---|---|---|---|
| 0.00 | 100,000.00 | 0.00 | 5 |
| 100,000.00 | 300,000.00 | 5,000.00 | 10 |
| 300,000.00 | 1,000,000.00 | 25,000.00 | 15 |

```
net_amount   = max(0, 100,000.00 + 250,000.00 − 0.00) = 350,000.00
bracket      = the third one, because 300,000.00 ≤ 350,000.00 < 1,000,000.00
tax_amount   = (350,000.00 − 300,000.00) × 15 ÷ 100 + 25,000.00 = 32,500.00
tax_amount   = 32,500.00 − 20,000.00 = 12,500.00
```

**Worked example, minimum threshold.** A computed amount of 180.00 with a minimum threshold of 200.00 becomes 0.00, and no withholding line is posted.

### 10.5 Numbering

A line without a number consumes the tax's withholding numbering series when the payment is created. A line without a number and without a series aborts:

```
Please enter withholding number for tax <tax name>
```

### 10.6 Net amount and the check adjustment

```
net_amount = payment_amount − Σ withholding_line.amount
```

When the payment is made with checks, the sum of the check amounts must equal the net amount. Because a change in the payment amount changes the withholding bases, which changes the withheld amounts, which changes the net amount, the system solves for the payment amount iteratively:

1. Let `checks_amount` be the sum of the check amounts and `net` the current net amount.
2. If the two are equal in the payment currency, stop.
3. Set `delta = checks_amount − net`. When `delta` is negative, raise the payment amount to `checks_amount` and recompute `delta`.
4. Add `delta` to the payment amount, recompute the bases, the withheld amounts and the net amount.
5. From the second iteration on, estimate the local slope as `(net − previous_net) ÷ previous_delta`, take `delta = max((checks_amount − net) ÷ slope, 0.01)`, add it and recompute.
6. Stop when the difference is zero in the payment currency, or after 201 iterations, in which case the payment amount is reset to its original value and a warning flag is raised.

A warning flag is shown whenever the check total and the net amount differ.

**Worked example.** Checks total 97,300.00. A 3 percent withholding applies to the untaxed base. Starting from a payment amount of 97,300.00 the net amount is 94,887.55, so `delta = 2,412.45`. Adding it gives 99,712.45 and a net of 97,238.10. The slope is `(97,238.10 − 94,887.55) ÷ 2,412.45 = 0.97434`, so the next step is `(97,300.00 − 97,238.10) ÷ 0.97434 = 63.53`. Adding it gives 99,775.98 and a net of 97,300.00, and the loop stops.

### 10.7 Journal entry

The Argentine withholding lines are written as write-off lines of the payment entry.

| Line | Account | Amount in currency | Sign |
|---|---|---|---|
| One per withholding line | the account of the tax's matching repartition line | the withheld amount | `+1` for a customer payment, `−1` for a vendor payment |
| One base line per distinct base amount | the company's Tax Base Account | the base amount | same sign |
| One base counterpart line per distinct base amount | the company's Tax Base Account | the base amount | opposite sign |

The base line carries every withholding tax that shares that base, so that each of them reports its own base on the return. The two base lines net to zero. Each line's name is the withholding number; the base lines' name is the comma-separated list of the numbers sharing that base.

**Worked example.** A vendor payment of 121,000.00 with a 3 percent turnover tax withholding of 2,700.00 on a base of 90,000.00 and a 2 percent earnings withholding of 1,800.00 on the same base.

| Line | Account | Debit | Credit |
|---|---|---|---|
| Counterpart | Trade payables | 121,000.00 | |
| Outstanding payments | Outstanding Payments | | 116,500.00 |
| Turnover tax withholding applied | Turnover tax withholding applied, province | | 2,700.00 |
| Earnings withholding applied | Withholding return to be paid | | 1,800.00 |
| Withholding base | Tax Base Account | | 90,000.00 |
| Withholding base counterpart | Tax Base Account | 90,000.00 | |
| **Totals** | | **211,000.00** | **211,000.00** |

**Guard.**

```
A payment cannot have withholding if the payment method has no outstanding accounts
```

### 10.8 Per-counterpart authorisation

A contact carries a list of dated withholding authorisations, each naming a tax, a start date, an end date and a certificate reference. The list restricts which withholding taxes are proposed for that counterpart and documents the exemption certificate the counterpart presented. The dates must be ordered:

```
"From date" must be lower than "To date" on Withholding (AR) taxes.
```

---

## 11. Check management

The Latin American check package, which Argentina depends on, adds two payment method families.

**New third-party check.** A payment of this method represents a check received from a customer. It creates a Latin America Check record holding the check number, the issuing bank, the issuer's tax identification number, the payment date, the amount and the outstanding journal item it produced.

**Existing third-party check.** A payment of this method moves a check that is already held: it is handed to a supplier, deposited at a bank, returned by the bank after rejection, or voided.

**Own checks.** The company's own checks are filled by hand rather than printed, may be deferred or electronic, and carry a cash-in date for a post-dated check.

| Check field | Meaning |
|---|---|
| Number | The check number. Unique per payment method line among checks that still have an outstanding line. |
| Issuing bank | Derived from the counterpart, editable. |
| Issuer tax identification number | Derived from the counterpart, editable, normalised on entry and validated on save. |
| Payment date | The date the check may be cashed. |
| Amount | Must be strictly greater than zero (`The amount of the check must be greater than 0`). |
| Outstanding journal item | The item the check produced. |
| Current journal | Derived from the last posted operation: where the check is now. |
| Issue state | `handed` (Handed), `debited` (Debited), `voided` (Voided), derived from the residual amount of the outstanding item. |
| Operations | Every payment that moved the check. |

**Guards.**

```
Can't delete a check if payment is In Process!
A payment with any Third Party Check or Own Check payment methods needs an outstanding account
You can't cancel or re-open a payment with checks if some check has been debited or been voided.
You can't mix checks of different currencies in one payment,
```

**Mass transfer.** A wizard moves several checks to another journal at once, creating one internal transfer per check. Guards:

```
All selected checks must be on the same journal and on hand
The register payment wizard should only be called on account.payment records.
You have selected payments which are not checks. Please call this action from the Third Party Checks menu
All the selected checks must be posted
All the selected checks must use the same currency
```

---

## 12. Delivery guide

The stock package adds a numbering mechanism for the transport document that accompanies goods.

**On the operation type.**

| Field | Type | Meaning |
|---|---|---|
| Document type | many_to_one to Latin America Document Type | The class assigned to the delivery guide. |
| Authorisation code | text | The code the administration issued for the numbering range. |
| Authorisation expiry date | date | The last day the code may be used. |
| Sequence from | text of 8 digits | The first number the administration allocated. |
| Sequence to | text of 8 digits | The last number allocated. |
| Delivery guide prefix | text, default `00001` | The point of sale part of the number. |
| Next delivery guide number | integer | The next counter value. |
| Delivery guide numbering series | many_to_one to Sequence | The series that produces the number. |

**On the transfer.**

| Field | Type | Meaning |
|---|---|---|
| Delivery guide number | text, read-only, not copied | The produced number. |
| Authorisation data | structured_data, not copied | The authorisation code, its expiry and the range, frozen at the moment of issuance. |
| May generate a delivery guide | boolean, derived | Whether the button is offered. |
| May send a delivery guide | boolean, derived | Whether the document may be mailed. |

**Validation.**

| Rule | Message |
|---|---|
| A range bound must be exactly eight digits. | `<value> is not a valid sequence number. Sequence numbers should contain exactly 8 digits (e.g. 00012345).` |
| The produced number must lie inside the authorised range. | `The delivery guide number <number> exceeds the range specified in the CAI. Please update the range or use a different CAI with a different range.` |
| Mailing requires an address. | `The partner does not have an email address.` |

---

## 13. Point of sale

The point of sale configuration is linked to the responsibility type catalogue and the identification type catalogue, both of which are loaded into the register client so that a cashier can record a counterpart's identification and responsibility at the till. A contact that has been used on a point of sale order may not be deleted:

```
Deleting this partner is not allowed.
```

---

## 14. Online checkout

| Setting | Location | Default | Effect |
|---|---|---|---|
| Display price without national taxes | Website | derived | Shows, beside the selling price, the price excluding the national taxes, which several provinces require for consumer transparency. |

The checkout form collects the identification type, the identification number and the responsibility type, validated with the same checkers and the same messages as the contact form.

---

## 15. Acceptance scenarios

**Given** a company in Argentina with no accounting,
**when** the registered taxpayer template is loaded,
**then** 299 accounts exist, the account code length is 12, the bank prefix is `1.1.1.02.`, the sales journal is named "Ventas Preimpreso" with the code `0001` and the point of sale number 1, and the default sales tax is the 21 percent value-added tax.

**Given** a sales journal that uses documents,
**when** its point of sale number is set to 37,
**then** its code becomes `00037`.

**Given** a sales journal with validated invoices,
**when** the user tries to change its point of sale number,
**then** the change is refused with `You can not change <journal> journal's configuration if it already has validated invoices`.

**Given** an invoice line with no value-added tax,
**when** the invoice is posted,
**then** the posting is refused with the single-value-added-tax message naming the line.

**Given** a vendor payment of 121,000.00 settling an invoice whose untaxed total is 100,000.00 and whose total is 121,000.00, with a 3 percent turnover tax withholding on the untaxed base and a non-taxable amount of 10,000.00,
**when** the withholding is computed,
**then** the base is 100,000.00, the net base is 90,000.00 and the withheld amount is 2,700.00.

**Given** a counterpart that already received 250,000.00 this month under the same earnings regime, on which 20,000.00 was withheld, and the three-bracket scale above,
**when** a further payment with a base of 100,000.00 is registered,
**then** the accumulated base is 350,000.00, the scale gives 32,500.00 and the amount withheld now is 12,500.00.

**Given** a withholding whose computed amount is 180.00 and whose minimum threshold is 200.00,
**when** the payment is created,
**then** the withheld amount is 0.00.

**Given** a check whose amount is zero,
**when** it is saved,
**then** the save is refused with `The amount of the check must be greater than 0`.
