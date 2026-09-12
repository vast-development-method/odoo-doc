# Analytic Accounting — Acceptance Criteria

Numbered Given, When and Then scenarios with concrete records, concrete inputs and exact expected
results. An implementation of this domain is correct when every scenario below produces the stated
outcome. Every rule of [business-rules.md](business-rules.md), every transition of
[state-machines.md](state-machines.md), every formula of [calculations.md](calculations.md) and
every procedure of [workflows.md](workflows.md) is exercised by at least one scenario.

Scenarios are numbered `AA-AC-nnn`. The mapping to the numbering of the earlier draft is in section
20.

## The standing fixture

Unless a scenario says otherwise:

- One company, **Main Company**, whose currency has two decimal places and a rounding step of 0.01.
- The base plan is the shipped plan **Project**, default applicability `optional`, designated by
  the system parameter `analytic.project_plan` (the base-plan parameter). Its stored column on
  Analytic Line is `account_id` (project account).
- A root plan **Plan 1**, default applicability `unavailable`, with one child plan **Plan Child**.
- A root plan **Plan 2**, with no explicit default applicability and therefore the installed default
  `optional`.
- Four analytic accounts, all shared by every company: **Account 1** in *Plan 1*; **Account 2** in
  *Plan Child*, whose root plan is therefore *Plan 1*; **Account 3** and **Account 4** in *Plan 2*.
- The decimal precision `Percentage Analytic` (the percentage precision) is two.
- The acting reader holds the analytic accounting permission group and, where a scenario needs it,
  the read-only accounting group.
- No lock date is set and the fiscal year ends on the thirty-first of December.

Notation used in the expected results:

| Written | Meaning |
|---|---|
| column(*P*) | The stored analytic account column of the root plan *P* on an analytic line: `account_id` for the base plan, `x_plan<plan identifier>_id` for any other root plan. |
| `{ "3" : 100 }` | A distribution document; the keys are analytic account identifiers, comma-separated inside one key. |
| `__update__` | The reserved partial-update marker of a distribution being written. |

Contents:

1. [Plans and dynamic columns](#1-plans-and-dynamic-columns)
2. [The base plan](#2-the-base-plan)
3. [Applicability](#3-applicability)
4. [Relevant plans](#4-relevant-plans)
5. [Distribution models](#5-distribution-models)
6. [Normalising and merging a distribution](#6-normalising-and-merging-a-distribution)
7. [Splitting analytic lines](#7-splitting-analytic-lines)
8. [Validating mandatory plans](#8-validating-mandatory-plans)
9. [Generating analytic lines on posting](#9-generating-analytic-lines-on-posting)
10. [Rounding](#10-rounding)
11. [Rebuilding a distribution from analytic lines](#11-rebuilding-a-distribution-from-analytic-lines)
12. [Derived journal items](#12-derived-journal-items)
13. [Reset, reversal and deletion](#13-reset-reversal-and-deletion)
14. [Analytic accounts](#14-analytic-accounts)
15. [Analytic lines](#15-analytic-lines)
16. [Searching and grouping by distribution](#16-searching-and-grouping-by-distribution)
17. [Redistribution and project profitability](#17-redistribution-and-project-profitability)
18. [Screens, the editor and reporting](#18-screens-the-editor-and-reporting)
19. [Access, companies and currencies](#19-access-companies-and-currencies)
20. [Mapping of the former scenario numbers](#20-mapping-of-the-former-scenario-numbers)
21. [Reconciliation notes](#21-reconciliation-notes)

---

## 1. Plans and dynamic columns

**AA-AC-001. A new root plan creates a stored column.**
Given no plan named *Test Plan* exists.
When a plan *Test Plan* is created with no parent.
Then a stored analytic account column named `x_plan` followed by the plan's internal identifier and
`_id` exists on Analytic Line, may be read and written, is copied when a line is duplicated, carries
a restricting deletion rule, and is covered by a balanced-tree index restricted to the rows where it
is not empty.

**AA-AC-002. Deleting a root plan removes its column.**
Given the plan *Test Plan* of `AA-AC-001` exists and no stored view definition names its column.
When the plan is deleted.
Then the column no longer exists on Analytic Line and reading it fails.

**AA-AC-003. A sub-plan owns no column but contributes a grouping column.**
Given a root plan *Parent Plan* exists, with the column `x_plan<identifier of Parent Plan>_id`.
When a plan *Test Plan* is created with the parent *Parent Plan*.
Then no stored column exists for *Test Plan* on Analytic Line, and a derived, read-only, unstored
grouping column named `x_plan<identifier of Parent Plan>_id_1` exists, labelled "Parent Plan (1)",
whose value is read as "the account held in the parent plan's column, then that account's plan".

**AA-AC-004. A grouping column survives while another plan occupies its depth.**
Given *Parent Plan* has two children, *Test Plan* and *Other Plan*, both at depth one.
When *Other Plan* is deleted.
Then the grouping column of depth one still exists.
When *Test Plan* is then deleted.
Then the grouping column of depth one no longer exists.

**AA-AC-005. A third level creates a second grouping column.**
Given *Parent Plan* has a child *Sub Plan* and a grandchild *Sub Sub Plan*.
Then two grouping columns exist on Analytic Line: `x_plan<identifier of Parent Plan>_id_1`, labelled
"Parent Plan (1)", and `x_plan<identifier of Parent Plan>_id_2`, labelled "Parent Plan (2)", the
second reading "the account in the parent plan's column, then its plan, then that plan's parent".

**AA-AC-006. A sub-plan of the base plan is prefixed.**
Given a plan *Base Child* is created with the base plan as its parent.
Then the grouping column created on Analytic Line is named `x_account_id_1`, because the computed
name would otherwise begin with `account_id`.

**AA-AC-007. Renaming a plan relabels its column and never renames it.**
Given a root plan *Test Plan* whose column is labelled "Test Plan".
When the plan's name is changed to *New name*.
Then the column keeps exactly the same name, its label becomes "New name", the stored values are
untouched, and every stored view definition naming that column still resolves.

**AA-AC-008. Promoting a sub-plan creates its column and removes the grouping column.**
Given a root plan *Parent Plan* has one child *Test Plan*, so *Test Plan* has no column and a
grouping column of depth one exists.
When the parent of *Test Plan* is cleared.
Then *Test Plan* owns a stored column on Analytic Line, with its partial index, and no grouping
column of depth one exists any more.

**AA-AC-009. Demoting a root plan removes its column and creates a grouping column.**
Given two root plans *Parent Plan* and *Test Plan*, so *Test Plan* owns a column and no grouping
column of depth one exists.
When the parent of *Test Plan* is set to *Parent Plan*.
Then *Test Plan* owns no column any more and a grouping column of depth one exists.

**AA-AC-010. A plan still named by a stored view definition cannot be deleted.**
Given a root plan *Test Plan* and a stored search view definition of Analytic Line containing a
field node naming its column.
When the plan is deleted.
Then the deletion is refused with a message beginning "Cannot rename/delete fields that are still
present in views:", the plan still exists, its column still exists, and no other plan or column has
been removed.

**AA-AC-011. Changing an account's plan moves the value between columns.**
Given an analytic line carrying *Account 1* in column(*Plan 1*) and nothing in column(*Plan 2*).
When the plan of *Account 1* is changed to *Plan 2*.
Then the analytic line carries nothing in column(*Plan 1*) and *Account 1* in column(*Plan 2*), and
the account's stored root plan is *Plan 2*.

**AA-AC-012. Changing an account's plan is refused when it would overwrite a value.**
Given an analytic line carrying *Account 1* in column(*Plan 1*) and *Account 3* in column(*Plan 2*).
When the plan of *Account 1* is changed to *Plan 2*.
Then the change is refused with "Whoa there! Making this change would wipe out your current data.
Let's avoid that, shall we?", accompanied by a button labelled "See them" that opens the offending
analytic lines in a list, and nothing at all is written.

**AA-AC-013. Changing an account's plan is allowed when the target already holds the same account.**
Given an analytic line carrying *Account 1* in column(*Plan 1*) and *Account 1* in column(*Plan 2*),
a state an import can produce.
When the plan of *Account 1* is changed to *Plan 2*.
Then the line carries nothing in column(*Plan 1*) and *Account 1* in column(*Plan 2*): the conflict
check tolerates a target that already holds one of the accounts being moved.

**AA-AC-014. A move between two sub-plans of one hierarchy migrates nothing.**
Given *Account 2* belongs to *Plan Child*, whose root is *Plan 1*, and an analytic line carries
*Account 2* in column(*Plan 1*).
When the plan of *Account 2* is changed to *Plan 1*.
Then the two column names are identical, no migration statement runs, and the line still carries
*Account 2* in column(*Plan 1*).

**AA-AC-015. Demoting a plan migrates the analytic line values, and promoting it moves them back.**
Given an analytic line carrying *Account 1* in column(*Plan 1*).
When *Plan 1* is given the parent *Plan 2*.
Then the line carries *Account 1* in column(*Plan 2*), column(*Plan 1*) no longer exists, and the
migration happened **before** the parent was written.
When the parent of *Plan 1* is cleared again.
Then column(*Plan 1*) exists again, the line carries *Account 1* in it and nothing in column(*Plan
2*), and the migration happened **after** the parent was written.

**AA-AC-016. Demoting a plan is refused when two accounts would collide.**
Given an analytic line carrying *Account 1* in column(*Plan 1*) and *Account 3* in column(*Plan 2*).
When *Plan 1* is given the parent *Plan 2*.
Then the change is refused with "Whoa there! Making this change would wipe out your current data.
Let's avoid that, shall we?" and nothing is written; column(*Plan 1*) still exists.

**AA-AC-017. Demotion migrates the accounts of intermediate levels too.**
Given a plan *Mid level* is a child of *Plan 1*, *Account 1* belongs to *Mid level*, and an analytic
line carries *Account 1* in column(*Plan 1*).
When *Plan 1* is given the parent *Plan 2*.
Then the line carries *Account 1* in column(*Plan 2*), because the migration selects every account
whose plan is the plan being moved **or any descendant of it**.
When the parent of *Plan 1* is cleared.
Then the line carries *Account 1* in column(*Plan 1*) and nothing in column(*Plan 2*).

**AA-AC-018. Deleting a plan deletes its descendants.**
Given *Plan 1* has the child *Plan Child*, which has the child *Plan Grandchild*, and no analytic
line and no stored view definition refers to any of their columns.
When *Plan 1* is deleted.
Then all three plans are gone, column(*Plan 1*) is gone with the values it held, and the grouping
columns of depths one and two are gone.

**AA-AC-019. The account count of a plan includes its whole sub-tree.**
Given a plan *Parent Plan* with a child *Sub Plan* and a grandchild *Sub Sub Plan*, and exactly one
analytic account in each of the three.
Then the all-accounts count is 3 for *Parent Plan*, 2 for *Sub Plan* and 1 for *Sub Sub Plan*, while
the direct account count is 1 for each.

**AA-AC-020. A plan is offered when an account exists only at the third level.**
Given the plans of `AA-AC-019`, each with one account.
When the relevant plans are asked for with no argument.
Then *Parent Plan* is one of them and *Sub Plan* and *Sub Sub Plan* are not, because only root plans
are offered.

**AA-AC-021. A plan whose sub-tree has no account is never offered.**
Given a root plan *Campaigns* with no account anywhere in its sub-tree.
When the relevant plans are asked for with no argument.
Then *Campaigns* is absent, whatever its default applicability.

**AA-AC-022. A reader holding only the analytic group may create a plan.**
Given a user holding only the analytic accounting permission group.
When that user creates a plan named *test plan*.
Then the plan exists and its recorded author is that user.

**AA-AC-023. Creating or deleting a plan makes the clients reload.**
Given a client has the analytic item list open.
When any create, write or delete on Analytic Plan succeeds.
Then the client reloads its definition of the plan-bearing entities, because their set of fields has
changed.

---

## 2. The base plan

**AA-AC-024. The base plan cannot be given a parent.**
Given the base plan is *Project*.
When a reader chooses a parent on the form of *Project*.
Then the form refuses immediately, before anything is written, with "You cannot add a parent to the
base plan 'Project'".

**AA-AC-025. The base-plan parameter refuses an unusable value.**
When the system parameter `analytic.project_plan` is written with the text `not a number`, or with
the identifier of a sub-plan, or with the identifier of a plan that does not exist.
Then the write is refused with "The value for the key must be the ID to a valid analytic plan that
is not a subplan", the placeholder being the parameter key, and the parameter keeps its former
value.

**AA-AC-026. Changing the base-plan parameter re-synchronises the columns and migrates nothing.**
Given the base plan is *Project* and *Plan 2* is another root plan owning the column column(*Plan
2*), and one analytic line carries *Account 3* in column(*Plan 2*) and an account of *Project* in
`account_id`.
When `analytic.project_plan` is set to the identifier of *Plan 2*.
Then *Project* receives a generated column named `x_plan<identifier of Project>_id`, **empty**;
column(*Plan 2*) is deleted together with the value it held, so *Account 3* is no longer recorded on
that line; and `account_id` keeps its value, which is from then on read as an account of *Plan 2*.

**AA-AC-027. Without a base plan nothing works.**
Given the parameter `analytic.project_plan` names no existing plan.
When any operation that needs the list of root plans runs — asking for the relevant plans, opening a
distribution editor, validating a distribution, posting an entry with a distribution.
Then it fails with "A 'Project' plan needs to exist and its id needs to be set as
`analytic.project_plan` in the system variables".

---

## 3. Applicability

**AA-AC-028. Only plans with accounts and a non-unavailable applicability are offered.**
Given the fixture, in which *Plan 1* is `unavailable` and *Plan 2* is `optional`.
When the relevant plans are asked for with no argument.
Then *Plan 2* is offered and *Plan 1* is not.
When *Plan 1* is changed to the default applicability `mandatory`.
Then both *Plan 1* and *Plan 2* are offered.

**AA-AC-029. A rule for a business domain overrides the default.**
Given *Plan 1* is `unavailable` by default and carries one rule with the business domain `general`
and the applicability `mandatory`.
When the relevant plans are asked for with the business domain `general`.
Then *Plan 1* is offered with the applicability `mandatory`, because the rule scores 0 + 1 = 1,
which is strictly greater than the baseline of 0.5.

**AA-AC-030. A rule overrides the default in both directions.**
Given *Plan 1* is `mandatory` by default and carries one rule with the business domain `general` and
the applicability `unavailable`.
When the relevant plans are asked for with the business domain `general`.
Then *Plan 1* is not offered.
When they are asked for with the business domain `purchase_order`.
Then *Plan 1* is offered with the applicability `mandatory`, because the rule is eliminated with
minus one and the default stands.

**AA-AC-031. A forced applicability shows every plan and evaluates no rule.**
Given the fixture with *Plan 1* `unavailable`.
When the relevant plans are asked for with the forced applicability `optional`.
Then both *Plan 1* and *Plan 2* are offered, both with the applicability `optional`, and no rule is
scored.

**AA-AC-032. A business-domain rule alone wins.**
Given a plan with the default applicability `optional` and one rule with the business domain
`invoice`, the applicability `mandatory`, no company, no prefix and no category.
When the applicability is computed with the business domain `invoice`.
Then it is `mandatory` (score 1).
When it is computed with the business domain `bill`.
Then it is `optional` (the rule is eliminated at minus one).

**AA-AC-033. A product category narrows a rule.**
Given a plan with the default applicability `optional` and one rule with the business domain
`invoice`, the product category *Services* and the applicability `mandatory`.
When the applicability is computed with the business domain `invoice` and a product of the category
*Services*.
Then it is `mandatory` (score 0 + 1 + 1 = 2).
When it is computed with the business domain `invoice` and no product.
Then it is `optional`, because the category criterion eliminates the rule.

**AA-AC-034. A child category does not match a parent category.**
Given the rule of `AA-AC-033` and a product whose category is *Consulting*, a child of *Services*.
When the applicability is computed with the business domain `invoice` and that product.
Then it is `optional`: the comparison is on the exact category.

**AA-AC-035. Two narrowing criteria are combined with a logical "and".**
Given a plan with the default applicability `optional` and one rule with the business domain
`invoice`, the product category *Services*, the account prefix `705` and the applicability
`mandatory`; and two financial accounts `705100` and `706100`.
When the applicability is computed with the business domain `invoice`, a product of the category
*Services* and the account `705100`.
Then it is `mandatory` (score 0 + 1 + 1 + 1 = 3).
When it is computed with the same product and the account `706100`.
Then it is `optional`, even though the category matched.

**AA-AC-036. A rule may list several account prefixes.**
Given *Plan 2* carries a rule with the business domain `invoice`, the company *Main Company*, the
account prefix `40, 41` and the applicability `mandatory`; and two financial accounts `400300` and
`410300`.
And a customer invoice of *Main Company* with two lines, one booked on `400300` and one on `410300`,
neither carrying a distribution.
When the invoice is posted with the validation flag on.
Then the posting is refused with "One or more lines require a 100% analytic distribution."
When the line booked on `400300` receives `{ "<Account 3>" : 100 }` and the posting is retried.
Then it is still refused, because the second line is still incomplete.
When the line booked on `410300` also receives `{ "<Account 3>" : 100 }` and the posting is retried.
Then the invoice is posted.

**AA-AC-037. A company rule wins for its own company and is excluded for another.**
Given a plan with the default applicability `optional`, a rule R1 with the business domain
`invoice`, no company and the applicability `mandatory`, and a rule R2 with the business domain
`invoice`, the company *Main Company* and the applicability `unavailable`, created after R1.
When the applicability is computed with the business domain `invoice` and the company *Main
Company*.
Then it is `unavailable` (1.5 against 1).
When it is computed with the business domain `invoice` and no company.
Then it is `mandatory`: both score 1 and the tie keeps the rule with the lower internal identifier,
which is R1.
When it is computed with the business domain `invoice` and the company *Other Company*.
Then it is `mandatory`, because R2 is excluded from the evaluation entirely.

**AA-AC-038. A company alone never overrides the default.**
Given the two rules of `AA-AC-037`.
When the applicability is computed with the company *Main Company* and **no** business domain.
Then it is `optional`: the scores are 0 and 0.5, and 0.5 is not strictly greater than the baseline
of 0.5.

**AA-AC-039. A product category alone overrides the default without a business domain.**
Given a plan with the default applicability `optional` and one rule with the business domain
`invoice`, the product category *Services* and the applicability `mandatory`.
When the applicability is computed with a product of the category *Services* and **no** business
domain.
Then it is `mandatory`: the base layer stops at zero and the category criterion adds one.
When it is computed with no argument at all.
Then it is `optional`, because the rule is eliminated by the category criterion.

**AA-AC-040. The company bonus still decides when no business domain is supplied.**
Given a plan with the default applicability `optional`, a rule R1 with the business domain
`invoice`, the product category *Services*, no company and the applicability `mandatory`, and a rule
R2 with the same business domain and category, the company *Main Company* and the applicability
`unavailable`.
When the applicability is computed with a product of the category *Services* and the company *Main
Company*, with no business domain.
Then it is `unavailable` (1.5 against 1).
When it is computed with the same product and no company.
Then it is `mandatory` (1 against 1, the tie keeping R1).

**AA-AC-041. Prefix and category without a business domain.**
Given a plan with the default applicability `optional` and one rule with the business domain
`invoice`, the product category *Services*, the account prefix `705` and the applicability
`mandatory`.
When the applicability is computed with a product of the category *Services*, the account `705100`
and no business domain.
Then it is `mandatory` (score 0 + 1 + 1 = 2).
When it is computed with the account `706100`.
Then it is `optional`.

**AA-AC-042. A rule of another company does not make a plan mandatory.**
Given a plan with the default applicability `optional`, one account in that plan, and one rule with
the business domain `general`, the applicability `mandatory` and the company *Company Two*; and a
record whose distribution is empty.
When its distribution is validated with the business domain `general`, the company *Main Company*
and the validation flag on.
Then the validation succeeds.
When the rule's company is cleared and the validation is repeated with the company *Main Company*,
and again with no company at all.
Then both are refused with "One or more lines require a 100% analytic distribution."

**AA-AC-043. The complete score table of one situation.**
Given the root plan *Departments* with the default applicability `optional`, and the situation: the
company *Northwind*, the business domain `invoice`, the product *Desk* of the category *Office
Furniture*, the financial account `701200`.
Then the scores are: a rule on `invoice` alone, 1; a rule on `invoice` and the company, 1.5; a rule
on `bill` and the company, minus one; a rule on `invoice`, the company and the prefix `70, 71`, 2.5;
a rule on `invoice`, the company and the prefix `60`, minus one; a rule on `invoice`, the company,
the prefix `70` and the category *Office Furniture*, 3.5; a rule on `invoice`, the company, the
prefix `70` and the category *Raw Materials*, minus one; and a rule naming only the company, 0.5.
And the winning rule is the one scoring 3.5.

**AA-AC-044. Every change to the applicability configuration drops the cache.**
Given the relevant plans have been asked for once inside a transaction, with a given set of
arguments, and the answer has been cached.
When a plan is created, a plan is deleted, a plan's default applicability is written, or an
applicability rule is created, written or deleted.
Then the next question with the same arguments is recomputed.
When any other field of a plan is written — its name, its colour index, its sequence, its
description.
Then the cached answer is kept.

---

## 4. Relevant plans

**AA-AC-045. The ordinary answer.**
Given the plans *Project* (base plan, sequence 10, `optional`, 12 accounts), *Departments*
(sequence 20, `optional`, 7 accounts), *Internal* (sequence 30, `unavailable`, 2 accounts) and
*Campaigns* (sequence 40, `optional`, no account).
When the relevant plans are asked for, for an invoice line with no account yet.
Then the answer is *Project* then *Departments*, each with its identifier, its name, its colour
index, its applicability, its all-accounts count and its stored column name; *Internal* is excluded
by its applicability and *Campaigns* by its empty sub-tree.

**AA-AC-046. A plan already used is still offered even when it became unavailable.**
Given *Plan 1* is `unavailable` and a document line's distribution names *Account 1* of *Plan 1*.
When the relevant plans are asked for with that account in the list of accounts already present.
Then *Plan 1* is in the answer with the applicability `optional`, so that the existing percentage
stays visible and editable, and it is placed after the kept plans, before the sort by sequence.

**AA-AC-047. The order of the answer.**
Given the plans of `AA-AC-045` with *Departments* given the sequence 5.
When the relevant plans are asked for.
Then *Departments* comes before *Project*; and when two plans share a sequence, the base plan comes
first, then the other kept plans in identifier order, then the forced plans, because the sort is
stable.

---

## 5. Distribution models

**AA-AC-048. No model matches when no criterion is supplied.**
Given a model M1 with the partner *Partner A*, the company *Main Company* and the distribution
`{ "<Account 3>" : 100 }`.
When a distribution is asked for with no criterion.
Then the answer is the empty document.

**AA-AC-049. A model matches on its partner.**
Given M1 as above and a model M2 with the partner *Partner B*, the company *Main Company* and the
distribution `{ "<Account 2>" : 100 }`.
When a distribution is asked for with the partner *Partner A* and the company *Main Company*.
Then the answer is `{ "<Account 3>" : 100 }`.
When it is asked for with the partner *Partner B* and the company *Main Company*.
Then the answer is `{ "<Account 2>" : 100 }`.

**AA-AC-050. Two models filling different plans are combined into one compound key.**
Given M1 (partner *Partner A*, company *Main Company*, `{ "<Account 3>" : 100 }`, *Account 3* in
*Plan 2*) and a later-created model M3 (partner *Partner A*, company *Main Company*,
`{ "<Account 1>" : 100 }`, *Account 1* in *Plan 1*), both with sequence 10.
When a distribution is asked for with the partner *Partner A* and the company *Main Company*.
Then the answer is `{ "<Account 1>,<Account 3>" : 100 }`: M3 is evaluated first because the
identifier tiebreak is descending, it claims *Plan 1*, and the merge of its result with M1 gives one
compound key at one hundred percent.

**AA-AC-051. A model that would fill an already filled plan is skipped entirely.**
Given three root plans A, B and C, the accounts A1 and A2 in A, B1 and B3 in B, C2 in C, and three
models all with the account prefix `123`: M1 with sequence 10 and `{ "A1,B1" : 100 }`, M2 with
sequence 20 and `{ "A2,C2" : 100 }`, M3 with sequence 30 and `{ "B3" : 100 }`.
When a distribution is asked for with the account prefix `123456` and the company *Main Company*.
Then the answer is `{ "A1,B1" : 100 }`: M2 touches plan A, already filled, and M3 touches plan B,
already filled, so both are skipped, M2 including its free plan C.
When the sequences become M2 = 1, M1 = 2, M3 = 3 and the question is repeated.
Then the answer is `{ "A2,C2,B3" : 100 }`.
When the sequences become M3 = 1, M1 = 2, M2 = 3 and the question is repeated.
Then the answer is `{ "B3,A2,C2" : 100 }`.

**AA-AC-052. Two accounts of one plan in a model are weighted against a one-plan model.**
Given a model M4 with sequence 1, the partner *Partner A*, the partner category *Tag X* and the
distribution `{ "<Account 1>" : 100 , "<Account 2>" : 100 }`, both accounts having the root plan
*Plan 1*; the models M3 and M1 of `AA-AC-050` with sequence 10; and *Partner A* carrying *Tag X*.
When a distribution is asked for with the partner *Partner A*, the company *Main Company* and the
partner category *Tag X*.
Then the answer is `{ "<Account 1>,<Account 3>" : 50 , "<Account 2>,<Account 3>" : 50 ,
"<Account 1>" : 50 , "<Account 2>" : 50 }`: M4 claims *Plan 1* with a total of two hundred, M3 is
skipped, and merging with M1's one hundred on *Plan 2* halves the cross product and leaves the
surplus of *Plan 1* as two entries of fifty.

**AA-AC-053. Models are ordered by sequence then by descending identifier.**
Given two models with sequence 10: M_a with the product *Desk*, giving `{ "<Account 3>" : 100 }`,
created first; and M_b with the partner *Partner A* and the product *Desk*, giving
`{ "<Account 4>" : 100 }`, created second; both accounts in *Plan 2*.
When a customer invoice is created for *Partner A* with the product *Desk*.
Then the line's distribution is `{ "<Account 4>" : 100 }`.
When one is created for *Partner B* with the product *Desk*.
Then the line's distribution is `{ "<Account 3>" : 100 }`.
When one is created for *Partner A* with the product *Chair*.
Then the line has no distribution.
When one is created for *Partner B* with the product *Chair*.
Then the line has no distribution.

**AA-AC-054. A more specific model wins only through its sequence.**
Given two root plans *Alpha* and *Beta*, the accounts A1 and A2 in *Alpha* and B1 in *Beta*, a model
"specific" with sequence 1, the partner *Acme*, the product category *Office Furniture* and
`{ "A1" : 100 }`, and a model "general" with sequence 10, the partner *Acme*, no category and
`{ "A2" : 100 }`.
When a bill line is entered for *Acme* with a product of the category *Office Furniture*.
Then the proposal is `{ "A1" : 100 }`, the general model being skipped because *Alpha* is taken.
When the two sequences are swapped.
Then the proposal is `{ "A2" : 100 }`.
When a third model with sequence 20, the partner *Acme* and `{ "B1" : 100 }` is added to the first
configuration.
Then the proposal is `{ "A1,B1" : 100 }`.

**AA-AC-055. A model applies and reapplies when the partner changes, and never erases with nothing.**
Given two models without a company: one for *Partner A* giving `{ "<Account 3>" : 100 }` and one for
*Partner B* giving `{ "<Account 4>" : 100 }`.
When a customer invoice is created with no partner and one line with the product *Desk*.
Then the line has no distribution.
When the partner is set to *Partner A*.
Then the line's distribution is `{ "<Account 3>" : 100 }`.
When the partner is set to *Partner B*.
Then it becomes `{ "<Account 4>" : 100 }`.
When the partner is set to one for which no model exists.
Then it stays `{ "<Account 4>" : 100 }`.
When the partner is cleared.
Then it stays `{ "<Account 4>" : 100 }`.

**AA-AC-056. A value typed by the reader survives the automatic proposal.**
Given the two models of `AA-AC-055` and an invoice whose partner is *Partner A*, whose line
therefore carries `{ "<Account 3>" : 100 }`.
When the reader types `{ "<Account 4>" : 100 }` on the line in the form and saves.
Then the stored distribution is `{ "<Account 4>" : 100 }`.

**AA-AC-057. A model may list several account prefixes, separated by a comma or a semicolon.**
Given a model with the account prefix `61;62` giving `{ "<Account 3>" : 100 }` and a model with the
account prefix `63, 64` giving `{ "<Account 4>" : 100 }`, both without a company.
When an invoice line's financial account is set to `611000`.
Then the line's distribution is `{ "<Account 3>" : 100 }`.
When it is set to `630000`.
Then it is `{ "<Account 4>" : 100 }`.
When it is set to `620000`.
Then it is `{ "<Account 3>" : 100 }`.
When it is set to `641000`.
Then it is `{ "<Account 4>" : 100 }`.
When it is set to `700000`.
Then the line keeps whatever it had.

**AA-AC-058. A model naming a company-specific account must belong to that company.**
Given an analytic account whose company is *Main Company*.
When a distribution model with no company, or with the company *Company Two*, is created with a
distribution naming that account.
Then the creation is refused with "You defined a distribution with analytic account(s) belonging to
a specific company but a model shared between companies or with a different company".

**AA-AC-059. Deleting a condition record deletes the model.**
Given a model naming the partner *Partner A*.
When *Partner A* is deleted.
Then the model is deleted as well; the same holds for the partner category, the product, the product
category and the company of a model.

**AA-AC-060. A model whose distribution names only deleted accounts fills nothing.**
Given a model whose distribution names one account, and that account is deleted.
When a distribution is asked for with criteria that match the model.
Then the model contributes nothing, claims no plan, and the next model in order may fill the plans
it would have filled.

---

## 6. Normalising and merging a distribution

**AA-AC-061. Percentages are rounded on write.**
When the distribution `{ "7" : 33.333333 , "8" : 66.666667 }` is written on a journal item.
Then the stored distribution is `{ "7" : 33.33 , "8" : 66.67 }`, whose total is exactly one hundred.

**AA-AC-062. Rounding breaks a tie away from zero.**
When the values 33.335 and 0.004 are written as percentages.
Then 33.34 and 0.00 are stored.

**AA-AC-063. A write without the marker replaces everything.**
Given a journal item carrying `{ "<Account 1>" : 40 , "<Account 2>" : 60 }`.
When an empty document is written.
Then the stored distribution is empty.

**AA-AC-064. Adding a plan to an existing distribution multiplies the two sides.**
Given a journal item carrying `{ "<Account 1>" : 40 , "<Account 2>" : 60 }`, both accounts having
the root plan *Plan 1*.
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 25 , "<Account 4>" : 75 }` is written.
Then the stored distribution is `{ "<Account 1>,<Account 3>" : 10 , "<Account 2>,<Account 3>" : 15 ,
"<Account 1>,<Account 4>" : 30 , "<Account 2>,<Account 4>" : 45 }` and *Plan 1* and *Plan 2* each
total one hundred.

**AA-AC-065. Two sides below one hundred percent keep their totals.**
Given a journal item carrying `{ "<Account 1>" : 20 , "<Account 2>" : 30 }`.
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 10 , "<Account 4>" : 40 }` is written.
Then the stored distribution is `{ "<Account 1>,<Account 3>" : 4 , "<Account 1>,<Account 4>" : 16 ,
"<Account 2>,<Account 3>" : 6 , "<Account 2>,<Account 4>" : 24 }`, each plan totalling fifty.

**AA-AC-066. Two sides above one hundred percent keep their totals.**
Given a journal item carrying `{ "<Account 1>" : 200 , "<Account 2>" : 300 }`.
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 100 , "<Account 4>" : 400 }` is written.
Then the stored distribution is `{ "<Account 1>,<Account 3>" : 40 , "<Account 2>,<Account 3>" : 60 ,
"<Account 1>,<Account 4>" : 160 , "<Account 2>,<Account 4>" : 240 }`, each plan totalling five
hundred.

**AA-AC-067. Updating both plans at once replaces both.**
Given a journal item carrying `{ "<Account 1>,<Account 3>" : 10 , "<Account 2>,<Account 3>" : 15 ,
"<Account 1>,<Account 4>" : 30 , "<Account 2>,<Account 4>" : 45 }`.
When `{ "__update__" : [ column(Plan 1) , column(Plan 2) ] , "<Account 1>,<Account 3>" : 45 ,
"<Account 2>,<Account 3>" : 30 , "<Account 1>,<Account 4>" : 15 , "<Account 2>,<Account 4>" : 10 }`
is written.
Then the stored distribution is exactly the written one, without the marker.

**AA-AC-068. Clearing one plan keeps the other with its aggregated totals.**
Given a journal item carrying `{ "<Account 1>,<Account 3>" : 45 , "<Account 2>,<Account 3>" : 30 ,
"<Account 1>,<Account 4>" : 15 , "<Account 2>,<Account 4>" : 10 }`.
When `{ "__update__" : [ column(Plan 1) ] }` is written, with no other entry.
Then the stored distribution is `{ "<Account 3>" : 75 , "<Account 4>" : 25 }`.

**AA-AC-069. An empty update list changes nothing.**
Given a journal item carrying `{ "<Account 1>" : 40 , "<Account 2>" : 60 }`.
When `{ "__update__" : [ ] }` is written.
Then the stored distribution is unchanged, because every plan is non-changing and the leftover
restores the whole document.

**AA-AC-070. Clearing every plan at once makes no change at all.**
Given a journal item carrying `{ "<Account 1>,<Account 3>" : 100 }`.
When `{ "__update__" : [ column(Plan 1) , column(Plan 2) ] }` is written with no other entry.
Then both sides of the merge are empty, the merged document is empty, and the caller leaves the
record exactly as it was: nothing is erased, nothing is created and no error is raised.

**AA-AC-071. The larger side keeps its surplus — the new side is larger.**
Given a journal item carrying `{ "<Account 1>" : 40 , "<Account 2>" : 60 }`.
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 33 , "<Account 4>" : 167 }` is written.
Then the stored distribution is `{ "<Account 1>,<Account 3>" : 6.6 , "<Account 1>,<Account 4>" :
33.4 , "<Account 2>,<Account 3>" : 9.9 , "<Account 2>,<Account 4>" : 50.1 , "<Account 3>" : 16.5 ,
"<Account 4>" : 83.5 }`, with *Plan 1* totalling one hundred and *Plan 2* two hundred.

**AA-AC-072. The larger side keeps its surplus — the old side is larger.**
Given a journal item carrying `{ "<Account 3>" : 33 , "<Account 4>" : 167 }`.
When `{ "__update__" : [ column(Plan 1) ] , "<Account 1>" : 40 , "<Account 2>" : 60 }` is written.
Then the stored distribution is `{ "<Account 3>,<Account 1>" : 6.6 , "<Account 4>,<Account 1>" :
33.4 , "<Account 3>,<Account 2>" : 9.9 , "<Account 4>,<Account 2>" : 50.1 , "<Account 3>" : 16.5 ,
"<Account 4>" : 83.5 }`; the identifiers of the non-changing side come first in every compound key.

**AA-AC-073. The larger side keeps its surplus — the new side is smaller.**
Given a journal item carrying `{ "<Account 1>" : 40 , "<Account 2>" : 60 }`.
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 20 , "<Account 4>" : 30 }` is written.
Then the stored distribution is `{ "<Account 1>,<Account 3>" : 8 , "<Account 1>,<Account 4>" : 12 ,
"<Account 2>,<Account 3>" : 12 , "<Account 2>,<Account 4>" : 18 , "<Account 1>" : 20 ,
"<Account 2>" : 30 }`.

**AA-AC-074. The larger side keeps its surplus — the old side is smaller.**
Given a journal item carrying `{ "<Account 3>" : 20 , "<Account 4>" : 30 }`.
When `{ "__update__" : [ column(Plan 1) ] , "<Account 1>" : 40 , "<Account 2>" : 60 }` is written.
Then the stored distribution is `{ "<Account 3>,<Account 1>" : 8 , "<Account 4>,<Account 1>" : 12 ,
"<Account 3>,<Account 2>" : 12 , "<Account 4>,<Account 2>" : 18 , "<Account 1>" : 20 ,
"<Account 2>" : 30 }`.

**AA-AC-075. A merge onto an empty document returns the incoming document.**
Given a journal item with no distribution.
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 25 , "<Account 4>" : 75 }` is written.
Then the stored distribution is `{ "<Account 3>" : 25 , "<Account 4>" : 75 }`, and no division by
zero occurs.

**AA-AC-076. The marker never survives.**
Given any of the scenarios `AA-AC-064` to `AA-AC-075`.
Then the stored distribution contains no key named `__update__`.

**AA-AC-077. A distribution that names a deleted account stays valid.**
Given a journal item carrying `{ "7,4" : 100 }`, where account 7 belongs to an optional plan and
account 4 to a mandatory one.
When account 7 and its plan are deleted.
Then the stored document is unchanged, the derived list of analytic accounts contains only account
4, the mandatory plan still totals one hundred, and the record is still valid.

---

## 7. Splitting analytic lines

**AA-AC-078. Adding a plan splits each analytic line.**
Given two analytic lines: L1 with *Account 1* in column(*Plan 1*) and the amount 40.00, and L2 with
*Account 2* in column(*Plan 1*) and the amount 60.00.
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 25 , "<Account 4>" : 75 }` is written on
both.
Then four analytic lines exist — (*Account 1*, *Account 3*, 10.00), (*Account 2*, *Account 3*,
15.00), (*Account 1*, *Account 4*, 30.00) and (*Account 2*, *Account 4*, 45.00) — the first two
being L1 and L2 rewritten and the last two newly created, and the total is still 100.00.

**AA-AC-079. The notification reports the created lines.**
Given `AA-AC-078`.
Then the acting reader receives the notification "2 analytic lines created".

**AA-AC-080. Splitting with a smaller new side leaves a remainder line.**
Given L1 (*Account 1*, 20.00) and L2 (*Account 2*, 30.00).
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 10 , "<Account 4>" : 40 }` is written on
both.
Then six lines exist: (*Account 1*, *Account 3*, 2.00), (*Account 1*, *Account 4*, 8.00),
(*Account 1*, none, 10.00), (*Account 2*, *Account 3*, 3.00), (*Account 2*, *Account 4*, 12.00) and
(*Account 2*, none, 15.00); the notification reads "4 analytic lines created".

**AA-AC-081. Splitting with a larger new side creates lines that carry only the new plan.**
Given L1 (*Account 1*, 200.00) and L2 (*Account 2*, 300.00).
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 100 , "<Account 4>" : 400 }` is written
on both.
Then eight lines exist: (*Account 1*, *Account 3*, 40.00), (*Account 1*, *Account 4*, 160.00),
(none, *Account 3*, 160.00), (none, *Account 4*, 640.00), (*Account 2*, *Account 3*, 60.00),
(*Account 2*, *Account 4*, 240.00), (none, *Account 3*, 240.00) and (none, *Account 4*, 960.00); the
notification reads "6 analytic lines created".

**AA-AC-082. Splitting with fractional percentages.**
Given L1 (*Account 1*, 40.00) and L2 (*Account 2*, 60.00).
When `{ "__update__" : [ column(Plan 2) ] , "<Account 3>" : 33 , "<Account 4>" : 167 }` is written
on both.
Then eight lines exist with the amounts 6.60, 33.40, 6.60, 33.40 from L1 and 9.90, 50.10, 9.90,
50.10 from L2; the two lines carrying no *Plan 1* account come from the leftover of the merge.

**AA-AC-083. Clearing one plan on analytic lines keeps the other and creates nothing.**
Given four analytic lines carrying an account of each plan, with the amounts 45.00, 30.00, 15.00 and
10.00.
When `{ "__update__" : [ column(Plan 1) ] }` is written on all four, with no other entry.
Then the four lines keep their amounts and their *Plan 2* accounts, their column(*Plan 1*) is empty,
and no line is created or deleted.

**AA-AC-084. An empty update list changes no analytic line.**
Given L1 (*Account 1*, 40.00) and L2 (*Account 2*, 60.00).
When `{ "__update__" : [ ] }` is written on both.
Then the two lines are unchanged and no line is created.

**AA-AC-085. Clearing every percentage changes nothing and raises nothing.**
Given one analytic line carrying *Account 1* in column(*Plan 1*) and *Account 3* in column(*Plan 2*).
When `{ "__update__" : [ column(Plan 1) , column(Plan 2) ] }` is written with no other entry.
Then the line still exists, still carries both accounts and the same amount, and no error is raised.

**AA-AC-086. A simple split without the marker.**
Given one analytic line of −1 000.00 carrying account twelve of the root plan *Projects*.
When `{ "12" : 60 , "13" : 40 }` is written on it, with no marker.
Then the merge returns the incoming document unchanged, the line itself becomes −600.00 with account
twelve, one new line of −400.00 with account thirteen is created, and the notification reads
"1 analytic lines created".

---

## 8. Validating mandatory plans

**AA-AC-087. A mandatory plan missing at posting.**
Given a root plan *Departments* whose applicability for the business domain `bill` is `mandatory`, a
root plan *Projects* with the default applicability `optional`, and a vendor bill with one product
line of 1 000.00 on the account `6100 Consultancy` whose distribution is `{ "12" : 100 }`, account
twelve belonging to *Projects*.
When the bill is posted from the interface.
Then the posting is refused with "One or more lines require a 100% analytic distribution.", no
analytic line is created and the entry stays draft, because *Departments* accumulates zero.

**AA-AC-088. One compound key satisfies two mandatory plans at once.**
Given the same configuration with *Projects* also `mandatory`, and the distribution
`{ "12,45" : 100 }`, account forty-five belonging to *Departments*.
When the bill is posted.
Then it is posted: the single key contributes one hundred to *Projects* and one hundred to
*Departments*, and the document's values total one hundred, not two hundred.

**AA-AC-089. A shortfall of one hundredth of a percent is refused.**
Given *Departments* mandatory and the distribution `{ "45" : 33.33 , "46" : 33.33 , "47" : 33.33 }`,
all three accounts in *Departments*.
When the bill is posted.
Then it is refused with the same message: the accumulated 99.99 compared with 100 at two digits is
minus one.

**AA-AC-090. Values just above and just below.**
Given one mandatory plan and one account of it.
Then `{ "4" : 100 }` is valid; `{ "4" : 100.01 }` is refused; `{ "4" : 99.9 }` is refused; and
`{ "4" : 99.999 }` is normalised to one hundred on write and is therefore valid.

**AA-AC-091. Posting without the validation flag ignores mandatory plans.**
Given the invoice of `AA-AC-087` with the distribution `{ "<Account 4>" : 0.9 }` and *Plan 2*
mandatory.
When the invoice is posted programmatically, or by the automatic posting job, which does not switch
the validation flag on.
Then the invoice is posted and the analytic lines are created from the incomplete distribution.

**AA-AC-092. Mass posting reports the offending journal items.**
Given two customer invoices, each with one product line whose mandatory plan is not satisfied.
When both are posted at once with the validation flag on.
Then the operation is refused, neither invoice is posted, and the refusal offers a list of the
offending journal items titled "Items With Missing Analytic Distribution" behind a button labelled
"See items".

**AA-AC-093. Only product lines are validated.**
Given an invoice whose product line satisfies every mandatory plan and whose tax line, payment term
line, section and note carry no distribution.
When the invoice is posted with the validation flag on.
Then it is posted: only journal items whose display type is `product` are checked.

**AA-AC-094. The invalid-analytics flag never blocks by itself.**
Given a draft invoice whose product line fails a mandatory plan and is booked on an income account.
Then the line carries the invalid-analytics flag and is highlighted, the invoice may still be saved,
and the refusal comes only at posting.
Given a second line booked on a receivable account with the same incomplete distribution.
Then that line does **not** carry the flag, because receivable, payable, cash and credit card
accounts are excluded from it.

**AA-AC-095. Validation ignores accounts that no longer exist.**
Given a plan *Test Plan* with an account *Test Account*, a mandatory plan *Mandatory Plan* with an
account *Mandatory Account*, and a record whose distribution is `{ "<Test Account>" : 100 }`.
When the distribution is validated with the validation flag on.
Then it is refused with "One or more lines require a 100% analytic distribution."
When the distribution becomes `{ "<Test Account>,<Mandatory Account>" : 100 }` and is validated
again.
Then it succeeds.
When *Test Account* and *Test Plan* are then deleted and the same document is validated again.
Then it still succeeds.

**AA-AC-096. Posting is refused when an analytic account is archived.**
Given a customer invoice whose line carries `{ "<Account 1>" : 100 }` and *Account 1* is archived.
When the invoice is posted.
Then the posting is refused with "You cannot post an entry with an archived analytic account:
Account 1", and no analytic line is created.
When *Account 1* is unarchived and the posting is retried.
Then the invoice is posted.

**AA-AC-097. An exchange difference entry is never blocked by a mandatory plan.**
Given a mandatory rule on *Plan 2* for the business domain `general`, a customer invoice in a
foreign currency posted at one rate with a complete distribution, and a credit note of that invoice
dated at a different rate.
When the credit note is posted with the validation flag on and reconciled with the invoice.
Then the credit note is posted and the exchange difference entry created by the reconciliation is
posted as well, because it is posted with the validation flag explicitly off.

**AA-AC-098. Confirming a source document runs the same check.**
Given a sales order whose line's product makes *Plan 2* mandatory for the business domain
`sale_order`, and the line carries no distribution.
When the order is confirmed.
Then the confirmation is abandoned with "One or more lines require a 100% analytic distribution."
When the line receives `{ "<Account 3>" : 100 }` and the confirmation is retried.
Then the order is confirmed, and the distribution is copied onto the invoice line created from it.

**AA-AC-099. A timesheet needs every mandatory plan except the base plan.**
Given a project bound to an analytic account of the base plan, and a root plan *Plan 2* mandatory
for the business domain `timesheet`.
When a timesheet line is recorded on that project with no account of *Plan 2*.
Then it is refused with "One or more lines require a 100% analytic distribution."
When an account of *Plan 2* is set on the line.
Then it is accepted, and the base plan's account is taken from the project without being asked for.

---

## 9. Generating analytic lines on posting

**AA-AC-100. Analytic lines follow the distribution, above one hundred percent.**
Given a customer invoice for *Partner A* dated the first of January with one line of the product
*Desk* at 200.00, whose distribution is `{ "<Account 3>" : 100 , "<Account 4>" : 50 }`, both accounts
in *Plan 2*.
When the invoice is posted.
Then two analytic lines exist: 200.00 carrying *Account 3* — the entry that closes the plan at one
hundred percent receives the whole outstanding amount — and 100.00 carrying *Account 4*, being fifty
percent of 200.00; both carry the partner *Partner A*, the product *Desk*, the quantity of the
journal item, the invoice's accounting date, the financial account of the line, the journal item
itself and the category `invoice`.

**AA-AC-101. Changing the distribution of a posted invoice regenerates the lines.**
Given the posted invoice of `AA-AC-100`.
When the line's distribution is changed to `{ "<Account 3>" : 100 , "<Account 4>" : 25 }`.
Then the two former analytic lines are deleted and two new ones exist, 200.00 on *Account 3* and
50.00 on *Account 4*, with **new** identifiers, and the journal entry itself is unchanged.

**AA-AC-102. The sixty and forty split of a one thousand expense line.**
Given a journal item debiting `6100 Consultancy` by 1 000.00, a quantity of one, and the
distribution `{ "12" : 60 , "13" : 40 }`, both accounts in one plan.
When the entry is posted.
Then two analytic lines exist: −600.00 on account twelve and −400.00 on account thirteen, the second
computed by the closing-line rule as − 1 000.00 × ( 100 − 60 ) ÷ 100; their sum is exactly −1 000.00;
and **both** carry the quantity one, not a share of it.

**AA-AC-103. The cross-plan split of one thousand.**
Given a vendor bill with one product line whose balance is 1 000.00 and the distribution
`{ "7,12" : 60 , "8,12" : 40 }`, accounts 7 and 8 in the plan *Departments* and account 12 in the
plan *Project*.
When the bill is posted.
Then two analytic lines exist: −600.00 carrying account 12 in column(*Project*) and account 7 in
column(*Departments*), and −400.00 carrying account 12 and account 8; their sum is exactly
−1 000.00; *Departments* shows −600.00 and −400.00 on two accounts and *Project* shows −1 000.00 on
one.

**AA-AC-104. Three thirds that divide exactly.**
Given a journal item of balance 1 000.00 and the distribution
`{ "7" : 33.33 , "8" : 33.33 , "9" : 33.34 }`, all three accounts in one plan.
When the entry is posted.
Then three analytic lines of −333.30, −333.30 and −333.40 exist, totalling exactly −1 000.00, and
the rounding pass changes nothing.

**AA-AC-105. A distribution below one hundred percent attributes less than the full amount.**
Given a journal item of balance 1 000.00 and the distribution
`{ "7" : 33.33 , "8" : 33.33 , "9" : 33.33 }`.
When the entry is posted.
Then three analytic lines of −333.30 each exist, totalling −999.90; the closing-line rule never
fires because no entry brings the plan to one hundred, and nothing corrects the tenth.

**AA-AC-106. A candidate that rounds to zero creates no line.**
Given a journal item of balance 0.01 and the distribution
`{ "A" : 33.33 , "B" : 33.33 , "C" : 33.34 }` in a company whose currency rounds onto 0.01.
When the entry is posted.
Then **no** analytic line is created at all: each candidate is about −0.0033, which passes the zero
test, so all three are dropped before the rounding pass and the cent is simply not analysed.

**AA-AC-107. The quantity is copied whole.**
Given a journal item with a quantity of 10 and the distribution `{ "1" : 60 , "2" : 40 }`.
When the entry is posted.
Then both analytic lines carry the quantity 10.

**AA-AC-108. The description of a generated line.**
Given four journal items of a posted entry, each with the distribution `{ "1" : 100 }`: the first
with the label "Consulting fee"; the second with no label and the reference `INV/2026/0007`; the
third with no label, no reference and the partner *Acme*; the fourth with no label, no reference and
no partner.
Then the descriptions of the four analytic lines are, respectively, "Consulting fee",
`INV/2026/0007`, "/ -- Acme" and "/ -- /".

**AA-AC-109. The date follows the lock date adjustment.**
Given a vendor bill dated the first of January in a company whose purchase lock date is the
thirty-first of January.
When the bill is posted.
Then the entry's accounting date is moved past the lock date first, the analytic lines are created
afterwards, and every analytic line carries the **adjusted** date.

**AA-AC-110. The user of a generated line.**
Given a posted invoice whose invoicing salesperson is *Sales User*, posted by *Accountant*.
Then every analytic line generated from it carries *Sales User*.
Given a posted miscellaneous entry with no salesperson, posted by *Accountant*.
Then every analytic line generated from it carries *Accountant*.

**AA-AC-111. The partner of an analytic line follows its journal item.**
Given a posted miscellaneous entry with a debit of 200.00 on the receivable account for *Partner A*
and a credit of 200.00 on the revenue account for *Partner B* carrying `{ "<Account 1>" : 100 }`.
Then one analytic line of 200.00 exists, with the partner *Partner B*.
When the analytic line is attached to the first journal item instead.
Then its partner becomes *Partner A*.
When both journal items are given *Partner B*.
Then the analytic line's partner becomes *Partner B*.

**AA-AC-112. The category of a generated line.**
Given three posted documents carrying a distribution: a customer invoice, a vendor bill and a
miscellaneous entry.
Then the analytic lines carry the categories `invoice`, `vendor_bill` and `other` respectively.

**AA-AC-113. A payment term line produces nothing.**
Given a posted customer invoice of 200.00 whose product line carries a distribution.
Then one analytic line exists for the product line and none for the receivable payment term line,
which carries no distribution.

---

## 10. Rounding

**AA-AC-114. A positive error of two steps over four lines.**
Given a customer invoice line whose balance is −182.25 and the distribution
`{ "1" : 94 , "2" : 2 , "3" : 2 , "4" : 2 }`, accounts 1 and 2 having the root plan *Plan 1* and
accounts 3 and 4 the root plan *Plan 2*, so that no entry closes either plan.
When the invoice is posted.
Then the unrounded candidates are 171.3150, 3.6450, 3.6450 and 3.6450, the rounded ones 171.32,
3.65, 3.65 and 3.65 with an accumulated error of +0.02, and the correction removes one hundredth
from the **first** candidate and then one from the second: the final amounts are 171.31, 3.64, 3.65
and 3.65, totalling exactly 182.25.

**AA-AC-115. A negative error of one step over four lines.**
Given the same journal item with `{ "1" : 25 , "2" : 25 , "3" : 25 , "4" : 25 }`.
When the invoice is posted.
Then the four candidates are 45.5625 each, rounding to 45.56 with an accumulated error of −0.01, and
one hundredth is **added** to the first: the final amounts are 45.57, 45.56, 45.56 and 45.56,
totalling exactly 182.25.

**AA-AC-116. Three thirds that do not divide.**
Given a journal item of balance 1 001.00 and the distribution
`{ "7" : 33.33 , "8" : 33.33 , "9" : 33.34 }`.
When the entry is posted.
Then the candidates are −333.6333, −333.6333 and −333.7334, rounding to −333.63, −333.63 and
−333.73 with an error of +0.01, and the final amounts are **−333.64**, −333.63 and −333.73,
totalling exactly −1 001.00: the cent lands on the first entry.

**AA-AC-117. A very small amount.**
Given a journal item of balance 0.10 and the distribution
`{ "A" : 33.33 , "B" : 33.33 , "C" : 33.34 }`.
When the entry is posted.
Then the three amounts are −0.04, −0.03 and −0.03, totalling −0.10.

**AA-AC-118. Seven slices with a two-step error.**
Given a journal item of balance 100.05 and seven slices of 14.29 except the last, 14.26.
When the entry is posted.
Then the candidates round to −14.30 six times and −14.27 once, with an accumulated error of −0.02,
and the correction adds one hundredth to the first and then to the second: the final amounts are
−14.29, −14.29, −14.30, −14.30, −14.30, −14.30 and −14.27, totalling exactly −100.05.

**AA-AC-119. Rounding uses the company currency, small case.**
Given a currency whose rounding step is one unit at one hundred units per unit of company currency,
and a customer invoice of 2.00 in that currency with the distribution `{ "<Account 1>" : 100 }`.
When the invoice is posted.
Then one analytic line of **0.02** exists, not zero.

**AA-AC-120. Rounding uses the company currency, fractional case.**
Given a currency whose rounding step is one unit at three units per unit of company currency, and a
customer invoice of 10.00 in that currency with the distribution `{ "<Account 1>" : 100 }`.
When the invoice is posted.
Then one analytic line of **3.33** exists, matching the journal item's balance exactly.

---

## 11. Rebuilding a distribution from analytic lines

**AA-AC-121. Editing an analytic line rebuilds the journal item's distribution.**
Given a posted customer invoice with one line of 100.00, a balance of −100.00 and the distribution
`{ "<Account 1>" : 40 , "<Account 2>" : 60 }`, therefore two analytic lines of 40.00 and 60.00.
When the analytic line of 40.00 is given *Account 3* in column(*Plan 2*) and the amount 50.00.
Then the journal item's distribution becomes
`{ "<Account 1>,<Account 3>" : 50 , "<Account 2>" : 60 }` and the edited analytic line is **not**
deleted and regenerated.

**AA-AC-122. Deleting an analytic line rebuilds the distribution.**
Given the state reached in `AA-AC-121`.
When the analytic line of 50.00 is deleted.
Then the journal item's distribution becomes `{ "<Account 2>" : 60 }`.

**AA-AC-123. Creating an analytic line rebuilds the distribution.**
Given the state reached in `AA-AC-122`.
When an analytic line named *Extra Analytic Line* with *Account 1*, the amount 30.00 and that
journal item is created.
Then the journal item's distribution becomes `{ "<Account 1>" : 30 , "<Account 2>" : 60 }`.

**AA-AC-124. Detaching and reattaching analytic lines.**
Given the state reached in `AA-AC-123`.
When the journal item link is cleared on both analytic lines.
Then the journal item has no distribution, and the distribution of the journal item pointed at
before the write was rebuilt.
When the link is set again on both.
Then the distribution is `{ "<Account 1>" : 30 , "<Account 2>" : 60 }` again.

**AA-AC-125. A zero balance yields one hundred percent.**
Given a posted customer invoice whose single line has a unit price of 0.00 and therefore a balance
of 0.00.
When an analytic line of 33.00 with *Account 1* is created on that journal item.
Then the journal item's distribution is `{ "<Account 1>" : 100 }`.

**AA-AC-126. Two lines with the same combination collapse.**
Given a posted journal item of balance −100.00 with two analytic lines that carry exactly the same
combination of accounts, of 30.00 and 70.00, the 70.00 one processed last.
Then the rebuilt distribution is `{ "<the combination>" : 70 }`, **not** 100: the last line
processed wins and part of the attribution is lost. This is the behaviour to reproduce.

**AA-AC-127. A manual edit may leave a distribution above one hundred.**
Given the two analytic lines of `AA-AC-121` before any edit, 40.00 and 60.00.
When the first line's amount is changed to 70.00.
Then the distribution becomes `{ "<Account 1>" : 70 , "<Account 2>" : 60 }`, totalling one hundred
and ten; the system records it as it stands and corrects nothing.

**AA-AC-128. Analytic lines created on a draft entry are consumed and removed.**
Given a draft journal entry created with two journal items: the first debited 2 000.00 and carrying
one analytic line of −2 000.00 that names *Account 2* through the magic column and *Account 1* and
*Account 3* in two plan columns; the second credited 2 000.00 and carrying one analytic line of
2 000.00 that names only *Account 2*.
Then **no** analytic line exists for the entry, the first journal item's distribution is
`{ "<Account 2>,<Account 1>,<Account 3>" : 100 }` and the second's is `{ "<Account 2>" : 100 }`.
When a further analytic line naming only *Account 1* is written on the first journal item.
Then still no analytic line exists and the first journal item's distribution is
`{ "<Account 1>" : 100 }`, because the rebuild takes the lines existing at that moment.
When the entry is posted.
Then analytic lines exist again, generated from the distributions.

---

## 12. Derived journal items

**AA-AC-129. Tax lines and balancing lines inherit the distribution.**
Given a miscellaneous entry with one line debited 100.00 on an account carrying a sale tax, with
that tax set on the line.
When `{ "<Account 1>" : 100 }` is written on all the lines of the entry and the entry is saved.
Then the automatic balancing line, the tax line and the original line all carry
`{ "<Account 1>" : 100 }`, and posting the entry produces one analytic line for each of them.

**AA-AC-130. Discount allocation lines carry a weighted distribution.**
Given a company whose discount allocation account is `DIS Discount Expense` and a customer invoice
with two untaxed lines: the product *Desk* at 200.00 with a twenty percent discount and
`{ "<Account 1>" : 100 }`, and the product *Chair* at 200.00 with a ten percent discount and
`{ "<Account 2>" : 100 }`.
When the invoice is posted.
Then the journal items are, in order: a product line of −160.00 with `{ "<Account 1>" : 100 }`; a
product line of −180.00 with `{ "<Account 2>" : 100 }`; a discount line of −40.00 with
`{ "<Account 1>" : 100 }`; a discount allocation line of +60.00 with
`{ "<Account 1>" : 66.67 , "<Account 2>" : 33.33 }`; a discount line of −20.00 with
`{ "<Account 2>" : 100 }`; and a payment term line of +340.00 with no distribution.
And the analytic lines are +160.00, +180.00, +40.00, −40.00 and −20.00 and +20.00, so that
*Account 1* nets 160.00 and *Account 2* nets 180.00.

**AA-AC-131. The discount allocation is recomputed when a line changes.**
Given a company whose discount allocation account is its default expense account and an invoice with
two lines of 1 000.00: the first with a twenty percent discount and
`{ "<Account 1>" : 60 , "<Account 2>" : 40 }`, the second with a ten percent discount and
`{ "<Account 3>" : 80 , "<Account 4>" : 20 }`.
When the discount of the second line is changed to twenty percent.
Then the journal item booked on the expense account has a balance of 400.00 and the distribution
`{ "<Account 1>" : 30 , "<Account 2>" : 20 , "<Account 3>" : 40 , "<Account 4>" : 10 }`.

**AA-AC-132. An early payment discount line takes its distribution from the models.**
Given a payment term granting an early payment discount, an invoice line carrying
`{ "<Account 1>" : 100 }`, and a distribution model whose account prefix matches the cash discount
account's code and whose distribution is `{ "<Account 3>" : 100 }`.
When the invoice is posted.
Then the discount line booked on the product's own account carries `{ "<Account 1>" : 100 }` and its
counterpart on the cash discount account carries `{ "<Account 3>" : 100 }`, and both produce
analytic lines.

---

## 13. Reset, reversal and deletion

**AA-AC-133. Resetting to draft deletes the analytic lines and keeps the distributions.**
Given the posted invoice of `AA-AC-100`.
When the invoice is reset to draft.
Then no analytic line refers to any of its journal items, the distributions stored on those items
are unchanged, and no error is raised.
When the invoice is posted again.
Then two analytic lines of 200.00 and 100.00 exist again, with new identifiers.

**AA-AC-134. Cancelling follows the reset path.**
Given a posted entry with analytic lines.
When it is cancelled and the cancellation resets it to draft.
Then its analytic lines are deleted exactly as in `AA-AC-133`.

**AA-AC-135. A reversal produces opposite lines and keeps the originals.**
Given a posted bill dated the first of March 2026, debiting `6100 Consultancy` by 1 000.00 with the
distribution `{ "12" : 60 , "13" : 40 }`, which produced −600.00 and −400.00.
When the bill is reversed in full on the thirty-first of March 2026 and the reversing entry is
posted.
Then the reversing entry's item credits 1 000.00 with the copied distribution, and two new analytic
lines of **+600.00** and **+400.00** dated the thirty-first of March exist; the two original lines
are untouched; account 12 shows 600.00 of debit and 600.00 of credit and a balance of zero; and a
balance computed for March alone still shows the original cost on the first of March.

**AA-AC-136. Deleting a journal item deletes its analytic lines.**
Given a posted entry, later reset to draft, whose journal item carries analytic lines created by a
manual edit.
When the journal item is deleted.
Then its analytic lines are deleted by cascade and no distribution rebuild is attempted on the
deleted item.

**AA-AC-137. A manual analytic line survives everything.**
Given an analytic line entered by hand with no journal item.
When any entry is posted, reset or reversed.
Then that line is untouched: it belongs to no journal item and no generation or deletion reaches it.

---

## 14. Analytic accounts

**AA-AC-138. The display name of an account.**
Given an account named *Seagate P2* with the reference `SEA` whose customer's commercial entity is
named *Deco Addict*.
Then its display name is "[SEA] Seagate P2 - Deco Addict".
Given an account named *Operating Costs* with no reference and no customer.
Then its display name is "Operating Costs".
Given an account named *Website redesign* with the reference `WEB-01` and no customer.
Then its display name is "[WEB-01] Website redesign".

**AA-AC-139. A name search matches the name or the reference.**
Given the account of `AA-AC-138` with the reference `SEA`.
When accounts are searched by the text `sea`.
Then that account is returned, whether the text matches the name or the reference, case-insensitively
and on a fragment.

**AA-AC-140. Duplicating an account renames the copy.**
Given an account named *Departments Budget*.
When it is duplicated with no explicit name.
Then the copy is named "Departments Budget (copy)" and every other stored field is copied.
When it is duplicated with the explicit name *Other*.
Then the copy is named "Other".

**AA-AC-141. Debit, credit and balance.**
Given an analytic account with four analytic lines of +2 400.00, −1 250.00, −340.50 and 0.00, all in
*Main Company*.
Then its credit is 2 400.00 — the zero line counts on the credit side, because the test is "greater
than or equal to zero" — its debit is 1 590.50 and its balance is 809.50.

**AA-AC-142. A balance over a date range.**
Given the analytic account *Website redesign* in a company reporting in euro with the lines
−1 200.00 on the fifteenth of January 2026, −450.00 on the third of February, +3 000.00 on the
twentieth of February, −800.00 on the fifth of March and −300.00 on the second of April.
Then with no date range the credit is 3 000.00, the debit is 2 750.00 and the balance is 250.00.
When the reporting context carries the from-date the first of February 2026 and the to-date the
thirty-first of March 2026.
Then the credit is 3 000.00, the debit is 1 250.00 and the balance is 1 750.00.

**AA-AC-143. A balance across currencies converts at today's rate.**
Given the range of `AA-AC-142` and the line of the fifth of March belonging to a subsidiary
reporting in United States dollars, recorded as −900.00, with the reader acting for both companies
whose own currency is the euro and today's rate being 1.20 United States dollars per euro.
Then the credit is 3 000.00, the debit is 450.00 + 750.00 = 1 200.00 and the balance is 1 800.00,
the conversion being made at **today's** rate and not at the rate of the fifth of March.
When the reader acts for the subsidiary alone.
Then only the −900.00 line remains, expressed in United States dollars with no conversion.

**AA-AC-144. The three totals may be summed in a grouped list.**
Given a list of analytic accounts grouped by plan.
Then each group shows the sum of the balance and, for a reader who also holds the read-only
accounting group or the invoicing group, the sums of the debit and the credit; the sums are computed
in memory over the accounts of the group, optionally converting each account's figure into the
currency of the company the reader is acting for.

**AA-AC-145. The company of an account cannot change once it has lines elsewhere.**
Given an analytic line of *Main Company* naming *Account 3*.
When the company of *Account 3* is set to *Company Two*.
Then the change is refused with "You can't change the company of an analytic account that already
has analytic items! It's a recipe for an analytical disaster!"
When the company of *Account 3* is cleared instead.
Then the change succeeds, because every existing line then satisfies the condition.

**AA-AC-146. A branch may use an account of its parent company.**
Given *Account 1* whose company is *Main Company* and a branch company *B Branch* whose parent is
*Main Company*.
When an analytic line of *B Branch* naming *Account 1* is created.
Then the creation succeeds; and setting the company of *Account 1* to *Main Company* while that line
exists is allowed, because *B Branch* is a descendant of the new company.

**AA-AC-147. The customer of an account must be compatible with its company.**
Given an account whose company is *Main Company*.
When its customer is set to a contact belonging to *Company Two*.
Then the write is refused with the platform's company-inconsistency message, beginning "Uh-oh!
You've got some company inconsistencies here:" and ending "To avoid a mess, no company crossover is
allowed!"

**AA-AC-148. Deleting an account used by an expense is refused.**
Given an expense whose distribution names *Account 1*.
When *Account 1* is deleted.
Then the deletion is refused with "You cannot delete an analytic account that is used in an
expense."

**AA-AC-149. Deleting an account bound to a project with tasks is refused.**
Given a project bound to *Account 1* and having at least one task.
When *Account 1* is deleted.
Then the deletion is refused with "Before we can bid farewell to these accounts, you need to tidy up
the projects linked to them by removing their existing tasks!"

**AA-AC-150. Deleting an account referenced by an analytic line is refused.**
Given an analytic line holding *Account 3* in column(*Plan 2*).
When *Account 3* is deleted.
Then the deletion is refused by the restricting deletion rule of the column.
When only a distribution — not a line — names *Account 3*.
Then the deletion succeeds and the distribution keeps the identifier, which is thereafter ignored.

**AA-AC-151. Archiving hides an account and blocks posting.**
Given *Account 1* is active and used in the distribution of a draft invoice.
When *Account 1* is archived.
Then a red ribbon reading "Archived" appears on its form, the change is written to its discussion
thread, the account disappears from the default account list and from the value lists of the
distribution editor, the existing distribution and the existing analytic lines still name it, and
posting the invoice is refused with the archived-account message.
When it is unarchived.
Then the change is written to the thread and the posting succeeds.

**AA-AC-152. Creating an account increases the counts of every ancestor plan.**
Given the plans of `AA-AC-019`.
When one more account is created in *Sub Sub Plan*.
Then the all-accounts count becomes 4 for *Parent Plan*, 3 for *Sub Plan* and 2 for *Sub Sub Plan*,
and the direct account count of *Sub Sub Plan* becomes 2.

---

## 15. Analytic lines

**AA-AC-153. An analytic line needs at least one account.**
When an analytic line is created with a description, a date and an amount but no plan column filled.
Then the creation is refused with "At least one analytic account must be set".
When one plan column is filled.
Then the creation succeeds.
When every plan column of an existing line is cleared.
Then the write is refused with the same message.

**AA-AC-154. The financial account must match the journal item.**
Given an analytic line attached to a journal item booked on the account `600000`.
When the line's financial account is set to `601000`.
Then the change is refused with "The journal item is not linked to the correct financial account".

**AA-AC-155. The required fields and their defaults.**
When an analytic line is created from the analytic item screen with only a description and one
account.
Then the date defaults to today in the reader's time zone, the amount defaults to 0.00, the company
defaults to the company the reader is acting for, the currency is that company's currency and the
category is `other`.

**AA-AC-156. The company of a line never changes.**
Given an analytic line of *Main Company*.
Then the company field is read-only on the form, and the line keeps *Main Company* for its whole
life.

**AA-AC-157. The product helper values a manual line.**
Given a product whose cost is 12.50 per unit and whose category resolves to the expense account
`600000`.
When a manual analytic line is created, the product is chosen and the quantity is set to 4.
Then the amount becomes −50.00, the unit becomes the product's own unit, and the financial account
becomes `600000`; the analytic account's debit increases by 50.00 and its balance decreases by
50.00.
When the quantity is changed to 8.
Then the amount becomes −100.00.
When no product is chosen at all.
Then the helper does nothing and the typed amount is kept.

**AA-AC-158. The profitability classification.**
Given six analytic lines: one on an account of the type `expense`; one on an account of the type
`income`; one with no financial account, the category `other` and the amount −120.00; one with no
financial account and the category `picking_entry`; one with no financial account, the category
`other` and the amount +250.00; one with no financial account, the category `other` and the amount
0.00.
Then their classifications are `loss`, `revenue`, `loss`, `loss`, `revenue` and `uncategorized`.
Given a line on an account of the type `asset_receivable`, and one on an account of the type
`liability_payable`.
Then both are `uncategorized`.
Given a line on an account of the type `expense_direct_cost` and one on an account of the type
`asset_fixed`.
Then both are `loss`, because only the first fragment of the account type is compared.

**AA-AC-159. The fiscal year search helper.**
Given a company whose fiscal year starts on the first of January, and today is the fifteenth of June
2026.
When analytic lines are searched with **any** condition on the fiscal year helper, whatever its
operator and its value.
Then the condition applied is "the date is on or after the first of January 2025".

**AA-AC-160. The magic column resolves through the reader's context.**
Given an analytic line carrying *Account 1* in column(*Plan 1*) and *Account 3* in column(*Plan 2*).
When the line is read with *Plan 1* named in the context.
Then the magic column reads *Account 1*.
When an account of *Plan 2* is written into the magic column.
Then it is stored in column(*Plan 2*), whatever the context says, because a write places the account
in the column of its own root plan.
When analytic lines are searched by the magic column for *Account 3*.
Then the condition expands into a disjunction over every root plan's column; a negated operator is
refused.

**AA-AC-161. The default ordering of the analytic item list.**
Given three analytic lines dated the first, the second and the second of March, the last two created
in that order.
Then the list shows the second of March created last, the second of March created first, then the
first of March: date descending, then internal identifier descending.

---

## 16. Searching and grouping by distribution

**AA-AC-162. Filtering an already-loaded set by account name.**
Given six distribution models: one with `{ Sales : 50 , Administrative : 50 }`, one with
`{ Research & Development : 100 }`, one with `{ Commercial : 100 }`, one with
`{ Commercial & Marketing : 100 }`, and two with no distribution at all.
When the loaded set is filtered by the condition "the distribution equals *Research & Development*".
Then only the second model is returned.
When it is filtered by "equals *Sales*" and then by "equals *Administrative*".
Then the first model is returned in both cases.
When it is filtered by "equals the empty text".
Then nothing is returned.
When it is filtered by "equals the identifier of *Commercial*".
Then the third model is returned.
When it is filtered by "contains *Commercial*".
Then the third and the fourth models are returned.
When it is filtered by "contains the empty text".
Then every model with a non-empty distribution is returned.
When it is filtered by "does not contain *Commercial*".
Then every model except the third and the fourth is returned.
When it is filtered by "does not contain the empty text".
Then only the two models with no distribution are returned.
When it is filtered by "differs from *Commercial & Marketing*".
Then every model except the fourth is returned.
When it is filtered by "differs from the empty text".
Then every model is returned.
When it is filtered by "differs from the identifier of *Commercial*".
Then every model except the third is returned.
When it is filtered by "is one of [ the identifier of *Commercial* ]".
Then the third model is returned.
When it is filtered by "is one of [ *Sales* , *Research & Development* ]" by identifier.
Then the first and the second models are returned.

**AA-AC-163. An unsupported operator is refused by the database search.**
When records are searched **in the database** with a condition on the distribution field using an
operator other than "in", "not in", "contains" and "does not contain".
Then the search fails with "Operation not supported".
When the same condition is applied to an already-loaded set, as in `AA-AC-162`.
Then the equality and inequality operators are accepted, because the condition is first rewritten as
a condition on the derived list of analytic accounts.

**AA-AC-164. "Has no distribution" works.**
When records are searched in the database with the condition "the distribution is one of [ the empty
value ]".
Then the records whose distribution is empty are returned, the list being passed through unchanged.
When the negated form of a name condition is used.
Then the records whose distribution is empty are additionally returned.

**AA-AC-165. Name resolution in a database search.**
When records are searched **in the database** with the inclusion operator and a list of account
names.
Then the elements are passed through unresolved and nothing matches; this is the **compatibility
finding** of [business-rules.md](business-rules.md) `AA-057`.
When the partial-match operator is used with a text fragment.
Then the accounts whose display name contains that fragment are resolved and the matching records
are returned.

**AA-AC-166. Grouping by distribution counts owners.**
When journal items are grouped by the distribution field with the count aggregate.
Then each row corresponds to one analytic account and counts the distinct **journal entries** whose
journal items name that account.
When purchase order lines are grouped the same way.
Then each row counts the distinct purchase orders.
When the same grouping is asked for with any other aggregate.
Then it fails with "analytic_distribution grouping does not accept the aggregate specification as
aggregate.", the placeholder being the offending aggregate.
When a model that declares no owner is grouped this way.
Then it fails with "the table name does not support analytic_distribution grouping.", the
placeholder being the offending table name.

**AA-AC-167. Cross-plan combinations are counted once per document.**
Given a posted customer invoice whose single line carries
`{ "<Account 3>,<Account 5>" : 20 , "<Account 3>,<Account 4>" : 80 }`, and a posted vendor bill with
two lines carrying `{ "<Account 3>,<Account 4>" : 100 }` and
`{ "<Account 3>,<Account 5>" : 50 , "<Account 4>" : 50 }`.
Then the invoice count of *Account 3* is 1 and its vendor bill count is 1.

---

## 17. Redistribution and project profitability

**AA-AC-168. Redistribution updates the existing lines in place.**
Given a valuation document with two analytic lines carrying the combinations (A1) and (A2), amounts
60.00 and 40.00.
When the document redistributes 100.00 with the target `{ "A1" : 70 , "A3" : 30 }` and a total
quantity of 5.
Then the line carrying (A1) is rewritten to 70.00 with the quantity 5 and **keeps its identifier**;
the line carrying (A2) is deleted, because its combination is no longer a key; and one new line of
30.00 with the quantity 5 is created for (A3).

**AA-AC-169. Redistribution deletes a line whose new amount rounds to zero.**
Given the same document and the target `{ "A1" : 100 , "A2" : 0.001 }`.
When the redistribution runs on a total amount of 100.00.
Then the line for (A2) is deleted, because its new amount passes the zero test.

**AA-AC-170. An empty target deletes every line.**
Given a valuation document with three analytic lines.
When it redistributes with an empty distribution.
Then all three lines are deleted and nothing is created.

**AA-AC-171. The plan-closing rule of a redistribution compares against the plan's own total.**
Given a valuation of 100.00 attributed `{ "1" : 33.33 , "2" : 33.33 , "3" : 33.34 }`, the three
accounts in one plan whose total is therefore one hundred.
Then the amounts are 33.33, 33.33 and 33.34, the last computed as ( 100 × 100 ÷ 100 ) − 66.66.
Given the same valuation attributed `{ "1" : 20 , "2" : 30 }` in one plan whose total is fifty.
Then the amounts are 20.00 and 30.00, the second closing against **fifty**, not one hundred, and the
two lines total 50.00.

**AA-AC-172. The vendor bills section of a project.**
Given a project whose analytic account is account 12, reporting in euro; a posted vendor bill with a
journal item of balance +1 000.00 and the distribution `{ "12" : 60 , "12,45" : 20 }`; and a draft
vendor bill with a journal item of balance +500.00 and `{ "12" : 100 }`; neither coming from a
purchase order.
Then the section identified `other_purchase_costs`, labelled "Vendor Bills" with the sequence 11,
shows a billed amount of −800.00 — the contribution being ( 60 + 20 ) ÷ 100 — and a to-bill amount
of −500.00.
When both bills are removed.
Then the section is not shown at all, because both totals are zero.

**AA-AC-173. The other-revenues and other-costs sections of a project.**
Given a project whose analytic account carries four analytic lines with no journal item, of +100,
−100, +50 and −50, in a company whose currency converts at 0.2 units of the project's currency, and
four more of +100, −100, +50 and −50 already in the project's currency.
Then the section identified `other_revenues_aal`, labelled "Other Revenues" with the sequence 14,
shows 180.00 as invoiced and 0.00 as to-invoice, and the section identified `other_costs_aal`,
labelled "Other Costs" with the sequence 15, shows −180.00 as billed and 0.00 as to-bill.
Given one further analytic line of the category `manufacturing_order` and one of the category
`picking_entry`.
Then neither changes those two sections, because both categories are excluded.

**AA-AC-174. The sections open their records only for an accounting reader.**
Given the sections of `AA-AC-173`.
When a reader who holds no accounting group opens the profitability panel.
Then the amounts are shown and no opening action is attached to the two analytic sections.
When a reader who holds the read-only accounting group opens it.
Then pressing a section opens the analytic items behind it, grouped by date, with the graph and
pivot presentations of the project accounting capability.

---

## 18. Screens, the editor and reporting

**AA-AC-175. Plan columns appear automatically on the analytic item list.**
Given three root plans, each with at least one account.
When the analytic item list is opened.
Then it shows the base plan's column and one column per other root plan, inserted after it in plan
order, each labelled with its plan's name and offering only the accounts of that plan or of any
descendant of it.

**AA-AC-176. Grouping by sub-plan depth is offered and folded.**
Given a root plan with a child and a grandchild, both depths populated with accounts.
When the analytic item search panel is opened.
Then a grouping entry exists for the root plan, and the entries for depth one and depth two are
folded into a single entry whose options are the two depths; choosing an option groups by that
depth.
When the grandchild plan is deleted and a stored personal filter still names its grouping column.
Then that entry is removed from the panel rather than failing.

**AA-AC-177. The gross margin screen of an account.**
Given an analytic account with analytic lines.
When the *Gross Margin* button is pressed on its form.
Then the analytic items whose magic column resolves to that account are listed, grouped by date, and
creating a line there pre-fills that account in the column of its own root plan.

**AA-AC-178. The list header names the account.**
Given the analytic item list opened from an analytic account.
Then the header reproduces "Entries: " followed by the account's name.

**AA-AC-179. The proposed percentage of a new row.**
Given a distribution editor showing a mandatory plan whose running total is 40 percent and an
optional plan whose running total is 90 percent.
When a row is added.
Then the proposed percentage is 60 percent, the mandatory plan deciding.
Given the mandatory plan complete at 100 percent and the optional plan at 90.
When a row is added.
Then the proposal is 10 percent.
Given every plan complete.
When a row is added.
Then the proposal is 100 percent, because the computed remainder is zero.

**AA-AC-180. The editor sums rows naming the same accounts and drops rows naming none.**
Given an editor with three rows: (*Account 1*) 30 percent, (*Account 1*) 20 percent, and one row
with no account and 50 percent.
When the editor is closed.
Then the written document is `{ "<Account 1>" : 50 }`.

**AA-AC-181. The editor drops accounts that no longer exist.**
Given a journal item whose stored distribution names an account that has been deleted.
When the editor is opened on that line.
Then the missing identifier is dropped and the cleaned document is saved immediately.

**AA-AC-182. Every exit from the editor saves.**
Given an editor with unsaved changes.
When Escape is pressed, or the reader clicks outside the widget, or the window is resized on a
desktop operating system.
Then the changes are saved; there is no discard.

**AA-AC-183. The closed cell shows one tag per plan.**
Given a distribution attributing (A1, B1, C1) 50, (A2, B1, C1) 50 and (A3, B1, C2) 50.
Then three tags are shown: "50% A1 | 50% A2 | 50% A3", "150% B1" and "C1 | 50% C2"; an account whose
total for its plan is exactly one hundred percent is shown without a percentage.

**AA-AC-184. The new-model shortcut pre-fills the model.**
Given an editor open on an invoice line for the partner *Acme*, with the product *Desk* and the
financial account whose display name begins with `600`.
When the new-model shortcut is used.
Then a distribution model form opens pre-filled with the grid as typed, the partner *Acme*, the
product *Desk* and the account prefix `600`, and only creation is offered.

**AA-AC-185. The distribution editor of a distribution model shows every plan.**
Given *Plan 1* is `unavailable` in every situation and has at least one account.
When the distribution editor is opened on a distribution model.
Then *Plan 1* is shown with the applicability `optional`, no rule having been evaluated, and the
new-model shortcut is disabled.

**AA-AC-186. The three totals are declared summable.**
Then the balance and, for a reader holding an accounting group, the debit and the credit are
declared as summable in a grouped list of analytic accounts.

**AA-AC-187. The analytic item screens open ungrouped.**
When the Analytic Items screen or the Analytic Reporting screen is opened from the menu.
Then the last-fiscal-year filter and the profit-and-loss-accounts filter are active and the list is
**not** grouped, because the requested default grouping names no existing filter; this is the
compatibility finding of [interfaces.md](interfaces.md) section 2.

---

## 19. Access, companies and currencies

**AA-AC-188. The analytic screens require the permission group.**
Given a reader without the analytic accounting permission group.
When that reader opens a customer invoice.
Then no analytic distribution cell is shown, and the analytic plan, account, item and distribution
model screens are not reachable.

**AA-AC-189. Enabling the setting grants the group.**
Given the analytic accounting setting is off.
When an administrator switches it on and saves.
Then every internal user holds the analytic accounting permission group and the full accounting
capability setting is on as well.
When the budget setting is switched on instead.
Then the analytic setting is switched on too.

**AA-AC-190. Master data of a parent company is visible from a branch; facts are not.**
Given an analytic account, an applicability rule and a distribution model whose company is *Main
Company*, and a branch *B Branch* whose parent is *Main Company*.
When a reader whose active company is *B Branch* lists them.
Then all three are returned.
When the same reader lists the analytic lines of *Main Company*.
Then none is returned.

**AA-AC-191. An analytic line is never visible from another company.**
Given an analytic line of *Company Two*.
When a reader whose active companies are only *Main Company* lists the analytic items.
Then that line is not returned, even when *Main Company* is the parent of *Company Two*.

**AA-AC-192. A shared account is visible everywhere.**
Given an analytic account with no company.
When any reader lists the analytic accounts.
Then it is returned, whatever the active companies.

**AA-AC-193. A reader with only the analytic group has the whole matrix.**
Given a reader holding only the analytic accounting permission group.
Then that reader may read, create, modify and delete plans, applicability rules, analytic accounts,
analytic lines and distribution models, subject to the record rules and to the guards of
[business-rules.md](business-rules.md).

**AA-AC-194. Elevated reads see what the reader cannot.**
Given an analytic line of *Company Two* naming *Account 3*, and a reader acting only for *Main
Company*.
When that reader changes the company of *Account 3* to *Main Company*.
Then the change is refused, because the consistency count is made with elevated rights and sees the
line of *Company Two*.

**AA-AC-195. A plan is global.**
Given two companies.
When a plan is created while acting for *Main Company*.
Then the plan is visible from *Company Two* as well; only its default applicability, which is stored
per company, and its rules, which may name a company, differ.

**AA-AC-196. The default applicability is stored per company.**
Given the plan *Plan 2* with the default applicability `optional` in *Main Company*.
When it is set to `mandatory` while acting for *Company Two*.
Then reading it while acting for *Main Company* still returns `optional`, and the relevant-plans
answer differs between the two companies.

---

## 20. Mapping of the former scenario numbers

The earlier draft of this folder numbered its scenarios `AN-AC-001` to `AN-AC-120`. The table maps
that scheme to this one; a scenario marked "new" was written for this consolidation, from the source
or from the worked examples of [calculations.md](calculations.md).

| This file | Former draft |
|---|---|
| `AA-AC-001` to `AA-AC-004` | `AN-AC-001` to `AN-AC-004` |
| `AA-AC-005`, `AA-AC-006` | new |
| `AA-AC-007` to `AA-AC-009` | `AN-AC-005` to `AN-AC-007` |
| `AA-AC-010` | `AN-AC-008` |
| `AA-AC-011` to `AA-AC-013` | `AN-AC-010` to `AN-AC-012` |
| `AA-AC-014` | new |
| `AA-AC-015` to `AA-AC-017` | `AN-AC-013` to `AN-AC-015` |
| `AA-AC-018` | new |
| `AA-AC-019`, `AA-AC-020` | `AN-AC-016`, `AN-AC-017` |
| `AA-AC-021` | new |
| `AA-AC-022` | `AN-AC-018` |
| `AA-AC-023` | new |
| `AA-AC-024` | `AN-AC-009` |
| `AA-AC-025`, `AA-AC-026` | `AN-AC-019`, `AN-AC-020` |
| `AA-AC-027` | new |
| `AA-AC-028` to `AA-AC-031` | `AN-AC-021` to `AN-AC-024` |
| `AA-AC-032`, `AA-AC-033` | `AN-AC-025`, `AN-AC-026` |
| `AA-AC-034` | new |
| `AA-AC-035` to `AA-AC-040` | `AN-AC-027` to `AN-AC-031`, with `AN-AC-032` at `AA-AC-036` |
| `AA-AC-041` | new |
| `AA-AC-042` | `AN-AC-033` |
| `AA-AC-043`, `AA-AC-044` | new |
| `AA-AC-045` | new |
| `AA-AC-046` | `AN-AC-034` |
| `AA-AC-047` | new |
| `AA-AC-048` to `AA-AC-053` | `AN-AC-035` to `AN-AC-040` |
| `AA-AC-054` | new |
| `AA-AC-055` to `AA-AC-058` | `AN-AC-041` to `AN-AC-044` |
| `AA-AC-059`, `AA-AC-060` | new |
| `AA-AC-061` | `AN-AC-045` |
| `AA-AC-062` | new |
| `AA-AC-063` | `AN-AC-046` and `AN-AC-053` |
| `AA-AC-064` to `AA-AC-069` | `AN-AC-047` to `AN-AC-052` |
| `AA-AC-070` | new |
| `AA-AC-071` to `AA-AC-074` | `AN-AC-054` to `AN-AC-057` |
| `AA-AC-075` | new |
| `AA-AC-076` | `AN-AC-058` |
| `AA-AC-077` | new |
| `AA-AC-078` | `AN-AC-059` |
| `AA-AC-079` | `AN-AC-066` |
| `AA-AC-080` to `AA-AC-085` | `AN-AC-060` to `AN-AC-065` |
| `AA-AC-086` | new |
| `AA-AC-087` to `AA-AC-090` | `AN-AC-080`, plus the variants of [calculations.md](calculations.md) section 8.5 |
| `AA-AC-091` | `AN-AC-081` |
| `AA-AC-092` | `AN-AC-082` |
| `AA-AC-093`, `AA-AC-094` | new |
| `AA-AC-095` | `AN-AC-084` |
| `AA-AC-096` | `AN-AC-079` |
| `AA-AC-097` | `AN-AC-083` |
| `AA-AC-098`, `AA-AC-099` | new |
| `AA-AC-100`, `AA-AC-101` | `AN-AC-067`, `AN-AC-068` |
| `AA-AC-102`, `AA-AC-103` | `AN-AC-070` and the worked example of section 9.4 |
| `AA-AC-104`, `AA-AC-105` | `AN-AC-071`, `AN-AC-073` |
| `AA-AC-106` to `AA-AC-113` | new |
| `AA-AC-114`, `AA-AC-115` | `AN-AC-074`, `AN-AC-075` |
| `AA-AC-116` | `AN-AC-072` |
| `AA-AC-117`, `AA-AC-118` | new |
| `AA-AC-119`, `AA-AC-120` | `AN-AC-076`, `AN-AC-077` |
| `AA-AC-121` to `AA-AC-125` | `AN-AC-085` to `AN-AC-089` |
| `AA-AC-126`, `AA-AC-127` | new |
| `AA-AC-128` | `AN-AC-090` |
| `AA-AC-129` to `AA-AC-131` | `AN-AC-093` to `AN-AC-095` |
| `AA-AC-132` | new |
| `AA-AC-133` | `AN-AC-069` |
| `AA-AC-134` to `AA-AC-137` | new |
| `AA-AC-138` to `AA-AC-141` | `AN-AC-098` to `AN-AC-100` |
| `AA-AC-142`, `AA-AC-143` | new |
| `AA-AC-144` | `AN-AC-101` and `AN-AC-113` |
| `AA-AC-145`, `AA-AC-146` | `AN-AC-096`, `AN-AC-097` |
| `AA-AC-147` | new |
| `AA-AC-148`, `AA-AC-149` | `AN-AC-102`, `AN-AC-103` |
| `AA-AC-150` to `AA-AC-152` | new |
| `AA-AC-153`, `AA-AC-154` | `AN-AC-104`, `AN-AC-105` |
| `AA-AC-155`, `AA-AC-156` | new |
| `AA-AC-157`, `AA-AC-158` | `AN-AC-106`, `AN-AC-107` |
| `AA-AC-159` | `AN-AC-108` |
| `AA-AC-160`, `AA-AC-161` | new |
| `AA-AC-162`, `AA-AC-163` | `AN-AC-110`, `AN-AC-111` |
| `AA-AC-164`, `AA-AC-165` | new |
| `AA-AC-166` | `AN-AC-112` |
| `AA-AC-167` | `AN-AC-078` |
| `AA-AC-168` to `AA-AC-174` | new |
| `AA-AC-175` | `AN-AC-114` |
| `AA-AC-176` | `AN-AC-115` |
| `AA-AC-177` | `AN-AC-116` |
| `AA-AC-178` | `AN-AC-117` |
| `AA-AC-179` to `AA-AC-187` | new |
| `AA-AC-188`, `AA-AC-189` | `AN-AC-118`, `AN-AC-119` |
| `AA-AC-190`, `AA-AC-191` | `AN-AC-120` and `AN-AC-109` |
| `AA-AC-192` to `AA-AC-196` | new |

---

## 21. Reconciliation notes

1. **A single source, corrected and extended.** Only one of the two drafts of this folder carried an
   acceptance-criteria document, with one hundred and twenty scenarios. Every one of them is kept,
   renumbered into the single `AA-AC-nnn` scheme of this file and mapped in section 20. Seventy-six
   scenarios are new: they cover the worked examples the other draft carried in its calculations
   document — the closing-line rule, the cross-plan split, the rounding passes, the merge branches,
   the balance over a date range, the profitability classification, the discount weighting, the
   redistribution and the project profitability sections — and the parts of the source neither draft
   exercised.
2. **Column names.** The former draft invented the column names `plan_account_<identifier>`,
   `plan_account_<identifier>_level_1` and `project_plan_account`. The reproduced names are
   `x_plan<plan identifier>_id`, `x_plan<plan identifier>_id_<depth>` and `account_id`; the
   scenarios use the notation column(*P*) so that a reader is never asked to guess which is which.
3. **The base-plan parameter message.** The former draft asserted the refusal text "The value for
   analytic.project_plan must be the identifier of a valid analytic plan that is not a subplan". The
   emitted text is "The value for the key must be the ID to a valid analytic plan that is not a
   subplan", with the key as its placeholder; `AA-AC-025` asserts the emitted text.
4. **The archived-account refusal.** The former draft asserted only that the message contains the
   words "archived analytic account". `AA-AC-096` asserts the full text with its placeholder, as
   `AA-078` states it.
5. **The grouping aggregate message.** The former draft asserted a message of the form "does not
   accept <aggregate> as aggregate". `AA-AC-166` asserts both reproduced texts, including the one
   raised for a model that declares no owner.
6. **The split scenarios.** The former draft's four split scenarios listed the resulting lines
   without saying which line was rewritten and which was created, and gave no notification count for
   three of them. `AA-AC-078` to `AA-AC-086` state both, because the identifiers of the surviving
   lines are observable.
