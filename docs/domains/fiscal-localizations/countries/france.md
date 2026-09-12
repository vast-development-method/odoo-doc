# France

The French package supplies one chart of accounts template of 656 accounts and 182 account groups built on the national accounting plan, together with six derived templates for the overseas departments and for Monaco, six tax groups, ten fiscal positions of which six are the per-territory domestic positions, a pair of rounding difference accounts, the statutory accounting entries export, point of sale certification with an inalterability chain and three immutable periodic closings, and the public invoicing portal flows: the regulatory data flow, the mandatory lifecycle status flow and the periodic transaction and payment reporting flow.

---

## 1. Templates

| Template code | Display name | Territory |
|---|---|---|
| `fr` | France | Mainland France |
| `mc` | Monaco | Monaco |
| `gf` | French Guiana | French Guiana |
| `gp` | Guadeloupe | Guadeloupe |
| `mq` | Martinique | Martinique |
| `re` | Réunion | Réunion |
| `yt` | Mayotte | Mayotte |

Each overseas template reuses the mainland chart and differs in its rate ladder, because the overseas departments have their own reduced rates and Mayotte is outside the value-added tax field altogether.

### 1.1 Company-level values written by the mainland template

| Company field | Value |
|---|---|
| Fiscal country | France |
| Account code length | 6 characters |
| Bank account code prefix | `512` |
| Cash account code prefix | `53` |
| Transfer account code prefix | `58` |
| Default point of sale receivable account | the point of sale receivable account |
| Gain exchange rate account | `766` |
| Loss exchange rate account | `666` |
| Bank suspense account | `471` |
| Cash discount write-off loss account | `665` |
| Cash discount write-off gain account | `765` |
| Deferred expense account | `486` |
| Deferred revenue account | `487` |
| Rounding difference loss account | `658` |
| Rounding difference profit account | `758` |
| Default sale tax | the 20 percent output tax |
| Default purchase tax | the 20 percent input tax |
| Income account | `707` |
| Expense account | `607` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `31` |
| Receivable account recorded as a Contact default | the trade receivable account |
| Payable account recorded as a Contact default | the trade payable account |
| Down payment account | `4191` |

### 1.2 Journals

The sales journal and the purchase journal are created with separate credit note numbering switched on.

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 656 |
| Account groups shipped | 182 |
| Code lengths present | 5 and 6 characters |
| Translations shipped | French |

**Account type distribution.**

| Account type | Count |
|---|---|
| Bank and Cash | 28 |
| Current Asset | 88 |
| Fixed Asset | 114 |
| Receivable | 6 |
| Equity | 28 |
| Expense | 190 |
| Income | 89 |
| Current Liability | 58 |
| Non-current Liability | 47 |
| Payable | 8 |

Several accounts ship archived, so that a company activates only the ones its plan uses.

---

## 3. Taxes and tax groups

| Group | Rate | Payable account | Receivable account |
|---|---|---|---|
| Value-added tax 0% | 0 | `44551` | `44567` |
| Value-added tax 2.1% | 2.1 | `44551` | `44567` |
| Value-added tax 5.5% | 5.5 | `44551` | `44567` |
| Value-added tax 8.5% | 8.5 | `44551` | `44567` |
| Value-added tax 10% | 10 | `44551` | `44567` |
| Value-added tax 20% | 20 | `44551` | `44567` |

The 8.5 percent rate is the overseas department rate; the overseas templates make it their standard rate.

Each rate exists as an output tax and an input tax, and the input taxes are split between goods, services and capital goods, because the return separates them. Intra-union acquisitions, imports and the reverse charge each have their own pair that charges and deducts the same amount.

---

## 4. Fiscal positions

| Sequence | Name | Detected automatically | Requires a tax number | Country or group | Active on install |
|---|---|---|---|---|---|
| 10 | Domestic, France, Monaco and the overseas departments | yes | no | the France, Monaco and overseas department group | yes |
| 20 | Domestic, Monaco | yes | no | Monaco | no |
| 21 | Domestic, French Guiana | yes | no | French Guiana | no |
| 22 | Domestic, Guadeloupe | yes | no | Guadeloupe | no |
| 23 | Domestic, Martinique | yes | no | Martinique | no |
| 24 | Domestic, Réunion | yes | no | Réunion | no |
| 25 | Domestic, Mayotte | yes | no | Mayotte | no |
| 30 | Intra-union, business to business | yes | yes | the European Union group without Monaco | yes |
| 40 | European Union private | yes | no | the same group | yes |
| 50 | Import and export outside Europe and the overseas territories | yes | no | none | yes |

The six per-territory positions ship archived; a company established in one of those territories activates its own and archives the general domestic position, because the rate ladder differs.

**Why a group without Monaco.** Monaco is inside the French value-added tax field but outside the union for the purpose of the intra-union rules, so the union group used by the two intra-union positions must exclude it.

---

## 5. Rounding difference accounts

| Setting | Default | Effect |
|---|---|---|
| Rounding difference loss account | `658` | Receives a negative rounding difference. |
| Rounding difference profit account | `758` | Receives a positive rounding difference. |

---

## 6. Statutory accounting entries export

A wizard produces the file the tax administration requires for an audit.

| Input | Type | Default | Meaning |
|---|---|---|---|
| Start date | date, required | from the report period | First day exported. |
| End date | date, required | from the report period | Last day exported. |
| Export kind | selection: official (posted entries only), non-official (posted and unposted entries) | official | Which entries are included. |
| Excluded journals | many_to_many to Journal, restricted to the company tree | empty | Journals to leave out. |
| Test file | boolean | false | Marks the file as a test. |
| File name | text of at most 256 characters, read-only | derived | The name the administration expects. |

**Content.** One row per journal item, with the journal code and name, the entry number and date, the account code and name, the counterpart code and name, the document reference and date, the label, the debit, the credit, the reconciliation reference and date, the validation date, the amount in the document currency and the currency code.

**Opening row.** Accounts whose balance is not carried forward into a new year are aggregated into a single opening row for unaffected earnings, because the file must show one opening line rather than one per account.

**Overseas territories.** The overseas departments are outside the union's fiscal territory and their companies have no mainland registration number; the export omits the number for them.

---

## 7. Point of sale certification

### 7.1 The chain

Every point of sale order of a French company carries:

| Field | Type | Meaning |
|---|---|---|
| Inalterability fingerprint | text, read-only, not copied | Computed over the order's immutable fields and the previous order's fingerprint. |
| Gap-free sequence number | integer, read-only, not copied | The order's position in the company's chain. |
| Text to hash | text, derived, not stored | The exact string the fingerprint is computed over, exposed so an auditor can reproduce it. |
| Previous order | many_to_one to Point of Sale Order, derived, stored, read-only, not copied | The order whose fingerprint this one chains to. |
| Producing version | text, read-only, not copied | The version of the software that created the order. |

**Guards.**

```
You have to set a country in your company setting.
An error occurred when computing the inalterability. Impossible to get the unique previous posted point of sale order.
According to the French law, you cannot modify a point of sale order. Forbidden fields: <fields>.
According to the French law, you cannot modify a point of sale order line. Forbidden fields: <fields>.
You cannot overwrite the values ensuring the inalterability of the point of sale.
According to French law, you cannot delete a point of sale order.
You cannot modify a fiscal position used in a POS order.
Accounting is not unalterable for the company <company>. This mechanism is designed for companies where accounting is unalterable.
Please contact your accountant to print the Hash integrity result.
```

### 7.2 The closings

Three scheduled jobs produce an immutable Sale Closing per company: daily, monthly and annual.

| Field | Meaning |
|---|---|
| Name | The frequency and the unique sequence number. |
| Closing start and closing end | The interval covered. The start is the end of the previous closing of the same frequency, or the beginning of the company's activity. |
| Frequency | Daily, Monthly, Annual. |
| Period total | The total posted to receivable accounts during the interval, excluding overlapping periods. |
| Cumulative grand total | The total posted to receivable accounts since the company began. |
| Sequence number | A gap-free counter drawn from the company's closing numbering series. |
| Last order and its fingerprint | The last receipt included and the fingerprint that anchors the chain. |

**Immutability.**

```
Sale Closings must never be modified or deleted under any circumstances.
```

**Worked example.** A company whose first day of trading produced 12,400.00 of receipts gets a daily closing with a period total of 12,400.00 and a cumulative grand total of 12,400.00 and the sequence number 1. The next day, which produced 9,600.00, gets a period total of 9,600.00 and a cumulative grand total of 22,000.00 and the sequence number 2. The monthly closing at the end of that month has its own sequence number 1 in its own frequency and a period total equal to the month's receipts.

### 7.3 Integrity report

A printed report recomputes the whole chain and reports the first divergence, naming the order, the stored fingerprint and the recomputed fingerprint. Printing it requires an accounting user.

---

## 8. Public invoicing portal

### 8.1 Company settings

| Setting | Type | Default | Effect |
|---|---|---|---|
| Send to the portal | boolean | true | Activates the regulatory data flow, the mandatory status flow and the periodic reporting flow. |
| Pilot phase | boolean | false | Participates in the pilot before the obligation begins. |
| Directory start date | date | empty | When the company was registered in the national directory. |
| Approved platform registered | boolean, derived | derived | Whether registration completed. |
| Portal identifier | text, derived and invertible | derived | The company's identifier on the portal. |
| Reporting periodicity | selection: real monthly normal regime, real normal quarterly regime, simplified regime monthly, franchised regime two-monthly | empty | Fixes the period and the due window of each reporting flow. |
| Reporting enabled | boolean, derived, read-only | derived | Whether periodic flows are produced. |
| Reporting start date | date, derived | derived | The first period reported. |
| Identity verification status | selection: processing, success, fail | empty | Where the identity check stands. |
| Authentication identifier | text, readable only by the accounting invoicing group | empty | The authentication reference. |
| Activity code | text | empty | The national activity code, printed on documents. |
| Closing numbering series | many_to_one to Sequence, read-only | created on load | Numbers the sale closings. |

### 8.2 Invoice fields

| Field | Values |
|---|---|
| Portal invoice status | In Progress, Sent, Done, Error |
| Portal lifecycle status | In Progress, Sent, Done, Error |
| Lifecycle residual | The amount of collected money still to be reported. |
| Exchange network status | Extended with a "With Payments" value and the portal's own statuses. |
| Reporting status | Out of scope, Pending, Error, plus the open and sent states of the flow it belongs to. |
| Reporting type | Transaction or Payment. |
| Reporting operation type | Sale or Purchase. |
| Reporting error message | The blocking validation errors. |
| Reporting has error | Whether the entry blocks its flow. |
| Sent in reporting flows | The flows that included it. |
| Last reporting flow | The most recent one, tracked. |

### 8.3 Reporting flow

The France Reporting Flow record is fully specified in [../entities.md](../entities.md) section 3.12. A daily scheduled job builds the flows of the current period per company, and a second job transmits the mandatory lifecycle statuses of previously sent invoices every twelve hours. A third job retrieves inbound regulatory documents every four hours.

**Period status.**

```
period_status = "open"   when today < due_period_start
period_status = "grace"  when due_period_start ≤ today ≤ due_period_end
period_status = "closed" when today > due_period_end
```

Only a flow in the grace window may be sent on its normal path.

**Worked example.** A company on the real monthly normal regime reports the transactions of March. The period runs from 1 March to 31 March; the due window runs from 1 April to the statutory filing day of April. Before 1 April the flow is Open and may be built but not sent; inside the window it is in Grace and may be sent; after the window it is Closed and a rectificative flow is required instead.

**Sending with invalid entries.** The ordinary send button refuses when any entry carries a blocking error:

```
This flow still contains invoices with validation errors. Fix them or use the 'Send without invalid invoices' button.
```

The alternative button opens a wizard that states `Some invoices were excluded due to validation errors.` and sends the flow without them.

**Guards.**

```
You cannot delete sent flows.
Flow <name> has already been sent.
No active PDP proxy user is configured for company <company>.
The flow payload is missing. Build the payload before sending.
The PDP proxy did not return a flow tracking identifier.
```

### 8.4 Point of sale reporting

The point of sale bridge adds till receipts to the periodic reporting flow, because a receipt to a private consumer is reported in the transaction flow rather than transmitted as an invoice.

---

## 9. Leave management for part-time workers

Two packages adapt leave accounting for part-time workers, adding a reference leave type on the company that fixes how a day of leave is measured for someone who does not work a full week. The mechanism belongs to [Time Off](../../time-off/README.md); the localization only supplies the company setting and the reference type.

---

## 10. Acceptance scenarios

**Given** a company in France with no accounting,
**when** the French template is loaded,
**then** 656 accounts and 182 account groups exist, the bank prefix is `512`, the cash prefix is `53`, the transfer prefix is `58`, the bank suspense account is `471`, the rounding difference accounts are `658` and `758`, six tax groups exist all pointing at `44551` and `44567`, and the sales and purchase journals number credit notes separately.

**Given** a company established in Réunion,
**when** the Réunion template is loaded,
**then** the domestic fiscal position for Réunion is active and the standard rate is 8.5 percent.

**Given** a posted point of sale order,
**when** any of its immutable fields is edited,
**then** the edit is refused with the forbidden-fields message naming them.

**Given** two trading days producing 12,400.00 and 9,600.00,
**when** the daily closings are produced,
**then** the first has a period total of 12,400.00 and a cumulative grand total of 12,400.00, and the second has 9,600.00 and 22,000.00.

**Given** a reporting flow whose due window has not opened,
**when** the send button is used,
**then** the flow is in the Open status and may not be sent.

**Given** a reporting flow containing two invoices with blocking errors,
**when** the ordinary send button is used,
**then** it is refused and the alternative button offers to send without them.
