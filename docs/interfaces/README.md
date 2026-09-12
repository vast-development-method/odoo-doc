# Interfaces

The contracts through which people and other systems use the platform: the screens a person works in, the request
endpoints a client or an integration calls, the operations those endpoints reach, the documents the platform prints and
exports, and the outside services it exchanges data with. Everything in this folder is the outside of the system; the
behaviour behind each contract is specified in the domain folders under [`../domains/`](../domains/) and in the
platform documents under [`../overview/`](../overview/), [`../runtime/`](../runtime/) and [`../data/`](../data/).

| Document | Content |
|---|---|
| [`desktop-workflows.md`](desktop-workflows.md) | How the desktop client composes views, actions and menus into working screens: the navigation model and the navigation trail, the menu map of every one of the thirty-five applications with the action, entity, views, default filters and access groups of each entry, the ten view kinds and what a person may do on each, searching, filtering, grouping and favourites, the command palette, the keyboard shortcuts, notifications, the button-by-button workflow of every major document, the five kinds of dashboard, and the rules and numbered scenarios that govern all of it. |
| [`endpoint-catalog.md`](endpoint-catalog.md) | Every request endpoint the platform exposes, grouped into thirty-three areas: 855 endpoints on 1029 path patterns, each with its path patterns, transport, authentication level, allowed methods, site-page flag, parameters, purpose, effect, failures and the capability packages that declare and extend it, preceded by the notation, the transports, the authentication levels, the cross-site submission rules, the three tokens that grant access to one record without a session, and the table that derives the complete failure set of any row from its transport, its authentication level, its path converters, its cross-site setting and the records it touches. |
| [`external-integrations.md`](external-integrations.md) | Every touchpoint with a system outside the platform, written as a contract with its direction, trigger, data exchanged, authentication, failure handling, idempotency, configuration and records written: electronic mail in both directions, delivery failures, text messages, postal mail, payment providers, shipping carriers and pickup networks, calendar synchronization, delegated sign-in, directory servers, the purchased data services, document exchange networks, image libraries, anti-robot verification, print on demand, cloud storage, translation, currency rates, address autocompletion, geolocation, maps, video embedding, connected devices, call relay, inbound and outbound webhooks and the publisher account services, with a summary table of all of them. |
| [`remote-transport-contracts.md`](remote-transport-contracts.md) | The request layer end to end: how an endpoint is declared and how overriding declarations merge, the three transport families and their envelopes, the ordered request lifecycle, sessions and their rotation, the four authentication levels and the application keys, the cross-site request forgery rules, the generic dispatch that exposes any entity operation, every endpoint the desktop client relies on with its inputs and outputs, the two alternative remote call protocols, the request context, the batching limits, the retry semantics, the security checks per endpoint and fifty numbered scenarios. |
| [`report-and-export-documents.md`](report-and-export-documents.md) | Printed documents and data files: the report definition with its stored field names, the rendering pipeline, the language rule, the shared layouts, the eighteen paper formats, the catalogue of all ninety-four printed documents with the content of each block by block, the export mechanisms for lists, analysis tables, spreadsheets and price lists, the import procedure with its matching, resolution, test run and messages, the data files the domains produce (electronic invoices, audit files, payment and bank files, label files, archives) and fifteen numbered scenarios. |
| [`service-layer.md`](service-layer.md) | The operation surface: the generic contract of every operation that exists on every entity, the relational write commands, the read formats, the conventions and patterns of business operations, the read specification grammar, the group payloads, the on-change protocol, the export and import contract, the discovery operations, the complete enumeration of the 1,016 named business operations that screens bind to a control grouped into thirty-eight areas, the transaction and concurrency rules, and fifty numbered scenarios. |

## Reading order

A reader new to the system reads [`desktop-workflows.md`](desktop-workflows.md) first, because it shows what the system
looks like to the people who use it. A reader building a client or an integration reads
[`remote-transport-contracts.md`](remote-transport-contracts.md), then [`service-layer.md`](service-layer.md), then
looks up the endpoint it needs in [`endpoint-catalog.md`](endpoint-catalog.md). A reader responsible for documents and
data exchange reads [`report-and-export-documents.md`](report-and-export-documents.md) and
[`external-integrations.md`](external-integrations.md).

## Where the generated catalogues are

These documents describe behaviour; the machine-readable and generated listings of the same material are elsewhere in
the repository and are linked from each document where they apply.

| Listing | Location |
|---|---|
| Every route with its authentication level, request type and purpose | [`../references/routes.md`](../references/routes.md) |
| Every printable report with its entity, template, file-name rule and attachment rule | [`../references/reports.md`](../references/reports.md) |
| Every view declaration grouped by entity | [`../references/views.md`](../references/views.md) |
| Every window, server and client action, and every menu | [`../references/actions-and-menus.md`](../references/actions-and-menus.md) |
| Every operation of every entity, classified by kind | [`../references/operation-index.md`](../references/operation-index.md) |
| The structured catalogues of routes, menus, actions, reports, views and message templates | [`../../schemas/interfaces/`](../../schemas/interfaces/) |

Domain-specific interfaces — the operations, endpoints, printed documents, message templates and scheduled jobs of one
domain — are additionally described in that domain's `interfaces.md`.
