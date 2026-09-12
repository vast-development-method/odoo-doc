# Austria

The Austrian package supplies one chart of accounts template built on the standardised chart with four-character codes, 61 taxes whose names and descriptions quote the box of the periodic return they feed, seven tax groups sharing a single tax account, four fiscal positions, the Germany-style printed document layout as the company's report layout and paper format, and external classification tags on the two utility liquidity accounts.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `at` |
| Display name | Austria |
| Parent template | none |
| Fiscal country | Austria |
| Account code length | 4 characters |
| Storno accounting | offered as an option, defaulting to off |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `280` |
| Cash account code prefix | `270` |
| Transfer account code prefix | `288` |
| Default point of sale receivable account | `2099` |
| Gain exchange rate account | `4860` |
| Loss exchange rate account | `7860` |
| Cash discount write-off loss account | `5800` |
| Cash discount write-off gain account | `8350` |
| Printed document layout | the Germany-style layout |
| Paper format | the Germany-style paper format |
| Default sale tax | the 20 percent output tax of box 022 |
| Default purchase tax | the 20 percent input tax of box 060 |
| Income account | `4000` |
| Expense account | `5010` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `1600` |
| Receivable account recorded as a Contact default | `2000` |
| Payable account recorded as a Contact default | `3300` |
| Stock valuation account recorded as a company property | `1600` |

The stock valuation account `1600` names its stock expense account (`5010`) and its stock variation account (`5880`).

### 1.2 Tags on the utility accounts

After the utility accounts are created, the package tags two of them so that the statutory classification of the chart is complete:

| Account | Tags added |
|---|---|
| Bank suspense account | the external classification tag for code `2300` and the standardised chart tag |
| Inter-banks transfer account | the external classification tag for code `2885` and the standardised chart tag |

This is the only place in the domain where a country package tags accounts the framework created rather than accounts the template shipped.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Code length | 4 characters |
| Account groups shipped | none |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 42 |
| Fixed Asset | 39 |
| Non-current Asset | 31 |
| Prepayments | 7 |
| Receivable | 4 |
| Equity | 5 |
| Current Year Earnings | 1 |
| Expense | 37 |
| Depreciation | 2 |
| Cost of Revenue | 14 |
| Income | 5 |
| Other Income and the liability types | the remainder |

---

## 3. Taxes

61 taxes. Every tax's description names the box of the periodic return it feeds and the legal paragraph it rests on, for example "Section 19 (1) second sentence, tax liability concerns the service recipient".

| Family | Rates | Boxes |
|---|---|---|
| Domestic sales | 20, 13, 10 percent | 022, 029, 006 |
| Domestic purchases | 20, 13, 10 percent | 060 and its companions |
| Reverse charge on sales | 0 percent, with several legal grounds under section 19 paragraphs 1, 1a, 1b, 1c and 1d | 021 |
| Reverse charge on purchases | 20, 13, 10 percent, charged and deducted | the acquisition boxes |
| Intra-union supply | 0 percent | the supply box |
| Intra-union acquisition | 20, 10 percent | the acquisition boxes |
| Export | 0 percent | the export box |
| Import | 20, 10 percent | the import boxes |
| Agricultural flat rate | 13, 10 percent | the flat-rate boxes |
| Special rates | 19, 4.9 percent | the special-rate boxes |

The 19 percent rate applies in two border municipalities and the 4.9 percent rate is an agricultural flat rate; both exist as their own tax groups.

---

## 4. Tax groups

| Group | Payable account | Receivable account |
|---|---|---|
| 0% | `3530` | `3530` |
| 4.9% | `3530` | `3530` |
| 10% | `3530` | `3530` |
| 12% | `3530` | `3530` |
| 13% | `3530` | `3530` |
| 19% | `3530` | `3530` |
| 20% | `3530` | `3530` |

All seven share the single tax account `3530`, which the periodic return settles.

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country or group |
|---|---|---|---|---|
| 10 | National | yes | no | Austria |
| 20 | European Union | yes | yes | European Union group |
| 30 | National and European Union without a tax number | yes | no | European Union group |
| 40 | Third countries | yes | no | none |

Each position carries account mappings that separate domestic, intra-union and export revenue.

---

## 6. Printed documents

The company's report layout and paper format are set to the Germany-style layout, which prescribes a fixed geometry: an address window positioned for a window envelope, a reference block carrying the document number, the document date, the customer number and the contact, an optional line position column controlled by a company setting, and a totals block.

---

## 7. Acceptance scenarios

**Given** a company in Austria with no accounting,
**when** the Austrian template is loaded,
**then** the account code length is 4, the bank prefix is `280`, the cash prefix is `270`, the transfer prefix is `288`, seven tax groups exist all pointing at `3530`, the report layout is the Germany-style layout, the bank suspense account carries the external classification tag for `2300` and the inter-banks transfer account carries the tag for `2885`.

**Given** the storno setting,
**when** the settings screen is opened for an Austrian company,
**then** the control is shown and defaults to off, because Austria is in the optional set.

**Given** a sale under section 19 paragraph 1 where the recipient owes the tax,
**when** the corresponding zero-rate tax is chosen,
**then** the base is reported in box 021 and the description quotes the legal ground.
