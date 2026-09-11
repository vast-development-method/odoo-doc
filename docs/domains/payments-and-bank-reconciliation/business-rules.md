# Business rules

Every validation, constraint, invariant, error message, permission check, locking rule and edge-case behavior of the payments and bank reconciliation domain.

Error texts are given exactly as the system produces them. Placeholders are written in words between angle brackets.

---

## 1. Payment

### 1.1 Database-level constraints

| Rule | Enforcement | Message |
|---|---|---|
| The amount is never negative. | A database check on the amount column: the amount must be greater than or equal to zero. | The payment amount cannot be negative. |

### 1.2 Record-level constraints

Checked whenever the named fields change.

| # | Trigger fields | Condition that fails | Message |
|---|---|---|---|
| 1 | payment method line | The Payment has no payment method line. | Please define a payment method line on your payment. |
| 2 | payment method line | The payment method line names a journal and that journal is not the Payment's journal. | The selected payment method is not available for this payment, please select the payment method again. |
| 3 | state, journal entry | The state is neither `draft` nor `canceled`, the Payment has no Journal Entry, and it has an outstanding account. | A payment with an outstanding account cannot be confirmed without having a journal entry. |

Rule 1 exists because the payment method line is a computed, stored, editable field and therefore cannot be declared mandatory at the field level.

Rule 3 is the invariant that makes the whole domain consistent: a confirmed Payment that is supposed to produce accounting must have produced it. It deliberately does not fire for a Payment whose method has no payment account, because that configuration produces no accounting at all.

### 1.3 Rules enforced by the operations

| Operation | Condition | Message |
|---|---|---|
| Confirm | The payment method requires a recipient bank account, the direction is outbound, and the recipient account is not trusted for outgoing payments. | To record payments with <the payment method name>, the recipient bank account must be manually validated. You should go on the partner bank account of <the counterparty display name> in order to validate it. |
| Prepare the journal entry | No outstanding account resolves. | You can't create a new payment without an outstanding payments/receipts account set either on the company or the <the payment method name> payment method in the <the journal display name> journal. |
| Create a Payment while the full accounting capability is absent | Neither the chart template's outstanding-receipts account (for an inbound payment) or outstanding-payments account (for an outbound one) nor the company's inter-bank transfer account exists. | No outstanding account could be found to make the payment |
| Change the amount of a Payment whose draft entry has several liquidity lines | The synchronisation cannot decide which line to change. | You cannot change the amount of a payment with multiple liquidity lines. |

### 1.4 Company consistency

Automatic company checking applies across the Payment's journal, Journal Entry, counterparty, recipient bank account, outstanding account and destination account. A value belonging to another company is refused by the platform's generic company-consistency check.

The company itself is recomputed from the journal and narrowed to a branch the current user may act for.

### 1.5 Duplicate detection

Duplicate detection is a **warning**, never a refusal. A Payment is a possible duplicate of another when all of the following hold:

1. both belong to the same company;
2. both name the same counterparty;
3. both carry the same date;
4. both carry the same direction;
5. both carry the same amount;
6. the other Payment's state is one of the matching states.

The matching states are `draft` and `in_process` when detecting from a Payment; `draft` and `posted` when detecting from the register-payment screen (which builds a throw-away Payment to run the same query).

The detection is skipped entirely for a Payment that has no counterparty, has a zero amount, or is already `in_process`.

When the Payment being edited has not been saved yet, its values are injected into the query directly so the warning appears before saving.

### 1.6 Deletion

Deleting a Payment is allowed. The sequence is: reset every non-draft entry to draft — which applies the general ledger's own guards on lock dates, hash chains and the audit trail — then delete every entry, then delete the Payment, then force the payment state of the previously reconciled documents to recompute.

### 1.7 Copying

A copy keeps the journal and the payment method line and drops: the Journal Entry, the state, the number, the sent flag, the payment reference, the settled documents and the paired-transfer reference.

### 1.8 Edge cases

| Situation | Behavior |
|---|---|
| The amount is zero. | Both reconciliation flags are forced to true. The entry, if produced, has two zero lines. |
| The outstanding account is changed on the journal after Payments exist. | The line-splitting rule repairs the split: when exactly one write-off line was found and either the liquidity or the counterpart group is empty, that single line is moved into the empty group. |
| The Payment has no counterparty. | The destination account falls back to the company's first receivable or payable account according to the counterparty kind. The duplicate check is skipped. |
| The recipient bank account is deleted. | Refused: the link restricts deletion. |
| The counterparty is deleted. | Refused: the link restricts deletion. |
| The Payment's method line has a journal different from the Payment's journal. | Refused by record-level rule 2. |
| The Payment's entry is posted and the Payment is edited. | The synchronisation skips it: a posted entry is never rewritten from the Payment. |

---

## 2. Payment method and payment method line

### 2.1 Uniqueness of a method

The pair (code, direction) is unique across the database: "The combination code/payment type already exists!"

### 2.2 Multiplicity on a journal

Checked whenever a journal's inbound or outbound method lines change.

**Rule A — no two identical lines of a restricted method on one journal and one direction.** For each direction, among the journal's lines of that direction, count the (method, name) pairs, ignoring the lines whose method's registry mode is neither `electronic` nor `unique`. A count above one fails:

> You can't have two payment method lines of the same payment type (<the direction>) and with the same name (<the line name>) on a single journal.

**Rule B — a unique method is attached at most once per company.** For each method whose registry mode is `unique`, if more than one journal of the company is linked to it, the method is collected.

**Rule C — an electronic method is attached at most once per company and per provider.** For each method whose registry mode is `electronic`, for each provider of the company that has that method's code, if more than one journal is linked to that (method, provider) pair, the method is collected.

If any method was collected by rule B or C:

> Some payment methods supposed to be unique already exists somewhere else.
> (<the display names of the collected methods, comma-separated>)

### 2.3 Eligibility of a method on a journal

A method may be offered on a journal only when the journal matches the method's eligibility domain (`entities.md`, section 2.4). The domain is evaluated with both the currency and the country filters when deciding whether the method may be *used*, and without them when computing which methods are technically *available*.

### 2.4 Deletion of a method line

A line that at least one Payment points at is never deleted. Instead its journal is cleared, which detaches it from the journal while preserving the historical reference from the Payments. Only lines with no Payment are actually deleted. The count is performed with elevated privileges so a restricted user cannot obtain a deletion by not seeing the Payments.

When the online-payment capability is installed, a further guard applies: a line linked to a provider whose state is enabled or test cannot be deleted at all:

> You can't delete a payment method that is linked to a provider in the enabled or test state.
> Linked providers(s): <the provider display names, comma-separated>

### 2.5 Deletion of a method

Deleting a Payment Method first deletes every Payment Method Line pointing at it, then the method. The line-deletion rule above still applies to each line.

### 2.6 Deletion of a journal

Deleting a liquidity journal deletes every Payment Method Line attached to it, then the journal, then the bank accounts that no surviving journal points at.

When the online-payment capability is installed, a journal linked to a provider whose state is not disabled cannot be deleted:

> You must first deactivate a payment provider before deleting its journal.
> Linked providers: <the provider display names, comma-separated>

---

## 3. Liquidity journal

These rules belong to the general ledger but govern this domain.

| Trigger | Condition that fails | Message |
|---|---|---|
| type, bank account | The journal is of type `bank`, has a bank account, and that account's company is set and differs from the journal's company. | The bank account of a bank journal must belong to the same company (<the company name>). |
| type, bank account | The journal is of type `bank`, has a bank account, and that account's holder is not the company's partner. | The holder of a journal's bank account must be the company (<the company name>). |
| company | Journal entries linked to the journal belong to a company that is not the new company or one of its descendants. | You can't change the company of your journal since there are some journal entries linked to it. |
| active | The journal is being archived and has at least one draft entry. | You can not archive a journal containing draft journal entries.\n\nTo proceed:\n1/ go to Accounting > Accounting > Journal Entries\n2/ filter on this journal and on 'Unposted' entries\n3/ select them all and post or delete them through the action menu |

Field restrictions:

- The journal's default account must be of type `asset_cash` or `liability_credit_card` for a `bank` journal, `liability_credit_card` for a `credit` journal, `asset_cash` for a `cash` journal.
- The suspense account must be of type `asset_current`.
- The profit account must be of type `income` or `income_other`; the loss account of type `expense`.
- The journal's sequence prefix is at most five characters and is unique per company.

---

## 4. Bank statement

### 4.1 Invariants

| Invariant | How it is kept |
|---|---|
| A statement belongs to exactly one journal. | The journal is computed from the journal of the statement's transactions. |
| A statement's date is the date of its last posted transaction. | Computed, recomputed whenever a transaction's ordering index or state changes. |
| A statement's first-line index is the smallest ordering index among its transactions. | Computed; it is the key by which statements are ordered and chained. |
| A statement's computed ending balance always equals its starting balance plus its posted transactions. | Computed; it cannot be edited. |
| The reported ending balance defaults to the computed one. | Computed, stored and editable; the computation copies the computed balance, so a new statement is complete until the user says otherwise. |

### 4.2 Rules enforced when grouping transactions

| Condition | Message |
|---|---|
| The selected transactions do not all belong to one journal. | A statement should only contain lines from the same journal. |
| The selected transactions are not contiguous, once cancelled transactions found inside the range are set aside. | Unable to create a statement due to missing transactions. You may want to reorder the transactions before proceeding. |

Cancelled transactions found inside the range are **added** to the selection rather than rejected, so that the run of ordering indexes stays unbroken.

### 4.3 Deletion protection

Deleting a Bank Transaction that belongs to a statement that is both valid and complete is refused:

> You can not delete a transaction from a valid statement.
> If you want to delete it, please remove the statement first.

The statement itself may be deleted; doing so releases its transactions.

### 4.4 Attachments

- Attachments named at creation, or on a write that targets a single statement, are re-pointed at the statement.
- A write that targets more than one statement silently drops the attachment values rather than attaching the same files to each.

### 4.5 Completeness and validity are warnings

Neither completeness nor validity blocks anything. They surface as the statement's *problem description*, as a flag on the journal, and as a warning on the dashboard card. A statement can be saved, kept and used while incomplete or invalid.

---

## 5. Bank transaction

### 5.1 Amount and currency consistency

Checked whenever the amount, the foreign amount, the currency, the foreign currency or the journal changes.

| Condition that fails | Message |
|---|---|
| The foreign currency equals the journal currency. | The foreign currency must be different than the journal one: <the journal currency name> |
| There is no foreign currency but a foreign amount is set. | You can't provide an amount in foreign currency without specifying a foreign currency. |
| There is a foreign currency but no foreign amount. | You can't provide a foreign currency without specifying an amount in 'Amount in Currency' field. |

The second and third rules make the foreign pair all-or-nothing.

At creation, rather than failing, a foreign currency equal to the journal currency is silently dropped and the foreign amount set to zero.

### 5.2 The suspense account

Creating a transaction without an explicit counterpart account requires the journal to have a suspense account:

> You can't create a new statement line without a suspense account set on the <the journal display name> journal.

### 5.3 The two structural invariants of the entry

Enforced when the entry's lines change and the change is propagated back to the transaction.

| Condition that fails | Message |
|---|---|
| The entry does not have exactly one liquidity line. | The journal entry <the entry display name> reached an invalid state regarding its related statement line.\nTo be consistent, the journal entry must always have exactly one journal item involving the bank/cash account. |
| The entry has more than one suspense line. | <the entry display name> reached an invalid state regarding its related statement line.\nTo be consistent, the journal entry must always have exactly one suspense line. |

Zero suspense lines is legal: it is what a fully reconciled transaction looks like.

The *liquidity* group is identified by the account being the journal's default account; when no line matches, the group falls back to the lines whose account type is `asset_cash` or `liability_credit_card`, and those lines are then removed from the *other* group. This fallback is what keeps a transaction usable after the journal's default account has been changed.

### 5.4 Editing a reconciled transaction

Changing the label, the amount, the foreign amount, the foreign currency, the currency or the counterparty of a Bank Transaction rewrites its entry and **deletes every line other than the liquidity and the suspense line**. A reconciled transaction therefore loses its allocation when its amount is corrected. The matchings that involved the deleted lines are removed by the deletion itself.

### 5.5 Undoing a reconciliation

| Condition | Message |
|---|---|
| The transaction is checked and reconciled, and the current user is not able to review the entry. | Validated entries can only be changed by your accountant. |

### 5.6 Deletion

- The protection of section 4.3 applies first.
- A transaction whose company has a restrictive audit trail has its entry **cancelled** rather than deleted; the transaction record itself is removed.
- Otherwise the transaction is removed and the entry force-deleted.

### 5.7 Edge cases

| Situation | Behavior |
|---|---|
| The transaction is created with no amount. | The amount defaults to zero; the entry has two zero lines; the transaction counts as reconciled because a zero amount has nothing to explain. |
| The transaction is created with a statement but no journal. | The journal is copied from the statement. |
| The transaction is not checked. | Its residual is the whole amount, whatever its journal items say; it is counted in the *to check* figure and not in the *to reconcile* figure. |
| The transaction's entry is cancelled. | It no longer contributes to any balance, but it is still used as an anchor when checking contiguity. |
| The transaction has no ordering index yet (it is being created in a form). | It is excluded from the statement's date and first-line index computations. |

---

## 6. Reconciliation model

| Trigger | Condition that fails | Message |
|---|---|---|
| label condition, label parameter | The condition is *match regex* and the parameter does not compile as a regular expression. | The regex is not valid |

Field rules enforced by the screen rather than by a constraint:

- the minimum bound is required when an amount condition other than *lower* is chosen;
- the maximum bound is required when the amount condition is *between*;
- the label parameter is required whenever a label condition is chosen;
- a line's account is required unless the line names a counterparty.

Two derived flags are recomputed from the conditions and the lines:

- **can be proposed** — true when the model is not a partner mapping and it has at least one of: a label condition, an amount condition, a counterparty list, or the automated trigger. A model without any of those would match every transaction and is therefore never proposed spontaneously.
- **mapped counterparty** — set when the model has a label condition, exactly one line, and that line names a counterparty and no account.

Duplicating a model produces a unique name by appending " (copy)" as many times as needed. An explicit name given in the copy values is used unchanged.

Archiving a model removes it from every proposal.

Company rule: a model is visible from a company when the model's company is an ancestor of it.

---

## 7. Reconciliation model line

| Trigger | Condition that fails | Message |
|---|---|---|
| amount text | The mode is `fixed` and the numeric reading is zero. | The amount is not a number |
| amount text | The mode is `percentage_st_line` and the numeric reading is zero. | Statement line percentage can't be 0 |
| amount text | The mode is `percentage` and the numeric reading is zero. | Balance percentage can't be 0 |
| amount text | The mode is `regex` and the text does not compile as a regular expression. | The regex is not valid |

Changing the amount mode resets the amount text: `100` for both percentage modes, `([\d,]+)` for the regular-expression mode, empty for the fixed mode.

The numeric reading of the amount text is the number it denotes, or zero when it is not a number — which is exactly why a zero reading is rejected for the three numeric modes.

Company checking applies to the line's account and to its taxes. Deleting a tax used by a line is restricted; deleting the account removes the line with it.

---

## 8. Reconciliation

### 8.1 Eligibility of a set of items

Checked on every top-level node of the plan, after the items already fully reconciled by this same operation have been set aside and after the narrowing rule of `calculations.md`, section 2.2.

| # | Condition that fails | Message |
|---|---|---|
| 1 | Any remaining item is already reconciled. | You are trying to reconcile some entries that are already reconciled. |
| 2 | Any remaining item belongs to a cancelled entry. | You can not reconcile cancelled entries. |
| 3 | The remaining items sit on more than one account. | Entries are not from the same account: <the account display names, comma-separated> |
| 4 | The remaining items belong to more than one root company. | Entries don't belong to the same company: <the company display names, comma-separated> |
| 5 | The single account is not reconcilable and its type is neither `asset_cash` nor `liability_credit_card`. | Account <the account display name> does not allow reconciliation. First change the configuration of this account to allow it. |

The checks run in that order, so the first failing one is the one reported.

### 8.2 Invariants of a matching

| Invariant | Enforcement |
|---|---|
| Both currency fields are set. | Constraint: "Missing foreign currencies on partials having ids: <the identifiers>" |
| All three amounts are positive. | By construction: the algorithm always takes minima of positive quantities. |
| A matching links exactly one debit item and one credit item. | Both links are mandatory. |
| The matching's date is the later of the two items' dates. | Computed. |
| The matching's company is the debit item's company when the debit item's entry is an invoice-like document, the credit item's company otherwise. | Computed; it decides where the exchange and cash-basis entries land. |

### 8.3 The matching number

A constraint enforces the permitted shapes (`calculations.md`, section 6.2). Violations raise internal errors rather than user-facing messages, because they indicate a corrupted state rather than a user mistake:

| Violation | Text |
|---|---|
| The number matches none of the permitted shapes. | Invalid matching number format |
| The number is an import placeholder but the item has matchings. | A temporary number can not be used in a real matching |
| The number is a partial number but the item has no matching. | Should have partials |
| The number is a partial number but the item has a full reconciliation. | Should not be partial number |
| The number is all digits but the item has no full reconciliation. | Should not be full number |
| The item has a full reconciliation but the number is not its identifier. | Matching number should be the full reconcile |
| The item has matchings but no number. | Should have number |

The second message is a user-facing validation; the others are internal assertions.

### 8.4 Exchange differences

| Condition that fails | Message |
|---|---|
| An exchange entry has to be created and the company has no exchange journal. | You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. |
| The company has no expense exchange account. | You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. |
| The company has no income exchange account. | You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. |

### 8.5 Cash-basis taxes

| Condition that fails | Message |
|---|---|
| A cash-basis entry has to be created and the company has no cash-basis journal. | There is no tax cash basis journal defined for the '<the company display name>' company.\nConfigure it in Accounting/Configuration/Settings |

### 8.6 Locking and dating

- An exchange-difference entry is dated on the later of the two matched items' dates, then pushed forward past the journal's accounting lock date, then raised again to the date of each item it fixes.
- A cash-basis entry is dated on the later of the settlement date and the day after the company's fiscal lock date for the cash-basis journal.
- A reversal produced by unreconciliation keeps the original entry's date, unless that date violates a lock date, in which case it is moved to the day after the latest violated lock date.
- Reconciling does not itself check lock dates: it writes no journal item on the matched entries. Only the derived entries are subject to lock dates.

### 8.7 Partial matching restrictions

A Bank Transaction may not be partially matched against a journal item belonging to a Payment when either:

- the Payment's payment method code is `sepa_ct` or starts with `iso20022` — that is, the Payment is part of a transfer file already submitted to the bank; or
- the Payment has an online transaction.

Such a Payment is all-or-nothing.

---

## 9. Bank account

### 9.1 Uniqueness

| Rule | Message |
|---|---|
| The pair (sanitized account number, holder) is unique. | The combination Account Number/Partner must be unique. |
| A bank account backs at most one journal. | A bank account can belong to only one journal. |

### 9.2 The trust flag

| Condition | Message |
|---|---|
| The trust flag is being set or cleared and the current user is not a bank-account validator. | You do not have the rights to trust or un-trust accounts. |
| After the write, an account is trusted and the current user cannot trust accounts. | You do not have the right to trust or un-trust a bank account. |
| The account is trusted and the write changes the account number, the sanitized number, the holder or the type, without also clearing the trust flag. | You cannot modify the account number or partner of an account that has been trusted. |

Who may trust: a user with elevated privileges, a user in the bank-account validator group, or a user in the system administrator group. In addition, the automated background user may not trust accounts, except while loading demonstration data or while running tests.

At creation the flag is always forced to false first, and only set afterwards when the user may trust. This makes it impossible to create an already-trusted account in one step by a user who lacks the permission.

### 9.3 Creation

| Condition | Message |
|---|---|
| An archived account with the same holder and the same number exists. | A bank account with Account Number <the number> already exists for Partner <the partner name>, but is archived. Please unarchive it instead. |

Every creation posts on the holder's message thread: "Bank Account <a link to the account> created".

### 9.4 The find-or-create algorithm

| Condition | Message |
|---|---|
| The account is not found, creation for the database's own company partners was not explicitly allowed, and the holder is one of those company partners. | Please add your own bank account manually: <the account number> (<the holder display name>) |

The algorithm is specified in `workflows.md`, section 11.1.

### 9.5 Modification and deletion

- Every write that changes a tracked field posts "Bank Account <a link> updated" on the holder's thread, and also on the previous holder's thread when the holder changed. The tracked fields are: the bank, the active flag, the account number, the holder name, the clearing number, the holder, the trust flag and the currency.
- Deleting never deletes: the record is archived and "Bank Account <a link> with number <the number> archived" is posted on the holder's thread.

### 9.6 The international account number

- The validation of `calculations.md`, section 14.3, runs as a constraint on every account whose inferred type is the international one. Its four messages are: "There is no IBAN code.", "The IBAN is invalid, it should begin with the country code", "The IBAN does not seem to be correct. You should have entered something like this <the template>\nWhere B = National bank code, S = Branch code, C = Account No, k = Check digit", and "This IBAN does not pass the validation check, please verify it."
- Asking for the basic bank account number of an account that is not of the international type: "Cannot compute the BBAN because the account number is not an IBAN."
- Both creation and modification rewrite a valid number into its pretty form (groups of four separated by single spaces); an invalid number is stored exactly as typed, and the constraint then decides whether it is acceptable.

### 9.7 Warnings

Neither warning blocks anything.

| Warning | Condition |
|---|---|
| Country mismatch | The account is untrusted, of the international type, the holder has a country, and the first two characters of the sanitized number differ from that country's code. |
| Money-transfer service | The account is untrusted, of the international type, and characters five to seven of the sanitized number are one of the known money-transfer institution codes; the service's name is shown. |
| Duplicate | Other active accounts carry the same number, within the same company or within no company; their holders are listed. |

---

## 10. Bank

- The bank identifier code is stored upper-cased: both creation and modification upper-case it.
- Choosing a country different from the selected state's country clears the state; choosing a state fills the country from it.
- No uniqueness is enforced on the bank identifier code; it is only indexed for search.

---

## 11. Register-payment screen

### 11.1 Refusals at opening

In this order:

| # | Condition | Message |
|---|---|---|
| 1 | The active model is neither the entry model nor the journal item model. | The register payment wizard should only be called on account.move or account.move.line records. |
| 2 | After filtering, no journal item has a residual on a receivable or payable account. | There's nothing left to pay for the selected journal items, so no payment registration is necessary. You've got your finances under control like a boss! |
| 3 | The selected items belong to more than one root company. | You can't create payments for entries belonging to different companies. |
| 4 | The items come from sibling companies and the user may not act for their root company. | You can't create payments for entries belonging to different branches without access to parent company. |
| 5 | The remaining items mix receivable and payable account types. | You can't register payments for both inbound and outbound moves at the same time. |
| 6 | Any selected document has payment state `blocked`. | You cannot register payments for blocked invoices. |

A default journal inherited from the calling list that is not a liquidity journal of the right company is silently discarded rather than refused.

### 11.2 Refusals while building the batches

| Condition | Message |
|---|---|
| The items belong to more than one root company. | You can't create payments for entries belonging to different companies. |
| There are no items. | You can't open the register payment wizard without at least one receivable/payable line. |

### 11.3 Refusal at creation

Batches for which a recipient bank account is required but missing or untrusted are skipped. When none survives:

> To record payments with <the payment method name>, the recipient bank account must be manually validated. You should go on the partner bank account in order to validate it.

### 11.4 Warnings

| Warning | Condition | Presentation |
|---|---|---|
| Payments in progress | Any selected document already has a Payment whose state is `in_process`. | Severity *danger*, message "There are payments in progress. Make sure you don't pay twice.", with a link labelled "Check them" that opens those Payments. |
| Untrusted recipient accounts | A recipient account is required and at least one batch's account is untrusted. | The accounts and the number of affected payments are listed, with a link that opens them for validation. |
| Missing recipient accounts | A recipient account is required and a batch's counterparty has none. | The counterparties are listed, with a link that opens them. |
| Duplicate payment | The Payment about to be created matches an existing one on company, counterparty, date, direction and amount, in state `draft` or `posted`. | The duplicates are listed. |

None of the warnings blocks the creation, except that an untrusted or missing account causes that batch to be skipped.

### 11.5 Forced choices

| Condition | Effect |
|---|---|
| At least one selected item belongs to a draft entry. | On creation, the difference handling is forced to *keep open*. |
| The screen is in discount mode. | The difference handling is forced to *mark as fully paid* and the write-off section is hidden. |
| The screen is not editable (more than one batch). | The amount, the counterparty, the counterparty kind, the source currency and the source amounts are cleared; the difference handling is cleared; the recipient account list is empty. |
| The direction of a batch differs from the direction of the chosen method line. | That batch uses the first available method line of the journal for its own direction. |

### 11.6 The custom amount

As soon as the typed amount differs from all four computed totals, it is remembered as a custom amount together with the currency it was typed in. From then on:

- the automatic recomputation of the amount stops;
- the switch sentence under the amount disappears;
- changing the currency converts the custom amount at the payment date;
- changing the payment date restores the custom amount unchanged.

---

## 12. Bank setup screen

- The holder is always forced to the active company's partner.
- The journal name is forced to the account number at creation; typing an account number copies it into the journal name.
- When no bank is chosen but a bank identifier code is typed, a Bank is searched by that code and created with that code as both its name and its code when not found.
- The default journal offered is the first journal of the wanted type that has no bank account **and** has never carried a journal entry.
- On save, either a new journal is created or the selected journal is repointed; in both cases the journal's name becomes the entered name.

---

## 13. Permission checks

| Operation | Required role |
|---|---|
| Read a Payment | Billing user or accounting reader. |
| Create, update, delete a Payment | Billing user. |
| Read a Bank Statement or a Bank Transaction | Accounting reader or billing user. |
| Create, update, delete a Bank Statement or a Bank Transaction | Basic accounting user. |
| Read a Reconciliation Model or its lines | Accounting reader or billing user. |
| Update a Reconciliation Model or its lines | Billing user. |
| Create or delete a Reconciliation Model or its lines | Basic accounting user. |
| Read a Partial or Full Reconciliation | Accounting reader. |
| Create, update, delete a Partial or Full Reconciliation | Billing user (and, in the full accounting capability, the accountant). |
| Read a Payment Method or a Payment Method Line | Any internal user. |
| Create and update a Payment Method Line | Billing user. |
| Create and delete a Payment Method | Billing user. Updating a Payment Method is not granted to anyone by the access matrix. |
| Use the register-payment screen | Billing user (create, read, update; no delete). |
| Trust or untrust a Bank Account | Bank-account validator, or system administrator. |
| Review a validated bank transaction (undo a reconciliation on a checked, reconciled transaction) | A user the entry's review rule accepts — in practice an accountant. |

Record rules restricting visibility by company:

| Entity | Rule |
|---|---|
| Payment | The company is one of the user's active companies. |
| Bank Statement | The company is one of the user's active companies, or is empty. |
| Bank Transaction | The company is one of the user's active companies. |
| Reconciliation Model and its lines | The model's company is an ancestor of one of the user's active companies. |
| Bank Account | Unrestricted for a billing user — an explicit rule grants full visibility so that a billing officer can see a counterparty's accounts even where another capability restricts them. |

---

## 14. Locking rules

| Situation | Rule |
|---|---|
| A Payment whose entry is posted | The entry is never rewritten by the Payment's synchronisation. The Payment must be reset to draft first, which applies the general ledger's lock-date, hash-chain and audit-trail guards. |
| A Bank Transaction whose entry is posted | The entry **is** rewritten by the transaction's synchronisation, because a bank transaction's entry is posted from the moment it is created. The general ledger's own protections on posted entries are bypassed for this specific synchronisation. |
| A trusted Bank Account | The number, the sanitized number, the holder and the type are locked while the trust flag is on. |
| A statement that is valid and complete | Its transactions cannot be deleted. |
| A company with a restrictive audit trail | Deleting a Bank Transaction cancels its entry instead of deleting it. |
| A checked and reconciled Bank Transaction | Only a user who may review entries can undo its reconciliation. |

---

## 15. System parameters

| Parameter | Effect |
|---|---|
| `account.skip_create_bank_account_on_reconcile` | When set to a value that reads as true, reconciling a bank transaction never creates a Bank Account for the reported account number: it only searches for one matching the number, the counterparty and a company that is empty or the transaction's company. |

---

## 16. Edge cases across the domain

| Situation | Behavior |
|---|---|
| A Payment's liquidity account is the journal's own default account rather than an outstanding account. | The Payment is considered matched by definition; its state reaches `paid` as soon as it is confirmed; it is counted in the *direct bank payments* figure of the dashboard rather than in the outstanding figure. |
| A Payment's outstanding account is of type `asset_cash`. | Confirming moves the Payment straight to `paid`. |
| A reconciliation would create an exchange difference on an exchange-difference line. | The exchange-line mode of `calculations.md`, section 3.4, suppresses the rate, so the matching only moves the company-currency amount and leaves the foreign amount untouched. |
| The two sides of a foreign-currency matching differ only by rounding. | The collapse rule of `calculations.md`, section 3.7, step 3, equalises them and avoids a spurious exchange difference. |
| An item takes part in the debit queue and in the credit queue at once (a positive balance with a negative foreign amount, or the reverse). | The pairing loop still terminates, because every iteration either produces a matching that strictly reduces a residual or removes a state from a queue. |
| A group of matched items is fully settled but one of the items has no matching at all. | No Full Reconciliation is created: an item with no matching never counts as settled, which prevents an exchange-difference line from closing a group by itself. |
| A Full Reconciliation is deleted. | The pointer on each item is cleared by the database, and the matching number of each item is explicitly recomputed: it becomes empty, or a partial number when matchings survive. |
| Items imported with a matching number starting with `I`. | They are not matched. They are matched later, per (number, account) group, once every entry of the group is posted; the account is made reconcilable if it was not; the matching runs with exchange differences and cash-basis entries disabled. |
| A reconciliation model line whose regular expression does not match the label. | That line produces nothing; the remaining amount stays on the suspense account. |
| A transaction is reconciled while the applied model is automated. | The entry is also marked as checked, so the transaction leaves the *to check* figure. |
