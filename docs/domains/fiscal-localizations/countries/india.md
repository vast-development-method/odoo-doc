# India

The Indian package supplies one chart of accounts template with a fiscal year ending in March, 428 taxes covering the goods and services tax in its three components, the compensation cess, the tax collected at source and the tax deducted at source, twelve tax groups, six fiscal positions generated at load time from the company's own state, a place of supply carried on every invoice, a goods and services tax treatment carried on every counterpart, a national tax account number shared by several contacts, a commodity code on every product and line, a withholding entry mechanism separate from the invoice, a section threshold alert, an electronic invoice registration flow and a goods movement permit flow.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `in` |
| Display name | India |
| Parent template | none |
| Fiscal country | India |
| Account code length | 6 characters |
| Rounding method written | Round per Line |
| Fiscal year last month | March |
| Print the invoice total in words | true |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `1002` |
| Cash account code prefix | `1001` |
| Transfer account code prefix | `1008` |
| Default point of sale receivable account | `10041` Debtors (point of sale) |
| Gain exchange rate account | `2013` |
| Loss exchange rate account | `2117` |
| Cash discount write-off loss account | `2132` |
| Cash discount write-off gain account | `2012` |
| Fiscal year last month | March |
| Default sale tax | 5 percent goods and services tax, sale |
| Default purchase tax | 5 percent goods and services tax, purchase |
| Deferred expense account | `10084` |
| Deferred revenue account | `10085` |
| Income account | `20011` |
| Expense account | `2107` |
| Withholding account | `100595` |
| Tax calculation rounding method | Round per Line |
| Receivable account recorded as a Contact default | `10040` Debtors |
| Payable account recorded as a Contact default | `11211` Creditors |

**Rounding method.** The country requires each tax to be rounded on each line, because the return reports per line. A worked example: three lines of 1,234.56 with an 18 percent tax give `round(1,234.56 × 0.18, 2) × 3 = 222.22 × 3 = 666.66` under the per-line method, against `round(3,703.68 × 0.18, 2) = 666.66` under the per-tax method; the two agree here, but a base of 10.03 on three lines gives 5.42 per line against 5.41 globally.

### 1.2 Post-load steps specific to the country

1. The company's registration flag is recomputed from its own tax identification number.
2. The two manual payment method lines of every bank journal are re-assigned their outstanding accounts, because the accounts did not exist when the journals were created.
3. When the company has the tax deducted at source feature on, the taxes of the group `TDS` are activated.
4. When the company has the tax collected at source feature on, the taxes of the group `TCS` are activated.

### 1.3 Cash rounding

The template ships one cash rounding rule, "half up", whose profit account is `213202` and whose loss account is `213201`.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 102 |
| Code lengths present | 4, 5 and 6 characters, padded to 6 on load |
| Account groups shipped | none |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 14 |
| Fixed Asset | 8 |
| Receivable | 3 |
| Equity | 1 |
| Expense | 36 |
| Income | 9 |
| Other Income | 1 |
| Current Liability | 27 |
| Payable | 3 |

**The tax accounts.**

| Code | Name | Purpose |
|---|---|---|
| `10051` | State goods and services tax receivable | Input state component |
| `10052` | Central goods and services tax receivable | Input central component |
| `10053` | Integrated goods and services tax receivable | Input integrated component |
| `100531` | Integrated tax paid on special economic zone and export sales | Tax paid on a zero-rated supply made with payment |
| `10054` | Integrated tax special economic zone and export control account | Control account for those supplies |
| `10057` | Reverse charge goods and services tax on purchase | Self-assessed tax |
| `10058` | Tax deducted at source receivable | Amounts others withheld from the company |
| `10059` | Tax current account, receivable | The receivable counterpart of every tax group |
| `100595` | Withholding account | Where a withholding entry posts |
| `11239` | Tax current account, payable | The payable counterpart of every tax group |

**Padding example.** The account whose template code is `10031` is stored as `100310`, because the template pads to six characters on the right with zeros. The account whose template code is `100531` is already six characters and is stored unchanged.

---

## 3. Taxes

428 taxes are shipped. Almost all are shipped archived; the accountant activates the rates the business actually uses, and the two withholding families are activated automatically when their company feature is switched on.

### 3.1 The three components

| Component | Meaning | Tax group | Applies when |
|---|---|---|---|
| Central component | The share of the goods and services tax that belongs to the central government. | `CGST` | The place of supply is in the supplier's own state. |
| State component | The share that belongs to the state or union territory. | `SGST/UTGST` | The place of supply is in the supplier's own state. |
| Integrated component | The whole tax, collected centrally and shared later. | `IGST` | The place of supply is in another state, or the supply is an export or a supply to a special economic zone. |

For an intra-state supply, the rate is split in half between the central and the state components; for an inter-state supply the whole rate is charged as the integrated component.

```
intra-state:  central = round(base × rate ÷ 2 ÷ 100, 2)
              state   = round(base × rate ÷ 2 ÷ 100, 2)
inter-state:  integrated = round(base × rate ÷ 100, 2)
```

**Worked example.** A base of 10,000.00 at 18 percent. Intra-state: central 900.00, state 900.00, total tax 1,800.00. Inter-state: integrated 1,800.00. The totals are identical; the accounts, the report tags and the return section differ.

### 3.2 Rate ladder

The seven published rates are 1, 2, 5, 12, 18, 28 and 40 percent.

**Combined taxes.** For each rate and each direction, a group tax is shipped whose two children are the half-rate central component and the half-rate state component. The group tax is what the user chooses; the two components are what the journal entry carries.

| Group tax | Rate | Children |
|---|---|---|
| 1% goods and services tax, sale | 1 | state 0.5 percent, central 0.5 percent |
| 2% goods and services tax, sale | 2 | state 1.2 percent, central 1.2 percent |
| 5% goods and services tax, sale | 5 | state 2.5 percent, central 2.5 percent |
| 12% goods and services tax, sale | 12 | state 6 percent, central 6 percent |
| 18% goods and services tax, sale | 18 | state 9 percent, central 9 percent |
| 28% goods and services tax, sale | 28 | state 14 percent, central 14 percent |
| 40% goods and services tax, sale | 40 | state 20 percent, central 20 percent |

The same seven exist for purchases, and seven more for purchases under the reverse charge, plus two for sales under the reverse charge (5 and 18 percent).

**Integrated taxes.** 33 sales taxes and 14 purchase taxes carry the integrated component directly, without a group: the seven ordinary rates, the seven special economic zone and export rates paid with tax, the seven under a letter of undertaking, the seven for a special economic zone under a letter of undertaking, and a zero-rate variant of each of the three zero-rated families.

### 3.3 Compensation cess

The cess is charged on top of a goods and services tax, on the same base, and is itself included in the base of nothing. Twelve cess taxes are shipped.

| Form | Computation |
|---|---|
| Percentage | `round(base × cess_rate ÷ 100, 2)` |
| Fixed amount per unit | `round(quantity × amount_per_unit, 2)` |
| Greater of the two | `max(quantity × unit_price × 0.21, quantity × 4.17)` |
| Combination | A group tax whose children are a percentage cess and a per-unit cess, for example "5 percent plus 1.591 per unit" |

**Worked example of the greater-of rule.** 1,000 units at 15.00 each. The percentage part is `1,000 × 15.00 × 0.21 = 3,150.00`; the per-unit part is `1,000 × 4.17 = 4,170.00`. The cess is 4,170.00.

**Worked example of the combination.** 200 units at 50.00 each, cess "5 percent plus 1.591 per unit". The percentage part is `round(10,000.00 × 0.05, 2) = 500.00`; the per-unit part is `round(200 × 1.591, 2) = 318.20`; the total cess is 818.20.

Every cess tax sets "include in base amount" to true so that a later tax in the sequence is computed on the price plus the cess, and sets "base affected by previous taxes" to false so that the cess itself is computed on the untaxed price.

### 3.4 Zero-rated and out-of-scope families

| Family | Meaning | Group |
|---|---|---|
| Nil rated | A supply whose published rate is zero. | Nil Rated |
| Exempt | A supply exempted by notification; the input tax attached to it is not deductible. | Exempt |
| Outside the goods and services tax | A supply the tax does not reach at all, such as alcohol for human consumption. | Non Goods and Services Tax Supplies |

One sales tax and one purchase tax exist for each of the three.

### 3.5 Tax collected at source

39 sales taxes, all percentage, in two groups: the historical group and the current group. A tax collected at source is added to the invoice in addition to the price, and the buyer may claim it back against its own income tax. Each carries a section, which links it to its threshold.

### 3.6 Tax deducted at source

258 taxes, all with the scope "none" so that they cannot be chosen on an invoice line: 69 sale-side and 69 purchase-side in the historical group, 60 sale-side and 60 purchase-side in the current group. They are used by the withholding entry mechanism of section 8. Each carries a section.

### 3.7 Tax fields

| Field on Tax | Values | Meaning |
|---|---|---|
| Reverse charge | boolean | The buyer accounts for the tax instead of the seller. |
| Goods and services tax component | `igst`, `cgst`, `sgst`, `cess`, derived | Which component the tax is, derived from its group. |
| Letter of undertaking | boolean | A zero-rated export tax used under a letter of undertaking. |
| Indian tax kind | `gst`, `tcs`, `tds_sale`, `tds_purchase`, `nil_rated`, `exempt`, `non_gst` | Which family the tax belongs to. |
| Section | many_to_one to India Section Alert | Which section of the law the tax implements. |

---

## 4. Tax groups

Twelve groups, all with the country India, all wired to the payable account `11239` and the receivable account `10059`.

| Group | Ordering | Purpose |
|---|---|---|
| State goods and services tax or union territory tax | default | The state component. |
| Central goods and services tax | default | The central component. |
| Integrated goods and services tax | default | The integrated component. |
| Compensation cess | default | The cess. |
| Goods and services tax | default | The parent of a combined rate. |
| Exempt | default | Exempt supplies. |
| Nil Rated | default | Nil-rated supplies. |
| Non Goods and Services Tax Supplies | default | Out-of-scope supplies. |
| Act 1961, tax collected at source | 100 | Historical collection group. |
| Act 1961, tax deducted at source | 100 | Historical deduction group. |
| Tax collected at source | 90 | Current collection group. |
| Tax deducted at source | 90 | Current deduction group. |

---

## 5. Fiscal positions

Two positions are shipped as data; six more are generated at load time from the company's own state, because the intra-state position must name that state.

### 5.1 Generated at load time

| Sequence | Name | Detection | Taxes it maps to |
|---|---|---|---|
| 1 | Within `<company state name>`, or "Intra State" when the company has no state | automatic; country India; the company's own state | the fourteen combined central-plus-state taxes |
| 2 | Inter State | automatic; the inter-state country group | the fourteen integrated taxes |
| 3 | Special Economic Zone | automatic; country India; the "other country" state | the fourteen integrated taxes |
| 4 | Export | automatic | the seven export integrated sales taxes paid with tax, plus the zero-rate variant, plus the seven integrated purchase taxes |
| 5 | Special Economic Zone under a letter of undertaking, without payment | not automatic; country India; the "other country" state | the seven special economic zone letter-of-undertaking taxes plus the zero-rate variant |
| 6 | Export under a letter of undertaking, without payment | not automatic | the seven export letter-of-undertaking taxes plus the zero-rate variant |

A **branch** company generates only the first two positions; the other four belong to the parent.

**Notes printed on the document.**

```
SUPPLY MEANT FOR EXPORT/SUPPLY TO SEZ UNIT OR SEZ DEVELOPER FOR AUTHORISED OPERATIONS ON PAYMENT OF INTEGRATED TAX.
SUPPLY MEANT FOR EXPORT/SUPPLY TO SEZ UNIT OR SEZ DEVELOPER FOR AUTHORISED OPERATIONS UNDER BOND OR LETTER OF UNDERTAKING WITHOUT PAYMENT OF INTEGRATED TAX.
```

### 5.2 Shipped as data

| Sequence | Name | Note printed |
|---|---|---|
| 10 | Reverse charge Intra State | `THE SUPPLY IS SUBJECT TO REVERSE CHARGE MECHANISM, SO THE RECIPIENT IS RESPONSIBLE FOR PAYING TAX.` |
| 20 | Reverse charge Inter State | the same note |

---

## 6. Place of supply and the goods and services tax treatment

### 6.1 Place of supply

Every invoice carries a **place of supply**, which is a country subdivision, derived from the counterpart's address and editable. It decides which fiscal position applies and therefore whether the rate is split or integrated.

The Country Subdivision record gains a two-character **tax identification prefix**, which is the first two digits of every registration number issued in that subdivision.

**Validation at posting.**

```
Please set a valid TIN Number on the Place of Supply <state name>
```

### 6.2 Treatment

Every counterpart carries a **goods and services tax treatment**, which is copied onto the invoice and frozen there.

| Value | Meaning | Registration number required |
|---|---|---|
| Registered Business, Regular | An ordinary registered business. | yes |
| Registered Business, Composition | A registered business under the composition scheme. | yes |
| Unregistered Business | A business below the registration threshold. | no |
| Consumer | A private individual. | no |
| Overseas | A counterpart outside the country. | no |
| Special Economic Zone | A unit inside a special economic zone. | yes |
| Deemed Export | A supply treated as an export although the goods do not leave the country. | yes |
| Unique identification number holders | Embassies and international bodies. | yes |

**Validation at posting.**

```
Partner <partner name> (<identifier>) GSTIN is required under GST Treatment <treatment name>
```

### 6.3 Registration number verification

When the company has the registration status feature on, a button on the contact checks the counterpart's registration number against the network and stores the result and the date.

| Condition | Message |
|---|---|
| The company is not Indian. | `You must be logged in an Indian company to use this feature` |
| No number is typed. | `Please enter the GSTIN` |
| The feature is off. | `This feature is not activated. Go to Settings to activate this feature.` |
| The network cannot be reached. | `Unable to connect with GST network` |
| The number is not recognised. | `The provided GSTIN is invalid. Please check the GSTIN and try again.` |
| The company has no valid number. | `Please set a valid GST number on company.` |
| No service and production environment is enabled. | `Please ensure that at least one Indian service and production environment is enabled,` |

---

## 7. Commodity code

| Location | Field | Behavior |
|---|---|---|
| Product Template | Commodity code | The harmonised system code for goods or the services accounting code for services. |
| Product Template | Commodity code warning | Derived. Warns when the code is absent or shorter than the company's declared digit count. |
| Journal Item | Commodity code | Derived from the product, editable, stored, not copied. |
| Company | Commodity code digit count | `4` (turnover below the small threshold), `6` (turnover above it), `8`. Derived and editable. |
| Unit of Measure | Unique quantity code | The code the return requires for the quantity. |

**Worked example.** A company whose declared digit count is 6 sells a product whose commodity code is `8471`. The warning names the product and states that the code has 4 digits where 6 are required.

---

## 8. Withholding

### 8.1 Two families

| Family | Direction | Who acts | Where it is recorded |
|---|---|---|---|
| Tax deducted at source | The payer deducts from what it pays. | The buyer on a bill, or the customer on the company's invoice. | A separate journal entry created by the withholding wizard. |
| Tax collected at source | The seller adds to what it charges. | The company on its own invoice. | An ordinary tax line on the invoice. |

### 8.2 Section thresholds

A section carries a per-transaction threshold, a cumulative threshold and the window over which the cumulative threshold is measured (a month or a financial year), and names whether the untaxed amount or the total amount is measured.

**Alert.** When a draft document for a counterpart crosses either threshold, a warning names the section and the amount reached, so that the operator adds the withholding.

**Worked example.** A section with a per-transaction threshold of 30,000.00 and a cumulative annual threshold of 100,000.00, measured on the untaxed amount. The counterpart has already been billed 85,000.00 this financial year. A new bill of 20,000.00 does not cross the per-transaction threshold but brings the cumulative figure to 105,000.00, so the alert fires.

### 8.3 The withholding wizard

Opened from a posted invoice, bill, credit note or refund, or from a payment.

| Input | Meaning |
|---|---|
| Reference | Free reference written on the entry. |
| Related invoice or related payment | The document being withheld against. |
| Journal | The withholding journal, defaulted from the company. |
| Date | The entry date. |
| Deduction regime | Derived from the counterpart's national tax account number entity: normal, lower, higher or no deduction. |
| Section tax | The withholding tax to apply; derived from the lines of the document. |
| Base | Derived from the tax and the document, editable. |
| Amount | Derived: the tax applied to the base. |

**Guards.**

```
TDS must be created from an Invoice or a Payment.
You can only create a withhold for only one record at a time.
TDS must be created from Posted Customer Invoices, Customer Credit Notes, Vendor Bills or Vendor Refunds.
Please set a partner on the <record> before creating a withhold.
Negative or zero values are not allowed in Base Amount for withhold
Negative or zero values are not allowed in TDS Amount for withhold
Please configure the withholding account from the settings
```

**Entry produced.** A vendor bill of 100,000.00 with a 10 percent withholding.

| Line | Account | Debit | Credit |
|---|---|---|---|
| Reduce the payable | Creditors | 10,000.00 | |
| Withholding payable | Withholding account `100595` | | 10,000.00 |

The entry is marked as an Indian withholding entry, carries a link back to the invoice or the payment, and is listed on the invoice under "Indian withholding entries" with a derived total.

**Fields on the invoice.**

| Field | Meaning |
|---|---|
| Is an Indian withholding entry | Marks the entry as produced by the wizard. |
| Withholding reference invoice | The document the entry withholds against. |
| Withholding reference payment | The payment the entry withholds against. |
| Withholding entries | Every entry raised against this document. |
| Withholding lines | The journal items of those entries. |
| Total withholding amount | Their sum. |
| Deduction regime | Related to the counterpart's national tax account number entity. |
| Show higher collection button | Derived; offered when the counterpart has no valid registration and a higher rate applies. |

---

## 9. Reference tables

| Table | Fields | Purpose |
|---|---|---|
| National tax account number entity | number, entity type derived from the fourth character, contacts, deduction regime, certificate, certificate file name, small enterprise class, small enterprise number | Groups every contact that shares one national tax account number and carries the deduction regime and the small enterprise registration that apply to all of them. |
| Port code | code, name, country subdivision | The customs port quoted on an export invoice. |
| Section alert | name, family, measured amount, two thresholds, window, taxes, report line | Thresholds and alerts. |
| Electronic way bill type | name, code, sub-type, sub-type code, allowed movement direction, active | The classes the permit portal accepts. |
| Optional holiday | name, date, company | A date an employee may choose as a restricted holiday. Deleting one that a leave request uses is refused: `You cannot delete an Optional Holiday that is linked to a leave request.` |

**National tax account number validation.** Ten characters: five letters, four digits and one letter, the fourth character naming the entity type.

```
The entered PAN <value> seems invalid. Please enter a valid PAN.
A PAN Entity with same PAN Number already exists.
```

---

## 10. Export fields on the invoice

| Field | Meaning |
|---|---|
| Shipping bill number | The customs document number. |
| Shipping bill date | Its date. |
| Port code | The customs port. |
| Reseller | A registered reseller, for a sale made on its behalf. Only counterparts with a tax identification number may be chosen. |
| Registration number | The counterpart's registration number frozen on the invoice. |

---

## 11. Return sections

Every journal item of a sale carries a **return section** that decides which part of the outward supplies return it feeds.

| Value | Meaning |
|---|---|
| Business to business under reverse charge | Registered counterpart, reverse charge. |
| Business to business regular | Registered counterpart, ordinary. |
| Business to consumer, large | Unregistered counterpart, inter-state, above the reporting threshold. |
| Business to consumer, small | Unregistered counterpart, below the threshold, reported in aggregate. |
| Export with payment | Export on which integrated tax was paid. |
| Export without payment | Export under a letter of undertaking. |
| Special economic zone with payment | Supply to a zone on which tax was paid. |
| Special economic zone without payment | Supply to a zone under a letter of undertaking. |
| Deemed export | Supply treated as an export. |
| Credit or debit note, registered, reverse charge | Correction of a reverse-charge supply. |
| Credit or debit note, registered, regular | Correction of an ordinary registered supply. |
| Credit or debit note, deemed export | Correction of a deemed export. |

---

## 12. Electronic invoice registration

| Phase | Behavior |
|---|---|
| Eligibility | The company has the electronic invoicing feature on, the invoice is a customer invoice or credit note, the counterpart is registered, and the invoice is posted. |
| Build | The payload is produced from the invoice, with the place of supply, the commodity codes, the three tax components and the export fields. |
| Transmit | The payload is sent through the accredited service provider. The company's credentials and the session token are held on the company, readable only by the administration group. |
| Answer | The registration number, the registration date and the visual code are stored, and the signed payload is attached. |
| Cancel | A cancellation requires a reason and free remarks, and is only accepted inside the portal's window. |

**Fields on the invoice.**

| Field | Values |
|---|---|
| Electronic invoicing status | `to_send` (To Send), `sent` (Sent), `cancelled` (Cancelled) |
| Attachment and file | The registered payload. |
| Cancel reason and cancel remarks | Required to cancel. |
| Content | The payload that would be sent, for inspection. |
| Error | The portal's rejection details as rich text. |

**State machine.**

| From | Trigger | Guard | To |
|---|---|---|---|
| (none) | Post an eligible invoice | eligibility | To Send |
| To Send | Send | credentials valid, no validation error | Sent |
| To Send | Send | validation error | To Send, with the error stored |
| Sent | Cancel | a reason and remarks are given; the window is open | Cancelled |

---

## 13. Goods movement permit

The permit is a separate record, fully specified in [../entities.md](../entities.md) section 3.9. Its lifecycle is Pending, Generated, Cancelled, with an additional Delivery Challan state when the movement travels under a delivery note instead of a permit.

**Two ways of obtaining it.**

1. **Direct.** A payload is built from the invoice or the transfer and sent to the permit portal.
2. **From the registered invoice.** When the invoice already carries a registration number, the permit is requested by quoting that number instead of re-sending the whole document. When the registration is not yet available:

```
waiting for IRN generation to create E-waybill
```

**Validity.** The expiry date is derived from the distance declared: the portal grants one day per fixed block of kilometres. The distance is therefore a required input and is tracked.

**Vehicle type rule.** When the transport mode is ship, the vehicle type is forced to over-dimensional cargo.

---

## 14. Company settings

| Setting | Type | Default | Effect |
|---|---|---|---|
| Registered under the goods and services tax | boolean, derived from the parent company, editable, stored | derived | Reveals the whole tax section. |
| Commodity code digit count | selection `4`, `6`, `8` | derived | Drives the product warning. |
| Instant payment address | text | empty | Printed as a payment visual code on the invoice. |
| Production environment | boolean, readable only by the administration group | true | Which portal environment is used. |
| Tax deducted at source feature | boolean, derived from the parent, editable, stored | derived | Activates the deduction taxes and the wizard. |
| Tax collected at source feature | boolean, derived from the parent, editable, stored | derived | Activates the collection taxes. |
| Withholding account | many_to_one to Account, company-checked | `100595` | Where a withholding entry posts. |
| Withholding journal | many_to_one to Journal, company-checked | empty | The journal of a withholding entry. |
| Tax deduction account number | text, mirrored on the company contact | empty | The national deduction account number. |
| National tax account number | many_to_one to India Permanent Account Number Entity, mirrored on the company contact, stored | empty | The company's own number. |
| National tax account number type | selection, related | derived | The entity class. |
| Registration status feature | boolean | false | Enables the counterpart lookup. |
| Service provider | selection over the accredited providers | empty | Which provider is used; changing it invalidates the stored tokens. |
| Manage reseller | boolean, backed by an access group | false | Reveals the reseller field on invoices. |
| Restrictive audit trail | boolean, forced on | true | Cannot be switched off. |

---

## 15. Acceptance scenarios

**Given** a company in India with no accounting and the state Maharashtra,
**when** the Indian template is loaded,
**then** the fiscal year ends in March, the rounding method is Round per Line, 102 accounts exist, twelve tax groups exist, and six fiscal positions exist of which the first is named "Within Maharashtra" and names Maharashtra as its state.

**Given** a branch company under an Indian parent,
**when** the template is loaded on the branch,
**then** only the intra-state and the inter-state fiscal positions are created for it.

**Given** a customer in the company's own state and an invoice line of 10,000.00 carrying the 18 percent combined tax,
**when** the totals are computed,
**then** the entry carries a central component of 900.00 and a state component of 900.00.

**Given** a customer in another state and the same line,
**when** the fiscal position "Inter State" is applied,
**then** the line carries the 18 percent integrated tax and the entry carries a single tax line of 1,800.00.

**Given** a line of 1,000 units at 15.00 carrying the cess "21 percent or 4.170 per unit, whichever is greater",
**when** the cess is computed,
**then** it is 4,170.00.

**Given** a counterpart whose treatment is Registered Business, Regular and who has no registration number,
**when** an invoice for that counterpart is posted,
**then** the posting is refused with the registration-required message naming the counterpart.

**Given** a posted vendor bill of 100,000.00 and a 10 percent deduction section,
**when** the withholding wizard is confirmed,
**then** a separate entry debits Creditors 10,000.00 and credits the withholding account 10,000.00, is marked as a withholding entry and is linked back to the bill.

**Given** a section with a cumulative annual threshold of 100,000.00 measured on the untaxed amount and a counterpart already billed 85,000.00 this year,
**when** a bill of 20,000.00 is drafted,
**then** the alert fires naming the section and the cumulative figure of 105,000.00.

**Given** a permit whose transport mode is set to ship,
**when** the record is saved,
**then** the vehicle type is over-dimensional cargo.

**Given** a generated permit,
**when** deletion is attempted,
**then** it is refused with `You cannot delete a generated E-waybill. Instead, you should cancel it.`
