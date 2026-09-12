# Analytic Accounting — Interfaces

Everything through which a person, a client or an integration reaches this domain: the menus, the
window actions and their stable paths, every screen with the fields it shows and the filters it
offers, the distribution editor in full, the named operations with their inputs and outputs, the
notifications and dialogues, the import and export behaviour, and the statement of what the domain
deliberately does not publish.

Contents:

1. [Navigation](#1-navigation)
2. [Window actions and stable paths](#2-window-actions-and-stable-paths)
3. [The plan screens](#3-the-plan-screens)
4. [The analytic account screens](#4-the-analytic-account-screens)
5. [The analytic item screens](#5-the-analytic-item-screens)
6. [The distribution model screens](#6-the-distribution-model-screens)
7. [Empty-state texts](#7-empty-state-texts)
8. [The distribution editor](#8-the-distribution-editor)
9. [Named operations](#9-named-operations)
10. [Dialogues and notifications](#10-dialogues-and-notifications)
11. [The analytic columns other screens gain](#11-the-analytic-columns-other-screens-gain)
12. [The project profitability entries](#12-the-project-profitability-entries)
13. [Reports and printable documents](#13-reports-and-printable-documents)
14. [Import and export](#14-import-and-export)
15. [Request endpoints and external integrations](#15-request-endpoints-and-external-integrations)
16. [Client behaviour a replacement must reproduce](#16-client-behaviour-a-replacement-must-reproduce)
17. [Reconciliation notes](#17-reconciliation-notes)

---

## 1. Navigation

Every entry below is inside the accounting application, whose top-level menu is "Invoicing", and
every one of them is visible only to a member of the analytic accounting permission group.

| Menu path | Order inside its parent | Opens |
|---|---|---|
| Accounting → Transactions → Analytic Items | 31, after Journal Entries | The analytic item screen |
| Configuration → Analytic Accounting | 40, the last entry of Configuration | A submenu, itself restricted to the group |
| Configuration → Analytic Accounting → Analytic Distribution Models | 10 | The distribution model screen |
| Configuration → Analytic Accounting → Analytic Accounts | 20 | The analytic account screen |
| Configuration → Analytic Accounting → Analytic Plans | 30 | The plan screen, restricted to root plans |
| Reporting → Management → Analytic Report | after Invoice Analysis | The analytic reporting screen, contributed by the general ledger capability over this domain's Analytic Line; visible to the read-only accounting group |

The Configuration menu itself is visible only to an accounting administrator, and the Accounting
menu only to the read-only accounting group, so a member of the analytic accounting group who holds
no accounting group sees the analytic screens only where another domain embeds them.

---

## 2. Window actions and stable paths

| Action name | Entity | Presentations | Stable path | Default filters, grouping and defaults |
|---|---|---|---|---|
| Analytic Plans | Analytic Plan | list, form | `analytic-plans` | Only plans with no parent. |
| Analytic Accounts | Analytic Account | list, cards, form | `analytic-accounts` | Archived accounts hidden. |
| Chart of Analytic Accounts | Analytic Account | list, cards, form | none | Archived accounts hidden. The same content as the previous action, opened with the list presentation forced first. |
| Analytic Items | Analytic Line | list, cards, form, graph, pivot | `analytic-items` | With the accounting capability installed: the last-fiscal-year filter and the profit-and-loss-accounts filter are switched on. |
| Analytic Distribution Models | Analytic Distribution Model | list, form | `analytic-distribution-models` | None; the list is ordered by sequence ascending then internal identifier descending. |
| Gross Margin | Analytic Line | list, form, graph, pivot | none | Opened from one analytic account: restricted to the analytic lines whose magic column resolves to that account, grouped by date, with that account pre-filled as the default of the magic column for creation. |
| Analytical Accounts | Analytic Account | list, form | none | Opened from one plan: the accounts whose plan is that plan or any descendant of it, with that plan pre-filled for creation. |
| Analytical Plans | Analytic Plan | list, form | none | Opened from one plan: its direct children, with the parent and the colour index pre-filled for creation. |
| Customer Invoices | Journal Entry | list, form | none | Opened from one analytic account: the sale documents having a journal item whose distribution names that account; creation disabled; the default document type is a customer invoice. |
| Vendor Bills | Journal Entry | list, form | none | Opened from one analytic account: the purchase documents, receipts included, having such a journal item; creation disabled; the default document type is a vendor bill. |
| Analytic Reporting | Analytic Line | list, cards, form, graph, pivot | `analytic-report` | The last-fiscal-year filter and the profit-and-loss-accounts filter switched on. Contributed by the general ledger capability. |

**Compatibility finding.** The Analytic Items action and the Analytic Reporting action both ask for
a default grouping named *group by analytic account*, but no filter of that name exists in the
search definition — the grouping filter on the base plan column is named after the column. The
requested default therefore activates nothing, and both screens open ungrouped. A corrected
behaviour would name the existing filter, so that both screens open grouped by the base plan's
analytic account. A rebuild that groups them by default is more useful and is no longer
bug-compatible on that detail.

---

## 3. The plan screens

### 3.1 The plan list

An editable list, several rows at a time, showing: a drag handle bound to `sequence` (sequence);
`name` (name); `default_applicability` (default applicability), required; `color` (colour index),
shown as a colour picker. Only plans with no parent are listed, so the list is the list of axes.

### 3.2 The plan form

| Element | Content and guard |
|---|---|
| Button box | **Subplans**, showing `children_count` (children plans count), which opens the direct children. **Analytic Accounts**, showing `all_account_count` (all analytic accounts count), which opens the accounts of the plan and of all its descendants. |
| Title | `name`. |
| Left group | `parent_id` (parent); `default_applicability`, required and hidden as soon as a parent is set; `color`, as a colour picker. |
| Applicability page | Titled "Applicability", hidden as soon as a parent is set. An editable list of the plan's applicability rules. |

The columns of the applicability list, in order: `business_domain` (business domain);
`display_account_prefix` (show the prefix field) and `account_prefix_placeholder` (prefix
placeholder), both invisible and present only to drive the next cell; `account_prefix` (financial
accounts prefixes), shown only when the flag is true and displaying its computed placeholder when
empty; `product_categ_id` (product category); `company_id` (company), shown only in a multi-company
database with the placeholder "Visible to all"; `applicability` (applicability). The three
criterion cells contributed by the general ledger capability are absent when that capability is not
installed.

Setting a parent on the form of the base plan is refused immediately, before anything is written,
with "You cannot add a parent to the base plan 'the plan name'" (`AA-003`).

---

## 4. The analytic account screens

### 4.1 The account list

An editable list, several rows at a time. Columns in order: `name` (analytic account), labelled
"Name"; `code` (reference); `partner_id` (customer); `plan_id` (plan); `company_id`, only in a
multi-company database; `debit` (debit) with a column total labelled "Debit"; `credit` (credit)
with a column total labelled "Credit"; `balance` (balance) with a column total labelled "Balance".
The company, the currency and the active flag are carried as hidden columns because the rows need
them. The visibility of the three totals is specified in
[configuration.md](configuration.md) section 8.

A second variant of the same list exists for use as a value picker; it differs only in that editing
several rows at once is switched off.

### 4.2 The account form

| Element | Content and guard |
|---|---|
| Button box | **Gross Margin**, showing `balance` as a monetary value, which opens the analytic items of the account grouped by date. **Customer Invoices**, showing `invoice_count` (invoice count), hidden when the count is zero. **Vendor Bills**, showing `vendor_bill_count` (vendor bill count), hidden when the count is zero. The last two are contributed by the general ledger capability. |
| Ribbon | "Archived", shown in red when `active` (active) is false. |
| Title | `name`, with the placeholder "e.g. Project XYZ" — reproduced. |
| Left group | `partner_id`; `code`. |
| Right group | `plan_id`, with quick creation disabled; `company_id`, only in a multi-company database, with the placeholder "Visible to all" and creation disabled; `currency_id` (currency), only for a reader with the multi-currency capability, creation disabled. |
| Discussion thread | Present at the bottom; it records every change of the name, the reference, the archival flag and the customer. |

### 4.3 The account cards

One card per account, showing the display name in bold and, under a separating rule, the text
"Balance: " followed by `balance` as a monetary value.

### 4.4 The account search panel

| Element | Behaviour |
|---|---|
| Text search labelled "Analytic Account" | Matches `name` **or** `code`, case-insensitive, on a fragment. |
| Text search on the customer | Matches `partner_id`. |
| Filter "Archived" | Keeps the accounts whose active flag is false. |
| Grouping "Associated Partner" | Groups by `partner_id`. |

The two account actions of section 2 switch the archived filter's complement on by default, so an
archived account appears only when the reader asks for it.

---

## 5. The analytic item screens

### 5.1 The analytic item list

An editable list, several rows at a time, which is how a distribution is applied to many analytic
lines in one action. Columns in order, with their default visibility:

| Column | Identifier | Shown by default | Note |
|---|---|---|---|
| Date | `date` | yes | |
| Description | `name` | yes | |
| The base plan's account | `account_id` | yes | Relabelled at run time with the base plan's name |
| Reference | `ref` | no | Added by the general ledger capability; hidden outright in the to-invoice context |
| Financial account | `general_account_id` | no | Added by the general ledger capability |
| Journal item | `move_line_id` | no | Added by the general ledger capability; rendered as a link that opens the entry |
| Product | `product_id` | no | Added by the general ledger capability |
| One column per other root plan | `x_plan<plan identifier>_id` | yes | Inserted at read time, immediately after the base plan's column, in reverse plan order so that the final order is the plan order |
| Analytic distribution | `analytic_distribution` | no | Rendered by the distribution editor in multiple-record mode; writing it **splits** the lines |
| Quantity | `unit_amount` | no | With a column total labelled "Quantity" |
| Unit | `product_uom_id` | no | |
| Partner | `partner_id` | no | |
| Company | `company_id` | yes | Only in a multi-company database |
| Amount | `amount` | yes | With a column total labelled "Total" |

The currency and the company are also carried as hidden columns, because the monetary rendering
needs them.

### 5.2 The analytic item form

Left group titled "Analytic Item": `name`; the base plan's column and, inserted after it, one
column per other root plan; `date`; `company_id`, only in a multi-company database. Right group
titled "Amount": `amount`; `ref` and `partner_id`, both added by the general ledger capability;
`unit_amount`; `product_id`, added by the general ledger capability; `product_uom_id`. A third
group titled "Accounting", added by the general ledger capability, carries `move_line_id`, with
creation disabled and rendered as a link that opens the entry, and `general_account_id`, read-only
as soon as a journal item is set.

### 5.3 The analytic item cards

One card per line: the description in bold and the date on the first row; the base plan's account
and the amount as a monetary value on the second.

### 5.4 The graph and pivot presentations

The graph measures `unit_amount`, rendered as a duration, and `amount`, and groups by the base
plan's column. The pivot measures `amount`, groups rows by the base plan's column and columns by
`date` by month; the general ledger capability adds the partner as a hidden row grouping.

### 5.5 The analytic item search panel

| Element | Behaviour |
|---|---|
| Text search on the description | Case-insensitive fragment match. |
| Search on the date | Date search. |
| Search on the magic column, the base plan's account, the product, the financial account and the partner | Added by the general ledger capability; the partner search matches the partner and all its children. |
| Filter "Date" | A period filter on `date`. |
| Filter "From last fiscal year" | Hidden from the panel and used only as a default: it applies the fiscal-year search helper, which resolves to "the date is on or after the first day of the fiscal year containing today, minus one year" (`AA-075`). |
| Filter "P&L Accounts" | Keeps the lines whose financial account belongs to the income or the expense group. Its label is reproduced as displayed. |
| Grouping "Date" | Groups by `date`. |
| Groupings added by the general ledger capability | By the base plan's account, by financial account, by category, by product and by partner. |
| Groupings inserted at read time | One per other root plan, and one per existing sub-plan depth of each root plan. |

**Folding of the plan groupings.** When one root plan has more than one sub-plan depth, the several
grouping entries that belong to that plan's column are folded into a **single** entry whose options
are the depths, and choosing an option selects that depth. A grouping entry naming a plan column or
a grouping column that no longer exists is removed from the panel, which keeps a stored personal
filter from referring to a deleted plan. The same folding applies in the list, the cards, the graph
and the pivot, and the pivot's own grouping menu resolves an option to the depth it names.

---

## 6. The distribution model screens

### 6.1 The distribution model list

An editable list, several rows at a time, ordered by `sequence` ascending then internal identifier
descending, with the row form reachable from the row. Columns: a drag handle bound to `sequence`;
`account_prefix` (accounts prefix), labelled "Accounts Prefixes" and displaying its computed
placeholder when empty; `partner_id` (partner); `partner_category_id` (partner category), hidden by
default; `product_id` (product); `product_categ_id` (product category), hidden by default;
`company_id`, only in a multi-company database, with the placeholder "Visible to all";
`analytic_distribution`, rendered by the distribution editor with the applicability forced to
`optional` and the new-model shortcut disabled. The prefix, the product and the product category
columns are contributed by the general ledger capability.

### 6.2 The distribution model form

Group titled "Conditions to meet": `partner_id`, `partner_category_id`, `account_prefix`,
`product_id`, `product_categ_id`, `company_id` — the last only in a multi-company database, with
the placeholder "Visible to all". Group titled "Distribution to apply": `analytic_distribution`,
rendered by the distribution editor with the applicability forced to `optional` and the new-model
shortcut disabled.

Forcing the applicability is what makes **every** root plan with at least one account visible in
that editor, whatever the applicability rules say (`AA-030`): a model must be able to propose an
account of a plan that is unavailable in the situation where the model is written.

---

## 7. Empty-state texts

| Screen | Text shown when nothing matches |
|---|---|
| Analytic Plans | "Click to add a new analytic account plan." |
| Analytic Accounts and Chart of Analytic Accounts | "Add a new analytic account" |
| Gross Margin | "No activity yet on this account", followed by three explanatory paragraphs: that sales orders and projects are implemented with analytic accounts and that costs and revenues can therefore be tracked to analyse margins; that costs are created automatically when supplier invoices, expenses or timesheets are recorded; and that revenues are created automatically when customer invoices are created, whether from sales orders at a fixed price, from timesheets on the work done, or from re-invoiced expenses. |
| Analytic Items | "No activity yet", followed by the same three paragraphs. |
| The distribution editor with no relevant plan | "No analytic plans found" |

The first two texts and the two headline texts are reproduced as the system displays them; the
explanatory paragraphs are restated here because they name the product and the specification's own
prose does not.

---

## 8. The distribution editor

The distribution editor is the single component through which a distribution is entered anywhere in
the system. Every screen that shows a distribution — a journal item, a sales order line, a purchase
order line, an expense, a reconciliation model line, an asset, a work centre, a distribution model,
an analytic line — uses this one component.

### 8.1 How a host field configures it

| Setting | Meaning |
|---|---|
| Business domain, constant | The business domain to pass when asking for the relevant plans. |
| Business domain, computed | An expression evaluated against the host record, used when the business domain depends on the document — for a journal item it yields `invoice`, `bill` or `general`. |
| Account field | The name of the host field holding the financial account, whose value is passed as the account of the situation and whose display name seeds the prefix of a new model. |
| Product field | The name of the host field holding the product, passed as the product of the situation. |
| Amount field | The name of the host field holding the amount; when it is set and the amount is not zero, a monetary column is shown. |
| Forced applicability | Makes every plan visible with that applicability and stops any rule from being evaluated. |
| Save-as-model shortcut | Enabled by default; the two distribution model screens disable it. |
| Multiple-record mode | Enables the per-plan update ticks and the partial-update marker. |
| Placeholder | The text shown in the closed cell when the distribution is empty. |

### 8.2 The closed cell

The cell shows one tag per root plan present in the distribution. The text of a tag lists the
accounts of that plan separated by " | ", each preceded by its percentage **unless** that account's
total for the plan is exactly one hundred percent, in which case only the account's display name is
shown. The tag takes the plan's colour index. Two worked examples:

| Distribution | Tags shown |
|---|---|
| Account A1 of plan A at 100, account B1 of plan B at 80.123, account C1 of plan C at 100 | "A1", "80.12% B1", "C1" |
| (A1, B1, C1) at 50, (A2, B1, C1) at 50, (A3, B1, C2) at 50 | "50% A1 \| 50% A2 \| 50% A3", "150% B1", "C1 \| 50% C2" |

Clicking a tag opens the editor. When the reader may not write the record, the tags are shown and
nothing opens.

### 8.3 The open editor

A dropdown carrying a header, a table and an add-row control.

- The header shows the word "Analytic"; the shortcut "New Model", whose hover text is "Save as new
  analytic distribution model", shown only when the shortcut is enabled and at least one row is
  valid; and a close control whose hover text is "Close".
- The table has one column per relevant plan, in the order the relevant-plans answer returns, then
  a column headed "Percentage", then, when a monetary column is enabled, a column headed with the
  host amount field's own label, then a narrow column carrying the delete control of each row.
- Each plan column's header shows the plan's name and, in parentheses, that plan's running total as
  a percentage: in red when the plan is mandatory and the total is not exactly one hundred percent,
  in green when the plan is mandatory and the total is exactly one hundred percent, and unstyled
  otherwise. In multiple-record mode the total is shown only for the plans being updated.
- In multiple-record mode each plan header additionally carries a link reading "Update" or "Don't
  update", which toggles that plan's participation in the partial-update marker. A plan that is not
  being updated has its cells blanked out in every row.
- The add-row control reads "Add a Line".
- When the relevant-plans answer is empty the table is replaced by the text "No analytic plans
  found".

### 8.4 Operations

| Operation | Behaviour |
|---|---|
| Open | Asks for the relevant plans with the host line's business domain, product, financial account, company and the accounts already named in the distribution; builds one row per entry of the stored document; adds an empty row when there is none; puts the focus in the first cell of the first row. In multiple-record mode the grid starts **empty** rather than showing the current values, and the current values are kept aside to render the tags. |
| Add a row | Computes the proposed percentage: among the plans whose running total is below one hundred percent, take the largest total of the **mandatory** plans when at least one mandatory plan is incomplete, otherwise the largest total of the optional plans, otherwise the total of the rows that name no account at all; the proposal is one hundred percent minus that value, floored at zero, and becomes one hundred percent when the result is zero. |
| Pick an account | Only accounts whose root plan is that column's plan are offered, and, when the host record has a company, only accounts with no company or with a company that is an ancestor of the host's company. Creating an account from the cell, opening it and quick-creating it are all disabled. |
| Type a percentage | Held and compared with the percentage precision plus **two** working digits (`AA-107`). |
| Type a monetary value | Recomputes the row's percentage as the typed value divided by the host record's amount, at the same working precision. |
| Delete a row | Removes it; when the last row is removed, an empty row is added immediately. |
| Close | Saves. Rows naming no account are dropped; rows naming exactly the same set of accounts are **summed**; in multiple-record mode the partial-update marker is set to the ticked plan columns. In multiple-record mode the record set is then reloaded and the grid is emptied again. |
| Save as a new model | Opens a new distribution model form pre-filled with the grid as typed, the host record's partner, the host line's product and the **first three characters** of the financial account's display name as the account prefix. Creation only: editing an existing model from here is not offered. |
| The host's account or product changes | The relevant plans are fetched again, because the applicability may have changed, and the grid is rebuilt from the stored value. |
| The stored value names a missing account | The editor drops the missing identifiers and immediately saves the cleaned document. |

### 8.5 Keyboard and pointer behaviour

| Gesture | Effect |
|---|---|
| Focusing or clicking the closed cell | Opens the editor. |
| Down arrow on the closed cell | Opens the editor. |
| Tab or Enter inside the editor | Moves to the next focusable cell; when the next focusable element is the add-row control and the current row is valid — at least one account and a non-zero percentage — a row is added instead; when there is no next element, the editor closes and saves. |
| Shift with Tab | Moves to the previous cell; when there is none, the editor closes and saves. |
| Escape | Closes the editor **and saves**; it is not a cancel. |
| Clicking outside the widget | Closes and saves, unless the click lands in a dialogue or a hovering panel opened by the editor itself. |
| Resizing the window | Closes and saves, except on a mobile operating system. |

A rebuild must reproduce the fact that every way out of the editor saves. There is no discard.

---

## 9. Named operations

These are the operations a client or an integration calls by name. Each is given with its inputs,
its output, what it reads, what it changes and where its behaviour is specified. The keys of the
answers below are reproduced, because a client reads them by name.

### 9.1 On Analytic Plan

| Operation | Inputs | Output | Effects and specification |
|---|---|---|---|
| Ask for the relevant plans | All optional: the business domain, the company, the product, the financial account, a forced applicability, and the list of analytic account identifiers already present | An ordered list of entries, each carrying `id` (plan identifier), `name` (plan name), `color` (colour index), `applicability` (applicability), `all_account_count` (all analytic accounts count) and `column_name` (stored column name) | Fills the transaction-scoped cache keyed by the exact argument set. Fails with "A 'Project' plan needs to exist and its id needs to be set as `analytic.project_plan` in the system variables" when no base plan is designated. Algorithm: [calculations.md](calculations.md) sections 2 and 3. Called by the distribution editor when it opens and whenever the host's account or product changes, and by the mandatory plan validation. |
| Open the child plans | One plan | A window action titled "Analytical Plans" on its direct children, with the parent and the colour index pre-filled for creation | — |
| Open the analytic accounts | One plan | A window action titled "Analytical Accounts" on the accounts whose plan is that plan or any descendant, with that plan pre-filled for creation | — |

### 9.2 On Analytic Distribution Model

| Operation | Inputs | Output | Effects and specification |
|---|---|---|---|
| Ask for a proposed distribution | One set of criteria: the company, the partner, the list of partner categories, the product, the product category, the account prefix — the full code of the line's financial account — and the root plans already filled by a related document | One distribution document, possibly empty | Reads models, accounts and plans; changes nothing. Algorithm: [calculations.md](calculations.md) sections 4, 5 and 6. Called by the automatic proposal on a journal item and by the early payment discount computation, which passes the cash discount account's code as the prefix together with the company, the commercial partner and the partner's categories. |

### 9.3 On any record that carries a distribution

| Operation | Inputs | Output | Effects and specification |
|---|---|---|---|
| Validate the distribution | The same situation arguments as the relevant-plans question, plus the caller's validation flag | Nothing on success | Refuses with "One or more lines require a 100% analytic distribution." Algorithm: [calculations.md](calculations.md) section 8. Called by journal entry posting, sales order confirmation, purchase order confirmation, expense approval, manufacturing order confirmation, transfer validation and timesheet creation. |
| Merge two distributions | The stored document and the incoming document | The merged document | Used internally by every write of a distribution. Algorithm: [calculations.md](calculations.md) section 7. |
| Write the distribution | The document to write | Nothing | Merges, normalises to the percentage precision, stores, and — for a journal item of a posted entry — deletes and regenerates the analytic lines. Specification: [workflows.md](workflows.md) workflow 19. |

### 9.4 On Journal Item

| Operation | Inputs | Output | Effects and specification |
|---|---|---|---|
| Create the analytic lines | A set of journal items | Nothing | Validates the mandatory plans of the product lines, prepares the candidates, corrects the rounding and creates every line in one operation with the synchronisation guard raised. Refuses with "One or more lines require a 100% analytic distribution." — alone when one entry is concerned, and with a redirect titled "Items With Missing Analytic Distribution" behind a button labelled "See items" when several are. Algorithm: [calculations.md](calculations.md) sections 8, 9 and 10. |
| Rebuild the distribution from the analytic lines | A set of journal items | Nothing | Rewrites each item's distribution from its analytic lines, with the guard raised; returns immediately when the guard is already raised. Algorithm: [calculations.md](calculations.md) section 11. Called by the creation, the modification and the deletion of an analytic line. |

### 9.5 On Analytic Line and the plan-bearing contract

| Operation | Inputs | Output | Effects and specification |
|---|---|---|---|
| Ask for the mandatory plans | A company and a business domain | One entry per plan whose applicability is `mandatory` in that situation, each carrying the plan's name and its stored column name | Used by the timesheet check, which additionally ignores the base plan's column because a timesheet takes that account from its project. |
| Write a distribution onto analytic lines | A set of analytic lines and a distribution document | Nothing | Splits each line: the first set of values is written onto the line itself and each further set creates a copy. Notifies the acting reader when at least one copy was made. Algorithm: [calculations.md](calculations.md) section 12. |
| Redistribute the analytic lines of a document | A target distribution, a total amount, a total quantity, the lines currently attached, the document supplying the template values, and a flag saying whether the amounts are added to the existing ones | The values of the lines still to create | Updates or deletes the existing lines **in place**, preserving their identifiers. Algorithm: [calculations.md](calculations.md) section 16. Called by stock move valuation, work order costing and manufacturing order costing. |

### 9.6 On Analytic Account

| Operation | Inputs | Output |
|---|---|---|
| Open the customer invoices | One account | A window action titled "Customer Invoices" on the sale documents having a journal item whose distribution names the account, with creation disabled |
| Open the vendor bills | One account | A window action titled "Vendor Bills" on the purchase documents, receipts included, having such a journal item, with creation disabled |
| Open the gross margin | One account | The Gross Margin window action of section 2 |
| Open the projects, the purchase orders, the manufacturing orders, the bills of materials and the work orders | One account | Contributed by [../projects-and-tasks/](../projects-and-tasks/README.md), [../purchasing/](../purchasing/README.md) and [../manufacturing/](../manufacturing/README.md) |

### 9.7 Reading operations with special behaviour

| Operation | Entity | Special behaviour |
|---|---|---|
| Read one record for a form | Analytic Account | Reading exactly one account puts that account's plan into the reader's context, which makes the magic column and the account's collection of analytic lines resolve to the right stored column. |
| Describe the fields | Any plan-bearing entity | Each plan column's label is replaced by the plan's name and its selectable set is restricted to the accounts of that plan or of any descendant. Skipped in the view-customisation mode and for a reader who may not read plans (`AA-114`). |
| Read a view definition | Any plan-bearing entity | The definition is patched in memory to insert one column per non-base root plan after the base plan's column and one grouping filter per root plan and per sub-plan depth after the base plan's grouping filter. The stored definition is never modified. |
| Group by a derived total | Analytic Account | The balance, the debit and the credit are declared summable; the sum is computed in memory over the records of the group, optionally converting each account's figure into the currency of the company the reader is acting for ([calculations.md](calculations.md) section 13.3). |
| Group by a distribution | Any distribution-bearing entity | Only the count aggregate is supported; each row is one analytic account and the count is of a per-entity owner (`AA-056`, `AA-058`). |
| Search a distribution | Any distribution-bearing entity | Four operators only, with the name resolution described in `AA-053` to `AA-057`. |
| Title of a list | Analytic Line | When the reader's context names an analytic account, the list header reproduces "Entries: " followed by the account's name. |

---

## 10. Dialogues and notifications

| Kind | Trigger | Text and controls |
|---|---|---|
| Redirecting warning | Re-parenting a plan, or moving an account to another plan, would make an analytic line hold two different accounts of one axis | "Whoa there! Making this change would wipe out your current data. Let's avoid that, shall we?", with a button labelled "See them" that opens the offending analytic lines in a list, in a dialogue. Nothing is written (`AA-011`). |
| Redirecting warning | Several journal entries are posted at once and at least one product line fails the mandatory plan validation | "One or more lines require a 100% analytic distribution.", with a button labelled "See items" that opens a list titled "Items With Missing Analytic Distribution" restricted to the offending journal items (`AA-082`). |
| Validation refusal | The same failure while exactly one entry is being posted | The same text, raised plainly, with no redirect. |
| Validation refusal | An archived analytic account is named in a distribution of an entry being posted | "You cannot post an entry with an archived analytic account: the account names", the placeholder listing every archived account found, separated by a comma and a space. |
| Validation refusal | An analytic line has no account in any plan column | "At least one analytic account must be set". |
| Validation refusal | An analytic line's financial account differs from that of its journal item | "The journal item is not linked to the correct financial account". |
| Refusal on the form | A parent is chosen on the base plan | "You cannot add a parent to the base plan 'the plan name'". |
| Refusal on write | The base-plan parameter is given an unusable value | "The value for the key must be the ID to a valid analytic plan that is not a subplan". |
| Refusal on delete | A plan's column is still named by a stored view definition | "Cannot rename/delete fields that are still present in views:" followed by the field list and the view name. |
| Refusal on delete | An analytic account is used by an expense | "You cannot delete an analytic account that is used in an expense." |
| Refusal on delete | A project bound to the analytic account still has tasks | "Before we can bid farewell to these accounts, you need to tidy up the projects linked to them by removing their existing tasks!" |
| Refusal on write | The company of an analytic account is changed while lines exist elsewhere | "You can't change the company of an analytic account that already has analytic items! It's a recipe for an analytical disaster!" |
| Refusal on write | A distribution model names an account of one company while the model is shared or belongs to another | "You defined a distribution with analytic account(s) belonging to a specific company but a model shared between companies or with a different company" |
| Success notification | Splitting analytic lines created at least one line | The number of created lines, a space, and the words "analytic lines created". |

Every text above is reproduced exactly as the system emits it; the placeholders are described in
words in [business-rules.md](business-rules.md), which also carries the rule identifier of each.

No electronic mail, no message posting and no scheduled activity is produced by this domain. The
changes tracked on an analytic account reach its followers through the discussion thread, which
belongs to [../messaging-and-activities/](../messaging-and-activities/README.md).

---

## 11. The analytic columns other screens gain

| Screen of another domain | What this domain adds |
|---|---|
| The journal item lines of an invoice, a bill, a credit note, a refund and a miscellaneous entry | The analytic distribution cell, with the business domain computed from the document, the financial account field, the product field and the amount field bound to the line's own; the invalid-analytics highlight on a product line whose distribution fails the mandatory plan validation. |
| The journal item list | The analytic distribution column, hidden by default, editable for several rows at once, which is how a distribution is mass-applied. |
| A sales order line, a purchase order line, an expense, a reconciliation model line, an asset and a work centre | The analytic distribution cell with the business domain of that document. |
| A project's dashboard and task list | The embedded entry described in section 12. |
| The financial reports | Analytic filtering and grouping, owned by [../financial-reporting/](../financial-reporting/README.md). |

Every one of those cells is hidden from a reader outside the analytic accounting permission group
(`AA-112`), and every one of them is the component of section 8.

---

## 12. The project profitability entries

The project accounting capability contributes, to a project:

1. Three profitability sections computed from records this domain owns — "Vendor Bills" with
   sequence 11, "Other Revenues" with sequence 14 and "Other Costs" with sequence 15 — whose
   arithmetic is in [calculations.md](calculations.md) section 17. Pressing a section opens the
   records behind it: the vendor bills section opens the bills in the vendor bill screen; the two
   analytic sections open the analytic items, grouped by date, with graph and pivot presentations
   that group rows by date and drop the base plan's column, and with the partner grouping of the
   pivot hidden while the date grouping is active. A section carries its opening action only for a
   reader who holds the read-only accounting group, and the vendor bills section also for a reader
   who holds the invoicing group.
2. An embedded entry named "Analytic Items", with sequence 105, offered from the project's task
   list and from the project dashboard, visible only to a member of the analytic accounting
   permission group and only for a project that has an analytic account. It opens the analytic
   items of that project's analytic account, with that account pre-filled for creation; creating a
   line is offered **only** when the screen is reached through the embedded entry.

---

## 13. Reports and printable documents

This domain owns **no** printable document and no file-producing report. Analytic figures are read
through the list, cards, graph and pivot presentations of the analytic items, through the three
column totals of the analytic account list, and through the Gross Margin button of an analytic
account. The analytic reporting screen of section 2 is a window action, not a printable document,
and it is contributed by the general ledger capability.

---

## 14. Import and export

| Aspect | Behaviour |
|---|---|
| Export | Every entity of this domain is exportable through the platform's record export, which writes the chosen fields of the chosen records. The fiscal-year search helper of Analytic Line is excluded from the exportable fields, because it is a search helper with no value of its own. |
| Import of analytic lines | Supported through the platform's record import. Creating an analytic line that points at a journal item rebuilds that journal item's distribution (`AA-093`); creating one on a journal item of a **draft** entry is allowed and the line is consumed and then deleted, because a draft entry never owns analytic lines (`AA-073`). The procedure is workflow 22 of [workflows.md](workflows.md). |
| Import of a distribution | A distribution is a structured document; importing it writes the document as supplied, then normalises every percentage to the percentage precision (`AA-041`). An imported document carrying the reserved key `__update__` (the partial-update marker) triggers the merge instead of a replacement (`AA-043`). |
| Import of plans and accounts | Ordinary. A plan created by an import creates its stored column exactly as a plan created by hand does, so importing a list of plans reshapes the analytic line table. |
| Loading template | Analytic Line accepts the file-loading template contributed by the timesheets domain, used to load timesheet lines. |
| Exchange formats | None. This domain publishes no structured document format and takes part in no electronic exchange. |

---

## 15. Request endpoints and external integrations

This domain publishes **no** request endpoint of its own. Its screens are reached through the stable
paths listed in section 2, and every other interaction goes through the named operations of section
9. It contacts no third-party service and holds no credential.

The customer portal exposes analytic lines of the timesheet kind through the portal endpoints of
[../customer-portal/](../customer-portal/README.md) and
[../timesheets/](../timesheets/README.md); those endpoints belong to those domains.

---

## 16. Client behaviour a replacement must reproduce

1. **Reload on any plan change.** Whenever a create, write or delete on Analytic Plan succeeds,
   every open client reloads its definition of the plan-bearing entities, because their set of
   fields has changed. A client that caches entity definitions must invalidate that cache on the
   same event, otherwise a newly created axis is invisible until the page is reloaded by hand
   (`AA-021`).
2. **Batched reading of analytic accounts.** The distribution editor reads the display name, the
   colour index and the root plan of many accounts at once; the reads issued while one screen is
   being rendered are collected and answered together. The behaviour is unchanged without batching;
   the responsiveness of a long list is not.
3. **Lazy fetching of the relevant plans.** In a list, the relevant plans are fetched only when an
   editor is actually opened, never once per row. A rebuild that fetches per row multiplies the
   cost of opening a list of journal items by the number of rows.
4. **The percentage held as a fraction.** The editor holds each row's percentage as a fraction of
   one and formats it as a percentage; it rounds at the percentage precision plus two digits and
   multiplies by one hundred only when writing the document. A rebuild that rounds earlier loses
   the two working digits `AA-107` requires.
5. **Every exit saves.** Section 8.5. A rebuild that treats Escape or a click outside as a discard
   changes observable behaviour.

---

## 17. Reconciliation notes

1. **A single source, restructured.** Only one of the two drafts of this folder carried an
   interfaces document. Every row of it is kept. It described the screens as "workflows on
   presentations" and listed the screens and their stable paths in its configuration document; this
   file uses the charter's division instead — menus, actions, views, operations, notifications,
   import and export here, and settings, groups, rules and shipped records in
   [configuration.md](configuration.md).
2. **Operation names.** The former draft named each service operation by its internal identifier.
   The charter reproduces identifiers only where they are contractual — storage names, transport
   names, column names, stored values, route paths and message keys — so section 9 names each
   operation in words and reproduces only the keys of the relevant-plans answer, which a client
   reads by name.
3. **The base-plan parameter message.** The former draft rewrote it in full words; section 10 quotes
   the emitted text.
4. **The default grouping of the analytic item screens.** Neither draft mentioned it. The default
   grouping requested by the two actions names no existing filter, so both screens open ungrouped;
   this is recorded as a **compatibility finding** in section 2.
5. **The editor's exits.** Only one draft described the keyboard behaviour, and neither said that
   Escape and a click outside both save. Section 8.5 states it, because a rebuild that guesses will
   guess "discard".
6. **The empty-state explanations.** The former draft summarised them in one line. Section 7 gives
   the headline texts as reproduced strings and the three explanatory paragraphs in full, restated
   rather than reproduced because their wording names the product.
