# Fiscal Localizations: Acceptance Criteria

Numbered scenarios that a rebuild must satisfy. Each one gives concrete starting records, a
concrete operation with concrete inputs, and the exact resulting records, amounts and states.

**How to read a scenario.** "Given" lists the records that exist before the operation and the
values that matter. "When" is one operation with its inputs. "Then" is the complete observable
result: records created or changed, amounts to the last decimal, states with their stored value,
and the exact text of any message.

**Currencies.** Where a scenario does not name a currency it uses a currency with two decimal
places, and the company currency is that currency. Where two currencies are involved they are
named "the company currency" and "the foreign currency" and the rate is given.

**Coverage.** Sections 1 to 14 cover the framework: the template registry, loading, reloading,
account codes, groups, taxes, fiscal positions, journals, company defaults, statutory reports,
carryover, withholding, document types and the country flows in their generic form. Section 15
covers permissions, section 16 rounding and section 17 the state machines that are not exercised
elsewhere. Country-specific scenarios live in each country's own file, listed in
[README.md](README.md); the counts are given in section 18.

Rule identifiers referenced below are those of [business-rules.md](business-rules.md). Formulas
referenced below are those of [calculations.md](calculations.md). State names and stored values are
those of [state-machines.md](state-machines.md).

---

## 1. Template registry and selection

**1.1 A template is offered for the company's country.**
**Given** a company whose country is Belgium and which has no chart of accounts,
**and** the Belgian capability package installed, contributing the templates `be_comp` (the full
Belgian chart) and `be_asso` (the chart for associations),
**when** the administrator opens the chart of accounts template selector,
**then** both templates appear at the top of the list, ordered before templates of other countries,
**and** no shared base template appears in the list.
Rules FLOC-RULE-002, FLOC-RULE-004.

**1.2 A shared base template cannot be chosen.**
**Given** the Spanish package installed, contributing `es_common` (the shared Spanish base),
`es_pymes` (the chart for small and medium enterprises) and `es_full` (the full chart),
**when** the administrator opens the selector,
**then** `es_pymes` and `es_full` are offered and `es_common` is not,
**and** selecting `es_common` through a direct write is still honoured, because a hidden template
remains loadable.
Rules FLOC-RULE-002, FLOC-RULE-003.

**1.3 Template codes are unique across packages.**
**Given** two capability packages that both declare a template whose code is `generic_coa`,
**when** the second package is installed,
**then** the installation is refused, because a template code identifies exactly one template in
the whole registry.
Rule FLOC-RULE-001.

**1.4 A branch company follows its parent.**
**Given** a parent company using `fr` (the French chart) and a branch company beneath it,
**when** the administrator opens the branch's accounting settings,
**then** the branch shows `fr` as its template code and the selector is not offered on the branch,
**and** the branch shares the parent's accounts, taxes and journals.
Rule FLOC-RULE-006.

**1.5 Only an administrator may load a template.**
**Given** a user who holds the accounting user right but not the accounting administrator right,
**when** that user attempts to select a template,
**then** the operation is refused by the access check and no record is created.
Rule FLOC-RULE-005.

**1.6 A load runs in the base language.**
**Given** a database whose active languages are French and Dutch, a user whose language is Dutch,
and the Belgian template,
**when** that user loads the template,
**then** every record is created with its base-language value first and the French and Dutch
translations are loaded afterwards from the package's translation files,
**and** the account named "Capital" in the base language carries the Dutch translation "Kapitaal"
and the French translation "Capital".
Rule FLOC-RULE-010.

**1.7 Uninstalling the package clears the code.**
**Given** three companies using `pt` (the Portuguese chart),
**when** the Portuguese capability package is removed,
**then** the template code of all three companies is cleared,
**and** their accounts, taxes, journals and posted journal items are untouched.
Rule FLOC-RULE-009.

**1.8 A second, different template is refused once entries exist.**
**Given** a company using `de_skr03` with 1,240 posted journal items,
**when** the administrator tries to load `de_skr04`,
**then** the load is refused with "Can't install chart of account, some accounting entries already
exist for the company.",
**and** the template code stays `de_skr03`.
Rules FLOC-RULE-007, FLOC-RULE-008; state machine section 2.

---

## 2. Account codes

**2.1 Right padding to the template's code length.**
**Given** a template whose code length is 6 and whose account rows carry the codes `1010`, `40` and
`1234567`,
**when** the template is loaded,
**then** the created accounts carry the codes `101000`, `400000` and `1234567`.
Rule FLOC-RULE-011; formula, calculations section 1.

**2.2 Padding preserves a dotted hierarchy.**
**Given** a template whose code length is 9 and whose rows carry `101.01.01` and `102.01`,
**when** the template is loaded,
**then** the created accounts carry `101.01.01` and `102.01000`.
Formula, calculations section 1.

**2.3 A free code is found by incrementing the last digit run.**
**Given** a company that already has accounts `102100` and `102101`,
**when** a utility account is created starting from `102100`,
**then** the code `102102` is used.
Rule FLOC-RULE-014; formula, calculations section 2.

**2.4 The digit run rolls over into the next order of magnitude.**
**Given** a company that already has `1598` and `1599`,
**when** a utility account is created starting from `1598`,
**then** the code `1600` is used, because the run of four digits is incremented as a number and
re-rendered in four digits.
Formula, calculations section 2.

**2.5 The suffix fallback is used when the digit run is exhausted.**
**Given** a company that already has `10.01.97`, `10.01.98` and `10.01.99`,
**when** a utility account is created starting from `10.01.97`,
**then** the code `10.01.97.copy` is used, and a further one takes `10.01.97.copy2`.
Formula, calculations section 2.

**2.6 A code with no digits falls straight through to the suffix fallback.**
**Given** a company that already has an account coded `hello`,
**when** a utility account is created starting from `hello`,
**then** the code `hello.copy` is used.
Formula, calculations section 2.

**2.7 No free code at all aborts the operation.**
**Given** a company that owns every code from `9998` to `9999` and every code from `9998.copy` to
`9998.copy99`,
**when** a utility account is created starting from `9998`,
**then** the operation aborts with "Cannot generate an unused account code.".
Formula, calculations section 2.

**2.8 The starting code of a prefixed utility account.**
**Given** a template whose code length is 6 and a bank prefix of `5710`,
**when** the first bank account is created for the company,
**then** the search starts at `571001`, so the created account carries `571001` when that code is
free.
Rule FLOC-RULE-013; formula, calculations section 3.

**2.9 Renumbering after a prefix change, short prefix to longer.**
**Given** a company whose bank prefix is `1014` and which owns the bank account `101401`,
**when** the prefix is changed to `5100`,
**then** the account's code becomes `510001`: the remainder after removing the old prefix is `01`,
the trimmed remainder is `1`, the target width is 6 − 4 = 2, so the padded remainder is `01`.
Rule FLOC-RULE-015; formula, calculations section 4.

**2.10 Renumbering after a prefix change, long prefix to shorter.**
**Given** a company whose bank prefix is `512` and which owns `512004`,
**when** the prefix is changed to `55`,
**then** the code becomes `550004`: remainder `004`, trimmed `4`, target width 6 − 2 = 4, padded
`0004`.
Formula, calculations section 4.

**2.11 A used account cannot be deleted.**
**Given** an account carrying 3 posted journal items,
**when** its deletion is attempted,
**then** the deletion is refused and the account remains, archivable but not deletable.
Rule FLOC-RULE-016.

**2.12 An account named by a fiscal position mapping is protected.**
**Given** the account `700000` used as the source account of an account mapping on the fiscal
position "Export",
**when** the deletion or the archiving of `700000` is attempted,
**then** it is refused.
Rule FLOC-RULE-017.

**2.13 An off-balance account is neither reconcilable nor taxed.**
**Given** a template row that creates an account of the off-balance type and also sets it
reconcilable and gives it a default tax,
**when** the template is loaded,
**then** the account is created off-balance, not reconcilable and with no tax.
Rule FLOC-RULE-018.

**2.14 A reload does not re-impose reconcilability.**
**Given** a company whose account `411000` was made non-reconcilable by hand, and a template that
declares it reconcilable,
**when** the same template is loaded again,
**then** `411000` stays non-reconcilable, and only its report tags are refreshed.
Rules FLOC-RULE-019, FLOC-RULE-020.

**2.15 A reload adopts an unbound account whose code matches with trailing zeros.**
**Given** a company that has an account `400000` created by hand and not bound to any template row,
and a template row whose code is `40` with a code length of 6,
**when** the template is loaded again,
**then** the existing `400000` is adopted as the row's account instead of a second account being
created, because the template code followed by any number of zero characters matches.
Rule FLOC-RULE-021.

---

## 3. Account groups

**3.1 Both prefixes must have the same length.**
**Given** an account group whose starting prefix is `40` and whose ending prefix is `4099`,
**when** it is saved,
**then** the save is refused, because the two prefixes must have the same number of characters.
Rule FLOC-RULE-022.

**3.2 One prefix defaults to the other.**
**Given** an account group created with the starting prefix `61` and no ending prefix,
**when** it is saved,
**then** the ending prefix becomes `61`.
Rule FLOC-RULE-023.

**3.3 The longest matching prefix wins.**
**Given** the account groups `4` (Receivables and payables) and `40` (Trade receivables), and the
account `401000`,
**when** group assignment runs,
**then** `401000` belongs to the group `40`, not to the group `4`.
Rule FLOC-RULE-024; calculations section 11.

**3.4 A reload leaves existing groups alone.**
**Given** a company that already has 34 account groups,
**when** the template is loaded again,
**then** no group is created, changed or deleted, whatever the template declares.
Rule FLOC-RULE-025.

---

## 4. Taxes and tax groups from a template

**4.1 Default repartition when the template gives none.**
**Given** a template row that creates a 21 percent sales tax with no repartition instructions,
**when** the template is loaded,
**then** the tax is created with the engine's default repartition: one base instruction and one tax
instruction at 100 percent on the invoice side, and the same two on the refund side.
Rule FLOC-RULE-026.

**4.2 A report tag must exist for the template's country.**
**Given** a template row whose repartition instruction names the tag `+21`, and no tag named `21`
for that country,
**when** the template is loaded,
**then** the load fails on that row, because a tag named on a repartition instruction must already
exist for the template's country.
Rule FLOC-RULE-027.

**4.3 A qualified tag name is treated as an external identifier.**
**Given** a repartition instruction naming `l10n_be.tax_tag_54`, and an installed capability
package whose technical name is `l10n_be`,
**when** the template is loaded,
**then** the tag is resolved by external identifier and no lookup by name is attempted.
Rule FLOC-RULE-028.

**4.4 A leading hyphen selects the negative tag.**
**Given** a repartition instruction naming `-54`,
**when** the template is loaded,
**then** the negative variant of the tag `54` is attached.
Rule FLOC-RULE-030.

**4.5 A materially changed tax is retired, not deleted.**
**Given** a company using a template whose 21 percent tax `S-21` exists with 2 repartition lines,
and a new version of the template in which the same tax is declared at 20 percent,
**when** the template is loaded again,
**then** the existing tax is renamed with the retirement marker and deactivated, a new tax at 20
percent is created, and every posted journal item still points at the retired tax.
Rules FLOC-RULE-031, FLOC-RULE-032.

**4.6 A materially unchanged tax is only refreshed.**
**Given** the same tax declared identically, with an amount equal at four decimal places,
**when** the template is loaded again,
**then** the tax keeps its identifier, its name and its repartition, and only its fiscal position
links, its source tax links and its report tags are rewritten.
Rule FLOC-RULE-033.

**4.7 A tax group keeps its accounts on a reload.**
**Given** a tax group whose tax payable account was changed by hand from `451000` to `451100`,
**when** the template is loaded again,
**then** the tax group keeps `451100`.
Rule FLOC-RULE-034.

**4.8 Default taxes are set only when absent.**
**Given** a company whose default sales tax is already the 6 percent tax,
**when** the template is loaded again and declares the 21 percent tax as the default,
**then** the company's default sales tax stays the 6 percent tax.
Rule FLOC-RULE-035.

**4.9 A product carrying another company's tax is corrected.**
**Given** a product whose customer taxes contain a tax belonging to company A, and a load running
for company B,
**when** the load writes company B's default taxes,
**then** the product's customer taxes for company B become company B's default sales tax, and
company A's tax is left in place for company A.
Rule FLOC-RULE-036.

**4.10 The cash basis flag follows the taxes produced.**
**Given** a top-level company with the cash basis flag off,
**when** a template is loaded that produces at least one tax whose exigibility is "on payment",
**then** the company's cash basis flag becomes true.
Rule FLOC-RULE-037.

---

## 5. Fiscal positions and detection

**5.1 A postal code range needs both bounds in order.**
**Given** a fiscal position with the lower bound `9500` and the upper bound `100`,
**when** it is saved,
**then** the save is refused with "Invalid \"Zip Range\", You have to configure both \"From\" and
\"To\" values for the zip range and \"To\" should be greater than \"From\".".
Rule FLOC-RULE-038.

**5.2 Numeric bounds are padded to a common width.**
**Given** a fiscal position with the bounds `100` and `9500`,
**when** it is saved,
**then** the stored bounds are `0100` and `9500`,
**and** a contact whose postal code is `0500` matches the position,
**and** a contact whose postal code is `9600` does not,
**and** a contact whose postal code is `500` does not, because the comparison is textual.
Rule FLOC-RULE-039; formula, calculations section 12.

**5.3 A foreign registration needs a country.**
**Given** a fiscal position with the foreign tax identification number `DE123456789` and no country,
**when** it is saved,
**then** the save is refused with "The country of the foreign VAT number could not be detected.
Please assign a country to the fiscal position.".
Rule FLOC-RULE-040.

**5.4 A registration in the company's own country needs a subdivision.**
**Given** a company whose fiscal country is the United States, and a fiscal position carrying a
United States registration number with no state,
**when** it is saved,
**then** the save is refused with "You cannot create a fiscal position with a foreign VAT within
your fiscal country without assigning it a state.".
Rule FLOC-RULE-041.

**5.5 A country outside the named group is refused.**
**Given** a fiscal position naming the country Switzerland and the country group of the European
Union,
**when** it is saved,
**then** the save is refused with "You cannot create a fiscal position with a country outside of
the selected country group.".
Rule FLOC-RULE-042.

**5.6 One registration number per country per company.**
**Given** a company that already holds a fiscal position with a German registration number,
**when** a second fiscal position with a different German registration number is saved for the same
company,
**then** the save is refused with "A fiscal position with a foreign VAT already exists in this
country.".
Rule FLOC-RULE-043.

**5.7 A malformed registration number is reported by the checker.**
**Given** a fiscal position for Germany with the number `DE12345`,
**when** it is saved,
**then** the number checker refuses it and names the record as `fiscal position [<name>]` in the
message.
Rule FLOC-RULE-044.

**5.8 Detection returns the first position satisfying all predicates.**
**Given** a company in Belgium with the automatic positions "Intra-Community" (sequence 20, country
group of the European Union, tax identification number required) and "Domestic" (sequence 10,
country Belgium),
**and** a customer in the Netherlands carrying the number `NL123456789B01`,
**when** an invoice is created for that customer,
**then** the position "Intra-Community" is applied, because "Domestic" fails the country predicate
and "Intra-Community" satisfies all five.
Rules FLOC-RULE-045, FLOC-RULE-046.

**5.9 A manual position on the delivery contact overrides detection.**
**Given** the same records, and the delivery contact of that customer carrying the fiscal position
"Export" set by hand,
**when** the invoice is created,
**then** "Export" is applied and detection does not run.
Rule FLOC-RULE-047.

**5.10 The invoicing address is used for a same-prefix intra-union pair.**
**Given** a company in Belgium with the number `BE0123456789`, a customer whose invoicing address is
in Belgium and whose number is `BE0987654321`, and a delivery address in France,
**when** the invoice is created,
**then** detection uses the invoicing address, so the domestic position is applied rather than the
intra-union one.
Rule FLOC-RULE-048.

**5.11 A position with no tax mapping strips mapped taxes.**
**Given** the fiscal position "Not subject to tax" with no tax mapping, an invoice line carrying the
21 percent domestic tax which is the source tax of at least one mapping elsewhere, and a second
line carrying a stamp duty that no position maps,
**when** the position is applied,
**then** the first line loses its tax and the second line keeps its stamp duty.
Rule FLOC-RULE-049; accounting effects section 5.1.

**5.12 An account mapping triple is unique.**
**Given** a fiscal position that already maps `700000` to `701000`,
**when** a second mapping from `700000` is added on the same position,
**then** the save is refused.
Rule FLOC-RULE-050.

**5.13 A reload only adds mappings that introduce a new account.**
**Given** a company whose position "Export" already maps `700000` to `705000` by hand, and a
template that declares the mapping `700000` to `701000`,
**when** the template is loaded again,
**then** the hand-made mapping is kept and the template's mapping is not applied, because it does
not introduce an account the position does not already map from.
Rule FLOC-RULE-051.

---

## 6. Journals and company defaults

**6.1 The six journals of every template.**
**Given** a company with no accounting,
**when** any template is loaded,
**then** six journals exist: a sales journal, a purchase journal, a miscellaneous journal, an
exchange difference journal, a cash basis journal and a general journal for the year-end entry, each
with the code the template declares.
Rule FLOC-RULE-054.

**6.2 Journal codes are translated.**
**Given** a template whose sales journal code is `INV` and a French translation `FAC`,
**when** the template is loaded in a database whose base language is French,
**then** the journal's code is `FAC`, even though the code field itself is not a translatable field.
Rule FLOC-RULE-052.

**6.3 A reload adopts the existing journal.**
**Given** a company whose sales journal was renamed "Customer Invoices 2024",
**when** the template is loaded again,
**then** the same journal is reused with its name intact and no second sales journal is created.
Rule FLOC-RULE-053.

**6.4 The company currency comes from the fiscal country.**
**Given** a company in Japan with no journal items and the currency of the company still set to the
database default,
**when** the Japanese template is loaded,
**then** the company currency becomes the Japanese currency and that currency is activated.
Rule FLOC-RULE-055.

**6.5 The company currency is frozen once entries exist.**
**Given** a company with 1 posted journal item,
**when** a change of the company currency is attempted,
**then** it is refused.
Rule FLOC-RULE-056.

**6.6 The tax inclusion default is frozen once invoicing has started.**
**Given** a company that has issued 12 invoices,
**when** a change of the default sales price tax inclusion is attempted,
**then** it is refused.
Rule FLOC-RULE-057.

**6.7 Receivable and payable defaults are company-scoped defaults on Contact.**
**Given** a load that declares the receivable account `411000` and the payable account `401000`,
**when** the load completes,
**then** two company-scoped default values exist for the Contact entity, one per field, and the
company record itself carries no receivable or payable field.
Rule FLOC-RULE-059.

**6.8 A reload does not re-impose the property defaults.**
**Given** a company whose default receivable account was changed by hand to `411100`,
**when** the template is loaded again,
**then** the default stays `411100`.
Rule FLOC-RULE-060.

**6.9 A current-year-earnings account always exists.**
**Given** a template that declares no account of the type "Current Year Earnings",
**when** it is loaded,
**then** the load creates one anyway, so that the year-end entry has a destination.
Rule FLOC-RULE-061.

**6.10 The fiscal country defaults to the company country.**
**Given** a company whose country is Italy and whose fiscal country is empty,
**when** the template is loaded,
**then** the fiscal country becomes Italy,
**and** the Italian country-specific fields become visible on invoices, contacts and the company.
Rules FLOC-RULE-062, FLOC-RULE-121.

**6.11 Reversal by negative amounts is forced where the law requires it.**
**Given** a company in a country whose law requires reversals to be posted as negative amounts on
the original side rather than as opposite entries,
**when** its template is loaded,
**then** the reversal setting is switched on and cannot be switched off,
**and** in a country where the method is merely permitted the setting is switched on and can be
switched off.
Rule FLOC-RULE-063.

**6.12 A restrictive audit trail can be forced.**
**Given** a company in a country whose package forces the restrictive audit trail,
**when** its template is loaded,
**then** the audit trail setting is switched on and the ordinary means of switching it off are
withdrawn.
Rule FLOC-RULE-064.

**6.13 The cost accounting flag stays off.**
**Given** any template,
**when** it is loaded,
**then** the cost accounting flag of the company is false unless the template sets it explicitly.
Rule FLOC-RULE-058.

**6.14 Every created record belongs to the loaded company.**
**Given** a database with three companies and a load running for the second,
**when** the load completes,
**then** every account, tax, tax group, journal, fiscal position, reconciliation model and report
created by the load names the second company, and none of them is visible to the first or the third.
Rule FLOC-RULE-130.

---

## 7. Statutory report structure

**7.1 A root report may not have a root report.**
**Given** a report used as the root of a set of variants,
**when** a root report is set on it,
**then** the save is refused.
Rule FLOC-RULE-065.

**7.2 A section may not have sections.**
**Given** a composite report with the section "Value-added tax",
**when** a section is added beneath that section,
**then** the save is refused.
Rule FLOC-RULE-066.

**7.3 Country availability requires a country.**
**Given** a report whose availability is "Country Matches" and whose country is empty,
**when** it is saved,
**then** the save is refused.
Rule FLOC-RULE-067.

**7.4 A parent line must sort before its children.**
**Given** a report with the parent line at sequence 30 and a child at sequence 20,
**when** it is saved,
**then** the save is refused.
Rule FLOC-RULE-068.

**7.5 A line may not be its own parent.**
**Given** a line whose parent is set to itself,
**when** it is saved,
**then** the save is refused.
Rule FLOC-RULE-069.

**7.6 A line may not have both children and a grouping.**
**Given** a line with two children,
**when** a grouping is set on it,
**then** the save is refused.
Rule FLOC-RULE-070.

**7.7 Line codes and expression labels are unique in their scope.**
**Given** a report with the line code `NET`,
**when** a second line with the code `NET` is added to the same report,
**then** the save is refused; and when a second expression labelled `balance` is added to a line
that already has one, the save is refused as well.
Rules FLOC-RULE-071, FLOC-RULE-072.

**7.8 Aggregation and external expressions reject grouping.**
**Given** a line with a grouping,
**when** an aggregation expression is added to it,
**then** the save is refused; the same happens for an external-value expression.
Rule FLOC-RULE-073.

**7.9 A domain expression needs a subformula.**
**Given** an expression whose engine is the domain engine and whose subformula is empty,
**when** it is saved,
**then** the save is refused.
Rule FLOC-RULE-074.

**7.10 Formulas are normalised on write.**
**Given** the aggregation formula `  OUT.balance   -   IN.balance  `,
**when** it is saved,
**then** the stored formula is `OUT.balance - IN.balance`.
Rule FLOC-RULE-076.

**7.11 A cross-report reference must resolve.**
**Given** an aggregation formula naming a line of a report that does not exist,
**when** it is saved,
**then** the save is refused; and a formula naming a line of the same report through the
cross-report syntax is refused as well.
Rule FLOC-RULE-077.

**7.12 Carryover labels are constrained.**
**Given** an expression whose carryover role is the source and whose label is `carryover_balance`,
**when** it is saved,
**then** the save is refused, because the label must begin with `_carryover_`; and a target
expression labelled `applied_balance` is refused, because it must begin with
`_applied_carryover_`.
Rules FLOC-RULE-078, FLOC-RULE-079.

**7.13 An unresolvable carryover target aborts the run.**
**Given** a carryover source whose named target does not exist and whose line has no expression
labelled `_applied_carryover_balance`,
**when** the report is run,
**then** the run aborts rather than silently discarding the amount.
Rule FLOC-RULE-080.

**7.14 A report with variants cannot be deleted.**
**Given** a root report with two variants,
**when** its deletion is attempted,
**then** it is refused.
Rule FLOC-RULE-081.

**7.15 Deleting a tax-tag expression archives used tags.**
**Given** an expression whose engine is the tax-tags engine and whose tags are used by 47 journal
items,
**when** the expression is deleted,
**then** the tags are archived, not deleted; and when no journal item uses them, they are deleted.
Rule FLOC-RULE-082.

**7.16 Renaming a tax-tag expression renames the tag only when the change is complete.**
**Given** the tag `54` used by two expressions,
**when** only one of the two is renamed,
**then** the tag keeps its name and a new tag is used by the renamed expression; when both are
renamed in the same operation, the tag itself is renamed.
Rule FLOC-RULE-083.

**7.17 Changing a report's country moves its tags only when exclusive.**
**Given** a report whose tags are used only by that report,
**when** its country changes from Belgium to Luxembourg,
**then** the tags move to Luxembourg; when another report uses the same tags, the tags stay and new
ones are created.
Rule FLOC-RULE-084.

**7.18 Copying a report renames codes and rewrites formulas.**
**Given** the report "Tax Return" with the lines `OUT`, `IN` and `NET`, where `NET.balance` is
`OUT.balance - IN.balance`,
**when** the report is copied,
**then** the copy is named "Tax Return (copy)", its lines are `OUT_COPY`, `IN_COPY` and `NET_COPY`,
and `NET_COPY.balance` is `OUT_COPY.balance - IN_COPY.balance`,
**and** copying the copy produces "Tax Return (copy) (copy)" with the codes `OUT_COPY_COPY`,
`IN_COPY_COPY` and `NET_COPY_COPY`.
Rule FLOC-RULE-085; formula, calculations section 10.

**7.19 Report line levels.**
**Given** a root line with no parent, a child of it and a grandchild,
**when** the report is rendered,
**then** the three lines carry the levels 1, 3 and 5; and a line explicitly set to level 0 has
children at level 3.
Formula, calculations section 9.

---

## 8. Carryover

**8.1 A negative net carries forward and is applied in the next period.**
**Given** a value-added tax return with the lines `OUT` (output tax), `IN` (input tax) and `NET`,
where `NET.balance` is `OUT.balance - IN.balance - NET._applied_carryover_balance`,
**and** March with an output tax of 4,000.00 and an input tax of 5,250.00,
**when** March is closed,
**then** `NET` is 4,000.00 − 5,250.00 − 0.00 = −1,250.00, the carryover expression evaluates to
1,250.00, and one external value of 1,250.00 dated 31 March is written against
`NET._applied_carryover_balance` for the company,
**and** when April is run with an output tax of 6,000.00 and an input tax of 4,000.00, the applied
carryover reads 1,250.00, `NET` is 6,000.00 − 4,000.00 − 1,250.00 = 750.00, the carryover expression
evaluates to 0.00 and nothing further is carried,
**and** the company pays 750.00.
Rule FLOC-RULE-128; formula, calculations section 8.

**8.2 The external value carries its origin.**
**Given** the March carryover of scenario 8.1,
**when** the external value is inspected,
**then** it names the target expression, the amount 1,250.00, the date 31 March, the company, the
origin line `NET` and the origin expression label `_carryover_balance`.
Formula, calculations section 8.

---

## 9. Withholding tax

**9.1 A withholding tax may not use a forbidden computation.**
**Given** a tax marked as withheld on payment,
**when** its computation is set to the group computation or to the tax-included percentage
computation,
**then** the save is refused.
Rule FLOC-RULE-086.

**9.2 Marking a tax as withheld forces two settings.**
**Given** a tax whose exigibility is "on payment" and whose prices include the tax,
**when** it is marked as withheld on payment,
**then** its exigibility becomes "on invoice" and its prices become tax excluded.
Rule FLOC-RULE-087.

**9.3 A withholding tax must have a negative amount.**
**Given** a tax whose amount is 3,
**when** it is marked as withheld on payment,
**then** the save is refused; with an amount of −3 the save succeeds.
Rule FLOC-RULE-088.

**9.4 Withholding taxes never enter the ordinary tax total.**
**Given** an invoice line of 10,000.00 carrying the 21 percent sales tax and a −3 percent
withholding tax,
**when** the invoice totals are computed,
**then** the untaxed total is 10,000.00, the tax total is 2,100.00 and the invoice total is
12,100.00; the withholding tax contributes nothing.
Rule FLOC-RULE-089.

**9.5 A withholding line's base must be positive.**
**Given** a withholding line with a base amount of 0.00,
**when** the payment is prepared,
**then** it is refused; and the same happens with a base amount of −100.00.
Rule FLOC-RULE-091.

**9.6 A withholding line may not use a liquidity account.**
**Given** a payment whose outstanding account is `580000`, and a withholding line whose account is
also `580000`,
**when** the payment is prepared,
**then** it is refused; the same happens when the account is the inter-banks transfer account.
Rule FLOC-RULE-092.

**9.7 A line without number and without series blocks preparation.**
**Given** a withholding line whose tax has no numbering series and whose number is empty, in a
country that requires the number,
**when** the payment is prepared,
**then** the preparation is blocked.
Rule FLOC-RULE-093.

**9.8 A negative net amount is refused.**
**Given** a payment of 1,000.00 with a withholding line of 1,200.00,
**when** the payment is prepared,
**then** it is refused, because the net amount would be −200.00.
Rule FLOC-RULE-094; formula, calculations section 6.5.

**9.9 The net amount of a payment with two withholdings.**
**Given** a payment of 10,000.00 with withholding lines of 1,000.00 and 200.00,
**when** the net amount is computed,
**then** it is 8,800.00, and the customer transfers 8,800.00.
Formula, calculations section 6.5.

**9.10 Conversion, case B: same currency.**
**Given** an invoice of 10,000.00 in the company currency carrying a 10 percent withholding tax, and
a payment in the same currency,
**when** the wizard opens,
**then** the line's original base amount is 10,000.00 and its original withheld amount is 1,000.00.
Formula, calculations section 6.2.

**9.11 Conversion, case C: source foreign, payment in the company currency.**
**Given** an invoice of 8,000.00 in a foreign currency with a 3 percent withholding, a payment in
the company currency, and a rate of 1.25 company units per foreign unit at the payment date,
**when** the wizard opens,
**then** the original base amount is round(8,000.00 × 1.25, 2) = 10,000.00 and the original withheld
amount is round(240.00 × 1.25, 2) = 300.00.
Rule FLOC-RULE-132; formula, calculations section 6.2.

**9.12 Conversion, case D: payment in a foreign currency.**
**Given** captured company-currency amounts of 10,000.00 and 300.00, and a payment in a foreign
currency whose rate from the company currency is 0.80,
**when** the wizard opens,
**then** the original base amount is round(10,000.00 × 0.80, 2) = 8,000.00 and the original withheld
amount is round(300.00 × 0.80, 2) = 240.00.
Formula, calculations section 6.2.

**9.13 Proration on a half payment.**
**Given** an invoice of 10,000.00 with nothing paid, and a payment of 5,000.00,
**when** the wizard computes the base,
**then** the split factor is 1, the percentage paid is 0.5 and the base amount is 5,000.00.
Formula, calculations section 6.3.

**9.14 Proration on a second instalment.**
**Given** an invoice of 10,000.00 of which 4,000.00 is already paid, so that the amount due is
6,000.00,
**when** the remaining 6,000.00 is paid,
**then** the split factor is 6,000 ÷ 10,000 = 0.6, the percentage paid is |6,000 ÷ 6,000| × 0.6 =
0.6 and the base amount is 6,000.00, which is the share of the original invoice being settled.
Formula, calculations section 6.3.

**9.15 The withheld amount is prorated in one rounding.**
**Given** an original base of 10,000.00, an original withheld amount of 1,000.00 and a base
overridden to 3,333.33,
**when** the withheld amount is recomputed,
**then** it is round(1,000.00 × 3,333.33 ÷ 10,000.00, 2) = round(333.333, 2) = 333.33, and not
330.00, which is what rounding the ratio first would give.
Rule FLOC-RULE-141; formula, calculations section 6.4.

**9.16 A fractional rate.**
**Given** a 10.666666666667 percent withholding on a base of 7,500.00, giving an original withheld
amount of round(7,500.00 × 0.10666666666667, 2) = 800.00, and a base overridden to 7,000.00,
**when** the withheld amount is recomputed,
**then** it is round(800.00 × 7,000.00 ÷ 7,500.00, 2) = round(746.666…, 2) = 746.67.
Formula, calculations section 6.4.

**9.17 Placeholder numbering across three lines.**
**Given** a numbering series whose prefix is `WH/`, whose padding is 5 and whose next value is 42,
and three withholding lines drawing on it,
**when** the wizard shows the lines,
**then** the placeholders are `WH/00042`, `WH/00043` and `WH/00044`, no value is consumed, and the
placeholder type of each line is `sequence`.
Rule FLOC-RULE-099; formula, calculations section 6.7; state machine section 4.

**9.18 Posting allocates the real numbers.**
**Given** the three lines of scenario 9.17,
**when** the payment is posted,
**then** the series is consumed three times, the lines carry the numbers `WH/00042`, `WH/00043` and
`WH/00044`, the placeholder type of each becomes `name` and the series' next value becomes 45.
State machine section 4.

**9.19 Refund detection inverts the direction.**
**Given** a withholding tax whose scope is "sale" and an outbound payment,
**when** the repartition is chosen,
**then** the refund half of the tax's repartition is used; the same happens for a purchase-scope tax
on an inbound payment.
Rule FLOC-RULE-098; formula, calculations section 6.6.

**9.20 The outstanding account is made reconcilable.**
**Given** a payment with withholding whose chosen outstanding account is not reconcilable,
**when** the payment is prepared,
**then** the account is made reconcilable so that the withholding items can be matched.
Rule FLOC-RULE-096.

**9.21 The withholding section disappears for a multi-entry payment.**
**Given** a payment registration that would produce two journal entries because two counterparts are
selected,
**when** the wizard opens,
**then** the withholding section is not shown.
Rule FLOC-RULE-097.

**9.22 Lines of two different payments cannot be prepared together.**
**Given** two payments each carrying withholding lines,
**when** the lines of both are prepared in one operation,
**then** the operation is refused.
Rule FLOC-RULE-095.

**9.23 Progressive scale withholding.**
**Given** a scale with the brackets (from 0.00: fixed 0.00, 5 percent from 0.00), (from 100,000.00:
fixed 5,000.00, 10 percent from 100,000.00) and (from 300,000.00: fixed 25,000.00, 15 percent from
300,000.00),
**and** a counterpart that has already received 250,000.00 in the period with 20,000.00 withheld,
**when** a payment of 100,000.00 is registered,
**then** the accumulated base is 350,000.00, the third bracket applies, the total withholding is
25,000.00 + (350,000.00 − 300,000.00) × 15 ÷ 100 = 32,500.00, the amount withheld now is
round(32,500.00 − 20,000.00, 2) = 12,500.00 and the counterpart receives 87,500.00.
Formula, calculations section 13.

**9.24 The printed label of a withholding tax.**
**Given** a withholding tax named "Income tax withholding 3%" whose invoice label is "Withholding
3%",
**when** the payment receipt is printed,
**then** the label shown is "Withholding 3%".
Rule FLOC-RULE-090.

---

## 10. Ledger effects of withholding, closing and cash basis taxes

**10.1 Inbound payment where the customer withholds.**
**Given** a posted customer invoice of 1,000.00 in the company currency carrying a 10 percent
withholding tax on payment, whose withholding account is "Income Tax Withheld Receivable",
**when** the customer pays and withholds 100.00, transferring 900.00,
**then** one journal entry is posted with five items: Outstanding Receipts debit 900.00, Accounts
Receivable credit 1,000.00, Income Tax Withheld Receivable debit 100.00, Withholding Tax Base debit
1,000.00 and Withholding Tax Base credit 1,000.00, totalling 2,000.00 on each side,
**and** the receivable item of 1,000.00 fully reconciles the invoice, which becomes paid,
**and** the outstanding item of 900.00 stays open until the bank statement line is reconciled,
**and** the two base items net to zero on the base account but carry the base report tags.
Accounting effects section 2.2.

**10.2 Outbound payment where the company withholds.**
**Given** a posted vendor bill of 5,000.00 carrying a 3 percent withholding tax on payment,
**when** the company pays 4,850.00,
**then** the entry is Accounts Payable debit 5,000.00, Outstanding Payments credit 4,850.00,
Withholding Tax Payable credit 150.00, Withholding Tax Base credit 5,000.00 and Withholding Tax Base
debit 5,000.00, totalling 10,000.00 on each side,
**and** the company owes 150.00 to the tax administration.
Accounting effects section 2.3.

**10.3 Partial payment with withholding.**
**Given** the invoice of scenario 10.1,
**when** the customer pays half,
**then** the percentage paid is 0.5, the base is 500.00, the withheld amount is round(100.00 ×
500.00 ÷ 1,000.00, 2) = 50.00 and the net is 450.00,
**and** the entry is Outstanding Receipts debit 450.00, Accounts Receivable credit 500.00, Income
Tax Withheld Receivable debit 50.00 and the two base items of 500.00,
**and** the invoice keeps a residual of 500.00.
Accounting effects section 2.4.

**10.4 Payment in a foreign currency.**
**Given** a customer invoice of 8,000.00 in a foreign currency with a 3 percent withholding, a rate
of 1.25 company units per foreign unit at the payment date, so that the rate from the company
currency to the foreign currency is 0.80,
**when** the payment is registered in the foreign currency,
**then** the line amounts are 8,000.00 base and 240.00 withheld in the foreign currency, and
round(8,000.00 ÷ 0.80, 2) = 10,000.00 and round(240.00 ÷ 0.80, 2) = 300.00 in the company currency.
Accounting effects section 2.5; formula, calculations section 6.6.

**10.5 Tax closing, net payable.**
**Given** a period with 21,000.00 of collected tax as a credit on "Tax Collected" and 13,400.00 of
deductible tax as a debit on "Tax Deductible", both in one tax group, with no advance payment,
**when** the period is closed,
**then** one entry dated on the last day of the period is posted in the tax closing journal with
"Tax Collected" debit 21,000.00, "Tax Deductible" credit 13,400.00 and "Tax Payable" credit
7,600.00, totalling 21,000.00 on each side,
**and** the company's tax lock date moves to the period end.
Rules FLOC-RULE-126, FLOC-RULE-127; accounting effects section 3.3.

**10.6 Tax closing, net receivable with an advance payment.**
**Given** a period with 4,000.00 collected, 5,250.00 deductible and 500.00 sitting as a debit on the
advance tax payment account,
**when** the period is closed,
**then** the entry is "Tax Collected" debit 4,000.00, "Tax Deductible" credit 5,250.00, "Tax Advance
Payment" credit 500.00 and "Tax Receivable" debit 1,750.00, totalling 5,750.00 on each side.
Accounting effects section 3.4.

**10.7 A closed period is locked.**
**Given** the closing of scenario 10.5,
**when** a journal item dated inside the closed period is created, modified or deleted,
**then** it is refused by the tax lock date.
Rule FLOC-RULE-127.

**10.8 A cash basis tax at the invoice.**
**Given** a template that creates a 16 percent sales tax whose exigibility is "on payment" with a
cash basis transition account, and a customer invoice of 1,000.00 carrying that tax,
**when** the invoice is posted,
**then** the entry is Sales credit 1,000.00, cash basis transition account credit 160.00 and
Accounts Receivable debit 1,160.00,
**and** the tax item carries no report tag, so the amount is absent from the return.
Accounting effects section 4.1.

**10.9 The same cash basis tax at the payment.**
**Given** the invoice of scenario 10.8,
**when** it is reconciled with a payment of 1,160.00,
**then** an additional entry dated on the reconciliation date is posted in the cash basis journal
with the cash basis base account debit 1,000.00 and credit 1,000.00, the transition account debit
160.00 and "Tax Collected" credit 160.00,
**and** the base items carry the base report tags and the tax item carries the tax report tags, so
the amount now appears in the return.
Accounting effects section 4.2.

**10.10 A cash basis tax on a partial payment.**
**Given** the same invoice,
**when** 580.00 of the 1,160.00 is paid,
**then** the transferred tax is round(160.00 × 580.00 ÷ 1,160.00, 2) = 80.00 and the reported base
is round(1,000.00 × 580.00 ÷ 1,160.00, 2) = 500.00.
Accounting effects section 4.3.

**10.11 Account substitution by a fiscal position.**
**Given** the fiscal position "Export" mapping the revenue account `700000` to `701000`, and an
invoice line whose product's income account is `700000`,
**when** the invoice is posted for an export customer,
**then** the revenue item credits `701000` and not `700000`,
**and** the counterpart line's receivable account is substituted the same way when the position maps
it.
Accounting effects section 5.2.

**10.12 Tax substitution by a fiscal position.**
**Given** the fiscal position "Intra-Community" mapping the domestic 21 percent tax to a 0 percent
intra-union tax, and a line carrying the domestic tax,
**when** the position is applied,
**then** the prepared line carries the 0 percent tax, the entry has no tax item and the base item
carries the intra-union tags instead of the domestic ones.
Accounting effects section 5.1.

---

## 11. Document types and numbering

**11.1 A journal that uses document types demands one.**
**Given** a sales journal marked as using document types, and a customer invoice in that journal with
no document type,
**when** the invoice is posted,
**then** the posting is refused, because the invoice must carry a document type and a document
number.
Rules FLOC-RULE-100, FLOC-RULE-101.

**11.2 An invoice-class document type may not be used on a credit note.**
**Given** a document type whose internal type is `invoice`,
**when** it is set on a credit note and the credit note is posted,
**then** the posting is refused; and a document type whose internal type is `credit_note` set on an
invoice is refused likewise.
Rule FLOC-RULE-102.

**11.3 Only invoice-class entries may live in such a journal.**
**Given** a journal that uses document types,
**when** a miscellaneous entry is posted in it,
**then** the posting is refused.
Rule FLOC-RULE-103.

**11.4 Numbering is per journal and per document type.**
**Given** a journal that uses document types, with the types "Invoice A" and "Invoice B",
**when** three invoices of type "Invoice A" and two of type "Invoice B" are posted,
**then** the first series has consumed three values and the second two, and the two series are
independent.
Rule FLOC-RULE-104.

---

## 12. Point of sale certification

**12.1 A validated order is hashed and chained.**
**Given** a company whose accounting is unalterable and a point of sale session with two orders
already validated,
**when** a third order is validated,
**then** it stores a hash computed over its own immutable fields together with the hash of the
second order, and its sequence position is the next one.
Rule FLOC-RULE-105; state machine section 26.1.

**12.2 A hashed order cannot be changed.**
**Given** a hashed order,
**when** a protected field of one of its lines is modified,
**then** the change is refused with "According to the French law, you cannot modify a point of sale
order line. Forbidden fields: `<fields>`.",
**and** an attempt to overwrite the hash itself is refused with "You cannot overwrite the values
ensuring the inalterability of the point of sale.",
**and** a deletion is refused with "According to French law, you cannot delete a point of sale
order.".
Rule FLOC-RULE-105.

**12.3 A closing is immutable.**
**Given** a daily closing produced by the scheduled job,
**when** a change or a deletion of it is attempted,
**then** it is refused: Sale Closings must never be modified or deleted under any circumstances.
Rule FLOC-RULE-106.

**12.4 The integrity check reports the first divergence.**
**Given** a chain of 120 orders in which the hash of the 57th no longer matches its content,
**when** the integrity check is run,
**then** it reports the 57th order as the first divergence and does not report the later ones
separately.
Rule FLOC-RULE-107.

**12.5 The integrity result is restricted.**
**Given** a user who is not an accounting user,
**when** the integrity result is printed,
**then** it is refused with "Please contact your accountant to print the Hash integrity result.",
**and** for a company whose accounting is not unalterable the refusal is "Accounting is not
unalterable for the company `<company>`. This mechanism is designed for companies where accounting
is unalterable.".
Rule FLOC-RULE-108.

---

## 13. Electronic invoicing, generic behaviour

**13.1 A second concurrent transmission is refused.**
**Given** an invoice whose transmission is running,
**when** a second transmission of the same invoice is started,
**then** it is refused with a message of the form "This document is being sent by another process
already." and nothing is transmitted twice.
Rule FLOC-RULE-109; state machine section 5.

**13.2 A transmitted document cannot be deleted.**
**Given** a generated goods movement permit,
**when** its deletion is attempted,
**then** it is refused with "You cannot delete a generated E-waybill. Instead, you should cancel
it.",
**and** the deletion of a sent reporting flow is refused with "You cannot delete sent flows.".
Rule FLOC-RULE-110.

**13.3 Blocking errors are shown together.**
**Given** an invoice missing the counterpart's identification number and carrying a product with no
classification code,
**when** transmission is attempted,
**then** nothing is transmitted and both problems are listed in one message, grouped by cause.
Rule FLOC-RULE-111.

**13.4 Reset to draft is blocked after transmission.**
**Given** an invoice whose country exchange state is a sent state,
**when** a reset to draft is attempted,
**then** it is refused, and the reset button is not offered.
Rule FLOC-RULE-112.

**13.5 A cancellation needs a reason.**
**Given** a registered invoice in a country that allows cancellation,
**when** a cancellation is requested with no reason,
**then** it is refused; with a reason and, where the country demands them, remarks, the cancellation
is transmitted.
Rule FLOC-RULE-113.

**13.6 A chained country stores the chain.**
**Given** a country that requires chaining and three invoices transmitted in order,
**when** the third is transmitted,
**then** it stores its own index in the chain and the fingerprint of the second, and the invoice
carries a copy of the index.
Rule FLOC-RULE-114.

---

## 14. Foreign registration and foreign taxes

**14.1 Foreign taxes are instantiated once.**
**Given** a Belgian company with a fiscal position carrying a German registration number, and no
German tax in the company,
**when** the foreign tax instantiation runs,
**then** the German taxes are created inside the Belgian chart, they carry Germany as their country,
their tax groups are named with the German template code as a prefix, and they are attached to that
fiscal position,
**and** running the instantiation again does nothing, because the company now has German taxes.
Rules FLOC-RULE-115, FLOC-RULE-118.

**14.2 Accounts are copied from the closest local equivalent.**
**Given** the same company, where the German template's tax uses an account whose closest Belgian
equivalent is `451000`,
**when** the instantiation runs,
**then** a copy of `451000` is created for the German tax,
**and** where no local equivalent can be found, the account of the German tax is left empty rather
than filled with a wrong account.
Rules FLOC-RULE-116, FLOC-RULE-117.

**14.3 Foreign fiscal position and source tax links are discarded.**
**Given** a German template whose taxes are attached to German fiscal positions and name German
source taxes,
**when** they are instantiated in the Belgian company,
**then** none of those links is reproduced; the taxes are attached only to the position that
triggered the instantiation.
Rule FLOC-RULE-119.

**14.4 The result is an accelerator.**
**Given** the instantiated German taxes,
**when** the accountant reviews them,
**then** the accounts and the report tags must be checked before the first German return is filed,
because the instantiation is an accelerator and not a certified configuration.
Rule FLOC-RULE-120.

**14.5 A report run under a foreign registration is scoped.**
**Given** the Belgian company with the German registration and 40 journal items of which 12 carry
the German fiscal position,
**when** the German return is run under that registration,
**then** only those 12 items are considered.
Rule FLOC-RULE-129.

**14.6 The header mode follows the availability of a template.**
**Given** a fiscal position for a country that has a registered template and a company with no tax
of that country,
**when** the registration number is entered,
**then** the header mode becomes `templates_found` and the generation action is offered,
**and** for a country with no template the header mode becomes `no_template`,
**and** clearing the number returns the header mode to empty.
State machine section 3.

---

## 15. Field visibility, validation and permissions

**15.1 A country field appears only for an enabled country.**
**Given** a company whose fiscal country is Spain and which holds a French registration,
**when** an invoice form is opened,
**then** the Spanish and the French country-specific fields are visible and the Italian ones are
not.
Rule FLOC-RULE-121.

**15.2 A mandatory country field is enforced at posting, not at entry.**
**Given** a country that requires a place of supply on every invoice,
**when** a draft invoice is saved without it,
**then** the save succeeds,
**and** when the invoice is posted, the posting is refused by the country's posting constraint.
Rule FLOC-RULE-122.

**15.3 An identification number is checked twice.**
**Given** a contact form for a country whose identification numbers carry a check digit,
**when** a malformed number is typed,
**then** the checker refuses it on entry,
**and** when the same value is written directly to the record, the checker refuses it again on save.
Rule FLOC-RULE-123.

**15.4 The online checkout validates the same way.**
**Given** a country package that adds a mandatory identification type and number to the contact,
**when** a customer completes the online checkout,
**then** the same two fields are requested, validated with the same checker and refused with the
same message.
Rule FLOC-RULE-124.

**15.5 Only an accounting manager may edit country configuration.**
**Given** a user with the accounting user right,
**when** that user tries to change the numbering series of a document type, or to activate a tax, or
to register a foreign tax identification number,
**then** the operation is refused by the access check,
**and** an accounting manager may perform all three.
Rule FLOC-RULE-016 and section 16 of business-rules.md.

**15.6 A reference table is shared by every company.**
**Given** three companies and one Latin America Document Type record,
**when** each company opens the document type list,
**then** all three see the same record, because the table is not company scoped.
Rule FLOC-RULE-133.

**15.7 A withholding line takes its company from its carrier.**
**Given** a payment belonging to company B,
**when** a withholding line is created on it,
**then** the line's company is B and the field cannot be typed.
Rule FLOC-RULE-131.

**15.8 A dated authorisation must be ordered.**
**Given** an authorisation whose start date is 1 June and whose end date is 1 May of the same year,
**when** it is saved,
**then** the save is refused.
Rule FLOC-RULE-134.

**15.9 An expired authorisation blocks posting.**
**Given** an authorisation valid from 1 January to 31 March and an invoice dated 2 April naming it,
**when** the invoice is posted,
**then** the posting is refused and the message names the validity window and the document date.
Rule FLOC-RULE-137.

**15.10 A document belongs to the period of its accounting date.**
**Given** an invoice issued on 31 March and dated 1 April for accounting,
**when** the March return is run,
**then** the invoice is not included; it appears in the April return.
Rule FLOC-RULE-135.

**15.11 The return period length is a company setting.**
**Given** a company whose return period is set to two months,
**when** the return is opened,
**then** the default period is the current two-month period, and the choices are monthly,
two-monthly, quarterly, four-monthly, half-yearly and yearly.
Rule FLOC-RULE-125.

---

## 16. Rounding

**16.1 Round per Tax and Round per Line agree on three lines of 33.33.**
**Given** a company whose tax rounding method is "Round per Tax", three invoice lines of 33.33 each
and a 21 percent tax, in a currency with two decimal places,
**when** the invoice is computed,
**then** the tax is round(99.99 × 0.21, 2) = round(20.9979, 2) = 21.00,
**and** with the method "Round per Line" the tax is round(6.9993, 2) × 3 = 7.00 × 3 = 21.00.
Rule FLOC-RULE-138; formula, calculations section 14.

**16.2 The two methods diverge on three lines of 10.03.**
**Given** the same company and three lines of 10.03 with the same tax,
**when** the invoice is computed,
**then** "Round per Tax" gives round(30.09 × 0.21, 2) = round(6.3189, 2) = 6.32,
**and** "Round per Line" gives round(2.1063, 2) × 3 = 2.11 × 3 = 6.33,
**and** the one-cent difference is the reason a country package writes the required method into its
template.
Formula, calculations section 14.

**16.3 The two methods agree on three lines of 10.05.**
**Given** three lines of 10.05 with the same tax,
**when** the invoice is computed,
**then** both methods give 6.33.
Formula, calculations section 14.

**16.4 Every amount is rounded to its own currency.**
**Given** a withholding line whose currency has two decimal places and a company currency with none,
**when** the line is converted,
**then** the line amount is rounded to two places and the company-currency amount to zero places.
Rule FLOC-RULE-139.

**16.5 An integer report column applies the report's integer rounding.**
**Given** a report column declared as an integer figure, a computed value of 1,234.56 and the
integer rounding setting "Nearest",
**when** the report is rendered,
**then** the cell shows 1,235; with the setting "Up" it shows 1,235 and with the setting "Down" it
shows 1,234.
Rule FLOC-RULE-140.

**16.6 A ratio is never rounded first.**
**Given** the proration of scenario 9.15,
**when** the withheld amount is recomputed,
**then** the multiplication and the division are carried out at full precision and one rounding is
applied at the end.
Formula, calculations section 14.

---

## 17. State machine coverage

**17.1 The generic exchange machine, ordinary path.**
**Given** an eligible posted invoice in a country whose flow follows the generic machine,
**when** it is transmitted and the polling job receives an acceptance,
**then** the state moves from empty to the to-send state, to the sent state, to the accepted state,
**and** the registration number, the visual code and any signature are stored, and a confirmation is
posted in the discussion thread.
State machine section 5.

**17.2 The generic exchange machine, refusal and correction.**
**Given** the same invoice,
**when** the polling job receives a refusal,
**then** the state becomes the rejected state, the errors are stored and posted, and after the
invoice is corrected and resent the state returns to the to-send state.
State machine section 5.

**17.3 The Italian machine, delivery to a public body.**
**Given** an Italian invoice to a public-sector counterpart whose exchange state is `processing`,
**when** the polling job reports the outcome `EC01`,
**then** the state becomes `accepted_by_pa_partner`,
**and** with the outcome `EC02` it becomes `rejected_by_pa_partner` and a credit note is required,
**and** when no answer arrives within fifteen days the state becomes
`accepted_by_pa_partner_after_expiry`.
State machine section 6.

**17.4 The Italian machine, rejection clears the payload.**
**Given** an Italian invoice in `processing` with an attached payload and a transaction reference,
**when** a refusal notice arrives,
**then** the state becomes `rejected`, the transaction reference is cleared and the payload
attachment is removed.
State machine section 6.

**17.5 A state the map does not cover clears the field.**
**Given** an Italian invoice in `processing`,
**when** the polling job reports a status the map does not cover,
**then** the state is cleared to empty so that the invoice can be sent again.
State machine section 6.

**17.6 The declaration of intent refuses use before validation.**
**Given** a declaration of intent in the state `draft`,
**when** an invoice names it,
**then** the invoice is refused with "The Declaration of Intent is in draft.",
**and** after the declaration is validated to `active` the invoice is accepted,
**and** once the declaration is `revoked` a new invoice is refused with "The Declaration of Intent
must be active.".
State machine section 7.

**17.7 The declaration of intent threshold warning.**
**Given** a declaration of intent with a threshold of 50,000.00, 45,000.00 already invoiced and
2,000.00 ordered but not yet invoiced,
**when** an invoice of 5,000.00 naming it is prepared,
**then** the remaining amount is 50,000.00 − 45,000.00 − 2,000.00 = 3,000.00, the invoice exceeds it
by 2,000.00, and a warning is shown that names the declaration, the threshold and the excess,
**and** the invoice is not blocked.
State machine section 7.

**17.8 The Hungarian machine, send timeout then confirmation.**
**Given** a Hungarian invoice being uploaded,
**when** the transport does not answer in time,
**then** the state becomes `send_timeout` and the chain index becomes the unknown marker −1,
**and** when the status query later reports success the state becomes `confirmed`.
State machine section 9.

**17.9 The Hungarian machine, annulment.**
**Given** a Hungarian invoice in `confirmed`,
**when** a technical annulment is requested and transmitted,
**then** the state becomes `cancel_sent`, then `cancel_pending` when the administration
acknowledges without approving, then `cancelled` when the annulment is approved,
**and** every invoice of the same chain that carries a state is cancelled with it and their chain
index becomes 0.
State machine section 9.

**17.10 A transmitted Hungarian invoice cannot be reset.**
**Given** a Hungarian invoice in `confirmed`,
**when** a reset to draft is attempted,
**then** it is refused with "Cannot reset to draft or cancel invoice `<name>` because an electronic
document was already sent to NAV!".
State machine section 9.

**17.11 The Basque Country machine, missing certificate.**
**Given** a company with the Basque regime enabled and no signing certificate,
**when** a registration document is transmitted,
**then** it is refused with "Please configure the certificate for TicketBAI." and the document stays
in `to_send`.
State machine section 10.

**17.12 The Basque Country machine, cancellation.**
**Given** an invoice whose registration document is `accepted` and whose derived state is `sent`,
**when** a cancellation document is produced and accepted,
**then** the invoice's derived state becomes `cancelled`.
State machine section 10.

**17.13 The verifiable-invoice machine, batch answer.**
**Given** three submission records in the waiting state, sent as one shipment,
**when** the administration answers `Correcto` for the first, `AceptadoConErrores` for the second and
`Incorrecto` for the third,
**then** their states become `accepted`, `registered_with_errors` and `rejected` respectively,
**and** the invoice of the third shows the derived state `rejected`.
State machine section 11.

**17.14 The verifiable-invoice machine cancels the ledger entry.**
**Given** an invoice whose derived verifiable state becomes `cancelled` and whose ledger entry is
still posted,
**when** the state is written,
**then** the ledger entry is cancelled.
State machine section 11.

**17.15 The Greek machine does not copy a failure onto the invoice.**
**Given** a Greek invoice with no state,
**when** a transmission fails and a document in `invoice_error` is written,
**then** the invoice's derived state stays empty and the failure is visible only on the document and
in the discussion thread.
State machine section 12.

**17.16 The Croatian statuses are independent.**
**Given** a Croatian invoice whose fiscalisation status is `0` and whose intermediary status is
`50`,
**when** the addendum is read,
**then** the invoice is fiscalised with the tax administration and undelivered to the counterpart,
so the seller must deliver a copy by other means, which the delivery channel type `1` records.
State machine section 13.

**17.17 The Danish machine refuses a late cancellation.**
**Given** an invoice whose network state is `processing`,
**when** the electronic document is cancelled,
**then** it is refused with "Cannot cancel an entry that has already been sent to Nemhandel".
State machine section 14.

**17.18 A Danish business response.**
**Given** a delivered inbound invoice,
**when** the company approves it,
**then** a response record with the code `BusinessAccept` is created in `processing`, and when the
network confirms delivery the record becomes `done` and the invoice's own state becomes
`BusinessAccept`.
State machine section 14.

**17.19 The Romanian machine without an index.**
**Given** a Romanian invoice accepted by the portal with no index returned,
**when** the state is written,
**then** it becomes `invoice_not_indexed`, and when the indexing job finds the index it becomes
`invoice_sent`.
State machine section 15.

**17.20 The Romanian machine on validation.**
**Given** a Romanian invoice in `invoice_sent`,
**when** the status job reports validation,
**then** the state becomes `invoice_validated`, the signature, certificate and download keys are
stored, "This invoice has been accepted by the SPV." is posted and the superseded `invoice_sent`
document records are removed.
State machine section 15.

**17.21 The Polish machine maps the answer codes.**
**Given** a Polish invoice in `sent`,
**when** the status job receives the code 200,
**then** the state becomes `accepted`, the national number is stored and "KSeF Status: Success (Code:
200). Invoice accepted." is posted,
**and** with the code 440 the state becomes `rejected` and "KSeF Status: Rejected (Code: 440).
Duplicate invoice." is posted,
**and** with an unmapped code the state does not change and "Unknown status received from KSeF
(Code: `<code>`): `<description>`" is written into the header.
State machine section 16.

**17.22 A rejected Polish invoice is cleared on reset.**
**Given** a Polish invoice in `rejected`,
**when** it is reset to draft,
**then** the status, the national number, the reference, the session identifier and the header are
all cleared.
State machine section 16.

**17.23 Bank account verification reuses the day's result.**
**Given** a supplier with a tax identification number and a bank account, checked this morning with
the result `valid`,
**when** a second payment to the same account is registered today,
**then** the existing record is reused and the register is not called again.
State machine section 16.2.

**17.24 A supplier without a number is not checked.**
**Given** a supplier with no tax identification number,
**when** a payment is registered,
**then** a record with the status `incomplete_partner` is written and no call is made.
State machine section 16.2.

**17.25 The Malaysian machine refuses a late cancellation.**
**Given** a Malaysian document validated 80 hours ago,
**when** a cancellation is requested,
**then** it is refused with "It has been more than 72h since the document validation, you can no
longer cancel it.\nInstead, you should issue a debit or credit note.".
State machine section 17.

**17.26 The Malaysian machine cancels the invoices of an invalid document.**
**Given** a Malaysian document in `in_progress` carrying two invoices,
**when** the portal answers `invalid`,
**then** the document's state becomes `invalid` and both invoices are cancelled.
State machine section 17.

**17.27 The Indian way bill can be reset only from two states.**
**Given** a way bill in `generated`,
**when** a reset to pending is attempted,
**then** it is refused with "Only Delivery Challan and Cancelled E-waybill can be reset to
pending.",
**and** a way bill in `cancel` or in `challan` is reset to `pending` successfully.
State machine section 18.

**17.28 Printing before generation is refused.**
**Given** a way bill in `pending`,
**when** the permit is printed,
**then** it is refused with "Please generate the E-Waybill to print it." or, where the delivery-note
route exists, with "Please generate the E-Waybill or mark the document as a Challan to print it.".
State machine section 18.

**17.29 The Indonesian payment code expires.**
**Given** a payment code created 36 minutes ago and still unpaid,
**when** the clean-up job runs,
**then** the record is removed, because a code older than thirty-five minutes can no longer be paid.
State machine section 19.2.

**17.30 A payment code for an unsupported record kind.**
**Given** a record kind the mechanism does not cover,
**when** a payment code is generated for it,
**then** it is refused with "QRIS capability is not extended to model %s yet!", the placeholder
carrying the technical name of the record kind.
State machine section 19.2.

**17.31 The Turkish counterpart lookup drives the document kind.**
**Given** a Turkish counterpart whose status is `earchive`,
**when** an invoice is issued to it,
**then** an archive invoice is produced and delivered by other means,
**and** for a counterpart whose status is `einvoice` an electronic invoice is sent through the
network to one of its registered aliases.
State machine section 20.

**17.32 The Vietnamese machine reports a payment change.**
**Given** a Vietnamese invoice in `sent`,
**when** the ledger's payment state changes,
**then** the invoice's state becomes `payment_state_to_update`,
**and** after the update is transmitted the state returns to `sent`.
State machine section 21.

**17.33 A cancelled Vietnamese invoice can be reissued.**
**Given** a Vietnamese invoice in `canceled` whose ledger entry is cancelled,
**when** the entry is reset to draft,
**then** every field of the flow is cleared and the state becomes empty, so the document may be
issued again.
State machine section 21.

**17.34 The Taiwanese refund needs the buyer's agreement.**
**Given** a Taiwanese credit note whose refund state is `to_be_agreed`,
**when** the buyer agrees,
**then** the state becomes `agreed` and the credit note may be issued,
**and** when the buyer refuses, the state becomes `disagreed` and the credit note cannot be issued
through the service.
State machine section 22.

**17.35 Demonstration mode marks a Jordanian invoice without transmitting.**
**Given** a Jordanian company in demonstration mode and an invoice in `to_send`,
**when** it is sent,
**then** the payload is produced and attached, nothing leaves the system and the state becomes
`demo`.
State machine section 23.

**17.36 The French reporting flow refuses a send with invalid documents.**
**Given** a flow in `ready` containing two documents with validation errors, on a day that is not
the last day of the grace period,
**when** the flow is sent,
**then** it is refused with "This flow still contains invoices with validation errors. Fix them or
use the 'Send without invalid invoices' button.",
**and** when the "send without the invalid ones" option is used, the flow is sent, the valid
documents are recorded against it and a rectificative flow is opened for the two invalid ones.
State machine section 25.1.

**17.37 The French reporting period status.**
**Given** a flow whose due period runs from 5 April to 15 April,
**when** the status is computed on 1 April, on 10 April and on 20 April,
**then** it is `open`, `grace` and `closed` respectively.
State machine section 25.2; rule FLOC-RULE-136.

**17.38 The French reporting status of an invoice.**
**Given** a posted reportable invoice covered by a flow whose period has not ended,
**when** its reporting status is computed,
**then** it is `pending`,
**and** when the period has ended the invoice shows the flow's own state,
**and** an entry with no reporting kind shows `out_of_scope`,
**and** an entry with a blocking error shows `error`.
State machine section 25.3.

**17.39 A duplicate French flow is not sent twice.**
**Given** a flow whose content is identical to the previous flow of the same scope,
**when** it is sent,
**then** nothing is transmitted and "This flow is identical to the previous flow `<name>`." is
posted once.
State machine section 25.1.

**17.40 A Latin America cheque is handed, then debited.**
**Given** an own cheque of 1,500.00 issued by a posted outbound payment,
**when** the payment is posted,
**then** the cheque's issue state is `handed` and a liquidity item of 1,500.00 exists with the
cheque's payment date as its maturity date,
**and** when the bank statement line is reconciled against it, the issue state becomes `debited`.
State machine section 27.

**17.41 A Latin America cheque is voided.**
**Given** the same cheque in `handed`, reconciled against a customer invoice,
**when** the void action is run,
**then** the payment is unreconciled from the invoice, a reversing entry named "Void check" is
created and posted in the journal of the outstanding item with one line on the payment's destination
account and one line on the outstanding account, both for 1,500.00 with the cheque's maturity date,
currency and counterpart,
**and** the reversing entry is reconciled against the cheque's item and against the payment's
receivable line,
**and** the cheque's issue state becomes `voided`.
State machine section 27.

**17.42 A payment carrying several cheques is split.**
**Given** an outbound payment of 3,000.00 carrying three own cheques of 1,000.00 each with different
payment dates,
**when** the payment is posted,
**then** a splitting entry is posted with one liquidity item per cheque, each named "Check
`<number>` - `<the payment's own display suffix>`" and carrying that cheque's payment date as its
maturity date, plus one counterpart line of 3,000.00,
**and** the counterpart line is reconciled against the payment's own liquidity item.
State machine section 27.

**17.43 A cheque number is padded.**
**Given** a cheque whose number is typed as `45`,
**when** the field is left,
**then** the stored number is `00000045`.
Entities section 3.15.

**17.44 Two issued own cheques may not share a number.**
**Given** an issued own cheque numbered `00000045` on a given payment method line,
**when** a second cheque with the same number is issued on the same method line,
**then** the save is refused by the uniqueness index,
**and** for a newly received third-party cheque the same combination only raises the warning "Other
checks were found with same number, issuer and bank. Please double check you are not encoding the
same check more than once. List of other payments/checks: `<names>`".
State machine section 27.

**17.45 A cheque cannot be received twice.**
**Given** a third-party cheque already in hand in a journal,
**when** an inbound payment that is not a transfer names it again,
**then** the payment is refused with "Some checks are already in hand and can't be received again.
Checks: `<names>`".
State machine section 27.

**17.46 A payment with a debited cheque cannot be reopened.**
**Given** a payment carrying a cheque whose issue state is `debited`,
**when** the payment is cancelled or reset to draft,
**then** it is refused with "You can't cancel or re-open a payment with checks if some check has
been debited or been voided. Checks:" followed by one line per cheque giving its number and its
issue state.
State machine section 27.

**17.47 A mass transfer needs one journal and one currency.**
**Given** three third-party cheques, two in one journal and one in another,
**when** the mass transfer wizard is opened on all three,
**then** it is refused with "All selected checks must be on the same journal and on hand",
**and** with cheques in two currencies the refusal is "All the selected checks must use the same
currency".
State machine section 27.

**17.48 A cheque of a draft payment cannot be moved.**
**Given** a cheque whose creating payment is still draft,
**when** a payment that would move it is posted,
**then** it is refused with "Selected checks \"`<names>`\" are not posted".
State machine section 27.

**17.49 A cheque payment needs an outstanding account.**
**Given** a payment whose method code is `own_checks` and whose journal has no outstanding account,
**when** the payment is posted,
**then** it is refused with "A payment with any Third Party Check or Own Check payment methods needs
an outstanding account".
State machine section 27.

**17.50 The activity state of a country record.**
**Given** an India Electronic Way Bill with an activity due yesterday,
**when** its activity state is read,
**then** it is `overdue`; with an activity due today it is `today` and with one due tomorrow it is
`planned`; with no open activity it is empty.
State machine section 29.

---

## 18. Country scenarios

Each country file carries its own numbered scenarios for the taxes, the fiscal positions, the
identification rules, the documents and the exchange flow of that country. The counts are:

| Country file | Scenarios |
|---|---|
| [Argentina](countries/argentina.md) | 8 |
| [Austria](countries/austria.md) | 3 |
| [Belgium](countries/belgium.md) | 4 |
| [Brazil](countries/brazil.md) | 7 |
| [Chile](countries/chile.md) | 5 |
| [Colombia](countries/colombia.md) | 3 |
| [Ecuador](countries/ecuador.md) | 4 |
| [France](countries/france.md) | 6 |
| [Germany](countries/germany.md) | 5 |
| [India](countries/india.md) | 10 |
| [Italy](countries/italy.md) | 7 |
| [Luxembourg](countries/luxembourg.md) | 3 |
| [Mexico](countries/mexico.md) | 6 |
| [Netherlands](countries/netherlands.md) | 3 |
| [Peru](countries/peru.md) | 5 |
| [Portugal](countries/portugal.md) | 3 |
| [Spain](countries/spain.md) | 6 |
| [Switzerland](countries/switzerland.md) | 4 |
| [United Kingdom](countries/united-kingdom.md) | 5 |
| [Uruguay](countries/uruguay.md) | 4 |

The remaining country files, and the catalogue of every shipped package, carry the configuration a
rebuild must reproduce for the countries that have no flow of their own; they are listed in
[country-packages.md](country-packages.md).
