# Finland

The Finnish package supplies one chart of accounts template with 971 accounts, 239 tax rows, six tax
groups and six fiscal positions, the national reference number on invoices, and the payload profile
used for electronic invoicing between Finnish businesses.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `fi` |
| Display name | Finland |
| Parent template | none |
| Fiscal country | Finland |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | euro |
| Default sale tax | the 25.5 percent sales tax |
| Default purchase tax | the 25.5 percent purchase tax |
| Invoice reference style | the national reference number form |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 971 |
| Account group rows shipped | none |
| Tax rows shipped | 239 |
| Tax group rows shipped | 6 |
| Fiscal position rows shipped | 6 |

971 account rows makes this one of the largest charts of the domain; it follows the national
business chart in full, so that a statutory financial statement can be produced without adding a
single account.

---

## 3. Taxes

| Family | Rates | Scope |
|---|---|---|
| Domestic sales | 25.5, 14, 10, 0 percent | sale |
| Domestic purchases | 25.5, 14, 10, 0 percent | purchase |
| Construction reverse charge | 25.5 percent | sale and purchase |
| Intra-union supply of goods | 0 percent | sale |
| Intra-union acquisition of goods | 25.5, 14, 10 percent | purchase |
| Intra-union services supplied | 0 percent | sale |
| Intra-union services received | 25.5 percent | purchase |
| Export outside the union | 0 percent | sale |
| Import from outside the union | 25.5, 14, 10 percent | purchase |

**Worked example.** A domestic sale of 800.00 at 25.5 percent produces a tax of round(800.00 ×
0.255, 2) = round(204.00, 2) = 204.00 and a total of 1,004.00.

**Worked example of the reduced rate.** A sale of 800.00 at 14 percent produces round(800.00 × 0.14,
2) = 112.00.

---

## 4. Tax groups

Six groups, one per rate that the return distinguishes: 25.5 percent, 14 percent, 10 percent, zero
rated, exempt and the reverse-charge group. Each names the liability account the periodic return
settles.

---

## 5. Fiscal positions

| Name | Detected automatically | Requires a tax number | Country or group |
|---|---|---|---|
| Domestic | yes | no | Finland |
| European Union, business to business | yes | yes | European Union group |
| European Union, business to consumer | yes | no | European Union group |
| Outside the union | yes | no | none |
| Construction reverse charge | no | no | none |
| One-stop shop | no | no | none |

---

## 6. Statutory reporting

One report structure is shipped, the periodic value-added tax return. Its first line is the balance
of the tax due, and its sections separate domestic sales by rate, intra-union transactions, imports
and the deductible tax.

---

## 7. The national reference number

The invoice reference style is the national reference number form, in which the reference printed on
the invoice is derived from the invoice number and closed with a check digit. The bank returns the
reference with the payment, which lets the reconciliation match the payment to the invoice without
a human decision. The check-digit arithmetic is part of the reference style and is specified in
[calculations.md](../calculations.md).

---

## 8. Sales orders

The sales package carries the reference and the payment terms from the sales order onto the invoice,
so that the reference a customer was told at ordering time is the one printed on the invoice.

---

## 9. Acceptance scenarios

**Given** a company in Finland with no accounting,
**when** the Finnish template is loaded,
**then** 971 accounts, six tax groups and six fiscal positions exist and the default sales tax is
the 25.5 percent tax.

**Given** an invoice line of 800.00 with the 25.5 percent tax,
**when** the totals are computed,
**then** the tax is 204.00 and the total is 1,004.00.

**Given** an invoice line of 800.00 with the 14 percent tax,
**when** the totals are computed,
**then** the tax is 112.00 and the total is 912.00.

**Given** a customer in another member state carrying a tax identification number,
**when** the invoice is created,
**then** the "European Union, business to business" position is detected and the domestic tax is
replaced by the zero-rated intra-union supply tax.
