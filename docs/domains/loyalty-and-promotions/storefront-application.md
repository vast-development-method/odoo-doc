# The storefront application

In the online shop the evaluation runs on the server, on the visitor's cart, and is re-run at every event that can change the cart. What distinguishes the storefront from the back office is that the shopper is not asked to choose: unambiguous rewards are claimed automatically, ambiguous ones are offered as buttons in the cart, and a shopper who removes a reward by hand is remembered so that the automatic claiming does not put it back.

## 1. When the cart is re-evaluated

| Event | What runs |
|---|---|
| The cart page is opened | The full evaluation, then the automatic claiming. |
| A product is added, its quantity is changed, or it is removed | The ordinary cart verification, then the full evaluation, then the automatic claiming, then the cart quantity stored in the session is refreshed because the claiming may have added or removed lines. |
| The cart is recomputed for any other reason | The full evaluation, then the automatic claiming, then the ordinary recomputation. |
| A shipping method is chosen or removed | The full evaluation, so that a free shipping reward follows the new shipping line. |
| A promotional code is submitted | The code application, then possibly a reward claim. |
| A reward is claimed from the cart | The reward application, then the full evaluation, then, when the chosen shipping method is free above a threshold and the reward is not a payment reward, the shipping price is recomputed and the shipping line rewritten or removed. |
| A payment is about to be finalized | The full evaluation and a total comparison. |
| The scheduled cleanup runs | Applied cards are detached from abandoned carts and each is re-evaluated. |

Before every evaluation, the cart first tries to apply a coupon code that a link stored in the session (section 3).

## 2. Automatic claiming

A claimable reward is applied without asking when **all** of the following hold:

1. its program is not nominative;
2. its program has exactly one reward;
3. it is not a free product reward that offers several products;
4. it is not among the cart's manually removed rewards;
5. it is not already applied on the cart.

Every reward that fails one of these tests stays visible in the cart as a button the shopper may press.

A reward whose application raises "there is nothing to discount" is silently skipped.

**Manually removed rewards**: deleting a reward line from the cart records that reward in the cart's list of manually removed rewards. The list is only fed by a deletion performed through the cart, not by a deletion performed in the back office. There is no operation that clears the list; claiming the reward again through a button removes the block for that claim only.

## 3. Coupon links

A shareable link has the shape `/coupon/<code>?r=<landing page>`.

1. The code is trimmed and stored in the visitor's session as the pending coupon code.
2. Any `coupon_error` and `coupon_error_type` values already present in the landing page's query string are discarded, so that only messages produced by the system are trusted.
3. When the visitor has a cart, the pending code is applied immediately:
   - on refusal, the landing page receives the refusal text under the key `coupon_error`;
   - on success, the pending code is removed from the session and the landing page receives the code under the key `notify_coupon`; when exactly one card offers exactly one reward and that reward is not a multi-product free product reward, that reward is claimed at once.
4. When the visitor has no cart, the landing page receives `The coupon will be automatically applied when you add something in your cart.` under the key `coupon_error`, with `coupon_error_type` set to `warning`, and the code stays in the session until a cart exists.
5. The visitor is redirected to the landing page with the modified query string.

The site layout renders the two keys: a failure appears as a message beginning with `Could not apply the promo code: ` followed by the code, and a success appears as `The following promo code was applied on your order: ` followed by the code.

## 4. The promotional code form

The cart page carries a form that submits a code. The same form is used for pricelist codes and for loyalty codes.

1. The code is applied to the cart.
2. When the refusal is flagged "not found", the text is handed to the ordinary pricelist code handling, which may still recognize it.
3. When the refusal is not flagged "not found", the refusal text is stored in the session under the promotional code error and displayed once on the next page render.
4. On success:
   - when exactly one card is returned and it offers exactly one reward, that reward is the candidate; when it offers several, the candidate is the one whose identifier the form carried, if any;
   - a candidate that is not a multi-product free product reward, or a multi-product one for which a product was chosen, is applied. A refusal during the application is stored as the promotional code error and the success message is not shown.
   - the code is stored in the session as the successful code and displayed once as `You have successfully applied the following code: <code>`.
5. The shopper is redirected to the page named by the form, defaulting to the cart.

A refusal produced outside this flow, for instance by a stale link, is rendered as `Invalid or expired promo code.`

## 5. Claiming a reward from the cart

The cart lists, for each card, the rewards it can pay for, and a button labelled `Use` for a reward of a program that needs a code or of an automatic program that applies to future orders, and `Claim` otherwise. Pressing the button submits the reward identifier, the card code and the page to return to.

1. Without a cart, the shopper is redirected to the return page.
2. An unparsable or unknown reward identifier redirects to the return page.
3. For a multi-product reward, the chosen product identifier submitted with the form is remembered for the application.
4. The claimable and showable rewards are recomputed and the card offering the reward is looked up.
5. When the submitted card code equals that card's code **and** the program either needs a code and is not a pure discount-code program, or is automatic, applies to future orders and is neither a loyalty nor an electronic wallet program, the request is turned into a code submission instead, so that the card is properly attached before the reward is claimed.
6. Otherwise the reward is applied directly, and the shopper is redirected to the return page.

### 5.1 Claimable and showable rewards

The cart shows more than the strictly claimable rewards: it also shows the rewards of the cards the shopper already owns, so that a loyalty balance or a gift card appears even before it has been attached.

To the claimable rewards computed by the ordinary algorithm, the storefront adds, for every card owned by the cart's customer whose program matches the program filter and whose `trigger` is `with_code`, or whose `trigger` is `auto` with `applies_on` equal to `future`, every reward of that program that is not already applied, unless:

- it is a global discount and the applied global discount is at least as good;
- it is a discount and the cart total including tax is zero;
- the card is expired;
- the points available are below the reward's required points.

Each entry renders the reward description, the formatted balance for a loyalty or wallet program, the masked card code (the last four characters right-aligned in a fourteen-character field padded with a star character) for a bearer card, and the expiration date when there is one. A multi-product reward renders a product chooser.

## 6. How the cart displays rewards

1. **Merged discount lines**: several discount lines produced by one reward application (one per tax group) are hidden and replaced by a single temporary line that is not stored. Its price, its tax-excluded subtotal and its tax-included total are the sums of the underlying lines, it carries no tax, its quantity is one and its description is the shortened description of the first underlying line. Only the visual line is shown because the cart does not show taxes per line.
2. **Hidden lines**: a discount reward line is never shown in the cart line list; a free product reward line is shown.
3. **Quantity badge**: the quantities of the reward lines are subtracted from the cart quantity, so a free product does not inflate the badge.
4. **Reorder**: no reward line may be reordered from a past order.
5. **Strikethrough price**: a reward line never shows a struck-through reference price.
6. **Sellability**: only a free product reward line counts as a sellable line; discount, payment and shipping reward lines do not.
7. **Zero-priced lines**: reward lines are excluded from the rule that refuses a checkout containing a zero-priced line, so a free gift whose product has no sale price does not block the order.
8. **Gift card codes**: a line paid with a bearer card shows the masked code and the expiration date under the product name; the same masking is used on the portal page of the order.
9. **Purchased gift cards**: once the order is confirmed, the cart confirmation page and the portal page list the codes of the gift cards the order produced, numbered from one upward in the order the cards were created, so that the first card is labelled `Gift #1`, the second `Gift #2` and the tenth `Gift #10`, each with its formatted value and a copy button.

## 7. The checkout summary

The order summary shown at checkout and in the express checkout carries three extra amounts:

1. **Shipping discount**: the sum of the tax-included totals of the free shipping reward lines, rendered as a monetary amount and also as a minor-unit integer for the express checkout.
2. **Discount amounts**: one amount per discount reward, computed as the sum of the tax-excluded subtotals of the lines of that reward, for the rewards that do not belong to a gift card or electronic wallet program; then one amount per line for the gift card and wallet payment lines, which are listed individually.
3. The express checkout address step also returns the shipping discount so that the wallet or express payment sheet shows the right total. The amount is the sum of the tax-included totals of the free shipping reward lines, negative, and is handed over both as a monetary amount and as a minor-unit integer.
4. **The amount offered to the express payment sheet excludes every delivery-related line and every free shipping reward line.** The set of lines that make up that amount is the cart's lines minus the shipping lines minus the free shipping reward lines. The shipping method, and therefore its discount, is chosen inside the express payment sheet, so neither may be part of the amount presented before that choice is made.

The amount used to decide whether shipping is free above a threshold excludes the payment reward lines (section 11.3 of [calculations.md](calculations.md)): paying part of an order with a gift card must not cost the shopper the free shipping they had earned. Whenever a reward is claimed and the chosen shipping method is free above a threshold, the shipping price is recomputed and the shipping line is rewritten, or removed when the shipping method can no longer be rated.

## 8. Revalidation at payment time

Before a payment transaction is tied to the order:

1. Remember the order total including tax.
2. Re-evaluate the order.
3. When the total including tax changed by more than the currency's rounding step, refuse the payment with `Cannot process payment: applied reward was changed or has expired.` followed by a new line and `Please refresh the page and try again.`

This closes the window in which a promotion expires, a program is archived or a card is emptied between the moment the shopper saw the total and the moment they paid.

## 9. Topping up an electronic wallet from the shop

A dedicated request adds a wallet top-up product to the cart and redirects to the cart page. It takes the trigger product identifier, adds one unit of it, and requires an identified visitor. This is the storefront counterpart of the counter's wallet button.

A warning is shown on the program screen when an electronic wallet program has a trigger product that is not published on the website, because a shopper could then never reach the top-up.

## 10. The portal card dialog

The portal dialog that shows a card's balance and history is extended in the storefront with the **published trigger products** of the card's program: for each trigger product that is published on the website, its identifier and its formatted list price in the company currency. This is what turns the wallet balance page into a page from which the customer can top up.

## 11. Website scoping

1. A program that names a website is only applicable to carts of that website; a program that names none is applicable to every website.
2. The channel flag used by the program filter is the online shop flag, not the sales flag, as soon as the order belongs to a website.
3. Two programs may carry the same promotional code as long as they are not both reachable from the same website (rule `LOY-023` in [business-rules.md](business-rules.md)). Unarchiving a program whose code collides on the same website fails; unarchiving one whose code only collides on another website succeeds.
4. The evaluation time zone of a cart is the website salesperson's time zone when that user has one.

## 12. Nominative programs and anonymous visitors

1. A cart belonging to the anonymous public visitor does **not** allow nominative programs: the loyalty and electronic wallet cards of a customer are not loaded onto it, and a nominative program evaluated against it reports `This program is not available for public users.`
2. A card created while the cart belonged to the public visitor has the public contact as its owner. As soon as the cart names a real customer, the evaluation rewrites the owner of every such card to that customer. This is what lets an anonymous shopper accumulate a next-order coupon and keep it after signing in at checkout.

## 13. Cleaning up abandoned carts

A scheduled cleanup detaches manually applied cards from draft carts that belong to a website, carry at least one applied card, and have not been written to for longer than the abandonment delay. The delay is four days by default and is set by the system parameter `website_sale_coupon.abandonned_coupon_validity`. Each affected cart is re-evaluated afterwards, which removes the reward lines the detached cards paid for and releases the cards for another shopper.

## 14. Cart line lookup

When a product is added to the cart, the search for an existing line to increment **ignores reward lines**. A free product given by a reward and the same product bought by the shopper therefore live on two separate lines, and adding the product never silently consumes the free one.

When a reward line's quantity is set to zero or less through the cart, the deletion is flagged so that the reward is recorded among the manually removed rewards.
