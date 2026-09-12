# Meal Ordering

This domain specifies the complete staff meal ordering capability of the system: the vendors that
deliver or serve meals to a workplace, the catalogue of meals each vendor offers, the priced extras
that may be added to a meal, the delivery locations of the employer, the individual meal orders
placed by employees, the internal prepaid account that each employee spends from, the notices shown
or pushed to employees while they order, and the analysis view that merges account top-ups with meal
charges into a single running statement.

The capability is a self-contained internal service. An employee browses a catalogue of meals that
are available today at their location, configures a meal with extras and a free-form note, adds it
to a cart, and confirms the cart. Confirming charges the meal against the employee's internal
account. A meal ordering administrator groups the day's confirmed orders by vendor, sends them to
the vendor by telephone or by an automatically composed electronic mail message, marks them received
when the delivery arrives, and pushes a delivery notice to the employees concerned. Separately the
administrator records the money an employee hands over, which credits the same internal account.

The domain owns **no journal entries**. The internal account is a private running total held inside
this domain and never posted to the general ledger. What the employer really owes the vendor, and
what the employee really owes the employer, reach the ledger through other domains and are described
in [`accounting-effects.md`](accounting-effects.md).

---

## 1. The questions this domain answers

1. Which vendors serve us, on which weekdays, until which date, in which time zone, and by which
   channel do we place the order with them?
2. What does each vendor offer, at what price, in which category, with which extras, and how many
   extras of each kind may or must be chosen?
3. Where are our people sitting, which vendors serve which of those locations, and which location is
   a given employee currently ordering for?
4. What has an employee ordered today, for how much, in which state of the ordering pipeline, and
   may they still change it?
5. How much money does an employee still have on their internal account, how far into overdraft are
   they allowed to go, and may they add one more meal?
6. When is the daily cut-off for each vendor, and what happens to orders placed after it?
7. What is sent to the vendor, in which layout, from which sender, and containing which lines?
8. What notices does an employee see while ordering, and what is pushed to them by conversation
   message, to whom, and at which hour of which weekday?
9. What did each employee consume and pay over time, and what is the resulting balance?

---

## 2. Capabilities covered

| Capability | Summary |
|---|---|
| Vendor register | A meal vendor built on a Contact record, carrying the postal address, the telephone number, the electronic mail address, the responsible administrator, the ordering channel, the daily cut-off hour with its morning or afternoon marker, the time zone, the weekday availability pattern, an optional final service date, the served locations and a delivery indicator. |
| Vendor scheduling | One scheduled action per vendor, created with the vendor and kept in step with it, which composes and sends the daily order message at the vendor's cut-off hour expressed in the vendor's own time zone. |
| Meal catalogue | Meals grouped in categories, each meal priced, described, illustrated, attached to exactly one vendor, optionally flagged as new until a date, and markable as a personal favourite by each employee. |
| Extras | Up to three independent groups of priced extras per vendor, each group with its own label and its own quantity discipline: none or more, one or more, or exactly one. |
| Locations | Named delivery locations with a free-form address, optionally restricted to one company; every vendor declares which locations it serves, and every employee carries the location they last ordered for. |
| Ordering | A cart-style flow: pick a meal, choose extras, add a note, add to cart, confirm the whole cart at once. Identical lines merge into one line with a higher quantity. |
| Order pipeline | Five states from to-order through ordered, sent and received, with a cancellation state and a reset path back to ordered. |
| Internal account | Two kinds of movement, credits recorded by an administrator and debits derived from orders, merged by an analysis view into one statement per employee, with a configurable permitted overdraft. |
| Spending control | Every operation that raises what an employee owes re-checks the balance and refuses when the permitted overdraft would be exceeded; the client hides the add-to-cart control in advance. |
| Vendor dispatch | A grouped operation per vendor that either composes and sends the order message or simply marks telephone orders as sent, and a matching operation that marks the day's sent orders received. |
| Delivery notice | A per-order operation that pushes a company-configurable delivery message to each ordering employee once, in that employee's language. |
| Notices | Notice records that either appear as a banner on the ordering screen for a chosen set of locations, or are pushed as conversation messages at a chosen hour on chosen weekdays to everyone or to employees who ordered within the last week, month or year. |
| Reporting | A read-only analysis view merging credits and charges, offered as a personal statement and as an administrator's control screen grouped by employee. |
| Client service | Six named service endpoints that feed the ordering screen: current information, empty the cart, confirm the cart, the top-up help text, and read and write the current location. |

---

## 3. Actors

| Actor | Description |
|---|---|
| Meal orderer | Any employee holding the ordering privilege. Browses the catalogue, configures and confirms orders, changes their own location, marks meals as favourites, reads their own account statement, and deletes their own orders while these are still to-order or cancelled. |
| Meal ordering administrator | Holds the administration privilege, which implies the ordering privilege. Maintains vendors, meals, categories, extras, locations and notices; sees and records account movements for everyone; orders on behalf of another employee; sends, receives, cancels, resets and notifies orders. |
| Vendor responsible | The administrator named on a vendor. Their formatted electronic mail address becomes the sender of the automatic order message for that vendor. |
| Vendor | The external party. Receives the daily order message or a telephone call, and delivers. It has no account in the system; it exists as a Contact and as a meal vendor record. |
| Scheduling service | The background worker that runs the per-vendor dispatch action and the per-notice push action at their next due instant. |

---

## 4. Entities the domain owns

| Full name | Transport name | Purpose | Reference page |
|---|---|---|---|
| Lunch Vendor | `lunch.supplier` | A meal vendor with its availability pattern, ordering channel, cut-off and extras configuration. | [`../../references/entities/lunch.supplier.md`](../../references/entities/lunch.supplier.md) |
| Lunch Product | `lunch.product` | One orderable meal offered by one vendor at one price. | [`../../references/entities/lunch.product.md`](../../references/entities/lunch.product.md) |
| Lunch Product Category | `lunch.product.category` | A grouping of meals such as sandwich, pizza, burger or drinks. | [`../../references/entities/lunch.product.category.md`](../../references/entities/lunch.product.category.md) |
| Lunch Topping | `lunch.topping` | A priced extra belonging to one vendor and to one of that vendor's three extra groups. | [`../../references/entities/lunch.topping.md`](../../references/entities/lunch.topping.md) |
| Lunch Location | `lunch.location` | A delivery point of the employer, with a free-form address. | [`../../references/entities/lunch.location.md`](../../references/entities/lunch.location.md) |
| Lunch Order | `lunch.order` | One order line: an employee, a meal, a quantity, extras, a note, a date, a location and a state. | [`../../references/entities/lunch.order.md`](../../references/entities/lunch.order.md) |
| Lunch Cash Move | `lunch.cashmove` | One manually recorded movement on an employee's internal account, normally a credit. | [`../../references/entities/lunch.cashmove.md`](../../references/entities/lunch.cashmove.md) |
| Lunch Cash Move Report | `lunch.cashmove.report` | A read-only analysis view that merges manual movements with order charges into one statement. | [`../../references/entities/lunch.cashmove.report.md`](../../references/entities/lunch.cashmove.report.md) |
| Lunch Alert | `lunch.alert` | A notice shown as a banner on the ordering screen or pushed as a conversation message on a schedule. | [`../../references/entities/lunch.alert.md`](../../references/entities/lunch.alert.md) |

### Entities the domain extends but does not own

| Full name | Transport name | Owning domain | What this domain adds |
|---|---|---|---|
| User | `res.users` | [Identity and Access](../identity-and-access/) | The last ordering location and the personal favourite meal list, both readable only by holders of the ordering privilege and both excluded from record duplication. |
| Company | `res.company` | [Contacts and Organizations](../contacts-and-organizations/) | The permitted overdraft amount and the delivery notice message. |
| Configuration Settings | `res.config.settings` | [Platform foundation](../../overview/package-system.md) | Two settings that write straight through to the two company fields above, plus the company currency for display. |
| Contact | `res.partner` | [Contacts and Organizations](../contacts-and-organizations/) | Nothing is added; a vendor points at a Contact and mirrors its name, address, telephone number and electronic mail address. |
| Scheduled Action | `ir.cron` | [Automation and Integration](../automation-and-integration/) | Nothing is added; one scheduled action is created, kept in step and deleted per vendor and per pushed notice. |

### Entities in the candidate list that this folder does not own

The candidate scope for this folder lists exactly the nine entities above and no platform entity, so
nothing is deferred. The generic entities the domain leans on — the scheduled action, the message
template, the notification layout, the access group, the record rule and the external identifier —
belong to the platform foundation and are specified in
[`../../overview/security-model.md`](../../overview/security-model.md),
[`../../runtime/scheduled-jobs.md`](../../runtime/scheduled-jobs.md) and
[`../../runtime/mail-gateway.md`](../../runtime/mail-gateway.md).

---

## 5. Reading order

1. [`README.md`](README.md) — this file: scope, actors, entity ownership, dependencies.
2. [`entities.md`](entities.md) — every entity with its complete field table, relations, constraints,
   ordering, display-name rule, archival behaviour and company behaviour.
3. [`state-machines.md`](state-machines.md) — the order pipeline, the archive flag machines and the
   vendor availability machine, each with its transition table and diagram.
4. [`workflows.md`](workflows.md) — the sixteen end-to-end procedures, step by step.
5. [`business-rules.md`](business-rules.md) — every validation, constraint, permission check and
   invariant, numbered, with its exact refusal message.
6. [`calculations.md`](calculations.md) — every formula: line price, account balance, permitted
   overdraft, hour conversion, next dispatch instant, availability and cut-off.
7. [`accounting-effects.md`](accounting-effects.md) — why the domain posts nothing, and what it
   causes elsewhere.
8. [`configuration.md`](configuration.md) — settings, privileges, access rights, record rules,
   scheduled actions, default records, server actions and the message template.
9. [`interfaces.md`](interfaces.md) — menus, screens, named operations, service endpoints, the
   ordering screen and the vendor message.
10. [`acceptance-criteria.md`](acceptance-criteria.md) — numbered scenarios with concrete numbers.
11. [`glossary.md`](glossary.md) — every term of the domain.

---

## 6. Files in this folder

| File | Content |
|---|---|
| [`README.md`](README.md) | Scope, questions answered, capabilities, actors, owned and extended entities, reading order, dependencies. |
| [`entities.md`](entities.md) | The nine owned entities and the extension fields, each with a complete field table, relations, uniqueness, ordering, display name, archival and company behaviour. |
| [`state-machines.md`](state-machines.md) | The order pipeline with five states, the three archive machines, and the vendor availability and cut-off machine, with transition tables, guards, refusal messages and diagrams. |
| [`workflows.md`](workflows.md) | Vendor set-up, catalogue set-up, locations, browsing, cart building, cart adjustment, cart confirmation, dispatch by electronic mail, dispatch by telephone, receipt and delivery notice, account crediting, notice pushing, withdrawal of a vendor or a meal, the ordering screen interaction by interaction, period settlement, and an index of symptoms. |
| [`business-rules.md`](business-rules.md) | Numbered rules with the prefix `MEAL`, each with condition, refusal message, enforcement point and severity, plus the rule index. |
| [`calculations.md`](calculations.md) | Line price, cart totals, account balance, permitted overdraft, add-control availability, decimal hour conversion, next dispatch instant, availability on a date and cut-off, each with a worked example. |
| [`accounting-effects.md`](accounting-effects.md) | The reasoned statement that the domain produces no journal entries, the internal account it keeps instead, and the ledger effects it triggers in other domains. |
| [`configuration.md`](configuration.md) | Company settings, privileges and groups, the access-right matrix, the six record rules, the two families of scheduled actions, the three server actions, the default records and the vendor message template. |
| [`interfaces.md`](interfaces.md) | The menu tree, every screen of every entity, the named operations, the six service endpoints, the ordering screen, printable output and import and export. |
| [`acceptance-criteria.md`](acceptance-criteria.md) | Numbered Given, When, Then scenarios covering the ordinary path, every refusal, every transition, rounding edges, several currencies and several companies. |
| [`glossary.md`](glossary.md) | Every term of the domain, defined, with the transport name where one exists. |

---

## 7. Dependencies on other domains

| Domain | Why |
|---|---|
| [Contacts and Organizations](../contacts-and-organizations/) | A vendor is built on a Contact and mirrors its name, postal address, telephone number, electronic mail address and company. The company carries the permitted overdraft and the delivery notice message. |
| [Identity and Access](../identity-and-access/) | Orders, account movements and favourites are keyed on the User. The two privileges of this domain, the access-right matrix and the six record rules are declared here and enforced by the platform mechanism. |
| [Messaging and Activities](../messaging-and-activities/) | The vendor record carries a message thread and scheduled activities. The delivery notice and the pushed notice are posted as notifications through the messaging service. The vendor order message is an electronic mail template rendered and queued by the messaging service. |
| [Automation and Integration](../automation-and-integration/) | One scheduled action per vendor and one per pushed notice; three server actions bound to the order list and card screens. |
| [Multi-currency](../multi-currency/) | Meal prices, extra prices, order totals and account movements are all expressed in a currency; the account statement sums them. |
| [Human Resources Core](../human-resources-core/) | The people who order are employees of the company; the ordering privileges sit under the human resources privilege category. |
| [General Ledger](../general-ledger/) | The counterpart of what this domain records internally: the vendor's bill and the employee's repayment are posted there, never here. |
| [Accounts Payable](../accounts-payable/) | Where the vendor's invoice for the delivered meals is recorded, if the employer chooses to record one. |
| [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/) | Where the cash an employee hands over is banked, if the employer chooses to bank it separately from petty cash. |

Platform documents this folder relies on:
[`../../overview/security-model.md`](../../overview/security-model.md),
[`../../overview/multi-company.md`](../../overview/multi-company.md),
[`../../overview/views-and-actions.md`](../../overview/views-and-actions.md),
[`../../overview/entity-and-field-system.md`](../../overview/entity-and-field-system.md),
[`../../runtime/scheduled-jobs.md`](../../runtime/scheduled-jobs.md),
[`../../runtime/mail-gateway.md`](../../runtime/mail-gateway.md),
[`../../runtime/notification-bus.md`](../../runtime/notification-bus.md),
[`../../runtime/translation.md`](../../runtime/translation.md),
[`../../data/domain-model.md`](../../data/domain-model.md),
[`../../interfaces/endpoint-catalog.md`](../../interfaces/endpoint-catalog.md),
[`../../interfaces/service-layer.md`](../../interfaces/service-layer.md).
