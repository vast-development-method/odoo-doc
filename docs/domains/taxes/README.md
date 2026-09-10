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
| `../products/` | Product sale and purchase taxes, product account tags, list price handling |
| `../point-of-sale/` | Consumer of the same engine on the client side; tax fields loaded |
| `../sales/`, `../purchasing/` | Order lines call the engine and fiscal positions identically |

## Behavior not present in the source set

The following capabilities that are commonly associated with tax management are **not** implemented by the packages covered here and are therefore only described as industry-standard defaults where a companion package is expected to add them: the periodic tax return (periodicity, return dates, closing journal and closing entry generation), the tax unit grouping of several companies into one return (only a report filter selection value `tax_units` exists), the evaluation of tax report expressions at report rendering time, and the actual online query of the European verification service (only the relay call and its webhook are present). These points are called out where relevant in `configuration.md`, `calculations.md` and `accounting-effects.md`.
