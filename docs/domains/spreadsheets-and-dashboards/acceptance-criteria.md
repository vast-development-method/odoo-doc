# Acceptance criteria

One hundred and twenty-five numbered scenarios. Each states concrete starting records, a concrete operation with concrete inputs, and the exact resulting records, values and states. A rebuild that satisfies every one of them behaves as this domain behaves.

Unless a scenario says otherwise:

- the installation holds one company, "Main Company", whose currency is the euro with two decimal places and the symbol after the number, and whose fiscal year ends on the thirty-first of December;
- the acting user is an internal user named Raoul who holds one further access group named "test group" and the data-export group;
- an administrator is a user holding the dashboard administrator group;
- "now" is 15 March 2026 at 14:30 in the reader's own time zone.

## 1. Configuring dashboard groups and dashboards

### AC-01 — A dashboard created with no audience takes the internal-user group

**Given** a Dashboard Group named "a group".
**When** an administrator creates a Spreadsheet Dashboard with the name "a dashboard" and that group, and nothing else.
**Then** the dashboard's `group_ids` holds exactly the internal-user group; `is_published` is true; `company_ids` is empty; `sequence` is 0; `main_data_model_ids` is empty; `sample_dashboard_file_path` is empty; and `spreadsheet_data` parses to a document with one sheet whose `id` is `sheet1`, whose `name` is the word "Sheet1" translated into the administrator's language, a `settings` section holding the administrator's locale, and `revisionId` set to `START_REVISION`.

### AC-02 — A dashboard without a name is refused

**Given** the same group.
**When** an administrator tries to create a Spreadsheet Dashboard with that group and no name.
**Then** the creation is refused by rule SD-004 with the platform's required-field refusal naming the field labelled "Name", and no record is created.

### AC-03 — A dashboard without a group is refused

**Given** nothing.
**When** an administrator tries to create a Spreadsheet Dashboard with the name "orphan" and no dashboard group.
**Then** the creation is refused by rule SD-005 with the platform's required-field refusal naming the field labelled "Dashboard Group", and no record is created.

### AC-04 — Duplicating a dashboard names the copy

**Given** a Spreadsheet Dashboard named "a dashboard" in a group named "a group", with `main_data_model_ids` holding Sales Order.
**When** an administrator duplicates it without supplying a name.
**Then** a second dashboard exists whose name is "a dashboard (copy)", whose group is "a group", whose workbook is a copy of the original's, and whose `main_data_model_ids` is **empty**.

### AC-05 — Duplicating with a name supplied

**Given** the same dashboard.
**When** an administrator duplicates it supplying the name "a copy".
**Then** the copy's name is "a copy", not "a copy (copy)".

### AC-06 — Duplicating a copy appends the suffix again

**Given** a dashboard named "a dashboard (copy)".
**When** an administrator duplicates it without supplying a name.
**Then** the new copy's name is "a dashboard (copy) (copy)".

### AC-07 — A shipped dashboard group cannot be deleted

**Given** a Dashboard Group named "a_group" that carries an external identifier whose module is the dashboards package.
**When** an administrator deletes it.
**Then** the deletion is refused by rule SD-009 with the message "You cannot delete %s as it is used in another module.", the placeholder being "a_group", and the group still exists.

### AC-08 — A group holding a dashboard cannot be deleted

**Given** a Dashboard Group with no external identifier holding one Spreadsheet Dashboard.
**When** an administrator deletes the group.
**Then** the deletion is refused by rule SD-010: storage reports that other records depend on this one, and the group still exists.

### AC-09 — Deleting a dashboard cascades to its shares

**Given** a Spreadsheet Dashboard with two Dashboard Shares.
**When** an administrator deletes the dashboard.
**Then** both shares are deleted, the workbook attachment of the dashboard is deleted, and the two shared addresses answer not found.

### AC-10 — An ordinary user cannot create a dashboard

**Given** Raoul, who is an internal user without the dashboard administrator group.
**When** Raoul tries to create a Spreadsheet Dashboard.
**Then** the creation is refused by rule SD-014 with the platform's access refusal naming the entity and the creation operation.

## 2. Publication and the favourite mark

### AC-11 — Unpublishing removes a dashboard from its group's published sub-list

**Given** a Dashboard Group named "Dashboard group" holding one Spreadsheet Dashboard, whose `published_dashboard_ids` therefore holds that dashboard.
**When** an administrator sets `is_published` to false.
**Then** `published_dashboard_ids` is empty, `dashboard_ids` still holds the dashboard, and the group no longer appears in the workspace sidebar.

### AC-12 — Republishing puts it back

**Given** the same group with its dashboard unpublished.
**When** an administrator sets `is_published` to true.
**Then** `published_dashboard_ids` holds the dashboard again and the group reappears in the sidebar.

### AC-13 — Starring and unstarring

**Given** a Spreadsheet Dashboard visible to Raoul, whose `favorite_user_ids` is empty.
**When** Raoul calls the favourite operation on that dashboard.
**Then** `favorite_user_ids` holds Raoul, `is_favorite` reads true for Raoul, and the sidebar shows a "FAVORITES" section holding that dashboard.
**When** Raoul calls the operation again.
**Then** `favorite_user_ids` is empty, `is_favorite` reads false for Raoul, and the "FAVORITES" section disappears.

### AC-14 — The favourite mark is per reader

**Given** a Spreadsheet Dashboard starred by Raoul.
**When** a second internal user named Alice reads it.
**Then** `is_favorite` reads false for Alice while it still reads true for Raoul, and Alice's sidebar has no "FAVORITES" section.

### AC-15 — The favourite operation refuses more than one record

**Given** two Spreadsheet Dashboards.
**When** the favourite operation is called on both at once.
**Then** the call is refused by rule SD-020 with the platform's single-record refusal, and neither dashboard's `favorite_user_ids` changes.

## 3. Visibility

### AC-16 — The audience rule hides a dashboard

**Given** a Spreadsheet Dashboard whose `group_ids` holds only an access group that Raoul does not hold.
**When** Raoul lists dashboards.
**Then** the dashboard is absent from the answer, with no message, by rule SD-015.

### AC-17 — An administrator sees it anyway

**Given** the same dashboard.
**When** an administrator lists dashboards.
**Then** the dashboard is present, by rule SD-017.

### AC-18 — The company rule applies to administrators too

**Given** a second company "Branch", and a Spreadsheet Dashboard whose `company_ids` holds only "Branch".
**When** an administrator whose only active company is "Main Company" lists dashboards.
**Then** the dashboard is absent, by rule SD-016, even though rule SD-017 matched it.
**When** the administrator activates "Branch" as well.
**Then** the dashboard is present.

### AC-19 — A dashboard with no company is visible in every company

**Given** a Spreadsheet Dashboard whose `company_ids` is empty.
**When** any reader in any active-company selection lists dashboards.
**Then** the dashboard is present, because the company rule accepts a dashboard with no company at all.

### AC-20 — A share is invisible to another user

**Given** a Dashboard Share created by the administrator.
**When** Raoul reads that share's `access_token`.
**Then** the read is refused by the platform's access refusal, by rule SD-018.

### AC-21 — Every internal user may create a share

**Given** a Spreadsheet Dashboard that Raoul may read.
**When** Raoul creates a Dashboard Share for it.
**Then** the share is created, its creating user is Raoul, and Raoul may read it.

## 4. Reading a dashboard

### AC-22 — The reader's locale replaces the stored one

**Given** a Spreadsheet Dashboard whose stored workbook carries any locale, and Raoul whose language is `en_US`.
**When** Raoul requests the dashboard reading route for it.
**Then** the answer's snapshot has `settings` holding a `locale` whose `code` is `en_US`, and `revisions` is an empty list.
**When** Raoul's language is changed to `fr_FR` and the route is requested again.
**Then** the locale's `code` is `fr_FR` and `revisions` is still empty. The stored dashboard has not changed.

### AC-23 — The default currency comes from the active company

**Given** the same dashboard and "Main Company" whose currency is the euro.
**When** Raoul requests the reading route.
**Then** the answer's `default_currency` holds the code `EUR`, the symbol `€`, two decimal places and the position `after`.

### AC-24 — The translation namespace names the shipping package

**Given** a Spreadsheet Dashboard that carries an external-identifier record whose module is the dashboards package.
**When** Raoul requests the reading route.
**Then** the answer's `translation_namespace` is the dashboards package's technical name.
**And Given** a dashboard with no external-identifier record.
**Then** the answer's `translation_namespace` is empty.

### AC-25 — A sample replaces an empty dashboard

**Given** a Spreadsheet Dashboard whose `sample_dashboard_file_path` names a shipped sample document holding one section, `sheets`, with an empty list; whose `main_data_model_ids` holds the bank entity; and an installation in which every bank record has been archived so that the entity holds no active record.
**When** Raoul requests the reading route.
**Then** the answer holds exactly two keys: `is_sample` set to true, and `snapshot` holding the sample document — a `sheets` key with an empty list. It holds no `revisions`, no `default_currency` and no `translation_namespace`.

### AC-26 — A measured entity the reader cannot read does not force a sample

**Given** a Spreadsheet Dashboard whose `sample_dashboard_file_path` is set and whose `main_data_model_ids` holds an entity Raoul may not read but which holds records.
**When** Raoul requests the reading route.
**Then** the answer carries no `is_sample` key, `revisions` is an empty list, and the live workbook is served: the count was taken with elevated rights and found records.

### AC-27 — No measured entity means never a sample

**Given** a Spreadsheet Dashboard with a sample path and an empty `main_data_model_ids`.
**When** Raoul requests the reading route.
**Then** the live workbook is served.

### AC-28 — A missing sample file falls back to the live workbook

**Given** a Spreadsheet Dashboard whose `sample_dashboard_file_path` names a document that does not exist, and one measured entity holding no record.
**When** Raoul requests the reading route.
**Then** the live workbook is served, with no message and no failure.

### AC-29 — An unreadable dashboard is not found

**Given** a Spreadsheet Dashboard outside Raoul's audience.
**When** Raoul requests its reading route.
**Then** the answer is not found, by rule SD-026.

## 5. Sharing

### AC-30 — Creating a share answers its address

**Given** a Spreadsheet Dashboard whose identifier is 7, and one workbook-file part whose path is `[Content_Types].xml`.
**When** the sharing operation is called with the dashboard identifier, the frozen workbook as text and that one part.
**Then** exactly one Dashboard Share exists for that dashboard; its `excel_export` is not empty; its `access_token` is a freshly generated universally unique identifier; and the answer equals its `full_url`, which is the installation's base web address followed by `/dashboard/share/`, the share's identifier, a slash and the token.

### AC-31 — The public page opens with the right token

**Given** the share of AC-30.
**When** anybody requests `/dashboard/share/<identifier>/<token>` with the stored token.
**Then** the page is served successfully, showing the dashboard's name and the sentence "Frozen and copied on" followed by the share's creation moment.

### AC-32 — The public page refuses a wrong token

**Given** the same share.
**When** anybody requests the page with the token `a-random-token`.
**Then** the request is forbidden, with the message "You don't have access to this dashboard. ", by rule SD-022.

### AC-33 — The public data route serves the stored workbook exactly

**Given** a Dashboard Share whose `spreadsheet_data` was set to the dashboard's own workbook text.
**When** anybody requests `/dashboard/data/<identifier>/<token>` with the stored token.
**Then** the answer is that exact document, unchanged: no locale substitution, no default currency, no translation namespace, no revisions.

### AC-34 — The public data route refuses a wrong token

**Given** the same share.
**When** anybody requests the data route with `a-random-token`.
**Then** the request is forbidden, by rule SD-022.

### AC-35 — Revoking the sharing user's access closes the address

**Given** a Spreadsheet Dashboard whose audience is the access group "test group", and a Dashboard Share created by Raoul, who holds that group.
**When** anybody requests the data route with the right token.
**Then** the request succeeds.
**When** Raoul is removed from "test group" and the same request is repeated.
**Then** the request is forbidden, by rule SD-022. Nothing about the share record has changed.

### AC-36 — Downloading the workbook file

**Given** a Dashboard Share whose `excel_export` holds the four bytes of the text `test`, and a signed-in user named Alex who holds the data-export group.
**When** Alex requests `/dashboard/download/<identifier>/<token>` with the right token.
**Then** the answer is the stored content, exactly those four bytes, named after the dashboard.

### AC-37 — Downloading without the export right is refused

**Given** the same share, and Alex from whom the data-export group has been removed.
**When** Alex requests the download route with the right token.
**Then** the request is refused with the message "You don't have the rights to export data. Please contact an Administrator.", by rule SD-023.

### AC-38 — Downloading with a wrong token is forbidden

**Given** the same share and a signed-in Alex holding the export group.
**When** Alex requests the download route with `a-random-token`.
**Then** the request is forbidden, by rule SD-022.

### AC-39 — Downloading after the sharing user lost access

**Given** a share created by Raoul over a dashboard whose audience is "test group", and a signed-in Alex holding the export group.
**When** Alex requests the download route with the right token.
**Then** it succeeds.
**When** Raoul is removed from "test group" and Alex repeats the request.
**Then** it is forbidden, by rule SD-022: the check is on the sharing user's rights, not on Alex's.

### AC-40 — Sharing twice with nothing in between reuses the address

**Given** an open dashboard that has just been shared once in this reading session.
**When** the reader opens the share control again without changing anything.
**Then** no request is sent, no second Dashboard Share is created, and the address shown is the one obtained the first time.

### AC-41 — Changing a filter and sharing again creates a second share

**Given** the same session, and a date filter set to `year` 2025.
**When** the reader sets the filter to `year` 2026 and opens the share control again.
**Then** the cells of the appended "Active Filters" sheet differ from those recorded at the first share, so a second Dashboard Share is created with its own token and its own address, and an export log entry is written for the operation `freeze`.

## 6. Workbook validation

### AC-42 — A workbook that does not decode is refused

**Given** any record carrying the Spreadsheet Document mixin.
**When** a value is written to `spreadsheet_binary_data` whose decoded content is not a well-formed structured document.
**Then** the write is refused by rule SD-001 with the message "Uh-oh! Looks like the spreadsheet file contains invalid data.", and the stored workbook is unchanged.

### AC-43 — A packaged workbook file is accepted

**Given** the same record.
**When** a value is written whose decoded content declares the key `[Content_Types].xml`.
**Then** the write is accepted and no reference walk is performed, by rule SD-003.

### AC-44 — A workbook naming a missing entity is reported

**Given** an installation executing its automated test suite, and a workbook holding one list element whose `model` is `nonexistent.model`, stored on a record whose display name is "a dashboard".
**When** the workbook is written.
**Then** the write is refused by rule SD-002 with the message "Uh-oh! Looks like the spreadsheet file contains invalid data.", two line breaks, and the single finding `- model 'nonexistent.model' used in 'a dashboard' does not exist`.

### AC-45 — A workbook naming a missing field is reported

**Given** the same installation, and a workbook holding one list element over Sales Order whose columns include `no_such_field`, stored on a record whose display name is "a dashboard".
**When** the workbook is written.
**Then** the write is refused with the finding `- field 'no_such_field' used in spreadsheet 'a dashboard' does not exist on model 'sale.order'`.

## 7. Account movement formulas

The scenarios of this section share these starting records unless they say otherwise. "Main Company" holds the account `sp1234566`, of type income, and the account `sp1234577`, of type expense; neither includes the initial balance. One entry dated 2 April 2022 debits `sp1234566` by 500.00 and credits `sp1234577` by 500.00. A second company holds the accounts `sp99887755` and `sp99887766` with an entry dated 2 February 2022 of 1500.00.

### AC-46 — An exact code, one year

**When** the total-debit and total-credit request is made with the period `year` 2022, the codes list holding `sp1234566`, no company and unposted entries included.
**Then** the answer is a debit of 500.00 and a credit of 0.00, so `ODOO.BALANCE` shows 500.00.

### AC-47 — Two codes

**When** the same request is made with the codes `sp1234566` and `sp1234577`.
**Then** the answer is a debit of 500.00 and a credit of 500.00, so `ODOO.BALANCE` shows 0.00.

### AC-48 — A prefix, and a repeated prefix

**When** the request is made with the single code `sp1234`.
**Then** the answer is a debit of 500.00 and a credit of 500.00.
**When** it is made with the codes `sp1234` and `sp1234`.
**Then** the answer is the same: a debit of 500.00 and a credit of 500.00. Duplicate prefixes cannot double-count.

### AC-49 — An empty code, an unmatched code, no code

**When** the request is made with the codes list holding one empty text, posted entries only.
**Then** the answer is a debit of 0.00 and a credit of 0.00.
**When** it is made with the code `10000000000`, which matches no account.
**Then** the answer is 0.00 and 0.00.
**When** it is made with an empty codes list.
**Then** the answer is 0.00 and 0.00: the total-debit family has no payable-and-receivable fallback.

### AC-50 — Companies are separate

**When** one round trip carries two requests, the first with the code `sp1234566` and the first company, the second with the code `sp99887755` and the second company, both for `year` 2022 with unposted entries included.
**Then** the answers are, in that order, a debit of 500.00 with a credit of 0.00, and a debit of 1500.00 with a credit of 0.00. Reversing the order of the two requests reverses the order of the two answers and changes neither.

### AC-51 — A year excludes later years

**Given** a second entry dated 2 April 2022 debiting `sp1234566` by 1000.00.
**When** the request is made for `year` 2021 with the code `sp1234566`.
**Then** the answer is a debit of 0.00 and a credit of 0.00.

### AC-52 — A shifted fiscal year

**Given** "Main Company" changed so that its fiscal year ends on the third of February, and an added entry dated 2 January 2023 debiting `sp1234566` by 1000.00.
**When** the request is made for `year` 2022 with the code `sp1234566` and unposted entries included.
**Then** the window is 4 February 2022 to 3 February 2023 and the answer is a debit of 1500.00 — 500.00 from 2 April 2022 plus 1000.00 from 2 January 2023 — and a credit of 0.00.

### AC-53 — Quarter, month and day on a profit-and-loss account

**Given** an added entry dated 2 July 2022 debiting and crediting `sp1234566` by 777.00, with the account still of type income.
**When** the request is made for `quarter` 3 of 2022 with the code `sp1234566`.
**Then** the answer is a debit of 777.00 and a credit of 777.00: a monthly and a quarterly window both start after 2 April 2022, so the 500.00 debit of that date is outside them.
**When** the request is made for `month` 7 of 2022 with an entry of 666.00 both ways dated 2 July 2022 instead.
**Then** the answer is a debit of 666.00 and a credit of 666.00.
**When** the request is made for `day` 2 July 2022 with an entry of 555.00 both ways dated that day instead.
**Then** the answer is a debit of 1055.00 and a credit of 555.00: a daily window starts at the fiscal year start of 1 January 2022, so it also catches the 500.00 debit of 2 April 2022.

### AC-54 — The same three on a balance-sheet account

**Given** `sp1234566` changed to the type `asset_receivable`, so that it includes the initial balance, and an added entry dated 2 July 2022 debiting and crediting it by 777.00.
**When** the request is made for `quarter` 3 of 2022, then `month` 7 of 2022, then `day` 2 July 2022, in each case with the code `sp1234566`.
**Then** every one of the three answers a debit of 1277.00 — 777.00 plus the cumulated 500.00 of 2 April 2022 — and a credit of 777.00.
**When** the request is made for `year` 2025 with no entry after 2022.
**Then** the answer is a debit of 500.00 and a credit of 0.00: a balance-sheet account cumulates forward indefinitely.

### AC-55 — A mixed selection of a balance-sheet and a profit-and-loss account

**Given** `sp1234566` of type `asset_receivable` and an added entry dated 2 July 2000 debiting `sp1234566` by 555.00 and crediting `sp1234577` by 555.00.
**When** the request is made for `year` 2022 with the codes `sp1234566` and `sp1234577`.
**Then** the answer is a debit of 1055.00 and a credit of 500.00: the 555.00 debit of the year 2000 is cumulated because it sits on a balance-sheet account, and the 555.00 credit of the same entry is excluded because it sits on a profit-and-loss account outside the window.

### AC-56 — Cancelled entries are never counted

**Given** `sp1234566` of type `asset_receivable` holding the posted 500.00 debit, and an added cancelled entry dated 2 April 2022 of 10000000.00 both ways.
**When** the request is made for `year` 2022 with posted entries only.
**Then** the answer is a debit of 0.00 and a credit of 0.00, because the 500.00 entry is a draft.
**When** the same request is made with unposted entries included.
**Then** the answer is a debit of 500.00 and a credit of 0.00: the cancelled entry is still excluded.

### AC-57 — Posted and unposted together

**Given** `sp1234566` of type `asset_receivable`, the original 500.00 entry left as a draft, and a second entry of 888.00 both ways dated 2 April 2022 which is then posted.
**When** the request is made for `year` 2022 with posted entries only.
**Then** the answer is a debit of 888.00 and a credit of 888.00.
**When** the same request includes unposted entries.
**Then** the answer is a debit of 1388.00 and a credit of 888.00.

### AC-58 — The first day of a shifted fiscal year

**Given** "Main Company" whose fiscal year ends on the third of February, and entries of 111.00 dated 3 February 2022 and 423.00 dated 4 February 2022, both debiting and crediting `sp1234566`.
**When** the request is made for `day` 4 February 2022 with the code `sp1234566` and unposted entries included.
**Then** the window is 4 February 2022 to 4 February 2022 and the answer is a debit of 423.00 and a credit of 423.00. The entry of the third of February belongs to the previous fiscal year.

### AC-59 — The cell audit action

**Given** the starting records of this section and the account `sp1234566` belonging to "Main Company".
**When** the audit operation is called with the period `year` 2022, the code `sp1234566`, that company and unposted entries included.
**Then** the answer is a window action over Journal Item, list presentation only, current target, titled "Cell Audit", whose record selection is the conjunction of: the journal item's account is `sp1234566`; the period disjunction with the last date 31 December 2022 and the first date 1 January 2022; the journal item's company is "Main Company"; and the journal entry's state is not `cancel`.

### AC-60 — The cell audit action with no code

**When** the same operation is called with a codes list holding one empty text and no company.
**Then** the selection's account condition names every account of "Main Company" whose type is `liability_payable` or `asset_receivable`, and the rest of the selection is as in AC-59.

### AC-61 — An empty request list

**When** any of the four fetching operations is called with an empty list of requests.
**Then** the answer is an empty list and no query is made.

## 8. Residual, partner balance and tagged balance

### AC-62 — Residual with and without codes

**Given** the records of [`calculations.md`](calculations.md) §8.2: Entry A of 2 February 2022 debiting the receivable account by 1500.00, Entry B of 2 February 2023 crediting the payable account by 2500.00, a customer payment of 150.00 dated 10 February 2022 reconciled against Entry A, and an outbound customer payment of 250.00 dated 10 February 2023, all posted.
**When** the residual operation is called for `year` 2023 with no code and posted entries only, and again with the receivable account's code.
**Then** the two answers are −900.00 and 1600.00.

### AC-63 — Residual by quarter and by day

**When** the residual operation is called for `quarter` 4 of 2022 and for `quarter` 1 of 2023, both with no code and posted entries only.
**Then** the answers are 1350.00 and −900.00.
**When** it is called for `day` 1 February 2022 and for `day` 2 February 2022.
**Then** the answers are 0.00 and 1350.00.

### AC-64 — Partner balance

**Given** Entry A of 2 February 2022 with partner A, debiting the receivable account by 1500.00 and crediting revenue; Entry B of 2 February 2023 with partner A, debiting expense by 2500.00 and crediting the payable account; and a draft copy of Entry B carrying partner B. Entries A and B are posted.
**When** the partner-balance operation is called for partner A, `year` 2023, no code, posted entries only, and again with the receivable account's code.
**Then** the two answers are −1000.00 and 1500.00.
**When** it is called for partner B with the same period and posted entries only.
**Then** the answer is 0.00, because partner B's only entry is a draft.

### AC-65 — An empty partner list answers zero without querying

**When** the partner-balance operation is called with a partner list that is empty once empty entries are dropped.
**Then** the answer is 0.00, no ledger query is made, and the answer is **not** the balance over every partner.

### AC-66 — Tagged balance

**Given** tag one carried by the receivable account and by the revenue account, tag two carried by the expense account; a posted entry dated 1 January 2025 debiting receivable by 100.00, debiting revenue by 50.00 and crediting expense by 150.00; and a draft entry dated 1 January 2024 debiting receivable by 100.00 and crediting expense by 100.00.
**When** four requests are made in one round trip: `year` 2025 with tag one and posted only; `year` 2025 with tags one and two and posted only; `year` 2024 with tag one and posted only; `year` 2024 with tag one and unposted included.
**Then** the four answers, in order, are 150.00, 0.00, 0.00 and 100.00.

### AC-67 — An empty tag list answers zero without querying

**When** the tagged-balance operation is called with a tag list that is empty once empty entries are dropped.
**Then** the answer is 0.00 and no ledger query is made.

## 9. Fiscal year and account groups

### AC-68 — Fiscal year boundaries around the end date

**Given** "Main Company" whose fiscal year ends on the third of February.
**When** the fiscal-dates operation is called for the days 5 March 2020, 3 February 2020 and 4 February 2020.
**Then** the three answers are, in order: start 4 February 2020 with end 3 February 2021; start 4 February 2019 with end 3 February 2020; and start 4 February 2020 with end 3 February 2021.

### AC-69 — Fiscal year with a named company, and with an unknown one

**Given** "Main Company" whose fiscal year ends on the seventh of June and a second company "Test company" whose fiscal year ends on the third of February.
**When** the operation is called with two requests for the day 4 February 2020, the first naming "Test company" and the second naming no company.
**Then** the answers are, in order, start 4 February 2020 with end 3 February 2021, and start 8 June 2019 with end 7 June 2020.
**When** the first request names the company identifier 999, which exists nowhere.
**Then** the first answer is nothing and the second is unchanged. Reversing the order of the two requests reverses the order of the two answers.

### AC-70 — Account group codes

**Given** "Main Company" holding exactly one account of type `income_other`, whose code is `450000`.
**When** the account-group operation is called with an empty list of types.
**Then** the answer is an empty list.
**When** it is called with the single type `income_other`.
**Then** the answer is one entry holding the single code `450000`.
**When** every account of that type is deleted and the call is repeated.
**Then** the answer is one entry holding an empty list.
**When** it is called with a value that is not an account type.
**Then** the answer is one entry holding an empty list.
**When** two further accounts of that type are created with the codes `123` and `789` and the call is repeated.
**Then** the answer is one entry holding the three codes `123`, `450000` and `789`.
**When** it is called with the two types `income` and `income_other`, and again with them reversed.
**Then** each type's codes come back in the position of that type in the request.

## 10. Currency

### AC-71 — Currency rates without and with a date

**Given** a company whose currency is the euro, current rates of 1.5 for the dollar and 1.2 for the Canadian dollar, and rates dated 11 November 2021 of 1.8 for the dollar and 1.9 for the Canadian dollar.
**When** the rate operation is called for the pairs dollar to euro, euro to dollar and dollar to Canadian dollar, with no date.
**Then** the three answers are 0.6666666667, 1.5000000000 and 0.8000000000.
**When** the same three are called for the date 11 November 2021.
**Then** the answers are 0.5555555556, 1.8000000000 and 1.0555555556.

### AC-72 — An unknown or empty currency code

**When** the rate operation is called with a source code of `INVALID`, or a target code of `INVALID`, or an empty source code, or an empty target code.
**Then** every one of the four answers is nothing, and a cell using the rate formula shows "Currency rate unavailable." by rule SD-038.

### AC-73 — The company currency structure

**Given** a company whose currency is the euro and a second whose currency is the dollar.
**When** the company-currency operation is called with no company, then with the second company's identifier, then with the identifier 123456 which exists nowhere.
**Then** the answers are: code `EUR`, symbol `€`, two decimal places, position `after`; code `USD`, symbol `$`, two decimal places, position `before`; and nothing. A cell using the third answer shows "Currency not available for this company." by rule SD-039.

### AC-74 — The number format built from a currency

**Given** the two currencies above.
**When** a formula whose company is the euro company produces a number, and a formula whose company is the dollar company produces one.
**Then** the two cells carry the formats `#,##0.00[$€]` and `[$$]#,##0.00`.

## 11. Periods, filters and offsets

### AC-75 — Reading a period argument

**When** a period argument holds, in turn, `Q1/2022`, `12/2022`, `2022`, the date serial number of 21 December 2022, `2999` and `3000`.
**Then** the six readings are: `quarter` 2022 quarter 1; `month` 2022 month 12; `year` 2022; `day` 21 December 2022; `year` 2999; and `day` 18 March 1908.

### AC-76 — An unreadable period

**When** a period argument holds a value that cannot be read as any of the four shapes.
**Then** the cell shows `'%s' is not a valid period. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022".` with the placeholder replaced by the value given, by rule SD-047.

### AC-77 — An offset below the lower bound

**Given** a balance formula whose period is `2022`.
**When** the offset is −200.
**Then** the requested year is 1822 and the cell shows "%s is not a valid year." with the placeholder replaced by 1822, by rule SD-048, and no request is made.
**When** the offset is −122.
**Then** the requested year is 1900, the check passes, and the request is made.

### AC-78 — Relative filter periods

**Given** now = 15 March 2026 at 14:30, a date filter, and an element whose matching has offset zero over a date field.
**When** the filter is set to each of the nine relative periods in turn.
**Then** the nine windows are exactly those of [`calculations.md`](calculations.md) §17.2 — for instance `last_7_days` gives 9 March 2026 to 15 March 2026 inclusive, and `last_12_months` gives 1 March 2025 to 28 February 2026.

### AC-79 — Two elements, one filter, two periods

**Given** a workbook with one date filter, a pivot whose matching over the invoice date has offset 0, and a second pivot whose matching over the same field has offset −1.
**When** the filter is set to `month` 2026-03.
**Then** the first pivot is narrowed to invoice dates from 1 March 2026 to 31 March 2026 and the second to invoice dates from 1 February 2026 to 28 February 2026.

### AC-80 — Setting a filter to the value already in force

**Given** a date filter resolved to `year` 2026, whether by a set value or by its default.
**When** the reader sets it to `year` 2026 again.
**Then** the command is refused with the marker `NoChanges`, no element becomes stale, and nothing is reloaded.

### AC-81 — Clearing a filter suppresses its default

**Given** a relational filter whose default is the marker `current_user`.
**When** the reader opens the dashboard.
**Then** the filter resolves to the operator `in` with the reader's own user identifier, and the active-filter count includes it.
**When** the reader clears the filter.
**Then** the filter resolves to nothing, the default does not return, and the active-filter count no longer includes it.

### AC-82 — Two filters cannot share a label

**Given** a workbook holding a filter labelled "Period".
**When** an author adds a second filter labelled "Period".
**Then** the command is refused with the marker `DuplicatedFilterLabel` and no filter is added.

### AC-83 — A filter formula naming no filter

**Given** a workbook holding one filter labelled "Period".
**When** a cell holds the filter-value formula with the argument "Salesperson".
**Then** the cell shows `Filter "%(filter_name)s" not found` with the placeholder replaced by "Salesperson", by rule SD-037.

## 12. Lists, pivots and drill-through

### AC-84 — A list cell and its format

**Given** a data-bound list over Sales Order with the columns customer, confirmation moment, state, untaxed amount and tags; and a second row whose customer is "Deco Addict", whose confirmation moment is 14 March 2026 at 09:30:00, whose state is the stored option `sale` labelled "Sales Order", whose untaxed amount is 1234.5 in dollars, and whose tags are "Priority" and "Retail".
**When** the five value cells of that row are evaluated.
**Then** they hold, in order: "Deco Addict" with no format; 46095.3958333333 with the format built from the reader's date format, a space and the reader's time format; "Sales Order" with no format; 1234.5 with the format `[$$]#,##0.00`; and "Priority, Retail" with no format.

### AC-85 — A list header with and without a supplied label

**Given** the same list.
**When** a header cell names the field `partner_id` with no third argument.
**Then** the cell shows that field's own label on Sales Order.
**When** the same header cell carries the third argument "Client".
**Then** the cell shows "Client".

### AC-86 — A list formula naming an unknown element

**When** a cell holds the list-value formula with the element identifier `99`, which the workbook does not have.
**Then** the cell shows `There is no list with id "%s"` with the placeholder replaced by `99`, by rule SD-030.

### AC-87 — An empty field name and an unknown field

**When** a list formula's field-name argument evaluates to an empty text.
**Then** the cell shows "The field name should not be empty.", by rule SD-031.
**When** it names `no_such_field`.
**Then** the cell shows "The field %s does not exist or you do not have access to that field" with the placeholder replaced by `no_such_field`, by rule SD-032.

### AC-88 — A field type with no cell representation

**When** a list cell reads a field whose type is a free-form structured value.
**Then** the cell shows `Fields of type "%s" are not supported` with the placeholder replaced by `json`, by rule SD-033.

### AC-89 — Reading beyond the fetched window

**Given** a data-bound list whose cells have so far asked for twenty rows, which have been fetched.
**When** a cell asks for row 25.
**Then** that cell shows the loading marker, the source's high-water mark becomes 25, exactly one reload is scheduled for the next cycle however many cells asked, and after the reload the cell shows the value at row 25.

### AC-90 — Drilling from a list cell

**Given** the list of AC-84, whose element records no window action identifier.
**When** the reader opens the cell menu on the value cell of row 2 and chooses "See record".
**Then** a window action opens on Sales Order, form presentation, on the record at position 2, carrying the element's reading context. A middle click opens the same in a new window.

### AC-91 — Drilling from a pivot cell

**Given** a data-bound pivot over Sales Order grouped by salesperson in rows and by order month in columns, with the measure being the untaxed amount, and a global filter narrowing it to the current year.
**When** the reader opens the cell menu on the value cell at salesperson "Mitchell Admin" and month March 2026, and chooses "See records".
**Then** a window action opens on Sales Order, list then form, titled with Sales Order's own label, whose record selection is the conjunction of the pivot's own selection, the filter's condition for the current year, the salesperson being Mitchell Admin and the order month being March 2026.

### AC-92 — A weekly pivot value of the wrong shape

**When** a pivot formula addresses a group at the granularity `week` with the value `2026-W12`.
**Then** the cell shows "Week value must be a string in the format %(example)s, but received %(received_value)s instead." with the example built from the current year — in 2026, `"52/2026"` — and the received value `2026-W12`, by rule SD-036.

### AC-93 — A pivot dimension the entity does not have

**When** a pivot formula names the dimension `no_such_field`.
**Then** the cell shows "Field %s does not exist" with the placeholder replaced by `no_such_field`, by rule SD-035.

### AC-94 — A data source over a missing entity

**Given** a workbook whose list element names the entity `nonexistent.model`.
**When** any cell reads that element.
**Then** the cell shows `The model "%(model)s" does not exist.` with the placeholder replaced by `nonexistent.model`, by rule SD-034.

### AC-95 — Inserting a list

**Given** an editable workbook whose next list identifier is `1`, and a sheet with 10 columns and 10 rows.
**When** the author inserts a list of 20 rows and 4 columns at the cell in the third column and the third row, offering the identifier `1`.
**Then** the element is registered under `1`, the next identifier becomes `2`, the sheet grows to at least 6 columns and at least 23 rows, the third row holds four header formulas naming the element and each column's field, and the twenty rows below hold four value formulas each, naming the element, the row position from 1 to 20 and the field.

### AC-96 — Inserting a list with the wrong identifier

**Given** the same workbook whose next list identifier is `1`.
**When** the author offers the identifier `5`.
**Then** the command is refused with the marker `InvalidNextId` and nothing is written.
**When** the author offers the identifier `1` twice.
**Then** the second command is refused with the marker `ListIdDuplicated`.

### AC-97 — Renaming a list to an empty name

**When** a rename command supplies an empty name.
**Then** it is refused with the marker `EmptyName`.

### AC-98 — A chart linked to a menu with no action

**Given** a data-bound chart linked to a menu that resolves but that opens no action.
**When** the reader activates the chart's menu control.
**Then** nothing opens and the reader is shown "The menu linked to this chart doesn't have an corresponding action. Please link the chart to another menu.", by rule SD-060.

## 13. Export logging

### AC-99 — The copy threshold

**Given** an open dashboard with one loaded list over Sales Order.
**When** the reader copies the rectangle from the first column and first row to the twentieth column and twentieth row, which covers 20 × 20 = 400 cells.
**Then** nothing is logged.
**When** the reader copies one column more, 21 × 20 = 420 cells.
**Then** one entry is written.

### AC-100 — What a log entry contains

**Given** an acting user whose identifier is 7, a request coming from the network address `10.0.0.4`, and one loaded list over Sales Order with the columns `name` and `amount_total` and the record selection `[("state", "=", "sale")]`.
**When** the operation `freeze` is logged.
**Then** the entry written at informational level reads: `User 7 exported (freeze) spreadsheet data (model: sale.order with fields: [name,amount_total] with domain [("state", "=", "sale")]) from 10.0.0.4`.

### AC-101 — A grouped source adds its groupings

**Given** the same acting user and one loaded data-bound pivot over Sales Order with the measure field `amount_total`, the dimensions `user_id` and `date_order:month`, and no record selection.
**When** the operation `download` is logged.
**Then** the rendered line reads `model: sale.order with fields: [amount_total] grouped by [user_id,date_order:month]`.

### AC-102 — Descriptions that are dropped

**Given** one description naming an entity that is not installed and one description with an empty field list.
**When** either is logged, alone.
**Then** it renders to nothing, and because every description of the request was dropped, no entry is written at all.

### AC-103 — An unaccepted operation name

**When** the logging route is called with the operation name `share`.
**Then** nothing is written and the request answers normally, by rule SD-058.

## 14. The personal board

### AC-104 — Pinning a view

**Given** Raoul viewing a list of Sales Orders narrowed to the state `sale`, grouped by salesperson, from a window action whose identifier is 42, with a personal board whose layout has at least one column.
**When** Raoul opens the dashboard entry of the cog menu, types "Confirmed orders" and activates "Add".
**Then** the request answers positively; one Custom View record is created, owned by Raoul, referring to the shipped board view, whose layout holds a pinned element at the **top of the first column** naming the action 42, the title "Confirmed orders", the presentation kind `list`, the record selection and a context that carries the groupings and the ordering but **not** the active-companies key. Raoul is notified with the title "“Confirmed orders” added to dashboard" and the body "Please refresh your browser for the changes to take effect."

### AC-105 — Pinning when the board has no column

**Given** a board layout whose board element holds no column.
**When** Raoul pins a view.
**Then** the request answers negatively, no Custom View record is created, and Raoul is notified with "Could not add filter to dashboard".

### AC-106 — Pinning with no action identifier

**When** the pinning request carries no window action identifier.
**Then** it answers negatively and stores nothing, by rule SD-064.

### AC-107 — Reading the board

**Given** Raoul with two Custom View records for the shipped board view, the newer one holding two pinned elements.
**When** Raoul opens the personal board.
**Then** the layout returned is the one of a single record the search selects, its identifier is returned alongside it as the customised view, every pinned element marked invisible has been removed, and the root of the layout is marked so that the board presentation is instantiated rather than an ordinary form.

### AC-108 — Removing a pinned element

**Given** an open board with two pinned elements in one column.
**When** Raoul activates the close control on the first and confirms "Are you sure that you want to remove this item?".
**Then** the element disappears, and the whole layout is stored back to the same customised view.

### AC-109 — Changing the layout

**Given** an open board whose layout is `1-1-1`, with one element in each of the three columns.
**When** Raoul chooses the layout `1`.
**Then** the elements of the second and third columns move, in order, to the end of the first column, the layout becomes `1`, and the whole layout is stored.

### AC-110 — The board creates no record

**When** a client opens the personal board and the platform initialises a placeholder record for the form.
**Then** the creation operation stores nothing and answers an empty set, by rule SD-061, and no row exists anywhere for the board.

## 15. Utilities

### AC-111 — Date and moment numbers

**When** the date 30 December 1899 and the date 1 October 2023 are converted.
**Then** the numbers are 0 and 45200.
**When** the moment 30 December 1899 at 00:00:00 is converted in coordinated universal time and in a zone eight hours ahead.
**Then** the numbers are 0 and 0.3333333333.
**When** the moment 1 October 2023 at 12:00:00 is converted in the same two zones.
**Then** the numbers are 45200.5 and 45200.8333333333.

### AC-112 — Converting a date pattern

**When** the patterns `%m/%d/%Y`, `%b/%a/%y`, `%B/%A/%Y`, `%m %d %Y`, `%m-%d-%Y`, `%m-%d/%Y`, `%m.%d.%Y`, `%a, %Y.eko %bren %da`, `%Y년 %m월 %d일` and `%w %x %Z %j %m %d %Y` are converted.
**Then** the results are, in order, `mm/dd/yyyy`, `mmm/ddd/yy`, `mmmm/dddd/yyyy`, `mm dd yyyy`, `mm-dd-yyyy`, `mm-dd-yyyy`, `mm/dd/yyyy`, `ddd yyyy mmm dd`, `yyyy mm dd` and `mm dd yyyy`.

### AC-113 — Converting a time pattern

**When** the patterns `%H:%M:%S`, `%I:%M:%S`, `%H:%M:%S %p`, `%H %M %S`, `%H %M:%S`, `%H-%M-%S`, `%H시 %M분 %S초` and `%H:%M:%S %f %z` are converted.
**Then** the results are, in order, `hh:mm:ss`, `hh:mm:ss a`, `hh:mm:ss a`, `hh mm ss`, `hh mm ss`, `hh:mm:ss`, `hh mm ss` and `hh:mm:ss`.

### AC-114 — Display names for a workbook

**Given** a contact named "Bob" whose identifier is known, an archived contact named "Bob" likewise, a contact named "Alice", and a user named "Alice".
**When** the display-name operation is called with, in turn: one contact; the archived contact; two contacts, Alice then Bob; one identifier that exists nowhere; a real contact followed by a missing one; and a user followed by a contact.
**Then** the answers are, in order: one name "Bob"; one name "Bob", because archived records are included; two names "Alice" and "Bob" in the order asked; one answer that is nothing; two answers, "Bob" then nothing; and two names, "Alice" then "Bob", drawn from two different entities in one call.

### AC-115 — Whether an entity has a searchable parent link

**When** the parent-link operation is called with the entity `res.users`, then with `ir.ui.menu`.
**Then** the answers are false and true.

### AC-116 — The session capability flag

**When** a signed-in user fetches the session description.
**Then** it carries the key `can_insert_in_spreadsheet` set to false.

### AC-117 — Extending a serialised document

**When** the extension rule is applied to the document `{}` with no pair, then with the pair `key` and the value `{}`, then with the pair `key` and the value `[]`, then with the pair `key` and the value `"value"`.
**Then** the results are `{}`, `{"key":{}}`, `{"key":[]}` and `{"key":"value"}`.
**When** it is applied to `{"a": 1}` with the pair `key` and the value `"value"`, then with the value `{"b": 2}`, then with the value `[]`.
**Then** the results are `{"a": 1,"key":"value"}`, `{"a": 1,"key":{"b": 2}}` and `{"a": 1,"key":[]}`.
**When** it is applied to `{"a": {}}` with the pair `key` and the value `{"b": 2}`, and to `{"a": []}` with the pair `key` and the value `[]`.
**Then** the results are `{"a": {},"key":{"b": 2}}` and `{"a": [],"key":[]}`.
**When** it is applied to `{}` with two pairs, `key1` with the value `1` and `key2` with the value `2`, and then to the same document surrounded by line breaks.
**Then** both results are `{"key1":1,"key2":2}`.

### AC-118 — The download file name

**Given** a Spreadsheet Dashboard whose display name is "Sales".
**When** the download file name is computed.
**Then** it is `Sales.osheet.json`.

## 16. Freezing and the frozen workbook

### AC-119 — What freezing does to a cell

**Given** an open dashboard, a loaded list, and the cell holding the list-value formula for row 1, column "name", evaluating to "Deco Addict".
**When** the dashboard is frozen.
**Then** the frozen document holds the plain text `Deco Addict` at that cell, and the workbook's list section is empty.

### AC-120 — A cell that evaluated to nothing

**Given** a cell holding a list-value formula that evaluates to the empty text.
**When** the dashboard is frozen.
**Then** the cell holds the formula that yields an empty text, so the cell exists and is empty rather than absent.

### AC-121 — A chart becomes a picture

**Given** a figure holding a data-bound bar chart 600 points wide and 400 points high.
**When** the dashboard is frozen.
**Then** the figure is tagged as an image, its content is a picture of the chart drawn on a white background at 600 by 400 points, and no chart definition remains for it.

### AC-122 — A link into the application is neutralised

**Given** a cell holding a link labelled "Open orders" whose target begins with `odoo://ir_menu_xml_id/`.
**When** the dashboard is frozen.
**Then** the cell holds a link with the same label whose target is `neutralized:link`, which is not external, whose address cannot be edited, which presents no address and which does nothing when followed.

### AC-123 — The filter sheet

**Given** a workbook with two filters, the first a text filter labelled "Salesperson" whose value is "Mitchell", the second a date filter labelled "Period" whose window is 1 March 2026 to 31 March 2026, and a reader whose date format is `mm/dd/yyyy`.
**When** the dashboard is frozen.
**Then** a sheet named "Active Filters" is appended holding: `Filter` and `Value` as bold headings in the first row; "Salesperson" in the first column of the second row with "Mitchell" beside it; "Period" in the first column of the third row with the serial number of 1 March 2026 beside it and the serial number of 31 March 2026 in the row below; the sheet has at least two columns and four rows. The first filter's definition gains the value "Mitchell" and the second's gains "03/01/2026, 03/31/2026".

### AC-124 — A workbook with no filter gains no sheet

**Given** a workbook with no global filter.
**When** the dashboard is frozen, or exported as a workbook file.
**Then** no "Active Filters" sheet is appended.

### AC-125 — Copying is refused in a frozen workbook

**Given** the public page of a shared dashboard.
**When** a copy is attempted.
**Then** it is refused with the marker `Readonly`, and the copy entries of the cell, column, row and edit menus are disabled.
