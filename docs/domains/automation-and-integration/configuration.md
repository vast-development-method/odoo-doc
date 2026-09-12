# Configuration

Everything an administrator sets, everything a package ships and everything the platform runs on a
schedule for this domain: settings, system parameters, sequences, shipped records, groups, access
rights, record rules, scheduled jobs, message templates, notification templates and activity types.

Contents:

1. [Settings shown in the general settings](#1-settings-shown-in-the-general-settings)
2. [System parameters](#2-system-parameters)
3. [Sequences](#3-sequences)
4. [Security groups](#4-security-groups)
5. [Access rights](#5-access-rights)
6. [Record rules](#6-record-rules)
7. [Field-level visibility](#7-field-level-visibility)
8. [Scheduled jobs](#8-scheduled-jobs)
9. [Message and notification templates](#9-message-and-notification-templates)
10. [Activity types](#10-activity-types)
11. [Shipped records](#11-shipped-records)
12. [Constants that are not settings](#12-constants-that-are-not-settings)
13. [Behaviour when the database is a neutralised copy](#13-behaviour-when-the-database-is-a-neutralised-copy)
14. [What a fresh installation looks like](#14-what-a-fresh-installation-looks-like)

---

# 1. Settings shown in the general settings

Every setting below is a field of the Configuration Settings screen. A setting whose *stored as*
column names a system parameter writes that parameter when the screen is saved and reads it when the
screen is opened; a setting with no parameter is computed or is used only during the save.

## 1.1 Attachments stored outside

| Field identifier | Label | Kind | Stored as | Default | Meaning |
|---|---|---|---|---|---|
| `cloud_storage_provider` | Cloud Storage Provider for new attachments | selection | `cloud_storage_provider` | empty | Which outside store new large attachments go to. The base package offers no value; the first provider package adds `azure` labelled "Azure Cloud Storage" and the second adds `google` labelled "Google Cloud Storage". Empty means the feature is off. |
| `cloud_storage_min_file_size_mb` | Minimum File Size (Megabyte) | decimal | not stored | derived | What an administrator types. On save it is multiplied by 1 000 000 and truncated into the byte field; on open it is the byte field divided by 1 000 000. |
| `cloud_storage_min_file_size` | Minimum File Size (bytes) | whole number | `cloud_storage_min_file_size` | 20 000 000 | The threshold at or above which the client uploads straight to the outside store. Its help text reads "webclient can upload files larger than the minimum file size (in bytes) as url attachments to the server and then upload the file to the cloud storage." |
| `cloud_storage_azure_account_name` | Azure Account Name | single line text | `cloud_storage_azure_account_name` | empty | The account that owns the container. |
| `cloud_storage_azure_container_name` | Azure Container Name | single line text | `cloud_storage_azure_container_name` | empty | The container blobs are written to. |
| `cloud_storage_azure_tenant_id` | Azure Tenant identifier | single line text | `cloud_storage_azure_tenant_id` | empty | The directory the application registration belongs to. |
| `cloud_storage_azure_client_id` | Azure Client identifier | single line text | `cloud_storage_azure_client_id` | empty | The application registration. |
| `cloud_storage_azure_client_secret` | Azure Client Secret | single line text | `cloud_storage_azure_client_secret` | empty | Its secret. |
| `cloud_storage_azure_invalidate_user_delegation_key` | Invalidate Cached Azure User Delegation Key | boolean | not stored | false | Setting it and saving raises the key sequence by one, which discards the cached signing key. |
| `cloud_storage_google_bucket_name` | Google Bucket Name | single line text | `cloud_storage_google_bucket_name` | empty | The bucket blobs are written to. |
| `cloud_storage_google_service_account_key` | Google Service Account Key | binary, not stored | — | empty | The uploaded key file. On change it is decoded into the next field; on open it is the next field encoded back. |
| `cloud_storage_google_account_info` | Google Service Account Info | single line text | `cloud_storage_google_account_info` | empty | The decoded key description, from which the signing key is built. |

**The four things a save does**, in order:

1. When a provider is already in force and the chosen provider differs, the outgoing provider is
   asked whether it may be switched off
   ([`business-rules.md#aut-100`](business-rules.md#aut-100),
   [`business-rules.md#aut-101`](business-rules.md#aut-101)).
2. The byte threshold is computed from the megabyte figure.
3. The parameters are written.
4. When a provider is chosen but its configuration is incomplete the save is refused
   ([`business-rules.md#aut-102`](business-rules.md#aut-102)); when the configuration is complete
   **and has changed**, the provider is verified by the probe of
   [`workflows.md`](workflows.md) §21.1 step 5.

**Completeness.** The first provider's configuration is complete when the account name, the
container name, the tenant identifier, the client identifier and the client secret are all filled.
The second provider's is complete when the bucket name and the account description are both filled.

## 1.2 Address resolution

| Field identifier | Label | Kind | Stored as | Default | Meaning |
|---|---|---|---|---|---|
| `geoloc_provider_id` | "API" | link to Geocoding Provider | `base_geolocalize.geo_provider` | the provider the Geocoder resolves at the time the screen is opened | Which provider resolves addresses. |
| `geoloc_provider_techname` | — | single line text, read-only | — | — | The chosen provider's technical name, used by the screen to show the key field only for the provider that needs it. |
| `geoloc_provider_googlemap_key` | "Google Map API Key" | single line text | `base_geolocalize.google_map_api_key` | empty | The service key the second provider needs. Its help text reads "Visit https://developers.google.com/maps/documentation/geocoding/get-api-key for more information." |

## 1.3 Address completion while typing

| Field identifier | Label | Kind | Stored as | Default | Meaning |
|---|---|---|---|---|---|
| `google_places_api_key` | "Google Places API Key" | single line text | `google_address_autocomplete.google_places_api_key` | empty | The service key for the suggestion and detail calls. Without it both calls answer with an empty result list. |

## 1.4 Company directory

| Field identifier | Label | Kind | Stored as | Meaning |
|---|---|---|---|---|
| `partner_autocomplete_insufficient_credit` | Insufficient credit | boolean, computed, not stored | — | True when the balance of the metered account for the company directory is at or below zero. The screen uses it to show a top-up prompt. |

The screen also carries an operation *redirect to buy autocomplete credit*, which opens the purchase
address of [`calculations.md`](calculations.md) §4.2 for the service technical name
`partner_autocomplete` in a new browser context.

## 1.5 Delegated mail authorisation

| Field identifier | Label | Kind | Stored as | Meaning |
|---|---|---|---|---|
| `google_gmail_client_identifier` | Gmail Client Id | single line text | `google_gmail_client_id` | The application registration used by the first provider. |
| `google_gmail_client_secret` | Gmail Client Secret | single line text | `google_gmail_client_secret` | Its secret. |
| `microsoft_outlook_client_identifier` | Outlook Client Id | single line text | `microsoft_outlook_client_id` | The application registration used by the second provider. |
| `microsoft_outlook_client_secret` | Outlook Client Secret | single line text | `microsoft_outlook_client_secret` | Its secret. |

When a pair is empty the corresponding provider falls back to the relay path of
[`workflows.md`](workflows.md) §22.2 step 3.

## 1.6 Metered services

The general settings carry a block that opens the metered-account list through the operation
labelled "View My Services". It is not a stored setting; it is a shortcut to the list described in
[`interfaces.md`](interfaces.md) §2.

---

# 2. System parameters

Parameters are read with elevated rights and, unless stated otherwise, have no shipped row: reading
one that has never been written yields the default in the table.

| Key | Default when absent | Written by | Read by |
|---|---|---|---|
| `base_geolocalize.geo_provider` | the first Geocoding Provider found | the settings screen | the Geocoder when choosing a provider |
| `base_geolocalize.google_map_api_key` | empty | the settings screen | the second geocoding provider |
| `google_address_autocomplete.google_places_api_key` | empty | the settings screen | both address-completion routes |
| `google_address_autocomplete.minimal_partial_address_size` | `5` | by hand | the suggestion route, which answers with an empty result list for a typed text no longer than this |
| `cloud_storage_provider` | empty | the settings screen | every outside-store operation |
| `cloud_storage_min_file_size` | `20000000` | the settings screen | the session description |
| `cloud_storage_azure_account_name` | empty | the settings screen | the first provider |
| `cloud_storage_azure_container_name` | empty | the settings screen | the first provider |
| `cloud_storage_azure_tenant_id` | empty | the settings screen | the first provider |
| `cloud_storage_azure_client_id` | empty | the settings screen | the first provider |
| `cloud_storage_azure_client_secret` | empty | the settings screen | the first provider |
| `cloud_storage_azure_user_delegation_key_sequence` | `0` | the settings screen, raised by one when the invalidation box is ticked | the signing-key cache |
| `cloud_storage_google_bucket_name` | empty | the settings screen | the second provider |
| `cloud_storage_google_account_info` | empty | the settings screen | the second provider |
| `google_gmail_client_id` | empty | the settings screen | the first mail provider |
| `google_gmail_client_secret` | empty | the settings screen | the first mail provider |
| `mail.server.gmail.iap.endpoint` | `https://gmail.api.odoo.com` | by hand | the first mail provider's relay path |
| `microsoft_outlook_client_id` | empty | the settings screen | the second mail provider |
| `microsoft_outlook_client_secret` | empty | the settings screen | the second mail provider |
| `mail.server.outlook.iap.endpoint` | `https://outlook.api.odoo.com` | by hand | the second mail provider's relay path |
| `microsoft_account.auth_endpoint` | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` | by hand | the second account behaviour's consent address |
| `microsoft_account.token_endpoint` | `https://login.microsoftonline.com/common/oauth2/v2.0/token` | by hand | the second account behaviour's token exchange |
| `microsoft_redirect_uri` | `urn:ietf:wg:oauth:2.0:oob` | **shipped** by the second account package | the second account behaviour's return address when none is passed |
| `mail.server.personal.limit.minutes_outlook` | `10` when unset or zero | by hand | how many messages a personal outgoing server of the second provider may send in one minute |
| `iap.endpoint` | `https://iap.odoo.com` | by hand | every metered call and the purchase address |
| `iap.partner_autocomplete.endpoint` | `https://partner-autocomplete.odoo.com` | by hand | the company directory calls |
| `enrich.endpoint` | `https://iap-services.odoo.com` | by hand | the lead enrichment call |
| `transifex.project_url` | `https://app.transifex.com/odoo` | **shipped** by the translation package | the platform link builder |
| `database.uuid` | set at creation by the platform | the platform | every metered call, the purchase address and both relay paths |
| `database.is_neutralized` | absent | the platform | account creation, which disables the token of a new account on a neutralised copy |
| `base.enable_programmatic_api_keys` | absent | by hand | the remote-call package, which uses it to decide whether an authorisation key may be created without a session |

**Two addresses are reproduced, not authored.** The relay, catalogue, directory and translation
addresses above are part of the integration contract: an installation that changes them talks to a
different service. They are written in code font for that reason.

---

# 3. Sequences

**The domain defines no numbering sequence.** Nothing it owns carries a reference number. Three
identifier-like values are generated instead, and none of them is a sequence:

| Value | How it is produced | Where specified |
|---|---|---|
| The webhook identifier of an Automation Rule | a universally unique identifier, regenerated on demand | [`calculations.md`](calculations.md) §2.1 |
| The token of a metered account | a universally unique identifier with its hyphens removed | [`calculations.md`](calculations.md) §7.1 |
| The middle part of an outside-store blob name | a fresh universally unique identifier per attachment | [`calculations.md`](calculations.md) §14.1 |

---

# 4. Security groups

| Group | Identifier | Members at installation | Implied by | What it opens |
|---|---|---|---|---|
| Technical Documentation | `api_doc.group_allow_doc` | the root user | the settings-administration group | The reflection documentation page and the two listing routes reached with a session ([`business-rules.md#aut-133`](business-rules.md#aut-133)) |

Every other capability in this domain is gated by two groups the platform already defines: the
internal-user group and the settings-administration group. A third, the technical-features group,
changes what is *shown* rather than what is allowed:

| Where the technical-features group changes something | Effect |
|---|---|
| The package-import menu entry | Hidden unless the reader is in the group |
| The importable-field tree | A one-to-many field gains a child for the database identifier |
| The import preview answer | Carries a flag telling the client the reader is in the group, which opens the advanced mapping editor |
| The privacy record description | Each record type is written as its display name, a hyphen and its transport name instead of its display name alone |
| The metered-account shortcut address | Produced only for a reader in the group |

---

# 5. Access rights

One row per declaration. A right that is not granted is refused.

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Automation Rule | settings administration | yes | yes | yes | yes |
| Data Import Session | internal user | yes | yes | yes | **no** |
| Data Import Column Mapping | internal user | yes | yes | yes | yes |
| Module Import Wizard | settings administration | yes | yes | yes | **no** |
| Module Activation Request | internal user | yes | yes | yes | **no** |
| Module Activation Review | settings administration | yes | yes | yes | **no** |
| Module | internal user | yes | no | no | no |
| Module Category | internal user | yes | no | no | no |
| Module Dependency | internal user | yes | no | no | no |
| Module Exclusion | internal user | yes | no | no | no |
| Recycling Model | settings administration | yes | yes | yes | yes |
| Recycling Record | settings administration | yes | yes | yes | yes |
| In-Application Purchase Account | settings administration | yes | yes | yes | yes |
| In-Application Purchase Account | internal user | yes | **no** | yes | **no** |
| In-Application Purchase Service | settings administration | yes | yes | yes | yes |
| In-Application Purchase Service | internal user | yes | no | no | no |
| Geocoding Provider | internal user | yes | no | no | no |
| Privacy Lookup Wizard | settings administration | yes | yes | yes | yes |
| Privacy Lookup Wizard Line | settings administration | yes | yes | yes | **no** |
| Privacy Log | settings administration | yes | yes | yes | yes |
| Code Translation | settings administration | yes | **no** | **no** | **no** |
| Onboarding | nobody | no | no | no | no |
| Onboarding | internal user | no | no | no | no |
| Onboarding | settings administration | yes | yes | yes | yes |
| Onboarding Step | nobody | no | no | no | no |
| Onboarding Step | internal user | no | no | no | no |
| Onboarding Step | settings administration | yes | yes | yes | yes |
| Onboarding Progress Tracker | nobody | no | no | no | no |
| Onboarding Progress Tracker | internal user | no | no | no | no |
| Onboarding Progress Tracker | settings administration | yes | yes | yes | yes |
| Onboarding Progress Step Tracker | nobody | no | no | no | no |
| Onboarding Progress Step Tracker | internal user | no | no | no | no |
| Onboarding Progress Step Tracker | settings administration | yes | yes | yes | yes |
| Tour | settings administration | yes | yes | yes | yes |
| Tour | internal user | yes | no | no | no |
| Tour Step | settings administration | yes | yes | yes | yes |
| Tour Step | internal user | yes | no | no | no |
| Sparse Fields Test | settings administration | yes | yes | yes | **no** |

**Why the four onboarding entities declare rights that grant nothing.** Declaring a row for the
anonymous reader and for the internal user with all four rights refused makes the refusal explicit
rather than inherited, so that a package adding a right later has to say so. Every operation that
records progress runs with elevated rights, which is how an ordinary user's progress is stored
although the user may not read the record.

**Why an internal user may create a metered account but not change one.** The find-or-create
procedure of [`workflows.md`](workflows.md) §14.1 runs on behalf of whoever needed the service. It
must be able to create the account; it must never let that user edit the token or the alert
configuration.

---

# 6. Record rules

| Rule | Entity | Groups it applies to | Condition | Effect |
|---|---|---|---|---|
| "Import: access own records" | Data Import Session | every group | the session's creator is the reader | A user sees only their own uploads. |
| "User In Application Purchase Account" | In-Application Purchase Account | internal user | the account's company list is empty, **or** it intersects the reader's allowed companies | An account scoped to companies is invisible outside them. Both rules are shipped with the no-update marker, so an installation may edit them and keep the edit through an upgrade of the package. |

No other entity of the domain carries a record rule.

---

# 7. Field-level visibility

Some fields are readable only by a group, whatever the record rules say.

| Entity | Field | Visible to |
|---|---|---|
| In-Application Purchase Account | `account_token` (Account Token) | settings administration |
| Google Gmail Mixin, on both mail-server entities | `google_gmail_refresh_token` (Refresh Token) | settings administration |
| Google Gmail Mixin | `google_gmail_access_token` (Access Token) | settings administration |
| Google Gmail Mixin | `google_gmail_access_token_expiration` (Access Token Expiration Timestamp) | settings administration |
| Google Gmail Mixin | `google_gmail_uri` (Resource identifier) | settings administration |
| Microsoft Outlook Mixin | `microsoft_outlook_refresh_token` (Outlook Refresh Token) | settings administration |
| Microsoft Outlook Mixin | `microsoft_outlook_access_token` (Outlook Access Token) | settings administration |
| Microsoft Outlook Mixin | `microsoft_outlook_access_token_expiration` (Outlook Access Token Expiration Timestamp) | settings administration |
| Microsoft Outlook Mixin | `microsoft_outlook_uri` (Authentication Resource identifier) | settings administration |

The account token's help text is "Account token is your authentication key for this service. Do not
share it." Its stored length is limited to forty-three characters.

---

# 8. Scheduled jobs

| Job | Runs | Interval at installation | Active at installation | What it does |
|---|---|---|---|---|
| "Automation Rules: check and execute" | the time-based automation processor | every 4 hours | **no** | Processes every rule with a time trigger, as [`workflows.md`](workflows.md) §6. It is switched on automatically the first time a rule with a time trigger exists and off again when none does; its interval is recomputed by [`calculations.md`](calculations.md) §3.1, but only ever shortened automatically. |
| "Data Recycle: Clean Records" | the recycling search and the notification | every 1 day, first run at three in the morning of the following day | yes | Searches every rule for candidates, validates the candidates of automatic rules, then sends the notifications due, as [`workflows.md`](workflows.md) §19.2 and §19.3. |
| "Transifex: Reload code translations" | the translation cache reload | every 7 days | yes | Empties the cached term table and loads it again for every installed package and every installed language except the base one, as [`calculations.md`](calculations.md) §16.3. |

**The housekeeping sweep.** One more piece of scheduled work belongs to the domain but is not a job
of its own: the platform's periodic vacuum calls the documentation package's sweep, which deletes
every cached documentation index attachment whose registry sequence is not the current one
([`calculations.md`](calculations.md) §7.4).

---

# 9. Message and notification templates

## 9.1 The package activation request

| Property | Value |
|---|---|
| Identifier | `base_install_request.mail_template_base_install_request` |
| Name | "Mail: Install Request" |
| Applies to | Module Activation Request |
| Subject | "Module Activation Request for \"{{ object.module_id.shortdesc }}\"" — the placeholder is the short description of the requested package |
| Sender | the requester's formatted address, or the reading user's when the requester has none |
| Recipient | the reviewer being written to, one message per reviewer |
| Language | the reviewer's own language, or the reading user's |
| Deleted after sending | yes |
| Layout | the light notification layout |

**Body.** A greeting "Hello,", then a line reading the requester's name, the words "has requested to
activate the", the package's short description and the word "module."; then the requester's own
justification, quoted; then a button labelled "Review Request" leading to the review screen for that
package; then "Thanks," and, when the requester has one, their signature. The button's colours are
taken from the requester's company, falling back to the two shipped colours.

**Sending.** Sent immediately, once per reviewer, the reviewer list being every member of the
settings-administration group.

**Answer to the requester.** A success notice reading "Your request has been successfully sent",
after which the request screen closes.

## 9.2 The recycling notification

| Property | Value |
|---|---|
| Identifier | `data_recycle.notification` |
| Kind | a rendered fragment, not a mail template |
| Subject of the message it is posted with | "Data to Recycle" |
| Posted on | the Recycling Model itself |
| Recipients | the Contacts of the rule's chosen recipients |

**Body.** "We've identified " then the count, then " records to clean with the '" then the record
type's display name, then "' recycling rule." followed by a line break and "You can validate those
changes " then a link labelled "here" pointing at the candidate list filtered to that rule, then a
full stop.

## 9.3 The consent failure pages

| Identifier | Shown when |
|---|---|
| `google_gmail.google_gmail_oauth_error` | The first provider's callback receives a failure, or the token exchange fails, or the consented address does not match |
| `microsoft_outlook.microsoft_outlook_oauth_error` | The same for the second provider |

Both render one failure text and one link. The text is the provider's own failure when there is one
and otherwise "An error occurred during the authentication process."; the link goes to the
application root when the provider returned a failure and to the record otherwise.

## 9.4 The onboarding panel fragments

| Identifier | Purpose |
|---|---|
| `onboarding.onboarding_panel` | Renders the panel: one card per step, each carrying its title, description, completion icon, button text, completion text, image, image alternative text, opening operation and rendering state |
| `onboarding.onboarding_container` | The frame around the cards, including the confirmation dialogue titled "Hide Onboarding Tips" whose body reads "Are you sure you want to hide these configuration steps?" and whose confirming button reads "Get them out of my sight!" |
| `onboarding.onboarding_step` | One card |

## 9.5 The documentation page

| Identifier | Purpose |
|---|---|
| `api_doc.docclient` | The single page that presents the reflection documentation. It is served with the framing prohibition set to deny. |

## 9.6 The digest tip

| Property | Value |
|---|---|
| Identifier | `base_automation.digest_tip_base_automation_0` |
| Name | "Tip: Automate everything with Automation Rules" |
| Order | 3 700 |
| Shown to | the settings-administration group |

**Body.** A heading repeating the name and a paragraph reading "Send an email when an object changes
state, archive records after a month of inactivity or remind yourself to follow-up on tasks when a
specific tag is added." then a line break and "With Automation Rules, you can automate any
workflow.", followed by an illustration.

## 9.7 The browser notifications

Four notifications are pushed to a single user's own browser channel rather than being stored.

| Channel name | Payload | Raised by |
|---|---|---|
| `iap_notification` | the message and the kind `success`, with an optional title | a metered call that succeeded |
| `iap_notification` | the message and the kind `danger`, with an optional title | a metered call that failed |
| `iap_notification` | the message and a given kind, with an optional title | any metered status report |
| `iap_notification` | a title, the kind `no_credit` and the purchase address of the named service | a metered call that found the balance exhausted |
| `simple_notification` | the kind `danger`, the title "Warning" and the message "No match found for %(partner_names)s address(es)." | address resolution that found nothing, the placeholder being the comma-separated display names |

---

# 10. Activity types

**The domain defines no activity type.** The Automation Rule carries the activity behaviour, so an
activity can be scheduled on a rule, but the types available are those the messaging domain defines;
see [`../messaging-and-activities/configuration.md`](../messaging-and-activities/configuration.md).
An Automation Rule may of course carry an action that schedules an activity on the record it
watches — except under the deletion trigger, which
[`business-rules.md#aut-007`](business-rules.md#aut-007) forbids.

---

# 11. Shipped records

## 11.1 Metered services

| Identifier | Name | Technical name | Description | Unit name | Whole units |
|---|---|---|---|---|---|
| `iap.iap_service_reveal` | Lead Generation | `reveal` | "Get quality leads and opportunities: convert your website visitors into leads, generate leads based on a set of criteria and enrich the company data of your opportunities." | `Credits` | yes |
| `partner_autocomplete.iap_service_partner_autocomplete` | Partner Autocomplete | `partner_autocomplete` | "Automatically enrich your contact base with corporate data." | `Enrichments` | yes |

Both are shipped as updatable records, so an upgrade of the package restores their descriptions.

## 11.2 Geocoding providers

| Identifier | Name | Technical name |
|---|---|---|
| `base_geolocalize.geoprovider_open_street` | Open Street Map | `openstreetmap` |
| `base_geolocalize.geoprovider_google_map` | Google Place Map | `googlemap` |

The technical name is what selects the behaviour; adding a row whose technical name has no behaviour
raises [`business-rules.md#aut-080`](business-rules.md#aut-080).

## 11.3 System parameters shipped as records

| Identifier | Key | Value |
|---|---|---|
| `microsoft_account.config_microsoft_redirect_uri` | `microsoft_redirect_uri` | `urn:ietf:wg:oauth:2.0:oob` |
| `transifex.transifex_project_url` | `transifex.project_url` | `https://app.transifex.com/odoo` |

Both are shipped with the no-update marker, so an installation may change them permanently.

## 11.4 Server actions shipped as menu entries

| Identifier | Name | Bound to | Restricted to | What it runs |
|---|---|---|---|---|
| `privacy_lookup.ir_action_server_action_privacy_lookup_partner` | Privacy Lookup | Contact, form view | settings administration | Opens the privacy wizard for that Contact |
| `privacy_lookup.ir_action_server_action_privacy_lookup_user` | Privacy Lookup | User, form view | settings administration | Opens the privacy wizard for that User's Contact |
| `privacy_lookup.ir_actions_server_archive_all` | Archive Selection | Privacy Lookup Wizard Line, list and card views | — | Archives every selected line's record |
| `privacy_lookup.ir_actions_server_unlink_all` | Delete Selection | Privacy Lookup Wizard Line, list and card views | — | Deletes every selected line's record |
| `transifex.action_code_translations` | Transifex Code Translations | Code Translation | settings administration | Loads the cache if needed and opens the list |
| `web_tour.tour_export_js_action` | "Export JS" | Tour, form view | — | Produces the exported description of [`calculations.md`](calculations.md) §17.2 and downloads it |

## 11.5 The digest tip

See §9.6.

## 11.6 A menu whose group restriction is removed

The package that adds activation requests **clears the group restriction of the technical-settings
menu**, so that an ordinary internal user can reach the package list and press *Request Access*.
That is a shipped change to an existing record, and a rebuild that keeps the restriction will make
the request feature unreachable.

---

# 12. Constants that are not settings

These values cannot be configured. They are listed here because a rebuild must reproduce them and an
administrator will look for them.

| Constant | Value | Where used |
|---|---|---|
| Resting scheduler interval | 4 hours | [`calculations.md`](calculations.md) §3.1 |
| Shortest scheduler interval | 1 minute | the same |
| Scheduler tolerance | one tenth of the shortest delay | the same |
| Delay factor, minutes | 1 minute | the same |
| Delay factor, hours | 60 minutes | the same |
| Delay factor, days | 1 440 minutes | the same |
| Delay factor, months | 43 200 minutes, that is thirty days | the same |
| Outgoing webhook timeout | 1 second | [`calculations.md`](calculations.md) §2.3 |
| Import session survival | 12 hours | [`entities.md`](entities.md) §7.2 |
| Field tree depth | 3 | [`calculations.md`](calculations.md) §5.1 |
| Error preview length | 200 bytes | [`calculations.md`](calculations.md) §5.2 |
| Download block size | 32 768 bytes | [`calculations.md`](calculations.md) §5.10 |
| Fuzzy match threshold | 0.2 | [`calculations.md`](calculations.md) §5.5 |
| Preview row count | 10 | [`workflows.md`](workflows.md) §9.2 |
| Example values per column | 5, each truncated to 50 characters | the same |
| Maximum archive member size | 104 857 600 bytes | [`calculations.md`](calculations.md) §6.1 |
| Metered call timeout | 15 seconds | [`calculations.md`](calculations.md) §4 |
| Automatic enrichment timeout | 5 seconds | [`workflows.md`](workflows.md) §16.1 |
| Lead enrichment timeout | 300 seconds | [`entities.md`](entities.md) §5.2 |
| Address completion timeout | 2.5 seconds | [`workflows.md`](workflows.md) §18 |
| Reverse geocoding timeout | 10 seconds | [`calculations.md`](calculations.md) §13.3 |
| Automatic recycling batch | 5 000 | [`calculations.md`](calculations.md) §9.3 |
| Manual recycling batch | 50 000 | the same |
| Outside-store upload address lifetime | 300 seconds | [`calculations.md`](calculations.md) §14.4 |
| Outside-store download address lifetime | 300 seconds | the same |
| Redirection cache margin | 10 seconds | the same |
| Delegation key lifetime | 7 days, refreshed when less than 1 day remains | [`calculations.md`](calculations.md) §14.2 |
| Signing service version | `2023-11-03` | the same |
| Mail token request timeout | 5 seconds | [`calculations.md`](calculations.md) §15.2 |
| Mail token validity threshold | 10 seconds | the same |
| Account token length | 32 characters, field limit 43 | [`calculations.md`](calculations.md) §7.1 |
| Connected-device event lifetime | 5 seconds | [`workflows.md`](workflows.md) §26.3 |
| Connected-device long-poll wait | 50 seconds | the same |
| Connected-device session timeout | 70 seconds | the same |
| Connected-device retry delay | 1.5 seconds, growing, capped at 15 seconds | the same |

---

# 13. Behaviour when the database is a neutralised copy

A copy of a production database made for testing is marked as neutralised. Three parts of this
domain react to the mark, so that a copy cannot spend real balance or talk to real services.

| What happens | Where |
|---|---|
| Every existing metered account token has the suffix `+disabled` appended, replacing any suffix it already had, but only when the token is at most thirty-three characters long. A token longer than that — which can only be a malformed one — is replaced entirely by `dummy_value+disabled`. | the metered package's neutralisation step |
| Every newly created metered account has its token replaced by the part before the first plus sign followed by `+disabled`. | the account creation itself, which reads `database.is_neutralized` |
| The address-completion service key is replaced by the text `dummy`. | the address-completion package's neutralisation step |
| The chosen outside-store provider parameter is deleted, which switches the feature off. | the outside-store package's neutralisation step |

The purchase address still works on a neutralised copy, because the hash is taken of the token
truncated at the plus sign ([`calculations.md`](calculations.md) §4.2).

---

# 14. What a fresh installation looks like

With every package of this domain installed and nothing configured:

1. **No automation rule exists.** The scheduled action for time-based rules exists but is switched
   off.
2. **Two metered services exist** and no account. The first account of a service is created the
   first time something needs it.
3. **Two geocoding providers exist.** The chosen one is whichever the parameter names, and, with no
   parameter, the first found.
4. **No outside store is enabled**, so every attachment stays in the ordinary store.
5. **No mail server uses a delegated authorisation**, and neither provider's credentials are
   configured, so pressing the connect button either uses the relay or refuses with
   [`business-rules.md#aut-121`](business-rules.md#aut-121).
6. **No recycling rule exists**, so the nightly job finds nothing and notifies nobody.
7. **The translation term cache is empty** until the weekly job or the menu entry loads it.
8. **The documentation group holds only the root user**, and every settings administrator inherits
   it.
9. **No onboarding panel and no tour is shipped by this domain**; other domains ship theirs and this
   domain provides the records that hold them.
10. **Every import mapping table is empty**, so the first import of a file proposes matches purely
    by name and distance.
