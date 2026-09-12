# Poland

The Polish package supplies one chart of accounts template with 238 accounts and 136 account groups,
128 tax rows, four tax groups and four fiscal positions, a catalogue of tax offices, three voucher
markers on invoices that the periodic file requires, a national invoicing system with its own status
map, and a verification of every supplier bank account against the national register before a
payment is made.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `pl` |
| Display name | Poland |
| Parent template | none |
| Fiscal country | Poland |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Polish złoty |
| Default sale tax | the 23 percent sales tax |
| Default purchase tax | the 23 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 238 |
| Account group rows shipped | 136 |
| Tax rows shipped | 128 |
| Tax group rows shipped | 4 |
| Fiscal position rows shipped | 4 |

---

## 3. Taxes

| Family | Rates | Scope |
|---|---|---|
| Domestic sales | 23, 8, 5, 0 percent | sale |
| Domestic purchases | 23, 8, 5, 0 percent | purchase |
| Reverse charge, domestic | 23 percent | sale and purchase |
| Intra-union supply of goods | 0 percent | sale |
| Intra-union acquisition of goods | 23, 8, 5 percent | purchase |
| Intra-union services | 0 and 23 percent | sale and purchase |
| Export | 0 percent | sale |
| Import | 23, 8, 5 percent | purchase |
| Exempt | none | sale and purchase |

**Worked example.** A sale of 1,000.00 at 23 percent produces a tax of 230.00 and a total of
1,230.00. At the 8 percent rate the tax is 80.00 and the total 1,080.00.

---

## 4. Tax offices

A Poland Tax Office record carries a code and a name and is the office the company files with. It is
shared by every company and is searchable by name and by code, ordered by code.

---

## 5. Voucher markers

The periodic file distinguishes three transactions involving vouchers. Each is a boolean on the
invoice, and each becomes a marker in the file.

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_pl_vat_b_spv` | Single-purpose voucher transfer | boolean | The transfer of a single-purpose voucher by a taxable person acting on its own behalf. |
| `l10n_pl_vat_b_spv_dostawa` | Single-purpose voucher supply | boolean | The supply of goods or services covered by a single-purpose voucher to a taxpayer. |
| `l10n_pl_vat_b_mpv_prowizja` | Multi-purpose voucher commission | boolean | The supply of agency and other services relating to the transfer of a voucher. |

---

## 6. The national invoicing system

### 6.1 Company configuration

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_pl_edi_register` | Registered with the national system | boolean | Whether the company files through the system. |

### 6.2 Invoice fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_pl_edi_status` | System status | selection, read-only | `sent`, `accepted`, `rejected`, `fetch_ready`, `fetched`, `fetch_failed`. |
| `l10n_pl_edi_ref` | System reference number | text, read-only | The reference the system returns for the submitted payload. |
| `l10n_pl_edi_number` | System number | text, read-only | The number the system allocates to an accepted invoice. |
| `l10n_pl_edi_session_id` | System session number | text, read-only | The session the payload was sent in. |
| `l10n_pl_edi_header` | Status header | rich text, read-only | A description of the current state with the next action. |
| `l10n_pl_edi_attachment_file` and `l10n_pl_edi_attachment_id` | Transmitted payload | binary and link to Attachment | The payload as sent. |
| `l10n_pl_edi_upo_file` and `l10n_pl_edi_upo_id` | Official acknowledgement | binary and link to Attachment | The acknowledgement document, downloadable only for an accepted invoice. |

### 6.3 The status map

The system answers with a numeric code. The full mapping onto the six states, together with the
message posted for each code, is in
[state-machines.md](../state-machines.md#161-invoice-status-in-the-national-system). In summary:
the codes 100 and 150 keep the invoice sent, the code 200 accepts it, the codes 405, 410, 415, 430,
435, 440, 450 and 500 reject it, and any other code leaves the state unchanged and writes an unknown
status message into the header.

### 6.4 Reset and retransmission

A rejected invoice may be reset to draft, which clears the status, the number, the reference, the
session identifier and the header so that the invoice can be sent again. An invoice that is sent or
accepted cannot be reset.

### 6.5 Inbound documents

Documents addressed to the company are announced and pulled: the placeholder is `fetch_ready`, the
successful pull is `fetched` and a failure is `fetch_failed`, retried by the job.

### 6.6 Public sector variant

A further package adds the public sector variant of the same system, whose payload names the public
body and its unit rather than an ordinary counterpart.

---

## 7. Supplier bank account verification

Before a payment to a supplier is registered, the supplier's bank account is checked against the
national register of taxpayers, because paying an account that is not registered against the
supplier's tax identification number costs the payer the right to deduct.

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_pl_verification_id` | Verification | Payment | link to Poland Bank Account Verification | The check that was made. |
| `l10n_pl_verification_status` | Verification status | Payment | selection, related | `valid`, `invalid`, `incomplete_partner`, `not_found_partner`, `error`. |
| `l10n_pl_verification_timestamp` | Verification moment | Payment | date and time, related | When the register answered. |
| `l10n_pl_verification_request_id` | Correlation identifier | Payment | text, related | The identifier the register returns, which is the evidence of the check. |
| `l10n_pl_bank_verification_ids` | Verifications | Payment Registration Wizard | collection | Every check made for the payments being registered. |
| `l10n_pl_bank_verification_invalid_bank_account_ids` | Unverified accounts | Payment Registration Wizard | collection of Bank Account | The accounts the register did not confirm. |
| `l10n_pl_not_found_partner_ids` | Counterparts not found | Payment Registration Wizard | collection of Contact | The counterparts the register does not know. |

The verification record stores copies of the account number and of the tax identification number, so
that a later change to the contact cannot alter the evidence. A second check on the same day for the
same pair reuses the stored result.

---

## 8. Acceptance scenarios

**Given** a company in Poland with no accounting,
**when** the Polish template is loaded,
**then** 238 accounts, 136 account groups, four tax groups and four fiscal positions exist.

**Given** an invoice line of 1,000.00 with the 23 percent tax,
**when** the totals are computed,
**then** the tax is 230.00 and the total is 1,230.00.

**Given** an invoice whose system status is `sent`,
**when** the status job receives the code 200,
**then** the status becomes `accepted`, the system number is stored and "KSeF Status: Success (Code:
200). Invoice accepted." is posted.

**Given** an invoice whose system status is `rejected`,
**when** it is reset to draft,
**then** the status, the number, the reference, the session identifier and the header are cleared.

**Given** an invoice whose status is `accepted` and whose reference number is empty,
**when** the acknowledgement is downloaded,
**then** it is refused with "This invoice does not have a KSeF Invoice Reference Number. It may not
have been sent yet.".

**Given** a supplier with a tax identification number and a bank account that the register does not
link to it,
**when** a payment is registered,
**then** a verification record with the status `invalid` is written and the wizard lists the account
among the unverified ones.

**Given** a supplier with no tax identification number,
**when** a payment is registered,
**then** a verification record with the status `incomplete_partner` is written and the register is
not called.
