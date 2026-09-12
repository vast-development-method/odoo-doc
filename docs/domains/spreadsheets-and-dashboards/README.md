# Spreadsheets and Dashboards

## Scope

This domain specifies everything the system does with **live spreadsheet documents and the dashboards built from them**: the stored workbook that any record can carry, the validation that protects that workbook from referring to data that no longer exists, the dashboards that are shipped with each capability package and the groups those dashboards are filed under, the reading surface that turns a stored workbook into a rendered dashboard for one reader in one company with one language, the data-bound list and pivot insertions that keep a workbook synchronised with records, the global filters that narrow every data-bound element of a workbook at once, the business formulas that read account movements, partner balances, tagged-account balances, residual amounts, fiscal periods, account groups and currency rates directly into cells, the sharing of a frozen copy of a dashboard through a tokenised web address that a reader outside the application can open, the export of that copy as a workbook file, and the personal board on which a user pins ordinary views.

Everything in this folder is derived from the behaviour of sixteen capability packages:

| Capability package (as a role, not a name) | What it contributes |
|---|---|
| **Spreadsheet engine** (`spreadsheet`) | The Spreadsheet Document mixin, the stored workbook and its validation, the empty-workbook default, the data-bound list and pivot elements, the data-bound charts, the global filters, the business formula surface shared by every consumer, the locale and currency services the workbook needs, the freeze algorithm, the export log, and the public read-only page |
| **Dashboards** (`spreadsheet_dashboard`) | The Spreadsheet Dashboard entity, the Dashboard Group entity, the Dashboard Share entity, the dashboard workspace, the reading routes, the sample-dashboard substitution, the favourite mark, the sharing routes, the access groups and record rules, and the menus |
| **Personal boards** (`board`) | The Dashboard Board abstract entity, the personal board form, the operation that pins an action onto that board, and the per-user stored board layout |
| **Accounting formulas** (`spreadsheet_account`) | The account-movement formulas, the partner-balance formula, the tagged-balance formula, the residual formula, the fiscal-year formulas, the account-group formula, the journal-item selection rule they share, and the cell audit action |
| **Shipped accounting dashboard** (`spreadsheet_dashboard_account`) | The `Invoicing` dashboard and its sample |
| **Shipped event dashboard** (`spreadsheet_dashboard_event_sale`) | The `Events` dashboard and its sample |
| **Shipped expense dashboard** (`spreadsheet_dashboard_hr_expense`) | The `Expenses` dashboard and its sample |
| **Shipped project dashboard** (`spreadsheet_dashboard_hr_timesheet`) | The `Project` dashboard |
| **Shipped conversation dashboards** (`spreadsheet_dashboard_im_livechat`) | The `Live Chat` and `Live Chat - Ongoing Sessions` dashboards, their samples, and the five session window actions and menus they link to |
| **Shipped counter-sale dashboard** (`spreadsheet_dashboard_pos_hr`) | The `Point of Sale` dashboard and its sample |
| **Shipped restaurant dashboard** (`spreadsheet_dashboard_pos_restaurant`) | The `POS - Restaurant` dashboard and its sample |
| **Shipped sales dashboards** (`spreadsheet_dashboard_sale`) | The `Sales` and `Product` dashboards and their samples |
| **Shipped billable-time dashboard** (`spreadsheet_dashboard_sale_timesheet`) | The `Timesheets` dashboard and its sample |
| **Shipped warehouse dashboard** (`spreadsheet_dashboard_stock_account`) | The `Warehouse Metrics` dashboard and its sample |
| **Shipped storefront dashboard** (`spreadsheet_dashboard_website_sale`) | The `eCommerce` dashboard and its sample |
| **Shipped course dashboard** (`spreadsheet_dashboard_website_sale_slides`) | The `eLearning` dashboard and its sample |

## Entities this folder owns

| Full name | Transport name | Kind | Reference page |
|---|---|---|---|
| Spreadsheet Document mixin | `spreadsheet.mixin` | abstract, contributes fields and operations to other entities | [`../../references/entities/spreadsheet.mixin.md`](../../references/entities/spreadsheet.mixin.md) |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | persistent, one table | [`../../references/entities/spreadsheet.dashboard.md`](../../references/entities/spreadsheet.dashboard.md) |
| Dashboard Group | `spreadsheet.dashboard.group` | persistent, one table | [`../../references/entities/spreadsheet.dashboard.group.md`](../../references/entities/spreadsheet.dashboard.group.md) |
| Dashboard Share | `spreadsheet.dashboard.share` | persistent, one table | [`../../references/entities/spreadsheet.dashboard.share.md`](../../references/entities/spreadsheet.dashboard.share.md) |
| Dashboard Board | `board.board` | abstract, no table of its own | [`../../references/entities/board.board.md`](../../references/entities/board.board.md) |

Five entities, no more. The candidate list for this domain contained no other entity.

## Entities this folder uses but does not own

| Full name | Transport name | Owned by |
|---|---|---|
| Account | `account.account` | [`../general-ledger/`](../general-ledger/) |
| Journal Item | `account.move.line` | [`../general-ledger/`](../general-ledger/) |
| Account Tag | `account.account.tag` | [`../general-ledger/`](../general-ledger/) |
| Journal Entry | `account.move` | [`../general-ledger/`](../general-ledger/) |
| Company | `res.company` | [`../contacts-and-organizations/`](../contacts-and-organizations/) |
| Currency | `res.currency` | [`../multi-currency/`](../multi-currency/) |
| Currency Rate | `res.currency.rate` | [`../multi-currency/`](../multi-currency/) |
| Language | `res.lang` | [`../contacts-and-organizations/`](../contacts-and-organizations/) |
| User | `res.users` | [`../identity-and-access/`](../identity-and-access/) |
| Access Group | `res.groups` | [`../identity-and-access/`](../identity-and-access/) |
| Model Definition | `ir.model` | [`../platform-foundation/`](../platform-foundation/) |
| Request Routing | `ir.http` | [`../platform-foundation/`](../platform-foundation/) |
| Attachment | `ir.attachment` | [`../platform-foundation/`](../platform-foundation/) |
| Menu | `ir.ui.menu` | [`../platform-foundation/`](../platform-foundation/) |
| Custom View | `ir.ui.view.custom` | [`../platform-foundation/`](../platform-foundation/) |
| Model Data | `ir.model.data` | [`../platform-foundation/`](../platform-foundation/) |

The extensions this domain adds to those entities — six new operations on Account, one on Company, two on Currency Rate, one on Currency, three on Language, one on Model Definition and one on Request Routing — are specified in [`entities.md`](entities.md) §8, because they exist only to serve this domain and no other domain describes them.

## What this domain does **not** re-specify

| Subject | Where it lives |
|---|---|
| The generic field system, computed fields, inverse rules, translation of stored text | [`../../overview/entity-and-field-system.md`](../../overview/entity-and-field-system.md) |
| How one capability package extends the entities of another | [`../../overview/inheritance-and-extension.md`](../../overview/inheritance-and-extension.md) |
| Access groups, access rights, record rules and the evaluation order between them | [`../../overview/security-model.md`](../../overview/security-model.md), [`../identity-and-access/`](../identity-and-access/) |
| Views, window actions, client actions, menus and the way a client resolves them | [`../../overview/views-and-actions.md`](../../overview/views-and-actions.md) |
| Attachments, binary storage and the stream that serves a stored file | [`../platform-foundation/`](../platform-foundation/) |
| Accounts, their types, their initial-balance flag, their tags, their codes | [`../general-ledger/`](../general-ledger/) |
| Journal entries, their posting, their cancellation, their residual computation, their reconciliation | [`../general-ledger/`](../general-ledger/), [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/) |
| The fiscal year end day and end month of a company, and the rule that derives a fiscal year from them | [`../general-ledger/`](../general-ledger/) |
| Currency rates, the rate lookup by date and company, and the conversion algorithm | [`../multi-currency/`](../multi-currency/) |
| Language records, their decimal separator, thousands separator, date format, time format and week start | [`../contacts-and-organizations/`](../contacts-and-organizations/) |
| The financial statements themselves — balance sheet, profit and loss, aged balances, tax reports | [`../financial-reporting/`](../financial-reporting/) |
| The records that the shipped dashboards measure: orders, invoices, expenses, timesheets, quantities on hand, conversations, courses | [`../sales/`](../sales/), [`../accounts-receivable/`](../accounts-receivable/), [`../expenses/`](../expenses/), [`../timesheets/`](../timesheets/), [`../inventory-operations/`](../inventory-operations/), [`../point-of-sale/`](../point-of-sale/), [`../events/`](../events/), [`../website-and-storefront/`](../website-and-storefront/), [`../learning-surveys-and-gamification/`](../learning-surveys-and-gamification/), [`../messaging-and-activities/`](../messaging-and-activities/) |
| The portal identity of a reader who opens a shared dashboard without an internal account | [`../customer-portal/`](../customer-portal/) |

## Capabilities covered

| Capability | Where specified |
|---|---|
| The stored workbook carried by any record, its binary form, its text form, its file name and its thumbnail | [`entities.md`](entities.md) §2 |
| The structure of a stored workbook: sheets, cells, figures, lists, pivots, charts, filters, locale, revision marker | [`document-format.md`](document-format.md) |
| Validation of a stored workbook against the models, fields and menus it names | [`business-rules.md`](business-rules.md) §2, [`document-format.md`](document-format.md) §7 |
| The Spreadsheet Dashboard with all of its fields, its ordering, its duplication rule and its company scope | [`entities.md`](entities.md) §3 |
| Dashboard groups, their ordering, their published sub-list and the deletion guard that protects shipped ones | [`entities.md`](entities.md) §4 |
| The publication mark and the per-user favourite mark | [`state-machines.md`](state-machines.md) §2, §3 |
| The dashboard workspace: group list, favourites section, selection, address state, mobile presentation | [`workflows.md`](workflows.md) §3, [`interfaces.md`](interfaces.md) §2 |
| Reading a dashboard: the reading route, company selection, locale substitution, default currency, translation namespace | [`workflows.md`](workflows.md) §4, [`calculations.md`](calculations.md) §4 |
| Sample dashboards: when a sample replaces the real one, and what the reader sees | [`workflows.md`](workflows.md) §5, [`state-machines.md`](state-machines.md) §5 |
| Data-bound lists: insertion, formulas, ordering, growth of the fetched window, value conversion per field type | [`workflows.md`](workflows.md) §11, [`calculations.md`](calculations.md) §14, §15 |
| Data-bound pivots: insertion, dimensions, granularity, measures, aggregators, positional references | [`workflows.md`](workflows.md) §12, [`calculations.md`](calculations.md) §16 |
| Data-bound charts: the twelve chart kinds, the click-through, the menu link | [`interfaces.md`](interfaces.md) §6, [`workflows.md`](workflows.md) §6.4 |
| Global filters: six kinds, their operators, their values, their defaults, the record selection they produce | [`global-filters.md`](global-filters.md) |
| Account movement formulas, partner balance, tagged balance, residual amount, account group, fiscal year | [`calculations.md`](calculations.md) §5–§12 |
| The journal-item selection rule shared by every accounting formula and by the cell audit action | [`calculations.md`](calculations.md) §6 |
| Currency rate formula and the number format derived from a company currency | [`calculations.md`](calculations.md) §13 |
| Date serial numbers and locale format conversion | [`calculations.md`](calculations.md) §2, §3 |
| Freezing a dashboard and sharing it through a tokenised address | [`workflows.md`](workflows.md) §8, [`calculations.md`](calculations.md) §19 |
| Opening, reading and downloading a shared dashboard from outside the application | [`workflows.md`](workflows.md) §9, §10, [`interfaces.md`](interfaces.md) §5 |
| The personal board: pinning an action, the stored per-user layout, the preprocessing of that layout | [`entities.md`](entities.md) §6, [`workflows.md`](workflows.md) §13 |
| Export logging: what is written, when, and for which operations | [`workflows.md`](workflows.md) §16, [`business-rules.md`](business-rules.md) §11 |
| Every access group, access right, record rule, menu, action, view and shipped record | [`configuration.md`](configuration.md) |
| Every route, named operation, formula, cell action and import or export path | [`interfaces.md`](interfaces.md) |
| Why the domain writes nothing to the ledger, and which ledger data it reads | [`accounting-effects.md`](accounting-effects.md) |

## Reading order

1. [`README.md`](README.md) — this file: scope, entities, dependencies, file list.
2. [`entities.md`](entities.md) — the five entities, field by field, and the operations this domain adds to entities owned elsewhere.
3. [`document-format.md`](document-format.md) — the structure of the stored workbook that those entities carry.
4. [`state-machines.md`](state-machines.md) — the seven state machines of the domain.
5. [`global-filters.md`](global-filters.md) — the filter system, because workflows and calculations both depend on it.
6. [`workflows.md`](workflows.md) — the sixteen end-to-end procedures.
7. [`business-rules.md`](business-rules.md) — every validation, permission check and invariant, numbered.
8. [`calculations.md`](calculations.md) — every formula with its rounding and a worked example.
9. [`accounting-effects.md`](accounting-effects.md) — the ledger position of the domain.
10. [`configuration.md`](configuration.md) — everything an installation ships or an administrator sets.
11. [`interfaces.md`](interfaces.md) — menus, views, routes, operations, formulas, import and export.
12. [`acceptance-criteria.md`](acceptance-criteria.md) — numbered scenarios with concrete numbers.
13. [`glossary.md`](glossary.md) — every term used above.

## Files in this folder

| File | Content |
|---|---|
| [`README.md`](README.md) | Scope, capability packages, entities owned and borrowed, dependencies, reading order, file list |
| [`entities.md`](entities.md) | The five entities in full: purpose, lifecycle, complete field tables, relations, constraints, ordering, display name, company behaviour, extension points, and the operations this domain adds to seven entities owned by other domains |
| [`document-format.md`](document-format.md) | The structure of a stored workbook: top-level sections, sheets, cells, figures, lists, pivots, charts, filters, links, the empty workbook, the frozen workbook, and the validation walk |
| [`state-machines.md`](state-machines.md) | Publication, favourite mark, dashboard rendering status, data-source status, sample-versus-live presentation, share-link reachability, and global-filter value state, each with states, transitions, guards and a diagram |
| [`global-filters.md`](global-filters.md) | The six filter kinds, every operator, every value shape, default values, the record-selection rule each produces, the field matching, the period offset, and the filter sheet written on export |
| [`workflows.md`](workflows.md) | Sixteen numbered end-to-end procedures with the records each step creates or changes and the failures each step can raise |
| [`business-rules.md`](business-rules.md) | Sixty-six numbered rules with exact messages, plus the permission matrix, the locking position and the rule index |
| [`calculations.md`](calculations.md) | Twenty calculation families with quantities named in words, evaluation order, rounding and worked numeric examples |
| [`accounting-effects.md`](accounting-effects.md) | The reasoned statement that the domain posts nothing, the ledger data it reads, the correctness obligations that follow, and links to the domains that do post |
| [`configuration.md`](configuration.md) | Privilege, group, access rights, record rules, menus, actions, views, shipped dashboard groups, shipped dashboards, the session capability flag, and the explicit absence of sequences, scheduled jobs, message templates and activity types |
| [`interfaces.md`](interfaces.md) | Menus, views, client action, six routes, named operations, the complete formula surface, cell actions, printing, import and export |
| [`acceptance-criteria.md`](acceptance-criteria.md) | One hundred and twenty-five numbered Given-When-Then scenarios with concrete records, inputs and results |
| [`glossary.md`](glossary.md) | Every term of the domain, defined |
