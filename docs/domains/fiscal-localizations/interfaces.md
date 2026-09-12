# Fiscal Localizations: Interfaces

The service operations a client or an integration invokes, the request endpoints the system exposes, the screens described as workflows on views, the printed documents, the exported files, the notifications and the scheduled jobs of this domain. Nothing here refers to a client technology; a screen is described by the fields it shows, the buttons it offers with their guards, the filters and groupings it supports, and the status bar it displays.

---

## 1. Service operations of the template framework

### 1.1 `select_chart_template(country)`

| Aspect | Detail |
|---|---|
| Inputs | `country` (reference to Country, optional; defaults to the active company's country) |
| Output | An ordered list of pairs `(template_code, display_name)` containing every **visible** template. |
| Ordering | Templates whose country equals `country` first, in their natural order; then every other template. When `country` is empty, the generic chart of accounts comes first instead. |
| Side effects | None. |
| Errors | None. |

### 1.2 `guess_chart_template(country)`

| Aspect | Detail |
|---|---|
| Inputs | `country` (reference to Country) |
| Output | The template code of the first entry of `select_chart_template(country)`. |
| Side effects | None. |
| Errors | None. |

### 1.3 `chart_template_mapping(include_invisible)`

| Aspect | Detail |
|---|---|
| Inputs | `include_invisible` (boolean, default false) |
| Output | A mapping from template code to a descriptor with the keys `name`, `country`, `country_reference`, `country_code`, `parent`, `sequence`, `visible`, `installed` and `contributing_package`. |
| Side effects | None. |
| Errors | None. |

### 1.4 `try_loading(template_code, company, install_demonstration_data, force_create)`

| Aspect | Detail |
|---|---|
| Inputs | `template_code` (text, optional; guessed from the company country when empty), `company` (reference to Company, required), `install_demonstration_data` (boolean, default false), `force_create` (boolean, default true) |
| Output | Nothing. |
| Side effects | Installs the contributing package if needed; sets the company's template code, fiscal country, currency and prefixes; deletes the previous configuration when the company has no accounting; creates accounts, account groups, taxes, tax groups, fiscal positions, journals and reconciliation models; creates the utility accounts; wires the company defaults; writes scoped defaults; loads translations; re-parents account groups; recurses into subsidiaries. |
| Errors | `Only administrators can install chart templates` when the caller is not a system administrator. `The <template code> chart template shouldn't be selected directly. Instead, you should directly select the chart template related to your country.` for the two shared West African bases. `Error while loading the localization: missing tax tag <tag name> for country <country name>. You should probably update your localization app first.` when a template names an unknown report tag. |
| Idempotence | Calling it again with the same template code performs a reload, which is a narrowed update, not a second creation. |

### 1.5 `instantiate_foreign_taxes(country, company)`

| Aspect | Detail |
|---|---|
| Inputs | `country` (reference to Country, required), `company` (reference to Company, required) |
| Output | The created records, grouped by entity. |
| Side effects | Creates tax groups and taxes whose country is `country`, prefixed with the source template code; creates the accounts they need by copying the closest local equivalents; sets the company's cash basis flag when any created tax is exigible on payment. |
| Guard | Returns silently when the company already has at least one tax whose country is `country`. |
| Errors | None beyond the ordinary access errors. |

### 1.6 `load_translations(languages, companies)`

| Aspect | Detail |
|---|---|
| Inputs | `languages` (list of language codes, optional; defaults to every installed language), `companies` (list of companies, optional; defaults to every company having a template) |
| Output | Nothing. |
| Side effects | Writes translations for every translatable field of every account, account group, tax, tax group, fiscal position, journal and reconciliation model that was created from template data, and for records of those entities that were not created from template data but lack a translation. Existing translations are never overwritten. |
| Errors | None; a missing report tag is tolerated during this operation. |

---

## 2. Service operations of the withholding framework

### 2.1 `compute_withholding_lines(base_lines, company)`

| Aspect | Detail |
|---|---|
| Inputs | `base_lines` (the rounded base lines of the invoices in the first batch), `company` |
| Output | A list of create, update and delete instructions for the caller's withholding line list. |
| Algorithm | Re-prepare every base line with withholding taxes enabled and no tax filter; add and round the tax details; aggregate by the key `(name, analytic_distribution, account, tax, skip, currency)`; for each non-skipped key, update the matching existing line or create one; delete every existing line whose key no longer appears; when several existing lines share a key, keep the first and delete the rest. |
| Side effects | None until the instructions are applied. |
| Errors | None. |

### 2.2 `update_withholding_placeholders(lines)`

| Aspect | Detail |
|---|---|
| Inputs | A set of withholding lines belonging to one carrier. |
| Output | Nothing. |
| Side effects | Writes `placeholder_value` and `previous_placeholder_type` on every line as described in [calculations.md](calculations.md) section 6.7. No numbering series value is consumed. |
| Errors | None. |

### 2.3 `prepare_withholding_journal_items(lines)`

| Aspect | Detail |
|---|---|
| Inputs | A set of withholding lines belonging to one carrier. |
| Output | A list of journal item values: one tax line per aggregated withholding tax, and one base line plus one base counterpart line per aggregated base key. |
| Side effects | Consumes the numbering series of every line that has no number, and writes the produced number on the line. This happens **after** the completeness check, so an incomplete set consumes nothing. |
| Errors | `Please enter the withholding number for the tax <tax name>` when a line has neither a number nor a series. `All withholding lines in self must have the same payment.` or `All withholding lines in self must have the same payment register.` when the set spans several carriers. |

### 2.4 `withholding_tax_domain(company, payment_direction)`

| Aspect | Detail |
|---|---|
| Inputs | `company`, `payment_direction` (`inbound` or `outbound`) |
| Output | A condition selecting the taxes available for withholding: `company IS company OR ancestor-of company`, `tax_scope = "purchase"` for an outbound payment or `"sale"` for an inbound one, and `withhold_on_payment = true`. |
| Side effects | None. |

---

## 3. Service operations of the report framework

### 3.1 `expand_aggregations(expressions)`

| Aspect | Detail |
|---|---|
| Inputs | A set of report expressions. |
| Output | The same set plus every expression it transitively depends on. |
| Algorithm | Repeatedly, for every aggregation expression not yet expanded: when the formula is `sum_children`, add the same-labelled expression of every child line; otherwise parse the formula into `line_code` to expression-label pairs, restrict the search to the same report or to the report named by a `cross_report(...)` subformula, and add every matching expression. Stop when no new expression is added. |
| Errors | `In report '<report>', on line '<line>', with label '<label>', The format of the cross report expression is invalid. Expected: cross_report(<report identifier>|<external identifier>) Example: cross_report(my_module.my_report) or cross_report(123)`; `In report '<report>', on line '<line>', with label '<label>', Failed to parse the cross report id or xml_id.`; `You cannot use cross report on itself`; `Cannot get aggregation details from a line not using 'aggregation' engine`. |

### 3.2 `matching_report_tags(expressions)`

| Aspect | Detail |
|---|---|
| Inputs | A set of report expressions. |
| Output | Every report tag, both variants, whose name matches the formula of a tax-tag expression in the set, restricted to the report's country. |
| Side effects | None. |

### 3.3 `carryover_target(expression, options)`

| Aspect | Detail |
|---|---|
| Inputs | A carryover expression and the report options. |
| Output | The expression that receives the carried amount. |
| Errors | `Could not determine carryover target automatically for expression <label>.` |

### 3.4 `copy_report(report)`

| Aspect | Detail |
|---|---|
| Inputs | A report. |
| Output | The copy. |
| Side effects | Copies the report, every line recursively with a `_COPY` code suffix, every expression, and every column; rewrites every aggregation formula and subformula to use the new codes. |
| Errors | None. |

---

## 4. Request endpoints

Every endpoint below is contributed by a country package. The authentication level is stated for each. An endpoint whose authentication level is "public" performs its own credential check on the payload.

| Path pattern | Method | Authentication | Purpose | Request | Response |
|---|---|---|---|---|---|
| `/nemhandel/webhook/new-message` | POST | public, with a token in the query | The Danish exchange service announces that new inbound documents are waiting. | A token identifying the registered proxy user. | Empty body with a no-content status. As a side effect the "retrieve new documents" scheduled job is woken immediately. |
| `/nemhandel/webhook/message-state-update` | POST | public, with a token in the query | The Danish exchange service announces that the state of a sent message changed. | A token. | Empty body with a no-content status; the "update message status" job is woken. |
| `/nemhandel/webhook/user-state-update` | POST | public, with a token in the query | The Danish exchange service announces that the company's own registration state changed. | A token. | Empty body with a no-content status; the "update participant status" job is woken. |
| `/l10n_ro_edi/authorize/<company identifier>` | GET | authenticated user | Starts the Romanian portal authorisation. | Nothing beyond the company in the path. | A redirect to the portal's authorisation page carrying the response type, the client identifier, the callback address and the token content type. Refused with `Client ID and Client Secret field must be filled.` when either credential is missing. |
| `/l10n_ro_edi/callback/<company identifier>` | GET | authenticated user | Receives the Romanian portal's authorisation answer and exchanges it for tokens. | An access key in the query. | Stores the access token, the refresh token and the two expiry dates on the company. Refused with `Access key not found. Please try again. Response: <parameters>` followed by `Received access key: <key>` when the key is absent, which happens when no certificate was presented. |
| `/api/signaturit_authentication_status/1/webhooks` | POST | public | Receives the identity-verification status of the French portal registration. | The verification outcome. | Empty body; the company's know-your-customer status is updated. |
| `/portal/state_infos/<country subdivision identifier>` | POST | public | Feeds the Peruvian address form: lists the cities of a subdivision. | The subdivision in the path. | A list of triples `(city identifier, city name, city code)`. |
| `/portal/city_infos/<city identifier>` | POST | public | Feeds the Peruvian address form: lists the districts of a city. | The city in the path. | A list of triples `(district identifier, district name, district code)`. |
| `/l10n_id_efaktur_coretax/download_attachments/<attachment identifiers>` | GET | authenticated user | Downloads the Indonesian electronic invoice payloads. | The attachments in the path. | The files, as a single archive when several are requested. |
| `/invoice/ecpay/agreed_invoice_allowance/<invoice identifier>` | POST | public | Receives the Taiwanese buyer's agreement to a credit note. | The agreement outcome. | Empty body; the invoice's refund state becomes Agreed or Disagreed. |
| `/shop/l10n_tw_invoicing_info` | GET | public | Shows the Taiwanese invoicing preferences form during checkout. | Nothing. | The form: carrier type, carrier number, donation code and paper-copy request. |
| `/shop/l10n_tw_invoicing_info/submit` | POST | public | Stores the Taiwanese invoicing preferences on the order. | The form fields. | A redirect back to the checkout. |
| `/payment/ecpay/check_mobile_barcode/<order identifier>` | POST | public | Validates a Taiwanese mobile carrier barcode against the provider. | The barcode. | Whether the barcode is valid. |
| `/payment/ecpay/check_love_code/<order identifier>` | POST | public | Validates a Taiwanese donation code against the provider. | The code. | Whether the code is valid. |

**Industry-standard completion**: every public endpoint above must reject a request whose token or signature does not match, must not disclose whether a company exists, and must be rate limited, because it is reachable without authentication.

---

## 5. Screens

Screens are described as workflows on views. A field marked "read-only" cannot be typed into; a field marked "conditional" appears only when its condition holds.

### 5.1 Accounting settings, fiscal localization section

**View kind.** Form, one record (the settings of the active company).

| Field shown | Condition | Behavior |
|---|---|---|
| Package | always | A selection of every visible template. Saving a different value triggers a load. |
| Fiscal country | always | Drives the visibility of every country-specific field elsewhere. |
| Bank account code prefix, cash account code prefix, transfer account code prefix | always | Changing a liquidity prefix renumbers the matching accounts. |
| Default sale tax, default purchase tax | always | Proposed on new lines. |
| Rounding method | always | Round per Tax or Round per Line. |
| Use cash basis, cash basis journal, cash basis base account | the company has at least one tax exigible on payment | Reveals the cash basis wiring. |
| Withholding tax base account | the withholding framework is installed | Fixes the account of every withholding base line. |
| Storno accounting | the fiscal country is in the mandatory or the optional set | Hidden elsewhere. |
| Restrictive audit trail | always | Cannot be switched off when a localization forces it. |
| Country credential blocks | one per installed country package whose country is enabled | Portal user names, passwords, keys, certificates, environment switches and purchase journals. |

**Buttons.**

| Button | Guard | Effect |
|---|---|---|
| Save | the user is an accounting adviser | Applies every change, including a template load when the package changed. |
| Register (per country flow) | the country's credentials are complete | Opens the country's registration wizard. |
| Deregister (per country flow) | the company is registered | Removes the registration from the exchange service. |

### 5.2 Chart of accounts

**View kind.** List, grouped by account group.

| Column | Notes |
|---|---|
| Code | Padded to the template's code length. |
| Name | Translatable. |
| Type | The account type. |
| Allow reconciliation | Never re-imposed by a reload. |
| Tags | The report tags refreshed by a reload. |
| Current balance | Derived from posted journal items. |

**Filters.** By account type, by whether the account is reconcilable, by whether it carries journal items, by account group.

**Buttons.** Create, duplicate (which searches a free code), archive. Deleting is refused when journal items exist (`You cannot perform this action on an account that contains journal items.`) or when a fiscal position maps the account (`You cannot remove/deactivate the accounts "<code> - <name>" which are set on the account mapping of a fiscal position.`).

### 5.3 Fiscal position form

**View kind.** Form.

| Field | Condition | Notes |
|---|---|---|
| Name, company, sequence | always | |
| Detect automatically | always | Reveals the five detection predicates. |
| Tax identification number required | detect automatically is on | |
| Country, country group, country subdivisions | detect automatically is on | |
| Postal code from, postal code to | detect automatically is on | Both required together. |
| Foreign tax identification number | always | Validated against the position's country. |
| Tax mapping (source tax, replacement taxes) | always | A line with no replacement removes the source tax. |
| Account mapping (source account, destination account) | always | Unique per triple. |

**Buttons.**

| Button | Guard | Effect |
|---|---|---|
| Create foreign taxes | the foreign registration header mode is "Templates Found" | Runs `instantiate_foreign_taxes` and links the created taxes to the position. |

**Status bar.** None; the fiscal position has no state.

### 5.4 Financial report designer

**View kind.** Form with three embedded lists: lines, columns and, per line, expressions.

| Field | Notes |
|---|---|
| Name, country, chart of accounts template, availability | The availability drives whether the report is offered. |
| Root report, sections | A root report may not itself have a root; a section may not have sections. |
| Every filter of [configuration.md](configuration.md) section 7.2 | Shown as switches and selections. |
| Lines: name, code, sequence, parent, level, group by, foldable, hide if zero, print on new page | Ordered by sequence; a parent must precede its children. |
| Expressions: label, engine, formula, subformula, date scope, figure type, blank if zero, auditable, carryover target | Validated on save. |
| Columns: name, expression label, sequence, sortable, figure type, blank if zero | |

**Buttons.** Duplicate (copies the whole tree and rewrites the formulas), archive. Deleting is refused when the report has variants (`You can't delete a report that has variants.`).

### 5.5 Financial report viewer

**View kind.** A hierarchical figure table.

| Element | Behavior |
|---|---|
| Period selector | Defaults to the report's default opening period. Offers the return period when the report is periodic. |
| Comparison selector | Offers a previous period and a growth comparison when the corresponding filters are on. |
| Company or tax unit selector | According to the multi-company filter. |
| Foreign registration selector | Shown when the report allows foreign registrations and the company holds at least one. |
| Line rows | Indented by level; foldable lines show an expansion control; a line with a grouping expands into one row per distinct value. |
| Figure cells | Rendered according to the column's figure type and the report's integer rounding; a zero is blank when the column or the expression says so. |
| Drill-down | Clicking an auditable figure opens the journal items behind it. A custom-function figure is not auditable. |
| Editable cells | An external-value expression whose subformula contains `editable` accepts a typed figure, rounded to the declared number of decimal places. |

**Buttons.** Export, print, close the period. Closing writes the entry described in [accounting-effects.md](accounting-effects.md) section 3 and moves the tax lock date.

### 5.6 Invoice form, country sections

Every country package adds a section to the invoice form, shown only when the country is enabled for the company. The recurring shape is:

| Element | Behavior |
|---|---|
| Document class and number | Shown when the journal uses document types. The class list is restricted by the counterpart's category and the entry kind. The number is typed when the journal does not number automatically. |
| Country identification fields | Place of supply, responsibility type, concept, service period, transaction code, exemption reason, classification codes. |
| Exchange state badge | A coloured badge showing the country exchange state. |
| Exchange header | A description of the state and of the next action. |
| Registration number and visual code | Shown once the administration accepted the document. |
| Error panel | The administration's rejection details, rendered as a list. |

**Buttons.**

| Button | Guard | Effect |
|---|---|---|
| Send | the invoice is posted, eligible, and has no blocking validation error | Builds and transmits the payload. |
| Check status | a transmission is pending | Polls the administration. |
| Cancel | the country allows cancellation and the window is open | Opens the cancellation wizard, which requires a reason code and, in several countries, free remarks. |
| Download payload | a payload exists | Downloads the transmitted file. |
| Print | the document is in a printable state | Produces the country layout. |

### 5.7 Withholding section of the payment registration wizard

**View kind.** A section of the wizard form containing an editable list.

| Element | Condition | Behavior |
|---|---|---|
| Withhold tax amounts switch | the company has a withholding tax matching the direction and the wizard will create one entry | Reveals the list. |
| Line: tax | always | Restricted to withholding taxes of the matching scope. |
| Line: number | always | Empty shows the placeholder the series would give. |
| Line: base | always | Editable; recomputes the withheld amount. |
| Line: withheld amount | always | Editable. |
| Line: account | the company sets no withholding tax base account | Otherwise hidden and defaulted. |
| Line: analytic distribution | always | Copied onto the base and tax lines. |
| Net amount | always | `amount − Σ line.amount`, read-only. |
| Outstanding account | the payment method line has no payment account | Required; proposed from the most recent comparable payment. |

**Buttons.**

| Button | Guard | Effect |
|---|---|---|
| Create payment | the net amount is not negative; every line has a number or a series; every base is positive; every account is valid | Creates the payment with its withholding lines and posts it. |

### 5.8 Country exchange document list

**View kind.** List with a form.

| Column | Notes |
|---|---|
| Reference | The document's own number. |
| Date or timestamp | When the interaction happened. |
| Related invoice or receipt | Link. |
| State | Coloured badge. |
| Chain index | Where the country chains documents. |
| Errors | Truncated in the list, full in the form. |

**Filters.** By state, by date range, by whether the document is still waiting, by company.

**Groupings.** By state, by month, by related journal.

**Buttons.** Submit, poll, cancel, download payload, regenerate payload. Deleting is refused by the country's own guard.

### 5.9 Country reference table list

**View kind.** List with an inline form.

| Column | Notes |
|---|---|
| Code | The administration's code. |
| Name or description | Translatable where the package marks it so. |
| Auxiliary columns | Country, region, parent, rate, applicability, depending on the table. |
| Active | Where the table supports archiving. |

**Filters.** By country where the table has one, by active state.

**Buttons.** Create and edit are restricted to the accounting adviser group; everyone may read.

---

## 6. Printed documents

| Document | Countries | Content, grouping and totals |
|---|---|---|
| Invoice with document class | Argentina, Chile, Colombia, Ecuador, Peru, Uruguay | The ordinary invoice plus the document class wording, the legal number built from the emission point and the counter, the counterpart's identification class and number, the issuer's responsibility category and, where required, the activities start date and the gross income registration. |
| Invoice with the Germany-style layout | Austria, Germany, Switzerland and every package that adopts the layout | A fixed geometry with an address window, a reference block carrying the document number, the date, the customer number and the contact, an optional line position column, and the totals block. |
| Dual-language invoice | Bahrain, Kuwait, Oman, Qatar, Saudi Arabia, United Arab Emirates | Every label and every line description rendered in both the local language and English, with the tax total shown separately. |
| Invoice with a payment visual code | Switzerland, Indonesia, Hong Kong, Singapore, Thailand, Vietnam, Brazil | A visual code encoding the payment instruction, placed in the payment block. |
| Invoice with a registration visual code | Saudi Arabia, Spain, Egypt, Greece, Malaysia, Jordan, Vietnam, Taiwan | A visual code encoding the registration data or the verification address returned by the administration. |
| Transport note | Italy | The transport note number and date, the carrier, the number of packages, the weight and the goods description, grouped by line. |
| Goods movement permit | India | The permit number, the issue and expiry dates, the document reference, the two billing parties, the two shipping parties, the distance, the transport mode, the vehicle registration and the line values with their taxes. |
| Delivery note with a transport declaration | Romania | The ordinary delivery note plus the transport declaration identifier. |
| Point of sale certification integrity result | France | For each company, whether certification is in force, the first and last certified receipt, their fingerprints and the first divergence found, if any. |
| Withholding certificate | Argentina, India, Philippines, Saudi Arabia, Turkey | The payer, the payee, the base, the rate, the withheld amount, the certificate number and the period. |
| Compliance letter | Malta | A letter stating the company's exemption registration, dated in the long form. |

**Guard on the integrity result.**

```
Please contact your accountant to print the Hash integrity result.
Accounting is not unalterable for the company <company>. This mechanism is designed for companies where accounting is unalterable.
```

---

## 7. Exported files

| Export | Country | Content | Trigger |
|---|---|---|---|
| Statutory accounting entries file | France | Every posted journal item of the period, one row per item, with the journal code and name, the entry number and date, the account code and name, the counterpart code and name, the document reference and date, the label, the debit, the credit, the reconciliation reference and date, the validation date, the amount in currency and the currency code. An unaffected-earnings opening row aggregates the accounts whose balance is not carried forward. | A wizard taking a start date, an end date, an official or non-official mode and a list of excluded journals. |
| Tax audit export | Hungary | Every invoice in the selected range, selected either by date or by serial number, with its full transmitted payload. | A wizard taking a selection mode, two dates or two numbers. |
| Withholding data export | Philippines | One row per withheld amount with the payee, the code, the base and the amount, laid out for the administration's uploader. | A wizard taking the entries to include. |
| Electronic invoice payload | every country with an exchange flow | The country payload, attached to the invoice and downloadable. | The Send or Download button. |
| Consolidated receipt payload | Malaysia | One document carrying every receipt of the period below the threshold, split into lines whenever continuity breaks. | The consolidation wizard taking a start date, an end date and a consolidation kind. |
| Ledger export | Bulgaria | The sales and purchase ledgers with the national document class and number per entry. | A report export. |
| Annexed declaration | Estonia, Peru, Ecuador | The transaction-level annex required beside the periodic return. | A report export. |

---

## 8. Notifications

| Notification | Trigger | Recipients | Content |
|---|---|---|---|
| Transmission accepted | The administration accepts a document. | The followers of the invoice. | The registration number, the acceptance timestamp and a link to the payload. |
| Transmission rejected | The administration rejects a document. | The followers of the invoice. | The full error list as reported, one line per error. |
| Cancellation accepted or rejected | The administration answers a cancellation request. | The followers of the invoice. | The outcome and the reason. |
| Threshold crossed | A draft bill crosses a withholding section's per-transaction or cumulative threshold. | The user editing the bill. | The section, the threshold and the amount reached. |
| Declaration of intent exceeded | A document would push a declaration's remaining amount below zero, or uses a revoked declaration. | The user editing the document. | The declaration, the amount invoiced, the amount planned and the overrun. |
| Registration state changed | An exchange service reports that the company's own registration changed. | The administrator. | The new state. |
| Flow contains invalid entries | A periodic reporting flow is built with entries carrying blocking errors. | The followers of the flow. | The count of invalid entries and a link to them. |
| Bank account not in the register | A Polish supplier's account fails the register check. | The user registering the payment. | The verification status and the correlation identifier. |

---

## 9. Scheduled jobs

The complete list, with intervals and purposes, is in [configuration.md](configuration.md) section 10. The behavioral requirements common to all of them are:

1. **Company iteration.** Each job iterates the companies that have the country's credentials filled and exits immediately for the others.
2. **Batching and committing.** A job that transmits or polls many documents processes them in batches and commits between batches, so that a failure late in the run does not lose the results already obtained.
3. **Locking.** A document being processed is locked so that a manual send and the job cannot act on it at once. A second attempt is refused with the country's message, for example `This document is being sent by another process already.`
4. **Rescheduling.** A job that is told by an administration to wait re-arms itself at the stated time rather than retrying immediately. The Spanish verifiable invoice job stores that instant on the company.
5. **Waking.** A job may be woken before its next scheduled run by an inbound notification endpoint; see section 4.
6. **Failure handling.** A job that fails repeatedly deactivates itself and the failure is visible on the job record. **Industry-standard completion**: the failure must be recorded with enough detail for an operator to distinguish a credential problem from an administration outage.

---

## 10. Wizards

| Wizard | Purpose | Inputs | Outputs and errors |
|---|---|---|---|
| Cancel electronic invoice (India) | Request cancellation of a registered invoice. | Invoice, cancel reason (required), cancel remarks (required). | Transmits the cancellation; the invoice state becomes Cancelled. |
| Cancel goods movement permit (India) | Cancel a permit. | Permit, cancel reason (required), cancel remarks. | Transmits the cancellation; the permit state becomes Cancelled. |
| Create withholding entry (India) | Create and post a withholding entry against an invoice or a payment. | Reference, related invoice or payment, journal, date, section tax, base, computed amount. | Posts the entry. Errors: `TDS must be created from an Invoice or a Payment.`, `You can only create a withhold for only one record at a time.`, `TDS must be created from Posted Customer Invoices, Customer Credit Notes, Vendor Bills or Vendor Refunds.`, `Please set a partner on the <record> before creating a withhold.`, `Negative or zero values are not allowed in Base Amount for withhold`, `Negative or zero values are not allowed in TDS Amount for withhold`, `Please configure the withholding account from the settings`. |
| Technical annulment (Hungary) | Request annulment of a transmitted invoice. | Invoice, annulment code (`ERRATIC_DATA`, `ERRATIC_INVOICE_NUMBER`, `ERRATIC_INVOICE_ISSUE_DATE`), reason (required). | Transmits the request; the invoice state becomes Cancellation request sent. |
| Receive bills (Hungary) | Fetch inbound invoices for a time window. | From timestamp, to timestamp. | Creates draft bills. Error: `The length of the interval specified by the query parameter can be up to 35 days.` |
| Tax audit export (Hungary) | Produce the audit file. | Selection mode (by date or by serial number), two dates or two numbers. | The file. Error: `No invoice to export!` |
| Reject invoice (Croatia) | Send a business-level rejection. | Invoice, rejection type (`N` data discrepancy without tax effect, `U` data discrepancy with tax effect, `O` other), description (required). | Transmits the rejection. |
| Consolidate receipts (Malaysia) | Build one document for a period's small receipts. | From date, to date, consolidation kind. | Creates the consolidated document. Errors: `Support for consolidated invoices in the invoicing app is not yet implemented.`, `Invalid Operation. No order to consolidate.` |
| Update document status (Malaysia) | Cancel or reject a validated document. | Document, reason (required), new status. | Transmits the change. Error: `You must provide a reason for updating the document.` |
| Request one-time password (Saudi Arabia) | Obtain the onboarding certificate. | Journal, renewal flag, one-time password. | Stores the certificate. Error: `Please provide an OTP to complete the onboarding process` |
| Registration (Denmark) | Register the company on the exchange network. | Contact electronic mail address, telephone number, endpoint type, endpoint value, operating mode, verification code. | Registers the company. Errors: `Cannot register a user with a <mode> application`, `Please enter a phone number to verify your application.`, `Please enter a primary contact email to verify your application.`, `Please fill in your company's VAT`, `Contact email and phone number are required.`, `Please first verify your phone number by clicking on 'Send a registration code by SMS'.`, `The verification code should contain six digits.`, `Connection error, please try again later.` |
| Rejection (Denmark) | Send a business-level rejection. | Entries, additional note. | Transmits the rejection. |
| Cancel invoice (Vietnam) | Request cancellation. | Invoice, reason (required), agreement document name, agreement document date. | Transmits the request. |
| Cancel invoice (Taiwan) | Request cancellation. | Invoice, reason (required). | Transmits the request. Error: `You must provide a reason for canceling the invoice.` |
| Print invoice (Taiwan) | Request a printed copy from the provider. | Invoice, business-to-consumer format (single-sided, double-sided, thermal), business-to-business format (two paper sizes). | Requests the print. Error: `Error: <provider message>` |
| Send reporting flow (France) | Send a flow that contains invalid entries. | Flow, warning text. | Sends the flow excluding the invalid entries. |
| Statutory entries export (France) | Produce the accounting entries file. | Start date, end date, export kind (official or non-official), excluded journals. | The file. |
| Handle payment visual code problems (Switzerland) | Print a batch of invoices when some cannot carry a payment visual code. | The counts and the two explanatory texts. | Prints the compliant ones and offers a list of the others. Errors: `No invoice was found to be printed.`, `All selected invoices must belong to the same Switzerland company` |
| Compliance letter (Malta) | Produce the exemption letter. | Company. | The letter. Error: `Compliance letters can only be created for companies registered in Malta. Please ensure the company's country is set to Malta.` |
| Withholding data export (Philippines) | Produce the withholding upload file. | Entries to include. | The file. |
| Mass check transfer (Latin America) | Move several third-party checks to another journal. | Payment date, destination journal, memo, checks. | Creates one internal transfer per check. Errors: `All selected checks must be on the same journal and on hand`, `The register payment wizard should only be called on account.payment records.`, `You have selected payments which are not checks. Please call this action from the Third Party Checks menu`, `All the selected checks must be posted`, `All the selected checks must use the same currency` |
