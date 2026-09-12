# Provider connector contracts

One section per provider package shipped with the platform. Each section states the credentials the connector needs, the operations it implements, the service addresses it talks to in production and in test mode, the content of every outbound request, the content of every inbound notification, how the provider's status is mapped onto a transaction state, how amounts are formatted, which currencies and countries are restricted, and how the notification is authenticated.

A note on the literal addresses in this file. Every web address, host name, path segment and query parameter name printed in a fenced span below is a data value owned by the third-party provider, not a word of this specification. They are reproduced character for character, including the short forms those providers use in their own host names and paths, because a replacement that alters one character of them does not reach the provider and therefore does not reach behavioural equivalence. They are never abbreviations of this specification's own vocabulary: wherever such a literal is used in the surrounding prose, the prose spells the concept out in full words.

---

# 0. The connector contract

A connector is a set of behaviours attached to one provider code. Every connector implements a subset of the following contract; everything it does not implement keeps the generic behaviour.

## 0.1 Behaviours on the provider

| Behaviour | Generic answer | Purpose |
|---|---|---|
| Feature support | tokenization false, manual capture unsupported, express checkout false, refund `none` | Declares what the connector can do. |
| Supported currencies | every currency, archived ones included | Narrows `available_currency_ids`. |
| Default payment method codes | the empty set | The methods activated when the provider is put in service. |
| Build the request web address | the empty text | Turns an endpoint name into a full address, choosing between the production and the test service. |
| Build the request headers | no headers | Authentication, content type, idempotency key, signature. |
| Build the request basic authentication | none | For providers that authenticate with a user name and a password. |
| Parse the response content | the structured data body of the answer | For providers that wrap their answer or use another format. |
| Parse the response error | the raw text of the answer | Extracts the human-readable message put into `The payment provider rejected the request.` |
| Should build the inline form | true | Whether the payment happens without leaving the platform. |
| Redirect form template | the provider's `redirect_form_view_id` | May differ for a validation operation. |
| Validation amount | 0 | The amount charged to prove a payment method works. |
| Reset values | none | The credential fields cleared by the Reset credentials operation. |
| Removal values | code `none`, state `disabled`, unpublished, the four templates emptied | Applied when the package is uninstalled. |
| Guided setup | no action | The first-time connection flow. |
| Provider search domain | the code alone | Extended by the custom connector with the custom mode. |

## 0.2 Behaviours on the transaction

| Behaviour | Generic answer | Purpose |
|---|---|---|
| Compute the reference | the generic algorithm | Adapts the reference to the provider's constraints. |
| Specific processing values | none | Extra values handed to the browser. |
| Specific rendering values | none | The values the redirect template needs, including the provider page address. |
| Send a payment request | nothing | Charges a token. |
| Send a capture request | nothing | Captures an authorized amount. |
| Send a void request | nothing | Releases an authorized amount. |
| Send a refund request | nothing | Refunds a confirmed amount. |
| Search by reference | reference plus provider code | Matches a notification to a transaction. |
| Extract the reference | the field named `reference` | Where the reference sits in the payment data. |
| Extract the amount data | an empty structure, which fails the amount check | The amount, currency code and precision reported by the provider. |
| Apply the updates | nothing | Writes the provider reference, the payment method and the state. |
| Extract the token values | none | The create values of a token. |
| Create a mandate | none | Values a document supplies for an electronic mandate. |

## 0.3 Behaviours on the token

| Behaviour | Generic answer | Purpose |
|---|---|---|
| Specific create values | none | Connector fields set at creation. |
| Build the display name | the padded payment details | May be customised. |
| Handle archiving | nothing | Clean-up at the provider. |

## 0.4 Feature support matrix

| Provider code | Tokenization | Manual capture | Refund | Express checkout | Payment flow |
|---|---|---|---|---|---|
| `adyen` | yes | `partial` | `partial` | no | on the platform, with redirection when the instrument requires it |
| `aps` | no | unsupported | `none` | no | on the provider's page |
| `asiapay` | no | unsupported | `none` | no | on the provider's page |
| `authorize` | yes | `full_only` | `full_only` | no | on the platform |
| `buckaroo` | no | unsupported | `none` | no | on the provider's page |
| `custom` | no | unsupported | `none` | no | no online payment at all |
| `demo` | yes | `partial` | `partial` | yes | simulated |
| `dpo` | no | unsupported | `none` | no | on the provider's page |
| `ecpay` | no | unsupported | `none` | no | on the provider's page |
| `flutterwave` | yes | unsupported | `none` | no | on the provider's page |
| `iyzico` | no | unsupported | `none` | no | on the provider's page |
| `mercado_pago` | yes | unsupported | `none` | no | on the platform or on the provider's page |
| `mollie` | no | unsupported | `none` | no | on the provider's page |
| `nuvei` | no | unsupported | `none` | no | on the provider's page |
| `paymob` | no | unsupported | `none` | no | on the provider's page |
| `paypal` | no | unsupported | `none` | no | on the platform |
| `payu` | no | unsupported | `none` | no | on the provider's page |
| `razorpay` | yes | `full_only` | `partial` | no | on the platform |
| `redsys` | no | unsupported | `none` | no | on the provider's page |
| `stripe` | yes | `full_only` | `partial` | yes | on the platform |
| `toss_payments` | no | unsupported | `none` | no | on the platform |
| `worldline` | yes | unsupported | `none` | no | on the provider's page |
| `xendit` | yes | unsupported | `none` | no | on the platform for cards, on the provider's page otherwise |

A provider whose manual capture is "unsupported" can never carry a transaction in the `authorized` state (PAY-RULE-030), and its `capture_manually` switch is hidden on the form.

---

# 1. Adyen

**Credentials**: merchant account, service key, client key, keyed-hash key, account address prefix.
**Features**: tokenization; manual capture, full and partial; refunds, full and partial.
**Flow**: the customer stays on the platform; the connector may still redirect for a three-domain secure challenge or for an instrument that has its own page.

## 1.1 Service addresses

The address is built from the account prefix, the endpoint and its version:

```
production : https://<prefix>-checkout-live.adyenpayments.com/checkout/V<version>/<endpoint>
test       : https://<prefix>.adyen.com/checkout/V<version>/<endpoint>
```

The prefix stored on the provider is reduced, on every create and write, to the first two dash-separated words of whatever was pasted, with any leading scheme removed.

| Endpoint | Version | Used for |
|---|---|---|
| `/paymentMethods` | 71 | Listing the instruments available for the payment context. |
| `/payments` | 71 | Making a payment. |
| `/payments/details` | 71 | Submitting the result of an additional action. |
| `/payments/{}/cancels` | 71 | Voiding. |
| `/payments/{}/captures` | 71 | Capturing. |
| `/payments/{}/refunds` | 71 | Refunding. |

The placeholder of the last three is the **source transaction's** provider reference.

**Headers**: the service key in the provider's dedicated key header; on a request that may be retried, the idempotency key under `idempotency-key`.
**Error parsing**: the `message` field of the answer.

## 1.2 Outbound requests

### List the instruments

Sent by the browser through the connector's own endpoint. Body: the merchant account; the formatted amount; the country code of the contact, falling back to the company's country and then to the Netherlands; the language tag of the browsing context in the provider's format, falling back to United States English; the shopper reference, which is a constant platform prefix followed by an underscore and the contact identifier; the channel `Web`. The answer is returned to the browser with the country code added.

### Make a payment (inline flow)

Sent by the browser through the connector's own endpoint, after the connector verifies an access token over the reference, the converted amount, the currency identifier and the contact identifier; a mismatch answers `Received tampered payment request data.` Body:

| Field | Value |
|---|---|
| `merchantAccount` | the merchant account |
| `amount` | the amount in minor units and the currency code |
| `applicationInfo` | the platform name, its version and its integrator name |
| `countryCode` | contact country, else company country, else the Netherlands |
| `reference` | the transaction reference |
| `paymentMethod` | the instrument details collected by the provider's browser component |
| `shopperReference` | the constant platform prefix and the contact identifier |
| `recurringProcessingModel` | `CardOnFile` |
| `shopperIP` | the customer's internet protocol address |
| `shopperInteraction` | `Ecommerce` |
| `shopperEmail`, `shopperName`, `telephoneNumber` | from the transaction snapshot, the name split into a first and a last name |
| `storePaymentMethod` | the transaction's tokenize flag |
| `authenticationData` | native three-domain secure preferred |
| `channel`, `origin`, `browserInfo` | required for three-domain secure |
| `returnUrl` | the connector's return endpoint with the reference appended under the same key the provider uses for the merchant reference |
| `billingAddress`, `deliveryAddress` | the invoicing and delivery addresses of the linked sales order, when there is one; each address falls back to `Unknown` per missing part and to the country code `ZZ` when the country is unknown |
| `lineItems` | one item with the converted amount, quantity 1 and the reference as description |
| `captureDelayHours` | 0, **only when the provider does not use manual capture** |

The last field is essential: without it, a merchant account configured with a capture delay would send authorization events that the connector could not tell apart from immediate captures.

### Charge a token

Same body, with these differences: `paymentMethod` carries the token's provider reference under `storedPaymentMethodId`; `shopperReference` is the token's own shopper reference; `recurringProcessingModel` is `Subscription`; `shopperInteraction` is `ContAuth`; there is no `storePaymentMethod`, no authentication data and no browser data.

### Capture, void, refund

| Operation | Body |
|---|---|
| Capture | merchant account, amount in minor units with the currency code, and the child transaction's reference |
| Void | merchant account and the child transaction's reference |
| Refund | merchant account, the **negated** child amount in minor units with the currency code, and the child reference |

After a capture or a void whose answer status is `received`, a message is logged on the linked documents (`The capture request of <amount> for transaction <reference> has been sent.` or `A request was sent to void the transaction <reference>.`), and the child transaction's provider reference is replaced by the new provider reference of the answer, because the provider issues a distinct reference per operation. The same replacement happens for a refund when the answer carries a reference and the status `received`.

## 1.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/adyen/payment_methods` | service | List the instruments. |
| `/payment/adyen/payments` | service | Make a payment. |
| `/payment/adyen/payments/details` | service | Submit the result of an additional action. |
| `/payment/adyen/return` | page, no new session | Return after a three-domain secure page; forces the operation to `online_redirect`, then submits the redirect result through the details endpoint, then redirects to the payment status page. |
| `/payment/adyen/notification` | `POST` | The webhook. |

The webhook body carries a list of notification items. For each item the connector matches the transaction, verifies the signature, then rewrites the item into the shape of a payment answer according to the event code:

| Event code | Success flag | Rewritten result code |
|---|---|---|
| `AUTHORISATION` | true | `Authorised` |
| `AUTHORISATION` | false | the item is skipped |
| `CANCELLATION` | true | `Cancelled` |
| `CANCELLATION` | false | `Error` |
| `REFUND` or `CAPTURE` | true | `Authorised` |
| `REFUND` or `CAPTURE` | false | `Error` |
| `CAPTURE_FAILED` | true | `Error` |
| any other | any | the item is skipped |

Acknowledgement body: the literal bracketed word `[accepted]`.

## 1.4 Signature

Shape 1. The signed values, in order: `pspReference`, `originalReference`, `merchantAccountCode`, `merchantReference`, the amount value, the amount currency, `eventCode`, `success`. Each value is escaped by prefixing every backslash and every colon with a backslash, and an absent value becomes the empty text. The values are joined with a colon. The keyed-hash key is read as hexadecimal into bytes, the keyed hash uses the secure hash algorithm, 256-bit variant, and the result is encoded in base 64. The received signature sits in the `hmacSignature` field of the additional data.

## 1.5 Status mapping

| Transaction state | Provider result codes |
|---|---|
| pending | `ChallengeShopper`, `IdentifyShopper`, `Pending`, `PresentToShopper`, `Received`, `RedirectShopper` |
| confirmed | `Authorised` |
| canceled | `Cancelled` |
| error | `Error` |
| refused (treated as error, with its own message) | `Refused` |

Rules applied on top of the mapping:

- `Authorised` with manual capture off confirms the transaction. With manual capture on it authorizes the transaction when the event code is `AUTHORISATION`, and confirms it otherwise (that is, on a capture event).
- A confirmed refund immediately wakes the post-processing job.
- `Error` on an authorization or a refund sets the transaction to `error` with `An error occurred during the processing of your payment. Please try again.`
- `Error` on a cancellation sets a **child** transaction to `error` with `The void of the transaction %s failed.`; on a **source** transaction it only logs that message and leaves the state alone in order that the void can be retried.
- `Error` on a capture or a capture failure behaves the same way with `The capture of the transaction %s failed.`
- `Refused` sets the transaction to `error` with `Your payment was refused. Please try again.`
- An unknown result code sets the transaction to `error` with the connector prefix followed by `Received data with invalid payment state: %s`.
- A missing result code sets the transaction to `error` with `Received data with missing payment state.`

The provider reference is written only for the event codes `AUTHORISATION` and `REFUND`, because a capture or a cancellation carries its own distinct reference.

## 1.6 Payment method resolution

The instrument is read from `paymentMethod`. When that field is a structure, the type is taken, and for a card (`scheme`) the brand is taken instead. When it is plain text, as in a webhook, it is used directly. The value is translated through the connector's mapping and looked up; a miss keeps the current method. The mapping is: `ach_direct_debit`→`ach`, `apple_pay`→`applepay`, `bacs_direct_debit`→`directdebit_GB`, `bancontact_card`→`bcmc`, `bancontact`→`bcmc_mobile`, `cash_app_pay`→`cashapp`, `gopay`→`gopay_wallet`, `becs_direct_debit`→`au_becs_debit`, `afterpay`→`afterpaytouch`, `klarna_pay_over_time`→`klarna_account`, `momo`→`momo_wallet`, `napas_card`→`momo_atm`, `paytrail`→`ebanking_FI`, `online_banking_czech_republic`→`onlineBanking_CZ`, `online_banking_india`→`onlinebanking_IN`, `fpx`→`molpay_ebanking_fpx_MY`, `p24`→`onlineBanking_PL`, `mastercard`→`mc`, `online_banking_slovakia`→`onlineBanking_SK`, `online_banking_thailand`→`molpay_ebanking_TH`, `open_banking`→`paybybank`, `samsung_pay`→`samsungpay`, `sepa_direct_debit`→`sepadirectdebit`, `sofort`→`directEbanking`, `unionpay`→`cup`, `wallets_india`→`wallet_IN`, `wechat_pay`→`wechatpayQR`.

## 1.7 Amounts

Minor units, with these deviations from the payment precision table: Chilean peso 2, Cape Verdean escudo 0, Indonesian rupiah 0, Icelandic króna 2.

## 1.8 Amount validation

Skipped when the answer asks for a redirection or a three-domain secure action, and when the result code is `Refused`; in both cases the transaction ends up pending or in error anyway. Otherwise the amount is read from the `amount` structure and converted back to major units with the connector's own precision.

## 1.9 Token values

Extracted from the additional data: the recurring detail reference becomes the provider reference, the card summary becomes the payment details, and the recurring shopper reference is stored on the token. Nothing is extracted when the recurring detail reference is absent.

## 1.10 Restrictions

No currency restriction and no country restriction beyond what the administrator sets.

---

# 2. Amazon Payment Services

**Credentials**: merchant identifier, access code, request hash phrase, response hash phrase.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 2.1 Service addresses

```
production : https://checkout.payfort.com/FortAPI/paymentPage
test       : https://sbcheckout.payfort.com/FortAPI/paymentPage
```

## 2.2 Outbound request

The customer's browser posts a form to the address above with:

| Field | Value |
|---|---|
| `command` | `PURCHASE` |
| `access_code` | the access code |
| `merchant_identifier` | the merchant identifier |
| `merchant_reference` | the transaction reference |
| `amount` | the amount in minor units, as text |
| `currency` | the currency code |
| `language` | the first two characters of the contact's language |
| `customer_email` | the normalised email address of the contact |
| `return_url` | the connector's return endpoint |
| `payment_option` | the payment method code in upper case, **omitted** when the method is the generic card method in order that the customer picks the brand on the provider's page |
| `signature` | computed over all the fields above |

## 2.3 Inbound

| Endpoint | Method | Session | Purpose |
|---|---|---|---|
| `/payment/aps/return` | `POST` | no new session | Return from checkout; verify; process; redirect to the payment status page. |
| `/payment/aps/webhook` | `POST` | yes | Webhook; verify; process; acknowledge with an empty body. |

The reference is read from `merchant_reference`.

## 2.4 Signature

Shape 2. Concatenate `key=value` for every field except the signature, with the keys sorted; wrap the result with the phrase on both sides; hash with the secure hash algorithm, 256-bit variant; render as hexadecimal. The response phrase is used for incoming data, the request phrase for outgoing data. The received signature sits in the `signature` field.

## 2.5 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `19` |
| confirmed | `14` |

A missing status gives `Received data with missing payment state.`; any other status gives `Received invalid transaction status %(status)s and reason '%(reason)s'.` with the provider's response message.

The provider reference is the `fort_id` field. The instrument is read from `payment_option`, lower-cased, and looked up without any mapping.

## 2.6 Amounts and restrictions

Minor units with the payment precision table. No currency or country restriction. References are always built from the time-based prefix in order that they contain only letters, digits, `-` and `_`.

---

# 3. AsiaPay

**Credentials**: brand, merchant identifier, secure hash secret, secure hash function.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 3.1 Service addresses

| Brand | Production | Test |
|---|---|---|
| `paydollar` | `https://www.paydollar.com/b2c2/eng/payment/payForm.jsp` | `https://test.paydollar.com/b2cDemo/eng/payment/payForm.jsp` |
| `pesopay` | `https://www.pesopay.com/b2c2/eng/payment/payForm.jsp` | `https://test.pesopay.com/b2cDemo/eng/payment/payForm.jsp` |
| `siampay` | `https://www.siampay.com/b2c2/eng/payment/payForm.jsp` | `https://test.siampay.com/b2cDemo/eng/payment/payForm.jsp` |
| `bimopay` | `https://www.bimopay.com/b2c2/eng/payment/payForm.jsp` | falls back to the `paydollar` test address |

The address is chosen by the brand; an unknown brand falls back to `paydollar`.

## 3.2 Outbound request

| Field | Value |
|---|---|
| `merchant_id` | the merchant identifier |
| `amount` | the amount in major units |
| `reference` | the transaction reference |
| `currency_code` | the provider's numeric code for the single currency allowed on the account |
| `mps_mode` | `SCP` |
| `return_url` | the connector's return endpoint |
| `payment_type` | `N` |
| `language` | the provider's language letter resolved from the browsing language |
| `payment_method` | the provider's instrument code, or `ALL` |
| `secure_hash` | computed over the five signed fields |

The currency code table maps twenty-two currency codes onto the provider's numeric codes: United Arab Emirates dirham 784, Australian dollar 036, Brunei dollar 096, Canadian dollar 124, Chinese yuan 156, euro 978, pound sterling 826, Hong Kong dollar 344, Indonesian rupiah 360, Indian rupee 356, Japanese yen 392, South Korean won 410, Macanese pataca 446, Malaysian ringgit 458, New Zealand dollar 554, Philippine peso 608, Saudi riyal 682, Singapore dollar 702, Thai baht 764, new Taiwan dollar 901, United States dollar 840, Vietnamese dong 704.

The language table maps: English `E`; Hong Kong and Taiwan Chinese `C`; mainland Chinese `X`; Japanese `J`; Thai `T`; French `F`; German `G`; Russian `R`; Spanish and Vietnamese `S`.

## 3.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/asiapay/return` | `GET` | Returns the browser to the payment status page and processes **nothing**, because the returned data carry no useful information and the provider offers no way to fetch it. |
| `/payment/asiapay/webhook` | `POST` | Verify; process; acknowledge with the two-letter text that spells the word "okay" in capitals. |

The reference is read from `Ref`.

## 3.4 Signature

Shape 1. Outgoing values in order: `merchant_id`, `reference`, `currency_code`, `amount`, `payment_type`. Incoming values in order: `src`, `prc`, `successcode`, `Ref`, `PayRef`, `Cur`, `Amt`, `payerAuth`. The shared secret is appended as the last element, the elements are joined with a vertical bar, and the configured hash function is applied. The received signature sits in `secureHash`.

## 3.5 Status mapping

| Transaction state | Provider success codes |
|---|---|
| confirmed | `0` |
| error | `1` |

A missing success code raises `Received data with missing success code.` Code `1` gives `An error occurred during the processing of your payment (success code %(success_code)s; primary response code %(response_code)s). Please try again.`, and any other code gives `Unknown success code: %s`.

The provider reference is `PayRef`. The instrument is read from `payMethod` and translated through a fifty-entry mapping onto the platform's method codes.

## 3.6 Amounts and restrictions

The amount is sent in major units. The amount reported back is read from `Amt` and paired with the single currency configured on the provider. Exactly one currency may be selected per account (PAY-RULE-012), and it must be one of the twenty-two listed above. References are limited to 35 characters.

---

# 4. Authorize.Net

**Credentials**: login identifier, transaction key, signature key, client key.
**Features**: tokenization; manual capture, full only; refunds, full only.
**Flow**: on the platform.

## 4.1 Service addresses

| Provider state | Service address |
|---|---|
| `enabled` | `https://api.authorize.net/xml/v1/request.api` |
| `test` | `https://apitest.authorize.net/xml/v1/request.api` |

Every request is a structured data body whose single top-level key is the operation name and which always carries the merchant authentication (login identifier and transaction key). The request timeout is sixty seconds. When the answer carries a result code of `Error`, the connector builds a failure structure with the first message code and text, extended by the texts of the transaction errors when there are any.

## 4.2 Operations

| Operation name | Purpose |
|---|---|
| `createTransactionRequest` with type `authOnlyTransaction` | Authorize without capturing. |
| `createTransactionRequest` with type `authCaptureTransaction` | Authorize and capture in one step. |
| `createTransactionRequest` with type `priorAuthCaptureTransaction` | Capture a previous authorization. |
| `createTransactionRequest` with type `voidTransaction` | Void a previous authorization. |
| `createTransactionRequest` with type `refundTransaction` | Refund a settled payment. |
| `createCustomerProfileFromTransactionRequest` | Create the customer and payment profiles from an authorized transaction. |
| `getCustomerPaymentProfileRequest` | Read the masked instrument details of a payment profile. |
| `deleteCustomerProfileRequest` | Delete a customer profile. |
| `getTransactionDetailsRequest` | Read the state and the instrument of a payment. |
| `getMerchantDetailsRequest` | Read the account currencies and the public client key. |
| `authenticateTestRequest` | Check the credentials. |

The authorization body carries: the transaction type, the amount as text, either the obfuscated instrument data collected by the provider's browser component or the profile and payment profile identifiers of the token, an order block with the reference truncated to 20 characters as the invoice number and to 255 characters as the description, the customer's email address, the customer's internet protocol address, and, only when no token is used, a billing block with the first name, last name, company name, address, city, state, postal code and country, each truncated to the length the provider allows (50, 50, 50, 60, 40, 40, 20, 60). For a contact that is a company, the whole name is used as the last name and as the company name, and the first name is empty.

The refund body carries the transaction type, the amount as text, the reference of the original payment, and a payment block rebuilt from the details read on that payment: the routing number, account number and account holder name for a bank account, or the masked card number and the literal text `XXXX` as the expiry date for a card.

## 4.3 Inbound

| Endpoint | Kind | Purpose |
|---|---|---|
| `/payment/authorize/payment` | service | Make the payment with the obfuscated instrument data. It verifies an access token over the reference and the contact identifier (`Received tampered payment request data.`), reads the transaction (`Transaction not found.`), locks the transaction row in order that a concurrent job cannot make the platform consume the single-use instrument data twice, sends the request and processes the answer. |

There is no webhook. Everything is learned from the answers of the requests the platform makes.

## 4.4 Status mapping

The answer is reduced to a response code, a transaction identifier, an operation kind and an account type.

| Response code | Meaning | Effect |
|---|---|---|
| `1` | approved | depends on the operation kind, see below |
| `2` | declined | the transaction is canceled with the provider's reason text as the state message |
| `4` | held for review | the transaction becomes pending |
| `3` or anything else | error | `Received data with status code "%(status)s" and error code "%(error)s".` |

| Operation kind | Effect of an approval |
|---|---|
| `auth_capture`, `prior_auth_capture` | confirmed |
| `auth_only` | authorized; then, when the transaction asks for a token, the token is created **before** anything else, and, for a validation transaction, the authorization is voided immediately |
| `void` | confirmed when the operation is a validation or a refund (the money did come back, or the validation succeeded), otherwise canceled, with `done` allowed as an extra source state |
| `refund` on a refund transaction | confirmed, and the post-processing job is woken at once |

## 4.5 Refund decision

Before refunding, the connector reads the payment's details at the provider and looks at its status:

| Provider status | Effect |
|---|---|
| `voided` | the refund transaction is canceled with `done` allowed as an extra source state, because the money never left |
| `refundPendingSettlement`, `refundSettledSuccessfully` | the refund transaction is confirmed and the post-processing job is woken, because the provider already refunded it |
| `authorizedPendingCapture`, `capturedPendingSettlement` | the payment is **voided** instead of refunded, because it is not settled yet |
| `settledSuccessfully` | the payment is refunded for the rounded amount |
| anything else | `The transaction is not in a status to be refunded. (status: %(status)s, details: %(message)s)` |

When the details cannot be read at all: `Could not retrieve the transaction details. (error code: %(error_code)s; error_details: %(error_message)s)`.

## 4.6 Token values

The connector creates a customer profile from the authorized payment, with a merchant customer identifier built from a constant platform prefix, the contact identifier and eight random hexadecimal characters, truncated to 20 characters, plus the contact's email address. It then reads the payment profile to obtain the last four characters of the card number or of the bank account number. The token stores the payment profile identifier as its provider reference, the customer profile identifier in its own field, and those four characters as the payment details. Nothing is created when the transaction already has a token or when the profile could not be created.

## 4.7 Amounts, validation and restrictions

Amounts are sent in major units as text. The amount reported back is read by fetching the payment's details and taking the authorized amount; the currency is the single currency configured on the account. When the details cannot be read, the amount check is skipped. Exactly one currency may be selected per account (PAY-RULE-012), because the provider requires one account per currency. The validation amount is 0.01.

---

# 5. Buckaroo

**Credentials**: website key, secret key.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 5.1 Service addresses

| Provider state | Service address |
|---|---|
| `enabled` | `https://checkout.buckaroo.nl/html/` |
| `test` | `https://testcheckout.buckaroo.nl/html/` |

## 5.2 Outbound request

| Field | Value |
|---|---|
| `Brq_websitekey` | the website key |
| `Brq_amount` | the amount in major units |
| `Brq_currency` | the currency code |
| `Brq_invoicenumber` | the transaction reference |
| `Brq_return`, `Brq_returncancel`, `Brq_returnerror`, `Brq_returnreject` | all four set to the connector's return endpoint; all four are included even though they are equal, because they take part in the signature |
| `Brq_culture` | the contact's language with the underscore replaced by a hyphen, only when the contact has a language |
| `Brq_signature` | computed over the fields above |

## 5.3 Inbound

| Endpoint | Method | Session | Purpose |
|---|---|---|---|
| `/payment/buckaroo/return` | `POST` | no new session | Return from checkout. |
| `/payment/buckaroo/webhook` | `POST` | yes | Webhook; acknowledges with an empty body. |

Both lower-case every key of the received data before anything else, because the provider's parameter names are case-insensitive. The reference is read from `brq_invoicenumber`.

## 5.4 Signature

Shape 2, with a filtering step. For incoming data every value is first decoded from its address encoding and the signature field is excluded. Only the keys that begin with `add_`, `brq_` or `cust_`, in any case, take part. They are sorted by their lower-cased key, because the underscore sorts between the upper-case and the lower-case letters and a naive sort would differ. The signing string is the concatenation of `key=value` with an empty value rendered as the empty text, followed by the secret key. The hash is the secure hash algorithm, 160-bit variant, rendered as hexadecimal.

## 5.5 Status mapping

| Transaction state | Provider status codes |
|---|---|
| pending | 790, 791, 792, 793 |
| confirmed | 190 |
| canceled | 890, 891 |
| refused | 690 |
| error | 490, 491, 492 |

A refusal gives `Your payment was refused (code %s). Please try again.`; an error gives `An error occurred during processing of your payment (code %s). Please try again.`; an unknown code gives `Unknown status code: %s.`

The provider reference is the first entry of the comma-separated `brq_transactions` field; a missing field gives `Received data with missing transaction keys`. The instrument is read from `brq_payment_method` and translated through a twenty-two entry mapping.

## 5.6 Amounts and restrictions

Major units. Supported currencies: euro, pound sterling, Polish złoty, Danish krone, Norwegian krone, Swedish krona, Swiss franc, United States dollar.

---

# 6. Custom Payment Modes

**Credentials**: none.
**Features**: none. There is no online payment at all.
**Custom modes**: wire transfer; cash on delivery (added by the Delivery package); pay on site (added by the Click and Collect package).

## 6.1 Flow

The rendering values contain only the platform's own processing address, which is the fixed path `/payment/custom/process`, and the transaction reference. That path is served by this domain itself, accepts the method `POST` only, is open to public visitors and carries no cross-site request forgery protection, because the form is submitted by the customer's browser from the payment form of the platform. The browser posts to that address, and the connector:

1. skips the amount validation entirely;
2. sets the transaction to pending and writes the log entry `Validated custom payment for transaction <reference>: set as pending.`;
3. redirects the browser to the payment status page.

## 6.2 Messages

The "received" message is never logged for a custom provider. The "sent" message is replaced by `The customer has selected <provider name> to make the payment.`

## 6.3 The communication given to the customer

```formula
communication = the payment reference of the first linked invoice, when there is one
              = the reference of the first linked sales order, when there is one
              = the transaction reference, otherwise
```

## 6.4 Wire transfer specifics

- When a wire transfer provider is created, its pending message is emptied, in order that the bank account details can be filled in.
- The Recompute pending message operation rebuilds it as a rich-text block with the heading `Please use the following transfer details`, the sub-heading `Bank Account` or `Bank Accounts`, and a bullet list of the display names of the bank accounts of every bank journal of the company. It only runs when the Accounting Payments package is installed.
- A separate internal step fills that message for every wire transfer provider that still has none.
- The switch `qr_code` offers the customer a machine-readable payment code; the payload is built by [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md).
- The provider search domain is extended with the custom mode, in order that the setup and removal steps of the three custom packages do not collide.
- The removal values additionally empty `custom_mode`.
- No accounting payment method and no outstanding account are created for the code `custom`.

---

# 7. Demo

**Credentials**: none.
**Features**: tokenization; manual capture, full and partial; refunds, full and partial; express checkout.
**State**: a demo provider may only be in `test` or `disabled` (PAY-RULE-013).

The connector simulates a provider without any network call. Every request it "sends" immediately feeds a payment structure back into the processing step.

| Simulated request | Payment data fed back |
|---|---|
| Charge a token | the reference and the token's simulated state |
| Capture | the reference, the simulated state `done`, and a flag that marks the event as a manual capture |
| Void | the reference and the simulated state `cancel` |
| Refund | the reference and the simulated state `done` |
| The customer's own choice on the payment form | the reference and the chosen simulated state |
| The three operations on the transaction form | the reference and the state `done`, `cancel` or `error` |

Endpoint: `/payment/demo/simulate_payment`, a service, which simply processes the structure it is given.

**Updates**: the provider reference becomes the four letters that spell the word "demo", a hyphen and the transaction reference. When the transaction asks for a token, the token is created **before** the state is set, in order that the simulated state and the payment details are stored on it. Then:

| Simulated state | Effect |
|---|---|
| `pending` | pending |
| `done` with manual capture on, no manual-capture flag, and an operation other than `refund` | authorized |
| `done` otherwise | confirmed; for a refund, the post-processing job is woken at once |
| `cancel` | canceled |
| anything else | error, with `You selected the following demo payment status: %s` |

**Amount validation**: always skipped.
**Token values**: the payment details supplied by the form, the provider reference is the literal text `fake provider reference`, and the simulated state is stored on the token in order that every later charge of that token ends in the same state. Nothing is extracted when the transaction is already confirmed or authorized.
**Token display name**: built without padding.
**Extra guard**: the portal transaction creation refuses a demo provider that is not among the compatible providers with `Provider %s is not properly configured.`

---

# 8. DPO

**Credentials**: service reference, company token.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 8.1 Service address

| Provider state | Service address |
|---|---|
| `enabled` and `test` | `https://secure.3gdirectpay.com/API/v6/` |

The requests and the answers are tagged markup documents, not structured data; the connector therefore sets the content type accordingly and parses the answer by reading the text of every direct child element of the root into a flat structure.

## 8.2 Outbound requests

**Create a transaction token.** The document carries the company token, the request name `createToken`, a transaction block (payment amount, payment currency, the reference as the company reference, the return address twice, and the customer's email address, first name, last name, city, country code and postal code) and a services block with the service reference, the transaction reference as the description, and the transaction's creation moment formatted as year, month, day, hour and minute. The answer's transaction token becomes the value of the single query parameter of the address the customer is redirected to:

| Element | Value |
|---|---|
| Redirect address, without the query part | `https://secure.3gdirectpay.com/payv2.php` |
| Name of the single query parameter | `ID`, which is this provider's own two-letter spelling of the word identifier and is sent exactly as written here |
| Value of that parameter | the transaction token returned by the create-token answer |
| Resulting address, with the parameter name spelled as the provider spells its own word for identifier | `https://secure.3gdirectpay.com/payv2.php?ID=<transaction token>` |
 A failure sets the transaction to `error` with the provider's message.

**Verify a token.** The document carries the company token, the request name `verifyToken` and the transaction token received on return. The answer is merged into the returned data before processing.

## 8.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/dpo/return` | `GET` | Return from checkout: match the transaction, verify the data by asking the provider, merge the answer into the returned data, process, redirect to the payment status page. A verification failure logs `Unable to verify the payment data.` and nothing is processed. |

There is no webhook and no signature: the authenticity comes from the verification call.

The reference is read from `CompanyRef`; the amount from `TransactionAmount` and `TransactionCurrency`; the provider reference from `TransID`.

## 8.4 Status mapping

| Transaction state | Provider result codes |
|---|---|
| pending | `003`, `007` |
| authorized (treated as confirmed) | `001`, `005` |
| confirmed | `000`, `002` |
| canceled | `900`, `901`, `902`, `903`, `904`, `950` |
| error | `801`, `802`, `803`, `804` |

The authorized codes and the confirmed codes are both mapped to the confirmed state, because the connector does not implement manual capture. An error gives `An error occurred during processing of your payment (code %(code)s: %(explanation)s). Please try again.`; an unknown code gives `Unknown status code: %s`.

## 8.5 Amounts and restrictions

Major units. No currency or country restriction.

---

# 9. ECPay

**Credentials**: merchant identifier, hash key, hash initialisation vector.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 9.1 Service addresses

```
production : https://payment.ecpay.com.tw/Cashier/AioCheckOut/V5
test       : https://payment-stage.ecpay.com.tw/Cashier/AioCheckOut/V5
```

## 9.2 Outbound request

| Field | Value |
|---|---|
| `MerchantID` | the merchant identifier |
| `MerchantTradeNo` | the transaction reference |
| `MerchantTradeDate` | the transaction's creation moment converted to the Taipei time zone and formatted as year/month/day hour:minute:second |
| `PaymentType` | `aio` |
| `TotalAmount` | the amount truncated to an integer |
| `TradeDesc` | a fixed description naming the platform |
| `ItemName` | the transaction reference |
| `ReturnURL` | the connector's webhook endpoint |
| `ChoosePayment` | `ALL` |
| `EncryptType` | `1` |
| `ClientBackURL`, `OrderResultURL` | the connector's return endpoint |
| `IgnorePayment` | every provider instrument code **except** those mapped to the chosen payment method, joined by `#` |
| `Language` | the provider's language code, omitted entirely when the browsing language resolves to none, because the provider then defaults to Taiwanese Chinese |
| `CheckMacValue` | computed over all the fields above |

The instrument mapping is: card → `Credit`; WeChat Pay → `WeiXin`; convenience store → `CVS` and `BARCODE`; bank transfer → `ATM` and `WebATM`; mobile wallet → `DigitalPayment`; Taiwan quick response payment → `TWQR`. Two further provider codes, buy-now-pay-later and the Apple wallet, are listed under a null platform code purely which makes them always part of the ignored set.

The language table maps English to `ENG`, Japanese to `JPN`, Korean to `KOR` and mainland Chinese to `CHI`; Taiwanese Chinese is deliberately absent.

## 9.3 Inbound

| Endpoint | Method | Session | Purpose |
|---|---|---|---|
| `/payment/ecpay/return` | `GET` and `POST` | no new session | Return from checkout. A plain `GET` with no payload happens when the customer clicks "return to the merchant" during a convenience-store or barcode payment; it is **not** a failure and nothing is processed. A `POST` with a payload is verified and processed. |
| `/payment/ecpay/webhook` | `POST` | no new session | Webhook. A notification whose `SimulatePaid` field is not `0` is a provider-side simulation and is skipped with a log entry. Acknowledgement body: the digit one, a vertical bar, and the two letters that spell "okay" in capitals. |

The reference is read from `MerchantTradeNo`.

## 9.4 Signature

Shape 2 with an encoding step. Sort the fields by their lower-cased key; build the text `HashKey=<key>&` followed by `field=value&` for every field followed by `HashIV=<vector>`; encode the whole text for use in a web address, treating the characters `-`, `_`, `.`, `!`, `*`, `(` and `)` as safe; lower-case it; hash it with the secure hash algorithm, 256-bit variant; upper-case the hexadecimal result. The received signature sits in `CheckMacValue` and is removed from the data before the expected signature is computed.

## 9.5 Status mapping

| Transaction state | Provider return codes |
|---|---|
| confirmed | `1`, `2`, `10100073` |

A missing return code gives `Received data with missing return code.`; any other code gives `An error occurred (return code %(return_code)s; return message %(return_message)s).`

The provider reference is `TradeNo`. The instrument is read from `PaymentType` and translated through a response mapping that only covers the values which identify a brand: convenience store chains and one wallet.

## 9.6 Amounts and restrictions

The amount is sent as an integer. The amount reported back is read from `TradeAmt` and paired with the transaction's own currency. Only the new Taiwan dollar is supported (PAY-RULE-012). References are at most 20 characters, purely alphanumeric, built with an empty separator.

---

# 10. Flutterwave

**Credentials**: public key, secret key, webhook secret.
**Features**: tokenization.
**Flow**: on the provider's page.

## 10.1 Service address

A single address for both modes: `https://api.flutterwave.com/v3/`. The secret key is sent as a bearer credential. The answer's `data` element is the parsed content; the `message` field is the error text.

## 10.2 Outbound requests

**Create a payment link.** Body: the transaction reference; the amount in major units; the currency code; the connector's return endpoint; a customer block with the email address, the name and the phone number reduced to digits after replacing a leading plus sign with a double zero; a customisation block with the company name and the company logo address; and the provider's instrument code for the chosen method. The answer's link is the address the customer is redirected to. A failure sets the transaction to `error`.

**Charge a token.** Body: the token's provider reference; the email address stored on the token; the amount; the currency code; the company's country code; the transaction reference; the first and last name; the customer's internet protocol address; and the connector's authorization return endpoint. A failure sets the transaction to `error`.

**Verify a payment.** A read of `transactions/verify_by_reference` with the transaction reference as a query parameter. Used on return from checkout.

## 10.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/flutterwave/return` | `GET` | Return from checkout. When the returned status is `cancelled` nothing is processed, because the provider sends no payment identifier in that case. Otherwise the authoritative data are fetched and processed. |
| `/payment/flutterwave/auth_return` | `GET` | Return after an authorization step; the answer is a serialised structure that is unpacked and handed to the return handler. |
| `/payment/flutterwave/webhook` | `POST` | Webhook. Only the event `charge.completed` is handled; the payload's `data` element is processed. Acknowledgement body: empty. |

The reference is read from `tx_ref`, falling back to `txRef`.

## 10.4 Verification

Shape 3: the provider sends the stored webhook secret in a header, and the connector compares it with a constant-time comparison.

## 10.5 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `pending`, `pending auth` |
| confirmed | `successful` |
| canceled | `cancelled` |
| error | `failed` |

An error gives `An error occurred during the processing of your payment (status %s). Please try again.`; an unknown status gives `Unknown payment status: %s`.

The provider reference is the payment identifier. **A pending token payment that carries an authorization address stores that address in the provider reference instead**, in order that the browser can be redirected to it; the real reference is written back when the transaction leaves the pending state. A token transaction that is pending, whose provider reference looks like a web address, is detected as "awaiting authorization" and is answered with a rendered redirect form pointing at that address.

The instrument is read from `payment_type`; for a card the card type is used instead, lower-cased; the only mapping entry is bank transfer.

## 10.6 Token values

Extracted from the card block: the last four digits as the payment details, the card token as the provider reference, and the customer's email address stored on the token. Nothing is extracted when the card block carries no token.

## 10.7 Amounts and restrictions

Major units. Supported currencies: pound sterling, Canadian dollar, Central African franc, Chilean peso, Colombian peso, Egyptian pound, euro, Ghanaian cedi, Guinean franc, Kenyan shilling, Malawian kwacha, Moroccan dirham, Nigerian naira, Rwandan franc, Sierra Leonean leone, São Tomé and Príncipe dobra, South African rand, Tanzanian shilling, Ugandan shilling, United States dollar, West African franc, Zambian kwacha. The provider is removed from the compatible providers for a validation operation (PAY-RULE-085). References are singularized in order that they are unique per merchant account.

---

# 11. Iyzico

**Credentials**: key identifier, key secret.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 11.1 Service addresses

| Provider state | Service address |
|---|---|
| `enabled` | `https://api.iyzipay.com` |
| `test` | `https://sandbox-api.iyzipay.com` |

The answer is accepted only when its `status` field says success; otherwise the connector raises `The payment provider rejected the request.` followed by the provider's error message. The error text is read from `errorMessage`.

## 11.2 Outbound requests

**Initialise a checkout form** at `payment/iyzipos/checkoutform/initialize/auth/ecom`. Body: a single basket item with the transaction identifier, the amount, a fixed name, the category `Service` and the item type `VIRTUAL`; a billing address with the snapshot address, the contact name, the city and the country name; a buyer block with the contact identifier, the first and last name, an identity number built by padding the contact identifier to five digits, the email address, the registration address, the city, the country name and the literal internet protocol address `0`; the callback address, which is the connector's return endpoint with the transaction reference as a query parameter; the transaction reference as the basket identifier; the currency code; the locale, Turkish when the browsing language is Turkish and English otherwise; the amount twice (as the paid price and as the price); and a fixed source name identifying the platform.

The answer's payment page address is split into its address and its query parameters, and both are handed to the redirect template in order that the parameters survive the redirection.

**Read the payment details** at `payment/iyzipos/checkoutform/auth/ecom/detail`, with the locale and the checkout token. Used by both inbound endpoints.

## 11.3 Inbound

| Endpoint | Method | Session | Purpose |
|---|---|---|---|
| `/payment/iyzico/return` | `POST` | no new session | Return from checkout. Reads the checkout token from the payload; a missing token logs `Received payment data with missing token.` and nothing is processed. |
| `/payment/iyzico/webhook` | `POST` | yes | Webhook. Reads the checkout token and the conversation identifier, which is the transaction reference. Acknowledgement body: empty. |

Both then fetch the authoritative details and **refuse the data outright** when the basket identifier of those details differs from the transaction reference.

## 11.4 Request signature

Shape 1, applied to every outbound request rather than to notifications. A random eight-character alphanumeric string is generated; the signed text is that string, a slash, the endpoint and the serialised body; the keyed hash uses the key secret and the secure hash algorithm, 256-bit variant, rendered as hexadecimal. The authorization header carries a scheme name followed by the base-64 encoding of `apiKey:<key>&randomKey:<random>&signature:<signature>`, and a second header carries the random string.

## 11.5 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `INIT_THREEDS`, `CALLBACK_THREEDS`, `INIT_BANK_TRANSFER`, `INIT_CREDIT`, `PENDING_CREDIT` |
| confirmed | `SUCCESS` |
| error | `FAILURE` |

An error gives `An error occurred during processing of your payment (code %(code)s: %(explanation)s). Please try again.`; an unknown status gives `Unknown status code: %s`.

The provider reference is `paymentId`. The instrument is resolved in three branches: when the data carry a card type, the card association is looked up (with `amex`→`american_express` and `mastercard`→`master_card`); when they carry a bank name, the bank transfer method is used; otherwise the placeholder method is used.

## 11.6 Amounts and restrictions

Major units; the amount reported back is read from `price` and `currency`. Supported currencies: Swiss franc, euro, pound sterling, Iranian rial, Norwegian krone, Russian rouble, Turkish lira, United States dollar.

---

# 12. Mercado Pago

**Credentials**: account country, access token, access token expiry, refresh token, public key.
**Features**: tokenization.
**Flow**: on the platform (inline card form) or on the provider's page.

## 12.1 Service addresses

| Target | Service address |
|---|---|
| `provider` | `https://api.mercadopago.com` |
| `proxy` | the platform's own relay service, version 1, on the path reserved for this provider |

The proxy is the platform's own relay used only by the authorization flow: exchanging an authorization code for tokens and refreshing an access token. Proxy answers are wrapped in a remote-call envelope.

**Headers**: a fixed platform identifier header on every request; the idempotency key on a retryable request; and, for a direct request that is neither a proxy call nor a token refresh, a bearer credential built from a valid access token. Fetching that token refreshes it when it has expired. The expiry stored on the provider is deliberately backdated by 31 days relative to what the provider announces, in order that the token is refreshed while the refresh token is still usable.

## 12.2 Outbound requests

**Create a checkout preference** at `/checkout/preferences`. Body: the base payload (the transaction reference as the external reference, the webhook address with the reference appended to its path, and the company name as the statement descriptor), plus automatic return for every outcome, the three return addresses all pointing at the connector's return endpoint, one item with the reference as title, quantity one, the currency code and the converted unit price, and a payer block with the name, the email address, the phone number and the address. The answer's initiation address is taken from the production field when the provider is enabled and from the sandbox field otherwise; it is split into an address and query parameters for the redirect template.

**Make a direct payment** at `/v1/payments`. Body: the base payload, an additional-information block with one item, a payer block with the first name, last name and email address, the transaction amount, the card token, the number of installments, the instrument identifier and the issuer identifier.

**Charge a token.** A fresh card token is first created at `/v1/card_tokens` from the stored card identifier, because the provider requires a new token per payment. Then a payment is made at `/v1/payments` with the base payload, the additional-information block, the transaction amount, the fresh token, one installment and a payer block that carries the stored customer identifier.

**Read a payment** at `/v1/payments/<identifier>`, used by both inbound endpoints.

**Create or find a customer and save a card**, used when tokenizing: search customers by email address, create one when none exists, then create a card from the payment token, the issuer identifier and the instrument identifier; the answer gives the card identifier and its last four digits.

## 12.3 Inbound

| Endpoint | Method | Authentication | Purpose |
|---|---|---|---|
| `/payment/mercado_pago/payments` | service | none beyond the transaction lookup | Make a direct payment from the inline form and process the answer. |
| `/payment/mercado_pago/return` | `GET` | the verification call | Return from checkout. When the payment identifier is the literal text `null` the customer pressed the return button and nothing is processed. |
| `/payment/mercado_pago/webhook/<reference>` | `POST` | the verification call | Webhook. Only the actions `payment.created` and `payment.updated` are handled; the other notifications the provider sends carry less information and are ignored. The reference is taken from the address path. Acknowledgement body: empty. |
| `/payment/mercado_pago/oauth/return` | `GET`, logged-in user | a cross-site request forgery token | Authorization return, see 12.6. |

Both the return and the webhook fetch the authoritative payment and **refuse** it when its external reference differs from the transaction reference.

The reference is read from `external_reference`.

## 12.4 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `pending`, `in_process`, `in_mediation`, `authorized` |
| confirmed | `approved`, `refunded` |
| canceled | `cancelled`, `null` |
| error | `rejected` |

A missing payment identifier gives `Received data with missing payment id.`; a missing status gives `Received data with missing status.`; an unknown status gives `Received data with invalid status: %s.` A rejection is translated into a human message through a table of sixteen status details, falling back to `Payment was not processed, use another card or contact issuer.` The table covers: credited, pending contingency, pending manual review, wrong card number, wrong expiry date, wrong other data, wrong security code, blacklisted card, authorization required, disabled card, card error, duplicated payment, high risk, insufficient funds, invalid installments and maximum attempts reached.

The instrument is resolved by translating the payment type through the mapping (`card`→`debit_card,credit_card,prepaid_card`, `paypal`→`digital_wallet`, `mastercard`→`master`, the provider's own wallet→`mercado_pago`), then, for a card, by using the instrument identifier. A method that is not found falls back to the placeholder method whose code is `unknown`.

## 12.5 Amounts and restrictions

The amount is rounded **down** to the connector's own precision: Colombian peso, Honduran lempira and Nicaraguan córdoba to zero decimals, every other currency to the payment precision. The amount reported back is read from the item unit price for a redirect or direct payment, and from the transaction amount for a token payment.

Supported countries: Argentina, Brazil, Chile, Colombia, Mexico, Peru, Uruguay. The account country decides the single allowed currency: Argentine peso, Brazilian real, Chilean peso, Colombian peso, Mexican peso, Peruvian sol, Uruguayan peso. The provider is removed from the compatible providers for a validation operation (PAY-RULE-085).

## 12.6 Guided setup and authorization

1. The setup refuses a company whose country is not supported: `Mercado Pago is not available in your country; please use another payment provider.`, offering a link to the provider list.
2. It refuses a provider without an account country: `Set the account country before connecting the account.`
3. It builds a return address carrying the provider identifier and a cross-site request forgery token, and redirects the browser to the proxy's authorization address with that return address and the account country.
4. The authorization return endpoint checks that the provider exists and has the right code (`Could not find Mercado Pago provider %s`), verifies the token, and, when the customer cancelled, simply returns to the provider form.
5. Otherwise it exchanges the authorization code for an access token, a refresh token and a public key through the proxy, writes them with the backdated expiry, sets the state to `enabled`, publishes the provider, switches tokenization on, and sets the currency from the account country. A failure renders an authorization error page with the provider's message and a link back to the provider form.

The Reset credentials operation clears the four credential fields and switches tokenization off.

## 12.7 Inline form values

The customer's email address, the public key, and the locale.

The locale is resolved in three steps:

1. Read the browsing language of the request, which is a text such as `pt_BR` (Brazilian Portuguese) or `es_419` (Latin-American Spanish). An absent language counts as the empty text.
2. Derive a country code from it: when the language is exactly `es_419`, which names a region and not a country, take the code of the company's country instead; in every other case take the part of the language after the last underscore, unchanged, which for a language without an underscore is the whole text.
3. Look the country code up in the table below. A code that is not in the table, and an empty code, both give the fallback locale `en-US` (United States English).

| Country | Country code as stored | Value written into the inline form |
|---|---|---|
| Argentina | `AR` | `locale` = `es-AR` |
| Brazil | `BR` | `locale` = `pt-BR` |
| Chile | `CL` | `locale` = `es-CL` |
| Colombia | `CO` | `locale` = `es-CO` |
| Mexico | `MX` | `locale` = `es-MX` |
| Peru | `PE` | `locale` = `es-PE` |
| Uruguay | `UY` | `locale` = `es-UY` |

The codes in the first column are written in capitals because the derivation compares them with the part of the language code after the underscore and with the country code of the company, both of which are stored in capitals; the comparison is exact and is not case-insensitive.

---

# 13. Mollie

**Credentials**: service key (live or test, according to the provider state).
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 13.1 Service address

A single address: `https://api.mollie.com/v2/`. Headers: the key as a bearer credential, a structured-data content type, and a user-agent string built from the platform version and the connector version. The error text is read from `detail`.

## 13.2 Outbound requests

**Create a payment** at `/payments`. Body: the transaction reference as the description; the amount as a structure with the currency code and the amount rendered as text with exactly as many decimals as the payment precision of that currency; the locale, when it is one of the twenty-one the provider supports, otherwise United States English; a single-entry list with the provider's instrument code; the return address with the transaction reference as a query parameter; and the webhook address with the same parameter. The provider reference is written immediately from the answer, because the provider does not send the reference back on return. The checkout address of the answer is split into an address and query parameters, which is necessary because the provider strips the parameters when only one instrument is enabled.

**Read a payment** at `/payments/<provider reference>`, used by both inbound endpoints.

## 13.3 Inbound

| Endpoint | Method | Session | Purpose |
|---|---|---|---|
| `/payment/mollie/return` | `GET` and `POST` | no new session | Return from checkout; fetch the authoritative payment; process. |
| `/payment/mollie/webhook` | `POST` | yes | Webhook; same handling; acknowledgement body: empty. |

The reference is read from the query parameter `ref` that the connector itself put in both addresses. There is no signature: the authenticity comes from the verification call.

## 13.4 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `pending`, `open` |
| authorized | `authorized` |
| confirmed | `paid` |
| canceled | `expired`, `canceled`, `failed`, with the state message `Cancelled payment with status: %s` |

An unknown status gives `Received data with invalid payment status: %s.`

The instrument is read from `method`; for a card the card label is used instead, lower-cased. The mapping is: Apple wallet→`applepay`, card→`creditcard`, bank transfer→`banktransfer`, the Belgian bank button→`kbc`, the Polish bank scheme→`przelewy24`, Single Euro Payments Area direct debit→`directdebit`.

Note that the connector maps the provider's authorized status onto the authorized transaction state even though it declares no manual capture support; a provider record whose support flag is empty would then fail the authorized-state check (PAY-RULE-030), therefore in practice that status only appears on accounts where the feature is not used.

## 13.5 Amounts and restrictions

Major units, rendered as text with the payment precision of the currency. Supported currencies: United Arab Emirates dirham, Australian dollar, Bulgarian lev, Brazilian real, Canadian dollar, Swiss franc, Czech koruna, Danish krone, euro, pound sterling, Hong Kong dollar, Croatian kuna, Hungarian forint, Israeli shekel, Icelandic króna, Japanese yen, Mexican peso, Malaysian ringgit, Norwegian krone, New Zealand dollar, Philippine peso, Polish złoty, Romanian leu, Russian rouble, Swedish krona, Singapore dollar, Thai baht, new Taiwan dollar, United States dollar, South African rand.

---

# 14. Nuvei

**Credentials**: merchant identifier, site identifier, secret key.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 14.1 Service addresses

```
production : https://secure.safecharge.com/ppp/purchase.do
test       : https://ppp-test.safecharge.com/ppp/purchase.do
```

## 14.2 Outbound request

The browser posts a form with: the snapshot address, city, country code, currency code, email address; the encoding name; the first name truncated to 30 characters and the last name truncated to 40; one item with the rounded amount, the reference as its name and quantity one; the reference as the product identifier; the contact's language; the merchant and site identifiers; the instrument filter mode and the provider's instrument code; the formatted phone number; the state code; a random user token; the transaction's creation moment formatted as year-month-day.hour:minute:second; the rounded total amount; the protocol version `4.0.0`; the postal code; the back and error addresses, which both carry the transaction reference and an access token in order that an abandonment can still be recognised; the notification address; and the pending and success addresses. A checksum over every parameter closes the form.

Two guards apply before the form is built:

- a method that requires a full name (the Brazilian bank slip) refuses a contact without both a first and a last name, with the connector prefix followed by `%(payment_method)s requires both a first and a last name.`;
- a method that refuses decimals (the Chilean bank redirect) forces the amount to zero decimals.

## 14.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/nuvei/return` | `GET` | Return from checkout. When the address carries the error access token, the customer abandoned the page or the provider failed to parse the request: the payment data are treated as empty and the access token over the reference is verified instead of a signature. Otherwise the returned data are verified with a signature and processed. |
| `/payment/nuvei/webhook` | `POST` | Webhook; verify; process; acknowledge with the two-letter text that spells the word "okay" in capitals. |

The reference is read from `productId`; when the data are empty, the reference put in the return address is used instead.

## 14.4 Signature

Shape 2. For incoming data the signed fields are, in this fixed order: `totalAmount`, `currency`, `responseTimeStamp`, `PPP_TransactionID`, `Status`, `productId`. For outgoing data every parameter of the form takes part, in the order it was built. The signing string is the secret key followed by the concatenated values, hashed with the secure hash algorithm, 256-bit variant, and rendered as hexadecimal. The received signature sits in `advanceResponseChecksum`.

## 14.5 Status mapping

| Transaction state | Provider statuses (lower-cased) |
|---|---|
| pending | `pending` |
| confirmed | `approved`, `ok` |
| error | `declined`, `error`, `fail` |

Empty payment data cancel the transaction with the state message `The customer left the payment page.` A missing status gives `Received data with missing payment state.`; an error gives `An error occurred during the processing of your payment (%(reason)s). Please try again.`; an unknown status gives `Received invalid transaction status %(status)s and reason '%(reason)s'.`

The provider reference is `TransactionID`. The instrument is read from `payment_method` and translated through the mapping: Astropay bank transfer, the Brazilian bank slip, card, the provider's local scheme, the Mexican convenience store scheme, the Brazilian instant scheme, the Colombian bank scheme, the Mexican interbank scheme and the Chilean bank redirect.

## 14.6 Amounts and restrictions

Major units, rounded down to zero decimals for the integer-only methods and to the currency's decimals otherwise; the amount check uses the same precision. Supported currencies: Argentine peso, Brazilian real, Canadian dollar, Chilean peso, Colombian peso, Mexican peso, Peruvian sol, United States dollar, Uruguayan peso.

---

# 15. Paymob

**Credentials**: account country, public key, secret key, keyed-hash key, service key.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 15.1 Service addresses

The address prefix is chosen by the account country: United Arab Emirates `uae`, Egypt `accept`, Oman `oman`, Saudi Arabia `ksa`, each forming `https://<prefix>.paymob.com`.

**Headers**: a bearer credential which is the secret key for a client request, a one-hour access token otherwise, and nothing at all for the token request itself. The token is obtained by posting the service key to the account token endpoint of the provider (`paymob` path `/api/auth/tokens`); a missing token gives `Could not generate a new access token.` The error parser recognises the provider's "field may not be blank" answer and turns it into `The following fields must be filled: %(fields)s` listing the missing billing fields.

## 15.2 Outbound requests

**Create an intention** at `/v1/intention/`, as a client request. Body: the transaction reference as the special reference; the amount in minor units; the currency code; the list of instrument names; the webhook address; the redirection address; and a billing block with the first name (falling back to the last name, then to the empty text), the last name, the email address, the street, the state name, the phone number stripped of spaces and the country code. The instrument names are the platform method codes with their underscores removed and the environment suffix `live` or `test` appended; when the chosen method is the Omani national scheme, the card method is appended as well, because the provider refuses that scheme on its own. The answer gives the intention order identifier, which becomes the provider reference, and a client secret; the customer is redirected to the unified checkout address with the public key and that secret as parameters.

**Synchronise the instruments.** Reads the provider's integration list (`paymob` path `/api/ecommerce/integrations`) with a page size of 500 and the live flag set from the provider state, keeps the gateways that map onto an enabled platform method, and writes back, for each of them, an integration name equal to the platform method code without underscores plus the environment suffix. The matching drops: gateways whose integration name mentions the Apple or the Google wallet; card gateways that are mail-order or authorization-only; and duplicates, keeping the most recently created gateway per method. When fewer gateways matched than there are enabled methods, the operation returns the warning `Payment methods not found` with `Not all enabled payment methods were found on your account.`; otherwise `Successfully synchronized with Paymob` with `Payment methods have been successfully set up!` A card gateway that offers installments is mapped onto the Egyptian installment method when that method exists.

## 15.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/paymob/return` | `GET` | Return from checkout; the order identifier of the data must equal the transaction's provider reference, otherwise the request is refused; then verify the signature and process. |
| `/payment/paymob/webhook` | `POST` | Webhook; the payload's object is normalised first (see below); the same order check applies; acknowledgement body: empty. |

**Normalisation.** The webhook sends parsed values while the redirect sends text, therefore both are reduced to the same shape: every signed field is rendered as text, a boolean becoming the words `true` or `false` and an absent field becoming `false`; then the message, the signature, the order identifier, the merchant order identifier and the three source-data fields are added.

The reference is read from `merchant_order_id`.

## 15.4 Signature

Shape 1. The signed fields, in this fixed order: `amount_cents`, `created_at`, `currency`, `error_occured`, `has_parent_transaction`, `id`, `integration_id`, `is_3d_secure`, `is_auth`, `is_capture`, `is_refunded`, `is_standalone_payment`, `is_voided`, `order`, `owner`, `pending`, `source_data.pan`, `source_data.sub_type`, `source_data.type`, `success`. They are concatenated with no separator; the keyed hash uses the keyed-hash key and the secure hash algorithm, 512-bit variant, rendered as hexadecimal. The received signature sits in `hmac`.

## 15.5 Status mapping

| Condition | Transaction state |
|---|---|
| `pending` is the word `true` | pending |
| `success` is the word `true` | confirmed |
| otherwise | error, with `An error occurred during the processing of your payment (%(msg)s). Please try again.` |

The connector does not write a provider reference during the update, because it was already written when the intention was created.

## 15.6 Amounts and restrictions

Minor units. The amount reported back is read from `amount_cents` and converted to major units. One currency per account, chosen by the account country: United Arab Emirates dirham, Egyptian pound, Omani rial, Saudi riyal. References are singularized in order that they are unique.

---

# 16. PayPal

**Credentials**: business email address, client identifier, client secret, access token, access token expiry, webhook identifier.
**Features**: none beyond a plain payment.
**Flow**: on the platform, through the provider's browser component.

## 16.1 Service addresses

| Provider state | Service address |
|---|---|
| `enabled` | `https://api-m.paypal.com` |
| `test` | `https://api-m.sandbox.paypal.com` |

**Headers**: a structured-data content type; a fixed partner attribution identifier; the idempotency key under the provider's own request-identifier header; and, except for the token request, a bearer credential. The token request authenticates with the client identifier and the client secret as a user name and a password and asks for a client-credentials grant; the token is refreshed when it is within five minutes of expiring; a missing token gives `Could not generate a new access token.` The error text is read from `message`.

## 16.2 Outbound requests

**Create an order** at `/v2/checkout/orders`. Body: the intent `CAPTURE`; one purchase unit with the transaction reference as its reference identifier, a description built from the company name and the reference, the amount as a currency code and a value, and a payee block with the company name as the brand name and the provider's business email address, plus the company email address when it is set; a shipping block built from the delivery address of the linked sales order or invoice, included only when that address has a street, a city, a country, and a postal code and a state when the country requires them; and a payment source block with the shipping preference (`SET_PROVIDED_ADDRESS` when a shipping address was included, `NO_SHIPPING` otherwise), the first and last name, and the invoicing address. For the public contact the invoicing address is reduced to the company's country code and no shipping address is sent. The answer's order identifier is handed to the browser.

**Capture an order** at `/v2/checkout/orders/<order identifier>/capture`, with the idempotency key.

**Create a webhook** at `/v1/notifications/webhooks` with the connector's webhook address and the three handled event names. Refused on a local address with the connector prefix followed by "You must have an HTTPS connection to generate a webhook."

**Verify a webhook signature** at `/v1/notifications/verify-webhook-signature` with the five transport headers, the stored webhook identifier and the whole event.

## 16.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/paypal/complete_order` | service, `POST` | Called by the browser once the customer approved the order: captures it, normalises the answer and processes it. |
| `/payment/paypal/webhook/` | `POST` | Webhook. Only three events are handled: order completed, order approved, and payment approval reversed. The event is normalised, the transaction matched, the signature verified by asking the provider, and the data processed. A verification failure sets the transaction to `error` with `Unable to verify the payment data`. Acknowledgement body: empty. |

**Normalisation.** Both shapes are reduced to: the keys of the payment source, the reference identifier of the first purchase unit, and then either the whole purchase unit plus the intent, the order identifier and the order status (for a webhook), or the first capture of that purchase unit plus the operation kind `CAPTURE` (for a capture answer). A capture answer with no capture logs `Invalid response format, can't normalize.`

The reference is read from `reference_id`.

## 16.4 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `PENDING`, `CREATED`, `APPROVED` |
| confirmed | `COMPLETED`, `CAPTURED` |
| canceled | `DECLINED`, `DENIED`, `VOIDED` |
| error | `FAILED` |

Empty payment data cancel the transaction with `The customer left the payment page.` A missing identifier or operation kind gives `Missing value for txn_id (%(txn_id)s) or txn_type (%(txn_type)s).`; an unknown status gives `Received data with invalid payment status: %s`. A pending state carries the provider's pending reason as its state message.

The instrument is always forced to the provider's own method when that method exists.

## 16.5 Amounts and restrictions

Major units. Supported currencies: Australian dollar, Brazilian real, Canadian dollar, Czech koruna, Danish krone, euro, Hong Kong dollar, Hungarian forint, Israeli shekel, Japanese yen, Malaysian ringgit, Mexican peso, new Taiwan dollar, New Zealand dollar, Norwegian krone, Philippine peso, Polish złoty, pound sterling, Russian rouble, Singapore dollar, Swedish krona, Swiss franc, Thai baht, United States dollar.

---

# 17. PayU

**Credentials**: key identifier, merchant salt (both filled by the authorization flow).
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 17.1 Service addresses

| Target | Service address |
|---|---|
| `enabled` | `https://secure.payu.in` |
| `test` | `https://test.payu.in` |
| `partner` | `https://partner.payu.in` |
| `proxy` | the platform's own relay service, version 1, on the path reserved for this provider |

The partner service is reached with a bearer credential obtained through the proxy; the payment service is reached by posting a form.

## 17.2 Outbound request

The browser posts a form to the payment address, path `_payment`, with: the key identifier; the transaction reference as the transaction identifier; the amount as text (the provider expects text despite its own documentation); a fixed product description; the first and last name; the email address; the phone number; the success and failure addresses, both the connector's return endpoint; the two webhook addresses, both the connector's webhook endpoint; the provider's instrument filter for the chosen method; and the signature.

## 17.3 Inbound

| Endpoint | Method | Session | Purpose |
|---|---|---|---|
| `/payment/payu/return` | `POST` | no new session | Return from checkout; verify; process. |
| `/payment/payu/webhook` | `POST` | yes | Webhook; verify; process; acknowledgement body: empty. |
| `/payment/payu/oauth/return` | `GET`, logged-in user, no new session | Authorization return. |

The reference is read from `txnid`.

## 17.4 Signature

Shape 2. Outgoing values, in this order: `key`, `txnid`, `amount`, `productinfo`, `firstname`, `email`, `udf1` to `udf10`, and the merchant salt. Incoming values, in the exactly reversed order: the salt, `status`, `udf10` down to `udf1`, `email`, `firstname`, `productinfo`, `amount`, `txnid`, `key`. The values are joined with a vertical bar, hashed with the secure hash algorithm, 512-bit variant, and rendered as lower-case hexadecimal. The received signature sits in `hash`.

## 17.5 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `pending` |
| confirmed | `success` |
| error | `failure` |

An error gives `%(code)s: %(explanation)s` with the provider's error message; an unknown status gives `Unknown status code: %s`.

The provider reference is `mihpayid`. The instrument is read from `mode`; the two card modes are folded into the card method, and the rest is translated through the response mapping (installments, net banking, buy-now-pay-later, the unified interface, wallets).

## 17.6 Amounts and restrictions

Major units as text. The amount reported back is read from `amount`; the provider sends no currency, therefore the transaction's own currency code is used, which effectively disables the currency half of the check while keeping the amount half. Only the Indian rupee is supported.

## 17.7 Guided setup and authorization

1. The setup refuses a company whose currency is not the Indian rupee: `PayU is not available in your country; please use another payment provider.`
2. It redirects to the proxy's authorization address with the provider identifier, a cross-site request forgery token and the return address.
3. The return endpoint checks the provider (`Could not find PayU provider with identifier %s`), verifies the token, and, when the customer cancelled, returns to the provider form.
4. Otherwise it exchanges the authorization code for an access token through the proxy, reads the merchant's credentials from the partner service, writes the production key and salt, sets the state to `enabled` and publishes the provider. A failure renders an authorization error page.

The Reset credentials operation clears the key identifier and the merchant salt.

---

# 18. Razorpay

**Credentials**: key identifier and key secret (classic), or account identifier, public token, refresh token, access token and access token expiry (authorized); webhook secret.
**Features**: tokenization; manual capture, full only; refunds, full and partial.
**Flow**: on the platform.

## 18.1 Service addresses

| Target | Service address |
|---|---|
| `provider` | `https://api.razorpay.com/<version>/<endpoint>`, where the version is 1 by default and 2 for the webhook endpoint |
| `proxy` | the platform's own relay service, version 1, on the path reserved for this provider |

**Authentication**: when an access token exists and no classic key is set, a bearer credential is used, refreshing the token when it has expired; when a classic key is set, basic authentication with the key identifier and the key secret is used. Proxy calls use neither. The error text is read from the description of the error block.

## 18.2 Outbound requests

**Create a customer** at `customers`: the contact name with commas replaced by spaces and truncated to 50 characters, the email address, the validated phone number, and a flag that tells the provider not to fail when the customer already exists.

**Create an order** at `orders`: the amount in minor units; the currency code; the primary instrument code, omitted for the four instruments the order service does not recognise (Indian installments, the Malaysian bank scheme, buy-now-pay-later, Indian wallets); for a direct payment or a validation, the customer identifier and, when tokenizing, a token block with the mandate maximum amount in minor units, an expiry ten years in the future and the frequency "as presented"; for a token payment, a capture flag that is the opposite of the provider's manual capture switch; and, when the provider captures manually, a payment block asking for manual capture with a two-hour expiry and a normal refund speed.

**Charge a token** at `payments/create/recurring`: the email address, the validated phone number, the order amount, the currency code, the order identifier, the customer identifier and the token identifier (both halves of the token's provider reference, which stores them separated by a comma), the transaction reference as the description, and a recurring flag.

**Capture** at `payments/<source provider reference>/capture`: the amount in minor units and the currency code.

**Refund** at `payments/<source provider reference>/refund`: the negated amount in minor units and a note carrying the transaction reference, which is how the refund is recognised in a later notification.

**Read a payment** at `payments/<identifier>`: used when the return data are incomplete.

**Create a webhook** at `accounts/<account identifier>/webhooks` on version 2: the webhook address, the current user's email address for alerts, a freshly generated random secret, and the five handled event names. The secret is then stored on the provider.

## 18.3 Inbound

| Endpoint | Method | Session | Purpose |
|---|---|---|---|
| `/payment/razorpay/return` | `POST` | no new session | Return from checkout. Processed only when the three expected fields (order identifier, payment identifier, signature) are all present; otherwise the customer cancelled or the payment failed and nothing is processed. The transaction is matched on the reference carried in the address. |
| `/payment/razorpay/webhook` | `POST` | yes | Webhook. Only five events are handled: payment authorized, payment captured, payment failed, refund failed, refund processed. The entity is extracted and tagged with its kind. Acknowledgement body: empty. |
| `/payment/razorpay/oauth/return` | `GET`, logged-in user | Authorization return. |

## 18.4 Signature

Two shapes. For a return, shape 1 over the text `<order identifier>|<payment identifier>` with the key secret and the secure hash algorithm, 256-bit variant. For a webhook, shape 1 over the raw request body with the webhook secret and the same algorithm; when no webhook secret is stored the connector logs `Missing webhook secret; aborting signature calculation.` and the verification fails. The received signature sits in the return payload for the first shape and in a request header for the second.

## 18.5 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `created`, `pending` |
| authorized | `authorized` |
| confirmed | `captured`, `refunded`, `processed` |
| error | `failed` |

The confirmed set deliberately contains `refunded`, in order that a payment already refunded at the provider is not reprocessed as something else.

Rules on top: an authorized status only moves the transaction to `authorized` when the provider captures manually, otherwise it is ignored; a confirmed status that carries a token identifier on a transaction that has no token yet and whose provider allows tokenization switches the tokenize flag on before confirming; a confirmed refund wakes the post-processing job; an error gives `An error occurred during the processing of your payment. Please try again.`; an unknown status gives the connector prefix followed by `Received data with invalid status: %s`; a missing status gives `Received data with missing status.`; a missing entity identifier gives `Received data with missing entity id.`

The provider reference and the instrument are **not** overwritten while the transaction is already `done` or `authorized`, because one reference may carry several provider-side attempts after a retry.

The reference is re-checked against the entity's description or its refund note and the request is refused when it differs.

## 18.6 Matching

A payment entity is matched by its description. A refund entity is matched by the reference in its note; when there is none, the refund was started at the provider, therefore the source transaction is found by the provider reference of the payment and a refund transaction is created (`Received incomplete refund data.` when the identifier or the amount is missing).

## 18.7 Guards

- Voiding is refused outright: `Transactions processed by Razorpay can't be manually voided from the system.`
- A token payment is refused when another token payment for the same document, with the same token, is still pending and less than 36 hours old: `Your last payment %s will soon be processed. Please wait up to 24 hours before trying again, or use another payment method.` The document is identified by the transaction reference with everything after the last hyphen removed.
- The phone number is validated and formatted; a missing number when tokenizing gives `The phone number is missing.` and an unparsable one gives `The phone number is invalid.`

## 18.8 Amounts and restrictions

Minor units with the payment precision table. The validation amount is 1. Ninety-two currencies are supported; the Indian rupee is the reference currency of the mandate limits. The guided setup refuses a company whose currency is not among the supported ones: `Razorpay is not available in your country; please use another payment provider.` The authorization flow clears the classic credentials, stores the account identifier, the public token, the refresh token, the access token and its expiry, enables and publishes the provider, and then tries to create the webhook, logging a failure without blocking. The Reset credentials operation clears the five authorization fields.

---

# 19. Redsys

**Credentials**: merchant code, merchant terminal, secret key.
**Features**: none beyond a plain payment.
**Flow**: on the provider's page.

## 19.1 Service addresses

```
production : https://sis.redsys.es/sis/realizarPago
test       : https://sis-t.redsys.es:25443/sis/realizarPago
```

## 19.2 Outbound request

The merchant parameters are a structure carrying: the amount in minor units as text; the numeric code of the currency; the merchant code; the merchant terminal; the transaction reference as the order; the webhook address; the operation type `0` (authorization); the return address twice, for success and for failure; the provider's instrument code (card by default); and a three-domain secure block with the city, the numeric country code, the street, the postal code, the cardholder name, the email address and, when it is known, the state code. The structure is serialised and encoded in base 64, and the form carries that text, the signature and the signature version name.

The instrument mapping is: the Spanish instant scheme `z`, card `C`, and the five card brands `1`, `2`, `8`, `6`, `9`.

The numeric country codes come from a complete table of the international country code standard, alphabetic to numeric.

## 19.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/redsys/return` | `GET` | Return from checkout; the merchant parameters are decoded from base 64 and parsed; verify; process. |
| `/payment/redsys/webhook` | `POST` | Webhook; same decoding; verify; process; acknowledge with the two-letter text that spells the word "okay" in capitals. |

The reference is read from `Ds_Order`. The provider reference is written to the transaction reference at creation time, because the provider uses the order as its own reference.

## 19.4 Signature

Shape 1 with a derived key. The secret key is decoded from base 64; a key is derived by encrypting the transaction reference, padded with null bytes to 16 characters, with the triple data encryption algorithm in cipher-block-chaining mode with a null initialisation vector; the keyed hash of the encoded merchant parameters with that derived key and the secure hash algorithm, 256-bit variant, is encoded in the address-safe base 64 alphabet. The received signature sits in `Ds_Signature`.

## 19.5 Status mapping

| Transaction state | Provider response codes |
|---|---|
| confirmed | `0000` to `0099`, `0400`, `0900` |
| canceled | `9915` |
| error | the fifty-four codes listed in the error table |

The error table groups the codes into six human messages: `Invalid card details.`; `Insufficient funds or limit exceeded.`; `Authentication failed.`; `Operation not allowed for this card or payment method.`; `Transaction declined by the bank.`; `Technical error. Please try again later.` An unknown code gives `Unknown status code: %s`.

The instrument is read from `Ds_Card_Brand` and translated through the same mapping as the request.

## 19.6 Amounts and restrictions

Minor units as text. The amount reported back is read from `Ds_Amount` and the currency is found by its numeric code. No currency or country restriction. References are between 9 and 12 characters, purely alphanumeric, built from a ten-digit timestamp with the separator `S`.

---

# 20. Stripe

**Credentials**: publishable key, secret key, webhook secret.
**Features**: tokenization; manual capture, full only; refunds, full and partial; express checkout.
**Flow**: on the platform.

## 20.1 Service addresses

| Target | Service address |
|---|---|
| `provider` | `https://api.stripe.com/v1/` |
| `proxy` | the platform's own relay service, on the path reserved for this provider, with the version as the last path element |

**Headers**: the secret key as a bearer credential; a fixed service version, which the setup-intent operation requires; the idempotency key on a retryable request; plus any extra header a fuller connected-account implementation adds. Proxy calls carry no headers. The error text is read from the message of the error block.

## 20.2 Outbound requests

**Create a payment intent** at `payment_intents`. Body: the amount in minor units; the lower-cased currency code; the transaction reference as the description; the capture method, manual when the provider captures manually and automatic otherwise; the instrument type, taken from the primary method and translated through the mapping; an instruction to expand the instrument in the answer; and the delivery address of the linked sales order or invoice. For a token payment it also carries the confirmation flag, the customer identifier stored as the token's provider reference, the off-session flag, the instrument identifier stored on the token and the mandate identifier when there is one. For a direct payment it carries a freshly created customer and, when tokenizing, the instruction to keep the instrument for later use plus, for the currencies in the mandate list, the mandate options.

**Create a setup intent** at `setup_intents` for a validation: the customer, the transaction reference as the description, the instrument type, and the mandate options for the currencies in the mandate list.

**Create a customer** at `customers`: the snapshot city, country code, street, postal code and state name, a description naming the contact and its identifier, the email address, the name and the phone number truncated to 20 characters.

**Capture** at `payment_intents/<source provider reference>/capture`. **Void** at `payment_intents/<source provider reference>/cancel`. **Refund** at `refunds` with the source provider reference and the negated amount in minor units.

**Create a webhook** at `webhook_endpoints` with the webhook address, the eight handled events and the fixed service version; the returned secret is stored. **Verify the express-checkout domain** at `apple_pay/domains` with the site's host name; an answer that is not in live mode gives `Please use live credentials to enable Apple Pay.`

**Connected account operations** go through the proxy: create an account, create an account link. The account link carries the connected account, the return and refresh addresses (both carrying the provider identifier, the menu identifier and, for the refresh address, the account identifier) and the link type.

## 20.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/stripe/return` | `GET` | Return from payment, used for both the direct and the redirected flows. Fetches the intent (a payment intent, or a setup intent for a validation) with the instrument expanded, refuses the data when the intent's description differs from the transaction reference, wraps the intent into the payment data and processes it. A failure logs `Failed to process the return from Stripe.` The redirection to the status page is done with the address parameters kept out of the log. |
| `/payment/stripe/webhook` | `POST` | Webhook, see below. Acknowledgement body: empty. A failure inside the handling is swallowed and still acknowledged, in order that the provider stops re-sending. |
| `/.well-known/apple-developer-merchantid-domain-association` | `GET` | Serves the static file the wallet provider reads to verify the site's domain. |
| `/payment/stripe/onboarding/return` | `GET`, logged-in user | Redirects to the provider form after the account onboarding. |
| `/payment/stripe/onboarding/refresh` | `GET`, logged-in user | Builds a new onboarding link and redirects to it. |

**Webhook handling.** Eight events are handled: payment intent processing, payment intent amount capturable updated, payment intent succeeded, payment intent payment failed, payment intent canceled, setup intent succeeded, charge refunded, charge refund updated. The connector builds a small payload with the object's description as the reference, the event kind and the object identifier, matches the transaction, verifies the signature, and then:

- for a payment intent event: fetches the instrument when the transaction asks for a token, then wraps the intent;
- for a setup intent event: fetches the instrument, then wraps the intent;
- for a charge refunded event: skips the event entirely when the charge was not captured (that is a void, not a refund); otherwise reads every refund of the charge, paging through them, and, for each refund that has no transaction yet (including refunds of capture children, which are grandchildren of the charge's transaction), creates a refund transaction with the amount converted to major units and processes it; the source transaction itself is not processed;
- for a charge refund updated event: wraps the refund, which allows a refund that had been reported as succeeded to be moved to `error`.

## 20.4 Signature

Shape 1 with a freshness check. The signature header carries comma-separated `name=value` pairs; the timestamp is read from the pair named with the letter t and the signature from the pair named `v1`. A missing webhook secret, a missing timestamp, a timestamp older than ten minutes, or a missing signature all answer "forbidden". The signed text is the timestamp, a dot and the raw request body; the keyed hash uses the webhook secret and the secure hash algorithm, 256-bit variant, rendered as hexadecimal.

## 20.5 Status mapping

| Transaction state | Provider statuses |
|---|---|
| ignored (the intent still needs an action) | `requires_confirmation`, `requires_action` |
| pending | `processing`, `pending` |
| authorized | `requires_capture` |
| confirmed | `succeeded` |
| canceled | `canceled` |
| error | `requires_payment_method`, `failed` |

A missing status gives `Received data with missing intent status.`; an unknown status gives `Received data with invalid intent status: %s.` For a payment, an error carries the provider's last error message, or `The customer left the payment page.` when there is none. For a refund, an error gives `The refund did not go through. Please log into your Stripe Dashboard to get more information on that matter, and address any accounting discrepancies.` and is allowed from the confirmed state, which is the only way a confirmed transaction becomes errored. A confirmed refund wakes the post-processing job.

The provider reference is the setup intent identifier for a validation, the refund identifier for a refund, and the payment intent identifier otherwise. The instrument is read from the expanded instrument's type; for a card whose platform method is also the card method, the card brand is used instead.

## 20.6 Matching

By reference when the payload carries one. A refund-update event carries no description, therefore the transaction is found by the provider reference, which for a refund transaction is the refund identifier.

## 20.7 Token values

The instrument must be present, otherwise the connector logs `requested tokenization from payment data with missing payment method`. The customer identifier is read from the payment intent (for a direct payment) or from the setup intent (for a validation); for a direct payment the mandate is read from the charge's instrument details. When the instrument carries no details of its own type, which happens when another instrument such as a direct debit was generated, the customer's first instrument is read instead. The token stores: the last four characters of the instrument as the payment details, the customer identifier as the provider reference, the instrument identifier and the mandate.

## 20.8 Amounts and restrictions

Minor units, with these deviations: Icelandic króna 2, Ugandan shilling 2, Malagasy ariary 0. The amount reported back comes from the refund for a refund and from the payment intent otherwise, and the same precision is used for the check.

Supported countries for the connected-account setup: forty-four countries. Six outlying territories are mapped onto their parent country: Martinique, Guadeloupe, French Guiana, Réunion, Mayotte and Saint-Martin all map to France. The setup refuses a company whose mapped country is not supported: `Stripe Connect is not available in your country, please use another payment provider.` When the provider is already enabled the setup simply closes the window.

Currencies for which an electronic mandate is attached: United States dollar, euro, pound sterling, Singapore dollar, Canadian dollar, Swiss franc, Swedish krona, United Arab Emirates dirham, Japanese yen, Norwegian krone, Malaysian ringgit, Hong Kong dollar.

The sensitive key added to the logging filter is the client secret of an intent.

---

# 21. Toss Payments

**Credentials**: client key, secret key; the webhook address is derived and shown read-only.
**Features**: none beyond a plain payment.
**Flow**: on the platform, through the provider's payment window.

## 21.1 Service address

A single address: `https://api.tosspayments.com/`. Authentication is basic, with the secret key as the user name and an empty password. The idempotency key is the order identifier, a colon and the payment key. The error text is the provider's message followed by its code in brackets.

## 21.2 Outbound requests

**Confirm a payment** at `/v1/payments/confirm`, with the data returned on the success redirect (the order identifier, the payment key and the amount).

The processing values handed to the browser are: the transaction reference as the order name, the contact name, email address and phone number, the success return address, and the failure return address carrying an access token over the reference.

## 21.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/toss-payments/success` | `GET` | Success return. The amount is validated **before** the confirmation request is sent, using the returned amount under the key the webhook uses, in order that a customer cannot lower it. When the validation passed, the confirmation request is sent and its answer is processed; a failure sets the transaction to `error` with the provider's message. |
| `/payment/toss-payments/failure` | `GET` | Failure return. The access token over the reference is verified; when it matches, the transaction is set to `error` with the provider's message and code. |
| `/payment/toss-payments/webhook` | `POST` | Webhook. Only the payment-status-changed event is handled. Acknowledgement body: empty. |

The reference is read from `orderId`.

## 21.4 Verification

Shape 3 with an exemption. The event's secret is compared with the secret stored on the transaction. The check is skipped for the statuses `EXPIRED` and `ABORTED`, because those events may exist for a payment the platform never initiated, or for one whose confirmation request failed before the secret could be stored.

## 21.5 Status mapping

| Transaction state | Provider status |
|---|---|
| confirmed | `DONE` |
| canceled | `EXPIRED` |
| error | `ABORTED`, and every unrecognised status, with `Received data with invalid payment status: %s` |

The statuses `CANCELED` and `PARTIAL_CANCELED` on an already confirmed transaction are ignored: refunds are not implemented, but the provider still notifies a manual cancellation made on its own interface.

The provider reference is `paymentKey`; the event secret is stored on the transaction for later verification.

## 21.6 Amounts and restrictions

Major units; the amount reported back is read from `totalAmount` and paired with the single supported currency. Only the South Korean won is supported (PAY-RULE-012). References are between 6 and 64 characters and contain only letters, digits, `-` and `_`; they are always built from the time-based prefix.

The sensitive key added to the logging filter is the payment secret.

---

# 22. Worldline

**Credentials**: merchant identifier, service key, service secret, webhook key, webhook secret.
**Features**: tokenization.
**Flow**: on the provider's page.

## 22.1 Service addresses

```
production : https://payment.direct.worldline-solutions.com/v2/<merchant identifier>/<endpoint>
test       : https://payment.preprod.direct.worldline-solutions.com/v2/<merchant identifier>/<endpoint>
```

**Headers**: an authorization header carrying the scheme name, the service key and the signature; the request moment in the locale-independent long date format; the content type, which is a structured-data type for a write and empty otherwise; and, on a retryable write, the idempotency key. The error text is the comma-separated list of the messages of the error block.

## 22.2 Request signature

Shape 1. The signed text is the concatenation, each followed by a line break, of: the request method; the content type; the date; then, when an idempotency key is used, the lower-cased idempotency header name, a colon and the key; then the full path including the version and the merchant identifier. The keyed hash uses the service secret and the secure hash algorithm, 256-bit variant, encoded in base 64.

## 22.3 Outbound requests

**Create a hosted checkout session** at `hostedcheckouts`. Body: a hosted-checkout block with the contact's language, the return address carrying the provider identifier, and an instruction not to show the provider's own result page; an order block with the amount in minor units and the currency code, a customer block (billing address, contact details, first and last name) and a references block carrying the transaction reference twice. Then either a redirect block (when the chosen method is one of the fourteen that redirect to a third party) carrying the instrument identifier, the return address and an instruction to force the capture; or a card block carrying the sale authorization mode, which forces the capture, and the tokenize flag, with either a specific instrument identifier or, for the card method and for methods that have brands, a filter restricting the checkout to the card group.

**Charge a token** at `payments`. Body: a card block with the sale authorization mode, the token's provider reference, and the two fields that mark the payment as merchant-initiated and subsequent; an order block with the amount in minor units, the currency code and the transaction reference.

**Read a checkout session** at `hostedcheckouts/<identifier>`, used on return.

## 22.4 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/worldline/return` | `GET` | Return from checkout. The provider identifier is read from the address and must resolve to a Worldline provider, otherwise the request is refused. The checkout session is fetched and its created-payment output is processed. A failure logs `Unable to process the payment data`. |
| `/payment/worldline/webhook` | `POST` | Webhook; verify the signature over the raw body; process; acknowledgement body: empty. |

**Signature of a notification**: shape 1 over the raw request body with the webhook secret and the secure hash algorithm, 256-bit variant, encoded in base 64; the received signature sits in a request header.

The reference is read from the merchant reference of the payment output, allowing for the payment result being wrapped in its own element when the payment failed.

## 22.5 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `CREATED`, `REDIRECTED`, `AUTHORIZATION_REQUESTED`, `PENDING_CAPTURE`, `CAPTURE_REQUESTED` |
| confirmed | `CAPTURED` |
| canceled | `CANCELLED` |
| declined (treated as error) | `REJECTED`, `REJECTED_CAPTURE` |

Rules on top:

- `AUTHORIZATION_REQUESTED` on a token payment sets the transaction to `error` with that status as the message; the next attempt to build the processing values then detects that message, resets the transaction to `draft` with the operation `online_redirect` and forces the redirect flow, in order that the customer can complete the authentication;
- a validation transaction whose status is `PENDING_CAPTURE` or `CAPTURE_REQUESTED` and whose data carry a token is confirmed at once, because the token is what matters;
- a cancellation gives `Transaction cancelled with error code %(error_code)s.`; a decline gives `Transaction declined with error code %(error_code)s.`; an unknown status gives `Received invalid transaction status %(status)s with error code %(error_code)s.`; a missing status gives `Received data with missing payment state.`

The provider reference is the payment identifier with everything after the last underscore removed, because the provider suffixes it with an attempt number. The instrument is read from the card or redirect output's instrument identifier and translated through a twenty-three entry numeric mapping.

## 22.6 Token values

Extracted from the card or redirect output: the last four characters of the card number as the payment details and the token as the provider reference. Nothing is extracted when there is no token.

## 22.7 Amounts and restrictions

Minor units with the payment precision table. No currency or country restriction. References are at most 30 characters; a longer one is rebuilt from a time-based prefix based on two letters.

---

# 23. Xendit

**Credentials**: public key, secret key, webhook token (all three secret).
**Features**: tokenization.
**Flow**: on the platform for cards, on the provider's page for every other instrument.

## 23.1 Service address

A single address: `https://api.xendit.co/`. Authentication is basic, with the secret key as the user name and an empty password. The error text is read from `message`.

## 23.2 Outbound requests

**Create an invoice** at `v2/invoices`, used for every instrument except the card. Body: the transaction reference as the external identifier; the rounded amount; the reference as the description; a customer block with the name and, when they are known, the email address, the mobile number and one address built from the city, country name, postal code, state name and street; the success return address, carrying the reference, an access token over the reference and the amount, and a success flag; the failure return address; the list of instrument codes; and the currency code. When the chosen method is the Malaysian bank scheme, the whole list of its thirty-nine bank codes is sent instead.

**Create a card charge** at `credit_card_charges`, used for the card instrument and for a token payment. Body: the token reference; the transaction reference as the external identifier; the rounded amount; the currency code; the authentication identifier when the payment went through an authentication step; and a recurring flag when the transaction uses or creates a token.

## 23.3 Inbound

| Endpoint | Method | Purpose |
|---|---|---|
| `/payment/xendit/payment` | service | Called by the browser to charge a card or a token. Verifies an access token over the reference (`The access token doesn't match the transaction reference.`), then creates the charge. |
| `/payment/xendit/webhook` | `POST` | Webhook; verify the callback token; process; acknowledgement body: a one-element list holding the word that spells "accepted". |
| `/payment/xendit/return` | `GET` | Return from checkout. When the success flag is set and the access token over the reference and the amount matches, a transaction that is still a draft is moved to pending. Then the browser is redirected to the payment status page. |

**Verification**: shape 3. The provider sends the stored webhook token in a request header; a missing token logs `Received payment data with missing token.` and an unequal one logs `Received payment data with invalid callback token %r.`; both answer "forbidden".

The reference is read from `external_id`.

## 23.4 Status mapping

| Transaction state | Provider statuses |
|---|---|
| pending | `PENDING` |
| confirmed | `SUCCEEDED`, `PAID`, `CAPTURED` |
| canceled | `CANCELLED`, `EXPIRED` |
| error | `FAILED`, with `An error occurred during the processing of your payment (%s). Please try again.` |

The provider reference is the identifier of the answer. The instrument is read from the payment method field; any of the thirty-nine Malaysian bank codes is folded back into the single Malaysian bank method; the rest is translated through an eleven-entry mapping.

## 23.5 Token values

The last four characters of the masked card number as the payment details, and the card token identifier as the provider reference.

## 23.6 Amounts and restrictions

Every supported currency is treated as having zero decimals, and the amount is rounded **down** to that precision both in the requests and in the amount check. Supported currencies: Indonesian rupiah, Malaysian ringgit, Philippine peso, Singapore dollar, Thai baht, United States dollar, Vietnamese dong.

For a validation operation the connector returns no redirect template at all, because the card instrument, which is the only one that can be tokenized, uses the inline flow and rendering a template would compute meaningless values.
