# Acceptance criteria

Behaviour a rebuild must exhibit, written as independently verifiable Given, When and Then scenarios with concrete values. Scenarios are numbered `AC-001` upward in one sequence. Each scenario names the rule, the workflow, the calculation or the state transition it verifies, so that the catalogue can be read as a conformance suite: every numbered rule of [business-rules.md](business-rules.md), every numbered calculation of [calculations.md](calculations.md), every numbered workflow of [workflows.md](workflows.md) and every transition of [state-machines.md](state-machines.md) is named by at least one scenario below.

Unless a scenario says otherwise, the fixture is: one company; one warehouse with the short code `WH` whose stock location is `WH/Stock`; a production location `Virtual Locations/Production` and an inventory-loss location `Virtual Locations/Inventory adjustment`; a repair operation type named "Repairs" with the sequence code `RO` and the shipped defaults; the *Product Unit* decimal precision set to two decimal places; the four shipped maintenance stages; and the shipped maintenance team "Internal Maintenance".

---

# Part one: Repair Order creation and defaults

### AC-001: A new Repair Order is numbered from its operation type

- **Given** the repair operation type "Repairs" of warehouse `WH` with the sequence code `RO` and no repair created yet,
- **When** a user creates a Repair Order for the product `Desk Combination` without supplying a reference,
- **Then** the repair's reference is `WH/RO/00001`, its state is `draft`, and a Stock Reference named `WH/RO/00001` exists and is linked to it,
- **And** the sequence that produced it is named as the warehouse name followed by " Sequence repair", is padded to five digits, is prefixed `WH/RO/` and belongs to the warehouse's company, so the second repair of that type reads `WH/RO/00002`.

*Verifies workflow 1, rules RM-001, RM-003 and RM-084, calculation 1, transition T-01.*

### AC-002: A supplied reference is kept

- **Given** the same fixture,
- **When** a Repair Order is created with the reference `LEGACY-7`,
- **Then** the reference stays `LEGACY-7` and no number is drawn from the sequence.

*Verifies rule RM-001.*

### AC-003: The six locations default from the operation type

- **Given** the repair operation type "Repairs" with the default source location `WH/Stock`, the default destination location `Virtual Locations/Production`, the default remove destination `Virtual Locations/Inventory adjustment`, the default recycle destination `WH/Stock`, the default product source `WH/Stock` and the default product destination `WH/Stock`,
- **When** a Repair Order is created under that type,
- **Then** its component source location is `WH/Stock`, its added-parts destination location is `Virtual Locations/Production`, its removed-parts destination location is `Virtual Locations/Inventory adjustment`, its recycled-parts destination location is `WH/Stock`, its product source location is `WH/Stock` and its product destination location is `WH/Stock`,
- **And** the added-parts destination and the removed-parts destination are read-only on every screen: they always mirror the operation type and cannot be typed over,
- **And** the other four locations may be edited afterwards and keep the edited value.

*Verifies workflow 1, rule RM-062 and calculation 2.*

### AC-004: The unit of measure defaults from the product and is not overwritten

- **Given** a product `Desk Combination` whose reference unit is Units,
- **When** a Repair Order is created naming that product with no unit chosen,
- **Then** the repair's unit of measure is Units;
- **And when** the user then sets the unit to Dozens and saves again without changing the product,
- **Then** the unit stays Dozens, because an existing value is never overwritten by the derivation.

*Verifies the unit derivation of [entities.md](entities.md), section 1.6.4.*

### AC-005: A part line takes the product's own unit

- **Given** a Repair Order for `Desk Combination`,
- **When** an `add` part for `Conference Chair`, whose reference unit is Units, is created with no unit chosen,
- **Then** that part's unit is Units.

*Verifies workflow 5.*

### AC-006: The operation type is hidden when only one exists

- **Given** a company owning exactly one repair operation type,
- **When** the Repair Order form is opened,
- **Then** the operation type field is not shown;
- **And given** the company owns two repair operation types,
- **Then** the field is shown.

*Verifies the selector-visibility derivation of [entities.md](entities.md), section 1.6.12.*

---

# Part two: Repair Order state transitions

### AC-007: A negative part quantity blocks confirmation

- **Given** a Repair Order in `draft` with one `add` part for `Product Storable No Tracking #1` whose demanded quantity is −1.0,
- **When** the user confirms the repair,
- **Then** the operation is refused with the message "You can not enter negative quantities.", and the repair stays in `draft`.

*Verifies rule RM-010 and the first guard of transition T-03.*

### AC-008: An unavailable product to repair raises the insufficient-quantity dialogue

- **Given** a Repair Order in `draft` whose product to repair is `Product Storable Serial`, tracked by unique serial number and storable, with no stock anywhere,
- **When** the user confirms the repair,
- **Then** the operation returns a request to open the Insufficient Repair Quantity Warning dialogue, pre-filled with that product, the repair's product source location, the converted quantity and the product's reference unit name, and the repair stays in `draft`.

*Verifies rule RM-011, workflow 9 and calculation 14.*

### AC-009: Accepting the dialogue confirms the repair and reserves each part according to its availability

- **Given** the repair of the previous scenario with three `add` parts: part A demanding 2.0 of `Product Storable No Tracking #1` of which 1.0 is in stock, part B demanding 2.0 of `Product Storable Lot` of which 3.0 is in stock, part C demanding 1.0 of `Product Storable No Tracking #2` of which none is in stock,
- **When** the user accepts the dialogue,
- **Then** the repair's state is `confirmed`, part A is partially available, part B is fully available, and part C is waiting for availability.

*Verifies workflow 9 and the Stock Move states of [state-machines.md](state-machines.md), section 3.*

### AC-010: A repair whose product is not storable confirms directly

- **Given** a Repair Order in `draft` whose product to repair is a service, or a good that does not track inventory, and whose parts all have non-negative demands,
- **When** the user confirms the repair,
- **Then** no dialogue is raised and the state becomes `confirmed`.

*Verifies rule RM-011.*

### AC-011: Start moves a confirmed repair to under repair

- **Given** a Repair Order in `confirmed`,
- **When** the user starts the repair,
- **Then** its state is `under_repair` and no movement has changed state.

*Verifies workflow 13 and transition T-05.*

### AC-012: Start confirms a repair that is still new

- **Given** a Repair Order in `draft` with no negative part quantities,
- **When** the user starts the repair,
- **Then** the confirmation side effects run and the final state is `under_repair`.

*Verifies workflow 13 and transition T-04.*

### AC-013: Ending a repair that is not under repair is refused

- **Given** a Repair Order in `confirmed`,
- **When** the user ends the repair,
- **Then** the operation is refused with the message "Repair must be under repair in order to end reparation." and nothing is written.

*Verifies rule RM-020 and the first guard of transition T-06.*

### AC-014: A tracked product without a lot blocks completion

- **Given** a Repair Order in `under_repair` whose product to repair is `Product Storable Serial` and whose lot field is empty,
- **When** the user ends the repair,
- **Then** the operation is refused with the message "Serial number is required for product to repair : Product Storable Serial", and the repair stays in `under_repair`.

*Verifies rule RM-021 and the third guard of transition T-06.*

### AC-015: Completion cancels the zero-quantity parts and completes the rest

- **Given** a Repair Order in `under_repair` with four `add` parts: A demanding 2.0 with a recorded quantity of 2.0, B demanding 2.0 with a recorded quantity of 2.0, C demanding 1.0 with a recorded quantity of 2.0, D demanding 0.0 with a recorded quantity of 0.0,
- **When** the user ends the repair,
- **Then** parts A, B and C are `done`, part D is `cancel`, the repair holds exactly four part movements — none was split — and the state is `done`.

*Verifies rules RM-023 and RM-025, workflow 14 and transitions T-26 and T-27.*

### AC-016: Completion creates the repaired-product movement with no transfer

- **Given** a Repair Order in `under_repair` for `Product Storable Serial` with the serial number `S1`, quantity 1.0, product source location `WH/Stock` and product destination location `WH/Stock`,
- **When** the user ends the repair,
- **Then** exactly one Stock Move is created for that product, from `WH/Stock` to `WH/Stock`, quantity 1.0, marked picked, with no transfer and no part kind, and it is written onto the repair as its inventory move,
- **And** none of the repair's movements belongs to any transfer.

*Verifies workflow 14 steps 8 and 9, and rule RM-028.*

### AC-017: The repaired-product detail line consumes the part detail lines

- **Given** the repair of the previous scenario with two `add` parts that were completed,
- **When** the repair is ended,
- **Then** the single detail line of the repaired-product movement records the detail lines of both part movements as consumed lines, and the traceability report of the repaired serial number reaches the two parts through them.

*Verifies rule RM-029 and calculation 10.*

### AC-018: A short part quantity asks for confirmation

- **Given** a Repair Order in `under_repair` with two `add` parts demanding 3.0 and 4.0, whose recorded quantities are 1.0 and 2.0,
- **Then** the repair reports incomplete parts;
- **When** the recorded quantities are corrected to 3.0 and 4.0,
- **Then** the repair no longer reports incomplete parts;
- **And when** they are set to 3.0 and 5.0,
- **Then** the repair still does not report incomplete parts, because an excess does not count.

*Verifies rule RM-022 and calculation 5.*

### AC-019: A completed repair cannot be cancelled

- **Given** a Repair Order in `done`,
- **When** the user cancels it,
- **Then** the operation is refused with the message "You cannot cancel a Repair Order that's already been completed", and the state stays `done`.

*Verifies rule RM-030, workflow 15 and transition T-14.*

### AC-020: Cancelling cancels every part

- **Given** a Repair Order in `confirmed` with three parts,
- **When** the user cancels the repair,
- **Then** the state is `cancel` and all three parts are `cancel`.

*Verifies rule RM-032, workflow 15 and transition T-08.*

### AC-021: Setting a cancelled repair back to new reopens the parts

- **Given** the cancelled repair of the previous scenario, whose three parts had generated Sales Order Lines on a quotation that is not itself cancelled and whose quantities were set to zero by the cancellation,
- **When** the user sets it back to new,
- **Then** the state is `draft` and all three parts are `draft`,
- **And** the Sales Order Line of each `add` part is restored to the sum of the demanded quantities of the `add` movements attached to it, so a line fed by one `add` movement of 2.00 reads 2.00 again,
- **And** the Sales Order Line of a part whose kind is `remove` or `recycle` is restored to zero, not to the movement's quantity.

*Verifies workflow 16, rule RM-033 and transitions T-10 and T-28.*

### AC-022: Deleting a new repair deletes its parts

- **Given** a Repair Order in `draft` with one `add` part,
- **When** the user deletes the repair,
- **Then** neither the repair nor the part movement exists any longer.

*Verifies rule RM-034, workflow 17 and transition T-12.*

### AC-023: Marking parts picked at completion is all or nothing

- **Given** a Repair Order under repair with three `add` parts: `Group gasket` demanding 2.00 with a recorded quantity of 2.00, `Water pump` demanding 1.00 with a recorded quantity of 1.00, and `Cable` demanding 1.00 with a recorded quantity of 1.00, and none of the three movements marked picked,
- **When** the repair is ended,
- **Then** all three movements are marked picked and all three reach `done`, so all three parts are consumed out of `WH/Stock`.

- **And given** the same repair, except that the technician marked only the `Group gasket` movement picked before pressing End Repair,
- **When** the repair is ended,
- **Then** `Group gasket` is still the only movement marked picked, it reaches `done` and its two units are consumed,
- **And** `Water pump` and `Cable` are left unmarked and are cancelled by the completion rather than carried forward, because completion runs with backorder creation suppressed; their goods are not consumed and no residual document records them,
- **And** the repair still reaches `done`.

*Verifies rules RM-024 and RM-025, and workflow 14 steps 4 and 10.*

### AC-024: The customer owns the repaired product only when the customer already owned enough

- **Given** the *Product Unit* decimal precision set to two decimal places, a storable product `Espresso machine` that is not tracked, a customer `Wood Corner`, and a Repair Order for 1.00 of `Espresso machine` for `Wood Corner` whose component source location and product source location are both `WH/Stock`,
- **When** `WH/Stock` holds 1.00 of `Espresso machine` recorded as owned by `Wood Corner`, and the repair is confirmed, started and ended,
- **Then** the detail line of the repaired-product movement records `Wood Corner` as the owner, because 1.00 is not lower than 1.00 at the *Product Unit* precision.

- **And given** the same repair where `WH/Stock` holds 1.00 of `Espresso machine` with no owner recorded,
- **When** the repair is completed,
- **Then** the detail line of the repaired-product movement records no owner, so the goods belong to the company.

- **And given** the same repair where `WH/Stock` holds 0.60 owned by `Wood Corner` and 0.40 with no owner, and the user accepted the insufficient-quantity dialogue at confirmation,
- **When** the repair is completed,
- **Then** the detail line records no owner, because the quantity owned by the customer is 0.60, which is lower than 1.00 at the *Product Unit* precision, and the two buckets are never added together.

*Verifies rule RM-026, calculation 15 and workflow 14 step 7.*

---

# Part three: parts, movements and locations

### AC-025: Part kinds resolve the locations

- **Given** a Repair Order whose component source location is `WH/Stock`, added-parts destination `Virtual Locations/Production`, removed-parts destination `Virtual Locations/Inventory adjustment` and recycled-parts destination `WH/Stock`,
- **When** three parts are created, one of each kind,
- **Then** the `add` part runs from `WH/Stock` to `Virtual Locations/Production`, the `remove` part from `Virtual Locations/Production` to `Virtual Locations/Inventory adjustment`, and the `recycle` part from `Virtual Locations/Production` to `WH/Stock`.

*Verifies rule RM-060 and calculation 2.*

### AC-026: The worked example of two parts added and one removed

- **Given** a Repair Order `WH/RO/00007` for an espresso machine with the serial number `ESP-0042`, quantity 1, customer Wood Corner, under the shipped operation type,
- **And** three parts: `add` two `Group gasket`, `add` one `Water pump`, `remove` one `Water pump`,
- **When** the repair is confirmed, started with recorded quantities equal to the demands, and ended,
- **Then** exactly four movements exist and are `done`:

  | Product | Quantity | From | To | Part kind |
  |---|---|---|---|---|
  | Group gasket | 2 | `WH/Stock` | `Virtual Locations/Production` | `add` |
  | Water pump | 1 | `WH/Stock` | `Virtual Locations/Production` | `add` |
  | Water pump | 1 | `Virtual Locations/Production` | `Virtual Locations/Inventory adjustment` | `remove` |
  | Espresso machine | 1 | `WH/Stock` | `WH/Stock` | none |

- **And** each of the first three carries the origin `WH/RO/00007`, the document reference `WH/RO/00007`, the repair's operation type and the repair's Stock Reference, and none carries a transfer;
- **And** the Parts list of the repair shows exactly the first three.

*Verifies calculation 2, rule RM-028 and workflow 14.*

### AC-027: Changing a repair location relocates the parts

- **Given** a confirmed Repair Order with one `add` part sourced from `WH/Stock`,
- **When** the component source location is changed to `WH/Shelf 2`,
- **Then** the part's source location becomes `WH/Shelf 2`;
- **And when** the product source location is changed instead,
- **Then** no part movement is relocated.

*Verifies rule RM-061.*

### AC-028: Changing the operation type renumbers and relocates

- **Given** a Repair Order created under an operation type whose sequence prefix is `PT1/` and whose source location is `Stock`, with one `add` part demanding 1 of a product held only in `Stock 2`,
- **Then** the repair is `PT1/00001`, its part's document reference is `PT1/00001`, its part is sourced from `Stock` and its reserved quantity is 0.0;
- **When** the operation type is changed to one whose sequence prefix is `PT2/` and whose source location is `Stock 2`,
- **Then** the repair is `PT2/00001`, the part's document reference is `PT2/00001`, the part is sourced from `Stock 2` and its reserved quantity is 1.0;
- **When** the operation type is changed back to the first one,
- **Then** the repair is `PT1/00002`;
- **When** the same first operation type is written a second time,
- **Then** the repair is still `PT1/00002`.

*Verifies rule RM-002, workflow 18 and calculation 1.*

### AC-029: Changing the scheduled date propagates to the movements

- **Given** a confirmed Repair Order scheduled on 1 October 2025 at 09:00 with two parts that are neither done nor cancelled,
- **When** the scheduled date is changed to 8 October 2025 at 14:00,
- **Then** both parts carry the date 8 October 2025 at 14:00;
- **And given** a third part that is already cancelled,
- **Then** its date is unchanged.

*Verifies rule RM-064 and workflow 19.*

### AC-030: Parts added to a running repair are confirmed at once

- **Given** a Repair Order in `confirmed`,
- **When** an `add` part is created on it,
- **Then** that part is not in `draft`: it has been confirmed, its procurement method has been adjusted and the replenishment scheduler has run for it.

*Verifies rule RM-014 and transition T-22.*

### AC-031: A part movement is never assigned by the automatic pass and never split

- **Given** a Repair Order in `under_repair` with one `add` part demanding 4.0 whose recorded quantity is 2.0,
- **When** the repair is ended,
- **Then** exactly one movement exists for that part, with a recorded quantity of 2.0, and no residual movement and no backorder transfer were created.

*Verifies rule RM-025 and the two departures of [state-machines.md](state-machines.md), section 3.3.*

### AC-032: Duplicating a part movement drops the repair link

- **Given** a part movement belonging to a Repair Order,
- **When** the movement is duplicated,
- **Then** the copy belongs to no Repair Order and carries no Sales Order Line.

*Verifies the duplication rule of [entities.md](entities.md), section 2.2.*

### AC-033: Removed and recycled parts are not forecast against stock

- **Given** a confirmed Repair Order with one `remove` part demanding 1.0 of a product with no stock,
- **Then** that part's forecast availability equals its own quantity and it carries no forecast expected date, so it does not make the repair report "Not Available".

*Verifies the forecast override of [entities.md](entities.md), section 2.2, and calculation 3.*

### AC-034: The parts catalog offers goods only

- **Given** a Repair Order and two products, `Conference Chair` of the goods kind and `Repair Service` of the service kind,
- **When** the catalog is opened from the repair,
- **Then** `Conference Chair` satisfies the catalog restriction and `Repair Service` does not.

*Verifies rule RM-091.*

### AC-035: Setting a catalog quantity to zero deletes the part

- **Given** a Repair Order with an `add` part for `Conference Chair` demanding 3.0,
- **When** the catalog sets the quantity of `Conference Chair` to 0,
- **Then** the part movement no longer exists;
- **And when** the catalog then sets it to 2,
- **Then** a new `add` part exists demanding 2.0, sourced from the repair's component source location and destined to the repair's added-parts destination location,
- **And** the operation returns the product's list price.

*Verifies workflow 6.*

### AC-036: A repair drawing from another warehouse than the return warns without blocking

- **Given** two warehouses, `WH` with the stock location `WH/Stock` and `WH2` with the stock location `WH2/Stock`, and a validated return transfer whose destination location is `WH2/Stock`,
- **When** a Repair Order is given that return as its return transfer while its component source location is `WH/Stock`,
- **Then** a non-blocking warning is raised, titled "Warning", carrying the message "Note that the warehouses of the return and repair locations don't match!",
- **And** the chosen values are kept: the record saves, the repair confirms, and nothing about the movements changes.

- **And given** the component source location is then changed to `WH2/Shelf 1`, which belongs to `WH2`,
- **Then** no warning is raised, because the two warehouses now match.

- **And given** a return transfer whose destination location belongs to no warehouse at all, for example the customer location,
- **Then** no warning is raised whatever the component source location, because the condition requires a warehouse on both sides.

*Verifies rule RM-063.*

---

# Part four: readiness and counters

### AC-037: Readiness is empty outside the two running states

- **Given** a Repair Order in `draft`,
- **Then** its component status text and its component status code are both empty;
- **And given** the same repair in `done`,
- **Then** both are empty again.

*Verifies calculation 3 step 1 and transitions T-15 and T-20.*

### AC-038: All parts on hand reports Available

- **Given** a confirmed Repair Order with two `add` parts, each demanding 2.0, each fully reserved, neither carrying a forecast expected date,
- **Then** the component status reads "Available", the code is `available`, the readiness boolean is true and the lateness boolean is false.

*Verifies calculation 3 worked example A, calculation 4 and transition T-17.*

### AC-039: A shortage reports Not Available and late

- **Given** the same repair where the second part demands 5.0 and the forecast can cover only 3.0,
- **Then** the component status reads "Not Available", the code is `late`, the readiness boolean is false and the lateness boolean is true.

*Verifies calculation 3 worked example B and transition T-16.*

### AC-040: An incoming part later than the schedule reports late

- **Given** a confirmed Repair Order scheduled on 1 October 2025, whose parts are all covered by incoming receipts the latest of which is expected on 4 October 2025,
- **Then** the component status reads "Exp " followed by the formatted date 4 October 2025 and the code is `late`.

*Verifies calculation 3 worked example C and transition T-19.*

### AC-041: An incoming part earlier than the schedule reports expected

- **Given** the same repair scheduled on 10 October 2025,
- **Then** the component status reads "Exp " followed by the formatted date 4 October 2025, the code is `expected`, and both derived booleans are false.

*Verifies calculation 3 worked example D and transition T-18.*

### AC-042: The overview counters of a repair operation type

- **Given** a repair operation type with three repairs confirmed with all parts available and scheduled next week, one confirmed with a missing part, two under repair, one completed and one cancelled,
- **Then** its confirmed counter reads 4, its under-repair counter reads 2, its ready counter reads 3 and its late counter reads 1.

*Verifies calculation 16 and rule RM-122.*

### AC-043: A non-repair operation type gets no repair counters

- **Given** an operation type whose code is `outgoing`,
- **Then** all four repair counters read false.

*Verifies calculation 16.*

### AC-044: The scheduled-date category search

- **Given** a Repair Order scheduled at the current moment and confirmed,
- **When** a search is run for the categories `yesterday` and `today` together,
- **Then** the repair is in the result;
- **And when** the search uses any operator other than membership in a list of categories,
- **Then** the search is rejected.

*Verifies calculation 12 and rule RM-121.*

### AC-045: The Check availability and Unreserve buttons appear only when they can act

- **Given** a Repair Order with two `add` parts, `Group gasket` demanding 2.00 and `Water pump` demanding 5.00, and a warehouse holding 2.00 gaskets and 2.00 pumps,
- **When** the repair is still in `draft`,
- **Then** neither the Check availability button nor the Unreserve button is shown.

- **And when** the repair is confirmed and nothing is reserved yet, so both part movements sit in the `confirmed` state with no reserved quantity,
- **Then** the Check availability button is shown and the Unreserve button is not.

- **And when** Check availability is pressed, so `Group gasket` becomes fully reserved at 2.00 and `Water pump` becomes partially available at 2.00 of 5.00,
- **Then** both buttons are shown at once: Unreserve because two detail lines now hold a reservation, Check availability because `Water pump` is still unpicked, still demands a quantity and is only partially available.

- **And when** the three missing pumps arrive and Check availability is pressed again, so both movements are fully reserved,
- **Then** only the Unreserve button is shown; the Check availability button disappears because no part is left in a reservable state.

- **And when** Unreserve is pressed, so both movements return to the `confirmed` state and no detail line holds a reserved quantity,
- **Then** only the Check availability button is shown again.

- **And when** the repair is completed, or cancelled,
- **Then** neither button is shown.

*Verifies calculation 11, workflow 10, workflow 11 and transitions T-24 and T-25.*

---

# Part five: Sales Order binding

### AC-046: Confirming a Sales Order raises one repair per qualifying line

- **Given** a Sales Order for the customer Wood Corner holding one line of `Repair Service`, a service whose service tracking is `repair`, with a quantity of 2.0, and one section line,
- **Then** before confirmation the order shows no repair;
- **When** the order is confirmed,
- **Then** exactly one Repair Order exists, bound to the order and to the service line, in state `confirmed`, with the order's customer and the warehouse's repair operation type.

*Verifies rule RM-045, workflow 4 and transition T-02.*

### AC-047: Zeroing the line cancels the repair, restoring it reconfirms

- **Given** the repair of the previous scenario,
- **When** the service line quantity is set to 0,
- **Then** the repair's state is `cancel`;
- **When** the quantity is set back to 1,
- **Then** the repair's state is `confirmed`;
- **And when** a section line on the same order is edited,
- **Then** nothing about the repair changes.

*Verifies rule RM-046.*

### AC-048: Cancelling the repair zeroes the line

- **Given** a repair bound to a Sales Order Line with a quantity of 1,
- **When** the repair is cancelled,
- **Then** the line's ordered quantity is 0;
- **When** the line quantity is then set to 3,
- **Then** the repair is `confirmed` again.

*Verifies rules RM-031 and RM-046.*

### AC-049: A quotation cannot be created twice

- **Given** a Repair Order already bound to a Sales Order,
- **When** the user creates a quotation from it,
- **Then** the operation is refused with a message beginning "You cannot create a quotation for a repair order that is already linked to an existing sale order." and continuing with "Concerned repair order(s):" and the repair's reference.

*Verifies rule RM-040.*

### AC-050: A quotation requires a customer

- **Given** a confirmed Repair Order with no customer,
- **When** the user creates a quotation from it,
- **Then** the operation is refused with a message beginning "You need to define a customer for a repair order in order to create an associated quotation." and continuing with "Concerned repair order(s):" and the repair's reference;
- **When** a customer is then set and the operation repeated,
- **Then** a Sales Order is created and linked, with the repair's company, the repair's customer, the warehouse of the repair's operation type and the repair's reference as its origin.

*Verifies rule RM-041 and workflow 21.*

### AC-051: Only added parts become Sales Order Lines

- **Given** a confirmed Repair Order with three `add` parts and no Sales Order,
- **When** a quotation is created from it,
- **Then** the resulting Sales Order holds exactly three lines;
- **And given** the repair then gains one `remove` part and one `recycle` part,
- **Then** the Sales Order still holds exactly three lines.

*Verifies rule RM-042.*

### AC-052: Changing a part's kind drives its Sales Order Line

- **Given** an `add` part whose Sales Order Line quantity equals the part demand,
- **When** the part's kind is changed to `remove`,
- **Then** the Sales Order Line quantity becomes 0;
- **When** it is changed back to `add`,
- **Then** the Sales Order Line quantity returns to the part's demand;
- **When** it is changed to `recycle`,
- **Then** the Sales Order Line quantity becomes 0 again;
- **When** it is changed back to `add`,
- **Then** the quantity returns to the part's demand.

*Verifies rule RM-042 and calculation 7.*

### AC-053: Quantities flow one way only

- **Given** a part demanding 1.0 whose Sales Order Line reads 1.0,
- **When** the Sales Order Line quantity is set to 5.0,
- **Then** the part still demands 1.0;
- **When** the part demand is then set to 3.0,
- **Then** the Sales Order Line reads 3.0.

*Verifies rule RM-043 and calculation 7.*

### AC-054: A part converted from remove to add creates its Sales Order Line

- **Given** a Repair Order bound to a Sales Order with some number of lines, plus one `remove` part and one `recycle` part,
- **When** the `remove` part's kind is changed to `add`,
- **Then** the Sales Order holds one line more than before and the new line's quantity equals that part's demand.

*Verifies rule RM-042.*

### AC-055: Deleting a part zeroes its Sales Order Line

- **Given** the part of the previous scenario with a Sales Order Line,
- **When** the part is deleted from the repair,
- **Then** the Sales Order Line still exists and its ordered quantity is 0.

*Verifies rule RM-042 and workflow 5.*

### AC-056: Ending the repair delivers the service line and the part lines

- **Given** a Repair Order raised by a Sales Order Line of 2.0 units of a repair service, and one `add` part whose recorded quantity equals its demand,
- **When** the repair is ended,
- **Then** the service line's delivered quantity equals its ordered quantity of 2.0, and the part line's delivered quantity equals the part's recorded quantity;
- **And** the Sales Order Line of a part that was deleted from the repair has a delivered quantity of 0.

*Verifies rules RM-027 and RM-048, and calculation 8.*

### AC-057: A repair raised from a repair-service line reports delivery on the generated order

- **Given** a Repair Order for a repair service with one `add` part of that same service product demanding 1.0,
- **When** the repair is confirmed, started and ended, then a quotation is created and confirmed,
- **Then** the repair's part movement has a recorded quantity of 1.0 and the Sales Order Line's delivered quantity is 1.0.

*Verifies rule RM-048.*

### AC-058: A repair-backed line creates no delivery

- **Given** a Sales Order carrying one line generated from an `add` part of a repair,
- **When** the Sales Order is confirmed,
- **Then** no delivery transfer is created for that line.

*Verifies rule RM-047.*

### AC-059: A discount entered on a generated line survives completion

- **Given** a Repair Order with one `add` part whose Sales Order Line carries a discount of 15 percent,
- **When** the repair is confirmed, started and ended,
- **Then** the discount on that line is still 15 percent.

*Verifies workflow 20 and calculation 6.*

### AC-060: The purchase price of a generated line follows the part's cost

- **Given** a product `Conference Chair` with a cost of 10.00 and a Repair Order with one `add` part for it,
- **When** a quotation is created from the repair,
- **Then** the generated line names `Conference Chair` and its purchase price is 10.00.

*Verifies workflow 21 step 5, in deployments that carry a margin capability.*

---

# Part six: warranty

### AC-061: Ticking the warranty zeroes the part prices

- **Given** a Repair Order bound to a Sales Order, with one `add` part for `Conference Chair` whose list price is 30.00, whose generated line reads 30.00,
- **When** the repair's warranty flag is ticked,
- **Then** that line's unit price is 0.00 and its stored manual-price marker is 0.00;
- **When** the flag is unticked,
- **Then** that line's unit price is 30.00 again.

*Verifies rule RM-044, workflow 20 and calculation 6.*

### AC-062: A part added while under warranty is priced at zero

- **Given** a Repair Order bound to a Sales Order whose warranty flag is already ticked,
- **When** an `add` part for a product with a list price of 30.00 is created,
- **Then** the generated Sales Order Line's unit price is 0.00.

*Verifies calculation 6.*

---

# Part seven: returns, transfers and lots

### AC-063: A repair created from a return inherits the return's values

- **Given** a validated delivery of one unit of `Test Product` to Wood Corner, and a validated return of that delivery landing in `WH/Shelf 2`,
- **When** the user triggers the create-repair action on the return and saves the repair for `Test Product`,
- **Then** the repair's component source location is the return's destination location, its customer is the return's contact, and its operation type is the repair operation type of the return's warehouse.

*Verifies workflow 2.*

### AC-064: Parts of a repair never join the return transfer

- **Given** the repair of the previous scenario with one `add` part for `Product 5`,
- **When** the repair is started and ended,
- **Then** the repair's state is `done` and the return transfer still holds exactly one movement.

*Verifies workflow 2 and rule RM-028.*

### AC-065: The product quantity is derived from the transfer

- **Given** a return transfer that brought back two units of lot `L-001` and one unit of lot `L-002` of a lot-tracked pump,
- **When** a repair is created from that transfer naming that product with no lot,
- **Then** its product quantity is 3.0;
- **When** lot `L-001` is chosen,
- **Then** its product quantity is 2.0;
- **When** lot `L-002` is chosen,
- **Then** its product quantity is 1.0.

*Verifies calculation 13.*

### AC-066: Changing the product clears an incompatible lot

- **Given** a Repair Order for `Product Storable Serial` with the serial number `sn_1` chosen,
- **When** the product is changed to `Product Storable No Tracking #1`,
- **Then** the lot field is empty.

*Verifies the lot derivation of [entities.md](entities.md), section 1.6.5.*

### AC-067: A repair can be raised from a lot of another company

- **Given** a Lot or Serial Number `sn_1` of `Desk Combination`,
- **When** the user opens the repairs of that lot and creates a Repair Order from that screen for that product and customer,
- **Then** the repair's company is the company carried by the screen context and the repair is created without error.

*Verifies workflow 3.*

### AC-068: Generating a serial number for the product to repair

- **Given** a Repair Order for `Product Storable Lot` whose operation type allows creating new lots, with no lot chosen,
- **When** the user triggers the serial generation,
- **Then** a Lot or Serial Number is created for that product and is written onto the repair.

*Verifies workflow 22.*

### AC-069: Generating a serial number without a sequence is refused

- **Given** a Repair Order for a product with no lot numbering sequence and no previous serial number,
- **When** the user triggers the serial generation,
- **Then** the operation is refused with the message "Please set the first Serial Number or a default sequence".

*Verifies rule RM-051.*

### AC-070: Creating a lot from a repair whose type forbids it is refused

- **Given** a Repair Order whose operation type does not allow creating new lots,
- **When** a lot creation is attempted from inside that repair,
- **Then** the operation is refused with the message "You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers"."

*Verifies rule RM-050.*

### AC-071: A tracked product held in a package can still be repaired

- **Given** two serial numbers `sn_1` and `sn_2` of `Desk Combination`, each held in a different package in `WH/Stock`,
- **And** a Repair Order for that product with the serial number `sn_1`, component source location `WH/Stock`, one `add` part demanding 1.0,
- **When** the repair is confirmed, started with the part's recorded quantity set to 1.0, and ended,
- **Then** the repair's state is `done`.

*Verifies workflow 14 with packaged goods.*

### AC-072: Lot counters on a lot record

- **Given** a lot `L-001` named as the product lot of two repairs, one `confirmed` and one `done`,
- **Then** the lot's in-repair counter reads 1 and its repaired counter reads 1;
- **And given** the lot appears on a completed `add` part of a third repair,
- **Then** the lot's repair-part counter reads 1 and the list of repairs that used it holds that third repair.

*Verifies rule RM-052.*

### AC-073: Component lots reach the invoice

- **Given** a Repair Order with one `add` part for `Product Storable Serial` reserved on the serial number `S1`, completed,
- **When** a quotation is created from the repair, confirmed and invoiced, and the invoice is posted,
- **Then** the invoice reports exactly one lot line, naming `Product Storable Serial` and the serial number `S1`.

*Verifies the Stock Move Line extension of [entities.md](entities.md), section 7.10.*

### AC-074: A unique-serial component removed by a repair can be consumed again

- **Given** a product `Press` tracked by unique serial number and a component `Control board` tracked by unique serial number,
- **And** a completed Manufacturing Order that produced one `Press` with the serial number `A1`, consuming one `Control board` with the serial number `B2`,
- **When** a Repair Order is created for `Press` with the lot `A1` and one `remove` part for `Control board`, the detail line of that part is set to the serial number `B2`, and the repair is confirmed, started and ended,
- **Then** the `remove` movement runs out of `Virtual Locations/Production` and belongs to no Manufacturing Order,
- **And when** a second Manufacturing Order for `Press` with the serial number `A2` is created, consuming `Control board` serial number `B2` again, and is marked done,
- **Then** it reaches `done` and no refusal about an already-consumed serial number is raised, because taking the component out of the repaired product released its serial number.

*Verifies rule RM-053, calculation 2 and workflow 14.*

### AC-075: A recycled unique-serial component returns to stock and can be consumed again

- **Given** a storable finished product `Cabinet` and a storable component `Motor` tracked by unique serial number, with one unit of `Motor` serial number `USN01` on hand in `WH/Stock`,
- **And** a completed Manufacturing Order that produced one `Cabinet`, consuming `Motor` serial number `USN01`,
- **When** a Repair Order for `Cabinet` with one `recycle` part for `Motor`, its detail line set to the serial number `USN01`, is confirmed, started and ended with the shipped recycle destination `WH/Stock`,
- **Then** the recycle movement runs from `Virtual Locations/Production` to `WH/Stock`, one unit of `USN01` is on hand again in `WH/Stock`, and that detail line counts as a returned serial number because its destination is internal,
- **And when** a second Manufacturing Order for `Cabinet` consumes `Motor` serial number `USN01`,
- **Then** it reaches `done` and its consumed detail line records the serial number `USN01`.

- **And when** the cycle is repeated — the component is recycled back to `WH/Stock` by a second repair, added back into the product by a third repair with an `add` part drawing the serial number `USN01` from `WH/Stock`, and recycled out again by a fourth repair —
- **Then** each of those three repairs reaches `done`, and a further Manufacturing Order still consumes `Motor` serial number `USN01` and reaches `done` without any refusal.

*Verifies rule RM-053, calculation 2 and workflow 14.*

### AC-076: A serial released by a repair survives an unbuild of the order that consumed it

- **Given** a storable finished product `Cabinet` and a storable component `Motor` tracked by unique serial number, serial number `USN01`,
- **And** a Repair Order on another product carrying one `remove` part for `Motor` whose detail line is the serial number `USN01`, confirmed, started and ended,
- **And** one unit of `USN01` recorded on hand in `WH/Stock` afterwards, so that the product's on-hand quantity is 1,
- **When** a Manufacturing Order for `Cabinet` consumes `Motor` serial number `USN01` and is marked done,
- **And** that Manufacturing Order is then unbuilt,
- **And** a second Manufacturing Order for `Cabinet` consumes `Motor` serial number `USN01` and is marked done,
- **Then** both manufacturing orders reach `done`, both record `USN01` as the consumed serial number, and the second raises no refusal.

*Verifies rule RM-053 and workflow 14.*

### AC-077: A serial removed into the inventory-loss location and brought back can be consumed again

- **Given** a storable component `Motor` tracked by unique serial number, serial number `SN01`, on hand in `WH/Stock`, and a finished product `Cabinet` built from one `Motor`,
- **And** a completed Manufacturing Order that produced one `Cabinet`, consuming `SN01`,
- **When** a Repair Order for `Cabinet` is created with one `remove` part for `Motor`, its detail line set to `SN01` and the part's destination location being the inventory-loss location `Virtual Locations/Inventory adjustment`, and the repair is confirmed, started and ended,
- **And** a separate completed movement of one unit of `SN01` from `Virtual Locations/Inventory adjustment` back to `WH/Stock` is recorded,
- **When** a second Manufacturing Order for `Cabinet` consumes `Motor` serial number `SN01` and is marked done,
- **Then** it reaches `done` and its consumed detail line reads product `Motor`, serial number `SN01`, quantity 1.0, state `done`,
- **And** the removal's destination here is the inventory-loss location, which is **not** internal, so this detail line does **not** add to the returned-serial-number count of rule RM-053, while the serial number nevertheless becomes consumable again, because what releases it is that the movement runs out of a production location under no Manufacturing Order. The two tests are independent and both hold.

*Verifies rule RM-053, its second effect, and calculation 2.*

---

# Part eight: replenishment and kits

### AC-078: A repair pulls a Manufacturing Order for a make-to-order part

- **Given** the replenish-on-order route active, its repair rule set to make to order, and a product routed through replenish on order and manufacture,
- **And** a Repair Order with one `add` part demanding 1.0 of that product,
- **When** the repair is confirmed,
- **Then** a Manufacturing Order exists for that product with a quantity of 1.0, its destination movement belongs to the repair, the Manufacturing Order's repair counter reads 1 and the repair's manufacturing counter reads 1.

*Verifies workflow 12 and rule RM-012.*

### AC-079: A reordering rule triggered by a repair produces a transfer, not a repair movement

- **Given** a reordering rule on `WH/Shelf 2` for `Product Storable No Tracking #1` with a minimum of 0 and a maximum of 1, fed by a pull rule from `WH/Stock`,
- **And** a Repair Order whose component source location is `WH/Shelf 2` with one `add` part demanding 1.0 of that product, whose product to repair is storable and out of stock,
- **When** the repair is confirmed through the insufficient-quantity dialogue,
- **Then** the repair is `confirmed`, and a movement exists for that product from `WH/Stock` to `WH/Shelf 2` that belongs to a transfer and to no Repair Order.

*Verifies workflow 12.*

### AC-080: A kit added to a confirmed repair is exploded

- **Given** a confirmed Repair Order with no parts, and a kit product whose bill of materials holds two components,
- **When** an `add` part for the kit is created on the repair,
- **Then** the repair holds exactly two parts, one per component, and none for the kit.

*Verifies workflow 7 and calculation 9.*

### AC-081: The explosion ratio multiplies the component quantities

- **Given** a kit that produces one unit from two gaskets and one seal,
- **When** an `add` part demanding three kits is created on a repair,
- **Then** the repair holds one part demanding six gaskets and one demanding three seals, each of kind `add`, each with the kit movement's source and destination locations and unit price,
- **And** a service component of the same kit produces no part at all.

*Verifies calculation 9.*

### AC-082: A product built from a kit can be repaired

- **Given** a Repair Order whose product to repair is a product with a kit bill of materials,
- **When** the repair is confirmed, started and ended,
- **Then** the repair completes without error.

*Verifies workflow 7 and workflow 14.*

---

# Part nine: valuation and invoicing

### AC-083: A repair part is expensed on the invoice when the repair posted nothing

- **Given** a company that recognises the cost of goods sold on the invoice, automated first-in-first-out valuation, a product `Part` received once at a unit cost of 10.00 and once at 25.00, with a sales price of 1.00, and no valuation account on the production location,
- **And** a Repair Order with one `add` part of 1.00 of `Part`, confirmed, started and ended,
- **When** a quotation is created from the repair, confirmed and invoiced, and the invoice is posted,
- **Then** the invoice's journal items are exactly: revenue credited 1.00, receivable debited 1.00, stock valuation credited 10.00, expense debited 10.00.

*Verifies [accounting-effects.md](accounting-effects.md), section 4, and workflow 23.*

### AC-084: A repair part valued at the repair is not expensed again on the invoice

- **Given** automated first-in-first-out valuation, five units of a product in `WH/Stock` at 10.00 each, a sales price of 20.00, and a valuation account set on the repair operation type's default destination location,
- **And** a Repair Order with one `add` part of 1.00 of that product, confirmed, started and ended,
- **When** a quotation is created from the repair, confirmed and invoiced, and the invoice is posted,
- **Then** the repair part movement carries a journal entry whose items are stock valuation credited 10.00 and the destination location's valuation account debited 10.00, with the Repair Order's reference as the entry reference and no counterparty,
- **And** the invoice's journal items are exactly: revenue credited 20.00 and receivable debited 20.00, with no cost-of-goods-sold items.

*Verifies rule RM-100 and [accounting-effects.md](accounting-effects.md), section 3.*

### AC-085: A removed part changes no inventory value

- **Given** a Repair Order with one `remove` part of a storable product, completed,
- **Then** that movement runs from the production location to the inventory-loss location, is classified neither incoming nor outgoing, consumes and creates no cost layer, and produces no journal entry.

*Verifies [accounting-effects.md](accounting-effects.md), section 1.*

### AC-086: A recycled part returns value to stock

- **Given** a Repair Order with one `recycle` part of a storable product, completed, with the shipped recycle destination `WH/Stock`,
- **Then** that movement runs from the production location to `WH/Stock`, is classified incoming, and creates a cost layer for the product.

*Verifies [accounting-effects.md](accounting-effects.md), section 1.*

### AC-087: The repaired-product movement is value-neutral

- **Given** a completed Repair Order whose product source location and product destination location are both `WH/Stock`,
- **Then** the repaired-product movement is classified neither incoming nor outgoing and produces no journal entry.

*Verifies [accounting-effects.md](accounting-effects.md), section 1.*

### AC-088: A repair-backed Sales Order Line reports no valued movements

- **Given** a company that recognises the cost of goods sold on the invoice, automated valuation, and a Repair Order with one `add` part of 1.00 of a storable product, confirmed, started and ended,
- **And** a quotation created from that repair and confirmed, so that the generated Sales Order Line's only movement is the repair's own `add` part movement, which is `done`,
- **When** the Sales Order Line is asked whether it has valued movements, which is the question that decides whether the costing entry is produced from the sales side,
- **Then** the answer is no, even though the movement is neither cancelled nor draft, because the movement belongs to a Repair Order,
- **And** no second costing entry is produced from the sales side for that line.

- **And given** an ordinary Sales Order Line on the same order, whose delivery movement belongs to no repair and is `done`,
- **Then** that line answers yes and is costed in the ordinary way.

*Verifies rule RM-101, the guard on the sales side, beside rule RM-100 on the invoice side verified by AC-084.*

---

# Part ten: configuration and access for repairs

### AC-089: A warehouse without a production location cannot be created

- **Given** a company whose only production-usage location has been changed to internal usage,
- **When** a Warehouse is created for that company,
- **Then** the creation is refused with the message "Can't find any production location."

*Verifies rule RM-080.*

### AC-090: A warehouse without an inventory-loss location cannot be created

- **Given** a company whose inventory-loss locations have all been changed to internal usage,
- **When** a Warehouse is created,
- **Then** the creation is refused with the message "No location of type Inventory Loss found".

*Verifies rule RM-081.*

### AC-091: Every warehouse gets a repair replenish-on-order rule

- **Given** the main company after installation,
- **When** the stock rules whose operation type carries the repair code are read for that company,
- **Then** exactly one exists, its procurement method is make to order, its action is pull, its automation is manual, its route is named "Replenish on Order (MTO)", its source location is the warehouse stock location, its destination location is the production location, its operation type is named "Repairs", and it is active.

*Verifies rule RM-082.*

### AC-092: Tag names are unique

- **Given** a Repair Tag named `Urgent`,
- **When** a second Repair Tag named `Urgent` is created,
- **Then** the creation is refused with the message "Tag name already exists!"

*Verifies rule RM-070.*

### AC-093: A product used by a repair with a foreign unit blocks a unit change

- **Given** a product whose reference unit is Units, and a Repair Order naming it with the unit Dozens,
- **When** the product's reference unit is changed,
- **Then** the change is refused with a message beginning "As other units of measure (ex : Dozens) than Units have already been used for this product";
- **And given** instead that every Repair Order naming it uses Units,
- **When** the reference unit is changed to Dozens,
- **Then** the change succeeds and those Repair Orders now carry Dozens.

*Verifies rule RM-090.*

### AC-094: A repair is visible only from its own company

- **Given** two companies A and B and a Repair Order of company A,
- **When** a user whose allowed companies are only B lists Repair Orders,
- **Then** that repair is not in the result.

*Verifies rule RM-111.*

### AC-095: The model access matrix for repair entities

- **Given** three users: `Ivan`, who holds the inventory user group; `Nora`, who holds only the internal user group and nothing this domain grants; and `Ada`, whose inventory administrator group implies the inventory user group,
- **Then** `Ivan` may read, create, change and delete Repair Orders,
- **And** `Ivan` may read, create, change and delete Repair Tags,
- **And** `Ivan` may read, create and change an Insufficient Repair Quantity Warning, but an attempt to delete one is refused, because the delete right is the single "no" in the matrix,
- **And** `Nora` may not read a Repair Order, a Repair Tag or an Insufficient Repair Quantity Warning at all,
- **And** `Ada` holds exactly the same rights on those three entities as `Ivan`, because this domain grants no group beyond the inventory user group and her extra rights come only from the implication.

*Verifies rule RM-110.*

### AC-096: Field-level visibility on the repair screens

- **Given** a user `Ivan` holding the inventory user group and none of the manufacturing user, purchase user, lot and serial number tracking, unit of measure, human resources user or multiple companies groups,
- **When** `Ivan` opens a Repair Order form,
- **Then** the manufacturing counter is not shown, the purchase counter is not shown, the lot or serial number field is not shown, and the parts list shows no unit column,
- **And** the printed Repair Order produced for `Ivan` omits the serial number line.

- **And given** `Ivan` is granted the manufacturing user group, **then** the manufacturing counter appears on the Repair Order form.
- **And given** `Ivan` is granted the purchase user group, **then** the purchase counter appears on the Repair Order form.
- **And given** `Ivan` is granted the lot and serial number tracking group, **then** the lot or serial number field appears on the form and the serial number appears on the printed document.
- **And given** `Ivan` is granted the unit of measure group, **then** the unit columns appear on the parts list and on the printed document.
- **And given** a user holding the human resources user group opens an Employee, **then** the equipment collection of that employee is shown to them, and it is not shown to a reader without that group.
- **And given** a user who does not hold the inventory user group opens a Sales Order, a Manufacturing Order or a Purchase Order, **then** no repair counter and no repair button appears on any of the three.
- **And given** a user who does not hold the equipment manager group opens an Equipment, **then** the cost field is not shown; opening a Maintenance Request, the category field is not shown.

*Verifies rule RM-112.*

### AC-097: The repairs menu appears according to the reader's groups

- **Given** `Ivan`, who holds the inventory user group but not the inventory administrator group and not the developer visibility group,
- **Then** `Ivan` sees the Repairs top-level menu and its Orders entry,
- **And** `Ivan` sees neither the Reporting section nor the Configuration section under Repairs.

- **And given** `Ada`, who holds the inventory administrator group,
- **Then** `Ada` additionally sees the Reporting section with its Repairs entry, which opens the Repair Orders Analysis screen, and the Configuration section with its Products entry,
- **And** `Ada` sees the Product Variants entry only when she also holds the product variants group,
- **And** `Ada` does not see the Repair Orders Tags entry, which is a developer-only entry.

- **And given** `Dev`, who holds the inventory administrator group and the developer visibility group,
- **Then** `Dev` additionally sees the Repair Orders Tags entry under Configuration.

- **And given** `Nora`, who holds neither the inventory user group nor the inventory administrator group,
- **Then** the Repairs top-level menu is not shown to her at all.

*Verifies rule RM-113 and the Repairs menu table of [configuration.md](configuration.md), section 5.1.*

---

# Part eleven: equipment

### AC-098: An equipment display name includes the serial number

- **Given** an Equipment named `Samsung Monitor 15` with the serial number `MT/127/18291015`,
- **Then** its display name is `Samsung Monitor 15/MT/127/18291015`;
- **And given** the serial number is cleared,
- **Then** its display name is `Samsung Monitor 15`.

*Verifies rule RM-202 and workflow 27.*

### AC-099: Serial numbers of equipment are unique

- **Given** an Equipment with the serial number `SN-9`,
- **When** a second Equipment is created with the serial number `SN-9`,
- **Then** the creation is refused with the message "Another asset already exists with this serial number!"

*Verifies rule RM-201.*

### AC-100: Duplicating an equipment drops its serial number

- **Given** an Equipment with the serial number `SN-9`,
- **When** it is duplicated,
- **Then** the copy has no serial number and the uniqueness constraint is not violated.

*Verifies the duplication rules of [entities.md](entities.md), section 9.4.*

### AC-101: Choosing a category replaces the technician

- **Given** an Equipment whose technician is the user `Alice`, and a category `Monitors - Test` whose responsible user is `Bob`,
- **When** the category `Monitors - Test` is chosen on the equipment form,
- **Then** the equipment's technician is `Bob`.

*Verifies rule RM-203.*

### AC-102: Assignment exclusivity and the assignment date

- **Given** an Equipment with the people bridge installed, with an employee and a department both set,
- **When** the Used By choice is set to Employee,
- **Then** the department is cleared, the employee is kept, and the assigned date is today;
- **When** it is set to Department,
- **Then** the employee is cleared, the department is kept, and the assigned date is today;
- **When** it is set to Other,
- **Then** both are kept and the assigned date is today.

*Verifies rules RM-204 and RM-205.*

### AC-103: Owner derivation

- **Given** the people bridge, an employee `Carol` whose user is `carol`, and a department `Support` whose manager's user is `dave`,
- **When** an Equipment is set to Used By Employee with the employee `Carol`,
- **Then** its owner is `carol`;
- **When** it is set to Used By Department with the department `Support`,
- **Then** its owner is `dave`;
- **When** it is set to Used By Other,
- **Then** its owner is the acting user;
- **And given** the employee `Carol` has no user account,
- **Then** the owner is empty.

*Verifies rule RM-206.*

### AC-104: Assigning an equipment subscribes and announces

- **Given** an Equipment with no owner,
- **When** an owner is set,
- **Then** that user's contact is among the followers and a message under the "Equipment Assigned" subtype has been posted,
- **And** the previous holder, when there was one, is still a follower, because subscribing never unsubscribes.

*Verifies rules RM-207 and RM-208.*

### AC-105: A manager creates an equipment and an ordinary user can open it

- **Given** an Equipment Manager and an ordinary Internal User,
- **When** the manager creates an Equipment named `Super Equipment` and sets its owner to the ordinary user,
- **Then** the ordinary user can open that equipment and read its name as `Super Equipment`.

*Verifies rule RM-252 together with rule RM-207.*

### AC-106: An ordinary user cannot create an equipment category

- **Given** an ordinary Internal User,
- **When** that user attempts to create an Equipment Category,
- **Then** the attempt is refused with an access error.

*Verifies rule RM-250 and workflow 26.*

### AC-107: A category in use cannot be deleted

- **Given** an Equipment Category holding one Equipment,
- **When** the category is deleted,
- **Then** the deletion is refused with the message "You can’t delete an equipment category if some equipment or maintenance requests are linked to it."

*Verifies rule RM-200.*

### AC-108: An empty category is folded and a filled one is not

- **Given** a newly created Equipment Category,
- **Then** its folding flag is true;
- **When** an Equipment of that category is created,
- **Then** its folding flag is false;
- **And given** every equipment of a category is archived,
- **Then** its folding flag is true again, because archived equipment is not counted.

*Verifies calculation 26 and rule RM-241.*

### AC-109: Serial matching requires the tracking group

- **Given** the inventory bridge, an Equipment with the serial number `SN-42`, and a Lot or Serial Number named `SN-42`,
- **When** the equipment is read by a user holding the lot and serial number tracking group and able to read lots,
- **Then** its serial-match flag is true and the Serial Number button is shown;
- **When** it is read by a user without that group,
- **Then** its serial-match flag is false and the button is not shown.

*Verifies rule RM-209 and workflow 28.*

### AC-110: Equipment used by a request cannot be deleted

- **Given** an Equipment named by one Maintenance Request,
- **When** the equipment is deleted,
- **Then** the deletion is refused by the referential guard,
- **And** the refusal also applies when the only request naming it is archived.

*Verifies rule RM-210.*

---

# Part twelve: maintenance requests

### AC-111: A new request lands in the first stage

- **Given** the four shipped stages,
- **When** an Internal User creates a Maintenance Request named `Resolution is bad` for an equipment, with no stage supplied,
- **Then** its stage is "New Request", its within-stage signal is In Progress, its archive flag is false, its kind is Corrective and its request date is today.

*Verifies workflow 29 and transition T-30.*

### AC-112: A stage change is applied

- **Given** the request of the previous scenario in "New Request",
- **When** the same user writes the stage "In Progress",
- **Then** its stage is "In Progress".

*Verifies workflow 31 and transition T-32.*

### AC-113: Reaching a closing stage stamps the close date

- **Given** a corrective Maintenance Request in "New Request" with a request date and no close date,
- **When** its stage is set to "Repaired", which carries the closing flag,
- **Then** its close date is today and its request date is unchanged.

*Verifies rule RM-225 and transition T-33.*

### AC-114: Returning to a non-closing stage clears the close date

- **Given** the request of the previous scenario,
- **When** its stage is set back to "In Progress",
- **Then** its close date is empty.

*Verifies rule RM-225 and transition T-32.*

### AC-115: A stage change resets the within-stage signal

- **Given** a Maintenance Request whose within-stage signal is Blocked,
- **When** its stage is changed without supplying a signal,
- **Then** its signal is In Progress;
- **And when** its stage and the signal Blocked are written in the same operation,
- **Then** its signal is Blocked.

*Verifies rule RM-226, workflow 32 and transition T-43.*

### AC-116: Several requests can be written at once

- **Given** two preventive Maintenance Requests, both with the signal In Progress,
- **When** both are written in one operation with the signal Blocked and the stage "New Request",
- **Then** both carry the signal Blocked and the stage "New Request".

*Verifies rule RM-226 on a set.*

### AC-117: An equipment with a closed request that lost its dates can still be read

- **Given** an Equipment with one corrective Maintenance Request in a closing stage,
- **When** the request's close date is forced empty, then its request date is forced empty, then both,
- **Then** the equipment record can be opened in each of the three situations without error, and its measurements are produced under the missing-date rules.

*Verifies calculations 20 and 21, edge cases.*

### AC-118: Cancelling a request hides it and ends its series

- **Given** a recurrent preventive Maintenance Request,
- **When** the user cancels it,
- **Then** its archive flag is true and its recurrence flag is false;
- **And when** it is then moved into "Repaired",
- **Then** no successor is created.

*Verifies rule RM-228, workflow 35 and transition T-50.*

### AC-119: Reopening returns a request to the first stage

- **Given** an archived Maintenance Request sitting in "Repaired",
- **When** the user reopens it,
- **Then** its archive flag is false, its stage is "New Request", its close date is empty and its within-stage signal is In Progress.

*Verifies rule RM-229 and transitions T-34 and T-51.*

### AC-120: An end earlier than the start is refused

- **Given** a Maintenance Request scheduled to start on 3 March 2025 at 09:00,
- **When** the scheduled end is set to 3 March 2025 at 08:00,
- **Then** the write is refused with the message "End date cannot be earlier than start date.";
- **And when** the scheduled end is set equal to the start,
- **Then** the write is accepted and the duration is 0.00.

*Verifies rule RM-220.*

### AC-121: An interval below one is refused

- **Given** a Maintenance Request,
- **When** its repeat interval is set to 0,
- **Then** the write is refused with the message "The repeat interval cannot be less than 1.",
- **And** the same refusal applies to a request that is not recurrent.

*Verifies rule RM-221.*

### AC-122: The scheduled end and the duration follow the start

- **Given** a Maintenance Request with no schedule,
- **When** the scheduled start is set to 3 March 2025 at 09:00,
- **Then** the scheduled end is 3 March 2025 at 10:00 and the duration is 1.00;
- **When** the end is changed to 3 March 2025 at 11:30,
- **Then** the duration is 2.50;
- **When** the end is changed to 3 March 2025 at 09:50,
- **Then** the duration is 0.83;
- **When** the start is then moved to 17 March 2025 at 08:00,
- **Then** the end is overwritten to 17 March 2025 at 09:00 and the duration falls back to 1.00.

*Verifies calculations 28 and 29.*

### AC-123: A corrective request cannot recur

- **Given** a preventive Maintenance Request whose recurrence flag is true,
- **When** its kind is changed to Corrective,
- **Then** its recurrence flag is false.

*Verifies rule RM-222.*

### AC-124: The technician is derived from the equipment, then from the category

- **Given** an equipment whose technician is `Alice` and whose category's responsible user is `Bob`,
- **When** a Maintenance Request names that equipment,
- **Then** its technician is `Alice`;
- **And given** the equipment has no technician,
- **Then** the request's technician is `Bob`;
- **And given** the resulting user's allowed companies do not include the request's company,
- **Then** the request's technician is empty.

*Verifies rule RM-237.*

### AC-125: The team is derived from the equipment and cleared on a company mismatch

- **Given** an equipment whose team is `Metrology`, belonging to company A,
- **When** a Maintenance Request of company A names that equipment,
- **Then** its team is `Metrology`;
- **And given** the request's company is B,
- **Then** its team is empty.

*Verifies rule RM-236.*

### AC-126: A bounded series needs an end date, and recurrence needs its three settings

- **Given** a preventive Maintenance Request `Descale the boiler` on which the recurrence flag is set to true,
- **Then** the repeat interval, the repeat unit and the repeat kind are all required, so none of the three may be left empty,
- **And when** the repeat kind is set to `until` while the end date is left empty,
- **Then** the record cannot be saved: the end date is required as soon as the series is declared bounded,
- **And when** an end date of 30 June 2025 is supplied,
- **Then** the record saves and the series will stop after the last occurrence whose planned start falls on or before 30 June 2025.

- **And given** the repeat kind is set back to `forever`,
- **Then** the end date is no longer required and may be cleared, and the record saves.

- **And given** the recurrence flag is set back to false, which the system also forces whenever the maintenance kind is not preventive,
- **Then** none of the four recurrence fields is required any more.

*Verifies rule RM-223, beside rule RM-222 verified by AC-123, and calculation 30.*

---

# Part thirteen: recurrence

### AC-127: A forever series creates its successor in the first stage

- **Given** a Maintenance Request named `Test forever maintenance`, preventive, recurrent, with the repeat kind Forever,
- **When** its stage is set to a stage carrying the closing flag,
- **Then** a second Maintenance Request named `Test forever maintenance` exists in the stage with the lowest sequence.

*Verifies rule RM-224, workflow 33 and transition T-33.*

### AC-128: The successor of a thirty-day series

- **Given** a preventive recurrent Maintenance Request `Descale the boiler`, repeating every 30 days, repeat kind Forever, scheduled from 3 March 2025 at 09:00 to 3 March 2025 at 11:30, therefore a duration of 2.50,
- **When** it is moved into the "Repaired" stage,
- **Then** a successor exists in "New Request" scheduled from 2 April 2025 at 09:00 to 2 April 2025 at 11:30, with the same subject, equipment, team, technician, priority, instructions and recurrence settings, and with no close date;
- **And** the original sits in "Repaired" with its close date set to today, its within-stage signal reset to In Progress and no open activity.

*Verifies calculation 30, its first worked example.*

### AC-129: A bounded series stops

- **Given** the same request with the repeat kind Until and an end date of 1 April 2025,
- **When** it is moved into the "Repaired" stage,
- **Then** no successor is created.

*Verifies calculation 30, its bounded example.*

### AC-130: A monthly series clamps to the end of a short month

- **Given** a preventive recurrent Maintenance Request scheduled on 31 January 2025 at 08:00, with a duration of 0, repeating every 1 month, Forever,
- **When** it is moved into a closing stage,
- **Then** the successor is scheduled from 28 February 2025 at 08:00 to 28 February 2025 at 09:00.

*Verifies calculation 30, its monthly example.*

### AC-131: A request with no schedule recurs from the current moment

- **Given** a preventive recurrent Maintenance Request with no scheduled date, repeating every 2 weeks, Forever, moved into a closing stage at 11 September 2025 14:22,
- **Then** the successor is scheduled from 25 September 2025 at 14:22 to 25 September 2025 at 15:22.

*Verifies calculation 30, its no-schedule example.*

### AC-132: Recurrence produces no duplicate activity

- **Given** a preventive recurrent Maintenance Request scheduled today, carrying exactly one maintenance activity,
- **When** it is moved into a stage carrying the closing flag,
- **Then** exactly one successor exists, the successor carries exactly one maintenance activity, and the original carries none.

*Verifies rule RM-227 and workflow 33.*

---

# Part fourteen: activities

### AC-133: The activity deadline uses the user's time zone

- **Given** a user whose time zone runs thirteen hours ahead of coordinated universal time,
- **When** that user creates a Maintenance Request scheduled at 08:00 on 11 January local time, which is 19:00 on 10 January in coordinated universal time,
- **Then** the maintenance activity of that request has a deadline of 11 January.

*Verifies calculation 31 and workflow 34.*

### AC-134: A request with no schedule carries no activity

- **Given** a Maintenance Request with a scheduled date and one maintenance activity,
- **When** the scheduled date is cleared,
- **Then** the request carries no maintenance activity.

*Verifies rule RM-227.*

### AC-135: Changing the equipment recreates the activity

- **Given** a Maintenance Request with a scheduled date and one maintenance activity whose note names the equipment `Drill A`,
- **When** the equipment is changed to `Drill B`,
- **Then** the request still carries exactly one maintenance activity, and its note names `Drill B`.

*Verifies rule RM-227.*

### AC-136: The activity is aimed at the technician

- **Given** a Maintenance Request with the technician `Alice` and the created-by user `Bob`, scheduled,
- **Then** the activity is assigned to `Alice`;
- **And given** the request has no technician,
- **Then** the activity is assigned to `Bob`;
- **And given** it has neither,
- **Then** the activity is assigned to the acting user.

*Verifies calculation 31.*

---

# Part fifteen: effectiveness measurements

### AC-137: Mean time between failures from three failures

- **Given** an Equipment with an effective date of 1 January 2025,
- **And** three corrective Maintenance Requests against it, all in a closing stage, with the request dates 10 February 2025, 11 April 2025 and 10 June 2025 and the close dates 12 February 2025, 14 April 2025 and 13 June 2025,
- **Then** its latest failure date is 10 June 2025,
- **And** its mean time between failures is 53 days, from 160 days divided by 3, truncated from 53.3333,
- **And** its estimated next failure is 2 August 2025,
- **And** its mean time to repair is 2 days, from 8 days divided by 3, truncated from 2.6666,
- **And** given a user typed 60 into the expected mean time between failures, that field still reads 60: it is a target entered by hand, never recomputed from the failures, and nothing derives from it.

*Verifies calculations 18, 19, 20, 21, 22 and 23.*

### AC-138: A fourth failure lowers the mean

- **Given** the equipment of the previous scenario plus a fourth closed corrective request dated 31 July 2025,
- **Then** its mean time between failures is 52 days, from 211 days divided by 4, truncated from 52.75.

*Verifies calculation 20.*

### AC-139: Preventive requests do not feed the measurements

- **Given** an Equipment with an effective date of 1 January 2025 and one preventive Maintenance Request in a closing stage dated 10 June 2025,
- **Then** its latest failure date is empty, its mean time between failures is 0, its mean time to repair is 0 and its estimated next failure is empty.

*Verifies rule RM-238 and calculation 18.*

### AC-140: An open corrective request does not feed the measurements

- **Given** an Equipment with one corrective Maintenance Request sitting in "In Progress",
- **Then** all four measurements read as if there were no request at all;
- **And given** a corrective request in a closing stage that has been archived,
- **Then** it **does** count, because the archive flag is not tested.

*Verifies rule RM-238 and calculation 18.*

### AC-141: A missing close date pulls the mean time to repair down

- **Given** the equipment of scenario AC-137 where the third request's close date has been forced empty,
- **Then** its mean time to repair is 1 day, from 5 days divided by 3, truncated from 1.6666.

*Verifies calculation 21, edge case.*

### AC-142: A zero mean time between failures produces no estimate

- **Given** an Equipment whose effective date equals the request date of its single closed corrective request,
- **Then** its mean time between failures is 0 and its estimated next failure is empty.

*Verifies calculation 22.*

### AC-143: Item counters against category counters

- **Given** an Equipment of the category `Monitors` with five requests: two in "New Request", one in "In Progress", one in "Repaired" and one in "In Progress" that is archived,
- **Then** the equipment's total count is 5 and its open count is 3,
- **And** the category's total count is 5 and its open count is 4.

*Verifies calculations 24 and 25, and rule RM-240.*

---

# Part sixteen: teams, aliases and departures

### AC-144: The team dashboard counters

- **Given** a Maintenance Team with seven requests that are neither in a closing stage nor archived, of which five carry a scheduled date, two of those five carry the High priority, and one of the two unscheduled ones is Blocked,
- **Then** the team's to-do count is 7, its scheduled count is 5, its high-priority count is 2, its blocked count is 1 and its unscheduled count is 2.

*Verifies calculation 27 and rule RM-242.*

### AC-145: A request raised by electronic mail lands on the team that owns the alias

- **Given** a Maintenance Team `Metrology` with a configured alias,
- **When** a message is sent to that alias address with the subject `Drill not working`,
- **Then** a Maintenance Request named `Drill not working` exists, its team is `Metrology`, its stage is the one with the lowest sequence, and the message body is the first message of its thread,
- **And** the carbon-copy addresses of the inbound message are retained on the request.

*Verifies workflow 30 and rule RM-239.*

### AC-146: The default team of a new request

- **Given** two teams, `Metrology` of company A and `Subcontractor` of company B, and a user acting in company A,
- **When** that user creates a Maintenance Request without choosing a team,
- **Then** the request's team is `Metrology`;
- **And given** company A owns no team,
- **Then** the request's team is the first team of any company.

*Verifies rule RM-235.*

### AC-147: A departure frees the equipment

- **Given** the people bridge, an employee `Carol` holding two pieces of Equipment,
- **When** the departure of `Carol` is registered with the Free Equiments option ticked,
- **Then** neither Equipment names `Carol` any longer, both still exist, neither is archived, and the owner of each is empty;
- **And given** the option is unticked,
- **Then** both still name `Carol`.

*Verifies rule RM-260 and workflow 36.*

---

# Part seventeen: multi-company maintenance

### AC-148: An equipment manager works across the companies they are allowed

- **Given** two companies A and B, an Equipment Manager allowed in both, and an ordinary Internal User allowed only in B,
- **When** the manager creates a Maintenance Team in A and another in B, and creates two Equipment Categories in B,
- **Then** all four records are created;
- **And when** the ordinary user attempts to create an Equipment Category in B,
- **Then** the attempt is refused with an access error.

*Verifies rules RM-250, RM-253 and RM-254.*

### AC-149: A record with no company is visible from every company

- **Given** the shipped team "Internal Maintenance", which has no company,
- **When** it is read by a user allowed only in company B,
- **Then** it is in the result.

*Verifies rule RM-254.*

### AC-150: An ordinary user sees only their own requests

- **Given** an ordinary Internal User `Erin` and three Maintenance Requests: one whose created-by user is `Erin`, one whose technician is `Erin`, and one that neither names nor is followed by `Erin`,
- **When** `Erin` lists Maintenance Requests,
- **Then** the first two are in the result and the third is not.

*Verifies rule RM-251.*

### AC-151: An ordinary user sees only the equipment they follow

- **Given** an ordinary Internal User `Erin`, an Equipment she follows and an Equipment she does not,
- **When** `Erin` lists Equipment,
- **Then** only the first is in the result.

*Verifies rule RM-252.*

---

# Part eighteen: printed document and screens

### AC-152: The printed repair order lists every part with its kind

- **Given** a Repair Order `WH/RO/00007` for the customer Wood Corner, product `Espresso machine`, serial number `ESP-0042`, state Repaired, responsible `Alice`, with the three parts of scenario AC-026 and the internal note `Replace the gasket kit next service`,
- **When** the printed Repair Order is produced,
- **Then** the document heading reads "Repair Order #" followed by `WH/RO/00007`, the information block names the customer, the product, the serial number for a reader holding the lot and serial number tracking group, the status and the responsible user,
- **And** the Parts table holds three rows reading "(Add) Group gasket" with the quantity 2, "(Add) Water pump" with the quantity 1 and "(Remove) Water pump" with the quantity 1,
- **And** the Repair Notes section reproduces the internal note,
- **And** no price, cost or total appears anywhere in the document.

*Verifies [interfaces.md](interfaces.md), section 9, and workflow 24.*

### AC-153: The printed repair order is rendered in the customer's language

- **Given** a Repair Order whose customer's language differs from the acting user's,
- **When** the printed Repair Order is produced,
- **Then** it is rendered in the customer's language and the file is named "Repair Order - " followed by the reference.

*Verifies [interfaces.md](interfaces.md), section 9, and workflow 24.*

### AC-154: Unsetting the product on a part keeps the incomplete flag correct

- **Given** a Repair Order with one `add` part for a product, demanding 1.0 and with a recorded quantity of 0.0,
- **When** the product is unset on that part and then set again through the form,
- **Then** the repair still reports incomplete parts.

*Verifies calculation 5 through the form.*

### AC-155: Changing the responsible after confirmation does not move the locations

- **Given** a Repair Order whose component source location and recycled-parts destination location were both changed by hand to `WH/Shelf 2`, then confirmed,
- **When** the responsible user is changed,
- **Then** both locations are still `WH/Shelf 2`;
- **And when** the repair is started, ended, and the responsible user changed again through the form,
- **Then** both locations are still `WH/Shelf 2`.

*Verifies that the location derivations depend on the operation type alone.*

### AC-156: The transfer form shows its repairs

- **Given** a validated return transfer from which two Repair Orders were created,
- **Then** the transfer's repair counter reads 2, and the action opens the list of the two repairs rather than a single form.

*Verifies [interfaces.md](interfaces.md), section 2, and workflow 25 step 2.*

---

# Part nineteen: the maintenance calendar

### AC-157: A recurrent request projects its future occurrences onto the calendar

- **Given** the maintenance calendar showing March 2025, so the displayed range runs from 1 March 2025 00:00 to 31 March 2025 23:59:59,
- **And** a preventive Maintenance Request `Clean the room` in the "New Request" stage, not cancelled, with the recurrence flag true, the scheduled date 6 March 2025 10:00, the scheduled end 6 March 2025 12:00, the duration 2.00, the repeat interval 1, the repeat unit weeks, the repeat kind `until` and the end date 27 March 2025,
- **Then** the calendar paints four events for that request: the real event on 6 March from 10:00 to 12:00, and three projected events labelled `Clean the room (+1)`, `Clean the room (+2)` and `Clean the room (+3)`, on 13, 20 and 27 March, each running from 10:00 to 12:00,
- **And** no fifth event is painted, because the next step would fall on 3 April 2025, past the end of the day of 27 March 2025,
- **And** exactly one Maintenance Request record named `Clean the room` exists: the three projections wrote nothing.

*Verifies calculation 32 and the calendar of [interfaces.md](interfaces.md), section 6.4.*

### AC-158: A projected occurrence is locked, the real event moves the request

- **Given** the calendar and the request of scenario AC-157,
- **When** the user drags the projected event `Clean the room (+2)` from 20 March to 19 March,
- **Then** nothing is written: the request still reads the scheduled date 6 March 2025 10:00, the scheduled end 6 March 2025 12:00 and the duration 2.00, and the calendar redraws the occurrence on 20 March, where the projection puts it,
- **And** resizing that projected event has no effect either.

- **And when** the user instead drags the real event from 6 March 2025 10:00 to 15 March 2025 10:00,
- **Then** the request reads the scheduled date 15 March 2025 10:00 and the scheduled end 15 March 2025 12:00, because the drag writes both bounds, and the duration is still 2.00,
- **And** the projections are recomputed from the new start: a single projected event `Clean the room (+1)` on 22 March from 10:00 to 12:00, because the next step, 29 March, is past the end of the day of the end date.

*Verifies calculation 32, calculation 29 and the drag rule of [interfaces.md](interfaces.md), section 6.4.*

### AC-159: Opening a projected occurrence opens the one real request

- **Given** three Maintenance Requests: `Send the mails` scheduled 24 February 2025 09:00, `Wash the car` scheduled 31 March 2025 09:00, and `Clean the room`, preventive and recurrent, on the equipment `Room`, scheduled 10 March 2025 09:00, repeat interval 1, repeat unit days, repeat kind `until`, end date 18 March 2025,
- **And** the calendar showing March 2025,
- **Then** `Clean the room` paints one real event on 10 March and eight projected events labelled `Clean the room (+1)` through `Clean the room (+8)`, on the eight days from 11 to 18 March,
- **When** the user double-clicks the projected event `Clean the room (+3)` on 13 March,
- **Then** the form of the single `Clean the room` Maintenance Request opens: the same record, not a copy and not a blank form,
- **And when** the subject is changed to `Make your bed` and the scheduled end is moved to 10 March 2025 11:00, and the form is saved,
- **Then** that one request reads the subject `Make your bed` and the duration 2.00,
- **And** the eight projections are relabelled `Make your bed (+1)` through `Make your bed (+8)` and each now runs for two hours,
- **And** exactly three Maintenance Requests still exist.

- **And when** the user instead uses the Edit control, or the Delete control, of the summary bubble of a projected event, or clicks the occurrence in the year shape of the calendar,
- **Then** each of those acts on the same single real request.

*Verifies calculation 32, calculation 29 and the opening rule of [interfaces.md](interfaces.md), section 6.4.*

### AC-160: A request stops projecting once it is closed or cancelled

- **Given** the calendar and the request of scenario AC-157,
- **When** the request is moved into the "Repaired" stage, which carries the closing flag,
- **Then** the calendar paints no projected event for it any more, only its real event on 6 March,
- **And** exactly one successor Maintenance Request has been created in the "New Request" stage, scheduled 13 March 2025 10:00 to 12:00, carrying the same recurrence settings,
- **And** the successor projects in turn: `Clean the room (+1)` on 20 March and `Clean the room (+2)` on 27 March, so the month still shows three future occurrences, one of which is now a real record.

- **And given** instead that the request is cancelled, which sets the archive flag to true,
- **Then** it projects nothing, and with the Active filter of the calendar entry applied it is not painted at all.

*Verifies calculation 32, calculation 30, and rules RM-224 and RM-228.*

### AC-161: A request scheduled before the displayed range still projects into it

- **Given** a preventive Maintenance Request `Replace the filters`, recurrent, with the scheduled date 8 January 2025 08:00, the duration 0.00, the repeat interval 1, the repeat unit months, the repeat kind `forever`, in a stage that does not close and not cancelled,
- **When** the calendar shows April 2025, so the displayed range runs from 1 April 2025 00:00 to 30 April 2025 23:59:59,
- **Then** the request is still fetched, because the fetch condition is that the scheduled date is not later than the end of the displayed range, with no lower bound,
- **And** its real event on 8 January is outside April and is not painted,
- **And** exactly one event is painted for it: the projected `Replace the filters (+3)` on 8 April from 08:00 to 09:00, the one-hour length coming from the zero duration,
- **And** the label reads three and not one, because the steps to 8 February and to 8 March were counted even though they fell before the displayed range and were not painted.

- **And given** the same request with the scheduled date 31 January 2025 09:00 and the duration 1.00, shown over January to June 2025,
- **Then** the projected starts are 28 February, 28 March, 28 April, 28 May and 28 June 2025 at 09:00, because each step is taken from the previous clamped moment and the series never returns to the thirty-first or the thirtieth.

*Verifies calculation 32.*

---

# Part twenty: maintenance access, menus and field visibility

### AC-162: A human resources officer manages equipment without a separate grant

- **Given** the people bridge installed, and a user `Fatima` who holds the human resources user group and no maintenance group granted directly,
- **Then** `Fatima` holds the equipment manager group through the group implication,
- **And** she may read, create, change and delete Equipment, Equipment Category, Maintenance Stage and Maintenance Team,
- **And** the cost field is shown to her on the Equipment form and the category field on a Maintenance Request.

- **And given** a user `Erin` who holds only the internal user group,
- **Then** `Erin` may read Equipment, Equipment Category, Maintenance Stage and Maintenance Team but may not create, change or delete any of them,
- **And** she may read, create, change and delete Maintenance Requests, which is the one maintenance entity the internal user group is given in full,
- **And** the cost field is not shown to her.

*Verifies rules RM-255, RM-250 and RM-257.*

### AC-163: The maintenance menus appear according to the reader's groups

- **Given** `Erin`, who holds only the internal user group,
- **Then** she sees the Maintenance top-level menu, the Dashboard entry, the Maintenance section with its Maintenance Requests and Maintenance Calendar entries, the Equipment entry, the first Reporting heading with its two headings, and the second Reporting heading with the Maintenance Requests Analysis entry,
- **And** she does not see the Configuration section.

- **And given** `Fatima`, who holds the equipment manager group,
- **Then** she additionally sees the Configuration section, with the Maintenance Teams entry and the Equipment Categories entry,
- **And** she does not see the Settings entry, which needs the settings administration group,
- **And** she does not see the Maintenance Stages entry nor the Activity Types entry, which are developer-only entries.

- **And given** `Sam`, who holds the equipment manager group and the settings administration group,
- **Then** he additionally sees the Settings entry, which opens the Maintenance section of the settings screen.

- **And given** `Dev`, who holds the equipment manager group and the developer visibility group,
- **Then** he additionally sees the Maintenance Stages entry and the Activity Types entry.

- **And given** an account that holds neither the equipment manager group nor the internal user group,
- **Then** the Dashboard, the Maintenance section, the Equipment entry, the first Reporting heading and the Configuration section are all hidden, so nothing of the maintenance menu is usable.

*Verifies rule RM-256 and the Maintenance menu table of [configuration.md](configuration.md), section 5.2.*

### AC-164: Field-level visibility in maintenance

- **Given** `Erin`, who holds only the internal user group, works in a single company and does not hold the developer visibility group,
- **When** she opens an Equipment form,
- **Then** the cost field is not shown, the assigned date is not shown, the scrap date is not shown, and the company field is not shown,
- **And when** she opens a Maintenance Request,
- **Then** the category field is not shown on the form, not offered as a column in the list, and not offered in the search panel, and the carbon-copy address field is not shown.

- **And given** `Fatima`, who holds the equipment manager group,
- **Then** the cost field and the category field are shown to her in all three places, while the assigned date, the scrap date and the carbon-copy address stay hidden, because those are developer-only fields rather than manager fields.

- **And given** `Dev`, who holds the equipment manager group and the developer visibility group,
- **Then** the assigned date, the scrap date and the carbon-copy address field are shown to him as well.

- **And given** `Mia`, who holds the equipment manager group and the multiple companies group,
- **Then** the company fields are shown to her on Equipment, Equipment Category, Maintenance Request and Maintenance Team.

*Verifies rule RM-257.*

---

# Part twenty-one: the point-of-sale bridge

### AC-165: A repair-backed Sales Order Line is flagged for the point of sale

- **Given** the point-of-sale bridge installed, a product `Test product 1`, and a Repair Order for `Test product 1` under the warehouse repair operation type, for the customer `Partner 1`, with one `add` part of 1.00 of `Test product 1`,
- **When** the repair is confirmed, started and ended, and a quotation is created from it,
- **Then** exactly two Stock Moves exist for `Test product 1`: the `add` part movement and the repaired-product movement,
- **And** the generated Sales Order Line for the part reports that it is backed by a repair, because at least one of its movements belongs to a Repair Order,
- **And** that flag is among the fields loaded into the point-of-sale screen data for Sales Order Lines.

- **And given** a line on another Sales Order whose only movement is an ordinary delivery movement belonging to no repair,
- **Then** that line reports the flag as false.

*Verifies the repair-backed flag of [entities.md](entities.md), section 7.4, and the point-of-sale package of [configuration.md](configuration.md), section 1.*

### AC-166: Settling a repair-backed order in the point of sale creates no new movement

- **Given** the state reached at the end of scenario AC-165, with exactly two Stock Moves for `Test product 1`,
- **When** a point-of-sale session is opened, the Sales Order is settled into a point-of-sale order and the order is paid,
- **Then** exactly two Stock Moves still exist for `Test product 1`: paying in the point of sale created none,
- **And** the delivery built for the point-of-sale order was built from the lines that are **not** repair-backed, the repair-backed line having been skipped, because its goods already moved through the Repair Order.

*Verifies rule RM-047 for the point-of-sale branch.*

### AC-167: A repair-backed line settles at its whole ordered quantity

- **Given** the Sales Order of scenario AC-165, whose repair-backed line orders 1.00 of `Test product 1`,
- **When** the line is settled into a point-of-sale order,
- **Then** the point-of-sale line carries the quantity 1.00, taken from the line's ordered quantity as it stands,
- **And** the ordinary rule, which would offer only the part of the ordered quantity that has not been delivered yet, is not applied to that line,
- **And** an ordinary, non-repair-backed line on the same order still settles under the ordinary rule.

*Verifies rule RM-049.*

---

# Part twenty-two: field domains, required values and configuration guards

### AC-168: The required fields of a Repair Order

- **Given** a blank Repair Order form,
- **When** the user tries to save with any one of the reference, the company, the state, the scheduled date, the operation type, the component source location, the product source location, the product destination location, the added-parts destination location, the removed-parts destination location or the recycled-parts destination location left empty,
- **Then** the save is refused and the empty field is reported as required,
- **And when** all eleven carry a value while the product to repair, the customer, the lot and the responsible user are all empty,
- **Then** the record saves, because those four are optional.

- **And given** a part line on the repair,
- **When** the user tries to save it without a part kind, or without a product,
- **Then** the save is refused, because both are required on every part.

*Verifies rule RM-006.*

### AC-169: Only a repair operation type of the repair's company may be chosen

- **Given** a company `Alpha` owning one repair operation type "Repairs" and one delivery operation type "Delivery Orders", and a company `Beta` owning its own repair operation type "Repairs",
- **When** a user creating a Repair Order in company `Alpha` opens the operation type field,
- **Then** only the "Repairs" type of `Alpha` is offered: the delivery type is excluded because its code is not `repair_operation`, and the `Beta` repair type is excluded because it belongs to another company,
- **And when** the `Beta` repair type is nevertheless written onto the repair by an integration,
- **Then** the write is refused by the company-consistency check.

*Verifies rule RM-004.*

### AC-170: Only goods, of the right company, and present on the return transfer, may be repaired

- **Given** a service product `Repair service`, a storable product `Espresso machine` belonging to no company, and a storable product `Boiler` belonging to company `Beta`,
- **When** a user creating a Repair Order in company `Alpha` opens the product field,
- **Then** `Espresso machine` is offered, `Repair service` is not offered because a service cannot be repaired, and `Boiler` is not offered because it belongs to another company.

- **And given** the repair is bound to a return transfer that moved only `Espresso machine`,
- **Then** `Espresso machine` is preselected on the repair, because the transfer moved exactly one product,
- **And** no product other than the ones that transfer moved is offered.

*Verifies rule RM-007.*

### AC-171: Only a return transfer may source a repair

- **Given** an ordinary outgoing transfer `WH/OUT/00012`, its validated return `WH/IN/00030`, and an ordinary receipt `WH/IN/00031` that is the return of nothing,
- **When** a user opens the return transfer field of a Repair Order,
- **Then** `WH/IN/00030` is offered and neither `WH/OUT/00012` nor `WH/IN/00031` is,
- **And given** the repair already names the product `Espresso machine`,
- **Then** only returns that moved `Espresso machine` are offered.

*Verifies rule RM-008.*

### AC-172: Only units compatible with the product may be used

- **Given** a product `Espresso machine` whose reference unit is Units, which also permits Dozens, and whose vendor price entry quotes it in Pallets,
- **When** a user opens the unit field of a Repair Order naming that product,
- **Then** Units, Dozens and Pallets are offered,
- **And** a unit belonging to none of those three sets, for example Kilograms, is not offered.

*Verifies rule RM-009.*

### AC-173: Company consistency across every reference of a repair

- **Given** a Repair Order of company `Alpha`,
- **When** any of the customer, the responsible user, the product to repair, the lot, the operation type, the six locations, the part movements, the repaired-product movement, the Sales Order, the Sales Order Line or the return transfer is set to a record belonging to company `Beta`,
- **Then** the write is refused with the platform's company-consistency error, which names the offending record and the company,
- **And when** the same field is set to a record belonging to no company at all,
- **Then** the write is accepted.

- **And given** the repair is confirmed, or a part is created on it while it is running,
- **Then** the same check runs over every part movement of the repair, so a part drawing from a `Beta` location is refused at that moment.

*Verifies rule RM-005.*

### AC-174: Confirmation touches only repairs that are still new

- **Given** a selection of four Repair Orders: one in `draft`, one in `confirmed`, one in `under_repair` and one in `done`,
- **When** the confirmation procedure is run over the whole selection,
- **Then** only the `draft` one is checked, has the procurement method of its parts adjusted, has its parts confirmed and is written to `confirmed`,
- **And** the other three are left exactly as they were: none regresses to `confirmed`, and no movement of theirs is touched.

*Verifies rule RM-013 and workflow 8.*

### AC-175: A new tag gets a colour drawn from the eleven presentational values

- **Given** no colour supplied,
- **When** a Repair Tag `Urgent` is created,
- **Then** its colour index is a whole number between 1 and 11 inclusive, drawn at random,
- **And** creating a hundred tags produces indices that all lie in that closed range and none of which is 0,
- **And** nothing in the domain reads the value: two tags sharing a colour behave identically.

- **And given** a colour index of 4 is supplied explicitly,
- **Then** the created tag carries 4 and no draw takes place.

*Verifies rule RM-071 and calculation 17.*

### AC-176: Creating a warehouse creates its repair operation type with the shipped defaults

- **Given** the repair capability installed, and a company holding a production-usage location and an inventory-loss location,
- **When** a warehouse with the short code `WH2` is created,
- **Then** exactly one repair operation type is created for it, named "Repairs", with the code `repair_operation`, the sequence code `RO`, allowed both to create new lots and to use existing ones,
- **And** its default source location, default recycle destination, default product source and default product destination are all the `WH2` stock location,
- **And** its default destination location is the lowest-numbered production-usage location of the company,
- **And** its default remove destination location is the lowest-numbered inventory-loss location of the company, or of the shared set when the company owns none,
- **And** the warehouse's repair operation type field points at it,
- **And** a numbering sequence named as the warehouse name followed by " Sequence repair", prefixed `WH2/RO/` and padded to five digits, exists for it.

*Verifies rules RM-083 and RM-084, beside rules RM-080 and RM-081 verified by AC-089 and AC-090.*

### AC-177: Renaming or archiving a warehouse follows through to its repair operation type

- **Given** the warehouse `WH2` of scenario AC-176, whose repair operation type carries the barcode `WH2RO`,
- **When** the warehouse short code is changed to `W H3`,
- **Then** the repair operation type's barcode becomes `WH3RO`: the spaces are removed, the letters upper-cased, and the literal `RO` appended,
- **And when** the warehouse is archived,
- **Then** the repair operation type is archived with it,
- **And when** the warehouse is unarchived,
- **Then** the repair operation type is active again.

*Verifies rule RM-085.*

### AC-178: Every quantity comparison uses the rounding of the stated unit

- **Given** a part movement whose unit is Units with a rounding of 1.0, meaning whole units only, demanding 3.0, with a recorded quantity of 2.6,
- **Then** the incomplete-parts flag compares 2.6 against 3.0 at the movement's own rounding, which makes them equal, so the flag is **not** set and ending the repair asks no partial-quantity question.

- **And given** the same movement with a recorded quantity of 0.4,
- **Then** the cancel-empty-parts rule compares 0.4 against zero at the same rounding, finds them equal, and cancels that part at completion.

- **And given** a part for a product whose reference unit rounds to two decimal places, with a forecast availability of 1.995 against a part quantity of 2.00,
- **Then** the readiness comparison is made at two decimal places, 2.00 against 2.00, so the part counts as available rather than short.

- **And given** the *Product Unit* decimal precision set to two decimal places, a repair asking for 1.004 and an available quantity of 1.00,
- **Then** the availability check at confirmation compares at two decimal places, finds 1.00 against 1.00, and confirms directly without raising the insufficient-quantity dialogue.

- **And given** a Sales Order Line whose unit rounds to whole units, carrying 0.4,
- **Then** the quantity-against-zero test of the sales binding treats it as zero, so the bound repair is cancelled.

*Verifies rule RM-120 and calculation 33.*

### AC-179: A maintenance stage in use cannot be deleted

- **Given** the four shipped stages and one Maintenance Request sitting in "In Progress",
- **When** a user with the equipment manager group tries to delete the "In Progress" stage,
- **Then** the deletion is refused, because a Maintenance Request points at a Maintenance Stage with a restricting link,
- **And when** the request is first moved to "New Request" and the deletion is retried,
- **Then** the "In Progress" stage is deleted,
- **And** the sequences of the remaining stages are unchanged, so "New Request" keeps the sequence 1.

*Verifies rule RM-230.*

---

## Reconciliation notes

1. **Scenario numbering.** One of the two earlier texts carried no scenarios at all and listed only the topics that scenarios would have to cover; the other carried one hundred and seventy-nine scenarios under two prefixes. Those scenarios are preserved here, renumbered into one sequence so that a single identifier scheme covers the file, and extended where the first text named a topic the second did not exercise: the archived-request case of the effectiveness population (AC-140), the folding of a category whose equipment has all been archived (AC-108), the numbering sequence created with a warehouse (AC-176), the retention of carbon-copy addresses on a request raised by electronic mail (AC-145), the rejection of an unsupported search operator on the scheduled-date category (AC-044), and the emptying of the equipment owner on departure (AC-147).
2. **Dates in scenarios.** One earlier text wrote dates in a purely numeric form. They are written in words here so that no reader has to guess a day-month order, except where the scenario asserts a formatted output, in which case the formatted string is quoted as the screen shows it.
