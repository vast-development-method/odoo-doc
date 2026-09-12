# Fiscal Localizations: Business Rules

The complete catalog of validations, guards, permissions, consistency rules, uniqueness rules, rounding rules, date rules and error messages that belong to this domain. Rules are numbered with the prefix `FLOC` so other documents can cite them. Messages are reproduced exactly as the user sees them, with product names removed. Rules whose origin is a general accounting principle rather than an observed behavior are marked **industry-standard completion**.

---

## 1. Template registry and selection

**FLOC-RULE-001. A template code is unique across all capability packages.** Two packages may not contribute the same template code. The last package evaluated wins silently if they do, which is why every country package prefixes its codes with its country code.

**FLOC-RULE-002. Shared base templates may not be selected directly.** The West African harmonised accounting system template and its non-profit variant are bases shared by sixteen countries and are never a company's own template. Attempting to load one whose code differs from the company's current template is refused:

```
The <template code> chart template shouldn't be selected directly. Instead, you should directly select the chart template related to your country.
```

**FLOC-RULE-003. A template that is not visible is hidden from selection but remains loadable.** Invisible templates exist to serve as parents and to be reached through the parent chain and through the foreign tax instantiation path.

**FLOC-RULE-004. The selection list is ordered by country relevance.** Templates whose country equals the company's country come first. When the company has no country, the generic chart of accounts comes first.

**FLOC-RULE-005. Only a system administrator may load a template.**

```
Only administrators can install chart templates
```

**FLOC-RULE-006. A branch company always uses its parent's template.** A company created under a parent that already has a template has that template loaded automatically at the end of the creating transaction. A load on a parent recurses into every child.

**FLOC-RULE-007. Loading a template on a company that already has posted accounting does not delete the existing configuration.** The purge step runs only when the company's root has no journal items at all, or when demonstration data was explicitly requested.

**FLOC-RULE-008. Changing the template after entries exist is not offered.** The settings screen exposes whether accounting entries already exist and the platform prevents the change. The functional rule is: selecting another package is only possible while no entry has been posted.

**FLOC-RULE-009. Uninstalling a package clears the template code of every company that used one of its templates.** The instantiated accounts, taxes and journals are left in place because they carry posted journal items.

**FLOC-RULE-010. A template load runs in the base language.** All shipped text is stored in the base language and the translations are applied afterwards, so that a load performed by a user working in another language produces the same stored data.

---

## 2. Account codes

**FLOC-RULE-011. Every account code produced by a template is padded on the right with zero characters to the template's code length.** The code length is the `code_digits` value of the template, defaulting to 6.

```
padded_code = code + "0" × max(0, code_digits − length(code))
```

**FLOC-RULE-012. Account codes are unique within a company hierarchy.** A code is available only when no account bearing it belongs to a parent company or a child company of the active company. Violation:

```
Account codes must be unique. You can't create accounts with these duplicate codes: <code>, <code>, ...
```

**FLOC-RULE-013. A utility account created from a prefix takes the first available code under that prefix.** The starting code is built as:

```
start_code = prefix left-justified to (code_digits − 1) with "0", followed by "1"     when length(prefix) < code_digits
start_code = prefix                                                                    otherwise
```

The first available code at or after the starting code is then searched.

**FLOC-RULE-014. Code increment algorithm.** Given a starting code, the first code tried is the starting code itself. When it is taken, the trailing run of digits is incremented, preserving its width, until the width is exhausted. When no digit increment is available, suffixes `.copy`, `.copy2`, `.copy3` up to `.copy99` are tried.

| Starting code | Codes tried in order |
|---|---|
| `102100` | `102100`, `102101`, `102102`, `102103`, ... |
| `1598` | `1598`, `1599`, `1600`, `1601`, ... |
| `10.01.08` | `10.01.08`, `10.01.09`, `10.01.10`, ... |
| `10.01.97` | `10.01.97`, `10.01.98`, `10.01.99`, `10.01.97.copy2`, `10.01.97.copy3`, ... |
| `1021A` | `1021A`, `1022A`, `1023A`, ... |
| `hello` | `hello`, `hello.copy`, `hello.copy2`, ... |
| `9998` | `9998`, `9999`, `9998.copy`, `9998.copy2`, ... |

**FLOC-RULE-015. Changing a liquidity account code prefix on the company rewrites the codes of the matching accounts.** When the bank account code prefix or the cash account code prefix changes, every account of the company whose code starts with the old prefix and whose type is Bank and Cash or Credit Card is renumbered, processing them in ascending code order, with:

```
new_code = new_prefix + strip_leading_zeros(remove_first_occurrence(current_code, old_prefix))
                        right-justified to (length(current_code) − length(new_prefix)) with "0"
```

Worked example: old prefix `1014`, new prefix `5100`, current code `101401`. Removing the old prefix leaves `01`; stripping leading zeros leaves `1`; right-justifying to 6 − 4 = 2 characters gives `01`; the result is `510001`.

**FLOC-RULE-016. An account that already carries journal items cannot be deleted.**

```
You cannot perform this action on an account that contains journal items.
```

**FLOC-RULE-017. An account referenced by a fiscal position account mapping cannot be deleted or archived.**

```
You cannot remove/deactivate the accounts "<code> - <name>" which are set on the account mapping of a fiscal position.
```

**FLOC-RULE-018. An off-balance account may not be reconcilable and may not carry taxes.**

```
An Off-Balance account can not be reconcilable
An Off-Balance account can not have taxes
```

**FLOC-RULE-019. On a reload, an account's reconcilability is never re-imposed.** The `reconcile` column is stripped from every account row before writing, because the user may have changed it and because forcing it could raise a partial reconciliation error.

**FLOC-RULE-020. On a reload, an existing account receives only its report tags.** Code, name, type and every other column are ignored for an account that already exists.

**FLOC-RULE-021. On a reload, an unbound account whose code matches the template's code followed by any number of zero characters is adopted.** Among candidates, an exact match with the padded code is preferred. The adopted account is bound to the template's external identifier for that company and marked as not to be updated.

---

## 3. Account groups

**FLOC-RULE-022. The starting and ending code prefixes of an account group must have the same length.**

```
The length of the starting and the ending code prefix must be the same
```

**FLOC-RULE-023. An account group's ending prefix defaults to its starting prefix, and vice versa.** When only one is given, or when the ending prefix sorts before the starting prefix, the missing or inconsistent one is replaced by the other.

**FLOC-RULE-024. An account belongs to the account group with the longest matching prefix.** The match is `group.code_prefix_start ≤ left(account.code, length(group.code_prefix_start))` and `group.code_prefix_end ≥ left(account.code, length(group.code_prefix_end))`; ties are broken by the longer starting prefix and then by the lower group identifier.

**FLOC-RULE-025. On a reload, account groups are not written at all when the company already has at least one.**

---

## 4. Taxes and tax groups from templates

**FLOC-RULE-026. A tax created by a template without explicit repartition instructions receives the engine's default repartition.** When the repartition instructions must be deferred to a second pass, a clearing instruction is prepended so that the defaults are removed before the template's own instructions are applied.

**FLOC-RULE-027. A report tag named on a template repartition instruction must already exist for the template's country.** Otherwise the load is aborted:

```
Error while loading the localization: missing tax tag <tag name> for country <country name>. You should probably update your localization app first.
```

The user is offered a button "Update app" leading to the package browser filtered on the country name and the accounting localization charts category.

**FLOC-RULE-028. A tag name that matches `package.identifier` and whose package part names an existing capability package is treated as an explicit external identifier and is not looked up by name.**

**FLOC-RULE-029. Tag names are normalised before lookup**: leading and trailing whitespace is removed and internal runs of whitespace are collapsed to a single space.

**FLOC-RULE-030. A leading hyphen on a tag name selects the negative variant of the tag.**

**FLOC-RULE-031. On a reload, a tax is considered materially changed when its computation type differs, its amount differs at four decimal places, or the number of non-clearing repartition instructions is neither zero nor equal to the number of repartition lines it already has.**

**FLOC-RULE-032. A materially changed tax is retired by renaming, never by deleting.** The retired tax keeps its journal items and its history. Names are `[old] <name>`, then `[old1] <name>`, `[old2] <name>` and so on, chosen so that the name stays unique within the tuple of name, scope, product applicability and company.

**FLOC-RULE-033. On a reload, a materially unchanged tax has only its fiscal position links, its source tax links and its report tags refreshed.** Amount, computation type, accounts, exigibility and every other property are left untouched.

**FLOC-RULE-034. On a reload, a tax group that already exists keeps the tax payable and tax receivable accounts it already has.**

**FLOC-RULE-035. Default company taxes are only set when absent.** After a load, the default sale tax becomes the first tax of the company whose scope is sale or all, and the default purchase tax the first tax whose scope is purchase or all, but only when the company had none.

**FLOC-RULE-036. Default taxes are forced onto products that already carry a tax from another company.** A product with no tax at all keeps none, because several flows require a taxless product.

**FLOC-RULE-037. The cash basis flag of a top-level company is switched on when the load produced at least one tax with exigibility "on payment".**

---

## 5. Fiscal positions

**FLOC-RULE-038. A postal code range requires both bounds and an ordered pair.**

```
Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From".
```

**FLOC-RULE-039. Numeric postal code bounds are left-padded with zero characters to a common length** before storage and comparison, so that a range from `100` to `9500` compares as `0100` to `9500`.

**FLOC-RULE-040. A foreign tax identification number on a fiscal position requires a country.**

```
The country of the foreign VAT number could not be detected. Please assign a country to the fiscal position.
```

**FLOC-RULE-041. A foreign registration inside the company's own fiscal country requires at least one country subdivision**, when the fiscal country has subdivisions.

```
You cannot create a fiscal position with a foreign VAT within your fiscal country without assigning it a state.
```

**FLOC-RULE-042. A fiscal position that names both a country and a country group requires the country to belong to the group.**

```
You cannot create a fiscal position with a country outside of the selected country group.
```

**FLOC-RULE-043. A company may hold at most one distinct foreign tax identification number per country.**

```
A fiscal position with a foreign VAT already exists in this country.
```

**FLOC-RULE-044. A foreign tax identification number is normalised and format-checked against its country** on entry and on save, using the same checker as a contact's number, with the record named `fiscal position [<name>]` in the resulting message.

**FLOC-RULE-045. Automatic detection evaluates five predicates and returns the first fiscal position satisfying all of them**: tax identification number requirement, postal code range, country subdivision, country, and country group with its subdivision exclusions.

**FLOC-RULE-046. Detection order is most-specific company first, then sequence ascending.** Specificity is measured by the length of the fiscal position's company parent chain.

**FLOC-RULE-047. A fiscal position set manually on the delivery contact, or failing that on the invoicing contact, overrides automatic detection.**

**FLOC-RULE-048. For an intra-union transaction between two parties whose tax identification numbers share the same country prefix, and whose contact country equals the company country, the invoicing address is used instead of the delivery address.**

**FLOC-RULE-049. A fiscal position with no tax mapping removes every tax that is attached to any fiscal position and keeps the rest.** A fiscal position with tax mappings replaces each incoming tax by the taxes that name it as a source tax, preserving order and removing duplicates.

**FLOC-RULE-050. An account mapping triple is unique.**

```
An account fiscal position could be defined only one time on same accounts.
```

**FLOC-RULE-051. On a reload, only account mappings that introduce a new account are applied.** A mapping whose source and destination both already resolve is skipped.

---

## 6. Journals from templates

**FLOC-RULE-052. Journal codes are translated even though the field is not translatable.** The target language is the company contact's language, or the session language when the contact has none. The explicit translation column wins; otherwise the compiled text catalogue of the contributing package, then of the accounting package, is consulted.

**FLOC-RULE-053. On a reload, an existing journal is adopted rather than duplicated.** Matching is by code first, then by the pair of type and name, using either the shipped name or its translation. The adopted journal is bound to the template's external identifier and marked as not to be updated.

**FLOC-RULE-054. The six journals every template creates** are Sales, Purchases, Miscellaneous Operations, Exchange Difference, Cash Basis Taxes and Bank, with the codes, colours, dashboard visibility and ordering listed in [configuration.md](configuration.md). A country package may add to or override this set.

---

## 7. Company defaults after a load

**FLOC-RULE-055. The company currency is set from the fiscal country's currency**, or from the parent company's currency for a subsidiary, and only while the company's root has no accounting entries.

**FLOC-RULE-056. The company currency cannot be changed once journal items exist.**

```
You cannot change the currency of the company since some journal items already exist
```

**FLOC-RULE-057. The default sales price tax inclusion cannot be changed once invoicing has started.**

```
Cannot change Price Tax computation method on a company that has already started invoicing.
```

**FLOC-RULE-058. The cost accounting flag defaults to false** when the template does not set it, so that moving from a template that enables it to one that does not clears it.

**FLOC-RULE-059. Default receivable and payable accounts are recorded as company-scoped default values on Contact, not as fields of the company.** The same mechanism records the default stock journal on Product Category and any additional properties the template declares.

**FLOC-RULE-060. On a reload, the company-level property defaults are not re-imposed.**

**FLOC-RULE-061. Every company must have an account of type "Current Year Earnings".** When none exists after a load, the user is redirected:

```
We cannot find a chart of accounts for this company, you should configure it.
Please go to Account Configuration and select or install a fiscal localization.
```

**FLOC-RULE-062. The fiscal country defaults to the company country** and is thereafter independently editable. Every country-specific field's visibility is driven by the fiscal country and by the countries of the company's foreign registrations.

**FLOC-RULE-063. Storno accounting is forced on for the countries where negative-amount reversals are mandatory and offered as an option where they are permitted.** Outside those two sets the option is hidden.

**FLOC-RULE-064. A localization may force the restrictive audit trail.** When it does, the option cannot be switched off:

```
Can't disable restricted audit trail: forced by localization.
```

---

## 8. Financial reports

**FLOC-RULE-065. A report used as a root report may not itself have a root report.**

```
Only a report without a root report of its own can be selected as root report.
```

**FLOC-RULE-066. A section of a composite report may not itself have sections.**

```
The sections defined on a report cannot have sections themselves.
```

**FLOC-RULE-067. Availability "Country Matches" requires a country.**

```
The Availability is set to 'Country Matches' but the field Country is not set.
```

Changing the availability away from "Country Matches" clears the country.

**FLOC-RULE-068. A parent line must appear before its children in sequence order.**

```
Line "<line>" defines line "<parent line>" as its parent, but appears before it in the report. The parent must always come first.
```

**FLOC-RULE-069. A line may not be its own parent.**

```
Line "<line>" defines itself as its parent.
```

**FLOC-RULE-070. A line may not have both children and a grouping.**

```
A line cannot have both children and a groupby value (line '<parent name>').
```

**FLOC-RULE-071. A line code is unique within a report.**

```
A report line with the same code already exists.
```

**FLOC-RULE-072. An expression label is unique within a line.**

```
The expression label must be unique per report line.
```

**FLOC-RULE-073. The aggregation and external engines do not support grouping.**

```
Groupby feature isn't supported by '<engine>' engine. Please remove the groupby value on '<report line>'
```

**FLOC-RULE-074. A domain expression must have a subformula.** Database check: `engine ≠ "domain" OR subformula IS NOT NULL`, message "Expressions using 'domain' engine should all have a subformula."

**FLOC-RULE-075. A formula must be syntactically valid for its engine.** Domain formulas must parse as a condition expression and be accepted by a journal item search. Account code formulas must split into terms each of which has a non-empty prefix. Aggregation formulas must match the arithmetic grammar or be the reserved word `sum_children`. Failure message:

```
Invalid formula for expression '<label>' of line '<line name>': <formula>
```

**FLOC-RULE-076. Formulas are whitespace-normalised on write**: leading and trailing whitespace removed, internal runs collapsed to single spaces.

**FLOC-RULE-077. A cross-report reference must name a different report and must be resolvable.**

```
In report '<report>', on line '<line>', with label '<label>',
The format of the cross report expression is invalid.
Expected: cross_report(<report_id>|<xml_id>)
Example:  cross_report(my_module.my_report) or cross_report(123)

In report '<report>', on line '<line>', with label '<label>',
Failed to parse the cross report id or xml_id.

You cannot use cross report on itself
```

**FLOC-RULE-078. A carryover source expression's label must begin with `_carryover_`.**

```
You cannot use the field carryover_target in an expression that does not have the label starting with _carryover_
```

**FLOC-RULE-079. A carryover target expression's label must begin with `_applied_carryover_`.**

```
When targeting an expression for carryover, the label of that expression must start with _applied_carryover_
```

**FLOC-RULE-080. A carryover target that cannot be determined aborts the computation.**

```
Could not determine carryover target automatically for expression <label>.
```

**FLOC-RULE-081. A report that has variants cannot be deleted.**

```
You can't delete a report that has variants.
```

**FLOC-RULE-082. Deleting a tax-tag expression archives its tags when journal items use them and deletes them otherwise**, after unlinking them from every tax repartition line.

**FLOC-RULE-083. Renaming a tax-tag expression renames the tag only when every expression using that tag is part of the same change**; otherwise a new tag is created.

**FLOC-RULE-084. Changing a report's country moves its tags to the new country only when no other report uses them**; otherwise new tags are created in the target country.

**FLOC-RULE-085. Copying a report renames every line code with the suffix `_COPY`, repeated until unique, and rewrites every aggregation formula and subformula accordingly.** The report name receives the suffix ` (copy)`, repeated until unique.

---

## 9. Withholding taxes

**FLOC-RULE-086. A withholding tax may not use the group computation or the tax-included percentage computation.**

```
Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations.
```

**FLOC-RULE-087. Marking a tax as withheld on payment forces exigibility to "on invoice" and price inclusion to "tax excluded".**

**FLOC-RULE-088. A tax with a non-negative amount cannot be a withholding tax.** Setting the amount to zero or above clears the withholding flag, and the flag is hidden.

**FLOC-RULE-089. Withholding taxes are excluded from every ordinary tax computation.** The invoice total, the tax lines and the invoice journal entry ignore them. Only a caller that explicitly requests withholding computation sees them.

**FLOC-RULE-090. A withholding tax's label in printed documents is its invoice label, never its name.** An empty invoice label produces no label, which lets a country hide withholding taxes from the invoice.

**FLOC-RULE-091. A withholding line's base amount must be strictly positive in the line currency.**

```
The base amount of a withholding tax line must be above 0.
```

**FLOC-RULE-092. A withholding line's account may not be a liquidity account of the payment nor the inter-banks transfer account.** The liquidity set is the journal's default account, the payment method line's payment account, every payment account of the journal's inbound and outbound payment method lines, and the payment's outstanding account.

```
The account "<account>" is not valid to use on withholding lines.
```

**FLOC-RULE-093. A withholding line without a number and without a numbering series blocks the preparation of the journal items**, before any series is consumed.

```
Please enter the withholding number for the tax <tax name>
```

**FLOC-RULE-094. The net amount of a payment with withholding may not be negative.**

```
The withholding net amount cannot be negative.
```

**FLOC-RULE-095. All withholding lines prepared together must belong to one payment, or to one payment registration wizard.**

```
All withholding lines in self must have the same payment.
All withholding lines in self must have the same payment register.
```

**FLOC-RULE-096. An outstanding account chosen for a payment with withholding is made reconcilable** when it is not a cash account, not a credit card account, not an off-balance account, and not already reconcilable.

**FLOC-RULE-097. The withholding section is hidden when the payment would create more than one journal entry.** A registration that cannot be edited as one payment, or that groups payments while grouping is disabled, offers no withholding.

**FLOC-RULE-098. For a refund, the payment direction is inverted before matching withholding taxes.** An inbound payment on a refund matches purchase-scope taxes and an outbound payment on a refund matches sale-scope taxes.

**FLOC-RULE-099. Withholding line numbering placeholders are recomputed for the whole set whenever any line's numbering source changes**, so that consecutive lines drawing on the same series display consecutive numbers.

---

## 10. Document types and numbering (Latin American pattern)

**FLOC-RULE-100. A journal may be marked as using document types.** Once a validated invoice exists in the journal, the marker cannot be changed:

```
You can not modify the field "Use Documents?" if there are validated invoices in this journal!
```

**FLOC-RULE-101. An invoice in a journal that uses document types must carry a document type and a document number when posted.**

```
The journal require a document type but not document type has been selected on invoices <numbers>.
Please set the document number on the following invoices <numbers>.
```

**FLOC-RULE-102. A document type whose internal type is "debit note" or "invoice" may not be used on a credit note, and a document type whose internal type is "credit note" may not be used on an invoice.**

```
You can not use a <type> document type with a refund invoice
You can not use a <type> document type with a invoice
```

Country packages may carve out exceptions; Argentina's internal document type 99 is one.

**FLOC-RULE-103. Only invoice-class entries may be posted in a journal that uses document types.**

```
The selected Journal can't be used in this transaction, please select one that doesn't use documents as these are just for Invoices.
```

**FLOC-RULE-104. A document type's numbering series is per journal and per document type.** A journal that uses document types holds one numbering series for each document type it is allowed to issue.

---

## 11. Point of sale certification

**FLOC-RULE-105. A certified point of sale order is immutable once hashed.** Modification and deletion of the order and of its lines are refused.

**FLOC-RULE-106. A sale closing may never be written or deleted.**

```
Sale Closings must never be modified or deleted under any circumstances.
```

**FLOC-RULE-107. An integrity check recomputes the whole chain and reports the first divergence.** A company for which certification is not in force is reported as such:

```
Accounting is not unalterable for the company <company>. This mechanism is designed for companies where accounting is unalterable.
```

**FLOC-RULE-108. Only an accounting user may print the integrity result.**

```
Please contact your accountant to print the Hash integrity result.
```

---

## 12. Electronic invoicing

**FLOC-RULE-109. A document already being transmitted cannot be transmitted again concurrently.** The transmission takes an exclusive lock and a second attempt is refused, for example:

```
This document is being sent by another process already.
```

**FLOC-RULE-110. A transmitted document may not be deleted.** Examples of the country messages:

```
You cannot delete a generated E-waybill. Instead, you should cancel it.
You cannot delete sent flows.
```

**FLOC-RULE-111. Blocking validation errors prevent transmission and are presented together.** Each country's builder collects its constraints, keyed so that the same problem reported by several lines is shown once.

**FLOC-RULE-112. A country flow may block resetting an invoice to draft once transmitted.** Examples:

```
Cannot reset to draft or cancel invoice <number> because an electronic document was already sent to NAV!
You cannot reset this invoice to draft.
```

**FLOC-RULE-113. Cancelling a transmitted document requires a reason code and, in several countries, free-text remarks**, and is only possible inside the country's cancellation window.

**FLOC-RULE-114. A country that requires document chaining stores, on every document, the index of the document in the chain and the fingerprint of the previous one.** The index is copied onto the invoice for display. A gap in the chain is an integrity failure.

---

## 13. Foreign tax instantiation

**FLOC-RULE-115. Foreign taxes are only instantiated when the target country has no tax at all in the company.** Otherwise the operation returns silently.

**FLOC-RULE-116. Accounts created for foreign taxes are copies of the closest local equivalent**, placed at the first free code after the local account's code, and named `<local account name> - <purpose> (<country code>)`.

**FLOC-RULE-117. When no local equivalent can be found, the account is left empty rather than filled with a wrong account.**

**FLOC-RULE-118. Foreign taxes carry the target country, are grouped under tax groups whose identifiers are prefixed with the source template code, and are attached to the fiscal position that triggered the instantiation.**

**FLOC-RULE-119. Fiscal position links and source tax links from the foreign template are discarded.** The foreign template's own fiscal positions are not created and the substitution relationships cannot be derived.

**FLOC-RULE-120. Instantiated foreign taxes are an accelerator, not a certified configuration.** The resulting set must be reviewed before use. **Industry-standard completion**: the reviewer should confirm the rate, the reporting tags, the accounts and the exemption handling against the target country's rules before issuing any document.

---

## 14. Company, contact and invoice field visibility

**FLOC-RULE-121. A country-specific field is visible only when the country is enabled for the company.** The enabled set is the fiscal country plus every country in which the company holds a foreign registration.

**FLOC-RULE-122. A country-specific field that is mandatory for posting is enforced by a posting constraint, never by a field-level requirement**, so that a draft document can be saved incomplete.

**FLOC-RULE-123. A country-specific identification number is validated by the country's checker on entry and again on save.** The two passes differ: the on-entry pass normalises without raising, the on-save pass raises.

**FLOC-RULE-124. A country package that adds a mandatory contact field also adds it to the online checkout form and validates it with the same checker and the same message.**

---

## 15. Tax period and closing

**FLOC-RULE-125. The tax return period length is a company setting**, with the values monthly, two-monthly, quarterly, four-monthly, half-yearly and yearly, and a starting month.

**FLOC-RULE-126. Closing a tax period writes one journal entry that empties the tax accounts into the tax group's payable or receivable account.** The entry is dated on the last day of the period.

**FLOC-RULE-127. A closed period is protected by the tax lock date.** Journal items dated on or before the tax lock date may not be created, modified or deleted, except under an explicit lock exception.

**FLOC-RULE-128. Carryover amounts are stored as external values dated at the end of the period they arose in**, and are read by the `_applied_carryover_` expression of the following period.

**FLOC-RULE-129. A report run under a foreign registration considers only the journal items whose fiscal position is that registration.** **Industry-standard completion**: journal items with no fiscal position are treated as domestic and are excluded from a foreign registration return.

---

## 16. Permissions

| Operation | Required access group |
|---|---|
| Load or reload a chart of accounts template | System Administrator |
| Change the company's package in settings | Accounting Adviser (settings access) |
| Create or edit a fiscal position, including foreign registration | Accounting Adviser |
| Instantiate foreign taxes | Accounting Adviser |
| Create or edit a financial report, its lines, expressions and columns | Accounting Adviser |
| Enter a financial report external value | Accounting Adviser |
| Read a country reference table (document types, identification types, tax offices, classification codes) | Any internal user |
| Create or edit a country reference table | Accounting Adviser |
| Create, edit or delete a withholding line | Accounting User |
| Transmit an electronic document | Accounting User |
| Cancel a transmitted electronic document | Accounting Adviser |
| Print the certification integrity report | Accounting User |
| Read a country document record (exchange history) | Accounting User |

**Industry-standard completion**: no group other than the System Administrator may run the template purge step, because it deletes configuration records across the whole company tree.

---

## 17. Currency and company consistency

**FLOC-RULE-130. Every record a template creates belongs to the company being loaded**, or, for accounts, to a company list containing it.

**FLOC-RULE-131. A withholding line's company is taken from its payment or wizard, never entered.** Its currency is the payment's currency, and its company currency is the company's.

**FLOC-RULE-132. A withholding line converts its source amounts into its own currency with the rate of the payment date**, using the four cases of [calculations.md](calculations.md) section 6.

**FLOC-RULE-133. A country reference table that is not company scoped is shared by every company** and may therefore not be edited to fit one company's needs. Country packages that need per-company variation add a company field explicitly.

---

## 18. Date rules

**FLOC-RULE-134. A dated authorisation (a partner withholding authorisation, a declaration of intent, a certificate validity) requires the start date to precede the end date.** Example message:

```
"From date" must be lower than "To date" on Withholding (AR) taxes.
```

**FLOC-RULE-135. A document is matched to a period by its accounting date, not by its issue date.**

**FLOC-RULE-136. A reporting flow has three period states.** Open before the grace window, Grace inside the sending window, Closed after the deadline. Only a flow in Grace may be sent on its normal path.

**FLOC-RULE-137. A validity window that has expired blocks the use of the authorisation at posting time**, with the country's message.

---

## 19. Rounding

**FLOC-RULE-138. The tax computation rounding method is a company setting seeded by the template.** Its two values are "Round per Tax" (globally) and "Round per Line". The template writes it when it declares it and the platform default is "Round per Tax".

**FLOC-RULE-139. Every monetary amount produced by this domain is rounded to the decimal places of its own currency field**, using half-up rounding away from zero. **Industry-standard completion** for the tie-breaking direction.

**FLOC-RULE-140. A report column declared as an integer figure applies the report's integer rounding setting**: Nearest, Up or Down.

**FLOC-RULE-141. A withholding line's withheld amount is recomputed as a proportion of its base and is rounded once, at the end**, never by rounding the ratio first.

---

## 20. Uniqueness constraints introduced by country packages

| Constraint | Scope | Message |
|---|---|---|
| Responsibility type name | Argentina | `Name must be unique!` |
| Responsibility type code | Argentina | `Code must be unique!` |
| Postal code range start | Brazil | `The "from" zip must be unique` |
| Postal code range end | Brazil | `The "to" zip must be unique.` |
| Territorial workplace code | Czech Republic | `The territorial workplace code must be unique` |
| Signing device per user and company | Egypt | `You can only have one thumb drive per user per company!` |
| Document type code | Italy | `Document Type code must be unique.` |

---

## 21. Rule index by workflow

| Workflow | Rules |
|---|---|
| Template selection | 001 to 010 |
| Template loading, accounts | 011 to 025 |
| Template loading, taxes | 026 to 037 |
| Template loading, fiscal positions | 038 to 051 |
| Template loading, journals and company | 052 to 064 |
| Report definition and computation | 065 to 085 |
| Withholding at payment | 086 to 099 |
| Document types and numbering | 100 to 104 |
| Point of sale certification | 105 to 108 |
| Electronic invoicing | 109 to 114 |
| Foreign tax instantiation | 115 to 120 |
| Field visibility and validation | 121 to 124 |
| Tax period and closing | 125 to 129 |
| Permissions and consistency | 130 to 133 |
| Dates | 134 to 137 |
| Rounding | 138 to 141 |

Country-specific posting constraints are catalogued in [country-packages.md](country-packages.md) and specified in full in the per-country files.
