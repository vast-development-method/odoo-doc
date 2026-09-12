# Colombia

The Colombian package supplies one chart of accounts template built on the single national accounting plan, 378 accounts and 401 account groups, 65 taxes covering the value-added tax, the consumption tax, the industry and commerce tax withholding, the income tax withholding ladder and the equity contribution, 43 tax groups, and the identification type catalogue with the administration's document codes. It depends on the debit note capability, because a Colombian seller corrects upwards with a debit note rather than with a new invoice.

---

## 1. Template

| Property | Value |
|---|---|
| Template code | `co` |
| Display name | Colombia |
| Parent template | none |
| Fiscal country | Colombia |
| Account code length | 6 characters (the framework default) |
| Cost accounting flag | true |

### 1.1 Company-level values

| Company field | Value |
|---|---|
| Bank account code prefix | `1110` |
| Cash account code prefix | `1105` |
| Transfer account code prefix | `1115` |
| Default point of sale receivable account | `130507` |
| Gain exchange rate account | `421005` |
| Loss exchange rate account | `530505` |
| Cash discount write-off loss account | `530535` |
| Cash discount write-off gain account | `421040` |
| Cash difference income account | `428000` |
| Cash difference expense account | `532000` |
| Default sale tax | the 19 percent sales tax |
| Default purchase tax | the 19 percent purchase tax |
| Income account | `417500` |
| Expense account | `610000` |
| Stock journal | the inventory valuation journal |
| Stock valuation account | `140500` |
| Receivable account recorded as a Contact default | `130500` |
| Payable account recorded as a Contact default | `220500` |
| Product Category income account recorded as a default | `417500` |
| Product Category expense account recorded as a default | `610000` |
| Stock valuation account recorded as a company property | `140500` |

The stock valuation account `140500` names its stock expense account (`621000`) and its stock variation account (`146501`).

---

## 2. Chart of accounts

| Measure | Value |
|---|---|
| Accounts shipped | 378 |
| Account groups shipped | 401 |
| Code length | 6 characters |
| Translations shipped | Spanish |

**Account type distribution.**

| Account type | Count |
|---|---|
| Current Asset | 54 |
| Non-current Asset | 46 |
| Receivable | 8 |
| Equity | 25 |
| Expense | 43 |
| Cost of Revenue | 24 |
| Income | 35 |
| Current Liability | 87 |
| Non-current Liability | 22 |
| Payable | 2 |
| Off-balance | 32 |

The 401 account groups reproduce the national plan's class, group, account and sub-account levels, each with a starting and an ending prefix.

---

## 3. Taxes

65 taxes in five families. Several are group taxes whose children are the components the ledger must separate.

### 3.1 Value-added tax

| Group | Rate | Meaning |
|---|---|---|
| Value-added tax 0% | 0 | Exempt or excluded supplies. |
| Value-added tax 5% | 5 | The reduced rate. |
| Value-added tax 19% | 19 | The standard rate. |
| Value-added tax 16% | 16 | The historical standard rate, kept for documents dated before the change. |
| Value-added tax 8% | 8 | A historical reduced rate. |
| Value-added tax 4%, 2.4%, 2%, 0.75% | 4, 2.4, 2, 0.75 | Historical rates kept for old documents. |
| Covered goods | 0 | Goods covered by a tax-free day. |

### 3.2 Consumption tax

| Group | Rate |
|---|---|
| Consumption tax 4% | 4 |
| Consumption tax 8% | 8 |
| Consumption tax 16% | 16 |

### 3.3 Value-added tax withholding

| Group | Rate |
|---|---|
| Withheld value-added tax 0.75% | 0.75 |
| Withheld value-added tax 2.4% | 2.4 |
| Withheld value-added tax 2.85% | 2.85 |
| Withheld value-added tax 16% | 16 |

### 3.4 Industry and commerce tax withholding

The municipal turnover tax is withheld at a rate expressed in thousandths of the base.

| Group | Rate |
|---|---|
| Withheld industry and commerce tax 0% | 0 |
| Withheld industry and commerce tax 0.414% | 0.414 |
| Withheld industry and commerce tax 0.69% | 0.69 |
| Withheld industry and commerce tax 0.966% | 0.966 |
| Withheld industry and commerce tax 1.104% | 1.104 |
| Withheld industry and commerce tax 1.38% | 1.38 |

**Worked example.** A base of 5,000,000 local units with a rate of 0.966 percent gives `round(5,000,000 × 0.00966, 0) = 48,300`.

### 3.5 Income tax withholding

Sixteen rates, from 0 to 33 percent, because the published rate depends on the nature of the payment and on whether the payee is registered.

| Group |
|---|
| Withheld income tax 0%, 0.1%, 0.5%, 1%, 1.5%, 2%, 2.5%, 3%, 3.5%, 4%, 6%, 7%, 10%, 11%, 20%, 33% |

### 3.6 Equity contribution

| Group | Rate |
|---|---|
| Equity contribution 4% | 4 |
| Equity contribution 8% | 8 |
| Equity contribution 16% | 16 |

### 3.7 Income perception

| Group | Rate |
|---|---|
| Income tax perception 0% | 0 |

### 3.8 Group taxes

Where a supply carries both a value-added tax and a withholding, the package ships a group tax whose children are the two components, so that the user picks one tax and the entry carries two lines with their own accounts and their own report tags.

---

## 4. Tax groups

43 groups, all with the country Colombia. They do not name payable or receivable accounts: the closing entry uses the accounts of the taxes themselves, because the national plan gives each family its own liability account.

---

## 5. Fiscal positions

The package ships no fiscal position as data. The domestic behavior is the default, and a company that exports creates its own position. **Industry-standard completion**: a replacement should ship at least a domestic position and an export position, because the automatic detection of section 8 of [../workflows.md](../workflows.md) needs a position to select.

---

## 6. Identification types

The package extends the shared Latin American identification type catalogue with the administration's document code, so that the code the electronic document carries can be derived from the class the operator chose.

| Class | Administration document code |
|---|---|
| Civil registry | `11` |
| Identity card | `12` |
| Citizenship card | `13` |
| Foreigner identity card | `21` |
| Foreigner card | `22` |
| Tax identification number | `31` |
| Passport | `41` |
| Foreign document | `42` |
| Foreign tax identification number | `50` |
| Identification number for foreigners without a tax number | `91` |

---

## 7. Debit notes

The package depends on the debit note capability. A Colombian seller that must increase an amount already invoiced issues a **debit note**, which is a separate document class with its own numbering, rather than a second invoice. The debit note wizard carries the document type selector of the shared Latin American mechanism.

---

## 8. Point of sale

The point of sale package adds the identification type and the identification number to the till, so that a receipt issued to an identified buyer carries the data the electronic receipt requires.

---

## 9. Acceptance scenarios

**Given** a company in Colombia with no accounting,
**when** the Colombian template is loaded,
**then** 378 accounts and 401 account groups exist, the bank prefix is `1110`, the cash prefix is `1105`, the transfer prefix is `1115`, the cash difference accounts are `428000` and `532000`, and 43 tax groups exist.

**Given** a base of 5,000,000 and a 0.966 percent industry and commerce withholding,
**when** the tax is computed,
**then** the withheld amount is 48,300.

**Given** a posted customer invoice that must be increased,
**when** a debit note is created from it,
**then** the debit note carries its own document class and its own number and references the original invoice.
