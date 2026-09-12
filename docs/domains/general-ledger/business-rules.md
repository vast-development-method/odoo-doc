# General Ledger — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behavior of the domain, with the exact user-facing message. Placeholders in the messages are written in words between asterisks.

Contents:

1. [Invariants](#1-invariants)
2. [Account rules](#2-account-rules)
3. [Account Group rules](#3-account-group-rules)
4. [Account Tag rules](#4-account-tag-rules)
5. [Journal rules](#5-journal-rules)
6. [Journal Entry rules](#6-journal-entry-rules)
7. [Journal Item rules](#7-journal-item-rules)
8. [Numbering rules](#8-numbering-rules)
9. [Locking rules](#9-locking-rules)
10. [Hashing rules](#10-hashing-rules)
11. [Audit trail rules](#11-audit-trail-rules)
12. [Reconciliation rules](#12-reconciliation-rules)
13. [Deletion rules](#13-deletion-rules)
14. [Permission checks](#14-permission-checks)
15. [Company rules](#15-company-rules)
16. [Wizard rules](#16-wizard-rules)
17. [Edge cases](#17-edge-cases)
18. [Field-level write protection](#18-field-level-write-protection)
19. [Concurrency and transactional rules](#19-concurrency-and-transactional-rules)
20. [Rules that other domains rely on](#20-rules-that-other-domains-rely-on)
21. [Rules this domain contributes to the Contact](#21-rules-this-domain-contributes-to-the-contact)

---

## 1. Invariants

These statements must hold at every commit. They are the properties an implementation is judged on.

| # | Invariant |
|---|---|
| I1 | For every Journal Entry, the sum of the balances of its items, rounded to the company currency, is zero. |
| I2 | For every Journal Item that is not a section, a subsection or a note, exactly one of the debit and the credit is non-zero, or both are zero. Formally the product of the two is zero. |
| I3 | For every Journal Item that is not a section, a subsection or a note, the balance and the foreign amount have the same sign or at least one of them is zero. |
| I4 | Every Journal Item that is not a section, a subsection or a note has an account. |
| I5 | Every section, subsection or note has no account, a zero debit, a zero credit and a zero foreign amount. |
| I6 | No two posted Journal Entries of the same journal carry the same number, unless the number is the placeholder. |
| I7 | An Account belongs to at least one company, and has a code in each of the root companies of those companies. |
| I8 | No two Accounts carry the same code in a company or in any ancestor or descendant of it. |
| I9 | An Account of type Receivable or Payable allows reconciliation. |
| I10 | An Account of type Off-Balance Sheet allows neither reconciliation nor taxes. |
| I11 | The three amounts of a Partial Reconciliation are positive. |
| I12 | The residual of an item equals its amount minus the matched amounts, in each of the two currencies. |
| I13 | An item has a matching number if and only if it participates in at least one match, or carries an imported label. |
| I14 | An item with a Full Reconciliation has that reconciliation identifier as its matching number, and a zero residual in both currencies. |
| I15 | A hashed entry never changes its number, its date, its journal, its company, nor the label, debit, credit, account or partner of any of its items. |
| I16 | A hashed entry is never deleted, never reset to draft, and none of its items is ever deleted. |
| I17 | The Hard Lock Date of a company never decreases and is never cleared. |
| I18 | No posted entry is dated on or before the Hard Lock Date of its company at the moment of posting. |
| I19 | An entry of a sale journal is a sale document or a plain entry; an entry of a purchase journal is a purchase document or a plain entry. |
| I20 | Every item of an entry with an off-balance account is on an off-balance account. |

---

## 2. Account rules

### Creation and modification

| Rule | Condition | Message |
|---|---|---|
| Code characters | The code may contain only letters, digits and dots | "The account code can only contain alphanumeric characters and dots. (account code: *the code*)" |
| Code present | Every root company of the companies of the account must have a code | "The code must be set for every company to which this account belongs." |
| Code unique | No account of a parent or a child company may carry the same code | "Account codes must be unique. You can't create accounts with these duplicate codes: *the codes, separated by commas*" |
| At least one company | The list of companies must not be empty | "The following accounts must be assigned to at least one company:" followed by one line per account, prefixed with "- " and carrying its display name |
| Liquidity accounts are single-company | An account of type Bank and Cash may belong to one company only | "Bank & Cash accounts cannot be shared between companies." |
| Company removal | A company may be detached only while no journal item of it or of a descendant uses the account | "You can't unlink this company from this account since there are some journal items linked to it." |
| Receivable and payable reconcile | An account of type Receivable or Payable must allow reconciliation | "You cannot have a receivable/payable account that is not reconcilable. (account code: *the code*)" |
| Off-balance does not reconcile | An off-balance account may not allow reconciliation | "An Off-Balance account can not be reconcilable" |
| Off-balance carries no tax | An off-balance account may not have default taxes | "An Off-Balance account can not have taxes" |
| Journal currency coherence | When a journal has a foreign currency different from the company currency, its default account, and the outstanding accounts of its money-in and money-out methods, must carry that same currency | "The foreign currency set on the journal '*the journal display name*' and the account '*the account display name*' must be the same." |
| Sale and purchase default account | An account used as the default account of a sale or purchase journal may not become Receivable or Payable | "The account is already in use in a 'sale' or 'purchase' journal. This means that the account's type couldn't be 'receivable' or 'payable'." |
| Bank account of a journal | An account used as the default account of a journal may not become Receivable or Payable | "You cannot change the type of an account set as Bank Account on a journal to Receivable or Payable." |
| Currency change | A currency may be set only when no journal item on the account already carries a different foreign currency | "You cannot set a currency on this account as it already has some journal entries having a different foreign currency." |
| Reconciliation switched off | Reconciliation may be switched off only when no partial reconciliation is pending on the account | "You cannot switch an account to prevent the reconciliation if some partial reconciliations are still pending." |
| Deprecation | An account used in a tax distribution may not be deprecated | "You cannot deprecate an account that is used in a tax distribution." |
| Name creation outside the chart screen | Creating an account by typing a name into a selection field is refused unless the creation comes from a file import | "Please create new accounts from the Chart of Accounts menu." |
| Generic record merge | The generic record-merge entry point, the one that merges arbitrary records of any kind, always refuses on accounts | "You cannot merge accounts." |
| Code generation exhausted | No free code could be found | "Cannot generate an unused account code." |

### Deletion and archival

| Rule | Message |
|---|---|
| The account carries journal items | "You cannot perform this action on an account that contains journal items." |
| The account is used in a fiscal position account mapping | "You cannot remove/deactivate the accounts "*the codes and names*" which are set on the account mapping of a fiscal position." |
| The account is used in a tax distribution line | "You cannot remove/deactivate the accounts "*the codes and names*" which are set on a tax repartition line." |

### Switching reconciliation on and off

Switching the flag **on** rewrites, for every item on the account that has no Full Reconciliation: the reconciled flag becomes true when the debit, the credit and the foreign amount are all zero and false otherwise; the residual in the company currency becomes the debit minus the credit; the residual in the item currency becomes the foreign amount.

Switching the flag **off** first refuses the operation when a pending partial reconciliation exists, then sets both residuals of every item without a Full Reconciliation to zero.

Accounts **can** be merged, but only through the dedicated account merge wizard, which applies its own preconditions and its own per-account blocking reasons; the refusal quoted above guards only the generic entry point, which the wizard does not use. The wizard is specified in `entities.md`, its algorithm in `workflows.md` and its rules in section 16 of this document.

### Unmerging

| Rule | Message |
|---|---|
| The user must have write access | the ordinary access error |
| The user must see every company of the account | "You do not have the right to perform this operation as you do not have access to the following companies: *the company names*." |
| The account must belong to more than one company | "Account *the account display name* cannot be unmerged as it already belongs to a single company. The unmerge operation only splits an account based on its companies." |

Before acting, a confirmation is shown: "Are you sure? This will perform the following operations:" followed, for each account, by "Account *the display name* will be split in *the number of companies*, one for each company:" and one indented line per company giving the company name and the display name the account has in it.

---

## 3. Account Group rules

| Rule | Message |
|---|---|
| The start prefix and the end prefix must have the same number of characters | "The length of the starting and the ending code prefix must be the same" |
| Two groups of the same company and the same prefix length may not overlap | "Account Groups with the same granularity can't overlap" |
| The parent chain may not form a cycle | "You cannot create recursive groups." |

Writing an empty end prefix together with a non-empty start prefix silently drops the empty value, and symmetrically, so that a single prefix can be typed into either box.

---

## 4. Account Tag rules

| Rule | Message |
|---|---|
| The triple (name, applicability, country) is unique | "A tag with the same name and applicability already exists in this country." |
| The three cash-flow tags shipped as reference data may not be deleted | "You cannot delete this account tag (*the tag name*), it is used on the chart of account definition." |
| A tag may not be deleted while accounts or journal items reference it | the ordinary restriction error of the database |

---

## 5. Journal rules

| Rule | Condition | Message |
|---|---|---|
| Code unique | The pair (company, code) is unique | "Journal codes must be unique per company." |
| Bank account company | The bank account of a bank journal must belong to the company of the journal | "The bank account of a bank journal must belong to the same company (*the company name*)." |
| Bank account holder | The holder of the bank account of a bank journal must be the partner of the company | "The holder of a journal's bank account must be the company (*the company name*)." |
| Bank account on write | Assigning a bank account whose holder is not the company partner | "The partners of the journal's company and the related bank account mismatch." |
| Company change | The company may not change while entries exist | "You can't change the company of your journal since there are some journal entries linked to it." |
| Default account type | The default account of a sale or purchase journal may not be Receivable or Payable | "The type of the journal's default credit/debit account shouldn't be 'receivable' or 'payable'." |
| Payment method duplication | Two lines of the same journal and the same direction may not share a unique or electronic method and a name | "You can't have two payment method lines of the same payment type (*inbound* or *outbound*) and with the same name (*the line name*) on a single journal." |
| Payment method uniqueness across journals | A unique method may be attached to one journal per company; an electronic method to one journal per company and provider | "Some payment methods supposed to be unique already exists somewhere else.\n(*the method display names*)" |
| Archiving | A journal holding draft entries may not be archived | "You can not archive a journal containing draft journal entries.\n\nTo proceed:\n1/ go to Accounting > Accounting > Journal Entries\n2/ filter on this journal and on 'Unposted' entries\n3/ select them all and post or delete them through the action menu" |
| Switching hashing off | Hashing may not be switched off once entries of the journal carry a hash | "You cannot modify the field *the field label* of a journal that already has accounting entries." |
| Code generation on import | The journal name must allow a free code to be derived | "Cannot generate an unused journal code. Please change the name for journal *the journal name*." |
| Code generation on duplication | A free code must exist | "Could not compute any code for the copy automatically. Please create it manually." |
| No journal of the needed type | Looking up a journal for a document | "No journal could be found in company *the company display name* for any of those types: *the types, separated by commas*" |
| Upload without a journal | Creating documents from files with no journal in the context and an ambiguous document type | "The journal in which to upload the invoice is not specified. " |
| Upload without a file | Creating documents from files with an empty selection | "No attachment was provided" |

Deleting a journal deletes its payment method lines and, when no other journal points to the same bank account, that bank account.

---

## 6. Journal Entry rules

### Structural rules

| Rule | Message |
|---|---|
| Balance (one entry) | "The entry is not balanced." |
| Balance (several entries) | "The following entries are unbalanced:" then one line per entry: two spaces, a hyphen, a space and the entry number |
| A company is mandatory | "We can't leave this document without any company. Please select a company for this document." |
| A purchase document needs a purchase journal | "Cannot create a purchase document in a non purchase journal" |
| A sale document needs a sale journal | "Cannot create a sale document in a non sale journal" |
| The currency rate of a foreign-currency document must be strictly positive | "The currency rate must be strictly positive." |
| A document scheduled for automatic posting and of the purchase family needs a document date | "For this entry to be automatically posted, it required a bill date." |
| Creating an entry directly in the posted state | "You cannot create a move already in the posted state. Please create a draft move and post it after." |
| Tax country coherence, with a fiscal position | "This entry contains taxes that are not compatible with your fiscal position. Check the country set in fiscal position and in your tax configuration." |
| Tax country coherence, without a fiscal position | "This entry contains one or more taxes that are incompatible with your fiscal country. Check company fiscal country in the settings and tax country in taxes configuration." |

### Rules on a posted entry

| Rule | Message |
|---|---|
| A posted entry may not have its lines, document date, accounting date, partner, payment terms, currency, fiscal position or cash-rounding method modified | "You cannot modify the following readonly fields on the posted move *the number or the reference or the identifier*: *the field names*" |
| A reviewed entry may be sent back to draft only by a user allowed to review | "Validated entries can only be changed by your accountant." |
| Marking an entry as reviewed requires the same permission | "You don't have the access rights to perform this action." |
| Changing the number or the date of a posted entry re-checks the lock dates and the tax lock date | the lock messages of section 9 |
| Leaving the posted state re-checks the lock dates and the tax lock date | the lock messages of section 9 |

### Rules on posting

Listed in `state-machines.md`; the messages are repeated here for completeness.

| Rule | Message |
|---|---|
| Permission | "You don't have the access rights to post an invoice." |
| Already posted or cancelled | "The entry *the number* (id *the identifier*) must be in draft." |
| No accountable item | "Even magicians can't post nothing!" |
| Future date without soft mode | "This move is configured to be auto-posted on *the date*" |
| Archived journal | "You cannot post an entry in an archived journal (*the journal display name*)" |
| Inactive currency | "You cannot validate a document with an inactive currency: *the currency name*" |
| Archived account | "A line of this move is using a archived account, you cannot post it." |
| Account of another company | "The entry is using accounts (*the account display names*) from a different company." |
| Archived analytic account | "You cannot post an entry with an archived analytic account: *the analytic account names*" |

All of these are gathered and raised together, one per line.

### Rules on state changes

| Rule | Message |
|---|---|
| Reset to draft from another state | "Only posted/cancelled journal entries can be reset to draft." |
| Reset to draft of a document requiring an approved cancellation | "You can't reset to draft those journal entries. You need to request a cancellation instead." |
| Reset to draft of an exchange-difference entry | "You cannot reset to draft an exchange difference journal entry." |
| Reset to draft of a cash-basis entry | "You cannot reset to draft a tax cash basis journal entry." |
| Reset to draft of a hashed entry | "You cannot reset to draft a locked journal entry." |
| Cancellation of an entry that is not draft after the reset | "Only draft journal entries can be cancelled." |
| Requesting a cancellation for a document that does not need one | "You can only request a cancellation for invoice sent to the government." |

### Rules on other operations

| Rule | Message |
|---|---|
| Switching a document between invoice and credit note when a number was consumed | "You cannot switch the type of a document with an existing sequence number." |
| Switching the type of a plain entry | "This action isn't available for this document." |
| Registering a payment on a document that is not posted | "You can only register payment for posted journal entries." |
| Registering a payment on a plain entry | "You cannot register payments for miscellaneous entries." |
| Registering a payment on a blocked document | "You cannot register payments for blocked invoices." |
| Blocking a paid or in-payment document | "You can't block a paid invoice." |
| An unsupported combination of communication type and standard on the journal | "The combination of reference model and reference type on the journal is not implemented" |
| Choosing a counterpart on an entry of a company that has no chart of accounts — that is, a counterpart for which **both** the receivable account and the payable account resolve to nothing, which happens only when no chart has been loaded for that company | "Cannot find a chart of accounts for this company, You should configure it. \nPlease go to Account Configuration." The text carries an explicit line break before its second sentence. The refusal is a redirect: it is shown with a button labelled "Go to the configuration panel" that opens the accounting settings. It applies to every document type, the plain entry included, because the counterpart is chosen the same way on all of them. |

---

## 7. Journal Item rules

### Database checks

| Rule | Message |
|---|---|
| Debit times credit must be zero for an accountable item | "Wrong credit or debit value in accounting entry!" |
| Balance and foreign amount must share their sign for an accountable item | "The amount expressed in the secondary currency must be positive when account is debited and negative when account is credited. If the currency is the same as the one from the company, this amount must strictly be equal to the balance." |
| An accountable item needs an account | "Missing required account on accountable line." |
| A section, subsection or note carries no amount and no account | "Forbidden balance or account on non-accountable line" |

### Account and currency coherence

| Rule | Message |
|---|---|
| The account must be active, unless the item was captured automatically | "The account *the account name* (*the account code*) is archived." |
| Writing an archived account | "You cannot use an archived account." |
| An account that forces a currency must be used with that currency, unless it is the default or the suspense account of the journal | "The account selected on your journal entry forces to provide a secondary currency. You should remove the secondary currency on the account." |

### Off-balance coherence

| Rule | Message |
|---|---|
| Mixing an off-balance account with any other type in one entry | "If you want to use "Off-Balance Sheet" accounts, all the accounts of the journal entry must be of this type" |
| Taxes on an off-balance item | "You cannot use taxes on lines with an Off-Balance account" |
| Reconciling an off-balance item | "Lines from "Off-Balance Sheet" accounts cannot be reconciled" |

### Receivable and payable coherence

| Rule | Message |
|---|---|
| A payable account in a sale document | "Account *the account code* is of payable type, but is used in a sale operation." |
| A receivable account in a purchase document | "Account *the account code* is of receivable type, but is used in a purchase operation." |
| In a sale document, an item is a payment-term item if and only if its account is Receivable | "Any journal item on a receivable account must have a due date and vice versa." |
| In a purchase document, an item is a payment-term item if and only if its account is Payable | "Any journal item on a payable account must have a due date and vice versa." |

### Tax coherence

| Rule | Message |
|---|---|
| Modifying the taxes of a posted item | "You cannot modify the taxes related to a posted journal item, you should reset the journal entry to draft to do so." |
| Cash-basis and non-cash-basis taxes sharing a tax grid on one item | "Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag." |

### Deductibility

| Rule | Message |
|---|---|
| A deductibility other than one hundred outside a vendor document | "Only vendor bills allow for deductibility of product/services." |
| A deductibility outside the range zero to one hundred | "The deductibility must be a value between 0 and 100." |

### Modifying a posted item

| Rule | Message |
|---|---|
| Modifying a hashed field of an item of a hashed entry | "You cannot edit the following fields: *the field labels*.\nThe following entries are already hashed:\n*the entry numbers, one per line*" |
| Modifying an item of a reconciled entry in a way that is not allowed | "You cannot do this modification on a reconciled journal entry. You can just change some non legal fields or you must unreconcile first.\nJournal Entry (id): *the entry number* (*the identifier*)" |

The last rule is enforced indirectly: writing one of the *reconciliation-protected* fields on a matched item breaks the matches instead of refusing the write, except when the only field written is the account and every item of the matched group is written at the same time, in which case the matches survive.

### The three protected field sets

| Set | Fields | Guarded by |
|---|---|---|
| Tax | balance, originating tax, taxes, tax grids | the Tax Return Lock Date |
| Fiscal | the tax set, plus account, journal, foreign amount, currency, partner | the Global, Sales, Purchase and Hard Lock Dates |
| Reconciliation | account, date, balance, foreign amount, currency | breaking the reconciliation |

### Deletion

| Rule | Message |
|---|---|
| Deleting an item with a non-zero amount from a posted entry | "You can't delete a posted journal item. Don’t play games with your accounting records; reset the journal entry to draft before deleting it." |
| Deleting a tax item while the entry still carries taxes | "You cannot delete a tax line as it would impact the tax report" |
| Deleting a payment-term item | "You cannot delete a payable/receivable line as it would not be consistent with the payment terms" |
| Deleting an item of a hashed entry | "You cannot delete journal items belonging to a locked journal entry." |

Deleting items first undoes their reconciliations, then checks the lock dates of the posted entries that hold a non-zero item, then checks the tax lock date.

---

## 8. Numbering rules

| Rule | Message |
|---|---|
| The number must be aligned with the accounting date | "The *the date field label* (*the date*) you've entered isn't aligned with the existing sequence number (*the number*). Clear the sequence number to proceed.\nTo maintain date-based sequences, select entries and use the resequence option from the actions menu, available in developer mode." |
| The number must be unique among posted entries of the journal | "Another entry with the same name already exists." |
| A number that no shape can read | "The sequence regex should at least contain the seq grouping keys. For instance:" followed by an example pattern |
| Changing the journal of an entry that was posted before, while it keeps a number | "You cannot edit the journal of an account move if it has been posted once, unless the name is removed or set to "/". This might create a gap in the sequence." |
| Changing the journal of an entry with an assigned counter other than zero or one | "You cannot edit the journal of an account move with a sequence number assigned, unless the name is removed or set to "/". This might create a gap in the sequence." |
| Typing a number that does not match the journal override pattern | "The Journal Entry sequence is not conform to the current format. Only the Accountant can change it." An accountant is allowed and the override pattern is then cleared. |
| Deleting an entry that is not the last of its chain | "You cannot delete this entry, as it has already consumed a sequence number and is not the last one in the chain. You should probably revert it instead." |

The date-alignment check is applied only to posted entries that are not in quick-encoding mode, and only to entries whose date is later than the value of the system parameter that bounds the check (by default the first of January 1970, so in practice always). The check compares the year and the month read from the number with the period boundaries derived from the deduced periodicity: the year parts must match the start and end years of the period, truncated to the width they have in the number, and the month part must match the month of the date.

Changing the journal of a draft entry that was never posted clears the number and recomputes it, so that the entry takes a number of the new journal.

---

## 9. Locking rules

### The five checks

| Check | When it runs | Lock dates consulted |
|---|---|---|
| Fiscal lock check on an entry | Posting; unposting; changing the number or the date of a posted entry; writing a fiscal-protected field of an item of a posted entry; deleting a non-zero item of a posted entry | Global, Sales (sale journals), Purchase (purchase journals), Hard |
| Tax lock check on items | Creating an item; writing a tax-protected field of an item of a posted entry; deleting items | Tax Return, Hard |
| Bank transaction check | Writing a Global or Hard Lock Date | the resulting fiscal lock date |
| Draft entry check | Writing a Hard Lock Date | the new Hard Lock Date |
| Hard lock monotonicity | Writing a Hard Lock Date | the previous Hard Lock Date |

### Messages

| Situation | Message |
|---|---|
| Adding or modifying an entry in a locked period | "You cannot add/modify entries prior to and inclusive of: *the list of lock dates*." |
| An item that affects the tax report in a period closed for tax | "The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: *the list of lock dates*." |
| Removing the Hard Lock Date | "The Hard Lock Date cannot be removed." |
| Moving the Hard Lock Date backwards | "A new Hard Lock Date must be posterior (or equal) to the previous one." |
| Draft entries inside the period being hard-locked | "There are still draft entries in the period you want to hard lock. You should either post or delete them." with a button "Show draft entries" leading to the list of those entries |
| Unreconciled bank transactions inside the period being locked | "There are still unreconciled bank statement lines in the period you want to lock.You should either reconcile or delete them." with a button "Show Unreconciled Bank Statement Line" |
| A lock date chosen in the automatic transfer wizard | "The date selected is protected by: *the list of lock dates*." |

The list of lock dates is rendered as a comma-and-"and" list of "*the lock date label* (*the date*)", sorted chronologically.

### The lock-date shift instead of a refusal

When an entry is **posted** into a locked period, the operation is not refused: the accounting date is moved forward by the rule of `calculations.md`. The refusal applies to modifications of an already posted entry and to deletions.

### Exceptions

| Rule | Message |
|---|---|
| An exception changes exactly one lock date | "A single exception must change exactly one lock date field." |
| An exception may not be duplicated | "You cannot duplicate a Lock Date Exception." |
| Only an accounting adviser may revoke an exception | "You cannot revoke Lock Date Exceptions. Ask someone with the 'Adviser' role." |
| No exception can relax the Hard Lock Date | enforced by the selection list, which offers only the four soft lock dates |

An exception applies only when its relaxed value is **strictly smaller** than the company value; an exception that would not relax anything is simply never selected.

---

## 10. Hashing rules

| Rule | Message |
|---|---|
| Modifying a hashed field of a hashed entry | "This document is protected by a hash. Therefore, you cannot edit the following fields: *the field labels*." |
| Modifying a hashed field of an item of a hashed entry | "You cannot edit the following fields: *the field labels*.\nThe following entries are already hashed:\n*the entry numbers, one per line*" |
| Deleting an item of a hashed entry | "You cannot delete journal items belonging to a locked journal entry." |
| Resetting a hashed entry to draft | "You cannot reset to draft a locked journal entry." |
| Switching off hashing on a journal with hashed entries | "You cannot modify the field *the field label* of a journal that already has accounting entries." |
| An unreconciled bank transaction in the chain | "An error occurred when computing the inalterability. All entries have to be reconciled." |
| No entry to hash in the chain | "This move could not be locked either because some move with the same sequence prefix has a higher number. You may need to resequence it." |
| A numbering gap in the chain | "An error occurred when computing the inalterability. A gap has been detected in the sequence." |
| Printing the integrity report without the accounting user group | "Please contact your accountant to print the Hash integrity result." |
| Reordering a chain by date in a hash-secured journal | "You can not reorder sequence by date when the journal is locked with a hash." |
| Merging two contacts when at least one journal item of a contact being absorbed belongs to an entry that carries a hash | "Partners that are used in hashed entries cannot be merged." |
| Merging two accounts when both carry hashed entries | "Contains hashed entries, but *the display name of the account that would survive* also has hashed entries." — the second account is greyed out in the merge dialogue instead of the operation being refused |

Hashing an entry in a journal that does **not** secure by default activates the group that shows the inalterability features, so that the user can see the hash column.

Every hashed entry receives the audit-trail message "This journal entry has been secured."

---

## 11. Audit trail rules

| Rule | Message |
|---|---|
| Deleting a posted entry when the company keeps a restrictive audit trail | "To keep the restrictive audit trail, you can not delete journal entries once they have been posted.\nInstead, you can cancel the journal entry." |
| Switching off the restrictive audit trail when a country package forces it | "Can't disable restricted audit trail: forced by localization." |

When an entry that was posted before is force-deleted, an administration log line is written before the deletion, of the form:

```
Force deleted Journal Entries by *the user name* (*the user identifier*)
Entries
*the entry number* (*the identifier*) amount *the total* *the currency name* and partner *the partner display name*
- *the account name* (*the account identifier*) with balance *the balance* *the currency name*
…
```

Every creation, modification and deletion of an item of an entry that was posted before is logged on the entry with the tracked values, under the messages "Journal Item *link* created", "Journal Item *link* updated" and "Journal Item *link* deleted", where the link carries the item identifier prefixed by a number sign.

Tracked fields at the entry level are the number, the reference, the accounting date, the state, the document type, the reviewed flag, the partner, the currency, the recipient bank account, the salesperson, the payment reference, the payment status, the origin and the source electronic-mail address. Tracked fields at the item level are the label, the account, the balance, the taxes, the tax grids and the due date.

---

## 12. Reconciliation rules

| Rule | Message |
|---|---|
| Some items are already fully matched | "You are trying to reconcile some entries that are already reconciled." |
| Some items belong to cancelled entries | "You can not reconcile cancelled entries." |
| The items are not on the same account | "Entries are not from the same account: *the account display names*" |
| The items belong to different companies | "Entries don't belong to the same company: *the company display names*" |
| The account does not allow matching and is not a liquidity account | "Account *the account display name* does not allow reconciliation. First change the configuration of this account to allow it." |
| Off-balance items | "Lines from "Off-Balance Sheet" accounts cannot be reconciled" |
| No exchange journal configured | "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |
| No exchange loss account configured | "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |
| No exchange gain account configured | "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |
| No cash-basis journal configured, while a cash-basis entry is needed | "There is no tax cash basis journal defined for the '*the company display name*' company.\nConfigure it in Accounting/Configuration/Settings" |
| A partial reconciliation whose currencies are unknown | "Missing foreign currencies on partials having ids: *the identifiers*" |

### Which fields break a reconciliation

Writing the account, the date, the balance, the foreign amount or the currency of a matched item removes every match it takes part in and, when the counterpart was a bank transaction, undoes the reconciliation of that transaction as well — but only when undoing it does not itself violate a lock date; transactions whose undoing would be refused are left alone.

The single exception: writing **only** the account, on **every** item of the matched group at once, keeps the matches.

### What unreconciliation does

1. Payments that were "paid" because of the removed match become "in process" again.
2. The Full Reconciliation of the group, if any, is deleted.
3. Cash-basis entries created from the removed matches and the exchange-difference entry of each removed match are reversed with a cancelling reversal when they are posted, or deleted when they are draft. The reversal is dated at the date of the original entry, moved to the day after the latest violated lock date when that date is locked, and referenced "Reversal of: *the original number*".
4. The matching numbers of the surviving items are recomputed.

---

## 13. Deletion rules

| Entity | Deletable when | Message when refused |
|---|---|---|
| Account | No journal item, no fiscal position account mapping, no tax distribution line references it | see section 2 |
| Account Group | always; children are re-parented | — |
| Account Tag | Not one of the three shipped cash-flow tags; not referenced by accounts or items | see section 4 |
| Journal | Nothing references it | the ordinary restriction errors |
| Journal Entry | Draft or cancelled; the last of its numbering chain, unless the user is an accountant, the company is in quick-encoding mode, or deletion is forced; never posted when the company keeps a restrictive audit trail; not hashed | see sections 6, 8 and 11 |
| Journal Item | Its entry is not posted, or its amount is zero; it is not a tax item of an entry that still carries taxes; it is not a payment-term item; its entry is not hashed | see section 7 |
| Partial Reconciliation | always; the side effects of section 12 apply | — |
| Full Reconciliation | always; the matching numbers fall back to the partial form | — |
| Lock Exception | not applicable: an exception is revoked, not deleted | — |
| Contact | No Journal Entry in the draft or the posted state names it as its counterpart. A Journal Entry in the cancelled state does **not** block the deletion | "The partner cannot be deleted because it is used in Accounting" — see section 21 |

---

## 14. Permission checks

| Operation | Required group |
|---|---|
| Post an entry | Invoicing |
| Mark an entry as reviewed, or reset a reviewed entry to draft | the reviewing permission (the accounting user group, and for a miscellaneous journal the reviewed flag is set automatically at posting) |
| Type a number that does not match the journal override pattern | Administrator |
| Revoke a lock exception | Administrator |
| Print the hash integrity report | Show Full Accounting Features |
| Resequence entries | Administrator |
| Secure entries through the wizard | Administrator |
| Create, modify and delete accounts, journals, journal groups and account groups | Administrator |
| Create, modify and delete entries and items | Invoicing |
| Read entries, items, accounts, journals and groups | Show Accounting Features - Readonly, or Invoicing |
| See the account code in a display name | Show Accounting Features - Readonly |
| Unmerge an account | write access on the account **and** access to every company of the account |

The complete matrix is in `configuration.md`.

---

## 15. Company rules

| Rule | Message |
|---|---|
| The fiscal-year end day must exist in the fiscal-year end month | "Invalid fiscal year last day" |
| The currency may not change once journal items exist | "You cannot change the currency of the company since some journal items already exist" |
| The price-inclusion default may not change once invoicing has started | "Cannot change Price Tax computation method on a company that has already started invoicing." |
| The restrictive audit trail may not be switched off when forced | "Can't disable restricted audit trail: forced by localization." |
| A chart of accounts must exist before accounting is used | "We cannot find a chart of accounts for this company, you should configure it. \nPlease go to Account Configuration and select or install a fiscal localization." with a button "Go to the configuration panel" |
| The opening entry may not be modified once posted | "You cannot import the "openning_balance" if the opening move (*the number*) is already posted. If you are absolutely sure you want to modify the opening balance of your accounts, reset the move to draft." |
| No miscellaneous journal when the opening entry must be created | "Please install a chart of accounts or create a miscellaneous journal before proceeding." |

The fiscal-year end day check accepts the twenty-ninth of February unconditionally, because the applicable year is unknown at configuration time.

---

## 16. Wizard rules

### Reversal

| Rule | Message |
|---|---|
| All selected entries must belong to one company | "All selected moves for reversal must belong to the same company." |
| All selected entries must be posted | "To reverse a journal entry, it has to be posted first." |
| The chosen journal must be of the same type | "Journal should be the same type as the reversed entry." |

### Automatic transfer

| Rule | Message |
|---|---|
| The selection must be journal items | "This can only be used on journal items" |
| Every selected item must belong to a posted entry | "Oops! You can only change the period or account for posted entries! Other ones aren't up for an adventure like that!" |
| No selected item may be reconciled | "Oops! You can only change the period or account for items that are not yet reconciled! Other ones aren't up for an adventure like that!" |
| All selected items must belong to one company hierarchy | "You cannot use this wizard on journal entries belonging to different companies." |
| At least one action must be possible | "No possible action found with the selected lines." |
| For a change of period, every selected item must be on an account of the same type | "All accounts on the lines must be of the same type." |
| The percentage must lie strictly above zero and at most one hundred for a change of period | "Percentage must be between 0 and 100" |
| The chosen date must not be locked | "The date selected is protected by: *the list of lock dates*." |

### Resequence

| Rule | Message |
|---|---|
| One journal only | "You can only resequence items from the same journal" |
| A journal with a dedicated credit-note numbering may not mix credit notes with other documents | "The sequences of this journal are different for Invoices and Refunds but you selected some of both types." |
| A journal with a dedicated payment numbering may not mix payment entries with other entries | "The sequences of this journal are different for Payments and non-Payments but you selected some of both types." |
| Reordering by date in a hash-secured journal | "You can not reorder sequence by date when the journal is locked with a hash." |

### Validate entries

| Rule | Message |
|---|---|
| The context must name a source | "Missing 'active_model' in context." |
| At least one draft entry with items must exist | "There are no journal items in the draft state to post." |

### Secure entries

| Rule | Message |
|---|---|
| A date is required | "Set a date. The moves will be secured up to including this date." |

### Account merge

Two refusals stop the dialogue outright, and two blocking reasons grey out one account line inside it without stopping anything.

**The two refusals**, both raised while the dialogue is being opened:

| Rule | Condition | Message |
|---|---|---|
| The selection must be accounts | The dialogue is opened from a list of records that are not accounts | "This can only be used on accounts." |
| At least two accounts | Fewer than two records are selected | "You must select at least 2 accounts." |

A third refusal is raised when the merge itself is launched, and again inside the merge of each group:

| Rule | Condition | Message |
|---|---|---|
| Write access | The acting user may not write on one of the accounts | the ordinary access error |
| Company access | One of the companies of the accounts is not among the companies the acting user may act for | "You do not have the right to perform this operation as you do not have access to the following companies: *the names of those companies, separated by a comma and a space*." — the same text as the unmerge check of section 2 |

**The two blocking reasons.** They are recomputed for a whole group every time any line of that group is ticked or unticked, and they are evaluated in this order, each only over the lines that are ticked and not already blocked:

| Order | Condition | Text written into the blocking reason |
|---|---|---|
| 1 | The account shares at least one company with an account that comes earlier in the group and is itself still eligible | "Belongs to the same company as *the display name of that earlier account*." |
| 2 | The account carries hashed entries and an earlier still-eligible account of the group already carries hashed entries | "Contains hashed entries, but *the display name of that earlier account* also has hashed entries." |

A blocked line stays visible, keeps its tick box and shows its reason in the information column; it is simply skipped by the merge. Two accounts of the same company are blocked because merging them would put two codes of one company on one account, which the code uniqueness rule forbids; the remedy offered is to re-point the journal items themselves. Two accounts that both carry hashed entries are blocked because a merge keeps only one of the two identifiers, and moving a hashed journal item to another account identifier would break the hash of its entry.

**The Merge button.** It is inert whenever every group holds fewer than two lines that are both ticked and unblocked, that is whenever the operation would do nothing.

**The heading of a group.** Each group shows a heading built from the first account of the group, as follows:

1. Start from the label of the account type of that account.
2. When that type is Receivable or Payable, replace the label by "Non-trade *the label*" when the account is flagged as non-trade, and by "Trade *the label*" otherwise.
3. Collect the additional elements, in this order: the name of the currency when the account restricts its items to one currency; the word "Reconcilable" when the account allows matching; the word "Deprecated" when the account is archived.
4. When the dialogue does **not** group by name: the heading is the label of step 2, followed — only when at least one additional element was collected — by a space and the additional elements between parentheses, separated by a comma and a space.
5. When the dialogue **does** group by name: the heading is the account name, a space, then between parentheses the label of step 2 followed by the additional elements, all separated by a comma and a space. In this form the parentheses are always present.

For example a trade receivable account in dollars that allows matching and is still in use gives, when the dialogue does not group by name, a heading of the shape "Trade Receivable (USD, Reconcilable)" — with the currency name reproduced as it is stored.

### Fiscal year opening

| Rule | Condition | Message |
|---|---|---|
| The fiscal-year end day must exist in the fiscal-year end month | The chosen day and month do not form a valid date. The test is made against the year 2020, a leap year, so that the twenty-ninth of February is accepted | "Incorrect fiscal year date: day is out of range for month. Month: *the chosen month*; Day: *the chosen day*" |

This check belongs to the wizard, not to the company, and its text is deliberately different from the company-level check quoted in section 15 ("Invalid fiscal year last day"). The reason the wizard carries its own check is that it writes the day and the month to the company in one single operation; a check placed on the company alone would be evaluated after each of the two values separately and would reject a legitimate pair such as moving from the thirty-first of December to the thirtieth of June, because it would see the thirty-first of June in between.

---

## 17. Edge cases

### An entry with no item

An entry with no item passes the balance check (the check only looks at entries that have items) but cannot be posted: "Even magicians can't post nothing!"

### An entry whose items are only sections and notes

The same: sections, subsections and notes are not accountable, so posting is refused.

### An item with a zero balance and a non-zero foreign amount

Allowed, and required: an exchange-difference item that corrects only the foreign amount looks exactly like this. Such an item is **not** treated as reconciled just because both residuals happen to be zero; it must participate in at least one match to count as reconciled in the full-reconcile detection.

### An item with a zero balance and a zero foreign amount on a reconcilable account

When reconciliation is switched on for the account, such an item is immediately flagged as reconciled.

### A number cleared by the user

Clearing the number of a posted entry is the deliberate way to create a gap. The gap detection then flags the entry that follows the hole. Nothing refuses the operation, but the securing routine will refuse to hash a chain containing the hole.

### Posting several entries in one transaction

The numbering cache means that only the **first** entry of each chain takes a database lock; the following ones take their numbers from the transaction cache. If the transaction rolls back, none of the numbers is consumed.

### Two documents of the same journal with different periodicities

The numbering search first determines the periodicity from the closest earlier number, then excludes the prefixes of finer periodicities. A journal that carried monthly numbers in one year and yearly numbers in another therefore continues each series independently, provided the prefixes differ.

### A credit note in a journal without a dedicated credit-note numbering

The credit note shares the chain of the invoices and does **not** receive the letter R prefix.

### An entry moved between journals

A draft entry never posted loses its number when the journal changes, and takes a new one at posting. An entry that was posted before keeps its number and the change is refused unless the number is cleared in the same operation.

### The same account used in two companies of one hierarchy

The account keeps one record and two codes. Reports of the parent company aggregate the items of both companies on that one account, each displayed with the code of the company being viewed.

### A company with no fiscal-year offset

When the fiscal year ends on the thirty-first of December, the year part of a generated number has four digits and the counter five; otherwise the year part is a two-digit range and the counter four. Changing the fiscal-year end therefore changes the *starting* number of a **new** chain but never rewrites the existing ones.

### Reconciling an item with itself

Impossible: the pairing loop splits items into a debit list and a credit list by sign, and one item can be in both lists only when its balance and its foreign amount have opposite signs, which the sign-coherence check forbids for accountable items.

### Reconciling two items of the same entry

Allowed by the algorithm, and used when an entry contains both a debit and a credit on the same reconcilable account; but the payment-status computation deliberately ignores matches whose counterpart belongs to the same entry.

### A match whose amount rounds to zero

The pairing loop still advances, because an item is dropped as soon as both of its residuals are zero; a match with a zero amount is simply not recorded (only results that produced values are appended).

### Deleting an entry that is the only one of its chain

Allowed: the chain-end test answers yes for a single entry with no previous number.

---

## 18. Field-level write protection

This section consolidates, per field, what may be written in each state. It is the reference an implementation should test against.

### On a Journal Entry

| Field | Draft | Posted | Cancelled | Hashed (always posted) |
|---|---|---|---|---|
| Number | free, subject to the date-alignment check and the journal pattern | free, subject to the same checks plus the lock-date re-check | free | **refused** |
| Reference | free | free | free | free |
| Accounting date | free | **refused** as a readonly field, and additionally lock-checked | free | **refused** |
| State | through the operations only | through the operations only | through the operations only | only the securing operation |
| Document type | at creation only | at creation only | at creation only | at creation only |
| Journal | free while never posted; refused while a number is kept | **refused** while a number is kept | same | **refused** |
| Company | free | **refused** (it is recomputed from the journal) | free | **refused** |
| Currency | free | **refused** as a readonly field | free | free |
| Items | free | **refused** as a readonly field | free | **refused** |
| Counterpart | free | **refused** as a readonly field | free | free |
| Payment terms, fiscal position, cash rounding | free | **refused** as a readonly field | free | free |
| Automatic posting mode and its end date | free | free | forced to "No" by the cancellation | free |
| Reviewed flag | free | free for a user allowed to review | free | free |
| Payment reference, narration, origin, salesperson, incoterm | free | free | free | free |
| Payment status | computed; the blocked value is set and cleared manually | same | same | same |

"Refused as a readonly field" means the write raises "You cannot modify the following readonly fields on the posted move *the identification*: *the field names*" unless the caller explicitly bypasses the check, which only internal routines do.

### On a Journal Item

| Field | Entry draft | Entry posted | Entry posted and matched | Entry hashed |
|---|---|---|---|---|
| Label | free | free | free | **refused** |
| Account | free | lock-checked | breaks the match unless the whole group is written together | **refused** |
| Balance, debit, credit | free | lock-checked (fiscal and tax) | breaks the match | **refused** |
| Foreign amount | free | lock-checked (fiscal) | breaks the match | free unless it is a hashed field — it is not, so free |
| Currency | free | lock-checked (fiscal) | breaks the match | free |
| Counterpart | free | lock-checked (fiscal) | free | **refused** |
| Counterpart, when the change comes from re-parenting the contact | free | **exempt from the lock check**: the whole set of items of that contact is rewritten in one operation with the lock check suppressed, so items of locked periods are rewritten too; see section 21 | same exemption | still **refused**, because the counterpart is a hashed field |
| Taxes, originating tax | free | **refused** | **refused** | **refused** |
| Tax grids | free | lock-checked (tax) | free | free |
| Due date | free | free | free | free |
| Analytic distribution | free; the analytic lines of a draft item are deleted | free; the analytic lines are deleted and recreated | free | free |
| Journal (related) | follows the entry | follows the entry | follows the entry | follows the entry |
| Date (related) | follows the entry | follows the entry | breaks the match when the entry date changes | follows the entry |
| Matching number | written only by the reconciliation routine; an imported label is rewritten with the letter `I` in front | same | same | same |

### The precedence of the checks on one write

For one write on one item of a posted entry, the checks run in this order and the first failure stops the operation:

```
 1. The archived-account check on the written account.
 2. The hashed-field check.
 3. For each item: the "will the value actually change" test; unchanged items are dropped
    from the write entirely.
 4. The tax-modification refusal.
 5. The fiscal lock check, when a fiscal-protected field changes.
 6. The tax lock check is deferred: the identifiers are collected and checked after the loop.
 7. The reconciliation break: the matches are removed and the linked bank transactions are
    unreconciled, except those whose unreconciliation would itself violate a lock date.
 8. The collected tax lock checks run.
 9. The balance invariant and the dynamic-line synchronisation are opened around the write.
10. The write happens, the tracked values are logged, and the account-and-journal coherence
    is re-checked when the account or the currency changed.
11. The tax lock check runs a second time, to catch an item that did not affect the tax
    report before the write but does after it.
```

---

## 19. Concurrency and transactional rules

| Rule | Statement |
|---|---|
| Numbering uniqueness | Guaranteed by a unique index over (number, journal) restricted to posted entries whose number is not the placeholder. The reservation is a direct write that takes an exclusive lock on the index entry. |
| Numbering retries | A uniqueness violation on the reservation is recovered by rolling back to a savepoint and trying the next counter, indefinitely until a free value is found. |
| Numbering cache | After the first reservation of a chain in a transaction, further numbers come from an in-transaction cache keyed by (the format filled with counter zero, the journal). The cache is cleared whenever the number field is written by an ordinary write. |
| Cache invalidation on rollback | A savepoint rollback or a commit must clear the numbering cache, because the lock it relied on is released. |
| Scheduled posting | The daily posting job locks each entry before posting it alone, and re-tests it against the search condition after the lock, so that a concurrently posted entry is skipped. |
| Sending job | The sending job locks the entries it takes, so two runs never process the same document. |
| Reconciliation | The whole plan is computed in memory against a snapshot of the residuals, and the matches are created in one operation, so a partially applied plan is never visible. |
| Balance invariant | The check is deferred to the end of the enclosing operation, so intermediate unbalanced states inside one write are allowed. |
| Lock date caching | The per-user lock dates are cached and explicitly invalidated when a company lock date is written and when an exception is created or revoked, for **every** company, because an exception of a parent company changes the value seen from a child. |

---

## 20. Rules that other domains rely on

An implementation must keep these guarantees because other domains are built on them.

| Guarantee | Relied on by |
|---|---|
| A posted entry never changes its accounting date except through an explicit, lock-checked write | every report, the tax return, the audit trail |
| A posted entry keeps its number for ever, and the pair (number, journal) is unique among posted entries | every legal document, the electronic exchange, the audit |
| A reconciliation never changes the balance of an item; it only creates matches | the reports, the aged balances |
| The residual of an item is always the balance minus the matched amounts, in both currencies | the payment status, the aged balances, the outstanding widgets |
| A matched group that nets to zero always carries a Full Reconciliation | the payment status, the exchange-difference reversal |
| Undoing a match always reverses or deletes the entries it produced | the tax return, the exchange result |
| An item on an account that does not allow matching always has zero residuals | the reports |
| Deleting an entry is impossible once it is hashed, and impossible once it is posted when the company keeps a restrictive audit trail | the legal archive |
| Every entry produced by any domain balances in the company currency | the whole ledger |


---

## 21. Rules this domain contributes to the Contact

The Contact itself belongs to `../contacts-and-organizations/`. This domain adds three rules to it, all three enforced from the accounting side and all three invisible to a reader of that folder alone.

### 21.1 A Contact used in accounting cannot be deleted

**Condition.** The deletion of one or more contacts is refused as soon as at least one Journal Entry in the **draft** or the **posted** state names any of them as its counterpart. The count is made ignoring the record rules, so an entry the acting user cannot see still blocks the deletion. An entry in the **cancelled** state does not block anything, and neither does a Journal Item whose entry has been cancelled.

**Message.** "The partner cannot be deleted because it is used in Accounting"

**Why the rule is on the entry and not on the item.** Every accountable Journal Item copies the commercial entity of the counterpart of its entry, so testing the entry covers the items; testing the item alone would miss an entry that carries a counterpart on its header but no counterpart on its lines.

### 21.2 A Contact used in a hashed entry cannot be merged

**Condition.** Merging contacts is refused when at least one Journal Item belonging to one of the contacts being **absorbed** sits in an entry that carries an inalterability hash. The test reads the items ignoring the record rules and stops at the first hit. Items of the surviving contact are not tested, because the surviving contact keeps its identifier.

**Message.** "Partners that are used in hashed entries cannot be merged."

**Why.** A merge re-points the journal items of the absorbed contacts to the identifier of the survivor. The counterpart identifier of an item is one of the values that enters the hash of its entry, so a merge would make the entry fail verification for ever. This is the same reasoning that blocks a merge of two accounts that both carry hashed entries.

### 21.3 Re-parenting a Contact rewrites its journal items

Writing the parent of a contact changes which commercial entity it belongs to, and the commercial entity is what the ledger books against. The write therefore carries a ledger consequence.

**Step 1 — the check before the write.** Before anything is written, the contacts of the operation whose parent actually changes are collected together with their journal items. When at least one of them has journal items, a new parent is being set, and the tax number of any of those contacts differs from the tax number of the new parent, the write is refused with "You cannot set a partner as an invoicing address of another if they have a different *the label of the tax number field for the country*." — that refusal is a tax rule and is specified in `../taxes/`. An empty tax number on either side counts as an empty text, so a contact without a tax number may be re-parented under a parent without one.

**Step 2 — the write itself.** The parent is written by the ordinary mechanism, which recomputes the commercial entity of the contact and of its own children.

**Step 3 — the propagation, for each affected contact in turn.**

1. The commercial entity of the contact is recomputed.
2. **Every** journal item that named that contact as its counterpart — the set collected in step 1, posted items of locked periods included — is rewritten in one single operation to the new commercial entity. The write is made with the **lock check suppressed**: the fiscal, tax, sale, purchase and hard lock dates are not consulted, and the write succeeds on items whose accounting date lies in a locked or even a hard-locked period.
3. Among the entries of those items, those whose own counterpart is that contact — that is, the entries **wholly dedicated** to it — have their commercial entity rewritten to the same value, again with the lock check suppressed. An entry that is shared between several counterparts, such as a miscellaneous entry or a grouped bank payment, keeps its commercial entity unchanged.
4. The message "The commercial partner has been updated for all related accounting entries." is logged on the contact.

**Why the whole set must be written at once.** The reconciliation check that runs on a write of the counterpart compares the counterpart of the items of one matched group; writing the items one at a time would make the group temporarily inconsistent and the check would refuse the second write. Writing them in one operation lets the check see the final, consistent state.

**Compatibility finding.** This is the only path in the domain that writes the counterpart of a posted journal item without consulting the lock dates, and section 18 records the exemption. It means that re-parenting a contact silently modifies items of periods that are closed, and of periods protected by the Hard Lock Date, which every other path treats as irreversible. A corrected behaviour would either refuse the re-parenting when any affected item lies on or before the effective hard lock date of its company, or leave those items on the old commercial entity and log which ones were skipped. The observed behaviour is the one specified above.
