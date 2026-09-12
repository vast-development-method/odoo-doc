# Denmark

The Danish package supplies one chart of accounts template with 488 accounts and 41 account groups,
351 tax rows, one tax group, eight fiscal positions, a national payment reference on invoices, a
registration on the national document exchange network with its own state machine, business-level
answers to inbound documents, and the national public-information payload format in two versions.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `dk` |
| Display name | Denmark |
| Parent template | none |
| Fiscal country | Denmark |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Danish krone |
| Default sale tax | the 25 percent sales tax |
| Default purchase tax | the 25 percent purchase tax |
| Tax computation rounding method | Round per Tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 488 |
| Account group rows shipped | 41 |
| Tax rows shipped | 351 |
| Tax group rows shipped | 1 |
| Fiscal position rows shipped | 8 |

A single tax group means that every tax settles into one liability account, which is what the
periodic return expects.

---

## 3. Taxes

| Family | Rate | Scope |
|---|---|---|
| Domestic sales | 25 percent | sale |
| Domestic purchases | 25 percent | purchase |
| Zero-rated sales | 0 percent | sale |
| Intra-union supply of goods | 0 percent | sale |
| Intra-union acquisition of goods | 25 percent | purchase |
| Intra-union services supplied | 0 percent | sale |
| Intra-union services received | 25 percent | purchase |
| Export outside the union | 0 percent | sale |
| Import from outside the union | 25 percent | purchase |
| Reverse charge, domestic | 25 percent | sale and purchase |
| Exempt | none | sale and purchase |

**Worked example.** An intra-union acquisition of 4,000.00 at 25 percent charges 1,000.00 and
deducts 1,000.00, so the amount due is 0.00 while both the output box and the input box report a
base of 4,000.00.

---

## 4. Statutory reporting

One report structure is shipped, the periodic value-added tax return, whose first line is the
balance of the tax due. It allows a foreign registration and is available when the company's fiscal
country is Denmark.

---

## 5. The national payment reference

The payment-reference package adds a creditor number to the company and a reference form to
invoices, so that a customer's bank transfer can be matched automatically.

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_dk_fik_creditor_number` | National payment reference creditor number | Company | text | The creditor number allocated to the company. It is derived from the company's first bank account number and from the invoice reference style, and it may be overridden. |

When the invoice reference style is the national payment reference form, the reference printed on
the invoice is built from the creditor number and the invoice number, and the payment matching
recognises it on the bank statement line.

---

## 6. The national exchange network

### 6.1 Registration

The company registers as a sender and receiver on the network through a proxy user. The registration
is driven by a wizard and its outcome is the company's registration state, specified in
[state-machines.md](../state-machines.md#143-company-registration-state).

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_dk_nemhandel_proxy_state` | Network registration state | Company | selection | `not_registered`, `in_verification`, `receiver`, `rejected`. |
| `nemhandel_contact_email` | Network contact electronic mail address | Company | text | Derived from the company's own address; the primary contact for network correspondence. |
| `nemhandel_phone_number` | Network telephone number | Company | text | Verified by a code sent as a text message before the registration is submitted. |
| `nemhandel_verification_code` | Network verification code | Company | text | The six-digit code received by text message. |
| `nemhandel_identifier_type` and `nemhandel_identifier_value` | Network endpoint type and value | Company and Contact | selection and text | The addressing scheme and the address on the network. |
| `nemhandel_purchase_journal_id` | Network purchase journal | Company | link to Journal | Where inbound documents become vendor bills. |
| `nemhandel_edi_user` | Network proxy user | Company | link to the exchange proxy user | The credential holder. |
| `nemhandel_edi_mode` | Network operating mode | Company | selection | Related to the proxy user's mode; a demonstration mode cannot register. |
| `is_nemhandel_journal` | Journal used for the network | Journal | boolean | Marks the journal that receives inbound documents. |

### 6.2 Sending and receiving

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `nemhandel_message_uuid` | Network message identifier | Journal Entry | text | The identifier the network returns for a handed-over payload. |
| `nemhandel_move_state` | Network status | Journal Entry | selection | `ready`, `to_send`, `processing`, `done`, `error`, and, with the answer package, `BusinessAccept` and `BusinessReject`. |
| `nemhandel_response_ids` | Network answers | Journal Entry | collection of Denmark Business Response | The business-level answers sent about this document. |
| `nemhandel_can_send_response` | Answer can be sent | Journal Entry | boolean | True when the document is inbound, delivered and not yet answered. |
| `nemhandel_verification_state` | Endpoint verification | Contact | selection | `not_verified`, `not_valid`, `valid`. |

The full transition tables, guards and messages are in
[state-machines.md](../state-machines.md#14-denmark-exchange-network).

### 6.3 Webhooks

Three public addresses let the network wake the system rather than be polled: one announcing new
inbound documents, one announcing a change in the state of a sent message and one announcing a
change in the company's own registration state. Each carries a token in the query and answers with
an empty body. They are listed in [interfaces.md](../interfaces.md).

---

## 7. The public-information payload

Two payload builders are shipped: version 2.01, used for the documents the older public bodies still
require, and version 2.1, used on the network. Both are recognised on import by the customisation
identifier the payload carries; a payload whose customisation identifier is exactly `OIOUBL-2.1` is
read by the version 2.1 builder.

A payload is produced only when the counterpart carries a tax identification number and its endpoint
verification state for that format is `valid`.

---

## 8. Business-level answers

An inbound document may be answered with an approval or a rejection. The answer is a Denmark
Business Response record whose kind is `BusinessAccept` (Approval) or `BusinessReject` (Rejection)
and whose state follows the machine in
[state-machines.md](../state-machines.md#142-denmark-business-response). A rejection carries an
additional note entered in the rejection wizard.

---

## 9. Acceptance scenarios

**Given** a company in Denmark with no accounting,
**when** the Danish template is loaded,
**then** 488 accounts, 41 account groups and one tax group exist, and eight fiscal positions exist.

**Given** a company whose registration state is `not_registered` and whose operating mode is the
demonstration mode,
**when** the registration wizard is run,
**then** it is refused with "Cannot register a user with a `<mode>` application".

**Given** a company whose registration state is `receiver`, a customer whose endpoint verification
state is `valid` and a posted customer invoice,
**when** the invoice is saved,
**then** its network status becomes `ready`.

**Given** that invoice queued and handed to the network,
**when** the status job reports delivery,
**then** the status becomes `done`,
**and** when the counterpart returns an approval, a response record is created and the status
becomes `BusinessAccept`.

**Given** an invoice whose network status is `processing`,
**when** the electronic document is cancelled,
**then** it is refused with "Cannot cancel an entry that has already been sent to Nemhandel".

**Given** an intra-union acquisition of 4,000.00 at 25 percent,
**when** the totals are computed,
**then** 1,000.00 is charged and 1,000.00 is deducted and the amount due is 0.00.
