# Financial Reporting — Business Rules

This file lists every validation, constraint, invariant, error message, permission check, locking
rule and edge-case behavior of the Financial Reporting domain. Error messages are given exactly as
the system produces them, with placeholders written out in words.

---

## 1. Index of rules

| Number | Subject | Kind | Enforced when |
|---|---|---|---|
| R-01 | A root report may not itself be a variant | validation | writing the root report link |
| R-02 | A parent line must precede its children | validation | writing the report's line set |
| R-03 | Sections may not have sections | validation | writing the section set |
| R-04 | Country required for country availability | validation | writing the availability condition or the country |
| R-05 | A report with variants may not be deleted | deletion guard | deleting a report |
| R-06 | Duplicating a report rewrites codes and formulas | invariant | duplicating |
| R-07 | Changing a report's country moves or duplicates its tags | invariant | writing the country |
| L-01 | A line's code is unique within its report | uniqueness | writing a line |
| L-02 | A line may not be its own parent | validation | writing the parent link |
| L-03 | A line may not have both children and a grouping key | validation | writing the parent link |
| L-04 | A line's report is its parent's report | invariant | writing the parent link |
| L-05 | A line's level derives from its parent's level | invariant | writing the parent link |
| L-06 | Deleting a line orphans its children, it does not delete them | invariant | deleting a line |
| E-01 | An expression's label is unique on its line | uniqueness | writing an expression |
| E-02 | A record filter expression must carry a subformula | database check | writing an expression |
| E-03 | Aggregation and external engines forbid grouping | validation | writing the engine or the grouping |
| E-04 | The formula must match its engine's grammar | validation | writing the formula |
| E-05 | The carry-over target field requires a carry-over label | validation | writing the carry-over target |
| E-06 | The carry-over target must name an applied-carry-over label | validation | writing the carry-over target |
| E-07 | A carry-over target must be resolvable | runtime | closing a period |
| E-08 | A cross-report clause must be well formed, resolvable and not self-referential | runtime | expanding the dependency graph |
| E-09 | Aggregation details may only be requested for an aggregation expression | runtime | expanding the dependency graph |
| E-10 | Formula and subformula are stored whitespace-normalized | invariant | writing |
| T-01 | A tax tag is unique per name, applicability and country | uniqueness | creating a tag |
| T-02 | The three cash-flow tags may not be deleted | deletion guard | deleting a tag |
| T-03 | A tag still used by Journal Items is archived, not deleted | invariant | deleting the last expression that uses it |
| T-04 | A tag still used by another expression is neither archived nor deleted | invariant | deleting an expression |
| T-05 | One tag per formula name, never a signed pair | invariant | creating or renaming |
| X-01 | A manual figure requires an editable subformula | permission | writing a cell |
| X-02 | A manual figure requires the accounting manager group | permission | writing a cell |
| X-03 | An external value is company-scoped and required | validation | writing |
| X-04 | Re-closing a period overwrites carry-over values, never doubles them | invariant | closing |
| K-01 | A write into a locked period is refused | locking | writing a manual figure, posting the closing entry |
| K-02 | Validating a return moves the tax return lock date forward only | invariant | validating |
| K-03 | The hard lock date admits no exception | locking | any write |
| K-04 | A hashed entry's hashed fields may not be edited | locking | editing a Journal Entry or Item |
| C-01 | The tax return journal must exist | precondition | validating a return |
| C-02 | The tax payable and receivable accounts must exist | precondition | validating a return |
| C-03 | The closing entry balances by construction | invariant | validating |
| C-04 | The closing entry carries no tax, no tag and no analytic distribution | invariant | validating |
| C-05 | A closing entry in a restricted journal may only be reversed | locking | cancelling or resetting |
| I-01 | The integrity check requires the accounting user group | permission | running the check |
| I-02 | A corrupted prefix stops being scanned | invariant | running the check |
| I-03 | A protected audit trail message may not be deleted or altered | locking | deleting or writing a message |
| I-04 | A sequence gap breaks the hash chain | runtime | hashing |
| A-01 | Read access to definitions follows the accounting groups | permission | reading |
| A-02 | Only the accounting manager group may change definitions | permission | writing |
| A-03 | A report is offered only where its availability condition holds | permission | listing reports |

---

## 2. Report Definition rules

### R-01 — A root report may not itself be a variant

*Condition:* the report named in the root report link must have an empty root report link of its
own. Variants are one level deep.

*Message:* **Only a report without a root report of its own can be selected as root report.**

*Why:* the variant selector shows a root and its variants as one flat list; a chain of roots would
make the list ambiguous.

### R-02 — A parent line must precede its children

*Condition:* walking the report's lines in ascending sequence, then ascending identifier, every
line whose parent is set must be encountered after its parent.

*Message:* **Line "the line name" defines line "the parent line name" as its parent, but appears
before it in the report. The parent must always come first.**

*Why:* the rendering walks the flat, ordered line list once. A child before its parent would
render above it.

### R-03 — Sections may not have sections

*Condition:* no report listed in the sections of a report may itself have a non-empty section set.

*Message:* **The sections defined on a report cannot have sections themselves.**

### R-04 — Country required for country availability

*Condition:* if the availability condition is `country`, the country link must not be empty.

*Message:* **The Availability is set to 'Country Matches' but the field Country is not set.**

*Related behavior:* changing the availability condition away from `country` clears the country
link through an on-change rule, so the two fields cannot drift apart from the user interface.

### R-05 — A report with variants may not be deleted

*Condition:* the report's variant set must be empty.

*Message:* **You can't delete a report that has variants.**

*Workaround for the user:* archive it, or delete the variants first.

### R-06 — Duplicating a report rewrites codes and formulas

*Invariant:* after duplicating a report, no aggregation formula and no subformula of the copy
refers to a line of the original. Every occurrence of an original line code, delimited by
non-word characters on both sides, has been replaced by the copied code, which is the original
code with `_COPY` appended as many times as needed for uniqueness.

*Consequence to test:* a copy renders exactly the same figures as its original, and changing the
original's lines afterwards does not change the copy.

### R-07 — Changing a report's country moves or duplicates its tags

*Invariant:* after changing a report's country, every tax tag expression of the report matches a
tag in the **new** country with the same name. Tags used exclusively by the reports being changed
are moved; tags shared with an unchanged report are left in place and new ones are created.

*Consequence to test:* the figures of an unchanged report that shared tags are unaffected.

---

## 3. Report Line rules

### L-01 — A line's code is unique within its report

*Condition:* the pair (report, code) is unique. An empty code may repeat.

*Message:* **A report line with the same code already exists.**

### L-02 — A line may not be its own parent

*Message:* **Line "the line name" defines itself as its parent.**

*Note:* deeper cycles (line A parent of B, B parent of A) are prevented by R-02, because at least
one of the two must precede the other.

### L-03 — A line may not have both children and a grouping key

*Condition:* a line may not be created or re-parented under a parent that has an authored
grouping key or a user grouping key.

*Message:* **A line cannot have both children and a groupby value (line 'the parent line
name').**

*Why:* a grouped line's sub-lines are generated from the ledger; authored children would compete
with them for the same level.

### L-04 — A line's report is its parent's report

*Invariant:* setting a parent sets the report. A root line must be given a report explicitly; a
line without either is rejected because the report link is required.

### L-05 — A line's level derives from its parent's level

*Invariant:*

```formula
level = 1                        when the line has no parent
level = parent_level + 3         when parent_level = 0
level = parent_level + 2         otherwise
```

The level may be overridden by hand; overriding it does not move the line in the tree, only its
indentation.

### L-06 — Deleting a line orphans its children

*Invariant:* deleting a line empties the parent link of its children, which become root lines of
the same report. Their expressions and their own children survive.

*Consequence:* deleting an intermediate heading flattens the tree; it does not remove data. A
rebuild that cascades the deletion to children changes observable behavior.

*Side effect:* the line's own expressions **are** deleted explicitly, which runs the tag
housekeeping of T-03 and T-04.

---

## 4. Report Expression rules

### E-01 — An expression's label is unique on its line

*Message:* **The expression label must be unique per report line.**

*Why:* a column selects an expression by label; two expressions with the same label on one line
would make the cell ambiguous.

### E-02 — A record filter expression must carry a subformula

*Condition:* enforced at the database level: the engine is not `domain`, or the subformula is not
empty.

*Message:* **Expressions using 'domain' engine should all have a subformula.**

### E-03 — Aggregation and external engines forbid grouping

*Condition:* an expression whose engine is `aggregation` or `external` may not live on a line
that has an authored grouping key or a user grouping key.

*Message:* **Groupby feature isn't supported by 'the engine label' engine. Please remove the
groupby value on 'the report line display name'.**

*Enforced when:* writing the engine, writing the line's authored grouping key, and writing the
line's user grouping key.

*Recovery in the user interface:* when a user's chosen grouping would break this rule, the user
grouping silently falls back to the authored grouping instead of raising.

### E-04 — The formula must match its engine's grammar

*Enforced for three engines at write time:*

| Engine | Check |
|---|---|
| `domain` | The formula parses as a literal list structure **and** the resulting condition is accepted by the Journal Item search. |
| `account_codes` | After removing every space and splitting before each plus and minus, every non-empty token matches the term grammar of [`calculations.md`](calculations.md) §6.1 and yields a non-empty selector. |
| `aggregation` | The whole formula matches, from first character to last, either the keyword `sum_children` or the arithmetic grammar of [`calculations.md`](calculations.md) §5.2. |

*Message, for all three:* **Invalid formula for expression 'the label' of line 'the line name':
the formula**

*Not checked at write time:* the `tax_tags`, `external` and `custom` engines. Their formulas are
free text at write time and are validated on evaluation.

*Worked failure:* writing the formula `test(12)` on an expression whose engine is `account_codes`
raises the message, because `test(12)` is neither a code prefix (it contains parentheses) nor a
tag selector (it does not start with `tag(`).

### E-05 and E-06 — The carry-over target field

| Condition | Message |
|---|---|
| The carry-over target is set on an expression whose label does not start with `_carryover_`. | **You cannot use the field carryover_target in an expression that does not have the label starting with _carryover_** |
| The carry-over target is set and the part after the period does not start with `_applied_carryover_`. | **When targeting an expression for carryover, the label of that expression must start with _applied_carryover_** |

### E-07 — A carry-over target must be resolvable

*When:* closing a period, for every expression labelled `_carryover_`*x* with a non-zero amount.

*Resolution:* the explicit target when set; otherwise the expression labelled
`_applied_carryover_`*x* on the same line.

*Message when neither exists:* **Could not determine carryover target automatically for
expression the label.**

### E-08 — Cross-report clauses

| Condition | Message |
|---|---|
| The subformula starts with `cross_report` but does not match `cross_report(` reference `)`. | **In report 'the report name', on line 'the line name', with label 'the label', The format of the cross report expression is invalid. Expected: cross_report(<report_id>\|<xml_id>) Example: cross_report(my_module.my_report) or cross_report(123)** |
| The reference is neither a number nor a known external identifier. | **In report 'the report name', on line 'the line name', with label 'the label', Failed to parse the cross report id or xml_id.** |
| The reference names the expression's own report. | **You cannot use cross report on itself** |

### E-09 — Aggregation details

*Message when the parse is requested for a non-aggregation expression:* **Cannot get aggregation
details from a line not using 'aggregation' engine**

### E-10 — Whitespace normalization

*Invariant:* on every write, the formula and the subformula are trimmed and every run of
whitespace inside them is collapsed to a single space. A multi-line formula written in a data
package is therefore stored on one line.

*Consequence to test:* writing a formula that differs from the stored one only in whitespace is a
no-operation and, for a tax tag expression, does not create or rename any tag.

---

## 5. Report Column rules

1. The name and the expression label are required.
2. The display type is required and defaults to `monetary`.
3. Two columns of the same report may carry the same expression label; both then show the same
   figure. This is allowed and is used when a report shows the same figure with two display
   types.
4. A column whose expression label matches no expression of any line renders an entirely empty
   column. This is not an error, because a variant may legitimately omit a column's figures.
5. A custom audit action, when set, replaces the default drill-down for every cell of that
   column.

---

## 6. Tax closing preconditions

### C-01 — The tax return journal must exist

*Condition:* the company must have a tax return journal, of type `general`.

*Behavior when missing:* the validation is refused and the user is redirected to the accounting
periods configuration, where the journal is set together with the opening date, the fiscal year
end and the periodicity.

### C-02 — The tax payable and receivable accounts must exist

*Condition:* for every tax group whose taxes moved in the period, the tax payable account must be
set when the group's net position is in favour of the authorities, and the tax receivable account
must be set when it is in favour of the company.

*Behavior when missing:* the validation is refused and the user is redirected to the tax group
concerned, so that the accounts can be filled in. Because a group's direction can change from one
period to the next, the practical rule a rebuild should apply is to require **both** accounts on
every group that participates.

### C-03 — The closing entry balances by construction

*Invariant:* the counterpart is computed as the negated sum of the already-rounded account lines
and the advance line, so the entry balances to the smallest unit of the company currency without
any rounding correction line.

### C-04 — The closing entry carries no tax, no tag and no analytic distribution

*Invariant:* no line of the closing entry has a tax set, a tax line reference, a tax tag or an
analytic distribution. Violating this would make the closing entry appear in the next period's
tax report and in the analytic ledger.

*Mechanism:* repartition lines that participate in the closing do not propagate the document's
analytic distribution, and the closing entry is generated with those three fields explicitly
empty.

### C-05 — A closing entry in a restricted journal may only be reversed

*Condition:* a posted entry that carries an inalterability hash cannot be reset to draft.

*Behavior:* cancelling the return produces a reversal entry instead of resetting.

---

## 7. Locking rules

### K-01 — A write into a locked period is refused

Five lock dates exist on the company. Four are **soft** — they admit exceptions — and one is
**hard**.

| Lock date | Scope | Exception possible |
|---|---|---|
| Global lock date (`fiscalyear_lock_date`) | Every entry | yes |
| Tax return lock date (`tax_lock_date`) | Entries carrying taxes | yes |
| Sales lock date (`sale_lock_date`) | Entries in sales journals | yes |
| Purchase lock date (`purchase_lock_date`) | Entries in purchase journals | yes |
| Hard lock date (`hard_lock_date`) | Every entry | **no** |

A date is violated when it is **on or after** the entry's accounting date — the lock is
inclusive of the day named.

*Message when an entry is created or modified in a locked period:*

**You cannot add/modify entries prior to and inclusive of: the list of violated lock dates, each
written as its label followed by its date in parentheses.**

*Message shown as a warning when a document's date falls before the tax return lock date, before
the document is posted:*

**The date is being set prior to: the list of violated lock dates. The Journal Entry will be
accounted on the proposed date upon posting.**

*Behavior for a new entry:* the accounting date is moved forward to the first date after the
violated lock date that the journal's sequence allows, rather than the entry being refused.

*Behavior for a manual figure in this domain:* the write is refused; there is no automatic date
shift, because an External Value is a declaration figure, not a ledger entry.

### K-02 — Validating a return moves the tax return lock date forward only

```formula
new_tax_lock_date = max( current_tax_lock_date , last day of the closed period )
```

The lock date is never moved backwards by this domain. Moving it back is a separate, tracked act
performed on the company.

### K-03 — The hard lock date admits no exception

A lock date exception can relax any of the four soft lock dates for a named user, a named
company, an optional single lock date field and a bounded time. It can never relax the hard lock
date.

### K-04 — A hashed entry's hashed fields may not be edited

*Condition:* an entry carrying an inalterability hash may not have any of its hashed fields
written, nor its hash itself.

*Message:* **You cannot edit the following fields: the list of field labels. The following
entries are already hashed: the list of entry numbers.**

*Hashed fields:* on the entry, its number, its accounting date, its journal and its company; on
each item, its label, its debit, its credit, its account and its partner.

---

## 8. Access and permission rules

### A-01 and A-02 — The access rights matrix

| Entity | Billing group | Read-only accounting group | Accounting manager group |
|---|---|---|---|
| Report Definition | read | read | read, create, update, delete |
| Report Line | read | read | read, create, update, delete |
| Report Expression | read | read | read, create, update, delete |
| Report Column | read | read | read, create, update, delete |
| Report External Value | none | read | read, create, update, delete |

Every row grants read only; no group below the manager may create, update or delete a definition.
The external value table is not readable by the billing group at all, because manual declaration
figures are not invoicing data.

### A-03 — Availability

A report is offered to a user only when the availability algorithm of
[`entities.md`](entities.md) §1.5 succeeds for the active company **and** the user can read the
Report Definition. A user who can read every definition still sees only the reports whose
availability condition holds.

### I-01 — The integrity check requires the accounting user group

*Message when the group is missing:* **Please contact your accountant to print the Hash integrity
result.**

The check itself reads Journal Entries with full privileges, ignoring record-level restrictions,
because a partially visible entry would hash differently. Only the digest is used; no value is
returned to the user.

### X-01 and X-02 — Editing a manual figure

Two conditions, both required:

1. The expression's subformula contains the `editable` clause.
2. The user holds the accounting manager group.

A read-only accountant sees the cell but cannot change it.

---

## 9. Tag ownership rules

### T-01 — Uniqueness

*Condition:* the triple (name, applicability, country) is unique across account tags.

*Message:* **A tag with the same name and applicability already exists in this country.**

### T-02 — The three cash-flow tags may not be deleted

*Condition:* the tags named "Operating Activities", "Financing Activities" and "Investing &
Extraordinary Activities", shipped with the application, may not be deleted.

*Message:* **You cannot delete this account tag (the tag name), it is used on the chart of
account definition.**

### T-03 and T-04 — Deleting a tax tag expression

Algorithm, per tag matched by the deleted expressions:

1. Search for any other expression with the `tax_tags` engine, in a report of the same country,
   whose formula matches the tag's name. If one exists, **do nothing** (T-04).
2. Otherwise, search for any Journal Item carrying the tag.
   - If one exists, the tag is **archived** (T-03) — never deleted, because posted entries
     reference it.
   - If none exists, the tag is **deleted**.
3. In both cases, the tag is first removed from every tax repartition line that carried it.

*Consequence to test:* deleting a report line whose tag is used on a posted invoice leaves one
archived tag and removes it from the tax's repartition lines. Deleting a report line whose tag is
also used by another report changes nothing.

### T-05 — One tag per formula name

*Invariant:* a formula creates and matches **exactly one** tag, whose name is the formula with
any leading minus removed. A formula of `-X` and a formula of `X`, in the same country, address
the same single tag; the minus is a sign instruction on the expression, not part of the tag's
name.

*Consequence to test:* writing the formula `-Buny` onto an expression whose formula was `55`
renames the tag from `55` to `Buny` and creates no tag named `-Buny`.

*Renaming versus creating:* changing the formula renames the existing tag when every expression
related to it is part of the same write; otherwise a new tag is created and the old one is left
for the other expressions.

---

## 10. Evaluation-time rules and edge cases

### 10.1 Empty results

| Situation | Behavior |
|---|---|
| An engine matches no Journal Item. | The figure is zero, not empty. A zero is displayed unless the blank-if-zero flag is set. |
| An external expression finds no External Value. | The figure is zero for `sum` and zero for `most_recent`. |
| An aggregation reference resolves to an expression that produced no figure. | The reference contributes zero. |
| An aggregation reference resolves to no expression at all. | Authoring error: the cell renders empty and the condition is reported to the designer. |
| A column's expression label matches no expression on a line. | The cell is empty, not zero. |

The distinction between an **empty** cell and a **zero** cell is meaningful and must be preserved:
a zero says "we looked and found nothing"; an empty cell says "this line has no such figure".

### 10.2 Division

| Situation | Behavior |
|---|---|
| An aggregation divides by zero and carries `ignore_zero_division`. | The whole expression yields zero. |
| An aggregation divides by zero without that clause. | The cell renders empty. |
| A growth percentage has a zero comparison figure and a zero current figure. | The growth cell is empty. |
| A growth percentage has a zero comparison figure and a non-zero current figure. | The growth cell shows the infinity marker. |

### 10.3 Rounding

1. Intermediate sums are never rounded.
2. A total is computed from unrounded figures and rounded once; it may therefore differ from the
   sum of the displayed lines. This is correct.
3. Where a legislation requires the total to equal the sum of the declared boxes, the definition
   aggregates the **rounded** expressions, conventionally labelled `balance_rounded`, not the raw
   ones.
4. The report-level whole-unit rounding, when set, applies before the presentation rounding unit
   and after any expression-level rounding clause.

### 10.4 Dates

| Situation | Behavior |
|---|---|
| The period's from-date is after its to-date. | The period is empty; every ledger figure is zero. A rebuild should refuse the option rather than render zeros. |
| A single-date report with a date scope of `strict_range`. | The window is that one day. |
| A date scope of `to_beginning_of_period` on a single-date report. | The window ends the day before that date. |
| A fiscal year shorter or longer than twelve months. | The declared fiscal year records supply the boundaries; the default last-day-and-month rule applies outside them. |
| A leap-day date shifted back one year. | The twenty-ninth of February becomes the twenty-eighth. |

### 10.5 Accounts

| Situation | Behavior |
|---|---|
| Two companies use the same account code for different accounts. | A consolidated report merges them into one line, by code. Use the account code mapping to give one of them a distinct code if that is wrong. |
| An account's code is company-dependent. | The prefix comparison uses the code as seen by the selected company. |
| A term selects accounts already selected by another term. | Both contribute; the engine does not deduplicate. |
| An account has a zero balance and a balance character is applied. | It is excluded by both `D` and `C`. |
| An account is archived. | Its items are still read; archival hides the account from pickers, not from history. |

### 10.6 Tags

| Situation | Behavior |
|---|---|
| A tax tag expression's tag is archived. | The tag is still matched and its items still counted; archival only hides it from pickers. |
| One Journal Item carries several tags. | Its whole balance contributes to each tag; tags are not shares. |
| A tax tag formula matches no tag because the report has no country. | The tag is created with an empty country, and matches only items whose tag also has an empty country. A report using the tax tag engine should always declare a country. |

### 10.7 Companies and currencies

| Situation | Behavior |
|---|---|
| Several companies with different currencies are selected. | Every figure is translated into the currency of the first selected company. |
| The currency translation mode is `current`. | Foreign balances move with the rate; the whole report changes when the rate changes. |
| The currency translation mode is `cta`. | Stored company-currency amounts are used as they stand and the residual difference appears on a cumulative translation adjustment line. |
| A tax unit is selected but the report's multi-company switch is `selector`. | The unit is ignored and the companies are treated individually. |
| A company is selected for which the report is not available. | Its data is still included; availability governs which reports are **offered**, not which companies are read. |

### 10.8 Exigibility

| Situation | Behavior |
|---|---|
| A payment-basis tax on an unpaid invoice. | Excluded while the only-tax-exigible flag is on; the base and tax items are invisible to the tax report. |
| The same invoice after payment. | The cash-basis entry carries the tags and is always exigible, so the amounts appear in the period of the payment. |
| An entry with no receivable and no payable line and no cash-basis values. | Always exigible, whatever its taxes. |
| An item carrying only tags, with no tax and no tax line reference. | Always exigible. |
| An item mixing a payment-basis tax and an invoice-basis tax that share a tag. | Refused by the ledger: **Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag.** |

### 10.9 Unfolding

| Situation | Behavior |
|---|---|
| An expansion would exceed the load-more limit. | The first *limit* sub-lines are shown with a continuation marker. |
| An expansion would exceed the prefix-group threshold. | One level of prefix groups is inserted instead. |
| Both limits would apply. | The prefix-group rule wins: the candidates are grouped first, and the load-more limit then applies to the prefix-group rows. |
| A line is collapsed and re-expanded. | The same sub-lines appear in the same order. |
| The unfold-all option is turned on with a very large ledger. | The prefix-group rule protects the rendering; the load-more limit still applies within each group. |

---

## 11. Integrity rules

### I-02 — A corrupted prefix stops being scanned

Once an entry of a sequence prefix fails every hash version, that entry is recorded as the
prefix's corruption and every later entry of the prefix is skipped. The finding names the first
corruption only.

*Message:* **Corrupted data on journal entry with id the numeric identifier (the entry number).**

Other findings:

| Status | Message |
|---|---|
| Journal with no hashed entry | **There is no journal entry flagged for accounting data inalterability yet.** |
| Prefix fully verified | **Entries are correctly hashed** |

Each finding also carries the journal's name followed by the sequence prefix in parentheses, and
a flag reading `V` when the journal runs in restricted mode and `X` otherwise.

### I-03 — A protected audit trail message may not be deleted or altered

*Protected* means: the message is a notification, and the record it is attached to falls under the
restrictive audit trail of its company (the exact reach is given in
[`calculations.md`](calculations.md) §17.3).

*Refused operations:* deletion; changing the record reference, the record model, the message type
or the subtype; changing the subject other than by whitespace; changing a non-empty body.

*Message:* **You cannot remove parts of a restricted audit trail. Archive the record instead.**

*Exception:* a message attached to a Journal Entry that has never been posted may still be
removed.

### I-04 — A sequence gap breaks the hash chain

When the hashing pass finds that the numbering of the entries to hash skips a number inside the
range being chained, it stops:

*Message:* **An error occurred when computing the inalterability. A gap has been detected in the
sequence.**

The gap must be resolved — by resequencing or by explaining the missing number — before the chain
can continue.

---

## 12. Invariants a rebuild must be able to demonstrate

1. **The balance sheet balances.** For every date, for every company set, assets equal
   liabilities plus equity, provided every Journal Entry balances.
2. **The trial balance ties.** The sum of the period debit column equals the sum of the period
   credit column, for every period and every filter combination.
3. **The profit and loss ties to the balance sheet.** The net result from the first day of the
   fiscal year to a date equals the current-year earnings line of the balance sheet at that date.
4. **The cash flow statement ties.** Opening cash plus the net increase equals closing cash.
5. **A drill-down sums to its figure.** For every auditable expression except a multiplying or
   dividing aggregation, the sum of the balances of the items listed by the audit action equals
   the displayed figure, up to the report's rounding.
6. **Expansion preserves totals.** The sum of a line's sub-lines equals the line, up to rounding,
   for every grouping and account expansion.
7. **Closing is idempotent.** Validating, cancelling and re-validating the same period produces
   the same carry-over values and an equivalent closing entry.
8. **Carry-over conserves value.** Over any run of consecutive periods, the sum of the declared
   figures plus the amount still carried at the end equals the sum of the periods' own movements.
9. **Comparison is symmetric.** The figures of a comparison column equal the figures the report
   shows when that comparison period is made the current period, with the same filters.
10. **Definitions are inert.** Changing a report definition changes no ledger balance and
    produces no Journal Entry.
