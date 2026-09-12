# Business rules

The complete rule catalogue of the Payment Providers domain: validations, guards, permissions, consistency rules, rounding and date rules, with the exact message shown to the user or written to the log. Rules are numbered `PAY-RULE-nnn` to let other documents cite them. A rule marked **industry-standard completion** is not observable in the reference behaviour and is stated here to give a replacement a complete specification.

---

## 1. Provider configuration

**PAY-RULE-001. Operating state.** A Payment Provider is in exactly one of three states: `disabled`, `enabled`, `test`. A provider in `disabled` is never offered to a customer and no request may be sent to it. A provider in `test` talks to the provider's sandbox service; every transaction it creates carries `is_live` false.

**PAY-RULE-002. Publishing follows the state.** When the user changes `state` in the form, `is_published` becomes true if and only if the new state is `enabled`. The rule is a form assistance, not a stored constraint: `is_published` may afterwards be changed independently.

**PAY-RULE-003. A disabled provider may not be published.** Toggling the published flag is refused when `state` is `disabled` and the provider is currently unpublished.
Message: `You cannot publish a disabled provider.`

**PAY-RULE-004. The company of a provider with transactions is frozen.** Changing `company_id` is refused as soon as at least one Payment Transaction references the provider.
Message: `You cannot change the company of a payment provider with existing transactions.`

**PAY-RULE-005. A custom mode belongs to a custom provider.** Database-level check: `custom_mode IS NULL OR (code = "custom" AND custom_mode IS NOT NULL)`.
Message: `Only custom providers should have a custom mode.`

**PAY-RULE-006. Manual capture needs compatible payment methods.** Switching `capture_manually` on is refused when at least one of the provider's active payment methods has `support_manual_capture` equal to `none`.
Message: `The following payment methods must be disabled in order to enable manual capture: %s` where `%s` is the comma-separated list of the offending method names.

**PAY-RULE-007. A method that cannot be captured manually may not be attached to a provider that captures manually.** A payment method is rejected when it is active, its own `support_manual_capture` (or that of its primary method when it is a brand) equals `none`, and at least one of its providers has `capture_manually` true.
Message: `The following payment methods cannot be enabled because their payment provider has manual capture activated: %s`.

**PAY-RULE-008. A payment method needs an enabled provider before it can be activated.** Setting `active` to true is refused when the primary method is currently inactive and every provider that supports it is disabled.
Message: `This payment method needs a partner in crime; you should enable a payment provider supporting this method first.`

**PAY-RULE-009. The placeholder payment method may not be deleted.** The shipped payment method whose code is `unknown` may never be deleted, except while a package is being uninstalled.
Message: `You cannot delete the default payment method.`

**PAY-RULE-010. Shipped providers may not be deleted.** Deleting a provider that carries an external identifier which does not begin with the export prefix is refused, except while a package is being uninstalled.
Message: `You cannot delete the payment provider %s; disable it or uninstall it instead.` where `%s` is the provider name.

**PAY-RULE-011. Credentials required once the provider is in service.** A credential field declared as required for a provider code must be filled on every provider whose `code` equals that code and whose `state` is `enabled` or `test`. The check runs after every creation and every write.
Message: `The following fields must be filled: %s` where `%s` is the comma-separated list of the human-readable labels of the empty fields.

**PAY-RULE-012. Per-connector currency restrictions.** Several connectors constrain `available_currency_ids` beyond the generic mechanism:

| Provider code | Condition | Message |
|---|---|---|
| `asiapay` | More than one currency selected while the state is not `disabled` | `Only one currency can be selected by AsiaPay account.` |
| `asiapay` | A selected currency is outside the provider's currency code table | `AsiaPay does not support the following currencies: %(currencies)s.` |
| `authorize` | More than one currency selected while the state is not `disabled` | `Only one currency can be selected by Authorize.Net account.` |
| `ecpay` | A selected currency is not the new Taiwan dollar | `ECPay only supports TWD.` |
| `mercado_pago` | The account country is set and the selection is not exactly the single currency of that country | `Only the currency %s is available for this account.` |
| `paymob` | More than one currency selected | `Only one currency can be selected per Paymob account.` |
| `paymob` | The selected currency is not one of the four supported currencies | `Only currencies supported by Paymob can be selected.` |
| `toss_payments` | The selection is not exactly the South Korean won | `Currencies other than KRW are not supported.` |

**PAY-RULE-013. A demo provider is never enabled.** A provider whose code is `demo` may only be in `test` or `disabled`.
Message: `Demo providers should never be enabled.`

**PAY-RULE-014. Mercado Pago must be connected before it is put in service.** A provider whose code is `mercado_pago` and whose state is not `disabled` must have an access token.
Message: `Mercado Pago credentials are missing. Click the "Connect" button to set up your account.`

**PAY-RULE-015. Mercado Pago must be connected before tokenization is allowed.** `allow_tokenization` may not be true on a Mercado Pago provider that has no public key.
Message: `Connect your account before enabling tokenization.`

**PAY-RULE-016. PayU must be connected before it is put in service.** A provider whose code is `payu` and whose state is not `disabled` must have both a key identifier and a merchant salt.
Message: `PayU credentials are missing. Click the "Connect" button to set up your account.`

**PAY-RULE-017. Razorpay must be connected before it is put in service.** A provider whose code is `razorpay` and whose state is not `disabled` must have either a connected-account identifier, or both a key identifier and a key secret.
Message: `Razorpay credentials are missing. Click the "Connect" button to set up your account.`

**PAY-RULE-018. A connected Stripe account is never in test mode.** A Stripe provider linked to a connected account may not be set to `test`.
Message: `You cannot set the provider to Test Mode while it is linked with your Stripe account.`

**PAY-RULE-019. A Stripe onboarding must be finished before the provider is enabled.** A Stripe provider whose onboarding is still running may not be set to `enabled`.
Message: `You cannot set the provider state to Enabled until your onboarding to Stripe is completed.`

**PAY-RULE-020. A journal used by a live provider may not be deleted.** Deleting a journal is refused, except while a package is being uninstalled, when at least one provider whose state is not `disabled` uses it.
Message: `You must first deactivate a payment provider before deleting its journal.` followed by a new line, `Linked providers: ` and the comma-separated provider names.

**PAY-RULE-021. A payment method line of a live provider may not be deleted.** Deleting an accounting payment method line is refused, except while a package is being uninstalled, when its provider's state is `enabled` or `test`.
Message: `You can't delete a payment method that is linked to a provider in the enabled or test state.` followed by a new line, `Linked providers(s): ` and the comma-separated provider display names.

**PAY-RULE-022. A connector package with payments may not be uninstalled.** Uninstalling a connector package is refused when at least one Payment uses the accounting payment method of that provider code.
Message: `You cannot uninstall this module as payments using this payment method already exist.`

**PAY-RULE-023. Merchant details may only be fetched from a live provider.** The Authorize connector's Update Merchant Details operation is refused while the provider is disabled.
Message: `This action cannot be performed while the provider is disabled.` Failure messages of the two calls: `Failed to authenticate.` followed by the provider's message, and `Could not fetch merchant details:` followed by the provider's message.

---

## 2. Payment tokens

**PAY-RULE-024. A token never belongs to the public contact.** Evaluated whenever `partner_id` changes.
Message: `No token can be assigned to the public partner.`

**PAY-RULE-025. Unarchiving requires a usable provider and method.** Setting `active` to true is refused when at least one token of the set has an inactive payment method or a provider whose state is `disabled`.
Message: `You can't unarchive tokens linked to inactive payment methods or disabled providers.`

**PAY-RULE-026. A token may only be charged by its own commercial family.** When a transaction is created with the token flow, the commercial contact of the paying contact must equal the commercial contact of the token's owner.
Message: `You do not have access to this payment token.`

**PAY-RULE-027. Token archiving cascades.** Tokens are archived automatically, without any confirmation, when: their provider moves away from `test` or `enabled`; their payment method is archived; their payment method is detached from their provider; or their payment method stops supporting tokenization. The user is warned beforehand in the form with the count of tokens concerned, but the warning does not block the change.

**PAY-RULE-028. A token is archived, never deleted.** No flow of this domain deletes a token. The link from a transaction to its token restricts deletion at the database level.

---

## 3. Payment transactions

**PAY-RULE-029. The reference is unique.** Database-level uniqueness on `reference` across the whole database, every company and every provider together.
Message: `Reference must be unique!`

**PAY-RULE-030. The authorized state requires provider support.** A transaction may not carry the state `authorized` when its provider's `support_manual_capture` is empty.
Message: `Transaction authorization is not supported by the following payment providers: %s` where `%s` lists the distinct provider names.

**PAY-RULE-031. A transaction is never created from an archived token.**
Message: `Creating a transaction from an archived token is forbidden.`

**PAY-RULE-032. No request is sent to a disabled provider.** Every outbound operation (charge a token, capture, void, refund) first checks the provider's state.
Message: `Making a request to the provider is not possible because the provider is disabled.`

**PAY-RULE-033. Only an authorized transaction may be voided.**
Message: `Only authorized transactions can be voided.`

**PAY-RULE-034. Only a confirmed transaction may be refunded.**
Message: `Only confirmed transactions can be refunded.`

**PAY-RULE-035. Capture, void and refund require write access.** Before any of these operations runs with elevated rights, the caller's write access on the selected transactions is checked against the normal access rules and record rules. A user who may not write on a transaction is refused by the access layer.

**PAY-RULE-036. State transitions are guarded.** A target state may only be reached from the source states listed in `workflows.md`, section 7.1. A transaction already in the target state is silently skipped with an informational log entry; a transaction in any other state is skipped with a warning log entry. No error is raised in either case, because the same notification may legitimately arrive twice or out of order.

**PAY-RULE-037. Every state change resets the post-processing flag.** Writing a new state also writes `last_state_change` with the current moment and `is_post_processed` false.

**PAY-RULE-038. The amount of the payment data must be present.** When the connector returns amount data whose amount or currency code is empty, the transaction is set to `error`.
Message: `The amount or currency is missing from the payment data.`

**PAY-RULE-039. The amount of the payment data must match the transaction.** The comparison uses the currency's own comparison which makes rounding differences below the currency precision irrelevant; see `calculations.md`, section "Amount validation".
Message: `The amount from the payment data doesn't match the one from the transaction.`

**PAY-RULE-040. The currency of the payment data must match the transaction.**
Message: `The currency from the payment data doesn't match the one from the transaction.`

**PAY-RULE-041. Validation transactions skip the amount check.** A transaction whose operation is `validation` never has its amount validated, because the provider may charge a different token-verification amount or nothing at all.

**PAY-RULE-042. Connectors may opt out of the amount check.** When the connector returns no amount data at all, the check is skipped. The custom and demo connectors always opt out. Several connectors opt out situationally: Adyen for redirection and challenge answers and for refusals; Authorize when the provider answered with an error and no transaction detail is available; Nuvei when the customer left the page without any data; Razorpay when the return data carry no amount.

**PAY-RULE-043. A failed amount check blocks the updates.** When the amount check moves a transaction into `error` and its state was different before, the connector's update step is not run at all: the provider reference, the payment method and the state are left untouched.

**PAY-RULE-044. Tokenization happens only once, only when asked, and only on success.** A token is created only when `tokenize` is true and the state after the update is `authorized` or `done`. Creating the token sets `tokenize` to false, therefore a repeated notification cannot create a second token.

**PAY-RULE-045. Tokenization cannot be forced from the browser.** When the transaction is created, `tokenize` is true only when the provider allows tokenization **and** the payment method supports tokenization **and** either the payment context requires tokenization or the customer asked for it. A client that sets the request flag by itself therefore gains nothing.

**PAY-RULE-046. Post-processing runs at most once per state.** The flag `is_post_processed` gates the step; it is cleared on every state change in order that a later state is processed again.

**PAY-RULE-047. Post-processing is retried for four days.** The scheduled job only considers transactions whose `last_state_change` is at most four days old. Older unprocessed transactions are abandoned by the job and must be processed by hand with the Post-process operation.

**PAY-RULE-048. A source transaction closes only when its children cover it exactly.** See `workflows.md`, section 5.3, and `calculations.md`, section "Source transaction closure". The comparison is an exact equality on the sum rounded to the currency's decimal places.

**PAY-RULE-049. Refund children never close their source.** Because the closure rule only considers children whose operation equals the source's operation, and a refund child carries the operation `refund`, refunds leave the source's state untouched.

**PAY-RULE-050. Child transaction references.** A capture or void child takes the reference prefix `P-` followed by the source reference; a refund child takes `R-` followed by the source reference. The uniqueness algorithm then appends a separator and a sequence number when that prefix already exists.

**PAY-RULE-051. A refund amount is stored negative.** The `amount` of a transaction whose operation is `refund` is always strictly negative. Everywhere the amount is shown to a human or sent to a provider it is negated back to a positive value.

**PAY-RULE-052. A capture or void child keeps the operation of its source.** This is what makes the closure rule work and what makes the accounting reconciliation of a capture child use the invoices of its source transaction.

---

## 4. Capture, void and refund wizards

**PAY-RULE-053. The amount to capture is within bounds.** `0 < amount_to_capture <= available_amount`.
Message: `The amount to capture must be positive and cannot be superior to %s.` where `%s` is the available amount formatted in the currency.

**PAY-RULE-054. A partial capture requires support on both sides.** When `support_partial_capture` is false, the amount to capture must equal the available amount exactly. `support_partial_capture` is true only when, for **every** selected transaction, both the provider's `support_manual_capture` and the primary payment method's `support_manual_capture` equal `partial`.
Message: `Some of the transactions you intend to capture can only be captured in full. Handle the transactions individually to capture a partial amount.`

**PAY-RULE-055. The void-the-rest option only exists when there is a rest.** `void_remaining_amount` is forced to false whenever `amount_to_capture` is not lower than `available_amount`.

**PAY-RULE-056. The amount to refund is within bounds.** `0 < amount_to_refund <= amount_available_for_refund`.
Message: `The amount to be refunded must be positive and cannot be superior to %s.` where `%s` is `amount_available_for_refund` rendered as a plain decimal number, with a full stop as the decimal separator, no thousands separator, no currency symbol and no padding to the currency's decimal places: an amount of 90 renders as `90.0` and an amount of 1111.11 renders as `1111.11`. This differs deliberately from PAY-RULE-055, whose message formats the amount in the transaction currency (`€1,111.11`); the two messages of this pair are not formatted alike and a replacement must reproduce each as stated, because a user-visible difference in either direction is a difference in behaviour.

**PAY-RULE-057. Refundable amount.** Only refund payments that actually exist reduce the refundable amount; a refund transaction stuck in a transient state does not, in order that a provider failure never blocks a second attempt. See `calculations.md`, section "Refundable amount".

---

## 5. Payment link wizard

**PAY-RULE-058. Nothing to pay.** When `amount_max` is zero or negative the wizard shows `There is nothing to be paid.`

**PAY-RULE-059. Positive amount.** When `amount` is zero or negative the wizard shows `Please set a positive amount.`

**PAY-RULE-060. Amount within the maximum.** When `amount` exceeds `amount_max` the wizard shows `Please set an amount lower than %s.` with the maximum formatted in the currency.

**PAY-RULE-061. Portal payment must be switched on.** When the invoice portal payment setting is off the wizard shows `Online payment option is not enabled in Configuration.`

---

## 6. Portal and request-handling rules

**PAY-RULE-062. A contact supplied in a payment link must be signed.** When the pay route receives a contact identifier, the access token must match the triple (contact identifier, amount, currency identifier). A mismatch answers "not found" rather than "forbidden", which prevents any information about existing identifiers from leaking.

**PAY-RULE-063. The transaction route is signed.** The transaction service verifies the access token against the same triple and answers "forbidden" on a mismatch.

**PAY-RULE-064. The transaction route accepts only known parameters.** Any parameter name outside the allowed list is rejected.
Message: `The following parameters are not whitelisted: %s`, where `%s` is the comma-separated list of the rejected parameter names. (The reference behaviour spells the word "parameters" in this one message with the vocabulary of its own programming language; the message is restated here in the language-neutral wording that a replacement must show.) The base allowed list is: provider identifier, payment method identifier, token identifier, amount, flow, tokenization requested, landing route, validation flag, cross-site request forgery token, and reference prefix. Document routes add their own names: the invoice route adds the next-installment name; the overdue-invoices route adds nothing.
Purpose: a caller must not be able to inject arbitrary values into the create values of a transaction.

**PAY-RULE-065. The currency must exist and be active.** The pay route answers "not found" when the resolved currency does not exist or is archived.

**PAY-RULE-066. The contact must be allowed to pay in the company.** The payment is allowed when the contact has no company or when the contact's company equals the company of the payment. Otherwise the page shows `Please switch to company <company name> to make this payment.` and no payment form at all.

**PAY-RULE-067. A logged-in user always pays as their own contact.** The contact supplied in the link is ignored for a logged-in user, in order that a created token is never assigned to the public user and in order that the record rules apply to the right contact. When the supplied contact differs from the user's contact, the page shows `Warning Make sure you are logged in as the correct partner before making this payment.`

**PAY-RULE-068. An anonymous visitor must resolve to an existing contact.** When the contact identifier of the link does not exist, the visitor is redirected to the login page with the current full path as return address.

**PAY-RULE-069. The confirmation page is signed.** The generic confirmation route answers "not found" when the access token does not match the triple (contact of the transaction, amount of the transaction, currency of the transaction). A missing or malformed transaction identifier redirects to the portal home instead.

**PAY-RULE-070. Inbound notifications are authenticated.** Every notification endpoint verifies the authenticity of the payload before it changes anything, using the connector's mechanism: a computed signature, a shared token in a header, a verification call back to the provider, or, when the provider sends nothing signable, an access token that the system itself embedded in the return address. A missing credential and an invalid credential both answer "forbidden" and both write a warning to the log: `Received payment data with missing signature.` and `Received payment data with invalid signature.`

**PAY-RULE-071. The reference in authoritative data is re-checked.** Connectors that fetch the authoritative payment data after a return or a notification compare the reference in that data with the reference of the matched transaction and answer "forbidden" when they differ. Paymob additionally compares the order identifier of the data with the transaction's provider reference.

**PAY-RULE-072. Requests that may be retried carry an idempotency key.** The key is the hexadecimal secure hash of the concatenation of the database identity, the transaction reference and an optional scope, which makes the same logical request always produce the same key while a different endpoint or a different database produces a different one.

**PAY-RULE-073. Sensitive payload keys are never written to the log.** A logging filter replaces the value of every key declared sensitive by the literal `[REDACTED]`, in dictionaries, in nested collections and inside serialised text. The base set of sensitive keys is empty; the Stripe connector adds the client secret of an intent, and the Toss Payments connector adds the payment secret.

**PAY-RULE-074. Credential visibility.** Every credential field marked secret in `entities.md`, section 1.4, is readable and writable only by the Administrator access group. A billing user who may read a provider sees the non-secret credentials only.

**PAY-RULE-075. Redirect endpoints do not create a session.** Endpoints that receive the customer's browser through a cross-site form post are declared not to save the session, in order that a browser which omits the session cookie on that request does not receive a new, empty session and lose the monitored transaction.

---

## 7. Availability rules

These rules are applied in order; each one narrows the set produced by the previous one, and each removal is recorded in the availability report with its reason.

**PAY-RULE-076. A provider must not be disabled.** Base search: providers of the paying company or of one of its ancestors whose `state` is `enabled` or `test`.

**PAY-RULE-077. An unpublished provider is reserved to internal users.** For a user who is not an internal user, providers whose `is_published` is false are removed. No reason is recorded for this removal, because the report is only shown to administrators, who are internal users and therefore never see providers removed by this rule.

**PAY-RULE-078. The contact's country must be allowed.** When the contact has a country, providers whose `available_country_ids` is not empty and does not contain it are removed. Reason: `incompatible country`.

**PAY-RULE-079. The amount must not exceed the maximum.** Skipped for a validation operation and when no currency is known. The amount is converted from the payment currency into the company's currency at today's rate; a provider is removed when its `maximum_amount` is set and is strictly lower than the converted amount. Reason: `maximum amount exceeded`.

**PAY-RULE-080. The currency must be allowed.** When a currency is known, providers whose `available_currency_ids` is not empty and does not contain it are removed. Reason: `incompatible currency`.

**PAY-RULE-081. Tokenization must be allowed when it is required.** When tokenization is forced by the caller or required by the payment context, providers whose `allow_tokenization` is false are removed. Reason: `tokenization not supported`.

**PAY-RULE-082. Express checkout must be allowed when it is used.** For an express checkout, providers whose `allow_express_checkout` is false are removed. Reason: `express checkout not supported`.

**PAY-RULE-083. The website must match.** When a website is known, providers whose `website_id` is set and different are removed. Reason: `incompatible website`.

**PAY-RULE-084. Cash on delivery must be allowed by the delivery method.** When the sales order's delivery method does not allow cash on delivery, providers whose `custom_mode` is `cash_on_delivery` are removed. Reason: `cash on delivery not allowed by selected delivery method`.

**PAY-RULE-085. Some providers cannot validate a payment method.** For a validation operation, the Flutterwave and Mercado Pago providers are removed. Reason: `tokenization without payment no supported`.

**PAY-RULE-086. Only primary payment methods are offered.** A brand is never selectable on the payment form; it only contributes its logo next to its primary method.

**PAY-RULE-087. A payment method needs at least one compatible provider.** Methods that none of the compatible providers supports are removed. Reason: `no supported provider available`.

**PAY-RULE-088. The contact's country must be supported by the method.** When the contact has a country, methods whose `supported_country_ids` is not empty and does not contain it are removed. Reason: `incompatible country`.

**PAY-RULE-089. The currency must be supported by the method.** When a currency is known, methods whose `supported_currency_ids` is not empty and does not contain it are removed. Reason: `incompatible currency`.

**PAY-RULE-090. The method must support tokenization when it is forced.** Reason: `tokenization not supported`.

**PAY-RULE-091. The method must support express checkout when it is used.** Reason: `express checkout not supported`.

**PAY-RULE-092. Availability of tokens.** For a payment, the tokens offered are those of the paying contact whose provider is one of the compatible providers. For a validation operation, the tokens offered are every token of the paying contact **and** of its commercial contact, whatever the state of their provider, in order that a customer can always delete an obsolete token.

---

## 8. Accounting consistency rules

**PAY-RULE-093. The payment journal is a bank journal.** Only journals of type bank may be selected as the provider's journal, and the journal must belong to the provider's company or to one of its ancestors.

**PAY-RULE-094. A Payment is created at most once per transaction.** Post-processing creates a Payment only when the transaction has none yet.

**PAY-RULE-095. A split source transaction produces no Payment of its own.** When at least one child of a confirmed transaction is in state `done` or `cancel`, the source transaction does not create a Payment; its capture children do.

**PAY-RULE-096. A validation transaction produces no Payment.** There is nothing to reconcile: the amount is either zero or immediately voided or refunded by the connector, and it never appears in a payout.

**PAY-RULE-097. Reconciliation target of a child transaction.** When the operation of a child transaction equals the operation of its source, the Payment is reconciled against the invoices of the **source** transaction; otherwise it is reconciled against the invoices of the transaction itself. This makes a capture child settle the invoice of the original payment, and a refund child settle nothing automatically.

**PAY-RULE-098. A Payment produced by a transaction is reconciled in full.** A bank transaction may not be partially reconciled against a journal item whose entry carries a Payment linked to a Payment Transaction.

**PAY-RULE-099. Online payment eligibility of an invoice.** An invoice may be paid online when all of these hold: the portal payment setting is on; the invoice is posted; its payment state is `not_paid`, `in_payment` or `partial`; its residual amount is not zero; its total is not zero; its type is a customer invoice; and it carries no transaction in state `pending` or `authorized` from a provider whose code is neither `none` nor `custom`. The refusal reasons are concatenated one per line from: `This invoice cannot be paid online.`, `There is no amount to be paid.`, `This invoice isn't posted.`, `This invoice has already been paid.`, `This is not an outgoing invoice.`, `There are pending transactions for this invoice.`

**PAY-RULE-100. Tokens offered for an offline payment exclude manual-capture providers.** Only tokens whose provider has `capture_manually` false are offered on a Payment or on the payment registration wizard, because an offline charge must settle immediately.

**PAY-RULE-101. One transaction per payment.** Creating a transaction from a Payment is refused when the Payment already has one.
Message: `A payment transaction with reference %s already exists.`

**PAY-RULE-102. A token is required to charge a payment.** Creating a transaction from a Payment is refused when the Payment has no token.
Message: `A token is required to create a new payment transaction.`

**PAY-RULE-103. A point of sale payment is never negative.**
Message: `The payment transaction (%d) has a negative amount.` Further failures: `The point of sale online payment (transaction %d) could not be saved correctly` when the Payment could not be created, and `The point of sale online payment (transaction %d) could not be saved correctly because the online payment method could not be found` when the point of sale configuration has no online payment method.

---

## 9. Company, currency and access scoping

**PAY-RULE-104. Provider scoping.** A user may read a Payment Provider whose company is the active company or an ancestor of it: `company parent_of user_enabled_companies`.

**PAY-RULE-105. Transaction scoping.** A user may read a Payment Transaction whose company is among the companies currently enabled for that user: `company IN user_enabled_companies`. Unlike providers and tokens, ancestors do not grant access.

**PAY-RULE-106. Token scoping.** Two rules combine. A public, portal or internal user may only see tokens whose `partner_id` equals their own contact. Every user may only see tokens whose company is the active company or an ancestor. Billing users are granted an additional rule whose domain is always true, which lifts the contact restriction for them.

**PAY-RULE-107. Capture wizard scoping.** A capture wizard is only visible to the user who created it.

**PAY-RULE-108. Company consistency.** A provider, its journal and its website must belong to the same company or to an ancestor of it. A token's company always mirrors its provider's. A transaction's company always mirrors its provider's.

**PAY-RULE-109. Model access.** The access matrix is in `configuration.md`, section "Access groups". In summary: providers and transactions are writable only by the Administrator group, except that billing users may read, write and create transactions; payment methods and tokens are readable by public, portal and internal users and writable only by administrators; the link wizard is reserved to billing users; the capture wizard is open to every internal user.

---

## 10. Rounding, dates and text rules

**PAY-RULE-110. Monetary rounding.** Every monetary amount is rounded to the number of decimal places of its own currency, using the currency's rounding step. Comparisons between two amounts of the same currency use the currency's comparison, which treats a difference smaller than half the rounding step as equality.

**PAY-RULE-111. Minor units use the payment industry precision, not the accounting precision.** When an amount is converted to the minor units a provider expects, the number of decimal places is taken from the payment currency table (the international currency code standard, with the deviations each provider declares), not from the currency record's own accounting precision. The conversion always rounds **down**. See `calculations.md`, section "Minor unit conversion".

**PAY-RULE-112. The amount check uses the payment precision.** The transaction amount is rounded **down** to the precision returned by the connector, or to the payment currency precision when the connector returns none, before it is compared with the amount reported by the provider.

**PAY-RULE-113. The amount written at creation is the amount sent.** After a transaction is created, its cached amount is discarded which makes the value read back exactly the value stored, and therefore exactly the value that will be rendered as text in a request. Without this, an amount such as 1111.11 could be sent as 1111.1100000000001.

**PAY-RULE-114. `last_state_change` is the clock of the state machine.** It is written on creation and on every accepted state change, never otherwise. The post-processing retry window is measured from it.

**PAY-RULE-115. Reference characters.** The generic reference algorithm first transliterates the prefix to plain unmarked letters, dropping every character that has no plain equivalent. Connectors then impose their own restrictions on length and character set; see `calculations.md`, section "Reference generation", and the per-connector entries.

**PAY-RULE-116. Address snapshot.** The nine contact fields of a transaction are copied once, at creation, and never refreshed. The street is the two street lines joined by a single space and trimmed; the email address is the first address obtained by normalising the contact's email field; the name falls back to the parent contact's name when the contact has none.

**PAY-RULE-117. Amount presentation.** Every amount shown in a logged message is formatted with the transaction's currency; the amount of a refund is negated first in order that the customer reads a positive number.

**PAY-RULE-118. Industry-standard completion: no silent overpayment.** A replacement must refuse to create a transaction whose amount is not strictly positive, except for a validation operation, whose amount may be zero. The reference behaviour relies on the calling flow to supply a sensible amount; stating the rule here removes the ambiguity.

**PAY-RULE-119. Industry-standard completion: currency of a child transaction.** A capture, void or refund child always carries the currency of its source transaction and may never be expressed in another currency, because the provider settles the original authorization.

**PAY-RULE-120. Industry-standard completion: ordering of notifications.** A replacement must tolerate notifications arriving out of order and more than once. The guard of PAY-RULE-036 already provides idempotency for repeated notifications; for out-of-order notifications the rule is that a state which is not reachable from the current state is ignored with a warning, never applied and never raised as an error.

---

## 11. Donations and website-scoped payment

**PAY-RULE-121. A donation transaction is flagged.** A transaction created through the donation endpoint carries the donation flag, which is mirrored on the Payment created from it and which is what makes the two donation messages be sent.

**PAY-RULE-122. A donation has a floor.** The donation endpoint refuses an amount lower than the minimum amount carried in the last part of its path.
Message: `Donation amount must be at least %.2f.`, where the placeholder is the minimum amount rendered with exactly two decimals.

**PAY-RULE-123. An anonymous donor identifies himself.** When the donor is not signed in, or supplies no contact, the three donor details are required and are checked in this order: name, then electronic mail address, then country.
Messages: `Name is required.`, `Email is required.`, `Country is required.`

**PAY-RULE-124. An anonymous donation is never tokenized.** When the donor is not signed in, the transaction is created with the website's public contact and with the tokenize flag forced off, and the "save my payment details" box is hidden for every provider on the donation form. This is what keeps the guard of PAY-RULE-024 from ever being reached by a donation.

**PAY-RULE-125. The donor details overwrite the contact snapshot.** For an anonymous donor, the name, the electronic mail address and the country of the transaction snapshot are replaced by the supplied donor details and the language of the transaction is set to the language of the request. For a signed-in donor whose contact carries no country, only the country is filled from the donor details.

**PAY-RULE-126. The access token of a donation is recomputed after the amount is known.** Because the donor may change the amount on the payment page, the token that protects the landing address is computed again over the contact, the amount actually stored on the transaction and the currency, and the landing route is updated with it.

**PAY-RULE-127. The internal donation notification is sent before the payment.** It is sent at the moment the transaction is created, not on confirmation, so that the recipient learns of an attempt even when the payment later fails. The donor's confirmation is sent only when the transaction reaches `done`.

**PAY-RULE-128. The website block shows the site's own offer.** The supported payment methods block computes its list with the company of the website and as the public user of the website, not with the company or the identity of the visitor, so that an editor sees exactly what a visitor will see. It shows the brands of every primary method that is active and has a compatible provider, plus every primary method that has no brand and has a compatible provider.

**PAY-RULE-129. Duplicating a website-restricted provider.** The website of a provider is not copied by the ordinary duplication; the Website Payment package copies it explicitly, and only when the company of the copy is the company of the source or one of its ancestors, so that a duplicate never becomes inconsistent with its own company. The availability consequence of the website field itself is PAY-RULE-083.

**PAY-RULE-130. On a website, the base address of a request wins.** When a request is being served by a website, every return address and every webhook address built for a provider uses the root address of that request rather than the database-wide base address, so that a database serving several sites sends the customer back to the site they came from. Addresses written in a non-Latin script are converted to their plain-letter transcription before they are given to a provider.

---

## 12. Index of rule identifiers

| Identifier | Short title | Section |
|---|---|---|
| PAY-RULE-001 | Operating state | 1. Provider configuration |
| PAY-RULE-002 | Publishing follows the state | 1. Provider configuration |
| PAY-RULE-003 | A disabled provider may not be published | 1. Provider configuration |
| PAY-RULE-004 | The company of a provider with transactions is frozen | 1. Provider configuration |
| PAY-RULE-005 | A custom mode belongs to a custom provider | 1. Provider configuration |
| PAY-RULE-006 | Manual capture needs compatible payment methods | 1. Provider configuration |
| PAY-RULE-007 | A method that cannot be captured manually may not be attached to a provider that captures manually | 1. Provider configuration |
| PAY-RULE-008 | A payment method needs an enabled provider before it can be activated | 1. Provider configuration |
| PAY-RULE-009 | The placeholder payment method may not be deleted | 1. Provider configuration |
| PAY-RULE-010 | Shipped providers may not be deleted | 1. Provider configuration |
| PAY-RULE-011 | Credentials required once the provider is in service | 1. Provider configuration |
| PAY-RULE-012 | Per-connector currency restrictions | 1. Provider configuration |
| PAY-RULE-013 | A demo provider is never enabled | 1. Provider configuration |
| PAY-RULE-014 | Mercado Pago must be connected before it is put in service | 1. Provider configuration |
| PAY-RULE-015 | Mercado Pago must be connected before tokenization is allowed | 1. Provider configuration |
| PAY-RULE-016 | PayU must be connected before it is put in service | 1. Provider configuration |
| PAY-RULE-017 | Razorpay must be connected before it is put in service | 1. Provider configuration |
| PAY-RULE-018 | A connected Stripe account is never in test mode | 1. Provider configuration |
| PAY-RULE-019 | A Stripe onboarding must be finished before the provider is enabled | 1. Provider configuration |
| PAY-RULE-020 | A journal used by a live provider may not be deleted | 1. Provider configuration |
| PAY-RULE-021 | A payment method line of a live provider may not be deleted | 1. Provider configuration |
| PAY-RULE-022 | A connector package with payments may not be uninstalled | 1. Provider configuration |
| PAY-RULE-023 | Merchant details may only be fetched from a live provider | 1. Provider configuration |
| PAY-RULE-024 | A token never belongs to the public contact | 2. Payment tokens |
| PAY-RULE-025 | Unarchiving requires a usable provider and method | 2. Payment tokens |
| PAY-RULE-026 | A token may only be charged by its own commercial family | 2. Payment tokens |
| PAY-RULE-027 | Token archiving cascades | 2. Payment tokens |
| PAY-RULE-028 | A token is archived, never deleted | 2. Payment tokens |
| PAY-RULE-029 | The reference is unique | 3. Payment transactions |
| PAY-RULE-030 | The authorized state requires provider support | 3. Payment transactions |
| PAY-RULE-031 | A transaction is never created from an archived token | 3. Payment transactions |
| PAY-RULE-032 | No request is sent to a disabled provider | 3. Payment transactions |
| PAY-RULE-033 | Only an authorized transaction may be voided | 3. Payment transactions |
| PAY-RULE-034 | Only a confirmed transaction may be refunded | 3. Payment transactions |
| PAY-RULE-035 | Capture, void and refund require write access | 3. Payment transactions |
| PAY-RULE-036 | State transitions are guarded | 3. Payment transactions |
| PAY-RULE-037 | Every state change resets the post-processing flag | 3. Payment transactions |
| PAY-RULE-038 | The amount of the payment data must be present | 3. Payment transactions |
| PAY-RULE-039 | The amount of the payment data must match the transaction | 3. Payment transactions |
| PAY-RULE-040 | The currency of the payment data must match the transaction | 3. Payment transactions |
| PAY-RULE-041 | Validation transactions skip the amount check | 3. Payment transactions |
| PAY-RULE-042 | Connectors may opt out of the amount check | 3. Payment transactions |
| PAY-RULE-043 | A failed amount check blocks the updates | 3. Payment transactions |
| PAY-RULE-044 | Tokenization happens only once, only when asked, and only on success | 3. Payment transactions |
| PAY-RULE-045 | Tokenization cannot be forced from the browser | 3. Payment transactions |
| PAY-RULE-046 | Post-processing runs at most once per state | 3. Payment transactions |
| PAY-RULE-047 | Post-processing is retried for four days | 3. Payment transactions |
| PAY-RULE-048 | A source transaction closes only when its children cover it exactly | 3. Payment transactions |
| PAY-RULE-049 | Refund children never close their source | 3. Payment transactions |
| PAY-RULE-050 | Child transaction references | 3. Payment transactions |
| PAY-RULE-051 | A refund amount is stored negative | 3. Payment transactions |
| PAY-RULE-052 | A capture or void child keeps the operation of its source | 3. Payment transactions |
| PAY-RULE-053 | The amount to capture is within bounds | 4. Capture, void and refund wizards |
| PAY-RULE-054 | A partial capture requires support on both sides | 4. Capture, void and refund wizards |
| PAY-RULE-055 | The void-the-rest option only exists when there is a rest | 4. Capture, void and refund wizards |
| PAY-RULE-056 | The amount to refund is within bounds | 4. Capture, void and refund wizards |
| PAY-RULE-057 | Refundable amount | 4. Capture, void and refund wizards |
| PAY-RULE-058 | Nothing to pay | 5. Payment link wizard |
| PAY-RULE-059 | Positive amount | 5. Payment link wizard |
| PAY-RULE-060 | Amount within the maximum | 5. Payment link wizard |
| PAY-RULE-061 | Portal payment must be switched on | 5. Payment link wizard |
| PAY-RULE-062 | A contact supplied in a payment link must be signed | 6. Portal and request-handling rules |
| PAY-RULE-063 | The transaction route is signed | 6. Portal and request-handling rules |
| PAY-RULE-064 | The transaction route accepts only known parameters | 6. Portal and request-handling rules |
| PAY-RULE-065 | The currency must exist and be active | 6. Portal and request-handling rules |
| PAY-RULE-066 | The contact must be allowed to pay in the company | 6. Portal and request-handling rules |
| PAY-RULE-067 | A logged-in user always pays as their own contact | 6. Portal and request-handling rules |
| PAY-RULE-068 | An anonymous visitor must resolve to an existing contact | 6. Portal and request-handling rules |
| PAY-RULE-069 | The confirmation page is signed | 6. Portal and request-handling rules |
| PAY-RULE-070 | Inbound notifications are authenticated | 6. Portal and request-handling rules |
| PAY-RULE-071 | The reference in authoritative data is re-checked | 6. Portal and request-handling rules |
| PAY-RULE-072 | Requests that may be retried carry an idempotency key | 6. Portal and request-handling rules |
| PAY-RULE-073 | Sensitive payload keys are never written to the log | 6. Portal and request-handling rules |
| PAY-RULE-074 | Credential visibility | 6. Portal and request-handling rules |
| PAY-RULE-075 | Redirect endpoints do not create a session | 6. Portal and request-handling rules |
| PAY-RULE-076 | A provider must not be disabled | 7. Availability rules |
| PAY-RULE-077 | An unpublished provider is reserved to internal users | 7. Availability rules |
| PAY-RULE-078 | The contact's country must be allowed | 7. Availability rules |
| PAY-RULE-079 | The amount must not exceed the maximum | 7. Availability rules |
| PAY-RULE-080 | The currency must be allowed | 7. Availability rules |
| PAY-RULE-081 | Tokenization must be allowed when it is required | 7. Availability rules |
| PAY-RULE-082 | Express checkout must be allowed when it is used | 7. Availability rules |
| PAY-RULE-083 | The website must match | 7. Availability rules |
| PAY-RULE-084 | Cash on delivery must be allowed by the delivery method | 7. Availability rules |
| PAY-RULE-085 | Some providers cannot validate a payment method | 7. Availability rules |
| PAY-RULE-086 | Only primary payment methods are offered | 7. Availability rules |
| PAY-RULE-087 | A payment method needs at least one compatible provider | 7. Availability rules |
| PAY-RULE-088 | The contact's country must be supported by the method | 7. Availability rules |
| PAY-RULE-089 | The currency must be supported by the method | 7. Availability rules |
| PAY-RULE-090 | The method must support tokenization when it is forced | 7. Availability rules |
| PAY-RULE-091 | The method must support express checkout when it is used | 7. Availability rules |
| PAY-RULE-092 | Availability of tokens | 7. Availability rules |
| PAY-RULE-093 | The payment journal is a bank journal | 8. Accounting consistency rules |
| PAY-RULE-094 | A Payment is created at most once per transaction | 8. Accounting consistency rules |
| PAY-RULE-095 | A split source transaction produces no Payment of its own | 8. Accounting consistency rules |
| PAY-RULE-096 | A validation transaction produces no Payment | 8. Accounting consistency rules |
| PAY-RULE-097 | Reconciliation target of a child transaction | 8. Accounting consistency rules |
| PAY-RULE-098 | A Payment produced by a transaction is reconciled in full | 8. Accounting consistency rules |
| PAY-RULE-099 | Online payment eligibility of an invoice | 8. Accounting consistency rules |
| PAY-RULE-100 | Tokens offered for an offline payment exclude manual-capture providers | 8. Accounting consistency rules |
| PAY-RULE-101 | One transaction per payment | 8. Accounting consistency rules |
| PAY-RULE-102 | A token is required to charge a payment | 8. Accounting consistency rules |
| PAY-RULE-103 | A point of sale payment is never negative | 8. Accounting consistency rules |
| PAY-RULE-104 | Provider scoping | 9. Company, currency and access scoping |
| PAY-RULE-105 | Transaction scoping | 9. Company, currency and access scoping |
| PAY-RULE-106 | Token scoping | 9. Company, currency and access scoping |
| PAY-RULE-107 | Capture wizard scoping | 9. Company, currency and access scoping |
| PAY-RULE-108 | Company consistency | 9. Company, currency and access scoping |
| PAY-RULE-109 | Model access | 9. Company, currency and access scoping |
| PAY-RULE-110 | Monetary rounding | 10. Rounding, dates and text rules |
| PAY-RULE-111 | Minor units use the payment industry precision, not the accounting precision | 10. Rounding, dates and text rules |
| PAY-RULE-112 | The amount check uses the payment precision | 10. Rounding, dates and text rules |
| PAY-RULE-113 | The amount written at creation is the amount sent | 10. Rounding, dates and text rules |
| PAY-RULE-114 | `last_state_change` is the clock of the state machine | 10. Rounding, dates and text rules |
| PAY-RULE-115 | Reference characters | 10. Rounding, dates and text rules |
| PAY-RULE-116 | Address snapshot | 10. Rounding, dates and text rules |
| PAY-RULE-117 | Amount presentation | 10. Rounding, dates and text rules |
| PAY-RULE-118 | Industry-standard completion: no silent overpayment | 10. Rounding, dates and text rules |
| PAY-RULE-119 | Industry-standard completion: currency of a child transaction | 10. Rounding, dates and text rules |
| PAY-RULE-120 | Industry-standard completion: ordering of notifications | 10. Rounding, dates and text rules |
| PAY-RULE-121 | A donation transaction is flagged | 11. Donations and website-scoped payment |
| PAY-RULE-122 | A donation has a floor | 11. Donations and website-scoped payment |
| PAY-RULE-123 | An anonymous donor identifies himself | 11. Donations and website-scoped payment |
| PAY-RULE-124 | An anonymous donation is never tokenized | 11. Donations and website-scoped payment |
| PAY-RULE-125 | The donor details overwrite the contact snapshot | 11. Donations and website-scoped payment |
| PAY-RULE-126 | The access token of a donation is recomputed after the amount is known | 11. Donations and website-scoped payment |
| PAY-RULE-127 | The internal donation notification is sent before the payment | 11. Donations and website-scoped payment |
| PAY-RULE-128 | The website block shows the site's own offer | 11. Donations and website-scoped payment |
| PAY-RULE-129 | Duplicating a website-restricted provider | 11. Donations and website-scoped payment |
| PAY-RULE-130 | On a website, the base address of a request wins | 11. Donations and website-scoped payment |

130 rules in total.
