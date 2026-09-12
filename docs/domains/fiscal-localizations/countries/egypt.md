# Egypt

The Egyptian package supplies one chart of accounts template with 205 accounts, eight fixed asset
models, 120 tax rows, fourteen tax groups and two fiscal positions, plus a tax administration
invoice integration that signs each document with a per-user hardware device, classifies every
product and every unit of measure with the administration's own codes, and returns a registration
identifier and a visual code that the invoice must display.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `eg` |
| Display name | Egypt |
| Parent template | none |
| Fiscal country | Egypt |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Egyptian pound |
| Default sale tax | the 14 percent sales tax |
| Default purchase tax | the 14 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 205 |
| Fixed asset model rows shipped | 8 |
| Tax rows shipped | 120 |
| Tax group rows shipped | 14 |
| Fiscal position rows shipped | 2 |

Fourteen tax groups is unusually many and reflects the number of separate levies the return
distinguishes: the general rate, the table taxes, the schedule taxes, the withholding levies and the
stamp duties, each of which settles into its own liability account.

---

## 3. Taxes

| Family | Rate | Scope |
|---|---|---|
| General sales | 14 percent | sale |
| General purchases | 14 percent | purchase |
| Zero rated | 0 percent | sale and purchase |
| Exempt | none | sale and purchase |
| Table tax on goods | the published rate of each schedule item | sale and purchase |
| Withholding on payments | negative rates | sale and purchase |
| Stamp duty | fixed amounts | sale and purchase |

**Worked example.** A sale of 5,000.00 at 14 percent produces a tax of round(5,000.00 × 0.14, 2) =
700.00 and a total of 5,700.00.

---

## 4. Statutory reporting

One report structure is shipped, the value-added tax return. Its first line is the balance, and its
sections separate the base and the tax of sales and of all other outputs.

---

## 5. Fields added by the base package

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_eg_eta_code` | Administration code | Tax | selection | The administration's code for the treatment the tax represents. The stored values include `t1_v001` (export), `t1_v002` (export to free areas and other countries) and the remaining published pairs of table and code, one per exemption and one per schedule item. |

---

## 6. The tax administration invoice integration

### 6.1 Company configuration

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_eg_client_identifier` | Administration client identifier | text | The credential issued to the taxpayer. |
| `l10n_eg_client_secret` | Administration secret | text | The matching secret. |
| `l10n_eg_production_env` | Production environment | boolean | False routes every call to the test service. |
| `l10n_eg_invoicing_threshold` | Invoicing threshold | decimal, default 0.0 | The amount above which the customer's tax identification number becomes mandatory on the invoice. |

### 6.2 Branch and activity

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_eg_branch_id` | Branch | Journal | link to Contact | The address of the subdivision of the company that issues from this journal. A company with one place of business points it at its own contact. |
| `l10n_eg_activity_type_id` | Activity code | Journal | link to Egypt Activity Type | The economic activity of the branch, taken from the administration's published list. |
| `l10n_eg_branch_identifier` | Branch identifier | Journal | text | The number the administration shows on the taxpayer profile. |

### 6.3 Signing device

An Egypt Signing Device record holds, per user and per company, the certificate and the personal
identification number of the hardware token used to sign a document. Signing is refused when no
device is registered for the user issuing the invoice.

### 6.4 Fields on the invoice

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_eg_eta_json_doc_file` | Structured data document | binary | The payload as transmitted. |
| `l10n_eg_uuid` | Document unique identifier | text | Derived from the answer; the administration's identifier for the document. |
| `l10n_eg_submission_number` | Submission identifier | text | Derived from the answer; the identifier of the batch the document travelled in. |
| `l10n_eg_long_id` | Long identifier | text | Derived from the answer; the value the visual code encodes. |
| `l10n_eg_qr_code` | Visual code | text | Derived from the invoice date, the long identifier and the company data; printed on the invoice. |
| `l10n_eg_signing_time` | Signing time | date and time | When the device signed the payload. |
| `l10n_eg_is_signed` | Signed | boolean | Whether a signature has been applied. |

### 6.5 Product and unit codes

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_eg_eta_code` | Item code | Product Variant | text | The product classification the administration expects. |
| `l10n_eg_eta_code` | Item code | Product Template | text | Derived from the variants; the same value when every variant agrees. |
| Unit code | Unit code | Unit of Measure | link to Egypt Unit of Measure Code | The administration's code for the unit. |

### 6.6 Exchange rate on the payload

The payload carries the exchange rate of the invoice date when the invoice is not in the company
currency. The rate is taken from the currency rate table, which the package extends so that a rate
can be recorded per company and per day for this purpose.

### 6.7 The flow

The invoice follows the generic country exchange machine of
[state-machines.md](../state-machines.md#5-the-generic-country-exchange-machine): the payload is
built and signed, transmitted, and the answer is polled. On acceptance the unique identifier, the
submission identifier, the long identifier and the visual code are stored; on refusal the errors are
posted in the discussion thread.

---

## 7. Acceptance scenarios

**Given** a company in Egypt with no accounting,
**when** the Egyptian template is loaded,
**then** 205 accounts, eight fixed asset models, fourteen tax groups and two fiscal positions exist.

**Given** an invoice of 5,000.00 with the 14 percent sales tax,
**when** the totals are computed,
**then** the tax is 700.00 and the total is 5,700.00.

**Given** a company whose invoicing threshold is 1,000.00 and an invoice of 1,500.00 to a customer
with no tax identification number,
**when** the invoice is transmitted,
**then** it is refused, because above the threshold the customer's number is mandatory.

**Given** a user with no signing device registered,
**when** that user transmits an invoice,
**then** the transmission is refused and the invoice stays untransmitted.

**Given** a transmitted invoice the administration accepts,
**when** the answer arrives,
**then** the document unique identifier, the submission identifier and the long identifier are
stored, the visual code is computed from the invoice date and the long identifier, and the invoice
becomes printable in its final form.
