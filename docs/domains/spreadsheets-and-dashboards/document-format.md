# The stored workbook

A workbook is one structured document. It is the value of `spreadsheet_binary_data` on every entity that declares the Spreadsheet Document mixin, and it is the payload every route of this domain moves. This document specifies its sections, the parts of it that are peculiar to this system rather than to spreadsheets in general, the validation walk that protects it, the empty workbook a new record starts with, and the frozen workbook a share carries.

The section names below are reproduced exactly, because a workbook written by one installation has to be readable by another.

## 1. Top-level sections

| Key | Kind | Meaning | Written by |
|---|---|---|---|
| `sheets` | list | The sheets of the workbook, in tab order | the engine |
| `settings` | structure | Workbook-wide settings; carries `locale` | the engine |
| `revisionId` | text | A marker identifying the revision the stored content corresponds to | the engine |
| `formats` | map from number to text | The number formats used anywhere in the workbook, each under a small integer key | the engine |
| `styles` | map from number to structure | The cell styles used anywhere in the workbook, each under a small integer key | the engine |
| `lists` | map from text to structure | The data-bound list elements, each under its own element identifier | this domain |
| `listNextId` | number | The identifier the next inserted list will take | this domain |
| `pivots` | map from text to structure | The pivot elements, each under its own element identifier; a pivot may be data-bound or ordinary | this domain and the engine |
| `globalFilters` | list | The global filters of the workbook, in presentation order | this domain |
| `chartOdooMenusReferences` | map from text to text | For each chart element identifier, the external identifier of the menu that chart links to | this domain |
| `odooVersion` | number | A compatibility marker for the shape of the filter section | this domain |
| `[Content_Types].xml` | text | Present only when the stored content is a workbook file rather than a native document; its presence exempts the content from the reference walk | an export |

A workbook that carries `[Content_Types].xml` is a packaged workbook file. The validation accepts it after decoding and performs no reference walk, because such a file has no live references left to check.

## 2. A sheet

| Key | Kind | Meaning |
|---|---|---|
| `id` | text | Stable identifier of the sheet, used by formulas; never translated |
| `name` | text | The tab label; translated for the creating user in a new workbook |
| `cells` | map from cell reference to cell | The cells that have content |
| `figures` | list | The floating elements laid over the grid |
| `formats` | map from cell reference to number | The format key used by each formatted cell |
| `styles` | map from cell reference to number | The style key used by each styled cell |
| `colNumber` | number | Number of columns |
| `rowNumber` | number | Number of rows |

A cell entry is either a text — the raw content, which is a formula when it begins with an equals sign — or a structure carrying that text under `content` together with further cell properties. Both shapes are read; the plain-text shape is the one a current workbook writes.

## 3. A figure

| Key | Kind | Meaning |
|---|---|---|
| `id` | text | Identifier of the figure |
| `tag` | text | The kind of figure: `chart`, `carousel` or `image` |
| `data` | structure | The definition, whose shape depends on `tag` |
| `width`, `height` | numbers | Size in points |

A figure tagged `chart` carries a chart definition whose `type` names the chart kind. A chart whose `type` begins with `odoo_` is data-bound: it queries records rather than reading cells. A figure tagged `carousel` carries several chart definitions under `chartDefinitions`, keyed by chart identifier, and any of them may be data-bound. A figure tagged `image` carries a picture under `path` and a size under `size`; freezing turns every data-bound chart into a figure of this kind.

The twelve data-bound chart kinds are listed in [`interfaces.md`](interfaces.md) §6.

## 4. Data-bound elements

### 4.1 A list element

| Key | Kind | Meaning |
|---|---|---|
| `model` | text | The entity the list reads |
| `columns` | list of text | The field paths shown, in column order |
| `domain` | structure | The record selection the list applies |
| `context` | structure | The reading context, with the reader's own context keys removed |
| `orderBy` | list | The ordering, each entry a field name and an ascending flag |
| `name` | text | The element's display name |
| `actionXmlId` | text | The external identifier of the window action a drill-through should reuse, when one is known |

A list element is referenced from cells by its key in `lists`, and its cells are written by the two list formulas of [`interfaces.md`](interfaces.md) §7.

### 4.2 A pivot element

| Key | Kind | Meaning |
|---|---|---|
| `type` | text | `ODOO` for a data-bound pivot; any other value marks a pivot over cell ranges, which this domain does not own |
| `model` | text | The entity the pivot reads |
| `columns` | list | The column dimensions, each with a `fieldName` and an optional `granularity` |
| `rows` | list | The row dimensions, same shape |
| `measures` | list | The measures, each with an identifier, a `fieldName` and an aggregator; a measure carrying `computedBy` is computed inside the workbook and is not fetched |
| `domain` | structure | The record selection |
| `context` | structure | The reading context |
| `sortedColumn` | structure | The column the pivot is sorted by, naming a measure |
| `collapsedDomains` | structure | Which groups are collapsed |
| `customFields` | structure | Groupings defined inside the workbook rather than on the entity |
| `actionXmlId` | text | As for a list |

Older workbooks name the two dimension lists `colGroupBys` and `rowGroupBys` and hold plain field names in them; both spellings are read.

### 4.3 A data-bound chart definition

| Key | Kind | Meaning |
|---|---|---|
| `type` | text | The chart kind, beginning with `odoo_` |
| `metaData` | structure | Carries `resModel`, `groupBy` and `measure` |
| `searchParams` | structure | Carries `domain` and `groupBy` |
| `fieldMatching` | structure | How each global filter reaches this chart's entity |

### 4.4 Field matching

Every data-bound element carries, for each global filter, the path by which that filter reaches the element's entity. The entry has three parts: `chain`, the dotted field path from the element's entity to the filtered field; `type`, the kind of that field; and, for date filters only, `offset`, a whole number of periods by which this element's window is shifted relative to the filter's own window. Field matching is specified in [`global-filters.md`](global-filters.md) §6.

## 5. Links out of a cell

A cell whose content is a link carries it in the ordinary link notation: a label in square brackets followed by a target in parentheses. Three target prefixes are peculiar to this system and are reproduced exactly:

| Prefix | Target | Meaning |
|---|---|---|
| `odoo://view/` | a structure describing an action | Opens a view: the remainder of the target is the action description, carrying `modelName`, `domain`, `context`, `views` and an ordering |
| `odoo://ir_menu_id/` | a menu identifier | Opens the menu with that internal identifier |
| `odoo://ir_menu_xml_id/` | a menu external identifier | Opens the menu with that external identifier |

A fourth target, `neutralized:link`, is written by the freeze algorithm. It renders as a label with no destination: it is not external, its address cannot be edited, it presents no address, and following it does nothing. Every one of the three prefixes above is rewritten to it when a dashboard is frozen for sharing, so that a reader outside the application is never offered a link into an application they cannot reach.

## 6. The empty workbook

A record created without a workbook receives exactly this:

| Section | Value |
|---|---|
| `sheets` | one sheet, with `id` set to `sheet1` and `name` set to the translated word "Sheet1" |
| `settings` | one key, `locale`, holding the creating user's locale as computed in [`calculations.md`](calculations.md) §3 |
| `revisionId` | `START_REVISION` |

The sheet identifier and the revision marker are fixed text. The sheet name is translated, so that a French-speaking author's first sheet is named in French; the identifier stays `sheet1` so that a formula written against it keeps working after the reader changes language.

## 7. The reference walk

The second stage of the workbook validation of [`entities.md`](entities.md) §2.5 walks the document and collects two sets: the field paths it names, grouped by entity, and the menu external identifiers it names. Each is then checked.

### 7.1 Collecting field paths

Field paths are collected from five places, and the results are merged per entity.

1. **Each list element.** Its entity is `model`; its paths are the `columns`, the field names in its `orderBy`, and every field name appearing in its `domain`.
2. **Each data-bound pivot element.** Its entity is `model`; its paths are the column dimensions, the row dimensions, the measures that are not computed inside the workbook and that are not the record count, and every field name appearing in its `domain`. When the pivot carries a `sortedColumn` whose measure is not the record count, that measure is added.
3. **Each data-bound chart.** Its entity is the `resModel` of its `metaData`; its paths are the groupings of the `metaData`, the groupings of the `searchParams`, every field name in the `domain` of the `searchParams`, and the `measure` unless it is the record count.
4. **Each view link in a cell.** Its entity is the `modelName` of the action description; its paths are every field name in the action's `domain`.
5. **Each global filter matching.** For every data-bound element, the `chain` recorded for each filter is added under that element's entity.

A workbook whose `odooVersion` is below five records filter matchings differently — under `pivotFields`, `listFields` and `graphFields` on each filter, keyed by element identifier, each holding a `field` — and the walk reads that shape instead for such a workbook.

Every collected path has its aggregator suffix removed before checking: a measure written as a field name, a colon and an aggregator is checked as the field name alone.

### 7.2 Checking field paths

For each entity and each path:

1. An entity that does not exist produces the finding `- model '<entity>' used in '<record name>' does not exist`, where the record name is the display name of the record being validated.
2. Otherwise the path is split on full stops and walked left to right from that entity. A segment that is not a field of the entity reached so far produces the finding `- field '<segment>' used in spreadsheet '<record name>' does not exist on model '<entity reached so far>'`, and the walk of that path continues with the next segment. A segment that is a relational field moves the walk to the entity it targets.

### 7.3 Checking menus

Menu external identifiers are collected from two places: the values of `chartOdooMenusReferences`, and every cell link whose target begins with `odoo://ir_menu_xml_id/`, whose remainder is the identifier.

For each identifier:

1. An identifier that resolves to no record produces the finding `- xml id '<identifier>' used in spreadsheet '<record name>' does not exist`.
2. A menu that resolves but that has no action **and** has a parent produces the finding `- menu with xml id '<identifier>' used in spreadsheet '<record name>' does not have an action`. A root menu is exempt, because a root menu always has an action in practice.

### 7.4 Reporting

When the walk produced no finding, the workbook is accepted. When it produced findings, they are joined with line breaks and reported together through rule [SD-002](business-rules.md#sd-002).

### 7.5 When the walk runs

The walk runs only while the installation is executing its automated test suite. In ordinary operation only the decoding stage of the validation runs.

**Compatibility finding.** The consequence is that an ordinary installation accepts a workbook naming a field that no longer exists, and the failure surfaces later as a cell error rather than as a refused save. A corrected behaviour would run the walk on every interactive save as well, and report the findings as warnings rather than as a refusal, so that an author who removes a field learns about the broken dashboards immediately while still being able to store work in progress. The behaviour is recorded here as observed.

## 8. The frozen workbook

Freezing turns a live dashboard into a snapshot. The algorithm is specified step by step in [`calculations.md`](calculations.md) §19; its effect on the document is:

| Section | Effect of freezing |
|---|---|
| `cells` | Every cell whose content is a formula calling a function of this domain, or the pivot family, or the workbook translation helper, is replaced by the text of its evaluated value. A cell that evaluated to nothing becomes the formula that yields an empty text. A formula that spills across a region has every cell of that region replaced as well |
| `formats` | Each replaced cell keeps the number format its formula produced, registered in the workbook's format table |
| cell links | Every link into the application becomes `neutralized:link`, keeping its label |
| `figures` | Every data-bound chart, and every carousel containing one, becomes an image figure holding a picture of the chart drawn on a white background at the figure's own size |
| `pivots` | Every data-bound pivot definition is dropped; the cells that referred to it now hold values |
| `lists` | Emptied completely |
| `globalFilters` | Kept, each gaining a `value` holding its current value rendered as text; values that render empty are omitted from that text |
| `sheets` | One sheet is appended, named "Active Filters", holding the filters and their values |

## 9. The filter sheet

The appended sheet has two heading cells, `Filter` in the first column and `Value` in the second, both bold. Then one block per filter, in the order the filters appear in the workbook:

1. The filter's label is written in the first column.
2. The filter's display value is written to the right of it. A display value that occupies two rows — which is the case for a date filter, whose value is a first date and a last date — occupies two rows here too, and the next filter starts below them.
3. The number of columns of the sheet grows to fit the widest display value, with a minimum of two.

The sheet is created with the same name in a workbook-file export, where it is appended as the last sheet.

## 10. Extending a stored workbook without parsing it

A serialised workbook is often large, and a consumer often has to add a handful of keys to it — a revision list, a default currency, a translation namespace — without paying to parse and re-serialise the whole document. The spreadsheet engine therefore offers a text-level extension rule that any consumer of a stored workbook may use:

1. Trim the serialised document of surrounding blank characters.
2. Remove its final closing brace.
3. When what remains is not the empty document, append a comma.
4. Append each key, a colon and the already-serialised value, separating consecutive pairs by commas.
5. Append a closing brace.

The values appended must already be serialised; the rule performs no conversion. Worked examples are in [`calculations.md`](calculations.md) §20.
