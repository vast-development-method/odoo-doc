# Configuration

Every setting, parameter, default, shipped record, access group, record rule and scheduled job of the Payment Providers domain, with its data type, its default value and its effect.

---

## 1. Capability packages

| Package | Purpose | Depends on |
|---|---|---|
| Payment Engine | The whole domain: entities, payment form, portal routes, state machine, post-processing. | Guided setup, Customer Portal |
| Accounting Payments | Adds the journal, the accounting payment method line, the creation of a Payment on confirmation, the refund wizard, the invoice portal payment and the payment link for invoices. | Payment Engine, Invoicing |
| One connector package per provider | Adds one value to the provider code list, the credential fields, the connector behaviour and the request endpoints. | Payment Engine |
| Custom Payment Modes | Adds the `custom` code, the custom mode list and the wire transfer behaviour. | Payment Engine |
| Demo | Adds the `demo` code and a connector that simulates every outcome. | Payment Engine |
| Website Payment | Adds the website restriction on a provider and the donation flow. | Payment Engine, Website |
| Point of Sale Online Payment | Adds the link between a transaction and a point of sale order. | Payment Engine, Point of Sale |
| Sales | Adds the link between a transaction and a sales order, the order communication type, and the confirmation and invoicing consequences. | Payment Engine, Sales |
| Delivery | Adds the cash-on-delivery custom mode and its availability filter. | Custom Payment Modes, Delivery |
| Click and Collect | Adds the pay-on-site custom mode. | Custom Payment Modes, Website Sales |

The complete list of connector packages, with the code each one adds, is in section 2.

---

## 2. Provider codes and their defaults

Each connector package adds exactly one value to the `code` list of Payment Provider. The base list contains only `none` ("No Provider Set"). When a connector package is uninstalled, every provider record carrying its code falls back to `none`.

| Code | Label | Shipped provider record name | Default payment method codes activated when the provider is put in service |
|---|---|---|---|
| `none` | No Provider Set | (none) | none |
| `adyen` | Adyen | Adyen | `card`, `visa`, `mastercard`, `amex`, `discover` |
| `aps` | Amazon Payment Services | Amazon Payment Services | `card`, `visa`, `mastercard`, `amex`, `discover` |
| `asiapay` | AsiaPay | Asiapay | `card`, `visa`, `mastercard`, `amex`, `discover` |
| `authorize` | Authorize.Net | Authorize.net | `ach_direct_debit`, `card`, `visa`, `mastercard`, `amex`, `discover` |
| `buckaroo` | Buckaroo | Buckaroo | `card`, `ideal`, `visa`, `mastercard`, `amex`, `discover` |
| `custom` | Custom | Wire Transfer (sequence 30) | `wire_transfer` when the custom mode is wire transfer; `cash_on_delivery` when it is cash on delivery; `pay_on_site` when it is pay on site |
| `demo` | Demo | Demo (sequence 40) | `demo` |
| `dpo` | DPO | DPO Pay | `dpo` |
| `ecpay` | ECPay | ECPay | `card`, `wechat_pay`, `cvs`, `bank_transfer`, `mobile_wallet`, `twqr`, `visa`, `mastercard`, `jcb`, `unionpay`, `ok_mart`, `hi_life`, `family_mart`, `ipass_money` |
| `flutterwave` | Flutterwave | Flutterwave | `card`, `mpesa`, `visa`, `mastercard`, `amex`, `discover` |
| `iyzico` | Iyzico | Iyzico | `card`, `mastercard`, `visa`, `amex`, `troy` |
| `mercado_pago` | Mercado Pago | Mercado Pago | `card`, `amex`, `visa`, `mastercard`, `argencard`, `ceconsud`, `cordobesa`, `codensa`, `lider`, `magna`, `naranja`, `nativa`, `oca`, `presto`, `tarjeta_mercadopago`, `shopping`, `elo`, `hipercard` |
| `mollie` | Mollie | Mollie | `card`, `visa`, `mastercard`, `amex`, `discover` |
| `nuvei` | Nuvei | Nuvei | `card`, `visa`, `mastercard`, `amex`, `discover`, `tarjeta_mercadopago`, `naranja` |
| `paymob` | Paymob | Paymob | `card` |
| `paypal` | PayPal | PayPal | `paypal` |
| `payu` | PayU | PayU | `card`, `netbanking`, `upi`, `amex`, `mastercard`, `rupay`, `visa` |
| `razorpay` | Razorpay | Razorpay | `card`, `netbanking`, `upi`, `visa`, `mastercard`, `amex`, `discover` |
| `redsys` | Redsys | Redsys | `card`, `bizum`, `visa`, `mastercard`, `amex`, `diners`, `jcb` |
| `stripe` | Stripe | Stripe | `card`, `bancontact`, `eps`, `ideal`, `p24`, `visa`, `mastercard`, `amex`, `discover` |
| `toss_payments` | Toss Payments | Toss Payments | `bank_transfer`, `card`, `mobile` |
| `worldline` | Worldline | Worldline | `card`, `amex`, `discover`, `mastercard`, `visa` |
| `xendit` | Xendit | Xendit | `card`, `dana`, `ovo`, `qris`, `fpx`, `touch_n_go`, `promptpay`, `linepay`, `shopeepay`, `appota`, `zalopay`, `vnptwallet`, `paynow`, `visa`, `mastercard`, `jcb`, `amex` |

### 2.1 The direct debit provider record

The Payment Engine package ships one further Payment Provider record that no connector package in this domain claims. Its stored values are:

| Field | Value |
|---|---|
| `name` | `SEPA Direct Debit`, the name of the euro-area direct debit scheme; the four-letter short form stands for Single Euro Payments Area and is the name the scheme uses for itself |
| `sequence` | 20 |
| `image_128` | the direct debit logo of that scheme |
| `payment_methods` | exactly one method, the one whose code is `sepa_direct_debit` |
| `code` | `none` |
| `state` | `disabled` |
| `is_published` | false |

Because no capability package of this domain declares the code `sepa_direct_debit` for Payment Provider, the record keeps `code` equal to `none` and `state` equal to `disabled` for the whole life of a system built from this specification. Its observable effects are exactly three, and a replacement must reproduce them:

1. Its card is listed on the provider list screen, sorted by its display sequence 20, that is before Wire Transfer (30) and Demo (40) and after every provider whose sequence is 0.
2. Its card colour is 4 (blue) and its card carries the control that installs the matching capability package, because the record names a package that is not installed (see `calculations.md` section 15).
3. Should such a package ever be installed and set the code to `sepa_direct_debit`, the rule of `entities.md` section 1.6.4 names the resulting accounting payment method line `Online SEPA` instead of the provider name, and the payment method whose code is `sepa_direct_debit` becomes activatable.

The outbound contract of a direct debit collection (the mandate, its reference, the pre-notification delay and the collection file) is not specified in this folder, because no capability package of this domain implements it. The direct debit mandate as an accounting instrument is owned by [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md).

---

## 3. Shipped records

### 3.1 Payment Provider records

Twenty-four Payment Provider records are shipped by the Payment Engine package: one card per connector package of this domain, plus the wire transfer, demo and euro-area direct debit cards. Each connector package then ships one update of its own card, which writes the provider code when that package is installed, so a database in which every connector package is installed has been written by 24 creation records and 23 update records. Each shipped record carries: a name, a logo, the installable package it belongs to, and, for most of them, the list of payment methods that provider supports. All of them are created with `code` equal to `none`, `state` equal to `disabled` and `is_published` false; the code is set by the connector package when it is installed. Three of them carry an explicit display sequence: Single Euro Payments Area Direct Debit 20, Wire Transfer 30, Demo 40; the others use 0 and are therefore sorted by name.

### 3.2 Payment Method records

Two hundred and thirty-two Payment Method records are shipped: two hundred and thirty by the Payment Engine package, one (`wire_transfer`) by the Custom Payment Modes package and one (`demo`) by the Demo package. Each carries: a name, a code, the display sequence 1000, `active` false, an image, the four support flags, and, for most of them, a list of supported countries and a list of supported currencies. Brands carry a primary payment method instead of the country and currency lists.

Every shipped method is inactive; a method becomes active only when a provider that supports it is put in service and the method's code is in that provider's default list, or when an administrator activates it by hand.

One method is special: the method whose code is `unknown` and whose name is `Payment method`. It carries all four support flags at their most permissive value (tokenization and express checkout allowed, manual capture and refund `partial`), it is inactive, and it may never be deleted (PAY-RULE-009). It is the fallback used when a provider reports an instrument that the catalogue does not know, and it is the method hidden from the confirmation page summary.

**Examples of shipped methods, to show the shape of the data.**

| Code | Name | Tokenization | Express checkout | Manual capture | Refund | Countries | Currencies |
|---|---|---|---|---|---|---|---|
| `ach_direct_debit` | ACH Direct Debit | no | no | `none` | `partial` | Puerto Rico, United States | United States dollar |
| `affirm` | Affirm | no | no | `full_only` | `partial` | United States, Canada | Canadian dollar, United States dollar |
| `afterpay` | Afterpay | no | no | `partial` | `partial` | Australia, New Zealand, United States, Canada | Canadian dollar, United States dollar, and others |
| `afterpay_riverty` | AfterPay | no | no | `partial` | `none` | Netherlands, Belgium, Germany, Austria, Finland | euro |
| `akulaku` | Akulaku PayLater | no | no | `none` | `none` | Indonesia | Indonesian rupiah |
| `alipay` | Alipay | no | no | `none` | `partial` | (all) | (all) |
| `amazon_pay` | Amazon Pay | yes | no | `partial` | `partial` | (all) | a fixed list |
| `bancontact` | Bancontact | no | no | `none` | `partial` | Belgium | euro |
| `bank_account` | Bank Account | no | no | `none` | `none` | (all) | (all) |
| `bank_transfer` | Bank Transfer | no | no | `none` | `none` | (all) | (all) |
| `unknown` | Payment method | yes | yes | `partial` | `partial` | (all) | (all) |

A replacement must ship the complete catalogue with, for every method, the four support flags and the country and currency restrictions; those values are what makes the availability filters meaningful.

### 3.3 Scheduled job

| Name | Model | Frequency | Runs as | Active by default | Body |
|---|---|---|---|---|---|
| Payment: Post-process transactions | Payment Transaction | every 10 minutes | the system user | **no** | Runs the post-processing search-and-process loop described in `workflows.md`, section 4.2. |

The job is switched on and off automatically: it is active whenever at least one Payment Provider of the database has a state other than `disabled`, and inactive otherwise. The switch is evaluated when a provider is created, when a provider's state changes, and once at installation time.

### 3.4 Server operation

| Name | Model | Effect |
|---|---|---|
| Start payment onboarding | Payment Provider | Creates an empty configuration-settings working copy and runs the guided setup operation on it. It is the target of the "activate a provider" button of the "no payment method available" notice. |

### 3.5 Screen actions

| Action | Entity | Views | Notes |
|---|---|---|---|
| Payment Providers | Payment Provider | card, list, form | Reachable at the path `payment-providers`. |
| Payment Transactions | Payment Transaction | list, card, form, graph, pivot | Reachable at the path `payment-transactions`. Empty-state text: `There are no transactions to show`. |
| Payment Transactions Linked To Token | Payment Transaction | list, form | Filtered on the token being viewed; creation disabled. |
| Payment Methods | Payment Method | list, card, form | Reachable at the path `payment-methods`. Restricted to primary methods; archived methods are shown; the "available methods" filter is applied by default. |
| Payment Tokens | Payment Token | list, form | Reachable at the path `payment-tokens`. |
| Payment Providers (guided setup) | Payment Provider | form | Opened with the Stripe onboarding context after a return from the provider. |

---

## 4. Settings and parameters

| Setting | Kind | Default | Effect |
|---|---|---|---|
| Portal payment enabled | stored parameter, boolean text | `True` | When off, no invoice may be paid online and the payment link wizard refuses to produce a usable link. Created by the Accounting Payments package and never overwritten on upgrade. |
| Automatic invoicing | stored parameter, boolean text | off | When on, confirming a transaction for a sales order also invoices that order (a down payment invoice when it is only partially paid, a final invoice when it is fully paid). Owned by [../sales/](../sales/README.md). |
| Asynchronous invoice sending | stored parameter, boolean text | off | When on, the invoices created by automatic invoicing are sent by a scheduled job instead of at once. Owned by [../sales/](../sales/README.md). |
| Default invoice email template | stored parameter, integer text | none | The template used when the invoices created by automatic invoicing are sent. Owned by [../sales/](../sales/README.md). |
| Active provider | derived setting | none | The first provider of the company whose state is not `disabled`. Shown on the settings screen with a link to its form. |
| Has enabled provider | derived setting | none | True when at least one provider of the company is `enabled`. Used to show or hide the guided setup. |
| Onboarding payment module | derived setting | none | The provider the guided setup proposes: Razorpay for an Indian rupee company, otherwise Stripe for a Stripe-supported country, otherwise Mercado Pago for a Mercado Pago-supported country, otherwise none. |

---

## 5. Master-data prerequisites

Before a provider can be put in service, the following must exist:

1. **A company** with a country and a currency. The country drives the guided setup and several connectors' restrictions; the currency is the unit of `maximum_amount`.
2. **A bank journal** in that company. Without it the provider has no `journal` and no Payment can be created. The journal is picked automatically as the first bank journal of the company.
3. **A chart of accounts** with an outstanding receipts account (and an outstanding payments account for refunds), or, failing that, an internal transfer account on the company.
4. **An accounting payment method** for the provider's code. It is created automatically when the connector package is installed.
5. **The payment methods** the provider supports, which are shipped inactive and activated by the provider activation step.
6. **A public contact**, because an anonymous visitor browses as it; a token may never be assigned to it (PAY-RULE-024).
7. **A base web address** for the database, because every return address and webhook address is built from it. Two connector operations refuse to run without a publicly reachable address: the PayPal webhook creation refuses a local address with "You must have an HTTPS connection to generate a webhook.", and the Stripe domain verification for the express checkout wallet only succeeds with live credentials ("Please use live credentials to enable Apple Pay.").

---

## 6. Access groups

| Access group | Payment Provider | Payment Method | Payment Token | Payment Transaction | Payment Link Wizard | Payment Capture Wizard | Payment Refund Wizard |
|---|---|---|---|---|---|---|---|
| Public (anonymous visitor) | none | read | read | none | none | none | none |
| Portal (customer) | none | read | read | none | none | none | none |
| Internal user | none | read | read | none | none | read, write, create | none |
| Billing user ("Invoicing") | none | read | read | read, write, create | read, write, create | read, write, create | read, write, create |
| Administrator | read, write, create, delete | read, write, create, delete | read, write, create, delete | read, write, create, delete | read, write, create | read, write, create | read, write, create |

Nobody may delete a wizard record; transient records are cleaned up by the platform.

### 6.1 What each group may do

| Group | Capabilities |
|---|---|
| Public visitor | See the payment form, create a transaction through the public routes (which run with elevated rights), pay, and see the status and confirmation pages. Never sees a provider record or a transaction record directly. |
| Portal customer | Everything the public visitor may do, plus manage their own tokens on the payment method management page, and see the transactions of their own documents through the portal pages of those documents. |
| Internal user | Everything a portal customer may do for themselves. Sees unpublished providers on a payment form. May open a capture wizard, but the underlying capture is refused unless the user also has write access on the transactions. |
| Billing user | Read, create and update transactions; generate payment links; capture, void and refund from an invoice or a Payment; read every token, including those of other contacts, through the additional record rule. |
| Administrator | Everything, including creating, configuring, enabling, publishing and deleting providers, reading and writing credentials, activating payment methods, reading the availability report on a payment form, and running the connector operations (create a webhook, verify a domain, synchronise payment methods, update merchant details, reset credentials). |

### 6.2 Record rules

| Rule | Entity | Applies to | Domain |
|---|---|---|---|
| Access providers in own companies only | Payment Provider | every user | `company parent_of user_enabled_companies` |
| Access transactions in own companies only | Payment Transaction | every user | `company IN user_enabled_companies` |
| Users can access only their own tokens | Payment Token | internal users, portal users, public users | `partner = user.contact` |
| Access tokens in own companies only | Payment Token | every user | `company parent_of user_enabled_companies` |
| Access every token | Payment Token | billing users | always true; it lifts the contact restriction for them |
| Payment Capture Wizard | Payment Capture Wizard | every user | `created_by_user = current user` |

---

## 7. Request endpoint inventory

The generic endpoints are listed in `interfaces.md`, section 2. Every connector adds its own; the complete per-connector list, with the authentication of each, is in `provider-connector-contracts.md`.

---

## 8. Screens and templates shipped

| Template | Purpose |
|---|---|
| Pay page | The generic payment page: breadcrumb, blocking notices, amount and reference summary, payment form. |
| Payment methods page | The payment method management page: breadcrumb and the payment form in validation mode. |
| Payment status page | The page the customer lands on after leaving the provider; polls the status service. |
| Payment confirmation page | The generic landing page with the state block and the summary. |
| Payment form | The list of tokens and payment methods, the expand button, the submit button and the availability report. |
| Token form, method form, form icon, form logo, submit button | The building blocks of the payment form. |
| Company mismatch warning | Shown when the contact may not pay in the company of the document. |
| No payment method available warning | Shown instead of the form when nothing is compatible; extended for administrators. |
| Availability report and availability report button | The diagnostic list of providers and methods with their availability and reason, visible only to administrators. |
| Portal breadcrumb, summary item, state header | Shared fragments. |
| Manage payment methods card | The entry added to the customer portal home; shown only when at least one method allowing tokenization is compatible or the customer already has tokens. |
| Express checkout template | The container of the express checkout buttons. |
| One redirect form template and one inline form template per connector | Registered on the provider record through the four template fields. |

---

## 9. What an administrator must decide

| Decision | Where | Consequence |
|---|---|---|
| Which providers to put in service, and in which state | `state` on each provider | Availability of the whole payment offer (PAY-RULE-076). |
| Whether each provider is visible to anonymous visitors | `is_published` | PAY-RULE-077. |
| Which payment methods to activate per provider | `active` on the method, `payment_methods` on the provider | PAY-RULE-087. |
| Whether customers may save their payment details | `allow_tokenization` | Tokens, recurring charges, one-click payments. |
| Whether payments are captured manually | `capture_manually` | The transaction stops at `authorized`; an employee must capture or void it. Incompatible methods must be deactivated first (PAY-RULE-006). |
| Whether express checkout is offered | `allow_express_checkout` | The express buttons appear on the cart. |
| The countries, currencies and maximum amount | `available_countries`, `available_currencies`, `maximum_amount` | The availability filters. |
| The journal and the outstanding account | `journal`, then the outstanding account on the payment method line | Where the money is held between the payment and the payout. |
| The four customer messages | `pending_message`, `authentication_message`, `done_message`, `cancel_message`, plus `pre_message` | What the customer reads on the status and confirmation pages. |
| For a wire transfer provider, the bank account details | the Recompute pending message operation | The transfer instructions the customer reads. |
| The communication printed on sales orders paid offline | `sales_order_reference_type` | The reference the customer puts on the transfer. |
