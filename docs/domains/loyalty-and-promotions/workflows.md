# Workflows

Every operational workflow of the Loyalty, Coupons and Promotions domain, end to end: the actors, the preconditions, the numbered steps, the decisions and branches, the records created or modified with the field values written, the notifications sent and the postconditions. The state tables of every lifecycle in the domain close the file.

Rules cited as `LOY-RULE-nnn` are defined in [business-rules.md](business-rules.md); formulas cited by number are defined in [calculations.md](calculations.md).

## 1. Configuration workflows

### 1.1 Create a discount or loyalty program

**Actor**: Sales Administrator, or Point of Sale Administrator when the program is meant for a counter.
**Preconditions**: the loyalty capability package is enabled; at least one sellable product exists when a template that needs one is chosen.

1. The actor opens the discount and loyalty list and asks for a new program. The creation screen offers the seven templates of section 1.9 of [entities.md](entities.md).
2. The actor picks a template. The system creates a Loyalty Program with the template's name, the template's `program_type` and the family defaults of that type: `applies_on`, `trigger`, `portal_visible`, `portal_point_name`, one or more Loyalty Rules, one or more Loyalty Rewards and zero or more Loyalty Communications.
3. Each created Loyalty Reward immediately receives a hidden discount product: a service product named after the reward description, not sellable, not purchasable, list price zero, no customer taxes, no vendor taxes, invoicing policy "ordered quantities".
4. The program form opens. The actor edits the name, the currency, the validity dates, the usage limit, the channel flags, the pricelist restriction, the website restriction and the counter restriction.
5. The actor edits the rules: minimum quantity, minimum purchase and its tax mode, the product filter, the promotional code, and the point grant with its mode.
   - Writing a code sets `mode` to `with_code`; clearing it sets `mode` to `auto` and clears the code.
   - Writing a code regenerates `promo_barcode`.
6. The actor edits the rewards: the reward type and all of its parameters, the point price, and the "clear the whole balance" flag.
   - Editing the description renames the hidden discount product and propagates the translations.
7. The actor edits the communication plan when `applies_on` is not `current` and the program is not a gift card or an electronic wallet program: rows of (trigger, milestone, email template).
8. Saving runs the validations of `LOY-001` to `LOY-010`.

**Postcondition**: an active program that every enabled channel will start evaluating on its next document update.

**Records written**: one Loyalty Program, its Loyalty Rules, its Loyalty Rewards, its Loyalty Communications, and one Product Variant per reward.

### 1.2 Create a gift card or electronic wallet program

**Actor**: Sales Administrator or Point of Sale Administrator.
**Preconditions**: a trigger product exists that represents the card or the top-up, sold as a service at the face value.

1. The actor opens the gift card and electronic wallet list and asks for a new program, choosing the gift card or the electronic wallet template.
2. The system creates the program with the family defaults: `applies_on` `future`, `trigger` `auto`, `portal_visible` true, `portal_point_name` the company currency symbol, one rule granting one point per unit of currency spent on the shipped trigger product (with the split option on for a gift card and off for an electronic wallet), one reward of mode `per_point` worth one unit of currency per point applicable to the order, and, for a gift card, one communication rule sending the shipped gift card email at creation.
3. The actor replaces the trigger products with the company's own gift card or top-up products. Writing `trigger_product_ids` writes the product filter of the rules.
4. The actor optionally sets the email template and, for a counter, the printed document. Setting the printed document before the email template is refused with `You must set 'Email template' before setting 'Print Report'.`
5. For an electronic wallet program on a website, a warning banner appears when at least one trigger product is not published; the shopper could otherwise never top up.

**Postcondition**: buying a trigger product now issues a card; the card can then pay for orders.

### 1.3 Archive and unarchive a program

**Actor**: Sales Administrator.

1. The actor archives the program. `active` becomes false.
2. The cascade archives every rule, every reward, every communication rule and every reward discount product of the program.
3. Every document that carries a reward line of that program loses it at its next evaluation, because the program no longer satisfies the program filter.
4. Unarchiving reverses the cascade. Before it completes, the code uniqueness rules are re-run; when another active rule already carries one of the codes being reactivated, the unarchive fails with `The promo code must be unique.` When several programs are unarchived at once and two of them share a code, the same message is raised.

**Postcondition**: the program is invisible to every channel while archived.

### 1.4 Delete a program

1. Deleting an active program is refused with `You can not delete a program in an active state`.
2. After archiving, deletion is still refused when at least one card references the program, because the card's reference to the program is a restricted reference.
3. When the program has no card, deletion removes the program and cascades to its rules, rewards and communication rules. The reward discount products survive as ordinary archived products.

## 2. Card workflows

### 2.1 Generate cards

**Actor**: Sales Administrator, Salesperson, or Point of Sale Cashier.
**Preconditions**: the program exists.

1. From the program screen the actor presses the generation button. Its label is `Generate Coupons` for a `coupons` program, `Generate Gift Cards` for a `gift_card` program and `Generate eWallet` for an `ewallet` program; the electronic wallet button opens the wizard with `mode` already set to `selected`.
2. The actor chooses `anonymous` or `selected`.
   - In `anonymous` mode the actor types a quantity.
   - In `selected` mode the actor picks customers and customer tags; the quantity is recomputed as the number of resolved customers. **When both lists are left empty the resolution matches every contact in the database**, so the quantity becomes the total number of contacts.
3. The actor types the grant (the starting balance), optionally a validity limit and optionally a description.
4. The wizard shows `You're about to generate <program type label> with a value of <grant> for <quantity> customers`, and a warning when emails will be sent.
5. Pressing the confirm button runs `generate_coupons`:
   1. Refuse with `Can not generate coupon, no program is set.` when no program is set.
   2. Refuse with `Invalid quantity.` when the quantity is zero or negative.
   3. Create one Loyalty Card per customer (or per unit of quantity in anonymous mode) with `program_id`, `points` equal to the grant, `expiration_date` equal to the validity limit and `partner_id` equal to the customer or empty.
   4. Each card receives a generated unique code.
   5. The "at creation" communication plan runs for every card that has a recipient.
   6. One Loyalty History Entry per card: `issued` equal to the grant, `used` zero, `description` equal to the typed description or `Gift For Customer`.

**Postcondition**: the cards exist, are listed behind the program's card button, and, when the program has an "at creation" plan and the cards have owners, their owners have received them by email.

### 2.2 Adjust the balance of a card

**Actor**: Salesperson or Point of Sale Cashier.

1. The actor opens the card and presses the balance, which opens the balance update wizard.
2. The wizard shows the old balance and asks for the new balance and a mandatory description.
3. Confirming:
   1. Refuse with `New Balance should be positive and different then old balance.` when the new balance equals the old one or is negative.
   2. Create a Loyalty History Entry: `issued` equal to the positive difference, or `used` equal to the absolute negative difference; `description` equal to the typed text, or `Gift for customer` when empty.
   3. Write the new balance into `points`. This write is tracked in the card's discussion thread and triggers the milestone communications.

### 2.3 Send a card by email

**Actor**: Salesperson or Sales Administrator.

1. The actor presses the send button on the card list, or the send action on the card form.
2. The system opens a message composition window pre-filled with the default template of the card (section 4.3 of [entities.md](entities.md)), the composition mode "comment", the light notification layout, and the forced-email flag.
3. The actor edits and sends. The message is posted on the card's discussion thread and delivered to the resolved recipient.

### 2.4 Share a coupon or a program by link

**Actor**: Sales Administrator.
**Preconditions**: the website capability package is enabled; the program is of type `coupons`, or its `trigger` is `with_code`, or one of its rules carries a code.

1. From a card, the actor presses the share action; from a program, the actor presses the program share action. Calling the share action with both a card and a program, or with neither, is refused with `Provide either a coupon or a program.`
2. The wizard opens, titled `Share ` followed by the family noun of the program.
3. The website defaults to the program's website, or to the only website when there is exactly one.
4. The code shown is the card's code when a card is shared, and the code of the program's first coded rule otherwise.
5. The actor edits the landing page (default `/shop`).
6. The link is `<website base address>/coupon/<code>?r=<landing page>`.
7. Pressing the short-link action reopens the wizard in short-link mode, where an existing tracked link for that address is reused or a new one created, and the shortened address is shown instead.
8. Validations: a `coupons` program requires a card (`A coupon is needed for coupon programs.`); a program restricted to a website may only be shared on that website (`The shared website should correspond to the website of the program.`).

### 2.5 Archive a card

1. The actor archives the card.
2. Before the archive is written, every pending point promise that links the card to a **draft** sales order is deleted, so that no quotation keeps promising points to a card that is no longer usable.
3. `active` becomes false. The card disappears from every default query, from the applicability searches and from the customer's portal.

### 2.6 Merge two contacts

**Actor**: Access Rights Administrator, through the contact merge wizard.

1. Before the generic reference rewrite, the nominative cards of the source contacts are collected: cards whose program has `applies_on` equal to `both`, or whose program is of type `ewallet` or `loyalty` with `applies_on` equal to `future`.
2. The cards are grouped by program. For each program:
   1. Sum the balances of the source cards.
   2. Look for a card of the same program already owned by the destination contact. When one exists, it becomes the surviving card and its own balance is added to the sum; otherwise the first source card becomes the surviving card.
   3. Write the destination contact and the summed balance onto the surviving card.
   4. Write balance zero and `active` false onto every other card of that group.
3. The generic reference rewrite then proceeds for every other reference.

**Postcondition**: the destination contact owns exactly one card per nominative program, carrying the total balance; the emptied duplicates are archived.

**Worked example**: contacts Alpha and Beta are merged into Gamma. Alpha owns a loyalty card of 120 points and an electronic wallet of 30.00; Beta owns a loyalty card of 45 points; Gamma already owns a loyalty card of 10 points. After the merge Gamma owns a loyalty card of `120 + 45 + 10 = 175` points and an electronic wallet of 30.00; Alpha's and Beta's loyalty cards are archived with balance zero, and Alpha's wallet survives as Gamma's wallet.

## 3. The sales order evaluation workflow

### 3.1 Re-evaluate an order

This is the routine that every other sales workflow calls. It is idempotent: running it twice on an unchanged order produces the same document.

**Actor**: the system, triggered by a confirmation, a reward claim, a code application, a price recomputation, a cart update or a manual reward refresh.
**Preconditions**: exactly one order.

**Step 1 - collect the programs to examine**

1. When the order allows nominative programs (`LOY-063`), search for cards that are not yet among the order's applied cards, that are owned by the order's customer, that have a strictly positive balance, and whose program is either an electronic wallet program or a loyalty program whose `applies_on` is not `current`. Add every such card to `applied_coupon_ids`. This is what silently puts a customer's wallet and loyalty card on every one of their orders.
2. `points_programs` are the programs of the cards named by the order's pending promises that carry a non-zero number of points.
3. `coupon_programs` are the programs of the order's applied cards.
4. `automatic_programs` are the programs matching the automatic candidate filter (section 3.3 of [calculations.md](calculations.md)) that are not already in `points_programs`.
5. `all_coupons` is the union of the cards of the pending promises and the applied cards.
6. Every program of the three sets is checked against the program filter. A program that fails it is marked with an error without further evaluation; the others are evaluated by the point computation (section 4 of [calculations.md](calculations.md)).
7. Applied cards whose expiration date is strictly before the order's reference date are removed from `applied_coupon_ids`, and every line referencing them is scheduled for deletion.
8. For every pending promise:
   - when the card's owner is the public customer and the order's customer is not, the card's owner is rewritten to the order's customer (this is how a card created for an anonymous online cart becomes the shopper's card once they identify themselves);
   - when the card has an owner different from the order's customer, the promise is set to zero points and scheduled for deletion;
   - otherwise the promise is kept and grouped by program.

**Step 2 - update the programs already applied**

For every program of `points_programs`:

- **When the program is in error** (no longer applicable, or its rules are no longer met):
  1. The cards of that program that were created by this order are removed from `all_coupons`.
  2. Every line referencing those cards is completely reset (point cost zero, unit price zero, reward and card cleared) and scheduled for deletion.
  3. When the program is not nominative, those cards are scheduled for deletion; when it is nominative, only the pending promises are scheduled for deletion, after being set to zero points.
  4. The program's rules are removed from the order's activated code rules.
- **When the program is still applicable**:
  1. Take the non-zero point values of the result; when the result yields none and the program is nominative, use the single value zero, so that the customer's card stays attached.
  2. Pair the existing promises with the values in order and rewrite their points.
  3. When there are more values than promises, create one Loyalty Card per missing value: `program_id` the program, `partner_id` the order's customer when the program type is `next_order_coupons` and empty otherwise, `points` zero, `order_id` this order. Creation runs with the loyalty email suppression and tracking suppression flags. Then create the matching promises.
  4. When there are more promises than values, the surplus promises are set to zero points, their cards are removed from `all_coupons` and both are scheduled for deletion.

For every program of `coupon_programs` (programs attached by a code): when the program no longer matches the program filter, or when its `applies_on` is `current` and its rules are no longer met, every line referencing its cards is completely reset and scheduled for deletion, and its cards are detached from `applied_coupon_ids` and from `all_coupons`.

**Step 3 - rebuild the reward lines**

1. Every line that has both a reward and a card is put into a **pool** and reset to a neutral state: point cost zero, unit price zero, technical unit price zero. The reward and the card are kept, so the line can be reused.
2. Walk the order lines and collect one entry per distinct reward grouping code: the reward, the card, the grouping code and the product. Entries whose reward belongs to a payment program are collected separately.
3. Process the ordinary entries first, then the payment entries (section 16 of [calculations.md](calculations.md)). For each entry:
   - skip it when its card is no longer in `all_coupons`, when the points available on the card are less than the reward's required points, or when the reward's program no longer matches the program filter; the pool lines of that entry will simply be deleted at the end;
   - otherwise compute the reward line values. When the computation refuses with `There is nothing to discount`, treat the result as an empty list.
   - Write the values over the pool lines: pair each value with a pool line and update it in place (preserving the line description when the product is unchanged), create extra lines when there are more values than pool lines, and leave the surplus pool lines in the pool.
4. Whatever remains in the pool is scheduled for deletion.

**Step 4 - apply the new automatic programs**

For every automatic candidate program that is not in error, attach it to the order: create its cards and its pending promises (section 3.4). No reward is claimed at this step; the rewards become claimable and are offered to the user.

**Step 5 - clean up**

Delete the scheduled lines in a single write, then the scheduled cards, then the scheduled promises. The deletions are deferred to the end because each one invalidates the computation caches.

**Postcondition**: the order carries exactly the reward lines that its current content justifies, the pending promises match the points its current content earns, and no orphan card or promise is left behind.

### 3.2 Attach a program to an order

Used by step 4 above and by the code application.

**Inputs**: a program, optionally a card, and the point computation result.

1. `points` is the first value of the result.
2. **When a card is given**: when the program is nominative, record a pending promise of `points` towards that card. Return the card.
3. **When no card is given and the program is nominative**: search for a card of that program owned by the order's customer.
   - When none exists and `points` is zero, refuse with `No card found for this loyalty program and no points will be given with this order.`
   - When one exists, record a pending promise of `points` towards it and return it.
4. **When no card is given and none was found**: keep the non-zero values of the result; create one card per value with `program_id` the program, `partner_id` the order's customer when the program is nominative or of type `next_order_coupons` and empty otherwise, `points` zero and `order_id` this order; record one promise per card with its value. Return the cards.

Cards created here are created with elevated rights, with loyalty emails suppressed and with tracking suppressed, because a quotation must be editable by a salesperson who has no right to create cards.

### 3.3 Apply a program with the full check

1. Refuse with `The program is not available for this order.` when the program does not match the program filter.
2. Refuse with `This program is already applied to this order.` when the program is already among the applied programs. The refusal is flagged as "already applied" so that callers can distinguish it.
3. When the program has rewards, determine its best global discount: when it has more than one global discount reward, the one with the largest discount amount against the order's discountable amount; otherwise the single one, if any. When that reward exists, a global discount is already applied, and the applied one is at least as good (section 7 of [calculations.md](calculations.md)), refuse with `This discount (<candidate description>) is not compatible with "<applied description>". Please remove it in order to apply this one.`
4. Run the point computation. When it returns an error, return that error.
5. Otherwise attach the program (section 3.2).

### 3.4 Apply a code to a sales order

**Actor**: Salesperson through the coupon code wizard, shopper through the online promotional code form, or the system through a pending coupon link.

1. Search among the rules that match the rule filter for one whose `mode` is `with_code` and whose `code` equals the typed text.
2. When such a rule exists and it is already among the order's activated code rules **and** its program already has a reward line on the order, refuse with `This promo code is already applied.`
3. **When no rule matched**, search for a Loyalty Card whose `code` equals the typed text.
   - When there is none, or its program is archived, or its program has no reward, or its program does not match the program filter, refuse with `This code is invalid (<code>).` and flag the result as "not found" so that the caller can fall back to interpreting the text as a pricelist code.
   - When the card has an expiration date strictly before the order's reference date, refuse with `This coupon is expired.`
   - When the card's balance is strictly less than the smallest required points among its program's rewards, refuse with `This coupon has already been used.`
   - Otherwise the program is the card's program.
4. When there is still no program, or the program is archived, refuse with `This code is invalid (<code>).` flagged as "not found".
5. When the program type is `loyalty` or `ewallet`, refuse with `This program cannot be applied with code.` A nominative balance is attached automatically and is never claimed by code.
6. **Lock the program row** for update without waiting. When the row is already locked by a concurrent transaction, the operation fails with a serialization error and the whole request is retried. This is what prevents two shoppers from consuming the last remaining use of a limited program at the same time.
7. When the program limits its usage and its total document count has reached the ceiling, refuse with `This code is expired (<code>).`
8. When a rule matched, add it to the order's activated code rules.
9. When a card was found, add it to the order's applied cards.
10. **Branch**:
    - when the program already grants points on this order, re-evaluate the order (section 3.1);
    - otherwise, when the program's `applies_on` is not `future` **or** no card was found, apply the program with the full check (section 3.3). When that refuses and the program is not nominative, or it is nominative but no card was found, undo step 8 and step 10's card attachment (unless the refusal was "already applied") and return the refusal.
11. Return the rewards claimable from the found or newly created cards.

**Postcondition**: the code is recorded on the order, the program grants its points, and the caller is handed the list of rewards the customer may now claim.

### 3.5 Claim a reward on a sales order

**Actor**: Salesperson through the reward wizard; the storefront claims some rewards automatically.

1. The actor presses the reward button on the order. The order is re-evaluated (section 3.1) and the claimable rewards are computed.
2. **Shortcut**: when exactly one card offers exactly one reward and that reward is not a multi-product reward, it is applied directly and the workflow ends.
3. **Shortcut**: when no reward is claimable, the workflow ends with nothing.
4. Otherwise the reward selection wizard opens, listing every claimable reward. The actor picks one and, for a multi-product reward, a product.
5. Applying:
   1. Refuse with `No reward selected.` when nothing was picked.
   2. Recompute the claimable rewards and find the card that offers the chosen reward. Refuse with `Coupon not found while trying to add the following reward: <description>` when none does.
   3. Apply the reward:
      - **Global discount guard**: when the reward is a global discount and a different global discount is already applied and the applied one is at least as good, refuse with `A better global discount is already applied.` When the applied one is worse, its lines are completely reset and added to the pool of lines the new reward may reuse.
      - **Future guard**: when the program is not nominative, its `applies_on` is `future` and the card is one this order promises points to, refuse with `The coupon can only be claimed on future orders.`
      - **Balance guard**: when the points available on the card are less than the reward's required points, refuse with `The coupon does not have enough points for the selected reward.`
      - Otherwise build the reward lines (section 9 of [calculations.md](calculations.md)) and write them onto the order.
   4. Re-evaluate the order, which reorders the discounts and recomputes every amount.
   5. Delete every card created by this order for a program whose `applies_on` is `current` that no reward line uses.
6. Cancelling the wizard performs only step 5.5.

### 3.6 Remove a reward from a sales order

**Actor**: Salesperson deleting a reward line, or shopper removing a reward in the cart.

1. Deleting one reward line also deletes every line sharing the same triple (reward, card, reward grouping code). A discount split over four tax groups disappears as one unit.
2. For each of those lines, when the order is confirmed, the point cost is added back to the card.
3. When the line's card is among the order's applied cards, it is detached from them.
4. Otherwise, when the card was created by this order for a program whose `applies_on` is `current` and no surviving line uses it, the card is deleted and every activated code rule of that program is removed from the order.
5. In the online shop, the removed reward is additionally recorded in `disabled_auto_rewards` so that the automatic claiming does not put it back.

### 3.7 Confirm a sales order

**Actor**: Salesperson, or the storefront on payment.
**Preconditions**: the order is a quotation.

1. For each order being confirmed, gather every card involved: the applied cards, the cards of the pending promises and the cards referenced by the lines. When the points available on any of them are negative, refuse the whole confirmation with `One or more rewards on the sale order is invalid. Please check them.`
2. Re-evaluate the order (section 3.1). This is the last chance for an expired program, an archived card or a no-longer-met condition to remove its reward lines.
3. Write the history entries (section 3.8).
4. When exactly one order is being confirmed, remember whether it still has claimable rewards.
5. Delete every card whose program has `applies_on` equal to `current` that this order promises points to and that no reward line uses. Such a card could never be spent and would be lost forever.
6. For every order that is not already confirmed, apply the point changes (section 5.1 of [calculations.md](calculations.md)) to the cards: `card.points = card.points + change`.
7. Run the ordinary confirmation of the order.
8. When the ordinary confirmation returned a plain success and step 4 found claimable rewards, return instead an informational notification titled `Rewards Available` with the message `There are available rewards not added to this order.`
9. Send the reward coupons: every card of the order's non-zero promises whose program has `applies_on` equal to `future` receives its "at creation" communication, forced to send immediately rather than through the outgoing queue. This is what emails the gift card the customer just bought and the next-order coupon the order just earned.

**Postcondition**: the cards carry their new balances, the history records the movement, the reward lines are frozen with the order, and the customer has received the cards the order produced.

### 3.8 Write the history entries of a confirmed order

1. Build a map per card: `issued` is the pending promise of the order towards that card; `cost` is the sum of the point costs of the order's lines that reference it.
2. Create one Loyalty History Entry per card in the map: `card_id`, `order_model` the sales order entity, `order_id` the order identifier, `description` `Order <order display name>`, `used` equal to `cost` (zero when absent) and `issued` equal to `issued` (zero when absent).

A card that both earns and spends on the same order gets a single entry carrying both numbers.

### 3.9 Changing a reward on an already confirmed order

Once the order is confirmed, every change to a reward line moves points immediately instead of waiting for a confirmation.

| Change | Effect |
|---|---|
| A reward line is created with a card and a non-zero point cost | The card's balance is reduced by the cost; the order's history entry for that card is updated by the same amount, or created when there is none. |
| The point cost of a line is written | The previous cost is given back to the previous card and the new cost is taken from the new card. When the card is unchanged, the order's history entry is adjusted by the difference in one operation; when the card changed, the old card's entry is reduced by the old cost and the new card's entry is increased by the new cost. |
| The card of a line is written | Same as above. |
| A reward line is deleted | Every line of the same reward application is deleted with it and each point cost is given back to its card. |

### 3.10 Cancel a confirmed sales order

**Actor**: Salesperson.

1. Remember which of the orders were confirmed.
2. Run the ordinary cancellation.
3. Delete every Loyalty History Entry that references one of the previously confirmed orders.
4. For every order that left the confirmed state, subtract the point changes from the cards: `card.points = card.points − change`. The customer gets back what they spent and loses what they earned.
5. Delete every reward line of the orders.
6. Delete every card that this order promised points to, whose program is not nominative, that was created by this order, and whose use count is zero. A gift card that was bought and already spent elsewhere survives; one that was never used disappears.
7. Delete every pending promise of the orders.

**Postcondition**: the cards are back to the balance they had before the order, and nothing links them to the cancelled order.

### 3.11 Duplicate a sales order

Duplicating an order copies its ordinary lines and then **deletes every reward line of the copy**. The applied cards, the activated code rules and the pending promises are not copied either. The copy is therefore a clean quotation that will earn its own rewards at its first evaluation.

### 3.12 Recompute the prices of an order

When a salesperson resets the prices of an order to the pricelist, the ordinary recomputation runs first; then, for every order that carries at least one reward line, the full evaluation of section 3.1 runs, so that percentage discounts follow the new prices.

## 4. Gift card and electronic wallet workflows

### 4.1 Buy a gift card on a sales order

**Actor**: Salesperson or shopper.
**Preconditions**: an active gift card program whose trigger products include the product being sold.

1. The customer's order carries a line with the gift card product, quantity `n`, unit price equal to the face value.
2. At the next evaluation, the gift card program is an automatic candidate. Its rule grants points per unit of currency spent with the split option on, and the program's `applies_on` is `future`, so the split branch runs: one point value per unit sold, each equal to the tax-included line total divided by the quantity, truncated to two decimals.
3. The evaluation creates one Loyalty Card per value, with balance zero, owner empty, and `order_id` set to this order, plus one pending promise per card.
4. Nothing is claimable on this order: a card generated by the order for a `future` program is explicitly excluded from the claimable computation.
5. On confirmation the promises are applied: each card receives its face value. The "at creation" communication sends the gift card email to the order's customer, with the printed gift card attached, and the codes appear on the order's portal page with a copy button.

**Worked example**: an order carries quantity 2 of a gift card product priced 50.00, no tax. The split branch produces `points_per_unit = round_down(1 × 100.00 ÷ 2, 2) = 50.00` twice, so `split_points = [50.00, 50.00]` and the result is `[0, 50.00, 50.00]`. Three promises are created; the one carrying zero points is deleted at confirmation because it claims nothing. Two cards of 50.00 are issued with two distinct codes.

### 4.2 Pay an order with a gift card

1. The customer gives the code; the salesperson enters it through the coupon code wizard, or the shopper types it in the cart.
2. The code application (section 3.4) attaches the card to the order and returns the gift card reward as claimable.
3. Claiming it produces the single payment line described in section 9.2 of [calculations.md](calculations.md): unit price the negative of the smaller of the balance and the order's discountable amount, taxes taken from the gift card discount product.
4. The payment line is always recomputed last, after every other discount, so that it pays the discounted total.
5. On confirmation the card's balance drops by the point cost and a history entry records the use. When the balance reaches zero the card can no longer be applied; the code application then answers `This coupon has already been used.`

### 4.3 Top up and spend an electronic wallet

1. **Top up**: the customer buys a top-up product of the electronic wallet program. The program's rule grants points per unit of currency spent with the split option **off**, and `applies_on` is `future`, so the aggregate branch runs and a single card is created for the order's customer.
   - The wallet program is nominative, so the card is always attached to a customer; an anonymous order cannot top up a wallet.
   - The bottomless-wallet guard stops the evaluation of a wallet program that names no trigger product, so a misconfigured wallet can never grant points on every purchase.
2. **Spend**: on any later order of the same customer, the evaluation automatically attaches every wallet card of that customer with a positive balance. The wallet reward then appears among the claimable rewards without any code.
   - A wallet code may never be applied by hand: the code application refuses with `This program cannot be applied with code.`
3. The wallet payment line carries **no tax**, unlike the gift card line.
4. The discountable amount of a wallet payment excludes the lines whose product is one of the wallet's own trigger products, so a wallet may not pay for its own top-up.

### 4.4 Issue next-order coupons

1. A program of type `next_order_coupons` has `applies_on` `future` and `trigger` `auto`; its rule usually carries a minimum purchase.
2. When an order meets the rule, the evaluation creates one card with `partner_id` set to the order's customer (this program type is the only non-nominative type that fills the owner) and a pending promise of the granted points.
3. Nothing is claimable on that order.
4. On confirmation the card receives its points and the "at creation" communication emails the coupon code with the printed coupon attached.
5. On a later order the customer enters the code, which attaches the card and makes the reward claimable.
6. When the order's customer was the public customer at the time the card was created and a real customer is set later, the evaluation rewrites the card's owner to the real customer.

## 5. Downstream workflows

### 5.1 Invoice an order that carries rewards

1. Reward lines are invoiced like ordinary lines, with their negative amounts and their own taxes.
2. A reward line may never be invoiced alone: an invoice containing only reward lines is not produced.
3. On the invoice, a line that came from a reward of type `discount` is classified as a discount line for reporting purposes.
4. When the order total including tax is zero, its reward total is not zero and automatic invoicing is enabled, an invoice is created and posted anyway, then marked as sent and dispatched with the configured invoice email template. See section 17 of [calculations.md](calculations.md).

### 5.2 Read a card on the customer portal

**Actor**: Customer with a portal account.

1. The customer's portal home lists, per program, the cards they own whose program is active, whose program type is `loyalty` or `ewallet`, and that are not expired.
2. Opening a card shows the balance formatted with the program's point name, the expiration date, the code, the last five history entries with a link to the sales order behind each one, and the three most expensive rewards the current balance can already pay for, ordered by required points descending.
3. The full history page is paginated and can be sorted by date (most recent first), by used amount, by description or by issued amount.
4. A customer may only read their own cards; requesting another customer's card redirects to the portal home.

### 5.3 Clean up abandoned online carts

**Actor**: scheduled cleanup job.

1. Find every draft sales order that belongs to a website, has at least one applied card, and has not been written to for longer than the abandonment delay (four days by default, overridable by the system parameter `website_sale_coupon.abandonned_coupon_validity`).
2. Clear the applied cards of those orders.
3. Re-evaluate each of them, which removes the reward lines those cards paid for and releases the cards for another shopper.

### 5.4 Guards that protect the configuration

| Attempted operation | Guard | Rule |
|---|---|---|
| Archive a product that is the hidden discount product of an active reward, or one of its discounted products | Refused with `This product may not be archived. It is being used for an active promotion program.` | `LOY-148` |
| Delete the shipped gift card product or the shipped wallet top-up product, as a variant or as a template | Refused with `You cannot delete <name> as it is used in 'Coupons & Loyalty'. Please archive it instead.` | `LOY-149` |
| Archive a pricelist referenced by an active program | Refused with `This pricelist may not be archived. It is being used for active promotion programs: <program names>` | `LOY-150` |
| Delete the hidden discount product of an existing reward | Refused by the restricted reference at the database level; the product must be archived instead. | `LOY-151` |
| Archive a program that owns a free product reward | Allowed. The hidden discount products of the rewards are archived, the reward products themselves stay active, and the product archiving guard does not fire because the rewards are archived first. | `LOY-152` |
| Delete a reward already used on a sales order line or a point-of-sale line | Silently converted into an archive of that reward. | `LOY-035` |
| Delete a program that is active | Refused with `You can not delete a program in an active state` | `LOY-011` |

## 6. State tables

### 6.1 Loyalty Program

| From state | Trigger or operation | Guard conditions | To state | Side effects |
|---|---|---|---|---|
| (none) | Create from a template or from the form | The family defaults are applied; at least one reward must remain | Active | Rules, rewards, communication rules and one hidden discount product per reward are created. |
| Active | Change `program_type` | none | Active | `applies_on`, `trigger`, `portal_visible`, `portal_point_name`, rules, rewards and communication rules are replaced by the family defaults of the new type; the at-least-one-reward validation is suspended for this write. |
| Active | Archive | none | Archived | Rules, rewards, communication rules and reward discount products are archived. Documents lose the program's reward lines at their next evaluation. |
| Archived | Unarchive | No other active rule carries any of the program's codes | Active | Rules, rewards, communication rules and reward discount products are reactivated. |
| Archived | Unarchive | Another active rule carries one of the codes | Archived | Refused with `The promo code must be unique.` |
| Active | Delete | none | Active | Refused with `You can not delete a program in an active state`. |
| Archived | Delete | No card references the program | (deleted) | Rules, rewards and communication rules are deleted by cascade. |
| Archived | Delete | At least one card references the program | Archived | Refused by the restricted reference of the card. |

### 6.2 Loyalty Card

| From state | Trigger or operation | Guard conditions | To state | Side effects |
|---|---|---|---|---|
| (none) | Generation wizard | Quantity strictly positive and a program is set | Issued | Balance set to the grant, code generated, "at creation" communication sent, one history entry created. |
| (none) | Order evaluation creates a card for a program | The program grants points | Issued with balance zero | A pending promise is recorded; the card carries the order reference. |
| (none) | Ticket confirmation at a counter | The ticket earned points | Issued | Balance set from the ticket, code taken from the device or generated, "at creation" communication sent and the printed document rendered. |
| Issued | Order confirmation applies the point change | none | Issued with a new balance | Balance increased by the promise and decreased by the reward costs; a history entry is written; milestone communications may be sent. |
| Issued | Order cancellation reverses the point change | The order was confirmed | Issued with the previous balance | The history entries of that order are deleted. |
| Issued | Order cancellation | The card is not nominative, was created by that order and has never been used | (deleted) | The card disappears. |
| Issued | Order evaluation finds the program no longer applicable | The card was created by this order and the program is not nominative | (deleted) | The reward lines it paid for are deleted. |
| Issued | Balance reaches zero | none | Emptied | Code application answers `This coupon has already been used.` |
| Issued or Emptied | The reference date passes `expiration_date` | `expiration_date` is set | Expired | The card is excluded from the claimable computation and from the code application. |
| Any | Archive | none | Archived | Pending promises towards draft orders are deleted first. |
| Any | Contact merge | The card is nominative and its owner is merged away | Archived with balance zero, or surviving with the summed balance | See section 2.6. |

### 6.3 Reward line on a sales order

| From state | Trigger or operation | Guard conditions | To state | Side effects |
|---|---|---|---|---|
| (none) | A reward is claimed | The card has enough points and the guards pass | Applied | One line per tax group for a discount, one line for a free product, a payment or free shipping; a grouping code is generated; the point cost is written on the first line. |
| Applied | The order is re-evaluated | The reward is still applicable | Applied with refreshed amounts | The lines are reset, recomputed and rewritten in place; a manually edited description survives when the product is unchanged. |
| Applied | The order is re-evaluated | The card, the points or the program no longer qualify | (deleted) | The lines are deleted in the cleanup step. |
| Applied | The user deletes one of the lines | none | (deleted) | Every line of the same reward application is deleted; the points are returned when the order is confirmed; the card may be detached or deleted. |
| Applied | A better global discount is applied | The new one is strictly better | (reset and reused) | The lines are reset completely and handed to the new reward as reusable lines. |
| Applied | The order is confirmed | none | Frozen | The point cost is moved onto the card and a history entry is written. |
| Frozen | The order is cancelled | none | (deleted) | The points are returned and the history entry is deleted. |
| Applied | The discountable amount becomes zero while a payment reward is applied | The reward is not a payment reward | Placeholder | A single line named `TEMPORARY DISCOUNT LINE` with quantity zero and price zero keeps the reward attached. |

### 6.4 Pending promise (Sales Order Coupon Points)

| From state | Trigger or operation | Guard conditions | To state | Side effects |
|---|---|---|---|---|
| (none) | The evaluation attaches a program | The program grants points, or the program is nominative | Pending | The promise carries the points the order will grant. |
| Pending | The evaluation recomputes the points | The program is still applicable | Pending with new points | The promise is rewritten in place. |
| Pending | The evaluation finds the program no longer applicable | none | (deleted) | The points are set to zero first, so that no intermediate computation uses them. |
| Pending | The card's owner differs from the order's customer | The card has an owner | (deleted) | The points are set to zero first. |
| Pending | The order is confirmed | none | Applied | The points are added to the card's balance and the promise stays as the record of what was granted. |
| Applied | The order is cancelled | none | (deleted) | The points are subtracted from the card's balance. |
| Pending | The card is archived | The order is still a draft | (deleted) | Removed before the archive is written. |

## 7. End-to-end walkthrough: four programs on one order

This walkthrough exercises the ordering rules, the point bookkeeping and the tax distribution together. Every intermediate number is given.

**Setup**

| Program | Type | Trigger | Applies on | Rule | Reward |
|---|---|---|---|---|---|
| Spring sale | `promotion` | `auto` | `current` | no product filter, minimum amount 100, 1 point per order | 10 percent on the order, 1 point required |
| Welcome code | `promo_code` | `with_code` | `current` | code `WELCOME`, no minimum | 5 percent on the order, 1 point required |
| Club | `loyalty` | `auto` | `both` | no product filter, 0.5 point per unit of currency spent | 20 off the order, mode `per_order`, 400 points required |
| Gift cards | `gift_card` | `auto` | `future` | the gift card product, 1 point per unit of currency spent, split on | 1 per point on the order, 1 point required |

**The order**

| Line | Product | Quantity | Unit price | Tax |
|---|---|---|---|---|
| 1 | Product Alpha | 2 | 200.00 | 20 percent, tax excluded |
| 2 | Product Beta | 1 | 100.00 | none |

Totals before any reward: 500.00 tax excluded, 80.00 tax, 580.00 tax included.

The customer owns a Club card holding 250.00 points and a gift card holding 60.00.

**Step 1: the first evaluation**

1. Nominative programs are allowed, so the customer's Club card is attached to the order as an applied card. The gift card is a bearer card of a non-nominative program, so it is **not** attached automatically; it must be entered by code.
2. Automatic candidates: Spring sale (trigger automatic, one automatic rule) and Gift cards. Welcome code is excluded because its trigger needs a code.
3. Spring sale: the minimum amount of 100 is met by 580.00 tax included, the quantity gate passes, and the grant is 1 point per order. A bearer card is created for the order with a promise of 1 point.
4. Club: the program is nominative and already attached. Its rule grants `round_down(0.5 × 580.00, 2) = 290.00` points. A promise of 290.00 points towards the customer's Club card is recorded.
5. Gift cards: the rule's product filter does not match any line, so the program grants nothing and no card is created for it.
6. Claimable rewards: the Spring sale discount (1 point available, 1 required) and the Club discount (`250.00 + 290.00 = 540.00` points available, 400 required).

**Step 2: the Spring sale discount is claimed**

1. Discountable amount, order applicability: line 1 contributes 400.00 base and 80.00 tax, so 480.00, with a bucket of 400.00 under the twenty percent tax; line 2 contributes 100.00 with a bucket of 100.00 under the empty tax set. `discountable = 580.00`.
2. `max_discount = min(+infinity, 580.00) = 580.00`; percent gives `min(580.00, 58.00) = 58.00`.
3. `discount_factor = 58.00 ÷ 580.00 = 0.10`.
4. Two reward lines: `−40.00` with the twenty percent tax and `−10.00` with no tax. The first carries the point cost of 1.
5. New totals: 450.00 tax excluded, 72.00 tax, 522.00 tax included.
6. Re-evaluation: the Club grant is recomputed. The threshold lines exclude nothing, and the money-mode amount includes the discount lines because their program type is `promotion`: `amount_paid = 580.00 − 48.00 − 10.00 = 522.00`, so the Club grant becomes `round_down(0.5 × 522.00, 2) = 261.00`. The promise is rewritten to 261.00.

**Step 3: the code `WELCOME` is entered**

1. A rule with that code is found and matches the rule filter. It is added to the order's activated code rules.
2. The program row is locked. There is no usage ceiling, so the check passes.
3. The program is not yet granting points, its `applies_on` is `current`, so it is attached with the full check: it is a global discount and a global discount (Spring sale, 10 percent) is already applied. Comparing the two against the discountable amount ignoring Spring sale: 10 percent gives 58.00 and 5 percent gives 29.00, neither exceeds the discountable amount, and the applied one is larger, so the attachment is refused with `This discount (5% on your order) is not compatible with "10% on your order". Please remove it in order to apply this one.` The rule activation is undone.

**Step 4: the Club discount is claimed**

1. Points available on the Club card: `250.00 + 261.00 − 0 = 511.00`, which is at least 400.
2. Discountable amount, order applicability: the two ordinary lines plus the two Spring sale reward lines. `discountable = 522.00`; buckets: `400.00 − 40.00 = 360.00` under the twenty percent tax and `100.00 − 10.00 = 90.00` under the empty tax set.
3. `max_discount = min(+infinity, 522.00) = 522.00`; mode `per_order` gives `min(522.00, 20.00) = 20.00`.
4. `discount_factor = 20.00 ÷ 522.00 = 0.038314…`.
5. Two reward lines: `−(360.00 × 0.038314…) = −13.79` with the twenty percent tax and `−(90.00 × 0.038314…) = −3.45` with no tax. Point cost 400.00 on the first.
6. New totals: 432.76 tax excluded, 69.24 tax, 502.00 tax included. The tax-included reduction is `13.79 × 1.20 + 3.45 = 16.55 + 3.45 = 20.00`, exactly the fixed amount.
7. Re-evaluation: the Club grant is recomputed once more. The money-mode amount now also includes the Club discount lines, but they are skipped because their reward's program is the Club program itself: `amount_paid = 580.00 − 48.00 − 10.00 = 522.00` and the grant stays 261.00.

**Step 5: the gift card code is entered**

1. No rule carries that code, so a card is looked up. It is found, it is not expired and its balance 60.00 is at least the smallest required points of its program (1), so it is attached as an applied card.
2. The program type is `gift_card`, so it is not refused as a nominative program, and its `applies_on` is `future` while a card was found, so the program is **not** attached with the full check: only the card is attached.
3. Claimable rewards now include the gift card reward.
4. The reward is claimed. Payment rewards are computed last, so the discountable amount is taken after the two discounts: `discountable = 502.00`, fixed-amount taxes included (there are none here). `max_discount = min(502.00, 1 × 60.00) = 60.00`. The single reward line is `−60.00` with no tax and a point cost of `round_currency(60.00 ÷ 1) = 60.00`.
5. New total including tax: 442.00.
6. Re-evaluation: the Club grant is recomputed. The money-mode amount now skips the gift card reward line because its program is a payment program: `amount_paid` stays 522.00 and the grant stays 261.00. Paying with a gift card therefore does not reduce the loyalty points earned.

**Step 6: confirmation**

1. Every involved card is checked: the Spring sale card has `0 + 1 − 1 = 0` points available, the Club card has `250.00 + 261.00 − 400.00 = 111.00`, the gift card has `60.00 + 0 − 60.00 = 0.00`. None is negative, so the confirmation proceeds.
2. The order is re-evaluated one last time; nothing changes.
3. History entries are written: the Spring sale card gets `issued` 1 and `used` 1; the Club card gets `issued` 261.00 and `used` 400.00; the gift card gets `issued` 0 and `used` 60.00. Each carries the description `Order <order name>` and a reference to the order.
4. Cards of `current` programs that claim no reward are deleted: the Spring sale card claims a reward, so it survives.
5. The point changes are applied: the Spring sale card goes to 0, the Club card goes to `250.00 + 261.00 − 400.00 = 111.00`, the gift card goes to `60.00 − 60.00 = 0.00`.
6. No card of a `future` program received points on this order, so no coupon email is sent.
7. The order is confirmed with a total of 442.00 including tax.

**Step 7: cancellation**

Cancelling the order deletes the three history entries, restores the Club card to 250.00 and the gift card to 60.00, deletes the five reward lines, deletes the Spring sale card (not nominative, created by this order, use count zero once its lines are gone) and deletes the three promises.
