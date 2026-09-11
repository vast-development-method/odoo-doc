# Global filters

A global filter is a named control that belongs to a whole workbook. Setting it narrows every data-bound element of that workbook at once — every list, every pivot, every data-bound chart — each through its own path to the filtered field. A dashboard's search bar is nothing but the filters of its workbook; a shared dashboard carries the values that were set when it was frozen.

This document specifies the six kinds of filter, every operator each kind offers, the shape of a value under each operator, the record selection each produces, the default-value system, the field matching that connects a filter to an element, the commands that create and change filters, and the way filters appear on a frozen or exported workbook.

## 1. Where a filter lives

A filter is one entry of the `globalFilters` section of the stored workbook ([`document-format.md`](document-format.md) §1). Its definition is stored; its **value** is not. A value lives only for the duration of a reading session, except in a frozen workbook, where the value at freezing time is written into the definition under `value` as rendered text and into a dedicated sheet.

| Key of a filter definition | Kind | Meaning | Applies to |
|---|---|---|---|
| `id` | text | Stable identifier used by the field matchings and by the commands | all kinds |
| `label` | text | The name the reader sees, and the name a formula uses to address the filter | all kinds |
| `type` | text | The kind: `date`, `text`, `relation`, `selection`, `boolean` or `numeric` | all kinds |
| `defaultValue` | varies | The value used when the reader has set none; see §4 | all kinds |
| `modelName` | text | The entity whose records the filter selects | `relation` |
| `resModel` | text | The entity carrying the selection field | `selection` |
| `selectionField` | text | The field whose stored options the filter offers | `selection` |
| `rangesOfAllowedValues` | list of cell ranges | Ranges whose cell values are the only values offered | `text` |

## 2. The six kinds and their operators

An operator is part of the **value**, not of the definition: the same text filter may be used with `ilike` on one reading and with `in` on the next. The operators a kind offers, in the order they are offered, are:

| Kind | Operators, in order |
|---|---|
| `text` | `ilike`, `not ilike`, `in`, `not in`, `starts with`, `set`, `not set` |
| `relation` | `in`, `not in`, `child_of`, `ilike`, `not ilike`, `set`, `not set` |
| `selection` | `in`, `not in` |
| `boolean` | `set`, `not set` |
| `numeric` | `=`, `!=`, `>`, `<`, `between` |
| `date` | none; a date filter has no operator, only a period |

Every operator name is reproduced exactly. The two operators `set` and `not set` behave identically for every kind that offers them and are specified once, in §3.6.

## 3. Value shapes and record selection, operator by operator

Throughout this section, *the field path* means the path recorded for this filter on the element being narrowed — the `chain` of its field matching (§6). An element with no path for a filter is not narrowed by that filter at all.

### 3.1 Text, with `ilike` or `not ilike`

| Part | Value |
|---|---|
| Value shape | an operator and a list of texts under `strings` |
| Valid when | `strings` is a non-empty list, every entry a text |
| Empty value | an empty `strings` list |
| Record selection | the disjunction over the texts: a record is kept when the field path matches any one of them under the operator |
| Cell rendering | the texts joined by a comma and a space |
| Search-bar facet | the texts themselves |

### 3.2 Text or relation, with `in` or `not in`

| Part | Value |
|---|---|
| Value shape | an operator and a list of texts under `strings` (text kind) or a list of record identifiers under `ids` (relation kind) |
| Valid when | the list is non-empty and every entry is of the right sort — a text, or a whole number |
| Record selection | one condition: the field path stands in, or does not stand in, the whole list |
| Cell rendering | the entries joined by a comma and a space |
| Search-bar facet | for texts, the texts; for records, their display names, each unreadable or missing record replaced by the text "Inaccessible/missing record ID" |

### 3.3 Text, with `starts with`

| Part | Value |
|---|---|
| Value shape | an operator and a list of texts under `strings` |
| Record selection | the disjunction over the texts: a record is kept when the field path begins with any one of them, matched without regard to letter case |
| Cell rendering and facet | as in §3.1 |

### 3.4 Relation, with `child_of`

| Part | Value |
|---|---|
| Value shape | an operator and a list of record identifiers under `ids` |
| Record selection | one condition: the field path is one of the named records or a descendant of one of them |
| Offered only when | the filtered entity has a stored, searchable parent link, which the operation `has_searchable_parent_relation` of [`entities.md`](entities.md) §8.6 answers |

### 3.5 Relation, with `ilike` or `not ilike`

A relational filter may also be used by text rather than by record: the value is a list of texts under `strings`, and the record selection is the disjunction of matches of the field path against each text. This is what allows a filter over a related entity to survive a workbook being opened by a reader who cannot read that entity's records.

### 3.6 Any kind that offers them, with `set` or `not set`

| Part | Value |
|---|---|
| Value shape | an operator alone, with no further part |
| Valid when | the operator is exactly `set` or exactly `not set` |
| Record selection | for `set`, the field path is different from the empty value; for `not set`, the field path equals the empty value |
| Cell rendering | the logical value true for `set`, false for `not set` |
| Search-bar facet | the operator's own label |

This is the only value shape a `boolean` filter has.

### 3.7 Selection, with `in` or `not in`

| Part | Value |
|---|---|
| Value shape | an operator and a list of stored option values under `selectionValues` |
| Valid when | the list is non-empty and every entry is a text |
| Record selection | one condition: the field path stands in, or does not stand in, the list of stored option values |
| Cell rendering | the stored option values joined by a comma and a space |
| Search-bar facet | the labels of those options, read from the selection field of `resModel`; an option whose value is not among the field's options is shown as its raw stored value |

A selection filter whose `selectionField` does not exist on `resModel` is a definition error and the search bar refuses to describe it.

### 3.8 Numeric, with `=`, `!=`, `>` or `<`

| Part | Value |
|---|---|
| Value shape | an operator and one number under `targetValue` |
| Valid when | `targetValue` is a number |
| Empty value | `targetValue` absent |
| Record selection | one condition comparing the field path to the number under the operator |
| Cell rendering | the number, or an empty cell when absent |
| Search-bar facet | the number written out, or nothing when absent |

### 3.9 Numeric, with `between`

| Part | Value |
|---|---|
| Value shape | an operator, a number under `minimumValue` and a number under `maximumValue` |
| Valid when | both are numbers |
| Empty value | both absent |
| Record selection | two conditions joined by conjunction: the field path is at least the minimum and at most the maximum |
| Cell rendering | two cells, one under the other: the minimum then the maximum |
| Search-bar facet | the two numbers written as a list |

### 3.10 Date

A date filter has no operator. Its value is one of five period shapes:

| Shape | Parts | Valid when |
|---|---|---|
| `relative` | a `period` naming one of the nine relative periods of §5.1 | the period is one of the nine |
| `month` | a `year` and a `month` | both are numbers and the month is between 1 and 12 |
| `quarter` | a `year` and a `quarter` | both are numbers and the quarter is between 1 and 4 |
| `year` | a `year` | the year is a number |
| `range` | an optional `from` and an optional `to`, each a date written as text | each present part is a text |

Any other shape is invalid and the value is refused.

The record selection is built in three steps:

1. The period is turned into a first moment and a last moment by the rule of [`calculations.md`](calculations.md) §17, shifted by the element's own period offset (§6.3).
2. Each moment is written as a date when the matched field is a date, and as a date and time when it is a date and time.
3. The conditions are: both moments present gives the conjunction of "at least the first" and "at most the last"; only the first gives "at least the first"; only the last gives "at most the last"; neither gives no condition at all.

The cell rendering of a date filter is **two cells, one under the other**: the first moment as a date, then the last moment as a date, each carrying the reader's date format. A missing moment renders as an empty cell.

## 4. Default values

A filter may carry a default. The default applies whenever the reader has not set a value and has not explicitly cleared the filter.

| Kind | Valid default |
|---|---|
| `text` | the same shape as a value: an operator and a list of texts |
| `selection` | an operator and a list of stored option values |
| `numeric` | an operator and its number or numbers |
| `boolean` | an operator, `set` or `not set` |
| `relation` | an operator and either a list of record identifiers or the single marker `current_user` in place of the list |
| `date` | one of twelve period names: the nine relative periods of §5.1, or `this_month`, `this_quarter`, `this_year` |

A default that does not match the table is refused when the filter is created or edited, with rule [SD-022](business-rules.md#sd-022).

### 4.1 Resolving a default at reading time

1. When the reader has set a value, that value is used and the default is ignored.
2. When the reader has explicitly cleared the filter, no value is used, whatever the default says.
3. When the filter has no default, no value is used.
4. Otherwise the default is resolved:
   - a date default of `this_year` becomes a `year` value carrying the current year;
   - a date default of `this_month` becomes a `month` value carrying the current year and the current month;
   - a date default of `this_quarter` becomes a `quarter` value carrying the current year and the quarter containing the current month;
   - a date default naming one of the nine relative periods becomes a `relative` value carrying that period;
   - a relational default whose list is the marker `current_user` becomes the same default with the reader's own user identifier as its single record identifier;
   - every other default is used unchanged.

**Why the marker rather than an identifier.** A dashboard shipped with "assigned to me" pre-selected cannot store a user identifier, because the identifier would be that of whoever built it. The marker defers the decision to reading time, and each reader sees their own records.

### 4.2 Empty values

Each operator has an *empty* value — the shape with nothing chosen: an empty text list, an empty identifier list, an empty selection list, an absent number, absent bounds. The `set` and `not set` operators and the date kind have no empty value, because the operator alone is already a choice. A value is considered empty when every part of it equals the corresponding part of the empty value for its operator; an empty value leaves the elements unnarrowed but still counts as set, so the default does not come back.

## 5. Periods

### 5.1 The nine relative periods

| Stored value | Label |
|---|---|
| `today` | "Today" |
| `yesterday` | "Yesterday" |
| `last_7_days` | "Last 7 Days" |
| `last_30_days` | "Last 30 Days" |
| `last_90_days` | "Last 90 Days" |
| `month_to_date` | "Month to Date" |
| `last_month` | "Last Month" |
| `last_12_months` | "Last 12 Months" |
| `year_to_date` | "Year to Date" |

The stored values are reproduced exactly; the labels are the texts the reader sees.

### 5.2 Turning a period into a window

The arithmetic that turns each period, and each fixed shape, into a first moment and a last moment — including the effect of a non-zero offset — is specified with worked examples in [`calculations.md`](calculations.md) §17.

### 5.3 Moving to the next or the previous period

The search bar offers a step forward and a step back. The result depends on the shape:

| Current shape | Next | Previous |
|---|---|---|
| `quarter` | the following quarter, rolling the year over after the fourth | the preceding quarter, rolling the year back before the first |
| `month` | the following month, rolling the year over after the twelfth | the preceding month, rolling the year back before the first |
| `year` | the year plus one | the year minus one |
| `range` | both ends moved forward by the length of the range in days, the length counting both ends | both ends moved back by the same length |
| `relative`, one of `today`, `yesterday`, `last_7_days`, `last_30_days`, `last_90_days` | the same relative window shifted one step forward, converted to a fixed `range` | the same window shifted one step back, converted to a fixed `range` |
| `relative`, `last_12_months` | the window shifted twelve months forward, snapped to whole months, converted to a fixed `range` | the same shifted back |
| `relative`, `last_month` | a `month` value for the current month | a `month` value for the month two months before the current one |
| `relative`, `month_to_date` | a `month` value for the month after the current one | a `month` value for the month before the current one |
| `relative`, `year_to_date` | a `year` value for the year after the current one | a `year` value for the year before the current one |
| a `range` with neither end | unchanged | unchanged |

Stepping away from a relative period therefore fixes it: a reader who steps back from "Last 7 Days" lands on a concrete range, and stepping forward again returns to that range moved, not to the relative period.

### 5.4 Naming a period

| Shape | Rendered name |
|---|---|
| `relative` | the label of the period from §5.1 |
| `month` | the month name and the year, as in "January 2026" |
| `quarter` | "Q" followed by the quarter number, a space, and the year |
| `year` | the year alone |
| `range` with both ends | the two dates written in full and joined by a dash |
| `range` with only a first end | "Since" followed by the date written in full |
| `range` with only a last end | "Until" followed by the date written in full |
| no value at all | "All time" |

## 6. Field matching

### 6.1 What it is

A filter names a concept — a period, a salesperson, a country. An element names an entity. The field matching is the bridge: for each pair (filter, element) it records how that filter reaches that element's entity.

| Part | Meaning |
|---|---|
| `chain` | The dotted field path from the element's entity to the field the filter narrows |
| `type` | The kind of the field at the end of the path, which decides how a date is written and how a value is compared |
| `offset` | Date filters only: a whole number of periods by which this element's window is shifted |

### 6.2 Validity

A matching that carries a non-zero offset but no `chain` or no `type` is refused, with rule [SD-022](business-rules.md#sd-022). A matching with no `chain` at all is valid and simply means that this filter does not narrow this element.

### 6.3 The offset

The offset lets one workbook show the same measure for several periods side by side: a filter set to a year, one pivot with offset zero and another with offset minus one, and the workbook shows this year beside last year without a second filter. The offset is applied when the period is turned into a window, before the window is turned into conditions, and its unit is the unit of the period: a `month` value shifts by months, a `quarter` value by quarters, a `year` value by years, a `range` value not at all, and a relative period by its own natural step, as set out in [`calculations.md`](calculations.md) §17.

### 6.4 Proposing a matching for a new element

When an element is added to a workbook that already has filters, the matchings of an existing element **over the same entity** are copied to it — the `chain` and the `type`, never the `offset`. An entity that no existing element uses gets no proposal and the author matches the fields by hand.

## 7. Commands

| Command | Refused when | Refusal marker |
|---|---|---|
| `ADD_GLOBAL_FILTER` | the label is empty | `InvalidFilterLabel` |
| | another filter already carries the same label | `DuplicatedFilterLabel` |
| | the default value does not match the kind and operator | `InvalidValueTypeCombination` |
| `EDIT_GLOBAL_FILTER` | no filter carries the given identifier | `FilterNotFound` |
| | the label is empty | `InvalidFilterLabel` |
| | another filter already carries the same label | `DuplicatedFilterLabel` |
| | the default value does not match the kind and operator | `InvalidValueTypeCombination` |
| `REMOVE_GLOBAL_FILTER` | no filter carries the given identifier | `FilterNotFound` |
| `MOVE_GLOBAL_FILTER` | no filter carries the given identifier | `FilterNotFound` |
| | the requested position is before the first or after the last | `InvalidFilterMove` |
| `SET_GLOBAL_FILTER_VALUE` | no filter carries the given identifier | `FilterNotFound` |
| | the value does not match the kind and operator | `InvalidValueTypeCombination` |
| | the value is identical to the current one | `NoChanges` |
| `SET_MANY_GLOBAL_FILTER_VALUE` | each contained value is checked as above | as above |
| `SET_DATASOURCE_FIELD_MATCHING` | a matching carries an offset without a path or a type | `InvalidFieldMatch` |

The refusal markers are reproduced exactly; a client maps them to the text it shows.

Adding a filter appends it at the end of the list. Removing a filter also discards any value the reader had set for it. Moving a filter changes its position by the requested number of places.

A text filter that carries ranges of allowed values has those ranges kept in step with the sheet: a range that is resized, moved or otherwise changed is updated; a range that disappears is dropped, and a filter that loses all its ranges stops restricting its values.

## 8. Setting, clearing and counting

Three states are possible for the pair (filter, reading session), and they are specified as a machine in [`state-machines.md`](state-machines.md) §7:

1. **Untouched** — the reader has set nothing; the default applies.
2. **Set** — the reader has set a value; that value applies.
3. **Cleared** — the reader has explicitly emptied the filter; nothing applies, and the default does not return.

A filter is *active* when resolving it yields a value. The active count shown beside the search bar is the number of filters in that condition.

## 9. Narrowing from a pivot cell

A reader who opens the context menu on a pivot **header** cell may ask the workbook to set the filters that correspond to that header. The rule is:

1. Take the last part of the cell's pivot position. A cell positioned on a measure rather than on a grouping offers nothing.
2. For each filter, take the matching recorded for this pivot. Keep the filter only when its path is exactly the field name of that last part.
3. Read the grouping value at that position. A position that holds no record offers nothing.
4. Translate the value according to the filter kind:
   - **date** — only when the grouping carries a period granularity. The grouping value's last slash-separated part is read as a year; an unreadable year offers nothing. A yearly grouping becomes a `year` value; a monthly grouping becomes a `month` value, falling back to a `year` value when the month part is outside one to twelve; a quarterly grouping becomes a `quarter` value, falling back to a `year` value when the quarter part is outside one to four. A grouping value of `false`, which is what a group with no date carries, offers nothing. A translated value identical to the value already set offers nothing.
   - **relation** — the grouping value is read as a record identifier; a value of `false`, meaning "none", offers nothing. When the identifier is not the one already set, the proposal is the operator `in` with that single identifier.
   - **text** — when the grouping value is not the one already set, the proposal is the operator `ilike` with that single text.
   - any other kind offers nothing.
5. The proposals are applied together, as one command.

The control is offered only when at least one filter matches and the cell is a header cell.

## 10. Filters on a frozen or exported workbook

Freezing a dashboard, and exporting one as a workbook file, both append a sheet named "Active Filters" built by the rule of [`document-format.md`](document-format.md) §9. In addition, freezing writes each filter's current value into the definition as rendered text: the cells of its display value, each formatted with its own number format and the reader's locale, with empty renderings dropped, joined by a comma and a space.

The public page of a shared dashboard shows only those filters whose rendered value is not empty, and shows them as read-only information rather than as controls: a frozen dashboard cannot be re-filtered, because it no longer holds the queries that filtering would change.

## 11. Addressing a filter from a cell

Three workbook functions read filters; they are listed with their arguments in [`interfaces.md`](interfaces.md) §7 and specified here.

- `ODOO.FILTER.VALUE` takes a filter label and yields that filter's display value — one cell for most kinds, two stacked cells for a date filter and for a numeric filter using `between`. A label that matches no filter raises rule [SD-021](business-rules.md#sd-021). The label is matched after both the given label and each filter's label have been passed through the workbook's own translation, so that a formula written in one language keeps working in another. Escaped quotation marks in the given label are unescaped before matching.
- `ODOO.FILTER.LABEL` takes the same argument and yields a *name* rather than a value: for a relational filter, the display names of the selected records joined by a comma and a space, or the raw value when nothing is selected; for a date filter whose window covers whole months, a compact name — the month and year when the window is one month, "Q" and the quarter and the year when it is one quarter, the year alone when it spans a whole year — and otherwise the two stacked dates; for every other kind, the display value unchanged.
- `ODOO.FILTER.VALUE.V18` is the same rule as `ODOO.FILTER.LABEL` under a second name. It exists so that a workbook written before the two were separated keeps producing what it produced then. It is not offered in the function list.
