# Belgium

The Belgian package supplies three chart of accounts templates arranged as a base and two visible variants, 426 accounts plus the variant-specific accounts, 21 depreciation models, a tax set organised around the four rates and the reporting boxes of the periodic return, four tax groups each carrying a fiscal-printer department letter, six fiscal positions including a reverse-charge position with a printed legal note and a non-deductible position used by default on purchase receipts, a structured payment reference model, a merchandise-versus-investment split of the purchase taxes, and a point of sale certification requirement.

---

## 1. Templates

| Template code | Display name | Parent | Visible | Ordering | Account code length |
|---|---|---|---|---|---|
| `be` | Base | none | no | default | 6 |
| `be_comp` | Companies | `be` | yes | 0 | 6 |
| `be_asso` | Associations and Foundations | `be` | yes | default | 6 |

The company template has ordering 0 and is therefore the guessed template for Belgium.

### 1.1 Company-level values written by the base template

| Company field | Value |
|---|---|
| Fiscal country | Belgium |
| Bank account code prefix | `550` |
| Cash account code prefix | `570` |
| Transfer account code prefix | `580` |
| Default point of sale receivable account | `4001` |
| Gain exchange rate account | `754` |
| Loss exchange rate account | `654` |
| Bank suspense account | `499` |
| Cash discount write-off loss account | `657000` |
| Cash discount write-off gain account | `757000` |
| Cash difference income account | `757100` |
| Cash difference expense account | `657100` |
| Inter-banks transfer account | `58` |
| Default sale tax | the 21 percent output tax on goods |
| Default purchase tax | the 21 percent input tax, box 81 |
| Default purchase receipt fiscal position | Non Deductible |
| Income account | `7000` |
| Expense account | `600` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `300` |
| Receivable account recorded as a Contact default | `400` |
| Payable account recorded as a Contact default | `440` |
| Down payment account | `46` |

The stock valuation account `300` names its stock expense account (`600`) and its stock variation account (`6090`).

### 1.2 Journals

Both the sales journal and the purchase journal are created with **separate credit note numbering** switched on, because the country requires credit notes to carry their own series.

### 1.3 Reconciliation models

Beyond the two the framework creates, the base template adds one:

| Symbolic identifier | Name | Line |
|---|---|---|
| `escompte_template` | Cash Discount | one line, 100 percent, account `653`, labelled "Cash Discount Granted" |

The name ships translated into French, Dutch and German.

### 1.4 Bank fee account override

The bank fee reconciliation model points at account `6560` when it exists, instead of the generic search by name.

### 1.5 Post-load step

For the two visible templates, the purchase journal's non-deductible account is set to account `416`.

---

## 2. Chart of accounts

| File | Rows | Code lengths |
|---|---|---|
| Base | 426 | 2 to 6 characters |
| Companies | 43 | 2 to 4 characters |
| Associations and Foundations | 68 | 2 to 4 characters |

**Account type distribution of the base file.**

| Account type | Count |
|---|---|
| Bank and Cash | 3 |
| Current Asset | 39 |
| Fixed Asset | 68 |
| Non-current Asset | 49 |
| Receivable | 8 |
| Equity | 7 |
| Current Year Earnings | 2 |
| Expense | 111 |
| Depreciation | 3 |
| Other Expense | 44 |
| Income | 6 |
| Other Income | 30 |
| Current and Non-current Liability and Payable | the remainder |

Several accounts ship with a default tax and with report tags already attached. **Depreciation models:** 21, each naming the asset account, the accumulated depreciation account, the depreciation expense account, the number of periods and the period length.

**Translations.** Every account name and description ships in French, Dutch and German.

---

## 3. Taxes

The tax set is organised around the boxes of the periodic return: each tax carries the report tags of the box that must receive its base and the box that must receive its amount. The four rates are 21, 12, 6 and 0 percent.

**Naming convention.** An output tax is named after its rate and the box of the return; an input tax is named after its rate and its box number, for example the 21 percent input tax of box 81.

**Product applicability split.** The package extends the tax product applicability with two Belgian values:

| Value | Meaning |
|---|---|
| Merchandise | Goods bought for resale, reported in box 81. |
| Investment | Capital goods, reported in box 83. |

The split exists because the return separates the deductible input tax by the nature of the purchase, and the package ships one purchase tax per rate and per nature.

---

## 4. Tax groups

| Group | Payable account | Receivable account | Fiscal printer department |
|---|---|---|---|
| Value-added tax 21% | `4512` | `4112` | `A` |
| Value-added tax 12% | `4512` | `4112` | `B` |
| Value-added tax 6% | `4512` | `4112` | `C` |
| Value-added tax 0% | `4512` | `4112` | `D` |

The department letter is what a certified fiscal printer expects on a receipt line, so that the printer's own totals agree with the ledger.

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country or group | Note printed |
|---|---|---|---|---|---|
| 10 | Domestic | yes | no | Belgium | none |
| 20 | Intra-Community | yes | yes | European Union group | none |
| 30 | European Union, business to consumer | yes | no | European Union group | none |
| 40 | Extra-Community | yes | no | none | none |
| 50 | Co-contractor | no | no | none | `Reverse charge: In the absence of a written objection within one month of receipt of the invoice, the customer is deemed to acknowledge that he is a taxable person required to submit periodic returns. If that condition is not met, the customer will bear the liability for the payment of interest and fines due.` |
| 60 | Non Deductible | no | no | none | none |

Each position also carries account mappings that redirect revenue and expense accounts.

---

## 6. Structured payment reference

The journal's payment reference model is extended with a Belgian form: a twelve-digit number rendered between two triples of plus signs and grouped as three digits, a slash, four digits, a slash, five digits, for example `+++000/2024/00182+++`. The last two digits are a check derived from the first ten. Removing the model from a journal resets it to the platform's own model.

---

## 7. Point of sale

The point of sale packages add the certification chain to orders issued in a restaurant or from a sales order, so that the sequence of receipts cannot be altered after the fact. The mechanism is the one described in [../workflows.md](../workflows.md) section 13.2.

---

## 8. Acceptance scenarios

**Given** a company in Belgium with no accounting,
**when** the company template is loaded,
**then** the chain `be_comp`, `be` is walked, the bank prefix is `550`, the cash prefix is `570`, the transfer prefix is `580`, the bank suspense account is `499`, the inter-banks transfer account is `58`, the sales and purchase journals number credit notes separately, four tax groups exist with the department letters `A` to `D`, and the purchase journal's non-deductible account is `416`.

**Given** a purchase of capital goods,
**when** the purchase tax whose applicability is Investment is chosen,
**then** the base is reported in the capital goods box rather than in the merchandise box.

**Given** a customer under the co-contractor regime,
**when** the fiscal position "Co-contractor" is applied,
**then** the invoice carries no tax and the reverse-charge note is printed.

**Given** a purchase receipt,
**when** it is created,
**then** the fiscal position "Non Deductible" is proposed by default.
