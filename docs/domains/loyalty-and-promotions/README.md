# Loyalty and Promotions

This domain specifies everything the system does with *promotional programmes*: the configuration
objects that describe an offer, the stored-value instruments that carry a balance, the algorithm
that decides which offers apply to a customer's order, the lines that the offers add to that order,
the way the whole set is recomputed each time the order changes, and the way conflicts between
competing offers are resolved.

One single configuration entity — the Loyalty Program (`loyalty.program`, table `loyalty_program`) —
covers eight commercially very different products: automatic promotions, promotional codes, printed
or mailed coupons, buy-more-get-more offers, next-order coupons, loyalty cards, gift cards and
electronic wallets. They differ only in the values of a handful of fields. Understanding that single
shape, and the way those fields interact, is the whole of the domain.

The domain owns no accounting document of its own. Discounts reach the ledger as negative revenue
lines on the sales document that carries them; gift cards and electronic wallets reach the ledger as
a sale of a service product when they are bought and as a negative revenue line when they are spent.
[accounting-effects.md](accounting-effects.md) gives every journal item.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Programme definition | One programme record with a programme type, a validity window, a currency, an optional price-list restriction, a usage limit, a company, and a portal visibility flag. |
| Programme types | Automatic promotion, discount code, coupon, buy X get Y, next-order coupon, loyalty card, gift card, electronic wallet — each a preset of the same fields, applied whenever the type changes. |
| Conditional rules | Zero or more rules per programme, each with a product filter (explicit products, a category, a tag, an extra filter expression), a minimum quantity, a minimum purchase amount evaluated tax-included or tax-excluded, an automatic or code-based trigger with a unique code, and a point-earning formula in one of three modes. |
| Point earning | Points per order, points per unit of currency spent, points per unit purchased; optional splitting of the earned points into one card per matched unit; rounding of earned points downward to two decimals. |
| Rewards | Free product (a chosen product or any product carrying a tag, in a chosen quantity) and discount (percentage, fixed amount per order, or fixed amount per point), each applying to the whole order, to the cheapest line, or to a filtered set of products, with an optional maximum discount amount and an optional wallet-emptying flag. |
| Free shipping | A discount reward whose product filter is the delivery products, which zeroes the delivery line and is re-evaluated whenever the carrier or the destination changes. |
| Loyalty cards | The instrument carrying a point balance, a unique bar-code-compatible code, an optional owner, an optional expiry date, a tracked balance, and a full movement history. |
| Stored value | Gift cards and electronic wallets: buying the top-up product credits the card with its paid amount; spending debits it; the spend is a payment-like reward always applied last. |
| The application algorithm | The five-step recomputation that decides which programmes are applicable, updates their point entries, rebuilds every reward line, applies newly applicable automatic programmes, and deletes what is left over. |
| Code entry | Entering a text code on an order: matching it against rule codes then against card codes, the seven distinct refusal messages, the row lock that serialises concurrent entry of the same code. |
| Conflict resolution | The "global discount" comparison that keeps only the single most advantageous order-wide discount, the ban on applying the same discount reward twice, the ban on discounting an order whose total is already zero, the ordering that always puts stored-value payments last. |
| Recomputation triggers | Order line change, price-list reprice, carrier change, customer change, code entry, reward claim, reward removal, confirmation, cancellation. |
| Card generation | The wizard that mints cards in bulk, anonymously or for a selected audience, with a granted balance, an expiry and a history entry. |
| Balance adjustment | The wizard that sets a card's balance to a new value and records the difference as issued or used. |
| Communications | Per-programme communication plans that send a message when a card is created and when a balance crosses a milestone. |
| Customer portal | The customer's card list, the card dialogue with balance, code, expiry, the last five movements and the three best affordable rewards, and the paged movement history. |
| Counter channel | The same programmes applied inside the selling application at the counter, with client-side point computation, code scanning and reward buttons. |
| Storefront channel | The same programmes applied to a web cart, with a pending-code mechanism for anonymous visitors, a claimable-reward list on the cart page, and an auto-apply setting. |
| Merging customers | Nominative cards of merged customers are consolidated onto one card whose balance is the sum. |

---

## 2. Entities of the domain

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Loyalty Program | `loyalty.program` | `loyalty_program` | One promotional offer: its type, validity, currency, scope and limits. |
| Loyalty Rule | `loyalty.rule` | `loyalty_rule` | One condition of a programme: what must be bought, how much, and how many points that earns. |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | One thing a customer may claim: a free product or a discount, with its price formula and its point price. |
| Loyalty Card | `loyalty.card` | `loyalty_card` | One instrument carrying a point balance and a unique code: a coupon, a loyalty card, a gift card or an electronic wallet. |
| Loyalty History | `loyalty.history` | `loyalty_history` | One movement on a card: points issued, points used, the document that caused it. |
| Loyalty Communication | `loyalty.mail` | `loyalty_mail` | One rule saying which message template to send on card creation or on reaching a point milestone. |
| Sales Order Coupon Points | `sale.order.coupon.points` | `sale_order_coupon_points` | The pending promise that a given order will credit a given card with a given number of points when it is confirmed. |

Transient (wizard) entities of the domain:

| Entity | Transport name | Purpose |
|---|---|---|
| Card Generation Wizard | `loyalty.generate.wizard` | Mints cards in bulk for anonymous holders or for a selected audience. |
| Card Balance Wizard | `loyalty.card.update.balance` | Sets a card's balance to a new value and records the difference. |
| Coupon Entry Wizard | `sale.loyalty.coupon.wizard` | Takes a code typed by a salesperson and applies it to a sales order. |
| Reward Selection Wizard | `sale.loyalty.reward.wizard` | Lets a salesperson pick one claimable reward (and, if several are possible, one free product) and adds it to the order. |
| Coupon Sharing Wizard | `coupon.share` | Produces a shareable web address that pre-loads a code into a visitor's cart. |

Entities owned by other domains that this domain extends, and whose extensions are specified here:

| Entity | Extension |
|---|---|
| Sales Order (`sale.order`) | Applied cards, code-enabled rules, pending point entries, reward total, gift-card count, portal loyalty data, and the whole application algorithm. |
| Sales Order Line (`sale.order.line`) | Reward marker, reward reference, card reference, reward grouping code, point cost. |
| Point of Sale Order (`pos.order`) | Applied cards, point changes per card, and the counter-side equivalents of the same algorithm. |
| Point of Sale Order Line (`pos.order.line`) | Reward reference, card reference, point cost, reward grouping code. |
| Product Variant and Product Template (`product.product`, `product.template`) | Archive and delete protection for products used by active programmes; the default picture of a gift-card product. |
| Price List (`product.pricelist`) | Archive protection when an active programme restricts itself to that price list. |
| Customer (`res.partner`) | The count of the customer's active cards and the action that lists them. |
| Customer Merge Wizard (`base.partner.merge.automatic.wizard`) | Consolidation of nominative cards. |
| Journal Item (`account.move.line`) | Classification of a reward line as a discount line for reporting. |

---

## 3. Reading order

1. **[glossary.md](glossary.md)** — the vocabulary. The words *programme*, *rule*, *reward*, *card*,
   *point*, *trigger*, *nominative*, *payment programme*, *global discount* and *discountable* all
   have precise meanings that the rest of the domain depends on. Read this first.
2. **[entities.md](entities.md)** — the seven persistent entities, field by field, with every
   default, every computed rule and every uniqueness constraint.
3. **[state-machines.md](state-machines.md)** — programmes and cards have no status column; their
   behaviour is governed by derived states (validity window, nominativeness, exhaustion, expiry) and
   by the state of the order that carries them. Every such derived state is tabulated here.
4. **[calculations.md](calculations.md)** — the heart of the domain: point earning, the discountable
   base in its four variants, reward pricing, the per-tax split, the maximum-discount ceiling, the
   point cost, and the ordering comparisons. Every formula has a worked example.
5. **[workflows.md](workflows.md)** — the five-step application algorithm in full, then every
   operational flow around it: entering a code, claiming a reward, removing a reward, confirming,
   cancelling, invoicing, buying and spending a gift card, the counter flow, the storefront flow.
6. **[business-rules.md](business-rules.md)** — every validation, every constraint, every error
   message with its exact text, every permission check, the row lock, the concurrency behaviour.
7. **[accounting-effects.md](accounting-effects.md)** — the journal items for a discount, for the
   sale of a gift card, for the redemption of a gift card, for an electronic wallet top-up and spend,
   and for a free product.
8. **[configuration.md](configuration.md)** — settings, parameters, default records, groups, access
   rights, record rules.
9. **[interfaces.md](interfaces.md)** — menus, views, remote operations, web addresses, printable
   documents, message templates.
10. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered scenarios with concrete numbers,
    including all the mandatory ones (a tenth off two hundred, a discount on the cheapest line, a
    free product, free shipping, a gift card of fifty partly spent, points at one per unit of
    currency redeemed at a hundred, two programmes competing, and a code that does not apply).

---

## 4. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Sales](../sales/README.md) | The Sales Order and Sales Order Line entities, their pricing (unit price, discount percentage, subtotal, total), their tax computation, their state machine, their invoicing. Reward lines **are** sales order lines; everything the sales domain says about a line's price, tax and invoicing applies to them unchanged unless this domain states an exception. |
| [Point of Sale](../point-of-sale/README.md) | The counter order, its lines, its client-side price computation and its session posting. The counter is a second channel for exactly the same programmes. |
| [Products and Catalog](../products-and-catalog/README.md) | Product variants, product categories, product tags and the service product type used for reward lines and top-up products. |
| [Taxes](../taxes/README.md) | The tax computation used to build the discountable base and to split a discount per tax group; the fiscal position mapping applied to reward lines. |
| [Pricing and Pricelists](../pricing-and-pricelists/README.md) | The price list a programme may be restricted to, and the reprice action that triggers a recomputation. |
| [Multi-Currency](../multi-currency/README.md) | The conversion of a programme's monetary amounts (minimum purchase, fixed discount, maximum discount) into the order's currency, and the rounding of a monetary point balance. |
| [Delivery and Shipping](../delivery-and-shipping/README.md) | The delivery product carried on a delivery line, which the free-shipping reward filters on. |
| [Contacts and Organizations](../contacts-and-organizations/README.md) | The customer who owns a nominative card, and the merge wizard. |
| [Messaging and Activities](../messaging-and-activities/README.md) | The message thread on a card and the template sending used by communication plans. |
| [Identity and Access](../identity-and-access/README.md) | The two selling groups that gate every access right of the domain. |
| [General Ledger](../general-ledger/README.md) and [Accounts Receivable](../accounts-receivable/README.md) | The journal entries that carry a discount or a stored-value movement to the books. |

---

## 5. What this domain deliberately does not do

- It never creates a journal entry itself. Its output is always a line on a document owned by
  another domain.
- It never changes the price of a non-reward line. A discount is always a separate negative line,
  never a reduction of an existing line's unit price or discount percentage. The only exception is
  the free-product reward, which is a normal product line carrying a hundred percent line discount.
- It never partially grants a reward. If the card has ninety points and the reward costs a hundred,
  nothing is granted; there is no pro-rated reward. The single exception is a stored-value payment,
  which is by construction proportional to the amount spent.
- It never lets a discount take an order's total below zero. Every discount is capped by the
  remaining discountable amount and by the order total.
