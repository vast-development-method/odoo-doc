# Multi-Currency

## Scope

This domain specifies how the system represents money.

Every monetary amount the platform stores is a pair: a number and a **Currency**
(`res.currency`, table `res_currency`). The number alone is meaningless. A currency in this
system is not merely a label: it is the object that decides how many decimal digits an amount
keeps, what multiple an amount is snapped onto, how two amounts are compared, when an amount
counts as zero, how the amount is rendered as text for a human reader, and — through its dated
rate records — what the amount is worth in another currency on a given day for a given company.

Because accounting is legally kept in one currency per company, every amount that reaches the
ledger exists twice: once in the currency the document was agreed in, and once in the currency
the company reports in. The two copies are tied together by a rate captured at a precise moment.
When the rate moves between the moment a receivable is recognised and the moment it is settled,
the two copies stop agreeing, and the system must recognise the difference as a realised gain or
loss. This domain specifies that whole chain end to end.

Concretely the domain covers:

- **The currency entity** — its three-letter code, its numeric code, its display name, its
  symbol, its symbol position, its rounding factor, its derived number of decimal places, its
  unit and subunit labels for amounts written out in words, its archival flag, and the
  uniqueness and positivity constraints that guard it.
- **The rounding primitives** — the single rounding routine the whole platform shares, written
  out as arithmetic including its error-compensation term; the five rounding methods; the
  zero test; the three-way comparison; the euclidean division; the string rendering; and the
  accurate reciprocal used to avoid dividing by small fractions. These are the same primitives
  the quantity side of the platform uses, and they are documented from the quantity side in
  [../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md).
  This file agrees with that one in every particular and adds the monetary specifics: the
  rounding factor comes from the currency rather than from a decimal-precision record, the
  factor may be a whole number rather than a fraction, and the derived number of decimal places
  is used for string rendering.
- **Currency rates** — the dated rate record, its company scoping to root companies only, its
  three interchangeable representations (the technical rate, the company rate and the inverse
  company rate), the sanitising rule that decides which of the three wins when more than one is
  supplied, the large-movement warning, the uniqueness constraint per day, and the positivity
  check.
- **The rate lookup algorithm** — the exact rule that, given a currency, a company and a date,
  produces a number: the latest rate on or before the date belonging to the company or shared by
  all companies, falling back to the earliest rate of any date, falling back to one.
- **Conversion** — the conversion of an amount from one currency to another, with the
  cross-rate arithmetic that routes any pair of foreign currencies through the company currency,
  the exact rounding of the result, and the short-circuits for a zero amount and for identical
  currencies.
- **Multi-currency journal items** — the pair of amounts every journal item carries, the rate
  field that binds them, the invariants that the pair must satisfy, the sign rules, and the case
  where the document currency equals the company currency.
- **The document rate** — the rate captured on an invoice or bill, the date it is taken at, the
  fact that it is stored and thereafter drives every line of the document, the constraint that
  forbids a non-positive rate, and the recomputation triggers.
- **Reconciliation across currencies** — how the residual amounts of two journal items in
  different currencies are matched, which currency the match is performed in, how the matched
  amounts are computed in each currency, the tolerance band that suppresses a spurious
  difference, and the exact exchange difference journal entry that is produced with its journal,
  its accounts, its sides, its amounts, its date and its reconciliation links.
- **Unreconciliation** — the reversal of the exchange difference entry, the conditions under
  which it is reversed rather than deleted, and the date the reversal is dated at.
- **Currency at every document boundary** — where a conversion happens and which date and rate
  it uses: invoice lines, tax lines, payment terms, payments, bank statement lines, analytic
  items, and reports.
- **Company currency rules** — the constraint that forbids changing a company currency once
  journal items exist, the constraint that forbids deactivating a currency a company uses, the
  automatic activation of the multi-currency permission group, and the exchange journal and
  exchange accounts a company must configure.
- **Amount formatting** — the rendering of a number as a human-readable amount under the
  reader's language: the decimal separator, the digit-grouping pattern, the thousands separator,
  the non-breaking spaces, the symbol placement, the trailing-zero suppression option, the
  compact metric rendering, and the rendering of an amount in words.
- **The complete catalogue of shipped currencies** — all one hundred and seventy currencies
  delivered as reference data with their symbol, rounding factor, derived decimal places and
  symbol position, plus the three unit-of-account currencies a country localization adds.

## What this domain is *not*

- It does not specify the chart of accounts, the journal entry state machine, the posting rules
  or the reconciliation model itself. Those belong to
  [../general-ledger/README.md](../general-ledger/README.md). This domain specifies only the
  currency behaviour *within* them.
- It does not specify how a payment is registered or how a bank statement is imported. Those
  belong to
  [../payments-and-bank-reconciliation/README.md](../payments-and-bank-reconciliation/README.md).
  This domain specifies the currency conversion those flows perform.
- It does not specify tax arithmetic. That belongs to [../taxes/README.md](../taxes/README.md).
- It does not specify pricelist currency conversion beyond the shared conversion routine; the
  pricelist rules themselves belong to the pricing domain.

## Entities

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Currency | `res.currency` | `res_currency` | A unit of money with its symbol, its rounding factor, its derived decimal places and its dated rates. |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | The value of one currency against the reference on one date for one root company. |
| Company (currency aspects) | `res.company` | `res_company` | Carries the reporting currency, the exchange journal and the two exchange difference accounts. |
| Journal Item (currency aspects) | `account.move.line` | `account_move_line` | Carries the pair of amounts, the document currency and the captured rate. |
| Journal Entry (currency aspects) | `account.move` | `account_move` | Carries the document currency and the captured document rate. |
| Partial Reconciliation (currency aspects) | `account.partial.reconcile` | `account_partial_reconcile` | Records a match with three amounts: one in the company currency and one in each side's currency. |
| Language (formatting aspects) | `res.lang` | `res_lang` | Supplies the decimal separator, the digit grouping and the thousands separator used when an amount is rendered. |

## Reading order

1. **[entities.md](entities.md)** — every field of every entity above, with its type, its
   defaults, its computation and its constraints.
2. **[calculations.md](calculations.md)** — the arithmetic core: the rounding routine, the
   comparison, the zero test, the rate lookup, the conversion, the cross-rate, the residual
   arithmetic and the exchange difference amounts, each with worked numeric examples and the
   complete catalogue of shipped currencies.
3. **[state-machines.md](state-machines.md)** — the state fields this domain touches: the
   archival state of a currency, the lifecycle of a rate, the reconciliation state of a journal
   item and the posting state of an exchange difference entry.
4. **[workflows.md](workflows.md)** — the operational sequences: entering a rate, invoicing in a
   foreign currency, paying at a different rate, reconciling, unreconciling, converting between
   two foreign currencies, and closing at a period-end rate.
5. **[business-rules.md](business-rules.md)** — every validation, every invariant and every
   error message.
6. **[accounting-effects.md](accounting-effects.md)** — the exchange difference journal entry
   specified line by line, plus every other place a currency conversion changes a posted amount.
7. **[configuration.md](configuration.md)** — the settings, the permission group, the access
   matrix, the record rules and the scheduled work.
8. **[interfaces.md](interfaces.md)** — the navigation, the views, the named remote operations,
   the data contracts the client receives and the import and export formats.
9. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given/When/Then scenarios with
   concrete numbers.
10. **[glossary.md](glossary.md)** — every term defined in full.

## Dependencies on other domains

| Depends on | For what |
|---|---|
| [../identity-and-access/README.md](../identity-and-access/README.md) | Companies, the root-company concept, permission groups and record rules. |
| [../general-ledger/README.md](../general-ledger/README.md) | Journal entries, journal items, accounts, journals, reconciliation and posting. |
| [../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md) | The shared rounding primitives, documented there from the quantity side. |

## Domains that depend on this one

| Domain | For what |
|---|---|
| [../general-ledger/README.md](../general-ledger/README.md) | Residual amounts in two currencies, the exchange difference entry, the balance invariant. |
| [../accounts-receivable/README.md](../accounts-receivable/README.md) | Invoice currency, the document rate, payment term amounts in two currencies. |
| [../accounts-payable/README.md](../accounts-payable/README.md) | Bill currency, vendor amounts, the document rate. |
| [../payments-and-bank-reconciliation/README.md](../payments-and-bank-reconciliation/README.md) | Payment currency, statement line currency, the write-off in a foreign currency. |
| [../taxes/README.md](../taxes/README.md) | Tax amounts held in two currencies and rounded per currency. |
| [../analytic-accounting/README.md](../analytic-accounting/README.md) | Analytic items are valued in the company currency; balances across companies are converted at today's rate. |
| [../inventory-valuation-and-costing/README.md](../inventory-valuation-and-costing/README.md) | Costs converted from a purchase currency to the company currency. |
| [../pricing-and-pricelists/README.md](../pricing-and-pricelists/README.md) | Prices converted between the pricelist currency and the document currency. |
| [../financial-reporting/README.md](../financial-reporting/README.md) | Report amounts converted and rounded for presentation. |

## The three numbers that matter

A rebuild that gets nothing else right must get these three right, because everything else is
built on them.

1. **The rounding factor.** Each currency declares the multiple that its amounts snap onto. For
   most currencies it is one hundredth. For fifteen currencies it is one. For seven it is one
   thousandth. For three it is one ten-thousandth. The rounding factor is authoritative; the
   number of decimal places is *derived from it*, never the other way round.

   ```formula
   decimal_places = ceiling( log10( 1 ÷ rounding_factor ) )   when 0 < rounding_factor < 1
   decimal_places = 0                                          otherwise
   ```

2. **The rate direction.** A stored rate answers the question "how many units of this currency
   do I get for one unit of the reference?". Conversion from the company currency *to* a foreign
   currency therefore multiplies by the rate; conversion from a foreign currency *to* the company
   currency divides by it. The system exposes the divided form as a separate derived number so
   that both directions are available without a division at the point of use.

3. **Rounding happens once, at the end.** A conversion multiplies first and rounds once, onto the
   destination currency's rounding factor. A conversion between two foreign currencies performs
   *one* multiplication by a composed cross-rate and *one* rounding, not two roundings.

Each of these is stated as arithmetic, with worked numbers, in
[calculations.md](calculations.md).
