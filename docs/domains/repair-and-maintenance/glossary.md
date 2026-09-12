# Glossary

The vocabulary of the repair and maintenance domain, in full words. Terms are grouped by the half of the domain they belong to; terms used by both are in the third group. A term written in code font is a reproduced identifier, and its full name in words is given with it.

---

## Repair vocabulary

**Repair Order.** One repair job, from the moment a broken item is taken in to the moment the work is declared finished. It names the item, its owner, where the work happens, which parts are consumed, removed or recovered, and how the work is billed. Transport name `repair.order`.

**Product to repair.** The item being repaired, named as a product. It must be goods, never a service. The Repair Order carries at most one, and a Repair Order with none is legal.

**Part.** One product consumed, discarded or recovered during the repair. Every part is an inventory movement attached to the Repair Order and classified by a part kind.

**Part kind.** The classification of a part, with three values. **Add** brings the part into the repaired item. **Remove** takes a part out of the repaired item and writes it off. **Recycle** takes a part out of the repaired item and returns it to stock. The part kind alone decides the two locations of the part's movement. Stored values `add`, `remove` and `recycle`.

**Added part.** A part of kind *add*. It leaves the component source location for the added-parts destination location. It is the only kind of part that is ever billed.

**Removed part.** A part of kind *remove*. It leaves the added-parts destination location for the removed-parts destination location, which is normally an inventory-loss location.

**Recycled part.** A part of kind *recycle*. It leaves the added-parts destination location for the recycled-parts destination location, which is normally the warehouse stock location.

**Component source location.** Where added parts are taken from. Defaults to the warehouse stock location. Editable on the Repair Order.

**Added-parts destination location.** Where added parts go, and where removed and recycled parts come from. Always mirrors the operation type's default destination location, which for a repair type is the company's production location. Read-only on the Repair Order.

**Removed-parts destination location.** Where removed parts are written off. Always mirrors the operation type's default remove destination location, which is normally the company's inventory-loss location. Read-only on the Repair Order.

**Recycled-parts destination location.** Where recycled parts are returned. Defaults to the warehouse stock location. Editable on the Repair Order.

**Product source location.** Where the item being repaired currently sits. Defaults to the warehouse stock location. Editable on the Repair Order. It is the location searched by the availability check at confirmation.

**Product destination location.** Where the repaired item is placed once the work is finished. Defaults to the warehouse stock location. Editable on the Repair Order.

**Production location.** A virtual location of production usage. It represents the inside of a product being built or repaired. Stock sitting there is not counted in the company's inventory valuation, which is why moving a part into it is treated as a consumption.

**Inventory-loss location.** A virtual location of inventory-loss usage. It absorbs quantities written off. Stock sitting there is not counted in the company's inventory valuation.

**Operation type.** The configuration record that governs a document's numbering, its default locations and its lot policy. A **repair operation type** is one whose code is the stored value `repair_operation`; it carries four extra location defaults that ordinary operation types do not have, and four dashboard counters over the repairs it governs.

**Repaired-product movement.** The inventory movement created when a repair is declared finished, carrying the repaired item itself from the product source location to the product destination location. It belongs to the Repair Order but carries no part kind, so it never appears in the Parts list, and it belongs to no transfer, so it never appears on the return transfer. Its detail line records every part detail line as a consumed line, which is what puts the repair into the traceability of the item's serial number.

**Consumed line.** A link from one detail line to another, recording that the goods of the second were used in producing the goods of the first. The repaired-product movement's detail line consumes every part detail line of the repair.

**Demanded quantity.** How much of a part the repair plans to use.

**Recorded quantity.** How much of a part the repair really used. Entered while the repair is under way.

**Incomplete part.** A part whose recorded quantity is strictly lower than its demanded quantity, at the rounding of its unit of measure. A repair holding at least one causes the completion prompt.

**Picked.** A marker on a movement saying that the goods were physically handled. Ending a repair marks every part picked when none was marked, and leaves the marks alone when at least one was already set — which then cancels the unmarked ones.

**Component status.** The human-readable readiness statement of a Repair Order's added parts, one of "Available", "Not Available", or "Exp " followed by a date. Its machine-readable counterpart, the component status code, takes the stored values `available`, `expected` and `late`.

**Readiness boolean and lateness boolean.** The two stored mirrors of the component status code, true exactly when the code is `available` and `late` respectively. They exist so that the operation-type dashboard can count without recomputing availability.

**Under warranty.** A flag on a Repair Order. When set, every Sales Order Line generated from an added part of that repair is priced at zero, so the customer pays for none of the parts.

**Stock Reference.** A shared reference record linking a Repair Order to the documents created to replenish its parts. A Manufacturing Order or a Purchase Order pulled by a repair part shares it, which is how each side counts and opens the other.

**Replenish on order.** The route that turns a demand into a replenishing document rather than into a reservation against existing stock. Every warehouse carries a pull rule on that route for its repair operation type, which is how a missing part becomes a Manufacturing Order or a Purchase Order.

**Make to order.** The procurement method assigned to a part movement that will be supplied by a replenishing document rather than from stock on hand. Its counterpart, make to stock, reserves from what is already there.

**Kit.** A product whose bill of materials is of the kit kind: it is never manufactured, it is expanded in place into its components wherever it is moved. A kit added as a part of a repair is replaced by one part per component.

**Explosion.** The act of expanding a kit into its components, recursively through nested kits, at a factor computed from the part's demanded quantity.

**Catalog.** The picking screen that lists products as cards with their prices and their current quantity on the document, and lets a user set quantities directly. Opened from a Repair Order it offers goods only, and every part it creates is of kind *add*.

**Insufficient-quantity dialogue.** The confirmation shown when the item to repair is not physically present at the product source location. It lists where the item actually is and lets the user confirm the repair anyway.

**Repair Tag.** A free label applied to Repair Orders, with a colour. Tag names are unique across the installation.

**Return transfer.** The validated transfer, itself the return of another transfer, through which a broken item came back from the customer. A Repair Order may be bound to one, which narrows its product and lot choices and derives its customer and quantity.

**Repair service.** A service product whose service tracking is the stored value `repair`. Confirming a Sales Order Line for such a product raises exactly one Repair Order, whatever the quantity ordered.

---

## Maintenance vocabulary

**Equipment.** One asset that the business maintains: a machine, a tool, a vehicle, a computer, a piece of furniture. It carries an identity, an owner, an assignment, a location, a warranty date, a category and its effectiveness measurements. Transport name `maintenance.equipment`.

**Equipment Category.** A grouping of equipment. It carries the responsible user proposed as technician on the equipment of the group, free comments, a colour, and the definition of the ad-hoc fields that the equipment of the group exposes.

**Maintained Item.** The reusable definition that makes any record maintainable: it contributes a company, an effective date, a Maintenance Team, a technician, the list of its Maintenance Requests, their counts and the four effectiveness measurements. Equipment is the only maintainable item this domain defines; another domain may make its own records maintainable, and the same computations then apply to them. Transport name `maintenance.mixin`.

**Maintenance Request.** One unit of maintenance work: a subject, an optional equipment, a kind, a schedule, a duration, a priority, a team, a technician, instructions and, for preventive work, a recurrence rule. Transport name `maintenance.request`.

**Corrective maintenance.** Work that repairs something that has already broken. Only corrective requests feed the effectiveness measurements. Stored value `corrective`.

**Preventive maintenance.** Work planned in order to stop something breaking. Only preventive requests may recur. Stored value `preventive`.

**Recurrent request.** A preventive request that automatically creates its successor when it reaches a stage flagged as closing.

**Successor.** The copy of a recurrent preventive request that the closing of that request creates, placed in the first stage and scheduled one repeat interval after its predecessor's planned start.

**Maintenance Stage.** One column of the maintenance pipeline. Stages are ordered by a sequence, may be folded in the pipeline, and may carry the closing flag. Transport name `maintenance.stage`.

**Closing stage.** A stage carrying the closing flag. Reaching it stamps the request's close date, marks its pending activity done, and, for a recurrent preventive request, creates the successor. Two shipped stages carry the flag: "Repaired", meaning the asset was saved, and "Scrap", meaning it was not.

**Folded stage.** A stage shown collapsed in the pipeline, in order to keep finished work out of the way.

**Within-stage signal.** A three-valued marker showing how a request is progressing inside its current stage: In Progress, Blocked, or Ready for next stage. Every stage change resets it to In Progress unless the same operation sets another value. Stored values `normal`, `blocked` and `done`.

**Archive flag.** The Maintenance Request's own hiding marker, presented to the user as "Cancelled". It is not the platform's standard archiving flag; the request has no standard archiving flag. Setting it also switches recurrence off.

**Maintenance Team.** A group of users responsible for Maintenance Requests. It publishes an incoming-mail alias that turns an inbound message into a request assigned to the team, and it owns a dashboard counting its open work. Transport name `maintenance.team`.

**Technician.** The person who performs the work. On a Maintenance Request the field is labelled Responsible; on the calendar the team members are called technicians. The technician receives the request's scheduled activity.

**Created-by user.** On a Maintenance Request, the person who raised it; the field also grants that person access to the request. On an Equipment, the corresponding notion is the owner.

**Owner.** On an Equipment, the person who holds or is accountable for the asset. Without the people bridge it is entered by hand; with it, it is derived from the assignment.

**Assignment mode.** The three-valued choice saying who uses a piece of equipment: an employee, a department, or both. Choosing one of the first two clears the other; choosing the third keeps both. Stored values `employee`, `department` and `other`.

**Assigned date.** The date the equipment was last assigned. With the people bridge it is stamped with today's date every time the assignment mode changes.

**Effective date.** The date an asset entered service. It is the starting point from which the mean time between failures is measured.

**Failure.** For the purposes of the effectiveness measurements, a corrective Maintenance Request that has reached a closing stage. Archived ones count; preventive ones never do.

**Latest failure date.** The most recent request date among the failures of an asset.

**Mean time between failures.** The observed average number of days an asset runs between failures, computed as the whole number of days from the effective date to the latest failure date, divided by the number of failures, truncated toward zero.

**Expected mean time between failures.** The number of days the business expects the asset to run between failures. Entered by hand; never computed; never compared automatically with the observed figure.

**Mean time to repair.** The observed average number of days a repair of the asset takes, computed as the sum of the whole-day spans from request date to close date across the failures, divided by their number, truncated toward zero.

**Estimated next failure.** The latest failure date advanced by the mean time between failures. Empty when the mean time between failures is zero.

**Scrap date.** The date an asset left service. Informational only; it neither archives the asset nor posts anything.

**Warranty expiration date.** The date an asset's warranty ends. Informational only; it is unrelated to the warranty flag of a Repair Order and triggers nothing.

**Serial match.** The detection that an equipment's serial number text is also a registered lot or serial number, which lets a user jump from the asset register to the goods traceability of the same number.

**Projected occurrence.** A future repetition of a recurrent Maintenance Request that the calendar draws but has not created. It is a rendering of the one real request at a later moment, labelled with the request's name followed by a plus sign and the occurrence number. It cannot be dragged or resized, and opening it opens the one real request. The successor record itself is created only when the request reaches a closing stage.

**Worksheet.** A structured form a technician fills while performing maintenance. Worksheets are supplied by a separate capability installed through the Custom Maintenance Worksheets setting; they are outside this domain.

---

## Vocabulary shared by both halves

**Capability package.** An installable unit of functionality. Installing it makes its entities, fields, screens and shipped records exist. Nine of them make up this domain.

**Bridge package.** A capability package that exists only to connect two others, installed automatically when both are present. Six of this domain's nine packages are bridges, and one of those six contributes nothing but the recognition of a supported combination.

**Property definition.** A description of ad-hoc fields held on a parent record. An Equipment Category holds the definition for its equipment; a repair operation type holds the definition for its Repair Orders.

**Properties.** The ad-hoc field values held on a record, shaped by the property definition on its parent.

**Discussion thread.** The message history attached to a record, with its followers. Repair Orders, Equipment, Maintenance Requests and Maintenance Teams all carry one.

**Follower.** A person subscribed to a record's discussion thread. For maintenance, following is also a permission: an ordinary user may read and raise requests for the equipment they follow, without holding the equipment manager role.

**Message subtype.** A named class of thread message that a follower may subscribe to or ignore. This domain ships five: Request Created and Status Changed on a Maintenance Request, Equipment Assigned on an Equipment, and two counterparts on an Equipment Category that relay the first and the third to the followers of the category.

**Scheduled activity.** A dated piece of work assigned to a person and attached to a record. Every open, scheduled Maintenance Request carries exactly one, deadlined on the scheduled date expressed in the acting user's time zone.

**Incoming-mail alias.** An inbound address that turns a received message into a record. A Maintenance Team publishes one, whose target entity is Maintenance Request and whose default values force the team.

**Equipment Manager.** The access group that may create, edit and delete equipment, equipment categories, maintenance stages and maintenance teams, and that sees every Maintenance Request and every Equipment record. A human resources officer holds it implicitly when the people bridge is installed.

**Inventory User.** The access group that operates repairs. Every permission on Repair Orders, Repair Tags and the insufficient-quantity dialogue is attached to it.

**Screen path.** The stable, shareable address segment of a screen. Appending a record's identifier opens that record's form; appending the reserved word `new` opens a blank form. The path grants no permission of its own: model access, record rules and menu visibility still apply.

**Truncation toward zero.** How a computed decimal becomes a whole-number field value: the fractional part is dropped, never rounded. A value of 53.99 becomes 53. This applies to the mean time between failures and the mean time to repair.

**Rounding of a unit of measure.** The smallest increment a unit recognises. Every quantity comparison in this domain names the rounding it uses, and no comparison is made by exact equality. The complete table is calculation 33 of [calculations.md](calculations.md).

**Compatibility finding.** A behaviour recorded here as observed, together with a note saying what a corrected behaviour would be, so that a rebuild makes an informed choice rather than an accidental one. This domain records three: the location at which the owner of the repaired product is tested, the absence of any reversal for a completed repair, and the misspelled screen path of the equipment categories.

**Industry-standard default.** A resolution stated here because the observed behaviour leaves a question open. This domain records two, both in [accounting-effects.md](accounting-effects.md): how analytic amounts reach a repair, and where the labour cost of maintenance is captured.
