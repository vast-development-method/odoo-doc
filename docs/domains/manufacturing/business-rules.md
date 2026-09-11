# Manufacturing — Business Rules

Every validation, constraint, invariant, error message, permission check, locking rule and
edge-case behaviour of the manufacturing domain.

Messages are reproduced exactly as the system emits them, with placeholders written out in
words in italics. A message that ends without a full stop in the system is reproduced
without one.

Three kinds of refusal exist and behave differently:

| Kind | When it is raised | Effect |
|---|---|---|
| Database check | Whenever the row is written. | The write fails; the message is shown. |
| Record constraint | After the write, on the fields the constraint watches. | The whole transaction is rolled back; the message is shown. |
| Operation refusal | Inside an action or a write override. | The operation stops; the message is shown. |

A fourth kind, the **interactive warning**, does not refuse anything: it is shown while the
user edits a form and the user may ignore it.

---

## 1. Bill of Materials

### 1.1 Database checks

| Rule | Condition | Message |
|---|---|---|
| Positive recipe quantity | `product_qty` must be strictly greater than zero. | The quantity to produce must be positive! |
| Non-negative component quantity | A component line's `product_qty` must be greater than or equal to zero. | All product quantities must be greater or equal to 0.<br>Lines with 0 quantities can be used as optional lines. <br>You should install the mrp_byproduct module if you want to manage extra products on BoMs! |

A component line of quantity zero is legitimate: it documents an optional component. It is
skipped by the kit-quantity computation and by the kit quantity derivation, both of which
would otherwise divide by zero.

### 1.2 Cycle prevention

**Watched fields.** `active`, `product_id`, `product_tmpl_id`, `bom_line_ids`. Writing
`sequence` on a batch also re-checks the whole prefetched batch once the last record of the
batch is written.

**Algorithm.**

1. Start from the recipes being written. If any of them has components, additionally load
   every recipe that the selection domain would return for any of those components (that is,
   the recipes that could produce a component), and check those too.
2. For each recipe to check that is active:
   - the *finished products* are the recipe's specific variant, or every variant of its
     template;
   - when any component line carries a variant restriction, group the finished products by
     the set of components that survive the skip rule for them, and check each group with
     its own component set; otherwise check all components against all finished products.
3. The recursive check, given a component set and a finished-product set:
   1. For each component: if the component is one of the finished products, refuse.
      Otherwise, if the component has not been visited yet, queue it for a recipe lookup.
   2. Resolve the recipes of the queued components in one lookup (any kind).
   3. For each component: if it has not been visited, record its sub-components as the
      components of its recipe that survive the skip rule for that component.
   4. For each component with sub-components, recurse with those sub-components and the
      finished-product set extended by that component.

**Message.**

> The current configuration is incorrect because it would create a cycle between these
> products: *the comma-separated display names of the finished products in the cycle*.

**Worked example.** Recipe A produces *Frame* from *Tube*. Recipe B produces *Tube* from
*Frame*. Saving B walks: components of B = {Frame}; Frame is not among B's finished products
{Tube}, so its sub-components are looked up = {Tube}; recursing with {Tube} and
{Tube, Frame} finds Tube among the finished products and refuses, naming *Tube* and
*Frame*.

### 1.3 Variant restriction constraints

**Watched fields.** `product_id`, `product_tmpl_id`, `bom_line_ids`, `byproduct_ids`,
`operation_ids`.

| Rule | Condition | Message |
|---|---|---|
| No restriction on a variant-specific recipe | The recipe names a specific variant **and** any component line, operation or by-product carries a restriction. | You cannot use the 'Apply on Variant' functionality and simultaneously create a BoM for a specific variant. |
| The restriction belongs to the recipe's template | A restriction value's product template differs from the recipe's product template. | The attribute value *the attribute value display name* set on product *the attribute value's product template display name* does not match the BoM product *the recipe's product template display name*. |

### 1.4 By-product constraints

Same watched fields.

| Rule | Condition | Message |
|---|---|---|
| A by-product is not the finished product | When the recipe names a variant: the by-product equals it. When it does not: the by-product's template equals the recipe's template. | By-product *the recipe display name* should not be the same as BoM product. |
| Non-negative cost share | A by-product's `cost_share` is strictly negative. | By-products cost shares must be positive. |
| Total cost share at most 100 | For **each** variant of the recipe's template, the sum of the cost shares of the by-product lines that are not skipped for that variant and whose quantity is not zero at their unit exceeds 100 when compared at two decimal places. | The total cost share for a BoM's by-products cannot exceed 100. |

Note that the total is checked **per variant**: two by-product lines of 60 percent each,
restricted to different mutually exclusive variants, are legal because no single variant
sees both.

### 1.5 Other recipe constraints

| Rule | Trigger | Message |
|---|---|---|
| A kit has no reordering rule | Writing `product_tmpl_id`, `product_id` or `type`: at least one reordering rule exists for a product covered by a kit recipe in the batch. | You can not create a kit-type bill of materials for products that have at least one reordering rule. |
| Positive batch size | Writing `enable_batch_size` or `batch_size`: batch sizing is enabled and the batch size is not strictly positive at the recipe unit's precision. | The batch size must be positive! |
| No deletion while orders run | Deleting a recipe for which at least one order exists in a state other than `done` or `cancel`. | You can not delete a Bill of Material with running manufacturing orders.<br>Please close or cancel it first. |
| Named creation is refused | Creating a recipe by typing a name where no default product template is supplied. | You cannot create a new Bill of Material from here. |
| A subcontracting recipe has no operation or by-product | Writing `operation_ids`, `byproduct_ids` or `type` on a recipe of kind `subcontract` that has either. | You can not set a Bill of Material with operations or by-product line as subcontracting. |
| Component cost shares are non-negative | Writing a component cost share (available where kit component cost sharing is enabled) that is negative. | Components cost share have to be positive or equals to zero. |

### 1.6 Interactive warnings on a recipe

| Trigger | Message |
|---|---|
| Changing the product variant, or the product template, while any line, operation or by-product carries a variant restriction. | **Warning** — Changing the product or variant will permanently reset all previously encoded variant-related data. |
| Changing the components, the quantity, the product or the template of a **kit** recipe whose component lines have already been used by at least one Stock Move. | **Warning** — The product has already been used at least once, editing its structure may lead to undesirable behaviours. You should rather archive the product and create a new one with a new bill of materials. |

The first warning is shown **after** the restriction fields have already been cleared: the
clearing is the behaviour, the warning merely reports it.

### 1.7 Operation constraints

| Rule | Trigger | Message |
|---|---|---|
| No cyclic operation dependency | Writing `blocked_by_operation_ids` where the resulting graph has a cycle. | You cannot create cyclic dependency. |

### 1.8 Side effects that are not refusals

- Writing any of `bom_line_ids`, `byproduct_ids`, `product_tmpl_id`, `product_id`,
  `product_qty` flags the matching orders as carrying an outdated recipe, and clears the
  flag on confirmed orders whose product no longer matches.
- Creating, writing, archiving or unarchiving an operation flags the recipe's orders
  outdated, with the unflagging pass suppressed.
- Archiving a recipe archives its operations; unarchiving unarchives them.
- Archiving an operation clears it from the component lines and by-product lines that named
  it.
- Moving an operation to another recipe clears it from the old recipe's component lines,
  by-product lines and predecessor lists.
- Archiving a product archives its recipes; unarchiving unarchives them.

---

## 2. Work Centre

| Rule | Kind | Condition | Message |
|---|---|---|---|
| A work centre is not its own alternative | Record constraint on `alternative_workcenter_ids` | The work centre appears in its own alternative list. | Workcenter *the work centre name* cannot be an alternative of itself. |
| Non-negative capacity | Database check on a capacity line | `capacity` is strictly negative. | Capacity should be a non-negative number. |
| One capacity line per product and unit | Unique index over *(work centre, product with empty treated as zero, unit)* | A duplicate is written. | Product/Unit capacity should be unique for each workcenter. |
| Unique tag name | Unique index on the tag name | A duplicate tag name is written. | The tag name must be unique. |
| Already unblocked | Operation refusal on the unblock action | The working state is not `blocked`. | It has already been unblocked. |

**Archival notice (not a refusal).** Archiving a work centre that is still referenced by at
least one operation succeeds and returns a sticky warning notice:

> Note that archived work center(s): '*the comma-separated names*' is/are still linked to
> active Bill of Materials, which means that operations can still be planned on it/them. To
> prevent this, deletion of the work center is recommended instead.

**Archival notice on a product.** Archiving a product that is still a component of an active
recipe succeeds and returns:

> Note that product(s): '*the comma-separated display names*' is/are still linked to active
> Bill of Materials, which means that the product can still be used on it/them.

---

## 3. Productivity logs

| Rule | Kind | Condition | Message |
|---|---|---|---|
| No double start | Record constraint on `workorder_id` | Two or more open logs (no end date) exist for the same user on the same Work Order. | The Workorder (*the work order display name*) cannot be started twice! |
| A productive reason must exist | Operation refusal when opening a timer within the expected duration | No Productivity Loss Reason of category `productive` exists. | You need to define at least one productivity loss in the category 'Productivity'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. |
| A performance reason must exist | Operation refusal when opening a timer beyond the expected duration, or when splitting a closing timer | No Productivity Loss Reason of category `performance` exists. | You need to define at least one productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. / You need to define at least one unactive productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. |

The two performance messages differ: the first is raised when a timer is being opened, the
second when an existing timer is being split at close.

---

## 4. Manufacturing Order — structural rules

### 4.1 Database checks

| Rule | Condition | Message |
|---|---|---|
| Unique reference per company | The pair *(reference, company)* must be unique. | Reference must be unique per Company! |
| Positive quantity to produce | `product_qty` must be strictly greater than zero. | The quantity to produce must be positive! |

### 4.2 Record constraints

| Rule | Watched fields | Condition | Message |
|---|---|---|---|
| Non-negative by-product cost share | `move_finished_ids` | Any by-product move has a strictly negative cost share. | By-products cost shares must be positive. |
| Total by-product cost share at most 100 | `move_finished_ids` | The sum of the cost shares of the non-cancelled by-product moves exceeds 100. | The total cost share for a manufacturing order's by-products cannot exceed 100. |
| At most one lot for a lot-tracked product | `lot_producing_ids` | The product's tracking is `lot` and more than one lot is set. | You cannot set more than 1 lot |
| Non-negative consumed quantity | `quantity`, `raw_material_production_id` (on the move) | A component move's consumed quantity is strictly negative at its unit. | Please enter a positive quantity. |

### 4.3 Structural refusals

| Operation | Condition | Message |
|---|---|---|
| Generate the finished moves | The order's product is also one of the recipe's by-products. | You cannot have *the product name*  as the finished product and in the Byproducts *(note the two spaces before "as", which the system emits)* |
| Generate a lot for a lot-tracked product | A producing lot is already set. | You cannot set more than 1 lot per product |
| Generate a lot or serial number | The product has no lot sequence, the shared serial sequence yields nothing, and the next serial cannot be derived. | Please set the first Serial Number or a default sequence |
| Write a product on a non-draft order | Always. | *(silently dropped — the value is removed from the write, no message)* |
| Write the start date | The order is `done` or `cancel` and the caller does not force the date. | You cannot move a manufacturing order once it is cancelled or done. |
| Set the destination of a manufacturing operation type | The destination location's usage is `inventory` (a scrap or adjustment location). | You cannot set a scrap location as the destination location for a manufacturing type operation. |

### 4.4 Deletion rules

| Operation | Condition | Message |
|---|---|---|
| Delete (checked even during uninstall) | Any order in the batch is `done`. | You cannot delete a manufacturing order that is already done. |
| Delete | Any order in the batch is `done`. | Cannot delete a manufacturing order in done state. |
| Delete | Any order in the batch is not `cancel`. | *the comma-separated display names* cannot be deleted. Try to cancel them before. |
| Delete an Unbuild Order | Any Unbuild Order in the batch is `done`. | You cannot delete an unbuild order if the state is 'Done'. |

Deleting an order first runs the cancellation and then deletes the Work Orders that are not
done, so the sequence for a running order is: cancel, remove Work Orders, then the two
delete checks above.

---

## 5. Manufacturing Order — lifecycle rules

### 5.1 Cancellation

| Operation | Condition | Message |
|---|---|---|
| Cancel | Any order is `done`. | You cannot cancel a manufacturing order that is already done. |

**Edge case — the flexible recipe.** After cancelling, an order whose recipe's consumption
policy is `flexible` and that is still neither `done` nor `cancel` is written to `done`.
The reasoning is that with a flexible recipe the move states alone cannot distinguish
"everything consumed, fewer units produced than planned, and that is fine" from "abandon the
rest": an explicit cancellation resolves the ambiguity in favour of closing the order.

**Edge case — cancelling all components.** Cancelling every component move of an order
directly, outside the order's own cancellation and without the suppression flag, cancels
the order itself.

### 5.2 Planning

| Operation | Condition | Message |
|---|---|---|
| Unplan | Any Work Order is `done`. | Some work orders are already done, so you cannot unplan this manufacturing order.<br><br>It'd be a shame to waste all that progress, right? |
| Unplan | Any Work Order is `progress`. | Some work orders have already started, so you cannot unplan this manufacturing order.<br><br>It'd be a shame to waste all that progress, right? |
| Plan a Work Order | A candidate work centre has no working schedule. | There is no defined calendar on workcenter *the work centre name*. |
| Plan a Work Order | No candidate work centre offers a slot within the horizon. | Impossible to plan the workorder. Please check the workcenter availabilities. |
| Simulate planning in the recipe report | No candidate work centre offers a slot. | Impossible to plan. Please check the workcenter availabilities. |

### 5.3 Splitting and merging

| Operation | Condition | Message |
|---|---|---|
| Split or merge | Any order is not `draft` or `confirmed`. | Only manufacturing orders in either a draft or confirmed state can be *split* / *merged*. |
| Split or merge | Any order has no recipe. | Only manufacturing orders with a Bill of Materials can be *split* / *merged*. |
| Merge | Fewer than two orders. | You need at least two production orders to merge them. |
| Merge | The orders do not all share one product **and** one recipe. | You can only merge manufacturing orders of identical products with same BoM. |
| Merge | Any order has an extra component move (no recipe line) or an extra by-product move (no by-product line). | You can only merge manufacturing orders with no additional components or by-products. |
| Merge | The orders do not all share one state. | You can only merge manufacturing with the same state. |
| Merge | The orders do not all share one operation type. | You can only merge manufacturing with the same operation type |
| Merge | Any order is a subcontracting order. | Subcontracted manufacturing orders cannot be merged. |
| Split | The supplied amounts exceed the quantity to produce, or the order is `done` or `cancel`, and the caller did not allow more. | Unable to split with more than the quantity to produce. |

### 5.4 Closing

| Operation | Condition | Message |
|---|---|---|
| Mark as done | Several orders in the selection need a lot or serial number and have none. | You need to generate Lot/Serial Number(s) to mark as done some productions |

A single such order does not raise: the system opens the lot-generation flow for it instead.

**Consumption policy gate.** When the consumption check finds any difference:

| Policy | Behaviour |
|---|---|
| `flexible` (Allowed) | The check does not run at all; the order closes silently. |
| `warning` (Allowed with warning) | The Consumption Warning assistant opens. Any manufacturing user may confirm. |
| `strict` (Blocked) | The Consumption Warning assistant opens. Only a manufacturing administrator may confirm; a plain manufacturing user can only reset the quantities to the expected values or abandon. |

The assistant's own policy is the strictest among its lines, so a selection mixing a
`warning` order and a `strict` order is gated as `strict`.

**Backorder policy gate.** The operation type decides what happens to the unproduced
remainder:

| Operation type backorder policy | Behaviour |
|---|---|
| `always` | The backorder is created without asking. |
| `ask` | The Backorder Confirmation assistant opens with one line per affected order. |
| `never` | No backorder; the order closes at the produced quantity and the remainder is dropped. |

When a selection mixes policies, the orders that force a backorder are carried through the
assistant's context and are backordered whatever the user answers for the others.

### 5.5 Expiry gate

When expiry tracking is installed, closing an order first checks the component move lines
for lots that raise an expiry alert. If any exist, the expiry confirmation assistant opens
instead of closing, showing either

> You are going to use the component *the product display name*, *the lot name* which is
> expired.
> Do you confirm you want to proceed?

for a single expired lot, or

> You are going to use some expired components.
> Do you confirm you want to proceed?

with the list of lots, for several. Confirming re-runs the closing with the expiry check
skipped.

---

## 6. Serial-number uniqueness

Run as part of the sanity checks before an order is closed. Four separate tests.

### 6.1 The finished serial number has not already been produced

Applies when the product is serial-tracked and producing lots are set. The lots to check are
those that are **not** among the lots consumed by the order's own component moves (a serial
number that was consumed and reissued is legitimate).

For those lots, the "already produced" test is:

1. Count the done move lines of quantity exactly 1 carrying one of those lots whose **source**
   location has production usage and whose move is not an unbuild move. Call this
   *duplicates*.
2. If *duplicates* is zero, the test is negative.
3. Otherwise count:
   - *duplicates from unbuild*: done move lines of quantity 1 carrying one of those lots,
     with no production order, whose destination location has production usage, and whose
     move **is** an unbuild move;
   - *removed*: done move lines carrying one of those lots whose source usage is not
     `inventory` and whose destination usage is `inventory`;
   - *unremoved*: done move lines carrying one of those lots whose source usage is
     `inventory` and whose destination usage is not `inventory`.
4. The test is negative — that is, the serial number is free — exactly when
   *(duplicates from unbuild or removed)* is non-zero **and**

   ```formula
   duplicates − duplicates_from_unbuild − removed + unremoved = 0
   ```

   Otherwise the test is positive.
5. Finally, a serial number already present with a non-zero quantity on this order's own
   finished move lines also makes the test positive.

**Message when positive:**

> Serial number(s) for product *the product name* already produced

### 6.2 A by-product serial number has not already been produced

For every finished move that is serial-tracked and is **not** for the order's own product,
and for every one of its move lines with a non-zero quantity, the same "already produced"
test is run on that line's lot, with that line excluded from the current-order check.

**Message:**

> The serial number *the lot name* used for byproduct *the product name* has already been
> produced

### 6.3 No component serial number is used twice on this order

For every component move that is serial-tracked and picked, and for every one of its picked
move lines with a non-zero quantity and a lot: if any **other** component move line of the
same order has a non-zero quantity and the same lot, refuse.

**Message:**

> The serial number *the lot name* used for component *the product name* has already been
> consumed

### 6.4 No component serial number has already been consumed elsewhere

For the serial numbers collected in §6.3:

1. Sum, per lot, the quantities of done move lines of quantity exactly 1 carrying that lot
   whose destination usage is `production` and that belong to a production order. Call this
   *consumed*.
2. Sum, per lot, the quantities of done move lines of quantity exactly 1 carrying that lot
   whose **source** usage is `production` and that either belong to no production order, or
   belong to a production order whose product is this order's product. Call this
   *cancelled*.
3. For each lot, when `consumed − cancelled > 0`, refuse with the message of §6.3 for that
   lot.

The *cancelled* term is what lets a serial number be consumed, unbuilt and consumed again.

---

## 7. Work Order rules

### 7.1 Refusals

| Operation | Condition | Message |
|---|---|---|
| Change the produced quantity | The Work Order is `done` or `cancel`. | You cannot change the quantity produced of a work order that is in done or cancel state. |
| Change the produced quantity | The value compares as negative at the order unit. | The quantity produced must be positive. |
| Change the order | The new order differs from the current one. | You cannot link this work order to another manufacturing order. |
| Change the work centre | The Work Order is `done` or `cancel`. | You cannot change the workcenter of a work order that is done. |
| Save with the start after the finish | Both dates set and the start is later. | The planned end date of the work order cannot be prior to the planned start date, please correct this to save the work order. |
| Clear the planned finish while a start remains | Either on writing the slot or interactively. | It is not possible to unplan one single Work Order. You should unplan the Manufacturing Order instead in order to unplan all the linked operations. |
| Start | Any work centre in the batch is `blocked`. | Please unblock the work center to start the work order. |
| Start | The Work Order is `done` or `cancel` and the caller does not tolerate it. | You cannot start a work order that is already done or cancelled |
| Mark as done | The work centre is `blocked`. | Please unblock the work center to validate the work order |
| Create a dependency | The graph would contain a cycle. | You cannot create cyclic dependency. |

### 7.2 Interactive warnings on a Work Order

The popover shown on a planned Work Order in state `blocked` or `ready` carries, in order:

| Condition | Colour | Message |
|---|---|---|
| The state is `blocked`, the earliest predecessor start exists, and it is **not** after this Work Order's start. | primary | Waiting the previous work order, planned from *the earliest predecessor start* to *the latest predecessor finish* |
| The planned finish is before the current instant. | warning | The work order should have already been processed. |
| The earliest predecessor start is after this Work Order's start. | danger | Scheduled before the previous work order, planned from *the earliest predecessor start* to *the latest predecessor finish* |
| Another Work Order in state `blocked` or `ready` occupies the same work centre over an overlapping window (both windows truncated to whole seconds). | danger | Planned at the same time as other workorder(s) at *the work centre display name* |

The popover's icon is a warning triangle when the last colour is warning or danger, and an
information circle otherwise. A replan control is offered unless there is no message or the
only colour is primary.

### 7.3 Invariants

1. A Work Order's ready quantity is zero when it is `done` or `cancel`.
2. A Work Order's state is `ready` exactly when its ready quantity is strictly positive,
   for as long as it stays in the `blocked`/`ready` pair.
3. A Work Order with a calendar slot always has both a planned start and a planned finish.
4. Deleting a Work Order rewires its predecessors directly to its successors, so the
   dependency graph stays connected and acyclic.
5. Writing a positive produced quantity on any Work Order of an order sets the producing
   quantity of every unfinished Work Order of that order to the minimum produced quantity
   across its Work Orders, when that minimum is positive, and pushes the value back to the
   order.

---

## 8. Unbuild Order rules

| Rule | Kind | Condition | Message |
|---|---|---|---|
| Positive quantity | Database check | `product_qty` is not strictly positive. | The quantity to unbuild must be positive! |
| A lot is required for a tracked product | Operation refusal | The product's tracking is not `none` and no lot is set. | You should provide a lot number for the final product. |
| The source order must be done | Operation refusal | A source order is named and its state is not `done`. | You cannot unbuild a undone manufacturing order. |
| Tracked components need a source order | Operation refusal | Any produce move or by-product consume move is tracked and no source order is named. | Please specify a manufacturing order.<br>It will allow us to retrieve the lots/serial numbers of the correct components and/or byproducts. |
| No deletion when done | Deletion check | The Unbuild Order is `done`. | You cannot delete an unbuild order if the state is 'Done'. |

**Insufficient stock (not a refusal).** When the available quantity of the product at the
source location for the chosen lot is less than the quantity to unbuild converted into the
product's own unit, the insufficient-quantity warning opens instead of performing the
unbuild. Its title is:

> *the product display name*: Insufficient Quantity To Unbuild

Confirming it performs the unbuild anyway.

**Edge case — repeated unbuilds of the same order.** The factor of an unbuild is always
`unbuild_quantity ÷ order_produced_quantity`, never against the remaining quantity.
Consecutive unbuilds of 2 and 3 from an order of 10 therefore return exactly the components
of 5 units. With serial-tracked components, the lots already restored by an earlier unbuild
of the same order are excluded from the matching, so no serial number is returned twice.

---

## 9. Kit rules

| Rule | Kind | Condition | Message |
|---|---|---|---|
| A kit product may not be counted directly | Record constraint on the Stock Quantity's product | The product has a kit recipe. | You should update the components quantity instead of directly updating the quantity of the kit product. |
| A kit product may not have a reordering rule | Record constraint on the reordering rule's product | A kit recipe exists for the product (variant-specific or template-wide) in the rule's company or in no company. | A product with a kit-type bill of materials can not have a reordering rule. |
| A kit recipe may not be created for a product with a reordering rule | Record constraint on the recipe | See §1.5. | You can not create a kit-type bill of materials for products that have at least one reordering rule. |

**Other kit behaviours that are not refusals.**

- A kit product's moves always bypass reservation.
- A kit product is excluded from the quantity-tracking machinery, so no quant is created for
  it.
- A kit product is valued at zero: it is excluded from the inventory-valuation product
  domain and its total value and average cost are forced to zero, so that its components are
  not counted twice.
- A kit product is excluded from the products the scheduler considers, in batches of 2000.
- Editing a kit recipe whose lines have already been used by a Stock Move raises the
  interactive warning of §1.6.

---

## 10. Locking rules

| Lock | Scope | Effect |
|---|---|---|
| Order locked flag (`is_locked`) | One order | A done, locked order cannot have its produced quantity changed. Every component and finished move of a locked order is itself locked. The default is true unless the user belongs to the unlocked-by-default group. |
| Automatic lock on completion | One order | Marking an order done always writes the locked flag to true, whatever the user's group. |
| Setting change | All running orders | Turning the unlocked-by-default setting on immediately unlocks every order that is neither `done` nor `cancel`; turning it off locks them all. |
| Lock control visibility | One order | Shown when the order is `done`, or when the user is **not** in the unlocked-by-default group, the record is saved, and the state is neither `cancel` nor `draft`. |
| Reservation exclusivity | One component move | A component move marked picked can no longer be unreserved through the order's unreserve control. |
| Produced-quantity change guard | One Work Order | A done or cancelled Work Order refuses a change of produced quantity, of work centre, or of order. |

---

## 11. Permission rules

### 11.1 Group membership

| Group | Implies |
|---|---|
| Manufacturing / User | Inventory / User |
| Manufacturing / Administrator | Manufacturing / User |

Four further groups are technical switches, implied by the corresponding settings:
*Manage Work Order Operations*, *Produce residual products*, *Unlocked by default*,
*Use Reception Report with Manufacturing Orders* and *Use Operation Dependencies*.

### 11.2 Checks embedded in behaviour

| Check | Where | Consequence |
|---|---|---|
| Manufacturing administrator | The Consumption Warning assistant with policy `strict` | Only an administrator may confirm and close. |
| Allocation group | Order completion and the allocation visibility computation | Without it the allocation control is never shown and the allocation report is never printed automatically. |
| Lot group | Automatic printing of lot labels | Without it the lot labels are not printed. |
| Unlocked-by-default group | The default of the locked flag and the visibility of the lock control | See §10. |
| By-products group | The attachments shown on a recipe | Without it the by-products' product documents are not included. |
| Product-variant group | The default variant of the recipe view and the attachment filter | Display only. |
| Portal user | Writes to a subcontracting order and to Stock Moves | See §12. |

### 11.3 Company consistency

Company consistency is enforced automatically between a record and every company-checked
relation it carries. It is additionally re-checked explicitly:

- when an order is confirmed;
- when an order is marked done (as part of the sanity checks);
- when an unbuild is performed.

Recipes, recipe lines, by-product lines, operations, work centres and work centre capacities
may carry an **empty** company, which means "all companies", and their record rules accept
an empty company. Orders, Work Orders, Unbuild Orders and Productivity Logs must carry a
company and their record rules do not accept an empty one.

---

## 12. Subcontracting rules

| Rule | Kind | Condition | Message |
|---|---|---|---|
| A subcontracting recipe has no operation or by-product | Record constraint | See §1.5. | You can not set a Bill of Material with operations or by-product line as subcontracting. |
| Subcontracted orders cannot be merged | Operation refusal | Any order in the merge has a subcontract receipt behind it. | Subcontracted manufacturing orders cannot be merged. |
| A portal user writes only the allowed fields | Operation refusal on writing an order | The acting user is a portal user, not acting with elevated rights, and writes any field outside the allowed set (`move_line_raw_ids`, `lot_producing_ids`, `qty_producing`, `product_qty`). | You cannot write on fields *the comma-separated field list* in mrp.production. |
| A portal user does not post stock | Operation refusal on creating or writing a Stock Move | The acting user is a portal user, not acting with elevated rights, and the state written or defaulted is `done`. | Portal users cannot create a stock move with a state 'Done' or change the current state to 'Done'. |
| Splitting a subcontracting order needs a lot | Operation refusal | No producing lot is set. | Please set a lot/serial for the currently opened subcontracting MO first. |
| Splitting after receipt | Operation refusal | The receipt move is already `done`. | The subcontracted goods have already been received. |
| The company's subcontracting location is protected | Record constraint on the location | An attempt is made to change the company of the company's own subcontracting location. | You cannot alter the company's subcontracting location |
| A subcontracting location is internal and company-bound | Record constraint on the location | A subcontractor location is not of internal usage or is not linked to the right company. | In order to manage stock accurately, subcontracting locations must be type Internal, linked to the appropriate company. |

**Advisory notice (not a refusal).** Recording a component consumption at a subcontractor
whose resupply transfer has not been validated shows:

> Make sure you validate or adapt the related resupply picking to your subcontractor in
> order to avoid inconsistencies in your stock.

**Behavioural rules.**

- A subcontract receipt move always bypasses reservation and is excluded from the
  available-move-line computation.
- A subcontract receipt move's quantity is editable only when the product is untracked;
  for a tracked product the quantities come from the lot lines.
- Cancelling a subcontract receipt move cancels the subcontracting orders behind it that
  are neither `done` nor `cancel`, except those that also serve a move that is being kept.
- Closing a subcontracting order skips the consumption check entirely.
- A subcontracting order reports having no Work Orders, whatever its recipe says, because a
  subcontracting recipe may not have operations.
- A subcontracting order never postpones its finish date by the work-centre-availability
  rule.
- Copying a subcontract receipt move, without an explicit source location, resets the source
  to the transfer's own source location.

---

## 13. Product and unit rules

| Rule | Kind | Condition | Message |
|---|---|---|---|
| A unit may not be changed once used | Operation refusal on changing a product's unit | A recipe, a recipe line or an order already uses a different unit for that product. | As other units of measure (ex : *the other unit name*) than *the product's unit name* have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| A component lot may not be created at a restricted operation type | Operation refusal on creating or editing a lot | The caller's context names an active order whose operation type does not allow creating component lots, and the lot's product is one of that order's components. | You are not allowed to create or edit a lot or serial number for the components with the operation type "Manufacturing". To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers for Components". |

When the units **do** agree, changing the product's unit rewrites the unit of every recipe,
recipe line and order that used it.

---

## 14. Invariants

These statements must hold at every commit boundary.

### 14.1 Structural

1. Every component move of an order belongs to that order (`raw_material_production_id`),
   and every finished or by-product move belongs to it (`production_id`); no move carries
   both.
2. A component move's destination is the product's production location for the order's
   company; a finished move's source is that same location.
3. Every move of an order carries the order's production group; the transfers of an order
   are exactly the transfers of the moves carrying that group.
4. An order has exactly one finished move for its own product per split generation; several
   may exist after a split or a merge, and the produced quantity is then distributed among
   them in proportion to their unit factors.
5. The sum of the quantities to produce over a production group equals the quantity to
   produce of the original order before the first split, unless a split deliberately
   cancelled the remainder.
6. A recipe of kind `phantom` never appears as the recipe of a Manufacturing Order: the
   order's recipe domain restricts to `normal`.
7. A recipe of kind `subcontract` never carries an operation or a by-product line.
8. No product graph reachable through recipes contains a cycle.

### 14.2 Quantitative

1. `qty_produced` is the sum of the picked, non-cancelled finished-move quantities for the
   order's own product.
2. `qty_remaining` of a Work Order is
   `max(round(qty_production − qty_reported_from_previous_wo − qty_produced), 0)` and is
   therefore never negative.
3. The total by-product cost share of an order never exceeds 100, and the finished product
   always takes `round_to_4_decimals(1 − share ÷ 100)` of the production cost, so the
   allocation is exhaustive.
4. A component move's unit factor is never a division by zero: the denominator is floored
   at 1.
5. The should-consume quantity of a component move is
   `round_at(move_unit, (qty_producing − qty_produced) × unit_factor)` and follows the
   producing quantity exactly.
6. A kit product's on-hand, forecast, incoming, outgoing and free quantities are always
   whole multiples of the kit's recipe quantity, because the minimum ratio is floored.

### 14.3 Temporal

1. A planned Work Order's window is exactly its calendar slot's window.
2. The start of a planned order is the earliest slot start among its unfinished Work
   Orders; its finish is the latest slot end.
3. A component move's date and deadline equal the order's start date.
4. A finished move's date equals the order's finish date and its deadline equals the order's
   deadline.
5. An order's deadline is the minimum of its finished moves' deadlines, when any has one.
6. A subcontracting order's component and finished moves are dated one second before the
   earliest receipt move line, so that the traceability report shows production before
   receipt.

### 14.4 Accounting

1. Every finished and by-product move of a closed order carries a unit price, except a
   by-product with a zero cost share under first-in-first-out or average costing, whose
   price is left to the ordinary incoming valuation.
2. The sum of the values of the finished and by-product moves of a closed order equals the
   production cost, up to the four-decimal rounding of the complement share.
3. A kit product contributes no valuation of its own; only its components are valued.
4. A work-in-progress journal entry is always paired with a reversal dated strictly after
   it.

---

## 15. Edge cases and their resolutions

| Situation | Resolution |
|---|---|
| A component appears twice in an exploded recipe (two lines, or two branches of nested kits). | Both leaves are recorded separately and produce two component moves. The consumption check sums them per product. The kit-quantity computation sums the per-kit requirement per component. |
| A component line has a quantity of zero. | It is an optional component: it produces a move of demand zero, and it is skipped by every division-by-quantity computation (kit quantities, on-hand derivation, delivered quantity). |
| A component is a service. | It never produces a component move and is skipped by the kit-quantity computation. |
| A recipe has no component lines at all. | The consumption check is skipped entirely for that order. |
| The order has no recipe. | No component moves and no Work Orders are generated; the consumption policy defaults to `flexible`; the recipe cost column of the overview report falls back to the order cost. |
| The quantity producing exceeds the quantity to produce. | The order becomes `to_close`; the backorder quantity is `max(product_qty − qty_producing, 0)` = 0, so no backorder is offered; the extra units are produced. |
| A backorder is created from an order whose name already ends in a hyphen and digits. | The suffix is replaced rather than appended, but only when the group's maximum backorder sequence is above 1 or the new sequence is above 1. |
| An order is split into more pieces than 999. | The zero padding is computed as `3 − 1 − floor(log10(sequence))`, which becomes negative and adds no zeros; the suffix is simply the hyphen and the number. |
| A move's demand is zero when a kit must be exploded. | The explosion factor is computed from the **done** quantity instead, and the generated moves carry the exploded quantity as their done quantity rather than their demand. |
| A finished move already carries a done quantity when the order is closed. | The distribution writes `(qty_producing − qty_produced) × unit_factor`, so the already-produced part is not counted twice. |
| A tracked component's upstream supply is short. | The distribution caps the consumed quantity at `available − taken`, where *available* is the sum of the done origin quantities and *taken* the sum already consumed by sibling moves. |
| A manual-consumption move is already picked when the producing quantity changes. | It is skipped by the distribution; its value stands. |
| A by-product move is already picked. | It is never rewritten by the distribution. |
| A work centre's efficiency is zero or empty. | The operation total-duration formula falls back to 100 percent. |
| A capacity line has a capacity of zero. | The default capacity is used, but the line's setup and cleanup times still apply. |
| An operation in computed-duration mode has no finished Work Order yet. | The cycle duration falls back to the manual duration. |
| Two Productivity Logs of the **same** category overlap. | Their durations are merged and counted once. |
| Two Productivity Logs of **different** categories overlap. | Both are counted, because the merge is done per category. |
| A blocking log spans a period when the work centre is closed. | Its duration counts only the working hours of the work centre's schedule; a productive or performance log counts wall-clock time. |
| A Work Order's real duration is written down to a smaller value. | Whole logs are removed from the start of the natural ordering while the amount to remove covers them, and the first log that is larger is shortened by moving its start forward. |
| An order is confirmed with a serial-tracked product and an order unit different from the product's unit. | The quantity and unit are converted to the product's own unit, and so is the finished move — a serial-tracked product is always produced in whole reference units. |
| An order's operation type is changed on a running order. | A new reference is allocated from the new type's sequence, the matching Stock Reference is renamed, and every component move is unreserved and re-reserved. |
| A recipe is updated while an order is running. | The order is flagged as carrying an outdated recipe; applying the update rewrites the components, by-products and Work Orders, preserving the Work Orders that are in progress, done or cancelled. |
| A recipe is updated and the order's product no longer matches. | The outdated flag is **cleared** on confirmed orders whose product or product template no longer matches the recipe. |
| Cancelling a move of a running order in a two- or three-step configuration. | The move's demand is set to zero before it is cancelled, so the upstream pick shrinks rather than being orphaned. |
| An order for a product with a kit recipe. | The recipe domain of an order only offers `normal` recipes, so a kit product can only be ordered through its components. |
| A procurement for a kit product. | It is replaced, before any rule runs, by one procurement per exploded leaf component. |
| A negative procurement reaches the manufacture rule. | No order is created. |
| A recipe enables batch sizing and the procured quantity is not a multiple of the batch. | Orders of the batch size are emitted repeatedly until the quantity is covered; the last order therefore over-produces by up to one batch minus one. |
| An order created by a rule has no responsible. | This is deliberate: it is what makes the order eligible to absorb a later procurement of the same kind. |
| An order is created by the scheduler from a reordering rule while a draft order already exists. | Draft orders created by the reordering rules are confirmed only **after** every reordering rule has run, so that the procurements they spawn do not conflict with the rules still to run. |
