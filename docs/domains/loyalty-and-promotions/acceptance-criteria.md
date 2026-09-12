# Acceptance criteria

Given/When/Then scenarios that a replacement implementation must pass. Every scenario is independently verifiable and uses concrete values. Unless a scenario says otherwise, amounts are in a currency with two decimal places and a rounding step of 0.01, no fiscal position is applied, no pricelist restriction is set, and the acting user is a Sales Administrator.

## 1. Program definition

**AC-LOY-001** Given a new program form, When the program type `promotion` is chosen, Then the program has `applies_on` `current`, `trigger` `auto`, `portal_visible` false, `portal_point_name` `Promo point(s)`, exactly one rule granting 1 point per order with a minimum amount of 50 and a minimum quantity of 0, exactly one reward requiring 1 point and discounting 10, and no communication rule.

**AC-LOY-002** Given a new program form, When the program type `gift_card` is chosen, Then the program has `applies_on` `future`, `trigger` `auto`, `portal_visible` true, `portal_point_name` equal to the company currency symbol, exactly one rule granting 1 point per unit of currency spent with the split option on and the shipped gift card product as its only product, exactly one reward of mode `per_point` worth 1 per point applicable to the order requiring 1 point and described `Gift Card`, and exactly one communication rule with trigger `create` using the shipped gift card template.

**AC-LOY-003** Given a program of type `promotion` whose rule was edited to grant 3 points per unit, When the program type is changed to `buy_x_get_y`, Then the edited rule is destroyed and replaced by a rule granting 1 point per unit paid on the first sellable product with a minimum quantity of 2, and the reward is replaced by a free product reward of that product requiring 2 points.

**AC-LOY-004** Given a program, When every reward is deleted, Then the operation is refused with `A program must have at least one reward.`

**AC-LOY-005** Given a program with `limit_usage` true, When `max_usage` is set to 0, Then the write is refused with `Max usage must be strictly positive if a limit is used.`

**AC-LOY-006** Given a program whose currency is one currency, When a pricelist expressed in another currency is added to its pricelist restriction, Then the write is refused with `The loyalty program's currency must be the same as all it's pricelists ones.`

**AC-LOY-007** Given a program, When `date_from` is set to 2026-06-30 and `date_to` to 2026-06-01, Then the write is refused with `The validity period's start date must be anterior or equal to its end date.`

**AC-LOY-008** Given an active program, When a user asks to delete it, Then the deletion is refused with `You can not delete a program in an active state`.

**AC-LOY-009** Given an active program with one rule, one reward and one communication rule, When it is archived, Then the rule, the reward, the communication rule and the reward's hidden discount product are all archived as well; When it is unarchived, Then all four are reactivated.

**AC-LOY-010** Given an archived program whose rule carries the code `SUMMER26` and an active program whose rule carries the same code, When the archived program is unarchived, Then the operation is refused with `The promo code must be unique.`

**AC-LOY-011** Given two archived programs whose rules both carry the code `SUMMER26`, When both are unarchived in a single operation, Then the operation is refused with `The promo code must be unique.`

**AC-LOY-012** Given the website capability package, an active program bound to website A whose rule carries `SUMMER26`, and an archived program bound to website B whose rule carries `SUMMER26`, When the archived program is unarchived, Then it succeeds.

**AC-LOY-013** Given a gift card program with no email template, When a printed document is set on it, Then the write is refused with `You must set 'Email template' before setting 'Print Report'.`

**AC-LOY-014** Given a program of type `loyalty`, When the point name is set to `Stars`, Then `portal_point_name` is `Stars`; Given a program of type `ewallet`, When the point name is set to `Stars`, Then `portal_point_name` is forced back to the program currency symbol.

**AC-LOY-015** Given a program of type `promotion`, When it is created with a value for `trigger_product_ids`, Then that value is ignored and the rule keeps the product filter installed by the family defaults; Given a program of type `gift_card`, When it is created with a value for `trigger_product_ids`, Then the rule's product filter is that value.

**AC-LOY-016** Given a program whose type has just been changed to `buy_x_get_y`, so that its rewards are the free product rewards of that family, When the program is written again with one further reward carrying only a description, Then that write is an ordinary write and not a type change: the existing rewards are kept, the new reward is created with the reward type `product` inherited from the program family, and every reward of the program therefore has the reward type `product`.

**AC-LOY-017** Given the program form with the type set to `buy_x_get_y`, When the reward the family defaults created is removed in the form and a new reward is added in its place with a reward product quantity of 2, Then saving the form produces a program with exactly one reward, of reward type `product`, with a reward product quantity of 2, and the at-least-one-reward validation is not raised, because the removal and the addition are part of the same write.

## 2. Rule definition

**AC-LOY-021** Given a rule, When `reward_point_amount` is set to 0, Then the write is refused with `Rule points reward must be strictly positive.`

**AC-LOY-022** Given a program of type `ewallet`, When its rule's split option is turned on, Then the write is refused with `Split per unit is not allowed for Loyalty and eWallet programs.`

**AC-LOY-023** Given a program whose `applies_on` is `both`, When a rule's split option is turned on, Then the write is refused with `Split per unit is not allowed for Loyalty and eWallet programs.`

**AC-LOY-024** Given an active rule carrying the code `TEN`, When a second active rule is given the code `TEN`, Then the write is refused with `The promo code must be unique.`

**AC-LOY-025** Given an active card carrying the code `044a-1234-5678`, When a rule is given that same code, Then the write is refused with `A coupon with the same code was found.`

**AC-LOY-026** Given an active rule carrying the code `TEN`, When a card is created with the code `TEN`, Then the creation is refused with `A trigger with the same code as one of your coupon already exists.`

**AC-LOY-027** Given a rule with `mode` `auto`, When a code is typed into it, Then `mode` becomes `with_code` and `promo_barcode` is regenerated; When the code is cleared, Then `mode` becomes `auto`.

**AC-LOY-028** Given a gift card program whose rule names no product, no category, no tag and an empty condition, When an order containing any product is evaluated, Then the program grants no point and issues no card.

## 3. Reward definition

**AC-LOY-031** Given a reward, When `required_points` is set to 0, Then the write is refused with `The required points for a reward must be strictly positive.`

**AC-LOY-032** Given a reward of type `product`, When `reward_product_qty` is set to 0, Then the write is refused with `The reward product quantity must be strictly positive.`

**AC-LOY-033** Given a reward of type `discount`, When `discount` is set to 0, Then the write is refused with `The discount must be strictly positive.`

**AC-LOY-034** Given a combination product, When it is chosen as a reward product, Then the write is refused with `A reward product can't be of type "combo".`

**AC-LOY-035** Given a reward created with the description `10% on your order`, Then a hidden service product named `10% on your order` exists, is not sellable, is not purchasable, is priced 0, has no customer tax and no vendor tax, and has the invoicing policy "ordered quantities".

**AC-LOY-036** Given a reward with a hidden discount product, When the description is changed to `Summer deal`, Then the hidden discount product is renamed `Summer deal`.

**AC-LOY-037** Given a reward, When it is duplicated, Then the copy has its own newly created hidden discount product, not the original one.

**AC-LOY-038** Given a reward already used by a sales order line, When a user deletes that single reward, Then the reward is archived instead of deleted and the sales order line still resolves it.

**AC-LOY-039** Given a reward with `discount_mode` `percent`, `discount` 10 and `discount_applicability` `order`, Then its description is `10% on your order`.

**AC-LOY-040** Given a reward of type `product` whose tag resolves to three products, Then `multi_product` is true, `reward_product_ids` holds the three products, and the description is `Free Product - [<the three display names separated by a comma and a space>]`.

**AC-LOY-041** Given a reward with `discount_max_amount` 40 in a currency whose symbol is placed before the amount, `discount_mode` `per_order`, `discount` 15 and `discount_applicability` `specific` matching more than one product, Then its description is `<symbol> 15 on specific products (Max <symbol> 40)`.

**AC-LOY-042** Given a product used as the hidden discount product of an active reward, When a user archives that product, Then the operation is refused with `This product may not be archived. It is being used for an active promotion program.` (`LOY-148`)

**AC-LOY-043** Given a pricelist referenced by an active program, When a user archives that pricelist, Then the operation is refused with `This pricelist may not be archived. It is being used for active promotion programs: <program names>` where the placeholder lists the names of every offending active program separated by a comma and a space (`LOY-150`).

**AC-LOY-044** Given a second language activated in the deployment, a program, and a reward of type `discount` created in the first language with the description `My Discount`, so that its hidden discount product is named `My Discount`, When the translations of the reward description are written as `Test Discount EN` for the first language and `Test Discount FR` for the second, Then the hidden discount product's name reads `Test Discount EN` in the first language and `Test Discount FR` in the second, without any further action (`LOY-033`).

**AC-LOY-045** Given the shipped gift card product, When a user deletes it as a product variant, Then the deletion is refused with `You cannot delete Gift Card as it is used in 'Coupons & Loyalty'. Please archive it instead.`; When the same user deletes the product template that carries it, Then the deletion is refused with the same message; Given the shipped wallet top-up product, Then both deletions are refused with `You cannot delete Top-up eWallet as it is used in 'Coupons & Loyalty'. Please archive it instead.` (`LOY-149`).

**AC-LOY-046** Given a reward and its hidden discount product, When a user deletes that product, Then the deletion is refused by the restricted reference of the reward and the product still exists (`LOY-151`); When the reward is archived instead, Then the hidden discount product is archived with it.

**AC-LOY-047** Given a program of type `buy_x_get_y` with one reward of type `product` whose reward product is an existing product Alpha that is active, When the program is archived, Then the archive succeeds, the hidden discount product of the reward is archived, the reward and the rule are archived, and Alpha stays active; no refusal `This product may not be archived. It is being used for an active promotion program.` is raised, because the reward is archived before its hidden discount product is written (`LOY-152`).

## 4. Cards

**AC-LOY-051** Given a generated card, Then its code is fourteen characters long and begins with `044`, and no other card carries the same code.

**AC-LOY-052** Given the generation wizard on a program, When the mode is `anonymous`, the quantity is 25 and the grant is 10, Then 25 cards are created with balance 10, no owner and a history entry each with `issued` 10, `used` 0 and the description `Gift For Customer` when no description was typed.

**AC-LOY-053** Given the generation wizard in mode `selected` with two customers chosen, Then the quantity is recomputed to 2 and two cards are created, one per customer, each owned by its customer.

**AC-LOY-054** Given the generation wizard in mode `selected` with no customer and no tag chosen, Then the quantity is the total number of contacts in the database and one card is created per contact.

**AC-LOY-055** Given the generation wizard with a quantity of 0, When the generation is confirmed, Then it is refused with `Invalid quantity.`

**AC-LOY-056** Given a card of a program of type `loyalty`, When a user types an expiration date on it, Then the entry is refused immediately with `Expiration date cannot be set on a loyalty card.`

**AC-LOY-057** Given a card whose balance is 40 and whose program's point name is `Loyalty point(s)`, Then its formatted balance is `40 Loyalty point(s)`; Given a card of a gift card program whose point name is the currency symbol and whose balance is 40.5, Then its formatted balance is the amount 40.50 rendered in that currency.

**AC-LOY-058** Given a card with balance 30, When the balance wizard is confirmed with a new balance of 30, Then it is refused with `New Balance should be positive and different then old balance.`; When it is confirmed with a new balance of −5, Then it is refused with the same message.

**AC-LOY-059** Given a card with balance 30, When the balance wizard is confirmed with a new balance of 50 and the description `Compensation`, Then a history entry is created with `issued` 20, `used` 0 and that description, and the card's balance becomes 50.

**AC-LOY-060** Given a card with balance 30, When the balance wizard is confirmed with a new balance of 12, Then a history entry is created with `used` 18 and `issued` 0, and the balance becomes 12.

**AC-LOY-061** Given a program with milestone communication rules at 100, 250 and 500 points and a card owned by a contact whose balance moves from 80 to 300 in one write, Then exactly one email is sent, the one of the 250 milestone.

**AC-LOY-062** Given the same program and a card whose balance moves from 300 to 200, Then no milestone email is sent.

**AC-LOY-063** Given a card with a pending promise of 40 points towards a draft sales order, When the card is archived, Then the pending promise is deleted first and the card becomes archived.

**AC-LOY-064** Given contacts Alpha and Beta owning loyalty cards of 120 and 45 points of the same program and a contact Gamma owning a card of 10 points of that program, When Alpha and Beta are merged into Gamma, Then Gamma's card holds 175 points and Alpha's and Beta's cards are archived with balance 0.

**AC-LOY-065** Given a contact owning three active unexpired cards with positive balances of active programs, and a child contact owning one such card, Then the parent's active card count is 4.

## 5. Applicability

**AC-LOY-071** Given a program whose `date_to` is today, When an order with no payment transaction is evaluated today, Then the program is applicable; When the same order is evaluated tomorrow, Then it is not.

**AC-LOY-072** Given a program whose `date_to` is 2026-03-31, an order whose earliest confirmed payment transaction was created at 2026-03-31 23:10 coordinated universal time, and an evaluation time zone one hour ahead, Then the reference date is 2026-04-01 and the program is not applicable.

**AC-LOY-073** Given a program whose `date_to` is 2026-04-01 and the same order, Then the program is applicable even though the order record is written later.

**AC-LOY-074** Given a program restricted to pricelist A, When an order priced with pricelist B is evaluated, Then the program grants nothing and no reward line is produced; When the order's pricelist is changed to A and the order is evaluated again, Then the program applies.

**AC-LOY-075** Given a program with `limit_usage` true and `max_usage` 1, and one confirmed order already carrying one of its rewards, When a second order is evaluated, Then the program is not an automatic candidate and no reward line is produced; When its code is entered on the second order, Then it is refused with `This code is expired (<code>).`

**AC-LOY-076** Given an electronic wallet program whose trigger product list is empty and an order carrying any product, Then the program grants 0 points and creates no card.

**AC-LOY-077** Given a program of type `coupons` with no rule and `applies_on` `current` and a card of that program, When the card's code is applied to an order, Then the reward is claimable without any condition being checked.

**AC-LOY-078** Given a program whose only rule requires a code, When an order is evaluated without that code, Then the program reports `This program requires a code to be applied.`

**AC-LOY-079** Given a program whose rule requires a minimum amount of 50 in the program currency and an order totalling 30 of eligible products, Then the program reports `To take advantage of this offer, your order must include at least 50 <currency name> of the eligible products.`

**AC-LOY-080** Given a program whose rule requires a minimum quantity of 3 of product Alpha and an order carrying 2 units of Alpha, Then the program reports `You don't have the required product quantities on your sales order.`

**AC-LOY-081** Given a nominative program and an online cart owned by the anonymous public visitor, Then the program reports `This program is not available for public users.`

**AC-LOY-082** Given a nominative program, an identified customer and an order that earns nothing, Then the program is reported applicable with 0 points, and the customer's existing card is attached to the order.

## 6. Point computation

**AC-LOY-091** Given a rule granting 1 point per order and an order with any content, Then the order grants exactly 1 point however many lines it carries.

**AC-LOY-092** Given a rule granting 0.1 point per unit of currency spent with no product filter, an order line of 3 units at 100.00 with no tax, and an automatic 10 percent global discount already applied, Then the order grants `round_down(0.1 × (300.00 − 30.00), 2) = 27.00` points.

**AC-LOY-093** Given a rule granting 1 point per unit paid on product Alpha and an order carrying 4 units of Alpha and 2 units of Beta, Then the order grants 4 points.

**AC-LOY-094** Given a rule granting 1 point per unit paid on product Alpha and an order carrying 4 paid units of Alpha and 1 free unit of Alpha given by a reward, Then the order still grants 4 points.

**AC-LOY-095** Given a rule whose minimum amount is 320 evaluated tax excluded and an order carrying one unit priced 320.00 with a 15 percent tax-included tax, Then the gate fails and no reward line is produced; When the tax is changed to tax excluded so that the tax-excluded amount becomes 320.00, Then the gate passes.

**AC-LOY-096** Given a gift card program whose rule grants 1 point per unit of currency spent with the split option on, and an order carrying 2 units of a gift card product priced 50.00 with no tax, Then the point result is `[0, 50.00, 50.00]`, two cards are created and each receives 50.00 at confirmation.

**AC-LOY-097** Given an electronic wallet program whose rule grants 1 point per unit of currency spent with the split option off, and an order carrying 2 units of a top-up product priced 50.00, Then the point result is `[100.00]` and a single card receives 100.00 at confirmation.

**AC-LOY-098** Given a loyalty program in a currency whose rounding step is 250 and whose rule grants 1 point per order, and a card holding 1 point, Then the points available on an order are 1, not 0, and a reward requiring 1 point is claimable.

**AC-LOY-099** Given a loyalty program whose rule grants points per unit of currency spent and a card whose balance would round, Then the points available on an order are rounded by the program currency's rounding step.

**AC-LOY-100** Given a card with balance 30.00, an order that will grant 120.00 points to it and an already claimed reward costing 100.00 points on that order, Then the points available on that order are 50.00.

## 7. Claimable rewards

**AC-LOY-111** Given an order carrying a card of a program whose `applies_on` is `future` that this very order created, Then no reward of that card is claimable on that order.

**AC-LOY-112** Given a card whose expiration date is yesterday, Then none of its rewards is claimable and the card is detached from the order at the next evaluation together with the lines it paid for.

**AC-LOY-113** Given an order whose discountable amount is zero and no payment reward applied, Then no discount reward is claimable.

**AC-LOY-114** Given an order of 100.00 fully paid by a gift card so that its total is 0.00, When a 10 percent code is applied, Then the discount is still claimable and, after the evaluation, the order total stays 0.00 because the gift card line shrinks to 90.00.

**AC-LOY-115** Given a discount reward already applied on an order and not belonging to a payment program, Then it does not appear again in the claimable rewards.

**AC-LOY-116** Given a free product reward whose only reward product is archived, Then it is not claimable.

**AC-LOY-117** Given a free shipping reward already applied on an order, Then no other shipping reward is claimable on that order.

## 8. Global discount selection

**AC-LOY-121** Given an order whose discountable amount is 500.00, an applied fixed discount of 80.00 and a candidate 10 percent discount worth 50.00, When the candidate is claimed, Then it is refused with `A better global discount is already applied.`

**AC-LOY-122** Given an order whose discountable amount is 500.00, an applied 5 percent discount worth 25.00 and a candidate 10 percent discount worth 50.00, When the candidate is claimed, Then the applied one is reset and reused for the candidate, and the order carries a 50.00 discount.

**AC-LOY-123** Given an order whose discountable amount is 40.00, an applied fixed discount of 80.00 and a candidate fixed discount of 50.00, When the candidate is claimed, Then the candidate replaces the applied one, because both exceed the discountable amount and the smaller one is preferred.

**AC-LOY-124** Given an order carrying one unit of product Alpha, a program giving 5 percent on orders of at least 2 units and a program giving 10 percent on orders of at least 4 units, When the order quantity is 1, Then no discount is applied; When it becomes 3, Then only the 5 percent discount is applied and its line description contains `Discount 5% on your order`.

## 9. Discount amounts and tax distribution

**AC-LOY-131** Given an order carrying 4 units at 100.00 with a 15 percent tax-excluded tax and 3 units at 100.00 with a 10 percent tax-included tax, totalling 672.73 tax excluded, 87.27 tax and 760.00 tax included, When a 10 percent order discount is claimed, Then exactly two reward lines are created: one of −40.00 carrying the 15 percent tax and one of −30.00 carrying the 10 percent tax; the order becomes 605.46 tax excluded, 78.54 tax and 684.00 tax included; and only the first line carries the point cost.

**AC-LOY-132** Given the same order and the same reward with `discount_max_amount` 50.00, Then the two reward lines are −26.32 and −19.74.

**AC-LOY-133** Given an order carrying 5 units of a chair at 100.00 with a 10 percent tax-included tax, 10 units of a bin at 100.00 with no tax, 7 units of a cabinet at 100.00 with a 15 percent tax-excluded tax, 2 units of a drawer at 100.00 with the same tax, and 3 units of a product at 100.00 with a 35 percent tax-included tax and a 50 percent tax-excluded tax, and three free product programs already applied giving 2 free chairs, 5 free bins and 3 free cabinets, so that the order totals 1 594.95 tax excluded and 1 901.11 tax included, When a 10 percent order discount is claimed, Then four reward lines are created, one per distinct tax combination, of −30.00, −50.00, −60.00 and −30.00, and the order becomes 1 435.45 tax excluded and 1 711.00 tax included.

**AC-LOY-134** Given the order of the previous scenario, When the reward line of one tax group is deleted, Then the three other lines of the same reward are deleted with it.

**AC-LOY-135** Given an order carrying 8 paid cabinets at 320.00 with a 15 percent tax-excluded tax, 2 free cabinets and 10 drawers at 25.00 with no tax, When a 20 percent discount on cabinets only is claimed, Then a single reward line of −512.00 carrying the 15 percent tax is created and the order total including tax is 2 605.20.

**AC-LOY-136** Given the same order and the same reward with `discount_max_amount` 200.00, Then the reward line is −173.91 and the order becomes 2 636.09 tax excluded and 2 994.00 tax included.

**AC-LOY-137** Given an order carrying 3 units at 100.00 with no tax and a promotion giving 10 percent on specific products matching everything, claimable three times, When the reward is claimed three times in a row with an evaluation between each claim, Then the order totals become 270.00, then 243.00, then 218.70.

**AC-LOY-138** Given an order of 10 chairs at 16.50 with no tax totalling 165.00 and a coupon worth 100.00 per point with 1 point, When the coupon is applied, Then the order total becomes 65.00 and stays 65.00 after a further evaluation.

**AC-LOY-139** Given that same order with a second program also worth 100.00 requiring a minimum amount of 100.00, When both are applied, Then the order total is 65.00; When the order quantity is raised to 15 (247.50) and the second discount is applied, Then the total is 47.50; When the quantity is lowered to 5 (82.50) and the order is re-evaluated, Then both discounts are removed and the total is 82.50.

**AC-LOY-140** Given an order of three lines of 100.00, one with a 15 percent tax-excluded tax and two with no tax, When a 10 percent order discount and then a coupon worth 288.50 are applied, Then the order total is exactly 0.00, the tax is 0.00 and the untaxed amount is 0.00; When the same two are applied in the reverse order, Then the total is 23.85, the untaxed amount is 22.71 and the tax is 1.14, and a further evaluation does not change these numbers.

**AC-LOY-141** Given a product priced 90.00 carrying a flat tax of 10.00 per unit, so that the order totals 100.00, and a percentage discount of 100 percent, Then the flat tax is not discounted and the order total cannot reach 0.00; Given the same order and a gift card of 100.00, Then the order total is exactly 0.00.

**AC-LOY-142** Given an order carrying two products of 100.00 each with a 15 percent tax-excluded tax, totalling 230.00, an electronic wallet of 115.00 and a coupon giving 100 percent on one of the two products, When the wallet reward is claimed, Then the order total is 115.00, the untaxed amount is 85.00 and the tax is 30.00; When the coupon is then applied, Then the order total is 0.00, the untaxed amount is −15.00 and the tax is 15.00.

**AC-LOY-143** Given a reward with `discount_applicability` `cheapest` and an order carrying 3 units at 30.00 and 2 units at 10.00, When a 50 percent cheapest-product discount is claimed, Then the discountable amount is the tax-included total of the 10.00 line divided by its quantity, that is 10.00, and the reward line is −5.00.

## 10. Free products

**AC-LOY-151** Given a program granting 1 point per unit of a cabinet with a minimum quantity of 3 and a reward giving 1 free cabinet for 3 points, When the order carries 2 cabinets, Then no free line exists; When it carries 3, Then one free line of quantity 1 exists with a point cost of 3; When it carries 6, Then the free quantity is 2; When it carries 75, Then it is 25; When it drops back to 6, Then it is 2 again.

**AC-LOY-152** Given a reward giving one free product Beta for 1 point granted per order on product Alpha, and an order carrying 4 units of Alpha and 1 unit of Beta, Then exactly 1 free Beta is given, because the free quantity is capped by what the order actually contains for that product.

**AC-LOY-153** Given a free product reward, Then its line carries the real product, the quantity earned, a discount percentage of 100, the product's own taxes mapped through the fiscal position, and a net amount of 0.00.

**AC-LOY-154** Given a free product reward whose reward product is also the rule product, When the free units are added, Then the points granted do not increase, because reward lines never count towards the quantity gate.

**AC-LOY-155** Given a multi-product free reward and the reward wizard, When no product is chosen, Then the first eligible product is proposed; When a product that is not eligible is forced, Then the operation is refused with `Invalid product to claim.`

## 11. Codes

**AC-LOY-161** Given an unknown code, When it is applied to an order, Then it is refused with `This code is invalid (<code>).` and the refusal is flagged "not found".

**AC-LOY-162** Given a card whose expiration date is yesterday, When its code is applied, Then it is refused with `This coupon is expired.`

**AC-LOY-163** Given a card whose balance is 0 and whose program's cheapest reward requires 1 point, When its code is applied, Then it is refused with `This coupon has already been used.`

**AC-LOY-164** Given a program of type `ewallet`, When a card code of that program is applied to an order, Then it is refused with `This program cannot be applied with code.`

**AC-LOY-165** Given a code already applied to an order whose program already has a reward line on that order, When the same code is applied again, Then it is refused with `This promo code is already applied.`

**AC-LOY-166** Given a coupon code applied to an order and the reward claimed, When the order is confirmed, Then the card's balance drops by the point cost; When a second order tries to apply the same code, Then it is refused with `This coupon has already been used.` for a single-use coupon.

**AC-LOY-167** Given two concurrent transactions applying the code of a program limited to one use, Then one of them obtains the program row lock and the other fails with a serialization error and is retried, so the program is used exactly once.

**AC-LOY-168** Given a code that unlocks a program with several rewards, When the code is applied through the coupon code wizard, Then the reward selection wizard opens listing exactly those rewards.

## 12. Confirmation and cancellation

**AC-LOY-171** Given an order whose reward lines would leave a card with a negative available balance, When the order is confirmed, Then the confirmation is refused with `One or more rewards on the sale order is invalid. Please check them.`

**AC-LOY-172** Given an order granting 25.00 points to a card holding 90.00 and spending 100.00 points on a reward, When it is confirmed, Then the card's balance becomes 15.00 and one history entry is created with `issued` 25.00, `used` 100.00, the description `Order <order name>` and a reference to the order.

**AC-LOY-173** Given that confirmed order, When it is cancelled, Then the history entry is deleted, the card's balance returns to 90.00 and the reward lines are deleted.

**AC-LOY-174** Given an order that created a bearer coupon of a `next_order_coupons` program and was confirmed, When it is cancelled, Then the coupon is deleted because it is not nominative, was created by that order and was never used.

**AC-LOY-175** Given an order that created a gift card which has already been spent on another order, When the first order is cancelled, Then the gift card survives because its use count is not zero.

**AC-LOY-176** Given an order with a claimable reward that the user did not add, When it is confirmed, Then the confirmation succeeds and an informational notification titled `Rewards Available` with the message `There are available rewards not added to this order.` is returned.

**AC-LOY-177** Given an order that earned a card of a program whose `applies_on` is `future`, When it is confirmed, Then the "at creation" communication of that card is sent immediately rather than queued.

**AC-LOY-178** Given an order carrying reward lines, When it is duplicated, Then the copy carries no reward line, no applied card, no activated code rule and no pending promise.

**AC-LOY-179** Given a confirmed order, When a reward line's point cost is changed from 30 to 50 on the same card, Then the card's balance drops by 20 more and the order's history entry for that card increases its `used` by 20.

**AC-LOY-180** Given a confirmed order, When a reward line is deleted, Then every line of that reward application is deleted and the whole point cost is given back to the card.

**AC-LOY-181** Given an order whose total including tax is 0.00, whose reward total is not 0 and automatic invoicing is enabled, When it is confirmed, Then an invoice is created and posted and, when it is ready to be sent, it is marked as sent and dispatched.

**AC-LOY-182** Given an order carrying only reward lines, When an invoice is requested, Then no invoice is produced, because a reward line may not be invoiced alone.

## 13. Gift cards and electronic wallets

**AC-LOY-191** Given an order carrying one gift card product of 50.00 with no tax, When the order is confirmed, Then one card with balance 50.00 is created, its code appears on the order's portal page with a copy button, and the gift card email with the printed gift card attached is sent to the order's customer.

**AC-LOY-192** Given an order of 80.00 with no tax and a gift card of 50.00, When the gift card code is applied and the reward claimed, Then a single reward line of −50.00 with no tax is created, the order total becomes 30.00 and the point cost is 50.00; When the order is confirmed, Then the card's balance becomes 0.00.

**AC-LOY-193** Given an order of 30.00 and a gift card of 50.00, When the reward is claimed, Then the reward line is −30.00, the order total is 0.00 and the card keeps 20.00.

**AC-LOY-194** Given an order of 200.00 and two gift cards of 100.00 each, When both are applied, Then the order total is 0.00 and each card is emptied.

**AC-LOY-195** Given an order of 100.00 with no tax, a gift card of 50.00 and a 10 percent code, When the gift card is applied first and the code second, Then, after the evaluation, the order total is 40.00, because the gift card is recomputed last against the discounted total of 90.00.

**AC-LOY-196** Given a gift card whose hidden discount product carries a 10 percent tax-included tax, When 100.00 is spent, Then the reward line has a tax-excluded base of −90.91, a tax of −9.09 and a tax-included total of −100.00.

**AC-LOY-197** Given an electronic wallet program and a customer owning a wallet with a positive balance, When any order of that customer is evaluated, Then the wallet card is attached automatically without any code and its reward becomes claimable.

**AC-LOY-198** Given an electronic wallet program, When an order carries a line of its own top-up product, Then the wallet reward's discountable amount excludes that line, so a wallet cannot pay for its own top-up.

**AC-LOY-199** Given an expired electronic wallet, Then its reward is not claimable.

**AC-LOY-200** Given a gift card program whose communication plan sends a template at card creation, an order whose salesperson's contact carries the address `sales@company.co`, and a copy of that order with no salesperson whose company contact carries the address `noreply@company.co`, When both orders are confirmed by an anonymous public visitor and each produces one gift card, Then exactly two messages are queued; the message of the first order has the salesperson's contact as its author and `sales@company.co` as its sender address, and the message of the second order has the company contact as its author and `noreply@company.co` as its sender address. A message is never left without a sender (section 4.3 of [entities.md](entities.md)).

## 14. Next-order coupons

**AC-LOY-201** Given a program of type `promotion` with `applies_on` `future`, a rule requiring 2 units and a free product reward, and an order carrying 1 unit, Then no coupon is generated; When the quantity becomes 2, Then exactly one coupon is generated with balance 0 and nothing is added to the order; When the quantity returns to 1, Then the coupon is not deleted but no further coupon is generated; When the quantity becomes 2 again, Then there is still exactly one coupon with balance 0 and no reward is claimable on that order.

**AC-LOY-202** Given a program of type `next_order_coupons` and an order of an identified customer that meets its rule, When the order is confirmed, Then the created card is owned by that customer and its code is emailed.

**AC-LOY-203** Given an online cart of the anonymous public visitor that earned a next-order coupon, When the shopper identifies themselves at checkout, Then the card's owner is rewritten to the real customer at the next evaluation.

## 15. Free shipping

**AC-LOY-211** Given an order with a shipping line priced 12.50 and a free shipping reward with no maximum, When the reward is claimed, Then a reward line of −12.50 carrying the shipping product's taxes is created and the shipping becomes free.

**AC-LOY-212** Given the same order and a free shipping reward with a maximum of 8.00, Then the reward line is −8.00.

**AC-LOY-213** Given a program whose rule requires a minimum purchase, When the order drops below that minimum, Then the free shipping reward line is removed at the next evaluation.

**AC-LOY-214** Given a shipping method that is free above 100.00 and an order of 120.00 partly paid with a gift card of 50.00, Then the shipping stays free, because the gift card line is excluded from the amount compared with the threshold.

**AC-LOY-215** Given a percentage discount and an order carrying a shipping line, Then the shipping line is not discounted and does not count towards the discountable amount.

**AC-LOY-216** Given an order whose only line is a free shipping reward line, When a user tries to delete it, Then the order remains valid and the reward may be removed only through the ordinary reward removal.

**AC-LOY-217** Given an automatic promotion giving 10 percent on the order with a rule requiring a minimum quantity of 2 and a minimum amount of 0, and an order carrying one unit of a product plus one shipping line of quantity 1, When the programs are evaluated, Then no reward line is created and the order still has exactly two lines, because the shipping line is a threshold-neutral line and does not count towards the quantity gate (`LOY-069`); When a second unit of the product is added, Then the quantity gate is met and the discount line is created.

## 16. Counter behavior

**AC-LOY-221** Given a counter whose currency differs from a program's currency, When the session is opened, Then that program is not loaded onto the device.

**AC-LOY-222** Given a program with a free product reward whose product is not available at a counter, When a session of a counter that publishes that program is opened, Then the opening is refused with `To continue, make the following reward products available in Point of Sale.` followed by a line naming the program and the product.

**AC-LOY-223** Given a gift card program with two rewards published at a counter, When a session is opened, Then it is refused with `Invalid gift card program. More than one reward.`

**AC-LOY-224** Given a gift card program whose rule grants 2 points per unit of currency spent, When a session is opened, Then it is refused with `Invalid gift card program rule. Use 1 point per currency spent.`

**AC-LOY-225** Given a ticket and a scanned coupon code whose card belongs to another customer and whose program is not a gift card program, Then the redemption returns `This coupon is invalid (<code>).`

**AC-LOY-226** Given a ticket and a scanned code whose program starts tomorrow, Then the redemption returns `This coupon is not yet valid (<code>).`

**AC-LOY-227** Given a ticket and a scanned card whose balance pays for no reward of its program, Then the redemption returns `No reward can be claimed with this coupon.`

**AC-LOY-228** Given a ticket and a scanned gift card whose card has no source document, Then the cashier is asked `This gift card is not linked to any order. Do you really want to apply its reward?`; a refusal answers `Unpaid gift card rejected.`

**AC-LOY-229** Given a device that has already activated a promotional code rule, When the same code is entered again, Then the device answers `That promo code program has already been activated.`

**AC-LOY-230** Given a ticket with a point change on a card whose balance on the server is smaller than the points to spend, When the payment is validated, Then the server answers `There are not enough points for the coupon: <code>.` and the device rewrites its local balances.

**AC-LOY-231** Given a ticket that creates a physical gift card whose typed code already exists on the server, When the payment is validated, Then the server answers `The following codes already exist in the database, perhaps they were already sold?` followed by the colliding code.

**AC-LOY-232** Given a confirmed ticket with a locally created nominative card and a customer who already owns a card of that program, When the confirmation exchange runs, Then the points land on the existing card and no duplicate card is created.

**AC-LOY-233** Given a confirmation exchange that is replayed for the same ticket, Then no point is applied twice, because an entry whose program already has a history entry for that ticket is dropped.

**AC-LOY-234** Given a ticket that sells a physical gift card with a typed code and an amount of 75.00, When it is confirmed, Then a card with that code and a balance of 75.00 is created, its source ticket is set and a history entry describes the assignment.

**AC-LOY-235** Given a restaurant ticket with courses, When a reward line is produced, Then it is assigned to the last course of the ticket.

**AC-LOY-236** Given a ticket onto which a sales order is being settled, Then no program is evaluated during the settlement, and the evaluation runs exactly once when the settlement ends.

**AC-LOY-237** Given a ticket line that came from a settled sales order, Then it earns no loyalty point.

**AC-LOY-238** Given a free product reward claimed at a counter, Then the ticket carries the paid product line unchanged plus a negative line on the hidden discount product cancelling the price of the free units.

**AC-LOY-239** Given a ticket carrying a gift card top-up line, When the cashier tries to refund it, Then the notification `Refunding a top up or reward product for an eWallet or gift card program is not allowed.` is shown and the refund is refused.

**AC-LOY-240** Given a restaurant counter carrying an automatic promotion that gives 10 percent on the order for a minimum quantity of 1, and a ticket opened at a table with one product priced 2.20, so that the ticket total is 1.98 after the discount, When the cashier leaves the table and later selects that table again, Then the reward line is still on the ticket, the rewards are re-evaluated once as part of selecting the table, and the ticket total is still 1.98.

**AC-LOY-241** Given an electronic wallet program, a ticket that sold one unit of a product and was invoiced, and a refund ticket of that ticket paid with the electronic wallet refund payment, When the refund ticket is invoiced, Then the credit note contains one line for the refunded product and the quantity on that line is 1, a positive number, because a credit note expresses the returned quantity positively and the sign is carried by the document type, not by the quantity.

## 17. Storefront behavior

**AC-LOY-251** Given a cart and an automatic program with exactly one non-nominative, non-multi-product reward, When the cart is updated, Then the reward is claimed automatically without the shopper acting.

**AC-LOY-252** Given that cart, When the shopper deletes the reward line, Then the reward is recorded among the manually removed rewards and is not claimed again automatically.

**AC-LOY-253** Given a visitor with no cart who follows a coupon link, Then the landing page shows `The coupon will be automatically applied when you add something in your cart.` and the code stays in the session; When the visitor adds a product, Then the code is applied at the next cart evaluation.

**AC-LOY-254** Given a visitor with a cart who follows a valid coupon link, Then the landing page shows `The following promo code was applied on your order: <code>` and, when exactly one card offers exactly one non-multi-product reward, that reward is already claimed.

**AC-LOY-255** Given a cart and a code that matches nothing, When it is submitted through the promotional code form, Then the text is handed to the pricelist code handling; when that also fails, the shopper sees the pricelist handling's own outcome.

**AC-LOY-256** Given a cart with a 10 percent discount split over three tax groups, Then the cart shows exactly one discount line whose amount is the sum of the three, with no tax.

**AC-LOY-257** Given a cart carrying a free product reward of quantity 2, Then the cart quantity badge does not count those 2 units.

**AC-LOY-258** Given a cart whose reward expired between the moment it was displayed and the moment payment is finalized, Then the payment is refused with `Cannot process payment: applied reward was changed or has expired.` followed by a new line and `Please refresh the page and try again.`

**AC-LOY-259** Given a cart carrying a free gift whose product has a sale price of 0.00, Then the checkout is not blocked by the zero-priced line rule.

**AC-LOY-260** Given a draft online cart with an applied card untouched for five days and an abandonment delay of four days, When the cleanup runs, Then the card is detached and the reward lines it paid for are removed.

**AC-LOY-261** Given a shopper adding a product that a reward already gave for free, Then a separate paid line is created and the free line is left untouched.

**AC-LOY-262** Given a cart and a customer owning a gift card, Then the cart lists the gift card reward with the card's masked code, showing only the last four characters right-aligned in a fourteen-character field.

**AC-LOY-263** Given a cart, a discount-code program whose code is `FREE` and whose single reward is a free shipping reward with a maximum amount of 6.00, and a shipping method whose price is at least 6.00, When the code is applied, the reward is claimed and the shopper picks that shipping method from the express checkout address step, Then the summary returned to the express payment sheet carries a shipping discount of −6.00, expressed as the minor-unit integer −600 for a currency with two decimal places.

**AC-LOY-264** Given a cart whose products total 100.00 before any shipping, a shipping method priced 10.00 already written on the cart, and the same discount-code program with a free shipping reward capped at 2.00, When the code is applied, the reward is claimed and the express payment values of the cart are computed, Then the amount offered to the express payment sheet is 100.00, expressed as the minor-unit integer 10000: both the shipping line and the free shipping reward line are excluded from it, because the shipping method and its discount are chosen inside the express payment sheet and are added afterwards.

## 18. Portal

**AC-LOY-271** Given a customer owning a loyalty card and an electronic wallet, Then the portal home lists both, grouped by program; an expired card and a card of an archived program are not listed.

**AC-LOY-272** Given a customer requesting the history page of a card they do not own, Then the request redirects to the portal home.

**AC-LOY-273** Given a card with seven history entries, When the portal dialog is opened, Then it shows the five most recent entries, each with a signed formatted movement and a link to the sales order behind it when there is one.

**AC-LOY-274** Given a card with a balance that pays for five rewards, When the portal dialog is opened, Then it shows the three most expensive of them, ordered by required points descending.

**AC-LOY-275** Given an electronic wallet program whose trigger products are published, When the portal dialog of a wallet card is opened in the online shop, Then the published trigger products are listed with their formatted prices.

## 19. Permissions and scoping

**AC-LOY-281** Given a user who is only an internal user, Then they may read no program, no rule, no reward, no card and no history entry.

**AC-LOY-282** Given a Salesperson, Then they may read a program but not modify it, may read and modify a card but not delete it, and may run the generation wizard.

**AC-LOY-283** Given a Salesperson creating a quotation that earns a loyalty card, Then the card and the pending promise are created successfully even though the Salesperson has no create right on cards.

**AC-LOY-284** Given a Salesperson cancelling an order carrying pending promises, Then the cancellation succeeds even though the Salesperson has no delete right on pending promises.

**AC-LOY-285** Given a program whose company is company A and a user whose allowed companies are company B only, Then the program, its rules, its rewards, its cards and its history entries are invisible to that user.

**AC-LOY-286** Given a program whose company is empty, Then it is visible to every user of every company.

**AC-LOY-287** Given a program whose company is a parent of the user's allowed company, Then it is visible to that user and applicable to that company's orders.

## 20. Idempotence and ordering

**AC-LOY-291** Given any order in any state, When the evaluation is run twice in a row without any other change, Then the second run writes no line, creates no card, deletes nothing and leaves every amount identical.

**AC-LOY-292** Given an order carrying an ordinary discount and a gift card payment, When the evaluation runs, Then the payment line is always recomputed after the discount line, whatever order they were claimed in.

**AC-LOY-293** Given a reward whose description was edited by hand on a line, When the evaluation recomputes that reward and the product is unchanged, Then the edited description is preserved.

**AC-LOY-294** Given a reward applied twice on the same order (a payment reward from two cards), Then the two applications carry different reward grouping codes and are deleted independently.

**AC-LOY-295** Given an order whose prices are reset to the pricelist and that carries at least one reward line, Then the full evaluation runs and every percentage discount follows the new prices.
