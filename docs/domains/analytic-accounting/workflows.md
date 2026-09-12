# Analytic Accounting — Workflows

Every operational procedure of the domain, end to end, with its actor, its preconditions, its
numbered steps, the records each step creates or changes, the operations invoked, the messages
emitted and its postconditions and failure conditions. The state tables that used to close this
file now live in [state-machines.md](state-machines.md), which carries the same transitions with
their guards and their refusal messages.

Vocabulary used in the steps:

- **Model that carries analysis**, or **plan-bearing model**: any model that implements the
  Analytic Plan Fields Mixin, which means it holds one analytic account column per root plan.
  Analytic Line is the one this domain owns; inventory valuation and manufacturing cost holders are
  others.
- **Distribution-bearing model**: any model that implements the Analytic Mixin, which means it
  holds an analytic distribution. Journal Item, Sales Order Line, Purchase Order Line, Expense,
  Asset, Reconciliation Model Line, Work Centre, the withholding tax line and Analytic Distribution
  Model are the ones in this edition.
- **Back-computation**: the recomputation of a journal item's distribution from its analytic lines.
- **The synchronisation guard**: the marker a caller raises so that writing analytic lines does not
  rewrite the distribution they came from, and writing a distribution does not delete and
  regenerate analytic lines. Every operation in which the system itself writes analytic lines runs
  with the guard raised.

Contents:

1. [Enable analytic accounting](#workflow-1-enable-analytic-accounting)
2. [Create a root plan](#workflow-2-create-a-root-plan)
3. [Create a sub-plan](#workflow-3-create-a-sub-plan)
4. [Rename a plan](#workflow-4-rename-a-plan)
5. [Turn a root plan into a sub-plan](#workflow-5-turn-a-root-plan-into-a-sub-plan)
6. [Turn a sub-plan into a root plan](#workflow-6-turn-a-sub-plan-into-a-root-plan)
7. [Delete a plan](#workflow-7-delete-a-plan)
8. [Designate another base plan](#workflow-8-designate-another-base-plan)
9. [Define the applicability of a plan](#workflow-9-define-the-applicability-of-a-plan)
10. [Create an analytic account](#workflow-10-create-an-analytic-account)
11. [Move an account to another plan](#workflow-11-move-an-account-to-another-plan)
12. [Change the company of an analytic account](#workflow-12-change-the-company-of-an-analytic-account)
13. [Archive an analytic account](#workflow-13-archive-an-analytic-account)
14. [Create an analytic distribution model](#workflow-14-create-an-analytic-distribution-model)
15. [Create a distribution model from a document](#workflow-15-create-a-distribution-model-from-a-document)
16. [Fill the analytic distribution of a document line](#workflow-16-fill-the-analytic-distribution-of-a-document-line)
17. [Automatic proposal of a distribution on a journal item](#workflow-17-automatic-proposal-of-a-distribution-on-a-journal-item)
18. [Post a journal entry and generate analytic lines](#workflow-18-post-a-journal-entry-and-generate-analytic-lines)
19. [Change the distribution of a posted journal item](#workflow-19-change-the-distribution-of-a-posted-journal-item)
20. [Reset a journal entry to draft](#workflow-20-reset-a-journal-entry-to-draft)
21. [Reverse a posted document](#workflow-21-reverse-a-posted-document)
22. [Import a journal entry with analytic lines](#workflow-22-import-a-journal-entry-with-analytic-lines)
23. [Edit or delete an analytic line of a posted journal item](#workflow-23-edit-or-delete-an-analytic-line-of-a-posted-journal-item)
24. [Mass-edit the distribution of several journal items](#workflow-24-mass-edit-the-distribution-of-several-journal-items)
25. [Split analytic lines by writing a distribution](#workflow-25-split-analytic-lines-by-writing-a-distribution)
26. [Enter a manual analytic line](#workflow-26-enter-a-manual-analytic-line)
27. [Validate mandatory plans when confirming a source document](#workflow-27-validate-mandatory-plans-when-confirming-a-source-document)
28. [Redistribute the analytic lines of a valuation document](#workflow-28-redistribute-the-analytic-lines-of-a-valuation-document)
29. [Consult analytic reporting](#workflow-29-consult-analytic-reporting)
30. [Consult the analytic sections of a project's profitability](#workflow-30-consult-the-analytic-sections-of-a-projects-profitability)
31. [Reconciliation notes](#reconciliation-notes)

---

## Workflow 1: Enable analytic accounting

**Actor.** A user of the accounting administration group.

**Preconditions.** The accounting settings screen is reachable.

**Steps.**

1. The actor opens the accounting settings and, in the block titled "Analytics", switches on the
   setting whose text is "Track costs & revenues by project, department, etc", reproduced as the
   system displays it.
2. Switching it on immediately switches on the full accounting capability setting as well; this is
   a form-level consequence applied before saving.
3. The actor saves. The analytic accounting permission group is granted to every user of the
   internal user group, which makes the following visible: the analytic plans, analytic accounts,
   analytic distribution models and analytic items screens, the analytic distribution cell on
   journal items and on every other distribution-bearing record, and the analytic columns of the
   reports.
4. Alternatively, switching on "Use budgets to compare actual with expected revenues and costs"
   switches the analytic setting on as well; the budgeting capability itself is not part of this
   edition, so nothing else happens.

**Postconditions.** The permission group is granted. No record of this domain is created: the
shipped plan named *Project*, the `Percentage Analytic` precision and the `analytic.project_plan`
parameter already exist from the installation of the capability.

**Failure conditions.** None. Switching the setting off revokes the group and hides the screens;
the records stay.

---

## Workflow 2: Create a root plan

**Actor.** A member of the analytic accounting permission group.

**Preconditions.** None.

**Steps.**

1. The actor opens the analytic plans screen, which lists only the plans with no parent, and
   chooses to create one.
2. The actor fills the name (required), optionally the description, leaves the parent empty,
   chooses the default applicability (required on the form of a root plan; the shipped default is
   `optional`) and optionally changes the colour index and the sequence.
3. On save the system:
   1. drops the cached list of relevant plans;
   2. computes the plan's column name, `x_plan` followed by the plan's identifier and `_id`
      ([calculations.md](calculations.md) section 1);
   3. creates, on every model that carries analysis, a stored link to Analytic Account with that
      name, labelled with the plan's name, copied when a record is duplicated, with the deletion
      rule restrict;
   4. creates on each such model a balanced-tree index over that column restricted to the rows
      where it is not empty.
4. Every open client reloads its definition of the models that carry analysis, because their set of
   fields has changed.

**Records written.** One Analytic Plan; one field definition and one index per model that carries
analysis.

**Postconditions.** Every analytic line, and every other plan-bearing record, now has one more
analytic account column. The new plan does not appear in any distribution editor yet, because its
sub-tree contains no account.

**Failure conditions.** A missing name is refused by the required-field check (`AA-001`).

---

## Workflow 3: Create a sub-plan

**Actor.** A member of the analytic accounting permission group.

**Preconditions.** A plan exists to be the parent.

**Steps.**

1. From the parent plan's form the actor presses the *Subplans* button, which opens the list of its
   children with the parent and the colour pre-filled, and creates one; or the actor creates a plan
   and sets its parent by hand.
2. The form hides the default applicability and the applicability page as soon as the parent is
   filled: applicability is a property of the root plan only.
3. On save the system:
   1. drops the cached list of relevant plans;
   2. deletes the stored column of this plan if it had one — a plan created directly with a parent
      never had one;
   3. computes the depth from the materialised path and the grouping column's name;
   4. creates, on every model that carries analysis, that derived read-only link to Analytic Plan,
      labelled with the root plan's name and the depth in parentheses, resolving through the root
      plan's column to the account's plan and then up *depth minus one* parent links, unless the
      column already exists, in which case only its label is refreshed;
   5. repeats the whole treatment for every child of the plan, walking the plans in materialised-path
      order, which guarantees that parents are always processed before their children.

**Postconditions.** The sub-plan owns no column. Accounts attached to it are stored in the root
plan's column. One grouping column per depth exists, which lets reports be grouped by sub-plan
level.

**Failure conditions.** Choosing the plan itself or one of its descendants as the parent is
impossible: the selectable set excludes them (`AA-002`).

---

## Workflow 4: Rename a plan

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor changes the name and saves.
2. The system re-synchronises the columns of every model that carries analysis: a root plan's stored
   column takes the new name as its label; the grouping column of each depth of that hierarchy takes
   the new root name and its depth in parentheses.
3. Every open client reloads.

**Postconditions.** The column names themselves never change; only the labels do. Renaming is
therefore always safe with respect to stored data and stored view definitions (`AA-015`).

---

## Workflow 5: Turn a root plan into a sub-plan

**Actor.** A member of the analytic accounting permission group.

**Preconditions.** The plan is not the base plan. The target parent is neither the plan itself nor
one of its descendants.

**Steps.**

1. The actor sets the parent on the plan's form. When the plan being edited is the base plan, the
   form refuses immediately with "You cannot add a parent to the base plan 'the plan name'" and
   nothing is written.
2. **Before** the parent is written, the system migrates the analytic line values. The current
   column is the plan's own column, the new column is the column of the future parent's root plan,
   and the accounts are every analytic account whose plan is the plan being demoted or any
   descendant of it:
   1. The system looks for a conflict: at least one analytic line where the new column already
      holds a value that is neither empty nor one of those accounts, while the current column holds
      one of them.
   2. When a conflict exists, the whole operation is refused with "Whoa there! Making this change
      would wipe out your current data. Let's avoid that, shall we?" and a button labelled "See
      them" that opens the offending analytic lines in a list. Nothing has been written.
   3. Otherwise, for every analytic line whose current column holds one of those accounts, the value
      is copied into the new column and the current column is cleared, in one bulk update; the
      cached analytic lines are then invalidated.
3. The parent is written. The column synchronisation removes the plan's own stored column from every
   model that carries analysis and creates or relabels the grouping column of its new depth, and
   does the same for every descendant.
4. Every open client reloads.

**Postconditions.** The demoted plan's accounts are now stored in the new root plan's column, the
demoted plan's column no longer exists, and a document line can carry only one account for the
merged hierarchy.

**Worked example.** Plan *One* (root, column `x_plan5_id`) and plan *Two* (root, column
`x_plan6_id`). An analytic line holds account 1 of *One* in `x_plan5_id` and nothing in
`x_plan6_id`. Setting the parent of *One* to *Two* moves account 1 into `x_plan6_id`, clears
`x_plan5_id` and then deletes `x_plan5_id`. Had the line held account 3 of *Two* in `x_plan6_id`,
the conflict check would have refused the change.

---

## Workflow 6: Turn a sub-plan into a root plan

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor clears the parent on the plan's form and saves.
2. The parent is written **first**. The column synchronisation removes the grouping column of the
   plan's former depth when no other plan remains at that level under that root, and creates the
   plan's own stored column on every model that carries analysis, with its partial index.
3. **After** the columns exist, the system migrates the analytic line values in the opposite
   direction: the new column is the promoted plan's own column, the current column is the former
   parent's root column, and the accounts are every analytic account whose plan is the promoted plan
   or a descendant of it. The same conflict check as in workflow 5 applies, with the same message.
4. Every open client reloads.

**Postconditions.** The promoted plan is independent again, its accounts are in its own column, and
a document line may now carry one account of each of the two hierarchies.

**Worked example with an intermediate level.** Plan *One* is a root plan, *Mid level* is its child,
and account 1 belongs to *Mid level*. An analytic line holds account 1 in *One*'s column. Demoting
*One* under *Two* moves account 1 into *Two*'s column, because the migration selects every account
whose plan is *One* **or any descendant**, not only the direct members. Promoting *One* again moves
account 1 back.

---

## Workflow 7: Delete a plan

**Actor.** A member of the analytic accounting permission group.

**Preconditions.** No stored view definition names the column being removed.

**Steps.**

1. The actor deletes the plan.
2. The system deletes the plan's own stored column from every model that carries analysis, together
   with the data it holds. When a stored view definition still names that column, the deletion is
   refused with "Cannot rename/delete fields that are still present in views:" followed by the list
   of field names and the name of the offending view, and nothing is deleted.
3. The plan and all its descendants are deleted, because the parent link cascades.
4. The system deletes the grouping columns that no longer correspond to any surviving plan: for each
   grouping column created for a given root plan and depth, the column is kept when at least one
   plan still exists with that root and at least that depth, and deleted otherwise.
5. The registry cache and the cached list of relevant plans are dropped, and every open client
   reloads.

**Postconditions.** The plan, its sub-plans, its column and the values that column held are gone.

**Failure conditions.** An analytic account of the deleted plan that is still held by an analytic
line in another column cannot itself be deleted (`AA-035`); in practice the accounts of a deleted
plan must be reassigned before the plan is deleted.

**Worked example.** A parent plan has two children, both at depth one, therefore one grouping column
for depth one exists. Deleting the first keeps the grouping column, because the second still
occupies depth one. Deleting the second afterwards removes it.

---

## Workflow 8: Designate another base plan

**Actor.** A user who may write system parameters.

**Preconditions.** No analytic line exists, or the actor accepts that no value will be migrated.

**Steps.**

1. The actor writes the system parameter `analytic.project_plan` with the identifier of another
   root plan, as a string of digits.
2. The guard refuses any value that is not digits, that names no plan, or that names a plan owning
   no stored column — which means a sub-plan — with "The value for the key must be the ID to a valid
   analytic plan that is not a subplan".
3. On success the former base plan is re-synchronised on every model that carries analysis and
   therefore receives a generated column of its own, **empty**; then the generated column the new
   base plan owned until now is deleted with its contents.

**Postconditions.** The fixed column `account_id` keeps the values it held, which are from now on
read as accounts of the new base plan. No value is migrated, which is why the operation belongs to
configuration time (`AA-018`).

---

## Workflow 9: Define the applicability of a plan

**Actor.** A member of the analytic accounting permission group.

**Preconditions.** The plan is a root plan.

**Steps.**

1. The actor opens the plan and sets the default applicability to `optional`, `mandatory` or
   `unavailable`. The value is stored per company: setting it while one company is active does not
   change the value seen from another company.
2. On the applicability page the actor adds rules inline. For each rule the actor picks the business
   domain (required), the applicability (required), optionally a company (the cell is shown only in
   a multi-company database, with the placeholder "Visible to all"), optionally an account prefix
   (the cell is shown only for the business domains that display it) and optionally a product
   category.
3. Each create, write or delete of a rule drops the cached list of relevant plans, so the next
   document line sees the new rules.

**Decisions the actor must understand.** A rule that names only a company never wins; a rule that
names the business domain always beats the default for that business domain; adding a prefix or a
category narrows the rule and eliminates it outside its narrow case; between two otherwise equal
rules, the one that also names the company wins. The complete arithmetic is in
[calculations.md](calculations.md) section 2.

**Postconditions.** The plan's applicability now depends on the situation: in every distribution
editor the plan appears as optional, appears with a red running total until it reaches one hundred
percent when mandatory, or does not appear at all when unavailable.

---

## Workflow 10: Create an analytic account

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor opens the analytic accounts screen, or the chart of analytic accounts offered from the
   accounting configuration menu, and chooses to create one.
2. The actor fills the name (required, with the placeholder "e.g. Project XYZ" reproduced from the
   form), optionally the customer, optionally the reference, chooses the plan (required, with no
   quick creation) and, in a multi-company database, optionally the company (with the placeholder
   "Visible to all").
3. On save the account becomes selectable in the column of its root plan on every model that carries
   analysis, and in the distribution editor for that root plan's column.

**Records written.** One Analytic Account. Every later change of its name, reference, archival flag
or customer is written to its discussion thread.

**Postconditions.** The account counts of its plan and of every ancestor plan increase by one. A
plan whose sub-tree had no account now appears in the distribution editors.

---

## Workflow 11: Move an account to another plan

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor changes the plan on the account and saves.
2. Before the change is applied, the system migrates the analytic line values from the column of the
   account's current root plan to the column of the new root plan, restricted to this one account,
   with the conflict check and the message of workflow 5.
3. When the two root plans are the same — a move between two sub-plans of one hierarchy — the column
   names are identical and nothing is migrated.

**Postconditions.** Every analytic line that carried the account now carries it in the new plan's
column, and the account's stored root plan is refreshed.

**Worked example.** An analytic line holds account 1 in the column of plan *One* and account 1 in
the column of plan *Two* at the same time, which an import can produce. Moving account 1 from *One*
to *Two* is allowed, because the conflict check tolerates a target column that already holds one of
the accounts being moved; the result is that the value is cleared from *One*'s column and kept in
*Two*'s.

---

## Workflow 12: Change the company of an analytic account

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor sets or changes the company on the account.
2. The system counts, with elevated rights, the analytic lines carrying the account whose company is
   neither the new company nor one of its descendants, and refuses when at least one exists, with
   "You can't change the company of an analytic account that already has analytic items! It's a
   recipe for an analytical disaster!".
3. Clearing the company, which makes the account shared, is always allowed, because every existing
   line then satisfies the condition.

**Postconditions.** The account's visibility changes. Analytic lines of branches of the company keep
pointing at the account.

---

## Workflow 13: Archive an analytic account

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor switches the archival flag off. A red ribbon reading "Archived" appears on the form and
   the change is written to the discussion thread.
2. The account disappears from the default lists and from the value lists of the distribution
   editors. Existing distributions and existing analytic lines keep referring to it.
3. Any attempt to post a journal entry whose journal items still name the archived account in their
   distribution is refused with "You cannot post an entry with an archived analytic account: the
   account names", the names being those of all the archived accounts found, separated by a comma
   and a space.

**Postconditions.** The account is out of use but its history is intact. Reactivating it removes the
posting block.

---

## Workflow 14: Create an analytic distribution model

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor opens the analytic distribution models screen and creates one, either in the editable
   list or in the form.
2. In the conditions group the actor fills any combination of partner, partner category, product,
   product category, account prefix and company. Every condition filled must hold for the model to
   apply; every condition left empty means "any value".
3. In the distribution group the actor opens the distribution editor. In this editor every plan is
   shown as optional whatever the applicability rules say, and the shortcut that creates a further
   model is disabled.
4. The actor orders the models by dragging the sequence handle. Among models with the same sequence
   the most recently created one is evaluated first.

**Records written.** One Analytic Distribution Model. The constraint of `AA-059` is checked: a model
whose distribution names an account belonging to one specific company must itself belong to that
company, failing which the save is refused with "You defined a distribution with analytic account(s)
belonging to a specific company but a model shared between companies or with a different company".

**Postconditions.** The next document line whose fields satisfy the conditions receives the model's
distribution as a proposal, subject to the combination rules of [calculations.md](calculations.md)
section 6.

---

## Workflow 15: Create a distribution model from a document

**Actor.** Any user editing a document line that carries a distribution.

**Steps.**

1. While the distribution editor is open on a document line, the actor presses the new-model
   shortcut.
2. A new analytic distribution model form opens, pre-filled with: the distribution currently typed
   in the editor, the document's partner, the document line's product, and the first three
   characters of the display name of the document line's financial account as the account prefix.
3. The actor adjusts the conditions and saves. Editing an existing model from this shortcut is not
   offered; only creation is.

**Postconditions.** A new model exists. The document line itself is not modified by this action.

---

## Workflow 16: Fill the analytic distribution of a document line

**Actor.** Any user with write access to the document: an accountant on a journal item, a
salesperson on a sales order line, a buyer on a purchase order line, an employee on an expense.

**Preconditions.** The analytic accounting permission group is granted. At least one plan has at
least one account in its sub-tree.

**Steps.**

1. The actor opens the document. The line's distribution may already have been proposed by the
   system when the line was created or when its account, partner or product changed; see workflow
   17.
2. The actor clicks the analytic distribution cell. The editor asks the system for the relevant
   plans, passing: the business domain of the document — `invoice` for a sale document, `bill` for a
   purchase document, `general` otherwise, and `expense`, `timesheet`, `sale_order`,
   `purchase_order`, `manufacturing_order` or `stock_picking` for the corresponding documents — the
   line's product, the line's financial account, the line's company, and the analytic accounts
   already present in the distribution.
3. The editor shows one column per relevant plan, in plan order, plus a percentage column and, when
   the host field names an amount field and the record's amount is not zero, a monetary column
   showing the line amount times the percentage. The header of each plan column shows the plan's
   running total, in red when the plan is mandatory and the total is not exactly one hundred
   percent, in green when the plan is mandatory and the total is exactly one hundred percent, and
   unstyled otherwise.
4. The actor adds a row. The percentage proposed for a new row is computed as follows: among the
   plans whose running total is below one hundred percent, take the largest total of the mandatory
   plans when at least one mandatory plan is incomplete, otherwise the largest total of the optional
   plans, otherwise the total of the rows that name no account at all; the proposed percentage is
   one hundred percent minus that value, floored at zero, and becomes one hundred percent when the
   result is zero.
5. The actor picks an account in one or more plan columns of the row and adjusts the percentage, or
   types a monetary value, in which case the percentage is recomputed as the typed value divided by
   the record's amount, at the percentage precision plus two working digits.
6. The actor may add further rows, delete rows — deleting the last row immediately adds an empty
   one — and repeat.
7. Closing the editor saves. The editor builds the document by dropping every row that names no
   account and by summing the percentages of rows that name exactly the same set of accounts. In
   single-record editing no update marker is produced, therefore the written document replaces the
   stored one entirely.
8. When the actor changes the line's financial account or product while the editor is open, the
   relevant plans are fetched again, because the applicability may have changed.

**Postconditions.** The line's distribution is stored with each percentage rounded to two decimal
places. On a posted journal item, the analytic lines are regenerated (workflow 19).

**Error paths.** When the stored distribution names an account that no longer exists, the editor
drops the missing identifiers and saves the cleaned document immediately. A mandatory plan left
incomplete is not refused here: it is refused at posting or at confirmation.

---

## Workflow 17: Automatic proposal of a distribution on a journal item

**Actor.** The system.

**Trigger.** A journal item is created, or its financial account, partner or product changes, while
the item is a product line or belongs to a document that is not an invoice or a receipt.

**Steps.**

1. The system computes the **related distribution**: the empty document by default; the
   distribution of the first linked sales order line when the sales capability is installed and such
   a link exists; the distribution of the linked purchase order line when the purchasing capability
   is installed and such a link exists.
2. The system collects the root plans of the accounts named in the related distribution.
3. The system asks the distribution models for a proposal, passing the line's product, the product's
   category, the partner, the partner's categories, the code of the line's financial account as the
   account prefix, the company, and the root plans collected in step 2 as already applied. The
   proposal is computed once per distinct set of arguments and reused for every line that shares
   them.
4. The line's distribution becomes the related distribution combined with the model proposal; when
   that combination is empty, the value already stored on the line is kept untouched (`AA-064`).

**Postconditions.** A line that already carries a manually typed distribution keeps it when no model
matches, and is overwritten when a model matches.

**Worked example.** Two models exist, one for a first partner giving account 3 and one for a second
partner giving account 4, both without a company condition. A new invoice with no partner receives
nothing. Setting the partner to the first sets the distribution to `{ "3" : 100 }`. Setting it to
the second sets it to `{ "4" : 100 }`. Setting it to a partner for which no model exists leaves
`{ "4" : 100 }` untouched. Clearing the partner also leaves `{ "4" : 100 }` untouched. Typing a
distribution by hand in the form after the proposal has been applied keeps the typed value when the
form is saved.

---

## Workflow 18: Post a journal entry and generate analytic lines

**Actor.** An accountant, or the automatic posting job.

**Preconditions.** The entry is in the draft state and is balanced.

**Steps.**

1. The general ledger performs its own posting validations; see
   [../general-ledger/workflows.md](../general-ledger/workflows.md).
2. The system collects, with elevated rights and with archived records included, every analytic
   account named in the distribution of every journal item of the entries being posted, and refuses
   the posting when any of them is archived, with "You cannot post an entry with an archived
   analytic account: the account names".
3. The accounting date of each entry is adjusted for the violated lock dates.
4. For all the journal items of all the entries being posted, in one batch:
   1. **Validation.** For every journal item whose display type is `product`, the distribution is
      validated against the mandatory plans, with the situation made of the company, the product,
      the financial account and the business domain (`invoice` for a sale document, `bill` for a
      purchase document, `general` otherwise). The validation runs only when the caller switched the
      validation flag on, which the posting button of the entry form and the mass validation
      assistant both do, and which programmatic and automatic posting do not.
      - When at least one journal item fails and exactly one entry is being posted, the posting is
        refused with "One or more lines require a 100% analytic distribution."
      - When at least one journal item fails and several entries are being posted, the posting is
        refused with the same message plus a redirect to the list of the offending journal items,
        titled "Items With Missing Analytic Distribution", reachable through a button labelled "See
        items".
   2. **Preparation.** For every journal item with a non-empty distribution, the candidate analytic
      lines are prepared as described in [calculations.md](calculations.md) section 9, and their
      amounts are corrected for rounding as described in section 10.
   3. **Creation.** All the candidate lines are created in one operation, with the synchronisation
      guard raised, which prevents the freshly created lines from immediately rewriting the
      distribution they came from.
5. The entry becomes posted.

**Records written.** One Analytic Line per surviving candidate, with the values listed in
[calculations.md](calculations.md) section 9.3.

**Postconditions.** Each journal item with a distribution has one analytic line per entry of its
distribution, minus the entries whose amount rounded to zero. The amounts of the analytic lines of
one journal item add up exactly to the negated balance of that journal item for every plan the
distribution completes to one hundred percent.

**Special cases.**

- An exchange difference entry created automatically during a reconciliation is posted with the
  validation flag explicitly switched **off**, so a mandatory plan never blocks it.
- A journal item that is not a product line — a tax line, an early payment discount line, a discount
  allocation line, a payment term line, a section or a note — is never validated, but it does
  generate analytic lines when it carries a distribution, because tax lines and discount lines
  inherit the distribution of the base lines they come from.
- A payment term line carries no distribution and produces nothing.

---

## Workflow 19: Change the distribution of a posted journal item

**Actor.** An accountant.

**Preconditions.** The entry is posted and the accounting date is not locked.

**Steps.**

1. The actor edits the analytic distribution cell of the journal item, or writes the field in a mass
   edit.
2. The system reads the distributions currently stored for the affected items straight from storage,
   before any recomputation, and merges each with the written document
   ([calculations.md](calculations.md) section 7): with an update marker only the named plans are
   replaced; without it the document replaces everything.
3. The system selects the journal items being written whose entry is posted, deletes all their
   analytic lines, and regenerates them exactly as in workflow 18 step 4, including the mandatory
   plan validation, the closing-line rule and the rounding correction. The regeneration runs with
   the synchronisation guard raised.
4. Journal items of a draft entry are not regenerated: only the distribution is stored.

**Postconditions.** The analytic lines match the new distribution. Their identifiers change, because
they are deleted and recreated. The financial ledger is untouched: no journal entry is created or
modified.

**Worked example.** A posted customer invoice line of 200.00 carries `{ "3" : 100 , "4" : 50 }`,
where accounts 3 and 4 both belong to the plan *Departments*. The analytic lines are 200.00 for
account 3 and 100.00 for account 4. Changing the distribution to `{ "3" : 100 , "4" : 25 }` deletes
both lines and creates 200.00 for account 3 and 50.00 for account 4.

---

## Workflow 20: Reset a journal entry to draft

**Actor.** An accountant.

**Preconditions.** The entry is posted or cancelled and the general ledger allows the reset.

**Steps.**

1. The general ledger performs its own checks.
2. Every analytic line of every journal item of the entry is deleted, with the synchronisation guard
   raised, therefore the distributions stored on the journal items are left untouched.
3. The entry becomes draft.

**Postconditions.** No analytic line refers to the entry any more; the distributions survive and
regenerate equivalent analytic lines, with new identifiers, when the entry is posted again.

---

## Workflow 21: Reverse a posted document

**Actor.** An accountant.

**Preconditions.** The document is posted.

**Steps.**

1. The general ledger creates the reversing entry, either as a draft or posted immediately; its
   journal items carry the **copied** distributions of the items they reverse, because the
   distribution is a copied field.
2. When the reversing entry is posted, workflow 18 runs for it like for any other entry. Because
   every balance is negated, every analytic amount is negated too.
3. The original analytic lines are **kept**; nothing deletes them.

**Postconditions.** The analytic account's balance returns to its value before the original entry,
and both movements remain visible with their own dates. The arithmetic and a worked example are in
[calculations.md](calculations.md) section 18.

---

## Workflow 22: Import a journal entry with analytic lines

**Actor.** An integration, or a user loading a file.

**Steps.**

1. The loading operation creates a draft journal entry whose journal items carry analytic line
   values directly, instead of a distribution.
2. Creating those analytic lines triggers the back-computation on their journal item, which rebuilds
   the journal item's distribution from them ([calculations.md](calculations.md) section 11).
3. Immediately after the journal items are created or written, the analytic lines belonging to
   journal items of a draft entry are deleted, with the synchronisation guard raised, because a
   draft entry must never own analytic lines.
4. Posting the entry later regenerates the analytic lines from the distribution reconstructed in
   step 2.

**Worked example.** A draft entry is created with two journal items. The first is a debit of
2 000.00 carrying one analytic line of −2 000.00 that names account 2 in the magic column and
accounts 1 and 3 in two plan columns; the second is a credit of 2 000.00 carrying one analytic line
of +2 000.00 that names account 2 only. After creation no analytic line exists, and the two journal
items carry `{ "2,1,3" : 100 }` and `{ "2" : 100 }` respectively. Writing a further analytic line on
the first journal item, naming only account 1, replaces its distribution with `{ "1" : 100 }`,
because the back-computation rebuilds the whole document from the lines that exist at that moment.
Posting the entry then creates the analytic lines again.

---

## Workflow 23: Edit or delete an analytic line of a posted journal item

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor opens the analytic items screen, finds the line and changes its amount, its accounts or
   the journal item it points at, or deletes it, or creates a new line pointing at a journal item.
2. After the change, the journal items affected — the one pointed at before the change and the one
   pointed at after it — have their distribution rebuilt from their remaining analytic lines
   ([calculations.md](calculations.md) section 11).
3. The rebuild writes the distribution with the synchronisation guard raised, therefore writing the
   rebuilt distribution does **not** delete and regenerate the analytic lines. The manual edit
   survives.

**Postconditions.** The distribution of the journal item and its analytic lines agree again.

**Error paths.** Setting a financial account that differs from the account of the journal item the
line points at is refused with "The journal item is not linked to the correct financial account".
Clearing every plan column is refused with "At least one analytic account must be set".

---

## Workflow 24: Mass-edit the distribution of several journal items

**Actor.** An accountant.

**Steps.**

1. The actor opens the journal items list, shows the analytic distribution column if it is hidden,
   selects several rows and clicks the analytic distribution cell of one of them.
2. The editor opens in multiple-record mode: it starts with an empty grid instead of the current
   values, and it shows one check box per plan indicating which plans the actor intends to replace.
   Only the ticked plans are put into the update marker.
3. The actor fills the grid and confirms.
4. The written document, carrying the update marker, is merged plan by plan into each selected
   journal item's stored document ([calculations.md](calculations.md) section 7), and each posted
   journal item's analytic lines are regenerated as in workflow 19.

**Postconditions.** Each selected journal item keeps its attribution on the plans that were not
ticked and receives the new attribution on the plans that were ticked, with the two sides multiplied
and balanced.

**Worked example.** Two posted journal items carry `{ "1" : 100 }` and `{ "2" : 100 }` on the plan
*Departments*. The actor ticks only the plan *Project* and enters seventy percent for account 11 and
thirty percent for account 12. The two journal items end with `{ "1,11" : 70 , "1,12" : 30 }` and
`{ "2,11" : 70 , "2,12" : 30 }` respectively, and four analytic lines replace the previous two.

---

## Workflow 25: Split analytic lines by writing a distribution

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor opens the analytic items screen, shows the analytic distribution column — hidden by
   default in that list — and selects one or several analytic lines.
2. The editor shows the line's own attribution as one row at one hundred percent. In multiple-record
   mode the actor ticks the plans to replace.
3. On confirmation the split algorithm of [calculations.md](calculations.md) section 12 runs for
   each selected line: the first resulting set of values is written onto the line itself and the
   other sets create copies.
4. A notification reading the number of created lines followed by "analytic lines created" is shown
   to the actor when at least one line was created.

**Postconditions.** The selected lines are replaced by a set of lines whose amounts add up to the
same total. When the merged document is empty, nothing at all happens and no error is raised.

---

## Workflow 26: Enter a manual analytic line

**Actor.** A member of the analytic accounting permission group.

**Steps.**

1. The actor opens the analytic items screen and creates a line, or opens an analytic account and
   presses the *Gross Margin* button, which opens the analytic items of that account with the
   account pre-filled as the default of the magic column, which in turn fills the column of that
   account's own root plan.
2. The actor fills the description (required), one or more plan columns, the date (required,
   defaulting to today in the reader's time zone), the amount, the quantity, the unit and, in a
   multi-company database, the company.
3. When the actor picks a product on a line that has the product field, the product valuation helper
   runs: it reads the product's standard price converted into the chosen unit of measure, or into
   the product's own unit when none is chosen; it sets the amount to the negated rounded value of
   the unit price times the quantity, rounded with the line's currency, or to two decimal places
   when the line has no currency; it sets the financial account to the product's expense account
   resolved through the product's category for the line's company; and it sets the unit to the
   product's unit when it was empty.
4. On save the constraint requires at least one plan column to be set.

**Postconditions.** A standalone analytic line exists, with no journal item. It contributes to the
account's debit, credit and balance exactly like a generated line.

**Worked example.** A product whose standard price is 12.50 per unit is entered with a quantity of 4
on a manual analytic line. The helper sets the amount to −50.00, the unit to the product's unit and
the financial account to the expense account of the product's category. The analytic account's debit
increases by 50.00 and its balance decreases by 50.00.

---

## Workflow 27: Validate mandatory plans when confirming a source document

**Actor.** A salesperson confirming a sales order, a buyer confirming a purchase order, an approver
approving an expense, a manufacturing manager confirming a manufacturing order, a warehouse operator
validating a transfer, an employee recording a timesheet.

**Steps.**

1. The actor presses the confirmation button. Every such button switches the validation flag on.
2. The document asks each of its lines to validate its distribution with the situation of that
   document:
   - a sales order line passes its product, the business domain `sale_order` and its company, and
     only the lines of an order still in a draft or sent state are checked;
   - a purchase order line passes its product, the business domain `purchase_order` and its company,
     and section and note lines are skipped;
   - an expense passes its financial account, its product, the business domain `expense` and its
     company, and only the expenses moving from draft or submitted to approved are checked;
   - a manufacturing order passes the business domain `manufacturing_order`;
   - a transfer passes the business domain `stock_picking`;
   - a timesheet passes the business domain `timesheet` and additionally requires an account on
     every mandatory plan **other than the base plan**, because a timesheet takes its base plan
     account from its project.
3. A failure raises "One or more lines require a 100% analytic distribution." and the confirmation
   is abandoned.

**Postconditions.** The document is confirmed only when every mandatory plan of every line totals
exactly one hundred percent. The owning domains are [../sales/](../sales/README.md),
[../purchasing/](../purchasing/README.md), [../expenses/](../expenses/README.md),
[../manufacturing/](../manufacturing/README.md),
[../inventory-operations/](../inventory-operations/README.md) and
[../timesheets/](../timesheets/README.md).

---

## Workflow 28: Redistribute the analytic lines of a valuation document

**Actor.** The system, on behalf of an inventory or manufacturing document.

**Trigger.** A valuation document recomputes its cost: a stock move is valued, a work order is
costed, a manufacturing order is costed.

**Steps.**

1. The document supplies the target distribution, the total amount, the total quantity, the analytic
   lines already attached to it, itself as the template for the values of new lines, and a flag
   saying whether the amounts are added to the existing ones.
2. The redistribution of [calculations.md](calculations.md) section 16 runs: lines whose combination
   of accounts is still present in the new distribution are rewritten with the new amount and the
   new quantity; lines whose combination has disappeared are deleted; lines whose new amount rounds
   to zero are deleted; remaining combinations produce the values of new lines, which the document
   then creates.
3. Unlike the regeneration of workflow 19, this operation **preserves the identifiers** of the
   surviving lines.

**Postconditions.** The analytic lines attached to the valuation document match the new
distribution. The triggering events and the amounts themselves belong to
[../inventory-valuation-and-costing/](../inventory-valuation-and-costing/README.md) and
[../manufacturing/](../manufacturing/README.md).

---

## Workflow 29: Consult analytic reporting

**Actor.** A member of the analytic accounting permission group, or an accounting read-only user.

**Steps.**

1. The actor opens the analytic items screen. The list shows the date, the description, the base
   plan column, one column per other root plan inserted automatically, the optional analytic
   distribution splitting column, the quantity with a column total, the unit, the partner, the
   company and the amount with a column total.
2. The actor groups by any plan column, by any sub-plan grouping column, by date, by partner, by
   product, by financial account or by profitability. The grouping menu lists one entry per root
   plan and one entry per existing sub-plan depth; when a root plan has several depths, the depths
   are offered as options of a single entry.
3. The actor switches to the graph or the pivot presentation, which measure the quantity and the
   amount and group by the base plan column, the pivot additionally grouping columns by month.
4. From an analytic account the actor presses *Gross Margin*, which opens the analytic items of that
   account grouped by date, with the account pre-filled for creation.
5. From an analytic account the actor may also open the customer invoices or the vendor bills whose
   journal items mention that account.

**Postconditions.** None; reporting is read-only.

---

## Workflow 30: Consult the analytic sections of a project's profitability

**Actor.** A project user; the actions behind the sections additionally require at least the
read-only accounting group.

**Steps.**

1. The actor opens the project's profitability panel.
2. The project accounting capability computes three sections from records this domain owns: the
   vendor bills that mention the project's analytic account but come from no purchase order, the
   other revenues and the other costs, both taken from the analytic lines of that account that have
   no journal item and whose category is neither a manufacturing order nor an inventory transfer.
   The arithmetic is in [calculations.md](calculations.md) section 17.
3. Pressing a section opens the records behind it: the vendor bills section opens the bills, the
   two analytic sections open the analytic items with the graph and pivot presentations of the
   project accounting capability, grouped by date.
4. From the project's task list or from its dashboard, the actor may open the embedded *Analytic
   Items* entry, which lists the analytic items of the project's analytic account; creation is
   offered only from the embedded entry.

**Postconditions.** None; the panel is read-only.

---

## Reconciliation notes

1. **Two state tables removed.** The draft that carried the workflows closed with two state tables —
   analytic lines against the state of the journal entry, and the position of a plan against the
   columns it owns. Both are now in [state-machines.md](state-machines.md) sections 4 and 2, with
   their guards and refusal messages; no row was lost.
2. **Workflows only one draft had.** Reversal (workflow 21) and the redistribution of a valuation
   document (workflow 28) came from one draft each; the designation of another base plan (workflow
   8) and the project profitability consultation (workflow 30) are new, written from the source,
   because both belong to this domain's scope.
3. **The editor's proposed percentage.** Only one draft carried the rule for the percentage proposed
   on a new row; it is kept verbatim in workflow 16 step 4, and restated in
   [interfaces.md](interfaces.md).
4. **The label of the analytic accounting setting.** Both drafts paraphrased it. Workflow 1 now
   quotes the emitted text, "Track costs & revenues by project, department, etc", as
   [configuration.md](configuration.md) section 1.1 and [business-rules.md](business-rules.md)
   `AA-110` do.
