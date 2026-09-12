# Fiscal Localizations: Configuration

Every setting, parameter, default and master-data prerequisite this domain reads or writes, with its data type, its default value and its effect; the access groups of the domain and what each may do; the scheduled jobs; and the master data that must exist before a country package can be used.

---

## 1. Company-level settings written by a chart of accounts template

These settings live on the Company record. A template load writes them from its company-level bundle and from its `res.company` bundle. The table gives the canonical name, the data type, the platform default before any template is loaded, and the effect.

| Setting | Type | Platform default | Effect |
|---|---|---|---|
| `chart_of_accounts_template_code` | selection (any registered template code) | empty | Names the template the company uses. Selecting a different value triggers a load; selecting the same value triggers a reload. |
| `expects_a_chart_of_accounts` | boolean | true | When false, the company is not offered a chart of accounts and no template is auto-loaded. |
| `fiscal_country` | many_to_one to Country | the company country | Decides which taxes, tax groups, report tags and country reports apply, and which country-specific fields are visible. |
| `currency` | many_to_one to Currency | the platform default currency | The company currency of every journal item. Written from the fiscal country's currency, or from the parent company's currency for a subsidiary, and only while the company's root has no journal item. |
| `bank_account_code_prefix` | text | empty | Where bank journal accounts, the bank suspense account and the two outstanding accounts are numbered. Changing it renumbers the matching existing accounts. |
| `cash_account_code_prefix` | text | empty | Where cash journal accounts are numbered. Changing it renumbers the matching existing accounts. |
| `transfer_account_code_prefix` | text | empty | Where the inter-banks transfer account is numbered. |
| `income_account` | many_to_one to Account | empty | Default credit account of a customer invoice line that resolves to nothing more specific; also written onto the sales journal's default account after a load. |
| `expense_account` | many_to_one to Account | empty | Default debit account of a vendor bill line; also written onto the purchase journal's default account after a load. |
| `default_sale_tax` | many_to_one to Tax | empty | Proposed on a new sales line. Set after a load to the first company tax whose scope is sale or all, only when previously empty. |
| `default_purchase_tax` | many_to_one to Tax | empty | Proposed on a new purchase line. Set after a load to the first company tax whose scope is purchase or all, only when previously empty. |
| `default_purchase_receipt_fiscal_position` | many_to_one to Fiscal Position | empty | Fiscal position proposed on a purchase receipt. |
| `tax_calculation_rounding_method` | selection: Round per Tax, Round per Line | Round per Tax | Whether each tax is rounded once for the document or once per line. See [calculations.md](calculations.md) section 14. |
| `inter_banks_transfer_account` | many_to_one to Account (must be reconcilable and of type Current Asset) | empty | Intermediate account of an internal transfer between two liquidity journals. Created as "Liquidity Transfer" during post-processing. |
| `bank_suspense_account` | many_to_one to Account | empty | Default suspense account of every Bank, Cash and Credit journal. Created as "Bank Suspense Account" during post-processing. |
| `cash_difference_income_account` | many_to_one to Account | empty | Profit account of a liquidity journal. Created as "Cash Difference Gain" under prefix `999`, tagged Investing activity. |
| `cash_difference_expense_account` | many_to_one to Account | empty | Loss account of a liquidity journal. Created as "Cash Difference Loss" under prefix `999`, tagged Investing activity. |
| `cash_discount_write_off_gain_account` | many_to_one to Account | empty | Created as "Cash Discount Gain" with the fixed code `999997`, type Other Income. |
| `cash_discount_write_off_loss_account` | many_to_one to Account | empty | Created as "Cash Discount Loss" with the fixed code `999998`, type Expense. |
| `exchange_gain_or_loss_journal` | many_to_one to Journal of type Miscellaneous | empty | Journal of the realised exchange difference entry. Pointed at the journal created under the symbolic identifier `exch`. |
| `gain_exchange_rate_account` | many_to_one to Account of an income group | empty | Credit side of a realised exchange gain. |
| `loss_exchange_rate_account` | many_to_one to Account of type Expense or Other Expense | empty | Debit side of a realised exchange loss. |
| `use_cash_basis` | boolean | false | Reveals the cash basis journal and the cash basis base account. Switched on automatically after a load that produced at least one tax with exigibility "on payment", for a top-level company. |
| `cash_basis_journal` | many_to_one to Journal | empty | Journal of the cash basis transfer entry. Pointed at the journal created under the symbolic identifier `caba`. |
| `cash_basis_base_account` | many_to_one to Account | empty | Account of the informational base pair in a cash basis entry. |
| `cost_accounting_flag` (perpetual expense recognition at the customer invoice) | boolean | false | When true, the cost of goods sold is recognised at the customer invoice; when false, at the delivery. Defaulted to false when the template does not set it. |
| `default_point_of_sale_receivable_account` | many_to_one to Account | empty | Receivable account of a point of sale session entry. |
| `deferred_expense_account` and `deferred_revenue_account` | many_to_one to Account | empty | Holding accounts of a spread expense or revenue. |
| `expense_accrual_account` and `revenue_accrual_account` | many_to_one to Account | empty | Holding accounts when an entry's period is moved. |
| `price_difference_account` | many_to_one to Account | empty | Holds the difference between the standard cost and the vendor price during perpetual valuation. |
| `separate_account_for_income_discount` and `separate_account_for_expense_discount` | many_to_one to Account | empty | Where a granted or received discount is booked separately. |
| `display_invoice_amount_total_in_words` | boolean | false | Prints the invoice total in words. Several countries require it. |
| `display_taxes_in_company_currency` | boolean | true | Prints a second tax total converted into the company currency. |
| `display_visual_code_on_invoices` | boolean | false | Prints a payment visual code on the invoice. |
| `storno_accounting` | boolean | computed | True when the fiscal country requires negative-amount reversals. |
| `restrictive_audit_trail` | boolean | false | Prevents deletion of journal-item logs. A localization may force it on. |
| `default_sales_price_includes_tax` | selection: Tax Included, Tax Excluded | Tax Excluded | Whether a typed sales price is understood as including tax. |
| `additional_properties` | mapping key to entity name | empty | Lets a template declare extra company-scoped default values to write after a load. |
| `code_digits` | integer | 6 | Total character length to which every account code of the template is right-padded with zeros. Not stored on the company; it is a template-level value consumed during the load. |

### 1.1 Storno accounting country sets

| Set | Countries (by name) | Behavior |
|---|---|---|
| Mandatory | Bosnia and Herzegovina, China, Czech Republic, Croatia, Poland, Romania, Serbia, Russia, Slovenia, Slovakia, Ukraine | The storno flag is computed true and the control is shown. |
| Optional | Austria, Switzerland, Germany, Italy | The flag defaults to false and the control is shown so the user may switch it on. |
| Everywhere else | all other countries | The flag is false and the control is hidden. |

The country codes above are data values of the country records: `BA`, `CN`, `CZ`, `HR`, `PL`, `RO`, `RS`, `RU`, `SI`, `SK`, `UA` for the mandatory set and `AT`, `CH`, `DE`, `IT` for the optional set.

### 1.2 Forced restrictive audit trail

Two country packages force the restrictive audit trail on and hide the control: the Germany package and the India package. Switching it off is refused:

```
Can't disable restricted audit trail: forced by localization.
```

---

## 2. Company-level values recorded as scoped defaults, not fields

Three values a template declares are **not** fields of the company. They are recorded as company-scoped default values on another entity, so that every new record of that entity starts with them.

| Template key | Target entity | Target field | Effect |
|---|---|---|---|
| `property_account_receivable_id` | Contact | receivable account | Debit account of a customer invoice for a contact that does not override it. |
| `property_account_payable_id` | Contact | payable account | Credit account of a vendor bill for a contact that does not override it. |
| `property_stock_journal` | Product Category | stock journal | Journal of a perpetual inventory entry for products in that category. |

Two further defaults are written unconditionally from the company after a load:

| Target entity | Target field | Value written |
|---|---|---|
| Product Category | income account | the company's income account |
| Product Category | expense account | the company's expense account |

A template may extend the first table through the `additional_properties` key, whose value is a mapping from template key to target entity name. Keys beginning with `property_stock_` are written as ordinary company fields instead, because they are fields of the company.

On a **reload**, every key beginning with `property_` is dropped: the defaults are never re-imposed.

---

## 3. Journals every template creates

| Symbolic identifier | Name | Type | Code | Shown on dashboard | Sequence |
|---|---|---|---|---|---|
| `sale` | Sales | Sale | `INV` | yes | 5 |
| `purchase` | Purchases | Purchase | `BILL` | yes | 6 |
| `general` | Miscellaneous Operations | Miscellaneous | `MISC` | no | 9 |
| `exch` | Exchange Difference | Miscellaneous | `EXCH` | no | (unset) |
| `caba` | Cash Basis Taxes | Miscellaneous | `CABA` | no | (unset) |
| `bank` | Bank | Bank | (assigned from the bank prefix) | yes | 7 |

The Sales and Purchases journals also take colour 11. The journal **code** is translated into the company contact's language even though the field is not translatable; see [business-rules.md](business-rules.md) rule `FLOC-RULE-052`.

A country package may add journals to this set (for example a dedicated cash journal, a cash basis journal with a country name, or one journal per point of sale) and may override the values above.

---

## 4. Reconciliation models every template creates

| Symbolic identifier | Name | Matching | Line |
|---|---|---|---|
| `internal_transfer_reco` | Internal Transfers | none | one line, 100 percent, labelled "Internal Transfers", account set after the load to the company's inter-banks transfer account |
| `bank_fees_reco` | Bank Fees | statement label contains the text `Bank Fees` | one line, 100 percent, labelled "Bank Fees", account set after the load to the first company account whose name contains "Bank Fees", or failing that the first account of type Expense |

On a **reload**, reconciliation models are dropped entirely and are never re-created or updated.

---

## 5. Utility accounts created after every load

Created only for a company that does not already carry the corresponding value. A subsidiary creates none and copies every value from the first company in its parent chain.

| Company field filled | Account name | Code | Account type | Reconcilable | Tags |
|---|---|---|---|---|---|
| Bank suspense account | Bank Suspense Account | first free code from the bank account code prefix | Current Asset | no | none |
| Cash discount write-off loss account | Cash Discount Loss | `999998` | Expense | no | none |
| Cash discount write-off gain account | Cash Discount Gain | `999997` | Other Income | no | none |
| Cash difference income account | Cash Difference Gain | first free code from prefix `999` | Other Income | no | Investing activity |
| Cash difference expense account | Cash Difference Loss | first free code from prefix `999` | Expense | no | Investing activity |
| Inter-banks transfer account | Liquidity Transfer | first free code from the transfer account code prefix | Current Asset | yes | none |
| (no company field) | Outstanding Receipts | first free code from the bank account code prefix | Current Asset | yes | none |
| (no company field) | Outstanding Payments | first free code from the bank account code prefix | Current Asset | yes | none |

The two outstanding accounts are created only for a company without a parent and are registered under the external identifiers `account_journal_payment_debit_account_id` and `account_journal_payment_credit_account_id` for that company.

A country package may override any row. For example the Mexico package replaces the two cash difference accounts by fixed codes `403.01.01` ("Other Income") and `601.84.02` ("Cash Difference Loss").

---

## 6. Withholding tax settings

| Setting | Location | Type | Default | Effect |
|---|---|---|---|---|
| `withhold_on_payment` | Tax | boolean | false | Excludes the tax from every ordinary computation and makes it available on payment withholding lines. Hidden when the tax amount is not negative. |
| `withholding_numbering_series` | Tax | many_to_one to Sequence, not copied on duplication, company-checked | empty | Produces the number written on a withholding line that the user leaves empty. |
| `withholding_tax_base_account` | Company | many_to_one to Account | empty | When set, every withholding base line uses this account and the account column disappears from the withholding line table. |

Switching `withhold_on_payment` on forces the tax's exigibility to "on invoice" and its price inclusion to "tax excluded". Setting the tax amount to zero or above clears the flag.

---

## 7. Financial report settings

### 7.1 Report-level

| Setting | Type | Default | Effect |
|---|---|---|---|
| `name` | text, translatable, required | none | Report title. The display name appends the country code in parentheses when a country is set. |
| `sequence` | integer | 0 | Ordering among reports. |
| `active` | boolean | true | Archived reports are hidden. |
| `root_report` | many_to_one to Financial Report | empty | Marks the report as a country variant of a generic report; the variant inherits the root's filter defaults. A root report may not itself have a root. |
| `sections` | many_to_many to Financial Report | empty | Makes the report composite. Sections may not have sections. |
| `composite_report` | boolean | computed from sections | True when sections exist. |
| `chart_of_accounts_template_code` | selection | empty | Used by the "Chart of Accounts Matches" availability. |
| `country` | many_to_one to Country | empty | Used by the "Country Matches" availability and owns the report's tags. |
| `availability` | selection: Country Matches, Chart of Accounts Matches, Always | Country Matches when a root report and a country are set, otherwise Always | Decides whether the report is offered to a company. |
| `only_tax_exigible_lines` | boolean | inherited, else false | Restricts the journal items considered to those whose tax is currently exigible. |
| `allow_foreign_registration` | boolean | inherited, else false | Permits running the report under a foreign registration fiscal position. |
| `load_more_limit` | integer | 0 | Number of child rows fetched per expansion; 0 means no limit. |
| `search_bar` | boolean | false | Offers a text filter over line names. |
| `prefix_groups_threshold` | integer | 4000 | Above this number of rows, results are grouped by account code prefix. |
| `integer_rounding` | selection: Nearest, Up, Down | empty | Rounding applied when figures are rendered as whole units. |
| `default_opening_period` | selection: This Year, This Quarter, This Month, Today, Last Month, Last Quarter, Last Year, This Return Period, Last Return Period | Last Month | Period pre-selected when the report opens. |
| `currency_translation` | selection: most recent rate at the report date, cumulative translation adjustment | cumulative translation adjustment | How balances in other currencies are converted. |

### 7.2 Report filters

Each filter shows or hides one control. Each defaults from the root report; failing that, from the single composite parent when the report is a section of exactly one composite report and is not itself reachable as a standalone report; failing that, from the value in the table.

| Filter | Values | Default |
|---|---|---|
| Multi-company | Use Company Selector, Use Tax Units | Use Company Selector |
| Date range | boolean | true |
| Draft entries | boolean | true |
| Unreconciled entries | boolean | false |
| Unfold all | boolean | false |
| Hide lines at zero | Enabled by Default, Optional, Never | Optional |
| Period comparison | boolean | true |
| Growth comparison | boolean | true |
| Journals | boolean | false |
| Analytic filter | boolean | false |
| Account groups | Enabled by Default, Optional, Never | Optional |
| Account types | Payable and receivable, Payable, Receivable, Disabled | Disabled |
| Partners | boolean | false |
| Favorite filters | boolean | false |
| Budgets | boolean | false |

### 7.3 Report line settings

| Setting | Type | Default | Effect |
|---|---|---|---|
| `name` | text, translatable, required | none | Displayed label. |
| `code` | text | empty | Unique within the report; referenced by aggregation formulas. |
| `sequence` | integer | 0 | Ordering; a parent must precede its children. |
| `parent_line` | many_to_one to Financial Report Line, deletion sets the link to empty | empty | Builds the tree. |
| `level` | integer, computed, editable | 1 for a root line | Indentation. See [calculations.md](calculations.md) section 9. |
| `group_by` | text, comma-separated journal item field names | empty | Expands the line into one sub-line per distinct value. |
| `user_group_by` | text | defaults to `group_by` | User-chosen grouping; reverts to `group_by` when unsupported. |
| `foldable` | boolean | false | Collapses the line by default. |
| `print_on_new_page` | boolean | false | This line and everything after it start a new printed page. |
| `action` | many_to_one to Action | empty | Turns the line into a link. |
| `hide_if_zero` | boolean | false | Hides the line and its children when every column is zero. |
| `horizontal_split_side` | selection: Left, Right | inherited from the parent | Used by two-column statements. |

### 7.4 Report expression settings

| Setting | Type | Default | Effect |
|---|---|---|---|
| `label` | text, required, copied on duplication | none | Unique within the line; named by a column. |
| `engine` | selection: Domain, Tax Tags, Aggregation, Prefix of Account Codes, External Value, Custom Function | none, required | How the figure is computed. |
| `formula` | text, required | none | Engine-specific. Whitespace-normalised on write. |
| `subformula` | text | empty | Engine-specific. Required for the Domain engine. |
| `date_scope` | selection: from the very start, from the start of the fiscal year, at the beginning of the fiscal year, at the beginning of the period, strictly on the given dates, from the previous return period | strictly on the given dates | Which journal items the engine sees. |
| `figure_type` | selection: Monetary, Percentage, Integer, Float, Date, Datetime, Boolean, String | empty | Rendering. |
| `growth_is_good_when_positive` | boolean | true | Colouring of the growth comparison. |
| `blank_if_zero` | boolean | false | Hides a zero figure. |
| `auditable` | boolean, computed, editable | true for every engine except Custom Function | Whether the figure can be expanded into journal items. |
| `carryover_target` | text `line_code.expression_label` | empty | Explicit carryover target. |

### 7.5 Report column settings

| Setting | Type | Default | Effect |
|---|---|---|---|
| `name` | text, translatable, required | none | Column header. |
| `expression_label` | text, required | none | Which expression of each line feeds the column. |
| `sequence` | integer | 0 | Column order. |
| `sortable` | boolean | false | Whether the report may be sorted on this column. |
| `figure_type` | selection as above | Monetary, required | Rendering. |
| `blank_if_zero` | boolean | false | Hides zero figures in this column. |
| `custom_audit_action` | many_to_one to Window Action | empty | Replaces the default drill-down. |

### 7.6 Report external value settings

| Setting | Type | Required | Effect |
|---|---|---|---|
| `name` | text | yes | Label of the manual figure. |
| `numeric_value` | decimal | no | The figure. |
| `text_value` | text | no | A textual figure. |
| `date` | date | yes | Places the value in a period. Ordering is by date then identifier. |
| `target_expression` | many_to_one to Financial Report Expression, cascade on deletion | yes | Which expression reads it. |
| `company` | many_to_one to Company | yes, defaults to the active company | Scope. |
| `carryover_origin_expression_label` | text | no | Set when the value was written by a carryover. |
| `carryover_origin_line` | many_to_one to Financial Report Line | no | Set when the value was written by a carryover. |

---

## 8. Fiscal position settings used by this domain

| Setting | Type | Default | Effect |
|---|---|---|---|
| `foreign_tax_identification_number` | text | empty | Declares a registration in another country. Normalised and format-checked against the position's country. |
| `country` | many_to_one to Country | empty | The country of the registration and of the automatic detection predicate. |
| `country_group` | many_to_one to Country Group | empty | Alternative to a country for detection. |
| `country_subdivisions` | many_to_many to Country Subdivision | empty | Detection predicate on the contact's subdivision. |
| `postal_code_from` and `postal_code_to` | text | empty | Detection predicate on the contact's postal code; both required together; numeric bounds are zero-padded to a common length. |
| `detect_automatically` | boolean | false | Includes the position in automatic detection. |
| `requires_tax_identification_number` | boolean | false | Detection predicate requiring a valid number on the contact. |
| `sequence` | integer | 0 | Detection order within a company specificity level. |
| `foreign_registration_header_mode` | selection: Templates Found, No Template | computed | Reveals the "Create foreign taxes" button. |

---

## 9. Country reference tables shipped by packages

Every country package ships reference records that are **not** company scoped and are shared by every company in the database. A package that needs per-company variation adds a company field explicitly. The recurring kinds are:

| Kind | Typical fields | Example countries |
|---|---|---|
| Document type | code, name, internal type, country, document sorting, report name, active | Argentina, Chile, Colombia, Ecuador, Peru, Uruguay, Italy, Bulgaria |
| Identification type | name, country, sequence, whether it is the tax identification number, active | every Latin American country |
| Tax office | code, name, region, workplace code | Czech Republic, Poland, Turkey |
| Classification code | code, name, parent | Croatia (product classification), Romania (procurement), Indonesia (goods and services), Malaysia (industry), India (harmonised system), Kenya (item code) |
| Administrative subdivision | name, parent, code | Peru (district), Brazil (city with postal code range), China (city) |
| Payment method code | code, name | Ecuador, Italy, Spain, Greece |
| Unit of measure code | internal unit of measure, country code | Egypt, Indonesia |
| Activity code | code, name | Egypt |
| Responsibility type | code, name, sequence | Argentina |
| Exemption or reason code | code, name, applicability | Turkey, Kenya, Spain, India, Saudi Arabia |
| Numbering symbol or template | code, name, validity | Vietnam |

Each package's exact tables are listed in its country file.

---

## 10. Scheduled jobs

Every job below is contributed by a country package. All of them are deactivated until the corresponding company setting is filled, because they exit immediately when no company is registered.

| Job | Interval | Entity it runs against | Purpose |
|---|---|---|---|
| Retrieve new documents (Denmark) | every 4 hours | Exchange Proxy User | Fetch inbound documents from the Danish exchange network and create draft vendor bills. |
| Update message status (Denmark) | every day | Exchange Proxy User | Poll the delivery state of documents already sent. |
| Webhook keep alive (Denmark) | every 2 weeks | Exchange Proxy User | Renew the inbound notification subscription. |
| Update participant status (Denmark) | every week | Exchange Proxy User | Refresh the registration state of the company's own endpoint. |
| Submit or cancel records (Spain, verifiable invoice registry) | every day | Verifiable Invoice Document | Transmit queued registrations and cancellations, respecting the administration's next-allowed-batch timestamp. |
| Retrieve new regulatory documents (France) | every 4 hours | Exchange Proxy User | Fetch documents and statuses from the public invoicing portal. |
| Send lifecycles (France) | every 12 hours | Exchange Proxy User | Transmit the mandatory lifecycle statuses of previously sent invoices. |
| Generate periodic reporting flows (France) | every day | Reporting Flow | Build the transaction and payment reporting flows for the current period. |
| Generate daily sales closing (France) | every day | Sale Closing | Produce the immutable daily point of sale closing. |
| Generate monthly sales closing (France) | every month | Sale Closing | Produce the immutable monthly closing. |
| Generate annual sales closing (France) | every 12 months | Sale Closing | Produce the immutable annual closing. |
| Fetch third-party issued invoices (Greece) | every day | Journal Entry | Download invoices issued against the company and create draft vendor bills. |
| Retrieve new documents (Croatia) | every 4 hours | Company | Fetch inbound documents from the Croatian intermediary. |
| Update statuses of documents (Croatia) | every 4 hours | Company | Poll outbound document statuses. |
| Archive signed documents (Croatia) | every 4 hours | Company | Store the signed payloads returned by the intermediary. |
| Update status of pending invoices (Hungary) | every day | Journal Entry | Poll the tax administration for the outcome of pending transmissions and of pending cancellations. |
| Fetch payment status (Indonesia) | every hour | Journal Entry | Poll the status of quick response payment transactions. |
| Receive invoices (Italy) | every day | Journal Entry | Fetch inbound invoices from the exchange system and create draft vendor bills. |
| Document synchronisation (Malaysia) | every hour | Malaysia Interchange Document | Poll the validation outcome of submitted documents. |
| Check invoice status (Poland) | every week | Journal Entry | Poll the national invoice registry for the outcome of submissions. |
| Download vendor bills (Poland) | every 3 hours | Journal Entry | Fetch inbound invoices from the national registry. |
| Refresh access tokens (Poland) | every 6 days | Company | Renew the registry session credentials. |
| Refresh access token (Romania) | every 30 days | Journal Entry | Renew the portal credentials before expiry. |
| Synchronise with the portal (Romania) | every day | Journal Entry | Poll outbound statuses and fetch inbound documents. |
| Retrieve new purchase documents (Turkey) | (package default) | Company | Fetch inbound invoices from the accredited intermediary. |
| Retrieve new sale documents (Turkey, registered channel) | (package default) | Company | Fetch copies of documents sent through the registered channel. |
| Retrieve new archive sale documents (Turkey) | (package default) | Company | Fetch copies of documents sent through the archive channel. |
| Retrieve invoice status (Turkey) | (package default) | Company | Poll outbound statuses. |
| Retrieve printable documents (Turkey) | (package default) | Company | Fetch the printable rendering produced by the intermediary. |

Framework-level jobs the domain relies on but does not own: the exchange proxy client's own polling, the document sending queue and the attachment garbage collector, all specified in [Electronic Invoicing and Document Exchange](../electronic-invoicing-and-document-exchange/interfaces.md) and [the platform scheduled jobs document](../../runtime/scheduled-jobs.md).

---

## 11. Access groups

The domain does not define its own access groups; it reuses the accounting groups and the administration group.

| Group | Canonical name | What it may do in this domain |
|---|---|---|
| System Administrator | Administration / Settings | Load and reload a chart of accounts template; run the purge step; install and uninstall a country package; read and write the credential fields of every electronic invoicing package (portal user name, password, signature key, client identifier, client secret, access token, private key, session key). |
| Accounting Adviser | Accounting / Adviser | Create and edit fiscal positions including foreign registrations; instantiate foreign taxes; create and edit financial reports, lines, expressions and columns; enter financial report external values; create and edit country reference tables; edit tax groups and their payable, receivable and advance accounts; cancel a transmitted electronic document; change the company package in the settings screen. |
| Accounting User | Accounting / Billing | Issue invoices carrying country-specific fields; transmit an electronic document; create, edit and delete withholding lines; print the certification integrity report; read country document records and their exchange history. |
| Internal User | Employee | Read country reference tables (document types, identification types, tax offices, classification codes, unit codes). |
| Point of Sale User | Point of Sale / User | Issue fiscal receipts, which consume the country numbering series and write the certification chain. |

**Industry-standard completion**: the credential fields listed under System Administrator are secrets; a replacement must not expose them to any lower group, must not print them in any report and must not include them in an export.

---

## 12. Master data prerequisites

Before a country package can be used, the following must exist.

1. **Country records** with their two-letter code, their currency and, where relevant, their subdivisions. A template's fiscal country is resolved by external identifier on the country record.
2. **Country groups** for the regional rules: the European Union group (used by the intra-union detection and by the one-stop shop), the Gulf Cooperation Council group (used by the Gulf invoice layout), and the West African harmonised accounting system does not use a group but a fixed list of sixteen countries.
3. **Currency records** for every fiscal country's currency, activated automatically when the template writes the company currency.
4. **Report tags** for the country's return, created by the country's report definition before any template tax is loaded. A tax that names a tag that does not exist aborts the load.
5. **Account tags** used by the utility accounts, in particular the Investing activity tag carried by the two cash difference accounts.
6. **Numbering sequences** for every country document type that is numbered, and for every withholding tax that carries a series.
7. **Unit of measure records** for countries that map internal units onto administration codes.
8. **Language records** for every translation the template ships, so that the translation pass has a target.

---

## 13. Numbering series used by the domain

| Series | Scope | Shape | Consumed by |
|---|---|---|---|
| Withholding number | one per withholding tax | free; typically a prefix and a padded counter | A withholding line without a typed number, at the moment the payment's journal items are prepared. |
| Document type number | one per pair of journal and document type | typically `<point of sale number>-<counter>` with fixed widths | A posted invoice in a journal that uses document types. |
| Certification chain sequence | one per company | a plain counter | Each document added to a country's fingerprint chain (Spain Basque Country, Spain verifiable invoice registry, Saudi Arabia). |
| Sales closing sequence | one per company | a plain counter | Each daily, monthly and annual point of sale closing. |
| Point of sale fiscal receipt sequence | one per point of sale configuration | country-defined | Each validated point of sale order in a country that requires fiscal receipt numbering. |

The numbering mechanism itself (prefix and suffix interpolation, padding, increment step, per-period sub-sequences, gap-free option) belongs to [the platform sequence catalogue](../../references/sequences.md).

---

## 14. Country settings index

The table lists, per country, the company-level settings that country package adds. The full effect of each is in the country's own file.

| Country | Company settings added |
|---|---|
| Argentina | Gross income number, gross income type, responsibility type, activities start date, tax base account for withholding. |
| Australia | Registered for goods and services tax, trading name. |
| Austria | (none beyond the template) |
| Belgium | (none beyond the template; the payment reference model gains a Belgian structured form) |
| Brazil | State tax identification number, municipal tax identification number. |
| Canada | Provincial sales tax number. |
| Chile | Company activity description. |
| Czech Republic | Trade registry, tax office. |
| Denmark | Exchange contact electronic mail address, exchange telephone number, exchange registration state, endpoint type, endpoint value, purchase journal for inbound documents, exchange proxy user. |
| Egypt | Portal client identifier, portal secret, production environment flag, invoicing threshold above which the customer tax number is mandatory. |
| Estonia | Rounding difference loss account, rounding difference profit account. |
| France | Closing numbering series, activity code, part-of-overseas-territory flag, rounding difference loss and profit accounts, point of sale certification sequence, public portal settings (send to the portal, pilot phase, directory start date, registration state, portal identifier, reporting periodicity, reporting enabled, reporting start date, know-your-customer status, authentication identifier). |
| Germany | Tax number, business identification number, forced restrictive audit trail. |
| Greece | Portal user identifier, portal subscription key, branch number, test environment flag. |
| Croatia | Intermediary user name, password, company identifier, software identifier, connection state, operating mode, purchase journal. |
| Hungary | Group tax number, tax regime, server mode, portal user name, password, signature key, replacement key, last transaction recovery timestamp. |
| India | Production environment flag, harmonised system code digit count, withholding features, withholding account, withholding journal, tax deduction account number, registration flag, registration status feature, electronic invoicing credentials and token, way bill credentials and token, forced restrictive audit trail. |
| Indonesia | (settings live on the electronic invoice package: branch number, buyer document type, taxable person flag, transaction code) |
| Italy | National fiscal code, tax system, exchange proxy user, registration flag, purchase journal for inbound invoices, register of companies data (office province, number, share capital, sole shareholder, liquidation state), tax representative flag and contact, declaration of intent tax and fiscal position. |
| Jordan | Income source sequence, secret key, client identifier, taxpayer type, demonstration mode, point of sale enabled and testing mode. |
| Kenya | Fiscal device proxy address, registry activation flag. |
| Luxembourg | (none beyond the template) |
| Malaysia | Exchange proxy user, identification type, identification number, industry classification, operating mode, default import journal, sales and service tax number, tourism tax number. |
| Mexico | Income account for returns and discounts, income account for re-invoicing. |
| Netherlands | Rounding difference loss account, rounding difference profit account. |
| Norway | Register of legal entities number. |
| Peru | (settings live on the contact and the journal) |
| Philippines | Branch code, revenue district office. |
| Poland | Tax office, registry integration flag, certificate, access token, refresh token, session identifier, session key, session initialisation vector. |
| Portugal | (none beyond the template) |
| Romania | Portal client identifier, client secret, access token, refresh token, access expiry date, refresh expiry date, callback address, test environment flag, journal for imported bills. |
| Saudi Arabia | Private key, portal mode, building number, plot identification, additional identification scheme and number, production flag. |
| Serbia | Portal key, demonstration environment flag. |
| Singapore | Unique entity number. |
| Slovakia | Trade registry, income tax identifier. |
| Spain | Simplified invoice limit; per exchange flow: certificates, tax agency, test mode, chain sequence, registry enablement, next batch timestamp, special tax regime. |
| Sweden | Organisation number; per journal, the payment reference length. |
| Switzerland | (the payment reference model gains a Swiss structured form) |
| Turkey | Intermediary key, test environment flag, purchase journal, tax office, export alias. |
| United Arab Emirates | Dual-language invoice layout flag (shared Gulf setting). |
| United Kingdom | (none beyond the template) |
| United States | (none beyond the template; bank accounts gain a routing number and an account type) |
| Uruguay | (settings live on the contact and the journal) |
| Vietnam | Portal user name, password, access token, token expiry, default numbering symbol, point of sale symbol. |
