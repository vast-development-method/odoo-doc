# Workflows

Every operational flow of the Payment Providers domain, end to end: actors, preconditions, numbered steps, branches, the records written at each step with their field values, the messages emitted, and the postconditions. The state machine tables are at the end of the file.

---

# 1. Setting up providers and methods

## 1.1 Install a provider connector package

**Actor**: system administrator.
**Preconditions**: the Payment Engine package is installed, which means one Payment Provider record exists per shipped provider with `code` equal to `none` and `state` equal to `disabled`.

1. The administrator opens the Payment Providers screen and clicks **Install** on the card of the provider whose package is not installed. The card is blue while the package is not installed.
2. The package is installed. Its installation hook runs the provider setup step with the provider's code (and, for a custom provider, its custom mode).
3. The setup step searches for the provider records matching that code, takes the first one as the reference record, and lists the companies of the database that have no provider with that code and that are not branches of another company.
4. For each such company, the reference record is copied with `company_id` set to that company. The copy has no credentials, `state` equal to `disabled`, `is_published` false, and no journal.
5. The Accounting Payments package extends the setup step: when the code is neither `none` nor `custom` and no accounting payment method exists with that code, it creates one with the provider's display label as name, the code, and the inbound direction.
6. The screen reloads.

**Postconditions**: every company of the database owns one disabled provider record for that provider; an accounting payment method exists for the provider's code.

## 1.2 Configure and enable a provider

**Actor**: system administrator.
**Preconditions**: the connector package is installed; the administrator holds an account with the provider and its credentials.

1. Open the provider form. The **Credentials** page shows only the fields of this provider's code; it is hidden entirely when the code is `none`.
2. Fill every credential field. Fields marked secret are only visible to the Administrator access group.
3. On the **Configuration** page, optionally:
   - restrict the offer with `maximum_amount`, `available_currency_ids` and `available_country_ids`;
   - switch on `allow_tokenization` (only shown when `support_tokenization` is true);
   - switch on `capture_manually` (only shown when `support_manual_capture` is set);
   - switch on `allow_express_checkout` (only shown when `support_express_checkout` is true).
4. On the **Messages** page, optionally replace the four state messages and the help message. The authorization message is only shown when the provider supports manual capture.
5. Set `state` to `Enabled` (or `Test Mode`). The form immediately sets `is_published` to true for `Enabled` and to false for any other state. If tokens exist and the stored state was `test` or `enabled`, a warning dialog announces that those tokens will be archived.
6. Save. The write performs, in order:
   1. archive every token of the providers that are leaving `test` or `enabled`;
   2. apply the write;
   3. check that every field required for this provider's code is filled; when one is empty the save is refused with `The following fields must be filled: %s` listing the human-readable labels of the missing fields;
   4. run the connector's own checks (for instance: a Mercado Pago provider may not be enabled without an access token, a demo provider may never be enabled, a Toss Payments provider may only carry the Korean won);
   5. activate the provider's default payment methods, and their brands, among the methods that are compatible with manual capture (see 1.3);
   6. switch the post-processing scheduled job on, because at least one provider is now not disabled.
7. The Accounting Payments package computes `journal_id`: the first bank journal of the company, and creates or re-points the accounting payment method line of the provider to that journal with the right outstanding account.

**Postconditions**: the provider is enabled, published, has a journal and a payment method line, and its default payment methods are active.

## 1.3 Activate a payment method for a provider

**Actor**: system administrator.

1. From the provider form, click **Enable Payment Methods**. The list of the provider's methods opens, archived methods included, without a create button.
2. Toggle the `active` flag of a method.
3. When switching a method on, the system checks that the primary method (the method itself when it is primary) is either already active or supported by at least one provider whose state is not `disabled`. Otherwise the change is refused with `This payment method needs a partner in crime; you should enable a payment provider supporting this method first.`
4. The system also checks that the method is compatible with manual capture: an active method whose `support_manual_capture` (taken from its primary method when it is a brand) is `none` may not be attached to a provider whose `capture_manually` is true. Otherwise the change is refused with `The following payment methods cannot be enabled because their payment provider has manual capture activated: %s`.
5. When switching a method off, every active token that uses it or one of its brands is archived; the user is warned first with the count.

## 1.4 Disable a provider

**Actor**: system administrator.

1. Set `state` to `Disabled` and save.
2. Every active token of that provider is archived.
3. `is_published` becomes false through the on-change rule.
4. After the write, every payment method whose supporting providers are now all disabled is deactivated, together with its brands.
5. If no provider of the database is left in a state other than `disabled`, the post-processing scheduled job is switched off.

**Postconditions**: the provider is no longer offered; its tokens are archived and cannot be unarchived while the provider stays disabled (rule PAY-RULE-025); and no request may be sent to it (rule PAY-RULE-032).

## 1.5 Reset the credentials of a provider

**Actor**: system administrator.

1. Click **Reset credentials** on the provider form.
2. The provider is written with `state` equal to `disabled`, `is_published` false, and the credential fields that the connector declares resettable set to empty. For Mercado Pago this also switches `allow_tokenization` off, because tokenization requires a connected account.
3. All the side effects of a state change to `disabled` (1.4) apply.

## 1.6 Publish or unpublish a provider

**Actor**: system administrator.

1. Click the **Published** or **Unpublished** button on the provider form.
2. If the provider's state is `disabled` and it is currently unpublished, the operation is refused with `You cannot publish a disabled provider.`
3. Otherwise `is_published` is inverted.

**Effect**: an unpublished provider is filtered out of the payment form for every user who is not an internal user. It stays available to internal users, and its existing tokens remain usable and visible on the payment method management page.

## 1.7 Guided setup of a first provider

**Actor**: system administrator, from a configuration screen or from the "no payment method available" notice on a payment form.

1. The system computes which provider to propose: Razorpay when the company's currency is the Indian rupee, otherwise Stripe when the company's country is supported by Stripe, otherwise Mercado Pago when the company's country is supported by Mercado Pago, otherwise none.
2. If none, nothing happens.
3. The package of the proposed provider is installed if needed, and a fresh environment including it is built.
4. The provider record of that code for the current company is looked up. If there is none, nothing happens.
5. The connector's guided setup operation runs and returns an action: an external redirect to the provider's authorization page for Mercado Pago, PayU and Razorpay; an external redirect to the provider's account onboarding link for Stripe; nothing for every other connector.
6. The provider returns the browser to the connector's return endpoint, which writes the obtained credentials, sets `state` to `enabled` and `is_published` to true, and redirects to the provider form.

## 1.8 Uninstall a provider connector package

**Actor**: system administrator.

1. The package removal hook runs the provider removal step with the provider's code.
2. The Accounting Payments package first checks whether any Payment uses the accounting payment method of that code. If one does, the uninstallation is refused with `You cannot uninstall this module as payments using this payment method already exist.`
3. Every provider record with that code is written with the removal values: `code` becomes `none`, `state` becomes `disabled`, `is_published` becomes false, and `redirect_form_view_id`, `inline_form_view_id`, `token_inline_form_view_id` and `express_checkout_form_view_id` are emptied. For custom providers, `custom_mode` is emptied as well.
4. Because `state` changes away from `test` or `enabled`, every token of those providers is archived and the payment methods that lose their last enabled provider are deactivated.
5. The accounting payment method of that code is deleted.

**Postconditions**: the provider records survive with all their history; the transactions keep pointing at them.

---

# 2. Paying from the portal

## 2.1 Display the payment form

**Actor**: customer (public visitor, portal user or internal user).
**Entry point**: the generic pay route, or a document portal page that embeds the same form.

Inputs: an optional reference prefix, an optional amount, an optional currency identifier, an optional contact identifier, an optional company identifier and an optional access token.

1. Cast the currency identifier, the contact identifier and the company identifier to integers, and the amount to a decimal. A malformed value becomes empty rather than an error, in order that a broken link does not block the payment.
2. If a contact identifier was supplied, verify the access token against the triple (contact identifier, amount, currency identifier). If it does not match, answer "not found" which prevents any information about existing identifiers from leaking.
3. Determine the paying contact:
   - a logged-in user always pays as their own contact; when the supplied contact differs, the page is rendered with a warning `Warning Make sure you are logged in as the correct partner before making this payment.`;
   - a logged-out visitor pays as the supplied contact; when that contact does not exist, the visitor is redirected to the login page with the current full path as the return address.
4. Fill the defaults: the reference prefix falls back to a time-based prefix; the amount falls back to zero; the company falls back to the contact's company and then to the user's company; the currency falls back to the company's currency.
5. Verify that the currency exists and is active; otherwise answer "not found".
6. Compute the compatible providers, the compatible payment methods and the available tokens with elevated rights, filling an availability report along the way (see `calculations.md`, sections "Provider availability" and "Payment method availability").
7. Check that the contact may pay in this company: the contact must either have no company or have exactly this company. When it may not, the page shows `Please switch to company <company name> to make this payment.` and no form.
8. Generate a fresh access token over (contact identifier, amount, currency identifier), because the contact or the currency may have been replaced in step 3 or 4.
9. Decide, per provider, whether the "save my payment method" checkbox is shown: it is shown when the provider allows tokenization and the context does not already require it.
10. Render the page. The page shows, in order: the breadcrumb, a blocking notice when the amount is zero (`There is nothing to pay.`), when the currency is missing (`Warning The currency is missing or incorrect.`), when there is no contact (`Warning You must be logged in to pay.`) or when the companies do not match; otherwise a summary block with the amount and the reference, followed by the payment form.
11. The payment form lists, in order: the customer's saved tokens (heading `Your payment methods`), then the compatible payment methods (heading `Payment method`, or `Other payment methods` and collapsed when tokens exist). A token is preselected when tokens exist; otherwise the single payment method is preselected when there is exactly one. A submit button labelled `Pay` closes the form. When neither tokens nor methods are available the form is replaced by the notice `No payment method available`, extended for administrators with the reason, a link to the availability report, a link to the provider list or the method list, and, when no provider is configured at all and a guided setup is possible, a button that starts it.

## 2.2 Create a transaction from the payment form

**Actor**: customer, through the transaction service of the payment form.
**Inputs**: the amount, the currency identifier, the contact identifier, the access token, and the form data: provider identifier, payment method identifier, token identifier, flow (`redirect`, `direct` or `token`), whether tokenization was requested, the landing route, the reference prefix, and whether the operation is a validation.

1. Cast the amount to a decimal.
2. Verify the access token against (contact identifier, amount, currency identifier). If it does not match, refuse the request as forbidden.
3. Verify that the supplied parameter names are all in the allowed list: provider identifier, payment method identifier, token identifier, amount, flow, tokenization requested, landing route, validation flag, the cross-site request forgery token, and the reference prefix. Any other name is rejected with `The following parameters are not whitelisted: %s`, listing the rejected names, and the answer is a bad request. Document-specific routes extend the allowed list with their own names.
4. Prepare the create values:
   - for the `redirect` and `direct` flows: the token is cleared, and tokenization is switched on only when the provider allows tokenization **and** the payment method supports tokenization **and** either the context requires tokenization or the customer asked for it. A customer who forces the flag through the browser therefore cannot create a token;
   - for the `token` flow: the token is read with elevated rights, and the commercial contact of the paying contact must equal the commercial contact of the token's owner, otherwise the request is refused with `You do not have access to this payment token.`; the payment method is taken from the token.
5. Compute the reference from the provider's code, the reference prefix and every other create value.
6. For a validation operation, replace the amount by the provider's validation amount and the currency by the provider's validation currency (see `calculations.md`, section "Validation amount and currency").
7. Create the transaction with elevated rights: provider, payment method, reference, amount, currency, contact, token, `operation` equal to `online_redirect`, `online_direct` or `online_token` (or `validation`), the tokenization flag, the landing route, plus any document links supplied by the calling route.
8. For the `redirect` and `direct` flows, log the "sent" message on the linked documents. For the `token` flow, charge the token immediately unless the caller asked to delay the charge.
9. Register the transaction as the one being monitored in the visitor's session.
10. Append to the landing route the transaction identifier and an access token. For a validation transaction the access token is recomputed over the provider's validation amount and currency.
11. Return the processing values of the transaction (see `interfaces.md`, section "Transaction processing values").

**Postconditions**: a transaction exists in state `draft`; the browser has everything it needs to continue.

## 2.3 Pay with redirection

**Actor**: customer.
**Precondition**: the chosen provider's connector uses the redirect flow for the chosen method, therefore `operation` is `online_redirect`.

1. While building the processing values, the system asks the connector for the redirect form template (which may differ for a validation operation), renders it with the connector's rendering values, and returns the rendered form.
2. The rendering values always contain the web address of the provider's payment page, under the key the redirect template reads, plus, depending on the connector, a set of parameters to submit and a signature. Several connectors obtain that web address by calling the provider first: Mollie and Mercado Pago create a payment or a preference, Xendit creates an invoice, Worldline creates a hosted checkout session, Iyzico initialises a checkout form, Flutterwave creates a payment link, DPO creates a transaction token, Paymob creates an intention. When that call fails, the connector sets the transaction to `error` with the provider's message and returns no rendering values.
3. The browser submits the rendered form to the provider.
4. The customer pays on the provider's page.
5. The provider sends the browser back to the connector's return endpoint. The handling is described in section 3.
6. The return endpoint redirects the browser to the payment status page.

## 2.4 Pay without leaving the platform

**Actor**: customer.
**Precondition**: the connector builds an inline form, therefore `operation` is `online_direct`.

1. The inline form template of the provider is rendered next to the chosen payment method on the payment form. It is given the connector's inline form values, which are a serialised structured data document; the values per connector are listed in `provider-connector-contracts.md`.
2. The customer types the payment details into the provider's own browser component; the details never reach the platform.
3. The browser calls the transaction service (2.2) with the flow `direct`, receives the processing values, and then calls the connector's own payment endpoint or confirms the provider-side intent, depending on the connector.
4. The connector processes the provider's answer immediately (section 3.3) and the browser is sent to the payment status page.
5. When the provider requires an additional step, such as a strong customer authentication challenge, the customer is redirected to the provider and comes back through the connector's return endpoint; the Adyen connector then rewrites the transaction's `operation` to `online_redirect`, and the Worldline connector resets the transaction to `draft` with operation `online_redirect` and forces the redirect flow.

## 2.5 Pay with a saved token

**Actor**: customer.
**Precondition**: the customer has at least one active token of a compatible provider.

1. The customer selects the token on the payment form and submits.
2. The transaction service creates the transaction with `operation` equal to `online_token` and the token set.
3. The charge runs at once: the provider's state is checked (a disabled provider refuses with `Making a request to the provider is not possible because the provider is disabled.`), the "sent" message is logged, and the connector sends the payment request. If the request raises a validation failure, the transaction is set to `error` with that message.
4. The connector processes the provider's answer (section 3.3).
5. The browser is sent to the payment status page.

## 2.6 Save a payment method without paying

**Actor**: logged-in customer, from the payment method management page.
**Precondition**: at least one provider allows tokenization and at least one of its methods supports tokenization.

1. The management page computes the compatible providers with the amount zero, tokenization forced and the validation flag set, and the compatible methods with tokenization forced. The available tokens are computed differently here: every token of the contact **and** of its commercial contact is listed, whatever the state of its provider, in order that a customer can always delete a token.
2. The page renders the payment form in validation mode: tokens cannot be selected, tokens can be deleted, and the submit button is labelled `Save`.
3. The customer picks a method and submits. The transaction service is called with the validation flag; the transaction is created with `operation` equal to `validation`, the provider's validation amount and the provider's validation currency.
4. The connector runs its validation flow. Providers that charge a small amount (Authorize charges 0.01, Razorpay charges 1.00) immediately void or cancel the charge afterwards; providers that support a zero-amount setup (Stripe) create a setup intent instead of a payment intent.
5. When the transaction reaches `authorized` or `done` and `tokenize` is true, the token is created (section 3.5).
6. The landing route of this flow is the management page itself, therefore the customer comes back to the list of saved methods.

**Postconditions**: a Payment Token exists for the customer, the provider and the method.

## 2.7 Delete a saved payment method

**Actor**: logged-in customer.

1. The customer clicks the delete icon next to a token on the management page.
2. The archive service reads, with elevated rights, the token whose identifier was given **and** whose contact is the user's contact or its commercial contact. When no such token exists, nothing happens.
3. `active` is set to false. The connector's archiving handler runs first with elevated rights.

**Postconditions**: the token no longer appears on any payment form. It is never deleted, and past transactions keep their link.

## 2.8 Payment status page and post-processing trigger

**Actor**: customer, after any payment flow.

1. The browser lands on the payment status route. The route reads the transaction identifier stored in the visitor's session.
2. When the session no longer holds one, the page shows `Your payment is on its way!`, `You should receive an email confirming your payment within a few minutes.` and `Don't hesitate to contact us if you don't receive it.`
3. Otherwise the page shows a status block whose heading and message depend on the transaction state and operation:

| State | Operation | Heading while processing | Message |
|---|---|---|---|
| `draft` | any | `Please wait...` | `Your payment has not been processed yet.` |
| `pending` | `validation` | `Please wait...` | `Saving your payment method.` |
| `pending` | other | `Please wait...` | the provider's pending message |
| `authorized` | any | `Please wait...` | the provider's authorization message |
| `done` | `validation` | none; on the confirmation page the heading is `Thank you!` | `Your payment method has been saved.` |
| `done` | other | none; on the confirmation page the heading is `Thank you!` | the provider's done message |
| `cancel` | `validation` | none | `The saving of your payment method has been canceled.` |
| `cancel` | other | none | the provider's cancel message |
| `error` | `validation` | none | `An error occurred while saving your payment method.` |
| `error` | other | none | `An error occurred during the processing of your payment.` |

   The transaction's `state_message` is appended below the message whenever it is set. A `Skip` link pointing at the landing route is offered while the page is polling.
4. The page polls the status service. Each poll runs the post-processing step when `is_post_processed` is false, then returns the provider code, the state and the landing route.
5. When the post-processing step fails because the database transaction could not be committed, the poll rolls back and asks the browser to retry. When it fails for any other reason, the poll rolls back, logs the exception and fails.
6. Once the transaction is no longer being processed, the browser follows the landing route.

## 2.9 Payment confirmation page

**Actor**: customer, arriving on the generic landing route.

1. The route reads the transaction identifier and the access token from the web address.
2. When the identifier is missing or malformed, the customer is redirected to the portal home.
3. The access token is verified against the triple (contact of the transaction, amount of the transaction, currency of the transaction). A mismatch answers "not found".
4. The page shows the state block (same table as 2.8, without the processing headings), then a summary with the amount, the reference, the payment method name (hidden when the method is the placeholder method whose code is `unknown`) and the provider name, and finally a button `Go to my Account`.

---

# 3. Processing a provider's answer

## 3.1 The three inbound channels

| Channel | When | Authentication | Typical answer |
|---|---|---|---|
| Response of an outbound request | The system called the provider and received the payment data in the reply. | The request itself was authenticated. | Processed immediately. |
| Return from checkout | The provider sends the customer's browser back to a return endpoint. | Signature of the returned data, or a second call to the provider to fetch the authoritative data, or an access token generated by the system. | Processed, then the browser is redirected to the payment status page. |
| Webhook notification | The provider calls a webhook endpoint, at any time, possibly several times. | Signature of the payload, a shared token in a header, or a verification call back to the provider. | Processed, then an acknowledgement body is returned. |

Every endpoint of every connector is public: the provider is not a logged-in user. The endpoints that receive the customer's browser after a form post are declared not to create a session, in order that a browser which does not send the session cookie on a cross-site redirect does not lose the session that holds the monitored transaction.

## 3.2 Verify a notification

1. Extract the received signature from the agreed place: a field of the payload, a request header, or an encoded parameter. When it is absent, log `Received payment data with missing signature.` and answer "forbidden".
2. Recompute the expected signature from the payload and the provider's secret, following the connector's algorithm (see `calculations.md`, section "Signature computation" and the per-connector entries).
3. Compare the two with a constant-time comparison. When they differ, log `Received payment data with invalid signature.` and answer "forbidden".

Three connectors verify differently: PayPal asks the provider to verify the signature of the event and refuses when the verification status is not a success; Xendit and Flutterwave compare a shared token sent in a header with the stored one; Toss Payments compares the secret stored on the transaction with the one in the event, and skips the check for the statuses that are exempt because no secret was ever stored. Nuvei, when the customer abandoned the page, verifies an access token that the system itself put in the return address instead of a provider signature.

## 3.3 Process payment data

This is the single entry point through which every channel feeds the payment data. Inputs: the provider code and the payment data.

1. If the caller already identified the transaction, use it. Otherwise search it by reference (3.4).
2. If no transaction was found, stop; the caller still acknowledges the notification.
3. Ensure exactly one transaction was found.
4. Remember the current state.
5. Validate the amount and the currency (see `calculations.md`, section "Amount validation"). A mismatch sets the transaction to `error`.
6. If the transaction is now in `error` and it was not before, stop: the updates are not applied.
7. Apply the connector's updates: the provider reference, the payment method actually used, and the new state (3.6).
8. If `tokenize` is true and the state is now `authorized` or `done`, create the token (3.5).

## 3.4 Match the payment data to a transaction

1. Ask the connector to extract the reference from the payment data. The default is the field named `reference`; every connector that names it differently overrides this.
2. When no reference can be extracted, log `Received payment data from provider <code> with missing reference` and return nothing.
3. Search for the transaction whose `reference` equals it **and** whose `provider_code` equals the provider code.
4. When nothing is found, log `No transaction found matching reference <reference>.` and return nothing.

Four connectors replace this search:

- **Adyen** matches by event kind. An authorization event is matched by reference. A cancellation, capture or capture-failure event is matched by the provider reference of the child transaction; the source transaction is found by the original provider reference; when the amounts do not agree the existing child is set to `error` with `The amount processed by Adyen for the transaction %s is different than the one requested. Another transaction is created with the correct amount.` and a new child is created; when no child exists at all, one is created from the event. A refund event is matched by provider reference, and a refund transaction is created from the source transaction when none exists.
- **Razorpay** matches a payment entity by the description field and a refund entity by the reference kept in the notes; a refund started at the provider is matched by the provider reference of the source transaction and a refund transaction is created, with `Received incomplete refund data.` when the identifier or the amount is missing.
- **Stripe** matches by reference, except for refund-update events, which carry no reference and are matched by the provider reference of the refund transaction.
- **Mercado Pago**, **Iyzico** and **Stripe** additionally re-check, after fetching the authoritative data, that the reference in that data equals the transaction's reference, and answer "forbidden" when it does not.

## 3.5 Tokenize a transaction

1. Ask the connector for the token values extracted from the payment data. When it returns nothing, stop.
2. Create a Payment Token with the transaction's provider, payment method and contact, plus the connector's values (always the provider reference, usually the clear payment details, and any connector-specific field).
3. Write the token on the transaction and set `tokenize` to false, in order that a later notification for the same transaction does not create a second token.
4. Log `Token <identifier> created for partner <identifier> from transaction <reference>.`

The demo connector tokenizes earlier, inside its update step, in order that the simulated state is stored on the token; its extraction step then returns nothing when the transaction is already `done` or `authorized`, to avoid a second token.

## 3.6 Apply the updates

Each connector writes, from the payment data:

1. the provider reference, when the data carry one;
2. the payment method actually used, by translating the provider's method code through the connector's mapping and looking it up; when nothing matches, the current method is kept (Mercado Pago falls back to the placeholder method whose code is `unknown`);
3. the new state, by classifying the provider's status through the connector's status mapping into pending, authorized, confirmed, canceled or error;
4. for a confirmed refund, an immediate wake-up of the post-processing job, because no customer browser will ever poll for a refund.

Two behaviours are shared by several connectors: when the provider reports a successful authorization and the provider record uses manual capture, the transaction goes to `authorized` rather than `done`; and an unknown status always sets the transaction to `error` with a message naming the unknown status.

## 3.7 Acknowledge

Each webhook endpoint answers with the body the provider expects. The bodies per connector are listed in `provider-connector-contracts.md`. A failure inside the processing of a Stripe event is swallowed and still acknowledged, in order that the provider does not keep re-sending it.

---

# 4. Post-processing

## 4.1 What post-processing does

Post-processing is the step that turns a transaction state into business consequences. It runs at most once per state: the flag `is_post_processed` is set when it runs and cleared on every state change.

**Actors**: the customer's browser through the status service, the operation button on the transaction form, or the scheduled job.

Base behaviour: set `is_post_processed` to true.

### 4.1.1 Accounting consequences

For every transaction whose state is `done`:

1. Post every linked invoice that is still a draft.
2. Create a Payment when all of the following hold: the operation is not `validation`; the transaction has no Payment yet; and no child transaction of it is in state `done` or `cancel`. The last condition means that a source transaction which was split into captures and voids does not produce a Payment itself; its children do.
3. When a Payment exists, log on the linked documents: `The payment related to transaction <transaction link> has been posted: <payment link>`.

For every transaction whose state is `cancel`, cancel its Payment.

The creation of the Payment is specified in `accounting-effects.md`.

### 4.1.2 Sales consequences

For every transaction whose state is `pending`:

1. Move the linked quotations that are still in the draft state to the "sent" state, without tracking.
2. For a custom provider, write the computed order communication on each linked order (see `calculations.md`, section "Sales order communication").
3. Unless the operation is `validation`, send the "payment succeeded" email for the linked orders.

For every transaction whose state is `authorized`: confirm the order when the amount is sufficient (4.1.4), then, unless the operation is `validation`, send the "payment succeeded" email for the orders that were not confirmed.

For every transaction whose state is `done`:

1. Unless the operation is `validation`, confirm the order when the amount is sufficient, then send the "payment succeeded" email for the orders that were not confirmed.
2. When the automatic invoicing setting is on, invoice the linked sales orders (4.1.5).
3. Run the accounting consequences, which post the invoices just created.
4. When the automatic invoicing setting is on and the caller did not ask to skip sending: if the asynchronous email setting is on and the invoice-sending job exists, wake that job; otherwise send the invoices at once.

### 4.1.3 Point of sale consequences

See section 6.

### 4.1.4 Confirm a sales order from a transaction

1. Only a transaction linked to exactly one sales order can confirm it; grouped payments over several orders never confirm anything.
2. The order must be a quotation (state draft or sent).
3. The order must not require a signature.
4. The amount received must reach the confirmation threshold of the order (the prepayment amount, or the full amount, according to the order's own rule, which belongs to the sales domain).
5. When all four hold, the order is confirmed, with the email sending and the signature inclusion switched on.

### 4.1.5 Invoice the linked sales orders

1. Keep the orders that are confirmed.
2. Split them into fully paid orders and partially paid orders.
3. For the partially paid orders, generate a down payment invoice for the transaction's amount.
4. For the fully paid orders, force the order lines to the "ordered quantity" invoicing policy and create a final invoice.
5. Make sure each created invoice has a portal access token.
6. Link the created invoices to the transaction.

## 4.2 The post-processing scheduled job

**Name**: "Payment: Post-process transactions". Runs as the system user every 10 minutes. It is inactive by default and is switched on whenever at least one provider is not disabled and off when none is.

1. When called without a specific set, search the transactions where `is_post_processed = false AND last_state_change >= now − 4 days`. Four days is the longest verification delay observed with the slowest provider.
2. For each transaction, in isolation:
   1. re-read `is_post_processed`, because another flow may have processed it since the search;
   2. run the post-processing step;
   3. commit.
3. When the database refuses the commit because of a concurrency conflict, roll back and leave the transaction for the next run.
4. On any other failure, log `An error occurred while post-processing transaction <reference>:` with the failure, and roll back.

**Postconditions**: a transaction whose customer never came back is still processed, for up to four days after its last state change.

---

# 5. Capture, void and refund

## 5.1 Capture an authorized amount

**Actor**: billing user, from a transaction, an invoice or a sales order.
**Precondition**: the provider uses manual capture; the transaction is in `authorized`.

1. The caller's write access on the transactions is checked; a user without it is refused by the access rules.
2. If no provider of the selected set supports partial capture, each authorized transaction is captured in full straight away (step 6 with the full amount) and a feedback notification is returned.
3. Otherwise the capture wizard opens on the transactions whose state is `authorized` or `done`.
4. The wizard computes the authorized, captured, voided and available amounts (see `calculations.md`, section "Capture allocation"), proposes `amount_to_capture` equal to the available amount, and offers the checkbox "void the remaining amount" only while the amount to capture is lower than the available amount.
5. On confirmation the wizard allocates the requested amount over the source transactions in their order, and, when asked, voids what is left of each source transaction:
   1. `remaining_to_capture` starts at `amount_to_capture`;
   2. for each source transaction in state `authorized`:
      - `source_remaining` = the source amount minus the sum of the amounts of its children that are `done`, rounded to the currency;
      - when `remaining_to_capture` is not zero: capture `min(source_remaining, remaining_to_capture)`; subtract it from both `remaining_to_capture` and `source_remaining`;
      - when `source_remaining` is not zero and the void checkbox is ticked: void `source_remaining`;
      - otherwise, when `remaining_to_capture` has reached zero and the void checkbox is not ticked: stop iterating.
6. Capturing an amount:
   1. refuse when the provider is disabled (`Making a request to the provider is not possible because the provider is disabled.`);
   2. create a child transaction with that amount, the same provider, payment method, currency, token, contact and operation as the source, the source as `source_transaction_id`, and a reference computed from the prefix `P-` followed by the source reference;
   3. log the child's "sent" message on the linked documents;
   4. send the capture request; a failure sets the child to `error` with the provider's message.
7. The provider answers, synchronously or by notification. When the answer confirms the capture, the child goes to `done`.
8. Confirming a child updates the source transaction state (see 5.3).
9. A feedback notification is returned.

## 5.2 Void an authorized amount

**Actor**: billing user.
**Precondition**: every selected transaction is in `authorized`.

1. Refuse with `Only authorized transactions can be voided.` when any selected transaction is not authorized.
2. For each transaction: compute the already captured amount as the sum of the amounts of its children that are `done` and share its operation, and void the difference.
3. Voiding an amount follows the same shape as capturing: refuse when the provider is disabled; create a child transaction with the amount, the same operation as the source, and the reference prefix `P-`; log the "sent" message; send the void request; a failure sets the child to `error`.
4. When the provider confirms the void, the child goes to `cancel`, and the source transaction state is updated (5.3).
5. The Razorpay connector refuses the void outright with `Transactions processed by Razorpay can't be manually voided from the system.`; only the amount left uncaptured expires at the provider.

## 5.3 How captures and voids close the source transaction

Whenever a child reaches `done` or `cancel`:

1. Collect the children of its source whose state is `done` or `cancel` **and** whose operation equals the child's operation.
2. Sum their amounts and round to the currency.
3. When the sum equals the source's amount:
   - if every one of those children is `cancel`, the source moves to `cancel`;
   - otherwise the source moves to `done`.
   The move is allowed only from `authorized`.
4. The source logs its "received" message.

While the sum is lower than the source amount, the source stays `authorized`, in order that the rest can still be captured or voided.

## 5.4 Refund a confirmed transaction

**Actor**: billing user, from the Payment (through the refund wizard) or from the transaction.
**Precondition**: the transaction is in `done`; the provider and the primary payment method both support refunds.

1. Refuse with `Only confirmed transactions can be refunded.` when any selected transaction is not confirmed.
2. From a Payment, the refund wizard first computes the refundable amount: the payment amount minus the absolute value of the sum of the amounts of the refund payments already linked to it. It refuses an amount that is not strictly positive or that exceeds it.
3. Refunding an amount:
   1. refuse when the provider is disabled;
   2. create a child transaction with the amount **negated**, the operation `refund`, the source as `source_transaction_id`, and a reference computed from the prefix `R-` followed by the source reference;
   3. log `The refund <link> of <formatted amount> has been initiated.` with the amount shown positive;
   4. send the refund request; a failure sets the refund transaction to `error`.
4. When the provider confirms, the refund transaction goes to `done`; the post-processing job is woken immediately, because no browser polls for a refund.
5. Post-processing creates a Payment for the refund transaction, with the direction reversed (see `accounting-effects.md`).
6. Refund transactions never change the state of their source transaction, because their operation differs from the source's operation.

The Authorize connector chooses between voiding and refunding, based on the state of the payment at the provider: a payment already voided at the provider cancels the source transaction; a payment already refunded at the provider confirms the refund transaction; a payment not yet settled is voided instead of refunded; a settled payment is refunded. A void that succeeds for a refund transaction marks that refund transaction as `done`, because the money did come back.

## 5.5 Refund initiated at the provider

Adyen, Razorpay and Stripe all recognise a refund that was started on the provider's own interface: the notification carries no reference known to the platform, therefore the source transaction is found by its provider reference and a refund child transaction is created on the spot with the amount converted back to major units. Stripe additionally pages through all the refunds of a charge and creates one refund transaction per refund that has no transaction yet, including refunds of capture children.

---

# 6. Paying documents

## 6.1 Pay a customer invoice from the portal

**Actor**: customer.

1. The portal page of the invoice offers the payment form when the invoice may be paid online (see `entities.md`, section 8.9).
2. The form's transaction route is the invoice transaction route, and its landing route is the portal page of the invoice with its access token.
3. The transaction route checks the document access with the supplied token; a failure answers `The access token is invalid.`
4. The transaction is created with the invoice linked through `invoice_ids`, and its reference prefix is computed from the invoice names (see `calculations.md`, section "Reference generation").
5. When the transaction is confirmed, post-processing posts the invoice if it is still a draft, creates the Payment and reconciles it with the invoice.

A second route pays every overdue invoice of the logged-in customer at once: it refuses an anonymous visitor with `Please log in to pay your overdue invoices` and refuses a set of invoices that do not share one currency with `Impossible to pay all the overdue invoices if they don't share the same currency.`

## 6.2 Pay a sales order from the portal

Same shape as 6.1, with the sales order linked through `sale_order_ids` and the reference prefix computed from the order names. Confirmation of the order happens in post-processing (4.1.4).

## 6.3 Pay with a wire transfer or another custom mode

**Actor**: customer.

1. The customer picks the wire transfer method on the payment form. The connector's rendering values contain only the platform's own custom-processing web address, which is the fixed path `/payment/custom/process`, and the reference. The browser posts the reference to that path.
2. The browser posts to that address. The controller processes the data with the provider code `custom`.
3. The connector skips the amount validation entirely, then sets the transaction to `pending` and logs `Validated custom payment for transaction <reference>: set as pending.`
4. No "received" message is logged on the documents for a custom provider; the "sent" message is replaced by `The customer has selected <provider name> to make the payment.`
5. The customer is redirected to the payment status page, which shows the provider's pending message. For a wire transfer provider that message lists the company's bank accounts.
6. The communication the customer must use is: the payment reference of the first linked invoice, or the reference of the first linked sales order, or, when neither exists, the transaction reference.
7. An employee later confirms the receipt of the money by reconciling the bank transaction; that step belongs to [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md).

## 6.4 Charge a token offline from a Payment

**Actor**: billing user.

1. The user creates a Payment (or opens the payment registration wizard), selects an electronic payment method line and a saved token. Only tokens of providers that do not use manual capture are offered.
2. On posting, the payments that have a token and no transaction yet are separated out. A transaction is created for each of them, with elevated rights, with `operation` equal to `offline`, the payment's amount, currency and contact, the token, a reference computed from the payment's memo, the payment itself, and the invoices taken from the screen context.
3. The other payments are posted normally.
4. Each new transaction is charged with its token.
5. The transactions are post-processed.
6. The payments whose transaction reached `done` are posted.
7. The payments whose transaction ended in a state other than `done`, `pending` or `authorized` are cancelled.

## 6.5 Register an online point of sale payment

**Actor**: customer, from a point of sale order.

1. The transaction is created with the point of sale order linked, and its reference prefix is the order's own reference.
2. Post-processing, for a transaction in state `authorized` or `done` whose point of sale order has no payment yet:
   1. refuse a non-positive amount with `The payment transaction (%d) has a negative amount.`;
   2. create the Payment when it does not exist, and refuse with `The point of sale online payment (transaction %d) could not be saved correctly` when it still does not;
   3. find the online payment method of the order, creating the configuration's online payment method when needed, and refuse with `The point of sale online payment (transaction %d) could not be saved correctly because the online payment method could not be found` when there is none;
   4. add a payment line to the point of sale order with the transaction amount, the moment of the last state change as payment date, the online payment method and the created Payment;
   5. write the point of sale payment method, order and session on the Payment;
   6. when the order is still a draft and is now fully paid, process it;
   7. notify the point of sale screen in order that it refreshes.

## 6.6 Generate a payment link

**Actor**: billing user or salesperson.

1. Open the payment link wizard from the document. The wizard reads its defaults from the document: currency, contact, amount, maximum amount and, for an invoice, the open installments and the early payment discount data.
2. Adjust the amount. The warning message rules of `entities.md`, section 6.2, apply.
3. The wizard computes the link (see `entities.md`, section 6.3) and offers it for copying.
4. The customer opens the link, which lands on the pay page or the document's portal page with the amount, the currency, the contact, the company and a signed access token.

## 6.7 Donate from a website page

**Actor**: any visitor of a public website page that carries the donation block.

The Website Payment capability package adds a donation block to the page editor and two endpoints. The block is a form whose action is the donation pay page and whose method is a post; the editor sets on it a recipient electronic mail address, a custom-amount mode, a list of prefilled amounts, one description per prefilled amount, a minimum amount, a maximum amount, a slider step and a default amount.

1. **The visitor picks an amount.** The block either shows the prefilled amount buttons with their descriptions and a free-amount box, or a slider between the minimum and the maximum with the configured step, according to the custom-amount mode. The visitor submits the form.
2. **The post is turned into a page address.** The donation pay endpoint receives the post, stores the amount, the currency, the donation options and the list of descriptions in the visitor's session, and answers with a redirection to the same address using the "see other" status, so that a refresh of the page does not repeat the post.
3. **The page is served.** The endpoint reads back from the session whatever was not passed again on the address, then applies three defaults: the currency falls back to the accounting currency of the active company; the amount falls back to 25; the donation options fall back to a free custom amount. For a visitor who is not signed in, the endpoint sets the paying contact to the public contact of the request and computes a signed access token over that contact, the amount and the currency, exactly as the ordinary pay page does (see [calculations.md](calculations.md) section 11).
4. **The payment form is rendered from the donation template** instead of the ordinary pay template, with these extra values: the donation flag, the paying contact, the label of the submit button, which is `Donate`, the transaction-creation address, which carries the minimum amount as the last part of its path, the donor details taken from the signed-in contact when there is one, the list of countries, the donation options and the prefilled amounts with their descriptions. For a visitor who is not signed in, the "save my payment details" box is hidden for every provider, so that no token can be created for the public contact.
5. **The visitor confirms.** The donation transaction endpoint is called with the amount, the currency, the contact, the access token, the donor details, an optional donor comment and the recipient electronic mail address. It refuses an amount below the minimum with `Donation amount must be at least %.2f.`, where the placeholder is the minimum amount rendered with two decimals; it refuses missing donor details with `Name is required.`, `Email is required.` and `Country is required.`, in that order.
6. **The transaction is created** by the ordinary transaction-creation service, with three additions: for a visitor who is not signed in, the paying contact is the website's public contact and the tokenize flag is forced off; the donation flag is set on the transaction; and, for a visitor who is not signed in, the contact snapshot fields for name, electronic mail address, country and language are overwritten with the donor details and the language of the request. For a signed-in visitor whose contact has no country, the country of the donor details is written to the snapshot.
7. **The access token is recomputed** over the contact, the amount actually chosen and the currency, because the visitor may have changed the amount on the payment page, and the landing address of the transaction is updated with it.
8. **An internal notification is sent immediately**, before the payment is even attempted, to the recipient electronic mail address configured on the block, with the subject `A donation has been made on your website` and the donation body rendered in the language of the recipient's user account, or in the language of the company contact when no user matches that address. The body carries the donor name, the donor electronic mail address, the donation date, the amount with the currency symbol, the donor comment when there is one, the provider code and the transaction reference.
9. **The processing values are returned** and the ordinary payment flow of section 2 continues unchanged.
10. **On confirmation**, the post-processing of a transaction whose state is `done` and whose donation flag is set does two more things than an ordinary transaction: it sends the donation confirmation message to the donor, with the subject `Donation confirmation`, in the language of the contact snapshot, whose body opens with `Dear ` and the donor name and thanks the donor for the donation, stating the amount and the creation date, and then carries the same detail table as the internal notification; and it writes a log entry on the Payment reading `Payment received from donation with following details:` followed by one line per filled value among the company, the contact, the contact name, the contact country and the contact electronic mail address, each prefixed by the label of that field.

The donation flag is also mirrored on the Payment, so that a donation can be told apart in the accounting screens.

## 6.8 Show the payment methods a website supports

**Actor**: any visitor of a public website page that carries the supported payment methods block.

1. The block calls the supported payment methods endpoint with an optional limit.
2. The endpoint computes the compatible providers of the website's own company, as the public user of the website so that an editor sees exactly what a visitor will see, for a zero amount, no currency and no country, restricted to the website. The company of the website is used rather than the company of the caller, because the block advertises the site and not the visitor.
3. It then selects the payment methods to show: every brand whose primary method is active and whose primary method has at least one of those providers, plus every primary method that has no brand at all and has at least one of those providers. The limit, when given, caps the number of records.
4. For each selected method it returns the name and the address of the method image.
5. The answer is marked as not cacheable for an internal user, so that an editor always sees the current list, and cacheable for seven days with one further day of stale reuse for everybody else.

---

# 7. State machine tables

The complete specification of every state field of this domain — the stored value, label and meaning of each state, the guards of each transition with their exact refusal texts, the side effects in the order in which they happen, and a diagram per machine — is in [state-machines.md](state-machines.md). The three tables below summarise the transitions that the flows of this document produce, in the order in which those flows produce them, and are consistent with that document.

## 7.1 Payment Transaction state

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| (none) | The transaction is created | The reference is unique; a token, when given, is active | `draft` | Snapshot of the contact; `is_live` set from the provider state; `last_state_change` set to now |
| `draft` | The provider reports a pending payment | none | `pending` | Received message logged; `is_post_processed` cleared |
| `draft` | The provider reports an authorization and the provider uses manual capture | The provider supports manual capture | `authorized` | Received message logged |
| `draft` | The provider reports a successful payment | none | `done` | Received message logged; source transaction re-evaluated |
| `draft` | The provider reports a cancellation | none | `cancel` | Received message logged; source transaction re-evaluated |
| `draft` | The provider reports a failure, or the request to the provider failed | none | `error` | Received message logged with the reason |
| `pending` | The provider reports an authorization | The provider supports manual capture | `authorized` | Received message logged |
| `pending` | The provider reports a successful payment | none | `done` | Received message logged; source transaction re-evaluated |
| `pending` | The provider reports a cancellation or an expiry | none | `cancel` | Received message logged |
| `pending` | The provider reports a failure | none | `error` | Received message logged |
| `authorized` | A capture child transaction is confirmed and the captured amount reaches the source amount | none | `done` | Received message logged |
| `authorized` | Void children cover the whole amount | Every child of the same operation is `cancel` | `cancel` | Received message logged |
| `authorized` | The provider reports the capture of the whole amount | none | `done` | Received message logged |
| `authorized` | The provider reports a cancellation | none | `cancel` | Received message logged |
| `authorized` | The provider reports a failure | none | `error` | Received message logged |
| `error` | The provider later reports a successful payment | none | `done` | Received message logged. This is the only transition out of `error`. |
| `done` | The Authorize connector discovers the payment was voided at the provider | Only that connector, which allows `done` as an extra source state | `cancel` | Received message logged |
| `done` | The Stripe connector reports that a succeeded refund was reversed | Only that connector, for refund transactions | `error` | Received message logged |
| any | A target state equal to the current state | none | unchanged | Informational log entry; nothing written |
| any | A target state whose allowed source states exclude the current state | none | unchanged | Warning log entry; nothing written |

## 7.2 Payment Provider state

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| `disabled` | The administrator selects Enabled | Every required credential filled; connector-specific checks pass | `enabled` | `is_published` set to true by the form; default payment methods activated; scheduled job switched on |
| `disabled` | The administrator selects Test Mode | Every required credential filled; the connector allows test mode | `test` | `is_published` set to false by the form; default payment methods activated; scheduled job switched on |
| `enabled` | The administrator selects Test Mode | The connector allows it (Stripe refuses while a connected account exists) | `test` | Tokens archived; `is_published` set to false |
| `test` | The administrator selects Enabled | Connector-specific checks pass (Stripe refuses while the onboarding is unfinished; the demo connector refuses any enabled state) | `enabled` | Tokens archived; `is_published` set to true |
| `enabled` or `test` | The administrator selects Disabled, or resets the credentials, or the package is uninstalled | none | `disabled` | Tokens archived; `is_published` set to false; unsupported payment methods deactivated; scheduled job possibly switched off |

## 7.3 Payment Token state

| From | Trigger | Guard | To |
|---|---|---|---|
| (none) | A transaction reaches `authorized` or `done` with `tokenize` true and the connector returns token values | The contact is not the public contact | active |
| active | The customer deletes it from the portal, or an administrator archives it | none | archived |
| active | Its provider leaves `test` or `enabled` | none | archived |
| active | Its payment method is archived, detached from the provider, or loses tokenization support | none | archived |
| archived | An administrator unarchives it | The payment method is active and the provider is not disabled, otherwise refused | active |

---

# 8. Worked end-to-end examples

The three examples below share this setting: company "Acme" with the accounting currency euro (`EUR`); a provider record named "Acme Card Gateway" with `state` equal to `enabled`, `support_manual_capture` equal to `partial`, `support_refund` equal to `partial`, a bank journal "Bank" whose outstanding receipts account is `101401 Outstanding Receipts`; a customer "Norbert Buyer" whose country is Belgium; a quotation `S00042` of 120.00 euro that requires full prepayment; the currency has two decimal places.

## 8.1 A payment of 120.00 that goes pending and then confirms the order

**Step 1. The customer opens the quotation portal page and clicks Pay.**

The payment form is rendered with `amount = 120.0`, `currency = EUR`, `partner = Norbert Buyer`, `transaction_route` = the sales order transaction route, `landing_route` = the portal page of `S00042`.

**Step 2. The customer selects the card method and submits.**

The transaction service creates:

| Field | Value |
|---|---|
| `reference` | `S00042` (no other transaction carries that reference yet) |
| `provider_id` | Acme Card Gateway |
| `payment_method_id` | Card |
| `amount` | 120.00 |
| `currency_id` | `EUR` |
| `partner_id` | Norbert Buyer |
| `partner_name`, `partner_email`, `partner_address`, `partner_zip`, `partner_city`, `partner_country_id`, `partner_phone`, `partner_lang`, `partner_state_id` | copied from the contact |
| `operation` | `online_redirect` |
| `state` | `draft` |
| `is_live` | true (the provider is `enabled`) |
| `tokenize` | false |
| `landing_route` | the portal page of `S00042`, with the transaction identifier and an access token appended |
| `sale_order_ids` | `S00042` |

The message `The transaction S00042 of €120.00 has been initiated.` is logged on `S00042`.

**Step 3. The provider answers "pending".**

The connector's status mapping classifies the provider status as pending. The transaction is written with `state = pending`, `state_message` empty, `last_state_change` = now, `is_post_processed` = false. The message `The transaction S00042 of €120.00 is pending.` is logged on `S00042`.

**Step 4. The browser polls the status service.**

Post-processing runs for the pending state: `S00042` moves from draft to sent, and the "payment succeeded" email is sent. `is_post_processed` becomes true.

**Step 5. Two hours later the provider sends a webhook notification reporting success.**

1. The signature is verified.
2. The transaction is matched by reference `S00042`.
3. The amount data extracted from the notification are `amount = 120.0`, `currency_code = EUR`. The transaction amount rounded down to two decimal places is 120.00. The comparison succeeds and the currency codes match, therefore the amount validation passes.
4. The connector writes the provider reference and sets the state to `done`. Allowed source states for `done` include `pending`, therefore the move is applied: `state = done`, `last_state_change` = now, `is_post_processed` = false.
5. The message `The transaction S00042 of €120.00 has been confirmed.` is logged on `S00042`.

**Step 6. The scheduled job (or the next poll) post-processes the confirmed transaction.**

1. The sales consequence runs first: the transaction is linked to exactly one quotation, the quotation does not require a signature, and 120.00 reaches the confirmation threshold of 120.00, therefore `S00042` is confirmed.
2. With automatic invoicing on, the order is fully paid, therefore a final invoice `INV/2026/00017` of 120.00 is created and linked to the transaction.
3. The accounting consequence posts `INV/2026/00017` and creates a Payment:

| Field | Value |
|---|---|
| `amount` | 120.00 |
| `payment_type` | `inbound` |
| `currency_id` | `EUR` |
| `partner_id` | the commercial contact of Norbert Buyer |
| `partner_type` | `customer` |
| `journal_id` | Bank |
| `payment_method_line_id` | the inbound line of Bank whose provider is Acme Card Gateway |
| `payment_transaction_id` | the transaction |
| `memo` | `S00042 - <provider reference>` |
| `invoice_ids` | `INV/2026/00017` |
| `destination_account_id` | the receivable account of the invoice's payment term line |

4. The Payment is posted; its journal entry debits `101401 Outstanding Receipts` 120.00 and credits the receivable account 120.00; the receivable line is reconciled with the invoice's receivable line, and the invoice becomes fully paid.
5. `The payment related to transaction S00042 has been posted: BNK1/2026/00031` is logged on the invoice, the order and the Payment.
6. `is_post_processed` becomes true.

## 8.2 Partial capture of 80.00 out of 120.00, then void of the remaining 40.00

Same setting, but the provider record has `capture_manually` true. The transaction `S00043` of 120.00 has reached `authorized`.

**Step 1. The billing user selects the transaction and clicks Capture Transaction.**

Because the provider's `support_manual_capture` is `partial`, the capture wizard opens with the transaction as its only source transaction. It computes:

```formula
authorized_amount = 120.00
captured_amount   = 0.00      (no confirmed child, and the source itself is not done)
voided_amount     = 0.00
available_amount  = 120.00 − 0.00 − 0.00 = 120.00
amount_to_capture = 120.00    (default)
```

**Step 2. The user sets the amount to capture to 80.00 and leaves the void checkbox unticked.**

The validity check passes because `0 < 80.00 ≤ 120.00`. `has_remaining_amount` becomes true because `80.00 < 120.00`, therefore the void checkbox becomes visible. Partial capture is supported by both the provider and the primary payment method, therefore the second part of the check passes.

**Step 3. Confirmation.**

```formula
remaining_to_capture = 80.00
source_remaining     = 120.00 − 0.00 = 120.00
amount_to_capture    = min(120.00, 80.00) = 80.00
```

A child transaction is created:

| Field | Value |
|---|---|
| `reference` | `P-S00043` |
| `amount` | 80.00 |
| `currency_id` | `EUR` |
| `operation` | `online_redirect` (copied from the source) |
| `source_transaction_id` | `S00043` |
| `state` | `draft` |

`The transaction P-S00043 of €80.00 has been initiated.` is logged. The capture request is sent. `remaining_to_capture` becomes 0.00 and `source_remaining` becomes 40.00. The void checkbox is not ticked and `remaining_to_capture` is zero, therefore the loop stops.

**Step 4. The provider confirms the capture.**

`P-S00043` moves to `done`. The source re-evaluation runs: the children of `S00043` in state `done` or `cancel` with operation `online_redirect` are `{P-S00043}`; their sum is 80.00; rounded to two decimals it is 80.00, which differs from 120.00, therefore `S00043` stays `authorized`.

Post-processing of `P-S00043` creates a Payment of 80.00 linked to the invoices of the source transaction (because the child's operation equals the source's operation, the reconciliation uses the source transaction's invoices).

**Step 5. The billing user clicks Void Transaction on `S00043`.**

`S00043` is still `authorized`, therefore the guard passes. The already captured amount is the sum of the amounts of the children of `S00043` that are `done` and share its operation, that is 80.00. The amount to void is `120.00 − 80.00 = 40.00`.

A second child is created:

| Field | Value |
|---|---|
| `reference` | `P-S00043-1` (the prefix `P-S00043` already exists, therefore the sequence number 1 is appended) |
| `amount` | 40.00 |
| `operation` | `online_redirect` |
| `source_transaction_id` | `S00043` |
| `state` | `draft` |

**Step 6. The provider confirms the void.**

`P-S00043-1` moves to `cancel`. The source re-evaluation runs: the children in `done` or `cancel` with operation `online_redirect` are `{P-S00043 (80.00, done), P-S00043-1 (40.00, cancel)}`; their sum is 120.00, which equals the source amount. Not every child is `cancel`, therefore the target state is `done`: `S00043` moves from `authorized` to `done`.

Post-processing of `S00043` does **not** create a Payment, because at least one child is in state `done` or `cancel`. Post-processing of `P-S00043-1` does not create a Payment either, because the void produced no money movement; its state is `cancel`, and the accounting consequence for a `cancel` transaction only cancels an existing Payment, of which there is none.

**Net result**: one Payment of 80.00 in the ledger, an authorized amount of 40.00 released at the provider, and the source transaction closed as confirmed.

## 8.3 A refund of 30.00 on a confirmed transaction of 120.00

Same setting as 8.1: the transaction `S00042` of 120.00 is `done` and its Payment `BNK1/2026/00031` of 120.00 is posted and reconciled.

**Step 1. The billing user opens `BNK1/2026/00031` and clicks Refund.**

The refund wizard computes:

```formula
payment_amount              = 120.00
refund payments up to now   = none, therefore their absolute sum is 0.00
amount_available_for_refund = 120.00 − 0.00 = 120.00
refunded_amount             = 120.00 − 120.00 = 0.00
amount_to_refund            = 120.00 (default)
support_refund              = partial   (both the provider and the primary method support partial)
has_pending_refund          = false
```

**Step 2. The user sets the amount to refund to 30.00 and confirms.**

`0 < 30.00 ≤ 120.00`, therefore the check passes. The transaction's refund operation runs with 30.00.

A refund transaction is created:

| Field | Value |
|---|---|
| `reference` | `R-S00042` |
| `amount` | −30.00 |
| `currency_id` | `EUR` |
| `operation` | `refund` |
| `source_transaction_id` | `S00042` |
| `token_id` | the token of the source transaction, when there was one |
| `state` | `draft` |

`The refund R-S00042 of €30.00 has been initiated.` is logged on the invoice and the order (the amount is shown positive).

**Step 3. The refund request is sent.**

The amount handed to the provider is the negated stored amount, 30.00, converted to the provider's unit; for a provider that works in minor units of the euro this is the integer 3000.

**Step 4. The provider confirms the refund.**

1. The amount validation negates the provider's positive amount for a refund operation: `amount = −30.0`. The transaction amount rounded down to two decimals is `−30.00`. They match.
2. The connector sets `R-S00042` to `done` and wakes the post-processing job at once.
3. The source transaction `S00042` is **not** re-evaluated, because the operation of the refund child (`refund`) differs from the operation of the source (`online_redirect`).
4. `The refund R-S00042 of €30.00 has been confirmed.` is logged.

**Step 5. Post-processing creates the refund Payment.**

| Field | Value |
|---|---|
| `amount` | 30.00 (the absolute value of the transaction amount) |
| `payment_type` | `outbound` (because the transaction amount is negative) |
| `partner_type` | `customer` |
| `journal_id` | Bank |
| `payment_transaction_id` | `R-S00042` |
| `source_payment_id` | `BNK1/2026/00031` |
| `memo` | `R-S00042 - <provider reference>` |

The journal entry credits `101401 Outstanding Receipts` 30.00 and debits the receivable account 30.00. Because the refund transaction's operation differs from the source's, the reconciliation uses the refund transaction's own invoices, which are empty, therefore no automatic reconciliation happens; an accountant matches the refund against a credit note afterwards.

**Step 6. A second refund is attempted.**

The wizard now computes `amount_available_for_refund = 120.00 − |−30.00| = 90.00`, and `refunded_amount = 30.00`. `refunds_count` on `S00042` is 1.
