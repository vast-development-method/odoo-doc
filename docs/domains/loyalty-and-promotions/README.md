# Loyalty and Promotions

This domain specifies everything the system does with *promotional programs*: the configuration
objects that describe an offer, the stored-value instruments that carry a balance, the algorithm
that decides which offers apply to a customer's document, the lines that the offers add to that
document, the way the whole set is recomputed each time the document changes, and the way conflicts
between competing offers are resolved.

One single configuration entity — the Loyalty Program (`loyalty.program`, table `loyalty_program`) —
covers eight commercially very different products: automatic promotions, promotional codes, printed
or mailed coupons, buy-more-get-more offers, next-order coupons, loyalty cards, gift cards and
electronic wallets. They differ only in the values of a handful of fields. Understanding that single
shape, and the way those fields interact, is the whole of the domain.

A company wants to influence buying behavior: *spend fifty and get ten percent off*, *buy three,
get one free*, *here is a code worth fifteen percent on your next order*, *collect one point per
unit of currency and trade two hundred points for five off*, *here is a gift card of fifty that
anyone can spend*, *here is your electronic wallet balance, use it to pay*. All of these are
expressed with four building blocks: a **Loyalty Program** that sets the type, the validity window,
the usage limits, the channels, the currency and the customer restrictions; one or more **Loyalty
Rules** that state the conditions to meet and how many points are granted when they are met; one or
more **Loyalty Rewards** that state what the customer receives and how many points that costs; and
**Loyalty Cards** that hold a unique code, a point balance and, when the program is nominative,
the customer that owns it.

A document — a sales order, a counter ticket, a storefront cart — is continuously re-evaluated
against the applicable programs. The evaluation computes the points each program would grant,
decides which rewards the accumulated points make claimable, and writes reward lines onto the
document. A reward line is a normal document line carrying a negative amount (a discount) or a
normal product line at a zero net price (a free product). Points move onto the cards only when the
document is confirmed; cancelling the document gives them back.

The domain is deliberately channel-neutral: the same program definition drives the sales
application, the counter application (which evaluates everything on the cashier's device so that a
counter keeps selling without a network connection) and the storefront (which claims unambiguous
rewards by itself and offers the rest as buttons in the cart).

The domain owns no accounting document of its own. Discounts reach the ledger as negative revenue
lines on the sales document that carries them; gift cards and electronic wallets reach the ledger as
the sale of a service product when they are bought and as a negative revenue line when they are
spent. [accounting-effects.md](accounting-effects.md) gives every journal item.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Program definition | One program record with a program type, a validity window, a currency, an optional price-list restriction, a usage limit, a company, per-channel availability flags and a portal visibility flag. |
| Program types | Automatic promotion, discount code, coupon, buy more get more, next-order coupon, loyalty card, gift card, electronic wallet — each a preset of the same fields, applied whenever the type changes. |
| Program templates | Nine named templates in two galleries, one per menu, each producing a complete working program. |
| Validity | Start date and end date, both inclusive, evaluated in a configurable time zone and, for a document being paid online, at the moment of the earliest confirmed payment transaction. |
| Usage limit | A maximum number of documents that may use the program, counted across the sales, counter and storefront channels together, enforced under a row lock so that the last use cannot be consumed twice. |
| Conditional rules | Zero or more rules per program, each with a product filter (explicit products, a category and its descendants, a tag, an extra filter expression), a minimum quantity, a minimum purchase amount evaluated tax-included or tax-excluded, an automatic or code-based trigger with a unique code and a scannable alternative to it, and a point-earning formula in one of three modes. |
| Point earning | Points per document, points per unit of currency spent, points per unit purchased; optional splitting of the earned points into one card per matched unit; truncation of earned points downward to two decimals. |
| Rewards | Free product (a chosen product or any product carrying a tag, in a chosen quantity) and discount (percentage, fixed amount per document, or fixed amount per point), each applying to the whole document, to the cheapest line, or to a filtered set of products, with an optional maximum discount amount and an optional whole-balance flag. |
| Tax-exact discounts | A discount is distributed across the tax combinations present on the document, one reward line per combination, so that every tax amount stays exact. |
| Free shipping | A reward type that zeroes the delivery line, capped by an optional maximum, re-evaluated whenever the carrier or the destination changes. |
| Loyalty cards | The instrument carrying a point balance, a unique barcode-compatible code, an optional owner, an optional expiry date, a tracked balance, a use count and a full movement history. |
| Stored value | Gift cards and electronic wallets: buying the top-up product credits the card with its paid amount; spending debits it; the spend is a payment-like reward always applied last. |
| The application algorithm | The five-step recomputation that decides which programs are applicable, updates their point entries, rebuilds every reward line, applies newly applicable automatic programs, and deletes what is left over. |
| Code entry | Entering a text code on a document: matching it against rule codes then against card codes, the seven distinct refusal messages, the row lock that serializes concurrent entry of the same code. |
| Conflict resolution | The "best global discount wins" comparison, including the rule that when two discounts both exceed the discountable amount the smaller one is preferred; the ban on applying the same discount reward twice; the ban on discounting a document whose total is already zero; the ordering that always puts stored-value payments last. |
| Recomputation triggers | Line change, price-list reprice, carrier change, customer change, code entry, reward claim, reward removal, confirmation, cancellation, cart update, payment. |
| Card generation | The wizard that mints cards in bulk, anonymously or for a selected audience, with a granted balance, an expiry and a history movement. |
| Balance adjustment | The wizard that sets a card's balance to a new value and records the difference as issued or used. |
| Communications | Per-program communication plans that send a message when a card is created and when a balance crosses a milestone, and print a document at a counter. |
| Customer portal | The customer's card list, the card dialogue with balance, code, expiry, the last five movements and the three best affordable rewards, and the paged movement history. |
| Counter channel | The same programs applied inside the selling application at the counter, with client-side point computation, offline operation, code scanning, physical gift cards and reward buttons. |
| Storefront channel | The same programs applied to a web cart, with a pending-code mechanism for anonymous visitors, a claimable-reward list on the cart page, automatic claiming and revalidation at payment. |
| Shareable links | A web address that applies a coupon or a program code and then redirects the visitor to a chosen page, optionally through a shortened tracked link. |
| Merging customers | Nominative cards of merged customers are consolidated onto one card whose balance is the sum. |

---

## 2. Entities the domain owns

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Loyalty Program | `loyalty.program` | `loyalty_program` | One promotional offer: its type, validity, currency, scope and limits. |
| Loyalty Rule | `loyalty.rule` | `loyalty_rule` | One condition of a program: what must be bought, how much, and how many points that earns. |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | One thing a customer may claim: a free product or a discount, with its price formula and its point price. |
| Loyalty Coupon | `loyalty.card` | `loyalty_card` | One instrument carrying a point balance and a unique code: a coupon, a loyalty card, a gift card or an electronic wallet. Called a Loyalty Card throughout this folder. |
| History for Loyalty cards and Ewallets | `loyalty.history` | `loyalty_history` | One movement on a card: points issued, points used, the document that caused it. Called a Loyalty History movement throughout this folder. |
| Loyalty Communication | `loyalty.mail` | `loyalty_mail` | One rule saying which message template to send, and which document to print, on card creation or on reaching a point milestone. |
| Sale Order Coupon Points | `sale.order.coupon.points` | `sale_order_coupon_points` | The pending promise that a given order will credit a given card with a given number of points when it is confirmed. |

Transient entities of the domain:

| Entity | Transport name | Purpose |
|---|---|---|
| Generate Coupons | `loyalty.generate.wizard` | Mints cards in bulk for anonymous holders or for a selected audience. |
| Update Loyalty Card Points | `loyalty.card.update.balance` | Sets a card's balance to a new value and records the difference. |
| Sale Loyalty - Apply Coupon Wizard | `sale.loyalty.coupon.wizard` | Takes a code typed by a salesperson and applies it to a sales order. |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | Lets a salesperson pick one claimable reward, and a free product when several are possible, and adds it to the order. |
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | Produces a shareable web address that pre-loads a code into a visitor's cart. |

Every one of these twelve entities belongs to this folder. No generic platform entity is claimed
here: the message templates, the printable documents, the system parameters, the access groups and
the record rules that this domain configures are owned by the platform foundation and by
[../identity-and-access/](../identity-and-access/), and are listed in
[configuration.md](configuration.md) with a link to the folder that owns each.

### Entities owned by other domains that this domain extends

| Entity | Owning domain | What this domain adds |
|---|---|---|
| Sales Order (`sale.order`) | [../sales/](../sales/) | Applied cards, code-enabled rules, pending point entries, reward total, gift-card count, portal loyalty summary, rewards the shopper removed by hand, and the whole application algorithm hooked into confirmation, cancellation, reprice and cart update. |
| Sales Order Line (`sale.order.line`) | [../sales/](../sales/) | Reward marker, reward reference, card reference, reward grouping code and point cost, plus overrides of description, discount, tax, display price, deletion, portal editability and sellability. |
| Point of Sale Order (`pos.order`) | [../point-of-sale/](../point-of-sale/) | Validation of the point changes computed on the device, creation of the cards the ticket earned, assignment of purchased gift cards, history movements, and attachment of the printed gift card to the receipt message. |
| Point of Sale Order Line (`pos.order.line`) | [../point-of-sale/](../point-of-sale/) | Reward marker, reward reference, card reference, reward grouping code and point cost, plus discount reporting and refund exclusion. |
| Point of Sale Configuration (`pos.config`) | [../point-of-sale/](../point-of-sale/) | Resolution of the programs available at a counter, pre-opening validation of those programs, and the code redemption service. |
| Point of Sale Session (`pos.session`) | [../point-of-sale/](../point-of-sale/) | The four loyalty entities are added to the data set loaded onto the cashier's device. |
| Product Variant and Product Template (`product.product`, `product.template`) | [../products-and-catalog/](../products-and-catalog/) | Archive and delete protection for products used by active programs, the default picture of a gift-card product, public access to reward product images, and the counter product loading. |
| Price List (`product.pricelist`) | [../pricing-and-pricelists/](../pricing-and-pricelists/) | Archive protection when an active program restricts itself to that price list. |
| Customer (`res.partner`) | [../contacts-and-organizations/](../contacts-and-organizations/) | The count of the customer's active cards and the action that lists them. |
| Customer Merge Wizard (`base.partner.merge.automatic.wizard`) | [../contacts-and-organizations/](../contacts-and-organizations/) | Consolidation of the nominative cards of the merged customers. |
| Journal Item (`account.move.line`) | [../general-ledger/](../general-ledger/) | Classification of a reward line as a discount line for reporting. |
| Barcode Rule (`barcode.rule`) | [../products-and-catalog/](../products-and-catalog/) | A coupon barcode rule type and the shipped rule that recognizes coupon and gift-card barcodes. |

---

## 3. Actors

| Actor | Description |
|---|---|
| Sales Administrator | Internal user in the sales administration group (`sales_team.group_sale_manager`). Creates, edits, archives and deletes programs, rules, rewards and communication plans; generates cards; shares coupon links. |
| Salesperson | Internal user in the own-documents sales group (`sales_team.group_sale_salesman`). Reads programs, rules, rewards and communication plans; reads and updates cards; generates cards; applies codes and rewards to orders; reads, creates and updates history movements. |
| Point of Sale Administrator | Internal user in the counter management group (`point_of_sale.group_pos_manager`). Creates, edits and deletes programs, rules, rewards and communication plans usable at a counter; creates cards. |
| Point of Sale Cashier | Internal user in the counter user group (`point_of_sale.group_pos_user`). Reads programs, rules and rewards; reads and updates cards; generates cards; scans coupon and gift-card codes; claims rewards on a ticket. |
| Customer | Buys under a promotion, receives coupons and gift cards by message, enters codes in the storefront, claims rewards from the cart, and reads card balances and history on the portal. |
| Storefront visitor without an account | May follow a coupon link; the code is remembered and applied as soon as a cart exists. Nominative programs are refused for such a visitor. |
| Scheduled clean-up | A non-interactive actor that detaches manually applied cards from abandoned storefront carts. |

---

## 4. Files in this folder

| File | Content |
|---|---|
| [entities.md](entities.md) | Every entity in full: purpose, life cycle, complete field table with identifier and full name, relations, constraints, ordering, display name, archival, company behavior and the extension points other packages contribute. |
| [state-machines.md](state-machines.md) | The seven derived state machines of the domain: program availability, card usability, card ownership, pending point entry, reward line, attachment of a card or rule to an order, and the counter point change; each with states, transitions, guards, refusal messages and a diagram. |
| [workflows.md](workflows.md) | End-to-end procedures: creating a program, generating cards, applying a code, claiming a reward, confirming and cancelling, buying and spending a gift card, topping up and spending a wallet, issuing next-order coupons, merging customers, and a full worked walkthrough with four programs on one order. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with exact error messages, guards, permissions, uniqueness, locking, rounding and date rules, and the mapping from the identifiers used before this consolidation. |
| [calculations.md](calculations.md) | Every formula and algorithm: point granting, claimable rewards, discountable amounts, tax distribution, free-quantity computation, best-discount comparison, point cost and rounding, each with worked numeric examples. |
| [accounting-effects.md](accounting-effects.md) | What reaches the general ledger, how reward lines are invoiced, how accounts are selected, and the boundary of this domain's accounting responsibility. |
| [configuration.md](configuration.md) | Settings, system parameters, access groups, model access rules, record rules, shipped records, menus, printable documents and master-data prerequisites. |
| [interfaces.md](interfaces.md) | Named operations, request addresses, screens described as workflows on views, printable documents, messages, scheduled jobs and the figures the domain exposes. |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given, When and Then scenarios with concrete numbers that a rebuild must pass. |
| [glossary.md](glossary.md) | Every term of the domain, defined. |
| [point-of-sale-application.md](point-of-sale-application.md) | The complete client-side evaluation contract used at a counter: data loading, offline behavior, code scanning, physical gift cards, electronic wallets, the two server exchanges and the restaurant specifics. |
| [storefront-application.md](storefront-application.md) | The storefront behavior: cart recomputation, automatic claiming, pending coupon links, the promotional code form, checkout display and payment-time revalidation. |

---

## 5. Reading order

1. **[glossary.md](glossary.md)** — the vocabulary. The words *program*, *rule*, *reward*, *card*,
   *point*, *trigger*, *nominative*, *payment program*, *global discount* and *discountable* all
   have precise meanings that the rest of the domain depends on. Read this first.
2. **[entities.md](entities.md)** — the seven persistent entities and the five transient ones, field
   by field, with every default, every computed rule and every uniqueness constraint.
3. **[state-machines.md](state-machines.md)** — programs and cards have no status column; their
   behavior is governed by derived states (validity window, exhaustion, expiry, nominativeness) and
   by the status of the document that carries them. Every such state is tabulated here.
4. **[calculations.md](calculations.md)** — the heart of the domain: point earning, the discountable
   base in its four variants, reward pricing, the per-tax split, the maximum-discount ceiling, the
   point cost and the ordering comparisons. Every formula has a worked example.
5. **[workflows.md](workflows.md)** — the five-step application algorithm in full, then every
   operational flow around it.
6. **[business-rules.md](business-rules.md)** — every validation, every constraint, every error
   message with its exact text, every permission check, the row lock and the concurrency behavior.
7. **[accounting-effects.md](accounting-effects.md)** — the journal items for a discount, for the
   sale of a gift card, for the redemption of a gift card, for an electronic wallet top-up and
   spend, and for a free product.
8. **[configuration.md](configuration.md)** — settings, parameters, shipped records, groups, access
   rights, record rules.
9. **[interfaces.md](interfaces.md)** — operations, addresses, screens, printable documents,
   messages.
10. **[point-of-sale-application.md](point-of-sale-application.md)** and
    **[storefront-application.md](storefront-application.md)** — the two channel contracts.
11. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered scenarios with concrete numbers,
    including a tenth off two hundred, a discount on the cheapest line, a free product, free
    shipping, a gift card of fifty partly spent, points at one per unit of currency redeemed at a
    hundred, two programs competing, and a code that does not apply.

---

## 6. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Sales](../sales/README.md) | The Sales Order and Sales Order Line entities, their pricing, their tax computation, their status machine and their invoicing. Reward lines **are** sales order lines; everything the sales domain says about a line's price, tax and invoicing applies to them unchanged unless this domain states an exception. |
| [Point of Sale](../point-of-sale/README.md) | The counter ticket, its lines, its client-side price computation and its session posting. The counter is a second channel for exactly the same programs. |
| [Website and Storefront](../website-and-storefront/README.md) | The cart life cycle, the promotional code form, the checkout summary, the express checkout amounts, the storefront restriction on a program, the shareable coupon link and the shortened link tracker. |
| [Products and Catalog](../products-and-catalog/README.md) | Product variants, product categories, product tags, the barcode nomenclature, and the service product type used for reward lines and top-up products. |
| [Taxes](../taxes/README.md) | The tax computation used to build the discountable base and to split a discount per tax combination, the identification of fixed-amount taxes, and the fiscal position mapping applied to reward lines. |
| [Pricing and Pricelists](../pricing-and-pricelists/README.md) | The price list a program may be restricted to, the price used to value a free product, and the reprice action that triggers a recomputation. |
| [Multi-Currency](../multi-currency/README.md) | The conversion of a program's monetary amounts (minimum purchase, fixed discount, maximum discount) into the document's currency, and the rounding of a monetary point balance. |
| [Delivery and Shipping](../delivery-and-shipping/README.md) | The delivery product carried on a delivery line, which the free-shipping reward reads and cancels, and the free-above-a-threshold recomputation. |
| [Contacts and Organizations](../contacts-and-organizations/README.md) | The customer who owns a nominative card, the customer tags used when generating cards, and the merge wizard. |
| [Messaging and Activities](../messaging-and-activities/README.md) | Message templates, the composition window, the discussion thread on a card and the tracked point balance. |
| [Identity and Access](../identity-and-access/README.md) | The four selling groups that gate every access right of the domain, the technical-features group, and the company record rules. |
| [General Ledger](../general-ledger/README.md) and [Accounts Receivable](../accounts-receivable/README.md) | The journal entries that carry a discount or a stored-value movement to the books, and the credit notes that reverse them. |
| [Analytic Accounting](../analytic-accounting/README.md) | The analytic distribution carried by a reward line. |
| [Inventory Valuation and Costing](../inventory-valuation-and-costing/README.md) | The inventory output entry produced when a stocked free product is delivered. |
| [Customer Relationship Management](../customer-relationship-management/README.md) | The sales access groups whose rights this domain extends. |

The platform documents that apply throughout are
[../../overview/security-model.md](../../overview/security-model.md) for the access model,
[../../runtime/scheduled-jobs.md](../../runtime/scheduled-jobs.md) for the abandoned-cart clean-up,
[../../runtime/transactions-and-concurrency.md](../../runtime/transactions-and-concurrency.md) for
the row lock that guards the usage ceiling,
[../../runtime/configuration-parameters.md](../../runtime/configuration-parameters.md) for the
system parameters, [../../runtime/mail-gateway.md](../../runtime/mail-gateway.md) for message
delivery, [../../runtime/report-rendering.md](../../runtime/report-rendering.md) for the printable
coupon and gift-card documents, and
[../../interfaces/endpoint-catalog.md](../../interfaces/endpoint-catalog.md) for the request
addresses listed in [interfaces.md](interfaces.md).

---

## 7. What this domain deliberately does not do

- It never creates a journal entry itself. Its output is always a line on a document owned by
  another domain.
- It never changes the price of a non-reward line. A discount is always a separate negative line,
  never a reduction of an existing line's unit price or discount percentage. The only exception is
  the free-product reward on a sales order, which is a normal product line carrying a hundred
  percent line discount; at a counter the same reward is instead a negative line on the hidden
  discount product.
- It never partially grants a reward. If the card has ninety points and the reward costs a hundred,
  nothing is granted; there is no pro-rated reward. The single exception is a stored-value payment,
  which is by construction proportional to the amount spent.
- It never lets a discount take a document's total below zero. Every discount is capped by the
  remaining discountable amount and by the document total including tax.
- It never defers the revenue of a gift card and never recognizes breakage. Both are configuration
  decisions, explained in [accounting-effects.md](accounting-effects.md).
- It never models a gift card as a payment. A gift card is a negative sales line, so it never
  appears in a bank reconciliation and never carries a payment reference.

---

## 8. Reconciliation notes

Two independently written descriptions of this domain were merged into this folder. The resolutions
that affect this file are:

1. **Folder title.** One version titled the domain "Loyalty and Promotions". The folder key
   and the title used throughout the repository are *Loyalty and Promotions*; coupons are covered by
   the same title.
2. **Entity naming.** One version listed the entities under invented storage names. The transport
   names and catalogue full names are reproduced here, with the shorter name used in prose stated
   next to each one where the catalogue name is a sentence rather than a name.
3. **Sibling folder keys.** One version linked to folders named *commerce-storefront*,
   *website-and-content-management* and *messaging-and-collaboration*. Those links now point to
   [../website-and-storefront/](../website-and-storefront/) and
   [../messaging-and-activities/](../messaging-and-activities/).
4. **Ownership of shared records.** Only one version stated which entities the folder owns. The
   twelve entities of section 2 are owned here; every message template, printable document, system
   parameter, access group and record rule this domain configures belongs to the platform foundation
   or to [../identity-and-access/](../identity-and-access/), and is linked from
   [configuration.md](configuration.md).
