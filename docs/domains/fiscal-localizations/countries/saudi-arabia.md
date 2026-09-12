# Saudi Arabia

The Saudi package supplies one chart of accounts template with 169 accounts, 174 tax rows, five tax
groups and two fiscal positions, the shared Gulf invoice layout, a withholding regime, an adjustment
reason on every credit note, a visual code on every invoice, and an onboarding and signing flow with
the tax authority in which the company obtains a compliance certificate and then a production
certificate, and every invoice is chained to the previous one by a hash.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `sa` |
| Display name | Saudi Arabia |
| Parent template | none |
| Fiscal country | Saudi Arabia |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Saudi riyal |
| Default sale tax | the 15 percent sales tax |
| Default purchase tax | the 15 percent purchase tax |
| Gulf bilingual invoice | true |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 169 |
| Tax rows shipped | 174 |
| Tax group rows shipped | 5 |
| Fiscal position rows shipped | 2 |

---

## 3. Taxes

| Family | Rate | Scope |
|---|---|---|
| Domestic sales | 15 percent | sale |
| Domestic purchases | 15 percent | purchase |
| Zero rated | 0 percent | sale and purchase |
| Exempt | none | sale and purchase |
| Reverse charge on imported services | 15 percent | purchase |
| Withholding on payments to non-residents | negative rates | purchase |

**Worked example.** A sale of 1,000.00 at 15 percent produces a tax of 150.00 and a total of
1,150.00.

---

## 4. Statutory reporting

Two report structures are shipped: the periodic value-added tax return, whose columns are the amount
and the tax amount, and the withholding return, which lists the amounts withheld from non-resident
suppliers.

---

## 5. The Gulf invoice layout

The shared Gulf package gives the invoice a bilingual layout, a per-line tax amount and a per-line
description in both languages.

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_gcc_dual_language_invoice` | Bilingual invoice | Company and Journal Entry | boolean | Whether the invoice is printed in both languages. |
| `l10n_gcc_invoice_tax_amount` | Line tax amount | Journal Item | decimal, derived from the line subtotal and the line total | The tax borne by the line, printed in its own column. |
| `l10n_gcc_line_name` | Line description | Journal Item | text, derived from the line name | The description as it is printed, in both languages. |
| `l10n_gcc_country_is_gcc` | Counterpart is in the Gulf | Journal Entry and Company | boolean, derived from the counterpart's country group | Drives the treatment of a supply to another Gulf state. |

---

## 6. Adjustment reasons and the visual code

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_sa_reason` | Adjustment reason | Journal Entry and the reversal wizard | selection | Required on a credit or debit note. The stored values are the published reason codes, of which the first is `BR-KSA-17-reason-1`, "Cancellation or suspension of the supplies after its occurrence either wholly or partially". |
| `l10n_sa_show_reason` | Show the reason | Journal Entry | boolean, derived | Whether the field applies to this document. |
| `l10n_sa_qr_code_str` | Visual code | Journal Entry | text, derived from the signed and total amounts, the company and the confirmation moment | The value the printed visual code encodes. |
| `l10n_sa_confirmation_datetime` | Issue moment | Journal Entry | date and time, read-only | The moment the invoice became final. |

The visual code is built as a sequence of fields, each a tag byte, a length byte and the value's
bytes, concatenated and encoded in base 64. The arithmetic and a worked example are in
[calculations.md](../calculations.md).

---

## 7. Onboarding and signing

### 7.1 Company and journal configuration

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_sa_csr` | Certificate signing request | Journal | binary | The request submitted to the compliance service. |
| `l10n_sa_csr_errors` | Onboarding errors | Journal | rich text | What the service objected to. |
| `l10n_sa_compliance_csid_json` | Compliance credential data | Journal | text | The data the compliance service returned. |
| `l10n_sa_compliance_csid_certificate_id` | Compliance certificate | Journal | link to Digital Certificate | The certificate used during the compliance checks. |
| `l10n_sa_compliance_checks_passed` | Compliance checks done | Journal | boolean, default false | Whether the sample documents were accepted. |
| `l10n_sa_production_csid_json` | Production credential data | Journal | text | The data the production service returned. |
| `l10n_sa_production_csid_certificate_id` | Production certificate | Journal | link to Digital Certificate | The certificate used for real documents. |
| `l10n_sa_production_csid_validity` | Production certificate validity | date and time, related | When the production certificate expires. |
| `l10n_sa_chain_sequence_id` | Chain sequence | Journal | link to a numbering series | The series that gives each invoice its position in the chain. |
| `l10n_sa_latest_submission_hash` | Latest submission hash | Journal | text | The hash of the last submitted invoice, which the next one must quote. |

### 7.2 Invoice fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_sa_uuid` | Document unique identifier | text | The identifier of this invoice in the authority's terms. |
| `l10n_sa_invoice_signature` | Payload signature | text | The signature applied to the unsigned payload. |
| `l10n_sa_chain_index` | Chain index | integer, read-only | The invoice's position in the chain; set only for an invoice that is in the chain. |
| `l10n_sa_edi_chain_head_id` | Chain stopping document | link to Journal Entry, read-only | The document that blocks the chain when an earlier one is unresolved. |

### 7.3 Onboarding

The onboarding is driven by a wizard that asks for the journal, whether this is a renewal, and the
one-time password the authority's portal displays. It is refused with "Please provide an OTP to
complete the onboarding process" when the password is missing. On success the compliance
certificate is stored; after the sample documents pass, the production certificate replaces it.

### 7.4 Chaining

Every invoice quotes the hash of the previous submitted invoice of the same journal. A gap makes
every later invoice invalid, which is why an unresolved invoice becomes the chain stopping document
and blocks the ones behind it until it is settled.

---

## 8. Point of sale

The point of sale packages add the bilingual receipt, the visual code and the signing of receipts
through the same certificates, so that a receipt is a valid simplified tax invoice.

---

## 9. Acceptance scenarios

**Given** a company in Saudi Arabia with no accounting,
**when** the Saudi template is loaded,
**then** 169 accounts, five tax groups and two fiscal positions exist and the bilingual invoice
setting is true.

**Given** an invoice line of 1,000.00 with the 15 percent tax,
**when** the totals are computed,
**then** the tax is 150.00 and the total is 1,150.00.

**Given** a credit note with no adjustment reason,
**when** it is posted,
**then** the posting is refused, because the reason is required.

**Given** an onboarding wizard with no one-time password,
**when** it is confirmed,
**then** it is refused with "Please provide an OTP to complete the onboarding process".

**Given** a journal whose latest submission hash is the hash of invoice number 41,
**when** invoice number 42 is submitted,
**then** its payload quotes that hash and, on acceptance, the journal's latest submission hash
becomes the hash of invoice 42 and the chain index of invoice 42 is 42.
