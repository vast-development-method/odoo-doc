# Inventory Operations — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behavior of the domain, with the exact user-facing message the system produces. Placeholders are written as italic descriptions of the value that is substituted.

The rules are grouped by the entity or the operation they guard. Section 14 collects the invariants that must hold at rest and that an implementation should be able to assert at any moment.

---

# 1. Warehouse

| # | Rule | Condition | Message |
|---|---|---|---|
| 1.1 | Name uniqueness | Two active or archived Warehouses of the same company share a name. | "The name of the warehouse must be unique per company!" |
| 1.2 | Short name uniqueness | Two Warehouses of the same company share a short name. | "The short name of the warehouse must be unique per company!" |
| 1.3 | Short name length | The short name is longer than five characters. | Rejected by the field's own length limit. |
| 1.4 | Company immutability | The company is written with a different value than the current one. | "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |
| 1.5 | Archiving with open work | The Warehouse is archived while at least one Stock Move of one of its Operation Types is neither done nor cancelled. | "You still have ongoing operations for operation types *the list of operation type names* in warehouse *the warehouse name*" |
| 1.6 | Archiving with foreign operation types | The Warehouse is archived while an Operation Type that does not belong to it uses one of its Locations as default source **and** as default destination. | "*the list of operation type names* have default source or destination locations within warehouse *the warehouse name*, therefore you cannot archive it." |
| 1.7 | No warehouse for the company | An Operation Type default Location must be computed and the company has no Warehouse. | For an inventory manager: a redirect to the Warehouse screen carrying "Please create a warehouse for company *the company display name*." with the button "Go to Warehouses". For anyone else: "Please contact your administrator to configure your warehouse." |
| 1.8 | Missing partner locations | The shared customer Location and the shared vendor Location are both absent and no Location of those usages exists. | "Can't find any customer or supplier location." |
| 1.9 | Missing generic route | A generic Route named in the warehouse generation cannot be found and creation is not permitted. | "Can't find any generic route *the route name*." |

**Warning (not an error).** Creating a Warehouse while neither the multi-warehouse nor the multi-location group is granted shows: title "Warning", message "Creating a new warehouse will automatically activate the Storage Locations setting".

---

# 2. Location

| # | Rule | Condition | Message |
|---|---|---|---|
| 2.1 | Barcode uniqueness | Two Locations of the same company share a barcode. | "The barcode for a location must be unique per company!" |
| 2.2 | Counting frequency sign | The counting frequency is negative. | "The inventory frequency (days) for a location must be non-negative" |
| 2.3 | Counting frequency overflow | The frequency pushes the next count date beyond the representable calendar. | "The selected Inventory Frequency (Days) creates a date too far into the future." |
| 2.4 | Single replenishment location per branch | A Location is marked as a replenishment Location while an ancestor or a descendant already is. | "Another parent/sub replenish location *the other location's name* exists, if you wish to change it, uncheck it first" |
| 2.5 | Scrap location versus manufacturing | The usage is set to inventory loss while the Location is the default destination of an Operation Type of the manufacturing kind. | "You cannot set a location as a scrap location when it is assigned as a destination location for a manufacturing type operation." |
| 2.6 | The shared inter-company location cannot be deleted | That Location is among those being deleted. | "The *the location name* location is required by the Inventory app and cannot be deleted, but you can archive it." |
| 2.7 | Company immutability | The company is written with a different value. | "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |
| 2.8 | Converting a stocked location to virtual | The usage is set to virtual while the Location holds quantity records. | "This location's usage cannot be changed to view as it contains products." |
| 2.9 | Converting a stocked location at all | The usage is changed while at least one of the changed Locations holds a quantity record with a strictly positive on-hand quantity. | "Internal locations having stock can't be converted" |
| 2.10 | Archiving a warehouse location | The Location is archived while an active Warehouse uses it as stock Location or as view Location. | "You cannot archive location *the location display name* because it is used by warehouse *the warehouse display name*" |
| 2.11 | Archiving a non-empty subtree | The Location is archived while an internal descendant still has a non-zero on-hand or reserved quantity. | "You can't disable locations *the comma-separated list of location display names* because they still contain products." |

**Cascades.** Archiving a Location archives its whole subtree (unless rule 2.11 stops it). Deleting a Location deletes its whole subtree.

---

# 3. Route and Stock Rule

| # | Rule | Condition | Message |
|---|---|---|---|
| 3.1 | Rule company matches route company (checked from the rule) | The rule's company differs from its Route's company, which is set. | "Rule *the rule display name* belongs to *the rule's company display name* while the route belongs to *the route's company display name*." |
| 3.2 | Rule company matches route company (checked from the route) | Same condition, checked when the Route's company changes. | Same message. |
| 3.3 | A pull rule needs a source | A pull rule runs without a source Location. | "No source location defined on stock rule: *the rule name*!" |
| 3.4 | Endless routing loop | A chain of rules would send a product back to a Location it has already left. | "Invalid rule's configuration, the following rule causes an endless loop: *the rule display name*" |
| 3.5 | No rule found | The rule engine cannot find a rule to satisfy a need. | Raised by `../replenishment-and-procurement/`; the collected messages are joined by line breaks into a single failure. |

**Cascades.** Archiving a Route archives its rules whose destination Location is still active; un-archiving it un-archives them. Deleting a Route deletes its rules. A Warehouse restricts the deletion of its receipt Route and its delivery Route.

---

# 4. Transfer

## 4.1 Structural rules

| # | Rule | Condition | Message |
|---|---|---|---|
| 4.1.1 | Reference uniqueness | Two Transfers of the same company share a reference. | "Reference must be unique per company!" |
| 4.1.2 | Operation Type immutability once final | The Operation Type is written on a done or cancelled Transfer. | "Changing the operation type of this record is forbidden at this point." |
| 4.1.3 | Scheduled date on a cancelled transfer | The scheduled date is written on a cancelled Transfer. | "You cannot change the Scheduled Date on a cancelled transfer." |
| 4.1.4 | Nothing to reserve | The availability action is pressed and no move is in a state that can be reserved. | "Nothing to check the availability for." |
| 4.1.5 | Deleting linked moves | Deleting a Transfer deletes its moves; a move that is neither draft nor cancelled and that is chained blocks the deletion. | "You can not delete moves linked to another operation" |

## 4.2 Validation sanity check

Checked in this order. When a **single** Transfer is being validated, the first failure stops everything:

| # | Condition | Message |
|---|---|---|
| 4.2.1 | The Transfer has no moves and no detail lines. | "You can’t validate an empty transfer. Please add some products to move before proceeding." |
| 4.2.2 | Every move that is neither done nor cancelled — restricted to the picked ones when at least one is picked — has a zero processed quantity at the Product Unit precision. | "Transfer trouble alert! Validating a zero quantity transfer? You're not moving invisible goods around are you?\nSet some quantities and let's get moving!" |
| 4.2.3 | The Operation Type allows creating or using lots and at least one line to check has neither a Lot nor a typed name. | "You need to supply a Lot/Serial number for products *the comma-separated product display names*." |

When **several** Transfers are validated together, the three failures are merged into one message instead: the empty ones contribute "Transfers *the comma-separated references*: Please add some items to move.", the ones missing lots contribute "\n\nTransfers *the references*: You need to supply a Lot/Serial number for products *the product display names*.", and the resulting text is stripped of leading whitespace before being raised. The zero-quantity case is **not** reported in the multiple-Transfer form.

Which lines are checked for a lot:

- For a Transfer that has no processed quantity anywhere, every line whose product is tracked.
- For every other Transfer, only the lines whose product is tracked, which are picked and whose quantity is non-zero.
- When the check is run for a whole batch, the choice between those two cases is made once for the batch rather than per Transfer, so a lot typed on one Transfer satisfies the whole batch.

## 4.3 Splitting

| # | Condition | Message |
|---|---|---|
| 4.3.1 | Every move has a zero processed quantity. | "*the transfer display name*: Nothing to split. Fill the quantities you want in a new transfer in the done quantities" |
| 4.3.2 | Every move is exactly fully processed. | "*the transfer display name*: Nothing to split, all demand is done. For split you need at least one line not fully fulfilled" |
| 4.3.3 | At least one move is over-processed. | "*the transfer display name*: Can't split: quantities done can't be above demand" |

## 4.4 Locking

- While the Transfer is **open** and locked, the demand of its moves is not editable. The indicator that drives the screen is: the demand is editable when the Transfer is unlocked **or** the move is in draft.
- While the Transfer is **done** and locked, the processed quantities are not editable and the scheduled date is frozen.
- Cancelling a Transfer forces the lock on.
- The lock flag is not copied, so a backorder or a return starts locked.

## 4.5 Warnings (not errors)

- Changing the source Location of a Transfer whose moves are chained shows: title "Warning: change source location", message "Updating the location of this transfer will result in unreservation of the currently assigned items. An attempt to reserve items at the new location will be made and the link with preceding transfers will be discarded.\n\nTo avoid this, please discard the source location change before saving."
- A contact carrying a transfer warning contributes its message, and its parent's message, to the Transfer's instruction text; this is shown only when the stock-warning group is active.

---

# 5. Stock Move

| # | Rule | Condition | Message |
|---|---|---|---|
| 5.1 | Writing the real quantity | The computed real-quantity field is written directly. | "The requested operation cannot be processed because of a programming error setting the `product_qty` field instead of the `product_uom_qty`." |
| 5.2 | Writing a quantity on a cancelled move | The processed quantity is written while any move of the set is cancelled. | "You cannot change a cancelled stock move, create a new line instead." |
| 5.3 | Changing the unit of a done move | The line unit is written while any move of the set is done, and the unit-conversion bypass is not active. | "You cannot change the UoM for a stock move that has been set to 'Done'." |
| 5.4 | Quantity respects the general precision | The value written as processed quantity differs from itself rounded to the Product Unit precision. | "The quantity done for the product *the product display name* doesn't respect the rounding precision defined on the system. Please change the quantity done or the rounding precision in your settings." One line per offending move, joined by line breaks. |
| 5.5 | Unreserving a done move | A done move whose destination usage is not inventory loss is unreserved. | "You cannot unreserve a stock move that has been set to 'Done'." |
| 5.6 | Cancelling a done move | Any move of the set is done and its destination usage is not inventory loss. | "You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place." |
| 5.7 | Splitting a final move | The move is done or cancelled. | "You cannot split a stock move that has been set to 'Done' or 'Cancel'." |
| 5.8 | Splitting a draft move | The move is in draft. | "You cannot split a draft move. It needs to be confirmed first." |
| 5.9 | Deleting a chained move | The move is neither draft nor cancelled and has originating or destination moves. | "You can not delete moves linked to another operation" |
| 5.10 | Serial-number generation count | The requested count is zero or absent. | "The number of Serial Numbers to generate must be greater than zero." |
| 5.11 | Serial-number generation without a product | The generation is requested with no product in context. | "No product found to generate Serials/Lots for." |
| 5.12 | Quantity per lot | The quantity per lot is zero or negative. | "The quantity per lot should always be a positive value." |
| 5.13 | Adding containers without a transfer | The add-packages action runs with no Transfer in context. | "You need a transfer to add these packages to." |
| 5.14 | Container split across locations | After completion, a destination container holds records with a strictly positive quantity in more than one Location. | "You cannot move the same package content more than once in the same transfer or split the same package into two location." followed by a line break, "Package: " and the container name. |
| 5.15 | Availability search value | The availability search is used with no value. | "Search not supported without a value." |
| 5.16 | Availability search state | The availability search is used with a state outside the four defined ones. | "Selection not supported." |
| 5.17 | Availability search operator | The availability search is used with an operator other than equals, not-equals, in or not-in. | "Operation not supported" |

## 5.1 Consequences of raising or lowering the demand

Writing the demand of a move that is neither draft nor done and that belongs to a Transfer:

1. A note describing the change is posted on the Transfer for every move whose demand really changes.
2. Unless unreservation is explicitly suppressed:
   - moves whose current processed quantity is **greater** than the new demand are unreserved entirely;
   - the other moves that were assigned become partially available;
   - the unreserved moves whose source usage is vendor, and the still-reserved moves whose source usage is vendor and whose state is partially available or assigned, are re-reserved immediately;
   - every remaining move has its status recomputed.

## 5.2 Consequences of changing the source location

Writing the source Location of a move:

1. For every detail line whose current source Location is not inside the move's new source Location: the move is scheduled for re-reservation, its supply method is reset to take-from-stock, its originating links are cleared, and the line is deleted.
2. The move's Warehouse is recomputed from the new source Location, falling back to the destination Location's Warehouse.

---

# 6. Stock Move Line

| # | Rule | Condition | Message |
|---|---|---|---|
| 6.1 | Lot belongs to the product | The line's Lot names a different product. | "This lot *the lot name* is incompatible with this product *the product display name*" |
| 6.2 | No negative quantity | The quantity is negative. | "You can not enter negative quantities." |
| 6.3 | One unit per serial number | The quantity in the product unit is neither zero nor exactly one for a serial-tracked product. | "You can only process 1.0 *the product unit name* of products with unique serial number." |
| 6.4 | Product change outside draft | The product is written while the line is not in draft. | "Changing the product is only allowed in 'Draft' state." |
| 6.5 | Lot change across products | The lot or the picked quantity record is written on a set of lines spanning more than one product. | "Changing the Lot/Serial number for move lines with different products is not allowed." |
| 6.6 | Negative reservation | The recomputed quantity in the product unit is negative. | "Reserving a negative quantity is not allowed." |
| 6.7 | Deleting a final line | The line's status is done or cancelled. | "Deleting product moves after the transfer is done?\n\nThat would be like going back in time to revert all operations triggered after this move. Who knows what the end result would be, So let's not do it.\n\nTry changing the “done” quantity to 0 instead." |
| 6.8 | Rounding at completion | The quantity rounded at the line unit's rounding step differs, at the Product Unit precision, from the quantity rounded to that precision. | "The quantity done for the product \"*the product display name*\" doesn't respect the rounding precision defined on the unit of measure \"*the line unit name*\". Please change the quantity done or the rounding precision of your unit of measure." |
| 6.9 | No negative quantity at completion | The quantity is strictly negative at completion. | "No negative quantities allowed" |
| 6.10 | Lot required at completion | A tracked product's line reaches completion with neither a Lot nor a typed name, and the line is not excluded from the requirement. | "You need to supply a Lot/Serial Number for product:\n" followed by one line per distinct product display name, each prefixed with "- ". |
| 6.11 | Packing across operation types | The put-in-pack action spans more than one Operation Type. | "You cannot pack products into the same package when they are from different transfers with different operation types" |

## 6.1 When a lot is **not** required

A line whose product is tracked escapes rule 6.10 when any of the following is true:

- it is excluded from the requirement: its move has an Operation Type, or the move is an adjustment move, or the line already has a Lot, or the move belongs to a Scrap. (Note the polarity: a line whose move has **no** Operation Type and which is not an adjustment, not already lotted and not a scrap is the one that fails immediately.)
- there is no Operation Type at all;
- the line already has a Lot;
- the Operation Type allows **neither** creating nor using existing lots. Turning both switches off is therefore the supported way to receive or ship tracked goods without recording numbers.

## 6.2 Warnings on serial numbers (not errors)

| Situation | Message |
|---|---|
| The same serial number is typed twice within one Transfer. | "You cannot use the same serial number twice. Please correct the serial numbers encoded." |
| A typed serial number already exists in stock. | "Serial number (*the typed name*) already exists in location(s): *the list of location display names*. Please correct the serial number encoded." |
| An existing serial number is being **assigned** although it is already somewhere. | "The Serial Number (*the serial number*) is already used in location(s): *the list of location display names*.\n\nIs this expected? For example, this can occur if a delivery operation is validated before its corresponding receipt operation is validated. In this case the issue will be solved automatically once all steps are completed. Otherwise, the serial number should be corrected to prevent inconsistent data." |
| An existing serial number is **used** from a Location where it is not, and a better Location was found in the same company. | "Serial number (*the serial number*) is not located in *the chosen source location display name*, but is located in location(s): *the list of location display names*.\n\nSource location for this move will be changed to *the recommended location display name*" — and the source Location is silently switched to the recommendation. |
| Same, but no suitable Location was found. | "Serial number (*the serial number*) is not located in *the chosen source location display name*, but is located in location(s): *the list of location display names*.\n\nPlease correct this to prevent inconsistent data." |
| Lots are assigned on a move and some of them are in Locations outside the Transfer's source. | "Unavailable Serial numbers. Please correct the serial numbers encoded: " followed, per offending record, by "\n(*the lot display name*) exists in location *the location display name*". |

The recommendation is chosen as follows: when a reference document Location is supplied, the first Location among those holding the serial number whose path contains that reference Location's path; otherwise the first Location among them whose usage is not customer. The recommendation is only applied when its company equals the checking company.

---

# 7. Stock Quantity

| # | Rule | Condition | Message |
|---|---|---|---|
| 7.1 | Storable only | The product is not storable. | "Quants cannot be created for consumables or services." |
| 7.2 | Never in a virtual location | The Location's usage is virtual. | "You cannot take products from or deliver products to a location of type \"view\" (*the location name*)." |
| 7.3 | Lot belongs to the product | The Lot names a different product. | "The Lot/Serial number (*the lot name*) is linked to another product." |
| 7.4 | One unit per serial number | For a serial-tracked product, the sum of quantities over the descendants of a Location for one serial number exceeds one in absolute value. Inventory-loss Locations are excluded. | "The serial number has already been assigned: \n Product: *the product display name*, Serial Number: *the serial number*" |
| 7.5 | No duplication | The duplicate action is used. | "You cannot duplicate stock quants." |
| 7.6 | Restricted creation in counting mode | A record is created in counting mode with a field that is neither an extension field nor one of the allowed creation fields. | "Quant's creation is restricted, you can't do this operation." |
| 7.7 | Restricted editing in counting mode | In counting mode, one of the protected fields (product, Location, lot, container, owner) is written. When the record sits in an inventory-loss Location the write is silently ignored instead. | "Quant's editing is restricted, you can't do this operation." |
| 7.8 | Deletion reserved to managers | A non-superuser who is not an inventory manager deletes a record. | "Quants are auto-deleted when appropriate. If you must manually delete them, please ask a stock manager to do it." |
| 7.9 | Relocation preconditions | The selection spans more than one company, or a record has no company, or a record's quantity is not strictly positive. | "You can only move positive quantities stored in locations used by a single company per relocation." |
| 7.10 | Unreserving more than exists | A negative reservation request exceeds the total reserved quantity of the gathered records. | "It is not possible to unreserve more products of *the product display name* than you have in stock." |
| 7.11 | At least one delta | Neither an on-hand delta nor a reserved delta is supplied. | "Quantity or Reserved Quantity should be set." |
| 7.12 | Removal strategy implemented | The resolved removal strategy key is not one of the implemented ones. | "Removal strategy *the method key* not implemented." |

**Allowed fields in counting mode.** On write: the counted quantity, the auto-apply counted quantity, the difference, the scheduled date, the assignee, the counted flag, the outdated indicator, the lot, the Location and the container. On create: those plus the product and the owner.

**Warning (not an error).** Typing a counted quantity on a record located in an inventory-loss Location shows: title "You cannot modify inventory loss quantity", message "Editing quantities in an Inventory Adjustment location is forbidden,those locations are used as counterpart when correcting the quantities."

**Manager deletion behavior.** When an inventory manager deletes records, the deletion is replaced by an adjustment to zero: the counted quantity is set to zero and applied, so that the disappearance is recorded as moves.

---

# 8. Lot or Serial Number

| # | Rule | Condition | Message |
|---|---|---|---|
| 8.1 | Uniqueness | The combination of product and name occurs more than once within one company, or once in a company and once with no company. | "The combination of lot/serial number and product must be unique within a company including when no company is defined.\nThe following combinations contain duplicates:\n" followed by one line per duplicate: " - Product: *the product display name*, Lot/Serial Number: *the name*" |
| 8.2 | Creation forbidden by the operation type | A Lot is created from a Transfer screen whose Operation Type does not allow creating lots. | "You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box \"Create New Lots/Serial Numbers\"." |
| 8.3 | Relocating a scattered lot | The single-Location field is written while the lot has a positive quantity in more than one Location. | "You can only move a lot/serial to a new location if it exists in a single location." |
| 8.4 | Company change while stocked elsewhere | The company is written while the lot currently sits in a Location belonging to another company. | "You cannot change the company of a lot/serial number currently in a location belonging to another company." |
| 8.5 | Product change after movements | The product is written and completed detail lines already exist for the lot with a different product. | "You are not allowed to change the product linked to a serial or lot number if some stock moves have already been created with that number. This would lead to inconsistencies in your stock." |

Note on the uniqueness check: lots belonging to **different** companies are deliberately not compared with each other; only duplicates within one company, and duplicates between one company and the company-less pool, are rejected. When any record being checked has no company, the check is performed with elevated rights so that it can see the whole pool.

---

# 9. Package and Package Type

| # | Rule | Condition | Message |
|---|---|---|---|
| 9.1 | Clearing the location of a full container | The Location is cleared while the container still holds contents. | "Cannot remove the location of a non empty package" |
| 9.2 | Moving an empty container | A Location is written on a container that holds nothing. | "Cannot move an empty package" |
| 9.3 | Destination container recursion | A container names one of its own destination-chain descendants as destination container. | "A package can't have one of its contained packages as destination container." |
| 9.4 | Group split at completion | The containers of one destination group end up in different Locations. | "Packages *the list of container names* are moved to different locations while being in the same container *the container name*." |
| 9.5 | Destination container already elsewhere | The destination container already holds contents located somewhere other than the group's new Location. | "Can't move a container having packages in another location (*the old location display name*) to a different location (*the new location display name*)." |
| 9.6 | Package Type barcode uniqueness | Two Package Types share a barcode. | "A barcode can only be assigned to one package type!" |
| 9.7 | Package Type dimensions | Height, width, length or maximum weight is negative. | "Height must be positive", "Width must be positive", "Length must be positive", "Max Weight must be positive" |

**Reusable containers.** A container whose type is reusable is never taken over as an entire package by the whole-container detection, and is never promoted as a parent container. Its goods are taken out of it instead of travelling with it.

---

# 10. Storage Category and Put-away Rule

| # | Rule | Condition | Message |
|---|---|---|---|
| 10.1 | Maximum weight sign | The maximum weight is negative. | "Max weight should be a positive number." |
| 10.2 | Capacity quantity sign | A capacity quantity is zero or negative. | "Quantity should be a positive number." |
| 10.3 | One capacity per product | Two capacities of the same category name the same product. | "Multiple capacity rules for one product." |
| 10.4 | One capacity per container type | Two capacities of the same category name the same container type. | "Multiple capacity rules for one package type." |
| 10.5 | Put-away rule company immutability | The company of a put-away rule is written with a different value. | "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |

**Warning (not an error).** Choosing the closest-location sublocation mode with a storage category that no descendant of the target Location carries shows: title "Warning", message "Selected storage category does not exist in the 'store to' location or any of its sublocations".

**Structural restrictions.** A put-away rule's arrival Location must have children. Its target Location must be a descendant of its arrival Location; changing the arrival Location resets the target when the target is no longer inside it. Its category choice is restricted to the categories flagged as usable for put-away rules.

---

# 11. Scrap

| # | Rule | Condition | Message |
|---|---|---|---|
| 11.1 | Positive quantity | The scrap quantity is zero at the chosen unit's precision. | "You can only enter positive quantities." |
| 11.2 | Deleting a completed scrap | The Scrap's status is done. | "You cannot delete a scrap which is done." |
| 11.3 | Insufficient quantity | The available quantity at the exact source characteristics is below the scrap quantity converted to the product unit. | Not an error: the shortage screen opens, titled "*the product display name*: Insufficient Quantity To Scrap". The person may confirm and scrap anyway. |

The availability test uses strict matching on Location, lot, container and owner, and is skipped entirely for products that are not storable.

---

# 12. Batch and wave transfers

| # | Rule | Condition | Message |
|---|---|---|---|
| 12.1 | Confirming an empty batch | The batch has no Transfers. | "You have to set some pickings to batch." |
| 12.2 | Deleting a completed batch | The batch's status is done. | "You cannot delete Done batch transfers." |
| 12.3 | Composition | A Transfer of the batch is not among the allowed Transfers (wrong state, or wrong Operation Type). | "The following transfers cannot be added to batch transfer *the batch name*. Please check their states and operation types.\n\nIncompatibilities: *the list of transfer references*" |
| 12.4 | Merge selection size | Fewer than two are selected. | "Please select at least two batch/wave transfers to merge." |
| 12.5 | Merge operation types | The selection spans more than one Operation Type. | "Batch/Wave transfers with different operation types cannot be merged." |
| 12.6 | Merge kinds | The selection mixes batches and waves. | "Batch transfers cannot be merged with wave transfers and vice versa." |
| 12.7 | Merge states | The selection spans more than one state. | "Batch/Wave transfers with different states cannot be merged." |
| 12.8 | Merge finality | The shared state is done or cancelled. | "You cannot merge done or cancelled batch/wave transfers." |
| 12.9 | Automatic batching needs a criterion | Automatic batching is on and no batch or wave grouping option is chosen. | "If the Automatic Batches feature is enabled, at least one 'Group by' option must be selected." |
| 12.10 | Batch composition wizard, company | The selected Transfers span more than one company. | "The selected pickings should belong to an unique company." |
| 12.11 | Wave composition wizard, operation type | The selected Transfers span more than one Operation Type. | "The selected transfers should belong to the same operation type" |
| 12.12 | Wave composition wizard, company of the operations | The selected detail lines span more than one company. | "The selected operations should belong to a unique company." |
| 12.13 | Wave composition wizard, company of the transfers | The selected Transfers span more than one company. | "The selected transfers should belong to a unique company." |
| 12.14 | Wave creation impossible | No wave could be created from the selection. | "Cannot create wave transfers" |

**Allowed Transfers of a batch.** The Transfers of the same company whose state is waiting-another-operation, waiting or ready — plus the draft ones when the batch itself is still draft — and, when the batch has an Operation Type, of exactly that Operation Type.

**Automatic detachment.** A Transfer that becomes done while other Transfers of its batch are not is removed from the batch. A Transfer that is cancelled while others of its batch are not is removed from the batch. A batch that is left with no Transfers while in progress is cancelled.

---

# 13. Settings and shared configuration

| # | Rule | Condition | Message |
|---|---|---|---|
| 13.1 | Multi-location cannot be switched off with several warehouses | The multi-location group is being withdrawn while both the multi-location and the multi-warehouse groups are implied for internal users. | "You can't deactivate the multi-location if you have more than once warehouse by company" |
| 13.2 | Lot tracking cannot be switched off with tracked products | The lot group is being withdrawn while at least one product has a tracking mode other than none. | "You have product(s) in stock that have lot/serial number tracking enabled. \nSwitch off tracking on all the products before switching off this setting." |
| 13.3 | Product unit change after movement | The unit of measure of a product is changed while moves or detail lines already exist in another unit. | "As other units of measure (ex : *the other unit name*) than *the product's unit name* have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one." |
| 13.4 | Unit ratio change after movement | The factor or relative unit of a unit of measure is changed while open moves, open detail lines or non-zero quantity records exist for products using it. | "You cannot change the ratio of this unit of measure as some products with this UoM have already been moved or are currently reserved." |
| 13.5 | Product company change with foreign movements | A product's company is changed while moves of it belong to another company. | "This product's company cannot be changed as long as there are stock moves of it belonging to another company." |
| 13.6 | Product company change with foreign quantities | A product's company is changed while quantity records of it belong to another company. | "This product's company cannot be changed as long as there are quantities of it belonging to another company." |
| 13.7 | On-hand quantity on an unsaved product | The on-hand quantity is edited before the product is saved. | "Save the product form before updating the Quantity On Hand." |

**Side effects of the multi-location setting.** Turning it on activates the internal-transfer Operation Type of every Warehouse and deactivates the two simplified Location screens. Turning it off deactivates the internal-transfer Operation Type of the Warehouses that both receive in one step and deliver in one step, and re-activates those screens.

---

# 14. Invariants

These must hold whenever no operation is in flight. An implementation should be able to check them and repair them with the housekeeping pass of `calculations.md`, section 12.

| # | Invariant |
|---|---|
| 14.1 | For every (product, Location, lot, container, owner) there is **at most one** Stock Quantity record. Duplicates are collapsed by the merge pass. |
| 14.2 | For every (product, Location, lot, container, owner), the reserved counter equals the sum of the quantities in the product unit of the open detail lines that draw from exactly those characteristics — unless the Location bypasses reservation, in which case the counter is zero. |
| 14.3 | A reserved counter is never negative. |
| 14.4 | No Stock Quantity record exists whose on-hand quantity, reserved quantity and counted quantity are all zero and which has no assignee. |
| 14.5 | No Stock Quantity record exists in a Location whose usage is virtual. |
| 14.6 | For every serial-tracked product and every serial number, the sum of on-hand quantities over the descendants of any Location, excluding inventory-loss Locations, is at most one in absolute value. |
| 14.7 | A Stock Move's processed quantity equals the sum of its detail lines' quantities converted into its line unit. |
| 14.8 | A Stock Move's status is consistent with its processed quantity, its demand and its originating moves, per the recomputation rule of `state-machines.md`, section 1.3 — except for done and cancelled moves, which are frozen. |
| 14.9 | A Transfer's status is exactly the value the derivation of `state-machines.md`, section 2.2, produces from its moves. |
| 14.10 | A Stock Move Line's status equals its Stock Move's status. |
| 14.11 | Every detail line of a done Transfer has a date; every done Stock Move has a date equal to the instant it was completed (or a later value written by hand). |
| 14.12 | A Location's full name equals its parent's full name, a slash and its own name, unless it has no parent or its own usage is virtual. |
| 14.13 | A Location's materialised path starts with its parent's path. |
| 14.14 | A container's full name equals its parent container's full name, " > " and its own name. |
| 14.15 | A container's Location equals the Location of its first positive-quantity record, or, when it has none, the Location of its first child container. |
| 14.16 | A container never names a descendant of its own destination chain as destination container. |
| 14.17 | A Stock Rule's company equals its Route's company whenever the Route has one. |
| 14.18 | A Warehouse's stock Location, input Location, quality control Location, output Location and packing Location are all descendants of its view Location. |
| 14.19 | A Transfer's reference is unique within its company. |
| 14.20 | A Lot's (product, name) pair is unique within its company and against the company-less pool. |
| 14.21 | Every move of a Transfer shares that Transfer's company. |
| 14.22 | The document references of a Transfer are the union of its moves' document references. |

---

# 15. Permission checks

| Operation | Requirement |
|---|---|
| Read the operational documents | Membership of the inventory user group, plus the record rules of `configuration.md`, section 5. |
| Create, edit, confirm, reserve, validate, cancel a Transfer | Inventory user group. |
| Create or edit a Stock Move, a Stock Move Line, a Scrap | Inventory user group. |
| Write a counted quantity and apply it | Inventory user group, **and** the counting-mode context must be active; otherwise the counted-quantity write is ignored entirely rather than refused. |
| Delete a Stock Quantity record | Inventory manager group (and the deletion becomes an adjustment to zero). |
| Create, edit or delete a Warehouse, a Location, an Operation Type, a Route, a Stock Rule, a Put-away Rule, a Storage Category, a Package Type, a Removal Strategy | Inventory manager group. |
| See the multi-location fields and screens | Multi-location group. |
| See the multi-warehouse fields and screens | Multi-warehouse group. |
| Use lots and serial numbers | Lot group. |
| Use containers | Container group. |
| Use owners on goods | Consignment group. |
| See the Reception Report | Reception-report group. |
| Use the routes and rules screens | Advanced-routing group. |
| Write the numbering sequence of an Operation Type | Performed with elevated rights on behalf of an inventory manager. |

**Elevated-rights operations.** The following always run with elevated rights regardless of the acting user: creating and updating numbering sequences; reading the removal strategy of a product category or of parent Locations; reading and writing Stock Quantity records during reservation and completion; creating the moves produced by a rule; the record housekeeping pass; the automatic batching search.

---

# 16. Locking and concurrency

| # | Rule |
|---|---|
| 16.1 | Before a Stock Quantity record is modified by the "increase or decrease" routine, a write lock is taken on the first candidate record. This serialises concurrent reservations of the same goods. |
| 16.2 | When the lock cannot be taken because a concurrent transaction already holds it, the routine creates a **new** record instead of waiting. The duplicate is collapsed later by the merge pass, which is why invariant 14.1 is only guaranteed at rest. |
| 16.3 | The merge pass runs inside a save point; a failure is logged and leaves the data untouched. |
| 16.4 | The Transfer lock flag is a business lock, not a concurrency lock: it prevents people from editing, not transactions from conflicting. |
| 16.5 | The whole-container detection can be suppressed for one operation (the backorder moves created during a completion are confirmed with it suppressed, so that the containers of the Transfer being validated are not disturbed). |
| 16.6 | Put-away can be suppressed for one operation, which the screens use when they are about to rewrite the destination Locations themselves. |
| 16.7 | Status recomputation can be suppressed for one operation, which the completion algorithm uses so that a move being completed is not pulled back to an open status. |

---

# 17. Edge-case behaviors

| # | Situation | Behavior |
|---|---|---|
| 17.1 | A move is created on a Transfer that is already done. | The move is created directly in status done and picked. |
| 17.2 | A move is created on a Transfer that is open but confirmed. | The move is flagged as additional, which makes the Transfer re-run its confirmation. |
| 17.3 | A move is created with both a lot list and either a quantity or detail lines. | The lot list is dropped from the creation values; the explicit lines win. |
| 17.4 | Both a lot list and a quantity are written in the same operation. | The keys are sorted so that the lot list is applied first and the quantity second, and the processed quantity is then forcibly recomputed from the lines. |
| 17.5 | A move's demand is negative. | The move is detached from its Transfer during merging, absorbed into a positive sibling when one matches, and otherwise turned into a return by swapping its Locations, negating its demand and switching to the return Operation Type. |
| 17.6 | A positive move is fully absorbed by a negative one. | Its demand becomes zero and it is cancelled, unless it is picked. |
| 17.7 | An over-processed move is validated. | No backorder move, no backorder question, no extra move; the surplus simply travels. See `calculations.md`, section 21. |
| 17.8 | A tracked product is delivered before its receipt is validated. | The delivery drives the source quantity negative and the serial-number warning explains that the situation resolves itself once the receipt is validated. |
| 17.9 | A negative quantity record coexists with positive ones of the same characteristics. | The negative pocket mechanism of `calculations.md`, section 3.3, makes the positive ones absorb it before anything can be reserved. |
| 17.10 | A lot-bearing record goes negative while lot-less stock exists at the same Location. | The completion compensates: the lot-less pocket is reduced and the lot's records are increased by the same amount. |
| 17.11 | A container is moved whose parent is only partly moved. | The parent is **not** taken along: the promotion only applies when every child of the parent is part of the move. |
| 17.12 | A container is emptied by a Transfer. | Its destination-container link is cleared when it no longer has any open detail line. |
| 17.13 | A Transfer is printed and then a new move is created that would have matched it. | The printed Transfer is excluded from the grouping search, so a new Transfer is created instead. |
| 17.14 | A Transfer groups moves with different contacts. | The Transfer's contact is cleared. |
| 17.15 | A Transfer groups moves with different origins. | The origins are concatenated with commas, in order and without duplicates. At creation only the first five distinct origins are kept and three dots are appended when there were more. |
| 17.16 | A scrap move sits inside an otherwise cancelled Transfer. | The Transfer's status becomes cancelled, not done. |
| 17.17 | An adjustment move has a difference of exactly zero and the application comes from the product's own quantity field. | No move is created at all. |
| 17.18 | A count is applied on a record whose on-hand quantity moved in the meantime. | The conflict screen opens; keeping the count applies it as typed, discarding clears it. |
| 17.19 | A detail line is created on a Transfer with no move. | A move is found or created for it, with a demand of zero for an open Transfer and with the line's quantity as demand for a done one. |
| 17.20 | A reservation would take a fractional quantity of a serial-tracked product. | The quantity is set to zero and nothing is reserved. |
| 17.21 | The gathering finds nothing and the strategy is least packages. | The container pre-selection abandons and the plain ordering is used. |
| 17.22 | The least-packages search exhausts memory. | The event is logged and the unrestricted filter is used. |
| 17.23 | A put-away rule points at a Location that fails every capacity check. | The walk continues to the next rule; when all fail, the goods stay in the arrival Location (or, for a virtual arrival Location, go to its first internal descendant). |
| 17.24 | A container is put away and its lines would end up in different Locations. | The grouping is abandoned and every line is reset to its move's destination Location. |
| 17.25 | The Warehouse short name contains spaces or lower-case letters. | Barcodes are generated from the short name with spaces removed and letters upper-cased; Location barcodes are only applied when free. |
| 17.26 | Two Operation Types share a sequence prefix. | Only a warning is shown; the configuration is accepted. |
| 17.27 | A batch sequence has no slash in its prefix. | The batch name falls back to the Operation Type prefix, a slash and the number, and a note is posted explaining the misconfiguration. |
| 17.28 | A return is validated short. | Its Operation Type's backorder policy is ignored, because a return always ignores backorders. |
| 17.29 | A Transfer's Operation Type is changed while it is open. | A new reference is drawn from the new sequence and both Locations are reset to the new type's defaults. |
| 17.30 | A quantity record is imported from a file with no Location. | The stock Location of the company's first Warehouse is used. |
| 17.31 | A quantity record is imported with a lot that belongs to another product. | A lot with the same name is searched for the correct product and created when missing; the record then points at it. |
| 17.32 | Records are imported one line at a time. | The automatic merge with an existing record is skipped during import, so that one imported line produces exactly one record; the merge pass collapses them afterwards. |
