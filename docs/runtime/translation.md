# Translation

Every piece of text a user sees comes from one of three places: a value stored on a record, a label or help text declared by a capability package on a field or on a closed-list value, or a literal produced by behaviour at the moment it runs. All three are translatable, and each uses a different storage and a different resolution rule.

This document specifies the language model, the per-language storage of translatable field values, the term-level translation of markup values, the resolution and fallback rules applied when a value is read, the marking and resolution of literals produced by behaviour, the translation of values shipped inside data files, the export and import of translation terms, the interaction between a translation import and the no-update rule of a capability package, the external translation service, and how translated labels reach views, printed documents, menus and messages. A replacement that follows these rules shows the same text to the same user in the same language, keeps the same values in storage, and produces comparable translation catalogues.

The language chosen for a request is resolved as specified in [`request-lifecycle.md`](request-lifecycle.md), section 11. The per-language shape of the record cache is in [`caching.md`](caching.md), section 3. Printed documents apply these rules through the template grammar of [`report-rendering.md`](report-rendering.md).

## 1. Vocabulary and the three kinds of translatable text

| Kind | Where the text lives | Who may change it | Resolution moment |
|---|---|---|---|
| Whole-value field translation | Inside the record, as one value per language | Any user allowed to write the field, and the translation import | When the field is read |
| Term-level field translation | Inside the record, as one complete document per language, matched term by term against the source document | Any user allowed to write the field, and the translation import | When the field is read |
| Behaviour literal translation | Inside the capability package's translation catalogues, never in the database | Only by shipping a new catalogue | When the literal is produced |

| Term | Meaning |
|---|---|
| Source language | The single language in which every source term is written and against which every translation is matched. Its code is `en_US`, English as used in the United States. |
| Language code | A locale identifier of the form of a language subtag alone, or a language subtag and a region subtag joined by an underscore. |
| Base language chain | The ordered list of language codes a given code falls back through, specified in section 4.3. |
| Translatable field | A field declared as translatable. It is whole-value when the declaration is a plain flag, and term-level when the declaration names a term-splitting rule. |
| Term | The smallest unit of text that is translated independently. For a whole-value field the entire value is one term; for a term-level field a term is one translatable fragment extracted from the markup, as specified in section 5. |
| Translation catalogue | A file holding source terms and their translations for one language and one or more capability packages. |
| Translation catalogue template | The same file with every translation empty, used as the starting point for a new language. |
| Delayed translation | A translation kept aside under a shadow key while the source term is being edited, not yet visible to ordinary readers, as specified in section 6.5. |

## 2. The Language entity

The transport name is `res.lang` and the full name is Language. Records are ordered by the active flag descending, then by name, so that the active languages come first.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Name | Text | yes | none | The display name of the language. Unique; the violation message is `The name of the language must be unique!` |
| `code` | Locale Code | Text | yes | none | The locale code. Unique; the violation message is `The code of the language must be unique!` It is used to set and read the locale for formatting. |
| `iso_code` | Standard language code | Text | no | none | The shortened form of the code: the code with the region subtag removed when the language subtag and the lowercased region subtag are identical, and the full code otherwise. It names the catalogue files and identifies the language in log messages. |
| `url_code` | Address Code | Text | yes | none | The code used inside public web addresses. Unique; the violation message is `The web address code of the language must be unique!` |
| `active` | Active | Boolean | no | false | Whether the language is available for use. Archived languages are hidden from every default query. |
| `direction` | Direction | Selection: `ltr` labelled `Left-to-Right`, `rtl` labelled `Right-to-Left` | yes | `ltr` | Writing direction. |
| `date_format` | Date Format | Selection | yes | `%m/%d/%Y` | The date layout. The value set is the nine combinations of the three orders — day, month, year; month, day, year; year, month, day — with the three separators, the slash, the hyphen and the full stop, each labelled with the corresponding rendering of the thirty-first of January of the current year. |
| `time_format` | Time Format | Selection | yes | `%H:%M:%S` | The time layout. Two values: the twenty-four-hour form labelled `13:00:00` and the twelve-hour form labelled ` 1:00:00 PM`. |
| `week_start` | First Day of Week | Selection: `1` labelled `Monday` through `7` labelled `Sunday` | yes | `7` | The first day of the week. |
| `grouping` | Separator Format | Selection: `[3,0]` labelled `International Grouping`, `[3,2,0]` labelled `Indian Grouping` | yes | `[3,0]` | The digit grouping pattern. Its help text reads `The International Grouping will represent 123456789 to be 123,456,789.00; The Indian Grouping will represent 123456789 to be 12,34,56,789.00` |
| `decimal_point` | Decimal Separator | Text, never trimmed | yes | `.` | The decimal separator. |
| `thousands_sep` | Thousands Separator | Text, never trimmed | no | `,` | The digit group separator. |
| `flag_image` | Image | Image | no | none | An optional flag image. |
| `flag_image_url` | Flag image address | Text, computed, not stored | no | derived | The address of the flag image: the stored image when one is present, otherwise the conventional flag image of the region subtag of the code, lowercased. |

Neither separator is trimmed, because a space is a legitimate group separator in several locales and trimming it would silently change the formatting.

The entity forbids the privileged list-editing commands from elevated sessions, so that a language cannot be created or deleted as a side effect of writing a list field on another record.

### 2.1 Validation rules

| Condition that fails | Message |
|---|---|
| No language would remain active. Checked only once the registry is ready, so that the check does not fire during installation. | `At least one language must be active.` |
| The date layout or the time layout uses a directive outside the permitted set. The two-digit year directive is permitted although discouraged; every other non-portable directive is rejected. | `Invalid date/time format directive specified. Please refer to the list of allowed directives, displayed when you edit a language.` |
| The code is modified after creation. | `Language code cannot be modified.` |
| A language is deactivated while a user prefers it. | `Cannot deactivate a language that is currently used by users.` |
| A language is deactivated while a contact prefers it. | `Cannot deactivate a language that is currently used by contacts.` |
| A language is deactivated while any user record, including archived ones, prefers it. | `You cannot archive the language in which the system was set up as it is used by automated processes.` |
| The source language is deleted. | `Base Language 'en_US' can not be deleted.` |
| The language of the current session is deleted. | `You cannot delete the language which is the user's preferred language.` |
| An active language is deleted. | `You cannot delete the language which is Active!` followed by a line break and `Please de-activate the language first.` |

### 2.2 Behaviour on the form

Changing the date layout or the time layout to a combination that mixes the twenty-four-hour directive with the morning and afternoon marker rewrites the twenty-four-hour directive into the twelve-hour one and raises a notification whose title is `Using 24-hour clock format with AM/PM can cause issues.` and whose message is `Changing to 12-hour clock format instead.`

### 2.3 Creation, activation and deactivation

**Activating an existing language** sets the active flag on the record whose code matches, searching archived records as well, and returns it.

**Activating and loading** does the same through the unarchive path, which additionally imports the translation catalogues of every installed capability package for that language, as specified in section 10.2.

**Creating a language from a locale** builds the record from the operating system's locale data:

1. Try every locale spelling derived from the code until one is accepted by the formatting library. When none is accepted, log `Unable to get information for locale <code>. Information from the default locale (<current>) have been used.`, in which the placeholders are the requested code and the locale actually in force.
2. Set the code to the requested code, and the standard language code to the shortened form of it.
3. Set the name to the supplied display name, or to the code when no name was given.
4. Set the active flag.
5. Set the date layout to the locale's date layout, with non-portable directives rewritten into their portable equivalents and the no-padding marker removed; set the time layout the same way.
6. Set the decimal separator to the locale's decimal separator and the group separator to the locale's group separator, correcting a badly encoded non-breaking space in either.
7. Set the grouping pattern to the locale's grouping pattern when it is one of the two permitted values, and to the international pattern otherwise.
8. Create the record, then restore the process locale.

**On creation**, a record with no address code receives the shortened form of its code, falling back to the full code.

**On activation**, when the activated language's address code still carries a region subtag, the platform tries to shorten it: if the bare language subtag is the address code of another, inactive language whose own code is not that bare subtag, then that other language's address code is set to its full code and the activated language takes the bare subtag. This gives the short, readable address form to the language actually in use.

**Deactivating** a language deletes the default-value records that set it as the default language of contacts, so that a contact created afterwards does not receive an inactive language.

Creating, writing or deleting a Language record clears the stable cache containers of the registry, because the set of active languages is cached. The containers are specified in [`caching.md`](caching.md), section 10.

### 2.4 The cached active-language data

Reading language properties on every request would be prohibitive, therefore a fixed set of fields of the **active** languages is cached per database and indexed by any one of them. The cached fields are the record key, the name, the code, the standard language code, the address code, the active flag, the direction, the date layout, the time layout, the first day of the week, the grouping pattern, the decimal separator, the group separator and the flag image address.

- The primary index is by code and is built by reading the active languages ordered by name.
- Any other index is derived from the primary one rather than by a second read.
- Asking for an index on a field outside the cached set raises `Field "<name>" is not cached`, with the field name substituted.
- Looking up a missing key returns a placeholder whose every cached field is empty, which reads as "no such active language" without raising.

Two convenience readings are built on the cache: resolving a code to a Language record, which is empty when the language is not active, and listing the installed languages as pairs of code and name sorted by name.

### 2.5 Number formatting

Formatting a number for a language applies the language's decimal separator and, when grouping is requested, its grouping pattern and group separator.

1. The pattern must be a single format specifier; otherwise raise `format() must be given exactly one %char format specifier`.
2. Render the value with the pattern.
3. Read the cached data of this language. When the language is not active, raise `The language <name> is not installed.`, with the name substituted.
4. If grouping was requested and the pattern renders a real number: split the rendering at the decimal point, group the integer part according to the grouping pattern by inserting the group separator, and join the parts with the language's decimal separator.
5. If grouping was requested and the pattern renders an integer: group the whole rendering according to the grouping pattern.
6. If grouping was not requested and the pattern renders a real number whose rendering holds a decimal point: replace the decimal point by the language's decimal separator.

The grouping pattern is a list of group sizes read from the least significant digit outwards. A size of zero repeats the previous size indefinitely; a size of minus one stops the grouping.

| Value | Pattern `[3,0]` | Pattern `[3,2,0]` |
|---|---|---|
| 123456789 | `123,456,789` | `12,34,56,789` |
| 1234.5 | `1,234.5` | `1,234.5` |
| 12345678.9 | `12,345,678.9` | `1,23,45,678.9` |

**Worked example of the Indian pattern.** The value is 12345678.9 and the pattern is `[3,2,0]`. The integer part is 12345678. Reading from the least significant digit, the first group takes 3 digits (678), the second takes 2 (45), and the third size is 0, which repeats the previous size of 2 indefinitely, giving 23 and then 1. The groups, read back in order, are 1, 23, 45 and 678, joined by the group separator: `1,23,45,678`. The decimal part is appended with the decimal separator: `1,23,45,678.9`.

## 3. Storage of translatable field values

### 3.1 Column shape

A translatable field is stored as a map from storage key to text, in a single column. The keys are language codes and, for term-level fields, the shadow keys of section 6.5. The source-language key is always present on a non-empty value.

Three declaration conflicts are reported.

| Conflict | Warning |
|---|---|
| A field is declared both per company and translatable. | `company_dependent field <field> cannot be translated` |
| A stored translatable field's derivation depends on the session, which would make the stored value differ per session. | `Translated stored fields (<field>) cannot depend on the session` |
| A path-following field is both stored and translatable, so that only the session language's value is recomputed. | `Translated stored path-following field (<field>) will not be computed correctly in all languages` |

### 3.2 Reading

Reading a translated field returns the text held under the first key of the fallback chain of section 4.4 that is present.

- In the record cache, the value of a translated field is kept per language. The cache is addressed by the effective language for whole-value fields and by the technical language for term-level fields.
- When the session asks for all languages at once, which the export and the editing screens do, the cache holds the complete map for each record: every active language plus the source language, with missing languages filled from the source value and, for term-level fields, missing shadow keys filled from the source shadow key when one exists.
- Reading in the source language never falls back, because the source key is the last element of every chain and the first when the language is the source language.

### 3.3 Writing a whole-value field

1. If the value is empty or absent, clear the whole map: the column becomes empty. Stop.
2. Take the effective language, defaulting to the source language.
3. Keep only the records whose cached value already differs from the new value.
4. Flush any pending empty values of those records, so that a pending clear is not reordered after the write.
5. If the field is not stored: update the cache for the effective language only; when the field is derived and has an inverse rule, invalidate the other languages instead, which forces them to be recomputed. Stop.
6. Invalidate the records that hold no pending change, because their cached value may be a fallback rather than a real value for this language.
7. Write the value under the key of the effective language.
8. If the effective language is not the source language **and** the source language is not an active language, also write the value under the source key.

Step 8 matters in an installation that has archived the source language. Without it, the source key would keep a stale value that every other language falls back to, and a record written only in the active languages would still read as the stale source text for a language with no value of its own.

### 3.4 Writing a term-level field

Writing a term-level field must preserve the translations of the terms that did not change, re-match the terms that moved, and drop the translations of the terms that disappeared.

1. If the value is empty or absent, clear the whole map and stop.
2. Neutralize every term of the new value, which makes the stored source the document itself rather than a translated rendering of it.
3. Take the technical language of section 4.2.
4. Extract the set of terms of the new value.
5. For each record: if the new value has no terms, the new map is the source key and the language key both holding the new value. Otherwise read the stored map directly from the column; if it is empty, the new map is again the source key and the language key both holding the new value.
6. Otherwise build the old mapping: for every non-shadow key of the stored map, take the shadow value when one exists and the plain value otherwise.
7. Take the value to compare against: the old mapping's entry for the technical language when it has one, and the source entry otherwise. Remove the technical language from the old mapping.
8. Build the translation dictionary from that value and the old mapping, as specified in section 6.2.
9. Match the new terms against the dictionary keys, as specified in section 6.3.
10. Recompose one new translation per remaining key of the old mapping, by walking the new value's terms and replacing each with the dictionary entry for that key, leaving a term unchanged when the dictionary holds no entry for it.
11. If the session asks for delayed translations, the new map is the stored map with every new translation written under its shadow key and the shadow key of the current language removed. Otherwise the new map is the set of new translations.
12. Set the technical language's key to the new value.
13. If the source language is not active, set the source key to the new value as well and remove the source shadow key.
14. Store the new map.

### 3.5 Searching

A filter on a translated field compares against the value resolved through the fallback chain, so a filter in French matches the French value when one exists and the source value otherwise.

When the field carries a similarity index and the comparison is one of the containment or pattern operators, a pre-filter is applied first: the whole stored map is rendered as an array of texts and compared with the pattern, which lets the similarity index narrow the candidate rows; the exact comparison on the resolved value is then applied to the narrowed set. The pre-filter is valid only for the containment and pattern operators. It is never applied to the negated comparisons, because a row whose French value does not match may still have a source value that does, and never to the ordered comparisons, because the array rendering has no order.

### 3.6 Duplication

Duplicating a record copies the whole map of every copied translatable field, so every translation is carried over. List-valued fields are duplicated in ascending key order, which keeps the correspondence between the original children and the copies and lets their translations be copied positionally.

## 4. The language of a read and the fallback chain

### 4.1 The session language

Every operation runs in a session that may carry a language code. The effective language is resolved as follows.

1. If the session carries no language code, the effective language is empty, which means the source language.
2. If the session carries a code other than the source-language code and no **active** language has that code, the operation fails with `Invalid language code: <code>`, with the code substituted.
3. Otherwise the effective language is the carried code.

### 4.2 The technical language

Term-level fields use a technical language code that differs from the effective language in two editing situations.

```formula
technical language = "_" + (effective language, or the source language when empty)
                     when the session is in translation-editing mode or in
                     translation-checking mode
technical language = effective language, or the source language when empty
                     otherwise
```

The underscore prefix selects the shadow keys that hold delayed translations, as specified in section 6.5. Whole-value fields never use the technical language: they always resolve with the effective language.

### 4.3 The base language chain

Every language code resolves through a chain of increasingly specific codes.

1. Let the bare code be the part of the code before the underscore, or the whole code when it has none. The chain starts with the bare code.
2. If the bare code is `es`, Spanish, and the code is neither `es_ES`, Spanish as used in Spain, nor `es_419`, Latin American Spanish, append `es_419`.
3. If the code is `zh_HK`, Chinese as used in Hong Kong, append `zh_TW`, Chinese as used in Taiwan.
4. If the code differs from the bare code, append the code.

| Code | Chain |
|---|---|
| `fr_BE`, French as used in Belgium | `fr`, `fr_BE` |
| `fr_FR`, French as used in France | `fr`, `fr_FR` |
| `en_US` | `en`, `en_US` |
| `es_MX`, Spanish as used in Mexico | `es`, `es_419`, `es_MX` |
| `es_ES` | `es`, `es_ES` |
| `es_419` | `es`, `es_419` |
| `zh_HK` | `zh`, `zh_TW`, `zh_HK` |
| `zh_TW` | `zh`, `zh_TW` |
| `de`, German | `de` |

The chain is ordered from the most general to the most specific. It is used when loading catalogues, where the later file overrides the earlier one, as specified in section 9.1, and when filtering imported rows, where only rows whose language belongs to the chain of the target language are kept, as specified in section 9.6.

### 4.4 The value fallback chain

Reading a translated value falls back through a chain of storage keys.

| Reading language | Chain of keys, in order |
|---|---|
| `_en_US` | `_en_US`, then `en_US` |
| `en_US` | `en_US` alone |
| Any other code beginning with an underscore | that code, then the same code without the prefix, then `_en_US`, then `en_US` |
| Any other code | that code, then `en_US` |

The first key that holds a value wins. A database read of a translated field therefore selects the first non-empty key of that chain, in one statement, without reading the whole map.

## 5. Extracting terms from markup values

A term-level field declares a term-splitting rule. Two rules exist: the markup rule and the rich-text rule. Both walk the document and identify the fragments that must be translated as a whole.

### 5.1 Which nodes are translatable

A node is translatable as a whole when all three of the following hold.

1. Its element name belongs to the set of inline elements, or the node is explicitly marked for inline translation by carrying the inline-translation class.
2. It carries no templating attribute: no attribute whose name begins with the templating prefix, no access-group attribute, and no attribute whose name ends with the explicit translation marker.
3. Every one of its children is itself translatable under the same rule.

The set of inline elements is fixed and consists of exactly: `abbr`, `b`, `bdi`, `bdo`, `br`, `cite`, `code`, `data`, `del`, `dfn`, `em`, `font`, `i`, `ins`, `kbd`, `keygen`, `mark`, `math`, `meter`, `output`, `progress`, `q`, `ruby`, `s`, `samp`, `small`, `span`, `strong`, `sub`, `sup`, `time`, `u`, `var`, `wbr`, `text`, `select` and `option`.

### 5.2 Which attributes are translatable

An attribute is translatable when its name ends with the explicit translation marker, or when the node carries no template-call attribute and the attribute name belongs to the fixed set: `string`, `add-label`, `help`, `sum`, `avg`, `confirm`, `placeholder`, `alt`, `title`, `aria-label`, `aria-keyshortcuts`, `aria-placeholder`, `aria-roledescription`, `aria-valuetext`, `value_label`, `data-tooltip`, `label`, `confirm-label`, `confirm-title` and `cancel-label`, together with every one of those names prefixed with the formatted-attribute prefix.

Two attributes are translatable only in context.

- The `value` attribute, when the node is a text input that is not a date picker, or a hidden input carrying the translatable-hidden-input class.
- The `text` attribute, when the node is a field node rendered as a web address.

For the extraction of client templates a narrower set applies, matching the set the client itself translates: `alt`, `aria-label`, `aria-placeholder`, `aria-roledescription`, `aria-valuetext`, `data-tooltip`, `label`, `placeholder` and `title`.

On a component node, which is an element whose name begins with an uppercase letter or which carries the component or slot attribute, only attributes ending with the explicit translation marker are translated.

### 5.3 The splitting algorithm

Processing one node:

1. Stop when the node is a comment or a processing instruction, or its element name is `script`, `style` or `title`, or it carries the translation-off marker, or it is an attribute node whose target attribute is not translatable, or it is the root node of a document that begins with a document type declaration.
2. Set the position to zero.
3. Repeat: if there is text to translate at this position, move every consecutive translatable child from this position into a temporary container, together with the text that precedes it; serialize the container's content, strip its surrounding whitespace, and translate it as one term. If a translation is produced, substitute it into the serialized content, re-parse the result, and use it when the result is still translatable and still carries text. Move the container's children back into the node at this position.
4. Stop when the position has reached the end of the node; otherwise process the child at this position recursively and advance.
5. For every attribute of the node that holds non-space text and is translatable: when the attribute name begins with the templating prefix, protect the embedded expressions, translate the remaining text, then restore the expressions; otherwise translate the attribute value. Keep the original value when no translation is produced.

"There is text to translate at this position" holds when there is non-space text before the child at that position, or when the child at that position is translatable and either carries a translatable attribute, or itself contains text to translate, or is followed by text to translate.

Embedded expressions inside a formatted attribute are protected by replacing each occurrence of the two expression syntaxes, the number-sign-and-brace form and the double-brace form, with a numbered placeholder before translating, and restoring them afterwards. A placeholder absent from the translation is restored as the empty value, so that a translator who drops a placeholder produces a shorter text rather than a broken one.

### 5.4 Term identity and the textual content of a term

Two terms are the same term when their serialized forms are identical, including their markup. For the purpose of re-matching a changed source, specified in section 6.3, the **textual content** of a term is used instead: the concatenation of its text nodes with runs of whitespace collapsed to single spaces.

A term is considered plain text when it contains no element at all.

### 5.5 Round-trip rules

- A markup value that does not parse as a well-formed document is re-parsed with the tolerant rich-text parser, wrapped in a container, translated, and the container is stripped from the result.
- A rich-text value is always wrapped in a container before parsing and the container is stripped afterwards; a non-breaking space is re-encoded as its named entity on output.
- A rich-text value that cannot be parsed at all logs `Cannot translate malformed rich text, using source value instead` and is returned unchanged.

## 6. Matching translations to a changing source

### 6.1 Why matching is needed

Term-level translations are positional: each language's stored document holds the same terms in the same order as the source document. When the source document changes, the stored documents of the other languages must be rebuilt in a way that keeps the translations of the terms which did not change.

### 6.2 The translation dictionary

Building the dictionary from a source value and a mapping from language to value:

1. Extract the terms of the source value. If there are none, the dictionary is empty; stop.
2. Give every source term an empty mapping in the dictionary.
3. For each language and value of the input: extract the terms of the value. If their number differs from the number of source terms, set every source term's entry for that language to the source term itself. Otherwise pair the terms positionally and set each source term's entry for that language to the matching term.

The count-mismatch rule is deliberate: a stored document whose structure diverged from the source is treated as untranslated rather than mis-aligned, so a structural drift degrades to the source text instead of scrambling the translations.

### 6.3 Re-matching terms when the source changes

When a term-level value is written, terms present in the new value keep their dictionary entries. Terms that disappeared are re-matched against the new terms.

1. Build an index from textual content to the list of new terms with that content.
2. For every old term that is not among the new terms: take the index entries whose key is a close match of the old term's textual content, with a similarity of at least 0.9, keeping only the best match.
3. If there is no candidate, the old term's translations are dropped.
4. Otherwise take the closest term among the new terms sharing that textual content, compared on the full serialized term.
5. If the dictionary already holds an entry for the closest term, leave it alone.
6. Otherwise, if the old term is plain text or the closest term is not plain text: when the closest term is not plain text, the write happens during a package installation, the technical language is the source language, and the splitting rule provides a structure adapter, build the adapter from the closest term; if the adapter rejects the old term because the structures differ, skip it; otherwise move the old term's translations to the closest term, each translation passed through the adapter. When those conditions do not all hold, move the old term's translations to the closest term unchanged.

The similarity used is the ratio of matching characters between the two textual contents, as produced by a longest-matching-block comparison. A threshold of 0.9 means that the two texts are ninety percent similar.

**Worked example.** A source term reads `Knife` and its French translation is `Couteau`. The source becomes `Knife ` with a trailing space. The textual contents are `Knife` and `Knife `, of lengths 5 and 6, sharing 5 matching characters.

```formula
similarity = 2 × matching characters ÷ ( length of first + length of second )
           = 2 × 5 ÷ ( 5 + 6 )
           = 10 ÷ 11
           = 0.9091
```

0.9091 is at least 0.9, so the translation `Couteau` is carried over to the new term. Had the source become `Cutlery knife`, the similarity would be below the threshold and the translation would be dropped.

### 6.4 The structure adapter

The structure adapter exists to carry translations across a source change that only altered presentation attributes. It is built from the new source term and applied to an old translated term.

1. Parse both terms inside a container.
2. Walk the two trees in parallel. If at any point the element names differ or the number of children differs, the structures do not match: return nothing.
3. For every matched pair of nodes: remove from the old node every presentation-condition attribute that the new source node does not carry, the presentation-condition attributes being `invisible`, `readonly`, `required`, `column_invisible` and the combined attribute form; then copy every attribute of the new source node onto the old node.
4. Return the old node serialized, with the container stripped.

The effect is that a translated view term keeps its translated text while adopting the new visibility and editability conditions of the source, which is what lets a capability package tighten a condition without discarding every translation of the screen.

### 6.5 Delayed translations

When the session asks for delayed translations, a write of a term-level value keeps the previous rendering visible and stores the freshly recomposed translations under **shadow keys**, which are the language code prefixed with an underscore. The shadow key of the language being written is removed.

Reading with the translation-checking or translation-editing session marker uses the technical language, which is the underscore-prefixed code, and therefore sees the shadow values. Ordinary reads keep seeing the previous values until the shadow values are confirmed. The fallback chain of section 4.4 makes a missing shadow key fall back first to the plain key, then to the source shadow key, then to the source key.

A translation import always clears the shadow key of every language it rewrites, so that an import confirms what it writes.

### 6.6 Rendering terms for the translation editor

When the session carries the translation-editing marker, reading a term-level field wraps every term in a marker element carrying five values: the entity name, the record key, the field name, the translation state and a digest of the source term.

The translation state is `translated` when the reading language is the record's base language or when the term differs from its source, and `to_translate` otherwise. The base language of a record is the source language, unless the entity overrides that rule; a record scoped to a public site uses the default language of that site.

The source terms used for the comparison are the terms of the record's value in its base language. When the number of terms of the read value differs from the number of terms of the base-language value, every translation is ignored and the base-language value is returned instead. A class marking a delayed translation is added when the value read differs from the value the ordinary read would return.

## 7. Behaviour literals

### 7.1 Marking

A literal produced by behaviour is translatable only when it is explicitly marked. Two markings exist.

- **Immediate**: the literal is resolved to a translation at the moment the marked expression is evaluated.
- **Deferred**: the literal is captured with its owning capability package and is resolved only when it is rendered as text. Deferred marking exists for literals declared before any session exists, for example the entries of a table of error messages declared at load time.

Only a literal may be marked. Marking an expression, a variable or an already formatted string produces a term that can never be matched against a catalogue entry.

### 7.2 Resolution

1. If the language is the source language, the translation is the source.
2. Otherwise look the source up in the owning package's catalogue for that language; when it is absent, the translation is the source.
3. If there are no arguments, return the translation.
4. If any argument is markup, escape the translation, so that the substitution is safe.
5. Resolve every deferred argument to text in the same language.
6. Replace every argument that is a collection other than text by its localized enumeration, using the language's list conjunction rules.
7. Substitute the arguments into the translation.
8. If the substitution fails, because of a wrong count, a wrong name or a wrong type, log `Bad translation <translation> for string <source>` and substitute into the source instead. A substitution failure on the source itself is not caught and propagates.

### 7.3 Determining the package and the language

The package that owns a literal is the package whose code produced it. For a deferred literal the package is captured when the literal is declared; for an immediate literal it is taken from the declaring location.

The language is determined in this order.

1. An explicitly supplied language.
2. A language carried by a local session variable at the call site.
3. A language carried by a session passed as an argument at the call site.
4. The language of the session of the record set the behaviour is running on.
5. The language of the session of the current request.
6. The preferred language of the acting user, read from the database.
7. The declared fallback language of the literal, if it declares one. Literals owned by the root package fall back to the source language.
8. None, in which case the source is used and the warning `no translation language detected, skipping translation <location>` states that no language could be detected.

### 7.4 Argument rules

| Rule | Correct form | Incorrect form |
|---|---|---|
| Substitute at resolution time, never before | Mark the pattern and pass the value as an argument. | Mark an already substituted string. |
| Keep one sentence in one term | Mark the whole sentence with a placeholder. | Concatenate several marked fragments. |
| Do not build plurals by concatenation | Mark one term per plural form and select between them. | Mark the singular and append a marked suffix. |
| Resolve at run time, not at declaration time | Use the deferred marking for literals declared outside a behaviour. | Use the immediate marking in a table declared at load time. |

Worked examples of argument handling, all evaluated in Belgian French, where the catalogue translates the pattern by appending the region name:

| Call | Result |
|---|---|
| Pattern `Code, %s, English`, one argument holding 1 | `Code, 1, Français, Belgium` |
| The same pattern, the argument being the byte sequence `Test byte string` | `Code, b'Test byte string', Français, Belgium`; a byte sequence is not treated as a collection |
| The same pattern, the argument being the list of `1`, `2` and `3` | `Code, 1, 2 et 3, Français, Belgium`; the list is enumerated with the French conjunction |
| The same pattern with two arguments | Fails: the pattern accepts one argument |
| Pattern `Code, %(num)s, %(symbol)s, English` with the named arguments 2 and a cheese symbol | `Code, 2, 🧀, Français, Belgium` |
| The same pattern with the second named argument being the list of `1`, `2` and `3` | `Code, 2, 1, 2 et 3, Français, Belgium` |
| The same pattern with only the second named argument supplied | Fails: a named argument is missing |
| The same pattern with the first named argument as markup and the second holding the cheese symbol between angle brackets | `Code, 2, &lt;🧀&gt;, Français, Belgium`, produced as markup |

### 7.5 Comparison and concatenation of deferred literals

A deferred literal is not a text value. Comparing it, ordering it or using it as a mapping key is refused; the comparison must be performed on its resolved text. Concatenating a deferred literal with text, on either side, resolves it first; concatenating two deferred literals resolves both.

**Worked example.** A deferred literal whose source is `Code Lazy, English` is owned by a package whose Klingon catalogue translates it to `Code Lazy, Klingon`.

| Operation | Result |
|---|---|
| Rendered as text with the session language `en_US` | `Code Lazy, English` |
| Rendered as text with the session language `tlh` | `Code Lazy, Klingon` |
| The text `Do you speak ` concatenated with the literal, session language `tlh` | `Do you speak Code Lazy, Klingon` |
| The literal concatenated with the text `, I speak it`, session language `tlh` | `Code Lazy, Klingon, I speak it` |
| The literal concatenated with itself, session language `tlh` | `Code Lazy, KlingonCode Lazy, Klingon` |
| The literal compared to a text value | Refused |

### 7.6 Client-side literals

Literals inside client code and client templates are marked with the client's own marking, immediate and deferred. They are delivered to the client as a translation table per package and per language, built from the same catalogues as specified in section 9.5. A package's client translations are delivered only when the package participates in the public-site translation set; a package joins that set either by carrying the public-site naming convention or by declaring itself a contributor through the request-handling extension point.

## 8. Translations of shipped data

### 8.1 The translated-column syntax

A data file may carry, next to the source value of a translatable field, the value of that field in one or more languages. The syntax is the field name, the translation marker `@`, and the language code, so that a name field translated into French as used in France is written as the name field's identifier followed by `@fr_FR`.

The syntax exists in both data-file shapes: as a second field element in a record document, and as a second column in a tabular data file.

### 8.2 Effect at load time

The ordinary data loader **ignores** every field whose name contains the translation marker: the record is created with its source value only. The translated values are consumed by the translation loader instead, which reads the same files a second time, once per installed language.

### 8.3 Effect on the exported catalogue

A record that carries at least one translated column inside its own package's data files is **excluded from the export entirely**, for every one of its fields. The reason is that the translation is already versioned with the package, so it must not be duplicated in a catalogue where a translator could change it independently and produce two competing sources of truth.

The exclusion is computed before the export: every data file and demonstration file of the package that is a record document or a tabular file is read, and every triple of entity, package and identifier that carries a translated column is collected.

### 8.4 Reading order and language matching

When a data file is read as a translation source for a target language, the translated columns are sorted by language code, which puts the more general languages first, and each row produces one entry per translated column. An entry is kept only when its language belongs to the base language chain of the target language, and the value is stored under the **target** language.

**Worked example**, loading a package for the target language `fr_BE`:

| Column in the data file | Kept for the target `fr_BE`? | Stored under |
|---|---|---|
| The name field followed by `@fr` | Yes: `fr` belongs to the chain of `fr_BE` | `fr_BE` |
| The name field followed by `@fr_BE` | Yes | `fr_BE` |
| The name field followed by `@fr_FR` | No: `fr_FR` does not belong to the chain of `fr_BE` | nothing |
| The name field followed by `@nl` | No | nothing |

Because the columns are processed from the most general to the most specific, a value declared for `fr` is overwritten by a value declared for `fr_BE` in the same row.

## 9. Translation catalogues

### 9.1 Catalogue files and their lookup order

A package carries its catalogues in two directories: the primary translation directory and the supplementary translation directory. Both are searched. For a target language the files are looked up in the order produced by taking each base language of the target language, from the most general to the most specific, and, for each of them, the file named after that base language in the primary directory and then the one in the supplementary directory.

**Worked example** for the target language `fr_BE`: the primary directory's `fr` file, the supplementary directory's `fr` file, the primary directory's `fr_BE` file, and the supplementary directory's `fr_BE` file.

Files are loaded in that order and a later file overrides an earlier one for the same source term, which is what makes a regional catalogue a delta over the base one.

The template file, holding every source term with an empty translation, is named after the package and lives in the primary directory.

### 9.2 The catalogue entry

| Part | Content |
|---|---|
| Source term | The text in the source language. |
| Translation | The text in the catalogue's language, empty when untranslated. |
| Owning packages | One or more package names, recorded as a comment line reading `module: <name>` for one package and `modules: <name>, <name>` for several. |
| Extra comments | Every comment line of the entry that is not the package line. Two markers are meaningful: the behaviour-literal marker and the client-literal marker, described in section 9.5. |
| References | One or more occurrences describing where the term comes from. |

Three reference kinds exist.

| Kind | Form | Meaning |
|---|---|---|
| Whole-value field | `model:<entity>,<field>:<package>.<identifier>` | The value of a whole-value translatable field of the record with that external identifier. |
| Term-level field | `model_terms:<entity>,<field>:<package>.<identifier>` | One term of a term-level translatable field of that record. |
| Behaviour or client literal | `code:<path>` followed by a line number | A literal marked in the package's code, client code, client template or spreadsheet data. |

A reference of any other kind is reported with `malformed translation catalog: unknown reference: <reference>`.

When a catalogue file is read from a path and a template file for the same package exists beside it, the entries of the template are merged into the read entries, which refreshes the references without touching the translations.

A term may not contain an escaped line break. One that does is fatal: `Translation terms may not include escaped newlines ('\n'), please use only literal newlines! (in '<term>')`.

### 9.3 Export

Two export scopes exist.

**Package scope**, for a list of package names or for every installed package:

1. Compute the exclusions: every triple of entity, package and identifier that the selected packages translate inside their own data files, as specified in section 8.3.
2. Collect the rows: for every external identifier whose package is one of the selected packages, grouped by the triple of entity, record and package, taking the smallest identifier name, ordered by package, then entity, then identifier name.
3. For each entity and the records it groups: read the records that exist. Missing ones are reported with `Unable to find records of type '<entity>' with external identifiers <names>`. An unknown entity is reported with `Unable to find object '<entity>'` and skipped. An entity marked non-translatable is skipped.
4. For the selection-value registry, drop the values whose field belongs to a non-translatable entity or whose field no longer exists. For the field registry, drop the fields that belong to a non-translatable entity or that no longer exist.
5. For each record and each field of the record: skip unless the field is translatable and stored; skip the name field of the abstract action entity, because it is stored as a shared column of the action hierarchy and would otherwise be exported twice; skip the label field of a field-registry record whose field does not export its label translations.
6. Read the record's value in the source language and in the export language, build the translation dictionary from the two, and push one entry per source term: the kind is the term-level kind when the field declares a term-splitting rule and the whole-value kind otherwise, the name is the entity name, a comma and the field name, the reference is the qualified identifier, the source is the term, and the translation is the translated term, or empty when the two are equal.
7. Then export the resource terms of section 9.5.

**Record scope**, for a list of record keys of one entity and an optional list of field names:

1. If the entity delegates to parent entities, group the requested field names that are translatable, not stored and derived from a parent by parent entity, and recurse into the linked parent records for those names.
2. If no requested field is both translatable and stored, stop.
3. Ensure every record carries an external identifier, creating the missing ones under the reserved export package, named with the table name, an underscore, the record key, an underscore and eight random hexadecimal digits.
4. Read the smallest qualified identifier per record.
5. Export as in the package scope.

Only terms containing at least one alphabetic character are exported: a term made only of punctuation, digits or spaces is dropped, which keeps separators and numeric literals out of the catalogue.

A term-extraction failure during the export is logged as `Failed to extract terms from <identifier> <entity>,<field>`, and a resource-extraction failure as `Failed to extract terms from <path>`; the export continues in both cases.

### 9.4 Catalogue file production

The catalogue writer groups the exported entries by source term.

1. For each exported row, made of the package, the kind, the name, the reference, the source, the translation and the comments: add the package to the group's package set; when the group has no translation yet and the translation differs from the source, take that translation; add the reference triple; add the comments.
2. For each source, in ascending order of the source text: when no language was requested, which produces a template, or when no translation was found, the translation is empty. Write one entry carrying the sorted package list as the package comment line, the sorted extra comments, the sorted references, the source and the translation.
3. Render a literal reference as `code:<path>` with the line number forced to zero, and a field-value reference as the kind, a colon, the name, a colon and the reference, with no line number.

Forcing the line number of a literal reference to zero keeps the catalogue stable when code moves inside a file, so that a pure refactoring produces no catalogue difference.

The file header lists the packages the file covers, one per line. The metadata block carries the project name and version, the creation and revision instants, the content type and encoding, and empty translator, team and plural-form entries.

Three output shapes exist.

| Shape | Content |
|---|---|
| Catalogue | One file holding every entry, for one language, or a template when no language is chosen. |
| Tabular | One table whose header row is the package, the kind, the name, the record reference, the source, the value and the comments, one row per exported row, with the comments joined by line breaks. |
| Archive | One catalogue per package, each placed at the conventional catalogue path inside that package's directory, named after the language, or after the package when producing a template. |

File naming of a produced export:

| Situation | Name |
|---|---|
| A language is chosen | The shortened form of that language's code |
| No language is chosen, that is a template | `new` |
| Entity scope | The entity name with full stops replaced by underscores |
| Exactly one package selected | That package's name |
| Extension | The one of the chosen shape; a template in the catalogue shape uses the template extension |

An unrecognised shape is refused with `Bad file format: <format>`.

### 9.5 Resource terms

Beyond record values, the package-scope export walks the file system for marked literals.

| Source | Where it is searched | Extractor | Comment marker added |
|---|---|---|---|
| Behaviour code | Every code file under the package directories and under the platform's own core directories, which are the entity layer, the persistence layer, the reporting layer, the package layer, the service layer and the tool layer, plus the files directly in the platform root | The code extractor, looking for the immediate and the deferred marking | The behaviour-literal marker |
| Client code | Client script files under a static source directory | The client code extractor, looking for the client marking | The client-literal marker |
| Client templates | Client template files under a static source directory | The template extractor of section 5.2 | The client-literal marker |
| Spreadsheet data | Files under a data directory whose name matches the dashboard naming convention | The spreadsheet extractor | The client-literal marker |

The spreadsheet extractor collects the arguments of the translation marking inside formulas, the label part of a link used as a cell value, chart titles, chart axis titles, scorecard baseline and key descriptions, and the labels of the global filters.

Each extracted term is pushed with the reference `code:` followed by the path relative to the package root prefixed with the package directory name, and with the line number, and with the translation already found in the package's catalogue for the export language. That last point is what makes an export in a language produce a complete catalogue rather than a template.

The package registration entity exposes an extension point that lets a package contribute further terms coming from stored attachments; the platform's own implementation contributes nothing.

### 9.6 Import

Reading a translation file produces rows. Four readers exist.

| Reader | Input | Notes |
|---|---|---|
| Catalogue reader | A catalogue file | Skips obsolete entries. Takes the package from the first package comment line. Emits one row per reference. |
| Tabular reader | A tabular translation file | The columns are the package, the kind, the name, the record reference, the source, the value and the comments. A numeric record reference is a database key; a non-numeric one is an external identifier split at the first qualification separator. For a field-value row the entity is the part of the name before the comma. Consecutive literal rows with the same source are collapsed into one. |
| Record-document data reader | A package data file in the record-document shape | Emits one row per translated column of every record element that carries at least one, sorted by language code. |
| Tabular data reader | A package data file in the tabular shape | Emits one row per translated column of every row, sorted by language code. The entity is taken from the file name. |

Filtering, applied to every row, with the valid languages being the base language chain of the target language:

1. Skip when the value is empty or the source is empty.
2. Skip when the kind is the literal kind. Literals are never imported.
3. Skip when the row's language, defaulting to the target language, is not in the valid set.
4. Skip when the entity is unknown.
5. Take the field named after the comma in the row's name. Skip when the field does not exist, is not translatable, or is not stored.
6. Build the qualified identifier from the row's package, the qualification separator and the row's identifier name. Skip when a restricting identifier list was supplied and the qualified identifier is not in it.
7. If the kind is the whole-value kind and the field is a whole-value field, record the value under the entity, the field, the qualified identifier and the target language.
8. If the kind is the term kind and the field is a term-level field, record the value under the entity, the field, the qualified identifier, the source term and the target language.

Rows accumulate across files, and the save step is performed once for a whole batch of files, which is what makes loading the catalogues of a hundred packages a small number of statements rather than one per file.

Two failures are logged rather than raised: `Couldn't read translation for lang '<code>', language not found` when the target language does not exist, and `couldn't read translation file [lang: <code>][format: <format>]` when the file cannot be read.

### 9.7 Saving imported translations

Three modes control how an imported value competes with a value already stored.

| Mode | Meaning |
|---|---|
| Keep existing, the default | A stored translation is never replaced. |
| Overwrite | A stored translation is replaced, **except** on records whose external identifier is flagged no-update. |
| Force overwrite | A stored translation is always replaced, including on records flagged no-update. |

**Term-level fields**, per entity, per field, per batch of identifiers:

1. Read the record key, the qualified identifier, the stored map and the no-update flag for every identifier.
2. For each record: if the stored map is empty, skip. Take the source as the source shadow key when present and the source key otherwise; if it is empty, skip.
3. Take the languages appearing in the imported rows for that record.
4. Build the translation dictionary from the source and, for every key of the stored map that is one of those languages, its shadow value when present and its plain value otherwise.
5. If the mode is force overwrite, or the mode is overwrite and the record is not flagged no-update: for every imported source term, update the dictionary entry with the imported translations.
6. Otherwise: for every imported source term, take the imported translations, update them with the stored translations that differ from the source term, and replace the dictionary entry with the result. The stored value therefore wins wherever it is a real translation rather than a copy of the source.
7. For each language, recompose the new value from the source and the dictionary, record the change when it differs from the stored value, and clear the shadow key of that language.
8. Apply every recorded change with a single statement per batch.

**Whole-value fields**, per entity, per field, per batch of identifiers, with one statement per batch: split the imported maps into those whose identifier is flagged no-update and the others, then merge them into the stored map with the following precedence, where "A over B" means that A wins on a key collision.

| Mode | Precedence |
|---|---|
| Force overwrite | The updatable imported values, over the no-update imported values, over the stored values |
| Overwrite | The updatable imported values, over the stored values, over the no-update imported values |
| Keep existing | The stored values, over the updatable imported values, over the no-update imported values |

Two identifiers pointing at the same record are merged deterministically in input order, the later one winning, which gives a record claimed by two packages the translation of the package loaded last.

After the save, every cached record value and every registry cache container is invalidated, and the informational line `translations are loaded successfully` is logged unless the importer was asked to stay silent.

## 10. Loading and updating languages

### 10.1 Updating the terms of a set of packages

1. If no language is given, take the codes of every installed language.
2. Keep, among the given packages, those whose state is installed, to install or to upgrade.
3. Order them topologically by their declared dependencies.
4. Load the terms in that order.

Loading the terms of an ordered list of packages for a list of languages:

1. Create a fresh importer.
2. For each package in order: skip it when it has no manifest on disk. For each language: for each catalogue path of that package and language, as determined in section 9.1, log `module <package>: loading translation file <path> for language <language>` and read it into the importer. Then, for each data file path of that package, meaning the record documents and tabular files listed in its data and demonstration attributes, read it into the importer as a data file for that language. When the language is not the source language and nothing was imported for it, log `module <package>: no translation for language <language>`.
3. Save the importer with the given mode.

The topological order matters: a package that overrides a record shipped by one of its dependencies must be imported after it, which makes its translation win under the merge rules of section 9.7.

### 10.2 When translations are loaded

| Event | Languages | Mode |
|---|---|---|
| A package is installed | Every installed language | The configured default overwrite setting |
| A package is upgraded | Every installed language | The configured default overwrite setting |
| A package is reinitialized | Every installed language | The configured default overwrite setting |
| A language is activated through the language screen | That language | Keep existing |
| A language is installed through the installation screen | The chosen languages | The screen's overwrite flag, which defaults to overwrite |
| A catalogue file is imported through the import screen | The chosen language | The screen's overwrite flag, which defaults to overwrite |

### 10.3 The Language Installation Wizard

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `lang_ids` | Languages | Many-to-many to Language, required | The languages to install. The selection shows archived languages as well. It defaults to the languages selected in the list the screen was opened from. |
| `overwrite` | Overwrite Existing Terms | Boolean, default true | Its help text reads `If you check this box, your customized translations will be overwritten and replaced by the official ones.` |
| `first_lang_id` | First Language | Many-to-one to Language, computed | The first chosen language, used to offer a switch. Its help text reads `Used when the user only selects one language and is given the option to switch to it` |

The install operation activates the chosen languages and updates the terms of every installed package for them with the chosen mode. When exactly one language was chosen, the screen reopens on a variant offering to switch the acting user to it; otherwise it closes with the notification `The languages that you selected have been successfully installed. Users can choose their favorite language in their preferences.` Activating languages directly from the language list produces the same notification.

### 10.4 The Translation Import Wizard

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `name` | Language Name | Text, required | The display name to give the language when it must be created. |
| `code` | Code | Text, required | The locale code. Its help text reads `ISO Language and Country code, e.g. en_US`, which names the standard language and region code and gives an example. |
| `data` | File | Binary, required | The catalogue or tabular translation file. |
| `filename` | File Name | Text, required | The file name, whose suffix selects the reader. |
| `overwrite` | Overwrite Existing Terms | Boolean, default true | Its help text reads `If you enable this option, existing translations (including custom ones) will be overwritten and replaced by those in this file` |

The import groups the requested imports by mode and, for each group, activates the named language, creating it from the locale when it does not exist, reads every file into one importer, and saves with that group's mode.

A file that cannot be read raises `File "<file name>" not imported due to format mismatch or a malformed file. (Valid formats are the tabular form and the catalog form)` followed by two line breaks, then `Technical Details:`, then the underlying reason.

Importing a catalogue never changes behaviour literals: those are read from the files shipped inside the package directories and are not stored in the database. A catalogue imported through this screen therefore updates record values only.

### 10.5 The Translation Export Wizard

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `name` | File Name | Text, read-only | The produced file name, as determined in section 9.4. |
| `lang` | Language | Selection, required, default the template marker | The installed languages, preceded by the entry `New Language (Empty translation template)`. |
| `format` | File Format | Selection, required, default the catalogue form | Three values: the tabular form, the catalogue form and the archive form. |
| `export_type` | Export Type | Selection, required, default the package scope | Two values: the package scope and the entity scope. |
| `modules` | Apps To Export | Many-to-many to Capability Package | The packages to export, restricted to installed packages. |
| `model_id` | Model | Many-to-one to Entity | The entity to export, restricted to non-transient entities. |
| `model_name` | Model Name | Text, computed from the entity | The transport name of the chosen entity. |
| `domain` | Domain | Text holding a condition, default the empty condition | The record filter applied in the entity scope. |
| `data` | File | Binary, read-only | The produced file. |
| `state` | State | Selection, default `choose` | `choose` before the export, `get` afterwards. |

The export operation resolves the language, which is empty when the template marker is chosen, runs the record-scope or the package-scope export, encodes the result, and reopens the screen in the `get` state offering the file. When no package is selected in the package scope, every installed package is exported.

An unsupported format raises `Unrecognized extension: must be one of the tabular form, the catalog form or the archive form (received <given>).`

## 11. The external translation service

A capability package integrates an external translation service, so that a reader of a translated term can open that term in the service and propose a correction. The integration stores nothing that the platform itself needs: it is a link builder plus a locally cached copy of the behaviour-literal catalogues.

### 11.1 The Code Translation entity

The transport name is `transifex.code.translation` and the full name is Code Translation. The entity keeps no creation or update stamps.

| Identifier | Full name | Type | Meaning |
|---|---|---|---|
| `source` | Code | Long text | The source term of a behaviour literal. |
| `value` | Translation Value | Long text | Its translation in the row's language. |
| `module` | Module | Text | The capability package the term belongs to. |
| `lang` | Language | Selection over the installed languages, not validated against the selection | The language of the translation. |
| `transifex_url` | Service address | Text, computed, not stored | The address at which the term may be edited in the external service. |

The language field is deliberately not validated against its selection, because a row may survive the deactivation of its language.

### 11.2 Loading the cached catalogues

1. Take an exclusive lock on the table without waiting. If the lock is not available, stop and report that nothing was loaded; another worker is already loading. The lock is what promises that the translations of a given pair of package and language are created exactly once.
2. If no package list was given, take every installed package. If no language list was given, take every installed language except the source language.
3. Read the distinct pairs of package and language already present in the table.
4. For each package and each language whose pair is not already present, read that package's behaviour-literal catalogue for that language and build one row per source term and translation.
5. Create the rows with elevated rights.

The reload operation deletes every row of the table and then performs the load above. It is the body of a scheduled job that runs every 7 days, so that a catalogue changed in the packages on disk is reflected in the table within a week. The job is listed in [`scheduled-jobs.md`](scheduled-jobs.md), section 12.3.

Opening the code-translation screen performs the load first, so that a screen opened before the job has ever run is not empty.

### 11.3 Building the service address

The address of the external service's project is held in the system parameter `transifex.project_url`. When it is unset, no address is produced and the feature is inert; this is how a deployment turns the integration off without uninstalling it.

1. Read the parameter and strip any trailing separator. Stop when it is empty.
2. Build the map from language code to standard language code from every Language record. Stop when it is empty.
3. Read the map from package name to service project name, which is cached. It is built by reading the service configuration files found beside the package directories: each section names a project and a package, in either the two-part or the six-part form. Stop when the map is empty.
4. For each translation to decorate: skip it when its source is empty or its language is the source language; skip it when its language has no standard language code; skip it when its package has no service project.
5. Take the first 50 characters of the source, remove the line breaks, escape the apostrophes, and encode the result for use in a web address. When the encoded form contains a plus sign, wrap it in apostrophes so that the service reads it as one phrase.
6. Build the address from the project base address, the project name, the fixed translate segment, the standard language code, the package name, an arbitrary numeric segment that the service's address shape requires, and a query naming the escaped source as a text search.

### 11.4 Decorating a record's translations

The generic operation that returns the translations of one field of one record is extended: after the ordinary translations have been read, the record's external identifier is resolved. When the record has none, or when the package part of that identifier is not among the packages being initialized, nothing is added. Otherwise every returned translation is stamped with that package name and decorated with the service address of section 11.3.

The result of the operation is therefore a list of entries, each carrying the language, the source term, the value, the package and the service address, together with a context stating the translation kind and whether the source is shown beside the translation.

## 12. Rendering translated text

### 12.1 Views

The structure of a view is stored in a term-level translatable field. Reading a view in a session resolves every term in the session language through the fallback chain, so the client receives a structure whose labels, help texts, placeholders, confirmation messages and button labels are already translated. The unmodified source structure is available through a separate derived field that reads with no language.

Because the terms of a view are matched positionally against the source structure, a view extension that inserts or removes a translatable node shifts the positions. The re-matching of section 6.3 restores the alignment for the terms whose textual content is unchanged and drops the translations of the terms that disappeared.

A parse failure while translating a view is reported as `Error while parsing view:` followed by two line breaks and the underlying reason.

### 12.2 Field labels and help texts

A field's label and help text are declared by the package in the source language and are reflected into the field registry, where they are stored as whole-value translatable fields. At render time:

1. If the field declares no label, or the session carries no language, return the declared label.
2. Otherwise read the cached label table of the field's entity for the session language and return the entry for that field name, falling back to the declared label.

The help text follows the same rule with the help table. The tables are cached per entity and per language and are invalidated whenever the registry cache containers are cleared, which a translation import always does.

A field may declare that its label must not be exported for translation. Such a field's label is excluded from the catalogue and its field-registry record's label is skipped by the export. This is used for fields whose label is a technical code that must stay identical in every language.

### 12.3 Closed-list labels

Every value of a closed list declared in a package is reflected into the selection-value registry with its label stored as a whole-value translatable field. At render time the label of a value is read from a cache keyed by entity and language, exactly like a field label, and falls back to the declared label when no translation exists.

Three consequences must be reproduced.

- A value introduced by an extension list is translated by the package that declared it with a label; an entry that only reorders existing values introduces no label and therefore no translatable term.
- Removing a package removes its values and their translations.
- A value list produced by a rule rather than enumerated carries no registry records and therefore no stored translations; its labels must be produced by the rule as behaviour literals, which are translated through section 7.

### 12.4 Menus, actions and shipped names

Menu item names, action names, report labels, scheduled job names, message template subjects and every other name field declared translatable follow the whole-value rules: one value per language inside the record, resolved through the fallback chain, imported from the catalogues, and protected from overwrite when the record's external identifier is flagged no-update.

### 12.5 Printed documents

A printed document is rendered from template views, which are term-level translatable exactly like screen views. Two language rules apply.

1. The rendering session's language decides the translated terms and every date, number and currency format.
2. A template may call a sub-template in an explicit language through the language directive of the template grammar; the sub-template and everything it renders use that language. The shipped document templates use this to render each copy of a document in the language preferred by the recipient of that copy.

When a rendered document is split into several parts before being converted, each part carries the language it was rendered in, and the surrounding layout is re-rendered in that same language, which makes headers and footers match the body. The mechanism is specified in [`report-rendering.md`](report-rendering.md), sections 4.6 and 4.14.

The printed name of a document is produced from a translatable pattern declared on the report, resolved in the rendering session's language.

### 12.6 Messages produced by behaviour

Validation messages, warnings, notifications and errors are behaviour literals: they are resolved at the moment they are produced, in the language determined by section 7.3. A message that must be read by a specific person, such as a customer or an approver, is produced in a session whose language is that person's preferred language.

## 13. Worked examples

### 13.1 Whole-value translation of a shipped record

A package ships, in a group that is not flagged no-update, a record with the identifier `record1` whose translatable name is `Tableware`. It also ships a catalogue for `fr_FR` holding a whole-value reference to that record's name field, the source `Tableware` and the translation `Vaisselle`.

| Read | Result |
|---|---|
| With no language | `Tableware` |
| In `fr_FR` | `Vaisselle` |
| In `fr_BE` | `Tableware`: the chain of `fr_BE` is `fr` then `fr_BE`, and the catalogue was stored under `fr_FR`, which is not in that chain, so nothing was stored for `fr_BE` and the source is returned |

To make the same translation apply to Belgian French, the catalogue must be named after `fr`, in which case the loader stores it under every installed target language whose chain contains `fr`.

### 13.2 Regional override

A package carries a general French catalogue translating `Tableware` to `Vaisselle`, and a Belgian French catalogue translating it to `Vaisselle, Belgium`.

| Active language | Read in that language | Why |
|---|---|---|
| `fr_BE` | `Vaisselle, Belgium` | The `fr_BE` file is read after the `fr` file and overrides it. |
| `fr_CA`, French as used in Canada | `Vaisselle` | Only the `fr` file matches the chain of `fr_CA`. |

The same holds for behaviour literals and, term by term, for term-level fields. A source value holds three terms, `Fork` as an attribute of the outer element and `Knife` and `Spoon` as the text of two inner blocks. The general French catalogue translates all three; the Belgian French catalogue translates all three with the region appended; the Canadian French catalogue translates only `Knife`.

| Read in | Attribute term | First inner term | Second inner term |
|---|---|---|---|
| `fr_BE` | `Fourchette, Belgium` | `Couteau, Belgium` | `Cuillère, Belgium` |
| `fr_CA` | `Fourchette` | `Couteau, Canada` | `Cuillère` |

### 13.3 The no-update rule protects a customized translation

A package ships a menu item in a group flagged no-update, named `Test translation model1`, together with a catalogue translating it to `Test translation import in french`.

| Step | Effect |
|---|---|
| 1 | The package is installed with `fr_FR` active. The menu item reads `Test translation import in french` in French. |
| 2 | A user renames it in French to `Nouveau nom`. |
| 3 | The package's terms are re-imported with the overwrite mode. The menu item still reads `Nouveau nom` in French, and still reads `Test translation model1` in the source language. |

Had the menu item been shipped outside a no-update group, the overwrite mode would have restored `Test translation import in french`. Had the import used the force-overwrite mode, it would have restored it even inside the no-update group.

### 13.4 Two identifiers on one record

A package ships a record with the identifier `record1` and the name `Tableware`, and a second external identifier `record1_alias` pointing at the same record. Its catalogue holds two entries: one referencing `record1` with the source `Tableware` translated as `Assiettes`, and one referencing `record1_alias` with the source `Tableware Unused` translated as `Vaisselle`.

Importing with the overwrite mode makes the record read `Vaisselle` in French, because the merge of section 9.7 is applied in input order and the later entry wins.

### 13.5 Round trip

| Step | Effect |
|---|---|
| 1 | The package is exported for `fr_FR` in the catalogue shape. The produced file holds every record term with its French value, every behaviour literal with its French value and every client literal with its French value. |
| 2 | The French values of a record are cleared by rewriting its source value in the source language. Reading the record in French now returns the source value. |
| 3 | The exported file is imported with the keep-existing mode. The French values are restored exactly, because the stored map holds nothing for French and the imported values therefore win. |

Exporting a package as a template, with no language chosen, produces exactly the shipped template file, except that records whose translations live in the package's own data files are absent, as specified in section 8.3.

### 13.6 Delegation and record-scope export

An entity has a whole-value translatable name field and a term-level translatable markup field. A second entity delegates to the first through a link field and declares no translatable stored field of its own.

| Export | Terms produced |
|---|---|
| The records of the first entity for `fr_FR` | `Fork` to `Fourchette`, `Furniture` to `Meuble`, `Knife` to `Couteau`, `Spoon` to `Cuillère`, `Tableware` to `Vaisselle` |
| The records of the second entity for `fr_FR` | `Fork` to `Fourchette`, `Knife` to `Couteau`, `Spoon` to `Cuillère`, `Tableware` to `Vaisselle` |

The second export contains the terms of the linked parent record, reached through the delegation, and nothing of its own, because the child declares no translatable stored field. The term `Furniture` is absent because it belongs to a record of the first entity that no record of the second entity delegates to.

## 14. Error and message catalogue

| Condition | Severity | Message |
|---|---|---|
| Session language not active | User error | `Invalid language code: <code>` |
| No active language would remain | Validation | `At least one language must be active.` |
| Forbidden date or time directive | Validation | `Invalid date/time format directive specified. Please refer to the list of allowed directives, displayed when you edit a language.` |
| Language code modified | User error | `Language code cannot be modified.` |
| Language in use by users | User error | `Cannot deactivate a language that is currently used by users.` |
| Language in use by contacts | User error | `Cannot deactivate a language that is currently used by contacts.` |
| Language used by automated processes | User error | `You cannot archive the language in which the system was set up as it is used by automated processes.` |
| Deleting the source language | User error | `Base Language 'en_US' can not be deleted.` |
| Deleting the session language | User error | `You cannot delete the language which is the user's preferred language.` |
| Deleting an active language | User error | `You cannot delete the language which is Active!` then a line break then `Please de-activate the language first.` |
| Duplicate language name | Validation | `The name of the language must be unique!` |
| Duplicate language code | Validation | `The code of the language must be unique!` |
| Duplicate address code | Validation | `The web address code of the language must be unique!` |
| Formatting a number for an inactive language | User error | `The language <name> is not installed.` |
| Wrong format specifier | Value error | `format() must be given exactly one %char format specifier` |
| Reading a language field that is not cached | User error | `Field "<name>" is not cached` |
| Locale unavailable when creating a language | Warning | `Unable to get information for locale <code>. Information from the default locale (<current>) have been used.` |
| Importing for an unknown language | Error log | `Couldn't read translation for lang '<code>', language not found` |
| Unreadable translation file | Error log | `couldn't read translation file [lang: <code>][format: <format>]` |
| Import screen, unreadable file | User error | `File "<name>" not imported due to format mismatch or a malformed file. (Valid formats are the tabular form and the catalog form)` then two line breaks then `Technical Details:` then a line break then the reason |
| Unsupported export shape | User error | `Unrecognized extension: must be one of the tabular form, the catalog form or the archive form (received <given>).` |
| Unknown translation file shape | User error | `Bad file format: <format>` |
| Unknown reference kind in a catalogue | Error log | `malformed translation catalog: unknown reference: <reference>` |
| Record referenced by the export is missing | Warning | `Unable to find records of type '<entity>' with external identifiers <names>` |
| Entity referenced by the export is unknown | Error log | `Unable to find object '<entity>'` |
| Term extraction failure during the export | Error log | `Failed to extract terms from <identifier> <entity>,<field>` |
| Resource extraction failure during the export | Error log | `Failed to extract terms from <path>` |
| Malformed rich text | Error log | `Cannot translate malformed rich text, using source value instead` |
| View parse failure while translating | User error | `Error while parsing view:` then two line breaks then the reason |
| Catalogue file loaded | Information | `module <package>: loading translation file <path> for language <language>` |
| No catalogue for a language | Information | `module <package>: no translation for language <language>` |
| Import finished | Information | `translations are loaded successfully` |
| Translation substitution failure | Error log | `Bad translation <translation> for string <source>` |
| No language could be detected for a literal | Warning | `no translation language detected, skipping translation <location>` |
| Per-company field declared translatable | Warning | `company_dependent field <field> cannot be translated` |
| Stored translated field depending on the session | Warning | `Translated stored fields (<field>) cannot depend on the session` |
| Stored path-following translated field | Warning | `Translated stored path-following field (<field>) will not be computed correctly in all languages` |
| Escaped line break inside a catalogue term | Fatal | `Translation terms may not include escaped newlines ('\n'), please use only literal newlines! (in '<term>')` |

## 15. Acceptance criteria

1. **Given** a record whose translatable name is `Tableware` in the source language, **when** the French value is set to `Vaisselle`, **then** reading with no language returns `Tableware`, reading in French returns `Vaisselle`, and both values are held in the same record.
2. **Given** a record with no Dutch value, **when** it is read in Dutch, **then** the source value is returned.
3. **Given** a record with a source value and a French value, **when** the field is set to empty, **then** both values are gone and reading in either language returns empty.
4. **Given** a term-level value with three terms, all translated in French, **when** the source is rewritten with the first term unchanged, the second reworded beyond the similarity threshold and the third removed, **then** in French the first term keeps its translation and the second reads as its new source text.
5. **Given** a term-level value whose term `Knife` is translated to `Couteau`, **when** the source term becomes `Knife ` with a trailing space, **then** the similarity is 0.9091, which is at least 0.9, and the translation is carried over.
6. **Given** a translated view term carrying a node with a visibility condition and a package upgrade that changes only that condition on the source term, **when** the package is installed and the source term is rewritten, **then** the French term keeps its translated text and adopts the new condition; **given** the two structures do not match node for node, **then** the translation is dropped instead.
7. **Given** the codes `fr_BE`, `es_MX`, `zh_HK`, `es_ES` and `de`, **when** their chains are computed, **then** they are `fr` and `fr_BE`; `es`, `es_419` and `es_MX`; `zh`, `zh_TW` and `zh_HK`; `es` and `es_ES`; and `de` alone.
8. **Given** a package carrying a general French catalogue and a Belgian French catalogue for the same source term, **when** its terms are loaded for Belgian French, **then** the Belgian value is stored; **when** they are loaded for Canadian French, **then** the general value is stored.
9. **Given** a behaviour producing a marked literal, **when** it runs in a French session and the package carries a French catalogue entry for that literal, **then** the French text is produced; **when** the package carries no entry, **then** the source text is produced.
10. **Given** a deferred literal declared outside any session, **when** it is rendered as text in the source language and then in another language, **then** it produces the source text and the translated text respectively, and comparing it directly to a text value is refused.
11. **Given** the calls listed in section 7.4, **when** each is evaluated in Belgian French, **then** the results are exactly as listed, including the list enumeration, the byte-sequence handling, the markup escaping and the two failures.
12. **Given** a catalogue holding literal entries, **when** it is imported through the import screen, **then** no stored record changes for those entries, and the behaviour still produces the text held in the package's own catalogue files.
13. **Given** a record document declaring both the name field and the same field with the `@fr_FR` marker, **when** the package is installed with French active, **then** the source value is the plain field's value, the French value is the marked field's value, and the loader never wrote the marked column as a source value.
14. **Given** a record whose French value is declared in the package's data file, **when** the package is exported as a template, **then** that record's term does not appear in the produced file.
15. **Given** a tabular data file with an identifier column, a name column and a name column marked `@fr_FR`, **when** the package is installed with French active, **then** the record carries both values and the marked column was ignored by the record loader.
16. **Given** the columns of section 8.4 and the target language Belgian French, **when** the package's terms are loaded, **then** only the `@fr` and `@fr_BE` columns are applied, in that order, and both are stored under Belgian French.
17. **Given** the scenario of section 13.3, **when** the terms are re-imported with the overwrite mode, **then** the customized French value survives and the source value is unchanged; **given** the record is not flagged no-update, **then** the shipped French value replaces the customization.
18. **Given** the same record flagged no-update with a customized French value, **when** the terms are imported with the force-overwrite mode, **then** the shipped French value replaces the customization.
19. **Given** a record with an existing French value different from the shipped one, **when** the terms are imported with the keep-existing mode, **then** the existing value survives; **given** the record has no French value at all, **then** the shipped value is stored.
20. **Given** the scenario of section 13.4, **when** the terms are imported with the overwrite mode, **then** the record's French value is the one of the later entry.
21. **Given** a package installed with French active and its terms loaded, **when** the package is exported for French, the record values are reset to their source values, and the exported file is imported with the keep-existing mode, **then** every record value is identical to what it was before the reset, for both whole-value and term-level fields.
22. **Given** a package carrying a behaviour literal, a client literal and a client template term, **when** it is exported for French, **then** the behaviour literal carries the behaviour marker comment, the client literal and the template term carry the client marker comment, no client-only term appears in the behaviour translation table, and no behaviour-only term appears in the client translation table.
23. **Given** a value whose terms include a colon, an ellipsis, the text `.00` and the word `Fork`, **when** the package is exported, **then** only `Fork` appears in the produced file.
24. **Given** an entity marked non-translatable holding records with translatable fields and external identifiers, **when** its package is exported, **then** none of its values appears in the produced file, and neither do its field labels nor its closed-list labels.
25. **Given** a field declared with label export disabled, **when** its package is exported, **then** the field's label does not appear in the produced file, while the labels of the entity's other fields do.
26. **Given** records with no external identifier, **when** they are exported in the record scope, **then** each receives an external identifier qualified by the reserved export package, named after the table, the record key and eight random hexadecimal digits, and the produced file references those identifiers.
27. **Given** the entities of section 13.6, **when** the child records are exported, **then** the terms of the linked parent records appear, resolved through the delegation.
28. **Given** a field whose label is `Name` and whose French label is `Nom`, **when** the entity's fields are described in a French session, **then** the label is `Nom`; in a session with no language it is `Name`.
29. **Given** a closed list with the values `draft` labelled `Draft` and `posted` labelled `Posted`, both translated into French, **when** the list is described in a French session, **then** the French labels are returned in the declared order and the stored values are unchanged.
30. **Given** a field with a help text translated into French, **when** the entity's fields are described in a French session, **then** the French help text is returned.
31. **Given** a view whose labels are translated into French, **when** it is read in a French session, **then** every translatable label, help text, placeholder and button label is in French, and the source structure is still available through the untranslated reading.
32. **Given** a document template that calls its body sub-template in the recipient's language and a recipient whose preferred language is French, **when** the document is printed by a user whose language is English, **then** the body is rendered in French, including its dates, numbers and currency formatting, and the surrounding layout is rendered in the same language.
33. **Given** a child entity whose display name comes from a translated field of its delegation parent, **when** the child is read in two languages, **then** the two display names differ accordingly.
34. **Given** an archived language and a set of installed packages carrying catalogues for it, **when** the language is activated, **then** the catalogues of every installed package are imported for it, in dependency order.
35. **Given** a language preferred by at least one user, **when** it is deactivated, **then** the operation is refused with `Cannot deactivate a language that is currently used by users.`; the same holds for a contact and for an archived user, with their own messages.
36. **Given** an installation in which the source language is archived and the session language is French, **when** a translatable field is written, **then** both the French value and the source value hold the written text.
37. **Given** the two grouping patterns of section 2.5, **when** the value 123456789 is formatted with grouping, **then** the results are `123,456,789` and `12,34,56,789` respectively, using the language's own group separator.
38. **Given** an existing language, **when** its code is written with a different value, **then** the write is refused with `Language code cannot be modified.`
39. **Given** an inactive language whose address code is the bare language subtag, and another language with that same subtag and a region subtag being activated, **when** the activation happens, **then** the inactive language's address code becomes its full code and the activated language takes the bare subtag.
40. **Given** a session carrying a language code for which no active language exists, **when** any read is attempted, **then** it fails with `Invalid language code: <code>`.
41. **Given** a term-level field written with the delayed-translation marker set, **when** an ordinary read follows, **then** the previous rendering is returned; **when** a read in translation-checking mode follows, **then** the freshly recomposed value is returned.
42. **Given** a term-level field whose stored map holds a shadow key for French, **when** a translation import rewrites French, **then** the shadow key is cleared.
43. **Given** the external service integration with no project address stored, **when** a translation is read, **then** no service address is produced and nothing fails.
44. **Given** the code-translation table already holding rows for a package and a language, **when** the load runs again, **then** no duplicate row is created for that pair.
45. **Given** two workers running the load at the same time, **when** the second cannot take the table lock, **then** it stops without creating any row and reports that nothing was loaded.
46. **Given** the reload job, **when** it runs, **then** every row of the code-translation table is deleted and the table is rebuilt from the catalogues on disk.
47. **Given** a translation whose language is the source language, **when** the service address is built, **then** no address is produced for it.
48. **Given** a translated field of a record whose external identifier belongs to a package that is not being initialized, **when** the translations are read, **then** no package name and no service address are added.

## 16. Reconciliation notes

1. The field identifiers of the Language entity were paraphrased. They are reproduced exactly: `name`, `code`, `iso_code`, `url_code`, `active`, `direction`, `date_format`, `time_format`, `week_start`, `grouping`, `decimal_point`, `thousands_sep`, `flag_image` and `flag_image_url`. The same applies to the three wizards, whose identifiers are reproduced in sections 10.3 to 10.5.
2. The sections were reordered so that this document reads storage first and resolution second, which is also the order the rest of this folder refers to: the storage of a translatable value is section 3 and the fallback chain is section 4. No content moved between the two subjects.
3. The external translation service was named in the job catalogue of [`scheduled-jobs.md`](scheduled-jobs.md) but was not specified in either source. It is specified in full in section 11, including the exclusive lock that makes the load idempotent, the parameter that turns the integration off, and the six steps that build the service address.
4. The source document illustrated the data-file syntax and the term-level worked examples with fragments of markup and of data files. Markup fragments are not admissible in this repository, so those illustrations are restated as prose and as tables. Every value, every language code and every expected result is unchanged.
5. The source document listed two modes for the import and named a third in the procedure. All three are enumerated in section 9.7, with the precedence table that distinguishes overwrite from force overwrite.
6. The similarity threshold of 0.9 was stated without its arithmetic. The formula and a worked example are given in section 6.3, so that a replacement can reproduce the borderline cases.
