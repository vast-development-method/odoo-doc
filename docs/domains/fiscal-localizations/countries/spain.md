# Spain

The Spanish package supplies a family of ten chart of accounts templates arranged in a three-level chain, separating the mainland regime from the Canary Islands regime and, inside each, the complete plan from the small and medium enterprise plan, the cooperative plans and the non-profit plans. It ships 598 common accounts and 930 account groups, 168 mainland taxes and 126 Canary Islands taxes, 29 mainland fiscal positions covering the withholding ladder, the equivalence surcharge, the agricultural regime and the customs declaration, a simplified invoice limit, and three separate electronic registration flows: the immediate information supply, the Basque Country registry and the national verifiable invoice registry, plus the public sector invoice format.

---

## 1. Templates

| Template code | Display name | Parent | Visible | Ordering |
|---|---|---|---|---|
| `es_common` | Common | none | no | default |
| `es_common_mainland` | Common Mainland | `es_common` | no | default |
| `es_canary_common` | Common Canary Islands | `es_common` | no | default |
| `es_pymes` | Small and medium enterprises (2008) | `es_common_mainland` | yes | 0 |
| `es_full` | Complete (2008) | `es_common_mainland` | yes | default |
| `es_coop_pymes` | Cooperatives, small and medium enterprises (2008) | `es_common_mainland` | yes | default |
| `es_coop_full` | Cooperatives, complete (2008) | `es_common_mainland` | yes | default |
| `es_assec` | Non-profit entities (2008) | `es_common_mainland` | yes | default |
| `es_canary_pymes` | Canary Islands, small and medium enterprises (2008) | `es_canary_common` | yes | default |
| `es_canary_full` | Canary Islands, complete (2008) | `es_canary_common` | yes | default |
| `es_canary_assoc` | Canary Islands, non-profit entities (2008) | `es_canary_common` | yes | default |

The two `es_common*` templates and `es_canary_common` are invisible: they exist only to be inherited. The small and medium enterprise template has ordering 0 and is therefore the guessed template for Spain.

**Chain example.** Loading `es_pymes` walks the chain `["es_pymes", "es_common_mainland", "es_common"]`. Delimited files are read generic first, so `account.account-es_common.csv` (598 rows), then `account.account-es_common_mainland.csv` (93 rows), then `account.account-es_pymes.csv` (9 rows). The result is the union, with the child's values winning on the nine overlapping rows.

### 1.1 Company-level values written by the common template

| Company field | Value |
|---|---|
| Fiscal country | Spain |
| Bank account code prefix | `572` |
| Cash account code prefix | `570` |
| Transfer account code prefix | `57299` (mainland) or `572999` (Canary Islands) |
| Default point of sale receivable account | `4301` |
| Gain exchange rate account | `768` |
| Loss exchange rate account | `668` |
| Bank suspense account | `572998` |
| Cash discount write-off loss account | `6060` |
| Cash discount write-off gain account | `7060` |
| Cash difference income account | `778` |
| Cash difference expense account | `678` |
| Deferred expense account | `480` |
| Deferred revenue account | `485` |
| Income account | `7000` |
| Expense account | `600` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `310` |
| Receivable account recorded as a Contact default | `4300` |
| Payable account recorded as a Contact default | `4100` |

The mainland chain additionally writes the default sales tax (21 percent) and the default purchase tax (21 percent for goods); the Canary Islands chain writes the 7 percent general indirect tax on both sides.

The stock valuation account `310` names its stock expense account (`601`) and its stock variation account (`611`).

---

## 2. Chart of accounts

| File | Rows | Code lengths |
|---|---|---|
| Common | 598 | 3 to 6 characters |
| Common mainland | 93 | 3 and 4 characters, some rows carrying no code |
| Canary Islands common | 8 | 4 and 5 characters |
| Small and medium enterprises | 9 | 3 and 4 characters |
| Complete | 109 | 3, 4 and 6 characters |
| Cooperatives, complete | 106 | 3, 4 and 6 characters |
| Cooperatives, small and medium enterprises | 64 | 3, 4 and 5 characters |
| Non-profit entities | 61 | 3 and 4 characters |

**Account groups.** 930 in the common template plus 16 in the cooperatives, small and medium enterprises template, reproducing the national plan's groups, sub-groups and accounts.

**Account type distribution of the common template.**

| Account type | Count |
|---|---|
| Bank and Cash | 6 |
| Current Asset | 90 |
| Fixed Asset | 84 |
| Non-current Asset | 3 |
| Receivable | 31 |
| Equity | 15 |
| Expense | 46 |
| Depreciation | 3 |
| Other Expense | 94 |
| Income | 103 |
| Other Income | 1 |
| Current Liability | 59 |
| Non-current Liability | 44 |
| Payable | the remainder |

**Depreciation models.** Seventeen in the common template and one more in the complete template, each naming the asset account, the accumulated depreciation account, the depreciation expense account and the number of periods.

**Translations.** Every account name and description ships in Spanish and in Catalan.

---

## 3. Taxes

### 3.1 Mainland

168 taxes. Every tax carries four Spanish fields:

| Field | Values | Meaning |
|---|---|---|
| Tax kind | `exento` (exempt), `sujeto` (subject), `sujeto_agricultura` (subject, agricultural regime), `sujeto_isp` (subject, reverse charge), `no_sujeto` (not subject), `no_sujeto_loc` (not subject by the place-of-supply rules), `ignore` (ignored in the registry) | How the registry must classify the tax. |
| Exempt reason | `E1` (article 20), `E2` (article 21), `E3` (article 22), `E4` (articles 23 and 24), `E5` (article 25), `E6` (others) | Why an exempt supply is exempt. |
| Capital goods | boolean | Marks a tax on capital goods, reported separately. |
| Registry applicability | `01` (value-added tax), `02` (production, services and imports tax), `03` (Canary Islands general indirect tax) | Which tax the verifiable invoice registry declares. |
| Public sector tax type | the legal type list (value-added tax, the Ceuta and Melilla tax, the Canary Islands tax, personal income tax, and others) | Which tax the public sector format declares. |

**Rates shipped.** 0, 2, 4, 5, 7.5, 10, 12, 21 percent, plus the agricultural regime at 10.5 percent and an exempt family.

### 3.2 Equivalence surcharge

A retailer under the equivalence surcharge regime pays, in addition to the ordinary tax, a surcharge that discharges its own output obligation. The package ships a surcharge for each rate.

| Surcharge rate | Applies with the tax rate |
|---|---|
| 0 percent | 0 percent |
| 0.26 percent | the tobacco rate |
| 0.5 percent | 4 percent |
| 0.62 percent | 5 percent |
| 1.4 percent | 10 percent |
| 1 percent | 7.5 percent |
| 5.2 percent | 21 percent |

**Worked example.** A wholesaler invoices a retailer under the surcharge regime for 1,000.00 of goods at 21 percent.

```
value-added tax        = round(1,000.00 × 0.21,   2) = 210.00
equivalence surcharge  = round(1,000.00 × 0.052,  2) =  52.00
invoice total          = 1,262.00
```

### 3.3 Withholding ladder

Thirteen withholding groups, each with its own rate, all wired to the payable account `4750` and the receivable account `4709`.

| Group |
|---|
| Withholding 0%, 1%, 2%, 7%, 9%, 15%, 18%, 19%, 19.5%, 20%, 21%, 24%, 35% |

### 3.4 Canary Islands

126 taxes implementing the general indirect tax of the archipelago, with its own rate ladder and its own seventeen tax groups. A Canary Islands company is subject to that tax instead of the mainland value-added tax, which is why the two chains are separate.

---

## 4. Tax groups

| Chain | Groups |
|---|---|
| Common | 15: the thirteen withholding groups, the exempt customs declaration group and the base-subtraction group. |
| Common mainland | 17: eight value-added tax rate groups (0, 2, 4, 5, 7.5, 10, 12, 21 percent), the agricultural regime group at 10.5 percent, the exempt group and the seven equivalence surcharge groups. |
| Canary Islands common | 17: the general indirect tax rate groups. |

Every mainland and common group is wired to the payable account `4750`. The receivable account is `4700` for the value-added tax groups and `4709` for the withholding groups.

---

## 5. Fiscal positions

29 positions in the mainland chain and 10 in the Canary Islands chain.

### 5.1 Automatically detected

| Sequence | Name | Requires a tax number | Country or group |
|---|---|---|---|
| 10 | Spain Domestic | no | Spain, restricted to the mainland country group |
| 20 | Intra-community | yes | European Union group |
| 30 | European Union private | no | European Union group |
| 40 | Extra-community | no | none |

### 5.2 Chosen by hand

| Family | Positions |
|---|---|
| Regime | Regime not subject to the place-of-supply rules; National reverse charge; Customs declaration |
| Equivalence surcharge | Equivalence surcharge; Reseller reverse charge equivalence surcharge |
| Personal income tax withholding | 1%, 2%, 7%, 9%, 15%, 18%, 19%, 20%, 21%, 24% |
| Lease withholding | 19% leases, 19.5% leases, 21% leases, and a second 19% lease position |
| Non-resident withholding | 24% for non-union residents, 19% for union residents, exempt for union residents, exempt for non-union residents |
| Agricultural regime | Agriculture; animal breeding and fishing |

The Canary Islands chain ships ten positions restricted to the Canary Islands country subdivisions.

---

## 6. Simplified invoice

| Setting | Location | Default | Effect |
|---|---|---|---|
| Simplified invoice limit | Company | 400 | Above this amount a simplified invoice may not be issued. |
| Is simplified | Journal Entry, derived and editable, stored | derived | Marks the document as a simplified invoice, which omits the customer's identification. |
| Simplified invoice journal | Point of Sale Configuration | empty | The journal that numbers simplified invoices issued at the till. |

**Worked example.** With the default limit of 400, a till sale of 380.00 may be issued as a simplified invoice; one of 420.00 requires the customer's identification and a full invoice.

---

## 7. Registration flows

The package supplies four separate flows. A company uses the ones its region and its size require.

### 7.1 Immediate information supply

| Element | Behavior |
|---|---|
| Settings | One or more certificates scoped to this flow, the tax agency (the national agency or one of the three provincial agencies of Gipuzkoa, Bizkaia and Navarra), and a test mode flag defaulting to true. |
| Required | Derived on the invoice: true when the company is enrolled and the invoice is in scope. |
| Transmission | Each posted invoice is transmitted; the agency returns a validation code. |
| Fields on the invoice | Validation code (tracked), registration date. |

### 7.2 Basque Country registry

| Element | Behavior |
|---|---|
| Settings | One or more certificates scoped to this flow, the tax agency (Araba, Bizkaia or Gipuzkoa), a test mode flag defaulting to true, a chain numbering series, and a derived licence text rendered as rich text. |
| Document | A Spain Basque Country Document per submission, with a chain index, a signature, a state and the agency's answer. |
| Chain | Every submission takes the next chain index; a rejected submission does not consume one. |
| Fields on the invoice | State (To Send, Sent, Cancelled), chain index, the submission document, the cancellation document, the two payload files and their names, whether the flow is required, a refund reason chosen from the legal list, and the list of vendor bills a single refund covers. |
| Refund reason | Required on a credit note; the values are the legal grounds for reducing a taxable base. |
| Visual code | The printed invoice carries a code encoding the verification address, whose check character is computed with an eight-bit cyclic redundancy check over the address. |

### 7.3 National verifiable invoice registry

| Element | Behavior |
|---|---|
| Settings | Certificates, an enablement flag, a test environment flag defaulting to true, a chain numbering series, the earliest instant at which the next batch may be sent, and a special tax regime (simplified, the agricultural and fisheries regime, or the equivalence surcharge). |
| Document | A Spain Verifiable Invoice Document per submission or cancellation, with a chain index, the payload, the errors, the agency's receipt reference and a state. |
| States | Rejected, Registered with Errors, Accepted, plus a Cancelled state derived on the invoice. |
| Batching | Documents waiting to be sent are transmitted in batches by a daily scheduled job, which respects the stored next-batch instant and re-arms itself. |
| Fingerprint chain | Each record's fingerprint is computed over its own identifying fields and the previous record's fingerprint, so a missing record breaks the chain. |
| Fields on the invoice | The documents, the derived state, the visual code, the derived warning level and warning text, the regime key with its dynamically restricted list, the substituted simplified invoice, the substituting invoices, and a refund reason (`R1` articles 80.1 and 80.2 and error of law, `R2` article 80.3, `R3` article 80.4, `R4` rest, `R5` corrective invoices concerning simplified invoices). |
| Deletion guard | `You cannot delete Veri*Factu Documents that are part of the chain of all Veri*Factu Documents.` |
| Point of sale | Receipts are registered the same way; the document carries a point of sale order instead of an invoice. |

### 7.4 Public sector invoice format

| Element | Behavior |
|---|---|
| Settings | Certificates scoped to this flow; a residence type derived from the company contact. |
| Contact | An administrative centre is a child contact of a public body, carrying a code of at most 10 characters, a list of roles chosen from the Spain Administrative Centre Role Type catalogue, a physical location number and a logical location number, each at most 14 characters. |
| Fields on the invoice | The payload file and its attachment, a correction reason code chosen from the legal list, the invoicing period start and end dates, and the payment means code. |
| Payment means codes | In cash, direct debit, receipt, credit transfer, accepted bill of exchange, documentary credit, contract award, bill of exchange, transferable promissory note, non-transferable promissory note, and the remaining legal codes. |
| Unit codes | Every unit of measure carries the legal unit code: units, hours, kilograms, litres, other, boxes, trays, barrels, jerricans, and the rest of the published list. |

---

## 8. Acceptance scenarios

**Given** a company in Spain with no accounting,
**when** the small and medium enterprise template is loaded,
**then** the chain `es_pymes`, `es_common_mainland`, `es_common` is walked, the bank prefix is `572`, the cash prefix is `570`, the transfer prefix is `57299`, 598 accounts from the common file plus 93 from the mainland file plus 9 from the small and medium enterprise file are merged, 930 account groups exist and the default sales tax is the 21 percent tax.

**Given** the Canary Islands complete template,
**when** it is loaded,
**then** the transfer prefix is `572999` and the default sales tax is the 7 percent general indirect tax.

**Given** a wholesale invoice of 1,000.00 at 21 percent to a retailer under the equivalence surcharge regime,
**when** the totals are computed,
**then** the tax is 210.00, the surcharge is 52.00 and the total is 1,262.00.

**Given** a company whose simplified invoice limit is 400 and a till sale of 420.00,
**when** the receipt is issued,
**then** a simplified invoice may not be used and the customer's identification is required.

**Given** a tax whose rate is zero and whose kind is exempt,
**when** it is saved without an exempt reason,
**then** the registry classification cannot be produced and the accountant must choose one of the six legal grounds.

**Given** a verifiable invoice document that has entered the chain,
**when** deletion is attempted,
**then** it is refused with the chain message.
