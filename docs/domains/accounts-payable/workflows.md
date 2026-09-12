# Accounts Payable — Workflows

Each workflow below states who performs it, its preconditions, its numbered steps, what records are created or updated at each step, and its postconditions. Roles are named by the security group that gates them; the full matrix is in `configuration.md` §5.

Roles used here:

| Role | Group | What it may do in this domain |
|---|---|---|
| Billing clerk | *Invoicing* (`account.group_account_invoice`) | Create, edit, post, reset and cancel purchase documents; register payments |
| Accountant | *Accountant* (`account.group_account_user`) | Everything the billing clerk may do, plus review, mass posting, cheque printing and journal configuration |
| Accounting manager | *Accounting Manager* (`account.group_account_manager`) | Everything, plus settings, lock dates and the chart of accounts |
| Read-only accountant | *Read-only Accounting* (`account.group_account_readonly`) | See documents, warnings and totals |
| Any internal user | — | Post a message with an attachment on a document; that triggers decoding only for active internal users |

---

## 1. Capture a vendor bill by hand

*Performed by*: billing clerk. *Precondition*: a purchase journal exists in the company.

1. From **Accounting → Vendors → Bills → New**, or from the purchase journal card on the accounting dashboard (*Create invoice/bill*), a draft document is created with type `in_invoice`, the journal pre-selected, the accounting date set to today and the status `draft`. The number shows the placeholder `/`.
2. The clerk selects the **vendor**. This triggers, in one recomputation pass:
   - the commercial partner;
   - the delivery address (the vendor's *delivery* address);
   - the **fiscal position** — detected from the vendor and the delivery address;
   - the **payment term** — the vendor's *vendor payment terms* property, keeping the current value when the vendor has none;
   - the **recipient bank account** — by the ranking of `calculations.md` §3;
   - the salesperson is forced empty;
   - the **abnormal amount and date warnings** are evaluated;
   - the duplicate set is re-evaluated.
3. The clerk types the supplier's own number into **Vendor Reference**. The duplicate set is re-evaluated; see §7.
4. The clerk sets the **Bill Date**. The accounting date is recomputed from it by `calculations.md` §2.3, and, when the recomputation moves it, the line dates and the document number are scheduled for recomputation.
5. The clerk adds **lines**. For each line with a product, the label, unit, unit price, taxes and expense account default as in `calculations.md` §9. For a line without a product the account falls back to the most frequent account of this vendor, then to the neighbours' account, then to the journal's default account.
6. Instead of typing the lines, or in addition to them, the clerk may use the **Auto-Complete** picker (`invoice_vendor_bill_id`, labelled *Vendor Bill*), which offers earlier bills and vendor credit notes of the same company. It is a transient control: it stores nothing, and it is company-checked, so only documents of the current document's company are offered. Choosing one performs, in this order:
   1. **Appends** to the current document a copy of **every product, section, subsection and note line** of the chosen document. Nothing already present is replaced and nothing is cleared, so choosing a second document appends a second batch, and choosing the same document twice appends its lines twice.
   2. Overwrites the current document's **currency** with the chosen document's currency.
   3. Overwrites the current document's **fiscal position** with the chosen document's fiscal position.
   4. **Empties the picker again**, so that the control shows nothing and the choice leaves no trace on the document.

   Each copied line is made with the entity's ordinary copy values, so it carries the product, the label, the quantity, the unit, the unit price, the discount, the taxes, the deductibility, the analytic distribution and the imported marker of the line it came from. Two things are dropped: the balance of a product line, which is recomputed from the unit price, and the account and balance of a section, subsection or note line, which such a line may not carry. The links to a purchase order line and to a sales order line are **not** carried over, because the copy is made without the business-field marker.

   Steps 2 and 3 are ordinary writes on the document, so they re-trigger the currency rate recomputation, the fiscal position's tax and account substitution, and therefore the whole dynamic-line rebuild of step 7. **No other header field is taken over**: the vendor, the commercial partner, the bill date, the accounting date, the vendor reference, the payment reference, the payment term, the recipient bank account, the incoterm and the journal all keep the values they had, even when the chosen document has different ones.

   The picker has **no precondition and refuses nothing**. Choosing a vendor credit note copies that credit note's lines onto a bill unchanged, and choosing a document written in another currency simply changes the currency of the current draft.
7. At each save the **dynamic lines** are rebuilt in this order:
   1. the private-share base lines (when at least one product line has a deductibility below 100 %);
   2. the tax lines, including the non-deductible tax line;
   3. the discount-allocation line pairs (when the company names a *Vendor Bills Discounts Account* and a product line carries a discount) — `accounting-effects.md` §2.6;
   4. the cash rounding line;
   5. the early payment discount line pairs (when the payment term grants a discount computed in the `mixed` mode) — `accounting-effects.md` §2.7;
   6. the **payable term lines**, from the *needed terms* map of `calculations.md` §5.
8. The **Due Date** shows the latest maturity date produced. Overriding it by hand replaces the term lines with a single line on that date.
9. The clerk may attach the supplier's file. If an internal user posts it in the chatter, the decoding contract of §5 runs on it.
10. The clerk posts (§8) or leaves the document in the *To Check* queue (§10).

*Postcondition*: a draft purchase document with balanced dynamic lines, or a posted one.

---

## 2. Capture a vendor bill by uploading files

*Performed by*: billing clerk. *Precondition*: a purchase journal exists.

1. The user drags one or more files onto the purchase journal card of the accounting dashboard, or uses **Upload** in the bills list. The card shows the hint *Drop and let the AI process your bills automatically.*
2. The files are stored as attachments and the creation operation is called with their identifiers and, in the context, the default journal and the default document type.
3. The journal is resolved: the journal on which the operation was called; failing that the journal named in the context; failing that the first journal of the company whose type matches the default document type (*purchase* for any purchase type). Failures:
   - no journal and an unclassifiable document type → *The journal in which to upload the invoice is not specified. *
   - no attachment → *No attachment was provided*
   - no journal of the needed type → *No journal could be found in company «company name» for any of those types: «types»*
4. The attachments are converted to **file-data records**, and embedded files inside Portable Document Format containers are extracted and added.
5. The files are grouped by the **origin attachment** rule: an extracted file joins the group of the container it came from; every other file forms its own group. So dropping five separate files creates **five** bills.
6. One empty document is created per group, with the journal and the document type from the context and with the technical marker that suppresses the "manually modified" flag.
7. Each group's stored attachments are moved onto its document, and the document's chatter receives *This document was created from the following attachment(s).* with those attachments.
8. For each document in turn, the **decoding contract** of §5 runs on its group with the "newly created" flag set. A group whose decoding produced nothing usable leaves the message *There was an error while importing the bill, you can find attached the incoming XML*
9. For each created document, **automatic posting** is attempted (§8.2).
10. The caller is redirected: to the form of the single document when exactly one was created, otherwise to a list of them, under the title *Generated Documents*.

*Postcondition*: one draft (or automatically posted) bill per business document found in the upload.

---

## 3. Receive a vendor bill by electronic mail

*Performed by*: the supplier, by sending mail to the journal's address. *Precondition*: the purchase journal has an alias name and the company has an alias domain.

1. The purchase journal owns a mail alias whose target model is the journal entry and whose defaults are: company = the journal's company, document type = `in_invoice`, journal = the journal itself. The local part is derived by the rule of `configuration.md` §3.2.
2. A message arriving at that address is routed to the document model.
3. **Route check**: if the message has **no attachment at all**, the document is *not* created. Instead a bounce message is composed from the failure template, carrying the company's electronic mail address and name (falling back to the current company's), and sent back to the sender with a reply-to of that same address and a references header that marks it as a loop-detection bounce. The route is dropped. Failure inside this step: when the alias's default company value is not an integer, *Default value for 'company_id' for «record» is not an integer*.
4. Otherwise a new document is created with:
   - the technical marker that suppresses the "manually modified" flag;
   - number forced to `/`, so that the mail subject does not become the document number;
   - **source electronic mail address** = the first address in the sender header;
   - **partner** = resolved as in step 5.
5. **Sender resolution**:
   1. Split the sender header into addresses and look for partners with those addresses, keeping only partners that belong to no company, to this company, or that are shared; never creating one.
   2. If a partner was found and it is **internal** — that is, the company's own partner is the partner or its parent, or every user linked to it is internal — then the mail was probably forwarded by a colleague: search the **body** for electronic mail addresses instead and resolve those.
   3. Remove every internal partner from the result.
   4. Take the first remaining partner, or none.
6. The subject is injected at the top of the body as a heading, so that the chatter shows it.
7. The number is recomputed, because it was given explicitly and the document may be the first of its journal.
8. The message is posted on the document. The **after-post hook** then runs with the "from alias" marker set:
   1. The attachments are converted to file-data records and embedded files are extracted.
   2. Each file is classified as **valid** — it is of a type that stays visible on the document, or it parses as an extensible-markup-language tree — or **extra**.
   3. The valid files are grouped by the **mixed-types** rule of `entities.md` §9.6. This is the rule that turns five Portable Document Format files into five bills but one Portable Document Format file plus one image plus one structured file into a single bill.
   4. The first group stays on the document just created; **one extra document is created per further group**, by copying the first.
   5. The message is split accordingly: the original message keeps the first group's attachments plus every extra file; a copy of the message is created on each further document with that group's attachments.
   6. Each document's attachment set is fixed: valid files are attached, extra files are detached.
   7. Finally each document is extended with its group by the decoding contract of §5, with the "newly created" flag set.

*Postcondition*: one draft bill per business document in the mail, each carrying the sender's file(s) and a chatter trace.

---

## 4. Enhance an existing bill from an attachment

*Performed by*: any internal user posting a message with a file, or the system.

1. A message carrying attachments is posted on an existing document, by electronic mail or from the interface.
2. Nothing happens when: there are no attachments; the message is neither an electronic mail nor a comment (that is, it was produced by application code); or the context disables attachment import.
3. The files are converted and unwrapped as usual and classified valid / extra.
4. The document's attachment set is fixed: valid files attached, extra files detached.
5. **Decoding runs only when the posting user is active and internal.** A message from the supplier therefore attaches the file but never rewrites the bill.
6. The message's attachment set is rewritten to the full converted set.

---

## 5. The decoding contract

*Performed by*: the system, on behalf of the packages that implement concrete formats.

1. For each file-data record in the group, resolve its **decoder information** if not resolved yet: a structure holding the decoding operation and a numeric priority, or nothing.
2. Sort the files by *(has a decoder, priority)* descending and keep the **first** one only.
3. If it has no decoder, or a priority of zero, log *Attachment(s) «names» not imported: no suitable decoder found.* technically and stop; the document keeps whatever it had.
4. Otherwise run the decoder inside the roll-back-able guard: commit what precedes, run, commit again, and on any failure roll back.
5. Outcomes:
   - the decoder answers nothing → success; the document now carries the decoded partner, reference, dates, currency, lines and taxes; the lines it created are marked **imported**, so their unit price and taxes will never be silently recomputed;
   - the decoder answers a reason → the document receives *Attachment «file name» not imported: «reason»* and nothing was written;
   - the decoder raises a redirecting warning → it is re-raised so the user gets the offered navigation;
   - the decoder raises anything else → the transaction is rolled back and the document receives, with elevated privileges, the three-part message *Error importing attachment «descriptor»:* / *This specific error occurred during the import:* / the error text.

6. **Mark what the decoder added.** The document's invoice lines are compared with the set that existed before step 4. Every line in the difference receives the **imported** marker. That marker is what protects a decoded line from having its unit price and taxes silently recomputed, and what makes the archived-account refusal skip it at posting.
7. **Try to link the document to a purchase order — but only when the decoder was the first thing to put lines on it.** When at least one line was added *and* the document carried **no** invoice line before step 4, the purchase-order matching hook of §6 is called on the document, with the document's own type as the default type for anything it creates, and with a **four-second** budget instead of the ten seconds of the manual path. The attempt is guarded: a user error or a value error raised by the matching is written to the technical log as *Failed to link bill to purchase order*, together with the error, and is **not** propagated — the import still succeeds and the document keeps the lines the decoder wrote. Any other kind of failure is not caught here. When the document already had invoice lines before decoding, no linking is attempted at all.
8. **Notify the journal's subscribers — but only for a newly created document.** When the caller passed the "newly created" flag (that is, on the upload path of §2 and the mailbox path of §3, and not on the enhance-an-existing-document path of §4):
   1. The document's **portal access token** is generated if it has none, and written to storage immediately, before anything else in this step. The early write is deliberate: the decoding of a scanned document may finish asynchronously and update the same document at any moment, and a token written later could be lost to that concurrent update, which would leave the notification with an unusable link.
   2. The attachment set to send is assembled: **every attachment of the document**, plus the attachment record of **every file of the decoded group**, plus the attachment record of **every file unwrapped from that group** (the files embedded inside containers). Repetitions are removed, so a file that is both attached and in the group is sent once.
   3. A **copy** of each of those attachments is made for the mail, with the same media type and the same content, and with a name formed by the five characters `MAIL_` followed by the original name.
   4. The journal-subscriber notification of `interfaces.md` §9 is sent, carrying those copies. It goes out only if the journal actually carries at least one notification address; otherwise nothing is sent.
   5. **Any** failure of this whole step — assembling the attachments, copying them, or sending — is written to the technical log and swallowed. The import is never failed because a notification could not be sent.
9. **Run the post-processing hook.** The document is finally handed to a post-processing hook that runs **unconditionally**: whether or not linking was attempted in step 7, and whether or not an order was found. The base package's hook does nothing; it exists so that a companion package can react to a document that has just been decoded and possibly matched. It is listed among the named operations in `interfaces.md` §6.1.

*Note*: the base package ships **no** decoder. The structured-document packages and the scanning service supply them; see `../electronic-invoicing-and-document-exchange/`.

---

## 6. Match a bill to a purchase order

*Performed by*: the system, when the purchasing package is installed.

1. The origin field of the bill is split on commas and whitespace into a list of references.
2. The matching hook is called with: that list, the vendor identifier, the document total and a timeout. The timeout is **ten seconds** on this, the manual path. The **decoding path calls the very same hook itself**, at the close of the decoding contract (§5 step 7), and gives it only **four seconds**, because the decoding of an uploaded or mailed-in document must not keep the caller waiting; on that path the call is also made only when the decoder was the first thing to put lines on the document, and a user error or a value error it raises is logged and ignored. The base package's hook does nothing; the purchasing package implements it.
3. When the hook finds orders, it links them and completes the bill's lines from the ordered quantities and prices. See [`../purchasing/workflows.md`](../purchasing/workflows.md).
4. Whether or not an order was found, the decoding path then runs the post-processing hook of §5 step 9. The manual path does not.

The decoder of a structured document calls the same hook with the references it read and a flag saying the values came from a scan.

---

## 7. Duplicate detection in practice

*Performed by*: the system, continuously while the document is edited.

1. Whenever the vendor reference, the type, the partner, the bill date, the tax totals or the currency changes, the duplicate set is recomputed by the predicate of `calculations.md` §4.3, comparing against **draft and posted** documents of the same company.
2. The **exact duplicate** flag is recomputed by `calculations.md` §4.4.
3. The interface shows a red banner when the exact flag is set, and an amber banner when duplicates exist, the exact flag is not set and the document is still draft. Both read *This document might be a duplicate of* followed by a button naming the first duplicate with its total.
4. When at least one detected duplicate is still a draft, the banner offers *Delete duplicate* (singular) or *Delete all duplicates* (plural). Confirming deletes **every** detected duplicate of the current document.
5. In the bill upload list, the vendor reference cell is shaded red or amber accordingly.
6. Automatic posting is suppressed while duplicates are detected; the chatter records *Auto-post was disabled on this invoice because a potential duplicate was detected.*
7. Detection never blocks posting: a clerk who knows better may post anyway.

A worked case with its warning is in `calculations.md` §4.7.

---

## 8. Post a bill

### 8.1 Manual posting of a single bill

*Performed by*: billing clerk. *Precondition*: the document is draft and belongs to the invoicing group's reach.

1. The clerk presses **Confirm**.
2. When abnormal-invoice detection is explicitly enabled through the context and the document carries an abnormal amount or date warning, the Confirm Entries dialogue opens instead (see §9). By default that detection is **disabled** at this point so that automated flows are not blocked.
3. Otherwise the posting routine of `state-machines.md` §1.3 runs with hard scheduling.
4. After posting, the **autopost learning wizard** may open (§8.3).

### 8.2 Automatic posting of a bill

*Performed by*: the system, immediately after a bill is created from an upload, and by the scheduled job for scheduled entries.

A bill posts itself when **all** of:

1. the company's *Auto-validate bills* switch is on (default: on);
2. the bill has a vendor;
3. the document is a purchase document (receipts included);
4. the vendor's *Auto-post bills* policy is **Always**;
5. there is **no abnormal amount warning**;
6. the journal does **not** secure entries with a hash.

Then: if duplicates were detected, nothing is posted and the chatter records the suppression message; otherwise the bill is posted.

### 8.3 The automatic-posting learning wizard

*Performed by*: the system, after a successful posting.

The wizard opens only when **all** of:

1. exactly one document was posted;
2. it is now `posted`;
3. it is a purchase document (receipts included);
4. its journal does not secure entries with a hash;
5. **at least one of its lines was captured by an importer** (the imported marker);
6. it has a vendor;
7. the vendor's policy is exactly **Ask after 3 validations without edits**;
8. the company's automatic-validation switch is on;
9. the document has **not** been manually modified — the flag, the writes that set it and the five operations that suppress it are in `business-rules.md` §19, and a single programmatic write from another domain is enough to fail this condition.

Then the system counts the consecutive run of unmodified bills: starting at 1 for the current bill, it walks the **ten** most recent other posted purchase documents of the same vendor, newest first by creation date, and adds one for each that was not manually modified, stopping at the first that was. If the run is **fewer than three**, nothing opens.

Otherwise a dialogue opens saying:

> Hey there !
> It looks like you've successfully validated the last **«count»** bills for **«vendor»** without making any corrections.
> Want to make your life even easier and automate bill validation from this vendor ?
> Don't worry, you can always change this setting later on the vendor's form. You also have the option to disable the feature for all vendors in the accounting settings.

The count is displayed as the number itself when it is below 10, and as `10+` otherwise. Three choices:

| Button | Effect on the vendor's policy |
|---|---|
| Activate auto-validation | `always` |
| Ask me later | `ask` |
| Never for this vendor | `never` |

---

## 9. Post many bills at once

*Performed by*: accountant. *Precondition*: several documents are selected, or a journal is open.

1. The accountant chooses **Confirm Entries** from the list's action menu, or presses the confirm action on a journal.
2. The wizard collects the documents: every selected document in `draft` state when the caller is the document list; every draft document of the journal when the caller is a journal. Only documents that have lines are kept. Failures: *Missing 'active_model' in context.* when neither; *There are no journal items in the draft state to post.* when the set is empty.
3. The dialogue shows:
   - a heading saying the selected entries (or invoices) will be posted and that some may be future-dated or require hashing;
   - when at least one entry is dated after today, the *Force* switch labelled *Confirm them now*, with the note that future-dated entries will otherwise auto-confirm on their own dates;
   - when at least one journal secures entries with a hash, the *Force Hash* switch labelled *Yes, confirm and secure them now.*;
   - when at least one document carries an abnormal-date warning, the list of affected vendors and the *Ignore future alerts* switch;
   - when at least one document carries an abnormal-amount warning, the same for amounts.
4. On **Confirm**:
   1. If *ignore abnormal amount* was ticked, every listed vendor gets its amount warning permanently silenced.
   2. If *ignore abnormal date* was ticked, the same for dates.
   3. If *Force* was ticked, every selected document's automatic posting mode is set to `no`.
   4. The set to post is every selected document, or, when *Force Hash* was not ticked, only those whose journal does not secure entries with a hash.
   5. That set is posted with **soft** scheduling when *Force* was not ticked, and hard scheduling when it was. Soft scheduling means future-dated documents are scheduled rather than posted.
   6. If exactly one of them qualifies, the autopost learning wizard opens; otherwise the dialogue closes.

### 9.1 The confirmation variant driven from the document list

A second entry point exists: *Confirm Entries* as a list action. It splits the selection itself:

1. Keep only draft documents that have lines; otherwise *There are no journal items in the draft state to post.*
2. Split off the documents **requiring confirmation**: those whose accounting date (or, failing that, bill date) is later than today, and those whose journal secures entries with a hash.
3. Post the rest immediately with hard scheduling.
4. Open the wizard on the remainder only.

---

## 10. Review a bill

*Performed by*: accountant.

1. Draft and unreviewed posted documents appear under the **To Review** filter — on documents, the filter is *reviewed is false and status is not draft*; on journal items, *the document's reviewed flag is false and the parent status is not draft*.
2. The purchase journal card on the dashboard shows the count and the summed amount of documents to review.
3. The accountant opens the document and presses **Review** (on the list line) or **Set as Reviewed** (on the form). Only posted documents are affected.
4. The flag may be cleared again by an authorised user. Attempting either without the review authority raises *You don't have the access rights to perform this action.*
5. Resetting a reviewed document to draft is refused for an unauthorised user: *Validated entries can only be changed by your accountant.*

---

## 11. Reset a bill to draft, and cancel it

*Performed by*: billing clerk (reset), accountant (cancel).

**Reset to draft.** Preconditions and side effects are in `state-machines.md` §1.4. In short: the document must be posted or cancelled; it must not require a cancellation request; it must not be hashed, an exchange-difference entry or a cash-basis entry. The button is hidden altogether when the document is hashed, when it is restricted by the hash rules, or when it needs a cancellation request.

**Cancel.** The document is first reset to draft (with those guards), then every reconciliation is removed, every payment whose entry this is becomes cancelled, and the status becomes `cancel`. The number is kept so that no numbering gap appears.

**Request cancellation.** When a localisation marks a document as needing a cancellation request, the reset button is replaced by *Request Cancellation*. The base implementation of that operation refuses: *You can only request a cancellation for invoice sent to the government.*

**Delete.** A draft document may be deleted outright when it satisfies the deletability rules of `state-machines.md` §1.6. Bulk removal classifies each document into reverse / cancel / unlink.

---

## 12. Reverse a bill

*Performed by*: accountant. *Precondition*: the document is posted.

1. From the document, choose **Reverse** (the action is titled *Credit Note* when the document is invoice-like).
2. The dialogue shows: the reversal date (default today), a reason, and the journal (defaulting to the first active journal among the selected documents' journals, restricted to journals of the same type as theirs). It also shows the residual and currency when exactly one document is selected.
3. Two buttons:
   - **Reverse** — creates one independent vendor credit note per bill;
   - **Reverse and Modify** — creates the cancelling reversal **and** a draft replacement bill copying only the product, section, subsection and note lines.
4. The defaults applied to each reversal are listed in `accounting-effects.md` §5.2.
5. Documents are split into two batches: those to be **cancelled by their reversal** (in *modify* mode, or when the document is a miscellaneous entry — and only when the reversal is not future-dated) and the others. The first batch is reversed with cancellation, the second without.
6. Each original receives the chatter line *This entry has been reversed* with a link.
7. The user is redirected to the reversal, or to the list of reversals.

A worked example including a **partly paid** bill is in `accounting-effects.md` §5.6.

---

## 13. Raise a debit note

*Performed by*: accountant. *Precondition*: the source documents are posted, are of type `out_invoice`, `in_invoice`, `out_refund` or `in_refund`, and none already has a debit note.

1. From the document list or form, choose **Add Debit Note**. Refusals at this point: *You can only debit posted moves.*; *You can't make a debit note for an invoice that is already linked to a debit note.*; *You can make a debit note only for a Customer Invoice, a Customer Credit Note, a Vendor Bill or a Vendor Credit Note.*
2. The dialogue asks for the debit note date (default today), a reason, an optional journal and the *Copy Lines* switch. The switch is **hidden** when the sources share the type `in_refund` or `out_refund`, so a debit note raised from a credit note through the dialogue leaves it at its default of off; what happens when a caller sets it anyway is the compatibility finding in `business-rules.md` §11.
3. On confirmation, each source is copied with the overrides of `accounting-effects.md` §9. The copy carries the business links of the source (purchase order links and the like).
4. The user is redirected to the debit note, or to the list of them, with the resulting type as the default for further creation.
5. The source document shows a *Debit Notes* smart button counting them; it is hidden while the count is zero (`interfaces.md` §2.1).
6. The chatter of the **new** document records *This debit note was created from: «link»*, the link naming the source. The source's own chatter records nothing.

---

## 14. Pay a bill by printed cheque

*Performed by*: accountant. *Precondition*: the bank journal offers the cheque payment method and a cheque layout is configured.

### 14.1 Configure once

1. On the bank journal, under the outgoing payment settings, the **Check Printing** group appears as soon as the cheque method is among the journal's outgoing methods.
2. Choose **Manual Numbering** when the stationery is blank; leave it off when the stationery is pre-printed with numbers.
3. When manual numbering is on, the **Next Check Number** is editable; writing it moves the journal's cheque sequence and sets its padding (see `calculations.md` §13.1).
4. Optionally choose a per-journal **Check Layout** overriding the company's.
5. On the company (Accounting settings) choose the **Check Layout**, whether to print the date caption, whether the stub may spill onto further pages, and the three margins.

### 14.2 Issue the cheque

1. Select the posted bill(s) and press **Register Payment**, or create an outgoing payment directly. Refusals: *You can only register payment for posted journal entries.*; *You cannot register payments for miscellaneous entries.*; *You cannot register payments for blocked invoices.*
2. Choose the bank journal and the payment method **Checks**.
3. While the payment is a draft and the journal uses manual numbering, the **Check Number** field previews the next number from the journal's cheque sequence.
4. On posting, the number is actually drawn from the sequence and written on the payment; the payment's journal items are labelled *Checks - «number»* (plus *: «memo»* when a memo exists); the payment status becomes `in_process`; the amount in words is computed.

### 14.3 Print

1. The payment is now in the **Checks to Print** filter and in the bank journal card's *Checks to print* counter.
2. Press **Print Check** on the payment, or select several and run the *Print Checks* action from the list or kanban.
3. Validations: at least one selected payment must use the cheque method and not yet be sent, otherwise *Payments to print as a checks must have 'Check' selected as payment method and not have already been reconciled*; all of them must belong to one bank journal, otherwise *In order to print multiple checks at once, they must belong to the same bank journal.*
4. **Manual numbering path**: drafts are posted, then the printable document is produced and the payments are marked as sent.
5. **Pre-printed path**: the pre-numbered dialogue opens, pre-filled with the highest existing cheque number of the journal plus one, padded to the same width. Its text reads *Please enter the number of the first pre-printed check that you are about to print on.* and *This will allow to save on payments the number of the corresponding check.* On **Print**: drafts are posted; unsent in-process payments are marked sent; the payments are renumbered in order from the entered number; the printable document is produced and the dialogue closes when the download completes. A non-numeric entry is refused: *Next Check Number should only contains numbers.*
6. Layout resolution: the journal's layout, falling back to the company's. Failures are the two redirecting warnings of `state-machines.md` §5.3.
7. The printed pages are composed as in `calculations.md` §14: page one carries the negotiable cheque, later pages carry `VOID` in place of the amount and of the amount in words.

### 14.4 Reprint, unmark, void

| Need | Operation | Effect |
|---|---|---|
| The printer jammed and nothing usable came out | **Unmark Sent** | The sent flag is cleared and the payment re-enters the print queue with the **same** number |
| The cheque was spoiled and must not be cashed | **Void Check** | The payment is reset to draft and then cancelled: reconciliations removed, the draft entry deleted, status `canceled`. The number stays on the record for audit |
| The whole run was misaligned | Re-run **Print Checks** after unmarking, and on the pre-printed path enter the correct first number | The payments are renumbered from that value |

A worked example of a cheque for 1 234.56 with its amount in words is in `calculations.md` §7.4, and its journal entry in `accounting-effects.md` §7.

---

## 15. Handle a purchase receipt

*Performed by*: billing clerk.

A purchase receipt is captured exactly like a bill (§1), with two differences:

1. its **fiscal position** is forced to the company's *Default Purchase Receipt Fiscal Position* when one is configured, regardless of the vendor;
2. it is never flagged as an **exact** duplicate (it can still be a probable one).

It is posted, reversed and paid exactly like a bill; its reversal is a vendor credit note.

---

## 16. Block and unblock payment of a bill

*Performed by*: accountant.

1. On a posted bill, toggle the payment block. Refused on a document already paid or in payment: *You can't block a paid invoice.*
2. The payment status becomes `blocked` and is never recomputed while it stays so, which keeps the bill out of payment proposals.
3. Toggling again sets it to `not_paid` and schedules a recomputation, so the true status returns.
4. Registering a payment on a blocked document is refused: *You cannot register payments for blocked invoices.*

---

## 17. Quick encoding of bills

*Performed by*: billing clerk in a bookkeeping practice. *Precondition*: the company's *Quick encoding* setting is `in_invoices` or `out_and_in_invoices`.

1. The bill form shows a **Total (Tax inc.)** field.
2. When the bill date is empty, the system suggests one: today, or the bill date of the most recent posted document of the same journal and company passed through the accounting-date algorithm.
3. The clerk selects the vendor and types the total. One line is generated from the most frequent account and taxes of that vendor (or the journal defaults), with the unit price back-computed from the total (`calculations.md` §11).
4. A rounding reconciliation nudges the first tax group so that the document total equals the typed total exactly.
5. Posting refuses a surviving mismatch: *The current total is «current» but the expected total is «expected». In order to post the invoice/bill, you can adjust its lines or the expected Total (tax inc.).*

---

## 18. Analyse purchases

*Performed by*: read-only accountant and above.

1. Open **Accounting → Reporting → Management → Bills Analysis** (or the analogous vendor analysis entry).
2. The list, pivot and graph read the **invoice analysis report** view: one row per product line, quantities restated in the product's reference unit, purchase amounts **negative**, company amounts converted to the active company's currency at today's rate.
3. Grouping by product, vendor, category, account, period, fiscal position or payment status is available; the **average price** measure is aggregated as a weighted average rather than as a mean of means.
4. Drilling into a row opens its journal entry.

Every column and its computation is in `entities.md` §3 and `calculations.md` §15.

---

## 19. Settle a bill paid by another company of the group

*Performed by*: the system. *Precondition*: the Intercompany Payment Clearing package is installed, both companies have a clearing journal, and the relevant intercompany accounts are set.

1. Company **B** pays a payment transaction attached to a document of company **A**.
2. When the document of **A** is posted, and the qualification conditions of `accounting-effects.md` §8.1 hold, the clearing runs.
3. For each settling payment in the `authorized` or `done` state, a clearing entry is created in **B**'s clearing journal and reconciled against the payment's own counterpart lines.
4. One settlement entry is created in **A**'s clearing journal and reconciled against the document's term lines.
5. Both companies end with mirrored intercompany balances; see `accounting-effects.md` §8.4.

---

## 20. Try the sample bill

*Performed by*: a new user, from the dashboard, when the demonstration data is installed.

1. The purchase journal card offers *try our sample* when no document exists yet.
2. The operation locates a purchase journal (from the context or the first one of the company) and a demonstration partner. Refusals: *No journal could be found in company «company» for any of those types: purchase*; *You may only use samples in demo mode, try uploading one of your invoices instead.*
3. A bill is created with: type `in_invoice`; that partner; a vendor reference of the form `DE` followed by the year and month of a date twelve days ago; that bill date; a due date thirty days later; the purchase journal; and two lines — five of *[FURN_8999] Three-Seat Sofa* at 1 500 and five of *[FURN_8220] Four Person Desk* at 2 350 — both on the journal's default account, falling back to the company's expense account.
4. Outside test execution a printable file is generated and attached so that the user sees a realistic scanned bill; inside tests only an empty message is posted.
