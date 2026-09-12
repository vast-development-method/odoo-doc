# General Ledger — Workflows

End-to-end operational sequences, step by step, with the role that performs each step, the preconditions, and the records created or updated at each step.

Contents:

1. [Setting up the chart of accounts](#1-setting-up-the-chart-of-accounts)
2. [Recording the opening balances](#2-recording-the-opening-balances)
3. [Creating an account by hand](#3-creating-an-account-by-hand)
4. [Splitting a shared account per company](#4-splitting-a-shared-account-per-company)
5. [Merging several accounts into one](#5-merging-several-accounts-into-one)
6. [Creating a journal](#6-creating-a-journal)
7. [Recording a miscellaneous entry](#7-recording-a-miscellaneous-entry)
8. [Posting an entry](#8-posting-an-entry)
9. [Posting entries in bulk](#9-posting-entries-in-bulk)
10. [Automatic and recurring posting](#10-automatic-and-recurring-posting)
11. [Resetting an entry to draft](#11-resetting-an-entry-to-draft)
12. [Cancelling an entry](#12-cancelling-an-entry)
13. [Reversing an entry](#13-reversing-an-entry)
14. [Deleting an entry](#14-deleting-an-entry)
15. [Matching journal items](#15-matching-journal-items)
16. [Undoing a match](#16-undoing-a-match)
17. [Moving an amount to another account](#17-moving-an-amount-to-another-account)
18. [Moving an amount to another period](#18-moving-an-amount-to-another-period)
19. [Renumbering a set of entries](#19-renumbering-a-set-of-entries)
20. [Closing a period with a lock date](#20-closing-a-period-with-a-lock-date)
21. [Granting a lock date exception](#21-granting-a-lock-date-exception)
22. [Securing entries with the hash chain](#22-securing-entries-with-the-hash-chain)
23. [Verifying the hash chain](#23-verifying-the-hash-chain)
24. [Closing a fiscal year](#24-closing-a-fiscal-year)
25. [The accounting onboarding path](#25-the-accounting-onboarding-path)

Roles used in this document:

| Role | Group |
|---|---|
| Billing clerk | Invoicing |
| Accountant | Show Full Accounting Features |
| Accounting administrator | Administrator |
| Auditor | Show Accounting Features - Readonly |
| The system | a scheduled job or an internal routine |

---

## 1. Setting up the chart of accounts

**Who:** accounting administrator. **Precondition:** the company exists and has a country.

```
 1. The administrator picks a chart template. The list offered is built from the installed
    and installable capability packages, each declaring one or more templates with a name,
    a country, a parent template and the package that ships it. The list is sorted so that
    the templates of the country of the company come first; when the company has no country,
    the generic template comes first.
 2. The system refuses two regional templates that must not be picked directly:
    "The *the template code* chart template shouldn't be selected directly. Instead, you
    should directly select the chart template related to your country."
 3. Only a system administrator may load a template: "Only administrators can install chart
    templates".
 4. If the company has no country, it takes the country of the template.
 5. If the package that ships the template is not installed, it is installed first.
 6. The loading runs with the language forced to the base language, with tracking disabled
    and with the account-group hierarchy synchronisation delayed.
 7. The system decides whether this is a *reload* (the company already carries this template
    code) or a first load.
 8. On a first load, and only when the company hierarchy has no journal item yet (or demo
    data is requested), every existing record of the template models is deleted for the
    company and its children, in reverse dependency order, plus the journal entries. The
    template models are, in dependency order: account groups, accounts, fiscal positions,
    tax groups, taxes, journals, reconciliation models. Records shared with a company outside
    the subtree are not deleted; they merely lose the companies of the subtree.
 9. The template data is read. A subsidiary loads only the company-level values of the
    template, not the records: it reuses the records of its parent.
10. On a reload, the pre-reload pass restricts the update to the main configuration values
    and never re-creates what the user has changed.
11. The pre-load pass:
      a. Determines the fiscal country from the template.
      b. Copies the template company values onto the company, excluding the name and the
         per-company property values that are not stock-related.
      c. Sets the company currency to the currency of the fiscal country, or to the parent
         company currency for a subsidiary — but only when no accounting exists yet.
      d. Sets the country when the company has none.
      e. Forces the anglo-saxon switch to false unless the template says otherwise.
      f. Pads every account code of the template on the right with zeros up to the template
         code width (six characters by default).
      g. Moves the fiscal positions and the reconciliation models to the end of the data so
         that they are created after the taxes and accounts they refer to.
      h. Drops every value whose field does not exist.
      i. Translates the fields that are technically untranslatable but must nevertheless be
         localised.
12. The load pass creates or updates every record, resolving cross references by external
    identifier. An identifier without a dot is interpreted as belonging to the accounting
    package and prefixed with the company identifier, so the identifier `cash` of company 3
    becomes the external identifier `account.3_cash`. Records whose values refer to records
    not yet created are delayed until their dependencies exist.
13. The post-load pass:
      a. Creates the utility accounts that the company needs and links them to the company
         fields (see the table below). A subsidiary takes those of its first ancestor.
      b. Creates the two outstanding accounts of the company.
      c. Creates the current-year-earnings account when none exists.
      d. Fills, on every bank, cash and credit-card journal, the suspense account, the
         profit account and the loss account from the company values when they are empty.
      e. Sets the cash-basis journal and the exchange journal of the company from the
         journals just created, when they are empty.
      f. Sets the default account of the sale journal to the company income account, and of
         the purchase journal to the company expense account.
      g. Sets the default sale tax and purchase tax of the company to the first matching tax.
      h. Forces the default taxes onto the products that already carry a tax in another
         company.
      i. Switches the cash-basis feature on when at least one tax is exigible on payment.
      j. Writes the per-company default values of the property accounts.
      k. Writes the company income and expense accounts as the default accounts of product
         categories.
      l. Points the internal-transfer reconciliation model at the liquidity transfer account
         and the bank-fee reconciliation model at an account whose name contains "Bank Fees",
         falling back to the first expense account.
      m. Starts the accounting onboarding for the company.
14. Translations are loaded, the account-group hierarchy is rebuilt for the company, and
    demo data is loaded when requested (a failure there does not roll back the chart).
15. Every child company of the company is loaded with the same template, recursively.
```

### Utility accounts created by the post-load pass

| Company field | Name | Code derivation | Type | Reconcilable | Tags |
|---|---|---|---|---|---|
| Journal suspense account | "Bank Suspense Account" | the bank prefix padded to the code width | Current Assets | no | — |
| Cash discount loss account | "Cash Discount Loss" | the fixed code `999998` | Expenses | no | — |
| Cash discount gain account | "Cash Discount Gain" | the fixed code `999997` | Other Income | no | — |
| Cash difference income account | "Cash Difference Gain" | the prefix `999` padded to the code width | Other Income | no | the investing cash-flow tag |
| Cash difference expense account | "Cash Difference Loss" | the prefix `999` padded to the code width | Expenses | no | the investing cash-flow tag |
| Liquidity transfer account | "Liquidity Transfer" | the transfer prefix padded to the code width | Current Assets | yes | — |
| *(no company field)* | "Outstanding Receipts" | the bank prefix padded to the code width | Current Assets | yes | — |
| *(no company field)* | "Outstanding Payments" | the bank prefix padded to the code width | Current Assets | yes | — |

Each prefix-derived code is obtained by the free-code walk of `calculations.md`, starting from the prefix padded on the right with zeros to the code width.

### Journals created by the generic template

| Key | Name | Type | Code | On the dashboard | Order |
|---|---|---|---|---|---|
| sale | Sales | Sales | `INV` | yes | 5 |
| purchase | Purchases | Purchase | `BILL` | yes | 6 |
| general | Miscellaneous Operations | Miscellaneous | `MISC` | no | 9 |
| exch | Exchange Difference | Miscellaneous | `EXCH` | no | — |
| caba | Cash Basis Taxes | Miscellaneous | `CABA` | no | — |
| bank | Bank | Bank | *(generated)* | yes | 7 |

### Reconciliation models created by the generic template

| Key | Name | Matching condition | Line |
|---|---|---|---|
| internal transfer | Internal Transfers | none | one line of 100 % labelled "Internal Transfers", pointing at the liquidity transfer account |
| bank fees | Bank Fees | the transaction label contains "Bank Fees" | one line of 100 % labelled "Bank Fees" |

---

## 2. Recording the opening balances

**Who:** accountant. **Precondition:** the chart of accounts exists and a miscellaneous journal exists.

```
 1. The accountant opens the chart of accounts and types an opening debit, an opening credit
    or an opening balance on each account — or imports a spreadsheet carrying an opening
    balance column.
 2. Each value is collected in a pending buffer rather than written immediately, so that
    importing a thousand accounts produces one update of the opening entry instead of a
    thousand.
 3. At the end of the operation the buffer is applied per company by the algorithm of
    `calculations.md`:
      a. the opening entry is created when it does not exist, in the first miscellaneous
         journal, dated the day before the opening date of the company (or the thirty-first
         of December of the previous year when no opening date is set), referenced "Opening
         Journal Entry";
      b. one item per account and per side is created, updated or deleted;
      c. the balancing item on the current-year-earnings account is recomputed so that the
         entry balances.
 4. The accountant reviews the entry and posts it. From that moment the opening balances can
    no longer be edited from the chart of accounts: "You cannot import the "openning_balance"
    if the opening move (*the number*) is already posted. …"
```

---

## 3. Creating an account by hand

**Who:** accounting administrator.

```
 1. The administrator opens the chart of accounts and adds a row.
 2. Typing a name that begins with a token containing a digit moves that token into the code
    box and leaves the rest as the name.
 3. Typing a code schedules the recomputation of the account type: the type is copied from
    the account whose code immediately precedes the new one.
 4. The tags are copied by the same rule.
 5. The reconcilable flag is derived from the type (see the table in `entities.md`).
 6. When the user belongs to several companies, a per-company code tab is shown, pre-filled
    with one row per company.
 7. On save, the code uniqueness across the company hierarchy is checked, and the presence of
    a code in every root company is checked.
```

---

## 4. Splitting a shared account per company

**Who:** accounting administrator with access to every company of the account.

```
 1. The administrator selects one or more shared accounts and runs the unmerge action.
 2. The system checks the write access, the company access and that each account belongs to
    more than one company.
 3. A confirmation dialogue lists, per account, how many accounts will be produced and what
    display name each will have.
 4. On confirmation, for each account:
      a. The first company is the active company when it is among the companies of the
         account, otherwise the first company of the account. It keeps the original record.
      b. One copy is created per other company, carrying the same name, that single company,
         and the values of every company-restricted link that belong to that company.
      c. Every stored reference to the original account is repointed to the copy of the
         company of the referring record: ordinary links, multi-value links, per-company
         link values, polymorphic references and typed identifier references.
      d. The per-company values stored on the original account are dispatched to the copies
         and removed from the original.
      e. External identifiers of the form "the company identifier, an underscore, the rest"
         are repointed to the matching copy.
      f. The original account keeps only the first company, and its company-restricted links
         are filtered to that company.
      g. A message is logged on each copy: "This account was split off from *link to the
         original* (*the first company name*)."
 5. The client is reloaded.
```

Nothing changes from an accounting point of view: every journal item keeps the code it had, because the codes were already per company.

---

## 5. Merging several accounts into one

**Who:** accounting administrator with access to every company of the accounts.

**What it is for.** A chart that grew by hand, or a chart loaded twice, ends up with several accounts that mean the same thing. Merging collapses them into one account that keeps **every** code and **every** company of the set, so that no journal item changes its code and no company loses its account. It is the mirror image of the split of section 4: the split turns one account shared by several companies into one account per company, the merge turns several accounts into one shared account.

**Preconditions.** The operation is offered on the chart of accounts to the Administrator group only. It refuses at once when the selection is not made of accounts, and when it holds fewer than two records. The two messages are in `business-rules.md`.

### Step A — building the groups

```
 1. The selected accounts are read, and every account whose type is Bank and Cash is dropped:
    a liquidity account is tied to its journal and to the money it holds, and must never be
    merged with another. Every other type is kept, the Credit Card type included.
 2. Each remaining account is reduced to a grouping key made of five values, in this order:
      - the account type,
      - the non-trade flag,
      - the currency the account restricts its items to (empty when it restricts none),
      - the reconcilable flag,
      - the active flag.
    When the "Group by name?" switch of the dialogue is ticked, the account name is appended
    to the key as a sixth value, so that two accounts that differ only by name fall into two
    groups instead of one.
 3. The accounts are grouped by that key, each group keeping the accounts in the order in
    which they were read.
 4. The dialogue is filled: for each group in turn, one heading row followed by one row per
    account of the group, numbered consecutively from one. The heading carries the group name
    built by the rule of `business-rules.md`; every account row starts ticked.
 5. Every account row is then checked against the two blocking reasons of `business-rules.md`
    (two accounts of one company, two accounts that both carry hashed entries). A blocked row
    keeps its tick box but is skipped by the merge. Ticking or unticking any row re-runs this
    check for the whole group, because whether one account is blocked depends on which others
    are still selected.
 6. The Merge button is inert while no group holds at least two rows that are both ticked and
    unblocked.
```

Changing the "Group by name?" switch, or changing the set of accounts, rebuilds the whole list from step 1, which discards any ticking the user had done.

### Step B — the merge itself

```
 1. The write access of the acting user on the selected accounts is checked, and then the
    companies: every company of those accounts must be a company the user may act for,
    otherwise the operation is refused with the company message of `business-rules.md`.
 2. For each group, the rows that are ticked and unblocked are collected. A group with fewer
    than two such rows is skipped entirely.
 3. The accounts of the group are ordered so that an account carrying hashed entries comes
    first; the order of the others is unchanged. At most one account of a group can carry
    hashed entries and still be unblocked, so this makes that one the survivor and guarantees
    that no hashed journal item ever changes its account identifier.
 4. The merge of one group then runs the eight steps below.
```

### Step C — merging one group of accounts

```
 1. Collect, before anything is changed:
      - the union of the companies of every account of the group;
      - the union of their per-company code maps, read directly from storage, giving one code
        per company over the whole group. No company can appear twice, because two accounts of
        one company are never both eligible.
    Nothing is written yet: writing the companies or the codes now would put two codes of one
    company on two coexisting accounts and trip the code uniqueness rule.
 2. The first account of the ordered group becomes the surviving account; the others are the
    accounts to remove.
 3. The write access and the company access are checked again, on the accounts of this group.
 4. Every stored reference to an account to remove is repointed to the survivor: for each
    table that holds a column pointing at the account table, the rows naming an account to
    remove are rewritten to name the survivor. Where that rewriting would break a uniqueness
    or a validity constraint of the target table — typically a link table that already holds
    the same pair for the survivor — the offending rows are deleted instead of rewritten,
    because a row pointing at an account that is about to disappear is worthless. This is what
    moves the journal items, the tax distribution lines, the fiscal position mappings, the
    journal default accounts, the reconciliation models and every other reference in one pass.
 5. The references that name a record by model name and identifier rather than by a typed
    column are repointed the same way, with the same deletion fallback. Five kinds are
    covered: attachments, followers, scheduled activities, messages and external identifiers.
 6. The translated names are merged. The stored translations of the name of every account of
    the group are read, and merged into one map by walking the accounts from the last to the
    first, each overwriting the previous: the **first account wins** for every language it has
    a translation in, and a language it has no translation in is filled from the first
    following account that has one. The merged map is written on the survivor, replacing its
    own name.
 7. The accounts to remove are deleted directly from storage, in one operation. Because the
    deletion is direct, the ordinary deletion guards of the Account (no journal item, no
    fiscal position mapping, no tax distribution line) are not consulted — they would all be
    satisfied anyway, since step 4 has just emptied those references. The caches of the
    external identifiers are cleared afterwards.
 8. The collected code map of step 1 is written on the survivor, then the collected companies,
    and the recomputation of the tags of the survivor is scheduled. The survivor now belongs
    to every company of the group and carries, in each of them, the code the merged account
    had there.
```

### Step D — the result

```
 1. A success notification is shown carrying the message "Accounts successfully merged!"
 2. Dismissing it closes the dialogue.
```

**What has changed in the ledger:** nothing. Every journal item keeps its amount, its date, its entry and its code, because the code is per company and every code was preserved. What has changed is the chart: several rows became one, and that one row is now shared by all the companies that had their own.

**What is lost.** The identifiers of the removed accounts, their own translated names in the languages the survivor already covers, and any row that step 4 or step 5 had to delete rather than rewrite. The survivor keeps its own identifier, so every reference to it, hashed entries included, stays valid.

---

## 6. Creating a journal

**Who:** accounting administrator.

```
 1. The administrator chooses a type. Choosing a type resets the code, the default account
    and the profit and loss accounts, proposes a code (the type prefix and the smallest free
    number), proposes a name placeholder, and for a sale or purchase journal proposes an
    incoming-mail local part.
 2. For a sale journal the default account is pre-filled with the company income account when
    it is active; for a purchase journal with the company expense account; for a bank or cash
    journal the profit and loss accounts are pre-filled with the company cash-difference
    accounts.
 3. On save the completion routine of `entities.md` fills whatever is still missing, creating
    a liquidity account for a bank, cash or credit-card journal.
 4. For a bank journal, supplying a bank account number creates the bank account record,
    attaches it to the company partner, gives it the currency of the journal and marks it as
    trusted for outgoing payments when the user is allowed to do so.
 5. Money-in and money-out method lines are created for the manual methods.
 6. The uniqueness of the code in the company is checked.
```

---

## 7. Recording a miscellaneous entry

**Who:** accountant.

```
 1. The accountant creates an entry. The journal defaults to the first miscellaneous journal
    of the company; when none exists the creation fails with "No journal could be found in
    company *the company display name* for any of those types: general".
 2. The accounting date defaults to today.
 3. The accountant adds items. For each item:
      a. The account defaults to the account of the two previous items of the same display
         type when they agree and the entry has more than two items, otherwise to the default
         account of the journal.
      b. The label, the partner and the currency default as described in `entities.md`.
      c. The balance of a new item defaults to the amount that balances the entry.
      d. Typing into the debit box clears the credit and sets the balance, and conversely.
      e. Typing a negative debit or credit sets the storno flag.
 4. Saving checks the balance invariant, the account and journal coherence and the tax lock
    date of every created item.
 5. The entry stays draft, with the number placeholder, until it is posted.
```

---

## 8. Posting an entry

**Who:** billing clerk or accountant.

```
 1. The user presses the post button. When the abnormal-amount or abnormal-date detection is
    switched on and at least one selected document carries such a warning, a confirmation
    wizard opens instead.
 2. The posting routine runs with hard mode (future entries are posted immediately) when
    triggered from the button, and with soft mode when triggered by another flow.
 3. All guards are evaluated and all failures are reported together (see
    `state-machines.md`).
 4. Entries dated in the future are, in soft mode, scheduled instead of posted.
 5. The accounting date of each remaining entry is shifted out of any locked period.
 6. Analytic lines are created from the analytic distribution of the items.
 7. A recurrence produces its next occurrence.
 8. Partner coherence is enforced on invoice-like documents.
 9. Related draft exchange-difference and cash-basis entries are added to the batch; matches
    whose source document changed are deleted.
10. The state is written to posted and the "posted before" flag is set. Writing the state
    triggers the numbering.
11. Numbering: every entry with no number and with a date takes the next number of its chain,
    under the concurrency discipline of `calculations.md`.
12. Hashing: after the write, the chains of the posted entries are hashed when their journal
    secures posted entries.
13. A posted entry that reverses a posted entry is reconciled with it.
14. Imported matching labels are resolved into real matches.
15. Customer and supplier ranks of the counterparts are increased.
16. A zero-total invoice-like document triggers the "paid" follow-up hook.
17. When the document is a vendor bill captured automatically from a trusted partner and at
    least three consecutive bills of that partner were accepted unmodified, a dialogue offers
    to switch that partner to automatic posting.
```

---

## 9. Posting entries in bulk

**Who:** billing clerk or accountant.

```
 1. The user selects entries in the list, or opens the action from a journal, and runs the
    validate action.
 2. The wizard collects the draft entries of the selection (or of the journal) that have at
    least one item. When there is none: "There are no journal items in the draft state to
    post."
 3. The wizard shows:
      a. a "Force" switch when at least one entry is dated in the future;
      b. a "Force Hash" switch when at least one entry belongs to a hash-secured journal;
      c. the counterparts carrying an abnormal-date warning and the counterparts carrying an
         abnormal-amount warning, each with a switch to silence that warning permanently.
 4. On validation:
      a. silencing a warning writes the corresponding ignore flag on those counterparts;
      b. "Force" sets the automatic posting mode of every selected entry to "No";
      c. when "Force Hash" is off, entries of hash-secured journals are **excluded** from the
         batch;
      d. the remaining entries are posted, in soft mode unless "Force" was ticked.
```

An alternative entry point posts directly the entries that need no confirmation (those not
dated in the future and not in a hash-secured journal) and opens the wizard only for the
others.

---

## 10. Automatic and recurring posting

**Who:** the system, once a day.

```
 1. A scheduled job runs daily. Its default first run is at two o'clock in the morning of the
    day after installation.
 2. It searches the draft entries whose accounting date is on or before today and whose
    automatic posting mode is not "No", limited to a batch of one hundred.
 3. It reports the remaining count to the job framework.
 4. It first tries to post the whole batch at once.
 5. If any entry fails, the transaction is rolled back and the entries are posted one at a
    time:
      a. the entry is locked and re-tested against the search condition;
      b. it is posted;
      c. on failure the transaction is rolled back, the message "The move could not be posted
         for the following reason: *the error message*" is posted on the entry as a comment,
         and the automatic posting mode of that entry is set to "No" so that the job does not
         retry it forever.
 6. Posting an entry whose mode is monthly, quarterly or yearly creates the next occurrence:
      a. the first entry of the recurrence points at itself;
      b. the next date advances by the period length while preserving the day of month of the
         first entry;
      c. nothing is created when the next date is after the end date, or when an occurrence
         of the same recurrence already exists at that date;
      d. the copy keeps the automatic posting mode, the end date, the recurrence origin and
         the salesperson; it shifts the document date by the same rule; and, when there is no
         payment term but a due date, it keeps the same interval between the due date and the
         accounting date.
```

---

## 11. Resetting an entry to draft

**Who:** accountant (and, for an entry marked reviewed, a user allowed to review).

```
 1. The user presses the reset button, which is shown only when the entry is not restricted
    by hashing, carries no hash, and is cancelled or posted without a pending cancellation
    request.
 2. The guards of `state-machines.md` are checked.
 3. The next draft occurrence of the recurrence, if any, is deleted.
 4. The analytic lines of every item are deleted.
 5. The state becomes draft and the sending data is cleared.
 6. For a sale document, the generated printable file is detached: its field link is cleared
    and its name is rewritten as "*the original name* (detached by *the user name* on *the
    date*)*the original extension*", so that a new one can be generated.
 7. The entry keeps its number. Posting it again reuses that number.
```

---

## 12. Cancelling an entry

**Who:** accountant.

```
 1. The user presses the cancel button.
 2. Every posted entry of the selection is first reset to draft, with all the guards of the
    reset.
 3. Any entry that is still not draft stops the operation: "Only draft journal entries can be
    cancelled."
 4. Every reconciliation of the items is undone.
 5. Every payment linked to the entry becomes cancelled.
 6. The automatic posting mode is set to "No" and the state becomes cancelled.
```

A cancelled entry keeps its number and its items so that the numbering chain has no hole and the audit trail stays complete.

---

## 13. Reversing an entry

**Who:** accountant.

```
 1. The user selects one or more posted entries of one company and opens the reversal
    wizard. The wizard refuses a selection spanning several companies or containing a
    non-posted entry.
 2. The user chooses a date (default today), optionally a reason, and a journal (default the
    first active journal of the selection; it must be of the same type).
 3. Two buttons are offered:
      - "Reverse": produce the reverse entries only.
      - "Reverse and Modify" (offered when the selection contains invoice-like documents):
        produce the reverse entries, cancel the originals with them, and additionally create
        a fresh draft copy of each original so the user can correct it.
 4. Default values for each reversal:
      - document type: the reverse of the original type (plain entry reverses to plain entry,
        customer invoice to customer credit note, and so on);
      - the reversal link to the original;
      - the partner of the original;
      - the recipient bank account cleared so that it is recomputed;
      - reference: "Reversal of: *the original number*, *the reason*" when a reason was given,
        otherwise "Reversal of: *the original number*";
      - accounting date and due date: the chosen date;
      - document date: the chosen date for an invoice-like document, otherwise nothing;
      - the chosen journal;
      - the payment term only when the original used a mixed early-discount computation;
      - the salesperson and the origin of the original;
      - automatic posting: "At Date" when the chosen date is in the future, otherwise "No".
 5. The selection is split into two batches:
      - entries that must be cancelled by their reversal: those not scheduled for future
        posting, and either the "Reverse and Modify" mode was chosen or the selection is
        made of plain entries;
      - all the others.
 6. For each batch:
      a. When the batch is the cancelling one, the reconciliations of the originals are first
         undone.
      b. Each original is copied with the default values. The copy carries the reverse type
         and the reversal link.
      c. On the copies, every item of a plain entry and every cost-of-goods-sold item has its
         balance and its foreign amount negated; under storno accounting the storno flag is
         also flipped. Items of invoice-like documents are **not** negated, because the
         reverse document type already carries the opposite sign.
      d. When the batch is the cancelling one, the copies are posted immediately in hard mode
         and then reconciled with the originals: the items of the original and of the reversal
         that are not yet matched are grouped by (account, currency), receivable and payable
         accounts first, and each group whose account allows matching — or is a liquidity
         account — is reconciled.
      e. A message is logged on each original: "This entry has been *link labelled "reversed"*".
      f. In the "Reverse and Modify" mode, a fresh draft copy of each original is created,
         keeping only the product, section, subsection and note items, dated at the chosen
         date, keeping the origin, and keeping a copy of the main attachment when the original
         was a purchase document.
 7. The user is redirected to the produced entries.
```

---

## 14. Deleting an entry

**Who:** accountant or accounting administrator.

```
 1. The gap flags of the neighbours are refreshed first, with the entry treated as invalidated.
 2. The chain-end guard runs: unless the user is an accounting administrator, or the company is
    in quick-encoding mode, or the deletion is explicitly forced, the entries must be the last
    ones of their numbering chain. Otherwise: "You cannot delete this entry, as it has already
    consumed a sequence number and is not the last one in the chain. You should probably revert
    it instead."
 3. The audit-trail guard runs: when the company keeps a restrictive audit trail, an entry that
    was posted at least once may not be deleted at all. Otherwise: "To keep the restrictive
    audit trail, you can not delete journal entries once they have been posted.\nInstead, you
    can cancel the journal entry."
 4. When the deletion is forced, an administration log line records the entries, their totals,
    their counterparts and the balance per account.
 5. The reconciliations of the items are undone.
 6. The items are deleted, which itself checks that no item belongs to a hashed entry, that no
    non-zero item belongs to a posted entry, that no tax item is removed while the entry still
    carries taxes, and that no payment-term item is removed.
 7. The entries are deleted.
```

### The automatic choice between deleting, cancelling and reversing

Some flows need to undo an entry without knowing whether it may be deleted. The rule is:

```
For each entry:
  - it can be unlinked when it carries no hash, its date is strictly after the fiscal lock
    date that applies to the user and the journal, it is not a posted cash-basis entry and
    it is not a posted exchange-difference entry;
  - among those, the ones protected by a restrictive audit trail (posted at least once in a
    company that keeps one) are cancelled instead of deleted;
  - the others are deleted, after being reset to draft when needed;
  - every entry that cannot be unlinked is reversed with a cancelling reversal.
```

---

## 15. Matching journal items

**Who:** accountant, or the system.

```
 1. The user selects items — from the journal-item list, from the outstanding-payments widget
    of an invoice, or from the bank reconciliation screen — and runs the reconcile action.
 2. The request is turned into a plan (see `calculations.md`), the items are sorted and split
    by currency, and the preconditions are checked.
 3. Around the whole operation, the balance invariant and the dynamic-line synchronisation of
    the affected entries are suspended and re-checked at the end.
 4. The payment status of every invoice-like document touched is remembered, so that the
    "became paid" hook can fire at the end.
 5. The residuals of every item are read once into memory and updated in memory as the matches
    are computed, so that the matches can be created in one operation.
 6. The pairing loop produces the list of matches and the list of exchange differences.
 7. The matches are created. Payments whose amount is now covered become "paid".
 8. The exchange-difference entries are created and, when both matched items belong to posted
    entries, posted; each is linked back to the match that produced it.
 9. Cash-basis entries are created for the matches on receivable and payable accounts when the
    company uses cash-basis taxes, unless the operation is a cancelling reversal or cash-basis
    creation is suppressed.
10. The connected groups are identified and, for each group that nets to zero, a Full
    Reconciliation is created; the matching numbers are recomputed.
11. When a cash-basis rounding item was created on a transition account, it is reconciled with
    the matching items of the cash-basis entries so that the transition account closes too.
12. The "became paid" hook fires for the documents that reached the paid or in-payment status.
```

### The three usual entry points

| Entry point | Plan |
|---|---|
| The reconcile action on a selection of items | one node containing the selection |
| Attaching one outstanding item to an invoice | one node containing that item plus every unmatched item of the invoice on the same account |
| The automatic matching of a reversal with its original | one node per (account, currency) pair, receivable and payable accounts first |

---

## 16. Undoing a match

**Who:** accountant.

```
 1. The user runs the unreconcile action on one or more items, or on a whole matched group.
 2. Every match in which those items participate is deleted.
 3. The side effects listed in `business-rules.md` run: payments revert to "in process", the
    Full Reconciliation is deleted, the cash-basis and exchange-difference entries are reversed
    or deleted, and the matching numbers are recomputed.
 4. The residuals of the items are recomputed and the payment status of the touched documents
    is recomputed.
```

The action that unreconciles "matched entries" from a list first expands the selection to every item transitively matched with the selected ones, so that the whole group is undone at once.

---

## 17. Moving an amount to another account

**Who:** accountant.

```
 1. The accountant selects posted, unreconciled journal items of one company hierarchy and
    opens the automatic transfer wizard with the "Change Account" action.
 2. The accountant chooses the destination account, the journal (default the company
    automatic-entry journal, restricted to miscellaneous journals) and the date (default
    today).
 3. A preview shows the entry that will be created, with the account, the label, the
    counterpart and the two amount columns; at most four entries are previewed and the rest is
    summarised as "*the count* moves".
 4. On confirmation the entry described in `calculations.md` is created and posted.
 5. The original items are reconciled with the mirror items on each source account that allows
    matching, per counterpart and per currency; items already on the destination account are
    reconciled with the counterpart items when the destination account allows matching.
 6. A message is logged on each source entry:
      "*the amount* (*D or C*) from **the source account display name** were transferred to
      **the destination account display name** by *link to the transfer entry*",
    as a bulleted list with one line per source account.
 7. A message is logged on the transfer entry:
      "This entry transfers the following amounts to **the destination account display name**"
    followed by a bulleted list of "*the amount* (*D or C*) from *link to the source entry*,
    **the source account display name**".
```

---

## 18. Moving an amount to another period

**Who:** accountant.

```
 1. The accountant selects posted, unreconciled journal items whose accounts all share one
    account type, and opens the automatic transfer wizard with the "Change Period" action.
 2. The wizard deduces the nature of the transfer from the sign of the selected balances:
    revenue when the total is negative, expense otherwise; and shows the corresponding accrual
    account of the company, which the accountant may change (the change is stored back on the
    company).
 3. The accountant sets the percentage (or the total amount, which is the same value expressed
    differently) and the target date.
 4. Choosing a locked target date is refused: "The date selected is protected by: *the list of
    lock dates*."
 5. A preview shows the entries that will be created.
 6. On confirmation:
      a. one destination entry is created at the chosen date, carrying two items per selected
         item as described in `calculations.md`;
      b. one cancelling entry is created per distinct lock-safe source date, carrying the
         mirrored pair of each item whose source date maps to it;
      c. all of them are posted;
      d. the mirror items are reconciled pairwise: the items are grouped by (label, account,
         counterpart, currency, analytic distribution), which naturally pairs each
         destination item with its cancellation, and each group on a reconcilable account with
         a non-zero balance is reconciled.
 7. A message is logged on each source entry:
      "Adjusting Entries have been created for this invoice:" followed by two bullets:
      "*link to the cancelling entry* cancelling *the percentage*% of *the amount*" and
      "*link to the destination entry* postponing it to *the target date*".
 8. A message is logged on the destination entry, one line per source entry:
      "Adjusting Entry *link* *the percentage*% of *the amount* recognized from *the source
      date*".
 9. A message is logged on each cancelling entry, one line per source entry:
      "Adjusting Entry *link* *the percentage*% of *the amount* recognized on *the target
      date*".
10. The user is redirected to the created entries.
```

---

## 19. Renumbering a set of entries

**Who:** accounting administrator (the action is visible in developer mode).

```
 1. The administrator selects entries of one journal and opens the resequence wizard.
 2. The wizard refuses a selection spanning journals, or mixing credit notes with other
    documents in a journal with a dedicated credit-note numbering, or mixing payment entries
    with other entries in a journal with a dedicated payment numbering.
 3. The first number defaults to the smallest current number of the selection; the periodicity
    is deduced from it and shown.
 4. The administrator chooses "Keep current order" or "Reorder by accounting date".
 5. A preview table shows, for each entry, the current number and the number it would get
    under each ordering; rows in the middle of a long run are collapsed.
 6. On confirmation the algorithm of `calculations.md` runs: every selected number is cleared
    and flushed, then the new numbers are written one entry at a time.
 7. Reordering by date in a hash-secured journal is refused.
```

---

## 20. Closing a period with a lock date

**Who:** accounting administrator.

```
 1. The administrator opens the accounting settings and sets one or more lock dates.
 2. Before the write, the validation runs:
      a. the Hard Lock Date may not be removed nor moved backwards;
      b. when a Hard Lock Date is set, no draft entry may exist on or before it; otherwise the
         operation is refused with a button leading to the offending entries;
      c. when a Global or Hard Lock Date is set, no unreconciled bank transaction may exist on
         or before the resulting fiscal lock date; otherwise the operation is refused with a
         button leading to those transactions.
 3. The write happens and is tracked in the company audit trail.
 4. The cached per-user lock dates are invalidated.
 5. Every active exception on a changed soft lock date is revoked and recreated against the
    new company value, so that the trace shows which company value each exception relaxed.
```

The practical closing sequence an accountant follows is: post or delete every draft entry of the period; reconcile every bank transaction of the period; set the Global Lock Date at the end of the period; set the Tax Return Lock Date when the return is filed (this happens automatically when a tax closing entry is posted); and, once the year is audited, set the Hard Lock Date, which can never be undone.

---

## 21. Granting a lock date exception

**Who:** accounting administrator.

```
 1. From the lock-date settings, the administrator opens the exception dialogue.
 2. The administrator chooses which lock date to relax, the value to relax it to (an empty
    value removes the lock date entirely for the beneficiary), the beneficiary (a named user,
    or nobody meaning everybody), an end moment (empty means forever) and a reason.
 3. On creation the exception records the current company value of that lock date.
 4. A message is posted on the company audit trail naming the exception, the beneficiary, the
    validity and the reason, with a tracked value showing the lock date moving from the company
    value to the exception value.
 5. The beneficiary can now record entries in the relaxed window; every other user is still
    blocked.
 6. The administrator can, at any time, open the exception and inspect the journal items that
    were created or modified while it was in force: the audit query selects the entries of the
    company (and its children) whose audit trail carries a message created after the exception
    was granted (and before its end moment), by the beneficiary when one was named, and whose
    accounting date lies between the relaxed value and the original company value — or whose
    accounting date was changed from or to a date in that window.
 7. Revoking the exception sets it inactive and stamps the end moment.
```

---

## 22. Securing entries with the hash chain

Three paths lead to the same routine.

### Automatically at posting

```
 1. The journal has the "Secure Posted Entries with Hash" switch on.
 2. Posting writes the state and then runs the securing routine on the posted entries.
 3. The chains are selected, the warnings are evaluated as refusals, and every entry from the
    last already hashed one up to the newly posted one receives a hash.
```

### On demand for one entry

```
 1. The accountant presses the secure button on an entry (visible when the inalterability
    features are shown).
 2. The same routine runs with hashing forced, so the journal setting is ignored.
 3. When the journal does not secure by default, the group that shows the inalterability
    features is activated for everybody, so that the hash becomes visible.
```

### In bulk up to a date

```
 1. The administrator opens the secure-entries wizard.
 2. The wizard proposes the highest date up to which everything is already secured, or today.
 3. For the chosen date it computes:
      - the chains that would be hashed and the entries in them;
      - the chains that contain an unreconciled bank transaction, which are excluded entirely;
      - the entries that cannot be hashed and are not protected by the Hard Lock Date;
      - the chains whose hashing would leave a numbering gap;
      - the entries beyond the chosen date that would be hashed anyway because they are in the
        middle of a selected chain.
 4. Each of those produces a warning with a button leading to the records concerned.
 5. On confirmation the routine runs with hashing forced and with the gap refusal disabled.
```

---

## 23. Verifying the hash chain

**Who:** accountant or auditor with the full accounting features.

```
 1. The user prints the hash integrity report for a company.
 2. Running it without the accounting user group is refused.
 3. For each journal of the company, the hashed entries are read in the order described in
    `calculations.md` and their hashes are recomputed, retrying older hash versions when the
    current one does not match.
 4. The report lists, per journal and numbering prefix:
      - the journal name, with the prefix in parentheses;
      - whether the journal secures posted entries, shown as a tick or a cross;
      - the status: verified, corrupted or no data;
      - for a verified chain: the first entry number, its hash and its date, and the same for
        the last entry;
      - for a corrupted chain: "Corrupted data on journal entry with id *the identifier*
        (*the number*)."
 5. The report also carries the printing date.
```

---

## 24. Closing a fiscal year

The domain does not ship a dedicated year-closing routine; the closing is the combination of the mechanisms above, and the profit-or-loss appropriation is produced by the reports rather than by a stored entry.

```
 1. The accountant posts every entry of the year and reconciles every bank transaction.
 2. The accountant reviews the balance of the current-year-earnings account: accounts whose
    type carries the balance forward are accumulated from the beginning of time, while income
    and expense accounts and the current-year-earnings account restart at each fiscal year.
    The profit or loss of the closing year is therefore reported by the balance sheet as the
    aggregate of the income and expense accounts, and is posted to the current-year-earnings
    account only when the accountant records an appropriation entry by hand.
 3. The accountant records that appropriation entry in a miscellaneous journal, debiting or
    crediting the current-year-earnings account against the retained-earnings account of the
    chart. **Industry-standard default**: the entry is dated on the last day of the fiscal
    year and its counterpart is an account of type Equity; the system does not create it.
 4. The accountant sets the Global Lock Date at the last day of the fiscal year.
 5. Once the statutory audit is finished, the accounting administrator sets the Hard Lock Date
    at the same day. From then on nothing in the year can change, and no exception can be
    granted.
 6. Optionally, the administrator runs the secure-entries wizard up to that date so that every
    entry of the year carries a hash.
```

The fiscal year boundaries themselves are computed by the rule of `calculations.md` from the fiscal-year end day and month of the company.

---

## 25. The accounting onboarding path

**Who:** accounting administrator, guided by a checklist on the accounting dashboard.

```
 1. Creating a company starts the accounting onboarding for it.
 2. The checklist attached to the accounting dashboard carries three steps, in this order:

      | Order | Title | Description | Button | Text once done |
      |---|---|---|---|---|
      | 1 | Set Company Data | Set your company's data for documents header/footer. | Let's start! | Looks great! |
      | 1 | Set Periods | Define your fiscal years & tax returns periodicity. | Configure | Step completed! |
      | 4 | Review Chart of Accounts | Set up your chart of accounts and record initial balances. | Review Accounts | Chart of accounts set! |

 3. Two further steps are shipped by the same package but are not attached to that checklist;
    a companion package may add them to one of its own panels:

      | Order | Title | Description | Button | Text once done |
      |---|---|---|---|---|
      | 3 | Documents Layout | Customize the look of your documents. | Customize | Looks great! |
      | 100 | Taxes | Choose a default sales tax for your products. | Set taxes | Step Completed! |

 4. The company-data step is marked done automatically as soon as the street of the company is
    filled; the tax step is marked done when the user validates it. Each step can also be
    skipped or reopened, and the progress is stored per company.
```

The bank setup dialogue is also reachable from the dashboard button of a bank journal that has no statement source configured, and from the financial configuration menu, in two variants: a bank account and a credit-card account.
