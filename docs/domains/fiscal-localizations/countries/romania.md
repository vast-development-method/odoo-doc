# Romania

The Romanian package supplies one chart of accounts template with 579 accounts and 139 account
groups, 340 tax rows, eighteen tax groups and nine fiscal positions, a public procurement
classification code on products, a national invoicing portal with an authorisation exchange and a
document history, and a goods movement declaration for transfers and for batches of transfers.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `ro` |
| Display name | Romania |
| Parent template | none |
| Fiscal country | Romania |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Romanian leu |
| Default sale tax | the 21 percent sales tax |
| Default purchase tax | the 21 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 579 |
| Account group rows shipped | 139 |
| Tax rows shipped | 340 |
| Tax group rows shipped | 18 |
| Fiscal position rows shipped | 9 |

Eighteen tax groups reflect the number of separate boxes the return keeps apart: each rate on sales,
each rate on purchases, the reverse-charge families, the intra-union families and the exempt
families each settle into their own liability or receivable account.

---

## 3. Taxes

| Family | Rates | Scope |
|---|---|---|
| Domestic sales | 21, 11, 0 percent | sale |
| Domestic purchases | 21, 11, 0 percent | purchase |
| Reverse charge, domestic | 21, 11 percent | sale and purchase |
| Intra-union supply of goods | 0 percent | sale |
| Intra-union acquisition of goods | 21, 11 percent | purchase |
| Intra-union services | 0 and 21 percent | sale and purchase |
| Export | 0 percent | sale |
| Import | 21, 11 percent | purchase |
| Exempt with credit and exempt without credit | none | sale and purchase |

**Worked example.** A sale of 1,000.00 at 21 percent produces a tax of 210.00 and a total of
1,210.00. A reverse-charge acquisition of 1,000.00 at 21 percent charges 210.00 and deducts 210.00,
so nothing is due while both boxes report a base of 1,000.00.

---

## 4. Statutory reporting

One report structure is shipped, the periodic return. Its first line is the balance, and its
sections separate the tax base of trade within the union from that outside it, and the deductible
tax from the collected tax.

---

## 5. Public procurement codes

A Romania Common Procurement Code record carries a code and a description. It is attached to a
product and travels into the payload, because a public body's system rejects a line that does not
name one.

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `cpv_code_id` | Public procurement code | Product Template | link to Romania Common Procurement Code | The classification of the goods or service. |

---

## 6. The national invoicing portal

### 6.1 Company configuration

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_ro_edi_client_id` | Portal client identifier | text | The credential issued to the taxpayer. |
| `l10n_ro_edi_client_secret` | Portal client secret | text | The matching secret. |
| `l10n_ro_edi_access_token` | Access token | text | Obtained by the authorisation exchange. |
| `l10n_ro_edi_refresh_token` | Refresh token | text | Used to renew the access token. |
| `l10n_ro_edi_access_expiry_date` | Access token expiry | date | When the access token stops working. |
| `l10n_ro_edi_refresh_expiry_date` | Refresh token expiry | date | When the refresh token stops working. |
| `l10n_ro_edi_callback_url` | Callback address | text, derived from the country code | The address the portal returns the authorisation answer to. |
| `l10n_ro_edi_test_env` | Use the test environment | boolean, default true | False routes every call to the live portal. |
| `l10n_ro_edi_anaf_imported_inv_journal_id` | Journal for imported bills | link to Journal | Where documents fetched from the portal become vendor bills. |

### 6.2 The authorisation exchange

Two addresses drive it, both listed in [interfaces.md](../interfaces.md).

1. The authorisation address redirects the user to the portal's own authorisation page, carrying the
   response type, the client identifier, the callback address and the token content type. It is
   refused with "Client ID and Client Secret field must be filled." when either credential is
   missing.
2. The callback address receives the answer and exchanges it for the two tokens and their two expiry
   dates. It is refused with "Access key not found. Please try again. Response: `<parameters>`"
   followed by "Received access key: `<key>`" when no key is present, which happens when the user
   presented no certificate.

### 6.3 Invoice fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_ro_edi_document_ids` | Portal documents | collection of Romania Interchange Document | The history of every attempt. |
| `l10n_ro_edi_state` | Portal status | selection, derived from the newest document and stored | `invoice_not_indexed`, `invoice_sent`, `invoice_refused`, `invoice_validated`. |
| `l10n_ro_edi_index` | Portal index | text, read-only | The index the portal allocates, by which an answer is looked up. |

### 6.4 The flow

The transitions, the guards and the messages are in
[state-machines.md](../state-machines.md#15-romania-portal). In summary: an invoice that is not
ready is refused with "The invoice is not ready to be sent: `<comma-separated reasons>`"; a portal
that has not finished answers "SPV has not finished processing the invoice, try again later."; a
validated invoice is announced with "This invoice has been accepted by the SPV." and its superseded
document records are removed; and an invoice that is sent or validated cannot be reset to draft.

### 6.5 Inbound documents

A job synchronises with the portal's message list and imports what it finds, posting "Synchronized
with SPV from message `<identifier>`" and, when the message carried no printable document, "No PDF
found: PDF imported from SPV.".

---

## 7. Goods movement declarations

A transfer that crosses the border, or that carries goods above the declarable threshold, is
declared to the portal.

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_ro_edi_stock_state` | Movement status | Transfer and Batch Transfer | selection, read-only | `stock_sent`, `stock_sending_failed`, `stock_validated`. |
| `l10n_ro_edi_stock_partner_id` | Declared counterpart | Transfer | link to Contact | The party the goods are declared as moving to or from, which is not always the transfer's own counterpart. |

The shipping method carries the transport data the declaration needs. A batch transfer sends one
declaration for the whole batch, and its status is the status of that declaration.

---

## 8. Acceptance scenarios

**Given** a company in Romania with no accounting,
**when** the Romanian template is loaded,
**then** 579 accounts, 139 account groups, eighteen tax groups and nine fiscal positions exist.

**Given** an invoice line of 1,000.00 with the 21 percent tax,
**when** the totals are computed,
**then** the tax is 210.00 and the total is 1,210.00.

**Given** a company with a client identifier and no client secret,
**when** the authorisation is started,
**then** it is refused with "Client ID and Client Secret field must be filled.".

**Given** an invoice accepted by the portal with no index returned,
**when** the state is written,
**then** it becomes `invoice_not_indexed`, and when the indexing job finds the index it becomes
`invoice_sent`.

**Given** an invoice whose status is `invoice_sent`,
**when** the status job reports validation,
**then** the status becomes `invoice_validated`, the signature, certificate and download keys are
stored, "This invoice has been accepted by the SPV." is posted, and the superseded documents are
removed.

**Given** an invoice whose status is `invoice_validated`,
**when** a reset to draft is attempted,
**then** it is refused.

**Given** a batch of four transfers declared as one movement,
**when** the portal validates the declaration,
**then** the batch and each of its four transfers show the status `stock_validated`.
