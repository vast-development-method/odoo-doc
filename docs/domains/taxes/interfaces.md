# Taxes — Interfaces

Everything the tax domain exposes: navigation, screens and what each shows, the operations another
component or an external caller may invoke, the routes, the printable output, the data contracts
consumed by the client-side engine, and the import and export shapes.

---

## 1. Navigation

All tax configuration lives under the accounting application's **Configuration** menu, itself
restricted to the accounting administrator group.

| Menu path | Opens | Visible to |
|---|---|---|
| Configuration → Accounting → Taxes | The tax list | accounting administrator |
| Configuration → Accounting → Fiscal Positions | The fiscal position list | accounting administrator |
| Configuration → Accounting → Tax Groups | The tax group list | developer mode only |
| Configuration → Accounting → Cash Rounding | The cash rounding list | the cash rounding group |
| Configuration → Settings | The accounting settings, including every company setting of `configuration.md` section 1 | the settings administrator group |
| Reporting → Taxes & Fiscal | The container under which tax returns appear | the read-only accounting group or the invoicing group |

Account tags have no menu of their own; they are reached from the tax form, from an account, or
from a product.

---

## 2. Screens

### 2.1 The tax list

Columns: name, tax type, tax scope, amount, active. The list is the default view of the tax window
action, which opens with two filters pre-applied — sales and purchases — and with archived records
**included**, so an administrator sees the whole picture at once.

### 2.2 The tax search panel

| Element | Content |
|---|---|
| Search fields | name, description, amount, company (multi-company installations only), fiscal positions |
| Filters | Sale; Purchase; Services; Goods; Domestic; Active; Inactive |
| Groupings | Company (multi-company only); Tax Type; Tax Scope; Fiscal Position |

A second, lighter search panel used inside other screens searches the name, the description and
the invoice label at once, plus the company.

**The free-text grammar.** Typing a compact code into the name search expands it into a wildcard
pattern as described in `entities.md` section 1.6, so `21M` finds "21% M", "21% EU M", "21% M.
Cocont" and "21% EX M".

### 2.3 The tax card view

Shows the name as a heading, the tax type and the tax scope as pills, and the description below.

### 2.4 The tax form

**Header block**

| Left | Right |
|---|---|
| Name; Tax Computation; Active (as a toggle) | Tax Type; Tax Scope; Amount (hidden unless the computation is fixed, percentage or division, with a percent sign shown except for a fixed tax); Fiscal Positions (as tags, placeholder "all"); Replaces (as tags, shown only when the tax replaces something or belongs to a non-domestic fiscal position; no creation from the field) |

**Page "Definition"** — hidden for a Group of Taxes:

- Group *Distribution for Invoices*: the invoice distribution as an editable list showing the
  percentage (with a dedicated percentage editor), the line kind, the account and the tags.
- Group *Distribution for Refunds*: the same for the refund distribution.
- For a Group of Taxes instead: the children as an ordered list showing a drag handle, the name,
  the computation kind and the amount. The children chooser is restricted to taxes whose tax type
  is `none` or the group's, and whose computation kind is not "group of taxes". The list is hidden
  when the group's own tax type is `none`.

**Page "Advanced Options"**

| Left | Right |
|---|---|
| Label on Invoices; Description; Tax Group (hidden and not required for a group); Include in Analytic Cost (analytic group only); Company (multi-company only); Country (required); Legal Notes | Included in Price (hidden for a group and for a reverse-charge tax; placeholder "Default"); Affect Base of Subsequent Taxes (hidden for a group); Base Affected by Previous Taxes (developer mode only, hidden for a group and for a price-included tax); Tax Exigibility as radio buttons (hidden for a group and when the company does not use cash basis); Cash Basis Transition Account (shown and required only when the exigibility is "based on payment") |

The form carries a message thread, which is where the tracked changes and the readable
distribution difference appear.

### 2.5 The fiscal position form

**Banner.** When the fiscal position carries a foreign registration number, has a country, and no
tax exists yet for that country, a banner invites the administrator to create that country's taxes
with a single click.

**Button box.** One button, *Taxes*, opening the taxes of the same company that either belong to
this fiscal position or belong to no fiscal position at all, with archived records included.

**Ribbon.** *Archived* when the active flag is clear.

**Fields**

| Left | Right |
|---|---|
| Fiscal Position (name); Company (multi-company only) | Detect Automatically; Tax registration required (only when detection is on); Foreign Tax Identification Number; Country Group (only when detection is on); Country (required when a foreign number is present); Federal States as tags restricted to the chosen country (only when detection is on or a foreign number is present, and the country has states); Zip Range, shown as *From* and *To* (only when detection is on and a country is chosen) |

**Page "Account Mapping"** — visible to the read-only accounting group and above. An editable list
of source account and destination account, each restricted to the fiscal position's company and
excluding off-balance accounts.

**Footer.** The legal notes, edited in place, with the placeholder *"Legal Notes…"*.

**Search panel.** Search on the name; filters *Archived* and *Domestic*.

### 2.6 The tax group form and list

List: name, sequence, country. Form: name, sequence, country, the three settlement accounts, the
preceding subtotal label and the point-of-sale label. Search panel: a country filter.

### 2.7 Tax fields on other screens

| Screen | Tax elements |
|---|---|
| Product form | Sales Taxes and Purchase Taxes as tag fields, each restricted to the matching tax type; Account Tags restricted to product tags; the price hint next to the sales price |
| Account form | Default Taxes as a tag field, showing the tax type and the company on each tag; Tags |
| Partner form | Tax Identification Number; Intra-Community Valid (read-only indicator); Fiscal Position under the accounting tab |
| Invoice and bill line | Taxes as a tag field, with archived taxes visible in the list and taxes replaced by the document's fiscal position hidden |
| Invoice and bill header | Fiscal Position |
| Journal item list | Taxes, Originator Tax, Originator Tax Distribution Line, Tags, Base Amount |
| Payment form | Withhold Tax Amounts and the withholding table, shown only when the company owns a matching withholding tax |
| Register payment wizard | The same, plus the Net Amount and the Outstanding Account |

### 2.8 The withholding table

Columns: Tax, Sequence Number (with the hint as a placeholder), Account (hidden when the company
has a default withholding base account), Analytic Distribution, Withholding base, Withholding
amount. Below the table: the **Net Amount**, read-only.

---

## 3. Operations another component may invoke

These are the contracts the rest of the system relies on. Each is described by what it takes and
what it returns; the algorithms are in `calculations.md`.

### 3.1 The engine

| Operation | Input | Output |
|---|---|---|
| Prepare a base line | a record or a set of values, plus overrides | a base line (`calculations.md` section 1.1) |
| Prepare a tax line | a record or a set of values, plus overrides | a tax line (section 1.4) |
| Add tax details to base lines | a list of base lines, a company, optionally a rounding method | each base line gains an unrounded tax details block |
| Round the tax details of base lines | a list of base lines, a company, optionally a list of existing tax lines | every amount is rounded and the deltas are set |
| Add accounting data | a list of base lines, a company, a flag asking whether cash basis tags count | each tax result gains its distribution data, accounts, tags and grouping key; each base line gains its tag set |
| Produce the tax entries | a list of base lines, a company, optionally a list of existing tax lines | four lists: entries to add, entries to delete, entries to update with their new amounts, and base lines to update with their new tags and amounts |
| Aggregate the tax details of one base line | a base line and a grouping function | a table from grouping key to the twenty-four amounts of section 7 |
| Aggregate across base lines | the previous result | one table for the whole document |
| Produce the totals block | a list of base lines, a currency, a company, optionally a cash rounding configuration | the structured totals of section 10 |
| Exclude tax groups from a totals block | a totals block and a list of tax group identifiers | a new totals block with those groups folded into the base |
| Map taxes through a fiscal position | a fiscal position and a set of taxes | the replaced set |
| Map an account through a fiscal position | a fiscal position and an account | the replacement or the original |
| Detect a fiscal position | a partner and an optional delivery address | a fiscal position or nothing |
| Adapt a unit price to new taxes | a price, a product, the original taxes, the new taxes | the adapted price |
| Split a base line | a base line, a company, a list of weights | that many base lines whose amounts add back exactly |
| Merge two tax details blocks | two blocks | one block |
| Reduce base lines to a target amount | base lines, a company, an amount kind and an amount | new base lines whose totals hit the target exactly |
| Prepare global discount lines | base lines, a company, an amount kind and an amount | negative base lines carrying the computation key `global_discount` |
| Prepare down payment lines | the same with a positive amount | base lines carrying the computation key `down_payment` |
| Flatten a set of taxes | a set of taxes | the ordered list of non-group taxes |
| Get the tags of a tax | a tax, a refund flag and a line kind | the tag set |
| The legacy all-in-one computation | a price, a currency, a quantity, a product, a partner, a refund flag, a price-inclusion flag, a cash-basis-tags flag and a rounding method | the untaxed total, the total with taxes, the total posted to no account, the base tags and one entry per produced tax result with its identifier, name, amount, base, sequence, account, analytic flag, settlement flag, reverse-charge flag, price-inclusion flag, exigibility, distribution line, group, tags and downstream taxes |

**The legacy all-in-one computation** deserves a note: it rounds the untaxed and included totals to
the currency unless the caller says otherwise, it reports the **raw** per-result base, and it adds
a "total posted to no account" figure which is the untaxed total plus the amounts of every
distribution line that has no account. It accepts a context marker forcing every tax to be treated
as price-included or price-excluded.

### 3.2 Tax identification numbers

| Operation | Input | Output |
|---|---|---|
| Run the checks | a country, a number, a record label, a validation mode | the normalised number and the country code it was validated for, or an error |
| Normalise a number | a country code and a number | the normalised number |
| Check a number | a country code and a normalised number | true or false |
| Build the failure message | a country code, the number and a record label | the message text |
| Is the registration valid for a company | a partner and a company | true or false, including the cross-border flag when it applies |
| Convert a national company number to its union form | the national number | the union form or nothing |

### 3.3 Cash basis

| Operation | Input | Output |
|---|---|---|
| Collect the cash basis data of a document | a document | the lines to process with their treatment, the four totals, the currency and the fully-paid flag; or nothing |
| Collect it for a set of partial reconciliations | the partials | the same per document, plus one entry per partial with its percentage, its payment rate, its settlement date, its counterpart document and whether both sides are posted |
| Create the cash basis entries | the partials | the created journal entries |

### 3.4 Withholding

| Operation | Input | Output |
|---|---|---|
| Derive the withholding lines | base lines and a company | the list of changes to apply to the existing lines |
| Build the journal items | the withholding lines | the list of journal items to create |
| The domain of available withholding taxes | a company and a payment direction | a filter |

### 3.5 Client-side data contract

The client-side copy of the engine needs data the browser cannot fetch on demand. Two operations
declare it:

| Operation | Purpose |
|---|---|
| Which product fields the taxes read | Returns the set of product field names every custom-formula tax in the set reads, so the client can preload them. |
| Which unit-of-measure fields the taxes read | The same for the unit of measure. |
| Default values for those fields | Returns, per field name, its type and the default value to use when there is no product at all: zero for an integer, a decimal or a monetary field. |
| Turn a product into values | Returns a flat table of those field values for a given product, reading them with elevated rights because a tax may depend on a restricted field. |

The client must also receive, per tax: the amount, the computation kind, the sequence, the
price-inclusion flag, the "affect base" and "base affected" flags, the children of a group, the
"has a negative distribution factor" flag, the tax group with its sequence and its receipt label,
and — for a custom-formula tax — the normalised formula.

---

## 4. Routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/base_vat/1/webhook_update_vies` | form-encoded request, cross-site protection disabled, no session saved | public | The callback the cross-border verification relay calls when a pending number resolves. Takes a signed callback token and a status. The token is verified against the fixed purpose text `vies_check`; when it does not verify, a warning is logged and nothing changes. When it verifies, the token yields the number, every partner carrying that number is found with elevated rights and the status is applied to all of them. |

The domain exposes no other route. The relay itself is called **out** at two addresses on the
configured endpoint: one to request a validity check and one to collect updates; both are
form-encoded requests carrying the client credentials and the installation identifier, with
timeouts of twenty and ten seconds respectively.

---

## 5. Printable documents

The tax domain does not own a printable document of its own; it contributes to the documents of
other domains.

### 5.1 The customer or vendor document

| Element | Content |
|---|---|
| The tax column of a line | The comma-separated **tax labels** of the line's taxes, skipping any tax whose label is empty. The label is the invoice label when set, otherwise the tax's name — except for a withholding tax, whose label never falls back to the name, so an unlabelled withholding tax prints nothing. |
| The line subtotal column | The untaxed subtotal when the company's default is "tax excluded", and the total with taxes when it is "tax included". |
| The totals block | One row per subtotal that has tax groups, then one row per tax group. When every tax group shares the same base, or when the group has no base to display, the row shows only the group name and the tax amount; otherwise it shows the group name, the base amount to display and the tax amount. |
| The cash rounding row | Shown when the totals block carries a cash rounding amount. |
| The grand total row | The total amount in document currency. |
| The company-currency block | A second, compact totals block in the company currency, shown when the company setting is on, the two currencies differ, at least one tax group is involved and the document is a sales document. |
| The fiscal position note | The fiscal position's legal note, printed after the totals when it is not empty. |
| The tax legal notes | The concatenation, in encounter order and without duplicates, of the legal notes of every tax used on the document, skipping empty ones. |

### 5.2 The payment receipt

*Requires the withholding capability.* When the payment carries withholding lines, a table is
printed before the ordinary content with four columns: Tax, Withholding number, Base, Amount. The
base and the amount are printed as absolute values in the payment's currency.

---

## 6. Notifications and message history

The tax domain produces no electronic mail. It writes to the message history of two entities.

| Entity | What is written |
|---|---|
| Tax | The tracked-field changes: name, tax type, computation kind, amount, price-inclusion override, "affect base of subsequent taxes", "base affected by previous taxes". Plus, when the tax is in use, a readable difference of the distribution: per line, renumbered within its document kind, an old-value/new-value pair for each of the percentage, the account, the list of tags and the settlement flag; added lines are shown with the word *New*, removed lines with the word *Removed*. |
| Partner | One message per cross-border verification outcome, with the four texts of `state-machines.md` section 5.2. |

Creating a tax deliberately writes **no** "created" message and deliberately does **not** subscribe
the creator to the record's thread.

---

## 7. External service integration

| Service | Direction | Contract |
|---|---|---|
| The cross-border verification relay | outbound and inbound | Outbound: a request for a validity check carrying the number, the installation identifier, the client identifier, the client token, a callback address and a signed callback token valid for seven days, timing out after twenty seconds; and a request for updates carrying only the identifiers, timing out after ten seconds. Inbound: the callback of section 4. A transport failure or a response without a status is treated as the status *fault*. |

---

## 8. Import and export

### 8.1 Importing a tax from an electronic document

When a received electronic document names a tax by its characteristics rather than by identifier,
the tax is resolved by a **search plan**: an ordered list of strategies, each producing an ordered
list of criteria. The first criterion that finds a tax wins, and the whole plan stops.

Every criterion is combined with a mandatory base filter built from the values read off the
document:

```
computation kind equals the read kind
and tax type equals the read type
and amount equals the read amount
and, when the read values came from an invoice being predicted, country equals that invoice's tax country
and, when a name was read, name equals it
and, when an exigibility was read, exigibility equals it
and, when a document-format tax category code was read and the field exists,
    that code is the read one or empty
```

The search is ordered by sequence then identifier, with the category code first when it took part.
Results are cached per company and per criterion for the duration of the transaction, including
negative results.

The three shipped strategies, in the order they are offered:

| Strategy | Criteria it produces |
|---|---|
| From the account's default taxes | One criterion: the tax must be among the read account's default taxes. Produces nothing when there is no account or the account proposes none. |
| From prediction on a bill | One criterion using the prediction service of the invoicing component, keyed on the invoice, the line label, the partner, the read computation kind, the read amount and the read tax type. Produces nothing when the prediction capability is absent. |
| From the price-inclusion flag | Up to four criteria, tried in order. When the read flag says "not included": price-excluded taxes of the read fiscal position, then price-excluded taxes regardless of fiscal position. When the read flag says "included" or is unknown: price-included taxes of the read fiscal position, then price-included taxes regardless. When the fiscal position is the domestic one, "of the read fiscal position" also admits taxes attached to no fiscal position. |

### 8.2 Importing a partner by its tax identification number

The partner-matching plan used when a received document is decoded tries the number first. It
builds the candidate spellings of the number: the number as read, the number prefixed with the
country code, and country-specific variants — for the Swiss number, the normalised number followed
by each of the three tax-regime suffixes.

### 8.3 Exporting the tax breakdown

The aggregation operations of section 3.1 are the export contract: a caller supplies a grouping
function and receives, per group, twenty-four amounts — base, tax and untaxed total, each in raw,
rounded and target form, each in both currencies. Structured invoice formats use the **raw**
amounts for per-line figures, because those formats carry more decimals than a currency does, and
the **rounded** amounts for document-level figures.

### 8.4 Stored extra tax data

The structured document stored on a journal item is itself an interface: it is what makes a
manually adjusted tax survive a save, a reload and a reversal. Its shape and its reload conditions
are in `calculations.md` section 9.3.

---

## 9. The client-side mirror

The engine exists twice: once on the server and once in the browser and the point-of-sale client.
The two copies must produce identical numbers for identical inputs, because a total computed while
a document is being edited must not change when the document is saved. This section states the
contract between them.

### 9.1 Which steps are mirrored

| Step | Mirrored | Why |
|---|---|---|
| Preparing a base line from a record or a set of values | yes | the client edits lines |
| Flattening and sorting | yes | |
| Batching | yes | |
| Every amount formula, including the custom formula | yes | |
| The extra-base propagation table | yes | |
| The single-line computation | yes | |
| The dual-currency conversion | yes | |
| The smooth delta distribution | yes | |
| The three document-wide rounding passes | yes | |
| Aggregating tax details | yes | |
| The totals block, except the non-deductible part | yes | the client shows the totals |
| The non-deductible part of the totals block | **no** | vendor bills are not edited offline |
| Exporting and importing the stored extra tax data | yes | the client must not lose a manual amount |
| Reversing the stored extra tax data by the quantity | **no** | reversal is a server operation |
| Turning a refund base line back into a normal one | **no** | server-only helper |
| Preparing a tax line from an existing record | **no** | the client has no journal items |
| The accounting derivation and the production of tax entries | **no** | the client posts nothing |
| The realignment on existing tax entries | **no** | |
| The cash basis mechanism | **no** | |
| The withholding mechanism | **no** | it runs at payment time |
| The fiscal position lookup tables | computed on the server, **used** by the client | |
| The unit price adaptation | yes | the client applies fiscal positions while editing |
| Splitting, merging, reducing to a target amount, dispatching | yes | discounts and down payments are edited on the client |
| The per-country number checks | **no** | validation happens on save |

### 9.2 What the client must be given

Per tax, at the moment the client may need it:

| Datum | Used for |
|---|---|
| identifier | grouping and the manual-amount table |
| amount | every formula |
| computation kind | the formula choice and the batching test |
| sequence | the ordering |
| the effective price-inclusion flag | the batching test and the evaluation pass |
| "affect base of subsequent taxes" | batching and propagation |
| "base affected by previous taxes" | batching and propagation |
| the children, for a Group of Taxes | flattening |
| "has a negative distribution factor" | the reverse-charge mirror record |
| the tax group's identifier, sequence, name, preceding subtotal and receipt label | the totals block |
| the normalised formula, for a custom-formula tax | evaluation |
| the product field names and the unit-of-measure field names the formulas read | preloading |

Per currency: its rounding step and its number of decimal places, for both the document currency
and the company currency.

Per product that may appear on a line: the values of the field names above, read with elevated
rights.

### 9.3 What the client must never assume

- That a tax it does not know about is absent. A filter may have removed a tax from the evaluation
  while keeping it on the line.
- That the untaxed total equals the raw base. It equals the first result's base.
- That a line's balance equals its rounded untaxed total. It equals that total plus the delta.
- That rounding one line at a time gives the document total. It does not, under "round per tax".

---

## 10. Ordering constraints on the engine's operations

Calling the operations of section 3.1 out of order produces wrong numbers rather than an error. The
constraints are:

```
prepare the base lines
  → add the tax details to ALL of them
    → round the tax details of ALL of them together
      → (optionally) add the accounting data
        → produce the tax entries
      → (optionally) aggregate
      → (optionally) produce the totals block
```

| Constraint | Consequence of breaking it |
|---|---|
| Add the tax details to every base line before rounding any of them | the redistribution has nothing to redistribute over |
| Round all the base lines in one call | the "round per tax" method degenerates to "round per line" |
| Add the accounting data only after rounding | the distribution shares are computed from unrounded amounts |
| Produce the tax entries only after the accounting data | there are no grouping keys yet |
| Produce the totals block only after rounding | the block shows unrounded figures |
| Pass the existing tax entries to the rounding call, not to the production call alone | manually typed tax amounts are lost |

---

## 11. The shape of the aggregated result

Every aggregation returns, per grouping key, twenty-four monetary values plus two bookkeeping
entries. They are listed here in full because structured document formats consume them directly.

| Name | Meaning |
|---|---|
| `base_amount_currency` / `base_amount` | the rounded base of this key, in the document and the company currency |
| `raw_base_amount_currency` / `raw_base_amount` | the same before rounding |
| `target_base_amount_currency` / `target_base_amount` | the same considering the manual amounts |
| `tax_amount_currency` / `tax_amount` | the rounded tax of this key |
| `raw_tax_amount_currency` / `raw_tax_amount` | the same before rounding |
| `target_tax_amount_currency` / `target_tax_amount` | the same considering the manual amounts |
| `total_excluded_currency` / `total_excluded` | the untaxed total of the base lines contributing to this key, including their deltas |
| `raw_total_excluded_currency` / `raw_total_excluded` | the same before rounding |
| `target_total_excluded_currency` / `target_total_excluded` | the same considering the manual amounts |
| `taxes_data` (per base line) | the subset of that line's tax results aggregated under this key |
| `base_line_x_taxes_data` (across base lines) | one pair per contributing base line: the line and its subset |

**The propagation rule inside an aggregation.** When a tax *A* affects the base of a tax *B*, the
untaxed total reported under *B*'s key is increased by *A*'s tax amount, in both the raw and the
rounded form. This is what makes an aggregation by tax report, for *B*, a base that already
contains *A*.

**The grouping function must never return nothing for a line that has taxes.** Returning nothing is
reserved for the "no tax at all" case, which the aggregation signals by calling the function once
with an empty tax result.
