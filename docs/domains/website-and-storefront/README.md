# Website and Storefront

This domain specifies the public-facing side of the system: the **website** (a content management
capability with pages, menus, themes, editable content, redirects, visitor tracking and multi-site
resolution), the **storefront** (the online sales channel built on top of it: published products,
categories, the shopping cart, the checkout flow, delivery selection, online payment, abandoned-cart
recovery, wish lists, product comparison, reviews and product feeds), the **customer portal** (the
authenticated document area where a signed-in customer sees quotations, orders, invoices and
tickets), and the **community modules** that publish content on the same site: the discussion forum
with its reputation economy, the blog, the tracked-link service and the public profile pages.

It also specifies the plumbing that every public page depends on: the request-to-site resolution
algorithm, the language segment in the address, the slug grammar, the record-visibility filters that
decide which records a given site may show, the page-visibility gate, the **copy-on-write algorithm
that gives one site its own version of a shared page**, the editable-content saving contract with
its sanitisation rules, the generic form builder with its field whitelists, and the anti-robot
challenge providers.

Three algorithms in this folder are load-bearing for any re-implementation and are written out step
by step:

1. The **cart algorithm** — how a line is found, added, updated, verified and removed, how the
   quantity actually granted may differ from the quantity requested, and how the cart is revalidated
   after every change (`workflows.md` §3, `calculations.md` §4).
2. The **checkout algorithm** — the ordered step list, the address rules, the delivery-method
   selection, the payment hand-off and the confirmation (`workflows.md` §4).
3. The **copy-on-write algorithm** — what happens the first time a shared page or template is edited
   in the context of one site, including the fate of every template that inherits from the copied
   one (`workflows.md` §7, `business-rules.md` §6).

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Multi-site hosting | Several independent sites in one database, each with its own domain name, company, languages, theme, public user, menus, pages and settings; a deterministic resolution of an incoming request to exactly one site. |
| Record visibility per site | A uniform rule: a record either belongs to one site or to all sites; one filter expression, applied everywhere, decides what a visitor of a given site may see. Combined with the published flag and the company filter. |
| Pages | Static pages backed by editable templates, with an address, a publication flag, a scheduled publication date, indexing control, header and footer options, per-page visibility (public, signed in, restricted group, password) and per-page tracking. |
| Menus | A per-site menu tree with ordering, nesting, mega-menu content, group restriction and automatic visibility from the target page. |
| Copy-on-write | Editing a shared template while acting for one site silently produces a site-specific copy, relocates the inheriting templates, duplicates the attached pages, and repoints the menus — leaving the other sites untouched. |
| Address rewriting | Permanent and temporary redirects, rewrites that rename a route while keeping the old address alive, and address suppression returning "not found", each optionally scoped to one site. |
| Language in the address | A language prefix in the path, automatic redirection to the visitor's preferred language, canonical addresses and alternate-language links. |
| Editable content | Inline editing of template fragments and of record fields, with a saving contract, a sanitisation policy, an image and attachment library, a snippet library and per-site style customisation files. |
| Themes | Installable themes shipping templates, assets, attachments, pages and menus as *theme records* that are copied into live records per site, with a documented removal and re-installation behaviour. |
| Form builder | A generic public form that writes a record of a whitelisted entity, with per-entity field whitelists, required-field enforcement, file attachment handling, an anti-robot challenge and an email notification. |
| Visitors and tracking | An anonymous visitor record keyed by a cookie, its link to a partner once known, and a page-visit log with visit counts. |
| Storefront catalogue | Published product templates and variants, public categories as a tree, ribbons, tags, attribute filters, full-text search with fuzzy matching, sorting, pagination and a grid layout model. |
| Product page | The combination contract (which variant a set of attribute choices resolves to), price and comparison price, availability text, images and documents, alternative and accessory products, unit-price display, reviews. |
| Cart | Session-bound draft order, line matching, quantity verification, stock limitation, combo and optional-product lines, zero-price blocking, accessory suggestions, reorder history, warnings. |
| Checkout | An ordered, per-site configurable step list: cart, address, extra information, confirmation, payment; guest, optional-account and mandatory-account modes; billing and delivery addresses; delivery method and rate; pickup points. |
| Payment and confirmation | Hand-off to the payment domain, the landing page, the confirmation page, the order state change and the confirmation message. |
| Abandoned-cart recovery | Detection of abandoned carts after a configurable delay, the filter that decides which of them deserve a message, the scheduled job and the recovery link that revives the cart. |
| Wish list and comparison | Per-visitor or per-partner wish lists with stock notifications, and a side-by-side comparison of products by attribute category. |
| Portal | The authenticated document area: the home page counters, the paginated document lists, the shared-record access token, the record preview with its message thread, and the access-granting wizard. |
| Forum | Questions, answers, comments, votes, accepted answers, tags, moderation, closing reasons and the complete reputation (karma) economy with every award and every gate. |
| Blog | Blogs, posts, tags, tag categories, cover properties, per-post visibility and the reading statistics. |
| Tracked links | Short addresses with campaign parameters and click statistics. |
| Public profiles | A public page per user showing reputation, badges and activity, with a per-user visibility choice. |

---

## 2. Entities of the domain

### 2.1 Site, structure and content

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Website | `website` | `website` | One public site: domain name, company, languages, theme, public user and every site-level setting. |
| Website Menu | `website.menu` | `website_menu` | One entry of one site's navigation tree. |
| Website Page | `website.page` | `website_page` | One static page: an address plus the template that renders it plus publication data. |
| Website Controller Page | `website.controller.page` | `website_controller_page` | A generated list-and-detail page pair for an arbitrary entity, published under a chosen path prefix. |
| Website Technical Page | `website.technical.page` | database view `website_technical_page` | A read-only listing of the code-defined addresses of the site, for the page manager. |
| View (template) | `ir.ui.view` | `ir_ui_view` | The renderable template; extended here with a site reference, a visibility mode, a password and a tracking flag. |
| Website Route | `website.route` | `website_route` | The catalogue of code-defined addresses, refreshed from the routing table, used when defining a rewrite. |
| Website Rewrite | `website.rewrite` | `website_rewrite` | One redirect, rewrite or suppression rule, optionally scoped to one site. |
| Website Visitor | `website.visitor` | `website_visitor` | One anonymous or identified visitor, keyed by a cookie. |
| Page Visit | `website.track` | `website_track` | One visit of one visitor to one page at one instant. |
| Website Snippet Filter | `website.snippet.filter` | `website_snippet_filter` | A named, parameterised record selection that a dynamic content block renders. |
| Website Configurator Feature | `website.configurator.feature` | `website_configurator_feature` | One selectable feature (a page or an application) offered by the initial site configurator. |
| Page Properties Wizard | `website.page.properties` | transient | The dialogue that edits a page's address, name, publication, visibility and options. |
| Robots Editor | `website.robots` | transient | The dialogue that edits the robot-exclusion text of a site. |
| Blocked Domain List Editor | `website.custom_blocked_third_party_domains` | transient | The dialogue that edits the list of third-party domains blocked before consent. |
| Assets Utility | `website.assets` | abstract | The service that writes per-site style customisation files. |
| Text Processor | `website.html.text.processor` | abstract | The service that rewrites generated page text during initial configuration. |

### 2.2 Theme records

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Theme Template | `theme.ir.ui.view` | `theme_ir_ui_view` | A template shipped by a theme, copied into a live template when the theme is applied to a site. |
| Theme Asset | `theme.ir.asset` | `theme_ir_asset` | An asset declaration shipped by a theme. |
| Theme Attachment | `theme.ir.attachment` | `theme_ir_attachment` | A file shipped by a theme. |
| Theme Menu | `theme.website.menu` | `theme_website_menu` | A menu entry shipped by a theme. |
| Theme Page | `theme.website.page` | `theme_website_page` | A page shipped by a theme. |
| Theme Utilities | `theme.utils` | abstract | The service that enables and disables named templates when a theme is applied. |

### 2.3 Mixins that other domains inherit

| Entity | Transport name | One-line purpose |
|---|---|---|
| Search Engine Metadata | `website.seo.metadata` | Title, description, keywords, social image and slug name for any published record. |
| Multi-site Restriction | `website.multi.mixin` | The single optional site reference and the "may this record be shown on the current site" test. |
| Published Flag | `website.published.mixin` | The publication flag, the publication right, the public address of a record. |
| Multi-site Published Flag | `website.published.multi.mixin` | The combination of the two above, with a site-aware published computation and search. |
| Searchable | `website.searchable.mixin` | The contract an entity implements to appear in the site-wide search. |
| Cover Properties | `website.cover_properties.mixin` | The serialised cover-image settings of a blog, post or event. |
| Page Options | `website.page_options.mixin` | Header and footer visibility, header overlay and header colours of a page-like record. |
| Portal Record | `portal.mixin` | The access token and the public address of a record shared with a customer. |

### 2.4 Storefront

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Public Category | `product.public.category` | `product_public_category` | One node of the storefront category tree (distinct from the internal product category). |
| Product Ribbon | `product.ribbon` | `product_ribbon` | A coloured corner label displayed over a product card. |
| Product Image | `product.image` | `product_image` | One extra image or video of a product template or variant. |
| Storefront Unit | `website.base.unit` | `website_base_unit` | A reference unit used to display a price per unit of measure. |
| Extra Field | `website.sale.extra.field` | `website_sale_extra_field` | A product field added to the specification table of the product page. |
| Checkout Step | `website.checkout.step` | `website_checkout_step` | One step of the checkout progress bar of one site, with its address, order, icon and publication flag. |
| Product Feed | `product.feed` | `product_feed` | A merchant product feed definition with its filter, its address and its validity state. |
| Wish List Entry | `product.wishlist` | `product_wishlist` | One product a visitor or partner saved for later, with the price seen at the time. |
| Attribute Category | `product.attribute.category` | `product_attribute_category` | A grouping of product attributes used to organise the comparison table. |
| Coupon Share Wizard | `coupon.share` | transient | Produces an address that applies a promotional code and lands on a chosen page. |

The storefront also extends entities owned by other domains; those extensions are specified here:
Sales Order (`sale.order`) and Sales Order Line (`sale.order.line`) — the cart; Product Template and
Product Variant (`product.template`, `product.product`) — publication, storefront ordering, ribbon,
alternative and accessory products; Price List (`product.pricelist`) — site availability and code
entry; Partner (`res.partner`) — publication and storefront address rules; Delivery Method
(`delivery.carrier`) — publication and storefront availability; Loyalty Program
(`loyalty.program`) — site restriction.

### 2.5 Portal

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Portal Share Wizard | `portal.share` | transient | Sends a record's public address and access token to chosen recipients. |
| Portal Access Wizard | `portal.wizard` | transient | Grants or revokes portal access for the contacts of selected records. |
| Portal Access Wizard Line | `portal.wizard.user` | transient | One contact inside the access-granting dialogue, with its current state and target e-mail address. |

### 2.6 Forum, blog, links and profiles

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Forum | `forum.forum` | `forum_forum` | One discussion space with its reputation table, its default mode and its welcome content. |
| Forum Post | `forum.post` | `forum_post` | One question, answer or comment, with votes, acceptance, state and moderation data. |
| Forum Vote | `forum.post.vote` | `forum_post_vote` | One up-vote or down-vote of one user on one post. |
| Forum Tag | `forum.tag` | `forum_tag` | One label attached to questions of one forum. |
| Closing Reason | `forum.post.reason` | `forum_post_reason` | A reason offered when closing a question, with its "offensive" flag. |
| Blog | `blog.blog` | `blog_blog` | One blog with its cover, subtitle and site restriction. |
| Blog Post | `blog.post` | `blog_post` | One article with its cover, tags, publication data and reading statistics. |
| Blog Tag | `blog.tag` | `blog_tag` | One label attached to articles. |
| Blog Tag Category | `blog.tag.category` | `blog_tag_category` | A grouping of blog labels for the filter side bar. |
| Tracked Link | `link.tracker` | `link_tracker` | A short address that records clicks and carries campaign parameters (owned by the marketing domain; its storefront usage is described here). |

---

## 3. Reading order

1. `README.md` — this file: scope, entity list, vocabulary.
2. `glossary.md` — every term used below in full. Read it before anything else.
3. `entities.md` — every field of every entity with type, default, computation and rules.
4. `state-machines.md` — the publication machine, the page-visibility machine, the cart-to-order
   machine, the checkout step machine, the forum post machine, the visitor machine, the blog post
   machine and the product feed machine.
5. `calculations.md` — every formula: site resolution, price display, availability, cart totals,
   delivery cost, reputation points, pagination, fuzzy search distance, abandoned-cart delay.
6. `workflows.md` — the operational procedures, above all the cart algorithm, the checkout
   algorithm and the copy-on-write algorithm.
7. `business-rules.md` — every validation, constraint, permission check and edge case with the
   exact message text.
8. `accounting-effects.md` — what this domain does and does not post, and the exact hand-off to the
   selling and payment domains.
9. `configuration.md` — settings, parameters, sequences, default records, groups, access rights,
   record rules and scheduled jobs.
10. `interfaces.md` — navigation, views, named remote operations, the complete address catalogue,
    printable documents, message templates and external services.
11. `acceptance-criteria.md` — numbered Given/When/Then scenarios with concrete numbers.

---

## 4. Dependencies on other domains

| Domain | What this domain takes from it | Where it is specified |
|---|---|---|
| Products and catalogue | Product templates, variants, attributes, attribute values, combinations, combos, documents, tags. | [`../products-and-catalog/entities.md`](../products-and-catalog/entities.md) |
| Pricing and price lists | The price computation algorithm, rule selection, the price list of a visitor. | [`../pricing-and-pricelists/calculations.md`](../pricing-and-pricelists/calculations.md) |
| Sales | The sales order, its lines, the confirmation algorithm, the invoicing, the portal acceptance. | [`../sales/workflows.md`](../sales/workflows.md) |
| Loyalty and promotions | Promotional codes, reward lines, the application algorithm on a cart. | [`../loyalty-and-promotions/workflows.md`](../loyalty-and-promotions/workflows.md) |
| Payment providers | Transactions, tokens, the express-checkout contract, the post-processing that confirms the order. | [`../payment-providers/state-machines.md`](../payment-providers/state-machines.md) |
| Delivery and shipping | Delivery methods, rate requests, pickup points, the shipping line. | [`../delivery-and-shipping/calculations.md`](../delivery-and-shipping/calculations.md) |
| Inventory operations | Free-to-promise quantities used by the storefront availability rules. | [`../inventory-operations/calculations.md`](../inventory-operations/calculations.md) |
| Taxes | Tax computation for displayed prices and for the order. | [`../taxes/calculations.md`](../taxes/calculations.md) |
| Messaging and activities | The message thread on a record, the mail templates, the notification recipients. | [`../messaging-and-activities/workflows.md`](../messaging-and-activities/workflows.md) |
| Identity and access | Users, groups, the public user, sign-up, access rights and record rules. | [`../identity-and-access/configuration.md`](../identity-and-access/configuration.md) |
| Contacts and organisations | Partners, addresses, countries, country states, languages. | [`../contacts-and-organizations/entities.md`](../contacts-and-organizations/entities.md) |
| Customer relationship management | The lead created by a public contact form. | [`../customer-relationship-management/workflows.md`](../customer-relationship-management/workflows.md) |
| Learning, surveys and gamification | Badges and the reputation display on public profiles. | [`../learning-surveys-and-gamification/entities.md`](../learning-surveys-and-gamification/entities.md) |

---

## 5. What this domain does not cover

- The internal back-office user interface framework, its client-side rendering and its assets
  pipeline, except where the public page rendering depends on them.
- Hosting, deployment, process management and server sizing — out of scope by instruction.
- The price computation itself: only its *invocation* from the storefront and its *display* rules
  are specified here; the algorithm lives in
  [`../pricing-and-pricelists/calculations.md`](../pricing-and-pricelists/calculations.md).
- The sales order confirmation side effects (procurement, invoicing) — see
  [`../sales/workflows.md`](../sales/workflows.md).
- The payment provider protocols — see [`../payment-providers/interfaces.md`](../payment-providers/interfaces.md).

---

## 6. Vocabulary shortcuts used throughout

| Term | Meaning in this folder |
|---|---|
| **Site** | One record of the Website entity. "Current site" is the one the present request resolved to. |
| **Generic record** | A record whose site reference is empty: it is shared by every site. |
| **Specific record** | A record whose site reference names one site: only that site sees it. |
| **Public user** | The anonymous identity a request runs as when nobody is signed in; each site names its own. |
| **Front end** | Request handling for a public page (as opposed to the back office). |
| **Cart** | A draft sales order bound to a site and to the visitor's session. |
| **Reputation** | The numeric score a forum participant accumulates; it gates every forum operation. |
| **Copy-on-write** | The rule that writing to a generic record while acting for one site creates and writes a specific copy instead. |
