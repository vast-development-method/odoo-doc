# Glossary

Terms used in the Payment Providers domain, with their meaning in this specification.

**Access token.** A short signed text that proves that a set of values (typically a contact, an amount and a currency) was produced by the system and has not been altered. It is computed as a keyed hash over those values joined by a vertical bar, and verified with a constant-time comparison.

**Acknowledgement body.** The exact answer a webhook endpoint returns in order that the provider considers the notification delivered and stops re-sending it.

**Authorization.** The provider's promise that the amount is available on the customer's instrument, together with a reservation of that amount. No money moves until the authorization is captured. A transaction in the `authorized` state carries an authorization.

**Availability report.** A diagnostic structure filled while the compatible providers and payment methods are computed. It records, for every provider and every method, whether it is available and, when it is not, why. It is shown only to administrators.

**Brand.** A payment method whose `primary_payment_method_id` is filled. A brand is never selectable on the payment form; its only role is to display its logo next to its primary method and to carry its own support flags.

**Capture.** The operation that turns an authorization into an actual charge. A capture creates a child transaction whose reference is prefixed with `P-` and whose operation is copied from its source transaction.

**Child transaction.** A transaction created from another one to capture part of it, void part of it, or refund part of it. Its `source_transaction_id` points at the transaction it came from.

**Commercial contact.** The contact that represents the company a person belongs to. Token ownership and payment reconciliation are checked against the commercial contact, in order that a delivery address may pay with a token saved by the head office.

**Compatible provider, compatible payment method.** A provider or a method that survived every availability filter for the current payment context.

**Connector.** The set of behaviours attached to one provider code: credentials, request building, notification handling, status mapping, amount formatting and restrictions.

**Custom mode.** The kind of offline flow a provider with the code `custom` represents: wire transfer, cash on delivery or pay on site.

**Direct payment.** A payment made without leaving the platform's page. The customer types the payment details into a component supplied by the provider, therefore the details never reach the platform. The transaction's operation is `online_direct`.

**Donation.** A payment made on a public website page through the donation block, with no document behind it. The transaction carries the donation flag, which makes the platform send an internal notification when the donation is created and a confirmation to the donor when it is confirmed.

**Donation block.** The page-editor block that offers a donation: it carries the recipient electronic mail address, the amounts offered, one description per amount, the minimum and maximum amounts, the slider step and the default amount.

**Express checkout.** A payment made through a wallet button that supplies the billing and delivery address itself, letting the customer skip the address steps of the checkout.

**Idempotency key.** A key sent with a request that may be retried, in order that the provider rejects a second, identical request instead of charging the customer twice.

**Inline form.** The fragment rendered next to a payment method on the payment form, which hosts the provider's own browser component.

**Installation state.** The state of the capability package that implements a connector, mirrored read-only on every provider that names that package. It decides whether the provider form offers an Install operation or the full configuration.

**Landing route.** The address the customer is sent to once the payment flow is finished. The generic flow appends the transaction identifier and an access token to it.

**Major unit, minor unit.** The major unit of a currency is the unit a person writes (one euro); the minor unit is the smallest unit a provider counts in (one cent). The number of decimal places used for the conversion is the payment precision, not the accounting precision.

**Manual capture.** The provider configuration in which an online payment is authorized first and captured in a second, deliberate step.

**Monitored transaction.** The transaction whose identifier is kept in the visitor's browsing session in order that the payment status page knows what to show and what to post-process.

**Notification.** Any message the provider sends to the platform: the answer to a request, the data carried on the customer's return from the provider's page, or an asynchronous webhook message.

**Operation.** The kind of a transaction: online payment with redirection, online direct payment, online payment by token, validation of a payment method, offline payment by token, or refund.

**Payment data.** The structure a connector feeds into the processing step. It may come from an answer, from a return address or from a webhook body, and it is always reduced to the same shape before it is processed.

**Payment link.** A signed web address, built by the Payment Link Wizard, that lets a customer pay a document without signing in. It carries the amount, the currency, the contact, the company and an access token.

**Payment method.** One payment instrument offered to the customer. A method is primary, and therefore selectable, or a brand of a primary method.

**Payment precision.** The number of decimal places the payment industry uses for a currency. It is taken from a fixed table, with the deviations each connector declares, and it is not necessarily the accounting precision of the currency record.

**Payment provider.** One account the company holds with one payment provider, in one company of the database.

**Payment token.** A reusable reference, held by the provider, to a customer's payment credentials, together with a short clear fragment used to display it.

**Post-processing.** The step that turns a transaction state into business consequences: confirming a sales order, posting and paying an invoice, registering a point of sale payment. It runs at most once per state.

**Post-processing flag.** The Boolean that records whether the consequences of the current state of a transaction have already been produced. Every accepted state change clears it; producing the consequences sets it.

**Primary payment method.** A payment method whose `primary_payment_method_id` is empty. Only primary methods appear on the payment form.

**Processing values.** The structure handed to the browser when a transaction is created: everything the browser needs to continue the payment, including the rendered redirect form when there is one.

**Provider reference.** The provider's own identifier for a transaction. It is distinct from the provider reference of a token, which identifies the saved credentials.

**Publication.** The flag that decides whether a provider is offered to people who are not internal users. A disabled provider may never be published, and reaching the disabled state unpublishes a provider.

**Redirect flow.** A payment in which the customer leaves the platform for the provider's page and comes back through a return endpoint. The transaction's operation is `online_redirect`.

**Reference.** The platform's own identifier for a transaction. It is unique across the whole database and is usually derived from the document being paid.

**Refund.** The operation that returns money to the customer. A refund creates a child transaction whose reference is prefixed with `R-`, whose amount is negative and whose operation is `refund`.

**Return endpoint.** The address the provider sends the customer's browser back to after the payment.

**Sandbox, test mode.** The provider's parallel environment in which no real money moves. A provider in the `test` state talks to it, and every transaction it creates is marked as not being a production transaction.

**Signature.** The proof that a notification was produced by the provider and not altered. It is computed from the payload and a shared secret, following the connector's own algorithm.

**Simulated status.** The status stored on a demonstration token that decides which state every transaction made with that token reaches. It exists only for the demonstration connector.

**Source transaction.** The transaction a child transaction was created from.

**Split authorization.** An authorization that has been divided between several capture and void children. Its source transaction closes only when the children cover its amount exactly.

**State message.** The free text stored beside the status of a transaction to explain a cancellation, an error or a pending state. It is appended to the message written on the linked documents for the cancelled and error states.

**Supported payment methods block.** The page-editor block that advertises, on a public page, the payment methods the current website can accept.

**Tokenization.** The act of saving a customer's payment credentials at the provider and recording the resulting reference as a Payment Token.

**Validation transaction.** A transaction whose only purpose is to prove that a payment method works and to obtain a token. Its amount is the provider's validation amount, which may be zero, and its amount is never checked against the provider's answer.

**Void.** The operation that releases an authorization without charging it. A void creates a child transaction whose reference is prefixed with `P-` and whose operation is copied from its source transaction; when it is confirmed the child's state is `cancel`.

**Webhook.** The endpoint a provider calls to report an event asynchronously, possibly long after the customer left, and possibly more than once for the same event.

**Website scoping.** The restriction of a provider to one website. A provider whose website is empty is offered on every website; a provider whose website is set is offered only on that one.
