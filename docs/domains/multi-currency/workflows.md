# Multi-Currency — Workflows

Every operational procedure of this domain, end to end: the actor, the preconditions, the numbered
steps, the branches, the records each step writes with their field values, the messages that can
stop the procedure, and the postconditions. The states these procedures move through, with their
stored values and their guards, are in [state-machines.md](state-machines.md); the arithmetic each
step performs is in [calculations.md](calculations.md); the rules cited as `MCUR-nnn` are in
[business-rules.md](business-rules.md).

**Currencies used in the examples.** The United States dollar (`USD`), the euro (`EUR`), the pound
sterling (`GBP`) and the Japanese yen (`JPY`), each written with its reproduced three-letter code
of the international currency-code standard. Amounts are written with a full stop as the decimal
separator. Unless a procedure says otherwise the company keeps its books in the United States
dollar, whose rounding factor is one hundredth.

Contents:

1. [Activate a foreign currency](#1-activate-a-foreign-currency)
2. [Record a currency rate by hand](#2-record-a-currency-rate-by-hand)
3. [Update rates automatically on a schedule](#3-update-rates-automatically-on-a-schedule)
4. [Set the main currency of a company](#4-set-the-main-currency-of-a-company)
5. [Configure the exchange difference journal and accounts](#5-configure-the-exchange-difference-journal-and-accounts)
6. [Issue a customer invoice in a foreign currency](#6-issue-a-customer-invoice-in-a-foreign-currency)
7. [Enter a vendor bill in a foreign currency](#7-enter-a-vendor-bill-in-a-foreign-currency)
8. [Override the rate applied to a document](#8-override-the-rate-applied-to-a-document)
9. [Register a payment in a currency other than the company currency](#9-register-a-payment-in-a-currency-other-than-the-company-currency)
10. [Reconcile foreign currency items and produce the exchange difference](#10-reconcile-foreign-currency-items-and-produce-the-exchange-difference)
11. [Create an exchange difference entry](#11-create-an-exchange-difference-entry)
12. [Undo a reconciliation](#12-undo-a-reconciliation)
13. [Record a bank transaction in a foreign currency](#13-record-a-bank-transaction-in-a-foreign-currency)
14. [Match a bank transaction against a foreign currency document](#14-match-a-bank-transaction-against-a-foreign-currency-document)
15. [Change the rounding factor of a currency](#15-change-the-rounding-factor-of-a-currency)
16. [Deactivate a currency](#16-deactivate-a-currency)
17. [Convert an amount between two foreign currencies](#17-convert-an-amount-between-two-foreign-currencies)
18. [Close a period across companies whose main currencies differ](#18-close-a-period-across-companies-whose-main-currencies-differ)
19. [Recognise unrealised gains and losses at a reporting date](#19-recognise-unrealised-gains-and-losses-at-a-reporting-date)
20. [Where the states of these procedures are specified](#20-where-the-states-of-these-procedures-are-specified)
21. [Reconciliation notes](#21-reconciliation-notes)

---

# 1. Activate a foreign currency

**Actor.** Accounting Manager, or Settings Administrator.

**Preconditions.** The currency exists in the shipped catalogue, or the actor creates it. The actor
holds write access on Currency.

## 1.1 Steps

1. The actor opens the currency catalogue. The list shows active and archived currencies together,
   because the action that opens it forces archived records to be included. Its columns are the
   currency code (`name`), the symbol (`symbol`), the full name (`full_name`), the last rate date
   (`date`) under the heading "Last Update", the current rate (`rate`), the inverse rate
   (`inverse_rate`, hidden by default) and the activity flag (`active`) as a toggle.
2. The actor locates the currency by code or by name, or starts a new one.
3. **Branch A — creating a currency.** The actor fills the currency code, the full name, the
   symbol, the symbol position (`position`), the currency unit label (`currency_unit_label`), the
   currency subunit label (`currency_subunit_label`) and, when the actor holds the technical
   features group, the rounding factor (`rounding`). On save:
   - the decimal places (`decimal_places`) are derived from the rounding factor and stored, by the
     formula of [calculations.md](calculations.md) section 2;
   - the uniqueness of the currency code is checked (`MCUR-001`), with the message "The currency
     code must be unique!";
   - the currency code is required and is capped at three characters (`MCUR-002`);
   - the symbol is required (`MCUR-004`);
   - the rounding factor must be strictly positive (`MCUR-003`), with the message "The rounding
     factor must be greater than 0!";
   - the multi-currency capability is re-evaluated (`MCUR-008`);
   - the cached catalogue of active currencies is invalidated (`MCUR-014`).
4. **Branch B — activating a shipped currency.** The actor switches the activity toggle on. The
   record is written with the activity flag true. No further validation applies, because activating
   a currency can never conflict with existing data.
5. In both branches the number of active currencies is counted after the write. When it is now
   greater than one:
   - the multi-currency permission group is applied to the internal-user group, which makes the
     document currency selector, the amount in currency column and the document rate field visible
     to every internal user;
   - the price list capability group is applied to the internal-user group when internal users do
     not already hold it, and a default price list is created, or reactivated, for every company.
6. The actor opens the currency's form and records at least one rate through procedure 2, unless
   the currency is the main currency of the company the actor is working in. In that case the
   derived flag `is_current_company_currency` (is current company currency) is true, the rates tab
   is hidden, and an informational banner reads "This is your company's currency."

## 1.2 Records written

| Record | Fields |
|---|---|
| Currency | `name`, `full_name`, `symbol`, `position`, `rounding`, the derived `decimal_places`, `active` set true, `currency_unit_label`, `currency_subunit_label`, optionally `iso_numeric` |
| Permission group membership | The multi-currency group, and possibly the price list group, applied to the internal-user group |
| Price List | One default price list per company, when the price list capability was newly granted |

## 1.3 Failure conditions

| Condition | Effect |
|---|---|
| The currency code duplicates an existing one, archived records included | The creation is refused with "The currency code must be unique!" and nothing is written |
| The currency code is empty or longer than three characters | The creation is refused by the required-field check or by the storage |
| The symbol is empty | The creation is refused by the required-field check |
| The rounding factor is zero or negative | The creation is refused with "The rounding factor must be greater than 0!" |

## 1.4 Postconditions

The currency can be chosen on documents, journals, accounts, price lists and companies. When it is
the second active currency, currency columns appear across the platform for every internal user
and the platform has entered the multi-currency state of
[state-machines.md](state-machines.md) section 3.

---

# 2. Record a currency rate by hand

**Actor.** Accounting Manager.

**Preconditions.** The currency exists. The actor holds write access on Currency Rate. Recording a
rate for the company's own main currency is possible and meaningful only when the platform
deliberately keeps rate rows for it; the ordinary case is a currency other than the main one.

## 2.1 Steps

1. The actor opens the currency's form and selects the Rates tab, or runs the contextual action
   "Show Currency Rates" from the currency, which opens the rate list filtered on that currency
   with the currency pre-filled on new rows.
2. The actor adds a line. The new row is initialised with the rate date (`name`) set to today in
   the actor's time zone, the currency (`currency_id`) set to the currency in context, and the
   company (`company_id`) set to the root company of the company the actor is working in.
3. The actor may change the rate date.
4. The actor may clear the company, which makes the row shared by every company. The company column
   is shown only to holders of the multi-company permission group and carries the placeholder
   "Visible to all" (`MCUR-193`).
5. The actor types a value into one of the two rate columns.
   - **Branch A** — the actor types into the column whose heading is the currency's code, then
     " per ", then the company currency's code. That column is the company rate (`company_rate`).
     The technical rate (`rate`) is derived as the company rate multiplied by the rate the
     company's own main currency carries in force for that company, and the inverse company rate
     (`inverse_company_rate`) is then derived as the reciprocal of the company rate.
   - **Branch B** — the actor types into the column whose heading is the company currency's code,
     then " per ", then the currency's code. That column is the inverse company rate. The company
     rate is derived as its reciprocal, and the technical rate follows.
   The arithmetic of the three representations is in [calculations.md](calculations.md) section 9.
6. The plausibility check of `MCUR-021` runs. When the implied technical rate differs by more than
   twenty percent from the technical rate of the latest strictly earlier row of the same currency
   and the same company scope, a non-blocking dialogue is shown, titled "Warning for " followed by
   the currency code, with the body "The new rate is quite far from the previous rate.⏎Incorrect
   currency rates may cause critical problems, make sure the rate is correct!" The actor may
   proceed.
7. The actor saves. The guards run in this order: the company must be a root company (`MCUR-020`);
   the triple of rate date, currency and company must be unique (`MCUR-022`); the technical rate
   must be strictly positive (`MCUR-023`).
8. On save, the derived inverse rate of every Currency is invalidated so that the current rate
   column of the currency list and the rate as text (`rate_string`) on every currency form refresh
   at the next read (`MCUR-036`).

## 2.2 Records written

| Record | Fields |
|---|---|
| Currency Rate | `name` (the rate date), `currency_id`, `company_id`, `rate`, after the precedence reduction of `MCUR-024` |

## 2.3 Failure conditions

| Condition | Message |
|---|---|
| The company is a branch | "Currency rates should only be created for main companies" |
| A row already exists for the same currency, company scope and day | "Only one currency rate per day allowed!" |
| The technical rate is zero or negative, including the case where none of the three representations was supplied | "The currency rate must be strictly positive." |
| The rate date is cleared | The required-field check of the persistence layer refuses the save |

## 2.4 Postconditions

Every conversion for a date on or after the rate date, and before the next row of the same scope,
uses the new rate. No existing journal item changes (`MCUR-031`). A draft document whose rate date
falls inside the new row's window picks the new rate up the next time its currency, its company or
its invoice date is touched, or when the actor runs the rate refresh operation of procedure 8.

---

# 3. Update rates automatically on a schedule

**Actor.** The scheduled rate updater, a background job. Configured by the Settings Administrator.

**Preconditions.** The automatic rate retrieval capability package is installed, which is what the
setting `module_currency_rate_live` (automatic currency rates) does. A rate service and an interval
are configured for the company. At least two currencies are active.

## 3.1 Steps

1. The job wakes at its configured moment.
2. For each company whose automatic retrieval is enabled and whose next run moment has been
   reached:
   1. the job determines the currencies to fetch: every active currency other than the company's
      main currency;
   2. the job calls the configured rate service, passing the company's main currency as the base
      and today's date. The service answers with a mapping from currency code to a value meaning
      "units of that currency per one unit of the base currency";
   3. for every returned code that matches an active currency, the job treats the returned value as
      the company rate and derives the technical rate by the arithmetic of `MCUR-024`;
   4. the job creates a Currency Rate for today, that currency and the company's root, or updates
      the existing row for that triple when one exists, which is the case when the job has already
      run today;
   5. a returned code matching no active currency is ignored, and an active currency the service
      does not return keeps its previous rate;
   6. the job advances the company's next run moment by the configured interval.
3. When the fetch is triggered interactively through the "Update now" control, steps 2.1 to 2.5 run
   immediately for the company the actor is working in and the next run moment is **not** advanced.
4. A failure of the external service is logged, leaves the rate table untouched and is not retried
   within the same run.

## 3.2 Branches

| Interval | Effect |
|---|---|
| Manually | The scheduled job never fires for that company; only the "Update now" control writes rates. |
| Daily | The next run moment advances by one day after each run. |
| Weekly | The next run moment advances by seven days. |
| Monthly | The next run moment advances by one month. |

## 3.3 Records written

One Currency Rate per fetched currency per run, always dated today and scoped to the company's
root, subject to the same three guards as procedure 2.

## 3.4 Postconditions

The rate table is current. The "Last Update" column of the currency catalogue shows today for every
fetched currency.

---

# 4. Set the main currency of a company

**Actor.** Settings Administrator or Accounting Manager.

**Preconditions.** The actor holds write access on Company.

## 4.1 Steps

1. The actor opens the accounting settings of the company and locates the Main Currency field,
   which is bound to the company's main currency (`currency_id`). The selector lists archived
   currencies as well, because choosing one activates it.
2. The actor selects a currency.
3. On save the guards run in this order:
   1. when the company is a branch, the value must equal the root company's value, otherwise the
      write is refused with "The *field label* of a subsidiary must be the same as it's root
      company." where the placeholder is the translated label of the field, "Currency"
      (`MCUR-170`);
   2. when the value differs from the current one and at least one journal item exists for the root
      company or for any company below it, the write is refused with "You cannot change the
      currency of the company since some journal items already exist" (`MCUR-171`).
4. When the write is accepted and the chosen currency is archived, it is activated silently
   (`MCUR-009`).
5. The multi-currency capability is re-evaluated as a consequence of that activation.
6. Every view whose rate column headings are generated from the company currency code is
   re-rendered, because those headings name the two currencies explicitly.

**Alternative entry point.** Selecting a country on the company form proposes that country's
currency (`MCUR-172`). The same guards then apply on save.

## 4.2 Records written

| Record | Fields |
|---|---|
| Company | `currency_id` |
| Currency | `active` set true, when the chosen currency was archived |

## 4.3 Postconditions

Every journal item of that company afterwards expresses its balance (`balance`), its debit
(`debit`) and its credit (`credit`) in the new currency. Every rate row is read against it through
the company rate derivation. From the first journal item on, the choice is frozen
([state-machines.md](state-machines.md) section 12).

---

# 5. Configure the exchange difference journal and accounts

**Actor.** Accounting Manager.

**Preconditions.** A chart of accounts is loaded for the company. A journal whose type is general
exists.

## 5.1 Steps

1. The actor opens the accounting settings and finds the "Default Accounts" block, whose
   "Exchange difference entries:" section is shown only to holders of the multi-currency permission
   group and only to holders of the accounting user group.
2. The actor sets the field labelled "Journal", which is the company's exchange difference journal
   (`currency_exchange_journal_id`). The selector offers only journals whose type is general.
3. The actor sets the field labelled "Gain", which is the company's gain exchange account
   (`income_currency_exchange_account_id`). The selector offers only accounts whose internal group
   is income.
4. The actor sets the field labelled "Loss", which is the company's loss exchange account
   (`expense_currency_exchange_account_id`). The selector offers only accounts whose type is
   expense or other expense.
5. The actor saves.

## 5.2 Records written

Company, fields `currency_exchange_journal_id`, `income_currency_exchange_account_id` and
`expense_currency_exchange_account_id`.

## 5.3 Failure conditions

A chart of accounts template normally pre-fills all three values when it is loaded. When any of the
three is missing, the first reconciliation that needs an exchange difference fails with the message
of `MCUR-102`, `MCUR-103` or `MCUR-104`, and **nothing** of that reconciliation is written: no
partial matching, no entry.

---

# 6. Issue a customer invoice in a foreign currency

**Actor.** Accountant.

**Preconditions.** The foreign currency is active and has at least one rate row, or is knowingly
left at the fallback rate of one (`MCUR-030`). The customer exists. A sale journal exists.

## 6.1 Steps

1. The actor creates a customer invoice. Its document currency (`currency_id`) is initialised from
   the journal's currency when the journal has one, and otherwise from the company's main currency
   (`MCUR-071`).
2. The actor selects the foreign currency. This is possible only while the document is in draft.
3. The document rate (`invoice_currency_rate`) is recomputed to the expected rate
   (`expected_currency_rate`) at the document's rate date, which is the invoice date
   (`invoice_date`) when one is set and today otherwise (`MCUR-081`, `MCUR-087`). The Currency Rate
   field is shown next to the currency to holders of the multi-currency permission group.
4. The actor sets the invoice date. The document rate is recomputed at the new date and every line
   balance is re-derived while every amount in currency is preserved.
5. The actor adds product lines. Unit prices are expressed in the document currency. When a price
   list in that currency applies to the customer, the price comes from the price list; otherwise
   the product's public price, which is expressed in the company currency, is converted into the
   document currency at the pricing date by the conversion of
   [calculations.md](calculations.md) section 8. See
   [../pricing-and-pricelists/README.md](../pricing-and-pricelists/README.md).
6. Taxes are computed on the document currency amounts. Each tax line carries its amount in
   currency (`amount_currency`) in the document currency, and its balance is derived from it:

   ```formula
   balance = round onto the company currency ( amount in currency ÷ document rate )
   ```

7. The payment term lines are generated. Each carries its share of the total in the document
   currency, with its balance derived by the same formula.
8. The actor posts the invoice. Before posting:
   - when the invoice date is empty it is set to today, with the manual-rate protection of
     `MCUR-082`;
   - the document currency must not be archived, otherwise "You cannot validate a document with an
     inactive currency: *the currency code*" (`MCUR-016`);
   - the account currency agreement of `MCUR-064` is checked for every line;
   - the entry must balance in the company currency column (`MCUR-072`).

## 6.2 Records written

| Record | Key field values |
|---|---|
| Journal Entry | `currency_id` set to the foreign currency, `invoice_currency_rate` set to the applied rate, `state` moving from `draft` to `posted`, the totals in both currencies |
| Journal Item, product line | `currency_id` the foreign currency, `amount_currency` the signed line subtotal, `balance` derived by the formula above, the account being the income account |
| Journal Item, tax line | the same currency, `amount_currency` the tax amount, `balance` derived, the account being the tax account |
| Journal Item, payment term line | the same currency, `amount_currency` the total, `balance` derived, the account being the customer's receivable account |

## 6.3 Worked instance

Company currency `USD`. The `EUR` rate on 15 January 2026 is 0.9200 units of `EUR` per one `USD`.
An invoice of 1000.00 `EUR` with no tax:

| Item | Currency | Amount in currency | Balance |
|---|---|---|---|
| Revenue | `EUR` | −1000.00 | −1086.96 |
| Trade receivables | `EUR` | +1000.00 | +1086.96 |

because 1000.00 ÷ 0.9200 = 1086.9565…, which rounds onto one hundredth to 1086.96.

## 6.4 Postconditions

The receivable item carries an open residual of 1000.00 `EUR` and 1086.96 `USD`. The invoice's
total is 1000.00 in the document currency and 1086.96 in the company currency.

---

# 7. Enter a vendor bill in a foreign currency

Identical to procedure 6 with four differences.

1. The document date is mandatory: posting a purchase document without one is refused with "The
   Bill/Refund date is required to validate this document." There is therefore no manual-rate
   protection step, because the date is always known before posting (`MCUR-082`).
2. The signs are mirrored: the expense lines carry a positive amount in currency and a positive
   balance, and the payable line carries negative values.
3. The expense account comes from the product or the product category, and the payable account
   comes from the vendor.
4. The document rate is still the rate from the company currency to the document currency, so the
   same formula derives the balances.

**Worked instance.** A bill of 500.00 `GBP` dated 1 February 2026 when the `GBP` rate is 0.7900:

| Item | Currency | Amount in currency | Balance |
|---|---|---|---|
| Expense | `GBP` | +500.00 | +632.91 |
| Trade payables | `GBP` | −500.00 | −632.91 |

because 500.00 ÷ 0.7900 = 632.9113…, which rounds to 632.91.

---

# 8. Override the rate applied to a document

**Actor.** Accountant.

**Preconditions.** The document is a draft invoice, bill, credit note, debit note or receipt whose
document currency differs from the company's main currency. The actor holds the multi-currency
permission group, without which the field is not displayed.

## 8.1 Steps

1. The actor opens the document and types a new value into the Currency Rate field.
2. The value must be strictly positive, otherwise "The currency rate must be strictly positive." is
   raised and the previous value is restored (`MCUR-080`).
3. Every base line and every tax line keeps its amount in currency and has its balance re-derived
   from the new rate (`MCUR-083`). When the company's tax rounding method rounds per tax, the
   re-derivation is performed on the aggregated tax group, so that the company currency column
   still balances exactly; the worked case is in [calculations.md](calculations.md) section 10.6.
4. The actor saves and posts. The typed value is what posting uses, and it is preserved
   (`MCUR-082`).
5. To discard the override the actor runs the rate refresh operation, which sets the document rate
   back to the expected rate at the document's rate date (`MCUR-085`).

## 8.2 Worked instance

A draft customer invoice whose expected rate is 2.0. The actor types 5.0. The invoice holds a
product line of 2000.00 in the document currency with a fifteen percent tax:

| Item | Amount in currency | Balance before | Balance after |
|---|---|---|---|
| Revenue | −2000.00 | −1000.00 | −400.00 |
| Tax | −300.00 | −150.00 | −60.00 |
| Trade receivables | +2300.00 | +1150.00 | +460.00 |

## 8.3 Postconditions

The document carries a document rate of 5.0 and an expected rate of 2.0. Posting changes neither. A
duplicate of the document does not carry the override: the copy recomputes the expected rate at its
own rate date (`MCUR-084`).

---

# 9. Register a payment in a currency other than the company currency

**Actor.** Accountant.

**Preconditions.** One or more posted documents with an open residual. A bank or cash journal
exists.

## 9.1 Steps

1. The actor selects the documents and runs the payment registration. The registration screen
   opens.
2. The screen derives its currency: the journal's currency when the journal has one, otherwise the
   currency of the selected documents when they agree, otherwise the company's main currency.
3. The screen derives the amount to pay by converting the residuals of the selected items into the
   screen's currency at the payment date, by the algorithm of
   [calculations.md](calculations.md) section 22.
4. **Branch A — the actor changes the payment date.** The residuals are converted again at the new
   date and the amount is refreshed, unless the actor had typed a custom amount, which is kept.
5. **Branch B — the actor changes the payment currency.** A custom amount that had been typed is
   converted from the previously selected currency into the new one at the payment date, so that
   the actor does not retype it. Otherwise the amount is derived again by step 3.
6. **Branch C — the actor types an amount.** The typed amount and the currency it was typed in are
   remembered as a custom amount. A later change of date leaves it untouched; a later change of
   currency converts it.
7. The actor confirms. A Payment is created with the chosen currency (`currency_id`) and amount
   (`amount`), the amount always being positive and the direction carried by the payment type.
8. The payment's journal entry is generated:

   ```formula
   liquidity amount in currency   = + payment amount            for an inbound payment
   liquidity amount in currency   = − payment amount            for an outbound payment
   liquidity balance              = convert ( liquidity amount in currency , from the payment currency to the company currency , for the company , at the payment date )
   counterpart amount in currency = − liquidity amount in currency
   counterpart balance            = − liquidity balance
   ```

9. The payment's counterpart item is matched against the selected documents' open items by
   procedure 10.

## 9.2 Records written

| Record | Key field values |
|---|---|
| Payment | `currency_id`, `amount`, `date`, the payment type, and the derived signed amount in the company currency (`amount_company_currency_signed`) |
| Journal Entry | the payment entry, with `currency_id` set to the payment currency |
| Journal Item, liquidity | `currency_id` the payment currency, `amount_currency` the signed amount, `balance` the converted amount, the account being the outstanding account or the bank account |
| Journal Item, counterpart | `currency_id` the payment currency, `amount_currency` the negated signed amount, `balance` the negated converted amount, the account being the receivable or payable account |

## 9.3 Worked instances

**One currency.** Company currency `USD`, the `EUR` rate on 10 March 2026 being 0.9500. An inbound
payment of 1000.00 `EUR`:

| Item | Currency | Amount in currency | Balance |
|---|---|---|---|
| Bank | `EUR` | +1000.00 | +1052.63 |
| Trade receivables | `EUR` | −1000.00 | −1052.63 |

because 1000.00 ÷ 0.9500 = 1052.6315…, which rounds to 1052.63.

**Mixed currencies.** Two open invoices are selected, one of 1000.00 `EUR` and one of 500.00 `USD`.
The payment is made in `EUR` on a date whose `EUR` rate is 0.9500. The proposed amount is

```formula
proposed amount = 1000.00 + round onto the euro ( 500.00 × 0.9500 ) = 1000.00 + 475.00 = 1475.00 euro
```

## 9.4 Postconditions

The payment is in the in-process state until its liquidity item is matched
([state-machines.md](state-machines.md) section 9). Its counterpart item offers a residual in both
columns, ready for procedure 10.

---

# 10. Reconcile foreign currency items and produce the exchange difference

**Actor.** Accountant, or the system when a payment is registered against selected documents.

**Preconditions.** Two or more posted journal items on the same reconcilable account and the same
root company, none of them already fully reconciled, none belonging to a cancelled entry.

## 10.1 Steps

1. The batch is validated against `MCUR-130` to `MCUR-134`. Any violation abandons the whole
   reconciliation with the corresponding message and writes nothing.
2. The batch is decomposed into a plan: the items are ordered by `MCUR-136` and split by currency by
   `MCUR-135`. Each currency group is matched on its own first; whatever remains open is then
   matched across groups.
3. The residual amounts of every item are read once and kept as running values, so that the whole
   batch can be written in one pass.
4. The debit items and the credit items are separated. An item counts as a debit when its balance
   is positive **or** its amount in currency is positive, and as a credit when either is negative.
   An item with a zero balance and a non-zero amount in currency therefore still takes part.
5. The matching loop takes the next available debit item and the next available credit item and
   computes one partial matching by the algorithm of [calculations.md](calculations.md) section 13,
   together with any exchange difference instruction by section 14. The side that is fully consumed
   is dropped and the loop takes the next one. The loop ends when either list is exhausted.
6. Every partial matching of the batch is created in one operation. Each carries the matched amount
   (`amount`), the matched amount in the debit currency (`debit_amount_currency`), the matched
   amount in the credit currency (`credit_amount_currency`), the debit item (`debit_move_id`), the
   credit item (`credit_move_id`), and the derived company (`company_id`), latest matched date
   (`max_date`), debit currency (`debit_currency_id`) and credit currency
   (`credit_currency_id`).
7. Every exchange difference entry of the batch is created in one operation by procedure 11, and
   each is then linked to one partial matching through the matching's exchange difference entry
   field (`exchange_move_id`), by `MCUR-110`.
8. When the company uses cash basis taxes and the account is a receivable or a payable account, the
   cash basis entries of the batch are created. They may themselves trigger further exchange
   differences on the tax transition account.
9. The connected components of the matching graph are recomputed and each item's matching number
   (`matching_number`) is updated: the identifier of the Full Reconciliation when the group closed,
   otherwise the letter `P` followed by the smallest identifier among the partial matchings of the
   group.
10. For each connected group, the closure test of [calculations.md](calculations.md) section 19
    decides whether a Full Reconciliation record is created covering every item and every partial
    matching of the group.
11. Documents whose payment state moved to paid or in payment are notified through the paid hook,
    which posts a message in the document's discussion thread and triggers any follow-up
    automation.

## 10.2 Failure conditions

| Condition | Message |
|---|---|
| An item is already fully reconciled | "You are trying to reconcile some entries that are already reconciled." |
| An item belongs to a cancelled entry | "You can not reconcile cancelled entries." |
| The items are not all on one account | "Entries are not from the same account: *the comma-separated account display names*" |
| The items do not all belong to one root company | "Entries don't belong to the same company: *the comma-separated company display names*" |
| The account allows neither reconciliation nor cash handling | "Account *the account display name* does not allow reconciliation. First change the configuration of this account to allow it." |
| An exchange difference is needed and the exchange journal, the loss account or the gain account is missing | The message of `MCUR-102`, `MCUR-103` or `MCUR-104`; nothing at all is written |

## 10.3 Postconditions

Every matched item's residual amount and residual amount in currency are reduced by the matched
amounts; the reconciled flag becomes true for the items that closed; one exchange difference entry
exists per rate movement absorbed.

## 10.4 Worked instance

The invoice of procedure 6 matched against the payment of procedure 9.

| Step | Result |
|---|---|
| Reconciliation currency | `EUR`, because both items offer a residual in it |
| Matched amount in the two document currencies | 1000.00 `EUR` on both sides |
| Matched amount in the company currency | 1052.63 `USD` |
| Exchange instruction | On the invoice's receivable item, company currency column, +34.33 |
| Exchange entry | Credit the receivable account 34.33, debit the loss account 34.33, both lines carrying `EUR` with an amount in currency of zero, dated by `MCUR-106` |
| Second partial matching | Matched amount 34.33, both document currency amounts zero, between the invoice's receivable item and the correction line |
| Final residuals | Invoice receivable 0.00 and 0.00; payment receivable 0.00 and 0.00 |
| Full Reconciliation | Created, covering four items and two partial matchings |

---

# 11. Create an exchange difference entry

**Actor.** The reconciliation engine. Never invoked directly by a user.

**Preconditions.** At least one exchange difference instruction has been produced by
[calculations.md](calculations.md) section 14.

## 11.1 Steps

1. The company is determined: the company of the invoice, bill, credit note, debit note or receipt
   among the entries involved when there is one, otherwise the company of the items involved. When
   no company can be determined, nothing is produced.
2. The exchange journal is read from that company. The candidate entry date is computed by
   `MCUR-106` and [calculations.md](calculations.md) section 16.
3. The entry header is prepared with the document type set to a miscellaneous entry, the entry
   number (`name`) set to `/` so that no number is taken from the sequence before posting, the
   accounting date set to the candidate date, the journal set to the exchange journal, and the
   always-tax-exigible flag set.
4. For each instruction, in the order the instructions were produced, two working values are
   derived: a correction balance in the company currency and a correction amount in currency in the
   corrected item's document currency. They are working values of this step alone and are not
   fields of any entity; they must not be confused with the residual amount and the residual amount
   in currency of a journal item.
   1. When the instruction names the company currency column, the correction balance is its amount,
      and the correction amount in currency is that same amount when the corrected item's currency
      is the company currency and zero otherwise. When the correction balance is zero at the
      company currency's precision, the instruction is skipped.
   2. When the instruction names the document currency column, the correction balance is zero and
      the correction amount in currency is its amount. When that is zero at the item currency's
      precision, the instruction is skipped.
   3. The entry date is raised to the corrected item's accounting date, whether or not the
      instruction was skipped (`MCUR-117`).
   4. The gain or the loss account is chosen from the sign of the instruction amount (`MCUR-105`).
   5. Two lines are appended:

| Line | Account | Debit | Credit | Amount in currency | Currency | Other values |
|---|---|---|---|---|---|---|
| Correction | The corrected item's account | The negated correction balance when it is negative, otherwise zero | The correction balance when it is positive, otherwise zero | The negated correction amount in currency | The corrected item's currency | The counterparty copied from the corrected item; marked as reconciling against that item; the label "Currency exchange rate difference" |
| Counterpart | The gain or the loss account | The correction balance when it is positive, otherwise zero | The negated correction balance when it is negative, otherwise zero | The correction amount in currency | The corrected item's currency | The counterparty copied from the corrected item; the label "Currency exchange rate difference"; the analytic distribution supplied by the caller, when one was supplied |

5. The configuration is verified for every journal involved in the batch: the journal must be set
   (`MCUR-102`), and its company must have both a loss account (`MCUR-103`) and a gain account
   (`MCUR-104`). A missing value abandons the whole reconciliation.
6. The entries are created with the "no further exchange difference" flag set, which is what
   prevents the matching of a correction line from producing a correction of its own
   (`MCUR-101`).
7. Each entry whose two matched items both belong to posted entries is posted immediately, without
   the soft posting delay. The others stay in draft.
8. The correction line of each entry is matched against the item it corrects, producing the second
   partial matching of the pair, in exchange-line mode (`MCUR-113`).

## 11.2 Postconditions

The corrected item's company currency residual, or its document currency residual, is nil. The
generated entry is itemised in [accounting-effects.md](accounting-effects.md) section 5.

---

# 12. Undo a reconciliation

**Actor.** Accountant.

**Preconditions.** At least one partial matching exists on the selected items.

## 12.1 Steps

1. The actor selects one or more journal items and runs the unreconcile operation, or opens the
   matching and removes it. The set of partial matchings to delete is the union of the matchings in
   which the selected items are the debit side and those in which they are the credit side. When
   the operation is run from a list, the set is expanded to every item connected through the
   matching graph, so that a group is undone as a whole.
2. Payments in the paid state whose signed amount corresponds to a matching being deleted are
   collected, to be returned to the in-process state afterwards (`MCUR-138`).
3. The cash basis entries generated by those matchings are collected, and so are the exchange
   difference entries linked through the matchings' exchange difference entry field.
4. The partial matchings are deleted first. This is what breaks the recursion: deleting a Full
   Reconciliation would otherwise delete matchings, which would try to delete the Full
   Reconciliation again.
5. The Full Reconciliation records that covered the deleted matchings are deleted.
6. The collected entries are processed:
   - an entry that is not in draft is reversed by the reversal operation in cancelling mode, with
     its accounting date set by [calculations.md](calculations.md) section 17 and its internal
     reference (`ref`) set to "Reversal of: *the entry number of the reversed entry*"; the reversal
     is itself matched against the original, so that both close and neither shows as an open item;
   - an entry that is still in draft is deleted outright.
7. The matching numbers of the items that remain connected are recomputed.
8. The collected payments are returned to the in-process state.

## 12.2 Postconditions

The items regain their residuals. The exchange difference entry and its reversal both remain in the
ledger with a net effect of zero, which preserves the audit trail. The item that was corrected shows
both the original correction line and its reversal among its matched lines, the original matched and
the reversal not.

## 12.3 Worked instance

Undoing the reconciliation of procedure 10 produces, for the exchange entry dated 31 March 2026
whose lines are a receivable credit of 34.33 and a loss debit of 34.33, a reversal dated 31 March
2026 whose lines are a receivable debit of 34.33 and a loss credit of 34.33. The invoice's
receivable item returns to a residual amount of 1086.96 and a residual amount in currency of
1000.00.

---

# 13. Record a bank transaction in a foreign currency

**Actor.** Accountant, or the bank feed import.

**Preconditions.** A bank journal exists with a default account and a suspense account.

## 13.1 Steps

1. A bank statement line is created with its date, its payment reference, its counterparty when
   known, and its amount (`amount`) expressed in the bank account currency. The journal currency
   (`currency_id`) is derived from the journal, and is the company's main currency when the journal
   has none (`MCUR-067`).
2. **Branch A — the transaction was executed in the bank account currency.** The transacted
   currency (`foreign_currency_id`) is left empty and the amount in currency stays zero.
3. **Branch B — the transaction was executed in another currency.** The actor selects the
   transacted currency and enters the amount in currency. When the actor leaves the amount in
   currency empty it is derived by converting the amount at the line's date (`MCUR-154`); an amount
   already entered is never overwritten. The consistency guards `MCUR-150` to `MCUR-152` run on
   save, and the silent drop of `MCUR-153` applies when a feed supplies a redundant transacted
   currency at creation.
4. The journal entry of the statement line is generated by
   [calculations.md](calculations.md) section 21, producing a liquidity line in the bank account
   currency and a counterpart line in the transacted currency, both valued in the company currency
   from the bank account currency amount (`MCUR-156`).
5. The counterpart line lands on the journal's suspense account until the transaction is matched.
   Without a suspense account and without an explicit counterpart account the generation is refused
   with the message of `MCUR-155`.

## 13.2 Worked instance, three currencies

Company currency `USD`, bank account in `EUR`, transaction executed in `GBP`. The bank reports
850.00 `EUR` in, stated as 730.00 `GBP`. The `EUR` rate on the transaction date is 0.9200, so
850.00 ÷ 0.9200 = 923.9130…, which rounds to 923.91.

| Item | Account | Currency | Amount in currency | Debit | Credit |
|---|---|---|---|---|---|
| Liquidity | Bank | `EUR` | +850.00 | 923.91 | 0.00 |
| Counterpart | Suspense | `GBP` | −730.00 | 0.00 | 923.91 |

## 13.3 Postconditions

The transaction appears in the reconciliation screen showing both the transacted amount and its
company currency equivalent. The amount in currency column of the entry does not balance and is not
required to (`MCUR-072`).

---

# 14. Match a bank transaction against a foreign currency document

**Actor.** Accountant.

**Preconditions.** A posted bank statement line with an open suspense counterpart, and one or more
open items on a receivable or payable account.

## 14.1 Steps

1. The actor opens the reconciliation screen for the statement line. Both the transacted amount and
   its company currency equivalent are displayed.
2. The actor selects the open items to settle.
3. For each selected item the counterpart amounts are derived from the transaction's own implied
   rates by [calculations.md](calculations.md) section 21.4, and not from the rate table
   (`MCUR-157`). Three branches apply, depending on whether the item's currency is the transacted
   currency, the bank account currency, or a third currency.
4. The suspense line is replaced by lines on the selected items' accounts carrying the derived
   amounts.
5. The generated lines are matched against the selected items by procedure 10. Because the
   counterpart amounts were derived from the transaction's own rates, the transaction side closes
   exactly, and any difference between the transaction's rate and the document's rate lands on the
   document side, where it produces the exchange difference.

## 14.2 Postconditions

The statement line is fully reconciled; the settled documents move to the paid or partially paid
payment state; one exchange difference entry exists per rate movement. The worked continuation of
procedure 13 is in [accounting-effects.md](accounting-effects.md) section 7.4.

---

# 15. Change the rounding factor of a currency

**Actor.** Settings Administrator holding the technical features group.

**Preconditions.** The actor understands that coarsening the factor is refused once the currency
has been used in accounting, and that refining it does not restate anything already stored.

## 15.1 Steps

1. The actor opens the currency form. The "Price Accuracy" group, which holds the rounding factor
   and the decimal places, is shown only to holders of the technical features group.
2. The actor types a new rounding factor. The decimal places are recomputed immediately and
   displayed, and the irreversibility panel of `MCUR-015` appears, reading "WARNING - This change is
   irreversible" followed by "You are changing decimals in your entire database, including
   invoices, tax amounts, accounting amounts, reports. This is probably not intended."
3. The actor saves.
   - **Branch A — refining the precision**, meaning a smaller rounding factor, for example from one
     hundredth to one thousandth: accepted, whether or not the currency has been used. Amounts
     already stored remain exactly representable on the finer grid.
   - **Branch B — coarsening the precision** while the currency has never been used in accounting:
     accepted.
   - **Branch C — coarsening the precision** while the currency has been used: refused with "You
     cannot reduce the number of decimal places of a currency which has already been used to make
     accounting entries." Nothing is written (`MCUR-006`).
4. In every branch the positivity check still applies: a zero or negative factor is refused with
   "The rounding factor must be greater than 0!" (`MCUR-003`).

## 15.2 Postconditions

Every later rounding, comparison, zero test, storage and rendering of an amount in that currency
uses the new factor. Amounts already stored are not restated.

---

# 16. Deactivate a currency

**Actor.** Accounting Manager.

## 16.1 Steps

1. The actor switches the activity toggle off, from the currency list or from the form.
2. The guard of `MCUR-007` runs. When any company uses the currency as its main currency the write
   is refused with "This currency is set on a company and therefore cannot be deactivated."
3. When the write is accepted:
   - every price list expressed in that currency is archived (`MCUR-010`);
   - the multi-currency capability is re-evaluated and is revoked when only one currency remains
     active (`MCUR-008`);
   - the cached catalogue of active currencies is invalidated (`MCUR-014`).
4. Draft documents already expressed in the currency remain, but can no longer be posted
   (`MCUR-016`). Posted documents and their valuations are unaffected.

## 16.2 Postconditions

The currency disappears from selection lists and from default queries. Every historical record that
references it still reads correctly, because an archived currency is still readable.

---

# 17. Convert an amount between two foreign currencies

**Actor.** Any component that needs a monetary amount expressed in another currency: a price list, a
report, a spreadsheet formula, a payment registration screen, a bank transaction.

**Preconditions.** Both currencies exist. A company and a date are known, or their defaults apply:
the company the caller is working in, and today in the caller's time zone.

## 17.1 Steps

1. When the amount is exactly zero, zero is returned and no rate is read (`MCUR-048`).
2. When the two currencies are the same record, the factor is one and no rate is read; the amount is
   still rounded onto that currency's rounding factor.
3. When one of the two currencies is empty, the missing one is taken to be the given one, which
   makes the conversion the identity. When both are empty the operation fails, because an amount
   cannot be converted from an unknown currency.
4. The rate in force is read for each currency, for the company's root and for the date, by the
   lookup of [calculations.md](calculations.md) section 7: the latest row dated on or before the
   date, a company-scoped row beating a shared row unconditionally, falling back to the earliest
   row of the currency and then to one.
5. The two rates are composed into one factor and the amount is multiplied by it **once**:

   ```formula
   converted amount = round onto the target currency ( amount × ( rate of the target currency ÷ rate of the source currency ) )
   ```

6. The single rounding uses the **target** currency's rounding factor. A caller that will round
   later may ask for the rounding to be suppressed, in which case the raw product is returned.

## 17.2 Failure conditions

Neither currency given: the operation fails rather than guessing. No other failure exists: a
currency with no rate row resolves to a rate of one (`MCUR-030`), and a date before the first row
resolves to the earliest row (`MCUR-029`).

## 17.3 Postconditions

Nothing is written. The result is a number in the target currency, exact to that currency's
rounding factor. A round trip through the other currency is not guaranteed to return the starting
amount (`MCUR-045`), and no stored amount may be re-derived that way.

---

# 18. Close a period across companies whose main currencies differ

**Actor.** Accountant or auditor preparing a consolidated report.

**Preconditions.** More than one company is selected and their main currencies are not all the
same. A reporting period, or a list of periods, is known.

## 18.1 Steps

1. The reporting layer asks whether the selected companies share one main currency. When they do, a
   synthetic table is used in which every factor is one, and no temporary storage is created
   ([calculations.md](calculations.md) section 20.1).
2. Otherwise a rate table is built for the duration of the report, with one row per company, per
   period and per rate type. Its columns are the company, the period key, the date the row is valid
   from, the date of the next change, the rate type and the factor.
3. The rows of the reporting company itself, and of every company whose main currency is the
   reporting currency, carry a factor of one for every rate type and every period.
4. For every other company:
   - the **current** factor is computed at the period's end date;
   - the **historical** factors are produced as a series of rows, one per rate change, each valid
     from its own date until the next change;
   - the **average** factor is the day-weighted mean of the factors in force across the period.
   The three formulas, with worked numbers, are in [calculations.md](calculations.md) sections 20.3,
   20.4 and 20.5.
5. Every amount produced by a company is multiplied by the factor the table gives for that company,
   that period and the rate type the report asks for: the current factor for balance sheet
   positions, the historical factor for balance sheet positions that must keep their
   transaction-date valuation, and the average factor for profit and loss positions.
6. The table lives only for the duration of the report and is discarded afterwards.

## 18.2 Records written

None. Consolidation produces no journal entry
([accounting-effects.md](accounting-effects.md) section 11).

## 18.3 Postconditions

The report is expressed in the reporting company's main currency. The difference between
translating a balance sheet at the closing factor and translating the corresponding result at the
average factor accumulates into a cumulative translation adjustment, which is an equity position
presented by the reporting layer and by the fiscal localization that prescribes it; this domain
supplies only the three rate types.

---

# 19. Recognise unrealised gains and losses at a reporting date

**Actor.** Accountant. **Industry-standard default**, as recorded in
[business-rules.md](business-rules.md) `MCUR-200`: the platform recognises realised differences
automatically at matching time and offers this report-driven adjustment for the unrealised ones.

**Preconditions.** Open items exist on balance sheet accounts whose journal items carry a foreign
currency. A rate is known for the reporting date, or the accountant supplies one.

## 19.1 Steps

1. The accountant opens the unrealised gain and loss report and chooses a reporting date. The
   report lists every open item on every balance sheet account whose items carry a foreign
   currency, grouped by account and by currency, showing for each group the balance in the foreign
   currency, the book value in the company currency, the value at the reporting date's rate, and
   the adjustment needed to move from the former to the latter.
2. The accountant may substitute a manual rate per currency for the reporting date instead of the
   rate table's rate. A banner then offers to reset to the rate table's value.
3. The accountant runs the adjustment operation and supplies a journal, an expense account, an
   income account, the reporting date and a reversal date.
4. One entry is posted at the reporting date, debiting or crediting each account for its adjustment
   amount, with the opposite side on the expense account when the total adjustment is a loss and on
   the income account when it is a gain, and with an amount in currency of zero on every line so
   that the foreign currency position is untouched.
5. One reversing entry is posted at the reversal date, by convention the first day of the following
   period.
6. After posting, the adjustment column of the report reads zero for every group.

## 19.2 Worked instance

At 30 June 2026 a customer still owes 1000.00 `EUR`, booked at 1086.96 `USD` when the rate was
0.9200. The rate at 30 June 2026 is 0.9400, so the value at that date is 1000.00 ÷ 0.9400 =
1063.8297…, which rounds to 1063.83. The adjustment is 1063.83 − 1086.96 = −23.13, an unrealised
loss. The two entries are itemised in [accounting-effects.md](accounting-effects.md) section 10.

## 19.3 Postconditions

The reporting period carries the revaluation; the following period does not, because the reversal
removes it. The later realised difference computed at settlement is therefore measured against the
original book value and is not double counted.

---

# 20. Where the states of these procedures are specified

| Procedure | State machine |
|---|---|
| 1, 16 | [state-machines.md](state-machines.md) section 1, the activity state of a currency |
| 1, 16 | [state-machines.md](state-machines.md) section 3, the multi-currency capability state |
| 2, 3 | [state-machines.md](state-machines.md) section 4, the lifecycle of a rate row |
| 6, 7, 8 | [state-machines.md](state-machines.md) section 5, the currency and rate state of a document |
| 10, 11, 12 | [state-machines.md](state-machines.md) section 6, the state of an exchange difference entry |
| 10, 12, 14 | [state-machines.md](state-machines.md) sections 7 and 8, the reconciliation state and the matching number of a journal item |
| 9, 10, 12 | [state-machines.md](state-machines.md) sections 9 and 10, the settlement state of a payment and the payment state of a document |
| 13, 14 | [state-machines.md](state-machines.md) section 11, the currency configuration state of a bank transaction |
| 4 | [state-machines.md](state-machines.md) section 12, the main currency state of a company |
| 15 | [state-machines.md](state-machines.md) section 2, the precision state of a currency |

---

# 21. Reconciliation notes

1. **The state tables.** One draft ended this file with a section of state tables. Those tables are
   now in [state-machines.md](state-machines.md), expanded with stored values, labels, meanings and
   exact refusal messages; section 20 above maps each procedure onto the machine it moves. Nothing
   of the original tables was dropped.
2. **Field identifiers.** One draft named fields by a canonical full name, the other by the stored
   identifier. Every field named in this file now carries the stored identifier in code font with
   its full name in words, matching [entities.md](entities.md).
3. **Section references into the arithmetic.** The two drafts numbered their calculation sections
   differently. Every reference in this file points at the consolidated numbering of
   [calculations.md](calculations.md).
4. **Procedures that neither draft carried.** Converting between two foreign currencies (procedure
   17), closing a period across companies whose main currencies differ (procedure 18) and posting
   the unrealised revaluation (procedure 19) were specified as arithmetic and as rules but never as
   procedures. They are written out here, because each is an operation a user performs and each has
   its own preconditions and failure conditions.
5. **The three-currency bank transaction.** Both drafts carried the same worked instance with the
   same numbers, one in its workflow document and one in its ledger-effects document. It is kept in
   both, because procedure 13 needs it to show the sequence and
   [accounting-effects.md](accounting-effects.md) section 7.3 needs it to show the posting.
