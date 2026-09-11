# Financial Reporting — Interfaces

This file specifies every interface the domain exposes: the navigation a user sees, the views and
what each shows, the named remote operations with their inputs and outputs, the routes, the
printable documents, the export formats, the notifications, and the external service contracts.

---

## 1. Navigation

### 1.1 The reporting menu

| Menu path | Contents |
|---|---|
| Reporting | The root of every report. Visible to the read-only accounting group and to the invoicing group. |
| Reporting → Statement Reports | Balance sheet, profit and loss, executive summary, cash flow statement. Visible to the read-only accounting group and the basic accounting group. |
| Reporting → Partner Reports | Partner ledger, aged receivable, aged payable. |
| Reporting → Taxes & Fiscal | The tax report and the national returns available for the company's country, plus the statistical trade declaration and the European sales list where the country requires them. |
| Reporting → Audit Reports | General ledger, trial balance, journal audit, audit trail, data inalterability check. |
| Reporting → Management | Invoice analysis, analytic report. |
| Configuration → Reporting | The report designer: the list of Report Definitions. Visible to the read-only accounting group; editable by the administrator group only. |
| Accounting → Tax Returns | The list of pending and past tax returns. |
| Accounting → Lock Dates | The five lock dates and the lock date exceptions. |
| Accounting → Secure Entries | The action that secures posted entries up to a date. Visible when the inalterability group is active, or in the technical mode. |

### 1.2 Window actions

| Action | Opens | Notes |
|---|---|---|
| One action per root report, of the *report* kind | The rendered report | Carries the report's identifier in its context. Created by the designer's "create menu entry" operation and by the country packages. |
| Accounting Reports | The list of Report Definitions | The designer. |
| Hash integrity result | A printable document over the active company | Produces the data inalterability check report. |
| Tax Returns | The tax return list | Reached from the dashboard tile of the tax return journal, and from the accounting menu. |
| Audit Trail | The list of tracked-change messages | Filtered to accounting-relevant records. |

### 1.3 The dashboard tile of the tax return journal

The tax return journal's tile on the accounting dashboard shows, in order:

1. Any configuration action still to perform: set the company data, set the periods, review the
   chart of accounts.
2. The next return due, with its period and its deadline; the deadline is shown in a warning
   colour once it has passed.
3. A link to the tax return list and a link to the posted closing entries of the journal.

---

## 2. Views

### 2.1 The rendered report

The rendered report is not an ordinary record list. It is a table with:

| Element | Contents |
|---|---|
| Title bar | The report's name; the variant selector when the report is a root with variants; the section tabs when the report is composite. |
| Filter bar | One control per enabled filter: date, comparison, journals, analytic, entries type, partners, account type, unreconciled, account groups, budgets, saved journal-item filters, decimal rounding unit, and the display options (hide lines at zero, split horizontally, unfold all). |
| Buttons | The printable document, the spreadsheet workbook, the filing file where one exists, the save-to-documents action, and the designer cog in technical mode. |
| Search box | Present when the report's search-bar flag is set; filters the visible lines by name. |
| Column headers | One group per period, current first; each group repeating the report's columns; then the growth column when applicable. |
| Rows | One per rendered line: an expander when unfoldable, the name indented by the level, one cell per column, an annotation marker when an annotation exists, and an audit marker on auditable figures. |

**Cell behaviors.**

| Cell kind | Behavior on activation |
|---|---|
| Auditable figure | Opens the audit drill-down (§4.3), or the column's custom audit action when one is set. |
| Editable figure | Becomes an input; committing writes an External Value. |
| Non-auditable figure | Inert. |
| Line name with an action | Runs the line's action. |
| Line name without an action, unfoldable | Expands or collapses. |

### 2.2 The report designer — Report Definition form

| Section | Fields |
|---|---|
| Header | Name; country; availability; root report; chart of accounts; sequence; active. |
| Lines tab | The line tree, editable inline: name, code, level, sequence, foldable, hide if zero, print on new page, grouping key, action; each line opens a dialog holding its expressions. |
| Columns tab | Name, expression label, sequence, display type, sortable, blank if zero, custom audit action. |
| Options tab | Every filter switch of [`entities.md`](entities.md) §1.7, plus only-tax-exigible, allow-foreign-value-added-tax, default opening date filter, currency translation, integer rounding, load more limit, search bar, prefix groups threshold. |
| Sections tab | The composite flag and the ordered section list. |
| Cog actions | Create menu entry; duplicate. |

### 2.3 The expression dialog

| Tab | Fields |
|---|---|
| Definition | Label; computation engine; formula; subformula. |
| Options | Date scope; display type; blank if zero; is growth good when positive; auditable; carry over to. |

### 2.4 The tax return list and form

**List columns:** the period; the deadline, in a warning colour once passed; the company and, where applicable, its branches; the return type; the three step markers (review, submit, pay), each turning to a completed state as it is done; the number of pending or passed checks, in a warning colour or a success colour; the amount to pay or reclaim; the primary and secondary action buttons; an overflow menu.

**Overflow menu:** view the closing entry; reverse the closing entry; reset to draft; mark as completed; export all returns of the period as a spreadsheet workbook.

**Form:** the checks as cards, each coloured by its state — success for reviewed, supervised or passed, danger for anomaly, neutral for to review — each opening the records that made it fail; the Validate button; the message log recording every state change of every check.

### 2.5 The check configuration list

Visible in technical mode under Configuration → Checks. One record per check: its name, the
return types it applies to, the country it applies to, whether it is optional, and how it is
evaluated.

### 2.6 The external value list

Visible in technical mode. Columns: date, name, target line, target expression label, numeric
value, text value, company, origin line, origin expression label, country. Used to audit what
was typed by hand and what was carried over.

### 2.7 The audit trail list

Columns: date and time; author; the record's model and display name; the description, which is
the message title followed by one line per tracked value of the shape *old value* ⇨ *new value*
(*field label*); and a marker when the message is protected by the restricted audit trail.

Filters: by Journal Entry, by partner, by account, by tax, by company, by protected flag, and by
free text matched against the old and new values.

---

## 3. Named remote operations

These are the operations an external caller may invoke. Inputs and outputs are described
abstractly; a rebuild maps them onto its own transport.

### 3.1 Read a report

| | |
|---|---|
| **Name** | Get report lines |
| **Inputs** | The report identifier; the option set of [`calculations.md`](calculations.md) §2.1. |
| **Output** | The resolved option set, echoed back with every default filled in; the ordered column definitions, one group per period; the ordered rows, each with an identifier, a level, a name, a parent identifier, a foldable flag, an unfolded flag, an optional action, an optional annotation, and one cell per column carrying a raw value, a formatted value, a display type, an auditable flag and an editable flag. |
| **Side effects** | None. |

### 3.2 Expand a line

| | |
|---|---|
| **Name** | Expand unfoldable line |
| **Inputs** | The report identifier; the option set; the line identifier; an optional offset for a continuation. |
| **Output** | The sub-lines in the same shape as §3.1, plus a continuation marker when more remain. |
| **Side effects** | None. |

### 3.3 Audit a figure

| | |
|---|---|
| **Name** | Open the items behind a figure |
| **Inputs** | The report identifier; the option set; the line identifier; the expression label; the period index. |
| **Output** | A window action over Journal Items carrying the drill-down restriction, a grouping by account and a default ordering by date. |
| **Failure** | Refused when the expression is not auditable. |

### 3.4 Write a manual figure

| | |
|---|---|
| **Name** | Set an editable figure |
| **Inputs** | The report identifier; the option set; the line identifier; the expression label; the value, numeric or textual. |
| **Output** | The refreshed report, as in §3.1. |
| **Side effects** | One Report External Value created, updated or deleted. |
| **Failure** | Refused without the administrator group; refused when the expression is not editable; refused when the date is locked. |

### 3.5 Annotate a line

| | |
|---|---|
| **Name** | Set a line annotation |
| **Inputs** | The report identifier; the line identifier; the period end date; the text. |
| **Output** | Acknowledgement. |
| **Side effects** | One annotation record created, updated or deleted. |

### 3.6 Export

| | |
|---|---|
| **Name** | Export report |
| **Inputs** | The report identifier; the option set; the format — printable document, spreadsheet workbook, or a country filing format named by its identifier. |
| **Output** | The generated file, with its name and its media type. |
| **Side effects** | None, unless the caller also asks for the file to be filed, in which case one attachment is created. |

### 3.7 Validate a tax return

| | |
|---|---|
| **Name** | Validate return |
| **Inputs** | The return identifier. |
| **Output** | The return in its new state, and the identifier of the posted closing entry. |
| **Side effects** | Carry-over external values written; one closing Journal Entry per company created and posted; the tax return lock date moved; the rendered report attached. |
| **Failure** | Any of the preconditions of [`business-rules.md`](business-rules.md) §6. |

### 3.8 Run the integrity check

| | |
|---|---|
| **Name** | Check hash integrity |
| **Inputs** | None beyond the active company. |
| **Output** | A mapping with the printing date and the list of findings, each carrying the journal name with its sequence prefix, the restricted-mode flag, the status, the message, and — for a verified finding — the first and last entry names, hashes and dates. |
| **Failure** | Refused without the accountant group, with *Please contact your accountant to print the Hash integrity result.* |

---

## 4. Routes

The domain exposes no route of its own. Three routes of adjacent domains carry its output:

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| The printable-document route of the application, addressed with the integrity report's name and the company's identifier | read | authenticated user | Returns the data inalterability check report as a printable document. |
| The printable-document route addressed with a report's identifier and its option set | read | authenticated user | Returns a rendered report as a printable document. |
| The attachment download route | read | authenticated user, or a document access token | Returns a filed export. |

Every route requires an authenticated session; none is public. No route of this domain accepts a
portal user.

---

## 5. Printable documents

### 5.1 The rendered report

| Aspect | Rule |
|---|---|
| Header | The company name and address block; the report name; the company or tax unit; the period; the presentation currency and rounding unit. |
| Column headers | Repeated at the top of every page. |
| Rows | One per rendered line, indented by the level; a line whose print-on-new-page flag is set starts a new page, and everything after it follows on that page. |
| Folding | Lines folded in the rendered view are printed folded; unfolded lines are printed expanded. |
| Annotations | Printed as numbered footnotes at the end. |
| Horizontal split | When the report supports it and the option is on, the left-side subtree and the right-side subtree are printed side by side. |
| Page format | The application's default page format, portrait. |

### 5.2 The data inalterability check report

Produced from the active company. Structure:

1. A heading: *Data Inalterability Check Report*, followed by the printing date.
2. A **configuration review** table with three columns: *Journal (Sequence Prefix)*, *Restricted*
   and *Check*. One row per finding. The restricted column shows `V` when the journal runs in
   restricted mode and `X` otherwise. The check column shows the finding's message.
3. When at least one finding is verified, a new page begins with a **data consistency check**
   table of five columns: *Journal*, *First Hash*, *First Entry*, *Last Hash*, *Last Entry*. The
   entry columns show the entry's number above its date.
4. A closing statement: *The hash chain is compliant: it is not possible to alter the data
   without breaking the hash chain for subsequent parts.*

---

## 6. Export formats

### 6.1 Spreadsheet workbook

| Aspect | Rule |
|---|---|
| Worksheets | One per section of a composite report; one otherwise. Named after the section. |
| Row 1 | The report's name, the company or tax unit and the period. |
| Row 2 | The column headers, one group per period. |
| Data rows | One per rendered line. The first cell carries the line's name, prefixed by two spaces per level of indentation. |
| Cell types | Monetary cells carry numbers with a currency number format; percentage cells carry a fraction with a percentage format; integer cells carry whole numbers; date cells carry dates; string cells carry text. Nothing is exported as pre-formatted text that a spreadsheet would have to parse back. |
| Folding | As printed: folded lines stay collapsed. |
| Totals | Exported as values, not as formulas, so that the exported figure always equals the rendered one. |

### 6.2 Country filing files

Each country package that supports electronic filing declares a format. The contract is the same
for all of them:

| Aspect | Rule |
|---|---|
| Addressing | A box of the legal file is mapped to a report line by that line's **code**, never by its name and never by its position. Renaming a line does not break a filing format; changing its code does. |
| Values | The value written is the figure of the expression whose label the format names, after the report's rounding, in the presentation currency, with the sign convention the authority requires. |
| Identification | The company's registration identifiers, its address and its representative are read from the company record. |
| Period | Written in the authority's own encoding of the period. |
| Missing box | A box the format requires whose line code is absent from the definition aborts generation with a message naming the code. |
| Attachment | The generated file is attached to the tax return. |

### 6.3 Filing a report into the documents application

The save-to-documents action takes a format, a document name, a target folder and a set of tags,
and creates one document holding the generated file.

### 6.4 Import

This domain imports nothing at run time. Report definitions are imported as ordinary data records
through the application's generic import facility, which accepts a tabular file whose columns are
field paths; a nested line tree is imported by giving each line's parent through the parent's
external identifier. Two practical rules:

1. Import the lines in tree order, parents first, or the parent-precedes-child validation fails.
2. Import the expressions after the lines, because an expression requires its line.

---

## 7. Notifications

| Notification | Trigger | Recipients | Contents |
|---|---|---|---|
| Tax return due | The deadline configured on the company is reached and the period's return is not yet submitted. | The users responsible for the tax return journal. | The period, the deadline and a link to the return. |
| Tax return deadline passed | The deadline has passed and the return is not submitted. | The same. | The same, marked as overdue. |
| Check state changed | A check is waived or fails again. | The followers of the return. | Written to the return's message log: which check, which state, which user, when. |
| Lock date changed | Any lock date is written on a company. | The followers of the company. | The tracked field, its old value and its new value. |
| Restricted audit trail changed | The flag is written on a company. | The followers of the company. | The tracked field, its old value and its new value. |

No email template belongs to this domain; every notification above is a tracked-field message or
an activity, delivered by the messaging domain.

---

## 8. External service integrations

### 8.1 Accounting platform synchronization

An external accounting platform may read the ledger and the reports through the application's
generic remote interface, authenticating with a per-user key. The contract is:

1. The external service is given the user's electronic mail address, a key created for that
   user, the address of the deployment and the name of the database.
2. The key is personal and grants the full access of that user's account. It can be read only at
   the moment it is created; a lost key is replaced, never recovered.
3. A key may be created for one database or for every database a user manages.
4. The external service reads the same entities this specification describes; no reporting-specific
   endpoint exists.

### 8.2 Electronic filing channels

Where a country's authority offers a direct channel, the country package implements the
transmission and stores the acknowledgement reference on the tax return. Where it does not, the
return shows the manual filing instructions and the user marks the return as submitted.

---

## 9. The shipped report catalogue, as a user sees it

| Report | Menu | Kind | Columns | Primary drill-down |
|---|---|---|---|---|
| Balance Sheet | Statement Reports | dynamic account tree under a static frame | Balance | account, then Journal Items |
| Profit and Loss | Statement Reports | dynamic account tree under a static frame | Balance | account, then Journal Items |
| Executive Summary | Statement Reports | static ratios | Balance | account for the monetary lines only |
| Cash Flow Statement | Statement Reports | static sections over tagged accounts | Balance | counterpart account, then Journal Items |
| General Ledger | Audit Reports | dynamic, one line per account | Debit, Credit, Balance | Journal Items, then the entry |
| Trial Balance | Audit Reports | dynamic, one line per account | Initial debit and credit, period debit and credit, end debit and credit | Journal Items |
| Partner Ledger | Partner Reports | dynamic, one line per partner | Debit, Credit, Balance | Journal Items, then the entry |
| Aged Receivable | Partner Reports | dynamic, one line per partner | Not due, 1–30, 31–60, 61–90, 91–120, Older, Total | open items, then the invoice |
| Aged Payable | Partner Reports | dynamic, one line per partner | the same six buckets and total | open items, then the bill |
| Journal Audit | Audit Reports | dynamic, one section per journal | depends on the journal type | entries, then items |
| Generic Tax report | Taxes & Fiscal | dynamic, grouped by tax type then tax | Net, Tax | base items and tax items |
| Group by: Account > Tax | Taxes & Fiscal, variant selector | dynamic | Net, Tax | the same |
| Group by: Tax > Account | Taxes & Fiscal, variant selector | dynamic | Net, Tax | the same |
| The one hundred and sixty-two national returns | Taxes & Fiscal, variant selector | static | per definition | per expression |
| Audit Trail | Audit Reports | list of messages | — | the changed record |
| Data Inalterability Check | Audit Reports | printable document only | — | — |
| Invoice Analysis | Management | analysis entity | per the Accounts Receivable domain | the invoice |
| Analytic Report | Management | analysis entity | per the Analytic Accounting domain | the analytic line |

---

## 10. Behavior contracts a client must honour

1. **The option set is authoritative.** A client sends options and receives figures; it never
   computes a figure itself, not even a total. Two clients sending the same options must receive
   identical figures.
2. **Row identifiers are opaque and stable within a rendering.** A client uses them to request
   expansions and audits; it must not parse them or reuse them across renderings with different
   options.
3. **Expansion never changes a parent.** A client that receives sub-lines must insert them
   without recomputing the parent's cells.
4. **Formatting is server-side.** Every cell carries both a raw value and a formatted value. A
   client displays the formatted value and uses the raw value only for sorting and for charting.
5. **Editability is declared per cell.** A client must not offer an input on a cell whose
   editable flag is false, and must expect a refusal even when the flag is true, because the lock
   date is checked at write time.
6. **Auditability is declared per cell.** A client must not offer the audit action on a cell
   whose auditable flag is false.
