# Automation and Integration

## Scope

This folder specifies the part of the system that makes the system act on its own, and the part
that makes it talk to the outside world.

Two families of behaviour are described, and they are more closely related than they first appear.

**Automation.** A configured rule watches a record type. When something happens to a record of that
type — it is created, a field changes, a tag is added, a stage is reached, a delay after a date
elapses, a message arrives, or an outside system calls a secret web address — the rule runs a
list of actions against that record. The rule owns no business meaning of its own; it borrows the
meaning of whatever record type it is attached to. That is why this domain sits underneath every
other domain in the repository: a sales order that mails its salesperson after three days of
inactivity, a manufacturing order that is archived a month after completion, a lead that is
assigned when a tag is added — all of them are the same mechanism configured differently.

**Integration.** Records enter and leave the system through a set of doors: a spreadsheet or
delimited file is mapped onto a record type and loaded; a packaged data module is uploaded and
installed; an outside program calls the system over a remote interface; the system calls an outside
metered service and is billed for it in units; attachments are stored in an outside object store
instead of the local one; electronic mail accounts are linked by delegated authorisation instead of
a password; postal addresses are resolved to coordinates; company data is fetched from a commercial
directory; translated terms are linked back to a translation platform; connected peripherals are
reached through a local agent.

Two smaller subjects complete the folder because they are configuration-driven guidance rather than
business records: the **onboarding panel**, which tracks which setup steps a company has completed,
and the **guided tour**, which walks a user through the interface. Both are stored records with a
completion state, both are driven from the client, and neither belongs to any business domain.

Finally the domain owns two housekeeping subjects that operate on other domains' records without
knowing anything about them: **data recycling**, which finds records that have aged past a
threshold and archives or deletes them, and **privacy lookup**, which finds every record in the
whole database that mentions one person and offers to archive or delete them one by one.

## What is in scope

| Subject | Where specified |
|---|---|
| Automation Rules: every trigger, the computed trigger helpers, the two filters, the watched-field set, the registry patching, the recursion guard, the time-based scheduler and its interval arithmetic | `entities.md`, `state-machines.md`, `workflows.md`, `calculations.md`, `business-rules.md` |
| Incoming webhooks: the secret address, the identifier rotation, the record-finding expression, the call log, the refusal answers | `entities.md`, `workflows.md`, `interfaces.md`, `business-rules.md` |
| Outgoing webhooks: the payload shape, the fixed keys, the send-and-forget delivery, the timeout | `workflows.md`, `calculations.md`, `interfaces.md` |
| Server Actions as used by automations: the extra usage value, the back-link, the stricter model restriction, the extra evaluation names | `entities.md`, `business-rules.md` |
| Metered service accounts and services: the token, the balance, the alert threshold, the company scoping, the purchase address, the credit query, the notifications | `entities.md`, `workflows.md`, `calculations.md`, `configuration.md` |
| Data import: file reading for four formats, header type inference, column-to-field matching with its distance score, deduplication, multi-column merging, fallback values, the dry run, the saved mapping | `workflows.md`, `calculations.md`, `business-rules.md`, `entities.md` |
| Module import: the archive contract, dependency ordering, the static and translation extraction, the asset declarations, the uninstall behaviour, the remote catalogue | `workflows.md`, `business-rules.md`, `interfaces.md` |
| Module activation requests and reviews: the request mail, the reviewer list, the dependency expansion | `entities.md`, `workflows.md`, `configuration.md` |
| Geocoding: the provider registry, the two providers, the query-string builders, the reverse lookup, the address autocomplete | `entities.md`, `calculations.md`, `interfaces.md` |
| Data recycling: rules, candidate search, the age threshold, manual and automatic modes, archive and delete, the notification cadence | `entities.md`, `workflows.md`, `calculations.md`, `state-machines.md` |
| Privacy lookup: the whole-database search, the anonymisation of the log, the per-line archive and delete, the mass operations | `entities.md`, `workflows.md`, `calculations.md`, `business-rules.md` |
| Cloud storage of attachments: the extra attachment kind, the blob naming, the signed upload and download addresses, the two providers, the provider switch guard | `entities.md`, `workflows.md`, `calculations.md`, `configuration.md` |
| Mail provider accounts: the two delegated-authorisation behaviours, the token lifecycle, the cross-site protection token, the authentication string, the mail-server constraints | `entities.md`, `state-machines.md`, `workflows.md`, `business-rules.md` |
| Translation platform links: the project map, the address formula, the cached code-translation table, the reload job | `entities.md`, `calculations.md`, `configuration.md` |
| Guided tours: the tour, its steps, the consumption record, the shareable address, the export | `entities.md`, `workflows.md`, `interfaces.md` |
| Onboarding panels: panels, steps, per-company progress, the three completion states, the once-only celebration | `entities.md`, `state-machines.md`, `workflows.md` |
| Connected device foundation: the local agent, the action and event contracts, the long-poll protocol, device detection | `interfaces.md`, `workflows.md` |
| Remote interfaces: the two historic call endpoints, the current typed endpoint, the fault codes, the version endpoint | `interfaces.md`, `business-rules.md` |
| Reflection documentation service: the index, the per-record-type document, the caching, the restricted group | `interfaces.md`, `configuration.md` |
| Attachment content extraction for five document families | `calculations.md`, `workflows.md` |
| Sparse storage of rarely filled fields | `entities.md`, `calculations.md` |
| Every access right, record rule, security group, system parameter, scheduled job, message template and route of the domain | `configuration.md`, `interfaces.md` |

## What is out of scope

| Subject | Folder |
|---|---|
| Messages, conversations, followers, activities, outgoing and incoming mail servers as records, mail templates as a rendering engine | [`../messaging-and-activities/`](../messaging-and-activities/) |
| Users, groups, access rights as a mechanism, authorisation keys, session handling | [`../identity-and-access/`](../identity-and-access/) |
| Contacts themselves, their coordinates fields, their company hierarchy, the city list | [`../contacts-and-organizations/`](../contacts-and-organizations/) |
| Record types, fields, views, menus, actions and modules as platform concepts | [`../platform-foundation/`](../platform-foundation/) |
| Text-message sending and its own metered service usage | [`../messaging-and-activities/`](../messaging-and-activities/) |
| Lead enrichment as a sales behaviour (this folder describes only the metered call) | [`../customer-relationship-management/`](../customer-relationship-management/) |
| Structured document exchange with trading partners | [`../electronic-invoicing-and-document-exchange/`](../electronic-invoicing-and-document-exchange/) |
| Point-of-sale use of connected peripherals | [`../point-of-sale/`](../point-of-sale/) |
| Spreadsheet documents and dashboards | [`../spreadsheets-and-dashboards/`](../spreadsheets-and-dashboards/) |

## Entities the folder owns

Every entity below is specified in full in [`entities.md`](entities.md). The transport name is the
identifier an outside caller uses; the storage name is the database table.

### Automation

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Automation Rule | `base.automation` | `base_automation` | One watched record type, one trigger, one condition pair and a list of actions |

### Metered outside services

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| In-Application Purchase Service | `iap.service` | `iap_service` | One purchasable outside service, its unit name and whether its balance is whole |
| In-Application Purchase Account | `iap.account` | `iap_account` | One credential for one service, with its mirrored balance and alert settings |
| Lead Enrichment Interface | `iap.enrich.api` | none (behaviour) | The call that turns electronic mail domains into company data |
| Partner Autocomplete Interface | `iap.autocomplete.api` | none (behaviour) | The call that searches and enriches company records |

### Data and module import

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Data Import Session | `base_import.import` | `base_import_import` | One uploaded file being mapped onto one record type |
| Data Import Column Mapping | `base_import.mapping` | `base_import_mapping` | A remembered column-heading-to-field association for one record type |
| Module Import Wizard | `base.import.module` | `base_import_module` | One uploaded packaged module archive being installed |
| Module Activation Request | `base.module.install.request` | `base_module_install_request` | A non-administrator's request to have a package activated |
| Module Activation Review | `base.module.install.review` | `base_module_install_review` | The administrator's confirmation screen for that request |

### Housekeeping over other domains' records

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Recycling Model | `data_recycle.model` | `data_recycle_model` | One recycling rule: a record type, a filter, an age threshold and an action |
| Recycling Record | `data_recycle.record` | `data_recycle_record` | One candidate found by a rule, awaiting validation or discard |
| Privacy Log | `privacy.log` | `privacy_log` | The anonymised trace of one privacy handling session |
| Privacy Lookup Wizard | `privacy.lookup.wizard` | `privacy_lookup_wizard` | One search for a person across every record type |
| Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `privacy_lookup_wizard_line` | One record found by that search, with its archive and delete controls |

### Outside accounts and platforms

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Google Service | `google.service` | none (behaviour) | The delegated-authorisation and request helper for one service family |
| Microsoft Service | `microsoft.service` | none (behaviour) | The same helper for the other service family |
| Google Gmail Mixin | `google.gmail.mixin` | none (behaviour) | Token storage and renewal for one electronic mail provider |
| Microsoft Outlook Mixin | `microsoft.outlook.mixin` | none (behaviour) | Token storage and renewal for the other electronic mail provider |
| Geocoding Provider | `base.geo_provider` | `base_geo_provider` | One address-resolution provider |
| Geocoder | `base.geocoder` | none (behaviour) | The dispatching address-resolution service |
| Transifex Translation | `transifex.translation` | none (behaviour) | The builder of a deep link into the translation platform |
| Code Translation | `transifex.code.translation` | `transifex_code_translation` | One cached source term and its translation for one package and language |

### Guidance

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Onboarding | `onboarding.onboarding` | `onboarding_onboarding` | One setup panel, identified by a one-word route segment |
| Onboarding Step | `onboarding.onboarding.step` | `onboarding_onboarding_step` | One card of one or more panels, with its opening operation |
| Onboarding Progress Tracker | `onboarding.progress` | `onboarding_progress` | One panel's completion state for one company |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `onboarding_progress_step` | One step's completion state for one company |
| Tour | `web_tour.tour` | `web_tour_tour` | One guided walkthrough, its starting address and its closing message |
| Tour Step | `web_tour.tour.step` | `web_tour_tour_step` | One click or input in a walkthrough |

### Storage technique

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Sparse Fields Test | `sparse_fields.test` | `sparse_fields_test` | The reference record that exercises serialised storage of rarely filled fields |

## Entities the folder extends but does not own

| Entity | Transport name | Owner | What this folder adds |
|---|---|---|---|
| Server Action | `ir.actions.server` | [`../platform-foundation/`](../platform-foundation/) | The `base_automation` usage value, the back-link to the rule, a stricter record-type restriction, a warning, and two extra names in the code evaluation context |
| Scheduled Action | `ir.cron` | [`../platform-foundation/`](../platform-foundation/) | A jump to the owning rule |
| Attachment | `ir.attachment` | [`../platform-foundation/`](../platform-foundation/) | The `cloud_storage` kind with its blob naming and signed addresses, and content extraction for five document families |
| Module | `ir.module.module` | [`../platform-foundation/`](../platform-foundation/) | The imported flag, the catalogue kind, the archive installer, the remote catalogue reader and the activation-request button |
| View Definition | `ir.ui.view` | [`../platform-foundation/`](../platform-foundation/) | Views coming from an imported package are validated as custom views |
| Field Definition | `ir.model.fields` | [`../platform-foundation/`](../platform-foundation/) | The `serialized` field kind and the serialisation-field link |
| Mail Server | `ir.mail_server` | [`../messaging-and-activities/`](../messaging-and-activities/) | Two delegated-authorisation kinds with their host, port, encryption and constraint sets |
| Incoming Mail Server | `fetchmail.server` | [`../messaging-and-activities/`](../messaging-and-activities/) | The same two kinds for incoming mail |
| Contact | `res.partner` | [`../contacts-and-organizations/`](../contacts-and-organizations/) | The privacy-lookup entry point, the autocomplete widgets and the enrichment note |
| Company | `res.company` | [`../contacts-and-organizations/`](../contacts-and-organizations/) | The one-time automatic enrichment flag and its guard |
| User | `res.users` | [`../identity-and-access/`](../identity-and-access/) | The tour switch, the remote-file import permission hook, the linked outside calendar tokens and the personal mail-server kinds |
| Configuration Settings | `res.config.settings` | [`../platform-foundation/`](../platform-foundation/) | Every setting listed in [`configuration.md`](configuration.md) |
| Request Routing | `ir.http` | [`../platform-foundation/`](../platform-foundation/) | Three additions to the session description and the translation source for imported packages |

## Reading order

1. [`README.md`](README.md) — this file: scope, entities, dependencies.
2. [`entities.md`](entities.md) — every record and every field.
3. [`state-machines.md`](state-machines.md) — the six state fields and their transitions.
4. [`workflows.md`](workflows.md) — the twenty-one end-to-end procedures.
5. [`business-rules.md`](business-rules.md) — every refusal, with its exact message.
6. [`calculations.md`](calculations.md) — every formula, with worked examples.
7. [`accounting-effects.md`](accounting-effects.md) — why the domain writes no ledger entry, and what it triggers elsewhere.
8. [`configuration.md`](configuration.md) — settings, parameters, groups, access, jobs, templates, shipped records.
9. [`interfaces.md`](interfaces.md) — menus, views, operations, routes, external services.
10. [`acceptance-criteria.md`](acceptance-criteria.md) — numbered scenarios with concrete numbers.
11. [`glossary.md`](glossary.md) — every term used above.

## Files in this folder

| File | Content |
|---|---|
| [`README.md`](README.md) | Scope, entity inventory, dependencies, reading order, file list |
| [`entities.md`](entities.md) | All thirty owned entities in full, plus the extension points added to eleven foreign entities |
| [`state-machines.md`](state-machines.md) | The six state machines with stored values, transitions, guards and diagrams |
| [`workflows.md`](workflows.md) | Twenty-one operational procedures, step by step |
| [`business-rules.md`](business-rules.md) | Ninety-four numbered rules with their exact refusal text |
| [`calculations.md`](calculations.md) | Twenty-six formulas and algorithms with worked examples |
| [`accounting-effects.md`](accounting-effects.md) | The reasoned statement that the domain posts nothing, and the indirect effects |
| [`configuration.md`](configuration.md) | Settings, system parameters, groups, access rights, record rules, scheduled jobs, templates, shipped data |
| [`interfaces.md`](interfaces.md) | Menus, views, named operations, routes, external services, import and export |
| [`acceptance-criteria.md`](acceptance-criteria.md) | Eighty-eight numbered scenarios |
| [`glossary.md`](glossary.md) | Every term of the domain, defined |

## Dependencies on other domains

| Depends on | For what |
|---|---|
| [`../platform-foundation/`](../platform-foundation/) | Record types, fields, views, actions, modules, attachments, scheduled actions, system parameters, logging |
| [`../identity-and-access/`](../identity-and-access/) | Users, groups, the administrator test, the technical-features group, session information |
| [`../messaging-and-activities/`](../messaging-and-activities/) | The message thread on the Automation Rule and the metered account, the notification used by recycling, the outgoing and incoming mail servers extended by the two delegated-authorisation behaviours, the message templates |
| [`../contacts-and-organizations/`](../contacts-and-organizations/) | Contacts and companies as the subjects of enrichment, geocoding and privacy lookup |
| [`../customer-relationship-management/`](../customer-relationship-management/) | The lead record that carries the enrichment correlation identifier |

## Domains that depend on this one

Every domain that ships an automation rule, an onboarding panel, a guided tour, an import template
or a metered service depends on this folder. The heaviest users are
[`../customer-relationship-management/`](../customer-relationship-management/) (lead enrichment and
lead generation), [`../messaging-and-activities/`](../messaging-and-activities/) (text messages and
postal mail as metered services), [`../contacts-and-organizations/`](../contacts-and-organizations/)
(company autocomplete and geocoding) and
[`../point-of-sale/`](../point-of-sale/) (connected peripherals).

## Platform documents

- [`../../overview/architecture.md`](../../overview/architecture.md) — how packages, records and the registry fit together.
- [`../../overview/entity-and-field-system.md`](../../overview/entity-and-field-system.md) — field kinds, computed fields, stored and unstored values, the meaning of a transient record.
- [`../../overview/inheritance-and-extension.md`](../../overview/inheritance-and-extension.md) — how one package adds fields and behaviour to another package's record type; the mechanism every behaviour in this folder uses.
- [`../../overview/package-system.md`](../../overview/package-system.md) — what a package is, what its manifest declares, how it is installed.
- [`../../overview/security-model.md`](../../overview/security-model.md) — groups, access rights, record rules, elevated execution.
- [`../../overview/views-and-actions.md`](../../overview/views-and-actions.md) — menus, actions, view kinds and the client contract.
- [`../../overview/messaging-model.md`](../../overview/messaging-model.md) — the message thread, used by the Automation Rule and the metered account.
- [`../../runtime/README.md`](../../runtime/README.md) — the request lifecycle that the routes in this folder plug into.
- [`../../data/README.md`](../../data/README.md) — the shape of the machine-readable catalogues.
- [`../../interfaces/README.md`](../../interfaces/README.md) — the interface catalogue of the whole system.
- [`../../references/entities/base.automation.md`](../../references/entities/base.automation.md) — the generated reference page for the Automation Rule; every other owned entity has one beside it.
