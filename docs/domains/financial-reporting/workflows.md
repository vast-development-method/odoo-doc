# Financial Reporting — Workflows

This file specifies the end-to-end operational workflows of the domain, step by step: who performs
them, what must be true before they start, what happens at each step, and which records are
created or updated.

Roles referred to throughout:

| Role | Group | What it may do here |
|---|---|---|
| Billing user | invoicing group | Open the reports the invoicing menus expose; no report designer, no closing. |
| Accountant, read-only | read-only accounting group | Open every report, unfold, drill down, compare, export. Cannot edit a manual figure, cannot close a period, cannot change a definition. |
| Accountant | accounting user group | Everything the read-only accountant may do, plus run the hash integrity check. |
| Accounting manager | accounting manager group | Everything above, plus create and change report definitions, edit manual figures, validate tax returns, move lock dates and grant lock date exceptions. |

---

## 1. Authoring a report definition

### 1.1 Creating a root report

*Performed by:* an accounting manager, with the designer interface enabled.

*Preconditions:* none.

1. Create the Report Definition: give it a name and, when it implements a national obligation, a
   country and the availability condition `country`.
2. Add at least one column. A column needs a name and an expression label; its display type
   defaults to `monetary`.
   *Record created:* one Report Column per column.
3. Add the lines. For each line: a name, a sequence, an optional code, and a parent when the line
   belongs under another.
   *Record created:* one Report Line per line. The line's level is derived from its parent; its
   report is derived from its parent when it has one.
4. Add the expressions of each line. For each: a label, an engine, a formula, an optional
   subformula and a date scope.
   *Record created:* one Report Expression per expression. For a tax tag expression, the tag is
   created in the report's country if it does not already exist.
   *Validation:* the formula is checked against the engine's grammar
   ([`entities.md`](entities.md) §3.9); the label must be unique on the line; a record filter
   expression must carry a subformula.
5. Create a menu entry for the report so that users can reach it.
   *Record created:* one window action of the report kind, carrying the report's identifier, and
   one menu item under the reporting menu.

*Postcondition:* the report appears in the reporting menu for every company satisfying its
availability condition.

*Failure conditions:* the availability condition is `country` but no country is set; a line
appears before its parent in the sequence order; a formula is invalid. Each raises the
corresponding message of [`business-rules.md`](business-rules.md).

### 1.2 Creating a variant

*Performed by:* an accounting manager.

*Preconditions:* the root report exists and is itself a root — it has no root report of its own.

1. Create a Report Definition and set its root report to the generic report.
2. Set its country. The availability condition becomes `country` automatically.
3. The filter switches are copied from the root report at creation, as **defaults**; later
   changes to the root do not propagate.
4. Add the variant's own columns, lines and expressions. A variant does not inherit the root's
   lines: it is a complete report of its own.

*Postcondition:* opening the root report from its menu entry shows a variant selector listing the
root and all of its variants; the variant whose country matches the company's fiscal country is
pre-selected.

*Failure:* choosing as root a report that already has a root raises *Only a report without a root
report of its own can be selected as root report.*

### 1.3 Assembling a composite report

*Performed by:* an accounting manager.

*Preconditions:* the reports to be used as sections exist and have no sections of their own.

1. Create or open the composite Report Definition.
2. Add the section reports, in the order they must appear.
   *Record updated:* rows in the section association table; the composite flag becomes true.
3. Each section inherits the composite report's filter switches, **provided** the section is not
   also reachable through its own menu entry and is not a section of any other composite report.

*Postcondition:* opening the composite report renders each section in order, each under a heading
carrying the section's name; printing produces one document containing all sections.

*Failure:* adding a report that itself has sections raises *The sections defined on a report
cannot have sections themselves.*

### 1.4 Duplicating a report to customize it

*Performed by:* an accounting manager.

1. Duplicate the report. The copy is named after the original with a copy marker appended,
   repeated until the name is unique.
2. Every line is copied recursively; each copied line's code becomes the original code with
   `_COPY` appended, repeated until unique.
3. Every aggregation formula and subformula in the copy is rewritten so that references to the
   original codes become references to the copied codes.
4. Every column is copied.
5. **No tag is created or removed** by duplication: the copied tax tag expressions match the same
   tags as the originals, because both reports are in the same country.

*Postcondition:* the copy renders exactly the same figures as the original.

### 1.5 Changing the country of a report

*Performed by:* an accounting manager. This is the workflow a country uses when it adopts another
country's report as its starting point.

1. Duplicate the source report (§1.4).
2. Set the copy's country to the new country.
3. For each tax tag expression of the copy, the tag housekeeping of
   [`entities.md`](entities.md) §1.10 runs: because the original still uses the tags in the old
   country, new tags with the same names are created in the new country.
4. Configure the new country's taxes to stamp the new tags.

*Postcondition:* the two reports have the same shape, the same tag names and different tags, one
set per country. Neither country's figures affect the other.

*Worked example:* a report with seven tax tag lines is duplicated and its copy is moved to
another country. Seven tags are created in the new country, with the same seven names. The
original's tags are untouched. If instead the **original** report's country is changed and some
of its tags are shared with a report that is not being changed, those shared tags stay where they
are and new ones are created; tags used only by the changed report are moved.

---

## 2. Opening and rendering a report

*Performed by:* any role that can reach the report's menu entry.

*Preconditions:* the report is available for the active company.

1. **Resolve the options.** The date filter takes the report's default opening date filter; the
   company filter takes the active companies or their tax unit; every other filter takes its
   stored default. A user's last-used options for the same report are restored when they exist.
2. **Resolve the variant.** When the report is a root report with variants, the variant whose
   country matches the company's fiscal country is selected; otherwise the root itself.
3. **Evaluate** with the pipeline of [`calculations.md`](calculations.md) §2.
4. **Render** the header (the report's name, the company or tax unit, the period, the
   presentation currency and the rounding unit) and the table.

*Postcondition:* a table of rows and columns is displayed; no record is created or changed.

### 2.1 Changing a filter

Every filter change re-runs steps 3 and 4 with the new options. The set of unfolded lines is
preserved across a filter change where the lines still exist, and discarded where they do not (a
partner line disappears when the partner filter excludes that partner).

Changing the **company** filter can change the presentation currency, which re-renders every
figure; changing the **date** filter re-evaluates everything; changing the **rounding unit** or
the **hide-zero** option re-renders without re-evaluating.

### 2.2 Saving a set of options

A user may save the current option set under a name, so that the report reopens with it. The
saved set is per user and per report and stores only the options, never figures.

---

## 3. Unfolding and drilling down

*Performed by:* any role that can open the report.

### 3.1 Unfolding a line

1. The user activates the folding control of a line in state `folded`.
2. The line's identifier joins the user's set of unfolded lines.
3. Sub-lines are produced by the rule of [`calculations.md`](calculations.md) §11, subject to the
   load-more limit and the prefix-group threshold.
4. The sub-lines are inserted under the line.

*Postcondition:* the parent's figures are unchanged; the sub-lines' figures sum to the parent's,
up to rounding, for every grouping and account expansion.

### 3.2 Auditing a figure

1. The user activates a figure whose expression is auditable.
2. The drill-down restriction is assembled: the base restriction of
   [`calculations.md`](calculations.md) §2.3, the expression's date window, and the engine's own
   selection (§§3.5, 4.7, 6.6, 7.6 of that file).
3. A list of Journal Items is opened with that restriction, grouped by account, showing date,
   entry number, partner, label, debit, credit and the running balance.
4. When the column declares a custom audit action, that action is opened instead, receiving the
   same restriction.

*Postcondition:* the sum of the listed items equals the audited figure, up to the report's
rounding, for every engine except the aggregation engine when its formula multiplies or divides,
and except the custom function engine.

*Failure:* a figure whose expression is not auditable offers no audit action.

### 3.3 Opening the underlying document

From an item row, the user opens the Journal Entry; from there, the invoice, bill, payment or
statement that produced it. This is an ordinary navigation and belongs to the other domains.

### 3.4 Annotating a line

*Performed by:* an accountant or above.

1. The user activates the annotation control of a line.
2. A free text is stored, keyed by the report, the line and the period's end date.
3. The annotation is displayed next to the line whenever the same report is opened on a period
   whose end date matches, and it is printed on the printable document as a numbered footnote.

*Record created:* one annotation record. No Journal Entry.

---

## 4. Comparing periods

*Performed by:* any role that can open the report.

*Preconditions:* the report's period comparison switch is on.

1. The user chooses a comparison mode: previous periods, same period last year, or a custom
   period, and a number of comparison periods for the first two.
2. The comparison periods are computed by the shifting rule of
   [`calculations.md`](calculations.md) §10.2.
3. The whole pipeline runs once per period; the column set is repeated once per period, current
   first.
4. When growth comparison is on and exactly one comparison period is selected, a growth column is
   appended and filled by the rule of [`calculations.md`](calculations.md) §10.4.

*Postcondition:* every figure of every period is computed from the ledger as it stands now, not
from a stored snapshot. Reopening the same comparison after new entries are posted in a past
period will show different figures for that past period; this is intended and is why closed
periods are locked.

---

## 5. Exporting

### 5.1 Printable document

1. The user requests the printable document.
2. The report is evaluated with the current options and the currently unfolded lines.
3. The document is laid out with the report's header, the column headers repeated on every page,
   one row per rendered line indented by its level, and page breaks before every line whose
   print-on-new-page flag is set.
4. Annotations are printed as numbered footnotes.
5. The horizontal split, when the report supports it and the option is on, prints the left-side
   subtree and the right-side subtree side by side.

*Record created:* none by default. When the user chooses to file the document, an attachment is
created on the record the user names.

### 5.2 Spreadsheet workbook

1. The report is evaluated as above.
2. One worksheet is produced per section for a composite report, or a single worksheet otherwise.
3. Row one carries the report's name, the company and the period; row two carries the column
   headers; the data rows follow, indented by leading spaces proportional to the level.
4. Monetary cells carry numbers, not text, with a number format matching the presentation
   currency and the rounding unit. Percentage cells carry a fraction with a percentage format.
5. Unfolded lines are exported expanded; folded lines are exported collapsed.

### 5.3 Electronic filing file

*Performed by:* an accounting manager, as part of a tax return.

1. The report is evaluated for the return's period and company or tax unit.
2. The country's filing format maps each declared box to a field of the legal file, by the line's
   **code**, not by its name or its position.
3. The file is produced and attached to the return.
4. Where the authority offers a direct channel, the file is transmitted and the acknowledgement
   reference is stored on the return.

*Failure:* a box the format requires whose line code is missing from the definition aborts the
generation with a message naming the missing code.

---

## 6. The tax return workflow

This is the only workflow of the domain that writes to the ledger.

### 6.1 Preconditions

1. The company has a tax return periodicity and a tax return journal.
2. Every tax that moved in the period belongs to a tax group, and that group has a tax payable
   account and a tax receivable account.
3. The report to be filed is available for the company's country.

### 6.2 Step 0 — the return appears

*Trigger:* a return period elapses, or an accounting manager creates a return by choosing a type
and a period.

*Record created:* one tax return in state "to review", plus one tax check record per check
declared for the country and the type, each in state "to review".

The return is listed on the accounting dashboard against the tax return journal, with its period,
its deadline and its three step markers.

### 6.3 Step 1 — review

*Performed by:* an accounting manager.

1. The user opens the return. The checks run and each moves to "passed" or "anomaly".
2. For each check in "anomaly", the user either fixes the underlying data — posting draft
   entries, attaching missing documents, completing the company data, correcting a tax's country
   — after which the check re-runs, or waives it as "reviewed" or "supervised".
   *Record updated:* the check's state, plus a note in the return's message log naming the user
   and the time.
3. The user opens the rendered report to verify the figures and, where the law allows, types
   manual adjustments into the editable boxes (§7).
4. The user activates **Validate**.

*Guards:* no check may remain in "anomaly"; see [`state-machines.md`](state-machines.md) §4.5.

*What Validate does, in order:*

1. Re-evaluate the report for the period and the company or tax unit.
2. Compute the carry-over amounts and write them
   ([`calculations.md`](calculations.md) §12.4).
   *Records created:* one Report External Value per carrying expression with a non-zero amount.
3. Compute the closing amounts ([`calculations.md`](calculations.md) §16).
4. Create the closing Journal Entry and post it
   ([`accounting-effects.md`](accounting-effects.md) §2).
   *Records created:* one Journal Entry with its items, per company.
5. Move the company's tax return lock date to the last day of the period, unless it is already
   later.
   *Record updated:* the company; the change is tracked.
6. Attach the rendered report as a printable document and as a spreadsheet workbook to the
   return.
7. Move the return to "reviewed".

*Failure conditions:* a missing tax return journal, a missing payable or receivable account, a
period ending in the future without confirmation. Each aborts the whole step — no carry-over is
written and no entry is created.

### 6.4 Step 2 — submit

*Performed by:* an accounting manager.

1. The user reviews the rendered report one last time.
2. The user activates **Submit**.
3. Where the country package provides a filing format, the file is generated and, where a direct
   channel exists, transmitted; otherwise the user is shown the manual filing instructions and
   marks the return as submitted.
4. The return moves to "submitted".

### 6.5 Step 3 — pay

*Performed by:* an accounting manager or a payment user.

1. The amount to pay, or the amount to be refunded, is shown together with the payment details
   the country package supplies, including a payment barcode where the country defines one.
2. The user either pays through the ordinary payment flow and reconciles the payment against the
   counterpart line of the closing entry, or marks the return as paid.
3. The return moves to "paid" and leaves the list of pending returns.

### 6.6 Undoing

| Action | Effect |
|---|---|
| Cancel the return | The closing entry is reversed by a new posted entry; the carry-over values written by the validation are deleted; the return moves to "cancelled". The lock date is **not** moved back. |
| Reset the return to draft | The closing entry returns to draft, which requires both a non-restricted journal and a lock date exception; the carry-over values are deleted; the return returns to "to review". |
| Move the lock date back | A separate act, performed on the company by a user with the lock date group; tracked. |

---

## 7. Editing a manual figure

*Performed by:* an accounting manager.

*Preconditions:* the cell's expression uses the external engine and its subformula contains
`editable`; the report's *date to* is after every applicable lock date.

1. The user activates the cell and types an amount.
2. The amount is rounded by the `rounding=` clause when present.
3. A Report External Value is created, or the existing manual value for the same expression, date
   and company is overwritten.
   *Record created or updated:* one Report External Value, with an empty origin line.
4. The report re-renders; every aggregation that depends on the cell is recomputed.

*Failure:* the date falls on or before an applicable lock date, and the write is refused with the
lock date message of [`business-rules.md`](business-rules.md) §7.2.

*Clearing:* emptying the cell deletes the manual value.

---

## 8. Closing a period with a carry-over

*Performed by:* an accounting manager, as part of §6.3.

*Preconditions:* the report has at least one line carrying the three-expression carry-over
pattern of [`calculations.md`](calculations.md) §12.2.

1. The report is evaluated for the period.
2. For each expression labelled `_carryover_`*x*:
   a. Its formula and bound clause are evaluated, giving the amount to carry out.
   b. If the amount is zero, nothing is written.
   c. Otherwise the target expression is resolved: the explicit carry-over target when set,
      otherwise the expression labelled `_applied_carryover_`*x* on the same line. A target that
      cannot be resolved aborts the closing with *Could not determine carryover target
      automatically for expression the label.*
   d. A Report External Value is created with the target expression, the last day of the period,
      the company, the amount, the origin line and the origin expression label. An existing value
      for the same four keys is overwritten.
3. The declared figure of the line is the floored value, so the declaration shows zero rather
   than a negative amount.

*Postcondition:* the next period's rendering picks the carried amount up through its
`_applied_carryover_`*x* expression.

*Idempotence:* re-validating the same period overwrites the same records; it never doubles the
carried amount.

---

## 9. Running the hash integrity check

*Performed by:* an accountant or above.

*Preconditions:* the user holds the accounting user group. Otherwise the request is refused with
*Please contact your accountant to print the Hash integrity result.*

1. The user activates the integrity check from the accounting menu.
2. The algorithm of [`calculations.md`](calculations.md) §17.2 runs over every journal of the
   company.
3. A printable document is produced listing, per journal and per sequence prefix, the restricted
   mode flag, the status, the message, and — for a verified prefix — the first and last entry
   with their numbers, dates and hashes.
4. The printing date is stamped on the document.

*Record created:* none; the document is produced on demand and is not stored unless the user
files it.

---

## 10. Reading the audit trail

*Performed by:* an accountant or above.

1. The user opens the audit trail from the review menu.
2. The list shows tracked-change messages with their date and time, their author, the record they
   concern, the type of change, and one line per changed field with its old and new values.
3. The list can be filtered by Journal Entry, partner, account, tax, company and by free text
   over the old and new values.
4. A message on a record protected by the restrictive audit trail is marked as protected and
   cannot be deleted or altered.

*Record created:* none.

---

## 11. Year-end interaction

The domain participates in the year-end close but does not own it. The sequence, in the order an
accountant performs it:

1. Reconcile every bank account up to the year end and confirm the closing book balances against
   the statements; the bank reconciliation report supports this.
2. Confirm or cancel every draft invoice, bill and entry; the draft-entries check of a tax return
   and the draft filter of every report support this.
3. Run the tax report for the last period of the year and close it (§6).
4. Reconcile every balance-sheet account: run the aged receivable and aged payable reports at the
   year end and explain every open item.
5. Book the year-end adjustments — depreciation, accruals, provisions, deferred revenue — as
   ordinary entries.
6. Run the balance sheet and the profit and loss at the year end and verify that the current-year
   earnings line of the balance sheet equals the net result of the profit and loss.
7. Set the global lock date to the last day of the closed fiscal year.
8. Where the country requires an explicit allocation, post the entry that moves the current-year
   earnings to retained earnings. Until that entry is posted, the balance sheet shows the result
   on the current-year earnings line; after it, on retained earnings. Both presentations balance.

The fiscal year itself is defined by the company's fiscal-year last day and last month; a company
whose fiscal period is longer or shorter than twelve months declares explicit fiscal year records
with a name, a start date and an end date, and the reports use those boundaries for every
fiscal-year-relative date scope. Once a declared fiscal period is over, the default periodicity
resumes.

---

## 12. Consolidating several companies

*Performed by:* an accountant with access to more than one company.

1. Select the companies in the company filter. The report's multi-company switch must be
   `selector`; when it is `tax_units`, only the companies of a tax unit can be selected together.
2. Every engine reads the Journal Items of all selected companies.
3. Figures in other currencies are translated into the presentation currency, which is the
   currency of the first selected company, by the mode of
   [`calculations.md`](calculations.md) §13.2.
4. External values of all selected companies are summed.
5. Account lines are merged **by account code**: two companies' accounts with the same code
   appear as one line. Accounts whose codes differ appear as separate lines; the account code
   mapping facility of the chart of accounts lets a company declare a second code so that it
   merges with its sibling's.

*Postcondition:* the consolidated balance sheet balances, because each company's entries balance
and the translation mode preserves that property.

*Limitation:* intercompany balances are **not** eliminated by the report. A consolidated balance
sheet shows both sides of an intercompany loan. Eliminating them requires either eliminating
entries in the ledger or a report definition with explicit elimination lines.
