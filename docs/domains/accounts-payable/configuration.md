# Accounts Payable — Configuration

---

## 1. Company settings

### 1.1 Payable and bill settings

| Setting (storage name) | Type | Default | Effect |
|---|---|---|---|
| *Expense Account* (`expense_account_id`) | account | none | The account used when validating a vendor bill. Seeds a new purchase journal's default account, and is written into the product category's default expense account when the chart is installed. Its help text notes that under perpetual inventory valuation in the cost-of-goods-sold accounting model the expense is instead recognised at the customer invoice |
| *Price Difference Account* (`price_difference_account_id`) | account | none | Holds the difference between a stockable product's standard price and its billed price under perpetual valuation |
| *Default Purchase Receipt Fiscal Position* (`account_purchase_receipt_fiscal_position_id`) | fiscal position | none | Forced onto every purchase receipt, overriding partner-based detection |
| *Auto-validate bills* (`autopost_bills`) | boolean | **true** | Master switch for automatic posting of vendor bills. When off, no bill is ever posted automatically and the learning wizard never opens |
| *Quick encoding* (`quick_edit_mode`) | selection: `out_invoices` / `in_invoices` / `out_and_in_invoices` | not set | Enables the *Total (Tax inc.)* one-line capture. Purchase journals honour `in_invoices` and `out_and_in_invoices`. It also relaxes the "you cannot delete an entry in the middle of the numbering chain" rule |
| *Separate account for expense discount* (`account_discount_expense_allocation_id`) | account | none | Where discounts taken on purchases are booked |
| *Restricted Audit Trail* (`restrictive_audit_trail`) | boolean, tracked | false | Prevents deleting the change log, and therefore prevents deleting any document that has ever been posted |
| *Forced Audit Trail* (`force_restrictive_audit_trail`) | boolean, computed | false | When a companion package forces the trail, the setting is hidden and cannot be switched off |
| Fiscal year end day and month (`fiscalyear_last_day`, `fiscalyear_last_month`) | integer | 31 / 12 | Shape the numbering series of a staggered year (see §3.1) |

### 1.2 Cheque printing settings

Shown under *Accounting → Configuration → Settings*, in the **Print Checks** block, and visible only when the cheque printing package is installed.

| Setting (storage name) | Type | Default | Effect |
|---|---|---|---|
| *Check Layout* (`account_check_printing_layout`) | selection | `disabled` (*None*) | The printable-document definition that draws the cheque. The base package ships **only** the value `disabled`; each country package adds one value per stationery it supports, and the value is literally the identifier of that printable document. `disabled` switches cheque printing off. **Required** in the settings form when the block is visible |
| *Multi-Pages Check Stub* (`account_check_printing_multi_stub`) | boolean | false | Continue the stub over several pages instead of cropping it to nine lines |
| *Check Top Margin* (`account_check_printing_margin_top`) | float | 0.25 | Printer alignment |
| *Check Left Margin* (`account_check_printing_margin_left`) | float | 0.25 | Printer alignment |
| *Check Right Margin* (`account_check_printing_margin_right`) | float | 0.25 | Printer alignment. Shown only when the company's country code is `CA` |
| *Print Date Label* (`account_check_printing_date_label`) | boolean | true | Print the date caption; disable when the stationery already carries it. Shown only when the company's country code is `CA` |

### 1.3 Intercompany clearing settings

*(Intercompany Payment Clearing package.)* The three settings sit together in one block of the settings form, inside the default-accounts group. The block is titled **Intercompany Clearing** and is **shown only to members of the multi-company group**; a single-company installation never sees it, and the mechanism therefore stays unconfigured there.

| Setting (storage name) | Label shown in the block | Type | Domain | Effect |
|---|---|---|---|---|
| *Intercompany Clearing Journal* (`account_interco_clearing_journal_id`) | **Journal** | journal | type is `general` | Where the two clearing entries are booked. Its presence is what enables the whole mechanism |
| *Intercompany Clearing Payable Account* (`account_interco_payable_id`) | **Account Payable** | account | account type is payable **and** the account is reconcilable | Where a bill paid by a sister company is cleared |
| *Intercompany Clearing Receivable Account* (`account_interco_receivable_id`) | **Account Receivable** | account | account type is receivable **and** the account is reconcilable | The mirror on the paying company's side |

Three further rules govern the block:

- **Required-ness.** Both account pickers become **required as soon as a clearing journal is chosen**, and are optional while it is empty. So the configuration is all-or-nothing: either no clearing journal and no accounts, or a journal together with both accounts.
- **Ordering hint.** Both account pickers are opened with the *sort by non-trade* hint, which lists the non-trade payable and non-trade receivable accounts first — those are the accounts a clearing arrangement normally uses, rather than the ordinary trade accounts.
- **Company check.** On the settings form all three pickers are company-checked against the company being configured. On the Company entity itself only the clearing **journal** declares a company check; the two account fields declare their account-type-and-reconcilable domains and nothing else (see `entities.md` §14). A rebuild that writes the accounts through the settings form gets the check; one that writes them directly onto the company does not.

---

## 2. Partner settings

| Setting (storage name) | Type | Default | Scope | Effect |
|---|---|---|---|---|
| *Account Payable* (`property_account_payable_id`) | account, restricted to payable accounts | none | per company | The second candidate for a payable term line's account |
| *Vendor Payment Terms* (`property_supplier_payment_term_id`) | payment term | none | per company | The default payment term of a purchase document |
| *Fiscal Position* (`property_account_position_id`) | fiscal position | none | per company | Remaps taxes and accounts |
| *Auto-post bills* (`autopost_bills`) | selection, **required** | `ask` | global | `always` — post this vendor's bills without review; `ask` — offer to automate after three untouched validations; `never` — never offer and never post automatically |
| (`ignore_abnormal_invoice_date`) | boolean | false | per company | Silences the abnormal-billing-frequency warning for this vendor |
| (`ignore_abnormal_invoice_amount`) | boolean | false | per company | Silences the abnormal-amount warning for this vendor |
| (`property_outbound_payment_method_line_id`) | payment method line, restricted to active journals and outbound methods of the company or its parents | none | per company | Steers the payment method the register proposes |
| `supplier_rank` | integer | 0 | global | Incremented at each posted purchase document, for the partner and for its commercial partner. Orders vendor pickers |

---

## 3. Sequences and numbering

### 3.1 The document number

A purchase document's number is produced by the journal's numbering series, whose grammar is specified in `../general-ledger/`. Two things are payable-specific.

**The starting pattern.** When a journal has no number yet, the pattern is derived as follows. Let the reference date be the accounting date, falling back to the bill date, falling back to today.

1. The **year part** is the four-digit year, unless the company's fiscal year does not end on 31 December: then it is a two-digit range. If the reference date is **after** the fiscal year end of its own calendar year, the range is `«this year, two digits»-«next year, two digits»`; otherwise it is `«previous year, two digits»-«this year, two digits»`. The fiscal end day is clamped to the length of the fiscal end month.
2. For a **sale, bank, cash or credit-card** journal the series is **annual**:
   `«journal code»/«year part»/00000`, or `.../0000` when the year is staggered (to keep the number short).
3. For every other journal — **including every purchase journal** — the series is **monthly**:
   `«journal code»/«year part»/«two-digit month»/0000`.
4. When the journal is a **self-billing** journal, the code is followed by the commercial partner's identifier padded with zeros to five characters (or the literal text `[Partner id]` when no partner is set):
   `«journal code»«partner identifier»/«year part»/«two-digit month»/0000`.
5. When the journal has a **separate refund sequence** and the document is a credit note (`out_refund` or `in_refund`), the pattern is prefixed with `R`.
6. When the journal has a **separate payment sequence** and the document is a payment's entry, the pattern is prefixed with `P`.
7. *(Debit Notes package.)* When the journal has a **dedicated debit note sequence** and the document is a bill or a customer invoice carrying a debit origin, the pattern is prefixed with `D`.

So a purchase journal coded `BILL` produces, for January 2026: bills `BILL/2026/01/0001`, credit notes `RBILL/2026/01/0001`, debit notes `DBILL/2026/01/0001`.

**The "last number" search.** When looking for the highest number already used, the search is narrowed:

- when the journal has a separate refund sequence: to credit notes only, or to non-credit-notes only, according to the document at hand;
- otherwise, when it has a separate payment sequence: to payments' entries only, or to non-payments only;
- when the journal is self-billing: to the same commercial partner (and to nothing at all when no partner is set);
- *(Debit Notes package.)* when the journal has a dedicated debit note sequence: to documents that likewise do, or do not, carry a debit origin.

An **anti-pattern** is also applied so that a yearly series does not accidentally match a monthly one and vice versa; the table of which regular expression may match which format is part of the numbering grammar in `../general-ledger/`.

### 3.2 The journal's mail alias

The local part of a purchase journal's reception address is derived as:

1. The first of these that is encodable and sanitisable: the current alias name, the journal name, the journal code, the literal word matching the journal type. A journal that is neither sale nor purchase gets **no** alias.
2. When the journal's company is **not** the main company, a company identifier is appended as `-«identifier»` unless already present. The identifier is the sanitised company name when that is encodable, and the company's numeric identifier otherwise.
3. The result is sanitised into a valid address local part.
4. At creation time, uniqueness within the alias domain is enforced by appending `-«journal code»` when an alias with that name already exists in the same domain (or in no domain).

The alias's defaults are: company = the journal's company; document type = `in_invoice` for a purchase journal, `out_invoice` for a sale journal, `entry` otherwise; journal = the journal itself.

### 3.3 The cheque number sequence

One sequence per journal, created automatically for every journal that has none — including, at installation time, every existing **bank** journal.

| Attribute | Value |
|---|---|
| Name | *«journal name»: Check Number Sequence* |
| Implementation | **no gap** |
| Padding | 5 |
| Increment | 1 |
| Company | the journal's company |

The padding is overwritten whenever a number is written explicitly (see `calculations.md` §13).

### 3.4 The payment sequence

A global sequence named *Payment*, code `account.payment`, prefix `PAY`, padding 5, belonging to no company. It numbers payment records themselves, not their journal entries.

---

## 4. Default and shipped records

| Record | Shipped by | Content |
|---|---|---|
| Cheque payment method | Check Printing Base | Name *Checks*, code `check_printing`, payment type **outbound**, mode `multi`, allowed journal type **bank** |
| *Print Checks* server action | Check Printing Base | Bound to the payment entity, offered on list and kanban views, restricted to the accountant group; its effect is to call the printing operation on the selection |
| *Confirm Entries* server action | Accounting | Bound to the journal entry, offered on list and kanban views, restricted to the invoicing group; its effect is to call the confirmation-with-dialogue operation |
| *Review Entries* server action | Accounting | Bound to the journal entry, offered on list and kanban views, restricted to the accountant group; its effect is to call the *Check selected* operation on the entity, which reads the documents named in the context and marks them reviewed |
| *Switch into invoice/credit note* server action | Accounting | Bound to the journal entry, offered on the form view, restricted to the invoicing group; its effect is to call the switch-type operation on the selection, and nothing when the selection is empty |
| *Pay* server action | Accounting | Bound to the journal entry, offered on the form view, restricted to the invoicing group; its effect is to call the force-register-payment operation on the selection, and nothing when the selection is empty |
| *(Un)Block Payment* server action | Accounting | Bound to the journal entry, offered on the form view, restricted to the invoicing group; its effect is to call the toggle-payment-block operation on the selection |
| *Share* server action | Accounting | Bound to the journal entry, offered on the form view, restricted to **no** group; its effect is to call the share operation, which produces a shareable link to the document |
| *Create Debit Note* action | Debit Notes | Bound to the journal entry, offered on list and kanban views, restricted to no group; it opens the *Create Debit Note* dialogue, which itself refuses sources that may not be debited (`business-rules.md` §11) |
| Cheque sequences | Check Printing Base, at installation | One per existing bank journal |
| The ten payment terms | Accounting | Enumerated in §4.1 |
| Product name similarity threshold | Accounting | A system parameter keyed `account.product_name_similarity_threshold` with value `0.9`, used when a decoder matches a product by name |

### 4.1 The payment terms shipped by the Accounting package

Ten payment terms are created by the Accounting package itself, independently of any chart of accounts. They are ordinary records: a user may rename, extend, archive or delete them, and a purchase document may use any of them as the vendor payment term. The entity and its four delay types are specified in `entities.md` §7 and §8; the distribution arithmetic is in `calculations.md` §5.

Every line below is a **percent** line. Unless a delay type is named, the line uses `days_after` (*Days after invoice date*), so its due date is the bill date plus the day count.

| External identifier | Name | Instalment lines | Early discount |
|---|---|---|---|
| `account_payment_term_immediate` | *Immediate Payment* | one line: 100 %, 0 days | none |
| `account_payment_term_15days` | *15 Days* | one line: 100 %, 15 days | none |
| `account_payment_term_21days` | *21 Days* | one line: 100 %, 21 days | none |
| `account_payment_term_30days` | *30 Days* | one line: 100 %, 30 days | none |
| `account_payment_term_45days` | *45 Days* | one line: 100 %, 45 days | none |
| `account_payment_term_end_following_month` | *End of Following Month* | one line: 100 %, delay type `days_after_end_of_next_month` (*Days after end of next month*), 0 days | none |
| `account_payment_term_30_days_end_month_the_10` | *10 Days after End of Next Month* | one line: 100 %, delay type `days_after_end_of_next_month`, 10 days | none |
| `account_payment_term_advance_60days` | *30% Now, Balance 60 Days* | two lines, in this order: 30 % at 0 days, then 70 % at 60 days. The second is the balance line and absorbs any rounding drift | none |
| `account_payment_term_30days_early_discount` | *2/7 Net 30* | one line: 100 %, 30 days | **on**: 2 %, within 7 days. The term is also flagged to be shown on the printed document |
| `account_payment_term_90days_on_the_10th` | *90 days, on the 10th* | one line: 100 %, delay type `days_end_of_month_on_the` (*Days end of month on the*), 90 days, day of the following month 10 | none |

Each of the ten carries a **note**, the free text reproduced on a document that uses the term. For nine of them the note is the words *Payment terms:* followed by a space and the term's own name — *Payment terms: Immediate Payment*, *Payment terms: 15 Days*, *Payment terms: 21 Days*, *Payment terms: 30 Days*, *Payment terms: 45 Days*, *Payment terms: End of Following Month*, *Payment terms: 10 Days after End of Next Month*, *Payment terms: 30% Now, Balance 60 Days* and *Payment terms: 90 days, on the 10th*. The tenth departs from the pattern: the note of *2/7 Net 30* reads *Payment terms: 30 Days, 2% Early Payment Discount under 7 days*.

The mode of the early payment discount on *2/7 Net 30* is not part of the shipped record: the term takes the default computation mode of its entity (`entities.md` §7.2), and the three modes and their consequences are specified in `calculations.md` §5.3.

---

## 5. Security groups and access rights

### 5.1 The groups

| Group | Label | Implies | Meaning in this domain |
|---|---|---|---|
| `account.group_account_invoice` | Invoicing | the internal-user group | Create, read, update and delete purchase documents and their lines; run the reversal, debit note, confirm-entries and autopost wizards; register payments. Cannot see journal entries as accounting objects, reports or reconciliation |
| `account.group_account_readonly` | Show Accounting Features - Readonly | the internal-user group | Read everything, including journal entries, the analysis report and advanced configuration |
| `account.group_account_basic` | Basic | Invoicing | Additional accounting features short of journal entries |
| `account.group_account_user` | Show Full Accounting Features | Basic and Read-only | The accountant: everything except advanced configuration. Gates the *Review* buttons and the *Print Checks* action |
| `account.group_account_manager` | Administrator | Invoicing | Full access including configuration; may override the numbering pattern and delete numbered entries in the middle of a chain |
| `account.group_account_secured` | Show Inalterability Features | — | Shows the secured status widget |
| `account.group_cash_rounding` | Allow the cash rounding management | — | Shows the cash rounding fields |
| `account.group_partial_purchase_deductibility` | Partial Purchase Deductibility | — | Shows the deductibility column on bill lines. **Granted automatically** to any user who posts a vendor bill carrying a line whose deductibility is not 100 |
| `account.group_validate_bank_account` | Validate bank account | implied by the system administrator group | May mark a bank account as trusted for outgoing payments |
| `account.group_delivery_invoice_address` | Delivery Address | — | Shows the delivery address on documents |

### 5.2 Access rights matrix

| Entity | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| Journal Entry (`account.move`) | Invoicing | yes | yes | yes | yes |
| Journal Entry | Read-only Accounting | no | yes | no | no |
| Journal Entry | Administrator | no | yes | no | no |
| Journal Entry | Portal | no | yes | no | no |
| Journal Item (`account.move.line`) | Invoicing | yes | yes | yes | yes |
| Journal Item | Read-only Accounting | no | yes | no | no |
| Journal Item | Administrator | no | yes | no | no |
| Journal Item | Portal | no | yes | no | no |
| Invoice Analysis Report (`account.invoice.report`) | Read-only Accounting | no | yes | no | no |
| Invoice Analysis Report | Invoicing | no | yes | no | no |
| Invoice Analysis Report | Administrator | no | yes | no | no |
| Payment Term (`account.payment.term`) | any internal user | no | yes | no | no |
| Payment Term | Portal | no | yes | no | no |
| Payment Term | Administrator | yes | yes | yes | yes |
| Payment Term Line (`account.payment.term.line`) | any internal user | no | yes | no | no |
| Payment Term Line | Administrator | yes | yes | yes | yes |
| Confirm Entries Wizard (`validate.account.move`) | Invoicing | yes | yes | yes | no |
| Autopost Bills Wizard (`account.autopost.bills.wizard`) | Invoicing | yes | yes | yes | no |
| Reversal Wizard (`account.move.reversal`) | Invoicing | yes | yes | yes | no |
| Add Debit Note Wizard (`account.debit.note`) | Invoicing | yes | yes | yes | no |
| Print Pre-numbered Cheques Wizard (`print.prenumbered.checks`) | Accountant | yes | yes | yes | no |

Note the deliberate asymmetry: the **Administrator** group alone has only read access on documents; it gains write access through the Invoicing group it implies.

### 5.3 Record rules

| Rule | Entity | Domain | Groups | Permissions |
|---|---|---|---|---|
| Account Entry | Journal Entry | company is among the user's allowed companies | all | all four |
| Entry lines | Journal Item | company is among the user's allowed companies | all | all four |
| All Journal Entries | Journal Entry | always true | Invoicing | all four |
| All Journal Items | Journal Item | always true | Invoicing | all four |
| Portal Personal Account Invoices | Journal Entry | status is neither cancelled nor draft, **and** the type is one of `out_invoice`, `out_refund`, `in_invoice`, `in_refund`, **and** the partner is the portal user's commercial partner or one of its children | Portal | all four (read is the only right granted) |
| Portal Invoice Lines | Journal Item | the parent status is neither cancelled nor draft, the document type is one of the same four, and the document partner is the portal user's commercial partner or one of its children | Portal | all four |
| Invoice Analysis multi-company | Invoice Analysis Report | company is among the user's allowed companies | all | all four |
| Journal multi-company | Journal | the journal's company is the user's company or a parent of it | all | all four |
| Account multi-company | Account | the account's companies include the user's company or a parent of it | all | all four |
| Account payment term company rule | Payment Term | the term has no company, or its company is the user's company or a parent of it | all | all four |
| Account payment company rule | Payment | company is among the user's allowed companies | all | all four |

Consequence for a **portal** user: a supplier with portal access can read the vendor bills whose partner is their own commercial entity, together with their lines, but nothing else.

---

## 6. Scheduled jobs

| Job | Model | Interval | First run | What it does |
|---|---|---|---|---|
| *Account: Post draft entries with auto_post enabled and accounting date up to today* | Journal Entry | every **1 day** | the next day at 02:00 | Searches for documents in `draft` status whose accounting date is on or before today and whose automatic posting mode is not `no`, limited to **100** per run. It first tries to post the whole batch at once; if any of them raises a user error, it rolls back and posts them one by one so that a single bad document does not block the rest. It reports its remaining workload to the scheduler so that the batch is resumed |
| *Send invoices automatically* | Journal Entry | every **1 day**, run as the root user | — | Sends the documents queued for asynchronous sending. Concerns customer documents; listed here because a self-billing purchase journal also sends |

### 6.1 The recurrence copy

Posting a document whose automatic posting mode is periodic (`monthly`, `quarterly`, `yearly`) copies the **next** occurrence, before the status changes:

1. The document becomes its own recurrence origin if it has none.
2. The next accounting date is the current one shifted by the period, measured against the **origin's** date so that the series does not drift.
3. If the recurrence has an end date and the next date is after it, nothing is copied.
4. If an occurrence with that origin already exists on that date, nothing is copied.
5. Otherwise the document is copied with: the same automatic posting mode, the same end date, the same origin, the same salesperson (otherwise the copy would be attributed to the automation user), the new accounting date, the bill date shifted by the same rule, and — when the document has **no** payment term — a due date preserving the original gap between the due date and the accounting date.

Resetting a document to draft **deletes** the next occurrence when it is still draft.

---

## 7. Journal configuration for payables

### 7.1 A purchase journal

| Field | Guidance |
|---|---|
| Type | `purchase` |
| Code | Defaults to `BILL` followed by a number, chosen so that no active or archived journal of the company already uses it; the search stops at 99 |
| Default Account | Seeded from the company's expense account; when that is absent, from the product category's default expense account fallback |
| Private Share Account (`non_deductible_account_id`) | Where the non-deductible part of mixed expenses lands. Falls back to the default account |
| Dedicated Credit Note Sequence (`refund_sequence`) | Defaults to **true** for purchase journals |
| Dedicated Debit Note Sequence (`debit_sequence`) | *(Debit Notes package.)* Defaults to **true** for purchase journals |
| Self Billing (`is_self_billing`) | When set, numbering uses a separate series per partner and the self-billing mail templates are used |
| Alias (`alias_name`) | The reception address; see §3.2. Forced empty when the type is neither sale nor purchase |
| Secure Posted Entries with Hash (`restrict_mode_hash_table`) | When set, posting hashes the chain; such a journal is excluded from mass posting unless *Force Hash* is ticked, and its documents can never be reset to draft |
| Sequence override pattern (`sequence_override_regex`) | Overrides the numbering grammar; only an accounting manager may write a number that does not match it |

A journal of type `purchase` may not hold a sale document and vice versa (see `business-rules.md` §6).

### 7.2 A bank journal used for cheques

| Field | Guidance |
|---|---|
| Outgoing payment methods | Must include **Checks**. It is added automatically to every new journal for which the method is available |
| Manual Numbering (`check_manual_sequencing`) | On for blank stationery, off for pre-printed stationery. Visible only when the cheque method is among the journal's outgoing methods and the type is `bank` |
| Next Check Number (`check_next_number`) | Visible only when manual numbering is on. Writing it moves the sequence; see `calculations.md` §13.1 |
| Check Layout (`bank_check_printing_layout`) | Per-journal override of the company layout; the choices are the company's layout list minus `disabled`. Its placeholder reads *Default* |
| Check Sequence (`check_sequence_id`) | Read-only, created automatically |

---

## 8. Decimal precisions used

| Precision name | Used for |
|---|---|
| *Product Price* | the unit price of a line |
| *Product Unit* | the quantity of a line |
| *Discount* | the discount percentage of a line |
| *Payment Terms* | the percentage of a payment term line, and the "sums to 100" constraint |

---

## 9. System parameters

| Key | Default | Purpose |
|---|---|---|
| `account.product_name_similarity_threshold` | `0.9` | The similarity above which a decoder accepts a product name match when importing a supplier document |
| `account.use_invoice_terms` | not set by this domain | When set, the terms and conditions text is copied onto **sale** documents; purchase documents are unaffected |

---

## 10. Onboarding and the empty-state offers

There is **no onboarding panel for a purchase journal.** The dashboard maps journal types to onboarding panels as follows:

| Journal type | Onboarding panel |
|---|---|
| `sale` | the invoice onboarding |
| `general` | the dashboard onboarding, whose steps are *Set Company Data*, *Set Periods* and *Review Chart of Accounts* |
| every other type, **including `purchase`** | none |

What a purchase journal card offers instead, when it holds no document at all, is:

1. the **upload zone**, with the image of a bill and the text *Drop and let the AI process your bills automatically.*;
2. the **sample bill** offer of `workflows.md` §20, which is hidden when the demonstration data is not installed — the availability test is simply whether the demonstration partner record exists.

The other onboarding steps shipped by the accounting package — *Set Company Data*, *Documents Layout*, *Set Periods*, *Review Chart of Accounts*, *Taxes* — are shared configuration and are specified in `../general-ledger/configuration.md`.

---

## 11. Installation-time behaviour

| Package | Hook | Effect |
|---|---|---|
| Check Printing Base | after installation | Creates a cheque sequence on **every** existing bank journal |
| Check Printing Base | on journal creation | Creates a cheque sequence on every newly created journal that has none, whatever its type |
| Check Printing Base | on journal creation | Adds the cheque method to the journal's default outgoing methods when the method is available for it |
| Accounting | on the payment record | A dedicated database column for the cheque number is created before the stored computed field is evaluated, so that installing on a large database does not exhaust memory |
