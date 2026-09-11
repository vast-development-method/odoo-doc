# Point of Sale — Configuration

Settings, system parameters, numbering sequences, shipped default records, security
groups, the access rights matrix, record rules and scheduled jobs.

---

## 1. Company settings

These live on the company record and apply to every point of sale of that company.

| Setting (storage name) | Type | Default | Meaning |
| --- | --- | --- | --- |
| Update quantities in stock (`point_of_sale_update_stock_quantities`) | Selection | `real` | `closing` "At the session closing" creates one delivery document per destination for the whole session when the session is validated; `real` "In real time" creates one delivery document per order at sale time. The value is frozen onto each session at its creation, so a mid-session change cannot split one session across both behaviors. |
| Self-service invoicing (`point_of_sale_use_ticket_qr_code`) | Boolean | true | Prints the link to the invoice request page on the receipt. |
| Generate a code on the receipt (`point_of_sale_ticket_unique_code`) | Boolean | false | Adds a five-character alphanumeric code to the receipt so that the customer can claim an invoice without being signed in. |
| Receipt portal link display (`point_of_sale_ticket_portal_url_display_mode`) | Selection, required | `qr_code_and_url` | How the link is printed: `qr_code` "QR code", `url` "URL", `qr_code_and_url` "QR code + URL". |
| Default receivable account for the counter (`account_default_pos_receivable_account_id`) | Many to one → Account | From the chart template | The intermediary account used by the closing entry when a payment method does not name its own. |
| Barcode nomenclature (`nomenclature_id`) | Many to one → Barcode Nomenclature | From the chart template or the shipped default | The nomenclature the selling application uses to interpret scanned codes. |
| Default sale tax (`account_sale_tax_id`) | Many to one → Tax | From the chart template | Not specific to this domain, but exposed on the counter settings screen. |

---

## 2. The settings screen

The settings screen edits one configuration at a time. The configuration edited is chosen
as follows: when the screen is opened from a configuration, that configuration; otherwise
the configuration of the acting company with the most recent write instant.

Every field of the screen whose name begins with the three letters `pos` followed by an
underscore is a proxy for the field of the same name on that configuration. The screen
handles them specially:

1. On save, those fields are removed from the values before the generic settings record is
   written, collected per configuration, and written **together** to the configuration in
   one operation. This matters because several configuration constraints span more than one
   field and would fire spuriously if the fields were written one at a time.
2. A proxy whose configuration-side field does not exist is dropped with a warning in the
   server log naming both field names.
3. The write to the configuration is made with the acting context marked as coming from the
   settings screen, which activates the two corrections described in
   [`business-rules.md`](business-rules.md) section 2.12.

Three global switches are raised implicitly:

| Switch | Raised when |
| --- | --- |
| Cash rounding is available platform-wide | Cash rounding is switched on for this configuration. |
| Pricelists are available platform-wide | The use-pricelist flag is switched on for this configuration. |
| The preset menu is visible | The use-presets flag is switched on for this configuration, or any other configuration already uses presets. |

And two are lowered with consequences:

| Switch lowered | Consequence |
| --- | --- |
| Pricelists no longer available platform-wide | Every configuration's use-pricelist flag is cleared. |
| Cash rounding no longer available platform-wide | Every configuration's cash-rounding flag is cleared. |

### 2.1 Capability switches offered by the settings screen

Each is a boolean that installs a capability when raised.

| Switch (storage name) | Capability |
| --- | --- |
| `module_pos_adyen` | A payment terminal integration whose transactions are processed by a named provider; the credentials live on the payment method. |
| `module_pos_stripe` | A second terminal integration of the same shape. |
| `module_pos_viva_com` | A third, which also supports tapping a card on a phone. |
| `module_pos_razorpay` | A fourth. |
| `module_pos_mercado_pago` | A fifth. |
| `module_pos_pine_labs` | A sixth. |
| `module_pos_qfpay` | A seventh. |
| `module_pos_pricer` | Electronic shelf labels showing product prices. |
| `pos_module_pos_discount` | The global discount button. |
| `pos_module_pos_hr` | The employee login screen. |
| `pos_module_pos_restaurant` | Floors, tables, courses and bill splitting. |
| `pos_module_pos_appointment` | Online booking. |
| `pos_module_pos_avatax` | External automatic tax determination. |
| `pos_module_pos_sms` | Sending receipts as text messages. |
| `group_pos_preset` | A visibility switch revealing the preset menu. |

### 2.2 Computed settings fields

| Field | Rule |
| --- | --- |
| Allowed pricelists | The pricelists the configuration may offer, given its currency and company. |
| Selectable categories | The counter categories the configuration may restrict itself to. |
| Order printer enabled | Recomputed when the restaurant capability changes; clearing it clears the printer list. |
| Cash drawer, electronic scale, print through proxy, scan through proxy | Each is recomputed from the presence of a hardware proxy or of directly attached devices, and remains writable. |
| Receipt header and footer | Recomputed when the custom-header flag changes, and remain writable. |
| Default and available fiscal positions | Recomputed when the tax regime selection changes, and remain writable. |
| Available categories | Recomputed when the category restriction changes, and remains writable. |
| Default pricelist and available pricelists | Recomputed when the use-pricelist flag changes, and remain writable. |
| Tip product | Recomputed when tipping is switched on, and remains writable. |

### 2.3 A shortcut on the settings screen

A button opens a new payment method form pre-filled for a chosen terminal provider: the
integration set to terminal, the terminal provider set to the chosen one, the journal set
to the first bank journal of the acting company or one of its parents, the name set to the
word "Bank" followed by the provider's name, and the configuration list set from the
acting context.

---

## 3. System parameters

| Parameter key | Default | Meaning |
| --- | --- | --- |
| `point_of_sale.limited_product_count` | 5000 | How many product templates are loaded into the selling application at session opening. A value that cannot be read as a whole number falls back to the default. Set on installation if not already present. |
| `point_of_sale.limited_customer_count` | 100 | How many partners are loaded at session opening, and the page size of the on-demand partner fetch. Same fallback rule. Set on installation if not already present. |
| `point_of_sale.log_order_data` | `False` | When the value is exactly the text `True`, every transmitted order payload is written in full to the server log. |
| `point_of_sale.use_lna` | absent | When set to any non-empty value, the selling application asks the browser for local-network-access permission before talking to a directly addressed printer. |

---

## 4. Numbering sequences

### 4.1 The shipped session sequence

| Attribute | Value |
| --- | --- |
| Name | POS Session |
| Code | `pos.session` |
| Prefix | The single character `/` |
| Padding | 5 |
| Company | None (shared by every company) |

A session's name is composed as described in [`calculations.md`](calculations.md)
section 11.4. Because the shipped prefix is exactly `/`, the configuration name is
prepended, so a session of a configuration named "Shop" is named, for example,
`Shop/00001`.

When a company-specific sequence with the code `pos.session` exists, it is preferred: the
lookup searches for a sequence with that code whose company is the configuration's company
or no company, ordered by company, taking the first.

### 4.2 The per-configuration sequences

Four sequences are created for every configuration at its creation. All four share:

| Attribute | Value |
| --- | --- |
| Code | `pos.order` |
| Padding | 6 (0 for the device sequence) |
| Company | The configuration's company |
| Implementation | No gap — every value is consumed in order, with no holes |

| Sequence | Name | Used for | Deleted with the configuration |
| --- | --- | --- | --- |
| Order sequence (`order_seq_id`) | `POS order from config #<configuration identifier>` | The prefix and suffix of the order name; the session-unique sequence number | No |
| Backend order sequence (`order_backend_seq_id`) | `POS order backend from config #<configuration identifier>` | The running number inside the receipt number and the tracking number | No |
| Order line sequence (`order_line_seq_id`) | `POS order line from config #<configuration identifier>` | The label of each order line | Yes |
| Device sequence (`device_seq_id`) | `POS device from config #<configuration identifier>` | The device identifier obtained when a device registers | Yes |

A sequence used by a configuration cannot be deleted; see
[`business-rules.md`](business-rules.md) section 9.

### 4.3 The generic order-line sequence

When an order line is created on an order whose configuration has no order-line sequence,
the label falls back to the next value of the platform-wide sequence coded
`pos.order.line`.

### 4.4 Numbering formats

Reproduced from [`calculations.md`](calculations.md) section 11 for convenience:

```formula
receipt_number = last_two_digits_of_current_year ‖ device_identifier ‖ "-" ‖ configuration_identifier ‖ "-" ‖ next_backend_sequence_value
```

```formula
tracking_number = next_backend_sequence_value  modulo  1000
```

```formula
order_name = prefix ‖ " - " ‖ last_hyphen_separated_part_of_receipt_number ‖ optional_suffix
```

Self-ordering prefixes the receipt number differently: the letter `K`, the configuration
identifier and a hyphen for a kiosk order; the letter `S` for a mobile self order.

---

## 5. Shipped records

### 5.1 The tip product

| Attribute | Value |
| --- | --- |
| Name | Tips |
| Internal reference | `TIPS` |
| Product category | Services |
| Weight | 0.01 |
| Available at the counter | false |
| Taxes | none |

### 5.2 A product category

A product category named "Food" is shipped for the demonstration scenarios.

### 5.3 Groupable units

Four shipped units of measure are marked as groupable at the counter, so that lines of the
same product in those units are merged in the cart: units, kilograms, six-packs and
dozens. The last two are only touched when they exist.

### 5.4 Coin and banknote denominations

Thirteen denominations are shipped, each attached to no configuration and therefore
offered on every till: 0.05, 0.10, 0.20, 0.25, 0.50, 1.00, 2.00, 5.00, 10.00, 20.00,
50.00, 100.00 and 200.00. Each is named with its value written with two decimals. All are
created only when absent.

### 5.5 Predefined notes

Four notes are shipped:

| Name | Sequence | Colour |
| --- | --- | --- |
| Wait | 1 | 1 |
| To Serve | 2 | 2 |
| Emergency | 3 | 3 |
| No Dressing | 4 | 4 |

### 5.6 Barcode rules

Four rules are added to the default nomenclature. The pattern grammar is the one of the
barcode domain: a run of digits matches itself, `{N}` marks a digit of the whole part of a
number, `{D}` a digit of its fractional part, and a dot matches any single character.

| Name | Kind | Sequence | Encoding | Pattern |
| --- | --- | --- | --- | --- |
| Cashier Barcodes | `cashier` | 50 | any | `041` |
| Customer Barcodes | `client` | 40 | any | `042` |
| Discount Barcodes | `discount` | 20 | any | `22{NN}` |
| Price Barcodes 2 Decimals | `price` | 14 | thirteen-digit European article number | `23.....{NNNDD}` |

### 5.7 Email template

| Attribute | Value |
| --- | --- |
| Name | Point of Sale: Receipt |
| Description | Sent to customers with the receipt in attachment |
| Entity | Point of Sale Order |
| Subject | `Your <the configuration name> receipt` |
| Sender | The company's formatted email, falling back to the order employee's |
| Recipient | Supplied at send time |
| Language | The customer's language, falling back to the acting user's |
| Auto-delete | false |
| Body | A greeting addressed to the customer by name (or a plain "Hello," when there is none); a thank-you naming the store; a line stating that the receipt for the purchase of the given date and time is attached; a closing salutation; the store name. |

### 5.8 Activity type

An activity type is shipped and used by the stale-session job: an activity scheduled on
the session for its opening user, reminding them to close it.

### 5.9 Client actions

Two reload actions are shipped: one that returns to the counter dashboard menu after the
selling application closes, and one that opens the product menu.

### 5.10 Digest indicator

The periodic digest gains one indicator, "POS Sales", enabled on the shipped default
digest. Its value is the sum of the totals of the orders of the period, in the acting
company, whose state is neither unfinished nor cancelled, dated by the order date. Reading
it requires membership of the point of sale user group; otherwise the computation raises
*"Do not have access, skip this data for user's digest email"* and the indicator is
skipped for that recipient. Clicking the indicator opens the counter sales graph.

### 5.11 Operation types

On installation, the missing counter operation types are created on every warehouse, so
that every warehouse has a point-of-sale outgoing type. That type hides the reservation
method selector, because counter deliveries are always reserved at validation time.

---

## 6. Security groups

| Group | Label | Implies | Shipped members |
| --- | --- | --- | --- |
| Point of Sale / User (`group_pos_user`) | User | — | — |
| Point of Sale / Administrator (`group_pos_manager`) | Administrator | Point of Sale / User, Inventory / User | The system superuser and the shipped administrator |
| Preset Menu (`group_pos_preset`) | Preset Menu | — | — |

The first two belong to a privilege named "Point of Sale" placed in the sales category at
sequence 21; the user group has sequence 10 and the administrator group sequence 20 inside
it. The preset group has sequence 30 and no privilege; it is a pure visibility group,
implied by the ordinary internal-user group when a restaurant scenario with presets is
loaded.

---

## 7. Access rights matrix

Read, write, create and delete permissions per entity and group. A blank cell means the
permission is not granted.

### 7.1 Entities owned by this domain

| Entity | Group | Read | Write | Create | Delete |
| --- | --- | --- | --- | --- | --- |
| Point of Sale Configuration | Point of Sale / User | yes | yes | | |
| Point of Sale Configuration | Platform administrator | yes | yes | | |
| Point of Sale Configuration | Point of Sale / Administrator | yes | yes | yes | yes |
| Point of Sale Session | Point of Sale / User | yes | yes | yes | |
| Point of Sale Order | Point of Sale / User | yes | yes | yes | yes |
| Point of Sale Order | Inventory / User | yes | | | |
| Point of Sale Order Line | Point of Sale / User | yes | yes | yes | yes |
| Point of Sale Pack Operation Lot | Point of Sale / User | yes | yes | yes | yes |
| Point of Sale Payment | Point of Sale / User | yes | yes | yes | yes |
| Point of Sale Payment Method | Point of Sale / User | yes | | | |
| Point of Sale Payment Method | Point of Sale / Administrator | yes | yes | yes | yes |
| Point of Sale Category | Ordinary internal user | yes | | | |
| Point of Sale Category | Point of Sale / Administrator | yes | yes | yes | yes |
| Point of Sale Bill | Point of Sale / User | yes | yes | yes | yes |
| Point of Sale Note | Point of Sale / User | yes | | | |
| Point of Sale Note | Point of Sale / Administrator | yes | yes | yes | yes |
| Point of Sale Preset | Point of Sale / User | yes | | | |
| Point of Sale Preset | Point of Sale / Administrator | yes | yes | yes | yes |
| Point of Sale Printer | Point of Sale / User | yes | | | |
| Point of Sale Printer | Point of Sale / Administrator | yes | yes | yes | yes |
| Point of Sale Order Analysis | Point of Sale / User | yes | | | |

Note that a cashier may create and delete orders, lines, lots, payments and denominations,
but may **not** delete a session (only cancel an unused one), may not create a
configuration, and may not change a payment method.

### 7.2 Entities owned by other domains

| Entity | Group | Read | Write | Create | Delete |
| --- | --- | --- | --- | --- | --- |
| Transfer | Point of Sale / User | yes | yes | yes | yes |
| Stock Move | Point of Sale / User | yes | yes | yes | yes |
| Warehouse | Point of Sale / User | yes | | | |
| Warehouse | Point of Sale / Administrator | yes | | | |
| Location | Point of Sale / Administrator | yes | | | |
| Product Variant | Point of Sale / User | yes | | | |
| Product Template | Point of Sale / User | yes | | | |
| Vendor Pricing | Point of Sale / User | yes | | | |
| Product Combo | Point of Sale / User | yes | | | |
| Product Combo Item | Point of Sale / User | yes | | | |
| Pricelist | Point of Sale / User | yes | | | |
| Pricelist | Point of Sale / Administrator | yes | yes | yes | yes |
| Journal | Point of Sale / User | yes | | | |
| Journal Entry | Point of Sale / User | yes | | | |
| Journal Item | Point of Sale / User | yes | | | |
| Bank Statement Line | Point of Sale / User | yes | yes | yes | |
| Bank Statement Line | Point of Sale / Administrator | yes | yes | yes | yes |
| Accounting Payment Method | Point of Sale / Administrator | yes | | | |
| Accounting Payment Method Line | Point of Sale / Administrator | yes | | | |
| Cash Rounding | Point of Sale / User | yes | | | |
| Barcode Nomenclature | Point of Sale / User | yes | | | |
| Barcode Nomenclature | Point of Sale / Administrator | yes | yes | yes | yes |
| Barcode Rule | Point of Sale / User | yes | | | |
| Barcode Rule | Point of Sale / Administrator | yes | yes | yes | yes |
| Decimal Precision | Point of Sale / User | yes | | | |

A cashier may therefore create and complete transfers (needed to hand goods over) and may
create cash statement lines (needed for a cash in) but may not delete one; only an
administrator may.

### 7.3 Wizard entities

| Entity | Group | Read | Write | Create | Delete |
| --- | --- | --- | --- | --- | --- |
| Point of Sale Details Wizard | Point of Sale / Administrator | yes | yes | yes | |
| Point of Sale Payment Wizard | Point of Sale / Administrator | yes | yes | yes | |
| Point of Sale Close Session Wizard | Point of Sale / User | yes | yes | yes | |
| Point of Sale Daily Sales Reports Wizard | Point of Sale / Administrator | yes | yes | yes | |
| Point of Sale Invoice Wizard | Point of Sale / User | yes | yes | yes | |
| Point of Sale Invoice Wizard | Point of Sale / Administrator | yes | yes | yes | yes |
| Point of Sale Confirmation Wizard | Point of Sale / User | yes | yes | yes | yes |
| Point of Sale Confirmation Wizard | Point of Sale / Administrator | yes | yes | yes | yes |

---

## 8. Record rules

All rules below are non-updatable shipped records.

| Rule | Entity | Applies to | Condition |
| --- | --- | --- | --- |
| Point Of Sale Order | Point of Sale Order | everyone | The order's company is in the acting company set. |
| Point Of Sale Order Line | Point of Sale Order Line | everyone | The line's company is in the acting company set. |
| Point Of Sale Session | Point of Sale Session | everyone | The session's configuration's company is in the acting company set. |
| Point Of Sale Config | Point of Sale Configuration | everyone | The configuration's company is in the acting company set. |
| Point Of Sale Order Analysis multi-company | Point of Sale Order Analysis | everyone | The row's company is in the acting company set. |
| PoS Payment Method | Point of Sale Payment Method | everyone | The method's company is in the acting company set. |
| PoS Payment | Point of Sale Payment | everyone | The payment's company is in the acting company set. |
| Point Of Sale Bank Statement Line POS User | Bank Statement Line | Point of Sale / User | The statement line belongs to a session. A cashier therefore sees only counter cash movements, never ordinary bank statement lines. |
| Point Of Sale Bank Statement Line Accountant | Bank Statement Line | Accounting / Invoicing | Everything. |
| Point Of Sale Bank Statement Accountant | Bank Statement | Accounting / Invoicing | Everything. |
| Invoice POS User | Journal Entry | Point of Sale / User | The entry has at least one counter order attached. A cashier therefore sees only the invoices raised from the counter. |
| Invoice Line POS User | Journal Item | Point of Sale / User | The item's entry has at least one counter order attached. |

---

## 9. Scheduled jobs

### 9.1 The stale-session reminder

The domain does not ship a job of its own; it appends one task to the replenishment
scheduler, and declares one extra unit of work so that the scheduler's progress reporting
stays accurate.

| Attribute | Value |
| --- | --- |
| Runs | Every time the replenishment scheduler runs |
| Selection | Sessions whose opening instant is more than seven days in the past and whose state is not closed |
| Action | For each such session that has no activity at all yet, schedule an activity of the shipped old-session type for the session's opening user |
| Note on the activity | *Your PoS Session is open since <opening instant>, we advise you to close it and to create a new one.* |

When the scheduler runs with its own database cursor, one unit of progress is committed
after the task.

### 9.2 The replenishment trigger at closing

Validating a session triggers the replenishment scheduler on the moves of the session's
transfers, with elevated rights, so that reordering rules see the consumption immediately
rather than at the next scheduler run.

---

## 10. Uninstallation

Uninstalling the capability runs a cleanup hook. Its effect is to remove the records that
would otherwise dangle: the counter operation types created on the warehouses and the
counter-specific data attached to them.

---

## 11. Onboarding scenarios

Five scenarios can be loaded from the dashboard, each creating a ready-to-use
configuration.

| Scenario | Configuration name | Cash journal name | Extra settings | Categories restricted to |
| --- | --- | --- | --- | --- |
| Clothes shop | Clothes Shop | Cash Clothes Shop | — | Upper, Lower, Others |
| Bakery | Bakery Shop | Cash Bakery | — | Breads, Pastries |
| Furniture | Furniture Shop | Cash Furn. Shop | Reuses a shipped cash payment method when the acting user may read it | Miscellaneous, Desks, Chairs |
| Plain retail | The company name | `Cash <company name>` | — | — |
| Bar (restaurant capability) | Bar | Cash Bar | Bill splitting on, restaurant capability on, default screen set to the register | Cocktails, Soft Drinks |
| Restaurant (restaurant capability) | Restaurant | Cash Restaurant | Bill splitting on, restaurant capability on, presets enabled with the shipped eat-in, take-away and delivery presets | Food, Drinks |

Every scenario:

- creates or reuses the point of sale journal and the three standard payment methods;
- hides the created cash journal from the accounting dashboard;
- registers a stable external reference for the configuration, suffixed with the company
  identifier when the acting company is not the main company;
- loads its category data, and its product data when demonstration data is asked for,
  first loading the generic product demonstration data if the product capability carries
  none;
- for the two restaurant scenarios, loads the floor data when absent and attaches the main
  floor and the patio floor;
- for the furniture and restaurant scenarios, when demonstration data is asked for, the
  acting company is the main company and the demonstration session does not exist yet,
  loads a set of demonstration orders and one closed demonstration session.

The dashboard asks the server what to offer, receiving four facts: whether any
configuration exists for the acting company, whether the company has a chart of accounts,
whether the restaurant capability is installed, and whether the acting company is the main
company. A button installs the restaurant capability on demand and reports whether that
capability carries demonstration data.

---

## 12. Client-side asset bundles

The selling application is served as its own asset bundle, separate from the
administrative interface, because it must boot without the full framework and must work
offline.

| Bundle | Purpose |
| --- | --- |
| Base application | The shared core: the styling variables, the framework core without the router and the debug tooling, the icon fonts, the message bus services and the connected-device services. |
| Selling application | The base application plus the luxon date library, the component framework, the barcode scanning library, the barcode parsing services including the Global Standards One parser, the tax helpers of the accounting domain, the report download helpers and every file of the counter's own source tree except the administrative and customer-display parts. The error handlers of the administrative interface are deliberately removed, because error handling at the counter is different. |
| Production bundle | The selling application plus its boot file. This is the bundle whose asset links are cached by the service worker. |
| Customer display bundle | The base application plus the logo, order-line and icon components and the customer-display source tree. |
| Administrative additions | The dashboard styling, the dashboard view, the local-network-access checklist, the payment provider cards and the placeholder list widget, all loaded into the ordinary administrative interface. |

---

## 13. Decimal precisions used

| Precision name | Used for |
| --- | --- |
| Product Unit | Quantities on order lines, the refundable-lines comparison, the report quantity rounding |
| Product Price | Unit prices, the combo distribution rounding, the report total rounding |

Currency rounding is taken from the currency itself, not from a decimal precision.
