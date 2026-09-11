# Glossary

Every term used in the payments and bank reconciliation domain, defined in full. Entries are alphabetical. Where a term has a stored name, that name is given in code font with its full meaning in words.

---

## A

**Accounting date**
The date at which a Journal Entry is recognised in the books. For a Payment's entry it is the Payment's date; for a Bank Transaction's entry it is the transaction's date; for an exchange-difference entry it is the later of the two matched items' dates, pushed forward past any lock date and then raised again to the date of each item it fixes. It is distinct from the date the record was created.

**Account holder**
The partner that owns a Bank Account (`partner_id`, meaning the account holder). A Bank Account always has one; a Bank Account whose holder is the company's own partner is a company account and may back a liquidity journal.

**Account holder name**
The name printed on the account statement (`acc_holder_name`), which may differ from the holder's own name. It defaults to the holder's name and is used as the beneficiary name in a payable quick response code.

**Account number**
The identifier of a Bank Account at its bank (`acc_number`). Stored as typed, except that a number that validates as an international bank account number is rewritten in groups of four. Its searchable form is the sanitized number.

**Allocation**
The act of deciding which accounts a Bank Transaction's amount belongs to, by replacing its suspense journal item with one or more counterpart journal items. An allocation is complete when nothing is left on the suspense account.

**Amount in currency**
See *Foreign amount*.

**Anchor**
In the running-balance and starting-balance computations, the last posted Bank Transaction before a given point that belongs to a statement. The computation restarts from the anchor's statement rather than replaying the whole history. In the running-balance walk, any transaction that is the first of a statement re-anchors the running total to that statement's starting balance, even when the transaction is draft or cancelled.

**Automated model**
A Reconciliation Model whose trigger is `auto_reconcile`. When it matches a Bank Transaction, its counterpart lines are written and reconciled without asking the user, and the transaction's entry is also marked as checked.

**Available payment method line**
A Payment Method Line of the Payment's journal whose direction matches the Payment's direction, minus the codes excluded by the exclusion hook. The Payment's method must be one of them.

---

## B

**Bank** (`res.bank`, table `res_bank`)
A financial institution, with a name, a postal address, an electronic-mail address, a telephone number and a bank identifier code. The bank identifier code is stored upper-cased and is indexed for search; the display name is the bank's name followed by " - " and the code when there is one.

**Bank Account** (`res.partner.bank`, table `res_partner_bank`)
An account number held by a partner at a bank. It carries the sanitized number, the inferred type, the holder, the holder name, an optional clearing number, an optional currency, an optional bank, a sequence that orders a partner's accounts, and the trust flag. It is archived rather than deleted.

**Bank feed** (`bank_statements_source`)
How a liquidity journal's transactions arrive. The base value is `undefined`; capability packages that import files or synchronise with a provider add their own values.

**Bank identifier code** (`bic`)
The international code identifying a bank, sometimes called a routing code. Stored upper-cased on the Bank record and read through the Bank Account as `bank_bic`.

**Bank Statement** (`account.bank.statement`, table `account_bank_statement`)
A checkpoint over a contiguous run of Bank Transactions of one journal, carrying a starting balance, a computed ending balance and a reported ending balance. It is optional: transactions may exist without ever being grouped into one.

**Bank Transaction** (`account.bank.statement.line`, table `account_bank_statement_line`)
One movement reported by the bank on a liquidity account. Called a *statement line* in the storage names. It owns a Journal Entry that is created and posted at once, with a liquidity journal item and, until the transaction is explained, a suspense journal item.

**Basic bank account number**
The part of an international bank account number that follows the four leading characters, that is, the country code and the two check digits. It is **not** the same as the domestic account number of most countries.

**Batch**
In the register-payment flow, a group of selected journal items that share a counterparty, an account, a currency, a recipient bank account and a counterparty kind. One batch produces one Payment when grouping is on, or one Payment per document when it is off. Batches may be merged (see *Batch merge*).

**Batch key**
The five-part tuple that decides which batch a journal item belongs to: the counterparty, the account, the currency, the recipient bank account (taken from the document when the document is invoice-like, otherwise empty) and the counterparty kind.

**Batch merge**
The rule that combines two batches differing only by their recipient bank account, applied when the counterparty has exactly one distinct inbound recipient account **and** exactly one distinct outbound recipient account. It exists so that a counterparty with one open invoice and one open credit note yields one net Payment.

**Billing user**
The role that may create and edit Payments, use the register-payment screen, read bank statements and transactions and update reconciliation models, but cannot see journal entries or accounting reports.

**Blocked**
A payment state (`blocked`) set manually on a document to exclude it from payment flows. The register-payment screen refuses to open on a blocked document.

---

## C

**Cancelled**
Either a state of a Payment (`canceled`), meaning the movement was abandoned; or a state of a Journal Entry (`cancel`), meaning the entry is void. A cancelled Bank Transaction contributes nothing to any balance but is still used as an anchor when checking contiguity.

**Cash-basis tax entry**
A Journal Entry produced when a reconciliation touches a receivable or payable account of a company that uses cash-basis taxes. It moves tax from a transitional account to the real tax account in proportion to what has been paid. Its content belongs to `../taxes/`; this domain supplies the trigger, the percentage, the payment rate, the settlement date and the both-posted flag.

**Cash-discount account**
One of two company accounts used when an early payment discount is granted: the **loss** account for an inbound document — a customer invoice, a customer receipt or a vendor credit note — and the **gain** account for an outbound document — a vendor bill, a vendor receipt or a customer credit note.

**Checked**
A boolean on a Journal Entry meaning that an accountant has reviewed it. For a Bank Transaction it decides which figure of the dashboard counts it, and it changes how the residual is computed: an unchecked transaction's residual is its whole amount, whatever its journal items say.

**Clearing number**
A national routing number stored on a Bank Account (`clearing_number`), used where the account number alone does not identify the branch.

**Collapse rule**
The rule in the foreign-currency matching that, when the two company-currency images of the matched foreign amount lie inside each other's rounding intervals, equalises both sides at the smaller remaining residual, so no spurious exchange difference is produced.

**Company currency**
The currency the books are kept in. Every balance is expressed in it. It is the pivot of every conversion between two foreign currencies.

**Completeness**
The property of a Bank Statement whose computed ending balance equals its reported ending balance, and which has at least one posted transaction. A statement may be saved and used while incomplete; the condition only produces a warning.

**Connected group**
The set of journal items reachable from a given item through matchings. It is found through the matching number rather than by walking the graph: every item carrying one of the numbers of the starting set belongs to it, except items whose number is an import placeholder.

**Counterpart journal item**
On a Payment's entry, the journal item on the destination account — the receivable or payable line that the settled documents are matched against. On a Bank Transaction's entry, one of the items that replaces the suspense item once the transaction is allocated.

**Counterparty**
The partner on the other side of a movement: the customer who pays, the vendor who is paid, or the company's own partner for an internal transfer. Stored as `partner_id` on a Payment and on a Bank Transaction.

**Counterparty kind** (`partner_type`)
Either `customer` or `supplier`. It selects which of the counterparty's two default accounts is used as the Payment's destination account, and it labels the counterparty field in the user interface.

**Credit**
The right-hand side of a journal item. A credit is a negative balance. An inbound Payment credits its destination account; an outbound Payment credits its outstanding account.

**Currency of a journal item** (`currency_id`)
The currency the item's foreign amount is expressed in. It equals the company currency when the item carries no foreign amount.

---

## D

**Debit**
The left-hand side of a journal item. A debit is a positive balance.

**Deferred matching**
The mechanism for journal items imported with a matching number starting with the letter `I`. Those items are not matched at import. Once every entry of a group sharing one placeholder number and one account is posted, the group is reconciled — the account being made reconcilable first when needed — with exchange differences and cash-basis entries disabled.

**Destination account** (`destination_account_id`)
The account the counterpart journal item of a Payment is written on: the counterparty's receivable account for a customer payment, its payable account for a vendor payment, the company's inter-bank transfer account for an internal transfer, or the company's first account of the relevant type when there is no counterparty.

**Direct bank payment**
A Payment whose outstanding account **is** the journal's own default account. It is matched by definition: there is nothing for a bank transaction to confirm. Such payments appear in a separate dashboard figure.

**Direction** (`payment_type`)
Either `inbound` — the company receives money — or `outbound` — the company sends money. It carries the sign of the movement; the Payment's amount itself is always positive.

**Discount mode**
The state of the register-payment screen when an early payment discount applies and the entered amount equals either the default or the full total. In discount mode the difference handling is forced to *mark as fully paid*, the write-off section is hidden, and the difference becomes the discount counterpart lines rather than a write-off.

**Document**
In this folder, any invoice, credit note, bill, refund or receipt whose receivable or payable journal item a payment settles.

---

## E

**Early payment discount**
A reduction granted when a document is paid before a date set by its payment term. Its definition belongs to `../accounts-receivable/`; this domain reads it when proposing an amount and books its counterpart lines when the payment is created.

**Eligibility check (matching)**
The five conditions a set of journal items must satisfy before it may be reconciled: none already reconciled, none cancelled, one account, one root company, and an account that is either reconcilable or of a cash or credit-card type.

**Eligibility domain (payment method)**
The set of journals a payment method may be attached to, derived from its registry entry: the journal types it accepts, optionally the currencies, optionally the country of the company's fiscal country.

**Exchange difference**
The gap, in the company currency, between the two sides of a matching whose foreign amounts agree but whose company-currency images do not, because the rate moved between the two documents. It is booked in a dedicated entry on the company's exchange gain or loss account.

**Exchange-line mode**
The condition under which no rate is applied to a matching: the reconciliation currency is the company currency, both items share a currency, and at least one of them has no foreign residual left. It is the case of matching an exchange-difference line, which must only change the company-currency amount.

---

## F

**First-line index** (`first_line_index`)
The smallest ordering index among a Bank Statement's transactions. Statements are ordered and chained by it, because two statements may share a date.

**Foreign amount** (`amount_currency`)
The signed amount of a journal item in the item's own currency: positive on the debit side, negative on the credit side. A Bank Transaction also has a foreign amount, which is the amount in the transaction's foreign currency when it has one.

**Foreign currency of a transaction** (`foreign_currency_id`)
The second currency the bank reported for a Bank Transaction. It must differ from the journal currency; a value equal to it is silently dropped at creation and refused afterwards.

**Forced balance**
A value passed when creating a Payment that replaces the computed company-currency amount of the liquidity item. Used when a write-off against an exchange-difference account is chosen and the payment currency is not the company currency, so the whole difference lands on the exchange side.

**Forced rate**
A rate passed to a reconciliation that replaces the rate table when converting a residual. Used when a write-off against an exchange-difference account is chosen and the payment currency **is** the company currency.

**Full Reconciliation** (`account.full.reconcile`, table `account_full_reconcile`)
The marker created when a connected group of matched journal items has no residual left. It links every matching and every item of the group, and gives them their permanent matching number.

---

## G

**Group payment**
A single Payment created for several documents of one counterparty, as opposed to one Payment per document. Controlled by the grouping switch of the register-payment screen.

**Group-payment sequence**
The per-company, gapless, year-ranged number sequence with the prefix `GROUP/<the year>/` and five digits of padding, used as the memo of a grouped customer payment.

---

## I

**Import placeholder**
A matching number starting with the letter `I`, written on journal items at import to say which of them belong together without matching them yet. See *Deferred matching*.

**In payment**
The payment state of a document that is fully covered but whose covering Payments have not all been confirmed by the bank. The state only exists when the full accounting capability is installed; otherwise the in-payment hook reports `paid` instead.

**In process** (`in_process`)
The state of a Payment that has been committed — its entry exists and is posted — but whose money the bank has not yet confirmed.

**Inbound**
See *Direction*.

**Installment**
One payment-term journal item of a document, with a due date and an amount. A document with several of them is paid in parts. The register-payment screen classifies each installment as overdue, next, before a given date, an early payment discount, or other, and derives the proposed amount from that classification.

**Installment mode**
The mode of the register-payment screen: `next`, `overdue`, `before_date` or `full`. It decides which of the four totals the proposed amount follows and which explanatory sentence is shown.

**Inter-bank transfer account** (`transfer_account_id`)
The company's reconcilable account of type `asset_current` through which money passes when it is moved between two liquidity journals. The shipped chart template creates it as "Liquidity Transfer".

**Internal transfer**
A movement of money between two of the company's own liquidity journals, recorded as two Payments facing each other through the inter-bank transfer account and cross-referenced through the paired-transfer field.

**International bank account number**
The standardised account identifier beginning with a two-letter country code and two check digits. This domain validates it with the modulo ninety-seven arithmetic of `calculations.md`, section 14, and derives from it the basic bank account number and the national bank, branch, account and check parts.

---

## J

**Journal** (`account.journal`, table `account_journal`)
Defined in `../general-ledger/`. This domain uses the three liquidity types — `bank`, `cash` and `credit` — and their default account, suspense account, payment method lines, bank account and bank-feed source.

**Journal amount**
In a Bank Transaction's arithmetic, the transaction's amount expressed in the journal's currency. It is the amount the bank actually moved on the account.

**Journal currency**
The currency a liquidity journal is held in. When the journal names no currency, it is the company currency.

**Journal Entry** (`account.move`, table `account_move`)
Defined in `../general-ledger/`. Every Payment that produces accounting owns one; every Bank Transaction owns one and delegates to it.

**Journal Item** (`account.move.line`, table `account_move_line`)
Defined in `../general-ledger/`. The unit of accounting. This domain reads and writes its account, balance, foreign amount, currency, residuals, matchings and matching number.

---

## L

**Label of a transaction** (`payment_ref`)
The description the bank reported for a Bank Transaction. It becomes the label of both journal items of the transaction's entry, and it is the text a reconciliation model's label condition is tested against, together with the raw payload and the note.

**Liquidity account**
An account of type `asset_cash` or `liability_credit_card` that represents money actually held: a bank account, a cash box, a credit-card account.

**Liquidity journal**
A journal of type `bank`, `cash` or `credit`. Every Payment and every Bank Transaction lives in one.

**Liquidity journal item**
On a Payment's entry, the item on the outstanding account — the side the bank will later confirm. On a Bank Transaction's entry, the item on the journal's default account — the side the bank has already confirmed. Exactly one liquidity item must exist on a Bank Transaction's entry at all times.

**Lock date**
A date past which entries may no longer be written, defined in `../general-ledger/`. Reconciling itself does not check lock dates, because it writes no journal item on the matched entries; only the derived entries — exchange differences, cash-basis entries and their reversals — are dated past them.

---

## M

**Matched** (`is_matched`)
The flag on a Payment saying that the bank has confirmed the money: the liquidity item has no residual left, or the journal's default account is itself the liquidity account.

**Matching**
See *Partial Reconciliation*.

**Matching number** (`matching_number`)
The label of the matching group a journal item belongs to. It is the decimal text of the Full Reconciliation's identifier when the group is fully matched, the letter `P` followed by the smallest matching identifier of the component when it is only partially matched, a value starting with `I` for an import placeholder, and empty when the item is unmatched.

**Memo** (`memo` on a Payment, `communication` on the register-payment screen)
The free label of a movement. Writing it on a Payment also writes it as the reference of the Payment's entry.

**Merchant-presented code**
A payable quick response code presented by the payee and scanned by the payer, encoded as tag-length-value fields ending with a sixteen-bit checksum. Its generator code is `emv_qr`.

**Method information registry**
The run-time table that says, for each payment method code, how it may be attached to journals: its multiplicity mode, the journal types it accepts, the currencies it requires and the country it requires.

**Mimicking rule**
The rule that lets a journal item booked in the company currency, on a receivable or payable account, take part in a foreign-currency reconciliation by pretending it also carries the counterpart's currency, converted at the rate of the relevant date.

**Multiplicity mode**
One of three values in the method information registry: `unique` (at most one journal per company), `electronic` (at most one journal per company and per online provider), `multi` (unlimited, repeatable on one journal).

---

## O

**Ordering index** (`internal_index`)
The text sort key of a Bank Transaction: the date on eight digits, then the complement of the sequence on ten digits, then the identifier on ten digits. It makes "the transactions before this one" a single string comparison.

**Outbound**
See *Direction*.

**Outstanding account** (`outstanding_account_id`)
The account a Payment's liquidity item is written on while the bank has not confirmed the movement. It is the payment account of the Payment's method line. The shipped chart template creates two of them: *Outstanding Receipts* for money coming in and *Outstanding Payments* for money going out, both reconcilable.

**Open balance**
In a reconciliation model's evaluation, what is still unexplained at the moment a line is evaluated, signed opposite to the transaction amount. It starts at minus the transaction amount and is reduced by each line already written.

---

## P

**Paid**
Either a Payment state (`paid`), meaning the money is confirmed; or a document's payment state (`paid`), meaning the document is fully covered and every covering Payment is matched.

**Paired transfer** (`paired_internal_transfer_payment_id`)
The cross-reference between the two Payments of an internal transfer.

**Partial Reconciliation** (`account.partial.reconcile`, table `account_partial_reconcile`)
The atom of matching: one record linking one debit journal item to one credit journal item, carrying the matched amount in three currencies — the company currency, the debit item's currency and the credit item's currency — all three always positive.

**Partner mapping**
A Reconciliation Model whose only purpose is to say which counterparty a transaction belongs to. It has a label condition, exactly one line, and that line names a counterparty and no account. It produces no counterpart line and is never proposed spontaneously.

**Payable code**
See *Quick response code*.

**Payment** (`account.payment`, table `account_payment`)
One movement of money between the company and one counterparty, owning at most one Journal Entry.

**Payment account** (`payment_account_id` on a Payment Method Line)
The outstanding account configured for one payment method on one journal. It becomes the Payment's outstanding account.

**Payment difference**
The gap between what the selected documents ask for and the amount entered on the register-payment screen. It may be kept open — leaving the documents partially paid — or written off, or, in discount mode, booked as the discount.

**Payment Method** (`account.payment.method`, table `account_payment_method`)
A catalogue entry for a way money moves, identified by a code and a direction. It is global: it belongs to no company and no journal.

**Payment Method Line** (`account.payment.method.line`, table `account_payment_method_line`)
The activation of one Payment Method on one Journal, carrying the label to display and the payment account to use.

**Payment Register** (`account.payment.register`)
The transient record that collects the user's choices while turning selected journal items into Payments.

**Payment reference** (`payment_reference` on a Payment)
The reference of the instrument used, for example a check number or the name of a transfer file. Distinct from the memo.

**Payment state** (`payment_state` on a Journal Entry)
The derived state of a document with respect to settlement: `not_paid`, `partial`, `in_payment`, `paid`, `reversed`, `blocked` or the retained legacy marker.

**Payment-like item**
A journal item whose entry has an originating Payment or an originating Bank Transaction. The rate rule prefers a payment-like item's own accounting rate over the rate table when the other item is not payment-like.

**Percentage of balance**
A reconciliation model line mode (`percentage`) whose amount is a percentage of the open balance at the moment the line is evaluated — so it consumes what the previous lines left.

**Percentage of statement line**
A reconciliation model line mode (`percentage_st_line`) whose amount is a percentage of the transaction amount as reported by the bank, regardless of what the previous lines consumed.

**Posted**
The state of a Journal Entry that has been committed to the books. A Bank Transaction's entry is posted from the moment it is created.

**Pretty form**
The presentation of a valid international bank account number in groups of four characters separated by single spaces.

---

## Q

**Quick response code**
A two-dimensional barcode that encodes payment instructions so a banking application can read them. Two generators are shipped: the Single Euro Payments Area credit transfer code (`sct_qr`, sequence 20) and the merchant-presented code (`emv_qr`, sequence 30). The first one that is eligible and whose data check passes is used.

---

## R

**Reconcilable account**
An account whose items may be matched against each other. Only items on a reconcilable account — or on an account of type `asset_cash` or `liability_credit_card` — have residuals and take part in reconciliation.

**Reconciled (journal item)** (`reconciled`)
True when both residuals of the item are zero.

**Reconciled (Payment)** (`is_reconciled`)
The flag saying that the documents have been settled: the counterpart and write-off items on a reconcilable account have no residual left.

**Reconciled (Bank Transaction)** (`is_reconciled`)
The flag saying that the transaction is fully explained: nothing is left on the suspense account.

**Reconciliation**
The act of recording that one debit journal item and one credit journal item settle each other, wholly or partly. It writes no accounting on the matched items; it creates Partial Reconciliations and, when needed, exchange-difference and cash-basis entries.

**Reconciliation currency**
The currency a single matching is computed in: the debit item's currency when it differs from the company currency and is available on both sides; otherwise the credit item's currency under the same condition; otherwise the company currency.

**Reconciliation Model** (`account.reconcile.model`, table `account_reconcile_model`)
A named preset of matching conditions and counterpart lines, used while reconciling bank transactions.

**Reconciliation Model Line** (`account.reconcile.model.line`, table `account_reconcile_model_line`)
One counterpart journal item a model writes, with its account, label, taxes, analytic distribution and amount rule.

**Reconciliation plan**
The list that tells a reconciliation what to match and in which order. A member that is a set of items is matched as one group; a member that is itself a list is matched sub-group by sub-group and then as a whole.

**Recipient bank account** (`partner_bank_id`)
The Bank Account the money is sent to (outbound) or received on (inbound). For an inbound Payment it is the journal's own account; for an outbound one it is one of the counterparty's.

**Rejected** (`rejected`)
The state of a Payment refused by the outside world: a returned direct debit, a bounced check, a declined card. The Journal Entry is left as it is.

**Reported ending balance** (`balance_end_real`)
The ending balance the bank states for a statement. Its computation copies the computed balance, so a new statement is complete by construction and the user only overrides it when the bank says otherwise.

**Residual (journal item)** (`amount_residual`, `amount_residual_currency`)
What is left to match on a journal item, in the company currency and in the item's own currency. Computed as the item's amount minus what matchings have consumed on the debit side plus what they have consumed on the credit side, each sum rounded once.

**Residual (Bank Transaction)** (`amount_residual`)
What is left to explain on a transaction, signed like the entry's journal items, expressed in the transaction currency. It is the whole amount when the transaction is unchecked; otherwise it is read from the suspense items.

**Reversed**
The payment state of a document covered exclusively by its own reversal documents, never by money.

**Rounding step**
The smallest representable increment of a currency, equal to ten raised to minus the number of decimal places. Every monetary value is rounded to a multiple of it, with halves going away from zero.

**Running balance** (`running_balance`)
The balance of a liquidity journal after a given Bank Transaction, in occurrence order, anchored on statements. It is not the same as the accumulated balance of the journal's items, which follows the recognition order.

---

## S

**Sanitized account number** (`sanitized_acc_number`)
The account number with every non-alphanumeric character removed and the rest upper-cased. It is the column searches and the uniqueness constraint use.

**Sent** (`is_sent`)
The flag saying that a payment instruction has left the company: a check has been printed, a transfer file has been produced. It gates the *Reject* and *Cancel* buttons on a Payment in process.

**Sequence (bank transaction)** (`sequence`)
An integer that orders transactions sharing a date. It works in reverse: a **higher** sequence sorts **earlier**, because the ordering index stores its complement.

**Settlement date**
In the cash-basis arithmetic, the date at which the reconciled amount was paid, used as the accounting date of the cash-basis entry: the later of the two matched items' dates when both matched documents are invoice-like, and the counterpart line's date otherwise.

**Sibling companies**
Two or more companies that span more than one company and of which none is an ancestor of the others. A selection drawn from sibling companies is registered on their **root** company, and only by a user who may act for it.

**Signed amount** (`amount_signed`)
A Payment's amount with its direction applied: negative for an outbound Payment, positive for an inbound one.

**Single Euro Payments Area credit transfer code**
The payable quick response code used in the euro payment zone, encoded as twelve newline-separated fields beginning with the service tag `BCD`. Its generator code is `sct_qr`.

**Source amount**
On the register-payment screen, the absolute sum of the residuals of a batch's journal items: in the company currency (`source_amount`) and in the batch's currency (`source_amount_currency`).

**Starting balance** (`balance_start`)
The balance of a liquidity journal before a statement's first transaction, derived by anchoring on the previous statement and adding every posted transaction in between.

**Statement line**
The storage name of what the user interface calls a Bank Transaction.

**Structured payment reference**
A payment communication that carries its own check digits so the receiving bank can verify it. Seven schemes are recognised: Belgian, Danish, Finnish, Norwegian and Swedish, Slovenian, Dutch, and the international creditor reference.

**Suspense account** (`suspense_account_id`)
The account a Bank Transaction posts its unexplained side on, until reconciliation replaces it with the definitive accounts. It falls back to the company's suspense account when the journal names none.

**Suspense journal item**
The item of a Bank Transaction's entry sitting on the suspense account. At most one may exist; none exists once the transaction is fully allocated.

---

## T

**Trailing check-digit algorithm**
The algorithm that doubles every second digit from the right, reduces any result above nine by nine, sums everything and requires the total to be a multiple of ten. Used by the Danish and by the Norwegian-and-Swedish structured references.

**Transaction amount**
In a Bank Transaction's arithmetic, the amount expressed in the transaction currency, which is the foreign currency when the transaction has one and the journal currency otherwise.

**Transaction currency**
The currency a Bank Transaction's amount is expressed in from the counterparty's point of view: the transaction's foreign currency when it has one, otherwise the journal currency.

**Transaction details** (`transaction_details`)
The raw payload the bank or the import produced for a Bank Transaction, kept for audit and used as part of the text pool a reconciliation model's label condition is tested against.

**Transaction type** (`transaction_type`)
The type code the bank reported for a transaction, when the format carries one.

**Trust flag** (`allow_out_payment`, labelled "Send Money")
The flag that allows a Bank Account to be used for outgoing payments. It defaults to false, may only be changed by a bank-account validator, and locks the account number, the holder and the type while it is on.

---

## U

**Unreconciliation**
The deletion of matchings. It deletes the Partial Reconciliations, then the Full Reconciliations they belonged to, then reverses or deletes the derived entries, then recomputes the matching numbers, and finally pushes back to `in_process` the Payments that were `paid` only because of those matchings.

---

## V

**Validity**
The property of a Bank Statement whose starting balance equals the reported ending balance of the previous statement of the same journal. The first statement of a journal is always valid. An invalid statement produces a warning, never a refusal, and flags its journal.

---

## W

**Withholding line**
A journal item supplied by a capability package and added to a Payment's entry before the counterpart is computed. When withholding lines are present, any supplied write-off lines are dropped, because the withholding capability already passes its own lines through the write-off path.

**Write-off line**
A journal item added to a Payment's entry to absorb the difference between the amount paid and the amount the documents ask for. Its account, label, counterparty and currency come from the register-payment screen; its sign follows the direction.

---

## Terms deliberately not used

For clarity, this folder avoids three phrasings that appear in the storage names and could mislead.

| Storage name | Why the plain reading is misleading | What this folder says instead |
|---|---|---|
| `statement line` | It suggests the line always belongs to a statement, while the statement is optional | Bank Transaction |
| `journal_id` on a Bank Account | It is a collection, despite the singular name | the journals backed by the account, of which a constraint allows at most one |
| `amount_residual` on a Bank Transaction | It is not a journal item residual: it is the unexplained part of the transaction, and it ignores the journal items entirely while the transaction is unchecked | the transaction's residual, defined in `state-machines.md`, section 5 |
