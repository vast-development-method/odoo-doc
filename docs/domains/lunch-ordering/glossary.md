# Meal Ordering — Glossary

Every term of the domain, defined. Terms that name a stored entity carry the reproduced transport
name in code font. Terms that name a stored value carry that value in code font. Terms whose
definition is a label a client displays carry it in quotation marks.

---

## A

**Account movement.** See *Lunch Cash Move*.

**Account statement.** The merged, read-only view of an employee's internal account: one positive
row per manually recorded movement and one negative row per committed order. See *Lunch Cash Move
Report*.

**Add-control flag.** The computed flag `display_add_button` on a Lunch Order, called in prose the
balance-allows-one-more flag. True when the employee's balance, after subtracting the total of their
other to-order lines of the same date, still reaches this line's total price. It hides or shows the
"Add To Cart" control on the order dialogue and the repeat control on the order list. Its arithmetic
is in [`calculations.md`](calculations.md#4-whether-one-more-of-a-line-is-affordable).

**Administration privilege.** The access group whose reproduced key is `group_lunch_manager` and
whose name is "Administrator". It implies the ordering privilege.

**Archive flag.** The boolean `active` present on the Lunch Vendor, the Lunch Product, the Lunch
Product Category, the Lunch Order and the Lunch Alert. A false value hides the record from every
default list without deleting it. The four archive machines are in
[`state-machines.md`](state-machines.md).

**Audience.** The stored value in the field `recipients` of a Lunch Alert, deciding who receives a
pushed notice: `everyone`, `last_week`, `last_month` or `last_year`. Only the pushed mode reads it.

**Available today.** The computed flag `available_today`. On a Lunch Vendor it means the vendor
serves on today's weekday, today's date read in the vendor's own time zone, and the last service
date has not been reached. On a Lunch Alert it means the notice applies on today's weekday and the
show-until date has not been reached.

---

## B

**Balance.** The figure the spending control tests: the sum of the employee's account statement rows,
rounded to two places, plus the company's permitted overdraft. Where the ordering screen shows the
label "Available Balance" the permitted overdraft is omitted.

**Banner mode.** The stored value `alert` in the field `mode` of a Lunch Alert, labelled "Alert in
app". A notice in this mode is shown as a warning block at the top of the ordering screen to
employees whose current location is among the notice's locations. It has no schedule.

---

## C

**Cancelled.** The stored state value `cancelled` of a Lunch Order, labelled "Cancelled". The line
will not be delivered and carries no charge.

**Cart.** Not an entity. The set of an employee's own Lunch Order records dated today or later whose
state is not the cancelled state. The ordering screen's side panel renders it.

**Cart collapsed state.** The lowest-ranked state among the cart's lines, by the rank `new`,
`ordered`, `sent`, `confirmed`, `cancelled`. It decides which panel controls are offered.

**Category.** See *Lunch Product Category*.

**Channel.** The stored value in the field `send_by` of a Lunch Vendor, labelled "Send Order By",
deciding how the day's order reaches the vendor: `phone` or `mail`. The channel decides whether a
scheduled action runs at all and whether the cut-off is meaningful.

**Contact.** The party record a Lunch Vendor stands for, owned by
[Contacts and Organizations](../contacts-and-organizations/). Its transport name is `res.partner`.

**Current lines.** An employee's Lunch Order records dated today or later whose state is not the
cancelled state. The six service endpoints all work on this set.

**Cut-off hour.** The decimal hour in the field `automatic_email_time` of a Lunch Vendor, labelled
"Order Time", between 0 and 12 inclusive, paired with a half-day marker. It is the hour of the
vendor's own day at which the day's order is dispatched.

**Cut-off passed.** The computed flag `order_deadline_passed`. On a Lunch Vendor it is true, for an
electronic mail vendor, once the current instant in the vendor's time zone is past the cut-off
instant; for a telephone vendor it is true whenever the vendor does not serve today. On a Lunch Order
it is true when the order date is in the past, equal to the vendor's flag when the order date is
today, and false when the order date is in the future.

---

## D

**Delivery indicator.** The stored value in the field `delivery` of a Lunch Vendor: `delivery` or
`no_delivery`. Informational only; no rule reads it.

**Delivery notice.** The message pushed to an employee when an administrator presses the notify
control on one of that employee's received lines. Its subject is the fixed text "Lunch notification"
and its body is the company's delivery notice message, rendered in the employee's own language.

**Delivery notice flag.** The boolean `notified` on a Lunch Order. It blocks a second push.

**Discipline.** The stored value in one of the three fields `topping_quantity_1`,
`topping_quantity_2` and `topping_quantity_3` of a Lunch Vendor, deciding how many extras of that
group a line must carry: `0_more` (none or more), `1_more` (one or more) or `1` (exactly one).

**Dispatch.** Passing the day's orders to a vendor: composing and queueing the order message for an
electronic mail vendor, or simply marking the lines sent for a telephone vendor. Either way the
lines move to the `sent` state.

---

## E

**Electronic mail vendor.** A Lunch Vendor whose channel is `mail`. Its day is dispatched
automatically by its own scheduled action at its own cut-off instant.

**Employee.** In this folder, the internal user who orders. The User entity is owned by
[Identity and Access](../identity-and-access/); this domain adds two fields to it.

**Extra.** See *Lunch Topping*.

**Extra group.** One of the three independent sets of extras a Lunch Vendor may offer. Each group
has a label, a discipline and a list. A line's extras of a group are held in the matching one of the
three fields `topping_ids_1`, `topping_ids_2` and `topping_ids_3`.

**Extras summary.** The stored text `display_toppings` on a Lunch Order: the names of the line's
distinct extras joined by the three-character separator space, plus sign, space.

---

## G

**Group number.** The whole number `topping_category` on a Lunch Topping, saying which of the
vendor's three extra groups the extra belongs to. The screens only ever produce 1, 2 and 3.

---

## H

**Half-day marker.** The stored value `am` or `pm` in the field `moment` of a Lunch Vendor, or
`notification_moment` of a Lunch Alert. It says whether the decimal hour is counted in the morning
half of the day or in the afternoon half.

---

## I

**Impersonation.** An administrator operating the ordering screen on behalf of another employee. It
is permitted only to holders of the administration privilege and is refused otherwise with the
message quoted in
[`business-rules.md`](business-rules.md#meal-030--only-an-administrator-may-order-for-someone-else).

**Internal account.** The private, single-entry running total this domain keeps per employee. It is
not a ledger; the reasoning is in [`accounting-effects.md`](accounting-effects.md).

---

## L

**Last service date.** The date in the field `recurrency_end_date` of a Lunch Vendor, labelled
"Until". It is the first date of unavailability, not the last date of service: the vendor is
unavailable on that date and on every later date.

**Line.** One Lunch Order record. There is no order header, so a line is the whole order.

**Location.** See *Lunch Location*.

**Lunch Alert** (`lunch.alert`). A notice to employees, shown as a banner on the ordering screen or
pushed as a conversation message on a schedule.

**Lunch Cash Move** (`lunch.cashmove`). One manually recorded movement on an employee's internal
account, normally a credit for money handed over.

**Lunch Cash Move Report** (`lunch.cashmove.report`). The read-only database view that merges manual
movements with order charges into one statement per employee.

**Lunch Location** (`lunch.location`). A delivery point of the employer, with a free-form address.
Its registered full name is "Lunch Locations"; this folder uses the singular.

**Lunch Order** (`lunch.order`). One order line: an employee, a meal, a quantity, extras, a note, a
date, a location and a state.

**Lunch Product** (`lunch.product`). One orderable meal offered by one vendor at one price. Called a
*meal* throughout this folder.

**Lunch Product Category** (`lunch.product.category`). A grouping of meals such as sandwich, pizza,
burger or drinks.

**Lunch Topping** (`lunch.topping`). A priced extra belonging to one vendor and to one of that
vendor's three extra groups. Its registered full name is "Lunch Extras"; this folder calls a single
record an *extra*.

**Lunch Vendor** (`lunch.supplier`). A meal vendor: a Contact plus an availability pattern, a
channel, a cut-off, served locations and three groups of extras. Called a *vendor* throughout this
folder.

---

## M

**Meal.** See *Lunch Product*.

**Meal figure.** The amount shown next to the meal name on a line of the ordering panel: the meal
price rounded to two places, multiplied by the line quantity. It excludes the extras, which carry
their own figures.

**Merge.** The machinery that collapses two identical lines into one. On creation it looks only for
a line in the to-order state; on a write that touches the note, the extras or the state it looks for
a line already in the state being written. Its full behaviour, including where it loses a quantity,
is in
[`state-machines.md`](state-machines.md#18-merge-on-write-and-what-it-does-to-a-state-change).

**Merge identity.** The six values that decide whether two lines are identical: the employee, the
meal, the date, the note, the location and the ordered list of extras.

---

## N

**New.** The stored state value `new` of a Lunch Order, labelled "To Order". The line is in the
employee's cart and nothing is committed.

**Note.** The free text in the field `note` of a Lunch Order, labelled "Notes", shown to the vendor
in the order message. It takes part in the merge identity.

**Notice.** See *Lunch Alert*.

**Novelty date.** The date in the field `new_until` of a Lunch Product, labelled "New Until". The
meal carries the pill "New" up to and including that date.

---

## O

**Ordered.** The stored state value `ordered` of a Lunch Order, labelled "Ordered". The employee has
confirmed the cart; the charge is committed and the vendor has not been told yet.

**Ordering privilege.** The access group whose reproduced key is `group_lunch_user` and whose name
is "User : Order your meal".

**Ordering screen.** The meal catalogue with its permanent side panel, reachable from the menu entry
"New Order" and at the short path `lunch`. It is specified in
[`interfaces.md`](interfaces.md#6-the-ordering-screen).

---

## P

**Panel.** The side region of the ordering screen: notices, employee, location, date, passed orders,
balance, the cart lines, the totals and the two controls.

**Permitted overdraft.** The company amount in the field `lunch_minimum_threshold`, presented on the
settings screen as "Maximum Allowed Overdraft". It is added to the statement sum before the balance
is tested, so it is how far below zero an employee may go.

**Pushed mode.** The stored value `chat` in the field `mode` of a Lunch Alert, labelled "Chat
notification". A notice in this mode is delivered as a conversation notification by its own
scheduled action.

---

## R

**Received.** The stored state value `confirmed` of a Lunch Order, labelled "Received". The delivery
has arrived. Note that the stored value and the label differ.

**Repeat.** The operation that copies a received line to today in the ordered state. Also called
re-ordering; the control is labelled "Re-order".

**Responsible.** The administrator named in the field `responsible_id` of a Lunch Vendor. Their
formatted electronic mail address becomes the sender of the vendor order message.

---

## S

**Scheduled action.** The background job record this domain creates one of per vendor and per pushed
notice. The entity is owned by [Automation and Integration](../automation-and-integration/).

**Sent.** The stored state value `sent` of a Lunch Order, labelled "Sent". The line has been passed
to the vendor. A line in this state is absent from the account statement, which is the compatibility
finding in [`entities.md`](entities.md#82-nature-and-composition).

**Served locations.** The locations a Lunch Vendor declares it serves, in the field
`available_location_ids`. An empty list means every location.

**Show-until date.** The date in the field `until` of a Lunch Alert, labelled "Show Until". It is the
first date on which the notice is no longer displayed.

**Spendable amount.** The intermediate figure of the add-control computation: the balance including
the permitted overdraft, less the total of the employee's other to-order lines for the same date.

**Statement.** See *Account statement*.

**Statement sum.** The unrounded total of an employee's account statement rows, before the
permitted overdraft is added.

---

## T

**Telephone vendor.** A Lunch Vendor whose channel is `phone`. It has no automatic dispatch, its
cut-off flag is false whenever it serves today, and its day is marked sent by the grouped dispatch
control.

**Time zone.** The stored value in the field `tz` of a Lunch Vendor or a Lunch Alert, labelled
"Timezone". It is the zone in which that record's day, weekday pattern and hour are measured; its
default is the creating user's own zone, or the zero-offset universal zone `UTC` when that user has
none.

**To-order.** The label of the stored state value `new`.

**Total price.** The stored monetary field `price` on a Lunch Order, labelled "Total Price": the
quantity multiplied by the sum of the meal price and every distinct extra price, rounded to the
currency's precision.

---

## U

**Unit price of a panel line.** The client-side figure used by the increment control: the meal price
rounded to two places plus each extra price rounded to two places, without the quantity.

---

## V

**Vendor.** See *Lunch Vendor*. The word is also the label of the field that points at the vendor
from a meal and from an order line.

---

## W

**Wallet.** The word the user-facing messages use for the internal account. It appears in the
spending refusal, in the balance warning on the order dialogue and in the top-up help text. This
folder uses "internal account" in prose and reproduces "wallet" only inside quoted messages.

**Weekday flags.** The seven booleans `mon`, `tue`, `wed`, `thu`, `fri`, `sat` and `sun` present on
both the Lunch Vendor and the Lunch Alert. On a vendor they default to true for Monday to Friday and
false for Saturday and Sunday; on a notice all seven default to true. Clearing a vendor weekday
cancels that vendor's future orders on that weekday.
