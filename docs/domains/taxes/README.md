# Taxes

## Scope

This domain specifies the tax engine of the accounting system: how taxes are defined, how tax amounts are computed from document lines, how the computed amounts become journal items, how those journal items are tagged for tax reporting, how taxes whose exigibility is deferred to the payment produce their own journal entries at reconciliation time, how fiscal positions substitute taxes and accounts depending on the counterpart, how withholding taxes are retained at payment time, how formula-based taxes are evaluated, and how value-added tax numbers of partners are validated and formatted.

Everything in this folder is derived from the behavior of the following capability packages: the core Accounting package (tax definitions, tax groups, distribution lines, fiscal positions, tags, the computation engine, cash basis, tax reports skeleton), the Custom Formula Taxes package (formula-based tax computation), the Withholding Tax on Payment package and its Point of Sale companion (withholding at payment), and the Value-Added Tax Number Validation package (per-country number checks and the online European verification switch).

The domain does **not** cover: the general ledger posting mechanics (see `../general-ledger/`), invoice lifecycle (see `../accounts-receivable/` and `../accounts-payable/`), payments and reconciliation core (see `../payments-and-bank-reconciliation/`), currency rates (see `../multi-currency/`), analytic distribution (see `../analytic-accounting/`), the sales and purchase documents that call the engine (see `../sales/` and `../purchasing/`), and point of sale orders (see `../point-of-sale/`; this domain only states which tax fields are loaded there).

## Capabilities covered

| Capability | Where specified |
|---|---|
| Tax definitions: amount types (percentage, fixed, percentage of tax-included price, group, custom formula), price inclusion, sequence, scope, exigibility, base chaining flags, analytic flag, tax group, country, legal notes | `entities.md`, `business-rules.md` |
| Distribution lines for invoices and refunds: factor, base or tax type, account, tags, use in tax closing | `entities.md`, `business-rules.md`, `accounting-effects.md` |
| Tax computation engine: base line and tax line dictionaries, evaluation order, batching, price-included extraction, fixed taxes with quantity, division taxes, reverse charge distributions, rounding per line versus per tax globally, currency rounding, dual-currency amounts, refund sign, manual amounts, discount and down payment reductions, tax totals summary, cash rounding interaction | `calculations.md` |
| Tax groups: payable, receivable and advance accounts, preceding subtotal, receipt label | `entities.md`, `configuration.md` |
| Fiscal positions: tax mapping through replacement taxes, account mapping, automatic detection by country, country group, state, postal code range, value-added tax requirement, foreign value-added tax registration, application on documents and journal items | `entities.md`, `calculations.md`, `workflows.md` |
| Tax tags (grids): creation from tax report expressions, landing on journal items, sign semantics on invoices and refunds, archival on report line deletion | `entities.md`, `accounting-effects.md`, `workflows.md` |
| Tax report skeleton: reports, lines, expressions of the tax tags engine, columns, external values, return-period date filters, tax exigibility filter | `entities.md`, `configuration.md` |
| Cash basis taxes: transition account, cash basis journal, base account, entries created at partial reconciliation, percentage paid, last-partial rounding, reversal at unreconciliation, dates and lock dates | `calculations.md`, `accounting-effects.md`, `workflows.md` |
| Withholding taxes on payment: negative taxes retained at payment time, withholding lines on the payment and on the payment register, sequence numbers, net amount, payment entry composition | `entities.md`, `calculations.md`, `accounting-effects.md`, `workflows.md` |
| Custom formula taxes: formula field, variables available, allowed grammar, evaluation as a fixed-amount tax | `entities.md`, `calculations.md`, `business-rules.md` |
| Company tax settings: rounding method, default sale and purchase taxes, price inclusion default, fiscal country, cash basis switch, cash basis journal and base account, withholding base account, online verification switch | `configuration.md` |
| Tax-related fields on products, accounts and partners and their defaulting rules | `entities.md`, `workflows.md` |
| Display rules of tax-included or tax-excluded prices on printed documents and of tax totals | `interfaces.md`, `calculations.md` |
| Value-added tax number validation and formatting: prefix handling, per-country check routines, check digit arithmetic, online European verification, webhook and scheduled synchronization | `calculations.md`, `business-rules.md`, `interfaces.md` |

## Entities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Tax | `account.tax` | `account_tax` | Definition of one tax and its computation parameters |
| Tax Distribution Line | `account.tax.repartition.line` | `account_tax_repartition_line` | One base or tax line of the invoice or refund distribution of a tax: factor, account, tags |
| Tax Group | `account.tax.group` | `account_tax_group` | Presentation and closing grouping of taxes, with closing accounts |
| Fiscal Position | `account.fiscal.position` | `account_fiscal_position` | Rule set substituting taxes and accounts for a counterpart or a territory |
| Fiscal Position Account Mapping | `account.fiscal.position.account` | `account_fiscal_position_account` | One account substitution of a fiscal position |
| Account Tag | `account.account.tag` | `account_account_tag` | Tag applicable to accounts, taxes (tax grids) or products |
| Accounting Report | `account.report` | `account_report` | Report definition; tax reports are reports whose expressions use the tax tags engine |
| Accounting Report Line | `account.report.line` | `account_report_line` | One line of a report |
| Accounting Report Expression | `account.report.expression` | `account_report_expression` | One computed value of a report line; the `tax_tags` engine creates tax tags |
| Accounting Report Column | `account.report.column` | `account_report_column` | One column of a report |
| Accounting Report External Value | `account.report.external.value` | `account_report_external_value` | Manually entered or carried-over value for an expression |
| Withholding Line (abstract) | `account.withholding.line` | none | Shared definition of a withholding line |
| Payment Withholding Line | `account.payment.withholding.line` | `account_payment_withholding_line` | Withholding line stored on a payment |
| Payment Register Withholding Line | `account.payment.register.withholding.line` | `account_payment_register_withholding_line` | Transient withholding line of the payment register wizard |

Entities of other domains that carry tax-related fields specified here: Journal Entry (`account.move`), Journal Item (`account.move.line`), Partial Reconciliation (`account.partial.reconcile`), Company (`res.company`), Product Template (`product.template`), Product (`product.product`), Account (`account.account`), Partner (`res.partner`), Country (`res.country`), Country Group (`res.country.group`), Payment (`account.payment`), Payment Register (`account.payment.register`), Cash Rounding (`account.cash.rounding`), Settings (`res.config.settings`).

## Reading order

1. `glossary.md` for the vocabulary (base line, tax line, batch, extra base, distribution line, tag, transition account, exigibility, special mode, computation key).
2. `entities.md` for the data model.
3. `calculations.md` for the engine; read it in order, the later sections depend on the earlier ones.
4. `accounting-effects.md` for every journal entry produced.
5. `workflows.md` for the operational sequences.
6. `business-rules.md` for validations and messages.
7. `state-machines.md`, `configuration.md`, `interfaces.md`.
8. `acceptance-criteria.md` to validate an implementation.

## Dependencies on other domains

| Domain | Dependency |
|---|---|
| `../general-ledger/` | Accounts, journals, journal entries and items, posting, lock dates, reconciliation core, chart templates |
| `../accounts-receivable/`, `../accounts-payable/` | Invoices and refunds whose lines feed the engine; payment terms; early payment discount lines; cash rounding lines; non-deductible lines |
| `../payments-and-bank-reconciliation/` | Payments and the payment register wizard (withholding lines), partial reconciliation (cash basis entries) |
| `../multi-currency/` | Currency rounding precision and decimal places, conversion rates used for company-currency amounts, exchange difference entries |
| `../analytic-accounting/` | Analytic distribution carried by tax lines when the tax is analytic |
| `../contacts-and-organizations/` | Partner country, state, postal code, value-added tax number, country groups |
| `../products-and-catalog/` | Product sale and purchase taxes, product account tags, list price handling |
| `../point-of-sale/` | Consumer of the same engine on the client side; tax fields loaded |
| `../sales/`, `../purchasing/` | Order lines call the engine and fiscal positions identically |

## Behavior not present in the source set

The following capabilities that are commonly associated with tax management are **not** implemented by the packages covered here and are therefore only described as industry-standard defaults where a companion package is expected to add them: the periodic tax return (periodicity, return dates, closing journal and closing entry generation), the tax unit grouping of several companies into one return (only a report filter selection value `tax_units` exists), the evaluation of tax report expressions at report rendering time, and the actual online query of the European verification service (only the relay call and its webhook are present). These points are called out where relevant in `configuration.md`, `calculations.md` and `accounting-effects.md`.

## Files in this folder

| File | Content |
|---|---|
| `README.md` | This file: scope, capabilities, entity list, reading order, dependencies, index of the worked examples |
| `entities.md` | Every entity in full, plus the tax-related fields added to entities of other domains, the storage-name index and a field-level cross-reference |
| `state-machines.md` | Tax availability, fiscal position availability, the account tag lifecycle, the exigibility lifecycle of a tax amount, the cross-border verification, the withholding numbering hint, the recomputation decision of a draft entry, the lifecycle of a tag on a journal item |
| `workflows.md` | Creating a tax, creating the report lines that create the grids, putting taxes on a line, recomputing a document, posting, reversing, cash basis at reconciliation, withholding at payment, re-deriving tags, fiscal positions, tax numbers, archiving, the chart template, every consumer of the engine, and a diagnosis checklist |
| `business-rules.md` | Every validation with its exact message, the locking rules, the permission matrix summary, the invariants, the edge cases, the full message index, the validation order and the concurrency notes |
| `calculations.md` | The engine in full: data shapes, rounding primitives, ordering and batching, every amount formula, the extra-base propagation table, the single-line algorithm, the document-wide rounding, the accounting derivation, manual amounts, the totals block, fiscal positions, cash basis, withholding, the document-level helpers, every country's tax-number arithmetic, and the reconstruction of the base-to-tax mapping |
| `accounting-effects.md` | Every journal item the domain causes, the cash basis entries, the withholding entries, a catalogue by document kind, a complete worked entry and a list of the mistakes that produce wrong numbers |
| `configuration.md` | Company settings, system parameters, sequences, shipped records, groups, the access matrix, record rules, scheduled jobs, the rounding-method and price-inclusion decision guides, three archetypal configurations and the interaction with localization packages |
| `interfaces.md` | Navigation, every screen, the operations other components invoke, the route, the printable output, the message history, the external service, import and export, the client-side mirror contract and the ordering constraints on the engine's operations |
| `acceptance-criteria.md` | Numbered Given / When / Then scenarios with concrete numbers, grouped from A to V |
| `glossary.md` | Every term defined in full, plus the words this folder deliberately avoids |

## Index of the worked examples

Every number below is derived from the specified algorithms and can be used directly as a test
fixture.

| Example | Where |
|---|---|
| A percentage tax, price-excluded | `calculations.md` section 4.2 |
| A percentage tax, price-included, exact and with a residue | `calculations.md` section 4.3 |
| Two price-included percentage taxes in one batch | `calculations.md` section 4.3 |
| A fixed tax with a quantity, with and without base inclusion | `calculations.md` section 4.1 |
| A division tax, price-excluded and price-included | `calculations.md` sections 4.4 and 4.5 |
| A group of taxes | `calculations.md` section 4.7 |
| A chain using base-amount inclusion, price-excluded and price-included | `calculations.md` section 5.3 |
| Round per line against round per tax on a three-line document, to the cent | `calculations.md` section 7.8 |
| The same in the price-included variant | `calculations.md` section 7.8 |
| A multi-currency three-line document | `calculations.md` section 7.9 |
| The smooth distribution of a delta, with and without a leftover | `calculations.md` section 7.1 |
| A refund and its tag signs | `calculations.md` section 8.7 |
| A fiscal position substitution, end to end | `calculations.md` section 11.5 |
| A payment of forty percent under deferred exigibility, and the settling payment | `calculations.md` section 12.5 |
| A three-way instalment needing the last-partial correction | `calculations.md` section 12.5 |
| A withholding on a customer payment, and on an instalment | `calculations.md` section 13.4 |
| Two subtotals in the totals block | `calculations.md` section 10.6 |
| Cash rounding to the nearest five cents, both strategies | `calculations.md` section 10.7 |
| The weighted-average analytic distribution of an aggregation | `calculations.md` section 14.4 |
| Thirty-plus tax identification number check digits, computed step by step | `calculations.md` section 15.6 |
| The reconstruction of the base-to-tax mapping on a seven-item entry | `calculations.md` section 16 |
| A complete posted entry with a fixed levy affecting the base of a percentage tax | `accounting-effects.md` section 11 |
| A reverse charge on a vendor bill | `accounting-effects.md` section 4 |
| A tax split over two accounts | `accounting-effects.md` section 3 |
| A withholding on a vendor payment | `accounting-effects.md` section 7.2 |

## How to verify an implementation

1. Implement the rounding primitive of `calculations.md` section 2 first and check it against the
   tie examples; every later number depends on it.
2. Implement the single-line computation of section 6 and check it against scenarios A1 to A21 of
   `acceptance-criteria.md`.
3. Implement the document-wide rounding of section 7 and check it against scenarios C1 to C12; this
   is where most implementations diverge.
4. Implement the accounting derivation of section 8 and check it against scenarios E1 to E14 and
   F1 to F10.
5. Implement the fiscal positions of section 11 and check them against scenarios G1 to G18.
6. Implement cash basis and withholding and check them against H1 to H13 and I1 to I16.
7. Implement the tax number checks of section 15 and check them against the worked verifications
   of section 15.6 and scenarios J1 to J23.
8. Finally, check the whole thing against the end-to-end entry of `accounting-effects.md`
   section 11.
