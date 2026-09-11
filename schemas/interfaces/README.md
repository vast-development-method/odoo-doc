# Interface catalogues

What the system exposes to people and to other systems. See [the catalogue index](../README.md) for the record shape.

| Catalogue | Content | Records |
|---|---|---|
| [`routes.json`](routes.json) | Every route: path, request kind, authentication level, methods, purpose and validation messages | 1,023 |
| [`window-actions.json`](window-actions.json) | Actions that open views on an entity, with their filter, context, view modes and target | 978 |
| [`server-actions.json`](server-actions.json) | Actions bound to entities that create records, update fields, send messages, call webhooks or run configured logic | 149 |
| [`client-actions.json`](client-actions.json) | Actions that open a screen not bound to one entity | 26 |
| [`address-actions.json`](address-actions.json) | Actions that open an address | 10 |
| [`report-actions.json`](report-actions.json) | Printable document definitions with their template, file naming rule and attachment policy | 94 |
| [`menus.json`](menus.json) | Every menu with its parent, action, sequence and group restriction | 893 |
| [`views/`](views/) | One catalogue per entity holding every view declaration: kind, inheritance, priority, and a structural summary naming the fields shown, the buttons, the filters and the groupings | 671 files |
| [`mail-templates.json`](mail-templates.json) | Message templates with recipients, subject and body | 69 |
| [`web-templates.json`](web-templates.json) | Rendering templates for printable documents, portal pages, storefront pages and messages, with their inheritance and the templates each calls | 2,163 |

The narrative counterparts are [the endpoint catalogue](../../docs/interfaces/endpoint-catalog.md), [remote transport contracts](../../docs/interfaces/remote-transport-contracts.md) and [printable documents and exports](../../docs/interfaces/report-and-export-documents.md).
