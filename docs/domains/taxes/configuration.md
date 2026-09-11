# Taxes — Configuration

Everything an administrator sets up before the domain can work: company settings, system
parameters, sequences, shipped default records, security groups, the access rights matrix, record
rules and scheduled jobs.

---

## 1. Company settings

These live on the Company entity and are exposed through the accounting settings screen. A setting
shown in the screen is a mirror of the company field; changing it changes the company.

### 1.1 Taxes section

| Setting | Company field | Type and values | Default | Effect |
|---|---|---|---|---|
| Default Sales Tax | `account_sale_tax_id` | link to one Tax | none | Proposed as the sales tax of a newly created product. |
| Default Purchase Tax | `account_purchase_tax_id` | link to one Tax | none | Proposed as the purchase tax of a newly created product. |
| Default Sales Price Include | `account_price_include` | `tax_included` or `tax_excluded` | `tax_excluded` | The fallback price-inclusion for every tax that does not override it. **Changing it changes the meaning of every stored price** on every tax that has no override. |
| Rounding Method | `tax_calculation_rounding_method` | `round_globally` (shown as "Round per Tax") or `round_per_line` (shown as "Round per Line") | `round_globally` | Governs the whole engine; see `calculations.md` sections 6 and 7. |
| Fiscal Country | `account_fiscal_country_id` | link to one Country | the company's country | The country whose tax reports the company files. Determines which taxes and which report tags are offered by default. |
| Taxes in company currency | `display_invoice_tax_company_currency` | boolean | true | Shows the tax totals of a customer document in the company currency as well, when the two differ and at least one tax group is involved. |
| Default Purchase Receipt Fiscal Position | `account_purchase_receipt_fiscal_position_id` | link to one Fiscal Position | none | Applied to a newly created purchase receipt. |

### 1.2 Cash basis section

| Setting | Company field | Type | Default | Effect |
|---|---|---|---|---|
| Cash Basis | `tax_exigibility` | boolean | false | Master switch. When off, the exigibility field is hidden on every tax of the company. Turning it off is refused while any tax of the company is exigible on payment. |
| Tax Cash Basis Journal | `tax_cash_basis_journal_id` | link to one Journal | none | The journal in which cash basis entries are created. Mandatory in practice: the first entry that needs it fails without it. |
| Base Tax Received Account | `account_cash_basis_base_account_id` | link to one Account | none | The account used for the pair of base items of a cash basis entry. When empty the original base line's own account is used. |

### 1.3 Withholding section

*Requires the withholding capability.*

| Setting | Company field | Type | Default | Effect |
|---|---|---|---|---|
| Withholding Tax Base | `withholding_tax_base_account_id` | link to one Account | none | The default account of a withholding line. When set, the account column is hidden on the withholding tables. |

### 1.4 Tax identification number section

*Requires the number validation capability.*

| Setting | Company field | Type | Default | Effect |
|---|---|---|---|---|
| Verify Tax Numbers | `vat_check_vies` | boolean | false | Turns on the cross-border verification of `calculations.md` section 15.5 for every partner. |

### 1.5 Locking

| Setting | Company field | Type | Effect |
|---|---|---|---|
| Tax Lock Date | `tax_lock_date` | date | Any change to a posted entry that affects the tax report and is dated at or before this date is refused; see `business-rules.md` section 6. |

### 1.6 Derived company fields

| Field | How it is derived |
|---|---|
| Domestic fiscal position (`domestic_fiscal_position_id`) | `entities.md` section 4.6. |
| Foreign registration countries (`multi_vat_foreign_country_ids`) | Every country for which a fiscal position of the company carries a foreign registration number. |
| Tax-enabled countries (`account_enabled_tax_country_ids`) | The fiscal country plus the foreign registration countries. Empty for a user with no access to the company. |
| Fiscal country group codes (`account_fiscal_country_group_codes`) | The codes of the country groups the fiscal country belongs to; a list with one empty string when there is no fiscal country. |

### 1.7 Actions reachable from the settings screen

| Action | Effect |
|---|---|
| Map the union-wide distance-selling taxes | Installs the corresponding localization capability if needed, then runs the mapping on every active company. The mapping itself belongs to `../fiscal-localizations/`. |

---

## 2. System parameters

| Parameter | Purpose | Allowed values |
|---|---|---|
| `iap_vies.endpoint` | The address of the cross-border verification relay. | Exactly one of the production address or the test address; any other value raises `Invalid IAP VIES endpoint`. The default is the test address when the validation capability was installed with demonstration data, and the production address otherwise. |
| `iap_vies.client_identifier` | The identifier this installation presents to the relay. | A universally unique identifier, generated on first use in its own transaction. |
| `iap_vies.client_token` | The secret this installation presents to the relay. | A random token, generated with the identifier. |
| `database.uuid` | The installation's own identifier, sent with every relay request. | Owned by the platform, not by this domain. |

When the periodic verification job does not exist, or the run is a test run, a fixed placeholder
pair is used instead of the stored credentials and the relay ignores it.

---

## 3. Sequences

The tax domain owns no numbering of its own except one optional sequence per withholding tax.

| Sequence | Owner | Purpose | Format |
|---|---|---|---|
| Withholding Sequence | a Tax whose "withhold on payment" flag is set | Numbers the withholding certificates. | Whatever the administrator configures: a prefix, a suffix, a padding and an increment. A typical configuration is a gapless implementation with a padding of four and an increment of one. |

**Consumption rule.** A value is drawn only when the payment's journal entry is built, and only
after **every** withholding line of that payment has been verified to have either a number or a
sequence. A failed attempt therefore never burns a number. The hint shown while editing uses the
sequence's *next* value offset by the line's rank among the lines sharing that sequence, without
consuming anything.

---

## 4. Shipped default records

### 4.1 Account tags shipped by the accounting capability

| Tag | Applicability | Note |
|---|---|---|
| Operating Activities | accounts | May never be deleted. |
| Financing Activities | accounts | May never be deleted. |
| Investing & Extraordinary Activities | accounts | May never be deleted. |

Attempting to delete one of these raises
*"You cannot delete this account tag (&lt;tag name&gt;), it is used on the chart of account definition."*

Every **tax** grid is created by a report expression, never shipped directly.

### 4.2 The generic chart of accounts template

The only chart template shipped by the accounting capability itself. Country-specific templates
belong to `../fiscal-localizations/`.

**Tax groups**

| Identifier | Name | Country | Tax payable account | Tax receivable account |
|---|---|---|---|---|
| `tax_group_15` | Tax 15% | the template's country | `tax_payable` | `tax_receivable` |
| `tax_group_0` | Tax 0% | the template's country | `tax_payable` | `tax_receivable` |

**Fiscal positions**

| Identifier | Name | Country | Detect automatically | Sequence |
|---|---|---|---|---|
| `template_generic_domestic_fiscal_position` | Domestic | the template's country | yes | 10 |
| `template_generic_export_fiscal_position` | Foreign Trade | none | yes | 20 |

**Taxes**

| Identifier | Name | Amount | Tax type | Tax group | Fiscal position | Replaces | Invoice distribution | Refund distribution |
|---|---|---|---|---|---|---|---|---|
| `sale_tax_template` | 15% | 15 | sale | Tax 15% | Domestic | — | base 100%, tax 100% to `tax_received` | the same |
| `purchase_tax_template` | 15% | 15 | purchase | Tax 15% | Domestic | — | base 100%, tax 100% to `tax_paid` | the same |
| `sale_export_tax_template` | 0% Exports | 0 | sale | Tax 0% | Foreign Trade | `sale_tax_template` | base 100%, tax 100% with **no account** | the same |
| `purchase_import_tax_template` | 0% Imports | 0 | purchase | Tax 0% | Foreign Trade | `purchase_tax_template` | base 100%, tax 100% with **no account** | the same |

This is the minimal complete illustration of the substitution mechanism: the domestic taxes belong
to the Domestic fiscal position, the zero-rated taxes belong to the Foreign Trade fiscal position
and each declares the domestic tax it replaces, so a counterpart in another country is invoiced at
zero percent.

### 4.3 Shipped report definitions of the tax kind

| Report | Columns | Distinguishing settings |
|---|---|---|
| Generic Tax report | *Net* (label `net`, monetary) and *Tax* (label `tax`, monetary) | Multi-company filter uses tax units; foreign registrations are allowed; the default opening period filter is "previous return period"; only tax-exigible lines are included. |
| Group by: Account then Tax | the same two columns | A variant of the generic report; always available. |
| Group by: Tax then Account | the same two columns | A variant of the generic report; always available. |

The three shipped report definitions carry **no lines and no expressions**: they are grouping
skeletons whose content is produced by the reporting component. Country-specific tax returns, with
their lines and their tax-tags expressions, belong to `../fiscal-localizations/`.

**Report settings that concern taxes**

| Setting | Values | Meaning |
|---|---|---|
| Availability (`availability_condition`) | `country` ("Country Matches"), `coa` ("Chart of Accounts Matches"), `always` ("Always") | Defaults to `country` when the report has a country, `always` otherwise. |
| Only Tax Exigible Lines (`only_tax_exigible`) | boolean | Restricts the report to journal items whose taxes are already exigible: items of an invoice-exigible tax, and items of a cash basis entry. |
| Allow Foreign Tax Registration (`allow_foreign_vat`) | boolean | Lets the report be opened for a country in which the company holds a foreign registration rather than only for its fiscal country. |
| Multi-company filter (`filter_multi_company`) | `selector` ("Use Company Selector") or `tax_units` ("Use Tax Units") | A tax report normally groups several companies into one legal filing. **The tax-unit grouping itself is not implemented by the packages covered here**; only the selection value exists. |
| Default opening period (`default_opening_date_filter`) | includes `previous_return_period` | Opens the report on the period that follows the last closed return. |

---

## 5. Security groups and the access rights matrix

### 5.1 Groups

| Group | Meaning for this domain |
|---|---|
| Any internal user | May read taxes, tax groups, distribution lines and fiscal positions. |
| Show Accounting Features — Readonly | May read taxes, tax groups, distribution lines and account tags. |
| Invoicing | May read taxes, tax groups, distribution lines and account tags; may use them on documents. |
| Show Full Accounting Features | May create, change and delete account tags. |
| Administrator (accounting) | May create, change and delete taxes, tax groups, distribution lines, fiscal positions and account mappings; may reach the configuration menus. |
| Allow the cash rounding management | Reveals the cash rounding menu and the cash rounding field on documents. |
| Partial Purchase Deductibility | Reveals the deductibility column on vendor documents. |
| Analytic Accounting | Reveals the analytic flag on a tax. |
| Developer mode | Reveals the "base affected by previous taxes" field and the tax group menu. |
| Settings administrator | May change the company's tax settings. |

### 5.2 Access rights matrix

Columns are create, read, update, delete.

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Tax | any internal user | | yes | | |
| Tax | Show Accounting Features — Readonly | | yes | | |
| Tax | Invoicing | | yes | | |
| Tax | Administrator | yes | yes | yes | yes |
| Tax Distribution Line | any internal user | | yes | | |
| Tax Distribution Line | Show Accounting Features — Readonly | | yes | | |
| Tax Distribution Line | Invoicing | | yes | | |
| Tax Distribution Line | Administrator | yes | yes | yes | yes |
| Tax Group | any internal user | | yes | | |
| Tax Group | Show Accounting Features — Readonly | | yes | | |
| Tax Group | Invoicing | | yes | | |
| Tax Group | Administrator | yes | yes | yes | yes |
| Fiscal Position | any internal user | | yes | | |
| Fiscal Position | Administrator | yes | yes | yes | yes |
| Fiscal Position Account Mapping | any internal user | | yes | | |
| Fiscal Position Account Mapping | Administrator | yes | yes | yes | yes |
| Account Tag | Show Full Accounting Features | yes | yes | yes | yes |
| Account Tag | Show Accounting Features — Readonly | | yes | | |
| Account Tag | Invoicing | | yes | | |
| Payment Withholding Line | the same groups as the Payment entity | | | | |
| Payment Register Withholding Line | the same groups as the Register Payment wizard | | | | |

### 5.3 Record rules

| Entity | Rule | Filter |
|---|---|---|
| Tax | Tax multi-company | the record's company is the acting company or one of its ancestors |
| Tax Group | Tax group multi-company | the same |
| Tax Distribution Line | Tax Repartition multi-company | the record has no company **or** its company is the acting company or one of its ancestors |
| Fiscal Position | Account fiscal Mapping company rule | the record's company is the acting company or one of its ancestors |

The "or one of its ancestors" shape is what makes a parent company's taxes usable by its branches
while keeping a branch's own taxes invisible to the parent.

Account Tags carry **no** company rule: a tag belongs to a country, not to a company.

---

## 6. Scheduled jobs

| Job | Owner | Frequency | What it does |
|---|---|---|---|
| Synchronise cross-border verification updates | the number validation capability | every day, run as the system user | Asks the relay for updates on numbers previously reported as pending, receives a table of number to status, groups the partners by number and applies each status, logging a message per partner. |

The tax domain declares no other job. The periodic tax return is **not** generated by the packages
covered here; see `accounting-effects.md` section 8.

---

## 7. Onboarding

The accounting onboarding offers one tax-related step: choosing the company's default sales tax.
Saving it writes the company's default sales tax and marks the step done. The step belongs to the
general ledger's onboarding sequence; only the field it writes belongs here.

---

## 8. Capability matrix

| Capability | Adds |
|---|---|
| Accounting (core) | Taxes, distribution lines, tax groups, fiscal positions, account tags, the engine, cash basis, the report skeleton, the tax lock date. |
| Custom Formula Taxes | The `code` computation kind, the formula field and its grammar. |
| Point of Sale companion to Custom Formula Taxes | Makes the same formula evaluable on the point-of-sale client. |
| Withholding Tax on Payment | The "withhold on payment" flag, the withholding sequence, the withholding lines on payments and on the register-payment wizard, the company's withholding base account. |
| Point of Sale companion to Withholding | Makes withholding usable from the point of sale. |
| Tax Number Validation | The per-country checks, the normalisers, the cross-border verification switch, the relay call, the callback route and the daily job. |
| Update Tax Grids | The maintenance operation that re-derives the report tags of existing journal items. |

---

## 9. Precisions and units

| Quantity | Precision |
|---|---|
| A tax's amount | four decimal places on a sixteen-digit number |
| A distribution line's percentage | twelve decimal places on a sixteen-digit number |
| Every monetary amount | the decimal places of the currency concerned; the document currency and the company currency are rounded independently |
| The smooth delta distribution | works in whole units of the last decimal place of the currency concerned |
| Comparisons of a price, a discount or a quantity when reloading stored manual amounts | the document currency's precision |
| Comparisons of a distribution factor total | two decimal places |

---

## 10. What an implementer must configure before the domain works at all

1. A company with a country and a fiscal country.
2. At least one Tax Group for that country, with a tax payable and a tax receivable account.
3. The accounts the distribution lines will post to.
4. The company's rounding method and default price inclusion.
5. For cash basis: the switch, the cash basis journal, the base tax received account, and a
   reconcilable transition account on every deferred tax.
6. For withholding: the withholding base account and, on each withholding tax, a sequence.
7. For a tax return: a report definition with a country, report lines, and one tax-tags expression
   per grid, so that the tags exist and can be attached to the distribution lines.
8. For automatic tax substitution: fiscal positions with their criteria, replacement taxes attached
   to them, and each replacement declaring the domestic tax it replaces.

---

## 11. Choosing the rounding method

The company-level rounding method is the single most consequential tax setting. This section
states the difference operationally so that an implementer and an administrator can choose.

| | Round per line | Round per tax |
|---|---|---|
| When is the raw base rounded | immediately, before any tax is computed | never |
| When is a tax amount rounded | immediately, as it is computed | only at the document-wide pass |
| Is the document total the correctly rounded value of the exact arithmetic | no; it is the sum of independently rounded line totals | yes |
| Can a line's reported base differ from its own rounded untaxed total | no | yes, by the delta |
| Is the forward and backward computation symmetric | not guaranteed | yes, for an unrounded input |
| Typical jurisdiction | those that require each line of an invoice to be individually correct | those that require the invoice total to be correct |
| Default | — | **this one** |

**The observable difference.** On three lines of quantity twelve point one two at a unit price of
twelve point one two with a twenty-three percent tax, the two methods give totals of five hundred
forty-two point zero one and five hundred forty-two point zero four respectively. The full
derivation is in `calculations.md` section 7.8.

**A consequence for implementers.** Under "round per tax" the amounts an implementation stores per
line are *not* simply the rounded arithmetic of that line. Any code that recomputes a line's
subtotal from its own price and quantity and compares it with the stored balance will report a
false mismatch whenever the delta is non-zero.

---

## 12. Choosing the default price inclusion

| | Tax excluded (the default) | Tax included |
|---|---|---|
| What a stored product price means | the amount before taxes | the amount the customer pays |
| What an invoice line's unit price means | the same | the same |
| What the line's subtotal column prints | the untaxed subtotal | the total with taxes |
| Effect of changing it | **every** tax that has no override changes meaning at once, and therefore every price does too | the same |

A tax may override the company default in either direction. The override is what an implementer
must consult; the company default is only the fallback.

**A trap.** Changing the company default does not change any stored number; it changes what every
stored number *means*. An installation that switches it after invoicing has begun will report
different totals for identical documents raised before and after the switch. The specification
offers no reconciliation for that; the setting is meant to be chosen once.

---

## 13. Three archetypal configurations

### 13.1 A single-country business with one rate

- One Tax Group for the country, with a payable and a receivable account.
- One sales tax and one purchase tax at the statutory rate, price-excluded, each with a base
  distribution line and one tax distribution line at one hundred percent.
- One report definition for the country with four lines and four tax-tags expressions: the sales
  base, the sales tax, the purchase base and the purchase tax. The four tags are attached to the
  four distribution lines.
- The rounding method as the jurisdiction requires.
- No fiscal position, no cash basis, no withholding.

### 13.2 A business trading inside an economic union

Everything in 13.1, plus:

- A **domestic** fiscal position with the company's country, "detect automatically" on, and the
  domestic taxes attached to it.
- An **intra-union** fiscal position with the union country group, "detect automatically" on and
  "tax registration required" on.
- One reverse-charge sales tax and one reverse-charge purchase tax attached to the intra-union
  fiscal position, each declaring the domestic tax it replaces, each with a distribution of plus
  one hundred and minus one hundred percent and two tax grids.
- An **export** fiscal position with no country, "detect automatically" on, a higher sequence than
  the intra-union one, and zero-rated taxes attached to it declaring the domestic taxes they
  replace.
- Cross-border verification switched on, so that the intra-union fiscal position only applies to a
  counterpart whose registration is confirmed.

The sequence matters: the domestic position must sort before the intra-union one, which must sort
before the export one, because the first match wins.

### 13.3 A business under a payment-based regime with withholding

Everything in 13.1, plus:

- The company's cash basis switch on, a cash basis journal and a base tax received account.
- Every affected tax's exigibility set to "based on payment", each with its own reconcilable
  transition account.
- A withholding tax per statutory retention rate, each with a negative amount, the "withhold on
  payment" flag, a sequence and a distribution posting to a tax-credit or tax-payable account.
- The company's withholding tax base account set, so that the per-line account column disappears.
- Every withholding tax attached to the products it applies to, so that the register-payment
  wizard proposes the lines by itself.

---

## 14. Interaction with localization packages

The packages covered here define the **mechanism**; a localization package supplies the **data**
for one country. The division is:

| Supplied here | Supplied by a localization |
|---|---|
| The Tax, Tax Group, Distribution Line, Fiscal Position and Account Tag entities | the actual taxes, groups, distributions and fiscal positions of a country |
| The engine, the rounding, the accounting derivation | nothing |
| The report skeleton and the tax-tags mechanism | the country's tax return definition, its lines and its expressions — and therefore its tags |
| The per-country number checks and normalisers | occasionally an override of one of them |
| The generic chart of accounts template | the country's chart template |
| The cash basis mechanism | which taxes a country defers |
| The withholding mechanism | the country's retention rates and certificate numbering |
| The union-wide distance-selling action stub | the mapping itself |

A localization must not redefine the engine. When a country needs arithmetic the engine does not
offer, the intended extension point is a **custom formula tax**, which keeps the two engine copies
in step because the formula is transported to the client.

---

## 15. What changes when a capability is removed

| Capability removed | Effect |
|---|---|
| Custom Formula Taxes | every tax whose computation kind was "custom formula" becomes a percentage tax and is archived |
| Withholding Tax on Payment | the flag, the sequence link and the lines disappear; existing payments keep their journal items, which are ordinary items by then |
| Tax Number Validation | numbers stop being normalised and checked; stored numbers are untouched; the cross-border flag stops being recomputed |
| Update Tax Grids | the maintenance operation disappears; existing tag links are untouched |
