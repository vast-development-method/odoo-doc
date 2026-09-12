# Norway

The Norwegian package supplies one chart of accounts template with 745 accounts, 148 tax rows and
five tax groups, a standard tax code on every tax that the administration expects in the return, and
the national register number on companies and contacts.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `no` |
| Display name | Norway |
| Parent template | none |
| Fiscal country | Norway |
| Account code length | 6 characters |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Currency | Norwegian krone |
| Default sale tax | the 25 percent sales tax |
| Default purchase tax | the 25 percent purchase tax |

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Account rows shipped | 745 |
| Account group rows shipped | none |
| Tax rows shipped | 148 |
| Tax group rows shipped | 5 |
| Fiscal position rows shipped | none |

The chart follows the national standard numbering, in which the first digit fixes the class, so the
745 accounts are already grouped by their codes without a separate group structure.

---

## 3. Taxes

| Family | Rates | Scope |
|---|---|---|
| Domestic sales | 25, 15, 12, 11.11, 0 percent | sale |
| Domestic purchases | 25, 15, 12, 0 percent | purchase |
| Import of goods | 25, 15 percent | purchase |
| Import of services with reverse charge | 25 percent | purchase |
| Export | 0 percent | sale |
| Exempt | none | sale and purchase |

The 11.11 percent rate is the effective rate applied to the sale of raw fish, which the law states
as a deduction from a gross amount rather than as an addition to a net one.

**Worked example.** A domestic sale of 4,000.00 at 25 percent produces a tax of 1,000.00 and a total
of 5,000.00. A sale of 4,000.00 at the reduced 15 percent rate produces 600.00.

---

## 4. The standard tax code

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_no_standard_code` | Standard tax code | Tax | text | The code the administration publishes for each standard tax treatment. It is what the periodic return groups by, so every tax the company uses must carry one. |

---

## 5. The register number

| Identifier | Full name | Host | Type | Meaning |
|---|---|---|---|---|
| `l10n_no_bronnoysund_number` | Register of legal entities number | Contact | text | The number the national register of legal entities allocates. |
| `l10n_no_bronnoysund_number` | Register of legal entities number | Journal Entry | text | Related to the counterpart's number; printed on the invoice, because an invoice to a registered entity must show it. |

---

## 6. Statutory reporting

One report structure is shipped, the periodic tax return. Its first line is the balance, and its
sections are the sales of goods and services in Norway by rate, the exports, the imports and the
deductible tax, each identified by the standard tax code of the taxes that feed it.

---

## 7. Journals

The package marks the journal used for the return and sets the tax closing journal, so that the
closing entry lands where the auditor expects it.

---

## 8. Acceptance scenarios

**Given** a company in Norway with no accounting,
**when** the Norwegian template is loaded,
**then** 745 accounts and five tax groups exist and the default sales tax is the 25 percent tax.

**Given** an invoice line of 4,000.00 with the 25 percent tax,
**when** the totals are computed,
**then** the tax is 1,000.00 and the total is 5,000.00.

**Given** an invoice line of 4,000.00 with the 15 percent tax,
**when** the totals are computed,
**then** the tax is 600.00 and the total is 4,600.00.

**Given** a tax with no standard tax code,
**when** the return is run,
**then** the amounts of that tax are not grouped into any section, which is why every tax the
company uses must carry a code.

**Given** a customer that is a registered legal entity with the register number `987654321`,
**when** an invoice is issued to it,
**then** the invoice carries and prints that number.
