# Multi-Currency

This domain specifies how the system represents money: what a currency is, what one unit of a
currency is worth on a given day for a given company, how an amount is rounded, compared,
converted and written out, how every journal item carries its amount twice — once in the currency
the document was agreed in and once in the currency the company keeps its books in — and how the
difference between those two copies is recognised in the ledger when the rate moves between the
moment a balance is recorded and the moment it is settled.

Every monetary amount the platform stores is a pair: a number and a Currency (`res.currency`,
table `res_currency`). The number alone is meaningless. A currency in this system is not merely a
label: it is the object that decides how many decimal digits an amount keeps, what multiple an
amount is snapped onto, how two amounts are compared, when an amount counts as zero, how the
amount is rendered for a human reader, and — through its dated rate records — what the amount is
worth in another currency on a given day for a given company.

Because accounting is legally kept in one currency per company, every amount that reaches the
ledger exists twice. The two copies are tied together by a rate captured at a precise moment. When
the rate moves between the moment a receivable is recognised and the moment it is settled, the two
copies stop agreeing, and the system must recognise the difference as a realised gain or loss.
This domain specifies that whole chain end to end.

## The five questions this domain answers for the whole platform

1. **What is a currency?** A three-letter code of the international currency-code standard, a
   symbol, a placement rule for the symbol, a rounding factor, the number of decimal places
   derived from that rounding factor, a unit name and a subunit name for writing amounts in words,
   and an activity flag.
2. **What is one unit of a currency worth, and when?** A per-company, per-date rate table. Each
   row states a rate valid from its date until the next row supersedes it. The rate used for a
   given date is the one from the latest row dated on or before that date.
3. **How is an amount converted?** A single multiplicative formula using two rate lookups,
   followed by one rounding onto the target currency's rounding factor.
4. **How is a foreign currency amount carried in the ledger?** Every journal item stores both a
   balance in the company currency and an amount in the document currency; the two must agree in
   sign, and their ratio is the implied rate at which the item was recorded.
5. **What happens when the rate moves between recording and settlement?** A realised exchange
   difference journal entry is generated automatically at reconciliation time, posted to a
   dedicated journal and to a gain or a loss account, and reversed when the reconciliation is
   undone.

## Scope

Concretely the domain covers:

- **The currency entity** — its three-letter code, its numeric code, its display name, its symbol,
  its symbol position, its rounding factor, its derived number of decimal places, its unit and
  subunit labels for amounts written out in words, its activity flag, and the uniqueness and
  positivity constraints that guard it.
- **The rounding primitives** — the single rounding routine the whole platform shares, written out
  as arithmetic including its error-compensation term; the five rounding methods; the zero test;
  the three-way comparison; the exact euclidean division; the string rendering; and the accurate
  reciprocal used to avoid dividing by a small fraction. These are the same primitives the
  quantity side of the platform uses, and they are documented from the quantity side in
  [../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md).
  This folder agrees with that one in every particular and adds the monetary specifics: the
  rounding factor comes from the currency rather than from a decimal-precision record, the factor
  may be a whole number rather than a fraction, and the derived number of decimal places is used
  for string rendering.
- **Currency rates** — the dated rate record, its company scoping to root companies only, its
  three interchangeable representations (the technical rate, the company rate and the inverse
  company rate), the sanitising rule that decides which of the three wins when more than one is
  supplied, the large-movement warning, the uniqueness constraint per day, and the positivity
  check.
- **The rate lookup algorithm** — the exact rule that, given a currency, a company and a date,
  produces a number: the latest rate on or before the date belonging to the company or shared by
  all companies, falling back to the earliest rate of any date, falling back to one.
- **Conversion** — the conversion of an amount from one currency to another, with the cross-rate
  arithmetic that routes any pair of foreign currencies through the reference of rate one in a
  single multiplication, the exact rounding of the result, and the short-circuits for a zero
  amount and for identical currencies.
- **Currency-aware arithmetic** — rounding an amount, comparing two amounts and testing an amount
  for zero at the currency's own precision, with precisely defined tie-breaking.
- **Multi-currency journal items** — the pair of amounts every journal item carries, the rate
  field that binds them, the invariants that the pair must satisfy, the sign rules, and the case
  where the document currency equals the company currency.
- **The document rate** — the rate captured on an invoice or a bill, the date it is taken at, the
  fact that it is stored and thereafter drives every line of the document, the constraint that
  forbids a non-positive rate, the recomputation triggers and the manual override.
- **Reconciliation across currencies** — how the residual amounts of two journal items in
  different currencies are matched, which currency the match is performed in, how the matched
  amounts are computed in each currency, the tolerance band that suppresses a difference that is
  only a rounding artefact, and the exact exchange difference journal entry that is produced with
  its journal, its accounts, its sides, its amounts, its date and its reconciliation links.
- **Unreconciliation** — the reversal of the exchange difference entry, the conditions under which
  it is reversed rather than deleted, and the date the reversal is dated at.
- **Currency at every document boundary** — where a conversion happens and which date and rate it
  uses: invoice lines, tax lines, payment terms, payments, bank statement lines, analytic items
  and reports.
- **Multi-currency bank transactions** — a bank transaction represented in up to three currencies
  at once: the company currency, the bank account's own currency and the currency actually
  transacted.
- **Company currency rules** — the constraint that forbids changing a company currency once
  journal items exist, the constraint that forbids deactivating a currency a company uses, the
  automatic granting of the multi-currency permission group, and the exchange journal and exchange
  accounts a company must configure.
- **Amount formatting and spelling** — the rendering of a number as a human-readable amount under
  the reader's language: the decimal separator, the digit-grouping pattern, the thousands
  separator, the non-breaking spaces, the symbol placement, the trailing-zero suppression option,
  the compact metric rendering, and the rendering of an amount in words.
- **Reporting rate tables** — the current, historical and average conversion factors used to
  consolidate figures produced by companies whose main currencies differ.
- **The complete catalogue of shipped currencies** — all one hundred and seventy currencies
  delivered as reference data with their numeric code, symbol, rounding factor, derived decimal
  places, symbol position and unit labels, the complete demonstration rate table, and the
  currencies a country package adds or activates.

## What this domain is not

- It does not specify the chart of accounts, the journal entry state machine, the posting rules or
  the reconciliation engine itself. Those belong to
  [../general-ledger/README.md](../general-ledger/README.md) and
  [../payments-and-bank-reconciliation/README.md](../payments-and-bank-reconciliation/README.md).
  This domain specifies only the currency behaviour *within* them.
- It does not specify how a payment is registered or how a bank statement is imported. This domain
  specifies the currency conversion those flows perform.
- It does not specify tax arithmetic. That belongs to [../taxes/README.md](../taxes/README.md).
- It does not specify price list rules beyond the shared conversion routine; those belong to
  [../pricing-and-pricelists/README.md](../pricing-and-pricelists/README.md).
- It does not own the chart of accounts, the journals or the journal entries; it owns the currency
  dimension of all of them.

## Capabilities delivered

| Capability | Summary |
|---|---|
| Currency master data | Create, edit, activate and deactivate currencies. One hundred and seventy currencies are shipped as reference data, every one of them inactive, which leaves a freshly configured platform with no active currency at all. A currency becomes active only through a later action: a company adopting it as its main currency (`MCUR-009`), a chart of accounts template or a country package that activates the currency of its country, or an explicit activation by a user. |
| Dated rate table | Record a rate for a currency on a date, optionally scoped to one root company; enter it either as units of the foreign currency per one unit of company currency or as the inverse; guard against an implausible rate entry. |
| Deterministic conversion | Convert any amount between any two currencies for a given company and date, with an exact, reproducible formula and a defined rounding step. |
| Currency-aware arithmetic | Round an amount, compare two amounts and test an amount for zero, all at the currency's own precision, with precisely defined tie-breaking. |
| Amount formatting and spelling | Render an amount with its symbol, grouping and decimal places; spell an amount in words using the unit and subunit labels for legal documents. |
| Foreign currency ledger entries | Carry an amount in the document currency next to the company currency balance on every journal item, with sign consistency enforced by a stored database check. |
| Document rate override | Record and, where allowed, manually override the rate applied to an invoice or a bill, and re-derive every line balance from it. |
| Realised exchange differences | Detect the residual arising from a rate movement during reconciliation, post a balanced exchange difference entry to the configured journal and gain or loss account, link it to the matching record, and reverse it on unreconciliation. |
| Multi-currency bank transactions | Represent a bank transaction in up to three currencies at once: the company currency, the bank account's own currency and the currency actually transacted. |
| Reporting rate tables | Produce current, historical and average conversion factors used to consolidate figures from companies whose main currencies differ. |
| Currency guards | Prevent deactivating a currency in use by a company, prevent lowering a currency's precision once it has been used to round accounting entries, and prevent changing a company's currency once entries exist. |

## Actors

| Actor | Responsibilities in this domain |
|---|---|
| Accounting Manager | Creates and activates currencies, maintains the rate table, configures the exchange difference journal and the gain and loss accounts, chooses the main currency of a company before any entry exists. |
| Accountant | Records invoices, bills, payments and miscellaneous entries in foreign currencies; overrides a document rate when a contract fixes it; reconciles foreign currency balances and reviews the generated exchange difference entries. |
| Settings Administrator | Enables the automatic rate retrieval capability, chooses the rate service and its interval, triggers an immediate rate refresh. |
| Any internal user | Reads currencies and rates, because every monetary field displayed needs them. |
| Portal user and public visitor | Reads currencies and rates, because published prices and portal documents display monetary amounts. |
| Scheduled rate updater | A background job that fetches rates from an external rate service at the configured interval and writes new Currency Rate records. |

## Entities this folder owns

| Entity | Transport name | Table | Reference page | One-line purpose |
|---|---|---|---|---|
| Currency | `res.currency` | `res_currency` | [../../references/entities/res.currency.md](../../references/entities/res.currency.md) | A unit of money with its symbol, its rounding factor, its derived decimal places and its dated rates. |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | [../../references/entities/res.currency.rate.md](../../references/entities/res.currency.rate.md) | The value of one currency against the reference of rate one on one date for one root company. |

Both entities, and every field of both, are specified in [entities.md](entities.md).

## Entities owned elsewhere whose currency behaviour this folder specifies

This domain adds no persistent field to an entity owned by another domain, but it defines the
meaning, the derivation and the constraints of every currency-related field on the entities below.
The owning folder defines the rest of each entity.

| Entity | Transport name | Owning folder | Currency aspects specified here |
|---|---|---|---|
| Journal Entry | `account.move` | [../general-ledger/](../general-ledger/) | The document currency, the company currency, the expected rate, the stored and overridable document rate, the signed totals in both currencies, and the archived-currency posting guard. |
| Journal Item | `account.move.line` | [../general-ledger/](../general-ledger/) | The item currency, the company currency, the amount in currency, the balance, the debit and the credit, the item rate, the same-currency flag, the two residual amounts, the sign consistency check and the derivation of the two amounts from the document rate. |
| Partial Reconciliation | `account.partial.reconcile` | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/) | The company currency, the debit and credit currencies, the three matched amounts, the link to the exchange difference entry, and the rule that the three amounts are always positive. |
| Full Reconciliation | `account.full.reconcile` | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/) | The closure test that decides, per currency situation, whether a group of matched items is fully reconciled. |
| Company | `res.company` | [../contacts-and-organizations/](../contacts-and-organizations/) | The main currency, its delegation from the root company, its automatic activation, the rule that it cannot change once entries exist, the exchange journal, the two exchange accounts and the two invoice presentation settings. |
| Journal | `account.journal` | [../general-ledger/](../general-ledger/) | The journal currency: when set and different from the company currency, the journal handles only that currency and forces it onto its liquidity and payment accounts. |
| Account | `account.account` | [../general-ledger/](../general-ledger/) | The account currency: when set, every journal item on the account must carry that currency; the consistency rule with the journal currency; the guard against setting it once items with another currency exist. |
| Bank Statement Line | `account.bank.statement.line` | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/) | The journal currency, the foreign currency, the two amounts, the three-currency derivation of the generated journal items and the bank-implied rates used for counterpart amounts. |
| Payment | `account.payment` | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/) | The payment currency, the company currency, the signed company-currency amount and the conversion of the payment amount at the payment date. |
| Price List | `product.pricelist` | [../pricing-and-pricelists/](../pricing-and-pricelists/) | The price list currency, the conversion of a public price into it, and the automatic archiving of price lists whose currency is deactivated. |
| Language | `res.lang` | [../contacts-and-organizations/](../contacts-and-organizations/) | The decimal separator, the thousands separator and the digit grouping pattern used when a monetary amount is rendered. |

Generic platform entities — those whose transport name begins with `ir.`, `base.`, `report.`,
`format.`, `properties.` or `change.` — belong to the platform foundation and to the overview
documents, not to this folder, even though they appear beside this domain's entities in the
package that ships them.

## Reading order

1. **[entities.md](entities.md)** — every field of every entity above, with its identifier, its
   full name, its type, its default, its derivation and its constraints.
2. **[state-machines.md](state-machines.md)** — the twelve state-bearing fields this domain owns
   or drives: the activity state of a currency, its precision latch, the multi-currency capability
   state of the platform, the lifecycle of a rate row, the currency and rate state of a document,
   the state of an exchange difference entry, the reconciliation state of a journal item seen from
   the currency side, its matching number, the settlement state of a payment, the payment state of
   a document, the currency configuration state of a bank transaction and the main currency state
   of a company.
3. **[workflows.md](workflows.md)** — the nineteen operational procedures, step by step:
   activating a currency, recording a rate, refreshing rates on a schedule, setting a company's
   main currency, configuring the exchange journal and accounts, invoicing and billing in a foreign
   currency, overriding a document rate, paying at a different rate, reconciling, creating the
   exchange difference entry, unreconciling, recording and matching a foreign currency bank
   transaction, changing a rounding factor, deactivating a currency, converting between two foreign
   currencies, closing a period at consolidation rates and posting the unrealised revaluation.
4. **[business-rules.md](business-rules.md)** — every validation, every invariant, every permission
   check and every exact error message, numbered `MCUR-nnn`.
5. **[calculations.md](calculations.md)** — the arithmetic core: the rounding routine, the
   comparison, the zero test, the euclidean division, the rate lookup, the conversion, the
   cross-rate, the three rate representations, the document rate, the residual arithmetic, the
   reconciliation currency choice, the partial amounts, the exchange difference amounts, the entry
   and reversal dates, the reporting rate tables, the bank transaction amounts, the formatting and
   the spelling — each with worked numeric examples.
6. **[accounting-effects.md](accounting-effects.md)** — the exchange difference entry specified
   item by item, its reversal, and every other place a currency conversion changes a posted
   amount.
7. **[configuration.md](configuration.md)** — the settings, the shipped currency catalogue, the
   demonstration rate table, the permission groups, the access matrix, the menus and the scheduled
   work.
8. **[interfaces.md](interfaces.md)** — the named operations, the screens, the printed documents,
   the scheduled job, the notifications and the integration contracts.
9. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given, When and Then scenarios
   with concrete numbers.
10. **[glossary.md](glossary.md)** — every term of the domain, defined.

## Files in this folder

| File | Content |
|---|---|
| [README.md](README.md) | This file: scope, capabilities, actors, entities, reading order and dependencies. |
| [entities.md](entities.md) | Every field of Currency and Currency Rate, plus the complete currency-related field inventory of the entities owned elsewhere, with types, defaults, derivations, constraints, indexes, validation messages and lifecycles. |
| [state-machines.md](state-machines.md) | Every state-bearing field: states with stored value, label and meaning; transition tables with triggers, guards and side effects; a diagram per machine. |
| [workflows.md](workflows.md) | End-to-end procedures with the records each step writes, the operations invoked and the failure conditions. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with exact messages, guards, permissions and consistency rules, and the mapping table of former rule identifiers. |
| [calculations.md](calculations.md) | Every formula and algorithm with rounding, precision, currency handling and worked numeric examples. |
| [accounting-effects.md](accounting-effects.md) | The ledger entries this domain produces, item by item, and the currency dimension of the entries produced by neighbouring domains. |
| [configuration.md](configuration.md) | Settings, the shipped currency catalogue, the demonstration rate table, master data prerequisites, access groups, menus and the configuration checklist. |
| [interfaces.md](interfaces.md) | Named operations, screens, printed documents, the scheduled job, notifications and integration contracts. |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given, When and Then scenarios with concrete numbers. |
| [glossary.md](glossary.md) | Every term of the domain, defined. |

## Dependencies on other domains

| Depends on | For what |
|---|---|
| [../platform-foundation/README.md](../platform-foundation/README.md) | Persistence, the derived-field recomputation and caching mechanism, translation of the unit and subunit labels, scheduled job execution, the decimal precision registry and the access control primitives. The platform mechanisms themselves are described in [../../overview/entity-and-field-system.md](../../overview/entity-and-field-system.md) and [../../overview/security-model.md](../../overview/security-model.md). |
| [../contacts-and-organizations/README.md](../contacts-and-organizations/README.md) | The Company entity and its root-company hierarchy, on which the main currency lives and to which rate rows are scoped; and the Language entity, which supplies the number format. |
| [../identity-and-access/README.md](../identity-and-access/README.md) | The permission group mechanism; the multi-currency capability is itself a group that is granted and revoked automatically. |
| [../general-ledger/README.md](../general-ledger/README.md) | Journal Entry, Journal Item, Account, Journal, the posting engine, the lock date rules and the entry numbering rules that the exchange difference entry obeys. |
| [../payments-and-bank-reconciliation/README.md](../payments-and-bank-reconciliation/README.md) | The reconciliation engine that calls into the exchange difference algorithm, Partial Reconciliation, Full Reconciliation, Payment and Bank Statement Line. |
| [../taxes/README.md](../taxes/README.md) | Tax amounts computed in the document currency and translated with the document rate; the cash basis mechanism, which triggers exchange differences of its own. |
| [../pricing-and-pricelists/README.md](../pricing-and-pricelists/README.md) | Price lists expressed in a currency other than the company currency, which consume the conversion algorithm defined here. |
| [../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md) | The shared rounding primitives, documented there from the quantity side. |
| [../fiscal-localizations/README.md](../fiscal-localizations/README.md) | The country-specific fields added to a currency and the currencies a country package adds or activates. |

## Domains that depend on this one

| Domain | For what |
|---|---|
| [../general-ledger/README.md](../general-ledger/README.md) | Residual amounts in two currencies, the exchange difference entry, the balance invariant. |
| [../accounts-receivable/README.md](../accounts-receivable/README.md) | Invoice currency, the document rate, payment term amounts in two currencies. |
| [../accounts-payable/README.md](../accounts-payable/README.md) | Bill currency, vendor amounts, the document rate. |
| [../payments-and-bank-reconciliation/README.md](../payments-and-bank-reconciliation/README.md) | Payment currency, statement line currency, the write-off in a foreign currency. |
| [../taxes/README.md](../taxes/README.md) | Tax amounts held in two currencies and rounded per currency. |
| [../analytic-accounting/README.md](../analytic-accounting/README.md) | Analytic items valued in the company currency; balances across companies converted at the current rate. |
| [../inventory-valuation-and-costing/README.md](../inventory-valuation-and-costing/README.md) | Costs converted from a purchase currency to the company currency. |
| [../pricing-and-pricelists/README.md](../pricing-and-pricelists/README.md) | Prices converted between the price list currency and the document currency. |
| [../financial-reporting/README.md](../financial-reporting/README.md) | Report amounts converted and rounded for presentation, and the three consolidation rate types. |
| [../sales/README.md](../sales/README.md) and [../purchasing/README.md](../purchasing/README.md) | Order currency and the conversion of a catalogue price into the order currency. |
| [../point-of-sale/README.md](../point-of-sale/README.md) | The currency data loaded into a session, and the rounding of a cash payment. |
| [../electronic-invoicing-and-document-exchange/README.md](../electronic-invoicing-and-document-exchange/README.md) | The currency codes, the applied rate and the two amount columns written into an exchanged document. |

Every other domain that stores a monetary amount depends on this one for the rounding factor and
the decimal places of the currency it stores it in.

## The three numbers that matter

A rebuild that gets nothing else right must get these three right, because everything else is
built on them.

1. **The rounding factor.** Each currency declares the multiple that its amounts snap onto. For
   one hundred forty-two of the shipped currencies it is one hundredth. For eighteen it is one.
   For seven it is one thousandth. For three it is one ten-thousandth. The rounding factor is
   authoritative; the number of decimal places is *derived from it*, never the other way round.

   ```formula
   decimal places = ceiling( log10( 1 ÷ rounding factor ) )   when 0 < rounding factor < 1
   decimal places = 0                                          otherwise
   ```

2. **The rate direction.** A stored technical rate answers the question "how many units of this
   currency do I get for one unit of the reference of rate one?". Conversion from the company
   currency *to* a foreign currency therefore multiplies by the rate of the foreign currency
   divided by the rate of the company currency; conversion from a foreign currency *to* the
   company currency divides by that same factor. The system exposes the divided form as a separate
   derived number so that both directions are available without a division at the point of use.

3. **Rounding happens once, at the end.** A conversion multiplies first and rounds once, onto the
   destination currency's rounding factor. A conversion between two foreign currencies performs
   *one* multiplication by a composed cross-rate and *one* rounding, not two roundings.

Each of these is stated as arithmetic, with worked numbers, in [calculations.md](calculations.md).

## Reconciliation notes

These notes record how the two independently written drafts of this folder were brought together.
Each of the other files of the folder carries its own notes on the points its subject settles.

1. **The number of documents.** One draft was written to a ten-document standard with no separate
   state-machine document and stated its state tables at the end of its workflow document; the
   other was partial and carried three documents. This folder holds the eleven documents the
   writing charter prescribes: the state tables have been moved into
   [state-machines.md](state-machines.md) and expanded with stored values, labels, meanings and
   exact refusal messages, and [workflows.md](workflows.md) points at them.
2. **The scope statement.** Both drafts described the same domain in different words. The scope,
   the capability table, the actor table and the two entity tables above merge them; nothing either
   draft claimed for the domain has been dropped, and the boundaries with the general ledger, the
   payments domain, the tax domain and the pricing domain are stated once, under "What this domain
   is not".
3. **Folder names.** One draft linked to sibling folders under working names that this repository
   does not use. Every link in this folder now uses the folder keys of the repository, in
   particular [../electronic-invoicing-and-document-exchange/](../electronic-invoicing-and-document-exchange/)
   and [../messaging-and-activities/](../messaging-and-activities/) where messaging is concerned.
4. **Rule identifiers.** One draft numbered its rules `MCUR-RULE-nnn`; the other stated its rules as
   prose inside its entity and calculation sections. A single scheme, `MCUR-nnn`, is now used
   throughout the folder, and [business-rules.md](business-rules.md) section 13 maps every former
   reference onto it.
5. **Where the reference data lives.** Both drafts carried the complete catalogue of the one
   hundred and seventy shipped currencies. It is published once, in
   [configuration.md](configuration.md) section 4.2, together with the complete demonstration rate
   table and the currencies that country packages add or activate.
