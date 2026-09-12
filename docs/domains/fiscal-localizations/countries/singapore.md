# Singapore

The Singaporean package supplies one chart of accounts template with 135 accounts, 149 tax rows,
five tax groups and two fiscal positions, the unique entity number on companies and contacts, the
export permit number and its date on invoices, and the international invoice payload profile the
country has adopted.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `sg` |
| Display name | Singapore |
| Parent template | none |
| Fiscal country | Singapore |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Singapore dollar |
| Default sale tax | the 9 percent sales tax |
| Default purchase tax | the 9 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 135 |
| Tax rows shipped | 149 |
| Tax group rows shipped | 5 |
| Fiscal position rows shipped | 2 |

---

## 3. Taxes

| Family | Rate | Scope |
|---|---|---|
| Standard-rated supplies | 9 percent | sale |
| Standard-rated purchases | 9 percent | purchase |
| Zero-rated supplies | 0 percent | sale |
| Exempt supplies | none | sale |
| Out-of-scope supplies | none | sale |
| Imports under the reverse charge | 9 percent | purchase |
| Customer accounting supplies | 9 percent | sale and purchase |

**Worked example.** A standard-rated sale of 1,000.00 at 9 percent produces a tax of 90.00 and a
total of 1,090.00.

---

## 4. Statutory reporting

One report structure is shipped, the periodic return. Its first line is the balance, and its
sections are the supplies, the purchases and the tax due, following the boxes of the paper form.

---

## 5. The unique entity number

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_sg_unique_entity_number` | Unique entity number | Contact | text | The registration number every registered entity in the country carries. |
| `l10n_sg_unique_entity_number` | Unique entity number | Journal Entry | text, related to the counterpart | Printed on the invoice, because an invoice to a registered entity must show it. |

---

## 6. Export permits

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_sg_permit_number` | Permit number | Journal Entry | text | The export permit number, which is what makes a zero-rated export defensible. |
| `l10n_sg_permit_number_date` | Permit date | Journal Entry | date | The date of that permit. |

---

## 7. Bank accounts

The package adds the local bank account format and its validation, so that a local account number
can be entered and checked without an international bank account number.

---

## 8. The international invoice profile

A separate package builds the international invoice payload profile the country has adopted. It
carries the unique entity number as the party identifier and the tax registration number as the tax
scheme identifier.

---

## 9. Acceptance scenarios

**Given** a company in Singapore with no accounting,
**when** the Singaporean template is loaded,
**then** 135 accounts, five tax groups and two fiscal positions exist and the default sales tax is
the 9 percent tax.

**Given** an invoice line of 1,000.00 with the 9 percent tax,
**when** the totals are computed,
**then** the tax is 90.00 and the total is 1,090.00.

**Given** a customer that is a registered entity with the unique entity number `201912345K`,
**when** an invoice is issued to it,
**then** the invoice carries and prints that number.

**Given** an export invoice with the permit number `SG2024-000123` dated 3 May,
**when** the invoice is printed,
**then** the permit number and its date appear on it and the zero rate is justified.
