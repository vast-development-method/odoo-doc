# The service layer

Every entity of the system exposes the same set of generic operations, and every capability package adds business operations on top of them following a small number of naming conventions. Together they form the service layer: the complete surface that a client, a script or an integration invokes, whatever the transport it uses to reach the server ([`remote-transport-contracts.md`](remote-transport-contracts.md)). This document specifies that surface: the contract of every generic operation, the conventions and patterns of business operations, the read specification grammar that shapes what a read returns, the on-change protocol, the export contract, the discovery operations that let a caller learn which entities, fields and operations exist in a given deployment, and the complete enumeration of the named business operations that the screens of the shipped installation bind to a control, grouped by the part of the system that owns the entity.

## 1. Model of an operation

### 1.1 Binding

Every operation is invoked on an entity, with a record set. Two bindings exist:

| Binding | Record set | Called with |
|---|---|---|
| Entity-level | Always empty; the operation concerns the entity, not particular records | No record keys. |
| Record-level | The records the operation acts on | The record keys as the first positional argument, or as the `ids` member on the direct transport. |

An entity-level operation called with record keys is a caller error and is refused (`"cannot call <entity>.<operation> with ids"`). A record-level operation may legitimately be called with an empty record set; several of them then act on "the entity in general" (checking a permission, creating a record, computing defaults).

### 1.2 Marks

| Mark | Meaning | Observable consequence |
|---|---|---|
| Entity-level | The operation does not act on specific records. | No record keys are consumed from the arguments. |
| Read-only | The operation promises to perform no write. | The request may be served on a read-only database connection; a write attempt causes a transparent replay on a read/write connection. |
| Private | The operation may not be invoked remotely. | Any remote call is refused with the private-operation access failure, even when the operation name has no leading underscore. |
| Deprecated | The operation is superseded. | It is omitted from the discovery documents and must not be used; this specification does not describe deprecated operations at all. |

An operation whose name starts with an underscore is internal by construction and is never remotely reachable.

### 1.3 The context

Every operation runs with a context (see [`remote-transport-contracts.md`](remote-transport-contracts.md), section 11). The context never grants access and never changes which records exist; it changes defaults, language, time zone, company scope, formatting and a small number of documented behaviors, each listed where it applies.

## 2. The generic service contract

The tables below are the complete contract of the operations that exist on **every** entity. Column meanings: *Binding* is `entity` or `record`; *Read-only* says whether the operation is served on a read-only connection; *Remote* says whether the operation can be invoked remotely.

### 2.1 Reading

| Operation | Binding | Read-only | Remote | Inputs | Output |
|---|---|---|---|---|---|
| `search` | entity | yes | yes | `domain`, `offset` (default 0), `limit` (default none), `order` (default the entity's ordering) | The array of matching record keys, in order. |
| `search_count` | entity | yes | yes | `domain`, `limit` (default none) | The number of matching records; when a limit is given, the count is capped at it. |
| `search_read` | entity | yes | yes | `domain` (default all), `fields` (default every readable field), `offset`, `limit`, `order`, plus the read options | The array of record payloads, one per record, in search order. Performs the search and the read in **one** transaction, therefore a record deleted concurrently cannot make it fail. |
| `read` | record | yes | yes | `fields` (default every readable field), `load` (default the display-name mode, the other mode being "keys only") | One payload per record that still exists, in record-set order. Records that vanished are silently omitted. |
| `name_search` | entity | yes | yes | `name` (default empty), `domain` (default none), `operator` (default `ilike`), `limit` (default 100) | The array of `[key, display name]` pairs of the records whose display name matches, further restricted by the filter. It is the reverse of the display-name rule, and the display name is computed with elevated rights. |
| `fields_get` | entity | no | yes | `allfields` (default all), `attributes` (default all) | For each readable field, the mapping of its attributes (section 9.1). |
| `default_get` | entity | no | yes | `fields` | The mapping of default values for those of the requested fields that have one (section 2.5). |
| `get_metadata` | record | no | yes | none | One entry per record with `id`, `create_uid`, `create_date`, `write_uid`, `write_date`, `xmlid` (the newest external name or false), `xmlids` (every external name with its protection flag, newest first) and `noupdate`. An entity that does not record creation and modification metadata answers only the key together with the three external-name members. |
| `get_external_id` | record | no | yes | none | A mapping from record key to its external name, the empty text when it has none. |
| `get_base_url` | record | no | yes | none | The public base address for the record; refuses more than one record. |
| `has_access` | record | no | yes | `operation` (`read`, `write`, `create`, `unlink`) | True when the caller may perform that operation on **all** the records; on an empty record set, whether the caller has any permission at all for it on the entity. |
| `get_property_definition` | entity | no | yes | `full_name` (`<properties field>.<property name>`) | The definition of one dynamic property: its name, label, type, target entity and allowed values. |
| `get_field_translations` | record | no | yes | `field_name`, `langs` (default every installed language) | The pair of: one entry per language with the source text and the translated text, and the field's own descriptor. |
| `get_views` | entity | yes | yes | `views`, `options` | The presentations and the field descriptions ([`remote-transport-contracts.md`](remote-transport-contracts.md), section 9.5). |
| `get_view` | entity | no | yes | `view_id`, `view_type`, options | One presentation description. |
| `get_formview_id` | record | yes | yes | `access_uid` | The key of the form presentation to use for this record, or false for the default one. |
| `get_formview_action` | record | yes | yes | `access_uid` | A window action opening this record in a form presentation, carrying the current context. |
| `formatted_read_group` | entity | yes | yes | `domain`, `groupby`, `aggregates`, `having`, `offset`, `limit`, `order` | The array of group payloads (section 7.6). |
| `formatted_read_grouping_sets` | entity | yes | yes | `domain`, `grouping_sets`, `aggregates`, `order` | One array of group payloads per grouping set, computed in a single pass. |
| `read_progress_bar` | entity | yes | yes | `domain`, `group_by`, `progress_bar` (`{field, colors, sum}`) | A mapping from group value, rendered as text, to a mapping from the coloured field value to the number of records. Values outside the declared colours are ignored, and every declared colour is present with a count of zero when empty. |
| `search_panel_select_range` | entity | no | yes | `field_name` plus the options of section 2.7 | `{"parent_field": <field name or false>, "values": [{"id", "display_name", "__count"?, <parent field>?}]}`, or `{"error_msg": "Too many items to display."}` when the limit is reached. |
| `search_panel_select_multi_range` | entity | no | yes | `field_name` plus the options of section 2.7 | `{"values": [{"id", "name", "__count", "group_id"?, "group_name"?}]}`, or the same error object. |
| `export_data` | record | no | yes | `fields_to_export` | `{"datas": <matrix of exported cells>}` (section 8). |

### 2.2 Writing

| Operation | Binding | Read-only | Remote | Inputs | Output and effects |
|---|---|---|---|---|---|
| `create` | entity | no | yes | One values mapping, or an array of them | The created records. Remotely: one key for a single mapping, an array of keys for an array. Defaults are applied for every field absent from the mapping; computed stored fields are computed; constraints and validations run before the call returns. |
| `write` | record | no | yes | One values mapping | True. Applied to every record of the set with the same values. |
| `unlink` | record | no | yes | none | True. Deletes the records, after the deletion guards of the entity. |
| `copy` | record | no | yes | `default` (a values mapping overriding the copy) | The new records, one per source record, created in one call. Translations of translated fields are copied too, except for fields named in `default`. |
| `copy_data` | record | no | yes | `default` | The array of values mappings that `copy` would use, without creating anything. Excluded from the copy: the technical columns, the hierarchy path, fields marked as not copied, and the fields inherited from a delegated parent that the caller overrode. |
| `copy_translations` | record | no | yes | `new` (the target records), `excluded` | Nothing. Copies the per-language texts from the source to the target. |
| `name_create` | entity | no | yes | `name` | The pair `[key, display name]` of a record created with only its naming field set. When the entity has no naming field, it answers false and logs `"Cannot execute name_create, no _rec_name defined on <entity>"`. |
| `action_archive` | record | no | yes | none | Sets the archive flag to false on the records that are still active. Entities without an archive flag refuse the call. |
| `action_unarchive` | record | no | yes | none | Sets the archive flag to true on the records that are archived. |
| `update_field_translations` | record | no | yes | `field_name`, `translations` (a mapping from language to text, or to a mapping of source text to translated text) | True. Replaces the per-language texts of one field. |
| `web_override_translations` | record | no | yes | `values` (a mapping from field name to text) | Nothing. For each field that is translated as a whole, clears every language and sets the given text for the reference language and for the caller's language. |
| `load` | entity | no | yes | `fields` (the field paths, one per column), `data` (the matrix of raw cells) | `{"ids": <array of keys or false>, "messages": [...], "nextrow": <integer>}` (section 8.5). |
| `onchange` | record | no | yes | `values`, `field_names`, `fields_spec` | The differences to apply and the warnings (section 7.8). |
| `onchange_batch` | entity | no | yes | `values_list`, `field_names`, `fields_spec` | One on-change result per entry, for new records only; refuses a non-empty record set. |

### 2.3 The client-facing composites

These operations exist to serve a whole screen in one round trip. They are the ones the desktop client uses; an integration may use them too.

| Operation | Binding | Read-only | Inputs | Output |
|---|---|---|---|---|
| `web_read` | record | yes | `specification` | One payload per record, shaped by the read specification (section 7). With a specification that contains only the record key, no read is performed at all and the payloads are built from the keys. |
| `web_search_read` | entity | yes | `domain`, `specification`, `offset`, `limit`, `order`, `count_limit` | `{"length": <integer>, "records": [<payload>]}` with the length rule of section 7.7. |
| `web_save` | record | no | `vals`, `specification`, `next_id` | Writes `vals` when the record set is non-empty, otherwise creates one record with them; then, when `next_id` is given, rebinds to that record; then returns its payload, read with the flag that makes binary fields answer their size instead of their content. |
| `web_save_multi` | record | no | `vals_list`, `specification` | Writes the matching entry of `vals_list` to each record, pairwise. A length mismatch fails with `"Each record must have a corresponding vals entry."`. Returns one payload per record. |
| `web_read_group` | entity | yes | `domain`, `groupby`, `aggregates`, `limit`, `offset`, `order`, `auto_unfold`, `opening_info`, `unfold_read_specification`, `unfold_read_default_limit`, `groupby_read_specification` | `{"groups": [...], "length": <integer>}` (section 7.6). |
| `web_name_search` | entity | yes | `name`, `specification`, `domain`, `operator`, `limit` | When the specification asks for the display name only, one payload per match with `id`, `display_name` and `__formatted_display_name` (the display name computed with the rich formatting flag). Otherwise the result of `web_read` with that specification over the matches. |
| `web_resequence` | record | no | `specification`, `field_name` (default `sequence`), `offset` (default 0) | Writes `offset`, `offset + 1`, … to the given field of the records in record-set order, then returns their payloads. When the entity has no such field, answers an empty array and writes nothing. |

### 2.4 Operations that are not remotely callable

The following generic operations exist but are explicitly marked private; a remote call is refused with `"Private methods (such as '<entity>.<operation>') cannot be called remotely."`:

`search_fetch`, `fetch`, `check_access`, `exists`, `lock_for_update`, `try_lock_for_update`, `browse`, `ensure_one`, `with_env`, `sudo`, `with_user`, `with_company`, `with_context`, `with_prefetch`, `mapped`, `filtered`, `filtered_domain`, `grouped`, `sorted`, `update`, `flush_model`, `flush_recordset`, `new`, `concat`, `union`, `invalidate_model`, `invalidate_recordset`, `modified`, `init`.

They are either in-memory manipulations with no meaning across a transport, or privilege-changing helpers that must never be reachable from outside (`sudo`, `with_user`), or cache and locking primitives.

The remote equivalent of the privilege helpers is: there is none. A caller acts as the user its credential designates, always.

### 2.5 How default values are determined

`default_get`, and therefore every creation from a form, resolves each requested field in this order and stops at the first hit:

1. A context entry named `default_<field>`.
2. For a field that is not company-dependent: a stored user default for the entity and field, matching the caller and the active company, and matching the optional condition.
3. The field's own declared default.
4. For a company-dependent field: the stored fallback value.
5. For a field inherited from a delegated parent that the caller may write: the parent entity's own default for it.

Then every resolved value is normalized to the writing format. For a relational field this means passing through the in-memory representation, which turns a list of link commands into a single replace command, because that is the only shape a client can apply to a fresh form.

Access is enforced while normalizing: a delete command on a related record requires deletion access on it, and an update command, or a link or unlink command on a one-to-many, requires write access on it.

Worked example. A window action carries the context `{"default_partner_id": 14, "default_line_ids": [[4, 8, 0], [4, 9, 0]]}` and the entity declares `date` defaulting to today and `state` defaulting to `draft`. A client asking for the defaults of `partner_id`, `line_ids`, `date`, `state` and `note` receives:

```
{"partner_id": 14,
 "line_ids": [[6, 0, [8, 9]]],
 "date": "2026-09-11",
 "state": "draft"}
```

`note` is absent because it has no default at any level.

### 2.6 Values accepted when writing

| Field type | Accepted value | Notes |
|---|---|---|
| `boolean` | true or false | |
| `integer` | integer | |
| `decimal` | number | Rounded to the field's precision on write. |
| `monetary` | number | Rounded to the precision of the record's currency field. |
| `text`, `long_text`, `rich_text` | text or false | False and the empty text are distinct for a nullable field. |
| `date` | the text `YYYY-MM-DD`, or false | |
| `datetime` | the text `YYYY-MM-DD HH:MM:SS` in coordinated universal time, or false | Never a local time; the display time zone is a presentation concern. |
| `binary`, `image` | the content encoded for transport, or false | |
| `selection` | one of the declared values, or false | An undeclared value is refused. |
| `reference` | the text `<entity>,<key>`, or false | |
| `many_to_one` | a record key, or false | |
| `one_to_many`, `many_to_many` | an array of relational commands (section 3) | A bare array of keys is not accepted on write. |
| `structured_data` | a nested object | |
| `properties` | an array of property entries, each carrying its definition and its value | |

### 2.7 The search-panel options

Both search-panel operations accept: `search_domain` (the base filter), `category_domain` (the filter contributed by the already selected categories), `filter_domain` (the filter contributed by the already selected filters), `comodel_domain` (a restriction on the candidate values), `enable_counters` (default false), `expand` (default false: return only the values actually present, rather than the whole range), `limit` (default none), and, for the range operation, `hierarchize` (default true, which adds the parent field when the target entity has one) and, for the multi-range operation, `group_by` and `group_domain`.

Rules: only `many_to_one` and `selection` fields are accepted by the range operation, otherwise `"Only types %(supported_types)s are supported for category (found type %(field_type)s)"`; the multi-range operation also accepts `many_to_many`, otherwise `"Only types %(supported_types)s are supported for filter (found type %(field_type)s)"`. When a limit is given and the number of values reaches it, no values are returned at all and the answer is the single member `error_msg` holding `"Too many items to display."`, because a truncated facet list would silently mislead the user. When hierarchization is on, the returned set is closed under the parent relation, and any value whose ancestor chain is not fully included is dropped.

## 3. Relational commands

Writing a one-to-many or many-to-many field uses a list of commands. Each command is a triple `[code, key, payload]`.

| Code | Name | Key | Payload | Effect |
|---|---|---|---|---|
| `0` | create | `0` | a values mapping | Creates a record in the target entity and links it. On a one-to-many, one new record per record written. On a many-to-many, one new record shared by every record written. |
| `1` | update | the target key | a values mapping | Writes those values on the linked record. |
| `2` | delete | the target key | `0` | Unlinks **and deletes** the target record. On a many-to-many the deletion may be refused when other records still reference it. |
| `3` | unlink | the target key | `0` | Removes the link only. On a one-to-many whose inverse cascades, the target is deleted; otherwise the inverse is emptied and the target is kept. |
| `4` | link | the target key | `0` | Adds a link to an existing record. |
| `5` | clear | `0` | `0` | Removes every link, as if unlink had been applied to each. |
| `6` | replace | `0` | an array of keys | Replaces the whole set of links by exactly those keys. |

A caller must send the literal triples; there is no named form on the wire.

Worked example. A sales order has three lines, with keys 11, 12 and 13. The client wants to change the quantity of line 12, delete line 13, add a new line, and link an existing line 20:

```
{"line_ids": [[1, 12, {"product_uom_qty": 5}],
              [2, 13, 0],
              [0, 0, {"product_id": 42, "product_uom_qty": 2}],
              [4, 20, 0]]}
```

After the write the order has lines 11, 12 (quantity 5), 20 and the newly created one; line 13 no longer exists.

Worked example of the replace command. `{"tag_ids": [[6, 0, [3, 5]]]}` leaves exactly tags 3 and 5 linked, whatever was linked before; no target record is deleted.

## 4. Read formats

The value a caller receives for a field depends on the read mode:

| Field type | Display-name mode (the default) | Keys-only mode |
|---|---|---|
| `many_to_one` | The pair `[key, display name]`, or false. The display name is computed with elevated rights, because the visibility of the *reference* follows the referring record, not the referenced one. A dangling reference answers false. | The key, or false. |
| `one_to_many`, `many_to_many` | The array of target keys, in the relation's order. | Identical. |
| `reference` | The text `<entity>,<key>`, or false. | Identical. |
| `date` | The text `YYYY-MM-DD`, or false. | Identical. |
| `datetime` | The text `YYYY-MM-DD HH:MM:SS` in coordinated universal time, or false. | Identical. |
| `binary`, `image` | The content encoded for transport, or, when the size flag is set on the context, a human-readable size such as `"12.50 Kb"`. | Identical. |
| `properties` | The array of property entries, each with its definition and its value; a relational property value additionally carries its display name. | The same without display names. |
| everything else | The stored value, with the null value normalized to false. | Identical. |

A record that no longer exists produces no payload at all: the array a read returns can be shorter than the array of keys the caller passed, and the caller must match on the `id` member rather than on position.

## 5. Conventions of business operations

Business operations are the operations a capability package adds on top of the generic contract. Their names carry meaning, and a replacement must keep the conventions because clients, automated actions and other packages rely on them.

| Prefix or shape | Meaning | Returns | Invoked from |
|---|---|---|---|
| `action_<verb>` | A user-triggered transition or command, for example `action_confirm`, `action_post`, `action_cancel`, `action_archive`. | Nothing, or an action to run next. | A button on a screen, an automation, an integration. |
| `action_open_<thing>`, `action_view_<thing>` | Navigation: opens a list or a form on related records. | Always a window action. | A statistical button or a menu entry. |
| `<verb>_<thing>` without a prefix | A plain service operation with a business meaning, for example `name_create`, `message_post`. | Whatever it computes. | Anywhere. |
| `_compute_<field>` | Derives one or several fields. | Nothing; it assigns the fields. | The record layer only; never remotely (leading underscore). |
| `_inverse_<field>` | Writes back a derived field. | Nothing. | The record layer only. |
| `_search_<field>` | Turns a condition on a non-stored field into a condition on stored ones. | A filter. | The record layer only. |
| `_onchange_<field>` or a declared on-change handler | Reacts to an edit in a form, before saving. | Nothing, or a warning. | The on-change protocol only (section 7.8). |
| `_check_<rule>` | A validation invoked after every write to the fields it watches. | Nothing; it raises a validation failure with the user-facing message. | The record layer only. |
| `_cron_<job>` | The body of a scheduled job. | Nothing, or a progress report. | The scheduler only. |
| `_<anything>` | Internal. | — | Never remotely. |

Rules that hold for every business operation:

1. **It validates its own preconditions.** An operation never assumes the caller checked the state; it re-checks and raises the business failure with the exact user-facing message.
2. **It is idempotent where the business allows it.** Confirming an already confirmed document either is a no-operation or fails with a message; it never duplicates effects.
3. **It works on a record set.** An operation that is meaningful on a single record only states that requirement explicitly and fails when given several; every other operation iterates.
4. **It performs the whole unit of work.** Because each remote call is its own transaction, anything that must be atomic belongs inside one operation (see [`remote-transport-contracts.md`](remote-transport-contracts.md), section 8.9).

## 6. Patterns

### 6.1 Wizards: transient entities with a launch action

An interaction that needs input before performing something is modelled as a **transient entity**: an entity whose records are ordinary rows that a maintenance job deletes after a while, and which is never referenced by permanent records.

The pattern, end to end:

1. A button or menu entry calls an operation that returns a window action whose target is a dialog, whose entity is the transient entity, whose presentation is a form, and whose context carries the seeds the wizard needs (typically `active_id`, `active_ids`, `active_model`, and any `default_<field>` entry).
2. The client opens the form. The record does not exist yet: the client calls the on-change protocol with an empty record set to obtain the defaults and the derived values.
3. The user fills the form. Each edit triggers the on-change protocol again.
4. The user presses the confirmation button. The client calls the save operation, which creates the transient record, then calls the button operation on it.
5. The button operation reads its own fields and the records named by `active_ids`, performs the work, and returns either nothing (the dialog closes), a close action, a new window action (chaining to another screen), or a notification action.
6. The transient record is left behind and collected later by the maintenance job.

Worked example, a wizard that registers a payment for several invoices: the action carries `{"active_model": "journal_entry", "active_ids": [31, 32]}`; the wizard entity computes the total amount from those invoices as a derived field; the confirmation operation creates the payment, reconciles it and returns `{"type": "ir.actions.act_window", "res_model": "payment", "res_id": 77, "views": [[false, "form"]], "target": "current"}`.

### 6.2 Batch operations

A batch operation is a record-level operation invoked on many records at once from a list screen, through a bound action. Requirements:

1. It must be written to work on the whole set at once: one query per step, not one per record.
2. It must be **partially safe**: either it validates every record up front and fails as a whole, or it is documented as skipping the records that do not qualify. The reference behavior for state transitions is to filter first: `action_confirm` on a set confirms the records in the qualifying state and ignores the others; `action_archive` archives only the currently active ones.
3. When the set is bounded by the active-keys limit, the client passes the filter instead of the keys; a batch operation must therefore also be reachable with a filter, through a wizard that re-runs the search.

### 6.3 Operations that return an action

An operation returns a mapping whose `type` member names the action kind. The client interprets it. The kinds and the members each one needs:

| Kind (the wire value of `type`) | Entity | Required members | Effect |
|---|---|---|---|
| `ir.actions.act_window` | Window Action | `res_model`, and either `views` or `view_mode` | Opens a screen. `res_id` opens one record; `domain` and `context` restrict and seed it; `target` is `current` (replaces the screen), `new` (a dialog), `fullscreen` or `main`. |
| `ir.actions.act_window_close` | Close Window Action | none | Closes the current dialog and refreshes the screen underneath. |
| `ir.actions.act_url` | Web Address Action | `url` | Opens an address; `target` `self` navigates, `new` opens a new view, `download` downloads. |
| `ir.actions.client` | Client Action | `tag` | Runs a named client behavior with `params`. |
| `ir.actions.report` | Report Action | `report_name`, `report_type` | Renders and downloads a document. |
| `ir.actions.server` | Server Action | `id` | Runs a server action, which itself returns one of the above or nothing. |

The values in the first column are wire values: they are what the `type` member literally contains, and clients branch on them. The second column is the entity that stores actions of that kind; its fields are specified in the platform foundation domain.

Returning nothing (or false) means "done, nothing else to do": the client simply refreshes.

### 6.4 Operations that return a notification

The notification pattern is a client action with the tag `display_notification` and these parameters:

| Parameter | Values | Meaning |
|---|---|---|
| `type` | `success`, `warning`, `danger`, `info` | The severity, which selects the colour. |
| `title` | text, optional | The heading. |
| `message` | text | The body. |
| `sticky` | boolean, default false | Whether the notification stays until dismissed. |
| `next` | an action, optional | The action to run after showing the notification, typically the close action. |

Worked example, the result of a bulk assignment operation:

```
{"type": "ir.actions.client",
 "tag": "display_notification",
 "params": {"type": "success",
            "title": "Leads Assigned",
            "message": "3 leads assigned to 2 salespersons.",
            "next": {"type": "ir.actions.act_window_close"}}}
```

A second pattern exists for a message that must reach the user **outside** the request that caused it (a long job, another user's action): the server pushes a notification on the recipient's channel with the same shape, and the client's notification bus displays it ([`remote-transport-contracts.md`](remote-transport-contracts.md), section 9.11).

### 6.5 Celebration effect

A window-close action may carry an `effect` member, whose `type` is `rainbow_man`, with a `message` and an optional `fadeout` (`slow`, `medium`, `fast`, `no`). The client shows a celebratory overlay. It is enabled per user by the celebration switch reported in the session information.

### 6.6 Reload

The client action with the tag `reload` restarts the client entirely. It is the answer of operations that invalidate everything the client holds: changing the user's own password, installing or removing a capability package, switching the active company set.

## 7. The read specification

### 7.1 Why it exists

A screen needs, in one round trip, the fields of the main record, the display names of the records it points to, some fields of the lines it owns, and sometimes fields two levels deep. The read specification is the recursive description of exactly that, and the same structure is used by every client-facing composite operation: `web_read`, `web_search_read`, `web_save`, `web_save_multi`, `web_name_search`, `web_resequence`, the group operations and the on-change protocol.

### 7.2 Grammar

```
specification := { field_name: field_specification, ... }

field_specification := {
    "fields":   specification,   # only for relational fields
    "context":  { key: value },  # only for relational fields
    "limit":    integer,         # only for one-to-many and many-to-many
    "order":    text             # only for one-to-many and many-to-many
}
```

Every member of a field specification is optional; an empty object `{}` is the normal specification of a plain field. The order of the members of `specification` is irrelevant; the order of the returned members is not guaranteed and a caller must address them by name.

### 7.3 Semantics per field type

| Field type | Field specification | Returned value |
|---|---|---|
| any plain type | `{}` | The value in the read format of section 4. |
| `many_to_one` | `{}` | The target key, or false. **Not** the pair with the display name: the composites use the keys-only read mode. |
| `many_to_one` | `{"fields": {...}}` | An object holding the requested fields of the target, always including its `id`, or false when the reference is empty. When `display_name` is among the requested fields, it is computed with elevated rights and added separately, for the reason given in section 4. |
| `many_to_one` | `{"fields": {...}, "context": {...}}` | The same, with the target read under the merged context (for example a different language). |
| `one_to_many`, `many_to_many` | `{}` | The array of target keys. |
| `one_to_many`, `many_to_many` | `{"fields": {...}}` | The array of objects, one per accessible target, each shaped by the nested specification. A target the caller may not read is dropped. |
| `one_to_many`, `many_to_many` | `{"fields": {...}, "limit": n}` | The first `n` targets of each record are read as objects; the remaining keys are still listed, as bare objects carrying only their `id`. |
| `one_to_many`, `many_to_many` | `{"order": "<order expression>"}` | The array of keys, re-sorted by that expression. |
| `reference` | `{"fields": {...}}` | An object shaped by the nested specification, whose `id` member is itself the object `{"id": <key>, "model": <entity>}`, or false when the target no longer exists. |
| `many_to_one_reference` | `{"fields": {...}}` | An object shaped by the nested specification with a plain `id`. When the target no longer exists, both this field and its companion entity-name field are set to false. |
| `properties` | `{"fields": {<property name>: <field specification>}}` | Only the named properties, in the order given, each with its definition and value; a relational property value is expanded with the nested specification when one is given. |

Additional rules, all observable:

- A field name in the specification that does not exist on the entity is ignored, not an error.
- A specification containing only `id` performs **no** read at all: the payloads are built from the keys. This is what makes reading a long list of related keys cheap.
- A payload always carries `id`. For a record that is not saved yet, the `id` member carries the origin key when it has one and false otherwise.
- When the caller has no entity-level read permission on the target entity, a specification carrying `order` yields an **empty** array: the relation is emptied rather than sorted, because sorting would require reading. A specification carrying `fields` is not filtered in that case, and the nested read then fails with the access refusal; a specification carrying neither returns the raw key array, which is what lets a relation point at an entity the caller cannot read while the client still knows how many targets there are.
- When a target the caller may not read has been left in the relation by a previous elevated write, it is filtered out of the key array before the payloads are built.
- Filtering and sorting of a relation are performed with the archived-records filter disabled, therefore an archived line that is still linked is still returned.

### 7.4 Worked example

Specification sent for a sales order form:

```
{"name": {},
 "state": {},
 "partner_id": {"fields": {"display_name": {}, "email": {}}},
 "currency_id": {"fields": {"display_name": {}}},
 "line_ids": {"fields": {"product_id": {"fields": {"display_name": {}}},
                         "product_uom_qty": {},
                         "price_subtotal": {}},
              "limit": 2,
              "order": "sequence asc, id asc"},
 "tag_ids": {}}
```

Answer for one order with three lines (keys 11, 12, 13):

```
[{"id": 5,
  "name": "S00021",
  "state": "draft",
  "partner_id": {"id": 14, "display_name": "Deco Addict", "email": "deco@example.com"},
  "currency_id": {"id": 1, "display_name": "EUR"},
  "line_ids": [{"id": 11, "product_id": {"id": 42, "display_name": "Office Chair"},
                "product_uom_qty": 3.0, "price_subtotal": 360.0},
               {"id": 12, "product_id": {"id": 43, "display_name": "Desk"},
                "product_uom_qty": 1.0, "price_subtotal": 250.0},
               {"id": 13}],
  "tag_ids": [7, 9]}]
```

Line 13 is beyond the limit, therefore only its key is returned; the client fetches it later when the user scrolls.

### 7.5 Deriving a specification from a screen

A client does not invent the specification: it derives it from the screen description returned by the presentation loader, by walking the description and, for each field element, recording the field name, adding `display_name` for a `many_to_one`, and recursing into the nested description of a relational field. For a one-to-many, the inverse field is removed from the nested specification, because it would point back at the record being edited. The result is exactly the set of fields the screen can display, which is what makes the round trip minimal.

### 7.6 Group payloads

`formatted_read_group` answers an array of group objects. Each object carries:

| Member | Content |
|---|---|
| `<groupby specification>` | The group value, formatted per type: for a relational grouping the pair `[key, display name]` (the display name computed with elevated rights) or false; for a date or date-time grouping the pair `[<start of the interval as text>, <label>]`; for a numeric date part the integer; for anything else the raw value. |
| `<aggregate specification>` | The aggregate value. |
| `__extra_domain` | The filter that selects exactly the records of this group, to be combined with the caller's filter. |
| `__count` | The number of records in the group, when requested. |
| `__fold` | Whether the group is folded by default, present only when group expansion is enabled and the grouping field's target entity has a folding flag. |

Grouping specification: a field name, or `<field>:<granularity>` for a date or date-time field, or a dotted path through a `many_to_one` (for example `partner_id.country_id`), or `<properties field>.<property name>`. Supported granularities: `day`, `week`, `month`, `quarter`, `year`, and the numeric parts `year_number`, `quarter_number`, `month_number`, `iso_week_number`, `day_of_year`, `day_of_month`, `day_of_week`, `hour_number`, `minute_number`, `second_number`. A date or date-time grouping without a granularity is refused.

Aggregate specification: `__count`, or `<field>:<function>` where the function is one of the database's aggregate functions plus the two extras `count_distinct` and `array_agg_distinct`; the alias `:recordset` is accepted and behaves as `:array_agg`.

Date and date-time grouping details: the interval start is rendered as text in the storage format; for a date-time field the interval bounds are converted from the context time zone to coordinated universal time before filtering, which correctly handles a daylight-saving change inside the interval; the label is formatted in the context language, except the week label, which is always `"W<week number> <year>"` because locale libraries disagree about week numbering.

Group expansion: when the context asks for it and the grouping is a single field carrying a group-expansion rule, the groups the rule names are added with zero aggregates, in the rule's order, reversed when the ordering is descending. Expansion is skipped when an offset is set or the limit is reached, in order that the pager stays consistent.

Temporal filling: when the context carries the filling flag, gaps between the first and the last group of a date or date-time grouping are filled with empty groups; the optional bounds `fill_from` and `fill_to` extend the filled range, and `min_groups` guarantees a minimum number of groups. Filling together with a limit or an offset is refused with `"You cannot used fill_temporal with a limit or an offset"`.

`web_read_group` wraps the above for a screen:

1. It groups by the first level only, with the caller's limit, offset and ordering, and computes `length`: when no group came back, zero; when the limit was reached, the limit plus the number of remaining groups; otherwise the offset plus the number of groups.
2. It decides which groups to **open**. A group is open when the caller's `opening_info` says it is unfolded, or, absent such information, when automatic unfolding is on and the group is not folded by default and fewer than the maximum number of groups (10, overridable by context) have been opened already. A group whose grouping value is an empty relation is folded by default. A folded group carries neither records nor subgroups.
3. For an open group that is not on the last grouping level, it computes the subgroups with the same rules and stores them under `__groups` as `{"groups": [...], "length": <integer>}`.
4. For an open group on the last level, it records the request and, at the end, performs **one** search per open group and **one** read for all of them together, storing the payloads under `__records`. The per-group limit is the caller's `unfold_read_default_limit` (80 by default) and the per-group offset comes from the opening information; an offset beyond the group's count is reset to zero and the group carries `__offset: 0` in order that the client can correct its pager.
5. The ordering used to read the records inside a group is the caller's ordering minus the parts that are constant within the group, followed by the entity's own ordering.
6. When `groupby_read_specification` names a grouping field, each group additionally carries `__values`, the payload of the grouping record read with that specification, or `{"id": false}` for the empty group.
7. `__fold` is removed from the answer: the client infers openness from the presence of `__records` or `__groups`.

### 7.7 The length of a search result

```
if no record was returned: length := 0
else:
    current := offset + number_of_returned_records
    if limit is set and (number_of_returned_records = limit and not (count_limit is set and count_limit ≤ current))
       or the context forces an exact count:
        length := search_count(domain, limit = count_limit)
    else:
        length := current
```

Worked example: 250 records match, the caller asks for `offset = 0, limit = 80, count_limit = 200`. Eighty records come back, the ceiling is not reached (200 > 80), therefore a bounded count runs and answers 200. The client displays `1-80 / 200+`. With `count_limit = 80` instead, the ceiling is reached and the length is 80, and the client displays `1-80 / 80+`.

### 7.8 The on-change protocol

The protocol lets a form stay consistent while the user types, without saving anything.

**Inputs**

| Name | Meaning |
|---|---|
| `values` | The current state of the form: a mapping from field name to value, in the write format, including the relational commands for the lines the user added, changed or removed. |
| `field_names` | The names of the fields the user just changed. An **empty** array means "this is a brand new record, give me everything". |
| `fields_spec` | The read specification of the form (section 7.2). It determines which fields are watched, which are returned, and how deeply the lines are described. |

The record set is the record being edited, or empty for a creation.

**Algorithm**

1. Flush pending writes, in order that derived values are computed on a consistent state.
2. Check access: `write` when a record is being edited, `create` when one is being created. (The user entity is exempt, because a user may edit their own record without general write access.)
3. When any name in `field_names` is not a field of the entity, answer an empty object.
4. **First call only** (`field_names` empty):
   1. `field_names` becomes the names present in `values` other than the record key.
   2. For every field of the specification that is absent from `values`, take the default; when there is one, add it to `values` and to `field_names`; when there is none, set it to false, except for a derived field that has no dependency, which is left unset in order that it is computed rather than forced.
5. Prefetch: read every field of the specification on the record, and, for every relational field with a nested specification, read the nested fields of every line named in `values` (existing lines plus the targets of update, link and replace commands), then copy those values onto the in-memory copies of the lines, in order that derived stored fields are not recomputed needlessly.
6. Build an in-memory record: for an existing record, a copy carrying the record's own values overlaid with the values the client sent minus the just-changed ones; for a creation, a new record carrying those same values with the just-changed ones set to false. Delegated parents are updated to match, in order that fields derived from a parent see their real inputs.
7. Take **snapshot zero** of the record through the specification, then apply the just-changed values, then refresh the snapshot for those fields.
8. Determine the work list: on a first call, every field of `values` followed by every field of the specification; otherwise the just-changed fields.
9. Mark the changed fields as modified, which schedules the recomputation of everything that depends on them, protecting the fields the user typed from being overwritten by their own derivation.
10. Loop: for every field of the work list, run its declared on-change handlers, each at most once per pass; a handler may assign fields on the record and may emit a warning. Then, unless the context disables recursion, rebuild the work list from the fields of the specification that are not done yet and whose snapshot changed, and repeat.
11. Take **snapshot one**.
12. The result's `value` member is the difference between snapshot one and snapshot zero, forced to "everything" on a first call.
13. Warnings are collected, without duplicates, in the order they were emitted. Exactly one warning is returned as it was emitted, its kind defaulting to `dialog` when the handler gave none. Several warnings are merged into one whose title is `"Warnings"`, whose kind is `dialog`, and whose body is built by writing, for each warning, its title, a blank line and its message, and joining those blocks with blank lines. A handler that gives no title contributes the title `"Warning"`.

**Output**

```
{
  "value":   { <field name>: <value or command list>, ... },
  "warning": { "title": <text>, "message": <text>, "type": "dialog" | "notification" }
}
```

Both members are optional. Rules for `value`:

- A plain or `many_to_one` field carries the value in the **read** format produced by the specification, therefore a `many_to_one` with a nested specification comes back as an object.
- A one-to-many or many-to-many field carries a **command list** the client applies to what it already holds:

| Situation | Command emitted |
|---|---|
| A line present before and absent now | `delete` for a one-to-many, `unlink` for a many-to-many, on the line's origin key (or 0 for a line that was never saved). |
| A line present in both whose values changed | `update` with only the changed members. |
| A line that did not exist before | `create` with the whole line payload. |
| A line that exists in the database and is newly linked | `link` carrying the line's payload, immediately followed by `update` with the differences, when there are any. |

- The record key is never part of `value`.

**Worked example.** A form on a sales order shows `partner_id`, `payment_term_id`, `currency_id` and the lines. The user selects a customer whose payment terms are "30 days" and whose currency is the dollar, and who has a default line discount of 10 per cent. The client calls:

```
values      = {"partner_id": 14, "payment_term_id": false, "currency_id": 1,
               "line_ids": [[0, "virtual_1", {"product_id": 42, "product_uom_qty": 1,
                                              "price_unit": 120.0, "discount": 0.0}]]}
field_names = ["partner_id"]
fields_spec = {"partner_id": {"fields": {"display_name": {}}},
               "payment_term_id": {"fields": {"display_name": {}}},
               "currency_id": {"fields": {"display_name": {}}},
               "line_ids": {"fields": {"product_id": {"fields": {"display_name": {}}},
                                       "product_uom_qty": {}, "price_unit": {},
                                       "discount": {}, "price_subtotal": {}}}}
```

The answer:

```
{"value": {
   "payment_term_id": {"id": 3, "display_name": "30 Days"},
   "currency_id": {"id": 2, "display_name": "USD"},
   "line_ids": [[1, "virtual_1", {"discount": 10.0, "price_subtotal": 108.0}]]}}
```

`partner_id` itself is not returned, because it did not change relative to the snapshot the client already has. The line command updates only the two members that changed.

**Worked example with a warning.** The user sets a quantity above the available stock. The handler emits a warning and the answer is:

```
{"value": {"line_ids": [[1, "virtual_1", {"price_subtotal": 1080.0}]]},
 "warning": {"title": "Not enough inventory!",
             "message": "You plan to sell 10.00 units but you only have 4.00 available.",
             "type": "dialog"}}
```

**Batch form.** `onchange_batch` applies the same algorithm independently to each entry of `values_list` and answers the array of results, in the same order. It refuses a non-empty record set: it exists only to prepare several new lines at once.

## 8. Export and import

### 8.1 The two modes

| Mode | Purpose | Column headers | Relational values |
|---|---|---|---|
| Re-importable | The produced file can be fed back into the import operation | The field paths themselves | External names, in order that references survive a transfer between databases. |
| Readable | The file is for a human or a spreadsheet | The labels the user chose | Display names. |

The mode is carried by the context flag `import_compat`, and by the `import_compat` member of the export request.

### 8.2 Field paths

A column is designated by a path of field names separated by slashes, with two special leaves:

| Leaf | Meaning |
|---|---|
| `id` | The external name of the record. In the re-importable mode it is created on the fly when the record has none, as `<table>_<key>_<eight random hexadecimal characters>`, and stored, in order that a later import updates the same record instead of creating a new one. |
| `.id` | The numeric key of the record. Available in the readable mode only. |
| `<properties field>.<property name>` | One dynamic property. |

Path depth is not limited for the export itself; the field picker only offers two levels of relation.

### 8.3 The exported matrix

`export_data` answers `{"datas": <matrix>}`. The matrix is built record by record:

1. Each record produces one **main row**, initialized with empty cells.
2. Each column is filled in order. A non-relational value is converted to its export form (a date as text, a selection as its label, a number as a number, a boolean as `True`/`False` in the readable mode).
3. A relational column is expanded by recursion: the sub-rows produced for the target records are merged, the first sub-row into the main row, the remaining sub-rows appended **after** the main row as continuation rows whose other columns stay empty. This is what makes a record with three lines occupy three rows.
4. In the re-importable mode a many-to-many is never expanded into rows: it is written into a single cell as a comma-separated list of external names, or of display names when the chosen sub-field is not the external name.
5. In the re-importable mode a reference field is written as `<entity>,<key>`.
6. When no sub-field was chosen for a relational column, the display name is used.
7. At the end, every cell holding a record designation is replaced by the record's external name, created if needed, in one pass per entity.

Worked example. Fields `name`, `partner_id/display_name`, `line_ids/product_id/display_name`, `line_ids/product_uom_qty` on one order with two lines:

```
["S00021", "Deco Addict", "Office Chair", 3.0]
["",       "",            "Desk",         1.0]
```

### 8.4 The export endpoints

Field discovery (`get_fields`) answers, for one level of one entity, the array of candidate columns sorted by label, each entry carrying:

| Member | Meaning |
|---|---|
| `id` | The path, prefixed by the parent path. |
| `string` | The label, prefixed by the parent label. |
| `value` | The value the client sends back: the path, or the path with `/id` appended for a relation. |
| `children` | Whether the entry can be expanded (true for a relation, up to two levels). |
| `field_type` | The field type. |
| `required` | Whether the field is required. |
| `relation_field` | The inverse field name for a one-to-many. |
| `default_export` | Whether the field belongs to the default re-importable selection. |
| `params` | For an expandable entry: the target entity, the prefix, the label and the parent field, to be sent back when expanding. |

Filtering rules: the external-name entry is labelled `"External ID"`; in the re-importable mode, read-only fields and explicitly excluded fields are dropped, and expanding a `many_to_one` or `many_to_many` offers only the external name and the naming field; in the readable mode the numeric-key entry `.id` is added; fields marked as not exportable are always dropped; entities that are not ordinary tables have no external-name entry. Dynamic properties are added as entries named `<properties field>.<property name>`, labelled `"<property label> (<owner record name>)"`, restricted to the definitions actually used by the records being exported.

Saved templates (`namelist`) resolve a stored list of paths into the same entries, with the labels of every level joined by slashes, and return them in the stored order.

The file endpoints receive `{model, fields: [{name, label}], ids, domain, groupby, import_compat, context}`. When `ids` is empty the export runs over the whole filter. Records are read in batches of 1000 and the in-memory cache is dropped between batches, therefore an export of any size uses bounded memory. Every export is logged with the caller, the number of records, the entity, the source address, the field list and either a sample of ten keys or the filter.

### 8.5 Grouped export

In the readable mode with a grouping, the spreadsheet export produces a tree:

1. The rows are produced once, prefixed with the numeric key column.
2. The groups are computed with their counts and the array of member keys.
3. Each row is attached to every group it belongs to, preserving the natural order of the records.
4. The tree is written group by group: a bold header row holding `"<indentation><label> (<count>)"` in the first column and the aggregates of the other columns, then the sub-groups recursively, then the member rows.
5. The aggregate of a column is computed from the field's declared aggregation: sum, maximum, minimum, logical conjunction, logical disjunction, or average. A leaf group aggregates its own rows; a parent aggregates its children's aggregates; an average aggregates the weighted sums of the children divided by the total count. Empty continuation cells are excluded from aggregation. A field whose aggregation is not one of the supported ones is left empty and logged.
6. A group whose value is empty is labelled `"Undefined"`, except for a boolean grouping, where false is a meaningful label.

The flat format refuses grouped exports with `"Exporting grouped data to csv is not supported."`.

### 8.6 File formats

| Format | Media type | Extension | Rules |
|---|---|---|---|
| Comma-separated values | `text/csv;charset=utf8` | `.csv` | Every cell is quoted. Empty and false become the empty text. A cell starting with `=`, `-` or `+` is prefixed with an apostrophe, in order that a spreadsheet does not interpret it as a formula. Binary cells are decoded to text. |
| Spreadsheet | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | `.xlsx` | The header row is bold; columns are 30 units wide; dates use `yyyy-mm-dd`, date-times `yyyy-mm-dd hh:mm:ss`, decimals `#,##0.00`, monetary values a pattern with as many decimals as the currency with the most decimals in the database. Text is wrapped and carriage returns are replaced by spaces. A binary cell that is not transport-encoded fails with `"Binary fields can not be exported to Excel unless their content is base64-encoded. That does not seem to be the case for <column>."`. The row and cell-length limits of section 12 of the transport document apply. |

The downloaded file is named `"<entity description> (<entity name>)<extension>"`, cleaned of characters that are invalid in a file name.

### 8.7 Import

`load` is the inverse operation. Inputs: `fields`, the array of column paths in the same order as the columns; `data`, the matrix of raw cells, all as text.

Processing:

1. A savepoint is taken, in order that a failure leaves nothing behind.
2. Rows are grouped into records: a row that carries a value in at least one non-one-to-many column starts a new record; the rows that follow and that carry values **only** in one-to-many columns are continuation rows of that record, and are folded into it recursively. This is exactly the inverse of the export row layout.
3. Each raw cell is converted to the write format according to its field type; a relational cell is resolved by external name, by numeric key, or by display name, in that order; an unresolvable value produces an error message rather than an exception.
4. Records are created or updated in batches. A record designated by an external name updates the existing record; a record designated by `.id` updates that record; otherwise a record is created.
5. When any message has the kind `error`, the savepoint is rolled back, the registry changes are discarded, and `ids` is false.

The answer is `{"ids": <array of created and updated keys, in input order, or false>, "messages": [...], "nextrow": <integer>}`. A message carries `type` (`error`, `warning` or `info`), `message`, the row range it concerns (`rows` with `from` and `to`), optionally `field` and `record`, and optionally `moreinfo` with further guidance (for example `"Resolve other errors first"`). `nextrow` is the first row not processed when a row limit was given, zero when everything was processed.

When more than ten errors occur **and** more than one error per ten records, the run stops early and adds the warning `"Found more than 10 errors and more than one error per 10 records, interrupted to avoid showing too many errors."`.

## 9. Discovery

A client or an integration never hardcodes the schema: it asks for it.

### 9.1 Field descriptions

`fields_get` answers, per readable field, a mapping of attributes. The attributes and their meaning:

| Attribute | Meaning |
|---|---|
| `type` | The field type (section 2.6 of this document; the canonical type vocabulary is in the charter's data-type list). |
| `string` | The label, translated. |
| `help` | The tooltip, translated. |
| `required` | Whether a value is mandatory. |
| `readonly` | Whether the field cannot be written. It is forced to true when the caller may read but not write the field. |
| `store` | Whether the value is stored or computed on the fly. |
| `searchable` | Whether the field may appear in a filter. |
| `sortable` | Whether the field may appear in an ordering. |
| `groupable` | Whether the field may be grouped on. |
| `aggregator` | The aggregation function used when grouping, when there is one. |
| `relation` | The target entity, for a relational field. |
| `relation_field` | The inverse field, for a one-to-many. |
| `model_field` | The companion field holding the entity name, for a dynamic reference. |
| `domain` | The default restriction applied when picking a target. |
| `context` | The context applied when reading or picking the target. |
| `selection` | The closed list of `[value, label]` pairs, for a selection field. |
| `digits` | The precision, for a decimal field. |
| `min_display_digits` | The minimum number of decimals to display. |
| `currency_field` | The field holding the currency, for a monetary field. |
| `translate` | Whether the field has per-language values. |
| `trim` | Whether leading and trailing spaces are removed on write. |
| `size` | The maximum length, when one is declared. |
| `change_default` | Whether this field's value can select user defaults for other fields. |
| `groups` | The access groups that may see the field. |
| `related` | The path, for a field mirrored from a related record. |
| `definition_record`, `definition_record_field` | Where the definitions of a dynamic-properties field live. |
| `default_export_compatible` | Whether the field belongs to the default re-importable export selection. |
| `exportable` | Whether the field may be exported. |
| `falsy_value_label` | The label to display instead of an empty value. |
| `tracking` | Whether changes are recorded in the record's discussion thread. |
| `name` | The field's own name. |

A caller may restrict both the fields and the attributes returned. A field the caller may not read is **absent**, which is how a client discovers its own field-level permissions.

### 9.2 Entity descriptions

`get_definitions`, reached through the two definition endpoints, answers for each requested entity:

```
{ <entity>: {
    "description": <the entity's label>,
    "fields": { <field>: { <a subset of the attributes above> } },
    "inherit": [ <the requested entities this one inherits from> ],
    "order": <the default ordering>,
    "parent_name": <the field forming the hierarchy>,
    "rec_name": <the naming field>,
    "has_activities": <true when the entity carries scheduled activities>
} }
```

Filtering rules that a replacement must reproduce: a relational field whose target is **not** among the requested entities is omitted, because the client could not follow it; a mirrored field whose source field was omitted is omitted too; for each kept field, the inverse fields that live in requested entities and that the caller may read are listed under `inverse_fname_by_model_name`; a dynamic reference field carries the name of its companion entity-name field under `model_name_ref_fname`; a field that is tracked in the discussion thread carries `tracking`.

### 9.3 Listing entities

| Operation | Answer |
|---|---|
| `get_available_models` on the entity registry | Every entity the caller may read, as `{"model", "display_name"}`, excluding transient and abstract entities and excluding callers that are not internal users. |
| `display_name_for` on the entity registry | For a given list of entity names, the same pairs; a name that is unknown **and** a name that is inaccessible both answer `{"model": <name>, "display_name": <name>}`, deliberately indistinguishable, in order that the operation does not leak the existence of entities. |

### 9.4 The documentation documents

The two documentation endpoints ([`remote-transport-contracts.md`](remote-transport-contracts.md), section 9.15) are the machine-readable catalogue of a live deployment: the index lists every capability package in dependency order and every entity with its label, its readable fields and its callable operations; the per-entity document adds the full field descriptions and, for every callable operation, its signature, its parameters with their kinds, defaults, types and documentation, its return description, the failures it declares, its marks (entity-level, read-only) and the entity and package that introduced it. Operations that are deprecated are absent from both.

This is the contract an integration should read first: it is generated from the running registry, therefore it always matches the deployment, including the fields and operations that capability packages added.

## 10. Named business operations by domain

Beyond the generic contract of section 2, every capability package adds **named business operations** to the entities it
owns. They follow the conventions of section 5 and they are the operations a screen binds to a control, an automation
invokes and an integration calls through the generic dispatch
([`remote-transport-contracts.md`](remote-transport-contracts.md), section 8).

The tables below enumerate every named operation that a screen of the shipped installation binds to a control: 1,016
operations on 285 entities, grouped into thirty-eight areas by the part of the system that owns the entity. For each one
the table gives the entity, the entity's transport name, the operation's identifier, and the label the control shows.
Control labels are **reproduced** verbatim from the shipped installation and are therefore given in quotation marks; a
label written as a field identifier in code font is a counter that the control displays instead of a fixed word, which is
how a statistic button is labelled, and the field is described on the entity's reference page. Three labels name the
product in the shipped installation (a video-meeting location, a document-exchange registration and a
document-exchange sender registration); they are shown here with the product's name replaced by "the platform", because
a replacement substitutes its own name and no observable behaviour depends on the word.

**What is true of all of them, and therefore stated once here rather than in every row:**

| Aspect | Rule |
|---|---|
| Inputs | The record set the control was pressed on, plus the request context. An operation that needs more input does not take arguments: it returns a window action that opens a transient entity, and the transient record carries the input (section 6.1). |
| Output | Nothing, which means "done, refresh"; or one action to run next, of one of the six kinds of section 6.3; or a notification action (section 6.4). |
| Preconditions | Re-checked inside the operation, never assumed from the fact that the control was visible. A control that is hidden by its own condition can still be reached through the generic dispatch, therefore the guard is in the operation. The exact guards and their refusal messages are specified in the `business-rules.md` and `state-machines.md` files of the owning folder. |
| Side effects | The records the operation creates, changes or deletes, the ledger entries it produces and the messages it posts are specified in the `workflows.md` and `accounting-effects.md` files of the owning folder. |
| Record set | An operation meaningful on one record only says so and refuses a larger set; every other operation iterates over the set (section 6.2). |
| Transaction | One call, one transaction; an operation therefore performs the whole unit of work, because two calls can never be made atomic ([`remote-transport-contracts.md`](remote-transport-contracts.md), section 8.9). |
| Access | The operation runs with the caller's rights. A control may additionally be restricted to an access group, which hides it; the access rules of the entity are enforced regardless. |

The complete catalogue of every operation of every entity, including the ones no screen binds to a control, is
[`../references/operation-index.md`](../references/operation-index.md).

### 10.1 Analytic accounting

10 operations on 3 entities. Preconditions, guards, refusal messages and side effects: [`../domains/analytic-accounting/`](../domains/analytic-accounting/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Analytic Account | `account.analytic.account` | `action_view_invoice` | `invoice_count` |
| Analytic Account | `account.analytic.account` | `action_view_mrp_bom` | `bom_count` |
| Analytic Account | `account.analytic.account` | `action_view_mrp_production` | `production_count` |
| Analytic Account | `account.analytic.account` | `action_view_projects` | `project_count` |
| Analytic Account | `account.analytic.account` | `action_view_purchase_orders` | `purchase_order_count` |
| Analytic Account | `account.analytic.account` | `action_view_vendor_bill` | `vendor_bill_count` |
| Analytic Line | `account.analytic.line` | `action_invoice_from_timesheet` | "Invoice" |
| Analytic Line | `account.analytic.line` | `action_sale_order_from_timesheet` | "Sales Order" |
| Analytic Plans | `account.analytic.plan` | `action_view_analytical_accounts` | "`all_account_count` Analytic Accounts" |
| Analytic Plans | `account.analytic.plan` | `action_view_children_plans` | `children_count` |

### 10.2 Attendance recording

7 operations on 2 entities. Preconditions, guards, refusal messages and side effects: [`../domains/attendances-and-working-time/`](../domains/attendances-and-working-time/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Attendance | `hr.attendance` | `action_approve` | "Approve" |
| Attendance | `hr.attendance` | `action_approve_overtime` | "Approve Extra Hours" / " " / "Approve" / "1" |
| Attendance | `hr.attendance` | `action_in_attendance_maps` | "View on Maps" |
| Attendance | `hr.attendance` | `action_out_attendance_maps` | "View on Maps" |
| Attendance | `hr.attendance` | `action_refuse` | "Refuse" |
| Attendance | `hr.attendance` | `action_refuse_overtime` | "Refuse Extra Hours" / " " / "Refuse" / "1" |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | `action_regenerate_overtimes` | "Regenerate overtimes" |

### 10.3 Automation and integration

4 operations on 2 entities. Preconditions, guards, refusal messages and side effects: [`../domains/automation-and-integration/`](../domains/automation-and-integration/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Recycling Model | `data_recycle.model` | `action_recycle_records` | "Run Now" |
| Recycling Model | `data_recycle.model` | `open_records` | `records_to_recycle_count` |
| Recycling Record | `data_recycle.record` | `action_discard` | "Discard" |
| Recycling Record | `data_recycle.record` | `action_validate` | "Validate" |

### 10.4 Calendar and scheduling

11 operations on 2 entities. Preconditions, guards, refusal messages and side effects: [`../domains/calendar-and-scheduling/`](../domains/calendar-and-scheduling/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Calendar Event | `calendar.event` | `action_join_video_call` | a control with no text of its own |
| Calendar Event | `calendar.event` | `action_open_calendar_event` | `res_model_name` |
| Calendar Event | `calendar.event` | `action_open_composer` | "Send Mail" / "Send email" / " EMAIL" |
| Calendar Event | `calendar.event` | `action_send_sms` | "Send SMS" / "SMS" |
| Calendar Event | `calendar.event` | `action_sendmail` | "Send Invitations" |
| Calendar Event | `calendar.event` | `do_accept` | "Accept" |
| Calendar Event | `calendar.event` | `do_decline` | "Decline" |
| Calendar Event | `calendar.event` | `do_tentative` | "Uncertain" |
| Calendar Event | `calendar.event` | `set_discuss_videocall_location` | "the platform meeting" / "Video" |
| Calendar Popover Delete Wizard | `calendar.popover.delete.wizard` | `action_delete` | "Delete" |
| Calendar Popover Delete Wizard | `calendar.popover.delete.wizard` | `action_send_mail_and_delete` | "Send and delete" |

### 10.5 Contacts and organizations

22 operations on 4 entities. Preconditions, guards, refusal messages and side effects: [`../domains/contacts-and-organizations/`](../domains/contacts-and-organizations/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Portal Sharing | `portal.share` | `action_send_mail` | "Send" |
| Grant Portal Access | `portal.wizard` | `action_grant_access` | "Grant Access" |
| Grant Portal Access | `portal.wizard` | `action_invite_again` | "Re-Invite" |
| Grant Portal Access | `portal.wizard` | `action_refresh_modal` | "Valid Email Address" / "Invalid Email Address" / "Email Address already taken by another user" |
| Grant Portal Access | `portal.wizard` | `action_revoke_access` | "Revoke Access" |
| Contact | `res.partner` | `action_event_view` | `event_count` |
| Contact | `res.partner` | `action_l10n_in_verify_gstin_status` | "Reverify GSTIN status" / "Check Status" |
| Contact | `res.partner` | `action_open_employees` | "`employees_count` Employee" |
| Contact | `res.partner` | `action_validate_tin` | "Validate" |
| Contact | `res.partner` | `action_view_certifications` | "`certifications_count` Certifications Certification" / "`certifications_company_count` Certifications Certification" |
| Contact | `res.partner` | `action_view_courses` | "`slide_channel_count` Courses" / "`slide_channel_company_count` Courses" |
| Contact | `res.partner` | `action_view_livechat_sessions` | `livechat_channel_count` |
| Contact | `res.partner` | `action_view_loyalty_cards` | `loyalty_card_count` |
| Contact | `res.partner` | `action_view_opportunity` | `opportunity_count` |
| Contact | `res.partner` | `action_view_partner_invoices` | "`currency_id` `total_invoiced` Invoiced" |
| Contact | `res.partner` | `action_view_pos_order` | `pos_order_count` |
| Contact | `res.partner` | `action_view_stock_serial` | "Lots/Serial Numbers" |
| Contact | `res.partner` | `action_view_tasks` | `task_count` |
| Contact | `res.partner` | `button_account_peppol_check_partner_endpoint` | "Verify" |
| Contact | `res.partner` | `button_nemhandel_check_partner_endpoint` | "Verify" |
| Contact | `res.partner` | `open_commercial_entity` | "the parent company" |
| Bank Accounts | `res.partner.bank` | `action_archive_bank` | "Archive" |

### 10.6 Customer relationship management

27 operations on 12 entities. Preconditions, guards, refusal messages and side effects: [`../domains/customer-relationship-management/`](../domains/customer-relationship-management/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `action_buy_credits` | "Buy credits." |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `action_get_lead_action` | "`lead_count` Leads" |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `action_get_opportunity_action` | "`lead_count` Opportunities" |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `action_submit` | "Submit" / "Retry" / "Generate Leads" |
| Lead | `crm.lead` | `action_assign_partner` | "Automatic Assignment" |
| Lead | `crm.lead` | `action_generate_leads` | "Generate Leads" |
| Lead | `crm.lead` | `action_open_livechat` | "View chat" |
| Lead | `crm.lead` | `action_redirect_to_livechat_sessions` | `visitor_sessions_count` |
| Lead | `crm.lead` | `action_redirect_to_page_views` | `visitor_page_count` |
| Lead | `crm.lead` | `action_restore` | "Restore" |
| Lead | `crm.lead` | `action_sale_quotations_new` | "New Quotation" |
| Lead | `crm.lead` | `action_schedule_meeting` | "`quotation_count` `sale_amount_total` Orders `sale_order_count`" / "`meeting_display_label` `meeting_display_date`" |
| Lead | `crm.lead` | `action_set_won_rainbowman` | "Won" |
| Lead | `crm.lead` | `action_show_potential_duplicates` | "`duplicate_lead_count` Similar Leads Similar Lead" |
| Lead | `crm.lead` | `action_view_sale_order` | "`sale_amount_total` Orders `sale_order_count`" |
| Lead | `crm.lead` | `action_view_sale_quotation` | `quotation_count` |
| Lead forward to partner | `crm.lead.forward.to.partner` | `action_forward` | "Send" |
| Get Lost Reason | `crm.lead.lost` | `action_lost_reason_apply` | "Mark as Lost" |
| Update the probabilities | `crm.lead.pls.update` | `action_update_crm_lead_probabilities` | "Update" |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `action_apply` | "Create Opportunity" |
| Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `action_mass_convert` | "Convert to Opportunities" |
| Opp. Lost Reason | `crm.lost.reason` | `action_lost_leads` | "`leads_count` Leads" |
| Merge Opportunities | `crm.merge.opportunity` | `action_merge` | "Merge" |
| Create new or use existing Customer on new Quotation | `crm.quotation.partner` | `action_apply` | "Confirm" |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `action_get_lead_tree_view` | "`lead_count` Leads" |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `action_get_opportunity_tree_view` | "`opportunity_count` Opportunities" |
| Sales Team | `crm.team` | `action_assign_leads` | "Assign Leads" |

### 10.7 Delivery and shipping

3 operations on 2 entities. Preconditions, guards, refusal messages and side effects: [`../domains/delivery-and-shipping/`](../domains/delivery-and-shipping/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `button_confirm` | "Update" / "Add" |
| Shipping Methods | `delivery.carrier` | `toggle_debug` | "No debug" / "Debug requests" |
| Shipping Methods | `delivery.carrier` | `toggle_prod_environment` | "Production Environment" / "Test Environment" / `is_published` |

### 10.8 Electronic invoicing and document exchange

27 operations on 10 entities. Preconditions, guards, refusal messages and side effects: [`../domains/electronic-invoicing-and-document-exchange/`](../domains/electronic-invoicing-and-document-exchange/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Consolidate Invoice Wizard | `myinvois.consolidate.invoice.wizard` | `button_consolidate` | "Create Consolidated Invoices" |
| MyInvois Document | `myinvois.document` | `action_cancel_submission` | "Cancel Submission" |
| MyInvois Document | `myinvois.document` | `action_open_consolidate_invoice_wizard` | "Consolidate Orders" |
| MyInvois Document | `myinvois.document` | `action_submit_to_myinvois` | "Submit To MyInvois" |
| MyInvois Document | `myinvois.document` | `action_update_submission_status` | "Update Submission Status" / "Fetch Status From MyInvois" |
| MyInvois Document | `myinvois.document` | `action_view_linked_orders` | `linked_order_count` |
| Document Status Update Wizard | `myinvois.document.status.update.wizard` | `button_request_update` | "Update Document" |
| Nemhandel Registration | `nemhandel.registration` | `button_check_nemhandel_verification_code` | "Activate Nemhandel (Demo)" / "Confirm" |
| Nemhandel Registration | `nemhandel.registration` | `button_deregister_nemhandel_participant` | "Cancel Registration" |
| Nemhandel Registration | `nemhandel.registration` | `button_nemhandel_registration_sms` | "Activate Nemhandel" / "Activate Nemhandel (Test)" |
| Nemhandel Registration | `nemhandel.registration` | `send_nemhandel_verification_code` | "Send again" |
| Nemhandel Rejection wizard | `nemhandel.rejection.wizard` | `button_send` | "Send Rejection" |
| Peppol Configuration Wizard | `pdp.config.wizard` | `button_peppol_unregister` | "Disconnect French electronic invoicing" |
| Peppol Configuration Wizard | `pdp.config.wizard` | `button_sync_form_with_peppol_proxy` | "Save" |
| PDP Registration | `pdp.registration` | `button_cancel_authentication` | "Cancel" |
| PDP Registration | `pdp.registration` | `button_deregister_pdp_participant` | "Cancel Registration" |
| PDP Registration | `pdp.registration` | `button_open_authentication_link` | "Open link" |
| PDP Registration | `pdp.registration` | `button_refresh_authentication` | "Refresh" |
| PDP Registration | `pdp.registration` | `button_register_pdp_participant` | "Validate Registration Migrate to the platform Validate Registration (Test) Migrate to the platform (Test) Validate Registration (Demo) Migrate to the platform (Demo)" |
| PDP Registration | `pdp.registration` | `button_trigger_authentication` | "Authenticate" |
| PDP Response wizard | `pdp.response.wizard` | `button_send` | "Send" |
| Peppol Configuration Wizard | `peppol.config.wizard` | `button_peppol_register_sender_as_receiver` | "Enable reception" |
| Peppol Configuration Wizard | `peppol.config.wizard` | `button_peppol_reset_to_sender` | "Disable the reception." |
| Peppol Configuration Wizard | `peppol.config.wizard` | `button_peppol_unregister` | "Remove from Peppol" |
| Peppol Configuration Wizard | `peppol.config.wizard` | `button_sync_form_with_peppol_proxy` | "Save" |
| Peppol Registration | `peppol.registration` | `button_register_peppol_participant` | "Activate Peppol" / "Activate Peppol (Test)" / "Activate Peppol (Demo)" |
| Peppol Registration | `peppol.registration` | `button_register_with_itsme` | "Authenticate" |

### 10.9 Events

21 operations on 7 entities. Preconditions, guards, refusal messages and side effects: [`../domains/events/`](../domains/events/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Event Booth | `event.booth` | `action_view_sale_order` | "Sale Order" |
| Event Booth | `event.booth` | `action_view_sponsor` | "Sponsor" |
| Event | `event.event` | `action_invite_contacts` | "Invite" |
| Event | `event.event` | `action_mass_mailing_attendees` | "Contact Attendees" |
| Event | `event.event` | `action_mass_mailing_track_speakers` | "Contact Speakers" |
| Event | `event.event` | `action_open_slot_calendar` | "`event_slot_count` Slot(s)" |
| Event | `event.event` | `action_view_linked_orders` | "`sale_price_total` Sales" |
| Event | `event.event` | `action_view_question_answers` | "Stats" |
| Event Lead Rules | `event.lead.rule` | `action_execute_rule` | "Execute Rule" |
| Event Question | `event.question` | `action_add_rule_button` | "Add a rule" |
| Event Question | `event.question` | `action_event_view` | "`event_count` events" |
| Event Question | `event.question` | `action_view_question_answers` | "Attendee answers" / "Stats" |
| Event Registration | `event.registration` | `action_cancel` | "Cancel" / "Cancel Registration" |
| Event Registration | `event.registration` | `action_confirm` | "Registered" |
| Event Registration | `event.registration` | `action_send_badge_email` | "Send by Email" |
| Event Registration | `event.registration` | `action_set_done` | "Mark as Attending" / "Attended" |
| Event Registration | `event.registration` | `action_view_pos_order` | "PoS Order" |
| Event Registration | `event.registration` | `action_view_sale_order` | "Sale Order" |
| Event Track | `event.track` | `action_add_quiz` | "Add Quiz" |
| Event Track | `event.track` | `action_view_quiz` | "Go to Quiz" |
| Edit Attendee Details on Sales Confirmation | `registration.editor` | `action_make_registration` | "Create/Update registrations" |

### 10.10 Expenses

14 operations on 5 entities. Preconditions, guards, refusal messages and side effects: [`../domains/expenses/`](../domains/expenses/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Expense | `hr.expense` | `action_approve` | "Approve" |
| Expense | `hr.expense` | `action_open_account_move` | `sale_order_id` / "Journal Entry" |
| Expense | `hr.expense` | `action_open_sale_order` | `sale_order_id` |
| Expense | `hr.expense` | `action_open_split_expense` | "Split" |
| Expense | `hr.expense` | `action_post` | "Post Journal Entries" |
| Expense | `hr.expense` | `action_refuse` | "Refuse" |
| Expense | `hr.expense` | `action_reset` | "Reset" |
| Expense | `hr.expense` | `action_split_wizard` | "Split Expense" |
| Expense | `hr.expense` | `action_submit` | "Submit" |
| Expense Approve Duplicate | `hr.expense.approve.duplicate` | `action_approve` | "Approve" |
| Expense Approve Duplicate | `hr.expense.approve.duplicate` | `action_refuse` | "Refuse" |
| Expense Posting Wizard | `hr.expense.post.wizard` | `action_post_entry` | "Post Expenses" |
| Expense Refuse Reason Wizard | `hr.expense.refuse.wizard` | `action_refuse` | "Refuse" |
| Expense Split Wizard | `hr.expense.split.wizard` | `action_split_expense` | "Split Expense" |

### 10.11 Fiscal localizations

40 operations on 18 entities. Preconditions, guards, refusal messages and side effects: [`../domains/fiscal-localizations/`](../domains/fiscal-localizations/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| French PDP Flow | `l10n.fr.pdp.reports.flow` | `action_build_payload_manual` | "Build Payload" |
| French PDP Flow | `l10n.fr.pdp.reports.flow` | `action_open_send_wizard` | "Send" |
| French PDP Flow | `l10n.fr.pdp.reports.flow` | `action_view_error_moves` | "View invoices" |
| French PDP Flow | `l10n.fr.pdp.reports.flow` | `action_view_initial` | "Initial" |
| French PDP Flow | `l10n.fr.pdp.reports.flow` | `action_view_moves` | "`move_ids` `move_ids`" |
| Send PDP Flow Wizard | `l10n.fr.pdp.reports.send.wizard` | `action_send_anyway` | "Send anyway" |
| Send PDP Flow Wizard | `l10n.fr.pdp.reports.send.wizard` | `action_view_errors` | "View invoices" |
| e-Waybill | `l10n.in.ewaybill` | `action_cancel_ewaybill` | "Cancel e-Waybill" |
| e-Waybill | `l10n.in.ewaybill` | `action_export_content_json` | "Download Content JSON" |
| e-Waybill | `l10n.in.ewaybill` | `action_generate_ewaybill` | "Generate e-Waybill" |
| e-Waybill | `l10n.in.ewaybill` | `action_print` | "Print" |
| e-Waybill | `l10n.in.ewaybill` | `action_reset_to_pending` | "Reset to Pending" |
| e-Waybill | `l10n.in.ewaybill` | `action_set_to_challan` | "Use as Challan" |
| Handles problems occurring while creating multiple quick response-invoices at once | `l10n_ch.qr_invoice.wizard` | `action_view_faulty_invoices` | "Check invalid invoices" |
| Handles problems occurring while creating multiple quick response-invoices at once | `l10n_ch.qr_invoice.wizard` | `print_all_invoices` | "Print All" |
| Thumb drive used to sign invoices in Egypt | `l10n_eg_edi.thumb.drive` | `action_set_certificate_from_usb` | "Get certificate" |
| MojEracun Reject Invoice Wizard | `l10n_hr_edi.mojeracun_reject_wizard` | `button_reject_invoice` | "Reject invoice" |
| Technical Annulment Wizard | `l10n_hu_edi.cancellation` | `button_request_cancel` | "Request Annulment" |
| Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | `l10n_hu_edi.tax_audit_export` | `action_export` | "Export" |
| Receive Bills Wizard | `l10n_hu_edi_receive.bills.wizard` | `action_receive_bills` | "Fetch Bills" |
| E-Faktur Document | `l10n_id_efaktur_coretax.document` | `action_download` | "Download" |
| E-Faktur Document | `l10n_id_efaktur_coretax.document` | `action_regenerate` | "Regenerate File" |
| Withhold Wizard | `l10n_in.withhold.wizard` | `action_create_and_post_withhold` | "Apply TDS" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `action_open_invoice_ids` | "Invoices" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `action_open_sale_order_ids` | "Sale Orders" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `action_reactivate` | "Reactivate" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `action_reset_to_draft` | "Reset to Draft" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `action_revoke` | "Revoke" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `action_terminate` | "Terminate" |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `action_validate` | "Validate" |
| Account payment check | `l10n_latam.check` | `action_show_journal_entry` | "Journal Entry" |
| Account payment check | `l10n_latam.check` | `action_show_reconciled_move` | "Reconciled move" |
| Account payment check | `l10n_latam.check` | `action_void` | "Void Check" |
| Account payment check | `l10n_latam.check` | `button_open_check_operations` | "Operations" |
| Account payment check | `l10n_latam.check` | `button_open_payment` | "Payment" |
| Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `action_create_payments` | "Create Transfers" |
| Exports 2307 data to a XLS file. | `l10n_ph_2307.wizard` | `action_generate` | "Generate" |
| Implements cancelling an ecpay invoice. | `l10n_tw_edi.invoice.cancel` | `button_request_cancel` | "Cancel Invoice" |
| Implements printingan ecpay invoice. | `l10n_tw_edi.invoice.print` | `button_print` | "Print Invoice" |
| E-invoice cancellation wizard | `l10n_vn_edi_viettel.cancellation` | `button_request_cancel` | "Request Cancellation" |

### 10.12 Fleet

12 operations on 7 entities. Preconditions, guards, refusal messages and side effects: [`../domains/fleet/`](../domains/fleet/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Vehicle | `fleet.vehicle` | `action_accept_driver_change` | "Apply New Driver" |
| Vehicle | `fleet.vehicle` | `action_open_employee` | "1 Employee" |
| Vehicle | `fleet.vehicle` | `action_open_odometer_report` | "Odometer Report" |
| Vehicle | `fleet.vehicle` | `action_view_bills` | `bill_count` |
| Vehicle | `fleet.vehicle` | `open_assignation_logs` | "1 Employee" / `history_count` |
| Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `action_get_attachment_view` | "Attachments" |
| Vehicle Contract | `fleet.vehicle.log.contract` | `action_open_employee` | "Employee `purchaser_employee_id`" |
| Services for vehicles | `fleet.vehicle.log.services` | `action_open_account_move` | "Service's Bill Service's Bill" |
| Model of a vehicle | `fleet.vehicle.model` | `action_model_vehicle` | "`vehicle_count` New Vehicle" |
| Brand of the vehicle | `fleet.vehicle.model.brand` | `action_brand_model` | `model_count` |
| Send mails to Drivers | `fleet.vehicle.send.mail` | `action_save_as_template` | "Save as new template" |
| Send mails to Drivers | `fleet.vehicle.send.mail` | `action_send` | "Send" |

### 10.13 General ledger and invoicing

87 operations on 15 entities. Preconditions, guards, refusal messages and side effects: [`../domains/general-ledger/`](../domains/general-ledger/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Account | `account.account` | `action_open_related_taxes` | "`related_taxes_amount` Taxes" |
| Create Automatic Entries | `account.automatic.entry.wizard` | `do_action` | "Create Journal Entries" |
| Autopost Bills Wizard | `account.autopost.bills.wizard` | `action_ask_later` | "Ask me later" |
| Autopost Bills Wizard | `account.autopost.bills.wizard` | `action_automate_partner` | "Activate auto-validation" |
| Autopost Bills Wizard | `account.autopost.bills.wizard` | `action_never_automate_partner` | "Never for this vendor" |
| Opening Balance of Financial Year | `account.financial.year.op` | `action_save_onboarding_fiscal_year` | "Apply" |
| Journal | `account.journal` | `action_configure_bank_journal` | "Bank Setup" |
| Journal | `account.journal` | `action_create_new` | "New" |
| Journal | `account.journal` | `action_open_provider_form` | "SETUP" |
| Journal | `account.journal` | `open_action` | "Transactions" |
| Account Lock Exception | `account.lock_exception` | `action_revoke` | "Revoke" |
| Account Lock Exception | `account.lock_exception` | `action_show_audit_trail_during_exception` | "Audit" |
| Account merge wizard | `account.merge.wizard` | `action_merge` | "Merge" |
| Journal Entry | `account.move` | `action_activate_currency` | "activate the currency of the bill" / "activate the currency of the invoice" |
| Journal Entry | `account.move` | `action_add_from_catalog` | "Catalog" |
| Journal Entry | `account.move` | `action_automatic_entry` | "Cut-Off" |
| Journal Entry | `account.move` | `action_cancel_nemhandel_documents` | "Cancel Nemhandel" |
| Journal Entry | `account.move` | `action_cancel_peppol_documents` | "Cancel PEPPOL" |
| Journal Entry | `account.move` | `action_check_l10n_it_edi` | "Check Sending" |
| Journal Entry | `account.move` | `action_debit_note` | "Debit Note" |
| Journal Entry | `account.move` | `action_delete_duplicates` | "Delete all duplicates Delete duplicate" |
| Journal Entry | `account.move` | `action_download` | "Download" |
| Journal Entry | `account.move` | `action_export_l10n_in_edi_content_json` | "Download EDI JSON" |
| Journal Entry | `account.move` | `action_export_xml` | "Download" |
| Journal Entry | `account.move` | `action_force_register_payment` | "Pay" |
| Journal Entry | `account.move` | `action_get_eta_invoice_pdf` | "Get ETA Invoice PDF" |
| Journal Entry | `account.move` | `action_invoice_sent` | "Send" |
| Journal Entry | `account.move` | `action_l10n_in_apply_higher_tax` | "Apply Higher TCS" |
| Journal Entry | `account.move` | `action_l10n_in_edi_force_cancel` | "Force Cancel" |
| Journal Entry | `account.move` | `action_l10n_in_ewaybill_create` | "Create e-Waybill" |
| Journal Entry | `account.move` | `action_l10n_in_withholding_entries` | "TDS `l10n_in_total_withholding_amount`" |
| Journal Entry | `account.move` | `action_l10n_it_edi_send` | "Send to SDI" |
| Journal Entry | `account.move` | `action_l10n_my_edi_send_invoice` | "Send To MyInvois" |
| Journal Entry | `account.move` | `action_l10n_my_edi_update_status` | "Update MyInvois Status" |
| Journal Entry | `account.move` | `action_l10n_pl_edi_get_invoice_UPO` | "Download UPO" |
| Journal Entry | `account.move` | `action_l10n_pl_edi_update_invoice_status` | "Check Sending" |
| Journal Entry | `account.move` | `action_l10n_ro_edi_download_attachment` | "Download the document sent" / "Download the signature" |
| Journal Entry | `account.move` | `action_l10n_ro_edi_fetch_status` | "Fetch status" |
| Journal Entry | `account.move` | `action_l10n_vn_edi_update_payment_status` | "Send Payment Status" |
| Journal Entry | `account.move` | `action_open_business_doc` | "1 Payment" |
| Journal Entry | `account.move` | `action_open_declaration_of_intent` | "Declaration of Intent" |
| Journal Entry | `account.move` | `action_open_expense` | `nb_expenses` |
| Journal Entry | `account.move` | `action_open_l10n_in_ewaybill` | "e-Waybill" |
| Journal Entry | `account.move` | `action_post` | "Post" / "Confirm" |
| Journal Entry | `account.move` | `action_post_sign_invoices` | "Sign Invoice" |
| Journal Entry | `account.move` | `action_print_pdf` | "Print" |
| Journal Entry | `account.move` | `action_purchase_matching` | "Purchase Matching" |
| Journal Entry | `account.move` | `action_register_payment` | "Pay" |
| Journal Entry | `account.move` | `action_retry_edi_documents_error` | "Retry" |
| Journal Entry | `account.move` | `action_reverse` | "Credit Note" |
| Journal Entry | `account.move` | `action_send_and_print` | "Send" |
| Journal Entry | `account.move` | `action_show_chain_head` | "Blocking Invoice" |
| Journal Entry | `account.move` | `action_show_myinvois_documents` | "MyInvois Documents" |
| Journal Entry | `account.move` | `action_update_fpos_values` | "Update Taxes and Accounts" |
| Journal Entry | `account.move` | `action_view_debit_notes` | "`debit_note_count` Debit Notes" |
| Journal Entry | `account.move` | `action_view_landed_costs` | "Landed Costs" |
| Journal Entry | `account.move` | `action_view_payment_transactions` | `transaction_count` |
| Journal Entry | `account.move` | `action_view_source_pos_orders` | `pos_order_count` |
| Journal Entry | `account.move` | `action_view_source_purchase_orders` | "`purchase_order_name` `purchase_order_count`" |
| Journal Entry | `account.move` | `action_view_source_sale_orders` | `sale_order_count` |
| Journal Entry | `account.move` | `action_view_wip_production` | "`wip_production_count` Manufacturing" |
| Journal Entry | `account.move` | `button_abandon_cancel_posted_posted_moves` | "Call off EDI Cancellation" |
| Journal Entry | `account.move` | `button_cancel` | "Cancel Entry" / "Cancel" |
| Journal Entry | `account.move` | `button_cancel_posted_moves` | "Request EDI Cancellation" |
| Journal Entry | `account.move` | `button_create_landed_costs` | "Create Landed Costs" |
| Journal Entry | `account.move` | `button_draft` | "Reset to Draft" |
| Journal Entry | `account.move` | `button_force_cancel` | "Force Cancel" |
| Journal Entry | `account.move` | `button_hash` | "Lock" |
| Journal Entry | `account.move` | `button_process_edi_web_services` | "Process now" |
| Journal Entry | `account.move` | `button_request_cancel` | "Request Cancel" |
| Journal Entry | `account.move` | `button_set_checked` | "Reviewed" |
| Journal Entry | `account.move` | `open_adjusting_entries` | `adjusting_entries_count` |
| Journal Entry | `account.move` | `open_adjusting_entry_origin_moves` | "`adjusting_entry_origin_label` `adjusting_entry_origin_moves_count`" |
| Journal Entry | `account.move` | `open_created_caba_entries` | "Cash Basis Entries" |
| Journal Entry | `account.move` | `open_payments` | `payment_count` |
| Journal Entry | `account.move` | `open_reconcile_view` | "Reconciled Items" |
| Journal Item | `account.move.line` | `action_payment_items_register_payment` | "Pay" |
| Journal Item | `account.move.line` | `action_post` | "Post" |
| Journal Item | `account.move.line` | `open_reconcile_view` | "-> View partially reconciled entries" |
| Journal Item | `account.move.line` | `set_moves_checked` | "Review" |
| Account Move Send Batch Wizard | `account.move.send.batch.wizard` | `action_send_and_print` | "Send" |
| Account Move Send Wizard | `account.move.send.wizard` | `action_send_and_print` | "Send" / "Generate" |
| Peppol Rejection wizard | `account.peppol.rejection.wizard` | `button_send` | "Send Rejection" |
| Secure Journal Entries | `account.secure.entries.wizard` | `action_secure_entries` | "Secure Entries" |
| Bill to Purchase Order | `bill.to.po.wizard` | `action_add_downpayment` | "Add Down Payment" |
| Bill to Purchase Order | `bill.to.po.wizard` | `action_add_to_po` | "Add Products" |
| Validate Account Move | `validate.account.move` | `validate_move` | "Confirm" |

### 10.14 Human resources

45 operations on 18 entities. Preconditions, guards, refusal messages and side effects: [`../domains/human-resources-core/`](../domains/human-resources-core/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Digest | `digest.digest` | `action_activate` | "Activate" |
| Digest | `digest.digest` | `action_deactivate` | "Deactivate" |
| Digest | `digest.digest` | `action_send_manual` | "Send Now" |
| Gamification User Badge Wizard | `gamification.badge.user.wizard` | `action_grant_badge` | "Grant Badge" / "Grant a badge" |
| Gamification Challenge | `gamification.challenge` | `action_check` | "Refresh Challenge" |
| Gamification Challenge | `gamification.challenge` | `action_report_progress` | "Send Report" |
| Gamification Challenge | `gamification.challenge` | `action_start` | "Start Challenge" |
| Gamification Challenge | `gamification.challenge` | `action_view_users` | `user_count` |
| Gamification Goal | `gamification.goal` | `action_cancel` | "Reset Completion" |
| Gamification Goal | `gamification.goal` | `action_fail` | "Goal Failed" |
| Gamification Goal | `gamification.goal` | `action_reach` | "Goal Reached" |
| Gamification Goal | `gamification.goal` | `action_start` | "Start goal" |
| Gamification Goal Wizard | `gamification.goal.wizard` | `action_update_current` | "Update" |
| Set Homework Location Wizard | `homework.location.wizard` | `set_employee_location` | "Set Location" |
| Bank Account Allocation Wizard | `hr.bank.account.allocation.wizard` | `action_save` | "Save" |
| Department | `hr.department` | `action_employee_from_department` | "Employees" / `total_employee` / "`total_employee` Employees" |
| Department | `hr.department` | `action_plan_from_department` | `plans_count` |
| Departure Wizard | `hr.departure.wizard` | `action_register_departure` | "Apply" |
| Employee | `hr.employee` | `action_create_user` | "Create User" |
| Employee | `hr.employee` | `action_open_allocation_wizard` | "Salary Allocation" |
| Employee | `hr.employee` | `action_open_courses` | `courses_completion_text` |
| Employee | `hr.employee` | `action_open_employee_cars` | `employee_cars_count` |
| Employee | `hr.employee` | `action_open_last_month_attendances` | "`hours_last_month` (+ `hours_last_month_overtime` ) ( `hours_last_month_overtime` ) Monthly Hours" |
| Employee | `hr.employee` | `action_open_versions` | "`show_leaves` `is_absent` `hr_icon_display` `allocation_remaining_display` / `allocation_display` Days Time Off" / "History `versions_count`" / "`related_partners_count` Contacts" / `employee_cars_count` / "`has_subscribed_courses` `courses_completion_text`" / "`attendance_state` `hours_last_month` `display_attendances` `hours_last_month` (+ `hours_last_month_overtime` ) ( `hours_last_month_overtime` ) Monthly Hours `hours_last_month` (+ `hours_last_month_overtime` ) ( `hours_last_month_overtime` ) Monthly Hours" / "`has_work_entries` Work Entries" / "`has_timesheet` Timesheets" / `equipment_count` |
| Employee | `hr.employee` | `action_open_work_entries` | "Work Entries" |
| Employee | `hr.employee` | `action_related_contacts` | "`related_partners_count` Contacts" |
| Employee | `hr.employee` | `action_time_off_dashboard` | "`allocation_remaining_display` / `allocation_display` Days Time Off" |
| Employee | `hr.employee` | `action_timesheet_from_employee` | "Timesheets" |
| Employee | `hr.employee` | `action_toggle_primary_bank_account_trust` | "Trust Bank Account" / "Untrust Bank Account" |
| Employee | `hr.employee` | `open_barcode_scanner` | "Read a badge" |
| Print Resume | `hr.employee.cv.wizard` | `action_validate` | "Print" |
| Employee Delete Wizard | `hr.employee.delete.wizard` | `action_archive` | "Archive Employees" |
| Employee Delete Wizard | `hr.employee.delete.wizard` | `action_confirm_delete` | "Ok" |
| Employee Delete Wizard | `hr.employee.delete.wizard` | `action_open_timesheets` | "See Timesheets" |
| Public Employee | `hr.employee.public` | `action_open_courses` | `courses_completion_text` |
| Public Employee | `hr.employee.public` | `action_open_last_month_attendances` | "`hours_last_month` (+ `hours_last_month_overtime` ) ( `hours_last_month_overtime` ) Monthly Hours" |
| Public Employee | `hr.employee.public` | `action_open_time_off_calendar` | "Time Off" |
| Public Employee | `hr.employee.public` | `action_timesheet_from_employee` | "Timesheets" |
| Skill level for employee | `hr.employee.skill` | `open_hr_employee_skill_modal` | "New" |
| Job Position | `hr.job` | `action_open_activities` | `activity_count` |
| Job Position | `hr.job` | `action_open_attachments` | `documents_count` |
| Job Position | `hr.job` | `action_open_employees` | `employee_count` |
| Talent Pool | `hr.talent.pool` | `action_talent_pool_add_talents` | "New Talent" |
| Version | `hr.version` | `action_open_version_form_view` | "View" |
| Contract Template Wizard | `hr.version.wizard` | `action_load_template` | "Load" |

### 10.15 Identity and access

16 operations on 3 entities. Preconditions, guards, refusal messages and side effects: [`../domains/identity-and-access/`](../domains/identity-and-access/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Access Groups | `res.groups` | `action_show_all_users` | `all_users_count` |
| User | `res.users` | `action_change_password_wizard` | "Change password" |
| User | `res.users` | `action_create_employee` | "Create employee" |
| User | `res.users` | `action_create_passkey` | "Add Passkey" |
| User | `res.users` | `action_karma_report` | `karma` |
| User | `res.users` | `action_open_employees` | `employee_count` |
| User | `res.users` | `action_related_contact` | "Contact" |
| User | `res.users` | `action_reset_password` | "Send an Invitation Email" / "Send Password Reset" |
| User | `res.users` | `action_revoke_all_devices` | "Log out from all devices" |
| User | `res.users` | `action_show_accesses` | `accesses_count` |
| User | `res.users` | `action_show_groups` | `groups_count` |
| User | `res.users` | `action_show_rules` | `rules_count` |
| User | `res.users` | `action_totp_disable` | "Disable" |
| User | `res.users` | `action_totp_enable_wizard` | "Enable 2FA" |
| User | `res.users` | `action_totp_invite` | "Invite to use 2FA" |
| Password Check Wizard | `res.users.identitycheck` | `action_use_password` | "Use password" |

### 10.16 Inventory operations

87 operations on 26 entities. Preconditions, guards, refusal messages and side effects: [`../domains/inventory-operations/`](../domains/inventory-operations/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Confirm Expiry | `expiry.picking.confirmation` | `confirm_produce` | "Confirm" |
| Confirm Expiry | `expiry.picking.confirmation` | `confirm_workorder` | "Confirm" |
| Inventory Adjustment Reference / Reason | `stock.inventory.adjustment.name` | `action_apply` | "Update Quantities" |
| Conflict in Inventory | `stock.inventory.conflict` | `action_keep_counted_quantity` | "Keep Counted Quantity" |
| Conflict in Inventory | `stock.inventory.conflict` | `action_keep_difference` | "Keep Difference" |
| Inventory Adjustment Warning | `stock.inventory.warning` | `action_reset` | "Continue" |
| Inventory Adjustment Warning | `stock.inventory.warning` | `action_set` | "Continue" |
| Inventory Locations | `stock.location` | `action_view_equipments_records` | "`equipment_count` Equipments" |
| Lot/Serial | `stock.lot` | `action_lot_open_quants` | "Location" |
| Lot/Serial | `stock.lot` | `action_lot_open_repairs` | "To Do: `in_repair_count` Done: `repaired_count`" |
| Lot/Serial | `stock.lot` | `action_lot_open_transfers` | `delivery_count` |
| Lot/Serial | `stock.lot` | `action_view_po` | "`purchase_order_count` Purchases" |
| Lot/Serial | `stock.lot` | `action_view_ro` | "Repair Parts: `repair_part_count`" |
| Lot/Serial | `stock.lot` | `action_view_so` | "`sale_order_count` Sales" |
| Stock Move | `stock.move` | `action_show_details` | "Details" |
| Product Moves (Stock Move Line) | `stock.move.line` | `action_open_add_to_wave` | "Add to Wave" |
| Product Moves (Stock Move Line) | `stock.move.line` | `action_put_in_pack` | "Put in Pack" |
| Snooze Orderpoint | `stock.orderpoint.snooze` | `action_snooze` | "Snooze" |
| Package | `stock.package` | `action_put_in_pack` | "Put in Pack" |
| Package | `stock.package` | `action_remove_package` | "Remove" |
| Package | `stock.package` | `action_view_picking` | "Package Transfers" |
| Stock Package Destination | `stock.package.destination` | `action_done` | "Confirm" |
| Stock Package History | `stock.package.history` | `action_show_package` | "View" |
| Transfer | `stock.picking` | `action_assign` | "Check Availability" |
| Transfer | `stock.picking` | `action_cancel` | "Cancel" |
| Transfer | `stock.picking` | `action_confirm` | "Mark as Todo" |
| Transfer | `stock.picking` | `action_detailed_operations` | "Moves" |
| Transfer | `stock.picking` | `action_generate_l10n_tr_edispatch_xml` | "Generate GİB e-Dispatch (XML)" |
| Transfer | `stock.picking` | `action_l10n_in_ewaybill_create` | "Create e-Waybill / Challan" |
| Transfer | `stock.picking` | `action_l10n_ro_edi_stock_fetch_status` | "Fetch Status" |
| Transfer | `stock.picking` | `action_l10n_ro_edi_stock_send_etransport` | "Send eTransport" / "Amend eTransport" |
| Transfer | `stock.picking` | `action_next_transfer` | "Next Transfer" |
| Transfer | `stock.picking` | `action_open_l10n_in_ewaybill` | "e-Waybill / Challan" |
| Transfer | `stock.picking` | `action_picking_move_tree` | "Operations" |
| Transfer | `stock.picking` | `action_put_in_pack` | "Put in Pack" |
| Transfer | `stock.picking` | `action_see_move_scrap` | "Scraps" |
| Transfer | `stock.picking` | `action_see_package_histories` | "Packages" |
| Transfer | `stock.picking` | `action_see_packages` | "Packages" |
| Transfer | `stock.picking` | `action_see_returns` | `return_count` |
| Transfer | `stock.picking` | `action_show_details` | "Details" |
| Transfer | `stock.picking` | `action_show_subcontract_details` | "Subcontracting Productions" / "Register components for subcontracted product" |
| Transfer | `stock.picking` | `action_view_batch` | "Batch" |
| Transfer | `stock.picking` | `action_view_mrp_production` | "`production_count` Manufacturing" |
| Transfer | `stock.picking` | `action_view_reception_report` | "Allocation" |
| Transfer | `stock.picking` | `action_view_repairs` | `nbr_repairs` |
| Transfer | `stock.picking` | `action_view_subcontracting_source_purchase` | "`subcontracting_source_purchase_count` Source PO" |
| Transfer | `stock.picking` | `button_validate` | "Validate" |
| Transfer | `stock.picking` | `do_print_picking` | "Print" |
| Transfer | `stock.picking` | `do_unreserve` | "Unreserve" |
| Transfer | `stock.picking` | `open_website_url` | "Tracking" |
| Transfer | `stock.picking` | `print_return_label` | "Print Return Label" |
| Transfer | `stock.picking` | `send_to_shipper` | "Send to Shipper" |
| Batch Transfer | `stock.picking.batch` | `action_assign` | "Check Availability" |
| Batch Transfer | `stock.picking.batch` | `action_batch_detailed_operations` | "Moves" |
| Batch Transfer | `stock.picking.batch` | `action_cancel` | "Cancel" |
| Batch Transfer | `stock.picking.batch` | `action_confirm` | "Confirm" |
| Batch Transfer | `stock.picking.batch` | `action_done` | "Validate" |
| Batch Transfer | `stock.picking.batch` | `action_l10n_ro_edi_stock_fetch_status` | "Fetch Status" |
| Batch Transfer | `stock.picking.batch` | `action_l10n_ro_edi_stock_send_etransport` | "Send eTransport" / "Amend eTransport" |
| Batch Transfer | `stock.picking.batch` | `action_open_label_layout` | "Print Labels" |
| Batch Transfer | `stock.picking.batch` | `action_print` | "Print" |
| Batch Transfer | `stock.picking.batch` | `action_put_in_pack` | "Put in Pack" |
| Batch Transfer | `stock.picking.batch` | `action_see_packages` | "Packages" |
| Batch Transfer | `stock.picking.batch` | `action_view_reception_report` | "Allocation" |
| Picking Type | `stock.picking.type` | `action_batch` | "`count_picking_batch` Batches" |
| Put In Pack Wizard | `stock.put.in.pack` | `action_put_in_pack` | "Put in Pack" |
| Quants | `stock.quant` | `action_apply_all` | "Apply All" |
| Quants | `stock.quant` | `action_apply_inventory` | "Apply" |
| Quants | `stock.quant` | `action_clear_inventory_quantity` | "Clear" |
| Quants | `stock.quant` | `action_inventory_history` | "History" |
| Quants | `stock.quant` | `action_reset` | "Clear" |
| Quants | `stock.quant` | `action_stock_quant_relocate` | "Relocate" |
| Quants | `stock.quant` | `action_view_orderpoints` | "Replenishment" |
| Quants | `stock.quant` | `action_view_stock_moves` | "History" |
| Stock Quantity Relocation | `stock.quant.relocate` | `action_relocate_quants` | "Confirm" |
| Stock Quantity History | `stock.quantity.history` | `open_at_date` | "Confirm" |
| Stock Request an Inventory Count | `stock.request.count` | `action_request_count` | "Confirm" |
| Return Picking | `stock.return.picking` | `action_create_exchanges` | "Return for Exchange" |
| Return Picking | `stock.return.picking` | `action_create_returns` | "Return" |
| Return Picking | `stock.return.picking` | `action_create_returns_all` | "Return All" |
| Stock Rules report | `stock.rules.report` | `print_report` | "Overview" |
| Scrap | `stock.scrap` | `action_get_stock_move_lines` | "Product Moves" |
| Scrap | `stock.scrap` | `action_get_stock_picking` | "Stock Operation" |
| Scrap | `stock.scrap` | `action_validate` | "Validate" / "Scrap Products" |
| Warehouse | `stock.warehouse` | `action_view_all_routes` | "Routes" |
| Warn Insufficient Quantity | `stock.warn.insufficient.qty` | `action_done` | "Confirm" |
| Warn Insufficient Scrap Quantity | `stock.warn.insufficient.qty.scrap` | `action_cancel` | "Discard" |

### 10.17 Inventory valuation and costing

2 operations on 1 entities. Preconditions, guards, refusal messages and side effects: [`../domains/inventory-valuation-and-costing/`](../domains/inventory-valuation-and-costing/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Stock Landed Cost | `stock.landed.cost` | `button_cancel` | "Cancel" |
| Stock Landed Cost | `stock.landed.cost` | `button_validate` | "Validate" |

### 10.18 Learning, surveys and community

34 operations on 7 entities. Preconditions, guards, refusal messages and side effects: [`../domains/learning-surveys-and-gamification/`](../domains/learning-surveys-and-gamification/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Course | `slide.channel` | `action_channel_enroll` | "Add Attendees" / `members_count` |
| Course | `slide.channel` | `action_channel_invite` | "Invite" |
| Course | `slide.channel` | `action_mass_mailing_attendees` | "Contact Attendees" |
| Course | `slide.channel` | `action_redirect_to_certified_members` | "`members_certified_count` Certified" |
| Course | `slide.channel` | `action_redirect_to_completed_members` | "`members_completed_count` Finished" |
| Course | `slide.channel` | `action_redirect_to_forum` | `forum_total_posts` |
| Course | `slide.channel` | `action_redirect_to_members` | "`members_all_count` Attendees" |
| Course | `slide.channel` | `action_view_ratings` | "`rating_avg_stars` /5 `rating_count` Reviews" |
| Course | `slide.channel` | `action_view_sales` | `product_sale_revenues` |
| Course | `slide.channel` | `action_view_slides` | `total_slides` |
| Course | `slide.channel` | `open_website_url` | "View course" |
| Channel Invitation Wizard | `slide.channel.invite` | `action_invite` | "Send" |
| Channel / Partners (Members) | `slide.channel.partner` | `action_archive` | "Archive" |
| Channel / Partners (Members) | `slide.channel.partner` | `action_unarchive` | "Unarchive" |
| Slides | `slide.slide` | `action_view_embeds` | "`embed_count` Embed Views" |
| Survey Invitation Wizard | `survey.invite` | `action_invite` | "Send" |
| Survey | `survey.survey` | `action_archive` | "True" / "Close" |
| Survey | `survey.survey` | `action_end_session` | "Close Live Session" / "End Live Session" |
| Survey | `survey.survey` | `action_open_session_manager` | "Open Session Manager" |
| Survey | `survey.survey` | `action_result_survey` | "See results" |
| Survey | `survey.survey` | `action_send_survey` | "True" / "Share" |
| Survey | `survey.survey` | `action_start_session` | "True" / "Create Live Session" / "Start Live Session" |
| Survey | `survey.survey` | `action_survey_preview_certification_template` | "Preview" |
| Survey | `survey.survey` | `action_survey_see_leads` | `lead_count` |
| Survey | `survey.survey` | `action_survey_user_input` | `answer_count` |
| Survey | `survey.survey` | `action_survey_user_input_certified` | `success_count` |
| Survey | `survey.survey` | `action_survey_user_input_completed` | `answer_done_count` |
| Survey | `survey.survey` | `action_survey_view_slide_channels` | `slide_channel_count` |
| Survey | `survey.survey` | `action_test_survey` | "Test" |
| Survey | `survey.survey` | `action_unarchive` | "Reopen" |
| Survey User Input | `survey.user_input` | `action_print_answers` | "Print" |
| Survey User Input | `survey.user_input` | `action_redirect_lead` | "Lead" |
| Survey User Input | `survey.user_input` | `action_redirect_to_attempts` | `attempts_count` |
| Survey User Input | `survey.user_input` | `action_resend` | "Resend Invitation" |

### 10.19 Loyalty and promotions

7 operations on 4 entities. Preconditions, guards, refusal messages and side effects: [`../domains/loyalty-and-promotions/`](../domains/loyalty-and-promotions/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | `action_generate_short_link` | "Generate Short Link" |
| Loyalty Coupon | `loyalty.card` | `action_coupon_send` | "Send" |
| Loyalty Coupon | `loyalty.card` | `action_coupon_share` | "Share" |
| Loyalty Coupon | `loyalty.card` | `action_loyalty_update_balance` | `points_display` |
| Update Loyalty Card Points | `loyalty.card.update.balance` | `action_update_card_point` | "Confirm" |
| Loyalty Program | `loyalty.program` | `action_open_loyalty_cards` | "`coupon_count` Coupons Loyalty Cards Promos Discount Gift Cards eWallets" |
| Loyalty Program | `loyalty.program` | `action_program_share` | "Share" |

### 10.20 Manufacturing

58 operations on 12 entities. Preconditions, guards, refusal messages and side effects: [`../domains/manufacturing/`](../domains/manufacturing/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Bill of Material | `mrp.bom` | `action_add_from_catalog` | "Catalog" |
| Bill of Material | `mrp.bom` | `action_compute_bom_days` | "Compute" |
| Bill of Material | `mrp.bom` | `action_copy_existing_operations` | "Copy Existing Operations" |
| Bill of Material | `mrp.bom` | `action_open_operation_form` | "Add Operation" |
| Bill of Material | `mrp.bom` | `action_see_attachments` | "Product Attachments" |
| Bill of Material | `mrp.bom` | `action_set_bom_on_orderpoint` | "Set as Bill of Materials" |
| Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) | `mrp.consumption.warning` | `action_cancel` | "Discard" |
| Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) | `mrp.consumption.warning` | `action_confirm` | "Force" / "Confirm" |
| Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) | `mrp.consumption.warning` | `action_set_qty` | "Set Quantities & Validate" |
| Manufacturing Order | `mrp.production` | `action_add_from_catalog_byproduct` | "Catalog" |
| Manufacturing Order | `mrp.production` | `action_add_from_catalog_raw` | "Catalog" |
| Manufacturing Order | `mrp.production` | `action_assign` | "Check availability" |
| Manufacturing Order | `mrp.production` | `action_cancel` | "Cancel" |
| Manufacturing Order | `mrp.production` | `action_clear_lot_producing_ids` | "Clear" |
| Manufacturing Order | `mrp.production` | `action_confirm` | "Confirm" |
| Manufacturing Order | `mrp.production` | `action_generate_bom` | "Generate BOM" |
| Manufacturing Order | `mrp.production` | `action_generate_serial` | "Generate Serial" / "Generate Lot" |
| Manufacturing Order | `mrp.production` | `action_open_project` | "Project" |
| Manufacturing Order | `mrp.production` | `action_product_forecast_report` | "Forecast Report" |
| Manufacturing Order | `mrp.production` | `action_see_move_scrap` | "`scrap_count` Scraps" |
| Manufacturing Order | `mrp.production` | `action_show_details` | "Details" / "Show Details" |
| Manufacturing Order | `mrp.production` | `action_start` | "Start" |
| Manufacturing Order | `mrp.production` | `action_update_bom` | "Update BoM" |
| Manufacturing Order | `mrp.production` | `action_view_mo_delivery` | `delivery_count` |
| Manufacturing Order | `mrp.production` | `action_view_move_wip` | "`wip_move_count` WIP" |
| Manufacturing Order | `mrp.production` | `action_view_mrp_production_backorders` | "`mrp_production_backorder_count` Backorders" |
| Manufacturing Order | `mrp.production` | `action_view_mrp_production_childs` | "`mrp_production_child_count` Child MO" |
| Manufacturing Order | `mrp.production` | `action_view_mrp_production_sources` | "`mrp_production_source_count` Source MO" |
| Manufacturing Order | `mrp.production` | `action_view_mrp_production_unbuilds` | "`unbuild_count` Unbuilds" |
| Manufacturing Order | `mrp.production` | `action_view_purchase_orders` | "`purchase_order_count` Purchases" |
| Manufacturing Order | `mrp.production` | `action_view_reception_report` | "Allocation" |
| Manufacturing Order | `mrp.production` | `action_view_repair_orders` | `repair_count` |
| Manufacturing Order | `mrp.production` | `action_view_sale_orders` | "`sale_order_count` Sale" |
| Manufacturing Order | `mrp.production` | `action_view_serial_numbers` | "`serial_numbers_count` Serial Numbers" |
| Manufacturing Order | `mrp.production` | `button_mark_done` | "Produce" / "Produce All" |
| Manufacturing Order | `mrp.production` | `button_plan` | "Plan" |
| Manufacturing Order | `mrp.production` | `button_unbuild` | "Unbuild" |
| Manufacturing Order | `mrp.production` | `button_unplan` | "Unplan" |
| Manufacturing Order | `mrp.production` | `do_unreserve` | "Unreserve" |
| Wizard to mark as done or create back order | `mrp.production.backorder` | `action_backorder` | "Create backorder" / "Validate" |
| Wizard to mark as done or create back order | `mrp.production.backorder` | `action_close_mo` | "No Backorder" |
| Assign serial numbers to production order | `mrp.production.serials` | `action_apply` | "Apply" |
| Assign serial numbers to production order | `mrp.production.serials` | `action_generate_serial_numbers` | "Generate" |
| Assign serial numbers to production order | `mrp.production.serials` | `action_split_and_assign_serials` | "Prepare MO" |
| Wizard to Split a Production | `mrp.production.split` | `action_return_to_list` | "Discard" |
| Wizard to Split a Production | `mrp.production.split` | `action_split` | "Split" |
| Wizard to Split Multiple Productions | `mrp.production.split.multi` | `action_prepare_split` | "Split Production" |
| Work Center Usage | `mrp.routing.workcenter` | `action_open_operation_form` | "Add a line" |
| Unbuild Order | `mrp.unbuild` | `action_validate` | "Unbuild" |
| Work Center | `mrp.workcenter` | `action_show_operations` | "Operations" |
| Work Center | `mrp.workcenter` | `action_work_order` | "WORK ORDERS" |
| Work Center | `mrp.workcenter` | `action_work_order_alternatives` | "PLAN ORDERS" |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `button_block` | "Block" |
| Work Order | `mrp.workorder` | `action_open_wizard` | "View WorkOrder" |
| Work Order | `mrp.workorder` | `action_see_move_scrap` | "`scrap_count` Scraps" |
| Work Order | `mrp.workorder` | `button_finish` | "Done" |
| Work Order | `mrp.workorder` | `button_pending` | "Pause" |
| Work Order | `mrp.workorder` | `button_start` | "Start" |

### 10.21 Marketing and mass mailing

79 operations on 21 entities. Preconditions, guards, refusal messages and side effects: [`../domains/marketing-and-mass-mailing/`](../domains/marketing-and-mass-mailing/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Marketing Card Campaign | `card.campaign` | `action_preview` | "Preview" |
| Marketing Card Campaign | `card.campaign` | `action_share` | "Send" |
| Marketing Card Campaign | `card.campaign` | `action_view_cards` | "Cards `card_count`" |
| Marketing Card Campaign | `card.campaign` | `action_view_cards_clicked` | "Opened `card_click_count`" |
| Marketing Card Campaign | `card.campaign` | `action_view_cards_shared` | "Shared `card_share_count`" |
| Marketing Card Campaign | `card.campaign` | `action_view_mailings` | "Mailings `mailing_count`" |
| Confirm Stock text message | `confirm.stock.sms` | `send_sms` | "Confirm" |
| Link Tracker | `link.tracker` | `action_view_statistics` | `count` |
| Link Tracker | `link.tracker` | `action_visit_page` | "Visit Page" |
| Link Tracker | `link.tracker` | `action_visit_page_statistics` | "Statistics" |
| Mailing Contact | `mailing.contact` | `action_add_to_mailing_list` | "Add to List" |
| Mailing Contact | `mailing.contact` | `action_import` | "Import" |
| Mailing Contact Import | `mailing.contact.import` | `action_import` | "Import" |
| Mailing Contact Import | `mailing.contact.import` | `action_open_base_import` | "Upload a file" |
| Add Contacts to Mailing List | `mailing.contact.to.list` | `action_add_contacts` | "Add" |
| Add Contacts to Mailing List | `mailing.contact.to.list` | `action_add_contacts_and_send_mailing` | "Add and Send Mailing" |
| Mailing List | `mailing.list` | `action_open_import` | "Import Contacts" |
| Mailing List | `mailing.list` | `action_send_mailing` | "Send Mailing" |
| Mailing List | `mailing.list` | `action_send_mailing_sms` | "Send SMS" |
| Mailing List | `mailing.list` | `action_view_contacts` | `contact_count` / "Total Contacts" |
| Mailing List | `mailing.list` | `action_view_contacts_blacklisted` | `contact_pct_blacklisted` |
| Mailing List | `mailing.list` | `action_view_contacts_bouncing` | `contact_pct_bounce` |
| Mailing List | `mailing.list` | `action_view_contacts_opt_out` | `contact_pct_opt_out` |
| Mailing List | `mailing.list` | `action_view_mailings` | `mailing_count` |
| Merge Mass Mailing List | `mailing.list.merge` | `action_mailing_lists_merge` | "Merge" |
| Mass Mailing | `mailing.mailing` | `action_buy_sms_credits` | "It appears you don't have enough IAP credits. Click here to buy credits." / "It appears your SMS account is not registered. Click here to set up your account." |
| Mass Mailing | `mailing.mailing` | `action_cancel` | "Cancel" |
| Mass Mailing | `mailing.mailing` | `action_compare_versions` | "Compare Version" |
| Mass Mailing | `mailing.mailing` | `action_duplicate` | "Duplicate" / "Create an Alternative" / "Create an Alternative Version" |
| Mass Mailing | `mailing.mailing` | `action_launch` | "Send" |
| Mass Mailing | `mailing.mailing` | `action_put_in_queue` | "Send" |
| Mass Mailing | `mailing.mailing` | `action_redirect_to_invoiced` | `sale_invoiced_amount` |
| Mass Mailing | `mailing.mailing` | `action_redirect_to_leads_and_opportunities` | "`use_leads` `crm_lead_count` Leads Opportunities" |
| Mass Mailing | `mailing.mailing` | `action_redirect_to_quotations` | `sale_quotation_count` |
| Mass Mailing | `mailing.mailing` | `action_reload` | "Refresh" |
| Mass Mailing | `mailing.mailing` | `action_remove_favorite` | "Remove from Templates" |
| Mass Mailing | `mailing.mailing` | `action_retry_failed` | "Retry" |
| Mass Mailing | `mailing.mailing` | `action_schedule` | "Schedule" |
| Mass Mailing | `mailing.mailing` | `action_select_as_winner` | "Send this as winner" |
| Mass Mailing | `mailing.mailing` | `action_send_mail` | "Send Now" |
| Mass Mailing | `mailing.mailing` | `action_send_winner_mailing` | "Send Winner Now" |
| Mass Mailing | `mailing.mailing` | `action_set_favorite` | "Add to Templates" |
| Mass Mailing | `mailing.mailing` | `action_test` | "Test" |
| Mass Mailing | `mailing.mailing` | `action_update_cards` | "Update `card_requires_sync_count` Cards" |
| Mass Mailing | `mailing.mailing` | `action_view_bounced` | `bounced_ratio` |
| Mass Mailing | `mailing.mailing` | `action_view_clicked` | `clicks_ratio` |
| Mass Mailing | `mailing.mailing` | `action_view_delivered` | `received_ratio` |
| Mass Mailing | `mailing.mailing` | `action_view_link_trackers` | `link_trackers_count` |
| Mass Mailing | `mailing.mailing` | `action_view_mailing_contacts` | "Add Mailing Contacts" |
| Mass Mailing | `mailing.mailing` | `action_view_opened` | `opened_ratio` |
| Mass Mailing | `mailing.mailing` | `action_view_replied` | `replied_ratio` |
| Mass Mailing | `mailing.mailing` | `action_view_traces_canceled` | "`canceled` emails have been cancelled and will not be sent." |
| Mass Mailing | `mailing.mailing` | `action_view_traces_failed` | "`failed` emails could not be sent." |
| Mass Mailing | `mailing.mailing` | `action_view_traces_process` | "`process` emails are being processed." |
| Mass Mailing | `mailing.mailing` | `action_view_traces_scheduled` | "`scheduled` emails are in queue and will be sent soon." |
| Mass Mailing | `mailing.mailing` | `action_view_traces_sent` | "`sent` emails have been sent." |
| schedule a mailing | `mailing.mailing.schedule.date` | `action_schedule_date` | "Schedule" |
| Sample Mail Wizard | `mailing.mailing.test` | `send_mail_test` | "Send test" |
| Test text message Mailing | `mailing.sms.test` | `action_send_sms` | "Send Test" |
| Mailing Statistics | `mailing.trace` | `action_view_contact` | "Open Recipient" |
| text message Account Verification Code Wizard | `sms.account.code` | `action_register` | "Register" |
| text message Account Registration Phone Number Wizard | `sms.account.phone` | `action_send_verification_code` | "Send verification code" |
| text message Account Sender Name Wizard | `sms.account.sender` | `action_set_sender_name` | "Set sender name" |
| Send text message Wizard | `sms.composer` | `action_send_sms` | "Send" / "Put in queue" |
| Send text message Wizard | `sms.composer` | `action_send_sms_mass_now` | "Send now" |
| Outgoing text message | `sms.sms` | `action_set_canceled` | "Cancel" |
| Outgoing text message | `sms.sms` | `action_set_outgoing` | "Retry" |
| text message Templates | `sms.template` | `action_create_sidebar_action` | "Add Context Action" |
| text message Templates | `sms.template` | `action_unlink_sidebar_action` | "Remove Context Action" |
| text message Twilio Connection Wizard | `sms.twilio.account.manage` | `action_reload_numbers` | "Reload Numbers from Twilio" |
| text message Twilio Connection Wizard | `sms.twilio.account.manage` | `action_save` | "Update Account" |
| text message Twilio Connection Wizard | `sms.twilio.account.manage` | `action_send_test` | "Send test SMS" |
| text message Twilio Connection Wizard | `sms.twilio.account.manage` | `action_unlink` | "Delete" |
| campaign tracking parameter Campaign | `utm.campaign` | `action_create_mass_sms` | "Send SMS" |
| campaign tracking parameter Campaign | `utm.campaign` | `action_duplicate` | "Duplicate" |
| campaign tracking parameter Campaign | `utm.campaign` | `action_redirect_to_invoiced` | `invoiced_amount` |
| campaign tracking parameter Campaign | `utm.campaign` | `action_redirect_to_leads_opportunities` | "`use_leads` `crm_lead_count` Leads Opportunities" |
| campaign tracking parameter Campaign | `utm.campaign` | `action_redirect_to_mailing_sms` | `mailing_sms_count` |
| campaign tracking parameter Campaign | `utm.campaign` | `action_redirect_to_quotations` | `quotation_count` |

### 10.22 Meal ordering

7 operations on 1 entities. Preconditions, guards, refusal messages and side effects: [`../domains/lunch-ordering/`](../domains/lunch-ordering/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Lunch Order | `lunch.order` | `action_cancel` | "Cancel" |
| Lunch Order | `lunch.order` | `action_confirm` | "Receive" / "Confirm" |
| Lunch Order | `lunch.order` | `action_confirm_orders` | "Confirm Orders" |
| Lunch Order | `lunch.order` | `action_notify` | "Send Notification" |
| Lunch Order | `lunch.order` | `action_reorder` | "Re-order" |
| Lunch Order | `lunch.order` | `action_reset` | "Reset" |
| Lunch Order | `lunch.order` | `action_send_orders` | "Send Orders" |

### 10.23 Messaging and activities

44 operations on 19 entities. Preconditions, guards, refusal messages and side effects: [`../domains/messaging-and-activities/`](../domains/messaging-and-activities/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Chatbot Script | `chatbot.script` | `action_test_script` | "Test" |
| Chatbot Script | `chatbot.script` | `action_view_leads` | `lead_count` |
| Chatbot Script | `chatbot.script` | `action_view_livechat_channels` | `livechat_channel_count` |
| Discussion Channel | `discuss.channel` | `action_unfollow` | "Leave" |
| Mail RTC session | `discuss.channel.rtc.session` | `action_disconnect` | "Disconnect" |
| Incoming Mail Server | `fetchmail.server` | `button_confirm_login` | "Test & Confirm" |
| Incoming Mail Server | `fetchmail.server` | `open_google_gmail_uri` | "Connect your Gmail account" |
| Incoming Mail Server | `fetchmail.server` | `open_microsoft_outlook_uri` | "Connect your Outlook account" |
| Incoming Mail Server | `fetchmail.server` | `set_draft` | "Reset Confirmation" |
| Livechat Channel | `im_livechat.channel` | `action_join` | "Join" / "Join Channel" |
| Livechat Channel | `im_livechat.channel` | `action_quit` | "Leave" / "Leave Channel" |
| Livechat Channel | `im_livechat.channel` | `action_view_chatbot_scripts` | `chatbot_script_count` |
| Activity | `mail.activity` | `action_cancel` | "Cancel" |
| Activity | `mail.activity` | `action_close_dialog` | "Schedule" / "Save" |
| Activity | `mail.activity` | `action_create_calendar_event` | "Schedule" |
| Activity | `mail.activity` | `action_done` | "Mark Done" / "Done" |
| Activity | `mail.activity` | `action_done_redirect_to_other` | "Mark Done" |
| Activity | `mail.activity` | `action_open_document` | "Open Document" |
| Activity | `mail.activity` | `action_reschedule_nextweek` | "Next Week" |
| Activity | `mail.activity` | `action_reschedule_today` | "Today" |
| Activity | `mail.activity` | `action_reschedule_tomorrow` | "Tomorrow" |
| Activity schedule plan Wizard | `mail.activity.schedule` | `action_create_calendar_event` | "Schedule" |
| Activity schedule plan Wizard | `mail.activity.schedule` | `action_schedule_activities` | "Save" |
| Activity schedule plan Wizard | `mail.activity.schedule` | `action_schedule_activities_done` | "Mark Done" |
| Activity schedule plan Wizard | `mail.activity.schedule` | `action_schedule_plan` | "Schedule" |
| Email Aliases | `mail.alias` | `open_document` | "Open Document" |
| Email Aliases | `mail.alias` | `open_parent_document` | "Open Parent Document" / "Open Owner" |
| Mail Blacklist | `mail.blacklist` | `action_add` | "Blacklist" |
| Remove email from blacklist wizard | `mail.blacklist.remove` | `action_unblacklist_apply` | "Remove address from blacklist" |
| Email composition wizard | `mail.compose.message` | `action_schedule_message` | "Schedule" |
| Email composition wizard | `mail.compose.message` | `action_send_mail` | "Send" / "Log" / "Send Mass Mailing" |
| Mail Group | `mail.group` | `action_go_to_website` | "Go to Website" |
| Mail Group | `mail.group` | `action_join` | "Join" |
| Mail Group | `mail.group` | `action_leave` | "Leave" |
| Mailing List Message | `mail.group.message` | `action_moderate_accept` | "Accept" / "Send" |
| Mailing List Message | `mail.group.message` | `action_moderate_allow` | "Whitelist" |
| Reject Group Message | `mail.group.message.reject` | `action_send_mail` | "Reject Silently" / "Send & Reject" / "Ban" / "Send & Ban" |
| Outgoing Mails | `mail.mail` | `action_open_document` | "Open Document" |
| Outgoing Mails | `mail.mail` | `action_retry` | "Retry" |
| Outgoing Mails | `mail.mail` | `action_send_and_close` | "Send & Close" |
| Message | `mail.message` | `action_open_document` | "Open Document" |
| Email Templates | `mail.template` | `action_open_mail_preview` | "Preview" |
| Phone Blacklist | `phone.blacklist` | `action_add` | "Blacklist" |
| Remove phone from blacklist | `phone.blacklist.remove` | `action_unblacklist_apply` | "Remove phone from blacklist" |

### 10.24 Payment providers

23 operations on 4 entities. Preconditions, guards, refusal messages and side effects: [`../domains/payment-providers/`](../domains/payment-providers/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Payment Capture Wizard | `payment.capture.wizard` | `action_capture` | "Capture" |
| Payment Provider | `payment.provider` | `action_paypal_create_webhook` | "Generate your webhook" |
| Payment Provider | `payment.provider` | `action_razorpay_create_webhook` | "Generate your webhook" |
| Payment Provider | `payment.provider` | `action_recompute_pending_msg` | " Reload Pending Message" |
| Payment Provider | `payment.provider` | `action_reset_credentials` | "Reset Your Razorpay Account" / "Disconnect" / "Disconnect Your Mercado Pago Account" |
| Payment Provider | `payment.provider` | `action_start_onboarding` | "Connect" / "Connect Stripe" |
| Payment Provider | `payment.provider` | `action_stripe_create_webhook` | "Generate your webhook" |
| Payment Provider | `payment.provider` | `action_stripe_verify_apple_pay_domain` | "Enable Apple Pay" |
| Payment Provider | `payment.provider` | `action_sync_paymob_payment_methods` | "Synchronize With Paymob" |
| Payment Provider | `payment.provider` | `action_toggle_is_published` | "Published" / "Unpublished" |
| Payment Provider | `payment.provider` | `action_update_merchant_details` | "Generate Client Key" / "Set Account Currency" |
| Payment Provider | `payment.provider` | `button_immediate_install` | "Install" |
| Payment Refund Wizard | `payment.refund.wizard` | `action_refund` | "Refund" |
| Payment Transaction | `payment.transaction` | `action_capture` | "Capture Transaction" |
| Payment Transaction | `payment.transaction` | `action_demo_set_canceled` | "Cancel" |
| Payment Transaction | `payment.transaction` | `action_demo_set_done` | "Authorize" / "Confirm" |
| Payment Transaction | `payment.transaction` | `action_demo_set_error` | "Set to Error" |
| Payment Transaction | `payment.transaction` | `action_post_process` | "Post-process" |
| Payment Transaction | `payment.transaction` | `action_view_invoices` | `invoices_count` |
| Payment Transaction | `payment.transaction` | `action_view_pos_order` | `pos_order_id` |
| Payment Transaction | `payment.transaction` | `action_view_refunds` | `invoices_count` / "`pos_order_id` `pos_order_id`" / `refunds_count` |
| Payment Transaction | `payment.transaction` | `action_view_sales_orders` | `sale_order_ids_nbr` |
| Payment Transaction | `payment.transaction` | `action_void` | "Void Transaction" |

### 10.25 Payments and bank reconciliation

24 operations on 4 entities. Preconditions, guards, refusal messages and side effects: [`../domains/payments-and-bank-reconciliation/`](../domains/payments-and-bank-reconciliation/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Payments | `account.payment` | `action_cancel` | "Cancel" |
| Payments | `account.payment` | `action_draft` | "Reset to Draft" |
| Payments | `account.payment` | `action_l10n_in_withholding_entries` | "TDS `l10n_in_total_withholding_amount`" |
| Payments | `account.payment` | `action_open_expense` | "Expense" |
| Payments | `account.payment` | `action_post` | "Confirm" |
| Payments | `account.payment` | `action_refund_wizard` | "Refund" |
| Payments | `account.payment` | `action_reject` | "Reject" |
| Payments | `account.payment` | `action_validate` | "Validate" |
| Payments | `account.payment` | `action_view_pos_order` | `pos_order_id` |
| Payments | `account.payment` | `action_view_refunds` | `refunds_count` |
| Payments | `account.payment` | `action_void_check` | "Void Check" |
| Payments | `account.payment` | `button_open_bills` | "`reconciled_bills_count` Bill" |
| Payments | `account.payment` | `button_open_invoices` | "`reconciled_invoices_count` Invoice Credit Note" |
| Payments | `account.payment` | `button_open_journal_entry` | "Journal Entry" |
| Payments | `account.payment` | `button_open_statement_lines` | "`reconciled_statement_lines_count` Transaction" |
| Payments | `account.payment` | `button_request_cancel` | "Request Cancel" |
| Payments | `account.payment` | `print_checks` | "Print Check" |
| Pay | `account.payment.register` | `action_create_payments` | "Create Payments" / "Create Payment" |
| Pay | `account.payment.register` | `action_open_missing_account_partners` | "View Partner(s)" |
| Pay | `account.payment.register` | `action_open_untrusted_bank_accounts` | "untrusted bank accounts" |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `action_reconcile_stat` | "Journal Entries" |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `action_set_auto_reconcile` | "Automate" |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `action_set_manual` | "Set Manual" |
| Print Pre-numbered Checks | `print.prenumbered.checks` | `print_checks` | "Print" |

### 10.26 Platform foundation

84 operations on 19 entities. Preconditions, guards, refusal messages and side effects: [`../overview/architecture.md`](../overview/architecture.md) and [`../runtime/README.md`](../runtime/README.md).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Automation Rule | `base.automation` | `action_open_scheduled_action` | "Scheduled action" |
| Automation Rule | `base.automation` | `action_rotate_webhook_uuid` | "Renew" |
| Automation Rule | `base.automation` | `action_view_webhook_logs` | "Logs" |
| Module Activation Request | `base.module.install.request` | `action_send_request` | "Request Activation" |
| Module Activation Review | `base.module.install.review` | `action_install_module` | "Install App" |
| Module Uninstall | `base.module.uninstall` | `action_uninstall` | "Uninstall" |
| Update Module | `base.module.update` | `action_module_open` | "Open Apps" |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `action_merge` | "Merge Contacts" |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `action_skip` | "Skip these contacts" |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `action_start_automatic_process` | "Merge Automatically" |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `action_start_manual_process` | "Merge with Manual Check" |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `action_update_all_process` | "Merge Automatically all process" |
| in-app purchase Account | `iap.account` | `action_buy_credits` | "Buy Credit" |
| in-app purchase Account | `iap.account` | `action_open_registration_wizard` | "Register" |
| in-app purchase Account | `iap.account` | `action_open_sender_name_wizard` | "Set Sender Name" |
| Server Actions | `ir.actions.server` | `action_open_automation` | "Automation" |
| Server Actions | `ir.actions.server` | `action_open_parent_action` | "Parent Action" |
| Server Actions | `ir.actions.server` | `action_open_scheduled_action` | "Scheduled Action" |
| Configuration Wizards | `ir.actions.todo` | `action_launch` | "Launch" |
| Configuration Wizards | `ir.actions.todo` | `action_open` | "Todo" / "Set as Todo" |
| Mail Server | `ir.mail_server` | `action_retrieve_max_email_size` | "Detect Max Limit" |
| Mail Server | `ir.mail_server` | `open_google_gmail_uri` | "Connect your Gmail account" |
| Mail Server | `ir.mail_server` | `open_microsoft_outlook_uri` | "Connect your Outlook account" |
| Module | `ir.module.module` | `action_open_install_request` | "Request Access" |
| Module | `ir.module.module` | `action_view_delivery_methods` | "Delivery Methods" |
| Module | `ir.module.module` | `button_choose_theme` | "Use this theme" |
| Module | `ir.module.module` | `button_immediate_install` | "Request Access" / "Activate" |
| Module | `ir.module.module` | `button_immediate_install_app` | "Activate" / "Upgrade" |
| Module | `ir.module.module` | `button_immediate_upgrade` | "Upgrade" |
| Module | `ir.module.module` | `button_refresh_theme` | "Update theme" |
| Module | `ir.module.module` | `button_remove_theme` | "Remove theme" |
| Module | `ir.module.module` | `button_uninstall_wizard` | "Uninstall" |
| Profiling results | `ir.profile` | `action_view_speedscope` | "View in speedscope" |
| Onboarding | `onboarding.onboarding` | `action_toggle_visibility` | "Toggle visibility" |
| Privacy Lookup Wizard | `privacy.lookup.wizard` | `action_lookup` | "Lookup" |
| Privacy Lookup Wizard | `privacy.lookup.wizard` | `action_open_lines` | `line_count` |
| Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `action_open_record` | "Open Record" |
| Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `action_unlink` | "Delete" |
| Companies | `res.company` | `action_all_company_branches` | "Branches" |
| Companies | `res.company` | `action_save_onboarding_company_data` | "Save" |
| Companies | `res.company` | `action_save_onboarding_sale_tax` | "Apply" |
| Config | `res.config` | `action_next` | "Apply" |
| Config | `res.config` | `action_skip` | "Cancel" |
| Config Settings | `res.config.settings` | `action_crm_assign_leads` | a control with no text of its own |
| Config Settings | `res.config.settings` | `action_eu_oss_tax_mapping` | "OSS Tax mapping" |
| Config Settings | `res.config.settings` | `action_l10n_my_edi_allow_processing` | "Register" |
| Config Settings | `res.config.settings` | `action_l10n_my_edi_unregister` | "Unregister" |
| Config Settings | `res.config.settings` | `action_open_abandoned_cart_mail_template` | "Customize Abandoned Email Template" |
| Config Settings | `res.config.settings` | `action_open_blocked_third_party_domains` | "Add domains to the block list" |
| Config Settings | `res.config.settings` | `action_open_cloud_storage_migration_configurations` | "Parameters" |
| Config Settings | `res.config.settings` | `action_open_nemhandel_form` | "Start sending via Nemhandel" |
| Config Settings | `res.config.settings` | `action_open_peppol_form` | "Activate Electronic Invoicing" / "Activate Peppol" |
| Config Settings | `res.config.settings` | `action_open_product_feeds` | "Manage feeds" |
| Config Settings | `res.config.settings` | `action_open_robots` | "Edit robots.txt" |
| Config Settings | `res.config.settings` | `action_open_sms_twilio_account_manage` | "Configure Twilio Account" |
| Config Settings | `res.config.settings` | `action_open_template_user` | "Default Access Rights" |
| Config Settings | `res.config.settings` | `action_pos_config_create_new` | "+ New Shop" |
| Config Settings | `res.config.settings` | `action_pos_printer_dialog` | "Add Printer" |
| Config Settings | `res.config.settings` | `action_sale_start_payment_onboarding` | "Activate `onboarding_payment_module`" |
| Config Settings | `res.config.settings` | `action_update_terms` | "Update Terms" |
| Config Settings | `res.config.settings` | `action_view_active_provider` | "Configure `active_provider_id`" |
| Config Settings | `res.config.settings` | `action_view_delivery_provider_modules` | "Find a Delivery Provider" |
| Config Settings | `res.config.settings` | `action_view_in_store_delivery_methods` | "Configure Pickup Locations" |
| Config Settings | `res.config.settings` | `action_w_payment_start_payment_onboarding` | "Activate `onboarding_payment_module`" |
| Config Settings | `res.config.settings` | `action_website_create_new` | "+ New Website" |
| Config Settings | `res.config.settings` | `button_deregister_nemhandel_participant` | "Deregister" |
| Config Settings | `res.config.settings` | `button_disconnect_this_database` | "Disconnect this database" |
| Config Settings | `res.config.settings` | `button_l10n_hr_activate_mojeracun` | "Activate" |
| Config Settings | `res.config.settings` | `button_l10n_hr_deactivate_mojeracun` | "Deactivate" |
| Config Settings | `res.config.settings` | `button_l10n_ro_edi_generate_token` | "Generate Token" |
| Config Settings | `res.config.settings` | `button_open_peppol_config_wizard` | "Advanced Configuration" |
| Config Settings | `res.config.settings` | `button_peppol_deregister` | "Disconnect French electronic invoicing" / "Disconnect" / "Disconnect Peppol" |
| Config Settings | `res.config.settings` | `button_peppol_disconnect_branch_from_parent` | "Disconnect" |
| Config Settings | `res.config.settings` | `button_peppol_register_sender_as_receiver` | "Register with the platform" |
| Config Settings | `res.config.settings` | `button_peppol_reregister` | "Complete Registration" |
| Config Settings | `res.config.settings` | `button_reconnect_this_database` | "Reconnect this database" |
| Config Settings | `res.config.settings` | `button_update_nemhandel_user_data` | "Update contact details" |
| Config Settings | `res.config.settings` | `open_company` | "Update Info" |
| Config Settings | `res.config.settings` | `open_email_layout` | "Update Mail Layout" |
| Config Settings | `res.config.settings` | `open_mail_templates` | "Review All Templates" |
| Config Settings | `res.config.settings` | `open_new_user_default_groups` | "Default Access Rights" |
| Config Settings | `res.config.settings` | `open_payment_method_form` | "Payment method" |
| Languages | `res.lang` | `action_activate_langs` | "Activate" |
| Languages | `res.lang` | `action_archive` | "Disable" |

### 10.27 Point of sale

27 operations on 7 entities. Preconditions, guards, refusal messages and side effects: [`../domains/point-of-sale/`](../domains/point-of-sale/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Point of Sale Configuration | `pos.config` | `action_close_kiosk_session` | "Close Session" |
| Point of Sale Configuration | `pos.config` | `action_open_wizard` | "Start Kiosk" / "Open Kiosk" |
| Point of Sale Configuration | `pos.config` | `open_existing_session_cb` | "Close" |
| Point of Sale Configuration | `pos.config` | `open_ui` | "Continue Selling Open Register" |
| Confirmation Wizard | `pos.confirmation.wizard` | `action_confirm` | "Confirm" |
| Multiple order invoice creation | `pos.make.invoice` | `action_create_invoices` | "Create" |
| Point of Sale Orders | `pos.order` | `action_create_invoices` | "Create Invoices" |
| Point of Sale Orders | `pos.order` | `action_pos_order_invoice` | "Invoice" |
| Point of Sale Orders | `pos.order` | `action_send_mail` | "email" |
| Point of Sale Orders | `pos.order` | `action_show_myinvois_documents` | "Consolidated Invoice" |
| Point of Sale Orders | `pos.order` | `action_stock_picking` | "`picking_count` `picking_count`" |
| Point of Sale Orders | `pos.order` | `action_view_attendee_list` | `attendee_count` |
| Point of Sale Orders | `pos.order` | `action_view_invoice` | "Invoice" |
| Point of Sale Orders | `pos.order` | `action_view_refund_orders` | `refund_orders_count` |
| Point of Sale Orders | `pos.order` | `action_view_refunded_order` | `refunded_order_id` |
| Point of Sale Orders | `pos.order` | `action_view_sale_order` | "`sale_order_count` Transferred from Sale" |
| Point of Sale Orders | `pos.order` | `button_l10n_jo_edi_pos` | "JoFotara (Jordan)" |
| Point of Sale Orders | `pos.order` | `print_event_badges` | "Print Event Badges" |
| Point of Sale Orders | `pos.order` | `print_event_tickets` | "Print Event Tickets" |
| Point of Sale Payment Methods | `pos.payment.method` | `action_stripe_key` | "Don't forget to complete Stripe connect before using this payment method." |
| Easily load a set of configuration options | `pos.preset` | `action_open_linked_config` | `count_linked_config` |
| Easily load a set of configuration options | `pos.preset` | `action_open_linked_orders` | `count_linked_orders` |
| Point of Sale Session | `pos.session` | `action_pos_session_closing_control` | "Close Session & Post Entries" |
| Point of Sale Session | `pos.session` | `action_show_payments_list` | `total_payments_amount` |
| Point of Sale Session | `pos.session` | `action_stock_picking` | "`picking_count` `picking_count`" |
| Point of Sale Session | `pos.session` | `action_view_order` | `order_count` |
| Point of Sale Session | `pos.session` | `open_frontend_cb` | "Continue Selling" |

### 10.28 Products and catalogue

34 operations on 7 entities. Preconditions, guards, refusal messages and side effects: [`../domains/products-and-catalog/`](../domains/products-and-catalog/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Product Attribute | `product.attribute` | `action_add_to_products` | "Add to products" |
| Product Attribute | `product.attribute` | `action_open_product_template_attribute_lines` | "`number_related_products` Products" |
| Product Attribute | `product.attribute` | `action_update_prices` | "Update extra prices" |
| Product Document | `product.document` | `action_open_pdf_form_fields` | "Configure dynamic fields" |
| Product Margin | `product.margin` | `action_open_window` | "Open Margins" |
| Pricelist | `product.pricelist` | `action_open_pricelist_report` | "Print" |
| Product Variant | `product.product` | `action_open_product_lot` | "Lot/Serial Numbers" |
| Product Variant | `product.product` | `action_product_forecast_report` | "`qty_available` `virtual_available` `virtual_available` `virtual_available` `uom_name` On Hand Forecasted" / "Forecast" |
| Product Variant | `product.product` | `action_used_in_bom` | `used_in_bom_count` |
| Product Variant | `product.product` | `action_view_bom` | `bom_count` |
| Product Variant | `product.product` | `action_view_mos` | "`mrp_product_qty` `uom_name` Manufactured" |
| Product Variant | `product.product` | `action_view_orderpoints` | "Min: `reordering_min_qty` Max: `reordering_max_qty`" / `nbr_reordering_rules` / "Replenishment" |
| Product Variant | `product.product` | `action_view_po` | "`purchased_product_qty` `uom_name` Purchased" |
| Product Variant | `product.product` | `action_view_related_putaway_rules` | "Putaway Rules" |
| Product Variant | `product.product` | `action_view_sales` | "`sales_count` `uom_name` Sold" |
| Product Variant | `product.product` | `action_view_stock_move_lines` | "In: `nbr_moves_in` Out: `nbr_moves_out`" |
| Product Variant | `product.product` | `action_view_storage_category_capacity` | "Storage Capacities" |
| Product Variant | `product.product` | `button_bom_cost` | "Compute Price from BoM" |
| Product Variant | `product.product` | `open_product_template` | "the product template." |
| Supplier Pricelist | `product.supplierinfo` | `action_set_supplier` | "Set as Supplier" |
| Product | `product.template` | `action_open_attribute_values` | "Configure" |
| Product | `product.template` | `action_open_documents` | `bom_count` / "`used_in_bom_count` `mrp_product_qty` `uom_name` Manufactured" / "`purchased_product_qty` `uom_name` Purchased" / `product_variant_count` / `product_document_count` / "`tracking` `show_on_hand_qty_status_button` `show_forecasted_qty_status_button` `qty_available` `virtual_available` `virtual_available` `virtual_available` `uom_name` On Hand Forecasted" / "Min: `reordering_min_qty` Max: `reordering_max_qty` `nbr_reordering_rules` In: `nbr_moves_in` Out: `nbr_moves_out` Lot/Serial Numbers Putaway Rules Storage Capacities" / "`sales_count` `uom_name` Sold" |
| Product | `product.template` | `action_open_product_lot` | "Lot/Serial Numbers" |
| Product | `product.template` | `action_product_tmpl_forecast_report` | "`qty_available` `virtual_available` `virtual_available` `virtual_available` `uom_name` On Hand Forecasted" |
| Product | `product.template` | `action_sync_gelato_template_info` | "Synchronize" |
| Product | `product.template` | `action_used_in_bom` | `used_in_bom_count` |
| Product | `product.template` | `action_view_mos` | "`mrp_product_qty` `uom_name` Manufactured" |
| Product | `product.template` | `action_view_orderpoints` | "Min: `reordering_min_qty` Max: `reordering_max_qty`" / `nbr_reordering_rules` |
| Product | `product.template` | `action_view_po` | "`purchased_product_qty` `uom_name` Purchased" |
| Product | `product.template` | `action_view_related_putaway_rules` | "Putaway Rules" |
| Product | `product.template` | `action_view_sales` | "`sales_count` `uom_name` Sold" |
| Product | `product.template` | `action_view_stock_move_lines` | "In: `nbr_moves_in` Out: `nbr_moves_out`" |
| Product | `product.template` | `action_view_storage_category_capacity` | "Storage Capacities" |
| Product | `product.template` | `button_bom_cost` | "Compute Price from BoM" |

### 10.29 Projects and tasks

29 operations on 6 entities. Preconditions, guards, refusal messages and side effects: [`../domains/projects-and-tasks/`](../domains/projects-and-tasks/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Project Milestone | `project.milestone` | `action_view_sale_order` | "Sales Order" / "View Sales Order" |
| Project Milestone | `project.milestone` | `action_view_tasks` | "View Tasks" |
| Project | `project.project` | `action_customer_preview` | "Preview" |
| Project | `project.project` | `action_open_share_project_wizard` | "Share Project" |
| Project | `project.project` | `action_view_sos` | "`sale_order_count` Sales Orders" / "0 Sales Order Make Billable" |
| Project | `project.project` | `action_view_tasks` | "`label_tasks` `closed_task_count` / `task_count` ( `task_completion_percentage` )" / "View Tasks" / "Create project" |
| Project Stage Delete Wizard | `project.project.stage.delete.wizard` | `action_archive` | "Archive Stages" |
| Project Stage Delete Wizard | `project.project.stage.delete.wizard` | `action_unarchive_project` | "Confirm" |
| Project Stage Delete Wizard | `project.project.stage.delete.wizard` | `action_unlink` | "Delete" |
| Project Sharing | `project.share.wizard` | `action_send_mail` | "Grant Portal Access" |
| Project Sharing | `project.share.wizard` | `action_share_record` | "Share Project" |
| Task | `project.task` | `action_convert_to_task` | "Convert to Task" |
| Task | `project.task` | `action_dependent_tasks` | `dependent_tasks_count` |
| Task | `project.task` | `action_open_parent_task` | "Parent Task" |
| Task | `project.task` | `action_open_ratings` | "`rating_avg_text` Last Rating" |
| Task | `project.task` | `action_open_subtasks` | "Sub-tasks `closed_subtask_count` / `subtask_count` ( `subtask_completion_percentage` )" |
| Task | `project.task` | `action_open_task` | "View Task" |
| Task | `project.task` | `action_project_sharing_open_blocking` | "Blocked Tasks `dependent_tasks_count`" |
| Task | `project.task` | `action_project_sharing_open_subtasks` | "`subtask_count` `display_in_project`" |
| Task | `project.task` | `action_project_sharing_recurring_tasks` | `recurring_count` |
| Task | `project.task` | `action_project_sharing_view_parent_task` | "Parent Task" |
| Task | `project.task` | `action_project_sharing_view_so` | "Sales Order" |
| Task | `project.task` | `action_recurring_tasks` | `recurring_count` |
| Task | `project.task` | `action_view_so` | "Sales Order" |
| Task | `project.task` | `action_view_subtask_timesheet` | "Time Spent on Sub-tasks:" |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `action_archive` | "Archive Stages" |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `action_confirm` | "Confirm" |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `action_unarchive_task` | "Confirm" |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `action_unlink` | "Delete" |

### 10.30 Purchasing

34 operations on 6 entities. Preconditions, guards, refusal messages and side effects: [`../domains/purchasing/`](../domains/purchasing/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `action_add_to_po` | "Add to PO" |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `action_match_lines` | "Match" |
| Purchase Order | `purchase.order` | `action_acknowledge` | "Acknowledge" |
| Purchase Order | `purchase.order` | `action_add_from_catalog` | "Catalog" |
| Purchase Order | `purchase.order` | `action_bill_matching` | "Bill Matching" |
| Purchase Order | `purchase.order` | `action_compare_alternative_lines` | "Compare Product Lines" |
| Purchase Order | `purchase.order` | `action_create_alternative` | "Create Alternative" |
| Purchase Order | `purchase.order` | `action_create_invoice` | "Create Bills" |
| Purchase Order | `purchase.order` | `action_product_forecast_report` | "Forecast Report" |
| Purchase Order | `purchase.order` | `action_purchase_comparison` | "Price Comparison `show_comparison`" |
| Purchase Order | `purchase.order` | `action_rfq_send` | "Send RFQ" / "Send PO" |
| Purchase Order | `purchase.order` | `action_view_dropship` | `dropship_picking_count` |
| Purchase Order | `purchase.order` | `action_view_invoice` | "`invoice_count` `invoice_ids`" |
| Purchase Order | `purchase.order` | `action_view_mrp_productions` | "`mrp_production_count` Manufacturing" |
| Purchase Order | `purchase.order` | `action_view_picking` | "Receive" / `incoming_picking_count` |
| Purchase Order | `purchase.order` | `action_view_repair_orders` | `repair_count` |
| Purchase Order | `purchase.order` | `action_view_sale_orders` | "`sale_order_count` Sale" |
| Purchase Order | `purchase.order` | `action_view_subcontracting_resupply` | "`subcontracting_resupply_picking_count` Resupply" |
| Purchase Order | `purchase.order` | `button_approve` | "Approve Order" |
| Purchase Order | `purchase.order` | `button_cancel` | "Cancel" |
| Purchase Order | `purchase.order` | `button_confirm` | "Confirm Order" |
| Purchase Order | `purchase.order` | `button_draft` | "Set to Draft" |
| Purchase Order | `purchase.order` | `button_lock` | "Lock" |
| Purchase Order | `purchase.order` | `button_unlock` | "Unlock" |
| Purchase Order | `purchase.order` | `print_quotation` | "Print" |
| Purchase Order Line | `purchase.order.line` | `action_choose` | "Choose" |
| Purchase Order Line | `purchase.order.line` | `action_clear_quantities` | "Clear Selected" / "Clear" |
| Purchase Requisition | `purchase.requisition` | `action_cancel` | "Cancel" |
| Purchase Requisition | `purchase.requisition` | `action_confirm` | "Confirm" |
| Purchase Requisition | `purchase.requisition` | `action_done` | "Close" |
| Purchase Requisition | `purchase.requisition` | `action_draft` | "Reset to Draft" |
| Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `action_cancel_alternatives` | "Cancel Alternatives" |
| Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `action_keep_alternatives` | "Keep Alternatives" |
| Wizard to preset values for alternative purchase order | `purchase.requisition.create.alternative` | `action_create_alternative` | "Create Alternative" |

### 10.31 Recruitment

14 operations on 5 entities. Preconditions, guards, refusal messages and side effects: [`../domains/recruitment/`](../domains/recruitment/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Get Refuse Reason | `applicant.get.refuse.reason` | `action_refuse_reason_apply` | "Refuse" |
| Send mails to applicants | `applicant.send.mail` | `action_send` | "Send" |
| Applicant | `hr.applicant` | `action_add_to_job` | "Move to this Job Position" |
| Applicant | `hr.applicant` | `action_create_meeting` | "`meeting_display_text` `meeting_display_date`" |
| Applicant | `hr.applicant` | `action_job_add_applicants` | "Create Applications" |
| Applicant | `hr.applicant` | `action_open_applications` | `application_count` |
| Applicant | `hr.applicant` | `action_open_employee` | "`employee_name` Employee" |
| Applicant | `hr.applicant` | `action_print_survey` | "Consult Interview" |
| Applicant | `hr.applicant` | `action_send_survey` | "Send Interview" |
| Applicant | `hr.applicant` | `action_talent_pool_add_applicants` | "Add Applicants " / "Add to Pool" |
| Applicant | `hr.applicant` | `action_talent_pool_stat_button` | `talent_pool_count` |
| Applicant | `hr.applicant` | `action_unarchive` | "Restore" |
| Add applicants to a job | `job.add.applicants` | `action_add_applicants_to_job` | "Create Applications" |
| Add applicants to talent pool | `talent.pool.add.applicants` | `action_add_applicants_to_pool` | "Add to Pool" |

### 10.32 Repair and maintenance

15 operations on 2 entities. Preconditions, guards, refusal messages and side effects: [`../domains/repair-and-maintenance/`](../domains/repair-and-maintenance/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Maintenance Equipment | `maintenance.equipment` | `action_open_matched_serial` | `serial_no` |
| Repair Order | `repair.order` | `action_add_from_catalog_repair` | "Catalog" |
| Repair Order | `repair.order` | `action_assign` | "Check availability" |
| Repair Order | `repair.order` | `action_create_sale_order` | "Create Quotation" |
| Repair Order | `repair.order` | `action_generate_serial` | "Creates a new serial/lot number" |
| Repair Order | `repair.order` | `action_repair_cancel` | "Cancel Repair" |
| Repair Order | `repair.order` | `action_repair_cancel_draft` | "Set to Draft" |
| Repair Order | `repair.order` | `action_repair_end` | "End Repair" |
| Repair Order | `repair.order` | `action_repair_start` | "Start Repair" |
| Repair Order | `repair.order` | `action_show_details` | "Details" |
| Repair Order | `repair.order` | `action_unreserve` | "Unreserve" |
| Repair Order | `repair.order` | `action_validate` | "Confirm Repair" |
| Repair Order | `repair.order` | `action_view_mrp_productions` | `production_count` |
| Repair Order | `repair.order` | `action_view_purchase_orders` | `purchase_count` |
| Repair Order | `repair.order` | `action_view_sale_order` | "Sale Order" |

### 10.33 Replenishment and procurement

5 operations on 1 entities. Preconditions, guards, refusal messages and side effects: [`../domains/replenishment-and-procurement/`](../domains/replenishment-and-procurement/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `action_product_forecast_report` | "Forecast Report" / "Due to receipts scheduled in the future, you might end up with excessive stock . Check the Forecasted Report  before reordering" |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `action_remove_manual_qty_to_order` | "Remove manually entered value and replace by the quantity to order based on the forecasted quantities" / "-" |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `action_replenish` | "Order" |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `action_replenish_auto` | "Automate" |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `action_stock_replenishment_info` | "Replenishment Information" / "Your product is missing a way to be replenished (Route, Vendor, Bill of Materials)." |

### 10.34 Sales

35 operations on 7 entities. Preconditions, guards, refusal messages and side effects: [`../domains/sales/`](../domains/sales/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Quotation's Headers & Footers | `quotation.document` | `action_open_pdf_form_fields` | "Configure dynamic fields" |
| Sale Loyalty - Apply Coupon Wizard | `sale.loyalty.coupon.wizard` | `action_apply` | "Apply" |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `action_apply` | "Apply" |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `action_cancel` | "Discard" |
| Cancel multiple quotations | `sale.mass.cancel.orders` | `action_mass_cancel` | "Cancel" |
| Sales Order | `sale.order` | `action_add_from_catalog` | "Catalog" |
| Sales Order | `sale.order` | `action_cancel` | "Cancel" |
| Sales Order | `sale.order` | `action_confirm` | "Confirm" |
| Sales Order | `sale.order` | `action_draft` | "Set to Quotation" |
| Sales Order | `sale.order` | `action_lock` | "Lock" |
| Sales Order | `sale.order` | `action_open_declaration_of_intent` | "Declaration of Intent" |
| Sales Order | `sale.order` | `action_open_delivery_wizard` | "Add shipping" / "Update shipping cost" |
| Sales Order | `sale.order` | `action_open_discount_wizard` | "Discount" / "Discounts" |
| Sales Order | `sale.order` | `action_open_reward_wizard` | "Reward" |
| Sales Order | `sale.order` | `action_preview_sale_order` | "Preview" |
| Sales Order | `sale.order` | `action_quotation_send` | "Send by Email" / "Send" / "Send PRO-FORMA Invoice" |
| Sales Order | `sale.order` | `action_recovery_email_send` | "Send a Recovery Email" |
| Sales Order | `sale.order` | `action_show_repair` | `repair_count` |
| Sales Order | `sale.order` | `action_unlock` | "Unlock" |
| Sales Order | `sale.order` | `action_update_prices` | "Update Prices" |
| Sales Order | `sale.order` | `action_update_taxes` | "Update Taxes" |
| Sales Order | `sale.order` | `action_view_attendee_list` | `event_booth_count` / `attendee_count` |
| Sales Order | `sale.order` | `action_view_booth_list` | `event_booth_count` |
| Sales Order | `sale.order` | `action_view_delivery` | `dropship_picking_count` / `delivery_count` |
| Sales Order | `sale.order` | `action_view_dropship` | `dropship_picking_count` |
| Sales Order | `sale.order` | `action_view_gift_cards` | `gift_card_count` |
| Sales Order | `sale.order` | `action_view_invoice` | `repair_count` / `expense_count` / "`project_count` Projects `tasks_count` Tasks `milestone_count`" / "`pos_order_count` Transferred to POS" / `invoice_count` / `delivery_count` |
| Sales Order | `sale.order` | `action_view_milestone` | "`timesheet_total_duration` `timesheet_encode_uom_id` Recorded" / `milestone_count` |
| Sales Order | `sale.order` | `action_view_mrp_production` | `mrp_production_count` |
| Sales Order | `sale.order` | `action_view_pos_order` | "`pos_order_count` Transferred to POS" |
| Sales Order | `sale.order` | `action_view_project_ids` | "`project_count` Projects `tasks_count` Tasks" |
| Sales Order | `sale.order` | `action_view_purchase_orders` | `purchase_order_count` |
| Sales Order | `sale.order` | `action_view_timesheet` | "`timesheet_total_duration` `timesheet_encode_uom_id` Recorded" |
| Discount Wizard | `sale.order.discount` | `action_apply_discount` | "Apply" |
| Update product attribute value | `update.product.attribute.value` | `action_confirm` | "Confirm" |

### 10.35 Site and storefront

6 operations on 4 entities. Preconditions, guards, refusal messages and side effects: [`../domains/website-and-storefront/`](../domains/website-and-storefront/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| User list of blocked 3rd-party domains | `website.custom_blocked_third_party_domains` | `action_save` | "Save" |
| Page | `website.page` | `action_page_debug_view` | "Settings" |
| Robots.txt Editor | `website.robots` | `action_save` | "Save" |
| Website Visitor | `website.visitor` | `action_send_chat_request` | "Chat" / "Send chat request" |
| Website Visitor | `website.visitor` | `action_send_mail` | "Email" / "Send Email" |
| Website Visitor | `website.visitor` | `action_send_sms` | "Send SMS" / "SMS" |

### 10.36 Taxes

2 operations on 1 entities. Preconditions, guards, refusal messages and side effects: [`../domains/taxes/`](../domains/taxes/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Fiscal Position | `account.fiscal.position` | `action_create_foreign_taxes` | "here" |
| Fiscal Position | `account.fiscal.position` | `action_open_related_taxes` | "Taxes" |

### 10.37 Time off

19 operations on 10 entities. Preconditions, guards, refusal messages and side effects: [`../domains/time-off/`](../domains/time-off/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Cancel Time Off Wizard | `hr.holidays.cancel.leave` | `action_cancel_leave` | "Cancel Time Off" |
| human resources Time Off Summary Report By Employee | `hr.holidays.summary.employee` | `print_report` | "Print" |
| Time Off | `hr.leave` | `action_approve` | "Approve" / "Validate" |
| Time Off | `hr.leave` | `action_back_to_approval` | "Back to Approval" |
| Time Off | `hr.leave` | `action_cancel` | "Cancel" |
| Time Off | `hr.leave` | `action_documents` | `supported_attachment_ids_count` |
| Time Off | `hr.leave` | `action_refuse` | "Refuse" |
| Accrual Plan Level | `hr.leave.accrual.level` | `action_save_new` | "Save & New" |
| Accrual Plan | `hr.leave.accrual.plan` | `action_create_accrual_plan_level` | "Create a milestone" |
| Accrual Plan | `hr.leave.accrual.plan` | `action_open_accrual_plan_employees` | `employees_count` |
| Time Off Allocation | `hr.leave.allocation` | `action_approve` | "Approve" / "Validate" |
| Time Off Allocation | `hr.leave.allocation` | `action_refuse` | "Refuse" |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `action_generate_allocations` | "Allocate Time Off" |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `action_generate_time_off` | "Generate Time Off" |
| Time Off Calendar | `hr.leave.report.calendar` | `action_approve` | "Approve" |
| Time Off Calendar | `hr.leave.report.calendar` | `action_refuse` | "Refuse" |
| Time Off Type | `hr.leave.type` | `action_see_accrual_plans` | "`accrual_count` Accruals" |
| Time Off Type | `hr.leave.type` | `action_see_days_allocated` | "`allocation_count` Allocations" |
| Time Off Type | `hr.leave.type` | `action_see_group_leaves` | "`group_days_leave` Time Off" |

### 10.38 Units of measure and packaging

1 operations on 1 entities. Preconditions, guards, refusal messages and side effects: [`../domains/units-of-measure-and-packaging/`](../domains/units-of-measure-and-packaging/).

| Entity | Entity identifier | Operation | Control label |
|---|---|---|---|
| Product Unit of Measure | `uom.uom` | `action_open_packaging_barcodes` | "Packaging Barcodes" |

## 11. Transactions, isolation and concurrency

1. **One call, one transaction.** Every remote call runs in its own transaction, committed on success, discarded on failure. Two calls are never in the same transaction.
2. **Repeatable reads.** Within a call, a record read twice yields the same values unless the call itself changed them.
3. **Automatic retry.** A call that fails on a serialization conflict, a deadlock or a lock timeout is replayed up to five times with an exponential random delay ([`remote-transport-contracts.md`](remote-transport-contracts.md), section 13). A call must therefore be safe to execute from the start more than once; anything that is not (consuming a number from an external counter, posting to a payment provider) must take a lock or be made idempotent by a stored marker.
4. **Deferred writes.** Values assigned during a call are flushed to the database before the commit and before any query that depends on them; a caller never observes a stale value inside its own call.
5. **Constraint timing.** Validations declared on fields run when those fields are written; database-level constraints are checked at the flush. Both surface as failures of the call, and the whole call is discarded.
6. **Locks.** An operation that must serialize itself takes an explicit row lock; a lock that cannot be taken fails with status `409` and the operation's own message.
7. **Work after the commit.** Notifications pushed to clients, and any other side effect that must not happen if the transaction is discarded, are registered as post-commit work and executed after the commit succeeds. A failure there does not roll the transaction back; it is logged.

## 12. Acceptance criteria

**AC-SERVICE-001 — Search and read are one transaction.** Given a record that another transaction deletes between two calls, when a caller uses the combined search and read operation, then the answer never reports a missing record; when a caller instead searches and then reads, then the read simply omits the deleted record.

**AC-SERVICE-002 — Read omits vanished records.** Given three keys of which one designates a deleted record, when they are read, then the answer holds two payloads and the caller matches them by their `id` member.

**AC-SERVICE-003 — Creation with a single mapping.** Given the creation operation invoked remotely with one values mapping, then the result is an integer key; invoked with an array of two mappings, then the result is an array of two keys in input order.

**AC-SERVICE-004 — Defaults resolution order.** Given a field with a declared default of 10, a stored user default of 20 and a context entry `default_<field>` of 30, when the defaults are requested, then the value is 30; without the context entry, 20; without the user default, 10.

**AC-SERVICE-005 — Defaults normalize relations.** Given a context carrying two link commands for a one-to-many field, when the defaults are requested, then the returned value is a single replace command listing both keys.

**AC-SERVICE-006 — Relational commands.** Given a record with lines 11, 12 and 13, when the write `[[1, 12, {...}], [2, 13, 0], [0, 0, {...}], [4, 20, 0]]` is applied, then line 12 is changed, line 13 no longer exists, a new line exists, line 20 is linked, and line 11 is untouched.

**AC-SERVICE-007 — Replace command deletes nothing.** Given a many-to-many currently holding 3, 5 and 7, when `[[6, 0, [3, 5]]]` is written, then the relation holds exactly 3 and 5 and record 7 still exists.

**AC-SERVICE-008 — Private operation.** Given any caller, when `sudo` or `with_user` is invoked remotely on any entity, then the call is refused with the private-operation access failure.

**AC-SERVICE-009 — Specification with only the key.** Given a specification containing only `id`, when a read is performed on 500 keys, then 500 payloads are returned, each holding only `id`, and no read of the entity is performed.

**AC-SERVICE-010 — Nested many-to-one.** Given the specification `{"partner_id": {"fields": {"display_name": {}}}}`, when a record whose reference is empty is read, then `partner_id` is false; when the reference is set, then it is an object holding `id` and `display_name`.

**AC-SERVICE-011 — Line limit.** Given a one-to-many with three targets and the specification `{"limit": 2, "fields": {...}}`, when the record is read, then the first two targets are full payloads and the third is an object holding only `id`.

**AC-SERVICE-012 — Line ordering.** Given a one-to-many and the specification `{"order": "name desc"}`, when the record is read, then the array of keys is sorted by the target's name descending, and archived targets are included.

**AC-SERVICE-013 — Inaccessible targets are filtered.** Given a one-to-many whose targets include one the caller may not read, when the record is read with a nested specification, then that target is absent from the answer and no access failure is raised.

**AC-SERVICE-014 — Reference field payload.** Given a reference field pointing at an existing record and the specification `{"fields": {"display_name": {}}}`, when the record is read, then the value is an object whose `id` member is `{"id": <key>, "model": <entity>}`; when the target has been deleted, then the value is false.

**AC-SERVICE-015 — First on-change call.** Given an empty record set and an empty list of changed fields, when the on-change protocol is invoked with a specification of five fields, then the answer carries a value for every one of the five that has a default or a derivation, including the ones the client did not send.

**AC-SERVICE-016 — On-change diff only.** Given a saved record and one changed field that causes two other fields to be derived, when the on-change protocol is invoked, then the answer carries exactly those two derived fields and not the changed field itself.

**AC-SERVICE-017 — On-change line commands.** Given a form with one unsaved line, when a change causes a member of that line to be recomputed, then the answer carries an `update` command on that line carrying only the recomputed member.

**AC-SERVICE-018 — On-change removed line.** Given a form where the user removed an existing line, when the on-change protocol is invoked, then the answer carries a `delete` command for a one-to-many, or an `unlink` command for a many-to-many, on that line's key.

**AC-SERVICE-019 — Merged warnings.** Given two handlers that each emit a warning during one on-change call, when it returns, then a single warning is present whose title is `"Warnings"`, whose body holds both messages separated by a blank line, and whose kind is `dialog`.

**AC-SERVICE-020 — On-change access check.** Given a caller without write access on an existing record, when the on-change protocol is invoked on it, then the call fails with the access failure before any handler runs.

**AC-SERVICE-021 — Save then read.** Given the save operation on an existing record, when it is called with values and a specification, then the values are written, the answer holds one payload shaped by the specification, and binary fields in that payload carry their size rather than their content.

**AC-SERVICE-022 — Save creates.** Given the save operation on an empty record set, when it is called, then a record is created and its payload is returned.

**AC-SERVICE-023 — Multi-save mismatch.** Given three records and two values mappings, when the multi-save operation is called, then it fails with `"Each record must have a corresponding vals entry."` and nothing is written.

**AC-SERVICE-024 — Resequencing.** Given four records passed in the order D, B, A, C with an offset of 10, when the resequencing operation is called on the sequence field, then D holds 10, B holds 11, A holds 12 and C holds 13, and the answer holds their payloads.

**AC-SERVICE-025 — Resequencing an entity without the field.** Given an entity with no such field, when the resequencing operation is called, then the answer is an empty array and nothing is written.

**AC-SERVICE-026 — Grouped read payload.** Given a grouping on a `many_to_one`, when the grouped read runs, then each group carries the pair `[key, display name]`, the requested aggregates, a filter that selects exactly its records, and, when requested, its count.

**AC-SERVICE-027 — Grouping by month.** Given a grouping `date:month` on a date field with records in June and September, when the grouped read runs without the filling flag, then two groups are returned; with the filling flag, then four groups are returned, July and August carrying zero aggregates.

**AC-SERVICE-028 — Filling with a limit is refused.** Given the filling flag and a limit, when the grouped read runs, then it fails with `"You cannot used fill_temporal with a limit or an offset"`.

**AC-SERVICE-029 — Automatic unfolding limit.** Given 25 groups and automatic unfolding, when the screen-level grouped read runs, then at most 10 groups carry records and the rest carry none.

**AC-SERVICE-030 — Group record paging.** Given an open group of 200 records with a default per-group limit of 80, when the screen-level grouped read runs, then the group carries 80 payloads and its count is 200; when the caller asks for an offset of 250 on that group, then the offset is reset and the group carries the first 80 payloads together with `__offset` equal to 0.

**AC-SERVICE-031 — Result length with a ceiling.** Given 250 matching records, a limit of 80 and a ceiling of 200, when the combined search and read runs, then the length is 200; with a ceiling of 80, then the length is 80.

**AC-SERVICE-032 — Name search.** Given the name search with the pattern `deco` and a limit of 2, when it runs, then at most two pairs are returned and each pair's display name matches the pattern under the given operator.

**AC-SERVICE-033 — Name creation without a naming field.** Given an entity with no naming field, when the name-creation operation is called, then it answers false and creates nothing.

**AC-SERVICE-034 — Archiving.** Given three records of which two are active, when the archive operation is called on all three, then the two active ones become archived and the already archived one is untouched; the unarchive operation is symmetrical.

**AC-SERVICE-035 — Copy excludes technical columns.** Given a record with an external name, a creation user and a creation instant, when it is copied, then the copy has a new key, no external name, the caller as creation user and the current instant as creation date, and every field marked as not copied holds its default.

**AC-SERVICE-036 — Copy of translations.** Given a record whose label has three translations, when it is copied without overriding that field, then the copy carries the same three translations; when the copy overrides it, then only the given value is set.

**AC-SERVICE-037 — Permission probe.** Given a caller without deletion rights, when the permission probe is invoked with the deletion operation on a record set, then it answers false and raises nothing.

**AC-SERVICE-038 — Field-level permission is visible.** Given a field restricted to an access group the caller is not in, when the field descriptions are requested, then that field is absent from the answer.

**AC-SERVICE-039 — Entity definitions filtering.** Given a request for two entities where the first has a relational field pointing at a third entity, when the definitions are requested, then that relational field is absent from the answer.

**AC-SERVICE-040 — Unknown and inaccessible entities are indistinguishable.** Given the labels of entities requested for a name that does not exist and for one the caller may not read, then both answers carry the requested name as the label.

**AC-SERVICE-041 — Export permission.** Given a caller without the export permission and who is not an administrator, when the export operation is invoked, then it fails with `"You don't have the rights to export data. Please contact an Administrator."`.

**AC-SERVICE-042 — Export row layout.** Given one record with two lines and the columns `name`, `line_ids/product_id/display_name`, when it is exported, then two rows are produced, the first carrying the record's name and the first line, the second carrying an empty name and the second line.

**AC-SERVICE-043 — Re-importable external names.** Given a record without an external name and an export in the re-importable mode including the `id` column, when it is exported, then an external name is created, stored, and written in the cell, and re-importing the file updates the same record instead of creating a new one.

**AC-SERVICE-044 — Formula protection.** Given a text cell whose content starts with `=`, when it is exported to the flat format, then the cell is prefixed with an apostrophe.

**AC-SERVICE-045 — Grouped aggregates.** Given a grouped spreadsheet export with a numeric column whose aggregation is the sum, when it is produced, then each group header row carries the sum of its rows, and a parent group carries the sum of its children; for a column whose aggregation is the average, the parent carries the count-weighted average.

**AC-SERVICE-046 — Import row folding.** Given a file whose second row carries values only in one-to-many columns, when it is imported, then it is folded into the record of the first row instead of creating a second record.

**AC-SERVICE-047 — Import failure is atomic.** Given a file where one row references an unresolvable target, when it is imported, then the answer carries `ids` equal to false, at least one message of kind `error` naming the row range, and no record has been created.

**AC-SERVICE-048 — Import error flood.** Given a file with more than ten errors and more than one error per ten records, when it is imported, then the run stops early and the answer carries the interruption warning.

**AC-SERVICE-049 — Wizard round trip.** Given a wizard opened from a list of two records, when the client calls the on-change protocol with an empty record set, then it receives the wizard's defaults including the values derived from the context keys naming the selected records; when it then saves and presses the confirmation button, then the work is performed and the returned action is the one the wizard declares.

**AC-SERVICE-050 — Button returning a notification.** Given a business operation that returns a notification action, when it is invoked through the button endpoint, then the result is a client action with the tag `display_notification`, carrying the severity, the message and the follow-up close action.

## 13. Reconciliation notes

Where the drafts merged into this document differed from each other or from the behaviour of the system, the point was
settled against the source of the system and against the generated catalogues
[`../references/operation-index.md`](../references/operation-index.md) and
[`../references/views.md`](../references/views.md). The resolutions:

1. **The named business operations were missing.** The plan for this document promises the named business operations
   exposed on entities, grouped by domain, with their inputs, outputs, preconditions and side effects. One draft
   specified only the generic contract and the conventions. Section 10 now enumerates all 1,016 named operations that
   the screens of the shipped installation bind to a control, on 285 entities, grouped into 38 areas, and states once —
   rather than 1,016 times — what is true of every one of them: the inputs, the shape of the output, that the
   preconditions are re-checked inside the operation, and where the guards, refusal messages and side effects of each
   one are specified.
2. **The superseded grouped read.** A generic operation named `read_group` exists on every entity and is marked as
   superseded; it is therefore not described. The grouped read a rebuild must expose is `formatted_read_group`, with
   `formatted_read_grouping_sets` for several groupings in one pass and `web_read_group` for a screen; all three are in
   section 2.1 and section 7.6.
3. **Section numbering.** Adding section 10 moved the transaction rules to section 11 and the acceptance criteria to
   section 12. No cross-reference from another document pointed at either, and the acceptance criteria keep their
   identifiers.
4. **Reproduced control labels.** Three control labels contain the product's own name in the shipped installation.
   Section 10 reproduces every label in quotation marks, and those three with the product's name replaced by "the
   platform", because a replacement substitutes its own name and no observable behaviour depends on the word.
