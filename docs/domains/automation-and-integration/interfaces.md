# Interfaces

Everything through which a person or another program reaches this domain: menus, screens, named
operations, request paths, notification channels, the outside services the system contacts, the
local agent that drives connected peripherals, and the import and export doors.

Contents:

1. [Menus](#1-menus)
2. [Screens](#2-screens)
3. [Named operations](#3-named-operations)
4. [Request paths](#4-request-paths)
5. [Additions to the session description](#5-additions-to-the-session-description)
6. [Notification channels](#6-notification-channels)
7. [The connected-device agent](#7-the-connected-device-agent)
8. [Outside services this domain contacts](#8-outside-services-this-domain-contacts)
9. [Import and export](#9-import-and-export)
10. [Printable documents](#10-printable-documents)
11. [Widgets this domain contributes](#11-widgets-this-domain-contributes)

---

# 1. Menus

| Menu path | Identifier | Opens | Restricted to |
|---|---|---|---|
| Settings → Technical → Automation → Automation Rules | `base_automation.menu_base_automation_form` | the automation rule list | settings administration, through the access right |
| Settings → Technical → Import Module | `base_import_module.menu_view_base_module_import` | the package import screen | the technical-features group |
| Data Cleaning | `data_recycle.menu_data_cleaning_root` | the root of the housekeeping application, ordered at 250 | settings administration |
| Data Cleaning → Recycle Records | `data_recycle.menu_data_recycle_record` | the candidate list, ordered at 10 | settings administration |
| Data Cleaning → Configuration | `data_recycle.menu_data_cleaning_config` | a container, ordered at 100 | settings administration |
| Data Cleaning → Configuration → Rules | `data_recycle.menu_data_cleaning_config_rules` | a container, ordered at 1 | settings administration |
| Data Cleaning → Configuration → Rules → Recycle Records | `data_recycle.menu_data_cleaning_config_rules_recycle` | the rule list, ordered at 10 | settings administration |
| Settings → Technical → In Application Purchase | `iap.iap_root_menu` | a container, ordered at 5 | settings administration |
| Settings → Technical → In Application Purchase → In Application Purchase Accounts | `iap.iap_account_menu` | the metered-account list, ordered at 10 | settings administration |
| Settings → Technical → Privacy | `privacy_lookup.privacy_menu` | a container, ordered at 26 | settings administration |
| Settings → Technical → Privacy → Privacy Logs | `privacy_lookup.pricacy_log_menu` | the log list, ordered at 1 | settings administration |
| Settings → Translations → Transifex Code Translations | `transifex.menu_transifex_code_translations` | the cached-term list | settings administration |
| Settings → Technical → User Interface → Onboardings | `onboarding.menu_onboarding` | the panel list, ordered at 1 | settings administration |
| Settings → Technical → User Interface → Onboardings Steps | `onboarding.menu_onboarding_step` | the step list, ordered at 1 | settings administration |
| Settings → Technical → User Interface → Tours | `web_tour.menu_tour_action` | the tour list, ordered at 5 | every internal user may read; only a settings administrator may change |

**One menu this domain unrestricts.** The package that adds activation requests removes the group
restriction from the technical-settings menu so that an ordinary internal user can reach the package
list; see [`configuration.md`](configuration.md) §11.6.

---

# 2. Screens

## 2.1 Automation Rule

| Screen | Kind | What it shows |
|---|---|---|
| `base_automation.view_base_automation_kanban` | card | The default view. Cards carry the name, the watched record type and the trigger. |
| `base_automation.view_base_automation_tree` | list | The same columns in a table. |
| `base_automation.view_base_automation_form` | form | The whole rule. |
| `base_automation.view_base_automation_search` | search | One filter, "Include Archived", which widens the search to archived and unarchived rules. The list action switches that filter on by default. |

**The form.** A button box carrying *Logs*; a title holding the name; a first group holding the
watched record type, the trigger, and, for each trigger that needs it, the trigger value, the
reference, the delay with its mode and unit, and the working schedule; a labelled row reading
"Secret URL" holding the web address with a *Renew* button beside it, shown only for the webhook
trigger; a *Scheduled action* button shown only for a time trigger; the two filters; a field
labelled "When updating" holding the watched-field set, or, for the live-update trigger, the
live-update field set; a labelled row reading "Target Record" holding the record-finding expression,
shown only for the webhook trigger; and a notebook with two pages, "Actions To Do" holding the
action list and "Notes" holding the description. The button box, the *Renew* button and the
*Scheduled action* button are shown only to the technical-features group.

## 2.2 Metered services

| Screen | Kind | What it shows |
|---|---|---|
| `iap.iap_account_view_tree` | list | Name, service, companies, token — technical-features group only — balance, and, hidden by default, the alert threshold and the description. Duplication is disabled. |
| `iap.iap_account_view_form` | form | Two groups. "Account Information" holds the name, the service — read-only once the service has locked it, and offering neither opening nor creation — the description, the companies restricted to the reader's allowed companies, and the token for the technical-features group. The second group holds the balance with a *Buy Credit* button beside it, the alert threshold, and the recipients, which are hidden while the threshold is zero and required once it is above zero. Duplication is disabled. |

Opening either view refreshes the balance, as [`state-machines.md`](state-machines.md) §3.

## 2.3 Data recycling

| Screen | Kind | What it shows |
|---|---|---|
| `data_recycle.view_data_recycle_model_list` | list | The rules. |
| `data_recycle.view_data_merge_model_form` | form | One rule, with *Run Now* and a button opening its candidates. |
| `data_recycle.view_data_recycle_record_list` | list | The candidates, each row offering *Validate* and *Discard*. |
| `data_recycle.view_data_recycle_record_search` | search | A filter on the activation flag, which is what separates pending candidates from discarded ones. The candidate list groups by rule in its side panel. |

## 2.4 Privacy

| Screen | Kind | What it shows |
|---|---|---|
| `privacy_lookup.privacy_lookup_wizard_view_form` | form | The name and the address to look up, and the *Lookup* button. |
| `privacy_lookup.privacy_lookup_wizard_line_view_tree` | list | The found records, grouped by record type by default, with creation and inline creation disabled. |
| the log list and the log form | list, form | The masked name, the masked address, the handler, the date, the found-record description, the execution details and the free note. Creation is disabled on both. |

## 2.5 Package import and activation

| Screen | Kind | What it shows |
|---|---|---|
| `base_import_module.view_base_module_import` | form, opened as a dialogue | The archive field, the force-initialisation box, the demonstration-data box, the dependency description, and the buttons *Install*, *Cancel* and, once the import is done, *Close*. |
| `base_install_request.base_module_install_request_view_form` | form, dialogue | A group headed "Send to:" holding the reviewer list, a group headed "Why do you need this module?" holding the justification, and the buttons *Request Activation* and *Cancel*. The justification field carries the guidance text "e.g. I'd like to use the SMS Marketing module to organize the promotion of our internal events, and exhibitions. I need access for 3 people of my team." |
| `base_install_request.base_module_install_review_view_form` | form, dialogue titled "You are about to install an extra application" | The rendered list of applications that would be brought in, and the buttons *Install App* and *Cancel*. |

The package list, the package cards and the package form each gain a *Request Access* button, shown
only for an uninstalled package that is not a paid one and only to a reader who is **not** a settings
administrator. The card and form views also gain the installer used for catalogue packages.

## 2.6 Guidance

| Screen | Kind | What it shows |
|---|---|---|
| the onboarding panel list and form | list, form | The panel's title, its route segment, its ordering, its completion message, its steps and its per-company progress. |
| the onboarding step list and form | list, form | The step's title, description, button text, completion text, completion icon, image, image alternative text, opening operation and panels. |
| `web_tour.tour_list` | list | Ordering handle, name, starting address, the custom flag and a start control. Creation and editing from the list are disabled. |
| `web_tour.tour_form` | form | The name, the ordering, the starting address, the custom flag, the sharing address with a copy control, the steps — shown only for a custom tour — and the closing message. Creation from the form is disabled. |
| `web_tour.tour_search` | search | A search on the name. |

## 2.7 Other screens

| Screen | Kind | What it shows |
|---|---|---|
| the geocoding provider form | form | The provider name and its technical name. Creation and deletion are disabled. |
| the Contact form, page "Partner Assignment" | added page | A group headed "Geolocation" showing the latitude, the longitude, the date the coordinates were obtained, and one of two buttons: "Compute based on address" while both coordinates are zero, and "Refresh" otherwise. |
| the record-type form and the field form | added field | The serialisation field, offered only among fields of the serialised kind on the same record type, and read-only for a field the platform itself defines. |
| the server action form | added button | A button that jumps to the owning Automation Rule. |
| the cached-term list | list | The source term, the translation, the package, the language and the platform link. |

---

# 3. Named operations

Operations a person or a program can invoke by name. Every one of them is described in full in
[`workflows.md`](workflows.md) or [`entities.md`](entities.md); this table is the index.

## 3.1 Automation

| Operation | On | Effect |
|---|---|---|
| Open the scheduled action | Automation Rule | Opens the time-based scheduler's own form; refuses with [`business-rules.md#aut-011`](business-rules.md#aut-011) when it is missing. |
| Renew the webhook identifier | Automation Rule | Writes a fresh identifier on every selected rule. Its help text reads "If renewed, update the secret URL in the third-party app that calls this webhook." |
| View the webhook logs | Automation Rule | Opens the platform log list filtered to the lines whose origin names this rule. |
| Execute the webhook | Automation Rule | Runs the rule for one payload. |
| Open the automation | Server Action, Scheduled Action | Jumps to the owning rule. |

## 3.2 Data import

| Operation | On | Effect |
|---|---|---|
| Get the field tree | Data Import Session | Returns the importable-field tree of [`calculations.md`](calculations.md) §5.1. |
| Parse the preview | Data Import Session | Reads the file and proposes a mapping. |
| Execute the import | Data Import Session | Loads the rows, optionally as a trial. |
| Get the import templates | every record type | Returns the label and the path of each example file a record type offers, so that the import screen can offer a starting point. |

## 3.3 Packages

| Operation | On | Effect |
|---|---|---|
| Import the module | Module Import Wizard | Installs the archive. |
| Get the names of dependencies to install | Module Import Wizard | Lists what installing the archive would bring in. |
| Open the module | Module Import Wizard | Jumps to the imported package. |
| Install this application immediately | Module | Downloads a catalogue package and installs it. |
| More information | Module | Returns the catalogue's description of a package. |
| Open the activation request | Module | Opens the request dialogue for that package. |
| Send the request | Module Activation Request | Sends one message per reviewer. |
| Install the application | Module Activation Review | Installs the package and sends the browser to the application home. |

## 3.4 Metered services

| Operation | On | Effect |
|---|---|---|
| Get | In-Application Purchase Account | Finds or creates the account for a technical name. |
| Get the account identifier | In-Application Purchase Account | The same, returning the identifier only. |
| Get the balance | In-Application Purchase Account | Asks the service for the balance of one technical name. |
| Get the purchase address | In-Application Purchase Account | Builds the address of [`calculations.md`](calculations.md) §4.2. |
| Get the configuration address | In-Application Purchase Account | Returns the address of the account list, or of one account, for the technical-features group only. |
| Buy credit | In-Application Purchase Account | Opens the purchase address. |
| Request enrichment | Lead Enrichment Interface | Enriches a batch of addresses. |
| Search by name | Contact | Suggests companies by name. |
| Search by tax registration number | Contact | Suggests companies by number, with the cross-border fallback. |
| Enrich by company registration number | Contact | Enriches from a directory number. |
| Enrich by regional tax number | Contact | Enriches from a regional number. |
| Enrich by internet domain | Contact | Enriches from a domain. |
| Post the enrichment message | Contact | Posts the enrichment result as an internal note. |
| Enrich automatically | Company | Runs the one-time enrichment. |

## 3.5 Housekeeping

| Operation | On | Effect |
|---|---|---|
| Run now | Recycling Model | Searches for candidates and, for a manual rule, opens them. |
| Open the records | Recycling Model | Opens the candidate list filtered to this rule. |
| Validate | Recycling Record | Archives or deletes the original and removes the candidate. |
| Discard | Recycling Record | Clears the candidate's activation flag. |
| Look up | Privacy Lookup Wizard | Runs the whole-database query and opens the lines. |
| Open the lines | Privacy Lookup Wizard | Opens the line list for this session. |
| Delete | Privacy Lookup Wizard Line | Deletes the found record. |
| Archive the selection | Privacy Lookup Wizard Line | Archives every selected line's record. |
| Delete the selection | Privacy Lookup Wizard Line | Deletes every selected line's record. |
| Open the record | Privacy Lookup Wizard Line | Opens the found record. |
| Privacy lookup | Contact | Opens the wizard for that Contact. |

## 3.6 Addresses, storage and guidance

| Operation | On | Effect |
|---|---|---|
| Find coordinates | Geocoder | Resolves one query string. |
| Build the query address | Geocoder | Builds the query string of [`calculations.md`](calculations.md) §13.1. |
| Resolve the coordinates | Contact | Resolves and writes the coordinates. |
| Bring back to local storage | Attachment | Downloads the blob and stores it locally. |
| Open the consent address | Mail Server, Incoming Mail Server | Starts the delegated authorisation. |
| Consume the tour | Tour | Marks the tour consumed by the reading user and returns the next one. |
| Get the current tour | Tour | Returns the tour to offer, or nothing. |
| Get the tour by name | Tour | Returns one tour's description. |
| Export the description | Tour | Produces the exported file and downloads it. |
| Switch the tour offer | User | Turns the offer on or off for the reading user. |
| Action-open a step | Onboarding Step | Runs the step's opening operation. |
| Mark a step as just done | Onboarding Step | Records completion for the reading company. |
| Mark a step as not done | Onboarding Step | Clears it. |
| Close the panel | Onboarding | Marks the panel closed for the reading company. |
| Reload the term cache | Code Translation | Empties and reloads. |
| Open the code translations | Code Translation | Loads if needed and opens the list. |

---

# 4. Request paths

Every path this domain adds. The *reached by* column says what credential the path requires: *no
credential* means the path answers before any user is identified, *a session* means an identified
user, *a bearer credential* means an authorisation key sent with the request, and *anyone* means the
path answers both identified and anonymous callers.

## 4.1 Automation

| Path | Methods | Reached by | Answer |
|---|---|---|---|
| `/web/hook/<string:rule_uuid>` | fetch and submit | anyone | A structured body carrying `ok` under `status` with the status 200; `error` with 404 when no rule carries the identifier; `error` with 500 when anything failed. The path is exempt from the cross-site form protection and does not open a session, because the caller is a program. |

The payload is the submitted structured body when the request carries one, and otherwise the query
arguments of the request as a mapping.

## 4.2 Data and package import

| Path | Methods | Reached by | Answer |
|---|---|---|---|
| `/base_import/set_file` | submit | a session | Writes the uploaded bytes, the file name and the reported media type onto the named session and answers with the write result. |
| `/base_import_module/login_upload` | submit | no credential; the request carries a login and a password | Authenticates, checks that the reader is an administrator ([`business-rules.md#aut-049`](business-rules.md#aut-049)), installs the archive and answers with the plain text of the result, or with the status 500 and the failure text. |

## 4.3 Reflection documentation

| Path | Methods | Reached by | Answer |
|---|---|---|---|
| `/doc`, `/doc/<model_name>`, `/doc/index.html` | fetch | a session | The documentation page, served with the framing prohibition set to deny. Restricted by [`business-rules.md#aut-133`](business-rules.md#aut-133). |
| `/doc/index.json` | fetch and submit | a session | The index: the installed packages, and for each record type its transport name, its display name, its fields with their labels and its public operations. Cached as described below. |
| `/doc-bearer/index.json` | fetch and submit | a bearer credential | The same index, without the group check. |
| `/doc/<model_name>.json` | fetch and submit | a session, read-only | One record type in full: the described record type, its enriched field descriptions and its public operations with their parsed signatures. |
| `/doc-bearer/<model_name>.json` | fetch and submit | a bearer credential, read-only | The same. |

**Caching of the index.** When the request asks for no cache, the index is built and answered with
the caching directive `no-store`, a disposition naming the file `odoo-doc-index.json` and the
reading language. Otherwise a key is computed by [`calculations.md`](calculations.md) §7.4; a client
that already holds that key is answered with the not-modified status; otherwise the index is looked
up among the attachments by its computed name, built and stored when absent, and streamed. The
per-record-type answer is cached the same way but always carries the directive `no-cache, private`.

## 4.4 Delegated authorisation

| Path | Methods | Reached by | Answer |
|---|---|---|---|
| `/google_account/authentication` | fetch | anyone | The generic return path of the first account behaviour. The state carries the service under `s` and the address to return to under `f`. A missing service, or a code without a return address, answers with the bad-request status. With a code, the tokens are exchanged and written onto the reading user's settings, and the browser is sent to the return address. With a failure, the browser is sent to the return address with the failure appended under `error`; with neither, the fixed value `Unknown_error` is appended. |
| `/microsoft_account/authentication` | fetch | anyone | The same for the second account behaviour, writing the tokens onto the reading user. |
| `/google_gmail/confirm` | fetch | a session | The first mail provider's callback: parses the state, finds the record, exchanges the code, verifies the address and writes the record. Refusals are [`business-rules.md#aut-123`](business-rules.md#aut-123) to [`business-rules.md#aut-126`](business-rules.md#aut-126). |
| `/google_gmail/iap_confirm` | fetch | a session | The first mail provider's relay callback: receives the record type, the record identifier, the protection token, the access token, the refresh token and the expiry, performs the same record and address checks, and writes the record. |
| `/microsoft_outlook/confirm` | fetch | a session | The second mail provider's callback. |
| `/microsoft_outlook/iap_confirm` | fetch | a session | The second mail provider's relay callback. |

**Where the browser goes afterwards.** To the record's own form when the reader is a settings
administrator and the record is not that reader's personal outgoing server; to the reader's own
preferences otherwise.

## 4.5 Address completion

| Path | Methods | Reached by | Answer |
|---|---|---|---|
| `/autocomplete/address` | submit | anyone | A mapping carrying the suggestions under `results` and the session token under `session_id`. Each suggestion carries the formatted address under `formatted_address` and the opaque place identifier under `google_place_id`. An empty list is answered when the key cannot be read, when the typed text is no longer than the configured minimum, or when the provider timed out or answered unreadably. |
| `/autocomplete/address_full` | submit | anyone, but internal users only in effect | The standard address fields, or nothing at all when the provider timed out or answered unreadably. A non-internal caller is refused by [`business-rules.md#aut-084`](business-rules.md#aut-084). |

## 4.6 Attachment upload

The messaging domain's attachment upload path is extended. When the request asks for an outside
upload:

1. With no provider configured the answer carries the text of
   [`business-rules.md#aut-107`](business-rules.md#aut-107) under `error` and nothing is created.
2. Otherwise the ordinary upload runs; an answer already carrying a failure is passed through
   unchanged.
3. On success the answer gains, under `upload_info`, the signed address, the request method, the
   expected success status and the request headers, so that the client puts the bytes straight to
   the provider.

## 4.7 Remote calls

| Path | Methods | Reached by | Answer |
|---|---|---|---|
| `/xmlrpc/<service>` | submit | no credential | A structured-markup call whose fault codes are texts. Faults are `warning -- Warning`, `warning -- MissingError`, `warning -- AccessError`, `AccessDenied` and `warning -- UserError`, each followed by two line breaks and the failure text; anything else answers with the failure's own text as the code and the trace as the value. |
| `/xmlrpc/2/<service>` | submit | no credential | The same call with whole-number fault codes: 1 for an application failure and for a client failure, 2 for a warning and for a user failure, 3 for a refused authentication, 4 for a refused access. |
| `/jsonrpc` | submit | no credential | A structured call taking the service, the operation and the arguments. |
| `/json/2/<model>/<method>` | submit | a bearer credential | The typed call. Takes the identifiers under `ids` and the reading context under `context`, and passes every other named argument to the operation. Refusals are [`business-rules.md#aut-134`](business-rules.md#aut-134) to [`business-rules.md#aut-137`](business-rules.md#aut-137). A record set in the answer is reduced to its identifiers. Whether the call may run against a read-only connection is decided per operation, by asking the operation itself. |
| `/json/2`, `/json/2/<path>` | fetch, submit, replace, delete, patch | anyone | The catch-all of [`business-rules.md#aut-138`](business-rules.md#aut-138). |
| `/web/version`, `/json/version` | fetch | no credential | A structured body carrying the release identification under `version_info` and `version`. Read-only. |

**Marshalling rules of the structured-markup call.** Frozen mappings are sent as ordinary mappings;
byte strings are sent as text in the standard sixty-four character encoding rather than as a binary
value; a date and time is sent as text in the storage form; a date is sent as text in the
standard year-month-day form; a lazily computed value is resolved and sent as its own kind; a rich
text value is sent as text; a command value is sent as a whole number; a defaulting mapping is sent
as a mapping. Every text has the control characters below the value 32 removed, except tabulation,
line feed and carriage return, because the markup forbids them and clients break on them.

## 4.8 The connected-device agent

See §7.

---

# 5. Additions to the session description

The session description the client receives when it starts gains three entries from this domain.

| Entry | Added by | Meaning |
|---|---|---|
| the minimum outside-upload size in bytes | the outside-store package | At or above this size the client uploads straight to the outside store. |
| the list of record types that never store attachments outside | the outside-store package | [`business-rules.md#aut-142`](business-rules.md#aut-142). |
| `tour_enabled` | the tour package | Whether the reading user is offered tours. |
| `current_tour` | the tour package | The description of the tour to offer, or nothing. |

The package-import package also extends the translation source the client is given, so that the
terms of an imported package are available to the interface.

---

# 6. Notification channels

| Channel | Payload | Raised by |
|---|---|---|
| `iap_notification` | a message and a kind, with an optional title; or a title, the kind `no_credit` and a purchase address | metered calls |
| `simple_notification` | the kind `danger`, the title "Warning" and the message "No match found for %(partner_names)s address(es)." | address resolution that found nothing |

Both are pushed to the calling user's own channel, not broadcast.

---

# 7. The connected-device agent

A separate program runs beside the peripherals and answers the paths below. It is not part of the
application server; the client and, in some flows, the server call it directly. Every path allows
any origin, because the client that calls it is served from a different address.

## 7.1 The action and event contract

| Path | Kind | Purpose |
|---|---|---|
| `/iot_drivers/action` | structured call | Ask one device to do something. Takes the session identifier, the device identifier and the action data. |
| `/iot_drivers/event` | structured call | Long-poll for events. Takes a listener carrying the session identifier, the devices to listen to and the marker of the last event seen. |
| `/iot_drivers/download_logs` | fetch | Download the agent's log file. |

**The nine device kinds** a device identifier may name instead of a device: `display`, `printer`,
`scanner`, `keyboard`, `camera`, `device`, `payment`, `scale` and `fiscal_data_module`. Naming a
kind picks the first device of that kind.

**The self-addressed action.** When the device identifier is the agent's own identifier, the action
`restart_odoo` publishes a success event, waits two seconds so that the waiting client receives it,
and restarts the agent; any other action answers with the value true, which is how a client tests
that the long-poll protocol works at all.

**Timings.** Events older than five seconds are discarded. A waiting call returns as soon as a newer
event for a listened device arrives, and otherwise after fifty seconds with nothing. A registered
listening session that has not been touched for seventy seconds is dropped. A client whose call
fails retries with a delay that starts at one and a half seconds, grows, and is capped at fifteen
seconds.

## 7.2 The status and maintenance paths

| Path | Kind | Purpose |
|---|---|---|
| `/` | fetch | The agent's home page. |
| `/status` | fetch | A human-readable status page. |
| `/logs` | fetch | A human-readable log page. |
| `/hw_proxy/hello` | fetch | A liveness probe. |
| `/hw_proxy/status_json` | structured call | The status of every driver as a structured body. |
| `/hw_proxy/scale_read` | structured call | The current reading of the first weighing device. |
| `/iot_drivers/ping` | fetch | A reachability probe. |
| `/iot_drivers/data` | fetch | The agent's own description and the devices it has found. |
| `/iot_drivers/version_info` | fetch | The agent's build identification. Available only on the appliance build. |
| `/iot_drivers/restart_odoo_service` | fetch | Restart the agent. |
| `/iot_drivers/iot_logs` | fetch | The log file. |
| `/iot_drivers/log_levels` | fetch | The current log levels. |
| `/iot_drivers/log_levels_update` | structured call | Change them. |
| `/iot_drivers/load_iot_handlers` | fetch | Reload the drivers. |
| `/iot_drivers/clear_credential` | fetch | Forget the stored credential. |
| `/iot_drivers/save_credential` | structured call | Store a credential. |
| `/iot_drivers/connect_to_server` | structured call | Point the agent at an application server. |
| `/iot_drivers/server_clear` | fetch | Forget that server. |
| `/iot_drivers/wifi` | fetch | The current wireless configuration. Appliance build only. |
| `/iot_drivers/update_wifi` | structured call | Change it. Appliance build only. |
| `/iot_drivers/wifi_clear` | fetch | Forget it. Appliance build only. |
| `/iot_drivers/six_payment_terminal_add` | structured call | Register a payment terminal. |
| `/iot_drivers/six_payment_terminal_clear` | fetch | Forget it. |
| `/iot_drivers/is_ngrok_enabled` | fetch | Whether remote access is on. Appliance build only. |
| `/iot_drivers/enable_ngrok` | structured call | Turn remote access on. Appliance build only. |
| `/iot_drivers/disable_ngrok` | structured call | Turn it off. Appliance build only. |
| `/iot_drivers/update_git_tree` | structured call | Update the agent's driver tree. Appliance build only. |

## 7.3 What the agent answers

An action answers with the value true when it was dispatched and false when the device is unknown,
in which case a warning naming the identifier is logged. The handler's own result is published as an
event carrying either a success or a failure, the original arguments and the session identifier;
printers and payment terminals publish their own events instead, because their results arrive
asynchronously. A repeated action identifier is ignored, so that a retried request does not act
twice.

---

# 8. Outside services this domain contacts

Every address below is reproduced because it is part of the integration contract: an installation
that changes it talks to a different service.

| Purpose | Address | Timeout | Credential sent |
|---|---|---|---|
| Metered service calls | the system parameter `iap.endpoint`, default `https://iap.odoo.com`, followed by the service path | 15 seconds | the account token and the database identifier |
| Balance and alert reading | the same endpoint, path `/iap/1/get-accounts-information` | 15 seconds | one token per account and the database identifier |
| Alert configuration push | the same endpoint, path `/iap/1/update-warning-email-alerts` | 15 seconds | the account token |
| Purchase page | the same endpoint, path `/iap/1/credit` | — | the hashed token in the query |
| Company directory | the system parameter `iap.partner_autocomplete.endpoint`, default `https://partner-autocomplete.odoo.com`, path `/api/dnb/1/<action>` | 15 seconds, 5 for the automatic company enrichment | the account token, the database identifier, the release identification, the reading language, the company's country code and postal code |
| Lead enrichment | the system parameter `enrich.endpoint`, default `https://iap-services.odoo.com`, path `/iap/clearbit/1/lead_enrichment_email` | 300 seconds | the account token and the database identifier |
| Cross-border tax number verification | the platform's own verification helper | 15 seconds | none |
| Address resolution, first provider | `https://nominatim.openstreetmap.org/search` | none set | none; a user-agent header identifying the caller is sent |
| Reverse resolution, first provider | `https://nominatim.openstreetmap.org/reverse` | 10 seconds | the same header |
| Address resolution, second provider | `https://maps.googleapis.com/maps/api/geocode/json` | none set | the service key in the query |
| Address suggestions | `https://maps.googleapis.com/maps/api/place/autocomplete/json` | 2.5 seconds | the service key, and a session token when the client supplied one |
| Address details | `https://maps.googleapis.com/maps/api/place/details/json` | 2.5 seconds | the same |
| First mail provider, token exchange | `https://oauth2.googleapis.com/token` | 5 seconds | the application identifier and secret |
| First mail provider, address verification | `https://www.googleapis.com/oauth2/v2/userinfo` | 5 seconds | the access token |
| First mail provider, relay start | the system parameter `mail.server.gmail.iap.endpoint`, default `https://gmail.api.odoo.com`, path `/api/mail_oauth/1/gmail` | 5 seconds | the database identifier and the callback address |
| First mail provider, relay refresh | the same endpoint, path `/api/mail_oauth/1/gmail_access_token` | 5 seconds | the refresh token and the database identifier |
| Second mail provider, consent and token | the system parameters `microsoft_account.auth_endpoint` and `microsoft_account.token_endpoint`, defaults `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` and `https://login.microsoftonline.com/common/oauth2/v2.0/token` | 5 seconds | the application identifier and secret |
| Second mail provider, relay | the system parameter `mail.server.outlook.iap.endpoint`, default `https://outlook.api.odoo.com` | 5 seconds | the database identifier and the callback address |
| Second account behaviour, service calls | `https://graph.microsoft.com` | the behaviour's own timeout | the access token |
| Outside store, first provider | `https://<account>.blob.core.windows.net/<container>/<blob>` | 5 seconds for the probe, 10 for bringing a blob back | a signature in the query |
| Outside store, first provider, delegation key | `https://login.microsoftonline.com/<tenant>/oauth2/v2.0/token` then the account's own service path | 5 seconds | the client identifier and secret |
| Outside store, second provider | `https://storage.googleapis.com/<bucket>/<blob>` | 5 seconds for the probe, 10 for bringing a blob back | a signature in the query |
| Package catalogue | `https://apps.odoo.com`, paths for the listing, the categories and `/loempia/download/data_app/<name>/<series>` | 5 seconds for a download | none |
| Translation platform | the system parameter `transifex.project_url`, default `https://app.transifex.com/odoo` | — | none; the address is only ever shown to a user, never called |

**Two of these are addresses the system never calls**: the purchase page and the translation
platform link are built and handed to the browser.

---

# 9. Import and export

## 9.1 What comes in

| Door | Formats | Where specified |
|---|---|---|
| Data import | delimited text, the two office workbook formats and the open document spreadsheet format | [`workflows.md`](workflows.md) §9 |
| Package import | one archive holding one or more packages | [`workflows.md`](workflows.md) §11 |
| Package catalogue install | the same archive, fetched rather than uploaded | [`workflows.md`](workflows.md) §12 |
| Incoming webhook | one structured body, or the query arguments | [`workflows.md`](workflows.md) §7 |
| Remote calls | the three call paths of §4.7 | — |
| Outside-store upload | the client puts the bytes straight to the provider | [`workflows.md`](workflows.md) §21.2 |

**Example files.** Every record type may declare example files for the import screen, each with a
label and a path. The declaration is empty by default and is filled by whichever domain owns the
record type.

## 9.2 What goes out

| Door | Shape | Where specified |
|---|---|---|
| Outgoing webhook | a structured body with three fixed keys and the chosen fields | [`calculations.md`](calculations.md) §2.3 |
| Tour export | one client script file, downloaded as an attachment | [`calculations.md`](calculations.md) §17.2 |
| Reflection documentation | the index and the per-record-type documents of §4.3 | — |
| Remote call answers | the marshalled results of §4.7 | — |
| Outside-store download | a redirection to a signed address | [`workflows.md`](workflows.md) §21.3 |
| Metered alert configuration | the push of [`calculations.md`](calculations.md) §4.3 | — |

## 9.3 What is deliberately not exported

The domain exports no report and no accounting file. The Privacy Log is readable in the interface
but has no export of its own; an installation that must hand a person a copy of what was found uses
the platform's own export of the log list.

---

# 10. Printable documents

**The domain defines no printable document.** Nothing it owns is printed. The nearest thing is the
tour export, which produces a file for a developer rather than a document for a reader, and the
reflection documentation, which is a page rather than a printable.

---

# 11. Widgets this domain contributes

| Widget | Where it is used | What it does |
|---|---|---|
| the company autocomplete widget | the name, the tax registration number and the company registration number of a Contact and of a Company | Calls the company directory as the user types and offers suggestions; choosing one enriches the record. |
| the address autocomplete widget | the street of a Contact, when the country catalogue enforces a city list | Calls the address suggestion path as the user types and fills the address fields from the chosen suggestion. |
| the copy control on an address | the tour's sharing address | Copies the address to the clipboard. |
| the tour start control | the tour list and form | Starts the tour in the browser. |
| the ordering handle | the tour steps, the tour list | Reorders rows by dragging. |
| the balance display with a purchase control | the metered account form | Shows the formatted balance and offers *Buy Credit*. |

Every widget is a client concern; the contract each of them relies on is a named operation of §3 or
a request path of §4, so a different client can reproduce the behaviour without reproducing the
widget.
