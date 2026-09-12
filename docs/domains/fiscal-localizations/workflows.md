# Fiscal Localizations: Workflows

Every operational sequence of the domain, from installing a country package and instantiating its chart of accounts template through to transmitting an electronic invoice and recording the tax administration's answer. Each workflow lists its actors, preconditions, numbered steps, decisions, the records written with their field values, the messages emitted, and the postconditions. State machine tables follow each workflow that drives a state field.

---

## 1. Template registry

### 1.1 What a template is

A **chart of accounts template** is a named bundle of shipped configuration data identified by a **template code** (a lowercase word such as `generic_coa`, `be_comp`, `de_skr03`, `es_pymes`, `ar_ri`). A template is not a stored record. It is assembled at read time from two sources:

1. **Template functions.** Each country package contributes a set of functions. Every function is annotated with the template code it serves and the target model it produces data for. The target model is one of `template_data` (company-level values that are not records), `res.company` (values written directly on the company), or one of the seven record models listed in section 1.3.
2. **Delimited data files.** For the record models, the default template function reads a delimited text file named after the model and the template code, held inside the contributing package under a fixed data folder.

### 1.2 The template mapping

For every installable capability package whose category is "Accounting localizations, charts of accounts", and for the core accounting package, the system evaluates the `template_data` functions and builds a mapping from template code to a descriptor:

| Descriptor key | Type | Meaning |
|---|---|---|
| `name` | text | Display name. Built as the country flag symbol, a space, the country name, and, when the template function supplies a name, a space-hyphen-space and that name. When the template declares no country, the function's own name is used unchanged. |
| `country` | text | Country code as given by the template function, or derived from the part of the template code before the first underscore when the function does not state one. A template function may explicitly state that it has no country, in which case the descriptor holds no country. |
| `country_id` | reference | The Country record matching that country code. |
| `country_code` | text | The two-letter country code of that Country record. |
| `parent` | text | The code of the parent template, if any. |
| `sequence` | integer | Ordering of templates contributed by the same package. Default 1. |
| `visible` | boolean | Default true. A template marked not visible is excluded from the selection list but remains loadable and remains part of parent chains. |
| `installed` | boolean | True when the contributing capability package is installed. |
| `module` | text | The contributing capability package. |

Templates contributed by one package are sorted by `sequence` ascending before being merged into the global mapping.

**Guard.** Two template codes, the West African harmonised accounting system template (`syscohada`) and its non-profit variant (`syscebnl`), are shared bases for sixteen countries. Selecting either directly is refused with the message:

```
The <template code> chart template shouldn't be selected directly. Instead, you should directly select the chart template related to your country.
```

### 1.3 The seven record models a template produces

Templates are loaded in a fixed model order. The order matters because later models reference earlier ones.

| Order | Model | Canonical name |
|---|---|---|
| 1 | Account Group | Account Group |
| 2 | Account | Account |
| 3 | Fiscal Position | Fiscal Position |
| 4 | Tax Group | Tax Group |
| 5 | Tax | Tax |
| 6 | Journal | Journal |
| 7 | Reconciliation Model | Reconciliation Model |

Before loading, two models are moved to the end of the sequence if present: Fiscal Position and Reconciliation Model. Fiscal Position is moved because taxes carry the links back to fiscal positions, and Reconciliation Model is moved because its lines reference accounts that must already exist.

### 1.4 Parent chain composition

`parent_chain(code)` is computed as follows:

1. Start with an empty ordered list.
2. While the current code is a known template code, append the current code to the list and replace the current code by the parent code recorded in that template's descriptor.
3. When the current code is no longer a known template code, the list is the parent chain.

The chain therefore reads from the most specific template to the most generic. For example the Spanish small and medium enterprise chart yields `["es_pymes", "es_common"]`, and the Argentine registered taxpayer chart yields `["ar_ri", "ar_base"]`.

**Composition order for functions.** Template data is assembled by iterating the list `[none] + parent_chain(code)`, where `none` denotes the functions that are registered without a template code and therefore apply to every template. For each element of that list, functions are grouped by target model and the models are visited in the order of section 1.3. Because the parent chain runs specific-first, a value produced by a more specific template is written *before* the parent's value and then **overwritten** by the parent when the parent also sets the same key on the same row. Country packages therefore place overrides in the parent-most template function that is still specific to the intended scope.

**Composition order for delimited data files.** Inside a single template function that reads delimited files, the chain is walked in *reverse* (`parent_chain(code)` reversed, that is generic first, specific last), and each file found is merged into the accumulated result by row identifier. A row present in both the parent file and the child file has the child's non-empty column values applied last, so the **child wins** for delimited data. When no parent chain exists, the file whose name carries no template suffix is read.

**Worked example.** The Spanish package ships an account file for the shared base template and an account file for the small and medium enterprise template. Loading `es_pymes` reads the base file first (598 account rows) then the small and medium enterprise file (9 rows). Nine of the base rows are replaced column by column, and the result is 598 accounts of which 9 carry the small and medium enterprise names and codes.

### 1.5 Reading a delimited data file

Each file has a header row. Column `id` holds the row's symbolic identifier. Every other column is either a field name of the target model, a field name suffixed with `@` and a language code (a translation of that field), or a path of the form `relation/field` or `relation/sub_relation/field` describing a nested record to create.

Parsing rules:

1. A row whose `id` is non-empty starts a new record and becomes the current record. Its columns are read into the record: a column is kept only when the value is non-empty and the column name (up to any `@`) is a field of the target model or carries a `@`.
2. A row whose `id` is empty continues the previous record. Only its path columns are read.
3. A path column creates, on first use of that path within the current record, a new nested record instruction appended to the named relation, and then writes the leaf field into that nested record. All path columns sharing the same path prefix within one row write into the same nested record.
4. Value conversion: a boolean, integer or decimal field value is parsed from its text form; a text field value is stripped of leading and trailing whitespace; a translation column value is kept verbatim; a path column is initialised as an empty list.

**Worked example.** Four consecutive rows of a tax file, the first carrying the identifier `sale_tax_template` and the following three empty, produce one Tax record with four repartition instructions:

```
row 1: id=sale_tax_template, name=15%, amount=15, type_tax_use=sale,
       repartition_line_ids/document_type=invoice,
       repartition_line_ids/factor_percent=100,
       repartition_line_ids/repartition_type=base
row 2: id=(empty), repartition_line_ids/document_type=invoice,
       repartition_line_ids/factor_percent=100,
       repartition_line_ids/repartition_type=tax,
       repartition_line_ids/account_id=tax_received
row 3: id=(empty), repartition_line_ids/document_type=refund, ... repartition_type=base
row 4: id=(empty), repartition_line_ids/document_type=refund, ... repartition_type=tax,
       account_id=tax_received
```

Result: Tax `sale_tax_template`, name "15%", amount 15, scope sale, with four repartition lines: invoice base 100 percent, invoice tax 100 percent to account `tax_received`, refund base 100 percent, refund tax 100 percent to account `tax_received`.

### 1.6 Tax report tag resolution

Tax repartition instructions may carry a `tag_ids` column. Its value is a text list of tag names separated by two vertical bars. Each name is resolved as follows:

1. If the name matches the pattern `word.word` and the part before the dot is the name of a known capability package, the name is taken to be an explicit external identifier and is passed through unchanged.
2. Otherwise the name is normalised by trimming and collapsing runs of whitespace to single spaces, and looked up among Account Tags whose applicability is "taxes" and whose country is the template's country, searched with archived tags included and in the base language.
3. When no tag is found and the caller has not asked for missing tags to be ignored, the load is aborted with a redirecting warning whose message is:

```
Error while loading the localization: missing tax tag <tag name> for country <country name>. You should probably update your localization app first.
```

The warning offers a button labelled "Update app" leading to the package browser filtered on the country name and on the accounting localization charts category. When the caller has asked for missing tags to be ignored (used during translation loading), the missing tag is recorded in the technical log and skipped.

A leading hyphen on a tag name selects the negative-signed variant of that tag. Tag creation itself is driven by report expressions, described in section 6.

---

## 2. Selecting a chart of accounts template

**Actors.** System Administrator.

**Preconditions.** The company exists. The accounting capability package is installed.

**Selection list.** The list offered to the user contains every visible template, ordered so that the templates whose country equals the company's country come first, in their natural order, and all other templates follow. When the company has no country, the generic chart of accounts is placed first instead.

**Guessing.** When no template code is supplied, the first entry of the selection list for the company's country is used. This is the operation `guess_chart_template(country)`.

**Automatic selection on package installation.** When a capability package that contributes templates moves from not installed to installed, and the company currently has no template, the system chooses the first template of that package whose country equals the company's country, or failing that the generic chart of accounts, and schedules it to be loaded once the registry is ready.

**Automatic selection on country assignment.** When the localization packages of a company's country are installed as a consequence of setting the company country, every company that has a country and no template is scheduled for loading. The template code used is the parent company's template when the company has a parent, otherwise the guess for the company's country. The generic chart of accounts is not auto-loaded through this path.

**Manual selection.** The Accounting settings screen shows a Package field listing the selection. Saving the settings with a different package triggers a load. The settings screen also reports whether the company already has accounting entries; changing the package once entries exist is refused by the loading guard of section 3.

**Subsidiary companies.** A subsidiary always follows its parent's template. Creating a company whose parent chain already has a template schedules a load of that same template on the new company at the end of the transaction.

**Uninstalling a package.** When a capability package that contributes templates is uninstalled, every company whose template code belongs to that package has its template code cleared.

---

## 3. Loading a chart of accounts template

**Operation name.** `try_loading(template_code, company, install_demonstration_data, force_create)`.

**Actors.** System Administrator. The operation refuses to run for a user who is not a system administrator, with the message:

```
Only administrators can install chart templates
```

**Inputs.**

| Input | Type | Default | Meaning |
|---|---|---|---|
| `template_code` | text | guessed from the company country | The template to load. |
| `company` | reference | required | The company to load into. When absent the operation returns without doing anything. |
| `install_demonstration_data` | boolean | false | Whether to load the demonstration data set after a first load. |
| `force_create` | boolean | true | When true, missing records are created. When false, no record is created and only existing records are updated. |

### 3.1 Step by step

1. **Guard on shared bases.** Refuse the two shared West African template codes when they differ from the company's current template, with the message given in section 1.2.
2. **Fetch the descriptor** for the template code. If the company has no country, set the company country to the descriptor's country.
3. **Install the contributing package** when it is not installed. The installation resets the running transaction and the operation continues against the refreshed registry.
4. **Set the working context**: the target company becomes the default and the only allowed company; discussion-thread tracking is disabled; account group re-parenting is deferred; the working language is forced to the base language so that shipped text is stored untranslated and translated afterwards; a flag marks that a template load is in progress, which suppresses the automatic localization package installation that a country change would otherwise trigger.
5. **Detect a reload.** `reload = (template_code equals company.chart_template)`. Write the template code on the company.
6. **Purge on a fresh load.** When this is not a reload, and either the company's root has no accounting entries at all or demonstration data was requested, delete the existing configuration in this order: Journal Entry, then the seven record models in reverse of the order in section 1.3 (Reconciliation Model, Journal, Tax, Tax Group, Fiscal Position, Account, Account Group). The purge runs only when the company has no parent. For each model, all records belonging to the company or any of its descendants are selected with archived records included. For a model scoped by a list of companies (Account), records that also belong to at least one company outside the branch are kept and merely have the branch companies removed from their company list; all other records are deleted. Deletion runs with the package-uninstall flag set, which lifts the ordinary deletion guards.
7. **Assemble the data** for the template code as described in section 1.4, producing a mapping from model name to a mapping from symbolic row identifier to field values, plus a separate `template_data` mapping of company-level values.
8. **Narrow to company values for a subsidiary.** When the company has a parent, discard every model except the company values.
9. **Pre-process a reload** when step 5 detected one; see section 4. Demonstration data is never loaded on a reload.
10. **Pre-process the load**; see section 3.2.
11. **Write the records**; see section 3.3.
12. **Post-process the load**; see section 3.4.
13. **Load translations**; see section 3.5.
14. **Re-parent account groups.** The deferred account group synchronisation runs now for the company: every account group is linked to the most specific enclosing group by code prefix.
15. **Load demonstration data** when requested and this was not a reload. The demonstration load runs inside a savepoint and in the user's original language. A failure inside the savepoint is recorded in the technical log and does not roll back the chart of accounts.
16. **Recurse into subsidiaries.** For every direct child company, run the same load with the same template code, the same demonstration flag and the same creation flag.

### 3.2 Pre-processing (step 10)

1. **Determine the fiscal country.** If the company values carry a fiscal country, resolve it; otherwise use the company's current fiscal country.
2. **Select the company-level values to write.** From `template_data`, keep every key that is a field of the company and that is not the template's display name, and drop every key beginning with `property_` except those beginning with `property_stock_` and except the key `additional_properties`. The dropped `property_` keys are default values for other models and are applied later in post-processing.
3. **Set the currency.** When the company's root has no accounting entries: a subsidiary takes its parent's currency; a top company takes the fiscal country's currency.
4. **Set the country** to the fiscal country when the company has none.
5. **Default the cost accounting flag.** The key that selects perpetual inventory expense recognition at the moment of the customer invoice is defaulted to false when the template does not set it, so that switching from a template that sets it true to one that omits it clears it.
6. **Write the values on the company.** This write activates the currency if it was inactive and reflects prefix changes onto existing liquidity accounts.
7. **Pad account codes.** Let `code_digits` be the template value `code_digits`, defaulting to 6. Every account row's code is left-justified within `code_digits` characters and padded on the right with the character zero. Codes already at or beyond that length are unchanged.

   ```
   padded_code = code followed by max(0, code_digits − length(code)) zero characters
   ```

   Worked example with `code_digits = 6`: code `1010` becomes `101000`; code `40` becomes `400000`; code `1234567` stays `1234567`.
8. **Move Fiscal Position and Reconciliation Model to the end** of the model ordering.
9. **Drop unknown columns.** Unless the caller asked for a completeness check, every key of every row whose name (up to any `@`) is not a field of the target model is removed. The translation-module bookkeeping key is preserved.
10. **Translate the untranslatable fields that must be translated anyway.** One field is treated this way: the Journal `code`. The target language is the company contact's language, or the active language of the session when the contact has none. For each such field, if the row carries an explicit translation column for that language (or for the generic form of that language, that is the part before the underscore), the translation replaces the value. Otherwise the value is looked up in the contributing package's compiled text catalogue for that language, and then in the accounting package's catalogue.

### 3.3 Writing the records (step 11)

The writer receives the model-to-rows mapping and processes models in the order established above. Two mechanisms make forward references possible.

**Deferral.** A field value is deferred to a later pass when all of the following hold: the field is relational; its target model has not been written yet; and its target model is either the current model itself or a model still queued. For a list-valued relation, the check descends into nested creation instructions. When a Tax row has its repartition instructions deferred, and the Tax does not already exist, a clearing instruction is prepended to the deferred list, because creating a Tax without repartition instructions causes the tax engine to generate default ones that must be removed. Deferred values are collected per model and appended to the queue as an extra pass over the same model.

**Symbolic reference resolution.** Just before writing a row, every value is resolved:

| Value shape | Resolution |
|---|---|
| falsy | written as false |
| text on a single-record relation, or text on an integer or polymorphic-identifier field that is not all digits | resolved through the external identifier lookup of section 3.6; a failure on the company model falls back to the current company's own value for that field, then to the root company's value, then to false; a failure on any other model records a technical log entry, drops the field and continues |
| list of instructions | for a create or update instruction, the nested values are resolved against the related model; for a replace-all instruction, each text element is resolved; for a link instruction whose identifier is text, the identifier is resolved |
| text on a list-valued relation | split on commas and turned into a single replace-all instruction over the resolved identifiers |

**Row identity.** Before writing, translation columns and the translation-module bookkeeping key are removed. If the symbolic identifier already resolves to an existing record, the row is written onto that record's database identifier and no external identifier is created. Otherwise the row is written with the external identifier `account.<company identifier>_<symbolic identifier>`, marked as not to be updated by future package upgrades.

### 3.4 Post-processing (step 12)

1. **Create the utility liquidity accounts.** For each of the six utility accounts below that the company does not already have, a record is created. A subsidiary does not create them; it copies the value from the first company in its parent chain.

   | Company field | Account name | Code source | Account type | Reconcilable | Tags |
   |---|---|---|---|---|---|
   | Bank Suspense Account | "Bank Suspense Account" | next free code under the bank account code prefix, padded to `code_digits` | Current Asset | no | none |
   | Cash Discount Write-Off Loss Account | "Cash Discount Loss" | fixed code `999998` | Expense | no | none |
   | Cash Discount Write-Off Gain Account | "Cash Discount Gain" | fixed code `999997` | Other Income | no | none |
   | Cash Difference Income Account | "Cash Difference Gain" | next free code under prefix `999`, padded to `code_digits` | Other Income | no | Investing activity |
   | Cash Difference Expense Account | "Cash Difference Loss" | next free code under prefix `999`, padded to `code_digits` | Expense | no | Investing activity |
   | Inter-Banks Transfer Account | "Liquidity Transfer" | next free code under the transfer account code prefix, padded to `code_digits` | Current Asset | yes | none |

2. **Create the outstanding accounts.** Only for a company without a parent, and without writing any company field: "Outstanding Receipts" and "Outstanding Payments", both Current Asset, both reconcilable, both coded under the bank account code prefix and padded to `code_digits`. They are registered under the external identifiers `account_journal_payment_debit_account_id` and `account_journal_payment_credit_account_id` for the company.
3. **Ensure an unaffected earnings account** exists. The first account of type "Current Year Earnings" for the company is returned; when none exists, the caller is redirected to the accounting configuration panel with the message:

   ```
   We cannot find a chart of accounts for this company, you should configure it.
   Please go to Account Configuration and select or install a fiscal localization.
   ```

4. **Wire the liquidity journals.** For every Bank, Cash and Credit journal of the company, set the suspense account to the company's bank suspense account when the journal has none, the profit account to the company's cash difference income account when the journal has none, and the loss account to the company's cash difference expense account when the journal has none.
5. **Wire the company journals.** When the company has no cash basis journal, point it at the journal created under the symbolic identifier `caba`. When the company has no exchange difference journal, point it at the journal created under the symbolic identifier `exch`.
6. **Wire the sale and purchase journals.** When a journal exists under the symbolic identifier `sale` and the company has an income account, set the sale journal's default account to it. When a journal exists under the symbolic identifier `purchase` and the company has an expense account, set the purchase journal's default account to it.
7. **Set the company default taxes.** When the company has no default sale tax, take the first tax of the company whose scope is sale or all. When the company has no default purchase tax, take the first tax of the company whose scope is purchase or all.
8. **Propagate default taxes to products.** When a default sale tax exists, every product template of this company that already has at least one sale tax but none belonging to this company has the company default sale tax forced onto it. The same rule applies to purchase taxes with the default purchase tax. Products with no tax at all are left untouched, because several flows (for example tips in point of sale) require a product without tax.
9. **Reveal cash basis fields.** When the company has no parent and at least one tax with exigibility "on payment" exists, set the company's cash basis flag to true.
10. **Write the property defaults.** For each entry of the property map, when `template_data` holds a value for the key and the key is a field of the target model, a company-scoped default value is recorded for that model and field pointing at the resolved record. The base property map is:

    | Key | Target model |
    |---|---|
    | `property_account_receivable_id` | Contact |
    | `property_account_payable_id` | Contact |
    | `property_stock_journal` | Product Category |

    A template may extend the map through the `additional_properties` key of `template_data`, whose value is a mapping from key to target model.
11. **Write the product category income and expense defaults.** A company-scoped default income account and default expense account for Product Category are recorded from the company's income account and expense account.
12. **Point the internal transfer reconciliation model** at the company's inter-banks transfer account.
13. **Point the bank fee reconciliation model** at the first account of the company whose name contains "Bank Fees"; failing that, the first account of the company whose type is Expense.
14. **Start the accounting onboarding** panels for the company.

### 3.5 Loading translations (step 13)

1. The target languages are the supplied list, or every installed language.
2. For each company in scope, the template data is re-assembled with missing tax tags tolerated and with the company as the active company, and the company-level values are dropped.
3. For every row of every model, for every field name appearing in the row (with any `@` suffix stripped), when the field is marked translatable, a translation is resolved for each target language:
   - the explicit translation column for that exact language, else
   - the explicit translation column for the generic form of that language, else
   - the field's base-language value looked up in the compiled text catalogue of the package recorded for that field in the translation-module bookkeeping key (defaulting to the accounting package), for that language, then for the generic form of that language.
   When a translation is found, it is queued for the record identified by the row's external identifier.
4. Records of the seven template models that were not created from template data are also translated. For each model with at least one translatable field, the system queries the records belonging to the companies in scope that have at least one translatable field lacking a translation in at least one target language, together with their external identifier and contributing package. For each such field with a base-language value, the value is looked up first in the contributing package's catalogue and then in the accounting package's catalogue. When the base-language value is a rich text value wrapped in a single division element and no direct match is found, the inner text is looked up and the result is re-wrapped in a division element.
5. All queued translations are saved without overwriting translations that already exist.

### 3.6 External identifier convention

Records created by a template load are addressable by a **company-qualified external identifier**:

```
company_qualified(symbolic_identifier, company) =
    symbolic_identifier                                  if it already contains a dot
    "account." + company.identifier + "_" + symbolic_identifier   otherwise
```

Resolution of a symbolic identifier first tries the current (or supplied) company's qualified identifier, and on failure tries the qualified identifier of the first company in the company's parent chain. A subsidiary therefore transparently sees the records created for its parent.

### 3.7 State of the company before and after

| Field | Before a first load | After a first load |
|---|---|---|
| Chart of accounts template code | empty | the loaded template code |
| Fiscal country | the company country or empty | the template's fiscal country |
| Currency | platform default | the fiscal country's currency, or the parent's currency for a subsidiary |
| Bank, cash and transfer account code prefixes | empty | template values |
| Default receivable and payable accounts | empty | recorded as company-scoped Contact defaults |
| Income account, expense account | empty | template values |
| Default sale tax, default purchase tax | empty | first matching tax |
| Cash basis journal, exchange difference journal | empty | the journals created under `caba` and `exch` |
| Suspense, outstanding, cash difference, cash discount, transfer accounts | empty | created utility accounts |

---

## 4. Reloading a chart of accounts template

**Trigger.** `try_loading` is called with a template code equal to the company's current template code.

**Intent.** Refresh the shipped configuration without destroying user data and without invalidating posted accounting. The rules below run as a pre-processing pass that rewrites the assembled data before the ordinary writer of section 3.3 executes it.

### 4.1 Global narrowing

1. Every `template_data` key beginning with `property_` is dropped: default receivable and payable accounts are never re-imposed on a reload.
2. Reconciliation Models are dropped entirely.
3. The company values are cleared except that the cost accounting flag is preserved at its current value.
4. Demonstration data is not loaded.

### 4.2 Journals

For each journal row:

- If the symbolic identifier already resolves, the row is dropped.
- Otherwise the system attempts to adopt an existing journal:
  1. If the row carries a `code`, search the company's journals (archived included) for a journal whose code equals the translated code (resolved for the company contact's language) or, failing a translation, the row's code.
  2. If nothing was found and the row carries both a `name` and a `type`, search the company's journals (archived included) for a journal of that type whose name is either the row's name or its translation, taking the first match. Matching on the name avoids breaking the uniqueness constraint on the journal's incoming mail alias.
  3. When a journal is adopted, the row is dropped and the journal is bound to the row's company-qualified external identifier, marked as not to be updated.

### 4.3 Account groups

If the company already has at least one account group (counted across all companies when the company has a parent), the Account Group model is dropped from the data.

### 4.4 Fiscal positions

For each fiscal position row:

- If the symbolic identifier does not match an existing fiscal position and creation is not forced, the row is skipped entirely.
- If the symbolic identifier does not match an existing fiscal position and creation is forced, the row is kept as is.
- If the symbolic identifier matches an existing fiscal position, the account mapping instructions are filtered: when creation is not forced the whole mapping is dropped; when creation is forced, only those create instructions are kept whose source account symbolic identifier does not resolve, or whose destination account symbolic identifier is present and does not resolve. In other words only mappings that introduce a *new* account are applied, and existing mappings are never overwritten.

### 4.5 Tax groups

For each tax group row:

- If the symbolic identifier does not match an existing tax group and creation is not forced, the row is skipped.
- If the symbolic identifier matches an existing tax group, the tax payable account and the tax receivable account are removed from the row whenever the symbolic identifier they carry already resolves, so an existing wiring is never replaced.

### 4.6 Taxes

Let `template_changed(tax, row)` be true when any of the following holds:

```
tax.amount_type ≠ row.amount_type (default "percent")
OR round(tax.amount, 4) ≠ round(row.amount, 4)   (default row amount 0)
OR count(row repartition instructions that are not "clear") ∉ {0, count(tax repartition lines)}
```

Branch A, the tax is new or materially changed:

1. When creation is not forced, skip the row.
2. When the context asks for newly created taxes to be active, force the row's active flag to true.
3. Identify the taxes to retire: the tax bound to the symbolic identifier if there is one (whose binding is then queued for deletion), otherwise every tax of the company whose name, scope and product applicability equal the row's.
4. Build the retirement key `(name, scope, product applicability, company)` from the first tax to retire. Count how many existing taxes already carry a name matching the pattern "optional `[old` + optional digits + `] ` prefix followed by the exact name" with the same remaining key parts. Call that count `matching_names`.
5. Rename each tax to retire. For the tax at position `index` (zero based) in the list, let `rename_index = index + matching_names`. When `rename_index` is zero the tax keeps its name. When `rename_index` is one the tax is renamed to `[old] <name>`. When `rename_index` is `n` greater than one the tax is renamed to `[old<n−1>] <name>`.

   Worked example: a company has taxes "21%" and "[old] 21%". A reload brings a changed "21%". `matching_names` is 2. The single tax to retire is at index 0, so `rename_index` is 2, and it is renamed "[old1] 21%". The new tax is created as "21%".
6. The row is kept and will create a new tax.

Branch B, the tax exists and is materially identical:

1. Remember the row's fiscal position links, its source tax links, and its repartition instructions, then clear the row.
2. Re-link fiscal positions: for each symbolic identifier in the fiscal position list, add a link instruction, keeping only identifiers that resolve to an existing fiscal position unless creation is forced.
3. Re-link source taxes: only when creation is forced, and only for identifiers that do not already resolve to an existing tax, add link instructions.
4. Restore the repartition instructions but reduce each nested instruction to its report tags alone. An instruction with no tags becomes a clearing of the tags. Nothing else about the repartition line is touched.

At the end of the pass, every queued binding deletion is executed: the external identifier records named `<company identifier>_<symbolic identifier>` under the accounting package are deleted.

### 4.7 Accounts

For each account row:

1. Compute `normalized_code` as the row's code padded to `code_digits` with zero characters.
2. Resolve the symbolic identifier to an existing account. If none resolves, or the resolved account's code does not match the pattern "row code followed by zero or more zero characters", search the company's accounts (archived included) for any account whose code matches that same pattern. Sort the matches so that an exact equality with `normalized_code` comes first and adopt the first one. Bind the adopted account to the row's company-qualified external identifier, marked as not to be updated.
3. Remove the `reconcile` column from the row. Re-imposing reconcilability would override a user setting and could raise a partial reconciliation error.
4. If an account was found and the row carries report tags, reduce the row to the tags alone.
5. Otherwise, if an account was found, or creation is not forced, skip the row.

### 4.8 Converting creations into updates

After all of the above, for every remaining row of every model, every list-valued relation whose value is a list of instructions is inspected. When the row's symbolic identifier resolves to an existing record, each create instruction at position `i` is converted into an update instruction targeting the existing related record at position `i`. This is what allows an existing tax's repartition lines to have their tags refreshed in place rather than being duplicated.

### 4.9 Reload state table

| Object | Exists before | Materially changed | Outcome |
|---|---|---|---|
| Account group | any exist | not relevant | model dropped, nothing written |
| Account | no | not relevant | created when creation forced, else skipped |
| Account | yes | not relevant | only report tags refreshed; code, name, type, reconcilability untouched |
| Tax group | no | not relevant | created when creation forced, else skipped |
| Tax group | yes | not relevant | payable and receivable accounts preserved when already wired |
| Tax | no | not relevant | created when creation forced, else skipped |
| Tax | yes | no | fiscal position links and report tags refreshed; nothing else changes |
| Tax | yes | yes | old tax renamed with an `[old]` prefix and unbound; a new tax is created when creation forced |
| Fiscal position | no | not relevant | created when creation forced, else skipped |
| Fiscal position | yes | not relevant | only account mappings that introduce a new account are added |
| Journal | bound | not relevant | dropped |
| Journal | unbound but matching code or name and type | not relevant | adopted and bound |
| Journal | no match | not relevant | created |
| Reconciliation model | any | not relevant | model dropped |
| Company default accounts | any | not relevant | not re-imposed |

---

## 5. Instantiating foreign taxes

**Actors.** Accounting Manager.

**Preconditions.** A fiscal position exists on the company, carries a foreign tax identification number and a country, and that country has no tax yet in this company.

**Trigger.** The button "Create foreign taxes" shown on the fiscal position when its foreign registration header mode is set.

**Steps.**

1. Guess the chart of accounts template for the fiscal position's country. If the contributing capability package is not installed, install it.
2. Verify that the company has no tax whose country is the target country. If any exists, stop silently.
3. Assemble the target template's tax group data and tax data.
4. **Map the tax group accounts.** For each of the three tax group account fields (tax payable account, tax receivable account, advance tax payment account) and each tax group row, when the row names an account symbolic identifier that has not been mapped yet, find the company's first tax group whose country is the company's fiscal country and that has a value in the same field. When found, create a new account in the company by copying that local account's type, reconcilability and non-trade flag, giving it the next free code after the local account's code and the name:

   ```
   <local account name> - Foreign tax account payable (<country code>)
   <local account name> - Foreign tax account receivable (<country code>)
   <local account name> - Foreign tax account advance payment (<country code>)
   ```

5. **Map the repartition accounts.** For each tax row and each tax-type repartition instruction that names an account, when that symbolic identifier has not been mapped yet, search for a comparable local repartition line. The search starts with the most restrictive filter and relaxes it one condition at a time until a match is found or the filter is exhausted:

   | Attempt | Conditions, in addition to "belongs to the company", "has an account", and "factor sign matches the template's factor sign" |
   |---|---|
   | 1 | tax scope equals the template tax's scope; tax country equals the company fiscal country; tax is one of the company's two default taxes |
   | 2 | tax scope equals the template tax's scope; tax country equals the company fiscal country |
   | 3 | tax scope equals the template tax's scope |
   | 4 | no additional condition |

   The factor sign comparison is "less than zero" when the template's factor percentage is negative and "greater than zero" otherwise. The default template factor percentage is 100. When a match is found, a new account is created by copying the matched account, with the next free code and the name `<local account name> - Foreign tax account (<country code>)`. When no match is found, the field is left empty rather than filled with a wrong account.
6. **Map the cash basis transition accounts.** Locate the company's first tax whose country is the fiscal country, whose exigibility is "on payment" and which has a cash basis transition account. Then iterate the template's tax rows, ordered so that rows having at least one repartition account come first. For every row with exigibility "on payment":
   - when a local cash basis tax was found, create a reconcilable copy of its transition account named `<local account name> - Cash basis transition account`;
   - otherwise, when the row has at least one repartition account already mapped, create a reconcilable copy of the first mapped account with the same name suffix;
   - otherwise leave the mapping empty.
   When at least one such row exists, set the company's cash basis flag to true.
7. **Rewrite the template data.** Replace every tax group account symbolic identifier and every repartition account symbolic identifier with the mapped account. Set every tax row's country to the target country. Prefix every tax row's tax group reference with the template code and an underscore. Drop every fiscal position link and every source tax link from the tax rows, because template fiscal positions must not be applied and the mapping cannot be determined. Replace the cash basis transition account reference with the mapped account.
8. **Prefix all symbolic identifiers** of the tax group rows and the tax rows with the template code and an underscore, to avoid collision with the local chart. For a group-type tax, each child tax reference is prefixed the same way.
9. **Write the records** using the ordinary writer of section 3.3.
10. **Link the created taxes** to the fiscal position that triggered the operation.

**Postcondition.** The company now owns a set of taxes whose country is the foreign country, grouped under prefixed tax groups, posting to newly created accounts placed next to their local counterparts, and attached to the foreign registration fiscal position. The result is explicitly a fast start, not a certified configuration.

---

## 6. Statutory report structure and preparation

### 6.1 Structure

A **Financial Report** is a tree of **Financial Report Lines**. Each line owns one or more **Financial Report Expressions**. A **Financial Report Column** names an expression label and decides how the resulting figure is rendered. An **Financial Report External Value** records a manually entered or carried-over figure for a given expression, company and date.

Report-level settings:

| Setting | Values | Effect |
|---|---|---|
| Availability | Country Matches, Chart of Accounts Matches, Always | Determines whether the report is offered to a company. Country Matches requires the report's country to equal the company's fiscal country or a country for which the company holds a foreign registration. Chart of Accounts Matches requires the report's chart of accounts template code to equal the company's. |
| Country | a country | Also determines which report tags belong to this report. |
| Chart of accounts | a template code | Used by the Chart of Accounts Matches availability. |
| Root report | another report | Marks this report as a country variant of a generic report. A variant inherits the root's filter defaults. |
| Sections | a list of reports | Makes this report composite: it presents each section in turn and prints them together. |
| Only tax exigible lines | boolean | Restricts the journal items considered to those whose tax is currently exigible. |
| Allow foreign registration | boolean | Permits running the report under a foreign registration fiscal position. |
| Load more limit | integer | Number of child rows fetched per expansion. |
| Search bar | boolean | Offers a text filter over line names. |
| Prefix groups threshold | integer, default 4000 | Above this number of rows, results are grouped by account code prefix. |
| Integer rounding | Nearest, Up, Down | Rounding applied when figures are rendered as whole units. |
| Default opening date filter | This Year, This Quarter, This Month, Today, Last Month, Last Quarter, Last Year, This Return Period, Last Return Period | The period pre-selected when the report opens. Defaults to Last Month. |
| Currency translation | Use the most recent rate at the date of the report, Use cumulative translation adjustment | How balances in other currencies are converted. Defaults to the cumulative translation adjustment. |

Filter settings, each of which shows or hides a control and each of which defaults from the root report, or from the single composite parent when the report is a section of exactly one composite report and is not itself reachable as a standalone report:

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

### 6.2 Line settings

| Setting | Meaning |
|---|---|
| Name | Displayed label, translatable, required. |
| Code | Unique within the report. Referenced by aggregation formulas. |
| Sequence | Ordering. A parent line must appear before its children. |
| Parent line | Builds the tree. A line cannot be its own parent. |
| Level | Computed. A root line is level 1. A child of a level 0 line gains 3; any other child gains 2. |
| Group by | Comma separated journal item field names. When set, the line expands into one sub-line per distinct value. |
| User group by | Same, editable by the user, defaulting to the built-in group by and reverting to it when the chosen value is not supported. |
| Foldable | When set, the line is collapsed by default and shows an expansion control. |
| Print on new page | This line and everything after it start a new printed page. |
| Action | Turns the line into a link executing the given action. |
| Hide if zero | The line and its children are hidden when all their columns are zero. |
| Horizontal split side | Left or Right; inherited from the parent. Used by two-column statements. |

### 6.3 Expression engines

Each expression has a label (unique within the line), an engine, a formula, an optional subformula, a date scope, a figure type, a growth-direction flag, a blank-if-zero flag and an auditable flag.

| Engine | Formula | Subformula | Meaning |
|---|---|---|---|
| Domain | A condition expression over journal items | required; `sum` or `-sum` | Sums the balances of the journal items matching the condition, negated when the subformula is `-sum`. |
| Tax Tags | A report tag name | none | Sums the signed contributions of journal items carrying the positive or negative variant of the named tag, resolved within the report's country. A leading hyphen on the name inverts the sign. |
| Aggregation | An arithmetic expression over terms of the form `line_code.expression_label`, numbers and the operators plus, minus, multiply and divide, with parentheses; or the reserved word `sum_children` | optional `cross_report(<report identifier or external identifier>)`, or `if_other_expr_above(line_code.label, value)` or `if_other_expr_below(line_code.label, value)` | Combines other expressions. `sum_children` sums the same-labelled expression of every child line. |
| Prefix of Account Codes | A signed sum of prefix terms | none | Sums balances of accounts whose code starts with the given prefixes. |
| External Value | `sum` or `most_recent` | `editable`, optionally with `;rounding=<digits>` | Reads manually entered or carried-over values. |
| Custom Function | The name of a computation supplied by a country package | free | Country-specific computation. |

**Prefix of account codes syntax.** The formula is split before every plus or minus sign after all spaces are removed. Each resulting term must match:

```
[+|-] prefix [ \( excluded_prefix , excluded_prefix , ... \) ] [D|C]
```

where `prefix` is either a run of letters, digits and dots, or the form `tag(external identifier)`, and the optional trailing letter restricts the term to accounts whose balance is a debit (`D`) or a credit (`C`). A term whose prefix is empty is rejected with:

```
Invalid formula for expression '<label>' of line '<line name>': <formula>
```

**Aggregation syntax.** The whole formula must match either the reserved word `sum_children` or a sequence of parenthesised numbers and line-code references separated by operators. The same message is raised on failure.

**Domain syntax.** The formula must parse as a condition expression and must be accepted by a search over journal items. The same message is raised on failure. A domain expression must always have a subformula; the database enforces this with the check:

```
engine ≠ "domain" OR subformula IS NOT NULL
```

and the message "Expressions using 'domain' engine should all have a subformula."

**Date scope.** One of: from the very start, from the start of the fiscal year, at the beginning of the fiscal year, at the beginning of the period, strictly on the given dates (default), from the previous return period.

**Figure type.** Monetary, Percentage, Integer, Float, Date, Datetime, Boolean, String.

**Auditable.** Computed from the engine: every engine except Custom Function is auditable, meaning the figure can be expanded into the journal items behind it.

### 6.4 Report tag lifecycle

Report tags are Account Tags with applicability "taxes" and a country. They are created and maintained by tax-tag expressions:

1. **On creating a tax-tag expression**, when no tag with that name exists for the report's country, a tag is created with the name stripped of any leading hyphen, applicability "taxes" and the report's country. The tag has a positive and a negative variant.
2. **On changing an expression's formula**, when no tag exists for the new name, the system checks whether *every* expression currently using the old tag is part of the change. If so, the existing tag is renamed. If not, a new tag is created for the new name.
3. **On changing an expression's engine to tax tags**, tags are created for the resulting names before the write.
4. **On deleting a tax-tag expression**, for every tag it used, the system looks for another tax-tag expression with the same formula in the same country. If one exists, nothing happens. If none exists, the system looks for a journal item carrying the tag: when one exists the tag is archived, otherwise it is deleted. In both cases the tag is first unlinked from every tax repartition line that referenced it.
5. **On changing a report's country**, every tax-tag expression of that report is examined. When all reports using a tag are among those being changed, the tag's country is changed. Otherwise the existing tag is kept and a new tag is created in the target country when none exists there.

### 6.5 Carryover

An expression whose label begins with `_carryover_` produces an amount to be moved into a later period. Its target is:

- the expression named by the carryover target setting, written as `line_code.expression_label`, or
- the expression of the same line whose label is `_applied_carryover_` followed by the part of the carryover expression's label after `_carryover_`.

Validation:

```
carryover target set AND label does not start with "_carryover_"
  → "You cannot use the field carryover_target in an expression that does not have the label starting with _carryover_"

carryover target set AND the label part after the dot does not start with "_applied_carryover_"
  → "When targeting an expression for carryover, the label of that expression must start with _applied_carryover_"

no target can be determined
  → "Could not determine carryover target automatically for expression <label>."
```

The carried amount is stored as a Financial Report External Value for the company, dated at the end of the period in which it arose, and referring back to the originating line and label. When the later period is computed, the `_applied_carryover_` expression reads the stored values.

**Worked example.** A country allows a value-added tax credit to be carried forward rather than refunded. Period March closes with a net credit of 1,250.00. The `_carryover_balance` expression of line `NET` evaluates to 1,250.00, its target `NET._applied_carryover_balance` receives an external value of 1,250.00 dated 31 March. In April, `NET._applied_carryover_balance` reads 1,250.00, the April net payable of 2,000.00 is reduced to 750.00, and nothing is carried further.

### 6.6 Report copy

Copying a report copies the whole tree:

1. The report is copied with the name `<name> (copy)`, repeating the suffix until the name is unique.
2. Each root line is copied recursively. A copied line's code is `<code>_COPY`, repeating the suffix until the code is unique. A line with no code keeps none.
3. Every expression of each line is copied onto the copied line.
4. Every aggregation formula and subformula on the copied report has each old line code replaced by its new code.
5. Every column is copied onto the new report.

**Deletion guard.** A report that has variants cannot be deleted:

```
You can't delete a report that has variants.
```

Other structural guards:

```
Only a report without a root report of its own can be selected as root report.
The sections defined on a report cannot have sections themselves.
The Availability is set to 'Country Matches' but the field Country is not set.
Line "<line>" defines line "<parent>" as its parent, but appears before it in the report. The parent must always come first.
Line "<line>" defines itself as its parent.
A line cannot have both children and a groupby value (line '<parent name>').
A report line with the same code already exists.
The expression label must be unique per report line.
Groupby feature isn't supported by '<engine>' engine. Please remove the groupby value on '<report line>'
```

Cross-report aggregation guards:

```
In report '<report>', on line '<line>', with label '<label>',
The format of the cross report expression is invalid.
Expected: cross_report(<report_id>|<xml_id>)
Example:  cross_report(my_module.my_report) or cross_report(123)

In report '<report>', on line '<line>', with label '<label>',
Failed to parse the cross report id or xml_id.

You cannot use cross report on itself
```

---

## 7. Registering a foreign tax identification number

**Actors.** Accounting Manager.

**Preconditions.** The company has a fiscal country. The target country is known.

**Steps.**

1. Create a fiscal position for the company. Give it a name, the target country, and enter the foreign tax identification number.
2. On entering the number, the system normalises and checks it against the target country's format. A malformed number is reported with the message produced by the number checker, naming the record as `fiscal position [<name>]`.
3. Validation runs:
   - the number requires a country: "The country of the foreign VAT number could not be detected. Please assign a country to the fiscal position."
   - a registration in the company's own fiscal country must name at least one country subdivision when that country has subdivisions: "You cannot create a fiscal position with a foreign VAT within your fiscal country without assigning it a state."
   - the country must belong to the country group when both are set: "You cannot create a fiscal position with a country outside of the selected country group."
   - only one distinct number may exist per country per company: "A fiscal position with a foreign VAT already exists in this country."
4. The company's list of foreign registration countries is recomputed as the distinct countries of its fiscal positions carrying a number.
5. The company's list of tax-enabled countries becomes the union of those countries and the fiscal country. Country-specific fields on invoices, contacts and companies are revealed for every country in that list.
6. The fiscal position's foreign registration header mode becomes "Templates Found" when the target country has no tax yet in the system and a template exists for it, which surfaces the "Create foreign taxes" button of section 5.

**Postcondition.** Reports whose availability is Country Matches and whose country is the registered country become available to the company when they permit foreign registration.

---

## 8. Automatic fiscal position detection

**Actors.** Any user creating an invoice, a sales order or a point of sale order.

**Inputs.** The invoicing contact and, optionally, the delivery contact.

**Steps.**

1. When there is no contact, no fiscal position is returned.
2. Determine whether the transaction is intra-union with an identical prefix: both the company and the contact must have a tax identification number, both prefixes must be codes of countries in the European Union country group, and the two prefixes must be equal.
3. When no delivery contact was supplied, or when the transaction is intra-union with an identical prefix and the contact's country equals the company's country, the delivery contact is set to the invoicing contact.
4. A fiscal position set manually on the delivery contact, or failing that on the invoicing contact, wins immediately and is returned.
5. When the invoicing contact has no country, no fiscal position is returned.
6. All fiscal positions of the company marked "Detect Automatically" are fetched and sorted so that positions belonging to the most specific company come first (by the length of the company's parent chain, descending) and then by sequence ascending.
7. The first position that satisfies all five of the following predicates against the delivery contact is returned:

   | Predicate | Satisfied when |
   |---|---|
   | Tax identification number required | the position does not require one, or the contact has a valid one for the company |
   | Postal code range | the position has no range, or the contact's postal code sorts between the range bounds inclusive |
   | Country subdivision | the position lists none, or the contact's subdivision is in the list |
   | Country | the position has none, or the contact's country equals it |
   | Country group | the position has none, or the contact's country is in the group and the contact's subdivision, if any, is not in the group's excluded subdivisions |

8. When no position satisfies all predicates, no fiscal position is returned.

**Postal code normalisation.** When both bounds are set, each numeric bound is right-justified to the length of the longer bound and padded on the left with zero characters. Storage and comparison then use the padded forms.

```
zip_from = "100", zip_to = "9500"  →  zip_from = "0100", zip_to = "9500"
```

**Range validation.** Both bounds must be set together and the upper bound must not sort before the lower bound:

```
Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From".
```

**Effect of the selected fiscal position.**

- Taxes: when the position lists no tax, every tax that is attached to any fiscal position is removed from the line and the rest are kept. When the position lists taxes, each incoming tax is replaced by the taxes of the position that name it as a source tax, or kept unchanged when the position maps nothing for it. Replacements preserve order and remove duplicates.
- Accounts: an account is replaced by the destination account of the mapping whose source account matches, and kept otherwise.

The account mapping is unique per triple of position, source account and destination account:

```
An account fiscal position could be defined only one time on same accounts.
```

### 8.1 Domestic fiscal position

The company's **domestic fiscal position** is derived: among the company's fiscal positions, keep those whose country equals the company country, plus those with no country whose country group contains the company country; sort by country identifier with positions having no country last, then by sequence; the first is the domestic position. It is used by country packages that must distinguish domestic from foreign operations at a glance.

---

## 9. Withholding tax at payment

**Actors.** Accountant.

**Preconditions.** At least one tax of the company is marked "Withhold On Payment". Such a tax has a negative amount, is not of the group or of the tax-included percentage computation type, has exigibility "on invoice" and price inclusion "tax excluded".

### 9.1 Invoicing phase

1. A withholding tax is set on invoice lines exactly like an ordinary tax.
2. The tax engine is instructed to exclude withholding taxes from every computation unless the caller explicitly asks for them. Consequently the invoice total, the tax lines and the journal entry of the invoice contain **no** withholding amount.
3. Product price display shows the withheld amount separately. The displayed string is built by computing the ordinary totals, then computing the withholding amounts with withholding taxes enabled, and joining the non-equal parts:

   ```
   (= <total including taxes> Incl. Taxes, <total excluding taxes> Excl. Taxes, <withheld> Tax Withheld)
   ```

   Each clause is omitted when its amount equals the entered price, and the withheld clause is omitted when the withheld amount is zero.

### 9.2 Registering the payment

1. The user opens the payment registration wizard on one or more invoices.
2. The wizard decides whether to show the withholding section. It is shown when the company has at least one withholding tax whose scope matches the payment direction, and when the wizard will create a single entry. The direction mapping is: an outbound payment matches purchase-scope taxes and an inbound payment matches sale-scope taxes; when any selected line is a refund the direction is inverted before the mapping.
3. On first opening, the wizard computes its withholding lines from the invoices in the first batch:
   1. Take the rounded base lines of every invoice in the batch.
   2. Re-prepare each base line with withholding taxes enabled and no tax filter.
   3. Compute the tax details and round them.
   4. Aggregate the results by the grouping key `(name, analytic distribution, account, tax, skip, currency)` where the name is the tax's name (or an explicit manual name), the account is the company's withholding tax base account when set and otherwise the base line's account, and `skip` is true for any tax that is not a withholding tax.
   5. For each aggregated key that is not skipped, update the matching existing line or create a new one with the source base amount, the source base amount in company currency, the negated source tax amount, and the negated source tax amount in company currency.
   6. Existing lines whose key no longer appears are deleted. When more than one existing line shares a key, the extras are deleted and the first is kept.
4. The user may edit each line: the tax, the base amount, the withheld amount, the account, the analytic distribution and the withholding number.
5. The wizard shows a net amount:

   ```
   net_amount = payment_amount − Σ withholding_line.amount
   ```

6. The wizard requires an outstanding account when the payment method line has no payment account. A default is proposed by looking up the most recent payment that used the same payment method line, whose method line has no payment account, and that has an outstanding account.
7. On confirmation:
   - a negative net amount is refused: "The withholding net amount cannot be negative.";
   - when an outstanding account was chosen, it is written on the payment and, when it is neither a cash account nor a credit card account nor an off-balance account and is not yet reconcilable, it is made reconcilable;
   - every wizard line is copied onto the payment as a stored withholding line, dropping the wizard link and the placeholder value.

### 9.3 Numbering

Each withholding tax may carry a numbering series. Lines display a placeholder reflecting what the series would give them:

| Condition | Placeholder type |
|---|---|
| no number entered and the tax has a series | Given By the Sequence |
| a number is entered | Given By the Name |
| no number and no series | Not defined |

When any line's placeholder type changes, all lines are re-numbered for display: lines are grouped by the series they would consume, and within a group the `n`-th line shows the value the series would produce at its current next value plus `n`. Lines without a series show no placeholder.

At the moment the journal items are prepared, every line without a number and without a series aborts the operation before any series is consumed:

```
Please enter the withholding number for the tax <tax name>
```

Then each line without a number consumes its series and stores the produced number.

### 9.4 Validation

| Rule | Message |
|---|---|
| The base amount must be strictly positive in the line currency. | "The base amount of a withholding tax line must be above 0." |
| The account must not be a liquidity account of the payment, nor the company's inter-banks transfer account. Liquidity accounts are the journal default account, the payment method line's payment account, every payment account of the journal's inbound and outbound method lines, and the payment's outstanding account. | "The account \"<account>\" is not valid to use on withholding lines." |
| A withholding tax may not use the group or the tax-included percentage computation. | "Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations." |
| All lines passed to journal item preparation must belong to one payment. | "All withholding lines in self must have the same payment." |
| All wizard lines passed to journal item preparation must belong to one wizard. | "All withholding lines in self must have the same payment register." |

### 9.5 State effects

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| Payment in draft, no withholding | User ticks Withhold Tax Amounts | at least one withholding tax matches the direction | Payment in draft with withholding | Withholding line table becomes editable |
| Payment in draft with withholding | User edits a line | none | same | Placeholders recomputed; base and withheld amounts recomputed |
| Payment in draft with withholding | User posts the payment | every line has a number or a series; every base amount positive; every account valid | Payment posted | Journal entry contains the outstanding line net of withholding, the counterpart line at gross, one tax line per withholding tax, and a pair of base and base-counterpart lines per aggregated base |

The resulting journal entry is specified in [accounting-effects.md](accounting-effects.md).

---

## 10. Electronic invoicing: the common message-level cycle

Country packages differ in payload format, transport and vocabulary, but almost all follow the same six-phase cycle. The per-country specialisations are in the country files.

### 10.1 Phases

| Phase | What happens | Records written |
|---|---|---|
| Eligibility | The invoice is examined for country, journal, contact, taxes, document type and company registration. A non-eligible invoice is simply not offered for transmission. | none |
| Validation | Blocking and non-blocking checks run over the invoice and its contacts. Blocking problems prevent transmission and are shown grouped by cause. | none |
| Build | A payload is produced from the invoice. It is attached to the invoice, and in most countries also recorded on a dedicated document record. | Attachment; country document record in state "to send" |
| Transmit | The payload is handed to the transport: a government portal, an accredited intermediary, or a document exchange network. A transmission identifier is stored. | Country state field set to sent or processing; transmission identifier stored |
| Poll | A scheduled job asks the transport for the outcome of every pending document. | Country state advanced to accepted, rejected or registered; errors stored; registration number and any visual code stored |
| Answer | On acceptance the invoice is annotated with the registration number, the visual code and any legally required stamp, and the document becomes printable in its final form. On rejection the errors are posted in the discussion thread and the invoice returns to a correctable state. | Discussion thread message; error fields |

### 10.2 Generic state machine for a country exchange state

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| (none) | Post the invoice | invoice is eligible | To Send | Payload may be built immediately or on demand |
| To Send | Send | no blocking validation error | Sent / Processing | Payload attached, transmission identifier stored |
| To Send | Send | blocking validation error | To Send | Error message shown, nothing transmitted |
| Sent / Processing | Poll returns acceptance | none | Accepted / Registered | Registration number, visual code and stamp stored; confirmation posted in the discussion thread |
| Sent / Processing | Poll returns rejection | none | Rejected / Error | Errors stored and posted; invoice may be reset to draft where the country allows it |
| Sent / Processing | Poll times out | country-defined timeout | Timeout | The document is retried by the scheduled job |
| Accepted / Registered | Request cancellation | country allows cancellation and the cancellation window is open | Cancellation Requested | Cancellation payload transmitted with a reason code and remarks |
| Cancellation Requested | Poll returns acceptance | none | Cancelled | Invoice annotated; a credit note is usually required in addition |
| Rejected / Error | Correct and resend | the invoice is back in a modifiable state | To Send | New payload built |

### 10.3 Common guards

1. **Locking.** A document being transmitted is locked so that two processes cannot send it at once. A second attempt is refused with a message of the form "This document is being sent by another process already."
2. **Immutability after acceptance.** Once accepted, the invoice's number, date, contact, lines and taxes may not change. Countries implement this through the ordinary posting lock plus a country constraint.
3. **Chaining.** Several countries require each document to carry a reference to the previous document of the same company, forming a hash chain. The chain index is stored on the document record and copied onto the invoice.
4. **Deletion.** A transmitted document may not be deleted. Attempts are refused with a country-specific message, for example "You cannot delete a generated E-waybill. Instead, you should cancel it." and "You cannot delete sent flows."

---

## 11. Preparing and closing a tax return

**Actors.** Accounting Manager.

**Preconditions.** A report exists whose availability matches the company, either through the fiscal country or through a foreign registration.

**Steps.**

1. Choose the period. The default period comes from the report's default opening date filter. Periodic returns use the return period, whose length is a company setting (monthly, two-monthly, quarterly, four-monthly, half-yearly, yearly).
2. Choose the scope: a single company, the branch tree, or a tax unit grouping several companies under one registration, depending on the report's multi-company filter.
3. Choose a foreign registration when the report permits it: the report is then computed for the journal items tagged with that registration's country.
4. Run the report. Each line's expressions are evaluated by their engines, aggregation expressions being expanded into their full dependency set first.
5. Review the figures. Every auditable expression can be expanded into the journal items behind it.
6. Enter or adjust manual figures where the report has external-value expressions.
7. Close the period. Closing writes a journal entry that moves the balances of the tax accounts into the tax payable or tax receivable account of each tax group, and stores any carryover as an external value dated at the end of the period. The entry is described in [accounting-effects.md](accounting-effects.md).
8. Export or transmit the return in the country's required form.

---

## 12. Installing and removing a country package

### 12.1 Installing

1. The administrator installs a country capability package, or the system installs it automatically as a consequence of setting the company's country, or of selecting one of its templates.
2. The package's shipped configuration records (document types, identification types, tax offices, classification codes, report structures, report tags) are created. These records are not company scoped and are shared by every company.
3. When the package contributes templates and the company has none, the first template of the package whose country matches the company is scheduled for loading.
4. Fields added by the package appear on invoices, contacts, companies and other hosts, subject to their visibility conditions, which are almost always "the company's fiscal country, or one of its foreign registration countries, is this country".

### 12.2 Removing

1. Every company whose template code belongs to the package has its template code cleared.
2. Records created by the package that are still referenced are protected by ordinary deletion rules. In particular the chart of accounts instantiated for a company is *not* removed, because its accounts carry posted journal items.

---

## 13. Point of sale localization pattern

Countries impose three kinds of requirement on point of sale, and the packages implement them with the same three patterns.

### 13.1 Fiscal receipt numbering

1. The point of sale configuration is linked to one or more document types or numbering series.
2. On validating an order, the order consumes the series attached to its configuration and stores the produced number on the order and on the generated journal entry.
3. The number is printed on the receipt together with the legally required wording.

### 13.2 Certification and inalterability

1. Each order stores a hash computed over its own immutable fields and the hash of the previous order of the same company, forming a chain.
2. Orders and their lines become non-modifiable and non-deletable once hashed. Attempts are refused.
3. A closing record is produced daily, monthly and annually by a scheduled job. Each closing stores the interval total, the cumulative grand total since the beginning, a sequence number, the last order included and that order's hash. Closings are immutable:

   ```
   Sale Closings must never be modified or deleted under any circumstances.
   ```

4. An integrity report recomputes the chain and reports the first divergence.

### 13.3 Electronic receipt transmission

1. The point of sale order is converted to the country's payload, usually reusing the invoice builder with a simplified-invoice profile.
2. Transmission happens per order, or in a consolidated batch covering a period for orders below the country's threshold.
3. The returned registration number and visual code are stored on the order and reprinted.

**Simplified invoice threshold.** Countries that allow a simplified receipt below an amount configure that amount on the point of sale configuration. Above the threshold, the customer's identification becomes mandatory and a full invoice is issued instead.

---

## 14. Website sales localization pattern

1. The checkout form is extended with the country's mandatory identification fields, for example the identification type and number, the responsibility type, or the address subdivisions the country requires.
2. The fields are validated on submission with the same checkers used on the contact form, and the same messages are shown.
3. The confirmed order carries the fields onto the invoice.
4. Where the country requires it, a website setting selects which contact classes may buy (for example, only final consumers, or any contact class).

---

## 15. Consolidated state machine index

| State field | Owner | Values |
|---|---|---|
| Chart of accounts template code | Company | any template code, or empty |
| Foreign registration header mode | Fiscal Position | Templates Found, No Template, empty |
| Placeholder type | Withholding Line | Given By the Sequence, Given By the Name, Not defined |
| Reporting flow state | France Reporting Flow | Ready, Error, Sent, Completed |
| Reporting period status | France Reporting Flow | Open, Grace, Closed |
| Electronic way bill state | India Electronic Way Bill | Pending, Generated, Cancelled, Challan |
| India exchange status | Journal Entry | To Send, Sent, Cancelled |
| Italy exchange state | Journal Entry | Being Sent, Requires user signature, Processing, Rejected, Accepted and forwarded, Forward failed, Forwarding, Accepted by the public administration, Rejected by the public administration, Accepted after expiry, Accepted with no answer, Other |
| Hungary exchange state | Journal Entry | Sent waiting for response, Timeout when sending, Confirmed, Confirmed with warnings, Rejected, Cancellation request sent, Timeout when requesting cancellation, Cancellation request pending, Cancelled |
| Basque Country state | Journal Entry | To Send, Sent, Cancelled |
| Verifiable invoice state | Journal Entry | Rejected, Registered with Errors, Accepted, Cancelled |
| Greece exchange state | Journal Entry | Invoice sent, Expense classification ready to send, Expense classification sent |
| Denmark exchange state | Journal Entry | Ready to send, Queued, Pending Reception, Done, Error |
| Malaysia document state | Malaysia Interchange Document | In progress, Valid, Invalid, Cancelled, Rejected |
| Romania document state | Romania Interchange Document | Sent, Accepted, Error |
| Indonesia document state | Indonesia Electronic Invoice Document | In progress, Done, Failed |
| France portal invoice status | Journal Entry | In Progress, Sent, Done, Error |
| France portal lifecycle status | Journal Entry | In Progress, Sent, Done, Error |

Each of these is fully specified in the corresponding country file.
