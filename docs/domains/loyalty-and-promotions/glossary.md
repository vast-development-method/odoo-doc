# Glossary

Terms of the Loyalty and Promotions domain, with full-word definitions. A term written in bold inside a definition is itself defined in this glossary.

**Aggregate branch** The branch of the point computation that adds the points of a **rule** to a single running total, producing one point value for the whole document. It is taken whenever the **split branch** is not.

**Applicability (of a discount)** Which part of a document a discount reward reduces: the whole order, one unit of the cheapest eligible line, or the lines whose product matches the reward's discounted-product filter.

**Applied card** A **card** attached to a document because its code was entered, or because the customer owns it and the program is nominative. Applied cards are the second source, besides the **pending promises**, of the cards a document may claim rewards from.

**Automatic candidate** A **program** whose trigger is automatic, at least one of whose rules is automatic, that matches the **program filter**, that has not reached its usage ceiling, and that does not already grant points on the document. Automatic candidates are attached to the document at every evaluation.

**Balance** The number of **points** a **card** currently holds.

**Bearer card** A **card** with no owner. Whoever holds the code may spend it. Coupons and gift cards are normally bearer cards.

**Buy some get some** A **program** family in which every purchased unit grants one credit and a number of credits is exchanged for free units of a product.

**Buy-some-take-some distribution** The arithmetic that turns a number of items and a "buy so many,
take so many" ratio into a number of free items, defined in [calculations.md](calculations.md),
section 10.1. It is what a counter uses to decide how many units of a free-product reward a ticket
has earned.

**Card** See **Loyalty Card**.

**Channel flag** One of the three booleans that say on which channel a **program** may be used: sales, point of sale, online shop.

**Claimable reward** A **reward** that a given **card** may pay for on a given document right now, after every guard has been applied.

**Clear the whole balance** A reward option that makes one claim consume the whole remaining **balance** of the card instead of exactly the required points, and yields exactly one occurrence of the reward.

**Communication plan** The set of **Loyalty Communication** rules of a **program**: what to send when a card is created and when a balance crosses a milestone.

**Confirmation exchange** The second and last conversation between a counter device and the server,
run once the ticket exists on the server: it creates the cards, applies the points, binds the reward
lines, sends the creation communications and writes the history.

**Counting lines** The document lines that carry a product and no **reward**, used to compute quantities and to judge the quantity gate of a **rule**. Shipping lines are excluded when the shipping capability package is present.

**Coupon** A **card** of a program of the `coupons` family: a bearer code that grants immediate access to a reward.

**Discount factor** The ratio of the granted discount to the **discountable amount**, capped at one, by which every entry of the **discountable per tax** breakdown is multiplied to produce the reward lines.

**Discountable amount** What a discount is allowed to reduce on a document, computed differently for each **applicability**. Every computation produces a total and a breakdown per tax combination.

**Discountable per tax** The breakdown of the **discountable amount** by tax combination. It is what lets a single discount become one document line per tax combination, so that the tax amounts stay exact.

**Electronic wallet** A **program** family that stores a monetary balance on a nominative **card**, credited by buying a top-up product and spent as a means of payment.

**Evaluation** The routine that re-examines a document against every applicable **program**, recomputes the points it grants, rebuilds its reward lines and cleans up what is no longer justified.

**Evaluation time zone** The time zone in which the **reference date** of a document is computed.

**Exhausted card** A **card** whose **balance** no longer reaches the smallest point price among the
**rewards** of its **program**. Its code is refused with "This coupon has already been used."

**Family defaults** The set of field values, rules, rewards and communication rules that a **program** receives when its type is set or changed.

**Fixed-amount tax** A tax expressed as a flat amount per unit rather than as a percentage. It is never reduced by a discount that does not belong to a **payment program**.

**Free product reward** A **reward** that gives a number of units of a product at no charge. On a sales order it is a product line with a hundred percent discount; at a counter it is a negative line on the **hidden discount product**.

**Gift card** A **program** family that stores a monetary balance on a bearer **card**, sold as a product and spent as a means of payment.

**Global discount** A **reward** of type discount whose applicability is the whole order and whose mode is a percentage or a fixed amount per order. Only one may be applied to a document at a time.

**Grant** The number of **points** a **rule** awards, together with the mode that says what it is counted on.

**Hidden discount product** The service product created automatically for every **reward**, used to carry the reward amount on a document line. It is not sellable, not purchasable and priced zero.

**History entry** See **Loyalty History movement**.

**Local identifier** The negative identifier a counter device gives a **card** that does not exist on
the server yet. The confirmation exchange returns the mapping from local identifiers to real ones.

**Loyalty Card** An individual coupon, gift card, electronic wallet or loyalty card: a unique code, a **balance**, an optional owner and an optional expiration date.

**Loyalty Communication** One rule of a **communication plan**: when to send, which email template to use, and which document to print at a counter.

**Loyalty History movement** One movement on a **card**: points issued, points used, a description and a reference to the document that caused it.

**Loyalty Program** The offer definition: the type, the validity window, the usage limit, the channels, the currency, the customer restrictions, and the collections of **rules**, **rewards** and communication rules.

**Loyalty Reward** One thing a customer may obtain from a **program**, with its point price and the parameters of its kind.

**Loyalty Rule** One condition set of a **program**, together with the **grant** awarded when the condition is met and the optional code that activates it.

**Manually removed reward** A **reward** whose line a shopper deleted from an online cart. Such a reward is never claimed automatically again on that cart.

**Milestone** A **balance** threshold that triggers an email when a **card** crosses it upward. Only the highest milestone crossed by one movement is sent.

**Next-order coupon** A **program** family that issues a bearer **card** to the customer when an order meets its rules, for use on a later order.

**Nominative program** A **program** whose points belong to an identified customer and are kept on a single **card** per customer. A program is nominative when its point usage mode is "current and future orders", or when it is a loyalty or electronic wallet program whose points apply to future orders.

**Payment program** A **gift card** or **electronic wallet** program. Its reward behaves as a means of payment: it applies to the whole document total including every tax, it may reduce **fixed-amount taxes**, and it is always recomputed after every other reward.

**Pending promise** See **Sales Order Coupon Points**.

**Placeholder line** The single line named `TEMPORARY DISCOUNT LINE`, of quantity zero, price zero
and point cost zero, that keeps a discount **reward** attached to a document whose discountable
amount fell to zero because a **payment program** already paid it all.

**Plural item name** The word a **program** type uses for its **cards** in a count, for example
`Gift Cards` or `Promos`. Listed in [entities.md](entities.md), section 1.7.

**Point correction** The adjustment that removes the contribution of free-product **reward lines**
from a point count, so that a rule granting points per unit of currency spent does not treat a
give-away as a payment. Defined in [calculations.md](calculations.md), section 10.3.

**Point cost** The number of **points** that claiming a **reward** consumes on a **card**. When a reward produces several document lines, only the first carries the cost.

**Point name** The label under which **points** are shown to the customer, for example "Loyalty point(s)" or a currency symbol.

**Points** The unit in which a **card**'s **balance** is expressed. For a gift card or an electronic wallet one point is one unit of the program currency; for a loyalty program it is whatever the program's point name says.

**Points available** The number of **points** of a **card** that may be spent on a given document right now: the stored balance, plus what the document will grant when it is confirmed, minus what its reward lines already consume.

**Pooled line** A **reward line** that a recomputation has neutralised — point cost, unit price and
technical unit price set to zero, references kept — so that it can be rewritten in place and keep a
description a salesperson edited.

**Preset** The set of field values, rules, rewards and communication plans a **program** receives
when its type is set or changed. Also called the family defaults.

**Program** See **Loyalty Program**.

**Program filter** The set of conditions a **program** must satisfy to apply to a document: active, published on the document's channel, of a compatible company, compatible with the document's pricelist and website, and valid at the document's **reference date**.

**Promotional code** The text a customer types to activate a **rule**. It is unique among active rules and may not collide with any active **card** code.

**Reference date** The date at which a document's **program filter** and card expirations are judged: today's date in the **evaluation time zone**, or the date of the earliest confirmed payment transaction of the document when there is one.

**Reward** See **Loyalty Reward**.

**Reward grouping code** A random text shared by every document line produced by one application of one **reward**, so that a discount split across several taxes can be recognized, recomputed and deleted as one unit.

**Reward line** A document line produced by claiming a **reward**: a negative line for a discount, a payment or free shipping, and a product line at a zero net price for a free product.

**Rule** See **Loyalty Rule**.

**Sales Order Coupon Points** The pending point effect of one sales order on one **card**: how many points that order will add to that card when it is confirmed. Also called a **pending promise**.

**Split branch** The branch of the point computation that produces one point value, and therefore one **card**, per matched unit instead of a single aggregate value. It is taken when the program applies to future orders, the rule's split option is on, and the grant is not per order.

**Threshold-neutral line** A document line that never counts towards a minimum purchase and never earns points: a shipping line or a free shipping **reward line**.

**Top-up product** The product whose purchase credits an **electronic wallet**.

**Trigger** Whether a **program** becomes available automatically or only after a code is entered.

**Trigger product** A product whose purchase activates a **program**, in particular the gift card product of a gift card program and the **top-up product** of an electronic wallet program.

**Usage ceiling** The maximum number of documents that may use a **program**, counted across the sales, point-of-sale and online shop channels together.

**Use count** The number of document lines that reference a **card**. A card with a non-zero use count is never deleted automatically.

**Validation exchange** The first conversation between a counter device and the server, run just
before the payment screen accepts a ticket: it checks that every card still exists, that every
balance is sufficient and that no new code collides.

## Reconciliation notes

1. Two independently written descriptions of this domain were merged into this folder. Their two
   vocabularies agreed on every term they shared; the terms above marked with a section reference
   were added during the merge so that every word used in the other files is defined here.
2. One version called a row of the movement log a *history entry* and the other a *movement*. The
   folder now uses **Loyalty History movement** for the record and *movement* for one of its rows,
   with *history entry* kept as a pointer to it.
3. One version spelled the common noun *programme*. The folder uses *program* throughout, which is
   also the spelling of the entity's own name, Loyalty Program.
