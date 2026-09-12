# United Kingdom

The United Kingdom package supplies one chart of accounts template of 118 accounts with six-character codes, nine depreciation models, seventeen taxes covering the three rates, the domestic and international reverse charges, the postponed import accounting mechanism and the Northern Ireland protocol taxes, four tax groups sharing one tax account and all declaring the same preceding subtotal label, and three fiscal positions of which one is restricted to a country group that contains the United Kingdom and Northern Ireland.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `uk` |
| Display name | United Kingdom |
| Parent template | none |
| Fiscal country | United Kingdom |
| Account code length | 6 characters |

### 1.1 Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 118 |
| Code length | 6 characters |
| Account groups shipped | none |
| Depreciation models shipped | 9 |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 7 |
| Fixed Asset | 20 |
| Receivable | 3 |
| Equity | 4 |
| Current Year Earnings | 1 |
| Expense | 57 |
| Cost of Revenue | 4 |
| Income | 7 |
| Current Liability | 9 |
| Payable | 6 |

**First accounts of the plan.**

| Code | Name | Type |
|---|---|---|
| `001000` | Technology | Fixed Asset |
| `001100` | Technology Depreciation | Fixed Asset |
| `002000` | Patents and Trademarks | Fixed Asset |
| `002100` | Patents and Trademarks Depreciation | Fixed Asset |
| `003000` | Installations | Fixed Asset |
| `220200` | Value-added tax control account | Current Liability |

Several accounts ship archived and several ship with a default tax attached.

**Depreciation models.**

| Model | Asset account | Accumulated depreciation account | Depreciation expense account | Periods | Period length, months |
|---|---|---|---|---|---|
| Technology | `001000` | `001100` | `751000` | 3 | 12 |
| Patents and Trademarks | `002000` | `002100` | `800000` | 10 | 12 |
| Installations | `003000` | `003100` | `800100` | 10 | 12 |
| Buildings | `004001` | `004101` | `800100` | 30 | 12 |
| Vehicles | `005000` | `005100` | `800100` | 5 | 12 |
| Furniture and material | `006000` | `006100` | `750600` | 5 | 12 |
| Plant and machinery | `007000` | `007100` | `800100` | 5 | 12 |
| Other property | `008000` | `008100` | `800100` | 5 | 12 |
| Vehicle accessories | `009000` | `009100` | `800100` | 5 | 12 |

**Worked example.** A vehicle capitalised at 30,000.00 under the Vehicles model depreciates over 5 periods of 12 months at `round(30,000.00 ÷ 5, 2) = 6,000.00` per year, debiting `800100` and crediting `005100`.

---

## 2. Taxes

Seventeen taxes, seven for sales and ten for purchases.

### 2.1 Sales

| Order | Name | Printed label | Rate | Product applicability | Tax group | Active | Fiscal position |
|---|---|---|---|---|---|---|---|
| 1 | 20% | 20% | 20 | any | Value-added tax 20% | yes | United Kingdom |
| 2 | 5% | 5% | 5 | any | Value-added tax 5% | yes | United Kingdom |
| 3 | 0% | 0% | 0 | any | Value-added tax 0% | yes | United Kingdom |
| 4 | Exempt | Exempt | 0 | any | Value-added tax 0% | yes | United Kingdom |
| 5 | 0% European Union (Northern Ireland) | 0% | 0 | any | Value-added tax 0% | no | Northern Ireland to European Union member states, business to business |
| 6 | 0% reverse charge | `Reverse Charge: S55A VATA 94 applies` | 0 | goods | Value-added tax 0% | yes | United Kingdom |
| 7 | 0% export | 0% | 0 | any | Value-added tax 0% | yes | Rest of the world |

The domestic reverse charge label is the exact wording the law requires on the invoice.

### 2.2 Purchases

| Order | Name | Printed label | Rate | Product applicability | Tax group | Active | Fiscal position |
|---|---|---|---|---|---|---|---|
| 101 | 20% G | 20% | 20 | goods | Value-added tax 20% | yes | United Kingdom |
| 102 | 20% S | 20% | 20 | services | Value-added tax 20% | yes | United Kingdom |
| 103 | Exempt | Exempt | 0 | any | Value-added tax 0% | yes | United Kingdom |
| 104 | 5% | 5% | 5 | any | Value-added tax 5% | yes | United Kingdom |
| 105 | 20% reverse charge, domestic | `20% Reverse Charge` | 20 | any | Value-added tax 20% | yes | United Kingdom |
| 106 | 20% European Union (Northern Ireland) | 20% | 20 | any | Value-added tax 0% | no | Northern Ireland to European Union member states |
| 107 | 0% European Union (Northern Ireland) | 0% | 0 | any | Value-added tax 0% | no | the same position |
| 108 | 20% reverse charge, international | `20% Reverse Charge` | 20 | services | Value-added tax 20% | yes | Rest of the world |
| 109 | 20% postponed import accounting | `20% PVA` | 20 | goods | Value-added tax 20% | yes | Rest of the world |
| 110 | 0% export | 0% | 0 | any | Value-added tax 0% | yes | Rest of the world |

**Reverse charge arithmetic.** A reverse-charge purchase tax charges the tax and deducts it in the same entry, so the amount due to the supplier is unchanged while both the output box and the input box of the return receive the amount.

**Worked example.** A purchase of services from abroad for 1,000.00 carrying the 20 percent international reverse charge.

```
tax charged (output box)  =  200.00
tax deducted (input box)  = −200.00
amount due to the supplier = 1,000.00
```

**Postponed import accounting.** Import tax is accounted for on the return instead of being paid at the border. The 20 percent postponed tax behaves like a reverse charge: it charges and deducts the same amount, and its report tags feed the postponed import boxes rather than the ordinary ones.

**Worked example.** Goods imported with a customs value of 5,000.00.

```
postponed tax charged = 1,000.00
postponed tax deducted = −1,000.00
cash paid at the border = 0.00
```

**Northern Ireland protocol.** Four taxes exist for movements between Northern Ireland and the member states, all shipped archived. A company that trades under the protocol activates them and the matching fiscal position.

---

## 3. Tax groups

| Group | Payable account | Receivable account | Preceding subtotal |
|---|---|---|---|
| Value-added tax 0% | `220200` | `220200` | Subtotal |
| Value-added tax 5% | `220200` | `220200` | Subtotal |
| Value-added tax 17.5% | `220200` | `220200` | Subtotal |
| Value-added tax 20% | `220200` | `220200` | Subtotal |

The 17.5 percent group is historical and carries no shipped tax; it exists so that a document dated before the rate change keeps its group.

All four share the single control account `220200`, which the periodic return settles.

---

## 4. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country group | Active on install |
|---|---|---|---|---|---|
| 2 | United Kingdom | yes | no | the United Kingdom and Northern Ireland group | yes |
| 3 | Northern Ireland to European Union member states, business to business | yes | yes | European Union group | no |
| 4 | Rest of the world | yes | no | none | yes |

The domestic position names a **country group** rather than a country, because Northern Ireland is treated as part of the domestic territory for goods while being inside the union's rules for movements to member states.

---

## 5. Acceptance scenarios

**Given** a company in the United Kingdom with no accounting,
**when** the template is loaded,
**then** 118 accounts with six-character codes and nine depreciation models exist, four tax groups exist all pointing at `220200` and all declaring the preceding subtotal "Subtotal", and three fiscal positions exist of which the Northern Ireland one is archived.

**Given** a purchase of services from abroad for 1,000.00 carrying the international reverse charge,
**when** the totals are computed,
**then** 200.00 is charged and 200.00 is deducted and the amount due to the supplier is 1,000.00.

**Given** goods imported with a customs value of 5,000.00 under postponed accounting,
**when** the bill is posted,
**then** 1,000.00 is charged and 1,000.00 is deducted and nothing is paid at the border.

**Given** a domestic supply of goods under the reverse charge,
**when** the invoice is printed,
**then** the tax line carries the exact label `Reverse Charge: S55A VATA 94 applies`.

**Given** a vehicle capitalised at 30,000.00 under the Vehicles model,
**when** depreciation runs,
**then** 6,000.00 is posted each year for five years, debiting `800100` and crediting `005100`.
