# Runtime

The runtime is everything the system does that is not a business decision: how a network message becomes a call on an endpoint and a response, how a conversation with one user is held across messages, how one unit of work is bounded by a transaction and what happens when two of them collide, what is remembered between requests and how that memory is invalidated across processes, how work happens without a client asking for it, how the server pushes information to a connected client, how binary content is stored and served, how outbound and inbound communication is queued and drained, how a document is rendered and a term is translated, which knobs change behaviour without changing data, and what record the system keeps of what it did.

This folder specifies all of it as observable behaviour, with no provisioning, sizing or hosting guidance. A replacement built from these documents responds the same way, retries the same failures the same number of times, expires the same things after the same delays, and leaves the same trail.

## The documents

| Document | Content |
|---|---|
| [`request-lifecycle.md`](request-lifecycle.md) | The ordered steps from an incoming message to a response: trusted proxy handling, inbound limits, session and database resolution, the three serving branches, registry acquisition, the read-only optimization and its escalation, path resolution, the four authentication levels, pre-dispatch and post-dispatch, language and time zone resolution, context construction, geographic resolution, the error envelopes of each transport family, rate limiting and the profiling hook. |
| [`transactions-and-concurrency.md`](transactions-and-concurrency.md) | The transaction boundary of every entry point, connections and cursors, repeatable-read isolation and the transaction timestamp, the four hook families and the commit and rollback sequences, flush points, savepoints, the two lock strengths and where they are taken, the retry loop with its attempts and randomized backoff, concurrent edits, long-running operations and registry-level concurrency. |
| [`caching.md`](caching.md) | The five cache layers: the record cache and its keying, prefetching, protection, the unit of work, invalidation, the recomputation engine, the on-change protocol, the registry cache containers with their invalidation keys and the complete catalogue of memoized operations, the inter-process signalling protocol, process-level caches, cache statistics and the derived-artifact caches for views, assets, translations and responses. |
| [`scheduled-jobs.md`](scheduled-jobs.md) | The three scheduling entities, the worker loop and its wake-up channel, due-job selection and locking, the two transactions of a job, the inner loop and the progress protocol, the decision table that produces the three outcomes, rescheduling including across a daylight-saving change, failure counting and deactivation, timeouts, the automatic cleanup job with every cleanup it runs, and the complete catalogue of shipped jobs. |
| [`notification-bus.md`](notification-bus.md) | The bus entity and its retention, channel naming and qualification, message shape, the pre-commit and post-commit send protocol with payload splitting, the relay loop, the persistent transport with its handshake, framing, limits, timeouts and close codes, the subscription protocol, the dispatch algorithm with its out-of-order compensation window, the polling fallback, presence, and the delivery guarantees. |
| [`mail-gateway.md`](mail-gateway.md) | The electronic-mail gateway in both directions: outgoing mail servers and the ordered rules that select one, the outgoing queue with its state machine, batching, grouping, failure classification and post-processing, the throttle of an individually owned server, the inbound poll with its two transactions and its deactivation rule, the routing of one inbound message, aliases and their policies, bounce detection and handling, and loop prevention. |
| [`background-workers.md`](background-workers.md) | What every queue shares: the five parts of a queue, where the commits fall, and the four idempotency patterns that make a batch-committing worker safe to re-run. Then each queue the platform ships apart from mail: deferred notifications, user-scheduled messages, text messages, postal letters, browser push notifications, digests, data recycling, time-based automation and marketing campaigns. |
| [`attachments-and-file-store.md`](attachments-and-file-store.md) | The Attachment entity, the two storage strategies and the migration between them, content addressing with collision detection, the write path with media-type detection, plain-text forcing and automatic image reduction, the access rules and tokens, deferred deletion and the file-store collection, image variants, the streaming contract with its validators and cache directives, uploading, attachments that serve a request path and attachments that back asset bundles. |
| [`report-rendering.md`](report-rendering.md) | The document template grammar with its directives, evaluation order and safety rules, the field converters, the Report Action entity, the print pipeline from a request to a paginated document with its splitting and attachment rules, paper formats, document layouts with their branding, headers and footers, barcodes and quick response codes, label sheets, and the two data-export formats. |
| [`translation.md`](translation.md) | The Language entity with its validations and number formatting, the per-language storage of translatable values, the language of a read and the fallback chain, term extraction from markup, the matching of translations to a changing source, behaviour literals, translations of shipped data, catalogues with their export and import, language activation, the external translation service, and how translated text reaches views, documents and messages. |
| [`sessions-and-authentication.md`](sessions-and-authentication.md) | Session identity and storage, the content of a session, persistence rules, soft and hard rotation with the successor chain, the binding token that ties a session to a user, device records and revocation, the four authentication levels in detail, interactive login with its second factor, application keys, the identity-check window and the group-driven lock timeouts, the cross-site request forgery token, expiry and logout. |
| [`logging-and-audit.md`](logging-and-audit.md) | The Log Entry entity and its producers, record stamps, change tracking with its value storage and access filters, login and device history, inalterability hash chains with the exact algorithm and its verification, performance profiles, server action history, and automation rule logging. |
| [`configuration-parameters.md`](configuration-parameters.md) | The System Parameter entity, its read and write semantics, its cache invalidation and its protected keys, the parameters seeded at database creation, and the complete catalogue of reserved platform parameters and of the parameters each capability package adds, with key, type, default and effect. |

## Scope boundary

| In scope | Out of scope |
|---|---|
| The ordered steps of serving a request, and every branch and error path | How the process is started, supervised or scaled |
| The isolation level, the retry count, the backoff distribution, the lock strengths | Which database product to use and how to tune it |
| Cache lifetimes, keys and invalidation signals | Cache sizing for a given workload |
| Scheduled execution semantics: due selection, locking, budgets, rescheduling, deactivation | How many workers to run |
| Queue batching, commit points, retry classification | Which external mail, text message or postal service to contract |
| Storage strategies, content addressing, collection | Where to mount the file store |
| The template grammar and the geometry handed to the conversion engine | Which conversion engine to install |
| Every reserved configuration parameter, its key, type, default and effect | The configuration file format |
| What is written to each audit trail and how long it is kept | Log shipping and retention infrastructure |

## Reading order

1. [`request-lifecycle.md`](request-lifecycle.md), then [`sessions-and-authentication.md`](sessions-and-authentication.md): together they define what happens to one incoming message.
2. [`transactions-and-concurrency.md`](transactions-and-concurrency.md), then [`caching.md`](caching.md): together they define what happens to the data that message touches.
3. [`scheduled-jobs.md`](scheduled-jobs.md), then [`background-workers.md`](background-workers.md) and [`mail-gateway.md`](mail-gateway.md): together they define what happens without a message.
4. [`notification-bus.md`](notification-bus.md) and [`attachments-and-file-store.md`](attachments-and-file-store.md): the two channels through which the server pushes and serves content.
5. [`report-rendering.md`](report-rendering.md) and [`translation.md`](translation.md): how a record becomes text a person can read, in the language they read.
6. [`configuration-parameters.md`](configuration-parameters.md) and [`logging-and-audit.md`](logging-and-audit.md): the knobs and the trail.

## Constants that appear in several documents

| Constant | Value | Where it matters |
|---|---|---|
| Maximum request body | 134 217 728 bytes | Request lifecycle, attachments |
| Session identifier length | 84 characters, of which the stable prefix is 42 | Sessions, cross-site token, devices |
| Session soft rotation age | 10 800 seconds, three hours | Sessions |
| Predecessor session grace | 120 seconds | Sessions, notification bus |
| Default session inactivity limit | 604 800 seconds, seven days | Sessions, devices |
| Identity confirmation grace | 600 seconds, ten minutes | Sessions |
| Transaction retry attempts | 5 | Transactions, every queue |
| Retry backoff | A uniformly random duration between zero and two raised to the attempt number, in seconds | Transactions |
| Isolation level | Repeatable read | Transactions |
| Scheduled job iteration budget | At least 10 iterations, then at least 10 seconds | Scheduled jobs |
| Scheduled job deactivation | 5 consecutive failures **and** 7 days | Scheduled jobs |
| Consecutive timeouts before failure | 3 | Scheduled jobs |
| Worker poll interval | 60 seconds plus a per-worker offset | Scheduled jobs |
| Notification first-poll window | 50 seconds | Notification bus |
| Notification out-of-order window | 10 seconds | Notification bus |
| Notification retention | 86 400 seconds, twenty-four hours | Notification bus |
| Batch deletion limit for cleanups | 100 000 rows | Scheduled jobs, sessions, logging |
| Static content cache | 604 800 seconds, one week | Request lifecycle, attachments |
| Fingerprinted content cache | 31 536 000 seconds, one year, immutable | Caching, attachments, report rendering |
| Device trace granularity | 3 600 seconds, one hour | Sessions, logging |
| Outgoing mail queue batch | 1 000 messages | Mail gateway |
| Inbound mail poll batch | 50 messages per server | Mail gateway |

## Cross-folder dependencies

| Dependency | What is relied upon |
|---|---|
| [`../overview/security-model.md`](../overview/security-model.md) | Access groups, record rules and field permissions, applied by the authentication step and by the attachment access rules. |
| [`../overview/entity-and-field-system.md`](../overview/entity-and-field-system.md) | Field types and the common stamp columns referred to throughout. |
| [`../overview/views-and-actions.md`](../overview/views-and-actions.md) | The declarative layout grammar and the view inheritance that document templates reuse. |
| [`../overview/messaging-model.md`](../overview/messaging-model.md) | Threads, messages, subtypes, followers and notifications, which the mail gateway and the change tracking write into. |
| [`../interfaces/`](../interfaces/) | The transport spellings of paths, headers, media types and envelope members that this folder deliberately does not repeat, and the exhaustive list of endpoints. |
| [`../references/scheduled-jobs.md`](../references/scheduled-jobs.md) | The machine-readable list of shipped jobs with the operation each one invokes. |
| [`../domains/`](../domains/) | The business meaning of every job, queue and parameter listed here; this folder specifies their execution, not their purpose. |
