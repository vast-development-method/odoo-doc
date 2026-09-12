# Multi-Currency — Acceptance Criteria

Numbered Given, When and Then scenarios that a replacement implementation must pass. Each is
independently verifiable and carries concrete records, concrete inputs and exact resulting values.
Scenario identifiers have the form `MCUR-AC-nnn` and are stable. The rule of
[business-rules.md](business-rules.md), or the section of another file of this folder, that each
scenario exercises is named at the end of the scenario.

**Messages** are reproduced between quotation marks, exactly as they are shown; a placeholder
inside a message is described in words and written in italics. **Identifiers** — field identifiers
and stored selection values — are written in code font.

## Field identifiers used in these scenarios

Every identifier below is the one under which the value is stored and transported. The full name in
words is given here once; the scenarios then use the identifier alone.

| Entity | Identifier | Full name |
|---|---|---|
| Currency | `name` | currency code |
| Currency | `full_name` | full name |
| Currency | `symbol` | symbol |
| Currency | `position` | symbol position |
| Currency | `rounding` | rounding factor |
| Currency | `decimal_places` | decimal places |
| Currency | `active` | activity flag |
| Currency | `iso_numeric` | international standard numeric code |
| Currency | `currency_unit_label`, `currency_subunit_label` | currency unit label, currency subunit label |
| Currency | `rate`, `inverse_rate`, `rate_string`, `date` | current rate, inverse rate, rate as text, last rate date |
| Currency | `is_current_company_currency`, `display_rounding_warning` | is current company currency, display rounding warning |
| Currency Rate | `name` | rate date |
| Currency Rate | `rate` | technical rate |
| Currency Rate | `company_rate`, `inverse_company_rate` | company rate, inverse company rate |
| Currency Rate | `currency_id`, `company_id` | currency, company |
| Company | `currency_id` | main currency |
| Company | `currency_exchange_journal_id` | exchange difference journal |
| Company | `income_currency_exchange_account_id`, `expense_currency_exchange_account_id` | gain exchange account, loss exchange account |
| Journal Entry | `currency_id`, `company_currency_id` | document currency, company currency |
| Journal Entry | `invoice_currency_rate`, `expected_currency_rate` | document rate, expected rate |
| Journal Entry | `invoice_date`, `state`, `payment_state`, `ref` | invoice date, posting state, payment state, internal reference |
| Journal Entry | `amount_total`, `amount_total_signed` | total amount, signed total amount |
| Journal Item | `currency_id`, `company_currency_id` | item currency, company currency |
| Journal Item | `balance`, `debit`, `credit`, `amount_currency` | balance, debit, credit, amount in currency |
| Journal Item | `amount_residual`, `amount_residual_currency` | residual amount, residual amount in currency |
| Journal Item | `reconciled`, `matching_number`, `currency_rate` | is reconciled, matching number, item rate |
| Partial Reconciliation | `amount`, `debit_amount_currency`, `credit_amount_currency` | matched amount, matched amount in the debit currency, matched amount in the credit currency |
| Partial Reconciliation | `exchange_move_id`, `full_reconcile_id` | exchange difference entry, full reconciliation |
| Journal | `currency_id` | journal currency |
| Account | `currency_id` | account currency |
| Bank Statement Line | `currency_id`, `foreign_currency_id`, `amount`, `amount_currency` | journal currency, transacted currency, amount, amount in currency |
| Payment | `currency_id`, `amount`, `state` | payment currency, amount, settlement state |

## The baseline

Unless a scenario says otherwise, the following applies.

| Element | Value |
|---|---|
| Company | "Main", whose main currency is the United States dollar, code `USD`, rounding factor one hundredth, two decimal places |
| Test currency | "Foreign", code `FRG`, rounding factor one thousandth, three decimal places |
| Rate rows of `FRG` | 1 January 1900 at a technical rate of 1.0; 1 January 2016 at 3.0; 1 January 2017 at 2.0 |
| Exchange journal | "Exchange Difference", of type general |
| Gain account | "Gain on exchange rate", an income account |
| Loss account | "Loss on exchange rate", an expense account |

Currency codes used in the scenarios, with their full names: `USD` the United States dollar, `EUR`
the euro, `GBP` the pound sterling, `JPY` the Japanese yen, `FRG` the fictitious currency "Foreign"
of the baseline, and `AAA` and `ZZZ`, two fictitious currencies used only in the ordering
scenarios.

---

## A. Currency master data

### MCUR-AC-001: a duplicate currency code is refused

**Given** a Currency exists with `name = "FRG"`.
**When** a user creates a second Currency with `name = "FRG"`.
**Then** the create is refused with the message "The currency code must be unique!" and no second record exists.
(`MCUR-001`)

### MCUR-AC-002: a zero rounding step is refused

**Given** a user is creating a Currency.
**When** the user supplies `rounding = 0`.
**Then** the create is refused with the message "The rounding factor must be greater than 0!"
(`MCUR-003`)

### MCUR-AC-003: decimal places follow the rounding step

**Given** a Currency.
**When** `rounding` is set to each of `0.01`, `0.001`, `0.0001`, `0.05`, `0.5`, `1`.
**Then** `decimal_places` becomes `2`, `3`, `4`, `2`, `1`, `0` respectively.
(`MCUR-005`; [calculations.md](calculations.md) section 2)

### MCUR-AC-004: the precision of a currency in use may not be lowered

**Given** the currency `FRG` has `rounding = 0.001` and at least one Journal Item exists whose `currency_id` is `FRG`.
**When** a user writes `rounding = 0.01` on `FRG`.
**Then** the write is refused with the message "You cannot reduce the number of decimal places of a currency which has already been used to make accounting entries." and `rounding` remains `0.001`.
(`MCUR-006`)

### MCUR-AC-005: the precision of a currency in use may be raised

**Given** the same starting point as `MCUR-AC-004`.
**When** a user writes `rounding = 0.0001` on `FRG`.
**Then** the write is accepted and `decimal_places` becomes `4`.
(`MCUR-006`)

### MCUR-AC-006: a currency used by a company may not be deactivated

**Given** company "Main" has `currency_id` equal to the United States dollar.
**When** a user writes `active = false` on the United States dollar.
**Then** the write is refused with the message "This currency is set on a company and therefore cannot be deactivated."
(`MCUR-007`)

### MCUR-AC-007: activating a second currency grants the multi-currency capability

**Given** exactly one currency is active and the internal user group does not hold the multi-currency access group.
**When** a user activates a second currency.
**Then** the internal user group holds the multi-currency access group, and it also holds the price list capability group, and every company has at least one price list.
(`MCUR-008`)

### MCUR-AC-008: deactivating back to one currency revokes the capability

**Given** two currencies are active and the internal user group holds the multi-currency access group.
**When** a user deactivates one of them, and it is not the main currency of any company.
**Then** the internal user group no longer holds the multi-currency access group.
(`MCUR-008`)

### MCUR-AC-009: deactivating a currency archives its price lists

**Given** the currency `FRG` is active, is not the main currency of any company, and two Price List records are expressed in it.
**When** a user writes `active = false` on `FRG`.
**Then** both Price List records have `active = false`.
(`MCUR-010`)

### MCUR-AC-010: setting a company currency activates it

**Given** the currency `FRG` has `active = false` and no journal item exists for company "Main".
**When** a user sets company "Main"'s `currency_id` to `FRG`.
**Then** the write succeeds and `FRG` has `active = true`.
(`MCUR-009`)

### MCUR-AC-011: deleting a currency deletes its rate rows

**Given** the currency `FRG` has three rate rows and is referenced by no other record.
**When** a user deletes `FRG`.
**Then** the three rate rows no longer exist.
(`MCUR-011`)

### MCUR-AC-012: currencies are ordered active first, then by code

**Given** the active currencies are `USD` and `FRG`, and the archived currencies are `AAA` and `ZZZ`.
**When** the currency catalogue is listed with archived records included and no explicit ordering.
**Then** the order is `FRG`, `USD`, `AAA`, `ZZZ`.
(`MCUR-012`)

### MCUR-AC-013: a draft document in an archived currency cannot be posted

**Given** a draft customer invoice whose `currency_id` is `FRG`, and `FRG` has just been archived.
**When** a user posts the invoice.
**Then** the post is refused with the message "You cannot validate a document with an inactive currency: FRG" and the invoice stays in draft.
(`MCUR-016`)

### MCUR-AC-014: the rounding warning appears only for a saved currency with a changed step

**Given** a saved Currency whose `rounding` is `0.01`.
**When** a user types `0.001` into `rounding` without saving.
**Then** `display_rounding_warning` is true and the irreversibility panel is shown.
**And when** the same value `0.01` is retyped.
**Then** `display_rounding_warning` is false.
(`MCUR-015`)

### MCUR-AC-015: a currency without a code, or with a code longer than three characters, is refused

**Given** a user is creating a Currency with `symbol = "Fr"` and `rounding = 0.01`.
**When** the user saves it with the currency code (`name`) left empty.
**Then** the create is refused by the required-field check and no record exists.
**And when** the user saves it with `name = "EURO"`, four characters long.
**Then** the create is refused, because the stored code admits at most three characters, and no record exists.
**And when** the user saves it with `name = "FRG"`.
**Then** the record is created and the currency code reads `FRG`.
(`MCUR-002`)

### MCUR-AC-016: a currency without a symbol is refused

**Given** a user is creating a Currency with `name = "FRG"` and `rounding = 0.01`.
**When** the user saves it with `symbol` left empty.
**Then** the create is refused by the required-field check and no record exists.
**And when** the user supplies `symbol = "Fr"` and saves.
**Then** the record is created and every amount printed in that currency carries `Fr`.
(`MCUR-004`)

### MCUR-AC-017: the decimal place count cannot be set by hand

**Given** a Currency whose `rounding` is `0.01` and whose `decimal_places` is therefore `2`.
**When** a user holding the technical features group writes `decimal_places = 5` without changing `rounding`.
**Then** the next read of `decimal_places` is `2` again, because the value is re-derived from `rounding`.
**And when** `rounding` is changed to `0.001`.
**Then** `decimal_places` becomes `3` without any further action.
**And given** a user who does not hold the technical features group.
**When** the currency form is rendered for that user.
**Then** neither `rounding` nor `decimal_places` is shown.
(`MCUR-005`)

### MCUR-AC-018: the currency catalogue is the same for every company

**Given** two companies "Foo" and "Bar", and a Currency `FRG`.
**When** a user working in "Foo" lists the currencies, then switches to "Bar" and lists them again.
**Then** both lists contain exactly the same Currency records, `FRG` included, because a Currency carries no company and no company filter is applied to it.
**And when** the user, working in "Bar", changes `FRG`'s `full_name`.
**Then** the new name is what the user working in "Foo" reads.
**And** only rate rows, not currencies, may be scoped to a company.
(`MCUR-013`)

### MCUR-AC-019: the cached currency catalogue is refreshed when a cached value changes

**Given** a client has fetched the cached catalogue of active currencies, which carries for each one its `name`, `symbol`, `position` and `decimal_places`.
**When** `FRG`'s `symbol` is changed from `Fr` to `F$` and the client fetches the catalogue again.
**Then** the entry for `FRG` reads `F$`.
**And when** a Currency is created, or deleted, or has its `active`, `name` or `position` changed, and the client fetches again.
**Then** the catalogue reflects that change too.
**And when** only a currency's `full_name` is changed and the client fetches again.
**Then** the catalogue is unchanged, because `full_name` is not one of the cached values and such a write does not invalidate the cache.
(`MCUR-014`)

---

## B. The rate table

### MCUR-AC-020: two rate rows for the same currency, company and day are refused

**Given** a rate row exists for `FRG`, company "Main" root, dated 2026-01-01.
**When** a user creates a second row for `FRG`, the same company and the same date.
**Then** the create is refused with the message "Only one currency rate per day allowed!"
(`MCUR-022`)

### MCUR-AC-021: a row scoped to a company and a row scoped to nobody may share a day

**Given** a rate row exists for `FRG`, company empty, dated 2026-01-01.
**When** a user creates a row for `FRG`, company "Main" root, dated 2026-01-01.
**Then** both rows exist.
(`MCUR-022`)

### MCUR-AC-022: a non-positive rate is refused

**When** a user creates a rate row with `rate = 0`, or with `rate = −1.5`.
**Then** the create is refused with the message "The currency rate must be strictly positive."
(`MCUR-023`)

### MCUR-AC-023: a rate row on a branch company is refused

**Given** company "Branch" has company "Main" as its parent.
**When** a user creates a rate row whose `company_id` is "Branch".
**Then** the create is refused with the message "Currency rates should only be created for main companies"
(`MCUR-020`)

### MCUR-AC-024: the latest rate on or before the date wins

**Given** the baseline rate rows for `FRG`.
**When** the rate in force is requested for each of the dates 2015-06-30, 2016-01-01, 2016-06-30, 2017-01-01, 2020-01-01.
**Then** the answers are `1.0`, `3.0`, `3.0`, `2.0`, `2.0`.
([calculations.md](calculations.md) section 7; `MCUR-028`)

### MCUR-AC-025: before the first rate, the first rate applies

**Given** a currency whose only rate rows are dated 2016-01-01 with rate `3.0` and 2017-01-01 with rate `2.0`.
**When** the rate in force is requested for 2015-01-01.
**Then** the answer is `3.0`.
(`MCUR-029`)

### MCUR-AC-026: an invoice dated before the first rate uses the first rate

**Given** a currency whose only rate rows are dated 2016-01-01 with rate `3.0` and 2017-01-01 with rate `2.0`, and a company whose main currency carries no rate.
**When** an invoice of `1000.00` in that currency is issued on 2015-01-01, on 2016-01-01 and on 2017-01-01 respectively.
**Then** the three invoices have `amount_total` equal to `1000.00` and `amount_total_signed` equal to `333.33`, `333.33` and `500.00` respectively.
(`MCUR-029`; [calculations.md](calculations.md) section 10.3)

### MCUR-AC-027: a currency with no rate row has a rate of one

**Given** a Currency with no rate row.
**When** `1000.00` of the company's main currency is converted into it on any date.
**Then** the result is `1000.00`.
(`MCUR-030`)

### MCUR-AC-028: a company-scoped rate wins over a shared rate

**Given** the currency `FRG` has a row with no company scope dated 2025-01-01 with rate `0.9500` and a row scoped to the root of company "Main" dated 2024-01-01 with rate `0.9300`.
**When** the rate in force for company "Main" is requested for 2025-06-01.
**Then** the answer is `0.9300`.
([calculations.md](calculations.md) section 7.4)

### MCUR-AC-029: the company rate and the technical rate agree when the main currency has no rate

**Given** the company's main currency carries no rate row.
**When** a user enters `company_rate = 0.9200` on a new rate row for `FRG`.
**Then** the stored `rate` is `0.9200` and `inverse_company_rate` is `1.0869565217391304`.
([calculations.md](calculations.md) section 9)

### MCUR-AC-030: the company rate is scaled by the main currency's own rate

**Given** the company's main currency carries a rate row dated 2026-01-15 with `rate = 1.2500`, and a rate row for `FRG` is being created dated 2026-02-01.
**When** a user enters `company_rate = 0.9200`.
**Then** the stored `rate` is `1.1500`, and converting `1.00` of the main currency into `FRG` on 2026-02-01 yields `0.92`.
([calculations.md](calculations.md) section 9)

### MCUR-AC-031: the inverse rate drives the company rate

**Given** a rate row is being created.
**When** a user enters `inverse_company_rate = 1.25`.
**Then** `company_rate` becomes `0.8`.
([calculations.md](calculations.md) section 9)

### MCUR-AC-032: field precedence when several rate fields are supplied

**When** a rate row is created with `rate = 2.0`, `company_rate = 5.0` and `inverse_company_rate = 0.1` supplied together, in a company whose main currency carries no rate.
**Then** the stored `rate` is `2.0`, `company_rate` reads `2.0` and `inverse_company_rate` reads `0.5`.
(`MCUR-024`)

### MCUR-AC-033: an omitted rate carries forward

**Given** the currency `FRG` has a rate row dated 2016-01-01 with `rate = 3.0`.
**When** a rate row dated 2016-06-01 is created for `FRG` with the same company scope and no rate supplied.
**Then** the stored `rate` of the new row is `3.0`.
(`MCUR-025`)

### MCUR-AC-034: an implausible rate raises a warning but saves

**Given** the currency `FRG` has a latest earlier row with `rate = 0.9200`, and the main currency carries no rate.
**When** a user types `company_rate = 0.7000` in a form.
**Then** a non-blocking warning is shown titled "Warning for FRG" with the body "The new rate is quite far from the previous rate." followed by "Incorrect currency rates may cause critical problems, make sure the rate is correct!"
**And when** the user saves.
**Then** the row is stored with `rate = 0.7000`.
(`MCUR-021`)

### MCUR-AC-035: a rate change within twenty percent raises no warning

**Given** the same starting point.
**When** a user types `company_rate = 0.7500`.
**Then** no warning is shown, because the relative change is `0.1848`.
(`MCUR-021`)

### MCUR-AC-036: changing a rate does not restate a posted entry

**Given** a posted invoice whose receivable item carries `balance = 1086.96` and `amount_currency = 1000.00`.
**When** the rate row that was in force on the invoice date is edited from `0.9200` to `0.8000`.
**Then** the receivable item still carries `balance = 1086.96` and `amount_currency = 1000.00`.
(`MCUR-031`)

### MCUR-AC-037: a rate lookup is re-evaluated when a rate changes

**Given** currency `A` with a rate row dated 2009-09-09 of `1`, and currency `B` with rate rows dated 2009-09-09 of `1` and 2011-11-11 of `2`.
**When** `100` of `A` is converted into `B` at 2010-10-10.
**Then** the answer is `100`.
**And when** the 2009-09-09 row of `B` is changed to `3` and the same conversion is repeated.
**Then** the answer is `300`.
**And when** a row dated 2010-10-10 of `4` is added for `B` and the same conversion is repeated.
**Then** the answer is `400`.
**And when** the conversion is repeated at 2011-11-11.
**Then** the answer is `200`.
([calculations.md](calculations.md) section 7)

### MCUR-AC-038: the rate date is required and defaults to today

**Given** today is 2026-05-04 in the user's time zone, and a user opens a new Currency Rate form for `FRG`.
**When** the form is rendered.
**Then** the rate date (`name`) is pre-filled with 2026-05-04.
**And when** the user clears the rate date and saves.
**Then** the save is refused by the required-field check and no row is written.
**And when** the user restores 2026-05-04 and saves.
**Then** the row exists and applies from 2026-05-04 onward.
(`MCUR-026`)

### MCUR-AC-039: asking for the preceding rate of a row with no date is refused

**Given** a Currency Rate being prepared whose rate date (`name`) is empty and whose technical rate (`rate`) has been left empty, so that the carry-forward of `MCUR-025` would have to look for the preceding row.
**When** the preceding rate is requested.
**Then** the request fails with the message "The name for the current rate is empty." followed by a new line and "Please set it.", and no row is written.
(`MCUR-027`)

---

## C. Conversion, rounding and comparison

### MCUR-AC-040: converting one thousand at a rate of 1.0850

**Given** a company whose main currency is the euro with no rate row, and the United States dollar with a rate row of `1.0850` dated 2026-02-01, both with a rounding step of `0.01`.
**When** `1000.00` euro is converted into dollars on 2026-02-10.
**Then** the result is `1085.00`.
**And when** `1000.00` dollars is converted into euro on the same date.
**Then** the result is `921.66`.
(`MCUR-040`; [calculations.md](calculations.md) section 8.4)

### MCUR-AC-041: the target currency's rounding step is used

**Given** the same setup plus the Japanese yen with a rate row of `165.20` and a rounding step of `1`.
**When** `1000.00` euro is converted into yen.
**Then** the result is `165200`.
**And when** `1000` yen is converted into euro.
**Then** the result is `6.05`.
(`MCUR-046`)

### MCUR-AC-042: converting zero performs no lookup and returns zero

**When** `0.0` is converted between any two currencies on any date.
**Then** the result is `0.0`.
([calculations.md](calculations.md) section 8.2)

### MCUR-AC-043: converting between identical currencies is the identity

**When** `1234.567` is converted from a currency into itself with rounding suppressed.
**Then** the result is `1234.567`, unchanged.
([calculations.md](calculations.md) section 8.2)

### MCUR-AC-044: rounding half away from zero

**Given** a currency with a rounding step of `0.01`.
**When** the amounts `2.675`, `−2.675`, `1.435`, `1.005` are rounded.
**Then** the results are `2.68`, `−2.68`, `1.44`, `1.01`.
(`MCUR-041`)

### MCUR-AC-045: rounding to a non-decimal step

**Given** a currency with a rounding step of `0.05`.
**When** the amounts `1.32`, `1.33`, `1.30` are rounded.
**Then** the results are `1.30`, `1.35`, `1.30`.
([calculations.md](calculations.md) section 3.4)

### MCUR-AC-046: rounding to whole units

**Given** a currency with a rounding step of `1`.
**When** `1234.56` and `1234.49` are rounded.
**Then** the results are `1235` and `1234`.
([calculations.md](calculations.md) section 3.4)

### MCUR-AC-047: comparison rounds before subtracting

**Given** a currency with a rounding step of `0.01`.
**When** `1.432` is compared with `1.431`.
**Then** the result is `0`.
**And when** `0.006` is compared with `0.002`.
**Then** the result is `1`.
(`MCUR-043`)

### MCUR-AC-048: the zero test rounds after subtracting

**Given** a currency with a rounding step of `0.01`.
**When** the amount `0.006 − 0.002` is tested for zero.
**Then** the result is true, even though comparing the two operands returns `1`.
(`MCUR-044`)

### MCUR-AC-049: an amount exactly one step is not zero

**Given** a currency with a rounding step of `0.01`.
**When** `0.01` is tested for zero.
**Then** the result is false.
**And when** `0.004` is tested.
**Then** the result is true.
(`MCUR-044`)

### MCUR-AC-050: a monetary value is rounded when written

**Given** a Journal Item whose `currency_id` has a rounding step of `0.01`.
**When** `amount_currency` is assigned `12.3456`.
**Then** reading the field back returns `12.35`, and the stored value has exactly two fractional digits.
(`MCUR-042`)

### MCUR-AC-051: assigning a monetary value with an ambiguous currency is an error

**Given** an operation assigning a monetary value across several records whose currency fields hold different currencies.
**When** the assignment is performed.
**Then** the operation fails rather than rounding to an arbitrary currency.
(`MCUR-042`)

---

## D. Foreign currency amounts on journal items

### MCUR-AC-060: the two amount columns must agree in sign

**When** a Journal Item is written with `balance = 100.00` and `amount_currency = −50.00`.
**Then** the write is refused with the message "The amount expressed in the secondary currency must be positive when account is debited and negative when account is credited. If the currency is the same as the one from the company, this amount must strictly be equal to the balance."
(`MCUR-060`)

### MCUR-AC-061: a zero in one column is accepted

**When** a Journal Item is written with `balance = 100.00` and `amount_currency = 0.00`, and another with `balance = 0.00` and `amount_currency = −50.00`.
**Then** both writes are accepted.
(`MCUR-060`)

### MCUR-AC-062: a line cannot carry both a debit and a credit

**When** a Journal Item is written with `debit = 10.00` and `credit = 5.00`.
**Then** the write is refused with the message "Wrong credit or debit value in accounting entry!"
(`MCUR-061`)

### MCUR-AC-063: a section or note line carries no amount

**When** a line whose display type marks it as a section is written with `amount_currency = 5.00`.
**Then** the write is refused with the message "Forbidden balance or account on non-accountable line"
(`MCUR-062`)

### MCUR-AC-064: an account with a currency forces that currency

**Given** an account whose `currency_id` is `FRG`, different from the company's main currency.
**When** a Journal Item is written on that account with `currency_id` equal to the company's main currency.
**Then** the write is refused with the message "The account selected on your journal entry forces to provide a secondary currency. You should remove the secondary currency on the account."
(`MCUR-064`)

### MCUR-AC-065: an invoice cannot switch to a currency that conflicts with its account

**Given** a draft customer invoice in `FRG` whose receivable line uses an account whose `currency_id` is `FRG`.
**When** a user changes the invoice's `currency_id` to the company's main currency.
**Then** the change is refused with the message "The account selected on your journal entry forces to provide a secondary currency. You should remove the secondary currency on the account."
(`MCUR-064`)

### MCUR-AC-066: an account currency may not be set once other currencies are present

**Given** an account with no `currency_id` on which a Journal Item exists whose `currency_id` is `FRG`.
**When** a user sets the account's `currency_id` to a different currency.
**Then** the write is refused with the message "You cannot set a currency on this account as it already has some journal entries having a different foreign currency."
(`MCUR-069`)

### MCUR-AC-067: the account currency must match the journal currency

**Given** a bank journal whose `currency_id` is `FRG`, different from the company's main currency, and whose default account has `currency_id` set to `FRG`.
**When** a user changes the default account's `currency_id` to another currency.
**Then** the write is refused with the message "The foreign currency set on the journal '*the journal display name*' and the account '*the account display name*' must be the same."
(`MCUR-068`)

### MCUR-AC-068: a journal currency drives the document currency

**Given** a sale journal whose `currency_id` is the company's main currency, and a second sale journal whose `currency_id` is `FRG`.
**When** a draft invoice created in the first journal is moved to the second.
**Then** the invoice's `currency_id` becomes `FRG` and every line's `currency_id` follows.
(`MCUR-071`)

### MCUR-AC-069: a company-currency line keeps its two columns equal

**Given** a miscellaneous journal entry line whose `currency_id` equals the company's main currency.
**When** `amount_currency` is set to `250.00`.
**Then** `balance` becomes `250.00`.
(`MCUR-066`)

### MCUR-AC-070: a foreign-currency line on a miscellaneous entry re-derives its balance

**Given** a miscellaneous journal entry dated 2017-01-01 with a line whose `currency_id` is `FRG`, whose applicable rate is `2.0`.
**When** `amount_currency` is set to `240.000`.
**Then** `balance` becomes `120.00`.
(`MCUR-066`)

### MCUR-AC-071: switching an invoice currency back and forth preserves the balance

**Given** a draft invoice in `FRG` dated 2016-01-20 whose first line's `balance` is `B`.
**When** the invoice's `currency_id` is changed to the company's main currency and then back to `FRG`, and the invoice is saved.
**Then** the first line's `balance` is again `B`.
([calculations.md](calculations.md) section 10)

### MCUR-AC-072: the amount in currency is derived from the balance and the rate

**Given** a miscellaneous journal entry dated 2017-01-01 whose document currency is `FRG`, whose applicable rate is `2.0`, containing a line whose `balance` is `120.00` and whose `amount_currency` has not been supplied.
**When** the entry is saved.
**Then** the line's `amount_currency` is `240.000`, that is `round( 120.00 × 2.0 , 0.001 )`.
**And given** a second line of the same entry whose `currency_id` equals the company's main currency and whose `balance` is `−75.00`.
**Then** that line's `amount_currency` is forced to `−75.00`, equal to its balance, because the entry is not an invoice, bill, credit note, debit note or receipt.
(`MCUR-065`)

### MCUR-AC-073: the currency of an item follows its entry

**Given** a draft miscellaneous journal entry whose `currency_id` is the company's main currency, with one ordinary line.
**When** the entry's `currency_id` is changed to `FRG`.
**Then** the line's `currency_id` becomes `FRG`.
**And given** a posted customer invoice in `FRG` that carries a cost of goods sold line.
**When** that line is read.
**Then** its `currency_id` is the company's main currency and not `FRG`, because a cost of goods sold line always takes the company currency whatever the entry carries.
(`MCUR-070`)

### MCUR-AC-074: only the company currency column has to balance

**Given** the three-currency bank transaction of `MCUR-AC-156`, whose generated entry holds a liquidity item with `debit = 923.91` and `amount_currency = 850.00` in `EUR`, and a counterpart item with `credit = 923.91` and `amount_currency = −730.00` in `GBP`.
**When** the entry is posted.
**Then** the posting succeeds, because the sum of `balance` over the accountable items is `923.91 − 923.91 = 0.00` at the company currency's precision;
**And** no balance check is applied to `amount_currency`, whose sum is `850.00 + (−730.00) = 120.00` and is deliberately not zero.
(`MCUR-072`)

### MCUR-AC-075: a journal currency is forced on the journal's accounts

**Given** a company whose main currency is `USD`, and a bank journal whose default account and whose linked bank account record both carry no currency.
**When** the journal's `currency_id` is set to `EUR`.
**Then** the journal's default account carries `currency_id = EUR`;
**And** the bank account record linked to the journal carries `EUR`;
**And** the journal's display name is suffixed with `(EUR)`;
**And** an entry created afterwards in that journal takes `EUR` as its document currency.
**And when** the journal's `currency_id` is set instead to `USD`, the company's own main currency.
**Then** the display name carries no currency suffix.
(`MCUR-067`)

---

## E. The document rate

### MCUR-AC-080: a non-positive document rate is refused

**Given** a customer invoice whose `currency_id` is `FRG` and whose `invoice_currency_rate` is `2.0`.
**When** a user types `0` into the Currency Rate field.
**Then** the change is refused with the message "The currency rate must be strictly positive." and `invoice_currency_rate` is still `2.0`.
**And when** a user types `−420`.
**Then** the same refusal occurs and the value is still `2.0`.
(`MCUR-080`)

### MCUR-AC-081: the document rate follows the invoice date

**Given** a currency with rate rows dated 2025-01-01 of `0.5` and 2025-02-01 of `0.4`, and a draft customer invoice in that currency with one line of `1000.00` and a fifteen percent tax.
**When** `invoice_date` is 2025-01-01.
**Then** the lines carry `amount_currency` of `−1000.00`, `−150.00`, `+1150.00` and `balance` of `−2000.00`, `−300.00`, `+2300.00`.
**And when** `invoice_date` is changed to 2025-02-01.
**Then** the amounts in currency are unchanged and the balances become `−2500.00`, `−375.00`, `+2875.00`.
(`MCUR-081`, `MCUR-083`)

### MCUR-AC-082: a manually entered rate drives every balance

**Given** a draft customer invoice in `FRG` with no invoice date, whose expected rate is `2.0`, holding one product line of `2000.00` and a fifteen percent tax.
**When** a user sets `invoice_currency_rate` to `5.0`.
**Then** the lines carry `balance` of `−400.00`, `−60.00`, `+460.00` against `amount_currency` of `−2000.00`, `−300.00`, `+2300.00`.
**And when** the invoice is posted.
**Then** the same values are preserved, `invoice_currency_rate` reads `5.0` and `expected_currency_rate` reads `2.0`.
(`MCUR-082`)

### MCUR-AC-083: a manual rate survives the automatic invoice date assignment

**Given** a draft customer invoice in `FRG` with `invoice_date` empty and `invoice_currency_rate` manually set to `5`, holding one line of `100.00`.
**When** the invoice is posted.
**Then** `invoice_date` is set to today, and the lines carry `amount_currency` of `−100.00` and `+100.00` against `balance` of `−20.00` and `+20.00`.
(`MCUR-082`)

### MCUR-AC-084: a rate equal to the expected rate is recomputed at posting

**Given** a draft customer invoice in a currency with rate rows dated 2025-01-01 of `3.0` and 2026-01-01 of `2.0`, created on 2025-01-02 with no invoice date, whose `invoice_currency_rate` therefore reads `3.0`.
**When** the invoice is posted on 2026-01-02.
**Then** `invoice_currency_rate` reads `2.0`, because the rate was not manual.
(`MCUR-082`)

### MCUR-AC-085: global tax rounding with a fractional rate keeps the entry balanced

**Given** a company whose tax rounding method is "round per tax", and a draft customer invoice in a foreign currency with one line of quantity `0.80` at a unit price of `894.34`.
**When** `invoice_currency_rate` is set to `1 ÷ 1189.5`.
**Then** the two lines carry `balance` of `−851053.94` and `+851053.94`.
([calculations.md](calculations.md) section 10.6)

### MCUR-AC-086: the document rate is not copied

**Given** a posted invoice whose `invoice_currency_rate` was manually set to `5.0`.
**When** the invoice is duplicated.
**Then** the copy's `invoice_currency_rate` equals the expected rate at the copy's own rate date, not `5.0`.
(`MCUR-084`)

### MCUR-AC-087: the rate refresh operation discards the override

**Given** a draft invoice whose `invoice_currency_rate` is `5.0` and whose `expected_currency_rate` is `2.0`.
**When** the rate refresh operation is run.
**Then** `invoice_currency_rate` becomes `2.0` and every balance is re-derived.
(`MCUR-085`)

### MCUR-AC-088: a line of a non-invoice entry reads the rate table

**Given** a miscellaneous journal entry dated 2016-06-01 with a line in `FRG`.
**When** the line's `currency_rate` is read.
**Then** it equals `3.0`, the rate in force on 2016-06-01.
(`MCUR-086`)

---

## F. Exchange differences

### MCUR-AC-100: a customer invoice collected at a weaker rate produces a loss

**Given** company main currency `USD`, a currency `EUR` with rounding step `0.01` and rate rows dated 2026-01-01 of `0.9200` and 2026-03-01 of `0.9500`; a posted customer invoice of `1000.00 EUR` dated 2026-01-15 whose receivable item carries `balance = 1086.96` and `amount_currency = 1000.00`; a posted payment of `1000.00 EUR` dated 2026-03-10 whose receivable item carries `balance = −1052.63` and `amount_currency = −1000.00`.
**When** the two receivable items are reconciled.
**Then** a first Partial Reconciliation exists with `amount = 1052.63`, `debit_amount_currency = 1000.00` and `credit_amount_currency = 1000.00`;
**And** an exchange difference entry exists in the exchange journal with one line crediting the receivable account `34.33` with `amount_currency = 0.00` and `currency_id = EUR`, and one line debiting the loss account `34.33` with `amount_currency = 0.00` and `currency_id = EUR`, both labelled "Currency exchange rate difference";
**And** a second Partial Reconciliation exists with `amount = 34.33` and both document currency amounts zero;
**And** both receivable items have `amount_residual = 0.00`, `amount_residual_currency = 0.00` and `reconciled = true`;
**And** a Full Reconciliation covers the four items.
(`MCUR-100`, `MCUR-105`; [calculations.md](calculations.md) section 14.6; [accounting-effects.md](accounting-effects.md) section 5.1)

### MCUR-AC-101: a customer invoice collected at a stronger rate produces a gain

**Given** the same invoice and a payment of `1000.00 EUR` whose receivable item carries `balance = −1123.60` and `amount_currency = −1000.00`.
**When** the two receivable items are reconciled.
**Then** the matched amount is `1086.96`, and the exchange difference entry debits the receivable account `36.64` and credits the gain account `36.64`, both lines carrying `EUR` with `amount_currency = 0.00`.
(`MCUR-105`; [calculations.md](calculations.md) section 14.7)

### MCUR-AC-102: a partial settlement at a different rate produces a loss and leaves a residual

**Given** two items on the same reconcilable account: a debit item dated 2017-01-01 with `balance = 60.00` and `amount_currency = 120.000` in `FRG`, and a credit item dated 2016-01-01 with `balance = −80.00` and `amount_currency = −240.000` in `FRG`.
**When** the two are reconciled.
**Then** a first Partial Reconciliation exists with `amount = 40.00`, `debit_amount_currency = 120.000`, `credit_amount_currency = 120.000`;
**And** an exchange difference entry exists crediting the debit item's account `20.00` and debiting the loss account `20.00`, both lines carrying `FRG` with `amount_currency = 0.000`;
**And** a second Partial Reconciliation exists with `amount = 20.00` and both document currency amounts zero;
**And** the debit item has `amount_residual = 0.00` and `amount_residual_currency = 0.000` and is reconciled, while the credit item has `amount_residual = −40.00` and `amount_residual_currency = −120.000` and is not.
([accounting-effects.md](accounting-effects.md) section 5.3)

### MCUR-AC-103: the mirror case produces a gain

**Given** a debit item dated 2016-01-01 with `balance = 40.00` and `amount_currency = 120.000` in `FRG`, and a credit item dated 2017-01-01 with `balance = −120.00` and `amount_currency = −240.000` in `FRG`.
**When** the two are reconciled.
**Then** a first Partial Reconciliation exists with `amount = 40.00` and both document currency amounts `120.000`;
**And** an exchange difference entry exists debiting the credit item's account `20.00` and crediting the gain account `20.00`;
**And** the debit item is reconciled and the credit item retains `amount_residual = −60.00` and `amount_residual_currency = −120.000`.
([accounting-effects.md](accounting-effects.md) section 5.4)

### MCUR-AC-104: a full settlement at a different rate produces a loss and a full reconciliation

**Given** a debit item dated 2017-01-01 with `balance = 60.00` and `amount_currency = 120.000` in `FRG`, and a credit item dated 2016-01-01 with `balance = −40.00` and `amount_currency = −120.000` in `FRG`.
**When** the two are reconciled.
**Then** a first Partial Reconciliation exists with `amount = 40.00` and both document currency amounts `120.000`, an exchange difference entry credits the debit item's account `20.00` and debits the loss account `20.00`, a second Partial Reconciliation of `20.00` links the debit item to the correction line, both original items are reconciled, and a Full Reconciliation exists.
([calculations.md](calculations.md) section 14.2)

### MCUR-AC-105: the exchange difference entry date follows the journal's accounting date rule

**Given** the setup of `MCUR-AC-104`, an exchange journal whose numbering resets monthly, and today's date being 2019-01-01.
**When** the reconciliation is performed.
**Then** the exchange difference entry carries `date = 2017-01-31`.
**And given** instead an exchange journal whose numbering resets yearly.
**Then** the entry carries `date = 2017-12-31`.
(`MCUR-106`)

### MCUR-AC-106: a missing exchange journal aborts the reconciliation

**Given** the setup of `MCUR-AC-104` and a company with no `currency_exchange_journal_id`.
**When** the reconciliation is attempted.
**Then** it fails with the message "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." and no Partial Reconciliation is created.
(`MCUR-102`)

### MCUR-AC-107: a missing loss account aborts the reconciliation

**Given** the same setup with an exchange journal but no `expense_currency_exchange_account_id`.
**When** the reconciliation is attempted.
**Then** it fails with the message "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."
(`MCUR-103`)

### MCUR-AC-108: a missing gain account aborts the reconciliation

**Given** the same setup with an exchange journal and a loss account but no `income_currency_exchange_account_id`.
**When** the reconciliation is attempted.
**Then** it fails with the message "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."
(`MCUR-104`)

### MCUR-AC-109: undoing a reconciliation reverses the exchange difference entry

**Given** the completed reconciliation of `MCUR-AC-104`, whose exchange difference entry is posted.
**When** the reconciliation is undone.
**Then** a reversing entry exists whose lines mirror the original, whose `ref` reads "Reversal of: *the entry number of the original entry*";
**And** the original entry's two lines and the reversal's two lines together show the reconciled flags `true`, `false`, `true`, `false` in the order original correction line, original counterpart line, reversal correction line, reversal counterpart line;
**And** the two original items regain their residuals.
(`MCUR-111`)

### MCUR-AC-110: a draft exchange difference entry is deleted rather than reversed

**Given** a reconciliation between two items at least one of which belongs to a draft entry, which produced a draft exchange difference entry.
**When** the reconciliation is undone.
**Then** the draft exchange difference entry no longer exists and no reversal was created.
(`MCUR-111`)

### MCUR-AC-111: the rounding tolerance prevents a spurious exchange difference

**Given** a company whose main currency is `USD`, a currency with a rounding step of `0.01` and a single rate row of `0.052972554919`, and two items on the same reconcilable account dated at that rate's date: a debit item with `balance = 377554.00` and `amount_currency = 20000.00` in that currency, and a credit item with `balance = −372239.38` and `amount_currency = −372239.38` in the company currency.
**When** the two are reconciled.
**Then** exactly one Partial Reconciliation exists with `amount = 372239.38`, `debit_amount_currency = 19718.47` and `credit_amount_currency = 372239.38`;
**And** no exchange difference entry was created;
**And** no Full Reconciliation exists, because the debit item retains `5314.62` and `281.53`.
([calculations.md](calculations.md) section 13.7)

### MCUR-AC-112: a three-line group reaches full reconciliation without an exchange difference

**Given** the two items of `MCUR-AC-111` plus a third item on the same account with `balance = −5314.62` and `amount_currency = −281.53` in the foreign currency.
**When** the three are reconciled together.
**Then** two Partial Reconciliations exist, one with `amount = 5314.62`, `debit_amount_currency = 281.53` and `credit_amount_currency = 281.53`, and one with `amount = 372239.38`, `debit_amount_currency = 19718.47` and `credit_amount_currency = 372239.38`;
**And** all three items have both residuals zero and are reconciled;
**And** a single Full Reconciliation covers all three.
**And** the same holds when every sign is inverted.
([calculations.md](calculations.md) section 13.7)

### MCUR-AC-113: a company-currency item mirrored at a tiny rate produces no exchange difference

**Given** a company whose main currency is `USD`, a currency with a rounding step of `0.001` and a single rate of `0.00001`, a credit item with `balance = −10.00` and `amount_currency = −10.00` in the company currency, and a debit item with `balance = 1000000.00` and `amount_currency = 100.000` in that currency.
**When** the two are reconciled.
**Then** one Partial Reconciliation exists with `amount = 10.00`, `debit_amount_currency = 0.001` and `credit_amount_currency = 10.00`;
**And** no exchange difference entry exists;
**And** the credit item is reconciled and the debit item retains `999990.00` and `99.999`.
([calculations.md](calculations.md) section 13.8)

### MCUR-AC-114: a payment in a foreign currency settling a company-currency invoice uses the payment's rate

**Given** a posted customer invoice of `60.00` in the company's main currency dated 2017-01-01, and a payment of `90.00` in `FRG` dated 2016-01-01 registered against it, `FRG` having rates of `3.0` from 2016-01-01 and `2.0` from 2017-01-01.
**When** the payment is created and reconciled against the invoice.
**Then** the invoice's receivable item retains `amount_residual = 30.00` and `amount_residual_currency = 30.00` and is not reconciled, while the payment's receivable item is fully reconciled.
(`MCUR-137`)

### MCUR-AC-115: matching items in a shared foreign currency always reaches a full reconciliation

**Given** three items in `FRG` dated 2017-01-01 with `amount_currency` of `+120.000`, `+120.000` and `−240.000`, and balances derived from any combination of the rates `1 ÷ 3` and `1 ÷ 2`.
**When** the three are reconciled.
**Then** a Full Reconciliation exists for every one of the eight rate combinations.
([calculations.md](calculations.md) section 19)

### MCUR-AC-116: a group of mixed currencies reaches a full reconciliation

**Given** five items on one reconcilable account: `+1200.00 / +3600.000 FRG` dated 2016-01-01, `+780.00 / +2340.000 FRG` dated 2016-01-01, `−240.00 / −960.000` in a second foreign currency dated 2017-01-01, `−720.00 / −2880.000` in the second currency dated 2017-01-01, and `−1020.00 / −4080.000` in the second currency dated 2017-01-01.
**When** all five are reconciled together.
**Then** every item ends with `amount_residual = 0.00`, `amount_residual_currency = 0.000` and `reconciled = true`, and a Full Reconciliation exists;
**And** the batch was split by currency before matching, the two `FRG` items being matched against the second currency's items only after each same-currency group had been matched on its own.
(`MCUR-135`; [calculations.md](calculations.md) section 19)

### MCUR-AC-117: a matching leaving a tail in the document currency is not a full reconciliation

**Given** two items in `FRG` with rounding step `0.001`, both with `balance = 0.00`, the first with `amount_currency = −0.020` and the second with `+0.010`.
**When** the two are reconciled.
**Then** no Full Reconciliation exists, because `0.010` remains open in the document currency.
**And when** a third item with `balance = 0.00` and `amount_currency = +0.010` is reconciled against the first.
**Then** a single Full Reconciliation covers all three.
([calculations.md](calculations.md) section 19)

### MCUR-AC-118: an exchange difference line consumes no document currency residual

**Given** the completed reconciliation of `MCUR-AC-102`.
**When** the second Partial Reconciliation is inspected.
**Then** its `amount` is `20.00` and both `debit_amount_currency` and `credit_amount_currency` are `0.000`.
(`MCUR-113`)

### MCUR-AC-119: the exchange difference entry is posted only when both sides are posted

**Given** two items to reconcile, both belonging to posted entries.
**When** the reconciliation runs.
**Then** the generated exchange difference entry has state posted and carries a number from the exchange journal's sequence.
**And given** instead that one of the two items belongs to a draft entry.
**Then** the generated exchange difference entry has state draft and its name is `/`.
(`MCUR-107`)

### MCUR-AC-120: the exchange difference entry carries no tax and is always exigible

**Given** any generated exchange difference entry.
**When** its lines are inspected.
**Then** no line carries a tax, and the entry is flagged always tax-exigible.
(`MCUR-108`)

### MCUR-AC-121: matching lines exclude the exchange difference lines in the counterparty view

**Given** three items in `FRG`: a debit item of `+120.00 / +240.000` dated 2017-01-01 and two credit items of `−40.00 / −120.000` dated 2016-01-01 each.
**When** the three are reconciled and the debit item's matched lines are inspected.
**Then** the count of all matched lines is four, the count excluding exchange difference lines is two, and the excluding view lists exactly the two credit items.
([entities.md](entities.md) section 4)

### MCUR-AC-122: every exchange difference line carries the standard label

**Given** the completed reconciliation of `MCUR-AC-100`, which produced a loss.
**When** the two lines of the exchange difference entry are read.
**Then** the correction line on the receivable account and the counterpart line on the loss account each carry the label "Currency exchange rate difference", character for character.
**And given** the reconciliation of `MCUR-AC-101`, which produced a gain.
**Then** both of its lines carry that same label.
(`MCUR-109`)

### MCUR-AC-123: the exchange difference entry is linked to one partial matching

**Given** the completed reconciliation of `MCUR-AC-100`, which produced two Partial Reconciliations and one exchange difference entry.
**When** the two Partial Reconciliations are read.
**Then** the first, the one with `amount = 1052.63`, carries the exchange difference entry in its `exchange_move_id`;
**And** the second, the one with `amount = 34.33`, carries none, because a generated entry is attached to the first matching that involves one of the items it corrects and that does not already carry an entry.
**And given** a batch that produces two exchange difference entries and three matchings.
**Then** the two entries are attached to two different matchings and no matching carries two entries.
(`MCUR-110`)

### MCUR-AC-124: an exchange difference line leaves the other amount column at zero

**Given** the completed reconciliation of `MCUR-AC-100`, whose correction is expressed in the company currency column.
**When** the two lines of the exchange difference entry are read.
**Then** both carry `currency_id = EUR` and `amount_currency = 0.00`, leaving the document currency residual of the corrected item untouched;
**And** the sign agreement check of `MCUR-060` is satisfied, because one of the two columns is zero.
**And given** instead a matching measured in the company currency that leaves a residual in the document currency, in which case the correction is expressed in the document currency column.
**Then** the two generated lines carry that document currency amount in `amount_currency` and a `balance` of `0.00`.
(`MCUR-112`)

### MCUR-AC-125: matching a correction line against the item it corrects produces no further difference

**Given** the exchange difference entry of `MCUR-AC-100` and its correction line on the receivable account.
**When** that correction line is matched against the invoice receivable item, which is the closing step of the reconciliation.
**Then** the matching runs with the exchange difference computation suppressed by the operation context, and no second exchange difference entry is created;
**And** the whole reconciliation ends with exactly one exchange difference entry, not an unbounded chain of corrections of corrections.
(`MCUR-101`)

---

## G. Reconciliation guards

### MCUR-AC-130: already reconciled items may not be reconciled again

**Given** two items that are fully reconciled with each other.
**When** a user attempts to reconcile them again.
**Then** the attempt fails with the message "You are trying to reconcile some entries that are already reconciled."
(`MCUR-130`)

### MCUR-AC-131: cancelled entries may not be reconciled

**Given** an item belonging to a cancelled entry.
**When** a user attempts to reconcile it.
**Then** the attempt fails with the message "You can not reconcile cancelled entries."
(`MCUR-131`)

### MCUR-AC-132: items from different accounts may not be reconciled together

**Given** two items on different accounts.
**When** a user attempts to reconcile them.
**Then** the attempt fails with the message "Entries are not from the same account: *the comma-separated account display names*"
(`MCUR-132`)

### MCUR-AC-133: items from different root companies may not be reconciled together

**Given** two items whose companies have different roots.
**When** a user attempts to reconcile them.
**Then** the attempt fails with the message "Entries don't belong to the same company: *the comma-separated company display names*"
(`MCUR-133`)

### MCUR-AC-134: an account that does not allow reconciliation refuses

**Given** two items on an account that is neither reconcilable, nor a cash account, nor a credit card account.
**When** a user attempts to reconcile them.
**Then** the attempt fails with the message "Account *the account display name* does not allow reconciliation. First change the configuration of this account to allow it."
(`MCUR-134`)

### MCUR-AC-135: an item with a zero balance and a non-zero document amount still participates

**Given** an item with `balance = 0.00` and `amount_currency = +0.010` in `FRG`.
**When** a reconciliation batch containing it is processed.
**Then** the item is treated as a debit side and takes part in the matching.
([workflows.md](workflows.md) procedure 10, step 4)

### MCUR-AC-136: items are matched in a defined order

**Given** five open items of one partner on one reconcilable account, all in `FRG`: item `A` with maturity date 2026-01-31, item `B` with no maturity date and accounting date 2026-02-15, item `C` with maturity date 2026-02-28 and `amount_currency = 100.000`, item `D` with maturity date 2026-02-28 and `amount_currency = 300.000`, and item `E` with maturity date 2026-03-31.
**When** the batch is ordered for matching.
**Then** the order is `A`, `B`, `C`, `D`, `E`: by maturity date, using the accounting date where there is no maturity date, then by currency, then by amount in currency, then by balance.
**And given** the same five items spread over two partners.
**Then** the items are additionally ordered by partner, ensuring that the items of one partner close among themselves before any cross-partner matching occurs.
**And given** an operation context that asks for the reduced ordering.
**Then** the ordering stops after the currency and the remaining keys are not applied.
(`MCUR-136`)

---

## H. Bank transactions in a foreign currency

### MCUR-AC-150: the foreign currency must differ from the bank account currency

**Given** a bank journal whose effective currency is `USD`.
**When** a statement line is written with `foreign_currency_id = USD`.
**Then** the write is refused with the message "The foreign currency must be different than the journal one: USD"
(`MCUR-150`)

### MCUR-AC-151: an amount in currency without a foreign currency is refused

**When** a statement line is written with `foreign_currency_id` empty and `amount_currency = 100.00`.
**Then** the write is refused with the message "You can't provide an amount in foreign currency without specifying a foreign currency."
(`MCUR-151`)

### MCUR-AC-152: a foreign currency without an amount is refused

**When** a statement line is written with `foreign_currency_id = FRG` and `amount_currency = 0.00`.
**Then** the write is refused with the message "You can't provide a foreign currency without specifying an amount in 'Amount in Currency' field."
(`MCUR-152`)

### MCUR-AC-153: a redundant foreign currency on create is silently dropped

**Given** a bank journal whose effective currency is `USD`.
**When** a statement line is created in one operation supplying that journal and `foreign_currency_id = USD`.
**Then** the created line has `foreign_currency_id` empty and `amount_currency = 0.00`, and no error is raised.
(`MCUR-153`)

### MCUR-AC-154: the amount in currency is derived when omitted

**Given** a bank journal in `USD`, the currency `FRG` with a rate of `3.0` on 2016-06-01.
**When** a statement line dated 2016-06-01 is created with `amount = 100.00` and `foreign_currency_id = FRG` and no amount in currency.
**Then** `amount_currency` becomes `300.000`.
(`MCUR-154`)

### MCUR-AC-155: an entered amount in currency is never overwritten

**Given** the same setup.
**When** the statement line is created with `amount_currency = 290.000` supplied explicitly.
**Then** `amount_currency` remains `290.000` after the line is saved and after `amount` is edited.
(`MCUR-154`)

### MCUR-AC-156: the three-currency transaction generates the expected items

**Given** a company whose main currency is `USD`, a bank journal whose currency is `EUR`, and `EUR` with a rate of `0.9200` on 2026-04-10.
**When** a statement line dated 2026-04-10 is created with `amount = 850.00`, `foreign_currency_id = GBP` and `amount_currency = 730.00`.
**Then** the generated entry has a liquidity item with `currency_id = EUR`, `amount_currency = 850.00`, `debit = 923.91`, and a counterpart item on the suspense account with `currency_id = GBP`, `amount_currency = −730.00`, `credit = 923.91`;
**And** the company currency value `923.91` was derived from the bank account currency amount `850.00 EUR` at the rate `0.9200`, not from the transacted amount `730.00 GBP`.
(`MCUR-156`; [calculations.md](calculations.md) section 21.3)

### MCUR-AC-157: a statement line without a suspense account is refused

**Given** a bank journal with no suspense account.
**When** a statement line is created without an explicit counterpart account.
**Then** the operation fails with the message "You can't create a new statement line without a suspense account set on the *journal display name* journal."
(`MCUR-155`)

### MCUR-AC-158: settling from a three-currency transaction uses the bank's implied rates

**Given** the transaction of `MCUR-AC-156`, settling an invoice of `730.00 GBP` whose receivable was booked at `945.00 USD`.
**When** the transaction is matched against the invoice.
**Then** the counterpart item carries `amount_currency = −730.00` in `GBP` and `credit = 923.91` in `USD`;
**And** an exchange difference entry credits the receivable account `21.09` and debits the loss account `21.09`.
(`MCUR-157`; [accounting-effects.md](accounting-effects.md) section 7.4)

---

## I. The main currency of a company

### MCUR-AC-170: a branch must share its root's currency

**Given** company "Branch" whose parent is company "Main" with main currency `USD`.
**When** a user writes `currency_id = FRG` on "Branch".
**Then** the write is refused with the message "The Currency of a subsidiary must be the same as it's root company."
(`MCUR-170`)

### MCUR-AC-171: the main currency cannot change once entries exist

**Given** company "Main" with main currency `USD` and at least one Journal Item.
**When** a user writes `currency_id = FRG` on "Main".
**Then** the write is refused with the message "You cannot change the currency of the company since some journal items already exist"
(`MCUR-171`)

### MCUR-AC-172: an entry on a branch blocks the root's currency change

**Given** company "Main" with no journal item of its own, and its branch "Branch" with one journal item.
**When** a user writes a different `currency_id` on "Main".
**Then** the write is refused with the same message, because the check covers the root and everything below it.
(`MCUR-171`)

### MCUR-AC-173: the main currency may change while no entry exists

**Given** company "Main" with no journal item anywhere below its root.
**When** a user writes `currency_id = FRG` on "Main".
**Then** the write succeeds and `FRG` becomes active if it was not.
(`MCUR-171`, `MCUR-009`)

### MCUR-AC-174: choosing a country proposes its currency

**Given** a company form with no country set.
**When** a user selects a country whose associated currency is `FRG`.
**Then** the form's `currency_id` becomes `FRG`.
(`MCUR-172`)

### MCUR-AC-175: rate column labels follow the company currency

**Given** company "Foo" with main currency `EUR` and company "Bar" with main currency `USD`.
**When** the currency form and the rate list are rendered for each company in turn.
**Then** the `company_rate` column heading reads "Unit per EUR" for "Foo" and "Unit per USD" for "Bar", and the `inverse_company_rate` heading reads "EUR per Unit" and "USD per Unit" respectively.
([interfaces.md](interfaces.md) section 2.4)

### MCUR-AC-176: a new company takes the main currency of its creator's company

**Given** a user whose active company is "Foo", whose main currency is `EUR`.
**When** that user creates a company "Baz" without naming a currency and without choosing a country.
**Then** "Baz" has `currency_id = EUR`.
**And when** the user then chooses for "Baz" a country whose associated currency is `USD`, while "Baz" still has no journal entry.
**Then** "Baz" has `currency_id = USD`, and `USD` is activated if it was archived (`MCUR-172`, `MCUR-009`).
(`MCUR-173`)

---

## J. Access and visibility

### MCUR-AC-190: every reader can read currencies and rates

**Given** an unauthenticated visitor, a portal user and an internal user.
**When** each reads a Currency and a Currency Rate.
**Then** each read succeeds.
(`MCUR-190`)

### MCUR-AC-191: an internal user without the manager group cannot write a rate

**Given** an internal user holding neither the system administration group nor the accounting manager group.
**When** the user attempts to create a Currency Rate.
**Then** the attempt is refused by the access control layer.
(`MCUR-191`)

### MCUR-AC-192: an accounting manager can write currencies and rates

**Given** a user holding the accounting manager group.
**When** the user creates, edits and deletes a Currency and a Currency Rate.
**Then** every operation succeeds.
(`MCUR-191`)

### MCUR-AC-193: the company column of a rate row is hidden without the multi-company group

**Given** a user who does not hold the multi-company access group.
**When** the rate list is rendered.
**Then** the `company_id` column is absent.
(`MCUR-193`)

### MCUR-AC-194: the technical rate is hidden without the technical features group

**Given** a user who does not hold the technical features group.
**When** the rate form is rendered.
**Then** the `rate` field is absent, and only `company_rate` and `inverse_company_rate` are shown.
(`MCUR-032`)

### MCUR-AC-195: currency fields appear only in a multi-currency setting

**Given** a platform with exactly one active currency, meaning that no internal user holds the multi-currency access group.
**When** an invoice form, a journal item list and a rate list are rendered.
**Then** the document currency selector, the amount in currency column, the document rate field and the company column of a rate row are all absent.
**And when** a second currency is activated, which grants the multi-currency access group by `MCUR-008`.
**Then** the same screens show the document currency selector, the amount in currency column and the document rate field;
**And** the company column of a rate row stays hidden unless the reader also holds the multi-company access group (`MCUR-193`).
(`MCUR-192`)

---

## K. Formatting and spelling

### MCUR-AC-200: formatting places the symbol as configured

**Given** a currency with `symbol = "$"`, `position = before`, `decimal_places = 2`, in a language grouping by three with a comma and a decimal point.
**When** `1234.5` is formatted.
**Then** the result is `$` followed by a non-breaking space followed by `1,234.50`.
**And given** a currency with `symbol = "€"` and `position = after`, in a language grouping by three with a full stop and a decimal comma.
**When** `1234.5` is formatted.
**Then** the result is `1.234,50` followed by a non-breaking space followed by `€`.
([calculations.md](calculations.md) section 24)

### MCUR-AC-201: negative zero is formatted without a sign

**Given** a currency with two decimal places.
**When** `−0.0` is formatted.
**Then** the result carries no minus sign.
([calculations.md](calculations.md) section 24)

### MCUR-AC-202: a currency with no fractional part is formatted without decimals

**Given** the Japanese yen with `decimal_places = 0`.
**When** `1234.56` is formatted.
**Then** the numeric part reads `1,235`.
([calculations.md](calculations.md) section 24)

### MCUR-AC-203: spelling an amount with a fractional part

**Given** the United States dollar with `currency_unit_label = "Dollars"` and `currency_subunit_label = "Cents"`, in English.
**When** `1234.56` is spelled.
**Then** the result is "One Thousand, Two Hundred And Thirty-Four Dollars and Fifty-Six Cents".
([calculations.md](calculations.md) section 25)

### MCUR-AC-204: spelling a whole amount omits the subunit clause

**Given** the same currency.
**When** `1234.00` is spelled.
**Then** the result is "One Thousand, Two Hundred And Thirty-Four Dollars", with no `and` clause.
([calculations.md](calculations.md) section 25)

### MCUR-AC-205: a leading zero in the fractional part is read as a whole number of subunits

**Given** the same currency.
**When** `1234.05` is spelled.
**Then** the result is "One Thousand, Two Hundred And Thirty-Four Dollars and Five Cents".
([calculations.md](calculations.md) section 25)

### MCUR-AC-206: spelling in a currency with no fractional part

**Given** the Japanese yen with `decimal_places = 0` and `currency_unit_label = "Yen"`.
**When** `1234.56` is spelled.
**Then** the result is "One Thousand, Two Hundred And Thirty-Five Yen", with no subunit clause.
([calculations.md](calculations.md) section 25)

### MCUR-AC-207: an unsupported spelling language falls back to English

**Given** a user whose language has no spelling rules available.
**When** an amount is spelled.
**Then** the words are produced in English rather than failing.
([calculations.md](calculations.md) section 25)

---

## L. Reporting rate tables

### MCUR-AC-220: a single-currency group needs no table

**Given** three companies that all have `USD` as their main currency.
**When** a consolidated report is prepared for them.
**Then** the synthetic table is used, every factor is one, and no temporary table is created.
([calculations.md](calculations.md) section 20.1)

### MCUR-AC-221: the current factor converts at the closing rate

**Given** a reporting company whose main currency is `USD` with no rate row, and a subsidiary whose main currency has a rate of `0.9500` at the period end date.
**When** the current factor is computed for that subsidiary and period.
**Then** it equals `1 ÷ 0.9500 = 1.052632`, and a balance of `10000.00` in the subsidiary's currency consolidates as `10526.32`.
([calculations.md](calculations.md) section 20.3)

### MCUR-AC-222: the average factor is day-weighted

**Given** a reporting company whose main currency is `USD` with no rate row, a subsidiary whose main currency has rates of `0.9000` from 2024-01-01 and `0.9500` from 2024-07-01, and a period from 2024-01-01 to 2024-12-31.
**When** the average factor is computed.
**Then** it equals `( 1.111111 × 182 + 1.052632 × 184 ) ÷ 366 = 1.081712`, and a result of `10000.00` in the subsidiary's currency consolidates as `10817.12`.
([calculations.md](calculations.md) section 20.5)

### MCUR-AC-223: the historical factor uses the rate of the movement date

**Given** the same subsidiary and reporting company.
**When** the historical rows are produced for the period ending 2024-12-31.
**Then** one row is valid from 2024-01-01 with a factor of `1 ÷ 0.9000 = 1.111111` and one row is valid from 2024-07-01 with a factor of `1 ÷ 0.9500 = 1.052632`, and each row's next-change date is the date of the following rate.
([calculations.md](calculations.md) section 20.4)

---

## M. Unrealised gains and losses

### MCUR-AC-240: the unrealised adjustment is posted and reversed

**Given** a company whose main currency is `USD`, an open receivable of `1000.00 EUR` booked at `1086.96 USD`, and a rate of `0.9400` at the reporting date 2026-06-30.
**When** the accountant posts the adjustment with a reversal date of 2026-07-01.
**Then** an entry dated 2026-06-30 debits the chosen expense account `23.13` and credits the receivable account `23.13`, both lines carrying `EUR` with `amount_currency = 0.00`;
**And** an entry dated 2026-07-01 reverses it exactly;
**And** the receivable's `amount_residual_currency` is still `1000.00`.
(`MCUR-200`)

### MCUR-AC-241: the adjustment does not disturb a later matching

**Given** the posted adjustment and its reversal of `MCUR-AC-240`.
**When** the receivable is later settled by a payment of `1000.00 EUR` and reconciled.
**Then** the realised exchange difference is computed from the receivable's original book value of `1086.96`, not from the adjusted value, because the adjustment was reversed before the settlement date.
(`MCUR-200`)

---

## N. Cross-cutting

### MCUR-AC-260: a conversion round trip is not assumed

**Given** a currency with a rounding step of `1` and a rate of `165.20` against a company currency with a rounding step of `0.01`.
**When** `0.07` of the company currency is converted into that currency and back.
**Then** the implementation does not assert that the result equals `0.07`; a difference of up to one rounding step of either currency is accepted.
(`MCUR-045`)

### MCUR-AC-261: both amount columns are written together

**Given** a Journal Item with `balance = 100.00` and `amount_currency = 200.000`.
**When** an operation writes only `balance = −100.00`.
**Then** the operation does not fail on the sign check, because `amount_currency` is written in the same statement or the check is deferred to the end of the transaction.
(`MCUR-063`)

### MCUR-AC-262: residual amounts are recomputed after every matching

**Given** the completed reconciliation of `MCUR-AC-100`.
**When** the invoice's receivable item is read.
**Then** `amount_residual = 0.00`, `amount_residual_currency = 0.00` and `reconciled = true`, computed as `1086.96 − ( 1052.63 + 34.33 )` and `1000.00 − ( 1000.00 + 0.00 )`.
([calculations.md](calculations.md) section 18)

### MCUR-AC-263: a matching number is assigned to every connected item

**Given** the completed reconciliation of `MCUR-AC-100`.
**When** each of the four items is read.
**Then** each carries the same `matching_number`, equal to the identifier of the Full Reconciliation.
**And given** instead the partial reconciliation of `MCUR-AC-102`.
**Then** every connected item carries `P` followed by the identifier of the smallest partial matching of the group.
([workflows.md](workflows.md) procedure 10, step 9)

### MCUR-AC-264: a document becomes paid when its receivable closes

**Given** the completed reconciliation of `MCUR-AC-100`.
**When** the invoice is read.
**Then** its payment state is paid, and a message recording the payment has been posted in its discussion thread.
([workflows.md](workflows.md) procedure 10, step 11)

---

## O. Further scenarios

These scenarios exercise the rules that neither draft covered with a numbered scenario. Their
numbers continue the same scheme and are stable.

### MCUR-AC-270: a currency is found by code, name, symbol or unit label

**Given** the currency `FRG` whose `full_name` is "Foreign", whose `symbol` is "Fr", whose
`currency_unit_label` is "Francs" and whose `currency_subunit_label` is "Centimes".
**When** the single search field of the currency catalogue is given each of the terms "FRG",
"Fore", "Fr", "Franc" and "Centi" in turn.
**Then** `FRG` is among the matches every time, because any one of the five fields containing the
term is a match.
**And when** the record-name search — the search a selector performs while a user types — is given
"Centi".
**Then** `FRG` is **not** matched, because that search looks only at the currency code and the full
name.
(`MCUR-017`)

### MCUR-AC-271: a fresh platform has no active currency, and demonstration data activates one

**Given** a platform loaded with reference data and without demonstration data.
**When** the currencies are counted.
**Then** one hundred and seventy Currency records exist and every one of them has `active` false,
so no internal user holds the multi-currency permission group.
**And when** the same platform is loaded with demonstration data instead.
**Then** the United States dollar has `active` true, exactly one currency is active, the
multi-currency permission group is still not held, and one hundred and sixty-three Currency Rate
rows exist.
(`MCUR-018`, `MCUR-008`; [configuration.md](configuration.md) section 4.4)

### MCUR-AC-272: the current rate shown on a currency depends on the company in context

**Given** company "Foo" whose main currency is `EUR` and company "Bar" whose main currency is
`USD`, and a currency `FRG` whose rate rows give, on today's date, a technical rate of 3.0 while
`EUR` carries no rate row and `USD` carries a rate row of 1.2500.
**When** a user working in "Foo" reads `FRG`'s `rate`.
**Then** the value is 3.0, because the evaluation currency is `EUR`, whose rate is one.
**And when** a user working in "Bar" reads the same field.
**Then** the value is 2.4, because 3.0 ÷ 1.2500 = 2.4.
**And when** a user working in "Bar" reads `USD`'s `rate_string`.
**Then** it is empty, because `USD` is that company's own main currency.
(`MCUR-019`)

### MCUR-AC-273: a company-scoped rate beats a shared rate with a later date

**Given** the currency `FRG` with a shared row dated 1 June 2026 at a technical rate of 5.0 and a
row scoped to the root of company "Main" dated 1 January 2026 at 4.0.
**When** the rate in force for company "Main" is requested for 1 July 2026.
**Then** the answer is 4.0, because any row naming the company beats every shared row whatever its
date.
**And when** the rate in force is requested for a company that has no row of its own.
**Then** the answer is 5.0.
(`MCUR-033`)

### MCUR-AC-274: rate rows are ordered newest first and searched by parsed date

**Given** three rate rows of `FRG` dated 1 January 2016, 1 January 2017 and 1 January 1900.
**When** the rate list is opened with no explicit ordering.
**Then** the order is 1 January 2017, 1 January 2016, 1 January 1900.
**And when** the search field is given a term that parses as a date in the reader's format,
"01/01/2017".
**Then** the row dated 1 January 2017 is matched.
**And when** the search field is given "3.0", which does not parse as a date.
**Then** the rows whose technical rate contains that text are matched instead.
(`MCUR-034`)

### MCUR-AC-275: the reciprocal of a rate never divides by zero

**Given** a rate row being edited whose `company_rate` is zero or unset.
**When** `inverse_company_rate` is read.
**Then** the company rate is first forced to one and the answer is 1, rather than a failure.
**And when** the same is done in the other direction, reading `company_rate` from an
`inverse_company_rate` of zero.
**Then** the answer is 1.
(`MCUR-035`)

### MCUR-AC-276: writing a rate refreshes every displayed current rate

**Given** a currency list already read once, showing `FRG` with a current rate of 3.0.
**When** a rate row of `FRG` is created or written and the list is read again.
**Then** the current rate column and the rate as text both show the new value, because writing a
rate row invalidates the derived inverse rate of every currency.
(`MCUR-036`)

### MCUR-AC-277: a rate is never rounded

**Given** a rate row whose technical rate is 0.052972554919 and a currency whose rounding factor is
one hundredth.
**When** an amount is converted with that rate.
**Then** the full stored precision of the rate is used in the multiplication and only the **result**
is rounded onto the target currency's rounding factor; the rate itself is neither rounded to two
decimal places nor to any other precision.
**And when** the rate is displayed on a rate row.
**Then** it is shown with twelve digits of which twelve are fractional, and on the currency list
with twelve digits of which six are fractional, both being presentation choices that do not change
the stored value.
(`MCUR-037`)

### MCUR-AC-278: a new rate row takes the root company, and may be shared

**Given** a user working in branch company "Branch", whose root is "Main".
**When** the user starts a new rate row.
**Then** `company_id` is pre-filled with "Main", the root company, and not with "Branch".
**And when** the user clears `company_id` and saves.
**Then** the row is stored with no company and applies to every company that has no row of its own
satisfying the date condition.
(`MCUR-038`, `MCUR-020`)

### MCUR-AC-279: a rate row cannot be archived

**Given** any Currency Rate row.
**When** an attempt is made to archive it.
**Then** no activity flag exists on the entity to write; the only ways to remove a row are deleting
it and deleting its currency.
**And when** the row is deleted.
**Then** the validity window of the preceding row extends forward over it, and no posted journal
item changes.
(`MCUR-039`, `MCUR-031`)

### MCUR-AC-280: a conversion between two foreign currencies performs one multiplication

**Given** a company reporting in `EUR`, which carries no rate row and therefore a technical rate of
1.0000, with `USD` at 1.1723 and `GBP` at 0.8391 on 15 March 2026.
**When** 500.00 `USD` is converted into `GBP` on that date.
**Then** the result is 357.89 `GBP`, obtained as one composed factor 0.8391 ÷ 1.1723 =
0.715772413204811…, one multiplication 500.00 × 0.715772413204811… = 357.886206602405…, and one
rounding onto one hundredth.
**And** a two-step route through the company currency, rounding after each step, would give 357.88,
which is not the specified behaviour.
**And when** 4321.09 `USD` is converted the same way.
**Then** the result is 3092.92, where the two-step route would give 3092.91.
(`MCUR-047`; [calculations.md](calculations.md) section 8.3)

### MCUR-AC-281: the short-circuits of a conversion

**When** a conversion is asked for with both currencies absent.
**Then** the operation fails rather than guessing a currency.
**And when** it is asked for with one currency absent.
**Then** the missing one is taken to be the given one and the amount is returned unchanged apart
from the rounding onto that currency.
**And when** rounding is suppressed by the caller and 500.00 `USD` is converted into `GBP` at the
rates of `MCUR-AC-280`.
**Then** the raw product 357.886206602405… is returned unrounded.
(`MCUR-048`)

### MCUR-AC-282: a precision must be given exactly one way

**When** the rounding routine, the comparison, the zero test or the euclidean division is called
with **both** a digit count and a rounding factor, or with **neither**.
**Then** the call fails loudly, because supplying both or neither is a programming error and not a
user error.
**And when** it is called with a rounding factor that is zero or negative.
**Then** the call fails.
**And when** it is called with a digit count that is not a whole number greater than or equal to
zero.
**Then** the call fails.
(`MCUR-049`)

### MCUR-AC-283: the five rounding methods

**Given** a rounding factor of one hundredth.
**When** the value 2.675 is rounded by each of the five methods in turn.
**Then** the results are: half away from zero 2.68; half towards zero 2.67; half to even 2.68;
away from zero 2.68; towards zero 2.67.
**And when** 2.665 is rounded half to even.
**Then** the result is 2.66, because the floor two hundred sixty-six is already even.
**And when** 2.671 is rounded away from zero and 2.679 is rounded towards zero.
**Then** the results are 2.68 and 2.67.
**And when** a method that is none of the five is named.
**Then** the call fails with a message naming the unknown method.
**And** the default method, and the only one money uses unless a caller names another, is half away
from zero.
(`MCUR-050`; [calculations.md](calculations.md) section 3.4)

### MCUR-AC-284: rendering is not rounding

**Given** an amount that has not been rounded onto its currency.
**When** it is rendered as text.
**Then** the renderer does not round it: a rebuild must round first and render second.
**And** the renderer must not use a shortest-representation conversion, because such a conversion
silently drops significant digits on large values.
(`MCUR-051`)

### MCUR-AC-285: a rounding factor is inverted exactly

**Given** the rounding factors one hundredth, one thousandth and one ten-thousandth.
**When** each is inverted by the rounding routine.
**Then** the results are exactly one hundred, one thousand and ten thousand, taken from the table of
thirty exact reciprocals.
**And when** a factor that is not in the table is inverted, for example five hundredths.
**Then** the coefficient-and-exponent rule produces exactly twenty.
**And when** the factor is one.
**Then** no inversion happens at all, because the factor is not smaller than one.
(`MCUR-052`; [calculations.md](calculations.md) section 3.2)

### MCUR-AC-286: an exact euclidean division

**Given** a dividend of 268.50, a divisor of 12.25 and a factor of one hundredth.
**When** the euclidean division is performed.
**Then** the quotient is 21 and the remainder is 11.25, and the postcondition holds exactly:
21 × 12.25 + 11.25 = 268.50.
(`MCUR-053`; [calculations.md](calculations.md) section 6)

### MCUR-AC-287: residual amounts exist only on reconcilable and cash-like accounts

**Given** a posted journal item of 100.00 on an expense account that allows neither reconciliation
nor cash handling.
**When** the item is read.
**Then** `amount_residual` is 0.00, `amount_residual_currency` is 0.00 and `reconciled` is false,
all three forced, and the item can never take part in a matching.
**And given** an item on a cash account, which allows no reconciliation but is of the cash type.
**Then** both residuals are computed normally and the item may be matched.
(`MCUR-073`, `MCUR-134`)

### MCUR-AC-288: matched foreign amounts are rounded inside the aggregation

**Given** a journal item in a currency whose rounding factor is one hundredth, matched by a long
chain of partial matchings whose matched amounts in that currency each carry more decimal places
than the currency allows.
**When** `amount_residual_currency` is computed.
**Then** the sums of the matched amounts in the debit currency and in the credit currency are each
rounded to the decimal places of the currency concerned **inside** the aggregation, before being
subtracted from the amount in currency.
**And** an implementation that sums first and rounds at the end may differ by one unit in the last
place over such a chain, which is not the specified behaviour.
(`MCUR-074`)

### MCUR-AC-289: which of the two amounts is derived

**Given** a miscellaneous journal entry dated 1 January 2017 whose document currency is `FRG`,
whose applicable rate is 2.0.
**When** a line is written supplying only `balance = 120.00`.
**Then** `amount_currency` becomes 240.000.
**And when** a line is written supplying only `amount_currency = 240.000`.
**Then** `balance` becomes 120.00.
**And when** a line is written supplying **both** `balance = 120.00` and `amount_currency =
239.000`.
**Then** both are kept exactly as supplied, because neither is re-derived from the other.
**And when** a line is written supplying **neither**.
**Then** both remain 0.00, and the sign check of `MCUR-060` is satisfied trivially.
(`MCUR-075`, `MCUR-065`)

### MCUR-AC-290: the rate date of a document

**Given** a draft customer invoice in `FRG` with no invoice date, on a day whose `FRG` rate is 2.0.
**When** `expected_currency_rate` is read.
**Then** it is 2.0, evaluated at today in the reader's time zone.
**And when** `invoice_date` is set to 1 January 2016, whose rate is 3.0.
**Then** `expected_currency_rate` becomes 3.0 and `invoice_currency_rate` follows.
(`MCUR-087`, `MCUR-081`)

### MCUR-AC-291: the expected rate of a document with no currency is one

**Given** a journal entry that carries no document currency at all.
**When** `expected_currency_rate` is read.
**Then** it is exactly 1, and no rate lookup is performed.
(`MCUR-088`)

### MCUR-AC-292: the entry date is raised by every submitted line

**Given** a reconciliation batch producing two exchange difference instructions, one for an item
dated 31 January 2026 and one for an item dated 28 February 2026, and the second instruction is
zero at its own precision and is therefore skipped.
**When** the exchange difference entry is created.
**Then** its accounting date is raised to 28 February 2026 even though the item dated 28 February
2026 produced no line, because the date is raised for every item submitted with the batch.
**And** an implementation that raises the date only for the items that produce lines dates the
entry one period earlier, which is not the specified behaviour.
(`MCUR-117`)

### MCUR-AC-293: the residuals are decremented after the exchange handling

**Given** the reconciliation of `MCUR-AC-100`, whose matched amount is 1052.63 in the company
currency and 1000.00 in each document currency.
**When** the running residuals are inspected immediately after the matching is computed and before
the next pair is taken.
**Then** the debit side has lost 1052.63 of company currency residual and 1000.00 of debit currency
residual, and the credit side has gained 1052.63 and 1000.00 respectively.
**And** a side is dropped from the matching loop only when **both** of its remaining amounts are
zero at their own currencies' precisions.
(`MCUR-115`)

### MCUR-AC-294: the currency a matching is measured in

**Given** a debit item in `FRG` and a credit item in `FRG`, both offering a residual in `FRG`.
**When** the pair is matched.
**Then** the reconciliation currency is `FRG`.
**And given** a debit item in `FRG` whose foreign residual is nil and a credit item in `FRG`.
**Then** the reconciliation currency is the company currency, because the debit side cannot offer a
residual in `FRG`.
**And given** a debit item in the company currency and a credit item in `FRG` that can offer a
residual in `FRG`.
**Then** the reconciliation currency is `FRG`, the debit item being expressed in it through the
mirror rate.
**And given** a chosen currency that one side cannot offer after all.
**Then** the pairing is abandoned: the side that lacks it is marked as having nothing left and the
loop advances to the next item on that side.
(`MCUR-116`)

### MCUR-AC-295: the correction line inherits the matching of the item it corrects

**Given** the exchange difference entry of `MCUR-AC-100`.
**When** its two lines are read.
**Then** the correction line carries the account, the counterparty and the currency of the invoice's
receivable item, and inherits that item's full reconciliation link;
**And** the counterpart line carries the same counterparty and the same currency;
**And** the counterpart line carries the analytic distribution supplied by the caller when one was
supplied, and none otherwise, while the correction line never carries one.
(`MCUR-118`)

### MCUR-AC-296: a payment moves to paid and back as its matchings come and go

**Given** an inbound Payment of 1000.00 `EUR` in the `in_process` state, with no outstanding
account, whose counterpart item is about to be matched.
**When** a Partial Reconciliation is created whose matched amount in the debit currency is 1000.00
`EUR`, equal to the payment's signed amount at the payment currency's precision.
**Then** the payment's `state` becomes `paid`.
**And when** that Partial Reconciliation is deleted.
**Then** the payment's `state` returns to `in_process`.
**And given** instead one payment of 1500.00 `EUR` settling two documents of 1000.00 `EUR` and
500.00 `EUR`.
**Then** no single matching equals the payment amount, and the payment becomes `paid` only when the
sum of the matched amounts across the documents it settles equals 1500.00 at the payment currency's
precision.
(`MCUR-138`; [state-machines.md](state-machines.md) section 9)

### MCUR-AC-297: an imported matching key is deferred until every entry is posted

**Given** two journal items imported with the matching key "A1", each belonging to a draft entry.
**When** they are created.
**Then** each carries a `matching_number` of `IA1`, the letter `I` having been prefixed to the
supplied key, and no Partial Reconciliation exists.
**And when** an attempt is made to give such a mark to an item that already has matchings.
**Then** the write is refused with "A temporary number can not be used in a real matching".
**And when** every entry carrying the mark has been posted.
**Then** the real matching is performed, with exchange differences and cash basis entries suppressed
for that operation, the account is switched to allow reconciliation when it did not, and the
matching numbers become `P` followed by an identifier, or the identifier of the Full Reconciliation
when the group closed.
([state-machines.md](state-machines.md) section 8)

### MCUR-AC-298: trailing zeros are suppressed only when the caller asks

**Given** a currency with two decimal places and the amount 1234.50.
**When** it is formatted without asking for trailing zeros to be suppressed.
**Then** the numeric part reads "1,234.50" in a language grouping by three with a comma and a
decimal point.
**And when** the amount 1234.00 is formatted with trailing zeros suppressed.
**Then** the numeric part reads "1,234", the run of trailing zeros having been removed together
with the decimal separator that preceded it.
(`MCUR-212`)

### MCUR-AC-299: the compact metric rendering

**Given** a currency whose symbol is a dollar sign placed before the amount.
**When** the values 123456.789, 123000.789, −123456.789 and 0.789 are rendered compactly.
**Then** the results are "123.5k", "123k", "-123.5k" and "0.8", and the first becomes "$123.5k"
once the symbol is attached.
**And when** a value larger than one million million is rendered.
**Then** the suffix stops at the fourth metric step and the number grows instead.
(`MCUR-215`; [calculations.md](calculations.md) section 24.5)

### MCUR-AC-300: a transaction executed in the company currency values from the transacted amount

**Given** a company whose main currency is `USD`, a bank journal whose currency is `EUR`, and a
statement line dated 10 April 2026 with `amount = 850.00` in `EUR`, `foreign_currency_id = USD` and
`amount_currency = 923.00`.
**When** the journal items are generated.
**Then** the company currency value is 923.00 and not the conversion of 850.00 `EUR` at the rate
table's rate, because the transacted currency is itself the company currency;
**And** the liquidity item carries `EUR` with an amount in currency of +850.00 and a debit of
923.00, while the counterpart item carries `USD` with an amount in currency of −923.00 and a credit
of 923.00.
(`MCUR-156`)

### MCUR-AC-301: the grouping pattern of the reader's language is applied

**Given** a language whose grouping pattern is `[3,0]` and whose thousands separator is a comma.
**When** one hundred twenty-three million four hundred fifty-six thousand seven hundred eighty-nine
is formatted.
**Then** the whole part reads "123,456,789".
**And given** a language whose grouping pattern is `[3,2,0]`.
**Then** the same value reads "12,34,56,789", because a zero element repeats the preceding group
size until the digits are exhausted.
**And given** a pattern whose second element is negative, `[2,-1]`, on the digits one two three four
five six and a fractional part of seven eight.
**Then** the result is "123456.78", because a negative element stops the grouping immediately.
(`MCUR-211`; [entities.md](entities.md) section 14.1)

### MCUR-AC-302: a currency with a rounding factor of five hundredths snaps onto five-cent multiples

**Given** a currency whose `rounding` is 0.05, whose `decimal_places` is therefore 2.
**When** the amounts 1.32, 1.33 and 1.30 are rounded onto it.
**Then** the results are 1.30, 1.35 and 1.30, so the reachable values are the multiples of five
hundredths even though two fractional digits are displayed.
**And given** a hypothetical currency whose `rounding` is 5, whose `decimal_places` is therefore 0.
**When** 1234.56 is rounded onto it.
**Then** the result is 1235, and the rounding onto multiples of five is applied even though nothing
in the rendered text shows it.
(`MCUR-003`, `MCUR-005`; [entities.md](entities.md) section 1.5)

---

## Reconciliation notes

1. **Provenance.** One draft carried this file with two hundred and sixty-four scenarios; the other
   carried no acceptance criteria at all, stating its cases as worked examples inside its entity and
   calculation documents. Every scenario of the first draft is kept, with its identifier unchanged,
   and every worked example of the second is already carried by
   [calculations.md](calculations.md) sections 3.3 to 3.4, 8.3, 13.6 to 13.8, 14.6, 14.7 and 26 to
   28, which this file cites rather than repeats.
2. **Rule identifiers.** The scenarios cited rules of the form `MCUR-RULE-nnn`. They now cite the
   consolidated `MCUR-nnn` scheme; the mapping of the former identifiers is in
   [business-rules.md](business-rules.md) section 13.
3. **Field identifiers.** Every field named in a scenario now uses the identifier under which the
   value is stored and transported, and the table at the head of this file gives the full name of
   each in words.
4. **Messages.** A message is written between quotation marks, not in code font, because code font
   in this repository marks a reproduced identifier and quotation marks mark reproduced text. The
   placeholders inside a message are written in italics.
5. **Section references.** Every reference into [calculations.md](calculations.md) follows the
   consolidated numbering of that file.
6. **Scenarios added in consolidation.** Group O covers the rules that neither draft exercised with
   a numbered scenario: `MCUR-017`, `MCUR-018`, `MCUR-019`, `MCUR-033` to `MCUR-039`, `MCUR-047`,
   `MCUR-048`, `MCUR-049` to `MCUR-053`, `MCUR-073` to `MCUR-075`, `MCUR-087`, `MCUR-088`,
   `MCUR-115` to `MCUR-118`, `MCUR-138`, `MCUR-211`, `MCUR-212` and `MCUR-215`, together with the
   matching-number import mark and the exception in `MCUR-156`.
