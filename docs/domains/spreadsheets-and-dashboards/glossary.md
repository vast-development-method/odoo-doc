# Glossary

Every term this folder uses in a sense that is particular to the domain, with the identifier it corresponds to where there is one. Terms are grouped by subject and alphabetical within each group.

## 1. Documents and their parts

**Workbook.** One structured document holding sheets, cells, figures, data-bound elements, global filters and settings. It is the value of `spreadsheet_binary_data` on every record that carries the Spreadsheet Document mixin, and it is the payload every route of this domain moves. Its sections are specified in [`document-format.md`](document-format.md).

**Workbook file.** A packaged archive of parts in the interchange format that spreadsheet applications exchange. A workbook may be *stored* as one, in which case its decoded content declares the key `[Content_Types].xml` and the reference walk is skipped; and a workbook may be *exported* as one, in which case the parts are packaged and streamed.

**Sheet.** One tab of a workbook, carrying a stable identifier that never changes and a name that is translated. The empty workbook has one sheet whose identifier is `sheet1` and whose name is the translated word "Sheet1".

**Cell.** One position in the grid of a sheet, holding either a literal value or a formula, and optionally a number format, a style and a link.

**Figure.** A floating element laid over the grid: a chart, a carousel of charts, or a picture. Freezing turns every data-bound chart figure into a picture figure.

**Carousel.** A figure holding several chart definitions, of which one is shown at a time.

**Number format.** The pattern that decides how a cell's value is written: how many decimal digits, whether a thousands separator appears, where a currency symbol sits, how a date is laid out. A format changes the presentation only, never the stored value.

**Revision marker.** The value of `revisionId`, identifying the revision the stored content corresponds to. The empty workbook carries the reproduced value `START_REVISION`. The freeze comparison of [`calculations.md`](calculations.md) §19.4 uses it to decide whether the content changed.

**Neutralised link.** The target `neutralized:link`, written by the freeze algorithm in place of every link into the application. It renders as a label with no destination.

**Loading marker.** The reserved error value "Loading...", shown by a cell whose data source has not answered yet. It is an error value by construction, so that a workbook still waiting for data can be recognised by looking for it.

**Spill region.** The block of cells a single formula fills when it produces more than one value. Freezing replaces every cell of the region, not only the one holding the formula.

## 2. Records

**Dashboard Board** (`board.board`). The abstract entity behind the personal board. It has no table: a board is a form with no record behind it, whose layout is stored per user as a customised view.

**Dashboard Group** (`spreadsheet.dashboard.group`). A named, ordered section of the dashboard workspace that holds dashboards. Seven are shipped; a group whose published sub-list is empty does not appear in the sidebar.

**Dashboard Share** (`spreadsheet.dashboard.share`). A frozen copy of one dashboard, reachable through a secret token, optionally carrying a packaged workbook file for download. It is created by sharing and it never re-reads the dashboard it copies.

**Spreadsheet Dashboard** (`spreadsheet.dashboard`). One dashboard: a workbook plus the audience, the companies, the group and the ordering that decide who sees it and where.

**Spreadsheet Document mixin** (`spreadsheet.mixin`). The abstract entity that gives any other entity the ability to carry one workbook, to validate it, to expose it as text and to name its download file. Three entities in this repository carry it: Spreadsheet Dashboard, Dashboard Share, and any entity a package outside this domain declares it on.

## 3. Fields and stored values

**`access_token`.** The secret part of a shared address, a freshly generated universally unique identifier. It is compared in constant time; see rule [SD-021](business-rules.md#sd-021).

**`company_ids` — companies.** The companies a dashboard is visible in. An empty list means every company, not no company.

**`excel_export`.** The packaged workbook file a Dashboard Share carries, when the sharing reader supplied workbook-file parts. Empty when they did not, in which case the download route streams nothing.

**`favorite_user_ids` — favourite users.** The users who have starred a dashboard. Written with elevated rights, because an ordinary reader may not write a dashboard.

**`full_url` — web address.** The complete shared address of a Dashboard Share, computed from the installation's base web address, the share's identifier and its token.

**`group_ids` — access groups.** The audience of a dashboard: a reader sees it when they hold at least one of these groups. Defaults to the internal-user group.

**`is_favorite`.** Whether the acting reader has starred the dashboard. Computed per reader, never stored.

**`is_published`.** Whether a dashboard is offered in the workspace. Default true. Publication is a listing decision, not an access decision.

**`main_data_model_ids` — measured entities.** The entities a dashboard measures. Used only to decide whether the dashboard is empty and therefore whether a sample replaces it. Never copied when a dashboard is duplicated.

**`sample_dashboard_file_path`.** The location of the sample document shipped with a dashboard.

**`sequence`.** A position, not a number drawn from a sequence. It orders dashboards within a group and groups within the sidebar.

**`spreadsheet_binary_data` — the workbook file.** The stored workbook, held as a file. Validated on every write.

**`spreadsheet_data` — the workbook text.** The same workbook as text, computed from the stored file and written back through it.

**`spreadsheet_file_name`.** The download name of a workbook: the record's display name followed by the reproduced suffix `.osheet.json`.

**`thumbnail`.** A small picture of a workbook, stored beside it. This domain stores it and never produces it.

## 4. Data-bound elements

**Data-bound element.** A list, a pivot or a chart inside a workbook that reads records rather than cells. Every one of them is backed by a data source, carries a record selection, and carries one field matching per global filter.

**Data source.** The object that owns the queries for one data-bound element and caches their answers. Its states are specified in [`state-machines.md`](state-machines.md) §6.

**Data-bound chart.** A figure whose chart kind begins with `odoo_`. Twelve kinds exist; see [`interfaces.md`](interfaces.md) §6.

**Dimension.** A grouping axis of a pivot: a field name, optionally a granularity, optionally marked positional.

**Field matching.** For one pair of a global filter and a data-bound element, the path by which that filter reaches that element's entity, the kind of the field at the end of the path, and — for date filters — a period offset.

**Field path.** A dotted sequence of field names walked from an entity to a field, possibly through relations. A list column, a filter matching and a validation finding are all expressed as field paths.

**Granularity.** The size of the period a date dimension groups by: a year, a quarter, a month, a week, a day, or the ordinal of one of those within its container. A date dimension with no granularity receives `month`.

**High-water mark.** The largest row position any cell of a list has asked for. The list fetches exactly that many records and no more.

**List element.** A data-bound element presenting records as rows and fields as columns, written into the grid as one header formula per column and one value formula per cell.

**Measure.** A quantity a pivot aggregates. A measure is *fetched* when the server computes it, and *computed inside the workbook* when the workbook does; only the first kind forces a reload when it changes.

**Pivot element.** A data-bound element presenting records grouped along row dimensions and column dimensions, with one or more measures. A pivot is data-bound when its kind is the reproduced value `ODOO`; a pivot over cell ranges is not owned by this domain.

**Positional reference.** A dimension addressed by the rank of its group inside its parent rather than by the group's value, written with a leading number sign.

**Record selection.** The set of conditions that decide which records a data-bound element reads: the element's own conditions, joined with the conditions every matched global filter contributes.

## 5. Global filters

**Global filter.** A named control belonging to a whole workbook, narrowing every data-bound element of it at once, each through its own field matching. Its definition is stored; its value is not, except in a frozen workbook.

**Active filter.** A filter whose resolution yields a value, whether from a set value or from a default. The active count beside the search bar is the number of them.

**Cleared.** The state of a filter whose reader has explicitly emptied it: nothing applies, and the default does not come back.

**Default value.** The value a filter takes when the reader has set none and has not cleared it. A relational default may be the marker `current_user`, which resolves to the reading reader.

**Empty value.** The shape of a value with nothing chosen: an empty text list, an empty identifier list, an absent number. An empty value still counts as set, so it suppresses the default.

**Operator.** Part of a filter's value, not of its definition. The same text filter may be used with one operator on one reading and another on the next.

**Period.** The value of a date filter: one of nine relative periods, or a fixed month, quarter, year or range.

**Period offset.** A whole number of periods by which one element's window is shifted relative to the filter's own, so that one filter can show two periods side by side.

**Relative period.** A period expressed against the moment of reading — `today`, `yesterday`, `last_7_days`, `last_30_days`, `last_90_days`, `month_to_date`, `last_month`, `last_12_months`, `year_to_date` — rather than by a fixed date.

**Untouched.** The state of a filter the reader has not set: the default applies.

**Window.** The first moment and the last moment a period resolves to, before either is turned into a condition.

## 6. Accounting vocabulary of this domain

**Account code prefix.** The text a code-based accounting formula matches against the beginning of an account's code, letter case included. One prefix may select many accounts; two identical prefixes select each account once.

**Cell audit.** The action that opens the journal items behind an accounting cell, in a window action titled "Cell Audit".

**Cumulative selection.** The half of the period condition that applies to an account including the initial balance: every item dated on or before the window's last date, whatever the first date is.

**Fiscal year.** The twelve-month period a company's accounts are kept in, fixed by the company's fiscal-year last day and last month. A year in an accounting formula is a fiscal year, never a calendar year unless the two coincide.

**Payable-and-receivable fallback.** The rule by which an empty account-code list means every payable and every receivable account of the company. It applies to the residual formula, the partner balance formula and the cell audit action, and to nothing else.

**Period condition.** The disjunction that splits an accounting selection into a cumulative half for balance-sheet accounts and a period-confined half for profit-and-loss accounts.

**Period-confined selection.** The half of the period condition that applies to an account not including the initial balance: only items dated inside the window.

**Posted condition.** Either "the entry is posted", or — when unposted entries are asked for — "the entry is in any state other than cancelled". A cancelled entry never contributes.

**Residual amount.** What remains unsettled on a journal item after reconciliation. It is a *current* property, so a residual asked for a past window reflects reconciliations performed since.

**Year offset.** A whole number added to the year of a parsed period before the request, checked against the lower bound of 1900 after the addition.

## 7. Reading, sharing and presenting

**Audience.** The access groups a dashboard names; a reader sees the dashboard when they hold at least one of them, unless they are a dashboard administrator.

**Dashboard administrator.** A user holding the group `spreadsheet_dashboard.group_dashboard_manager`. The only authority of this domain: creation, modification and deletion of dashboards and groups, and unrestricted visibility of dashboards within the active companies.

**Dashboard presentation.** The reading mode in which a workbook is drawn without its editing surfaces: no formula bar, no sheet tabs to edit, no grid selection to write into. Every dashboard, every frozen page and every workbook being printed is in this mode.

**Drill-through.** Opening the records behind a cell, a pivot cell or a chart item, in a window action.

**Freezing.** Turning a live dashboard into a self-contained snapshot: values instead of formulas, pictures instead of data-bound charts, neutralised links, no lists, no data-bound pivots, one appended filter sheet. Specified in [`calculations.md`](calculations.md) §19.

**Frozen workbook.** The result of freezing. A frozen workbook cannot be copied from, cannot be re-filtered, and holds no query.

**Locale.** The eight-part description that tells a workbook how to write numbers, dates and times, where a week starts and which character separates formula arguments. A dashboard is served with the *reader's* locale, whatever locale it was stored with.

**Published sub-list.** The dashboards of a group whose `is_published` is true. The workspace lists only groups whose published sub-list is not empty.

**Reachability.** Whether a handed-out shared address still resolves: reachable, refused for a token mismatch, refused because the sharing user lost the right, or gone. Specified in [`state-machines.md`](state-machines.md) §8.

**Reading payload.** The four-part answer of the dashboard reading route: the snapshot, the revisions, the default currency and the translation namespace.

**Reading session.** One reader's open view of one dashboard. Filter values, data-source states and the rendering status live here and nowhere else; nothing about them is stored.

**Revocation.** The loss of reachability of a shared address caused by the *sharing* user losing the right to read the dashboard. It changes no record.

**Sample dashboard.** A demonstration workbook served in place of the real one when the dashboard measures at least one entity that holds no record. Nothing about a sample is stored and a reader cannot interact with it.

**Sharing.** Freezing a dashboard, storing the frozen copy as a Dashboard Share, and answering with a tokenised address.

**Translation namespace.** The capability package that shipped a dashboard, sent with the reading payload so that the client can look up the translated form of the labels inside the workbook.

**Workspace.** The screen the dashboards menu opens: a sidebar of groups and dashboards, a control panel carrying the filters, the share control and the favourite star, and the active dashboard's grid.

## 8. Validation and logging

**Export log.** An entry written to the application log for each of four operations — a large copy, a download, a freeze and a print — naming the acting user, the operation, one line per loaded data source, and the network address the request came from.

**Finding.** One line of the reference walk's report, naming a missing entity, a missing field, a missing menu external identifier, or a menu that opens nothing.

**Reference walk.** The second stage of workbook validation: collecting every field path and every menu external identifier the document names, and checking each. It runs only while the installation is executing its automated test suite.

**Refusal marker.** A short reproduced token — `NoChanges`, `FilterNotFound`, `InvalidNextId` and the rest — by which a workbook command refuses. A client turns the marker into its own text; the marker itself is never shown.

## 9. The personal board

**Board layout.** The description of a personal board: a layout style, a set of columns, and the pinned elements inside them.

**Layout style.** One of the five reproduced values `1`, `1-1`, `1-1-1`, `1-2` and `2-1`, naming the number of columns and their relative widths. The shipped board uses `2-1`.

**Pinned element.** One entry of a board layout, naming a window action, a title, a presentation kind, a reading context and a record selection.

**Pinning.** Adding the current screen to the personal board from the cog menu, which stores a new customised view holding the whole layout with the new element at the top of the first column.

**Custom view.** The per-user record that stores a board layout. Read and written with elevated rights, because an ordinary user may neither read nor write views; the record's owner field confines the effect to that user.

## 10. Words used in their ordinary sense but worth pinning down

**Element identifier.** The key under which a list or a pivot is registered in the workbook, and the first argument of every formula that reads it. It is workbook-local and has nothing to do with a record identifier.

**External identifier.** The stable, package-qualified name of a shipped record. Its two consequences in this domain: a record is recognised on upgrade, and a Dashboard Group carrying one cannot be deleted.

**Elevated rights.** Performing one operation with the rights of the system rather than of the acting user. This domain elevates in exactly four places: writing the favourite mark, counting records of a measured entity the reader cannot read, resolving a share on the three public routes, and reading and writing a board layout. Every one of them is bounded, and each is justified where it is specified.

**Presentation.** A way of showing records — a list, a form, a card, a chart, a pivot. Used in place of the word a client would use for the same thing.

**Round trip.** One call carrying several requests, answered by one list of answers in the same order. Every fetching operation of this domain works this way, so that a workbook with fifty accounting cells makes one call rather than fifty.
