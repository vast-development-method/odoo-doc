# Financial Reporting — Glossary

Every term used in this domain, defined in full. Terms are listed alphabetically. A term written
in *italics* inside a definition is itself defined in this glossary.

---

**Account.** A heading of the chart of accounts on which *Journal Items* are posted. Carries a
code, a name, an *account type*, an *internal group*, a set of *account tags* and a flag saying
whether it carries its balance forward. Read by this domain, never written.

**Account code prefix engine.** The *computation engine* whose stored value is `account_codes`.
Its formula is a signed sum of *terms*, each selecting *accounts* by the beginning of their code
or by an *account tag*, optionally excluding sub-prefixes and optionally keeping only the
accounts whose own balance is a debit or a credit. See
[`calculations.md`](calculations.md) §6.

**Account tag.** A label attached to an *account* or produced by a tax's *repartition line* and
stamped onto a *Journal Item*. Tags whose applicability is "taxes" are the *tax tags* this domain
creates and destroys; tags whose applicability is "accounts" classify accounts, notably into the
three activity sections of the *cash flow statement*.

**Account type.** One of nineteen classifications of an *account* — receivable, bank and cash,
current assets, non-current assets, prepayments, fixed assets, payable, credit card, current
liabilities, non-current liabilities, equity, current year earnings, income, other income,
expenses, other expenses, depreciation, cost of revenue, off-balance sheet. Determines which
statement line an account falls under and whether it carries its balance forward.

**Advance tax payment account.** The account of a *tax group* holding instalments already paid to
the authorities. Its balance is consumed by the *tax closing entry* before the *tax payable
account* is used.

**Aged payable.** The statement listing open *payable* items by partner and by how long they have
been outstanding at a stated reference date, split into *aging buckets*.

**Aged receivable.** The same statement for *receivable* items.

**Aging bucket.** One of the six columns of an aged statement: not due, one to thirty days, thirty-one
to sixty, sixty-one to ninety, ninety-one to one hundred and twenty, and older. Assignment is by
*days overdue* at the reference date.

**Aggregation engine.** The *computation engine* whose stored value is `aggregation`. Its formula
is arithmetic over references to other *expressions*, written as a *line code* followed by a
period and an *expression label*, or the single keyword `sum_children`. It never reads the
ledger. See [`calculations.md`](calculations.md) §5.

**Annotation.** A free text a user attaches to a rendered line for a given period end date. It is
displayed beside the line and printed as a numbered footnote.

**Applied carry-over expression.** The *expression* whose label begins with
`_applied_carryover_`, which reports how much was carried **into** the current period. Always
uses the *external value engine*.

**Audit drill-down.** The operation that lists the *Journal Items* behind a displayed figure. Only
available on an *auditable* expression.

**Auditable.** A flag on an *expression*, true by default for every *computation engine* except
the *custom function engine*, saying that the figure offers the *audit drill-down*.

**Audit trail.** The list of tracked changes to accounting records, with the date and time, the
author, the record, the type of change and the old and new values of every changed field. Made
immutable by the *restricted audit trail* setting.

**Availability condition.** The rule deciding whether a *Report Definition* is offered for the
active company: always, when the country matches, or when the chart of accounts matches.

**Balance.** See *signed balance*.

**Balance character.** The letter `D` or `C` optionally ending a *term* of the *account code
prefix engine*, keeping only the *accounts* whose own balance over the same window is a debit or
a credit respectively.

**Balance sheet.** The statement showing assets, liabilities and equity at one date. Balances by
construction because every *Journal Entry* balances.

**Base restriction.** The conjunction of conditions every ledger-reading *computation engine*
applies: company, entry state, journal, analytic, partner, account type, unreconciled, row kind
and *tax exigibility*. See [`calculations.md`](calculations.md) §2.3.

**Blank if zero.** A flag on an *expression* or a *Report Column* that displays an empty cell
instead of a zero. It changes only the display; an *aggregation* referencing the expression still
receives zero.

**Bound clause.** A *subformula* clause of the *aggregation engine* or the *external value
engine* that conditions or clamps the figure: `if_above`, `if_below`, `if_between`,
`if_other_expr_above`, `if_other_expr_below`.

**Carried in.** The amount an *applied carry-over expression* reports for the current period.

**Carried out.** The amount a *carry-over expression* computes for transfer to the next period.

**Carry-over.** The mechanism transferring an amount from one period's declaration to the next
without creating any *Journal Entry*, using *Report External Values*. See
[`calculations.md`](calculations.md) §12.

**Carry-over expression.** The *expression* whose label begins with `_carryover_`, which computes
how much must be carried out.

**Carry-over target.** The field naming the *applied carry-over expression* that receives a
*carry-over expression*'s amount, when it is not the one on the same line.

**Cash basis.** The accounting method under which a tax becomes due when the cash moves rather
than when the document is issued. Implemented through *tax exigibility* and the cash-basis
entries created on reconciliation.

**Cash flow statement.** The statement showing how the period's activity changed cash and cash
equivalents, split into operating, investing and extraordinary, financing and unclassified
activities by the *account tags* of the counterpart accounts.

**Column.** See *Report Column*.

**Comparison period.** An additional period whose figures are displayed beside the current
period's, obtained by shifting the current period back by one period, by one year, or by an
arbitrary user-supplied range.

**Composite report.** A *Report Definition* assembled from *sections*, each of which is itself a
whole report. Rendered and printed as one document.

**Computation engine.** The interpreter of an *expression*'s formula. Six exist: the *record
filter engine* (`domain`), the *tax tag engine* (`tax_tags`), the *aggregation engine*
(`aggregation`), the *account code prefix engine* (`account_codes`), the *external value engine*
(`external`) and the *custom function engine* (`custom`).

**Continuation marker.** The row appended after the sub-lines of a *partially loaded* expansion,
carrying the offset reached and the number of sub-lines remaining.

**Cumulative translation adjustment.** The residual difference that arises when balances held in
other currencies are consolidated at their recorded rates. Presented on a dedicated line so that
the *balance sheet* still balances.

**Current year earnings.** The *account type* that does not carry its balance forward, and the
*balance sheet* line holding the profit or loss of the current fiscal year. Equals the net result
of the *profit and loss* for the year to date.

**Custom function engine.** The *computation engine* whose stored value is `custom`. Its formula
names a function supplied by an extension package and its *subformula* names the key to read in
the mapping the function returns. Not *auditable* by default.

**Date scope.** The per-*expression* rule deciding which accounting dates the *Journal Items* it
reads must fall in: `strict_range`, `from_beginning`, `from_fiscalyear`,
`to_beginning_of_fiscalyear`, `to_beginning_of_period`, `previous_return_period`. See
[`calculations.md`](calculations.md) §9.

**Days overdue.** The whole number of days between an item's due date and the reference date of
an aged statement, positive when the item is late.

**Declared figure.** The amount a tax box shows on the legal declaration, which may differ from
the *displayed* movement when a *carry-over* floors it at zero.

**Display type.** How a figure is rendered: monetary, percentage, integer, float, date, datetime,
boolean or string. Declared on an *expression* or, when the expression leaves it empty, on the
*Report Column*.

**Drill-down.** See *audit drill-down* and *unfolding*.

**Editable cell.** A cell whose *expression* uses the *external value engine* with the `editable`
*subformula* clause, into which a user with the administrator group may type a figure, creating a
*manual value*.

**Effective lower bound.** The lower bound of a *date scope* after the *carry-forward correction*
has been applied for the *account* being read: none for an account that carries its balance
forward, the start of the fiscal year otherwise.

**Engine.** See *computation engine*.

**Exigibility.** See *tax exigibility*.

**Expression.** See *Report Expression*.

**Expression label.** The short name distinguishing the *expressions* of one *Report Line*, and
the key by which a *Report Column* selects which expression to display. Conventionally `balance`
for a single-figure line and `base` and `tax` for a tax box.

**External value.** See *Report External Value*.

**External value engine.** The *computation engine* whose stored value is `external`. Its formula
is `sum` or `most_recent`, and it reads *Report External Values* rather than the ledger.

**Figure.** One computed number, identified by the triple of *Report Line*, *expression label*
and period column.

**Filter switch.** One of the fifteen boolean or selection fields on a *Report Definition* that
turn a user-facing filter on or off for that report.

**Fiscal year.** The company's accounting year, defined by a last day and a last month, or by
explicit fiscal year records when a period is shorter or longer than twelve months. Bounds every
fiscal-year-relative *date scope*.

**Foldable.** A flag on a *Report Line* making it start folded rather than expanded.

**Formula.** The text a *computation engine* interprets. Stored whitespace-normalized.

**Formula shortcut.** One of five non-stored fields on a *Report Line* that create, update or
delete the line's `balance` *expression* of a given engine in one stroke, so that a data package
can declare a line and its expression together.

**General ledger.** The statement listing, per *account*, the opening balance, the period's
debits and credits and the closing balance, each account unfolding into its *Journal Items*.

**Grouping key.** A comma-separated list of *Journal Item* field names on a *Report Line*, making
the line expand into one sub-line per distinct combination of those field values. Exists in two
forms: the authored one, fixed by the definition, and the user one, changeable on the rendered
report.

**Growth comparison.** The extra percentage column shown when exactly one *comparison period* is
selected, giving the relative change of each figure with the absolute value of the comparison
figure as denominator.

**Hard lock date.** The *lock date* that admits no exception and is never moved by this domain.

**Hash chain.** The sequence of digests linking the secured *Journal Entries* of a journal, each
computed from the previous digest and the entry's own hashed fields. See
[`calculations.md`](calculations.md) §17.1.

**Hash integrity check.** The verification recomputing every hash of every chain and reporting,
per journal and *sequence prefix*, whether the chain is verified, corrupted or empty.

**Hide if zero.** A flag on a *Report Line* hiding it and its descendants when all its figures
are zero.

**Hierarchy filter.** The option nesting *account* lines inside their account groups.

**Horizontal split.** The presentation showing a report as two facing halves, driven by the
left-or-right side declared on each *Report Line* and inherited by its children.

**Initial balance.** The accumulated balance of an *account* before the first day of the report
period. Produced by the *date scope* `to_beginning_of_period`.

**Integer rounding.** The optional per-report rounding of every monetary figure to a whole unit
of currency, in one of three modes: nearest, up (away from zero) or down (towards zero).

**Internal group.** The coarse classification of an *account* into equity, asset, liability,
income, expense or off-balance. Derived from the *account type*.

**Journal.** The book a *Journal Entry* belongs to. Journals have types: sales, purchase, cash,
bank, credit card and miscellaneous. The *tax return journal* is of type miscellaneous.

**Journal audit.** The statement listing, per journal, the entries of the period and, for sales
and purchase journals, a tax summary and a report of numbering gaps.

**Journal Entry.** The balanced document of the ledger. Read by this domain; created by it only
as the *tax closing entry*.

**Journal Item.** One line of a *Journal Entry*, carrying an account, a debit, a credit, a
partner, a date, tags and an analytic distribution. The only source of ledger figures.

**Line.** See *Report Line*.

**Line code.** The short identifier of a *Report Line*, unique within its report, by which
*aggregation* formulas, *carry-over targets* and filing formats address it.

**Load more limit.** The maximum number of sub-lines one expansion produces before a
*continuation marker* is emitted instead.

**Lock date.** A date on the company on or before which entries may not be created or modified.
Five exist: global, tax return, sales, purchase and hard. The first four are *soft lock dates*.

**Lock date exception.** A grant relaxing a *soft lock date* for a named user, a named company,
optionally one lock date field, for a bounded time.

**Manual value.** A *Report External Value* typed by a user into an *editable cell*. Recognized
by having no origin line.

**Most recent.** The *external value engine* formula selecting the last *Report External Value*
of the window in the ordering by date then by identifier.

**Net position.** The sum of the closing amounts of every tax account in a period. Negative means
the company owes the authorities; positive means the authorities owe the company.

**Partial reconciliation.** A partial match between a debit item and a credit item, reducing both
residuals. Its own date decides whether an aged statement at a past reference date sees it.

**Partner ledger.** The statement listing, per partner, the *Journal Items* on receivable and
payable accounts with a running balance.

**Period comparison.** See *comparison period*.

**Prefix group.** An intermediate level inserted when an expansion would produce more sub-lines
than the *prefix groups threshold*, grouping the candidates by the shortest prefix of their
grouping string that yields more than one group.

**Prefix groups threshold.** The per-report number, four thousand by default, above which
*prefix groups* are used.

**Presentation currency.** The currency in which a report's figures are displayed: that of the
first selected company.

**Previous return period.** The *date scope* covering the whole tax return period immediately
before the one containing the report's end date, using the company's periodicity.

**Profit and loss.** The statement showing income less expenses for a period, also called the
income statement.

**Record filter engine.** The *computation engine* whose stored value is `domain`. Its formula is
a condition over *Journal Item* fields and its mandatory *subformula* is one of `sum`,
`sum_if_pos`, `sum_if_neg` or `count_rows`, optionally negated.

**Repartition line.** The rule of a tax deciding how the taxed amount is split into base and tax
parts, which account each part posts to, and which *tax tags* each part stamps. Carries the *use
in tax closing* flag.

**Report Column.** The entity declaring one displayed column, bound to an *expression label*.

**Report Definition.** The entity holding a report's name, availability, *filter switches*,
presentation rules and column set.

**Report Expression.** The entity holding one *figure* of one *Report Line*: a label, a
*computation engine*, a formula, a *subformula*, a *date scope* and display flags.

**Report External Value.** The entity holding a figure that is not in the ledger: a *manual
value* typed by a user, or a *carry-over* amount written when a period is closed. Dated,
company-scoped and attached to exactly one *Report Expression*.

**Report Line.** The entity holding one row of a report: its name, its place in the tree, its
optional *line code*, its behavior flags and its *expressions*.

**Residual.** The unreconciled part of a *Journal Item*, computed at a reference date by
subtracting only the *partial reconciliations* dated on or before that date.

**Restricted audit trail.** The company setting making tracked-change messages on accounting
records immutable.

**Restricted journal.** A journal whose posted entries join a *hash chain*. The setting cannot be
switched off once an entry has been posted in it.

**Root report.** A generic, country-neutral *Report Definition* that national *variants* refer
to. A report with no root report is itself a root report.

**Rounding unit.** The user-chosen presentation scale: units, thousands or millions.

**Section.** A whole *Report Definition* used as one part of a *composite report*.

**Secured entry.** A posted *Journal Entry* carrying an inalterability hash.

**Secured sequence number.** The gap-free number assigned to a *secured entry*, used to order the
*hash chain* across *sequence prefixes*.

**Sequence prefix.** The part of an entry's number that does not vary within a numbering period —
the journal code and the period part. The *hash integrity check* reports one finding per prefix.

**Signed balance.** The debit of a *Journal Item* minus its credit. Positive for a debit,
negative for a credit. The only ledger quantity the engines read.

**Soft lock date.** A *lock date* that a *lock date exception* can relax: the global, tax return,
sales and purchase lock dates.

**Subformula.** The optional text modifying a *computation engine*'s interpretation of its
formula. Mandatory for the *record filter engine*.

**Sum children.** The *aggregation engine* formula summing, over the direct children of the
line, the *expression* of each child carrying the same label as the summing expression.

**Tax check.** One automatic verification attached to a *tax return*, in state to review, passed,
anomaly, reviewed or supervised. A check in anomaly blocks validation.

**Tax closing entry.** The single *Journal Entry* this domain creates: it clears the period's tax
account movements against the *tax payable account*, the *tax receivable account* and the
*advance tax payment account*. See [`accounting-effects.md`](accounting-effects.md) §2.

**Tax exigibility.** Whether a tax is due on the document or on the payment. An item of a
payment-basis tax enters a tax report only once its cash-basis entry exists. See
[`calculations.md`](calculations.md) §2.4.

**Tax grid.** The user-facing name of a *tax tag* as configured on a tax.

**Tax group.** The entity grouping taxes that are declared and paid together, holding the *tax
payable account*, the *tax receivable account* and the *advance tax payment account*.

**Tax payable account.** The account credited by the *tax closing entry* when the *net position*
is in favour of the authorities.

**Tax receivable account.** The account debited by the *tax closing entry* when the *net
position* is in favour of the company.

**Tax report.** Any report declaring tax to an authority. The application ships one generic
definition with two grouping variants; the country packages ship one hundred and sixty-two
national definitions.

**Tax return.** The object turning one rendering of one *tax report*, for one period and one
company or *tax unit*, into a filed declaration and a posted *tax closing entry*. Has three
steps: review, submit, pay.

**Tax return journal.** The miscellaneous journal in which the *tax closing entry* is posted.

**Tax return lock date.** The *soft lock date* applying to entries carrying taxes, moved forward
automatically when a *tax return* is validated.

**Tax tag.** An *account tag* whose applicability is "taxes", created by a *tax tag expression*,
stamped onto *Journal Items* by a tax's *repartition lines*, and read back by that expression.
Exactly one tag exists per formula name and country; the leading minus of a formula is a sign
instruction, not part of the name.

**Tax tag engine.** The *computation engine* whose stored value is `tax_tags`. Its formula is a
tag name, optionally prefixed with a minus.

**Tax unit.** A set of companies filing one declaration through a representative. The report is
consolidated for the unit; the *tax closing entry* is produced per company.

**Term.** One signed piece of an *account code prefix engine* formula, consisting of an optional
sign, a selector, an optional exclusion list and an optional *balance character*.

**Trial balance.** The statement listing, per *account*, the initial balance split by sign, the
period's gross debits and credits, and the end balance split by sign. Its control property is
that the two period columns have equal totals.

**Unclassified activities.** The section of the *cash flow statement* receiving cash movements
whose counterpart account carries none of the three activity *account tags*.

**Unfolding.** Expanding a *Report Line* into its children, its grouped sub-lines or its
generated sub-lines.

**Unreconciled filter.** The option restricting a report to *Journal Items* on reconcilable
accounts whose residual is not zero.

**Use in tax closing.** The flag on a *repartition line* deciding whether its *Journal Items*
participate in the *tax closing entry*. Computed as: the repartition type is "of tax", an account
is set, and the account's *internal group* is neither income nor expense. Overridable by hand.
A repartition line that participates does not propagate the document's analytic distribution.

**Variant.** A country-specific *Report Definition* referring to a *root report*. Selected from
the variant selector when the root report is opened.

**Window.** The interval of accounting dates a *date scope* produces for a given report period
and a given *account*.
