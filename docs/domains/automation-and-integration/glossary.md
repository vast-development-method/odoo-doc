# Glossary

Every term this folder uses in a sense that is not the everyday one, defined. Terms are grouped by
subject and, within a group, alphabetically. A term written in code font is a reproduced identifier
or stored value; its meaning in words follows.

Contents:

1. [Automation](#1-automation)
2. [Webhooks](#2-webhooks)
3. [Data import](#3-data-import)
4. [Packages](#4-packages)
5. [Metered outside services](#5-metered-outside-services)
6. [Company data and addresses](#6-company-data-and-addresses)
7. [Housekeeping over other domains' records](#7-housekeeping-over-other-domains-records)
8. [Attachments stored outside](#8-attachments-stored-outside)
9. [Delegated mail authorisation](#9-delegated-mail-authorisation)
10. [Guidance](#10-guidance)
11. [Translation, documentation and remote calls](#11-translation-documentation-and-remote-calls)
12. [Connected devices](#12-connected-devices)
13. [Terms borrowed from the platform](#13-terms-borrowed-from-the-platform)

---

# 1. Automation

**Action list.** The ordered set of Server Actions attached to an Automation Rule. Every action in
the list inherits the rule's watched record type and carries the usage value `base_automation`,
which is what separates it from an action a user placed on a menu.

**After-filter.** The condition a record must satisfy **after** the change for the rule to run. Its
stored identifier is `filter_domain` and its label is "Apply on". For several triggers it is written
automatically from the trigger and the chosen value; a user may narrow it further.

**Automation Rule.** One watched record type, one trigger, one pair of filters and a list of
actions. The rule owns no business meaning of its own; it borrows the meaning of the record type it
is attached to. Transport name `base.automation`.

**Before-filter.** The condition a record must satisfy **before** the change for the rule to run.
Its stored identifier is `filter_pre_domain` and its label is "Before Update Domain". It is never
checked on creation, because a record that did not exist satisfied nothing. Only the tag trigger
fills it automatically.

**Code action.** A Server Action whose kind is the one that evaluates an expression. It is the only
kind a live-update rule may hold.

**Delay.** The whole number of units a time trigger waits. Its stored identifier is
`trg_date_range`. It is always positive; the direction is carried by the delay mode.

**Delay mode.** Whether the delay is counted **before** the date field or **after** it. Stored values
`before` and `after`.

**Delay unit.** The unit the delay counts in. Stored values `minutes`, `hour`, `day` and `month`.
The unit named in the singular is reproduced as it stands; a month counts as thirty days when the
scheduler compares intervals, and as a calendar month when a window is computed.

**Live-update field set.** The fields whose change on an open form makes a live-update rule run.
Stored identifier `on_change_field_ids`, label "On Change Fields Trigger". Changing this set empties
the client's template cache, because the affected forms must be told to report those fields.

**Live-update trigger.** The trigger `on_change`, which runs the rule against an unsaved form rather
than against a stored record.

**Processing register.** The per-operation record of which rules have already run on which records.
It is what makes a rule fire once per record per operation, however many times the operation writes.

**Recursion guard.** The behaviour the processing register produces: a rule that changes the very
field it watches does not fire itself again.

**Registry patch.** The interception installed on a watched record type so that creating, writing,
deleting or changing a record of that type runs the rules attached to it. Patches are reinstalled
whenever a rule is saved, deleted or duplicated, and every other worker is told to reload.

**Scheduler.** The scheduled action named "Automation Rules: check and execute", which processes
every rule with a time trigger. Its interval is recomputed from the shortest delay and is only ever
shortened automatically.

**Server Action.** A named, configured operation that the platform can run against a record. This
domain does not own the entity; it adds the usage value, the back-link to a rule and a stricter
record-type restriction. Transport name `ir.actions.server`.

**Time trigger.** One of the three triggers processed by the scheduler rather than by an
interception: `on_time`, `on_time_created` and `on_time_updated`.

**Trigger.** What makes a rule run. Nineteen values exist, listed in
[`entities.md`](entities.md) §1.4.

**Trigger date field.** The date or date-and-time field a time trigger watches. Stored identifier
`trg_date_id`.

**Trigger value.** The selection value a stage, state or priority trigger waits for. Stored
identifier `trg_selection_field_id`.

**Trigger reference.** The record a stage, tag or user trigger waits for. Stored identifier
`trg_field_ref`.

**Watched-field set.** The fields whose change makes an update rule run. Stored identifier
`trigger_field_ids`, label "Trigger Fields". An empty set means every field is watched.

**Working schedule.** A calendar of working days used to count a day-based delay, so that a
three-day notice does not fall over a weekend. Stored identifier `trg_date_calendar_id`.

---

# 2. Webhooks

**Call log.** Lines written to the platform's log table when a rule asks for them, each carrying the
rule identifier as its origin so that one rule's whole history can be read at once. Switched on by
the field `log_webhook_calls`, labelled "Log Calls".

**Incoming webhook.** A rule whose trigger is `on_webhook`, reachable at a secret address that any
outside program may call.

**Outgoing webhook.** A Server Action of the kind that submits a structured body to a configured
address after the transaction commits.

**Payload.** The structured body an incoming call carries, or the query arguments of the request
when it carries no body. It is handed to the record-finding expression under the name `payload`.

**Record-finding expression.** The expression that turns a payload into the record the rule should
run on. Stored identifier `record_getter`. Its help text reads "This code will be run to find on
which record the automation rule should be run."

**Secret address.** The address of an incoming webhook, made of the base address, the fixed segment
`/web/hook/` and the rule's webhook identifier. Anyone who knows it can trigger the rule, which is
why it is renewable.

**Send-and-forget.** The delivery discipline of an outgoing webhook: it is posted after the
transaction commits, with a one-second timeout, and a read timeout is treated as neither success nor
failure.

**Webhook identifier.** The universally unique identifier that makes a rule's secret address secret.
Stored identifier `webhook_uuid`. It is not copied when a rule is duplicated.

---

# 3. Data import

**Advanced editor.** The mapping editor that shows paths across relations. It opens on a fresh parse
when any heading or any proposed mapping crosses a relation.

**Column mapping.** The association from a column of the file to a field, or to a path of fields.
Held in the options while an import is being prepared, and remembered afterwards as a Data Import
Column Mapping.

**Data Import Column Mapping.** One remembered association: for this record type, a column with this
heading means this field. Transport name `base_import.mapping`.

**Data Import Session.** One uploaded file being mapped onto one record type. Transient; it survives
twelve hours. Transport name `base_import.import`.

**Dense matrix.** The rows reduced to the mapped columns only, with the heading row and the empty
rows removed.

**Distance.** A number between 0 and 1 saying how unlike two texts are. 0 means identical, 1 means
nothing in common. A heading matches a field when the distance is below 0.2.

**Fallback value.** The value substituted when a cell does not match what a boolean or selection
field accepts. The marker `skip` means the row is skipped instead.

**Field tree.** The importable fields of the target record type, three levels deep, with the
external and database identifiers added under every relation.

**Heading.** The text in the first row of a column when the options say the file has headings.

**Header type.** A kind a column could hold, inferred from its preview values. A column may allow
several.

**Options mapping.** The mapping of reading and loading choices that travels with every call. It is
never stored on the session.

**Preview.** The first ten rows of the file, used to infer the kinds, to build the proposal and to
show the user what will be loaded.

**Proposal.** The system's suggested column mapping, built from remembered mappings, exact matches
and distance, and then deduplicated.

**Skip count.** How many data rows to drop before importing, used to resume a batched import. It is
applied after the heading row is dropped and after empty rows are discarded.

**Trial run.** An import executed inside a savepoint that is always rolled back. It reports what
would happen and changes nothing.

---

# 4. Packages

**Activation request.** A non-administrator's request that a package be activated. Transport name
`base.module.install.request`.

**Activation review.** The administrator's confirmation screen for such a request, showing every
application the activation would bring in. Transport name `base.module.install.review`.

**Archive.** The uploaded file holding one or more packages, each as a top-level directory with a
manifest.

**Asset declaration.** A manifest entry naming a file the client must load. For an imported package
the file is served from an attachment, so a wildcard cannot be resolved and is refused.

**Catalogue.** The remote directory of packages this installation can fetch and install. Reached at
`https://apps.odoo.com`.

**Imported package.** A package installed from an archive rather than from the installation's own
directories. It carries the imported flag, its views are validated as custom views, and uninstalling
it deletes its row.

**Manifest.** The declaration file at the root of a package directory, naming the package's
dependencies, data files and assets.

**Module Import Wizard.** The screen that receives an archive and installs it. Transport name
`base.import.module`.

**Package kind.** Whether a package is an ordinary one or comes from the industry catalogue. Stored
values `official` and `industries`.

---

# 5. Metered outside services

**Account token.** The credential that identifies this installation to one metered service. Thirty-
two hexadecimal characters, visible only to the settings-administration group, never copied when a
record is duplicated.

**Alert threshold.** The balance at or below which the service is asked to write to the chosen
recipients. Pushed to the service when it changes; not enforced by this system.

**Balance.** The count of service units the outside service reports, stored as text already
formatted with the unit name. It is not money and it is not maintained by this system.

**In-Application Purchase Account.** One credential for one metered service, with its mirrored
balance, its alert configuration and its company scope. Transport name `iap.account`.

**In-Application Purchase Service.** One purchasable outside service: its name, its technical name,
its description, its unit name and whether it counts in whole units. Transport name `iap.service`.

**Insufficient balance.** The distinct failure a service raises when the balance will not cover a
call. It carries the remaining balance, the service name, the purchase address and its own message.

**Neutralised copy.** A copy of a production database marked as such. Every account token has
`+disabled` appended so that the copy cannot spend real balance.

**Purchase address.** The address at which more units are bought, built from the endpoint, the
database identifier, the service technical name and the hashed token.

**Registration state.** What the outside service says about the account. Stored values `banned`,
`registered` and `unregistered`.

**Service lock.** The flag that stops the service of an account being changed once the outside
service has recognised it.

**Unit name.** What one unit of a service is called, for example `Credits` or `Enrichments`. It is
translated and it is appended to the formatted balance.

---

# 6. Company data and addresses

**Company directory.** The outside service that suggests and enriches company records. Reached at
`https://partner-autocomplete.odoo.com`.

**Coordinates.** The latitude and the longitude written onto a Contact by address resolution. They
belong to the contacts domain; this domain only computes them.

**Cross-border verification.** The public service consulted when the directory finds nothing for a
tax registration number. Its answer is turned into a single suggestion.

**Enrichment.** Filling a company's Contact from outside data. It happens once automatically at
company creation and on demand from the interface.

**Geocoder.** The dispatching behaviour that turns an address into coordinates by calling whichever
provider is chosen. Transport name `base.geocoder`.

**Geocoding Provider.** One address-resolution provider, identified by a technical name that selects
its behaviour. Transport name `base.geo_provider`.

**Internet domain of a company.** The part after the at-sign of the company's electronic mail
address when that part is not a consumer mail provider, and otherwise the host of the company's site
address.

**Place identifier.** The opaque handle a suggestion carries, used to ask the provider for the full
address.

**Reverse resolution.** Turning coordinates into a description of a place. Used to say where a
visitor is.

**Session token.** The handle a client may pass through a series of address-completion calls so that
the provider bills them as one lookup.

**Suggestion.** One candidate company or one candidate address offered while a user types.

---

# 7. Housekeeping over other domains' records

**Candidate.** One record a recycling rule has found. Transport name `data_recycle.record`. It is
deleted when validated and merely deactivated when discarded.

**Discard.** Marking a candidate as not to be acted on, by clearing its activation flag. A discarded
candidate is not proposed again, because the search reads archived candidates too.

**Include archived.** The rule option that widens the search to archived originals. It is offered
only when the action is deletion.

**Privacy Log.** The masked trace of one handling session: who handled it, when, what was found and
what was done. Exactly one per session. Transport name `privacy.log`.

**Privacy Lookup Wizard.** One search for a person across every record type of the database.
Transport name `privacy.lookup.wizard`.

**Privacy Lookup Wizard Line.** One record that search found, with its archive control, its delete
control and its execution details. Transport name `privacy.lookup.wizard.line`.

**Recycling Model.** One recycling rule: a record type, a filter, an age threshold, a mode and an
action. Transport name `data_recycle.model`.

**Recycling action.** What happens to an original when a candidate is validated. Stored values
`archive` and `unlink`.

**Recycling mode.** Whether the rule proposes or acts. Stored values `manual` and `automatic`.

**Validate.** Applying a rule's action to a candidate's original and then deleting the candidate.

---

# 8. Attachments stored outside

**Blob.** One stored object in an outside store. Its name is the attachment identifier, a fresh
universally unique identifier and the file name, joined by oblique strokes.

**Cross-origin rules.** The store's own declaration of which origins may reach it directly. The
second provider's bucket has them written when the configuration is verified.

**Delegation key.** The short-lived key the first provider issues, with which signatures are made.
Valid seven days, cached per database, refreshed when less than one day remains.

**Minimum file size.** The size at or above which the client uploads straight to the outside store.
Twenty megabytes by default.

**Never-outside record types.** The record types whose attachments must stay local because their own
logic reads the bytes. The client is told the list in the session description.

**Outside store.** A provider that holds attachment bytes instead of the local store. Two are
offered; the stored values are `azure` and `google`.

**Plain address.** The unsigned address that identifies a blob. It is stored on the attachment and
grants no access.

**Probe.** The temporary blob uploaded and downloaded when a provider's configuration is verified.

**Signed address.** A plain address with a signature and its parameters in the query, valid for a
limited time and for one request method.

**Storage kind.** How an attachment's bytes are held. This domain adds the value `cloud_storage`
beside the platform's own values.

---

# 9. Delegated mail authorisation

**Access token.** The short-lived credential presented to the mail protocol. Renewed when it is
missing, when its expiry is missing, or when its expiry minus ten seconds is in the past.

**Authentication string.** The text the mail protocols expect, made of the mailbox address, a
control character, the words `auth=Bearer` and the access token, and two more control characters.

**Consent address.** The address the browser is sent to so that the mailbox owner may agree.

**Cross-site protection token.** A keyed digest over a purpose text and the record's transport name
and identifier. The callback refuses any return whose token does not match in constant time.

**First provider, second provider.** The two electronic mail providers this domain supports. The
first is the one whose outgoing host is `smtp.gmail.com`; the second is the one whose outgoing host
is `smtp.outlook.com`.

**Refresh token.** The long-lived credential from which access tokens are obtained. The second
provider replaces it on every renewal; the first does not.

**Relay path.** The route taken when the installation has no application registration of its own:
the relay obtains the consent, exchanges the code and hands the tokens back.

**Return address.** The address the provider sends the browser back to, carrying the authorisation
code and the state.

**State.** The mapping carried through the consent so that the callback knows which record to write:
the record's transport name, its identifier and the protection token.

---

# 10. Guidance

**Closing message.** The text shown once when a panel becomes complete. Its default is "Nice work!
Your configuration is done."

**Consumption.** The record that a user has seen a tour, kept as a link between the tour and the
user.

**Onboarding.** One setup panel, identified by a one-word route segment. Transport name
`onboarding.onboarding`.

**Onboarding Progress Tracker.** One panel's completion state for one company. Transport name
`onboarding.progress`.

**Onboarding Progress Step Tracker.** One step's completion state for one company. Transport name
`onboarding.progress.step`.

**Onboarding Step.** One card of one or more panels, with its title, its description, its button
text, its completion text and its opening operation. Transport name `onboarding.onboarding.step`.

**Opening operation.** The named operation a step's button runs. A step linked to a panel must have
one.

**Per-company step.** A step whose completion is recorded separately for each company. A panel that
holds one becomes per-company itself and stays so.

**Rendering state.** The state the panel shows, which merges the stored completion states so that a
step just completed can be celebrated once and then settles.

**Route segment.** The one word that identifies a panel in an address. Unique across panels.

**Tour.** One guided walkthrough of the interface, its starting address, its steps and its closing
message. Transport name `web_tour.tour`.

**Tour Step.** One click or one input in a walkthrough: what it acts on, what it does, where the
tooltip sits and what the tooltip says. Transport name `web_tour.tour.step`.

**Tour switch.** The per-user flag that decides whether tours are offered. Its default is true for
an administrator of an installation with no demonstration data, outside a test.

---

# 11. Translation, documentation and remote calls

**Bearer credential.** An authorisation key sent with a request instead of a session. Two
documentation routes and the typed call route accept it.

**Cached term.** One source term and its translation for one package and one language, held so that
a translator can be sent straight to the platform. Transport name `transifex.code.translation`.

**Fault code.** The value a structured-markup call returns when an operation fails. One endpoint
returns texts, the other whole numbers.

**Project map.** The mapping from package to translation project, read once per process from the
platform's configuration files.

**Reflection documentation.** The generated description of every record type, field and public
operation, served as one page and as structured documents.

**Registry sequence.** The number that changes whenever the set of installed packages or defined
record types changes. It is part of the documentation cache key.

**Typed call.** The current call route, which names the record type and the operation in the path
and takes the identifiers and the reading context as named arguments.

---

# 12. Connected devices

**Agent.** The separate program that runs beside the peripherals and answers the device paths. It is
not part of the application server.

**Device kind.** One of the nine names a caller may use instead of a device identifier, in which
case the first device of that kind is used: `display`, `printer`, `scanner`, `keyboard`, `camera`,
`device`, `payment`, `scale` and `fiscal_data_module`.

**Event.** A record that something happened on a device, carrying the moment, the device identifier,
the owning session and the outcome. Events older than five seconds are discarded.

**Listener.** The description a client sends when it waits: its session identifier, the devices it
cares about and the marker of the last event it saw.

**Long poll.** The waiting discipline: a call returns as soon as a newer event arrives and otherwise
after fifty seconds, and the client immediately calls again.

**Session identifier.** The handle that ties a client's actions and the events they produce
together.

---

# 13. Terms borrowed from the platform

These terms are defined in the platform documents and are repeated here only so that a reader of
this folder is not stopped by them.

**Archived.** A record whose activation flag is false. It stays in the database and is hidden from
ordinary searches. See [`../../overview/entity-and-field-system.md`](../../overview/entity-and-field-system.md).

**Base address.** The address at which this installation answers requests.

**Base language.** The language in which labels and messages are written before translation.

**Computed field.** A field whose value is derived rather than typed. It may be stored or not, and
it may be writable when it declares how to write back.

**Elevated rights.** Running an operation with the access of the system rather than of the reader.
Every rule's actions, every recycling disposal and every privacy archive or delete run this way.

**Filter.** A stored list of conditions, each a field path, an operator and a value.

**Mixin, behaviour.** A record type with no table of its own whose fields and operations are added
to other record types. This folder calls them behaviours.

**Reading context.** The mapping of per-call settings — the language, the companies, the flags —
that travels with every operation.

**Record rule.** A condition added to every search and every read of a record type for the members
of a group. See [`../../overview/security-model.md`](../../overview/security-model.md).

**Registry.** The in-memory description of every record type and field, rebuilt when packages change
and reloaded by every worker when it is marked as changed.

**Savepoint.** A point inside a transaction that can be returned to without abandoning the whole
transaction. Every import run is wrapped in one.

**Scheduled action.** A named piece of work the platform runs on an interval. See
[`../../overview/architecture.md`](../../overview/architecture.md).

**Storage name.** The name of the database table behind a record type.

**System parameter.** A stored key and value pair used as configuration. Read with elevated rights.

**Transient record.** A working record the platform deletes after a while. Import sessions, the
package import screen, the activation request and review, and the privacy wizard and its lines are
all transient.

**Transport name.** The name by which an outside caller addresses a record type. It is contractual.

**Universally unique identifier.** A one-hundred-and-twenty-two-bit random identifier in its
canonical thirty-six-character form. Used for webhook identifiers, account tokens and blob names.
