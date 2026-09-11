# Expenses — Configuration

Every setting, system parameter, numbering series, shipped record, security group, access right,
record rule, scheduled job, activity type, message subtype and message template the domain
contributes.

---

## 1. What has to be configured before an expense can be posted

| # | Thing | Where | Consequence if missing |
|---|---|---|---|
| 1 | At least one expensable category | §4 | Receipt upload refuses with the message of rule EXP-ATT-6 in [`business-rules.md`](business-rules.md); submission refuses with the no-category message |
| 2 | An expense account reachable from the category, the company or the journal | §9.1 and [`calculations.md`](calculations.md) §6.2 | Posting refuses with rule EXP-ACC-1 |
| 3 | A purchase journal for employee-paid expenses | §9.1 | The posting dialogue offers no journal |
| 4 | A work contact on every employee, with a payable account property | [`../human-resources-core/`](../human-resources-core/) | Posting refuses with rule EXP-ACC-2 |
| 5 | At least one outbound payment method line, for company-paid expenses | §9.2 | Posting refuses with rule EXP-PST-5 |
| 6 | An outstanding-payments account, or a chart of accounts able to create one | [`calculations.md`](calculations.md) §6.4 | Posting refuses with rule EXP-ACC-4 when the account exists but is archived |
| 7 | An alias domain, if expenses are to arrive by electronic mail | §10 | The mailbox setting is offered but the address cannot be completed |

---

## 2. Settings

The settings are gathered under an application block named *"Expenses"*, visible only to an expense
Administrator. It carries two panels.

### 2.1 The *Expenses* panel

| Setting | Storage name | Kind | Meaning |
|---|---|---|---|
| *"Incoming Emails"* — *"Create expenses from incoming emails"* | `hr_expense_use_mailgateway` | true or false, backed by the system parameter `hr_expense.use_mailgateway` | Switches the expense mailbox on. Its explanatory title reads: *"Send an email to this email alias with the receipt in attachment to create an expense in one click. If the first word of the mail subject contains the category's internal reference or the category name, the corresponding category will automatically be set. Type the expense amount in the mail subject to set it on the expense too."* |
| *"Alias"* | `hr_expense_alias_prefix` | text | The local part of the mailbox address. Shown only while the mailbox is on and an alias domain exists. |
| (the domain part) | `hr_expense_alias_domain_id` | reference to an alias domain | The domain part of the mailbox address. Offered as a selector with no creation and no opening. |
| *"Reimburse in Payslip"* — *"Refund employees via their payslips."* | `module_hr_payroll_expense` | true or false | Activates the payroll-reimbursement capability. |
| *"Expense Digitalization (OCR)"* — *"Digitalize your receipts with OCR and Artificial Intelligence"* | `module_hr_expense_extract` | true or false, per company | Activates the optical-character-recognition digitisation capability, whose own label reads *"Send bills to OCR to generate expenses"*. Its explanatory title reads *"use OCR to fill data from a picture of the bill"*. |
| *"Expense Card"* — *"Create prepaid virtual and physical cards for both one-time and recurring expenses, integrated into Odoo through Stripe Issuing."* | `module_hr_expense_stripe` | true or false, per company | Activates the company-card issuing capability. While it is on, the panel shows *"Save this page and come back here to set up the feature."* |

The last three are activation switches: turning one on installs a capability package and then the
switch stays on. The three long strings above are reproduced verbatim because they are the labels a
user sees; two of them name the vendor of the system and a third-party card issuer, and are the
only such strings in this file.

When the mailbox setting is turned **off**, both alias fields are cleared by their computations, so
the address disappears from the settings even though the alias record itself survives.

### 2.2 The *Accounting* panel

| Setting | Storage name | Kind | Meaning |
|---|---|---|---|
| *"Employee Expense Journal"* — *"Default accounting journal for expenses paid by employees."* | `expense_journal_id` | reference to a journal, per company | Mirrors the company's default expense journal. Restricted to purchase journals of the company. |
| *"Payment methods"* — *"Payment method allowed for expenses paid by company."* | `company_expense_allowed_payment_method_line_ids` | set of references to payment method lines, per company | Mirrors the company's allow-list. Shown as tags with the placeholder *"All payment methods allowed"* and no creation. |

### 2.3 What saving the settings does to the alias

| Situation | Effect of saving |
|---|---|
| No alias record exists and a local part was typed | Create an alias with that local part, the alias domain of the acting company, a contact policy of `employees`, and the Expense entity as its target; and register it under the external identifier `hr_expense.mail_alias_expense` so that it is found again |
| An alias record exists and the typed local part differs from its own | Write the new local part onto the alias |
| The domain part was changed | Write the new alias domain onto the alias |
| No alias record exists and no local part was typed | Nothing |

Reading the settings fills the two alias fields from the alias record when there is one, and leaves
them empty otherwise.

---

## 3. System parameters

| Parameter | Values | Effect |
|---|---|---|
| `hr_expense.use_mailgateway` | true or false | Whether the expense mailbox is active. **Shipped as true.** It is read with elevated rights when the empty-list help of an expense view is rendered, to decide whether to advertise the address. |

---

## 4. Shipped expense categories

Six Product Variants are shipped, each a service, each with the "can be expensed" flag set, each
purchasable and none sellable, and all in the product category *Expenses* when that category
exists. Their images are shipped alongside.

| External identifier | Name | Internal reference | Unit cost | Reference unit | Description | Rebilling policy | Invoicing policy |
|---|---|---|---|---|---|---|---|
| `expense_product_meal` | *Meals* | `FOOD` | 0.00 | *Units* | *"Restaurants, business lunches, etc."* | *Sales price* | as shipped |
| `expense_product_travel_accommodation` | *Travel & Accommodation* | `TRANS & ACC` | 0.00 | *Units* | *"Hotel, plane ticket, taxi, etc."* | *At cost* | delivered quantities |
| `expense_product_mileage` | *Mileage* | `MIL` | **1.00** | *Kilometres* | none | *Sales price* | delivered quantities |
| `expense_product_gift` | *Gifts* | `GIFT` | 0.00 | *Units* | *"Gifts to customers or vendors"* | as shipped | as shipped |
| `expense_product_communication` | *Communication* | `COMM` | 0.00 | *Units* | *"Phone bills, postage, etc."* | *At cost* | as shipped |
| `product_product_no_cost` | *Expenses* | `EXP_GEN` | 0.00 | *Units* | none | as shipped | as shipped |

Four properties of this table carry behaviour:

1. *Mileage* is the only **quantity-driven** category shipped: its non-zero unit cost switches the
   pricing model of [`calculations.md`](calculations.md) §2.1.
2. Shipping *Mileage* also **activates the kilometre unit of measure**, which is inactive by
   default.
3. `EXP_GEN` is the category the bulk receipt-upload operation looks for by internal reference; it
   falls back to the first expensable variant when that reference is absent
   ([`workflows.md`](workflows.md) §2.2).
4. The rebilling policies in the last two columns are applied by the expense-rebilling capability
   when it is installed; without that capability every category's policy is *No*.

None of the six carries a tax: a new installation starts untaxed, and the default supplier taxes
are deliberately cleared when a category is created through the expense-category action (rule
EXP-EXT-11).

---

## 5. Numbering series

| External identifier | Name | Code | Prefix | Padding |
|---|---|---|---|---|
| `seq_hr_expense_invoice` | *"Expense invoice"* | `hr.expense.invoice` | `EXP/` | 3 |

The Expense entity itself is **not** numbered: it has no reference field and is displayed by its
description. The series is shipped for the accounting documents the domain produces and is not
consumed by the Expense. The entries produced by posting take their numbers from the numbering
series of their journal, as specified in
[`../general-ledger/configuration.md`](../general-ledger/configuration.md).

---

## 6. Security groups

### 6.1 The privilege

| External identifier | Name | Sequence | Category |
|---|---|---|---|
| `res_groups_privilege_expenses` | *"Expenses"* | 12 | the human-resources application category |

A privilege is the unit a user's access is chosen by on the user form: one privilege, one of its
levels.

### 6.2 The three levels

| External identifier | Label | Sequence | Implies | Shipped members |
|---|---|---|---|---|
| `group_hr_expense_team_approver` | *"Team Approver"* | 10 | the internal-user group | none |
| `group_hr_expense_user` | *"All Approver"* | 20 | `group_hr_expense_team_approver` | none |
| `group_hr_expense_manager` | *"Administrator"* | 30 | `group_hr_expense_user` | the system user and the administrator user |

What each level means in behaviour is specified in [`business-rules.md`](business-rules.md) §3.1.
Two further groups outside this domain change what a user may do here:

| Group | Effect here |
|---|---|
| the accounting invoicing privilege | May create, read and update expenses but **not delete** them; is the only role that may post; may open the posting dialogue; sees the *Reset* button on every status |
| the full accounting privilege | Additionally receives the unrestricted record rule of §8 |

---

## 7. Model access rights

Read, write, create and delete permissions per entity and group. A blank cell means the permission
is not granted by that row.

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Expense (`hr.expense`) | internal user | yes | yes | yes | yes |
| Expense | Team Approver | yes | yes | yes | yes |
| Expense | Administrator | yes | yes | yes | yes |
| Expense | accounting invoicing privilege | yes | yes | yes | **no** |
| Journal (`account.journal`) | Team Approver | yes | no | no | no |
| Journal Entry (`account.move`) | Team Approver | yes | no | no | no |
| Journal Item (`account.move.line`) | Team Approver | yes | no | no | no |
| Analytic Line (`account.analytic.line`) | Team Approver | yes | yes | yes | yes |
| Activity Type (`mail.activity.type`) | Administrator | yes | yes | yes | yes |
| Expense Refusal Dialogue (`hr.expense.refuse.wizard`) | Team Approver | yes | yes | yes | no |
| Duplicate Expense Confirmation Dialogue (`hr.expense.approve.duplicate`) | Team Approver | yes | yes | yes | no |
| Expense Split Dialogue (`hr.expense.split.wizard`) | internal user | yes | yes | yes | no |
| Expense Split Line (`hr.expense.split`) | internal user | yes | yes | yes | yes |
| Expense Posting Dialogue (`hr.expense.post.wizard`) | accounting invoicing privilege | yes | yes | yes | yes |

Two rows deserve comment:

- The internal user is granted **every** permission on the Expense entity at this layer. All the
  narrowing is done by the record rules of §8, not by the access matrix. A rebuild that enforces
  narrowing at this layer instead will refuse operations the system allows.
- The Team Approver's read access to journals, entries and items is what lets an approver open the
  entry produced from an expense they approved. It is narrowed by the two record rules of §8.3 to
  the entries and items that actually carry an expense.

---

## 8. Record rules

### 8.1 On the Expense

| External identifier | Name | Groups | Scope |
|---|---|---|---|
| `hr_expense_comp_rule` | *"Expense multi company rule"* | **global** | every operation, restricted to expenses whose company is among the companies the user is acting for |
| `ir_rule_hr_expense_manager` | *"Manager Expense"* | the full accounting privilege, All Approver | unconditional |
| `ir_rule_hr_expense_approver` | *"Team Approver Expense"* | Team Approver | the employee is the user's own, **or** the employee's department is managed by the user, **or** the employee is at or below one of the user's employees, **or** the employee's designated expense approver is the user, **or** the expense's manager is the user |
| `ir_rule_hr_expense_employee` | *"Employee Expense"* | internal user | the employee's designated expense approver is the user and the status is *Draft*, *Submitted*, *Approved* or *Refused*; **or** the employee is the user's own and the status is *Draft* |
| `ir_rule_hr_expense_employee_not_draft` | *"Employees can't modify an expense that is not in draft state"* | internal user | **read only** — create, write and delete are switched off: the employee is the user's own and the status is **not** *Draft*; **or** the employee's designated expense approver is the user and the status is *Submitted*, *Approved* or *Refused* |

### 8.2 On the Expense Split Dialogue

| External identifier | Name | Groups | Scope |
|---|---|---|---|
| `ir_rule_hr_expense_split_employee` | *"Employee Expense Split"* | internal user | the expense is in *Draft* **and** its employee is the user's own or its manager is the user |
| `ir_rule_hr_expense_split_approver` | *"Approver Expense Split"* | Team Approver | the expense is in *Draft* or *Submitted* **and** its manager is the user or is empty |
| `ir_rule_hr_expense_split_user` | *"All approver Expense Split"* | All Approver | the expense is in *Draft* or *Submitted* **and** its employee is **not** the user's own, or its manager is the user or is empty |
| `ir_rule_hr_expense_split_manager` | *"Manager Expense Split"* | Administrator | the expense is in *Draft* or *Submitted* |
| `ir_rule_hr_expense_split_accountant` | *"Accountant Expense Split"* | accounting invoicing privilege | the expense is in *Draft*, *Submitted* or *Approved* |

The five together are what make the split button's visibility rule of EXP-LIF-4 enforceable: even
if a user reaches the dialogue by another route, the rules of their own group decide which
expenses' dialogues they may read and write.

### 8.3 On the accounting entities

| External identifier | Name | Groups | Scope |
|---|---|---|---|
| `hr_expense_team_approver_account_move_rule` | *"Expense Team Approver Account Move"* | Team Approver | journal entries that carry at least one expense |
| `hr_expense_team_approver_account_move_line_rule` | *"Expense Team Approver Account Move Line"* | Team Approver | journal items that carry an expense |

---

## 9. Company configuration

### 9.1 Fields added to the Company

| Field | Storage name | Restriction | Use |
|---|---|---|---|
| *"Default Expense Journal"* | `expense_journal_id` | purchase journals of the company | First candidate of the posting dialogue's journal default; written back automatically the first time a journal is chosen and the company has none. Help text: *"The company's default journal used when an employee expense is created."* |
| *"Payment methods available for expenses paid by company"* | `company_expense_allowed_payment_method_line_ids` | outbound payment method lines attached to an existing, active journal of the company | When non-empty it **replaces** the otherwise automatic list of selectable payment methods on an expense |

### 9.2 The selectable payment methods

Computed for each expense, with elevated rights, from its company:

```formula
selectable = the company's allow-list , when that list is not empty
           = every outbound payment method line whose journal belongs to the company
             and is active , otherwise
```

The payment method of the expense defaults to the **first** entry of that list.

### 9.3 Branch companies

An expense created in a branch company produces its entry and its payment in that branch company,
not in the parent. The posting dialogue's journal default searches the parents in order — nearest
first — for a default expense journal, so a branch with no journal of its own inherits the nearest
ancestor's.

---

## 10. The expense mailbox

| External identifier | Local part | Target entity | Contact policy |
|---|---|---|---|
| `mail_alias_expense` | `expense` | the Expense entity | `employees` |

The contact policy `employees` means only senders whose address belongs to an internal user are
accepted; every other message is rejected by the routing layer of
[`../messaging-and-activities/`](../messaging-and-activities/) before this domain sees it.

The record is shipped without an alias domain; the domain is supplied by the setting of §2.1 or by
the company's own alias domain when the alias is created from the settings.

---

## 11. Activity type

| External identifier | Name | Summary | Icon | Target entity |
|---|---|---|---|---|
| `mail_act_expense_approval` | *"Expense Approval"* | *"Expense Approval"* | a currency symbol | the Expense entity |

Scheduled on submission, marked done on approval, removed on reset and on refusal
([`workflows.md`](workflows.md) §3.1).

An Administrator may manage activity types for the Expense entity through the *Activity Types*
window action, which is placed under *Configuration* but is hidden from ordinary users.

---

## 12. Message subtypes

Six subtypes are shipped for the Expense entity, **none** of them subscribed to by default.

| External identifier | Name | Description |
|---|---|---|
| `mt_expense_approved` | *"Approved"* | *"Expense approved"* |
| `mt_expense_refused` | *"Refused"* | *"Expense refused"* |
| `mt_expense_paid` | *"Paid"* | *"Expense paid"* |
| `mt_expense_reset` | *"Draft"* | *"Expense reset to Draft"* |
| `mt_expense_entry_delete` | *"Journal Entry Deleted"* | *"Journal entry deleted"* |
| `mt_expense_entry_draft` | *"Journal Entry Reset to Draft"* | *"Journal entry reset to draft"* |

Which one is broadcast on a given status change is specified in
[`state-machines.md`](state-machines.md) §3.6, including the **compatibility finding** that the
*Refused* subtype is never selected.

---

## 13. Message templates

Four templates are shipped. None of them is a configurable message template a user can edit; each
is a rendering fragment invoked from a fixed point in the flow.

| External identifier | Where it is used | What it renders |
|---|---|---|
| `hr_expense_template_refuse_reason` | posted on each expense when it is refused | *"Your Expense «description» has been refused"* followed by a list holding *"Reason: «the reason»"* |
| `hr_expense_template_register` | the mailbox acknowledgement, employee **with** a user | *"Dear «employee name»,"*; *"Your expense has been successfully registered."*; when the employee has a user, *"You can now submit it to the manager from the following link."*; *"Category: «category name»"* or, when no category was recognised, *"Category: not found"* and *"The first word of the email subject did not correspond to any category code. You'll have to set the category manually on the expense."*; *"Price: «unit price»«currency symbol»"*; and a link labelled *"View Expense"* |
| `hr_expense_template_register_no_user` | the mailbox acknowledgement, employee **without** a user | the same body, wrapped in a framed layout that shows the company's logotype when the company does not use the default one |
| `hr_expense_template_submitted_expenses` | the weekly reminder | heading *"Expenses approval"*; *"Dear «manager name»,"*; *"New expenses are waiting for your approval. You can Review them by following this link."*; a button labelled *"View expenses"*; then the company's name, telephone number, electronic-mail address and website |

Two details of the acknowledgement are worth stating because a rebuild will otherwise differ: the
price it shows is the **unit price**, not the total, and the currency is rendered as a symbol
appended to the number with no space.

---

## 14. Scheduled job

| Name | Runs on | Interval | What it does |
|---|---|---|---|
| *"HR Expense: Send Submitted Expenses Mail"* | the Expense entity | every **1 week** | Sends the approval reminder of [`workflows.md`](workflows.md) §12 to every approver who has submitted expenses waiting |

The job is the only electronic-mail notification the approval chain produces. There is no daily
digest, no escalation and no reminder to the employee.

---

## 15. Digest tip

| External identifier | Sequence | Audience | Text |
|---|---|---|---|
| `digest_tip_hr_expense_0` | 1100 | every internal user | Title *"Tip: Snap pictures of your receipts with the remote app"*; body *"Do not keep your expense tickets in your pockets any longer. Just snap a picture of your receipt and let Odoo digitalizes it for you. The OCR and Artificial Intelligence will fill the data automatically."* with an illustration |

The body is reproduced verbatim, including its grammatical slip and the vendor's name, because it
is a shipped user-visible string. Digest tips themselves belong to
[`../human-resources-core/`](../human-resources-core/).

---

## 16. Onboarding tour

| External identifier | Name | Completion message |
|---|---|---|
| `hr_expense_tour` | `hr_expense_tour` | *"There you go - expense management in a nutshell!"* |

A guided walk-through that creates one expense, attaches a receipt, submits it and approves it. The
tour framework itself belongs to [`../../interfaces/README.md`](../../interfaces/README.md).

---

## 17. The expense dashboard document

| External identifier | Name | Group it appears in | Visible to | Sequence | Published |
|---|---|---|---|---|---|
| `spreadsheet_dashboard_expense` | *"Expenses"* | the finance dashboard group | expense Administrators | 40 | yes |

A shipped spreadsheet document whose main data source is the Expense entity, together with a sample
version used when the installation holds no expense yet. The document itself, and the dashboard
mechanism, belong to
[`../spreadsheets-and-dashboards/`](../spreadsheets-and-dashboards/); what this domain contributes
is the document and the declaration that its main entity is the Expense.

---

## 18. Document layout variant

A localisation capability adds a document-title block to the printable expense document so that it
conforms to the standardised business-letter layout used in one jurisdiction. The block sets the
document title to *"Expenses Report"* when the record being printed is an Expense. It changes
nothing else: the same content, the same figures, the same ordering. The layout itself is specified
in [`../fiscal-localizations/`](../fiscal-localizations/).

---

## 19. Analytic configuration

| Thing | Value |
|---|---|
| Business domain added to the analytic plan applicability | `expense`, labelled *"Expense"*, deleted in cascade with the plan |
| Where it is used | The mandatory-plan validation of rule EXP-ANA-1, and the analytic selector on the expense form and list |
| Account-prefix criterion | Always displayed for an applicability whose business domain is `expense` (rule EXP-EXT-9) |

The analytic selector on the expense form passes two hints to the distribution widget: the category
field and the account field, so that the widget can offer the plans that apply to them; and the
business domain `expense`, so that the mandatory plans are highlighted. The list view passes the
category field and the business domain only.
