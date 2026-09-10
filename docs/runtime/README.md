# Runtime

How the system behaves while it runs, independent of any deployment.

| Document | Content |
|---|---|
| `request-lifecycle.md` | Routing, dispatching, request types, sessions, authentication levels, cross-site request forgery protection, context propagation, language and time zone resolution, error mapping |
| `transactions-and-concurrency.md` | Unit of work, flush ordering, database transactions, isolation, retry on serialization failure, locking, savepoints, recompute protection, cache invalidation across workers |
| `caching.md` | Record cache, prefetching, method-level caches, registry signaling, invalidation |
| `scheduled-jobs.md` | The scheduler: job definitions, intervals, priorities, triggers, locking, failure handling, progress and time budgets |
| `notification-bus.md` | Channels, long polling, presence, cross-worker fan-out |
| `mail-gateway.md` | Outgoing mail queue, mail servers, bounce handling, incoming mail routing to threads, aliases |
| `attachments-and-file-store.md` | Attachments, checksums, file store layout, garbage collection, access control, image processing |
| `report-rendering.md` | Template rendering, paper formats, headers and footers, Portable Document Format generation, barcodes and quick response codes |
| `translation.md` | Translatable fields, term extraction, language activation, per-record translations of rich text |
| `background-workers.md` | Long-running work: queue semantics, retry, idempotency, digest and data recycling jobs |
