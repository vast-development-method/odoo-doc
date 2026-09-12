# Payment Providers

The Payment Providers domain lets a customer pay a document (a customer invoice, a sales order, a point of sale order, a donation, or an arbitrary amount) electronically, and lets the company follow that money from the moment the customer clicks a payment button until the money is recorded in the accounts. It owns the configuration of every payment provider the company has an account with, the catalogue of payment methods and their brands, the saved payment credentials of a customer (payment tokens), and the Payment Transaction entity with its complete state machine, its child transactions for partial captures, voids and refunds, its reference generation, its post-processing queue, and its contract with each provider's online service.

## Business scope

| In scope | Out of scope (owned elsewhere) |
|---|---|
| Payment Provider configuration: state (disabled, enabled, test), credentials, published flag, availability by country, currency and amount, supported payment methods, capture mode, tokenization, express checkout, customer-facing messages | The bank journal, the outstanding receipts account and the reconciliation of the resulting Payment: see [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) |
| Payment Method catalogue, brands as children of a primary method, per-method support flags, supported countries and currencies, activation per provider, method images | The accounting payment method lines attached to a journal: see [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) |
| Payment Token: saving a customer's payment credentials at the provider, reusing them, archiving them, the customer portal page that manages them | Subscription contracts and recurring billing schedules, which are not part of this specification; this domain supplies only the reusable token and the operation that charges it without any customer interaction |
| Payment Transaction: reference generation, amount and currency, contact snapshot, operation kind, state machine (draft, pending, authorized, done, cancel, error), state message, child transactions, landing route, post-processing | The Journal Entry produced when a transaction is confirmed: see [../general-ledger/](../general-ledger/README.md) |
| Manual capture and void of an authorized amount, full and partial, with the capture wizard | Manual bank transfers entered by an accountant: see [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) |
| Refund of a confirmed transaction, full and partial, with the refund wizard | Credit notes and their accounting: see [../accounts-receivable/](../accounts-receivable/README.md) |
| The payment form and the portal payment flow: the pay page, the payment method management page, the transaction creation service, the confirmation page, the status page and its polling service | The customer portal shell, breadcrumbs and access tokens of documents: see [../customer-portal/](../customer-portal/README.md) |
| Provider notification handling: return from checkout, webhook notifications, signature verification, idempotency, transaction matching by reference | Electronic document exchange: see [../electronic-invoicing-and-document-exchange/](../electronic-invoicing-and-document-exchange/README.md) |
| Per-provider connector contracts for every provider package shipped with the platform | Provider-side account opening, pricing and settlement, which happen outside the system |
| Confirmation of a sales order and registration of a payment against customer invoices when a transaction is confirmed | The sales order confirmation rules themselves: see [../sales/](../sales/README.md); the invoice payment state: see [../accounts-receivable/](../accounts-receivable/README.md) |
| Online payment of a point of sale order | Point of sale sessions, orders and closing: see [../point-of-sale/](../point-of-sale/README.md) |

## Capabilities delivered

1. Configure one Payment Provider record per provider account and per company, hold its credentials in fields that only the administrator access group may read, and move it between the disabled, test and enabled states with all the side effects that each move implies.
2. Publish or unpublish a provider in order that it is offered to customers on the public website or reserved to internal users.
3. Restrict a provider to a list of countries, a list of currencies and a maximum amount, and report to an administrator exactly why a provider or a payment method is not offered on a given payment form.
4. Maintain a catalogue of payment methods, group card brands under a primary card method, and activate each method for each provider.
5. Offer the customer a payment form that lists their saved payment methods first and then the compatible payment methods, in both a redirect flow (the customer leaves for the provider's page) and an inline flow (the customer stays on the platform's page).
6. Create a Payment Transaction with a unique reference derived from the document being paid, take a snapshot of the paying contact, and hand the provider everything it needs to process the payment.
7. Receive the provider's answer, either synchronously (the response of a request the system made), on return from the provider's checkout page, or asynchronously (a webhook notification), verify its authenticity, match it to a transaction and move that transaction through its state machine exactly once.
8. Save a customer's payment credentials as a Payment Token after a successful payment or after a dedicated zero-amount or small-amount validation transaction, and charge that token later without any customer interaction.
9. Authorize an amount without charging it, then capture it in full or in parts, and void whatever remains, keeping one child transaction per capture and per void.
10. Refund a confirmed transaction in full or in part, keeping one child transaction per refund, and recognise refunds initiated on the provider's own interface.
11. Post-process every transaction exactly once: confirm the sales order, post and pay the customer invoice, register the point of sale payment, and retry for up to four days through a scheduled job when the customer's browser never came back.
12. Generate a payment link for a document in order that the customer can pay without logging in.

## Actors

| Actor | Role in this domain |
|---|---|
| Customer (public visitor, portal user or internal user acting for themselves) | Chooses a payment method on the payment form, is redirected to the provider or fills an inline form, returns to the confirmation page, saves and deletes their own payment tokens. |
| Billing user (access group "Invoicing") | Reads, creates and updates Payment Transactions, generates payment links, captures, voids and refunds transactions from a customer invoice or a Payment, and reads every Payment Token. |
| Salesperson | Sees the transactions linked to a sales order and generates a payment link for a quotation. |
| System administrator (access group "Administrator") | Creates, configures, enables, disables, publishes and deletes Payment Providers; reads and writes provider credentials; activates payment methods; reads and writes every token and transaction; reads the availability report. |
| Payment provider service | Sends notifications to the platform's return and webhook endpoints, signs them, and answers the platform's requests. |
| Scheduled execution | Runs the post-processing job every ten minutes while at least one provider is not disabled. |

## Entities owned by this domain

| Full name | Transport name | Storage name | Kind | Purpose |
|---|---|---|---|---|
| Payment Provider | `payment.provider` | `payment_provider` | persistent | One account with one payment provider, for one company: credentials, state, availability, supported payment methods, feature support and customer-facing messages. |
| Payment Method | `payment.method` | `payment_method` | persistent | One payment instrument (card, bank transfer, wallet, deferred payment) or one brand of a primary instrument, with its own support flags and its own country and currency restrictions. |
| Payment Token | `payment.token` | `payment_token` | persistent | A reusable reference, held by the provider, to a customer's payment credentials, together with the clear part of those credentials for display. |
| Payment Transaction | `payment.transaction` | `payment_transaction` | persistent | One attempt to move money for one document: amount, currency, contact snapshot, provider, method, token, operation kind and state. |
| Payment Capture Wizard | `payment.capture.wizard` | `payment_capture_wizard` | transient | Working copy used to capture all or part of one or several authorized amounts and optionally void the rest. |
| Payment Link Wizard | `payment.link.wizard` | `payment_link_wizard` | transient | Working copy used to build a signed payment web address for a document. |

Six entities, and no more, are defined by this domain. One further transient entity, the Payment Refund Wizard, is defined by the accounting payments capability package and is therefore owned by [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md); because every rule it enforces is a rule of the refund contract of a Payment Transaction, its fields and its validation are specified in [entities.md](entities.md) section 7 of this folder. The owning domain specifies what the resulting refund Payment does in the ledger, and must not restate the wizard's fields.

## Entities from other domains that this domain extends

| Entity | Owner domain | What this domain adds |
|---|---|---|
| Request Routing | [../platform-foundation/](../platform-foundation/README.md) | The operation that lists the capability packages whose front-end translations are loaded on public pages. This domain appends its own package to that list, in order that every message of the payment form, the payment status page and the payment method management page reaches a public visitor in their own language. |
| Contact | [../contacts-and-organizations/](../contacts-and-organizations/README.md) | `payment_token_ids` (the tokens of the contact) and `payment_token_count`. |
| Country | [../contacts-and-organizations/](../contacts-and-organizations/README.md) | `is_stripe_supported_country` and `is_mercado_pago_supported_country`, two derived flags used by the guided setup. |
| Company | [../contacts-and-organizations/](../contacts-and-organizations/README.md) | On creation of a company, every installed provider of the current company is duplicated into the new company. |
| Configuration Settings | [../platform-foundation/](../platform-foundation/README.md) | `active_provider_id`, `has_enabled_provider`, `onboarding_payment_module` and the guided setup operation. |
| Journal | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) | A journal that is used by a provider which is not disabled may not be deleted. |
| Payment Method Line | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) | `payment_provider_id` and `payment_provider_state`; a line linked to a provider in the enabled or test state may not be deleted. |
| Payment | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) | `payment_transaction_id`, `payment_token_id`, `amount_available_for_refund`, `suitable_payment_token_ids`, `use_electronic_payment_method`, `source_payment_id`, `refunds_count`, and the ability to post a payment by charging a token. |
| Payment Registration Wizard | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) | `payment_token_id`, `suitable_payment_token_ids`, `use_electronic_payment_method`. |
| Payment Refund Wizard | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) | The whole wizard is driven by this domain's refund contract: `support_refund`, `has_pending_refund`, the refundable amount and the operation that creates the refund transaction. The owning domain specifies the Payment side of the resulting refund; the transaction side is specified in `entities.md` section 7 of this folder, and the two specifications must not be duplicated. |
| Journal Entry (customer invoice) | [../accounts-receivable/](../accounts-receivable/README.md) | `transaction_ids`, `authorized_transaction_ids`, `transaction_count`, `amount_paid`, the online-payment eligibility test, and the capture and void operations reachable from an invoice. |
| Sales Order | [../sales/](../sales/README.md) | `transaction_ids` (through the transaction's `sale_order_ids`), the confirmation of a quotation when a transaction reaches the confirmed state, and automatic invoicing. |
| Point of Sale Order | [../point-of-sale/](../point-of-sale/README.md) | `pos_order_id` on the transaction and the registration of an online point of sale payment. |
| Bank Transaction | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) | A bank transaction may not be partially reconciled against a Payment that came from a Payment Transaction. |

## Cross-domain dependencies

These domains must exist before this one can be implemented:

1. [../contacts-and-organizations/](../contacts-and-organizations/README.md) for Contact, Company, Country and Country State, and for the public contact concept used by the guard that forbids assigning a token to the public contact.
2. [../platform-foundation/](../platform-foundation/README.md) for Currency and its rounding and conversion rules, for the view and template registry that holds the payment form templates, for the module registry that drives the installation state of a provider, for scheduled execution, for access groups and record rules, and for the keyed hash used by access tokens.
3. [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) for Journal, Payment, Payment Method Line and the outstanding accounts; a provider without a journal cannot record money.
4. [../general-ledger/](../general-ledger/README.md) for the Journal Entry that the Payment produces and for reconciliation.
5. [../accounts-receivable/](../accounts-receivable/README.md) for the customer invoice, its payment state and its portal pages.
6. [../sales/](../sales/README.md) for the sales order confirmation and automatic invoicing triggered by a confirmed transaction.
7. [../customer-portal/](../customer-portal/README.md) for the portal shell, its access tokens and its record pages, and [../website-and-storefront/](../website-and-storefront/README.md) for the website scoping of a provider and the donation flow.
8. [../point-of-sale/](../point-of-sale/README.md) for the online payment of a point of sale order.

## Reading order

Read the eleven documents of this folder in this order. Each one assumes the ones before it.

1. This page, for the scope, the actors and the entities the folder owns.
2. [glossary.md](glossary.md), for the vocabulary used everywhere else.
3. [entities.md](entities.md), for every field of every entity.
4. [state-machines.md](state-machines.md), for the eight state fields and everything that moves them.
5. [workflows.md](workflows.md), for the end-to-end procedures that chain those transitions together.
6. [business-rules.md](business-rules.md), for the numbered catalogue of refusals, guards and permissions.
7. [calculations.md](calculations.md), for the arithmetic the workflows rely on.
8. [accounting-effects.md](accounting-effects.md), for the ledger consequences.
9. [configuration.md](configuration.md), for what must be set up before any of this works.
10. [interfaces.md](interfaces.md), for the operations, endpoints and screens.
11. [provider-connector-contracts.md](provider-connector-contracts.md), for the twenty-three connector contracts, read as a reference rather than end to end.
12. [acceptance-criteria.md](acceptance-criteria.md), last, as the test of whether a rebuild is correct.

## Navigation

| File | Contents |
|---|---|
| [entities.md](entities.md) | Every field of Payment Provider, Payment Method, Payment Token, Payment Transaction and the three wizards, plus every field this domain adds elsewhere, with identity, ordering, constraints, on-change behaviour and lifecycle. |
| [state-machines.md](state-machines.md) | The eight state fields of the domain: the status of a Payment Transaction, its post-processing flag, the operating state, publication flag and mirrored installation state of a Payment Provider, the activation of a Payment Token and of a Payment Method, and the simulated status of a demonstration token; each with its states, its transition table, its guards with their exact refusal texts and a diagram. |
| [workflows.md](workflows.md) | Every end-to-end flow: configure and enable a provider, pay from the portal in the redirect, inline and token flows, save a payment method, handle a notification, capture, void, refund, post-process, generate a payment link, and the state machine summary tables. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with exact messages, guards, permissions, company and currency consistency, uniqueness and rounding rules. |
| [calculations.md](calculations.md) | Reference generation, amount conversion to minor units, amount validation, capture and void arithmetic, refundable amount, token display name, availability filters and signature computation, each with worked examples. |
| [accounting-effects.md](accounting-effects.md) | What a transaction causes in the ledger: the Payment created on confirmation, account selection, reconciliation with the invoice, refunds, partial captures and voids, and the boundary with the payments domain. |
| [configuration.md](configuration.md) | Every setting, shipped record, access group, record rule, scheduled job and master-data prerequisite. |
| [interfaces.md](interfaces.md) | Service operations, request endpoints, screens described as workflows on views, notifications and scheduled jobs. |
| [acceptance-criteria.md](acceptance-criteria.md) | Given/When/Then scenarios covering every rule, every state transition and every formula. |
| [provider-connector-contracts.md](provider-connector-contracts.md) | The connector contract of every provider package: credentials, endpoints, outbound request content, inbound notification content, status mapping, amount formatting, restrictions and test mode. |
| [glossary.md](glossary.md) | Domain terms. |
