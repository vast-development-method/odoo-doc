# Meal Ordering — Business rules

Every validation, constraint, invariant, permission check and locking rule of the domain is listed
here with a stable identifier, the condition that makes it fire, the exact text shown when it
refuses, a description of each placeholder in words, the point at which it is enforced and its
severity.

Identifiers use the prefix `MEAL` and a three-digit sequence number. They are stable within this
folder: a rule keeps its number even if the surrounding text is rewritten. The index is in
[section 9](#9-rule-index).

Severities:

- **Blocking** — the operation is refused and the whole transaction is rolled back.
- **Silent** — the operation returns without doing anything and without telling the caller.
- **Screen** — the control is hidden or disabled before the operation can be invoked; the underlying
  operation does not repeat the check.
- **Structural** — the platform refuses the write because a field is required, a reference is
  missing or a table constraint fails.

---

## 1. Ordering rules

### MEAL-001 — The vendor must serve on the order date

- **Applies to.** The confirm-cart operation on Lunch Order.
- **Condition.** For any selected line, the line's vendor does not serve on the line's own order
  date: the weekday flag matching that date is clear, or the vendor's last service date is set and
  the order date is on or after it.
- **Message.** "The vendor related to this order is not available at the selected date."
- **Placeholders.** None.
- **Enforced at.** The start of the confirm-cart operation, line by line, before any state is
  written.
- **Severity.** Blocking. A single failing line refuses the whole cart.

### MEAL-002 — The meal must not be archived

- **Applies to.** The confirm-cart operation on Lunch Order.
- **Condition.** Any selected line points at a meal whose archive flag is false.
- **Message.** "Product is no longer available."
- **Placeholders.** None.
- **Enforced at.** The confirm-cart operation, after MEAL-001 and before any state is written.
- **Severity.** Blocking.

### MEAL-003 — The vendor must serve today for a repeat

- **Applies to.** The repeat operation on Lunch Order.
- **Condition.** The vendor of the line being repeated does not serve today, today being read in the
  vendor's own time zone.
- **Message.** "The vendor related to this order is not available today."
- **Placeholders.** None.
- **Enforced at.** The repeat operation, before the copy is made.
- **Severity.** Blocking.

### MEAL-011 — A group whose discipline is one or more must carry at least one extra

- **Applies to.** Lunch Order, on creation and on every write that touches any of the three extras
  fields.
- **Condition.** For a group number between one and three: the vendor offers at least one extra in
  that group, the group's discipline is one or more, and the line carries no extra of that group.
- **Message.** "You should order at least one %s" where the placeholder is replaced by the vendor's
  label for that extra group — the first, second or third label according to the group number.
- **Enforced at.** The constraint that watches the three extras fields, after the write.
- **Severity.** Blocking.

### MEAL-012 — A group whose discipline is exactly one must carry exactly one extra

- **Applies to.** Lunch Order, on creation and on every write that touches any of the three extras
  fields.
- **Condition.** For a group number between one and three: the vendor offers at least one extra in
  that group, the group's discipline is exactly one, and the line carries a number of extras of that
  group other than one — none, or two or more.
- **Message.** "You have to order one and only one %s" where the placeholder is replaced by the
  vendor's label for that extra group.
- **Enforced at.** The same constraint as MEAL-011, evaluated for each of the three groups in turn.
- **Severity.** Blocking.

### MEAL-021 — Orders may not be added after the vendor's cut-off

- **Applies to.** The order dialogue.
- **Condition.** The line's cut-off flag is true: the order date is before today, or the order date
  is today and the vendor's own cut-off flag is true.
- **Message shown.** The warning block "The orders for this vendor have already been sent."
- **Enforced at.** The order dialogue, which hides the add control.
- **Severity.** Screen. The confirm-cart operation performs no cut-off check, so a line that is
  already in the cart when the cut-off passes can still be confirmed, and an administrator who
  creates a line directly is not stopped. **Compatibility finding.** A corrected behaviour would
  repeat the check inside the confirm-cart operation, so that the cut-off cannot be bypassed by
  keeping a dialogue open or by writing the record directly.

### MEAL-022 — A repeat performs no balance check

- **Applies to.** The repeat operation on Lunch Order.
- **Condition.** The employee's balance would fall below the permitted overdraft once the copy is
  charged.
- **Behaviour observed.** The repeat control is hidden unless the balance-allows-one-more flag is
  true, but the operation itself makes the copy in the ordered state without re-checking the
  balance, so invoking it directly overdraws the account.
- **Severity.** Screen only. **Compatibility finding.** A corrected behaviour would run the balance
  check after the copy, exactly as the confirm-cart operation does, and roll the copy back when it
  fails.

### MEAL-045 — A repeated line inherits the delivery-notice flag

- **Applies to.** The repeat operation on Lunch Order.
- **Condition.** The line being repeated has already had its delivery notice pushed.
- **Behaviour observed.** The copy carries the delivery-notice flag forward, so the notify operation
  drops the copy from its selection and the employee is never told that the repeated meal arrived.
- **Severity.** Silent. **Compatibility finding.** A corrected behaviour would reset the
  delivery-notice flag on the copy, as it resets the state and the date.

---

## 2. Spending rules

### MEAL-020 — The balance may not fall below the permitted overdraft

- **Applies to.** Lunch Order, at every point that raises what an employee owes: the confirm-cart
  operation, the quantity operation used by both the increment control and the merge, and therefore
  also the creation path that merges into an existing line.
- **Condition.** After the change, and after every pending write has been flushed, the sum of the
  employee's account statement rows plus the company's permitted overdraft is strictly less than
  zero. The exact arithmetic is in
  [`calculations.md`](calculations.md#3-the-internal-account-balance).
- **Message.** "Oh no! You don’t have enough money in your wallet to order your selected lunch! Contact your lunch manager to add some money to your wallet."
- **Placeholders.** None. The apostrophe is the typographic right single quotation mark; the text is
  reproduced exactly.
- **Enforced at.** After the write, so the whole operation is rolled back when it fires.
- **Severity.** Blocking.

### MEAL-046 — The add control is hidden before the balance is exhausted

- **Applies to.** The order dialogue and the catalogue cards.
- **Condition.** The balance left after subtracting the total of the employee's other to-order lines
  for the same date does not reach this line's price.
- **Message shown.** "Your wallet does not contain enough money to order that. To add some money to your wallet, please contact your lunch manager."
- **Enforced at.** The balance-allows-one-more computation, which drives the visibility of the add
  control and of the repeat control.
- **Severity.** Screen. The blocking equivalent is MEAL-020.

### MEAL-047 — The top-up help text

- **Applies to.** The service endpoint at the route `/lunch/payment_message`.
- **Behaviour.** The endpoint renders one fixed fragment whose whole content is the sentence "To add
  some money to your wallet, please contact your lunch manager." The system offers no self-service
  top-up: money reaches an account only through a Lunch Cash Move recorded by an administrator.
- **Severity.** Informational.

---

## 3. Catalogue rules

### MEAL-005 — An active meal may not sit in an archived category

- **Applies to.** Lunch Product, on creation and on every write that touches the archive flag or the
  category.
- **Condition.** The meal is active and its category is archived.
- **Message.** "The following product categories are archived. You should either unarchive the categories or change the category of the product." followed by a line break and then the names of the archived categories of the offending meals, one per line.
- **Placeholders.** The trailing block is the list of archived category names, joined by line breaks.
- **Enforced at.** The constraint that watches the archive flag and the category.
- **Severity.** Blocking. It also fires indirectly when a vendor is un-archived, because that
  operation re-activates every meal of the vendor unconditionally.

### MEAL-006 — An active meal may not belong to an archived vendor

- **Applies to.** Lunch Product, on creation and on every write that touches the archive flag or the
  vendor.
- **Condition.** The meal is active and its vendor is archived.
- **Message.** "The following suppliers are archived. You should either unarchive the suppliers or change the supplier of the product." followed by a line break and then the names of the archived vendors of the offending meals, one per line.
- **Placeholders.** The trailing block is the list of archived vendor names, joined by line breaks.
- **Enforced at.** The constraint that watches the archive flag and the vendor.
- **Severity.** Blocking.

### MEAL-013 — A category needs a name

- **Applies to.** Lunch Product Category.
- **Condition.** The category name is empty.
- **Message.** The platform's own required-field message, naming the field by its label "Product
  Category".
- **Severity.** Structural.

### MEAL-014 — A meal needs a name

- **Applies to.** Lunch Product. The name is required and translatable.
- **Severity.** Structural.

### MEAL-015 — A meal needs a category

- **Applies to.** Lunch Product. The category reference is required.
- **Severity.** Structural.

### MEAL-016 — A meal needs a vendor

- **Applies to.** Lunch Product. The vendor reference is required. It is also what supplies the
  meal's company.
- **Severity.** Structural.

### MEAL-017 — A meal needs a price

- **Applies to.** Lunch Product. The price is required and is held with the accounting decimal
  precision.
- **Severity.** Structural.

### MEAL-018 — A meal's category and favourites must be company-compatible

- **Applies to.** Lunch Product, which has the automatic company check switched on.
- **Condition.** The meal's category, or a user in the meal's favourite list, belongs to a company
  that is neither empty nor equal to the meal's own company.
- **Message.** The platform's own company-incompatibility message, which names the offending record
  and the two companies.
- **Severity.** Blocking.

### MEAL-038 — An extra needs a name, a price and a group number

- **Applies to.** Lunch Topping. The name, the price and the group number are all required; the
  group number defaults to one.
- **Severity.** Structural.

### MEAL-039 — An extra written through a group field takes that group's number

- **Applies to.** Lunch Vendor, on creation and on write.
- **Behaviour.** Values supplied through the second group's list have their group number forced to
  two; values supplied through the third group's list have their group number forced to three. The
  first group relies on the field default of one. A write command that carries no values, such as a
  deletion command, is left alone.
- **Severity.** Automatic correction, never a refusal.

---

## 4. Vendor rules

### MEAL-004 — Only an electronic mail vendor can be sent a message

- **Applies to.** The dispatch operation on Lunch Vendor.
- **Condition.** The operation is invoked on a vendor whose channel is telephone.
- **Message.** "Cannot send an email to this supplier!"
- **Severity.** Blocking. Unreachable through the grouped dispatch control, which routes telephone
  vendors down the path that only marks lines sent.

### MEAL-007 — A vendor needs a Contact

- **Applies to.** Lunch Vendor. The Contact reference is required, and every mirrored field reads
  through it.
- **Severity.** Structural.

### MEAL-008 — A vendor needs a time zone

- **Applies to.** Lunch Vendor. The time zone is required and defaults to the creating user's time
  zone, or to `UTC`, the zero-offset universal time zone, when the creating user has none.
- **Severity.** Structural.

### MEAL-009 — The cut-off hour must lie between zero and twelve

- **Applies to.** Lunch Vendor, as a table constraint named `_automatic_email_time_range`.
- **Condition.** The cut-off hour is below zero or above twelve.
- **Message.** "Automatic Email Sending Time should be between 0 and 12"
- **Severity.** Structural. The constraint is checked before the scheduled action is
  re-synchronised, so a rejected value never produces a wrong next due instant.

### MEAL-010 — The three extra labels and the three disciplines are required

- **Applies to.** Lunch Vendor. Each of the three labels is required and defaults to "Extras",
  "Beverages" and "Extra Label 3" respectively. Each of the three disciplines is required and
  defaults to none-or-more.
- **Severity.** Structural.

### MEAL-040 — A vendor cannot be deleted while meals point at it

- **Applies to.** Lunch Vendor.
- **Condition.** At least one Lunch Product references the vendor, whose reference is required.
- **Message.** The platform's own referential-integrity message, naming the referring records.
- **Severity.** Blocking. Extras are not an obstacle: they are deleted by cascade.

### MEAL-041 — A vendor and a notice each need a scheduled action

- **Applies to.** Lunch Vendor and Lunch Alert. The scheduled action reference is required and read
  only, and it is created automatically before the record itself. Deleting the scheduled action
  deletes the owning record by cascade.
- **Severity.** Structural. The notice's push operation can clear this required reference; see the
  compatibility finding at transition N3 of [`state-machines.md`](state-machines.md).

### MEAL-048 — Changing a vendor's company rewrites its orders

- **Applies to.** Lunch Vendor, on write.
- **Behaviour.** When the company is written, every Lunch Order whose vendor is in the written set
  has its company rewritten to the same value, in one write, whatever state those orders are in and
  however old they are.
- **Severity.** Automatic side effect, never a refusal. **Compatibility finding.** The rewrite is
  unbounded in time and reaches archived and received orders, which changes historical records and
  can move them out of a user's company visibility. A corrected behaviour would restrict the rewrite
  to orders that are not yet received.

### MEAL-049 — Clearing a weekday cancels future orders

- **Applies to.** Lunch Vendor, on write.
- **Condition.** A weekday flag is written with a false value.
- **Behaviour.** Every Lunch Order of that vendor whose state is to-order or ordered, whose date is
  today or later in the vendor's time zone and whose date falls on one of the cleared weekdays, is
  moved to the cancelled state in one write.
- **Severity.** Automatic side effect. Sent and received orders are untouched.

---

## 5. Location and notice rules

### MEAL-019 — A location needs a name

- **Applies to.** Lunch Location. The name is required.
- **Severity.** Structural.

### MEAL-026 — A notice can be pushed only while it is active and in the pushed mode

- **Applies to.** The push operation on Lunch Alert.
- **Condition.** The notice is archived, or its mode is the banner mode, and the operation is
  invoked anyway.
- **Message.** "Cannot send a chat notification in the current state"
- **Severity.** Blocking. Unreachable in normal operation because the scheduled action is
  deactivated in both cases; reachable when the operation is invoked directly.

### MEAL-027 — The notification hour must lie between zero and twelve

- **Applies to.** Lunch Alert, as a table constraint named `_notification_time_range`.
- **Condition.** The notification hour is below zero or above twelve.
- **Message.** "Notification time must be between 0 and 12"
- **Severity.** Structural.

### MEAL-028 — A notice needs a name and a message

- **Applies to.** Lunch Alert. Both are required and both are translatable.
- **Severity.** Structural.

### MEAL-029 — A notice needs at least one location

- **Applies to.** The notice form, which marks the location list as required.
- **Condition.** The location list is empty.
- **Message.** The platform's own required-field message, naming the field by its label "Location".
- **Severity.** Screen. The stored entity permits an empty list, and a notice created without the
  form and without locations is never shown as a banner, because the banner query requires the
  employee's location to be in the list, but **is** pushed to everyone, because the push operation
  narrows by location only when the list is non-empty.

### MEAL-050 — The half-day marker is required on both hour fields

- **Applies to.** Lunch Vendor and Lunch Alert. The marker is required and defaults to the morning
  half on both.
- **Severity.** Structural.

---

## 6. Account rules

### MEAL-023 — A movement needs a date

- **Applies to.** Lunch Cash Move. The date is required and defaults to today in the recording
  user's time zone.
- **Severity.** Structural.

### MEAL-024 — A movement needs an amount

- **Applies to.** Lunch Cash Move. The amount is required. Any sign is accepted: a positive amount
  credits the account, a negative amount charges it.
- **Severity.** Structural.

### MEAL-025 — A movement needs a currency

- **Applies to.** Lunch Cash Move. The currency is required and defaults to the recording user's
  current company currency.
- **Severity.** Structural.

### MEAL-051 — Amounts of different currencies are summed without conversion

- **Applies to.** The internal account balance.
- **Behaviour observed.** The balance adds every statement row of an employee irrespective of the
  row's currency and irrespective of the company behind it. Two movements of one hundred in
  different currencies contribute two hundred.
- **Severity.** Silent. **Compatibility finding.** A corrected behaviour would either convert every
  row into one presentation currency at the row's date, or keep a separate balance per currency and
  test the balance of the currency the order is charged in. The specified behaviour here is the
  observed one, because a rebuild that converts would refuse orders the present system accepts.

### MEAL-052 — The statement is read-only

- **Applies to.** Lunch Cash Move Report.
- **Behaviour.** The entity has no table; it is a database view. Creation, update and deletion are
  impossible, and the access matrix grants read alone.
- **Severity.** Structural.

---

## 7. Permission and visibility rules

### MEAL-030 — Only an administrator may order for someone else

- **Applies to.** All six service endpoints of the domain.
- **Condition.** The request names an employee other than the requesting user, and the requesting
  user does not hold the administration privilege.
- **Message.** "You are trying to impersonate another user, but this can only be done by a lunch manager"
- **Severity.** Blocking, raised as an access refusal.

### MEAL-031 — An employee may write only on their own uncompleted orders

- **Applies to.** Lunch Order, as a record rule that restricts the write operation for every
  internal user.
- **Condition.** The order's state is the received state, or the order belongs to another employee.
- **Message.** The platform's own access-refusal message for a record rule, naming the entity and
  the operation.
- **Severity.** Blocking. A second record rule grants holders of the administration privilege an
  unrestricted write, and because the two rules are attached to different groups they are combined
  disjunctively, so an administrator may write on any order in any state.

### MEAL-032 — An employee may delete only orders that are to-order or cancelled

- **Applies to.** Lunch Order, as a record rule that restricts the delete operation for holders of
  the ordering privilege.
- **Condition.** The order's state is ordered, sent or received.
- **Message.** The platform's own access-refusal message for a record rule.
- **Severity.** Blocking. The rule is attached to the ordering privilege only; the administration
  privilege implies the ordering privilege, so an administrator is bound by the same rule for
  deletion and must cancel a line before deleting it.

### MEAL-033 — An employee sees only their own account movements

- **Applies to.** Lunch Cash Move, as two record rules.
- **Behaviour.** Holders of the ordering privilege see movements whose employee is themselves.
  Holders of the administration privilege see every movement. The two rules are attached to
  different groups and so are combined disjunctively.
- **Severity.** Blocking for the reader who is out of scope; the record simply does not appear.

### MEAL-034 — Company visibility

- **Applies to.** Lunch Vendor, Lunch Order, Lunch Product, Lunch Product Category and Lunch
  Location, each through one global record rule.
- **Condition.** The record's company is neither empty nor among the reader's currently active
  companies.
- **Behaviour.** The record does not appear. Because the rules are global they are combined
  conjunctively with every other rule, so no privilege escapes them.
- **Severity.** Blocking for the reader who is out of scope.
- **Not covered.** Lunch Topping, Lunch Alert and Lunch Cash Move Report carry no company rule.
  **Compatibility finding.** An extra of a vendor belonging to another company is readable, and a
  notice is readable and pushable across companies. A corrected behaviour would add the same rule to
  the extra, keyed on the extra's own company, and would give the notice a company field and a rule.

### MEAL-053 — The ordering privilege is required for the personal fields

- **Applies to.** The two fields this domain adds to the User: the last ordering location and the
  favourite meal list.
- **Behaviour.** Both are marked as belonging to the ordering privilege, so a user who does not hold
  it can neither read nor write them, on their own record or on any other. The service endpoint that
  writes the location does so with elevated rights so that an administrator can set it for the
  employee they are impersonating.
- **Severity.** Structural.

### MEAL-054 — The statement is not protected by a record rule

- **Applies to.** Lunch Cash Move Report.
- **Behaviour observed.** Read access is granted to every internal user with no record rule. The
  personal statement screen adds a filter on the reading user, but the filter is part of the screen,
  not of the entity.
- **Severity.** Silent. **Compatibility finding.** A rebuild should carry the two record rules of
  the Lunch Cash Move onto the report so that the protection of the underlying data is not lost in
  the view.

---

## 8. Concurrency and locking

### MEAL-042 — The state and the price are read-only fields

- **Applies to.** Lunch Order. The state and the total price are declared read only, so no screen
  writes them directly; the state changes only through the named operations, and the price is
  recomputed from the extras, the meal and the quantity.
- **Severity.** Structural.

### MEAL-043 — The date is editable only while the line is to-order

- **Applies to.** Lunch Order, on every screen that shows the date.
- **Condition.** The state is anything other than to-order.
- **Behaviour.** The date field, and the employee field on the order list, become read only.
- **Severity.** Screen. The entity itself permits the write; the record rule of MEAL-031 is what
  stops an employee changing the date of a received line.

### MEAL-044 — The balance check reads committed and pending writes together

- **Applies to.** Every enforcement of MEAL-020.
- **Behaviour.** Before the balance is read, every pending write of the transaction is flushed to
  the database, so the statement view sees the change being validated. Two concurrent transactions
  that each pass the check may therefore both commit and leave the account beyond the permitted
  overdraft; the domain takes no lock on the employee's account.
- **Severity.** Silent. **Compatibility finding.** A rebuild that must guarantee the overdraft limit
  should take a row-level lock on the employee before reading the balance, or re-check the balance
  with a serialisable isolation level. The observed behaviour is optimistic and can be defeated by
  two simultaneous confirmations.

### MEAL-055 — Merging is not concurrency-safe

- **Applies to.** The merge machinery on creation and on write.
- **Behaviour.** The search for a matching line and the increment of its quantity are two separate
  operations with no lock between them. Two simultaneous creations of the same meal by the same
  employee can each find no match and each create a line, leaving two lines that should have been
  one.
- **Severity.** Silent. **Compatibility finding.** A rebuild should lock the candidate set, or
  accept the duplicate and merge it later.

---

## 9. Rule index

| Identifier | Subject | Entity or operation | Severity |
|---|---|---|---|
| MEAL-001 | Vendor must serve on the order date | Confirm-cart operation | Blocking |
| MEAL-002 | Meal must not be archived | Confirm-cart operation | Blocking |
| MEAL-003 | Vendor must serve today for a repeat | Repeat operation | Blocking |
| MEAL-004 | Only an electronic mail vendor can be sent a message | Dispatch operation | Blocking |
| MEAL-005 | Active meal may not sit in an archived category | Lunch Product | Blocking |
| MEAL-006 | Active meal may not belong to an archived vendor | Lunch Product | Blocking |
| MEAL-007 | Vendor needs a Contact | Lunch Vendor | Structural |
| MEAL-008 | Vendor needs a time zone | Lunch Vendor | Structural |
| MEAL-009 | Cut-off hour between zero and twelve | Lunch Vendor | Structural |
| MEAL-010 | Extra labels and disciplines required | Lunch Vendor | Structural |
| MEAL-011 | Group of discipline one-or-more needs at least one extra | Lunch Order | Blocking |
| MEAL-012 | Group of discipline exactly-one needs exactly one extra | Lunch Order | Blocking |
| MEAL-013 | Category needs a name | Lunch Product Category | Structural |
| MEAL-014 | Meal needs a name | Lunch Product | Structural |
| MEAL-015 | Meal needs a category | Lunch Product | Structural |
| MEAL-016 | Meal needs a vendor | Lunch Product | Structural |
| MEAL-017 | Meal needs a price | Lunch Product | Structural |
| MEAL-018 | Meal must be company-compatible | Lunch Product | Blocking |
| MEAL-019 | Location needs a name | Lunch Location | Structural |
| MEAL-020 | Balance may not fall below the permitted overdraft | Lunch Order | Blocking |
| MEAL-021 | No adding after the cut-off | Order dialogue | Screen |
| MEAL-022 | Repeat performs no balance check | Repeat operation | Screen |
| MEAL-023 | Movement needs a date | Lunch Cash Move | Structural |
| MEAL-024 | Movement needs an amount | Lunch Cash Move | Structural |
| MEAL-025 | Movement needs a currency | Lunch Cash Move | Structural |
| MEAL-026 | Notice pushable only when active and in the pushed mode | Push operation | Blocking |
| MEAL-027 | Notification hour between zero and twelve | Lunch Alert | Structural |
| MEAL-028 | Notice needs a name and a message | Lunch Alert | Structural |
| MEAL-029 | Notice needs at least one location | Notice form | Screen |
| MEAL-030 | Only an administrator may order for someone else | Service endpoints | Blocking |
| MEAL-031 | Employee writes only their own uncompleted orders | Lunch Order | Blocking |
| MEAL-032 | Employee deletes only to-order or cancelled orders | Lunch Order | Blocking |
| MEAL-033 | Employee sees only their own movements | Lunch Cash Move | Blocking |
| MEAL-034 | Company visibility on five entities | Five entities | Blocking |
| MEAL-038 | Extra needs a name, a price and a group number | Lunch Topping | Structural |
| MEAL-039 | Extras take the group number of the field they arrive through | Lunch Vendor | Automatic |
| MEAL-040 | Vendor cannot be deleted while meals point at it | Lunch Vendor | Blocking |
| MEAL-041 | Vendor and notice each need a scheduled action | Two entities | Structural |
| MEAL-042 | State and price are read only | Lunch Order | Structural |
| MEAL-043 | Date editable only while to-order | Lunch Order | Screen |
| MEAL-044 | Balance check is optimistic | Balance check | Silent |
| MEAL-045 | Repeated line inherits the delivery-notice flag | Repeat operation | Silent |
| MEAL-046 | Add control hidden before the balance is exhausted | Order dialogue | Screen |
| MEAL-047 | Top-up help text | Service endpoint | Informational |
| MEAL-048 | Vendor company change rewrites its orders | Lunch Vendor | Automatic |
| MEAL-049 | Clearing a weekday cancels future orders | Lunch Vendor | Automatic |
| MEAL-050 | Half-day marker required on both hour fields | Two entities | Structural |
| MEAL-051 | Currencies are summed without conversion | Balance | Silent |
| MEAL-052 | The statement is read only | Lunch Cash Move Report | Structural |
| MEAL-053 | Ordering privilege required for the personal fields | User | Structural |
| MEAL-054 | The statement carries no record rule | Lunch Cash Move Report | Silent |
| MEAL-055 | Merging is not concurrency-safe | Merge machinery | Silent |

## 10. Compatibility findings in one place

| Rule or section | Finding | Corrected behaviour |
|---|---|---|
| MEAL-021 | The cut-off is enforced only by the screen. | Repeat the check inside the confirm-cart operation. |
| MEAL-022 | A repeat can overdraw the account. | Run the balance check after the copy and roll back on failure. |
| MEAL-045 | A repeat inherits the delivery-notice flag. | Reset the flag on the copy. |
| MEAL-034 | Extras and notices carry no company rule. | Add a company rule to the extra and a company field and rule to the notice. |
| MEAL-044 | The balance check takes no lock. | Lock the employee's account before reading the balance. |
| MEAL-048 | A vendor company change rewrites historical orders. | Restrict the rewrite to orders that are not yet received. |
| MEAL-051 | Currencies are summed without conversion. | Convert to one presentation currency, or keep one balance per currency. |
| MEAL-054 | The statement is readable by every internal user. | Carry the two movement record rules onto the report. |
| MEAL-055 | Merging is not concurrency-safe. | Lock the candidate set, or merge duplicates afterwards. |
| [`entities.md`](entities.md#16-archival-and-company-behaviour) | Un-archiving a vendor re-activates meals whose category is archived and then fails. | Re-activate only meals whose category is active. |
| [`entities.md`](entities.md#82-nature-and-composition) | Sent orders leave the account statement. | Include the sent state in the statement. |
| [`entities.md`](entities.md#82-nature-and-composition) | The statement describes a meal in the base language. | Resolve the meal name in the reading user's language. |
| [`state-machines.md`](state-machines.md#18-merge-on-write-and-what-it-does-to-a-state-change) | A merge into a sent or received line loses the quantity. | Exclude sent and received lines from the set of merge targets. |
| [`state-machines.md`](state-machines.md#18-merge-on-write-and-what-it-does-to-a-state-change) | The merge match uses the writing user's location. | Read the location from the line being written. |
| [`state-machines.md`](state-machines.md#54-the-gap-on-the-show-until-date-itself) | A notice is skipped on its final day. | Align the two comparisons on the show-until date. |
| [`state-machines.md`](state-machines.md#53-transition-table) | The push operation clears a required reference. | Leave the scheduled action in place and inactive. |
| [`interfaces.md`](interfaces.md#7-the-vendor-order-message) | The per-location block of the vendor message never renders. | Read the location list from the same payload key it is supplied under. |
