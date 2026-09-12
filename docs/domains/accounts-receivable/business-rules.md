# Business rules of the Accounts Receivable domain

Every validation, constraint, invariant, permission check, locking rule and edge case, with the exact
message text the system produces. Placeholders are written out in words in italics.

---

## 1. Invariants that always hold

| # | Invariant |
| --- | --- |
| I1 | The sum of the balances of a document's lines is zero, rounded to the company currency. |
| I2 | On any accountable line, at most one of debit and credit is non-zero. |
| I3 | On any accountable line, the balance and the amount in currency have the same sign (either may be zero). |
| I4 | Every accountable line has an account. |
| I5 | A section, a subsection or a note has no account, no debit, no credit and a zero amount in currency. |
| I6 | On a sale document, a line is a payment term line **if and only if** its account is of the receivable kind. |
| I7 | On a sale document, no line uses a payable account. |
| I8 | Every accountable line of an invoice carries the commercial entity of the document's partner. |
| I9 | The sum of the payment term line amounts equals the document total, exactly, in both currencies. |
| I10 | A posted document has a number other than `/`, and that number is unique within its journal among posted documents. |
| I11 | The line currency of every line of an invoice equals the document currency, except cost-of-goods-sold lines which are always in the company currency. |
| I12 | A tax line exists for each distinct tax repartition grouping key with a non-zero amount; no tax line exists for a grouping key whose amount is zero in both currencies, unless the repartition explicitly keeps zero lines. |

---

## 2. Stored database checks

These are enforced by the storage layer on every write, and their violation message is fixed.

| Check | Condition | Message |
| --- | --- | --- |
| credit and debit | a non-section line must have debit × credit = 0 | Wrong credit or debit value in accounting entry! |
| sign consistency | a non-section line must have balance and amount in currency of the same sign | The amount expressed in the secondary currency must be positive when account is debited and negative when account is credited. If the currency is the same as the one from the company, this amount must strictly be equal to the balance. |
| account required | a non-section line must have an account | Missing required account on accountable line. |
| non-accountable emptiness | a section, subsection or note must have a zero amount in currency, zero debit, zero credit and no account | Forbidden balance or account on non-accountable line |
| unique number | at most one posted document per (number, journal) with a number other than `/` | Another entry with the same name already exists. |

---

## 3. Validation on the document

### 3.1 Balance

Checked around every create, write and delete of a document or of its lines, unless the caller
explicitly disables the check.

- One unbalanced document:
  > The entry is not balanced.
- Several unbalanced documents:
  > The following entries are unbalanced:
  >
  > &nbsp;&nbsp;- *the first document number*
  > &nbsp;&nbsp;- *the second document number*
  > … one line per document.

### 3.2 Journal kind must match the document type

Checked whenever the journal or the document type changes.

- > Cannot create a sale document in a non sale journal
- > Cannot create a purchase document in a non purchase journal

### 3.3 Currency rate

Checked whenever the rate changes. Applies only when the document is an invoice (receipts included)
and its currency differs from the company currency.

- > The currency rate must be strictly positive.

### 3.4 Tax country compatibility

Checked whenever the lines, the fiscal position or the company change. Let the *impacted countries*
be the countries of the taxes used on the lines and of the taxes that produced the tax lines. If that
set is non-empty and differs from the document's tax country:

- when the document has a fiscal position whose country also differs from the impacted countries:
  > This entry contains taxes that are not compatible with your fiscal position. Check the country
  > set in fiscal position and in your tax configuration.
- otherwise:
  > This entry contains one or more taxes that are incompatible with your fiscal country. Check
  > company fiscal country in the settings and tax country in taxes configuration.

### 3.5 Automatic posting requires a document date

Checked whenever auto-post or the document date changes; applies to purchase documents only.

- > For this entry to be automatically posted, it required a bill date.

### 3.6 Lock dates

Checked when a posted document's number or accounting date changes, when a posted document leaves
the posted status, and when a document is deleted. The violated lock dates are the fiscal year lock
date, the sale lock date (for a sale journal), the purchase lock date (for a purchase journal) and
the hard lock date.

- > You cannot add/modify entries prior to and inclusive of: *the list of violated lock dates with
  > their names and values*.

A separate check applies to lines that affect the tax report (they carry taxes, a tax repartition, or
a tax grid whose applicability is "taxes"), against the tax lock date and the hard lock date:

- > The operation is refused as it would impact an already issued tax statement. Please change the
  > journal entry date or the following lock dates to proceed: *the list of violated lock dates*.

A caller may bypass the lock check with an explicit marker; this is used only by internal flows that
have already validated the dates.

---

## 4. Validation on a line

### 4.1 Account must be usable

Checked at posting, and on write when the account or the currency changes.

- Archived account (unless the line was imported or the caller skips the check):
  > The account *the account name* (*the account code*) is archived.
- Account forcing a secondary currency that is neither the company currency nor the line currency:
  > The account selected on your journal entry forces to provide a secondary currency. You should
  > remove the secondary currency on the account.
- Writing an archived account onto a line:
  > You cannot use an archived account.

### 4.2 Receivable and payable consistency

Checked whenever the account or the display type changes.

On a **sale document** (receipts included):

- an account of the payable kind is refused:
  > Account *the account code* is of payable type, but is used in a sale operation.
- being a payment term line and having a receivable account must coincide; either one without the
  other is refused:
  > Any journal item on a receivable account must have a due date and vice versa.

On a **purchase document** (receipts included):

- an account of the receivable kind is refused:
  > Account *the account code* is of receivable type, but is used in a purchase operation.
- > Any journal item on a payable account must have a due date and vice versa.

### 4.3 Off-balance accounts

Checked whenever the account, the taxes, the originator tax or the reconciled flag changes.

- > If you want to use "Off-Balance Sheet" accounts, all the accounts of the journal entry must be
  > of this type
- > You cannot use taxes on lines with an Off-Balance account
- > Lines from "Off-Balance Sheet" accounts cannot be reconciled

### 4.4 Mixing cash-basis and ordinary taxes

Checked whenever the taxes or the tax repartition changes. When a line carries both a tax whose
exigibility is "on payment" and a tax whose exigibility is not, and the two share a base grid:

- > Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share
  > some tag.

The same is refused when a tax line's own grids overlap the base grids of the other exigibility.

### 4.5 Deductibility

Checked whenever the deductibility changes.

- A deductibility other than one hundred on a document that is not a vendor bill or purchase receipt:
  > Only vendor bills allow for deductibility of product/services.
- A deductibility outside the range zero to one hundred:
  > The deductibility must be a value between 0 and 100.

### 4.6 Modifying a line of a posted document

- Changing the taxes or the originator tax of a posted line:
  > You cannot modify the taxes related to a posted journal item, you should reset the journal entry
  > to draft to do so.
- Changing a field of a line belonging to a hashed document:
  > You cannot edit the following fields: *the list of field labels*.
  >
  > The following entries are already hashed:
  > *one document number per line*
- Changing a field that would invalidate a reconciliation breaks that reconciliation automatically,
  unless the only changing field is the account **and** every line of the reconciliation group is
  part of the same write (which allows moving a whole reconciliation to another account).

### 4.7 Deleting a line

- A non-zero line of a posted document:
  > You can't delete a posted journal item. Don’t play games with your accounting records; reset the
  > journal entry to draft before deleting it.
- A tax line, when the document still has taxed lines, deleted outside the synchronisation:
  > You cannot delete a tax line as it would impact the tax report
- A payment term line deleted outside the synchronisation:
  > You cannot delete a payable/receivable line as it would not be consistent with the payment terms
- Any line of a hashed document:
  > You cannot delete journal items belonging to a locked journal entry.

Deleting a line first removes its reconciliations, then checks the lock dates on posted documents,
then logs the deletion on the document's message thread with the tracked values:

> Journal Item *a link labelled with the line identifier* deleted

---

## 5. Rules on writing to a document

### 5.1 Reviewed flag

- Marking a document as reviewed, or resetting a reviewed document to draft, requires the reviewer
  right:
  > You don't have the access rights to perform this action.
  and, for the reset:
  > Validated entries can only be changed by your accountant.

### 5.2 Hash protection

- > This document is protected by a hash. Therefore, you cannot edit the following fields: *the list
  > of field labels*.

### 5.3 Changing the journal

- On a document that has been posted at least once, unless the number is being cleared or set to `/`:
  > You cannot edit the journal of an account move if it has been posted once, unless the name is
  > removed or set to "/". This might create a gap in the sequence.
- On a document that already has a number whose sequence position is neither zero nor one, unless the
  number is being cleared and quick-encoding mode is off:
  > You cannot edit the journal of an account move with a sequence number assigned, unless the name
  > is removed or set to "/". This might create a gap in the sequence.

When the journal is changed on a document that has never been posted and no number is supplied, the
number is cleared and recomputed. Changing the journal also schedules the company and the currency
for re-derivation; the complete list of these chained recomputations is in
[`entities.md`](entities.md) section 1.14.

### 5.4 Readonly fields on a posted document

The following may not be written while the document is posted, unless the caller explicitly skips the
check: the invoice lines, the journal items, the document date, the accounting date, the partner, the
payment terms, the currency, the fiscal position and the cash rounding method.

> You cannot modify the following readonly fields on the posted move *the document number, or its
> reference, or its identifier*: *the comma-separated field names*

### 5.5 Journal number pattern

When the journal defines a number pattern and the number being written does not match it:

> The Journal Entry sequence is not conform to the current format. Only the Accountant can change it.

A user in the accountant group is allowed through; doing so clears the journal's pattern.

### 5.6 The document must keep a company

Checked on the form the moment the company field is edited, and again whenever the company is written.
Clearing the company is refused with:

> We can't leave this document without any company. Please select a company for this document.

This is deliberately not a stored constraint. A stored constraint is evaluated when the record is
saved, and by then the field rules that need the company — the journal, the currency, the accounts,
the taxes and the rate — have already run and would have failed first. The check therefore fires
while the form is still open, before anything is saved.

The same rule then performs the second half of its work: when the journal presently on the document
does not belong to the newly chosen company (nor to one of that company's ancestors), the journal is
scheduled for re-derivation, so the document leaves the field with a journal of the right company.
When the present journal is already valid for the new company it is left alone.

### 5.7 Decoding an attachment dropped on a document

A file dropped on a draft document may be offered to the decoding path that turns a received document
into lines. The document refuses to be decoded when it already carries invoice lines, and the reason
returned — shown to the user as the explanation for why nothing was decoded — is:

> The invoice already contains lines.

The condition is exactly "the document has at least one invoice line"; the document's status, its
type and the nature of the file are not examined by this particular refusal. On the receivable side
the practical consequence is that a file dropped on a document that already has lines is kept as a
plain attachment. The rest of the decoding path belongs to
[`../electronic-invoicing-and-document-exchange/workflows.md`](../electronic-invoicing-and-document-exchange/workflows.md)
and, for received vendor documents, to
[`../accounts-payable/workflows.md`](../accounts-payable/workflows.md).

---

## 6. Rules on deleting a document or a contact

### 6.1 Deleting a document

1. A document that has consumed a number and is not the last in its numbering chain may not be
   deleted, unless the user is in the accountant group, or quick-encoding mode is on for the company,
   or the caller forces the deletion:
   > You cannot delete this entry, as it has already consumed a sequence number and is not the last
   > one in the chain. You should probably revert it instead.
2. When the company enforces a restrictive audit trail, a document that has been posted at least once
   may not be deleted:
   > To keep the restrictive audit trail, you can not delete journal entries once they have been
   > posted.
   > Instead, you can cancel the journal entry.
3. Deletion removes every reconciliation of the lines, deletes the lines, then deletes the document.
   The numbering gap flag is refreshed first. When a forced deletion removes documents that were
   posted before under a restrictive audit trail, a technical log entry records the user, the
   documents, their totals, their partners and their per-account balances.

### 6.2 Deleting a contact that a document names

Deleting a contact is refused as soon as at least one journal entry names that contact as its partner
and that entry's status is draft or posted. Cancelled entries do not protect a contact. The count is
taken with elevated rights, so a contact is protected by documents the deleting user cannot even see.
The message is:

> The partner cannot be deleted because it is used in Accounting

There is no full stop at the end; the text is reproduced exactly. The check applies to every contact,
whether it is a company, an individual or a child contact of a company, and it is evaluated once for
the whole set being deleted, so deleting several contacts at once is refused as soon as any one of
them is named by a document.

---

## 7. Rules on posting

The complete guard list, with messages, is in [`state-machines.md`](state-machines.md) section 1.4.
Summarised here as a checklist:

| # | Rule |
| --- | --- |
| P1 | The user must be in the invoicing group. |
| P2 | In quick-encoding mode the typed total must equal the computed total. |
| P3 | The recipient bank account must not be archived. |
| P4 | On an inbound document the recipient bank account must be trusted for outgoing payments, or be cleared. |
| P5 | The total must not be negative. |
| P6 | A customer is required on a sale document. |
| P7 | A document date is required on a purchase document; on a sale document it defaults to today. |
| P8 | Each line's account must be compatible with the journal. |
| P9 | The document must be draft. |
| P10 | At least one accountable line must exist. |
| P11 | A future-dated auto-post document may not be force-posted. |
| P12 | The journal must be active. |
| P13 | The currency must be active. |
| P14 | No archived account may be used. |
| P15 | Every account must belong to the document's company or an ancestor. |
| P16 | No archived analytic account may be referenced. |
| P17 | Lock dates push the accounting date forward rather than blocking. |

---

## 8. Rules on resetting to draft and cancelling

See [`state-machines.md`](state-machines.md) sections 1.6 and 1.3. Summarised:

| # | Rule | Message |
| --- | --- | --- |
| R1 | Only posted or cancelled documents may be reset | Only posted/cancelled journal entries can be reset to draft. |
| R2 | A document awaiting an approved cancellation request may not be reset | You can't reset to draft those journal entries. You need to request a cancellation instead. |
| R3 | An exchange difference entry may not be reset | You cannot reset to draft an exchange difference journal entry. |
| R4 | A cash-basis tax entry, or the origin of one, may not be reset | You cannot reset to draft a tax cash basis journal entry. |
| R5 | A hashed document may not be reset | You cannot reset to draft a locked journal entry. |
| R6 | Only draft documents may be cancelled (a posted one is reset first) | Only draft journal entries can be cancelled. |
| R7 | Requesting a cancellation on a document that does not need one | You can only request a cancellation for invoice sent to the government. |

---

## 9. Rules on the payment status

- Blocking a paid or in-payment document:
  > You can't block a paid invoice.
- The imported-balance payment status (`invoicing_legacy`) is never overwritten by the computation.
- The blocked payment status is never overwritten by the computation; it is cleared only by the
  toggle.

---

## 10. Rules on payment terms

### 10.1 Payment term validation

Checked whenever the lines or the early discount flag change.

- The percentages of the percent lines must add up to exactly one hundred, compared at the
  payment-term decimal precision:
  > The Payment Term must have at least one percent line and the sum of the percent must be 100%.
  (This also fires when the term has no line at all, since the sum is then zero.)
- An early payment discount requires a single line:
  > The Early Payment Discount functionality can only be used with payment terms using a single 100%
  > line.
- > The Early Payment Discount must be strictly positive.
- > The Early Payment Discount days must be strictly positive.

### 10.2 Payment term line validation

- A percent line's amount must lie between zero and one hundred inclusive:
  > Percentages on the Payment Terms lines must be between 0 and 100.
- The "days on the next month" value must be numeric and within range:
  > The days added must be between 0 and 31.
  when it parses as a number outside the range, and
  > The days added must be a number and has to be between 0 and 31.
  when it does not parse as a number at all. A leading minus sign is stripped before the numeric test,
  so a negative value fails the range test rather than the numeric test.

### 10.3 Deleting a payment term

> Uh-oh! Those payment terms are quite popular and can't be deleted since there are still some
> records referencing them. How about archiving them instead?

---

## 11. Rules on cash rounding

- > Please set a strictly positive rounding value.
- A non-blocking warning when the "add a rounding line" strategy is chosen and no profit account is
  configured for the current company:
  > **Warning for Cash Rounding Method: *the method name***
  > You must specify the Profit Account (company dependent)
- Under the "modify the biggest tax amount" strategy, when the document carries no tax line at all,
  no rounding line is produced and the document total is left unrounded. This is an edge case an
  implementation must reproduce: the strategy silently does nothing rather than falling back to the
  other strategy.

---

## 12. Rules on sending

See [`state-machines.md`](state-machines.md) section 3.4. Summarised:

| Rule | Message |
| --- | --- |
| The document must be posted | You can't generate invoices that are not posted. |
| The document must be a sale document | You can only generate sales documents. |
| The chosen printable layout must be an invoice layout | The sending of invoices is not set up properly, make sure the report used is set for invoices. |
| A printable layout must exist for the document type | There is no template that applies to this move type. |
| At least one printable layout must apply to invoices at all | There is no template that applies to invoices. |
| Batch sending needs the background job enabled — for an administrator | Batch invoice sending is unavailable. Please, activate the cron to enable batch sending of invoices. *(with a link to the job configuration)* |
| Batch sending needs the background job enabled — for anyone else | Batch invoice sending is unavailable. Please, contact your system administrator to activate the cron to enable batch sending of invoices. |
| In single-document mode every recipient must have an e-mail address (a blocking alert) | Partner(s) should have an email address. |

In batch mode the missing-e-mail condition is a warning rather than a blocker, with the action
"View Partner(s)".

**Where the last rule fires.** "There is no template that applies to invoices." is not raised at send
time. It is raised while the set of printable layouts available to invoices is being computed — the
set of layouts declared for the journal entry entity, flagged as invoice layouts, whose own record
filter accepts a customer invoice, a customer credit note and a sales receipt all three at once. That
set feeds two places: the layout selector on the contact form and the layout selector of the sending
dialogue. A company whose only layouts are excluded by their own filters therefore cannot open either
selector; the refusal appears when the form or the dialogue is opened, not when Send is pressed.

---

## 13. Rules on reversal and debit notes

### 13.1 Reversal

| Rule | Message |
| --- | --- |
| All selected documents share one company | All selected moves for reversal must belong to the same company. |
| Every selected document is posted | To reverse a journal entry, it has to be posted first. |
| The chosen journal's kind matches the sources' journals | Journal should be the same type as the reversed entry. |

### 13.2 Debit note

| Rule | Message |
| --- | --- |
| Every selected document is posted | You can only debit posted moves. |
| No selected document is already the source of a debit note | You can't make a debit note for an invoice that is already linked to a debit note. |
| Every selected document is a customer invoice, a customer credit note, a vendor bill or a vendor credit note | You can make a debit note only for a Customer Invoice, a Customer Credit Note, a Vendor Bill or a Vendor Credit Note. |

The "copy lines" option is *hidden* when the selected sources share one document type and that type
is a credit note, but hiding it does not force it off, and it is not hidden when the sources have
mixed types. The lines are copied whenever the option is on, whatever the source's type — see the
compatibility finding in [`accounting-effects.md`](accounting-effects.md) section 5.

---

## 14. Warnings that do not block

These appear as banners or dialogue boxes; the user may proceed.

### 14.1 No chart of accounts

Raised as a redirecting warning when a partner is chosen and that partner has neither a receivable
nor a payable account configured:

> Cannot find a chart of accounts for this company, You should configure it.
> Please go to Account Configuration.

with the button "Go to the configuration panel".

### 14.2 Number warnings

- A **name warning** flag is raised in the form when the typed number is not empty, is not `/`, and
  is lexically less than or equal to the highest existing number in the scope, and quick-encoding
  mode is off.
- A **format-change dialogue** appears when a number is typed that changes the detected numbering
  format, while the accounting date and the journal are unchanged. Its title is:

  > The sequence format has changed.

  and its body is two paragraphs. The first is:

  > It was previously '*the previous number*' and it is now '*the new number*'.

  The second depends on the detected reset rule:

  | Reset rule | Text |
  | --- | --- |
  | monthly | The sequence will restart at 1 at the start of every month.\nThe year detected here is '*the year*' and the month is '*the month*'.\nThe incrementing number in this case is '*the formatted counter*'. |
  | yearly | The sequence will restart at 1 at the start of every year.\nThe year detected here is '*the year*'.\nThe incrementing number in this case is '*the formatted counter*'. |
  | fiscal year range | The sequence will restart at 1 at the start of every financial year.\nThe financial start year detected here is '*the start year*'.\nThe financial end year detected here is '*the end year*'.\nThe incrementing number in this case is '*the formatted counter*'. |
  | fiscal year range and month | The sequence will restart at 1 at the start of every month.\nThe financial start year detected here is '*the start year*'.\nThe financial end year detected here is '*the end year*'.\nThe month detected here is '*the month*'.\nThe incrementing number in this case is '*the formatted counter*'. |
  | never | The sequence will never restart.\nThe incrementing number in this case is '*the formatted counter*'. |

### 14.3 The alert banner

The document exposes a set of alerts, each with a level and a message. The base platform produces:

| Key | Level | Condition | Message |
| --- | --- | --- | --- |
| tax lock date | warning | draft, the user is in an accounting group, and a tax lock date message exists | the tax lock date message |
| auto-post at date | info | draft and auto-post is "At Date" | This move is configured to be posted automatically at the accounting date: *the accounting date*. |
| recurring auto-post | info | draft and auto-post is yearly, quarterly or monthly | *the recurrence name* auto-posting enabled. Next accounting date: *the accounting date*. — and, when an end date is set, followed by a space and: The recurrence will end on *the end date* (included). |
| empty lines | info | draft, a purchase document, and at least two invoice lines whose total is zero | We've noticed some empty lines on your invoice. — with the action "Remove empty lines" which deletes those lines |
| being sent | info | a send is in flight | This invoice is being sent in the background. |
| credit warning | warning | the user is in an accounting group and the credit warning text is non-empty | the credit warning text (see 14.4) |
| abnormal amount | warning | the abnormal-amount text is non-empty | see 14.5 |
| abnormal date | warning | the abnormal-date text is non-empty | see 14.5 |

### 14.4 The credit limit warning

Computed only on a **draft customer invoice** and only when the company's credit limit feature is on.
The algorithm and the four message variants are in [`entities.md`](entities.md) section 6.3.

### 14.5 The abnormal amount and date warnings

Computed only on a **draft purchase document** with a non-zero total, and skipped entirely when the
partner has both "ignore abnormal invoice date" and "ignore abnormal invoice amount" set, or when the
caller disables abnormal detection. They therefore never fire on a customer invoice; the two fields
exist on the shared entity and are always empty for receivable documents. Their specification belongs
to [`../accounts-payable/business-rules.md`](../accounts-payable/business-rules.md).

---

## 15. Duplicate detection

### 15.1 What counts as a duplicate

Recomputed whenever the reference, the document type, the partner, the document date, the totals or
the currency changes. It compares the document against every other document, of the same company,
whose status is draft or posted, and:

- whose document type matches — identical, or both among {vendor bill, purchase receipt}, or both
  among {customer invoice, sales receipt};
- whose currency is identical;
- whose commercial entity is identical, or the candidate has no commercial entity and the other
  document is draft;
- and then, per direction:

**Sale side** (customer invoice, customer credit note, sales receipt): the total **and** the document
date must both be identical.

**Purchase side** (vendor bill, vendor credit note, purchase receipt): either
1. the references are identical **and** (either document has no date, or the two dates fall in the
   same calendar year); or
2. the commercial entities, the totals and the document dates are identical and the total is not
   zero.

Only documents the current user may read are returned. When the document being checked is not yet
saved, its live values are injected into the comparison, so the warning appears while typing.

### 15.2 Derived flags

- **Has draft duplicates**: at least one detected duplicate is draft.
- **Is an exact duplicate**: purchase side only — some duplicate shares the reference, a compatible
  type, the partner, the document date and the total.

### 15.3 What the user can do

A button offers to delete every detected duplicate of the current document.

---

## 16. Permission checks

| Operation | Requirement |
| --- | --- |
| Post a document | membership of the invoicing group (bypassed for elevated callers) |
| Mark a document as reviewed, or reset a reviewed document to draft | the right to review |
| Read the outstanding-credits block, the payments block, the credit warning, the partner's receivable and payable aggregates and credit limit | membership of the invoicing group or of the read-only accounting group |
| Change a number that does not match the journal's pattern | membership of the accountant group |
| Delete a document that is not the last of its numbering chain | membership of the accountant group, or quick-encoding mode on the company |
| Mark a bank account as trusted for outgoing payments | membership of the bank-account validation group or of the system administration group, and not the automated system user unless installing demonstration data or running tests |
| Grant the partial-deductibility group | happens automatically at posting when a purchase document has a line below one hundred percent deductibility |

The complete access rights matrix and record rules are in
[`configuration.md`](configuration.md).

---

## 17. Locking rules

| Lock | What it prevents |
| --- | --- |
| Fiscal year lock date | adding or modifying any entry dated on or before it |
| Sale lock date | the same, for entries in a sale journal |
| Purchase lock date | the same, for entries in a purchase journal |
| Tax lock date | modifying anything that would change an already-filed tax return |
| Hard lock date | the same as the fiscal year lock date, with no exception possible |
| Inalterability hash | editing or deleting any hashed field, any line, or the document itself |
| Reviewed flag | resetting to draft without the reviewer right |
| Posted status | editing the readonly fields listed in 5.4 |
| Reconciliation | editing the fields that a reconciliation depends on; the reconciliation is broken instead |

Lock date exceptions may be granted per user and per lock kind; they belong to
[`../general-ledger/configuration.md`](../general-ledger/configuration.md).

---

## 18. Edge cases an implementation must reproduce

1. **A payment term whose lines all fall on the same date** produces a single receivable line whose
   amount is the sum, because the needed-terms key is the maturity date.
2. **A payment term with a fixed line larger than the total** produces a last line with a negative
   amount, because the balance rule takes whatever residual remains — including a negative one. No
   validation prevents this.
3. **A document with no payment term** still produces exactly one receivable line, dated at the
   document's due date.
4. **A derived line family whose needed amounts all cancel to zero** produces no line at all; the
   aggregation rule drops such keys.
5. **A user who manually creates the derived lines before the platform does** keeps them: the
   synchroniser stops when the need was previously empty and the existing non-trivial keys changed.
6. **A user edit that does not change the need** is preserved: the synchroniser stops when the needed
   map before and after are equal.
7. **A deleted derived line is not resurrected** when the need has not changed: keys naming
   line identifiers that no longer exist are dropped from the before-state.
8. **Recycling** means the identifier of a receivable line survives a change of maturity date; any
   record pointing at that line keeps pointing at the right thing.
9. **A zero-total invoice** that is posted immediately triggers the "invoice paid" hook, so downstream
   domains treat it as settled without any payment.
10. **A negative-total invoice** cannot be posted; the user is told to make a credit note instead.
11. **Changing the currency of a draft invoice** forces a full recomputation of the tax lines from the
    bases, not a rate reapplication, because the tax amounts in the old currency are meaningless.
12. **Changing only the currency rate** preserves the tax amounts in the document currency and
    recomputes only their company-currency balances.
13. **A document whose lines were supplied wholesale by a caller** (amounts on the base lines
    included) is not recomputed: the tax synchroniser detects that changed base lines carry explicit
    amounts and stops.
14. **A cash rounding difference of zero** deletes any existing rounding line rather than leaving a
    zero line behind.
15. **Switching the cash rounding strategy** deletes the existing rounding line before building the
    new one, because the two strategies produce structurally different lines.
16. **The early payment discount in the "always" mode survives a reversal**: the credit note keeps the
    same payment term so that its early payment discount lines mirror the invoice's.
17. **A credit note against an already-paid invoice** does not change the invoice; it stands as an
    open credit until refunded or attached to another invoice.
18. **A payment term line label without a payment reference** on a multi-instalment term reads
    `installment #1`, with the leading space stripped.
19. **A document whose partner has no receivable account** falls back through the company's own
    partner record and then to any active receivable account of the company; only if none exists does
    the account stay empty and posting fails on the "missing required account" check.
20. **A line whose account is the journal's default account or the journal's suspense account** is
    exempted from the journal-compatibility check.
