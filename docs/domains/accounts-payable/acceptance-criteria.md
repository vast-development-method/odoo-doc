# Accounts Payable — Acceptance Criteria

Every scenario is written in Given / When / Then form with concrete numbers. Unless a scenario says otherwise, the shared setting is:

| Element | Value |
|---|---|
| Company | Northwind; company currency **Euro**, rounding factor 0.01, fiscal year ending 31 December |
| Purchase journal | code `BILL`, default expense account **600000 Expenses**, dedicated credit-note sequence on, dedicated debit-note sequence on, no hash |
| Bank journal | code `BNK1`, outstanding payments account **101402 Outstanding Payments**, cheque method available |
| Vendor | *Papeterie Lambert*, payable account **400000 Account Payable**, automatic posting policy `ask` |
| Tax | *Purchase 15 %*, tax-excluded, distributing 100 % to **131000 Tax Paid** on both the invoice and the refund distribution |
| Product A | *Office chair*, expense account 600000, supplier tax Purchase 15 %, purchase price 800.00 per unit, reference unit *Units* |
| Product B | *Desk lamp*, expense account **600100 Expenses (lamps)**, supplier taxes Purchase 15 % and **Purchase 15 % (b)** (a copy distributing to the same account), purchase price 160.00 per dozen, reference unit *Dozens* |
| Today | 11 September 2026, unless the scenario freezes another date |

---

## Group A — Capturing a bill

### A1. A bill of two expense lines with a purchase tax

**Given** a draft vendor bill in journal `BILL` with vendor *Papeterie Lambert*, vendor reference `LAM-9912`, bill date 1 January 2026 and payment term *Immediate* (one line, percent 100, zero days after the bill date),
**and** line 1 is one unit of *Office chair* and line 2 is one dozen of *Desk lamp*,
**When** the document is saved,
**Then** line 1 shows label *Office chair*, unit *Units*, quantity 1, unit price 800.00, taxes {Purchase 15 %}, account 600000, subtotal 800.00 and total 920.00;
**and** line 2 shows label *Desk lamp*, unit *Dozens*, quantity 1, unit price 160.00, taxes {Purchase 15 %, Purchase 15 % (b)}, account 600100, subtotal 160.00 and total 208.00;
**and** the document totals are untaxed 960.00, tax 168.00, total 1 128.00;
**and** exactly two tax lines exist: *Purchase 15 %* for 144.00 on account 131000 with a tax base of 960.00, and *Purchase 15 % (b)* for 24.00 on account 131000 with a tax base of 160.00;
**and** exactly one payable term line exists: account 400000, maturity 1 January 2026, credit 1 128.00, amount in currency −1 128.00, label `LAM-9912`;
**and** the document's due date is 1 January 2026.

**When** the bill is posted,
**Then** its status is `posted`, its number is `BILL/2026/01/0001`, its payment status is `not_paid`, its reviewed flag is true;
**and** the journal items are exactly:

| Account | Debit | Credit |
|---|---|---|
| 600000 Expenses | 800.00 | — |
| 600100 Expenses (lamps) | 160.00 | — |
| 131000 Tax Paid (Purchase 15 %) | 144.00 | — |
| 131000 Tax Paid (Purchase 15 % (b)) | 24.00 | — |
| 400000 Account Payable | — | 1 128.00 |

**and** the signed totals are untaxed −960.00, tax −168.00, total −1 128.00, residual 1 128.00;
**and** the vendor's supplier rank increased by 1.

### A2. Posting refuses a bill without a bill date

**Given** the bill of A1 with the bill date cleared,
**When** posting is attempted,
**Then** it is refused with *The Bill/Refund date is required to validate this document.*

### A3. Posting refuses a bill without a vendor

**Given** the bill of A1 with the vendor cleared,
**When** posting is attempted,
**Then** it is refused with *The field 'Vendor' is required, please complete it to validate the Vendor Bill.*

### A4. Posting refuses a negative total

**Given** a draft bill whose only line is one unit at −50.00,
**When** posting is attempted,
**Then** it is refused with *You cannot validate an invoice with a negative total amount. You should create a credit note instead. Use the action menu to transform it into a credit note or refund.*

### A5. Posting refuses an empty document

**Given** a draft bill whose only line is a note,
**When** posting is attempted,
**Then** it is refused with *Even magicians can't post nothing!*

### A6. A purchase document cannot live in a sale journal

**Given** a draft bill,
**When** its journal is changed to a journal of type `sale`,
**Then** the write is refused with *Cannot create a purchase document in a non purchase journal*

### A7. A receivable account cannot be used on a bill

**Given** the posted bill of A1 reset to draft,
**When** line 1's account is changed to a receivable account coded `121000`,
**Then** the write is refused with *Account 121000 is of receivable type, but is used in a purchase operation.*

### A8. A payable account must carry a maturity date

**Given** a draft bill,
**When** a product line is given the payable account 400000,
**Then** the write is refused with *Any journal item on a payable account must have a due date and vice versa.*

---

## Group B — Dates

### B1. A late bill lands in its own month

**Given** today is 11 September 2026, the purchase journal's highest number is `BILL/2026/08/0007` (a monthly series),
**When** a bill is captured with bill date 20 August 2026,
**Then** its accounting date becomes **31 August 2026**
**and** on posting its number is `BILL/2026/08/0008`.

### B2. A bill of the current month keeps today

**Given** the same journal, today 11 September 2026,
**When** a bill is captured with bill date 2 September 2026,
**Then** its accounting date becomes **11 September 2026** (the maximum of the bill date and today).

### B3. A future-dated bill keeps its own date

**Given** the same journal, today 11 September 2026,
**When** a bill is captured with bill date 25 September 2026,
**Then** its accounting date becomes **25 September 2026** (again the maximum of the two).

### B4. A locked period pushes the accounting date forward

**Given** a purchase lock date of 31 July 2026 and today 11 September 2026,
**When** a bill is captured with bill date 15 July 2026,
**Then** the candidate date becomes 1 August 2026 (the violated lock date plus one day);
**and** because September is later than August, the accounting date becomes **31 August 2026**;
**and** while the document is draft the banner reads *The date is being set prior to: «the formatted lock dates». The Journal Entry will be accounted on 31 August 2026 upon posting.*

### B5. A yearly series pushes to 31 December

**Given** a purchase journal whose numbering resets **yearly** and whose highest number is `BILL/2025/00012`, today 11 September 2026,
**When** a bill is captured with bill date 3 November 2025,
**Then** its accounting date becomes **31 December 2025**.

---

## Group C — Payment terms

### C1. Sixty days from end of month

**Given** the vendor payment term *60 days end of month* — one line, percent 100, delay type *Days after end of month*, 60 days,
**and** a bill dated **14 April 2026** of untaxed 3 000.00 and tax 600.00, total 3 600.00 Euro,
**When** the document is saved,
**Then** the last day of the bill's month is 30 April 2026;
**and** the due date of the single instalment is 30 April 2026 plus 60 days = **29 June 2026**;
**and** exactly one payable term line exists, credit 3 600.00, amount in currency −3 600.00, maturity 29 June 2026;
**and** the document's due date is 29 June 2026.

### C2. Two instalments, thirty per cent then the balance

**Given** the term: line 1 percent 30, *Days after end of month*, 0 days; line 2 percent 70, *Days after end of month*, 60 days,
**and** the same bill of 3 600.00 dated 14 April 2026,
**When** the document is saved,
**Then** line 1 produces an instalment of `round_to(Euro, 3 600.00 × 30 ÷ 100) = 1 080.00` due **30 April 2026**;
**and** line 2, being the last line, takes the residual 2 520.00 due **29 June 2026**;
**and** the two payable term lines are labelled `«vendor reference» installment #1` and `«vendor reference» installment #2`, ordered by maturity date;
**and** the document's due date is 29 June 2026.

### C3. The last line absorbs the rounding drift

**Given** a term of three percent lines 33.33 / 33.33 / 33.34 with delays 0, 30 and 60 days after the bill date,
**and** a bill of 100.00 Euro dated 1 March 2026,
**When** the document is saved,
**Then** the instalments are 33.33 due 1 March, 33.33 due 31 March and **33.34** due 30 April;
**and** they sum to exactly 100.00.

### C4. A fixed line followed by a balance line

**Given** a term: line 1 *fixed* 500.00, *Days after invoice date* 0; line 2 *percent* 100, *Days after invoice date* 30,
**and** a bill of 1 200.00 Euro dated 1 March 2026, document currency = company currency so the rate is 1,
**When** the document is saved,
**Then** line 1 produces 500.00 due 1 March 2026;
**and** line 2, being last, produces the residual **700.00** due 31 March 2026 — its declared 100 % is ignored because the last line is always the balance.

### C5. A term whose percentages do not sum to 100 is rejected

**Given** a new payment term with two percent lines of 50 and 40,
**When** it is saved,
**Then** it is refused with *The Payment Term must have at least one percent line and the sum of the percent must be 100%.*

### C6. The fourth delay type

**Given** a term: one line, percent 100, delay type *Days end of month on the*, 10 days, days-on-next-month 15,
**and** a bill dated 25 April 2026,
**When** the document is saved,
**Then** 25 April plus 10 days is 5 May 2026; plus one month is 5 June 2026; the day forced to 15 gives **15 June 2026**.

### C7. Early payment discount, mode *On early payment*

**Given** a term: one line percent 100, 30 days after the bill date; early discount on, 2 %, 10 days; mode `included`,
**and** a bill dated 1 June 2026 of untaxed 1 000.00 and tax 210.00, total 1 210.00,
**When** the document is saved,
**Then** the payable term line is 1 210.00 due 1 July 2026, with discount date **11 June 2026** and discounted amount `round_to(Euro, 1 210.00 × 0.98) = 1 185.80`.

### C8. Early payment discount, mode *Never*

**Given** the same term with mode `excluded`,
**Then** the discounted amount is `round_to(Euro, 1 210.00 − 1 000.00 × 0.02) = 1 190.00`.

### C9. The early discount refuses a multi-line term

**Given** a term with two lines,
**When** the early discount is switched on,
**Then** it is refused with *The Early Payment Discount functionality can only be used with payment terms using a single 100% line. *

---

## Group D — Duplicate detection

### D1. An exact duplicate, red warning

**Given** bill A: type `in_invoice`, partner *Papeterie Lambert*, vendor reference `FA-2026-0412`, bill date 2 March 2026, currency Euro, total 1 452.00, **posted**,
**When** bill B is captured with exactly the same partner, reference, bill date, currency and total, and stays draft,
**Then** B's duplicate set is {A};
**and** B's exact-duplicate flag is **true**;
**and** B shows the **red** banner *This document might be a duplicate of* naming A with its total;
**and** because A is posted, B's draft-duplicate flag is false, so the delete action is hidden;
**and** if B would otherwise be posted automatically, it is not, and its chatter records *Auto-post was disabled on this invoice because a potential duplicate was detected.*

### D2. A probable duplicate, amber warning

**Given** bill A as in D1,
**When** bill B is captured with reference `FA 2026 0412` (different text), same partner, same bill date 2 March 2026, same total 1 452.00, same currency,
**Then** case 1 of the matching predicate fails and case 2 succeeds, so A is still detected;
**and** the exact-duplicate flag is **false** because the references differ;
**and** B shows the **amber** banner while it is still draft.

### D3. A different calendar year is not a duplicate

**Given** bill A as in D1,
**When** bill B is captured with the same reference `FA-2026-0412` but bill date 2 March **2025**,
**Then** case 1 fails (different years) and case 2 fails (different bill dates);
**and** B's duplicate set is empty and no banner appears.

### D4. A different currency is not a duplicate

**Given** bill A as in D1,
**When** bill B is identical but written in United States dollars,
**Then** the currency condition fails and no duplicate is detected.

### D5. A cancelled document is not a duplicate

**Given** bill A as in D1 but **cancelled**,
**When** bill B is captured identically,
**Then** no duplicate is detected, because only draft and posted documents are matched.

### D6. Three copies detect each other

**Given** three bills of the same vendor, the same bill date and the same reference `X-1`, all draft,
**When** the reference is (re)assigned on all three,
**Then** each one's duplicate set contains exactly the **other two**.

### D7. Deleting the duplicates

**Given** bill B whose duplicate set is {A} and A is still draft,
**When** the delete action is used on B,
**Then** A is deleted and B's duplicate set becomes empty.

### D8. A document being typed is compared before it is saved

**Given** bill A posted with reference `R-7`, partner *Papeterie Lambert*, bill date 1 February 2026, total 960.00,
**When** a new bill form is filled in with the same partner, the same bill date and the same reference, with lines that make the total 960.00, and the form has **not** been saved,
**Then** the duplicate set already contains A, because the comparison injects the in-memory values and reads the total from the live tax totals.

---

## Group E — The reviewed flag and mass posting

### E1. Posting marks a bill reviewed

**Given** a draft bill,
**When** it is posted,
**Then** its reviewed flag becomes true (the journal is a purchase journal and the review predicate answers true in the base package).

### E2. The to-review queue

**Given** a posted bill whose reviewed flag was cleared by an accountant,
**When** the *To Review* filter is applied,
**Then** the bill appears, because the filter is *reviewed is false and the status is not draft*;
**and** the purchase journal card shows a review count of 1 and the summed amount.

### E3. Mass posting schedules future-dated documents

**Given** three draft bills dated 1 September 2026, 11 September 2026 and 30 September 2026, today being 11 September 2026,
**When** they are selected and confirmed **without** ticking *Force*,
**Then** the first two are posted;
**and** the third stays draft, its automatic posting mode becomes *At Date*, and its chatter records *This move will be posted at the accounting date: 30 September 2026*.

### E4. Mass posting with Force

**Given** the same three bills,
**When** they are confirmed **with** *Force* ticked,
**Then** all three are posted immediately and each one's automatic posting mode is set to `no` beforehand.

### E5. Mass posting skips hashing journals

**Given** two draft bills, one in a journal that secures entries with a hash,
**When** they are confirmed **without** ticking *Force Hash*,
**Then** only the bill of the non-hashing journal is posted.

### E6. Mass posting with nothing to do

**Given** a selection containing only posted documents,
**When** the confirm dialogue is opened,
**Then** it refuses with *There are no journal items in the draft state to post.*

### E7. Silencing abnormal warnings from the dialogue

**Given** two draft bills of *Papeterie Lambert* carrying an abnormal-amount warning,
**When** they are confirmed with *Ignore future alerts* ticked in the amount block,
**Then** the vendor's *ignore abnormal invoice amount* flag becomes true for the company;
**and** later bills of that vendor no longer produce an amount warning.

---

## Group F — Automatic posting

### F1. A vendor set to Always is posted automatically

**Given** the company's *Auto-validate bills* switch on, the vendor's policy `always`, a purchase journal without hashing,
**When** a bill of that vendor is created by uploading a file, with no abnormal-amount warning and no duplicate,
**Then** it is posted immediately after creation.

### F2. A duplicate blocks automatic posting

**Given** the same setting, but a duplicate is detected,
**Then** the bill stays draft and its chatter records *Auto-post was disabled on this invoice because a potential duplicate was detected.*

### F3. An abnormal amount blocks automatic posting

**Given** the same setting, but the bill carries an abnormal-amount warning,
**Then** the bill stays draft and nothing is written to the chatter.

### F4. The learning wizard opens after three untouched bills

**Given** the vendor's policy `ask`, the company switch on, no hashing,
**and** the two most recently created posted purchase documents of that vendor were **not** manually modified,
**When** a third bill of that vendor, carrying at least one imported line and never manually modified, is posted,
**Then** the run count is 3 and the dialogue opens reading *It looks like you've successfully validated the last 3 bills for Papeterie Lambert without making any corrections.*
**and** choosing *Activate auto-validation* sets the vendor's policy to `always`.

### F5. The learning wizard does not open below the threshold

**Given** the same setting but the immediately preceding posted bill of that vendor **was** manually modified,
**Then** the run count is 1 and no dialogue opens.

### F6. The learning wizard does not open for a manually typed bill

**Given** a bill of a vendor with policy `ask` that has **no** imported line,
**When** it is posted,
**Then** no dialogue opens, because at least one line must carry the imported marker.

### F7. The scheduled job posts due scheduled bills

**Given** fifty draft bills whose automatic posting mode is *At Date* and whose accounting dates are on or before today,
**When** the daily job runs,
**Then** all fifty are posted in one batch;
**and** if any one of them raises a user error, the batch is rolled back and each is posted individually, so the forty-nine valid ones still post.

---

## Group G — Reversal and refund

### G1. A refund of a partly paid bill

**Given** the bill of A1, posted as `BILL/2026/01/0003` for 1 128.00 with a single payable term line due 1 January 2026,
**and** an outgoing payment of 400.00 Euro dated 20 January 2026 reconciled against it,
**Then** the bill's residual is 728.00 and its payment status is `partial`.

**When** the user reverses it with the plain *Reverse* route, reversal date 5 February 2026, reason *Lamps returned*, and deletes the chair line from the resulting draft,
**Then** the credit note is of type `in_refund`, numbered `RBILL/2026/02/0001`, referencing the bill, with vendor reference *Reversal of: BILL/2026/01/0003, Lamps returned*, bill date, accounting date and due date 5 February 2026, and **no** payment term;
**and** its journal items are 160.00 credit on 600100, 24.00 credit and 24.00 credit on 131000 (both from the **refund** distribution), and 208.00 debit on 400000;
**and** its totals are untaxed 160.00, tax 48.00, total 208.00, signed total +208.00.

**When** the credit note's payable line is reconciled against the bill's payable line,
**Then** the bill's residual becomes 1 128.00 − 400.00 − 208.00 = **520.00** and its payment status stays `partial`;
**and** the credit note's residual becomes 0 and its payment status becomes **`paid`** — not `reversed`, because the counterpart type set is `{in_invoice}`.

**When** a second outgoing payment of 520.00 is registered and reconciled on 28 February 2026,
**Then** the bill's residual is 0 and its payment status becomes `paid` (or `in_payment` where an accounting package enables that value, until the bank statements are reconciled).

### G2. A cancelling reversal of a partly paid bill unwinds the payment

**Given** the same bill, partly paid by 400.00,
**When** the user chooses *Reverse and Modify* on the reversal dialogue,
**Then** every reconciliation of the bill's lines is removed first, so the 400.00 payment becomes unreconciled and its own status falls back to `in_process`;
**and** a credit note of the **full** 1 128.00 is created and posted on the reversal date;
**and** the two payable term lines are reconciled, clearing both;
**and** the bill's payment status becomes **`reversed`**, because its only counterpart type is `in_refund`;
**and** a draft replacement bill is created carrying only the product, section, subsection and note lines of the original, with the original's main attachment copied onto it.

### G3. A full reversal of an unpaid bill

**Given** the posted, unpaid bill of A1,
**When** it is reversed with cancellation,
**Then** the credit note mirrors every line with every side reversed;
**and** the bill's payment status becomes `reversed` and the credit note's becomes `paid`.

### G4. The reversal journal must match

**Given** a posted bill in journal `BILL`,
**When** a reversal is attempted into a journal of type `sale`,
**Then** it is refused with *Journal should be the same type as the reversed entry.*

### G5. A future-dated reversal is scheduled

**Given** a posted bill and a reversal date of 31 December 2026, today 11 September 2026,
**When** the reversal is created,
**Then** its automatic posting mode is `at_date` and it is **not** reconciled with the original even when the cancelling route was chosen (the cancelling batch only takes non-future reversals).

### G6. The payment term is cleared except in the mixed discount mode

**Given** a posted bill whose payment term grants an early discount computed in the `mixed` mode,
**When** it is reversed,
**Then** the credit note **keeps** that payment term;
**and** for any other term the credit note has none.

---

## Group H — Cheque printing

### H1. Amount in words for one thousand two hundred thirty-four point five six

**Given** an outgoing payment of **1 234.56** United States dollars with the *Checks* method, the user's language English, the currency's decimal places 2, unit label *Dollars*, subunit label *Cents*,
**When** the amount in words is computed,
**Then** the formatted amount is `1234.56`, the integral part is 1234 and the fractional part is 56;
**and** `1 234.56 − 1 234 = 0.56` is not zero in that currency, so the two-part form applies;
**and** the words are **One Thousand, Two Hundred And Thirty-Four Dollars and Fifty-Six Cents** — 69 characters;
**and** the printed line is those 69 characters, then one space, then **130 asterisks**, for exactly 200 characters;
**and** the numeric amount printed beside it is `$ 1,234.56`.

### H2. Amount in words with no cents

**Given** the same payment for 1 234.00 dollars,
**Then** the words are **One Thousand, Two Hundred And Thirty-Four Dollars** (the single-part form).

### H3. The document-level words strip commas

**Given** a document whose total is 1 234.56 dollars,
**Then** its *amount total in words* reads *One Thousand Two Hundred And Thirty-Four Dollars and Fifty-Six Cents* — the comma after *Thousand* removed.

### H4. Manual numbering consumes the sequence at posting

**Given** bank journal `BNK1` with manual numbering on and next cheque number `00042`,
**When** a cheque payment is created and posted,
**Then** its cheque number is `00042`;
**and** its journal items are labelled *Checks - 00042*;
**and** the journal's next cheque number becomes `00043`.

### H5. Ten bills paid by one grouped cheque

**Given** ten posted vendor bills of 100.00 each and bank journal `BNK1` with manual numbering on and next number `00042`,
**When** one grouped cheque payment is registered for all ten,
**Then** the payment's amount is 1 000.00, its cheque number is `00042` and its amount in words is the words rendering of 1 000.00;
**and** with the multi-page stub option **on**, the printed pages number `ceiling(10 ÷ 9) = 2`;
**and** with it **off**, exactly **one** page is produced, flagged as cropped, showing the first eight stub lines.

### H6. Ten bills paid by ten separate cheques

**Given** the same ten bills and the journal's next number `11111`,
**When** ten separate cheque payments are created,
**Then** their cheque numbers are exactly `11111` through `11120`.

### H7. Pre-printed stationery renumbers on printing

**Given** bank journal `BNK1` with manual numbering **off**, whose greatest existing cheque number is `000120`,
**When** three posted cheque payments are selected and *Print Checks* is used,
**Then** the dialogue opens pre-filled with `000121` (the greatest plus one, padded to six characters);
**and** confirming with `000121` numbers the three payments `000121`, `000122`, `000123`, marks them sent, and produces the printable document.

### H8. A non-numeric cheque number is refused

**When** `12A3` is entered in the pre-numbered dialogue,
**Then** it is refused with *Next Check Number should only contains numbers.*

### H9. A duplicate cheque number is refused

**Given** a posted cheque payment in `BNK1` numbered `0042`,
**When** another posted payment in the same journal is given the number `42`,
**Then** it is refused with *The following numbers are already used:* followed by *42 in journal BNK1* — because the comparison is made on the integer value.

### H10. Printing refuses a mixed selection

**Given** two cheque payments drawn on two different bank journals,
**When** *Print Checks* is used on both,
**Then** it is refused with *In order to print multiple checks at once, they must belong to the same bank journal.*

### H11. Printing refuses when no layout is configured

**Given** the company's cheque layout set to *None*,
**When** *Print Checks* is used,
**Then** a redirecting warning is raised reading *You have to choose a check layout. For this, go in Invoicing/Accounting Settings, search for 'Checks layout' and set one.* with the navigation labelled *Go to the configuration panel*.

### H12. Unmark sent puts the cheque back in the queue

**Given** a cheque payment numbered `00042`, status `in_process`, sent,
**When** *Unmark Sent* is used,
**Then** the sent flag is cleared, the cheque number is unchanged, and the payment reappears in the *Checks to Print* filter and in the bank journal card's counter.

### H13. Void cancels the payment and releases the bill

**Given** a cheque payment of 400.00 reconciled against a bill, status `in_process`, sent,
**When** *Void Check* is used,
**Then** the payment is reset to draft and then cancelled: its reconciliation is removed, its draft journal entry is deleted, its status becomes `canceled`;
**and** the bill's residual returns to its pre-payment value and its payment status returns to `not_paid`;
**and** the cheque number stays on the payment record.

### H14. A very large cheque number

**Given** a posted cheque payment,
**When** its number is set to `2147483647` and then to `2147483648`,
**Then** both writes succeed;
**and** printing a further payment on the same journal proposes `2147483649`.

### H15. Stub lines in a foreign currency

**Given** a bill of 150.00 in the company currency (dollar), posted,
**and** a partial cheque payment of 150.00 in a second currency (euro) whose rate makes 150.00 euro worth 75.00 dollar,
**When** the stub is built,
**Then** it holds exactly one line: due date 1 January 2016, number the bill's number, total `$ 150.00`, residual `$ 75.00`, paid `150.00 €` — the total and residual in the **bill's** currency, the paid amount in the **payment's** currency.

### H16. The cheque number reaches the journal items

**Given** an outgoing payment with the cheque method, number `2147483647`, amount 100.00, and no memo,
**When** it is posted,
**Then** every one of its journal items is labelled *Checks - 2147483647*.

---

## Group I — Line defaulting and deductibility

### I1. The purchase unit comes from the supplier record

**Given** product *Desk lamp* whose reference unit is *Units* and whose supplier record for this company specifies the purchase unit *Dozens*,
**When** the product is chosen on a **bill** line,
**Then** the line's unit is **Dozens**;
**and** on a customer invoice line the same product would give *Units*.

### I2. The label uses the purchase description

**Given** product *Office chair* whose purchase description is *Ergonomic, five-castor base*,
**When** it is chosen on a bill line,
**Then** the label reads *Office chair* then a line break then *Ergonomic, five-castor base*;
**and** on a customer invoice the sale description would be used instead.

### I3. The taxes come from the supplier taxes

**Given** product *Desk lamp* with supplier taxes {Purchase 15 %, Purchase 15 % (b)} and sale taxes {Sale 15 %},
**When** it is chosen on a bill line,
**Then** the line's taxes are the two purchase taxes.

### I4. An account with taxes seeds a line with no product

**Given** account 600000 carrying the purchase-typed tax *Purchase 15 %*,
**When** a bill line is created with that account and no product,
**Then** the line's taxes become {Purchase 15 %}.

### I5. An imported line is never overwritten

**Given** a bill line created by a decoder, marked imported, with unit price 97.31 and no taxes,
**When** a product is set on that line,
**Then** the unit price stays 97.31 and the taxes stay empty.

### I6. The most frequent account of a vendor

**Given** that over the past two years, journal items of *Papeterie Lambert* in this company used account 600100 twelve times and account 600000 five times, both being expense accounts,
**When** a bill line for that vendor is created without a product,
**Then** its account becomes **600100**.

### I7. The neighbours' account

**Given** a bill with three product lines whose last two both use account 600300,
**When** a fourth product line is added with no product and a vendor that has no history,
**Then** its account becomes **600300**.

### I8. The journal default is the last resort

**Given** a bill with one product line and no vendor history,
**When** a second product line is added with no product,
**Then** its account becomes the journal's default account **600000** (the neighbour rule needs more than two lines on the document).

### I9. Partial deductibility

**Given** a bill line of 100.00 with tax *Purchase 20 %* and a deductibility of **75 %**, the journal's private share account being **610000 Private Share**,
**When** the document is saved and posted,
**Then** the journal items are: 600000 debit 100.00; 600000 credit 25.00; 610000 debit 25.00; 131000 debit 15.00; 610000 debit 5.00; 400000 credit 120.00;
**and** the entry balances at 145.00 on each side;
**and** the two aggregate lines are labelled *«document number» - private part* and *«document number» - private part (taxes)*;
**and** the acting user is granted the partial-purchase-deductibility group if they lacked it.

### I10. Deductibility is refused outside purchase documents

**Given** a **customer invoice** line,
**When** its deductibility is set to 80,
**Then** it is refused with *Only vendor bills allow for deductibility of product/services.*

### I11. Deductibility out of range

**When** a bill line's deductibility is set to 120,
**Then** it is refused with *The deductibility must be a value between 0 and 100.*

---

## Group J — Upload, mail and decoding

### J1. Five separate files make five bills

**Given** five distinct Portable Document Format files,
**When** they are dropped on the purchase journal card,
**Then** five draft bills are created, one per file, each carrying its file and the chatter message *This document was created from the following attachment(s).*
**and** the caller is redirected to a list of the five under the title *Generated Documents*.

### J2. A container and its embedded structured file make one bill

**Given** one Portable Document Format file with an embedded extensible-markup-language file,
**When** it is dropped,
**Then** the embedded file joins the container's group and **one** bill is created carrying both.

### J3. A mail with three representations makes one bill

**Given** a message to the purchase journal's address carrying one Portable Document Format file, one image and one extensible-markup-language file, all with unrelated names,
**When** it is routed,
**Then** the mixed-types grouping places all three in one group (no two share a format label), and **one** bill is created.

### J4. A mail with three Portable Document Format files makes three bills

**Given** the same address and a message carrying three Portable Document Format files,
**Then** three groups are formed and **three** bills are created; the original message keeps the first group's files and two copies of the message are created on the other two bills.

### J5. A mail with no attachment bounces

**Given** a message to the purchase journal's address with no attachment at all,
**Then** **no** document is created;
**and** a bounce is sent to the sender whose body reads *Hi,* / *Your email has been discarded. the e-mail address you have used only accepts new invoices:* / *For new invoices, please ensure a PDF or electronic invoice file is attached* / *To add information to a previously sent invoice, reply to your "sent" email* / *For any other question, write to «the company electronic mail address».* / the company name.

### J6. A mail forwarded by an internal user resolves the real sender

**Given** an internal user forwards a supplier's message to the journal address, and the supplier's address appears in the forwarded body,
**Then** the created bill's partner is the **supplier**, not the internal user, because the resolver falls back to the body when the sender resolves to an internal partner.

### J7. A decoder failure is reported without losing the bill

**Given** a bill created from an upload whose decoder raises,
**Then** the transaction is rolled back to the state before decoding;
**and** the bill still exists with its attachment;
**and** its chatter carries *Error importing attachment '«file name»' (type=pdf):* / *This specific error occurred during the import:* / the error text.

### J8. A decoder refusal is reported as a reason

**Given** a decoder that answers *unsupported profile*,
**Then** the chatter carries *Attachment «file name» not imported: unsupported profile*.

### J9. No decoder means nothing is written

**Given** a file for which no decoder declares a non-zero priority,
**Then** the bill stays empty and only a technical log line is written.

### J10. A supplier posting a message does not trigger decoding

**Given** an existing bill and a message posted on it by a **portal** user carrying a structured file,
**Then** the file is attached but **no** decoding runs, because decoding requires an active internal user.

### J11. Uploading without a journal

**Given** a context naming neither a journal nor a classifiable document type,
**When** the creation operation is called,
**Then** it is refused with *The journal in which to upload the invoice is not specified. *

---

## Group K — Reset, cancel and delete

### K1. Reset to draft keeps the number

**Given** a posted bill numbered `BILL/2026/01/0003`,
**When** it is reset to draft,
**Then** its status is `draft`, its number is still `BILL/2026/01/0003`, its analytic lines are deleted, its sending data is cleared;
**and** its attachments are **not** detached (detaching only applies to sale documents).

### K2. Reset refuses a hashed document

**Given** a posted bill in a journal that secures entries with a hash, already hashed,
**When** a reset to draft is attempted,
**Then** it is refused with *You cannot reset to draft a locked journal entry.*
**and** the reset button is not shown at all.

### K3. Cancel removes reconciliations

**Given** a posted bill of 1 128.00 fully paid by a cheque,
**When** the bill is cancelled,
**Then** the reconciliation is removed, the bill's status is `cancel`, its automatic posting mode is `no`, and the payment's own status recomputes back from `paid`.

### K4. Cancel refuses an unresettable document

**Given** a posted bill that is a cash-basis entry,
**When** cancel is attempted,
**Then** the underlying reset refuses with *You cannot reset to draft a tax cash basis journal entry.*

### K5. Deleting a numbered bill in the middle of a chain

**Given** posted bills numbered `BILL/2026/01/0001`, `.../0002`, `.../0003`, the middle one reset to draft,
**When** a billing clerk (not an accounting manager) deletes it, quick encoding being off,
**Then** it is refused with *You cannot delete this entry, as it has already consumed a sequence number and is not the last one in the chain. You should probably revert it instead.*
**and** an accounting manager may proceed, after which `.../0003` is flagged as having made a gap.

### K6. The restrictive audit trail blocks deletion

**Given** the company's restrictive audit trail on and a bill that has been posted before,
**When** deletion is attempted,
**Then** it is refused with *To keep the restrictive audit trail, you can not delete journal entries once they have been posted.* newline *Instead, you can cancel the journal entry.*

### K7. Bulk removal classifies each document

**Given** three bills: one draft never posted; one posted and reversible; one posted in a company with the restrictive audit trail,
**When** the bulk removal routine runs,
**Then** the first is deleted, the second is reset to draft and deleted (or reversed if it fails the deletability test), and the third is cancelled;
**and** any document that is not deletable at all is reversed with a cancelling reversal.

### K8. Switching the type of a numbered document

**Given** a bill that has been posted and numbered,
**When** the type switch is attempted,
**Then** it is refused with *You cannot switch the type of a document with an existing sequence number.*

---

## Group L — Payment status

### L1. Partial payment

**Given** a posted bill of 1 128.00,
**When** 400.00 is paid and reconciled,
**Then** the residual is 728.00 and the payment status is `partial`.

### L2. Full payment

**When** the remaining 728.00 is paid and reconciled,
**Then** the residual is 0 and the payment status is `paid` (in the base package; an accounting package answers `in_payment` until the bank statements are reconciled).

### L3. Offset entirely by a credit note

**Given** a posted bill of 208.00 and a posted vendor credit note of 208.00, reconciled against each other,
**Then** the bill's payment status becomes **`reversed`** and the credit note's becomes `paid`.

### L4. A zero-total bill

**Given** a bill whose total is 0.00,
**When** it is posted,
**Then** its payment status is immediately `paid`.

### L5. Blocking

**Given** a posted, unpaid bill,
**When** the payment block is toggled,
**Then** the status becomes `blocked` and stays so through every recomputation;
**and** registering a payment is refused with *You cannot register payments for blocked invoices.*
**and** toggling again sets it to `not_paid` and triggers a recomputation.

### L6. Blocking a paid bill is refused

**Given** a fully paid bill,
**When** the block is attempted,
**Then** it is refused with *You can't block a paid invoice.*

### L7. Unreconciling reverts the status

**Given** a bill with status `paid`,
**When** its reconciliation is undone,
**Then** the status returns to `not_paid` (or `partial` if some reconciliation remains).

---

## Group M — The invoice analysis report

### M1. A bill line appears with negative amounts

**Given** the posted bill of A1,
**When** the analysis report is read,
**Then** it holds **two** rows (one per product line; tax lines and the payable term line are excluded);
**and** the row for line 1 has quantity `1 × 1 × (−1) = −1`, untaxed amount in currency `800.00 × (−1) = −800.00`, untaxed amount `−800.00 × 1 = −800.00` (the negative of the ledger balance, times the conversion rate 1), total `920.00 × (−1) ÷ 1 = −920.00`, total in currency `−920.00`;
**and** its margin is **0**, because the document type is a purchase type.

### M2. Quantities are restated in the template unit

**Given** the row for line 2, whose line unit is *Dozens* and whose template unit is *Dozens*,
**Then** the unit factor is 1 and the quantity is `1 × 1 × (−1) = −1`.

**Given** instead a bill line of 3 *Dozens* of a product whose reference unit is *Units*,
**Then** the quantity is `3 × 12 × (−1) = −36` and the reported unit is *Units*.

### M3. The weighted average price

**Given** two bill rows for the same product: quantity −36 with untaxed amount −360.00, and quantity −12 with untaxed amount −130.00,
**When** the *Average Price* measure is aggregated over both,
**Then** the result is `(−360.00 + −130.00) ÷ (−36 + −12) = −490.00 ÷ −48 = 10.2083…` per **Unit** — not the mean of the two per-row averages.

### M4. Zero quantity does not break the average

**Given** a bill row with quantity 0 and a balance of 50.00,
**Then** its per-row average price is 0 (the division is coalesced) and it contributes nothing to the aggregate.

### M5. Inventory value on a purchase

**Given** a bill line of 5 units of a product whose standard price for this company is 42.00, conversion rate 1,
**Then** the inventory sign for a purchase type is **+1** and the inventory value is `1 × 5 × 1 × (+1) × 42.00 = 210.00`.

### M6. Margin on a customer invoice and on a credit note

**Given** a customer invoice line of 2 units with a ledger balance of −300.00 and a standard price of 100.00,
**Then** the margin is `1 × (300.00 − 2 × 100.00) = 100.00`.
**Given** the mirror credit note line with a ledger balance of +300.00,
**Then** the margin is `1 × (−300.00 + 2 × 100.00) = −100.00`.

### M7. Multi-currency conversion

**Given** two companies, one in euro (the active company) and one in United States dollars, and a stored rate making one euro worth 1.10 dollars today,
**When** the report is read across both,
**Then** the euro company's rows use a conversion rate of 1;
**and** the dollar company's rows use `unit_factor(euro, today) ÷ rate_record(dollar, latest on or before today)`, which turns a dollar balance into euros;
**and** the untaxed amount and average price columns are expressed in **euro** while the "in currency" columns stay in the document currency.

### M8. Only product lines are rows

**Given** a bill with one product line, one tax line, one payable term line, one section line and one rounding line,
**Then** the report holds exactly **one** row.

### M9. Multi-company visibility

**Given** a user whose allowed companies are A and B,
**Then** the report shows only rows whose company is A or B, by the multi-company record rule.

---

## Group N — Debit notes

### N1. A debit note from a bill

**Given** a posted bill `BILL/2026/01/0003`,
**When** a debit note is created with date 20 February 2026, reason *Freight omitted*, no journal override and *Copy Lines* off,
**Then** a draft document of type `in_invoice` is created with vendor reference *BILL/2026/01/0003, Freight omitted*, bill and accounting dates 20 February 2026, the same journal, **no** payment term, **no** lines, and a debit origin pointing at the bill;
**and** the **debit note's** chatter records *This debit note was created from: «link»*, the link naming `BILL/2026/01/0003`;
**and** the bill's own chatter records nothing about the debit note;
**and** the bill shows a *Debit Notes* counter of 1, the smart button having been hidden while the count was zero.

### N2. Numbering with a dedicated debit note sequence

**Given** the purchase journal's dedicated debit note sequence on,
**When** the debit note of N1 is posted in February 2026,
**Then** its number begins with `DBILL/2026/02/` and it is numbered independently of ordinary bills.

### N3. A debit note from a vendor credit note, raised through the dialogue

**Given** a posted vendor credit note of one line, 200.00 on account 600000,
**When** a debit note is raised from it through the *Create Debit Note* dialogue,
**Then** the *Copy Lines* switch is **not shown**, because the source's type is `in_refund`;
**and** the switch therefore keeps its default of off;
**and** the produced document's type is `in_invoice`, it carries **no** line, and its debit origin is the credit note.

### N3b. A debit note from a vendor credit note, with *Copy Lines* set programmatically

**Given** the same posted vendor credit note of one line, 200.00 on account 600000,
**When** the wizard is driven without the form and *Copy Lines* is set to true,
**Then** the produced document's type is `in_invoice` and it **does** carry a copy of the credit note's line, 200.00 on account 600000 — the guard meant to refuse this never matches (**compatibility finding**, `business-rules.md` §11);
**and** a rebuild implementing the corrected behaviour produces a document with **no** line instead, which is the only difference between the two behaviours.

### N5. The printed debit note

**Given** the debit note of N1, posted,
**When** its printable document is produced,
**Then** the title reads *Debit Note* rather than *Invoice*, and would read *Draft Debit Note*, *Cancelled Debit Note*, *Proforma Debit Note*, *Draft Proforma Debit Note* or *Cancelled Proforma Debit Note* in the corresponding states;
**and** no link to `BILL/2026/01/0003` appears anywhere on the printed page — the debit origin is shown only on the form;
**and** the date caption is the ordinary purchase-document caption, **not** *Debit Note Date*, because the substituted caption serves the `out_invoice` branch only (**compatibility finding**, `interfaces.md` §8.1).

### N4. A second debit note is refused

**Given** a bill that already has a debit note,
**When** another is attempted,
**Then** it is refused with *You can't make a debit note for an invoice that is already linked to a debit note.*

---

## Group O — Quick encoding

### O1. One line generated from a typed total

**Given** the company's quick encoding set to *Customer Invoices and Vendor Bills*, a draft bill with vendor *Papeterie Lambert* who has no history, the journal default account 600000 carrying the purchase tax *Purchase 21 %*,
**When** the total 100.00 is typed,
**Then** one line is generated on account 600000 with tax *Purchase 21 %* and a unit price of `82.64` (100.00 treated as tax-included);
**and** the rounding reconciliation nudges the tax from 17.35 to **17.36** so that the document total is exactly 100.00.

### O2. The mixed early-discount adjustment

**Given** the same setting plus a payment term with a 2 % early discount in the `mixed` mode and a single 21 % percentage tax,
**When** 100.00 is typed,
**Then** the net is `round_to(Euro, 100.00 ÷ ((1 − 0.02) × 0.21 + 1)) = round_to(Euro, 100.00 ÷ 1.2058) = 82.93`.

### O3. A surviving mismatch blocks posting

**Given** a quick-encoded bill whose typed total is 100.00 and whose computed total is 99.50 after manual line edits,
**When** posting is attempted,
**Then** it is refused with *The current total is €99.50 but the expected total is €100.00. In order to post the invoice/bill, you can adjust its lines or the expected Total (tax inc.).*

### O4. The suggested bill date

**Given** quick encoding on, an empty bill date, and a previous posted document of the same journal and company whose bill date is 31 July 2026, today being 11 September 2026,
**When** the form opens,
**Then** the bill date is suggested as **31 July 2026** passed through the accounting-date algorithm, which for a monthly series in a later month yields **31 July 2026**.

---

## Group P — Abnormal detection

### P1. An unusually large bill warns

**Given** a vendor with fifteen prior posted bills of the same type, company and currency whose totals average 1 000.00 with a sample standard deviation of 50.00,
**When** a draft bill of 1 500.00 is captured,
**Then** the wiggle room is `2 × 50.00 = 100.00` and 1 500.00 lies outside `[900.00, 1 100.00]`, so the warning fires reading *The amount for Papeterie Lambert appears unusual. Based on your historical data, the expected amount is €1,000.00 (± €100.00).* / *Please verify if this amount is accurate.*
**and** automatic posting of this bill is suppressed.

### P2. Too little history means no warning

**Given** a vendor with only four prior posted bills,
**Then** the window rows numbered 10 to 30 yield nothing, the fallback deviations of ten thousand million are used, and **no** warning fires.

### P3. An early bill warns

**Given** a vendor billing monthly, with a computed mean gap of 30.5 days and a sample deviation of 1.0 day, whose last bill date is 1 August 2026,
**When** a draft bill dated 10 August 2026 is captured,
**Then** the mean exceeds 25, so the deviation becomes 2.0 and the wiggle room 4.0;
**and** `10 August − 1 August = 9` days is below `integer_part(30.5 − 4.0) = 26`, so the warning fires reading *The billing frequency for Papeterie Lambert appears unusual. Based on your historical data, the expected next invoice date is not before 27 August 2026 (every 30 (± 4) days).* / *Please verify if this date is accurate.*

### P4. Silencing

**Given** the vendor's *ignore abnormal invoice date* flag set for the company,
**Then** the date warning never fires for that vendor, whatever the history.

---

## Group Q — Intercompany clearing

### Q1. A bill of company A paid by company B

**Given** companies A and B of one group, each with an intercompany clearing journal; A has an intercompany payable account `A-INTERCO-PAY`; B has an intercompany receivable account `B-INTERCO-REC`,
**and** a vendor bill of A for 500.00 with a payable line of 500.00 credit on A's account 400000,
**and** a payment transaction on that bill carried by a payment of **B** in the `done` state, whose own counterpart line is 500.00 debit on B's account 400000,
**When** the bill of A is posted,
**Then** a clearing entry is created in **B**'s clearing journal with reference the payment memo, holding: 500.00 credit on B's 400000 (the payment's own counterpart account) with the bill's partner, and 500.00 debit on `B-INTERCO-REC` with **A**'s company partner; it is posted and its first line is reconciled against the payment's counterpart lines;
**and** a settlement entry is created in **A**'s clearing journal with reference *Interco Settlement - «the payment memo»*. Its amounts follow the formula of `calculations.md` §16: the sum of the balances of the bill's payable lines is −500.00, so `clearing_counterpart_balance = +500.00`; the first line carries `− clearing_counterpart_balance` and the second `clearing_counterpart_balance`. The entry is therefore 500.00 **credit** on `A-INTERCO-PAY` with **B**'s company partner, and 500.00 **debit** on A's 400000 with the payment's partner;
**and** the second line is reconciled against the bill's payable line, clearing the bill;
**and** both labels read `«the bill partner's display name» / «the bill's number»`.

### Q2. Nothing happens without the configuration

**Given** the same situation but company A has no intercompany clearing journal,
**Then** no clearing entry is created and the bill stays unpaid.

---

## Group R — Multi-currency

### R1. A bill in a foreign currency

**Given** a bill written in United States dollars with a document rate of 1.20 dollars per euro, holding the two lines of A1 (800.00 and 160.00 dollars) with the same taxes,
**When** it is posted,
**Then** the commercial amounts stay 800.00, 160.00, 144.00, 24.00 and 1 128.00 **dollars**;
**and** the balances are 666.67, 133.33, 120.00, 20.00 and −940.00 **euros**;
**and** the entry balances exactly, the payable term line absorbing any residue as the balance line.

### R2. A zero or negative rate is refused

**Given** a bill in a foreign currency,
**When** its document rate is set to 0,
**Then** it is refused with *The currency rate must be strictly positive.*

### R3. An archived currency blocks posting

**Given** a bill whose currency has been archived,
**When** posting is attempted,
**Then** it is refused with *You cannot validate a document with an inactive currency: USD*
**and** the *Activate Currency* action reactivates it.

---

## Group S — Access and multi-company

### S1. A billing clerk may post but not review in a narrowed installation

**Given** an installation where the review predicate is narrowed to the accountant group,
**When** a billing clerk tries to set the reviewed flag,
**Then** it is refused with *You don't have the access rights to perform this action.*

### S2. A user without the invoicing group cannot post

**Given** a user holding only the read-only accounting group,
**When** posting is attempted,
**Then** it is refused with *You don't have the access rights to post an invoice.*

### S3. A portal supplier sees only their own bills

**Given** a portal user whose commercial partner is *Papeterie Lambert*,
**When** they open the portal document list,
**Then** they see only documents whose status is neither cancelled nor draft, whose type is one of the four invoice types, and whose partner is *Papeterie Lambert* or one of its children;
**and** the *Bills* filter narrows to the three purchase types.

### S4. Cross-company accounts are refused at posting

**Given** a bill of company A whose line uses an account belonging only to company C (neither A nor a parent of A),
**When** posting is attempted,
**Then** it is refused with *The entry is using accounts («the account display names») from a different company.*

### S5. Documents are scoped by company

**Given** a user whose allowed companies are A and B,
**Then** the document list, the journal item list and the analysis report all hide documents of company C.


---

## Group T — Discounts, auto-complete and the decoding contract

### T1. Discount-allocation lines on a bill

**Given** the company's *Vendor Bills Discounts Account* set to **710000 Purchase Discounts**, and a draft bill of one line: 10 units of a service at 50.00 each, 10 % line discount, expense account 600000, no tax, document currency Euro,
**When** the bill is saved and posted,
**Then** the entry holds four items: `product` 600000 debit 450.00; `discount` 600000 debit 50.00; `discount` 710000 credit 50.00; `payment_term` 400000 credit 450.00;
**and** both `discount` items are labelled *Discount*, carry no tax, carry no maturity date and are not reconcilable;
**and** the document total is 450.00, so the supplier is owed the net amount.

### T2. Discount allocation is skipped when the accounts coincide

**Given** the same company setting, and a draft bill whose single discounted line already uses account **710000 Purchase Discounts**,
**When** the bill is saved,
**Then** **no** `discount` item is produced, because the line's own account equals the allocation account.

### T3. Early-payment-discount lines on a bill in the mixed mode

**Given** the vendor's payment term *2/7 Net 30* (early discount on, 2 %, 7 days, one instalment of 100 % at 30 days) with the computation mode `mixed`, and a draft bill of one line: 1 unit at 1 000.00 on 600000 with the tax *Purchase 20 %* (tax-excluded, 100 % to 131000),
**When** the bill is saved and posted,
**Then** the entry holds five items: `product` 600000 debit 1 000.00 carrying the tax; `epd` 600000 credit 20.00 carrying the tax; `epd` 600000 debit 20.00 with the taxes cleared; `tax` 131000 debit 196.00; `payment_term` 400000 credit 1 196.00;
**and** both `epd` items are labelled *Early Payment Discount (2.0%)*;
**and** the expense account nets to 1 000.00 while the deductible input tax is 196.00 rather than 200.00.

### T4. The same term in the other two modes produces no such line

**Given** the same bill but the computation mode `included` (*On early payment*), and then `excluded` (*Never*),
**When** the bill is saved in each case,
**Then** no `epd` item exists in either case, the tax is 200.00 and the total is 1 200.00.

### T5. A fixed-amount duty is not discounted

**Given** the bill of T3 with a second tax on the line: a fixed amount of 5.00 per unit,
**When** the bill is saved,
**Then** the fixed-amount tax is dispatched out of the base lines before the early-discount aggregation, so it is **not** among the taxes carried by the base-shift item and the discount is not applied to the amount it represents;
**and** the amount of that fixed tax is therefore not reduced by the two per cent.

### T6. Private-share lines on a foreign-currency bill

**Given** a bill written in United States dollars at a document rate of 1.25 dollars per euro, company currency Euro, with one line of 100.00 dollars at 75 % deductibility and a 20 % purchase tax,
**When** the bill is saved,
**Then** the `non_deductible_product` item carries **balance −25.00 and amount in currency −20.00**, and the `non_deductible_product_total` item carries **balance +25.00 and amount in currency +20.00** — the two currency columns exchanged with respect to every other item of the entry (**compatibility finding**, `calculations.md` §10);
**and** a rebuild implementing the corrected behaviour produces −20.00 / −25.00 and +20.00 / +25.00 respectively.

### T7. Auto-complete appends the lines of an earlier bill

**Given** a posted bill of two lines (*Office chair* 800.00 and *Desk lamp* 160.00) in United States dollars with fiscal position *Reverse charge*, and a new draft bill of the same company already carrying one typed line *Courier* 30.00, in Euro, with no fiscal position and vendor *Alpha Supplies*,
**When** the earlier bill is chosen in the **Auto-Complete** picker,
**Then** the draft carries **three** lines: *Courier* 30.00, *Office chair* 800.00 and *Desk lamp* 160.00 — the copies are appended, nothing is replaced;
**and** the draft's currency becomes United States dollars and its fiscal position becomes *Reverse charge*;
**and** the vendor stays *Alpha Supplies*, and the bill date, accounting date, vendor reference, payment reference, payment term, recipient bank account and journal are unchanged;
**and** the picker is empty again, so nothing records that the auto-complete happened.

### T8. Choosing the same source twice appends twice

**Given** the draft of T7 after the auto-complete,
**When** the same earlier bill is chosen again,
**Then** the draft carries **five** lines: *Courier*, *Office chair*, *Desk lamp*, *Office chair*, *Desk lamp*.

### T9. A decoder refuses a document that already has lines

**Given** a draft bill that already carries one product line, and a file whose decoder consults the standard refusal helper,
**When** the decoding contract runs on that file,
**Then** nothing is written to the document;
**and** its chatter receives *Attachment «the file name» not imported: The invoice already contains lines.*

### T10. Purchase-order linking from the decoding path

**Given** an empty bill created from an uploaded file, and a decoder that adds two lines and no line existed before,
**When** the decoding contract completes,
**Then** the two lines are marked imported;
**and** the purchase-order matching hook is called once with a **four-second** budget, not the ten seconds of the manual path;
**and** when that hook raises a user error, the failure is logged as *Failed to link bill to purchase order* and the import still succeeds with the two lines in place;
**and** the post-processing hook runs afterwards in every one of those cases.

### T11. No purchase-order linking when the document already had lines

**Given** an existing draft bill that already carries one line, and an attachment posted on it by an internal user whose decoder adds two more lines,
**When** the decoding contract completes,
**Then** the two new lines are marked imported;
**and** the purchase-order matching hook is **not** called from this path;
**and** the post-processing hook still runs.

### T12. The journal-subscriber notification after decoding

**Given** a purchase journal carrying one notification address, and a file uploaded onto it whose decoder succeeds,
**When** the newly created document finishes decoding,
**Then** the document's portal access token exists and has been written before the notification is composed;
**and** one *Journal Notification* mail is sent to that address, carrying a copy of every attachment of the document and of the decoded group, each copy named `MAIL_` followed by the original name;
**and** when the sending fails, the failure is logged and the document, its lines and its attachments are kept exactly as decoded.

### T13. Every write marks the document as manually modified

**Given** a bill created from an uploaded file and decoded, whose manually-modified flag is false,
**When** any routine writes any field on it without naming that flag and without the suppression marker — for example another domain writing the delivery date,
**Then** the flag becomes true in the same write;
**and** the automatic-posting learning wizard no longer opens for that document, and the vendor's run of unmodified bills is broken at it (**compatibility finding**, `business-rules.md` §19).

### T14. Posting does not mark the document

**Given** the same decoded bill with the flag false,
**When** it is posted,
**Then** the flag is still false, because the whole posting routine carries the suppression marker;
**and** the learning wizard may therefore open, subject to the other eight conditions of `workflows.md` §8.3.

### T15. Naming an extra recipient grants portal access to a bill

**Given** a posted vendor bill whose partner is *Papeterie Lambert*, and a contact *Auditor* who is neither that partner nor an internal user,
**When** a chatter message is posted on the bill naming *Auditor* as an extra recipient,
**Then** the bill's portal access token is generated if it did not exist;
**and** *Auditor* is placed in the additional-intended-recipient group, which is evaluated before every other recipient group;
**and** the notification sent to *Auditor* carries a button leading to the bill's portal page with that token, so *Auditor* can read the bill without any further permission being granted.
