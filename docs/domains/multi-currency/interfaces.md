# Multi-Currency — Interfaces

The named operations a client or an integration invokes, the screens described by the fields they
show and the controls they offer, the printed output, the scheduled job, the notifications and the
integration contracts. No client technology is named anywhere: a screen is described by what it
shows, what it lets a user do and what guards those controls.

**How operations are named.** Every operation below is named in words, because an operation name is
not part of the contract this specification preserves: the contract is the operation's inputs, its
output, its side effects and its error paths, and those are stated in full. Identifiers that *are*
contractual — field identifiers, stored selection values and reproduced messages — appear in code
font or in quotation marks as everywhere else in this repository.

Contents:

1. [Named operations](#1-named-operations)
2. [Screens](#2-screens)
3. [Printed documents and reports](#3-printed-documents-and-reports)
4. [Scheduled jobs](#4-scheduled-jobs)
5. [Notifications and messages](#5-notifications-and-messages)
6. [Integration contracts](#6-integration-contracts)
7. [Reconciliation notes](#7-reconciliation-notes)

---

# 1. Named operations

## 1.1 Conversion and rate lookup

### Convert an amount

| Aspect | Specification |
|---|---|
| Inputs | The amount, the source currency, the target currency, optionally a company defaulting to the company the caller is working in, optionally a date defaulting to today in the caller's time zone, and optionally a flag suppressing the rounding, which defaults to rounding. |
| Output | A number expressed in the target currency. |
| Behaviour | The algorithm of [calculations.md](calculations.md) section 8: one composed factor, one multiplication, one rounding onto the target currency's rounding factor. |
| Side effects | None. The rate lookup result is cached, keyed on the two currencies, the company and the date, and the cache is cleared by any change to a rate row. |
| Error paths | Both currencies absent: the operation fails, because an amount cannot be converted from an unknown currency. One currency absent: the missing one is taken to be the given one and the conversion is the identity. A currency with no rate row resolves to a rate of one (`MCUR-030`) rather than failing. |

### Get the conversion factor between two currencies

| Aspect | Specification |
|---|---|
| Inputs | The source currency, the target currency, optionally a company, optionally a date. |
| Output | The multiplicative factor of [calculations.md](calculations.md) section 8.1. Exactly one when the two currencies are the same record. |
| Side effects | None. |
| Error paths | None. |

### Get the rates in force for a set of currencies

| Aspect | Specification |
|---|---|
| Inputs | A set of currencies, a company, a date. |
| Output | A mapping from currency to the technical rate in force, resolved by the lookup of [calculations.md](calculations.md) section 7. |
| Side effects | None. |
| Error paths | An empty input set returns an empty mapping. |

## 1.2 Currency-aware arithmetic

| Operation | Inputs | Output | Behaviour |
|---|---|---|---|
| Round an amount onto a currency | One currency, one amount | A number | [calculations.md](calculations.md) section 3. Exactly one currency must be supplied; supplying several is an error condition, because one amount cannot belong to two currencies. |
| Compare two amounts at a currency's precision | One currency, two amounts | Minus one, zero or one | [calculations.md](calculations.md) section 4. Each operand is rounded before the subtraction. |
| Test an amount for zero at a currency's precision | One currency, one amount | True or false | [calculations.md](calculations.md) section 5. The amount is rounded and its absolute value compared strictly against the rounding factor. |
| Format an amount for a reader | One currency, one amount, optionally a language, optionally a flag suppressing trailing zeros | Text | [calculations.md](calculations.md) section 24. |
| Spell an amount in words | One currency, one amount | Text | [calculations.md](calculations.md) section 25. Returns the empty string and logs a warning when no spelling facility is available; falls back to English when the reader's language has no spelling rules. |
| Divide exactly into whole multiples | One precision, a dividend, a divisor | A quotient and a remainder | [calculations.md](calculations.md) section 6. |

## 1.3 Catalogue and capability operations

### Fetch the cached catalogue of active currencies

| Aspect | Specification |
|---|---|
| Inputs | None. |
| Output | A mapping from currency identifier to a record carrying the currency code, the symbol, the symbol position and the decimal place count. The decimal place count is delivered as a pair whose first element is a large upper bound on the significant digits and whose second element is the currency's own decimal place count, which is the shape the display layer expects. |
| Scope | Active currencies only. |
| Side effects | Cached across requests and invalidated by the five keys of `MCUR-014`. Executed with elevated rights, because every reader needs it. |

### Test whether a currency has been used in accounting

| Aspect | Specification |
|---|---|
| Inputs | One currency. |
| Output | True when at least one journal item exists whose item currency or whose company currency is that currency. |
| Used by | The precision guard of `MCUR-006`. |
| Side effects | None. Executed with elevated rights, because the caller may not be allowed to read every journal item. |

### Re-evaluate the multi-currency capability

| Aspect | Specification |
|---|---|
| Inputs | None. |
| Output | None. |
| Behaviour | Counts the active currencies and grants or revokes the multi-currency permission group on the internal-user group by `MCUR-008`. Granting also grants the price list capability and creates or reactivates a default price list per company when the price list capability was not already held. |
| Called by | Every Currency creation, every deletion, and every write that touches the activity flag. |

## 1.4 Document rate operations

### Refresh the document rate

| Aspect | Specification |
|---|---|
| Target | One or more Journal Entries that are invoices, bills, credit notes, debit notes or receipts. |
| Inputs | None beyond the target. |
| Output | None. |
| Side effects | Sets the document rate (`invoice_currency_rate`) to the expected rate (`expected_currency_rate`), which re-derives every line's balance while preserving every amount in currency. |
| Guards | The document must be in draft; on a posted document the general ledger's posted-entry protection refuses the write (`MCUR-085`). |

## 1.5 Reporting rate table operations

| Operation | Inputs | Output | Behaviour |
|---|---|---|---|
| Test whether a set of companies is single-currency | A set of companies | True or false | True when every company in the set shares one main currency. |
| Build the synthetic single-currency table | A set of companies, a flag requesting the translation-adjustment rate types | A table definition | A table in which every factor is one, with one row per company and per requested rate type and no date bounds. No temporary storage is created. |
| Build a simple rate table | A set of companies | A table definition | Returns the synthetic table when the set is single-currency; otherwise builds the full table for a single period ending today and returns a reference to it. |
| Build a full rate table | A set of companies, a list of periods each carrying a key, a start date and an end date, and a flag requesting the translation-adjustment rate types | None; a table exists for the duration of the operation | Builds the current rows always, and the historical and average rows when the flag is set, by [calculations.md](calculations.md) section 20. |

The table's columns are the company, the period key, the date the row is valid from, the date of
the next change, the rate type and the factor. The rate type carries one of the three stored values
`historical`, `current` and `average`.

## 1.6 Data feeds for embedded clients

### Fetch the company currency for a spreadsheet

| Aspect | Specification |
|---|---|
| Inputs | Optionally a company, defaulting to the company the caller is working in. |
| Output | A record with four members: the currency code, the symbol, the decimal place count and the symbol position. Returns false when the company does not exist. |
| Access | Read-only; callable by any reader. |

### Fetch one conversion factor for a spreadsheet

| Aspect | Specification |
|---|---|
| Inputs | The source currency code, the target currency code, optionally a date, optionally a company. |
| Output | The conversion factor between the two codes. Returns false when either code is empty or matches no currency. |
| Behaviour | The codes are matched against the currency code with archived currencies included, so that a spreadsheet formula keeps working after a currency has been deactivated. |

### Fetch many conversion factors for a spreadsheet

| Aspect | Specification |
|---|---|
| Inputs | A list of requests, each carrying a source code, a target code, optionally a date and optionally a company. |
| Output | The same list with a factor added to each request. |
| Access | Read-only. The batch form exists so that a spreadsheet holding hundreds of conversion formulas issues one call. |

### Load the currencies a point of sale session needs

| Operation | Output |
|---|---|
| The session's currency filter | The filter selecting the currencies a point of sale session needs: the company's main currency, the session configuration's currency, and the currency of every price list loaded into the session. |
| The session's currency field list | The fields loaded for each currency: the internal identifier, the currency code, the symbol, the symbol position, the rounding factor, the current rate, the decimal place count and the numeric code of the international currency-code standard. |

---

# 2. Screens

## 2.1 The currency catalogue, list

**Reached from** Accounting, Configuration, Accounting, Currencies. **Visible to** holders of the
accounting manager group. The action forces archived records to be included, so the list shows
active and archived currencies together, archived rows rendered in a muted style.

| Column | Field | Notes |
|---|---|---|
| Currency | `name` | The three-letter code. |
| Symbol | `symbol` | |
| Name | `full_name` | Optional column, shown by default. |
| Last Update | `date` | The rate date of the most recent rate row. |
| Current Rate | `rate` | Rendered with twelve digits of which six are fractional. |
| Inverse rate | `inverse_rate` | Optional column, hidden by default, rendered with the same digits. |
| Active | `active` | A toggle. Switching it writes the record immediately and runs the guard of `MCUR-007` and the side effects of `MCUR-008` and `MCUR-010`. |

**Filters.** "Active", selecting the currencies whose activity flag is true, with the help text
"Show active currencies"; and "Inactive", selecting those whose activity flag is false, with the
help text "Show inactive currencies".

**Search.** A single field matching the typed term against the currency code, the full name, the
symbol, the currency unit label or the currency subunit label; any one of them containing the term
is a match (`MCUR-017`).

**Ordering.** Activity flag descending, then currency code ascending, so active currencies come
first and each block is alphabetical (`MCUR-012`).

## 2.2 The currency catalogue, card view

A compact alternative for narrow displays. Each card shows the currency code in bold, the symbol in
a pill, the badge `inactive` when the record is archived, the rate as text (`rate_string`) line,
and, when the currency has at least one rate row, the words "Last update:" followed by the last
rate date.

## 2.3 The currency form

**Banner.** When the derived flag `is_current_company_currency` is true, an informational banner
reads "This is your company's currency."

**First group, left column.** The currency code, the full name shown under the label "Name", and
the activity flag as a toggle.

**First group, right column.** The symbol, the currency unit label, the currency subunit label and
the symbol position.

**The Price Accuracy group.** Shown only to holders of the technical features group. It holds the
rounding factor and the decimal places. Immediately after the decimal places, when the derived flag
`display_rounding_warning` is true, a red panel appears containing:

- the heading "WARNING - This change is irreversible";
- the paragraph "You are changing decimals in your entire database, including invoices, tax
  amounts, accounting amounts, reports. This is probably not intended.";
- a control that opens the decimal precision registry described in
  [configuration.md](configuration.md) section 5.

**The Rates tab.** Hidden when the currency is the main currency of the company in context, because
a currency has no rate against itself. It shows the rate rows in a list that is editable in place,
limited to twenty-five rows at a time, newest first, with new rows added at the top and with the
columns of section 2.4.

**Contextual action.** "Show Currency Rates", which opens the rate list filtered on this currency
with the currency pre-filled on new rows.

## 2.4 The rate list

Editable in place. New rows are added at the bottom in the standalone list and at the top in the
currency form's tab.

| Column | Field | Notes |
|---|---|---|
| Date | `name` | Required, defaults to today in the reader's time zone. |
| Company | `company_id` | Shown only to holders of the multi-company group, with the placeholder "Visible to all". |
| The currency's code, then " per ", then the company currency's code | `company_rate` | Twelve digits of which twelve are fractional. In the standalone list, where no currency is in context, the heading reads "Unit per " followed by the company currency's code. |
| The company currency's code, then " per ", then the currency's code | `inverse_company_rate` | Twelve digits of which twelve are fractional. In the standalone list the heading reads the company currency's code followed by " per Unit". |
| Technical Rate | `rate` | Optional column, hidden by default, visible only to holders of the technical features group (`MCUR-032`). |
| Last updated on | The change moment every record carries | Optional column, hidden by default. |

**Guards on editing.** Typing in either rate column triggers the plausibility warning of
`MCUR-021`. Saving runs `MCUR-020`, then `MCUR-022`, then `MCUR-023`.

**Search.** A single field matching the typed term against the rate date after parsing it as a date
in the reader's language and date format, and against the technical rate when it cannot be parsed
as a date (`MCUR-034`).

**Ordering.** Rate date descending, then internal identifier ascending.

## 2.5 The rate form

Left group: the rate date, the technical rate (only for holders of the technical features group),
the company rate and the inverse company rate. Right group: the currency and the company (only for
holders of the multi-company group). The currency is read-only once the row exists.

## 2.6 The accounting settings screen

Four blocks concern this domain; their fields and defaults are in
[configuration.md](configuration.md) sections 1 and 2.

**Currencies block.**

| Control | Behaviour |
|---|---|
| Main Currency selector | Writes the company's main currency. The selector includes archived currencies. Guarded by `MCUR-170` and `MCUR-171`. |
| Currencies | Opens the currency catalogue. |
| Automatic Currency Rates toggle | Shown only to holders of the multi-currency group. Installs or removes the automatic rate retrieval capability. |

**Automatic rate block**, present only when the capability is installed: the service selector, the
interval selector, the next run date and an "Update now" control beside it.

**Default Accounts block, "Exchange difference entries:".** Shown only to holders of the
multi-currency group. Three labelled fields: "Journal", "Gain" and "Loss".

**Customer Invoices block.** The two toggles "Display the total amount of an invoice in letters"
and "Taxes are also displayed in local currency on invoices".

## 2.7 A document form in a foreign currency

| Element | Behaviour |
|---|---|
| Currency selector | Shown beside the journal. Editable only while the document is in draft. Changing it re-derives every line's currency and every line's balance. |
| Currency Rate field | Shown only to holders of the multi-currency group, and only when the document currency differs from the company currency. Editable while the document is in draft. Guarded by `MCUR-080`. |
| Reset rate control | Runs the refresh operation of section 1.4. Shown only when the applied rate differs from the expected rate. |
| Line amounts | The unit price, the subtotal and the total are shown in the document currency. |
| Totals block | Shown in the document currency. When the company setting for taxes in the company currency is on, each tax line additionally shows its company currency value. |
| Inactive currency warning | When the document is in draft and its currency is archived, a warning is shown and posting is refused with the message of `MCUR-016`. |

## 2.8 The journal item list

| Column | Shown when |
|---|---|
| Currency | The multi-currency group is held. |
| Amount in Currency | The multi-currency group is held. |
| Debit, Credit, Balance | Always. Expressed in the company currency. |
| Residual Amount | On reconcilable accounts. Expressed in the company currency. |
| Residual Amount in Currency | On reconcilable accounts, when the multi-currency group is held. |
| Matching number | Always. It shows the Full Reconciliation's identifier, or the letter `P` followed by an identifier for a group that has not closed, or a mark beginning with `I` for an item imported with a matching key. |

A control opens the matched lines of an item. Two variants exist: one showing every matched line,
and one excluding the lines of generated exchange difference entries, so that an accountant sees the
business counterparties rather than the technical corrections.

## 2.9 The bank transaction form

| Field | Behaviour |
|---|---|
| Amount | Expressed in the bank account currency, whose code is displayed beside it. |
| Foreign Currency | Shown only to holders of the multi-currency group. Guarded by `MCUR-150`. |
| Amount in Currency | Shown only when a transacted currency is chosen. Derived on first entry by `MCUR-154` and never overwritten afterwards. Guarded by `MCUR-151` and `MCUR-152`. |

## 2.10 The reconciliation screen

For a transaction carrying a transacted currency, the screen shows both the transacted amount with
its currency and the company currency equivalent, on the transaction line and on every proposed
counterpart. Selecting a counterpart derives its amounts from the transaction's own implied rates
(`MCUR-157`) rather than from the rate table, and the resulting figures are displayed before the
accountant validates.

## 2.11 The payment registration screen

| Field | Behaviour |
|---|---|
| Amount | Expressed in the screen's currency. Derived by [calculations.md](calculations.md) section 22 unless the accountant typed a custom amount. |
| Currency | Editable. Changing it converts a custom amount from the previously selected currency at the payment date. |
| Payment Date | Changing it re-derives the amount, unless a custom amount was typed, in which case the custom amount is kept. |
| Amount to pay in the company currency, and amount to pay in the foreign currency | Informational, showing the residuals of the selected documents in both columns. |

---

# 3. Printed documents and reports

| Output | What this domain contributes |
|---|---|
| Customer invoice, vendor bill, credit note, debit note, receipt | Every monetary figure is rendered with the document currency's symbol, symbol position and decimal place count by the formatting algorithm of [calculations.md](calculations.md) section 24. When the company setting for taxes in the company currency is on, each tax line additionally shows its value in the company currency. When the setting for the total in letters is on, the document total is spelled out below the totals block by section 25. |
| Payment receipt | The payment amount in the payment currency and, when it differs, its company currency equivalent. |
| General ledger, trial balance, balance sheet, profit and loss statement | Expressed in the company currency. When the report spans companies whose main currencies differ, each company's figures are multiplied by the factor the rate table of section 1.5 supplies for that company, that period and the requested rate type. |
| Journal item export | Both amount columns are exported — the company currency balance and the document currency amount — each accompanied by its currency code. |
| Unrealised gain and loss report | One line per account and currency showing the balance in the foreign currency, the book value in the company currency, the value at the reporting date's rate and the adjustment. It offers a manual rate per currency and a control that posts the adjustment entry and its reversal (`MCUR-200`, workflow 19 of [workflows.md](workflows.md)). |

---

# 4. Scheduled jobs

| Job | Frequency | Behaviour |
|---|---|---|
| Automatic currency rate update | Runs on the platform's scheduler; each company is processed when its own next run moment has been reached | Workflow 3 of [workflows.md](workflows.md). It fetches rates for every active currency other than the company's main currency, writes or updates one Currency Rate per currency for today and for the company's root, and advances the company's next run moment by the configured interval. A service failure is logged and leaves the rate table untouched. |

No other scheduled job belongs to this domain. Exchange difference entries are produced
synchronously by a reconciliation and never by a job.

---

# 5. Notifications and messages

| Moment | Kind | Content |
|---|---|---|
| Editing a rate in a form by more than twenty percent | Non-blocking dialogue | Title "Warning for " followed by the currency code; body quoted in `MCUR-021`. |
| Editing a rate in a form, in a company whose fiscal country is Egypt, with more than five significant fractional digits | Non-blocking dialogue | Title "Warning for " followed by the currency code; body quoted in `MCUR-021`. |
| Editing the rounding factor of a saved currency | In-form panel | Quoted in `MCUR-015` and reproduced in section 2.3 above. |
| Lowering an entry of the decimal precision registry | Non-blocking dialogue | Quoted in [calculations.md](calculations.md) section 1. |
| A document becoming paid as a result of a reconciliation | A message in the document's discussion thread | Posted by the paid hook of the reconciliation. This domain contributes the fact that the document closed, not the message text. |
| Any refused write | Blocking message | The exact texts are catalogued in [business-rules.md](business-rules.md). |

Changes to a journal item's balance and to a journal entry's document currency are recorded in the
document's discussion thread, because both fields are change-tracked. Changes to the document rate
are **not** tracked.

---

# 6. Integration contracts

| Contract | Direction | Shape |
|---|---|---|
| Rate service | Inbound | Described in [configuration.md](configuration.md) section 2: given a base currency code, a set of currency codes and a date, the service returns a mapping from currency code to units of that currency per one unit of the base currency. |
| Electronic invoice export | Outbound | The document currency's alphabetic code and, where the format requires it, its numeric code are written into the exported document, together with the document total in that currency and, where the format requires it, the tax amounts in the company currency and the applied rate. Owned by [../electronic-invoicing-and-document-exchange/README.md](../electronic-invoicing-and-document-exchange/README.md); this domain supplies the codes, the rate and the two amount columns. |
| Electronic invoice reception | Inbound | The imported document's currency code is matched against the currency code of the catalogue. When it matches an archived currency, that currency is **not** activated automatically and the created document fails the posting guard of `MCUR-016` until an administrator activates it. The applied rate is taken from the imported document when it states one, and from the rate table otherwise. |
| Payment file export | Outbound | The payment currency's alphabetic code accompanies every amount. |
| Spreadsheet | Outbound | The three operations of section 1.6. |
| Point of sale session | Outbound | The two operations of section 1.6 that load the currencies and their fields into a session. |

---

# 7. Reconciliation notes

1. **Operation names.** One draft gave every operation a coined identifier in code font. A coined
   identifier is not a reproduced one, and code font in this repository means "reproduced exactly";
   the operations are therefore named in words here, and every input, output, side effect and error
   path that draft specified is kept unchanged.
2. **The rate list columns.** Both drafts described the two rate columns. The generated headings,
   the twelve fractional digits, the position of a new row — bottom in the standalone list, top in
   the currency form's tab — and the twenty-five row limit of the tab are stated here once, in
   sections 2.3 and 2.4.
3. **Section references.** Every reference into [calculations.md](calculations.md) follows the
   consolidated numbering of that file.
4. **The matching number column.** One draft described only the two values `P` and the full
   identifier. The column can also show a mark beginning with `I`, for an item imported with a
   matching key whose real matching is deferred; see [state-machines.md](state-machines.md)
   section 8.
5. **The link to the exchange domain.** One draft linked the electronic invoice contracts to a
   folder key that this repository does not use. Section 6 links to
   [../electronic-invoicing-and-document-exchange/README.md](../electronic-invoicing-and-document-exchange/README.md).
