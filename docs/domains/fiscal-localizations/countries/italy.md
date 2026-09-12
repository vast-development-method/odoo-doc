# Italy

The Italian package supplies one chart of accounts template of 184 accounts, 108 taxes covering eight value-added tax rates, the exemption families with their legal notes, the split payment mechanism, the pension fund contributions, the social security contribution for agents and the withholding taxes, 14 tax groups each declaring the subtotal it must be excluded from, six fiscal positions with printed legal notes, the national fiscal code and the exchange destination code on every counterpart, a document type catalogue, a transport document, the declaration of intent mechanism with its consumable threshold, and the electronic invoicing cycle with the national exchange system.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `it` |
| Display name | Italy |
| Parent template | none |
| Fiscal country | Italy |
| Account code length | 6 characters (the shipped codes are 4 characters and are padded) |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `182` |
| Cash account code prefix | `180` |
| Transfer account code prefix | `183` |
| Default point of sale receivable account | `1508` |
| Gain exchange rate account | `3220` |
| Loss exchange rate account | `4920` |
| Cash discount write-off loss account | `4111` |
| Cash discount write-off gain account | `3111` |
| Default sale tax | 22 percent sales tax |
| Default purchase tax | 22 percent purchase tax on goods |
| Deferred expense account | `1902` |
| Deferred revenue account | `2702` |
| Income account | `3101` |
| Expense account | `4101` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `1404` |
| Receivable account recorded as a Contact default | `1501` |
| Payable account recorded as a Contact default | `2501` |

The stock valuation account `1404` names its stock expense account (`4101`) and its stock variation account (`4131`).

**Padding example.** The account whose template code is `1501` is stored as `150100`.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 184 |
| Code length | 4 characters, padded to 6 |
| Account groups shipped | none |
| Translations shipped | Italian |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 27 |
| Fixed Asset | 1 |
| Non-current Asset | 21 |
| Receivable | 2 |
| Equity | 2 |
| Expense | 61 |
| Income | 13 |
| Other Income | 4 |
| Current Liability | 33 |
| Payable | 6 |
| Off-balance | 14 |

Key tax accounts: `2605` value-added tax, `2609` pension funds, `2610` agents' social security, `2611` withholdings.

---

## 3. Taxes

108 taxes. Every tax carries an **exemption reason** when its rate is zero, and a **legal note** that is printed on the invoice.

### 3.1 Value-added tax rates

Each rate exists four times: sale on goods, sale on services, purchase on goods and purchase on services, because the return separates goods from services.

| Rate | Names | Tax group | Active on install |
|---|---|---|---|
| 22 percent | 22%, 22% S, 22% G | 22% value-added tax | yes |
| 10 percent | 10%, 10% S, 10% G | 10% value-added tax | yes |
| 5 percent | 5%, 5% S, 5% G | 5% value-added tax | no |
| 4 percent | 4%, 4% S, 4% G | 4% value-added tax | no |
| 2 percent | 2% | 2% value-added tax | no |
| 12 percent | 12% | 12% value-added tax | no |
| 20 percent | 20% | 20% value-added tax | no |
| 21 percent | 21% | 21% value-added tax | no |
| 0 percent | 0%, 0% S, 0% G | Value-added tax free | yes |

The rates 12, 20 and 21 percent are historical and exist so that a document dated before a rate change keeps the rate that applied then.

**Product applicability.** A tax whose name ends in `S` has the product applicability "Services"; one whose name ends in `G` has "Goods". The applicability is what lets the system propose the right tax for a product without the operator choosing.

### 3.2 Exemption families

| Name | Legal note printed | Exemption reason |
|---|---|---|
| 0% EU G | Article 41 of the intra-union decree | the intra-union supply code |
| 0% EU S | Article 7-ter of the value-added tax decree | the out-of-scope services code |
| 0% EX | Outside the European Union | the export code |
| 0% EX N7 | Outside the European Union | the special export code |
| 0% Art.15 | Article 15 of the value-added tax decree | `N1` excluded under article 15 |
| 0% E (declaration of intent) | Article 8, paragraph 1, letter c of the value-added tax decree, letter of intent | `N3.5` |

**Exemption reason codes.**

| Code | Meaning |
|---|---|
| `N1` | Excluded under article 15. |
| `N2.1` | Not subject under articles 7 to 7-septies. |
| `N2.2` | Not subject, other cases. |
| `N3.1` | Not taxable, exports. |
| `N3.2` | Not taxable, intra-union supplies. |
| `N3.3` | Not taxable, supplies to San Marino. |
| `N3.4` | Not taxable, transactions treated as exports. |
| `N3.5` | Not taxable, following a declaration of intent. |
| `N3.6` | Not taxable, other transactions that do not create a deduction limit. |
| `N4` | Exempt. |
| `N5` | Margin scheme. |
| `N6.1` to `N6.9` | Reverse charge, one code per sector (scrap, gold, construction subcontracting, buildings, mobile telephones, electronics, cleaning, energy certificates, other). |
| `N7` | Value-added tax paid in another member state. |

**Validation.**

```
If the tax amount is 0%, you must enter the exoneration code and the related legal notes.
Split Payment is not compatible with exoneration of kind 'N6'
```

### 3.3 Split payment

A public administration customer does not pay the tax to the seller; it pays it directly to the administration. The split payment family charges the tax and immediately removes it from the amount due, so the base and the tax are both declared while the customer pays only the base.

Its tax group declares the preceding subtotal "Split Payments Excluded", which is the label printed above the subtotal that excludes it.

**Worked example.** An invoice of 1,000.00 with the 22 percent tax under the split payment regime.

```
base                          = 1,000.00
value-added tax declared      =   220.00
split payment deduction       =  −220.00
amount due from the customer  = 1,000.00
```

### 3.4 Pension funds, agents' social security and withholdings

| Family | Tax group | Preceding subtotal label | Account |
|---|---|---|---|
| Pension fund contribution | Pension Funds | Pension Funds Excluded | `2609` |
| Agents' social security contribution | Agents' social security | Agents' social security Excluded | `2610` |
| Withholding tax | Withholdings | Withholdings Excluded | `2611` |

Each of these appears on the invoice as a separate subtotal, because the amount the customer pays differs from the taxable base by each of them in turn.

**Fields on a tax.**

| Field | Values |
|---|---|
| Withholding type | the administration's withholding type codes, including the agents' social security type |
| Withholding reason | the administration's reason codes, including `ZO` other reason |
| Pension fund type | the administration's pension fund codes |

**Validations.**

```
Tax '<name>' has a withholding type so the amount must be negative.
Tax '<name>' has a withholding type, so the withholding reason must also be specified
Tax '<name>' has a withholding reason, so the withholding type must also be specified
Tax '<name>' has one of withholding and pension fund types that do not relate to ENASARCO, and one that does.
Tax '<name>' has withholding type ENASARCO, the withholding reason should be [ZO] - Other reason.
```

**Worked example of a professional invoice.** Fees of 1,000.00, a 4 percent pension fund contribution and a 20 percent withholding.

```
fees                       = 1,000.00
pension fund contribution  =    40.00     (4 percent of the fees)
taxable base               = 1,040.00
value-added tax at 22%     =   228.80
withholding at 20%         =  −200.00     (20 percent of the fees)
amount due from client     = 1,068.80
```

---

## 4. Tax groups

14 groups, all with the country Italy. Each declares the label of the subtotal it must be excluded from, and several carry a point of sale receipt label, which is the department number a fiscal printer expects.

| Group | Preceding subtotal | Receivable and payable account | Receipt label |
|---|---|---|---|
| 2% value-added tax | Taxable | `2605` | none |
| 4% value-added tax | Taxable | `2605` | `3` |
| 5% value-added tax | Taxable | `2605` | `4` |
| 10% value-added tax | Taxable | `2605` | `2` |
| 12% value-added tax | Taxable | `2605` | none |
| 20% value-added tax | Taxable | `2605` | none |
| 21% value-added tax | Taxable | `2605` | none |
| 22% value-added tax | Taxable | `2605` | `1` |
| Value-added tax excluded under article 15 | Taxable | `2605` | none |
| Value-added tax free | Taxable | `2605` | `10` |
| Split Payment | Split Payments Excluded | `2605` | none |
| Pension Funds | Pension Funds Excluded | `2609` | none |
| Agents' social security | Agents' social security Excluded | `2610` | none |
| Withholdings | Withholdings Excluded | `2611` | none |

---

## 5. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country or group | Legal note printed |
|---|---|---|---|---|---|
| 10 | Domestic | yes | no | Italy | none |
| 20 | Intra-Community | yes | yes | European Union group | `Invoice issued in accordance with Article 17, Paragraph 2 of Presidential Decree No. 633 dated October 26, 1972, with VAT to be paid by the recipient.` |
| 30 | European Union, business to consumer | yes | no | European Union group | none |
| 40 | Import and Export | yes | no | none | none |
| 50 | Split Payment | no | no | none | `Operations subject to split payment, the seller does not collect the value-added tax pursuant to article 17-ter of Presidential Decree 633 of 1972, the tax is paid by the buyer.` |
| 90 | Construction Subcontractors, reverse charge | no | no | none | `Operation subject to Reverse Charge and exempt from stamp duty, the seller does not collect the value-added tax pursuant to article 17, paragraph 6 of Presidential Decree 633 of 1972.` |
| 10 (contributed by the declaration of intent package) | Declaration of Intent | no | no | none | `Not taxable (article 8, paragraph 1, letter c of Presidential Decree 633 of 1972), letter of intent.` |

---

## 6. Counterparts and the company

### 6.1 Contact

| Field | Type | Meaning |
|---|---|---|
| National fiscal code | text of at most 16 characters | The identity code of a person or a business. |
| Certified mail address | text | The certified electronic mail address the exchange system may use. |
| Destination code | text of 6 or 7 characters | The recipient's address inside the exchange system. A six-character code marks a public administration. |
| Declarations of intent | one_to_many to Italy Declaration of Intent | Declarations the counterpart has issued. |

**Check constraints.**

```
Codice fiscale must have between 11 and 16 characters.
Destination Code (SDI) must have between 6 and 7 characters.
```

**Format validation.**

```
Invalid Codice Fiscale '<value>': should be like 'MRTMTT91D08F205J' for physical person and '12345670546' for businesses.
```

### 6.2 Company

| Field | Meaning |
|---|---|
| National fiscal code | Mirrored from the company contact and stored. |
| Tax system | The regime the company is subject to, declared on every invoice. |
| Registered for exchange | Whether the company receives inbound invoices through the exchange service. |
| Purchase journal for inbound invoices | Where inbound invoices become draft bills. |
| Listed in the register of companies | Reveals the five register fields. |
| Register office province | The province of the register office, restricted to Italy. |
| Register number | At most 20 characters. |
| Paid-up share capital | Mandatory for a capital company. |
| Sole shareholder | Not a limited liability company, single shareholder, several shareholders. |
| Liquidation state | In liquidation, not in liquidation. |
| Has a tax representative | Whether a resident representative acts for a non-resident company. |
| Tax representative contact | The representative. |
| Declaration of intent tax | The zero-rate tax applied under a declaration. |
| Declaration of intent fiscal position | The position applied under a declaration. |

**Validations.**

```
The Italian default purchase journal requires a default account.
All fields about the Economic and Administrative Index must be completed.
If one of Share Capital or Sole Shareholder is present, the other one must be too.
You must select a tax representative.
Your tax representative partner must have a tax number.
Your tax representative partner must have a country.
Please fill your codice fiscale to be able to receive invoices from FatturaPA
```

---

## 7. Document types and payment methods

### 7.1 Document types

A shared catalogue with a unique code, a translatable name and a side (sale or purchase). The codes name the classes the exchange system recognises: the ordinary invoice, the deferred invoice, the third-party invoice, the advance invoice on a fee, the advance invoice on a freelance fee, the self-billed invoice for a purchase from a non-resident, the self-billed invoice for a reverse charge, the integration document for an intra-union purchase, the credit note, the debit note and the simplified invoice and its credit note.

```
Document Type code must be unique.
```

On an invoice, the document type is derived from the entry kind, the counterpart and the fiscal position, and remains editable.

### 7.2 Payment methods

Every payment method line carries an Italian payment method code, defaulting to the code for a bank transfer. The invoice carries a derived, editable payment method that the payload transmits.

---

## 8. Transport document

| Field | Type | Meaning |
|---|---|---|
| Number | text of at most 20 characters, required | The transport note number. |
| Date | date, required | The transport note date. |
| Invoices | one_to_many to Journal Entry | The invoices that reference it. |

An invoice that summarises several deliveries quotes its transport document, whose number and date are transmitted with the payload.

---

## 9. Declaration of intent

A habitual exporter issues a declaration allowing its suppliers to invoice without tax up to a threshold. The record is fully specified in [../entities.md](../entities.md) section 3.10.

**Application.** When a document is prepared for a counterpart, the system looks for a declaration of that counterpart, for that company, valid on that date, in that currency, whose remaining amount is still positive. When found, the declaration is proposed on the document, which then carries:

| Field | Meaning |
|---|---|
| Declaration of intent | The declaration applied. |
| Declaration of intent date | The date at which validity is judged: the invoice date, or the order date for a quotation. |
| Use the declaration of intent | Derived; true when a declaration applies. |
| Declaration of intent amount | How much of the declaration the document consumes. |
| Declaration of intent warning | The threshold warning text. |

**Threshold arithmetic.**

```
invoiced          = Σ declaration amount of the posted invoices linked to the declaration
not_yet_invoiced  = Σ not-yet-invoiced amount of the linked quotations and orders
remaining         = threshold − invoiced − not_yet_invoiced
```

**Worked example.** A declaration with a threshold of 50,000.00. Posted invoices under it total 38,000.00 and open orders total 9,000.00, so the remaining amount is 3,000.00. A new order of 5,000.00 would push the remaining amount to −2,000.00; the warning names the declaration, the two totals and the overrun of 2,000.00.

**Effect on taxes.** Applying the declaration replaces the ordinary tax by the company's declaration of intent tax, which is the zero-rate tax carrying the exemption reason `N3.5` and the legal note about article 8.

**State machine.** Draft, Active, Revoked, Terminated, with the transitions given in [../entities.md](../entities.md) section 3.10.

```
You cannot delete Declarations of Intents that are already used on at least one Invoice or Sales Order.
The Protocol Number of a Declaration of Intent must be unique! Please choose another one.
The Threshold of a Declaration of Intent must be positive.
```

---

## 10. Stamp duty

An invoice whose exempt base exceeds the legal threshold carries a stamp duty, stored as a decimal on the invoice and transmitted with the payload.

```
stamp_duty_applies = (Σ base of the lines carrying an exempt tax) > threshold
```

**Worked example.** With a threshold of 77.47 and a fixed amount of 2.00, an invoice carrying 150.00 of exempt base carries a stamp duty of 2.00; one carrying 60.00 carries none.

---

## 11. Public procurement references

| Field | Meaning |
|---|---|
| Tender unique identifier | Required on an invoice to a public administration under a tender. |
| Public investment unique identifier | Required on an invoice to a public administration under a funded investment. |
| Origin document type | Purchase Order, Contract or Agreement. |
| Origin document name | The reference of that document. |
| Origin document date | Its date. |

---

## 12. Electronic invoicing

### 12.1 Registration

The company registers with the exchange service through a proxy user whose kind is the Italian exchange. Registration requires the national fiscal code.

### 12.2 State machine

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| (none) | Post an eligible invoice | the counterpart has a destination code or a certified mail address | (empty) | The send button appears |
| (empty) | Send | no blocking validation error; the document is locked | Being Sent To the exchange system | Payload attached, transaction reference stored |
| Being Sent | Service accepts the upload | none | Exchange system Processing | Header text updated |
| Processing | Service reports rejection | none | Exchange system Rejected | Errors posted in the discussion thread; the invoice may be reset to draft |
| Processing | Service reports delivery | none | Accepted, Forwarded to Partner | Confirmation posted |
| Processing | Service reports a failed forward | none | Accepted, Forward Failed | The seller must deliver a copy by other means |
| Processing | Service reports it is still forwarding | none | Forwarding | |
| Processing | Public administration accepts | none | Accepted by the public administration | |
| Processing | Public administration rejects | none | Rejected by the public administration | A credit note is required |
| Processing | The acceptance window expires | fifteen days | Accepted after expiry | |
| Processing | No answer at all | none | Accepted with no answer | |
| any | Anything the service reports that is not mapped | none | Other | The raw status is stored |

**Guards and messages.**

```
This move is not waiting for updates from the SdI.
This document is being sent by another process already.
An error occurred while downloading updates from the Proxy Server: (<code>) <message>
```

### 12.3 Fields on the invoice

| Field | Meaning |
|---|---|
| Exchange state | The twelve states above. |
| Exchange header | A rich-text description of the state with the next action. |
| Exchange transaction | The service's transaction reference. |
| Payload file and payload name | The transmitted payload. |
| Proxy operating mode | Related to the company's proxy user. |
| Button label | Derived; what the send button says in the current state. |
| Is a self-billed invoice | Derived; true when the company issues the document on behalf of a non-resident supplier or under the reverse charge. |
| Counterpart is a public administration | Derived; true when the destination code is six characters long. |

### 12.4 Inbound invoices

A scheduled job runs daily and fetches invoices addressed to the company from the exchange system, creating draft vendor bills in the purchase journal named on the company.

---

## 13. Acceptance scenarios

**Given** a company in Italy with no accounting,
**when** the Italian template is loaded,
**then** 184 accounts exist with six-character codes, the bank prefix is `182`, the cash prefix is `180`, the transfer prefix is `183`, 14 tax groups exist and the withholding group points at account `2611`.

**Given** a tax whose rate is zero and which carries no exemption reason,
**when** it is saved,
**then** the save is refused with the exoneration message.

**Given** an invoice of 1,000.00 under the split payment regime with the 22 percent tax,
**when** the totals are computed,
**then** the declared tax is 220.00, the split payment deduction is −220.00 and the amount due is 1,000.00.

**Given** fees of 1,000.00, a 4 percent pension fund contribution, the 22 percent tax and a 20 percent withholding,
**when** the totals are computed,
**then** the taxable base is 1,040.00, the tax is 228.80, the withholding is −200.00 and the amount due is 1,068.80.

**Given** a declaration of intent with a threshold of 50,000.00, 38,000.00 invoiced and 9,000.00 planned,
**when** an order of 5,000.00 is prepared under it,
**then** the warning names an overrun of 2,000.00.

**Given** an invoice that has been transmitted,
**when** a second transmission is attempted while the first is running,
**then** it is refused with `This document is being sent by another process already.`

**Given** a counterpart whose destination code is six characters long,
**when** an invoice is prepared for it,
**then** the counterpart is treated as a public administration and the tender and investment identifiers become relevant.
