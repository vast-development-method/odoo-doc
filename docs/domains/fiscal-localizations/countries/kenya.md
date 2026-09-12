# Kenya

The Kenyan package supplies one chart of accounts template with 127 accounts, 116 tax rows, five tax
groups and two fiscal positions, a catalogue of administration codes that justify a rate or an
exemption on a line, customer withholding certificates on invoices, and a signing flow through a
certified fiscal device that returns a device serial number, a device invoice number, a signing
moment and a visual code.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `ke` |
| Display name | Kenya |
| Parent template | none |
| Fiscal country | Kenya |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Kenyan shilling |
| Default sale tax | the 16 percent sales tax |
| Default purchase tax | the 16 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 127 |
| Tax rows shipped | 116 |
| Tax group rows shipped | 5 |
| Fiscal position rows shipped | 2 |

---

## 3. Taxes

| Family | Rate | Scope |
|---|---|---|
| General sales | 16 percent | sale |
| General purchases | 16 percent | purchase |
| Reduced rate | 8 percent | sale and purchase |
| Zero rated | 0 percent | sale and purchase |
| Exempt | none | sale and purchase |
| Withholding on sales | negative rates | sale |
| Withholding on purchases | negative rates | purchase |

**Worked example.** A sale of 20,000.00 at 16 percent produces a tax of round(20,000.00 × 0.16, 2) =
3,200.00 and a total of 23,200.00.

---

## 4. Statutory reporting

Two report structures are shipped: the periodic tax return, whose first columns are the base and the
tax of each section, and the withholding return, which lists the amounts withheld by customers and
the certificates that support them.

---

## 5. Item codes

An Kenya Item Code record is a code published by the administration that justifies a given rate or
exemption for a class of goods or services. It carries a code and a description and is searchable by
either.

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_ke_item_code_id` | Item code | Journal Item | link to Kenya Item Code | The code that justifies the rate or the exemption applied to this line. |

---

## 6. Withholding certificates

When a customer withholds tax on a payment, it issues a certificate. The invoice records it.

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_ke_wh_certificate_number` | Withholding certificate number | Journal Entry | text | The number of the certificate the customer issued. |
| `l10n_ke_wh_certificate_date` | Certificate date | Journal Entry | date | The date of that certificate. |

The two fields feed the withholding return and are the evidence for the credit the company claims.

---

## 7. The certified fiscal device

### 7.1 Company configuration

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_ke_cu_proxy_address` | Device proxy address | text, default `http://localhost:8069` | The address of the proxy that talks to the device attached to the till. The default points at the local machine, because the device is physically present. |
| `l10n_ke_oscu_is_active` | Device flow active | boolean, derived | Whether the company is set up for the device flow. |

### 7.2 Contact configuration

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_ke_exemption_number` | Exemption number | text | The exemption number the administration issued to this counterpart. |

### 7.3 Invoice fields

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `l10n_ke_cu_datetime` | Device signing moment | date and time | When the device signed the invoice. |
| `l10n_ke_cu_serial_number` | Device serial number | text | The serial number of the device that signed. |
| `l10n_ke_cu_invoice_number` | Device invoice number | text | The number the device allocated. |
| `l10n_ke_cu_qrcode` | Device visual code | text | The value the visual code printed on the invoice encodes. |
| `l10n_ke_cu_show_send_button` | Send button shown | boolean, derived from the company and the invoice | Whether the invoice can still be signed. |

### 7.4 The flow

1. The invoice is posted and the send button appears.
2. The payload is handed to the proxy, which passes it to the device.
3. The device signs and returns the serial number, the device invoice number, the signing moment and
   the visual code, all of which are stored on the invoice.
4. The invoice becomes printable with the visual code and the device data, which is what makes it a
   valid tax invoice.
5. A device that is unreachable leaves the invoice unsigned; the send button stays.

---

## 8. Acceptance scenarios

**Given** a company in Kenya with no accounting,
**when** the Kenyan template is loaded,
**then** 127 accounts, five tax groups and two fiscal positions exist.

**Given** an invoice line of 20,000.00 with the 16 percent tax,
**when** the totals are computed,
**then** the tax is 3,200.00 and the total is 23,200.00.

**Given** an invoice line carrying a zero-rated tax and no item code,
**when** the invoice is examined for the return,
**then** the line has no justification for the rate, which is what the item code exists to supply.

**Given** a posted invoice and a reachable device,
**when** the invoice is signed,
**then** the device serial number, the device invoice number, the signing moment and the visual code
are stored and the send button disappears.

**Given** a customer that withholds 2,000.00 and issues a certificate numbered `WHT/2024/0042`
dated 15 March,
**when** the certificate is recorded on the invoice,
**then** the withholding return lists the amount with that number and that date.
