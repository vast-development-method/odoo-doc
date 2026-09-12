# Analytic Accounting — Business Rules

The complete, numbered catalogue of validations, guards, constraints, invariants, permission
checks and locking rules of the domain. Every rule carries a stable identifier of the form
`AA-nnn` that the other files of this folder cite. Messages are reproduced exactly as the user
sees them; a placeholder inside a message is described in words.

Contents:

1. [Plans, hierarchy and dynamic columns](#1-plans-hierarchy-and-dynamic-columns) — `AA-001` to `AA-021`
2. [Applicability](#2-applicability) — `AA-022` to `AA-030`
3. [Analytic accounts](#3-analytic-accounts) — `AA-031` to `AA-040`
4. [Distributions](#4-distributions) — `AA-041` to `AA-058`
5. [Distribution models](#5-distribution-models) — `AA-059` to `AA-065`
6. [Analytic lines](#6-analytic-lines) — `AA-066` to `AA-077`
7. [Generation, validation and posting](#7-generation-validation-and-posting) — `AA-078` to `AA-095`
8. [Company and currency consistency](#8-company-and-currency-consistency) — `AA-096` to `AA-101`
9. [Rounding, ordering and dates](#9-rounding-ordering-and-dates) — `AA-102` to `AA-108`
10. [Permissions](#10-permissions) — `AA-109` to `AA-115`
11. [Rule index](#11-rule-index)
12. [Mapping of the former rule identifiers](#12-mapping-of-the-former-rule-identifiers)
13. [Reconciliation notes](#13-reconciliation-notes)

---

## 1. Plans, hierarchy and dynamic columns

**AA-001.** An Analytic Plan must have a name. The name is translatable: each installed language
may carry its own value, and the untranslated value is used when no translation exists.

**AA-002.** A plan may have at most one parent, and the parent may be neither the plan itself nor
any of its descendants. The selectable values for the parent exclude the plan and its whole
sub-tree, which makes a cycle impossible; the materialised path maintained by the tree machinery
enforces the same invariant in storage.

**AA-003.** The base plan — the plan whose identifier is stored in the system parameter
`analytic.project_plan` — may never be given a parent. Attempting it raises, while the form is
still being edited and before anything is written: "You cannot add a parent to the base plan 'the
plan name'". The placeholder is the base plan's name.

**AA-004.** A root plan owns exactly one stored analytic account column on every model that
implements the Analytic Plan Fields Mixin. The base plan's column is named `account_id`; every
other root plan's column is named `x_plan`, the plan's internal identifier and `_id`.

**AA-005.** A sub-plan owns no stored column. Its accounts are stored in the column of its root
plan. As a direct consequence, a record that carries plan columns holds at most one analytic
account per root plan, whatever the depth of the sub-plan that account belongs to.

**AA-006.** For every hierarchy depth that exists under a root plan, one derived read-only
grouping column exists on every model that carries plan columns, named after the root plan's
column name, an underscore and the depth, prefixed with `x_` when the result would otherwise begin
with `account_id`. Its label is the root plan's name, a space, and the depth in parentheses. It is
created when the first plan of that depth appears and deleted when the last plan of that depth
disappears.

**AA-007.** The stored column a root plan owns is a link to Analytic Account, marked manual,
copied when a record is duplicated, with a deletion rule of **restrict**; on a model with a real
table it additionally carries a balanced-tree index restricted to the rows where the column is not
empty, and the field is marked as indexed.

**AA-008.** The grouping column a sub-plan owns is a link to Analytic Plan, marked manual, not
stored, read-only, and reads its value through the chain "the root plan's stored column, then that
account's plan, then that plan's parent repeated *depth minus one* times".

**AA-009.** A plan cannot be deleted while a stored view definition still names the column it
owns. The deletion is refused with "Cannot rename/delete fields that are still present in views:"
followed by the list of field names and the name of the offending view, and nothing at all is
deleted.

**AA-010.** Deleting a plan deletes every descendant of it, because the parent link cascades;
deletes the stored column it owned together with the values that column held; and deletes the
grouping columns that no longer correspond to any surviving plan level.

**AA-011.** Changing the parent of a plan, or the plan of an analytic account, is refused when the
change would make an analytic line hold two different accounts in what would become a single
column. The message is "Whoa there! Making this change would wipe out your current data. Let's
avoid that, shall we?", accompanied by a button labelled "See them" that opens the offending
analytic lines in a list. The check tolerates the case where the target column already holds one
of the accounts being moved, because no value is then lost.

**AA-012.** When a plan is demoted — a parent is set — the values of the analytic lines are
migrated **before** the parent is written, because the plan's own column ceases to exist as part
of that write. When a plan is promoted — the parent is cleared — the values are migrated **after**
the parent is written, because the plan's own column is created as part of that write.

**AA-013.** The migration of analytic line values selects every analytic account whose plan is the
plan being moved **or any descendant of it**, not only the accounts attached directly to that
plan.

**AA-014.** A grouping column is deleted only when no plan is left at its level: the test reads
the root plan's identifier out of the column name, falling back to the base plan when the name
carries no digits, reads the depth from the trailing number, and searches for a plan with that
root whose materialised path has at least that many path separators. While such a plan exists the
column is kept.

**AA-015.** Changing a plan's name changes only the label of the column or grouping column it
owns, never the column name. Renaming a plan is therefore always safe with respect to stored data
and stored view definitions.

**AA-016.** When the system parameter `analytic.project_plan` designates no plan, or designates a
plan that does not exist, every operation that needs the list of root plans fails with "A
'Project' plan needs to exist and its id needs to be set as `analytic.project_plan` in the system
variables".

**AA-017.** The system parameter `analytic.project_plan` may only be written with a string of
digits naming an existing plan that currently owns a stored column, which means an existing root
plan. Any other value is refused with "The value for the key must be the ID to a valid analytic
plan that is not a subplan"; the placeholder is the parameter key.

**AA-018.** Changing `analytic.project_plan` migrates no data: the former base plan receives a
new, empty generated column, and the generated column of the new base plan is deleted together
with its contents. The fixed column keeps the values it held, which are from then on read as
accounts of the new base plan. The operation is safe only before any analytic line exists, and a
rebuild must say so.

**AA-019.** The default applicability of a plan is stored per company: each company holds its own
value for the same plan, and reading the field returns the value of the company the reader is
acting for. It is required on the form of a root plan and hidden on the form of a sub-plan.

**AA-020.** The following operations, and only these, invalidate the cached list of relevant
plans: creating a plan, deleting a plan, writing a plan's default applicability, and creating,
writing or deleting an applicability rule. Writing any other field of a plan leaves the cache in
place. The cache lives for the duration of one database transaction and is keyed by the exact set
of situation arguments.

**AA-021.** A plan itself has no company and no record rule restricts it; plans are not
archivable, so an unused plan is deleted or simply left with no account; and every successful
create, write or delete of a plan makes every open client reload its definition of the models that
carry analysis, because their set of fields has changed.

---

## 2. Applicability

**AA-022.** An applicability rule must carry a business domain and an applicability. The closed
list of business domains depends on the installed capabilities and is given in
[entities.md](entities.md) section 4.2.

**AA-023.** An applicability rule is evaluated only for the plan it is attached to, and only for a
root plan: sub-plans have no applicability of their own.

**AA-024.** A rule whose company is set and differs from the company supplied in the situation is
excluded from the evaluation entirely, and therefore never prevents a company-independent rule
from winning. A rule is kept when it has no company, when the situation supplies no company, or
when the two companies are equal.

**AA-025.** A rule must score **strictly more** than the baseline of zero point five to override
the plan's default applicability. A rule whose only distinguishing criterion is the company scores
exactly zero point five and can therefore never override the default on its own.

**AA-026.** Within one rule, the financial account prefix and the product category are combined
with a logical "and": a mismatch on either one eliminates the rule even when the other matches,
and an eliminated rule scores minus one. A mismatch on the business domain eliminates the rule the
same way.

**AA-027.** When the situation supplies **no** business domain, the base part of the score returns
immediately with zero, or zero point five when both the rule and the situation name a company; the
prefix and category criteria of the general ledger capability are nevertheless still evaluated on
top of that base score, and each matching one adds one point. A rule that names a matching product
category therefore overrides the default even with no business domain supplied, while a rule that
names only a company never does.

**AA-028.** A plan whose whole sub-tree contains no analytic account is never shown in a
distribution editor, whatever its applicability.

**AA-029.** A plan whose computed applicability is `unavailable` is not shown in a distribution
editor, **unless** the distribution already names one of its accounts, in which case it is shown
with the applicability `optional`, which keeps the existing percentage visible and editable.

**AA-030.** When the caller forces an applicability — as the distribution editor of a distribution
model does — every root plan with at least one account in its sub-tree is shown with the forced
applicability, and no rule is evaluated at all.

---

## 3. Analytic accounts

**AA-031.** The company of an analytic account cannot be changed while at least one analytic line
referring to that account belongs to a company that is neither the new company nor one of its
descendants. The message is "You can't change the company of an analytic account that already has
analytic items! It's a recipe for an analytical disaster!". Clearing the company, which makes the
account shared by every company, is always allowed, because every existing line then satisfies the
condition.

**AA-032.** Changing the plan of an analytic account migrates that account's value in the analytic
lines from the column of its current root plan to the column of its new root plan, under the
conflict rule and the message of `AA-011`. When the two root plans coincide, the column names are
identical and nothing is migrated.

**AA-033.** An analytic account may not be deleted while an expense refers to it in its
distribution. The message is "You cannot delete an analytic account that is used in an expense."

**AA-034.** An analytic account may not be deleted while a project bound to it still has tasks.
The message is "Before we can bid farewell to these accounts, you need to tidy up the projects
linked to them by removing their existing tasks!"

**AA-035.** An analytic account may not be deleted while an analytic line refers to it in a plan
column, because that reference carries the deletion rule restrict. A distribution that merely
names the account is not a reference and does not block the deletion; identifiers of deleted
accounts are ignored wherever a distribution is read.

**AA-036.** The customer of an analytic account must belong to the account's company or be shared.
A violation raises the platform's company-consistency message: "Uh-oh! You've got some company
inconsistencies here:" followed by one line per violation of the form "- 'the record' belongs to
company 'the company' while 'the field label' (the field identifier: the values) belongs to another
company." and the closing line "To avoid a mess, no company crossover is allowed!"

**AA-037.** An analytic account must have a name and a plan. Two accounts of the same plan may
share a name and a reference; there is no uniqueness constraint beyond the internal identifier.

**AA-038.** An archived analytic account is hidden from the default lists and from the value lists
of the distribution editor, but existing distributions and existing analytic lines keep referring
to it, and its history stays intact. Reactivating it removes the posting block of `AA-078`.

**AA-039.** Duplicating an analytic account sets the copy's name to the original's name followed
by a space and the word *(copy)* in parentheses, unless a name is supplied explicitly, and copies
every other stored field.

**AA-040.** The display name of an analytic account is built as: the name; prefixed with the
reference in square brackets and a space when a reference exists; suffixed with a space, a hyphen,
a space and the commercial partner's name when the account's customer has a commercial partner
with a name. A search by name matches either the name or the reference.

---

## 4. Distributions

**AA-041.** Every percentage written into a distribution is rounded to the number of decimal
places of the decimal precision record named `Percentage Analytic`, which is two, before it is
stored. The reserved key `__update__` is exempt from that rounding. A distribution that is empty
or absent is stored as nothing rather than as an empty document.

**AA-042.** A distribution key is a comma-separated list of analytic account identifiers naming at
most one account per root plan. A key may name accounts of several different root plans, which
attributes the same percentage to the combination of them and produces a single analytic line
carrying all of them.

**AA-043.** A distribution written without the reserved key `__update__` replaces the stored
distribution entirely. A distribution written with that key replaces only the part of the stored
distribution that concerns the plans whose column names are listed in it and keeps the rest,
multiplying the two sides as described in [calculations.md](calculations.md) section 7.

**AA-044.** The reserved key `__update__` never survives in a stored distribution: it is consumed
by the merge. A distribution read back from storage never contains it.

**AA-045.** A merge that produces an empty distribution is treated by the callers as "no change
requested": the record is left exactly as it was, and no analytic line is created or deleted.

**AA-046.** A distribution is not required to total one hundred percent. A total below one hundred
attributes only part of the amount; a total above one hundred attributes more than the amount,
which is a legitimate way of modelling a cost analysed twice on the same axis. Only a mandatory
plan forces the total to be exactly one hundred.

**AA-047.** The total of one plan is the sum of the percentages of every distribution entry naming
an account whose **root** plan is that plan. Accounts of two different sub-plans of the same root
plan therefore add up together, and a compound key contributes its **whole** percentage to **each**
root plan it touches, not a share of it.

**AA-048.** A distribution that names an analytic account that no longer exists remains valid. The
missing identifiers are ignored when the distribution is validated, when its account list is
derived, and when analytic lines are generated from it.

**AA-049.** The partial-update merge multiplies the non-changing side by the changing side, entry
by entry, and balances the two totals: the side with the larger total keeps its surplus as entries
naming only its own accounts. The full algorithm, with its ratio and its leftover, is in
[calculations.md](calculations.md) section 7.

**AA-050.** The merge never divides by zero: when the non-changing side is empty the cross product
never runs, and the result is the changing side unchanged. When the changing side is empty the
mirror case gives the non-changing side unchanged. When both are empty the result is the empty
document, which `AA-045` then treats as "no change".

**AA-051.** The order of the identifiers produced by a merge is: the identifiers of the
non-changing side in ascending order, followed by the identifiers of the changing side in
ascending order, joined by commas.

**AA-052.** On every model that stores a distribution and has a real table, a generalised inverted
index is created over the array of account identifiers extracted from the keys of the stored
document; extraction splits the rendered keys on every run of non-digit characters.

**AA-053.** A condition written on a distribution field **in the database** supports only the
operators "in", "not in", "contains" and "does not contain". Any other operator raises "Operation
not supported". A list containing the empty value is passed through unchanged so that "has no
distribution" works.

**AA-054.** The negated forms of those operators additionally match records whose distribution is
empty. When the resolution of a text value finds no account at all, the condition collapses to a
constant: false for the positive form, true for the negative one.

**AA-055.** When an already-loaded set of records is filtered instead, a condition on the
distribution field is first rewritten as the same condition on the derived list of analytic
accounts; that list is a collection of records, so the equality and inequality operators are
accepted there as well, matched against an account display name or an account identifier.

**AA-056.** Grouping a report by a distribution field expands each record into one row per
analytic account named in its distribution and counts a per-model owner: the journal entry for a
journal item, the purchase order for a purchase order line, and the record itself for an asset and
for an expense.

**AA-057. Compatibility finding.** When a distribution field is searched in the database with the
inclusion operator and a **text** value, the resolution of names to accounts tests whether the
whole value is text rather than each element of it. The observed consequences are: a list of
account names is passed through unresolved and matches nothing; a single text value is iterated
character by character and each character is resolved as an exact display name, which normally
finds nothing and collapses the condition to a constant. The partial-match operator resolves the
value correctly, and filtering an already-loaded set through `AA-055` resolves names correctly. A
corrected behaviour would test each element of the value and resolve every text element by exact
display name; a rebuild that does so is more useful but is no longer bug-compatible on that path.

**AA-058.** Grouping by a distribution field supports only the count aggregate. Any other
aggregate raises "analytic_distribution grouping does not accept the aggregate specification as
aggregate." and grouping on a model that declares no owner raises "the table name does not support
analytic_distribution grouping." In both messages the placeholder is the offending aggregate
specification and the offending table name respectively; both texts are reproduced.

---

## 5. Distribution models

**AA-059.** A distribution model whose distribution names at least one analytic account that
belongs to one specific company must itself belong to that same company. The message is "You
defined a distribution with analytic account(s) belonging to a specific company but a model shared
between companies or with a different company".

**AA-060.** Every condition set on a distribution model must hold for the model to apply. A
condition left empty means "any value" and never disqualifies the model. A criterion the caller
does not supply falls back to the empty value and therefore matches only the models that leave
that criterion empty.

**AA-061.** The account prefix condition is evaluated as a prefix match of the document line's
financial account code against any one of the pieces obtained by splitting the model's prefix on a
comma or a semicolon, each optionally followed by spaces. It is the only condition not evaluated
by the stored query; the query contributes nothing for it and the selected set is filtered in
memory afterwards.

**AA-062.** Distribution models are evaluated in ascending sequence, then **descending** internal
identifier. Among models with the same sequence, the most recently created one is evaluated first.
There is no score: a model naming five conditions does not beat a model naming one unless its
sequence says so.

**AA-063.** A distribution model is skipped entirely when at least one of the root plans it would
fill has already been filled by a higher-priority model or by the related source document, even
when the model would also fill a plan that is still empty. A model whose distribution names no
existing account fills nothing and is skipped as well.

**AA-064.** A model proposal never overwrites a non-empty distribution with nothing: when the
proposal computed for a journal item is empty, the value already stored on the journal item is
kept.

**AA-065.** Deleting the partner, the partner category, the product, the product category or the
company referenced by a distribution model deletes the model.

---

## 6. Analytic lines

**AA-066.** A record carrying plan columns must have at least one of them set. A line with no
account at all is refused with "At least one analytic account must be set". The check runs
whenever any plan column is written.

**AA-067.** The financial account of an analytic line must equal the account of the journal item
the line points at, when it points at one. A violation raises "The journal item is not linked to
the correct financial account". The check runs whenever either field is written.

**AA-068.** An analytic line must carry a description, a date, an amount and a company. The
description and the date have no default beyond today's date in the reader's time zone for the
date; the amount defaults to zero; the company defaults to the company the reader is acting for.

**AA-069.** The amount of an analytic line is expressed in the company currency and never in a
document currency. When a journal item is expressed in a foreign currency, the analytic amount
derives from that journal item's balance, which is already converted, and is rounded with the
**company** currency's rounding step.

**AA-070.** The sign convention is: a positive amount is revenue, a negative amount is cost. An
analytic line generated from a journal item takes the negated balance of that journal item times
the percentage, therefore a debit becomes a cost and a credit becomes revenue.

**AA-071.** The company of an analytic line is set at creation and never changed afterwards; the
field is read-only on the form.

**AA-072.** An analytic line is visible only to a reader for whom the line's company is one of the
companies currently active. Unlike analytic accounts, applicability rules and distribution models,
analytic lines are never shared and are never visible from a parent company that is not active.

**AA-073.** A draft journal entry never owns analytic lines. Analytic lines created or written on
a journal item of a draft entry are consumed to rebuild that journal item's distribution and are
then deleted immediately, with the synchronisation guard raised.

**AA-074.** Deleting a journal item deletes its analytic lines by cascade. Resetting a journal
entry to draft deletes the analytic lines of all its journal items without touching their
distributions.

**AA-075.** Any condition written on the fiscal year search helper of an analytic line is
replaced, whatever the operator and the value, by "the date is on or after the first day of the
fiscal year containing today, minus one year".

**AA-076.** When a journal item's distribution is rebuilt from its analytic lines, two analytic
lines carrying exactly the same combination of accounts collapse into one entry of the
distribution, and that entry keeps the percentage of the **last** line processed instead of their
sum. Splitting one analytic line into two lines with the same accounts therefore loses part of the
distribution.

**AA-077.** When the journal item's balance is zero, the percentage derived for each of its
analytic lines is one hundred, because no proportion can be computed.

---

## 7. Generation, validation and posting

**AA-078.** A journal entry cannot be posted while any analytic account named in the distribution
of any of its journal items is archived. The message is "You cannot post an entry with an archived
analytic account: the account names", the placeholder listing the names of all the archived
accounts found, separated by a comma and a space. The check reads the accounts with elevated
rights and with archived records included.

**AA-079.** The mandatory plan validation runs only when the caller has switched the validation
flag on. The posting button of the journal entry form, the mass validation assistant, the
confirmation buttons of sales orders and purchase orders, the approval action of expenses, the
manufacturing order confirmation and the transfer validation all switch it on. Programmatic
posting and the automatic posting job do not, and therefore never block on a mandatory plan.

**AA-080.** The mandatory plan validation of a journal item applies only to journal items whose
display type is `product`. Tax lines, payment term lines, early payment discount lines, discount
allocation lines, sections and notes are never validated.

**AA-081.** A journal item fails the mandatory plan validation when, for at least one plan whose
computed applicability is `mandatory` in the item's situation, the sum of the percentages
attributed to accounts of that root plan differs from one hundred after rounding both operands to
two decimal places. The message is "One or more lines require a 100% analytic distribution."

**AA-082.** When several journal entries are posted at once and at least one journal item fails,
the same message is raised together with a redirect to the list of the offending journal items,
titled "Items With Missing Analytic Distribution", opened through a button labelled "See items".
When exactly one entry is being posted, the plain message is raised without a redirect.

**AA-083.** The situation passed to the mandatory plan validation of a journal item is: the item's
company, the item's product, the item's financial account, and the business domain `invoice` when
the entry is a sale document or a sale receipt, `bill` when it is a purchase document or a
purchase receipt, and `general` otherwise.

**AA-084.** The invalid analytics flag marks a product journal item whose distribution fails the
mandatory plan validation, excluding items booked on an account whose type is receivable, payable,
cash or credit card. It is a presentation aid computed inside an exception guard and does not by
itself prevent posting.

**AA-085.** An exchange difference journal entry created automatically while reconciling is posted
with the validation flag explicitly switched off, therefore a mandatory plan never blocks a
reconciliation. Its journal items carry a distribution only when the caller supplies one.

**AA-086.** Analytic lines are generated for every journal item of a posted entry that carries a
non-empty distribution, including tax lines, early payment discount lines and discount allocation
lines, which inherit their distribution from the base lines they derive from.

**AA-087.** A candidate analytic line whose amount passes the zero test at the company currency's
rounding step is not created at all, and therefore contributes nothing to the rounding pass.

**AA-088.** Within one journal item, the distribution entry that brings a plan's cumulative
percentage to exactly one hundred — compared at the percentage precision — receives the exact
outstanding amount for that plan, the negated balance times one hundred minus the percentage
already distributed, divided by one hundred, instead of its own proportional share. This is what
makes the analytic lines of a fully distributed plan add up exactly to the negated balance.

**AA-089. Compatibility finding.** When one distribution entry closes two root plans at the same
time with different accumulated drifts, the loop over the accounts of the key computes an amount
for each of them and keeps the amount computed for the **last** account of the key, so only that
plan is closed exactly. In a well-formed distribution every root plan reaches the same accumulated
total at the same entry, so every pass computes the same number and the order is immaterial. A
corrected behaviour would compute one amount per root plan and create one analytic line per plan,
which would change the number of records produced; the observed behaviour is the one to reproduce.

**AA-090.** After all the candidate lines of one journal item have been rounded, the accumulated
rounding error is removed by adjusting the candidates one by one, starting with the first, each by
a step that is at least one rounding unit, until the remaining error passes the zero test. The
remainder therefore lands on the **first** slices, not on the last.

**AA-091.** Regenerating the analytic lines of a posted journal item deletes the existing lines
and creates new ones; their identifiers are not preserved. The redistribution operation used by
valuation documents, by contrast, updates the existing lines in place, deletes only the ones whose
new amount rounds to zero or whose combination has disappeared, and therefore preserves the
identifiers of the surviving lines.

**AA-092.** Both the generation of analytic lines from a posted journal item and their deletion on
a reset to draft run with the synchronisation guard raised, which prevents writing analytic lines
from immediately rewriting the distribution they were built from.

**AA-093.** Creating, writing — on the amount, on the journal item link or on any plan column — or
deleting an analytic line by hand rebuilds the distribution of the journal item pointed at before
the operation and of the journal item pointed at after it. The rebuild itself writes the
distribution with the synchronisation guard raised, therefore the manually edited analytic lines
are not deleted and regenerated.

**AA-094.** Posting adjusts the accounting date of each entry for the violated lock dates
**before** the analytic lines are created, so a generated analytic line always carries the final
accounting date of its journal item.

**AA-095.** Writing a distribution onto an analytic line splits it: the first set of values
produced by the split is written onto the line itself and each remaining set creates a copy of the
line. When at least one copy was made, the acting user receives the notification "the number of
created lines analytic lines created". When the merged distribution is empty, the line is left
exactly as it was.

---

## 8. Company and currency consistency

**AA-096.** An analytic account is visible when it has no company, or when its company is one of
the active companies or an ancestor of one of them. A branch therefore uses the analytic accounts
of its parent company, and an account created for a parent company may be used on the documents of
its branches.

**AA-097.** An applicability rule and a distribution model are visible under the same condition as
an analytic account: no company, or a company that is an ancestor of one of the active companies.

**AA-098.** The analytic account chosen in a plan column of an analytic line must be compatible
with the line's company: it must have no company, or a company that is the line's company or an
ancestor of it. The same holds for the line's partner, product, financial account and journal, and
for the partner and the product of a distribution model.

**AA-099.** The three derived totals of an analytic account are expressed in the account's company
currency, or in the currency of the company the reader is acting for when the account has no
company. Amounts of analytic lines expressed in another currency are converted before being added.

**AA-100.** The derived totals of an analytic account consider only analytic lines whose company
is empty or one of the active companies, and honour the optional date range given by the reporting
context: a from-date restricts the lines to those dated on or after it, a to-date to those dated on
or before it.

**AA-101.** The conversion used by those totals is made at **today's** rate, for the company the
reader is acting for, not at the rate of the line's own date. A balance whose lines span currencies
is therefore a today's-rate figure and changes from one day to the next. This is current behaviour
and must be reproduced.

---

## 9. Rounding, ordering and dates

**AA-102.** Percentage rounding uses the percentage precision, which is two decimal places.
Monetary rounding uses the rounding step of the company currency. Both round halves away from
zero.

**AA-103.** Comparisons of percentages against one hundred, and comparisons of a plan's cumulative
percentage against its closing value, are performed after rounding both operands to the percentage
precision.

**AA-104.** Default ordering: plans by sequence ascending then internal identifier ascending;
analytic accounts by plan then name ascending; analytic lines by date descending then internal
identifier descending; distribution models by sequence ascending then internal identifier
descending.

**AA-105.** Applicability rules are evaluated in internal identifier order and a tie on the score
is kept by the rule evaluated first, which is the rule with the lower internal identifier.

**AA-106.** The date of an analytic line generated from a journal item is that journal item's
accounting date, which may have been shifted by the lock dates during posting. The date of a
manual analytic line defaults to today in the reader's time zone.

**AA-107.** The distribution editor holds and compares percentages with two decimal places more
than the percentage precision; it treats a plan as complete when its running total rounds to
exactly one hundred percent at that working precision, and it recomputes a row's percentage from a
typed monetary value at the same working precision.

**AA-108.** The quantity of a journal item is copied whole onto every analytic line generated from
it and is never divided by the percentage. The quantity describes the underlying transaction, not
the slice.

---

## 10. Permissions

**AA-109.** Reading, creating, modifying and deleting an Analytic Plan, an Analytic Plan
Applicability, an Analytic Account, an Analytic Line and an Analytic Distribution Model all require
membership of the analytic accounting permission group. No other group of this domain exists and
no other group grants any right on these entities.

**AA-110.** The analytic accounting permission group is granted and revoked through the accounting
setting whose text is "Track costs & revenues by project, department, etc", reproduced as the
system displays it, which grants it to every user of the internal user group. Switching that
setting on also switches the full accounting capability setting on.

**AA-111.** In the analytic account list, the derived totals debit and credit are hidden columns
when the accounting capability is not installed, and, when it is, all three totals — debit, credit
and balance — are columns restricted to a reader who additionally holds the read-only accounting
group or the invoicing group. The balance is shown without any such restriction on the *Gross
Margin* button of the account's form, therefore to every member of the analytic accounting
permission group.

**AA-112.** The distribution editor and the analytic columns of a document are shown only to
members of the analytic accounting permission group; readers outside it see the document without
any analytic column and reach no analytic screen.

**AA-113.** A reader who holds only the analytic accounting permission group may create plans,
accounts, applicability rules, distribution models and analytic lines; the creating user is
recorded as the author of the record.

**AA-114.** The field descriptions and the view patching that insert one column per root plan are
applied only when the reader may read Analytic Plan, and are skipped entirely in the
view-customisation mode, which lets a customisation tool see the raw stored definition.

**AA-115.** Four reads are performed with elevated rights, and a rebuild must reproduce the
elevation or the behaviour changes: the resolution of the list of root plans; the company
consistency check of `AA-031`, which counts analytic lines the reader may not see; the name search
on the customer of an analytic account; and the archived-account check of `AA-078`, which must see
archived accounts.

---

## 11. Rule index

| Identifier | Subject | Carries a message |
|---|---|---|
| `AA-001` | Plan name required | no |
| `AA-002` | One parent, no cycle | no |
| `AA-003` | Base plan may not have a parent | yes |
| `AA-004` | One stored column per root plan | no |
| `AA-005` | A sub-plan owns no column | no |
| `AA-006` | One grouping column per depth | no |
| `AA-007` | Properties of the stored column | no |
| `AA-008` | Properties of the grouping column | no |
| `AA-009` | Column still used by a stored view | yes |
| `AA-010` | Deleting a plan cascades | no |
| `AA-011` | Column rewrite conflict | yes |
| `AA-012` | Migration before demotion, after promotion | no |
| `AA-013` | Migration covers the whole sub-tree | no |
| `AA-014` | A grouping column is kept while its depth is occupied | no |
| `AA-015` | Renaming changes labels only | no |
| `AA-016` | The base plan must exist | yes |
| `AA-017` | Base-plan parameter value | yes |
| `AA-018` | Base-plan parameter change migrates nothing | no |
| `AA-019` | Default applicability is per company | no |
| `AA-020` | Relevant-plan cache invalidation | no |
| `AA-021` | Plans are global, not archivable, reload clients | no |
| `AA-022` | Rule requires domain and applicability | no |
| `AA-023` | Rules belong to root plans | no |
| `AA-024` | Rules of another company are excluded | no |
| `AA-025` | The baseline of zero point five | no |
| `AA-026` | Prefix and category combined with "and" | no |
| `AA-027` | Scoring without a business domain | no |
| `AA-028` | A plan with no account is never offered | no |
| `AA-029` | An unavailable plan already used is forced back | no |
| `AA-030` | Forced applicability | no |
| `AA-031` | Company of an account with lines | yes |
| `AA-032` | Plan of an account | yes (through `AA-011`) |
| `AA-033` | Deletion while used by an expense | yes |
| `AA-034` | Deletion while a project has tasks | yes |
| `AA-035` | Deletion while referenced by a line | yes (platform message) |
| `AA-036` | Customer company consistency | yes |
| `AA-037` | Name and plan required | no |
| `AA-038` | Archiving hides but keeps history | no |
| `AA-039` | Duplication renames the copy | no |
| `AA-040` | Display name and name search | no |
| `AA-041` | Percentage normalisation on write | no |
| `AA-042` | Shape of a key | no |
| `AA-043` | Write with and without the marker | no |
| `AA-044` | The marker never survives | no |
| `AA-045` | An empty merge changes nothing | no |
| `AA-046` | Totals need not be one hundred | no |
| `AA-047` | Totals are per root plan | no |
| `AA-048` | Missing accounts are ignored | no |
| `AA-049` | The merge multiplies and balances | no |
| `AA-050` | The merge never divides by zero | no |
| `AA-051` | Order of the merged identifiers | no |
| `AA-052` | The inverted index | no |
| `AA-053` | Supported search operators | yes |
| `AA-054` | Negated forms and empty distributions | no |
| `AA-055` | Filtering an already-loaded set | no |
| `AA-056` | Grouping counts owners | no |
| `AA-057` | Name resolution in a database search (compatibility finding) | no |
| `AA-058` | Grouping aggregates | yes |
| `AA-059` | Model company consistency | yes |
| `AA-060` | Every condition must hold | no |
| `AA-061` | The prefix condition is evaluated in memory | no |
| `AA-062` | Model ordering | no |
| `AA-063` | A model touching a taken plan is skipped | no |
| `AA-064` | An empty proposal erases nothing | no |
| `AA-065` | Cascade deletion of models | no |
| `AA-066` | At least one analytic account | yes |
| `AA-067` | Financial account consistency | yes |
| `AA-068` | Required fields of a line | no |
| `AA-069` | Company currency only | no |
| `AA-070` | Sign convention | no |
| `AA-071` | Company set at creation | no |
| `AA-072` | Strict company visibility | no |
| `AA-073` | A draft entry owns no lines | no |
| `AA-074` | Cascade and reset deletions | no |
| `AA-075` | Fiscal year search helper | no |
| `AA-076` | Duplicate combinations collapse | no |
| `AA-077` | Zero balance gives one hundred percent | no |
| `AA-078` | Archived account blocks posting | yes |
| `AA-079` | The validation flag | no |
| `AA-080` | Only product lines are validated | no |
| `AA-081` | The mandatory plan failure | yes |
| `AA-082` | Mass posting redirect | yes |
| `AA-083` | The situation of a journal item | no |
| `AA-084` | The invalid analytics flag | no |
| `AA-085` | Exchange difference entries | no |
| `AA-086` | Which journal items generate lines | no |
| `AA-087` | Candidates rounding to zero | no |
| `AA-088` | The closing-line rule | no |
| `AA-089` | Concurrent closing (compatibility finding) | no |
| `AA-090` | Rounding-error cancellation | no |
| `AA-091` | Regeneration versus redistribution | no |
| `AA-092` | The synchronisation guard | no |
| `AA-093` | Manual edits rebuild the distribution | no |
| `AA-094` | Lock dates are applied first | no |
| `AA-095` | Splitting a line | yes (notification) |
| `AA-096` | Account visibility | no |
| `AA-097` | Rule and model visibility | no |
| `AA-098` | Company consistency of relations | no |
| `AA-099` | Currency of the totals | no |
| `AA-100` | Companies and dates of the totals | no |
| `AA-101` | Conversion at today's rate | no |
| `AA-102` | Rounding rules | no |
| `AA-103` | Comparisons after rounding | no |
| `AA-104` | Default ordering | no |
| `AA-105` | Rule evaluation order and ties | no |
| `AA-106` | Dates of analytic lines | no |
| `AA-107` | Editor working precision | no |
| `AA-108` | The quantity is copied, not divided | no |
| `AA-109` | Access matrix | no |
| `AA-110` | Granting the group | no |
| `AA-111` | Debit and credit visibility | no |
| `AA-112` | Editor visibility | no |
| `AA-113` | Creation rights | no |
| `AA-114` | Field descriptions and view patching | no |
| `AA-115` | Elevated-rights reads | no |

## 12. Mapping of the former rule identifiers

Two drafts of this folder existed before consolidation. One numbered its rules `AN-RULE-001` to
`AN-RULE-096`; the other stated its rules in per-entity validation tables without identifiers. The
table below maps the single scheme of this file to both.

| This file | Former numbered draft | Former table-based draft |
|---|---|---|
| `AA-001` | `AN-RULE-001` | Analytic Plan field table, name |
| `AA-002` | `AN-RULE-002` | Analytic Plan field table, parent |
| `AA-003` | `AN-RULE-003` | Analytic Plan validation table, first row |
| `AA-004` | `AN-RULE-004` | Dynamic column contract, column naming |
| `AA-005` | `AN-RULE-005` | Dynamic column contract, column naming |
| `AA-006` | `AN-RULE-006` | Dynamic column contract, grouping column |
| `AA-007` | — | Dynamic column contract, synchronisation |
| `AA-008` | — | Dynamic column contract, grouping column |
| `AA-009` | `AN-RULE-009` | — |
| `AA-010` | `AN-RULE-010` | Plan lifecycle, deletion |
| `AA-011` | `AN-RULE-011` | Moving analytic items between columns; Analytic Account validation table |
| `AA-012` | `AN-RULE-012` | Plan lifecycle, re-parenting |
| `AA-013` | `AN-RULE-013` | — |
| `AA-014` | — | Deleting a plan and its columns, step 4 |
| `AA-015` | `AN-RULE-008` | Plan lifecycle, rename |
| `AA-016` | `AN-RULE-016` | Analytic Plan validation table, second row |
| `AA-017` | `AN-RULE-015` | — |
| `AA-018` | `AN-RULE-017` | — |
| `AA-019` | `AN-RULE-014` | Analytic Plan field table, default applicability |
| `AA-020` | `AN-RULE-007`, `AN-RULE-090` | Applicability lifecycle and caching; relevant-plans answer |
| `AA-021` | — | Analytic Plan multi-company behaviour |
| `AA-022` | `AN-RULE-018` | Applicability field table |
| `AA-023` | `AN-RULE-019` | Applicability scoring |
| `AA-024` | `AN-RULE-020` | Applicability algorithm, step 3 |
| `AA-025` | `AN-RULE-021` | Why the baseline is zero point five |
| `AA-026` | `AN-RULE-022` | The score of one rule |
| `AA-027` | — | The no-business-domain case (corrected) |
| `AA-028` | `AN-RULE-023` | The relevant-plans answer, step 2 |
| `AA-029` | `AN-RULE-024` | The relevant-plans answer, step 3 |
| `AA-030` | `AN-RULE-025` | Applicability algorithm, step 1 |
| `AA-031` | `AN-RULE-027` | Analytic Account validation table |
| `AA-032` | `AN-RULE-028` | Analytic Account field table, plan |
| `AA-033` | `AN-RULE-029` | — |
| `AA-034` | `AN-RULE-030` | — |
| `AA-035` | `AN-RULE-031` | — |
| `AA-036` | `AN-RULE-035` | Analytic Account field table, customer |
| `AA-037` | `AN-RULE-026` | Analytic Account field table |
| `AA-038` | `AN-RULE-032` | Analytic Account field table, active |
| `AA-039` | `AN-RULE-033` | Ordering, display and search, duplication |
| `AA-040` | `AN-RULE-034` | Ordering, display and search, display name |
| `AA-041` | `AN-RULE-036` | Percentage normalisation |
| `AA-042` | `AN-RULE-037` | The shape of a distribution |
| `AA-043` | `AN-RULE-038` | The merge algorithm, when it runs |
| `AA-044` | `AN-RULE-039` | The shape of a distribution, reserved key |
| `AA-045` | `AN-RULE-040` | The merge algorithm, degenerate cases |
| `AA-046` | `AN-RULE-041` | — |
| `AA-047` | `AN-RULE-042` | Validating a distribution, step 4 |
| `AA-048` | `AN-RULE-043` | Reading a key |
| `AA-049` | — | The merge algorithm |
| `AA-050` | — | The merge algorithm, degenerate cases |
| `AA-051` | — | The merge algorithm, step 6 |
| `AA-052` | — | The mixin's index |
| `AA-053` | `AN-RULE-044` | Searching a distribution |
| `AA-054` | `AN-RULE-044` | Searching a distribution |
| `AA-055` | `AN-RULE-044` | Filtering in memory |
| `AA-056` | `AN-RULE-045` | Grouping by distribution |
| `AA-057` | — | — (new, from the source) |
| `AA-058` | `AN-RULE-045` | Grouping by distribution |
| `AA-059` | `AN-RULE-046` | Distribution Model validation table |
| `AA-060` | `AN-RULE-047` | The condition fields |
| `AA-061` | `AN-RULE-048` | The condition fields, sixth row |
| `AA-062` | `AN-RULE-049` | The ordering, and what "more specific" means |
| `AA-063` | `AN-RULE-050` | Combining the matching models |
| `AA-064` | `AN-RULE-051` | — |
| `AA-065` | `AN-RULE-052` | Distribution Model field table |
| `AA-066` | `AN-RULE-053` | The mixin's constraint |
| `AA-067` | `AN-RULE-054` | Analytic Item validation table |
| `AA-068` | `AN-RULE-053` | Analytic Item field table |
| `AA-069` | `AN-RULE-055` | Shared primitives |
| `AA-070` | `AN-RULE-056` | The base formula |
| `AA-071` | `AN-RULE-057` | Analytic Item field table, company |
| `AA-072` | `AN-RULE-058` | Analytic Item multi-company behaviour |
| `AA-073` | `AN-RULE-061` | — |
| `AA-074` | `AN-RULE-062` | Analytic Item field table, journal item |
| `AA-075` | `AN-RULE-063` | Analytic Item field table, fiscal year search |
| `AA-076` | `AN-RULE-059` | — |
| `AA-077` | `AN-RULE-060` | Back-computing a distribution |
| `AA-078` | `AN-RULE-064` | Analytic Account purpose and lifecycle |
| `AA-079` | `AN-RULE-065` | When validation runs |
| `AA-080` | `AN-RULE-066` | Which journal items are checked |
| `AA-081` | `AN-RULE-067` | The message |
| `AA-082` | `AN-RULE-068` | The message, two presentations |
| `AA-083` | `AN-RULE-069` | Which journal items are checked |
| `AA-084` | `AN-RULE-070` | When validation runs, second caller |
| `AA-085` | `AN-RULE-071` | — |
| `AA-086` | `AN-RULE-072` | — |
| `AA-087` | `AN-RULE-073` | The last-slice rule, step 2.3 |
| `AA-088` | `AN-RULE-074` | The last-slice rule |
| `AA-089` | `AN-RULE-075` | The last-slice rule, first consequence |
| `AA-090` | `AN-RULE-076` | Rounding the slices and cancelling the error |
| `AA-091` | `AN-RULE-077` | Changing a distribution on a posted entry |
| `AA-092` | `AN-RULE-078` | Keeping the journal item's distribution in step |
| `AA-093` | `AN-RULE-079` | Keeping the journal item's distribution in step |
| `AA-094` | — | — (new, from the source) |
| `AA-095` | — | Writing a distribution onto an analytic item |
| `AA-096` | `AN-RULE-080` | Analytic Account multi-company behaviour |
| `AA-097` | `AN-RULE-081` | Applicability and model multi-company behaviour |
| `AA-098` | `AN-RULE-082` | Field tables, company-checked fields |
| `AA-099` | `AN-RULE-083` | The debit, credit and balance |
| `AA-100` | `AN-RULE-084` | The algorithm, steps 1 to 3 |
| `AA-101` | — | The algorithm, second detail |
| `AA-102` | `AN-RULE-085` | Shared primitives |
| `AA-103` | `AN-RULE-086` | Validating a distribution |
| `AA-104` | `AN-RULE-087` | Ordering sections of each entity |
| `AA-105` | `AN-RULE-088` | — |
| `AA-106` | `AN-RULE-089` | Quantities, products and other copied values |
| `AA-107` | — | — (new, from the source) |
| `AA-108` | — | The sixty and forty worked example |
| `AA-109` | `AN-RULE-091` | Configuration and security |
| `AA-110` | `AN-RULE-092` | Configuration and security |
| `AA-111` | `AN-RULE-093` | — |
| `AA-112` | `AN-RULE-094` | — |
| `AA-113` | `AN-RULE-095` | — |
| `AA-114` | `AN-RULE-096` | Defaulting and presentation |
| `AA-115` | — | The base plan; company consistency |

## 13. Reconciliation notes

1. **Scoring without a business domain.** The numbered draft said the prefix and category criteria
   are still evaluated when the situation supplies no business domain; the table-based draft said
   an early return makes that impossible. The source has two layers: the base scoring returns
   early, and the layer added by the general ledger capability then adds the prefix and category
   points on top of whatever the base returned, unless the base already eliminated the rule.
   `AA-027` states the source behaviour, and the scenarios of
   [acceptance-criteria.md](acceptance-criteria.md) cover both readings.
2. **Rule ties.** Only the numbered draft stated that a tie keeps the rule examined first. The
   source compares strictly greater, and rules are read in internal identifier order, so the
   earlier rule keeps the win: `AA-105`.
3. **The base-plan parameter message.** One draft rewrote the message in full words. Messages are
   reproduced, so `AA-017` quotes it as emitted.
4. **The prefix cell.** `AA-022` no longer states which business domains show the prefix cell; that
   presentation rule lives with the field, in [entities.md](entities.md) section 4.1, because two
   capabilities contribute to it.
5. **The label of the analytic accounting setting.** Both drafts paraphrased it as "Track costs and
   revenues by project, department, and other axes". The emitted text is "Track costs & revenues by
   project, department, etc", and a label is reproduced rather than authored, so `AA-110` now quotes
   it as displayed. [configuration.md](configuration.md) section 1.1 carries the same text together
   with the hover text.
6. **The visibility of the three derived totals.** One draft restricted the debit and the credit to
   the accounting groups and left the balance open to every member of the analytic accounting group.
   The account list hides the debit and the credit outright without the accounting capability and,
   with it, restricts all three columns to the read-only accounting group or the invoicing group;
   only the *Gross Margin* button of the form is unrestricted. `AA-111` now states both halves, and
   [configuration.md](configuration.md) section 8 repeats them per field.
