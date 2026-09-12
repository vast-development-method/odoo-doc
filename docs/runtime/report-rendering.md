# Report rendering

Printed documents, electronic-mail bodies, portal pages and the cards of a board layout are all produced by one **document template engine**: a small declarative grammar in which the markup of the result is written directly and the dynamic parts are expressed as reserved attributes. This document specifies that grammar completely, the value formatters it uses to render a field, the pipeline that turns a print request into a page-numbered document carrying the company's letterhead, the page geometries and the shipped letterhead designs, the barcode and quick response code generator, the label sheets, and the two data-export formats.

The grammar is language-neutral: a replacement may compile it, interpret it, or translate it into another form, as long as the observable output is the same. The declarative layout grammar of screens is in [`../overview/views-and-actions.md`](../overview/views-and-actions.md); the inheritance mechanism that lets a capability package modify a shipped template is the same one used for screen layouts and is specified there. The compiled-template cache and the asset-bundle cache are in [`caching.md`](caching.md), sections 10 and 14. The language chosen for a render is resolved as specified in [`translation.md`](translation.md).

Throughout this document, an attribute name written in code font, such as `t-if`, is a reproduced identifier: a replacement that must read templates written for the original has to accept exactly that spelling.

## 1. The template grammar

### 1.1 Shape

A template is a well-formed element tree. Every element and every attribute that is **not** a directive is copied to the output as written. A directive is an attribute whose name begins with `t-`. A directive attached to an element governs that element.

The reserved element name `t` produces no output of its own. It exists in order to carry a directive without adding an element to the result: an element named `t` that carries a conditional emits its children when the condition holds and nothing at all otherwise.

A template has a name, which is either its external identifier or the value of its `t-name` attribute when it sits inside a container of several templates. A template is stored either as a view definition whose kind is the template kind, or read from a capability package's template file.

### 1.2 Evaluation order

Several directives may sit on one element. They are always evaluated in this fixed order, which is what makes an element carrying both a loop and a condition mean "loop, then test", and never the reverse.

| Step | Directive | Note |
|---|---|---|
| 1 | `t-elif` | Consumed by the preceding `t-if`. |
| 2 | `t-else` | Consumed by the preceding `t-if`. |
| 3 | `t-debug` | |
| 4 | `t-groups` | |
| 5 | `t-as`, `t-foreach` | The loop is entered before anything below is evaluated. |
| 6 | `t-if` | Therefore evaluated once per iteration. |
| 7 | `t-call-assets` | |
| 8 | `t-lang` | |
| 9 | `t-options` | |
| 10 | `t-call` | |
| 11 | `t-att` and every `t-att-` and `t-attf-` variant | |
| 12 | `t-field`, `t-esc`, `t-raw`, `t-out` | |
| 13 | the opening tag | Internal. |
| 14 | `t-set` | Therefore a variable set on an element is visible to that element's children but not to its own attributes. |
| 15 | the inner content | Internal. |
| 16 | the closing tag | Internal. |

An element that carries no directive at all and whose attributes are all static is emitted verbatim, without any work at render time.

After every directive has been consumed, no attribute whose name begins with `t-` may remain on the element. One that does is a template failure, which is what turns a misspelt directive into an immediate error rather than into a silent attribute in the output.

### 1.3 Conditionals

| Directive | Value | Meaning |
|---|---|---|
| `t-if` | An expression. | The element, its attributes and its content are emitted only when the expression is true. |
| `t-elif` | An expression. | Emitted when every preceding test of the chain was false and this one is true. It must immediately follow an element carrying `t-if` or `t-elif`, ignoring whitespace and comments between them. |
| `t-else` | Nothing; the attribute carries an empty value. | Emitted when every preceding test of the chain was false. The same placement rule applies. |

The chain is detected when the template is compiled, not when it is rendered: the element carrying `t-if` looks at its following siblings and consumes the ones that carry `t-elif` or `t-else`, marking them as already compiled so that they are not emitted a second time in their own right.

**Worked example.** A variable is set to 1. A first element carries a condition that is false and holds the text `10`. The next sibling carries `t-elif` testing that the variable equals 1, together with a loop over the three integers 0, 1 and 2 and an output of the loop variable. The output is three elements holding `0`, `1` and `2`, because the loop runs **inside** the branch that the test selected, and the branch is selected once for the whole chain.

### 1.4 Loops

`t-foreach` takes an expression producing the collection and requires `t-as` on the same element naming the loop variable. Writing one without the other is a template failure.

| Collection kind | The loop variable holds |
|---|---|
| An ordered list | Each element in order. |
| A mapping | Each key. |
| A record set | Each record, one at a time. |
| An integer | Each integer from zero up to, but excluding, that value. |

Inside the body, with the loop variable named by `t-as`, five further names exist, each formed from the loop variable's name and a fixed suffix.

| Suffix | Value |
|---|---|
| `_value` | The current value. Identical to the loop variable for a list, a record set and an integer; for a mapping it is the value while the loop variable is the key. |
| `_index` | The zero-based position. |
| `_size` | The size of the collection, when it is known. |
| `_first` | True when the position is zero. |
| `_last` | True when the position is the last one. It requires the size to be known. |

**Scope rule.** Names created inside the loop body exist only inside it. A name that already existed **outside** the loop keeps, after the loop, the value it had at the end of the last iteration; a name created inside the loop does not exist after it. This is what lets a template accumulate a total across the iterations by setting the accumulator before the loop, while a scratch variable set inside the loop leaves nothing behind.

### 1.5 Variables

| Directive | Value | Meaning |
|---|---|---|
| `t-set` | A name. | Declares or reassigns a name. |
| `t-value` | An expression. | The value assigned. |
| `t-valuef` | A format string. | The value assigned, produced by substitution as specified in section 1.7. |
| `t-valuef.translate` | A format string. | The same, with the literal parts translated. |

When neither `t-value` nor `t-valuef` is written, the **rendered body** of the element becomes the value, and that value is markup-safe, which is what makes it safe to emit later without being escaped a second time.

### 1.6 Output

| Directive | Value | Meaning |
|---|---|---|
| `t-out` | An expression. | Emits the value. A value that is already markup-safe is emitted as it is; any other value is escaped. When the value is empty or false, the element's **body** is emitted instead, as a default. |
| `t-field` | A path made of a record name, a full stop and a field name. | Emits the value of that field of that record, formatted by the converter of section 2. The body is the default when the value is empty. |
| `t-esc` | An expression. | An older spelling of `t-out`, kept because templates in the field use it. |
| `t-raw` | An expression. | An older spelling of `t-out` that never escapes. |

Three rules govern whether the surrounding element appears at all.

1. When the emitted content is neither empty nor false, the element and the content are emitted.
2. When the content is empty or false and the element has a body, the element and its body are emitted.
3. When the content is empty or false and the element has no body, **nothing** is emitted, unless the converter asked for the element to be forced. A converter asks for it when it needs the element as an anchor for in-place editing.

The reserved name `0` holds the rendered body handed to a template by its caller, as specified in section 1.9. Emitting that name is how a layout template places the content of the document that called it.

### 1.7 Attributes

| Directive | Value | Meaning |
|---|---|---|
| `t-att-` followed by a name | An expression. | Adds the attribute with the value of the expression. An empty or false value removes the attribute rather than emitting it empty. |
| `t-attf-` followed by a name | A format string. | The same, with the value produced by substitution. |
| `t-attf-` followed by a name and `.translate` | A format string. | The same, with the literal parts of the format string translated. |
| `t-att` | An expression producing a mapping. | Each key and value pair becomes one attribute. |
| `t-att` | An expression producing a pair. | The first element is the attribute name, the second its value. |

A **format string** mixes literal text with substitutions written either as a pair of opening braces, the expression and a pair of closing braces, or as a number sign, an opening brace, the expression and a closing brace. The two spellings are equivalent.

Static attributes, attributes produced by the three attribute directives and attributes produced by a namespace declaration are merged into one mapping and then passed through an **attribute post-processing** step before being emitted. The built-in post-processing blanks the target of a link whose address, after unquoting and after removing whitespace, uses a scheme that can execute code. That is what prevents a template from emitting an executable link out of stored data.

### 1.8 Options

| Directive | Value | Meaning |
|---|---|---|
| `t-options` | An expression producing a mapping. | Configuration for the directive on the same element. |
| `t-options-` followed by a key | An expression. | One entry of that mapping. Setting one key this way and setting the whole mapping with one entry are equivalent. |

Options configure `t-field`, `t-out` and `t-call`. Writing options on an element that carries none of those three is a template failure, detected by the internal check that every option must be consumed by some directive.

### 1.9 Calling another template

`t-call` takes a format string producing the name of the template to render in place of the element. Rendering a call proceeds as follows.

1. Copy the current value mapping.
2. Evaluate the element's own attributes and its `t-set` directives into that copy.
3. Render the element's **body** into that copy, and store the rendered body under the reserved name `0`.
4. Copy the compilation options and merge the element's `t-options` into them, together with the calling template's namespace declarations.
5. Obtain the compiled form of the named template.
6. Render it with the copied value mapping.

Three consequences follow.

- Names set **inside** the body of the call exist only for the called template; they do not leak back into the caller.
- Names set **before** the call in the calling template are visible to the called template, because the mapping is copied rather than replaced.
- The called template reaches the caller's body by emitting the reserved name `0`.

### 1.10 Rendering in another language

`t-lang`, written on the same element as `t-call`, evaluates to a language code and makes the called template render in that language: the translated terms of the called template, and the translated values of any field it renders, are taken from that language. It behaves exactly as the language option would, and it works **only** together with `t-call`. There is no way to switch the language of part of a template without extracting that part into its own template.

A template that renders translatable **record** values must additionally re-read the record in that language, because the language of the template and the language in which a record's stored translations are read are two different things: the first is a property of the render, the second a property of the execution context the record was read in. Re-reading is unnecessary, and costly, when the template renders no translatable record value. The storage and the fallback order of record translations are in [`translation.md`](translation.md), sections 4 and 3.

### 1.11 Group restriction

`t-groups`, also spelled `groups`, takes a comma-separated list of access group external identifiers, each optionally prefixed by an exclamation mark to exclude it. The element and its subtree are emitted only when the acting user satisfies the expression. Unlike the screen grammar, where group restrictions are resolved once when the layout is combined, this is evaluated at render time, on every render.

### 1.12 Asset bundles

`t-call-assets` takes a format string naming an asset bundle. At render time the bundle is resolved into an ordered list of style and script references, and one element is emitted per reference.

| Option | Effect |
|---|---|
| `css` | Emit only the style references. |
| `js` | Emit only the script references. |
| `defer_load` | Attach the scripts so that they run after the document has been parsed. |
| `lazy_load` | Attach the scripts so that they are fetched only when first needed. |
| `media` | Set the media condition of the style references. |

### 1.13 Diagnostics

`t-debug`, with an empty value, suspends the rendering at that point in the platform's debugging facility. It is meaningful only when the deployment runs in development mode.

`t-translation` set to `off` on an element suppresses term extraction and translation for that element and its subtree. It is the only directive of this grammar that is also accepted in every screen layout kind.

### 1.14 Escaping and safety

The output is markup-safe text. Characters with a special meaning in markup are replaced by their entity form unless the value is already marked safe.

| Value | Escaped on output |
|---|---|
| Ordinary text | Yes |
| The result of `t-call` | No, it is already markup |
| The result of a body-valued `t-set` | No |
| A rich-text field value | No |
| A value produced by the escaping helper | No; escaping it again would double-escape it |
| A value produced by the sanitizing helper | No |
| A value explicitly marked safe | No |

Marking a value safe is an assertion, not a check. A template that marks stored user input safe reintroduces exactly the injection the escaping was protecting against. To force the escaping of a value that is marked safe, convert it back to ordinary text first.

### 1.15 The rendering environment

Unless the caller asked for the minimal environment, the following names are always available.

| Name | Value |
|---|---|
| `true`, `false` | The boolean constants. Present even in the minimal environment. |
| `debug` | The diagnostic state of the current request. |
| `user_id` | The acting user record. |
| `res_company` | The acting company record, read with elevated rights. |
| `request` | The current request, when the render happens inside one. |
| `test_mode_enabled` | Whether the deployment runs in automated-test mode. |
| `json` | The structured-data helper, with its parsing and serializing operations. |
| `quote_plus` | The web-address encoding helper. |
| `time`, `datetime`, `relativedelta` | The date and time helpers. |
| `image_data_uri` | A helper that turns stored image bytes into an inline image reference. |
| `floor`, `ceil` | Rounding helpers, present so that a template does not need a round trip to round a number. |
| `env` | The execution environment, which gives access to any entity. |
| `lang` | The active language code. |
| `keep_query` | A helper that rebuilds the current web address keeping the named query parameters. |

A report template additionally always receives the following.

| Name | Value |
|---|---|
| `time` | The time helper. |
| `user` | The user record of the person printing. |
| `res_company` | That user's company. |
| `web_base_url` | The deployment's base address. |
| `context_timestamp` | A helper that takes a moment expressed in coordinated universal time and returns it in the time zone of the person printing. Whatever zone the given moment carries is ignored: the moment is first treated as coordinated universal time, then converted. |
| `is_html_empty` | A helper reporting whether a rich-text value is empty once its markup has been stripped. |

### 1.16 Compilation, caching and failures

A template is compiled once into a render procedure and the result is cached. The cache key is the template reference together with the context values that change the generated code: the language, the two branding switches, the translation-editing switch and the profiling switch. Creating, writing or deleting any view definition clears the cache, as does a change to the set of installed capability packages. The container itself is described in [`caching.md`](caching.md), section 10.

Failures carry their location: the template reference, the path of the element inside the template, and the element's own markup. That is what lets a failure message point at the exact element rather than at a line of generated code.

A template rendered in the restricted mode, in which no data store is reachable, refuses the three data-bound directives with `Fields are not allowed in this rendering mode.`, `Widgets are not allowed in this rendering mode.` and `Assets are not allowed in this rendering mode.`

### 1.17 Template inheritance

A template stored as a view definition is extended exactly like a screen layout: a child view definition whose parent is the template carries locators and positions, and the resolution algorithm produces the effective template. That algorithm is specified in [`../overview/views-and-actions.md`](../overview/views-and-actions.md). This is how a capability package adds a block to a shipped document without touching the shipped definition.

## 2. Rendering a field inside a template

`t-field`, and `t-out` carrying a `widget` option, both hand the value to a **converter** chosen from the widget name, or, for `t-field` without a widget, from the field's data type.

### 2.1 Converter selection

1. If the options carry a widget, the kind is the widget name.
2. Otherwise the kind is the field's data type.
3. Take the converter registered for that kind, or the generic converter when none is registered.

The generic converter escapes the value and emits it; an empty or false value produces nothing.

### 2.2 Metadata attributes

For `t-field`, and only when the branding switch or the translation-editing switch is on, the element additionally receives six attributes: the entity name, the record key, the field name, the converter kind, the original expression, and a read-only marker when the field is read-only. Those attributes are what allows an in-place editor to write the value back. They are absent from an ordinary print, which is why a printed document carries no editing metadata.

For `t-out` with a widget, only the converter kind and the expression are emitted.

### 2.3 The converter catalogue

| Kind | Renders | Options |
|---|---|---|
| generic | The escaped value; nothing when empty. | none |
| `integer` | The number with the thousands separator of the active language. A minus sign is followed by a zero-width joiner so that it never wraps away from the digits. | `format_decimalized_number` to abbreviate large numbers; `precision_digits`, the digits kept in the abbreviated form, default 1 |
| `float` | The number rounded and formatted with the separators of the active language. | `precision`, the number of decimal places; `decimal_precision`, the name of a stored precision setting, which wins over `precision`; `min_precision`, the least number of decimals to show |
| `date` | The date in the active language's date pattern. | `format`, an explicit pattern |
| `datetime` | The moment converted to the acting time zone and rendered with the active language's date and time patterns. | `format`; `tz_name` to render in that zone instead; `time_only`; `date_only`; `hide_seconds` |
| `text` | The escaped value with line breaks turned into line-break elements. | none |
| `selection` | The label of the current value in the active language. | `selection`, an explicit mapping from stored value to label |
| `many2one` | The display name of the linked record, read with elevated rights, with line breaks turned into line-break elements. | none |
| `many2many` | The display names of the linked records, read with elevated rights, joined by a comma and a space. | none |
| `one2many` | The same, for a list of records. | none |
| `html` | The rich-text value, re-parsed and re-emitted after its attributes have been passed through the attribute post-processing of section 1.7. | none |
| `image` | An image element whose source is the stored bytes inlined. The media type is detected from the bytes; a web-optimised image is converted when the target cannot display it; content that is not an image is refused with `Non-image binary fields can not be converted to markup`; unreadable content is refused with `Invalid image content`. | none |
| `image_url` | An image element whose source is the stored address. | none |
| `monetary` | The amount rounded to the currency's decimal places and formatted with the language's separators, with the currency symbol before or after it according to the currency's own convention, separated by a non-breaking space. The amount is wrapped in an inline element carrying the currency-value class. | `display_currency`, mandatory when the field is not a monetary field; `from_currency` to convert from that currency; `date`, the conversion date, default today; `company_id`, the company whose rates are used, default the acting company; `decimal_places` to override; `label_price` to render the decimal part smaller than the integer part |
| `float_time` | A number of hours rendered as hours and minutes, so that 1.5 becomes `01:30`. | none |
| `time` | A number of hours between zero and twenty-four rendered as a clock time in the active language's short pattern. A negative value is refused with `The value (<value>) passed should be positive`; twenty-four or more is refused with `The hour must be between 0 and 23`. | `format`, one of `short`, `medium`, `long`, `full` |
| `duration` | A number rendered as a span in words, so that 1.5 hours becomes `1 hour 30 minutes`. | `unit`, how to read the stored number, one of `second` (the default), `minute`, `hour`, `day`, `week`, `month`, `year`; `round`, the unit the result is rounded to, default `second`; `digital` to render `01:00` instead of `1 hour`; `format`, one of `long` (the default), `short`, `narrow`; `add_direction` to render `in 3 days` rather than `3 days` |
| `relative` | The signed distance between the value and a reference moment, in words. | `now`, the reference moment, default the current one |
| `barcode` | An image element holding a generated barcode, as specified in section 7. A value containing a character outside the plain seven-bit set is rendered as text instead. Attributes of the image element may be set through options whose name begins with `img_`; when no alternative text is given, `Barcode <value>` is used, with the value substituted. | `symbology`, default `Code128`; `width`, default 600; `height`, default 100; `humanreadable`, default 0; `quiet`, default 1; `mask` |
| `contact` | A formatted contact block built from the linked contact: the name, the address, the telephone number, the electronic-mail address and the tax identification number, each optional. | `fields`, the ordered list of parts to show, default the name, the address, the telephone number and the electronic-mail address; `separator`, the character joining the address lines, one of a space, a comma, a dash, a vertical bar or a slash, the default being a line break; `no_marker` to hide the small icons; `no_tag_br` to join the address lines with a comma instead of line breaks; `phone_icons` to keep the telephone icons even when markers are hidden; `country_image` to show the country flag when the field is present; `null_text` to render a placeholder block when the link is empty |
| `qweb` | The value, which must be a link to a template, is rendered as a template. A link to anything else emits nothing and logs a warning. The kind name is a reproduced stored value. | `values`, the value mapping handed to that template |

**The default precision of the decimal converter.** When no precision option is given:

```formula
integer digits     = the number of digits before the decimal point
available decimals = 14 − integer digits, floored at 0
precision used     = the smaller of 6 and the available decimals
decimals shown     = at least 1
```

Trailing zeros beyond the last significant digit are dropped.

**Worked example.** The value is 1234.5. The integer digits are 4, so the available decimals are 14 − 4 = 10 and the precision used is 6. The formatted value has at least one decimal and drops the trailing zeros, so in a language whose thousands separator is a comma and whose decimal separator is a full stop the result is `1,234.5`; in a language whose thousands separator is a space and whose decimal separator is a comma the result is `1 234,5`.

**Worked example of the monetary converter.** The amount is 1234.5, the currency has two decimal places, its symbol is the euro sign and its convention places the symbol after the amount, in a language whose thousands separator is a space and whose decimal separator is a comma.

```formula
rounded amount   = 1234.5 rounded to 2 decimals = 1234.50
formatted amount = 1 234,50
```

The result is the formatted amount wrapped in the currency-value element, then a non-breaking space, then the euro sign.

## 3. The Report Action entity

The transport name is `ir.actions.report` and the full name is Report Action. Records are ordered by name, then by key.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Name | Text, translatable | yes | none | The label of the print entry, and the file name when no name expression exists. |
| `model` | Model Name | Text | yes | none | The transport name of the entity the document is about. |
| `model_id` | Model | Many-to-one to Entity, computed with a search rule | no | derived | The same entity, as a link, so that the form can offer a selector. |
| `report_type` | Report Type | Selection: `qweb-html`, `qweb-pdf`, `qweb-text` | yes | `qweb-pdf` | The output form. The three stored values are reproduced exactly; their labels are `HTML`, `PDF` and `Text`, and their meanings are: render to a page shown in place, render to a paginated document that is downloaded, and render to unformatted text. |
| `report_name` | Template Name | Text | yes | none | The external identifier of the template to render. |
| `report_file` | Report File | Text, stored, writable | no | empty | The path of the file the template comes from. |
| `print_report_name` | Printed Report Name | Text, translatable | no | empty | An expression producing the file name, evaluated with the record available under the name `object` and the time helper under the name `time`. |
| `group_ids` | Groups | Many-to-many to Access Group, through the association table `res_groups_report_rel` | no | empty | Who may print it. |
| `multi` | On Multiple Doc. | Boolean | no | false | When true the entry is not offered on a single-record screen. |
| `paperformat_id` | Paper Format | Many-to-one to Paper Format, indexed when not empty | no | empty | The page geometry. When empty, the acting company's format is used. |
| `attachment` | Save as Attachment Prefix | Text | no | empty | An expression producing the name of the attachment the result is stored under. Empty means the result is not stored. |
| `attachment_use` | Reload from Attachment | Boolean | no | false | When true, an existing attachment with that name is returned instead of rendering again. |
| `domain` | Filter domain | Text holding a condition | no | empty | The entry is offered only for records satisfying the condition. |
| `binding_model_id`, `binding_type`, `binding_view_types` | Action Binding | See the action binding fields | no | `binding_type` defaults to `report` | Where the entry appears in the interface. |

The three stored values of the output form name the template engine and the output format. This specification never spells the engine out in prose; where a template tests the output form, it compares against the reproduced stored value, and the conventional test for "am I being converted to a paginated document" compares the output form with `qweb-pdf`.

**Resolving a report action from a reference.** The reference may be the record key, the record itself, an external identifier, or the value of the template name field. Resolution tries, in order:

1. A numeric key.
2. A record of the report action entity. A record of another entity is refused with `Expected report of type ir.actions.report, got <entity>`.
3. A search by the template name field.
4. An external identifier. A reference resolving to another entity is refused with `Fetching report <reference>: type <entity>, expected ir.actions.report`.

A reference that resolves to nothing is refused with `Fetching report <reference>: report not found`.

**Filtering by condition.** Given an entity and a set of record keys, the print entries offered are those with no condition, plus those whose condition is satisfied by at least one of the records.

**Producing the action.** Printing a set of records produces an action carrying: the kind marker of a report action, the external identifier of the template, the output form, the file path, the label, the extra data mapping when the caller supplied one, and the caller's context with the active record keys set to the records being printed.

There is one exception. When the acting user is an administrator, the acting company has **no** letterhead chosen, and the caller did not suppress the check, the action returned is instead the letterhead configuration screen, carrying the print action in its own context together with the flag that closes the screen once the document has been downloaded. That is what makes the very first print of a fresh deployment ask for a letterhead before printing anything.

## 4. The rendering pipeline

### 4.1 Entry points

Rendering resolves the report from its reference and then dispatches on the output form:

| Output form | Produces | Second value returned |
|---|---|---|
| `qweb-html` | The rendered markup. | The marker for markup. |
| `qweb-text` | The rendered text. | The marker for plain text. |
| `qweb-pdf` | The bytes of the paginated document. | The marker for the paginated form. |
| anything else | Nothing at all. | Nothing. |

Each entry point is also reachable as a request: one that renders in place, one that renders to a paginated document, one that renders to text, and one that wraps any of them in a download response.

### 4.2 The rendering context

1. Start from a copy of the extra data, or from an empty mapping.
2. Set the output form under the name `report_type`.
3. If an entity named `report.` followed by the template's external identifier exists, merge in the mapping that entity's context builder returns when called with the record keys and the data.
4. Otherwise set three names: the record keys under `doc_ids`, the entity name under `doc_model`, and the records themselves under `docs`.
5. Set the emptiness helper under `is_html_empty`.

A custom context builder **replaces** the three default names rather than extending them. A template that still wants them must put them back explicitly. This is the mechanism by which a document that aggregates several entities, such as a valuation report or an ageing balance, computes everything it needs in one place.

### 4.3 Rendering to markup and to text

Both render the named template with the context of section 4.2, plus the always-present report names of section 1.15. The markup result is returned as it stands; the text result is returned without any markup processing at all.

### 4.4 Rendering to a paginated document

1. Set the output form in the data to the paginated marker.
2. If the deployment is running automated tests and the caller did not force real rendering, return the markup rendering instead, so that a test suite never depends on the conversion engine.
3. Produce one stream per record, as specified in section 4.5.
4. If the report stores attachments, the record keys are unique, and the caller did not suppress storing, create the missing attachments as specified in section 4.7.
5. Merge the streams in key order into one document, as specified in section 4.8.
6. Return the merged bytes.

### 4.5 Producing one stream per record

1. Determine whether the key list contains the same key twice.
2. Prepare an ordered mapping from record key to a pair of stream and attachment.
3. For each record, in the order of the key list, skipping keys already seen: start with no stream and no attachment. If the report stores attachments, there are no duplicates, and storing is not suppressed, look for the attachment of that record whose name is the result of the name expression. If it exists and the report reuses stored attachments, the stream is that attachment's bytes; if the attachment holds an image rather than a document, it is first converted to a one-page document.
4. Collect the keys whose stream is still empty; call them the missing keys.
5. If there are no keys at all, or there are missing keys, continue with steps 6 to 12. Otherwise return.
6. If the conversion engine is not available, fail with `Unable to find the document conversion engine on this system. The document can not be created.`
7. Render the markup for the missing keys, with the diagnostic switch forced off.
8. Split the markup into bodies, a header and a footer, as specified in section 4.6.
9. If the report stores attachments, there are no duplicates, and the set of record keys found in the markup differs from the missing keys, fail with the template-structure message below.
10. Convert the bodies, the header and the footer into one document.
11. If there are duplicates, or there were no keys at all, return one stream holding the whole document.
12. If exactly one key was missing, that key's stream is the whole document. Otherwise split the document per record as specified in section 4.6.2.

The template-structure failure message is:

```
Report template “<report name>” has an issue, please contact your administrator.

Cannot separate file to save as attachment because the report's template does not carry the entity name and the record key as metadata attributes on the div with the 'article' classname.
```

The placeholder is the report's label.

The diagnostic switch is forced off during the conversion because the conversion engine loads the styles and scripts asynchronously, and the split, unminified form of the asset bundles is not always ready in time; without the switch being cleared, headers and footers disappear at random.

### 4.6 Splitting the rendered markup

The rendered markup is one page holding, for each record, one block carrying the class `header`, one carrying the class `article` and one carrying the class `footer`. It is taken apart as follows.

1. Parse the markup.
2. Build a container gathering every element with the class `header`, and another gathering every element with the class `footer`.
3. Prepare an empty list of bodies and an empty list of record keys.
4. For each element with the class `article`, in order: read its declared language attribute when it has one; render the minimal page template, in that language, with the element as its content and with substitution disabled; append the result to the bodies. If the element declares the entity being printed, append its declared record key to the record keys; otherwise append nothing in that position, keeping the two lists aligned.
5. If the list of bodies is empty, use the remaining content of the page as the single body. A template with no article element therefore still prints.
6. Collect every attribute of the root element whose name begins with `data-report-`; these are the geometry overrides.
7. Render the minimal page template with the header container as its content and with substitution **enabled**, and do the same with the footer container.

The **minimal page template** is a bare page that loads the document style bundles and, when substitution is enabled, a small script that runs once the page has been laid out. The conversion engine renders the header and the footer once per page and passes the current page number, the total page count and the index of the body being rendered as query parameters. The script reads them and:

1. writes the current page number into every element carrying the class `page`;
2. writes the total page count into every element carrying the class `topage`;
3. writes the section numbers into the elements carrying the classes `section`, `subsection` and `subsubsection`;
4. keeps, inside the header container, **only** the header of the body currently being rendered, and does the same for the footer container.

The fourth point is what allows one conversion run to produce documents for several records with different letterheads: every header is concatenated into one container, and the script selects the right one per page.

#### 4.6.2 Splitting the produced document per record

1. If the produced document has exactly as many pages as there are missing records, page number n belongs to record number n, in order.
2. Otherwise, if every missing record key was found in the markup: read the outline of the produced document. If it has no outline, return one stream for the whole document. Otherwise take the page numbers of the top-level outline entries, sorted, with duplicates removed. If the number of outline entries equals the number of records **and** the first entry is on the first page, record number n gets the pages from its outline page up to, but excluding, the next record's outline page, with the last record taking all the remaining pages. Otherwise render each record separately, one conversion run per record.
3. Otherwise return one stream for the whole document.

The outline is built from the top-level headings of the markup, which is why a document template that starts each record with a heading can be split even when records span different numbers of pages, while one that does not falls back to one conversion run per record.

### 4.7 The attachment rule

For each record key and its stream:

1. If an attachment already exists for it, skip it.
2. If the record key is unknown, or the stream is empty, log the following and skip it:

```
These documents were not saved as an attachment because the template of <template name> doesn't have any headers seperating different instances of it. If you want it saved, please print the documents separately
```

The placeholder is the template's name. The message is reproduced verbatim, including its spelling of "seperating".

3. Evaluate the name expression with the record available under the name `object` and the time helper under the name `time`. If the result is empty, skip it.
4. Create an attachment holding the stream's bytes, named that way, of the binary kind, attached to the record.

If creating the attachments is refused by the permission rules, log `Cannot save the document attachments <names> for user <user>` and continue; otherwise log `The documents <names> are now saved`.

Two fields interact:

| `attachment` | `attachment_use` | Behaviour |
|---|---|---|
| empty | any | Nothing is stored and nothing is reused: the document is rendered on every request. |
| set | false | The document is rendered on every request **and** stored the first time. |
| set | true | The stored document is returned when it exists; otherwise it is rendered and stored. This is the mode for documents that must never change once issued, such as a posted invoice. |

Storing is skipped entirely when the same record key appears twice in the request, because the produced document could not then be attributed to a record unambiguously.

### 4.8 Merging and failures

When several streams must be returned as one file they are concatenated in key order. A stream that cannot be read is recorded and the merge continues. If any stream failed, the whole request fails with a message and a follow-up action:

- the message `The platform is unable to merge the generated documents because of <n> corrupted file(s)`, in which the placeholder is the number of unreadable streams;
- an action that opens the problematic records, as a list and a form when there are several and as the form alone when there is exactly one;
- a button labelled `View Problematic Record(s)`.

A merge that fails for a structural reason rather than for a specific stream raises `The platform is unable to merge the generated documents.`

### 4.9 Geometry arguments

The page geometry handed to the conversion engine is built from the paper format of the report, or, when the report declares none, of the acting company, and is then overridden by the attributes beginning with `data-report-` found on the root element of the rendered markup.

| Source | Argument produced |
|---|---|
| `format`, when it is not `custom` | The named page size. |
| `page_width` and `page_height`, when the format is `custom` | Explicit width and height, in millimetres. |
| `margin_top`, or the override `data-report-margin-top` | Top margin. |
| `margin_bottom`, or the override `data-report-margin-bottom` | Bottom margin. |
| `margin_left`, `margin_right` | Side margins. |
| `dpi`, or the override `data-report-dpi` | Output resolution in dots per inch. On a platform that cannot render below ninety-six dots per inch, a smaller value is raised to ninety-six and the substitution is logged. When the engine supports it, a zoom factor is also passed, computed as 96 ÷ resolution, so that the physical size stays correct. |
| `header_spacing`, or the override `data-report-header-spacing` | The gap between the header and the body. |
| `orientation`, unless landscape was forced | Portrait or landscape. |
| `header_line` | Draw a rule under the header. |
| `disable_shrinking` | Disable the automatic shrink-to-fit. |
| The system parameter `report.print_delay`, default 1 000 | How long to wait after layout before capturing, in milliseconds, so that scripts have run. |
| The caller's landscape flag | Forces landscape. |
| The caller's viewport flag | Forces a viewport of 1 024 by 1 280 points, or 1 280 by 1 024 in landscape. |

Local file access is always disabled, which is why every image a document uses must be inlined or served over the deployment's own address.

### 4.10 Internal links and authentication

The conversion engine fetches the deployment's own addresses, for style bundles, images and barcodes, as a separate client that carries none of the caller's credentials. A **temporary session** is therefore created for it: copied from the acting session with the diagnostic switch cleared and tracing disabled, written to the session store, handed to the engine as a cookie, and deleted afterwards. Session storage and rotation are specified in [`sessions-and-authentication.md`](sessions-and-authentication.md).

The base address used is the system parameter `report.url` when it is set, and otherwise the deployment's own base address.

### 4.11 Large tables

A rendered body larger than four mebibytes is re-parsed, and every table holding more than five hundred rows is split into consecutive tables of five hundred rows each, preserving the table's attributes. The conversion engine's layout cost grows faster than linearly with the number of rows in one table; the split turns an hour-long conversion of a quarter of a million rows into roughly a minute.

### 4.12 Failure codes of the conversion

| Situation | Behaviour |
|---|---|
| Success | The document is returned. |
| The engine reports a soft failure while several bodies were being converted, and the engine is not the patched build | Fail with `Tried to convert multiple documents using an unpatched conversion engine` |
| The engine reports a soft failure otherwise | The engine's message is logged as a warning and the document is kept. |
| The engine was killed for lack of memory | Fail with `The document conversion engine failed (error code: -11). Memory limit too low or maximum file number of subprocess reached. Message : <last 1000 characters>` |
| Any other failure code | Fail with `The document conversion engine failed (error code: <code>). Message: <last 1000 characters>` |

The engine's readiness is reported as one of five states: not installed; too old, meaning below version `0.12.0`; ready; not enough workers, which happens with fewer than two request workers and makes the engine unable to fetch the deployment's own addresses; and broken, meaning present but not answering.

### 4.13 Downloading

The download request receives the address of the rendering request and the output form. It calls the rendering request and then sets the file name:

```formula
file name = report label + "." + the extension of the output form
```

If exactly one record is addressed and the report has a name expression, the report label is replaced by the expression's result, with the same extension.

A failure during a download is wrapped into a structured failure payload carrying the serialized reason, and the response is a server failure so that the client can show the reason rather than a blank download.

A registry of print handlers is consulted **before** the built-in behaviour. Each registered handler may claim the request, for instance in order to send the document to a connected printer instead of downloading it. When a handler claims the request and the action carries the flag that closes the screen after the download, a close-window action is produced.

### 4.14 Language per recipient

The pipeline never switches language on its own. A document is rendered in the language of the person printing unless the template asks otherwise, and it asks with the two-template pattern:

1. The outer template loops over the records and calls the body template once per record, carrying `t-lang` set to that record's recipient language.
2. The body template re-reads the record in that language, then calls the layout.

Each record is therefore rendered in **its own** recipient's language inside one print run. The language of the block is also written onto the article element as a language attribute, which the splitting step of section 4.6 uses to render each body in the right language.

To keep the letterhead in one fixed language while translating only the body, the layout call itself carries `t-lang` set to that fixed language.

## 5. Paper formats

### 5.1 The Paper Format entity

The transport name is `report.paperformat` and the full name is Paper Format.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Name | Text | yes | none | A mnemonic. |
| `default` | Default paper format? | Boolean | no | false | Marks the format as a sensible default. |
| `format` | Paper size | Selection over the named page sizes and `custom` | no | `A4` | A named page size, or the marker for explicit dimensions. |
| `margin_top` | Top Margin | Decimal, millimetres | no | 40 | Top margin. |
| `margin_bottom` | Bottom Margin | Decimal, millimetres | no | 20 | Bottom margin. |
| `margin_left` | Left Margin | Decimal, millimetres | no | 7 | Left margin. |
| `margin_right` | Right Margin | Decimal, millimetres | no | 7 | Right margin. |
| `page_height` | Page height | Integer, millimetres | no | none | Used only when the format is `custom`. |
| `page_width` | Page width | Integer, millimetres | no | none | Used only when the format is `custom`. |
| `orientation` | Orientation | Selection: `Landscape`, `Portrait` | no | `Landscape` | Page orientation. |
| `header_line` | Display a header line | Boolean | no | false | Draw a rule under the header. |
| `header_spacing` | Header spacing | Integer, millimetres | no | 35 | Gap between the header and the body. |
| `disable_shrinking` | Disable smart shrinking | Boolean | no | false | Disable the automatic shrink-to-fit. |
| `dpi` | Output resolution | Integer, dots per inch | yes | 90 | Output resolution. |
| `css_margins` | Use style-sheet margins | Boolean | no | false | When true, the margins are applied by the document styles rather than by the conversion engine, which is what the modern letterheads rely on. |
| `report_ids` | Associated reports | One-to-many to Report Action | no | empty | The reports explicitly bound to this format. |
| `print_page_width`, `print_page_height` | Print page width, Print page height | Decimal, computed, not stored | no | derived | The effective page size: the named size's dimensions, or the custom ones, with width and height swapped when the orientation is landscape. |

Declaring both a named format and explicit dimensions is refused with `You can select either a format or a specific page width/height, but not both.`

### 5.2 The named page sizes

Dimensions are given as width by height in millimetres, in portrait.

| Stored value | Width | Height |
|---|---|---|
| `A0` | 841 | 1 189 |
| `A1` | 594 | 841 |
| `A2` | 420 | 594 |
| `A3` | 297 | 420 |
| `A4` | 210 | 297 |
| `A5` | 148 | 210 |
| `A6` | 105 | 148 |
| `A7` | 74 | 105 |
| `A8` | 52 | 74 |
| `A9` | 37 | 52 |
| `B0` | 1 000 | 1 414 |
| `B1` | 707 | 1 000 |
| `B2` | 500 | 707 |
| `B3` | 353 | 500 |
| `B4` | 250 | 353 |
| `B5` | 176 | 250 |
| `B6` | 125 | 176 |
| `B7` | 88 | 125 |
| `B8` | 62 | 88 |
| `B9` | 33 | 62 |
| `B10` | 31 | 44 |
| `C5E` | 163 | 229 |
| `Comm10E` | 105 | 241 |
| `DLE` | 110 | 220 |
| `Executive` | 190.5 | 254 |
| `Folio` | 210 | 330 |
| `Ledger` | 431.8 | 279.4 |
| `Legal` | 215.9 | 355.6 |
| `Letter` | 215.9 | 279.4 |
| `Tabloid` | 279.4 | 431.8 |
| `custom` | the explicit width | the explicit height |

The two sizes whose stored dimensions are already landscape, `Ledger` and `Tabloid`, keep their stored values; the derived print size swaps width and height only when the orientation field says landscape.

### 5.3 The shipped formats

| Name | Format | Orientation | Top | Bottom | Left | Right | Header spacing | Resolution | Styles control the margins |
|---|---|---|---|---|---|---|---|---|---|
| `A4` (the default) | `A4` | Portrait | 52 | 32 | 0 | 0 | 52 | 90 | yes |
| `US Letter` | `Letter` | Portrait | 52 | 32 | 0 | 0 | 52 | 90 | yes |
| `US Batch Deposit` | `Letter` | Portrait | 15 | 30 | 10 | 10 | 15 | 90 | no |
| `A4 Label Sheet` | `A4` | Portrait | 0 | 0 | 0 | 0 | not applicable | 96 | no, and shrinking is disabled |
| `Dymo Label Sheet` | `custom`, 32 by 57 | Landscape | 0 | 0 | 0 | 0 | not applicable | 96 | no, and shrinking is disabled |

A company's own format is chosen on the company record and defaults to the `A4` format.

## 6. Document layouts, headers and footers

### 6.1 The layout templates

Three public layout templates exist. A document template calls one of them and places its own content inside.

| Layout | Purpose |
|---|---|
| External layout | The letterhead used for anything a customer or a supplier sees. It draws the header with the company branding, the recipient address block, the document title, the content, and the footer with the company's footer text and the page numbering. |
| Internal layout | A plain header for internal documents: the current moment on the left, the company name in the middle, and the current page number, a slash and the total page count on the right; then the content, wrapped in the article element that carries the record metadata. |
| Basic layout | No header and no footer at all: only the article element carrying the record metadata, wrapped in the page container. |

### 6.2 Choosing the company and the design

1. If the name `o` is not set, take the value of the name `doc` as the document record.
2. If the name `company` is not set: take the value of the name `company_id` when it is set; otherwise, when the document record has a company field, take that company read with elevated rights; otherwise take the acting company.
3. If the company has chosen a letterhead, call that letterhead with the content. Otherwise call the `Light` letterhead with the content.

The same resolution is used by the internal layout. Reading the company with elevated rights is deliberate: a user printing a document belonging to a company they may not read must still get the right letterhead.

### 6.3 The Report Layout entity

The letterheads offered to the user are records of the entity whose transport name is `report.layout` and whose full name is Report Layout, ordered by sequence, then by key.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `view_id` | Document Template | Many-to-one to View Definition | yes | none | The template that draws the letterhead. |
| `name` | Name | Text | no | none | The label shown in the chooser. |
| `sequence` | Sequence | Integer | no | 50 | The order in the chooser. |
| `image` | Preview image source | Text | no | none | The address of the preview thumbnail. |
| `pdf` | Preview document source | Text | no | none | The address of the preview document. |

The shipped letterheads:

| Sequence | Name | Design |
|---|---|---|
| 2 | `Light` | Small logotype on the left, company details on the right, thin footer rule. This is the fallback when the company has chosen none. |
| 3 | `Boxed` | Large logotype, boxed table headers and a filled total row using the primary colour. |
| 4 | `Bold` | Large logotype, thick rules above the table head and below the table body in the secondary colour. |
| 5 | `Striped` | Large logotype and tagline on the left, company details on the right, striped table rows. |
| 6 | `Bubble` | Rounded information block and filled table headers in the primary colour, tinted with white. |
| 7 | `Wave` | Wavy shapes drawn in the secondary colour, tinted information block. |
| 8 | `Folder` | Folder-shaped header drawn from the primary colour, footer rule in the secondary colour. |

### 6.4 Company branding

| Identifier | Full name | Type | Default | Effect on documents |
|---|---|---|---|---|
| `logo` | Company Logo | Image | empty | Drawn in the header; when empty no image is drawn. |
| `report_header` | Company Tagline | Rich text, translatable | empty | Drawn in the header or in the footer, depending on the letterhead. |
| `report_footer` | Report Footer | Rich text, translatable | derived | The footer text of every document. Its default is the telephone number, the electronic-mail address, the website and the tax identification number of the company, joined by spaces, in that order, skipping the empty ones. |
| `company_details` | Company Details | Rich text, translatable | derived | The address block drawn in the header. Its default is the company's formatted postal address, with the missing components removed from the format and the company name prepended when the format does not already contain it. |
| `is_company_details_empty` | Company details are empty | Boolean, computed | derived | True when the details, stripped of markup, are empty. When it is true, the header falls back to rendering the company's contact record with the contact converter, showing the name and the address and no markers. |
| `paperformat_id` | Paper Format | Many-to-one to Paper Format | the `A4` format | The page geometry used when a report declares none. |
| `external_report_layout_id` | Document Template | Many-to-one to View Definition | empty | The chosen letterhead. |
| `font` | Font | Selection | `Lato` | The document typeface. The allowed values are `Lato`, `Roboto`, `Open Sans`, `Montserrat`, `Oswald`, `Raleway`, `Tajawal` and `Fira Mono`. |
| `primary_color` | Primary Color | Text | empty, meaning very dark grey | Headings, total emphasis and the tagline. |
| `secondary_color` | Secondary Color | Text | empty, meaning very dark grey | Section labels, rules and tinted blocks. |
| `layout_background` | Background Image | Selection: `Blank`, `Demo logo`, `Custom` | `Blank` | Whether the body carries a watermark: none, a shipped demonstration mark, or the uploaded image. |
| `layout_background_image` | Background Image File | Binary | empty | The watermark used when the choice is `Custom`. |

Changing the typeface, either colour or the chosen letterhead invalidates the generated document style sheet, which is stored as an attachment and rebuilt on demand. The attachment mechanism is in [`attachments-and-file-store.md`](attachments-and-file-store.md).

The colours are applied through a generated style sheet scoped per company, so that a document of one company keeps its own branding even when several companies are printed in one run.

| Element | Colour |
|---|---|
| The document title | Primary |
| The labels of the information block | Secondary |
| The total emphasis | Primary |
| The company tagline | Primary |
| Boxed letterhead: the total row background | Primary, with the text switched to black or white depending on the lightness of that colour |
| Bold letterhead: the rule above the table head and below the table body | Secondary |
| Folder letterhead: the folder shape | Primary mixed with 92 percent white; the footer rule in the secondary colour |
| Wave letterhead: the information block | Secondary for the border, secondary mixed with 92 percent white for the background |
| Bubble letterhead: the information block | The same as the wave letterhead; the table head and the total row background in the primary colour, with the text switched for contrast |

The contrast rule is: when the lightness of the background colour exceeds fifty percent the text is black, otherwise it is white.

### 6.5 The address block

Every external letterhead calls one shared address block.

1. When no address value is set, draw nothing but an empty structural container, so that the layout keeps its column geometry.
2. When an information block is also set, draw the information block and the address side by side; which of the two comes first depends on whether the letterhead places the address on the custom side.
3. Otherwise draw the address alone, right-aligned, occupying five of the twelve columns.

The address value is normally produced by the contact converter of section 2.3. The information block is a free slot used for a second address, for instance the delivery address on a shipping document.

### 6.6 Page numbering

Page numbering is not computed by the template. The template emits two empty elements.

| Class | Replaced at layout time by |
|---|---|
| `page` | The number of the page being rendered, starting at one. |
| `topage` | The total number of pages of the produced document. |

The substitution script described in section 4.6 fills them. The conventional rendering in the footer of every external letterhead is the word `Page`, the current page number, a slash and the total page count, drawn only when the document is being converted to a paginated document, because a page shown in place has no pages. When the caller sets the name `display_name_in_footer`, the record's name is drawn before the page numbers.

The internal layout draws the same two elements in its header instead of in a footer.

### 6.7 The structural contract of a document template

A document template whose result must be splittable per record, and storable as one attachment per record, has to satisfy three rules.

1. Each record's content is wrapped in an element carrying the class `article`.
2. That element carries the entity name and the record key as metadata attributes and, when the record is rendered in a specific language, that language as a third metadata attribute. Calling one of the three public layouts satisfies this automatically.
3. Each record's content starts with a top-level heading, so that the outline-based split of section 4.6.2 can find the record boundaries when records span several pages.

The body of the document sits inside an element carrying the class `page`; the header and the footer sit in elements carrying the classes `header` and `footer`.

## 7. Barcodes and quick response codes

### 7.1 The generator

The generator takes a symbology, a value and a set of options.

1. Apply the defaults: width 600, height 100, human-readable false, quiet true, no mask, border 4, error-correction level `L`. Each option is coerced to its type; an unrecognised error-correction level falls back to `L`.
2. When human-readable is true, select the typeface used for the readable characters.
3. If width × height is greater than 1 200 000, or if the greater of width and height is greater than 10 000, fail with `Barcode too large`.
4. Normalize the symbology:
   - `UPCA` with a value of 11, 12 or 13 characters becomes `EAN13`; a value of 11 or 12 characters is left-padded with one zero.
   - `auto` becomes `EAN8` for a value of 8 characters, `EAN13` for a value of 13 characters, and `Code128` for any other length.
   - `QR`, the two-dimensional quick response symbology, ignores the quiet-zone option; setting quiet to false instead sets the border to zero.
   - `EAN8` or `EAN13` whose value does not satisfy that symbology's check-digit rule becomes `Code128`, because the generator would otherwise silently encode a corrected value that differs from the one asked for.
5. Draw the barcode as an image.
6. When a mask is named and registered, apply it to the drawn image.
7. Return the image bytes.

On a drawing failure: `Code128` fails with `Cannot convert into barcode.`; `QR` fails with `Cannot convert into matrix code.`; every other symbology is retried with `Code128`.

The mask registry is an extension point: a mask takes the width, the height and the drawn image and returns a modified image. It is what allows a payment quick response code to carry a logotype in its centre.

### 7.2 Symbologies

The accepted symbologies are `Codabar`, `Code11`, `Code128`, `EAN13`, `EAN8`, `Extended39`, `Extended93`, `FIM`, `I2of5`, `MSI`, `POSTNET`, `Standard39`, `Standard93`, `UPCA`, `USPS_4State`, `QR` and the pseudo-value `auto`, which selects one of the others by the length of the value. All of these are reproduced stored values.

### 7.3 Options

| Option | Default | Meaning |
|---|---|---|
| `width` | 600 | Image width in pixels. |
| `height` | 100 | Image height in pixels. |
| `humanreadable` | 0 | Draw the value in readable characters under the bars. |
| `quiet` | 1 | Draw the mandatory blank margins. |
| `mask` | none | The name of a registered post-processing mask. |
| `barBorder` | 4 | The border of the quick response symbology. |
| `barLevel` | `L` | The error-correction level of the quick response symbology: `L` tolerates up to 7 percent damage, `M` up to 15 percent, `Q` up to 25 percent, `H` up to 30 percent. Any other value falls back to `L`. |

### 7.4 Using a barcode in a template

Two forms are available: the field converter of section 2.3, or an image element whose source is the barcode request. The barcode request accepts either the symbology and the value as path segments, or both together with the width and the height as query parameters.

The barcode request answers with an image and a long-lived, immutable cache directive, so that a document that repeats one code on every page fetches it once. A value that cannot be encoded answers with a failure whose description is `Cannot convert into barcode.`

## 8. Label documents

A label document is an ordinary report whose paper format is a label sheet and whose template lays out a grid of identical cells.

### 8.1 The label layout chooser

Printing labels goes through a transient record that asks for the sheet geometry. Its transport name is `product.label.layout`.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `print_format` | Format | Selection: `dymo`, `2x7xprice`, `4x7xprice`, `4x12`, `4x12xprice` | yes | `2x7xprice` | The sheet geometry. The first value is one label per page on a roll; the others are a grid, with a price when the value ends in `xprice`. |
| `custom_quantity` | Copies | Integer | yes | 1 | How many copies of each label to print. |
| `product_ids` | Product Variants | Many-to-many to Product Variant | no | empty | What to print labels for. |
| `product_tmpl_ids` | Products | Many-to-many to Product | no | empty | The same, at the product level. |
| `extra_html` | Extra Content | Rich text | no | empty | Extra content placed in each cell. |
| `pricelist_id` | Pricelist | Many-to-one to Price List | no | empty | The price list whose price is printed, when the chosen format includes a price. |
| `rows`, `columns` | Rows, Columns | Integer, computed, not stored | no | derived | Parsed from the chosen format: the part before the first letter `x` is the number of columns and the part after it the number of rows; a format that is not of that shape yields one column and one row. |

Preparing the print:

1. If the copy count is not strictly positive, fail with `You need to set a positive quantity.`
2. Choose the template: the roll template when the format is the roll format, otherwise the template named for the number of columns and rows, with the no-price suffix appended when the format name does not contain `xprice`.
3. If neither product variants nor products are given, fail with `No product to print, if the product is archived please unarchive it before printing its label.`
4. Build the data: the entity being printed, the mapping from each record key to the copy count, the chooser's own key, and whether the format includes a price.

The produced print action carries the flag that closes the screen once the document has been downloaded, and it skips the letterhead check, because a label sheet has no letterhead.

### 8.2 The shipped label formats

| Stored value | Columns | Rows | Price | Paper format |
|---|---|---|---|---|
| `dymo` | 1 | 1 | yes | `Dymo Label Sheet`: 32 by 57 millimetres, landscape, no margins, 96 dots per inch, shrinking disabled |
| `2x7xprice` | 2 | 7 | yes | `A4 Label Sheet` |
| `4x7xprice` | 4 | 7 | yes | `A4 Label Sheet` |
| `4x12` | 4 | 12 | no | `A4 Label Sheet` |
| `4x12xprice` | 4 | 12 | yes | `A4 Label Sheet` |

Each shipped label report sets its file name expression to the text `Products Labels - ` followed by the record's name.

### 8.3 What a label cell contains

A label cell holds, in this order: the product's display name; optionally its price taken from the chosen price list; its barcode drawn with the default symbology; and the extra content from the chooser. The grid is drawn as a table whose cell size is the page size divided by the number of columns and rows, with no page margins, which is why the label paper formats set every margin to zero and disable the automatic shrink-to-fit.

## 9. Exporting data

Two export formats are offered. The list of offered formats is itself a request that answers, for each format, its tag, its label and, when the format cannot be produced in this deployment, the reason.

| Tag | Label | Media type | Extension |
|---|---|---|---|
| `xlsx` | Spreadsheet workbook | The spreadsheet workbook media type | `.xlsx` |
| `csv` | Comma-separated values | The comma-separated values media type, with the eight-bit unicode character set | `.csv` |

Both tags are reproduced stored values; they are the file extensions themselves. When the workbook writer is not available, the workbook entry carries the reason naming the missing writer and its least supported version, and the format cannot be chosen.

### 9.1 Choosing the fields

The export dialog builds a tree of exportable fields. Listing the fields of an entity, given a prefix and the kind of the parent field:

1. Read the descriptions of the entity's fields.
2. In the import-compatible mode: when the parent is a relation, offer only the record key and the display-name field of the related entity; do not offer a read-only field; do not offer a field the caller excluded.
3. In the plain mode: offer one extra pseudo-field for the record's own numeric key.
4. Never offer a field marked as not exportable.
5. Label the record-key pseudo-field `External Identifier`.
6. When the entity is not an ordinary table, do not offer the record key at all.
7. Expand user-defined fields into one entry per defined property, labelled with the property label, a space, an opening parenthesis, the record that defines it and a closing parenthesis; skip separators and properties whose target entity is unknown.
8. Sort the result by label, ignoring case.
9. For each field, the entry's path is the prefix joined to the field name by a slash. A relation field at a depth below three is expandable: its value is the path followed by the external-identifier segment, and expanding it lists the related entity's fields with that path as the new prefix.

The two modes differ in intent. The **import-compatible** mode offers only what can be read back by the importer, which means external identifiers for relations and no read-only field. The plain mode offers the numeric key and every readable field.

An export template stores a named list of field paths for reuse. Resolving one turns the stored paths back into labelled entries, fetching the descriptions of each level in one request per level rather than one per field.

### 9.2 The export request

1. Take the named records, or every record matching the condition when no keys were named.
2. When the entity is not an ordinary table, drop the record-key field from the requested fields.
3. Build the column headers: the field paths in the import-compatible mode, the trimmed labels otherwise.
4. If the mode is not import-compatible and a grouping was given: export the records with their numeric key first; build the group tree from the grouped read, keeping the natural order of the records inside each group; render the grouped form of section 9.4.
5. Otherwise export the records in batches, invalidating each batch's cache after it has been converted so that a large export does not grow without bound in memory, and render the flat form.
6. Log `User <key> exported <n> <entity> records from <address>. Fields: <paths>. Keys sample: <first ten keys>`, or the same line ending with `. Condition: <condition>` when the export was driven by a condition rather than by explicit keys.
7. Respond with the rendered file, the media type of the format, and a file name built from the entity description, a space, an opening parenthesis, the entity name, a closing parenthesis and the extension, cleaned of characters that are not valid in a file name.

The value of each cell is produced by the generic export conversion of a record: one row per record, except that a record with several related lines produces one row per line, with the parent's cells left empty on the continuation rows.

### 9.3 The comma-separated values form

1. Write the header row.
2. For each row, for each cell: an empty or false value becomes the empty text; a binary value is decoded to text; a text value beginning with an equals sign, a hyphen or a plus sign is prefixed with an apostrophe so that a spreadsheet application does not read it as a formula.
3. Write the row.

Every field is quoted. A grouped export is not supported in this form and is refused with `Exporting grouped data to comma-separated values is not supported.`

### 9.4 The spreadsheet workbook form

One sheet is produced. The first row holds the column headers in bold, and every column is thirty characters wide.

| Cell kind | Format |
|---|---|
| Text | Wrapped; carriage returns replaced by spaces; a value longer than the workbook's per-cell limit is replaced by `The content of this cell is too long for a spreadsheet workbook (more than <limit> characters). Please use the comma-separated values format for this export.` |
| Binary | Decoded to text; content that is not text is refused with `Binary fields can not be exported to Excel unless their content is base64-encoded. That does not seem to be the case for <column header>.` |
| A list or a mapping | Rendered as text. |
| Date | Wrapped, with the number format `yyyy-mm-dd`. |
| Date and time | Wrapped, with the number format `yyyy-mm-dd hh:mm:ss`. |
| Decimal | Wrapped, with the number format `#,##0.00`. |
| Monetary | Wrapped, with a number format having as many decimals as the currency with the greatest number of decimal places in the deployment, defaulting to two. |

A request whose row count exceeds the workbook format's limit is refused with `There are too many rows (<count> rows, limit: <limit>) to export as Excel 2007-2013 (.xlsx) format. Consider splitting the export.` That message is reproduced verbatim, including the third-party format name it contains.

**The grouped form.** Groups are written recursively, deepest last. For each group, at a given depth:

1. Write the group header row. Its first cell holds four spaces per depth level, the group label, a space, an opening parenthesis, the number of records and a closing parenthesis, in bold on a grey background. Every other column holds the aggregate of that column for this group, in bold on grey, with the decimal or monetary number format when the column is numeric and rendered as text otherwise.
2. Write each child group, one depth level deeper.
3. Write the rows of the records of this group.

A group whose value is empty is labelled `Undefined`, except when the grouping field is boolean, in which case the empty value keeps its own label.

**Aggregates.** A column is aggregated only when its field declares an aggregator and its path contains no slash, that is only for a field belonging directly to the exported entity. The supported aggregators are the greatest value, the least value, the sum, "true for all" and "true for any". The average is computed as the sum divided by the number of records of the group. A cell whose value is the empty text, which is what a continuation row holds, is excluded from the aggregation. An unsupported aggregator is skipped and logged as `Unsupported export of aggregator '<name>' for field <field> on model <entity>`.

**Worked example.** Three invoices are exported grouped by customer, with the columns for the number and the total. The customer `ACME` has two invoices of 100.00 and 250.00; the customer `Globex` has one invoice of 70.00. The sheet holds six rows:

| Row | First column | Second column |
|---|---|---|
| 1 | `Number` | `Total` |
| 2 | `ACME (2)` | 350.00 |
| 3 | `INV/2026/0001` | 100.00 |
| 4 | `INV/2026/0002` | 250.00 |
| 5 | `Globex (1)` | 70.00 |
| 6 | `INV/2026/0003` | 70.00 |

Rows 2 and 5 are bold on a grey background, and their totals carry the decimal number format. The sum of the second group is 350.00 + 70.00 = 420.00, which is not written anywhere: no grand total row is produced.

## 10. The request paths

| Path pattern | Method | Authentication | Answer |
|---|---|---|---|
| `/report/<converter>/<template name>` and the same followed by a slash and the comma-separated record keys | read | authenticated user | The rendered result. The converter segment is `html`, `pdf` or `text`; any other value is refused with `Converter <name> not implemented.` Extra query parameters become the extra data mapping; a parameter named `options` is parsed as structured data and merged; a parameter named `context` is parsed as structured data and merged into the acting context. |
| `/report/barcode` and `/report/barcode/<symbology>/<value>` | read | public | The barcode image, with a long-lived immutable cache directive. |
| `/report/download` | write | authenticated user | The rendered document with a file-name header, as specified in section 4.13. |
| `/report/check_wkhtmltopdf` | read | authenticated user | The readiness state of the conversion engine, as specified in section 4.12. The path is a reproduced route path. |
| `/web/export/formats` | read | authenticated user | The offered export formats. |
| `/web/export/get_fields` | read | authenticated user | The exportable field tree of section 9.1. |
| `/web/export/namelist` | read | authenticated user | The labelled entries of a stored export template. |
| `/web/export/xlsx` and `/web/export/csv` | write | authenticated user | The exported file. One path exists per format tag. |

The two export paths are declared as write operations even though they only read, because a large export must not run on a read-only cursor. The read-only decision is specified in [`request-lifecycle.md`](request-lifecycle.md), section 7.

## 11. Worked examples

### 11.1 A three-record print that splits cleanly

A report of the paginated form prints three invoices. Its template calls the external layout once per record and each record's content starts with a top-level heading.

| Step | Effect |
|---|---|
| 1 | The action resolves; the acting company has a letterhead, so the letterhead check does not divert the print. |
| 2 | No attachment name expression is set, so no attachment is looked for and all three keys are missing. |
| 3 | The markup is rendered once, holding three header blocks, three article blocks and three footer blocks. |
| 4 | The split produces three bodies, three record keys, one header container and one footer container. |
| 5 | One conversion run produces a document. Each record fits on one page, so the document has three pages and the count matches the number of records: page 1 is the first record's stream, page 2 the second's, page 3 the third's. |
| 6 | The three streams are merged in key order into one document of three pages, which is what the caller receives. |

### 11.2 The same print where one record spans two pages

| Step | Effect |
|---|---|
| 1 to 4 | As above. |
| 5 | The document has four pages, which does not equal the three records, so the outline is read. It holds three top-level entries, on pages 1, 3 and 4, and the first is on the first page. |
| 6 | The first record takes pages 1 to 2, the second page 3, the third page 4. |
| 7 | The merge produces the same four-page document, and each record's stream is available separately for storing as an attachment. |

Had the template omitted the top-level headings, the outline would be empty and one undivided four-page document would be returned, with nothing stored as an attachment.

### 11.3 A stored invoice document printed twice

The invoice report sets a name expression and reuses stored attachments.

| Step | Effect |
|---|---|
| 1 | The first print finds no attachment, renders the markup, converts it, and stores the bytes as an attachment named by the expression. |
| 2 | The second print evaluates the expression, finds that attachment, and uses its bytes as the stream. No markup is rendered and the conversion engine is not started. |
| 3 | The invoice is later modified and printed again. The expression still produces the same name, so the **stored** document is returned: the changes do not appear. That is the intended behaviour of a document that must not change once issued. |

### 11.4 One print run in two recipient languages

Two sales orders are printed together. The first recipient's language is French as used in France, the second's is English as used in the United States.

| Step | Effect |
|---|---|
| 1 | The outer template loops over the two records and calls the body template once per record with `t-lang` set to that record's recipient language. |
| 2 | Each body template re-reads its record in that language, so the translatable product descriptions come out in the recipient's language. |
| 3 | Each article element carries its language as a metadata attribute. |
| 4 | The split renders each body in the language its article declares, so the layout's own labels also come out right. |
| 5 | The header of each page is selected by the substitution script from the concatenated header container, so each order carries its own letterhead in its own language. |

### 11.5 A barcode whose check digit is wrong

A template asks for the symbology `EAN8` and the value `11111111`.

| Step | Effect |
|---|---|
| 1 | The size check passes: 600 × 100 = 60 000, which is below 1 200 000, and neither dimension exceeds 10 000. |
| 2 | The symbology is `EAN8`, so the check-digit rule is evaluated. The correct check digit for the first seven digits is 5, not 1, so the value fails the rule. |
| 3 | The symbology becomes `Code128`, which encodes arbitrary text. |
| 4 | The drawn code encodes exactly `11111111`. Had the symbology stayed `EAN8`, the generator would have drawn `11111115`, a different value, which is precisely the silent corruption the substitution prevents. |

## 12. Acceptance criteria

**Template grammar**

1. **Given** an element carrying a false condition and holding `A`, followed by a sibling carrying `t-else` and holding `B`, **when** the template is rendered, **then** only the second element and its content are emitted.
2. **Given** an element carrying a loop over the three integers 1, 2 and 3 with the loop variable named, a condition excluding the value 2, and an output of the loop variable, **when** it is rendered, **then** two elements are emitted holding `1` and `3`, which proves that the loop runs before the test.
3. **Given** a loop over a mapping of two entries with the loop variable named, **when** it is rendered, **then** the loop variable takes the keys and the derived name ending in `_value` takes the values.
4. **Given** a variable set to 1 before a loop over 1 and 2 that adds the loop variable to it and also sets a second variable, **when** the loop ends, **then** the first variable is 4 and the second does not exist.
5. **Given** a variable whose value is a rendered body containing an element, **when** it is emitted, **then** the element is emitted as markup, not escaped.
6. **Given** a variable whose value is a text string containing markup characters, **when** it is emitted, **then** the characters are escaped.
7. **Given** an output directive whose value is false on an element that has a body, **when** it is rendered, **then** the element and its body are emitted.
8. **Given** the same directive on an element with no body, **when** it is rendered, **then** nothing at all is emitted.
9. **Given** an attribute directive whose format string selects between two class names by the parity of an index equal to 3, **when** it is rendered, **then** the odd class name is emitted.
10. **Given** an attribute directive whose expression is false, **when** it is rendered, **then** the attribute is absent from the output.
11. **Given** a caller that sets a name inside the body of a call and a called template that emits that name and then the reserved name `0`, **when** it is rendered, **then** the name is visible to the called template, the caller's body is emitted after it, and the name does not exist in the caller after the call.
12. **Given** `t-options` written on an element carrying none of `t-field`, `t-out` and `t-call`, **when** the template is compiled, **then** compilation fails.
13. **Given** an element carrying an attribute beginning with `t-` that is not a known directive, **when** the template is compiled, **then** compilation fails.
14. **Given** a link element whose address, after unquoting and after whitespace removal, uses an executable scheme, **when** it is rendered, **then** the emitted address is empty.
15. **Given** a template rendered in the restricted mode that uses `t-field`, **when** it is rendered, **then** it fails with `Fields are not allowed in this rendering mode.`

**Field rendering**

16. **Given** a decimal field holding 1234.5 with no precision option, in a language using a space as thousands separator and a comma as decimal separator, **when** it is rendered, **then** the output is `1 234,5`.
17. **Given** a monetary field holding 1234.5 in a currency with two decimals whose symbol is placed after the amount, in the same language, **when** it is rendered, **then** the output is the amount `1 234,50` wrapped in the currency-value element, followed by a non-breaking space and the symbol.
18. **Given** a decimal field holding 1.5 rendered with the duration converter, the unit `hour` and the digital option, **when** it is rendered, **then** the output is `01:30`.
19. **Given** the same field with the unit `hour` and no digital option, **when** it is rendered, **then** the output is `1 hour 30 minutes`.
20. **Given** a link to a contact rendered with the contact converter and the default parts, **when** it is rendered, **then** the output holds the name, the address, the telephone number and the electronic-mail address, in that order, with the address lines separated by line breaks.
21. **Given** a binary field whose content is not an image rendered with the image converter, **when** it is rendered, **then** it fails with `Non-image binary fields can not be converted to markup`.
22. **Given** a closed-list field, **when** it is rendered, **then** the output is the label of the current value in the active language, not the stored value.
23. **Given** a value of 24 rendered with the clock-time converter, **when** it is rendered, **then** it fails with `The hour must be between 0 and 23`.

**Rendering pipeline**

24. **Given** a report whose output form is `qweb-html`, **when** it is rendered, **then** no conversion engine is involved and the result is the rendered markup.
25. **Given** a report whose output form is `qweb-pdf`, three records and a template producing exactly one page per record, **when** it is rendered, **then** the result holds three pages and each record's stream is its own page.
26. **Given** the same report and a template producing two pages for the first record and one for each of the others, each record starting with a top-level heading, **when** it is rendered, **then** the split follows the outline and the first record gets two pages.
27. **Given** the same report whose template does not mark the records with their entity and key, and whose action stores attachments, **when** it is rendered, **then** it fails with the template-structure message of section 4.5.
28. **Given** a report with a name expression and the reuse flag set, **when** the same record is printed twice, **then** the second print returns the stored attachment and performs no conversion.
29. **Given** a report with a name expression and the reuse flag cleared, **when** the same record is printed twice, **then** two conversions happen and exactly one attachment exists.
30. **Given** a request naming the same record key twice, **when** it is rendered, **then** nothing is stored as an attachment and one undivided document is returned.
31. **Given** a template calling the body template with the recipient's language and two records whose recipients speak different languages, **when** both are printed in one run, **then** each record's body is in its own recipient's language and each body carries that language as a metadata attribute.
32. **Given** a rendered body larger than four mebibytes holding a table of 1 200 rows, **when** it is prepared for conversion, **then** the conversion receives three tables of 500, 500 and 200 rows.
33. **Given** a paper format whose resolution is 90 and a platform that cannot render below 96 dots per inch, **when** the geometry is built, **then** the conversion receives 96 and the substitution is logged.
34. **Given** a root element carrying a top-margin override, **when** the geometry is built, **then** that override wins over the paper format's top margin.
35. **Given** a conversion that is killed for lack of memory, **when** the failure is reported, **then** the message names the failure code −11 and the last 1 000 characters of the engine's output.
36. **Given** two streams of which one cannot be read, **when** they are merged, **then** the request fails with the corrupted-file message naming 1 and offers the button `View Problematic Record(s)`.
37. **Given** the deployment running automated tests without forced real rendering, **when** a paginated report is rendered, **then** the markup rendering is returned instead.

**Layouts**

38. **Given** a company with no chosen letterhead, **when** a document calls the external layout, **then** the `Light` letterhead is used.
39. **Given** an administrator, a company with no chosen letterhead and a print request, **when** the action is produced, **then** the letterhead configuration screen is returned instead, carrying the print action in its context.
40. **Given** a record whose company differs from the acting company, **when** a document calls the external layout without naming a company, **then** the record's company is used, read with elevated rights.
41. **Given** a company whose details are empty, **when** the header is drawn, **then** the company's contact record is rendered through the contact converter instead of the details block.
42. **Given** a paginated document, **when** it is produced, **then** every footer shows the current page number and the total page count; **given** the same document rendered in place, **then** no page numbers are drawn.
43. **Given** a company whose primary colour is light, **when** a filled block is drawn, **then** the text on it is black; **given** a dark colour, **then** it is white.

**Barcodes and quick response codes**

44. **Given** the symbology `UPCA` and a twelve-character value, **when** the code is generated, **then** the symbology used is `EAN13` and the value is left-padded with one zero.
45. **Given** the symbology `EAN8` and the value `11111111`, whose check digit is wrong, **when** the code is generated, **then** the symbology used is `Code128` and the encoded value is exactly `11111111`.
46. **Given** a requested image of 4 000 by 4 000 pixels, **when** the code is generated, **then** it fails with `Barcode too large`.
47. **Given** the symbology `auto` and a thirteen-character value, **when** the code is generated, **then** `EAN13` is used; with a value of any other length than 8 or 13, `Code128` is used.
48. **Given** the symbology `QR` with the quiet option set to false, **when** the code is generated, **then** the border is set to zero and the quiet option is otherwise ignored.
49. **Given** the symbology `QR` and an error-correction level that is not one of the four, **when** the code is generated, **then** the level `L` is used.

**Labels**

50. **Given** the format `4x12` and a copy count of 3, **when** the labels are printed, **then** twelve rows of four cells are drawn per page and each product appears three times.
51. **Given** a copy count of zero, **when** the print is prepared, **then** it fails with `You need to set a positive quantity.`
52. **Given** no product selected, **when** the print is prepared, **then** it fails with `No product to print, if the product is archived please unarchive it before printing its label.`
53. **Given** the format `dymo`, **when** the dimensions are computed, **then** they are one column and one row, because the value contains no letter `x` in the position that would make it a grid.

**Exports**

54. **Given** a comma-separated values export of a text value beginning with an equals sign, **when** the cell is written, **then** it begins with an apostrophe.
55. **Given** a grouped comma-separated values export, **when** it is requested, **then** it is refused with `Exporting grouped data to comma-separated values is not supported.`
56. **Given** a workbook export grouped by customer as in the worked example of section 9.4, **when** it is produced, **then** the sheet holds the group rows with the counts and the summed totals in exactly the positions listed there.
57. **Given** a workbook export of a date value, **when** the cell is written, **then** it holds a real date with the number format `yyyy-mm-dd`, not text.
58. **Given** a workbook export whose row count exceeds the format's limit, **when** it is requested, **then** it is refused with the too-many-rows message naming the count and the limit.
59. **Given** an import-compatible field listing, **when** it is built, **then** read-only fields are absent and a relation offers only the external identifier and the display-name field of the related entity.
60. **Given** a column whose field declares no aggregator, **when** a grouped workbook export is produced, **then** the group header row leaves that column empty rather than summing it.

## 13. Reconciliation notes

1. The three stored values of the output form were replaced by full-word spellings in the source document, on the ground that the original spellings contain abbreviations. They are stored selection values and are therefore contractual: they are reproduced here as `qweb-html`, `qweb-pdf` and `qweb-text`, with their meaning given in words. The same applies to the two export tags, reproduced as `xlsx` and `csv`, and to the converter kind `qweb`.
2. The field identifiers of the Report Action and the Paper Format were paraphrased. They are reproduced exactly: `model`, `model_id`, `report_type`, `report_name`, `report_file`, `print_report_name`, `group_ids`, `multi`, `paperformat_id`, `attachment`, `attachment_use`, `domain`; and `format`, `margin_top`, `margin_bottom`, `margin_left`, `margin_right`, `page_height`, `page_width`, `orientation`, `header_line`, `header_spacing`, `disable_shrinking`, `dpi`, `css_margins`, `report_ids`, `print_page_width`, `print_page_height`.
3. The company letterhead field was given under two names. Its reproduced name is `external_report_layout_id`.
4. The base-address parameter of the conversion engine was paraphrased. Its reproduced name is `report.url`.
5. The source document illustrated the grammar with fragments of markup. Markup fragments are not admissible in this repository, so every illustration has been restated as prose or as a worked example that names the directives and describes the output in words. No rule was dropped in the restatement; the evaluation order, the scope rules, the three emptiness rules of the output directives and the six steps of a call are all stated in full.
6. The two-dimensional symbology was renamed in the source document. Its stored value is `QR`, and the full name in words is the quick response symbology; both are used here, the stored value in code font and the full name in prose.
7. The barcode option `barBorder` was described as the border of "the matrix symbology". It applies to the quick response symbology, which is the only two-dimensional one the generator accepts.
8. Data export was specified in the same document as printing. It stays with printing here, because it is the second way the platform turns records into a file, it shares the request layer and the file-naming rules, and no other document of this folder owns it.
9. The label chooser's field identifiers were paraphrased. They are reproduced as `print_format`, `custom_quantity`, `product_ids`, `product_tmpl_ids`, `extra_html`, `pricelist_id`, `rows` and `columns`.
10. The endpoint catalogue the source document linked to does not exist in this repository. The request paths are listed in section 10, and the read-only decision that governs them is in [`request-lifecycle.md`](request-lifecycle.md).
