# Interfaces

Every surface through which a person or another system reaches this domain: menus, screens, routes, named operations, workbook formulas, cell actions, printing, import and export, and the registries a package outside the domain plugs into.

## 1. The surfaces at a glance

| Surface | Audience | Where |
|---|---|---|
| The dashboards menu and the dashboard workspace | Any internal user | §2 |
| The dashboards configuration screens | A dashboard administrator | §3 |
| The personal board | Any internal user | §4 |
| The public page of a shared dashboard | Anybody holding the address | §5 |
| Data-bound charts and their click-through | Any reader of a dashboard | §6 |
| The workbook formula surface | An author, and every reader through the cells | §7 |
| Routes | Clients, and anybody holding a shared address | §8 |
| Named operations | Clients | §9 |
| Cell menu entries | Any reader of a dashboard | §10 |
| Printing | Any reader of a dashboard | §11 |
| Import and export | An administrator, and a reader with the export right | §12 |
| Registries other packages plug into | Other packages | §14 |

The domain has no external integration at all: it contacts no third-party service, and no third-party service contacts it (§13).

## 2. The dashboard workspace

### 2.1 Reaching it

Three ways, all landing on the same client action:

1. The root menu "Dashboards".
2. Its child menu "Dashboards".
3. The address path `dashboards`, typed or bookmarked.

The action accepts one parameter, `dashboard_id`, naming the dashboard to open. The workspace writes the active dashboard's identifier back into the address under the same key as the reader moves between dashboards, so any state of the workspace can be bookmarked.

### 2.2 The sidebar

On a screen that is not small, the workspace draws a sidebar on the left. Its content:

1. A section headed "FAVORITES", present only when the reader has starred at least one dashboard, holding those dashboards.
2. One section per Dashboard Group whose published sub-list is not empty, in `sequence` order, headed by the group's name in capitals, holding that group's published dashboards in `sequence` order.

Each entry shows the dashboard's name and carries the name as its title, so a name too long for the sidebar can still be read. The active entry is marked. Beside each entry the workspace draws the first component registered in the dashboard-action registry, when there is one, handing it the dashboard's identifier, the dashboard's data and a callback that opens the dashboard for editing (§14.1).

A control at the top right of the sidebar collapses it. Collapsed, the sidebar becomes a single bar showing the active dashboard's group name, a slash, and the active dashboard's name, and clicking anywhere on it expands the sidebar again.

### 2.3 The control panel

| Control | Shown when |
|---|---|
| The search bar, holding the workbook's global filters | The active dashboard is loaded, is not a sample, and its workbook has at least one filter |
| The dashboard picker | The screen is small and the active dashboard is not a sample |
| The share control | The active dashboard is not a sample |
| The favourite star, filled or hollow | A dashboard is active and it is not a sample |
| The search-bar toggle | The screen is small |

The star carries the title "Toggle favorite".

### 2.4 The main area

| State of the active dashboard | What is drawn |
|---|---|
| No dashboard at all | The heading "No available dashboard" |
| `Loading` | The heading "Loading..." |
| `Error` | The text "An error occured while loading the dashboard" |
| `Loaded`, ordinary screen | The workbook grid, in dashboard presentation |
| `Loaded`, ordinary screen, sample | The same grid, marked as a sample |
| `Loaded`, small screen | The figures of the workbook stacked one under the other, without the grid |

The three texts are reproduced as the reader sees them, including the spelling of the third.

### 2.5 The search bar

The search bar is the workbook's global filters and nothing else: there is no free-text search over the dashboard. Each filter is drawn according to its kind, with the controls of [`global-filters.md`](global-filters.md) §2 and §3. A date filter draws a period picker with a step-back and a step-forward control ([`global-filters.md`](global-filters.md) §5.3). The number of active filters is shown beside the bar.

On a small screen the filters move into a panel that the search-bar toggle opens.

## 3. The configuration screens

### 3.1 Navigation

The menu "Dashboards", then "Configuration", then "Dashboards", opens the window action over Dashboard Group with a list and a form.

### 3.2 The group list

Two columns: a drag handle that rewrites `sequence`, offered only to a user holding the technical-settings group, and the name. The list is titled "Dashboards".

### 3.3 The group form

The name as the heading, then one page named "Spreadsheets" holding the group's dashboards through the dashboard list of §3.4. A dashboard added from that page belongs to the group being edited.

### 3.4 The dashboard list

Editable in place at the bottom, with creation from the list itself disabled. Its seven columns and their visibility are in [`configuration.md`](configuration.md) §9.1. Three of them deserve a note here:

- **Access groups** is marked required by the control, although the underlying field is not required in storage. Leaving it empty would make the dashboard invisible to every non-administrator, and the control prevents that by accident rather than by rule.
- **Companies** shows the placeholder "Visible to all" when empty, which is the exact meaning of an empty list, and forbids creating a company from the control.
- **The workbook** is offered only to a user in the extended-visibility group, as a file control labelled "Data" whose download name comes from `spreadsheet_file_name`. The control is the workbook file control of §12.3: it uploads and validates, and its download does nothing.

### 3.5 The dashboard form and card

A form with the name, the dashboard group, the companies, the access groups and the workbook; and a card presentation showing the name alone, used on small screens.

## 4. The personal board

### 4.1 Reaching it

The menu "Dashboards", then "My Dashboard".

### 4.2 The empty board

A board with no pinned element shows:

- the heading "Your personal dashboard is empty";
- the explanation "To add your first report into this dashboard, go to any menu, switch to list or graph view, and click "Add to Dashboard" in the extended search options.";
- and the hint "You can filter and group data before inserting into the dashboard using the search options."

All three texts are reproduced.

### 4.3 Pinning a view

On any window action over any entity, in any presentation other than a form, the cog menu offers an entry drawn with the dashboards icon and labelled "Dashboard". Opening it reveals the label "Add to my dashboard", a text box pre-filled with the current screen's display name, and a button labelled "Add". The text box submits on the enter key.

What is sent: the window action's identifier, the current presentation kind, the current record selection, the name, and a context assembled as follows.

1. Start from the reader's global context, dropping every key whose name begins with `search_default_`, so that the pre-selected filters of the current screen are not frozen into the pinned element.
2. Add the search panel's own context.
3. Add the current ordering under the key `orderedBy`.
4. Add the current groupings under the key `group_by`.
5. Add the key `dashboard_merge_domains_contexts` set to false.

The active-companies key is then removed by the server, by rule [SD-065](business-rules.md#sd-065).

The answer is a logical value. On success the reader is notified with the title "“%s” added to dashboard", where the placeholder is the name they gave, and the body "Please refresh your browser for the changes to take effect.", shown as a warning; the text box is then reset to the screen's display name. On failure the reader is notified with "Could not add filter to dashboard", shown as a failure.

The notice about refreshing is a **compatibility finding**: the pinned element is stored immediately and correctly, but the board's own screen, if open in another tab, keeps the layout it read when it opened. A corrected behaviour would invalidate that layout so that no refresh is needed.

### 4.4 The board itself

The board draws its columns side by side according to its layout, and inside each column one panel per pinned element, in order. Each panel has a header showing the element's title and, on a screen that is not small, three controls:

| Control | Effect |
|---|---|
| A close control | Asks "Are you sure that you want to remove this item?" and, on confirmation, removes the element and stores the layout |
| A minimise control | Folds the element, hiding its content, and stores the layout |
| A restore control | Unfolds a folded element and stores the layout |

The panel's body is the pinned action rendered without its own control panel. A list is rendered without row selectors. Selecting a record inside a pinned element opens that record in a form, using the form presentation of the pinned action when it offers one.

A layout control at the top right offers the five layouts `1`, `1-1`, `1-1-1`, `1-2` and `2-1`, each shown as a small picture, the current one marked. Choosing a layout with fewer columns than the current one moves every element of the columns that disappear into the last column that remains, then stores the layout.

Elements may be dragged between columns and within a column by their headers; a drop rewrites the order and stores the layout.

On a small screen the board is forced to the single-column layout and the close control is not offered.

### 4.5 How the board is stored

Every one of the actions above serialises the whole board back into a layout description and sends it to the platform's custom-view update address with the identifier of the custom view the board was read from. Pinning from another screen, by contrast, creates a **new** custom view record; see [`workflows.md`](workflows.md) §15.

## 5. The public page of a shared dashboard

### 5.1 What the page shows

| Element | Detail |
|---|---|
| The dashboard's name | At the top left, emphasised |
| The freezing moment | The sentence "Frozen and copied on" followed by the share's creation moment |
| A download control | Only when the *requesting* user holds the export group; it carries the title "Download" and points at the download route |
| The identity controls | The signed-in reader's menu, or an invitation to sign in, both drawn by the portal |
| The workbook | Filling the rest of the page, in dashboard presentation, marked as frozen |

### 5.2 What the page is given

The page is handed the session description and three properties: the address of the data route, the address of the download route or an empty text, and the presentation mode, which is the reproduced value `dashboard`.

### 5.3 What the reader can do

| Action | Available |
|---|---|
| Read every cell, scroll, switch sheets | yes |
| Copy a selection | no; refused by rule [SD-056](business-rules.md#sd-056), and the copy entries of the cell menu, the column menu, the row menu and the edit menu are disabled |
| Change a filter | no; a frozen workbook holds no query to re-run |
| See the filters that were in force | yes, through a control that reveals them; only filters whose rendered value is not empty are shown, and they are shown as information, not as controls |
| Download the workbook file | only when the download control was drawn |
| Print | yes; §11 |
| Follow a link into the application | no; every such link was neutralised at freezing time |

### 5.4 The file menu

The page's file menu carries one added entry, labelled "Download", allowed in a read-only workbook, visible only when a download address was supplied.

## 6. Data-bound charts

### 6.1 The twelve kinds

A chart is data-bound when its stored kind begins with `odoo_`. Twelve kinds exist; the stored values are reproduced.

| Stored kind | What it draws |
|---|---|
| `odoo_bar` | Bars, vertical or horizontal, plain or stacked |
| `odoo_line` | A line, plain or stacked, optionally with the area under it filled |
| `odoo_combo` | Bars and a line together |
| `odoo_pie` | A pie or a ring |
| `odoo_scatter` | Points |
| `odoo_waterfall` | Contributions accumulating to a total |
| `odoo_pyramid` | Two opposed bar series sharing one axis |
| `odoo_radar` | A closed polygon over several axes, plain or filled |
| `odoo_geo` | Regions of a map shaded by value |
| `odoo_funnel` | Successively narrowing stages |
| `odoo_treemap` | Nested rectangles sized by value |
| `odoo_sunburst` | Nested rings sized by value |

### 6.2 The twenty offered variants

The author chooses a variant; each variant fixes a kind and, where the kind has options, their values.

| Variant | Kind | Options fixed | Label | Family |
|---|---|---|---|---|
| `odoo_line` | `odoo_line` | not stacked, not filled | "Line" | line |
| `odoo_stacked_line` | `odoo_line` | stacked, not filled | "Stacked Line" | line |
| `odoo_area` | `odoo_line` | not stacked, filled | "Area" | area |
| `odoo_stacked_area` | `odoo_line` | stacked, filled | "Stacked Area" | area |
| `odoo_bar` | `odoo_bar` | not stacked, vertical | "Column" | column |
| `odoo_stacked_bar` | `odoo_bar` | stacked, vertical | "Stacked Column" | column |
| `odoo_horizontal_bar` | `odoo_bar` | not stacked, horizontal | "Bar" | bar |
| `odoo_horizontal_stacked_bar` | `odoo_bar` | stacked, horizontal | "Stacked Bar" | bar |
| `odoo_combo` | `odoo_combo` | none | "Combo" | line |
| `odoo_pie` | `odoo_pie` | not a ring | "Pie" | pie |
| `odoo_doughnut` | `odoo_pie` | a ring | "Doughnut" | pie |
| `odoo_scatter` | `odoo_scatter` | none | "Scatter" | miscellaneous |
| `odoo_waterfall` | `odoo_waterfall` | none | "Waterfall" | miscellaneous |
| `odoo_pyramid` | `odoo_pyramid` | none | "Population Pyramid" | miscellaneous |
| `odoo_radar` | `odoo_radar` | not filled | "Radar" | miscellaneous |
| `odoo_filled_radar` | `odoo_radar` | filled | "Filled Radar" | miscellaneous |
| `odoo_geo` | `odoo_geo` | none | "Geo chart" | miscellaneous |
| `odoo_funnel` | `odoo_funnel` | cumulative | "Funnel" | miscellaneous |
| `odoo_treemap` | `odoo_treemap` | none | "Treemap" | hierarchical |
| `odoo_sunburst` | `odoo_sunburst` | none | "Sunburst" | hierarchical |

Four of the twelve kinds — bars, lines, combinations and waterfalls — are drawn with a component that lets the reader zoom into a range; the other eight are drawn without zooming.

### 6.3 Clicking a chart

Clicking a bar, a slice, a point or a region opens the records behind it, by [`workflows.md`](workflows.md) §6.4. A middle click opens them in a new window instead of the current one.

### 6.4 Linking a chart to a menu

A chart may instead be linked to a menu. The link is stored in the workbook's `chartOdooMenusReferences` section, keyed by the chart's identifier and holding the menu's external identifier. A chart so linked draws a control that opens the menu's action.

A linked menu that resolves but opens no action shows the notice of rule [SD-060](business-rules.md#sd-060) and opens nothing. A linked menu external identifier that resolves to nothing is a finding of the reference walk, [`document-format.md`](document-format.md) §7.3.

## 7. The formula surface

Fifteen functions, all filed under the reproduced category name `Odoo` so that they appear together in the formula list. Argument names are reproduced exactly, because the formula help shows them and an author types positionally against them.

### 7.1 Data-bound lists

| Function | Arguments | Answer |
|---|---|---|
| `ODOO.LIST` | `list_id` (text), `index` (text), `field_name` (text) | "Get the value from a list." The value of that field for the record at that position, converted by [`calculations.md`](calculations.md) §15 and formatted by §14 |
| `ODOO.LIST.HEADER` | `list_id` (text), `field_name` (text), `field_display_name` (text, optional) | "Get the header of a list." The supplied display name when it is not empty, otherwise the field's own label |

The argument help texts are, in order: "ID of the list.", "Position of the record in the list.", "Name of the field."; and for the header function "ID of the list.", "Technical field name.", "Name of the field."

The position counts from one. An unknown element raises rule [SD-030](business-rules.md#sd-030); an empty field name raises rule [SD-031](business-rules.md#sd-031); an unknown or unreadable field raises rule [SD-032](business-rules.md#sd-032).

### 7.2 Global filters

| Function | Arguments | Answer |
|---|---|---|
| `ODOO.FILTER.VALUE` | `filter_name` (text) | "Return the current value of a spreadsheet filter." The filter's display value: one cell for most kinds, two stacked cells for a date filter and for a numeric filter using `between` |
| `ODOO.FILTER.LABEL` | `filter_name` (text) | "Return the label of the current value of a spreadsheet filter." A name rather than a value; the rule is in [`global-filters.md`](global-filters.md) §11 |
| `ODOO.FILTER.VALUE.V18` | `filter_name` (text) | The same rule as the label function, under a second name, hidden from the formula list |

The argument help is "The label of the filter whose value to return." for all three. The compatibility function's own description is "Compatibility version of ODOO.FILTER.VALUE for v18 spreadsheets. Required for date filters. Optional for others."; the description is reproduced although the prose of this specification does not otherwise name versions.

A label matching no filter raises rule [SD-037](business-rules.md#sd-037). Escaped quotation marks in the supplied label are unescaped before matching, and both the supplied label and each filter's label pass through the workbook's translation first.

### 7.3 Currency

| Function | Arguments | Answer |
|---|---|---|
| `ODOO.CURRENCY.RATE` | `currency_from` (text), `currency_to` (text), `date` (date, optional), `company_id` (number, optional) | "This function takes in two currency codes as arguments, and returns the exchange rate from the first currency to the second as float." |

Argument help, in order: "First currency code.", "Second currency code.", "Date of the rate.", "The company to take the exchange rate from." The rule is [`calculations.md`](calculations.md) §13.1; an unavailable rate raises rule [SD-038](business-rules.md#sd-038).

### 7.4 Account movements

Three functions sharing five arguments:

| Function | Description |
|---|---|
| `ODOO.DEBIT` | "Get the total debit for the specified account(s) and period." |
| `ODOO.CREDIT` | "Get the total credit for the specified account(s) and period." |
| `ODOO.BALANCE` | "Get the total balance for the specified account(s) and period." |

| Argument | Kind | Help |
|---|---|---|
| `account_codes` | text | "The prefix of the accounts." |
| `date_range` | text or date | "The date range. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022"." |
| `offset` | number, default 0 | "Offset applied to the years." |
| `company_id` | number, optional | "The company to target (Advanced)." |
| `include_unposted` | logical, default false | "Set to TRUE to include unposted entries." |

### 7.5 Residual, partner balance, tagged balance

| Function | Description | Arguments |
|---|---|---|
| `ODOO.RESIDUAL` | "Return the residual amount for the specified account(s) and period" | `account_codes` (text, optional), `date_range` (text or date, optional), `offset`, `company_id`, `include_unposted` |
| `ODOO.PARTNER.BALANCE` | "Return the partner balance for the specified account(s) and period" | `partner_ids` (text), then the five arguments of the residual function |
| `ODOO.BALANCE.TAG` | "Return the balance of accounts for the specified tag(s) and period" | `account_tag_ids` (text), `date_range` (text or date, optional), `offset`, `company_id`, `include_unposted` |

Argument help specific to these three: for the optional codes, "The prefix of the accounts. If none provided, all receivable and payable accounts will be used."; for the partners, "The partner ids (separated by a comma)."; for the tags, "The tag ids (separated by a comma)."

### 7.6 Fiscal year and account groups

| Function | Description | Arguments |
|---|---|---|
| `ODOO.FISCALYEAR.START` | "Returns the starting date of the fiscal year encompassing the provided date." | `day` (date, help "The day from which to extract the fiscal year start."), `company_id` (number, optional, help "The company.") |
| `ODOO.FISCALYEAR.END` | "Returns the ending date of the fiscal year encompassing the provided date." | `day` (date, help "The day from which to extract the fiscal year end."), `company_id` (number, optional, help "The company.") |
| `ODOO.ACCOUNT.GROUP` | "Returns the account codes of a given group." | `type` (text), whose help reads "The technical account type (possible values are: %s)." with the placeholder replaced by the eighteen stored account-type values joined by a comma and a space |

The account-type argument has an assistant: while the author is typing the first argument of the account-group function, the eighteen stored values are proposed, each quoted, each described by its label, and the first proposal is pre-selected. The eighteen values and labels are in [`calculations.md`](calculations.md) §12.2.

### 7.7 Pivots

The pivot family of functions belongs to the workbook engine and is used unchanged. This domain contributes the data-bound pivot kind, whose stored value is `ODOO`, and with it: the dimension naming of [`calculations.md`](calculations.md) §16.1, the granularities of §16.2, the candidacy rules of §16.3, the aggregators of §16.4, and support for positional references, which are addressed by prefixing the dimension name with a number sign.

## 8. Routes

### 8.1 Routes this domain serves

| Address | Identity required | Method | Answer |
|---|---|---|---|
| `/spreadsheet/dashboard/data/<dashboard>` | a signed-in user | any | The reading payload of [`calculations.md`](calculations.md) §4, as a structured document; or the sample answer |
| `/dashboard/share/<share identifier>/<token>` | none | any | The public page of §5 |
| `/dashboard/data/<share identifier>/<token>` | none | `GET` | The stored frozen workbook, streamed exactly as stored |
| `/dashboard/download/<share identifier>/<token>` | a signed-in user | any | The packaged workbook file stored on the share, named after the dashboard |
| `/spreadsheet/log` | a signed-in user | `POST` | Nothing; the request writes an entry to the application log |
| `/board/add_to_dashboard` | a signed-in user | any | A logical value saying whether the element was pinned |

The first four are read-only routes: they are declared as such and may be served by a reader that cannot write.

Detailed behaviour, refusals and record effects:

| Address | Refusals | Rules |
|---|---|---|
| `/spreadsheet/dashboard/data/<dashboard>` | Not signed in: sent to sign in. Unknown or unreadable dashboard: not found | [SD-026](business-rules.md#sd-026) to [SD-029](business-rules.md#sd-029) |
| `/dashboard/share/<share identifier>/<token>` | Unknown share: not found. Bad token or revoked sharing user: forbidden, with the message of [SD-022](business-rules.md#sd-022) | [SD-022](business-rules.md#sd-022), [SD-025](business-rules.md#sd-025) |
| `/dashboard/data/<share identifier>/<token>` | The same two | The same |
| `/dashboard/download/<share identifier>/<token>` | Not signed in: sent to sign in. Bad token, revoked sharing user, or unknown share: forbidden. No export right: refused with the message of [SD-023](business-rules.md#sd-023) | [SD-022](business-rules.md#sd-022) to [SD-024](business-rules.md#sd-024) |
| `/spreadsheet/log` | None; an unaccepted operation name writes nothing and answers normally | [SD-058](business-rules.md#sd-058), [SD-059](business-rules.md#sd-059) |
| `/board/add_to_dashboard` | None; the four preconditions answer negatively rather than refusing | [SD-064](business-rules.md#sd-064) |

The dashboard reading route resolves its dashboard through the caller's own rights, so the record rules of [`configuration.md`](configuration.md) §5 apply before anything else happens. The three shared-dashboard routes resolve their share with elevated rights and then apply the access check of [`entities.md`](entities.md) §5.4 instead, because the caller may have no identity at all.

### 8.2 The logging request

The logging route takes two parts: the operation name, which must be one of the four reproduced values `download`, `copy`, `freeze` and `print`, and a list of data-source descriptions. Each description carries:

| Key | Meaning |
|---|---|
| `resModel` | The entity the source reads |
| `type` | The kind of source: `graph` for a data-bound chart, `pivot` for a data-bound pivot, `list` for a list |
| `fields` | For a chart, its measure; for a pivot, the field names of its measures; for a list, its columns |
| `groupby` | For a chart and a pivot, the groupings; absent for a list |
| `domain` | The record selection, including the conditions the global filters contributed |

The rendering of each description into one log line, and of the whole entry, is in [`business-rules.md`](business-rules.md#sd-059).

### 8.3 Addresses this domain's clients call

Four addresses are called by clients of this domain and served elsewhere. They are part of the contract a rebuild has to satisfy, so each is specified here.

| Address | Called by | What it must do |
|---|---|---|
| `/spreadsheet/data/<entity>/<identifier>` | The helper that builds a workbook model for any record carrying the mixin | Answer the record's workbook and its revision list |
| `/spreadsheet/xlsx` | The download action of §12.1 | Accept an archive name, the workbook-file parts and the data-source descriptions; package the parts into one archive; answer it as a download; and write the export log entry for the operation `download` |
| `/web/view/edit_custom` | The personal board, whenever its layout changes | Replace the layout of the named custom view, refusing when that record belongs to another user with the message "Custom view %(view)s does not belong to user %(user)s", the two placeholders being the record's identifier and the acting user's login name |
| `/web/action/load` | Each pinned element of the personal board | Answer the definition of the window action with that identifier |

The first two are workbook addresses and the last two are platform addresses. Where the behaviour above is not fully determined by this domain — the exact shape of the packaged archive, for instance — the resolution stated is the ordinary one: one archive entry per supplied part, at the path each part names, compressed, with any part that names a picture replaced by that picture's content. **Industry-standard default.**

## 9. Named operations

Operations a client may call by name. Each row states the entity it is called on, whether it addresses the whole entity or a set of records, and what it answers.

### 9.1 On the Spreadsheet Document mixin

| Operation | Addressing | Inputs | Answer | Notes |
|---|---|---|---|---|
| `get_display_names_for_spreadsheet` | the entity | a list of pairs of entity transport name and record identifier | one display name per input pair, in the same order | Read-only. Archived records are included. A pair naming a record that does not exist yields nothing in that position. Several entities may be mixed in one call |

### 9.2 On Spreadsheet Dashboard

| Operation | Addressing | Inputs | Answer | Notes |
|---|---|---|---|---|
| `action_toggle_favorite` | exactly one record | none | nothing | Adds or removes the acting reader in `favorite_user_ids`, with elevated rights. More than one record is refused by rule [SD-020](business-rules.md#sd-020) |

### 9.3 On Dashboard Share

| Operation | Addressing | Inputs | Answer | Notes |
|---|---|---|---|---|
| `action_get_share_url` | the entity | one structure holding at least `dashboard_id` and `spreadsheet_data`, and optionally `excel_files` | the full shared address | When workbook-file parts were supplied they are packaged into one archive and stored in `excel_export`, and the parts key is dropped before the record is created |

### 9.4 On Account

| Operation | Inputs | Answer |
|---|---|---|
| `spreadsheet_fetch_debit_credit` | a list of request structures, each with `date_range`, `codes`, `company_id`, `include_unposted` | one structure per request, each with `debit` and `credit`; read-only |
| `spreadsheet_fetch_residual_amount` | the same shape | one structure per request, each with `amount_residual`; read-only |
| `spreadsheet_fetch_partner_balance` | the same shape plus `partner_ids` | one structure per request, each with `balance` |
| `spreadsheet_fetch_balance_tag` | the same shape with `account_tag_ids` instead of `codes` | one structure per request, each with `balance` |
| `get_account_group` | a list of account types | one list of account codes per type, in the same order |
| `spreadsheet_move_line_action` | one request structure | a window action over Journal Item, list presentation, current target, titled "Cell Audit", carrying the selection of [`calculations.md`](calculations.md) §6 with the payable-and-receivable fallback on; read-only |

Every one of the six answers in the order of its inputs, whatever order the underlying grouping produced. An empty input list answers an empty list.

### 9.5 On Company, Currency, Currency Rate, Language and Model Definition

| Entity | Operation | Inputs | Answer |
|---|---|---|---|
| Company | `get_fiscal_dates` | a list of structures, each with `company_id` and `date` | one structure per input with `start` and `end`, or nothing for an input whose company does not exist; read-only |
| Currency | `get_company_currency_for_spreadsheet` | an optional company identifier | the code, the symbol, the decimal places and the position, or nothing; read-only |
| Currency Rate | `get_rates_for_spreadsheet` | a list of structures, each with `from`, `to`, and optionally `date` and `company_id` | each input structure copied and given a `rate`, which is nothing when the rate cannot be derived; read-only |
| Language | `get_locales_for_spreadsheet` | none | one locale structure per language record, including inactive ones; read-only |
| Model Definition | `has_searchable_parent_relation` | a list of entity transport names | for each name, whether that entity has a stored, searchable parent link; read-only. An entity the caller may not read answers false, as does an entity that does not exist |

### 9.6 On Dashboard Board

| Operation | Inputs | Answer |
|---|---|---|
| `get_view` | a view identifier and a view kind | The layout: the reader's own customised layout when one exists for that view, otherwise the shipped one, in both cases after the preprocessing of [`entities.md`](entities.md) §6.4, and with the identifier of the customised view when one was used |

### 9.7 The editing extension point

The workspace calls an operation named `action_edit_dashboard` on Spreadsheet Dashboard, passing one dashboard identifier, when the reader activates the control that a package registered in the dashboard-action registry (§14.1). The answer is expected to be an action, which the workspace opens. No package of this domain implements the operation; the control is not drawn when no package registered one, so the call is never made in an installation limited to this domain.

## 10. Cell menu entries

Four entries are contributed to the menu a reader opens on a cell. The sequence decides the order among all entries of that menu.

| Entry | Label | Sequence | Offered when | Effect |
|---|---|---|---|---|
| Pivot drill-through | "See records" | 175 | The cell holds exactly one pivot formula, the pivot is data-bound, the cell is neither empty nor in error, the cell is positioned on a real group, and a record selection can be derived | [`workflows.md`](workflows.md) §6.3 |
| Accounting audit | "See records" | 176 | The cell holds exactly one accounting formula and is neither empty nor in error | [`workflows.md`](workflows.md) §6.5 |
| List drill-through | "See record" | 200 | The cell holds exactly one list formula, it is the value formula rather than the header formula, and the cell is neither empty nor in error | [`workflows.md`](workflows.md) §6.2 |
| Filter from a pivot header | offered by the workbook engine | — | The cell is a pivot header cell and at least one filter is matched to the grouping at that position | [`global-filters.md`](global-filters.md) §9 |

Each of the three drill-through entries opens in a new window on a middle click.

The copy entries of the cell menu, the column menu, the row menu and the edit menu are disabled whenever the workbook is frozen.

## 11. Printing

A reader may print the open workbook, either with the platform's print shortcut — the control key or the command key together with the letter `p` — or from a menu entry that calls the same preparation.

The preparation:

1. Load the printing assets.
2. Record the current viewing rectangle, the current scroll position and the current presentation mode.
3. When the workbook is not already in dashboard presentation, switch it to dashboard presentation and wait until no drawing animation is running.
4. Scroll to the top left.
5. Resize the viewing rectangle so that it covers the whole used area of the active sheet: its width is the right edge of the last used column and its height is the bottom edge of the last used row.
6. Print.

After printing, the viewing rectangle, the scroll position and the presentation mode are restored, and — in the dashboard workspace — an export log entry is written for the operation `print`.

Printing therefore prints the whole of the active sheet, not the part of it that happens to be on screen.

## 12. Import and export

### 12.1 Downloading a workbook from inside the application

A client action, registered under the reproduced name `action_download_spreadsheet`, downloads a workbook as a workbook file. It is loaded lazily: the registered handler first loads the workbook bundle, which replaces the handler with the real one, and repeats the action. When the bundle fails to load, the handler is replaced by one that shows the notice "%s couldn't be loaded", with the action's name substituted, as a failure.

The real handler:

1. Checks that the acting user holds the export group. Without it, the notice of rule [SD-066](business-rules.md#sd-066) is shown with the title "Access Error" and nothing else happens.
2. When workbook-file parts were not supplied by the caller, builds a workbook model from the supplied document and revisions, waits for every data source to settle, collects the descriptions of the loaded data sources, and exports the workbook as file parts.
3. Posts the archive name, the parts and the descriptions to the workbook-file address of §8.3.

### 12.2 Downloading a shared dashboard's workbook file

Through the download route of §8.1. The stored archive is streamed under the share's name, which is the dashboard's name. A share created without workbook-file parts has an empty archive field and the stream is empty; this is not an error.

### 12.3 Uploading a workbook

The dashboard configuration list offers a workbook file control. It behaves as the platform's ordinary file control with two differences: it uses the workbook file name for the download name, and its download does nothing at all, so a reader cannot pull the raw document out of the control. Uploading runs the validation of [`entities.md`](entities.md) §2.5 immediately, while the row is still being edited.

### 12.4 The download file name

A record carrying the mixin computes a download name from its display name:

```formula
download file name = display name + ".osheet.json"
```

The suffix is reproduced exactly. It is recomputed whenever the display name changes and is never stored.

### 12.5 The filter sheet

Every workbook-file export, and every freeze, appends one sheet named "Active Filters" holding the filters and their values. Its layout is in [`document-format.md`](document-format.md) §9. A workbook with no global filter gains no such sheet.

### 12.6 What cannot be imported or exported

| Not offered | Note |
|---|---|
| Importing a dashboard from a workbook file | The workbook field accepts a packaged workbook file and stores it, and the validation lets it through without a reference walk, but nothing converts it into a live dashboard |
| Exporting a dashboard as a comma-separated file | Not offered by this domain |
| Exporting the dashboard records themselves | The platform's ordinary record export applies, subject to the access rights of [`configuration.md`](configuration.md) §3 |

## 13. External integrations

**None.** The domain contacts no service outside the installation. The three addresses it composes — the shared page, the shared data and the shared download — are addresses of the installation itself, built on the installation's own base web address.

The one thing that leaves the installation is the shared address itself, which the sharing reader copies to their clipboard and hands to whoever they choose. A clipboard that refuses the write is ignored and the address stays on screen.

## 14. Registries other packages plug into

### 14.1 The dashboard-action registry

A package may register a component that the workspace draws beside every dashboard entry in the sidebar. Only the first registered component is used. It receives the dashboard's identifier, the dashboard's data and a callback that calls `action_edit_dashboard` and opens whatever it answers.

### 14.2 The formula registry

A package may register a further workbook function. A function filed under the category `Odoo` appears in the formula list beside the fifteen of §7.

### 14.3 The chart registries

A package may register a further chart kind, a further variant of an existing kind, and a drawing component for it. A kind whose stored value begins with `odoo_` is treated as data-bound throughout: it is loaded before a freeze, it is turned into a picture by the freeze, and its descriptions are collected for the export log.

### 14.4 The cell menu registry

A package may register a further entry in the cell menu, with a label, a sequence, a visibility rule and an effect. The four entries of §10 are registered the same way.

### 14.5 The field-matching registry

A package that adds a new kind of data-bound element registers, under that element's kind, how a global filter reaches it: how to read its field matchings, how to write them, and how to apply the resulting conditions. The three kinds this domain registers are the list, the data-bound pivot and the data-bound chart.

### 14.6 The command palette

Inside a workbook that is not in dashboard presentation, every enabled entry of every top-bar menu is offered in the platform's command palette, named by its parent menu, a slash and its own label. The entry that inserts a link is filed in a category of its own so that it appears first. The shortcut that would otherwise open a link from the grid — the control key together with the letter `k` — is removed so that the palette can use it.

### 14.7 Dialogs raised by a workbook

A workbook raises three kinds of interruption through the platform: a confirmation, a notification and an error. The confirmation and the error are shown in a dialog titled "Odoo Spreadsheet", which is reproduced as the reader sees it. A confirmation that offers a cancellation labels its buttons "Yes" and "No"; one that does not labels its single button "Confirm".
