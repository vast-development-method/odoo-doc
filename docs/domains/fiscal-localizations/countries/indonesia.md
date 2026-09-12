# Indonesia

The Indonesian package supplies one chart of accounts template with 114 accounts, 78 tax rows and
six tax groups, a payment code scheme that produces a scannable payment instruction for an invoice
or a receipt, and an electronic invoice document that gathers customer invoices, classifies each
product and each unit of measure with the administration's codes, and produces the payload the
taxpayer uploads.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `id` |
| Display name | Indonesia |
| Parent template | none |
| Fiscal country | Indonesia |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Indonesian rupiah |
| Default sale tax | the 11 percent sales tax |
| Default purchase tax | the 11 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 114 |
| Tax rows shipped | 78 |
| Tax group rows shipped | 6 |
| Fiscal position rows shipped | none |

---

## 3. Taxes

| Family | Rate | Scope |
|---|---|---|
| Domestic sales | 11 percent | sale |
| Domestic purchases | 11 percent | purchase |
| Luxury goods sales tax | the published rate per class | sale and purchase |
| Withholding on services | negative rates | purchase |
| Zero rated | 0 percent | sale |
| Exempt | none | sale and purchase |

**Worked example.** A sale of 10,000,000 rupiah at 11 percent produces a tax of 1,100,000 and a
total of 11,100,000. The Indonesian rupiah has no decimal places, so every amount is rounded to a
whole unit.

---

## 4. The payment code scheme

### 4.1 Company and bank account configuration

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_id_qris_api_key` | Payment scheme key | Company | text | The credential issued to the merchant. |
| `l10n_id_qris_mid` | Merchant identifier | Company | text | The merchant number the scheme allocates. |

### 4.2 Indonesia Quick Response Transaction

One record per code presented to a customer. Its fields and its two-state lifecycle are specified in
[entities.md](../entities.md) and [state-machines.md](../state-machines.md#192-indonesia-quick-response-transaction).
The essentials:

- The amount is stored as a whole number of rupiah, because the scheme accepts no fraction.
- The record names the record it settles, by kind and by identifier, and the bank account that
  produced the code.
- A status check asks the scheme whether the code was paid; when it was, the payment is recorded
  against the invoice or the receipt.
- A code that is still unpaid thirty-five minutes after its creation is removed by the clean-up job,
  because it can no longer be paid.
- Generating a code for a record kind the mechanism does not cover is refused with "QRIS capability
  is not extended to model %s yet!".

---

## 5. The electronic invoice document

### 5.1 Company configuration

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_id_tku` | Branch number | text | The branch number of the company; empty for the head office. |

### 5.2 Contact fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_id_buyer_document_type` | Buyer document type | selection, default `TIN` | `TIN` (tax identification number), `NIK` (national identity number), `Passport`, `Other`. |
| `l10n_id_buyer_document_number` | Buyer document number | text | The number of that document. |
| `l10n_id_nik` | National identity number | text | The buyer's national identity number when it differs from the document number. |
| `l10n_id_pkp` | Registered taxable person | boolean | Derived from the country code and the tax identification number; true when the counterpart is a registered taxable person. |

### 5.3 Invoice fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_id_kode_transaksi` | Transaction code | selection, default `04` | The first two digits of the tax code, derived from the counterpart and overridable. |
| `l10n_id_coretax_document` | Electronic invoice document | link to Indonesia Electronic Invoice Document | The document the invoice was gathered into. |
| `l10n_id_coretax_efaktur_available` | Document available | boolean | Derived from the taxes of the lines and the counterpart; true when the invoice qualifies. |
| `l10n_id_coretax_add_info_07` and `l10n_id_coretax_add_info_08` | Additional information for codes 07 and 08 | selection | The additional information the administration requires for the two facility codes. |
| `l10n_id_coretax_facility_info_07` and `l10n_id_coretax_facility_info_08` | Facility information for codes 07 and 08 | selection | The facility the exemption relies on. |
| `l10n_id_coretax_custom_doc` | Custom document | text | Supporting documentation required with code 07 or 08. |
| `l10n_id_coretax_custom_doc_month_year` | Custom document month and year | date | The period the supporting documentation covers. |

### 5.4 Product and unit codes

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_id_product_code` | Product code | Product Template | link to Indonesia Product Code | The administration's goods or services classification. |
| Unit code | Unit code | Unit of Measure | link to Indonesia Unit of Measure Code | The administration's unit code. |

### 5.5 The document lifecycle

An Indonesia Electronic Invoice Document gathers posted customer invoices and credit notes of one
company that are not already on another document. Its lifecycle and its two refusal messages are in
[state-machines.md](../state-machines.md#191-indonesia-electronic-invoice-document):

> "Some documents don't have a transaction code: `<list of invoice numbers>`"

> "Some documents are not Customer Invoices: `<list of invoice numbers>`"

Once generated, the payload is attached to the document and downloaded for upload to the
administration's own portal. Regenerating replaces the attachment.

---

## 6. Point of sale

The point of sale package adds the payment code scheme as a payment method: the cashier presents the
code, the status check confirms the payment, and the receipt is closed. The transaction records are
the same ones as on an invoice.

---

## 7. Acceptance scenarios

**Given** a company in Indonesia with no accounting,
**when** the Indonesian template is loaded,
**then** 114 accounts, 78 tax rows and six tax groups exist.

**Given** a sale of 10,000,000 rupiah at 11 percent,
**when** the totals are computed,
**then** the tax is 1,100,000 and the total is 11,100,000, with no decimal places.

**Given** an electronic invoice document holding two invoices, one of which has no transaction code,
**when** the payload is generated,
**then** it is refused with "Some documents don't have a transaction code: `<list of invoice
numbers>`" and no attachment is produced.

**Given** a payment code created for an invoice and paid,
**when** the status check runs,
**then** the transaction is marked paid and the payment is recorded against the invoice.

**Given** a payment code created 36 minutes ago and still unpaid,
**when** the clean-up job runs,
**then** the transaction record is removed.
