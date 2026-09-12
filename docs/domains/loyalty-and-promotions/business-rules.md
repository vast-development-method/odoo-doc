# Business rules

The complete rule catalogue of the Loyalty and Promotions domain: validations, guards, permissions
per operation, company and currency consistency rules, locking rules, uniqueness rules, rounding
rules, date rules and the exact text of every message shown to the user. Every rule carries a stable
identifier of the form LOY-nnn, unique within this file, so that the other files of the folder can
cite it. Section 17 indexes every identifier, and section 18 maps them back to the two schemes used
before this folder was consolidated.

A note on message wording: user-facing messages are reproduced exactly as the user sees them, in
quotation marks, except that the name of the reference product is removed and channel names that
appear abbreviated in the interface are written in full ("point of sale" in place of the abbreviated
form). A placeholder inside a message is written between angle brackets and its content is described
in words.

## 1. Program definition

**LOY-001** A Loyalty Program must carry a name. The name is translatable and is never shown to the customer.

**LOY-002** A Loyalty Program must carry a currency. When the company of the program is set or changed and that company has a currency, the program currency is set to it; the user may override it afterwards.

**LOY-003** Every pricelist listed in `pricelist_ids` must have the same currency as the program. Violation message: "The loyalty program's currency must be the same as all it's pricelists ones." The selection list of the field is restricted to pricelists of the program currency, so the rule can only be violated by a programmatic write or by changing the currency after the pricelists were chosen.

**LOY-004** When both `date_from` and `date_to` are set, `date_from` must be earlier than or equal to `date_to`. Violation message: "The validity period's start date must be anterior or equal to its end date." Both bounds are inclusive.

**LOY-005** A Loyalty Program must end every write with at least one reward. Violation message: "A program must have at least one reward." The check is suspended for a write that changes the program type, because such a write deletes the old rewards and creates the new ones in a single operation.

**LOY-006** When `limit_usage` is true, `max_usage` must be strictly positive. This is enforced at the database level. Violation message: "Max usage must be strictly positive if a limit is used."

**LOY-007** Changing `program_type` rewrites `applies_on`, `trigger`, `portal_visible`, `portal_point_name`, the rules, the rewards and the communication plan with the family defaults of the new type. Every field the user had edited on those collections is lost. The screen forbids the change once the program has at least one card.

**LOY-008** `portal_point_name` is forced to the currency symbol for a program of type `gift_card` or `ewallet`, and is left to the family default or to the user's value for every other type. When the currency has no symbol the name becomes the empty string.

**LOY-009** On creation, `trigger_product_ids` is dropped from the submitted values unless the program type is `gift_card` or `ewallet`, because writing it would overwrite the product filters that the family defaults just installed on the rules.

**LOY-010** Writing `active` on a program propagates it to every rule, every reward, every communication rule and every reward discount product of the program, including records that are currently archived.

**LOY-011** A program whose `active` is true may not be deleted. Violation message: "You can not delete a program in an active state"

**LOY-012** A program that has issued at least one card may not be deleted, because a card's reference to its program is a restricted reference. The cards must be deleted first.

**LOY-013** Unarchiving a program re-runs the code uniqueness rules of its rules. When another active rule already carries one of the codes, or when two programs unarchived in the same operation share a code, the operation fails with "The promo code must be unique."

**LOY-014** Setting the point-of-sale printed document on a gift card or electronic wallet program requires the email template to be set first. Violation message: "You must set 'Email template' before setting 'Print Report'." The two names in the message are the on-screen labels of the two fields.

**LOY-015** Writing the simplified email template field on a gift card or electronic wallet program rewrites the whole communication plan: clearing it deletes every rule of the plan; setting it on a program with no plan creates one rule with trigger `create`; setting it on a program that already has rules rewrites every rule to trigger `create` with that template. On any other program type the write does nothing.

**LOY-016** Setting `pos_ok` to false empties `pos_config_ids`.

**LOY-017** A program is only usable at a counter whose currency is the program currency. This is a filter, not a validation: a program in another currency is simply never loaded onto that counter.

## 2. Rule definition

**LOY-018** `reward_point_amount` must be strictly positive. Enforced at the database level. Violation message: "Rule points reward must be strictly positive."

**LOY-019** `reward_point_split` may not be true when the program's `applies_on` is `both` or the program type is `ewallet`. Violation message: "Split per unit is not allowed for Loyalty and eWallet programs." Splitting means issuing several cards, which makes no sense for a balance that must stay unique per customer.

**LOY-020** The split option only takes effect when the program's `applies_on` is `future` and `reward_point_mode` is not `order`. In every other configuration it is ignored.

**LOY-021** Among all active rules, a promotional code is unique. Violation message: "The promo code must be unique." The check covers both the rules being written together and the rules already in the database.

**LOY-022** No active card may carry a code that an active rule carries. Violation message on the rule side: "A coupon with the same code was found." Violation message on the card side: "A trigger with the same code as one of your coupon already exists."

**LOY-023** When the website capability package is present, LOY-021 is relaxed: two rules may share a code as long as they are not both reachable from the same website. A rule bound to a website conflicts with another rule bound to the same website and with any rule bound to no website; two rules bound to different websites never conflict. The message is unchanged.

**LOY-024** Writing a code sets `mode` to `with_code`; clearing the code sets `mode` to `auto`. Setting `mode` to `auto` clears the code. The two derivations are mutually consistent, so a user can drive the pair from either field.

**LOY-025** Writing a code regenerates `promo_barcode` with a freshly generated machine-readable code. A barcode is therefore never reused across two different promotional codes.

**LOY-026** A rule with `mode` equal to `with_code` contributes nothing to a document until its code has been entered on that document.

**LOY-027** A rule whose product filter is empty matches every product, **except** on a program of type `gift_card`, where such a rule is ignored entirely so that a misconfigured gift card program issues nothing.

## 3. Reward definition

**LOY-028** `required_points` must be strictly positive. Enforced at the database level. Violation message: "The required points for a reward must be strictly positive."

**LOY-029** For a reward of type `product`, `reward_product_qty` must be strictly positive. Enforced at the database level. Violation message: "The reward product quantity must be strictly positive."

**LOY-030** For a reward of type `discount`, `discount` must be strictly positive. Enforced at the database level. Violation message: "The discount must be strictly positive."

**LOY-031** A reward product may not be a combination product. Violation message: "A reward product can't be of type “combo”."

**LOY-032** Every reward owns exactly one hidden discount product, created automatically when the reward is created or when its description is written while it has none. The product is a service, not sellable, not purchasable, priced zero, with no customer taxes, no vendor taxes and the "ordered quantities" invoicing policy. It is never copied when the reward is duplicated, so a duplicate receives its own product.

**LOY-033** Writing the reward description renames its hidden discount product. Updating the description translations updates the product name translations in the same languages.

**LOY-034** Archiving a reward archives its hidden discount product; unarchiving reactivates it.

**LOY-035** Deleting a single reward that is already used by at least one sales order line, or by at least one point-of-sale line, archives it instead of deleting it. The document keeps a valid reference.

**LOY-036** Deleting a reward re-runs LOY-005 on its program, because removing a collection member does not by itself trigger the collection validation.

**LOY-037** A reward is a **global discount** when its type is `discount`, its applicability is `order` and its mode is `per_order` or `percent`. Only one global discount may be applied to a document at a time.

## 4. Card rules

**LOY-038** A card code is globally unique. Enforced at the database level. Violation message: "A coupon/loyalty card must have a unique code."

**LOY-039** A card code is generated as the three characters `044` followed by eleven characters taken from a randomly generated universally unique value, giving a fourteen-character code that the shipped barcode rule recognizes (it matches codes beginning with `043` or `044`).

**LOY-040** An expiration date may not be set on a card of a program of type `loyalty`. The entry is refused as soon as it is typed, with "Expiration date cannot be set on a loyalty card."

**LOY-041** A card whose `expiration_date` is strictly earlier than the reference date of the document is excluded from the claimable rewards, is removed from the document's applied cards at the next evaluation, and cannot be applied by code ("This coupon is expired.").

**LOY-042** Creating a card runs the "at creation" communication plan of its program, unless the calling context suppresses loyalty emails or suppresses sending. A card without a resolvable recipient sends nothing.

**LOY-043** Writing the balance of a card runs the milestone communication plan, unless the calling context suppresses loyalty emails. Only cards with an owner, with a resolvable recipient and with a strictly increased balance are considered, and only the highest crossed milestone is sent.

**LOY-044** Archiving a card first deletes every pending promise that links it to a draft sales order.

**LOY-045** The recipient of a card communication is the first non-empty value among the card's owner, the customer of the sales order that generated it, and the customer of the point-of-sale ticket that generated it.

**LOY-046** A card's balance is read through the program currency's rounding step only when at least one rule of the program grants points per unit of currency spent. For a program that grants points per order or per unit paid, the raw value is used, so a currency with a rounding step larger than one cannot collapse a single point to zero.

**LOY-047** A card of a program whose `applies_on` is `future` and that was created by the document being evaluated is never claimable on that document.

**LOY-048** A card whose owner differs from the document's customer has its pending promise on that document set to zero and deleted at the next evaluation.

**LOY-049** A card owned by the public customer has its owner rewritten to the document's customer as soon as the document names a real customer.

**LOY-050** A card that this document promises points to, whose program has `applies_on` equal to `current`, and that no reward line uses, is deleted at confirmation and whenever the reward selection wizard closes. Such a card could never be spent.

**LOY-051** A card that this document created, whose program is not nominative, and whose use count is zero, is deleted when the document is cancelled.

**LOY-052** The pair (order, card) of a pending promise is unique. Enforced at the database level. Violation message: "The coupon points entry already exists."

## 5. Applicability

**LOY-053** A program applies to a document only when it is active, its channel flag for that document's channel is true, its company is empty or is the document's company or that company's parent, its pricelist restriction is empty or contains the document's pricelist, and the document's reference date lies within the inclusive validity window.

**LOY-054** For a document that belongs to a website, the sales channel flag is replaced by the online shop flag, and the program's website restriction must be empty or equal to the document's website.

**LOY-055** The reference date of a document is today's date in the evaluation time zone, unless the document has at least one payment transaction in state "done" or "authorized", in which case it is the date of the earliest such transaction converted into the evaluation time zone.

**LOY-056** The evaluation time zone is the company contact's time zone, then the value of the system parameter `loyalty.timezone`, then coordinated universal time. For an online cart, the website salesperson's time zone takes precedence.

**LOY-057** A program whose `limit_usage` is true and whose total document count has reached `max_usage` is excluded from the automatic candidates, and its code is refused with "This code is expired (<code>)."

**LOY-058** The total document count of a program is the number of distinct sales orders carrying one of its rewards plus the number of distinct point-of-sale tickets carrying one of its rewards. A document counts once per program however many reward lines it carries.

**LOY-059** A program of type `ewallet` that names no trigger product grants nothing: the evaluation of its rules stops at the first rule. This prevents a wallet from being credited by every purchase.

**LOY-060** A program with no rule at all and `applies_on` equal to `current` is considered to have matched every gate, so a coupon program without rules is claimable as soon as its card is applied.

**LOY-061** A non-nominative program whose gates were not all met reports, in this order of priority: "This program requires a code to be applied." when no code gate was passed; then "To take advantage of this offer, your order must include at least <amount> <currency name> of the eligible products." when no amount gate was passed, where the amount is the smallest `minimum_amount` among the program's rules; then "You don't have the required product quantities on your sales order."

**LOY-062** A nominative program is reported as applicable even when it grants zero points, so that the customer's existing balance remains claimable. The only error it can report is "This program is not available for public users.", raised when the document's customer is the public customer and the document does not allow nominative programs.

**LOY-063** A sales order always allows nominative programs. An online cart allows them only when the visitor is not the anonymous public user.

**LOY-064** A discount reward line created by an **automatic** program does not reduce the amount used to judge a minimum purchase; a discount reward line created by a program that needed a code does. A line is also never counted against its own program's minimum purchase.

**LOY-065** A line that is an item of a combination product never counts on its own; the combination line contributes the sum of its items instead.

**LOY-066** Quantities are converted into the product's reference unit of measure before they are compared with `minimum_qty`.

**LOY-067** Reward lines are excluded from the lines that count towards the quantity gate, so free units never earn further free units.

**LOY-068** In the `money` grant mode, lines that carry a reward of a gift card program, of an electronic wallet program, or of a program of the same type as the program being evaluated are excluded from the paid amount. Every other discount reduces the paid amount and therefore the points earned.

**LOY-069** When the shipping capability package is present, shipping lines and free shipping reward lines never count towards a minimum purchase and never earn points.

**LOY-070** Points granted in the `money` mode are truncated downward to two decimal places, never rounded up.

**LOY-071** A program may only be attached once to a document. A second attempt is refused with "This program is already applied to this order."

**LOY-072** A program that does not match the program filter is refused with "The program is not available for this order."

## 6. Claiming a reward

**LOY-073** A reward is claimable only when the points available on the card are greater than or equal to `required_points`.

**LOY-074** A discount reward is not claimable when the discountable amount of the document is zero, unless a payment reward is already applied and the reward being examined is not itself a payment reward.

**LOY-075** A discount reward that does not belong to a payment program and is already applied on the document is not claimable again. A payment reward may be applied once per card.

**LOY-076** A free product reward is not claimable when its product is archived. A reward with a product tag needs at least one active product behind the tag; a reward without a tag needs its reward product to be active.

**LOY-077** When a global discount is already applied and a second one is claimed, the better one wins (section 7 of [calculations.md](calculations.md)). When the applied one wins, the claim is refused with "A better global discount is already applied." When the candidate wins, the applied lines are reset and reused.

**LOY-078** When two global discounts both exceed the discountable amount, the smaller one is considered better, so that the customer keeps the more valuable voucher.

**LOY-079** A reward of a non-nominative program whose `applies_on` is `future` may not be claimed on the document that created its card. Refusal message: "The coupon can only be claimed on future orders."

**LOY-080** A reward may not be claimed when the card does not hold enough points. Refusal message: "The coupon does not have enough points for the selected reward."

**LOY-081** A discount with nothing to discount is refused with "There is nothing to discount", except when a payment reward is applied on the document, in which case a placeholder line named `TEMPORARY DISCOUNT LINE` with quantity zero, price zero and point cost zero is produced instead.

**LOY-082** A free product reward claimed with a product that is not among its eligible products is refused with "Invalid product to claim."

**LOY-083** Only one free shipping reward may be applied at a time. When one is applied, every other shipping reward disappears from the claimable set.

**LOY-084** Payment rewards are always recomputed after every other reward, so that a gift card pays the already-discounted total.

**LOY-085** A payment reward may not pay for its own program's trigger products: those lines are removed from its discountable amount.

**LOY-086** Fixed-amount taxes are never discounted by a reward that does not belong to a payment program. A payment reward discounts them, because it must be able to bring the total to zero.

**LOY-087** A discount is never larger than the document's total including tax, and never larger than `discount_max_amount` when that value is not zero.

**LOY-088** In the `per_point` mode of a non-payment program, the available points are first truncated downward to a whole multiple of `required_points`, because a reward may not be granted partially. For a payment program the available points are used as they are.

**LOY-089** In the `per_point` mode, the point cost is the discount actually granted divided by the per-point value, rounded by the card's currency. In every other mode the cost is `required_points`, or the whole available balance when `clear_wallet` is true.

**LOY-090** A discount split across several tax combinations produces one line per combination, and only the first of those lines carries the point cost.

**LOY-091** A reward line description is suffixed with ` - On products with the following taxes: <names>` only when the discount was split into more than one line and at least one of the mapped taxes has a name.

**LOY-092** When a reward is recomputed and a line for the same product already exists, that line's description is preserved, so a manual edit survives.

## 7. Confirmation, cancellation and invoicing

**LOY-093** Confirming an order whose available points on any involved card are negative is refused with "One or more rewards on the sale order is invalid. Please check them."

**LOY-094** Confirming an order re-evaluates it first, so a reward that has become invalid is removed before the points move.

**LOY-095** Confirming an order writes one Loyalty History movement per card involved, carrying both the points granted and the points spent, with the description `Order <order display name>` and a reference to the order.

**LOY-096** Confirming a single order that still has claimable rewards shows an informational notification titled "Rewards Available" with the message "There are available rewards not added to this order." The notification never blocks the confirmation.

**LOY-097** Confirming an order sends the "at creation" communication of every card the order granted points to whose program has `applies_on` equal to `future`, with immediate delivery rather than queued delivery.

**LOY-098** Cancelling a previously confirmed order deletes its history movements, reverses the point changes on every card, deletes its reward lines, deletes the non-nominative cards it created that were never used, and deletes its pending promises.

**LOY-099** Duplicating an order deletes every reward line of the copy and copies neither the applied cards, nor the activated code rules, nor the pending promises.

**LOY-100** Recomputing the prices of an order that carries at least one reward line re-runs the full evaluation.

**LOY-101** A reward line may never be invoiced alone.

**LOY-102** An invoice line that comes from a reward of type `discount` is classified as a discount line for reporting; the same applies to an invoice line whose product is one of the reward discount products of the point-of-sale ticket behind the invoice.

**LOY-103** An order whose total including tax is zero and whose reward total is not zero is invoiced anyway when automatic invoicing is enabled: the lines are forced to the "ordered quantities" invoicing policy, the invoice is created and posted, and, when it is ready, it is marked as sent and dispatched with the configured invoice email template.

**LOY-104** On a confirmed order, creating, rewriting or deleting a reward line moves the points on the card immediately and updates the order's history movement by the same amount.

**LOY-105** Deleting one line of a reward deletes every line of the same reward application.

**LOY-106** Reward lines are read-only on the customer portal page and are excluded from the sellable-line checks.

**LOY-107** The quantity, the unit price and, once the order is confirmed, the taxes of a reward line are read-only on the order screen.

## 8. Code application

**LOY-108** A code is first looked up among the rules that match the rule filter, then among the cards. A rule and a card may never carry the same code (LOY-022).

**LOY-109** A code whose rule is already activated on the document and whose program already has a reward line on the document is refused with "This promo code is already applied."

**LOY-110** A code that matches nothing, or whose card belongs to an archived program, a program without rewards or a program that does not match the program filter, is refused with "This code is invalid (<code>)." and the refusal is flagged "not found", which lets the online shop fall back to interpreting the text as a pricelist code.

**LOY-111** A card whose balance is strictly less than the smallest `required_points` among its program's rewards is refused with "This coupon has already been used."

**LOY-112** A program of type `loyalty` or `ewallet` may never be applied by code. Refusal message: "This program cannot be applied with code."

**LOY-113** Before the usage ceiling is checked, the program row is locked for update without waiting. A concurrent transaction that already holds the lock makes the current one fail with a serialization error, and the whole request is retried. This is what makes the usage ceiling exact under concurrency.

**LOY-114** A successful code application records the rule among the document's activated code rules and the card among its applied cards, then either re-evaluates the document (when the program already grants points on it) or attaches the program with the full check.

**LOY-115** When the attach step refuses and the program is not nominative, or it is nominative but no card was found, the rule activation and the card attachment are undone before the refusal is returned. A refusal flagged "already applied" leaves the card attached.

## 9. Permissions

**LOY-116** No access is granted to any loyalty entity by the internal user group (`base.group_user`) alone. Access is granted by the sales groups and by the point-of-sale groups; a deployment with neither installed exposes nothing.

**LOY-117** A Salesperson (the own-documents sales group `sales_team.group_sale_salesman`) may read programs, rules, rewards and communication rules, may read and update cards, may create cards through the generation wizard, may read, create and update history movements, and may create and update the pending promises of their own orders but may not delete them. A Sales Administrator (the sales administration group `sales_team.group_sale_manager`) may additionally create, update and delete programs, rules, rewards, communication rules and pending promises, and may create but not delete cards.

**LOY-118** A Point of Sale Cashier (the counter user group `point_of_sale.group_pos_user`) may read programs, rules, rewards and communication rules, may read and update cards, may run the generation wizard and the balance update wizard, and may create history movements. A Point of Sale Administrator (the counter management group `point_of_sale.group_pos_manager`) may additionally create, update and delete programs, rules, rewards and communication rules, and may create cards.

**LOY-119** Nobody may delete a card through an access rule: the delete permission is not granted to any group on the card entity.

**LOY-120** Programs, rules, rewards, cards and history movements are filtered by a company record rule: a record is visible when its company is empty, is one of the user's allowed companies, or is a parent of one of them.

**LOY-121** Card creation, card deletion, pending promise creation and pending promise deletion performed by the evaluation run with elevated rights, so that a salesperson who may not create cards can still save a quotation that earns one.

**LOY-122** The image of a reward discount product is readable by anonymous visitors, so that a reward can be pictured in a public cart.

## 10. Rounding, currency and date rules

**LOY-123** Point values are stored with two decimal places. Every inexact point computation truncates downward.

**LOY-124** Monetary reward amounts are rounded by the document currency when they are written on a line. The distribution factor is kept at full precision, so the sum of the rounded lines may differ from the intended ceiling by one rounding step per line.

**LOY-125** `minimum_amount` is converted from the program currency into the document currency at today's rate using the program's company, or the acting company when the program has none.

**LOY-126** `discount_max_amount` and a `per_order` or `per_point` discount value are converted from the program currency into the document currency at today's rate.

**LOY-127** Both bounds of the validity window are inclusive: a program whose `date_to` is the reference date is still valid on that date.

**LOY-128** A card whose `expiration_date` equals the reference date is still usable; it becomes unusable the following day.

**LOY-129** Two monetary amounts are compared through the document currency's rounding step; a difference smaller than half a step is treated as equality.

## 11. Consistency rules

**LOY-130** A program, its rules, its rewards, its cards and its history movements always share the same company, because the company is mirrored from the program and stored on each of them.

**LOY-131** A card always has the currency and the point name of its program; neither can be set independently.

**LOY-132** A reward line always carries the taxes it compensates, not the taxes of its hidden discount product, except for a gift card payment line, which carries the taxes of the gift card discount product.

**LOY-133** A free product line always carries the product's own taxes mapped through the document's fiscal position, and a discount percentage of 100.

**LOY-134** The reward grouping code is identical on every line of one reward application and different for every application, including two applications of the same reward.

## 12. Point-of-sale specific rules

The complete contract is in [point-of-sale-application.md](point-of-sale-application.md); the rules that a replacement must enforce on the server side are listed here.

**LOY-135** A counter session may not open while a reward product of one of its programs, or a gift card rule product of one of its gift card programs, is not available at the counter. Refusal message: "To continue, make the following reward products available in Point of Sale." followed, for each offending product, by a new line, a tab and either "Program: <program name>, Reward Product: <product name>" or "Program: <program name>, Rule Product: <product name>".

**LOY-136** A gift card program usable at a counter must have exactly one rule and exactly one reward, the rule must grant one point per unit of currency spent, and the reward must be a discount of one unit of currency per point. Refusal messages, in the order they are checked: "Invalid gift card program. More than one reward.", "Invalid gift card program. More than one rule.", "Invalid gift card program rule. Use 1 point per currency spent.", "Invalid gift card program reward. Use 1 currency per point discount."

**LOY-137** When the counter is set to print gift cards, the gift card program must have an email template and a printed document. Refusal messages: "There is no email template on the gift card program and your point of sale is set to print them." and "There is no print report on the gift card program and your point of sale is set to print them."

**LOY-138** The server revalidates every point change the device computed before a ticket is accepted. A card that no longer exists or whose program is archived produces "Some coupons are invalid. The applied coupons have been updated. Please check the order." together with the list of removed cards. A card whose balance is smaller than the points the device wants to spend produces "There are not enough points for the coupon: <code>." together with the current balances. A new code that already exists in the database produces "The following codes already exist in the database, perhaps they were already sold?" followed by a new line and the list of colliding codes.

**LOY-139** The counter code redemption service refuses, in this order: an unknown code or an archived program with "This coupon is invalid (<code>)."; an expired card, a program past its end date, or a program that reached its usage ceiling with "This coupon is expired (<code>)."; a program that has not started with "This coupon is not yet valid (<code>)."; a program with no reward the balance can pay for with "No reward can be claimed with this coupon."; a program restricted to pricelists that do not include the counter's pricelist with "This coupon is not available with the current pricelist."; and a program of type `promo_code` reached through a card code with "This programs requires a code to be applied."

**LOY-140** The counter code redemption searches cards of the counter's programs whose owner is empty or is the ticket's customer, or whose program type is `gift_card`, ordered by owner and then by balance descending, and takes the first one. The ordering lets a bearer coupon be used several times when several identical ones exist.

## 13. Storefront specific rules

The complete contract is in [storefront-application.md](storefront-application.md).

**LOY-141** A reward is claimed automatically in the cart only when its program is not nominative, its program has exactly one reward, the reward is not a multi-product free product reward, the reward is not in the cart's list of manually removed rewards, and the reward is not already applied.

**LOY-142** Removing a reward line from the cart adds its reward to the cart's list of manually removed rewards, which prevents the automatic claiming from putting it back.

**LOY-143** A coupon link visited without a cart stores the code in the session and answers "The coupon will be automatically applied when you add something in your cart." The stored code is applied at the first evaluation of the cart that follows.

**LOY-144** Before a payment is finalized, the cart is re-evaluated; when the total including tax changed, the payment is refused with "Cannot process payment: applied reward was changed or has expired." followed by a new line and "Please refresh the page and try again."

**LOY-145** Several discount lines produced by one reward are merged into a single visual line in the cart; the merged line carries no tax and its amount is the sum of the underlying lines.

**LOY-146** Reward lines do not count towards the cart quantity badge.

**LOY-147** A reward line priced at zero does not block the checkout, and reward lines are excluded from the rule that forbids zero-priced lines.

## 14. Messages raised by the wizards

| Wizard | Condition | Message |
|---|---|---|
| Card Generation Wizard | No program set | "Can not generate coupon, no program is set." |
| Card Generation Wizard | Quantity zero or negative | "Invalid quantity." |
| Card Balance Wizard | New balance equal to the old one, or negative | "New Balance should be positive and different then old balance." |
| Coupon Entry Wizard | No order | "Invalid sales order." |
| Coupon Entry Wizard | The code application refused | the refusal text, verbatim |
| Reward Selection Wizard | No reward picked | "No reward selected." |
| Reward Selection Wizard | No card offers the picked reward | "Coupon not found while trying to add the following reward: <reward description>" |
| Coupon Sharing Wizard | A `coupons` program without a card | "A coupon is needed for coupon programs." |
| Coupon Sharing Wizard | The website differs from the program's website | "The shared website should correspond to the website of the program." |
| Coupon Sharing Wizard | Opened with both a card and a program, or with neither | "Provide either a coupon or a program." |

## 15. Guards on records owned by other domains

These five guards live on entities owned by other domains but are installed and enforced by this domain, because they protect the master data that programs and rewards depend on. They are cited from [workflows.md](workflows.md) section 5.4 and from [configuration.md](configuration.md) section 8.1.

**LOY-148** Archiving a Product is refused while at least one **active** Loyalty Reward either names that product as its hidden discount product or lists it among its discounted products. The check runs on any write that sets `active` to false on a set in which at least one product is currently active; a write that leaves `active` untrue or that reactivates a product is never checked. The search for the offending reward is performed with elevated rights, so a user who may not see the program is stopped just the same. Violation message: "This product may not be archived. It is being used for an active promotion program."

**LOY-149** The shipped gift card product and the shipped wallet top-up product may not be deleted, neither as a product variant nor as the product template that carries the variant. Violation message: "You cannot delete <name> as it is used in 'Coupons & Loyalty'. Please archive it instead." The placeholder is the display name of the product with the internal reference omitted from it. The guard is not applied when the capability package that ships the two products is itself being removed, because the two products are removed together with it.

**LOY-150** Archiving a Pricelist is refused while at least one **active** Loyalty Program lists that pricelist among its `pricelist_ids`. Violation message: "This pricelist may not be archived. It is being used for active promotion programs: <program names>" where the placeholder is the names of every offending active program, separated by a comma and a space, in the order the programs are returned by the search. The search is performed with elevated rights.

**LOY-151** The hidden discount product of a reward may not be deleted while that reward exists: the reward's reference to its hidden discount product is a restricted reference and the deletion is refused at the database level. The product has to be archived instead, which is what the cascade of LOY-010 does.

**LOY-152** Archiving a program archives the hidden discount product of each of its rewards but never the reward product of a free product reward: the real product that is given away stays active. The propagation of LOY-010 writes `active` on the rules, then on the rewards, then on the communication rules, then on the hidden discount products, in that order; because the rewards are already archived by the time the hidden discount products are written, LOY-148 finds no active reward and does not refuse the archive.

## 16. Further invariants

These rules complete the catalogue with invariants that govern ordering, derivation and the shape of
the records, and that the sections above assume without stating.

**LOY-153** The default ordering of Loyalty Program is `sequence` ascending, then insertion order.
That order is also the order in which automatic programs are offered to a document, so moving a
program up the list makes it the one whose global discount is compared first.

**LOY-154** The default ordering of Loyalty Reward is `required_points` ascending. When several
rewards of one program are affordable, the cheapest is therefore the one offered first, and it is
the one the automatic claiming of a single-reward program picks.

**LOY-155** The default ordering of Loyalty History is identifier descending, so the newest movement
of a card comes first everywhere it is listed.

**LOY-156** `total_order_count` is derived, never stored. Its base value is zero; the sales channel
adds `order_count` and the counter channel adds `pos_order_count`. A rebuild that stores the count
instead of deriving it must recompute it whenever a reward line is created or deleted on either
channel, or the usage ceiling will drift.

**LOY-157** `loyalty_data` on a sales order is empty for every order that is not confirmed, and for a
confirmed order that has no history movement. It is never written by a user.

**LOY-158** `available_on` on a Loyalty Program is declared but never persisted: it carries a label
in the form and holds no value. A rebuild must not create a column for it.

**LOY-159** A reward line's point cost is carried by exactly one line of a claim, whichever tax
combination that line represents. Summing `points_cost` over the lines of a claim therefore yields
the cost once, and a rebuild that writes the cost on every line would debit the card several times
over.

**LOY-160** A card's `currency_id` and `point_name` are mirrors of its program and can never be set
independently; a rule's and a reward's `company_id` are stored mirrors of the program's company,
written for the sole purpose of letting the company record rule filter on them.

## 17. Index of rule identifiers

| Identifier | Subject |
|---|---|
| LOY-001 | Program name required |
| LOY-002 | Program currency required and derived from the company |
| LOY-003 | Price-list currency must match the program currency |
| LOY-004 | Validity window must be ordered |
| LOY-005 | A program keeps at least one reward |
| LOY-006 | A usage cap must be strictly positive |
| LOY-007 | Changing the program type rewrites the preset |
| LOY-008 | Point label forced to the currency symbol for stored-value programs |
| LOY-009 | Trigger products dropped on creation of a non-stored-value program |
| LOY-010 | Archiving a program cascades to rules, rewards, plans and hidden products |
| LOY-011 | An active program may not be deleted |
| LOY-012 | A program that has issued cards may not be deleted |
| LOY-013 | Un-archiving re-runs the promotional-code uniqueness checks |
| LOY-014 | A printable document needs a message template first |
| LOY-015 | Writing the simplified template rewrites the whole communication plan |
| LOY-016 | Turning off the counter flag empties the till restriction |
| LOY-017 | A program is used only at a till of its own currency |
| LOY-018 | Rule point amount strictly positive |
| LOY-019 | Split per unit forbidden on accumulating and wallet programs |
| LOY-020 | Split per unit only takes effect on future, non-per-order grants |
| LOY-021 | A promotional code is unique among active rules |
| LOY-022 | A rule code and a card code may not collide |
| LOY-023 | Storefront relaxation of the code uniqueness rule |
| LOY-024 | Code and application mode derive from each other |
| LOY-025 | Writing a code regenerates the scannable alternative |
| LOY-026 | A code-mode rule contributes nothing until its code is entered |
| LOY-027 | An empty product filter matches everything except on a gift card program |
| LOY-028 | Required points strictly positive |
| LOY-029 | Free product quantity strictly positive |
| LOY-030 | Discount magnitude strictly positive |
| LOY-031 | A reward product may not be a combination product |
| LOY-032 | Every reward owns one hidden discount product |
| LOY-033 | The description renames and re-translates the hidden product |
| LOY-034 | Archiving a reward archives its hidden product |
| LOY-035 | Deleting a used reward archives it instead |
| LOY-036 | Deleting a reward re-runs the at-least-one-reward check |
| LOY-037 | Definition of a global discount |
| LOY-038 | A card code is globally unique |
| LOY-039 | How a card code is generated |
| LOY-040 | No expiry date on a loyalty card |
| LOY-041 | An expired card is excluded, detached and refused |
| LOY-042 | Card creation runs the "at creation" communication plan |
| LOY-043 | A balance change runs the milestone communication plan |
| LOY-044 | Archiving a card first deletes its draft-order promises |
| LOY-045 | How the recipient of a card communication is resolved |
| LOY-046 | When a balance is rounded by the program currency |
| LOY-047 | A future-program card is not claimable on the document that created it |
| LOY-048 | A promise towards a card of another customer is removed |
| LOY-049 | A card of the anonymous visitor is re-owned when the document names a customer |
| LOY-050 | An unusable current-order card is deleted at confirmation |
| LOY-051 | An unused card created by a cancelled document is deleted |
| LOY-052 | One pending promise per order and card |
| LOY-053 | The program applicability filter |
| LOY-054 | Storefront substitution in the applicability filter |
| LOY-055 | The reference date of a document |
| LOY-056 | The evaluation time zone |
| LOY-057 | A program at its usage cap is excluded and its code refused |
| LOY-058 | How the total document count is composed |
| LOY-059 | A wallet program without trigger products grants nothing |
| LOY-060 | A program with no rule and current applicability matches unconditionally |
| LOY-061 | The three gate refusals of a non-nominative program, in priority order |
| LOY-062 | A nominative program is applicable even at zero points |
| LOY-063 | Which documents allow nominative programs |
| LOY-064 | Automatic discounts do not reduce a minimum purchase; coded ones do |
| LOY-065 | A combination item never counts on its own |
| LOY-066 | Quantities are converted to the reference unit of measure |
| LOY-067 | Reward lines never count towards the quantity gate |
| LOY-068 | Which reward lines are excluded from the amount paid |
| LOY-069 | Shipping lines are threshold-neutral and earn nothing |
| LOY-070 | Points per unit of currency are truncated downward |
| LOY-071 | A program may be attached only once |
| LOY-072 | A program outside the applicability filter is refused |
| LOY-073 | A reward is claimable only at or above its point price |
| LOY-074 | No discount on a zero discountable amount, except under a payment reward |
| LOY-075 | An applied non-payment discount is not claimable again |
| LOY-076 | A free product reward needs a live product |
| LOY-077 | The better global discount wins |
| LOY-078 | Two over-large discounts: the smaller one wins |
| LOY-079 | A future-program reward may not be claimed on its own document |
| LOY-080 | A reward may not be claimed without enough points |
| LOY-081 | Nothing to discount, and the placeholder line |
| LOY-082 | A free product must be among the reward's products |
| LOY-083 | Only one free shipping reward at a time |
| LOY-084 | Payment rewards are recomputed last |
| LOY-085 | A payment reward may not pay for its own top-up |
| LOY-086 | Fixed-amount taxes are discounted only by a payment reward |
| LOY-087 | A discount is capped by the document total and by the maximum amount |
| LOY-088 | Per-point mode truncates the points to whole claims outside a payment program |
| LOY-089 | How the point cost is computed in each mode |
| LOY-090 | One line per tax combination, cost on the first line only |
| LOY-091 | When the tax-name suffix is appended to a reward line description |
| LOY-092 | A manually edited line description survives a recomputation |
| LOY-093 | Confirmation refuses a negative available balance |
| LOY-094 | Confirmation recomputes the document first |
| LOY-095 | Confirmation writes one history movement per card |
| LOY-096 | Confirmation notifies about rewards not added |
| LOY-097 | Confirmation sends the coupons the document earned, immediately |
| LOY-098 | What cancellation of a confirmed order undoes |
| LOY-099 | Duplication carries no promotion |
| LOY-100 | Repricing an order that carries rewards re-runs the recomputation |
| LOY-101 | A reward line may never be invoiced alone |
| LOY-102 | Invoice lines from discount rewards are classified as discount lines |
| LOY-103 | A fully rewarded order is invoiced anyway |
| LOY-104 | Reward line changes on a confirmed order move points at once |
| LOY-105 | Deleting one line of a claim deletes the whole claim |
| LOY-106 | Reward lines are read-only on the portal and not sellable |
| LOY-107 | Which reward line fields are read-only on the order screen |
| LOY-108 | A code is matched against rules first, then against cards |
| LOY-109 | An already applied promotional code is refused |
| LOY-110 | An unmatched or unusable code is refused and flagged "not found" |
| LOY-111 | A card that pays for no reward is refused as already used |
| LOY-112 | Loyalty and wallet programs may never be applied by code |
| LOY-113 | The program row lock that makes the usage cap exact |
| LOY-114 | What a successful code application records |
| LOY-115 | What a refused attach step undoes |
| LOY-116 | The internal user group alone grants nothing |
| LOY-117 | Rights of the Salesperson and the Sales Administrator |
| LOY-118 | Rights of the Point of Sale Cashier and Administrator |
| LOY-119 | No group may delete a card |
| LOY-120 | The company record rule on the five stored entities |
| LOY-121 | Which writes of the recomputation run with elevated rights |
| LOY-122 | Public access to the image of a hidden discount product |
| LOY-123 | Point values carry two decimal places and truncate downward |
| LOY-124 | Reward amounts are rounded per line by the document currency |
| LOY-125 | Conversion of a minimum purchase amount |
| LOY-126 | Conversion of a maximum discount and of a fixed discount value |
| LOY-127 | Both bounds of the validity window are inclusive |
| LOY-128 | The expiry date itself is still usable |
| LOY-129 | Monetary comparisons use the document currency's rounding step |
| LOY-130 | Program, rules, rewards, cards and history share one company |
| LOY-131 | A card's currency and point label follow its program |
| LOY-132 | A reward line carries the taxes it compensates |
| LOY-133 | A free product line carries the product's taxes and a hundred percent discount |
| LOY-134 | The reward grouping code identifies one claim |
| LOY-135 | A counter session needs its reward products available at the counter |
| LOY-136 | The shape a gift card program must have at a counter |
| LOY-137 | Printing gift cards at a counter needs a template and a document |
| LOY-138 | The server revalidates the device's point changes before payment |
| LOY-139 | The six refusals of the counter code redemption service, in order |
| LOY-140 | How the counter chooses the card behind a scanned code |
| LOY-141 | When a cart claims a reward automatically |
| LOY-142 | A reward removed by hand is never claimed automatically again |
| LOY-143 | A coupon link visited without a cart is remembered |
| LOY-144 | The cart is revalidated before a payment is finalised |
| LOY-145 | Several discount lines are merged into one visual cart line |
| LOY-146 | Reward lines do not count in the cart quantity badge |
| LOY-147 | A zero-priced reward line never blocks the checkout |
| LOY-148 | Archive guard on a product used by an active reward |
| LOY-149 | Delete guard on the shipped gift card and top-up products |
| LOY-150 | Archive guard on a price list used by an active program |
| LOY-151 | A hidden discount product may not be deleted while its reward exists |
| LOY-152 | Archiving a program never archives a free product |
| LOY-153 | Program ordering also fixes the order of automatic candidates |
| LOY-154 | Reward ordering offers the cheapest affordable reward first |
| LOY-155 | History ordering puts the newest movement first |
| LOY-156 | The total document count is derived, never stored |
| LOY-157 | The loyalty summary is empty on an unconfirmed order |
| LOY-158 | The channel label carrier holds no value |
| LOY-159 | The point cost of a claim is carried once |
| LOY-160 | Mirrored currency, point label and company may not be set independently |

## 18. Mapping from the identifiers used before this consolidation

Two independently written descriptions of this domain were merged into this folder. One numbered its
rules `LOY-RULE-nnn` with gaps between the groups; the other used a per-topic prefix and cited only
one identifier, `BR-P-03`, from its entity document. Both schemes are replaced by the single
contiguous scheme of this file. The table below maps every former identifier to its new one, so that
a reader holding either earlier text can find the rule again.

| Former identifier (first version) | Former identifier (second version) | New identifier |
|---|---|---|
| LOY-RULE-001 | — | LOY-001 |
| LOY-RULE-002 | — | LOY-002 |
| LOY-RULE-003 | — | LOY-003 |
| LOY-RULE-004 | — | LOY-004 |
| LOY-RULE-005 | BR-P-03 | LOY-005 |
| LOY-RULE-006 | — | LOY-006 |
| LOY-RULE-007 | — | LOY-007 |
| LOY-RULE-008 | — | LOY-008 |
| LOY-RULE-009 | — | LOY-009 |
| LOY-RULE-010 | — | LOY-010 |
| LOY-RULE-011 | — | LOY-011 |
| LOY-RULE-012 | — | LOY-012 |
| LOY-RULE-013 | — | LOY-013 |
| LOY-RULE-014 | — | LOY-014 |
| LOY-RULE-015 | — | LOY-015 |
| LOY-RULE-016 | — | LOY-016 |
| LOY-RULE-017 | — | LOY-017 |
| LOY-RULE-021 | — | LOY-018 |
| LOY-RULE-022 | — | LOY-019 |
| LOY-RULE-023 | — | LOY-020 |
| LOY-RULE-024 | — | LOY-021 |
| LOY-RULE-025 | — | LOY-022 |
| LOY-RULE-026 | — | LOY-023 |
| LOY-RULE-027 | — | LOY-024 |
| LOY-RULE-028 | — | LOY-025 |
| LOY-RULE-029 | — | LOY-026 |
| LOY-RULE-030 | — | LOY-027 |
| LOY-RULE-031 | — | LOY-028 |
| LOY-RULE-032 | — | LOY-029 |
| LOY-RULE-033 | — | LOY-030 |
| LOY-RULE-034 | — | LOY-031 |
| LOY-RULE-035 | — | LOY-032 |
| LOY-RULE-036 | — | LOY-033 |
| LOY-RULE-037 | — | LOY-034 |
| LOY-RULE-038 | — | LOY-035 |
| LOY-RULE-039 | — | LOY-036 |
| LOY-RULE-040 | — | LOY-037 |
| LOY-RULE-041 | — | LOY-038 |
| LOY-RULE-042 | — | LOY-039 |
| LOY-RULE-043 | — | LOY-040 |
| LOY-RULE-044 | — | LOY-041 |
| LOY-RULE-045 | — | LOY-042 |
| LOY-RULE-046 | — | LOY-043 |
| LOY-RULE-047 | — | LOY-044 |
| LOY-RULE-048 | — | LOY-045 |
| LOY-RULE-049 | — | LOY-046 |
| LOY-RULE-050 | — | LOY-047 |
| LOY-RULE-051 | — | LOY-048 |
| LOY-RULE-052 | — | LOY-049 |
| LOY-RULE-053 | — | LOY-050 |
| LOY-RULE-054 | — | LOY-051 |
| LOY-RULE-055 | — | LOY-052 |
| LOY-RULE-056 | — | LOY-053 |
| LOY-RULE-057 | — | LOY-054 |
| LOY-RULE-058 | — | LOY-055 |
| LOY-RULE-059 | — | LOY-056 |
| LOY-RULE-060 | — | LOY-057 |
| LOY-RULE-061 | — | LOY-058 |
| LOY-RULE-062 | — | LOY-059 |
| LOY-RULE-063 | — | LOY-060 |
| LOY-RULE-064 | — | LOY-061 |
| LOY-RULE-065 | — | LOY-062 |
| LOY-RULE-066 | — | LOY-063 |
| LOY-RULE-067 | — | LOY-064 |
| LOY-RULE-068 | — | LOY-065 |
| LOY-RULE-069 | — | LOY-066 |
| LOY-RULE-070 | — | LOY-067 |
| LOY-RULE-071 | — | LOY-068 |
| LOY-RULE-072 | — | LOY-069 |
| LOY-RULE-073 | — | LOY-070 |
| LOY-RULE-074 | — | LOY-071 |
| LOY-RULE-075 | — | LOY-072 |
| LOY-RULE-076 | — | LOY-073 |
| LOY-RULE-077 | — | LOY-074 |
| LOY-RULE-078 | — | LOY-075 |
| LOY-RULE-079 | — | LOY-076 |
| LOY-RULE-080 | — | LOY-077 |
| LOY-RULE-081 | — | LOY-078 |
| LOY-RULE-082 | — | LOY-079 |
| LOY-RULE-083 | — | LOY-080 |
| LOY-RULE-084 | — | LOY-081 |
| LOY-RULE-085 | — | LOY-082 |
| LOY-RULE-086 | — | LOY-083 |
| LOY-RULE-087 | — | LOY-084 |
| LOY-RULE-088 | — | LOY-085 |
| LOY-RULE-089 | — | LOY-086 |
| LOY-RULE-090 | — | LOY-087 |
| LOY-RULE-091 | — | LOY-088 |
| LOY-RULE-092 | — | LOY-089 |
| LOY-RULE-093 | — | LOY-090 |
| LOY-RULE-094 | — | LOY-091 |
| LOY-RULE-095 | — | LOY-092 |
| LOY-RULE-096 | — | LOY-093 |
| LOY-RULE-097 | — | LOY-094 |
| LOY-RULE-098 | — | LOY-095 |
| LOY-RULE-099 | — | LOY-096 |
| LOY-RULE-100 | — | LOY-097 |
| LOY-RULE-101 | — | LOY-098 |
| LOY-RULE-102 | — | LOY-099 |
| LOY-RULE-103 | — | LOY-100 |
| LOY-RULE-104 | — | LOY-101 |
| LOY-RULE-105 | — | LOY-102 |
| LOY-RULE-106 | — | LOY-103 |
| LOY-RULE-107 | — | LOY-104 |
| LOY-RULE-108 | — | LOY-105 |
| LOY-RULE-109 | — | LOY-106 |
| LOY-RULE-110 | — | LOY-107 |
| LOY-RULE-111 | — | LOY-108 |
| LOY-RULE-112 | — | LOY-109 |
| LOY-RULE-113 | — | LOY-110 |
| LOY-RULE-114 | — | LOY-111 |
| LOY-RULE-115 | — | LOY-112 |
| LOY-RULE-116 | — | LOY-113 |
| LOY-RULE-117 | — | LOY-114 |
| LOY-RULE-118 | — | LOY-115 |
| LOY-RULE-131 | — | LOY-116 |
| LOY-RULE-132 | — | LOY-117 |
| LOY-RULE-133 | — | LOY-118 |
| LOY-RULE-134 | — | LOY-119 |
| LOY-RULE-135 | — | LOY-120 |
| LOY-RULE-136 | — | LOY-121 |
| LOY-RULE-137 | — | LOY-122 |
| LOY-RULE-141 | — | LOY-123 |
| LOY-RULE-142 | — | LOY-124 |
| LOY-RULE-143 | — | LOY-125 |
| LOY-RULE-144 | — | LOY-126 |
| LOY-RULE-145 | — | LOY-127 |
| LOY-RULE-146 | — | LOY-128 |
| LOY-RULE-147 | — | LOY-129 |
| LOY-RULE-151 | — | LOY-130 |
| LOY-RULE-152 | — | LOY-131 |
| LOY-RULE-153 | — | LOY-132 |
| LOY-RULE-154 | — | LOY-133 |
| LOY-RULE-155 | — | LOY-134 |
| LOY-RULE-161 | — | LOY-135 |
| LOY-RULE-162 | — | LOY-136 |
| LOY-RULE-163 | — | LOY-137 |
| LOY-RULE-164 | — | LOY-138 |
| LOY-RULE-165 | — | LOY-139 |
| LOY-RULE-166 | — | LOY-140 |
| LOY-RULE-171 | — | LOY-141 |
| LOY-RULE-172 | — | LOY-142 |
| LOY-RULE-173 | — | LOY-143 |
| LOY-RULE-174 | — | LOY-144 |
| LOY-RULE-175 | — | LOY-145 |
| LOY-RULE-176 | — | LOY-146 |
| LOY-RULE-177 | — | LOY-147 |
| LOY-RULE-181 | — | LOY-148 |
| LOY-RULE-182 | — | LOY-149 |
| LOY-RULE-183 | — | LOY-150 |
| LOY-RULE-184 | — | LOY-151 |
| LOY-RULE-185 | — | LOY-152 |
| — (new in this consolidation) | — | LOY-153 to LOY-160 |

## 19. Reconciliation notes

1. **Rule numbering.** The gapped scheme `LOY-RULE-001` … `LOY-RULE-185` of the first version and the
   single `BR-P-03` citation of the second were renumbered into the contiguous scheme LOY-001 …
   LOY-160 of this file. Section 18 gives the full mapping; every citation in the other files of the
   folder was rewritten at the same time.
2. **Card code length (LOY-039).** One version stated that the generated code is thirteen characters
   long. The slice taken from the universally unique identifier is eleven characters, so the code is
   **fourteen** characters long and begins with `044`. The corrected length is used here, in
   [entities.md](entities.md) and in [acceptance-criteria.md](acceptance-criteria.md).
3. **System parameter keys (LOY-056, LOY-057, LOY-103, LOY-143).** One version invented descriptive
   keys. The reproduced keys `loyalty.timezone`,
   `loyalty.compute_all_discount_product_ids`,
   `website_sale_coupon.abandonned_coupon_validity`, `sale.automatic_invoice` and
   `sale.default_invoice_email_template` are used throughout, including the spelling of the third
   one, which is part of the stored key.
4. **Access group identifiers (LOY-116 to LOY-118).** One version named the groups by invented
   identifiers. The reproduced identifiers `base.group_user`, `sales_team.group_sale_salesman`,
   `sales_team.group_sale_manager`, `point_of_sale.group_pos_user` and
   `point_of_sale.group_pos_manager` are used, each with its role named in words.
5. **Field identifiers.** Every rule that names a field now names the reproduced storage name
   (`max_usage`, `discount`, `minimum_qty`, `pos_ok`, `sale_ok`, `ecommerce_ok`,
   `pos_config_ids`, `discount_max_amount`, `reward_product_qty` and the rest) rather than the
   full-word form one version had invented for it. The full names are carried in the field tables of
   [entities.md](entities.md).
6. **Rules added during the merge (LOY-153 to LOY-160).** The second version stated a number of
   invariants — the two default orderings that decide which program and which reward is considered
   first, the derivation of the usage count, the emptiness of the loyalty summary on a quotation, the
   label carrier that holds no value, the single point cost of a claim and the mirrored fields — that
   the first version's catalogue did not number. They are numbered here so that nothing is lost.
