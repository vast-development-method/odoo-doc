# Workflows

Sixteen end-to-end procedures. Each step states what it reads, what it creates or changes, and how it can fail. Rule references point at [`business-rules.md`](business-rules.md); formula references at [`calculations.md`](calculations.md).

## 1. Configuring a dashboard group

**Actor.** A user holding the dashboard administrator group.

1. The administrator opens the dashboards configuration list, reached from the dashboards menu, configuration submenu, dashboards entry.
2. The list shows every Dashboard Group, ordered by `sequence` then by identifier, each row showing a drag handle — offered only to a technical user — and the name.
3. Creating a row creates one Dashboard Group. The name is required; an empty name is refused by rule [SD-003](business-rules.md#sd-003).
4. Dragging a row rewrites the `sequence` of the moved row and of the rows it passed, so that the stored order matches the shown order.
5. Opening a row opens the group form: the name as a heading, and one page named "Spreadsheets" holding the group's dashboards in the dashboard list presentation.
6. Deleting a row runs the guard of [`entities.md`](entities.md) §4.4. A group shipped by a capability package is refused by rule [SD-009](business-rules.md#sd-009). A group still holding dashboards is refused by the storage rule that protects the link, rule [SD-010](business-rules.md#sd-010).

## 2. Configuring a dashboard

**Actor.** A user holding the dashboard administrator group. Nobody else may create, write or delete a dashboard; rule [SD-004](business-rules.md#sd-004).

1. From the group form, page "Spreadsheets", the administrator works on the group's dashboards directly. The list is not creatable from that page; a dashboard is added by the create control of the list embedded there, which the dashboards configuration provides, or by duplicating an existing one.
2. Each row carries: a drag handle for the sequence, shown only to a technical user; the name; the access groups, as tags, required; the companies, as tags, shown only in a multi-company installation, with a placeholder reading "Visible to all" and with creation of new companies forbidden from the widget; the workbook itself, as a file control, shown only to a user in the extended-visibility group; the publication toggle; and, optionally shown, the dashboard group.
3. Setting the name is required; rule [SD-003](business-rules.md#sd-003).
4. Setting the dashboard group is required; rule [SD-003](business-rules.md#sd-003).
5. Leaving the access groups empty removes the dashboard from everybody's workspace, because the audience rule of [`configuration.md`](configuration.md) §5 then matches nobody. The list control marks the field required to prevent that by accident.
6. Leaving the companies empty means every company.
7. Uploading a workbook file into the workbook control runs the validation of [`entities.md`](entities.md) §2.5 immediately, while the row is still being edited, and a bad file is refused before the row is saved; rules [SD-001](business-rules.md#sd-001) and [SD-002](business-rules.md#sd-002).
8. Saving writes the dashboard. A dashboard created with no workbook receives the empty workbook of [`document-format.md`](document-format.md) §6, whose sheet name is translated into the creating administrator's language.
9. Duplicating a dashboard applies §14.

## 3. Opening the dashboard workspace

**Actor.** Any internal user.

1. The user activates the dashboards menu, or its dashboards submenu, or navigates to the address path `dashboards`. Both menu entries point at the same client action.
2. The client action starts and asks the workspace loader to load.
3. The loader reads every Dashboard Group whose published sub-list is non-empty, taking for each the name and, for each published dashboard, its identifier, its name and its favourite mark. The read passes through the reader's own rights, so:
   - a group whose published dashboards are all outside the reader's audience comes back with an empty sub-list and is dropped;
   - a dashboard restricted to companies none of which are active for the reader is not returned.
4. Each returned dashboard becomes an entry at status `NotLoaded` ([`state-machines.md`](state-machines.md) §4).
5. The initial dashboard is chosen, in this order: the one recorded in a restored session state; otherwise the one named by the action's parameters; otherwise the first dashboard of the first group; otherwise none, and the reader sees the text "No available dashboard".
6. The chosen dashboard is activated, which triggers §4.
7. The address is updated to carry the active dashboard's identifier, so that the page can be reopened on the same dashboard.
8. The sidebar is drawn: a "FAVORITES" section first when the reader has starred at least one dashboard, then one section per group in `sequence` order, each holding its published dashboards in `sequence` order. On a small screen the sidebar is replaced by a picker in the control panel.

## 4. Reading one dashboard

**Actor.** Any internal user who may read the dashboard.

1. The workspace issues a read to the dashboard reading route, addressing the dashboard by identifier. The entry moves to `Loading`.
2. The route resolves the dashboard through the reader's rights. A dashboard that does not exist, or that the reader may not read, yields not found; rule [SD-014](business-rules.md#sd-014).
3. The route reads the active companies from the request: the companies cookie when present, otherwise the reader's own company. The value is a list of identifiers separated by hyphens, and each is read as a number. The rest of the work happens with exactly those companies active.
4. The route decides between sample and live by the rule of [`state-machines.md`](state-machines.md) §5.2.
5. **Sample branch.** The sample workbook is loaded from the path and returned as the structured answer `{snapshot, is_sample}` with the sample mark set to true. The procedure ends.
6. **Live branch.** The route builds the reading payload:
   1. The workbook text is parsed.
   2. The reader's locale, computed by [`calculations.md`](calculations.md) §3, is written into the workbook's settings under `locale`, creating that section when absent. This overwrites whatever locale the workbook was stored with, so two readers of the same dashboard see their own number and date conventions.
   3. The default currency of the active company is read, as code, symbol, number of decimal places and symbol position.
   4. The translation namespace is read: the capability package that shipped this dashboard, or nothing.
   5. The four parts are returned together with an empty revision list.
7. The answer is served as a structured document.
8. The workspace builds a workbook model from the answer, in dashboard presentation mode, with the default currency attached and the translation namespace attached, and activates the model's first sheet when it is not already active. The entry moves to `Loaded`.
9. The model registers a listener so that every arrival of data from a data source triggers a re-evaluation of the cells.

**Failures.** Any failure of steps 1 to 7 moves the entry to `Error` and the reader sees "An error occured while loading the dashboard".

## 5. Showing a sample instead of an empty dashboard

**Actor.** The reading route, automatically.

1. The route checks that `sample_dashboard_file_path` is set. An unset path ends the check and the live branch is taken.
2. The route asks whether the dashboard is empty. For each entry of `main_data_model_ids`, in order: the model is read; when the reader may not read it, the count is taken with elevated rights instead; the count is a search for at most one record with no conditions. The first model that yields zero makes the dashboard empty and stops the walk. A dashboard with no measured models is never empty.
3. The route loads the file named by the path. A missing file ends the check and the live branch is taken.
4. The sample answer is returned.

**Consequences for the reader.** A sample dashboard hides the search bar, the sharing control, the favourite star and the small-screen picker, and the grid is marked as a sample. Nothing about the sample is stored, and the reader cannot interact with it beyond looking.

## 6. Reading a value and drilling through to records

### 6.1 Evaluating a cell backed by a list

1. The cell's formula names a list element, a row position and a field path.
2. The element identifier is checked; an unknown element raises rule [SD-017](business-rules.md#sd-017).
3. The field path is checked for emptiness; an empty path raises rule [SD-016](business-rules.md#sd-016).
4. The data source is consulted. Its status decides what comes back, by [`state-machines.md`](state-machines.md) §6.2.
5. A valid source returns the record at that position, converted from storage form to cell form by [`calculations.md`](calculations.md) §15, and a number format chosen by [`calculations.md`](calculations.md) §14.
6. A position beyond the fetched window widens the window and returns the loading marker.
7. A field path the source has not fetched yet is added to the fetch set and the loading marker is returned.
8. A field path that does not exist, or that the reader may not read, raises rule [SD-018](business-rules.md#sd-018).

### 6.2 Drilling from a list cell to one record

1. The reader opens the context menu on the cell and chooses "See record". The entry is offered only when the cell holds exactly one list formula, that formula is the value formula rather than the header formula, and the cell is neither empty nor in error.
2. The formula's row position is evaluated. A position that is not a number ends the procedure silently.
3. The data source is loaded if needed, and the record identifier at that position is read. A position holding no record ends the procedure silently.
4. A window action is opened on that record in form presentation, reusing the window action named by the element's `actionXmlId` when one is recorded, and otherwise a plain form action on the element's entity with the element's reading context.

### 6.3 Drilling from a pivot cell to a list of records

1. The reader opens the context menu on the cell and chooses "See records". The entry is offered only when the cell holds exactly one pivot formula, the pivot is data-bound, the cell is neither empty nor in error, the cell is positioned on a real group, and a record selection can be derived from that position.
2. The pivot's data source is loaded.
3. The record selection for that cell is computed: the pivot's own selection, the global-filter conditions, and one condition per grouping level of the cell's position.
4. The entity's label is read and used as the action title.
5. A window action is opened showing a list and a form, reusing `actionXmlId` when recorded.

### 6.4 Drilling from a chart

1. The reader clicks a bar, a slice or a point of a data-bound chart, or middle-clicks it to open in a new window.
2. The clicked item is turned into a record selection by combining the chart's own selection with one condition per grouping level of the clicked item.
3. A window action is opened showing a list and a form, titled with the clicked item's name, reusing `actionXmlId` when recorded.
4. A chart that is instead linked to a menu opens that menu's action. A linked menu that has no action shows the notice of rule [SD-034](business-rules.md#sd-034) and nothing is opened.

### 6.5 Auditing an accounting cell

1. The reader opens the context menu on a cell holding exactly one accounting formula that is neither empty nor in error, and chooses "See records".
2. The formula's arguments are evaluated in place: for the partner balance formula, the partner identifiers, the account codes, the period, the offset, the company and the unposted flag; for the tagged balance formula, the tag identifiers then the rest; for the other four, the account codes then the rest.
3. Account codes are split on commas and trimmed. An argument that is missing or in error yields an empty list.
4. The period is parsed by [`calculations.md`](calculations.md) §10. A period that is missing or in error is replaced by the current year for the residual, partner balance and tagged balance formulas, and left unset for the others.
5. The offset is read as a whole number, defaulting to zero, and added to the period's year.
6. The company is read as a whole number or left unset; the unposted flag is read as a logical value, defaulting to false when it cannot be read.
7. The parameters are sent to the audit operation, which builds the journal-item selection of [`calculations.md`](calculations.md) §6 — with the payable-and-receivable fallback enabled — and returns a window action listing those journal items, titled "Cell Audit".
8. The action is opened.

## 7. Marking a dashboard as a favourite

**Actor.** Any internal user who may read the dashboard.

1. The reader activates the star in the control panel of the open dashboard.
2. The favourite operation is called on that dashboard alone; addressing more than one record is refused by rule [SD-020a](business-rules.md#sd-020a).
3. The operation adds or removes the reader's user identifier in `favorite_user_ids`, with elevated rights.
4. The workspace flips the mark it holds for that entry without refetching, so the star and the "FAVORITES" section update at once.

## 8. Sharing a dashboard

**Actor.** Any internal user who may read the dashboard.

1. The reader opens the share control in the control panel of the open dashboard. The control is not offered while a sample is shown.
2. The workbook waits until no cell is in the loading state: every data-bound chart, pivot and list is loaded, every used map outline is loaded, and the cells are re-evaluated until none of them holds the loading marker.
3. The workbook is frozen by the algorithm of [`calculations.md`](calculations.md) §19, producing a document with values instead of formulas, pictures instead of data-bound charts, neutralised links, no list elements, no data-bound pivot elements, each filter carrying its rendered value, and one appended sheet named "Active Filters".
4. The result is compared with the last share made in the same session. Nothing is sent when the workbook's revision marker, the cells of the appended filter sheet and the locale code are all unchanged; the previously obtained address is reused.
5. Otherwise an export log entry is written for the operation `freeze`, by §16.
6. The workbook is also exported as workbook-file parts.
7. The sharing operation is called with the dashboard identifier, the frozen workbook as text and the workbook-file parts. It packages the parts into one archive, stores the archive in `excel_export`, creates the Dashboard Share — which generates a token — and returns the address.
8. The address is shown and copied to the reader's clipboard. A clipboard that refuses the write is ignored; the address stays on screen.

**Records created.** One Dashboard Share, one attachment for its frozen workbook, and one attachment for its packaged workbook file when parts were supplied.

## 9. Opening a shared dashboard

**Actor.** Anybody holding the address, with or without an account.

1. The address is requested. The route reads the share with elevated rights.
2. A share that does not exist yields not found.
3. The access check of [`entities.md`](entities.md) §5.4 runs; a failure of either part is refused by rule [SD-011](business-rules.md#sd-011), answered as forbidden.
4. The route decides whether to offer a download: it offers one only when the *requesting* user holds the export group. An anonymous reader does not, so no download control is drawn for them.
5. The public page is rendered: the dashboard's name, the sentence "Frozen and copied on" followed by the share's creation moment, a download control when step 4 allowed one, and the identity controls of the portal — the signed-in reader's menu, or a sign-in invitation.
6. The page is handed the session description and, alongside it, the address of the data route, the address of the download when allowed, and the presentation mode `dashboard`.
7. The page fetches the data route (§10), builds a workbook model in dashboard presentation mode marked as frozen, and renders it.
8. Copying from a frozen workbook is refused by rule [SD-033](business-rules.md#sd-033), and the copy entries of the cell, column, row and edit menus are disabled.
9. The page offers, in its file menu, a download entry visible only when a download address was supplied.
10. A dashboard-mode page with filters offers a control that reveals the filters and their frozen values; filters whose rendered value is empty are not shown.

## 10. Reading the data of a shared dashboard

**Actor.** Anybody holding the address.

1. The data route is requested with the share identifier and the token.
2. A share that does not exist yields not found.
3. The access check of [`entities.md`](entities.md) §5.4 runs; a failure is refused by rule [SD-011](business-rules.md#sd-011).
4. The stored frozen workbook is streamed back exactly as stored — the same document the sharing reader froze, with no locale substitution, no currency and no revisions.

## 11. Downloading the workbook file of a shared dashboard

**Actor.** A signed-in user holding the address.

1. The download route is requested with the share identifier and the token. The route requires a signed-in user; an anonymous caller is sent to sign in.
2. The share is read with elevated rights. A share that does not exist yields an empty set and the access check refuses it.
3. The access check of [`entities.md`](entities.md) §5.4 runs; a failure is refused by rule [SD-011](business-rules.md#sd-011).
4. The requesting user's export right is checked; a user without it is refused by rule [SD-013](business-rules.md#sd-013).
5. The packaged workbook file stored in `excel_export` is streamed, named after the share — that is, after the dashboard.
6. A share created without workbook-file parts has nothing in `excel_export` and the stream is empty.

## 12. Inserting a data-bound list into a workbook

**Actor.** An author working in an editable workbook.

1. The author chooses a number of records and a set of columns and asks for the list to be inserted at the current cell.
2. The command carries: the sheet, the anchor column and row, the identifier the new element will take, its definition, the number of rows and the columns.
3. The identifier is checked: it must be exactly the next identifier the workbook expects, refused otherwise with the marker `InvalidNextId`; and it must not already be in use, refused otherwise with `ListIdDuplicated`.
4. The element is registered under that identifier and the workbook's next identifier is advanced by one.
5. The sheet is grown so that the block fits: columns are added after the last column when the distance from the anchor to the last column is smaller than the number of columns; rows are added after the last row when the distance from the anchor to the last row is smaller than the number of rows plus one.
6. The header row is written at the anchor row: one header formula per column, carrying the element identifier, the field name and — when the column has a label — that label as a third argument.
7. The body is written below it: for each row from one to the number of rows, and for each column, one value formula carrying the element identifier, the row position and the field name.
8. The element inherits the field matchings of any existing element over the same entity, by [`global-filters.md`](global-filters.md) §6.4, and the filter conditions are applied to its data source at once.

Re-inserting the same element elsewhere repeats steps 5 to 7 without registering anything. Duplicating an element copies the definition, names the copy after the original with " (copy)" appended unless a name is given, registers it under a new identifier and advances the next identifier. Renaming refuses an empty name with the marker `EmptyName`. Every one of these commands refuses an unknown element with `ListIdNotFound`.

## 13. Inserting a data-bound pivot into a workbook

**Actor.** An author working in an editable workbook.

1. The author asks for a pivot over an entity, with row dimensions, column dimensions and measures.
2. The definition is registered. Every date or date-and-time dimension with no stated granularity receives the granularity `month`, and its name with granularity is rewritten accordingly.
3. A field may serve as a measure when it is a whole number, a decimal number or a monetary amount that carries an aggregator, or when it is a link to another entity; and when it is not the identifier field, not a dotted path, and stored.
4. A field may serve as a dimension when it is groupable and its type has a normalisation rule.
5. A field may carry a grouping defined inside the workbook when it is groupable, is not itself such a grouping, and is a link to another entity, a text, a list of links or a stored option.
6. The available granularities are, for a date: year, quarter number, quarter, month number, month, week number, week, day of month, day, day of week. For a date and time, the same ten plus hour number, minute number and second number.
7. The cells are written by the pivot family of formulas, which the engine owns; the element identifier, the measure and the pairs of dimension and value are their arguments.
8. The element inherits field matchings as in §12.8.

Changing a pivot's definition reloads its data only when the change can alter the answer: a change of dimensions, of record selection, of context, of entity, of workbook-defined groupings, or a change of measures that adds, removes or redefines a measure that is fetched rather than computed inside the workbook. A change that only reorders or re-labels measures updates the presentation without a reload. A change of sort column or of collapsed groups updates the presentation only.

## 14. Duplicating and deleting a dashboard

1. The administrator duplicates a dashboard. Every field is copied except `main_data_model_ids`.
2. When no name was supplied, the copy's name is the original's name followed by " (copy)"; [`calculations.md`](calculations.md) §18.
3. The copy's workbook attachment is a copy of the original's.
4. Deleting a dashboard deletes its shares by cascade, and its workbook attachment.
5. Deleting a dashboard that is the last published one of its group makes that group disappear from the workspace, because the workspace lists only groups with a non-empty published sub-list.

## 15. Building a personal board

**Actor.** Any internal user.

1. The user opens an ordinary view — a list, a chart, a pivot — narrows it with the search controls, and asks for it to be added to the personal board, giving it a name.
2. The request carries: the window action's identifier, the context to remember, the record selection, the presentation kinds to offer, and the name.
3. The board's shipped action is read with elevated rights. The procedure continues only when that action exists, addresses the board entity, offers a form as its first presentation, and an action identifier was supplied; otherwise the request answers negatively and nothing is stored.
4. The board layout is read for this user, already merged with any existing customisation and already preprocessed.
5. The first column of the layout is located. A layout with no column ends the procedure negatively.
6. The remembered context has the active-companies key removed, so that the pinned element follows the companies the reader has active at reading time rather than those active at pinning time.
7. A pinned-action element is inserted at the top of that column, carrying the action identifier, the name, the presentation kinds, the context and the record selection.
8. The whole layout is stored as a new Custom View record owned by this user and referring to the shipped board view, written with elevated rights. The request answers positively.
9. Opening the board afterwards reads the newest such record for this user — the search takes one record — and returns its layout in place of the shipped one, after the preprocessing of [`entities.md`](entities.md) §6.4.

**Note.** Each addition stores a further Custom View record rather than replacing the previous one, and the board reads one of them. Removing a pinned element, or resetting the board, is done by deleting those records.

## 16. Logging an export of data

**Actor.** The workbook, automatically.

1. One of four operations occurs: a copy of a large region, a download, a freeze for sharing, or a print.
2. A copy is logged only when the copied region covers more than four hundred cells, counting every cell of every selected rectangle.
3. The workbook collects one description per loaded data source: for a chart, the entity, the kind `graph`, the measure, the groupings and the record selection; for a data-bound pivot, the entity, the kind `pivot`, the measure field names, the dimension names with their granularities and the record selection including the filter conditions; for a list, the entity, the kind `list`, the columns, no grouping and the record selection. Sources that are not valid are skipped.
4. The descriptions are posted to the logging route with the name of the operation.
5. The route accepts only the four operation names `download`, `copy`, `freeze` and `print`; any other name is ignored and nothing is written.
6. Each description is rendered as one line: the entity, then the field names, then the groupings when there are any, then the record selection when there is one. A description naming an entity that does not exist, or carrying no fields, is dropped.
7. A request whose descriptions all dropped writes nothing.
8. Otherwise one entry is written to the application log, at informational level, naming the acting user's identifier, the operation, every rendered description, and the network address the request came from.

The rendering is specified in [`business-rules.md`](business-rules.md) §11.
