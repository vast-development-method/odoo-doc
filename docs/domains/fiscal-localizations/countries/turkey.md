# Türkiye

The Turkish package supplies one chart of accounts template with 267 accounts and 64 account groups,
110 tax rows and thirteen tax groups, a default sales return account, a connection to an accredited
intermediary that keeps the directory of counterparts able to receive electronic invoices, an
electronic invoice and archive invoice flow, an electronic dispatch note with trailer plates, and an
extended set of invoice fields carrying the administration's own scenario, type, exemption and
withholding codes.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `tr` |
| Display name | Türkiye |
| Parent template | none |
| Fiscal country | Türkiye |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Turkish lira |
| Default sale tax | the 20 percent sales tax |
| Default purchase tax | the 20 percent purchase tax |
| `l10n_tr_default_sales_return_account_id` | Default sales return account, a company-dependent link to Account, derived and overridable |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 267 |
| Account group rows shipped | 64 |
| Tax rows shipped | 110 |
| Tax group rows shipped | 13 |
| Fiscal position rows shipped | none |

---

## 3. Taxes

| Family | Rates | Scope |
|---|---|---|
| Domestic sales | 20, 10, 1, 0 percent | sale |
| Domestic purchases | 20, 10, 1, 0 percent | purchase |
| Withholding on sales | the published fractions of the tax | sale |
| Special consumption tax | the published rates | sale and purchase |
| Banking and insurance transaction tax | the published rates | sale |
| Stamp duty | fixed amounts | sale and purchase |
| Exempt | none | sale and purchase |

**Worked example.** A sale of 1,000.00 at 20 percent produces a tax of 200.00 and a total of
1,200.00. With a two-tenths withholding, 40.00 of the 200.00 is withheld by the buyer and 160.00 is
paid to the seller with the base.

---

## 4. Statutory reporting

One report structure is shipped, the periodic return, whose columns are the base and the tax and
whose sections follow the boxes of the paper form.

---

## 5. The accredited intermediary

### 5.1 Company configuration

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_tr_nilvera_api_key` | Intermediary key | text | The credential issued to the company. |
| `l10n_tr_nilvera_use_test_env` | Use the test environment | boolean, required, default true | False routes every call to the live service. |
| `l10n_tr_nilvera_purchase_journal_id` | Intermediary purchase journal | link to Journal, derived and overridable | Where inbound documents become vendor bills. |

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `is_nilvera_journal` | Journal used for the intermediary | Journal | boolean | Marks the journal that receives inbound documents. |

### 5.2 Counterpart directory

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_tr_nilvera_customer_status` | Counterpart status | Contact | selection, read-only, tracked | `not_checked`, `earchive`, `einvoice`. |

A counterpart whose status is `einvoice` receives an electronic invoice through the network at one
of its registered aliases, each of which is a Turkey Electronic Invoicing Alias record. A
counterpart whose status is `earchive` receives an archive invoice, which the seller delivers by
other means. A counterpart tag is used to group the counterparts that have been looked up.

### 5.3 Invoice fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_tr_nilvera_uuid` | Document unique identifier | text, read-only | The identifier of the document at the intermediary. |
| `l10n_tr_nilvera_send_status` | Intermediary status | selection, read-only, default `not_sent` | `error`, `not_sent`, `sent`, `succeed`, `waiting`, `unknown`. |

### 5.4 Extended invoice fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_tr_gib_invoice_scenario` | Invoice scenario | selection, default `TEMELFATURA` | `TEMELFATURA` (Basic), `KAMU` (Public) and the remaining published scenarios. |
| `l10n_tr_gib_invoice_type` | Administration invoice type | selection, derived and overridable | The published document type of the invoice. |
| `l10n_tr_is_export_invoice` | Export invoice | boolean | Marks a document that leaves the customs territory. |
| `l10n_tr_shipping_type` | Shipping method | selection | `1` sea transportation, `2` railway transportation, `3` road transportation and the remaining published modes. |
| `l10n_tr_exemption_code_id` | Exemption reason | link to Turkey Tax Code | The administration code that justifies an exemption. |
| `l10n_tr_exemption_code_domain_list` | Allowed exemption codes | binary, derived | The codes the chosen taxes allow, which narrows the selection. |
| `l10n_tr_tax_withholding_code_id` | Withholding reason | link to Turkey Tax Code | The administration code that justifies a withholding. |
| `l10n_tr_ctsp_number` | Customs tariff number | text, on the invoice line and derived from the product | The tariff classification of the goods. |
| `l10n_tr_nilvera_customer_status` | Counterpart status | selection, related to the counterpart | Shown on the invoice so that the sender knows which document kind will be produced. |

### 5.5 Tax codes and tax offices

A Turkey Tax Code record is an administration code attached to a tax, an exemption or a withholding.
A Turkey Tax Office record is the office a company or a counterpart files with. Both are shared
reference tables.

### 5.6 The flow

The transitions and the guards are in
[state-machines.md](../state-machines.md#20-turkey-exchange-and-dispatch). Inbound documents whose
status at the intermediary is `succeed` are fetched in creation order, so that an interrupted run
resumes where it stopped.

---

## 6. Electronic dispatch notes

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_tr_nilvera_dispatch_state` | Dispatch status | Transfer | selection, tracked | `to_send`, `sent`. |
| Trailer plates | Trailer plates | Transfer | collection of Turkey Trailer Plate | The registration plates of the trailers carrying the goods. |

The operation type of the transfer decides whether a dispatch note is required.

---

## 7. Identification number formats

A further package adds the two national identification number formats, one for individuals and one
for legal entities, and validates them on the contact form and on the online checkout with the same
checker and the same message.

---

## 8. Acceptance scenarios

**Given** a company in Türkiye with no accounting,
**when** the Turkish template is loaded,
**then** 267 accounts, 64 account groups and thirteen tax groups exist.

**Given** an invoice line of 1,000.00 with the 20 percent tax,
**when** the totals are computed,
**then** the tax is 200.00 and the total is 1,200.00.

**Given** a counterpart whose status is `not_checked`,
**when** an invoice to it is sent,
**then** the send is refused until the directory lookup has run.

**Given** a counterpart whose status is `earchive`,
**when** an invoice is issued,
**then** an archive invoice is produced and must be delivered by other means.

**Given** an invoice handed to the intermediary,
**when** the refresh job reports a status the system does not map,
**then** the invoice's status becomes `unknown` and the raw answer is kept.

**Given** a transfer whose dispatch status is `to_send` and two trailer plates,
**when** the dispatch note is sent,
**then** the status becomes `sent` and both plates travel in the payload.
