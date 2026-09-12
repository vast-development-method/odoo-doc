# Malaysia

The Malaysian package supplies one chart of accounts template with 77 accounts, 140 tax rows, ten
tax groups and five fiscal positions, a customs tariff code on products, and a portal integration
that submits every invoice, consolidates small point of sale receipts into one periodic document,
validates counterpart identification numbers, and gives the buyer seventy-two hours to reject a
document and the issuer the same window to cancel it.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `my` |
| Display name | Malaysia |
| Parent template | none |
| Fiscal country | Malaysia |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Malaysian ringgit |
| Default sale tax | the 10 percent sales tax |
| Default purchase tax | the 10 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 77 |
| Tax rows shipped | 140 |
| Tax group rows shipped | 10 |
| Fiscal position rows shipped | 5 |

Seventy-seven accounts is the smallest chart of the domain among the countries with a file of their
own; the Malaysian chart is deliberately a starting point that a business extends.

---

## 3. Taxes

| Family | Rate | Scope |
|---|---|---|
| Sales tax on goods | 10 percent and 5 percent | sale |
| Service tax | 8 percent and 6 percent | sale |
| Purchases | the matching rates | purchase |
| Zero rated | 0 percent | sale and purchase |
| Exempt | none | sale and purchase |
| Tourism tax | a fixed amount per night | sale |
| Low-value goods tax | 10 percent | sale |

**Worked example.** A sale of goods of 2,000.00 at 10 percent produces a tax of 200.00 and a total
of 2,200.00. A service of 2,000.00 at 8 percent produces 160.00 and a total of 2,160.00.

---

## 4. Statutory reporting

One report structure is shipped, the periodic sales and service tax return, whose columns are the
rate, the quantity sold and the amount, and whose sections follow the boxes of the paper form.

---

## 5. Fields added

### 5.1 Company

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `sst_registration_number` | Sales and service tax registration number | text | The registration the return is filed under. |
| `ttx_registration_number` | Tourism tax registration number | text | The separate registration for the tourism levy. |
| `l10n_my_edi_mode` | Portal operating mode | selection, default `test` | `test` (Pre-Production) or `prod` (Production). |
| `l10n_my_edi_proxy_user_id` | Portal proxy user | link to the exchange proxy user | The credential holder. |
| `l10n_my_edi_default_import_journal_id` | Default import journal | link to Journal | Where invoices fetched from the portal become vendor bills. |

### 5.2 Contact

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_my_identification_type` | Identification type | selection | Which national document identifies the counterpart. |
| `l10n_my_identification_number` | Identification number | text | The number of that document. A placeholder derived from the chosen type shows the expected shape. |
| `l10n_my_edi_industrial_classification` | Industry classification | link to Malaysia Industry Classification | The published industry code of the counterpart. |
| `l10n_my_tin_validation_state` | Identification validation state | selection | `valid` or `invalid`, or empty when never checked. |

### 5.3 Product

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_my_tax_classification_code` | Customs tariff code | text | The unique identifier used to classify an item for official declaration. |
| `l10n_my_edi_classification_code` | Portal classification code | selection | The portal's own classification, whose stored values are the three-digit codes of the published list, from `001` (breastfeeding equipment) and `002` (child care centres) onwards. |

### 5.4 Tax

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_my_tax_type` | Tax type | selection, derived from the amount and the country and overridable | The portal's classification of the levy the tax represents. |
| `l10n_my_tax_exemption_reason` | Tax exemption reason | text | Used when a consolidated document includes exempt supplies. |

### 5.5 Invoice

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_my_edi_document_ids` | Portal documents | collection of Malaysia Interchange Document | Every document the invoice has travelled in. |
| `l10n_my_edi_state` | Portal state | selection, derived from those documents and stored | `in_progress`, `valid`, `rejected`, `invalid`, `cancelled`. |
| `l10n_my_invoice_need_edi` | Portal submission required | boolean, derived | True when the company, the country, the state and the document kind all call for a submission. |
| `l10n_my_edi_exemption_reason` | Buyer exemption reason | text | The buyer's exemption certificate number or the gazette order it relies on. |
| `l10n_my_edi_custom_form_reference` | Customs form reference | text | The reference of the customs form the supply relates to. |
| `l10n_my_edi_display_tax_exemption_reason` | Show the exemption reason | boolean, derived | Whether the field is relevant for the taxes on the invoice. |

---

## 6. The portal

### 6.1 Malaysia Interchange Document

One document carries one or more invoices, or one consolidated batch of point of sale receipts. It
has its own numbering series, a discussion thread and scheduled activities, and its state machine is
in [state-machines.md](../state-machines.md#17-malaysia-portal).

### 6.2 Consolidation

Receipts below the threshold at which a full invoice is required are gathered into one document per
period by the consolidation wizard, which takes a start date, an end date and the consolidation
kind. It refuses with "Invalid Operation. No order to consolidate." when the period holds nothing,
and with "Support for consolidated invoices in the invoicing app is not yet implemented." when the
selection is not point of sale receipts.

### 6.3 Status window

The document status may be changed for seventy-two hours after the validation moment: the issuer may
cancel and the buyer may reject, in both cases with a reason. Afterwards a debit or credit note is
the only remedy, and the attempt is refused with the message quoted in
[state-machines.md](../state-machines.md#17-malaysia-portal).

### 6.4 Polling

After a submission the status is asked up to three times, one second apart, and the loop stops as
soon as no document of the submission is still being validated. Afterwards the scheduled job takes
over; a valid document is not asked about again for an hour.

---

## 7. The international invoice profile

A separate package builds the international invoice payload profile Malaysia has adopted, which is
what a Malaysian company exchanges with a counterpart on the international network rather than with
the portal. It adds the two registration numbers to the payload.

---

## 8. Acceptance scenarios

**Given** a company in Malaysia with no accounting,
**when** the Malaysian template is loaded,
**then** 77 accounts, ten tax groups and five fiscal positions exist.

**Given** a sale of goods of 2,000.00 at 10 percent,
**when** the totals are computed,
**then** the tax is 200.00 and the total is 2,200.00.

**Given** a counterpart whose identification validation state is `invalid`,
**when** an invoice to it is submitted,
**then** the submission is refused and the reason is posted.

**Given** a document validated eighty hours ago,
**when** a cancellation is requested,
**then** it is refused with "It has been more than 72h since the document validation, you can no
longer cancel it.\nInstead, you should issue a debit or credit note.".

**Given** a document in validation carrying two invoices,
**when** the portal answers that it is invalid,
**then** the document's state becomes `invalid` and both invoices are cancelled.

**Given** a period with no point of sale receipt below the threshold,
**when** the consolidation wizard is run,
**then** it is refused with "Invalid Operation. No order to consolidate.".
