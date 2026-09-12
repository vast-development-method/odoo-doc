# Glossary

Terms used in the Replenishment and Procurement domain, written in full words. A term that names an entity is also listed in `entities.md`; a term that names a rule is cited by its number from `business-rules.md`.

---

**Arrival handling.** The second half of route execution: when goods arrive at a location, the push rule of that location decides where they go next. See "Push rule" and `workflows.md`, section 7.

**Automatic trigger.** The value `auto` of a Reordering Rule's trigger. An automatically triggered rule is evaluated and executed by the scheduler and by the event-driven trigger; it may never be snoozed and may never carry a manual quantity override.

**Available quantity.** In the split of the supply method "take from stock, if unavailable, trigger another rule": the free quantity of the product at the source location, reduced by what earlier moves of the same batch have already claimed, floored at zero.

**Buy rule.** A Stock Rule whose action is `buy`. When a need appears at its destination location, it creates or extends a draft purchase order line instead of a stock move. It has no source location; the source of the eventual receipt is the vendor location of the vendor chosen at run time.

**Chain (of moves).** A sequence of stock moves linked through the destination and origin relations, each step of which brings the goods closer to the location where they are needed. A chain is built backwards by make-to-order supply methods, and forwards by push rules.

**Cumulative lead time.** See "Lead days".

**Deadline (of a move).** The latest moment by which the move must be completed to keep a downstream promise. It is distinct from the scheduled date and is what lateness is measured against. Writing a new deadline shifts the whole chain by the same number of days.

**Deadline date (of a reordering rule).** The last date on which an order must be placed to avoid the stock falling below the minimum quantity. Empty when no dip is foreseen inside the replenishment horizon. Formula in `calculations.md`, section 11.

**Delay alert date.** The greatest scheduled date among a move's not-yet-completed origin moves, when that date is later than the move's own scheduled date. A move with a non-empty delay alert date is shown as late.

**Demand graph.** The saw-tooth picture shown in the Replenishment Information wizard, which projects how often the rule would order at the historic rate of demand. Formulas in `calculations.md`, section 20.

**Destination location (of a rule).** For a pull, buy or manufacture rule, the location where the need must appear for the rule to be selected. For a push rule, the location the goods are sent to.

**Drop ship to a subcontractor.** The variant in which purchased components go straight from the vendor to the subcontractor's location, and, in the reverse direction, from the subcontracting location to the customer.

**Drop shipment.** A single stock move that takes goods from a vendor location straight to a customer location, so that the goods never enter a location of the company while ownership still passes through it. Recognized by `RP-RULE-250`.

**Effective date (of a purchase order).** The earliest completion date among the order's completed transfers whose destination location is not a vendor location. A return to the vendor never moves it. Rule `RP-RULE-350`.

**Effective days to arrival.** The number of days between a purchase order's order date and the day the goods actually arrived, or, while nothing has been received, until the line's planned date. Formula in `calculations.md`, section 30.

**Effective route, effective vendor, effective bill of materials.** The value a Reordering Rule would actually use: the value set explicitly on the rule when there is one, and otherwise the value the system would derive. Formulas in `calculations.md`, section 25.

**Event-driven trigger.** The mechanism that runs automatic reordering rules immediately when a transfer is confirmed, without waiting for the daily scheduled run. Disabled by the stored parameter `inventory.disable_automatic_scheduler`.

**Exchange.** The operation that creates a return and, for a receipt, immediately the return of that return, so that the goods come back in. The exchange moves are detached from their originating moves, which stops them from counting as purchase returns. Rules `RP-RULE-339` and `RP-RULE-340`.

**Fallback multiple.** The replenishment multiple that is derived when the Reordering Rule leaves it empty: the vendor's unit for a buying chain, the bill of materials' unit for a manufacturing chain. It is shown as a grey placeholder and is applied when the quantity to order is rounded.

**Forecast date.** See "Lead horizon date".

**Forecast quantity.** The quantity of a product that is expected at a location at a given moment: the on-hand quantity plus the incoming moves minus the outgoing moves that are not yet completed and that are scheduled up to that moment.

**Free quantity.** The on-hand quantity of a product at a location that is not reserved by any move.

**Grouping mode (of a vendor).** The setting that decides which replenishment needs for that vendor are collected into one request for quotation: `On Order`, `Daily`, `Weekly` or `Always`. See `RP-RULE-097` to `RP-RULE-102`.

**In transit (on the forecast report).** A quantity that belongs to the warehouse but is not in the warehouse stock location, and therefore cannot be reserved for a delivery from that stock location.

**Inter-warehouse resupply route.** A route generated automatically when one warehouse is declared as the supplier of another. It moves goods from the supplying warehouse's output location, through a transit location, into the supplied warehouse's stock location.

**Lead days.** The cumulative number of calendar days contributed by the rules of a chain: the rule lead times, the vendor lead time, the days to purchase, the manufacturing lead time and the days to supply components. It does not include the replenishment horizon.

**Lead horizon date.** Today plus the lead days plus the replenishment horizon. This is the moment at which a Reordering Rule reads the forecast.

**Make to order.** The supply method `make_to_order`, labelled "Trigger Another Rule". The stock available at the source location is ignored; a new need is created there and rule selection runs again for it. The created document is bound to the move that asked for it.

**Make to stock.** The supply method `make_to_stock`, labelled "Take From Stock". The goods are taken from the stock available at the source location; no supply need is created.

**Manual trigger.** The value `manual` of a Reordering Rule's trigger. A manually triggered rule appears on the replenishment report and is executed only when a user presses Order. Only a manual rule may be snoozed.

**Manufacture rule.** A Stock Rule whose action is `manufacture`. When a need appears at its destination location, it creates or extends a manufacturing order.

**Minimum stock rule.** A synonym of Reordering Rule.

**Multiple.** See "Replenishment multiple".

**Need.** See "Procurement request".

**Operation type.** The document template that a rule stamps on the documents it creates: it carries the sequence, the default source and destination locations, the reservation method and the code (`incoming`, `outgoing`, `internal`, `manufacturing`, `dropship`).

**Order date (of a purchase order).** The moment the order should be placed with the vendor, labelled "Order Deadline". It is the expected arrival minus the vendor lead time. The days to purchase does not move it.

**Order to Max.** The manual operation that forces a Reordering Rule's quantity to order to the multiple-rounded difference between the maximum quantity and the forecast quantity, and then orders.

**Origin (of a document).** The free text that names the document that caused this one. For a document created by a reordering rule it is the rule's reference, optionally followed by the names of the originating stock references.

**Pick before manufacturing.** The manufacturing step configuration in which components are first picked from stock into a pre-production location and only then consumed. It owns the global rule slot `pick_before_manufacturing_make_to_order_pull`, which this domain lists only because a warehouse maintains it alongside its own slots.

**Procurement request.** The unit of work of this domain: a need for a quantity of a product at a location on a date, together with a value map that carries the routes, the dates, the downstream moves and everything else the rules need. It is never stored; its values are serialized onto the moves it creates. Structure in `entities.md`, section 10.

**Pull rule.** A Stock Rule whose action is `pull` or `pull_push`, used in the pull direction: when a need appears at its destination location, it creates a stock move from its source location.

**Purchase analysis view.** The read-only analytical view over purchase order lines to which this domain adds the warehouse of the operation type, the effective date and the effective days to arrival. Specified in `entities.md`, section 27.

**Purchase return.** A stock move that sends goods back towards the vendor. A move counts as one when its destination location usage is `supplier`, or when it has an originating returned move and either its destination is the shared inter-company transit location or the originating returned move's source location usage is `supplier`. A purchase return subtracts from the received quantity of its purchase order line unless the "Update Quantities on Purchase Order" flag was unticked. Rules `RP-RULE-332` to `RP-RULE-336`.

**Push rule.** A Stock Rule whose action is `push` or `pull_push`, used in the push direction: when goods arrive at its source location, it sends them onward to its destination location, either by creating a new move ("Manual Operation") or by rewriting the destination of the existing move ("Automatic No Step Added").

**Quantity in progress.** Quantities that are already committed to a location but that the stock forecast cannot see, because no stock move exists for them yet: draft, sent and to-approve purchase order lines, and draft manufacturing orders. Formula in `calculations.md`, section 5.

**Quantity to order.** What a Reordering Rule proposes to order: the manual override when one is set, otherwise the computed quantity. Formula in `calculations.md`, section 7.

**Reordering Rule.** The record stating that a product at a location must never be forecast below a minimum quantity, and that the system must order enough to reach a maximum quantity when it is.

**Replenishment horizon.** The company setting that makes reordering rules look further ahead than the pure lead time when they read the forecast. A horizon of zero gives strict just-in-time behavior.

**Replenishment location.** A location whose "replenish location" flag is true. The replenishment report watches those locations and creates a temporary reordering rule for every negative forecast it finds in one.

**Replenishment multiple.** A unit of measure to a whole number of which the quantity to order is rounded up. When it is empty, the fallback multiple may still apply.

**Replenishment report.** The screen that lists every reordering rule together with the temporary rules it creates on the fly for the shortages it detects, and that lets a user adjust the quantities and order.

**Request for quotation.** A purchase order that has not yet been confirmed. It is the document a `buy` rule creates or extends.

**Return to the vendor.** The workflow that turns a completed receipt into a transfer in the opposite direction, ending at the vendor location. Specified in `workflows.md`, section 25.

**Route.** An ordered, named collection of Stock Rules that can be selected on a product, a product category, a warehouse, a package type, a sales order line or a shipping method.

**Rule chain.** The ordered set of rules a need would travel through. It is built by repeating one step until it stops: select the rule for the product at the current location; when no rule is found, stop; when the found rule takes from stock or is not a pull rule, add it to the chain and stop; otherwise add it to the chain, move the current location to that rule's source location and repeat. It is the input of every lead time computation.

**Rule selection.** The algorithm that returns the single applicable rule for a product at a location: it walks up the location hierarchy and, at each level, tries the route sources in a fixed precedence. Specified in `workflows.md`, section 1.

**Scheduled date (of a move).** The moment the operation is planned to happen. It is what the warehouse plans against, as opposed to the deadline, which is what lateness is measured against.

**Scheduler.** The daily automated run that recomputes and executes every automatic reordering rule, reserves every waiting move whose reservation date has been reached, and merges duplicated stock quantity records.

**Security lead time, sales.** The company setting that schedules a procurement created from a sales order line a number of calendar days earlier than the promised delivery date, leaving the promised date as the deadline.

**Snooze.** Hiding a manual reordering rule from the replenishment report until a chosen date. An automatic rule may never be snoozed.

**Source location (of a rule).** For a pull rule, the location the goods are taken from. For a push rule, the location where goods arrive. A buy rule has none.

**Stock Rule.** One step of a route: it says what to do when a need appears at a location (pull, buy, manufacture) or what to do when goods arrive at a location (push).

**Store after manufacturing.** The manufacturing step configuration in which finished goods are first placed in a post-production location and only then stored. It owns the global rule slot `store_after_manufacturing_rule`, listed here for the same reason as the previous entry.

**Supply method.** The field of a Stock Rule that says where the goods come from: take from stock, trigger another rule, or take from stock and trigger another rule for the missing quantity only.

**Take from stock, if unavailable, trigger another rule.** The supply method `make_to_stock_else_make_to_order`. The move itself takes from stock and keeps its full demand; only the missing quantity becomes a new, unlinked need.

**Temporary reordering rule.** A rule created by the replenishment report under the superuser account with the trigger `manual`, a minimum of zero, a maximum of zero and the name `Replenishment Report`. It is deleted as soon as the shortage it describes is covered.

**Transit location.** A location with the usage `transit`, used as the hand-over point between two warehouses. Each company has an internal transit location; a shared inter-company transit location exists for transfers between companies.

**Unwanted replenish.** The flag that warns that ordering the proposed quantity would push the forecast above the maximum quantity, which happens whenever a replenishment multiple forces a larger order.

**Update quantities on purchase order.** The flag of a return line that decides whether the returned quantity changes the received quantity of the purchase order line. Ticked by default. Rule `RP-RULE-336`.

**Vendor Delay Report.** The read-only view that measures vendor punctuality, one row per purchase order line, whose on-time delivery rate aggregates as a weighted average rather than as a plain sum.

**Vendor lead time.** The number of calendar days a vendor needs between the confirmation of a purchase order and the arrival of the goods. It is carried by the Vendor Price and is the only purchase-side buffer that moves the dates of a purchase order.

**Vendor Price.** A line of a product's purchase price list: a contact, a minimum quantity, a unit, a price, a currency, validity dates and a lead time. Vendor selection chooses one of them for each buy request.

**Vendor selection.** The algorithm that chooses the Vendor Price a buy request will use. Specified in `calculations.md`, section 14.

**Warning activity.** The scheduled activity created on a product template when a reordering rule fails during the scheduler, carrying the failure message as its note and assigned to the product's responsible person.
