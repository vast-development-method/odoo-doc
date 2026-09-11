# Point of Sale — Entities

This file specifies every entity of the point of sale domain: its purpose, its lifecycle,
its complete field table, its relations, its uniqueness rules, its defaults, its computed
values with the rules that produce them, its ordering, its display rule, its archival
behavior and its multi-company behavior.

Field tables use three columns:

- **Field (storage name)** — the human name followed by the storage name in code font.
- **Type** — the data type. `Many to one`, `One to many`, `Many to many`, `Text`,
  `Character`, `Integer`, `Decimal`, `Monetary`, `Boolean`, `Date`, `Date and time`,
  `Selection`, `Binary image`, `Structured document` (a stored tree of named values, in
  JavaScript Object Notation form).
- **Meaning and rules** — everything else: required, default, computed and from what,
  stored or not, read-only, copy behavior, tracking, company scoping, indexing,
  on-delete behavior, selection values with their labels.

Unless stated otherwise a field is optional, writable, stored, copied when the record is
duplicated and not indexed.

---

## 1. Point of Sale Configuration

**Transport name** `pos.config`, **table** `pos_config`.

### 1.1 Purpose

A Point of Sale Configuration is one till: one selling point with its own device
settings, its own journals, its own payment methods, its own product restriction and its
own sequences. Every session, every order and every receipt belongs to exactly one
configuration. A company may hold any number of configurations; a configuration belongs to
exactly one company.

### 1.2 Lifecycle

1. **Created** — by an administrator, or by one of the shipped onboarding scenarios
   (clothes shop, bakery, furniture shop, plain retail). On creation the system creates
   four numbering sequences for it, installs any optional capability its `module_` flags
   request, extends the implied security groups required by its `group_` flags, and
   refreshes the visibility of the preparation printer menu.
2. **In use** — sessions are opened and closed against it. While a session is not closed,
   a restricted set of fields cannot be changed (see
   [`business-rules.md`](business-rules.md)).
3. **Archived** — the active flag is cleared. Archiving is refused while a session is
   open.
4. **Deleted** — allowed; the order-line sequence and the device sequence are deleted with
   it, the order sequence and the backend order sequence are not.

### 1.3 Field table — identification and general

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Point of Sale name (`name`) | Character | Required. The internal identification of the till. Used as the display name, as the fallback prefix of order names and, when the session sequence has no prefix of its own, as the prefix of the session name. |
| Active (`active`) | Boolean | Default true. Clearing it archives the configuration. Cannot be cleared while a session of this configuration is not closed. Setting it back to true is always allowed, even with an open session. |
| Company (`company_id`) | Many to one → Company | Required. Defaults to the company of the acting user. Scopes payment methods, journals, pricelists, warehouses and the record rule. |
| Universally unique identifier (`uuid`) | Character | Read-only, not copied. Defaults to a freshly generated universally unique identifier. Used by the selling application to keep client-generated data from colliding between configurations. |
| Access token (`access_token`) | Character | Defaults to the first sixteen hexadecimal characters of a freshly generated universally unique identifier. Used to address the change-feed channel of this configuration and to authenticate the customer display. |
| Last data change (`last_data_change`) | Date and time | Computed and stored. Set to the current transaction time whenever any of the following change: use-pricelist flag, default pricelist, available pricelists, payment methods, category restriction flag, available categories, employee-login flag, global-discount flag, tip-product flag, default preset, online-booking flag, cash-rounding flag, rounding method, only-round-cash flag. The selling application compares this value with its own cached copy to decide whether it must reload master data. |

### 1.4 Field table — accounting

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Point of Sale journal (`journal_id`) | Many to one → Journal | The journal in which the session closing entry and the invoice-payment entries are posted. Restricted to journals of type miscellaneous or sale. Company-checked. On delete: restrict. Default: the journal whose code is the four letters `POSS` in the acting company, created on the spot if it does not exist (name "Point of Sale", type miscellaneous). |
| Invoice journal (`invoice_journal_id`) | Many to one → Journal | The journal used to create invoices from counter orders. Restricted to journals of type sale. Company-checked. Default: the first sale journal of the acting company. |
| Currency (`currency_id`) | Many to one → Currency | Computed and stored, computed with elevated rights. Equals the currency of the point of sale journal when that journal carries one, otherwise the currency of the journal's company, otherwise the currency of the configuration's company. Every amount handled by the selling application is expressed in this currency. |
| Cash rounding enabled (`cash_rounding`) | Boolean | When set, the payable total of every order is rounded to a cash denomination. |
| Rounding method (`rounding_method`) | Many to one → Cash Rounding | The cash rounding definition to apply: its rounding step, its rounding method and its profit and loss accounts. Must use the add-a-rounding-line strategy when cash rounding is enabled. |
| Only apply rounding on cash (`only_round_cash_method`) | Boolean | When set, the rounding is applied only when at least one cash tender is used on the order, and only to the part of the amount settled in cash. |
| Closing entry by product (`is_closing_entry_by_product`) | Boolean | When set, the sales lines of the session closing entry are broken down per product instead of being aggregated per account, sign, tax set and base tag set; quantities are then also carried on those lines. |

### 1.5 Field table — inventory

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Operation type (`picking_type_id`) | Many to one → Operation Type | Required. On delete: restrict. Restricted to outgoing operation types of a warehouse belonging to the acting company. Default: the point-of-sale operation type of the first warehouse of the acting company. Determines the source location of counter deliveries and, through its return counterpart, the destination of refunds. |
| Warehouse (`warehouse_id`) | Many to one → Warehouse | Computed and stored, writable, precomputed. On delete: restrict. Equals the warehouse of the operation type when it has one, otherwise the first warehouse of the acting company. |
| Ship later (`ship_later`) | Boolean | Enables capturing a shipping date on an order; deliveries are then created through the procurement rules instead of being validated immediately. |
| Deferred route (`route_id`) | Many to one → Route | The route used for products delivered later. |
| Shipping policy (`picking_policy`) | Selection | Required, default `direct`. Values: `direct` "As soon as possible", `one` "When all products are ready". Governs how the deferred delivery is scheduled. |

### 1.6 Field table — devices and peripheral behavior

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Is a point-of-sale box (`is_posbox`) | Boolean | Declares that a hardware proxy box is attached. |
| Other devices (`other_devices`) | Boolean | Declares that devices are attached directly, without a hardware proxy box. |
| Proxy address (`proxy_ip`) | Character, at most 45 characters | Hostname or internet protocol address of the hardware proxy. Auto-detected when empty. |
| Cash drawer (`iface_cashdrawer`) | Boolean | Opens the cash drawer automatically at the end of a cash payment. |
| Electronic scale (`iface_electronic_scale`) | Boolean | Enables the weighing integration for products flagged to be weighed. |
| Print through proxy (`iface_print_via_proxy`) | Boolean | Sends receipts to the hardware proxy instead of the browser print dialog. |
| Scan through proxy (`iface_scan_via_proxy`) | Boolean | Enables a remotely connected barcode scanner and card reader. |
| Large scrollbars (`iface_big_scrollbars`) | Boolean | Widens the scrollbars for imprecise industrial touchscreens. |
| Receipt printer address (`epson_printer_ip`) | Character | The local internet protocol address of a directly addressed receipt printer, or its serial number. When a serial number is entered (a value with no dot in it) it is converted, before storing, into a certificate-bearing hostname: the serial number is hashed with the two-hundred-fifty-six-bit secure hash algorithm, the digest is encoded in base thirty-two, the padding characters are stripped, the result is lower-cased and the fixed certificate domain is appended. A value containing a dot is stored unchanged. |
| Order printer enabled (`is_order_printer`) | Boolean | Enables preparation printing. Clearing it also clears the printer list. |
| Order printers (`printer_ids`) | Many to many → Point of Sale Printer | The preparation printers of this till, through the relation table `pos_config_printer_rel`. |

### 1.7 Field table — receipt, display and interface

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Automatic receipt printing (`iface_print_auto`) | Boolean | Default false. Prints the receipt automatically when the order is validated. |
| Skip preview screen (`iface_print_skip_screen`) | Boolean | Default true. Skips the receipt preview when the receipt can be printed automatically. |
| Tax display (`iface_tax_included`) | Selection | Required, default `total`. Values: `subtotal` "Tax-Excluded Price", `total` "Tax-Included Price". Governs which unit price is shown on the product buttons and the order lines. |
| Group products by category (`iface_group_by_categ`) | Boolean | Lays the product grid out grouped by category. |
| Show product images (`show_product_images`) | Boolean | Default true. |
| Show category images (`show_category_images`) | Boolean | Default true. |
| Custom header and footer (`is_header_or_footer`) | Boolean | Enables the two free-text receipt blocks below. |
| Receipt header (`receipt_header`) | Text | Inserted at the top of the printed receipt. Writable only by an administrator. |
| Receipt footer (`receipt_footer`) | Text | Inserted at the bottom of the printed receipt. Writable only by an administrator. |
| Basic receipt (`basic_receipt`) | Boolean | Prints a second receipt without prices, suitable as a gift receipt. |
| Customer display background image (`customer_display_bg_img`) | Binary image, at most 1920 by 1920 | Background of the second screen shown to the customer. |
| Customer display background image name (`customer_display_bg_img_name`) | Character | File name of the above. |
| Note models (`note_ids`) | Many to many → Point of Sale Note | The predefined notes offered on this till. |
| Predefined coins and banknotes (`default_bill_ids`) | Many to many → Point of Sale Bill | The denominations offered as one-touch buttons on the payment screen. |
| Fallback barcode nomenclature (`fallback_nomenclature_id`) | Many to one → Barcode Nomenclature | A second nomenclature tried when the company nomenclature does not match a scanned code. |

### 1.8 Field table — pricing, taxes and customers

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Use a pricelist (`use_pricelist`) | Boolean | When cleared, the default pricelist is not sent to the selling application at all and product list prices are used. |
| Default pricelist (`pricelist_id`) | Many to one → Pricelist | The pricelist used when no customer is selected, or when the selected customer has no pricelist of their own. |
| Available pricelists (`available_pricelist_ids`) | Many to many → Pricelist | The pricelists the cashier may switch to. |
| Tax regime selection (`tax_regime_selection`) | Boolean | Enables choosing a fiscal position on the order. When cleared, the fiscal position list is emptied on write. |
| Fiscal positions (`fiscal_position_ids`) | Many to many → Fiscal Position | The fiscal positions the cashier may switch to. |
| Default fiscal position (`default_fiscal_position_id`) | Many to one → Fiscal Position | Applied to every new order. When the tax regime selection is enabled, it is automatically added to the available list on write. When the fiscal position is archived, this field is cleared on every configuration that refers to it. |
| Line discounts (`manual_discount`) | Boolean | Default true. Allows a per-line percentage discount. |
| Restrict price modifications to administrators (`restrict_price_control`) | Boolean | Only users in the administrator group may change a unit price on an order. |
| Margins and costs visible to everyone (`is_margins_costs_accessible_to_every_user`) | Boolean | Default false. When false, only administrators see cost and margin in the product information panel. |
| Restrict categories (`limit_categories`) | Boolean | Restricts the product catalogue to the selected category trees. |
| Available categories (`iface_available_categ_ids`) | Many to many → Point of Sale Category | The category trees the till displays. Empty means no restriction. |
| Tip product enabled (`iface_tipproduct`) | Boolean | Enables tipping. |
| Tip product (`tip_product_id`) | Many to one → Product Variant | The product used to carry a tip amount on the order and the receipt. Default: the shipped tip product if it exists and belongs to the acting company, otherwise the first product whose internal reference is the four letters `TIPS`. When tipping is switched on and the field is emptied, the shipped tip product is restored; if that product is missing the write is refused. |
| Trusted configurations (`trusted_config_ids`) | Many to many → Point of Sale Configuration, self-referencing through `pos_config_trust_relation` | Configurations whose open orders this till may see and settle. All of them must use the same currency as this configuration. |

### 1.9 Field table — presets and service modes

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Use presets (`use_presets`) | Boolean | Enables the service-mode selector. |
| Default preset (`default_preset_id`) | Many to one → Point of Sale Preset | Applied to every new order when presets are enabled. Automatically added to the available list after any write. |
| Available presets (`available_preset_ids`) | Many to many → Point of Sale Preset | The presets the cashier may choose from. |

### 1.10 Field table — session and cash control

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Advanced cash control (`cash_control`) | Boolean | Computed, not stored. True when at least one of the configuration's payment methods is of the cash kind. Governs whether an opening count and a closing count are requested. |
| Set maximum difference (`set_maximum_difference`) | Boolean | Enables the authorised-difference limit below. |
| Authorised difference (`amount_authorized_diff`) | Decimal | The largest difference between the counted cash and the theoretical cash that a non-administrator may post at closing. Beyond it the closing is refused and the cashier is told to call an administrator. Only transmitted to the selling application when the maximum-difference flag is set. |
| Sessions (`session_ids`) | One to many → Point of Sale Session, inverse `config_id` | Every session ever opened on this till. |
| Current session (`current_session_id`) | Many to one → Point of Sale Session | Computed, not stored. The most recent non-closed, non-rescue session, or empty. |
| Current session state (`current_session_state`) | Character | Computed, not stored. The state value of the current session, or empty. |
| Has active session (`has_active_session`) | Boolean | Computed, not stored. True when any non-closed session exists, rescue sessions included. |
| Number of rescue sessions (`number_of_rescue_session`) | Integer | Computed, not stored. Count of non-closed rescue sessions. |
| Current session responsible (`current_user_id`) | Many to one → User | Computed, not stored. The user who opened the current non-rescue session. |
| Current session user name (`pos_session_username`) | Character | Computed, not stored. The name of that user. |
| Current session state label (`pos_session_state`) | Character | Computed, not stored. Duplicate of the state value, used by the dashboard. |
| Current session duration (`pos_session_duration`) | Character | Computed, not stored. The whole number of days between the session opening instant and now; zero when the session has not started. |
| Last session closing cash (`last_session_closing_cash`) | Decimal | Computed, not stored. The counted closing balance of the most recently closed session of this till, ordered by closing instant descending. Zero when there is none. |
| Last session closing date (`last_session_closing_date`) | Date | Computed, not stored. The closing instant of that session, expressed in the acting time zone as a date. |
| Session statistics (`statistics_for_current_session`) | Structured document | Computed, not stored. See section 1.15. |

### 1.11 Field table — payment

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Payment methods (`payment_method_ids`) | Many to many → Point of Sale Payment Method | Not copied. The tenders offered on this till. Default: see section 1.13. At least one is required to open the till. All must belong to the configuration's company. All must be expressed in the configuration's currency. |
| Fast payment validation (`use_fast_payment`) | Boolean | Shows one-touch validate buttons on the product screen. Automatically cleared when the fast payment method list becomes empty. |
| Fast payment methods (`fast_payment_method_ids`) | Many to many → Point of Sale Payment Method, through `pos_payment_method_config_fast_validation_relation` | Computed and stored, writable. Recomputed from the payment method list by dropping any method that is no longer offered on the till. |
| Automatically validate terminal payments (`auto_validate_terminal_payment`) | Boolean | Default true. Validates the order as soon as the payment terminal reports success. |

### 1.12 Field table — sequences, capability flags and groups

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Order sequence (`order_seq_id`) | Many to one → Sequence | Read-only, not copied. Created with the configuration. |
| Backend order sequence (`order_backend_seq_id`) | Many to one → Sequence | Read-only, not copied. Created with the configuration. Supplies the running number inside the receipt number. |
| Order line sequence (`order_line_seq_id`) | Many to one → Sequence | Read-only, not copied. Created with the configuration. Supplies the line label. Deleted with the configuration. |
| Device sequence (`device_seq_id`) | Many to one → Sequence | Read-only, not copied. Created with the configuration. Supplies device identifiers. Deleted with the configuration. |
| Restaurant capability (`module_pos_restaurant`) | Boolean | Requests the restaurant capability. Cannot be changed while a session is open. |
| Automatic tax mapping capability (`module_pos_avatax`) | Boolean | Requests the external automatic tax determination capability. |
| Global discounts capability (`module_pos_discount`) | Boolean | Requests the global discount button. |
| Online booking capability (`module_pos_appointment`) | Boolean | Requests the online booking capability. |
| Employee login capability (`module_pos_hr`) | Boolean | Requests the employee login screen. |
| Text message capability (`module_pos_sms`) | Boolean | Requests sending receipts as text messages. |
| Administrator group (`group_pos_manager_id`) | Many to one → Security Group | Default: the point of sale administrator group. Transmitted to the selling application so that it can tell administrators from cashiers. |
| User group (`group_pos_user_id`) | Many to one → Security Group | Default: the point of sale user group. Transmitted for the same purpose. |
| Full accounting installed (`is_installed_account_accountant`) | Boolean | Computed, not stored. True when the full accounting capability is installed. |
| Company has a chart of accounts (`company_has_template`) | Boolean | Computed, not stored. True when the root company already has accounting data or a chart template. |
| Track order edits (`order_edit_tracking`) | Boolean | Default false. Records quantity reductions and line deletions in the order's message thread and lists the edited orders in the session's thread at closing. |

### 1.13 Default payment methods

When a configuration is created without an explicit payment method list, the default is
computed as follows.

1. Build a base condition: the method belongs to the acting company (or one of its
   parents, per the standard company condition), the method does not identify the customer
   (its identify-customer flag is false), and either the method has no journal or the
   journal has no currency of its own or the journal currency is the company currency.
2. Collect every non-cash method matching the base condition.
3. Collect at most one cash method matching the base condition that is not yet attached to
   any configuration.
4. If both collections are empty, create a journal and a set of payment methods (see
   section 1.14) and return the created methods.
5. Otherwise return the union of the two collections.

### 1.14 Creating the journal and the standard payment methods

This procedure runs when a configuration is created by an onboarding scenario or when no
usable payment method exists.

1. Ensure the point of sale journal exists: search for a journal with code `POSS` in the
   acting company; create it if missing with name "Point of Sale" and type miscellaneous.
2. **Cash method.** If a reference to an existing cash method was supplied and the acting
   user may read it, reuse it. Otherwise create a cash journal (name "Cash" unless
   overridden, type cash, acting company) whose default account is the account of type
   cash and short-term deposits named "Cash" in the root company when such an account
   exists, then create a cash payment method with the same name pointing at that journal.
   Register the external reference on the created method when one was supplied.
3. **Bank method.** Search for an existing payment method whose journal is of type bank in
   the acting company or any of its parents. If none exists, take the first bank journal of
   those companies; if there is none, refuse with: *"Ensure that there is an existing bank
   journal. Check if chart of accounts is installed in your company."* Create a payment
   method named "Card" on that journal, with sequence 1, whose outstanding account is the
   chart template's inbound outstanding payment account, falling back to the company
   transfer account.
4. **Customer account method.** Search for an existing payment method with no journal in
   the acting company or any of its parents. If none exists, create one named "Customer
   Account", with sequence 2 and the identify-customer flag set.
5. Return the journal and the three methods.

### 1.15 Session statistics

The session statistics document is recomputed on read for the current non-closed,
non-rescue session; it is false when there is none. Its shape is:

- `cash.raw_opening_cash` — the opening balance as a number.
- `cash.opening_cash` — the opening balance formatted in the configuration currency.
- `date.is_started` — whether the session has an opening instant.
- `date.start_date` — the opening instant rendered as abbreviated month and day in the
  acting time zone.
- `orders.paid` — false when there is nothing to report, otherwise a document with
  `amount` (the sum of the totals of all paid and posted orders, refunds included),
  `count` (see below) and `display` (the amount formatted in the configuration currency
  followed by the count and the word "order" or "orders").
- `orders.draft` — false when there is no unfinished order, otherwise the same shape over
  the unfinished orders.

The paid count is computed as follows: group the refund orders by the order they refund
and total the absolute values of their totals; then count the non-refund paid or posted
orders whose total is *not* exactly equal to the amount refunded against them. In other
words, an order that has been refunded in full stops being counted.

### 1.16 Order and receipt numbering

Every counter order receives three identifiers.

```formula
receipt_number = last_two_digits_of_current_year ‖ device_identifier ‖ "-" ‖ configuration_identifier ‖ "-" ‖ next_backend_sequence_value
```

```formula
tracking_number = next_backend_sequence_value  modulo  1000
```

- `next_backend_sequence_value` is the next value of the backend order sequence of the
  configuration. That sequence is created with six digits of padding, no-gap
  implementation and the code `pos.order`.
- `device_identifier` is the identifier obtained by the device when it registered; it is
  the string `0` for a device that never registered.
- `configuration_identifier` is the numeric identifier of the configuration record.
- The tracking number is the running number reduced modulo one thousand and rendered
  without leading zeroes; it is the short number shouted or displayed when the order is
  ready.

The **order name** is derived from the receipt number at the moment the order becomes
paid:

```formula
order_name = sequence_prefix ‖ " - " ‖ last_dash_separated_part_of_receipt_number ‖ optional_suffix
```

where `sequence_prefix` is the prefix of the configuration's order sequence, falling back
to the configuration name when the sequence has no prefix, and `optional_suffix` is
` - ` followed by the sequence suffix when the sequence has one. A refund order is named
instead as the refunded order's name followed by a space and the word `REFUND`.

**Worked example.** Configuration identifier 3, named "Shop", order sequence with no
prefix, backend sequence at value `000127`, device identifier `0`, current year 2026.
Receipt number = `260-3-000127`. Tracking number = 127 modulo 1000 = `127`. Order name,
once paid = `Shop - 000127`.

### 1.17 Ordering, display and archival

- **Ordering**: the platform default (by identifier ascending).
- **Display name**: the configuration name.
- **Archival**: the active flag. Archived configurations disappear from the dashboard.
- **Multi-company**: a record rule limits visibility to configurations whose company is in
  the acting company set.

---

## 2. Point of Sale Session

**Transport name** `pos.session`, **table** `pos_session`.

### 2.1 Purpose

A session is one trading period of one configuration. It is:

- the unit of **cash accountability** — an opening count, the cash movements, a closing
  count and a difference;
- the unit of **accounting posting** — a single balanced journal entry summarises
  everything sold during the period that was not invoiced individually;
- the unit of **inventory posting** when the company chose deferred stock updates — a
  single delivery document per destination location covers the whole period;
- the unit of **data loading** — the selling application downloads its master data when
  the session opens.

The session record carries a message thread and an activity list.

### 2.2 Lifecycle

| Step | State after | What happens |
| --- | --- | --- |
| Created | `opening_control` | The starting balance is pre-filled from the previous session's counted closing balance. |
| Opening control confirmed | `opened` | The opening instant is stamped, the counted opening cash is recorded, the difference against the pre-filled balance is written to the thread, and the session receives its name from the session sequence. |
| Closing control requested | `closing_control` | The closing instant is stamped. Refused while an unfinished order exists. |
| Validated | `closed` | The closing entry is created, posted and reconciled; deferred deliveries are created; uninvoiced paid orders become posted. |

A session in opening control with no order may be deleted outright.

### 2.3 Field table — identification and period

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Session identifier (`name`) | Character | Read-only. Default the single character `/`. Replaced when the session leaves opening control: the configuration name (only when the session sequence prefix is exactly `/`) followed by the next value of the session sequence, followed by the previous value of the field when that value was not `/`. |
| Point of sale (`config_id`) | Many to one → Point of Sale Configuration | Required, indexed. |
| Company (`company_id`) | Many to one → Company | Read-only, related to the configuration's company. |
| Currency (`currency_id`) | Many to one → Currency | Related to the configuration's currency, writable through the relation. |
| Opened by (`user_id`) | Many to one → User | Required, indexed, writable. Default: the acting user. On delete: restrict. |
| Opening date (`start_at`) | Date and time | Read-only. Stamped when the session leaves opening control. |
| Closing date (`stop_at`) | Date and time | Read-only, not copied. Stamped when closing control begins, or when the session is validated without cash control. |
| Status (`state`) | Selection | Required, read-only, indexed, not copied, default `opening_control`. Values: `opening_control` "Opening Control", `opened` "In Progress", `closing_control` "Closing Control", `closed` "Closed & Posted". |
| Recovery session (`rescue`) | Boolean | Read-only, not copied. Marks a session that was created automatically to receive orders belonging to an already closed session. Rescue sessions are excluded from the one-open-session constraint. |
| Access token (`access_token`) | Character | Not copied. From the change-feed mixin; generated on creation. |
| Opening notes (`opening_notes`) | Text | Free text captured at opening; echoed in the thread. |
| Closing notes (`closing_notes`) | Text | Free text captured at closing; echoed in the thread. |
| Update stock at closing (`update_stock_at_closing`) | Boolean | Set on creation from the company setting: true when the company updates stock quantities at session closing, false when it updates them in real time. Frozen for the life of the session, so that a mid-session change of the company setting cannot split one session across both behaviors. |

### 2.4 Field table — cash control

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Has cash control (`cash_control`) | Boolean | Computed, not stored. Equals the configuration's cash-control flag when the session has a cash journal, otherwise false. |
| Cash journal (`cash_journal_id`) | Many to one → Journal | Computed and stored. The journal of the first cash-kind payment method of the configuration. Exactly one cash register is supported per session. |
| Starting balance (`cash_register_balance_start`) | Monetary | Read-only. On creation, when the configuration has cash control and the session is not a rescue session, pre-filled with the counted closing balance of the most recent other session of the same configuration (zero when there is none). Overwritten with the counted opening cash when opening control is confirmed. |
| Ending balance (`cash_register_balance_end_real`) | Monetary | Read-only. The cash actually counted at closing. For a rescue session with cash control it is computed instead as the starting balance plus the cash payments of the non-draft, non-cancelled orders. |
| Theoretical closing balance (`cash_register_balance_end`) | Monetary | Computed, not stored, read-only. See the formula below. |
| Before-closing difference (`cash_register_difference`) | Monetary | Computed, not stored, read-only. Counted ending balance minus theoretical closing balance. |
| Total cash transactions (`cash_real_transaction`) | Monetary | Read-only. Frozen at validation time to the sum of the amounts of the session's cash statement lines, so that the theoretical balance of a closed session no longer moves when a statement line is later touched. |
| Cash lines (`statement_line_ids`) | One to many → Bank Statement Line, inverse `pos_session_id` | Read-only. The manual cash-in and cash-out movements and the closing difference line. Deleted with the session. |

**Theoretical closing balance.**

```formula
captured_cash_payments = sum of the amounts of every Point of Sale Payment of this session
                         whose payment method is the first cash-kind method of the configuration
                         and whose order is in state paid, invoiced or posted
```

```formula
total_cash = ( cash_real_transaction  if the session is closed
               otherwise  sum of the amounts of the session's cash statement lines )
             + captured_cash_payments
```

```formula
cash_register_balance_end = cash_register_balance_start + total_cash
```

```formula
cash_register_difference = cash_register_balance_end_real − cash_register_balance_end
```

When the configuration has no cash-kind payment method, both the theoretical closing
balance and the difference are zero.

### 2.5 Field table — orders, payments and documents

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Orders (`order_ids`) | One to many → Point of Sale Order, inverse `session_id` | Every order attached to the session, including unfinished and cancelled ones. |
| Order count (`order_count`) | Integer | Computed, not stored. Number of orders. |
| Payment methods (`payment_method_ids`) | Many to many → Point of Sale Payment Method | Related to the configuration's payment methods. |
| Total payments amount (`total_payments_amount`) | Decimal | Computed, not stored. Sum of the amounts of the session's captured payments (payments whose order is paid, invoiced or posted). |
| Journal entry (`move_id`) | Many to one → Journal Entry, indexed | The session closing entry. Empty until validation; deleted again when validation produced no line at all. |
| Bank payments (`bank_payment_ids`) | One to many → Accounting Payment, inverse `pos_session_id` | The accounting payments created at closing for bank-kind tenders, both aggregated and per-customer. |
| Transfers (`picking_ids`) | One to many → Transfer, inverse `pos_session_id` | Delivery documents attached to the session, both real-time ones and the deferred ones created at closing. |
| Transfer count (`picking_count`) | Integer | Computed, not stored. Number of transfers whose session is this one. |
| Failed transfers (`failed_pickings`) | Boolean | Computed, not stored. True when at least one such transfer is not in the done state. |
| Uses company currency (`is_in_company_currency`) | Boolean | Computed, not stored. True when the session currency equals the company currency. Controls whether accounting lines carry a foreign-currency amount. |

### 2.6 Uniqueness, ordering, display, archival, multi-company

- **One open session per till.** Creating or writing a session is refused when more than
  one non-closed, non-rescue session exists for the same configuration; the message is
  *"Another session is already opened for this point of sale."* The check is skipped
  during onboarding creation.
- **Ordering**: by identifier descending, so the newest session is first.
- **Display name**: the session identifier.
- **Archival**: none; sessions are never archived. Deleting a session deletes its cash
  statement lines first.
- **Multi-company**: a record rule limits visibility to sessions whose configuration's
  company is in the acting company set.

### 2.7 Derived operations exposed on the session

| Operation | Result |
| --- | --- |
| Closing control data | The dashboard shown on the closing screen: order count and total, opening notes, the default cash details (name, expected total, opening balance, payment total, the cash-in and cash-out list, the method identifier), one entry per non-cash method with its name, total, number of payments, identifier and kind, whether the acting user is an administrator, and the authorised difference when the maximum-difference flag is set. |
| Cash-in and cash-out list | Every cash statement line of the session sorted by creation instant, each with a label (the statement label, or "Cash in *n*" / "Cash out *n*" numbered separately per direction), amount, identifier, creation instant and the partner name recorded as cashier. |
| Session orders | Orders of the session whose scheduled time is empty or not in the future. Used by every closing guard, so that orders scheduled for a later time do not block the closing. |
| Closed orders | Orders of the session whose state is neither unfinished nor cancelled. This is the set that the closing entry is built from. |
| Total discount | The sum, over the lines of the closed orders with a positive discount percentage, of the discount amount of the line. |
| Invoice totals | For every invoiced order: the invoice identifier, the signed invoice total in company currency, the invoice number and the receipt number. |
| Total invoiced | The sum of the paid amounts of the invoiced orders. |
| Related accounting documents | The union of: the invoices of the orders, the invoice-payment entries, the closing entry, the accounting entries of the stock moves of the transfers, the cash statement entries, the bank payment entries, and the "other related" entries found by searching journal items whose reference is one of the split-method closing-difference references or the string `pos_order_` followed by a closed order's identifier. |

---

## 3. Point of Sale Order

**Transport name** `pos.order`, **table** `pos_order`.

### 3.1 Purpose

A Point of Sale Order records one transaction at the counter: a sale, a refund, or a
mixture. It is created in the browser, transmitted to the server, paid, optionally
invoiced, and finally posted as part of the session closing entry or reversed out of it
when invoiced later.

The order record carries a message thread, a portal access token and a portal page.

### 3.2 Lifecycle

| State | Meaning |
| --- | --- |
| `draft` "New" | Being composed, or transmitted but not yet paid. The only state in which lines may be changed or the order deleted. |
| `paid` "Paid" | Fully tendered. Lines are frozen. |
| `done` "Posted" | Included in a posted session closing entry, or individually invoiced. |
| `cancel` "Cancelled" | Abandoned before payment. |

### 3.3 Field table — identification

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Order reference (`name`) | Character | Required, read-only, not copied, default the single character `/`. Filled at the moment the order becomes paid, from the receipt number, or as the refunded order's name followed by ` REFUND`. |
| Receipt number (`pos_reference`) | Character | Read-only, not copied, indexed. Assigned at creation from the configuration's backend sequence (see section 1.16). |
| Tracking number (`tracking_number`) | Character | Read-only, not copied. The short customer-facing number (see section 1.16). |
| Universally unique identifier (`uuid`) | Character | Read-only, not copied. Default: a freshly generated universally unique identifier, produced by the browser for orders created there. Subject to a database uniqueness constraint whose violation message is *"An order with this uuid already exists"*. This is the idempotency key of the transmission protocol. |
| Session-unique sequence number (`sequence_number`) | Integer | Not copied. The next value of the configuration's order sequence with the sequence prefix and suffix stripped. Some legal regimes require a strictly increasing per-session number. |
| Order name (`floating_order_name`) | Character | A free label for an order that is not yet attached to a table or a customer. Carried into the invoice narration. |
| Access token | Character | From the portal mixin; lets a customer open the order page without logging in. |
| Ticket code (`ticket_code`) | Character | A five-character alphanumeric code printed on the receipt so that the customer can request an invoice from the portal. |
| Origin (`source`) | Selection | Default `pos`. Base value: `pos` "Point of Sale". Extension packages add further origins (for example self-ordering kiosk and mobile). |

### 3.4 Field table — parties, period and context

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Session (`session_id`) | Many to one → Point of Sale Session, indexed | Restricted in the interface to sessions in the opened state. An order transmitted against a session that is already closing or closed is re-homed to the open session of the same configuration; if there is none the transmission is refused with *"No open session available. Please open a new session to capture the order."* |
| Point of sale (`config_id`) | Many to one → Point of Sale Configuration | Computed and stored from the session's configuration, writable. |
| Company (`company_id`) | Many to one → Company | Required, read-only, indexed. Defaulted from the session's configuration. |
| Fiscal country code (`country_code`) | Character | Related to the company's fiscal country code. |
| Currency (`currency_id`) | Many to one → Currency | Related to the configuration's currency. |
| Currency rate (`currency_rate`) | Decimal, zero decimal places | Computed and stored with elevated rights, read-only. The conversion rate from company currency to order currency applicable at the order date. |
| Date (`date_order`) | Date and time | Read-only, indexed, default now. The order currently being transmitted has its date replaced by the server clock, so that the authoritative timestamp of the live order is the server's. |
| Scheduled time (`preset_time`) | Date and time | The hour of the day for which the order is scheduled (pickup or delivery slot). |
| Preset (`preset_id`) | Many to one → Point of Sale Preset | The service mode. Defaulted from the configuration when presets are enabled. |
| Employee (`user_id`) | Many to one → User | Default the acting user. The person operating the till. |
| Customer (`partner_id`) | Many to one → Partner, indexed when not null | Selecting a customer copies their pricelist onto the order. A customer identifier that no longer exists is dropped on transmission, together with the to-invoice flag. |
| Contact email (`email`) | Character | Computed and stored, writable. Defaults to the customer's email. |
| Contact mobile (`mobile`) | Character | Computed and stored, writable. Defaults to the customer's phone, formatted for the customer's country. On write, reformatted for the customer's country, falling back to the acting company's country. |
| Pricelist (`pricelist_id`) | Many to one → Pricelist | Defaulted from the configuration; replaced by the customer's pricelist when a customer is selected. |
| Fiscal position (`fiscal_position_id`) | Many to one → Fiscal Position | Writable. Defaulted from the configuration's default fiscal position. Maps taxes and accounts of every line. |
| Sales journal (`sale_journal`) | Many to one → Journal | Related to the configuration's point of sale journal, stored, read-only. On delete: restrict. |
| Available payment methods (`available_payment_method_ids`) | Many to many → Point of Sale Payment Method | Related to the configuration's payment methods, read-only, not stored. |
| Stock references (`stock_reference_ids`) | Many to many → Stock Reference, through `stock_reference_pos_order_rel` | The grouping tokens that tie deferred deliveries back to this order. |

### 3.5 Field table — amounts

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Taxes (`amount_tax`) | Monetary | Required, read-only. The tax part of the order, signed. |
| Total (`amount_total`) | Monetary | Required, read-only. The tax-inclusive total, signed. |
| Paid (`amount_paid`) | Monetary | Required. The sum of the amounts of the payments, change included (change is a negative payment). Recomputed server-side on transmission; the client value is not trusted. |
| Returned (`amount_return`) | Monetary | Required, read-only. The absolute value of the sum of the negative payments — that is, the change given back. |
| Difference (`amount_difference`) | Monetary | Read-only. Paid minus total. |
| Tip amount (`tip_amount`) | Monetary | Read-only. The tip recorded on the order. |
| Already tipped (`is_tipped`) | Boolean | Read-only. Set when a deferred tip has been applied. |

The three amounts are recomputed together whenever the payments or the lines change:

1. Paid = the sum of the payment amounts.
2. Returned = minus the sum of the negative payment amounts.
3. Build the tax base lines from the order lines, add the tax detail, round the tax detail
   per base line, and obtain the tax totals summary for the order currency and company.
   Cash rounding is handed to the summary only when the configuration has cash rounding,
   does *not* restrict rounding to cash, and has a rounding method.
4. Let the refund factor be minus one when the order is flagged as a refund or when the
   total is already negative, and plus one otherwise.
5. Taxes = refund factor × the summary's tax amount in order currency.
6. Total = refund factor × the summary's total amount in order currency.
7. Difference = paid − total.

Failing to resolve a currency at step 3 raises *"You can't: create a pos order from the
backend interface, or unset the pricelist, or create a pos.order in a python test with Form
tool, or edit the form view in studio if no PoS order exist"*.

### 3.6 Field table — cost and margin

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Margin (`margin`) | Monetary | Computed, not stored. The sum of the line margins when every line cost is known, otherwise zero. |
| Margin percentage (`margin_percent`) | Decimal, twelve digits with four decimals | Computed, not stored. Margin divided by the signed untaxed amount, or zero when that amount is zero. |
| Total cost computed (`is_total_cost_computed`) | Boolean | Computed, not stored. True when every line has its cost computed. |

```formula
signed_untaxed_amount = round_to_currency( sum over lines of line_untaxed_amount ) × order_sign
```

```formula
margin_percent = margin ÷ signed_untaxed_amount      (zero when the denominator is zero)
```

`order_sign` is minus one for a refund order and plus one otherwise.

### 3.7 Field table — state, invoicing and documents

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Status (`state`) | Selection | Read-only, not copied, indexed, default `draft`. Values: `draft` "New", `cancel` "Cancelled", `paid` "Paid", `done` "Posted". |
| Is a refund (`is_refund`) | Boolean | Read-only, default false. Set on orders produced by the refund procedure. |
| To invoice (`to_invoice`) | Boolean | Not copied. Requests an invoice for this order. |
| Invoice (`account_move`) | Many to one → Journal Entry, read-only, not copied, indexed when not null | The customer invoice or credit note issued for this order. |
| Is invoiced (`is_invoiced`) | Boolean | Computed, not stored. True when the invoice link is filled. |
| Invoice status (`invoice_status`) | Selection | Computed, not stored. `invoiced` "Fully Invoiced" when an invoice exists, otherwise `to_invoice` "To Invoice". |
| Session journal entry (`session_move_id`) | Many to one → Journal Entry | Related to the session's closing entry, read-only, not copied. |
| Reversal entries (`reversed_move_ids`) | One to many → Journal Entry, inverse `reversed_pos_order_id` | The entries created when this order was invoiced after its session had already closed, to take it back out of the closing entry. |
| Payments (`payment_ids`) | One to many → Point of Sale Payment, inverse `pos_order_id` | The tenders. |
| Lines (`lines`) | One to many → Point of Sale Order Line, inverse `order_id` | Copied when the order is duplicated. |
| Transfers (`picking_ids`) | One to many → Transfer, inverse `pos_order_id` | Delivery documents raised for this order alone. |
| Transfer count (`picking_count`) | Integer | Computed, not stored. |
| Failed transfers (`failed_pickings`) | Boolean | Computed, not stored. True when any transfer is not done. |
| Operation type (`picking_type_id`) | Many to one → Operation Type | Related to the session's configuration operation type, writable through the relation. |
| Shipping date (`shipping_date`) | Date | When set, the goods are not handed over now; the delivery is raised through the procurement rules for that date. |

### 3.8 Field table — refunds and editing

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Number of refund orders (`refund_orders_count`) | Integer | Computed, not stored. The number of distinct orders that contain a line refunding a line of this order. |
| Refunded order (`refunded_order_id`) | Many to one → Point of Sale Order | Computed, not stored. The first order that this order's lines refund. |
| Has refundable lines (`has_refundable_lines`) | Boolean | Computed, not stored. True when any line still has a quantity greater than the quantity already refunded, compared at the product-unit decimal precision. |
| Edited (`is_edited`) | Boolean | Computed, not stored. True when any line is flagged edited, or when a line was deleted. |
| Has deleted line (`has_deleted_line`) | Boolean | Set when a line is deleted while edit tracking is on. Once true it can never be set back to false through an ordinary write. |
| Order edit tracking (`order_edit_tracking`) | Boolean | Related to the configuration's flag, read-only. |
| Number of prints (`nb_print`) | Integer | Read-only, not copied, default zero. Counts receipt printings. Once above zero, the payments of the order can no longer be changed (except to a cancelled payment status). |
| Last preparation change (`last_order_preparation_change`) | Character | A structured document, stored as text, describing the last state of the order that was sent to the preparation printers, together with a metadata block carrying the server date of that state. Used to compute the delta to print. |
| Internal note (`internal_note`) | Text | A note for the staff, not printed on the customer receipt. |
| General customer note (`general_customer_note`) | Text | A note for the whole order; carried onto the invoice as a note line. |

### 3.9 Constraints, ordering, display, archival, multi-company

- **Unique universally unique identifier** — database constraint; message *"An order with
  this uuid already exists"*.
- **Deletion** is refused unless the order is unfinished or cancelled: *"In order to
  delete a sale, it must be new or cancelled."* An unfinished order is first cancelled (so
  that the change feed and the selling application stay in step) and then deleted.
- **Ordering**: by order date descending, then name descending, then identifier
  descending.
- **Display name**: the order reference.
- **Archival**: none.
- **Multi-company**: a record rule limits visibility to orders whose company is in the
  acting company set.
- **Customer deletion**: a partner with counter orders cannot be deleted — *"You cannot
  delete a customer that has point of sales orders. You can archive it instead."*

---

## 4. Point of Sale Order Line

**Transport name** `pos.order.line`, **table** `pos_order_line`.

### 4.1 Purpose

One product line of a counter order: the product, the quantity, the unit price, the
discount, the taxes, the chosen attribute values, the lots, the notes, the combo linkage
and the computed cost.

### 4.2 Field table

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Line label (`name`) | Character | Required, not copied. Assigned at creation: the next value of the configuration's order-line sequence when the order's configuration has one, otherwise the next value of the generic order-line sequence. |
| Order (`order_id`) | Many to one → Point of Sale Order | Required, indexed, on delete cascade. |
| Company (`company_id`) | Many to one → Company | Related to the order's company, stored. |
| Universally unique identifier (`uuid`) | Character | Read-only, not copied, default a freshly generated universally unique identifier. Database uniqueness constraint; message *"An order line with this uuid already exists"*. |
| Product (`product_id`) | Many to one → Product Variant | Required. Restricted to sellable products. |
| Full product name (`full_product_name`) | Character | The rendered product description including chosen attribute values. Used verbatim on receipts and invoices. |
| Quantity (`qty`) | Decimal, product-unit precision | Default 1. Negative on a refund line. |
| Product unit (`product_uom_id`) | Many to one → Unit of Measure | Related to the product's reference unit. |
| Unit price (`price_unit`) | Decimal, unrestricted precision | The price before discount, expressed tax-excluded or tax-included exactly as the product's taxes require. |
| Price extra (`price_extra`) | Decimal | The sum of the attribute price supplements folded into the unit price. |
| Price type (`price_type`) | Selection | Default `original`. Values: `original` "Original" (the price came from the product or the pricelist), `manual` "Manual" (the cashier typed it), `automatic` "Automatic" (a program such as loyalty set it). |
| Discount percentage (`discount`) | Decimal, unrestricted precision | Default zero. A percentage applied to the unit price. |
| Discount notice (`notice`) | Character | A free label explaining the discount. |
| Taxes (`tax_ids`) | Many to many → Tax, through `account_tax_pos_order_line_rel` | Read-only. The taxes of the product, before fiscal position mapping. |
| Taxes to apply (`tax_ids_after_fiscal_position`) | Many to many → Tax | Computed, not stored. The taxes after the order's fiscal position has mapped them. |
| Extra tax data (`extra_tax_data`) | Structured document | Carries per-line custom data for the tax engine (for example an externally determined tax amount). |
| Tax-excluded amount (`price_subtotal`) | Monetary | Required, read-only. |
| Tax-included amount (`price_subtotal_incl`) | Monetary | Required, read-only. |
| Currency (`currency_id`) | Many to one → Currency | Related to the order's currency. |
| Total cost (`total_cost`) | Decimal, displayed with at least the product-price precision | Read-only. The cost of the quantity sold, expressed in the order currency. |
| Total cost computed (`is_total_cost_computed`) | Boolean | True once the cost has been established. |
| Margin (`margin`) | Monetary | Computed, not stored. |
| Margin percentage (`margin_percent`) | Decimal, twelve digits with four decimals | Computed, not stored. |
| Selected attributes (`attribute_value_ids`) | Many to many → Product Template Attribute Value | The chosen values, including the ones that do not create a variant. |
| Custom values (`custom_attribute_value_ids`) | One to many → Product Attribute Custom Value, inverse `pos_order_line_id` | Free-text values typed for custom attributes. Stored and writable. |
| Lots and serial numbers (`pack_lot_ids`) | One to many → Point of Sale Pack Operation Lot, inverse `pos_order_line_id` | The captured lot names. |
| Product note (`note`) | Character | A note attached to the line, shown on the preparation ticket. |
| Customer note (`customer_note`) | Character | A note attached to the line; carried onto the invoice as a note line under the product line. |
| Combo parent (`combo_parent_id`) | Many to one → Point of Sale Order Line, indexed when not null | The combo header line this line belongs to. |
| Combo children (`combo_line_ids`) | One to many → Point of Sale Order Line, inverse `combo_parent_id` | The component lines of a combo header. |
| Combo item (`combo_item_id`) | Many to one → Product Combo Item | Which choice of the combo this line realises. |
| Refunded line (`refunded_orderline_id`) | Many to one → Point of Sale Order Line, indexed when not null | Set on a refund line; points at the line being refunded. |
| Refunding lines (`refund_orderline_ids`) | One to many → Point of Sale Order Line, inverse `refunded_orderline_id` | The lines that refunded this line. |
| Refunded quantity (`refunded_qty`) | Decimal | Computed, not stored. Minus the sum of the quantities of the refunding lines whose order is not cancelled. Because refund quantities are negative, this value is positive. |
| Edited (`is_edited`) | Boolean | Default false. Set when the quantity is reduced while edit tracking is on. |

### 4.3 Line amount computation

Whenever the unit price, taxes, quantity, discount or product change, the two amounts are
recomputed:

```formula
price_after_discount = price_unit × ( 1 − discount ÷ 100 )
```

The taxes to apply (after fiscal position mapping) are then evaluated over
`price_after_discount` with quantity `qty × order_sign`, for the order currency, the
product and the order's customer. The tax-excluded amount is the engine's excluded total
and the tax-included amount is the engine's included total. `order_sign` is minus one when
the order is a refund and plus one otherwise.

A second, lighter recomputation path runs when only the quantity, discount, unit price or
taxes change on the form: if the line has no taxes, both amounts are simply
`price_after_discount × qty`; if it has taxes, the engine is called with quantity `qty`
and no partner.

### 4.4 Refund guard

When a line refunds another line, changing its quantity re-checks the outstanding
quantity:

```formula
already_refunded = sum of the quantities of the other refunding lines of the same refunded line
                   whose order is not cancelled
```

```formula
total_refunded = | already_refunded | + | qty |
```

If `| refunded_line_qty | − total_refunded` is negative the change is refused with *"You
cannot refund more than the outstanding quantity for this product."*

### 4.5 Cost of the line

The cost is established once, and only once (the computed flag guards it):

1. Take the product's cost currency.
2. If stock moves were supplied and the product is storable with a first-in-first-out or
   average cost method, take the unit price of those moves (the moves filtered to this
   product). If that unit price is zero and the order has a shipping date, fall back to
   the unit cost of the refunded line (its total cost divided by its quantity) for a refund
   line, or to the product's standard price otherwise.
3. Otherwise take the product's standard price.
4. Convert from the cost currency to the order currency, for the order's company, at the
   order date (today when the order has no date), **without rounding**, and multiply by the
   line quantity.

```formula
total_cost = qty × convert_unrounded( product_cost , from = cost_currency , to = order_currency , at = order_date )
```

### 4.6 Margin of the line

```formula
line_margin = ( price_subtotal × order_sign ) − total_cost
```

```formula
line_margin_percent = line_margin ÷ ( price_subtotal × order_sign )     (zero when price_subtotal is zero)
```

A line whose product is a combo header always reports a margin and a margin percentage of
zero, because its value is carried by its component lines.

### 4.7 Discount amount of the line

```formula
undiscounted_included = tax_engine_included_total( price_unit , qty , taxes_after_fiscal_position )
```

```formula
line_sign = −1 if price_unit × qty < 0 , otherwise +1
```

```formula
discount_amount = line_sign × ( | undiscounted_included | − | price_subtotal_incl | )
```

### 4.8 Deletion rule, ordering, display

- A line may only be deleted while its order is unfinished or cancelled: *"You can only
  unlink PoS order lines that are related to orders in new or cancelled state."*
- Deleting a line while edit tracking is on sets the order's deleted-line flag and posts
  *"<product name>: Deleted line (quantity: <quantity>)"* in the order thread.
- Reducing a quantity while edit tracking is on sets the line's edited flag and posts
  *"<product name>: Ordered quantity: <old quantity>→<new quantity>"*.
- **Display name**: the product.
- **Multi-company**: a record rule limits visibility to lines whose company is in the
  acting company set.

---

## 5. Point of Sale Pack Operation Lot

**Transport name** `pos.pack.operation.lot`, **table** `pos_pack_operation_lot`.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Order line (`pos_order_line_id`) | Many to one → Point of Sale Order Line, indexed when not null | The line the lot belongs to. |
| Order (`order_id`) | Many to one → Point of Sale Order | Related to the line's order, writable through the relation. |
| Lot name (`lot_name`) | Character | The lot or serial number as typed or scanned. |
| Product (`product_id`) | Many to one → Product Variant | Related to the line's product, writable through the relation. |

**Display name**: the lot name. One record is expected per unit for a serial-tracked
product and one per lot for a lot-tracked product.

---

## 6. Point of Sale Payment

**Transport name** `pos.payment`, **table** `pos_payment`.

### 6.1 Purpose

One tender applied to one order. Change given back to the customer is itself recorded as a
payment with a negative amount and the change flag set, so that the sum of the payment
amounts always equals the amount the customer actually parted with.

### 6.2 Field table

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Label (`name`) | Character | Read-only. The change payment is labelled with the word "return". |
| Order (`pos_order_id`) | Many to one → Point of Sale Order | Required, indexed, on delete cascade. |
| Amount (`amount`) | Monetary in the order currency | Required. Negative for change and for refunds. |
| Payment method (`payment_method_id`) | Many to one → Point of Sale Payment Method | Required. Must be one of the configuration's payment methods. |
| Date (`payment_date`) | Date and time | Required, read-only, default now. |
| Currency (`currency_id`) | Many to one → Currency | Related to the order currency. |
| Conversion rate (`currency_rate`) | Decimal | Related to the order's currency rate. |
| Customer (`partner_id`) | Many to one → Partner | Related to the order's customer. |
| Session (`session_id`) | Many to one → Point of Sale Session | Related to the order's session, stored, indexed. |
| Employee (`user_id`) | Many to one → User | Related to the session's opening user. |
| Company (`company_id`) | Many to one → Company | Related to the order's company, stored. |
| Is change (`is_change`) | Boolean | Default false. Marks the negative cash payment representing change. |
| Accounting entry (`account_move_id`) | Many to one → Journal Entry, indexed when not null | The invoice-payment entry created for this tender when the order is invoiced. |
| Universally unique identifier (`uuid`) | Character | Read-only, not copied, default a freshly generated universally unique identifier. Database uniqueness constraint; message *"A payment with this uuid already exists"*. |
| Card type (`card_type`) | Character | For example credit card or debit card. |
| Card brand (`card_brand`) | Character | |
| Card number, last four digits (`card_no`) | Character | |
| Cardholder name (`cardholder_name`) | Character | |
| Payment reference number (`payment_ref_no`) | Character | The reference returned by the terminal. |
| Approval code (`payment_method_authcode`) | Character | |
| Issuer bank (`payment_method_issuer_bank`) | Character | |
| Payment mode (`payment_method_payment_mode`) | Character | |
| Transaction identifier (`transaction_id`) | Character | |
| Payment status (`payment_status`) | Character | The terminal's status string. |
| Payment receipt information (`ticket`) | Character | The text block the terminal wants printed on the receipt. |

### 6.3 Rules

- **Amount may not be edited on a posted order.** Writing an amount when the order is
  posted or already invoiced raises *"You cannot edit a payment for a posted order."*
- **Method must be offered on the till.** Writing a method that is not among the session
  configuration's payment methods raises *"The payment method selected is not allowed in
  the config of the POS session."*
- **Ordering**: by identifier descending.
- **Display name**: the label followed by the amount formatted in the payment currency,
  or just the formatted amount when there is no label.
- **Multi-company**: a record rule limits visibility to payments whose company is in the
  acting company set.

### 6.4 Receivable lines selected for invoice reconciliation

When an invoice is reconciled against the entries of its counter payments, the lines to
reconcile are chosen by sign so that a receivable account shared with the ordinary
customer receivable cannot be mismatched:

1. Skip payments with no accounting entry.
2. For each remaining payment, walk the lines of its entry and keep a line only when its
   balance is not zero, its account is the target receivable account, and it is not
   already reconciled.
3. For a positive payment keep only the lines with a negative balance; for a negative
   payment keep only the lines with a positive balance.

---

## 7. Point of Sale Payment Method

**Transport name** `pos.payment.method`, **table** `pos_payment_method`.

### 7.1 Purpose

A way of being paid. Three kinds exist and the kind is derived, never chosen directly:

| Kind (`type`) | Derivation | Accounting treatment at session closing |
| --- | --- | --- |
| `cash` "Cash" | The method's journal is of type cash | A bank statement line in the cash journal plus a counterpart receivable line in the closing entry |
| `bank` "Bank" | The method's journal is of type bank | An accounting payment through the method's outstanding account, plus a counterpart receivable line in the closing entry |
| `pay_later` "Customer Account" | The method has no journal, or a journal of some other type | A receivable line in the closing entry and nothing else; the amount stays owed by the customer |

### 7.2 Field table

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Method (`name`) | Character | Required, translatable. Shown on the payment screen. |
| Sequence (`sequence`) | Integer | Not copied. Controls the order of the payment buttons. The only field that may be changed while a session using the method is open. |
| Kind (`type`) | Selection | Computed, not stored. Values `cash` "Cash", `bank` "Bank", `pay_later` "Customer Account". |
| Cash (`is_cash_count`) | Boolean | Computed and stored. True exactly when the kind is cash. |
| Journal (`journal_id`) | Many to one → Journal, indexed when not null, company-checked | On delete: restrict. Only cash and bank journals are allowed, and a cash journal that is already attached to a payment method cannot be chosen again. Leaving it empty makes the method a customer-account method. Selecting a bank journal pre-fills the outstanding account from the chart template's inbound outstanding payment account, falling back to the company transfer account. Selecting a journal of any other type raises *"Only journals of type 'Cash' or 'Bank' could be used with payment methods."* |
| Outstanding account (`outstanding_account_id`) | Many to one → Account | On delete: restrict. The transit account debited when an accounting payment is created for a bank tender. |
| Intermediary account (`receivable_account_id`) | Many to one → Account | On delete: restrict. Restricted to reconcilable accounts of the receivable kind. Overrides the company's default point-of-sale receivable account in the closing entry. |
| Identify customer (`split_transactions`) | Boolean | Default false. Forces a customer on every order paid with this method and splits the closing entry into one line per payment instead of one aggregated line per method. |
| Points of sale (`config_ids`) | Many to many → Point of Sale Configuration | The tills that offer this method. Cleared when the method is duplicated. |
| Open sessions (`open_session_ids`) | Many to many → Point of Sale Session | Computed, not stored. Non-closed sessions of the tills that offer this method. |
| Company (`company_id`) | Many to one → Company | Default the acting company. |
| Default receivable account name (`default_pos_receivable_account_name`) | Character | Related to the display name of the company's default point-of-sale receivable account, read-only. |
| Integration (`payment_method_type`) | Selection | Required, default `none`. Three values. `none` — no integration is required. `terminal` — the tender is taken on a payment terminal. `qr_code` — the customer is shown a scannable quick response code carrying the bank transfer details; this value is only offered when at least one quick response code payment format is available. |
| Use a payment terminal (`use_payment_terminal`) | Selection | The terminal provider. The list of providers is contributed by the terminal packages; it is empty in the base capability. |
| Hide the terminal selector (`hide_use_payment_terminal`) | Boolean | Computed, not stored. True when no provider is available at all, or the kind is cash or customer account, or the integration is not terminal. |
| Quick response code format (`qr_code_method`) | Selection | Not copied. Which quick response code payment format to generate. |
| Hide the quick response code format selector (`hide_qr_code_method`) | Boolean | Computed, not stored. True when the integration is not quick response code, or only one format exists. |
| Default quick response code (`default_qr`) | Character | Computed, not stored. A quick response code generated with no amount, so that the selling application can display something while offline. Empty when generation fails. |
| Image (`image`) | Binary image, at most 50 by 50 | The logo shown on the payment button. |
| Active (`active`) | Boolean | Default true. |

### 7.3 Write restriction while sessions are open

Any write is refused when the method is offered on a till with a non-closed session,
unless every written field is in the whitelist (which contains only the sequence). The
message is *"Please close and validate the following open PoS Sessions before modifying
this payment method. Open sessions: <names>"*.

### 7.4 Integration exclusivity

Setting the integration forces the fields belonging to the other integrations to empty:

- integration `terminal` clears the quick response code format;
- integration `qr_code` clears the terminal provider;
- integration `none` clears both.

On a write that does not itself change the integration, the same clearing is applied only
to the fields actually present in the write, and the records are split into three groups
(terminal, quick response code, neither) so that each group gets the right treatment.

### 7.5 Validation rules

- **Quick response code configuration.** When the integration is quick response code the
  journal must be a bank journal with a bank account — otherwise *"At least one bank
  account must be defined on the journal to allow registering QR code payments with Bank
  apps."* A format must be chosen — otherwise *"You must select a QR-code method to
  generate QR-codes for this payment method."* The bank account must be able to produce
  that format for the company currency; the format's own error message is reported.
- **Company agreement.** Every till offering the method must belong to the method's
  company — otherwise *"The points of sale for the payment method <name> must belong to its
  company."*
- **One shop per cash method.** A cash method (or a method on a cash journal) may be
  attached to at most one till — otherwise *"Validation Error: You cannot assign the same
  Cash payment method to multiple POS Shops. Please create a separate Cash payment method
  for each shop."*

### 7.6 Duplication

Duplicating a method clears the till list. If the original is on a cash journal and the
duplicate would keep that same journal, the journal is cleared on the duplicate, because a
cash journal may carry only one payment method.

### 7.7 Ordering, display, archival, multi-company

- **Ordering**: by sequence, then identifier.
- **Display name**: the method name.
- **Archival**: the active flag. Archived methods are still loaded into the selling
  application (the load condition explicitly accepts both active and archived methods) so
  that historical orders remain readable.
- **Multi-company**: a record rule limits visibility to methods whose company is in the
  acting company set.

---

## 8. Point of Sale Category

**Transport name** `pos.category`, **table** `pos_category`.

### 8.1 Purpose

A merchandising tree used only to lay out the product buttons and to route preparation
printing. It is independent of the accounting product category.

### 8.2 Field table

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Category name (`name`) | Character | Required, translatable. |
| Parent category (`parent_id`) | Many to one → Point of Sale Category, indexed | |
| Children categories (`child_ids`) | One to many → Point of Sale Category, inverse `parent_id` | |
| Sequence (`sequence`) | Integer | Display order. |
| Image (`image_512`) | Binary image, at most 512 by 512 | |
| Image, small (`image_128`) | Binary image, at most 128 by 128 | Related to the large image, stored. |
| Has image (`has_image`) | Boolean | Computed, not stored. True when the small image is present. Exists so that the selling application can know whether to request the image without downloading it. |
| Colour (`color`) | Integer | Default: a pseudo-random whole number between zero and ten inclusive, drawn at creation. |
| Availability after (`hour_after`) | Decimal | Default 0.0. The hour of the day, as a decimal number of hours, from which products of this category may be ordered online or through self-ordering. |
| Availability until (`hour_until`) | Decimal | Default 24.0. The hour of the day until which they may be ordered. |

### 8.3 Rules

- **No cycles.** A category may not be its own ancestor: *"Error! You cannot create
  recursive categories."*
- **Hour range.** Each hour must lie between 0.0 and 24.0 inclusive — otherwise *"The
  Availability Until must be set between 00:00 and 24:00"* or *"The Availability After must
  be set between 00:00 and 24:00"*. The until hour must not be smaller than the after
  hour — otherwise *"The Availability Until must be greater than Availability After."*
- **Deletion** is refused while any session anywhere is not closed: *"You cannot delete a
  point of sale category while a session is still opened."*
- **Ordering**: by sequence, then name.
- **Display name**: the names of the ancestors and of the category itself joined with
  ` / `.
- **Descendants**: the category together with, recursively, all of its children.

---

## 9. Point of Sale Bill

**Transport name** `pos.bill`, **table** `pos_bill`.

A coin or banknote denomination offered as a one-touch button on the payment screen.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Name (`name`) | Character | The label on the button. |
| Value (`value`) | Decimal, sixteen digits with four decimals | Required. The denomination. |
| Points of sale (`pos_config_ids`) | Many to many → Point of Sale Configuration | The tills that offer this denomination. A denomination attached to no till is offered on every till. |

Creating a denomination by typing its name parses the name as a number; a non-numeric name
is refused with *"The name of the Coins/Bills must be a number."*

**Ordering**: by value ascending.

---

## 10. Point of Sale Note

**Transport name** `pos.note`, **table** `pos_note`.

A predefined note offered as a one-touch button.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Name (`name`) | Character | Required. Database uniqueness constraint on the name; message *"A note with this name already exists"*. |
| Sequence (`sequence`) | Integer | Default 1. |
| Colour (`color`) | Integer | |

**Ordering**: by sequence.

---

## 11. Point of Sale Preset

**Transport name** `pos.preset`, **table** `pos_preset`.

### 11.1 Purpose

A named bundle of service settings: eat in, take away, delivery, returns. Choosing a
preset on an order switches the pricelist and the fiscal position, may require the customer
to be identified, may flip the whole cart into negative quantities, and may bind the order
to a time slot.

### 11.2 Field table

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Label (`name`) | Character | Required, translatable. |
| Pricelist (`pricelist_id`) | Many to one → Pricelist | Applied to orders using this preset. |
| Fiscal position (`fiscal_position_id`) | Many to one → Fiscal Position | Applied to orders using this preset. |
| Identification (`identification`) | Selection | Required, default `none`. Values: `none` "Not required", `address` "Address", `name` "Name". Governs what the cashier or the self-ordering customer must supply. |
| Return mode (`is_return`) | Boolean | Default false. Every quantity added to the cart is negative. |
| Colour (`color`) | Integer | Default 0. |
| Image (`image_512`) | Binary image, at most 512 by 512 | |
| Image, small (`image_128`) | Binary image, at most 128 by 128 | Related, stored. |
| Has image (`has_image`) | Boolean | Computed, not stored, from the large image. |
| Manage orders by time (`use_timing`) | Boolean | Default false. Enables slot booking. |
| Working schedule (`resource_calendar_id`) | Many to one → Working Schedule | The opening hours used to generate slots. |
| Attendances (`attendance_ids`) | One to many → Working Schedule Attendance | Related to the schedule's attendance lines, writable through the relation. |
| Capacity (`slots_per_interval`) | Integer | Default 5. How many orders may be booked in one slot. |
| Interval length in minutes (`interval_time`) | Integer | Default 20. |
| Linked orders count (`count_linked_orders`) | Integer | Computed, not stored. |
| Linked configurations count (`count_linked_config`) | Integer | Computed, not stored. Counts tills where the preset is either the default or among the available presets. |

### 11.3 Rules

- **Attendance sanity.** For every attendance line the start hour, reduced modulo
  twenty-four, must be strictly smaller than the end hour reduced modulo twenty-four —
  otherwise *"The start time must be before the end time."*
- **Deletion** is refused when the preset is attached to any configuration: *"You cannot
  delete a preset that is linked to a POS configuration."*

### 11.4 Slot usage

The available-slots operation returns a single entry, the usage map. The usage map is
built as follows: search the orders of this preset whose session is opened, whose
scheduled time is set, whose state is unfinished or paid, and which were created within
the last day; group their identifiers by scheduled time rendered as
`year-month-day hour:minute:second`. The selling application compares the length of each
group with the capacity to decide whether a slot is still bookable.

---

## 12. Point of Sale Printer

**Transport name** `pos.printer`, **table** `pos_printer`.

A preparation printer: the kitchen or bar device that receives the items to prepare.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Printer name (`name`) | Character | Required, default the word "Printer". |
| Printer type (`printer_type`) | Selection | Default `iot`. Two values. `iot` — the printer is attached to an internet-of-things gateway and is reached through it. `epson_epos` — the printer is addressed directly over the local network at its own address. |
| Proxy address (`proxy_ip`) | Character | The address or hostname of the gateway. |
| Direct printer address (`epson_printer_ip`) | Character | Default the four zero-groups `0.0.0.0`. Required when the printer type is the direct one — otherwise *"Epson Printer IP Address cannot be empty."* A value with no dot is converted to a certificate-bearing hostname exactly as described for the configuration's receipt printer address. |
| Printed product categories (`product_categories_ids`) | Many to many → Point of Sale Category, through `printer_category_rel` | Only lines whose product belongs to one of these categories are sent to this printer. |
| Company (`company_id`) | Many to one → Company | Required, default the acting company. |
| Points of sale (`pos_config_ids`) | Many to many → Point of Sale Configuration, through `pos_config_printer_rel` | |

A configuration-level switch decides whether the local-network-access permission must be
requested from the browser before talking to a directly addressed printer; it is read from
the system parameter for local network access.

---

## 13. Stock Reference

**Transport name** `stock.reference`, **table** `stock_reference`.

The base entity belongs to the inventory domain. This domain adds:

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Counter orders (`pos_order_ids`) | Many to many → Point of Sale Order, through `stock_reference_pos_order_rel` | The counter orders that this reference groups. |

A reference is created, named after the order, when a ship-later order launches its
procurement, unless the order already has one. Moves created by those procurements inherit
the reference, and the transfer they are grouped into inherits the session and the order
from it.

---

## 14. Point of Sale Order Analysis

**Transport name** `report.pos.order`.

A read-only analytical view, one row per order line, used for pivots and graphs. Every
column is read-only.

| Column (storage name) | Type | Meaning |
| --- | --- | --- |
| Order date (`date`) | Date and time | The order date. |
| Order reference (`order_id`) | Many to one → Point of Sale Order | |
| Employee (`user_id`) | Many to one → User | |
| Customer (`partner_id`) | Many to one → Partner | |
| Product (`product_id`) | Many to one → Product Variant | |
| Product category (`product_categ_id`) | Many to one → Product Category | |
| Point of sale category (`pos_categ_id`) | Many to one → Point of Sale Category | |
| Company (`company_id`) | Many to one → Company | |
| Point of sale (`config_id`) | Many to one → Point of Sale Configuration | |
| Session (`session_id`) | Many to one → Point of Sale Session | |
| Status (`state`) | Selection | The order state. |
| Pricelist (`pricelist_id`) | Many to one → Pricelist | |
| Product quantity (`product_qty`) | Decimal | The quantity, signed. |
| Total without tax (`price_subtotal`) | Decimal | |
| Total with tax (`price_total`) | Decimal | |
| Average price (`average_price`) | Decimal | |
| Total cost (`total_cost`) | Decimal | |
| Margin (`margin`) | Decimal | |
| Number of lines (`nbr_lines`) | Integer | |
| Invoiced (`invoiced`) | Boolean | |
| Delay of validation (`delay_validation`) | Integer | Days between the order date and the invoice date. |

**Multi-company**: a record rule limits visibility to rows whose company is in the acting
company set.

---

## 15. Wizard entities

### 15.1 Point of Sale Details Wizard (`pos.details.wizard`)

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Start date (`start_date`) | Date and time | Required. Default: the earliest of the most recent opening instants of the sessions of each configuration that opened within the last two days; the current transaction instant when there is none. |
| End date (`end_date`) | Date and time | Required. Default: now. |
| Points of sale (`pos_config_ids`) | Many to many → Point of Sale Configuration, through `pos_detail_configs` | Default: every configuration the acting user may read. |

Changing the start date to an instant later than the end date drags the end date to the
same instant. Changing the end date to an instant earlier than the start date drags the
start date to the same instant. The action renders the sales details document for the
chosen range and configurations.

### 15.2 Point of Sale Daily Sales Reports Wizard (`pos.daily.sales.reports.wizard`)

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Session (`pos_session_id`) | Many to one → Point of Sale Session | Required. |

The action renders the sales details document for that one session, passing its
configuration as the configuration filter and no date range.

### 15.3 Point of Sale Payment Wizard (`pos.make.payment`)

Registers a payment on an existing order from the administrative interface. The order is
named by the acting context.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Point of sale configuration (`config_id`) | Many to one → Point of Sale Configuration | Required. Default: the configuration of the session of the order named in the context. |
| Amount (`amount`) | Decimal, unrestricted precision | Required. Default: the order total minus the order's paid amount, where the order total is replaced by minus the refunded order's paid amount when this order refunds another order in full (that is, when the two totals cancel out at the currency's precision). |
| Payment method (`payment_method_id`) | Many to one → Point of Sale Payment Method | Required. Default: the payment methods of the order's session sorted with the cash methods first, taking the first one. |
| Payment reference (`payment_name`) | Character | A free label carried onto the payment as its label. |
| Payment date (`payment_date`) | Date and time | Required, default now. |

Confirming the wizard:

1. Refuses when the chosen method identifies the customer and the order has none:
   *"Customer is required for <method name> payment method."*
2. When the amount is not zero at the order currency's precision, adds a payment whose
   amount is the wizard amount passed through the payable-amount rounding, forcing the
   rounding when the method is a cash method or the configuration does not restrict
   rounding to cash.
3. When the order is unfinished and now fully paid, runs the post-transmission processing
   (paid check, delivery, cost), sends the order onward, broadcasts a synchronisation
   notification and closes the wizard.
4. Otherwise reopens itself so that a further tender can be added.

### 15.4 Point of Sale Invoice Wizard (`pos.make.invoice`)

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Consolidated billing (`consolidated_billing`) | Boolean | Default true. When true, orders sharing a configuration, a customer, an employee and a fiscal position receive one invoice; when false, each order is invoiced separately. |
| Order count (`count`) | Integer | Computed, not stored. The number of orders named by the acting context. |

Confirming the wizard:

1. Keeps only the selected orders whose invoice status is to-invoice and whose state is
   neither unfinished nor cancelled. When none remain: *"No valid orders were selected. No
   new invoices could be generated"*.
2. When more than one order remains and any of them refunds an order that was itself
   invoiced, refuses with *"The following refund orders can't be part of a consolidated
   invoice because they refunded invoiced orders. Each refund order should be handled
   separately.\n\n<one line per order, each the order name followed by its receipt number
   in parentheses>"*.
3. Without consolidated billing, or with exactly one order, invoices each order on its own.
4. With consolidated billing: when every order belongs to one configuration, exactly one
   customer appears across the selection and at least one order has no customer, the
   confirmation wizard of section 15.6 is offered instead.
5. Otherwise the orders are grouped by configuration, then by customer, then by employee,
   then by fiscal position. A group whose customer is empty refuses with *"Kindly ensure
   that each order contains a customer."* Each group is invoiced as one document.
6. When any invoice was produced, the invoices are displayed.

### 15.5 Point of Sale Close Session Wizard (`pos.close.session.wizard`)

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Amount to balance (`amount_to_balance`) | Decimal | The unbalanced amount of the refused closing entry. |
| Account (`account_id`) | Many to one → Account | The account on which to post that amount. Default: the company's default point-of-sale receivable account, falling back to the fallback customer receivable account. |
| Account read-only (`account_readonly`) | Boolean | True when the acting user is not allowed to read accounting data, in which case the account may not be changed. |
| Message (`message`) | Text | The explanation shown to the user. |

### 15.6 Point of Sale Confirmation Wizard (`pos.confirmation.wizard`)

Offered when a consolidated invoice is asked for a selection that shares one configuration
and exactly one customer but where some orders carry no customer at all.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Message (`message`) | Text | Read-only. Default: *"It seems that the POS order(s) <the names of the orders with no customer, joined by a comma and a space> do not have a customer.\n\nWould you like to set <the single customer's name> as the customer for the selected POS order(s)?"* |

Confirming writes that customer onto every selected order and reopens the invoice wizard
for the same selection.

---

## 16. Extensions this domain adds to entities owned by other domains

| Entity | Added field (storage name) | Meaning |
| --- | --- | --- |
| Company | Update quantities in stock (`point_of_sale_update_stock_quantities`) | Selection, default `real`. Values: `closing` "At the session closing", `real` "In real time". |
| Company | Self-service invoicing (`point_of_sale_use_ticket_qr_code`) | Boolean, default true. Prints the portal link on the receipt. |
| Company | Generate a code on the receipt (`point_of_sale_ticket_unique_code`) | Boolean. Adds a five-character code to the receipt so that the customer can claim an invoice. |
| Company | Receipt portal link display (`point_of_sale_ticket_portal_url_display_mode`) | Selection, required, default `qr_code_and_url`. Three values. `qr_code` — the link is printed only as a scannable quick response code. `url` — only as a printed uniform resource locator. `qr_code_and_url` — as both. |
| Journal | Counter payment methods (`pos_payment_method_ids`) | One to many → Point of Sale Payment Method. |
| Journal Entry | Counter orders (`pos_order_ids`) | One to many → Point of Sale Order, inverse `account_move`. |
| Journal Entry | Counter payments (`pos_payment_ids`) | One to many → Point of Sale Payment, inverse `account_move_id`. |
| Journal Entry | Refunded invoices (`pos_refunded_invoice_ids`) | Many to many → Journal Entry, through `refunded_invoices`. The invoices that a counter credit note reverses. |
| Journal Entry | Reversed counter order (`reversed_pos_order_id`) | Many to one → Point of Sale Order, indexed when not null. Set on the reversal entry created when an order is invoiced after its session closed. |
| Journal Entry | Sessions (`pos_session_ids`) | One to many → Point of Sale Session, inverse `move_id`. |
| Journal Entry | Counter order count (`pos_order_count`) | Integer, computed. |
| Accounting Payment | Counter payment method (`pos_payment_method_id`) | Many to one → Point of Sale Payment Method. |
| Accounting Payment | Forced outstanding account (`force_outstanding_account_id`) | Many to one → Account, company-checked, indexed when not null. When set it overrides the computed outstanding account. |
| Accounting Payment | Session (`pos_session_id`) | Many to one → Point of Sale Session, indexed when not null. |
| Bank Statement Line | Session (`pos_session_id`) | Many to one → Point of Sale Session, not copied, indexed when not null. |
| Transfer | Session (`pos_session_id`) | Many to one → Point of Sale Session, indexed. |
| Transfer | Counter order (`pos_order_id`) | Many to one → Point of Sale Order, indexed. |
| Operation Type | Has documents to print (`has_stock_reports_to_print`) | Boolean, computed. True when any automatic printing flag of the operation type is set. |
| Product Template | Available at the counter (`available_in_pos`) | Boolean, default false. |
| Product Template | To be weighed (`to_weight`) | Boolean. Sends the product to the scale instead of asking a quantity. |
| Product Template | Counter categories (`pos_categ_ids`) | Many to many → Point of Sale Category. |
| Product Template | Public description (`public_description`) | Rich text, translatable. Emptied when it contains no visible content. |
| Product Template | Suggested products (`pos_optional_product_ids`) | Many to many → Product Template, through `pos_product_optional_rel`. Offered when the product is added to the cart. |
| Product Template | Colour index (`color`) | Integer, computed and stored, writable. Defaults to the colour of the first counter category. |
| Product Template | Counter display order (`pos_sequence`) | Integer, not copied. Default: one more than the current maximum across all products, or one when there is none. |
| Unit of Measure | Group products at the counter (`is_pos_groupable`) | Boolean. Lines of a groupable unit are merged in the cart. |
| Partner | Counter order count (`pos_order_count`) | Integer, computed, visible to the point of sale user group. Counts the orders of the partner and of every descendant partner. |
| Partner | Counter orders (`pos_order_ids`) | One to many → Point of Sale Order, read-only. |
| Partner | Counter address (`pos_contact_address`) | Character, computed. The formatted address without the company line. |
| Partner | Invoice emails (`invoice_emails`) | Character, computed, read-only. The partner's email followed by the emails of the child contacts of the invoice kind, comma separated. |
| Partner | Automatic fiscal position (`fiscal_position_id`) | Many to one → Fiscal Position, computed. The fiscal position the rules select for this partner in the acting company. |
| Tax | Counter order lines (`pos_order_line_ids`) | Many to many → Point of Sale Order Line, through `account_tax_pos_order_line_rel`, read-only, not copied. Makes a tax count as used as soon as a counter line carries it. |
| Barcode Rule | Rule kinds | Adds `weight` "Weighted Product", `price` "Priced Product", `discount` "Discounted Product", `client` "Client", `cashier` "Cashier"; each falls back to the default kind when its defining capability is removed. |

---

## 17. Restaurant extension entities

### 17.1 Restaurant Floor

**Transport name** `restaurant.floor`, **table** `restaurant_floor`.

A named seating area belonging to one or more restaurant-enabled tills.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Floor name (`name`) | Character | Required. |
| Points of sale (`pos_config_ids`) | Many to many → Point of Sale Configuration | Not copied. Restricted to configurations with the restaurant capability. |
| Tables (`table_ids`) | One to many → Restaurant Table, inverse `floor_id` | |
| Background colour (`background_color`) | Character | A colour expressed in a form the browser understands. |
| Background image (`background_image`) | Binary | |
| Floor background image (`floor_background_image`) | Binary image | |
| Sequence (`sequence`) | Integer | Default 1. |
| Active (`active`) | Boolean | Default true. |

Rules:

- **Deletion** is refused while a session of any restaurant-enabled till using the floor
  is not closed. The message begins *"You cannot remove a floor that is used in a PoS
  session, close the session(s) first:"* and then lists, one per line, *"Floor: <floor
  name> - PoS Config: <configuration name>"*.
- **Write** of the till list or of the active flag is refused while any till using the
  floor has an active session: *"Please close and validate the following open PoS Session
  before modifying this floor. Open session: <names>"*.
- **Deactivating a floor** is refused while an unfinished order sits on one of its tables:
  *"You cannot delete a floor when orders are still in draft for this floor."* Otherwise
  every table of the floor is deactivated and then the floor itself.
- **Ordering**: by sequence, then name.

### 17.2 Restaurant Table

**Transport name** `restaurant.table`, **table** `restaurant_table`.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Floor (`floor_id`) | Many to one → Restaurant Floor, indexed when not null | |
| Table number (`table_number`) | Integer | Required, default 0. The number painted on the floor plan. |
| Shape (`shape`) | Selection | Required, default `square`. Values `square` "Square", `round` "Round". |
| Horizontal position (`position_h`) | Decimal | Default 10. Distance in screen points from the left edge to the table's centre. |
| Vertical position (`position_v`) | Decimal | Default 10. Distance in screen points from the top edge to the table's centre. |
| Width (`width`) | Decimal | Default 50, in screen points. |
| Height (`height`) | Decimal | Default 50, in screen points. |
| Seats (`seats`) | Integer | Default 1. The default number of guests served at the table. |
| Colour (`color`) | Character | A background colour expressed in a form the browser understands. |
| Parent table (`parent_id`) | Many to one → Restaurant Table | Set when tables are pushed together into a group. Assigning a parent that would create a cycle is silently reverted to the previous parent. |
| Active (`active`) | Boolean | Default true. |

Rules:

- **Display name**: the floor name, a comma, a space, and the table number.
- **Deletion** is refused while any session of a restaurant-enabled till serving the
  table's floor is not closed: *"You cannot remove a table that is used in a PoS session,
  close the session(s) first."*
- A separate check refuses removing a table that still carries unfinished orders: *"You
  cannot delete a table when orders are still in draft for this table."*

### 17.3 Restaurant Order Course

**Transport name** `restaurant.order.course`, **table** `restaurant_order_course`.

A course groups the lines of an order that must reach the kitchen together.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Order (`order_id`) | Many to one → Point of Sale Order | Required, indexed, on delete cascade. |
| Course index (`index`) | Integer | Default 0. The order in which courses are served. |
| Fired (`fired`) | Boolean | Default false. Set when the course has been sent to preparation. |
| Fired date (`fired_date`) | Date and time | Stamped automatically the first time the fired flag is set, both on creation and on write. |
| Universally unique identifier (`uuid`) | Character | Read-only, not copied, default a freshly generated universally unique identifier. |
| Order lines (`line_ids`) | One to many → Point of Sale Order Line, inverse `course_id` | Read-only. |

### 17.4 Additions to existing entities

| Entity | Added field (storage name) | Meaning |
| --- | --- | --- |
| Point of Sale Configuration | Bill splitting (`iface_splitbill`) | Boolean. Enables splitting one order's payment. |
| Point of Sale Configuration | Bill printing (`iface_printbill`) | Boolean. Allows printing the bill before payment. Defaulted to true when the configuration is created with the restaurant capability. |
| Point of Sale Configuration | Restaurant floors (`floor_ids`) | Many to many → Restaurant Floor, not copied. A forbidden-change field while a session is open. Cleared when the restaurant capability is switched off. |
| Point of Sale Configuration | Set tip after payment (`set_tip_after_payment`) | Boolean. Lets the terminal authorisation be adjusted after the customer has left. Forced to false whenever the restaurant capability or the tip product flag is switched off. |
| Point of Sale Configuration | Default screen (`default_screen`) | Selection, default `tables`. Values `tables` "Tables", `register` "Register". |
| Point of Sale Order | Table (`table_id`) | Many to one → Restaurant Table, read-only, indexed when not null. |
| Point of Sale Order | Guests (`customer_count`) | Integer, read-only. |
| Point of Sale Order | Courses (`course_ids`) | One to many → Restaurant Order Course. |
| Point of Sale Order Line | Course (`course_id`) | Many to one → Restaurant Order Course, on delete set to empty, indexed when not null. |

When the restaurant capability is on, an incoming order is matched to an existing record
not only by its universally unique identifier but also by table: an unfinished order for
the same table on the same configuration is treated as the same order. Creating a
configuration with the restaurant capability and no floor creates a floor named after the
company with one square table numbered 1, one seat, positioned at horizontal 100 and
vertical 100, 130 by 130 in size.

---

## 18. Self-ordering extension entities

### 18.1 Self Order Custom Link

**Transport name** `pos_self_order.custom_link`, **table** `pos_self_order_custom_link`.

An extra navigation button on the self-ordering landing page.

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Label (`name`) | Character | Required, translatable. |
| Address (`url`) | Character | Required. The uniform resource locator the button opens. |
| Points of sale (`pos_config_ids`) | Many to many → Point of Sale Configuration | Restricted to configurations whose self-ordering mode is not disabled. Empty means every such configuration. |
| Style (`style`) | Selection | Required, default `primary`. Values `primary` "Primary", `secondary` "Secondary", `success` "Success", `warning` "Warning", `danger` "Danger", `info` "Info", `light` "Light", `dark` "Dark". |
| Preview (`link_html`) | Rich text | Computed and stored, read-only. A rendered button carrying the chosen style and the escaped label. |
| Sequence (`sequence`) | Integer | Default 1. |

Every configuration automatically receives, on creation and after every write, a link
labelled "Order Now" pointing at its own products page, unless such a link already exists.

### 18.2 Additions to the configuration

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Self ordering mode (`self_ordering_mode`) | Selection | Required, default `nothing`. Four values. `nothing` — self-ordering is disabled. `consultation` — the customer may read the menu on their own device but not order. `mobile` — the customer may read the menu and order from their own device. `kiosk` — ordering happens on a shared device at the counter. |
| Self ordering service mode (`self_ordering_service_mode`) | Selection | Required, default `counter`. Values `counter` "Pickup zone", `table` "Table". |
| Pay after (`self_ordering_pay_after`) | Selection | Required, default `meal`. Values `meal` "Meal", `each` "Each Order". Forced to `each` for kiosk mode, and for mobile mode whenever the service mode is the pickup zone or the restaurant capability is off. Choosing `meal` in mobile mode forces the service mode to table. |
| Status (`status`) | Selection | Computed, not stored. Values `inactive` "Inactive", `active` "Active". |
| Self ordering address (`self_ordering_url`) | Character | Computed. The shortened address of the self-ordering entry point. |
| Default language (`self_ordering_default_language_id`) | Many to one → Language | Default: the language of the acting context. |
| Available languages (`self_ordering_available_language_ids`) | Many to many → Language | Default: every installed language. |
| Home images (`self_ordering_image_home_ids`) | Many to many → Attachment | Default: three shipped landing images. Forced public. |
| Background images (`self_ordering_image_background_ids`) | Many to many → Attachment, through `pos_self_order_background_rels` | Default on creation: one shipped kiosk background. Forced public. |
| Brand image (`self_ordering_image_brand`) | Binary image, at most 1200 by 250 | |
| Brand image name (`self_ordering_image_brand_name`) | Character | |
| Default user (`self_ordering_default_user_id`) | Many to one → User | The access rights used when a self-ordering visitor arrives and no session is open. Default: the first user of the acting company who is a point of sale administrator. |
| Has paper (`has_paper`) | Boolean | Default true. Cleared by the kiosk when its printer reports being out of paper. |

Rules:

- **The default user must be a counter user.** When self-ordering is enabled and the
  default user is missing or is neither a point of sale user nor an administrator:
  *"The Self-Order default user must be a POS user"*.
- **No cash in kiosk mode.** When the self-ordering mode is kiosk and any payment method
  of the till is a cash method: *"You cannot add cash payment methods in kiosk mode."*
- Rotating the access token also regenerates the table identifiers, invalidating every
  previously printed table code.

### 18.3 Additions to the order

| Field (storage name) | Type | Meaning and rules |
| --- | --- | --- |
| Table stand number (`table_stand_number`) | Character | The number of the physical stand the customer takes away from the kiosk. |
| Self-ordering table (`self_ordering_table_id`) | Many to one → Restaurant Table, read-only | The table identified from the scanned code. Realigned automatically when the order is transferred to another table. |
| Origin (`source`) | Selection | Extended with `mobile` "Self-Order Mobile" and `kiosk` "Self-Order Kiosk". |

A self-ordered line carries an additional combo reference (`combo_id`, many to one →
Product Combo) and may be transmitted with a parent reference expressed as a universally
unique identifier rather than an identifier; the reference is resolved on create and on
write and then removed from the values.

---

## 19. Employee extension

The employee capability adds to the order and to the payment the employee who performed
the action, and to the cash statement line the employee who moved the cash. It adds to the
configuration the list of employees allowed to log in, and to the session the current
employee. The selling application then asks for a personal identification number or a
badge scan before letting anyone sell, and the cash-in and cash-out permission checks are
evaluated against the employee rather than the platform user.

---

## 20. Sales-order extension

Settling a sales order at the counter adds a link from the counter order line back to the
sales order line it delivers, and from the counter order back to the sales order. The
configuration gains the ability to list the sales orders that may be settled, and the
sales team gains counter figures. The counter delivery documents created for a settled
sales order are attached to the sales order as well, so that its delivery status advances.

---

## 21. Loyalty extension

The loyalty capability adds to the order line the reward it realises and the coupon it
consumes, and to the order the set of coupons applied. Loyalty cards gain a link to the
counter orders that earned or spent their points. The reward lines are ordinary order
lines carrying a negative amount and the reward's product; they are aggregated into the
session closing entry exactly like any other line, using the reward product's income
account after fiscal position mapping.

---

## 22. Online payment extension

The online payment capability adds a payment method kind that is settled through a payment
provider rather than at the counter. It adds to the order the online payment transactions
attached to it, and to the payment method the providers it accepts. An online payment
produces an accounting payment of its own, so the closing entry treats it exactly like a
bank tender whose outstanding account is the provider's.
