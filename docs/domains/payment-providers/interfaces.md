# Interfaces

The service operations a client or an integration invokes, the request endpoints, the screens described as workflows on views, the notifications, and the scheduled jobs of the Payment Providers domain.

---

# 1. Service operations

Operations are named with a full-word snake_case identifier. Unless stated otherwise, an operation acts on a set of records and the guards of `business-rules.md` apply.

## 1.1 On Payment Provider

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| `get_compatible_providers` | company, contact, amount, currency, force tokenization flag, express checkout flag, validation flag, an availability report to fill, plus any context a capability package reads | The compatible providers | Fills the availability report | none |
| `is_tokenization_required` | the payment context | boolean | none | none |
| `should_build_inline_form` | validation flag | boolean | none | none |
| `get_validation_amount` | none | decimal | none | none |
| `get_validation_currency` | the payment method, when known | a currency | none | none |
| `get_redirect_form_view` | validation flag | a template | none | none |
| `get_supported_currencies` | none | a set of currencies | none | none |
| `get_default_payment_method_codes` | none | a set of codes | none | none |
| `setup_provider` | the provider code and the connector's extra arguments | none | Copies the reference provider into every company that lacks one; creates the accounting payment method | none |
| `remove_provider` | the provider code | none | Writes the removal values on every provider of that code; deletes the accounting payment method | `You cannot uninstall this module as payments using this payment method already exist.` |
| `toggle_post_processing_cron` | none | none | Switches the scheduled job on or off | none |
| `archive_linked_tokens` | none | none | Archives every token of the providers | none |
| `activate_default_payment_methods` | none | none | Activates the compatible default methods and their brands | none |
| `deactivate_unsupported_payment_methods` | none | none | Deactivates the methods whose providers are all disabled | none |
| `send_api_request` | request method, endpoint, query parameters, body, structured body, transaction reference, connector arguments | the parsed response | Writes a log entry before and after; masks sensitive keys | `Could not establish the connection to the payment provider.` on a connection failure or a timeout; `The payment provider rejected the request.` followed by the provider's message on an error status |
| `ensure_payment_method_line` | allow-create flag | none | Creates, re-points or deletes the accounting payment method line | none |
| `reset_credentials` | none | boolean | Disables, unpublishes and clears the connector's credentials | none |
| `toggle_is_published` | none | none | Inverts the published flag | `You cannot publish a disabled provider.` |
| `start_onboarding` | the menu the setup was started from | an action | Connector-specific | Connector-specific |

The request helper is a layered contract. The generic operation builds the web address, the headers and the basic authentication by calling four hooks that each connector overrides (`build_request_web_address`, `build_request_headers`, `build_request_authentication`, and the two parsers `parse_response_content` and `parse_response_error`), sends the request with a ten-second timeout, logs it, and parses the answer. Two further helpers exist for connectors that talk through the platform's proxy service: `prepare_proxy_payload`, which wraps the data in a remote-call envelope with a fresh unique call identifier, and `parse_proxy_response`, which unwraps the answer and raises `The payment provider rejected the request.` followed by the proxy's message when the envelope carries an error.

## 1.2 On Payment Method

| Operation | Inputs | Output |
|---|---|---|
| `get_compatible_payment_methods` | the compatible providers, contact, currency, force tokenization flag, express checkout flag, an availability report to fill | The compatible primary methods |
| `get_from_code` | a provider-specific code and an optional mapping | The matching method, or nothing |

## 1.3 On Payment Token

| Operation | Inputs | Output | Side effects |
|---|---|---|---|
| `get_available_tokens` | the compatible providers, the contact, the validation flag | The tokens to offer | none |
| `build_display_name` | maximum length, padding flag | text | none |
| `get_linked_records_info` | none | a list of records that use the token | none |
| `handle_archiving` | none | none | Connector-specific clean-up before the token is archived |

## 1.4 On Payment Transaction

| Operation | Inputs | Output | Side effects | Errors |
|---|---|---|---|---|
| `compute_reference` | provider code, prefix, separator, create values | a unique reference | none | none |
| `compute_reference_prefix` | separator, create values | a prefix or the empty text | none | none |
| `get_processing_values` | none | the processing values (1.5) | May call the provider, and may therefore change the state | none |
| `get_specific_processing_values` | the generic processing values | connector values | May call the provider | none |
| `get_specific_rendering_values` | the processing values | connector rendering values | May call the provider | Connector-specific |
| `charge_with_token` | none | none | Logs the sent message, sends the payment request, sets the state to `error` on failure | `Making a request to the provider is not possible because the provider is disabled.` |
| `capture` | amount to capture | the capture child transaction | Creates the child, logs, sends the request | same as above |
| `void` | amount to void | the void child transaction | Creates the child, logs, sends the request | same as above |
| `refund` | amount to refund | the refund child transaction | Creates the child, logs, sends the request | same as above |
| `create_child_transaction` | amount, refund flag, extra create values | the child transaction | Creates it with the right reference prefix, sign and operation | none |
| `process` | provider code, payment data | the updated transaction | Validates the amount, applies the updates, tokenizes | none; failures are expressed as a state change to `error` |
| `search_by_reference` | provider code, payment data | the transaction, or nothing | May create a child transaction for a capture, void or refund started at the provider | none |
| `extract_reference` | provider code, payment data | text | none | none |
| `validate_amount` | payment data | none | May set the transaction to `error` | none |
| `extract_amount_data` | payment data | amount, currency code, optional precision, or nothing | none | none |
| `apply_updates` | payment data | none | Writes the provider reference, the payment method and the state | none |
| `tokenize` | payment data | none | Creates the token and clears the tokenize flag | none |
| `extract_token_values` | payment data | the token create values | May call the provider | none |
| `set_pending`, `set_authorized`, `set_done`, `set_canceled`, `set_error` | state message, extra allowed source states | the transactions actually updated | Writes the state, the message, the change moment and clears the post-processing flag; logs the received message; re-evaluates the source transaction for `done` and `cancel` | none |
| `update_state` | allowed source states, target state, state message | the transactions actually updated | as above | none |
| `update_source_transaction_state` | none | none | Closes the source transaction when its children cover it | none |
| `post_process` | none | none | The business consequences of the current state | Propagated from the consequences |
| `cron_post_process` | none | none | The retry loop | Swallowed and logged per transaction |
| `get_last` | none | the last transaction that is not a draft | none | none |
| `log_sent_message`, `log_received_message`, `log_message_on_linked_documents` | a message for the last one | none | Writes a message on every linked document | none |

## 1.5 Transaction processing values

The structure returned to the browser when a transaction is created. It always contains:

| Key | Meaning |
|---|---|
| provider identifier | The provider handling the transaction. |
| provider code | Used by the browser to pick the right client behaviour. |
| reference | The transaction reference. |
| amount | The rounded amount. |
| currency identifier | The currency. |
| contact identifier | The paying contact. |
| should tokenize | Whether a token will be created. |
| state | The state of the transaction after every connector call that may have happened while the values were being built. |
| state message | The state message, for the same reason. |
| rendered redirect form | Present only for the `online_redirect` and `validation` operations, and only when the connector declares a redirect template. It is the fully rendered form that the browser submits to the provider. |

Connectors add their own keys; they are listed per connector in `provider-connector-contracts.md`.

## 1.6 Utility operations

| Operation | Inputs | Output |
|---|---|---|
| `generate_access_token` | an ordered list of values | a signed token |
| `check_access_token` | a token and the same ordered list | boolean |
| `add_to_report` | the report, the records, the availability flag, the reason | none |
| `singularize_reference_prefix` | prefix, separator, maximum length | a time-suffixed prefix |
| `to_major_currency_units` | minor amount, currency, optional precision | decimal |
| `to_minor_currency_units` | major amount, currency, optional precision | integer |
| `get_language_code` | language tag, mapping, fallback | a provider language code |
| `format_partner_address` | two street lines | one line |
| `split_partner_name` | a full name | first name, last name |
| `get_customer_internet_protocol_address` | none | text |
| `check_rights_on_recordset` | a set of records | none; fails when the caller may not write |
| `generate_idempotency_key` | the transaction, an optional scope | text |

---

# 2. Request endpoints

Every endpoint of this domain is reachable without authentication unless the authentication column says "logged-in user". The "session" column says whether the endpoint may create a new browsing session; endpoints marked "no" are those a provider redirects the browser to with a form post.

## 2.1 Generic endpoints

| Path | Method | Kind | Authentication | Session | Purpose |
|---|---|---|---|---|---|
| `/payment/pay` | `GET` | page | public | yes | Render the payment page. Parameters: reference prefix, amount, currency identifier, contact identifier, company identifier, access token, plus any parameter a capability package reads (invoice identifier, sales order identifier). |
| `/my/payment_method` | `GET` | page | logged-in user | yes | Render the payment method management page. |
| `/payment/transaction` | none | service | public | yes | Create a draft transaction and return its processing values. Inputs: amount, currency identifier, contact identifier, access token, and the allowed form parameters. Errors: forbidden on a bad access token; bad request on an unexpected parameter name; `You do not have access to this payment token.` |
| `/payment/confirmation` | `GET` | page | public | yes | Render the generic confirmation page. Inputs: transaction identifier, access token. |
| `/payment/archive_token` | none | service | logged-in user | yes | Archive one of the caller's own tokens. Input: token identifier. Output: nothing. |
| `/payment/status` | `GET` | page | public | yes | Render the payment status page for the transaction stored in the session. |
| `/payment/status/poll` | none | service | public | yes | Run the post-processing of the monitored transaction and return its provider code, state and landing route. May ask the caller to retry when the database refused the commit. |
| `/payment/custom/process` | `POST` | page | public | yes | Receive the browser of a customer who chose a custom payment mode (wire transfer, cash on delivery, pay on site). Input: the transaction reference, in the posted form data. Sets the transaction to pending without any amount validation and redirects to `/payment/status`. Cross-site request forgery protection is switched off on this path. Added by the Custom Payment Modes package. |

## 2.2 Endpoints added by the Accounting Payments package

| Path | Method | Kind | Authentication | Purpose |
|---|---|---|---|---|
| `/invoice/transaction/<invoice identifier>` | none | service | public | Create a transaction that pays one invoice. Verifies the document access token; answers `The access token is invalid.` on failure. Allows the extra parameter "next installment name". |
| `/invoice/transaction/overdue` | none | service | public | Create one transaction that pays every overdue invoice of the logged-in customer. Errors: `Please log in to pay your overdue invoices`; `Impossible to pay all the overdue invoices if they don't share the same currency.` |

## 2.3 Endpoints added by the Website Payment package

| Path | Method | Kind | Authentication | Purpose |
|---|---|---|---|---|
| `/donation/pay` | `GET` and `POST` | page | public | Render the donation payment page. On a post, store the amount, the currency, the donation options and the descriptions in the session and answer with a "see other" redirection to the same path. On a get, read the missing values back from the session, default the currency to the accounting currency of the active company, the amount to 25 and the donation options to a free custom amount, set the paying contact to the public contact and compute the access token for a visitor who is not signed in, then render the donation template. Not listed in the site map; listed in the site's content index under the name `Donation Payment`. |
| `/donation/transaction/<minimum amount>` | none | service | public | Create a donation transaction. Refuses an amount below the minimum with `Donation amount must be at least %.2f.`; refuses missing donor details with `Name is required.`, `Email is required.` and `Country is required.`. Allows four extra parameters beyond the generic ones: the donor comment, the recipient electronic mail address, the donor details and the reference prefix. Sends the internal donation notification and returns the processing values. |
| `/website_payment/snippet/supported_payment_methods` | `GET` | service | public | Return the payment methods that the current website advertises, as a list of names and image addresses. Optional input: a limit. Read-only. The answer is not cacheable for an internal user and cacheable for seven days, with one further day of stale reuse, for everybody else. |

## 2.4 Connector endpoints

Every connector adds between one and four endpoints, in these shapes:

| Shape | Typical path | Method | Purpose |
|---|---|---|---|
| Return from checkout | `/payment/<code>/return` | `GET` or `POST` | Receive the customer's browser after the provider's page; verify; process; redirect to the payment status page. |
| Webhook | `/payment/<code>/webhook` | `POST` | Receive an asynchronous notification; verify; process; acknowledge. |
| Inline payment | `/payment/<code>/payment` or `/payment/<code>/payments` | service | Called by the browser during an inline payment; makes the request to the provider and processes the answer. |
| Authorization return | `/payment/<code>/oauth/return` | `GET`, logged-in user | Receive the browser after the provider's authorization screen; exchange the authorization code for credentials; enable the provider. |

The exact path, method, authentication, inputs, verification and acknowledgement body of every connector endpoint are in `provider-connector-contracts.md`.

---

# 3. Screens

Screens are described as workflows on views. No client technology is implied.

## 3.1 Payment Provider

### 3.1.1 Card view (the default)

- One card per provider, coloured by `color`: blue when the package is not installed, yellow when disabled, orange in test mode, green when enabled.
- Each card shows the logo, the name, and, when the package is not installed, an **Install** button; when the package is additionally flagged as requiring a separate commercial licence (`module_to_buy` true), the button is replaced by an **Upgrade** link that opens the supplier's pricing page in a new window.
- Creation and quick creation are disabled.

### 3.1.2 List view

Columns: a drag handle bound to `sequence`, the name, the code (technical users only), the state, the available countries (hidden by default), the company (multi-company databases only). Creation is disabled.

### 3.1.3 Form view

- **Status ribbon**: `Disabled` in red when the package is installed and the state is `disabled`; `Test Mode` in orange when the state is `test`.
- **Button box**: one toggle button showing `Published` in green or `Unpublished` in red, bound to the publish operation; hidden while the package is not installed.
- **Header area**: the logo (editable only when the package is installed), the name, and, when the package is not installed, the **Install** button or the **Upgrade** link.
- **Permanent notice** while the record has no identifier: `Warning Creating a payment provider from the CREATE button is not supported. Please use the Duplicate action instead.`
- **Main group** (hidden unless the package is installed or absent): the code (technical users only, read-only once the record exists), the state as a radio group, the company (multi-company databases only).
- **Credentials page**: hidden when the code is `none`; filled by the connector with its own fields, the secret ones visible only to administrators, and with the connector's own buttons (Connect, Create webhook, Generate client key, Set account currency, Verify domain, Synchronise payment methods, Reset credentials).
- **Configuration page**, group "Payment Form": the supported payment methods as read-only tags (hidden while disabled), a link **Enable Payment Methods**, then `allow_tokenization`, `capture_manually` and `allow_express_checkout`, each shown only when the matching support flag says the connector implements it.
- **Configuration page**, group "Availability": `maximum_amount`; `available_currency_ids` as tags with the placeholder `Select currencies. Leave empty not to restrict any.`, visible to administrators only; `available_country_ids` as tags with the placeholder `Select countries. Leave empty to make available everywhere.`
- **Messages page**: `pre_msg`, `pending_msg`, `auth_msg` (only when manual capture is supported), `done_msg`, `cancel_msg`.

### 3.1.4 Search view

Filters: providers whose package is installed, grouping by code, grouping by state, grouping by company.

## 3.2 Payment Transaction

### 3.2.1 Form view

Read-only; neither creation nor editing is possible.

- **Status bar**: the state.
- **Header buttons**:
  - **Capture Transaction**, highlighted, visible only when the state is `authorized`;
  - **Void Transaction**, visible only when the state is `authorized`, with the confirmation question `Are you sure you want to void the authorized transaction? This action can't be undone.`;
  - **Post-process**, visible only to technical users and only while the transaction is not post-processed, with the tooltip `Run the post-processing step for this transaction.`
- **Button box**: **Refunds** with the count, visible only when there is at least one refund.
- **Left group**: reference, source transaction (only when set), amount, payment method, provider, company (multi-company databases only), provider reference, token (only when set), creation moment, last state change, production-environment flag, post-processed flag (technical users only).
- **Right group**: the contact, then the address block built from the five snapshot address fields, then the email address, the phone number and the language.
- **Child transactions**: a list, shown only when there is at least one.
- **Message group**: the state message, shown only when it is set.

### 3.2.2 List view

Columns: reference, creation moment, payment method, provider, contact, contact name, amount, state, company (multi-company databases only), production-environment flag (hidden by default). Creation disabled.

### 3.2.3 Card view

Reference and amount on the first line, contact name on the second. Creation disabled.

### 3.2.4 Search view

Search fields: reference, provider, contact, contact name. Filter: `Production Environment`. Groupings: provider, contact, state, production environment, company.

### 3.2.5 Analysis views

- Graph: number of transactions by creation month and state.
- Pivot: amount by creation month in rows and state in columns, with the record count shown.

## 3.3 Payment Method

- **Form view**: the image, the name, then three pages: **Providers** (the providers that support the method), **Brands** (the child methods, reorderable), **Configuration** (the code, the sequence, the four support flags, the supported countries and currencies). The configuration page is meant for technical users; the documentation warns that changing it only works within the real capabilities of the method and the provider.
- **List view** and **card view**: name, image and active flag.
- **Search view**: a filter that shows only the methods available for at least one provider that is not disabled.
- The screen action opens with archived methods visible, because every shipped method starts inactive.

## 3.4 Payment Token

- **Form view**: the display name, the contact, the provider, the payment method, the payment details, the provider reference, the active flag, and a button **Payments** that opens the transactions linked to the token.
- **List view**: contact, provider, payment details, active flag.
- **Search view**: a filter for archived tokens, groupings by provider, contact and company.
- **On the contact form**: a button showing `payment_token_count` that opens the token list filtered on that contact.

## 3.5 Payment Capture Wizard

A dialog. It shows: the authorized amount, the already captured amount, the already voided amount, the maximum capture allowed, the amount to capture (editable), and, only while the amount to capture is lower than the maximum, the checkbox that voids the rest. When at least one child transaction is still a draft, the dialog warns that a previous request has not been answered yet. When at least one transaction is handled by the Adyen connector, the dialog adds the connector's note that captures may also be started from the provider's own interface. Buttons: **Capture** and **Close**.

## 3.6 Payment Link Wizard

A dialog. It shows the amount (editable), the maximum amount, the already paid amount, the currency, the contact and the contact's email address, then the computed link with a copy-to-clipboard control, and the warning message when there is one. For an invoice it also shows the amount due, the open installments preview and the early payment discount information. For a sales order it shows the confirmation message that explains what paying this amount will do. Button: **Close**.

## 3.7 Payment Refund Wizard

A dialog opened from a Payment. It shows the payment amount, the already refunded amount, the maximum refund allowed and the amount to refund (editable), and warns when a refund is already pending. Button: **Refund**.

## 3.8 Customer portal

| Page | Contents |
|---|---|
| Portal home | A card **Payment methods** with the text `Manage your payment methods`, shown only when at least one compatible method allows tokenization or the customer already has tokens. |
| Payment page | Breadcrumb, blocking notices, amount and reference summary, payment form. |
| Payment method management page | Breadcrumb, payment form in validation mode with a delete control per token and the submit label `Save`. |
| Payment status page | Breadcrumb, state block with a spinning icon while processing, amount and reference summary, and a `Skip` link to the landing route. |
| Payment confirmation page | Breadcrumb, state block, summary with amount, reference, payment method (hidden for the placeholder method) and provider, and a `Go to my Account` button. |

## 3.9 The payment form

The payment form is the same fragment on every page that offers a payment. Its inputs are the payment context: reference prefix, amount, currency, contact, the compatible providers, the compatible methods, the available tokens, the availability report, the transaction route, the landing route and the access token. Its options are: the mode (`payment` or `validation`), whether tokens may be selected, whether tokens may be deleted, which token is preselected, the per-provider tokenization checkbox map, whether the submit button is shown, and its label.

Behaviour:

- Tokens are listed first under `Your payment methods`; the first one is preselected when selection is allowed.
- Payment methods are listed under `Payment method`, or under `Other payment methods` and collapsed behind a `Choose another method` button when tokens exist.
- A method with exactly one entry in the whole list is preselected when no token is preselected.
- A method with brands shows the brand logos next to it.
- Selecting a method reveals its inline form when the connector declares one.
- The submit button is disabled until a payment option is selected.
- Administrators see a small diagnostic control next to the heading that expands the availability report.

---

# 4. Notifications

| Notification | Channel | Recipients | Trigger |
|---|---|---|---|
| Transaction initiated | Message on the linked documents | The followers of those documents | A transaction, capture, void or refund is created. |
| Transaction state changed | Message on the linked documents | The followers of those documents | Every accepted state change, except for validation transactions and custom providers. |
| Payment posted | Message on the linked documents | The followers of those documents | A Payment is created by post-processing. |
| Capture requested | Message on the linked documents | The followers | The Adyen connector, when the provider acknowledges a capture request: `The capture request of <amount> for transaction <reference> has been sent.` |
| Void requested | Message on the linked documents | The followers | The Adyen connector, when the provider acknowledges a void request: `A request was sent to void the transaction <reference>.` |
| Capture or void failed on a source transaction | Message on the linked documents | The followers | The Adyen connector: `The capture of the transaction %s failed.` or `The void of the transaction %s failed.` The source transaction keeps its state in order that the operation can be retried. |
| Operation feedback | Transient notification in the screen | The user who ran the operation | Capture, void and refund return `Your payment operation has been successfully submitted.` or `Your payment operation could not be completed for following transactions: <references>`. |
| Webhook created | Transient notification | The administrator | `Your Razorpay webhook was successfully set up!`, `You Stripe Webhook was successfully set up!`, `Your Stripe Webhook is already set up.`, `You cannot create a Stripe Webhook if your Stripe Secret Key is not set.` |
| Domain verified | Transient notification | The administrator | `Your web domain was successfully verified.` |
| Payment methods synchronised | Transient notification | The administrator | `Successfully synchronized with Paymob` with `Payment methods have been successfully set up!`, or `Payment methods not found` with `Not all enabled payment methods were found on your account.` |
| Quotation payment succeeded | Email | The customer | Post-processing of a pending, authorized or confirmed transaction linked to a sales order that was not confirmed by it. Owned by [../sales/](../sales/README.md). |
| Invoice sent | Email | The customer | Automatic invoicing after a confirmed transaction, immediately or through the sending job. Owned by [../sales/](../sales/README.md). |
| Donation notification | Email | The recipient address configured on the donation block | Creation of a donation transaction, before the payment is attempted. Subject: `A donation has been made on your website`. Rendered in the language of the user whose electronic mail address matches the recipient, or in the language of the company contact when none matches. |
| Donation confirmation | Email | The donor | Post-processing of a confirmed transaction whose donation flag is set. Subject: `Donation confirmation`. Rendered in the language recorded in the contact snapshot of the transaction. |
| Donation details logged on the Payment | Log entry on the Payment | The followers of the Payment | Post-processing of a confirmed donation transaction: `Payment received from donation with following details:` followed by one line per filled value among the company, the contact, the contact name, the contact country and the contact electronic mail address. |
| Point of sale screen refresh | Live channel message | The point of sale screens of the configuration | An online point of sale payment was registered. The message carries only the order identifier, never any payment detail. |

---

# 5. Scheduled jobs

| Job | Frequency | Runs as | Active | Body |
|---|---|---|---|---|
| Payment: Post-process transactions | every 10 minutes | the system user | only while at least one provider is not disabled | Processes every transaction that is not post-processed and whose last state change is at most four days old, one at a time, committing after each and rolling back on a conflict. |
| Send invoices created by automatic invoicing | owned by [../sales/](../sales/README.md) | owned by [../sales/](../sales/README.md) | owned by [../sales/](../sales/README.md) | Woken by post-processing when asynchronous sending is on. |

---

# 6. Printed documents and exported files

This domain produces none. The provider's own receipt, when it exists, is produced by the provider. The payment receipt that an accountant may print belongs to [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md).

The single message template of this domain is the donation body, shared by the donation notification and the donation confirmation. It is rendered inside the light notification frame and contains: a heading, `Donation notification` for the internal notification and `Donation` for the donor's copy; for the donor's copy only, a salutation `Dear ` followed by the donor name and a paragraph thanking the donor for a donation of the amount, rendered as money in the transaction currency, made on the creation date, followed by `We appreciate your support for our organization as such.` and `Regards.`; and, in both copies, a detail table with the rows `Donor Name:`, `Donor Email:`, `Donation Date:`, `Amount(` followed by the currency symbol and `):`, `Comment:` (internal notification only, and only when a comment was given), `Payment Method:` carrying the provider code, and `Payment ID:` carrying the transaction reference.

The only report-like screen fragment is the **availability report**, which is not a printed document but a diagnostic block rendered inside the payment form for administrators. Its content is:

- the heading `Availability report`;
- the sub-heading `Payment providers`, then one line per provider with a green check or a red cross, the provider name, its identifier, and, when it is unavailable, `Reason: ` followed by the reason;
- the sub-heading `Payment methods`, then one line per method with the same shape, plus a line `Supported providers: ` listing each provider of the method in green when that provider is available and in red when it is not.
