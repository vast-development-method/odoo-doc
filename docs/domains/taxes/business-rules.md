# Taxes — Business rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behaviour of
the tax domain, with the exact user-facing message the system produces. Placeholders are written
out in words between angle brackets.

Rules are grouped by the entity or the moment they apply to. A rule marked **on save** runs
whenever the listed fields change, on creation and on modification alike.

---

## 1. Tax — structural rules

### 1.1 Name uniqueness

**On save** of the company, name, tax type, tax scope or country.

Two taxes may not share the quadruple (name, tax type, tax scope, country) within the same company
tree, unless their tax type is `none`. The search is run in batches of one hundred records and
walks the whole tree rooted at the tax's root company, with record-level filtering bypassed.

> Tax names must be unique!
> - &lt;the duplicate's name&gt; in &lt;the duplicate's company name&gt;

with one dash line per duplicate found.

**Edge case.** Two taxes of type `none` may share a name; they are only ever selected through a
Group of Taxes, so the ambiguity is harmless.

### 1.2 Tax group country consistency

**On save** of the tax group.

When the chosen tax group has a country, it must be the tax's own country.

> The tax group must have the same country_id as the tax using it.

### 1.3 Cash basis transition account

**On save** of the exigibility or the transition account.

When the exigibility is "based on payment", the transition account must allow reconciliation. The
check is skipped while a chart of accounts template is being loaded.

> The cash basis transition account needs to allow reconciliation.

The transition account may additionally not be of the receivable or payable kinds; this is enforced
by the field's own restriction rather than by a message.

### 1.4 Children of a Group of Taxes

**On save** of the children or the tax type.

1. The children graph may not contain a cycle.
   > Recursion found for tax “&lt;the tax's name&gt;”.
2. Every child's tax type must be `none` or equal to the parent's, and every child's tax scope must
   be empty or equal to the parent's.
   > The application scope of taxes in a group must be either the same as the group or left empty.
3. No child may itself be a Group of Taxes.
   > Nested group of taxes are not allowed.

### 1.5 Changing the company

**On save** of the company, unless the record is being created.

The company may not change while any journal item references the tax, either as a base tax or as
the originator of a tax item, outside the new company's tree.

> You can't change the company of your tax since there are some journal items linked to it.

### 1.6 Deletion

A tax may not be deleted while it is in use — that is, while at least one journal item or one
reconciliation model line references it.

> You cannot delete taxes that are currently in use. Consider archiving them instead.

The check is skipped when the deletion is part of uninstalling a capability.

### 1.7 Duplication

Duplicating a tax names the copy *"&lt;the original name&gt; (copy)"* unless a name is supplied. The
distribution lines are copied, including their tags.

### 1.8 Rules added by the withholding capability

**On save** of the computation kind or the "withhold on payment" flag:

> Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations.

**On change in the form** of the "withhold on payment" flag, when it becomes true: the exigibility
is forced to "based on invoice" and the price-inclusion override is forced to "tax excluded".

**On change in the form** of the amount, when it becomes zero or positive: the "withhold on
payment" flag is cleared, because the field is hidden for a non-negative tax.

### 1.9 Rules added by the custom formula capability

**On save** of the computation kind or the formula, when the kind is "custom formula", the formula
must satisfy the grammar of `calculations.md` section 4.6. The possible messages are:

> Invalid formula

> Field '&lt;the field name&gt;' is not accessible

> Invalid AST node: &lt;the construct's name&gt;

> Only int, float or None are allowed as constant values

> Unknown identifier: &lt;the name&gt;

> Only read access to identifiers is allowed

> Unknown function call

> Kwargs are not allowed

> Only product['string'] or uom['string'] read-access is allowed

At evaluation time, one further message:

> Only primitive types are allowed in python tax formula context.

Uninstalling the capability rewrites every custom-formula tax to a percentage tax and archives it.

---

## 2. Tax Distribution — structural rules

**On save** of the invoice distribution, the refund distribution or the full distribution.

A Group of Taxes that has **no** distribution line at all is exempt from all six rules below.

| # | Rule | Message |
|---|---|---|
| 1 | Each of the two lists contains exactly one line of kind `base`. | Invoice and credit note distribution should each contain exactly one line for the base. |
| 2 | The two lists have the same number of lines. | Invoice and credit note distribution should have the same number of lines. |
| 3 | Each list contains at least one line of kind `tax`. | Invoice and credit note repartition should have at least one tax repartition line. |
| 4 | Walking both sorted lists in parallel, the line at each index has the same kind and the same percentage in both. | Invoice and credit note distribution should match (same percentages, in the same order). |
| 5 | The sum of the **positive** factors of the invoice `tax` lines equals one, compared at two decimal places. | Invoice and credit note distribution should have a total factor (+) equals to 100. |
| 6 | When any negative factor exists among the invoice `tax` lines, the sum of the negative factors equals minus one, compared at two decimal places. | Invoice and credit note distribution should have a total factor (-) equals to 100. |

Both lists are sorted by (sequence, identifier) before rules 4 to 6 are applied.

**Field-level restrictions.**

- The account of a distribution line may not be of the receivable, payable or off-balance kinds.
- Only tags whose applicability is `taxes` may be attached.
- The tag chooser is further restricted to tags whose country is empty, the company's fiscal
  country, or one of the countries in which the company holds a foreign registration.
- Switching a line's kind to `base` clears its account.

**Invariant.** The engine relies on rule 5: the sum of the positive shares of a tax amount is the
tax amount. Rule 6 guarantees that a reverse charge nets to zero. Rule 4 guarantees that a credit
note redistributes an amount exactly as the invoice it reverses did.

---

## 3. Tax Group

**On save** — no constraint of its own. The consistency rule lives on the tax (section 1.2).

**Invariant.** Every tax has a tax group; the field is required and precomputed.

---

## 4. Fiscal Position

### 4.1 Postal code range

**On save** of either bound.

Both bounds must be set together, and the upper bound must not be smaller than the lower bound
when compared as text.

> Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From".

**Edge case.** The comparison is textual, which is why both bounds are left-padded with zeros to a
common length when they are entirely numeric.

### 4.2 Foreign registration

**On save** of the country, the country group, the states or the foreign registration number, when
a foreign registration number is present.

| # | Rule | Message |
|---|---|---|
| 1 | A country is set. | The country of the foreign value-added tax number could not be detected. Please assign a country to the fiscal position. |
| 2 | When the country equals the company's fiscal country and that country has states, at least one state is selected. | You cannot create a fiscal position with a foreign value-added tax within your fiscal country without assigning it a state. |
| 3 | When both a country and a country group are set, the country belongs to the group. | You cannot create a fiscal position with a country outside of the selected country group. |
| 4 | No other fiscal position of the same company already holds a **different** foreign registration number for the same country. | A fiscal position with a foreign value-added tax already exists in this country. |

Rule 4 permits several fiscal positions to carry the **same** number for the same country.

### 4.3 The registration number itself

Writing the foreign registration number runs the validation pipeline of `calculations.md`
section 15 with the record label *"fiscal position [&lt;the fiscal position's name&gt;]"*. While the
form is open the check runs in silent mode, so the number is normalised without an error; on save
it runs in error mode.

### 4.4 Account mapping uniqueness

The triple (fiscal position, source account, destination account) is unique.

> An account fiscal position could be defined only one time on same accounts.

Both accounts are required and must belong to the fiscal position's company tree.

---

## 5. Documents carrying taxes

### 5.1 Tax country consistency

**On save** of the lines, the fiscal position or the company of an entry.

Let the set of impacted countries be the countries of every base tax and every originator tax on
the entry's items. When that set is non-empty and differs from the entry's tax country:

- when the entry has a fiscal position and the impacted countries differ from the fiscal position's
  country:
  > This entry contains taxes that are not compatible with your fiscal position. Check the country set in fiscal position and in your tax configuration.
- otherwise:
  > This entry contains one or more taxes that are incompatible with your fiscal country. Check company fiscal country in the settings and tax country in taxes configuration.

The entry's tax country is recomputed before the check runs. It is the fiscal position's country
when the fiscal position carries a foreign registration number, and the company's fiscal country
otherwise.

**Edge case.** A single entry may not mix taxes of two countries, even when both are enabled for the
company, because the comparison is between the whole set and a single country.

### 5.2 Mixing exigibilities on one line

**On save** of the base taxes or the distribution line of a journal item.

A journal item may not carry both a tax exigible on payment and a tax exigible on invoice when the
two share a report tag on their `base` distribution line for the applicable document kind.

The check is in two stages:

1. Split the item's base taxes into those exigible on payment and the others. Collect the `base`
   tags of each group from the applicable distribution (refund when the item is a refund line,
   invoice otherwise). If the two tag sets intersect, refuse.
2. Otherwise, when the item is itself a tax item, compare its own distribution line's tags against
   the **opposite** group's base tags — the invoice-exigible group when the item's tax is exigible
   on payment, the payment-exigible group otherwise. If they intersect, refuse.

> Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag.

**Why.** A shared tag would have to appear both on the original entry and on the cash basis entry,
so the same base would be reported twice. The documented workaround is a Group of Taxes whose
children have the tax type `none` and only one of which carries the common tag.

### 5.3 Deductibility

**On save** of the deductible percentage of a journal item.

1. Only a vendor document may carry a deductibility other than one hundred percent, compared at two
   decimal places.
   > Only vendor bills allow for deductibility of product/services.
2. The percentage must lie between zero and one hundred inclusive.
   > The deductibility must be a value between 0 and 100.

### 5.4 An off-balance account may not carry taxes

**On save** of an account's reconcilable flag, kind or default taxes.

> An Off-Balance account can not have taxes

---

## 6. Locking

### 6.1 The tax lock date

**On save** of a posted journal item.

For every item of a **posted** entry, when the entry's date falls at or before the effective hard
tax lock date of the company **and** the item affects the tax report, the change is refused.

> The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: &lt;the list of violated lock dates and their names&gt;.

**An item affects the tax report** when it carries at least one base tax, **or** it is a tax item
(it has an originator tax), **or** it carries at least one tag whose applicability is `taxes`.

An entry affects the tax report when any of its items does.

The lock date consulted is the "hard" one, meaning lock exceptions granted to the acting user are
**not** honoured for this particular check.

### 6.2 The warning message on a draft entry

A draft entry whose date falls in a locked period and which affects the tax report shows a
non-blocking message computed from the entry's accounting date and that fact. Posting will move
the entry to a permitted date or refuse, according to the general ledger's lock rules.

### 6.3 Cash basis entries may not be reset to draft

> You cannot reset to draft a tax cash basis journal entry.

The check looks at both the originating reconciliation link and the origin document link, because
undoing a reconciliation clears the first but not the second.

### 6.4 Cash basis entries may not be deleted while posted

A posted entry that is a cash basis entry, or that is the origin of one, cannot be unlinked; the
general deletion path reverses it instead.

---

## 7. Withholding

| # | Rule | Message |
|---|---|---|
| 1 | A withholding line's base amount must be strictly positive, compared at the line currency's precision. | The base amount of a withholding tax line must be above 0. |
| 2 | A withholding line's account may not be a liquidity account of the owner's journal, nor any outstanding account of that journal's payment method lines, nor the owner's own outstanding account, nor the company's internal transfer account. | The account "&lt;the account's display name&gt;" is not valid to use on withholding lines. |
| 3 | Before any sequence value is consumed, every line must have either a number or a tax carrying a withholding sequence. | Please enter the withholding number for the tax &lt;the tax's name&gt; |
| 4 | The net amount of a registered payment may not be negative. | The withholding net amount cannot be negative. |
| 5 | Building the journal items requires every line in the call to belong to the same payment. | All withholding lines in self must have the same payment. |
| 6 | The same, for the register-payment wizard. | All withholding lines in self must have the same payment register. |

**Invariant.** The withholding amount of a line is always derived from its base by the same ratio
as the original amounts, so editing the base rescales the amount; editing the amount by hand
overrides it and the override survives until the base changes again.

**Invariant.** A withholding tax never appears in a document total: the engine installs a filter
that removes every withholding tax, unless the caller explicitly asked for them.

---

## 8. Cash basis

| # | Rule | Behaviour |
|---|---|---|
| 1 | The company must have a cash basis journal at the moment the first cash basis entry is needed. | Refused: *There is no tax cash basis journal defined for the '&lt;company name&gt;' company.\nConfigure it in Accounting/Configuration/Settings* |
| 2 | A document that mixes currencies across its term lines and its deferred lines produces **no** cash basis entry at all. | Silently skipped. |
| 3 | A document with no term line (no receivable or payable item) produces no cash basis entry. | Silently skipped. |
| 4 | A partial whose amount rounds to zero in the relevant currency is skipped. | Silently skipped. |
| 5 | On the last partial of a now fully paid document, each tax share is forced to the tax item's remaining residual rather than its computed percentage. | Guarantees the sum matches to the cent. |
| 6 | The cash basis switch on a company cannot be turned off while any tax of that company is exigible on payment. | The switch is forced back on and a warning is shown: *You cannot disable this setting because some of your taxes are cash basis. Modify your taxes first before disabling this setting.* |

---

## 9. Tax identification numbers

### 9.1 The explicit "no number" marker

A number consisting of the single character solidus means "explicitly no number" and is always
accepted. Any other single character is refused in error mode:

> To explicitly indicate no (valid) value-added tax, use '/' instead.

(The message ends with a trailing space.)

### 9.2 The failure message

> The &lt;label&gt; number [&lt;the number&gt;] for &lt;the record label&gt; does not seem to be valid.
> Note: the expected format is &lt;the example&gt;

and, when the record has no name:

> The &lt;label&gt; number [&lt;the number&gt;] does not seem to be valid.
> Note: the expected format is &lt;the example&gt;

The label is the word *"VAT"*, replaced by the country's own label for the number when the checked
country is the acting company's country and that country defines one. The note is omitted when the
country has no example. When the failure happened on a second, union-wide attempt, the message is
followed by a blank line and:

> If you are trying to input a European number, this is the expected format: &lt;the example&gt;

### 9.3 The doubled prefix

A number that begins with its own country prefix twice is refused even when the remainder would
validate.

### 9.4 Suppressing the check

The check is suppressed when: the caller passes the *off* mode; the caller sets the
"skip number validation" context marker (used by pushes from external platforms where the
installation has no control over the numbers); or the country has no check routine at all.

### 9.5 Editing a number that is already used

- A partner's country may not be changed once a non-draft invoice exists for that partner.
- A partner's number may not be changed once a non-draft invoice exists for that partner or any of
  its children under the same commercial partner.
- A partner may not be attached as an invoicing address of another partner when the two carry
  different numbers and the partner already has journal items:
  > You cannot set a partner as an invoicing address of another if they have a different &lt;the country's label for the number&gt;.

### 9.6 Cross-border verification

- A partner whose parent carries the same number inherits the parent's verification flag and no
  request is sent.
- When **no** company at all has the verification switch on, the flag is set to false with no call.
- The relay endpoint must be one of the two known addresses:
  > Invalid IAP the cross-border registration checking service endpoint
- During a file import, the recomputation of the flag is cancelled entirely.
- The callback route verifies the signed token before applying a status; a token that does not
  verify is logged and ignored with no change.

---

## 10. Permission checks

Access rights are listed in full in `configuration.md` section 5. The rules that matter
operationally are:

| Operation | Required group |
|---|---|
| Read a tax, a tax group, a distribution line, a fiscal position | any internal user |
| Create, change or delete a tax, a tax group or a distribution line | accounting administrator |
| Create, change or delete a fiscal position or an account mapping | accounting administrator |
| Create, change or delete an account tag | full accounting user |
| Read an account tag | the invoicing group or the read-only accounting group |
| See the "base affected by previous taxes" field | developer mode |
| See the analytic flag on a tax | the analytic accounting group |
| Reach the tax group menu | developer mode |
| Reach the cash rounding menu | the cash rounding group |
| Change a company's tax settings | the settings administrator group |

Every one of the four tax entities also carries a company record rule: a user only sees records
whose company is the acting company or one of its ancestors. The distribution line's rule also
admits records with no company at all.

---

## 11. Invariants the engine must preserve

These are not enforced by a message; an implementation that breaks one produces wrong numbers.

1. **The sum of the rounded tax amounts of a group equals the rounded sum of the raw amounts.**
   Guaranteed by the redistribution pass of `calculations.md` section 7.5.
2. **The sum of the base journal items' balances equals the rounded untaxed total of the document.**
   Guaranteed by the delta of section 7.6; the balance of a base item is the rounded untaxed total
   **plus** the delta, never the rounded total alone.
3. **The sum of the distribution shares of one tax equals that tax's rounded amount.** Guaranteed by
   the residue redistribution of section 8.2 together with structural rule 5 of section 2.
4. **A reverse charge nets to zero in the ledger.** Guaranteed by structural rule 6 of section 2
   together with the mirror record the engine creates.
5. **The invoice and the credit note of the same amounts produce equal and opposite journal items.**
   Guaranteed by structural rule 4 of section 2 together with the sign of the document.
6. **A cash basis entry never makes more tax exigible than the original document carried.**
   Guaranteed by the residual-based correction on the last partial.
7. **Computation keys never leak cents across subsets.** Guaranteed by including the key in both
   redistribution groupings.
8. **The engine's two implementations agree.** Every step marked *mirrored* in `calculations.md`
   exists twice and must produce identical numbers; a divergence shows up as a total that changes
   when the document is saved.
9. **A tax that is not stored yet sorts before saved taxes of the same sequence**, because its
   identifier is absent. An implementation must reproduce that ordering or an unsaved edit will
   preview differently from the saved result.
10. **Rounding is always half away from zero** unless a cash rounding configuration says otherwise.

---

## 12. Edge cases and their defined behaviour

| Situation | Behaviour |
|---|---|
| A percentage batch whose total percentage is exactly minus one hundred percent | The extraction factor is zero and every tax in the batch has an amount of zero, instead of a division by zero. |
| A division batch whose total percentage is exactly one hundred percent | The multiplier is one, so the tax equals the base, instead of a division by zero. |
| A custom formula that divides by zero | Evaluates to zero. |
| A base line with no tax at all | The untaxed total and the total with taxes both equal the raw base; the totals block still counts the line in the untaxed amount. |
| A tax amount that rounds to zero in both currencies | No journal item is produced, unless the grouping key carries the "keep zero line" marker. |
| A negative unit price with a fixed tax | The fixed tax amount is negated, so the tax follows the sign of the line. |
| A quantity of zero | The raw base is zero; percentage and division taxes give zero; a fixed tax also gives zero because it multiplies by the quantity. |
| A currency with no decimal places | Every rounding step is one; the smooth distribution allocates whole units. |
| A rate of zero on a base line | Every company-currency amount is zero. |
| Cash rounding with the "adjust the biggest tax" strategy on a document with no tax | The rounding is abandoned and the delta reset to zero. |
| A group of taxes whose children were all removed by a filter | The group contributes nothing; no item names it. |
| A tax whose distribution line has no account | The tax item is posted on the **base line's own account**. |
| Two different taxes producing items with the same grouping key | They are merged into one journal item; the grouping key includes the distribution line, so this only happens for the same distribution line. |
| A document reversed twice | The refund flag derived from the sign is inverted once per reversal, so the second reversal behaves like the original. |
| A fiscal position with no tax at all | Mapping keeps only the taxes that belong to **no** fiscal position; every tax bound to some fiscal position is dropped. |
| A replaced tax that maps to several replacements | All of them are applied to the line. |
| A postal code range written with different lengths | Both bounds are zero-padded to the longer length before comparison, but only when they are entirely numeric. |

---

## 13. Index of every user-facing message

Every message the tax domain can produce, in one place, with the condition that triggers it and
the kind of refusal. A *validation* refusal rolls the whole operation back; a *warning* is shown
without blocking.

| Message | Kind | Triggered by |
|---|---|---|
| Tax names must be unique!\n- &lt;name&gt; in &lt;company&gt; | validation | section 1.1 |
| The tax group must have the same country_id as the tax using it. | validation | section 1.2 |
| The cash basis transition account needs to allow reconciliation. | validation | section 1.3 |
| Recursion found for tax “&lt;name&gt;”. | validation | section 1.4 |
| The application scope of taxes in a group must be either the same as the group or left empty. | validation | section 1.4 |
| Nested group of taxes are not allowed. | validation | section 1.4 |
| You can't change the company of your tax since there are some journal items linked to it. | operation refused | section 1.5 |
| You cannot delete taxes that are currently in use. Consider archiving them instead. | validation | section 1.6 |
| Invoice and credit note distribution should each contain exactly one line for the base. | validation | section 2 rule 1 |
| Invoice and credit note distribution should have the same number of lines. | validation | section 2 rule 2 |
| Invoice and credit note repartition should have at least one tax repartition line. | validation | section 2 rule 3 |
| Invoice and credit note distribution should match (same percentages, in the same order). | validation | section 2 rule 4 |
| Invoice and credit note distribution should have a total factor (+) equals to 100. | validation | section 2 rule 5 |
| Invoice and credit note distribution should have a total factor (-) equals to 100. | validation | section 2 rule 6 |
| Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From". | validation | section 4.1 |
| The country of the foreign VAT number could not be detected. Please assign a country to the fiscal position. | validation | section 4.2 rule 1 |
| You cannot create a fiscal position with a foreign VAT within your fiscal country without assigning it a state. | validation | section 4.2 rule 2 |
| You cannot create a fiscal position with a country outside of the selected country group. | validation | section 4.2 rule 3 |
| A fiscal position with a foreign VAT already exists in this country. | validation | section 4.2 rule 4 |
| An account fiscal position could be defined only one time on same accounts. | database constraint | section 4.4 |
| A tag with the same name and applicability already exists in this country. | database constraint | `entities.md` section 6.2 |
| You cannot delete this account tag (&lt;name&gt;), it is used on the chart of account definition. | operation refused | `configuration.md` section 4.1 |
| This entry contains taxes that are not compatible with your fiscal position. Check the country set in fiscal position and in your tax configuration. | validation | section 5.1 |
| This entry contains one or more taxes that are incompatible with your fiscal country. Check company fiscal country in the settings and tax country in taxes configuration. | validation | section 5.1 |
| Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag. | validation | section 5.2 |
| Only vendor bills allow for deductibility of product/services. | validation | section 5.3 |
| The deductibility must be a value between 0 and 100. | validation | section 5.3 |
| An Off-Balance account can not have taxes | operation refused | section 5.4 |
| The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: &lt;lock dates&gt;. | operation refused | section 6.1 |
| You cannot reset to draft a tax cash basis journal entry. | operation refused | section 6.3 |
| There is no tax cash basis journal defined for the '&lt;company&gt;' company.\nConfigure it in Accounting/Configuration/Settings | operation refused | section 8 rule 1 |
| You cannot disable this setting because some of your taxes are cash basis. Modify your taxes first before disabling this setting. | warning | section 8 rule 6 |
| Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations. | operation refused | section 1.8 |
| The base amount of a withholding tax line must be above 0. | operation refused | section 7 rule 1 |
| The account "&lt;account&gt;" is not valid to use on withholding lines. | operation refused | section 7 rule 2 |
| Please enter the withholding number for the tax &lt;tax&gt; | operation refused | section 7 rule 3 |
| The withholding net amount cannot be negative. | operation refused | section 7 rule 4 |
| All withholding lines in self must have the same payment. | internal assertion | section 7 rule 5 |
| All withholding lines in self must have the same payment register. | internal assertion | section 7 rule 6 |
| Invalid formula | validation | section 1.9 |
| Field '&lt;field&gt;' is not accessible | validation | section 1.9 |
| Invalid AST node: &lt;construct&gt; | validation | section 1.9 |
| Only int, float or None are allowed as constant values | validation | section 1.9 |
| Unknown identifier: &lt;name&gt; | validation | section 1.9 |
| Only read access to identifiers is allowed | validation | section 1.9 |
| Unknown function call | validation | section 1.9 |
| Kwargs are not allowed | validation | section 1.9 |
| Only product['string'] or uom['string'] read-access is allowed | validation | section 1.9 |
| Only primitive types are allowed in python tax formula context. | validation | section 1.9 |
| To explicitly indicate no (valid) VAT, use '/' instead. | validation | section 9.1 |
| The &lt;label&gt; number [&lt;number&gt;] for &lt;record&gt; does not seem to be valid. \nNote: the expected format is &lt;example&gt; | validation | section 9.2 |
| The &lt;label&gt; number [&lt;number&gt;] does not seem to be valid. \nNote: the expected format is &lt;example&gt; | validation | section 9.2 |
| If you are trying to input a European number, this is the expected format: &lt;example&gt; | appended | section 9.2 |
| Invalid IAP VIES endpoint | operation refused | section 9.6 |
| You cannot set a partner as an invoicing address of another if they have a different &lt;label&gt;. | operation refused | section 9.5 |
| Update with children taxes that are child of multiple parents is not supported. | operation refused | `workflows.md` section 9 |
| The VIES check is pending. The status will be updated soon. | logged message | `state-machines.md` section 5.2 |
| The VIES check failed. Please check the Tax ID manually. | logged message | `state-machines.md` section 5.2 |
| The Intra-Community validity has been updated to: &lt;status&gt;. | logged message | `state-machines.md` section 5.2 |
| Untaxed Amount | label | the default subtotal name in the totals block |
| &lt;name&gt; (copy) | label | the name of a duplicated tax |
| WH Tax: &lt;name&gt; | label | a withholding tax journal item |
| WH Base: &lt;names&gt; | label | a withholding base journal item |
| WH Base Counterpart: &lt;names&gt; | label | its counterpart |
| private part (taxes) | label | the non-deductible tax journal item |
| Reversal of: &lt;number&gt; | label | the reference of a reversed cash basis entry |
| &lt;amount&gt; Incl. Taxes / &lt;amount&gt; Excl. Taxes / &lt;amount&gt; Tax Withheld | label | the product price hint |
| &lt;name&gt; taxes | label | the title of the window opened by a fiscal position's Taxes button |

---

## 14. Order in which the validations run

When several rules could refuse the same operation, the order matters for which message the user
sees. The order is determined by when each check is evaluated.

1. **Field-level restrictions** — the account kinds allowed on a distribution line, the tax types
   allowed among a group's children, the tag applicability — are enforced by the field itself and
   fail first.
2. **Value normalisers** run next: the postal code padding, the tax identification number
   normalisation, the wrapping of a plain description in a block element, the dispatch of the two
   distribution lists into the single stored list.
3. **Record constraints** run when the record is written: uniqueness, the structural distribution
   rules, the group rules, the fiscal position rules.
4. **Database constraints** run at the same moment: the tag uniqueness, the account mapping
   uniqueness.
5. **Cross-record constraints** run on the entry: the tax country consistency, the mixed
   exigibility rule, the deductibility rules.
6. **Locking** runs last, when the entry is posted or when a posted entry is changed.

A consequence: an attempt to save a tax with both a duplicate name and a broken distribution
reports the **name** problem, because the uniqueness constraint is declared before the
distribution one.

---

## 15. Concurrency

| Situation | Behaviour |
|---|---|
| Two users recompute the same draft entry at once | The second write recomputes from the state the first left; the redistribution is deterministic, so the result is the same whichever order they run in. |
| A sequence value is drawn for a withholding certificate | The value is drawn inside the transaction that builds the payment entry; a rollback returns it only when the sequence is configured to be gapless. |
| Two reconciliations of the same invoice are created at once | Each produces its own cash basis entry; the last-partial correction only fires on the partial that settles the document, so at most one of them applies it. |
| The cross-border verification credentials are generated | They are written in their **own** transaction, so an error later in the current transaction cannot lose them; a concurrent generation is detected by re-reading inside that transaction. |
| A tag is renamed while another user attaches it | The rename changes the tag's name, not its identity, so the attachment survives. |
