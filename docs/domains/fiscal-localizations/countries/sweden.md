# Sweden

The Swedish package supplies three chart of accounts templates built on the national standard
chart — the full chart, the simplified annual-report chart and the full annual-report chart — with
537 account groups, 176 tax rows, four tax groups and twenty fiscal positions, and a reference
number on invoices and on supplier payments that the bank returns with the payment.

---

## 1. Templates

| Template code | Display name | Parent | Account rows |
|---|---|---|---|
| `se` | Sweden, standard chart | none | 332 |
| `se_K2` | Sweden, simplified annual report | `se` | 856 |
| `se_K3` | Sweden, full annual report | `se` | 26 |

The two annual-report templates change the chart to match the annual report the company files: the
simplified variant replaces most of the standard chart with the accounts the simplified report
expects, and the full variant adds twenty-six accounts to the standard chart.

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Swedish krona |
| Default sale tax | the 25 percent sales tax |
| Default purchase tax | the 25 percent purchase tax |
| Invoice reference style | the national reference number form |

---

## 2. Chart of accounts

| Measure | `se` | `se_K2` | `se_K3` |
|---|---|---|---|
| Account rows shipped | 332 | 856 | 26 |
| Account group rows shipped | 537 | none | none |
| Tax rows shipped | 176 | none | none |
| Tax group rows shipped | 4 | 4 | 4 |
| Fiscal position rows shipped | 20 | none | none |

The taxes, the tax groups and the fiscal positions live on the standard chart, so the two
annual-report templates inherit them through the parent chain. Only the accounts and the four tax
groups differ.

---

## 3. Taxes

| Family | Rates | Scope |
|---|---|---|
| Domestic sales | 25, 12, 6, 0 percent | sale |
| Domestic purchases | 25, 12, 6, 0 percent | purchase |
| Reverse charge, construction | 25 percent | sale and purchase |
| Intra-union supply of goods | 0 percent | sale |
| Intra-union acquisition of goods | 25, 12, 6 percent | purchase |
| Intra-union services | 0 and 25 percent | sale and purchase |
| Export | 0 percent | sale |
| Import | 25, 12, 6 percent | purchase |
| Exempt | none | sale and purchase |

**Worked example.** A sale of 1,000.00 at 25 percent produces a tax of 250.00 and a total of
1,250.00. At 12 percent the tax is 120.00 and at 6 percent 60.00.

---

## 4. Fiscal positions

Twenty positions are shipped, which is more than any other European chart of this domain. They
distinguish, for both directions, the domestic supply, the intra-union supply of goods, the
intra-union supply of services, the acquisition of goods, the acquisition of services, the export,
the import, the construction reverse charge, the trade in second-hand goods and the supply to a
diplomatic buyer. Each maps the domestic taxes to the taxes whose report tags feed the right box of
the return.

---

## 5. Statutory reporting

One report structure is shipped, the periodic tax return. Its first line is the balance, and its
first section is the taxable sales or withdrawals excluding tax, followed by the sections of the
paper form in order.

---

## 6. The reference number

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_se_invoice_ocr_length` | Reference number length | Company | integer, default 6 | The total length of the reference number the company issues, including the check digit. |
| `l10n_se_check_vendor_ocr` | Check supplier references | Contact | boolean | Whether this supplier issues reference numbers that must be checked before a payment is made. |
| `l10n_se_default_vendor_payment_ref` | Default supplier payment reference | Contact | text | The reference this supplier always expects, used when the bill carries none. |

The reference printed on a customer invoice is derived from the invoice number and closed with a
check digit, and its total length is the company's configured length. The bank returns the reference
with the payment, which lets the reconciliation match the payment to the invoice.

**Worked example.** A company whose reference length is 6 issues invoice number 123. The reference
is the invoice number padded to five digits, `00123`, followed by the check digit computed over
those five digits, so the printed reference is six characters long.

---

## 7. Acceptance scenarios

**Given** a company in Sweden with no accounting,
**when** the standard template is loaded,
**then** 332 accounts, 537 account groups, four tax groups and twenty fiscal positions exist.

**Given** the same company,
**when** the simplified annual-report template is loaded instead,
**then** 856 accounts exist and the taxes, the tax groups and the fiscal positions are those of the
standard chart, because the template inherits them through its parent.

**Given** an invoice line of 1,000.00 with the 25 percent tax,
**when** the totals are computed,
**then** the tax is 250.00 and the total is 1,250.00.

**Given** a company whose reference number length is 6 and invoice number 123,
**when** the invoice is printed,
**then** the reference is six characters long, ends with its check digit and is the value the bank
will return with the payment.

**Given** a supplier marked as issuing reference numbers, and a vendor bill with no payment
reference,
**when** a payment is registered,
**then** the supplier's default reference is used.
