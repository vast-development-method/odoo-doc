# Financial Reporting — Configuration

This file specifies everything a deployment configures for this domain: the settings, the system
parameters, the sequences and numbering formats, the default records shipped with the
application, the security groups, the complete access rights matrix, the record rules and the
scheduled jobs.

---

## 1. Company settings

These settings live on the Company record and are edited from the accounting settings page. Each
is company-scoped: a database with several companies holds one value per company.

### 1.1 Fiscal periods

| Setting (storage name) | Type | Default | Meaning |
|---|---|---|---|
| Fiscal year last day (`fiscalyear_last_day`) | Integer | 31 | The day of the month on which the fiscal year ends. Required. |
| Fiscal year last month (`fiscalyear_last_month`) | Selection of the twelve months | December | The month in which the fiscal year ends. Required. |

Together these two define every fiscal-year-relative date scope
([`calculations.md`](calculations.md) §9). A company that closes on 30 June sets the day to 30 and
the month to June; the fiscal year containing 2026-05-17 then runs 2025-07-01 to 2026-06-30.

A company whose fiscal period is longer or shorter than twelve months declares explicit fiscal
year records, each with a name, a start date and an end date. Those records override the
day-and-month rule for the periods they cover; outside them the default rule resumes.

### 1.2 Lock dates

| Setting (storage name) | Type | Tracked | Meaning |
|---|---|---|---|
| Global Lock Date (`fiscalyear_lock_date`) | Date | yes | Any entry up to and including that date is postponed to a later time, in accordance with its journal's sequence. |
| Tax Return Lock Date (`tax_lock_date`) | Date | yes | Any entry with taxes up to and including that date is postponed to a later time. Set automatically when a tax closing entry is posted. |
| Sales Lock Date (`sale_lock_date`) | Date | yes | Any sales entry up to and including that date is postponed. |
| Purchase Lock Date (`purchase_lock_date`) | Date | yes | Any purchase entry up to and including that date is postponed. |
| Hard Lock Date (`hard_lock_date`) | Date | yes | Any entry up to and including that date is postponed. Irreversible and admits no exception. |

The first four are **soft**: a lock date exception can relax them. The fifth is **hard**.

Each also has a per-user derived counterpart — the effective lock date for the requesting user,
after applying any exception granted to that user. The derived value depends on the requesting
user and on whether exceptions are being ignored, and is invalidated whenever the stored date
changes, an exception is created, or an exception is revoked.

### 1.3 Tax return

| Setting | Type | Meaning |
|---|---|---|
| Periodicity | Selection: monthly, every two months, quarterly, every four months, twice a year, yearly | How often the tax return is filed. Determines the `previous_return_period` date scope and the periods the return list proposes. |
| Deadline | Integer, a day of the month | When the reminder to file is raised. |
| Journal | Link to a Journal of type `general` | Where the tax closing entry is posted. |
| Opening date | Date | The first date from which accounting data exists; used to bound the "from the very start" scopes and the opening entry. |

These four are set from the accounting dashboard, through the actions offered on the tax return
journal: set the company data, set the periods, and review the chart of accounts.

### 1.4 Reporting

| Setting (storage name) | Type | Default | Meaning |
|---|---|---|---|
| Dynamic Reports (`module_account_reports`) | Boolean, installs a package | off | Turns on the dynamic reporting engine and the statement reports. When off, only the invoice analysis report and the printable documents are available. |
| Restricted Audit Trail (`restrictive_audit_trail`) | Boolean, tracked | off | Prevents deletion of the logs attached to accounting records. See §1.5. |
| Forced Audit Trail (`force_restrictive_audit_trail`) | Boolean, derived | — | True when the company's country package requires the restricted audit trail. When true, the setting above cannot be switched off. |

*Validation:* switching the restricted audit trail off while the forced flag is true raises
**Can't disable restricted audit trail: forced by localization.**

### 1.5 What the restricted audit trail protects

When the flag is on for a company, tracked-change messages become immutable for:

- that company's record itself, reached through any of its Journal Items;
- every Journal Entry of that company;
- every account that is in use and belongs to that company;
- every tax that appears as the tax line of any of that company's Journal Items;
- every partner that appears on any of that company's Journal Items.

The exact refusals are listed in [`business-rules.md`](business-rules.md) §11, rule I-03.

### 1.6 Inalterability

| Setting | Where | Meaning |
|---|---|---|
| Secure Posted Entries with Hash | On a Journal, advanced settings | Turns the journal into a *restricted* journal: every entry posted in it joins a hash chain. Once an entry has been posted in a restricted journal, the setting can no longer be switched off and no secured entry can be edited. |
| Secure Entries (an action, not a setting) | Accounting menu | Secures every posted entry, in every journal, up to a chosen date, independently of the journals' own settings and of their types. |

Two indicators appear on a secured entry: a lock marker next to its posted state, and a secured
flag in its other-information tab. A "not secured" filter on the entry and item lists finds
posted entries that are not yet secured.

---

## 2. System parameters

| Parameter | Type | Default | Meaning |
|---|---|---|---|
| Integrity hash batch size | Integer | 1000 | How many entries the hash integrity check fetches per batch. Between batches the read cache is cleared, so a very long chain can be verified without holding it all in memory. |
| Maximum hash version | Integer | 4 | The highest hash version the integrity check will try when recomputing an entry's hash. Version one hashes the entry's date, journal and company and each item's debit, credit, account and partner; versions two to four add the entry's number and each item's label; version three and above render monetary values with exactly the currency's decimal places; version four and above prefix the stored hash with its version. |
| Prefix groups threshold | Integer, per report | 4000 | Stored on the Report Definition, not globally. See [`calculations.md`](calculations.md) §11.4. |
| Load more limit | Integer, per report | empty | Stored on the Report Definition. See [`calculations.md`](calculations.md) §11.3. |

---

## 3. Sequences and numbering

This domain introduces **no sequence of its own**. Two numbering facilities are consumed:

### 3.1 The tax closing entry's number

The closing entry is numbered by the sequence of the **tax return journal**, following the
numbering rules of the [General Ledger](../general-ledger/README.md) domain. The format is the
journal's own: a prefix derived from the journal code, a period part derived from the entry's
accounting date, and a counter, for example a code, a slash, the year, a slash and a five-digit
counter. A rebuild must not give the closing entry a numbering of its own.

### 3.2 The secured sequence number

Entries that join a hash chain also receive a gap-free secured sequence number, used by the
integrity check to order the chain across sequence prefixes. It is assigned by the General Ledger
domain when the entry is hashed.

---

## 4. Default records shipped with the application

### 4.1 Report definitions

Three report definitions ship with the accounting application itself:

| External identifier | Name | Root | Columns | Settings |
|---|---|---|---|---|
| `account.generic_tax_report` | Generic Tax report | — | `net` (monetary), `tax` (monetary) | multi-company filter `tax_units`; foreign value-added tax allowed; default opening date filter `previous_return_period`; only tax exigible |
| `account.generic_tax_report_account_tax` | Group by: Account > Tax | `account.generic_tax_report` | `net`, `tax` | availability `always` |
| `account.generic_tax_report_tax_account` | Group by: Tax > Account | `account.generic_tax_report` | `net`, `tax` | availability `always` |

One hundred and sixty-two further definitions ship with the country packages; they are enumerated
in [`calculations.md`](calculations.md) §15.3.

### 4.2 Account tags for the cash flow statement

| External identifier | Name | Applicability |
|---|---|---|
| `account.account_tag_operating` | Operating Activities | accounts |
| `account.account_tag_financing` | Financing Activities | accounts |
| `account.account_tag_investing` | Investing & Extraordinary Activities | accounts |

These three cannot be deleted; see [`business-rules.md`](business-rules.md) rule T-02. The chart
of accounts templates apply them to the appropriate accounts when a company's chart is installed.

### 4.3 Printable document definitions

| External identifier | Name | Applies to | Produces |
|---|---|---|---|
| `account.action_report_account_hash_integrity` | Hash integrity result | the Company | The integrity findings as a printable document. |

### 4.4 Menu structure

| External identifier | Menu path | Visible to |
|---|---|---|
| `account.menu_finance_reports` | Reporting | read-only accounting group, invoicing group |
| `account.account_reports_legal_statements_menu` | Reporting → Statement Reports | read-only accounting group, basic accounting group |
| `account.account_reports_partners_reports_menu` | Reporting → Partner Reports | inherited |
| `account.account_reports_taxes_and_fiscal_menu` | Reporting → Taxes & Fiscal | inherited |
| `account.account_reports_management_menu` | Reporting → Management | inherited |
| `account.menu_action_account_invoice_report_all` | Reporting → Management → Invoice Analysis | inherited |
| `account.menu_action_analytic_reporting` | Reporting → Management → Analytic Report | read-only accounting group |
| `account.account_report_folder` | Configuration → Reporting | read-only accounting group |

Each root report creates its own menu entry under Reporting when the designer asks for one; the
entry opens a window action of the report kind carrying the report's identifier.

---

## 5. Security groups

| External identifier | Name | Implies | Role in this domain |
|---|---|---|---|
| `account.group_account_invoice` | Invoicing | the internal user group | Invoices, payments and basic invoice reporting. Reads the report definitions but not the external values. Cannot see journal entries, reconciliation or the statement reports. |
| `account.group_account_basic` | Basic | the invoicing group | Additional accounting features such as basic bank reconciliation; still no journal entries. |
| `account.group_account_readonly` | Show Accounting Features - Readonly | the internal user group | Sees everything, including journal entries, advanced configuration and every report. Changes nothing. |
| `account.group_account_user` | Show Full Accounting Features | the basic group and the read-only group | The accountant: does everything except advanced configuration. Required to run the hash integrity check. |
| `account.group_account_manager` | Administrator | the invoicing group | Full access including configuration. The only group that may create or change report definitions, edit manual figures, validate tax returns and grant lock date exceptions. |
| `account.group_account_secured` | Show Inalterability Features | — | Reveals the inalterability controls: the secured indicators, the not-secured filter and the secure-entries action. |

In a deployment with invoicing only, the invoicing group and the administrator group are the two
that matter; the others give a shallow view of accounting features.

---

## 6. Access rights matrix

One row per entity and group. `C` = create, `R` = read, `U` = update, `D` = delete.

| Entity | Invoicing | Basic | Read-only | Accountant | Administrator |
|---|---|---|---|---|---|
| Report Definition (`account.report`) | — | R | R | R | C R U D |
| Report Line (`account.report.line`) | — | R | R | R | C R U D |
| Report Expression (`account.report.expression`) | — | R | R | R | C R U D |
| Report Column (`account.report.column`) | — | R | R | R | C R U D |
| Report External Value (`account.report.external.value`) | — | — | R | R | C R U D |
| Account Tag (`account.account.tag`) | R | R | R | R | C R U D |
| Tax Group (`account.tax.group`) | R | R | R | R | C R U D |
| Journal Entry and Journal Item | per the General Ledger domain | | | | |

Notes:

1. The basic accounting group and the read-only group get read access to the four definition
   tables so that a report can be rendered; the definitions are not secret.
2. The accountant group inherits read access through the basic and read-only groups; it is not
   granted create, update or delete on the definitions, because a report definition is
   configuration, not bookkeeping.
3. The external value table is invisible to the invoicing group and the basic group: manual
   declaration figures are not invoicing data.
4. The right to *edit a cell* in a rendered report is the right to create and update an external
   value, that is the administrator group. A read-only accountant sees the editable cell and
   cannot change it.

---

## 7. Record rules

| Rule | Entity | Applies to | Condition |
|---|---|---|---|
| Report External Value multi-company | Report External Value | all groups | The record's company must be one of the companies currently allowed for the user. |

The four definition tables carry **no** record rule: report definitions are company-neutral and
every user who may read them may read all of them. Company scoping of the *figures* happens at
evaluation time, through the company filter and through the company of the Journal Items read.

The Journal Entry and Journal Item rules of the
[General Ledger](../general-ledger/configuration.md) domain do apply to every drill-down, with
one deliberate exception: the hash integrity check reads entries with full privileges, because a
partially visible entry would hash differently. Only the digest leaves that privileged read.

---

## 8. Scheduled jobs

This domain adds **no scheduled job of its own**. Three jobs of adjacent domains affect it:

| Job | Frequency | Effect on reporting |
|---|---|---|
| Post draft entries with automatic posting enabled and an accounting date up to today | daily, at two o'clock | Moves entries from draft to posted, so a report run after the job includes entries a report run before it excluded, when the draft-entries filter is off. |
| Send invoices automatically | daily | No direct effect; it may post invoices that were waiting to be sent. |
| Tax return reminder | per the company's periodicity and deadline | Raises the activity that tells the accountant a return is due. The return record itself is created when its period elapses. |

A rebuild is free to create the pending tax return records either eagerly, by a scheduled job
running at the start of each period, or lazily, when the tax return list is opened. The lazy form
is preferable because it avoids creating returns for companies that are later archived.

---

## 9. Configuring a country's tax report end to end

The full sequence a country package, or a user setting up a custom return, performs:

1. **Create the report.** Name it, set its root report to the generic tax report, set its
   availability to country matches and set the country.
2. **Create the boxes.** One Report Line per box of the legal form, with a code equal to the box
   identifier the law uses. Nest them to match the form's sections.
3. **Create the tag expressions.** On every box that receives amounts from taxes, create one or
   two expressions with the tax tag engine: label `base` for the untaxed amount and label `tax`
   for the tax amount. The formula is the grid identifier; the leading minus is added where the
   box must show a credit balance as positive. Writing the expression creates the tag.
4. **Create the adjustment expressions.** On every box the law lets the filer correct, add an
   expression with the external engine, the formula `sum` and the subformula
   `editable;rounding=2` — or `editable;rounding=0` where the form takes whole units.
5. **Create the subtotal expressions.** On each subtotal box, an aggregation expression summing
   the codes of its constituents.
6. **Create the result expressions.** On the pay box, an aggregation expression with
   `if_above(CUR(0))`; on the reclaim box, the same arithmetic with `if_below(CUR(0))` and a
   negation. Where the law requires a carry-over, add the three-expression pattern of
   [`calculations.md`](calculations.md) §12.2.
7. **Configure the taxes.** For every tax used in the country, set its tax grids to the tags
   created in step 3, on both the invoice and the refund repartition sets, matching the sign
   convention chosen in step 3.
8. **Configure the tax groups.** Give every tax a tax group, and give every tax group a tax
   payable account, a tax receivable account and, where the country has instalments, an advance
   tax payment account.
9. **Check the closing visibility.** For every tax whose tax repartition line should not appear
   in the closing entry — a non-deductible tax posted to an expense account, for instance — leave
   the use-in-tax-closing flag off. The flag defaults to on for a repartition line of the type
   "of tax" whose account is set and whose account's internal group is neither income nor
   expense.
10. **Test.** Post an invoice, a bill and a credit note using the country's taxes; check the
    rendered report; validate a closing and check the entry.

---

## 10. Configuring a consolidation

1. Ensure every company to be consolidated uses account codes that agree where the accounts
   correspond. Where they do not, use the account code mapping of the chart of accounts to give
   an account a second code under which it merges with its counterpart.
2. Ensure the report's multi-company switch is `selector`.
3. Select the companies. The presentation currency is that of the first selected company.
4. Choose the currency translation mode on the report: keep stored company-currency amounts and
   show the residual on a cumulative translation adjustment line, or translate everything at the
   rate of the report date.
5. Remember that intercompany balances are **not** eliminated; see
   [`workflows.md`](workflows.md) §12.

---

## 11. Configuring a tax unit

1. Create the tax unit, naming its member companies, its country and its representative.
2. Set the report's multi-company switch to `tax_units`.
3. Give each member company the intercompany accounts that the unit's closing needs.
4. Validate the unit's return from the representative company; one closing entry per member is
   produced, and the representative's entry carries the intra-unit settlement lines
   ([`accounting-effects.md`](accounting-effects.md) §6).

---

## 12. Configuration checklist for a new deployment

| Step | Where | Why |
|---|---|---|
| Set the fiscal year last day and month | company settings | Every fiscal-year-relative date scope depends on it. |
| Set the accounting opening date | accounting periods | Bounds the "from the very start" scopes. |
| Set the tax return periodicity, deadline and journal | accounting periods | Required before any return can be validated. |
| Give every tax group a payable and a receivable account | tax groups | Required before any return can be validated. |
| Decide the restricted audit trail | accounting settings | Cannot be switched off later where a country forces it. |
| Decide which journals run in restricted mode | journals | Cannot be switched off once an entry is posted in them. |
| Install the country package | modules | Brings the national report definitions, the taxes, the grids and the tax groups. |
| Verify the cash-flow tags on the chart | chart of accounts | An untagged account falls into the unclassified section of the cash flow statement. |
| Set the global lock date after each closed year | lock dates | Freezes the audited period. |
