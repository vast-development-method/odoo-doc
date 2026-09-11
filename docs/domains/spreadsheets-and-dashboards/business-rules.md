# Business rules

Sixty-six numbered rules. Each rule states the condition that makes it fire, the exact text shown when it refuses, the placeholders of that text described in words, and where the rule is applied. Every quoted message is reproduced, not authored; a rebuild must produce the same text with its own values substituted.

The index of all rule identifiers is §14.

## 1. Naming and scope of the rules

The prefix `SD` stands for this domain. Numbers are stable: a rule that is superseded keeps its number. A rule that refuses with no message — a storage constraint, a record rule, an access right — says so explicitly and names what the reader sees instead.

## 2. Workbook integrity

### SD-001

**Subject.** The stored workbook of any record carrying the Spreadsheet Document mixin.

**Fires when.** The content of `spreadsheet_binary_data` is not empty and cannot be decoded as text, or the decoded text is not a well-formed structured document.

**Message.** "Uh-oh! Looks like the spreadsheet file contains invalid data."

**Placeholders.** None.

**Applied at.** Creation, every write of `spreadsheet_binary_data`, every write of `spreadsheet_data` — which passes through the binary field — and interactively while a form is open, before saving.

### SD-002

**Subject.** The references inside a stored workbook.

**Fires when.** The reference walk of [`document-format.md`](document-format.md) §7 produced at least one finding.

**Message.** "Uh-oh! Looks like the spreadsheet file contains invalid data.\n\n%(errors)s", where the escape shown stands for two line breaks and the placeholder for the findings joined by single line breaks.

**Placeholders.** `%(errors)s` — the list of findings, one per line. Each finding takes one of four forms, reproduced exactly:

| Finding | Meaning |
|---|---|
| `- model '<entity>' used in '<record name>' does not exist` | A data-bound element names an entity that is not installed |
| `- field '<segment>' used in spreadsheet '<record name>' does not exist on model '<entity>'` | A field path names a field that the entity reached so far does not have |
| `- xml id '<identifier>' used in spreadsheet '<record name>' does not exist` | A chart or a cell link names a menu external identifier that resolves to nothing |
| `- menu with xml id '<identifier>' used in spreadsheet '<record name>' does not have an action` | A menu that is not a root menu is named but opens nothing |

The record name in each finding is the display name of the record being validated.

**Applied at.** The same points as SD-001, but only while the installation is executing its automated test suite. See the compatibility finding in [`document-format.md`](document-format.md) §7.5.

### SD-003

**Subject.** A stored workbook that is a packaged workbook file rather than a native document.

**Fires when.** Never; this rule grants rather than refuses. A decoded content that declares the key `[Content_Types].xml` is accepted without the reference walk of SD-002.

**Message.** None.

## 3. Required values

### SD-004

**Subject.** Spreadsheet Dashboard.

**Fires when.** `name` is empty.

**Message.** The platform's own required-field refusal, naming the field by its label "Name". No message is authored by this domain.

**Applied at.** Creation and write, and in storage, where the column is declared not null.

### SD-005

**Subject.** Spreadsheet Dashboard.

**Fires when.** `dashboard_group_id` is empty.

**Message.** The platform's own required-field refusal, naming the field by its label "Dashboard Group".

**Applied at.** Creation and write, and in storage.

### SD-006

**Subject.** Dashboard Group.

**Fires when.** `name` is empty.

**Message.** The platform's own required-field refusal, naming the field by its label "Name".

**Applied at.** Creation and write, and in storage.

### SD-007

**Subject.** Dashboard Share.

**Fires when.** `dashboard_id` is empty.

**Message.** The platform's own required-field refusal.

**Applied at.** Creation and write, and in storage.

### SD-008

**Subject.** Dashboard Share.

**Fires when.** `access_token` is empty.

**Message.** The platform's own required-field refusal.

**Applied at.** Creation and write, and in storage. In practice the field is never empty, because its default generates a universally unique identifier.

## 4. Deletion and duplication

### SD-009

**Subject.** Dashboard Group.

**Fires when.** A group is being deleted, it has an external identifier, and that identifier does not begin with `__export__`.

**Message.** "You cannot delete %s as it is used in another module."

**Placeholders.** `%s` — the group's name.

**Applied at.** Ordinary deletion. The guard is suspended while the capability package that owns the group is itself being removed.

### SD-010

**Subject.** Dashboard Group.

**Fires when.** A group is being deleted while at least one Spreadsheet Dashboard still refers to it.

**Message.** None from this domain. The link is declared with deletion restricted, so storage refuses and the platform reports that other records depend on this one.

**Applied at.** Deletion.

### SD-011

**Subject.** Spreadsheet Dashboard.

**Fires when.** Never; this rule states a consequence. Deleting a dashboard deletes every Dashboard Share that copies it, because that link cascades, and deletes the attachments holding its workbook and its thumbnail.

**Message.** None.

### SD-012

**Subject.** Spreadsheet Dashboard.

**Fires when.** A dashboard is duplicated and the caller supplied no name.

**Effect.** The copy's name is "%s (copy)".

**Placeholders.** `%s` — the original's name.

**Applied at.** Duplication. A caller that supplies a name suppresses the rule.

### SD-013

**Subject.** Spreadsheet Dashboard.

**Fires when.** A dashboard is duplicated.

**Effect.** `main_data_model_ids` is not copied; the copy starts with no measured models and therefore never shows a sample.

**Message.** None.

## 5. Access and visibility

### SD-014

**Subject.** Every entity of the domain.

**Effect.** The access matrix below is the whole of the entity-level permission system of this domain. A right that is absent is refused by the platform with its own access refusal, naming the entity and the operation.

| Entity | Group | Read | Create | Update | Delete |
|---|---|---|---|---|---|
| `spreadsheet.dashboard` | internal user | yes | no | no | no |
| `spreadsheet.dashboard` | dashboard administrator | yes | yes | yes | yes |
| `spreadsheet.dashboard.group` | internal user | yes | no | no | no |
| `spreadsheet.dashboard.group` | dashboard administrator | yes | yes | yes | yes |
| `spreadsheet.dashboard.share` | internal user | yes | yes | yes | yes |
| `board.board` | internal user | yes | no | no | no |

`spreadsheet.mixin` has no access matrix of its own: it is abstract and every check falls on the host entity.

### SD-015

**Subject.** Spreadsheet Dashboard, for readers holding the internal-user group.

**Effect.** A reader sees a dashboard only when at least one of the dashboard's `group_ids` is among the reader's own groups, direct or implied.

**Message.** None; a dashboard that does not match is simply absent from every read.

### SD-016

**Subject.** Spreadsheet Dashboard, for every reader including administrators of other kinds.

**Effect.** A reader sees a dashboard only when at least one of the dashboard's `company_ids` is among the reader's currently active companies, or when the dashboard has no company at all.

**Message.** None.

### SD-017

**Subject.** Spreadsheet Dashboard, for readers holding the dashboard administrator group.

**Effect.** Every dashboard matches, whatever its audience. The rule is a widening of SD-015 for that group; SD-016 still applies, because it is declared for every group.

**Message.** None.

### SD-018

**Subject.** Dashboard Share, for readers holding the internal-user group.

**Effect.** A user sees only the shares they created themselves. Reading another user's share — including reading its `access_token` — is refused by the platform's access refusal.

**Message.** None from this domain.

### SD-019

**Subject.** The favourite mark.

**Effect.** The favourite operation writes `favorite_user_ids` with elevated rights, because an ordinary reader has no write right on a dashboard. The elevation covers exactly one association row naming the acting reader.

**Message.** None.

### SD-020

**Subject.** The favourite operation.

**Fires when.** The operation is called on a set that does not hold exactly one dashboard.

**Message.** The platform's own single-record refusal.

**Applied at.** Every call.

## 6. The shared address

### SD-021

**Subject.** The token part of the access check.

**Effect.** The supplied token is compared with the stored token by a comparison whose duration does not depend on the number of leading characters that match. An absent or empty supplied token fails without any comparison.

**Why.** A comparison that stops at the first differing character leaks the stored token one character at a time to a caller who measures response times. The address is the only secret protecting a shared dashboard, so the comparison has to be constant-time.

### SD-022

**Subject.** Every route that serves a shared dashboard: the page, the data and the download.

**Fires when.** The token part fails, or the sharing user may no longer read the dashboard.

**Message.** "You don't have access to this dashboard. " — reproduced with its trailing space.

**Placeholders.** None.

**Applied at.** The page route, the data route and the download route, each before doing anything else. The caller receives a forbidden answer.

### SD-023

**Subject.** The download route of a shared dashboard.

**Fires when.** The requesting user does not hold the export group.

**Message.** "You don't have the rights to export data. Please contact an Administrator."

**Placeholders.** None.

**Applied at.** After the access check of SD-022 has passed.

### SD-024

**Subject.** The download route of a shared dashboard.

**Effect.** The route requires a signed-in user. An anonymous caller is sent to sign in rather than refused, because the export group cannot be evaluated for a caller with no identity.

**Message.** None.

### SD-025

**Subject.** The page route and the data route of a shared dashboard.

**Fires when.** No Dashboard Share carries the given identifier.

**Message.** The platform's own not-found answer.

**Note.** The download route does not make this check separately: a missing share yields an empty set whose access check fails, so the caller is refused by SD-022 rather than told that nothing is there. That is the safer of the two answers and is recorded here as deliberate.

## 7. The dashboard reading route

### SD-026

**Subject.** The dashboard reading route.

**Fires when.** The route is called by a caller with no identity, or for a dashboard identifier that resolves to nothing under the caller's own rights — which includes a dashboard excluded by SD-015 or SD-016.

**Message.** The platform's own sign-in redirection for the first case, and its not-found answer for the second.

### SD-027

**Subject.** The dashboard reading route.

**Effect.** The active companies for the whole of the route's work are read from the companies cookie of the request, as identifiers separated by hyphens. A request carrying no such cookie uses the reader's own company alone.

**Message.** None.

### SD-028

**Subject.** The dashboard reading route, live branch.

**Effect.** The reader's locale replaces whatever locale the workbook was stored with, under `settings` and `locale`. A workbook with no settings section gains one.

**Message.** None.

### SD-029

**Subject.** The dashboard reading route.

**Effect.** The sample workbook is served in place of the real one exactly when all three conditions of [`state-machines.md`](state-machines.md) §5.2 hold. A dashboard with no measured models is never considered empty; a measured model the reader may not read is counted with elevated rights rather than refused.

**Message.** None.

## 8. Evaluating a cell

### SD-030

**Subject.** The two list formulas.

**Fires when.** The element identifier given as the first argument names no list element of this workbook.

**Message.** `There is no list with id "%s"`.

**Placeholders.** `%s` — the identifier as it was given.

### SD-031

**Subject.** The two list formulas.

**Fires when.** The field-name argument evaluates to an empty text.

**Message.** "The field name should not be empty."

**Placeholders.** None.

### SD-032

**Subject.** The two list formulas.

**Fires when.** The field path is not a field of the element's entity, or the reader may not read it. The two cases are not distinguished, deliberately: telling a reader that a field exists but is closed to them is itself a disclosure.

**Message.** "The field %s does not exist or you do not have access to that field"

**Placeholders.** `%s` — the field path as it was given.

### SD-033

**Subject.** A list cell reading a field whose type has no cell representation.

**Fires when.** The field is a free-form structured value.

**Message.** `Fields of type "%s" are not supported`

**Placeholders.** `%s` — the name of the field type, which is `json` for the only type currently in this case.

### SD-034

**Subject.** Any data source.

**Fires when.** The entity the element names is not installed.

**Message.** `The model "%(model)s" does not exist.`

**Placeholders.** `%(model)s` — the entity's transport name.

### SD-035

**Subject.** A pivot formula.

**Fires when.** A dimension argument, or a measure argument, names a field the pivot's entity does not have.

**Message.** "Field %s does not exist"

**Placeholders.** `%s` — the field name, or the measure identifier.

### SD-036

**Subject.** A pivot formula addressing a weekly grouping.

**Fires when.** The week argument is not a text of the form week number, a slash, and a four-digit year.

**Message.** "Week value must be a string in the format %(example)s, but received %(received_value)s instead."

**Placeholders.** `%(example)s` — a correct example built from the current year, of the form `"52/<current year>"`; `%(received_value)s` — the value that was given.

### SD-037

**Subject.** The three filter formulas.

**Fires when.** No filter of the workbook carries the given label, after both the given label and each filter's label have been passed through the workbook's translation.

**Message.** `Filter "%(filter_name)s" not found`

**Placeholders.** `%(filter_name)s` — the label as it was given.

### SD-038

**Subject.** The currency rate formula.

**Fires when.** The rate lookup answers nothing — because a code is empty, because a code names no currency, or because no rate can be derived.

**Message.** "Currency rate unavailable."

**Placeholders.** None.

### SD-039

**Subject.** The number format derived from a company's currency.

**Fires when.** The company identifier given names no company.

**Message.** "Currency not available for this company."

**Placeholders.** None.

## 9. Commands that change a workbook

### SD-040

**Subject.** Adding or editing a global filter.

**Fires when.** The filter's label is empty.

**Refusal marker.** `InvalidFilterLabel`. A client turns the marker into its own text.

### SD-041

**Subject.** Adding or editing a global filter.

**Fires when.** Another filter of the same workbook already carries the same label.

**Refusal marker.** `DuplicatedFilterLabel`.

**Why.** A formula addresses a filter by label, so two filters with one label would make a formula ambiguous.

### SD-042

**Subject.** Adding or editing a global filter, and setting a filter's value.

**Fires when.** The default value, or the value, does not match the filter's kind and the operator it carries, by the tables of [`global-filters.md`](global-filters.md) §3 and §4.

**Refusal marker.** `InvalidValueTypeCombination`.

### SD-043

**Subject.** Moving a global filter.

**Fires when.** The requested position is before the first or after the last filter, or the filter does not exist.

**Refusal markers.** `InvalidFilterMove` for the position; `FilterNotFound` for the filter.

### SD-044

**Subject.** Setting a filter's value.

**Fires when.** The new value equals the value currently in force.

**Refusal marker.** `NoChanges`. This is not an error: it prevents a pointless reload of every element matched to that filter.

### SD-045

**Subject.** Recording the field matchings of a data source.

**Fires when.** A matching carries a non-zero period offset but no field path or no field type.

**Refusal marker.** `InvalidFieldMatch`.

### SD-046

**Subject.** The commands that create, duplicate, rename and change list elements.

**Fires when.** One of four conditions:

| Condition | Marker |
|---|---|
| The identifier offered for a new element is not the identifier the workbook expects next | `InvalidNextId` |
| The identifier offered for a new element is already in use | `ListIdDuplicated` |
| A command names an element identifier that does not exist | `ListIdNotFound` |
| A rename supplies an empty name | `EmptyName` |

## 10. The accounting formulas

### SD-047

**Subject.** The period argument of the six accounting formulas.

**Fires when.** The argument cannot be read as a quarter, a month, a year or a day, by the precedence of [`calculations.md`](calculations.md) §10.

**Message.** `'%s' is not a valid period. Supported formats are "21/12/2022", "Q1/2022", "12/2022", and "2022".`

**Placeholders.** `%s` — the value that was given. The three example formats inside the message are part of the text and are reproduced with it.

### SD-048

**Subject.** The six accounting formulas, after the year offset has been applied.

**Fires when.** The resulting year is below 1900.

**Message.** "%s is not a valid year."

**Placeholders.** `%s` — the resulting year.

**Why.** Workbook date numbering starts at the thirtieth of December 1899, so a period before 1900 cannot be represented, and a year at or below one makes the server-side date arithmetic fail. The check is made immediately before the call so that an offset applied to a valid year is checked too.

### SD-049

**Subject.** The two fiscal-year formulas.

**Fires when.** The fiscal-date lookup answers nothing for the requested company — which happens when the company identifier names no company.

**Message.** "The company fiscal year could not be found."

**Placeholders.** None.

### SD-050

**Subject.** The residual formula.

**Fires when.** The residual lookup answers nothing.

**Message.** "The residual amount for given accounts could not be computed."

**Placeholders.** None.

### SD-051

**Subject.** The partner balance formula.

**Fires when.** The partner balance lookup answers nothing.

**Message.** "The balance for given partners could not be computed."

**Placeholders.** None.

### SD-052

**Subject.** The tagged balance formula.

**Fires when.** The tagged balance lookup answers nothing.

**Message.** "The balance for given account tag could not be computed."

**Placeholders.** None.

### SD-053

**Subject.** The partner balance operation.

**Effect.** A request whose partner list, once empty entries are dropped, is empty answers zero without consulting the ledger at all. It does **not** fall back to every partner.

**Message.** None.

### SD-054

**Subject.** The tagged balance operation.

**Effect.** A request whose tag list, once empty entries are dropped, is empty answers zero without consulting the ledger.

**Message.** None.

### SD-055

**Subject.** The account-code argument of the accounting formulas.

**Effect.** Empty entries are dropped from the code list. What an empty list then means depends on the formula:

| Formula family | Empty code list means |
|---|---|
| Total debit, total credit, total balance, tagged balance | no account at all, so the answer is zero |
| Residual amount, partner balance, and the cell audit action | every payable and every receivable account of the company |

**Message.** None.

## 11. The frozen workbook and the export log

### SD-056

**Subject.** A workbook marked as frozen — that is, the public page of a shared dashboard.

**Fires when.** A copy is attempted.

**Refusal marker.** `Readonly`. The copy entries of the cell menu, the column menu, the row menu and the edit menu are disabled as well, so the refusal is normally unreachable through the presentation.

### SD-057

**Subject.** The export log.

**Effect.** A copy is logged only when the copied region covers more than four hundred cells. The size is the sum, over every selected rectangle, of its width in columns multiplied by its height in rows.

```formula
copied cells = sum over each selected rectangle of ( (last column − first column + 1) × (last row − first row + 1) )
```

A copy of the rectangle from the first column and first row to the twentieth column and twentieth row covers 20 × 20 = 400 cells and is **not** logged; one more column makes 21 × 20 = 420 and is logged.

### SD-058

**Subject.** The logging route.

**Effect.** Only four operation names are accepted, reproduced exactly: `download`, `copy`, `freeze`, `print`. A request naming anything else writes nothing and answers normally.

### SD-059

**Subject.** The logging route.

**Effect.** Each data-source description is rendered as one line, and a description that renders to nothing is dropped. A description is dropped when its entity is missing or not installed, or when its field list is empty. The line is built as:

```formula
line = "model: " + entity transport name + " with fields: [" + field names joined by commas + "]"
      + ( " grouped by [" + grouping names joined by commas + "]" when there is at least one grouping )
      + ( " with domain " + the record selection when there is one )
```

The whole entry written to the application log is:

```formula
entry = "User " + acting user identifier + " exported (" + operation name + ") spreadsheet data ("
      + the rendered lines joined by "), (" + ") from " + the network address of the request
```

A request whose descriptions all dropped writes no entry.

### SD-060

**Subject.** A data-bound chart linked to a menu.

**Fires when.** The linked menu resolves but opens no action.

**Message.** "The menu linked to this chart doesn't have an corresponding action. Please link the chart to another menu." — reproduced with its grammatical slip.

**Placeholders.** None.

**Compatibility finding.** The article in the message is wrong. A corrected behaviour would read "does not have a corresponding action". The text is reproduced as observed because support procedures and automated checks key on it.

## 12. The personal board

### SD-061

**Subject.** Dashboard Board.

**Effect.** The creation operation writes nothing and answers with an empty set. A client that initialises a placeholder record when it opens the board therefore stores nothing.

**Message.** None.

### SD-062

**Subject.** The board layout.

**Effect.** Reading the board layout looks for a Custom View record owned by the calling user and referring to the requested view, taking one. Writing a board layout creates such a record owned by the calling user. Both are done with elevated rights, because an ordinary user may neither read nor write views; the record's owner field confines the effect to that user.

**Message.** None.

### SD-063

**Subject.** Every board layout returned, shipped or customised.

**Effect.** Every pinned action marked invisible is removed, at any depth, before the layout leaves the server; and the root of the layout is marked so that the client instantiates the board presentation rather than an ordinary form.

**Message.** None.

### SD-064

**Subject.** The operation that pins an action onto the board.

**Effect.** The operation answers negatively, and stores nothing, unless all four conditions hold: the board's shipped action exists; it addresses the board entity; its first presentation is a form; and an action identifier was supplied. It also answers negatively when the layout has no column to insert into.

**Message.** None; the caller receives a negative answer.

### SD-065

**Subject.** The context stored with a pinned action.

**Effect.** The active-companies key is removed from the context before it is stored.

**Why.** A pinned element that remembered the companies active at pinning time would keep showing those companies' records for ever, and the company selector at the top of the page would appear to do nothing. Dropping the key lets the element follow the reader's current selection.

## 13. Downloading a workbook from the application

### SD-066

**Subject.** The client action that downloads a workbook file.

**Fires when.** The acting user does not hold the export group.

**Message.** "You don't have the rights to export data. Please contact an Administrator.", shown as a notice of the failure kind with the title "Access Error".

**Placeholders.** None.

**Applied at.** Before the workbook is built, so that a user without the right never pays for the export.

## 14. Index of rule identifiers

| Rule | Subject | Kind |
|---|---|---|
| [SD-001](#sd-001) | Stored workbook must decode | validation with message |
| [SD-002](#sd-002) | Stored workbook references must exist | validation with message |
| [SD-003](#sd-003) | Packaged workbook files skip the reference walk | exemption |
| [SD-004](#sd-004) | Dashboard name required | required value |
| [SD-005](#sd-005) | Dashboard group required on a dashboard | required value |
| [SD-006](#sd-006) | Dashboard group name required | required value |
| [SD-007](#sd-007) | Share must name a dashboard | required value |
| [SD-008](#sd-008) | Share must carry a token | required value |
| [SD-009](#sd-009) | A shipped dashboard group cannot be deleted | deletion guard with message |
| [SD-010](#sd-010) | A dashboard group holding dashboards cannot be deleted | storage constraint |
| [SD-011](#sd-011) | Deleting a dashboard deletes its shares | cascade |
| [SD-012](#sd-012) | Naming of a duplicated dashboard | naming rule |
| [SD-013](#sd-013) | A duplicate carries no measured models | duplication rule |
| [SD-014](#sd-014) | Entity access matrix | permission |
| [SD-015](#sd-015) | Audience selection for internal users | record rule |
| [SD-016](#sd-016) | Company selection for every reader | record rule |
| [SD-017](#sd-017) | Dashboard administrators see every dashboard | record rule |
| [SD-018](#sd-018) | A share is visible only to its creator | record rule |
| [SD-019](#sd-019) | The favourite mark is written with elevated rights | permission |
| [SD-020](#sd-020) | The favourite operation addresses one record | precondition |
| [SD-021](#sd-021) | Token comparison is constant-time | security rule |
| [SD-022](#sd-022) | Shared-dashboard access refusal | validation with message |
| [SD-023](#sd-023) | Export right for the shared download | validation with message |
| [SD-024](#sd-024) | The shared download requires a signed-in user | permission |
| [SD-025](#sd-025) | Unknown share on the page and data routes | not found |
| [SD-026](#sd-026) | The reading route requires an identity and a readable dashboard | permission |
| [SD-027](#sd-027) | Active companies come from the request | reading rule |
| [SD-028](#sd-028) | The reader's locale replaces the stored one | reading rule |
| [SD-029](#sd-029) | When a sample replaces the real workbook | reading rule |
| [SD-030](#sd-030) | A list formula must name an existing element | validation with message |
| [SD-031](#sd-031) | A list formula's field name must not be empty | validation with message |
| [SD-032](#sd-032) | A list formula's field must exist and be readable | validation with message |
| [SD-033](#sd-033) | Unsupported field type in a list cell | validation with message |
| [SD-034](#sd-034) | A data source's entity must exist | validation with message |
| [SD-035](#sd-035) | A pivot dimension or measure must exist | validation with message |
| [SD-036](#sd-036) | Weekly pivot values have a fixed shape | validation with message |
| [SD-037](#sd-037) | A filter formula must name an existing filter | validation with message |
| [SD-038](#sd-038) | Currency rate unavailable | validation with message |
| [SD-039](#sd-039) | Company currency unavailable | validation with message |
| [SD-040](#sd-040) | A filter must have a label | command refusal |
| [SD-041](#sd-041) | Filter labels are unique within a workbook | command refusal |
| [SD-042](#sd-042) | A filter value must match its kind and operator | command refusal |
| [SD-043](#sd-043) | A filter move must stay within the list | command refusal |
| [SD-044](#sd-044) | Setting the value already in force changes nothing | command refusal |
| [SD-045](#sd-045) | A period offset requires a field path and a type | command refusal |
| [SD-046](#sd-046) | Identifier and name rules for list elements | command refusal |
| [SD-047](#sd-047) | Accounting period format | validation with message |
| [SD-048](#sd-048) | Accounting year lower bound | validation with message |
| [SD-049](#sd-049) | Fiscal year unavailable | validation with message |
| [SD-050](#sd-050) | Residual amount unavailable | validation with message |
| [SD-051](#sd-051) | Partner balance unavailable | validation with message |
| [SD-052](#sd-052) | Tagged balance unavailable | validation with message |
| [SD-053](#sd-053) | An empty partner list answers zero | computation rule |
| [SD-054](#sd-054) | An empty tag list answers zero | computation rule |
| [SD-055](#sd-055) | What an empty account-code list means | computation rule |
| [SD-056](#sd-056) | Copying is refused in a frozen workbook | command refusal |
| [SD-057](#sd-057) | The copy logging threshold | logging rule |
| [SD-058](#sd-058) | Accepted log operation names | logging rule |
| [SD-059](#sd-059) | How a log entry is rendered | logging rule |
| [SD-060](#sd-060) | A chart's linked menu must open an action | validation with message |
| [SD-061](#sd-061) | The board creates no record | lifecycle rule |
| [SD-062](#sd-062) | A board layout belongs to one user | permission |
| [SD-063](#sd-063) | Invisible pinned actions are removed | presentation rule |
| [SD-064](#sd-064) | Preconditions for pinning an action | precondition |
| [SD-065](#sd-065) | Active companies are not stored with a pinned action | storage rule |
| [SD-066](#sd-066) | Export right for the in-application download | validation with message |

## 15. Locking and concurrency

The domain declares no lock of any kind.

| Situation | Resolution |
|---|---|
| Two administrators write the same dashboard | Last write wins, on the whole record including the whole workbook. There is no merge of two workbooks |
| Two readers set the same filter | Each reader has their own reading session; filter values are never shared and never stored |
| A reader stars a dashboard while an administrator writes it | Independent: the star is one association row, the write touches columns |
| A share is read while the dashboard it copies is edited | Independent: the share holds its own frozen copy and never re-reads the dashboard's workbook |
| A share is read while the sharing user's rights change | The next read applies the new rights; there is no cached decision |
| Two data sources of one workbook load at once | Each keeps only its most recent load; an earlier load still in flight is abandoned |
