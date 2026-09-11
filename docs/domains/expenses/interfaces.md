# Expenses — Interfaces

Every menu, window action, view, named operation, dialogue, printable document, route, message
template and external touch point of the domain.

---

## 1. Overview

The domain owns one application menu, four window actions on the Expense entity, one on expense
categories, two on configuration, and four short-lived dialogues. It contributes buttons and lists
to four other applications: accounting, sales, projects and human resources.

Every screen in this domain is built on the same single entity. There is no document hierarchy and
no line editor; an expense is one record with one receipt.

---

## 2. Menus

| Menu | Parent | Sequence | Visible to | Opens |
|---|---|---|---|---|
| *Expenses* (`menu_hr_expense_root`) | the application bar | 230 | everyone who sees the application | — |
| *My Expenses* (`menu_hr_expense_my_expenses`) | *Expenses* | 1 | every internal user | — |
| *My Expenses* (`menu_hr_expense_my_expenses_all`) | *My Expenses* | 1 | every internal user | the *My Expenses* action (§4.1) |
| *Expenses to Process* (`menu_hr_expense_expenses_to_process`) | *My Expenses* | 1 | every internal user | the *Expenses to Process* action (§4.2) |
| *Reporting* (`menu_hr_expense_reports`) | *Expenses* | 4 | every internal user | — |
| *Expenses Analysis* (`menu_hr_expense_all_expenses`) | *Reporting* | 0 | every internal user | the *Expenses Analysis* action (§4.3) |
| *Configuration* (`menu_hr_expense_configuration`) | *Expenses* | 100 | — | — |
| *Settings* (`menu_hr_expense_global_settings`) | *Configuration* | 0 | the system-administration group | the settings action (§4.7) |
| *Expense Categories* (`menu_hr_product`) | *Configuration* | 10 | expense Administrators | the expense-category action (§4.6) |
| *Activity Types* (`hr_expense_menu_config_activity_type`) | *Configuration* | — | the developer group only | the activity-type action (§4.8) |
| *Employee Expenses* (`menu_hr_expense_account_employee_expenses`) | the accounting application's payables menu | 22 | All Approvers | the *Employee Expenses* action (§4.4) |

The two entries named *My Expenses* — a section and the action under it — are deliberate: the
section also holds *Expenses to Process*, which is what an approver uses.

---

## 3. Views

### 3.1 The list

The base list is *"Expenses"*, sampled when empty, multi-editable, decorated in the information
colour for draft rows. Its columns:

| Column | Shown by default | Editable when | Notes |
|---|---|---|---|
| Employee | yes | the expense is editable | shown with the employee's portrait |
| Description | yes | the expense is editable | |
| Department | hidden | never | |
| Expense Date | yes | the expense is editable | |
| Category | yes | the expense is editable | required |
| Paid By | yes | the expense is editable | |
| Activities | yes | — | the activity summary widget |
| Analytic Distribution | yes | the expense is editable | analytic group only; carries the category hint and the business domain `expense` |
| Account | hidden | the expense is editable | accounting readers only |
| Journal | hidden | never | accounting readers only |
| Payment Method | hidden | the expense is editable | accounting readers only |
| Manager | hidden | the expense is editable | |
| Company | yes | never | multi-company only |
| Unit Price | hidden | never | shown in the company currency with its full precision |
| Quantity | hidden | the expense is editable **and** the category has a cost | |
| Included taxes | hidden | the expense is editable **and** the category has a cost | accounting roles only |
| Tax amount | hidden | never | summed in the footer as *"Total Taxes"*; accounting roles only |
| Number of Attachments | yes | never | rendered as a paper-clip with a count, and only when the count is greater than zero |
| Total | yes | the expense is editable **and** the category has **no** cost | company currency; summed in the footer as *"Total Amount"*; shown in bold |
| Total In Currency | hidden | the expense is editable, the expense is multi-currency **and** the category has no cost | multi-currency only; shown in bold |
| Currency | hidden | the expense is editable **and** the category has no cost | multi-currency only |
| Status | yes | never | a badge: information for *Draft*, warning for *Submitted*, success for *Approved*, *Posted*, *In Payment* and *Paid*, danger for *Refused* |

Two derived lists inherit it: one adds the dashboard band (§3.6), and one adds the band and is used
by *My Expenses*.

The list header carries three extra buttons beside the ordinary ones — *Submit*, *Approve* and
*Post Entries* — each shown only when the current selection makes it meaningful, and an *Upload*
button, labelled *Scan* on a small screen.

### 3.2 The form

Header, in order:

| Element | Shown when |
|---|---|
| *Submit* | the status is *Draft* |
| *Approve* | the status is *Submitted* **and** the approvability flag is true |
| *Post Journal Entries* | the status is *Approved*; accounting invoicing privilege only |
| *Attach Receipt* | always — placed **first** when the expense has no attachment, **last** when it has one |
| *Refuse* | the status is *Submitted* or *Approved*; Team Approvers only |
| *Reset* | the reset flag is true and the status is not *Draft*; for a user **without** accounting rights the button is additionally hidden on *Posted*, *In Payment* and *Paid* |
| *Split Expense* | see rule EXP-LIF-4 in [`business-rules.md`](business-rules.md) |
| the status bar | always |

The status bar shows a different set of stages depending on where the record is, so that a status
that does not apply is not displayed as a skipped stage:

| Current status | Stages shown |
|---|---|
| *Draft*, *Approved*, *Posted*, *Paid* | Draft, Approved, Posted, Paid |
| *Submitted* | Draft, Submitted, Approved, Posted, Paid |
| *In Payment* | Draft, Approved, Posted, In Payment, Paid |
| *Refused* | Draft, Approved, Posted, Paid, Refused |

Body:

- The description as the record title, with the placeholder *"e.g. Lunch with Customer"*.
- Two smart buttons: *Split*, shown when the expense belongs to a split family, which reopens the
  whole family; and *Journal Entry*, shown when an entry exists, restricted to the accounting
  invoicing privilege. With the rebilling capability a third appears: *Sales Order*, shown when the
  expense names one, restricted to salespeople.
- The category, with its own description shown in italics underneath when it has one.
- Then, according to the pricing model: either the unit price and the quantity with its unit
  (quantity-driven), or the total with its currency selector and a parenthetical
  *"(incl «tax amount» tax)"* shown only when the tax amount is non-zero (amount-driven).
- Then, when the expense is multi-currency or the category has a cost, the company-currency total
  beside the rate label.
- The taxes, with the receipt-currency tax amount beside them.
- The employee, offered against the public employee projection for users without human-resources
  access and against the full employee entity for those with it.
- The payment mode as radio buttons, read-only outside *Draft*.
- On the right: the expense date, the company, the manager with the placeholder
  *"Auto-validation"*, the vendor (company payment mode only), the account, the payment method
  (company payment mode only, required there), the analytic distribution and — with the rebilling
  capability — the customer to rebill to.
- The internal notes at the foot.
- The receipt preview panel beside the form, and the message thread below.

A second form inherits this one with the header hidden and the employee and company forced
read-only. It is used where an expense is shown inside another record's screen.

### 3.3 Banners on the form

| Banner | Shown when | Text |
|---|---|---|
| same receipt | the same-receipt set is not empty, and only while editing | *"An expense with the same receipt already exists."* where the word *expense* is a link opening the other expenses |
| unit-cost change | on the **category** form, while the unit cost is being edited and the test of [`calculations.md`](calculations.md) §2.6 passes | *"There are unsubmitted expenses linked to this category. Updating the category cost will change expense amounts. Make sure it is what you want to do."* |

### 3.4 The card view

Sampled when empty, with no quick creation, grouped by activity state in a progress bar coloured
success for planned, warning for today and danger for overdue. Each card shows the description in
bold, the receipt-currency total on the right, the employee with their portrait, the status as a
coloured label and the expense date.

Three variants inherit it: one hides the employee and the status, one adds the drop zone, and one
adds the drop zone and the dashboard band.

### 3.5 The analysis views

| View | Rows | Columns | Measures |
|---|---|---|---|
| Pivot *"Expenses Analysis"* | employee | expense date by month | total in receipt currency |
| Graph *"Expenses Analysis"* | expense date and employee | — | total in company currency, with the tax amount available |
| Activity *"Expenses"* | one row per employee, showing their portrait, the description and the receipt-currency total | — | — |

### 3.6 The dashboard band

A band of three figures shown above the list and above the card view in *My Expenses*. Each figure
is a large monetary amount with a caption, separated by arrow glyphs.

| Caption | Figure |
|---|---|
| *"To Submit"* | the sum defined in [`calculations.md`](calculations.md) §12 for the draft status |
| *"Waiting Approval"* | the same for the submitted status |
| *"Waiting Reimbursement"* | the same for the approved status, employee-paid only |

Pressing a figure deactivates every active filter whose criterion mentions the status and then
activates the filter of the same name, so the list below shows exactly the records the figure
counted. The band is hidden on a small screen. In developer mode a question mark is shown at the
end with the explanation *"Numbers computed from your personal expenses."*

The figures are recomputed whenever the view's own filter changes, and the view's current criterion
is passed into the computation, so filtering the list narrows the figures too.

### 3.7 The search view

| Search field | Matches on |
|---|---|
| *Expense* | the description, case-insensitively |
| *Department* | the department's name |
| *Company* | the company's name; multi-company only |
| *Employee* | the employee's name |

| Filter | Criterion | Restricted to |
|---|---|---|
| *My Expenses* | the employee's user is the acting user | — |
| *My Team* | the employee's hierarchical parent's user is the acting user | Team Approvers; help text *"Expenses of Your Team Member"* |
| *Paid by Company* | payment mode `company_account` | — |
| *Paid by Employee* | payment mode `own_account` | — |
| *To Submit* | status `draft` | — |
| *Waiting Approval* | status `submitted` | — |
| *To Post* | status `approved` | — |
| *Waiting Reimbursement* | status `approved` and payment mode `own_account` | — |
| *Expense Date* | a date range on the expense date | — |
| *All to Post* | status `approved` | hidden; used by the *Employee Expenses* action |
| *All to Pay* | status `posted` and payment mode `own_account` | hidden; used by the *Employee Expenses* action |
| *All Paid* | status `in_payment` or `paid`, payment mode `own_account` | hidden |
| *My Activities*, *Late Activities*, *Today Activities*, *Future Activities* | the activity criteria of [`../messaging-and-activities/`](../messaging-and-activities/) | hidden |

Groupings offered: *Employee*, *Category*, *Status*, *Expense Date*, *Company* (multi-company only)
and *Department*.

A second search view inherits it and adds a side panel with three facets: the status, expanded and
multi-selectable with counters; the employee, limited to twenty entries, flat, single-selection;
and the company, expanded, multi-company only.

---

## 4. Window actions

### 4.1 *My Expenses* (`hr_expense_actions_my_all`)

| Property | Value |
|---|---|
| Route segment | `expenses` |
| Entity | Expense |
| Views | list, card, form, graph, pivot, activity — the list and the card view being the dashboard-band variants |
| Search view | the base search view |
| Default filter | *My Expenses* |
| Empty-list help | an illustrated drop zone with the heading *"Upload or drop an expense receipt"*, followed by the mailbox advertisement when the mailbox is on |

### 4.2 *Expenses to Process* (`hr_expense_actions_to_process`)

| Property | Value |
|---|---|
| Route segment | `expenses-to-process` |
| Views | list, card, form, graph, pivot, activity |
| Search view | the side-panel search view |
| Default side-panel selection | status *Submitted* |
| Empty-list help | the same drop zone as §4.1 |

This is the approver's screen, and the address the weekly reminder links to.

### 4.3 *Expenses Analysis* (`hr_expense_actions_all`)

| Property | Value |
|---|---|
| Route segment | `expenses-analysis` |
| Views | graph, pivot, list, form — the graph, pivot and plain list being named explicitly |
| Search view | the side-panel search view |
| Default side-panel selection | *Draft*, *Submitted*, *Approved*, *Posted*, *In Payment*, *Paid* — every status but *Refused* |
| Empty-list help | *"No data yet!"* and *"Create new expenses to get statistics."* |

### 4.4 *Employee Expenses* (`action_hr_expense_account`)

| Property | Value |
|---|---|
| Route segment | `expenses-employee` |
| Views | list (the dashboard variant), card, form, pivot, graph |
| Search view | the base search view |
| Default filters | *All to Post* **and** *All to Pay* |
| Empty-list help | *"Create a new expense"* and *"Once you have created your expense, submit it to your manager who will validate it."* |

Placed in the accounting application under payables. Its two default filters combine to the
accountant's working set: everything approved and everything posted but not yet reimbursed.

### 4.5 The two department actions

| Action | Route segment | Purpose |
|---|---|---|
| *Expense to Approve* (`action_hr_expense_department_to_approve`) | `expense-to-approve` | The submitted expenses of one department. Restricted to that department by the record the action is launched from; the side panel defaults to *Submitted*. |
| *Expense Analysis* (`action_hr_expense_department_filtered`) | — | The graph and pivot views grouped by one department, which is also the default for new records. |

Both are reached from the department card view, which shows the count of expenses awaiting
approval as a link and offers *Expenses* in its reports menu, each restricted to Team Approvers.

### 4.6 *Expense Categories* (`hr_expense_product`)

| Property | Value |
|---|---|
| Entity | Product Variant |
| Restriction | the "can be expensed" flag is set |
| Views, in order | a dedicated list, a dedicated card view, a dedicated form |
| Defaults for a new record | expensable, kind *service*, and — with the rebilling capability — a rebilling policy of *At cost* |
| Empty-list help | *"No expense categories found. Let's create one!"* and *"Expense categories can be reinvoiced to your customers."* |

The dedicated **form** shows: the unit-cost banner; the image; the name with the placeholder
*"e.g. Lunch"*; the unit cost with, beside it and only when the cost is non-zero, the word *per* and
the reference unit; the internal reference; the product category; the company; a description
labelled *Guideline* with the placeholder *"e.g. Restaurants: only week days, for lunch"*; the
expense account and the supplier taxes with the placeholder *"no taxes"*. The unit cost carries the
help text:

> "When the cost of an expense product is different than 0, then the user using this product won't be able to change the amount of the expense, only the quantity. Use a cost different than 0 for expense categories funded by the company at fixed cost like allowances for mileage, per diem, accommodation or meal."

With the rebilling capability the form additionally shows an *Invoicing* group holding the
rebilling policy as radio buttons with its explanatory sentence underneath, the list price shown
only when the policy is *Sales price*, and the customer taxes shown whenever the policy is not
*No*.

The dedicated **list** shows the name, the internal reference, the description labelled *Note*, the
sales price, the unit cost and the supplier taxes; with the rebilling capability it adds the
rebilling policy and the customer taxes.

### 4.7 Settings (`action_hr_expense_configuration`)

Opens the settings form scrolled to the *Expenses* block.

### 4.8 Activity types (`mail_activity_type_action_config_hr_expense`)

Lists the activity types that either target no entity or target the Expense entity, and defaults a
new one to the Expense entity.

### 4.9 *Expenses* from a sales order (`hr_expense_action_from_sale_order`)

Lists the expenses pointing at one sales order and defaults a new one to that order. Reached from
the order's *Expenses* smart button.

---

## 5. Named operations

| Operation | Entity | Effect | Specified in |
|---|---|---|---|
| *Submit* (`action_submit`) | Expense | Submits, or approves automatically | [`workflows.md`](workflows.md) §3 |
| *Approve* (`action_approve`) | Expense | Approves, or opens the duplicate dialogue | [`workflows.md`](workflows.md) §4 |
| *Refuse* (`action_refuse`) | Expense | Opens the refusal dialogue | [`workflows.md`](workflows.md) §5 |
| *Reset* (`action_reset`) | Expense | Removes the accounting and returns to *Draft* | [`workflows.md`](workflows.md) §6 |
| *Post Journal Entries* (`action_post`) | Expense | Posts both payment modes | [`workflows.md`](workflows.md) §8 |
| *Register Payment* (`action_pay`) | Expense | Opens the register-payment dialogue on the expense's entry, pre-filling the recipient bank account when the entry names exactly one | [`workflows.md`](workflows.md) §11 |
| *Split Expense* (`action_split_wizard`) | Expense | Builds two proposed pieces and opens the split dialogue | [`workflows.md`](workflows.md) §7 |
| *Split* (`action_open_split_expense`) | Expense | Opens the whole split family as a list titled *"Split Expenses"* | — |
| *Journal Entry* (`action_open_account_move`) | Expense | Opens the entry for an employee-paid expense, and the **payment** for a company-paid one | — |
| *Sales Order* (`action_open_sale_order`) | Expense | Opens the sales order on its ordinary form | — |
| (similar receipts) (`action_show_same_receipt_expense_ids`) | Expense | Opens the same-receipt set, titled *"Expenses with a similar receipt to «this expense's description»"* | [`calculations.md`](calculations.md) §7.2 |
| (duplicate acknowledgement) (`action_approve_duplicates`) | Expense | Posts, on every duplicate, a message authored by the system partner reading *"«acting user's name» confirms this expense is not a duplicate with similar expense."* | — |
| (dashboard figures) (`get_expense_dashboard`) | Expense | Returns the three dashboard figures | [`calculations.md`](calculations.md) §12 |
| (create from files) (`create_expense_from_attachments`) | Expense | Creates one expense per uploaded file | [`workflows.md`](workflows.md) §2.2 |
| (attach a receipt) (`attach_document`) | Expense | Sets the last uploaded file as the main attachment | [`workflows.md`](workflows.md) §2.3 |
| *Expenses* (`action_open_expense`) | Journal Entry | Opens the entry's expenses: a form for one, a list for several | — |
| *Expense* (`action_open_expense`) | Payment | Opens the payment's expenses | — |
| *Expenses* (`action_open_project_expenses`) | Project | Opens the expenses whose analytic distribution names the project's account, carrying the project in context | [`workflows.md`](workflows.md) §2.6 |

---

## 6. Dialogues

### 6.1 The posting dialogue

| Property | Value |
|---|---|
| Title | *"Post expenses paid by the employee"*, or *"Post expenses"* when no company-paid expense was posted in the same operation |
| Fields | *Journal*, *Accounting date* |
| Buttons | *Post Expenses* (primary), *Cancel* |

### 6.2 The refusal dialogue

| Property | Value |
|---|---|
| Action name | *"Refuse Expense"* |
| Size | medium |
| Fields | *Reason*, required, full width |
| Buttons | *Refuse* (primary), *Cancel* |

### 6.3 The duplicate confirmation dialogue

| Property | Value |
|---|---|
| Action name | *"Validate Duplicate Expenses"* |
| Leading text | *"The following approved expenses have similar employee, amount and category than some expenses of this report. Please verify this report does not contain duplicates."* |
| List columns, all read-only | expense date, employee with portrait, category, company-currency total, description, manager with portrait, approval date |
| Buttons | *Refuse* (primary), *Approve* (secondary), *Cancel* — each of the first two hidden when the list is empty |

The *Refuse* button being the primary one is deliberate: the dialogue exists because the system
suspects a duplicate, and the safer answer is offered first.

### 6.4 The split dialogue

| Property | Value |
|---|---|
| Title | *"Expense split"* |
| Warning | *"The total amount doesn't match the original amount."*, shown while the sum check fails |
| Line columns, editable at the bottom | description, category, employee with portrait, taxes as tags, tax amount, analytic distribution, total — the total being read-only when the chosen category has a unit cost; with the rebilling capability, a customer-to-rebill column whose editability follows the category's policy |
| Footer figures | the running total (shown in the danger colour while the check fails), the original amount, the running tax total |
| Buttons | *Split Expense* — rendered disabled while the check fails and active once it passes — and *Cancel* |

---

## 7. Capturing receipts

### 7.1 Upload and drag-and-drop

The list and the card view of the expense actions both carry:

- a hidden file selector accepting any type and any number of files;
- a primary button labelled *Upload* on a large screen and *Scan* on a small one;
- a full-view drop zone that highlights while a file is dragged over it and shows a large upload
  glyph.

Dropping files, choosing files, or pressing the illustrated empty-list drop zone all lead to the
same operation: upload the files, then create one expense per file
([`workflows.md`](workflows.md) §2.2), then show the created expenses in a list named
*"Generate Expenses"*. A second upload while that list is open widens the list instead of replacing
it.

When the upload produces nothing usable, a notification reads *"An error occurred during the
upload"*.

### 7.2 The share target

The views register themselves as a share target: files shared to the application from elsewhere on
the device are collected when the view starts and fed into the same upload operation, so a receipt
photographed in another application arrives as an expense without the user opening a file selector.

### 7.3 The receipt panel and the attachment counter

The form shows the main attachment beside it in a preview panel. The list shows, for each row with
at least one attachment, a paper-clip glyph with the count in superscript; rows with no attachment
show nothing.

### 7.4 The application download code

Links marked as application links do not navigate. On a large screen they open a modal titled
*"Download our App"* showing a quick-response code generated from the link's address, under the
heading *"Scan this QR code to get the Odoo app:"*. On a small screen the link is followed
directly. The heading is reproduced verbatim; it is the only place in this folder where a shipped
interface string names the vendor's application.

---

## 8. Printable documents

### 8.1 The expense document

| Property | Value |
|---|---|
| Name | *"Expenses Report"* |
| Entity | Expense |
| Format | a portable document rendered from a template |
| File name | *"Expense - «employee name» - «expense description with every solidus removed»"* |
| Offered as | a print action bound to the Expense entity |

Layout: the external company layout; the heading *"Expenses Report"*; the expense description as a
sub-heading; then a two-column header block holding *Employee*, *Date* (omitted when the expense
has no date), *Manager* (omitted when there is none) and *Paid by*; then a table.

The table has one row — an expense is a single line — with the columns *Name*, *Unit Price*,
*Quantity*, *Taxes*, *Subtotal in currency* (shown only when the receipt currency differs from the
company currency) and *Subtotal*. The taxes cell lists the labels of the expense's taxes separated
by a comma and a space, skipping taxes that have no label.

The totals block then shows *Untaxed Amount*, *Taxes* and, in bold, *Total*, all three in the
**company** currency.

### 8.2 Appending the receipts to the document

Rendering the expense document does more than render the template. For each expense being printed:

1. Render the template to a document stream.
2. Read every attachment of that expense.
3. For each attachment:
   - when it is itself a portable document, convert it to a stream directly;
   - otherwise render the **image template** (§8.3) with that attachment and take its stream.
4. Append every attachment stream to the document, each starting a new page.
5. If an attachment cannot be appended, do not fail: log on the expense's message thread
   *"The attachment («attachment name») has not been added to the report due to the following
   error: '«the error»'"* and skip that attachment.
6. Replace the expense's stream with the combined one.

The result is one file holding the expense sheet followed by every receipt, which is what an
auditor needs.

### 8.3 The receipt image document

| Property | Value |
|---|---|
| Name | *"Expense Report Image"* |
| Entity | Expense |
| Offered as | not bound to a menu; invoked only by §8.2 |

It renders, on a basic layout, the attachment's record name as a heading and the attachment itself
as an image — and renders nothing at all when the attachment is a portable document, that case
being handled by the direct conversion of §8.2.

### 8.4 The localised variant

A localisation capability inserts a document-title block before the page, setting the title to
*"Expenses Report"* when the record is an Expense, so that the sheet conforms to a standardised
business-letter layout. Nothing else changes. See [`configuration.md`](configuration.md) §18.

---

## 9. The electronic-mail interface

| Direction | Trigger | Content |
|---|---|---|
| Inbound | a message to the expense mailbox address | Creates an expense ([`workflows.md`](workflows.md) §2.4). The message becomes the first message of the thread and its attachments become the expense's receipts. |
| Outbound | the mailbox acknowledgement | Posted as a note on the expense when the employee has a user; sent directly to the sender's address otherwise. Subject *"Re: «the original subject»"*. |
| Outbound | the weekly reminder | Sent to each approver's employee work address, falling back to their own. Subject *"New expenses waiting for your approval"*. |
| Outbound | the refusal message | Posted on the expense with the comment subtype, so every follower is notified. |

The templates are listed in [`configuration.md`](configuration.md) §13.

The empty-list help of every expense view is extended, while the mailbox is on and the alias has
both a local part and a domain, with a muted line reading *"Tip: try sending receipts by email"*
followed by the address rendered as a link whose subject is pre-filled with
*"Lunch with customer $12.32"*. The space in that subject is encoded in the link so that the
message client shows it correctly.

---

## 10. Presence in other applications

| Application | Where | What |
|---|---|---|
| Accounting | the payables menu | the *Employee Expenses* action (§4.4) |
| Accounting | the journal-entry form's button box | an *Expenses* smart button showing the count of expenses the entry carries, hidden when the count is zero |
| Accounting | a dedicated entry list | used when a posting produced several entries, titled *"New expense entries"* |
| Accounting | the payment form's button box | an *Expense* smart button, shown when the payment carries expenses |
| Sales | the order form's button box | an *Expenses* smart button showing the expense count, hidden when it is zero |
| Sales | a dedicated order list | used by the expense's order selector; it hides the monetary columns from users who are neither salespeople nor accounting users |
| Human resources | the employee form's managers group | the *Expense* approver field, with the placeholder *"Auto-validation"* |
| Human resources | the employee list | the expense approver as a hidden-by-default column |
| Human resources | the employee search view | a grouping by *Expense Approver* |
| Human resources | the department card view | the count of expenses awaiting approval as a link, and *Expenses* in the reports menu — both for Team Approvers |
| Projects | beside the task views and beside the project update dashboard | an *Expenses* embedded action, for All Approvers |
| Projects | the profitability panel | an *Expenses* section with a cost side and, with the project-and-sales capability, a revenue side |
| Products | the product form | the "can be expensed" flag beside the sales flag |
| Products | the product search view | an *Expenses* filter on the expensable flag |
| Dashboards | the finance dashboard group | the shipped expense dashboard document |

### 10.1 The activity menu

The global activity menu opens an activity group for the Expense entity in the **card** view rather
than the list view when the window is narrower than the medium breakpoint, because an expense card
is readable on a telephone and a row of the expense list is not.

---

## 11. Routes

| Route segment | Action |
|---|---|
| `expenses` | *My Expenses* |
| `expenses-to-process` | *Expenses to Process* |
| `expenses-analysis` | *Expenses Analysis* |
| `expenses-employee` | *Employee Expenses* |
| `expense-to-approve` | *Expense to Approve*, for one department |

The acknowledgement message links to the created expense through the generic record address of
[`../../interfaces/README.md`](../../interfaces/README.md); the weekly reminder links to
`expenses-to-process`.

---

## 12. Import and export

The domain adds no import or export format of its own. Three generic mechanisms apply:

| Mechanism | Applicability |
|---|---|
| The tabular import and export of [`../../interfaces/README.md`](../../interfaces/README.md) | Every field of the Expense entity that is stored and writable, with the caveat that the status cannot be imported — it is derived — and that the approval state can be, which is how a bulk migration reproduces an approval chain |
| The programmatic interface of [`../../runtime/request-lifecycle.md`](../../runtime/request-lifecycle.md) | Every named operation of §5 |
| The mailbox of §9 | One expense per message |

There is **no** structured-document exchange for expenses: an expense is an internal record, and the
only document that leaves the organisation because of one is the customer invoice raised from a
rebilling line, which is specified in
[`../electronic-invoicing-and-document-exchange/`](../electronic-invoicing-and-document-exchange/).

---

## 13. Field-level visibility summary

Three groups change what a user sees on the expense screens.

| Group | Hides when absent |
|---|---|
| the multi-company group | the company column, the company field and the company grouping |
| the multi-currency group | the currency selector, the receipt-currency total column and the currency column |
| the units-of-measure group | the unit selector beside the quantity |
| the analytic-accounting group | the analytic distribution on the form, the list and the split dialogue |
| accounting readers | the account, the journal and the payment method columns |
| accounting invoicing privilege | the *Post Journal Entries* button, the *Journal Entry* smart button, the taxes and tax-amount columns |
| Team Approver | the *Refuse* button, the *My Team* filter |
| salespeople | the *Sales Order* smart button |
