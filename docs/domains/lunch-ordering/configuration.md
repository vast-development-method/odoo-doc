# Meal Ordering — Configuration

Everything an administrator or an installer sets up: the company settings, the privileges and
groups, the access-right matrix, the record rules, the two families of scheduled actions, the server
actions, the default records installed with the capability, the message template and the
translatable strings.

---

## 1. The capability package

| Property | Value |
|---|---|
| Package name in words | Meal Ordering |
| Reproduced package key | `lunch` |
| Category shown in the package list | Human Resources, sub-category Meal Ordering |
| Summary | Handles the meal orders of the employer's staff. |
| Prerequisite capability | The messaging capability, which supplies the message thread, the activity list, the notification service and the electronic mail template engine. |
| Is a top-level application | Yes; it installs its own top-level menu. |
| Menu sequence | 235, which places the menu among the workforce applications. |
| Package sequence in the installer list | 300. |

The package carries a description that explains the arrangement: employers order sandwiches, pizzas
and similar from regular vendors for their staff; managing that becomes administratively heavy once
the number of employees or of vendors grows; the capability makes the management easier and gives
employees tools and a record of their own; and it removes the need for employees to carry coins.

---

## 2. Company settings

Both settings live on the Company and are surfaced on the configuration screen under a section
headed "Lunch". Both are marked as depending on the company, so each company keeps its own value.

| Setting on the screen | Label on the screen | Company field | Type | Default | Meaning |
|---|---|---|---|---|---|
| Overdraft | "Maximum Allowed Overdraft", with the help text "Maximum overdraft that your employees can reach" and the inner label "Overdraft" | `lunch_minimum_threshold` | decimal, displayed as an amount in the company currency | 0 | How far below zero an employee's internal account may go before ordering is refused. It is added to the statement sum before the sign is tested. A value of 25 lets an employee owe 25. |
| Reception notification | "Lunch notification message", inside a setting headed "Reception notification" with the help text "Send this message to your users when their order has been delivered." | `lunch_notify_message` | rich text, translatable | The two lines "Your lunch has been delivered." and "Enjoy your meal!" | The body pushed to an employee when an administrator sends the delivery notice for one of that employee's received orders, rendered in the employee's own language. |

The configuration screen also reads the company currency, purely so that the overdraft amount can be
formatted. The screen is reachable from the menu entry "Settings" under the configuration menu, and
opens the platform's settings form pre-scoped to this capability with picture size reduction turned
off so that the rich-text editor receives the full message.

There are no system parameters, no sequences and no decimal precision records of the domain's own.
The meal price borrows the accounting decimal precision, which belongs to
[`../general-ledger/configuration.md`](../general-ledger/configuration.md).

---

## 3. The privilege category

| Record | Value |
|---|---|
| Privilege name | "Lunch" |
| Sequence within its category | 16 |
| Parent category | Human Resources |

The privilege groups the two access groups below into one selector on the user form, so that a user
is given no meal ordering access, the ordering access, or the administration access.

---

## 4. Access groups

| Reproduced key | Name shown | Sequence | Implies | Members installed with the package | Comment shown on the group |
|---|---|---|---|---|---|
| `group_lunch_user` | "User : Order your meal" | 10 | nothing | none | none |
| `group_lunch_manager` | "Administrator" | 20 | the ordering group | the platform's root user and the installation's administrator user | "Be able to create new products, cashmoves and to confirm or cancel orders." |

Because the administration group implies the ordering group, every administrator is also an orderer
and is bound by every rule attached to the ordering group, including the deletion rule on orders.

The vendor form restricts the responsible selector to users who hold the administration group, and
additionally excludes shared users.

---

## 5. Access rights

One row per entity and group. A tick means the operation is permitted at the entity level; record
rules then narrow it per record.

| Entity | Group | Read | Create | Update | Delete |
|---|---|---|---|---|---|
| Lunch Cash Move | ordering group | yes | no | no | no |
| Lunch Cash Move | administration group | yes | yes | yes | yes |
| Lunch Order | ordering group | yes | yes | yes | yes |
| Lunch Order | administration group | yes | yes | yes | yes |
| Lunch Product | ordering group | yes | no | no | no |
| Lunch Product | administration group | yes | yes | yes | yes |
| Lunch Product Category | ordering group | yes | no | no | no |
| Lunch Product Category | administration group | yes | yes | yes | yes |
| Lunch Alert | every internal user | yes | no | no | no |
| Lunch Alert | administration group | yes | yes | yes | yes |
| Lunch Cash Move Report | every internal user | yes | no | no | no |
| Lunch Location | ordering group | yes | no | yes | no |
| Lunch Location | administration group | yes | yes | yes | yes |
| Lunch Topping | ordering group | yes | no | no | no |
| Lunch Topping | administration group | yes | yes | yes | yes |
| Lunch Vendor | ordering group | yes | no | no | no |
| Lunch Vendor | administration group | yes | yes | yes | yes |

Three rows deserve a note.

- **Lunch Location, ordering group, update without create.** An orderer may rename a location or
  change its address but may not add or remove one. This is unusual and is the observed behaviour.
  **Compatibility finding.** The update right appears to exist so that the location selector on the
  ordering screen can write, but that write is on the User, not on the location, and is performed
  with elevated rights in any case. A corrected behaviour would grant read alone.
- **Lunch Alert and Lunch Cash Move Report, every internal user.** Read is granted to the whole
  internal population, not only to orderers, so that the ordering screen can show banners and
  balances before an employee has been given the ordering privilege.
- **Lunch Order, ordering group, full rights.** Every operation is granted at the entity level and
  the narrowing is done entirely by the two record rules below.

---

## 6. Record rules

Rules that name no group are **global** and are combined conjunctively with every other rule. Rules
that name a group are combined disjunctively among themselves for a reader who holds several of
those groups.

| Reproduced key | Name | Entity | Applies to | Groups | Condition |
|---|---|---|---|---|---|
| `lunch_mind_your_own_food_money` | "lunch.cashmove: do not see other people's cashmove" | Lunch Cash Move | read, create, update, delete | ordering group | The movement's employee is the reader. |
| `lunch_mind_other_food_money` | "lunch.cashmove: do see other people's cashmove" | Lunch Cash Move | read, create, update, delete | administration group | Always true. |
| `lunch_order_rule_delete` | "lunch.order: Only new and cancelled order lines deleted." | Lunch Order | delete only | ordering group | The order's state is the to-order state or the cancelled state. |
| `lunch_order_rule_write` | "lunch.order: Don't change confirmed order" | Lunch Order | update only | every internal user | The order's state is not the received state **and** the order's employee is the reader. |
| `lunch_order_rule_write_manager` | "manager can do whatever" | Lunch Order | update only | administration group | Always true. |
| `ir_rule_lunch_supplier_multi_company` | "Lunch supplier: Multi Company" | Lunch Vendor | all four operations | none, therefore global | The vendor's company is among the reader's active companies, or is empty. |
| `ir_rule_lunch_order_multi_company` | "Lunch order: Multi Company" | Lunch Order | all four operations | none, therefore global | The order's company is among the reader's active companies, or is empty. |
| `ir_rule_lunch_product_multi_company` | "Lunch product: Multi Company" | Lunch Product | all four operations | none, therefore global | The meal's company is among the reader's active companies, or is empty. |
| `ir_rule_lunch_product_category_multi_company` | "Lunch product category: Multi Company" | Lunch Product Category | all four operations | none, therefore global | The category's company is among the reader's active companies, or is empty. |
| `ir_rule_lunch_location_multi_company` | "Lunch location: Multi Company" | Lunch Location | all four operations | none, therefore global | The location's company is among the reader's active companies, or is empty. |

Every rule above is installed as non-updatable, so an installation that edits a rule keeps its edits
across package updates. The two access groups and the privilege category, by contrast, are
updatable, so a package update restores their names, sequences and implications.

No record rule exists on the Lunch Topping, the Lunch Alert or the Lunch Cash Move Report — see rule
[MEAL-034](business-rules.md#meal-034--company-visibility) and rule
[MEAL-054](business-rules.md#meal-054--the-statement-is-not-protected-by-a-record-rule).

---

## 7. Scheduled actions

The domain declares no fixed scheduled action. It creates one per vendor and one per notice, at the
moment the vendor or the notice is created, and keeps each in step with its owner.

### 7.1 The per-vendor dispatch action

| Property | Value at creation | Value after synchronisation |
|---|---|---|
| Name | "Lunch: send automatic email" | The fixed prefix "Lunch: send automatic email to " followed by the vendor's name |
| Owner | The platform's root user | unchanged |
| Active | false | true when the vendor is active **and** its channel is electronic mail; false otherwise |
| Interval | once every one day | unchanged |
| Entity it acts on | Lunch Vendor | unchanged |
| Kind | a coded action | unchanged |
| Body | empty | a three-line body: two comment lines stating that the action is controlled by the Lunch Vendor entity and must not be edited directly, and one line that invokes the dispatch operation on that one vendor |
| Next due instant | unset | computed by the rule in [`calculations.md`](calculations.md#7-next-dispatch-instant) |

Creating a vendor also creates one external identifier for the server action behind the scheduled
action, named with the fixed prefix `lunch_supplier_cron_sa_` followed by that server action's
record identifier, declared in the package `lunch` and marked as not to be overwritten on package
update. Deleting the vendor deletes the scheduled action and the server action; deleting the
scheduled action deletes the vendor by cascade.

The action is re-synchronised whenever any of these vendor fields is written: the name, the archive
flag, the channel, the cut-off hour, the half-day marker and the time zone. When the cut-off hour is
among them it is written to the table first, so that its range constraint is checked before the next
due instant is computed.

### 7.2 The per-notice push action

| Property | Value at creation | Value after synchronisation |
|---|---|---|
| Name | "Lunch: alert chat notification" | The fixed prefix "Lunch: alert chat notification (" followed by the notice's name and a closing parenthesis |
| Owner | The platform's root user | unchanged |
| Active | false | true when the notice is active **and** its mode is the pushed mode **and** its show-until date is empty or today is on or before it |
| Interval | once every one day | unchanged |
| Entity it acts on | Lunch Alert | unchanged |
| Kind | a coded action | unchanged |
| Body | empty | a three-line body: two comment lines stating that the action is controlled by the Lunch Alert entity and must not be edited directly, and one line that invokes the push operation on that one notice |
| Next due instant | unset | computed by the same rule as the vendor's |

Creating a notice also creates one external identifier for the server action behind the scheduled
action, named with the fixed prefix `lunch_alert_cron_sa_` followed by that server action's record
identifier, declared in the package `lunch` and marked as not to be overwritten on package update.

The action is re-synchronised whenever any of these notice fields is written: the name, the archive
flag, the display mode, the show-until date, the notification hour, the half-day marker and the time
zone.

### 7.3 Consequences for a rebuild

A rebuild that uses one shared scheduled job instead of one action per record must reproduce three
observable properties: the dispatch happens at the vendor's own cut-off instant expressed in the
vendor's own time zone; a change to the hour takes effect immediately, moving the next run rather
than waiting for the current one; and an action that has already run today is pushed to tomorrow
rather than run twice.

---

## 8. Server actions

Three server actions are installed and bound to the order list and card screens, so that they appear
in the action menu when lines are selected.

| Name | Bound to | Screens | Effect |
|---|---|---|---|
| "Lunch: Receive meals" | Lunch Order | list and cards | Invokes the receipt operation on every selected line, moving each to the received state. |
| "Lunch: Cancel meals" | Lunch Order | list and cards | Invokes the cancel operation on every selected line, moving each to the cancelled state. |
| "Lunch: Send notifications" | Lunch Order | list and cards | Invokes the delivery notice operation on every selected line. |

All three are installed as updatable, so a package update restores them if they were edited.

---

## 9. Default records installed with the capability

These records are installed once and are never overwritten by a later package update. Each is also
marked so that an installation which deleted it does not have it restored.

| Reproduced key | Entity | Values |
|---|---|---|
| `lunch_location_main` | Lunch Location | Name "HQ Office", no address, no company. |
| `categ_sandwich` | Lunch Product Category | Name "Sandwich", no picture beyond the packaged default. |
| `categ_pizza` | Lunch Product Category | Name "Pizza", with a packaged pizza picture. |
| `categ_burger` | Lunch Product Category | Name "Burger", with a packaged burger picture. |
| `categ_drinks` | Lunch Product Category | Name "Drinks", with a packaged drink picture. |
| `partner_hungry_dog` | Contact | Name "Lunch Supplier". |
| `supplier_hungry_dog` | Lunch Vendor | Points at the Contact above; serves the location "HQ Office"; every other value takes its field default, so the vendor serves Monday to Friday, orders by telephone, has a cut-off hour of 12 in the morning half and three extra groups labelled "Extras", "Beverages" and "Extra Label 3" with the none-or-more discipline. |

A demonstration data set, installed only when demonstration data is requested, adds an address to
the default location, renames the default Contact and gives it a full postal address and an
electronic mail address, and adds a catalogue of priced meals across the four categories. The
demonstration data is not part of the specified behaviour; a rebuild need not reproduce it.

---

## 10. The vendor order message template

| Property | Value |
|---|---|
| Reproduced key | `lunch_order_mail_supplier` |
| Name | "Lunch: Supplier Order" |
| Entity it renders for | Lunch Vendor |
| Description | "Sent to vendor with the order of the day" |
| Sender | The sender address supplied in the rendering payload, which is the responsible administrator's formatted electronic mail address |
| Recipient | The Contact identifier supplied in the rendering payload, which is the vendor's Contact |
| Use the entity's own default recipient | No |
| Subject | The fixed text "Orders for " followed by the company name supplied in the payload |
| Language | The language supplied in the rendering context as the default language |

The body is described in [`interfaces.md`](interfaces.md#7-the-vendor-order-message). It is
installed as updatable, so a package update restores the packaged body over an installation's edits;
an installation that must keep its own wording should duplicate the template rather than edit it.

---

## 11. Fixed fragments

| Reproduced key | Name | Content |
|---|---|---|
| `lunch_payment_dialog` | "Lunch Payment Dialog" | The single sentence "To add some money to your wallet, please contact your lunch manager." |

The fragment is returned by the service endpoint at the route `/lunch/payment_message` and is the
only thing the system says about topping up an account.

---

## 12. Translatable strings

| Where | What is translatable |
|---|---|
| Lunch Product | The meal name and the description. |
| Lunch Product Category | The category name. |
| Lunch Alert | The notice name and the message. |
| Company | The delivery notice message. |
| Every label, message and control text listed in this folder | Through the platform's ordinary translation mechanism, described in [`../../runtime/translation.md`](../../runtime/translation.md). |

Two renderings deliberately switch language:

- The delivery notice is rendered in **the receiving employee's** language, with the subject and the
  company message both resolved in that language, and the resolution is cached per pair of company
  and language so that a hundred employees sharing a language are rendered once.
- The account statement's description of an order charge is **not** switched: it takes the meal name
  in the base language of the installation. See the compatibility finding in
  [`entities.md`](entities.md#82-nature-and-composition).

---

## 13. What an installation must decide

| Decision | Where it is made | Consequence |
|---|---|---|
| How far employees may overdraw | The company setting `lunch_minimum_threshold` | Sets the floor of the balance check. A value of zero means employees must always pay in advance. |
| What the delivery notice says | The company setting `lunch_notify_message` | The body pushed on receipt. |
| Which locations exist | Lunch Location records | Without at least one, the catalogue shows nothing and says "No location found". |
| Which vendors serve which locations | The vendor's served-location list | An empty list means the vendor serves every location. |
| Whether a vendor is ordered by telephone or by message | The vendor's channel | The channel decides whether a scheduled action runs and whether the cut-off is enforced at all. |
| What time the cut-off is, and in which time zone | The vendor's cut-off hour, half-day marker and time zone | Decides the dispatch instant and the point after which the dialogue refuses to add. |
| How many extras of each group may be chosen | The three disciplines on the vendor | Decides which of the two extras refusals can fire. |
| Whether notices appear as banners or arrive as messages | The notice's display mode | Decides whether a scheduled action is created and run. |
