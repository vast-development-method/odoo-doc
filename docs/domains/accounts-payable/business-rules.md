# Accounts Payable — Business Rules

Every rule below is stated with its exact trigger, its condition and the exact user-facing message the system produces. Placeholders in messages are written in words between guillemets.

---

## 1. Invariants

| # | Invariant | Enforced by |
|---|---|---|
| I1 | A posted purchase document's journal items balance: the rounded sum of the balances is zero, rounded to the company currency's number of decimal places | the balance check that wraps every creation and every write |
| I2 | Every accountable journal item has an account | a database check |
| I3 | A journal item's debit and credit are never both non-zero | a database check |
| I4 | A journal item's amount in document currency carries the same sign as its balance | a database check |
| I5 | A presentation line (section, subsection, note) has no account, no amount in currency, no debit and no credit | a database check |
| I6 | Exactly the payable term lines of a purchase document sit on payable accounts, and exactly they carry a maturity date | the payable-and-receivable check |
| I7 | The instalment amounts produced by a payment term sum exactly to the document total, in both currencies | the balance-line rule of the distribution algorithm |
| I8 | Within one journal, no two posted documents share a number other than the placeholder `/` | a unique database index |
| I9 | Within one bank journal, no two posted payments share a cheque number when compared as integers | an explicit uniqueness check |
| I10 | A purchase document lives in a purchase journal | a validation |
| I11 | A cancelled document keeps its number, so the numbering series has no hole caused by cancellation | the cancel operation does not clear the number |

---

## 2. Validations at creation and at every write

### 2.1 Creation

| Condition | Message |
|---|---|
| A creation payload asks for the `posted` status directly | *You cannot create a move already in the posted state. Please create a draft move and post it after.* |

After creation the *manually modified* flag is forced to false, so that a decoder's output is not counted as a human edit.

### 2.2 Write on any document

| # | Condition | Message | Kind |
|---|---|---|---|
| W1 | Setting the reviewed flag without the review authority | *You don't have the access rights to perform this action.* | access error |
| W2 | Setting the status back to `draft` on a reviewed document without the review authority | *Validated entries can only be changed by your accountant.* | validation error |
| W3 | Touching a hash-protected field (the number, the accounting date, the journal, the company, or the hash itself) on a hashed document | *This document is protected by a hash. Therefore, you cannot edit the following fields: «the field labels, comma-separated».* | user error |
| W4 | Changing the journal of a document that has been posted before, while its number is neither empty nor `/` and is not being cleared in the same write | *You cannot edit the journal of an account move if it has been posted once, unless the name is removed or set to "/". This might create a gap in the sequence.* | user error |
| W5 | Changing the journal of a document whose number is assigned and whose sequence number is neither 0 nor 1, while quick encoding is off and the number is not being cleared in the same write | *You cannot edit the journal of an account move with a sequence number assigned, unless the name is removed or set to "/". This might create a gap in the sequence.* | user error |
| W6 | Changing the number or the accounting date of a **posted** document | the fiscal lock date check and the tax lock date check are re-run; see §4 | — |
| W7 | Moving a **posted** document out of the posted status | the same two checks are re-run | — |
| W8 | Writing any of the lines, the bill date, the accounting date, the partner, the payment term, the currency, the fiscal position or the cash rounding rule on a document that is or becomes `posted`, unless the read-only check is explicitly skipped | *You cannot modify the following readonly fields on the posted move «the number, or the vendor reference, or the identifier»: «the field names, comma-separated»* | user error |
| W9 | Writing a number that does not match the journal's numbering override pattern, by a user who is not an accounting manager | *The Journal Entry sequence is not conform to the current format. Only the Accountant can change it.* | user error. An accounting manager instead **clears** the journal's override pattern and the write proceeds |

Writing the numbering fields (the sequence prefix, the sequence number, the journal or the number) also re-evaluates the gap flags of this document and of its neighbours in the series.

### 2.3 Balance

| Condition | Message |
|---|---|
| Exactly one document is unbalanced after the write | *The entry is not balanced.* |
| Several documents are unbalanced | *The following entries are unbalanced:* followed by one line per document reading two spaces, a dash, a space and the document number |

The check is a single grouped query over the journal items, rounding the summed balance to the company currency's decimal places, and it deliberately avoids depending on computed stored fields because it also runs during creation.

### 2.4 Deletion

| # | Condition | Message |
|---|---|---|
| D1 | Deleting a document that has consumed a number and is **not** the last of its numbering chain, by a user who is not an accounting manager, while quick encoding is off for every company involved and the forced-delete marker is absent | *You cannot delete this entry, as it has already consumed a sequence number and is not the last one in the chain. You should probably revert it instead.* |
| D2 | Deleting a document that has been posted before, in a company with the restrictive audit trail, without the forced-delete marker | *To keep the restrictive audit trail, you can not delete journal entries once they have been posted.* newline *Instead, you can cancel the journal entry.* |

When deletion does proceed under the forced-delete marker on documents that were posted before in a restrictive-audit-trail company, a technical log line is written recording the acting user, and, per document, its number, identifier, total, currency and partner, followed by one line per account with that account's name, identifier and balance.

Deleting a document first removes every reconciliation of its journal items, then deletes the items, then the document.

---

## 3. Validations at posting

The posting routine collects **all** applicable messages and raises them together, one per line, except where marked *immediate*.

### 3.1 Authority

| Condition | Message | Kind |
|---|---|---|
| The caller is not a superuser and does not hold the invoicing group | *You don't have the access rights to post an invoice.* | access error, immediate |

### 3.2 Quick encoding

| Condition | Message |
|---|---|
| Quick encoding is on for this document, a total was typed, and it differs from the computed total in the document currency | *The current total is «the computed total» but the expected total is «the typed total». In order to post the invoice/bill, you can adjust its lines or the expected Total (tax inc.).* |

### 3.3 Party and bank account

| Condition | Message | Kind |
|---|---|---|
| The recipient bank account is set but archived | *The recipient bank account linked to this invoice is archived.* newline *So you cannot confirm the invoice.* | collected |
| The document is **inbound** (a vendor credit note on the payable side), its recipient bank account is set, that account is not trusted for outgoing payments, and the caller is the superuser, a public user or a portal user | — no message; the recipient bank account is silently **cleared**, so that automated flows are not blocked | — |
| Same, but the caller is a normal user **who may grant trust** | *The company bank account («the account's display name») linked to this invoice is not trusted. Go to the Bank Settings, double-check that it is yours or correct the number, and click on Send Money to trust it.* offered with a navigation to that bank account labelled *Bank settings* | redirecting warning, immediate |
| Same, but the caller may **not** grant trust | *The bank account of your company is not trusted. Please ask an admin or someone with approval rights to check it.* | user error, immediate |
| The document total is strictly negative in the document currency | *You cannot validate an invoice with a negative total amount. You should create a credit note instead. Use the action menu to transform it into a credit note or refund.* | collected |
| A **purchase** document has no vendor | *The field 'Vendor' is required, please complete it to validate the Vendor Bill.* | collected |
| A **sale** document has no customer | *The 'Customer' field is required to validate the invoice.* newline *You probably don't want to explain to your auditor that you invoiced an invisible man :)* | collected |

### 3.4 Dates

| Condition | Behaviour |
|---|---|
| A **sale** document has no bill date | the bill date is set to today; when the user had overridden the currency rate, that override is preserved while the date is written |
| A **purchase** document has no bill date | refused: *The Bill/Refund date is required to validate this document.* |

### 3.5 Lines and journal

| Condition | Message |
|---|---|
| A line's account is archived, the line is not marked imported, and the deprecation check is not skipped | *The account «the account name» («the account code») is archived.* (immediate, raised by the account-and-journal check) |
| A line's account forces a secondary currency that is neither the company currency nor the line currency | *The account selected on your journal entry forces to provide a secondary currency. You should remove the secondary currency on the account.* (immediate) |
| The document is already `posted` or `cancel` | *The entry «the number» (id «the identifier») must be in draft.* |
| The document has no line other than sections, subsections and notes | *Even magicians can't post nothing!* |
| Hard (non-soft) posting of a document whose automatic posting mode is not `no` and whose accounting date is later than today | *This move is configured to be auto-posted on «the date in the user's date format»* |
| The journal is archived | *You cannot post an entry in an archived journal («the journal's display name»)* |
| The document currency is archived | *You cannot validate a document with an inactive currency: «the currency code»* |
| Any line's account is archived (checked again over the whole document, unless skipped) | *A line of this move is using a archived account, you cannot post it.* |
| Any line's account belongs to none of the document company's own or parent companies | *The entry is using accounts («the accounts' display names, listed»») from a different company.* |
| Any analytic account referenced by any line's distribution is archived | *You cannot post an entry with an archived analytic account: «the names, comma-separated»* (immediate, raised after the collected batch) |

---

## 4. Lock dates

### 4.1 The hard fiscal lock check

Run whenever a posted document's number or accounting date changes, and whenever a posted document leaves the posted status. It asks the company for the **hard** violations of the fiscal-year lock, and — depending on the journal type — of the sale lock or the purchase lock, at the document's accounting date. The tax lock is **not** part of this check.

| Condition | Message |
|---|---|
| At least one such lock is violated | *You cannot add/modify entries prior to and inclusive of: «the formatted lock dates».* |

The check is skipped entirely when the explicit bypass marker is present in the context.

### 4.2 The hard tax lock check

Run in the same circumstances, on the journal items. For each item of a **posted** document, it asks for the hard violations of the **tax** lock at the document's accounting date, and fires only when the item actually affects the tax report — that is, it carries taxes, is itself a tax line, or carries at least one reporting grid whose applicability is *taxes*.

| Condition | Message |
|---|---|
| A tax-affecting item of a posted document sits in a tax-locked period | *The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: «the formatted lock dates».* |

### 4.3 The soft shift at posting

Posting does **not** refuse a locked period outright; it moves the accounting date forward to the first open date by the algorithm of `calculations.md` §2.3, and warns beforehand through the banner of `calculations.md` §2.4.

### 4.4 Deletability

A document may be unlinked only when its accounting date is strictly later than the effective lock date for its journal (in addition to the other conditions of `state-machines.md` §1.6).

---

## 5. Line-level rules on a purchase document

### 5.1 Payable and receivable coherence

| Condition | Message |
|---|---|
| On a **purchase** document, a line sits on a **receivable** account | *Account «the account code» is of receivable type, but is used in a purchase operation.* |
| On a **purchase** document, a line is a payable term line but its account is not payable, **or** its account is payable but it is not a payable term line | *Any journal item on a payable account must have a due date and vice versa.* |
| On a **sale** document, a line sits on a **payable** account | *Account «the account code» is of payable type, but is used in a sale operation.* |
| On a **sale** document, the mirror-image mismatch | *Any journal item on a receivable account must have a due date and vice versa.* |

### 5.2 Off-balance accounts

| Condition | Message |
|---|---|
| A document mixes an off-balance account with any other account type | *If you want to use "Off-Balance Sheet" accounts, all the accounts of the journal entry must be of this type* |
| A line on an off-balance account carries taxes, or is itself a tax line | *You cannot use taxes on lines with an Off-Balance account* |
| A line on an off-balance account is reconciled | *Lines from "Off-Balance Sheet" accounts cannot be reconciled* |

### 5.3 Deductibility

| Condition | Message |
|---|---|
| A line whose deductibility differs from 100 (compared to two decimal digits) on a document that is **not** a purchase document (receipts included) | *Only vendor bills allow for deductibility of product/services.* |
| A deductibility below 0 or above 100 | *The deductibility must be a value between 0 and 100.* |

Consequence of posting a vendor bill carrying a line below 100 % deductibility: the acting user is **automatically granted** the partial-purchase-deductibility group when they did not have it, so that the private-share lines stay visible to them afterwards.

### 5.4 Taxes and countries

| Condition | Message |
|---|---|
| The document's lines carry taxes belonging to countries other than the document's tax country, **and** the document has a fiscal position whose country is not among them | *This entry contains taxes that are not compatible with your fiscal position. Check the country set in fiscal position and in your tax configuration.* |
| The same, without a fiscal position | *This entry contains one or more taxes that are incompatible with your fiscal country. Check company fiscal country in the settings and tax country in taxes configuration.* |

The document's **tax country** is the fiscal position's country when the position declares a foreign tax registration, and the company's fiscal country otherwise.

### 5.5 Cash-basis and ordinary taxes sharing a grid

| Condition | Message |
|---|---|
| A line carries both a tax exigible on payment and a tax exigible on invoice whose **base** distributions share at least one reporting grid; or a tax line whose grids overlap with the base grids of the other exigibility | *Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag.* |

### 5.6 Reconciled lines

| Condition | Message |
|---|---|
| A modification that the system classifies as legally significant is attempted on a line of a **posted** document that has at least one partial reconciliation | *You cannot do this modification on a reconciled journal entry. You can just change some non legal fields or you must unreconcile first.* newline *Journal Entry (id): «the number» («the identifier»)* |

---

## 6. Journal and document type

| Condition | Message |
|---|---|
| A purchase document (receipts included) whose journal is not of type *purchase* | *Cannot create a purchase document in a non purchase journal* |
| A sale document (receipts included) whose journal is not of type *sale* | *Cannot create a sale document in a non sale journal* |
| A document whose automatic posting mode is not `no`, which is a purchase document, and which has no bill date | *For this entry to be automatically posted, it required a bill date.* |
| An invoice-like document whose currency differs from the company currency and whose document rate is not strictly positive | *The currency rate must be strictly positive.* |

---

## 7. Reset to draft, cancel and reverse

| # | Condition | Message |
|---|---|---|
| R1 | Resetting a document that is neither posted nor cancelled | *Only posted/cancelled journal entries can be reset to draft.* |
| R2 | Resetting a document that requires a cancellation request | *You can't reset to draft those journal entries. You need to request a cancellation instead.* |
| R3 | Resetting an exchange-difference entry | *You cannot reset to draft an exchange difference journal entry.* |
| R4 | Resetting a cash-basis entry, or one created from a cash-basis origin | *You cannot reset to draft a tax cash basis journal entry.* |
| R5 | Resetting a hashed entry | *You cannot reset to draft a locked journal entry.* |
| R6 | Cancelling a document that, after the automatic reset, is still not draft | *Only draft journal entries can be cancelled.* |
| R7 | Requesting a cancellation on a document that does not need one | *You can only request a cancellation for invoice sent to the government.* |
| R8 | Reversing documents belonging to more than one company | *All selected moves for reversal must belong to the same company.* |
| R9 | Reversing a document that is not posted | *To reverse a journal entry, it has to be posted first.* |
| R10 | Reversing into a journal whose type differs from the original's | *Journal should be the same type as the reversed entry.* |
| R11 | Switching a document's type when it has ever been numbered | *You cannot switch the type of a document with an existing sequence number.* |
| R12 | Switching the type of a miscellaneous entry | *This action isn't available for this document.* |

The reset-to-draft button is hidden (rather than refusing) when the document is hashed, when the hash rules restrict it, or when it needs a cancellation request; and it is shown only for documents in the `cancel` status or in the `posted` status.

---

## 8. Payment-related rules

| Condition | Message |
|---|---|
| Registering a payment on a document that is not posted | *You can only register payment for posted journal entries.* |
| Registering a payment on a miscellaneous entry | *You cannot register payments for miscellaneous entries.* |
| Registering a payment on a document whose payment status is `blocked` | *You cannot register payments for blocked invoices.* |
| Blocking a document already paid or in payment | *You can't block a paid invoice.* |
| Posting an outgoing payment whose method requires a recipient bank account, when that account is not trusted for outgoing payments | *To record payments with «the payment method line's name», the recipient bank account must be manually validated. You should go on the partner bank account of «the partner's display name» in order to validate it.* |

---

## 9. Payment term rules

| Condition | Message |
|---|---|
| The percent lines of a term do not sum to exactly 100, at the *Payment Terms* decimal precision | *The Payment Term must have at least one percent line and the sum of the percent must be 100%.* |
| The early payment discount is enabled on a term with more than one line | *The Early Payment Discount functionality can only be used with payment terms using a single 100% line. * |
| The early payment discount percentage is not strictly positive while the discount is enabled | *The Early Payment Discount must be strictly positive.* |
| The early payment discount day count is not strictly positive while the discount is enabled | *The Early Payment Discount days must be strictly positive.* |
| A percent line's amount is below 0 or above 100 | *Percentages on the Payment Terms lines must be between 0 and 100.* |
| A term line's "days on the next month" value is not numeric | *The days added must be a number and has to be between 0 and 31.* |
| That value is numeric but outside 0 … 31 | *The days added must be between 0 and 31.* |
| Deleting a payment term that at least one document still references | *Uh-oh! Those payment terms are quite popular and can't be deleted since there are still some records referencing them. How about archiving them instead?* |

---

## 10. Cheque printing rules

### 10.1 Numbering

| Condition | Message |
|---|---|
| A cheque number containing anything other than decimal digits | *Check numbers can only consist of digits* |
| Two posted payments in the same bank journal whose cheque numbers are equal when read as integers | *The following numbers are already used:* newline, then one line per clash reading *«the number» in journal «the journal's display name»* |
| Writing a journal's next cheque number with anything other than digits | *Next Check Number should only contains numbers.* |
| Writing a journal's next cheque number lower than the sequence's current next value | *The last check number was «the current next value». In order to avoid a check being rejected by the bank, you can only use a greater number.* |
| Writing a journal's next cheque number greater than 2 147 483 647 | *The check number you entered («the value») exceeds the maximum allowed value of 2147483647. Please enter a smaller number.* |
| Entering a non-numeric value in the pre-numbered printing dialogue | *Next Check Number should only contains numbers.* |

The cheque number field is presented as **read-only in every view**: the field description itself is forced read-only when the interface asks for it. It is set by posting, by the pre-numbered dialogue, or through data import.

### 10.2 Printing

| Condition | Message |
|---|---|
| No selected payment uses the cheque method, or every one of them has already been sent | *Payments to print as a checks must have 'Check' selected as payment method and not have already been reconciled* |
| The selection spans more than one bank journal | *In order to print multiple checks at once, they must belong to the same bank journal.* |
| No cheque layout is configured on the journal or the company, or it is the value `disabled` | *You have to choose a check layout. For this, go in Invoicing/Accounting Settings, search for 'Checks layout' and set one.* — offered with a navigation to the accounting configuration action labelled *Go to the configuration panel* |
| The configured layout no longer resolves to a printable document definition | *Something went wrong with Check Layout, please select another layout in Invoicing/Accounting Settings and try again.* — with the same navigation |

The *Print Check* button itself is hidden when the payment method is not the cheque method, when the payment has already been sent, or when the company's layout selection offers only the value `disabled`.

### 10.3 Voiding

Voiding is offered only when the payment method is the cheque method, the payment status is `in_process` and the cheque has been sent. It resets the payment to draft and then cancels it; the guards of §7 apply to the underlying entry reset.

---

## 11. Debit note rules

| Condition | Message |
|---|---|
| Any selected source is not posted | *You can only debit posted moves.* |
| Any selected source already has a debit note | *You can't make a debit note for an invoice that is already linked to a debit note.* |
| Any selected source is not a customer invoice, a customer credit note, a vendor bill or a vendor credit note | *You can make a debit note only for a Customer Invoice, a Customer Credit Note, a Vendor Bill or a Vendor Credit Note.* |

The *Copy Lines* switch is silently ignored when the source is a credit note.

---

## 12. Upload and mail rules

| Condition | Message |
|---|---|
| Uploading with no journal resolvable and a document type that is neither a sale nor a purchase type | *The journal in which to upload the invoice is not specified. * |
| Uploading with no attachment | *No attachment was provided* |
| Uploading when the company has no journal of the required type | *No journal could be found in company «the company's display name» for any of those types: «the types, comma-separated»* |
| A message arrives at a journal's address with **no attachment** | No document is created; a bounce is sent to the sender built from the mail-gateway failure template, carrying the alias company's electronic mail address and name (falling back to the current company's) |
| The alias's default company value is not an integer | *Default value for 'company_id' for «the record» is not an integer* |
| A decoder answers a refusal reason | the document's chatter receives *Attachment «the file name» not imported: «the reason»* |
| A decoder raises | the document's chatter receives, with elevated privileges, *Error importing attachment «the descriptor»:* then *This specific error occurred during the import:* then the error text; the transaction is rolled back |
| No file in the group has a usable decoder | nothing is written; a technical log line records *Attachment(s) «the names» not imported: no suitable decoder found.* |
| Creation from attachments produced a document that could not be extended | its chatter receives *There was an error while importing the bill, you can find attached the incoming XML* |

The decoding of a message posted on an **existing** document only runs when the posting user is **active and internal**. A message from a supplier therefore never rewrites a bill.

---

## 13. The sample bill

| Condition | Message |
|---|---|
| No purchase journal exists | *No journal could be found in company «the company's display name» for any of those types: purchase* |
| The demonstration partner does not exist | *You may only use samples in demo mode, try uploading one of your invoices instead.* |

---

## 14. Automatic posting rules

A bill is posted automatically only when **all** of these hold. Any one failing silently prevents automatic posting; only the duplicate case leaves a trace.

1. The company's *Auto-validate bills* switch is on.
2. The document has a vendor.
3. The document is a purchase document (receipts included).
4. The vendor's *Auto-post bills* policy is exactly `always`.
5. The document carries **no abnormal amount warning**.
6. The journal does **not** secure entries with a hash.
7. **No duplicate was detected.** When duplicates were detected the bill stays draft and the chatter records *Auto-post was disabled on this invoice because a potential duplicate was detected.*

The learning wizard opens only under the nine conditions of `workflows.md` §8.3, and only when the run of consecutive unmodified bills reaches **three**.

---

## 15. Locking, concurrency and ordering

| Aspect | Rule |
|---|---|
| Numbering | The number is drawn at posting from the journal's series. A no-gap series serialises concurrent posters; the hash-chain option adds a second serialisation point |
| Cheque numbering | The journal's cheque sequence is a **no-gap** sequence, so two users posting cheques on the same journal are serialised and cannot draw the same number |
| Duplicate detection | A read-only query; it never locks and never blocks. It may therefore miss a duplicate created in a concurrent, uncommitted transaction |
| Decoding | Runs inside a commit / roll-back guard, so a failing decoder never leaves a half-written document, but it **does** commit the work that preceded it |
| Dynamic line synchronization | Runs inside the balance check, so a synchronization that would unbalance the document raises before the write completes |
| Line ordering | Product and presentation lines keep their user order; tax lines sort at 10000, rounding lines at 11000, payable term lines at 12000; private-share product lines sort immediately after the line they derive from, and the private-share totals last |
| Document ordering | Accounting date descending, then number descending, then bill date descending, then identifier descending |

---

## 16. Edge cases

| Situation | Behaviour |
|---|---|
| A bill whose total is exactly zero | It still produces a payable term line of zero; posting immediately treats it as paid |
| A bill with a negative total | Posting is refused; the user is told to create a credit note or to switch the type |
| A bill whose vendor has no payable account and whose company partner has none either | The payable account falls back to the first active payable account of the company; when the company has none, the line has no account and the database check refuses the write |
| A bill whose vendor reference duplicates a document of **another** company | Not detected: the predicate requires the same company |
| A bill whose vendor reference duplicates a **cancelled** document | Not detected: only draft and posted documents are matched |
| A bill whose vendor reference duplicates a document of the same reference but in a different **calendar year** | Not detected: suppliers restart their numbering yearly |
| A bill with no vendor yet, typed against a draft document that has one | Detected, because the partner condition accepts "the examined document has no commercial partner and the candidate is a draft" |
| A bill in a foreign currency whose instalments do not divide evenly | The **last** term line absorbs the difference, so the entry balances exactly |
| A payment term whose last line is a *fixed* line | It is still treated as the balance line and takes the whole residual; its declared amount is ignored |
| A payment term line whose day count would land on 31 February | The month-addition clamps to the last valid day |
| A cheque printed across several stub pages | Only page one is negotiable; later pages show the literal text `VOID` in place of the amount and of the amount in words |
| A cheque stub that does not fit and multi-page stubs are off | The list is cropped to eight lines and the page is flagged as cropped, so the layout can print an ellipsis |
| A cheque voided and its number re-entered on another payment | Accepted by the uniqueness check, because that check only compares **posted** payments |
| An imported line whose account was archived after import | Accepted: the archived-account refusal skips lines marked imported |
| A bill posted while its accounting date falls in a locked period | The accounting date is moved forward silently; the banner warned beforehand |
| A bill whose journal secures entries with a hash, selected in a mass posting | Skipped unless *Force Hash* is ticked |
| A bill dated in the future, mass-posted with soft scheduling | Not posted; its automatic posting mode becomes *At Date* and the chatter records the scheduled date |
| Resetting to draft a bill that is the source of an exchange-difference entry | Refused indirectly: the exchange entry itself cannot be reset, and the reconciliation protection refuses the change |
| Deleting a bill in the middle of a numbering series | Refused for non-managers; a manager gets a warning but may proceed, and the gap flags of the neighbours are updated |
| Two bills of the same vendor, same date, same total, but different currencies | Not duplicates: the currency must match |
| A vendor credit note detected as a duplicate of a bill | Impossible: the type-compatibility condition pairs `in_invoice` with `in_receipt` and `out_invoice` with `out_receipt`, and otherwise requires equality; `in_refund` only matches `in_refund` |
