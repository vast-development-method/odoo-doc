# Configuration

Every setting, parameter, default, access group, shipped record and master-data prerequisite of the Loyalty and Promotions domain, with its data type, its default and its effect.

## 1. Master-data prerequisites

| Prerequisite | Owning domain | Why it is needed |
|---|---|---|
| At least one company with a currency | [../contacts-and-organizations/](../contacts-and-organizations/) | The program's currency defaults to the company currency, and the company drives the record rules and the currency conversions. |
| At least one sellable product | [../products-and-catalog/](../products-and-catalog/) | Several program templates preselect the first sellable product as the rule product and the reward product. |
| A product category for services | [../products-and-catalog/](../products-and-catalog/) | The shipped gift card and wallet top-up products are placed in it; the hidden discount products inherit their income account from their category. |
| A gift card product and a wallet top-up product sold as services at their face value | [../products-and-catalog/](../products-and-catalog/) | They are the trigger products of the gift card and electronic wallet programs. Two are shipped and may be replaced. |
| Income accounts reachable from the product categories | [../general-ledger/](../general-ledger/) | The negative reward lines need an income account when the order is invoiced. |
| At least one pricelist per currency used | [../pricing-and-pricelists/](../pricing-and-pricelists/) | Only needed when a program is restricted to pricelists; the restriction refuses pricelists of another currency. |
| Email templates defined on the Loyalty Card entity | [../messaging-and-activities/](../messaging-and-activities/) | The communication plan may only use templates defined on that entity. Two are shipped. |
| A barcode nomenclature | [../products-and-catalog/](../products-and-catalog/) | The shipped coupon barcode rule attaches to the default nomenclature so that a counter can scan coupon codes. |
| A shipping method | [../delivery-and-shipping/](../delivery-and-shipping/) | Only needed for free shipping rewards, which exist only when the shipping capability package is enabled. |

## 2. Capability packages

The domain is delivered as seven packages. Each one adds behavior to the previous ones; none of them can be used alone except the first.

| Package | Adds |
|---|---|
| Coupons and Loyalty | The four core entities, the two wizards, the portal pages, the printed documents, the shipped products, the shipped gift card program, the guards on products and pricelists, and the contact card count. |
| Sale Loyalty | The whole sales application: the evaluation on the order, the reward and coupon wizards, the pending promises, the sales channel flag, the portal display and the invoicing classification. |
| Sale Loyalty - Delivery | The free shipping reward type, the exclusion of shipping lines from the thresholds and the point counting, and the free-shipping-above-a-threshold recomputation. |
| Point of Sale - Coupons and Loyalty | The counter application: the data loading, the counter restriction, the code redemption service, the ticket confirmation exchange, the coupon barcode rule and the counter printed documents. |
| Point of Sale - Sales Loyalty | The bridge that suppresses the evaluation while a sales order is settled onto a ticket and that excludes settled lines from the point counting. |
| Point of Sale - Restaurant Loyalty | The bridge that assigns reward lines to the last course and re-evaluates when the table changes. |
| Coupons, Promotions, Gift Card and Loyalty for the online shop | The storefront application: the online shop flag, the website restriction, the coupon link, the promotional code form, the automatic claiming, the checkout display, the payment revalidation, the wallet top-up route and the shareable link wizard. |

## 3. Settings screen

| Setting | Where | Type | Default | Effect |
|---|---|---|---|---|
| Coupons and Loyalty (also labelled "Discounts, Loyalty and Gift Card" on some screens) | Sales configuration, pricing section | boolean | false | Enables the domain by installing the Coupons and Loyalty package together with the Sale Loyalty package. Disabling it removes the menus but leaves the data. |
| `Discount & Loyalty` menu | Point of sale configuration | action button | not applicable | Opens the discount and loyalty program list filtered to the program types other than gift card and electronic wallet. |
| `Gift cards & eWallet` menu | Point of sale configuration | action button | not applicable | Opens the program list filtered to the gift card and electronic wallet types. |

There is no per-company setting and no per-counter setting other than the program's own `pos_config_ids` restriction.

## 4. System parameters

System parameters are key-and-value records held by the platform, shared by every company and settable by an administrator; the platform document [../../runtime/configuration-parameters.md](../../runtime/configuration-parameters.md) describes the store itself. Each key below is reproduced exactly, because an existing deployment and its integrations read it under that spelling — including the misspelling in the third key, which is part of the stored value. Every value is stored as text and is interpreted as described in the Effect column.

| Parameter | Shipped value | Value assumed when the parameter is absent | Effect |
|---|---|---|---|
| `loyalty.compute_all_discount_product_ids` | `False` | `enabled` | When the effective value is exactly `enabled`, the discounted-product filter of every reward is expanded eagerly into a stored list of products, and the serialized condition sent to a counter is the literal text `null`. When it is anything else, the expansion is skipped, the product list stays empty and the serialized condition is sent instead, so that the counter evaluates the condition itself. The shipped value therefore disables the expansion; a deployment with few products may enable it to make the counter faster. |
| `loyalty.timezone` | not shipped | the coordinated universal time zone | The time zone used to judge program validity when the company's own contact declares none. |
| `website_sale_coupon.abandonned_coupon_validity` | not shipped | `4` | Number of days after which a draft online cart that has not been written to is considered abandoned and has its manually applied cards detached. |
| `sale.automatic_invoice` | not shipped | false | Owned by [../sales/](../sales/), where it backs the Automatic Invoice setting. When true, an order whose total including tax is zero and whose reward total is not zero is invoiced and posted automatically. |
| `sale.default_invoice_email_template` | not shipped | none | Owned by [../sales/](../sales/), where it backs the Email Template setting of automatic invoicing. The template used to send the invoice produced by the rule above. |

## 5. Access groups

This domain ships **no access group of its own**. It grants rights to five groups owned by other
domains, and to no other group. Each group is named here by its reproduced identifier and by its
role in words; [../../overview/security-model.md](../../overview/security-model.md) describes the
access model itself.

| Group identifier | Role in words | Owning domain | Role in this domain |
|---|---|---|---|
| `base.group_user` | Internal user | [../identity-and-access/](../identity-and-access/) | Explicitly granted **nothing** on every loyalty entity. A deployment with neither the sales nor the counter capability exposes no loyalty data at all. |
| `sales_team.group_sale_salesman` | Salesperson, own documents only | [../customer-relationship-management/](../customer-relationship-management/) | Reads the configuration, uses the cards, applies codes and rewards. |
| `sales_team.group_sale_manager` | Sales Administrator | [../customer-relationship-management/](../customer-relationship-management/) | Maintains the configuration and the shareable links. |
| `point_of_sale.group_pos_user` | Point of Sale Cashier | [../point-of-sale/](../point-of-sale/) | Reads the configuration, uses the cards at a counter. |
| `point_of_sale.group_pos_manager` | Point of Sale Administrator | [../point-of-sale/](../point-of-sale/) | Maintains the configuration usable at a counter. |

The technical-features group `base.group_no_one` of
[../identity-and-access/](../identity-and-access/) is granted no right at all; it only reveals the
advanced fields on the rule and reward screens — the free-form product conditions, the point-grant
block on program types that hide it, the hidden discount product and the whole-balance flag.

## 6. Model access rules

Rights are written as read / write / create / delete. Column headings use the group identifiers of
section 5.

| Entity | `base.group_user` | `sales_team.group_sale_salesman` | `sales_team.group_sale_manager` | `point_of_sale.group_pos_user` | `point_of_sale.group_pos_manager` |
|---|---|---|---|---|---|
| Loyalty Program (`loyalty.program`) | none | read | read, write, create, delete | read | read, write, create, delete |
| Loyalty Rule (`loyalty.rule`) | none | read | read, write, create, delete | read | read, write, create, delete |
| Loyalty Reward (`loyalty.reward`) | none | read | read, write, create, delete | read | read, write, create, delete |
| Loyalty Communication (`loyalty.mail`) | none | read | read, write, create, delete | read | read, write, create, delete |
| Loyalty Card (`loyalty.card`) | none | read, write | read, write, create | read, write | read, write, create |
| Loyalty History movement (`loyalty.history`) | none | read, write, create | read, write, create | read, write, create | read, write, create |
| Card Generation Wizard (`loyalty.generate.wizard`) | none | read, write, create | read, write, create | read, write, create | read, write, create |
| Card Balance Wizard (`loyalty.card.update.balance`) | none | read, write, create | read, write, create | read, write, create | read, write, create |
| Sales Order Coupon Points (`sale.order.coupon.points`) | none | read | read, write, create, delete | none | none |
| Coupon Entry Wizard (`sale.loyalty.coupon.wizard`) | none | read, write, create | read, write, create | none | none |
| Reward Selection Wizard (`sale.loyalty.reward.wizard`) | none | read, write, create | read, write, create | none | none |
| Coupon Sharing Wizard (`coupon.share`) | none | none | read, write, create | none | none |

Two consequences a rebuild must reproduce:

1. **Nobody may delete a Loyalty Card through the access rules.** Every deletion of a card performed
   by the recomputation runs with elevated rights.
2. A Salesperson may not create or delete pending point entries, yet saving a quotation creates them
   and cancelling an order deletes them. Those writes also run with elevated rights.

## 7. Record rules

Five record rules, one per stored entity, all with the same shape and all applying to every operation:

A record passes when its company is empty, **or** its company is one of the user's allowed
companies, **or** its company is a parent of one of the user's allowed companies.

| Rule name | Entity |
|---|---|
| Loyalty program multi company rule | Loyalty Program |
| Loyalty card multi company rule | Loyalty Card |
| Loyalty history multi company rule | Loyalty History movement |
| Loyalty rule multi company rule | Loyalty Rule |
| Loyalty reward multi company rule | Loyalty Reward |

The company of a rule, a reward, a card and a history movement is a stored mirror of the program's company, written for exactly this purpose.

## 8. Shipped records

### 8.1 Products

| Record | Name | Price | Type | Attributes |
|---|---|---|---|---|
| Gift card product | `Gift Card` | 50 | service | not purchasable, in the services category, carries the shipped gift card picture; when the sales package is present it carries no customer tax; when the point-of-sale package is present it is available at a counter and carries no customer tax. |
| Wallet top-up product | `Top-up eWallet` | 50 | service | not purchasable, in the services category; when the sales package is present it carries no customer tax; when the point-of-sale package is present it is available at a counter and carries no customer tax. |

Neither product may be deleted, as a variant or as a template; the attempt is refused with `You cannot delete <name> as it is used in 'Coupons & Loyalty'. Please archive it instead.` See `LOY-149` in [business-rules.md](business-rules.md).

### 8.2 The shipped gift card program

| Record | Values |
|---|---|
| Program `Gift Cards` | type `gift_card`, `applies_on` `future`, `trigger` `auto`, portal visible, portal point name `$`, email template the shipped gift card template; when the point-of-sale package is present, printed document the shipped gift card document. |
| Its rule | `reward_point_amount` 1, `reward_point_mode` `money`, `reward_point_split` true, products limited to the shipped gift card product. |
| Its reward | type `discount`, mode `per_point`, `discount` 1, applicability `order`, `required_points` 1. |

### 8.3 Email templates

| Template | Subject | Recipient | Attachment |
|---|---|---|---|
| `Gift Card: Gift Card Information` | `Your Gift Card at <company name>` | the default recipients of the card | the printed gift card |
| `Coupon: Coupon Information` | `Your reward coupon from <company name> ` | the default recipients of the card | the printed coupon |

Both templates are defined on the Loyalty Card entity, use the default recipient resolution of the card, send from the company email address for the coupon template, and are deleted once they have been sent.

### 8.4 Printed documents

| Document | Name | Defined on | Bound as |
|---|---|---|---|
| Coupon document | `Coupon Code` | Loyalty Card | a printing action available from the card list and form |
| Gift card document | `Gift Card` | Loyalty Card | a printing action available from the card list and form |

Both render one page per card, in the language of the card's recipient when they have one.

### 8.5 Barcode rule

| Record | Values |
|---|---|
| `Coupon & Gift Card Barcodes` | attached to the default barcode nomenclature, sequence 50, type `coupon`, any encoding, pattern matching codes that begin with `043` or `044`. |

### 8.6 System parameter

The parameter `loyalty.compute_all_discount_product_ids` is shipped with the value `False`. The shipped record is written only when the parameter does not already exist, so a value an administrator has set for a deployment is preserved and never replaced by the shipped value.

## 9. Screens, actions and menus

### 9.1 Window actions

| Action | Opens | Restriction |
|---|---|---|
| `Discount & Loyalty` | the program list and form at the path `discount-loyalty` | programs whose type is neither `gift_card` nor `ewallet` |
| `Gift cards & eWallet` | the program list and form at the path `gift-cards-ewallet`, with the gift card and wallet form | programs whose type is `gift_card` or `ewallet`; the creation defaults to `gift_card` and the template chooser shows the gift card and wallet templates |
| `Coupons` | the card list and form | cards of the program the action was opened from; creation is disabled, so cards are only created through the generation wizard or by a document |
| `Generate` | the generation wizard, as a dialog | none |
| `Available Rewards` | the reward selection wizard, as a dialog | none |
| `Enter Promotion or Coupon Code` | the coupon code wizard, as a dialog | none |

The program list of both program actions is shown before the form, and the gift card action uses a simplified form.

### 9.2 Menus

| Menu | Parent | Access group | Sequence |
|---|---|---|---|
| `Discount & Loyalty` | the sales product catalogue menu | `sales_administrator` | 40 |
| `Gift cards & eWallet` | the sales product catalogue menu | `sales_administrator` | 50 |
| `Discount & Loyalty` | the point-of-sale catalogue menu | `point_of_sale_manager` | 91 |
| `Gift cards & eWallet` | the point-of-sale catalogue menu | `point_of_sale_manager` | 92 |
| `Loyalty` | the online shop menu | `sales_administrator` | 4 |
| `Discount & Loyalty` | the `Loyalty` menu above | inherited | inherited |
| `Gift cards & eWallet` | the `Loyalty` menu above | inherited | inherited |

### 9.3 Defaults applied when a screen opens

| Screen | Default |
|---|---|
| Card list opened from a program | the program of the action becomes the default program of the generation wizard; the wizard's mode defaults to `selected` for an electronic wallet program and to `anonymous` otherwise; the list title and the card-count label use the family noun of the program. |
| Generation wizard | the program is the one the wizard was opened from; `mode` is `anonymous` unless the action said otherwise; the grant is 1. |
| Coupon code wizard | the order is the one the wizard was opened from. |
| Reward selection wizard | the order is the one the wizard was opened from; the reward list is the claimable rewards, restricted to the rewards named by the calling action when there are any. |
| Coupon share wizard | the program, the card and the website are taken from the record the share action was pressed on. |
| New rule or new reward inside a program | the field defaults of the program's family are applied, so that a rule added by hand to a promotion starts with the promotion defaults rather than with the plain field defaults. |

## 10. What a deployment must decide

| Decision | Options | Consequence |
|---|---|---|
| Income account of the hidden discount products | the product category account, a dedicated contra-revenue account | Determines whether discounts are reported separately from gross revenue. See [accounting-effects.md](accounting-effects.md). |
| Account of the gift card and wallet products | an income account, a liability account | A liability account is required to defer the revenue of unspent cards. The hidden discount product of the payment reward must use the **same** account. |
| Expansion of the discounted-product filter | disabled (shipped) or enabled | Enabled makes a counter evaluate faster but stores a product list per reward that must be recomputed whenever a product changes. |
| Evaluation time zone | the company contact's time zone, or the `loyalty.timezone` parameter | Determines on which calendar day a validity window opens and closes. |
| Abandonment delay for online carts | four days, or any other number of days | Determines how long a shopper's applied coupon stays reserved on an abandoned cart. |
