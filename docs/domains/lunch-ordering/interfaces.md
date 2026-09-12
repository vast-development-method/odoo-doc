# Meal Ordering — Interfaces

Every menu, screen, named operation, service endpoint, message and integration point of the domain.

---

## 1. The menu tree

The top-level menu is named "Lunch", carries a packaged icon, sits at sequence 235 among the
workforce applications and is visible to holders of the ordering privilege.

| Level | Entry | Sequence | Visible to | Opens |
|---|---|---|---|---|
| 1 | "Lunch" | 235 | the ordering group | the first child |
| 2 | "My Lunch" | 50 | the ordering group | the first child |
| 3 | "New Order" | 1 | the ordering group | the ordering screen, at the short path `lunch` |
| 3 | "My Order History" | 2 | the ordering group | the employee's own order list |
| 3 | "My Account History" | 3 | the ordering group | the employee's own account statement |
| 2 | "Manager" | 51 | the administration group | the first child |
| 3 | "Today's Orders" | default | the administration group | the day's orders grouped by vendor |
| 3 | "Control Vendors" | default | the administration group | every order grouped by vendor |
| 3 | "Control Accounts" | default | the administration group | the account statement grouped by employee |
| 3 | "Cash Moves" | default | the administration group | the manual movement list |
| 2 | "Configuration" | 53 | the administration group | the first child |
| 3 | "Settings" | 1 | the administration group | the settings screen scoped to this capability |
| 3 | "Vendors" | 2 | the administration group | the vendor screens |
| 3 | "Locations" | 3 | the administration group | the location screens |
| 3 | "Products" | 4 | the administration group | the meal screens |
| 3 | "Product Categories" | 5 | the administration group | the category screens |
| 3 | "Alerts" | 6 | the administration group | the notice screens |

---

## 2. Screens of the Lunch Order

### 2.1 The ordering screen

The ordering screen is not a screen of the order at all: it is the meal catalogue with a side panel
that behaves as a cart. It is opened by "New Order" and is reachable directly at the short path
`lunch`. It is specified in [section 6](#6-the-ordering-screen).

### 2.2 The order dialogue

Opened by clicking a catalogue card or a catalogue row, titled "Configure Your Order", shown as a
modal dialogue, pre-filled with the clicked meal and with the current employee, date and location as
defaults.

| Region | Content |
|---|---|
| Header | The meal picture at full width on the left; the meal name as a first-level heading and the running total price as a second-level heading on the right. |
| First extras block | The vendor's first extras label, then a set of tick boxes over the vendor's group-one extras. Hidden when the vendor has no group-one extra. |
| Second extras block | The same for group two. |
| Third extras block | The same for group three. |
| Description | The meal's description, read only. |
| Note | A free-text field with the placeholder "Information, allergens, ...". |
| Cut-off warning | A warning block reading "The orders for this vendor have already been sent.", shown when the line's cut-off flag is true. |
| Balance warning | A warning block reading "Your wallet does not contain enough money to order that. To add some money to your wallet, please contact your lunch manager.", shown when the balance-allows-one-more flag is false. |
| Footer | The control "Add To Cart", with the keyboard shortcut letter w, hidden when the cut-off has passed or the balance does not allow it; and the control "Discard", with the keyboard shortcut letter x. |

The following fields are present but never shown: the company, the date, the currency, the quantity,
the meal, the state, the category, the three extras-offered flags, the vendor, the cut-off flag and
the availability-today flag. They are loaded because the visibility rules and the computations read
them.

### 2.3 The order list

Used by "My Order History", "Today's Orders" and "Control Vendors". Creation and inline editing are
switched off; groups are expanded by default; cancelled rows are shown muted.

| Column | Notes |
|---|---|
| Order date | Read only unless the state is the to-order state. |
| Vendor | |
| Meal | |
| Extras summary | Overflowing text is clipped. |
| Notes | Overflowing text is clipped. |
| Employee | Shown with the employee's picture. Read only unless the state is the to-order state. |
| Location | |
| Price | Summed per group under the heading "Total", formatted as an amount. |
| State | Shown as a badge: the to-order state in the warning colour, the received state in the success colour, the sent state in the information colour and the ordered state in the danger colour. |
| Company | Only when several companies are in use. |

Controls on each row:

| Control | Label | Shown when | Visible to |
|---|---|---|---|
| Repeat | "Re-order" | the repeat flag and the balance flag are both true | the ordering group |
| Receive | "Confirm" | the state is the sent state | the administration group |
| Cancel | "Cancel" | the state is neither cancelled nor received | the administration group |
| Reset | "Reset" | the state is the cancelled state | the administration group |
| Notify | "Send Notification" | the state is the received state and the delivery notice has not been sent | the administration group |

A header control labelled "Receive" applies the receipt operation to the selected rows.

When rows are grouped by vendor, the group header carries two further controls: "Send Orders", shown
when the vendor has at least one ordered line today, and "Confirm Orders", shown when the vendor has
at least one sent line today.

### 2.4 The order cards

Creation and editing are switched off. Each card shows the meal name in bold, the state as a label
whose colour follows the same mapping as the list, the note, the price with a coin symbol, the date
with a clock symbol, a row of controls and the employee with their picture.

| Control | Symbol | Shown when | Visible to |
|---|---|---|---|
| Order | telephone | the state is neither sent, ordered nor received | the administration group |
| Send | paper plane | the state is the ordered state | the administration group |
| Receive | tick | the state is the sent state | the administration group |
| Cancel | cross | the state is neither cancelled nor received | the administration group |
| Notify | envelope | the state is the received state and the delivery notice has not been sent | the administration group |

### 2.5 The order analyses

| Screen | Axes |
|---|---|
| Cross-table | The order date across the columns, the vendor down the rows. Sample data is shown when the set is empty. |
| Chart | The meal as the measure axis. Sample data is shown when the set is empty. |

### 2.6 The order search panel

| Item | Kind | Effect |
|---|---|---|
| Product | search field | Matches the meal name or the note. |
| User | search field | Matches the employee. |
| "My Orders" | filter | The employee is the reader. |
| "Not Received" | filter | The state is not the received state. |
| "Received" | filter | The state is the received state. |
| "Cancelled" | filter | The state is the cancelled state. |
| "Today" | filter | The order date is today. |
| "Archived" | filter | The archive flag is false. |
| "User" | grouping | By employee. |
| "Vendor" | grouping | By vendor. |
| "Order Date" | grouping | By order date, by day. |

### 2.7 The three order screens

| Screen | Opened by | Default view | Pre-applied filters and flags | Empty-state text |
|---|---|---|---|---|
| "My Orders" | "My Order History" | list, then cards, then the cross-table | the reader's own orders, grouped by order date, with the repeat controls enabled | "No previous order found" and "There is no previous order recorded. Click on \"My Lunch\" and then create a new lunch order." |
| "Today's Orders" | "Today's Orders" | list, then cards | grouped by vendor, filtered to today | "Nothing to order today" and "Here you can see today's orders grouped by vendors." |
| "Control Vendors" | "Control Vendors" | list, then cards, then the cross-table | grouped by vendor | "No lunch order yet", "Summary of all lunch orders, grouped by vendor and by date." and a three-line legend explaining that the telephone symbol announces that the order has been placed, the tick that it has been received and the red cross that it is not available |

---

## 3. Screens of the Lunch Vendor

### 3.1 The vendor form

| Region | Content |
|---|---|
| Ribbon | "Archived" in the danger colour when the vendor is archived. |
| Title | The label "Vendor" and the name as a first-level heading, with the placeholder "e.g. The Pizzeria Inn". |
| Left block | The Contact reference, which pre-fills a newly created Contact as an organisation with the typed name, city, street, second street line, country subdivision, postal code, country and telephone number; then the address block: street, second street line, city, country subdivision with the country and postal code as context, postal code, country. |
| Right block | The electronic mail address, required when the channel is electronic mail; the telephone number, required when the channel is telephone; the company, only when several companies are in use, with the placeholder "Visible to all"; the responsible, required when the channel is electronic mail, shown only in the developer-visible group, restricted to non-shared users. |
| "Availability" block | The time zone, shown only in the developer-visible group; the seven weekday tick boxes rendered by the week-day control; the last service date, shown only in the developer-visible group. |
| "Orders" block | The delivery indicator; the served locations as tags; the channel as a radio choice; the cut-off hour with its half-day marker, both shown only when the channel is electronic mail, the hour rendered as a clock time. |
| Extras blocks | Three repetitions of: the group's label, the group's discipline at half width, and an inline editable list of the group's extras with the columns name and price. |
| Footer | The message thread, follower list and activity list contributed by the messaging capability. |

### 3.2 The vendor list

Three columns: the name, the telephone number forced to left-to-right reading order, and the
electronic mail address.

### 3.3 The vendor cards

Each card shows the display name in bold — which is the name followed by the telephone number when
one is set — then the city alone, the country alone, or the city and the country separated by a
comma, and then the electronic mail address when one is set, clipped if it overflows.

### 3.4 The vendor search panel

A search field on the name and a filter "Archived" on the archive flag.

### 3.5 The vendor screen

"Vendors" opens cards first, then the list, then the form.

---

## 4. Screens of the Lunch Product and the Lunch Product Category

### 4.1 The meal search panel

| Item | Kind | Effect |
|---|---|---|
| Product | search field | The meal name. |
| Category | search field | The category. |
| Vendor | search field | The vendor. |
| Description | search field | The description. |
| "Available Today" | filter | The vendor is available today. |
| "Monday" through "Sunday" | seven filters | The vendor's flag for that weekday is set. |
| "Archived" | filter | The archive flag is false. |
| "Vendor" | grouping | By vendor. |
| "Category" | grouping | By category. |
| Side panel | two multiple-choice facets | "Categories" with a cutlery symbol and counts, "Vendors" with a lorry symbol and counts. |

### 4.2 The meal list

Columns: the name, the category, the vendor, the company when several are in use, the description
and the price as an amount. A second, derived list is used inside the ordering screen: it disables
creation and attaches the ordering behaviour so that clicking any cell opens the order dialogue.

### 4.3 The meal form

An archived ribbon, the picture as an avatar, the name as a first-level heading, then the category,
the vendor and the price on the left, and the novelty date and the company on the right with the
placeholder "Visible to all", then the description.

### 4.4 The meal cards

Two card layouts exist.

- **The ordering layout**, given the highest priority so that the ordering screen picks it up. It
  disables creation, editing and group creation, and attaches the ordering behaviour so that
  clicking anywhere on the card opens the order dialogue. Each card shows the picture with the
  packaged default meal picture as the placeholder, the favourite marker, the name, a pill reading
  "New" in the success colour when the novelty date has not passed, the price, the vendor and the
  description.
- **The plain layout**, used from the configuration menu. It permits creation but not inline
  editing, and shows the picture, the name, the price, the vendor and the description.

### 4.5 The meal screens

| Screen | Opened by | Views | Empty-state text |
|---|---|---|---|
| "Products" | the configuration menu | list, cards, form | "Create a new product for lunch" and "A product is defined by its name, category, price and vendor." |
| "Products", pre-grouped by vendor | the counter control on a category | cards, list, form | the same |
| "Order Your Lunch" | "New Order", also at the short path `lunch` | the ordering cards, then the ordering list | "There is no product available today" and "To see some products, check if your vendors are available today and that you have configured some products" |

### 4.6 The category screens

| Screen | Content |
|---|---|
| List | The category name under the heading "Product Category", and the company when several are in use. |
| Form | A counter control that opens the meals of the category, hidden when the count is zero; the picture as an avatar; the name as a first-level heading; the company with the placeholder "Visible to all". |
| Cards | The picture, a counter badge that opens the meals of the category, the name in bold and the company. |
| Search panel | A search field on the name and a filter "Archived". |
| Screen | "Product Categories" opens the list, then the form, then the cards, with the empty-state text "Create a new product category" and "Here you can access all categories for the lunch products." |

---

## 5. Screens of the Lunch Location, the Lunch Cash Move, the statement and the Lunch Alert

### 5.1 Locations

| Screen | Content |
|---|---|
| Form | The name, the address and the company with the placeholder "Visible to all". |
| List | The same three columns, editable inline at the bottom. |
| Cards | The name in bold, the company and the address. |
| Search panel | Search fields on the name and on the address. |
| Screen | "Lunch Locations" opens the list, then the form, then the cards, with the empty-state text "To see some locations, create one using the create button". |

### 5.2 Manual account movements

| Screen | Content |
|---|---|
| List | The date, the employee, the description and the amount, summed under the heading "Total" and formatted as an amount. |
| Form | The employee, marked required on the screen; the date; the amount; and the description under its own label. |
| Cards | The description in bold, the amount with a coin symbol in a pill, the date with a clock symbol and the employee with their picture. |
| Search panel | Search fields on the description and the employee; the filters "My Account grouped", which restricts to the reader and groups by employee, and "By User", which groups by employee. |
| Screen | "Cash Moves" opens the list, then the cards, then the form, with the empty-state text "Register a payment" and "Payments are used to register liquidity movements. You can process those payments by your own means or by using installed facilities." |

### 5.3 The account statement

Two screens read the same entity with different views and different filters.

| Screen | Opened by | View | Filters | Empty-state text |
|---|---|---|---|---|
| "My Account" | "My Account History" | a list with creation disabled, showing the date, the description and the amount summed under "Total" | restricted to the reading employee | "No cash move yet" and "Here you can see your cash moves.", then "A cash move can either be an expense or a payment. An expense is automatically created when an order is received while a payment is a reimbursement to the company encoded by the manager." |
| "Control Accounts" | "Control Accounts" | a list showing the date, the employee with their picture, the description and the amount summed under "Total"; also cards and a form | grouped by employee by default | "Create a new payment" and "A cashmove can either be an expense or a payment.", then "An expense is automatically created at the order receipt." and "A payment represents the employee reimbursement to the company." |

The statement's own search panel offers search fields on the description and the employee, a filter
"Payment" restricting to rows whose amount is above zero, a filter "My Account grouped" and a filter
"By User". A second search panel, used by the control screen, offers the two search fields and a
grouping "By Employee".

The statement also has a form and a card layout, both read only in practice because the entity
cannot be written.

### 5.4 Notices

| Screen | Content |
|---|---|
| List | The name, the display mode, the displayed-today flag and the archive flag as a toggle. The message column is loaded but hidden. |
| Form | The name as a first-level heading with the placeholder "e.g. Order before 11am"; then the display mode as a radio choice, the audience as a radio choice shown only in the pushed mode, the locations as tags marked required, the show-until date and the archive flag as a toggle; then the notification hour label shown only in the pushed mode, the seven weekday tick boxes, the notification hour and its half-day marker shown only in the pushed mode with the hour required there, and the time zone shown only in the developer-visible group; then the message. |
| Cards | The name in bold; the display mode and, in the pushed mode, the audience, the notification hour and its marker; the locations as tags. |
| Search panel | A search field on the message; the filters "Currently inactive" on the displayed-today flag being false, "Active" and "Archived". |
| Screen | "Lunch Alerts" opens the list, then the form, then the cards, over both active and archived records, with the empty-state text "Create new lunch alerts". |

---

## 6. The ordering screen

The ordering screen is the meal catalogue with a permanent side panel. Two view kinds carry it, the
cards and the list, and both behave identically.

### 6.1 What the screen adds to an ordinary catalogue

1. **A location filter that cannot be switched off.** On opening, the screen asks the service
   endpoint at the route `/lunch/user_location_get` for a location and remembers it. The catalogue
   query is then narrowed to the meals available at that location. While no location is known the
   screen lists nothing at all.
2. **A date filter tied to a date picker.** Choosing a date in the panel switches on the weekday
   filter matching that date and switches off any weekday filter already active, so the catalogue
   always shows the meals of exactly one weekday.
3. **An employee the screen may impersonate.** An administrator may pick another employee in the
   panel; every subsequent service call carries that employee, and the order dialogue receives them
   as its default employee.
4. **A side panel that is opened as a sliding drawer on a small screen**, behind a control reading
   "Your Cart" followed by the cart total.

### 6.2 The panel

| Region | Content |
|---|---|
| Notices | One warning block holding every banner notice that is displayable today, whose mode is the banner mode and whose locations include the current location. |
| Employee | The employee's picture and name. For an administrator the name becomes a selector over non-shared users. |
| Location | A selector over every location. Choosing one writes it on the employee and re-narrows the catalogue. When no location exists at all the panel says "No lunch location available." |
| Date | A date picker, defaulting to today. |
| "Passed orders" | A collapsed block, shown only when at least one current line is sent or received, holding those lines and carrying a count badge. |
| "Available Balance" | The employee's balance without the permitted overdraft, with a coin symbol. |
| "Your Order" | The heading above the open lines. When the cart's collapsed state is neither to-order nor ordered, the text "Nothing to order, add some meals to begin." is shown instead. |
| Open lines | One block per to-order or ordered line. |
| Totals | "Total", "Already Paid" and "To Pay". |
| Controls | "Order Now", shown when the cart's collapsed state is the to-order state; "Clear Order", shown when the collapsed state is the to-order or the ordered state. |

### 6.3 A line in the panel

Each line shows the quantity, the meal name and the meal figure; the state as a pill whose colour
follows the mapping to-order to the secondary colour, received to the success colour, sent to the
information colour and ordered to the primary colour; the note in a shaded box with a note symbol;
the location and the date; one row per extra with a leading plus sign and the extra figure; and, for
an open line, the decrement control, the quantity, the increment control and the line figure.

The decrement control is disabled when the line is sent or received. The increment control is
disabled when the line is sent or received, or when the balance pre-check of
[`calculations.md`](calculations.md#42-the-clients-own-pre-check) fails.

### 6.4 When there is no location

Both view kinds replace the ordinary empty-state text with the two lines "No location found" and
"Please create a location to start ordering." whenever the screen has no location, and list nothing.

---

## 7. The vendor order message

The message is rendered from the template named in
[`configuration.md`](configuration.md#10-the-vendor-order-message-template), from a payload the
dispatch operation assembles rather than from the vendor record's own fields.

### 7.1 The payload

| Key of the payload | Content |
|---|---|
| Company name | The company name of the first collected line. |
| Currency | The currency of the first collected line. |
| Vendor Contact | The vendor's Contact, used as the recipient. |
| Vendor name | The vendor's name, used in the greeting. |
| Sender address | The responsible administrator's formatted electronic mail address. |
| Total | The sum of the collected lines' total prices. |
| Lines | One entry per collected line: the meal name, the note, the quantity, the price, the extras summary, the employee name and the employee's current location name. Sorted by the location's record identifier. |
| Locations | One entry per distinct current location of the ordering employees: the name and the address. Sorted by location name. |

### 7.2 The body

| Region | Content |
|---|---|
| Header | The words "Lunch Order" in small type on the left and the company logotype on the right, unless the company still uses the default logotype. A horizontal rule below. |
| Greeting | "Dear " followed by the vendor name, then "Here is, today orders for " followed by the company name and a colon. |
| Location block | A paragraph headed "Location" followed by one line per location, reading the location name, a space, a colon, a space and the address. |
| Table | Six columns headed "Product", "Comments", "Person", "Site", "Qty" and "Price". One row per line: the meal name; the extras summary and, beneath it in grey, the note; the employee name; the location name; the quantity, right-aligned; the price formatted in the payload currency, right-aligned. |
| Total row | A row whose fifth cell reads "Total" in bold above a top rule and whose sixth cell carries the payload total, formatted in the payload currency, in bold. |
| Closing | "Do not hesitate to contact us if you have any questions." |
| Footer | The company name, then the company telephone number, electronic mail address and web address, separated by vertical bars where two or more are present, the last two rendered as links. |

**Compatibility finding.** The location block reads the list of locations from a name that the
payload does not supply, and iterates over a second name that is never set either, so the block
never renders and the vendor never sees the delivery addresses. Everything else in the body renders
correctly, because it reads the payload through the key the dispatch operation actually sets. A
corrected behaviour would read the location list from the payload key that carries it, and would
iterate over that same list.

The message's subject is the fixed text "Orders for " followed by the company name.

---

## 8. Named operations

Operations an outside caller may invoke on a record set. Each is listed with the entity it belongs
to, what it takes, what it returns and what it changes.

| Operation | Entity | Takes | Returns | Effect |
|---|---|---|---|---|
| Confirm cart | Lunch Order | a set of lines | nothing | Guards MEAL-001 and MEAL-002, writes the ordered state, then guards MEAL-020. |
| Repeat | Lunch Order | exactly one line | the screen instruction that opens the personal order list | Guards MEAL-003, then copies the line to today in the ordered state. |
| Receive | Lunch Order | a set of lines | nothing | Writes the received state. |
| Cancel | Lunch Order | a set of lines | nothing | Writes the cancelled state. |
| Reset | Lunch Order | a set of lines | nothing | Writes the ordered state. |
| Send | Lunch Order | a set of lines | nothing | Writes the sent state. |
| Notify | Lunch Order | a set of lines | nothing | Pushes the delivery notice once per employee and sets the delivery-notice flag. |
| Change quantity | Lunch Order | a set of lines and a signed step | nothing | Raises or lowers the quantity of the lines that are not sent or received, archiving a line whose quantity is at or below the size of a negative step, then guards MEAL-020. |
| Add to cart | Lunch Order | one line | the value true | Does nothing itself; it exists so that the order dialogue can be an ordinary record form rather than a dialogue on a transient record. |
| Dispatch the day | Lunch Vendor | a set of vendors | the screen instruction that shows the notice "The orders have been sent!" and closes the dialogue | For electronic mail vendors, composes and queues the message and marks their lines sent; for telephone vendors, marks their lines sent. |
| Receive the day | Lunch Vendor | a set of vendors | the screen instruction that shows the notice "The orders have been confirmed!" and closes the dialogue | Marks the vendors' sent lines of today received. |
| Send the automatic message | Lunch Vendor | exactly one vendor | nothing | The operation the per-vendor scheduled action invokes. |
| Push the notice | Lunch Alert | exactly one notice | nothing | The operation the per-notice scheduled action invokes. |
| Read the balance | Lunch Cash Move | a User and a flag saying whether to include the permitted overdraft | an amount | Computes the balance by the formula of [`calculations.md`](calculations.md#3-the-internal-account-balance). |

---

## 9. Service endpoints

Six endpoints serve the ordering screen. All six require an authenticated session and all six speak
the structured-notation remote procedure convention described in
[`../../interfaces/remote-transport-contracts.md`](../../interfaces/remote-transport-contracts.md).
All six accept an optional employee and an optional context; naming another employee without the
administration privilege is refused by rule
[MEAL-030](business-rules.md#meal-030--only-an-administrator-may-order-for-someone-else).

### 9.1 `/lunch/infos`

Returns everything the panel needs.

| Key returned | Content |
|---|---|
| Employee name | The employee's name, read with elevated rights. |
| Employee picture address | The address of the employee's small avatar image. |
| Balance | The employee's balance **without** the permitted overdraft. |
| Balance including the allowance | The employee's balance **with** the permitted overdraft. |
| Is an administrator | Whether the **requesting** user holds the administration privilege, not whether the named employee does. |
| Portal group identifier | The record identifier of the platform's portal access group, so that the client can exclude portal users from its selectors. |
| Locations | Every location, as pairs of record identifier and name. |
| Currency | The symbol and the position of the employee's company currency. |
| Current location | The employee's current location, as a pair of record identifier and name. When the employee has none, or theirs belongs to a company that is not active, the first visible location is written onto the employee and returned. |
| Notices | The message of every notice that is displayable today, whose mode is the banner mode and whose locations include the current location. |
| Total, already paid, still to pay | The three cart figures, each formatted to two decimal places. Present only when the employee has at least one current line. |
| Collapsed state | The lowest-ranked state among the current lines. Present only when the employee has at least one current line. |
| Lines | One entry per current line, sorted by date: the record identifier; a four-part meal entry of record identifier, name, the meal figure and the rounded unit price; one entry per extra of name, extra figure and rounded unit price; the quantity; the line's total price; the stored state; the translated state label; the date; the location name; the note. Present only when the employee has at least one current line. |

"Current lines" means the employee's lines dated today or later whose state is not the cancelled
state.

### 9.2 `/lunch/trash`

Cancels and then deletes the employee's current lines that are not sent and not received. Returns
nothing.

### 9.3 `/lunch/pay`

Invokes the confirm-cart operation on the employee's current lines that are in the to-order state.
Returns true when the employee had at least one current line, false when they had none. Note that
the affirmative answer does not mean that anything was confirmed: an employee whose current lines
are all already ordered receives true and nothing changes.

### 9.4 `/lunch/payment_message`

Returns the rendered fixed fragment described in
[`configuration.md`](configuration.md#11-fixed-fragments), under a single key.

### 9.5 `/lunch/user_location_set`

Takes a location and writes it as the employee's current location, with elevated rights so that the
write succeeds for an impersonated employee. Returns true.

### 9.6 `/lunch/user_location_get`

Returns the record identifier of the employee's current location when it is set and its company is
empty or among the request's active companies; otherwise the record identifier of the first location
whose company is empty or among those companies; otherwise nothing.

---

## 10. Printable documents

The domain defines no printable document. Nothing in it is designed to be produced as a page-layout
file. Two things a user may want to print exist as ordinary screens and are printed through the
platform's generic list export:

- the day's orders grouped by vendor, from "Today's Orders";
- an employee's statement, from "My Account History" or "Control Accounts".

---

## 11. Import and export

| Path | What is possible |
|---|---|
| Import | Every stored entity of the domain may be imported through the platform's generic import, keyed on the reproduced field names of [`entities.md`](entities.md). Importing a Lunch Vendor creates its scheduled action, exactly as an interactive creation does. Importing a Lunch Order bypasses no rule: the extras discipline, the balance check and the record rules all apply. |
| Export | Every stored entity may be exported through the platform's generic export. The Lunch Cash Move Report may be exported too, which is the only way to take the merged statement out of the system. |
| No dedicated format | The domain defines no file format, no structured document and no exchange schema of its own. |

---

## 12. External integrations

The domain contacts no third-party service. Its only outbound traffic is the vendor order message,
which is handed to the platform's own outgoing message queue and delivered by the mechanism
described in [`../../runtime/mail-gateway.md`](../../runtime/mail-gateway.md).

The delivery notice and the pushed notice are internal notifications, delivered through the
notification mechanism described in
[`../../runtime/notification-bus.md`](../../runtime/notification-bus.md) and, where the recipient's
preferences call for it, also by electronic mail through the same gateway. The delivery notice is
rendered with the light notification layout.
