# Client architecture

The desktop client is a single page that reads the declarative view descriptions of [views and actions](views-and-actions.md) and turns them into screens. This document specifies its observable behaviour as a contract: what the server hands it at start-up, how it loads the navigation tree, how it decides what is on screen and what the navigation trail says, what the control panel offers and exactly which condition each control contributes, how a table screen and a record screen load, edit, recompute, save, discard and navigate, how a list of related records is edited and what is sent back, what the notice, dialog, celebration and command mechanisms guarantee, which keyboard shortcuts exist and what they do, which services the screens depend on, and how the point of sale client behaves when the network is unavailable.

A replacement client that satisfies this contract can be written with any technology, and a replacement server that satisfies the payloads described here can serve the original client. Everything that concerns *what* a screen looks like is in [views and actions](views-and-actions.md); everything that concerns record semantics is in [record operations and query notation](record-operations-and-query-notation.md); permission evaluation is in [the security model](security-model.md); printing and exporting are in [report rendering](../runtime/report-rendering.md); the assembly of the static files the client loads is in [the package system, section 22](package-system.md#22-client-asset-bundles).

Nothing in this document is enforcement. Every restriction a client applies is guidance that a caller speaking the transport can ignore; the gates that are enforcement are in [the security model, section 20](security-model.md#20-what-is-not-enforcement).

---

## Table of contents

1. [Structure](#1-structure)
2. [Bootstrap](#2-bootstrap)
3. [The navigation tree](#3-the-navigation-tree)
4. [The action manager](#4-the-action-manager)
5. [The control panel](#5-the-control-panel)
6. [The search model](#6-the-search-model)
7. [Loading a screen](#7-loading-a-screen)
8. [The record screen](#8-the-record-screen)
9. [The table screen](#9-the-table-screen)
10. [Editing a list of related records](#10-editing-a-list-of-related-records)
11. [Navigating between records](#11-navigating-between-records)
12. [Feedback mechanisms](#12-feedback-mechanisms)
13. [The top bar](#13-the-top-bar)
14. [The command palette](#14-the-command-palette)
15. [Keyboard shortcuts](#15-keyboard-shortcuts)
16. [The service catalogue](#16-the-service-catalogue)
17. [The point of sale client](#17-the-point-of-sale-client)
18. [Acceptance criteria](#18-acceptance-criteria)
19. [Invariants a rebuild must preserve](#19-invariants-a-rebuild-must-preserve)
20. [Reconciliation notes](#20-reconciliation-notes)

---

## 1. Structure

### 1.1 Registries

The client is assembled from **registries**: named, ordered maps from a key to a contributed value. A capability package contributes to a registry instead of editing the code that reads it. Every registry entry may declare a sequence number; entries are read in ascending sequence, then in insertion order. The registries a replacement must provide, with the shape of their entries:

| Registry | Key | Value | Read by |
|---|---|---|---|
| `services` | service name | a descriptor with a dependency list and a start procedure | the service starter (section 1.2) |
| `main_components` | component name | a top-level component and its properties | the page root, which renders them all at once |
| `actions` | client behaviour tag | either a screen component or a plain procedure | the action manager, for a client action |
| `action_handlers` | action category name | a handler procedure | the action manager, for an unknown category |
| `views` | view kind, or a specialization name | a view descriptor (parser, data model, controller, renderer, default page sizes) | the view loader |
| `fields` | widget name | a widget descriptor (component, accepted data types, accepted options, property extraction) | every view that renders a field |
| `view_widgets` | widget name | a component | the `widget` node of a view |
| `formatters` / `parsers` | data type or widget name | a value-to-text and a text-to-value procedure | every widget |
| `systray` | item name | a component | the top bar |
| `user_menuitems` | item name | a procedure returning an entry descriptor | the user menu |
| `cogMenu` | item name | a component with an availability predicate | the actions menu of a screen |
| `favoriteMenu` | item name | a component | the favourites menu |
| `command_provider` | provider name | a procedure returning commands, with an optional namespace | the command palette |
| `command_categories` | category name | an ordering | the command palette |
| `command_setup` | namespace prefix | the prompt, the empty message and the namespace label | the command palette |
| `error_handlers` | handler name | a procedure returning whether it handled the failure | the failure service |
| `error_dialogs` | server failure name | a dialog component | the failure handler |
| `error_notifications` | server failure name | a notice descriptor | the failure handler |
| `effects` | effect name | a procedure returning a component and its properties | the celebration service |
| `ir.actions.report handlers` | handler name | a procedure that may intercept a print request | the action manager |

### 1.2 Services

A **service** is a singleton with a name, a declared dependency list and a start procedure that returns the object other code uses. Services are started in dependency order; a cycle or a missing dependency is a startup failure. Every screen and every widget reaches a service through one shared environment object, never through a global.

### 1.3 The environment

One environment object is shared by the whole page. It carries the started services, an event bus, the current screen configuration (the acting action, its identifier, the current view kind, a procedure returning the display name of the current screen), and a flag telling whether the viewport is narrow. Screens may extend it locally for their subtree, which is how a dialog exposes its own closing procedure to its content.

## 2. Bootstrap

### 2.1 The startup sequence

1. The page is served with an inline payload holding the session data ([section 2.2](#22-the-session-payload)) and the colour scheme.
2. The translation payload for the active language is fetched ([section 2.4](#24-translations)).
3. The asset bundles declared for the back office are loaded ([the package system, section 22](package-system.md#22-client-asset-bundles)).
4. Every service in the registry is started in dependency order.
5. The navigation tree is loaded ([section 3](#3-the-navigation-tree)).
6. The page root component is mounted; it renders the top bar, the action area and every entry of the top-level component registry — notices, dialogs, overlays and the loading indicator.
7. The address in the location bar is parsed and the matching action is run ([section 4.7](#47-the-stack-the-trail-and-the-address)). When the address names nothing, the user's start screen is run: the action named by the user's start-screen field when it is set, otherwise the first application of the navigation tree.


### 2.2 The session payload

The payload is produced by the server for the back office. Every key is part of the contract.

| Key | Type | Meaning |
|---|---|---|
| `uid` | integer or false | The key of the acting user; false for an anonymous visitor. |
| `is_system` | boolean | Whether the user belongs to the system administration group. |
| `is_admin` | boolean | Whether the user is an administrator. |
| `is_public` | boolean | Whether the user is the anonymous identity. |
| `is_internal_user` | boolean | Whether the user is an internal member of the organization. |
| `user_context` | mapping | The contextual keys of the user: language, time zone, active companies. It is also stored back into the session when it differs from what the session held. |
| `db` | text | The name of the data store the session is bound to. |
| `registry_hash` | text | A signed fingerprint of the currently loaded set of packages. A client caches server answers against it and discards the cache when it changes. |
| `user_settings` | mapping | The per-user preferences record (section 2.3). |
| `server_version`, `server_version_info` | text, list | The platform version, as text and as structured parts. |
| `support_url` | text | The address of the support page. |
| `name` | text | The user's display name. |
| `username` | text | The user's login. |
| `quick_login` | boolean | Whether the shortened login flow is offered. |
| `partner_write_date` | datetime | The last change of the user's contact record; used to invalidate cached avatars. |
| `partner_display_name` | text | The display name of that contact. |
| `partner_id` | integer or nothing | The key of that contact. |
| `base_web_address` | text | The canonical address of this deployment, used to build absolute links. |
| `active_ids_limit` | integer | The greatest number of record keys the client may put in a contextual key when the user selects "all records matching the search". Default `20000`. |
| `max_file_upload_size` | integer | The greatest accepted upload size, in bytes. |
| `profile_session`, `profile_collectors`, `profile_params` | text, list, mapping | The state of the performance-recording feature. |
| `home_action_id` | integer or false | The action to run at startup for this user. |
| `currencies` | mapping | Every currency, keyed by its record key, with its symbol, decimal places and symbol position, in order that monetary values can be formatted without a round trip. |
| `bundle_params` | mapping | The parameters the asset bundles were built with: at least the language, plus the diagnostic mode when one is active. |
| `test_mode` | boolean | Whether the deployment runs in automated-test mode. |
| `view_info` | mapping | Per view kind: its display name, its icon and whether it shows many records (section 2.5). |
| `groups` | mapping | A small set of precomputed group memberships the client needs before any round trip; at minimum the "may export data" group. |
| `user_companies` | mapping | Present only for an internal user. `current_company` is the key of the acting company; `allowed_companies` maps each allowed company key to its key, name, sequence, child keys, parent key and currency key; `disallowed_ancestor_companies` maps the ancestors the user is **not** allowed to act for to the same shape minus the currency, in order that the company tree can be drawn completely while only the allowed nodes are selectable. |
| `show_effect` | boolean | Present only for an internal user: whether celebration effects are played. |

A front-office page receives a reduced payload: the identity flags, the acting user key, the package fingerprint, a marker saying the page is a front-office page, the performance-recording state, the celebration switch, the currencies, the shortened-login switch, the bundle parameters, the test-mode flag, and the version when a user is logged in.

### 2.3 User preferences

The preferences record carries at least: the preferred colour scheme, the notification sound settings, the pinned screens, and the per-action ordering of embedded entries (used by [views and actions, section 26](views-and-actions.md#26-embedded-actions)). It is created on first read when it does not exist.

### 2.4 Translations

A separate request returns, for a set of packages and one language: the translated terms of the client code and of the client templates, plus the language's formatting data (date format, time format, decimal separator, thousands separator, week start, direction) and a fingerprint. The client sends the fingerprint it already has; the server answers with an empty body when nothing changed. Before a session exists, a minimal payload limited to the packages that declare themselves bootstrap-capable is served, in the base language of the browser, in order that the login page can be translated.

### 2.5 View kind metadata

`view_info` gives, per view kind: the display name, the icon and whether the type shows several records at once. The built-in entries are:

| Layout type | Shows many records |
|---|---|
| `list` | yes |
| `form` | **no** |
| `graph` | yes |
| `pivot` | yes |
| `kanban` | yes |
| `calendar` | yes |
| `search` | not applicable |

Capability packages add entries, for instance the activity view and the hierarchy view. A window action offering a view kind absent from this mapping is a configuration failure: `View types not defined <types> found in act_window action <key>`; an action offering no usable view at all fails with `No view found for act_window action <key>`.

## 3. The navigation tree

The client loads the whole tree once, as specified in [views and actions, section 24](views-and-actions.md#24-menus), and keeps it in memory for the life of the page. The menu service exposes:

| Operation | Contract |
|---|---|
| `get_all()` | Every visible entry. |
| `get_menu(application key)` | The sub-tree of one application. |
| `get_current_app()` | The application the current screen belongs to. |
| `get_menu_as_tree(key)` | The sub-tree as nested entries, each with its own children resolved. |
| `select_menu(entry)` | Load the entry's action and run it; when the entry is an application root, the navigation trail is cleared first. |
| `set_current_menu(entry)` | Record which entry is highlighted, without running anything. |
| `reload()` | Re-fetch the tree, used after a package installation. |

### 3.1 The delivered payload

What the server delivers is **not** the stored Menu record of [views and actions, section 24](views-and-actions.md#24-menus). It is a purpose-built payload, keyed by entry key, and every entry carries exactly these keys:

| Key | Value |
|---|---|
| `identifier` | The entry's own key. |
| `name` | The label, translated into the acting user's language. |
| `application` | The key of the application root this entry belongs to. Every delivered entry has one. |
| `action_model` | The transport name of the entity of the entry's action, or false when the entry has no action. |
| `action_identifier` | The identifier of the entry's action, or false. |
| `action_path` | The path of the entry's action ([views and actions, section 16.1](views-and-actions.md#161-common-fields)), or false. |
| `web_icon` | The icon descriptor of the entry. |
| `web_icon_data` | The icon's content, rendered as text. |
| `web_icon_data_mimetype` | The media type of that content, so that the client can render it without sniffing. |
| `external_identifier` | The complete external identifier of the entry — package name, a full stop, local name — or the empty text when the entry has none. |
| `children` | The keys of this entry's **visible** children, in the order of [views and actions, section 24.2](views-and-actions.md#242-ordering). |

To those eleven the client adds two derived values of its own, used by the command palette and the application switcher: the concatenation of the ancestor labels (`parents`), and a link target built from the entry key and its action key.

**The synthetic root entry.** The payload additionally carries one entry under the key `root`, which corresponds to no Menu record: its `identifier` is false, its `name` is the literal text `root`, and its `children` are the keys of the application roots. It exists so that the tree has a single entry point and the client can walk it without a special case for the top level.

**Broken chains are dropped.** An entry that is itself visible but whose chain of ancestors to an application root is broken — because an intermediate folder is not visible to this user ([views and actions, section 24.1](views-and-actions.md#241-visibility)) — is **removed** from the payload rather than reparented. The rule guarantees the invariant the client depends on: every delivered entry belongs to exactly one application, so `get_current_app()` always has an answer and the application switcher never shows an orphan.

**The roots-only operation.** A second operation returns only the application roots, each with its label, its sequence, its parent, its action and its icon content, plus the list of all root keys. A client loads that first and draws the application switcher with it, then loads the full tree; the switcher is therefore usable before the whole tree has arrived, which matters because the full tree of a large installation is the largest single payload of the bootstrap.

**Caching.** Both results are cached per user and per language — per user because visibility is computed from the user's groups, per language because the labels are translated. The full tree is additionally cached per diagnostic state, because diagnostic mode adds entries that are otherwise hidden. Installing or removing a package invalidates all of them, which is what the reload operation above relies on.

## 4. The action manager

### 4.1 The controller stack

The client keeps an ordered **controller stack**. Each entry describes one screen: the action it belongs to, the view kind, the component to render, the properties handed to it, a run identifier unique to the entry, the exported state of the screen, and the display name shown in the navigation trail. The **last** entry is what the user sees. Entries before it form the navigation trail.

An entry may be **virtual**: reconstructed from the address in the location bar without having been visited in this page. A virtual entry carries only enough information to be restored on demand.

### 4.2 Running an action

1. Load the action and pre-process it, passing any additional context ([views and actions, section 16.3](views-and-actions.md#163-how-an-action-reaches-the-client)).
2. When the action's target is the main area, mark the trail to be cleared.
3. Dispatch on the action's kind: an address action follows [section 4.3](#43-web-address-action); a window action follows [section 4.4](#44-window-action); a close action follows [section 4.5](#45-close-window-action); a client action follows [section 4.6](#46-client-action); a server action is run on the server and the action it returns is then run, or a close action when it returns nothing, the path of the original action being inherited when the returned action has none; a report action follows [report rendering](../runtime/report-rendering.md); anything else is handed to the handler registered for that category, failing with **"The ActionManager service can't handle actions of type "** followed by the kind when none is registered.


Only the **latest** request survives: a new request cancels the answer of the previous one, which is what prevents a slow screen from replacing a faster one the user asked for afterwards.

Before a window action or a client action is placed in the main area (not in a dialog, not in a new window), every screen currently on the stack is asked to commit its uncommitted changes. A screen that refuses aborts the navigation and the stack is left untouched.

### 4.3 Web address action

1. Take the action's address.
2. When the address begins with neither the transfer-protocol prefix nor a leading slash, prefix it with a leading slash.
3. When the action's target is the current window, the page navigates to the address.
4. When the target is a download, a new window is opened at the address.
5. Otherwise a new window is opened at the address; then, when the action carries the closing flag, a close action is run, and otherwise the caller's completion callback is invoked.


When the new window is blocked, the sticky warning of [section 12.1](#121-notices) is raised.

### 4.4 Window action

1. Build the offered list: for every view entry of the action except the search entry, a tuple of the view kind, its icon, its display name and whether it shows many records.
2. If an offered kind is unknown to the session metadata, fail. If the offered list is empty, fail.
3. Choose the entry whose kind equals the requested kind, and the first entry when none matches.
4. When the viewport is narrow, look for an entry whose kind equals the action's narrow-screen kind **and** whose "shows many records" flag equals the chosen entry's; when such an entry exists, it becomes the chosen one.
5. Create a controller for the action, the chosen entry and the offered list, and remember it on the action under the chosen kind.
6. Update the screen with that controller and the options.


A controller is remembered per (action run, view kind), in order that switching from a table to a form and back restores the table where it was.

### 4.5 Close window action

When a dialog is open, the topmost dialog is removed and its completion callback is invoked with the action's payload. When no dialog is open, the completion callback supplied by the caller is invoked with the same payload. Nothing is added to the stack.

### 4.6 Client action

When the registered value is a screen component: unless the target is a dialog or a new window, the uncommitted-changes check runs first; the component may declare its own target, which then overrides the action's; the component may extract its properties from the action; the resulting entry is pushed like any other. When the registered value is a plain procedure, it is called with the environment, the action and the options, and the action it returns, if any, is run in turn.

### 4.7 The stack, the trail and the address

**Placing a controller.**

1. Determine the position at which the new controller is inserted.
2. When the trail is to be cleared, the position is the beginning.
3. Otherwise, when the caller asked to replace the current action, the position is that of the first controller belonging to the same action; when the caller asked to replace the previous action, the position is that of the last controller of the previous action; otherwise the position is the end of the stack.
4. The next stack is the stack up to that position followed by the new controller.


Switching to a view that shows many records replaces the whole segment of the current action; switching to a view that shows one record appends to it. This is what makes the trail read `Sales Orders / SO0042` and not `Sales Orders / Sales Orders / SO0042`.

**The trail** is the list of stack entries, excluding entries whose action is the application menu. Each item exposes: the entry's run identifier, its display name (read lazily, in order that a record renamed on screen renames its trail item), whether it is a record screen, the address that restores it, and the procedure that returns to it.

**The address** mirrors the stack. It carries, for each entry: the action (its path when it has one, otherwise its key, otherwise the behaviour tag, otherwise the entity name), the view kind when it is not implicit, the record key (or the literal `new` for an unsaved record), the acting parent record when there is one, and any screen-specific state the screen chose to publish. The last entry's values are also hoisted to the top level of the address, which is what makes a single screen's address short and shareable. The whole thing is also written to the per-tab storage, in order that a reload restores the trail.

**Restoring from an address.** Each entry of the address becomes a virtual controller. Their display names are resolved in one batch request that, for each entry, returns either the display name of the named record, the name of the named action, or a failure marker; a failure marker drops that entry and every entry before it. The last entry is then run as a real action. When the named action no longer exists, the client retries with the trail truncated by one, and when nothing is left it falls back to the user's start screen.

**Returning to a trail item.** The uncommitted-changes check runs first. A virtual item is turned into a real action run. A real item is re-rendered from its exported state; for a record screen the last record key it exported is used, in order that paging inside the record screen is not lost when the user leaves and comes back.

### 4.8 Dialogs

An action whose target is `new` is rendered inside a dialog instead of being pushed onto the stack. The dialog size is taken from the contextual key `dialog_size` and defaults to `large`. Only one action dialog is open at a time; opening a second one replaces the first. Closing a dialog invokes the completion callback given when it was opened. While a dialog is open, switching view is refused, because the view switcher belongs to the screen behind the dialog.

### 4.9 New window

An action may be asked to open in a new browser tab. The current action and the current address state are written to the per-tab storage, the address of the new screen is computed, and the tab is opened at that address. No uncommitted-changes check is performed, because the current screen is not left.

## 5. The control panel

Every multi-record screen and every record screen is framed by the same control panel.

| Zone | Content |
|---|---|
| Trail | The navigation trail of section 4.7, the last item being the current screen's display name. |
| Screen buttons | The buttons declared in the view's `header` element, plus the standard create, save and discard buttons of the view kind. |
| Search area | The search input with its facets (section 6). Hidden when the action offers no search view. |
| View switcher | One control per offered view kind, in the order of the action's view list, each with the icon and the display name from the session metadata. |
| Pager | Present on a multi-record screen and on a record screen reached from one (section 9). |
| Actions menu | The contextual menu holding the standard record operations and the bound actions of binding type `action` ([views and actions, section 23](views-and-actions.md#23-binding-actions-to-entities)) plus every available entry of the actions-menu registry. |
| Print menu | The bound actions of binding type `report`. |

### 5.1 The standard entries of the actions menu

| Entry | Availability |
|---|---|
| Duplicate | The view allows duplication and exactly one record is addressed. |
| Archive / Unarchive | The entity has the archive flag and the view allows modification. Archiving asks `Are you sure that you want to archive all the selected records?` (many) or `Are you sure that you want to archive this record?` (one), with the accepting caption `Archive`. |
| Delete | The view allows deletion. |
| Export | The user belongs to the export group and the view allows exporting. |
| Edit properties | The view contains a user-defined-fields element. |
| Importing records | Contributed by the package that imports records from files; available on a multi-record screen whose view allows importing. |
| Add to dashboard | Contributed by the dashboard package; available on a multi-record screen with a search view. |
| Export all | Contributed on a table screen; exports every record matching the current search rather than the current page. |

Duplicating many records asks `Are you sure that you want to duplicate all the selected records?` with the accepting caption `Confirm`; a partial failure raises the notice `Some records could not be duplicated`.

### 5.2 The selection bar

On a multi-record screen, selecting rows reveals a bar stating how many records are selected and offering to extend the selection to **every** record matching the current search. When the extended selection is used and the number of matching records exceeds the session's record-key limit, operations act on the first records only and the notice `Only the first %(count)s records have been deleted (out of %(total)s selected)` is raised, with the two numbers filled in.

## 6. The search model

The search model owns the condition, the grouping, the ordering and the extra contextual keys that the screen applies. It is built once per action run from the search view, the action's own condition and context, and the search defaults.

### 6.1 Search items

Parsing the search view ([views and actions, section 7](views-and-actions.md#7-the-search-view)) produces an ordered list of **search items**, each with a type, a description, a group number and, when relevant, a field name and a condition.

| Item type | Produced by | Contributes |
|---|---|---|
| `field` | a `field` element | a condition built from the typed value |
| `field_property` | a `field` element on a user-defined-fields container | a condition on one property |
| `filter` | a `filter` element with a condition | that condition |
| `dateFilter` | a `filter` element with a date attribute | a condition over the selected periods |
| `groupBy` | a `filter` element whose context sets a grouping | a grouping |
| `dateGroupBy` | the same on a date field | a grouping with an interval |
| `favorite` | a saved filter | a condition, a grouping, an ordering and contextual keys, all at once |
| `comparison` | a period comparison (industry-standard completion, section 6.7) | a second period evaluated beside the first |

Items carry a **group number**: consecutive items of the same kind written without a separator share it.

### 6.2 The query

The **query** is the ordered list of activated items, each with the values that parameterize it: the typed values of a search field, the selected period options of a period filter, the selected intervals of a date grouping.

Activating an item appends it to the query; activating an already active item removes it. Activating an item of a group that already has an active item of a **different** kind replaces it when the kind is exclusive (a favourite replaces the whole query; a period option replaces the other options of the same custom set).

### 6.3 Composing the condition

The composed condition is the conjunction of: the action's own condition, which is the global condition; for each group of activated items, the disjunction of the conditions contributed by the activated items of that group; and the condition contributed by the side panel.


The rule is the same one stated in [views and actions, section 7.5](views-and-actions.md#75-how-filters-combine), applied to the runtime query rather than to the view description.

Worked example. The action's condition is `[("company", "=", 3)]`. The user activates the two filters `Draft` and `Posted` of one group, the filter `Late` of another group, and types `ACME` in the customer search field. The applied condition is:

> the company is 3, **and** the state is either draft or posted, **and** the delay is below 15, **and** the customer name contains "ACME" without regard to case


### 6.4 Composing the grouping, the ordering and the context

- **Grouping**: the concatenation of the groupings contributed by the activated items, in activation order, followed by the action's own grouping when the query contributes none. A date grouping contributes `<field>:<interval>` and, when several intervals are selected, one entry per interval ordered from the coarsest to the finest.
- **Ordering**: the ordering contributed by the active favourite, when there is one; otherwise the ordering the screen itself holds; when the screen is grouped and the user asked to order by group size, the ordering is by the group count, ascending or descending.
- **Context**: the merge of the action's context and the contexts contributed by the activated items, the later ones winning.

### 6.5 Facets

The search input shows one **facet** per group of activated items.

| Facet property | Value |
|---|---|
| `group` | The group the facet stands for. |
| `type` | `field`, `filter`, `groupBy`, `favorite` or `comparison`. |
| `values` | The descriptions of the activated items: the typed values for a search field; the item description for a filter; `<description>: <interval>` for a date grouping; `<description>: <period>` for a period filter. |
| `separator` | `>` for a grouping facet, the translated word `or` for every other kind. |
| `title` | The field label, for a search-field facet. |
| `icon`, `colour` | Per facet type. |
| `condition` | The composed condition of the group, shown in diagnostic mode. |

When the screen has a default grouping that no facet expresses, and the view kind is not a board, a read-only facet describing that default grouping is shown first.

Removing a facet removes every item of its group from the query. The search input also offers clearing the whole query, and clearing only the conditions while keeping the groupings.

### 6.6 Favourites

A favourite is a Saved Filter record ([views and actions, section 25](views-and-actions.md#25-saved-filters)).

| Field | Type | Meaning |
|---|---|---|
| `name` | `text`, required | The label. |
| `users` | `many_to_many` to User, delete **cascade** | Empty means shared with everyone; one entry means private to that user. |
| `domain` | `long_text`, required, default `[]` | The stored condition. |
| `context` | `long_text`, required, default `{}` | The stored contextual keys, including the grouping under the key `group_by`. |
| `sort` | `text`, required, default `[]` | The stored ordering, as a list of `<field>` or `<field> desc` entries. A value that is not a list is refused by the check `Invalid sort definition`. |
| `model_definition` | `selection` over the entity names, required | The entity the favourite applies to. |
| `is_default` | `boolean` | Whether it is applied automatically when the screen opens. |
| `action` | `many_to_one` to Action, delete **cascade** | The action it is scoped to. Empty means it applies to every screen on that entity. |
| `embedded_action` | `many_to_one` to Embedded Action, delete **cascade** | The embedded entry it is scoped to. |
| `embedded_parent_res` | `integer` | The parent record the entry is scoped to. Only meaningful together with an embedded entry; a value without one is refused. |
| `active` | `boolean`, default true | Archived favourites are not offered. |

Default ordering: entity, then name, then key descending.

**Loading.** The favourites offered on a screen are those whose entity matches, whose action is either the current action or empty, whose embedded entry matches exactly, whose parent record matches exactly, and whose user list contains the acting user or is empty.

**Saving.** Saving the current query as a favourite:

1. The context is the composed context merged with the contexts published by the screen.
2. Remove from that context every key that also exists in the user context, and every key whose name begins with the search-default prefix or the side-panel-default prefix.
3. The condition is the composed condition **without** the action's own condition.
4. The grouping is the composed grouping.
5. The ordering is the one published by the screen, and the composed ordering when the screen publishes none.
6. The users are the acting user alone for a private favourite, and nobody for a shared one.
7. Store a saved filter carrying the typed description as its name, the current action, the screen's entity, the condition, the default flag, the users, the ordering rendered as text, and a context holding the grouping and the remaining keys.


An empty description is refused with the notice `A name for your favorite filter is required.` The favourite editor opens the stored record in a form dialog.

Only one favourite can be active at a time: activating one deactivates the other and replaces the whole query.

### 6.7 Period comparison (industry-standard completion)

The reference behaviour offers, next to the period filters, a comparison control that evaluates the same aggregation over a shifted period and shows both series. The mechanism is not part of the base package examined here; the following contract is an **industry-standard completion**, consistent with the period filter rules of [views and actions, section 7.4](views-and-actions.md#74-the-filter-node), and a replacement should implement it that way:

1. The comparison control is offered only when a period filter is active.
2. Its options are `previous period` and `previous year`.
3. Activating an option adds a `comparison` facet and makes the screen evaluate its aggregation twice: once with the active condition, once with the same condition in which every leaf of the active period filter is shifted by one period or by one year.
4. Aggregating screens (the cross table and the chart) display both series and the signed relative variation `(current − comparison) ÷ |comparison|`, rendered as a percentage with one decimal, and display no variation when the comparison value is zero.
5. Screens that list records ignore the comparison.

### 6.8 The side panel

The side panel declared in the search view ([views and actions, section 7.6](views-and-actions.md#76-the-search-panel)) is loaded separately from the search items.

1. For each category section: fetch the distinct values of the field, ordered by the entity's own ordering and restricted to the section's limit; build the value tree, nesting children under parents when the hierarchy option is enabled; and prepend the synthetic value that stands for all of them.
2. For each filter section: fetch the values allowed by the section's condition, grouped by the section's grouping field when one is declared; and, when counters are enabled, fetch the number of matching records per value.


The panel's contribution to the condition is the conjunction of: for each category, `field child_of <selected value>` when the field is hierarchical and `field = <selected value>` otherwise, omitted when the synthetic "All" value is selected; and for each filter, `field in <selected values>`, omitted when nothing is selected.

Sections are reloaded whenever the part of the condition that does **not** come from the panel changes; a section is not reloaded because of its own selection, in order that the values of a section do not disappear as the user ticks them.

## 7. Loading a screen

### 7.1 Loading the views

1. Ask the server for the views of the entity, passing the list of view references and kinds and the options ([views and actions, section 14](views-and-actions.md#14-view-resolution-the-contract-a-client-relies-on)).
2. For each returned view, parse its description into a view model holding: the active operations — create, write, delete and duplicate; the field nodes with their widget, options and conditions; the buttons with their parameters; the default page size and the default group page size; the default ordering and the default grouping; and the kind-specific attributes of [views and actions, sections 4 to 12](views-and-actions.md#4-the-form-view).
3. Keep the field descriptions returned beside the views.


The answer is cached per the entity, the view list and the context that affects it and the cache is cleared whenever a view definition or a saved filter is created, written or deleted, and whenever the package fingerprint changes.

### 7.2 The data request

A screen never asks for "all fields". It builds a **field specification**: a nested mapping naming exactly the fields it renders and, for each relation, the fields of the related records it renders.

The specification is built recursively over the active fields:

1. Skip a field that is itself a property of a custom-properties container.
2. For a to-many field: when a sub-view exists and the field is not unconditionally invisible, the entry carries the specification of the sub-view's active fields, the field's context evaluated against the record, the sub-view's page size as the limit, and the sub-view's ordering when one is set.
3. For a many-to-one or a polymorphic link: the entry carries the specification of the sub-view's active fields when one exists, and otherwise asks only for the display name.
4. For any other field: the entry is empty, which asks for the value alone.
5. A field the acting user may not read never appears in the specification, because it never appears among the active fields.


An unconditionally invisible field (`invisible="True"` or `invisible="1"`) has its nested content dropped, but the field itself is still requested, because an expression elsewhere may read it. During a recomputation round trip the nested content of such a field **is** requested, because the server needs the whole picture to compute.

Three operations consume the specification:

| Operation | Used for | Contract |
|---|---|---|
| `web_search_read(entity, condition, specification, offset, limit, order, count_limit)` | An ungrouped multi-record screen. | Returns the records and the total count. When a count limit is given, the count may be returned as "at least that many". |
| `web_read_group(entity, condition, grouping, aggregates, ...)` | A grouped multi-record screen. | Returns the groups with their aggregates and their counts. |
| `web_read(entity, record keys, specification)` | A record screen and the records of a group being opened. | Returns the records. |

All three are called with the contextual key that makes binary fields return their size instead of their content, in order that a form does not download every attachment.

### 7.3 Paging and counting

A multi-record screen holds an offset, a page size and a count limit.

| Rule | Behaviour |
|---|---|
| Initial page size | The view's `limit`, else `80` for a top-level table, `40` for a table embedded in a relation field, `40` per column for a grouped board. |
| Initial count limit | The view's `count_limit`, else `10000`. |
| Effective count limit | Raised to at least `offset + limit`, in order that the current page can always be counted. |
| Requested count | `count_limit + 1`, in order that the screen can tell "exactly n" from "more than n". |
| Empty page | When an offset beyond the last record yields no record, the offset is reset to zero and the request is repeated once. |
| Grouped screens | The group page size is the view's `groups_limit`, else `80` when groups open collapsed and `10` when they open expanded. |

### 7.4 Demonstration records

When a view declares the demonstration flag and the query returns nothing, the screen fills itself with generated records. The generation rules are fixed, in order that the same screen always shows the same plausible data:

| Field | Generated value |
|---|---|
| a display name or a `name` field on a people-shaped entity (users, contacts, employees, followers, mailing contacts) | one of five sample person names, chosen from the record's position |
| a display name or a `name` field on the country entity | one of five sample country names |
| a display name on any other entity | one of five short sample phrases |
| a `name` or `reference` field | the text `REF` followed by the record's position padded to four digits |
| a field whose name contains `description`, `label`, `title`, `subject` or `message` | one of the five short sample phrases |
| a field whose name contains `email` | the sample person name, lowercased, spaces replaced by a dot, followed by `@sample.demo` |
| a field whose name contains `phone` | `+1 555 754 ` followed by the record's position padded to four digits |
| a field whose name contains a web address | `http://sample<n>.com` |
| any other single-line or multi-line text field | empty |
| a date or a date and time | a random moment within sixty days of now, before or after |
| a decimal | a random number between zero and one hundred, with two decimals |
| an integer | a random number between zero and fifty; when the field name contains `color`, either zero or a random number between zero and seven |
| a monetary value | a random number between zero and one hundred thousand |
| a link to one record, to the currency entity | the first currency |
| a link to one record, to the attachment entity | empty |
| any other link to one record | a random key drawn from a small fixed range |
| a list of records or an association | two random keys drawn from that same range, duplicates removed |
| a selection | a random allowed value |
| anything else | empty |

The generated records are inert: any real operation (creating a record, adding a column, changing the search) discards them and re-runs the real query.

## 8. The record screen

### 8.1 States

| State | Meaning |
|---|---|
| new | The record has no key. Its values come from the defaults round trip. |
| clean | The record has a key and no pending change. |
| dirty | The record has at least one pending change. |
| invalid | At least one required, visible field is empty, or a nested record is invalid. |

### 8.2 Loading

1. When the record key is empty, obtain the values by the defaults round trip: run the recomputation operation with no records, no changes and no changed field names, passing the specification.
2. Otherwise read the record with the specification; when the record does not exist, raise the fetch failure naming the key.
3. Build the record object from the server values, an empty change set, and the evaluation context — the field values plus the context, the acting user's identifier, the current date, the current instant, and the parent record when the record is nested.


### 8.3 Changing a value

1. Mark the record dirty.
2. Normalise the changes: a many-to-one given as a display name only is created on the fly through a name-based creation round trip and replaced by the created record; a many-to-one given as a key only is completed with its display name through a read round trip; a polymorphic link is completed the same way; a to-many change is turned into the command list of [section 10.1](#101-the-commands); a custom-properties change is normalised against its definition; and a markup change is normalised.
3. When the record is selected and the screen is in multiple-record editing mode, hand the change to the multiple-record path of [section 9.4](#94-editing-several-records-at-once) instead of applying it here.
4. Apply the changes to the change set.
5. When any changed field is declared as triggering a recomputation, run the round trip of [section 8.4](#84-the-recomputation-round-trip) and merge its answer.
6. Re-evaluate the conditions of every node against the new values.


A change to a link to one record that resolves to the same key and the same display name is dropped before the round trip, in order that merely re-selecting the same value does not mark the record dirty.

### 8.4 The recomputation round trip

1. The context is the record's context; when exactly one field changed, that field's own context is merged in.
2. Invoke the server's recomputation operation with the record keys, the changes, the names of the changed fields and the specification.
3. When the answer carries a warning of the dialog kind, open a dialog with its title and message; any other warning raises a sticky warning notice with its title and message.
4. Return the answer's values.


The specification sent during a recomputation includes the nested content of unconditionally invisible relation fields, unlike the specification used for reading.

For a nested record, the changes sent also carry the pending changes of the **parent** under the name of the inverse field, plus the parent's key when the parent already exists, in order that a computation that reads the parent sees the values the user typed rather than the stored ones.

### 8.5 Validity

A record is valid when every active field that is neither invisible nor a property satisfies its rule:

| Field kind | Rule |
|---|---|
| Boolean, integer, decimal number, monetary amount | Always valid |
| Markup | Invalid when required and the value has no length |
| To-many | Invalid when required and the list is empty, or when any dirty nested record is itself invalid |
| Custom properties | Invalid when any property definition lacks a name or a label |
| Structured document | Invalid when required and the value has no key |
| Anything else | Invalid when required and the value is empty |


When a save is attempted on an invalid record, the fields at fault are marked and the notice `Missing required fields` is raised with the failure styling.

### 8.6 Saving

1. Abandon every nested record that is new, untouched and invalid.
2. If the record is not valid, show the missing-required-fields notice and stop, reporting failure.
3. The changes are the pending changes without the record key.
4. If the record already exists and there is no change: when a next record key was asked for, load that record and stop; otherwise drop the change set, mark the record clean and stop, reporting success.
5. If the screen is being closed urgently, take the path of [section 8.9](#89-leaving-the-page) instead.
6. Ask the screen's hook whether the save may proceed; stop when it refuses.
7. Invoke the combined write-and-read operation with the entity, the record key when it exists, the changes, the specification and, when one was asked for, the next record key.
8. On success: replace the server values with the answer, drop the change set, mark the record clean, and load the next record when one came back.
9. On failure: keep the change set, keep the record dirty, and report the failure to the caller so that the leave protocol of [section 8.8](#88-leaving-a-screen-with-pending-changes) can present it.


`web_save` creates the record when the key list is empty and updates it otherwise, and returns the record read back through the specification, which is what lets one round trip both write and refresh the screen.

### 8.7 Discarding

Discarding restores the values the record had when it was loaded or last saved, drops every pending change including those of nested records, clears the invalid marks, and, for a record that was never saved, abandons it.

### 8.8 Leaving a screen with pending changes

1. If the record is not dirty, or the leave is forced, allow the navigation.
2. Otherwise attempt to save without reloading.
3. On failure, present the failure dialog: its title is **"Oh snap!"** and its body is the failure message. The dialog offers **"Stay here"**, which refuses the navigation; **"Discard changes"**, which discards the change set and allows the navigation; and, when the failure carries a follow-up action, a redirect button that runs that action with the leave forced and refuses the original navigation.


When the failure is a permission failure carrying a suggested company the user is allowed to act for but has not activated, that company is activated silently and the save is retried once instead of showing the dialog.

### 8.9 Leaving the page

When the browser announces that the page is being closed, the record attempts an **urgent save**: the change set is sent through the fire-and-forget transport, which cannot be cancelled. When the payload is too large for that transport, the save does not happen, the page closing is cancelled, and the sticky notice `Heads up! Your recent changes are too large to save automatically. Please click the <upload icon> button now to ensure your work is saved before you exit this tab.` is raised. When the tab merely becomes hidden and no form dialog is open, an ordinary save is attempted.

During an urgent save the recomputation round trips are skipped, because there is no time for them.

### 8.10 Concurrency

Two screens showing the same record are kept in step: after a save, every other loaded record with the same entity and key receives the returned values. Two users editing the same record are not serialised by the client; the server's last-writer-wins rule applies, except where an entity declares a concurrency guard (see [record operations and query notation](record-operations-and-query-notation.md)). A save that fails because the record was deleted meanwhile raises the fetch failure and the screen reloads its record list without the missing key.

## 9. The table screen

### 9.1 Reading

A table screen loads the page described in section 7.3 and renders one row per record. A grouped table loads the groups first and opens a group by loading its records with the group's own condition.

### 9.2 Editing in place

A table whose view declares `editable` puts the addressed row into edit mode.

1. If another row is being edited, attempt to leave it ([section 9.3](#93-leaving-a-row)); abandon the gesture when that fails.
2. Put the row in edit mode and place the focus in its first editable cell.


A new row is inserted at the top or at the bottom according to the `editable` value, is created through the defaults round trip, and is abandoned when it is left untouched.

### 9.3 Leaving a row

1. When the caller asks to discard, discard the row's changes.
2. When the row is new and untouched, abandon it.
3. When the caller asks to validate and the row is invalid, mark the offending fields and stay on the row.
4. Otherwise save the row.


Clicking the discard button is detected on the button press rather than on the release, in order that leaving the cell does not save the row before the discard is registered.

### 9.4 Editing several records at once

When the view declares the multiple-record editing flag and more than one row is selected, changing a field of the edited row proposes to apply the change to every selected row.

1. Ask the screen's hook whether the save may proceed.
2. For each changed field that holds many records: take the command list built on the edited row; add the display name to each link command, so that the other rows do not have to read it back; and apply the same command list to the same field of every other selected row.
3. Apply the changes to every selected row.
4. Split the selected rows into valid and invalid: a row is valid when none of the changed fields is read-only on it and the row passes the validity check.
5. If no row is valid, raise the missing-required-fields notice, discard the invalid rows and stop.
6. Otherwise ask for confirmation, stating how many records will change, and on confirmation save the valid rows in one call; on refusal discard every change.
7. On a failure, discard the changes of every row and reload the affected rows, so that the table never shows a value the server rejected.


The confirmation dialog is titled `Confirmation` and states:

- `This update will only consider the records of the current page.` when the selection was extended to every matching record;
- `Among the %(total)s selected records, %(valid_count)s are valid for this update.` when some rows were excluded;
- `Are you sure you want to update %(count)s records?` always;
- then a table listing, per changed field, either `Field: <label>` and `Update to: <rendered value>` (or `None` when the value is empty), or, for an association, the rows `Add:` and `Remove:` with the affected records as tags.

### 9.5 Reordering rows

A table containing a handle widget lets the user drag rows. Dropping a row rewrites the ordering field of every record between the old and the new position, in one batch, and the screen re-reads the affected rows. Reordering is refused when the table is grouped by something other than the ordering field, or when the view forbids modification.

### 9.6 Optional columns

A column declared optional can be shown or hidden from the column chooser. The user's choice is stored per entity and view in the browser and is restored on the next visit. A column hidden this way is still part of the field specification only when another element needs it.

## 10. Editing a list of related records

A field holding many related records is not written as a list of keys. It is written as an ordered **command list**, which is what lets the server distinguish "create this new line", "modify that line", "detach that line" and "delete that line".

### 10.1 The commands

| Command | Written as | Meaning |
|---|---|---|
| create | `(0, placeholder, values)` | Create a new related record with those values and link it. |
| update | `(1, key, values)` | Write those values on the linked record. |
| delete | `(2, key)` | Unlink **and delete** the related record. |
| detach | `(3, key)` | Unlink the related record, leaving it in place. |
| link | `(4, key)` | Link an existing record. |
| clear | `(5)` | Unlink every currently linked record. |
| replace | `(6, ·, keys)` | Replace the whole set of linked records by those keys. |

### 10.2 Which commands each editing gesture produces

| Gesture | List of records (`one_to_many`) | Association (`many_to_many`) |
|---|---|---|
| Add a new line inline | create | create |
| Edit a line | update | update |
| Remove a line with the row control | delete | detach |
| Pick an existing record | link | link |
| Unpick a tag | detach | detach |
| Empty the whole field | clear | clear |
| Replace the whole set (for instance from the multiple-record editor) | replace, followed by the update commands that survive | the same |

The difference between the two kinds is the removal gesture: removing a line of a list of records **deletes** it, because such a line has no existence outside its parent, whereas removing an entry of an association only **detaches** it.

### 10.3 Normalisation before sending

1. For each stored command: an update command addressed at a record that was never loaded is expanded into the commands the server originally sent for it, with their values converted back to the transport form.
2. A create command whose record has meanwhile been saved — because a button inside the line's dialog saved it — becomes a link command on the resulting key.
3. A create or update command contributes only when it carries values, except a create command, which always contributes.
4. Every other command is kept as it is.


### 10.4 Editing modes

| Mode | When | Behaviour |
|---|---|---|
| Inline table | The field renders a table sub-view that declares `editable` | Rows are edited in place, exactly as in section 9.2. |
| Dialog | The field renders a table sub-view that is not editable, or a board | Activating a line opens the form sub-view in a dialog whose buttons are `Save & Close`, `Save & New` (when creation is allowed) and `Discard`, plus `Remove` when the line may be removed. |
| Tags | The field uses a tag widget | Typing searches the related entity; selecting adds a link command; removing a tag adds a detach command. |
| Tick boxes | The field uses the tick-box widget | Each candidate record is one box; ticking links, unticking detaches. |

### 10.5 Nested validity and saving

A nested record is validated as part of its parent (section 8.5). A nested record that is new, untouched and invalid is abandoned rather than blocking the parent's save. The whole command list is sent inside the parent's single write; nested records are never saved independently, which is what makes a form with lines atomic.

### 10.6 Ordering inside the list

A list sub-view keeps its own ordering. Sorting it by a column re-sorts the loaded records in memory when every record is loaded, and issues a fresh read when the list is paged. The ordering is preserved across a save of the parent, unless the save navigates to another record.

## 11. Navigating between records

### 11.1 The pager

The pager shows the interval `[offset + 1, min(offset + page size, total)]` when the page size is greater than one, and the single number `offset + 1` otherwise, followed by the total. The total is rendered as `<n>+` when it is a lower bound rather than an exact count; activating it asks the server for the exact count.

Controls: previous page, next page, and a text field in which the user may type `min`, `min-max`, `min,max` or `min;max`. Typed bounds are clamped into `[1, total]`; an inverted interval is normalised; a single number selects a page of one record. Paging wraps: the next control on the last page returns to the first.

### 11.2 Paging on a record screen

On a record screen reached from a multi-record screen, the pager walks the record keys of that screen. Moving to another record:

1. If the record is dirty, save it and ask the server to return the **next** record in the same round trip; on failure, present the dialog of [section 8.8](#88-leaving-a-screen-with-pending-changes).
2. Otherwise load the next record.
3. If the record cannot be fetched, reload the list without the missing keys.


Asking the server for the next record inside the save is what makes "save and go to the next record" a single round trip.

### 11.3 Keeping the position

A screen exports its state when it is left: the offset, the page size, the ordering, the opened groups, the selected rows, the scroll position and, for a record screen, the current record key. Returning to it through the navigation trail restores that state.

## 12. Feedback mechanisms

### 12.1 Notices

The notice service shows transient panels stacked in a corner.

| Option | Default | Meaning |
|---|---|---|
| `title` | none | An optional heading. |
| `type` | neutral | One of `info`, `success`, `warning`, `danger`. |
| `sticky` | false | When false the notice closes by itself after the automatic-close delay (`4000` milliseconds); when true it stays until dismissed. |
| `autocloseDelay` | `4000` | The delay in milliseconds. |
| `className` | empty | Extra presentation classes. |
| `buttons` | empty | A list of `{ name, icon, primary, action }` entries rendered as controls inside the notice. |
| `onClose` | none | Invoked when the notice is removed, whether by the delay or by the user. |

Adding a notice returns a procedure that removes it, which is how a long-running mechanism replaces its own notice instead of stacking a new one.

### 12.2 Dialogs

The dialog service stacks modal dialogs; only the topmost one receives keyboard focus, and keyboard navigation is trapped inside it. A dialog declares a size (`small`, `medium`, `large`, `extra large`, `full screen`), a title, a body and a footer. Closing invokes the completion callback given when it was opened.

The confirmation dialog is the standard shape: a title (default `Confirmation`), a body, an accepting button and a refusing button (default caption `Cancel`). The default caption of the accepting button is a two-letter affirmative acknowledgement in the reference behaviour; because this specification writes no abbreviations, it is named `Confirm` here, and a replacement may use either spelling as long as it is used consistently. The alert dialog is the same with only the accepting button and the default title `Alert`.

### 12.3 Celebration effects

The celebration service plays a registered effect. The only built-in effect draws a banner with a message and an image, and:

| Parameter | Default | Meaning |
|---|---|---|
| `type` | the banner effect | Which registered effect to play. |
| `message` | `Well Done!` | The text shown. |
| `img_url` | the default smiling image | The image shown. |
| `fadeout` | `medium` | `fast`, `medium`, `slow` or `no`; `no` keeps the effect until the user dismisses it. |

When the user's celebration switch is off, the effect is not drawn and the message is raised as an ordinary notice instead.

### 12.4 Failure handling

Every unhandled failure is offered to the registered failure handlers in sequence order; the first one that claims it stops the chain.

| Handler | Claims | Behaviour |
|---|---|---|
| server failure | A failure carrying a server payload | When the failure name matches an entry of the failure-notice registry, the registered notice is raised instead of a dialog. Otherwise the dialog registered for that failure name, or for the failure class named in the payload's context, is opened; when nothing is registered, the generic server failure dialog is opened. |
| lost connection | A failure meaning the request never completed | Raises the sticky notice `Connection lost. Trying to reconnect...` once, then probes the server every `2000` milliseconds with an exponentially growing delay multiplied by `1.5` plus a random jitter of up to `500` milliseconds. On success the sticky notice is removed and the informational notice `Connection restored. You are back online.` is raised. |
| oversized request | A failure meaning the payload exceeded the accepted size | Opens the dialog titled `The request sent to the server was too large`. |
| client failure | Anything else | Opens the generic client failure dialog. |

The generic failure dialog shows the message, a control captioned `See technical details` / `Hide technical details` that reveals the technical trace, and a copy control that answers `Copied`. The titles per kind are the platform's generic failure titles for a server failure, a client failure, a network failure and a warning.

The dialog titles used for the common server failures are: `Access Denied`, `Access Error`, `Missing Record`, `Missing Action`, `Invalid Operation` (used both for a business-rule refusal and for a server action that has warnings), `Validation Error`, `Warning`, and the mail-delivery failure title.

### 12.5 The busy indicator

Two distinct mechanisms exist.

**The loading indicator** watches every non-silent request. When the number of pending requests goes from zero to one, a timer of `250` milliseconds starts; if requests are still pending when it fires, a small panel appears showing the word `Loading` and the number of pending requests. It disappears as soon as no request is pending.

**Blocking** is explicit: a caller blocks the interface and must unblock it. Blocking is counted, which makes nested blocks safe; unblocking more often than blocking logs the warning `Unblock was called more times than block, you should only unblock the interface if you have previously blocked it.` and resets the counter to zero. While blocked, an overlay prevents interaction and shows a sequence of reassuring messages as time passes: after 20 units `Loading...`; after 40 `Still loading...`; then `Still loading...` / `Please be patient.`; then `Don't leave yet,` / `it's still loading...`; then `You may not believe it,` / `but the application is actually loading...`; then `Take a minute to get a coffee,` / `because it's loading...`; then `Maybe you should consider reloading the application by pressing F5...`.

## 13. The top bar

### 13.1 Composition

From left to right: the application switcher, the current application's name, the entries of the current application's sub-tree, a "more" menu holding the entries that do not fit, then the right-hand area holding every entry of the system-tray registry in ascending sequence, ending with the user menu.

The built-in system-tray entries are:

| Entry | Sequence | Content |
|---|---|---|
| the compact menu | `0`, narrow viewports only | Replaces the whole left part of the bar with a single control opening a panel that holds the application list, the current application's entries and the user menu. |
| the company switcher | `1` | Present only when the user is allowed to act for more than one company. |
| the user menu | `0`, wide viewports | The avatar and the user menu. |

Capability packages add further entries, for instance the conversation indicator and the call indicator.

### 13.2 The company switcher

Shows the tree of allowed companies with a tick per company and a highlight on the acting company. Rules:

1. At least one company must stay active; unticking the last one is refused.
2. Ticking a parent ticks every allowed descendant; unticking a parent unticks them.
3. Companies the user is not allowed to act for are drawn for context but cannot be ticked.
4. Confirming writes the new set into the acting context and reloads the current screen, in order that every subsequent query is scoped to the new set.

The company scoping rules themselves are in [multi-company](multi-company.md). The keyboard accelerator of the switcher is the accelerator modifier together with shift and `u`.

### 13.3 The user menu

The entries, in sequence order:

| Sequence | Entry | Behaviour |
|---|---|---|
| `20` | `Help` | Opens the support address from the session payload in a new window. |
| `30` | `Shortcuts` | Hidden on a narrow viewport. Shows the command modifier together with `K` beside its label and opens the command palette with a footer describing the shortcuts. |
| `40` | a separator | |
| `50` | `My Preferences` | Opens the acting user's own record in the preferences form. |
| `60` | `My Account` | Opens the publisher's account page in a new window; when the address cannot be obtained, a default account address is used. |
| `65` | `Install App` | Shown only when the page can be installed as a standalone application. On a small set of applications it installs that application in its own scope; otherwise it installs the whole back office. |
| `70` | `Log out` | Ends the session and navigates to the login page; when the page runs as a scoped application, the address it returns to is that application's start address. |

## 14. The command palette

### 14.1 Opening

The palette opens with the command modifier together with `K`, and from the `Shortcuts` entry of the user menu. It is a dialog with one text input and a grouped result list.

### 14.2 Namespaces

The **first character** typed selects a namespace when it is registered as one.

| Namespace | Prompt | Empty message | Provides |
|---|---|---|---|
| (default) | `Search for a command...` | `No command found` | Every command registered by the current screen and every visible keyboard-accelerated control. |
| `/` | `Search for a menu...` | `No menu found` | Applications and navigation entries. |
| `@` | contributed by the messaging package | | People and conversations. |

Further namespaces are contributed by capability packages.

### 14.3 Providers

| Provider | Contributes |
|---|---|
| registered commands | Every command the current screen registered, with its category, its label, its optional accelerator and its procedure. Commands whose availability predicate is false are dropped; duplicates within one category are dropped. |
| accelerated controls | Every visible, enabled control carrying an accelerator. The label is the control's tooltip, its title, its hint text, or the first fifty characters of its text, truncated with an ellipsis, or `no description provided`. The category is taken from the nearest ancestor that declares one, and a control under the category `disabled` is skipped. The accelerator shown is the accelerator modifier plus the control's own character. |
| navigation | Under the namespace `/`: applications matched loosely against the typed text, and navigation entries matched loosely against their full path read from the leaf upwards. |
| diagnostics | The diagnostic commands, when diagnostic mode is active. |

### 14.4 Categories and ordering

Results are grouped by category, and categories are shown in this order: `app` (10), `smart_action` (15), `actions` (30), `default` (50), `view_switcher` (100), `debug` (110). Under the navigation namespace the categories are `apps` (10) and `menu_items` (20). A command whose category is unknown falls back to `default`.

### 14.5 Behaviour

Typing filters the results; under a namespace that declares loose matching, the match is a subsequence match rather than a substring match. Arrow keys move the selection, Enter runs the selected command, Escape closes the palette. Running a command that is an accelerated control focuses and activates that control. On a macOS-style keyboard the displayed accelerators translate the control modifier to the command key and the accelerator modifier to the control key.

## 15. Keyboard shortcuts

### 15.1 The accelerator mechanism

An accelerator is expressed as a sequence of modifiers and one key, joined by `+`, in the fixed order `alt`, `control`, `shift`, key. The modifiers are normalised per platform: on a macOS-style keyboard the physical control key means `alt` and the physical command key means `control`, which is what makes the same declaration work everywhere.

Accepted keys: the letters `a` to `z`, the digits `0` to `9`, the navigation keys (`arrowleft`, `arrowright`, `arrowup`, `arrowdown`, `pageup`, `pagedown`, `home`, `end`, `backspace`, `enter`, `tab`, `delete`, `space`), `escape`, `<` and `>`. Digits are read from the physical key position, and a letter that is not produced by the current keyboard layout is also read from the physical position, in order that a non-Latin keyboard layout still triggers the shortcuts.

A registration may declare: that it repeats while the key is held; that it fires even while an editable element has focus; that it is global rather than scoped to the topmost interactive area; a restricted area; an availability predicate; and an element on which an overlay badge is drawn. Holding the accelerator modifier alone reveals those badges over every control that declares an accelerator.

Only registrations belonging to the topmost interactive area fire; a dialog therefore suppresses the shortcuts of the screen behind it.

### 15.2 The catalogue

All of the following are pressed together with the accelerator modifier.

| Shortcut | Where | Effect |
|---|---|---|
| `h` | top bar | Open the application switcher. |
| `b` | navigation trail | Return to the previous trail item. |
| `c` | record screen, table screen | Create a new record. |
| `c` | record dialog, picker dialog | `Save`, `Save & Close`, or `New` in a picker. |
| `s` | record screen, editable table | Save. |
| `j` | record screen, editable table, dialogs | Discard, `Cancel`, or `Close`. |
| `n` | record dialog | `Save & New`. |
| `n` | pager | Next page. |
| `p` | pager | Previous page. |
| `k` | related-record dialog | `Remove`. |
| `x` | record screen | Remove the line (inside a relation dialog). |
| `x` | confirmation dialog | The refusing button. |
| `q` | confirmation dialog | The accepting button. |
| `u` | screen | Open the actions menu. |
| `shift+u` | screen | Open the actions menu of the selection. |
| `shift+q` | control panel | Open the search menu. |
| `v` | export dialog, picker dialog, scale selector | The confirming control (`Export`, `Select`, the scale list). |
| `z` | export dialog, picker dialog | `Close`. |
| `shift+u` | top bar | Open the company switcher. |

Shortcuts used **without** the accelerator modifier: `Escape` closes the topmost dialog, popover or drop-down; `Enter` validates the focused control; `Control+Enter` validates a multi-line input; the arrow keys move between cells of an editable table and between entries of a drop-down; `Tab` moves to the next editable cell and, on the last cell of the last row, creates the next row in an editable table.

## 16. The service catalogue

| Service | Depends on | Contract |
|---|---|---|
| `orm` | not applicable | The typed record operations: `call`, `create`, `read`, `write`, `unlink`, `search`, `searchRead`, `searchCount`, `webSearchRead`, `webRead`, `webSave`, `webSaveMulti`, `webReadGroup`, `formattedReadGroup`, `formattedReadGroupingSets`, `webResequence`. Every argument is validated before the request: an entity name must be a non-empty text, a key list must be a list of integers, a field list must be a list of texts, a condition must be a list, a values mapping must be a mapping. A cached variant returns a previously received answer while refreshing it in the background. |
| remote call transport | not applicable | Issues one request, announces its start and its completion on a bus (which the loading indicator and the caches listen to), and turns a server failure payload into a typed failure carrying the failure name, the message, the code, the sub-kind and the payload. Recognized transport failures: lost connection, aborted request, oversized request. Settings accepted per call: `silent` (do not count in the loading indicator), `cache`, `headers`, and a raw-transport switch. |
| `action` | `dialog`, `effect`, `localization`, `notification`, `title`, `ui` | The action manager of section 4. Public operations: `doAction`, `doActionButton`, `switchView`, `restore`, `loadState`, `loadAction`, and read-only access to the current screen and the current action. |
| `menu` | not applicable | The navigation tree of section 3. |
| `view` | `orm` | Loads and caches view models (section 7.1). |
| `field` | `orm` | Loads and caches field descriptions per entity, for the condition editor and the field pickers. |
| `name` | `orm` | Batches display-name lookups: several widgets asking for the display name of related records in the same tick produce one request. |
| `notification` | not applicable | Section 12.1. |
| `dialog` | not applicable | Section 12.2. |
| `effect` | overlay | Section 12.3. |
| `error` | not applicable | Section 12.4. |
| `ui` | not applicable | The busy indicator, the active-area stack and the viewport size classes. `isSmall` is true at the two smallest sizes. |
| `hotkey` | `ui` | Section 15. |
| `command` | not applicable | Registers and lists the commands of the current screen, and opens the palette. |
| `popover`, `tooltip`, `overlay` | not applicable | Positioned transient layers. |
| `title` | not applicable | Owns the page title, composed of named parts, in order that several mechanisms can contribute without overwriting each other. |
| `localization` | not applicable | The language's formatting data: date and time patterns, decimal and thousands separators, direction, week start, and the number grouping. |
| `currency` | not applicable | The currencies from the session payload, and the monetary formatting rules. |
| `user` | not applicable | The acting user: key, name, login, language, time zone, group membership tests (cached per group), the active companies, the celebration switch, and the settings record. |
| `company` (the company switcher's model) | `user` | The allowed companies, the acting company, the activation procedure and the reload it triggers. |
| `http` | not applicable | Plain requests for non-record endpoints, including file uploads. |
| `file_upload` | `notification` | Tracks uploads, shows their progress and reports failures. |
| `datetime_picker` | `localization` | The shared calendar and clock popovers. |
| `bottom_sheet`, `sortable` | `ui` | Narrow-viewport panels and drag-and-drop. |
| `profiling` | `orm` | The performance recorder. |
| `lazy_session` | not applicable | Fetches the parts of the session payload that are deliberately deferred. |
| `reloadCompany` | `action` | Reloads the current screen after the acting company set changes. |
| notification bus | transport | Subscribes to server-pushed messages on named channels and dispatches them to listeners. It reconnects with a growing delay, announces its connection state, and, when several tabs of the same session are open, elects one tab to hold the connection and relays the messages to the others. Contributed by the messaging infrastructure package; it is the mechanism behind live counters, incoming messages and the "a new version is available, please refresh" notice. |

## 17. The point of sale client

The point of sale is a **separate client**: it is a client action that replaces the whole page, it keeps its own copy of the data it needs, and it is designed to keep selling while the network is unavailable.

### 17.1 Local storage of the working set

On startup the client asks the server for the description of every entity it uses (its fields and its relations), stores that description in the browser's key-value storage under a key derived from the point of sale configuration, and creates a local database in the browser whose stores mirror those entities. It then loads the working set: the configuration, the price lists, the taxes, the products, the partners it is allowed to see, the open session and its orders and order lines.

The local database name is derived from the point of sale configuration key and the data store name, in order that two configurations on the same device do not share data.

### 17.2 Detecting the connection state

1. Cancel any pending re-check.
2. Assume the client is online.
3. Send the lightweight probe request.
4. On success, run the synchronisation of [section 17.4](#174-synchronising) and announce that the client is online.
5. On a lost-connection failure, mark the client offline; and when the browser still claims to be online, schedule another check two seconds later.


The check also runs on the browser's own online and offline announcements. The probe exists because a browser reports itself offline when no interface is connected even though the server may still be reachable, and reports itself online when an interface is connected even though the server may be unreachable.

### 17.3 Behaviour while offline

| Area | Behaviour |
|---|---|
| Entity descriptions | Read from the browser's key-value storage instead of from the server. |
| Reading records | Served from the local database. A record that is not there cannot be fetched; the client does not attempt the round trip. |
| Following relations | The recursive fetch of missing related records is skipped entirely. |
| Creating and changing orders | Performed locally. Every change is written to the local database after a short idle delay, grouped, in order that a crash loses at most that delay. |
| Sending an order | The request is attempted; when it fails with a lost connection and the operation is marked as queueable and is not already queued, it is appended to the pending queue with a timestamp, a try counter and a unique marker, and the failure is swallowed. |
| Operations that are not queueable | The failure is raised to the caller, which shows it. |
| Opening a session | Refused while offline when the session is not already open locally. |
| Leaving the page | Refused while there are paid orders written locally but not yet confirmed by the server, in order that a reload cannot lose a paid order. |

### 17.4 Synchronising

When connectivity returns, the pending queue is replayed in order. Each entry is retried; a successful entry is removed. Entries that keep failing keep their try counter, which is what lets the interface show that something is stuck. The client also reconnects its push channels and re-subscribes to the channels it had.

### 17.5 Identifiers

Records created offline receive a locally generated marker rather than a server key, built from a per-device sequence obtained when the device was first registered. The marker is replaced by the server key when the record is synchronised, and every local reference to it is rewritten. This is what allows two devices working offline to create orders without colliding.

## 18. Acceptance criteria

**Bootstrap and navigation**

**AC-CLI-1.** *Given* a user whose start-screen field is empty, *when* the page is opened at the root address, *then* the first application of the navigation tree is opened.
**AC-CLI-2.** *Given* a user whose start-screen field names an action, *when* the page is opened at the root address, *then* that action is run.
**AC-CLI-3.** *Given* an address naming an action and a record, *when* the page is loaded, *then* the navigation trail has two items, the first being the action's multi-record screen and the second the record, and the record screen is displayed.
**AC-CLI-4.** *Given* an address naming an action that no longer exists, *when* the page is loaded, *then* the client retries with the trail shortened by one item, and *when* nothing remains it opens the user's start screen.
**AC-CLI-5.** *Given* a navigation entry that is an application root, *when* it is selected, *then* the navigation trail contains exactly one item.
**AC-CLI-6.** *Given* a screen with a dirty record, *when* another navigation entry is selected and the user chooses `Stay here`, *then* the screen is unchanged and the navigation did not happen.

**Actions**

**AC-CLI-7.** *Given* a window action offering a table and a form, *when* it is run, *then* the table is displayed and the trail has one item.
**AC-CLI-8.** *Given* that table, *when* a row is activated, *then* the form is displayed and the trail has two items.
**AC-CLI-9.** *Given* that form, *when* the view switcher is used to return to the table, *then* the trail has one item again, not three.
**AC-CLI-10.** *Given* a window action whose target is `new`, *when* it is run, *then* it is rendered in a dialog and the controller stack is unchanged.
**AC-CLI-11.** *Given* a window action whose target is `main`, *when* it is run, *then* the navigation trail contains only that action.
**AC-CLI-12.** *Given* a server action returning a window action, *when* a button of type `action` naming it is activated, *then* the returned window action is run and its context carries `active_model`, `active_id` and `active_ids`.
**AC-CLI-13.** *Given* a button with `type="object"` whose operation returns nothing, *when* it is activated, *then* a close window action is produced: a dialog closes, or nothing visible happens outside a dialog.
**AC-CLI-14.** *Given* a report action of the portable-document kind, *when* it is run, *then* the file is downloaded and, *when* the action carries the closing flag, the current dialog closes.

**Search**

**AC-CLI-15.** *Given* the search state of the worked example of section 6.3, *when* the screen queries the server, *then* the condition sent is exactly the composed one.
**AC-CLI-16.** *Given* two filters of the same group both active, *when* one is deactivated, *then* the remaining condition is that of the other filter alone, without the disjunction.
**AC-CLI-17.** *Given* a favourite marked as default on the entity with no action, *when* any screen on that entity opens, *then* the favourite is active and its condition, grouping and ordering are applied.
**AC-CLI-18.** *Given* an active favourite, *when* another favourite is activated, *then* the first is deactivated and the whole query is replaced.
**AC-CLI-19.** *Given* a query with a grouping and a condition, *when* it is saved as a favourite with a typed name, *then* a Saved Filter is stored whose condition excludes the action's own condition, whose context holds the grouping under the key `group_by`, and which contains no key of the acting user's context and no preset search key.
**AC-CLI-20.** *Given* a favourite saved with no name, *when* saving is attempted, *then* the notice `A name for your favorite filter is required.` is raised and nothing is stored.
**AC-CLI-21.** *Given* a side panel category with a hierarchical field, *when* a value is selected, *then* the contributed condition uses the descendant operator and includes the descendants.

**Record screen**

**AC-CLI-22.** *Given* a form with a required, visible, empty field, *when* saving is attempted, *then* no request is issued, the field is marked and the notice `Missing required fields` is raised.
**AC-CLI-23.** *Given* a form with a required field that is invisible, *when* saving is attempted, *then* the emptiness of that field does not block the save.
**AC-CLI-24.** *Given* a field carrying the recomputation flag, *when* its value is changed, *then* exactly one recomputation round trip is issued, carrying every pending change and the name of the changed field.
**AC-CLI-25.** *Given* a field **not** carrying the recomputation flag, *when* its value is changed, *then* no round trip is issued.
**AC-CLI-26.** *Given* a link-to-one-record field into which a name that matches no record is typed and confirmed as a new record, *then* a record is created from that name and the field holds it.
**AC-CLI-27.** *Given* a saved record with no pending change, *when* saving is requested, *then* no request is issued and the record stays clean.
**AC-CLI-28.** *Given* a new record, *when* it is saved, *then* one write request creates it, the returned key becomes the record's key, and the returned values replace the screen's values.
**AC-CLI-29.** *Given* a dirty record and a save that fails with a business-rule refusal while the user is leaving the screen, *then* the failure dialog offers `Stay here` and `Discard changes`, and choosing the latter discards the changes and completes the navigation.
**AC-CLI-30.** *Given* a dirty record and a browser page-close announcement, *when* the change set fits the fire-and-forget transport, *then* it is sent and the page closes; when it does not fit, the close is cancelled and the oversize notice is raised.

**Table screen**

**AC-CLI-31.** *Given* an editable table and a row being edited, *when* another row is activated, *then* the first row is saved before the second becomes editable, and a failure to save keeps the first row in edit mode.
**AC-CLI-32.** *Given* an editable table with `editable="top"`, *when* a record is created, *then* the new row appears first.
**AC-CLI-33.** *Given* a new row that was never touched, *when* the user leaves it, *then* it is abandoned and no record is created.
**AC-CLI-34.** *Given* a table declaring the multiple-record editing flag with three rows selected and one of them edited, *when* the change is confirmed, *then* one write addressed to the three record keys is issued.
**AC-CLI-35.** *Given* the same situation where one of the three rows has the changed field read-only, *then* the confirmation states `Among the 3 selected records, 2 are valid for this update.` and only two record keys are written.
**AC-CLI-36.** *Given* a table with a handle column, *when* a row is dragged to another position, *then* the ordering field of the affected rows is rewritten in one batch and no other row is touched.
**AC-CLI-37.** *Given* a grouped table whose count limit is `10000` and a query matching more, *then* the pager shows `10000+` and activating the total asks for the exact count.

**Related records**

**AC-CLI-38.** *Given* a list-of-records field and a line removed with the row control, *then* the command list contains a delete command for that line.
**AC-CLI-39.** *Given* an association field and a tag removed, *then* the command list contains a detach command, not a delete command.
**AC-CLI-40.** *Given* a list-of-records field with one new line and one edited line, *when* the parent is saved, *then* one write on the parent carries both a create command and an update command, and no separate write on the related entity is issued.
**AC-CLI-41.** *Given* a new, untouched, invalid line inside a list-of-records field, *when* the parent is saved, *then* the line is abandoned and the parent saves.
**AC-CLI-42.** *Given* a line opened in a dialog whose button saves the related record, *when* the parent is later saved, *then* the create command for that line has become a link command on the resulting key.

**Feedback**

**AC-CLI-43.** *Given* a request that fails because the connection was lost, *then* the sticky notice `Connection lost. Trying to reconnect...` appears once, regardless of how many requests failed at the same moment, and is replaced by `Connection restored. You are back online.` when the probe succeeds.
**AC-CLI-44.** *Given* a request that takes longer than `250` milliseconds, *then* the loading indicator appears showing the number of pending requests, and disappears as soon as none is pending.
**AC-CLI-45.** *Given* a button carrying the blocking flag, *when* it is activated, *then* the interface is blocked until the action completes, and the blocking is released even when the action fails.
**AC-CLI-46.** *Given* a server action that returns a celebration effect and a button that also declares one, *then* the button's declaration is played.
**AC-CLI-47.** *Given* a user whose celebration switch is off, *when* an effect is requested, *then* its message is raised as a notice and no banner is drawn.

**Shortcuts and palette**

**AC-CLI-48.** *Given* a record screen with pending changes, *when* the accelerator modifier together with `s` is pressed, *then* the record is saved.
**AC-CLI-49.** *Given* a dialog open over a screen, *when* a shortcut registered by the screen is pressed, *then* nothing happens, because only the topmost interactive area answers.
**AC-CLI-50.** *Given* the palette opened and `/` typed, *then* only applications and navigation entries are offered, with the prompt `Search for a menu...`.
**AC-CLI-51.** *Given* the palette opened with no text, *then* every command of the current screen and every visible accelerated control is offered, grouped by category in the order of section 14.4.

**Point of sale**

**AC-CLI-52.** *Given* the client is offline and an order is completed, *then* the order is written to the local database, no failure is shown, and the order is appended to the pending queue.
**AC-CLI-53.** *Given* pending queue entries and connectivity returning, *then* the entries are replayed in order and removed on success.
**AC-CLI-54.** *Given* paid orders written locally and not yet confirmed, *when* the page is about to be closed, *then* the closing is refused.
**AC-CLI-55.** *Given* the client is offline, *when* a record that is not in the local database is requested, *then* no request is issued and the caller receives nothing.

**AC-CLI-56.** *Given* a menu entry that the acting user may see but whose parent folder is restricted to a group the user does not hold, *when* the navigation tree is delivered, *then* the entry is absent from the payload, and every delivered entry names an application root that is present.

**AC-CLI-57.** *Given* a delivered navigation tree, *when* the payload is inspected, *then* it holds an entry under the key `root` whose identifier is false, whose name is the text `root` and whose children are the keys of the application roots.

**AC-CLI-58.** *Given* the same user and the same language, *when* the navigation tree is requested twice in diagnostic mode and once outside it, *then* two cache entries exist, one per diagnostic state, and each is reused on the second request.

---

## 19. Invariants a rebuild must preserve

1. The client never decides a permission; it renders the flags the server sent with the view and re-asks the server for anything it does not hold.
2. Only the **latest** action request survives: a new request cancels the answer of the previous one.
3. A screen with unsaved changes is always asked before it is left, and a refusal aborts the navigation without touching the stack.
4. Switching to a view that shows many records replaces the current action's segment of the trail; switching to a view that shows one record appends to it.
5. The address in the location bar mirrors the stack, and reloading the page restores the same trail.
6. The composed condition of a screen is the conjunction of the action's own condition, one disjunction per group of activated search items, and the side panel's contribution.
7. A change to a field declared as triggering a recomputation always produces exactly one round trip, and the answer replaces the client's values rather than being merged field by field by the client's own rules.
8. A save sends the pending changes and nothing else, and a failed save keeps the change set and keeps the record dirty.
9. A list of related records is sent as a command list, never as a whole value.
10. Every notice, dialog and celebration is a presentation effect; none of them changes what the server stored.

---

## 20. Reconciliation notes

Three topics could reasonably have been specified here and are specified elsewhere instead, and two further decisions are recorded so that a reader who expects to find a subject in this document knows where it went. Everything stated here was verified against the running system.

1. **What belongs here and what belongs with the views.** The grammar of a view description, the widget catalogue, the resolution passes and the actions are specified in [views and actions](views-and-actions.md), because they are what the server produces. This document specifies what the client does with them, and repeats none of the grammar.
2. **Where the asset bundles are specified.** A bundle's content is decided by which packages are installed and in what order, so it is specified in [the package system, section 22](package-system.md#22-client-asset-bundles). [Section 2.1](#21-the-startup-sequence) names the step at which the bundles are loaded and nothing more.
3. **Where printing belongs.** A report action is dispatched by the action manager of [section 4.2](#42-running-an-action) and rendered by [report rendering](../runtime/report-rendering.md); the rendering pipeline is not repeated here.
4. **The period-comparison control.** The platform examined for this specification does not carry the comparison mechanism that comparable systems offer. It is stated as an **industry-standard default** in [section 6.7](#67-period-comparison-industry-standard-completion), marked as such, and a rebuild that implements it that way is conformant.
5. **Acceptance criteria identifiers.** The scenarios of this document are numbered in one series with the prefix `AC-CLI`.
6. **The delivered navigation payload against the stored menu record.** These are two different things and are easy to confuse. The stored record is in [views and actions, section 24](views-and-actions.md#24-menus); the eleven keys a client actually receives, the synthetic root entry and the dropping of entries whose chain to an application root is broken are in [section 3.1](#31-the-delivered-payload).

---

## Related documents

- [Views and actions](views-and-actions.md) — the view descriptions, widgets, actions and menus this client renders.
- [Record operations and query notation](record-operations-and-query-notation.md) — the operations the screens invoke and the filter notation the search model produces.
- [The security model](security-model.md) — what the flags served with a view mean, and why none of them is enforcement.
- [Multi-company](multi-company.md) — the company switcher and the activation the client sends with every call.
- [The package system](package-system.md) — the asset bundles the client loads at start-up.
- [Architecture](architecture.md) — the request path behind every round trip described here.
- [Report rendering](../runtime/report-rendering.md) — what happens after a report action is dispatched.
- [Translation](../runtime/translation.md) — the translation payload fetched at start-up.
- [The notification bus](../runtime/notification-bus.md) — the channel the client listens on for pushed changes.
- [Desktop workflows](../interfaces/desktop-workflows.md) — the end-to-end procedures a person follows on these screens.
