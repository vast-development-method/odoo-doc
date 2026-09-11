# Point of Sale — Interfaces

Navigation, views, named remote operations, routes, reports, templates, notifications and
integration contracts.

---

## 1. Navigation

### 1.1 The menu tree

| Menu | Parent | Sequence | Visible to | Opens |
| --- | --- | --- | --- | --- |
| Point of Sale | (top level, sequence 50) | — | Point of Sale / User and Point of Sale / Administrator | — |
| Dashboard | Point of Sale | 1 | inherited | The configuration dashboard |
| Orders | Point of Sale | 10 | inherited | — |
| Orders › Orders | Orders | 2 | Point of Sale / User and Administrator | The order list and form |
| Orders › Payments | Orders | 3 | Point of Sale / User and Administrator | The tender list and form |
| Orders › Sessions | Orders | — | inherited | The session list and form |
| Orders › Customers | Orders | 100 | inherited | The customer list of the accounting domain |
| Orders › Preparation Printers | Orders | 99 | inherited; the menu itself is hidden unless at least one configuration has order printing on | The printer list, board and form |
| Reporting | Point of Sale | 90 | inherited | — |
| Reporting › Session Report | Reporting | 5 | inherited | The daily sales report wizard |
| Reporting › Orders | Reporting | 3 | inherited | The order analysis pivot and graph |
| Reporting › Sales Details | Reporting | 4 | inherited | The sales details wizard |
| Configuration | Point of Sale | 100 | Point of Sale / Administrator only | — |
| Configuration › Settings | Configuration | 0 | Platform administrator only | The settings screen for this capability |
| Configuration › Point of Sales | Configuration | — | inherited | The configuration list |
| Configuration › Products | Configuration | 12 | inherited | — |
| Configuration › Products › Product Categories | Products | — | inherited | The counter category list |
| Configuration › Products › Attributes | Products | — | inherited | The product attribute list |
| Configuration › Payment Methods | Configuration | 3 | Point of Sale / User and Administrator | The payment method list and form |
| Configuration › Presets | Configuration | 3 | Preset Menu group only | The preset list and form |
| Configuration › Coins/Bills | Configuration | 4 | Point of Sale / Administrator | The denomination list and form |
| Configuration › Note Models | Configuration | 11 | inherited | The predefined note list |

### 1.2 Window actions

| Action | Entity | Views | Notes |
| --- | --- | --- | --- |
| Dashboard | Point of Sale Configuration | Board | The landing screen: one card per configuration. |
| Point of Sales | Point of Sale Configuration | List, form | |
| Orders | Point of Sale Order | List, form | |
| Orders of one session | Point of Sale Order | List (without the session column), form | Opened from a session. |
| Point of Sale Analysis | Point of Sale Order | Graph | Opened by the digest indicator. |
| Order Lines | Point of Sale Order Line | List | |
| Order Lines of the day | Point of Sale Order Line | List | Filtered to today. |
| All Sales Lines | Point of Sale Order Line | List, pivot | |
| Sessions | Point of Sale Session | List, form | |
| Payments | Point of Sale Payment | List, form | |
| Payments of a session | Point of Sale Payment | List, form | Grouped by payment method by default; restricted to the captured payments of the session. |
| Payment Methods | Point of Sale Payment Method | List, form | |
| Product Categories | Point of Sale Category | List, form | |
| Presets | Point of Sale Preset | List, form | |
| Coins/Bills | Point of Sale Bill | List, form | |
| Note Models | Point of Sale Note | List | |
| Preparation Printers | Point of Sale Printer | List, board, form | Carries an explanatory empty-state text. |
| Orders Analysis | Point of Sale Order Analysis | Pivot, graph | |
| Sales Details | Point of Sale Details Wizard | Form (dialog) | |
| Session Report | Point of Sale Daily Sales Reports Wizard | Form (dialog) | |
| Settings | Settings record | Form | Opened with this capability selected. |
| Cash register | Bank Statement Line | List, board | Opened from a session; restricted to that session's cash movements. |
| Journal Items | Journal Item | List | Opened from a session; restricted to the items of every accounting document related to the session, grouped by document, filtered to posted. |
| Pickings | Transfer | List | Opened from a session or an order; restricted to its transfers. |
| Customer Invoice(s) | Journal Entry | Form, or list and form when there is more than one | Opened from an order. |
| Create Invoice(s) | Point of Sale Invoice Wizard | Form (dialog, medium) | Opened from a selection of orders. |
| Payment | Point of Sale Payment Wizard | Form (dialog) | Opened from an order. |
| Force Close Session | Point of Sale Close Session Wizard | Form (dialog) | Offered when the closing entry does not balance. |
| Return Products | Point of Sale Order | Form | Opens the newly created refund order. |
| Refunded Order | Point of Sale Order | Form | Opens the order this one refunds. |
| Refund Orders | Point of Sale Order | List, form | Opens the orders that refunded this one. |
| Linked Orders | Point of Sale Order | List | Opened from a preset. |
| Linked point of sale Configurations | Point of Sale Configuration | List | Opened from a preset. |
| Rescue Sessions | Point of Sale Session | Form when there is exactly one, otherwise list and form | Opened from the dashboard. |
| Session | Point of Sale Session | Form, list | Opened from the dashboard's "close session" button. |
| Send Email | Message Composer | Form (dialog) | Opened from a selection of orders in mass-mail mode with the receipt template pre-selected. |

---

## 2. Views

### 2.1 The configuration dashboard

One card per active configuration, showing:

- the configuration name;
- the state of its current session and who opened it;
- how long the current session has been open, in whole days;
- the counted closing balance and the closing date of the last closed session;
- the session statistics document: the opening cash formatted in the configuration
  currency, the start date as abbreviated month and day, the paid-order total with its
  count, and the unfinished-order total with its count;
- a primary button that opens the selling application, or continues the current session;
- a button that opens the current session's form so that it can be closed from the
  administrative interface;
- a badge and a button when rescue sessions exist, showing how many;
- a link to the configuration's orders and to its settings.

When no configuration exists for the acting company, the dashboard shows the onboarding
scenarios instead, and offers to install the restaurant capability. It is told four facts
by the server: whether any configuration exists, whether the company has a chart of
accounts, whether the restaurant capability is installed, and whether the acting company
is the main company.

### 2.2 The configuration form

Grouped into: general settings (name, company, journals, currency); products (category
restriction, available categories, images); pricing (pricelists, discounts, price
control, cash rounding); payment (methods, fast payment); inventory (operation type,
warehouse, ship later, route, shipping policy); devices (hardware proxy, printers, scale,
cash drawer, receipt printer address, customer display background); receipt (header,
footer, basic receipt, automatic printing, preview skipping); and capability switches.

A modal variant of the same form is opened from the dashboard.

### 2.3 The session form

- **Header**: the state as a progress bar over the four states, with buttons to open the
  selling application, to request the closing control and to validate.
- **Buttons**: order count, transfer count (highlighted when a transfer failed), the cash
  register, the journal items, and the payment list.
- **Body**: the configuration, the opening user, the currency, the opening and closing
  instants, the state, the recovery flag, the opening and closing notes, and the cash
  block — starting balance, theoretical closing balance, counted ending balance,
  difference — shown only when the session has cash control.
- **Tabs**: the orders, the cash movements, and the closing entry.
- **Chatter**: the opening and closing cash messages, the edited-orders message, the
  closed-register message and the cash-movement deletions.

### 2.4 The order form

- **Header**: the state as a progress bar, with buttons to register a payment, to create
  an invoice, to return products and to send a receipt by email.
- **Buttons**: the invoice, the transfers, the refunded order, the refund orders and the
  session journal entry.
- **Body**: the receipt number, the tracking number, the order name, the date, the
  session, the configuration, the customer, the employee, the pricelist, the fiscal
  position, the preset, the scheduled time, the shipping date and the invoice status.
- **Tabs**: the lines (product, full name, quantity, unit price, discount, taxes, the two
  amounts, the lots, the customer note, the combo linkage, the refund linkage), the
  tenders (method, amount, date, card details, terminal details) and extra information
  (margin, margin percentage, total cost, notes, print count, edited flag).
- **Chatter**: the payment-change messages and the edit-tracking messages.

### 2.5 The payment method form

The name, the image, the sequence, the journal, the kind (read-only, derived), the
identify-customer flag, the intermediary account, the outstanding account, the
configurations, the company, the integration selector and — depending on the integration —
the terminal provider or the quick response code format. The open sessions using the
method are shown so that the operator understands why the form may be read-only.

### 2.6 List and search views

Every list offers the ordinary grouping and filtering. Notable defaults:

| List | Default grouping or filter |
| --- | --- |
| Payments of a session | Grouped by payment method |
| Journal items of a session | Grouped by accounting document, filtered to posted |
| Order lines of the day | Filtered to today |

---

## 3. Named remote operations

These are the operations the selling application, the customer display and the
self-ordering pages invoke. Each is listed with its inputs and its outputs.

### 3.1 On the configuration

| Operation | Inputs | Output |
| --- | --- | --- |
| Open the selling application | — | An action opening the selling application's address for this configuration, creating a session first when there is none. Refuses for the system superuser. |
| Open the existing session | — | An action opening the current session's form. |
| Open the rescue sessions | — | An action opening the one rescue session, or the list when there is more than one. |
| Register a new device identifier | — | A document carrying the next value of the device sequence. |
| Notify synchronisation | The session identifier, the device identifier, a map from entity to record identifiers | Nothing. Broadcasts the change on this configuration's feed and on every trusted configuration's feed, carrying the changed identifiers and, for the named entities, the records themselves. |
| Read the open orders | A map from entity to a selection condition, and a map from entity to the identifiers the caller already holds | Two maps: the records matching each condition (read with the loading field set of that entity, merged with the order data), and, per entity, the identifiers the caller holds that no longer exist or that refer to a cancelled order. |
| Get the product loading information | — | The number of product templates matching this configuration's condition and the configured limit. |
| Get the limited partner selection | An offset | The partner identifiers of that page of the bounded selection. |
| Get the session statistics | A session | The statistics document described in [`entities.md`](entities.md) section 1.15. |
| Get the dashboard state | — | Whether any configuration exists for the acting company, whether the company has a chart of accounts, whether the restaurant capability is installed, whether the acting company is the main company. |
| Install the restaurant capability | — | Whether the installed capability carries demonstration data. |
| Load the demonstration data | — | Loads the scenario data matching the configuration's external reference, falling back to the furniture scenario. |
| Load an onboarding scenario | Whether to include demonstration data | The identifier of the created configuration. One operation per scenario. |
| Update the customer display | The order to show and a device identifier | Nothing. Broadcasts to that display. |
| Get the customer display data | — | The configuration identifier, its access token, whether a background image exists, the company identifier and the proxy address. |
| Get the records named by external reference | A list of external references | The identifiers of those that exist. |

### 3.2 On the session

| Operation | Inputs | Output |
| --- | --- | --- |
| Load the data | The entities to load (empty means all) | A map from entity to the list of records, each read with that entity's loading field set. Entities the acting user may not read come back empty and the failure is logged. |
| Load the data parameters | — | A map from entity to its loading field set and, per field, its relation description: the field name, the owning entity, whether it is computed, whether it is related, the related entity, the field kind, and — for a single-valued relation — its on-delete behavior, for a one-to-many relation its inverse field name, and for a many-to-many relation its relation table name. |
| Filter the local data | A map from entity to the identifiers the caller holds | Per entity, the identifiers that no longer exist plus the identifiers of records that are archived or unreadable. |
| Set the opening control | The counted amount and the notes | Nothing. Performs the opening transition. |
| Update the closing control state | The notes | Nothing. Stamps the closing instant, stores the notes and posts the closing cash message. |
| Post the closing cash details | The counted cash | Success, or a refusal document. |
| Get the closing control data | — | The document described in [`entities.md`](entities.md) section 2.7. |
| Close the session from the interface | Pairs of payment method identifier and difference amount | Success, or a refusal document carrying the message, whether to redirect, and the identifiers of the orders still unfinished. |
| Get the cash-in and cash-out list | — | The list described in [`entities.md`](entities.md) section 2.7. |
| Try a cash in or cash out | The direction, the amount, the reason, the partner, and extra values carrying the translated movement kind | Nothing. |
| Delete a cash movement | The statement line identifier and the partner performing the deletion | Nothing. |
| Log a partner message | The partner, the action text and the message kind | Nothing. Posts one of three messages: *Action cancelled (<action>)*, *Cash drawer opened (<action>)*, *Cash move deleted: <action>*. |
| Delete an opening-control session | — | Success, or a refusal. |
| Get the attributes by attribute line | — | Per attribute line: its identifier, the attribute name, the attribute display kind, its sequence, and the list of values, each with its name, whether it is custom, its colour, its image, its price supplement and the identifier of the template attribute value. Only attributes that do not create a variant are included. |
| Get the pricelist rules for a set of products | The template identifiers, the variant identifiers and the configuration | The matching pricelist rules and their pricelists. |
| Find a product by barcode | The barcode and the configuration | The product data. |
| Get the total discount | — | The summed discount amount over the closed orders. |
| Get the partners domain | — | An extra condition applied to the partner selection; empty by default, extended by capabilities. |
| Check whether a valid product exists | — | Whether any product is available at the counter with a non-negative list price, excluding the special products, active or archived. |

### 3.3 On the order

| Operation | Inputs | Output |
| --- | --- | --- |
| Synchronise from the interface | A list of order payloads | The saved orders with their lines, lots, custom attribute values, tenders and related accounting documents. |
| Read the orders | A selection condition | The same shape. |
| Read one order by universally unique identifier | The identifier | The same shape. |
| Search paid orders | The configuration, a selection condition, a limit and an offset | A document with, per order, its identifier and the instant of the latest change to one of its lines or to a line refunding one of its lines; plus the total count. Only orders of this configuration and its trusted configurations, in the same currency, that are neither unfinished nor cancelled — or that are refunds and not unfinished. |
| Remove from the interface | A list of order identifiers | The identifiers removed. Only unfinished orders are affected: they are cancelled, their tenders are deleted and they are deleted. |
| Cancel | — | The cancelled orders, read for the interface. |
| Add a payment | The tender values | Nothing. Recomputes the paid amount. |
| Get the preparation change | — | The stored last-preparation-change document. |
| Get the stock documents to print | — | The automatic printing actions of the order's transfers. |
| Send the receipt | The email address, the receipt image and the plain receipt image | Nothing. Sends the shipped template with the images and, when the order is invoiced, the invoice document, attached. |
| Get the existing lots | The company, the configuration and the product | Per lot with a positive quantity in the configuration's source location or a descendant: its identifier, its name and its quantity. |
| Refund | — | An action opening the created refund order. |

### 3.4 On the product template

| Operation | Inputs | Output |
| --- | --- | --- |
| Load products from the counter | The configuration, a selection condition, an offset and a limit | The templates and, with them: their variants; their combos and combo items, including the templates of the combo components; their categories and every ancestor category; the pricelist rules and pricelists that apply; their attribute lines, attribute values, attributes (archived ones included) and exclusions; and their product units. |
| Create a variant from the counter | The chosen attribute values and the configuration | The created variant, read with the counter field set. |

### 3.5 On the partner

| Operation | Inputs | Output |
| --- | --- | --- |
| Get new partners | The configuration, a selection condition and an offset | The partners and their fiscal positions. With an empty condition the next page of the bounded selection is returned; with a non-empty one the whole partner set is searched, one hundred at a time from the offset. |

### 3.6 On the payment method

| Operation | Inputs | Output |
| --- | --- | --- |
| Get the provider status | A list of capability names | The name and installation state of each. |
| Get a quick response code | The amount, the free communication, the structured communication, the currency and the debtor partner | The code as an image encoded in text. Refuses when the method is not configured for codes. |

### 3.7 On the preset

| Operation | Inputs | Output |
| --- | --- | --- |
| Get the available slots | — | The usage map: per scheduled instant rendered as year-month-day hour:minute:second, the identifiers of the orders already booked at it. |

### 3.8 On the restaurant entities

| Operation | Inputs | Output |
| --- | --- | --- |
| Create a floor from the interface | The name, the background colour and the configuration | The created floor with an empty table list. |
| Rename a floor | The new name | Nothing. |
| Deactivate a floor | The session | Deactivates every table and then the floor. Refuses while unfinished orders sit on the floor. |
| Check whether tables still hold unfinished orders | — | True, or a refusal. |
| Set a table's parent | The parent table and the configuration | The table read for the interface; a cycle silently restores the previous parent. |

### 3.9 On the report

| Operation | Inputs | Output |
| --- | --- | --- |
| Get the sales details | A start instant, a stop instant, a set of configurations and a set of sessions | The document described in [`calculations.md`](calculations.md) section 14. |

---

## 4. Routes

### 4.1 The selling application

| Path | Method | Authentication | Purpose |
| --- | --- | --- | --- |
| `/pos/ui/<configuration identifier>` and `/pos/ui/<configuration identifier>/<any sub-path>` | Any | Signed-in user | Opens the selling application for that configuration. Refused for a non-internal user with a not-found answer. Selects the session: an opening-control or opened non-recovery session of the acting user for that configuration; failing that, any opening-control or opened non-recovery session for that configuration. Redirects to the dashboard when the configuration is missing, archived, or already has an active session that this request cannot see. Otherwise takes a no-wait row-level lock on the configuration and opens a session. Renders the application page with: the session information forced to the session's single company, the company's barcode nomenclature, the configuration's fallback nomenclature, whether the request came from the administrative interface, the session identifier, the configuration identifier, the configuration's access token, the configuration's last-data-change instant rendered as year-month-day hour:minute:second, the list of addresses to cache, and the local-network-access flag. The answer is marked as not to be stored by any cache. |
| `/pos/web` and `/pos/ui` | Any | Signed-in user | Compatibility entry points that forward to the route above. |
| `/pos/service-worker.js` | Any | Signed-in user | Serves the offline service worker, declared as executable browser code and allowed to control the whole `/pos` scope. |
| `/pos/ping` | Remote procedure call | Signed-in user | Answers a fixed token. Used as the connectivity probe. |
| `/pos/sale_details_report` | Any | Signed-in user | Renders the sales details document as a Portable Document Format file for a start and stop instant passed as parameters. |

### 4.2 The customer display

| Path | Method | Authentication | Purpose |
| --- | --- | --- | --- |
| `/pos_customer_display/<configuration identifier>/<device identifier>` | Any | Public | Renders the second screen. The access token passed as a parameter is compared with the configuration's token in constant time; a mismatch answers not-found. The page receives the language of the acting user falling back to the company partner's, the public session information, the configuration's display data and the device identifier. |

### 4.3 The invoice request pages

| Path | Method | Authentication | Purpose |
| --- | --- | --- | --- |
| `/pos/ticket` | Read or submit | Public | The request form. On a read with an order's universally unique identifier as a parameter, redirects straight to the validation page with that order's token. On a submit, validates the three fields, looks the order up, and redirects to the validation page. Not listed in the site map. |
| `/pos/ticket/validate` | Read or submit | Public | The validation page, addressed by the order's access token. Behaves as described in [`workflows.md`](workflows.md) section 10.4. Not listed in the site map. |

### 4.4 Self-ordering

| Path | Method | Authentication | Purpose |
| --- | --- | --- | --- |
| `/pos-self/<configuration identifier>` and `/pos-self/<configuration identifier>/<any sub-path>` | Any | Public | The self-ordering application. Listed in the site map. |
| `/pos-self/data/<configuration identifier>` | Remote procedure call | Public | Returns the data the self-ordering application needs. |
| `/pos-self/relations/<configuration identifier>` | Remote procedure call | Public | Returns the relation descriptions for that data. |
| `/pos-self-order/process-order/<device kind>` | Remote procedure call | Public | Submits an order. Sanitises every line as described in [`business-rules.md`](business-rules.md) section 12. |
| `/pos-self-order/get-order/<order identifier>` | Remote procedure call | Public | Reads one order back. |
| `/pos-self-order/validate-partner` | Remote procedure call | Public | Validates and records the customer details captured by the self-ordering application. |
| `/pos-self-order/remove-order` | Remote procedure call | Public | Cancels a self-ordered transaction. |
| `/pos-self-order/send_self_order_receipt` | Remote procedure call | Public | Sends the receipt by email. |
| `/pos-self-order/get-user-data` | Remote procedure call | Public | Returns the visitor's own details. |
| `/pos-self-order/get-slots` | Remote procedure call | Public | Returns the bookable slots of the chosen preset. |
| `/pos-self-order/change-printer-status` | Remote procedure call | Public | Records that the kiosk printer has or has not got paper. |
| `/pos_self_order/kiosk/increment_nb_print/` | Remote procedure call | Public | Increments the print counter of a kiosk order. |
| `/kiosk/payment/<configuration identifier>/<device kind>` | Remote procedure call | Public | Starts the kiosk payment flow. |
| `/pos-self/ping` | Remote procedure call | Public | The self-ordering connectivity probe. |

### 4.5 Online payment of a counter order

| Path | Method | Authentication | Purpose |
| --- | --- | --- | --- |
| `/pos/pay/<order identifier>` | Read | Public | The payment page for one counter order. Not listed in the site map. |
| `/pos/pay/transaction/<order identifier>` | Remote procedure call | Public | Creates the payment transaction for that order. |
| `/pos/pay/confirmation/<order identifier>` | Read | Public | The landing page after the provider returns. |

---

## 5. Reports and printable documents

### 5.1 Sales Details

| Attribute | Value |
| --- | --- |
| Name | Sales Details |
| Entity | Point of Sale Session |
| Kind | Portable Document Format rendered from a template |
| Bound to | The session form, as a printable document |

Content, in order: a header naming the company, the configurations covered, the period and
the session name when exactly one session was requested; the opening and closing notes
when exactly one session is covered; the products sold, grouped by counter category, each
row giving the product name, its barcode, the quantity, the unit price, the discount, the
unit, the total paid and the base amount, with a per-category quantity and total and a
grand quantity and total; the same table for the products refunded; the taxes table for
each of the two sections, giving per tax the base amount and the tax amount, with a
summary row; the payments table, one row per payment method and session, giving the total,
the expected count, the counted amount, the difference and the movement list; the cash
rounding total; the discount count and discount amount; the invoice list per session with
the grand invoice total; and the grand total paid.

### 5.2 The receipt

The receipt is not a server-rendered document; it is composed and printed by the selling
application, and is sent by email as an image. Its content is:

| Block | Content |
| --- | --- |
| Header | The configuration's receipt header when the custom-header flag is set; the company name, address, telephone, email, website, company registry and value-added tax identifier. |
| Order identification | The order name, the receipt number, the tracking number, the date and time, the cashier or employee, and the table and guest count for a restaurant order. |
| Lines | Per line: the quantity and unit, the full product name, the selected attribute values, the customer note, the unit price when the quantity is not one, the discount and the original price when a discount applies, and the line amount — tax-included or tax-excluded according to the tax display setting. Combo components are shown indented under their header. |
| Totals | The tax-excluded total, one line per tax group carrying its receipt label and its amount, the tax total, the rounding line when cash rounding applied, and the grand total. |
| Tenders | One line per tender with the method name and the amount; then the change. Card tenders also print the card brand, the last four digits, the approval code and the terminal's own receipt text. |
| Notes | The order's general customer note. |
| Invoice link | When the company asks for it: the address of the invoice request page as a quick response code, as printed text, or both; and the five-character code when the company asks for one. |
| Footer | The configuration's receipt footer when the custom-header flag is set. |

A second, price-free receipt is printed when the basic-receipt flag is set, suitable as a
gift receipt.

### 5.3 The preparation ticket

Printed on the preparation printers, one per printer, containing only the lines whose
product belongs to one of that printer's categories. It shows the delta against the last
printed state: the added quantities, the removed quantities, the changed notes and the
newly fired courses. It carries the order's tracking number, the table, the course index
and the time.

### 5.4 The bill

A pre-payment rendering of the order, printed when bill printing is enabled. It does not
change the order state and does not increment the print counter.

### 5.5 User Labels

| Attribute | Value |
| --- | --- |
| Name | User Labels |
| Entity | User |
| Kind | Portable Document Format rendered from a template |
| Bound to | The user form, as a printable document |

Prints a badge label for a user, carrying the cashier barcode built from the shipped
cashier barcode rule.

### 5.6 The invoice

The ordinary customer invoice document of the accounts receivable domain, extended so that
the lots captured on the counter order lines are listed: per lot, the product name, the
quantity (the line quantity for a lot-tracked product, one for a serial-tracked product),
the unit name and the lot name. The extension is skipped while the invoice is a draft.

### 5.7 Self-ordering codes

A Portable Document Format sheet of codes is produced for a self-ordering configuration:
one code per active table when the mode is mobile with table service and the restaurant
capability, giving the floor name, the table number and the table's short address;
otherwise six copies of the generic short address. Codes can only be produced in mobile or
browse-only mode (*"QR codes can only be generated in mobile or consultation mode."*) and
table codes require at least one table (*"In Self-Order mode, you must have at least one
table to generate quick response codes"*).

---

## 6. Email and message templates

### 6.1 The receipt email

Described in [`configuration.md`](configuration.md) section 5.7. Sent with the receipt
image attached, named `Receipt-<order name>.jpg`; the price-free receipt when there is one,
named `Receipt-<order name>-1.jpg`; and, when the order is invoiced, the rendered invoice
named `<order name>.pdf`.

A second, inline message is composed when the receipt is sent from the order itself:

> Dear *<customer name, or the word Customer>*,
> Here is your Receipt *(and Invoice, when the order is invoiced)* for *<order name>*
> amounting in *<the total formatted in the selling currency>* from *<company name>*.

### 6.2 Messages posted on the order thread

| Trigger | Body |
| --- | --- |
| The tenders change | `Payment changes:` followed by a bulleted list, one item per change, worded as in [`workflows.md`](workflows.md) section 7.2. |
| A line quantity is reduced with edit tracking on | `<product name>: Ordered quantity: <old quantity>→<new quantity>` |
| A line is deleted with edit tracking on | `<product name>: Deleted line (quantity: <quantity>)` |

### 6.3 Messages posted on the session thread

| Trigger | Body |
| --- | --- |
| Opening control confirmed with a cash method | `Opening cash difference: <amount>`, `Opening cash expected: <amount>`, `Opening cash counted: <amount>`, each on its own line, followed by `Opening control message: <notes>` when notes were given. |
| Opening control confirmed without a cash method but with notes | `Opening control message: <notes>` |
| Closing control | `Closing difference: <amount>`, `Closing expected: <amount>`, `Closing counted: <amount>`, plus the notes line. |
| Closing finished and the acting user has an email address | `Closed Register` |
| Edit tracking on and at least one order edited | `Edited order(s) during the session:` followed by a bulleted list of links to the edited orders. |
| A cash movement is deleted | `Cash move deleted: <cashier name>: <amount>` |
| An action is cancelled from the interface | `Action cancelled (<action>)` |
| The cash drawer is opened manually | `Cash drawer opened (<action>)` |

### 6.4 Messages posted on accounting documents

| Document | Body |
| --- | --- |
| The cash-difference statement entry | `Related Session: <a link to the session>` |
| The invoice of one or more counter orders | `This invoice has been created from the point of sale session:` followed by links to the orders |

### 6.5 The stale-session activity

Scheduled on the session for its opening user, with the note *"Your PoS Session is open
since <opening instant>, we advise you to close it and to create a new one."*

### 6.6 Notifications pushed to a user

| Trigger | Kind | Message |
| --- | --- | --- |
| An invoice of a counter order is reset to draft while its session is open | Danger, sticky | You can't reset this invoice to draft because the point of sale session is still open. Please close the ongoing session first, then try again. |

---

## 7. The change feed

The configuration owns a private channel addressed by its access token. Each message is
addressed as the token, a hyphen and the message name, so that a listener can subscribe to
one kind.

| Message name | Payload |
| --- | --- |
| `SYNCHRONISATION` | The records of the named static entities read with their counter field set, the identifiers changed per entity, the session identifier and the device identifier that caused the change. |
| `CLOSING_SESSION` | The device identifier from the acting context and the session identifier. |
| `ORDER_STATE_CHANGED` | Empty; a signal to refetch. |
| `PAYMENT_STATUS` | The payment result and a document carrying the order and its lines read with the self-ordering field set. |
| `UPDATE_CUSTOMER_DISPLAY-<device identifier>` | The order to display. |

A device ignores a message whose device identifier equals its own, so a device does not
react to its own echo.

---

## 8. Integration contracts

### 8.1 Payment terminals

Specified in [`workflows.md`](workflows.md) section 6. Each terminal capability
contributes one value to the terminal provider selection and implements the five
operations. A capability may also contribute settings to the settings screen and fields to
the payment method.

### 8.2 Quick response code payments

The payment method holds a format. Generating a code takes the amount, a free-text
communication, a structured communication, the currency and the debtor partner, and
returns an image encoded in text, produced by the bank account of the method's journal in
that format. A method also precomputes an amount-less code at configuration time, so that
the selling application can display something while offline; a generation failure leaves
that field empty rather than blocking.

### 8.3 Hardware proxy and directly attached devices

Two paths exist. With a hardware proxy, the selling application addresses the proxy at the
configured address and asks it to print, to open the drawer, to read the scale and to
relay scans. Without one, the application talks to the devices directly: the receipt
printer is addressed at its own address over the local network, and the browser is asked
for local-network-access permission first when the corresponding system parameter is set.

A directly addressed printer whose address is a serial number is reached at a
certificate-bearing hostname derived from that serial number, as described in
[`entities.md`](entities.md) section 1.6.

### 8.4 The offline service worker

Registered for the whole selling-application scope. It caches the asset links of the
production bundle plus the two entry addresses of the configuration, and serves them when
the network is unavailable.

### 8.5 The local database

Orders and master data are held in a local database in the browser. Every change to an
order is written immediately. The store is keyed by the configuration's universally unique
identifier, so two configurations on the same device do not collide.

---

## 9. Import and export

The domain ships no import or export format of its own. Three data contracts are
nevertheless exchanged in structured form:

| Contract | Direction | Shape |
| --- | --- | --- |
| The loading contract | Server to selling application | Per entity, the field list, the relation descriptions and the records. |
| The transmission contract | Selling application to server | A list of order payloads, each carrying the order fields, the line commands, the tender commands, a map of cross-references expressed as universally unique identifiers, and the last-preparation-change document. |
| The self-ordering contract | Public page to server | The same shape, restricted and sanitised as described in [`business-rules.md`](business-rules.md) section 12. |

Standard record import and export apply to the configuration, the categories, the
denominations, the notes, the presets and the payment methods through the ordinary list
views.

---

## 10. The loading contract, entity by entity

At session opening the server answers two questions for every entity in the load list:
**which fields** it exposes and **which records** it sends. This section reproduces both.

### 10.1 How a field set is interpreted

- A field set that is **empty** means *every readable field of the entity*. Three entities
  use this: the configuration, the order and the tender. They are transmitted in full.
- A field set that names fields means *exactly those fields*. Relations are transmitted as
  bare identifiers, never as nested records; the selling application resolves them against
  the other entities it received.
- Alongside the field set, each entity's **relation descriptions** are transmitted: per
  field, its name, the owning entity, whether it is computed, whether it is related, the
  related entity, the field kind, and — for a single-valued relation — its on-delete
  behavior, for a one-to-many relation its inverse field name, for a many-to-many relation
  its relation table name. A field that is not a relation is described with its name, its
  kind and the two computed and related flags only. When the field set is empty, every
  non-manual field is described; when it is not, only the named fields are.
- Records the acting user may not read are silently dropped from the answer.

### 10.2 The field sets

| Entity | Field set |
| --- | --- |
| Session | identifier, name, opening user, configuration, opening instant, closing instant, payment methods, state, deferred-stock flag, starting balance, access token |
| Configuration | every field, plus the computed extras of section 10.3 |
| Order | every field |
| Order line | quantity, selected attributes, custom values, unit price, universally unique identifier, tax-excluded amount, tax-included amount, order, product note, price type, product, discount, taxes, lots, customer note, refunded quantity, price extra, full product name, refunded line, combo parent, combo children, combo item, refunding lines, extra tax data, last write instant |
| Lot on a line | lot name, order line, last write instant |
| Tender | every field |
| Payment method | identifier, name, cash flag, terminal provider, identify-customer flag, kind, image, sequence, integration, default quick response code |
| Printer | identifier, name, proxy address, printed categories, printer type, direct printer address |
| Counter category | identifier, name, parent, children, last write instant, has-image flag, colour, sequence, availability until, availability after |
| Denomination | identifier, name, value |
| Preset | identifier, name, pricelist, fiscal position, return-mode flag, colour, has-image flag, last write instant, identification requirement, timing flag, capacity, interval length, attendances |
| Working schedule attendance | the fields the preset needs to generate slots |
| Company | identifier, currency, email, website, company registry, value-added tax identifier, name, telephone, partner, country, state, tax rounding scope, barcode nomenclature, self-service invoicing flag, receipt code flag, receipt link display mode, street, city, postal code, fiscal country |
| User | identifier, name, partner, and the derived role (see section 10.3) |
| Partner | identifier, name, street, second street line, city, state, country, value-added tax identifier, language, telephone, postal code, email, barcode, last write instant, pricelist, parent name, counter address, invoice emails, fiscal position, company flag, receivable account |
| Product template | identifier, display name, standard price, product category, counter categories, taxes, barcode, name, list price, favourite flag, internal reference, to-be-weighed flag, unit, sales description, description, tracking, kind, service tracking, storable flag, last write instant, colour, counter display order, available-at-the-counter flag, attribute lines, active flag, small image, combos, variants, public description, suggested products, sequence, tags, currency, cost currency |
| Product variant | identifier, list price, display name, template, variant values, currency, cost currency, template attribute values, barcode, tags, internal reference, standard price |
| Product attribute | name, display kind, variant-creation mode |
| Template attribute line | display name, attribute, values, active flag |
| Template attribute value | attribute, attribute line, attribute value, price supplement, name, custom flag, colour, image, exclusions |
| Template attribute exclusion | excluded values, template attribute value |
| Custom attribute value | custom value, template attribute value, order line, last write instant |
| Combo | identifier, name, items, base price, free quantity, maximum quantity, currency |
| Combo item | identifier, combo, product, extra price, currency |
| Tax | identifier, name, price-included behavior, base-inclusion flag, base-affected flag, negative-factor flag, computation kind, children, amount, company, sequence, tax group, fiscal positions |
| Tax group | identifier, name, receipt label |
| Product unit | identifier, barcode, product, unit |
| Decimal precision | identifier, name, digits |
| Unit of measure | identifier, name, factor, groupable flag, hierarchy path, rounding, plus whatever extra fields the loaded taxes need for their computation |
| Country | identifier, name, code, value-added tax label |
| Country state | identifier, name, code, country |
| Language | identifier, name, code, flag image address, display name |
| Product category | identifier, name, parent, removal strategy |
| Pricelist | identifier, name, display name, currency, rules |
| Pricelist rule | product template, product, pricelist, surcharge, discount, rounding step, minimum margin, maximum margin, company, currency, start date, end date, computation kind, fixed price, percentage, base pricelist, base, product category, minimum quantity |
| Cash rounding | identifier, name, rounding step, rounding method, strategy |
| Fiscal position | identifier, name, display name, tax map, taxes |
| Operation type | identifier, uses-created-lots flag, uses-existing-lots flag, has-documents-to-print flag |
| Currency | identifier, name, symbol, position, rounding step, rate, decimal places, numeric code |
| Predefined note | name, colour |
| Product tag | name, counter description, colour, has-image flag, last write instant |
| Installed capability | identifier, name, state |
| Accounting entry | identifier, name |
| Account | identifier, non-trade flag |
| Removal strategy | method |
| Restaurant floor | name, background colour, tables, sequence, configurations, floor background image, active flag |
| Restaurant table | table number, width, height, horizontal position, vertical position, parent, shape, floor, colour, seats, active flag |
| Restaurant course | universally unique identifier, fired flag, order, lines, index, last write instant |

### 10.3 Computed extras on the configuration

The configuration record transmitted to the selling application carries, beyond its
stored fields:

| Extra | Value |
| --- | --- |
| Server version | The platform's version information. |
| Base address | The configuration's base address. |
| Server date | The instant the data was read, used as the basis for incremental loading. |
| Cash movement permission | Whether the acting user may record a cash in or out. |
| Cash deletion permission | Whether the acting user may delete a cash movement. |
| Special products | The identifiers of the products the counter treats specially (the tip product). |
| Product default values | The default values of the product fields the loaded taxes need for their computation. |
| Pricelist | Blanked when the configuration does not use pricelists, so that the application falls back to list prices. |
| Value-added tax regime flag | Whether the company's country is a member of the European economic area. |

The user record carries one extra: a role of either `manager` or `cashier`, derived from
whether the user belongs to the configuration's administrator group. The raw group list
used to derive it is removed before transmission.

### 10.4 The selection conditions that depend on already-loaded data

Several conditions are expressed in terms of what has already been loaded, which is why
the load order matters:

| Entity | Depends on |
| --- | --- |
| Order line | The loaded orders |
| Lot on a line | The loaded order lines |
| Custom attribute value | The loaded order lines |
| Tender | The loaded orders |
| Counter category | The loaded printers (every category routed to a printer is loaded even under a restriction) |
| Partner | The loaded orders (their customers are always included) |
| Fiscal position | The loaded presets and the loaded partners |
| Template attribute line | The loaded product templates |
| Template attribute exclusion | The loaded product templates and the loaded template attribute values |
| Product unit | The loaded product variants |
| Currency | The loaded pricelists |
| Account | The receivable accounts of the loaded partners |

### 10.5 Currency conversion at load time

Three loaded amounts are converted into the selling currency at load time, at today's
rate, each from its own currency: a product template's list price and its standard price
(the latter from the cost currency), a combo's base price and a combo item's extra price.
A record already expressed in the selling currency is left untouched.

### 10.6 On-demand loading

| Trigger | What is fetched |
| --- | --- |
| A product is scanned or searched that was not loaded | The matching templates and, with them, their variants, their combos and combo items (including the templates of the components), their product categories and every ancestor, the applicable pricelist rules and pricelists, their attribute lines, values, attributes (archived ones included) and exclusions, and their product units. |
| A customer is searched that was not loaded | The matching partners and their fiscal positions. |
| A variant must be created from a chosen attribute combination | The created variant, read with the variant field set. |
| The open orders of a trusted configuration must be refreshed | The matching records per entity, plus the identifiers to drop. |
