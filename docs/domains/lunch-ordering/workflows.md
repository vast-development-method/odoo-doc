# Meal Ordering — Workflows

This document specifies the end-to-end procedures of the domain. Each procedure gives its actor, its
preconditions, its numbered steps, the records each step creates or changes, the named operations it
invokes and its failure conditions. Refusal messages are quoted in full in
[`business-rules.md`](business-rules.md); the rule identifier is given at each failure point.

The sixteen procedures are:

1. Setting up a vendor.
2. Setting up the meal catalogue.
3. Setting up locations and choosing one.
4. Browsing the catalogue.
5. Building the cart.
6. Adjusting the cart.
7. Confirming the cart.
8. Dispatching the day's orders to a vendor by electronic mail.
9. Dispatching the day's orders to a vendor by telephone.
10. Receiving the delivery and notifying employees.
11. Crediting an employee's internal account.

Two further procedures, both automatic, and three reference sections close the document:

12. Pushing a scheduled notice.
13. Withdrawing a vendor, a category or a meal.
14. The ordering screen, interaction by interaction.
15. Reviewing and settling accounts at the end of a period.
16. What can go wrong, and where it is specified.

---

## 1. Setting up a vendor

**Actor.** Meal ordering administrator.
**Precondition.** A Contact exists for the vendor, or will be created inline from the vendor form.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The administrator opens the vendor list from the configuration menu and starts a new record. | None yet. | The menu is invisible without the administration privilege. |
| 2 | The administrator names the vendor. The name is written through to the Contact. | A Contact is created inline when the administrator types a name that matches none, taking the typed name, street, second street line, city, country subdivision, postal code, country and telephone number as defaults and being marked as an organisation. | The Contact reference is required — rule MEAL-007. |
| 3 | The administrator fills the postal address, the telephone number and the electronic mail address. Each is written through to the Contact. | The Contact is updated. | The electronic mail address is required on screen when the channel is electronic mail; the telephone number is required on screen when the channel is telephone. |
| 4 | The administrator sets the company, or leaves it empty to share the vendor with every company. | The vendor's company is written, mirrored onto the Contact. | None. |
| 5 | The administrator sets the responsible administrator. The selectable set is limited to users holding the administration privilege and excludes shared users. | The vendor's responsible is written. | None. Leaving it empty leaves the automatic order message without a sender address. |
| 6 | The administrator ticks the weekdays the vendor serves and, if the arrangement ends, sets the last service date. | Seven boolean flags and the last service date are written. Clearing a weekday cancels the vendor's future orders on that weekday — transition T9 of [`state-machines.md`](state-machines.md). | None. |
| 7 | The administrator sets the time zone. It defaults to the administrator's own time zone, and to `UTC`, the zero-offset universal time zone, when the administrator has none. | The time zone is written. | Required — rule MEAL-008. |
| 8 | The administrator picks the served locations, or leaves the list empty so that the vendor serves every location. | Association rows in `lunch_location_lunch_supplier_rel`. | None. |
| 9 | The administrator picks the ordering channel and, for electronic mail, the cut-off hour and its half-day marker. | The channel, the cut-off hour and the marker are written. | The cut-off hour must be between 0 and 12 inclusive — rule MEAL-009. |
| 10 | The administrator labels the three extra groups, sets each group's discipline and fills each group's list of priced extras. | Up to three groups of Lunch Topping records, each with its group number forced by the field it was entered through. | Each label is required — rule MEAL-010. Each discipline is required. |
| 11 | The record is saved. | One Lunch Vendor. One scheduled action, created before the vendor itself, owned by the platform's root user, running once a day, initially inactive. One external identifier for the server action behind that scheduled action, named with the fixed prefix `lunch_supplier_cron_sa_` followed by the server action's record identifier, declared in the package named `lunch` and marked as not to be overwritten on package update. The scheduled action is then synchronised: it becomes active when the vendor is active and the channel is electronic mail, its name becomes the fixed prefix "Lunch: send automatic email to " followed by the vendor name, and its next due instant is computed by the rule in [`calculations.md`](calculations.md#7-next-dispatch-instant). | Any refusal above rolls the whole creation back, including the scheduled action. |

**Later changes.** Changing the name, the archive flag, the channel, the cut-off hour, the half-day
marker or the time zone re-synchronises the scheduled action. Changing the company rewrites the
company of every order of that vendor. Clearing a weekday cancels future orders on that weekday.

**Deletion.** Deleting the vendor deletes its extras by cascade, then deletes its scheduled action
and the server action behind that action. Deletion is refused while any meal points at the vendor.

---

## 2. Setting up the meal catalogue

**Actor.** Meal ordering administrator.
**Precondition.** At least one vendor exists.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The administrator creates the categories from the configuration menu: a name, optionally a company, optionally a picture. A category created without a picture starts with the packaged default meal picture. | One Lunch Product Category. The four resized copies of the picture are stored. | The name is required — rule MEAL-013. |
| 2 | The administrator creates a meal: a name, a category, a vendor, a price, optionally a description and a picture, optionally a novelty date. | One Lunch Product. Its company is taken from the vendor and stored. The four resized copies of the picture are stored. | The name, the category, the vendor and the price are all required — rules MEAL-014 to MEAL-017. The category and the favourite users must belong to a company compatible with the meal's company — rule MEAL-018. The category must not be archived — rule MEAL-005. The vendor must not be archived — rule MEAL-006. |
| 3 | The administrator repeats step 2 for every meal of every vendor. | More Lunch Product records. | As above. |
| 4 | From the category form the administrator can jump to the meals of that category through the counter control, which opens the meal screen pre-filtered and pre-defaulted on the category. | None. | The control is hidden when the category holds no meal. |

---

## 3. Setting up locations and choosing one

**Actor.** Administrator for creation; any meal orderer for the choice.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The administrator creates the locations: a name, a free-form address, optionally a company. | One Lunch Location each. | The name is required — rule MEAL-019. Holders of the ordering privilege may read and update locations but may not create them — see the access matrix in [`configuration.md`](configuration.md#5-access-rights). |
| 2 | An employee opens the ordering screen. The screen asks the service endpoint at the route `/lunch/user_location_get` for the location to show. | None. | None. |
| 3 | The endpoint returns the employee's last ordering location when it is set and its company is empty or among the employee's active companies. Otherwise it returns the oldest location whose company is empty or among the active companies. | None. | When no location at all is visible, the catalogue shows the notice "No location found" and, beneath it, "Please create a location to start ordering." and lists nothing. |
| 4 | The employee changes the location in the selector. The screen calls the service endpoint at the route `/lunch/user_location_set`. | The employee's last ordering location is written, with elevated rights so that the write succeeds for an impersonated employee. | An administrator impersonating another employee is allowed; a non-administrator attempting it is refused — rule MEAL-030. |
| 5 | The catalogue query is narrowed to meals available at the chosen location: meals whose vendor serves that location, plus meals whose vendor declares no location at all. | None. | None. |

---

## 4. Browsing the catalogue

**Actor.** Meal orderer.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The employee opens the ordering screen from the menu entry "New Order", which reaches the catalogue at the short path `lunch`. | None. | The menu is invisible without the ordering privilege. |
| 2 | The screen loads the catalogue as cards, with a side panel offering the categories and the vendors as multiple-choice filters, both showing counts. | None. | None. |
| 3 | The screen adds the location filter of step 5 above and, when the employee picks a date, the weekday filter matching that date, replacing any weekday filter already active. | None. | None. |
| 4 | The employee may narrow further with the search field on the meal name, the category, the vendor or the description, and with the ready-made filters: available today, available on each of the seven weekdays, and archived. | None. | None. |
| 5 | Each card shows the picture, the favourite marker, the name, a "New" ribbon when the novelty date has not passed, the price, the vendor and the description. | None. | None. |
| 6 | The employee clicks the favourite marker. | The employee is added to or removed from the meal's favourite list, by writing on the employee's own favourite meal list rather than on the meal. | None. |
| 7 | The employee clicks a card. The order form opens as a dialogue titled "Configure Your Order", pre-filled with the meal, and with the impersonated employee, the chosen date and the chosen location as defaults. | None yet. | None. |

---

## 5. Building the cart

**Actor.** Meal orderer, or an administrator impersonating one.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The dialogue shows the meal picture, the meal name and the running price, then one block per offered extra group, then the description, then the note field with the placeholder "Information, allergens, ...". | None yet. | None. |
| 2 | A block is shown only when the vendor has at least one extra in that group. Each block is a set of tick boxes limited to the extras of that vendor and that group. | None yet. | None. |
| 3 | When the vendor's cut-off for the chosen date has passed, the dialogue shows the warning "The orders for this vendor have already been sent." and hides the add control. | None. | The employee cannot add to the cart — rule MEAL-021 as enforced by the screen. |
| 4 | When the employee's balance would not cover this line, the dialogue shows the warning "Your wallet does not contain enough money to order that. To add some money to your wallet, please contact your lunch manager." and hides the add control. | None. | The employee cannot add to the cart — rule MEAL-020 as enforced in advance by the screen. |
| 5 | The employee presses "Add To Cart". | The dialogue saves the record, which creates the Lunch Order. The named operation invoked by the control does nothing itself; the record has already been written by the save. | The extras discipline of every offered group must be satisfied — rules MEAL-011 and MEAL-012. |
| 6 | Creation looks for an existing line of the same employee, meal, date, note, location and ordered list of extras that is still in the to-order state. | When one is found, that line's quantity rises by exactly one and no new record is created — transition T2. When none is found, a new line is created in the to-order state. | The balance after the increment must not fall below the permitted overdraft — rule MEAL-020. |
| 7 | The dialogue closes and the ordering screen refreshes its panel by calling the service endpoint at the route `/lunch/infos`. | None. | None. |

---

## 6. Adjusting the cart

**Actor.** Meal orderer, or an administrator impersonating one.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The panel lists the employee's current lines, split into an expandable "Passed orders" block for lines in the sent or received state and an open block for lines in the to-order or ordered state. | None. | None. |
| 2 | Each open line offers a decrement control and an increment control around the quantity. | None yet. | None. |
| 3 | Pressing increment invokes the quantity operation with a step of plus one. | The line's quantity rises by one, provided the line is not sent or received. The balance of the line's employee is then re-checked. | The balance must not fall below the permitted overdraft — rule MEAL-020, which rolls the increment back. |
| 4 | Pressing decrement invokes the quantity operation with a step of minus one. | When the quantity is greater than one, it falls by one. When the quantity is at or below one, the line is archived instead and its quantity is left as it was. | The balance is re-checked, but a decrement can only improve it. |
| 5 | The increment control is disabled in advance when the balance left after the employee's unpaid subtotal would not cover one more unit of that line. | None. | None. |
| 6 | Pressing "Clear Order" calls the service endpoint at the route `/lunch/trash`. | Every line of the employee dated today or later that is not cancelled and not in the sent or received state is first moved to the cancelled state and then deleted. | The deletion record rule permits deletion only in the to-order and cancelled states — rule MEAL-032. |
| 7 | The panel shows three totals: "Total" over every current line, "Already Paid" over the lines whose state is not to-order, and "To Pay" as the difference. | None. | None. |

---

## 7. Confirming the cart

**Actor.** Meal orderer, or an administrator impersonating one.
**Precondition.** The cart's collapsed state is the to-order state, that is at least one line is
still in the to-order state.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The employee presses "Order Now". The screen calls the service endpoint at the route `/lunch/pay`. | None yet. | The control is hidden unless the cart's collapsed state is to-order. |
| 2 | The endpoint collects the employee's lines dated today or later that are not cancelled, keeps only those in the to-order state, and invokes the confirm-cart operation on them. | None yet. | When the employee has no current line at all the endpoint answers negatively and nothing happens. |
| 3 | The operation checks each line's vendor against the line's own date. | None. | Refusal MEAL-001 when any vendor does not serve on the line's date. |
| 4 | The operation checks that no selected line's meal is archived. | None. | Refusal MEAL-002. |
| 5 | The operation writes the ordered state on every selected line. Because the write touches the state, the merge machinery runs: each written line looks for another line, of the same employee, meal, date, note, location and extras, already in the ordered state. | A line that finds such a match is archived and the matched line's quantity rises by the archived line's quantity. Lines with no match simply change state. | None; see the compatibility finding in [`state-machines.md`](state-machines.md#18-merge-on-write-and-what-it-does-to-a-state-change) for the case where the match is already sent or received. |
| 6 | The operation re-computes the balance of every employee in the selection. | None. | Refusal MEAL-020, which rolls the entire confirmation back so that no line ends in the ordered state. |
| 7 | The endpoint answers affirmatively and the screen reloads the panel. Every confirmed line now appears as a negative row on the employee's account statement. | None. | None. |

---

## 8. Dispatching the day's orders to a vendor by electronic mail

**Actor.** The scheduling service, or a meal ordering administrator pressing the dispatch control.
**Precondition.** The vendor's channel is electronic mail.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The vendor's scheduled action reaches its next due instant and runs the dispatch operation for that one vendor. Alternatively the administrator opens "Today's Orders", which groups the day's lines by vendor, and presses "Send Orders" on a vendor group. The control is shown only when that vendor has at least one active ordered line dated today. | None yet. | None. |
| 2 | The operation checks that the vendor serves today. | None. | The run ends silently, without an error and without changing anything. |
| 3 | The operation checks that the channel is electronic mail. | None. | Refusal MEAL-004. Only reachable when the operation is invoked directly on a telephone vendor. |
| 4 | The operation collects the vendor's active lines dated today, in the vendor's own time zone, whose state is ordered, sorted by employee and then by meal. | None. | When the set is empty the run ends silently. |
| 5 | The operation assembles the message payload: the company name of the first line, its currency, the vendor's Contact, the vendor name, the responsible administrator's formatted electronic mail address as the sender, and the sum of the line prices as the total. | None. | None. |
| 6 | The operation assembles one entry per line — meal name, note, quantity, price, extras summary, employee name and the employee's current location name — sorted by the location's record identifier, and one entry per distinct location — name and address — sorted by location name. | None. | None. |
| 7 | The operation renders the message template and queues the message to the vendor's Contact, with the subject "Orders for " followed by the company name, and the sender taken from the payload. | One outgoing message and its tracking record, owned by the messaging domain. | None. |
| 8 | The operation marks every collected line as sent. | Every collected line moves to the sent state. Each such line leaves the account statement until it is marked received. | See the compatibility finding on merge targets in [`state-machines.md`](state-machines.md#18-merge-on-write-and-what-it-does-to-a-state-change). |
| 9 | When the administrator triggered the dispatch from a screen, a success notice is shown reading "The orders have been sent!" and the dialogue closes. | None. | None. |

---

## 9. Dispatching the day's orders to a vendor by telephone

**Actor.** Meal ordering administrator.
**Precondition.** The vendor's channel is telephone.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The administrator opens "Today's Orders" and reads the vendor's group: one line per employee and meal, with the extras summary, the note, the employee, the location, the price and the state. | None. | None. |
| 2 | The administrator telephones the vendor and places the order. | Nothing in the system. | None. |
| 3 | The administrator presses "Send Orders" on the vendor group. The operation splits the selected vendors into those whose channel is electronic mail, which follow procedure 8, and the rest. | None yet. | None. |
| 4 | For the telephone vendors the operation collects their active ordered lines dated today, in each vendor's time zone, and marks them sent without composing any message. | Every collected line moves to the sent state. | None. |
| 5 | A success notice is shown reading "The orders have been sent!". | None. | None. |

An administrator may instead press the send control on a single card, which marks that one line sent
without touching the rest of the vendor's day.

---

## 10. Receiving the delivery and notifying employees

**Actor.** Meal ordering administrator.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The delivery arrives. The administrator opens "Today's Orders" or "Control Vendors". | None. | None. |
| 2 | The administrator presses "Confirm Orders" on the vendor group. The control is shown only when the vendor has at least one active sent line dated today. | The vendor's active sent lines dated today move to the received state and re-enter the account statement. A success notice reads "The orders have been confirmed!". | None; the operation has no guard. |
| 3 | Alternatively the administrator selects lines in the list and presses the header control "Receive", or the per-line receipt control, or the card receipt control, or runs the bound server action "Lunch: Receive meals". | The selected lines move to the received state. | None. |
| 4 | The administrator presses the notify control on the received lines, or runs the bound server action "Lunch: Send notifications". | Lines already notified are dropped. For each remaining distinct employee, one notification is posted to that employee's Contact with the subject "Lunch notification" and, as the body, the company's delivery notice message rendered in that employee's language, using the light notification layout. Every selected line has its delivery-notice flag set. | The operation ends silently when every selected line was already notified. |
| 5 | A line that goes wrong is cancelled instead, with the per-line or per-card cancel control, or with the bound server action "Lunch: Cancel meals". | The line moves to the cancelled state and leaves the account statement. | None. |
| 6 | A line cancelled by mistake is put back with the reset control. | The line returns to the ordered state. | None. |
| 7 | An employee who wants the same meal again presses the repeat control on a received line in their order history. | A copy of the line is created, dated today and already in the ordered state, carrying the same meal, extras, note, quantity, employee and location. The screen navigates to the personal order list. | Refusal MEAL-003 when the vendor does not serve today. The control is hidden when the balance would not cover the repeat, but the operation itself performs no balance check — see rule MEAL-022. |

---

## 11. Crediting an employee's internal account

**Actor.** Meal ordering administrator.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The employee hands money to the administrator. | Nothing in the system. Nothing in the ledger — see [`accounting-effects.md`](accounting-effects.md). | None. |
| 2 | The administrator opens "Cash Moves" from the manager menu and creates a movement: the employee, the date, the amount, a description. | One Lunch Cash Move with the currency defaulted to the administrator's current company currency. | The date, the amount and the currency are required — rules MEAL-023 to MEAL-025. |
| 3 | The movement immediately becomes a positive row on that employee's account statement and raises the balance. | None beyond step 2. | None. |
| 4 | The administrator reviews balances on "Control Accounts", which shows the merged statement grouped by employee with a total per group. | None. | None. |
| 5 | The employee reviews their own statement on "My Account History", which shows the same merged rows filtered to the employee, with a summed total. | None. | None. |
| 6 | A correction is entered as a further movement with a negative amount. | One more Lunch Cash Move. | None. |

---

## 12. Pushing a scheduled notice

**Actor.** The scheduling service.
**Precondition.** A Lunch Alert exists whose mode is the pushed mode and whose scheduled action is
active.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The notice's scheduled action reaches its next due instant and runs the push operation for that one notice. | None yet. | None. |
| 2 | The operation checks that the notice is displayable today: today's weekday flag is set and the show-until date is either empty or strictly later than today. | When it is not displayable and the show-until date is strictly earlier than today, the scheduled action is deleted and the notice's reference to it is cleared. | The run ends without sending — see the gap on the show-until date in [`state-machines.md`](state-machines.md#54-the-gap-on-the-show-until-date-itself). |
| 3 | The operation checks that the notice is active and in the pushed mode. | None. | Refusal MEAL-026. |
| 4 | The operation builds the audience: every order whose state is not cancelled; narrowed to orders whose employee's current location is among the notice's locations, when the notice declares any; narrowed to orders dated on or after a cut-off date when the audience is not everyone. The cut-off date is today minus one week, four weeks or fifty-two weeks for the last-week, last-month and last-year audiences respectively. | None. | None. |
| 5 | The operation collects the Contacts of the employees behind those orders. | None. | When the set is empty nothing is posted. |
| 6 | The operation posts one notification on the notice record itself, with the subject "Your Lunch Order" and the notice message as the body, addressed to those Contacts. | One message and its notifications, owned by the messaging domain. | None. |

**Banner notices** need no procedure: the ordering screen asks for every notice that is displayable
today, whose mode is the banner mode and whose locations include the employee's current location,
and renders each message inside a single warning block at the top of the panel.

---

## 13. Withdrawing a vendor, a category or a meal

**Actor.** Meal ordering administrator.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | To stop a vendor for a period, the administrator clears the weekday flags that no longer apply. | Every to-order and ordered line of that vendor dated today or later that falls on a cleared weekday moves to the cancelled state. | None. |
| 2 | To stop a vendor from a date onwards, the administrator sets the last service date. | Nothing is cancelled. The vendor becomes unavailable on that date and every later date. | None. |
| 3 | To stop a vendor entirely, the administrator archives it. | Every meal of the vendor is archived. The scheduled action becomes inactive. | None. |
| 4 | To bring the vendor back, the administrator un-archives it. | Every meal of the vendor is un-archived; the scheduled action becomes active again when the channel is electronic mail. | Refusal MEAL-005 when any of those meals sits in a still-archived category, which fails the whole un-archive. |
| 5 | To withdraw a category, the administrator archives it. | Every meal in the category is archived. | None. |
| 6 | To bring the category back, the administrator un-archives it. | Every meal in it whose vendor is active is un-archived; the rest stay archived. | None. |
| 7 | To withdraw a single meal, the administrator archives it. | The meal leaves the catalogue. Existing orders keep pointing at it. Confirming such an order is refused with MEAL-002. | None. |
| 8 | To delete a vendor outright, the administrator deletes the record. | The vendor's extras are deleted by cascade; then the scheduled action and the server action behind it are deleted. | Deletion is refused while any meal points at the vendor. |

---

## 14. The ordering screen, interaction by interaction

This procedure specifies what the ordering screen does at each moment, because the screen is the
only place where several of the domain's rules are visible and because a rebuild must reproduce the
sequence of service calls to reproduce the observed behaviour.

| # | Trigger | What the screen does | What the server does | What the screen then shows |
|---|---|---|---|---|
| 1 | The screen is opened. | Calls the endpoint at the route `/lunch/user_location_get` with no employee named. | Returns the employee's current location when it is set and visible to one of the request's active companies, otherwise the oldest visible location, otherwise nothing. | Remembers the location and narrows the catalogue query to the meals available there. |
| 2 | The catalogue is about to load. | Adds a condition on the availability-at-location field to whatever the search panel already produced. | Resolves that condition into: the meal's vendor serves the given location, **or** the meal's vendor declares no location at all. | The card or row set. When the screen has no location it lists nothing and shows the two lines "No location found" and "Please create a location to start ordering." |
| 3 | The panel is about to render. | Calls the endpoint at the route `/lunch/infos`. | Assembles the employee's name and picture address, both balances, the administrator flag of the **requesting** user, the portal group identifier, every location, the currency symbol and position, the current location, the banner notices, and — when the employee has current lines — the three totals, the collapsed state and the line entries. | The whole panel. |
| 4 | The employee picks a date. | Switches off any active weekday filter and switches on the one matching the chosen date; stores the date so that the order dialogue receives it as a default. | Nothing. | A catalogue narrowed to the meals of that weekday. |
| 5 | An administrator picks another employee. | Stores that employee and re-calls the endpoint at the route `/lunch/infos` naming them. | Checks the impersonation rule, then answers for the named employee. | A panel showing the named employee's balance, location and lines. |
| 6 | The employee picks a location. | Calls the endpoint at the route `/lunch/user_location_set`. | Writes the location onto the employee with elevated rights. | The panel is reloaded and the catalogue re-narrowed. |
| 7 | The employee clicks a card or a row. | Opens the order dialogue titled "Configure Your Order", passing the meal as a default and, when they are set, the impersonated employee, the chosen date and the chosen location. | Nothing until the dialogue saves. | The dialogue. |
| 8 | The dialogue is saved by "Add To Cart". | Saves the record, then invokes the named add-to-cart operation, which does nothing, then closes and asks the panel to refresh. | Creates or merges the line, running the extras discipline and the balance check. | The refreshed panel. |
| 9 | The employee presses an increment or decrement control. | Calls the quantity operation on that line with a step of plus or minus one, then refreshes the panel. | Adjusts the quantity or archives the line, then re-checks the balance. | The refreshed panel. |
| 10 | The employee presses "Order Now". | Calls the endpoint at the route `/lunch/pay`, then refreshes the panel. | Confirms the employee's to-order lines. | The refreshed panel, with the confirmed lines now shown as ordered. |
| 11 | The employee presses "Clear Order". | Calls the endpoint at the route `/lunch/trash`, then refreshes the panel. | Cancels and deletes the employee's lines that are not sent and not received. | The refreshed panel, emptied of those lines. |
| 12 | The screen is narrow. | Renders a bottom bar with a control reading "Your Cart" and the cart total, which opens the panel as a sliding drawer. | Nothing. | The same panel content in a drawer. |

**Failure conditions of the sequence.** Every call in steps 3, 5, 6, 10 and 11 that names another
employee is refused for a non-administrator with the impersonation message. Step 8 may be refused by
the extras discipline or by the balance check. Step 10 may be refused by the availability check, the
archived-meal check or the balance check. In every case the screen reports the refusal and the panel
is left as it was, because the whole transaction is rolled back.

---

## 15. Reviewing and settling accounts at the end of a period

**Actor.** Meal ordering administrator, then the accountant.

| Step | What happens | Records created or changed | Failure |
|---|---|---|---|
| 1 | The administrator opens "Control Accounts", which shows the merged statement grouped by employee with a summed total per group and a grand total. | None. | None. |
| 2 | The administrator reads each employee's balance. A negative balance means the employee has consumed more than they have paid in; a positive balance means the opposite. | None. | None. |
| 3 | For each employee in debt, the administrator collects the money and records a Lunch Cash Move with the collected amount. | One Lunch Cash Move per employee. | Rules MEAL-023 to MEAL-025. |
| 4 | Where the employer recovers through the payroll instead, the administrator records the same amounts as Lunch Cash Move records anyway, so that the internal balances return to zero, and passes the figures to the payroll capability. | One Lunch Cash Move per employee. | Nothing links the two automatically; an administrator who forgets step 4 leaves every recovered employee permanently in debt inside this domain. |
| 5 | The accountant reads the grand total of the same screen and writes the entries described in [`accounting-effects.md`](accounting-effects.md), in the domains that own them. | Journal entries in other domains. | None here. |
| 6 | Nothing in this domain is closed, locked or carried forward. The statement remains a complete history from the first movement onwards. | None. | None. |

---

## 16. What can go wrong, and where it is specified

| Symptom | Cause | Where it is specified |
|---|---|---|
| The catalogue is empty although meals exist. | The screen has no location, or the vendors of the meals do not serve the chosen location, or the chosen weekday is not served. | Procedures 3 and 4; rule MEAL-034 for the company case. |
| The add control is missing on the dialogue. | The vendor's cut-off has passed for the chosen date, or the balance would not cover the line. | Rules MEAL-021 and MEAL-046. |
| Confirming the cart is refused. | The vendor does not serve one line's date, a meal is archived, or the balance would fall below the permitted overdraft. | Rules MEAL-001, MEAL-002 and MEAL-020. |
| A quantity disappears after receipt. | An identical line was merged into a line already sent or received. | The compatibility finding in [`state-machines.md`](state-machines.md#18-merge-on-write-and-what-it-does-to-a-state-change). |
| A balance rises between dispatch and receipt. | Sent lines are absent from the statement. | The compatibility finding in [`entities.md`](entities.md#82-nature-and-composition). |
| A notice is never pushed on its last day. | The displayability rule and the scheduled action's activity rule compare the show-until date differently. | The compatibility finding in [`state-machines.md`](state-machines.md#54-the-gap-on-the-show-until-date-itself). |
| The vendor message shows no delivery addresses. | The location block reads a name the payload does not supply. | The compatibility finding in [`interfaces.md`](interfaces.md#7-the-vendor-order-message). |
| Un-archiving a vendor fails. | One of its meals sits in a still-archived category. | The compatibility finding in [`entities.md`](entities.md#16-archival-and-company-behaviour). |
| Historical orders change company. | The vendor's company was rewritten. | Rule MEAL-048. |
| Two identical lines exist where one was expected. | Two simultaneous additions each found no match. | Rule MEAL-055. |
