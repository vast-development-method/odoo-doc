# General Ledger — Acceptance Criteria

Numbered scenarios in Given / When / Then form with concrete numbers. An implementation of this domain is correct when every scenario below produces the stated result.

Unless a scenario says otherwise:

- the company currency is the euro with a rounding step of 0.01;
- the foreign currency called *the foreign currency* has a rounding step of 0.01 and a rate of **3 units per euro**;
- the second foreign currency called *the other foreign currency* has a rounding step of 0.01 and a rate of **4 units per euro**;
- the fiscal year ends on the thirty-first of December;
- no lock date is set;
- the acting user belongs to the Invoicing group and, where the scenario needs it, to the Administrator group.

Contents:

1. [Accounts](#1-accounts)
2. [Account groups and roots](#2-account-groups-and-roots)
3. [Journals](#3-journals)
4. [The balance invariant](#4-the-balance-invariant)
5. [Debit, credit and storno](#5-debit-credit-and-storno)
6. [Posting](#6-posting)
7. [Numbering](#7-numbering)
8. [Gap detection](#8-gap-detection)
9. [Resequencing](#9-resequencing)
10. [The accounting date rule](#10-the-accounting-date-rule)
11. [Lock dates](#11-lock-dates)
12. [Lock exceptions](#12-lock-exceptions)
13. [The hash chain](#13-the-hash-chain)
14. [Reset to draft, cancel and delete](#14-reset-to-draft-cancel-and-delete)
15. [Reversal](#15-reversal)
16. [Automatic and recurring posting](#16-automatic-and-recurring-posting)
17. [Reconciliation in the company currency](#17-reconciliation-in-the-company-currency)
18. [Reconciliation in a foreign currency](#18-reconciliation-in-a-foreign-currency)
19. [Exchange differences](#19-exchange-differences)
20. [Matching numbers](#20-matching-numbers)
21. [Unreconciliation](#21-unreconciliation)
22. [The opening entry](#22-the-opening-entry)
23. [Automatic transfers](#23-automatic-transfers)
24. [Multi-company](#24-multi-company)
25. [Access rights](#25-access-rights)
26. [Rounding edge cases](#26-rounding-edge-cases)

---

## 1. Accounts

**1.1 — Creating an account inherits the type of the preceding code.**
Given a chart in which the account `400000` has the type Payable and the account `600000` has the type Expenses,
When a user creates an account with the code `400100` and no explicit type,
Then the type of the new account is Payable.

**1.2 — Creating an account before every existing code falls back to current assets.**
Given a chart whose smallest code is `100000`,
When a user creates an account with the code `000001` and no explicit type,
Then the type of the new account is Current Assets.

**1.3 — A receivable account must be reconcilable.**
Given a new account with the type Receivable,
When the user clears the reconcilable flag and saves,
Then the save is refused with "You cannot have a receivable/payable account that is not reconcilable. (account code: *the code*)".

**1.4 — Changing the type to Receivable forces the reconcilable flag on.**
Given an account of type Current Assets with the reconcilable flag off,
When the type is changed to Receivable,
Then the reconcilable flag becomes true.

**1.5 — Changing the type between two neutral asset types keeps the flag.**
Given an account of type Current Assets with the reconcilable flag **on**,
When the type is changed to Non-current Assets,
Then the reconcilable flag stays on.

**1.6 — An off-balance account cannot be reconcilable.**
Given an account of type Off-Balance Sheet,
When the user sets the reconcilable flag,
Then the save is refused with "An Off-Balance account can not be reconcilable".

**1.7 — Duplicate codes in a company hierarchy are refused.**
Given a parent company holding the account `701000`,
When a user creates an account `701000` in a child company of it,
Then the save is refused with "Account codes must be unique. You can't create accounts with these duplicate codes: 701000".

**1.8 — The same code may exist in two unrelated companies.**
Given two companies with no ancestor in common,
When each of them gets an account `701000`,
Then both saves succeed and the two accounts are distinct records.

**1.9 — One account, two codes.**
Given an account belonging to a parent company with the code `701000` and to its subsidiary with the code `70100`,
When a user of the subsidiary opens the chart of accounts,
Then the account is displayed with the code `70100`;
And when a user of the parent company opens it, the code shown is `701000`.

**1.10 — The free-code walk increments the digits.**
Given the accounts `102100`, `102101` and `102102` already exist,
When a free code is requested starting from `102100`,
Then `102103` is returned.

**1.11 — The free-code walk falls back to a copy suffix.**
Given every code from `10.01.98` to `10.01.99` already exists,
When a free code is requested starting from `10.01.97`,
Then `10.01.97.copy2` is returned (the walk tried `10.01.98` and `10.01.99` first).

**1.12 — The free-code walk on a code with no digits.**
When a free code is requested starting from `hello` and `hello` is taken,
Then `hello.copy` is returned.

**1.13 — An invalid code is refused.**
When a user saves an account with the code `40-100`,
Then the save is refused with "The account code can only contain alphanumeric characters and dots. (account code: 40-100)".

**1.14 — An account with items cannot be deleted.**
Given an account that carries one journal item,
When the user deletes it,
Then the deletion is refused with "You cannot perform this action on an account that contains journal items."

**1.15 — Typing a code inside the name splits it.**
When a user types `701000 Product Sales` into the name box of a new account whose code is empty,
Then the code becomes `701000` and the name becomes `Product Sales`.

**1.16 — Switching reconciliation off with pending matches is refused.**
Given an account carrying an item that participates in a partial match and has no full reconciliation,
When the reconcilable flag is cleared,
Then the operation is refused with "You cannot switch an account to prevent the reconciliation if some partial reconciliations are still pending."

**1.17 — Switching reconciliation on rewrites the residuals.**
Given an account with the reconcilable flag off carrying an item with a balance of 500.00, a foreign amount of 1 500.00 and residuals of 0.00,
When the reconcilable flag is set,
Then the residual in the company currency becomes 500.00, the residual in the item currency becomes 1 500.00 and the reconciled flag becomes false.

**1.18 — A bank-and-cash account cannot be shared.**
When a user assigns two companies to an account of type Bank and Cash,
Then the save is refused with "Bank & Cash accounts cannot be shared between companies."

**1.19 — Unmerging preserves the codes.**
Given an account shared by a parent company (code `550000`) and a subsidiary (code `55000`), each holding journal items,
When the account is unmerged,
Then two accounts exist, one per company, with the codes `550000` and `55000`;
And every journal item still reports the same code it reported before;
And each new account carries a message "This account was split off from *link* (*the parent company name*)."

---

## 2. Account groups and roots

**2.1 — Prefix lengths must match.**
When a group is created with a start prefix `40` and an end prefix `4199`,
Then the save is refused with "The length of the starting and the ending code prefix must be the same".

**2.2 — A single prefix fills both ends.**
When a group is created with a start prefix `40` and an empty end prefix,
Then the end prefix becomes `40`.

**2.3 — Overlapping groups of the same length are refused.**
Given a group with the prefixes `40` to `41`,
When another group of the same company is created with the prefixes `41` to `42`,
Then the save is refused with "Account Groups with the same granularity can't overlap".

**2.4 — Parenting is derived from the prefixes.**
Given a group A with the prefixes `4` to `4` and a group B with the prefixes `40` to `41`,
When the hierarchy is rebuilt,
Then the parent of B is A and A has no parent.

**2.5 — The most specific group wins.**
Given a group A with the prefixes `4` to `4`, a group B with the prefixes `40` to `41` and an account `400100`,
Then the group of the account is B.

**2.6 — The display name of a group.**
Given a group named `Suppliers` with the prefixes `40` to `41`,
Then its display name is `40-41 Suppliers`;
And a group named `Suppliers` with both prefixes equal to `40` displays as `40 Suppliers`.

**2.7 — Deleting a group re-parents its children.**
Given A is the parent of B and B is the parent of C,
When B is deleted,
Then the parent of C becomes A.

**2.8 — The root of an account.**
Given the account code `701000`,
Then its root is `70`, and the parent of that root is `7`.

---

## 3. Journals

**3.1 — The default code of a new sale journal.**
Given the company already has the sale journals with the codes `INV1` and `INV2`,
When the user creates a sale journal,
Then the proposed code is `INV3`.

**3.2 — The name placeholder.**
Given a new purchase journal whose proposed code is `BILL2`,
Then the name placeholder is `Vendor Bills (2)`.

**3.3 — The code is unique per company.**
When a second journal of the same company is saved with the code `INV1`,
Then the save is refused with "Journal codes must be unique per company."

**3.4 — Duplicating a journal invents a code.**
Given a journal with the code `BNK1` and the name `Bank`,
When it is duplicated and the codes `BNK1` and `BNK2` are taken,
Then the copy has a code built from the letters of the original code and the first free counter, and the name `Bank (copy)`.

**3.5 — Creating a bank journal creates a liquidity account.**
Given the company has a bank-account code prefix of `5500` and six-character codes,
When a bank journal is created with no default account,
Then an account of type Bank and Cash is created with the first free code from `550000` and it becomes the default account of the journal.

**3.6 — Archiving a journal with draft entries is refused.**
Given a journal holding one draft entry,
When the journal is archived,
Then the operation is refused with the four-line message beginning "You can not archive a journal containing draft journal entries."

**3.7 — Switching hashing off after hashing is refused.**
Given a journal that secures posted entries and holds one hashed entry,
When the switch is cleared,
Then the operation is refused with "You cannot modify the field Secure Posted Entries with Hash of a journal that already has accounting entries."

**3.8 — Changing the company of a journal with entries is refused.**
Given a journal holding one entry,
When its company is changed,
Then the operation is refused with "You can't change the company of your journal since there are some journal entries linked to it."

**3.9 — The display name carries a foreign currency.**
Given a bank journal named `Bank Foreign` whose currency is the foreign currency and whose company currency is the euro,
Then the display name is `Bank Foreign (*the foreign currency name*)`.

**3.10 — The foreign currency propagates to the liquidity account.**
Given a bank journal with a default account and no currency,
When the journal currency is set to the foreign currency,
Then the currency of the default account becomes the foreign currency.

---

## 4. The balance invariant

**4.1 — A balanced entry saves.**
Given a miscellaneous entry with an item of +1 250.00 and an item of −1 250.00,
When it is saved,
Then the save succeeds.

**4.2 — An unbalanced entry is refused.**
Given a miscellaneous entry with an item of +1 250.00 and an item of −1 240.00,
When it is saved,
Then the save is refused with "The entry is not balanced."

**4.3 — Several unbalanced entries are reported together.**
Given two entries `MISC/2026/03/0001` and `MISC/2026/03/0002` are written in one operation and both are unbalanced,
Then the save is refused with "The following entries are unbalanced:" followed by a blank line, then "  - MISC/2026/03/0001" and "  - MISC/2026/03/0002".

**4.4 — An imbalance in the foreign currency is tolerated.**
Given an entry with an item of +100.00 euros and 300.00 foreign units and an item of −100.00 euros and 250.00 foreign units — that is, a foreign amount of −250.00 —,
When it is saved,
Then the save succeeds, because the invariant only applies to the company currency.

**4.5 — A rounding difference below the step does not break the balance.**
Given a company currency with a rounding step of 0.01 and items whose balances sum to 0.004,
Then the entry is considered balanced, because the sum rounded to 0.01 is 0.00.

**4.6 — The default balance of a new item balances the entry.**
Given a draft entry with one item of +1 250.00,
When a second item is added with no amount,
Then its proposed balance is −1 250.00.

---

## 5. Debit, credit and storno

**5.1 — A positive balance is a debit.**
Given an item whose balance is 300.00 in a company that does not use storno accounting,
Then its debit is 300.00 and its credit is 0.00.

**5.2 — A negative balance is a credit.**
Given an item whose balance is −300.00,
Then its debit is 0.00 and its credit is 300.00.

**5.3 — Typing into the credit box.**
Given an empty item,
When 300.00 is typed into the credit box,
Then the debit becomes 0.00 and the balance becomes −300.00.

**5.4 — A negative debit marks the item as storno.**
Given a company that uses storno accounting,
When −300.00 is typed into the debit box,
Then the storno flag is set, the balance becomes −300.00, the debit becomes −300.00 and the credit becomes 0.00.

**5.5 — The sign coherence check.**
When an item is saved with a balance of 100.00 and a foreign amount of −300.00,
Then the save is refused with "The amount expressed in the secondary currency must be positive when account is debited and negative when account is credited. If the currency is the same as the one from the company, this amount must strictly be equal to the balance."

**5.6 — A section carries no amount.**
When a section item is saved with a balance of 100.00,
Then the save is refused with "Forbidden balance or account on non-accountable line".

**5.7 — An accountable item needs an account.**
When a product item is saved with no account,
Then the save is refused with "Missing required account on accountable line."

---

## 6. Posting

**6.1 — The happy path.**
Given a balanced draft miscellaneous entry dated 5 March 2026 in a journal with the code `MISC` that has no earlier entry,
When it is posted,
Then its state is posted, its number is `MISC/2026/03/0001`, its "posted before" flag is true, and it is included in every report.

**6.2 — An entry with no accountable item.**
Given a draft entry whose only item is a note,
When it is posted,
Then the posting is refused with "Even magicians can't post nothing!"

**6.3 — An archived account blocks posting.**
Given a draft entry using an archived account,
When it is posted,
Then the posting is refused with "A line of this move is using a archived account, you cannot post it."

**6.4 — An archived journal blocks posting.**
Given a draft entry in an archived journal named `Old Misc`,
When it is posted,
Then the posting is refused with "You cannot post an entry in an archived journal (Old Misc)".

**6.5 — Several failures are reported together.**
Given a draft entry using an archived account **and** in an archived journal,
When it is posted,
Then both messages appear, one per line, in a single refusal.

**6.6 — Posting a posted entry.**
Given a posted entry numbered `MISC/2026/03/0001` with the identifier 42,
When it is posted again,
Then the posting is refused with "The entry MISC/2026/03/0001 (id 42) must be in draft."

**6.7 — A future entry in soft mode is scheduled.**
Given today is 10 March 2026 and a draft entry is dated 1 April 2026,
When it is posted in soft mode,
Then it stays draft, its automatic posting mode becomes "At Date" and the message "This move will be posted at the accounting date: 04/01/2026" is logged.

**6.8 — A future entry in hard mode is posted.**
Given the same entry with the automatic posting mode "No",
When it is posted from the post button (hard mode),
Then it is posted immediately and its accounting date stays 1 April 2026.

**6.9 — A future entry already scheduled cannot be forced by the button.**
Given the same entry with the automatic posting mode "At Date",
When it is posted in hard mode,
Then the posting is refused with "This move is configured to be auto-posted on 04/01/2026".

**6.10 — Posting an entry of another company's account.**
Given an entry of company A using an account that belongs only to company B,
When it is posted,
Then the posting is refused with "The entry is using accounts (*the account display names*) from a different company."

**6.11 — Posting without the permission.**
Given a user who belongs only to the Show Accounting Features - Readonly group,
When that user posts an entry,
Then the posting is refused with "You don't have the access rights to post an invoice."

---

## 7. Numbering

**7.1 — The starting number of a miscellaneous journal.**
Given a journal with the code `MISC`, no earlier entry, a fiscal year ending 31 December and an entry dated 5 March 2026,
When the entry is posted,
Then its number is `MISC/2026/03/0001`.

**7.2 — The starting number of a sale journal.**
Given the same conditions in a sale journal with the code `INV`,
Then the number is `INV/2026/00001`.

**7.3 — The starting number with a staggered fiscal year.**
Given a fiscal year ending 15 April, a purchase journal with the code `BILL` and a vendor bill dated 17 April 2024,
When it is posted,
Then its number is `BILL/24-25/04/0001`.

**7.4 — The same month on the other side of the fiscal boundary.**
Given a fiscal year ending 15 April and a purchase journal with the code `BILL` holding no entry yet,
When a draft bill dated 17 April 2024 is prepared, its proposed number is `BILL/24-25/04/0001` (nothing is consumed while it stays draft);
And when a bill dated 10 April 2024 is posted, its number is `BILL/23-24/04/0001`;
And when a bill dated 11 April 2024 is posted, its number is `BILL/23-24/04/0002`;
And when a bill dated 18 April 2024 is posted, its number is `BILL/24-25/04/0001`, because the month of April is cut in two by the fiscal-year boundary and the second half is the first month of the new fiscal year.

**7.5 — The counter increments within a chain.**
Given `MISC/2026/03/0001` exists and is posted,
When a second entry dated in March 2026 is posted in the same journal,
Then its number is `MISC/2026/03/0002`.

**7.6 — A new month restarts a monthly chain.**
Given `MISC/2026/03/0007` is the highest March number,
When an entry dated 2 April 2026 is posted,
Then its number is `MISC/2026/04/0001`.

**7.7 — A new year restarts a yearly chain.**
Given `INV/2026/00042` is the highest number,
When an invoice dated 3 January 2027 is posted,
Then its number is `INV/2027/00001`.

**7.8 — Format inference, twelve cases.**
Given the previous number of the chain is the value in the first column, and three entries are posted: one in the same month, one in the next month and one in the next year,
Then the three numbers are:

| Previous number | Same month | Next month | Next year |
|---|---|---|---|
| `JRNL/2016/00001` | `JRNL/2016/00002` | `JRNL/2016/00003` | `JRNL/2017/00001` |
| `JRNL/2015-2016/00001` | `JRNL/2015-2016/00002` | `JRNL/2016-2017/00001` | `JRNL/2016-2017/00002` |
| `JRNL/2015-16/00001` | `JRNL/2015-16/00002` | `JRNL/2016-17/00001` | `JRNL/2016-17/00002` |
| `JRNL/15-16/00001` | `JRNL/15-16/00002` | `JRNL/16-17/00001` | `JRNL/16-17/00002` |
| `1234567` | `1234568` | `1234569` | `1234570` |
| `20190910` | `20190911` | `20190912` | `20190913` |
| `2016-0910` | `2016-0911` | `2016-0912` | `2017-0001` |
| `201603-10` | `201603-11` | `201604-01` | `201703-01` |
| `16-03-10` | `16-03-11` | `16-04-01` | `17-03-01` |
| `2016-10` | `2016-11` | `2016-12` | `2017-01` |
| `045-001-000002` | `045-001-000003` | `045-001-000004` | `045-001-000005` |
| `JRNL/2016/00001suffix` | `JRNL/2016/00002suffix` | `JRNL/2016/00003suffix` | `JRNL/2017/00001suffix` |

(The reference entries are dated 12 March 2016, 12 April 2016 and 12 March 2017, and the fiscal year runs from April to March for the year-range rows.)

**7.9 — A short counter grows instead of overflowing.**
Given the previous number is `TEST_ORDER/2016/1`,
When twenty-three further entries are posted,
Then the numbers run `TEST_ORDER/2016/2` … `TEST_ORDER/2016/24` without padding and without failure.

**7.10 — The prefix of the newest entry wins, not the alphabetically greatest.**
Given `INV/2016/00001` and `INV/2016/00002` exist and the newest entry of the chain has been renamed `FACT/2016/00001`,
When a new entry is posted,
Then its number is `FACT/2016/00002`.

**7.11 — Ordering when posting several entries at once.**
Given the highest number is `XMISC/2016/00001` and six draft entries are created with the dates 5, 6, 7, 4, 5 and 5 March 2019 (in that order of creation), the first one having its number cleared,
When all six are posted in one operation,
Then the numbers are, in the order of creation: `XMISC/2019/00002`, `XMISC/2019/00005`, `XMISC/2019/00006`, `XMISC/2019/00001`, `XMISC/2019/00003`, `XMISC/2019/00004` — the assignment follows the accounting date, then the reference, then the creation order.

**7.12 — Two entries cannot share a number.**
Given `XMISC/2019/00001` is posted,
When a second posted entry of the same journal is renamed `XMISC/2019/00001`,
Then the write is refused by the uniqueness constraint with "Another entry with the same name already exists."

**7.13 — A journal may switch from yearly to monthly.**
Given a chain with `MISC/00001` in 2016, then a manual `MISC/2017/00001` in 2017, then a manual `MISC/2017/02/00001` in February 2017,
Then a new entry dated in 2016 gets `MISC/00002`, a new entry dated in January 2017 gets `MISC/2017/00002` and a new entry dated in February 2017 gets `MISC/2017/02/00002`.

**7.14 — Introducing a fiscal-year prefix changes the following chain.**
Continuing the previous scenario, given a manual `MISC/2016-2017/00001` is added in March 2017,
Then a new entry dated in 2016 gets `MISC/00003`, a new entry dated in January 2017 now gets `MISC/2016-2017/00002`, a new entry dated in February 2017 still gets `MISC/2017/02/00003`, and a new entry dated in March 2017 gets `MISC/2016-2017/00003`.

**7.15 — A monthly shape wins over a two-digit year range.**
Given the previous number is `MISC/01-02/00001` dated 1 February 2101,
When an entry dated 1 March 2101 is posted,
Then its number is `MISC/01-03/00001` — the shape is read as monthly (year 01, month 02), not as a year range.

**7.16 — A journal pattern override forces the year-range reading.**
Continuing the previous scenario, given the journal pattern is set to the year-range shape and the previous number is `MISC/00-01/00001`,
When an entry dated 1 March 2101 is posted,
Then its number is `MISC/00-01/00002`.

**7.17 — A non-numeric number gets a counter appended.**
Given the previous number of a chain is `N'importe quoi?`,
When a further entry is posted,
Then its number is `N'importe quoi?1`.

**7.18 — Credit notes have their own chain.**
Given a sale journal with a dedicated credit-note numbering and the invoice `INV/2026/00005`,
When a customer credit note dated in 2026 is posted,
Then its number is `RINV/2026/00001`, and the next invoice still gets `INV/2026/00006`.

**7.19 — A date that no longer matches the number clears it.**
Given a draft entry, never posted, numbered `MISC/2026/03/0001`,
When its accounting date is changed to 5 April 2026,
Then the number is cleared and a fresh April number is taken at posting.

**7.20 — A date that no longer matches on a posted entry is refused.**
Given a posted entry numbered `MISC/2026/03/0001`,
When its accounting date is changed to 5 April 2026,
Then the change is refused with "The Date (04/05/2026) you've entered isn't aligned with the existing sequence number (MISC/2026/03/0001). Clear the sequence number to proceed." followed by the second sentence about resequencing.

**7.21 — The journal of a numbered entry cannot change.**
Given a posted entry numbered `MISC2/2026/03/0001` with a counter of 1 that was posted before,
When its journal is changed without clearing the number,
Then the change is refused with "You cannot edit the journal of an account move if it has been posted once, unless the name is removed or set to "/". This might create a gap in the sequence."

**7.22 — Clearing the number allows the journal change.**
Given the same entry,
When the number is set to the placeholder and the journal is changed in the same operation,
Then the change succeeds and a number of the new journal is taken at the next posting.

**7.23 — Concurrency.**
Given two transactions post an entry of the chain `INV/2026/` whose highest number is `INV/2026/00007`,
Then one transaction obtains `INV/2026/00008` and the other `INV/2026/00009`; no number is skipped and none is duplicated.

**7.24 — Several numbers in one transaction.**
Given three entries of the same chain are posted in one operation,
Then only the first assignment opens a database savepoint; the two others take their numbers from the transaction cache; and if the transaction rolls back, none of the three numbers is consumed.

---

## 8. Gap detection

**8.1 — A deleted middle entry flags its successor.**
Given `MISC/2026/03/0001`, `MISC/2026/03/0002` and `MISC/2026/03/0003` exist,
When the second one is deleted,
Then `MISC/2026/03/0003` is flagged as having made a gap and the journal reports a numbering hole.

**8.2 — Filling the gap clears the flag.**
Continuing the previous scenario, when a new entry is numbered `MISC/2026/03/0002`,
Then the flag of `MISC/2026/03/0003` is cleared and the journal no longer reports a hole.

**8.3 — A draft entry between two posted entries is flagged.**
Given `MISC/2026/03/0001` is posted, `MISC/2026/03/0002` is draft and `MISC/2026/03/0003` is posted,
Then `MISC/2026/03/0002` is flagged as having made a gap.

**8.4 — Suffixes make independent chains.**
Given the numbers `A/0001X` and `A/0002X` and the numbers `A/0001Y` and `A/0002Y` coexist in one journal with the same prefix,
Then deleting `A/0001X` flags `A/0002X` only; the chain ending in `Y` is untouched, because the neighbour search requires the same suffix.

**8.5 — An entry numbered through the locked increment is never flagged.**
Given an entry is posted and takes the next number of its chain in the same transaction,
Then it is never flagged as having made a gap, even if a concurrent transaction leaves a hole later.

---

## 9. Resequencing

**9.1 — Keeping the current order.**
Given the entries `XMISC/2019/10001`, `XMISC/2019/10002`, `XMISC/2019/10003`, `XMISC/2019/10004` and `XMISC/2019/10006` with the accounting dates 5, 6, 7, 4 and 5 March 2019 respectively,
When they are resequenced with the first number `XMISC/2019/10001` and the ordering "Keep current order",
Then the numbers become, in the order of the current numbering: `10001`, `10002`, `10003`, `10004` and `10005`.

**9.2 — Reordering by accounting date.**
Given the same selection,
When the ordering "Reorder by accounting date" is chosen,
Then the entry dated 4 March takes `10001`, the two entries dated 5 March take `10002` and `10003` (in the order of their current numbers), the entry dated 6 March takes `10004` and the entry dated 7 March takes `10005`.

**9.3 — Reformatting a year-range prefix.**
Given the five posted entries `XMISC/2022-2023/00001`, `XMISC/2022-2023/00002`, `XMISC/2022-2023/00003`, `XMISC/2023-2024/00001` and `XMISC/2023-2024/00002`, dated 1, 2 and 3 March 2023 and 1 and 2 April 2023, with a fiscal year ending 31 March,
When they are resequenced with the first number `XMISC/22-23/00001` and the ordering "Keep current order",
Then the numbers become `XMISC/22-23/00001`, `XMISC/22-23/00002`, `XMISC/22-23/00003`, `XMISC/23-24/00001` and `XMISC/23-24/00002`.

**9.4 — The counter of the first number applies to the last period only.**
Given the same five entries, whose periods appear in the selection in the order 2022-2023 then 2023-2024,
When they are resequenced with the first number `XMISC/22-23/00005`,
Then the 2022-2023 entries — not the last period met — start at the counter 1 and take `XMISC/22-23/00001`, `XMISC/22-23/00002` and `XMISC/22-23/00003`;
And the 2023-2024 entries — the last period met — start at the counter of the first number and take `XMISC/23-24/00005` and `XMISC/23-24/00006`.

**9.5 — Renumbering by date in a hash-secured journal is refused.**
Given a journal that secures posted entries,
When the resequence wizard is run with the ordering "Reorder by accounting date",
Then the operation is refused with "You can not reorder sequence by date when the journal is locked with a hash."

**9.6 — A selection spanning journals is refused.**
When the resequence wizard is opened on entries of two journals,
Then it fails with "You can only resequence items from the same journal".

**9.7 — Mixing credit notes with invoices is refused.**
Given a sale journal with a dedicated credit-note numbering,
When the resequence wizard is opened on a selection containing both an invoice and a credit note,
Then it fails with "The sequences of this journal are different for Invoices and Refunds but you selected some of both types."

---

## 10. The accounting date rule

**10.1 — A late vendor bill of a past month.**
Given today is 10 March 2026, the purchase journal numbers monthly and has January numbers, and a vendor bill has the document date 20 January 2026 and no lock date is violated,
When it is posted,
Then its accounting date becomes 31 January 2026.

**10.2 — A vendor bill of the current month.**
Given today is 10 March 2026 and the document date is 3 March 2026,
When it is posted,
Then its accounting date becomes 10 March 2026.

**10.3 — A vendor bill dated in the future.**
Given today is 10 March 2026 and the document date is 20 March 2026,
When it is posted,
Then its accounting date stays 20 March 2026, because the rule returns the later of the candidate and today.

**10.4 — A customer invoice in the past with nothing locked.**
Given today is 10 March 2026, the sale journal numbers yearly and the invoice date is 20 January 2026,
When it is posted,
Then the accounting date stays 20 January 2026.

**10.5 — A customer invoice in a locked period.**
Given the same invoice and a Global Lock Date of 31 January 2026,
When it is posted,
Then the candidate becomes 1 February 2026 and the yearly branch returns the earlier of today (10 March 2026) and 31 December 2026, that is **10 March 2026**.

**10.6 — A customer invoice in a locked period with a monthly sale chain.**
Given the same invoice, the same lock date, but the sale journal numbers monthly,
Then the candidate becomes 1 February 2026 and the monthly branch returns the earlier of today (10 March 2026) and 28 February 2026, that is **28 February 2026**.

**10.7 — A vendor bill in a locked period.**
Given today is 10 March 2026, the Global Lock Date is 31 January 2026, the purchase journal numbers monthly and the bill date is 20 January 2026,
When it is posted,
Then the candidate becomes 1 February 2026 and the accounting date becomes 28 February 2026.

**10.8 — A journal with no earlier number.**
Given a purchase journal with no entry at all, today 10 March 2026 and a bill dated 20 January 2026,
Then the monthly branch applies and the accounting date becomes 31 January 2026.

**10.9 — A yearly purchase chain.**
Given a purchase journal whose highest number is `BILL/2025/00010` (a yearly shape), today 10 March 2026 and a bill dated 20 December 2025,
Then the yearly branch applies, the year of today is later than the year of the bill, and the accounting date becomes 31 December 2025.

---

## 11. Lock dates

**11.1 — Posting into a locked period shifts the date.**
Given a Global Lock Date of 31 January 2026 and today 10 March 2026,
When a draft miscellaneous entry dated 15 January 2026 is posted,
Then the posting succeeds and the accounting date is moved out of the locked period by the rule of section 10.

**11.2 — Modifying a posted entry in a locked period is refused.**
Given a posted entry dated 15 January 2026 and a Global Lock Date of 31 January 2026,
When its reference is changed together with its date,
Then the operation is refused with "You cannot add/modify entries prior to and inclusive of: Global Lock Date (01/31/2026)."

**11.3 — A date exactly equal to the lock date is locked.**
Given a Global Lock Date of 31 January 2026,
Then an entry dated 31 January 2026 is inside the locked period.

**11.4 — The sales lock date applies only to sale journals.**
Given a Sales Lock Date of 31 January 2026 and no other lock date,
Then a posted miscellaneous entry dated 15 January 2026 may still be modified, while a posted customer invoice of the same date may not.

**11.5 — The tax lock date applies only to tax-bearing items.**
Given a Tax Return Lock Date of 31 January 2026,
When the balance of an item **without** taxes of a posted entry dated 15 January 2026 is changed,
Then the change succeeds;
And when the balance of an item **with** a tax is changed,
Then the change is refused with "The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: Tax Return Lock Date (01/31/2026)."

**11.6 — Several violated lock dates are listed chronologically.**
Given a Sales Lock Date of 31 January 2026 and a Global Lock Date of 28 February 2026,
When a posted customer invoice dated 15 January 2026 is modified,
Then the message lists "Sales Lock Date (01/31/2026) and Global Lock Date (02/28/2026)".

**11.7 — The hard lock date cannot be removed.**
Given a Hard Lock Date of 31 December 2025,
When it is cleared,
Then the operation is refused with "The Hard Lock Date cannot be removed."

**11.8 — The hard lock date cannot move backwards.**
Given a Hard Lock Date of 31 December 2025,
When it is set to 30 November 2025,
Then the operation is refused with "A new Hard Lock Date must be posterior (or equal) to the previous one."

**11.9 — Draft entries block a hard lock date.**
Given one draft entry dated 15 December 2025,
When the Hard Lock Date is set to 31 December 2025,
Then the operation is refused with "There are still draft entries in the period you want to hard lock. You should either post or delete them." and a button "Show draft entries".

**11.10 — Unreconciled bank transactions block a lock date.**
Given one unreconciled bank transaction dated 15 December 2025,
When the Global Lock Date is set to 31 December 2025,
Then the operation is refused with "There are still unreconciled bank statement lines in the period you want to lock.You should either reconcile or delete them." and a button "Show Unreconciled Bank Statement Line".

**11.11 — The lock dates of the ancestors apply.**
Given a parent company with a Global Lock Date of 31 December 2025 and a subsidiary with none,
Then an entry of the subsidiary dated 15 December 2025 is inside the locked period.

**11.12 — The later of the two wins.**
Given a parent company with a Global Lock Date of 31 December 2025 and a subsidiary with one of 31 January 2026,
Then the effective Global Lock Date for an entry of the subsidiary is 31 January 2026.

---

## 12. Lock exceptions

**12.1 — An exception relaxes the lock date for its beneficiary.**
Given a Global Lock Date of 31 December 2025 and an active exception for the user *A* relaxing it to 30 November 2025,
Then the user *A* may modify a posted entry dated 15 December 2025, while any other user may not.

**12.2 — An exception for everybody.**
Given the same exception with no beneficiary,
Then every user may modify that entry.

**12.3 — An exception that removes the lock date.**
Given an exception whose relaxed value is empty,
Then, for its beneficiary, the effective Global Lock Date is the minimal date and nothing is locked by it.

**12.4 — An exception may not relax the hard lock date.**
Given a Hard Lock Date of 31 December 2025,
Then no exception can be created for it, because the selection list offers only the four soft lock dates.

**12.5 — An expired exception has no effect.**
Given an exception whose end moment is in the past,
Then its state is expired and the effective lock date is again the company value.

**12.6 — Exactly one lock date per exception.**
When an exception is created setting both the Global Lock Date and the Sales Lock Date,
Then the creation is refused with "A single exception must change exactly one lock date field."

**12.7 — Revocation requires the administrator group.**
Given a user of the Invoicing group,
When that user revokes an exception,
Then the operation is refused with "You cannot revoke Lock Date Exceptions. Ask someone with the 'Adviser' role."

**12.8 — Changing the company lock date recreates the exception.**
Given an active exception relaxing the Global Lock Date from 31 December 2025 to 30 November 2025,
When the company Global Lock Date is set to 31 January 2026,
Then the original exception is revoked and a new active exception is created recording the company value 31 January 2026 and relaxing it to 30 November 2025.

**12.9 — An exception may not be duplicated.**
When a user duplicates an exception,
Then the operation is refused with "You cannot duplicate a Lock Date Exception."

**12.10 — The audit query of an exception.**
Given an exception granted to the user *A* on 3 February at ten o'clock, relaxing the Global Lock Date from 31 December 2025 to 30 November 2025,
When the audit action is opened,
Then it lists the journal items of entries of the company whose audit trail carries a message created by *A* after that moment, and whose accounting date lies between 30 November 2025 and 31 December 2025, or whose accounting date was changed from or to a date in that window.

---

## 13. The hash chain

**13.1 — Posting in a securing journal hashes the entry.**
Given a journal that secures posted entries and no earlier hashed entry,
When an entry is posted,
Then its hash is stored as a dollar sign, the current hash version, a dollar sign and sixty-four hexadecimal characters;
And the message "This journal entry has been secured." is logged on it.

**13.2 — The chain binds the entries.**
Given two entries of the same chain are hashed in order,
Then the input of the second hash is the digest of the first (without the version marker) followed by the serialised content of the second.

**13.3 — The serialised content.**
Given an entry of journal 3 in company 1, numbered `MISC/2026/03/0001`, dated 5 March 2026, with two items of identifiers 51 and 52 as described in `calculations.md`,
Then the serialised document contains exactly fourteen keys, sorted as: `company_id`, `date`, `journal_id`, `line_51_account_id`, `line_51_credit`, `line_51_debit`, `line_51_name`, `line_51_partner_id`, `line_52_account_id`, `line_52_credit`, `line_52_debit`, `line_52_name`, `line_52_partner_id`, `name`;
And the monetary values read `1000.00` and `0.00`, that is with exactly two decimals;
And an empty counterpart reads `False`.

**13.4 — A hashed entry cannot change its date.**
Given a hashed entry,
When its accounting date is changed,
Then the change is refused with "This document is protected by a hash. Therefore, you cannot edit the following fields: Date."

**13.5 — An item of a hashed entry cannot change its amount.**
Given a hashed entry numbered `MISC/2026/03/0001`,
When the debit of one of its items is changed,
Then the change is refused with "You cannot edit the following fields: Debit.\nThe following entries are already hashed:\nMISC/2026/03/0001".

**13.6 — A hashed entry cannot be reset to draft.**
When the reset button is used on a hashed entry,
Then the operation is refused with "You cannot reset to draft a locked journal entry."

**13.7 — An item of a hashed entry cannot be deleted.**
When an item of a hashed entry is deleted,
Then the deletion is refused with "You cannot delete journal items belonging to a locked journal entry."

**13.8 — A gap blocks the hashing.**
Given a chain whose posted entries carry the counters 1, 2 and 4 and none is hashed,
When the entry with the counter 4 is posted in a securing journal,
Then the operation is refused with "An error occurred when computing the inalterability. A gap has been detected in the sequence."

**13.9 — The bulk wizard tolerates the gap.**
Given the same chain,
When the secure-entries wizard is run up to a date covering the three entries,
Then the three are hashed and the wizard shows the warning "Securing these entries will create at least one gap in the sequence." with a button "Review Entries".

**13.10 — An unreconciled bank transaction blocks the chain.**
Given a chain containing the entry of an unreconciled bank transaction,
When the chain is hashed at posting,
Then the operation is refused with "An error occurred when computing the inalterability. All entries have to be reconciled.";
And when the bulk wizard is run instead, the whole chain is skipped and the warning "There are still unreconciled bank statement lines before the selected date. The entries from journal prefixes containing them will not be secured: *the list of prefixes*" is shown with the severity danger.

**13.11 — Hashing a chain hashes its predecessors.**
Given the chain `INV/2026/` holds the posted, unhashed entries with the counters 1, 2 and 3, and the journal has just been switched to securing,
When the entry with the counter 3 is hashed on demand,
Then the entries with the counters 1, 2 and 3 are hashed, in that order.

**13.12 — The verification detects a modification.**
Given a chain of three hashed entries whose second entry is altered directly in the database,
When the integrity report is printed,
Then the row for that journal and prefix reports "Corrupted data on journal entry with id *the identifier* (*the number*)." and the third entry is not checked.

**13.13 — The verification of an untouched chain.**
Given a chain of three hashed entries and no modification,
Then the row reports "Entries are correctly hashed" and the second table shows the first and the last entry with their hashes and their dates.

**13.14 — Printing without the permission.**
Given a user of the Invoicing group only,
When that user prints the integrity report,
Then the operation is refused with "Please contact your accountant to print the Hash integrity result."

---

## 14. Reset to draft, cancel and delete

**14.1 — A posted entry returns to draft and keeps its number.**
Given a posted entry numbered `MISC/2026/03/0002`,
When it is reset to draft,
Then its state is draft, its number is still `MISC/2026/03/0002` and its analytic lines are deleted.

**14.2 — Posting it again reuses the number.**
Continuing the previous scenario, when it is posted again,
Then its number is still `MISC/2026/03/0002`.

**14.3 — An exchange-difference entry cannot return to draft.**
When the reset button is used on an exchange-difference entry,
Then the operation is refused with "You cannot reset to draft an exchange difference journal entry."

**14.4 — A cash-basis entry cannot return to draft.**
When the reset button is used on a cash-basis entry,
Then the operation is refused with "You cannot reset to draft a tax cash basis journal entry."

**14.5 — Cancelling a posted entry.**
Given a posted, reconciled entry,
When it is cancelled,
Then it is first reset to draft, its reconciliations are undone, its linked payments are cancelled, its automatic posting mode becomes "No" and its state is cancelled;
And it keeps its number.

**14.6 — A cancelled entry is excluded from the ledger.**
Given a cancelled entry,
Then it appears in no report and no aggregation of posted items.

**14.7 — Deleting the last entry of a chain.**
Given `MISC/2026/03/0003` is the highest number of its chain and it is draft,
When a user of the Invoicing group deletes it,
Then the deletion succeeds.

**14.8 — Deleting a middle entry is refused for a clerk.**
Given `MISC/2026/03/0002` is draft and `MISC/2026/03/0003` exists,
When a user of the Invoicing group deletes it,
Then the deletion is refused with "You cannot delete this entry, as it has already consumed a sequence number and is not the last one in the chain. You should probably revert it instead."

**14.9 — An accountant may delete it and leave a gap.**
Given the same entry,
When a user of the Administrator group deletes it,
Then the deletion succeeds and `MISC/2026/03/0003` is flagged as having made a gap.

**14.10 — The restrictive audit trail forbids deleting a posted entry.**
Given the company keeps a restrictive audit trail and an entry that was posted once, now draft,
When it is deleted,
Then the deletion is refused with "To keep the restrictive audit trail, you can not delete journal entries once they have been posted.\nInstead, you can cancel the journal entry."

**14.11 — Deleting a posted item is refused.**
Given a posted entry with an item of 500.00,
When that item is deleted,
Then the deletion is refused with "You can't delete a posted journal item. Don’t play games with your accounting records; reset the journal entry to draft before deleting it."

**14.12 — The automatic choice.**
Given four posted entries: one hashed, one dated inside the fiscal lock date, one in a company with a restrictive audit trail and one ordinary,
When the automatic undo routine runs on the four,
Then the first two are reversed with cancelling reversals, the third is cancelled and the fourth is reset to draft and deleted.

---

## 15. Reversal

**15.1 — Reversing a plain entry.**
Given a posted plain entry dated 5 March with a debit of 1 000.00 on Rent expense and a credit of 1 000.00 on Suppliers,
When it is reversed on 31 March,
Then a plain entry dated 31 March is created with a credit of 1 000.00 on Rent expense and a debit of 1 000.00 on Suppliers;
And the two are reconciled on Suppliers, producing a Full Reconciliation;
And a message "This entry has been *link labelled "reversed"*" is logged on the original.

**15.2 — The reference of the reversal.**
Given the original is numbered `MISC/2026/03/0001` and no reason is given,
Then the reference of the reversal is "Reversal of: MISC/2026/03/0001";
And with the reason "Duplicate", it is "Reversal of: MISC/2026/03/0001, Duplicate".

**15.3 — Under storno accounting.**
Given the same original in a company that uses storno accounting,
Then the reversal carries a debit of −1 000.00 on Rent expense and a credit of −1 000.00 on Suppliers.

**15.4 — A reversal dated in the future is scheduled.**
Given today is 10 March and the reversal date chosen is 1 April,
Then the reversal is created with the automatic posting mode "At Date" and is **not** used to cancel the original.

**15.5 — Reversing an invoice produces a credit note.**
Given a posted customer invoice of 121.00,
When it is reversed,
Then a customer credit note is created whose items are **not** negated, and whose totals are `amount_untaxed` 100.00, `amount_tax` 21.00 and `amount_total` 121.00 — with the opposite ledger sign because the direction sign of a credit note is +1.

**15.6 — Reverse and modify.**
Given a posted customer invoice,
When "Reverse and Modify" is used,
Then a credit note is created, posted and reconciled with the invoice, and a fresh **draft** customer invoice is created carrying only the product, section, subsection and note items of the original, dated at the chosen date.

**15.7 — A selection spanning companies is refused.**
When the reversal wizard is opened on entries of two companies,
Then it fails with "All selected moves for reversal must belong to the same company."

**15.8 — A draft entry cannot be reversed.**
When the reversal wizard is opened on a draft entry,
Then it fails with "To reverse a journal entry, it has to be posted first."

**15.9 — The journal type must match.**
Given a selection of miscellaneous entries,
When a sale journal is chosen,
Then the save is refused with "Journal should be the same type as the reversed entry."

---

## 16. Automatic and recurring posting

**16.1 — The daily job posts a scheduled entry.**
Given a draft entry dated 1 April 2026 with the automatic posting mode "At Date",
When the job runs on 1 April 2026,
Then the entry is posted.

**16.2 — A failure switches the mode off.**
Given the same entry but using an archived account,
When the job runs,
Then the whole batch is rolled back, the entry is retried alone, the message "The move could not be posted for the following reason: A line of this move is using a archived account, you cannot post it." is posted on it as a comment, and its automatic posting mode becomes "No".

**16.3 — A monthly recurrence produces the next occurrence.**
Given an entry dated 15 January 2026 with the automatic posting mode "Monthly" and no end date,
When it is posted,
Then a copy dated 15 February 2026 is created, pointing at the first entry of the recurrence, keeping the mode, the end date and the salesperson.

**16.4 — The day of the month is preserved.**
Given the first entry of a monthly recurrence is dated 31 January 2026,
When the occurrence dated 28 February 2026 is posted,
Then the next occurrence is dated 31 March 2026 — the day of the first entry is restored whenever the target month allows it.

**16.5 — The end date stops the recurrence.**
Given a quarterly recurrence whose end date is 30 June 2026 and whose current occurrence is dated 1 April 2026,
When it is posted,
Then no occurrence is created for 1 July 2026.

**16.6 — No duplicate occurrence.**
Given an occurrence dated 15 February 2026 already exists for a recurrence,
When the occurrence dated 15 January 2026 is posted,
Then no second occurrence is created for 15 February 2026.

**16.7 — Resetting to draft deletes the next draft occurrence.**
Given a monthly recurrence with a posted occurrence dated 15 January and a draft occurrence dated 15 February,
When the January occurrence is reset to draft,
Then the February occurrence is deleted.

**16.8 — A vendor document scheduled without a document date.**
When a vendor bill with the automatic posting mode "At Date" is saved without a document date,
Then the save is refused with "For this entry to be automatically posted, it required a bill date."

---

## 17. Reconciliation in the company currency

**17.1 — A full match of two items.**
Given a receivable debit of 1 000.00 and a credit of 1 000.00 on the same account, both in the company currency,
When they are reconciled,
Then one match of 1 000.00 is created, both residuals are 0.00, both items are reconciled and one Full Reconciliation is created whose identifier becomes the matching number of both items.

**17.2 — A partial match.**
Given a receivable debit of 1 210.00 and a credit of 500.00,
When they are reconciled,
Then one match of 500.00 is created, the debit keeps a residual of 710.00, the credit keeps 0.00, no Full Reconciliation is created, and both items carry a matching number of the form `P` followed by the identifier of the match.

**17.3 — Several credits against one debit.**
Given a debit of 1 000.00 and the credits 300.00, 400.00 and 500.00, all in the company currency, and a further debit of 200.00,
When the five are reconciled together,
Then the group nets to zero, four matches are created and one Full Reconciliation covers all five items.

**17.4 — The pairing order follows the due date.**
Given a debit of 1 000.00 and two credits of 600.00 due on 1 March and 400.00 due on 1 February,
When the three are reconciled,
Then the February credit is matched first.

**17.5 — Items of different accounts are refused.**
When two items on different accounts are reconciled,
Then the operation fails with "Entries are not from the same account: *the two display names*".

**17.6 — An account that does not allow matching is refused.**
When two items on an expense account are reconciled,
Then the operation fails with "Account *the display name* does not allow reconciliation. First change the configuration of this account to allow it."

**17.7 — A liquidity account may be reconciled even without the flag.**
Given two items on an account of type Bank and Cash whose reconcilable flag is off,
When they are reconciled,
Then the operation succeeds.

**17.8 — Cancelled entries are refused.**
When an item of a cancelled entry is reconciled,
Then the operation fails with "You can not reconcile cancelled entries."

**17.9 — Already reconciled items are refused.**
When two fully reconciled items are reconciled again,
Then the operation fails with "You are trying to reconcile some entries that are already reconciled."

**17.10 — Extending a partially matched group is allowed.**
Given a group carrying the matching number `P17` in which one item is fully matched and another is not,
When both are reconciled with a new item,
Then the already matched item is silently dropped from the check and the operation succeeds.

---

## 18. Reconciliation in a foreign currency

**18.1 — Both items in the same foreign currency.**
Given a debit of 1 200.00 euros with 3 600.00 foreign units and a credit of −240.00 euros with −480.00 foreign units, −720.00 euros with −1 440.00 foreign units, −1 020.00 euros with −2 040.00 foreign units, and a further debit of 120.00 euros with 360.00 foreign units,
When the five are reconciled together,
Then every residual reaches zero in both currencies and one Full Reconciliation is created (the foreign amounts sum to 3 600 − 480 − 1 440 − 2 040 + 360 = 0 and the balances sum to 1 200 − 240 − 720 − 1 020 + 120 = −660, so exchange differences of 660.00 euros in total are produced).

**18.2 — Two different foreign currencies.**
Given debits of 1 200.00 euros with 3 600.00 units of the foreign currency and 780.00 euros with 2 340.00 units of the same currency, and credits of −240.00 euros with −960.00 units of the other foreign currency, −720.00 euros with −2 880.00 units and −1 020.00 euros with −4 080.00 units,
When the five are reconciled together,
Then the optimiser splits them into one node per currency, both nodes are processed first and the union afterwards, every residual reaches zero and one Full Reconciliation is created.

**18.3 — The reconciliation currency is the debit currency when both publish it.**
Given a debit in the foreign currency and a credit in the foreign currency, both with a non-zero foreign residual,
Then the match is made in the foreign currency.

**18.4 — An invoice in the company currency matched with a payment in a foreign currency.**
Given a receivable debit of 952.38 euros with no foreign currency on a receivable account, and a credit of −909.09 euros with −1 000.00 foreign units,
Then the debit publishes the foreign currency too, converting 952.38 at the rate obtained from the payment, and the match is made in the foreign currency.

**18.5 — An exchange-difference item is matched without a rate.**
Given a debit of 0.00 euros with 100.00 foreign units and a credit of −43.29 euros with 0.00 foreign units, both in the same foreign currency,
Then the exchange-line mode is detected, no rate is applied, and the match adjusts only the company-currency amount.

**18.6 — A zero-balance item with a foreign amount still participates.**
Given an item whose balance is 0.00 and whose foreign amount is 100.00,
Then it is placed in the debit list of the pairing loop, because the filter accepts a positive balance **or** a positive foreign amount.

---

## 19. Exchange differences

**19.1 — A loss.**
Given the company currency is the euro, an invoice receivable of 1 000.00 foreign units booked at 952.38 euros and a payment of 1 000.00 foreign units booked at 909.09 euros,
When the two are reconciled,
Then the match records 909.09 euros and 1 000.00 foreign units on each side;
And an exchange-difference entry is created with a credit of 43.29 on the receivable account and a debit of 43.29 on the exchange **loss** account, both labelled "Currency exchange rate difference" and both with a foreign amount of 0.00;
And the first of the two is matched with the invoice item, which then reaches zero in both currencies.

**19.2 — A gain.**
Given the same invoice and a payment of 1 000.00 foreign units booked at 1 000.00 euros,
Then the exchange-difference entry debits the bank account 47.62 and credits the exchange **gain** account 47.62.

**19.3 — The date of the exchange entry.**
Given the invoice is dated 15 January and the payment 20 February,
Then the exchange entry is dated 20 February, pushed through the accounting-date rule of the exchange journal, and never earlier than either of the two item dates.

**19.4 — The exchange entry is posted only when both sides are posted.**
Given one of the two matched entries is draft,
Then the exchange entry stays draft and is posted together with it.

**19.5 — Rounding noise is suppressed.**
Given a debit of 377 554.00 euros with 20 000.00 foreign units and a credit of −5 314.62 euros with −281.53 foreign units,
When they are reconciled in the foreign currency for 281.53 units,
Then the debit side converts to 5 314.64 euros with the tolerance range 5 314.54 to 5 314.73, the credit side converts to 5 314.62 with the range 5 314.53 to 5 314.71, each value lies inside the other range, the matched amount is forced to 5 314.62 and **no** exchange-difference entry is created.

**19.6 — A missing exchange journal.**
Given the company has no exchange journal,
When a match that needs an exchange difference is made,
Then the operation fails with "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."

**19.7 — A missing loss account.**
Given the company has an exchange journal but no loss account,
Then the operation fails with "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."

---

## 20. Matching numbers

**20.1 — A partial match.**
Given two items matched by a match with the identifier 17 and the group is not closed,
Then both items carry the matching number `P17`.

**20.2 — A closed group.**
Given the group closes and the Full Reconciliation has the identifier 9,
Then both items carry the matching number `9`.

**20.3 — Merging two groups.**
Given items A and B share the match 17 and items C and D share the match 23, and a new match 31 links B and C,
Then all four items carry `P17`, because the component number is the smallest match identifier of the component.

**20.4 — Removing the Full Reconciliation.**
Given four items carrying the matching number `9` and their Full Reconciliation is deleted while the matches survive,
Then the four items fall back to `P` followed by the smallest surviving match identifier.

**20.5 — Removing every match.**
Given the same items and every match is deleted,
Then their matching number becomes empty.

**20.6 — An imported label.**
Given a journal item is imported with the matching label `A17`,
Then the stored matching number is `IA17`.

**20.7 — Resolving imported labels.**
Given three items on one account carry the label `IA17` and belong to three entries,
When the last of those entries is posted,
Then the account is switched to reconcilable if it was not, the three items are reconciled with exchange differences and cash-basis entries suppressed, and their matching number becomes a real one.

**20.8 — An imported label on a matched item is refused.**
When an imported label is written on an item that already participates in real matches,
Then the write is refused with "A temporary number can not be used in a real matching".

---

## 21. Unreconciliation

**21.1 — Undoing a match restores the residuals.**
Given a debit of 1 210.00 matched for 500.00,
When the match is deleted,
Then the debit residual returns to 1 210.00 and the matching numbers are cleared.

**21.2 — Undoing reverses the exchange entry.**
Given a match that produced a posted exchange-difference entry,
When the match is deleted,
Then the exchange entry is reversed with a cancelling reversal referenced "Reversal of: *the exchange entry number*".

**21.3 — Undoing deletes a draft exchange entry.**
Given the exchange entry is still draft,
When the match is deleted,
Then the exchange entry is deleted rather than reversed.

**21.4 — The reversal date respects the lock dates.**
Given the exchange entry is dated 15 January 2026 and a Global Lock Date of 31 January 2026 is in force,
When the match is deleted,
Then the cancelling reversal is dated 1 February 2026.

**21.5 — Writing a protected field breaks the reconciliation.**
Given a matched item,
When its balance is changed,
Then every match it participates in is deleted before the write.

**21.6 — Changing the account of a whole group keeps the matches.**
Given a matched group of three items,
When the account of **all three** is changed in one operation,
Then the matches survive.

**21.7 — Changing the account of one item of the group breaks them.**
Given the same group,
When the account of only one item is changed,
Then the matches are deleted.

**21.8 — Undoing an entire matched group.**
Given four items in one matched group and two of them are selected,
When the "unreconcile matched entries" action is run,
Then the selection is first expanded to all four and every match of the group is deleted.

---

## 22. The opening entry

**22.1 — The date of the opening entry.**
Given the company opening date is 1 January 2026,
Then the opening entry is dated 31 December 2025.

**22.2 — With no opening date.**
Given the company has no opening date and the current year is 2026,
Then the opening entry is dated 31 December 2025.

**22.3 — The balancing item.**
Given opening amounts of 10 000.00 debit on Bank and 4 000.00 credit on Suppliers,
Then the entry carries a third item crediting 6 000.00 on the Current Year Earnings account labelled "Automatic Balancing Line".

**22.4 — A balanced set produces no balancing item.**
Given opening amounts of 10 000.00 debit on Bank, 3 500.00 debit on Customers, 4 000.00 credit on Suppliers and 9 500.00 credit on Capital,
Then no balancing item is created.

**22.5 — Updating an amount adjusts the balancing item.**
Continuing scenario 22.3, when the Bank amount is changed to 11 000.00,
Then the existing Bank item is updated and the balancing credit becomes 7 000.00.

**22.6 — Setting an amount to zero deletes its item.**
Continuing scenario 22.3, when the Suppliers amount is set to zero,
Then the Suppliers item is deleted and the balancing credit becomes 10 000.00.

**22.7 — The earnings account is created when missing.**
Given the chart has no account of type Current Year Earnings and the code `999999` is already used,
When an opening amount is recorded,
Then an account is created with the code `999998` (or the first free code below it), the name "Profit or Loss Appropriation" and the type Current Year Earnings.

**22.8 — Modifying a posted opening entry is refused.**
Given the opening entry is posted and numbered `MISC/2025/12/0001`,
When an opening amount is typed,
Then the operation is refused with the message that begins "You cannot import the "openning_balance" if the opening move (MISC/2025/12/0001) is already posted."

---

## 23. Automatic transfers

**23.1 — Changing the account.**
Given two posted, unreconciled receivable items of the same customer, 300.00 and 700.00 in the company currency,
When they are transferred to a doubtful-debt account on 30 June,
Then one entry is created and posted with a debit of 1 000.00 on the doubtful-debt account labelled "Transfer from *the receivable account display name*" and a credit of 1 000.00 on the receivable account labelled "Transfer to *the doubtful-debt account display name*";
And the two original items and the credit item are reconciled together.

**23.2 — Two different counterparts produce two counterpart items.**
Given two receivable items of 300.00 and 700.00 belonging to two different customers,
Then two counterpart items are created, one per customer, of 300.00 and 700.00.

**23.3 — A change of account never uses the "Transfer from" label with several source accounts.**
Given the selected items sit on two different accounts,
Then the counterpart item is labelled "Transfer counterpart".

**23.4 — Changing the period, full amount.**
Given a posted entry dated 20 December 2025 crediting 1 200.00 on a revenue account, today is 31 December 2025 and the target date is 1 January 2026,
When the whole amount is deferred,
Then the cancelling entry dated 31 December 2025 debits 1 200.00 on the revenue account and credits 1 200.00 on the deferred-revenue account;
And the destination entry dated 1 January 2026 credits 1 200.00 on the revenue account and debits 1 200.00 on the deferred-revenue account;
And both are labelled "Cut-off *the source number*";
And the two deferred-revenue items are reconciled with each other.

**23.5 — Changing the period, forty per cent.**
Given the same source and a percentage of 40,
Then the amounts are 480.00 and the labels become "Cut-off *the source number* 40.00%";
And 720.00 remains in December.

**23.6 — The nature is deduced from the sign.**
Given the sum of the selected balances is negative,
Then the revenue accrual account is used; and with a positive sum, the expense accrual account.

**23.7 — Mixing account types is refused.**
When the change-of-period action is prepared on items whose accounts have different types,
Then it fails with "All accounts on the lines must be of the same type."

**23.8 — A reconciled item is refused.**
When the wizard is opened on a reconciled item,
Then it fails with "Oops! You can only change the period or account for items that are not yet reconciled! Other ones aren't up for an adventure like that!"

**23.9 — A locked target date is refused.**
Given a Global Lock Date of 31 December 2025,
When the target date 15 December 2025 is chosen,
Then the save fails with "The date selected is protected by: Global Lock Date (12/31/2025)."

**23.10 — The percentage is bounded.**
When a percentage of 120 is typed for a change of period,
Then the save fails with "Percentage must be between 0 and 100".

---

## 24. Multi-company

**24.1 — A journal of a parent company is visible from a subsidiary.**
Given a journal of the parent company,
Then a user whose active company is the subsidiary can see it, because the record rule matches ancestors.

**24.2 — An entry of a parent company is not visible from a subsidiary.**
Given an entry of the parent company and a user whose only active company is the subsidiary,
Then the entry is not visible, because the record rule requires the exact company.

**24.3 — The company of an entry follows the journal.**
Given a journal of the subsidiary,
When an entry is created in it while the active company is the parent,
Then the company of the entry becomes the first accessible branch of the journal company, that is the subsidiary.

**24.4 — An account shared by two companies keeps one balance per company view.**
Given an account shared by a parent and a subsidiary with items in both,
Then the current balance shown to a user of the parent includes both, and the one shown to a user of the subsidiary includes only the subsidiary items.

**24.5 — Reconciling across companies is refused.**
When two items of two different company hierarchies are reconciled,
Then the operation fails with "Entries don't belong to the same company: *the two display names*".

**24.6 — The fiscal year is delegated to the root company.**
Given the root company ends its fiscal year on 31 March,
Then every subsidiary uses 31 March, whatever is stored on it.

**24.7 — Loading a chart template cascades.**
Given a parent company loads a template,
Then every child company loads the same template, and a child reuses the records of the parent rather than creating its own.

---

## 25. Access rights

**25.1 — A read-only user cannot create an entry.**
Given a user of the Show Accounting Features - Readonly group only,
When that user creates an entry,
Then the creation is refused.

**25.2 — A billing clerk can create and post.**
Given a user of the Invoicing group,
Then that user can create, read, modify, delete and post entries and items.

**25.3 — A billing clerk cannot create an account.**
Given the same user,
When an account is created,
Then the creation is refused; only the Administrator group can create accounts.

**25.4 — A portal user sees only its own documents.**
Given a portal user whose commercial entity is the customer C,
Then only the posted or cancelled-excluded documents of the four invoice types whose counterpart is C or a descendant of C are visible.

**25.5 — The account code is hidden from a user without the read group.**
Given a user who belongs neither to the Show Accounting Features - Readonly group nor to any group implying it,
Then the display name of an account is the name alone, without the code.

**25.6 — Revoking an exception requires the administrator group.**
Covered by scenario 12.7.

**25.7 — Resequencing requires the administrator group.**
Given a user of the Invoicing group,
When the resequence wizard is opened,
Then the access is refused, because the wizard is granted only to the Administrator group.

**25.8 — Typing a non-conforming number.**
Given a journal with a numbering override pattern and a user of the Invoicing group,
When a number that does not match the pattern is typed,
Then the write is refused with "The Journal Entry sequence is not conform to the current format. Only the Accountant can change it.";
And when an Administrator does the same, the write succeeds and the override pattern of the journal is cleared.

---

## 26. Rounding edge cases

**26.1 — A currency with a five-cent step.**
Given a currency whose rounding step is 0.05,
Then 1.23 rounds to 1.25 and 1.22 rounds to 1.20.

**26.2 — Half away from zero.**
Given a currency whose rounding step is 0.01,
Then 1.005 rounds to 1.01 and −1.005 rounds to −1.01.

**26.3 — A currency with no subdivision.**
Given a currency whose rounding step is 1,
Then 1 234.5 rounds to 1 235 and an item on that currency never carries a fractional foreign amount.

**26.4 — The balance invariant uses the company rounding step.**
Given the company currency has a step of 1 and the items sum to 0.4,
Then the entry is balanced.

**26.5 — A conversion at a rate that does not divide evenly.**
Given a rate of 1.10 units per euro and a foreign amount of 1 000.00,
Then the balance is 1 000.00 ÷ 1.10 = 909.0909… rounded to 909.09.

**26.6 — The residual reaches exactly zero.**
Given a debit of 909.09 matched for 909.09,
Then the residual is 0.00 and the item is reconciled at the precision of both currencies.

**26.7 — A residual below the rounding step counts as zero.**
Given a debit of 909.09 matched for 909.085 (a value only reachable through an intermediate computation),
Then the stored match is rounded to 909.09 and the residual is 0.00.

**26.8 — The tolerance range uses half the step of the source currency.**
Given a source currency with a step of 0.01 and an amount of 281.53 converted at a rate of 0.05297255491929631,
Then the three values computed are the conversions of 281.525, 281.53 and 281.535, each rounded to the target currency.
